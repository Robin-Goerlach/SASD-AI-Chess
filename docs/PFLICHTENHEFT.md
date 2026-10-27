# SASD-AI-Chess – Pflichtenheft

**Version:** 0.1 · **Stand:** 27.10.2026 · **Status:** technischer Entwurf, keine Implementierungsbehauptung  
**Referenzen:** [Lastenheft](LASTENHEFT.md) (LR-IDs), [Architektur](ARCHITEKTUR.md), [Testkonzept](TESTKONZEPT.md), [Roadmap](../ROADMAP.md).

## 1. Umsetzungskonzept

Ein **modularer C++20-Monolith für die Engine** mit klarem Chess Core, klassischer/NNUE-Bewertung, iterativer Alpha-Beta-Suche und UCI-Adapter; daneben **eigenständige CLI-Prozesse** für Match-Controller und Datenimport; Python/PyTorch ausschließlich für Training, Modell-/Statistikauswertung. Alle externen Daten werden an Adaptergrenzen validiert. Technologieentscheidungen werden per [ADR](adr/README.md) dokumentiert.

**Definition of Done pro Anforderung:** funktionsfähiger Code + passende reproduzierbare Tests + dokumentierte Schnittstelle/Fehlerfälle + erfasster Status. Beispielbefehle in dieser Spezifikation sind **nicht implementiert**.

## 2. Verantwortlichkeiten / Schnittstellen

| Baustein | Eingabe | Ausgabe / Vertrag | Kein Zugriff auf |
| --- | --- | --- | --- |
| **Chess Core** | Position, Move | neuer/reversibler Zustand; Legalität/Ergebnis; FEN/SAN | UCI, FS, UI, Python |
| **Search** | Position, SearchLimits, Abbruchtoken, Evaluator | SearchResult inkl. bestmove, PV, Statistiken | stdin/stdout, PGN-Dateien |
| **Classical/NNUE Evaluator** | Position, festes Modell/Parameter | Bewertung mit definierter Perspektive | Match-Uhr, Netz, Trainer |
| **UCI Adapter** | zeilenorientierte Textkommandos | UCI-Antworten; kontrollierter Engine-Lebenszyklus | Regeln duplizieren |
| **Match Controller** | zwei UCI-Programme, Matchlimits | autoritatives Partieprotokoll, Zeit/Urteil, PGN/JSON | interne Suchstrukturen |
| **Dataset Import** | PGN, zstd, Evaluations-JSONL | Game-/Position-/Labelrecord, Manifest, Fehlerbericht | Python-Modellgewichte |
| **Trainer** | Dataset/Manifest + TrainingConfig | exportiertes Modell + Run-Protokoll | UCI stdout/Chess Core verändern |

## 3. Detaillierte funktionale Realisierung

### PF-01 – Chess Core (LR-F01, LR-F02, LR-Q01)

- Modell `Position` mit Feldern, Side-to-Move, Castling-Rights, en-passant-Ziel, Halbzugzähler und Zugnummer.
- Move umfasst Ausgang/Ziel, Promotion und nötige interne Flags.
- Zugerzeugung prüft Königsschutz, Rochade über bedrohte Felder, korrektes en passant, vier Promotiontypen; kein König darf geschlagen werden.
- ``make_move / unmake_move`` reproduzieren exakten Zustand einschließlich Hash/History.
- Endzustände: Matt, Patt, unzureichendes Material, fünfmalige Wiederholung / 75-Züge automatisch; drei-/fünfzig-Züge-Recht als **beanspruchbar**. Partieresultate durch Aufgabe/Zeit werden nicht aus FEN erfunden.
- FEN lesen/schreiben, SAN lesen/schreiben, UCI-Koordinatenzüge; Spielhistorie explizit separat.

**Nachweis:** ``TC-CORE-*`` inklusive Perft und gezielter Sonderfälle. Internes Board-Layout bleibt nicht Teil der öffentlichen API.

### PF-02 – Suchengine (LR-F03, LR-F11, LR-Q02)

