# SASD-AI-Chess – Datenquellen, Lizenzen und Provenienz

**Stand:** 27.10.2026 · **Status:** Quellenkatalog und Importpolitik, noch keine importierten Daten  
**Zugehörige Anforderungen:** [Lastenheft LR-F08..F10, F16..F17](LASTENHEFT.md), [Issue #1](https://github.com/Robin-Goerlach/SASD-AI-Chess/issues/1).

## 1. Öffentliche Datenquellen (Beispiele)

| Quelle | Datentyp | Format | Lizenz-/Prüfhinweis |
| --- | --- | --- | --- |
| [Lichess Open Database](https://database.lichess.org/) | Standardpartien (ausgewählte Monate) | komprimiertes PGN (`.pgn.zst`) | Lichess Standardspiele i. d. R. **CC0**; die konkrete Sammlung und Lizenz vor Weitergabe prüfen |
| [Lichess Evaluations](https://database.lichess.org/) | mit Stockfish bewertete Stellungen | komprimiertes JSONL | Quelle/Score-Schema/CC0-Angaben je Sammlung prüfen |
| [Lichess Broadcasts](https://database.lichess.org/) | Turnier-/Broadcastpartien | PGN | nicht mit Standardspielen gleichsetzen; **CC BY-SA 4.0** kann gelten |
| [The Week in Chess](https://theweekinchess.com/twic) | Turnierpartien | PGN | Nutzungsrechte und Weiterverbreitung **gesondert** prüfen |
| Eigener UCI Match Controller | selbst erzeugte Partien | PGN + JSON/Manifest | Erzeugungs-/Enginebedingungen und eventuelle Drittanbieterrechte festhalten |

**Wichtig:** „kostenlos herunterladbar“ bedeutet **nicht** „beliebig weiterverbreitbar“. Der eigene Quellcode steht unter MIT; die Lizenz einer importierten PGN-Datei, eines Modells oder von Fremdcode ändert sich dadurch nicht. Keine Lizenz aus einem Quellnamen ableiten, sondern zum Zeitpunkt des Downloads dokumentieren.

## 2. Importpflichten

Pro Datei/Quelle mindestens: Herkunft/URL, Abrufdatum, Quelle/Edition, ausdrücklich angegebene Lizenz inkl. Link/Version, ursprünglicher Dateiname/Format, Dateigröße, SHA-256, lokale unveränderte Kopie, Importkonfiguration, Parser-/Schachkernversion, Fehler-/Akzeptanzstatistik.

**PGN:** Tags und SAN-Hauptvariante, Start-FEN, Ergebnis, Elo, Variante, Bedenkzeit erhalten, soweit vorhanden. Fehlende Werte als **null/unbekannt**. Defekte/nicht unterstützte Varianten nicht stillschweigend korrigieren. **Nie** eine PGN aus unbestätigten Quellen als menschlich „perfekten“ Lehrsatz behandeln.

**Bewertungen:** Scores können Centipawns, Matt-Entfernungen oder WDL enthalten. Immer Perspektive (Weiß oder side-to-move), Engine-Version, Suchtiefe/Knoten und Schema festhalten; erst danach harmonisieren. Eine PGN-Partie ist nicht dasselbe wie eine Stockfish-Bewertung.

## 3. Speicher-/Versionspolitik

```text
data/
  raw/                  # unveränderte externe Quellen (nicht committen)
  catalog/              # Quelle/Lizenz/Import-Provenienz
  datasets/<id>/         # Shards + manifest.json
  models/<id>/           # Gewichte + Modellmanifest
  experiments/<id>/      # Logs, Runs, Partieergebnisse
```

Diese Pfade sind **geplant**, nicht aktuell vorhandene Verzeichnisse. Große Daten/Modelle werden über externe Artefaktspeicher oder lokale Pfade verwaltet, **nicht** per Git. Hashbasierte IDs und manifeste Schema-Versionen sorgen für Nachprüfbarkeit.

**Datenschutz:** Öffentliche PGN-Daten können Spielernamen/Pseudonyme enthalten. Personenbezug und Weitergabe sind bewusst zu prüfen; Trainingsdatensätze nur soweit erforderlich erzeugen, keine persönlichen Zusatzdaten unreflektiert anreichern.

## 4. Empfehlung für den ersten Datentest

- **100 rechtlich unbedenkliche, kleine PGN-Partien** für Parser-/FEN-/SAN-/Replay-Regressionsfälle.
- Danach **10.000 Partien** einer dokumentierten Lichess-Standardsammlung mit festgelegter Selektion, um Deduplizierung, Filtern, Importstatistiken und Splitlogik zu prüfen.
- Erst nach verlässlichem Import **bewertete Stellungen** für NNUE-Value-Training einsetzen; reine Spielzüge dürfen zunächst für Imitation dienen.
- Eigene selbstgespielte Partien **erst nach** überprüftem UCI-Match-Controller als Trainingsquelle zulassen.

**Weitere technische Festlegungen:** [TRAINING.md](TRAINING.md), [PROTOKOLLE.md](PROTOKOLLE.md), [TESTKONZEPT.md](TESTKONZEPT.md).
