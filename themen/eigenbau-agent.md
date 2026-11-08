# Eigenbau-Agent auf Azure Foundry

Stand: 6. November 2026, Messung 18:34 bis 19:00 UTC in einem Testtenant. Zurück zur [Startseite](../README.md). Alle Daten sind erfunden.

**Kurz gesagt:** Der selbst gebaute Agent mit Werkzeugen beantwortet 30 von 31 Testfragen richtig. Er liest auch den Scan, den Text im Bild und alle sieben Listenfragen. Falsch ist nur die Teams-Frage, denn Teams ist keine Wissensquelle. Das kostet mit GPT 6.1 Sol rund 3 US-Cent je Frage und knapp 30 Sekunden. Das einfache RAG über die Retrieval API kommt auf 23 richtige für unter 1 Cent je Frage. Es scheitert an denselben Stellen wie der Microsoft-Index: Scans, Bilder, Synonyme und Auszüge, in denen der gesuchte Absatz fehlt. Der Vorsprung des Agenten kommt nicht von einer besseren Suche, sondern daher, dass er ganze Dateien liest, Texterkennung nutzt und mit Python rechnet.

## 1. Aufbau des Tests

| | |
|---|---|
| **Modell** | GPT 6.1 Sol als eigenes Deployment in Azure Foundry, Global Standard, Responses API, `reasoning.effort: medium` |
| **Wissen** | Dokumentbibliothek einer SharePoint-Site mit erfundenem Projektarchiv, 241 Dateien, nur lesend. Keine Teams-Nachrichten |
| **Anmeldung** | Modell und Document Intelligence mit dem Token der Azure CLI. Microsoft Graph mit dem delegierten Token des angemeldeten Nutzers über eine bereits vorhandene Entra-App-Registrierung |
| **Testsätze** | Dieselben 31 [Testfragen](testfragen.md) wie in [Copilot Studio im Test](copilot-studio.md): 12 Vergleichsfragen (V), 12 Formatprobe (F), 7 Listenfragen (L) |
| **Anweisung** | Die Anweisung des Studio-Agenten, ergänzt um Hinweise zu den Werkzeugen |
| **Bewertung** | Vorschlag per Regex, danach jede Antwort von Hand gelesen. 6 Vorschläge wurden korrigiert. **R** richtig, **T** teilweise, **F** falsch, **N** nichts gefunden |

### Die zwei Varianten

| | Variante 1: einfaches RAG | Variante 2: agentische Schleife |
|---|---|---|
| **Ablauf** | Modell bildet 2 bis 3 Stichwortsuchen, Retrieval API liefert Textstellen, Modell antwortet | Modell ruft Werkzeuge auf, bis es antwortet. Höchstens 15 Schritte, höchstens 0,80 USD je Frage |
| **Suche** | Copilot Retrieval API, auf die Site begrenzt, 25 Treffer je Anfrage, höchstens 40.000 Zeichen Kontext | Graph-Suche (`/search/query`), Stichwörter |
| **Werkzeuge** | keine | Suchen, Ordner auflisten, Datei lesen, ganzen Ordner laden, Texterkennung, Python |
| **Dateien lesen** | nur die Auszüge der Retrieval API | ganze Dateien über Graph, mit Textextraktion für Word, Excel mit allen Blättern, PowerPoint samt Sprechernotizen, PDF, Markdown, Text, BPMN |
| **Scans und Bilder** | nein | Document Intelligence `prebuilt-read`, Stufe F0, auch für Bilder in Folien und Word |
| **Rechnen** | nein | Python in einem Arbeitsordner, ohne Netz (macOS-Sandbox), 60 Sekunden |
| **Modellaufrufe je Frage** | genau 2 | 3 bis 9, im Mittel 4,7 |

**Warum RAG mit Umformulierung:** Ganze Sätze finden über die Retrieval API in diesem Tenant nichts, siehe [Index und Retrieval](index-und-retrieval.md). Ohne den ersten Modellaufruf, der Stichwörter bildet, hätte Variante 1 fast überall N.

## 2. Summen je Variante

