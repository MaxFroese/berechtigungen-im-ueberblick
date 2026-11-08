# Eigener Index mit Foundry IQ

Stand: 5. November 2026, Messung 16:08 bis 16:41 UTC in einem Testtenant. Alle Daten sind erfunden. Alle genutzten Features sind Preview. Die Grundlagen zur Retrieval API stehen in [Index und Retrieval](index-und-retrieval.md).

**Kurz gesagt:** Ja, ein eigener Index findet ganze Sätze. Bei zehn Vergleichsfragen stehen 6 bis 8 richtige Antworten gegen 0 bei der Retrieval API mit denselben Sätzen und 5 bei von Hand gewählten Stichwörtern. Synonyme sind aber nicht automatisch gelöst: Die Kernfrage nach den 38 Minuten beantwortet der Index ab Werk falsch mit dem alten Wert, erst eine gelockerte Reranker-Schwelle löst das. Die kostenlose Stufe hat entgegen der Doku gereicht. Die entfernte SharePoint-Quelle verhält sich wie die direkte API. Der Dokumentenfilter für Rechte griff im begrenzten Test, mit einem echten zweiten Nutzer wurde er aber nicht geprüft.

## Die Kette: Wo eine Antwort verloren gehen kann

Ein Index ist eine Kette aus Schritten. An jeder Stelle kann die richtige Antwort verloren gehen, und am Ende klingt sie trotzdem sicher. Als Beispiel dient ein Satz aus einem erfundenen Fachkonzept (`Fachkonzept_Zaehlerportal_v3.docx`): „Für das Portal gilt eine Wiederanlauffrist von 38 Minuten.“ Eine ältere Fassung (`v2_ALT`) nennt noch 120 Minuten.

1. **Dokument → Text.** Der Indexer holt die Datei über Microsoft Graph und zieht den Text heraus. Das Fachkonzept wird zu rund 168.000 Zeichen Text, ein gescanntes PDF fast leer, weil dort der Text nur ein Bild ist. Was hier fehlt, fehlt für immer. Mit eingeschalteter Bildbeschreibung (`disableImageVerbalization: false`) beschreibt ein Modell die Bilder, und der Scan wurde auffindbar. Die Beschreibung ist englisch, der zitierte Text bleibt deutsch.
2. **Text → Chunks.** Der Text wird in Stücke von 2.000 Zeichen geteilt, die sich um 200 Zeichen überlappen. Aus 28 Dateien wurden 137 Chunks. Der 38-Minuten-Satz liegt mitten im langen Fachkonzept, umgeben von Fülltext. In der kurzen alten Fassung steht die alte Angabe fast allein im Chunk. Genau das kippt später die Rangfolge. Die Überlappung kostet rund 11 Prozent mehr Tokens beim Einbetten.
3. **Chunk → Embedding.** Ein Embedding-Modell macht aus jedem Chunk 1.536 Zahlen. Ähnliche Bedeutung ergibt nahe Zahlen, auch ohne gemeinsames Wort. So findet der Index „Bei welcher Temperatur wird die Kühlung im Lager kritisch?“ in „Die Kühlkette im Depot Zyntharok reißt bei über acht Grad“, die Retrieval API nicht. Das Einbetten kostet fast nichts: 97.706 Tokens rund 0,2 Cent.
4. **Frage → Kandidaten.** Die Frage läuft hybrid, als Stichwort- und Vektorsuche, und wird zusammengeführt. Die 38-Minuten-Stelle lag bei reiner Stichwortsuche auf Platz 12, bei reiner Vektorsuche auf Platz 10, hybrid auf Platz 5, hybrid mit Reranker auf Platz 4. Die Kombination bringt hier also wirklich etwas.
5. **Rerank und Schwelle.** Ein stärkeres Modell bewertet die besten 50 Kandidaten mit Noten von 0 bis 4, Foundry IQ wirft alles unter einer Schwelle weg. Die alte Stelle mit 120 Minuten bekam 2,33, die richtige mit 38 Minuten nur 1,60. Die Schwelle lag gemessen bei rund 1,9. Die richtige Stelle flog raus, und die Antwort lautete im selben sicheren Ton „höchstens 120 Minuten“.
6. **Planung.** Bei `low` und `medium` zerlegt ein Modell die Frage erst in Teilfragen. Aus der Ausfall-Frage wurde etwa „Zählerportal SLA Verfügbarkeit“. Das klingt klug, verfehlt aber das Wort „Wiederanlauffrist“, und auch die umformulierte Frage trifft am Ende auf dieselbe Schwelle.
7. **Kontext → Antwort.** Fand die Suche beide Fassungen, antwortete das Modell richtig und meldete den Widerspruch. Das Modell ist also gut, wenn es die Stelle sieht. Die Fehler entstehen vorher.

