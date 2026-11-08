# Testergebnisse aus dem Testtenant

Getestet wurde in einem Testtenant mit ausschließlich erfundenen Daten, im Zeitraum bis 8. November 2026. Die Fragen und Sollantworten stehen in [Testfragen](testfragen.md), die ausführliche Auswertung der Suche in [Index und Retrieval](index-und-retrieval.md).

**Kurz gesagt:** Der Stichwortindex von Microsoft 365 war nach gut einer halben Stunde bereit, ganze Fragen lieferten über die Retrieval API aber auch nach rund 18,5 Stunden keine Textstellen. Scans, Text in Bildern und BPMN-Dateien wurden nicht gefunden. Ein Copilot-Studio-Agent mit SharePoint als Quelle beantwortete Einzelfragen gut, Listenfragen über viele Projekte bleiben auf allen Wegen unzuverlässig. `Sites.Selected` funktioniert wie dokumentiert.

## Welche Lizenzen für welche Tests nötig waren

| Lizenz oder Ressource | Wofür nötig |
|---|---|
| Microsoft 365 mit Copilot-Lizenz (im Test nur ein Platz) | Retrieval API, Copilot-Chat, SharePoint als Wissensquelle im Agent Builder. Rechtetests liefen deshalb über das eine lizenzierte Konto |
| Konto ohne Office-, SharePoint- oder Teams-Lizenz | Ließ sich per Graph trotzdem als Team-Mitglied und Gruppenbesitzer eintragen und zählte als zweiter Berechtigter für die Indexierung |
| Copilot Studio (Testversion) mit Dataverse-Umgebung | Agent anlegen, veröffentlichen, Evaluation. In der Testversion waren Claude-Modelle gesperrt, und „Apps (Preview)“ hatte keinen Credit-Topf |
| Pay-as-you-go-Plan in der Power Platform, abgerechnet über Azure | Copilot Credits für die Agent-Läufe |
| Power Automate Premium | Kurzzeitiger HTTP-Auslöser für den Testaufruf eines Flows (Premium-Connector) |
| Azure-Abonnement (Testguthaben) | Foundry mit Modell-Deployments (Sweden Central), App Service B1, Document Intelligence F0, Azure AI Search für den Foundry-IQ-Vergleich |

Ein Azure-Budget mit Warnschwellen warnt nur, es stoppt keine Kosten.

## Indexierung und Suche

Alle Dateien wurden am 4. November 2026 um 22:20 UTC hochgeladen, 24 weitere erfundene Projekte (213 Dateien) um 22:30 UTC. Gemessen wurde alle 30 Minuten über Graph-Suche und Retrieval API mit dem lizenzierten Konto.

| Sonde | Ergebnis |
|---|---|
| Kunstwörter in Word, PDF-Text, PDF-Tabelle, Excel (Blatt 1 und 3), Folientext, Sprechernotiz, Markdown, Text | Nach 35 Minuten gefunden, in Graph-Suche und Retrieval API |
| Massendaten (Stichwörter wie „Budget Velmora“) | Nach 47 Minuten im Stichwortindex |
| Code im Scan und Code im Bild | Nicht gefunden |
| Text aus `.bpmn` | Nicht gefunden |
| Kunstwort in der privaten Site | Nach 3,5 bis 4 Stunden gefunden |
| Ganze Fragen ohne Kunstwort | Auch nach rund 18,5 Stunden keine Textstellen |

**Erkenntnisse:**

- Der Stichwortweg war deutlich schneller bereit als das „täglich“ aus der Doku. Die Doku bezieht sich aber auf den semantischen Index.
- „Budget Projekt Velmora“ fand Dateien, „Budget im Projekt Velmora“ nicht, auch nicht auf Englisch, mit Pfadfilter oder über den Beta-Endpunkt. Offenbar müssen alle Wörter vorkommen. Die Ähnlichkeitswerte waren 0 oder fehlten, die Retrieval API arbeitete im Test also vermutlich als reine Stichwortsuche. Ob der semantische Teil später nachkommt oder im Tenant fehlte, ist in [Index und Retrieval](index-und-retrieval.md) weiterverfolgt.
- Ein Treffer auf ein exaktes Kunstwort zeigt nur, dass das Wort im Index ist, nicht dass eine Frage dazu richtig beantwortet wird.
- Ein Konto ohne Lizenz als zweiter Berechtigter hat die Indexierung der privaten Site nicht verhindert.
- Die regelmäßige Messung fiel einmal aus, weil ein neues Recht zu den angeforderten Standardrechten hinzugefügt wurde, dem noch nicht zugestimmt war. Danach scheitert die Token-Anfrage, bis jemand zustimmt.