| Variante | R | T | F | N | Kosten gesamt | Kosten je Frage | Dauer im Mittel | Median |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| **1 Einfaches RAG** | 23 | 2 | 1 | 5 | 0,29 USD | 0,9 US-Cent | 10,3 s | 9,4 s |
| **2 Agentische Schleife** | **30** | 0 | 1 | 0 | 1,03 USD | 3,3 US-Cent | 29,2 s | 19,6 s |
| Zum Vergleich: Studio-Agent, GPT-5 Chat | 22 | 3 | 3 | 3 | nicht sichtbar | nicht sichtbar | rund 20 s | |

Der Studio-Vergleich stammt aus den früheren Läufen mit GPT-5 Chat auf dem Standard-Harness. Die Läufe mit GPT 6.1 Sol auf dem neuen Microsoft Copilot Harness stehen in [Copilot Harness im Praxistest](copilot-harness-praxistest.md): 10 von 10 Vergleichsfragen richtig, 7 bis 34 Credits je Frage, also rund 5-mal so teuer wie dieser Agent.

### Nach Testsatz

| Testsatz | RAG R/T/F/N | Agent R/T/F/N | RAG USD je Frage | Agent USD je Frage | RAG Dauer | Agent Dauer |
|---|---|---|---:|---:|---:|---:|
| Vergleichsfragen, 12 | 7 / 2 / 1 / 2 | **11** / 0 / 1 / 0 | 0,0122 | 0,0289 | 10,4 s | 29,4 s |
| Formatprobe, 12 | 10 / 0 / 0 / 2 | **12** / 0 / 0 / 0 | 0,0030 | 0,0089 | 8,4 s | 9,3 s |
| Listenfragen, 7 | 6 / 0 / 0 / 1 | **7** / 0 / 0 / 0 | 0,0156 | 0,0830 | 13,3 s | 63,0 s |

### Token

| | Eingabe | davon aus Cache | Ausgabe | Modellaufrufe |
|---|---:|---:|---:|---:|
| RAG, 31 Fragen | 89.374 | 5.656 | 8.611 | 62 |
| Agent, 31 Fragen | 1.115.725 | 795.395 | 17.461 | 145 |

Der Agent schickt bei jedem Schritt den ganzen Verlauf erneut. 71 Prozent seiner Eingabe kamen deshalb aus dem Prompt-Cache, und der kostet ein Zwanzigstel. Ohne Cache läge der Agent bei rund 2,41 statt 1,03 USD.

## 3. Ergebnisse je Frage

