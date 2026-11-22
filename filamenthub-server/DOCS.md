# FilamentHub Server

Läuft rund um die Uhr auf deinem Pi. Er hört am Drucker mit (LAN-Modus) und speichert jeden Druck: Name, Start, Ende, fertig oder abgebrochen und welcher AMS-Slot benutzt wurde. Die Apps (Handy, Windows) holen die Liste und ziehen die Gramm von der richtigen Spule ab, auch wenn beim Druck keine App offen war.

## Einrichten

1. Reiter **Konfiguration** ausfüllen:
   - **Drucker-IP**: am besten im Router eine feste IP für den Drucker einstellen.
   - **Seriennummer**: steht am Drucker (Einstellungen → Gerät) oder in Bambu Studio.
   - **Zugangscode**: am Drucker unter Einstellungen → WLAN/LAN → Zugangscode.
   - **API-Key** (optional): eigenes Passwort. Leer lassen: der Pi erzeugt selbst einen Schlüssel (ab 1.3.8 Pflicht).
   - **gemini_api_key** (optional, ab 1.3.19): Schlüssel für die Foto-Erkennung. In der App ein Foto vom Spulen-Etikett machen, die Spule wird ausgefüllt. Kostenlosen Schlüssel holst du auf aistudio.google.com. Leer = Foto-Erkennung aus.
2. Am Drucker müssen **Nur-LAN-Modus** und **Entwicklermodus** an sein.
3. Reiter **Info**: „Beim Systemstart starten“ und „Watchdog“ einschalten, dann **Starten**.
4. Im Reiter **Protokoll** muss stehen: `Drucker ... LAN-Modus` und `FilamentHub Server läuft`.

## In der App

Menü → Verbindungen → **FilamentHub Server**: Adresse des Pi eintragen (zum Beispiel `192.168.68.10`, der Port 3000 wird ergänzt), den **Schlüssel** aus dem Reiter **Protokoll** (Zeile `Schlüssel für die Apps: …`) eintragen und auf **Prüfen** tippen. Unterwegs geht dieselbe Adresse über Tailscale. Mit Geräte-Sync-Passwort reicht es, den Schlüssel auf einem Gerät einzutragen.

## Was der Server ohne App macht (ab 1.1.0)

- **Push aufs Handy** über die Home-Assistant-App: Druck fertig/abgebrochen, Spule fast leer, Drucker aus, Warnungen. Ohne Einstellung geht der Push an alle Handys mit der Home-Assistant-App. Nur bestimmte Handys: bei **notify_service** z. B. `mobile_app_pixel_8` eintragen.
- **Foto am Druckende** (Kamera des Druckers, sonst die Kamera aus Home Assistant).
- **Abzug der Gramm** direkt in der Cloud der App, genau einmal je Druck. Die App schickt dem Server dafür einmal ihren Cloud-Zugang und die Slot-Zuordnung.
- **Aus nach Druck** und **Aus nach Trocknen** (Steckdose über Home Assistant).
- **Verzögerter Druck**: setzt zur Zeit fort, auch wenn alle Apps zu sind.
- **Warnung**, wenn ein Druck viel länger dauert als gewohnt, und **Spulen-Check** beim Start.
- **Sicherung** jede Nacht (Spulen, Verlauf, Druckliste), 30 Tage, in der App herunterladbar.

## Neu in 1.3.105 (Archiv 20 GB)

Das Datei-Archiv fasst jetzt bis 20 GB und 2000 Dateien (vorher 2 GB und 400). Einzelne Dateien dürfen weiter bis 512 MB groß sein.

## Neu in 1.3.104 (ESP-Datei vom Update-Server)

Die ESP-Datei `filament-ams.yaml` holt das Add-on jetzt von `https://filament.tdhub.de/updates/filament-ams.yaml` (die alte Adresse bei github.io gibt es nicht mehr). Das Add-on selbst kommt aus dem öffentlichen Repo `https://github.com/tomden234/filamenthub-addon`.

## Neu in 1.3.103 (Aufgeräumt)

Die doppelte Spiegelung der Update-Dateien im Hub ist entfernt (die Dateien liefert Nginx auf dem Homeserver). `GITHUB_RELEASE_TOKEN` wird nicht mehr gebraucht und kann aus `/etc/filamenthub.env` gelöscht werden.

## Neu in 1.3.102 (Eigener Token für Issues)

Fehlerberichte und „Geht nicht“-Meldungen können mit einem eigenen Token angelegt werden, der nur das Recht „Issues“ hat (Umgebung `GITHUB_TOKEN_ISSUES`, auf dem Homeserver in `/etc/filamenthub.env`). Er hat Vorrang vor `github_token`. Die Option `github_token` bleibt für die Sicherung außer Haus (Gists).

## Neu in 1.3.101 (Update-Diagnose)

