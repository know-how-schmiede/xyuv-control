# XYUV-Control

## 4-Achs-CNC-Controller für Heißdrahtschneider

`XYUV-Control` ist ein eigenständiger CNC-Controller für 4-achsige Heißdrahtschneider.

Das Projekt konzentriert sich ausschließlich auf:

- Maschinensteuerung
- G-Code-Verarbeitung
- Bewegungsplanung
- Stepper-Ansteuerung
- Referenzierung
- Heizdrahtsteuerung
- Jobverwaltung
- Webinterface

Die eigentliche CAM-Software wird als separates Projekt entwickelt. Als möglicher Name dafür ist aktuell **FoamCutPilot** vorgesehen.

---

# 1. Projektziel

Ziel von `XYUV-Control` ist die Entwicklung einer kompakten und eigenständigen CNC-Steuerung für einen 4-achsigen Heißdrahtschneider.

Die Maschine soll insbesondere für das Schneiden von Schaumstoffteilen wie:

- Tragflächen
- Nurflügeln
- Leitwerken
- aerodynamischen Profilen
- allgemeinen Hartschaum- und Styroporbauteilen

eingesetzt werden.

Die Steuerung basiert auf einem:

**Raspberry Pi Pico 2 W**

und verwaltet vier unabhängig angesteuerte Schrittmotorachsen.

Die Achsen werden als:

- **X / Y** – linke Maschinenseite
- **U / V** – rechte Maschinenseite

bezeichnet.

Beide Seiten bewegen jeweils ein Ende des Heißdrahtes. Dadurch können links und rechts unterschiedliche Konturen gleichzeitig abgefahren werden.

---

# 2. Maschinenprinzip

Die Maschine besteht aus zwei vertikalen XY-Portalen.

```text
Linke Seite                         Rechte Seite

       Y                                  V
       ↑                                  ↑
       │                                  │
       ●──────────────────────────────────●
                   Heizdraht
       │                                  │
       └────────→ X                       └────────→ U
```

Die Position des Drahtes wird vollständig durch vier Koordinaten beschrieben:

```text
X Y U V
```

Ein Bewegungsbefehl könnte beispielsweise lauten:

```gcode
G1 X120.0 Y45.0 U95.0 V52.0 F500
```

Alle vier Achsen müssen ihre jeweiligen Zielpositionen synchron erreichen.

---

# 3. Steuerungsplattform

Als zentrale Steuerung ist vorgesehen:

**Raspberry Pi Pico 2 W**

Der Pico übernimmt:

- G-Code-Verarbeitung
- Bewegungsplanung
- 4-Achs-Interpolation
- Stepper-Steuerung
- Referenzfahrt
- Endschalterüberwachung
- Heizdrahtsteuerung
- Jobverwaltung
- WLAN-Kommunikation
- Webinterface

Die Firmware soll überwiegend in **MicroPython** entwickelt werden.

Zeitkritische Funktionen werden über die **PIO-Einheiten des RP2350** realisiert.

Grundstruktur:

```text
MicroPython
     │
     ▼
G-Code Parser
     │
     ▼
Motion Planner
     │
     ▼
Look-Ahead Buffer
     │
     ▼
Segment Buffer
     │
     ▼
PIO Step Generator
     │
     ▼
STEP / DIR
```

MicroPython übernimmt damit hauptsächlich die übergeordnete Logik, während die präzise Impulserzeugung hardwaregestützt erfolgt.

---

# 4. Schrittmotoren

Vorgesehen sind:

**STEPPERONLINE NEMA 17, 42 Ncm, 1,5 A**

Aktuelle Eckdaten:

- Baugröße: NEMA 17
- Schrittwinkel: 1,8°
- 200 Vollschritte/Umdrehung
- Haltemoment: ca. 42 Ncm
- Phasenstrom: ca. 1,5 A
- Abmessungen: ca. 42 × 42 × 39 mm
- bipolar
- 4 Leitungen

Benötigt werden vier Motoren für:

```text
X
Y
U
V
```

Das ausgewählte Set enthält fünf Motoren, sodass ein Motor als Ersatzmotor verwendet werden kann.

Produktlink:

https://link.amazon/B0c90P3gu

---

# 5. Stepper-Treiber

Als Stepper-Treiber sind derzeit **DRV8825-Module** vorgesehen.

Je Achse wird ein eigener Treiber verwendet:

```text
DRV8825 X
DRV8825 Y
DRV8825 U
DRV8825 V
```

Ansteuerung:

```text
STEP
DIR
```

Zusätzlich können berücksichtigt werden:

```text
ENABLE
M0
M1
M2
FAULT
```

`ENABLE` könnte gemeinsam für alle vier Treiber verwendet werden.

Der Motorstrom muss an jedem DRV8825 korrekt eingestellt werden.

Bei höheren Strömen muss auf ausreichende Kühlung geachtet werden.

