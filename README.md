# SenseCAP D1 - Loxone & LoxBerry Integration via OpenHASP 🚀

Dieses Repository liefert eine vollständige Anleitung und die benötigten Dateien, um ein **SenseCAP D1** Touch-Display (geflasht mit OpenHASP) nahtlos in ein **Loxone Smart Home** zu integrieren. Die Kommunikation läuft lokal und blitzschnell über einen **LoxBerry** (MQTT Gateway).

Das perfekte, hochgradig individualisierbare und kostengünstige Smart-Home-Display – ideal für den Schreibtisch, die Küche oder das Wohnzimmer!

## 🌟 Features dieses Setups
* **Minimalistisches UI-Design:** Riesige Darstellung der Außentemperatur und eine farblich abgesetzte Box für die Warmwassertemperatur der Wärmepumpe.
* **Smarte Loxone-Helligkeitssteuerung:** Das Display wird nicht über einen starren internen Timer gedimmt, sondern intelligent über Loxone gesteuert (Tagsüber 100% an, Nachts nur aktiv, wenn Bewegung/Licht im Raum erkannt wird).
* **100% Lokal:** Keine Cloud-Abhängigkeit, volle Kontrolle durch Loxone und MQTT.

---

## 🛠️ Voraussetzungen
1. **SenseCAP D1** (oder ein ähnliches ESP32-Display), geflasht mit [OpenHASP](https://openhasp.com/).
2. **LoxBerry** mit installiertem "MQTT Gateway" Plugin.
3. **Loxone Miniserver**.
4. Datenquellen in Loxone (z.B. Außentemperatur und Warmwassertemperatur, in unserem Fall via HeishaMon von einer Panasonic Wärmepumpe).

---

## ⚙️ Schritt 1: OpenHASP Display Layout konfigurieren
Zuerst laden wir das Design auf das Display. Die fertige Layout-Datei findest du hier im Repository.

1. Lade dir die Datei `pages.jsonl` aus diesem Repository herunter.
2. Öffne die Weboberfläche deines SenseCAP D1.
3. Navigiere zu **File Editor**.
4. Lade die heruntergeladene `pages.jsonl` hoch (oder öffne die bestehende Datei und ersetze den Inhalt mit dem aus diesem Repo) und klicke auf **Save**.
   *(Hinweis: Falls das Display nach dem Neustart die große Schriftart für die Außentemperatur nicht lädt, ändere in der Datei den Wert `"text_font":120` auf `"text_font":100`.)*

---

## 📡 Schritt 2: MQTT & System-Einstellungen im Display
Das Display muss nun mit dem LoxBerry verbunden werden und seine Eigensteuerung abgeben.

1. Gehe in der Weboberfläche zu **Configuration -> MQTT**.
2. Trage die IP-Adresse, den Port (meist `1883`), sowie Benutzername und Passwort deines LoxBerry MQTT Brokers ein. 
3. Notiere dir den **Node Name** (in dieser Anleitung nutzen wir den Namen `sensecap`).
4. Gehe zu **Configuration -> Display** und setze **Short Idle** und **Long Idle** auf `0`. (Damit übergeben wir die Helligkeitssteuerung komplett an Loxone).
5. Starte das Display neu (**System -> Restart**).

---

## 🟢 Schritt 3: Loxone Konfiguration (Werte senden)
Nun bringen wir Loxone bei, die Temperaturwerte bei jeder Änderung an das Display zu schicken.

1. Lege in der Loxone Config einen **Virtuellen Ausgang** an:
   * **Adresse:** `/dev/udp/<IP-DEINES-LOXBERRYS>/11884` *(Port des LoxBerry MQTT Gateways prüfen!)*
2. Lege darunter zwei **Virtuelle Ausgang Befehle** an. Wichtig: Den Haken bei "Als Digitalausgang verwenden" **entfernen**.

**Befehl 1: Außentemperatur**
* Befehl bei EIN: `hasp/sensecap/command/p1b10.text <v> °C`

**Befehl 2: Warmwasser**
* Befehl bei EIN: `hasp/sensecap/command/p1b11.text <v> °C`

3. Verbinde in der Loxone Programmierung die entsprechenden analogen Temperatur-Ausgänge mit diesen Befehls-Bausteinen.

---

## 💡 Schritt 4: Smarte Display-Beleuchtung via Loxone
Damit das Display nachts nicht stört, steuern wir die Helligkeit aktiv aus Loxone heraus.

1. Erstelle einen weiteren **Virtuellen Ausgang Befehl** unter dem bestehenden UDP-Ausgang. Nenne ihn "Display Backlight".
2. Setze hier den Haken bei **Als Digitalausgang verwenden**!
3. Trage folgende Befehle ein:
   * **Befehl bei EIN:** `hasp/sensecap/command/backlight {"state":"on","brightness":255}`
   * **Befehl bei AUS:** `hasp/sensecap/command/backlight {"state":"off"}`
4. **Die Logik in Loxone:** * Füge einen **ODER**-Baustein ein. 
   * Verbinde auf den ersten Eingang den Loxone-Baustein **Tageslicht**. 
   * Verbinde auf den zweiten Eingang den Status der **Raumbeleuchtung** (oder eines Präsenzmelders in der Küche). 
   * Verbinde den Ausgang des ODER-Bausteins mit deinem neuen "Display Backlight" Befehl.
   * *Ergebnis:* Das Display ist von Sonnenauf- bis Sonnenuntergang aktiv. Nachts schaltet es sich nur ein, wenn man den Raum betritt bzw. das Licht einschaltet.

---

> **Ein Projekt von lox-config.de** > Du benötigst Hilfe bei der Loxone-Programmierung oder möchtest dieses Setup als fertige Lösung in dein Smart Home integrieren? Besuche uns auf [lox-config.de](https://lox-config.de) – deinem Systemintegrator für maßgeschneiderte Loxone-Projekte (Hardware-Einbau in der Oberpfalz & Programmierung deutschlandweit).