Findet der Server die Update-Releases bei GitHub nicht (404), meldet er das jetzt als Fehler mit Hinweis auf die Schlüssel-Rechte. Der Stand der Update-Dateien steht im lokalen `/api/status-info`.

## Neu in 1.3.100 (Updates vom eigenen Server)

Der Server holt die Update-Dateien (Windows-Installer, `latest.yml`, APK, Handy-Oberfläche) aus den GitHub-Releases und liefert sie unter `/updates/` aus. Dafür braucht er in der Umgebung `GITHUB_RELEASE_TOKEN` (nur Leserecht auf Contents des Repos). Ohne Schlüssel bleibt alles beim Alten.

## Neu in 1.3.99 (Archiv-Umzug und Status-Info)

Archiv als ZIP im Format des Hubs (`/api/hub/archive/zip`, `/api/hub/archive/import-zip`), Liste (`/api/hub/archive/manifest`) und Übernahme vom alten Server mit Fortschritt (`/api/hub/archive/pull`). Neuer lokaler Status für den Hub: `/api/status-info`.

## Neu in 1.3.98 (Komplett-Sicherung mit Archiv)

Neue Schnittstellen `/api/hub/export-full` und `/api/hub/import-full`: Spulen, Verbrauch, Einstellungen und das Dateiarchiv als eine ZIP-Datei sichern und wieder einspielen (nur hinzufügen, nichts wird gelöscht).

## Neu in 1.3.97 (3D-Ansicht der Warteschlange)

Der Server liest das 3D-Modell der Platte aus der gespeicherten 3MF-Datei und liefert es als kleines Netz (höchstens 30.000 Dreiecke). Die App zeigt es drehbar, wenn man in der Warteschlange aufs Vorschaubild drückt und „3D“ wählt. Geht nur für Einträge, deren Datei auf dem Server gespeichert ist.

## Neu in 1.3.96 (Daten-Umzug, Health)

Neu: `GET /api/health` (ohne Anmeldung, nur ok und Version) für Überwachung. Export des Spulen-Dokuments als Datei und Import von Gerätedaten ohne Duplikate (Menü → Verbindungen → Cloud & Geräte-Sync). Vor dem Import legt der Server eine Sicherung an. Ordner per `DATA_DIR`, Adresse per `HOST`.

## Neu in 1.3.95 (Spule wirklich drehen)

Neuer Befehl „Chip neu lesen“ für einen AMS-Platz (1 bis 4). Das AMS dreht die Spule und liest den Chip, wie „Neu einlesen“ in Bambu Studio. Nur im Leerlauf. Genutzt von der Sandbox (Schalter „Echter Drucker dreht mit“).

## Neu in 1.3.94 (Bluetooth-Test)

- Neu: Der Server darf an das Bluetooth des Pi. Im Testmenü sucht der Pi auf Knopfdruck nach Bluetooth-Geräten, so siehst du, ob er deinen Etikettendrucker findet.

## Neu in 1.3.93 (Trocknen aus, Start prüfen)

- Warteschlangen-Taster: trocknet das AMS, schaltet der Pi das Trocknen ab und startet den Druck direkt.
- Der Eintrag bleibt in der Warteschlange, bis der Drucker den Druck wirklich angenommen hat (kein falsches „gestartet“ mehr).

## Neu in 1.3.92 (Rahmen passen)

- Objekt überspringen: Rahmen werden auf die Größe der Draufsicht umgerechnet und sitzen auf den Teilen.

## Neu in 1.3.91 (Teile selbst wählen)

- Objekt überspringen: Bild ist jetzt die Draufsicht der Platte, die Rahmen passen auf die Teile.
- Ausgewähltes Teil leuchtet umrandet, Liste und Bild folgen einander.

## Neu in 1.3.90 (Zeitraffer einstellbar)

Zeitraffer lässt sich in den Pi-Einstellungen ausschalten und der Bildabstand wählen. Bricht ein Druck ab, baut der Pi zusätzlich einen Rückblick aus den letzten 40 Bildern.

## Neu in 1.3.89 (Zeitraffer)

Der Pi nimmt während des Drucks alle 90 Sekunden ein Kamerabild auf und baut am Ende mit ffmpeg einen kurzen Zeitraffer-Clip (die letzten 8 bleiben). Abspielen in der App: Play-Symbol auf dem Foto in der Druckliste.

## Neu in 1.3.88 (Zeitreise)

Der Pi zeigt die Nachtsicherungen als Zeitreise: Vorschau je Tag und Zurückholen einer Spule, der fehlenden Spulen oder des ganzen Tages (mit Sicherheitskopie vorher).

## Neu in 1.3.87 (Objekt überspringen)

Der Pi liefert die Objekte des laufenden Drucks aus dem Datei-Archiv, damit die App einzelne Teile überspringen kann.

