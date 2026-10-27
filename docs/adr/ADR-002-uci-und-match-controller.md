# ADR-002 – UCI-Adapter und separater Match-Controller

- **Datum:** 2026-10-27
- **Status:** Accepted (Projektanforderung, noch nicht implementiert)
- **Referenzen:** [Protokolle](../PROTOKOLLE.md), [Roadmap](../../ROADMAP.md)

## Kontext

SASD-AI-Chess soll gegen andere Schachprogramme sowie gegen eine **zweite unabhängig gestartete eigene Instanz** spielen. Engines beantworten Stellung-/Zuganfragen; die Spielleitung und Spieluhr sind eine andere Verantwortung.

## Entscheidung

- Standard-**UCI** über stdin/stdout als primäre öffentliche Engine-Schnittstelle.
- Eigenständiges **Match-Controller-Programm** startet und überwacht zwei UCI-Unterprozesse (auch dieselbe Binary zweimal), führt Partie/Legalkontrolle/Uhr/Resultat, exportiert PGN und strukturierte Ergebnisse.
- Der Controller besitzt einen autoritativen Spielzustand und nutzt dieselbe Chess-Core-Semantik; eine Engine-Ausgabe wird niemals unvalidiert übernommen.
- Engine-Suche, UCI-Textparser, Betriebssystemprozesssteuerung und Match-Zeitlogik bleiben voneinander isoliert.

## Alternativen

- **Proprietäre TCP-Zugschnittstelle:** verhindert zunächst breite Kompatibilität und erzeugt Netzwerksicherheits-/Versionierungsaufwand.
- **Zwei UCI-Prozesse direkt verbinden:** führt keine Uhr, Farben, vollständige Stellung oder Schiedsrichterlogik; UCI ist kein Peer-to-Peer-Matchprotokoll.
- **Match und Engine in einem Prozess:** erschwert Isolation, Crash-Handling und fairen Versionsvergleich.

## Konsequenzen / Tests

Separate Pipes und Prozesslebenszyklen; `uci/isready`-Handshake, `go/stop/quit`; Timeout, illegale Züge und Partiebeendigung testbar. Prüfungen: TC-UCI und TC-MATCH. Netzwerkworker sind spätere Erweiterungen, kein Bestandteil des UCI-Vertrags.