| # | Was geprüft wird | RAG | Agent | RAG USD | Agent USD | RAG s | Agent s | Schritte |
|---|---|---|---|---:|---:|---:|---:|---:|
| V01 | Budget Velmora, Widerspruch | R | R | 0,0086 | 0,0148 | 16,1 | 20,6 | 4 |
| V02 | Velmora live | R | R | 0,0077 | 0,0137 | 12,0 | 19,6 | 4 |
| V03 | Synonym „nicht verfügbar“ | T, nur alter Wert 120 Min. | **R** | 0,0046 | 0,0274 | 11,8 | 23,3 | 5 |
| V04 | Ende einer langen Datei | T, nur alter Wert 6 Jahre | **R** | 0,0175 | 0,0290 | 10,7 | 21,3 | 5 |
| V05 | Excel-Tabelle | R | R | 0,0164 | 0,0081 | 8,4 | 10,7 | 3 |
| V06 | Tabelle im PDF | R | R | 0,0161 | 0,0256 | 8,8 | 22,1 | 4 |
| V07 | Text nur im Bild | N | **R**, per Texterkennung | 0,0167 | 0,0255 | 9,0 | 16,6 | 4 |
| V08 | Sprechernotiz | R | R | 0,0167 | 0,0263 | 8,3 | 28,1 | 6 |
| V09 | Scan ohne Textebene | N | **R**, per Texterkennung | 0,0171 | 0,0188 | 11,5 | 18,4 | 4 |
| V10 | Lumeno in mehreren Projekten | R | R | 0,0053 | 0,0617 | 11,2 | 74,1 | 5 |
| V11 | Teams-Nachricht (Z2) | F, Hessen | F, Hessen | 0,0173 | 0,0418 | 10,5 | 36,5 | 9 |
| V12 | Brunnhalde, nicht in den Quellen (Z3) | R | R | 0,0022 | 0,0536 | 6,8 | 61,3 | 6 |
| F01 | Nord, Word Fließtext | R | R | 0,0029 | 0,0071 | 8,4 | 7,0 | 3 |
| F02 | Süd, Word Tabelle | R | R | 0,0023 | 0,0061 | 7,3 | 7,1 | 3 |
| F03 | Ost, PDF Fließtext | R | R | 0,0026 | 0,0072 | 6,6 | 7,0 | 3 |
| F04 | West, PDF Tabelle | R | R | 0,0021 | 0,0062 | 8,5 | 7,0 | 3 |
| F05 | Hafen, PDF als Scan | N | **R** | 0,0024 | 0,0113 | 7,6 | 14,1 | 5 |
| F06 | Mühle, Excel Blatt 1 | R | R | 0,0026 | 0,0062 | 9,9 | 7,3 | 3 |
| F07 | Damm, Excel Blatt 3 | R | R | 0,0085 | 0,0144 | 9,0 | 6,2 | 3 |
| F08 | Brücke, Folientext | R | R | 0,0025 | 0,0070 | 9,2 | 7,6 | 3 |
| F09 | Wehr, Sprechernotiz | R | R | 0,0024 | 0,0071 | 8,8 | 6,7 | 3 |
| F10 | Turm, Text im Bild | N | **R** | 0,0020 | 0,0218 | 7,6 | 20,1 | 6 |
| F11 | Kanal, Markdown | R | R | 0,0024 | 0,0061 | 9,4 | 12,3 | 3 |
| F12 | Schleuse, Text | R | R | 0,0032 | 0,0061 | 9,0 | 9,1 | 3 |
| L01 | Mesaro Data, 8 Projekte | R | R | 0,0137 | 0,0545 | 11,8 | 50,2 | 5 |
| L02 | Pia Lorenzen, 5 Projekte | R | R | 0,0180 | 0,0680 | 14,1 | 53,3 | 7 |
| L03 | Wie viele mit Trevanto, 7 | R | R | 0,0113 | 0,0679 | 10,4 | 61,7 | 5 |
| L04 | Höchstes aktuelles Budget | R | R | 0,0215 | 0,1202 | 11,8 | 85,6 | 8 |
| L05 | Ist-Kosten über Plan, 5 | **N** | R | 0,0033 | 0,0907 | 10,5 | 87,9 | 8 |
| L06 | Go-live 2023, 9 Projekte | R, mit Vorbehalt | R, mit Vorbehalt | 0,0261 | 0,1592 | 24,3 | 87,2 | 9 |
| L07 | Palvera: Leitung und Partner | R | R | 0,0148 | 0,0201 | 10,5 | 15,0 | 3 |

**Was auffällt:**

- **RAG scheitert an Auszügen, nicht an der Suche.** Bei V03 und V04 fand die Retrieval API die richtige Datei, ein Fachkonzept in Fassung v3. Ihr Auszug enthielt aber nicht den Absatz mit 38 Minuten oder 11 Jahren. Das Modell sagte richtig, dass v3 gilt und der Wert fehlt. Der Agent las die ganze Datei und fand beide Werte.
- **Scans und Bilder sind für RAG unsichtbar,** wie für jeden Weg über den Microsoft-Index. Der Agent erkennt sie an einem Hinweis der Textextraktion („Folie 3 enthält 1 Bild“, „Kein Text gefunden, vermutlich Scan“) und ruft dann selbst die Texterkennung auf. Das gelang in allen vier Fällen.
- **Listen nach Bedingung:** Bei L05 findet RAG mit den Stichwörtern „Ist-Kosten“ nichts. Der Agent lädt alle 241 Dateien als Text, sucht mit Python die Abschlussberichte und vergleicht Plan und Ist. Ergebnis: alle 5, mit Beträgen.
- **L06 bei beiden mit Vorbehalt:** Beide nennen die richtigen 9 Projekte, schreiben aber, belegt seien nur die geplanten Termine. Gewertet ist das als R, weil die Liste stimmt.
- **V11 bleibt falsch.** Der richtige Wert „Thüringen“ steht nur in einer Teams-Nachricht. Die Graph-Suche nach Chatnachrichten braucht das Recht `Chat.Read`, das die vorhandene App-Registrierung nicht hat (HTTP 403). Ein Werkzeug für Teams-Kanäle wäre möglich, ist aber nicht gebaut. Beide Varianten weisen darauf hin, dass Teams nicht geprüft ist.
- **Keine erfundenen Werte.** Wo RAG nichts fand, sagte es das. Der Agent stieß nie an die Schrittgrenze, höchstens 9 von 15 Schritten.