## Neu in 1.3.86 (Wartungsheft)

Der Pi zählt Druckstunden mit und meldet fällige Wartung (Stangen, Druckplatte, Düse, Riemen) mit Knopf „Erledigt“.

## Neu in 1.3.85 (Zurückholen)

- Neue Seiten: Vorschau der Sicherung außer Haus und Zurückholen (alles oder nur fehlende Spulen).
- Vor dem Zurückholen: Sicherheitskopie des aktuellen Stands nur auf der SSD, die Sicherung außer Haus bleibt unberührt.

## Neu in 1.3.84 (Umbau für die Sandbox)

- Das Lesen der Druckdateien (.3mf) liegt jetzt in src/utils/threeMf.ts, damit die Sandbox der App dieselben Regeln nutzt. Am Verhalten des Pi ändert sich nichts.

## Neu in 1.3.83 (Chip-Abgleich)

- Alle 30 min ohne laufenden Druck: Rest laut Bambu-Chip (remain × tray_weight) gegen remainingWeight. Abweichung > 50 g und > 15 % → Push mit „Chip“ / „App“. „Chip“ setzt das Gewicht und schreibt die Differenz in den Verlauf.

## Neu in 1.3.82 (Eigene Lösungen)

- Fehler-Push mit Texteingabe „Was hat geholfen?“ (Home-Assistant-App). Gespeichert je Fehlercode in /data/fixes.json (höchstens 200). Gleicher Fehler → „Letztes Mal half: …“ in der Meldung.

## Neu in 1.3.81 (Fehler-Melder)

- POST /api/hub/error-report: App-Fehler → GitHub-Issue (Option github_token). Je Fehler und Version einmal, höchstens 5 am Tag. IPs, Schlüssel, Adress-Zusätze und Mails werden entfernt.

## Neu in 1.3.80 (Erste-Schicht-Foto)

- Schicht 2 beginnt → Push mit Kamerabild und Knöpfen „Sieht gut aus“ / „Abbrechen“ (bricht nur genau diesen Druck ab). Einmal je Druck, nur wenn ein Bild da ist (Licht geht dafür kurz an).

## Neu in 1.3.79 (Licht fürs Foto)

- Kammerlicht aus → für jedes Foto (Druckende, Push mit Bild) 3 s an, danach wieder aus. War es an, ändert sich nichts.

## Neu in 1.3.78 (Sicherung außer Haus)

- Nach der täglichen Sicherung: verschlüsselt (AES-256-GCM, PBKDF2 310 000 Runden) in einen privaten Gist (Datei filamenthub-sicherung.json, alte Stände als Gist-Versionen).
- Optionen: github_token (fein abgestuft, Recht „Gists: Lesen und Schreiben“) und sicherung_passwort (mind. 8 Zeichen). Ohne Passwort ist die Sicherung nicht lesbar.
- Push, wenn die letzte älter als 2 Tage ist. Notfall: /api/hub/offsite/latest liefert die entschlüsselte neueste Sicherung.

## Neu in 1.3.77 (Fertiges Paket)

- Das Add-on kommt fertig aus GitHub (ghcr.io/tomden234/filamenthub-server-<art>), der Pi baut nichts mehr. „Automatisch aktualisieren“ im Add-on einschalten.
- Nach jedem Update: Push mit Neuerungen und Knöpfen „Geht“ / „Geht nicht“. „Geht nicht“ legt ein GitHub-Issue an (Option github_token).
- ESP-YAML kommt von der Webseite (erst nach grüner ESPHome-Prüfung).
- Erscheint ein Update, kann das Paket noch einige Minuten im Bau sein. Klappt die Installation nicht, kurz warten und erneut versuchen.

## Neu in 1.3.76 (Wochen-Selbsttest)

- Sonntag mit dem Wochenbericht: Drucker MQTT/FTP/Kamera, Home Assistant, Push-Weg, Spulen-Dokument, Sicherung (< 2 Tage), AMS-ESP, NFC-Leser 1-5 (Zustand „OK“), SHT31 Box und Trockner.
- Ergebnis im Bericht, eigene Push nur bei Fehlern. Drucker aus → seine Prüfungen übersprungen.

## Neu in 1.3.75 (Vergessen-Wächter)

- Steckdose an, 3 h kein Druck, keine AMS-Trocknung, kein „Aus nach …“ scharf → Push mit Knopf „Ausschalten“ (prüft vor dem Schalten, ob doch ein Druck läuft).

## Neu in 1.3.74 (Trocken-Erkennung)

- Absolute Feuchte im Trockner (SHT31, Magnus-Formel). Trockner ≥ 90 min warm (ab 40 °C) und Schwankung in der letzten Stunde < 0,3 g/m³ → Push „Spule trocken“.
- Neue Optionen: `trockner_steckdose` (z. B. `switch.trockner`) → Knopf „Trockner aus“; `trockner_auto_aus` → schaltet selbst ab. Danach bucht 1.321 „zuletzt getrocknet“ beim Abkühlen.