---

# 6. Microstepping

Der DRV8825 unterstützt Microstepping bis 1/32.

Aktuell wird zunächst:

**1/16 Microstepping**

vorgesehen.

Bei 1,8°-Motoren:

```text
200 × 16 = 3200 Microsteps/Umdrehung
```

Bei einer beispielhaften GT2-Riemenscheibe mit 20 Zähnen:

```text
20 × 2 mm = 40 mm/Umdrehung
```

ergibt sich:

```text
3200 / 40 = 80 Steps/mm
```

Theoretische Auflösung:

```text
0,0125 mm/Microstep
```

Diese Auflösung ist für einen Heißdrahtschneider mehr als ausreichend.

Alternativ kann später auch 1/8 verwendet werden.

---

# 7. Mechanischer Antrieb

Der endgültige mechanische Antrieb ist noch offen.

Aktuell erscheint ein **GT2-Zahnriemenantrieb** besonders interessant.

Prinzip:

```text
NEMA17
   │
GT2-Riemenscheibe
   │
GT2-Zahnriemen
   │
Schlitten
```

Vorteile:

- hohe Geschwindigkeit
- geringe bewegte Masse
- geringe Reibung
- einfache Konstruktion
- ausreichende Genauigkeit

Alternativ können Spindelantriebe untersucht werden.

---

# 8. Motorversorgung

Aktuell wird ein **24-V-System** favorisiert.

Die DRV8825 übernehmen die Stromregelung für die Schrittmotoren.

Die Motoren besitzen einen Nennstrom von etwa:

```text
1,5 A pro Phase
```

Möglicherweise genügt für die Maschine bereits eine Einstellung von ungefähr:

```text
1,0 bis 1,3 A
```

Der optimale Wert wird später experimentell bestimmt.

---

# 9. G-Code als Maschinenformat

`XYUV-Control` soll standardnahen G-Code verarbeiten.

Gründe:

- etablierter CNC-Standard
- menschenlesbar
- einfach zu archivieren
- unabhängig vom CAM
- unabhängig vom Controller
- leicht überprüfbar
- später eventuell kompatibel zu anderen Steuerungen

Beispiel:

```gcode
; XYUV-Control
; Profil: NACA 2412

G21
G90

G28

G0 X120 Y80 U120 V80

M3 S65
G4 P3

G1 X121.20 Y82.40 U121.00 V82.70 F500
G1 X124.70 Y87.10 U124.00 V87.80 F500
G1 X129.30 Y90.50 U128.10 V91.20 F500

M5

G0 X20 Y20 U20 V20

M2
```

---

# 10. Geplanter G-Code-Umfang

Die erste Firmware soll nur einen überschaubaren G-Code-Subset unterstützen.

| Befehl | Funktion |
|---|---|
| G0 | Positionierfahrt |
| G1 | lineare 4-Achs-Bewegung |
| G4 | Wartezeit |
| G21 | Millimeter |
| G90 | absolute Positionierung |
| G91 | relative Positionierung |
| G28 | Referenzfahrt |
| F | Vorschub |
| M3 | Heizdraht einschalten |
| M5 | Heizdraht ausschalten |
| M2 | Programmende |

Zunächst nicht erforderlich:

- Werkzeugwechsel
- Spindelsteuerung
- Kühlmittel
- Fräszyklen
- Werkzeugkorrekturen
- G2/G3-Kreisinterpolation

---

# 11. Trennung von CAM und Controller

Die komplette Geometrieberechnung erfolgt außerhalb von `XYUV-Control`.

Die spätere Web-CAM-Software kennt beispielsweise:

- Wurzelprofil
- Endprofil
- Profiltiefe
- Schränkung
- Abstand der Portale
- Werkstückposition
- Schnittreihenfolge

Das CAM erzeugt daraus konkrete:

```text
X / Y / U / V
```

Koordinaten.

`XYUV-Control` muss keine Tragflächengeometrie kennen.

Die Aufgabe des Controllers lautet:

> Die vom G-Code vorgegebenen X/Y/U/V-Bewegungen präzise und synchron auszuführen.

Als separates Web-CAM-Projekt ist aktuell der Name:

**FoamCutPilot**

vorgesehen.

---

# 12. Bewegungsplanung

Alle vier Achsen müssen synchron interpoliert werden.

Beispiel:

```text
X = 8000 Schritte
Y = 1600 Schritte
U = 6000 Schritte
V = 2800 Schritte
```

Alle Achsen starten gemeinsam und erreichen ihre jeweilige Zielposition gleichzeitig.

Als Verfahren ist eine mehrdimensionale:

**DDA-/Bresenham-artige Interpolation**

vorgesehen.

Darüber hinaus wird ein Look-Ahead-Buffer benötigt.

