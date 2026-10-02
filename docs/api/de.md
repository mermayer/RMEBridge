# RME Bridge – lokale REST-API v1

API-Version: **1**. Die installierte Firmwareversion liefert `GET /api/v1` im Feld `firmwareVersion`. Die REST-API ist für ein **vertrauenswürdiges lokales Netz** bestimmt. Sie
hat noch **keine Anmeldung**; deshalb weder Portfreigabe noch Zugriff aus
einem nicht vertrauenswürdigen Netz einrichten. Die bisherigen `/api/...`-Endpunkte bleiben
für die Bridge-Webseite erhalten, sind aber kein stabiler Vertrag für andere
Programme. Neue Integrationen verwenden `/api/v1/...`.

## API auslesen, ohne den DAC zu verändern

1. Die Bridge-IP von ihrer Netzwerkseite ablesen. Im Browser
   `http://BRIDGE-IP/api/v1` öffnen. Es erscheint JSON mit
   `firmwareVersion` und den Links.
2. `http://BRIDGE-IP/api/v1/status`,
   `http://BRIDGE-IP/api/v1/capabilities` und
   `http://BRIDGE-IP/api/v1/parameters` öffnen. Letzteres zeigt die vom DAC
   bestätigten Nicht-EQ-Parameter samt Modellgrenzen. `writable: true`
   bedeutet, dass ein Schreibbefehl für diesen Parameter verfügbar ist.
   Maßgeblich sind das tatsächlich erkannte Modell und die aktuelle DAC-Rückmeldung.
   Unter `/api/v1/time` muss nach der WLAN-Verbindung
   `synchronized: true` mit `unixSeconds` erscheinen. Vorher ist der Wert
   bewusst `null`.
Beispiele für API-Aufrufe im Browser und mit PowerShell stehen auch im
[API-Kapitel des Benutzerhandbuchs](../user/de/api.md).

## Regeln

- JSON über HTTP; Antworten mit `Cache-Control: no-store`. Zahlen für
  Lautstärke sind **Zehntel dB**: `-605` bedeutet −60,5 dB. Die gültigen
  Werte und die aktuellen Bedienbedingungen liefert `/capabilities`.
- `null` bedeutet: Wert gegenwärtig nicht bestätigt. Ein DAC-Ausfall macht
  alte Statuswerte nicht zu neuen Istwerten.
- `selectedTarget` ist der von der Bridge **adressierte** Ausgang:
  `3` = Line Out, `6` = Phones 1/2, `9` = Phones 3/4 bzw. IEM.
  `activeOutputCode` ist ein separater, roher DAC-Status. Das Ändern von
  `selectedTarget` schaltet den physischen Ausgang **nicht** um.
  Nur der separate Befehl `POST /api/v1/output/toggle` betätigt die
  physische Toggle-Funktion des per MIDI erkannten ADI-2 DAC FS mit aktivem
  „Toggle Ph/Line“ oder „Toggle plugged“. Beide Phones-Pegel müssen vom DAC bestätigt
  und höchstens −60 dB sein. Die Webseite folgt nach bestätigter
  Umschaltung mit ihrem Steuerziel dem neuen Ausgang; ein direkter API-Aufruf
  ändert `selectedTarget` nicht.
  Die Bridge-Webseite begrenzt beim Wechsel zu Phones dessen bestätigten
  Pegel nötigenfalls auf −60 dB, bevor sie die Auswahl als abgeschlossen
  meldet. Ein bereits leiserer Pegel bleibt unverändert. Direkte API-Aufrufe
  von `/settings/target` führen diese Webseitenfolge nicht automatisch aus.
- Die Komfort-Endpunkte für Lautstärke und AutoDark adressieren den per MIDI
  erkannten ADI-2 DAC FS. `/api/v1/parameters` bietet einzelne,
  rückbestätigte Änderungen der dokumentierten Nicht-EQ-Werte für das
  erkannte Modell. Nur Einträge mit `writable: true` können geändert werden.
  Eine manuelle Modellauswahl ersetzt keine MIDI-Erkennung. Gewöhnliche
  DAC-Befehle benötigen keine zusätzliche Bestätigungsabfrage.