## Neu in 1.3.73 (Trockner-Wächter)

- Grenzen je Material (PLA/PVA 55, TPU 60, PETG/PP 70, HIPS 75, ABS/ASA 85, PA/PC 90 °C), unbekannt: Trocken-Temperatur der Spule + 10 °C.
- AMS: 10 Minuten über der Grenze der empfindlichsten Spule auf Platz 1-4 → Push mit „Trocknung stoppen“. Nach 30 Minuten mehr als 10 °C unter dem Ziel → „AMS heizt nicht“.
- Externer Trockner: Spule an Leser 5, Temperatur vom SHT31 im Trockner.

## Neu in 1.3.72 (Trocknen bucht sich selbst)

- AMS-Trocknung bis zum Ende (mind. 1 h): „Zuletzt getrocknet“ für die Spulen auf Platz 1-4.
- Externer Trockner: Spule an Leser 5 (Tag) bzw. Platz 5, Wärme vom SHT31 im Trockner (ab 40 °C). Nach Abkühlen (10 min unter 35 °C) und genug Wärme (75 % der Trockenzeit der Spule, 2-6 h) wird gebucht.

## Neu in 1.3.71 (Feuchte-Wächter)

- Feuchte von AMS und Lagerbox (SHT31 am AMS-ESP) wird alle 30 s geprüft, Grenzen aus der App (Standard 25 %, 5 Minuten, Wiederholung nach 12 h).
- AMS: Push mit Knopf „Jetzt trocknen“ (Temperatur/Dauer wie in der App: niedrigste Temperatur der eingelegten Spulen). Während eines Drucks startet die Trocknung danach.
- ESP, Box- und Trockner-Sensor: Push nach 30 Minuten ohne Werte, und wenn sie wieder da sind (nur Sensoren, die schon einmal Werte hatten).

## Neu in 1.2.0

- **Warteschlange**: In der App (Werkstatt → Warteschlange) Dateien vom Drucker vormerken. Taster „Platte frei“ am AMS-ESP (GPIO26 → GND) startet den nächsten Druck, 10 LEDs (GPIO14) zeigen, wie viele warten.
- **Drucker wartet**: Push mit Foto und den Knöpfen „Weiter“ / „Abbrechen“ (Home-Assistant-App auf dem Handy).
- **Strom** der Drucker-Steckdose je Woche, **Wochenbericht** sonntags 18 Uhr, **Ruhezeit** (lautlos), **Drucker antwortet nicht**.
- **ESP-Update übers WLAN**: Mit der ESPHome-App in Home Assistant legt der Server die neueste AMS-YAML in den ESPHome-Ordner (eigene Werte bleiben). Dann nur „Installieren“ → „Drahtlos“.
- **App direkt vom Pi**: Knopf „Web-UI öffnen“ oder `http://<IP des Pi>:3000`. Über http geht im Browser der verschlüsselte Geräte-Sync nicht; Handy- und Windows-App sind davon nicht betroffen.

## Neu in 1.3.10 (Abgleich & Schutz)

- Alle Dateien werden stromausfallsicher geschrieben (Zwischendatei + Umbenennen, Vorversion `.prev`). Ist eine Datei kaputt, liest der Server die Vorversion.
- Neue Dateien auf dem Drucker werden alle 20-60 s gesucht (vorher alle 3 s), beim Drucken alle 60 s. Taster bei leerer Warteschlange sucht sofort.
- Der Abzug läuft im Hintergrund und hält Warnungen und „Aus nach Druck“ nicht mehr auf. Die Verbindung zu Home Assistant baut sich nach Abbrüchen selbst neu auf.
- Kamera und Live-Stream nutzen einen 12-Stunden-Token statt des Schlüssels in der Adresse.

## Neu in 1.3.70 (aufgeräumt)

- Unbenutzte Exporte entfernt (Toter-Code-Finder knip), keine Änderung im Verhalten.

## Neu in 1.3.69 (FTP-Lücke geschlossen)

- FTP-Bibliothek `basic-ftp` auf 6.2.6 aktualisiert (bekannte Sicherheitslücke GHSA-5pm8-86m8-9m6r).

## Neu in 1.3.68 (Drucker wieder da)

- 1.3.67 zeigte das Druckerbild nicht (Home Assistant nimmt das selbst gesetzte `image.`-Element nicht als Bild). Jetzt: Text-Karte mit `<img>`, Adresse aus `sensor.filamenthub_drucker_bild` (Attribut `url`, Datei `drucker-<kennung>.svg`) – wechselt ohne Neuladen.
- Spulen-Schilder (Material, Restgewicht, Schrift Outfit) stecken im Bild. Temperatur-Karten als eigene Bild-Karte darunter (`temperaturen.svg`, ändert sich nie), Werte und −/+ darüber.
- Reste aus 1.3.67 (`image.filamenthub_drucker`, `sensor.filamenthub_bild_slot_*`) werden entfernt.