- Zunächst garantierte legale Zugwahl; darauf aufbauend iterative Vertiefung, Negamax/Alpha-Beta, Quiescence Search, Transposition Table, Move Ordering, Classical Evaluator.
- SearchLimits unterstützt `depth`, `nodes`, `movetime`, `wtime/btime/winc/binc`; Kontrolle mit monotoner Uhr und Abbruchtoken.
- `SearchResult` liefert auch in Endstellungen wohldefiniertes „kein Zug“, ohne Engine-Absturz.
- Spätere Optimierungen ändern keine Spielregeln und werden einzeln benchmarked.

**Nachweis:** ``TC-SEARCH-*``, keine ungültigen Züge, pünktliches Stop/Timeout-Verhalten mit definierter Toleranz.

### PF-03 – UCI (LR-F04, LR-Q03)

- Zeilenorientiertes stdin/stdout; `uci` → Identifikation + `uciok`; `isready` → `readyok`; `ucinewgame`; `position startpos|fen ...`; `go`; `stop`; `quit`.
- Reader bleibt unabhängig von Suchthread. Genau eine finale `bestmove`-Antwort pro gestarteter Suche im definierten Lebenszyklus; kein unverlangtes Debug auf stdout.
- Ungültige oder unbekannte Befehle werden begrenzt behandelt; optionale UCI-Features nur anmelden, wenn implementiert.
- Schachmatt-/Patt-Stellung ohne Zug → `bestmove 0000`.

**Nachweis:** ``TC-UCI-*``, einschließlich Unterbrechung, neuem Spiel, mehrfachen Kommandos, Pipes und Prozessende.

### PF-04 – Engine-gegen-Engine (LR-F05, LR-F06, LR-F07)

- Eigenes Match-Programm startet **zwei unabhängige** konfigurierbare UCI-Prozesse, auch denselben Pfad zweimal; UCI-Kommunikation je mit eigenem stdin/stdout/stderr.
- Controller synchronisiert Optionen, Anfangsstellung und vollständige Zugfolge; prüft **jeden Zug** mit autoritativem Chess Core.
- Monotone Uhren, Zeitkontrollen, Schachregeln, Schiedsrichterentscheidungen, Zeit-/Crash-/Fehlzugbehandlung, begrenzte Ausgaben.
- PGN plus strukturiertes Matchresultat mit Time-Control, Engines/Hashes, Modellen, Farben, Start-FEN, Abbruchursache, Seed.

**Nachweis:** ``TC-MATCH-*``, komplette Partie zwei eigener Prozesse, Rotationen von Farben, deterministischer Fehlerfall.

### PF-05 – Partien/Datenpipeline (LR-F08, LR-F09, LR-F10, LR-F16, LR-F17, LR-Q04, LR-Q09)

- PGN-Adapter parst Tags, SAN, Kommentare und sichere Variantenbehandlung; `SetUp/FEN`, Ergebnis und Mehrpartienarchiv.
- Streaming-PGN/ZSTD mit festen Limits; ungültige Partie in Quarantäne mit Quelle/Position/Fehler, nachfolgende gültige Partien weiterverarbeiten.
- Filter nach Elo, Zeitkontrolle, Variante, Resultat, Quelle; fehlende Werte als „unbekannt“, nicht geschätzt.
- Rekonstruktion pro Halbzug liefert Stellung, legalen Spielzug, Ergebnis- und optionale Referenzlabels.
- SHA-256/Manifest, Quelllizenz, Filterkonfiguration, Schema-ID, eindeutige Partien, gruppierte Train/Validation/Test-Splits (Partie-/Duplikatcluster), unabhängiger Holdout.
- Bewertete Positionen (JSONL/FEN) über eigenen, explizit versionierten Importadapter; Score-Perspektive, Matt-/CP-Semantik und Bewertungstiefe immer erfassen.

**Nachweis:** ``TC-DATA-*``, Replay gegen Referenz-FEN, defekte PGNs, Deduplizierung, Split ohne bekannte Leckagen, Streaming-/Ressourcencheck.

### PF-06 – Lernen und Modelleinbindung (LR-F12, LR-F14, LR-F15)

- Python-/PyTorch-Training als separater Prozess. Anfänglich: **kleines NNUE-artiges CPU-Value-Netz** auf legal rekonstruierten Positionen und qualitätsgeprüften Labels.
- `feature_schema_version`, Modellformatversion, Train-/Test-Split, Labelperspektive, Seeds und reproduzierbare Run-IDs.
- Export in ein **validierbares Runtime-Format**, ggf. quantisiert; C++-Inferenz ohne Python/GPU; dieselben Feature-Testvektoren in Python und C++.
- Unbekanntes/inkompatibles Modell → verständlicher Fehler, kein stiller Fallthrough in ein anderes Modell.

