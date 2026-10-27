# ADR-004 – Klassische Engine vor kleinem NNUE-artigem Modell

- **Datum:** 2026-10-27
- **Status:** Proposed (technisch bevorzugter Lösungsweg; Modell-Feindesign offen)
- **Referenzen:** [Training](../TRAINING.md), [Roadmap](../../ROADMAP.md)

## Kontext

Die Engine soll **schnell spielbar und reproduzierbar testbar** sein, obwohl KI-Training erst nach geprüften Schachregeln, Match- und Importinfrastruktur sinnvoll wird. Eine reine Deep-Learning-Engine könnte große Trainingshardware und eine andere Sucharchitektur voraussetzen.

## Vorgeschlagene Entscheidung

1. Klassische Stellungsbewertung + iterative Alpha-Beta-Suche als verständliche, eigenständig lauffähige Baseline.
2. Trennung des Bewertungsverfahrens über `IPositionEvaluator`.
3. Danach ein **kleines, CPU-taugliches NNUE-artiges Value-Modell** aus Python exportieren und in C++ inferieren; kein PyTorch in der Engine.
4. Ein Modell nur nach Inferenzparität, Ressourcen- und **gepaarten Spielstärketests** freigeben. Classical bleibt als Rückvergleich verfügbar.
5. Größere Policy+Value-/MCTS-Forschung später über eigene ADRs.

## Alternativen

- **Nur klassische Engine:** schnell und wartbar, erfüllt aber nicht das Ziel einer trainierten neuronalen Bewertung.
- **Nur großes GPU-Netz mit MCTS:** höherer Forschungs-/Hardware- und Portierungsaufwand für V1.
- **Stockfish-Code übernehmen:** könnte schneller spielstark werden, wäre nicht eigenständig und hätte GPLv3-Lizenzfolgen für eine abgeleitete Distribution.

## Folgen / zu entscheiden

Genaues Feature-Encoding, Netzdimensionen, Labeltransformation, Quantisierung, CP/WDL-Kalibrierung und Testschwellen **noch offen**. Vor Umsetzung Mini-Benchmark und ADR-Akzeptanzentscheidung; Nachweise TC-ML und TC-EVAL. Ein besserer Offline-Loss ist **kein** Nachweis größerer Elo-Spielstärke.
