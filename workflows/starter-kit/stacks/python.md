# Stack-Pack: Python (Web-Anwendung oder Kommandozeile) mit uv

Gilt, wenn `docs/stack.md` unter „Stack-Packs" `python` nennt. Zielumgebung ist Linux. Die Anwendung läuft im Betrieb als Container oder direkt unter Linux, bedient wird sie über den Browser.

## Struktur
- `pyproject.toml` und `uv.lock` liegen im Repo. Die Python-Version steht in `.python-version` und in `requires-python`.
- Code im Paket `src/<paket>/`, Tests in `tests/`.
- **Fachlogik getrennt von der Technik:** Rechnen, Prüfen und Auswerten sind reine Funktionen in eigenen Modulen, ohne Zugriff auf Web, Dateien, Netz oder Uhr. Das heutige Datum und Zufallswerte kommen als Parameter herein. Die Web-Schicht bleibt dünn und ruft nur auf.
- Web: FastAPI mit Uvicorn, falls `docs/stack.md` nichts anderes nennt. Oberfläche mit serverseitigen Templates (Jinja2) und wenig eigenem JavaScript, ohne Build-Kette. Alle Skripte, Schriften und Stile liegen im Repo, nichts kommt von einem CDN.
- Ein Endpunkt `/health` antwortet ohne Anmeldung mit dem Stand der Anwendung (Version, Stand der geladenen Datenbestände).
- Konfiguration nur über Umgebungsvariablen, gelesen an genau einer Stelle (Settings-Modul). `.env.example` nennt alle Namen ohne Werte. Keine Zugangsdaten im Code, in Tests oder in Logs.

## Abhängigkeiten
- Nur mit `uv add` bzw. `uv add --dev`, nie `pip install`. `uv sync --frozen` muss auf einer frischen Maschine durchlaufen.
- Jede neue Abhängigkeit steht mit Grund im PR-Text. Bevorzuge die Standardbibliothek, wenn sie reicht.

## Eingaben, Dateien, Daten
- Eingaben an der Grenze mit Pydantic prüfen. Fehler als klare Meldung an den Nutzer, nie als Stacktrace.
- Uploads: Größe begrenzen, den Typ am Inhalt prüfen und nicht an der Endung, unter einem selbst erzeugten Namen ablegen. Der Dateiname des Nutzers ist nie Teil eines Pfads.
- Pfade mit `pathlib` bilden und prüfen, dass sie im Datenverzeichnis (`DATA_DIR`) bleiben.
- Verboten: `shell=True`, `eval`, `exec`, `pickle` oder `yaml.load` auf fremden Daten. XML nur mit `defusedxml`.
- Bestehende Datendateien nie direkt überschreiben: neue Datei schreiben und atomar ersetzen, die vorige Fassung als Sicherung behalten.
- Logs enthalten keine Dokumentinhalte und keine Schlüssel.

## Tests
- `pytest` unter `tests/`. Fachlogik mit festen Eingaben und festen Erwartungswerten, die Web-Schicht mit dem `TestClient` von FastAPI.
- **Kein Netz in Tests:** `pytest-socket` mit `--disable-socket --allow-unix-socket` in der pytest-Konfiguration. Kein Test hängt vom echten Datum ab.
- Browser-Tests mit `pytest-playwright`. Das Paket `playwright` muss **genau die Version des Job-Images** haben (`playwright --version` im Container), sonst passen die mitgelieferten Browser nicht. Browser-Tests starten die Anwendung selbst auf einem freien Port.
- Erwartungswerte mit fachlichem Gehalt (Einstufungen, Grenzwerte, Fristen) stammen aus der Spec oder aus den Referenzdaten des Projekts. Du denkst sie dir nie aus. Fehlen sie: Rückfrage.

## Auslieferung
- `Dockerfile` im Projektstamm: fest benanntes, schlankes Python-Basisimage, `uv sync --frozen --no-dev`, Nutzer ohne Root-Rechte, `HEALTHCHECK` gegen `/health`. Dazu `compose.yaml` mit einem Volume für `DATA_DIR`; der Port ist als Vorgabe an `127.0.0.1` gebunden.
- **Grenze der Factory:** Im Job-Container gibt es kein Docker. Du kannst das Image nicht bauen und behauptest das auch nicht. Änderst du `Dockerfile` oder `compose.yaml`, steht im PR-Text unter „Einschätzung" `Bitte genau prüfen` mit dem Hinweis, dass der Mensch das Image einmal bauen und starten muss.

## Befehle (Standard, siehe `.factory/setup.json`)
- install `uv sync --frozen` · dev `uv run uvicorn <paket>.web:app --host 0.0.0.0 --port $PORT --reload` · preview `uv run uvicorn <paket>.web:app --host 0.0.0.0 --port $PORT`
- checks: lint `uv run ruff check .` · format `uv run ruff format --check .` · typecheck `uv run pyright` · test `uv run pytest -q` · audit `uv run pip-audit --skip-editable`
- Ruff mindestens mit den Regelgruppen `E`, `F`, `B`, `S`, `A`, `C4`, `UP`, `I`, `RUF`. Pyright im Modus `strict` für `src/`. Eine einzelne Ausnahme (`# noqa`, `# type: ignore`) braucht den Regelcode, einen Grund im Kommentar und einen Eintrag im PR-Text unter „Annahmen".
