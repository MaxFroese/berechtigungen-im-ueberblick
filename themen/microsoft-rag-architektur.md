# Microsoft-RAG-Architektur

Stand: 4. November 2026 aus Doku und Fachliteratur, am 5. November um Messungen ergänzt. Zurück zur [Startseite](../README.md).

Diese Seite ist die Vertiefung: wie Microsofts Suche laut Quellen aufgebaut ist. Die Einführung und alle Messwerte stehen in [Index und Retrieval](index-und-retrieval.md).

**Kurz gesagt:** Microsoft hat nicht ein RAG-System, sondern eine gemeinsame Index- und Abrufschicht, auf der mehrere Produkte sitzen. Im Hintergrund läuft eine ständige Verarbeitung, die Inhalte aufnimmt, indexiert und einbettet. Bei einer Anfrage holt sich das Modell daraus Textstellen, und die Rechte des Nutzers begrenzen, was es sehen darf.

Die Interna sind nur teilweise öffentlich, deshalb sind die Aussagen markiert:

- **[Beleg]:** von Microsoft dokumentiert.
- **[Extern]:** aus Sicherheitsforschung oder Fachquellen, teils älter.
- **[Schluss]:** technische Ableitung.
- **[Offen]:** dazu wurde nichts Belastbares gefunden.
- **[Gemessen]:** im Testtenant selbst beobachtet, am 5. November. Eine kleine Stichprobe mit erfundenen Daten.

## 1. Die Begriffe ohne Marketing

| Begriff | Technisch eher |
|---|---|
| **Microsoft Graph** | API und Datenmodell über M365-Inhalte, plus Signale wie Personen und Aktivität |
| **Substrate** | Gemeinsame Speicher- und Indexschicht hinter vielen M365-Diensten [Extern] |
| **Microsoft Search** | Klassische Suche mit Stichwörtern, KQL und Personalisierung |
| **Semantic Index** | Vektorindex neben dem Stichwortindex |
| **Copilot Search** | Suchoberfläche mit KI-Antwort aus Copilot Chat |
| **Retrieval API** | RAG als Dienst: Frage rein, Textstellen raus |
| **Work IQ** | Sammelbegriff, siehe Abschnitt 5 |
| **Copilot Connectors** | Aufnahme externer Daten in Graph und Index |
| **Foundry IQ** | Eigene Wissensdatenbank auf Azure AI Search, außerhalb dieser Schicht. Kann die Retrieval API aber als Quelle nutzen |

„Work-IQ-Adapter“ ist kein Microsoft-Begriff. Das Bild passt aber halbwegs: Ein Teil von Work IQ ist tatsächlich eine dünne Schicht vor Graph.

## 2. Der Hintergrundprozess: Wie etwas in den Index kommt

```text
Datei geändert
  → Warteschlange (hinzufügen, ändern, löschen)
  → Crawl: Text, Metadaten, Sprache
  ├→ Indexer: Stichwortindex
  └→ Chunks → Embeddings → Vektorindex
```

Dieses Bild ist eine Rekonstruktion aus den folgenden Punkten, kein offizielles Diagramm.