## 4. Wiederholbarkeit

Je zwei Listenfragen dreimal zusätzlich zum Hauptlauf, also viermal insgesamt. L01 ist eine Liste nach Namen, L05 eine nach Bedingung.

| Variante | Frage | Hauptlauf | Wdh. 1 | Wdh. 2 | Wdh. 3 | Kosten je Lauf | Dauer je Lauf |
|---|---|---|---|---|---|---:|---:|
| RAG | L01 Mesaro Data | R 8/8 | R 8/8 | R 8/8 | R 8/8 | 0,006 bis 0,014 USD | 12 bis 16 s |
| RAG | L05 Ist über Plan | N | N | N | **R 5/5** | 0,003 bis 0,020 USD | 10 bis 15 s |
| Agent | L01 Mesaro Data | R 8/8 | R 8/8 | R 8/8 | R 8/8 | 0,055 bis 0,074 USD | 50 bis 64 s |
| Agent | L05 Ist über Plan | R 5/5 | R 5/5 | R 5/5 | R 5/5 | 0,087 bis 0,121 USD | 86 bis 130 s |

**Einordnung:**

- **Der Agent ist im Ergebnis stabil, im Weg nicht.** Die Antworten stimmten in allen acht Läufen. Token und Dauer schwankten aber um bis zu 50 Prozent, weil er je Lauf andere Werkzeuge in anderer Reihenfolge nimmt.
- **RAG hängt an einem einzigen Stichwort.** Bei L05 bildete das Modell dreimal Anfragen wie „Ist-Kosten Plan-Kosten“ und fand 0 Treffer. Im vierten Lauf kam „Kosten Plan Ist“ dazu, das traf 13 Dateien, und die Antwort war vollständig richtig. Gleiches Modell, gleiche Frage, Ergebnis von N bis R.

