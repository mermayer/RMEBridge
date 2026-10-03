# RME Bridge

[English](README.md)

Steuere deinen RME ADI-2 über eine Webseite im Heimnetz – mit einem ESP32-S3, USB-MIDI und einem optionalen IR-Sender zum Ein- und Ausschalten. Kein Roon, RoonPilot oder ständig laufender Computer erforderlich.

## Benutzerhandbuch

| Thema | Anleitung |
|---|---|
| Hier beginnen | [Überblick und Wegweiser](docs/user/de/index.md) |
| Board und Firmware | [Anschlüsse und Flashen](docs/user/de/installation.md) |
| Ersteinrichtung | [WLAN und DAC verbinden](docs/user/de/first-start.md) |
| Webseite | [Alle Seiten erklärt](docs/user/de/web-interface.md) |
| DAC-Einstellungen | [Regler, Schalter und Optionen](docs/user/de/dac-settings.md) |
| Profile und Sicherung | [Speichern und Wiederherstellen](docs/user/de/profiles.md) |
| Hilfe | [Fehlerhilfe und Begriffe](docs/user/de/troubleshooting.md) |
| REST-API | [Einführung mit Beispielen](docs/user/de/api.md) |

Die Anleitung beginnt beim leeren Board und setzt keine ESP32- oder MIDI-Kenntnisse voraus. Sie berücksichtigt ADI-2 DAC, ADI-2 Pro und ADI-2/4 Pro SE. Die verfügbaren Einstellungen richten sich nach dem erkannten Gerät; modellabhängige Besonderheiten sind in den jeweiligen Kapiteln erklärt.

Die Installation ist über einen eigenen RMEBridge-Webinstaller vorgesehen, wie bei RoonPilot und der IR Bridge. Der Installer wird separat bereitgestellt und ist noch nicht verfügbar; die Anleitung beschreibt bereits den vorgesehenen Ablauf.

![RME Bridge: DAC-Steuerung mit Lautstärkeregler und bestätigten Ausgangswerten](docs/user/assets/screenshots/de-control.png)

Die Abbildungen zeigen die Bridge-Oberfläche mit Beispieldaten. WLAN-Namen und IP-Adressen darin sind keine persönlichen Gerätedaten. Dieses Repository enthält die Benutzer- und API-Dokumentation sowie die dazugehörigen Abbildungen, keine Bridge-Firmware.

## API-Referenz

- [REST-API Deutsch](docs/api/de.md)
- [OpenAPI-Spezifikation](docs/api/openapi.json)

Die API ist für ein vertrauenswürdiges lokales Netz bestimmt und besitzt keine Anmeldung. Keine Portfreigabe oder öffentliche Internet-Erreichbarkeit einrichten.
