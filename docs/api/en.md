# RME Bridge – local REST API v1

API version: **1**. `GET /api/v1` reports the installed firmware version in `firmwareVersion`. The REST API is for a **trusted local
network only**. It has **no authentication** yet: do not forward a router
port or expose it to an untrusted network. The older `/api/...` paths remain
for the Bridge website but are not a stable integration contract. New clients
should use `/api/v1/...`.

## Read the API without changing the DAC

1. Find the Bridge IP on its Network page. Open
   `http://BRIDGE-IP/api/v1` in a browser. It returns JSON with
   `firmwareVersion` and links.
2. Open `/api/v1/status`, `/api/v1/capabilities`, and
   `/api/v1/parameters`. The latter lists confirmed non-EQ DAC values and
   model-specific limits. Its `writable: true` only means the guarded MIDI
   write operation is available for that parameter. Availability depends on
   the MIDI-identified model and the current DAC readback.
   `/api/v1/time` should show `synchronized: true` and `unixSeconds` once
   home Wi-Fi is ready; before synchronization it deliberately returns `null`.
Browser and PowerShell request examples are also available in the
[API chapter of the user guide](../user/en/api.md).

## Rules

- JSON over HTTP; responses use `Cache-Control: no-store`. Volume integers
  are **tenths of a dB**: `-605` means −60.5 dB. Consult `/capabilities`
  for limits and current operating conditions.
- `null` means the DAC value is not currently confirmed. An old value is
  not presented as current after a DAC outage.
- `selectedTarget` is the output **addressed by the Bridge**: `3` = Line
  Out, `6` = Phones 1/2, `9` = Phones 3/4 or IEM. `activeOutputCode` is a
  separate raw DAC status value. Saving `selectedTarget` does **not** switch
  the physical DAC output.
  Only the separate `POST /api/v1/output/toggle` triggers the physical
  output switch on a MIDI-identified ADI-2 DAC FS
  configured for Toggle Ph/Line or Toggle plugged. Both headphone levels
  must be DAC-confirmed and at or below −60 dB. After confirmation, the
  website follows the new active output with its control target; a direct
  API call does not change `selectedTarget`.
  When switching to Phones, the Bridge website reduces its confirmed level
  to at most −60 dB if needed before reporting the selection complete. A
  quieter level stays unchanged. Direct `/settings/target` API calls do not
  automatically perform this website sequence.
- The convenience endpoints for volume and AutoDark address a
  MIDI-identified ADI-2 DAC FS. `/api/v1/parameters` provides individually
  confirmed changes to documented non-EQ values for the identified model.
  Only entries with `writable: true` can be changed. A manual model
  preference never replaces MIDI identification. Routine DAC commands
  do not require additional confirmation dialogs.
- Raw parameter values have different scales: half-dB for Loudness gain,
  half-dB increments for its low-volume reference (raw `-98` means `−49.0 dB`), and hundredths for balance
  or width. Use each returned `min`, `max`, and `step` rather than assuming
  a global range.
- HTTP `202 Accepted` means queued, **not** confirmed. Poll the returned
  `statusUrl` for `state: "confirmed"`. The eight most recent operations
  remain in RAM; old URLs return `404` after reboot or replacement.
- IR Power confirms **transmission only** (`sent: true`,
  `confirmed: false`). It cannot assert the actual DAC power state.
- The Bridge synchronizes UTC using SNTP after home Wi-Fi has an IP address.
  Until then, `unixSeconds` is `null` and event `time` is `0`; uptime remains
  valid. The website renders event dates in the **browser's local time zone**.
- Accent colour is a website preference with nine available colours.
  It does not change DAC settings. The default is blue (`#5BA8FF`).
