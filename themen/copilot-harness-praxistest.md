# Copilot Harness im Praxistest

Stand: 6. November 2026, Tests von 18 bis 21 Uhr in einem Testtenant. Zurück zur [Startseite](../README.md). Alle Daten sind erfunden. Die Begriffe erklärt [Harnesses im Vergleich](harnesses-im-vergleich.md).

**Kurz gesagt:** Der neue Microsoft Copilot Harness (früher „GitHub Copilot harness“) ist im Test deutlich stärker als der Standard-Harness. Er öffnet ganze Dateien, liest Scans und Text in Bildern, rechnet in einer Sandbox und erstellt fertige Excel-Dateien. Von 20 Testfragen beantwortete er 17 richtig, darunter genau die, an denen der Standard-Agent scheiterte. Die Kosten liegen bei 7 bis 35 Credits je Einzelfrage und 40 bis 200 Credits für Listen und Dateien, also 0,07 bis 2 USD. Das ist viel weniger, als die Blogs schreiben, aber 3- bis 10-mal mehr als ein selbst gebauter Agent mit demselben Modell auf Foundry. Die größte Schwäche ist nicht das Können, sondern die Verlässlichkeit: Dieselbe Frage gelang in zwei von vier Läufen, auch ein Skill mit fester Vorgehensweise half nicht sicher.

## 1. Aufbau des Tests

| | |
|---|---|
| **Agent** | „Projektwissen Harness“, per Power Platform CLI auf dem neuen Harness angelegt |
| **Anweisung** | Dieselbe wie beim Standard-Agenten, Fassung v2 |
| **Wissen** | Dieselbe SharePoint-Site mit dem erfundenen Projektarchiv |
| **Modell** | GPT 6.1 Sol. Voreingestellt war Claude Sonnet 4.6, das braucht eine Admin-Freigabe für Anthropic |
| **Prüfweg** | Vorschau in Studio, jede Frage in einem neuen Chat, Credits vom Zähler in der Vorschau |
| **Vergleich** | Standard-Agent aus [Copilot Studio im Test](copilot-studio.md) und [Eigenbau-Agent auf Foundry](eigenbau-agent.md) |
| **Bewertung** | Von Hand gegen die Sollantwort: **R** richtig, **T** teilweise, **F** falsch, **N** nichts gefunden |

**Wichtig für den Vergleich:** Der Standard-Agent lief mit GPT-5 Chat, der neue Harness und der Eigenbau mit GPT 6.1 Sol. Ein Teil des Vorsprungs kommt also vom Modell. Wo der Standard-Harness mit GPT-5 Reasoning lief, steht das dabei.

Aus Zeitgründen liefen nicht alle 31 [Testfragen](testfragen.md). Die Formatprobe lief nur für die fünf aufschlussreichen Fälle, die Listen für die drei, an denen der Standard-Agent scheiterte.

## 2. Ergebnisse je Frage

| # | Was geprüft wird | Standard, GPT-5 Chat | Copilot Harness | Credits | Eigenbau-Agent |
|---|---|---|---|---:|---|
| V1 | Widerspruch beim Budget | R | R, nennt alle drei Fassungen | 7,3 | R |
| V2 | Plan gegen Ist | R | R | 17,0 | R |
| V3 | Synonym | R | R | 7,6 | R |
| V4 | Ende einer langen Datei | R | R | 7,4 | R |
| V5 | Excel-Tabelle | R | R | 14,1 | R |
| V6 | Tabelle im PDF | R | R | 23,0 | R |
| V7 | **Text nur im Bild**, Eskalationsgrenze auf Folie 1 | N | **R**, 7.500 € | 22,1 | R |
| V8 | Sprechernotiz | R | R | 15,7 | R |
| V9 | Scan ohne Textebene | T | **R**, 14.11.2025, 4 Restmängel | 21,9 | R |
| V10 | Mehrere Projekte | T, nur Velmora | **R**, Velmora und Tessaro | 15,2 | R |
| Z2 | Teams-Nachricht | F | F, mit Hinweis auf Teams | 11,8 | F |
| Z3 | Nicht in den Quellen | R | T, zweimal „Suche schlägt fehl“ | 12,4 | R |
| F1 | Word Fließtext | R | R | 26,2 | R |
| F2 | Word Tabelle | R | R | 13,4 | R |
| F3 | PDF Fließtext | R | R | 24,8 | R |
| F5 | **PDF als Scan** | N | **R** | 34,4 | R |
| F10 | **Text nur im Bild** | N | **R** | 16,9 | R |
| L4 | Höchstes Budget | R | **N, dann R** | 12,2 und 62,5 | R |
| L5 | Ist-Kosten über Plan | F, prüft 2 | **R**, alle 5 mit Beträgen | 49,4 | R |
| L6 | Go-live 2023 | F, 2 von 9 | R, alle 9, mit Vorbehalt | 39,2 | R |

