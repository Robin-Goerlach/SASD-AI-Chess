# Changelog – SASD-AI-Chess

Änderungen an Quellcode, Schnittstellen, Dokumentation und Meilensteinstatus werden nachvollziehbar dokumentiert. Format angelehnt an *Keep a Changelog*; bis zur ersten Version existieren **keine freigegebenen Engine-Versionen**.

## [Unreleased]

### Added – 2026-10-27

- Initiales Repository mit README und MIT-Lizenz.
- Deutsche modulare [Architekturbaseline](docs/ARCHITEKTUR.md) für Chess Core, UCI, Match-Controller, PGN/Datasets und KI-Training.
- Dokumentationspaket: [Roadmap](ROADMAP.md), [Agentenregeln](AGENTS.md), [Lastenheft](docs/LASTENHEFT.md), [Pflichtenheft](docs/PFLICHTENHEFT.md), [Testkonzept](docs/TESTKONZEPT.md), [Protokolle](docs/PROTOKOLLE.md), [Training](docs/TRAINING.md), [Datenquellen](docs/DATENQUELLEN.md), [ADR-Index](docs/adr/README.md), Beitragshinweise und Security-Policy.
- PGN-Import-Anforderung in [Issue #1](https://github.com/Robin-Goerlach/SASD-AI-Chess/issues/1) erfasst.
- Ausführliche projektweite [README.md](README.md) mit Systemüberblick, UCI-/PGN-/KI-Konzepten, geplantem Buildablauf und ausdrücklich gekennzeichnetem Implementierungsstand.
- Eine in die README eingebettete, skalierbare SVG-Konzeptgrafik der zukünftigen Desktop-Oberfläche unter `docs/images/sasd-ai-chess-ui-concept.svg`.

### Not yet implemented

- CMake-Build, Chess Core, Engine-UCI, Match-Controller, PGN-Importer, neuronale Inferenz, Training und CI. Dokumentation ersetzt **keine** Implementierung.

## Pflege

Bei jeder relevanten Codeänderung Eintrag unter `Unreleased` ergänzen. Releaseeinträge erst nach realer Freigabe anlegen; jedes Release mit nachgewiesenen Test-, Build- und Kompatibilitätsergebnissen verknüpfen.
