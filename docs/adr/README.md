# Architecture Decision Records (ADRs)

Dieses Verzeichnis enthält nachvollziehbare, **einzeln versionierte Architekturentscheidungen**. Ein ADR erläutert **Kontext, Entscheidung, Alternativen, Konsequenzen und Status**; es ist keine Implementierung oder Testbestätigung.

| ADR | Thema | Status |
| --- | --- | --- |
| [ADR-001](ADR-001-modulare-laufzeit-und-training.md) | C++-Engine, Python-Trainer, klare Modulgrenzen | Accepted – Architekturbaseline |
| [ADR-002](ADR-002-uci-und-match-controller.md) | UCI-Adapter + eigener Match-Controller | Accepted – Anforderung |
| [ADR-003](ADR-003-pgn-und-dataset-versionierung.md) | PGN-Import, Herkunft, Shards und Splits | Accepted – Anforderung |
| [ADR-004](ADR-004-classical-und-nnue.md) | Klassische Baseline und später NNUE | Proposed – Modell-Feindesign offen |

Die neun in [ARCHITEKTUR.md](../ARCHITEKTUR.md) Abschnitt 14 skizzierten **ADR-001 bis ADR-009** sind eine anfängliche Entscheidungsliste; hier werden die wesentlichen Punkte in eigenständige Records überführt. Die dort noch nicht separat ausgearbeiteten Entscheidungen (z. B. Offline-first, Model-Promotion und Entkopplung der SASD-Projekte) werden bei einer fachlichen Änderung oder vor Implementierung ebenfalls formalisiert. Keine zweite, widersprechende Nummernserie erzeugen.

## Regeln für neue ADRs

1. Nächste eindeutige Nummer vergeben; Titel und Datum hinzufügen.
2. Status: Proposed → Accepted / Rejected / Superseded (mit Nachfolger).
3. Problem und Treiber, mindestens eine echte Alternative, Entscheidung, Konsequenzen und Testbarkeit beschreiben.
4. Betroffene Schnittstellen/Formatversionen, Migration und Build-/Testauswirkungen angeben.
5. Unter keinen Umständen eine akzeptierte ADR wortlos überschreiben; neue Entscheidung mit `Supersedes` verlinken.

Die Projekt-[MIT-Lizenz](../../LICENSE) erlaubt nicht automatisch das Kopieren fremder Bibliotheken oder Modelle. Abhängigkeiten/Lizenzkonflikte explizit bewerten.