## Neu in 1.3.67 (kein Überlappen, Spulenwahl gilt, Bild ohne Neuladen)

- Druckerbild: Schrift und Knöpfe kleiner, passen auch bei schmalem Bild (ca. 250 px). Spulen-Schilder werden ins Bild gezeichnet und wachsen mit.
- Bild läuft über `image.filamenthub_drucker` (entity_picture zeigt auf `drucker-<kennung>.svg`): neue Spulenfarben erscheinen ohne Neuladen; der Dashboard-Aufbau bleibt gleich (kein „Dashboard geändert“-Hinweis mehr). Beschriftung über `sensor.filamenthub_bild_slot_N_material/_rest`.
- Spulenwahl im Dashboard gilt, solange am Drucker dieselbe Spule steckt (auch wenn Material/Farbe laut Drucker nicht passen).

## Neu in 1.3.66 (besser lesbar)

- Theme: dunkle Glaskarten mit weißer Schrift auf dem Pastell-Hintergrund, Eingabefelder leicht hell mit türkiser Linie, Menüs deckend, Schrift Outfit im ganzen Dashboard.

## Neu in 1.3.65 (neue Beschriftung, Temperatur-Karten, Schrift Outfit)

- Spulen-Schilder: dunkle Pille mit Rand in Spulenfarbe, Material groß, Restgewicht darunter.
- Temperaturen als Karten unter dem Drucker (Düse 1 und 2 nebeneinander, Bett darunter): oben Name und Soll, groß der Ist-Wert, runde −/+ (5 °C).
- Schrift „Outfit“ (SIL OFL 1.1): der Server legt sie in `www/filamenthub-dashboard` ab und meldet `fonts.css` als Dashboard-Ressource an. Ohne Ressourcen-Verwaltung (Dashboards im YAML-Modus) erscheint die Ersatzschrift.

## Neu in 1.3.64 (Spulen wie vorher, Schimmer in Spulenfarbe)

- Druckerbild wieder auf Basis des Originalfotos: Spulen sehen aus wie vorher, der Schimmer im Gehäuse unter jeder Spule hat ihre echte Farbe (leicht).
- Leere Plätze: Spalte aus dem Foto des leeren AMS, Zeile für Zeile auf das Druckerfoto ausgerichtet, Deckel im Licht der Stege, Einzug original ohne Farbschimmer, weicher Rand.

## Neu in 1.3.63 (leeres AMS sauber)

- Grundbild ist jetzt der Drucker mit leerem AMS; gefüllte Plätze bekommen ihre Spule eingezeichnet, leere bleiben sauber leer (Deckel und Einzug wie im Foto).

## Neu in 1.3.62 (Temperaturen im Druckerbild, leere Plätze echt leer)

- Düse 1, Düse 2 und Bett stehen direkt im Druckerbild: Name und Ist-Wert, darunter Soll; −/+ ändern das Soll in 5-°C-Schritten, der Server schickt es an den Drucker. Die Soll-Kacheln in der Mitte sind weg.
- Leere AMS-Plätze zeigen ein Foto vom leeren AMS statt einer grauen Fläche.

## Neu in 1.3.61 (echte Spulenfarben aus dem Shop)

- Der Server lädt je Spule das Produktbild aus dem Shop-Link (alle 4 Minuten eine Spule), liest es mit ffmpeg und misst die echte Filamentfarbe (Hintergrund und Pappspule fallen weg). Gemerkt in `spool-colors.json`; neue oder geänderte Spulen werden neu gemessen, Fehlschläge nach 3 Tagen.
- Druckerbild, Spulen-Sensoren und Testbild nutzen die gemessene Farbe; die Daten in der App bleiben unverändert. Passt das Bild nicht (ganz andere Farbe), bleibt die App-Farbe.
- Spulen im Bild matt dargestellt. Protokoll: `[Hub] Spulenfarbe …`.

## Neu in 1.3.60 (Spule wählen = am Drucker gesetzt, Farben aus der App)

- Spule im Dashboard gewählt → Material, Farbe und Düsenbereich gehen auch an den AMS-Platz am Drucker (wie in der App).
- Druckerbild und Spulen-Sensoren nehmen Name, Material und Farbe der Spule aus der App, nicht mehr die Werte vom Drucker. Leere Plätze hellgrau.
- Testbild aller Spulen aus der App: `http://<Home-Assistant>/local/filamenthub-dashboard/spulen-test.svg`.
- Auswahlfelder hell, aufgeklappte Liste deckend. Lagerbox/Trockner: 4 einzelne Anzeigen, Feuchte blau (💧), Temperatur gelb bis rot (🔥).