```text
G-Code
   ↓
Parser
   ↓
Look-Ahead
   ↓
Motion Planner
   ↓
Segment Buffer
   ↓
PIO
   ↓
Motoren
```

Die Maschine darf nicht an jedem einzelnen G-Code-Punkt anhalten.

---

# 13. Besonderheit des Heißdrahtschnitts

Der Heizdraht soll während eines Schnitts möglichst nicht im Material stehen bleiben.

Ein Stillstand würde lokal weiteres Material aufschmelzen und kann verursachen:

- Kerben
- Vertiefungen
- Maßfehler
- beschädigte Profiloberflächen

Ein Schnitt soll daher grundsätzlich:

**vollständig in einem Zug**

durchgeführt werden.

Eine normale Pause-Funktion wird zunächst nicht vorgesehen.

Bei einem kritischen Fehler:

1. Heizdraht AUS
2. Motoren stoppen
3. Job abbrechen
4. Maschine neu referenzieren
5. Schnitt neu starten

---

# 14. Referenzfahrt

Nach dem Einschalten muss die Maschine zunächst referenziert werden.

Danach kennt die Steuerung ihren Maschinenkoordinatenraum.

Beispielsweise:

```text
X = 0
Y = 0
U = 0
V = 0
```

Von dieser Position aus können alle weiteren Positionen reproduzierbar angefahren werden.

---

# 15. Werkstückposition

Die Position des Schaumblocks soll möglichst durch mechanische Anschläge reproduzierbar definiert werden.

Prinzip:

```text
Maschinenreferenz
       ↓
fester Offset
       ↓
Schaumblock
       ↓
Profilposition
```

Damit kann auf ein manuelles Antasten weitgehend verzichtet werden.

---

# 16. Normaler Arbeitsablauf

```text
Maschine einschalten
        ↓
Webinterface im Browser öffnen und anmelden
        ↓
G-Code-Datei bei Bedarf hochladen
        ↓
Referenzfahrt
        ↓
G-Code auswählen
        ↓
Start
        ↓
Startposition automatisch anfahren
        ↓
Heizdraht einschalten
        ↓
Vorheizen
        ↓
Schnitt durchführen
        ↓
Heizdraht ausschalten
        ↓
Parkposition anfahren
```

Langfristiges Ziel:

```text
Datei wählen → START
```

---

# 17. Webinterface

Der Pico 2 W soll ein reduziertes Webinterface bereitstellen.

## Einrichtung und Anmeldung

Die Ersteinrichtung erfolgt per USB. In einer lokalen Konfigurationsdatei
auf dem Pico werden WLAN-SSID, WLAN-Passwort und ein lokaler Benutzer mit
Zugangsdaten für das Webinterface definiert.

Nach einem Neustart übernimmt der Pico die Konfiguration, verbindet sich mit
dem WLAN und stellt das Webinterface bereit. Die erreichbare Adresse wird
über USB ausgegeben. Der Benutzer öffnet diese Adresse im Browser und meldet
sich mit den konfigurierten Zugangsdaten an.

Upload und Maschinensteuerung sind nur nach erfolgreicher Anmeldung möglich.
Die Zugangsbeschränkung wird auch an den Server-Endpunkten geprüft. Format der
Konfigurationsdatei, Passwortspeicherung und Schutz der Anmeldung bei der
Übertragung werden im Firmwarekonzept festgelegt.

## Bedienung im Browser

Nach der Anmeldung kann der Benutzer:

- G-Code-Dateien hochladen und einen Job auswählen
- die Referenzfahrt auslösen
- die Achsen zur Einrichtung per Maus verfahren
- den ausgewählten Job starten und stoppen
- Positionen, Maschinenzustand, Fortschritt und Fehler anzeigen

Ein Upload startet keinen Job. Die Datei wird vor der Ausführung geprüft;
der Start erfolgt ausdrücklich über die Bedienoberfläche. Nach Neustart
oder erneuter Anmeldung wird keine Bewegung automatisch wieder aufgenommen.

Beispiel:

```text
XYUV-Control

Maschine:
● Referenziert
● Endschalter OK
● Draht AUS

Position:
X  120.00
Y   80.00
U  120.00
V   80.00

Job:
NACA2412.nc

[ REFERENZFAHRT ]

[ G-CODE HOCHLADEN ]  [ JOB AUSWÄHLEN ]

[ STARTPOSITION ANFAHREN ]

Heizdraht:
Leistung: 65 %
Vorheizzeit: 3 s

[ START SCHNITT ]

██████████████░░░░ 75 %

[ NOT-STOP ]
```

---

# 18. Handsteuerung / Jog

Eine vollständige manuelle Bedienung ist für den normalen Betrieb nicht notwendig.

Eine einfache Jog-Funktion soll jedoch im Bereich:

**Service / Einrichtung**

vorhanden sein.