- Profiles are named snapshots of **DAC-confirmed values** stored persistently
  on the Bridge. Version 1 contains model ID, addressed output, AutoDark,
  and that output's volume only. Saving, importing, or deleting a profile
  never changes the DAC. Website application excludes volume by default;
  IR Power is never part of a profile.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v1` | API version, access warning, links |
| GET | `/api/v1/status` | USB/MIDI/DAC state, three outputs, AutoDark/display/standby, IR |
| GET | `/api/v1/time` | Synchronization state, Unix seconds or `null`, uptime |
| GET | `/api/v1/capabilities` | Read/write availability, current conditions, volume limits |
| GET | `/api/v1/settings` | Language, accent colour, model preference, addressed output |
| PUT | `/api/v1/settings/language` | Save `{"language":"de"}` or `"en"` |
| PUT | `/api/v1/settings/accent-color` | Save `{"accentColor":"#5BA8FF"}` from the nine-colour website palette |
| PUT | `/api/v1/settings/model` | Save `{"model":0}` for auto; `113`, `114`, `115` as probe/IR preference only |
| PUT | `/api/v1/settings/target` | Save `{"target":3}` / `6` / `9`; does not physically switch outputs |
| POST | `/api/v1/output/toggle` | Physically toggle the MIDI-identified ADI-2 DAC FS output; see below |
| GET | `/api/v1/network` | Wi-Fi status, never the password |
| PUT | `/api/v1/network` | `{"ssid":"…","password":"…"}`; protected setup AP only |
| PUT | `/api/v1/volume` | One bounded absolute volume step, below |
| PUT | `/api/v1/auto-dark` | `{"enabled":true}` or `false` |
| GET | `/api/v1/parameters` | Read confirmed non-EQ values and model limits |
| PUT | `/api/v1/parameters` | Queue exactly one guarded parameter change |
| GET | `/api/v1/operations/{id}` | DAC confirmation of a volume/AutoDark/parameter/toggle operation |
| POST | `/api/v1/refresh` | Queue a DAC status request, then read `/status` |
| POST | `/api/v1/ir/power` | `{"action":"on"}` or `"off"`; transmission receipt only |
| GET | `/api/v1/events` | Newest-first event records |
| DELETE | `/api/v1/events` | Deliberately clear the event log |
| GET | `/api/v1/profiles` | Read the profile library, at most eight entries |
| POST | `/api/v1/profiles` | `{"name":"Evening","target":3}`: capture DAC-confirmed state |
| PUT | `/api/v1/profiles/{id}` | Replace a profile with newly confirmed DAC state |
| DELETE | `/api/v1/profiles/{id}` | Delete a profile, without changing the DAC |
| GET | `/api/v1/profiles/backup` | Download the versioned JSON library backup |
| PUT | `/api/v1/profiles/backup` | Replace the library from a valid backup; requires `confirmReplace: true` |

### Toggle the physical output

Read `activeOutputCode`, `usbEpoch`, `outputToggleControl`, and any
`outputToggleReason` from `/status`. Only when `outputToggleControl` is true,
POST the freshly read values:

```json
{"expectedActiveOutputCode":0,"usbEpoch":5}
```

HTTP `202` returns a `statusUrl`. Poll it for `confirmed`, then read
`/status` again. `observed` is the new `activeOutputCode`. After a failure
or timeout, do not blindly repeat this non-idempotent toggle. `409` guards
stale state, the wrong model, unsuitable toggle configuration and unsafe
headphone levels. USB transmission alone is not DAC confirmation.

### Status and capabilities

`/status` exposes `online`, `usb`, `midi`, `usbProblem`, `vid`, `pid`,
`deviceId`, `model`, `busy`, `unconfirmed`, `clockSynchronized`,
`unixSeconds`, `selectedTarget`, `usbEpoch`,
`activeOutputCode`, `outputToggleControl`, `outputToggleReason`,
`irTransmitterReady`, `irModelId`, `irModelSource`,
`autoDark`, `displayMode`, `standby`, and `outputs`. Each output has
`address`, `name`, `volumeTenths`, `source`, `loudness`, `mute`.
`irModelSource`: `1` = confirmed over USB, `2` = manual preference,
`3` = last detected, `0` = unknown.

`/capabilities` contains `volume`, `autoDark`, `source`, `loudness`,
`mute`, `displayMode`, `standby`, `parameters`, `irPower`, and `outputToggle`.
Each has `readable`,
`writable`, and `reason`. Current reasons: `model_not_tested`,
`dac_not_ready`, `model_unknown`, `ir_unavailable`, `output_unknown`,
`toggle_not_configured`, `headphone_level_unknown`,
`unsafe_headphone_level`.
Volume adds `minTenths: -1145`, `maxTenths: 60`, `stepTenths: 5`,
`maxIncreaseTenths: 10`, and `maxDecreaseTenths: 30`.

`volume` and `autoDark` still describe their older convenience endpoints.
For `source`, `loudness`, `mute`, `displayMode`, and `standby`, `writePath`
points to the parameter API. The `parameters` feature reports overall
technical availability; for an individual setting, **always** use its
own `writable` field from `GET /parameters`.

### Non-EQ parameters

`GET /api/v1/parameters` returns `modelId`, `selectedTarget`, `usbEpoch`,
and `values`. Each reported entry includes `address`, `index`, `value`,
`min`, `max`, `step`, `risk`, and `writable`. A value not confirmed by the
DAC is omitted. Addresses: `0` = analogue input (Pro families), `3` =
Line Out, `6` = Phones 1/2, `9` = Phones 3/4/IEM, `12` = device. The list
covers input reference and processing; output source, level, mono, balance,
crossfeed, filters, Loudness, mute and dim; operating mode, phones, digital
paths, clock, display, standby, and key remapping. A remapped key may
trigger an existing EQ action, but this API does not edit an EQ curve.
EQ and EQ-preset contents are deliberately excluded. Loading or overwriting
the DAC's internal setups is also excluded: the original manual says setups
can affect level and current EQ settings, while the available MIDI table
does not describe a separate reliable confirmation of that action. Output
volume (`index: 12`) keeps its separately bounded
`/volume` path; sample rate is read-only while USB is connected.

The **DAC state** page shows the confirmed model and output volumes also
available from `GET /api/v1/status`, together with clock,
mode, source, reference, mute, Loudness,
and Crossfeed values from `GET /api/v1/parameters`.
`outputs[*].eqEnabled` and `bassTrebleEnabled` in `GET /api/v1/status`
add the MIDI-reported enable switches of the output EQ channels (`null`
until reported). The DSP column shows settings, not guaranteed audible
processing at every sample rate. The page checks the model and
`usbEpoch` in both responses; offline or mismatched snapshots never display
stale values. Unsupported parameters are omitted. Digital input sync,
bit depth, and physical output state are not
available as decoded UI or API fields.

For example, change DAC-reported Auto Standby from `0` to `1`:

```http
PUT /api/v1/parameters
Content-Type: application/json

