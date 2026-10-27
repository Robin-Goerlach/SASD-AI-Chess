# AGENTS.md – Arbeitsregeln für Coding Agents

**Geltung:** gesamtes Repository SASD-AI-Chess (root). Diese Anweisungen sind repositoryweite Arbeitsregeln für menschliche Beiträge und KI-Coding-Agents. Spezifische Unterverzeichnisregeln dürfen sie ergänzen, nicht stillschweigend aufheben.  
**Stand:** 27.10.2026 · Siehe [ROADMAP.md](ROADMAP.md) und [docs/ARCHITEKTUR.md](docs/ARCHITEKTUR.md).

## 1. Kontext, Wirklichkeit, Priorität

**Projekt:** eigenständige, portable **C++20-Schachengine mit lernfähiger KI**, Python/PyTorch nur für Training; **UCI** für Engines; **PGN** für gespeicherte Partien; Match-Controller für zwei eigenständige UCI-Prozesse. **SASD-Chess ohne KI** ist ein separates Repository, kein Verzeichnis oder automatisches Submodul dieses Projekts.

**Aktueller Ist-Stand:** Die Architektur- und Planungsdokumentation wurde begonnen. Geplante Programme, Ordner, CLI-Befehle, Unit-Tests und grüne CI-Läufe **sind nicht bereits vorhanden**, nur weil sie in Dokumenten stehen. **Vor jeder Änderung tatsächliche Dateien, Branch, HEAD, Issues und Workflow-Status überprüfen.**

**Prioritäten:** korrekt und sicher → nachvollziehbar/testbar → wartbar und kommentiert → interoperabel → optimiert. Keine erfundenen Messergebnisse, Commits, CI-Erfolge, Elo-Werte oder Trainingsfortschritte. Arbeitsphasen möglichst mit wirklich lauffähigen Zwischenständen abschließen.

## 2. Vorgehen in jeder Entwicklungsaufgabe

1. **Lesen:** README, Architektur, Roadmap, betroffene ADRs, Lasten-/Pflichtenheft und tatsächliche Quell-/Testdateien.
2. **Eingrenzen:** Anforderungen und Abnahmetests bestimmen. Kleine, in sich verständliche Slices bevorzugen.
3. **Architektur bewahren:** Chess Core ohne UCI, Python, Datenbank, UI oder Trainer; getrennte Prozesse für Match-Controller und Trainingsprogramme.
4. **Implementieren:** klare C++-APIs/Ownership/Fehlerfälle, CMake-Targets und kleine Testfälle. Keine unbestätigten externen Abhängigkeiten hinzufügen.
5. **Prüfen:** vorhandene Builds und Tests ausführen; für nicht ausführbare Plattformtests den Grund melden. Perft, UCI- und PGN-Regressionen nach Betroffenheit.
6. **Dokumentieren:** betroffene Anforderungen, Roadmap-Status **nur bei nachweislicher Erfüllung**, Changelog und ggf. ADR nachführen.
7. **Git:** vorhandene Branch-Protection-/PR-Regeln respektieren, keine destruktiven Force-Pushes oder Fremdänderungen überschreiben; kleine aussagekräftige Commits. Vor Merge/Änderungen auf `main` aktuellen Remotezustand prüfen. Niemals Tests als bestanden melden, wenn sie nicht liefen.

## 3. Technische Leitlinien

