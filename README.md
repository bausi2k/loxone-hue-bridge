# Loxone Hue Bridge 🇦🇹

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://www.buymeacoffee.com/bausi2k)

**Loxone Hue Bridge** ist eine bidirektionale Schnittstelle zwischen dem **Loxone Miniserver**, der **Philips Hue Bridge (V2 / API)** und optional **MQTT**.

Sie ermöglicht eine extrem schnelle, lokale Steuerung ohne Cloud-Verzögerung und nutzt die moderne Hue Event-Schnittstelle (SSE), um Statusänderungen in Echtzeit an Loxone (UDP) und MQTT Broker zurückzumelden.

## 🚀 Features V2.3.0

* **Nativer „Alles" Befehl:** Nutzt die Hue `bridge_home` API für blitzschnelles Ausschalten des gesamten Hauses.
* **Hue Effekte & Alert:** Steuere Lampen mit atmosphärischen Effekten direkt aus Loxone:
    * `/{name}/alert` → Einmaliges Blinken (Alarmmeldung, Türklingel)
    * `/{name}/candle` / `/fire` / `/prism` / `/sparkle` → Persistente Atmosphäre-Effekte
    * `/{name}/noeffect` → Effekt stoppen
    * `/{name}/sunrise/30` → 30-Sekunden Sonnenaufgang (oder beliebige Dauer)
* **Modularer Kern:** Hochperformante und wartbare Backend-Architektur durch saubere Modul-Trennung (`lib/`).
* **Smart Setup:** Automatische Suche der Hue Bridge und Pairing per Web-Interface.
* **Live Dashboard:** Echtzeit-Anzeige aller Lichter (mit Live-Werten für Kelvin/Hex/Dim), Sensoren und Batterieständen (inkl. Warnsystem bei ≤ 10 %).
* **Smart Mapping:** Einfache Zuordnung per „Klick & Wähl" mit automatischer Duplikatsfilterung bei erkannten Befehlen.
* **Erweiterter Diagnose-Tab:** Zeigt Gerätestatus, Zigbee-Konnektivität pro Gerät und eine vollständige Übersicht aller Lampen-Fähigkeiten (Dimmen, Farbe, Weißton, unterstützte Effekte).
* **Automatisierte Tests:** Maximale Zuverlässigkeit durch eine Test-Infrastruktur mit 16 Tests und > 85 % Code-Coverage.
* **Persistent Logging (SQLite):** Dank nativer SQLite-Datenbank bleiben Logs auch nach Neustarts erhalten und sind extrem performant durchsuchbar (Volltextsuche & Filter).
* **Backup & Restore:** Lade deine komplette Konfiguration inkl. Mappings als Backup herunter und stelle sie bei Bedarf wieder her.
* **Loxone Integration:**
    * **Steuern:** Schalten, Dimmen, Warmweiß & RGB (via Virtueller Ausgang).
    * **Empfangen:** Bewegung, Taster, Helligkeit, Temperatur, Batterie (via UDP Eingang).
* **MQTT Support:** Sendet alle Statusänderungen parallel an einen MQTT Broker (z.B. für Home Assistant, ioBroker).
* **Stabilität:** Integrierter Watchdog überwacht die Verbindung und eine intelligente Queue verhindert Überlastung der Bridge (Error 429).
* 🎛️ **Individuelle Geräte-Steuerung:** Detaillierte Einstellungen pro Gerät direkt im UI (z. B. Deaktivieren von Überblendzeiten/Dynamics für Drittanbieter-Relais).
* 📊 **Smartes Dashboard:** Live-Status aller Geräte, Batteriewarnungen und komfortable System-Konfiguration per Web-Interface.

---

## 📋 Voraussetzungen

* Philips Hue Bridge (V2, eckiges Modell)
* Loxone Miniserver
* Ein Server für Docker (z.B. Raspberry Pi, Synology, Unraid)
* *Nur bei manueller Installation:* Node.js 24+

---

## 🛠 Installation (Empfohlen: Docker)

Du musst keinen Code mehr bauen. Du brauchst nur Docker und eine `docker-compose.yml`.

1. **Ordner erstellen:**
   Erstelle einen Ordner (z.B. `loxone-hue-bridge`) auf deinem Server.

2. **Datei erstellen:**
   Erstelle darin eine `docker-compose.yml` mit folgendem Inhalt:

   ```yaml
   services:
     loxhuebridge:
       image: ghcr.io/bausi2k/loxone-hue-bridge:latest
       container_name: loxhuebridge
       restart: always
       
       # WICHTIG für Loxone UDP Kommunikation
       network_mode: "host"
       
       environment:
         - TZ=Europe/Vienna
         
       volumes:
         # Nur noch der Data-Ordner ist wichtig
         - ./data:/app/data
