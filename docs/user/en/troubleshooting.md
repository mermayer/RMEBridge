# Troubleshooting and useful terms

[← Profiles](profiles.md) · [Next: REST API →](api.md)

First identify **which part** is not working: power, Wi-Fi, USB, MIDI replies or IR. Reaching the website does not prove MIDI works; an off DAC does not mean the Bridge failed. Change one thing at a time and check the result.

## No port appears when flashing

| Observation | Check and remedy |
|---|---|
| No board offered in the browser | Use UART/COM, a known data cable and a direct computer port. A charge-only cable can power the board without exposing a port. |
| Only USB JTAG/serial debug unit appears | Probably native USB: change to UART/COM. Do not copy RoonPilot's plug-rotation rule. |
| Windows shows an unknown device | Identify the USB-serial chip; for CH343 install the official [WCH driver](https://www.wch-ic.com/downloads/CH343SER_EXE.html) if needed. |
| Visible port cannot be opened | Close other serial monitors, flash tools or browser tabs using it. |
| Connection waits for the board | Use the [BOOT/RESET sequence](installation.md#2-enter-flashing-mode). The tool must report ESP32-S3. |
| Writing is interrupted | Try 115200 baud, a short data cable and stable power. Check files and offsets. Do not erase everything as a first response. |

The chip's ROM download mode may remain reachable after interrupted installation even when the application will not boot. Use BOOT/RESET and write the correct complete package. General diagnostics: [Espressif troubleshooting](https://docs.espressif.com/projects/esptool/en/latest/esp32s3/troubleshooting.html).

## Setup Wi-Fi or its form is missing

An unconfigured board starts **RME-Bridge-…**. Read boot messages at **115200 baud through UART/COM**. Check normal boot versus repeated restarts. The AP password is individual, not a shared default.

Setup Wi-Fi can disappear after successful home-network connection. This is normal; use the home IP rather than `192.168.4.1`.

If Bridge Wi-Fi shows only a DAC page without fields, explicitly open **http://192.168.4.1/setup**. The form is not the Network status page. Stay connected despite “No internet” and prevent automatic phone network switching.

## The setup password is lost

Check your private note of the first-start message. Before home Wi-Fi configuration, [boot messages](installation.md#4-read-the-setup-password) can show it again. This is not assured after configuration. The Network page does not display it.

**Erase Flash** and complete reinstallation create a new password but lose Wi-Fi details, preferences and profiles. Export profiles first if reachable. Erasure is a last deliberate resort, not ordinary Wi-Fi repair.

## The website is unreachable on home Wi-Fi

1. Check the **current IP** in the router or boot messages. An old DHCP address may belong to another device. The hostname starts with `rme-bridge-…`.
2. Enter **http://YOUR-IP/**. The Bridge serves HTTP, not HTTPS. The example `192.0.2.55` is not your Bridge.
3. Both clients must reach one another. Guest isolation, Wi-Fi client isolation or a VPN can prevent this despite “Wi-Fi connected”.
4. The Bridge needs compatible **2.4-GHz Wi-Fi**. Check SSID, password and router. Hotel browser logins and enterprise sign-in are outside setup.
5. After roughly 30 seconds without home Wi-Fi, protected setup Wi-Fi reappears. Correct credentials through `/setup`. The Bridge also keeps trying home Wi-Fi.

The phone can use 5 GHz if the router joins both bands into the same reachable network. Mutual reachability matters, not matching radio bands.

## The DAC never becomes ready

| Observation | Meaning and next step |
|---|---|
| Website works, DAC not connected | Bridge and Wi-Fi work. Switch on the DAC and check USB. |
| USB present, no MIDI values | Check **MIDI Control: ON**. USB detection alone is not a valid RME MIDI reply. |
| Wrong or unknown model | Check automatic detection, exact model and manufacturer firmware. Selecting an image cannot replace identification. |
| Values vanish after USB loss | Old values are deliberately not shown as current. Reconnect and wait for replies. |
| Board restarts when connecting something | Check power, cable and IR module. Rewire only without power; do not close arbitrary solder jumpers. |

The DAC stays on its **own supply**. Its cable belongs on native USB of the Bridge, not UART/COM. It cannot also connect to a computer over USB. If MIDI Control is absent, consult [RME downloads](https://rme-audio.de/downloads.html) for model and firmware support.

## Values change but no music plays

The Bridge carries **no USB audio**. Check separate audio input, source, mute, connected output and Pro operating mode. USB-MIDI cannot supply music to a selected USB-audio input.

You may be editing a channel you are not listening to. **Line Out / Phones / IEM** initially chooses an editing target. Read [physical switching](web-interface.md#physically-switch-the-active-dac-output) before concluding an inaudible level change is a fault.

## A control is grey or an option is missing

Models support different options. Unreported parameters are hidden. A value may be read-only or unchangeable in the current mode. Writes also need a current, confirmed starting value.

Convenient volume/AutoDark controls and profiles are enabled for the detected **ADI-2 DAC FS** identity in this firmware. Manual Pro selection cannot unlock them. The [model table](index.md#which-rme-dac-can-i-use) distinguishes functions.

A target change may briefly wait for a pending command. After failure read state again, check target and actual value, then continue. Do not send contradictory commands repeatedly.

## Phones cannot be selected or switched

On a **target change**, Phones level must be known. Above −60 dB the Bridge must first be able to lower it. Missing readings or write permission prevent completion.

For **Switch on DAC**, both headphone paths must additionally be known and at or below −60 dB. **Mute Line Out with headphones** must be **Toggle** or **Plugged in**. This runs the DAC toggle function, not arbitrary output selection. If unconfirmed, check actual state first: another toggle could switch back.

## IR is ready but On or Off does nothing

“IR output ready” does not detect a connected LED. With power disconnected check **GPIO14**, common ground and the module's specified supply. **5V/5Vin** is not a guaranteed output. Do not drive high-power LEDs directly from a GPIO.

Check aim and clear sight. Without a detected DAC choose its correct model; connected MIDI identity takes priority. A receipt confirms sending only. Check the device and later MIDI connection to see if it switched on. Do not hold Off: its separate code is sent once.

## A profile or backup behaves unexpectedly

| Observation | Explanation |
|---|---|
| Source or Loudness not restored | Not included: profiles store model, target, AutoDark and one level. |
| Applying leaves volume unchanged | Including saved volume is off by default. |
| Phones nevertheless became quieter | Target-change reduction is independent of optional profile volume. |
| Import leaves DAC unchanged | Import replaces the library only. **Apply profile** sends commands later. |
| Old profiles disappeared | A backup replaces the whole library instead of adding entries. |
| Stop did not recover old values | Further steps stop; confirmed changes remain. |
| RME Remote backup rejected | Different format and scope. |

Read current state after interruption. DAC changes or external level changes can stop application. [The profile chapter](profiles.md) explains repetition and backup.

## Time is missing or the event log is empty

The Bridge queries a time server after home Wi-Fi connects. Without a reply it invents no date. Boot, uptime and sequence remain useful. The browser timezone determines display and daylight-saving time; check its clock and timezone.

The log is a **100-entry ring**, not an archive. Oldest entries are replaced. RTC memory does not guarantee retention through power loss. Download important events as JSON. Ordinary volume steps are intentionally not logged. **Refresh** fetches new entries; **Problems only** can make an error-free log look empty.

## Network and privacy

Website and API use HTTP with **no login**. Any reachable network client can control permitted functions. Use a trusted network, no internet port forwarding and no unprotected public Wi-Fi. The AP password protects setup Wi-Fi, not a home-network login.

Profile backups contain no Wi-Fi credentials. Initial serial messages can contain the AP password: remove it before sharing. Check screenshots for private network details. Logs contain no Wi-Fi passwords but may contain profile names.

## Reporting a problem usefully

Record firmware version, exact DAC model and its firmware, board revision, sockets and last action. Include full error, time or boot/uptime and a log download if useful. Distinguish **website** from **DAC itself**. Do not share passwords or unredacted boot messages.

## A small glossary

| Term | Meaning here |
|---|---|
| Firmware / flashing | Board software / writing it into persistent memory |
| UART / COM | Serial computer connection, also the power path here |
| Native USB / host | Connection through which the Bridge controls the DAC as a device |
| MIDI | Control information, not audio in this arrangement |
| AP / access point | The Bridge's setup Wi-Fi |
| DHCP / IP | Router assigns the address used to open the Bridge |
| Actual / target value | Confirmed state / intended setting |
| dB / dBu | Relative digital level / analogue reference unit, not interchangeable |
| AutoDark / standby | Blank display / standby device state |
| Profile / library | Limited snapshot / all Bridge profiles |
| REST / JSON | HTTP programming interface / structured text format |

[Next: Access state and settings through the API →](api.md)