- **C++:** C++20; portable Standardbibliothek wo möglich; explizite Namespaces `sasd::chess::...`; minimale Header-Abhängigkeiten; klare `const`-/RAII-/Ownership-Semantik; keine unnötigen globalen Zustände oder UB. Plattformdetails hinter schmalen Adaptern.
- **Build:** CMake; eigenständige Targets für `core`, `search`, `evaluation`, `uci`, `match`, `dataset`; kein Zwang, PyTorch fürs Engine-Binary zu installieren. Warnungen ernst nehmen; Debug + Release bei passender Umgebung prüfen.
- **Kommentare:** Dokumentation der öffentlichen C++-APIs mit **ausführlichen englischen Doxygen-Kommentaren (`///` oder `/** ... */`)**. Auch nichttriviale Invarianten, Schachsonderregeln, mathematische Begründungen und knifflige Stellen **englisch inline** erklären. „Was tut die nächste Zeile?“ nicht redundant kommentieren. Fach-/Architekturdokumentation primär **Deutsch**.
- **Python:** eigener `training/`-Bereich; type hints, Docstrings, klare Seeds, reproduzierbare Konfiguration; getrennte Trainings-/Inferenzumgebungen. Modellschnittstelle explizit versionieren.
- **Core:** legale Züge und reversible Zustandsübergänge **vor** Bitboard-/SIMD-Optimierungen; korrekte Zähler/Rochade/en-passant/Wiederholung; Perft nach jeder Regeländerung.
- **Suche:** `stop` unterbrechbar, UCI-Reader unabhängig, SearchLimits als typisiertes Objekt; Bewertungsperspektive niemals implizit mischen.
- **I/O:** Protokollstrings **nur an Rändern**; `stdout` der UCI-Engine nur UCI; `stderr` für Debug; Parserlimits, Timeouts und Fehlerprotokolle.
- **Lizenz:** das **eigene** Repository steht unter [MIT](LICENSE). Fremdcode, Trainer, Modelle und PGN-Daten **nicht automatisch** als MIT behandeln; Lizenz/SPDX und Herkunft vor Einbindung/Verteilung prüfen. Insbesondere keinen GPL-Stockfish-Code in MIT-Quellcode kopieren, ohne die Lizenzfolgen ausdrücklich zu entscheiden.
- **Daten:** keine privaten Schlüssel, Token, riesigen PGN-Archive, Trainings-Shards, Modellgewichte oder persönlichen Pfade einchecken. Lizenz und Prüfsummen in Manifeste aufnehmen; Daten-splitting gruppenweise, keine bekannten Doppelstellungen als unbeachtete Leakage.
- **Security:** keine Shell-Interpolation bei UCI-Prozessstart; fremde Engine-Binärdateien nur explizit freigegeben; PGN/JSONL/FEN/UCI als untrusted input; Ressourcenlimits und Crashfälle testen.

## 4. Architekturgrenzen: explizite Verbote

**Nicht tun:**
- Schachregeln noch einmal im Match-Controller oder Python-Trainer unabhängig nachprogrammieren, wenn der geprüfte Core die Funktion liefert. Python-seitige Umwandlungen gegen Referenztestvektoren prüfen.
- UCI-Logik in `search`/`core`, PGN-Dateieinlesen in `evaluation`, PyTorch im Engine-Prozess oder Training im UCI-Hotpath verstecken.
- Ergebnisse aus einem einzelnen Testmatch als statistisch gesicherte Verbesserung ausgeben.
- „AGENTS.md“ in „AGENT.md“ umbenennen. [AGENT.md](AGENT.md) ist nur ein Kompatibilitätshinweis.
- Ohne nachvollziehbare Entscheidung eine neue GUI, Cloud-Infrastruktur, REST-API, Abhängigkeit zu SASD-UI-Toolkit oder Chess960 einführen.
- Aus Versehen bereits existierende, vom Nutzer gepflegte Dateien vollständig überschreiben.

**Erlaubt nach begründetem ADR:** andere interne Brettdarstellung, alternative Evaluatoren, SQLite-Katalog, C++-Parserbibliothek, Remote-Worker und GPU-Modelle.

## 5. Regeln für Tests / Entwicklungsnachweis

**Wenn echte Targets vorhanden sind** (diese Befehle sind bis dahin **zukünftige Beispiele**, keine Statusbehauptung):

```sh
cmake -S . -B build/debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build/debug
ctest --test-dir build/debug --output-on-failure
```

Bei MSVC-Multi-Config das aktive `--config Debug` bzw. `Release` angeben; `CMAKE_BUILD_TYPE` ist dort nicht die Konfigurationsauswahl. Engine- und Match-Integrationen mit **echten Unterprozessen** prüfen; FEN-/SAN-/UCI-Parser mit ungültigen Eingaben fuzz-/grenzwerttesten; Python-Featurekodierung gegen C++-Testvektoren. Ohne lokale Toolchain Tests als **nicht ausgeführt** kennzeichnen und genaue Kommandos für den Anwender dokumentieren.

**Done-Definition:** spezifizierte Abnahme erfüllt, relevante Tests nachweislich grün oder Blocker klar ausgewiesen, dokumentierte Bedienung, keine unversionierten Änderungen am Daten-/Modellformat, Changelog/Status aktualisiert, keine Regressionen.

## 6. Konkret nächste Arbeit

**M0 zuerst:** CMake-/CI-Skelett mit erstem Build, disziplinierten Core-Typen und ausführbarem Test. Danach **M1 Chess Core** mit Perft und Sonderregeln; dann UCI/Match; erst danach große Datenimporte und trainierte Bewertung. Abweichende Nutzeraufträge haben Vorrang, aber Architekturkonsequenzen und Tests sind zu benennen.

**Referenzen:** [Roadmap](ROADMAP.md) · [Architektur](docs/ARCHITEKTUR.md) · [Pflichtenheft](docs/PFLICHTENHEFT.md) · [Testkonzept](docs/TESTKONZEPT.md) · [ADR-Verzeichnis](docs/adr/README.md) · [Contributing](CONTRIBUTING.md).
