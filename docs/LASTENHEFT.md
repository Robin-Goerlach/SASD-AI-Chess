# SASD-AI-Chess – Lastenheft

**Version:** 0.1 · **Stand:** 27.10.2026 · **Status:** Anforderungsbaseline / fachlicher Entwurf  
**Auftraggeberziel:** eigenständige, erweiterbare und wissenschaftlich nachvollziehbare Schach-KI.  
**Geltungsbereich:** SASD-AI-Chess, **nicht** das separate Projekt SASD-Chess.  
**Gegenstück:** [Pflichtenheft](PFLICHTENHEFT.md) und [Architektur](ARCHITEKTUR.md).

## 1. Ausgangssituation und Bedarf

Gesucht wird eine Schach-KI, die auf üblichen PCs läuft, Schach regelkonform spielt, gegen externe Engines und gegen **eine zweite eigene Instanz** antreten kann und mit heruntergeladenen Schachpartien trainierbar wird. Ihre Verbesserungen müssen anhand dokumentierter Experimente und Partien überprüfbar sein. Der Nutzer legt Wert auf **saubere Architektur, ausführlich kommentierten, verständlichen Code und später getrennte Optimierungsphasen**.

## 2. Projektziele

**Z-01 – Eigenständigkeit:** Die fertige Engine kann ohne Stockfish, Netzwerk, Python oder GPU spielen.  
**Z-02 – Interoperabilität:** UCI ermöglicht den Zugtausch mit Fremdengines und dem separaten SASD-Chess, sofern dieses UCI unterstützt.  
**Z-03 – Spiel-/Trainingsdaten:** PGN-Partien werden geladen, wiedergegeben, validiert und in Lernbeispiele umgewandelt.  
**Z-04 – Trainierbarkeit:** verschiedene Bewertungsmodelle lassen sich offline trainieren, vergleichen und versioniert bereitstellen.  
**Z-05 – Wissenschaftlicher Wert:** Hypothesen, Datensätze, Konfigurationen, Experimente und Resultate bleiben nachvollziehbar.  
**Z-06 – Portabilität:** Windows, Linux und BSD sind Zielplattformen (jeweils durch echte Tests nachzuweisen).

## 3. Zielgruppen und Nutzungsszenarien

- **Spieler / Schachoberfläche:** Partien per UCI gegen die Engine spielen.
- **Entwickler / Tester:** Zwei Engines lokal gegeneinander antreten lassen, Regeln und Suchleistung prüfen.
- **Forscher / Trainer:** PGN-Daten importieren, gefilterte Datasets erzeugen, Modelle vergleichen.
- **Automatisierter Betrieb:** Turnierläufe, Benchmarks, Datenpipelines und Regressionen ohne GUI.

**Beispiel A:** Eine externe UCI-Oberfläche startet SASD-AI-Chess und fordert `bestmove` an.  
**Beispiel B:** Ein Match-Controller startet zweimal dieselbe Engine und schließt eine Partie inklusive PGN ab.  
**Beispiel C:** Ein heruntergeladenes PGN-Archiv wird in legal rekonstruierte Stellungen und Labels zerlegt.  
**Beispiel D:** Ein neues NNUE-Modell wird erst nach bestandenen Prüfungen statt einer älteren Version freigegeben.

## 4. Funktionale Anforderungen („Was?“)

| ID | Priorität | Anforderung / beobachtbares Ergebnis |
| --- | --- | --- |
| **LR-F01** | MUSS | Standardschach mit allen legalen Zügen, Schach, Matt, Patt und regelgerechten Remisbedingungen |
| **LR-F02** | MUSS | Stellungsimport/-export in FEN, eindeutige Zugnotation (UCI), SAN für Partien |
| **LR-F03** | MUSS | Standalone-Engine, die auf Anfrage legale Züge berechnet |
| **LR-F04** | MUSS | Standardisierte UCI-Kommunikation mit anderen Programmen |
| **LR-F05** | MUSS | Zwei getrennte Engine-Prozesse können gegeneinander spielen, auch dieselbe Binärdatei zweimal |
| **LR-F06** | MUSS | Match-Controller führt unabhängigen Spielzustand, Regeln und Bedenkzeiten |
| **LR-F07** | MUSS | Partien und Ergebnisse als PGN exportieren |
| **LR-F08** | MUSS | Eine und viele PGN-Partien laden, auf Legalität prüfen und vollständig wiedergeben |
| **LR-F09** | MUSS | Große, auch komprimierte Partiensammlungen ressourcenbegrenzt importieren |
| **LR-F10** | MUSS | Aus Partien versionierte Trainingsbeispiele erzeugen und Datenqualität dokumentieren |
| **LR-F11** | MUSS | Klassische, nachvollziehbare Stellungsbewertung und Suche als Baseline |
| **LR-F12** | MUSS | Offline ein eigenes neuronales Bewertungsmodell trainieren und in C++ ausführen |
| **LR-F13** | MUSS | Engine-/Modellversionen gegeneinander vergleichen und Resultate speichern |
| **LR-F14** | MUSS | Modelle und Experimente mit Herkunft, Konfiguration und Hashes identifizieren |
| **LR-F15** | SOLL | Automatisierte Selbstspiel-Trainingsschleife mit kontrollierter Modellfreigabe |
| **LR-F16** | SOLL | Nach Elo, Zeitkontrolle, Quelle, Variante und Qualität filtern sowie Duplikate erkennen |
| **LR-F17** | SOLL | Bereits analysierte Stellungsbewertungen aus externen Formaten übernehmen |
| **LR-F18** | SOLL | Analysedaten und Ergebnisse für wissenschaftliche Wiederholung exportieren |
| **LR-F19** | KANN | Netzwerkbasierte Match-/Trainingsworker nach einer separaten Sicherheitsprüfung |
| **LR-F20** | KANN | Alternative neuronale Sucharchitekturen, GUI und zusätzliche Varianten |