**Summe der zehn Vergleichsfragen:** Standard 7 R, 2 T, 1 N. Copilot Harness **10 R**. Der Standard-Harness mit GPT-5 Reasoning kam auf 9 R, 1 N.

**Was auffällt:**

- **Er liest, was der Index nicht hat.** Beide Scans und beide Texte in Bildern gelangen. Mit dem Standard-Harness waren das vier Fehlschläge. Die Aktivitätsansicht zeigt, wie: Er holt die ganze Datei in die Sandbox, dann laufen eingebaute Skills für PDF oder PowerPoint.
- **Listen nach Bedingung gehen.** „Ist über Plan“ war beim Standard-Agenten falsch, hier vollständig, mit Plan, Ist und Differenz.
- **Die Antworten sind besser belegt.** Er nennt Datei, Stand und warum eine Fassung gilt, oft als Tabelle. Das hat die Anweisung schon beim Standard-Agenten verlangt, hier wird sie genauer befolgt.
- **Bei „nicht in den Quellen“ meldet er einen Fehler.** Die lexikalische Suche nach dem nicht vorhandenen Ort schlug zweimal fehl, statt nichts zu finden. Er erfand nichts, sagte aber auch nicht, dass es dazu nichts gibt.
- **Teams bleibt unsichtbar.** Ohne Work IQ (Preview) gibt es keine Teams-Nachrichten. Er weist aber darauf hin.

## 3. Was im Harness passiert

Die Aktivitätsansicht zeigt die Werkzeuge, die Microsoft dem Agenten mitgibt:

| Werkzeug | Was es tut |
|---|---|
| Lexikalische Suche | Stichwortsuche in den Wissensquellen |
| Semantische Suche | Bedeutungssuche |
| Textstellen abrufen | Textstellen zu einem Treffer |
| Ganze Datei abrufen | **Die ganze Datei** in die Sandbox laden |
| Filter nach Metadaten | Filtern nach Spalten der Bibliothek |
| Eingebaute Skills für Word, PDF, Excel, PowerPoint | Microsofts Lese-Anleitungen je Format |
| Python in der Sandbox | Rechnen, Tabellen, Dateien erzeugen |

**Grenzen, die er selbst nannte oder die sichtbar wurden:**

- **30 Suchaufrufe je Antwort.** So steht es in der Beschreibung des Suchwerkzeugs: „Budget: 30 calls per user turn“.
- **Rund 15 ganze Dateien je Antwort.** Bei der Excel-Aufgabe öffnete er 15 Dateien ganz und nahm den Rest aus Textstellen. Diese Werte markierte er gelb.
- **Kein Ordner-Listing.** Er kann nicht „alle Dateien im Archiv“ auflisten, nur suchen. Auf die Bitte, jede Projektmappe zu lesen, antwortete er zuerst, man möge den Ordner als ZIP hochladen. Der Eigenbau hat ein Werkzeug zum Auflisten von Ordnern und kennt dieses Problem nicht.
- **Er wählt seinen Weg jedes Mal neu.** Bei „höchstes Budget“ nahm er einmal den Filter nach Metadaten, fand keine Spalte „Budget“ und gab auf. Ein anderes Mal öffnete er die Angebote und fand die richtige Antwort.

## 4. Excel-Datei über alle Projekte

**Aufgabe:** Eine Excel-Datei mit allen Projekten im Archiv, mit Leitung, Partner, Budget, Plan, Ist, Abweichung und Go-live.

