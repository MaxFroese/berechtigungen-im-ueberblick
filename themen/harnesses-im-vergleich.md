# Harnesses im Vergleich

Stand: 6. November 2026. Zurück zur [Startseite](../README.md). Quellen sind die Microsoft-Doku und -Blogs, verlinkt unten. Wo nur Blogs Dritter etwas sagen, steht das dabei.

**Kurz gesagt:** Ein Harness ist die Laufzeit zwischen Modell und Agent. Sie entscheidet, wann das Modell gefragt wird, was es zu sehen bekommt und welches Werkzeug als Nächstes läuft. Copilot Studio hat seit August 2026 mehrere davon. Der **Standard-Harness** führt aus, was gebaut wurde: Topics, Pfade, Flows. Der **Microsoft Copilot Harness** (bis Oktober „GitHub Copilot harness“) bekommt ein Ziel und plant selbst, mit Sandbox, Dateien, Skills und Gedächtnis. Er kostet ab dem ersten Klick Credits, auch mit Copilot-Lizenz. Derselbe Harness steckt hinter **Copilot Cowork**. Der **Copilot-Chat-Harness** ist der von Copilot Chat selbst und dient nur dazu, Copilot Chat zu erweitern.

Der in [Copilot Studio im Test](copilot-studio.md) geprüfte Projektwissen-Agent läuft auf dem Standard-Harness, erkennbar an seinen Topics. Ein Nachbau auf dem Microsoft Copilot Harness ist in [Copilot Harness im Praxistest](copilot-harness-praxistest.md) beschrieben.

## 1. Was ein Harness ist

Das Modell denkt, der Harness führt. Er legt fest:

- wann das Modell gerufen wird und mit welchem Kontext,
- welche Werkzeuge, Wissensquellen, MCP-Server oder anderen Agenten es nutzen darf,
- was mit dem Ergebnis passiert: antworten, weitersuchen, neu versuchen, abbrechen,
- wie abgerechnet wird.

Dasselbe Modell kann im einen Harness eine Frage nach drei Suchen beantworten und im anderen eine Stunde an einem Excel-Abgleich arbeiten. Der Unterschied liegt also nicht nur im Modell, sondern auch in dem, was der Harness ihm erlaubt.

## 2. Die Harnesses im Überblick

| | Copilot-Chat-Harness | Standard-Harness | Microsoft Copilot Harness |
|---|---|---|---|
| **Wofür** | Copilot Chat um eigenes Wissen und Aktionen erweitern | Regelbasierte Agenten, feste Gesprächsabläufe, Agent Flows | Lange, mehrstufige Aufgaben über mehrere Systeme mit Dateien als Ergebnis |
| **Wie er arbeitet** | Wie Copilot Chat selbst | Folgt gebauten Topics, Variablen und Pfaden; generative Orchestrierung wählt zwischen ihnen | Nimmt ein Ziel, plant Schritte, prüft Zwischenergebnisse, versucht Alternativen |
| **Wenn etwas schiefgeht** | | Läuft den gebauten Pfad | Wiederholt, nimmt einen anderen Weg |
| **Dateien** | | Kein Schwerpunkt | Liest und erstellt Word, Excel, PowerPoint, PDF |
| **Code** | | Code Interpreter, einzuschalten | Bash-Container in einer Sandbox, immer an |
| **Skills, Gedächtnis** | | Nein | Ja, Gedächtnis als Preview |
| **Veröffentlichen** | Nur intern | Intern und extern, viele Kanäle | Intern und extern, weniger Kanäle |
| **Kosten** | In der Copilot-Lizenz, mit Fair-Use-Grenzen | In der Copilot-Lizenz, wenn ein lizenzierter Nutzer in M365 fragt, sonst feste Sätze | Immer Credits nach Verbrauch, ab dem Bauen |

Daneben gibt es den **Customer Conversation Harness** für Kundenservice per Web-Chat und Telefon, mit Übergabe an Menschen. Er rechnet ebenfalls nach Verbrauch ab und spielt hier keine Rolle.

Im Power Platform Admin Center zeigt die Agentenliste unter **Manage, Copilot Studio** eine Spalte **Harness** mit den Werten „GitHub Copilot“, „Standard“ und „Copilot Chat“. Daran lässt sich ablesen, welcher Agent auf welchem läuft.

## 3. Standard-Harness

Er ist das klassische Copilot Studio: Der Ersteller definiert die Bausteine, der Harness führt sie aus.

**Seine Aufgaben:**