**Merksatz:** Bei einer falschen Antwort zuerst die gelieferten Stellen prüfen, nicht das Modell. Deshalb trennt die Auswertung „Stelle gefunden“ und „Note der Antwort“.

## Der Testaufbau

- **Index:** Azure AI Search, Stufe Free, Region France Central, API `2026-08-01-preview`. Embeddings mit `text-embedding-3-small`, Planung, Bildbeschreibung und Antworten mit `gpt-5.4-mini`.
- **Inhalt:** Das Test-Projektarchiv ohne Archivordner (28 Dateien, 137 Chunks), aus dem Archiv nur Excel, PowerPoint und PDF (60 Dateien, 61 Chunks) und eine vertrauliche Site (2 Dateien). Nicht im Index: eine Site mit Prozessdaten und alle Word-Dateien im Archiv, etwa Fachkonzepte und Angebote. Der Grund ist das Tokenbudget, siehe „Grenzen“.
- **Gleichzeitig:** Retrieval API und Graph-Suche liefen mit denselben Fragen.
- **Bewertung:** Dasselbe Modell und dieselbe Kontextgrenze (8.000 Zeichen) für alle Wege. R richtig, T teilweise, F falsch, N nichts gefunden. Automatisch per Regex mit 16 Tests bewertet, die Antworten des Hauptlaufs zusätzlich von Hand gelesen, vom Lauf mit der entfernten Quelle eine Stichprobe.

## Vergleich je Weg: die zehn Fragen

| Weg | 1 Budget | 2 Live | 3 Ausfall | 4 Archiv | 5 Risiko | 6 Sparte | 7 Bild | 8 Notiz | 9 Scan | 10 Lumeno | R | T | F | N | Median |
|---|---|---|---|---|---|---|---|---|---|---|---:|---:|---:|---:|---:|
| Retrieval API, Satz | N | N | N | N | N | N | N | N | N | N | 0 | 0 | 0 | 10 | 0,5 s |
| Retrieval API, Stichwort | R | R | **F** | N | R | N | N | R | N | R | 5 | 0 | 1 | 4 | 0,5 s |
| Graph-Suche, Satz | N | N | N | N | N | N | N | N | N | N | 0 | 0 | 0 | 10 | 0,4 s |
| Graph-Suche, Stichwort | R | R | **F** | N | N | N | N | R | N | R | 4 | 0 | 1 | 5 | 0,5 s |
| Index, `minimal` | R | R | **F** | R | R | R | N | R | N | T | 6 | 1 | 1 | 2 | 0,8 s |
| Index, `low` | R | R | N | R | R | R | N | R | N | T | 6 | 1 | 0 | 3 | 3,3 s |
| Index, `medium` | R | R | **F** | R | R | R | N | R | N | R | 7 | 0 | 1 | 2 | 5,3 s |
| Index, `minimal`, Schwelle 0 | R | R | R | R | R | R | N | R | N | R | 8 | 0 | 0 | 2 | 0,8 s |

**Einordnung:**
- **Fairness:** Die Stichwörter waren vorab festgelegt. Mit anderen Wörtern ginge mehr. „Ablesedaten Archivierung“ traf nichts, weil die Datei „archiviert“ schreibt. Das zeigt zugleich die Schwäche des Wegs.
- **Frage 3 ist auf vier Wegen falsch, und alle klingen sicher.** Ursache ist die Reranker-Schwelle, siehe oben. In drei Wiederholungen blieb das gleich.
- **Schwelle 0 ist kein Freifahrtschein.** Sie liefert 33 bis 58 Stücke statt 0 bis 13. Das kostet Kontext, und bei großen Beständen wächst der Lärm. Der Vorsprung gegenüber `minimal` beruht hier auf nur zwei Fragen. Der sinnvolle Wert liegt irgendwo zwischen 0 und 1,9 und muss an echten Fragen eingestellt werden.
- **Veraltete Werte:** Bei Frage 4 meldet der Index den Widerspruch in allen vier Varianten und nennt v3 als aktueller. Das leistet das Modell, sobald beide Stellen vorliegen.
- **Fragen 7 und 9** stehen nur in Bildern. Das N zeigt also erwartetes Verhalten, siehe „Formate“.

