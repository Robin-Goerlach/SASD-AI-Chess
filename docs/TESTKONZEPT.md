# SASD-AI-Chess – Test- und Qualitätssicherungskonzept

**Version:** 0.1 · **Stand:** 27.10.2026 · **Status:** geplanter Testkatalog; **kein** Testergebnis  
**Verweise:** [Pflichtenheft](PFLICHTENHEFT.md) · [Roadmap](../ROADMAP.md) · [Architektur](ARCHITEKTUR.md).

## 1. Testphilosophie

Die Schach-KI darf niemals durch KI-Qualität einen Fehler in den Spielregeln kaschieren. Daher gelten **Regelkorrektheit, UCI-Protokollkonformität, Zeit-/Prozesssicherheit und Datenvalidierung** als harte Gates. Leistung und Elo werden **separat** gemessen, nicht aus Testdurchläufen abgeleitet.

**Testzustände:** nicht implementiert / nicht ausgeführt / bestanden / fehlgeschlagen / wegen Umgebung blockiert. „Bestanden“ nur mit realem Lauf, Commit-ID, Plattform, Compiler-/Runtimeversion, Kommando und Ergebnis.

## 2. Testpyramide und konkrete Tests

### TC-CORE – Schachregeln (M1)

| ID | Testfall | Erwartung |
| --- | --- | --- |
| TC-CORE-001 | Start-FEN einlesen/schreiben | identische gültige Stellung und Felder/Zähler |
| TC-CORE-002 | Startstellung Perft Tiefe 1–4 | **20 / 400 / 8.902 / 197.281** Knoten |
| TC-CORE-003 | Kiwipete-/weitere Perft-Referenzen mit geprüften FEN | publizierte, vor Test fixierte Knotenwerte |
| TC-CORE-004 | Lange Rochade durch Schach / besetzte Zwischenfelder | illegal; kurze Rochade separat korrekt |
| TC-CORE-005 | En passant bei gefesseltem Bauer | Zug illegal, wenn danach eigener König im Schach |
| TC-CORE-006 | Bauernumwandlung | D/T/L/S und Farbwechsel korrekt |
| TC-CORE-007 | Schachmatt und Patt | legaler Zugbestand leer, Ursache korrekt unterschieden |
| TC-CORE-008 | Wiederholung und 50/75-Züge-Regeln | claimbare und automatische Remisfälle getrennt |
| TC-CORE-009 | `make/unmake` bei zufälligen legalen Zugsequenzen | Positionszustand, Hash, Historie unverändert |
| TC-CORE-010 | FEN/SAN: ungültige Züge/Eingaben | deterministischer Parse-/Legalitätsfehler |
| TC-CORE-011 | Differentialtest gegen unabhängigen Referenzprüfer | legaler Zugbestand (inkl. Sonderfälle) identisch |

**Hinweis:** Perft-Werte zählen *Zugknoten*, nicht Trainingserfolg. Weitere Positionen erst nach Übernahme einer überprüften Referenz-FEN-/Zahlenliste. Stellungen zur Testgenerierung stets fixieren (Seed + FEN).

### TC-SEARCH – Suche und klassische Evaluation (M2)

- TC-SEARCH-001: alle zurückgegebenen `bestmove`-Züge sind in der geprüften Position legal.
- TC-SEARCH-002: bekannte Mattstellungen ergeben korrekt keine/Zug-Lösungen; kein Zugriff außerhalb Array.
- TC-SEARCH-003: `go depth` und `go movetime` beachten harte Suchlimits und definierte Toleranzen.
- TC-SEARCH-004: `stop` während Suche und `quit` beim Beenden ohne Deadlock/Use-after-free.
- TC-SEARCH-005: Transposition-/Evaluation-Caches verletzen keine Halbzug-/Wiederholungsregeln.

### TC-UCI – Protokollintegration (M2)

- TC-UCI-001: `uci` ergibt `id name`/`uciok`, `isready` ergibt `readyok`.
- TC-UCI-002: `position startpos moves ...` erzeugt erwartete FEN; `position fen ...` ebenso.
- TC-UCI-003: nach `go` genau eine finale Antwort pro Suche, legaler Zug bzw. `bestmove 0000` in terminaler Stellung.
- TC-UCI-004: stop/quit/ucinewgame während bzw. nach Suche; keine Prozesshängung.
- TC-UCI-005: malformed/unknown/zu lange Eingabezeilen, EOF, ungültige Optionen ohne Crash.
- TC-UCI-006: stdout nur gültiges UCI; Logging nach stderr oder `info string`.

### TC-MATCH – Zwei UCI-Prozesse (M3)

- TC-MATCH-001: **dieselbe Engine zweimal** gegen sich selbst, getrennte Prozess-IDs und Pipes.
- TC-MATCH-002: eine andere UCI-kompatible Engine bei lokaler Verfügbarkeit.
- TC-MATCH-003: Zeitüberschreitung, Absturz, `bestmove` illegal/fehlend, EOF, endlose `info`-Ausgabe.
- TC-MATCH-004: korrektes Resultat bei Matt, Patt, Aufgabe, Remis und Zeitentscheidung; PGN/JSON stimmen überein.
- TC-MATCH-005: vollständige Zugfolge samt Repetitionshistorie an jede Engine; Farbtausch gepaarter Eröffnungen.