Die Bedienung erfolgt nach Anmeldung über Schaltflächen im Webinterface.
Jeder Mausklick löst zunächst einen begrenzten Verfahrschritt aus; die
zulässigen Verfahrwege und Maschinenzustände werden vor der Bewegung geprüft.

Beispielsweise:

```text
X ±1 mm
Y ±1 mm
U ±1 mm
V ±1 mm

optional:
±10 mm
```

Verwendung:

- Aufbau
- Wartung
- Kalibrierung
- Drehrichtung prüfen
- Endschalter testen
- Mechanik überprüfen

---

# 19. Heizdraht

Als aktueller Kandidat ist vorgesehen:

**WhaleO Schneidedraht Styropor**

Eigenschaften laut Produktbeschreibung:

- Nickel-Chrom-Heizdraht
- Durchmesser: **0,7 mm**
- Länge der Rolle: **10 m**
- vorgesehen zum Heißschneiden von Styropor und Hartschaum

Produktlink:

https://link.amazon/B05TdTHjj

Der Draht wird zunächst als Test- und Entwicklungskandidat betrachtet.

Die endgültige Eignung hängt insbesondere ab von:

- tatsächlichem spezifischem Widerstand
- verwendeter Drahtlänge
- gewünschter Schneidetemperatur
- erforderlichem Strom
- erforderlicher Spannung
- mechanischer Vorspannung
- thermischer Ausdehnung

---

# 20. Elektrische Auslegung des Heizdrahtes

Für die Dimensionierung müssen noch folgende Größen bestimmt werden:

```text
Drahtmaterial: Nickel-Chrom
Durchmesser:   0,7 mm
Länge:         abhängig von Maschinenbreite
```

Darauf basierend werden später bestimmt:

```text
Widerstand R
Strom I
Spannung U
Leistung P
```

Grundlagen:

```text
R = ρ × l / A
```

und:

```text
P = U × I
```

beziehungsweise:

```text
P = I² × R
```

Da die genaue NiCr-Legierung des Produktes noch nicht abschließend bekannt ist, soll der Widerstand pro Meter vorzugsweise praktisch gemessen werden.

---

# 21. Heizdraht-Leistungsstufe

Für die Heizdrahtsteuerung ist aktuell ein:

**25-A-Heizbett-MOSFET-Modul**

aus dem 3D-Druck-Bereich vorgesehen.

Produktlink:

https://link.amazon/B04MTLvCB

Prinzip:

```text
Pico 2 W
    │
    │ PWM
    ▼
25-A-MOSFET-Modul
    │
    ▼
Heizdraht
```

Das Modul übernimmt ausschließlich die Leistungsstufe.

Der Pico erzeugt das Steuersignal.

---

# 22. Alternative MOSFET-Leistungsstufe

Als alternative Leistungsstufe wurde folgendes Modul betrachtet:

**DC 5–36 V, 400 W FET Trigger Switch Drive Module**

Produktlink:

https://link.amazon/B07ElQg3q

Diese Variante bleibt als Alternative beziehungsweise Testmodul dokumentiert.

Aktuell wird jedoch die robustere Heizbett-MOSFET-Bauform bevorzugt.

---

# 23. PWM-Heizdrahtregelung

Der Draht soll nicht nur EIN/AUS geschaltet, sondern über PWM geregelt werden.

Beispiel:

```gcode
M3 S65
```

bedeutet:

```text
Heizdraht EIN
Leistung 65 %
```

```gcode
M5
```

bedeutet:

```text
Heizdraht AUS
```

Die tatsächliche PWM-Frequenz wird später experimentell festgelegt.

---

# 24. Fail-Safe-Verhalten des Heizdrahtes

Der Heizdraht muss bei:

- Reset
- Firmwareabsturz
- Bootvorgang
- Kommunikationsfehler
- Not-Aus

automatisch ausgeschaltet bleiben beziehungsweise ausgeschaltet werden.

Der MOSFET-Steuereingang erhält daher einen definierten Hardwarezustand.

Beispielsweise:

```text
Pico GPIO
    │
    ├────── MOSFET Trigger
    │
   10 kΩ
    │
   GND
```

Bei inaktivem oder hochohmigem Pico-Ausgang bleibt die Heizung damit AUS.

---

# 25. Spannungsversorgung

Aktuell wird ein gemeinsames:

**24-V-Netzteil**

favorisiert.

Grundstruktur:

```text
                    24-V-Netzteil
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
       Stepper-Zweig           Heizdraht-Zweig
             │                       │
       4 × DRV8825               Sicherung
             │                       │
       4 × NEMA17                   ▼
                             MOSFET-Leistungsmodul
                                     │
                                     ▼
                                 Heizdraht

                         │
                         ▼
                     DC/DC 5 V
                         │
                         ▼
                     Pico 2 W
```

Ob ein einziges 24-V-Netzteil für Motoren und Heizdraht sinnvoll dimensioniert werden kann, wird nach der Heizdrahtberechnung entschieden.