### Sonden

| Sonde | Retrieval API | Graph-Suche | Index `minimal` | Index `low` |
|---|---|---|---|---|
| 12 Kunstwörter ohne Scan und Bild | alle gefunden | alle gefunden, nach Datei gezählt | alle gefunden | nicht gemessen |
| Synonym Kühlung und Temperatur | 0 Treffer | 0 Treffer | **gefunden** | **gefunden** |
| Satz Budget Velmora | 0 Treffer | 0 Treffer | **gefunden** | **gefunden** |
| Satz Zählerportal, 38 Minuten | 0 Treffer | 0 Treffer | nur alter Wert | nur alter Wert |
| „Reklamation an Lieferant“ im BPMN | gefunden | gefunden | **0 Treffer** | gefunden |
| Scan, Text im Bild | 0 Treffer | 0 Treffer | 0 Treffer | nicht gemessen |

`relevanceScore` der Retrieval API war in allen 20 Treffern wieder genau 0. Im Index sind die Werte echte Reranker-Noten von 0,9 bis 3,2.

## Was die Planung bringt und kostet

| | `minimal` | `low` | `medium` |
|---|---:|---:|---:|
| Richtig von 10 | 6 | 6 | 7 |
| Median Antwortzeit | 0,8 s | 3,3 s | 5,3 s |
| Modell-Tokens je Frage, Mittel | 0 | 1.187 | 3.341, bei der Liste 10.636 |
| Kosten Modell je Frage | 0 | rund 0,12 US-Cent | rund 0,4 US-Cent |
| Abrechnungstokens Suchdienst je Frage | 27.687 | 71.364 | 116.594 |
| Eingabe | nur `intents` | Chatnachrichten | Chatnachrichten |

**Einordnung:** Bei diesen einfachen Fragen brachte die Planung kaum etwas. Der einzige Gewinn von `medium` ist Frage 10, und das Beispiel ist zu klein für eine Rangliste. Der Mehrwert dürfte eher bei Fragen mit mehreren Teilen und bei Listen liegen. `low` ist außerdem nicht stabil: Dieselbe Listenfrage lieferte einmal 49 Stücke und im nächsten Lauf nur 3.

## Listenfrage: Budget aller 24 Projekte

Gezählt wird, wie viele Projekte mit aktuellem Budget in den gelieferten Stellen stehen.

| Anfrage | Retrieval API | Graph-Suche | Index `minimal` | `minimal`, Schwelle 0 | `low` | `medium` |
|---|---:|---:|---:|---:|---:|---:|
| Wie hoch ist das Budget aller Projekte? | 0 | 0 | 14 | 17 | 23 | 21 |
| Budget aller Projekte | 0 | 0 | 21 | 24 | 21 | nicht gemessen |
| budget | 24 | 25 Dateien, nur Vorschau | 24 | 24 | 24 | nicht gemessen |

**Mit echter Modellantwort:**

| Weg | Stücke | Tokens | Aktuelle Budgets in der Antwort | Was das Modell dazu sagt |
|---|---:|---:|---:|---|
| Retrieval API, `budget` | 25 | 4.809 | 24 von 24, Summe stimmt mit 4.613.000 € | „kann vollständig sein“ |
| Index `minimal`, Schwelle 0, „Budget aller Projekte“ | 55 | 10.610 | 23 von 24, Ostrel fehlt, dazu Velmora | „kann vollständig sein“ |
| Index `low`, ganzer Satz | 3 | 704 | **0 von 24**, nur Velmora | **„kann vollständig sein“** |

**Einordnung:** Auch der Index löst Listen nicht. Die Trefferzahl schwankt mit der Wortwahl. Im schlechtesten Fall fand das Modell ein einziges Projekt und hielt die Liste trotzdem für möglicherweise vollständig, obwohl der Prompt ausdrücklich danach fragte. Das Ergebnis ist sogar zu günstig, weil die Angebote mit veralteten Werten im Index fehlten: 0 alte Werte sagt hier nichts. Für Listen bleibt eine einmalige vollständige Auslese der bessere Weg (App „Projektkartei“, siehe [Übersicht der Apps](../apps/README.md)).

## Formate

