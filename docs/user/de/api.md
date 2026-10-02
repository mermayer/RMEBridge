# REST-API für eigene Anwendungen

[← Fehlerbehebung](troubleshooting.md) · [Zur Übersicht](index.md)

Dieses Kapitel ist optional. Für die normale Bedienung reichen die Webseiten. Die **REST-API** ist eine weitere Schnittstelle zu derselben Bridge: Programme können Status und Einstellungen als **JSON** abfragen und freigegebene Änderungen über HTTP anfordern. Roon oder RoonPilot werden nicht benötigt.

## Zuerst nur lesen

Ersetze `192.0.2.55` durch deine tatsächliche Bridge-IP. Öffne im Browser:

```text
http://192.0.2.55/api/v1
http://192.0.2.55/api/v1/status
http://192.0.2.55/api/v1/capabilities
http://192.0.2.55/api/v1/parameters
```

Diese **GET**-Abfragen verändern den DAC nicht. JSON-Feldnamen sind für Programme bestimmt: `online` beschreibt DAC-Verfügbarkeit; `selectedTarget` ist das Bearbeitungsziel, nicht der physisch aktive Ausgang. `null` bedeutet unbekannt oder nicht bestätigt, nicht automatisch Null oder „Aus“.

Windows-Beispiel, ebenfalls nur lesend:

```powershell
$bridgeBaseUrl = 'http://192.0.2.55'
$dacStatus = Invoke-RestMethod -Uri "$bridgeBaseUrl/api/v1/status"
$dacStatus | ConvertTo-Json -Depth 6
```

Neue Anwendungen verwenden **`/api/v1`**. Die unversionierten internen Webseitenpfade sind nicht der verbindliche Integrationsweg.

## Adressen und Einheiten

| Feld | Bedeutung |
|---|---|
| Modell `113` / `114` / `115` | ADI-2 DAC / ADI-2 Pro / ADI-2/4 Pro SE |
| Modellpräferenz `0` | Automatische Suche, keine erfundene DAC-Identität |
| Ziel `3` / `6` / `9` | Line Out / Phones 1/2 / Phones 3/4 beziehungsweise IEM |
| Parameteradresse `0` / `12` | Pro-Analogeingang / Geräteparameter |
| `volumeTenths` | Zehntel dB: `-675` = **−67,5 dB** |
| Loudness-Rohwerte | Halbe dB: Bass `10` = **+5 dB**, Vol-Ref `-98` = **−49 dB** |
| Balance / Width | Hundertstel: Width `100` = **1,00** |
| `min`, `max`, `step` | Bereich und Raster des einzelnen Parameters |
| `writable` | Aktuell freigegebener technischer Schreibweg |
| `usbEpoch` | Kennung der USB-Verbindung; schützt vor alten Zuständen nach Neuverbindung |

Skalen unterscheiden sich. Lies die Metadaten jeder Option, statt global umzurechnen. Nur bestätigte Parameter des erkannten Modells werden gelistet. Modellpräferenz ersetzt keine Erkennung. Details: [technische API-Referenz](../../api/de.md) und [OpenAPI JSON](../../api/openapi.json).

## Verfügbare Endpunkte

