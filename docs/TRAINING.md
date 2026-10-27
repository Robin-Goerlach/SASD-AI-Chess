# SASD-AI-Chess – Trainings- und Experimentkonzept

**Version:** 0.1 · **Stand:** 27.10.2026 · **Status:** Forschungs-/Umsetzungsplan, nicht ausführbar  
**Bezug:** [Architektur](ARCHITEKTUR.md) · [Roadmap M4–M7](../ROADMAP.md) · [Datenquellen](DATENQUELLEN.md).

## 1. Lernziele strikt auseinanderhalten

| Lernaufgabe | Eingaben | Ziel / Label | Grenzen |
| --- | --- | --- | --- |
| **Zugimitation** | Stellung und gespielter Zug aus PGN | Wahrscheinlichkeit des gespielten legalen Zuges | gespielter Zug ist nicht unbedingt optimal |
| **Value-/NNUE-Training** | codierte Stellung mit Qualitätsmerkmalen | konsistente CP-/Mate-/WDL-Zielwerte aus geprüfter Quelle | Quelle, Tiefe, Perspektive und Ergebnis müssen dokumentiert sein |
| **Selbstspiel/iteratives Lernen** | aktuelle Engine- und Modellversion | neue Spiele, Resultate und analysierte Stellungen | selbst erzeugte Daten garantieren keinen Fortschritt |

**Baseline-Vorschlag:** Start mit klassischer Stellungsbewertung und Alpha-Beta-Suche. Danach **kleines NNUE-artiges Value-Modell** auf CPU; Policy+Value/MCTS als späteres unabhängiges Experiment, nicht als versteckte V1-Pflicht.

## 2. Datenpipeline und Qualitätskontrolle

```text
PGN/.pgn.zst oder FEN-Evaluationsdaten
      ↓
Unveränderte Quelle + Hash + Lizenz
      ↓
Stream → Parse → Legalitäts-/Variantenprüfung → Quarantäne bei Fehler
      ↓
GameRecord / PositionRecord / LabelRecord mit Provenienz
      ↓
Deduplizierung + Filter + gruppierte Train/Validation/Test-Zuordnung
      ↓
Versionierte Shards + DatasetManifest
      ↓
Feature Encoder (feste Version)
      ↓
Trainings-/Validierungs- und Testlauf
```

**Kein Leakage:** Splits auf Partie-/Duplikatcluster-Ebene **vor** dem Sample-Schreiben zuordnen; bekannte identische Positionen und gleiche Eröffnungssequenzen kontrollieren. Ein zusätzlicher zeitlich oder quellenbezogen unabhängiger Holdout misst echte Generalisierung. Nicht nur zufällig einzelne Halbzüge auf Trainings-/Testgruppen verteilen.

**Labelperspektive:** Ein `cp`-Wert kann aus Sicht von Weiß oder der Seite am Zug definiert sein. Erst Quellsemantik aufnehmen, dann einheitlich normalisieren; Mate-Limits und WDL-Probabilitäten **nicht** unkontrolliert mit CP addieren. Fehlende Bewertungen nicht als null deklarieren. Für Imitation dürfen auch Partien ohne Engineanalyse enthalten sein; Value-Training benötigt geeignete Labels.

**Datenquellen:** bevorzugt Lichess Open Database mit Lizenzprüfung je Datensatz. Fremd-PGN und Stockfish-analysierte Positionen sind *Lehrmaterial*, Stockfish ist **keine notwendige Runtime-Abhängigkeit**. Datenmengen zunächst klein halten (100–10.000 Partien für Pipelineprüfung).

## 3. Modelldesign und Featurevertrag

Für die erste NNUE-Version vorgesehen:

- Eingabe: legaler Chess-Core-Zustand plus wohldefiniertes Feature-Encoding; Kontext (Zugrecht, Königszonen/Material) gemäß dokumentiertem `feature_schema_version`.
- Output: Bewertung nach intern einheitlicher Perspektive, ggf. kalibrierte WDL; CPU-Auswertung in C++ ohne Python.
- Python-Training: PyTorch, fest definierte Aktivierungs-/Lossfunktion, Seeds, Mini-Batches, Early Stopping, trainierbare Hyperparameter.
- Export: versioniertes Format, Gewichts-/Schemahash, Dimensionen, Quantisierung, Testvektoren. Kein Pickle-/Python-Objektloader in der Engine.
- Klassischer Evaluator bleibt als unabhängige Vergleichsbasis erhalten.
- Ein Modelltausch darf während einer Partie / laufenden Suche nicht unbemerkt erfolgen.