Frage jeweils: „Wie lautet der Wartungscode für die Pumpstation …?“

| Stelle | Retrieval API | Index `minimal` | Index mit Bildbeschreibung |
|---|---|---|---|
| Word, PDF, Excel, PowerPoint samt Notiz, Markdown, Text (10 Stellen) | 0 von 10, ganze Frage | **10 von 10** | nicht einzeln gezählt |
| PDF als Scan | Nein | Nein | **Ja** |
| PowerPoint, Text im Bild | Nein | Nein | **Ja** |
| Frage 7, Eskalation 7.500 € im Bild | Nein | Nein | **Ja, Stelle gefunden** |

Die Bildbeschreibung lief nur auf zwei Testbeständen. Sie kostet wenig: rund 7.100 Tokens bei 14 Dateien, geschätzt als Differenz aus dem Azure-Monitor-Wert. Beim Lauf warnt der Indexer vor Zusatzkosten für die Bildextraktion, siehe „Kosten“. Eine Modellantwort oder der Scan aus Frage 9 wurden mit dieser Quelle nicht gemessen.

## Rechte

Mit der Option `ingestionPermissionOptions` schreibt der Indexer zu jedem Chunk, welche Konten und Gruppen ihn sehen dürfen. Bei der Abfrage sagt ein Nutzertoken, wer fragt, und der Dienst filtert.

| Aufruf | Quelle mit `ingestionPermissionOptions` | Dieselbe Site ohne diese Option |
|---|---|---|
| **Ohne Nutzertoken** | 200, **0 Treffer, kein Hinweis** | 200, **Inhalt vollständig sichtbar**, Kennwort lesbar |
| Token eines berechtigten Testnutzers | Treffer mit `Korvantis` und `Morwenthal` | Abruf wie ohne Token. Direktsuche: 400 |
| Kein gültiges Token | 401 | 401 |
| Token für Graph statt Search | 401 `InvalidAudience` | nicht gemessen |
| App-Token ohne Nutzer | Abruf 206 und 0 Treffer, Direktsuche 400 | nicht gemessen |

Alle Aufrufe liefen mit dem Admin-Schlüssel des Suchdiensts. Er umgeht den Filter also nicht, nur „Elevated Read“ mit Entra-Anmeldung und passender Rolle tut das.

**Wichtig ist die rechte Spalte:** Wird die Option beim Anlegen vergessen, liefert jeder Aufruf mit Dienstzugang vertrauliche Inhalte, egal wer fragt. Nachrüsten geht laut Doku nicht, die Quelle muss neu angelegt werden. Bei der Retrieval API prüft SharePoint selbst, dort gibt es diese Falle nicht.

**Prüfbar ohne zweites Konto?** Teilweise.
- **Geprüft:** Ohne Token kein Treffer. Ein berechtigter Nutzer bekommt Treffer. Falsche Token werden abgelehnt. Die gespeicherten Rechte lassen sich per Elevated Read einsehen: Jeder Chunk trägt ein Konto und vier Gruppen, darunter drei SharePoint-Gruppen, und der berechtigte Testnutzer gehört zur zugehörigen Gruppe.
- **Nicht geprüft:** Dass ein echter Nutzer ohne Recht bei gültigem Token nichts sieht. Das ist der entscheidende Fall, die Filterlogik selbst ist damit nicht belegt. Ein App-Token taugt nicht als Ersatz, weil der Dienst es ablehnt, statt zu filtern.
- **Hinweis:** Für den Index braucht der zweite Nutzer keine Copilot-Lizenz, nur ein Konto, das sich anmelden kann.

**Ebenfalls nicht geprüft:** Entzug von Rechten und wie schnell der Index nachzieht, weil an den Sites nichts geändert werden sollte. Laut Doku werden Änderungen an übergeordneten Ebenen nicht automatisch übernommen, dafür ist `resync` nötig. Der Indexer meldete dagegen „Folder propagation“ als aktiv. Was davon stimmt, ist ungetestet, und im Betrieb wäre das heikel.

**Widerspruch in der Doku:** Der Artikel zur Abfrage nennt Ergebnisse ohne Token bei indexierten Quellen ungefiltert, der Artikel zur Fehlersuche sagt, geschützter Inhalt kommt nicht zurück. Gemessen gilt der zweite Fall, aber nur, wenn die Quelle mit Rechten angelegt wurde.

## Entfernte Quelle gegen direkte API