Alternativ könnten beide Leistungsbereiche getrennte Netzteile erhalten.

---

# 26. Heizdraht-Absicherung

Der Heizdrahtzweig erhält eine eigene Sicherung.

Diese wird nach dem tatsächlichen Betriebsstrom dimensioniert.

Zu berücksichtigen sind:

- Sicherungswert
- Leitungsquerschnitt
- MOSFET-Belastbarkeit
- Klemmen
- Steckverbinder
- Kühlung
- Netzteilleistung

---

# 27. Gemeinsame Masse

Bei Verwendung nicht galvanisch getrennter MOSFET-Module müssen Steuerung und Leistungsteil einen gemeinsamen Bezug besitzen.

```text
Pico GND
   │
   ├──────────── GND
   │
MOSFET GND
   │
24-V-GND
```

Die genaue Masseführung wird später im Schaltplan definiert.

---

# 28. G-Code-Speicherung

G-Code-Dateien werden zunächst im internen Flash des Pico gespeichert.

Sie werden nicht komplett in den RAM geladen.

Prinzip:

```text
Flash
 ↓
G-Code-Leser
 ↓
Parser
 ↓
Look-Ahead
 ↓
Motion Buffer
 ↓
PIO
```

Damit hängt die maximale Jobgröße hauptsächlich vom verfügbaren Flash und nicht vom RAM ab.

---

# 29. Dateiübertragung

Primärer Übertragungsweg:

**WLAN**

```text
CAM
 │
 │ WLAN
 ▼
Pico Webserver
 │
 ▼
Upload
 │
 ▼
interner Flash
 │
 ▼
Jobausführung
```

Der komplette Job befindet sich vor dem Start auf dem Pico.

Der Upload erfolgt über das Webinterface nach Anmeldung mit dem lokalen
Benutzer. Eine erfolgreich übertragene Datei wird als auswählbarer Job
gespeichert und erst durch einen gesonderten Startbefehl ausgeführt.

Ein WLAN-Ausfall während des Schnitts beeinflusst damit den laufenden Job nicht.

---

# 30. USB

USB dient zunächst für:

- Firmwareinstallation
- Update
- WLAN-Konfiguration und Einrichtung des lokalen Benutzers
- Debugging
- Service

Konfigurationsänderungen werden nach einem Neustart übernommen. USB bleibt
für Konfiguration und Diagnose nutzbar, wenn WLAN-Zugangsdaten falsch sind
oder keine WLAN-Verbindung hergestellt werden kann.

Ein USB-Stick für G-Code wird für Version 1 nicht vorgesehen.

---

# 31. Optionale microSD-Erweiterung

Eine spätere microSD-Erweiterung kann vorgesehen werden.

Beispiel:

```text
/jobs
    NACA2412.nc
    E205.nc
    Nurfluegel_L.nc
    Nurfluegel_R.nc
```

Die erste Version soll jedoch ohne SD-Karte funktionieren.

---

# 32. Sicherheitsfunktionen

Vorgesehen sind:

- Referenzschalter
- Software-Endlagen
- G-Code-Grenzprüfung
- Heizdraht-Sicherung
- Not-Aus
- Heizdraht sofort AUS bei Fehler
- Jobabbruch bei kritischem Fehler
- definierter AUS-Zustand beim Booten

Vor dem Jobstart werden mindestens die Maschinenkoordinaten geprüft:

```text
0 ≤ X ≤ Xmax
0 ≤ Y ≤ Ymax
0 ≤ U ≤ Umax
0 ≤ V ≤ Vmax
```

---

# 33. Stromausfall und Jobabbruch

Nach Verlust der Versorgung ist die reale Maschinenposition nicht mehr sicher bekannt.

Daher zunächst kein Resume.

```text
Fehler / Stromausfall
        ↓
Heizdraht AUS
        ↓
Motoren stoppen
        ↓
Job ungültig
        ↓
neu referenzieren
        ↓
Job neu starten
```

---

# 34. Vorgesehene Firmwarestruktur

```text
firmware/
│
├── main.py
├── config.py
│
├── cnc/
│   ├── planner.py
│   ├── interpolator.py
│   ├── motion.py
│   ├── homing.py
│   └── limits.py
│
├── gcode/
│   ├── parser.py
│   └── commands.py
│
├── hardware/
│   ├── pio_stepper.py
│   ├── drivers.py
│   ├── endstops.py
│   ├── wire_control.py
│   └── emergency_stop.py
│
├── jobs/
│   └── job_manager.py
│
└── network/
    ├── wifi.py
    ├── webserver.py
    └── api.py
```

Diese Struktur ist zunächst nur ein Entwurf.

---

# 35. Aktuelle Systemarchitektur

