# Entwicklungsplan: xyuv-control

![xyuv-control – X/Y/U/V Heißdraht-CNC](images/xyuv-control-banner.png)



## Ziel und Ausgangspunkt

xyuv-control soll eine CNC-Steuerung für einen Heißdraht-Schaumschneider mit
vier Achsen (X/Y/U/V) werden. Als Plattform sind Raspberry Pi Pico 2 W,
MicroPython und G-Code vorgesehen. Dieses Dokument beschreibt die geplante
Entwicklung; es dokumentiert noch keine implementierten Funktionen.

Die Entwicklung beginnt mit einem einachsigen Testaufbau. Damit werden
Elektronik, Bewegungssteuerung und Fehlerbehandlung zunächst an einem Motor
geprüft. Anschließend wird die Steuerung schrittweise auf vier Achsen erweitert.
Meilensteine und Versionsprotokolle stehen in [TIMELINE.md](TIMELINE.md).

## Phase 1: Einachsigen Testaufbau vorbereiten

- Einen Schrittmotor mit passendem STEP/DIR-Treiber, Motorversorgung und
  mechanischer Testachse auswählen und dokumentieren.
- Pinbelegung für STEP, DIR, ENABLE und einen Referenzschalter festlegen;
  Logikpegel, gemeinsame Masse und Treiberanschluss prüfen.
- Verfahrweg, Schritte pro Millimeter, maximale Geschwindigkeit und
  Beschleunigung zunächst konservativ konfigurieren.
- Eine kabelgebundene Bedien- und Diagnoseverbindung vorsehen, beispielsweise
  über USB. WLAN ist für die ersten Bewegungstests keine Voraussetzung.
- Eine vom Programm unabhängige Möglichkeit zum Abschalten der Motorversorgung
  vorsehen. Die Tests beginnen ohne beheizten Draht.

**Abschlusskriterium:** Der Aufbau und seine Anschlussbelegung sind nachvollziehbar
dokumentiert; der Treiber bleibt beim Start deaktiviert und kann sicher
abgeschaltet werden.

## Phase 2: Eine Achse zuverlässig bewegen

- STEP/DIR-Ausgabe zunächst bei niedriger Geschwindigkeit prüfen. Die konkrete
  Umsetzung, etwa mit PIO, wird anhand von Messungen gewählt.
- Positive und negative Verfahrbewegungen, Geschwindigkeitsbegrenzung und
  Beschleunigungsrampen implementieren.
- Positionsführung, Referenzfahrt und Softwaregrenzen nach erfolgreicher
  Referenzfahrt ergänzen. Vorher nur begrenzte manuelle Testbewegungen zulassen.
- Definierte Zustände für bereit, in Bewegung, gestoppt und Fehler einführen.
  Nach Neustart oder einem Fehler, bei dem Schritte verloren gehen können,
  gilt die Position als unbekannt und muss erneut referenziert werden.
- Referenzschalter, Abbruch während einer Bewegung und unerwartete Eingaben
  prüfen. Ein Software-Stopp ergänzt die unabhängige Abschaltmöglichkeit.
- Wiederholgenauigkeit, Schrittfrequenz und Grenzen des Aufbaus messen und
  zusammen mit den verwendeten Einstellungen protokollieren.

**Abschlusskriterium:** Wiederholte Hin- und Rückfahrten sowie Referenzfahrten
funktionieren innerhalb der zuvor festgelegten Toleranz; Abbruch und
Grenzverletzungen führen zu einem definierten Zustand.

## Phase 3: G-Code am einachsigen Aufbau

- Ein ausdrücklich dokumentiertes G-Code-Teilset einführen: zunächst G0/G1,
  G90/G91, G21 und Vorschub F, beschränkt auf die angeschlossene Achse.
- Unbekannte Befehle, nicht unterstützte Achsen, ungültige Werte und Bewegungen
  außerhalb der Grenzen vor der Ausführung mit verständlichem Fehler ablehnen.
- Parser, Bewegungsplanung und Hardwareausgabe trennen. Achsenparameter werden
  konfiguriert, damit der Ausbau keine Kopie der Einachsenlogik benötigt.
- Aufträge kontrolliert starten und abbrechen; nach Verbindungsabbruch oder
  Neustart darf kein Auftrag automatisch wieder anlaufen.
- Kleine Testprogramme mit absoluten und relativen Bewegungen ausführen und
  Soll-/Ist-Verfahrwege vergleichen.

**Abschlusskriterium:** Ein dokumentiertes Testprogramm wird reproduzierbar
ausgeführt; fehlerhafte Programme lösen keine unbeabsichtigte Bewegung aus.

## Phase 4: Auf vier Achsen erweitern

- Die gemeinsame Achsenabstraktion zunächst mit zwei Achsen prüfen und danach
  X/Y/U/V mit jeweils eigenem Treiber und Referenzschalter anschließen.
- Die Zuordnung festhalten: X/Y bewegen ein Drahtende, U/V das andere.
  Drehrichtung, Schritte pro Millimeter und Verfahrgrenzen je Achse kalibrieren.