| Schritt | Ergebnis |
|---|---|
| Erster Versuch | Lehnte ab, bat um ZIP. Grund: kein Ordner-Listing |
| Zweiter Versuch, „nutze die Suche“ | Erstellte eine Projektübersicht als Excel-Datei in der Sandbox, mit Quellen, Lücken gelb markiert |
| Gegenprobe gegen die Sollwerte | **24 von 24 Projekten, 120 von 120 Feldern richtig.** 12 Abschlussberichte erkannt, auch das stimmt |
| Kosten | 195,6 Credits für drei Antworten, rund 2 USD |
| Dauer | rund 6 Minuten |

Der Standard-Agent kann keine Dateien erstellen. Copilot Chat legt bei solchen Listen zwei bis drei Projekte vor und hört auf. Das ist der klarste Fall, in dem der neue Harness etwas kann, was vorher ohne Code nicht ging.

## 5. Verlässlichkeit und Skills

Dieselbe Frage „Welches Projekt hat das höchste aktuelle Budget?“, je in einem neuen Chat:

| Lauf | Mit Skill? | Weg | Ergebnis | Credits |
|---|---|---|---|---:|
| 1 | nein | Nur Filter nach Metadaten | N, „nicht belastbar bestimmbar“ | 12,2 |
| 2 | nein | Angebote gesucht und geöffnet | R, Zellerau 268.000 € | 62,5 |
| 3 | ja | Skill, 24 Projekte verglichen | R, Zellerau | 145,3 |
| 4 | ja | Skill gewählt, dann wieder nur Metadaten | N | 25,5 |

**Der Skill** für Portfolio-Auswertungen schreibt einen festen Ablauf vor: nach beauftragten Angeboten und Abschlussberichten suchen, jede Datei ganz öffnen, in der Sandbox rechnen. Er verbietet ausdrücklich, wegen fehlender Metadaten aufzugeben. In Lauf 4 meldete die Oberfläche, dass der Skill genutzt wurde, und trotzdem gab der Agent nach vier Metadaten-Filtern auf.

**Was das heißt:** Ein Skill ist eine Anleitung, keine Programmierung. Das Modell kann ihr folgen, muss aber nicht. Der Eigenbau auf Foundry war bei zwei Listenfragen in je vier Läufen viermal richtig. Er hat ein Werkzeug, das Ordner auflistet, und muss nicht raten, wo die Daten liegen.

## 6. Workflows

Die Workflows in Studio sind die neue Fassung der Agent Flows auf dem Copilot Harness. Getestet wurde ein Workflow mit manuellem Start und einem Agent-Schritt, der den Projektwissen-Agenten aufruft.

| Beobachtung | Bewertung |
|---|---|
| **Bausteine:** Agent, Classify, Extract, Copilot, Human review, Connector, Function, Variable, If/Else, Loop | Wie Power Automate, ergänzt um KI-Schritte und Freigabe durch Menschen |
| **„Function“ heißt nicht Code.** Dahinter stehen Datenoperationen, HTTP, Datum, Zeitplan und Request | Wer rechnen will, macht das im Agent-Schritt in der Sandbox oder mit Ausdrücken |
| **Agent-Schritt** mit Text oder strukturierter Ausgabe (eigene Felder) | Gut, um Ergebnisse an feste Schritte weiterzugeben |
| **Testlauf eines Schritts** | Alle 12 Abschlussberichte ganz geöffnet, Plan und Ist richtig |
| **Nimmt die veröffentlichte Fassung des Agenten.** Der erste Lauf scheiterte mit „model isn't available“, weil dort noch Sonnet 4.6 stand | Nach Änderungen am Agenten immer veröffentlichen |
| **Ganzer Lauf erst nach Veröffentlichen des Workflows** | Nicht gemacht, damit kein Trigger aktiv wird |
| **Credits je Workflow-Lauf** | In der Oberfläche nicht angezeigt |

**Lohnt sich das?** Ja, für Abläufe mit festen Schritten und einer Stelle, an der jemand lesen und entscheiden muss. Beispiel: Neue Datei im Ordner, Agent liest sie und füllt strukturierte Felder, Freigabe durch einen Menschen, dann Eintrag in eine Liste. Der KI-Teil ist dann klein und prüfbar, der Rest deterministisch. Für Fragen im Chat bringt ein Workflow nichts.

## 7. Kosten