## Neu in 1.3.59 (Dashboard kompakt, Spulen automatisch erkannt)

- Spulen-Auswahl zeigt immer, was gerade im Drucker steckt: eigene Zuordnung, solange Material und Farbe passen, sonst die passendste Spule aus dem Lager.
- Layout: links Drucker (halb breit) neben der Warteschlange, darunter die Spulen; Mitte kleine Anzeigen mit Icons, Soll-Werte als Kacheln mit −/+, Live-Werte als Kacheln; rechts Lagerbox und Trockner.
- Hintergrund „Pastell-Abend“.
- Dashboard-Takt läuft unabhängig von anderen Takt-Fehlern.

## Neu in 1.3.58 (Dashboard: Spulen aus dem Lager, AMS in echten Farben, Düse 2)

- Je Platz (`input_select.filamenthub_slot_N_spule`, N = 1-4 AMS, 5 extern) eine Auswahl aller Spulen mit Rest, z. B. „Sunlu PLA+ Gelb · 640 g (64 %)“. Die Wahl ändert die Platz-Belegung; die Apps übernehmen sie beim nächsten Abgleich. Die alten Felder Material/Farbe/Gramm werden gelöscht.
- **Düse 2**: Sensor `sensor.filamenthub_duese_2` und Eingabe `input_number.filamenthub_duese_2_soll` (0-300 °C).
- Druckerbild von vorn, die vier AMS-Spulen in den Farben, die der Drucker meldet (`drucker.svg`, wird bei neuer Spule neu gebaut).
- Warteschlange wie in der App: Vorschaubild, Spulen je Filament, Zeit, Warnungen in Rot.
- Temperatur-Anzeigen klein (¼), Feuchte groß, Verläufe in kräftigen Farben, heller Hintergrund, Eingabefelder dunkles Teal.

## Neu in 1.3.57 (Dashboard in 3 Spalten, Eingabefelder)

Dashboard „Drucker“: links Drucker (ohne Kasten), Mitte Drucker-Werte, rechts AMS und Trockner. Der Server legt Eingabefelder an (`input_number.filamenthub_duese_soll`, `_bett_soll`, `input_select.filamenthub_slot_N_material` / `_farbe`, `input_number.filamenthub_slot_N_gramm`, N = 1-4 AMS, 5 extern):
- **Soll-Temperatur** geht als G-Code an den Drucker (Düse 0-300 °C, Bett 0-120 °C).
- **Material, Farbe, Gramm** ändern die Spule, die dem Platz in der App zugeordnet ist (ohne Zuordnung wird die Eingabe verworfen).
Glas-Theme: `themes/filamenthub.yaml`. In `configuration.yaml` muss stehen: `frontend:` / `themes: !include_dir_merge_named themes`, danach Home Assistant neu starten. Ohne das sind die Karten dunkel statt durchsichtig, sonst funktioniert alles.

## Neu in 1.3.56 (X2D-Foto im Dashboard)

Das Dashboard zeigt ein X2D-Foto (liegt als `www/filamenthub-dashboard/x2d.webp`) mit den Werten daneben. Aufbau: Foto links, Anzeigen rechts (Abschnitte-Ansicht). Foto tauschen: neues Bild in `server/hub/dashboardPhoto.ts` einsetzen, `ART_VERSION` hochzählen.

## Neu in 1.3.55 (Dashboard „Drucker“)

Der Server setzt Druckerwerte als Sensoren in Home Assistant (`sensor.filamenthub_status`, `_fortschritt`, `_duese`, `_bett`, `_restzeit`, `_schicht`, `_auftrag`, `_fach`, `_spule_*`, `_strom_woche`, `_kosten_woche`) und legt das Dashboard **Drucker** selbst an (Seitenleiste, Adresse `/filamenthub-drucker`). Grafiken liegen in `www/filamenthub-dashboard/`. Nötig: `homeassistant_config:rw` (ist gesetzt). Neu anlegen: App → Verbindungen → FilamentHub Server → **Dashboard anlegen**. Die Sensoren sind nach einem Neustart von Home Assistant kurz weg und kommen innerhalb von 30 Sekunden wieder.

## Neu in 1.3.35

- **FTP zum Drucker**: eine Sitzung wird offen gehalten und wiederverwendet (gegen „421 too many connections“), nach 421 fünf Minuten Pause.
- **Amazon-Links aus Bestellungen**: jede Bestell-Mail füllt `product-catalog.json`. Alte Bestellungen in Gmail markieren und „Als Anhang weiterleiten“ an das Bestell-Postfach: der Pi lernt nur die Links, legt keine Spulen an.

## Neu in 1.3.34

