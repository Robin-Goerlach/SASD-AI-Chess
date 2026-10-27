# ADR-003 – PGN-Import und versionierte Trainingsdaten

- **Datum:** 2026-10-27
- **Status:** Accepted (Anforderungs-/Architekturentscheidung, noch nicht implementiert)
- **Referenzen:** [Datenquellen](../DATENQUELLEN.md), [Training](../TRAINING.md), [Issue #1](https://github.com/Robin-Goerlach/SASD-AI-Chess/issues/1)

## Kontext

Das System soll online verfügbare Partien **herunterladen können, wenn Nutzer dies wünschen**, und lokal aus PGN wiedergeben und Trainingsmaterial generieren. PGN ist menschenlesbar und verbreitet, aber nicht für schnelle ML-Batchzugriffe oder wissenschaftliche Provenienz ausgelegt. Daten enthalten Fehler, Dubletten, Varianten und unterschiedliche Lizenzbedingungen.

## Entscheidung

- **PGN** als Austauschformat für gespeicherte Partien; **FEN/SAN** zur regelkonformen Rekonstruktion; UCI bleibt separate Runtime-Schnittstelle.
- Streaming-Eingabe für `.pgn` und `.pgn.zst`, optional gesonderter Lichess-Bewertungsdatenadapter.
- Eingaben durch gemeinsamen Chess Core validieren; defekte Partien in Quarantäne, mit nachvollziehbarem Fehlerbericht.
- **Versionierte, unveränderliche interne Datasets** mit Manifest (Schema, Quellen, Lizenz, Hash, Selektion, Split, Zähler), Deduplizierung und Leakageschutz. Große Archive und Modelle werden nicht eingecheckt.
- Lokale Dateisystem-Ablage am Anfang; SQLite-Katalog oder Binär-/Parquet-Shards erst nach begründeter Auswahl.

## Alternativen

- **Nur PGN direkt beim Training lesen:** redundante Parserkosten, schwierige Split-Reproduzierbarkeit und keine konsistente Datenherkunft.
- **Sofort zentrale Datenbank / Cloud:** unnötige Infrastruktur und Netzwerkabhängigkeit.
- **Training ohne Legalitätsprüfung:** Gefahr, fehlerhafte Stellungen und Ergebnisse zu erlernen.

## Konsequenzen / Tests

Datenstand ist durch Manifest reproduzierbar; Parser benötigt Ressourcenlimits; Lizenz- und Score-Perspektiven-Audit. Test: TC-DATA inkl. defekte PGNs, zstd, Dubletten und Split-Clustering.

**Offen:** genaues Shard-/Katalogschema und Bibliothekswahl → separater ADR vor Implementierung.