**Nachweis:** ``TC-ML-*``, Python/C++-Parität mit Toleranz, fester Mini-Trainingslauf, Inferenz-Latenz, Modell-Metadaten.

### PF-07 – Evaluation (LR-F13, LR-F18, LR-Q08, LR-Q10)

- Matches mit paarweise identischen Eröffnungen und wechselnden Farben; Versions-/Hardware-/Zeitkontrollgleichheit dokumentieren.
- Separate Offline-Metriken (Loss/Kalibrierung) und reale Stärke-Metriken (Matchscore/Elo-Differenz samt Unsicherheit).
- Kandidat freigeben nur nach **vorab festgelegter** Qualitätsentscheidung; negative und nicht signifikante Ergebnisse bleiben erhalten.
- Selbstspieltraining erst danach als Erweiterung, mit unveränderlicher Modellversion pro Partie.

**Nachweis:** ``TC-EVAL-*`` inklusive Zurückweisung eines schwachen Modells und rekonstruierbarer Experimentberichte.

## 4. Nichtfunktionale Umsetzung

| LR-ID | Technische Maßnahme | Nachweis |
| --- | --- | --- |
| LR-Q01 | gemeinsamer Chess Core, Perft, make/unmake-Propertytests | TC-CORE |
| LR-Q02 | Engine-CMake-Target nur mit C++-Runtime und Modellformat | Clean Build, Offline Smoke |
| LR-Q03 | CTest-/Python-Tests, Log-/Fehlerregeln, CI-Matrix | TC-BUILD / Statusberichte |
| LR-Q04 | MIT für eigenen Code, Lizenzmanifest für Fremddaten/Abhängigkeiten | Lizenz-Audit |
| LR-Q05 | begrenzte Ein-/Ausgaben, Quarantäne, Prozess-/Zeitlimits | TC-SEC |
| LR-Q06 | Doxygen/englische Inline-Kommentare, Architekturgrenzen | Review |
| LR-Q07 | Windows/Linux/BSD-Adapter und reale Compiler-Logs | TC-BUILD |
| LR-Q08 | Run-Manifest, Hashes, Seeds, Holdout, Konfidenz | TC-EVAL |
| LR-Q09 | Streaming-ZSTD, bounded queues und Chunk-Writer | TC-DATA |
| LR-Q10 | Benchmarks + gepaarte Match-Evaluation | TC-PERF |

## 5. Traceability-Matrix und Ausbaustufen

| Lastenheft | Umsetzung | Hauptereignis | Abnahmetests |
| --- | --- | --- | --- |
| LR-F01..F02 | PF-01 | M1 | TC-CORE |
| LR-F03, F11 | PF-02 | M2 | TC-SEARCH |
| LR-F04 | PF-03 | M2 | TC-UCI |
| LR-F05..F07 | PF-04 | M3 | TC-MATCH |
| LR-F08..F10, F16..F17 | PF-05 | M4 | TC-DATA |
| LR-F12, F14..F15 | PF-06 | M5/M7 | TC-ML |
| LR-F13, F18 | PF-07 | M6 | TC-EVAL |
| LR-F19..F20 | gesondertes ADR/Pflichtenheft bei Beauftragung | nach M7 | noch zu definieren |

## 6. Technische Grenzen und offene Entscheidungen

**Nicht umgesetzt:** die oben beschriebenen Module und Befehle sind noch nicht Teil einer funktionierenden Anwendung. Exakte Tools/Bibliotheken, Datenformat (JSONL/Parquet/Binär), NNUE-Feature-Encoding, quantisierte Gewichtsrepräsentation, Rating-Methoden und Unterstützungsgrenzen von BS-Versionen werden als **explizite ADRs** vor der jeweiligen Implementierung festgelegt. Keine stillschweigende Spezifikationsänderung.

**Erste überprüfbare Lieferung:** M0 CMake-/Test-Skelett, dann M1 Regeln/FEN/Perft. Nach jedem Meilenstein erhält die Dokumentation ein wirklich belegtes Statusupdate.
