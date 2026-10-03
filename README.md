# RME Bridge

[Deutsch](README.de.md)

Control your RME ADI-2 from a web page on your home network, using an ESP32-S3, USB-MIDI and an optional IR transmitter for power control. No Roon, RoonPilot or continuously running computer is required.

## User guide

| Topic | Guide |
|---|---|
| Start here | [Introduction and guide](docs/user/en/index.md) |
| Board and firmware | [Connections and flashing](docs/user/en/installation.md) |
| First setup | [Connect Wi-Fi and the DAC](docs/user/en/first-start.md) |
| Website | [Every page explained](docs/user/en/web-interface.md) |
| DAC settings | [Knobs, switches and options](docs/user/en/dac-settings.md) |
| Profiles and backup | [Save and restore](docs/user/en/profiles.md) |
| Help | [Troubleshooting and glossary](docs/user/en/troubleshooting.md) |
| REST API | [Introduction with examples](docs/user/en/api.md) |

The guide starts with an empty board and assumes no ESP32 or MIDI knowledge. It covers ADI-2 DAC, ADI-2 Pro and ADI-2/4 Pro SE. Available controls depend on the identified device; model-specific differences are explained in the relevant chapters.

Installation is planned through a dedicated RMEBridge web installer, following the same workflow as RoonPilot and the IR Bridge. The installer will be supplied separately and is not yet available; the guide already describes the intended steps.

![RME Bridge: DAC control, volume knob and confirmed output values](docs/user/assets/screenshots/en-control.png)

Illustrations show the Bridge interface with example data. Their Wi-Fi names and IP addresses are not personal device information. This repository contains user and API documentation and its illustrations, not Bridge firmware.

## API reference

- [REST API English](docs/api/en.md)
- [OpenAPI specification](docs/api/openapi.json)

The API is intended for a trusted local network and has no authentication. Do not expose it to the public internet or forward a router port.
