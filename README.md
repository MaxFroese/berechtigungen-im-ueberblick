# Microsoft 365, Copilot und KI-Agenten im Praxistest

Stand: November 2026.

Diese Sammlung hält fest, was bei einer Erkundung von Microsoft 365, Azure, Copilot und Sprachmodellen herauskam: was tatsächlich funktioniert, was es kostet, welche Infrastruktur dafür nötig ist und wo die Grenzen liegen. Der Blick ist der einer kleinen Beratungsfirma (Systemhaus), die mit einem externen IT-Dienstleister arbeitet und prüft, welche KI-Anwendungsfälle sich lohnen.

Alle Tests liefen in einem eigenen Testtenant mit erfundenen Daten. Firmen, Personen und Projekte in den Beispielen sind ausgedacht. Zahlen und Preise beziehen sich auf den Stand der Tests und können sich ändern.

## Lesereihenfolge

| Reihenfolge | Dokument | Worum es geht |
|---|---|---|
| 1 | [Grundlagen](themen/grundlagen.md) | Anmeldung, Graph API, Teams und Rechte am Beispiel einer kleinen Frage-App |
| 2 | [Index und Retrieval](themen/index-und-retrieval.md) | Wie Suche und RAG in Microsoft 365 arbeiten und was die Messungen zeigen |
| 3 | [Harnesses im Vergleich](themen/harnesses-im-vergleich.md) | Copilot Chat, Copilot Studio, Copilot Harness, Cowork: Aufgaben, Grenzen, Abrechnung |
| 4 | [Die Apps](apps/README.md) | Sieben Prototypen: Ziel, Lösung, was live lief und warum nicht |

## Themen

| Thema | Dokument | Kurz |
|---|---|---|
| Grundlagen | [grundlagen.md](themen/grundlagen.md) | Der Weg von der App-Registrierung bis zum Token und zur Antwort |
| Index und Retrieval | [index-und-retrieval.md](themen/index-und-retrieval.md) | Stichwort- gegen Vektorsuche, Messungen mit der Retrieval API |
| Eigener Index im Vergleich | [eigener-index-foundry-iq.md](themen/eigener-index-foundry-iq.md) | Foundry IQ: ganze Sätze gehen, Listen und Synonyme bleiben schwierig |
| Copilot-Technik vertieft | [microsoft-rag-architektur.md](themen/microsoft-rag-architektur.md) | Begriffe, Abfragepfad, Sucharten und Lizenzen |
| Copilot Studio | [copilot-studio.md](themen/copilot-studio.md) | Agent mit SharePoint-Wissen, Evaluation, Modellwahl und Kosten |
| Harnesses | [harnesses-im-vergleich.md](themen/harnesses-im-vergleich.md) | Welche Umgebung wofür taugt und wie sie abgerechnet wird |
| Copilot Harness im Test | [copilot-harness-praxistest.md](themen/copilot-harness-praxistest.md) | Testfragen, Excel, Workflows, Skills, Kosten |
| Eigenbau auf Foundry | [eigenbau-agent.md](themen/eigenbau-agent.md) | RAG und agentische Schleife mit eigenem Code, Kosten je Anfrage |
| Anwendungsfälle | [einordnung-anwendungsfaelle.md](themen/einordnung-anwendungsfaelle.md) | Allgemeines Raster: welche KI-Wünsche wie angegangen werden |
| Projektagent | [umsetzungsplan-projektagent.md](themen/umsetzungsplan-projektagent.md) | Architekturentwurf für einen Agenten, der Projektstände mitführt |
| Testfragen | [testfragen.md](themen/testfragen.md) | Die Fragen, mit denen alle Varianten verglichen wurden |
| Testergebnisse | [testergebnisse.md](themen/testergebnisse.md) | Alle Ergebnisse im Überblick |

## Buch: Berechtigungen im Überblick

Ein eigenständiger Überblickstext in 21 Kapiteln mit Diagrammen und Glossar: Entra ID, Azure, Microsoft 365, GitHub, KI-Agenten, Lizenzen und Abrechnung, Recht und Datenweg, Arbeit mit externem IT-Dienstleister, Power Platform, Dataverse, ALM, Copilot Studio sowie SAP und die Schnittstelle zu Microsoft.

- Lesen als HTML: [buch/index.html](buch/index.html). Nach dem Klonen lokal im Browser öffnen oder über GitHub Pages.
- Quelltext der Kapitel als Markdown: [buch/kapitel/](buch/kapitel/)

## Hinweis

Keine Rechtsberatung. Wo es um Datenschutz, Verträge oder Lizenzen geht, ist das eine technische Einordnung und ersetzt keine Prüfung im Einzelfall.
