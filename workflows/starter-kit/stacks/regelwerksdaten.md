# Stack-Pack: Regelwerksdaten (Rechtslisten, Grenzwerte, Kataloge)

Gilt, wenn `docs/stack.md` unter „Stack-Packs" `regelwerksdaten` nennt. Für Anwendungen, deren Ergebnis von amtlichen oder normativen Inhalten abhängt (Stofflisten, Anhänge von Verordnungen, Grenzwerte, Fristen, Kataloge). Ein falscher Eintrag ist hier ein fachlicher Fehler mit Folgen für den Nutzer, kein Schönheitsfehler.

## Die Grundregel
**Regelwerksinhalte kommen nie von dir.** Du schreibst keinen Listeneintrag, keinen Grenzwert, keine Einstufung und kein Datum aus deinem Wissen, aus einer Webseite oder aus einem Beispiel in den Code oder in eine Datendatei. Inhalte entstehen ausschließlich, indem ein Import-Programm eine Quelldatei liest. Du baust und testest das Programm, nicht den Inhalt.

Fehlt dir eine Quelldatei, ein Erwartungswert oder eine fachliche Regel: Rückfrage. Raten ist hier der schwerste Fehler.

## Quellen
- Rohdateien liegen unverändert unter `quellen/<regelwerk>/<stand>/`. **Der Mensch legt sie dort ab**, oder die Spec nennt Adresse und erwartete Prüfsumme. Du lädst keine amtliche Datei auf eigene Faust nach und ersetzt keine vorhandene.
- Zu jeder Rohdatei gehört ein Eintrag in `quellen/QUELLEN.json`: Titel, Herausgeber, Adresse, Abrufdatum, Stand bzw. Fassung, SHA-256 der Datei, wer sie abgelegt hat.

## Datendateien
- Der Import schreibt nach `daten/<regelwerk>.json`. Jede Datendatei beginnt mit einem Block `quelle`: Rohdatei, deren SHA-256, Stand, Name und Version des Import-Programms.
- Ein JSON-Schema je Datendatei liegt unter `schemas/`. Die Anwendung prüft beim Start jede Datendatei gegen ihr Schema und gegen die Prüfsumme im Block `quelle`. Schlägt das fehl, startet sie nicht, statt mit zweifelhaften Daten zu rechnen.
- Fachliche Regeln (Schwellen, Zuordnungen, Ausnahmen) stehen als Daten mit Fundstelle (Verordnung, Artikel, Anhang, Nummer) in einer Datendatei, nicht verstreut im Programmcode.
- Oberfläche und jede Ausgabe nennen den Stand der verwendeten Regelwerke.

## Import-Programm
- **Deterministisch:** dieselbe Rohdatei ergibt eine byte-gleiche Datendatei (sortierte Schlüssel, feste Reihenfolge, kein Zeitstempel des Laufs).
- **Laut statt still:** Jede Zeile der Quelle wird entweder übernommen oder mit Grund gezählt und gemeldet. Unerwartete Spalten, leere Pflichtfelder, doppelte Schlüssel oder ein unbekanntes Format brechen den Import ab. Ein Parser, der Zeilen still überspringt, ist ein blockierender Befund.
- **Änderungsbericht:** Vor dem Ersetzen einer Datendatei schreibt der Import einen Bericht gegenüber dem bisherigen Stand (neu, entfallen, geändert, je mit Schlüssel). Ohne Bericht wird nichts ersetzt. Der bisherige Stand bleibt als eigene Datei erhalten.
- Ein Trockenlauf ist die Vorgabe. Geschrieben wird nur mit ausdrücklichem Schalter.

## Tests
- Für jedes Import-Programm eine kleine, von Hand gebaute Testquelle unter `tests/quellen/` mit dem erwarteten Ergebnis: Normalfall, Sonderzeichen, leere Felder, doppelte Schlüssel, unbekannte Spalte.
- Ein Test prüft die Byte-Gleichheit bei zweimaligem Lauf, einer das Schema, einer den Änderungsbericht.
- **Stichproben an echten Daten gibt die Spec vor** (etwa „Eintrag X hat den Wert Y, Zeilenzahl ist N"). Du erfindest keine Stichproben. Nennt die Spec keine, steht das im PR-Text unter „Einschätzung" mit `Bitte genau prüfen`.
- Bekannte Fehlalarme und frühere Fehler bleiben als benannte Tests dauerhaft erhalten.

## Im Pull Request
- Ändert der PR eine Datendatei, steht der Änderungsbericht im PR-Text, zusammen mit Rohdatei und Prüfsumme.
- Ein PR, der eine Datendatei ändert, ohne dass eine Rohdatei oder das Import-Programm sich geändert hat, ist nicht zulässig.