Die Vermutung aus der Doku stimmt: Die entfernte SharePoint-Quelle verhält sich wie die Retrieval API.

| Weg | R | T | F | N | Median |
|---|---:|---:|---:|---:|---:|
| Direkte API, Satz | 0 | 0 | 0 | 10 | 0,5 s |
| Entfernte Quelle `minimal`, Satz | 0 | 0 | 0 | 10 | 0,4 s |
| Direkte API, Stichwort | 5 | 0 | 1 | 4 | 0,6 s |
| Entfernte Quelle `minimal`, Stichwort | 5 | 0 | 1 | 4 | 0,5 s |
| Entfernte Quelle `low`, Satz | 3 | 1 | 0 | 6 | 2,3 s |

Die Noten sind bei gleicher Eingabe in allen zehn Fragen gleich. Ohne Nutzertoken antwortet die entfernte Quelle mit 502 und einer klaren Meldung. Das ist sicherer als der leere Treffer beim Index. Sie liefert zudem echte Reranker-Werte, auch wenn `relevanceScore` der direkten API 0 ist.

Die Planung hilft nur teilweise: Sie macht aus Sätzen Stichwörter, und das ergibt 3 richtige statt 0. Der Plan fiel aber teils englisch aus, etwa „Velmora live launch date“, und traf dann nichts. Das Umschreiben der Frage in Stichwörter (siehe [Index und Retrieval](index-und-retrieval.md), Abschnitt 5) hilft also, ersetzt aber keinen Vektor. **Warum Sätze über die Retrieval API nichts finden, bleibt offen.**

## Aufbauzeit

| Schritt | Dauer |
|---|---:|
| Suchdienst Free | 7 s |
| App-Registrierung mit Zustimmung und Freigabe auf zwei Sites | 27 s |
| Embedding-Deployment | 8 s |
| Quelle anlegen | 24 bis 29 s je Quelle |
| Indexieren, 28 Dateien, 137 Chunks | 13 s |
| Indexieren, 60 Dateien | 18 s |
| **Maschinenzeit zusammen** | **rund 2 Minuten** |
| **Vom ersten Befehl bis zur fertigen Wissensdatenbank, mit Fehlversuchen** | **19 Minuten** |

Dazu kommen Doku lesen und Skripte schreiben. Ein Aufbau von Hand im Portal wurde nicht gemessen. Neue Inhalte waren in unter zwei Minuten abfragbar, die Retrieval API braucht dafür bis zu mehrere Stunden (siehe [Index und Retrieval](index-und-retrieval.md)). Danach läuft der Indexer standardmäßig einmal täglich.

**Was an Infrastruktur nötig war:** ein Azure-Abo mit Suchdienst, ein Embedding-Deployment, eine Entra-App-Registrierung mit Admin-Zustimmung und `Sites.Selected`-Freigaben auf den Sites sowie ein App-Geheimnis oder eine Identität für den Indexer.

## Kosten

**Dieser Test:** Azure rund **0,20 US-Dollar** nach Listenpreis. Tokens gesamt **268.058** bei einem selbst gesetzten Budget von 300.000: 97.706 Embeddings und 170.352 `gpt-5.4-mini`, beides aus Azure Monitor. Dazu 3,8 Millionen Abrechnungstokens des Suchdiensts, die in der Freimenge von 50 Millionen je Monat liegen und keine Modell-Tokens sind. Die Rechnung war zum Messzeitpunkt noch nicht sichtbar.