- **Topics auslösen.** Eine Nutzerabsicht oder ein Systemereignis wie „Nachricht erhalten“ startet ein Topic, etwa für Suche, Eskalation oder Fehlerbehandlung.
- **Generative Orchestrierung.** Das Modell wählt zwischen Topics, Wissensquellen und Aktionen und formuliert die Suche selbst. Das ist vermutlich der Grund, warum im Test die Synonym-Frage gelang.
- **Wissen abfragen.** SharePoint, Dateien, Websites, Graph Connectors, Dataverse, Azure AI Search. Bei SharePoint bekommt er Textstellen aus demselben Index wie Copilot, deshalb fehlen Scans und Text in Bildern.
- **Aktionen ausführen.** Connectors, MCP, Prompts, Agent Flows, Computer Use.
- **Fest antworten.** Classic Answers sind fest geschriebene Antworten ohne Modell.

**Stärken:** vorhersagbar, schnell, viele Kanäle (Teams, M365, Website, SharePoint, WhatsApp, Dynamics Contact Center), eigene Entitäten, eigene Anmeldung, Azure AI Search als Wissen.

**Grenzen:** Kontext ist über Topics, Variablen und Unteragenten verteilt. Bei komplexen Abläufen müssen Übergaben und Ausnahmen selbst gebaut werden. Genau das zeigte der Test: Bei „Ist über Plan“ schaut der Agent zwei Berichte an und hört auf.

**Abrechnung (feste Sätze):**

| Baustein | Credits | Mit Copilot-Lizenz in M365 |
|---|---|---|
| Classic Answer | 1 | frei |
| Generative Answer | 2 | frei |
| Agent Action (Trigger, Deep Reasoning, Topic-Wechsel) | 5 | frei |
| Tenant Graph Grounding | 10 | frei |
| Agent Flow, je 100 Aktionen | 13 | frei, nur mit Trigger „When an agent calls the flow“ |
| Reasoning-Modell, je 1.000 Token | 10 | frei |
| Computer Use | 5 je Aktion | **nicht** frei |

„Frei“ gilt nur, wenn ein Nutzer mit Copilot-Lizenz angemeldet in Copilot Chat, Teams oder SharePoint fragt. Bei Website-Kanal, Nutzern ohne Lizenz oder autonomen Läufen wird gezählt. Abgerechnet wird erst nach dem Veröffentlichen, Testchat und Testläufe im Flow-Designer kosten nichts. Das passt zu den 0 Credits, die am 6. November nach den Evaluationsläufen angezeigt wurden.

## 4. Microsoft Copilot Harness

Bis Oktober hieß er „GitHub Copilot harness“, weil ihn das GitHub-Team gebaut hat und er auf dem GitHub Copilot SDK läuft. Seit dem 28. Oktober heißt er Microsoft Copilot Harness und steckt laut Microsoft hinter Copilot Studio, Copilot Code, der GitHub-Copilot-App, CLI und VS Code. Der Name täuscht: Die Daten gehen nicht an den GitHub-Copilot-Dienst, es gelten die Zusagen von Copilot Studio.

**Allgemein verfügbar seit 3. August 2026.**

**Seine Aufgaben:**

- **Planen statt Pfade abgehen.** Beschrieben werden Ziel, Grenzen und Werkzeuge. Er zerlegt die Aufgabe, ruft Werkzeuge, bewertet das Ergebnis und entscheidet, ob er weitermacht, umsteuert, delegiert oder aufhört. Microsoft nennt das Agentic Loop.
- **In der Sandbox arbeiten.** Jede Aufgabe läuft in einem eigenen Bash-Container mit Dateisystem. Dort kann er rechnen, Dateien ganz lesen und neue Dateien schreiben.
- **Dateien als Ergebnis.** Word, Excel, PowerPoint und PDF liest und erstellt er selbst.
- **Skills.** Wiederverwendbare Anweisungspakete, zum Beispiel ein Prüfverfahren. Sie ersetzen die Prompts des Standard-Harness.
- **Hooks (Preview).** Workflows, die an festen Punkten laufen: Sitzungsbeginn, vor einem Werkzeugaufruf, bei Fehlern. Sie ersetzen Topics.
- **Gedächtnis (Preview).** Behält, was ein Nutzer früher gesagt oder korrigiert hat.
- **Unteragenten mit gemeinsamem Kontext.** Er kann mehrere Teilaufgaben parallel laufen lassen und behält den Überblick.
- **Work IQ (Preview).** Zugriff auf Mail, Kalender, Dateien, Teams-Nachrichten und Personen. Das beträfe die Testfrage Z2, deren Antwort nur in einer Teams-Nachricht steht.
- **Foundry IQ.** Eine in Foundry gebaute Wissensbasis als Quelle, siehe [Eigener Index mit Foundry IQ](eigener-index-foundry-iq.md).

