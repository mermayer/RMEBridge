# RME Bridge

Steuere deinen RME ADI-2 über eine Webseite im Heimnetz – mit einem ESP32-S3, USB-MIDI und einem optionalen IR-Sender zum Ein- und Ausschalten. Kein Roon, RoonPilot oder ständig laufender Computer erforderlich.

Control your RME ADI-2 from a web page on your home network, using an ESP32-S3, USB-MIDI and an optional IR transmitter for power control. No Roon, RoonPilot or continuously running computer is required.

## Benutzerhandbuch / User guide

| Thema / Topic | Deutsch | English |
|---|---|---|
| Hier beginnen / Start here | [Überblick und Wegweiser](docs/user/de/index.md) | [Introduction and guide](docs/user/en/index.md) |
| Board und Firmware / Board and firmware | [Anschlüsse und Flashen](docs/user/de/installation.md) | [Connections and flashing](docs/user/en/installation.md) |
| Ersteinrichtung / First setup | [WLAN und DAC verbinden](docs/user/de/first-start.md) | [Connect Wi-Fi and the DAC](docs/user/en/first-start.md) |
| Webseite / Website | [Alle Seiten erklärt](docs/user/de/web-interface.md) | [Every page explained](docs/user/en/web-interface.md) |
| DAC-Einstellungen / DAC settings | [Regler, Schalter und Optionen](docs/user/de/dac-settings.md) | [Knobs, switches and options](docs/user/en/dac-settings.md) |
| Profile und Sicherung / Profiles and backup | [Speichern und Wiederherstellen](docs/user/de/profiles.md) | [Save and restore](docs/user/en/profiles.md) |
| Hilfe / Help | [Fehlerhilfe und Begriffe](docs/user/de/troubleshooting.md) | [Troubleshooting and glossary](docs/user/en/troubleshooting.md) |
| REST-API | [Einführung mit Beispielen](docs/user/de/api.md) | [Introduction with examples](docs/user/en/api.md) |

Die Anleitung beginnt beim leeren Board und setzt keine ESP32- oder MIDI-Kenntnisse voraus. Sie berücksichtigt ADI-2 DAC, ADI-2 Pro und ADI-2/4 Pro SE. Die verfügbaren Einstellungen richten sich nach dem erkannten Gerät; Einschränkungen sind in den jeweiligen Kapiteln erklärt.

The guide starts with an empty board and assumes no ESP32 or MIDI knowledge. It covers ADI-2 DAC, ADI-2 Pro and ADI-2/4 Pro SE. Available controls depend on the identified device; limitations are explained in the relevant chapters.

![RME Bridge: DAC control, volume knob and confirmed output values](docs/user/assets/screenshots/en-control.png)

Die Abbildungen zeigen die Bridge-Oberfläche mit Beispieldaten. WLAN-Namen und IP-Adressen darin sind keine persönlichen Gerätedaten. Dieses Repository enthält die Benutzer- und API-Dokumentation sowie die dazugehörigen Abbildungen, keine Bridge-Firmware.

Illustrations show the Bridge interface with example data. Their Wi-Fi names and IP addresses are not personal device information. This repository contains user and API documentation and its illustrations, not Bridge firmware.

## API-Referenz / API reference

- [REST-API Deutsch](docs/api/de.md)
- [REST API English](docs/api/en.md)
- [OpenAPI-Spezifikation / OpenAPI specification](docs/api/openapi.json)

Die API ist für ein vertrauenswürdiges lokales Netz bestimmt und besitzt keine Anmeldung. Keine Portfreigabe oder öffentliche Internet-Erreichbarkeit einrichten.

The API is intended for a trusted local network and has no authentication. Do not expose it to the public internet or forward a router port.