{"address":12,"index":2,"value":1,"expected":0,"usbEpoch":7}
```

`expected` and `usbEpoch` must come from a **fresh** `/parameters` response.
An output parameter must also address `selectedTarget`. `risk: true` marks
possible signal-path or level changes but does not add a confirmation prompt.
HTTP `202` only means queued. Poll `statusUrl`
until `confirmed` or `failed`, then read the DAC value again. A USB drop or
mode change can prevent confirmation even if a command took effect: **do not
blindly retry**. Parallel writes are rejected.

The built-in bilingual website groups these controls into Output, Input,
Device, and Display. DAC commands are sent one at a time without dialogs
and checked against the DAC readback.

### Volume and operation confirmation

Read `/status` before every write. Example for Line Out when the DAC has
confirmed −60.0 dB (`-600`):

```http
PUT /api/v1/volume
Content-Type: application/json

{"target":3,"expectedTenths":-600,"valueTenths":-605}
```

The Bridge checks the selected output, confirmed starting value, 0.5-dB
grid, full range, and **at most +1.0 dB or −3.0 dB per command**. For
larger changes, a client must send confirmed segments. No write is made
without a current DAC value or while another command is active.

The response is HTTP `202` with `operationId`, `state: "pending"`,
`statusUrl`, and a matching `Location` header. Poll that URL until
`confirmed` or `failed`, for example:

```json
{"id":123,"state":"confirmed","kind":"volume","target":3,
 "requested":-605,"observed":-605,"failure":null}
```

`requested`/`observed` mean tenths of a dB for volume, and `0`/`1` for
AutoDark. Failure codes are `dac_mismatch`, `response_timeout`,
`usb_failed`, or `disconnected`. A late matching DAC report may correct
`dac_mismatch` to `confirmed`; critical automations should also check the
current `/status` value.

### Profiles and backup

`GET /api/v1/profiles` and `/profiles/backup` use the same library format.
The backup response adds `exportedAt` (Unix seconds or `null`):

```json
{"format":"rme-bridge-profiles","version":1,"maxProfiles":8,
 "volumeAppliedByDefault":false,"profiles":[
  {"id":1,"name":"Evening","modelId":113,"target":3,
   "autoDark":true,"volumeTenths":-675}]}
```

Names must be unique and at most 48 UTF-8 bytes. Capture requires an
online ADI-2 DAC FS with confirmed volume and AutoDark values. `POST`
creates a profile (`201`); `PUT /profiles/{id}` refreshes its snapshot
(`200`). Neither sends a DAC command. Profile format version 1 uses
`modelId: 113` for the ADI-2 DAC family; it must match the connected DAC
when applying the profile.

For restore, add `"confirmReplace":true` to the exported JSON and send
it with `PUT /profiles/backup`. The Bridge validates format, version,
model, ranges, IDs, and unique names **before** atomically replacing the
library. An empty backup removes all profiles. Wi-Fi credentials and
general Bridge preferences are not in this profile backup. Import does
not change the DAC.

Before applying a profile, the website displays its model, output,
AutoDark, and volume. It uses the existing versioned API operations:
select the addressed output, change AutoDark if needed, and optionally
move volume in individually confirmed bounded steps. It checks `usbEpoch`,
model, actual value, and each operation result between steps. External
clients can follow the same sequence; there is no background `apply`
endpoint yet. Previously confirmed steps are not rolled back on failure.

### Errors and events

`400` = malformed JSON/field; `403` = Wi-Fi write outside the setup AP;
`404` = unknown path/operation; `409` = DAC unavailable, stale value,
active command, or invalid DAC value; `500` = local save failure. On `409`,
`reason` is one of `busy`, `stale`, `not_ready`, `invalid`. Do not blindly
retry the same volume request: fetch `/status` again first.

`/events` returns `capacity`, `boot`, and `events` with `sequence`, `boot`,
`uptime` in seconds, `time` as Unix seconds (`0` before SNTP), `kind`,
`detail`, and `repeats`. Event kinds:
`1` boot, `2–6` Wi-Fi/setup, `7–10` USB, `11` DAC identified, `12–13`
DAC command, `14–16` local settings, `17` USB host error, `18–19` IR
Power, `20` clock synchronized, `21` accent changed, `22` profile saved,
`23` profile deleted, `24` profile library imported. The log resides in
RTC memory and may be lost on power loss.