- Rohwerte aus `/parameters` haben je Einstellung unterschiedliche Einheiten:
  etwa halbe dB bei Loudness-Gain und beim Low-Volume-Referenzwert
  (`-98` bedeutet dort `−49,0 dB`) und Hundertstel bei Balance/Breite. Immer `min`, `max` und `step` der
  konkreten Antwort verwenden, keine allgemeinen Zahlenbereiche annehmen.
- `202 Accepted` heißt lediglich „angenommen“. Erst der Abruf der
  `statusUrl` mit `state: "confirmed"` bestätigt die DAC-Rückmeldung.
  Bis zu acht aktuelle Aufträge sind im RAM abrufbar; nach Neustart oder
  späterer Überschreibung liefert die alte URL `404`.
- IR-Power bestätigt nur das **Senden** (`sent: true`, `confirmed: false`).
  Ohne gesonderte DAC-Rückmeldung ist der tatsächliche Ein-/Aus-Zustand
  unbekannt.
- Die Bridge synchronisiert UTC per SNTP, sobald das Heim-WLAN eine IP-Adresse
  hat. Vor der ersten Synchronisierung ist `unixSeconds: null`; Ereignisse
  haben dann `time: 0` und nur eine zuverlässige Laufzeit `uptime`. Die
  Webseite zeigt Zeitstempel in der **lokalen Zeitzone des Browsers**.
- Die Akzentfarbe ist eine Webseiten-Einstellung mit neun verfügbaren Farben.
  Sie beeinflusst keine DAC-Einstellungen; der Standard ist Blau (`#5BA8FF`).
- Profile sind benannte, dauerhaft auf der Bridge gespeicherte **Momentaufnahmen
  bestätigter DAC-Werte**. Version 1 enthält nur Modellkennung, adressierten
  Ausgang, AutoDark und dessen Lautstärke. Speichern, Importieren und Löschen
  ändern den DAC nicht. Beim Anwenden über die Webseite bleibt die Lautstärke
  standardmäßig unverändert; IR-Power ist nie Teil eines Profils.

## Endpunkte

| Methode | Pfad | Bedeutung |
|---|---|---|
| GET | `/api/v1` | API-Version, Einschränkung und Links |
| GET | `/api/v1/status` | USB/MIDI/DAC-Zustand, alle drei Ausgänge, AutoDark/Display/Standby, IR |
| GET | `/api/v1/time` | Zeitsynchronisierung, Unix-Sekunden oder `null`, Laufzeit |
| GET | `/api/v1/capabilities` | Les-/Schreibbarkeit je Funktion, aktuelle Bedingungen und Lautstärkegrenzen |
| GET | `/api/v1/settings` | Sprache, Akzentfarbe, Modellpräferenz und adressierter Ausgang |
| PUT | `/api/v1/settings/language` | `{"language":"de"}` oder `"en"` speichern |
| PUT | `/api/v1/settings/accent-color` | `{"accentColor":"#5BA8FF"}` aus der neunfarbigen Webseiten-Palette speichern |
| PUT | `/api/v1/settings/model` | `{"model":0}` für Automatik; `113`, `114`, `115` nur als Such-/IR-Präferenz |
| PUT | `/api/v1/settings/target` | `{"target":3}` / `6` / `9` speichern; kein physisches Umschalten |
| POST | `/api/v1/output/toggle` | Aktiven Ausgang am per MIDI erkannten ADI-2 DAC FS physisch umschalten; siehe unten |
| GET | `/api/v1/network` | WLAN-Status ohne Passwort |
| PUT | `/api/v1/network` | `{"ssid":"…","password":"…"}`; ausschließlich über den geschützten Setup-AP |
| PUT | `/api/v1/volume` | Ein begrenzter absoluter Lautstärkeschritt, siehe unten |
| PUT | `/api/v1/auto-dark` | `{"enabled":true}` oder `false` |
| GET | `/api/v1/parameters` | Bestätigte Nicht-EQ-Werte und Modellgrenzen lesen |
| PUT | `/api/v1/parameters` | Genau eine geschützte Parameteränderung annehmen |
| GET | `/api/v1/operations/{id}` | DAC-Bestätigung eines Lautstärke-/AutoDark-/Parameter-/Toggle-Auftrags |
| POST | `/api/v1/refresh` | Neue DAC-Statusabfrage anstoßen; danach `/status` lesen |
| POST | `/api/v1/ir/power` | `{"action":"on"}` oder `"off"`; nur Sendebestätigung |
| GET | `/api/v1/events` | Ereignisse, neueste zuerst |
| DELETE | `/api/v1/events` | Ereignislog bewusst leeren |
| GET | `/api/v1/profiles` | Profilbibliothek lesen, maximal acht Einträge |
| POST | `/api/v1/profiles` | `{"name":"Abendhören","target":3}`: bestätigten DAC-Zustand speichern |
| PUT | `/api/v1/profiles/{id}` | Profil mit bestätigtem aktuellem DAC-Zustand erneut speichern |
| DELETE | `/api/v1/profiles/{id}` | Profil aus der Bibliothek löschen; keine DAC-Änderung |
| GET | `/api/v1/profiles/backup` | Versionierten JSON-Export der Bibliothek abrufen |
| PUT | `/api/v1/profiles/backup` | Bibliothek aus gültigem Backup vollständig ersetzen; `confirmReplace: true` erforderlich |

