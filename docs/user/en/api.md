# REST API for your own applications

[← Troubleshooting](troubleshooting.md) · [Guide overview](index.md)

This chapter is optional. Web pages are sufficient for ordinary use. The **REST API** lets programs read state and settings as **JSON** and change settings over HTTP. Roon and RoonPilot are not required.

## Start with read-only requests

Replace `192.0.2.55` with your actual Bridge IP. Open in a browser:

```text
http://192.0.2.55/api/v1
http://192.0.2.55/api/v1/status
http://192.0.2.55/api/v1/capabilities
http://192.0.2.55/api/v1/parameters
```

These **GET** requests do not change the DAC. JSON is intended for programs: `online` describes DAC availability; `selectedTarget` is the editing target, not the physically active output. `null` means unknown or unconfirmed, not automatically zero or Off.

Windows example, also read-only:

```powershell
$bridgeBaseUrl = 'http://192.0.2.55'
$dacStatus = Invoke-RestMethod -Uri "$bridgeBaseUrl/api/v1/status"
$dacStatus | ConvertTo-Json -Depth 6
```

Use **`/api/v1`** for new integrations. The website's unversioned internal paths are not the stable integration contract.

## Addresses and units

| Field | Meaning |
|---|---|
| Model `113` / `114` / `115` | ADI-2 DAC / ADI-2 Pro / ADI-2/4 Pro SE |
| Model preference `0` | Automatic probing, not a fabricated DAC identity |
| Target `3` / `6` / `9` | Line Out / Phones 1/2 / Phones 3/4 or IEM |
| Parameter address `0` / `12` | Pro analogue input / device parameters |
| `volumeTenths` | Tenths of a dB: `-675` = **−67.5 dB** |
| Loudness raw values | Half-dB: bass `10` = **+5 dB**, Vol-Ref `-98` = **−49 dB** |
| Balance / Width | Hundredths: Width `100` = **1.00** |
| `min`, `max`, `step` | Bounds and grid of each parameter |
| `writable` | Whether this value can be changed in the current DAC state |
| `usbEpoch` | USB-session identity, guarding against stale state after reconnection |

Scales differ. Read each option's metadata instead of assuming one conversion. Only confirmed parameters of the identified model are listed. A model preference does not replace identification. Details: [technical reference](../../api/en.md) and [OpenAPI JSON](../../api/openapi.json).

## Available endpoints

| Method | Path under `/api/v1` | Purpose |
|---|---|---|
| GET | empty, meaning `/api/v1` | Version and links |
| GET | `/status` | USB, MIDI, model, levels, AutoDark, IR and output state |
| GET | `/capabilities` | Device functions and current operating conditions |
| GET | `/parameters` | Confirmed non-EQ values and bounds |
| GET | `/time` | Time synchronisation, UTC or unknown, uptime |
| GET | `/network` | Wi-Fi status without its password |
| GET | `/settings` | Language, accent, model preference and target |
| PUT | `/settings/language` | Save `de` or `en` |
| PUT | `/settings/accent-color` | Save a website-palette colour |
| PUT | `/settings/model` | Save probing/IR preference |
| PUT | `/settings/target` | Save editing target, no physical switch |
| PUT | `/volume` | Request one bounded absolute level step |
| PUT | `/auto-dark` | AutoDark on/off |
| PUT | `/parameters` | Change one parameter |
| POST | `/output/toggle` | Run the DAC's physical toggle function |
| POST | `/refresh` | Request a new DAC status query |
| POST | `/ir/power` | Send the model's On/Off code |
| GET | `/operations/{id}` | Follow a MIDI write operation |
| GET / DELETE | `/events` | Read / deliberately clear the log |
| GET / POST | `/profiles` | Read library / capture confirmed state |
| PUT / DELETE | `/profiles/{id}` | Recapture / delete profile |
| GET / PUT | `/profiles/backup` | Export / replace entire library |
| PUT | `/network` | Save credentials through protected setup AP only |

There is no single background endpoint to **apply** a profile. The website sequences target selection and confirmed commands. External clients must implement its order and interruption rules.

## Confirm operations

**HTTP 202 Accepted** means “queued”, not “DAC already changed”. The response includes `operationId` and `statusUrl`. Poll that URL until `state` is **`confirmed`** or **`failed`**, then read actual state again. The eight most recent operations remain in RAM; old URLs may return `404` after reboot or replacement.

Example **only for freshly confirmed Line Out at −60.0 dB**:

```http
PUT /api/v1/volume
Content-Type: application/json

{"target":3,"expectedTenths":-600,"valueTenths":-605}
```

This requests **−60.5 dB**. Do not send fixed numbers unchecked: target and starting value must match current state. One command allows at most **+1 dB or −3 dB**, on a 0.5-dB grid. Larger moves require confirmed segments.

Parameter writes require fresh `expected` and `usbEpoch` fields. MIDI operations cannot run in unlimited parallel. `409` can mean busy, stale, not ready or invalid. Read again and inspect the reason first: an unconfirmed command may already have taken effect.

## Website sequences are not single API commands

Direct `/settings/target` calls **do not automatically reduce Phones level**. Custom clients must implement safe target selection. `/output/toggle` also leaves `selectedTarget` unchanged; after confirmation the website follows with separate target selection.

Do not blindly repeat toggles: the same command twice can switch back. The technical reference specifies prerequisites and expected output state. IR reports **`sent: true`, `confirmed: false`**, making no claim about actual power. Import with `confirmReplace: true` replaces the whole profile library without changing DAC settings.

## Access and reference

The API has no login and is for a **trusted local network**. Do not expose it with router port forwarding. Keep passwords out of public command examples.

For payloads, errors, operation results and backup format: [English technical reference](../../api/en.md), [deutsche Referenz](../../api/de.md), [OpenAPI JSON](../../api/openapi.json). Check actual device availability through `/capabilities` and each option through `/parameters`.

[Return to the guide →](index.md)