**Cross-Language-Test:** FEN → Feature-Encoder in Python und C++ muss bitgenau (oder gemäß expliziter Spezifikation) übereinstimmen. Python-Float- und C++-Quantisierungs-Outputs werden mit dokumentierter Fehlertoleranz verglichen.

## 4. Experimentmanifest (inhaltlicher Entwurf)

```json
{
  "schema_version": 1,
  "experiment_id": "example-not-a-real-run",
  "source_commit": "git-sha-placeholder",
  "dataset_id": "dataset-sha-placeholder",
  "model_architecture": "nnue-small-proposal",
  "feature_schema_version": 1,
  "random_seed": 12345,
  "label_type": "value",
  "label_perspective": "side_to_move",
  "splits": ["train", "validation", "test"],
  "hardware": "to-be-recorded",
  "training_metrics": {},
  "game_metrics": {},
  "status": "example_only"
}
```

Ein tatsächlicher Run erfasst zusätzlich Bibliotheks-/Compilerversionen, GPU/CPU, Konfiguration, Lernrate, Batchgröße, Epochanzahl, Quantisierungsprofil, Eingabe-/Ausgabedateihashes, Wand-/CPU-Zeit, Loss-/WDL-Kalibrierung und Test-Matches. Seed garantiert allein keine bitgenaue Reproduktion bei jeder Parallel-/GPU-Konfiguration.

## 5. Modellfreigabe: offline ≠ stärker im Spiel

**Phase A – Formale Validierung:** Dataset geprüft; Schema/Modellformat passt; Export- und C++-Inferenzparität; Modelldatei unbeschädigt.  
**Phase B – Offline-Qualität:** Holdout-Loss, Kalibrierung, systematische Fehler für Stellungen verschiedener Phasen; Baselinevergleich.  
**Phase C – Engine-Eigenschaften:** Laufzeit (Inferenz pro Stellung), Speicherbedarf, Nodes/s, Such-/Abbruch-Stabilität.  
**Phase D – Spielstärke:** feste Eröffnungs-Suite mit Farbtausch, gleiche Bedenkzeit und Hardware, Vergleich gegen vorige freigegebene Version; Elo-/Score-Unsicherheit oder vorher definierter sequentieller Test.

**Promotion:** Kandidat erhält Status `candidate`. Freigabe zu `approved` nur bei erfüllten vorher definierten Gates. Ein Kandidat mit niedrigerem Loss, aber schwächerer Matchperformance bleibt **abgelehnt oder unentschieden**, nicht automatisch produktiv.

## 6. Selbstspiel-Zyklus und Gegenmaßnahmen

```text
Approved Model A → zwei UCI-Instanzen
        ↓
Match-Controller → eigene PGN + Ergebnis-/Zeitprotokolle
        ↓
Validierung, Diversity-Filter, Quelle/Modellhash
        ↓
Neue Dataset-Version → Python-Training → Candidate B
        ↓
Independent Holdout + Paired Matches
        ├── erfolgreicher Nachweis → Freigabe
        └── unklar/schlechter → Archivierung + Analyse
```

**Risiken:** einseitige Eröffnungen, Trainings-Kollaps, Stellungen mit falschen Labels, selbstverstärkende Fehler, Verlust von Diversität, Hardware-/Zufallseffekte. Gegenmaßnahmen: Eröffnungsrotation, Farbtausch, mehrere Seeds, unabhängiger Holdout, regelkonformer Partienimport und unveränderliche Quellen-/Artefakt-IDs.

## 7. Zielbetrieb und Grenzen

Zuerst lokal/offline, mit kleinen verifizierten Datasets. Größere Trainingsläufe, GPU und verteilte Erzeugung sind **spätere Mess-/ADR-getriebene Optionen**. Training kann mit Stockfish-Bewertungen unterstützt werden, ohne Stockfish-Code zu kopieren. Fremddaten und Modelle bleiben lizenzrechtlich eigenständig.

**Abnahme:** testbares kleines Trainings-/Exportbeispiel (TC-ML), nachvollziehbarer Run und tatsächlicher Vergleich (TC-EVAL). Am Dokumentationsstand liegt **noch kein Trainingsergebnis** vor.