- **[Beleg] Crawl und Indexer:** Beide sind geteilte Dienste, die Warteschlangen abarbeiten, ohne Echtzeit-Dashboard. Diese Beschreibung stammt von 2018 bis 2020, das Grundprinzip dürfte aber geblieben sein.
- **[Beleg] Vektoren:** Microsoft nennt vektorisierte Indizes, und `relevanceScore` ist die Kosinus-Ähnlichkeit zwischen Anfrage und Textstelle.
- **[Gemessen] Vektoren:** Über die Retrieval API war der Wert in 23 Messläufen immer genau 0,0, und ganze Sätze fanden nichts. Eine aktive Vektorsuche war im Test also bisher nicht sichtbar. Copilot Chat fand dagegen einen ganzen Satz, sofern das Fachwort darin stand.
- **[Beleg] Aktualität:** Neue Dateien auf Sites mit mindestens zwei berechtigten Nutzern werden täglich indexiert, Änderungen an bereits indexierten Dateien laut Doku sofort. Mails kommen fast in Echtzeit in den Nutzerindex.
- **[Gemessen] Aktualität:** Stichwörter fanden sich je nach Site nach bis zu 35 Minuten, nach 1,5 bis 2 Stunden und nach rund 4 Stunden. Graph-Suche und Retrieval API liefen teils auseinander: Bei einem Bestand von 38 Dateien fand die Graph-Suche nach 29 Minuten alle, die Retrieval API nach 53 Minuten noch keine.
- **[Beleg] Formate:** Semantisch und hybrid geht nur bei `.doc`, `.docx`, `.pptx`, `.pdf`, `.aspx` und `.one`. Alle anderen Formate, also auch Excel, bleiben bei der Stichwortsuche. Bilder und Diagramme werden nicht gelesen.
- **[Gemessen] Formate:** Scan und Text im Bild wurden nie gefunden, passend zur Doku. Kunstwörter in Excel-Blättern, Sprechernotizen und PDF-Tabellen wurden gefunden.
- **[Beleg] Ort:** Der Tenant-Index liegt in einem isolierten Container in der Region der SharePoint-Site, bei EU-Kunden in der EU.
- **[Beleg] Gemeinsamer Schalter:** Wird eine Site aus der Suche ausgeschlossen, fällt sie aus beiden Indizes. Das spricht für eine gemeinsame Pipeline.
- **[Extern] Teams:** Nachrichten liegen in Cosmos DB, eine Kopie landet für die Compliance-Suche im Exchange-Postfach (Quelle von 2021). Die Copilot-Suche nutzt vermutlich einen ähnlichen Weg [Schluss].
- **[Schluss] Zwei Geschwindigkeiten:** Dass neue Dateien „täglich“, Änderungen aber „sofort“ laufen, spricht für einen getrennten Batch-Weg bei der Ersterfassung.
- **[Offen]:** Chunk-Größe, Embedding-Modell, ANN-Verfahren und der genaue Auslöser. Hinweise auf DiskANN betrafen Azure AI Search, nicht Copilot.
- **[Gemessen] Textstellen:** Die Retrieval API lieferte je Datei 136 bis 4.731 Zeichen, im Mittel rund 570. Das zeigt die Länge der Treffer, nicht die Chunk-Größe im Index.

Der Nutzerindex deckt nicht alles ab: Microsoft nennt ihn für OneDrive ein „personalized working set“, also nicht den gesamten Inhalt.

## 3. Der Abfragepfad: Was bei einer Frage passiert

```text
Prompt
 → Router wählt das Modell
 → Modell entscheidet: Suche nötig?
 → Anfrage wird umgeschrieben
 → Suche je Quelle: Dateien, Mail, Meetings, Teams, Personen, Connectoren
 → Stichwort + Vektor, nur Berechtigtes
 → Textstellen mit Quellen
 → Modell schreibt Antwort
 → Nachbearbeitung, Zitate, Filter
```

- **[Beleg]:** Prompt an Copilot, Zugriff auf Graph und Index, geänderter Prompt ans Modell, danach erneut Graph und Index zur Nachbearbeitung. Dazu kommen Modell-Router, hybrider Index und Umschreiben der Anfrage.
- **[Extern]:** Extrahierte Systemprompts zeigten 2024 ein Werkzeug `search_enterprise`, später mehrere `search_enterprise_*`. Das heißt: Das Modell ruft die Suche als Tool auf. Das kann sich seitdem geändert haben.
- **[Extern]:** Sicherheitsforscher beschreiben, dass Treffer als normaler Text im Modellkontext landen. Genau das machte Prompt-Injection wie EchoLeak möglich.
- **[Schluss]:** Die Rechte werden bei der Abfrage geprüft, nicht beim Einbetten. Der Index ist tenantweit, gezeigt wird aber nur, was der Nutzer öffnen darf.
- **[Offen]:** Reranker, Fusion von Stichwort und Vektor und die Trefferzahl in Copilot Chat.
- **[Gemessen] Wortwahl:** Dieselbe Sache in Copilot Chat: Mit einer Umschreibung nannte er die falsche Datei und „keine belastbare Angabe“. Mit dem Fachwort kam die richtige Antwort samt Zitat. Das sind zwei einzelne Fragen.

Es gibt zwei getrennte Lesearten:

- **Chunk-Suche:** Das ist der Normalfall, und das Modell sieht nur Ausschnitte.
- **Volltext-Lesen:** Bei deklarativen Agenten mit höchstens 20 ausdrücklich angegebenen Dateien durchsucht Copilot laut Doku den ganzen Inhalt.

Viele schlechte Antworten dürften aus der Chunk-Suche kommen: Die richtige Stelle wurde gar nicht gefunden [Schluss].

## 4. Die Sucharten im Vergleich