**Was er gegenüber dem Standard-Harness nicht kann:**

| Fehlt | Bemerkung |
|---|---|
| Computer Use | geplant |
| Azure AI Search als Wissen | nicht unterstützt |
| Prompts als Werkzeug | durch Skills ersetzt |
| Kanäle SharePoint, WhatsApp, Dynamics Contact Center | nicht unterstützt |
| Eigene Anmeldung | nur Entra ID oder anonym |
| Eigene Entitäten | nicht unterstützt |
| Orchestrierung einstellen | geht nicht, er orchestriert immer selbst |

**Ein Wechsel ist nicht möglich.** Ein Agent bleibt auf dem Harness, mit dem er angelegt wurde. Ein Umzug heißt neu bauen.

**Sprache:** Es gibt keine Spracheinstellung mehr, die Antwortsprache kommt aus der Anweisung. Für die englischen Antworten, die GPT-5 Reasoning im Test trotz deutscher Anweisung gab, wäre das der Hebel.

## 5. Wo die Abrechnung auseinanderläuft

Das ist der eigentliche Bruch zwischen den beiden Harnesses:

| | Standard | Microsoft Copilot Harness |
|---|---|---|
| **Copilot-Lizenz deckt ab** | Fragen lizenzierter Nutzer in M365 | **Nichts** |
| **Was gezählt wird** | Feste Sätze je Baustein | Token, Werkzeuge (auch Wissen und MCP) und der Harness selbst |
| **Ab wann** | Ab Veröffentlichung | **Ab dem ersten Bauschritt:** Erstellen in natürlicher Sprache, Preview, Tests, Evaluationen |
| **Größenordnung** | 2 Credits je generativer Antwort | Laut Blogs Dritter 100 bis 300 Credits für eine leichte Aufgabe, 300 bis 700 für eine mittlere. Microsoft zeigt die Spannen nur als Grafik. Im eigenen Test waren es 7 bis 34 Credits je Einzelfrage, siehe [Praxistest](copilot-harness-praxistest.md#7-kosten) |
| **Wer zahlt** | Umgebung, aus Paketen oder Pay-as-you-go | Ebenso |

Ein Credit kostet bei Pay-as-you-go 0,01 USD, ein Paket mit 25.000 Credits 200 USD im Monat. Eine leichte Aufgabe läge nach den Blogzahlen bei 1 bis 3 USD.

**Für einen Testtenant heißt das:** Ein Evaluationslauf mit 12 Fragen auf dem neuen Harness kostet Credits, auf dem Standard-Harness im Entwurf nicht. Er läuft über den Pay-as-you-go-Plan der Umgebung auf das verknüpfte Azure-Abo. Vor dem ersten Versuch lohnt eine Monatsgrenze für den Agenten: Power Platform Admin Center, **Licensing, Copilot Studio, Manage Agents, Set limit**, dazu **Stop usage**. Der Haken: Der Agent erscheint dort erst nach dem ersten Verbrauch. Azure-Budgets warnen nur, sie stoppen nichts.

**Wer bauen darf:** Für den neuen Harness gibt es keinen eigenen Schalter. Es gelten die Umgebungsrechte und die Einstellung „Copilot Studio authors“. Wer Agenten bauen darf, darf auch teure bauen.

## 6. Copilot Cowork und Copilot Chat

Cowork ist kein Studio-Agent, sondern Microsofts eigener Agent in der Microsoft-365-Copilot-App. Microsoft beschreibt den neuen Studio-Harness als die Technik „hinter Copilot Cowork“. Cowork ist also ein fertig konfigurierter Agent auf diesem Harness, Studio erlaubt, einen eigenen zu bauen.

| | Copilot Chat | Copilot Cowork |
|---|---|---|
| **Wofür** | Fragen, Ideen, erste Entwürfe, einzelne Aktionen | Abgegebene Arbeit: mehrstufig, lang, über viele Werkzeuge, mit fertigem Ergebnis |
| **Wo es läuft** | Im Gespräch | In der Cloud, auch wenn der Laptop aus ist |
| **Modelle** | Von Microsoft gewählt | Zur GA Anthropic (Opus 4.8, Sonnet 4.6), in Frontier mit Auswahl |
| **Arbeitsdaten** | Graph-Grounding | Work IQ: Mail, Kalender, Dateien, Teams, Personen |
| **Skills** | Ab November auch in Chat | Ja |
| **Lokaler Browser** | Nein | Edge auf dem eigenen Gerät, auch auf Seiten mit Anmeldung (Frontier) |
| **Lizenz** | In der Copilot-Lizenz, seit 2. bzw. 9. November mit Fair-Use-Grenzen | Copilot-Lizenz **und** Credits je Aufgabe |
| **Standard** | An | Aus, Admin schaltet frei |

**Weniger beschränkt, aber nicht frei:** Cowork beantwortet auch normale Fragen und wirkt weniger eingeschränkt als Copilot Chat. Dann kostet aber auch jede normale Frage Credits. Microsoft zählt bei Cowork Modellantworten, Werkzeug- und Skill-Aufrufe, Bilder und Browser-Aufgaben. In Chat ist dieselbe Frage in der Lizenz enthalten. Seit November teilen sich Chat und Cowork Startseite und Verlauf, Cowork kann auf frühere Chats zugreifen. Die Grenze verschwimmt also in der Oberfläche, nicht in der Abrechnung.

**Gemeinsamer Topf:** Credit-Pakete gelten für Copilot Studio und für Cowork. Was Studio-Agenten verbrauchen, fehlt Cowork. Studio wird im Power Platform Admin Center gesteuert, Cowork im Microsoft 365 Admin Center unter **Copilot, Cost Management**. Dort gibt es Grenzen je Tenant, Gruppe und Nutzer. Die Einstellung **Auto-apply new services** ist standardmäßig an. Neue kostenpflichtige Dienste fallen dann ohne Rückfrage unter bestehende Ausgaberegeln.

## 7. Copilot-Chat-Harness

Hierzu sagt Microsoft am wenigsten. Belegt ist: Er nutzt dieselbe Laufzeit wie Copilot Chat, ist „ideal, um Copilot-Chat-Erlebnisse anzupassen“, und lizenzierte Nutzer nutzen ihn im Rahmen der Fair-Use-Grenzen. Laut Blogs Dritter veröffentlicht er nur intern. Vermutlich sind damit Agenten gemeint, die wie mit dem Agent Builder direkt in Copilot Chat laufen. Geprüft ist das nicht.

Praktisch heißt das: Er kann das, was Copilot Chat kann, und nicht mehr. Scans, Listen nach einer Bedingung und Dateien als Ergebnis gehen dort genauso wenig.

## 8. Welcher wofür

| Aufgabe | Harness | Warum |
|---|---|---|
| Fragen an ein Projektarchiv, viele Nutzer | Standard | Schnell, in der Lizenz enthalten, im Test 7 von 10 richtig |
| Copilot Chat um eine Wissensquelle ergänzen | Copilot Chat | Kein eigener Agent nötig, nur intern |
| Liste nach Bedingung, z. B. „Ist über Plan“ | Copilot Harness oder strukturierte Ablage | Er kann alle Berichte in der Sandbox öffnen und rechnen. Im Praxistest gelang das, aber nicht in jedem Lauf |
| Scan oder Bild lesen | Copilot Harness | Er liest ganze Dateien statt Indexstellen. Im Praxistest bestätigt |
| Rechnungen gegen Bestellungen prüfen, Ausnahmen weiterleiten | Copilot Harness | Microsofts Paradebeispiel |
| Einmalige große Aufgabe für eine Person | Cowork | Kein Bauen nötig, Kosten pro Aufgabe sichtbar |
| Kundenchat auf der Website, WhatsApp | Standard | Kanäle fehlen im Copilot Harness |

Microsofts Empfehlung für bestehende Agenten: Nicht umziehen, nur weil es den neuen Harness gibt. Ein einfacher Fall wird dort langsamer und teurer, nicht besser.

## 9. Widersprüche in der Doku

- **Reasoning-Modelle:** Der Vergleichsleitfaden sagt, Reasoning-Modelle gibt es „nur im GitHub Copilot Harness“. Die Preisseite des Standard-Harness rechnet sie aber ab, und GPT-5 Reasoning lief im Test auf dem Standard-Harness. Vermutlich ist der Leitfaden älter oder meint neuere Modelle.
- **Ab wann der Standard-Harness zählt:** Die Microsoft-Doku zum neuen Harness sagt „nach dem Veröffentlichen“. Die Preisseite des Standard-Harness nennt das nur für Agent Flows ausdrücklich.
- **„Apps (Preview)“** verlangte im Test einen eigenen Credit-Topf. Das passt dazu, dass Apps aus Copilot Studio über die *Copilot Managed Runtime* laufen, die im Microsoft 365 Admin Center abgerechnet wird.

## 10. Erste Probefrage auf dem neuen Harness

Am 6. November 2026 wurde der Projektwissen-Agent auf dem Microsoft Copilot Harness nachgebaut, mit derselben Anweisung und derselben SharePoint-Site.

| | |
|---|---|
| **Anlegen** | Per CLI mit `pac copilot init --authoring-mode cli-copilot`, dann als Lösung importiert. Die Wissensquelle übernimmt die CLI bei diesem Typ nicht, sie kam von Hand in der Oberfläche dazu |
| **Modell** | Voreingestellt war Claude Sonnet 4.6. Das scheitert mit „You don't have access to the model“, solange Anthropic im Tenant nicht freigegeben ist. Genommen: GPT 6.1 Sol |
| **Frage** | „Wie lautet der Wartungscode für die Pumpstation Hafen?“, der Code steht nur in einem Scan |
| **Antwort** | **QUINDARO-5512**, mit dem Scan als Quelle. Richtig. Der Standard-Agent fand ihn in keinem Lauf |
| **Weg** | Lexikalische und semantische SharePoint-Suche, dann Abruf der ganzen Datei und Microsofts eingebaute Lese-Skills für PDF und PowerPoint |
| **Kosten** | 34,44 Credits laut Zähler in der Vorschau, rund 0,34 USD. Deutlich unter den 100 bis 300 Credits aus den Blogs |
| **Admin Center** | Der Agent stand nach dem Lauf noch nicht unter Manage Agents, eine Grenze ließ sich also noch nicht setzen |

Damit ist die Vermutung aus [Copilot Studio im Test](copilot-studio.md) belegt: Dieser Harness öffnet die Datei selbst, statt nur Indexstellen zu nehmen. Die ausführlichen Tests, darunter Excel-Ausgabe, Workflow, Skill, Kosten und der Vergleich mit einem [Eigenbau auf Foundry](eigenbau-agent.md), stehen in [Copilot Harness im Praxistest](copilot-harness-praxistest.md).

## Offene Punkte

- Dieselben Fragen in Cowork stellen und die Credits je Aufgabe ablesen.
- Welche Werte zeigt die Harness-Spalte im Admin Center für die getesteten Agenten?
- Ist **Auto-apply new services** im Cost Management des M365 Admin Centers in einem frischen Tenant an, und welche Dienste fallen darunter?

## Quellen

Microsoft:
- [Harnesses in Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/harnesses-overview)
- [Compare Copilot Studio agent harnesses](https://learn.microsoft.com/en-us/microsoft-copilot-studio/guidance/choose-harness)
- [Agents powered by the GitHub Copilot harness overview](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/overview), mit Feature-Tabelle
- [Overview of usage-based billing](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/billing-credit-overview)
- [Standard harness licensing](https://learn.microsoft.com/en-us/microsoft-copilot-studio/billing-licensing)
- [Billing rates and management](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-messages-management)
- [Manage costs for agents powered by the GitHub Copilot harness](https://learn.microsoft.com/en-us/power-platform/admin/manage-usage-github-copilot-harness)
- [Usage-based billing for Copilot Credits](https://learn.microsoft.com/en-us/microsoft-365/copilot/usage-based-billing-overview-copilot-credits)
- [Introducing a new harness for Copilot Studio](https://techcommunity.microsoft.com/blog/copilot-studio-blog/more-powerful-agents-and-workflows-for-autonomous-business-processes-introducing/4542969), 3. August 2026
- [What's new in Copilot Studio, August 2026](https://www.microsoft.com/en-us/copilot/blog/copilot-studio/new-and-improved-github-copilot-harness-agent-skills-and-richer-context/)
- [Introducing the Microsoft Copilot harness](https://blogs.microsoft.com/blog/2026/10/28/introducing-the-microsoft-copilot-harness-unlocking-the-potential-of-truly-multi-model-agentic-work/), 28. Oktober 2026
- [Copilot Cowork is now generally available](https://www.microsoft.com/en-us/microsoft-365/blog/2026/06/16/copilot-cowork-is-now-generally-available/), 16. Juni 2026
- [What's new in Microsoft Copilot, October 2026](https://techcommunity.microsoft.com/blog/microsoft-copilot-blog/what%E2%80%99s-new-in-microsoft-copilot--october-2026/4568528)

Dritte, für Copilot-Chat-Harness und Kostenspannen:
- [RagnarPitla: Which harness](https://github.com/RagnarPitla/build-copilot-studio-agents-V2-Aug-2026/blob/main/docs/01-which-harness.md)
- [A Guide to Cloud: GitHub Copilot Harness explained](https://www.aguidetocloud.com/blog/copilot-studio-github-copilot-harness-explained/)
- [Trusted Tech Team: Copilot Cowork Pricing](https://www.trustedtechteam.com/blogs/microsoft-365/copilot-cowork-pricing-explained)
