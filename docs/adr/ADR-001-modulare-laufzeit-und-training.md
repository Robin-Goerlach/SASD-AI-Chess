# ADR-001 – C++-Engine, Python-Training und entkoppelte Module

- **Datum:** 2026-10-27
- **Status:** Accepted (Architekturentscheidung, noch nicht implementiert)
- **Treiber:** regelkonforme, portable Schachengine; verständlicher Code; unabhängige ML-Forschung
- **Referenzen:** [ARCHITEKTUR](../ARCHITEKTUR.md), [Pflichtenheft](../PFLICHTENHEFT.md)

## Kontext

Der Engine-Hotpath benötigt deterministische Schachzustände, effiziente Suche und portable Ausführung. Modelltraining benötigt hingegen Datenpipelines, automatische Differenzierung und flexible ML-Werkzeuge. Beide Lebenszyklen unterscheiden sich erheblich.

## Entscheidung

- **C++20 und CMake** für Chess Core, Search, Evaluation, UCI, Match und Dataset-CLI.
- **Python/PyTorch** nur für Trainer, Experimentanalyse und Export; nicht als Runtime-Abhängigkeit der Engine.
- **Ports-and-Adapters und modulare Monolithen**: Core kennt weder I/O noch Trainer; Suchservice verwendet einen schmalen Position-Evaluator-Port.
- Engine, Match-Controller und Importer sind **separat ausführbar**, dürfen aber geprüfte fachliche Core-Bibliotheken wiederverwenden.

## Alternativen

- **Alles Python:** hoher Forschungsnutzen, aber vermeidbare Hotpath-/Deployment-Abhängigkeit.
- **Alles C++ inklusive Training:** einfacherer Sprachstack, aber deutlich höhere ML-Entwicklungskosten.
- **Microservices ab Tag eins:** zu hohe Infrastrukturkosten und Fehleroberfläche ohne anfänglichen Nutzen.

## Konsequenzen / Überprüfung

Cross-Language-Feature- und Inferenzparität verpflichtend. CMake-Build darf PyTorch nicht verlangen. Portabilität für Windows/Linux/BSD ist Ziel, erfordert reale Buildtests; keine garantierte ABI-Stabilität im frühen Entwicklungsstadium. Architekturgrenzen in PR-Reviews prüfen.

**Offen:** konkrete Versionen/Dependencies, Versionierungsformat des Modell-Exports und Bibliotheksauswahl; dafür separate ADRs vor Implementierung.
