# Contributing to SASD-AI-Chess

Vielen Dank für Beiträge! Dieses Projekt priorisiert **korrekte Schachregeln, verständliche Architektur und reproduzierbare KI-Forschung**. Siehe [AGENTS.md](AGENTS.md) auch für detaillierte technische Vorgaben.

## Vor einer Änderung

1. [ROADMAP.md](ROADMAP.md) und [docs/ARCHITEKTUR.md](docs/ARCHITEKTUR.md) lesen.
2. Bestehende Issues, insbesondere [PGN/Dataset-Import #1](https://github.com/Robin-Goerlach/SASD-AI-Chess/issues/1), auf Überschneidungen prüfen.
3. Bei größeren Änderungen zuerst eine knappe Problemdefinition, Alternativen, Abnahmetests und ggf. ADR festhalten.
4. Tatsächlichen Repo-/CI-Stand feststellen; in der Planungsphase sind Build- und Testbefehle **noch nicht zwingend ausführbar**.

## Entwicklungsprinzipien

- **Klein und überprüfbar:** ein in sich stimmiger Slice mit Quellcode, Tests, Doku.
- **Architektur:** Chess Core unabhängig von Trainings-/UI-/UCI-/Dateisystemdetails; UCI über separaten Adapter; Match-Controller als Prozess.
- **C++20/CMake**, möglichst wenige, gepflegte Abhängigkeiten; Python/PyTorch getrennt für Training.
- **Codekommentare und Doxygen auf Englisch**, Architektur/Fachtexte auf Deutsch.
- **Erst korrekt und kompilierbar, später optimieren.** Komplexe Heuristiken brauchen reproduzierbare Benchmarks.
- **Git:** bestehende Haupt-/Feature-Branch- und Reviewregeln gelten; keine fremden Commits umschreiben. Commit-Nachrichten beispielsweise `feat(core): add en passant legality tests` oder `docs: clarify dataset split contract`.

## Tests bei Änderungen

| Geänderte Komponente | Erwarteter Nachweis |
| --- | --- |
| Chess Core | Unit + FEN-/Legalitätstests und relevante Perft-Stellungen |
| Search/Evaluation | deterministische Basisfälle, Such-Stop, Regression-/Performancevergleich |
| UCI | Handshake, Such-/Abbruchprotokoll, ungültige Befehle, kein Debug auf stdout |
| Match | zwei unabhängige UCI-Prozesse, Zeit-/Absturz-/Illegalitätsfälle, PGN-Endergebnis |
| PGN/Datasets | gültige/ungültige PGNs, komprimierter Stream, Lizenz/Manifest, Dubletten und Split |
| Training/Modelle | Mini-Trainingslauf, Seeds, Feature-Parität, Export-Import und Holdout |
| Dokumentation | Links, IDs und Status gegen realen Ist-Zustand prüfen |

Keine erfundenen grünen Tests. Wenn eine Plattform nicht verfügbar ist, im Beitrag kenntlich machen, einschließlich noch ausstehender Checks.

## Pull-Request-Checkliste

- [ ] Nutzeranforderung / Issue und Architekturbezug genannt
- [ ] Schnittstellen/Fehlerfälle dokumentiert und englisch kommentiert
- [ ] Tests ergänzt und mit konkretem Resultat genannt
- [ ] Lizenz und Herkunft neuer Abhängigkeiten/Daten geprüft
- [ ] Roadmap/Changelog bei tatsächlichem Fortschritt angepasst
- [ ] Keine Geheimnisse, fremden Großarchive oder Modell-Binärgewichte committet
- [ ] Performance- oder Elo-Aussagen mit Messmethode und Unsicherheit belegt

## Lizenz

Eigene Beiträge zum Repository folgen der vorhandenen [MIT-Lizenz](LICENSE), sofern nicht anders vereinbart. Drittanbieter-Datensätze, Schachengines oder Modelle können andere Bedingungen besitzen; deren Lizenzen und Herkunft nicht entfernen.

## Security

Sicherheitsrelevante Probleme bitte nach [SECURITY.md](SECURITY.md) behandeln; vertrauliche Sicherheitslücken nicht unnötig in öffentliche Issues schreiben.
