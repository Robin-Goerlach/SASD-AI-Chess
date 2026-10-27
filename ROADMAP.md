# SASD-AI-Chess – Entwicklungsroadmap

**Status:** verbindliche Planungsgrundlage, Version 0.1 · **Stand:** 27.10.2026  
**Repository:** https://github.com/Robin-Goerlach/SASD-AI-Chess  
**Basis:** [Architektur](docs/ARCHITEKTUR.md), [Lastenheft](docs/LASTENHEFT.md), [Pflichtenheft](docs/PFLICHTENHEFT.md) und [Testkonzept](docs/TESTKONZEPT.md).

## 1. Zielbild und Prioritäten

SASD-AI-Chess soll **eigenständig Schach spielen**, über **UCI** mit externen Engines kommunizieren, **zwei Instanzen gegeneinander** antreten lassen, **PGN-Partien laden und daraus lernen**, neuronale Bewertungen ausführen und Verbesserungen reproduzierbar messen.

**Nicht verhandelbare Reihenfolge:** korrekter Chess Core → spielbare Baseline/UCI → stabiler Match-Controller → verlässlicher Partienimport → NNUE-Training → Evaluation/Selbstspiel. Funktionierende, verständliche Implementierung **vor** Optimierung. Jede neue Ebene muss die vorhandene Engine weiterhin ohne Python oder GPU lauffähig lassen.

**Definitionen:** ⬜ geplant · 🟡 begonnen/teilweise · ✅ nachweislich abgeschlossen · ⛔ blockiert. **Nur abgelegte, überprüfbare Resultate rechtfertigen ✅.** Zeitangaben sind bewusst nicht festgeschrieben; Abnahmen sind wichtiger als spekulative Termine.

## 2. Meilensteinübersicht

| ID | Status (27.10.2026) | Meilenstein | Abhängigkeit | Fertig, wenn ... |
| --- | --- | --- | --- | --- |
| **M0** | 🟡 | Projekt- und Architekturgrundlage | keine | CMake/CI, Repo-Konventionen, minimale Bibliothek/Tests und Dokumentation tatsächlich vorhanden |
| **M1** | ⬜ | Schachregeln / Chess Core | M0 | Perft, Legalitäts- und Rücknahme-Tests bestanden |
| **M2** | ⬜ | UCI-Engine und klassische Suche | M1 | UCI-Handshake, gültige `bestmove`, Stop und Grundsuche getestet |
| **M3** | ⬜ | Zwei-Engine-Matches | M2 | zwei eigene Prozesse können regelkonform mit Uhr spielen und PGN exportieren |
| **M4** | ⬜ | PGN-Import / Dataset-Pipeline | M1, M3 für Selbstspieldaten | großer PGN-Bestand reproduzierbar importierbar, splitbar und versionierbar |
| **M5** | ⬜ | Erstes trainiertes NNUE-artiges Modell | M2, M4 | PyTorch-Training, C++-Inferenzparität, Modell lädt korrekt |
| **M6** | ⬜ | Wissenschaftliche Evaluation / Modellfreigabe | M3, M5 | gepaarte Vergleichspartien, Statistik, reproduzierbare Experimentartefakte |
| **M7** | ⬜ | Selbstspiel und Skalierung | M4–M6 | geschlossener, protokollierter Trainingszyklus ohne stille Modellübernahme |

### M0 – Fundament (teilweise begonnen)

**Bereits vorhanden:** MIT-Lizenz, README, Architekturgrundlage, ergänzende Planungs-/Engineering-Dokumente. Dies ist **noch kein lauffähiges Programm**.

**Arbeitspakete:**
- M0.1 Ziel-/Ist-Struktur und CMake-Projekte für Engine, Core, Testprogramme und spätere Tools anlegen.
- M0.2 C++20-Toolchainmatrix und Compiler-Warnungen; getrennte Debug-/Release-Konfigurationen.
- M0.3 Kleine, isolierte Core-Schnittstellen, Testframework-Entscheidung und ein erster ausführbarer Smoke-Test.
- M0.4 CI für verfügbare Plattformen; BSD-Builds mit dokumentierten realen Ergebnissen statt behaupteter Unterstützung.
- M0.5 Changelog, ADR-Verfahren und reproduzierbare Entwickleranleitung pflegen.

**Exit:** frischer Checkout konfiguriert, kompiliert und führt wenigstens einen echten Test aus. Compilerlogs/CI-Links dokumentiert. Keine fiktiven grünen Builds.

### M1 – Chess Core

**Arbeitspakete:**
- M1.1 Typen `Color`, `Piece`, `Square`, `Move`, unveränderliche API-Konventionen.
- M1.2 `Position` mit Zugrecht, Rochade, en passant, Halbzug- und Zugzähler; FEN-Parser/-Serializer.
- M1.3 Legale Zugerzeugung (inkl. Rochade, en passant, Umwandlung, Schachschutz).
- M1.4 Zustandsänderung und reversible `make/unmake`-Semantik für schnelle Suche.
- M1.5 Partiehistorie, Wiederholung, Matt, Patt, beanspruchbare/automatische Remisregeln.
- M1.6 SAN-Zugnotation und Referenz-`perft`.

**Exit:** alle bekannten Teststellungen und kniffligen Sonderregeln bestehen Unit-/Differentialtests; kein KI-Modul erforderlich. Keine Optimierung darf die Legalität ändern.

### M2 – Spielbare Engine/UCI

**Arbeitspakete:** UCI-Parser/Dispatcher; `uci`/`isready`/`ucinewgame`/`position`/`go`/`stop`/`quit`; erste legale Zugauswahl, klassische Evaluation; iterative Vertiefung, Alpha-Beta, Transposition Table und Zeit-/Abbruchkontrolle; keine Trainingslaufzeit-Abhängigkeit.

