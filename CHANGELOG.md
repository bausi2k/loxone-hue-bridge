# Changelog

Alle nennenswerten Änderungen an diesem Projekt werden in dieser Datei dokumentiert.

Das Format basiert auf [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
und dieses Projekt hält sich an [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://www.buymeacoffee.com/bausi2k)

## [2.4.3] - 2026-08-06
**Dauerbetrieb & Einstellungen** – Behebt einen Fehler, der die Bridge bis zum Neustart komplett lahmlegen konnte, dazu drei Stellen, an denen ein einmaliger Fehler zu einem dauerhaften Ausfall führte, und vier Fehler beim Speichern der Einstellungen.

### 🐞 Bugfixes
- **Queue-Blockade bei hängender Bridge (kritisch):** Keiner der Befehls-Requests hatte ein Timeout gesetzt – der Standardwert von axios ist „unbegrenzt". Antwortete die Hue Bridge nicht mehr (Verbindung steht, aber keine Antwort), blieb die Warteschlange dauerhaft stehen und die Bridge nahm zwar weiter Loxone-Befehle an, leitete aber nichts mehr weiter. Nur ein Neustart half. Mit einer Test-Bridge nachgewiesen: nach der Wiederherstellung kamen vorher **0 von 5** Befehlen an, jetzt erholt sich die Verbindung selbstständig. Zusätzlich abgesichert durch eine Zeitgrenze in der Warteschlange, eine Obergrenze für gepufferte Befehle und das Verfallen verwaister Gerätesperren.
- **MQTT-Passwort wurde bei jedem Speichern gelöscht:** Da das Passwort aus Sicherheitsgründen nicht an das Dashboard ausgeliefert wird, blieb das Eingabefeld leer und überschrieb beim Speichern den hinterlegten Wert. Jeder Klick auf „Speichern" im System-Tab hat damit die MQTT-Anmeldung zerstört. Ein leeres Feld bedeutet jetzt „unverändert lassen", und das Dashboard zeigt an, ob ein Passwort hinterlegt ist.
- **Drosselung wirkte nach einem Neustart nicht:** Der eingestellte Wert wurde nur zur Laufzeit übernommen; nach jedem Neustart galten wieder die Standardwerte. Außerdem sank das Intervall für Gruppenbefehle beim ersten Speichern von 1100 ms auf 100 ms und provozierte genau die Überlastung (HTTP 429), die die Drosselung verhindern soll. Untergrenze für Gruppen ist jetzt 1000 ms.
- **MQTT gab nach einem Anmeldefehler endgültig auf:** Ein vorübergehend nicht erreichbarer oder neu startender Broker legte MQTT bis zum Neustart still. Jetzt wird mit wachsendem Abstand erneut versucht (ab 30 s, maximal 15 Minuten).
- **UDP-Verbindung zu Loxone wurde nach einem Fehler nie wiederhergestellt:** Schloss das Betriebssystem den Socket, schlugen alle weiteren Statusmeldungen still fehl – Loxone bekam dauerhaft keine Werte mehr, ohne dass es auffiel. Der Socket wird jetzt neu aufgebaut.
- **Falscher Aus-Status an Loxone:** Ein Befehl ohne Schaltanteil meldete fälschlich „Licht aus". Bislang latent, wäre mit den geplanten relativen Dimmbefehlen real geworden.
- **Farbton bei gesättigten Farben:** Blau wurde als `#38b1ff`, Magenta als `#ff51ff` an Loxone und MQTT gemeldet. Ursache war ein Abschneiden statt eines gemeinsamen Herunterskalierens der Farbkanäle.
- **Debug-Modus wirkte erst nach einem Neustart**, wenn er über den Einrichtungsassistenten gesetzt wurde.
- **Leere Zahlenfelder** in den Einstellungen landeten als `null` in der Konfiguration und machten UDP-Versand bzw. Drosselung unbrauchbar, ohne dass die Einrichtung als unvollständig galt.
- **Unbekannte API-Pfade** lieferten HTTP 200 und tauchten als vermeintlich neuer Loxone-Befehl im Dashboard auf; jetzt HTTP 404.
- **Absturzursachen werden vollständig protokolliert:** Unbehandelte Promise-Fehler beendeten den Prozess bisher, ohne den Grund zu hinterlassen – nach dem automatischen Neustart war die Ursache verloren.

### 🧪 Tests
- Abdeckung von 34 auf 60 Tests erweitert. Neu abgedeckt: Verhalten der Warteschlange bei hängender Bridge, Selbstheilung von MQTT und UDP, Farbumrechnung gesättigter Farben und das Speichern der Einstellungen. Eine Quelltext-Prüfung verhindert, dass künftig wieder Anfragen ohne Timeout entstehen.

## [2.4.2] - 2026-08-04
**Stabilität & Datenhygiene** – Behebt vier Fehler, die im laufenden Betrieb auftreten: fehlerhafte Farbwerte Richtung Loxone, ein funktionsloser Löschen-Button, eine unbegrenzt wachsende Logdatenbank und versehentlich versionierte Laufzeitdaten.

### 🐞 Bugfixes
- **Division durch Null in der Farbumrechnung:** `xyToHex()` teilte durch die Leuchtdichte-Komponente `y`, ohne den Nenner zu prüfen. Bei `y=0` – von der Hue-Bridge bei nicht initialisierten Leuchten gemeldet – entstand `#NaNNaNNaN`, das per UDP an Loxone und an MQTT ausgeliefert wurde. Zusätzlich fängt `componentToHex()` nicht-endliche Werte ab. Gültige Farben bleiben unverändert.
- **Löschen-Button und Modal-Checkboxen funktionslos bei Apostroph im Namen:** Die Inline-Handler bauten den Loxone-Namen als JavaScript-String-Literal in ein HTML-Attribut. Ein Name wie `anna's lampe` erzeugte ungültiges JavaScript (`SyntaxError: missing ) after argument list`). Die Handler werden jetzt programmatisch gebunden, der Name ist nie mehr Teil eines Attributs.
- **HTML-Escaping im gesamten Dashboard:** Gerätenamen, Log-Meldungen und erkannte Befehle wurden ungeescaped gerendert. Namen mit spitzen Klammern verloren Teile (`Küche & Esszimmer "Decke" <1>` → `<1>` verschwand), und Fremdinhalt aus eingehenden Requests landete ausführbar im DOM. Betrifft Log-Konsole, Mapping-Liste, Details-Modal, Diagnose-Tab, Einstellungen und Export-Liste.
- **Log-Rotation:** `logs.db` wuchs unbegrenzt – es gab weder ein `DELETE` noch einen WAL-Checkpoint. Die Datenbank wird jetzt auf 50.000 Einträge begrenzt (Gegenstück zum bestehenden `MAX_RAM_LOGS` des RAM-Modus), gebündelt alle 1.000 Schreibvorgänge und einmalig beim Start. `PRAGMA wal_checkpoint(TRUNCATE)` gibt den Speicher tatsächlich frei.
- **XML-Export:** Namen werden beim Herunterladen der Loxone-Vorlagen einzeln URL-kodiert, ein `&` im Namen schnitt die Anfrage vorher ab.

### 🔄 Verbesserungen
- **Repository-Hygiene:** `data/logs.db` war versioniert und wurde aus dem Tracking genommen; ein wirkungsloses `.gitignore`-Fragment entfernt.
- **Schlankeres Docker-Image:** `.dockerignore` schließt jetzt `data/`, lokale Konfigurationsdateien, Doku-Assets und Testdateien aus. Das Image enthält nur noch Anwendungscode und Abhängigkeiten.
- **Längenbegrenzung für erkannte Befehle:** Namen aus dem Request-Pfad werden auf 64 Zeichen gekürzt.
- **Datenbank-Index** auf die Spalte `category`, nach der die Logabfrage filtert.

### 🧪 Tests
- Abdeckung von 18 auf 34 Tests erweitert: Randwerte der Farbumrechnung, Escaping-Verhalten (geprüft gegen den echten Quelltext), Log-Rotation und die Längenbegrenzung. Ein Regressionstest verhindert, dass künftig wieder Inline-Handler mit interpolierten Werten entstehen.

## [2.4.1] - 2026-06-23
### 🐞 Bugfixes
- **GitHub Actions:** Token-Konfiguration im Docker-Publish-Workflow korrigiert.

## [2.4.0] - 2026-06-23
### 🔄 Verbesserungen
- **Privates Deployment:** `docker-compose.yml` baut das Image jetzt lokal (`build: .`), statt es von `ghcr.io` zu laden. Der Publish-Workflow wurde entsprechend angepasst.
- **Abhängigkeiten:** `package-lock.json` aktualisiert und repariert.

## [2.3.2] - 2026-06-19
### 🌟 New Features
- **CGDESIGN Design-System:** Integration des CGDESIGN VARIABLE EXPORT (v1.0.0) in `public/style.css`. Neue Farb- und Radius-Token (`--accent-hue`, `--glass-blur`, hierarchische Radien), Glassmorphismus für Karten und Listenelemente, weiche Übergänge bei Buttons und Tabs sowie ein Verlaufsschema für den Titel. Bestehende CSS-Variablen wurden auf die neuen Token gemappt, sodass die Darstellung kompatibel bleibt.
- **Kontrast-Korrektur:** Hover-Effekte auf Listen- und Tabellenelementen funktionieren jetzt in hellem und dunklem Modus korrekt.

## [2.3.1] - 2026-05-13
### 🐞 Bugfixes
- **XML-Export:** Sonderzeichen in Geräte- und Loxone-Namen werden beim Export der Loxone-Vorlagen korrekt maskiert (`&`, `<`, `>`, `"`, `'`). Vorher erzeugten Namen wie `Wohnzimmer & Esszimmer` ungültiges XML, das der Miniserver nicht einlesen konnte.
- **README:** Fehlerhafter `git clone`-Befehl korrigiert.

## [2.3.0] - 2026-05-04
### 🌟 New Features
- **Hue Effekte & Alert:** Lampen können jetzt per einfachem Befehl in spezielle Effektmodi versetzt werden – vollständig rückwärtskompatibel zu allen bestehenden Steuerungen.
  - `/{name}/alert` → Einmaliges Breathe-Blinken (ideal für Alarmierung, Türklingel-Bestätigung, etc.)
  - `/{name}/candle` → Kerzenflackern 🕯️ (persistent bis zum Stoppen)
  - `/{name}/fire` → Feuereffekt 🔥 (persistent, nur neuere Lampen)
  - `/{name}/prism` → Regenbogen-Farbwechsel 🌈 (persistent, nur Farblampen)
  - `/{name}/sparkle`, `/opal`, `/glisten` → weitere atmosphärische Effekte
  - `/{name}/noeffect` → Aktiven Effekt stoppen
  - `/{name}/sunrise/30` → 30-Sekunden Sonnenaufgang-Simulation 🌅 (oder beliebige Dauer in Sekunden)
- **Erweiterter Diagnose-Tab:** Der Diagnose-Tab zeigt jetzt drei Abschnitte:
  1. 📋 Geräte & Batterien (bekannt)
  2. 🌐 Bridge & Zigbee Netzwerk – Verbindungsstatus (`connected` / `connectivity_issue`) jedes einzelnen Zigbee-Geräts, Bridge-ID und Zeitzone
  3. 🎭 Lampen-Fähigkeiten – Übersichtstabelle zeigt pro Lampe, ob Dimmen ✅, Farbe ✅ und Weißton ✅ unterstützt werden, sowie alle verfügbaren Effekte.
- **Nativer "Alles" Befehl:** Der Befehl `/all` (bzw. `/alles`) nutzt nun die native `bridge_home` Ressource der Hue Bridge, um das gesamte Zuhause nahezu verzögerungsfrei zu schalten. Im UI ist die Option „🏠 Alle Lichter (bridge_home)" jetzt im Dropdown wählbar.
- **Batterie-Warnsystem:** Geräte mit einem Batteriestand von ≤ 10 % werden im Dashboard optisch hervorgehoben (rotes Badge + Leer-Symbol 🪫).
- **Automatisierte Tests:** Einführung einer robusten Test-Infrastruktur basierend auf dem nativen Node.js Test-Runner (`node:test`) mit 16 Tests und > 85 % Abdeckung der Kernmodule.

### 🔄 Verbesserungen & Refactoring
- **Backend-Modularisierung:** Komplette Neustrukturierung der `server.js`. Die Logik wurde in saubere Module im Ordner `lib/` (`logger`, `config`, `loxone`, `mqtt`, `hue`, `routes`) ausgelagert, was die Wartbarkeit und Stabilität massiv erhöht.
- **Frontend-Cleanup:** Trennung von HTML, CSS und JavaScript. Die `index.html` wurde bereinigt, Styles wanderten in `style.css` und die Logik in `app.js`.
- **Smarte Listen:** Die Liste der „Neu erkannten Befehle" filtert nun automatisch Duplikate.
- **Robustheit:** Zuvor leere `catch`-Blöcke loggen nun detaillierte Fehlermeldungen.

## [2.2.0] - 2026-02-26
### 🌟 New Features
- **Dynamics ignorieren:** Es kann nun pro Lampe/Gruppe individuell eingestellt werden, ob weiche Übergänge (Transition/Dynamics) gesendet werden sollen. Für reine An/Aus-Schalter (ohne Dimmfunktion) wird dies automatisch erzwungen.
- **Interaktive UI & Detail-Ansicht:** Die Gerätekarten im Dashboard sind nun klickbar. Ein Modal zeigt Live-Status, technische Details und erlaubt individuelle Geräte-Einstellungen (Loxone Sync & Dynamics ignorieren).
- **Slider für Timings:** Übergangszeit und Drosselung lassen sich im System-Tab nun intuitiv per Schieberegler (0-1000ms) einstellen.

### 🔄 Verbesserungen
- **Smarte Sortierung:** Schalter und Diagnose-Einträge werden nun ebenfalls priorisiert nach niedrigstem Batteriestand sortiert.
- **Diagnose-Icons:** Optische Aufwertung und bessere Übersichtlichkeit des Diagnose-Tabs durch Geräte-Typ-Icons.

## [2.1.2] - 2026-02-17
### 🐛 Bugfixes
- **UI Settings:** Fehlende Eingabefelder für "Übergangszeit" und "Drosselung" im System-Tab hinzugefügt.
- **Diagnose Tab:** Fehler behoben, der das Laden der Diagnose-Tabelle verhinderte (`loadDiagnostics is not defined`).
- **Server Stabilität:** Kritischen Fehler beim Start behoben (Hoisting Problem bei `REQUEST_QUEUES`).
- **Sonoff / On-Off Fix:** Reine Schaltaktoren erhalten keine `dynamics` Parameter mehr (behebt Probleme mit Sonoff ZBMINIR2).
- **Sensor Sortierung:** Sensoren werden nun nach Batterie-Status (leer zuerst) und Aktivität sortiert.

## [2.1.1] - 2026-02-16
### 🐛 Bugfixes
- **Sonoff / On-Off Fix:** Reine Schaltaktoren (ohne Dimm-Funktion) erhalten nun keine `dynamics` Parameter mehr. Das behebt Probleme mit Geräten wie dem Sonoff ZBMINIR2, die sich sonst nicht ausschalten ließen.
- **Queue Timing:** Die Einstellung `throttleTime` (Drosselung) gilt nun auch korrekt für Gruppen- und Zonen-Befehle (war vorher fest auf 1100ms).
- **Sensor Sortierung:** Im Dashboard werden Sensoren nun nach Wichtigkeit sortiert (Leere Batterie -> Aktiv -> Name).

## [2.1.0] - 2026-01-29
### 🌟 New Features
- **SD-Card Mode:** Neue Option in den Systemeinstellungen, um das Schreiben von Logs auf die Festplatte zu deaktivieren (schont SD-Karten auf Raspberry Pi). Logs werden dann nur im RAM gehalten.
- **Robustheit:** Neuer Crash-Monitor fängt kritische Fehler ab und verhindert, dass der Server bei kleineren Problemen komplett abstürzt.

### 🐛 Bugfixes
- **MQTT:** Fix für Abstürze bei leeren Benutzer/Passwort-Feldern und Endlos-Schleifen bei Authentifizierungsfehlern.
- **Datenbank:** Server startet nun auch, wenn die `logs.db` gesperrt oder beschädigt ist (Fallback auf RAM-Modus).

## [2.0.0] - 2026-01-29
### 💥 Major Changes
- **Core Engine Upgrade:** Umstellung auf **Node.js 24 LTS**.
- **Native SQLite Integration:** Logs werden nun persistent in einer lokalen SQLite-Datenbank (`data/logs.db`) gespeichert statt nur im Arbeitsspeicher.
    - *Vorteil:* Logs überleben Neustarts und ermöglichen eine Historie von Millionen Einträgen ohne RAM-Verbrauch.
    - *Performance:* Nutzung des neuen `node:sqlite` Moduls für maximale Geschwindigkeit ohne externe C++ Abhängigkeiten.
- **UI Overhaul:** Komplettes Redesign des Dashboards.
    - Auslagerung der Styles in `style.css`.
    - Neue **Filter-Leiste** für Logs (Kategorien + Volltextsuche).
    - Verbesserte **Sensor-Gruppierung** (Kontakte, Bewegung, Sonstige).
    - **Backup & Restore:** Vollständige Sicherung und Wiederherstellung der Konfiguration direkt über das Web-Interface.

### 🐛 Bugfixes
- **Grouped Lights:** Fix für fehlenden Status von Lichtgruppen (Zimmer/Zonen) nach Neustart. Der Endpunkt `grouped_light` wird nun beim Start synchronisiert.
- **Zero-Value Display:** Korrektur eines Fehlers im Frontend, bei dem Werte von `0` (z.B. Licht Aus, Keine Bewegung) fälschlicherweise als "leer" interpretiert und ausgeblendet wurden.
- **Log Formatting:** Fix für Zeilenumbrüche in der Log-Ansicht für bessere Lesbarkeit.

---
---

## [1.8.0] - 2026-01-21

### 🚀 Features
- **MQTT Support:** Die Bridge kann nun Statusänderungen (Licht, Sensoren, Taster) parallel an einen MQTT Broker senden.
    - Konfiguration im Tab "System" (Broker, Port, User, Passwort).
    - Topic-Struktur: `loxhue/<typ>/<name>/<attribut>` (z.B. `loxhue/light/kueche/bri`).
    - Ideal für die Integration in Home Assistant, ioBroker oder Node-RED.
- **Erweitertes Dashboard:**
    - **Licht-Gruppierung:** Im Tab "Lichter" werden Lampen nun übersichtlich in "Eingeschaltet" 💡 und "Ausgeschaltet" 🌑 unterteilt.
    - **Live-Info Modal:** Das Info-Icon (ℹ️) zeigt nun Live-Werte der Lampe an (Helligkeit %, Kelvin, Hex-Code), was das Debuggen massiv erleichtert.

### 🛠 Verbesserungen
- **Stabilität:** Beinhaltet alle Fixes aus v1.7.x (Watchdog gegen Verbindungsabbrüche, Queue-Drosselung).
- **UI:** Neuer Toggle-Switch im System-Tab, um MQTT global an- oder abzuschalten.

---

## [1.7.3] - 2026-01-20

### 🛡️ Stabilität
- **EventStream Watchdog:** Behebt das Problem ("Zombie Connection"), bei dem nach längerer Laufzeit (10-14 Tage) keine Sensor-Updates mehr empfangen wurden.
    - Der neue Watchdog prüft auf eingehende Daten (inkl. Hue Heartbeats).
    - Bei Stille (>60s) wird die Verbindung proaktiv getrennt und neu aufgebaut.

### 🚀 Features
- **Configurable Throttling:** Die Drosselung der Befehls-Queue ist nun im System-Tab einstellbar (0ms - 1000ms).
    - Ermöglicht Power-Usern, die Reaktionsgeschwindigkeit zu erhöhen oder bei Verbindungsproblemen (Error 429) konservativer zu agieren.
    - Standardwert: 100ms.

---

## [1.7.2] - 2025-12-15

### 🐛 Bugfixes
- **Button Event Cache Fix:** Behebt ein Problem, bei dem wiederholte Tastendrücke (z.B. zweimaliges Drücken für "An" und "Aus") von der internen Cache-Logik verschluckt wurden, da sich der Status-Text (z.B. `short_release`) nicht geändert hatte.
    - **Jetzt:** Events von Tastern (`button`) und Drehreglern (`rotary`) umgehen nun den Cache und senden **immer** ein UDP-Paket an Loxone, auch wenn der Wert identisch zum vorherigen ist.
    - Sensoren (Temp, Motion, Lux) werden weiterhin dedupliziert, um das Netzwerk nicht zu fluten.

---

## [1.7.1] - 2025-12-15

### 🛡️ Global Rate Limiting
- **Traffic Queue:** Implementierung einer globalen Warteschlange, um Fehler bei der Hue Bridge ("429 Too Many Requests") zu verhindern.
    - Befehle für Einzel-Lichter werden auf max. 8-10 pro Sekunde begrenzt.
    - Befehle für Gruppen/Zonen werden auf max. 1 pro Sekunde begrenzt.
    - Loxone kann nun "feuern" so schnell es will (z.B. Szenen), die Bridge arbeitet alles sauber nacheinander ab.

### 🛠 Fixes & Verbesserungen
- **Smart Button Logic:** Taster-Events werden nun sauber gefiltert (`short_release` & `long_press`), um Fehlschaltungen zu vermeiden.
- **Rotary (Drehregler):** Sendet nun `cw` (rechts) und `ccw` (links) als Text für einfachere Einbindung in Loxone.
- **Discovery:** Tap Dial Switch wird nun vollständig erkannt (4 Tasten + Drehring separat).

---

## [1.7.0] - 2025-12-12

### 🚀 Major Features
- **Tap Dial Switch Support:** Der Philips Hue Tap Dial Switch wird nun vollständig unterstützt!
    - Alle 4 Tasten werden als einzelne Geräte erkannt.
    - Der Drehring (Rotary) wird als eigenes Gerät erkannt.
- **Smart Button Logic:** Taster-Events werden nun gefiltert:
    - Nur noch `short_release` (Klick) und `long_press` (Halten) werden an Loxone gesendet.
    - Irrelevante Events wie `initial_press` oder `repeat` werden unterdrückt, um Traffic zu sparen.
- **Rotary Logic:** Der Drehring sendet nun `cw` (Clockwise) und `ccw` (Counter-Clockwise) als Text an Loxone. Das ermöglicht das direkte Anbinden an `V+` und `V-` Eingänge von Dimmern.

### 🛠 Verbesserungen
- **XML Export:** Der Input-Generator erstellt nun automatisch digitale Eingänge für Drehregler (CW/CCW).
- **Stabilität:** `dotenv` Dependency entfernt und `package.json` Laderoutine abgesichert (verhindert Abstürze in Docker-Umgebungen).
- **UI:** Verbesserte Log-Darstellung mit Kategorien (Light, Sensor, Button).

---

## [1.6.3] - 2025-12-08

### 🛠 Bugfixes & Kompatibilität
- **3rd-Party Controller Fix:** Bei einer eingestellten Transitionszeit von `0ms` wird das `dynamics`-Objekt nun komplett aus dem Befehl entfernt (statt `duration: 0` zu senden).
    - Dies behebt Probleme mit günstigen Zigbee-Controllern, die bei `duration: 0` abstürzen oder den Befehl ignorieren.
    - Das Licht nutzt in diesem Fall das Standard-Fading des Controllers.

---

## [1.6.1] - 2025-12-03

### 🛠 Verbesserungen
- **UI Fix:** Layout-Korrektur beim Hinweis für den "All"-Befehl (Text überlappte mit Eingabefeld).
- **Styling:** Abstände in der Verbindungs-Karte optimiert.

---

## [1.6.0] - 2025-12-03

### 🚀 Features
- **Loxone Sync (Rückkanal für Lichter):** Neues Opt-In Feature im Dashboard (Tab "Lichter").
    - Ermöglicht es, den Status von Lichtern (An/Aus, Helligkeit) per UDP an Loxone zu senden, wenn diese extern (z.B. via Hue App, Alexa, Dimmschalter) geschaltet wurden.
    - Perfekt für den Eingang `Stat` am EIB-Taster Baustein, um die Visualisierung synchron zu halten.
    - Standardmäßig deaktiviert, um Netzwerk-Traffic gering zu halten.

### 🛠 Verbesserungen
- **UI Fixes:** Korrektur beim Laden der Transition-Time (0ms wurde fälschlicherweise als 400ms interpretiert).
- **Icon Cleanup:** Beim Speichern von Mappings werden Icons (💡, 🏠, etc.) im Namen nun zuverlässiger entfernt.

---

## [1.5.1] - 2025-12-03

### ⚡ Optimierungen
- **Smart "All" Logic:** Der Befehl `/all/0` nutzt nun eine **fixe Verzögerung von 100ms** zwischen den Lampen (statt abhängig von der Transition Time). Dies garantiert eine sichere Entlastung der Bridge und des Stromnetzes, unabhängig von Benutzereinstellungen.
- **Transition Fix:** Bei "Alles"-Befehlen wird die Übergangszeit (Transition) temporär auf 0ms gesetzt, damit das Ausschalten sofort sichtbar ist, während die Schleife läuft.
- **Queue Stability:** Rückkehr zur stabilen "1-Slot-Buffer" Logik für die Befehlswarteschlange, um Seiteneffekte bei schnellen Schaltvorgängen zu vermeiden.

---

## [1.5.0] - 2025-12-02

### 🚀 Features
- **Diagnose Tab:** Neuer Tab im Dashboard zeigt den Gesundheitsstatus des Zigbee-Netzwerks (Verbindungsstatus, MAC-Adresse, Zuletzt gesehen) und den Batteriestatus aller Geräte.
- **Smart "All" Command:** Der Befehl `/all/0` (oder `/alles/0`) schaltet nun alle gemappten Lichter nacheinander mit einem Sicherheitsabstand von 100ms. Dies schützt die Bridge vor Überlastung und erzeugt einen angenehmen "Wellen-Effekt".

### ⚡ Optimierungen
- **Queue Logic:** Verbesserte Warteschlange für Lichtbefehle. Verhindert das Verschlucken von schnellen Ein/Aus-Schaltvorgängen (Hybrid Queue).
- **Logging:** Zeitstempel im Log sind nun präzise (Millisekunden) und im 24h-Format. Rate-Limit Fehler (429) werden sauber abgefangen.

---

## [1.4.0] - 2025-12-02

### ⚡ Optimierungen (Logic & Performance)
- **Zero-Latency Switching:** Reine Schaltbefehle (Ein/Aus) ignorieren nun die eingestellte Übergangszeit und schalten sofort (0ms), um eine spürbare Verzögerung zu vermeiden.
- **Stable Queue:** Die Warteschlange wurde stabilisiert ("1-Slot-Buffer"). Dies verhindert das Verschlucken von schnellen Schaltfolgen (An -> Aus -> An), behält aber die "Last-Wins"-Logik für flüssiges Dimmen bei.

### 🛡️ Stabilität
- **Rate Limit Handling (429):** Fehlercode 429 ("Too Many Requests") der Hue Bridge wird nun abgefangen und als Warnung geloggt, anstatt den Log mit HTML-Fehlerseiten zu fluten.
- **Error Throttling:** Bei Fehlern wird eine kurze Wartezeit (100ms) eingefügt, um die Bridge nicht weiter zu belasten.

### 📝 Logging
- **Präzise Zeitstempel:** Logs enthalten nun Millisekunden (`HH:MM:SS.mmm`) für genaueres Debugging von Timing-Problemen.
- **24h Format:** Zeitstempel werden nun erzwungen im deutschen 24h-Format ausgegeben.

---

## [1.3.0] - 2025-12-01

### 🚀 Neu (Features)
- **Smart Lighting:**
    - **Transition Time:** Einstellbare Überblendzeit (0-500ms) im System-Tab für weichere Lichtwechsel.
    - **Command Queueing:** Verhindert "Stottern" bei schnellen Slider-Bewegungen (Loxone -> Hue). Befehle werden gepuffert.
    - **RGB Fallback:** Sendet Loxone Farben an eine reine Warmweiß-Lampe, berechnet die Bridge nun automatisch die passende Farbtemperatur (Wärme basierend auf Rot/Blau-Anteil).
    - **Capabilities:** Die Bridge liest die physikalischen Kelvin-Grenzen der Lampen aus und skaliert Loxone-Werte exakt auf diesen Bereich.
- **UI & DX:**
    - **Color Dot:** Farbiger Punkt in der Liste zeigt den aktuellen Status der Lampe.
    - **Device Details:** Info-Button (ℹ️) zeigt technische Daten (Modell, Farbraum, Kelvin-Range) im Overlay.
    - **Export Filter:** Im Export-Dialog können nun gezielt einzelne Geräte per Checkbox ausgewählt werden.

### 🛠 Verbesserungen
- **Backend:** `server.js` nutzt nun zentrales Config-Management für Transition Time.
- **Frontend:** Optimierte Dropdowns (keine bereits gemappten Geräte mehr sichtbar).
- **Docker:** Healthcheck und Pfad-Optimierungen.

---

## [1.1.0] - 2025-11-27

### 🚀 Neu (Features)
- **UI Dashboard:**
    - Live-Werte: Anzeige von Temperatur, Lux, Batteriestand (<20% = 🚨) und Schaltzustand direkt in der Liste.
    - Color Dot: Farbiger Indikator zeigt die aktuelle Lichtfarbe an (berechnet aus XY/Mirek).
    - Selection Mode: Gezielter XML-Export von ausgewählten Geräten via Checkboxen.
    - Unique Name Check: Warnung beim Überschreiben von bestehenden Mappings.
- **Hardware Support:**
    - **Rotary Support:** Volle Unterstützung für den Hue Tap Dial Switch (Drehring sendet relative Werte).
- **Technical:**
    - **Initial Sync:** Lädt beim Start sofort alle aktuellen Zustände der Lampen.
    - **Smart Fallback:** Automatische Umrechnung von RGB zu Warmweiß für Lampen, die keine Farbe unterstützen (Berechnung der "Wärme" aus Rot/Blau-Anteil).
    - **Filtered XML:** XML-Export berücksichtigt jetzt die Auswahl im UI.

### 🐛 Fehlerbehebungen (Fixes)
- Behoben: Falsche Darstellung im Dropdown bei bereits zugeordneten Geräten.
- Behoben: Checkbox-Status Verlust bei Live-Updates (durch Modal-Overlay gelöst).
- Behoben: Slash `/` wurde bei Sensoren im Export-Overlay fälschlicherweise angezeigt.

---

## [1.0.0] - 2025-11-27

### 🎉 Initial Release
- **Core:** Bidirektionale Kommunikation (Loxone HTTP -> Hue / Hue SSE -> Loxone UDP).
- **Docker:** Robustes Setup mit `data/` Ordner Persistence und Host-Network Support.
- **Setup:** Automatischer Wizard zur Erkennung der Bridge und Konfiguration von Loxone IP/Ports.
- **UI:** Modernes Dashboard mit 4 Tabs (Lichter, Sensoren, Schalter, System) und Dark Mode.
- **Integration:** XML-Template Generator für Loxone Config (Inputs/Outputs).
- **Logging:** Runtime Debug-Toggle und In-Memory Log-Buffer im UI.