| Weg | Verfahren | Rückgabe | Voraussetzung |
|---|---|---|---|
| **Graph-Abruf/Delta** | Keine Suche, reine Aufzählung | Dateien und Daten | Graph-Rechte |
| **Microsoft Search** | Stichwörter und Personalisierung | Trefferliste | Keine Copilot-Lizenz |
| **Graph `/search/query`** | Stichwörter, KQL [Schluss]. Ganze Sätze: 0 von 10 [Gemessen] | Treffer mit Rang und Kurztext | Delegiert oder App-Rechte |
| **Copilot Search** | Semantisch und Stichwort | Treffer, teils KI-Antwort | Copilot-Lizenz |
| **Copilot Chat mit Arbeitsdaten** | Modell plus Such-Tools | Fertige Antwort | Copilot-Lizenz |
| **Retrieval API** | Hybrid laut Doku. Im Test ganze Sätze 0 von 10, Stichwörter 5 von 10 richtig [Gemessen] | Bis zu 25 Dateien mit Textstellen, ungeordnet | Copilot-Lizenz, sonst 0,10 $ pro Aufruf (Preview) |
| **Copilot Search API** | Hybrid, nur OneDrive, Beta | Dokumente | Copilot-Lizenz |
| **Work IQ `retrieve`** | Index, optional Connectoren (Preview) | Markdown und Treffer | Copilot Credits |
| **Work IQ `ask`** | Komplette Copilot-Antwort | Text | Copilot Credits |
| **Copilot Studio: SharePoint-URL** | GraphSearch, optional semantisch | Antwort des Agenten | Copilot Studio, für Semantik zusätzliche Kosten |
| **Copilot Studio: Upload aus SharePoint** | **Kopie nach Dataverse** mit eigener Indexierung | Antwort des Agenten | Dataverse-Speicher |
| **Foundry IQ, entfernte Quelle** | Ruft die Retrieval API. Noten bei gleicher Eingabe gleich [Gemessen] | Textstellen oder Antwort | Suchdienst und Copilot-Lizenz je Nutzer |
| **Foundry IQ, eigener Index** | Eigener Chunk- und Vektorindex in Azure AI Search. 6 bis 8 von 10 richtig [Gemessen] | Textstellen oder Antwort | Suchdienst, Embedding-Modell, Rechte selbst einrichten |
| **Eigene Auslese** (App „Projektkartei“) | Keine Suche: einmal alles lesen, dann abfragen | Vollständige Listen | Graph-Rechte, Tokens beim Einlesen |

Wichtig bei den Copilot-Studio-Zeilen: Der Upload nach Dataverse ist ein anderer Index als der Semantic Index. Zwei Agenten auf denselben Dateien können deshalb unterschiedlich antworten [Schluss].

Die Doku widerspricht sich auch: Laut einer Seite hat Microsoft Search keine semantische Suche, laut einer anderen verbessert der Semantic Index sie. Die Messung passt eher zur ersten Aussage: Die Graph-Suche fand ganze Sätze nicht.

**Weitere Grenzen der Retrieval API laut Doku:** Die Anfrage darf 1.500 Zeichen lang sein, 200 Anfragen pro Nutzer und Stunde, nur eine Datenquelle je Aufruf, Tabellentext nur bei `.doc`, `.docx` und `.pptx`. Falsche KQL-Filter laufen ohne Fehler und ohne Filter durch.

Mehr zu den Copilot-Studio-Wegen in [Copilot Studio](copilot-studio.md), zum eigenen Index in [Eigener Index mit Foundry IQ](eigener-index-foundry-iq.md), zu den Apps in der [Übersicht der Apps](../apps/README.md).

## 5. Was Work IQ wirklich ist

Der Name deckt mehrere Dinge ab:

- **In Copilot:** der Schalter, ob Antworten Arbeitsdaten nutzen. Das ist in der Premium-Lizenz enthalten.
- **Als API, Chat (`ask`, A2A, REST):** Copilot als entfernter Dienst.
- **Als API, Context (`retrieve`):** verwaltetes RAG.
- **Als API, Tools (`fetch`, `create_entity`, `do_action` und andere):** generische MCP-Tools als Schicht vor Graph, mit Richtlinien-Engine.
- **Als API, Workspaces:** Arbeitsspeicher in SharePoint Embedded.

Für Massenarbeit passt das schlecht: Sammlungen liefern standardmäßig 25, höchstens 100 Einträge, `$skip` ist blockiert, Dateien sind standardmäßig auf 4 MB begrenzt, und es gibt keine reine App-Anmeldung.

## 6. Was mit welcher Lizenz geht

Vor jeder Planung steht die Frage, welche Lizenzen vorhanden sind.