| | Credits | USD |
|---|---:|---:|
| Einzelfrage, Mittel aus 17 Fragen | rund 17 | 0,17 |
| Einzelfrage, Spanne | 7 bis 34 | 0,07 bis 0,34 |
| Liste nach Bedingung | 39 bis 63 | 0,39 bis 0,63 |
| Liste mit Skill, 24 Projekte verglichen | 145 | 1,45 |
| Excel über alle Projekte | 196 | 1,96 |
| **Alle Tests zusammen** | **rund 820** | **rund 8,20** |

Ein Credit kostet bei Pay-as-you-go 0,01 USD. Gelesen ist das vom Zähler in der Vorschau. Das Admin Center zeigte bis zum Ende der Tests noch keinen Verbrauch an. Die Workflow-Läufe sind nicht enthalten. Da der neue Harness ab dem ersten Bauschritt zählt, sind auch diese Testläufe kostenpflichtig, eine Copilot-Lizenz deckt nichts davon ab.

**Im Vergleich, gleiche Fragen:**

| Weg | Einzelfrage | Listenfrage | Mit Copilot-Lizenz |
|---|---:|---:|---|
| Standard-Harness, Nutzer ohne Lizenz | 2 Credits je generativer Antwort, plus Reasoning-Token falls Reasoning-Modell | ebenso | **frei** in Teams und Copilot Chat |
| **Copilot Harness** | 7 bis 34 Credits, 0,07 bis 0,34 USD | 39 bis 196 Credits | **nicht frei**, auch Bauen und Testen kosten |
| Eigenbau-Agent, gleiches Modell | 0,6 bis 6 Cent | 5 bis 16 Cent | entfällt, Azure-Rechnung |
| Eigenbau-RAG | 0,2 bis 2 Cent | 1 bis 3 Cent | entfällt |

**Einordnung:** Die Blogs nennen 100 bis 300 Credits für eine leichte Aufgabe. Gemessen waren es bei Einzelfragen 7 bis 34. Gegenüber dem Eigenbau ist der Harness bei Einzelfragen rund 5-mal, bei Listen 3- bis 10-mal teurer. Dafür entfallen Code, Betrieb, Anmeldung, Oberfläche und Sandbox.

**Hochgerechnet** [Annahme] je Person mit 10 Fragen und 2 Auswertungen am Arbeitstag, 20 Tage: 200 Fragen zu 17 Credits und 40 Auswertungen zu 100 Credits ergeben rund 7.400 Credits, also rund **75 USD je Person und Monat**. Mit dem Standard-Harness und Copilot-Lizenzen wären die Fragen frei, die Auswertungen gingen aber nicht.

## 8. Wo RAG hier steht

Alle Microsoft-Wege beginnen mit demselben Schritt: Suche im Index, Textstellen ans Modell. Der Unterschied liegt darin, was danach erlaubt ist.

| Stufe | Wer | Was das Modell bekommt | Scans, Bilder | Listen nach Bedingung |
|---|---|---|---|---|
| 1. Klassisches RAG | Copilot Chat, Retrieval API, Eigenbau-RAG | Textstellen aus dem Index | nein | nein |
| 2. RAG mit Umformulierung | Standard-Harness mit generativer Orchestrierung | Textstellen aus mehreren selbst formulierten Suchen | nein | selten |
| 3. RAG plus Reasoning | Standard-Harness mit GPT-5 Reasoning | wie 2, öfter gesucht | teilweise | teilweise |
| 4. **Agentisches Lesen** | **Copilot Harness**, Eigenbau-Agent | Ganze Dateien, Texterkennung, Rechnen | **ja** | **ja** |
| 5. Strukturierte Ablage | Projektkartei (siehe [Apps](../apps/README.md)), Dataverse | Einmal ausgelesene Tabelle | ja, beim Auslesen | **immer** |

Der Sprung von Stufe 3 auf 4 kommt nicht von einer besseren Suche, sondern vom Lesen der ganzen Datei. Das zeigt der Eigenbau besonders klar: Seine Suche ist simpler als Microsofts, trotzdem löst er alles außer Teams. Mehr zur Suche steht in [Index und Retrieval](index-und-retrieval.md) und [Microsoft RAG Architektur](microsoft-rag-architektur.md).

## 9. Ein verlässlicher Agent wie Claude Code, in Microsoft-Grenzen?

Gemeint ist ein Agent, dem man sagt: „Das war gut, mach das jedes Mal so“, und der liest, ändert und schreibt, wie gewünscht.

