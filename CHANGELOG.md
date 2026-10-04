# Changelog

Alle nennenswerten Änderungen an diesem Projekt werden in dieser Datei dokumentiert.

Das Format basiert auf [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
und dieses Projekt hält sich an [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://www.buymeacoffee.com/bausi2k)

## [2.14.0] - 2026-10-04
**Nicht erreichbare Lampen melden keine Zustandswerte mehr** – Die Bridge liefert für eine nicht erreichbare Lampe weiter den Sollzustand. Bisher lief er nach Loxone und MQTT, Loxone sah „an", obwohl die Lampe dunkel war. Jetzt bleibt er zurück, bis die Lampe wieder erreichbar ist.

### ✨ Neu
- **Bei `reachable = 0` gehen `on`, `bri`, `mirek` und `hex` der Lampe weder per UDP noch per MQTT hinaus.** Loxone behält den letzten bekannten Wert, `reachable` zeigt, dass er nicht belastbar ist. Das gilt für Ereignisse, den Start und das Echo eines angenommenen Befehls. Sensoren, Taster und Gruppen sind nicht betroffen.
- **`reachable` geht jetzt auch ohne Loxone Sync per UDP an Loxone.** Es ist eine Statusmeldung und kein Rückkanal des Schaltzustands.
- **`reachable` wird beim Start erneut gesendet,** auch wenn sich der Wert nicht geändert hat, und zwar vor den Lampenzuständen. Nach einem Neustart von Loxone kennt es den Wert damit sofort.

### 📐 Hinweise
- **Verhaltensänderung gegenüber 2.13.0:** Dort liefen die Zustandswerte weiter. Wer in Loxone ohnehin auf `reachable` filtert, merkt nichts.
- **Ausgeschaltet am Wandschalter:** Die Lampe geht von selbst auf `reachable = 0`. Loxone sieht dann kein „on 0", sondern `reachable 0` und den alten Wert. `on` springt bewusst nicht auf 0.
- Kommt eine Lampe zurück, geht erst `reachable 1` hinaus, danach der frisch geladene Zustand.
- Gruppen haben weiterhin kein `reachable`.

### 🧪 Tests
- Abdeckung von 463 auf 474 Tests erweitert.

## [2.13.0] - 2026-10-03
**Nicht erreichbare Lampen erkennen** – Eine Lampe, die per Zigbee nicht erreichbar ist, stand bisher weiter auf „an". Die Bridge meldet für sie den Sollzustand, und Befehle an sie galten als ausgeführt. Jetzt gibt es einen eigenen Statuswert `reachable`, und abgelehnte Befehle werden erkannt.

### ✨ Neu
- **Statuswert `reachable` (1/0) für Lampen.** Er wird beim Start und live aus der Zigbee-Verbindung der Bridge geladen und geht per UDP (`hue.<name>.reachable`, bei aktivem Loxone Sync) und MQTT hinaus. In Loxone legst du dafür einen virtuellen UDP-Eingang mit der Befehlserkennung `hue.<name>.reachable \v` an; der XML-Export enthält ihn nicht.
- **Zustand nach der Rückkehr.** Kommt eine Lampe wieder, lädt die Bridge ihren Zustand neu, denn eine Bedienung am Wandschalter in der Zwischenzeit wird nicht als Ereignis gemeldet.
- **Anzeige:** Das Dashboard zeigt „NICHT ERREICHBAR", die Diagnose nennt es in der Geräteliste, und der Detaildialog hat die Zeile „Erreichbarkeit".
- **Spalte „Nicht ausgeführt"** in der Zuverlässigkeit der Befehle, im Detaildialog mit dem letzten Fehler der Bridge.

### 🐛 Behoben
- **Befehle, die die Bridge ablehnt, galten als ausgeführt (#67).** Die Bridge antwortet mit HTTP 200 oder 207 und einer Fehlerliste, etwa „communication issues". Bei einer Lampe wird der Sollzustand jetzt nicht mehr gesetzt, es wird nicht wiederholt, und das Log enthält eine Warnung mit dem Text der Bridge. Bei einer Gruppe bleibt der Zustand gesetzt, weil die übrigen Lampen den Befehl ausgeführt haben; die Warnung erscheint trotzdem.

### 📐 Hinweise
- **`on` bleibt unverändert.** Eine nicht erreichbare Lampe steht weiter auf dem Sollzustand der Bridge. Filtere in Loxone auf `reachable`.
- Nur Lampen bekommen `reachable`, keine Gruppen, Sensoren oder Taster.
- Ob die Bridge einen verpassten Befehl nach der Rückkehr selbst nachholt, ist ungeprüft. Du kannst in Loxone die steigende Flanke von `reachable` nutzen, um den Sollzustand erneut zu senden.

### 🧪 Tests
- Abdeckung von 444 auf 463 Tests erweitert.

## [2.12.2] - 2026-10-03
**Drehring-Eingänge für Dial-Geräte wieder im Export** – Korrektur zu 2.12.1. Dort fehlten im Loxone-Export bei Zuordnungen, die an einer Taste eines Hue Tap Dial hängen, die Eingänge für die Drehrichtung.

### 🐛 Behoben
- **Zuordnungen an einer Taste eines Hue Tap Dial bekommen im Export wieder die Eingänge „Rotary CW" und „Rotary CCW".** 2.12.1 erkannte Drehringe nur noch am Drehring-Dienst selbst. Ein Drehereignis kommt aber bei jeder Zuordnung des Geräts an, wenn der Drehring nicht selbst zugeordnet ist – bei so einer Taste also genau dort, wo Loxone den Eingang braucht. Erkannt wird jetzt am Gerät: Hat es einen Drehring-Dienst, bekommen alle seine Zuordnungen die Eingänge.

### 📐 Hinweise
- **Berichtigung zu 2.12.1:** Die dortige Aussage, Einzeltasten bekämen nie Drehereignisse und die Eingänge seien für sie wirkungslos, galt nicht für Dial-Geräte. Für Geräte ohne Drehring bleibt sie richtig: Dort entstehen keine Rotary-Eingänge mehr.
- Betroffen war nur der Download der Loxone-Eingaben. Wer ihn mit 2.12.1 neu erzeugt hat, holt ihn mit 2.12.2 noch einmal. Der laufende Betrieb und bereits eingerichtete Eingänge in Loxone waren nicht betroffen.

### 🧪 Tests
- Abdeckung von 438 auf 444 Tests erweitert.

## [2.12.1] - 2026-10-02
**Umbenennungen aus der Hue-App kommen an** – Wer eine Lampe, eine Gruppe oder einen Sensor in der Hue-App umbenannte, sah in der Zuordnung weiter den alten Namen, ebenso im Backup und als Kommentar im XML-Export für Loxone. Der Name wird jetzt aus der Bridge nachgezogen.

### 🐛 Behoben
- **Der Name eines Geräts wird mit der Bridge abgeglichen** – beim Start und beim Laden der Zielliste im Dashboard. Geändert wird nur die Beschriftung: Befehlsnamen, die Zuordnung per UUID, die Einstellungen je Gerät und alles, was in Loxone programmiert ist, bleiben unberührt. Jede Änderung steht im Log: `Name aktualisiert (wcspiegel): "alt" -> "neu"`.
- Umbenannt wird nur, was die Bridge liefert. Fehlt ein Gerät in der Antwort oder fällt eine Abfrage aus, bleibt der gespeicherte Name stehen – ein fehlender Eintrag ist kein Beleg für ein gelöschtes Gerät.
- Die Zusätze `(Taste n)` und `(Drehring)` bleiben erhalten. Der Export erkennt Drehringe also weiterhin.

### 📐 Hinweise
- Beim ersten Start nach dem Update werden veraltete Beschriftungen einmalig korrigiert. Wer in der `mapping.json` oder einem Backup von Hand eigene Texte in `hue_name` gepflegt hat, verliert sie; die Oberfläche bietet dafür keine Eingabe.
- Die Rotary-Eingänge im Loxone-Export entstehen jetzt nur noch für den Drehring selbst. Erkannt wird er am Diensttyp bei der Bridge, nicht mehr am Namenstext. Vorher bekam auch eine einzelne Taste eines Geräts mit „Dial" im Namen diese Eingänge – ohne Wirkung, denn für sie kommen nie Drehereignisse an.

### 🧪 Tests
- Abdeckung von 420 auf 438 Tests erweitert.

## [2.12.0] - 2026-10-02
**Alle Geräteinformationen auf einen Klick** – Eine Rückmeldung zu einem Gledopto-Controller (RGB- und Warmweiß-Ausgang) warf die Frage auf, welchen Weißton-Bereich die Bridge für die Lampe meldet. Das war nirgends sichtbar: Die Diagnose zeigte bei „Weiß" nur ein Häkchen. Jetzt steht dort der Bereich, und mit ihm alles, was die Bridge und wir über ein Gerät wissen.

### ✨ Neu
- **Diagnose → Geräte ist nach Typ gruppiert** (Licht, Gruppe, Sensor, Schalter, jeweils mit Anzahl). Jede Zeile ist klickbar, auch per Tastatur, und trägt ein kleines ⓘ als Hinweis. Die Spalte „Typ" entfällt, der Gruppenkopf ersetzt sie.
- **Das Detailfenster zeigt jetzt alle Angaben zu einem Gerät:**
    - **Fähigkeiten** mit dem Weißton-Bereich in mirek und Kelvin – als *verwendet* und als *von der Bridge gemeldet*. Meldet die Bridge keinen Bereich, steht dort ausdrücklich „Ersatzwert". Dazu kleinste Helligkeit, Farbraum und Effekte.
    - **Gerät** (Hersteller, Produkt, Modell, Software, Zertifizierung), **Zigbee** (Status, MAC-Adresse, Kanal), **Strom** (Batterie), bei Gruppen die **Mitglieder**.
    - **Alle übrigen Bridge-Dienste** mit sämtlichen Feldern: Zustand des Lichts, Bewegung, Temperatur, Helligkeit, Kontakt, Tasten.
    - **Zuverlässigkeit** seit dem letzten Neustart.
    - Aufklappbar: **lokale Daten** (Mapping und Status, jedes Feld) und die **Rohdaten der Bridge** als JSON – gedacht zum Kopieren in ein Issue.
- Das Fenster öffnet sich auch dann, wenn die Zielliste das Gerät nicht enthält. Die Bridge-Daten werden nach dem Öffnen nachgeladen: Das Fenster geht sofort auf, und eine träge Bridge blockiert es nicht. Antwortet sie nicht, bleiben die lokalen Daten sichtbar und der Grund steht im Fenster.

### 🔒 Sicherheit
- **axios 1.19.0 → 1.20.0.** `npm audit` meldete zwölf Hinweise (acht mit hohem Schweregrad), alle in 1.20.0 behoben. Sie betreffen überwiegend Fetch- und HTTP/2-Adapter, Proxys und Weiterleitungen, die wir nicht nutzen; die tatsächliche Angriffsfläche im lokalen Netz war gering.
- **ip-address 10.5.0 → 10.7.2** (Dependabot, indirekte Abhängigkeit von `mqtt`), behebt vier Hinweise.
- Die neue Detail-Route fragt nur Geräte ab, die einem Mapping zugeordnet sind, und prüft die UUID, bevor sie in einen Pfad zur Bridge gelangt. Sie ist damit kein Zugang zu beliebigen Bridge-Ressourcen.

### 📐 Hinweise
- Am Schaltverhalten ändert sich nichts. Die Umrechnung von Kelvin auf den gemeldeten Bereich der Lampe gab es schon; sichtbar war nur nie, welcher Bereich dabei zugrunde liegt.
- Die Chips auf den Lampen- und Sensor-Tabs öffnen dasselbe Fenster und zeigen die neuen Abschnitte ebenfalls.

### 🧪 Tests
- Abdeckung von 400 auf 420 Tests erweitert.

## [2.11.2] - 2026-09-28
**Wartezeit in der Befehlswarteschlange sichtbar** – Nach Meldungen über Lichter, die mehrere Sekunden zum Schalten brauchten, wurde ein Logexport ausgewertet. Ergebnis: kein Absturz, kein fehlschlagender Healthcheck, kein Fehler in unserem Code – die spürbaren Verzögerungen gingen durchgängig auf zwei Ursachen zurück, die außerhalb der Bridge liegen: Netzwerkaussetzer der Hue Bridge selbst (ECONNREFUSED, HTTP 503, Timeouts) und Befehlsschwärme, die sich die gemeinsame Warteschlange aller Lichter teilen. Damit sich das künftig ohne Log-Archäologie klären lässt, protokolliert die Bridge jetzt die tatsächliche Wartezeit.

### ✨ Neu
- **Jeder gesendete Befehl protokolliert seine Wartezeit in der Warteschlange:** `OUT -> Hue (wcspiegel) [Warteschlange 18432ms]: {...}`. Das trennt direkt, ob ein Befehl in unserer Schlange wartete oder ob die Bridge selbst gebraucht hat.
- **Ab 2 Sekunden Wartezeit gibt es zusätzlich eine Warnmeldung** – mit Namen, Wartezeit und der Zahl noch wartender Befehle. Sie ist auch **ohne eingeschalteten Debug-Modus** sichtbar, anders als die reine Protokollzeile.
- Auch eine durch Überholung verworfene Nacharbeit („Einschalten aufteilen", „Befehl wiederholen") meldet jetzt ihre Wartezeit – sie ist selbst ein Symptom einer langsamen Warteschlange.

### 📐 Hinweise
- Diese Version ändert nichts am Schaltverhalten selbst. Sie macht nur sichtbar, was vorher nur aus Rohlogs rekonstruierbar war.
- Bei der Untersuchung bestätigt: kein einziger Absturz-Eintrag im gesamten Beobachtungszeitraum, kein Docker-Healthcheck definiert, der eigene Neustarts auslösen könnte.

### 🧪 Tests
- Abdeckung von 394 auf 400 Tests erweitert.

## [2.11.1] - 2026-09-23
**Ueberholte Folgebefehle gehen nicht mehr hinaus** – Bei schneller Befehlsfolge (etwa einem Dimmvorgang mit Zwischenwerten) konnte die zweite Haelfte von „Einschalten aufteilen" oder eine geplante „Befehl wiederholen"-Wiederholung ueberholt und trotzdem gesendet werden. Sichtbar als kurzes Aus-An-Flackern, obwohl der zuletzt gewuenschte Zustand laengst feststand.

### 🐛 Behoben
- **Ueberholte Folgebefehle werden jetzt verworfen statt gesendet.** Betroffen waren die Optionen „Einschalten aufteilen" und „Befehl wiederholen": Ihre Nacharbeit prüfte bislang nur beim Planen, ob sie noch aktuell ist – nicht mehr unmittelbar vor dem Senden. Dazwischen liegt die Befehlsdrosselung, in einem am Betrieb gemessenen Fall knapp zwei Sekunden, in denen zwei weitere Befehle eintrafen. Die Prüfung sitzt jetzt zusätzlich direkt vor dem Senden; ein verworfener Befehl steht als `SKIP` im Debug-Log.
- **Live am Produktivsystem nachvollzogen:** Eine Lampe mit „Einschalten aufteilen" erhielt binnen zehn Sekunden zehn abwechselnde Ein-/Aus-Befehle. Ohne die Korrektur landeten die Telegramme über drei Sekunden verteilt und in vertauschter Reihenfolge auf der Leitung – mit der Korrektur nur noch das jeweils Aktuelle.

### 📐 Hinweise
- „Zustand nachlesen und korrigieren" war von diesem Fehler nicht betroffen: Es sendet keine geplante Nacharbeit, sondern liest den Zustand nach und entscheidet dann neu.
- Primärbefehle werden nie verworfen – nur die Nacharbeit einer Funkmaßnahme.
- Am Verhalten der Optionen selbst ändert sich nichts. Wer „Einschalten aufteilen" oder „Befehl wiederholen" bereits nicht empfohlen bekommen hat, bekommt sie weiterhin nicht empfohlen (siehe 2.9.1).

### 🧪 Tests
- Abdeckung von 387 auf 394 Tests erweitert.
- Die neuen Tests steuern den Ablauf über ein kontrolliertes Versprechen statt über Wartezeiten, damit sie nicht von der Auslastung der Maschine abhängen. Beide Absicherungen wurden gegengeprüft, indem der jeweilige Fehler absichtlich wieder eingebaut wurde.

## [2.11.0] - 2026-09-21
**Was auf dem Handy nicht ging** – Die Oberflächen-Umstellung aus 2.10.0 hatte drei Stellen übersehen, die erst im Alltag auffielen: ein Detailfenster, in dem man feststeckte, Gerätekarten, die aus dem Bildschirm liefen, und Farben, die den Dunkelmodus nicht kannten. Dazu eine klarere Sensorliste.

### 🐛 Behoben
- **Im Geräte-Detailfenster war man auf dem Handy gefangen.** Gemessen auf 402 × 874: 1154 px Inhalt in einer 699 px hohen Box, ohne Scrollbereich. Der Schließen-Knopf lag 300 px unterhalb des Sichtbaren, rund 455 px Inhalt waren unerreichbar. Das Kreuz zum Schließen war 10 × 15 px groß – Apple nennt 44 × 44 als Mindestmaß. Die Fensterhöhe richtet sich jetzt zusätzlich nach `dvh`, weil `vh` auf iOS die große Ansicht ohne die Safari-Leisten meint.
- **Gerätekarten liefen rechts aus dem Bildschirm.** Bei 1900 px standen 45 px waagrechter Überlauf zu Buche, 12 von 14 Karten waren betroffen, der größte Überstand betrug 102 px. Lange Namen ohne Leerzeichen konnten nicht umbrechen und schoben die Badges hinaus.
- **Farben, die den Dunkelmodus nicht kannten.** Sechs Chips und der Export-Knopf trugen feste Hellfarben. Der Knopf stand im Dunkelmodus mit weißer Schrift auf fast weißem Grund (Kontrast 1,06 : 1), der Chip „🔓 OFFEN" bei 1,27 : 1, der Batterie-Chip unter 20 % bei 1,80 : 1 in beiden Modi. Alle liegen jetzt über dem Richtwert 4,5.

### ✨ Neu
- **Beschriftete Blöcke in der Sensorliste.** Innerhalb jeder Gruppe steht jetzt „Ausgelöst (2)" über „Ruhig (10)" beziehungsweise „Offen" über „Geschlossen". Bisher trennte die beiden nur eine dünne Linie, der man nicht ansah, was sie bedeutet. Fühler ohne Zustandsbegriff – Temperatur, Helligkeit – bekommen bewusst keine Blocknamen.

### 🔧 Verbessert
- **„AUS" ist nicht mehr hinterlegt.** Nur der eingeschaltete Zustand ist eine Meldung; der ausgeschaltete ist der Normalfall.
- Die Überschrift der rechten Spalte heißt jetzt „Zugeordnet". Sie meinte immer die eingerichteten Zuordnungen – mit einem Block „Ausgelöst" darunter wäre „Aktiv" zweideutig geworden.

### 📐 Hinweise
- **Node 24 ist jetzt in `package.json` gefordert.** Das war schon vorher so – Abbild und CI laufen seit jeher darauf – stand aber nirgends. Unter Node 22 gab es statt einer Meldung einen Absturz mit `ERR_UNKNOWN_BUILTIN_MODULE` für `node:sqlite`.
- Ob eine Gerätekarte eng ist, hängt am Gitter und nicht an der Fensterbreite. Die Karte stellt sich deshalb anhand ihrer eigenen Breite um. Browser ohne Unterstützung dafür (vor Chrome 105, Safari 16, Firefox 110) zeigen sie einzeilig, ohne Überlauf.
- **Die Release-Notes zu 2.10.0 waren falsch.** Der Tag wurde gesetzt, bevor der zugehörige CHANGELOG-Abschnitt geschrieben war; der Workflow hat deshalb den Text von 2.9.1 veröffentlicht. Der Abschnitt unten trägt nach, was in 2.10.0 tatsächlich enthalten ist, und die Notizen auf GitHub sind inzwischen korrigiert.
- Am Verhalten der Brücke selbst ändert sich auch in dieser Version nichts. Es bleibt eine reine Oberflächen-Ausgabe.

### 🧪 Tests
- Abdeckung von 244 auf 387 Tests erweitert (2.10.0 und 2.11.0 zusammen).
- Neu darunter: eine Prüfung, dass Fensterabfragen im Stylesheet am Dateiende stehen – bei gleicher Spezifität gewinnt sonst die spätere Regel, und ein Umbruch blieb wirkungslos, obwohl er richtig geschrieben war.
- Ebenfalls neu: ein Abgleich der Node-Version zwischen `package.json`, `Dockerfile` und Workflow sowie ein Abgleich der Versionsnummer zwischen `package.json`, `package-lock.json`, CHANGELOG und README. Letzterer hätte die falschen Notizen zu 2.10.0 verhindert.

## [2.10.0] - 2026-09-17
**Die Oberfläche neu geordnet** – Hell- und Dunkelmodus, eine Navigationsspalte statt Reiterleiste, und die Inhalte in Untertabs sortiert.

Hinweis: Der Tag wurde gesetzt, bevor dieser Abschnitt geschrieben war. Auf GitHub stand deshalb zunächst der Text von 2.9.1; die Notizen dort sind seit 21.09.2026 korrigiert.

### ✨ Neu
- **Hell- und Dunkelmodus, dreistufig.** Der Schalter in der Kopfzeile kennt Hell, Dunkel und System; System ist die Vorgabe und folgt der Einstellung des Geräts. Die Wahl bleibt erhalten und wird vor dem ersten Zeichnen angewendet, damit nicht kurz das falsche Schema aufblitzt.
- **Navigationsspalte statt Reiterleiste.** Auf breiten Schirmen steht die Navigation links und bleibt beim Blättern stehen. Auf dem Handy liegt sie hinter einem Burgermenü, das sich über die Ansicht legt und beim Wechsel von selbst schließt.
- **Untertabs in Diagnose und System.** Statt einer langen Kachelkolonne gibt es je drei Bereiche: Netzwerk · Geräte · Zuverlässigkeit und Einstellungen · Wartung · Logs. Die Wahl wird je Gruppe getrennt gemerkt.
- **Namensvorschlag beim Anlegen.** Erst das Hue-Gerät wählen, dann den Namen – und der wird aus dem Gerätenamen vorgeschlagen: „WC Spiegel" wird zu `wcspiegel`, „Küche Arbeitslicht" zu `kuechearbeitslicht`. Umlaute werden ausgeschrieben, nicht weggeworfen. Ist der Name vergeben, wird durchnummeriert. Wer selbst tippt, behält seine Eingabe.

### 🔧 Verbessert
- **Breite Bildschirme werden genutzt.** Bis 1800 px Inhaltsbreite, die Geräteliste in mehreren Spalten statt einer einzigen langen Kolonne.
- **Die Version steht in der Kopfzeile** statt als erste Zeile in den Einstellungen. Es ist die Angabe, die man am häufigsten kurz nachsieht. Solange sie noch nicht geladen ist, bleibt der Platz leer statt eine leere Pille zu zeigen.
- **Die Einstellungen sind in Abschnitte gegliedert**, deren Überschriften dem Farbschema folgen statt einem festen Grauton.

## [2.9.1] - 2026-09-16
**Nachsteuern, ohne dass man es sieht** – Die Prüfung, ob eine Leuchte einen Befehl wirklich ausgeführt hat, lag bisher zu früh und korrigierte zu grob: Sie schaltete das Gerät sichtbar ein zweites Mal. Beides ist behoben, und zwar auf Grundlage von 52 im Betrieb gemessenen Fällen statt einer Schätzung.

### 🔧 Verbessert
- **Es wird nur noch nachgesendet, was tatsächlich fehlt.** Stimmt allein die Helligkeit nicht, geht auch nur sie hinaus – ohne den Einschaltbefehl. Die Leuchte schaltet dadurch nicht sichtbar ein zweites Mal. Stimmt dagegen der Schaltzustand nicht, geht weiterhin der vollständige Befehl hinaus; eine Helligkeit an eine ausgeschaltete Leuchte nützt nichts.
- **Die Prüfzeitpunkte liegen jetzt bei 5, 60 und 120 Sekunden** statt bei 15 und 90. Über elf Tage gemessen trat der Helligkeitseinbruch im Mittel nach rund 50 Sekunden auf, in Einzelfällen erst nach 112. Die alten 15 Sekunden bestätigten deshalb fast immer einen Zustand, der erst danach kippte – von 52 Fällen lagen nur 5 darunter. Die neue erste Stufe bei 5 Sekunden fängt stattdessen das ab, was der Funk beim ersten Versuch verschluckt hat.
- Damit ersetzt die Prüfung, was bisher „Befehl wiederholen" blind erledigt hat: Statt denselben Befehl ein zweites Mal zu schalten, wird nachgesehen und gezielt nachgereicht. Im Log steht danach, **was** gefehlt hat.

### ✨ Neu
- **Zigbee-Kanal und Extended PAN ID im Diagnose-Tab.** Die Frequenz steht dabei – erst damit lässt sich ohne Nachschlagen beurteilen, ob das Zigbee-Netz einem WLAN-Kanal ins Gehege kommt. Zigbee und WLAN benutzen völlig verschiedene Kanalnummern; ein direkter Vergleich der Nummern führt in die Irre.

### 🔒 Sicherheit
- **Zwei gemeldete Schwachstellen im Query-String-Parser geschlossen** (beide moderat, beide in `qs`). Nur die Abhängigkeiten ändern sich, kein Sprung auf eine neue Hauptversion des Webservers – der Fix ist innerhalb der bisherigen Version zu haben.

### 📐 Hinweise
- Geräte mit eingeschaltetem Nachlesen erzeugen jetzt **drei** Abfragen je Befehl statt zwei.
- Wird nur die Helligkeit nachgesendet, verlässt sich die Brücke auf den kurz zuvor gelesenen Zustand. Schaltet jemand genau in diesem Moment die Leuchte aus, geht die Helligkeit an eine ausgeschaltete Leuchte – folgenlos, aber anders als bisher, wo der mitgesendete Einschaltbefehl sie wieder eingeschaltet hätte.
- Ein neuer Befehl verwirft die geplante Prüfung des vorherigen. Durch die späteren Zeitpunkte passiert das häufiger: Wer innerhalb von zwei Minuten erneut schaltet, bekommt für den ersten Befehl keine Prüfung mehr. Das ist gewollt – der zweite Befehl bringt seine eigene mit.
- „Befehl wiederholen" bleibt als Option erhalten, wird für den Alltag aber nicht mehr gebraucht.

### 🧪 Tests
- Abdeckung von 224 auf 244 Tests erweitert.

## [2.9.0] - 2026-08-26
**Weniger überflüssige Funkbefehle, Logs über Tage statt Stunden** – Zwei Engstellen, die im Alltag niemandem auffallen, aber beide dieselbe Ursache hatten: Es wurde mehr gesendet und mehr gelesen als nötig. Bei Gruppen ging fast jeder zweite Befehl unnötig hinaus, und ein Logexport reichte nie weiter zurück als etwa zwei Tage – zu wenig, um einer Störung auf die Spur zu kommen, die nur alle paar Tage auftritt.

### ✨ Neu
- **Logexport über einen Zeitraum.** Neben dem Download-Knopf steht ein Auswahlfeld: letzter Tag, 3, 7 oder 14 Tage. Bisher waren es immer die letzten 10.000 Zeilen – im Betrieb etwa 50 Stunden. Die Datenbank bewahrt rund zehn Tage auf, diese Daten waren also längst vorhanden, nur nicht abrufbar. Genau daran sind zwei Auswertungen gescheitert, weil die Stichprobe zu klein blieb. Ohne Auswahl bleibt alles wie bisher.

### 🔧 Verbessert
- **Identische Gruppenbefehle gehen nicht mehr doppelt hinaus.** Alle Gruppen teilen sich eine Warteschlange, die höchstens einen Befehl pro Sekunde durchlässt – mehr verträgt die Hue Bridge nicht. Über fünf Tage gemessen war mehr als die Hälfte dieser Befehle wortwörtlich identisch mit dem vorhergehenden: eine Farbtemperatur-Nachführung, die im Minutentakt denselben Wert schickt, solange das Licht an ist. An einem Tag lief das über neun Stunden am Stück. Jeder dieser Befehle hat einen Platz belegt, den ein echter Befehl brauchte. **47,9 % davon entfallen jetzt.**
- **Zwei Gruppen im selben Raum schalten wieder näher beieinander.** Weil die Warteschlange verstopft war, hing die zweite Gruppe im Büro im Mittel gut eine Sekunde hinter der ersten – in Lastspitzen standen beide sekundenlang gegenläufig, eine an und eine aus. Das tritt nun deutlich seltener auf.
- **Die Suche im Logfenster durchsucht die gesamte Datenbank.** Bisher sah sie nur die letzten 10.000 Zeilen, weil erst geladen und dann gefiltert wurde. Ein Suchbegriff, dessen letzter Treffer drei Tage zurückliegt, wird jetzt gefunden.
- **Das Dashboard lädt nur noch, was es anzeigt** – 100 Zeilen statt 10.000 bei jeder Aktualisierung.
- **Mehrfach angegebene Suchparameter** führen nicht mehr zu einem Serverfehler, sondern zu einer verständlichen Meldung.

### 📐 Hinweise
- Das Überspringen gilt **nur für Gruppen**. Bei einzelnen Leuchten ist der Anteil identischer Wiederholungen gering, und die Warteschlange dort ist zehnmal schneller.
- Geräte mit „Befehl wiederholen", „Einschalten aufteilen" oder „Zustand nachlesen" sind ausgenommen. Dort ist das wiederholte Senden ja gerade der Zweck.
- **Spätestens alle fünf Minuten geht auch ein unveränderter Befehl wieder hinaus.** Die Brücke merkt sich beim Senden, was sie angeordnet hat – nicht, was die Leuchte tatsächlich getan hat. Ohne diese Auffrischung bliebe eine Gruppe, die einen Befehl nie ausgeführt hat, dauerhaft im falschen Zustand. Fünf Minuten nehmen fast den ganzen Gewinn mit: eine Minute spart nur 4,7 %, fünfzehn Minuten bringen gegenüber fünf nur 2,6 Prozentpunkte mehr, verdreifachen aber die Zeit, in der eine Abweichung unbemerkt stehen bleibt.
- Eine Änderung, die jemand direkt in der Hue-App vornimmt, wird weiterhin korrigiert – sie meldet sich über den Eventstream, und der Befehl geht dann hinaus.
- Übersprungene Befehle stehen im Debug-Modus als `SKIP` im Log. Ohne diesen Eintrag sähe es aus, als wäre ein Befehl verlorengegangen.
- Die Aufbewahrung der Logdatenbank bleibt unverändert bei 50.000 Zeilen, also rund zehn Tagen.

### 🧪 Tests
- Abdeckung von 193 auf 224 Tests erweitert.
- Ehrlich zur Wirkung im Dashboard: Die Log-Abfrage hat den Betrieb zwar blockiert, aber gemessen nur 2,96 ms alle zwei Sekunden. Als Erklärung für verpasste Ereignisse oder verzögerte Rückmeldungen scheidet sie damit aus – das war eine Vermutung, die die Messung nicht bestätigt hat. Die Abfrage ist jetzt trotzdem rund neunzigmal schneller.

## [2.8.0] - 2026-08-19
**Zustand nachlesen statt der Bridge glauben** – Die Hue Bridge bestätigt einen Befehl sofort, noch bevor die Leuchte ihn ausgeführt hat. Stimmt beides nicht überein, fällt das erst eine Minute später auf – und bis dahin hat Loxone einen Zustand gemeldet bekommen, den es nie gab. Neu ist eine Option, die genau das prüft und bei echter Abweichung nachsteuert.

### ✨ Neu
- **Option „Zustand nachlesen und korrigieren" je Gerät:** 15 und 90 Sekunden nach einem Befehl wird der tatsächliche Zustand der Leuchte abgefragt. Nur wenn er nachweislich abweicht, geht der Befehl ein zweites Mal hinaus. Die beiden Zeitpunkte sind nicht geraten, sondern aus dem Dauerbetrieb abgeleitet: ein Helligkeitseinbruch zeigte sich nach 11 Sekunden, unerwartete Ein-/Aus-Wechsel nach 49 bis 82 Sekunden. Eine einzelne Prüfung fängt immer nur eines von beidem.
- **Neue Spalte „Nachgesteuert" im Diagnose-Tab**, in der Form „2 / 14“: wie oft eine Prüfung eine Abweichung gefunden hat, gemessen an allen Prüfungen. Bei Geräten ohne eingeschaltetes Nachlesen bleibt die Spalte leer – eine 0 würde dort fälschlich nahelegen, es sei geprüft worden.

### 🔧 Verbessert
- **Ehrlichere Beschriftung der Funkmaßnahmen.** Für „Einschalten aufteilen“ und „Befehl wiederholen“ ließ sich in einer Messung über 46 Stunden **kein Nutzen nachweisen**. Entscheidend war die Vergleichsleuchte ohne jede Maßnahme: sie hat sich im selben Zeitraum genauso verbessert. Die ursprünglichen Ausgangswerte beruhten auf zu wenigen Schaltvorgängen, um daraus etwas abzuleiten. Beide Optionen bleiben erhalten, sind aber nicht mehr als empfohlen gekennzeichnet; der Hilfetext benennt das Ergebnis offen.
- Vermutlicher Grund für ihr Versagen: die Wiederholung folgt bereits nach 300 Millisekunden und fällt damit in dasselbe Störfenster. Ist der Funkweg veraltet, ist er es auch 300 Millisekunden später. Das Nachlesen setzt deshalb deutlich später an.

### 📐 Hinweise
- Das Nachlesen läuft **nur für ausdrücklich markierte Geräte** und über dieselbe Warteschlange wie alle anderen Befehle. Bei 30 Leuchten wären es sonst 60 zusätzliche Abfragen pro Schaltvorgang.
- Ist die Bridge beim Nachlesen nicht erreichbar, wird **nicht** nachgesteuert. Ein fehlgeschlagener Abruf sagt nichts über die Leuchte aus.
- Die drei Maßnahmen schließen einander weiterhin aus – gemeinsam aktiviert ließe sich nicht mehr sagen, welche geholfen hat.

### 🧪 Tests
- Abdeckung von 174 auf 193 Tests erweitert.

## [2.7.1] - 2026-08-14
**Einschaltbefehl aufteilen** – Manche Leuchten gehen an, bleiben aber auf ihrer geringsten Helligkeit stehen. Die Ursache ließ sich diesmal am laufenden System nachweisen und gezielt behandeln. Dazu alle bekannten Sicherheitslücken in den Abhängigkeiten.

### ✨ Neu
- **Option „Einschalten aufteilen" je Gerät:** Statt Einschalten, Helligkeit und Farbe in einem Rutsch zu senden, geht zuerst nur das Einschalten hinaus und 100 ms später der Rest. Hintergrund: Die Hue Bridge setzt aus einem Befehl mehrere Funktelegramme ab. War die Leuchte stundenlang aus, ist ihr Funkweg im Netz veraltet – das Einschalten wird so lange wiederholt, bis es ankommt, doch der unmittelbar folgende Helligkeitswert fällt genau in das Zeitfenster, in dem die Verbindung noch nicht steht. Er geht verloren, und die Leuchte bleibt auf ihrem gespeicherten Wert stehen; nach dem Ausschalten ist das die unterste Stufe. Die Pause liegt jetzt dort, wo die Verbindung entsteht.
- **Die beiden Gegenmaßnahmen sind jetzt ein Auswahlfeld** („Keine" / „Einschalten aufteilen" / „Befehl wiederholen") statt zweier Häkchen. Sie schließen einander aus – gemeinsam aktiviert ließe sich nicht mehr sagen, welche geholfen hat.

### 🔒 Sicherheit
- **Alle 9 gemeldeten Schwachstellen in den Abhängigkeiten behoben** (5 hoch, 4 mittel). Nur `package-lock.json` ändert sich; alle Sprünge bleiben innerhalb der bisherigen Versionsbereiche. Praktisch erreichbar waren in einer Heimnetz-Installation zwei davon, beide über die Weboberfläche und beide auf Überlastung ausgelegt.

### 🧹 Aufgeräumt
- **Fünf Dateien aus dem Projektwurzelverzeichnis entfernt**, die dort nicht hingehörten. Vier davon führte Node bei jedem Testlauf mit aus, ohne dass sie irgendetwas prüften – sie gaben ihr Ergebnis nur auf der Konsole aus und konnten daher nie fehlschlagen. Eine davon war ein versehentlich hereingeratenes Probe-Skript für ein ganz anderes Gerät, das bei jedem Durchlauf echte Netzwerkanfragen stellte und allein 31 Sekunden kostete. Die zwei inhaltlich sinnvollen Fälle – Batterieanzeige und Entdopplung erkannter Befehle – sind als richtige Tests mit Zusicherungen neu geschrieben.

### 🧪 Tests
- Abdeckung von 147 auf 174 Tests erweitert. Der Testlauf dauert dadurch nur noch ein Viertel so lang.
- Ein zeitkritischer Test der Warteschlange prüfte einen Zwischenzustand und schlug dadurch je nach Auslastung der Maschine fehl. Er hatte den Build der Version 2.7.0 blockiert – diese Versionsnummer wurde deshalb nie veröffentlicht.
- Eine Prüfung sorgt künftig dafür, dass im Wurzelverzeichnis keine Datei mehr liegt, die der Testlauf versehentlich mit ausführt.

## [2.6.1] - 2026-08-13
**Sensorliste nach Aktivität sortiert** – Ein ausgelöster Melder stand bisher weiter unten als ein ruhender mit schwächerer Batterie. Genau das Gerät, nach dem man in der Liste sucht, landete dadurch am Ende.

### 🔧 Verbessert
- **Aktive Sensoren stehen jetzt oben.** Ausgelöste Bewegungsmelder und offene Kontakte werden vor die ruhenden gereiht, getrennt durch eine dezente Linie. Innerhalb beider Blöcke gilt unverändert: schwache Batterie zuerst, dann alphabetisch. Temperatur- und Helligkeitswerte zählen dabei bewusst nicht als „aktiv" – sie liegen immer an und würden den oberen Block dauerhaft füllen.
- Die Aufteilung geschieht innerhalb der bestehenden Gruppen (Kontakte, Bewegung, Sonstige), die Überschriften und ihre Zähler bleiben erhalten. Die Listen der Lichter und Schalter sind unverändert.

### 🧪 Tests
- Abdeckung von 135 auf 147 Tests erweitert.

## [2.6.0] - 2026-08-13
**Unzuverlässige Leuchten erkennen und ausgleichen** – Manche Leuchten führen Befehle nicht zuverlässig aus: sie schalten nicht ab, oder sie gehen an und bleiben dabei auf ihrer alten, oft sehr niedrigen Helligkeit stehen. Betroffen sind typischerweise Geräte anderer Hersteller. Diese Version macht das Problem im Dashboard sichtbar und bietet eine Gegenmaßnahme.

### ✨ Neu
- **Zuverlässigkeit der Befehle im Diagnose-Tab:** Für jede Leuchte wird gezählt, wie viele Befehle gesendet, wie viele bestätigt und wie viele später von der Bridge widerrufen wurden – samt Quote und dem letzten Vorfall im Klartext, etwa `on=false → on=true nach 70 s`. Die Bridge selbst liefert keinerlei Angaben zur Funkqualität, weder Signalstärke noch Verbindungsgüte; diese Messung ersetzt das.
- **Option „Befehl wiederholen" je Gerät:** Sendet jeden Befehl kurz darauf ein zweites Mal. Für Leuchten gedacht, die einzelne Funkbefehle verschlucken. Bewusst pro Gerät einschaltbar und nicht als Standardverhalten – eine Wiederholung kann sonst gegen einen Wandschalter oder die Hue-App arbeiten, wenn die parallel bedient werden. Sie senkt die Fehlerrate deutlich, beseitigt die Ursache aber nicht.
- **Helligkeitsänderungen werden protokolliert.** Bisher landeten sie nur im internen Status; im Log tauchten sie nie auf. Dadurch war „die Leuchte geht an, bleibt aber dunkel" im Nachhinein nicht nachvollziehbar.

### 🐞 Bugfixes
- **Waagrechtes Scrollen auf dem Handy:** Die neue Tabelle im Diagnose-Tab bekam einen eigenen Scrollbereich. Ohne den hätte sich die gesamte Seite verschoben.

### 🧪 Tests
- Abdeckung von 113 auf 135 Tests erweitert. Neu abgedeckt: Zuordnung von Ereignissen zu gesendeten Befehlen samt Zeitfenster und Rundungstoleranz, die Wiederholungslogik einschließlich Abbruch bei einem neueren Befehl, und die Diagnose-Route.

## [2.5.0] - 2026-08-12
**Ereignisverarbeitung & Farbtreue** – Ausgelöst durch die Auswertung von 5000 Logzeilen aus 18 Stunden Dauerbetrieb. Sie brachte zwei Fehler ans Licht, die im Alltag ständig auftraten: der Watchdog startete die Ereignisverbindung 48-mal ohne Grund neu, und dabei gingen vereinzelt Zustandsmeldungen verloren. Dazu ein sauberes Herunterfahren, korrigierte Farbumrechnung und eine abgesicherte Auslieferung.

### 🐞 Bugfixes
- **Watchdog startete die Ereignisverbindung ständig ohne Grund neu:** Die Schwelle lag bei 60 Sekunden ohne Daten. Ein Mitschnitt am Eventstream zeigt, dass die Bridge nur beim Verbindungsaufbau ein Lebenszeichen sendet und danach ausschließlich echte Ereignisse – in ruhigen Phasen ist Stille also völlig normal. Die Folge waren **48 Neustarts in 18 Stunden**, nachts im Takt von 5 Minuten, jeder mit einer vollständigen Neuabfrage aller Geräte. Während des Neuaufbaus war die Bridge jeweils kurz taub. Die Schwelle liegt jetzt bei 5 Minuten.
- **Verlorene Zustandsmeldungen bei großen Ereignissen (Datenverlust):** Ereignisse, die größer als ein Netzwerkpaket sind, treffen in mehreren Stücken ein. Sie wurden bisher stückweise ausgewertet – die erste Hälfte scheiterte als ungültiges JSON, die zweite verschwand kommentarlos. Zweimal in 18 Stunden ist damit ein Zustandswechsel nie bei Loxone angekommen. Die Daten werden jetzt über Paketgrenzen hinweg zusammengesetzt.
- **Ein fehlender Endpunkt riss die gesamte Geräteabfrage ab:** Beim Start werden sieben Ressourcen parallel geladen. Fehlte eine einzige – ältere Firmware, kein Kontaktsensor – ging der komplette Abgleich verloren und der Statusspeicher blieb bis zum ersten Ereignis leer. Im Dashboard blieb aus demselben Grund die Geräteliste leer. Ausfälle werden jetzt einzeln behandelt und protokolliert.
- **Doppelte Ereignisverbindung nach einem Neustart:** Die abgebrochene Verbindung konnte ihrerseits noch einen Neuaufbau anstoßen, während der neue bereits lief – mit doppelten UDP-Paketen an Loxone und doppelten MQTT-Meldungen als Folge. Jede Verbindung kennt jetzt ihre Generation und überholte Verbindungen werden ignoriert.
- **Kein sauberes Herunterfahren:** Es gab keine Behandlung von `SIGTERM`. Jedes `docker stop` beendete den Prozess mitten im Schreibvorgang und ließ die Logdatenbank in einem Zustand zurück, den SQLite beim nächsten Start erst wiederherstellen musste. Server, Ereignisverbindung, MQTT, UDP und Datenbank werden jetzt in fester Reihenfolge geschlossen.
- **Datenordner hing am Arbeitsverzeichnis:** Ein Start aus einem anderen Verzeichnis legte dort einen leeren `data/`-Ordner an – Bridge-IP, App-Key und sämtliche Zuordnungen wirkten verloren. Der Ordner liegt jetzt fest beim Projekt und lässt sich über `DATA_DIR` bewusst umlenken.
- **Farbumrechnung war in sich widersprüchlich:** Hin- und Rückrichtung nutzten Matrizen aus unterschiedlichen Quellen, die nicht zueinander passten. Schwerer wog, dass neutrales Weiß nicht auf dem Weißpunkt D65 landete – **Weiß aus Loxone kam grünstichig an der Lampe an**. Beide Matrizen sind jetzt aus den Primärfarben der Hue-Farblampen (Gamut C) neu berechnet und exakt zueinander invers. Die Sättigung bleibt erhalten, der Grünstich verschwindet.

### 🔧 Auslieferung
- **Die Testsuite läuft jetzt vor jedem Release-Build.** Bisher wurde das Image gebaut und als `latest` veröffentlicht, ohne dass je ein Test gelaufen war.
- **Der Release-Workflow lässt sich von Hand starten.** Bei v2.4.3 erzeugte der Tag-Push keinen Lauf, und ohne manuellen Start blieb nur, den Tag zu löschen und neu zu pushen.

### 🧪 Tests
- Abdeckung von 60 auf 113 Tests erweitert. Neu abgedeckt: Zusammensetzen der Ereignisdaten über Paketgrenzen, Ausfall einzelner Ressourcen, Generationswechsel der Ereignisverbindung, Herunterfahren samt Reihenfolge und Zeitgrenze, Farbmatrizen mit Weißpunkt- und Rundlaufprüfung sowie die Release-Konfiguration selbst.

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