| Lizenz | Das geht |
|---|---|
| **Keine Copilot-Lizenz** | Microsoft Search, Graph, Copilot Chat mit Web und hochgeladenen Dateien |
| **Keine Copilot-Lizenz, aber Pay-as-you-go** | Zusätzlich Agenten auf Arbeitsdaten und Retrieval API für SharePoint |
| **Copilot Premium** | Chat mit Arbeitsdaten, Copilot Search, Retrieval API ohne Aufpreis |
| **Premium plus Credits** | Zusätzlich Cowork und Copilot Code |

## 7. So lässt sich die Blackbox messen

Da die Interna nicht offenliegen, bleibt nur das Messen von außen. Diese Messungen wurden geplant, und so weit sind sie gekommen:

| Messung | Vorgehen | Stand |
|---|---|---|
| **Latenz** | Datei mit Kunstwort hochladen, regelmäßig abfragen | Gemessen, siehe oben |
| **Stichwort oder Vektor** | Frage nur mit Synonymen stellen | Gemessen: 0 Treffer bei Graph-Suche und Retrieval API. Ein Copilot-Studio-Agent fand die Stelle, weil er die Suche selbst umformuliert, siehe [Copilot Studio](copilot-studio.md) |
| **Textstellen** | Länge der zurückgegebenen Stellen auswerten | Grob gemessen, siehe oben |
| **Abdeckung** | Fakten am Anfang, in der Mitte und am Ende langer Dateien | Teilweise: Stichwort bei rund 63 % der Datei gefunden. Nicht systematisch |
| **Formate** | Denselben Fakt in allen Formaten ablegen | Gemessen, siehe oben |
| **Rechte** | Zugriff entziehen und messen, wann der Treffer verschwindet | **Offen**, braucht ein zweites Konto |
| **Tools** | Netzwerkanfragen von Copilot im Browser ansehen | **Offen**. Das wäre nur ein Hinweis, die Formate können sich ändern |

So nutzt man am meisten von dem, was schon da ist: Index und Abruf von Microsoft übernehmen, messen, wo sie versagen, und nur diese Lücken selbst schließen. Die Lücken sind im Test bisher ganze Sätze über die API, Listen und Scans.

## Quellen

- [Semantischer Index](https://learn.microsoft.com/en-us/microsoftsearch/semantic-index-for-copilot)
- [Retrieval API](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/ai-services/retrieval/overview)
- [Retrieval API: Pay-as-you-go](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/ai-services/retrieval/paygo-retrieval)
- [Copilot Search API](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/ai-services/search/overview)
- [Copilot Search](https://learn.microsoft.com/en-us/microsoft-365/copilot/microsoft-365-copilot-search)
- [Copilot-Überblick und Lizenzen](https://learn.microsoft.com/en-us/copilot/microsoft-365/microsoft-365-copilot-architecture)
- [Nutzungsbasierte Abrechnung](https://learn.microsoft.com/en-us/microsoft-365/copilot/usage-based-billing-overview-copilot-credits)
- [Work IQ](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/)
- [Work IQ API](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/api-overview)
- [Work IQ MCP-Tools](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/mcp/tool-reference)
- [Work IQ `retrieve`](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/work-iq/mcp/retrieve)
- [Graph-Suche: Überblick](https://learn.microsoft.com/en-us/graph/search-concept-overview)
- [Graph-Suche: API](https://learn.microsoft.com/en-us/graph/api/search-query)
- [Copilot Connectors](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/overview-copilot-connector)
- [Optimize Content Retrieval](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/optimize-content-retrieval)
- [Copilot Studio: Wissensquellen](https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-copilot-studio)
- [Foundry IQ FAQ](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/foundry-iq-faq)
- [Copilot Studio: unstrukturierte Daten](https://learn.microsoft.com/en-us/microsoft-copilot-studio/knowledge-add-unstructured-data)
- [SharePoint-Online-Suche](https://techcommunity.microsoft.com/blog/spblog/introduction-to-sharepoint-online-search/170708)
- [Teams-Nachrichten und Substrate](https://practical365.com/how-advanced-ediscovery-efficiently-performs-compliance-searches-in-microsoft-teams/)
- [Zenity: Copilot-Systemprompt](https://labs.zenity.io/p/stealing-copilots-system-prompt)
- [Varonis: SearchLeak](https://www.varonis.com/blog/searchleak)
- [Bargury: Living off Microsoft Copilot](https://www.mbgsec.com/talks/2024-09-26-owasp-globalappsec-sf2024-living-off-microsoft-copilot/)
- [Dark Reading: EchoLeak](https://darkreading.com/application-security/researchers-detail-zero-click-copilot-exploit-echoleak)