Zum Vergleich: Der Microsoft Copilot Harness beantwortete die Listenfrage nach dem höchsten Budget nur in zwei von vier Läufen richtig, weil ihm ein Werkzeug zum Auflisten von Ordnern fehlt (siehe [Praxistest](copilot-harness-praxistest.md#5-verlässlichkeit-und-skills)).

## 5. Kosten

**Preise** laut [Azure Retail Prices API](https://prices.azure.com/api/retail/prices), abgerufen am 6.11.2026, Filter `serviceName eq 'Foundry Models'`, Region `swedencentral`, Meter `6.1-sol … Std Gl`, gültig seit 1.10.2026.

| Meter, Global Standard, kurzer Kontext | USD je 1 Mio. Token |
|---|---:|
| Eingabe (`ShortCo Inp Std Gl`) | 2,00 |
| Eingabe aus Cache (`ShortCo Cd Inp Std Gl`) | 0,10 |
| Schreiben in den Cache (`ShortCo Cd Wr Std Gl`) | 2,50 |
| Ausgabe, inklusive Reasoning (`ShortCo Opt Std Gl`) | 10,00 |
| Langer Kontext: Eingabe / Ausgabe (`LongCo`) | 4,00 / 15,00 |

Gerechnet ist mit dem kurzen Kontext. Wo die Grenze zum langen Kontext liegt, nennt die API nicht. Der größte einzelne Aufruf hatte rund 47.000 Token Eingabe. Die API meldet getrennt `cached_tokens` und `cache_write_tokens`, beide sind mit ihrem eigenen Preis gerechnet. Beim Agenten wurden im Hauptlauf 278.105 Token in den Cache geschrieben, das kostet je Token ein Viertel mehr als normale Eingabe.

| Posten | USD |
|---|---:|
| Probeläufe beim Bauen | 0,19 |
| RAG, Hauptlauf 31 Fragen | 0,29 |
| Agent, Hauptlauf 31 Fragen | 1,03 |
| RAG, 6 Wiederholungen | 0,04 |
| Agent, 6 Wiederholungen | 0,52 |
| **Gesamt, nach Listenpreis** | **2,08** von höchstens 10 |
| Document Intelligence, F0 | 0, 36 Anfragen in den Messläufen, dazu wenige Probeläufe, alles in der Freimenge |
| Retrieval API und Graph-Suche | 0, Copilot-Lizenz vorhanden |

Im Azure Cost Management war davon am Messtag noch nichts zu sehen, es bucht mit Verzögerung. Damit die Kosten trotzdem nicht aus dem Ruder laufen, führte der Agent einen eigenen Kostenzähler aus den gemeldeten Token, mit harter Grenze von 9 USD für alle Läufe zusammen.

**Hochgerechnet auf 1.000 Fragen mit dieser Mischung:** RAG rund 9 USD, Agent rund 33 USD Modellkosten. Bei vielen Listenfragen ist der Agent eher bei 80 USD. Document Intelligence wäre ab 500 Seiten im Monat kostenpflichtig, das ist nicht gerechnet.

## 6. Aufwand und nötige Infrastruktur

| | |
|---|---|
| **Bauzeit** | Rund 15 Minuten bis zum ersten Messlauf, gebaut von einem KI-Agenten, der vorhandene Bausteine aus früheren Test-Apps wiederverwenden konnte. Für einen Menschen ohne diese Vorlagen eher ein bis zwei Tage |
| **Maschinenzeit** | RAG 31 Fragen 5,5 Minuten, Agent 31 Fragen 15 Minuten, beide parallel |
| **Code** | Rund 800 Zeilen Python, davon rund 580 für die beiden Varianten selbst, der Rest für Messung und Auswertung. Dazu eine kleine Brücke zu einer vorhandenen Textextraktion in TypeScript |
| **Wiederverwendet** | Anmeldung und Graph-Aufrufe, Textextraktion für alle Formate (aus der Projektkartei, siehe [Apps](../apps/README.md)), Document Intelligence aus einer anderen Test-App |
| **Neue Azure-Ressourcen, App-Registrierungen, Rechte** | keine |

Was an Infrastruktur nötig war, war also schon vorhanden: ein Foundry-Deployment des Modells, eine Document-Intelligence-Ressource, eine Entra-App-Registrierung mit delegierten Graph-Rechten zum Lesen von SharePoint und eine Copilot-Lizenz für die Retrieval API. Ohne diese Vorarbeit kämen Anlegen der Ressourcen, App-Registrierung, Consent für die Graph-Rechte und die Anmeldung hinzu. Für einen Betrieb mit mehreren Nutzern fehlen außerdem eine Oberfläche, ein Server oder eine Web-App, die Token-Erneuerung und eine sichere Ausführung des Python-Codes.

## 7. Eigenbau gegen Microsoft

| | Eigenbau | Copilot Studio, Standard-Harness |
|---|---|---|
| **Scans und Bilder** | **Gelesen**, per Texterkennung bei Bedarf | Mit GPT-5 Chat nie gefunden. GPT-5 Reasoning las den Scan, das Bild nicht |
| **Listen nach Bedingung** | **Gelöst**, alle Dateien laden und mit Python rechnen | Scheitert mit GPT-5 Chat, besser mit Reasoning |
| **Lange Dateien, Auszug ohne die Stelle** | **Gelöst**, liest die ganze Datei | Gelöst nach besserer Anweisung (Frage 4) |
| **Teams-Nachrichten** | Nicht angebunden, bräuchte `Chat.Read` oder ein Kanal-Werkzeug | Ebenfalls nicht |
| **Kosten sehen** | **Je Frage auf den Cent**, aus den Token | Credits erst Tage später, je Lauf nicht sichtbar |
| **Kosten je Frage** | 1 bis 16 US-Cent | unbekannt |
| **Nachvollziehbar** | **Jeder Werkzeugaufruf protokolliert** | Aktivitätsansicht in der Oberfläche, Antworttexte nur mit Studio-Rechten |
| **Evaluation** | Eigenes Skript, Bewertung von Hand | Eingebaut, aber zu großzügig |
| **Rechte** | Läuft mit dem Token des Nutzers über Graph, sieht also nur, was er sehen darf. Geprüft nur mit einem Konto | Rechte über die Anmeldung, von Microsoft verwaltet |
| **Bau und Betrieb** | Code, Python und Node, Token-Erneuerung, Sandbox für den Python-Code, jemand muss es pflegen | Kein Code, Oberfläche, Microsoft pflegt |
| **Oberfläche für Nutzer** | Keine, nur Kommandozeile | Teams und Copilot fertig |
| **Dauer** | Agent 7 bis 130 s, Listen über eine Minute | rund 20 s je Frage |
| **Daten auf dem Rechner** | Der Agent lädt bei Listen die ganze Bibliothek, rund 10 MB, in einen Arbeitsordner | Bleibt in Microsoft 365 |

Gegenüber dem Microsoft Copilot Harness, der ebenfalls ganze Dateien liest und rechnet, liegt der Eigenbau bei einem Fünftel bis einem Zehntel der Kosten und war bei den Listenfragen verlässlicher. Dafür bringt der Harness Oberfläche, Anmeldung, Sandbox und Betrieb fertig mit (siehe [Praxistest](copilot-harness-praxistest.md#7-kosten)).

**Wo der Eigenbau schlechter ist:**

- **Langsam bei Listen.** Er lädt für jede Listenfrage alle 241 Dateien neu. Mit einem Zwischenspeicher wären es Sekunden, der fehlt hier bewusst, damit die Dauer ehrlich bleibt. Bei großen Beständen geht das so nicht. Dann braucht es einen Index oder eine strukturierte Ablage wie die Projektkartei.
- **Python-Code des Modells läuft auf dem Rechner.** Hier abgesichert mit der macOS-Sandbox: kein Netz, Schreiben nur im Arbeitsordner. In einem echten Betrieb gehört das in einen eigenen Container.
- **Er verlässt sich auf Hinweise der Extraktion.** Ohne den Satz „Folie 3 enthält 1 Bild“ hätte er die Texterkennung nicht gestartet. Bilder in PDF-Dateien mit Textebene erkennt er nicht.
- **F0 bremst.** Document Intelligence F0 erlaubt rund 1 Anfrage je Sekunde und liest nur die ersten 2 Seiten. Für lange Scans reicht das nicht.

## 8. Was nicht geklappt hat oder offen ist

- **Teams-Nachrichten:** Die Graph-Suche nach Chatnachrichten antwortet mit 403, es fehlt `Chat.Read`. Neue Rechte wurden für den Test nicht vergeben. Frage V11 ist deshalb in beiden Varianten F.
- **Werkzeuge über Chat Completions:** Das Modell lehnt Werkzeuge zusammen mit Reasoning in `/v1/chat/completions` ab. Der Agent nutzt deshalb die Responses API, mit `store: false` und verschlüsseltem Reasoning im Verlauf.
- **Erster RAG-Versuch:** Die Planung bildete zuerst Stichwörter wie „Zählerportal Ausfallzeit“, die nie wörtlich vorkommen, und fand 0 Treffer. Die Anweisung sagt jetzt, dass die Wörter UND-verknüpft sind und eine Anfrage nur den Eigennamen enthalten soll.
- **Automatische Bewertung:** Der Regex lag sechsmal daneben, etwa bei „7 unterschiedliche Projekte“ und bei Antworten, die nur den alten Wert nennen, ihn aber als überholt kennzeichnen. Ohne Lesen von Hand geht es auch hier nicht.
- **Kosten nur nach Listenpreis.** Ein Abgleich mit Azure Cost Management wurde nicht gemacht.
- **Rechte mit zweitem Nutzer** nicht geprüft, wie in den anderen Tests.