**Im Betrieb,** Preise laut [Azure Retail API](https://prices.azure.com) vom 5.11.2026 und [Pay-as-you-go-Seite](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/ai-services/retrieval/paygo-retrieval), Beispiel 3.000 Abfragen je Monat:

| Posten | Eigener Index, Basic | Retrieval API |
|---|---:|---:|
| Grundpreis | 0,0907 €/h, rund **66 € je Monat** (0,101 $/h, rund 74 $) | 0, wenn Copilot-Lizenz vorhanden |
| Je Abfrage | `minimal` rund 0,06 US-Cent, `low` rund 0,28 US-Cent, abzüglich Freimenge von 50 Millionen Tokens | Lizenz: 0. Ohne Lizenz **0,10 $ je Aufruf** |
| 3.000 Abfragen | rund 75 bis 81 $, plus Antwortmodell | Lizenz: 0. Pay-as-you-go: 300 $, plus Antwortmodell |
| Indexieren | Text: 0,02 $ je Million Tokens, fast nichts | 0 |
| Betrieb | Dienst, App, Geheimnis oder Identität, Zeitplan, Rechte | keiner |
| Nötig | Azure-Abo, Admin-Zustimmung | Mindestens eine Copilot-Lizenz im Tenant, auch für Pay-as-you-go |

Grob ab **740 Aufrufen je Monat** ist Basic günstiger als Pay-as-you-go. Der Preis einer Copilot-Lizenz ist hier nicht verglichen. Eine dauerhafte Free-Stufe ist trotz dieses Tests nicht einzuplanen: Laut Doku kann sie bei Inaktivität gelöscht werden, hat 50 MB Speicher (im Test 5 MB belegt) und ist für diesen Indexer nicht vorgesehen.

**Offener Kostenpunkt:** Die Bildextraktion kostet laut Preisliste 1 $ je 1.000 Bilder. Bei den wenigen Bildern im Test wären das unter 1 Cent. Ob auf Free überhaupt etwas abgerechnet wird, war nicht zu sehen.

## Die Doku gegen die Messung

| Thema | Doku | Gemessen |
|---|---|---|
| SharePoint-Indexer | Basic oder höher | **Läuft auf Free** |
| Rechte aufnehmen | Kostenpflichtige Stufe | **Läuft auf Free** |
| Modell in der Wissensdatenbank | Managed Identity nötig | **Geht mit Schlüssel** |
| Reranker-Schwelle | Antwort schließt alles unter 2,5 aus | Gemessen kommt ab rund 1,9 durch |
| Vorab-Registrierung für den Indexer | Formular nötig | Lief ohne |
| `includeFolder` | Beispiel ohne `https://` | Ohne `https://` Fehler „Invalid URI“ |
| Delegierter Weg, Device Code | Security Defaults blockieren | Nicht versucht, App-Berechtigung reichte |
| Nur `Sites.Selected` ohne `Files.Read.All` | Für Rechte in der Tabelle mit `Files.Read.All` | **Reichte**, Rolle `read` auf zwei Sites |

Das alles kann Microsoft jederzeit schließen, auf diese Abweichungen sollte niemand bauen.

## Stolpersteine

- **Graph-Vorschau kürzt:** Die Vorschau der Graph-Suche kürzt Kunstwörter, eine Prüfung auf den Text meldet dann „nicht gefunden“. Gezählt werden muss die gefundene Datei. Bei 10 Sonden war das der Unterschied.
- **Bewertung zu streng:** „Das steht in den Textstellen nicht“ zählte zunächst als falsch statt als nichts gefunden. Das fiel erst beim Lesen der Antworten auf. Automatische Bewertung braucht Stichproben von Hand.
- **`minimal` will andere Eingabe:** Chatnachrichten ergeben dort 400, es braucht `intents`.
- **Wissensdatenbank nach Änderung:** Direkt nach einem Umbau antwortete der erste Abruf mit 400, der nächste ging. Die Ursache ist ungeklärt.
- **Tokenbudget:** 61 Prozent der Tokens gingen in Planung und Antworten, die Planung allein lag mit 80.000 weit über der Schätzung. Das Indexieren lässt sich nicht je Token stoppen, Azure Monitor meldet nur mit Verzögerung.
- **Drosselung der Retrieval API:** Ein Zähler je Prozess reicht nicht, wenn mehrere Skripte parallel laufen. Im Test waren es rund 130 Aufrufe in einer Stunde, alle Antworten waren 200.
- **Geheimnisse im Echo:** Die Antwort beim Anlegen der Quelle enthält App-Geheimnis und Modellschlüssel im Klartext. Wer die Antwort einfach loggt, hat sie im Log. Sie sollte geschwärzt gespeichert werden.

## Grenzen

- **Eine kleine, saubere Testmenge.** Die Dateien sind wenige und gleichförmig. Echte Bestände verhalten sich anders.
- **Der Index war kleiner als die Vergleichsmenge.** Die Retrieval API durchsucht alles, der Index nicht die rund 148.000 Tokens Word-Dateien im Archiv und nicht die Prozess-Site. Mit diesem Rauschen wären die Noten des Index eher schlechter.
- **Nur `contentExtractionMode: minimal`.** Der Modus `standard` mit Content Understanding kostet extra und blieb ungetestet.
- **Teams-Nachrichten** waren nicht im Index. Fragen, deren Antwort nur in einem Teams-Chat steht, lassen sich auf diesem Weg nicht beantworten.
- **Ein Antwortmodell:** Andere Modelle können bei Widersprüchen anders urteilen.
- **Einzellauf:** Nur Frage 3 ist wiederholt. `low` schwankt nachweislich.
- **Die kostenlose Stufe verhält sich großzügiger als beschrieben.** Das Verhalten auf Basic ist nicht gemessen.

## Vorläufiges Urteil

**Der eigene Index ist technisch klar besser bei Fragen in ganzen Sätzen, und das ist im Test der Hauptmangel der Retrieval API.** Das Argument hat Gewicht, weil die API hier 0 von 10 liefert. Der Befund gilt aber nur für diesen Tenant, solange der Grund offen ist.

| Situation | Empfehlung |
|---|---|
| Fester Bestand, Fragen in natürlicher Sprache, eigene App, jemand kümmert sich | **Eigener Index lohnt sich** |
| Scans und Bilder sind wichtig, ohne eigene Bildstrecke | **Eigener Index mit Bildbeschreibung lohnt sich** |
| Viele Nutzer ohne Copilot-Lizenz, ab rund 740 Aufrufen je Monat | Index rechnet sich gegen Pay-as-you-go |
| Nutzer tippen Stichwörter, oder Copilot ist schon da | Retrieval API, oder einfach Copilot |
| Listen und Summen über alle Projekte | Keins von beiden: einmalige vollständige Auslese |
| Kunden dürfen sich nicht sehen, Rechte sind kritisch | **Entfernte Quelle oder Retrieval API**, solange der Filter nicht mit echten Nutzern geprüft ist |

**Vor allem ohne interne IT: im Zweifel nicht.** Die Technik war in Minuten aufgebaut, sie ist also nicht das Problem. Das Risiko liegt im Betrieb: Eine vergessene Option beim Anlegen legt vertrauliche Inhalte offen, ohne dass es auffällt. Rechteänderungen ziehen nicht zuverlässig nach. Das Geheimnis läuft ab. Und die Reranker-Schwelle muss jemand verstehen, sonst kommt eine falsche Antwort im selben sicheren Ton. Ohne jemanden, der das dauerhaft verantwortet, ist ein Index für einen kleinen Bestand ohne heikle Rechte vertretbar, für Kundendaten nicht.

## Offene Punkte und lohnende Nachtests

1. **Echter Nutzer ohne Recht.** Solange das aussteht, bleibt die Rechtefrage offen.
2. **Voller Bestand mit allen Word-Dateien.** Das braucht mehr als 300.000 Tokens.
3. **Antwort über die Bildquelle** für Fragen 7 und 9, auch mit dem Scan aus Frage 9.
4. **Reranker-Schwelle zwischen 0 und 1,9** gegen mehr Fragen einstellen.
5. **Tatsächliche Kosten** in Azure Cost Management gegen die geschätzten 0,20 Dollar prüfen.
6. **Rechteentzug:** Wie schnell und ob überhaupt der Index nachzieht.

Alle angelegten Ressourcen (Suchdienst, Embedding-Deployment, App-Registrierung, Site-Freigaben, Rollen) wurden nach dem Test wieder gelöscht.

**Quellen:** [SharePoint indexiert](https://learn.microsoft.com/en-us/azure/search/agentic-knowledge-source-how-to-sharepoint-indexed), [SharePoint-Indexer](https://learn.microsoft.com/en-us/azure/search/search-how-to-index-sharepoint-online), [Rechte aufnehmen](https://learn.microsoft.com/en-us/azure/search/search-indexer-sharepoint-access-control-lists), [Abruf](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-retrieve), [Fehlersuche Rechte](https://learn.microsoft.com/en-us/azure/search/troubleshoot-sharepoint-query-permission-filtering), [Regionen](https://learn.microsoft.com/en-us/azure/search/search-region-support), [Limits](https://learn.microsoft.com/en-us/azure/search/search-limits-quotas-capacity), [Modelle](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-how-to-create-knowledge-base), [Agentic Retrieval](https://learn.microsoft.com/en-us/azure/search/agentic-retrieval-overview), [Semantic Ranker](https://learn.microsoft.com/en-us/azure/search/semantic-search-overview). Die Preisseite zeigte im Abruf keine Zahlen, deshalb stammen die Preise aus der Azure Retail API.