| Was man braucht | Wo es das bei Microsoft gibt | Stand im Test |
|---|---|---|
| **Ganze Dateien lesen, rechnen, Dateien schreiben** | Copilot Harness, Cowork | Geht gut, Excel fehlerfrei |
| **Vorgehen festhalten** | Skills im Copilot Harness, Skills in Cowork und bald in Copilot Chat | Wird genutzt, aber **nicht verlässlich befolgt** |
| **Sich Korrekturen merken** | Gedächtnis (Preview) | Nicht getestet |
| **Feste Schritte garantieren** | Workflows mit Agent-Schritt, Hooks (Preview) | Workflow geht, garantiert aber nur den Rahmen, nicht den Agent-Schritt |
| **Ergebnis prüfen** | Evaluation in Studio | Vorhanden, Bewertung bisher zu großzügig |
| **In SharePoint zurückschreiben** | Connector-Schritt im Workflow, Werkzeuge im Agenten | Nicht getestet |
| **Ordner durchgehen** | Fehlt bei der Wissensquelle. Geht über einen SharePoint-Connector als Werkzeug | Nicht getestet |
| **Kosten im Griff** | Grenze je Agent im Admin Center | Erst nach dem ersten Verbrauch einstellbar, Anzeige verzögert |

**Was fehlt zu Claude Code:** Claude Code arbeitet in einem Ordner, den es auflisten, lesen und ändern darf. Was es tut, steht im Verlauf, und mit Hooks und Tests lässt sich ein Ablauf erzwingen. Der Copilot Harness hat die Sandbox und die Skills, aber keinen Ordner, nur eine Suche. Darum wählt er seinen Weg jedes Mal neu und scheitert manchmal am ersten Schritt.

**Was heute am nächsten kommt:**

1. **Workflow als Gerüst:** Start, dann ein Connector-Schritt, der die Dateiliste eines Ordners holt. Dann eine Schleife über die Dateien, je Datei ein Agent-Schritt mit strukturierter Ausgabe. Am Ende ein fester Schritt, der die Ergebnisse in eine Liste oder Excel-Datei schreibt. Die KI liest nur je eine Datei, der Rest ist deterministisch. Das ist die Form, die sich für „jedes Mal genau so“ eignet. Gebaut ist sie im Test noch nicht.
2. **Agent für die freien Fragen:** Der Copilot Harness für alles, was nicht vorher feststeht. Mit dem Wissen, dass er einen von vier Läufen verliert und man nachfragen muss.
3. **Eigenbau, wenn es verlässlich und günstig sein muss:** Foundry plus eigene Werkzeuge. Mehr Kontrolle, ein Zehntel der Kosten, aber Code und Betrieb.

## 10. Power Automate, Dataverse und kleine Teams

**Klassische Automatisierung** setzt voraus, dass Daten an einer festen Stelle in fester Form liegen. Ein Formular schreibt in eine Liste, ein Flow reagiert auf eine neue Zeile, Dataverse hat Spalten mit Typen. Power Automate liest Felder, nicht Texte.

**Ein kleines Team** arbeitet oft anders. Es legt in einem SharePoint-Projekt Angebote, Protokolle, Präsentationen und Scans ab, mit Namen wie „final_v2_überarbeitet“. Einen Datenstand mit Feldern gibt es nicht. Für Power Automate ist das wertlos: Es gibt keine Zeile, auf die ein Flow reagieren könnte, und kein Feld „Budget“.

**Hier ändert sich etwas mit dem neuen Harness.** Er macht aus der Ablage selbst die Datenquelle. Die Excel-Aufgabe zeigt das: aus 24 unordentlichen Projektordnern eine fehlerfreie Tabelle, ohne dass jemand vorher ein Schema gebaut hat.

| | Klassisch, Power Automate plus Dataverse | Agentisch, Copilot Harness |
|---|---|---|
| **Voraussetzung** | Feste Ablage mit Feldern, jemand pflegt sie | Dateien, wie sie sind |
| **Aufwand vorher** | Hoch: Schema, Formulare, Disziplin im Team | Niedrig: Wissensquelle anschließen |
| **Verlässlichkeit** | Sehr hoch, gleiche Eingabe gibt gleiches Ergebnis | Mittel, je Lauf anders |
| **Kosten je Lauf** | Fast null | 0,10 bis 2 USD |
| **Für ein kleines Team** | Lohnt meist nicht, die Pflege ist zu teuer | **Passt**, solange jemand das Ergebnis prüft |
| **Für eine große Organisation** | Passt, die Pflege trägt sich | Teuer und schwer zu kontrollieren |