```text
                    CAM-System
                        │
                 Profilberechnung
                        │
                        ▼
                     G-Code
                    X Y U V
                        │
                       WLAN
                        │
                        ▼
┌──────────────────────────────────────┐
│ XYUV-Control                         │
│ Raspberry Pi Pico 2 W                │
│                                      │
│ Webinterface                         │
│      ↓                               │
│ Jobverwaltung                        │
│      ↓                               │
│ G-Code Parser                        │
│      ↓                               │
│ Look-Ahead Planner                   │
│      ↓                               │
│ 4-Achs-Interpolation                 │
│      ↓                               │
│ Segment Buffer                       │
│      ↓                               │
│ PIO Step Generator                   │
│                                      │
│ PWM-Heizdrahtsteuerung               │
│ Endschalter                          │
│ Not-Aus                              │
└────┬────────┬────────┬────────┬──────┘
     │        │        │        │
  DRV8825  DRV8825  DRV8825  DRV8825
     │        │        │        │
     X        Y        U        V
     │        │        │        │
  NEMA17   NEMA17   NEMA17   NEMA17

                    PWM
                     │
                     ▼
             25-A-MOSFET-Modul
                     │
                     ▼
              0,7-mm-NiCr-Draht
```

---

## 35.1 Eigene Controllerplatine

Für XYUV-Control wird eine eigene Controllerplatine als Träger für den
Raspberry Pi Pico 2 W und vier DRV8825-Treibermodule geplant. Die vorhandene
externe MOSFET-Leistungsstufe für den Schneiddraht bleibt über einen
Steueranschluss angebunden. Der Heizdraht-Leistungsstrom wird zunächst nicht
über die Controllerplatine geführt.

### Bestückung und Anschlüsse

- Pico 2 W mit zugänglichem USB-Anschluss und BOOTSEL-Taster; für den
  Prototyp sind austauschbare Steckmodule vorgesehen.
- Vier eindeutig orientierte DRV8825-Steckplätze für X/Y/U/V mit Platz für
  Kühlkörper, Luftführung und Zugang zur Motorstromeinstellung.
- Vier ausreichend strombelastbare Motoranschlüsse mit Beschriftung der
  Wicklungspaare und Achsen.
- Vier Referenzschalteranschlüsse mit definierten 3,3-V-Eingangspegeln,
  Pull-Widerständen, Entstörung und Schutz gegen Störungen auf langen Leitungen.
  Öffnerkontakte werden bevorzugt; Verhalten bei Kabelbruch wird festgelegt.
  Aktive Sensoren mit 5 V oder 24 V benötigen eine passende Eingangsschaltung.
- Anschluss für Schneiddraht-PWM und Signalmasse zur externen Leistungsstufe.
  Die erforderlichen Pegel werden am konkreten MOSFET-Modul geprüft; bei
  Bedarf wird eine Pegelanpassung vorgesehen. Ein Hardware-Pulldown hält
  die Heizung bei inaktivem Pico ausgeschaltet.
- Anschluss für eine unabhängige Not-Aus-Abschaltung von Motor- und
  Heizleistung sowie ein Eingang zur Rückmeldung ihres Zustands. Die
  Leistungsabschaltung erfolgt über eine geeignet dimensionierte externe
  Schaltung und ist unabhängig von Firmware und Webinterface.
- Status-LEDs mit Vorwiderständen für Versorgung, Bereitschaft/WLAN, Job
  und Fehler. Eine Heizungs-LED zeigt ohne zusätzliche Messung nur den
  Steuerbefehl, nicht den tatsächlichen Stromfluss an.

### Versorgung und Schutz

- Versorgungseingang für die geplanten 24 V mit passender Sicherung,
  Verpolschutz und dimensioniertem Schutz gegen Spannungsspitzen.
- DC/DC-Abwärtswandler von 24 V auf 5 V für den Pico und gegebenenfalls
  weitere Verbraucher; Stromreserve, Wärmeentwicklung und Störverhalten
  werden bei der Bauteilauswahl berücksichtigt.
- Versorgung des Pico über VSYS mit Entkopplung zur gleichzeitigen
  USB-Versorgung, beispielsweise über eine zusätzliche Schottky-Diode
  oder die im Pico-Datenblatt beschriebene MOSFET-Schaltung. Die externe
  Versorgung darf weder USB noch den DC/DC-Wandler rückwärts speisen.
  24 V dürfen nicht an VSYS oder GPIO gelangen.
- Lokale Abblock- und ausreichend dimensionierte Pufferkondensatoren an
  den Treiberversorgungen; Bestückung auf den gewählten Modulen prüfen.
- Definierte Zustände für ENABLE, RESET und SLEEP: Motoren bleiben beim
  Einschalten und Reset zunächst deaktiviert. Microstepping wird über
  Jumper oder feste Beschaltung eingestellt; FAULT-Rückmeldungen werden
  nach Prüfung der Modulbeschaltung ausgewertet.