- **Spulen-Links**: Produktseite aus der offiziellen Produktliste (Bambu, Prusa, SUNLU, 3DJake) bzw. aus den Amazon-Treffern (Gemini wählt). Der Pi füllt alle 3 Minuten den Link der nächsten Spule ohne Produktseite (`product-links.json`, Ordner `sitemaps`).
- **Test-Mail auslesen**: prüft Gemini mit einer Beispiel-Mail, ohne etwas zu speichern.

## Neu in 1.3.33

- **Zustellung bestätigen**: Die Zustell-Mail setzt die bestellte Spule nur auf „zugestellt“ (Push „Filament zugestellt“). Erst dein „Habe ich“ in der App macht sie zur normalen Spule. Die Mail liefert außerdem das Lieferdatum („Kommt in 3 Tagen“).

## Neu in 1.3.32

- **Produktseite finden**: Der Pi sucht per Gemini mit Google-Suche die genaue Produktseite einer Spule (Klick auf den Shop-Link in der App) und merkt sie sich (`product-links.json`). Braucht `gemini_api_key`.
- **Bestell-Mails sicherer**: Teillieferungen, Wiederholung bei Fehlern, Spam-Ordner wird mitgelesen, Zustellung legt keine neuen Spulen an.

## Neu in 1.3.31

- **Bestell-Mails**: Neue Optionen `mail_user` und `mail_password` (App-Passwort eines zweiten Gmail-Kontos, IMAP aktiv). Der Pi prüft alle 10 Minuten, Gemini liest Bestellung, Versand und Zustellung. Spulen erscheinen als „Bestellt/Unterwegs“, bei Zustellung kommt ein Push. Das Passwort bleibt nur auf dem Pi. Anleitung im Testmenü, Punkt 7. Braucht `gemini_api_key`.

## Neu in 1.3.30

- **Datei-Archiv**: Der Pi sichert jede Datei aus der Warteschlange (Ordner `archive`, bis 20 GB / 2000 Dateien). In der App: Menü „Datei-Archiv“ mit Suche, Sortierung, Material-Filter, Stern (wird nie gelöscht) und „Nochmal drucken“.

## Neu in 1.3.29

- **Smart sortieren**: Vorschlag für eine bessere Reihenfolge der Warteschlange (Platte, Düse, Spulen, Nacht), erst Vorschau, dann Übernehmen.
- **Temperatur-Wächter**: Warnung, wenn die Düsentemperatur der Datei stark von der Spule abweicht.

## Neu in 1.3.28

- **Alle Platten**: Dateien mit mehreren Platten kommen als einzelne Karten hintereinander in die Warteschlange, mit Projekt-Kopf (Fortschritt, Zeit, Gramm).

## Neu in 1.3.27

- **Druckzeit**: Der Pi liest die Slicer-Zeit aus der Druckdatei und lernt aus echten Drucken einen Faktor je Material (Datei `hub-timefactor.json`). Die Warteschlange zeigt Slicer-Zeit, berechnete Zeit und die Fertig-Uhrzeit.

## Neu in 1.3.26

- **Platte & Düse im Check**: Der Pi liest aus der Druckdatei (.3mf), welche Druckplatte und Düse sie braucht. Die Warteschlange zeigt beides und warnt gelb, wenn es sich zum Druck davor ändert.

## Neu in 1.3.25

- **Foto-Erkennung schneller**: Der Pi nimmt zuerst das schnelle Modell Gemini 3.5 Flash-Lite. Im Protokoll steht jetzt, wie lange die Erkennung gedauert hat („Gemini … fertig in … s“).

## Neu in 1.3.24

- **Foto-Erkennung**: Ist das Gemini-Guthaben leer (Code 402), sagt die App das jetzt klar und nennt den Ausweg.

## Neu in 1.3.23

- **Foto-Erkennung repariert**: Der Pi nutzte einen Modellnamen, den es nicht gibt („gemini-3.8-flash“). Jetzt der Alias `gemini-flash-latest` mit festen Reserve-Modellen. Fehler zeigen klare Texte (Schlüssel, Limit, Modell) und stehen im Pi-Protokoll.

## Neu in 1.3.22

- **Test „Druck fertig“ mit altem Druck**: Testmenü → 6. Push-Meldungen. Der Pi nimmt den neuesten fertigen Druck mit Foto und schickt den echten Push mit Bild, Dauer, Abzug und Strom.

## Neu in 1.3.21

- **Wochen wählen**: In der App (Einstellungen → FilamentHub Server → Wochen) stehen die letzten 8 Wochen. Jede lässt sich aufklappen und als Push senden. Frühere Wochen zählen Montag bis Sonntag.
- **Berichtsseite im App-Design** (Sand). „App öffnen“ geht zur Webseite der App; eigene Adresse mit der Umgebungsvariable `APP_URL` möglich.