### TC-DATA – Import und Qualität (M4)

- TC-DATA-001: ein PGN mit mehreren Partien, Tags, Kommentaren und Varianten liefert Hauptvarianten/FEN/Ergebnis korrekt.
- TC-DATA-002: `SetUp=1` mit `FEN` startet in korrekter gültiger Stellung; Nicht-Standardschach wird markiert/ausgeschlossen.
- TC-DATA-003: illegales SAN, defekte Tags, Encoding-Fehler und riesige Zeilen → Quarantäne, definierter Bericht.
- TC-DATA-004: `.pgn.zst` streamend einlesen; begrenzter Speicher bei großen/unbekannten Entpackgrößen.
- TC-DATA-005: Spielduplikate und gleiche Positionen erkennen; keine bekannten Leckagen zwischen Splits.
- TC-DATA-006: Missing Elo/Result/TimeControl klar als „unbekannt“; Filter rechnen korrekt.
- TC-DATA-007: identische Quelldatei+Konfiguration → identische manifestierte Positionen/Hashes (wo deterministisch vorgesehen).
- TC-DATA-008: cp/mate/WDL-Labelperspektive und Quellinformationen unverändert dokumentiert.

### TC-ML / TC-EVAL – Training und Wissenschaft (M5–M7)

- TC-ML-001: Python-/C++-Encoder erzeugen denselben `feature_schema_version`-konformen Vektor.
- TC-ML-002: CPU-C++-Inferenz entspricht exportiertem Python-Modell innerhalb dokumentierter Quantisierungs-Toleranzen.
- TC-ML-003: inkompatible, beschädigte oder zu große Modelle werden mit Fehler abgewiesen.
- TC-ML-004: kleinstes reproduzierbares Trainings-/Exportbeispiel schreibt Run-/Model-Manifest.
- TC-EVAL-001: paarweise Vergleichsspiele mit gleichen Eröffnungen, Farbtausch, fixen Limits.
- TC-EVAL-002: Offline-Loss und Matchrating getrennt; Unsicherheiten aus tatsächlicher Stichprobe berichten.
- TC-EVAL-003: bewusst schwacher Modellkandidat wird nicht automatisch promoted.
- TC-EVAL-004: gespeichertes Experiment kann anhand der aufgezeichneten Versionen/Seeds nachvollzogen werden.

### TC-SEC / TC-PERF / TC-BUILD

- TC-SEC: UCI-/PGN-/ZSTD-/Modellparser adversarial testen (Größenlimits, Dekompressionsbomben, ungültiges UTF-8), fremde Prozesse kontrolliert beenden, keine ungewollte Shell-Ausführung.
- TC-PERF: repräsentative Stellungssuite, Nodes/s, Suchzeit, Inferenzlatenz, RAM-Spitzen, PGN/s, Training Samples/s; Hardware/Compiler exakt angeben.
- TC-BUILD: frischer Checkout, **Debug/Release**; reale Windows-MSVC-, Linux-GCC-/Clang- und nach Verfügbarkeit FreeBSD-/OpenBSD-Builds. Kein „BSD geprüft“ ohne BSD-Test.

## 3. Testumgebungen und Automatisierung

- **C++:** CMake + CTest, separate Unit-, Protokoll-, Prozess- und Perft-Tests.
- **Python:** isolierte virtuelle Umgebung und wiederholbare Mini-Tests; bestimmte PyTorch-Version via Lock/Constraints, sobald festgelegt.
- **CI:** mindestens ein Linux- und Windows-Build sobald Code existiert; BSD zunächst dokumentierte manuelle Durchläufe und später geeignete Runner; Jobs und Ergebnisse verlinken.
- **Fehlersuche:** Sanitizer (ASan/UBSan) wo verfügbar; Valgrind/Fuzzer optional; Absturzlogs und Minimal-FEN/PGN als reproduzierbare Testfälle.

Geplante Beispielkommandos **erst nach M0**:
```sh
cmake -S . -B build/debug -DCMAKE_BUILD_TYPE=Debug
cmake --build build/debug
ctest --test-dir build/debug --output-on-failure
```

MSVC-Multi-Config: `cmake --build build --config Debug` und `ctest --test-dir build -C Debug` verwenden, nicht nur `CMAKE_BUILD_TYPE`.

## 4. Abnahmeregel, Regression, Protokollvorlage

**Schwellen:** 100 % definierte harte Core-/Protokoll-/Sicherheits-Tests für eine Freigabe; Leistungs- und KI-Experimente werden gesondert bewertet. Konkrete zeitliche Toleranzen/Performanceziele erst nach gemessener Baseline in ADR festschreiben.

**Pro Testbericht mindestens:** Zeitpunkt; Git-Commit; System/OS/Architektur; Compiler und Version; Buildkonfiguration; Befehl; Testfall/Datensatz/Hash; erwartetes/ist Ergebnis; Logs/Referenzen; bekannte Einschränkungen. Statistische KI-Tests zusätzlich Gegner/Modell, Time-Control, Öffnungen, Farben, Partienanzahl, Score, Konfidenz und vorher festgelegte Hypothese.

**Stand am 27.10.2026:** **alle TC-* sind geplant, nicht ausgeführt**. Dieses Dokument darf nicht als Nachweis implementierter Funktionalität interpretiert werden.
