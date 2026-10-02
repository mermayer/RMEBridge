# Fehlerbehebung und wichtige Begriffe

[← Profile](profiles.md) · [Weiter: REST-API →](api.md)

Prüfe zuerst, **welcher Teil** nicht funktioniert: Versorgung, WLAN, USB-Verbindung, MIDI-Antwort oder IR-Sendung. Eine erreichbare Webseite beweist noch keine MIDI-Verbindung; ein ausgeschalteter DAC bedeutet nicht, dass die Bridge ausgefallen ist. Ändere jeweils nur eine Sache und prüfe danach den Status.

## Beim Flashen erscheint kein Anschluss

| Beobachtung | Prüfen und beheben |
|---|---|
| Kein Board im Browser auswählbar | UART-/COM-Buchse verwenden. Sicher datenfähiges Kabel und direkten Computeranschluss probieren. Ein Ladekabel liefert Strom, macht aber keinen Port sichtbar. |
| Nur USB JTAG/serial debug unit erscheint | Vermutlich die native Buchse: zur UART-/COM-Buchse wechseln. Nicht die RoonPilot-Regel mit dem gedrehten Stecker übernehmen. |
| Windows zeigt ein unbekanntes Gerät | USB-Seriell-Chip prüfen; bei CH343 gegebenenfalls den offiziellen [WCH-Treiber](https://www.wch-ic.com/downloads/CH343SER_EXE.html) installieren. |
| Sichtbarer Port lässt sich nicht öffnen | Andere serielle Monitore, Flashprogramme oder Browser-Tabs schließen, die ihn verwenden. |
| Verbindung wartet auf das Board | Die [BOOT-/RESET-Schrittfolge](installation.md#2-den-flashmodus-erreichen) verwenden. Das Werkzeug muss ESP32-S3 melden. |
| Schreiben bricht ab | Mit 115200 Baud, kurzem Datenkabel und stabiler Versorgung versuchen. Dateien und Adressen kontrollieren. Nicht vorsorglich alles löschen. |

Nach einer unterbrochenen Installation kann der ROM-Downloadmodus des Chips weiterhin erreichbar sein, obwohl die Anwendung nicht startet. BOOT/RESET verwenden und das passende vollständige Paket erneut schreiben. Allgemeine Verbindungsdiagnose: [Espressif-Fehlerhilfe](https://docs.espressif.com/projects/esptool/en/latest/esp32s3/troubleshooting.html).

## Das Einrichtungs-WLAN oder Formular fehlt

Ein nicht eingerichtetes Board startet **RME-Bridge-…**. Lies seine Startmeldungen mit **115200 Baud an UART/COM**. Prüfe, ob es startet oder wiederholt neu startet. Das AP-Passwort ist individuell, kein gemeinsames Standardpasswort.

Nach erfolgreicher Heimnetz-Verbindung kann das Einrichtungs-WLAN verschwinden. Das ist normal. Verwende dann die Heimnetz-IP statt `192.168.4.1`.

Siehst du im Bridge-WLAN nur eine DAC-Seite ohne Eingabefelder, öffne ausdrücklich **http://192.168.4.1/setup**. Das WLAN-Formular ist nicht die Netzwerk-Statusseite. Bleibe trotz „Kein Internet“ im Bridge-WLAN; verhindere das automatische Zurückwechseln des Smartphones.

## Das Einrichtungs-Passwort ist verloren

Prüfe deine private Notiz der Erststartzeile. Vor der Heimnetz-Einrichtung kann die [Startmeldung](installation.md#4-das-einrichtungs-passwort-ablesen) es erneut nennen. Bei einer bereits eingerichteten Bridge ist diese Ausgabe nicht zugesichert. Die Netzwerkseite zeigt es nicht an.

**Erase Flash** und vollständige Neuinstallation erzeugen ein neues Passwort, löschen aber WLAN-Daten, Bridge-Einstellungen und Profile. Falls die Bridge noch erreichbar ist, vorher Profile exportieren. Löschen ist der letzte bewusste Ausweg, keine gewöhnliche WLAN-Reparatur.

## Die Webseite ist im Heimnetz nicht erreichbar

1. Prüfe im Router oder in den Startmeldungen die **aktuelle IP**. Eine alte DHCP-Adresse kann inzwischen einem anderen Gerät gehören. Der Hostname beginnt mit `rme-bridge-…`.
2. Gib **http://DEINE-IP/** in die Adresszeile ein. Die Bridge bietet HTTP, nicht HTTPS. `192.0.2.55` aus den Beispielen ist nicht deine Bridge.
3. Beide Geräte müssen einander erreichen können. Gastnetz-Trennung, WLAN-Client-Isolation oder ein VPN können das verhindern, obwohl „WLAN verbunden“ angezeigt wird.
4. Die Bridge benötigt kompatibles **2,4-GHz-WLAN**. WLAN-Name, Passwort und Router prüfen. Hotel-Browserlogins und Unternehmensanmeldungen sind nicht Teil der Einrichtung.
5. Nach etwa 30 Sekunden ohne Heimnetz-Verbindung startet der geschützte Zugangspunkt wieder. Dort unter `/setup` korrigieren. Die Bridge versucht gleichzeitig weiter, das Heimnetz zu erreichen.

Das Smartphone darf 5 GHz verwenden, wenn der Router beide Bänder im gleichen erreichbaren Netz verbindet. Entscheidend ist gegenseitige Erreichbarkeit, nicht dieselbe Funkfrequenz.

## Der DAC wird nicht bereit

| Beobachtung | Bedeutung und nächster Schritt |
|---|---|
| Webseite erreichbar, DAC nicht verbunden | Bridge und WLAN laufen. DAC einschalten und USB prüfen. |
| USB vorhanden, keine MIDI-Werte | **MIDI Control: ON** am DAC prüfen. USB-Erkennung allein ist keine gültige RME-MIDI-Antwort. |
| Unbekannte oder falsche Modellkennung | Automatische Erkennung, DAC-Modell und Hersteller-Firmware prüfen. Manuelle Bildauswahl ersetzt keine Identifikation. |
| Werte verschwinden nach USB-Ausfall | Alte Werte werden absichtlich nicht als aktuell angezeigt. Neu verbinden und auf Antworten warten. |
| Board startet beim Anschließen neu | Versorgung, Kabel und IR-Modul prüfen. Verkabelung nur stromlos ändern, keine beliebigen Lötbrücken schließen. |

Der DAC bleibt am **eigenen Netzteil**. Das Datenkabel gehört an native USB der Bridge, nicht UART/COM. Der DAC kann nicht zugleich per USB am Computer hängen. Fehlt MIDI Control, hilft die [RME-Dokumentation](https://rme-audio.de/Downloadbereich.html), Modell und passende Firmware zu prüfen.

## Werte ändern sich, aber es kommt keine Musik

Die Bridge transportiert **kein USB-Audio**. Prüfe separaten Audioeingang, Quelle, Mute, angeschlossenen Ausgang und Pro-Betriebsart. USB-MIDI liefert keine Musik an einen ausgewählten USB-Audioeingang.

Vielleicht bearbeitest du einen anderen Kanal als den gerade hörbaren. **Line Out / Phones / IEM** ist zunächst das Bearbeitungsziel. Lies die [physische Umschaltung](web-interface.md#den-aktiven-dac-ausgang-wirklich-umschalten), bevor du aus einem unhörbaren Pegelschritt auf einen Defekt schließt.

## Ein Regler ist grau oder eine Option fehlt

Nicht jeder DAC unterstützt jede Option. Nicht gemeldete Modellparameter werden ausgeblendet. Ein Wert kann nur lesbar oder im aktuellen Modus unveränderbar sein. Schreiben benötigt außerdem einen aktuellen, bestätigten Anfangswert.

Die komfortablen Lautstärke-/AutoDark-Regler und Profile sind in dieser Firmware für die erkannte **ADI-2 DAC FS**-Kennung freigegeben. Eine manuelle Pro-Auswahl aktiviert sie nicht. Die [Modelltabelle](index.md#welcher-rme-dac-passt) trennt verfügbare Funktionen.

Ein Zielwechsel kann kurz auf einen offenen Befehl warten. Nach einem Fehler neu lesen, Ziel und tatsächlichen Wert prüfen, dann fortfahren. Nicht mehrere widersprüchliche Befehle hintereinander senden.

## Phones lässt sich nicht auswählen oder umschalten

Beim **Zielwechsel** auf Phones muss sein Pegel bekannt sein. Über −60 dB muss die Bridge ihn erst absenken können. Fehlender Wert oder Schreibfreigabe verhindern den Abschluss.

Für **Am DAC umschalten** müssen zusätzlich beide Kopfhörerpfade bekannt und höchstens −60 dB laut sein. **Line Out stumm bei Kopfhörer** muss **Umschalten** oder **Eingesteckt** sein. Dieser Schalter führt die DAC-Toggle-Funktion aus, keine freie Ausgangswahl. Bleibt die Umschaltung unbestätigt, erst den tatsächlichen Zustand prüfen: Ein weiterer Toggle könnte zurückschalten.

## IR ist bereit, aber Ein oder Aus wirkt nicht

„IR-Ausgang bereit“ erkennt keine angeschlossene LED. Kontrolliere stromlos **GPIO14**, gemeinsame Masse und modulgerechte Versorgung. **5V/5Vin** ist kein zugesicherter 5-V-Ausgang. Keine leistungsstarke LED direkt am GPIO betreiben.

Prüfe Ausrichtung und freie Sicht. Ohne erkannten DAC das richtige Modell wählen; bei USB-Verbindung hat die erkannte Identität Vorrang. Eine Sendebestätigung bestätigt nur das Senden. Am DAC und seiner späteren MIDI-Verbindung siehst du, ob er eingeschaltet wurde. Den Aus-Knopf nicht lange halten: Der separate Code wird einmal gesendet.

## Profil oder Backup verhält sich unerwartet

| Beobachtung | Erklärung |
|---|---|
| Quelle oder Loudness nicht wiederhergestellt | Nicht im Profil enthalten: Es speichert Modell, Ziel, AutoDark und einen Pegel. |
| Profil angewendet, Pegel unverändert | Die zusätzliche Anwendung der gespeicherten Lautstärke ist standardmäßig aus. |
| Phones wurde trotzdem leiser | Zielwechsel-Absenkung gilt unabhängig von optionaler Profil-Lautstärke. |
| Import ändert den DAC nicht | Import ersetzt nur die Bibliothek. Erst **Profil anwenden** sendet Befehle. |
| Alte Profile fehlen nach Import | Das Backup ersetzt alles, es wird nicht hinzugefügt. |
| Stoppen stellt alte Werte nicht wieder her | Weitere Schritte stoppen; bestätigte Änderungen bleiben. |
| RME-Remote-Backup wird abgewiesen | Anderes Format und anderer Sicherungsumfang. |

Bei einer Unterbrechung erst Werte neu lesen. DAC-Wechsel oder externe Pegeländerung können die Folge stoppen. [Die Profilanleitung](profiles.md) erklärt Wiederholen und Sichern.

## Uhrzeit fehlt oder das Ereignislog ist leer

Nach Heimnetz-Verbindung fragt die Bridge einen Zeitserver ab. Ohne Antwort wird kein Datum erfunden. Boot, Laufzeit und Sequenz bleiben nutzbar. Die Browser-Zeitzone bestimmt die Anzeige einschließlich Sommer-/Winterzeit; dessen Uhr und Zeitzone prüfen.

Das Log ist ein Ring mit **100 Einträgen**, kein Archiv. Alte Einträge werden verdrängt. Im RTC-Speicher kann es Stromausfälle nicht sicher überleben. Wichtige Ereignisse als JSON herunterladen. Normale Lautstärkeschritte werden bewusst nicht protokolliert. **Aktualisieren** holt neue Einträge; **Nur Probleme** kann ein fehlerfreies Log leer erscheinen lassen.

## Netzwerk und Datenschutz

Webseite und API verwenden HTTP und haben **keine Anmeldung**. Jeder erreichbare Netzclient kann freigegebene Funktionen steuern. Nur vertrauenswürdiges Netz nutzen, keine Internet-Portfreigabe und kein ungeschütztes öffentliches WLAN. Das AP-Passwort schützt den Zugangspunkt, ist aber keine Anmeldung im Heimnetz.

Profil-Backups enthalten keine WLAN-Zugangsdaten. Die serielle Erststartzeile enthält dagegen das AP-Passwort: vor Weitergabe entfernen. Screenshots auf private Netzwerkinformationen prüfen. Logs enthalten keine WLAN-Passwörter, können aber Profilnamen enthalten.

## Für eine hilfreiche Fehlermeldung

Notiere Firmwareversion, genaues DAC-Modell und dessen Firmware, Boardrevision, Buchsen und letzte konkrete Aktion. Ergänze vollständige Meldung, Zeitpunkt oder Boot/Laufzeit und gegebenenfalls einen Log-Download. Unterscheide **Webseite** und **DAC selbst**. Keine Kennwörter oder unbereinigten Erststartmeldungen mitsenden.

## Kleine Begriffshilfe

| Begriff | Bedeutung hier |
|---|---|
| Firmware / Flashen | Boardsoftware / in den dauerhaften Speicher schreiben |
| UART / COM | Serieller Computeranschluss, hier auch Versorgungsweg |
| Native USB / Host | Verbindung, über die die Bridge den DAC als Gerät steuert |
| MIDI | Steuerinformationen, keine Audiodaten in diesem Aufbau |
| AP / Zugangspunkt | Eigenes Einrichtungs-WLAN der Bridge |
| DHCP / IP | Router vergibt die Netzwerkadresse zum Aufrufen der Bridge |
| Istwert / Zielwert | Bestätigter Zustand / beabsichtigte Einstellung |
| dB / dBu | Relativer digitaler Pegel / analoge Referenzgröße, nicht austauschbar |
| AutoDark / Standby | Display dunkel / Bereitschaftszustand |
| Profil / Bibliothek | Begrenzter Zustand / alle Bridge-Profile |
| REST / JSON | HTTP-Programmierschnittstelle / strukturiertes Textformat |

[Weiter: Status und Einstellungen über die API erreichen →](api.md)