| Methode | Pfad unter `/api/v1` | Zweck |
|---|---|---|
| GET | leer, also `/api/v1` | Version und Links |
| GET | `/status` | USB, MIDI, Modell, Pegel, AutoDark, IR und Ausgangsstatus |
| GET | `/capabilities` | Verfügbarkeit und Sperrgründe |
| GET | `/parameters` | Bestätigte Nicht-EQ-Werte und Grenzen |
| GET | `/time` | Zeitabgleich, UTC oder unbekannt, Laufzeit |
| GET | `/network` | WLAN-Status ohne Passwort |
| GET | `/settings` | Sprache, Farbe, Modellpräferenz und Ziel |
| PUT | `/settings/language` | `de` oder `en` speichern |
| PUT | `/settings/accent-color` | Farbe der Webseitenpalette speichern |
| PUT | `/settings/model` | Such-/IR-Präferenz speichern |
| PUT | `/settings/target` | Bearbeitungsziel speichern, keine physische Umschaltung |
| PUT | `/volume` | Einen begrenzten absoluten Pegelschritt anfordern |
| PUT | `/auto-dark` | AutoDark ein/aus |
| PUT | `/parameters` | Einen freigegebenen Parameter ändern |
| POST | `/output/toggle` | Freigegebene physische DAC-Umschaltung |
| POST | `/refresh` | Neue DAC-Statusabfrage anfordern |
| POST | `/ir/power` | Modellbezogenen Ein-/Aus-Code senden |
| GET | `/operations/{id}` | MIDI-Schreibauftrag verfolgen |
| GET / DELETE | `/events` | Log lesen / bewusst löschen |
| GET / POST | `/profiles` | Bibliothek lesen / bestätigten Zustand erfassen |
| PUT / DELETE | `/profiles/{id}` | Profil neu erfassen / löschen |
| GET / PUT | `/profiles/backup` | Bibliothek exportieren / vollständig ersetzen |
| PUT | `/network` | WLAN-Daten speichern, nur über geschützten Einrichtungs-AP |

Es gibt keinen einzelnen Hintergrund-Endpunkt zum **Anwenden** eines Profils. Die Webseite führt Zielwahl und bestätigte Befehle als Folge aus. Eigene Clients müssen deren Reihenfolge und Abbruchbedingungen selbst umsetzen.

## Aufträge bestätigen lassen

**HTTP 202 Accepted** heißt „angenommen“, nicht „DAC bereits geändert“. Die Antwort enthält `operationId` und `statusUrl`. Frage die URL ab, bis `state` **`confirmed`** oder **`failed`** ist, dann den tatsächlichen Zustand erneut lesen. Die acht jüngsten Aufträge liegen im RAM; alte URLs können nach Neustart oder Verdrängung `404` liefern.

Beispiel **nur für einen frisch bestätigten Line-Out-Pegel von −60,0 dB**:

```http
PUT /api/v1/volume
Content-Type: application/json

{"target":3,"expectedTenths":-600,"valueTenths":-605}
```

Das fordert **−60,5 dB** an. Nicht ungeprüft diese festen Zahlen senden: Ziel und Anfangswert müssen zum aktuellen Status passen. Ein Befehl erlaubt höchstens **+1 dB oder −3 dB**, im 0,5-dB-Raster. Größere Wege benötigen bestätigte Teilschritte.

Parameter-Schreiben benötigt frische `expected`- und `usbEpoch`-Werte. Parallele MIDI-Aufträge werden nicht unbegrenzt ausgeführt. `409` kann beschäftigt, veraltet, nicht bereit oder ungültig bedeuten. Erst neu lesen und den Grund prüfen: Ein unbestätigter Befehl kann am Gerät bereits gewirkt haben.

## Nicht jede Webseitenfolge ist ein API-Befehl

Direkte `/settings/target`-Aufrufe führen **keine automatische Phones-Absenkung** aus. Eigene Clients müssen die sichere Zielwahl umsetzen. `/output/toggle` ändert außerdem nicht `selectedTarget`; die Webseite folgt nach Bestätigung separat mit der Zielwahl.

Toggle nicht blind wiederholen: Derselbe Befehl zweimal kann zurückschalten. Die technische Referenz nennt Freigabebedingungen und erwarteten Ausgangszustand. IR meldet **`sent: true`, `confirmed: false`** und behauptet damit keinen tatsächlichen Power-Zustand. Profilimport mit `confirmReplace: true` ersetzt die gesamte Bibliothek, verändert aber keine DAC-Einstellung.

## Zugriff und Referenz

Die API hat keine Anmeldung und ist für ein **vertrauenswürdiges lokales Netz** gedacht. Keine Router-Portfreigaben ins Internet. Kennwörter nicht in öffentliche Befehlsbeispiele schreiben.

Für Payloads, Fehler, Operationsergebnisse und Backup-Format: [deutsche technische Referenz](../../api/de.md), [English reference](../../api/en.md), [OpenAPI JSON](../../api/openapi.json). Verfügbarkeit immer am Gerät über `/capabilities` und für die einzelne Option über `/parameters` prüfen.

[Zurück zum Einstieg →](index.md)