**Exit:** UCI-Client kann einen legalen Zug von der Engine empfangen, Positionen setzen, Suche stoppen und den Prozess ordnungsgemäß beenden; UCI-`stdout` enthält keine Debug-Ausgaben. Zuerst korrekt, dann stärker.

### M3 – Match-Controller

**Arbeitspakete:** Start zweier Prozesse; Handshake/Fehlerfälle; monotone Uhren; alternierende Stellungen mit vollständiger Historie; Validierung jedes Zuges; Zeitüberschreitung/Absturz/illegaler Zug; gepaarte Farben/Eröffnungen; PGN-/JSON-Ergebnis.

**Exit:** zwei SASD-AI-Chess-Prozesse spielen eine komplette regelkonforme Partie, einschließlich erzwungener End- und Fehlerfälle. Gegenprobe mit unabhängiger UCI-Engine, wenn lokal vorhanden.

### M4 – PGN/Trainingsdaten

**Arbeitspakete:** streamingfähiger PGN-Parser; SAN/FEN-Startstellungen; Kommentare/Varianten sicher behandeln; zstd-Dekompression; Datenvalidierung/Quarantäne; Importfilter, Provenienz/Lizenz, Deduplizierung, gruppierte Train/Validation/Test-Splits, versioniertes Dataset-Manifest; optional Bewertungsdatenadapter.

**Exit:** kleine Testdatei exakt rekonstruiert; größere komprimierte Sammlung ohne unbeschränkten RAM importiert; defekte Partien sauber gemeldet; Dataset wiederholbar. Details: [Issue #1](https://github.com/Robin-Goerlach/SASD-AI-Chess/issues/1).

### M5 – ML-Bewertung V1

**Arbeitspakete:** versioniertes Feature-Encoding; Referenzlabels (z. B. geprüfte Engine-Bewertungen/WDL); Python/PyTorch-Trainer; kleiner NNUE-artiger Evaluator; Export/Quantisierung; C++-Loader; Inferenzparitäts-Tests gegen Python; klassische Baseline beibehalten.

**Exit:** C++ kann ein eigenes, validiertes Modell offline auswerten. Dokumentation von Training Loss, Holdout-Metriken, Latenz und Modellgröße. **Noch kein behaupteter Elo-Gewinn ohne Matches.**

### M6 – Evaluationslabor

**Arbeitspakete:** feste Opening-Suite mit Farbtausch; Spielzeit- und Hardwarekontrollen; Rating-/Konfidenzauswertung; Regression-Benchmarks; Modellmanifest und -register; Freigabe-/Ablehnungsregeln.

**Exit:** Wiederholung eines Experiments anhand gespeicherter Konfiguration, Daten-/Modellhashes und Match-Protokolle möglich. Kandidaten werden nach **vorher definierten** Kriterien bewertet.

### M7 – Selbstspiel und mehr Rechenleistung

**Arbeitspakete:** selbst erzeugte PGNs einspeisen; Wiederholungsschleife Modell → Match → Daten → Training → Kandidat → Prüfung; ausgewogene Positionsauswahl; Worker-Queue/Checkpointing, bei Bedarf getrennte Remote-Worker.

**Exit:** mindestens ein vollständiger, protokollierter Zyklus; schwächere Kandidaten können bewusst abgelehnt werden; keine von externen Diensten abhängige Engine.

## 3. Abhängigkeiten und kritischer Pfad

```text
M0 → M1 → M2 → M3 → M6 → M7
       └────→ M4 → M5 ─┘
              ↑      ↑
       PGN braucht M1; NNUE braucht M2/M4.
```

**Parallelarbeit:** PGN-Spezifikation und Trainings-Experimente können früh vorbereitet werden. **Implementierung** des Importers muss denselben geprüften Chess Core verwenden; vor M1 keine zweite, abweichende Zugregelimplementierung erstellen. M6-Planung beginnt früh, belastbare Elo-/Freigabetests erst nach Match-Controller.

## 4. Versions- und Releasepolitik (Vorschlag)

- `0.1.x`: Architektur, Infrastruktur, Core-Prototypen (noch keine Featuregarantien).
- `0.2.x`: nachgewiesene regelkonforme Core- und UCI-Basis.
- `0.3.x`: Match-Controller und erste PGN-Importe.
- `0.4.x`: Trainingsdatenpipeline und NNUE-Versuch.
- `1.0`: erst nach dokumentierten M1–M6-Abnahmen, reproduzierbarem Build und klaren Limitierungen.

**Hinweis:** Dies sind Zielmarken, keine veröffentlichten Releases oder zugesagte Zeitpunkte.

## 5. Änderungsmanagement und Qualität

Jeder Arbeitsslice besitzt Issue/Anforderung, kleine nachvollziehbare Änderung, passenden Test, Doku/Changelog-Anpassung und kurzen Ergebnisbericht (**durchgeführt / nicht durchgeführt / bekannte Einschränkungen**). Änderungen an Modulgrenzen, Daten-/Modellformat oder Lizenzstrategie benötigen vorher ein [ADR](docs/adr/README.md). Große Modelle/Partiensammlungen gehören **nicht ins Git-Repository**.

**Aktuell nächster tatsächlicher Entwicklungsschritt:** M0.1–M0.3: CMake-Projekt, Chess-Core-Schnittstellen und erster real ausführbarer Test. Die Roadmap wird nach *echter* Umsetzung mit Testnachweisen aktualisiert.