## Neu in 1.3.20

- **Push-Test**: In der App Testmenü → 6. Push-Meldungen: jede Art Push einzeln oder alle nacheinander auslösen, dazu „Wochenbericht jetzt senden“.
- **Wochenbericht als Seite**: Die letzten 12 Berichte bleiben gespeichert. Tipp auf den Wochenbericht-Push öffnet `http://<Pi>:3000/bericht/<Kennung>` (Heimnetz oder Tailscale). Der Pi lernt seine Adresse aus den Anfragen der App, einmal die App mit dem Pi verbinden genügt.

## Neu in 1.3.19

- **Reicht die Spule?** Beim Einreihen und Starten rechnet der Pi die Gramm je Filament (plus 10 %) gegen den Rest der Spule im gewählten Platz. Reicht es nicht, steht es im Push und die Karte in der App wird rot (mit Ersatz-Spulen und Nachkauf-Link). Gestartet wird trotzdem, der Rest ist eine Schätzung.
- **Push-Knöpfe** bei „In der Warteschlange“: Jetzt starten, Löschen, Später. „Jetzt starten“ geht nur bei freier Platte, sonst fragt der Pi erst „Platte ist frei, starten“.
- **gemini_api_key** (optional): schaltet die Foto-Erkennung der Spulen am Pi ein.

## Neu in 1.3.18

- Warteschlangen-LEDs gleichen sich selbst ab (ESP wieder online, falsche Zahl, jede Minute).
- Neue Dateien: ohne Neues alle 20-30 s suchen; USB-Stick und weitere Ordner werden durchsucht; gleicher Dateiname wird als neu erkannt.

## Neu in 1.3.17

- Dauer-Protokoll `print-stats.json` (Start, Ende, Gramm je Material je Druck, bis 100.000 Drucke) für die Fun Facts. Übernimmt beim ersten Start die schon aufgezeichneten Drucke.

## Neu in 1.3.16

- Taster 4 s halten = laufenden Druck abbrechen (braucht ESP-Firmware 1.241, Entität „Taster lang“).
- Doppelklick wird auch ohne neue Firmware erkannt (zwei schnelle „Platte frei“).

## Neu in 1.3.15

- LED-Test in der App: Pi ignoriert die Warteschlange währenddessen und stellt sie danach wieder her.
- Taster doppelt: in den Einstellungen wählbar (pausieren/fortsetzen, ersten Eintrag nach hinten, nichts).

## Neu in 1.3.14

- Warteschlange: Wahl des AMS-Platzes pro Filament (Standard automatisch), gilt beim Start.
- Bei offener Warteschlange in der App sucht der Pi alle 8 s nach neuen Dateien.

## Neu in 1.3.13 (Fehler-Blinken aus)

- Die Status-LED hörte nach einem Fehler nicht auf zu blinken, weil der Drucker nach dem Quittieren am Display weiter „FAILED“ meldet. Der Pi schaltet den Streifen jetzt aus, sobald der Fehlercode weg ist (Prüfung alle 30 s). Blinkt er ohne gemeldeten Fehlercode, bleibt er an.

## Neu in 1.3.12 (npoint aus)

- npoint.io ist abgeschaltet: der Server nimmt keine npoint-Adresse mehr an, die Route `/api/cloud-proxy` ist weg. Im Add-on ist die Datei auf dem Pi (`pi:`) jetzt der **Standard-Speicher**, auch wenn noch keine App umgestellt hat oder in der Einstellung noch eine alte npoint-Adresse steht.
- Die Apps ziehen ihre Daten beim Start von selbst von npoint auf den Pi um (wenn Adresse und Schlüssel des Pi eingetragen sind).

## Neu in 1.3.9 (Pi speichert)

- Die Spulen-Daten liegen als Datei im Speicher der App (`/data/filamenthub-doc.json`, Vorversion als `.prev`). In der App unter Cloud & Geräte-Sync auf **Auf Pi-Server umstellen** tippen: vorhandene Daten werden einmal übernommen. Danach braucht es weder npoint noch Gist noch Token. Abzug, Warteschlange und Tages-Sicherung nutzen dieselbe Datei.

## Neu in 1.3.8 (Sicherheit)

- Der Server antwortet nur noch mit Schlüssel. Ohne eigenen API-Key erzeugt er beim ersten Start einen (steht im Protokoll, bleibt bei Updates gleich).

## Hinweise

- Drucker, die der Server nicht sieht (Pi aus oder Drucker aus), kann er nicht aufzeichnen. Ein Druck, der beim Neustart des Pi läuft, wird danach weiter verfolgt.
- Die Daten liegen im Speicher der App (`/data`) und gehen bei einem Update nicht verloren.
- Auf GitHub Pages (https) kann die Seite keinen http-Server ansprechen. Nutze dort die Handy- oder Windows-App.