**Laut Doku [Doku]:** Die [Retrieval API](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/ai-services/retrieval/overview) liest Tabellen nur aus `.doc`, `.docx` und `.pptx`, Bilder gar nicht, semantisch nur `.doc`, `.docx`, `.pptx`, `.pdf`, `.aspx` und `.one`. Limit: 200 Anfragen pro Nutzer und Stunde, höchstens 25 Ergebnisse pro Anfrage. Der [semantische Index](https://learn.microsoft.com/en-us/microsoftsearch/semantic-index-for-copilot) nennt Word, PowerPoint, PDF, ASPX und OneNote, aber weder Excel noch Markdown oder BPMN. Die Formatprobe zeigt, dass Excel und Markdown trotzdem gefunden werden.

## Übersicht der Tests

| Test | Datum | Ergebnis | Kosten | Nötige Rechte |
|---|---|---|---|---|
| Eigene Web-App (RAG über Retrieval API und Modell in Foundry) | 4. Nov. 2026 | „budget velmora“ ergab 189.000 € mit zwei Quellen, 501 Tokens. Als Teams-Tab im Browser getestet: nach einmaliger Zustimmung kein eigenes Login. Teams Desktop nicht getestet | App Service B1 läuft durchgehend, dazu Modell-Tokens | Delegiert `Files.Read.All`, `Sites.Read.All`, Copilot-Lizenz für die Retrieval API |
| `Sites.Selected` | 4. Nov. 2026 | Vor der Freigabe beide Sites 403, danach die freigegebene Site 200 und die vertrauliche weiter 403 | Keine | App-Recht `Sites.Selected`, zum Freigeben `Sites.FullControl.All` |
| Copilot-Studio-Agent per CLI | 4. Nov. 2026 | Angelegt, SharePoint-Wissen ergänzt, Websuche aus, veröffentlicht | Copilot Credits | Umgebung mit Dataverse, Anmeldung über die Power Platform CLI (`pac`) |
| Copilot-Studio-Agent, Evaluation | 6. Nov. 2026 | 10 Vergleichsfragen: 6 R, 2 T, 2 N. Synonym-Frage richtig, mit Hinweis auf die alte Fassung. Bild, Scan und Dateiende nicht gefunden. Listenfrage unvollständig, aber als vollständig behauptet. 4 Minuten für 12 Fragen | Copilot Credits | Testsatz per CSV in der Oberfläche, Lauf mit Benutzerprofil, sonst kein SharePoint-Zugriff |
| Copilot Studio, Anweisung v2 und weitere Läufe | 6. Nov. 2026 | 7 R, 2 T, 1 N, bei Wiederholung bestätigt. Formatprobe 10 von 12, nur Scan und Bild fehlen. Mit GPT-5 Reasoning 9 R, 1 N, liest sogar den Scan, antwortet aber meist auf Englisch | Copilot Credits über Azure abgerechnet | Lauf auf der veröffentlichten Fassung brach mit HTTP 500 ab, auf dem Entwurf lief er |
| Genehmigungs-Flow per API | 4. Nov. 2026 | Liste und Flow angelegt, zwei Läufe erfolgreich, Prüfregel korrekt (129,50 € automatisch, 612 € Genehmigung nötig) | Gering | Bestehende SharePoint-Verbindung, für den Testaufruf Power Automate Premium |
| Foundry IQ gegen Retrieval API | 5. Nov. 2026 | Siehe [Eigener Index mit Foundry IQ](eigener-index-foundry-iq.md) | 268.058 Modell-Tokens, laut Listenpreis rund 0,20 $ | Azure-Abonnement |

Noch nicht getestet: Skills für Folien und BPMN in Cowork.

Einzelbewertungen pro Frage stehen in [Testfragen](testfragen.md), die Auswertung des Agenten in [Copilot Studio im Test](copilot-studio.md).

## Listenfragen über viele Projekte

Auf „budget“ nannte die Web-App 19 Projekte. Alle 18 geprüften Werte aus den Massendaten waren richtig. Sechs Projekte fehlten aber, ohne Hinweis in der Antwort. In späteren Läufen tauchten diese sechs wieder auf, die Treffermenge schwankt also. Die feste Grenze bleibt: Die Retrieval API liefert höchstens 25 Dateien, bei „Festpreis“ kamen 18 von 24 Projekten. **Für Listen über alle Projekte reicht dieser Weg nicht.** Auch der Copilot-Studio-Agent behauptete bei einer Listenfrage Vollständigkeit, obwohl ein Projekt fehlte.

## Anmeldung und API-Zugriff

- **Device-Code-Anmeldung** wird von den Security Defaults blockiert (Fehler 530035). Die normale Browser-Anmeldung funktioniert, die Security Defaults können anbleiben.
- **Azure CLI** reicht für Azure und Entra, aber nicht für Dateien und Kanalnachrichten. Dafür braucht es eine eigene App-Registrierung mit delegierten Graph-Rechten.
- **Foundry** lässt sich mit Entra-Anmeldung statt API-Key aufrufen. Die Web-App war dafür mit einer Managed Identity eingerichtet, ohne Key und Secret im Code.
- **Graph-Suche:** Der Parameter `region` wird bei delegierten Rechten abgelehnt.
- **Server-Schutz der Web-App:** Anfragen ohne oder mit falschem Token wurden mit 401 abgewiesen. Geprüft werden Token, Tenant und Scope.
- **Teams-Tab:** Läuft über Nested App Authentication. Die Web-Version von Teams warnt beim Hochladen einer eigenen App, „Show the app anyway“ reicht.

## Power Automate und Copilot Studio

- Ein Flow mit Knopf-Auslöser ließ sich per API nicht mit Eingabewerten starten. Getestet wurde daher mit einem kurzzeitigen HTTP-Auslöser (Premium), danach wieder zurückgestellt.
- Eine bestehende Verbindung ließ sich wiederverwenden. Entgegen der Vermutung ließ sich auch eine neue Genehmigungsverbindung ohne Zustimmung im Browser anlegen.
- Ein Agent konnte per API eine Power-Automate-Genehmigung im Namen des angemeldeten Kontos beantworten. Wer delegierten Zugriff hat, kann also auch Genehmigungen erteilen.
- Die Evaluationsmethode „Answer quality“ in Copilot Studio prüft nicht gegen die Sollantwort. Sie gab einer falschen und einer unvollständigen Antwort ein Pass und wertete eine richtige Enthaltung als Fail. Für einen Vergleich muss „Compare meaning“ dazu.
- Per `pac` lassen sich Testsätze auflisten, Läufe starten und Status abrufen, die Antworttexte aber nicht. Die liefert nur der CSV-Export in der Oberfläche.

## Weitere Apps

Projektkartei, Beleg-Prüfer, Freigabe-Röntgen, Protokollpilot, Genehmigungsbot und Prozessatlas sind in [Apps im Überblick](../apps/README.md) beschrieben. Kurz:

- Die Apps sind als Demo im Browser geprüft, nicht auf Safari, echten Handys oder mit einem zweiten Konto.
- **Protokollpilot:** Die Anlage von Aufgaben in Planner ist real getestet, To Do und der Abruf eines echten Transkripts über die API liefen in einem Nachtest ebenfalls real.
- **Genehmigungsbot:** Liste, Webhook und abgewiesene Fälschungen real beobachtet. Der Durchlauf in Teams klappte: Karte nach Sekunden, Klick bis zum Eintrag in der Liste. Zwei echte Personen sind nicht getestet.
- **Prozessatlas:** Live-Weg mit echter Anmeldung und Foliensatz in PowerPoint sind ungeprüft.
- Ein App-Service-Plan B1 lag mit sieben kleinen Apps bei rund 80 % Speicher.

## Offene Punkte

- Ob der semantische Teil der Retrieval API im Tenant überhaupt aktiv war.
- Höhe der verbrauchten Copilot Credits pro Lauf.
- Teams Desktop und echte Mehrpersonen-Abläufe.
- Skills für Folien und BPMN in Cowork.