**Die gute Mischung für das kleine Team**, nach Microsofts eigenem Muster „Data Curation“:

1. **Einmal im Monat oder auf Knopfdruck** läuft ein Copilot-Harness-Agent über die Projektablage und schreibt eine Projektübersicht als Excel oder SharePoint-Liste. Das kostet ein paar Euro.
2. **Diese Übersicht ist dann die feste Datenquelle.** Fragen wie „welches Projekt hat das höchste Budget“ beantwortet sie immer gleich, und Power Automate kann auf Änderungen reagieren.
3. **Für Einzelfragen** reicht Copilot Chat oder der Standard-Agent. Mit Copilot-Lizenz sind sie frei, sie finden aber keine Scans.
4. **Ein Mensch prüft** die Übersicht, bevor sie gilt. Die Excel-Datei markiert selbst, was sie nur aus Textstellen hat.

Das ist im Kern, was die selbst gebaute Projektkartei (siehe [Apps](../apps/README.md)) tut. Jetzt geht es auch ohne Code, aber mit Credits je Lauf.

## 11. Welcher Weg wofür

| Aufgabe | Empfehlung | Warum |
|---|---|---|
| Einzelfragen an das Archiv, viele Nutzer mit Copilot-Lizenz | Standard-Harness oder Copilot Chat | Frei in der Lizenz, 7 bis 9 von 10 richtig |
| Fragen, bei denen Scans oder Bilder wichtig sind | Copilot Harness, oder Texterkennung beim Ablegen | Nur agentisches Lesen sieht sie |
| Liste oder Auswertung über alle Projekte, einmalig | Copilot Harness | Excel fehlerfrei, rund 2 USD |
| Dieselbe Auswertung regelmäßig, verlässlich | Workflow mit Schleife über Dateien, oder Eigenbau | Skill allein reicht nicht |
| Günstig und nachvollziehbar je Cent | Eigenbau auf Foundry | Ein Zehntel der Kosten, jeder Schritt im Log |
| Teams-Nachrichten einbeziehen | Work IQ (Preview) oder Eigenbau mit dem Graph-Recht `Chat.Read` | Sonst unsichtbar für alle Wege |
| Feste Abläufe mit Freigabe | Workflow mit Human review | Rahmen deterministisch, KI nur an einer Stelle |

## 12. Bedienung und Stolpersteine

- **Anlegen per CLI geht.** `pac copilot init --authoring-mode cli-copilot` legt einen Agenten auf dem neuen Harness an. Mit `pac copilot pack` und `pac solution import` kommt er in die Umgebung. `push` geht erst danach und übernimmt die Wissensquelle nicht.
- **Claude-Modelle** sind voreingestellt, laufen aber erst nach Freigabe für Anthropic durch einen Admin.
- **Workflows nutzen die veröffentlichte Fassung** des Agenten. `pac copilot publish` reicht.
- **Kosten live nur in der Vorschau.** Der Zähler steht oben im Chat. Im Admin Center erscheint der Agent erst nach Stunden, vorher lässt sich keine Grenze setzen.
- **Studio im Browser** lud nach Seitenwechseln oft nur weiße Seiten. Ein neuer Tab half jedes Mal.
- **Jede Frage im neuen Chat**, sonst rechnet der Zähler mehrere Fragen zusammen und Kontext läuft mit.
- **Pay-as-you-go-Plan beachten.** Die Agenten auf dem neuen Harness laufen nur, solange die Umgebung mit einem Abrechnungsplan verknüpft ist. Wird der Plan gelöst, hören sie auf zu antworten. Das ist zugleich der sicherste Schalter gegen ungewollte Kosten.

## Offene Punkte

- Stimmen die Credits im Admin Center, sobald sie erscheinen, mit den rund 820 vom Zähler in der Vorschau überein?
- Workflow mit Connector, Schleife über die Dateien und Excel-Ausgabe bauen. Das ist der Weg zu „jedes Mal genau so“.
- Gedächtnis und Work IQ testen. Liest der Agent dann die Teams-Nachricht?
- Cowork mit denselben Fragen, sofern im Tenant freigeschaltet.
