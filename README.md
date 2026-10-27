# SASD-AI-Chess

**SASD-AI-Chess** ist ein eigenständiges Open-Source-Projekt für eine portable C++-Schachengine mit trainierbarer KI. Es verbindet regelkonformes Schach, UCI-Interoperabilität, reproduzierbare KI-Experimente und maschinelles Lernen. Das separate Projekt **SASD-Chess** (ohne KI) ist kein Bestandteil dieses Repositorys, kann aber künftig als UCI-Gegner dienen.

**Projektstatus (27.10.2026):** Dokumentations- und Architekturphase (M0). Dieses Repository enthält **noch keine spielbare Engine, kein trainiertes Modell und keine produktive Import-/Trainingspipeline**. Beschriebene CLI-Befehle, Builds und Tests sind **Zielentwürfe**, sofern nicht ausdrücklich als implementiert gekennzeichnet.

## Dokumentation

| Dokument | Inhalt |
| --- | --- |
| [Architektur](docs/ARCHITEKTUR.md) | Modulgrenzen, Ports/Adapter, Systemkontext, Daten und ML |
| [Roadmap](ROADMAP.md) | Reihenfolge, Meilensteine, Abnahmen und Status |
| [Lastenheft](docs/LASTENHEFT.md) | Fachliche Ziele und Anforderungen („Was?“) |
| [Pflichtenheft](docs/PFLICHTENHEFT.md) | Geplante technische Erfüllung und Nachweise („Wie?“) |
| [Protokolle](docs/PROTOKOLLE.md) | UCI, Match-Controller, FEN/PGN, Datenverträge |
| [Training](docs/TRAINING.md) | Datensätze, Trainingslauf, NNUE, Modellpromotion |
| [Datenquellen](docs/DATENQUELLEN.md) | Schachpartien, Bewertungsdaten, Provenienz und Lizenzen |
| [Testkonzept](docs/TESTKONZEPT.md) | Qualitätsgates, Testfälle und Plattformmatrix |
| [Architekturentscheidungen](docs/adr/README.md) | Einzelne ADRs und Entscheidungsstatus |
| [Contributing](CONTRIBUTING.md) | Mitarbeit, Branches, Review- und Änderungsregeln |
| [Agent-Anweisungen](AGENTS.md) | Repositoryweite Regeln für Coding Agents |
| [Security](SECURITY.md) | Sicherer Umgang mit Dateien, Prozessen und Meldungen |
| [Changelog](CHANGELOG.md) | Dokumentierte Änderungen |

Die Schreibweise `AGENTS.md` ist das maßgebliche Konventionsformat für Coding Agents; [`AGENT.md`](AGENT.md) ist eine Weiterleitung.

## Geplante Programme und Schnittstellen

- **`sasd-ai-chess`**: lokale, headless UCI-Engine (C++20); arbeitet ohne Python/GPU.
- **`sasd-ai-chess-match`**: Match-Controller mit zwei separaten UCI-Prozessen (auch zweimal unsere eigene Engine).
- **`sasd-ai-chess-dataset`**: PGN-/Bewertungsdaten-Importer mit Streaming, Validierung und versioniertem Export.
- **Python-Trainingspaket**: PyTorch-Training, Modelltests, Export und Auswertung.

**UCI** ist für den aktiven Zugtausch zwischen Engines gedacht; **PGN** ist für gespeicherte Partien. Beides wird benötigt und nicht verwechselt.

## Entwicklungsgrundsätze

Saubere Architektur und korrekte Schachregeln vor Optimierungen; verständliche **englische Doxygen-/Inline-Codekommentare**, deutschsprachige Fach- und Architekturunterlagen; nachvollziehbare Git-Commits, Tests und Experimente; keine ungeprüften Bibliotheks-/Datendownloads. Der aktuelle MIT-Lizenztext liegt in [LICENSE](LICENSE); Fremdcode und Fremddaten besitzen **eigene** Lizenzbedingungen.

**Arbeitsfortschritt:** [ROADMAP.md](ROADMAP.md) ist maßgeblich; GitHub [Issue #1](https://github.com/Robin-Goerlach/SASD-AI-Chess/issues/1) beschreibt den geplanten PGN-/Trainingsdatenimport.
