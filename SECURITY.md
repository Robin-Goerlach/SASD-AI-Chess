# Sicherheit – SASD-AI-Chess

**Projektphase:** Architektur und Dokumentation; es gibt zum Stand 27.10.2026 noch **keine veröffentlichte, ausführbare Engine**. Diese Regeln betreffen geplante Parser, Trainingswerkzeuge und Match-Prozesse.

## Sicherheitsmeldung

Vertrauliche Sicherheitsprobleme bitte **nicht** mit Exploitdaten in einem öffentlichen Issue melden. Wenn GitHub beim Repository die Funktion **„Report a vulnerability“ / Private Vulnerability Reporting** anbietet, diese nutzen. Andernfalls zuerst über einen geeigneten privaten Kanal des Repository-Inhabers Kontakt aufnehmen; ohne etablierten sicheren Kontakt keine Zugangsdaten oder vertrauliche Informationen senden. Es gibt aktuell keine zugesagte Reaktionsfrist.

## Vertrauensgrenzen

- **PGN/FEN/SAN/JSONL** stammen oft aus dem Internet: Größenlimits, robuste Parser, Zeichenkodierungsprüfung, Dekompressionsbegrenzung, Quarantäne defekter Einträge; niemals als Code ausführen.
- **Fremde UCI-Engines** sind Programme mit Betriebssystemrechten: nur explizit konfigurierte ausführbare Pfade, Argumentvektoren ohne Shell-Kommandozusammenbau; Timeouts, Prozessgruppen, kontrollierte Beendigung und begrenzte Puffer.
- **UCI stdout** ausschließlich protokollkonform; stdout/stderr getrennt konsumieren, sonst Deadlock-Risiko.
- **Modelle/Artefakte** nur mit erlaubter Schema-Version, Größenprüfung, Checksums und vertrauenswürdigem Exportformat laden; **kein automatisches ungesichertes Deserialisieren ausführbaren Python-Codes**.
- **Training und Experimente** lokal/offline möglich; keine verpflichtenden Netzwerkdienste; Remote-Worker später ausschließlich nach Authentisierung und Sicherheits-ADR.
- **Abhängigkeiten/Lizenzen** mit Bezugsquelle und Version festhalten; keine geheimen Tokens, Trainingsarchive oder privaten Dateipfade in Commits.

## Sicherheitsprüfungen

Fuzz-/Negativtests für FEN/PGN/UCI/Modelldateien, Zeitlimits/Absturztests für Match-Prozesse, Speicherauslastung bei ZSTD-Archiven und reproduzierbare Build-Checks gehören in [docs/TESTKONZEPT.md](docs/TESTKONZEPT.md). Unterstützte Versionen werden erst mit tatsächlichen Releases festgelegt.