**Prioritäten:** MUSS = Zielumfang bis stabiler V1-Baseline (nicht zwingend im ersten MVP); SOLL = vorgesehen nach passendem Meilenstein; KANN = Option, keine Pflicht.

## 5. Nichtfunktionale Anforderungen

| ID | Priorität | Qualitätsanspruch |
| --- | --- | --- |
| **LR-Q01** | MUSS | Schachregeln nachweisbar korrekt; reversible Züge und bekannte Perft-Referenzen |
| **LR-Q02** | MUSS | Keine Python-/GPU-/Internetpflicht für einen Engine-Lauf |
| **LR-Q03** | MUSS | Tests, nachvollziehbare Fehlermeldungen und überprüfbare Implementierungsstände |
| **LR-Q04** | MUSS | Transparente Daten-/Modellprovenienz und rechtlich geprüfte Lizenzbedingungen |
| **LR-Q05** | MUSS | Untrusted Input und fremde Prozesse mit Ressourcenlimits, Isolation und Timeouts behandeln |
| **LR-Q06** | MUSS | Wartbare Modularchitektur, englische Codekommentare/Doxygen, deutschsprachige Dokumentation |
| **LR-Q07** | SOLL | Auf Windows/Linux/BSD mit geringstmöglichen Plattform-Sonderfällen nutzbar |
| **LR-Q08** | SOLL | Reproduzierbare Trainings-/Matchberichte inklusive Seed, Splits, Versionen, Messunsicherheit |
| **LR-Q09** | SOLL | Große Datenbestände streamend statt vollständig im RAM verarbeiten |
| **LR-Q10** | SOLL | Aussagekräftige Laufzeit- und Spielstärkekennzahlen ermitteln, ohne unbegründete Zielwerte zu garantieren |

## 6. Nichtziele und Randbedingungen

In V1 **nicht erforderlich**: grafische Oberfläche, Webserver, Cloudzwang, Chess960, mobile Apps, garantierte Großmeister-/Stockfish-Spielstärke, verteilte GPU-Trainingscluster. Das Projekt steht unter [MIT-Lizenz](../LICENSE); **Drittanbieter-Engines, Modelle und PGNs unterliegen ihren jeweils eigenen Bedingungen**. Trainingsdaten sollen grundsätzlich nicht ins Git-Repository.

## 7. Abnahme auf Auftraggeberebene

**A-01:** Eine echte Testoberfläche erhält UCI-Antworten und einen legalen Zug.  
**A-02:** Zwei getrennte eigene Prozesse spielen eine vollständige Partie mit Ergebnis/PGN.  
**A-03:** Eine PGN-Testsammlung kann verlustfrei interpretiert, rekonstruiert und protokolliert werden; fehlerhafte Einträge werden gemeldet.  
**A-04:** Ein eigenes NNUE-artiges Modell wird trainiert, geladen und ohne Python durch die C++-Engine ausgewertet.  
**A-05:** Vergleichspartien/Benchmarks mit dokumentiertem Datenstand erlauben eine belastbare Aussage über den Lernfortschritt.  
**A-06:** Builds, Tests und Einschränkungen werden pro tatsächlich geprüfter Plattform dokumentiert.

Die detaillierte technische Prüfung und Anforderungsrückverfolgbarkeit regeln [Pflichtenheft](PFLICHTENHEFT.md) und [Testkonzept](TESTKONZEPT.md). **Noch kein Abnahmekriterium gilt durch die bloße Erstellung dieses Dokuments als erfüllt.**