- Lineare Bewegungen über einen gemeinsamen Bewegungsplan koordinieren:
  alle beteiligten Achsen beginnen und beenden einen G0/G1-Satz gemeinsam.
  Einzelachsengrenzen müssen auch bei kombinierten Bewegungen gelten.
- Für vier Achsen die Bedeutung von Vorschub F ausdrücklich festlegen und
  dokumentieren, insbesondere bei unterschiedlich langen Wegen der Drahtenden.
- Referenzfahrtreihenfolge und Verhalten bei einem Fehler an einer einzelnen
  Achse festlegen; ein solcher Fehler stoppt die gesamte koordinierte Bewegung.
- Gleichzeitige Pulsausgabe unter Last messen. Falls die gemessene Leistung
  nicht reicht, Pulssteuerung und Planung vor weiteren Funktionen überarbeiten.

**Abschlusskriterium:** Zwei- und Vierachsbewegungen mit unterschiedlichen Wegen,
Richtungswechseln und stillstehenden Einzelachsen funktionieren synchron;
Referenzierung und Fehlerbehandlung sind für alle Achsen geprüft.

## Phase 5: Gesamtsystem und erste Schnitte

- Mechanik, Drahtführung und zulässige Arbeitsbereiche des vollständigen
  Schneiders prüfen und zunächst Trockenläufe ohne Drahtheizung durchführen.
- Drahtheizung, Abschaltung und gegebenenfalls deren Steuerung separat
  spezifizieren. Ein Hardwarefehler oder Steuerungsabbruch muss auch für die
  Heizung ein definiertes Verhalten haben.
- Erst nach bestandenen Trockenläufen kontrollierte Testschnitte durchführen
  und Schnittgeometrie, Vorschub und Wiederholgenauigkeit protokollieren.
- Installation, unterstützten G-Code, Bedienung, Kalibrierung und bekannte
  Grenzen dokumentieren.

**Abschlusskriterium:** Ein vollständiger Referenzauftrag lässt sich vom Start
über die Referenzfahrt bis zum Schnitt nachvollziehbar wiederholen; dokumentierte
Stopp- und Fehlerfälle sind geprüft.

## Phase 6: WLAN-Zugang und Webbedienung

Die Ersteinrichtung erfolgt per USB direkt auf dem Pico. Eine lokale
Konfigurationsdatei enthält WLAN-SSID, WLAN-Passwort und einen lokalen Benutzer
mit Zugangsdaten für die Weboberfläche. Änderungen werden nach einem Neustart
übernommen. USB bleibt der Zugang für Konfiguration und Diagnose, auch wenn
die WLAN-Verbindung nicht hergestellt werden kann.

- Nach dem Neustart verbindet sich der Pico mit dem konfigurierten WLAN und
  stellt eine Weboberfläche bereit. Die erreichbare Adresse wird über USB
  ausgegeben.
- Der Benutzer öffnet die Oberfläche im Browser und meldet sich mit dem
  lokalen Benutzer an. Upload und Steuerbefehle erfordern eine gültige
  Anmeldung; die Zugangskontrolle gilt auch für die zugehörigen Server-Endpunkte.
- G-Code-Dateien lassen sich hochladen, auswählen und vor dem Start prüfen.
  Ein Upload startet keine Bewegung; ungültige Dateien werden abgelehnt.
- Die Mausbedienung umfasst manuelles Verfahren der Achsen, Referenzfahrt
  sowie Start und Stopp eines ausgewählten Auftrags. Positionen und
  Steuerungszustand werden in der Oberfläche angezeigt.
- Manuelles Verfahren erfolgt zunächst über begrenzte Einzelschritte pro
  Klick. Zulässige Verfahrwege und Zustände gelten auch für Webbefehle.
- Verhalten bei WLAN-Abbruch, abgelaufener Sitzung und mehreren geöffneten
  Browsern festlegen und prüfen. Eine neue Verbindung oder Anmeldung darf
  keine Bewegung automatisch wieder aufnehmen.

**Abschlusskriterium:** Konfiguration per USB, Neustart, WLAN-Verbindung,
Anmeldung, G-Code-Upload und Mausbedienung funktionieren im Trockenlauf;
unautorisierte Zugriffe und Verbindungsabbrüche sind geprüft.

## Offene Entscheidungen

- Motortyp, Treiber, Versorgung, Mechanik und Referenzschalter.
- Messbare Toleranzen für Position, Wiederholgenauigkeit und Pulstiming.
- Maximale Schrittfrequenz und geeignete Pulserzeugung auf dem Pico 2 W.
- Vorschubdefinition für koordinierte X/Y/U/V-Bewegungen.
- Umfang der Drahtheizungssteuerung.
- Format der Konfigurationsdatei, Speicherung des lokalen Benutzerpassworts
  und Absicherung der Anmeldung bei der Übertragung.
- Dateigrößenlimit und Speicherverwaltung für hochgeladene G-Code-Dateien.
- Verhalten laufender Aufträge bei WLAN-Abbruch oder abgelaufener Sitzung
  sowie Regeln für gleichzeitige Bedienung über mehrere Browser.

Diese Entscheidungen werden beim jeweiligen Meilenstein festgehalten. Neue
Funktionen bauen auf bestandenen Prüfungen des vorherigen Schritts auf.