### Physischen Ausgang umschalten

`/status` liefert `activeOutputCode`, `usbEpoch`, `outputToggleControl` und
gegebenenfalls `outputToggleReason`. Nur bei `outputToggleControl: true`
einen einmaligen POST mit dem **soeben gelesenen** Code und Epoch senden:

```json
{"expectedActiveOutputCode":0,"usbEpoch":5}
```

Die Antwort `202` enthält `statusUrl`. Dort auf `confirmed` warten und danach
`/status` erneut lesen. `observed` enthält dann den neuen
`activeOutputCode`; bei `failed` oder Timeout nicht blind wiederholen.
`409` schützt vor veraltetem Status, falschem Modell, ungeeigneter
Toggle-Konfiguration oder zu hohem Phones-Pegel. Ohne DAC-Rückmeldung gilt
ein USB-Sendeerfolg ausdrücklich nicht als Umschaltung.

### Status und Fähigkeiten

`/status` liefert `online`, `usb`, `midi`, `usbProblem`, `vid`, `pid`,
`deviceId`, `model`, `busy`, `unconfirmed`, `clockSynchronized`,
`unixSeconds`, `selectedTarget`, `usbEpoch`,
`activeOutputCode`, `outputToggleControl`, `outputToggleReason`,
`irTransmitterReady`, `irModelId`, `irModelSource`,
`autoDark`, `displayMode`, `standby` und `outputs`.
Jedes Element von `outputs` enthält `address`, `name`, `volumeTenths`,
`source`, `loudness`, `mute`. `irModelSource` ist `1` = über USB bestätigt,
`2` = manuelle Präferenz, `3` = zuletzt erkannt, `0` = unbekannt.

`/capabilities` enthält die Funktionen `volume`, `autoDark`, `source`,
`loudness`, `mute`, `displayMode`, `standby`, `parameters`, `irPower`,
`outputToggle`. Jedes Objekt hat
`readable`, `writable`, `reason`. Gründe sind derzeit `model_not_tested`,
`dac_not_ready`, `model_unknown`, `ir_unavailable`, `output_unknown`,
`toggle_not_configured`, `headphone_level_unknown` und
`unsafe_headphone_level`.
`volume` ergänzt `minTenths: -1145`, `maxTenths: 60`, `stepTenths: 5`,
`maxIncreaseTenths: 10` und `maxDecreaseTenths: 30`.

`volume` und `autoDark` beschreiben weiterhin die alten Komfort-Endpunkte.
Bei `source`, `loudness`, `mute`, `displayMode` und `standby` nennt
`writePath` die neue Parameter-API. `parameters` meldet deren technische
Verfügbarkeit insgesamt; für eine konkrete Einstellung ist **immer**
`GET /parameters` mit ihrem eigenen `writable`-Feld maßgeblich.

### Nicht-EQ-Parameter