- Leistungs- und Signalstrompfade so führen, dass Motorströme keine
  störenden Spannungsabfälle in der Pico- oder Schaltermasse erzeugen.
  Leiterbahnen, Stecker und Sicherungen werden nach tatsächlichem Strom
  und thermischen Bedingungen dimensioniert.

### Layout, Service und Erweiterungen

- Antennenbereich des Pico am Platinenrand freihalten; Ausschnitt und
  Abstände nach Pico-2-W-Datenblatt berücksichtigen.
- Befestigungsbohrungen, eindeutige Steckerkennzeichnung, Pin-1-Markierungen
  und Platinenrevision vorsehen.
- Testpunkte für Versorgung, Masse, STEP/DIR, ENABLE und Heizungs-PWM sowie
  Zugang zu SWD und RUN/Reset vorsehen.
- GPIO-Belegung vor dem Layout vollständig planen, einschließlich LEDs,
  Schaltern, Fehlerleitungen und Reserveanschlüssen. Bei Bedarf LEDs oder
  langsame Statussignale über einen I/O-Expander anbinden.
- Optional: Lüfteranschluss, weitere Endschalter, Strom-/Temperaturmessung
  und Drahtbrucherkennung. Diese Erweiterungen sind für die erste Platine
  noch nicht verbindlich festgelegt.

Die Funktionsliste ist damit als Grundlage für den Schaltplan festgelegt.
Vor Fertigung werden Modul-Pinbelegung, Schutzschaltungen, Not-Aus-Konzept,
GPIO-Budget, Versorgung und thermische Auslegung geprüft. Der erste
Platinenprototyp wird zunächst ohne beheizten Draht in Betrieb genommen.

Technische Grundlagen:

- [Raspberry Pi Pico 2 W Datasheet](https://datasheets.raspberrypi.com/picow/pico-2-w-datasheet.pdf)
- [TI DRV8825 Datasheet](https://www.ti.com/lit/ds/symlink/drv8825.pdf)

---

# 36. Teile- und Bestellliste

Die folgende Liste dokumentiert den aktuellen Planungsstand und ist noch keine endgültige Stückliste.

## 36.1 Raspberry Pi Pico 2 W

**Anzahl:** 1

Aufgaben:

- zentrale Steuerung
- WLAN
- G-Code
- Webinterface
- Bewegungsplanung
- PIO-Step-Erzeugung
- Heizdrahtregelung

Bezugslink:

Noch offen.

---

## 36.2 Schrittmotoren

### STEPPERONLINE NEMA 17 – 42 Ncm / 1,5 A / 42 × 39 mm

**Anzahl:** 1 Pack mit 5 Motoren

Verwendung:

- X
- Y
- U
- V
- 1 × Reserve

Link:

https://link.amazon/B0c90P3gu

---

## 36.3 Stepper-Treiber

### DRV8825 Stepper-Treibermodule

**Anzahl:** mindestens 4

Empfehlung:

```text
4 benötigt
+ 1 bis 2 Reserve
```

Bezugslink:

[Vorgesehene Treibermodule bei Amazon](https://link.amazon/B02MzzIzi)

Produktdaten und genaue Modul-Pinbelegung sind vor dem PCB-Layout noch zu
prüfen; der Kurzlink konnte beim Eintragen nicht abgerufen werden.

---

## 36.4 Heizdraht

### WhaleO Nickel-Chrom-Schneidedraht

**Anzahl:** 1 Rolle

Daten:

- NiCr
- 0,7 mm
- 10 m

Link:

https://link.amazon/B05TdTHjj

---

## 36.5 Heizdraht-Leistungsmodul – bevorzugt

### 25-A-MOSFET-Heizbettmodul

**Anzahl:** 1

Das angebotene Set enthält zwei Module, sodass eines als Reserve verwendet werden kann.

Verwendung:

- PWM-Leistungsregelung des Heizdrahtes

Link:

https://link.amazon/B04MTLvCB

---

## 36.6 Alternative Heizdraht-Leistungsstufe

### DC 5–36 V / 400 W FET Trigger Switch

**Status:** Alternative / Test

Link:

https://link.amazon/B07ElQg3q

---

## 36.7 Noch zu beschaffende elektrische Komponenten

Noch nicht konkret ausgewählt:

- 24-V-Netzteil
- DC/DC-Wandler 24 V → 5 V
- Sicherung für Heizdraht
- Sicherungshalter
- Hauptschalter
- Not-Aus
- Endschalter
- Verkabelung
- Motorstecker
- Heizdrahtanschlüsse
- Kühlkörper für DRV8825
- eventuell Lüfter
- Verteilerklemmen
- Controllerplatine mit Pico- und Treibersockeln
- Bauteile für Versorgungsschutz, USB-Entkopplung und Pufferung
- Referenzschalter-Eingangsbeschaltung und Status-LEDs
- Not-Aus-Rückmeldeanschluss und externe Leistungsabschaltung

---

## 36.8 Noch zu beschaffende mechanische Komponenten

Noch offen:

- GT2-Zahnriemen
- GT2-Riemenscheiben
- Umlenkrollen
- Linearführungen
- Lager
- Achsschlitten
- Rahmenmaterial
- Drahtspannmechanismus
- Federn oder Gewichte für Heizdrahtspannung
- Werkstückanschläge

---

# 37. Aktuell festgelegte beziehungsweise favorisierte Hardware

| Komponente | Aktueller Stand |
|---|---|
| Projektname | XYUV-Control |
| Controller | Raspberry Pi Pico 2 W |
| Controllerplatine | Eigene Trägerplatine für Pico und vier DRV8825-Module geplant |
| Achsen | X / Y / U / V |
| Motoren | NEMA 17, 42 Ncm, 1,5 A |
| Anzahl Motoren | 4 + 1 Reserve |
| Stepper-Treiber | DRV8825 |
| Microstepping | zunächst 1/16 |
| Mechanik | GT2 aktuell favorisiert |
| Motorspannung | voraussichtlich 24 V |
| Heizdraht | NiCr, 0,7 mm |
| Heizdraht-Leistungsstufe | 25-A-Heizbett-MOSFET |
| Heizdrahtregelung | PWM |
| Datenformat | G-Code |
| Dateiübertragung | WLAN |
| Firmware | MicroPython + PIO |

---

# 38. Noch zu definierende Punkte

## Firmware

- G-Code-Dialekt
- Parser
- Look-Ahead
- Beschleunigungsprofil
- Feedrate-Definition für vier Achsen
- PIO-Konzept
- Buffergrößen
- maximale Step-Frequenz
- Fehlerzustände
- Statusmodell

## Controllerplatine

- Schaltplan und GPIO-Belegung
- genaue DRV8825-Modulvariante und Steckplatzbelegung
- Steckertypen, Pinbelegung und Strombelastbarkeit
- DC/DC-Wandler, Versorgungsschutz und USB-Entkopplung
- Referenzschalterbeschaltung und unterstützte Sensortypen
- Not-Aus-Leistungsabschaltung und Rückmeldung
- Kühlung, Leiterbahnauslegung und Antennenfreiraum
- Status-LED-Belegung und optionale Erweiterungsanschlüsse
- Platinenabmessungen und Befestigung

## Mechanik

- Verfahrwege
- Portalabstand
- Maschinenbreite
- maximale Drahtlänge
- Riemenantrieb
- Riemenscheiben
- Steps/mm
- Beschleunigung
- Höchstgeschwindigkeit
- Referenzschalter
- Parkposition

## Heizdraht

- tatsächlicher Widerstand pro Meter
- benötigte Drahtlänge
- optimaler Strom
- Betriebsspannung
- Leistung
- Schneidetemperatur
- PWM-Bereich
- Vorheizdauer
- Drahtspannung
- thermische Längenausdehnung
- Drahtbrucherkennung
- Strommessung

## Stromversorgung

- benötigte Heizdrahtleistung
- benötigter Gesamtstrom
- Netzteilleistung
- Sicherungen
- Kabelquerschnitte
- Kühlung
- Masseführung

## Webinterface

- Format der per USB gepflegten Konfigurationsdatei
- Passwortspeicherung und Schutz der Anmeldung bei der Übertragung
- Sitzungsdauer und Regeln für mehrere gleichzeitig geöffnete Browser
- Dateigrößenlimit und Speicherverwaltung für Uploads
- Upload
- Jobauswahl
- Referenzfahrt
- Start
- Status
- Fortschritt
- Fehler
- Service-Jog
- Konfiguration

## CAM-Schnittstelle

- Koordinatensystem
- G-Code-Konvention
- Vorschubdefinition
- Startposition
- Parkposition
- Heizdrahtbefehle
- Metadaten

---

# 39. Projektstatus

Der aktuelle technische Stand von `XYUV-Control` lautet:

```text
Raspberry Pi Pico 2 W
+
MicroPython
+
PIO
+
4 × DRV8825
+
4 × NEMA17
+
X/Y/U/V
+
GT2-Antrieb wahrscheinlich
+
0,7-mm-NiCr-Heizdraht
+
25-A-MOSFET-Leistungsstufe
+
24-V-System wahrscheinlich
+
PWM-Heizdrahtregelung
+
G-Code
+
WLAN-Dateiübertragung
+
Webinterface
+
automatischer Schnittablauf
```

Diese Projektbeschreibung bildet die aktuelle Arbeitsgrundlage.

Die einzelnen Bereiche werden im weiteren Projektverlauf schrittweise spezifiziert, getestet und gegebenenfalls angepasst.
