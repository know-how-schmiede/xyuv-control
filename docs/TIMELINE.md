# Timeline und Versionsschritte

Der fachliche Entwicklungsplan steht in [project.md](../project.md).
Alle folgenden Meilensteine sind geplant. Zielversionen beschreiben die
vorgesehene Reihenfolge und sind noch keine veröffentlichten Releases.
Termine werden ergänzt, sobald Hardware, Aufwand und Verfügbarkeit geklärt sind.

## Geplante Meilensteine

| Zielversion | Meilenstein | Voraussetzung | Abschlusskriterium | Status | Zieltermin |
| --- | --- | --- | --- | --- | --- |
| 0.1.0 | Einachsiger Testaufbau | Komponenten und Anschlussplan festgelegt | Aufbau dokumentiert; Start mit deaktiviertem Treiber und unabhängige Abschaltung geprüft | Geplant | Offen |
| 0.2.0 | Einachsige Bewegung und Referenzfahrt | 0.1.0 abgeschlossen | Verfahrbewegungen, Rampen, Referenzierung, Grenzen und Abbruch geprüft; Messwerte protokolliert | Geplant | Offen |
| 0.3.0 | G-Code für eine Achse | 0.2.0 abgeschlossen | Dokumentiertes Teilset und Testprogramm funktionieren; ungültige Eingaben werden sicher abgelehnt | Geplant | Offen |
| 0.4.0 | Koordinierter Zweiachsen-Test | 0.3.0 abgeschlossen | Gemeinsame Bewegungsplanung und synchrone Ausgabe bei unterschiedlichen Verfahrwegen geprüft | Geplant | Offen |
| 0.5.0 | Vier Achsen X/Y/U/V | 0.4.0 abgeschlossen | Alle Achsen kalibriert; Vorschubdefinition, Referenzierung und gemeinsame Fehlerbehandlung geprüft | Geplant | Offen |
| 0.6.0 | Vollständiger Aufbau und Trockenläufe | 0.5.0 abgeschlossen | Referenzauftrag ohne Heizung wiederholbar; Arbeitsbereiche und Stoppfälle geprüft | Geplant | Offen |
| 0.7.0 | Erste kontrollierte Testschnitte | 0.6.0 abgeschlossen; Heizung und Abschaltung spezifiziert und geprüft | Schnittgeometrie und Wiederholgenauigkeit anhand festgelegter Kriterien dokumentiert | Geplant | Offen |
| 1.0.0 | Dokumentierter Grundbetrieb | Vorherige Meilensteine abgeschlossen | Referenzauftrag und Fehlerfälle bestanden; Installation, Bedienung, Kalibrierung und Grenzen dokumentiert | Geplant | Offen |

WLAN-Bedienung und zusätzliche G-Code-Funktionen erhalten eigene Meilensteine,
sobald ihr Umfang feststeht. Sie sind keine Voraussetzung für die ersten
einachsigen Tests.

## Pflege der Timeline

- Status je Meilenstein: **Geplant**, **In Arbeit**, **Blockiert** oder
  **Abgeschlossen**. Bei Blockierung den Grund und den nächsten Schritt ergänzen.
- Beim Abschluss Datum, tatsächlich erreichte Version und Prüfbelege festhalten.
- Änderungen an Umfang oder Reihenfolge mit kurzer Begründung dokumentieren.
- Zieltermine sind Planung; tatsächliche Abschlussdaten werden separat erfasst.
- Versionsschritte unten in zeitlicher Reihenfolge ergänzen. Die Versionsnummer
  allein bestätigt keinen bestandenen Meilenstein.

## Vorlage für einen Versionsschritt

Den folgenden Abschnitt für jeden geplanten oder abgeschlossenen Versionsschritt
kopieren und die Platzhalter ersetzen. Nicht durchgeführte Prüfungen ausdrücklich
als offen kennzeichnen.

```markdown
### Version <MAJOR.MINOR.PATCH> – <Kurzbezeichnung>

- Status: <Geplant / In Arbeit / Blockiert / Abgeschlossen>
- Zugehöriger Meilenstein: <Zielversion und Name>
- Zieltermin: <YYYY-MM-DD / Offen>
- Abschlussdatum: <YYYY-MM-DD / Noch offen>
- Referenz: <Commit, Tag oder Pull Request / Noch offen>

#### Ziel
<Welche konkrete Fähigkeit soll dieser Schritt erreichen?>

#### Änderungen
- <Neue oder geänderte Funktion, Hardware oder Dokumentation>

#### Aufbau und Konfiguration
- Hardware/Firmware: <Komponenten und verwendeter Stand>
- Achsenparameter: <Schritte/mm, Geschwindigkeit, Beschleunigung, Grenzen>
- Weitere Einstellungen: <Relevante Werte oder Verweis auf Konfiguration>

#### Prüfung und Ergebnis
| Prüfung | Erwartung / Toleranz | Ergebnis / Messwert | Beleg |
| --- | --- | --- | --- |
| <Testfall> | <Vorab festgelegtes Kriterium> | <Bestanden / Fehlgeschlagen / Offen; Messwert> | <Protokoll oder Datei> |

#### Bekannte Grenzen und offene Punkte
- <Problem, Auswirkung und nächster Schritt; gegebenenfalls Blockierungsgrund>

#### Entscheidung zum Meilenstein
<Abgeschlossen oder noch offen, mit Begründung anhand der Abschlusskriterien>

#### Nächster Versionsschritt
<Geplante Version, Ziel und Voraussetzungen>
```

## Versionsprotokoll

Noch keine abgeschlossenen Versionsschritte dokumentiert.