`GET /api/v1/parameters` liefert `modelId`, `selectedTarget`, `usbEpoch`
und `values`. Jeder vorhandene Wert enthält `address`, `index`, `value`,
`min`, `max`, `step`, `risk`, `writable`. Fehlende DAC-Rückmeldungen fehlen
auch in der Liste. Adressen: `0` = analoger Eingang (Pro-Modelle), `3` =
Line Out, `6` = Phones 1/2, `9` = Phones 3/4/IEM, `12` = Gerät. Die Liste
umfasst Eingangspegel/-verarbeitung, Ausgangsquelle und -pegel, Mono,
Balance, Crossfeed, Filter, Loudness, Mute/Dim, Betriebsart, Kopfhörer,
Digitalwege, Takt, Display, Standby und Tastenbelegung. Die Tastenbelegung
kann vorhandene EQ-Aktionen einer Taste zuweisen, bearbeitet aber keine
EQ-Kurve. Equalizer und EQ-Preset-Inhalte fehlen bewusst. Das Laden und
Überschreiben geräteinterner Setups bleibt ebenfalls ausgenommen: Laut
Original-Handbuch können Setups auch Pegel und aktuelle EQ-Einstellungen
ändern, ohne dass die vorliegende MIDI-Tabelle eine sichere, getrennte
Bestätigung dafür beschreibt. Die gesonderte Lautstärke (`index: 12` am
Ausgang) bleibt beim
bewährten begrenzten `/volume`-Pfad; Sample Rate wird über USB nur gelesen.

Die Seite **DAC-Zustand** zeigt die bestätigte Geräteidentität und die
Ausgangslautstärken, die auch `GET /api/v1/status` liefert, sowie die Takt-, Modus-,
Quellen-, Referenz-, Mute-, Loudness- und Crossfeed-Werten aus
`GET /api/v1/parameters`. Die Darstellung prüft Modell und `usbEpoch` beider
Antworten; bei Offline-Zustand oder nicht passendem Stand zeigt sie keine
alten Werte. `outputs[*].eqEnabled` und `bassTrebleEnabled` in
`GET /api/v1/status` geben zusätzlich die per MIDI gemeldeten Schalter der
Ausgangs-EQ-Kanäle an (`null`, bis ein Wert vorliegt). Die DSP-Spalte zeigt
Einstellungen, nicht die bei jeder Abtastrate tatsächliche Signalwirkung.
Nicht verfügbare Parameter fehlen. Digital-Input-Sync, Bit-Tiefe und
physische Ausgangszustände sind derzeit weder
in der Ansicht noch als dekodierte API-Felder verfügbar.

Beispiel: vom DAC gemeldetes Auto Standby von `0` auf `1` setzen:

```http
PUT /api/v1/parameters
Content-Type: application/json

{"address":12,"index":2,"value":1,"expected":0,"usbEpoch":7}
```

`expected` und `usbEpoch` müssen aus einer **frischen** `/parameters`-Antwort
stammen. Bei einem Ausgangsparameter muss dessen Adresse außerdem dem
`selectedTarget` entsprechen. `risk: true` kennzeichnet eine mögliche
Signalweg- oder Pegeländerung, löst aber keine zusätzliche Bestätigung aus.
HTTP `202` bestätigt nur das Einreihen. Danach `statusUrl` bis `confirmed`
oder `failed` abfragen und den Wert erneut vom DAC lesen. Ein USB-Abbruch
oder Moduswechsel kann die Bestätigung verhindern, obwohl der Befehl bereits
wirkte: in diesem Fall **nicht blind wiederholen**. Es wird niemals ein
zweiter Schreibbefehl parallel angenommen.

Die eingebaute Webseite bietet dafür die Bereiche Ausgang, Eingang, Gerät
und Anzeige in Deutsch und Englisch. DAC-Befehle werden ohne Dialog einzeln
gesendet und anschließend anhand der DAC-Rückmeldung geprüft.

### Lautstärke und Auftragsbestätigung

Vor jedem Schreibbefehl `/status` lesen. Beispiel für Line Out bei
bestätigten −60,0 dB (`-600`):

```http
PUT /api/v1/volume
Content-Type: application/json

{"target":3,"expectedTenths":-600,"valueTenths":-605}
```

Die Bridge prüft Zielausgang, aktuell bestätigten Wert, 0,5-dB-Raster,
Gesamtbereich und **höchstens +1,0 dB bzw. −3,0 dB je Befehl**. Größere
Änderungen muss ein Client in bestätigte Teilschritte zerlegen. Ohne aktuellen
DAC-Wert oder bei parallelem Auftrag wird nicht geschrieben.

Antwort: HTTP `202` mit `operationId`, `state: "pending"`, `statusUrl` und
gleichnamigem `Location`-Header. Diese URL bis `confirmed` oder `failed`
abfragen, zum Beispiel:

```json
{"id":123,"state":"confirmed","kind":"volume","target":3,
 "requested":-605,"observed":-605,"failure":null}
```

`requested` und `observed` sind bei Lautstärke Zehntel dB, bei AutoDark
`0`/`1`. Ein `failed`-Auftrag nennt `dac_mismatch`, `response_timeout`,
`usb_failed` oder `disconnected`. `dac_mismatch` kann durch eine verspätete
passende DAC-Meldung nachträglich zu `confirmed` werden; für kritische
Automationen zusätzlich den aktuellen `/status` prüfen.

### Profile und Backup

`GET /api/v1/profiles` und `/profiles/backup` liefern dasselbe
Bibliotheksformat. Die Backup-Antwort ergänzt `exportedAt` (Unix-Sekunden
oder `null`). Beispiel:

```json
{"format":"rme-bridge-profiles","version":1,"maxProfiles":8,
 "volumeAppliedByDefault":false,"profiles":[
  {"id":1,"name":"Abendhören","modelId":113,"target":3,
   "autoDark":true,"volumeTenths":-675}]}
```

Profilnamen müssen eindeutig sein und dürfen höchstens 48 UTF-8-Bytes
umfassen. Für die Momentaufnahme muss ein passender ADI-2 DAC FS online sein;
beide gespeicherten Werte müssen vom DAC bestätigt sein. Ein `POST` legt ein
neues Profil an (`201`), ein `PUT /profiles/{id}` ersetzt dessen Momentaufnahme
(`200`). Weder Vorgang sendet einen DAC-Befehl. Das Profilformat Version 1
verwendet `modelId: 113` für die ADI-2-DAC-Familie; die Modellkennung muss
beim Anwenden mit dem angeschlossenen DAC übereinstimmen.

Zum Wiederherstellen das exportierte JSON mit `confirmReplace: true`
ergänzen und per `PUT /profiles/backup` senden. Die Bridge validiert Format,
Version, Modell, Wertebereiche, IDs und eindeutige Namen **vor** dem
atomaren Ersetzen der Bibliothek. Ein leeres Backup löscht alle Profile.
WLAN-Daten und die allgemeinen Bridge-Einstellungen sind nicht Bestandteil
dieses Profil-Backups. Der DAC bleibt beim Import unverändert.

Die Webseite zeigt vor „Profil anwenden“ Modell, Ausgang, AutoDark und Pegel.
Sie verwendet die vorhandenen versionierten API-Aufträge: Zielausgang wählen,
AutoDark nur bei Unterschied senden und optional die Lautstärke in begrenzten,
einzeln bestätigten Schritten nachführen. `usbEpoch`, Modell, Istwert und
Auftragsergebnis werden dazwischen geprüft. Externe API-Clients können
denselben Ablauf verwenden; ein eigener Hintergrund-`apply`-Endpunkt ist
noch nicht vorhanden. Bei Abbruch bleiben bereits bestätigte Schritte
erhalten; es gibt keinen ungeprüften Rollback.

### Fehler und Ereignisse

`400` = ungültiges JSON/Feld, `403` = WLAN-Schreiben außerhalb des Setup-AP,
`404` = unbekannter Pfad/Auftrag, `409` = DAC nicht bereit, veralteter Wert,
laufender Auftrag oder ungültiger DAC-Wert, `500` = lokales Speichern
fehlgeschlagen. Bei `409` liefert `reason` einen maschinenlesbaren Wert:
`busy`, `stale`, `not_ready` oder `invalid`. Nicht blind denselben
Lautstärkebefehl wiederholen: erst `/status` neu lesen.

`/events` liefert `capacity`, `boot` und `events` mit `sequence`, `boot`,
`uptime` (Sekunden), `time` (Unix-Sekunden oder `0` vor SNTP), `kind`,
`detail`, `repeats`. `kind`: `1` Start,
`2–6` WLAN/Setup, `7–10` USB, `11` DAC erkannt, `12–13` DAC-Befehl,
`14–16` lokale Einstellungen, `17` USB-Host-Fehler, `18–19` IR-Power,
`20` Zeit synchronisiert, `21` Akzentfarbe geändert, `22` Profil gespeichert,
`23` Profil gelöscht, `24` Profilbibliothek importiert. Das Log liegt im
RTC-Speicher; Stromverlust kann es löschen.
