# Stack-Pack: LLM-Anbindung (Sprachmodelle in der Anwendung)

Gilt, wenn `docs/stack.md` unter „Stack-Packs" `llm-anbindung` nennt. Für Anwendungen, die selbst Sprachmodelle aufrufen, über einen Anbieter im Netz oder über ein lokal betriebenes Modell.

## Eine Stelle, ein Vertrag
- Modellaufrufe gibt es in genau einem Modul. Der Rest der Anwendung kennt nur dessen Schnittstelle (Eingabe, Ausgabe, Fehlerzustände).
- Angesprochen wird eine OpenAI-kompatible Schnittstelle. Adresse, Schlüssel und Modell kommen aus Umgebungsvariablen (`LLM_BASE_URL`, `LLM_API_KEY`, `LLM_MODEL`). Damit läuft dieselbe Anwendung gegen einen Anbieter oder gegen einen lokalen Server (llama.cpp, vLLM, Ollama), ohne Codeänderung.
- Dokumentinhalte gehen ausschließlich an die konfigurierte Adresse. Kein zweiter Dienst, keine Telemetrie, keine Inhalte in Logs.

## Antworten sind ungeprüfte Daten
- Jede Modellantwort wird gegen ein JSON-Schema geprüft, bevor sie weiterverwendet wird. Sie wird nie ausgeführt und nie als Pfad, Abfrage oder Befehl verwendet.
- Eine ungültige, unvollständige oder abgeschnittene Antwort führt zu einem ausdrücklichen Fehlerzustand, den der Nutzer sieht. Es gibt keine still eingesetzten Ersatzwerte und kein „bestes Raten".
- Verlangt die Spec einen Beleg (Seite, Fundstelle, Zitat), wird er mitgeführt und gegen das Dokument geprüft. Ein Wert ohne den verlangten Beleg gilt als nicht gefunden.
- Erschlossenes und Belegtes bleiben in den Daten getrennt.

## Grenzen
- Harte Obergrenzen im Code, je Aufruf und je Lauf: Tokens, Zeit, Anzahl Wiederholungen (mit wachsender Wartezeit), Anzahl Dokumente. Wird eine Grenze erreicht, bricht der Lauf mit klarer Meldung ab.
- Tokens und Kosten je Aufruf werden protokolliert und im Ergebnis ausgewiesen.

## Tests
- **Kein Test ruft ein echtes Modell auf.** Tests arbeiten mit aufgezeichneten Antworten unter `tests/aufzeichnungen/`, die ein Ersatz-Client abspielt. In der Factory gibt es keinen Schlüssel für die Modelle der Anwendung, und du forderst keinen an.
- Getestet werden mit Aufzeichnungen: der Normalfall, eine ungültige Antwort, eine abgeschnittene Antwort, eine Zeitüberschreitung, ein Fehler des Dienstes, das Erreichen jeder Obergrenze.
- Neue Aufzeichnungen erzeugt nur der Mensch mit einem Skript des Projekts. Du schreibst Aufzeichnungen nicht von Hand so um, dass ein Test grün wird. Brauchst du eine neue: Rückfrage.

## Prompts und Modellwahl
- Prompts und die Kennung des Modells liegen als eigene, versionierte Dateien im Repo, nicht als Zeichenketten im Code verstreut.
- Eine Änderung an Prompt oder Modell ist ein eigenes Feature. Ob die Ergebnisse dadurch besser oder schlechter werden, kann die Factory nicht beurteilen: Das zeigt nur ein Lauf des Menschen gegen den Referenzbestand. Ein solcher PR trägt immer die Einschätzung `Bitte genau prüfen` und nennt, welcher Referenzlauf vor dem Merge nötig ist.
