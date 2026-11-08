# Index und Retrieval verstehen

Stand: 5. November 2026, letzte Messung 16:51 UTC. Zurück zur [Startseite](../README.md). Alle Testdaten sind erfunden.

**Kurz gesagt:** Eine KI kann nicht alle Dateien einer Organisation lesen, bevor sie antwortet. Deshalb gibt es einen Index, also ein vorbereitetes Verzeichnis, aus dem sie passende Textstellen holt. Im Testtenant funktioniert über die Retrieval API bisher nur die Stichwortsuche. Ganze Sätze finden nichts, obwohl die Doku sie empfiehlt. Der Grund ist noch offen.

## 1. Das Bild: Register und Bibliothekarin

Ein Vergleich mit einer Bibliothek hilft.

| | Stichwortsuche | Vektorsuche, auch „semantisch“ |
|---|---|---|
| **Vergleich** | Das Register hinten im Buch | Eine Bibliothekarin, die weiß, worum es geht |
| **Wie sie sucht** | Kommt genau dieses Wort vor? | Klingt diese Stelle inhaltlich ähnlich? |
| **Stark bei** | Namen, Kürzeln, Nummern wie `ORKAVIL-1147` | Umschreibungen und Synonymen |
| **Schwach bei** | „Wiederanlauffrist“, wenn nach „nicht verfügbar“ gefragt wird | Exakten Kennungen |
| **Fachwort** | lexikalisch, auch Keyword | Embedding, Kosinus-Ähnlichkeit |

Wer beides kombiniert, sucht **hybrid**. Das verspricht Microsoft für die Retrieval API.

**Ein Beispiel aus den Tests:** In einer Datei steht: „Die Kühlkette im Depot Zyntharok reißt bei über acht Grad.“

- Die Suche nach `Zyntharok` findet sie, das ist die Stichwortsuche.
- Die Frage „Bei welcher Temperatur wird die Kühlung im Lager kritisch?“ teilt fast kein Wort mit der Datei. Sie kann nur die Vektorsuche finden. Im Test lieferte sie **0 Treffer**.

## 2. Der Weg einer Datei in den Index

```text
Datei in SharePoint hochgeladen oder geändert
  → Warteschlange
  → Text und Metadaten werden herausgelesen
  ├→ Stichwortindex: welches Wort steht wo?
  └→ Text in Abschnitte teilen (Chunks) → in Zahlen umwandeln (Embeddings) → Vektorindex
```

Das ist eine Rekonstruktion aus der Doku, kein offizielles Diagramm. Die Belege stehen in der [Microsoft-RAG-Architektur](microsoft-rag-architektur.md), Abschnitt 2.

**Daraus folgen drei Dinge:**

1. **Der Index ist eine Kopie und läuft nach.** Direkt nach dem Upload findet die Suche die Datei nicht und meldet trotzdem keinen Fehler.
2. **Nur Text kommt hinein.** Die Retrieval API unterstützt laut Doku keine Inhalte aus Bildern und Diagrammen. Das deckt sich mit den Scans im Test.
3. **Nicht jedes Format bekommt Vektoren.** Semantisch und hybrid geht laut Doku nur bei `.doc`, `.docx`, `.pptx`, `.pdf`, `.aspx` und `.one`. **Excel, Markdown und Text bleiben reine Stichwortsuche.** Das ist wichtig, wenn Budgets in Excel stehen.

## 3. Der Weg einer Frage: RAG

RAG heißt: erst passende Textstellen holen, dann das Modell nur daraus antworten lassen.

```text
Frage: „Wie hoch ist das Budget bei Velmora?“
  → Retrieval API: höchstens 25 Dateien mit Textstellen
  → Rechtefilter: nur was dieser Nutzer öffnen darf
  → Modell bekommt Frage + Textstellen und schreibt die Antwort mit Quellen
```

**Die wichtigste Schwachstelle:** Das Modell sieht nur, was die Suche liefert. Fehlt die richtige Stelle, antwortet es aus dem Rest und klingt genauso sicher. Bei einer Liste über alle Projekte kann so still etwas fehlen.

**Rechte:** Der Index gilt für den ganzen Tenant, gefiltert wird bei der Abfrage. Vermutlich ist das so gebaut, belegt ist nur das Ergebnis: Gezeigt wird, was der Nutzer öffnen darf. Das löst keine zu breiten Freigaben. Was zu weit freigegeben ist, findet die KI auch. Genau dafür gibt es die App „Freigabe-Röntgen“, siehe [Übersicht der Apps](../apps/README.md).

### Wer bei Microsoft was ist

Viele Namen, aber im Kern dieselbe Schicht. Die [Microsoft-RAG-Architektur](microsoft-rag-architektur.md) erklärt das ausführlich, auch den Abfragepfad von Copilot und alle Sucharten.

```text
Copilot Chat     Copilot Search     Agenten     eigene App
      \                |              |          /
        Retrieval API, Work IQ, Graph-Suche
                       |
    Stichwortindex + Semantic Index (Vektoren)
                       |
          SharePoint, OneDrive, Mail, Teams
```

Das Bild ist vereinfacht und aus der Doku abgeleitet. Nicht jedes Produkt nutzt jeden Weg, und wie Copilot intern fragt, ist nur teilweise öffentlich.

| Name | Was es ist |
|---|---|
| **Microsoft Graph** | API und Datenmodell über M365 |
| **Semantic Index** | Der Vektorindex neben dem Stichwortindex |
| **Retrieval API** | RAG als Dienst: Frage rein, Textstellen raus |
| **Copilot Chat** | Modell plus Such-Werkzeuge, liefert die fertige Antwort |
| **Copilot Search** | Die Suchseite mit Trefferliste, teils mit KI-Antwort |
| **Work IQ** | Sammelbegriff, unter anderem Schalter in Copilot und APIs für Antworten und Abruf |
| **Foundry IQ** | Eigene Wissensdatenbank auf Azure AI Search, außerhalb dieser Schicht |

## 4. Was gemessen wurde

Automatische Abfragen alle 30 Minuten, seit dem Abend des 4.11.

### Doku gegen Messung

| Thema | Microsoft sagt | Gemessen | Urteil |
|---|---|---|---|
| **Ganze Fragen** | Natürliche Sprache wird unterstützt, die Anfrage soll ein einzelner Satz sein | 0 Treffer, rund 18,5 Stunden nach dem Upload | **Widerspruch, offen** |
| **Stichwörter** | Hybrid enthält auch die Stichwortsuche | Finden zuverlässig. „Budget Projekt Velmora“ trifft, „Budget im Projekt Velmora“ nicht. Offenbar müssen alle Wörter vorkommen | Passt, aber streng |
| **Ähnlichkeitswert** | `relevanceScore` ist die Kosinus-Ähnlichkeit | In allen 23 Läufen der Sonden immer genau 0,0 | **Spricht gegen eine aktive Vektorsuche im Test** |
| **Scans und Text in Bildern** | Nicht unterstützt | Nie gefunden, auch nicht von der Graph-Suche | Passt |
| **Tabellen** | Nur bei `.doc`, `.docx` und `.pptx` | Kunstwort in einer PDF-Tabelle wurde trotzdem gefunden | Für Stichwörter besser als beschrieben |
| **Neue Dateien** | Bei Sites mit mindestens zwei Berechtigten täglich | Stichwörter meist schneller, siehe unten | Besser als beschrieben |

### Was je Format gefunden wurde

Gesucht wurde ein Kunstwort, das nur an dieser einen Stelle steht.

| Stelle | Gefunden? |
|---|---|
| Word: Fließtext und Tabelle | Ja |
| PDF: Fließtext und Tabelle | Ja |
| Excel: erstes und drittes Blatt | Ja |
| PowerPoint: Folientext und Sprechernotiz | Ja |
| Markdown und Text | Ja |
| PDF als Scan | **Nein** |
| PowerPoint: Text nur im Bild | **Nein** |

Ein Agent in Copilot Studio fand am 6.11. genau dieselben 10 von 12 Stellen, siehe [Copilot Studio](copilot-studio.md). Er nutzt also denselben Index. Mit GPT-5 Reasoning las derselbe Agent bei den Vergleichsfragen allerdings das gescannte Abnahmeprotokoll, warum, ist offen.

Die App „Prozessatlas“ las Scans und Bilder dagegen in 4 von 4 Fällen, weil dort ein Modell das Bild selbst liest (siehe [Übersicht der Apps](../apps/README.md)). Wer das braucht, muss es also selbst bauen.

### Wie lange es dauerte

| Inhalt | Upload (UTC) | Erstmals gefunden | Dauer |
|---|---|---|---|
| Latenzprobe, Team-Site | 4.11., 22:20 | 22:55 | bis 35 Minuten |
| Massendaten, Team-Site | 4.11., 22:30 | Bei der Nachmessung um 23:17 | bis 47 Minuten |
| Private Site | 4.11., 22:20 | 5.11., 02:31 | rund 4 Stunden |
| Prozessdaten, eigene Site | 5.11., 10:20 | Zwischen 11:47 und 12:17 | 1,5 bis 2 Stunden |
| Freigabe-Labor, 38 Dateien | 5.11., 11:21 | Graph-Suche alle nach 29 Minuten, Retrieval API nach 53 Minuten noch 0 | Über 53 Minuten |

**Merksatz:** Es gibt keine feste Wartezeit und keine Fortschrittsanzeige. Beide Suchwege können auch auseinanderlaufen, wie das Labor zeigt.

### Vollständigkeit

Eine Listenfrage hängt an der Wortwahl und am Limit von 25 Dateien.

| Anfrage | Projekte erreicht |
|---|---|
| `budget` | 24 von 24, knapp im Limit. Am 4.11. waren es noch 19 |
| `Festpreis` | 18 von 24 |
| `Wie hoch ist das Budget aller Projekte?` | 0 von 24, ohne Fehlermeldung |

Ein Copilot-Studio-Agent schaffte Listen nach einem Namen fast vollständig, etwa 7 von 7 Projekten mit Trevanto Systems. Listen nach einer Bedingung wie „Ist-Kosten über Plan“ scheiterten mit GPT-5 Chat (0 von 5), mit GPT-5 Reasoning gelangen sie (5 von 5). Details in [Copilot Studio](copilot-studio.md). Wie sich Listen vollständig beantworten lassen, zeigt die App „Projektkartei“ in der [Übersicht der Apps](../apps/README.md).

## 5. Warum finden ganze Sätze nichts?

Das ist die wichtigste offene Frage. Das sind Vermutungen, keine Befunde:

| Vermutung | Dafür spricht | Dagegen spricht | So lässt es sich prüfen |
|---|---|---|---|
| **Vektorindex wird noch gebaut** | Doku nennt „täglich“, es sind erst rund 18,5 Stunden, Werte stehen auf 0 | Stichwörter gehen schon | Sätze nach 24 und 48 Stunden prüfen |
| **Kleiner oder neuer Tenant wird anders behandelt** | Nur eine Copilot-Lizenz, Testabo | Nichts dazu in der Doku gefunden | Vergleich mit einem produktiv genutzten Tenant |
| **Nur reine Stichwortsuche in diesem Zugang** | Werte immer 0, strenges Verhalten | Doku sagt ausdrücklich hybrid | Antwort von Microsoft, oder Test in deren Demo |

Ein [eigener Index](eigener-index-foundry-iq.md) fand ganze Sätze über dieselben Dateien. Am Inhalt liegt es also nicht, sondern am Weg über die Retrieval API.

Falls es sich nicht klärt, gibt es einen einfachen Ausweg: Ein günstiges Modell schreibt die Frage erst in Stichwörter um, also von „Wie hoch ist das aktuelle Budget im Projekt Velmora?“ zu „Budget Velmora“. Das umgeht die Hürde, nicht aber das Synonym-Problem. Gebaut oder gemessen ist das noch nicht.

## 6. Welcher Weg für welche Aufgabe

| Aufgabe | Weg | Warum |
|---|---|---|
| Einzelfrage ohne Bauen | Copilot Chat | Fertig, schreibt die Frage vermutlich selbst um. Bisher nur zwei Einzelfragen getestet, die Wortwahl machte den Unterschied |
| Fragen an ein Archiv ohne Code, auch mit Synonymen | Agent in Copilot Studio | Formuliert die Suche selbst um, fand die 38 Minuten. Scans und Bilder bleiben unsichtbar, siehe [Copilot Studio](copilot-studio.md) |
| Einzelfrage in eigener App, Begriffe bekannt | Retrieval API | Textstellen fertig, Rechte inklusive. Braucht Copilot-Lizenz oder Pay-as-you-go |
| Dateien finden, Treffer als Liste | Graph-Suche (`/search/query`) | Keine Copilot-Lizenz nötig, liefert aber keine fertigen Textstellen fürs Modell |
| Alle Projekte vollständig auswerten | Einmal auslesen, dann abfragen (App „Projektkartei“) | Suche garantiert nie Vollständigkeit |
| Bekannte Datei lesen | Graph direkt | Keine Suche, kein Warten auf den Index |
| Scans und Bilder | Eigener Schritt mit Bildmodell oder Document Intelligence | Der Index kann das nicht |
| Volle Kontrolle, ganze Sätze, Quellen außerhalb von M365 | Azure AI Search oder Foundry IQ | Eigener Index, eigene Regeln, aber auch eigener Betrieb und eigene Rechteprüfung |

## 7. Foundry IQ: eine eigene Wissensdatenbank

**Gemessen am 5.11.** Details im [eigenen Index mit Foundry IQ](eigener-index-foundry-iq.md). Alles Angelegte ist wieder gelöscht.

| Zehn Vergleichsfragen | Richtig | Teilweise | Falsch | Nichts |
|---|---:|---:|---:|---:|
| Retrieval API, ganzer Satz | 0 | 0 | 0 | 10 |
| Retrieval API, von Hand gewählte Stichwörter | 5 | 0 | 1 | 4 |
| Graph-Suche, ganzer Satz | 0 | 0 | 0 | 10 |
| Graph-Suche, von Hand gewählte Stichwörter | 4 | 0 | 1 | 5 |
| Eigener Index, ohne Modellplanung (`minimal`) | 6 | 1 | 1 | 2 |
| Eigener Index, mit Planung (`low`) | 6 | 1 | 0 | 3 |
| Eigener Index, mit Planung (`medium`) | 7 | 0 | 1 | 2 |
| Eigener Index, Reranker-Schwelle 0 | 8 | 0 | 0 | 2 |

- **Ganze Sätze:** Der eigene Index findet sie. Das spricht dafür, dass die Sätze nicht am Inhalt scheitern, sondern am Weg über die Retrieval API. Warum, bleibt offen.
- **Synonyme:** Nicht gelöst. Die Frage nach den 38 Minuten beantwortet der Index ab Werk falsch mit 120 Minuten, weil die richtige Stelle unter die Schwelle fällt.
- **Planung per Modell:** Brachte hier kaum etwas, kostete aber 3 bis 5 Sekunden.
- **Listen:** Nicht gelöst. Mit ganzem Satz erreichte der Index 14 bis 23 von 24 Projekten.
- **Entfernte Quelle:** Die Noten waren in allen 10 Fragen gleich wie bei der direkten API. Die Vermutung aus der Doku (siehe unten) ist also bestätigt.
- **Der gefährliche Fund:** Dieselbe Site ohne die Rechte-Option beim Anlegen war komplett lesbar, auch ein Kennwort darin, ohne jeden Hinweis. Mit Option lieferte ein Aufruf ohne Nutzertoken 0 Treffer. Ob ein echter Nutzer ohne Recht wirklich nichts sieht, ist nicht geprüft, dafür fehlte ein zweites Konto.

**Vorläufiges Urteil:** Lohnt sich bei Fragen in natürlicher Sprache oder Scans, wenn jemand den Index verantwortet. Ohne interne IT und für Kundendaten im Zweifel nicht.

### Was die Doku vor der Messung erwarten ließ

**Was es ist:** Eine Wissensdatenbank in Microsoft Foundry, die mehrere Quellen bündelt. Technisch steckt **Azure AI Search** dahinter, das Formular braucht deshalb einen Suchdienst. Die Suchmaschine heißt „agentic retrieval“: Ein Modell zerlegt die Frage in Teilfragen, wählt die Quellen und sucht bei Bedarf ein zweites Mal.

**Es gibt zwei ganz verschiedene Wege zu SharePoint:**

| | Entfernt (remote) | Indexiert |
|---|---|---|
| **Wie** | Fragt zur Laufzeit die **Retrieval API** | Kopiert die Inhalte in einen eigenen Index |
| **Hilft bei ganzen Sätzen?** | Vermutlich nicht, gleiche Grundlage | Das wäre der eigentliche Test |
| **Rechte** | Automatisch, jeder Nutzer braucht eine Copilot-Lizenz | Müssen beim Anlegen ausdrücklich eingerichtet und geprüft werden |
| **Aktualität** | Wie die Retrieval API | Nach dem Zeitplan des Indexers |
| **Aufwand** | Gering | Suchdienst, App-Registrierung, Embedding-Modell |
| **Status** | Preview | Preview |

**Der Preis der Kontrolle:** Beim eigenen Index gelten Rechte nur, wenn sie eingerichtet sind. Das [FAQ](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/foundry-iq-faq) warnt ausdrücklich davor, dass sie sonst nicht automatisch greifen. Bei zwei Kunden, die sich gegenseitig nicht sehen dürfen, wäre das der kritische Punkt.

**Kosten:** Die kostenlose Stufe reichte im Test, obwohl die Doku für den SharePoint-Indexer Basic verlangt. Das kann Microsoft jederzeit ändern. Dauerbetrieb auf Basic kostet rund 66 € im Monat, dazu kommen Modellkosten. Mit `gpt-5.4-mini` lief auch die Planung.

## 8. Copilot Chat und die Wortwahl

Als Gegenprobe ging dieselbe Synonym-Frage an Copilot Chat: „Wie lange darf das Zählerportal nach einem Ausfall höchstens nicht verfügbar sein?“ Richtig sind 38 Minuten. Der Wert 120 Minuten stammt aus einer alten Fassung.

Hätte Copilot richtig geantwortet, wäre das ein Hinweis auf einen Weg gewesen, der über die API bisher nicht offensteht. Ein Beweis für Vektoren wäre es nicht, weil Copilot die Frage auch selbst umschreiben kann.

**Ergebnis vom 5.11., 17 Uhr:** Copilot Chat antwortete, es gebe keine belastbare Höchstdauer, und nannte als Quelle `Fachkonzept_Rautal`, eine andere Datei. Die richtige Datei `Fachkonzept_Zaehlerportal_v3.docx` kam nicht vor. Copilot hat also nicht geraten, aber trotz vorhandener Antwort „nichts gefunden“ gemeldet. Mit dem Fachwort aus dem Dokument klappte es dagegen: „Was steht im Fachkonzept Zählerportal zur Wiederanlauffrist?“ ergab 38 Minuten mit Zitat, und Copilot nannte die 120 Minuten richtig als ältere Fassung.

Dieselbe Sache, andere Wörter, anderes Ergebnis. Das ist ein Hinweis, dass auch Copilot hier von den richtigen Wörtern abhängt, aber nur ein einzelner Vergleich. Wer nicht weiß, wie es im Dokument heißt, dem hilft das Fachwort nicht. Genau das soll die semantische Suche eigentlich lösen.

**Wie streng die Stichwortsuche ist,** zeigt ein einfacher Vergleich in einer eigenen App auf der Retrieval API: `budget velmora` trifft, `Budget im Projekt Velmora` nicht. Ein einziges kleines Wort entscheidet über Treffer oder keinen Treffer.

**Wartezeit selbst messen:** Eine Datei mit einem frei erfundenen Kunstwort in SharePoint ablegen und alle paar Minuten danach suchen. Das zeigt die Verzögerung des Index direkt.

## 9. Was das für kleine Organisationen bedeuten könnte

Das sind vorläufige Schlüsse aus einem kleinen Test mit erfundenen Daten.

- **Fragen in Sätzen bleiben ungeklärt.** Bevor eine eigene App auf die Retrieval API setzt, sollte dieser Punkt in einem produktiv genutzten Tenant geprüft sein.
- **Listen nicht über die Suche beantworten.** Das Limit von 25 und die Wortabhängigkeit bleiben, auch wenn einzelne Abfragen gut aussehen.
- **Ablage zählt.** Scans und Bilder sind für den Index unsichtbar, Excel bleibt bei Stichwörtern. Wichtiges gehört in Word oder PDF mit Textebene.
- **Eigener Index nur mit Grund.** Er bringt Kontrolle, verlagert aber die Rechteprüfung zur eigenen Organisation. Ohne interne IT wiegt das schwer.

## Offene Punkte

- Finden ganze Sätze über die Retrieval API nach einem oder zwei Tagen etwas? Gemessen wurde bis rund 18,5 Stunden nach dem Upload, bis dahin nicht.
- Copilot Chat systematisch gegen die API vergleichen, siehe [Testfragen](testfragen.md).
- Sieht ein Konto ohne Recht die Datei wirklich nicht? Mit einem zweiten Konto nicht getestet.
- Warum findet die Retrieval API Sätze nicht, ein eigener Index auf denselben Dateien aber schon?
- Trifft dieselbe Frage in Microsofts interaktiver Demo der Retrieval API? Falls ja, läge der Unterschied am Aufruf oder an den Rechten.

## Zum Weiterlesen

- **Vertiefung mit Belegen:** [Microsoft-RAG-Architektur](microsoft-rag-architektur.md): Begriffe, Index, Abfragepfad, alle Sucharten, Work IQ, Lizenzen und Quellen
- **Agent ohne Code:** [Copilot Studio](copilot-studio.md): Evaluation, Anweisung, Modellwahl, Kosten
- **Eigener Index:** [Eigener Index mit Foundry IQ](eigener-index-foundry-iq.md)
- **In der eigenen App:** [Grundlagen, Kapitel 4](grundlagen.md#4-index-und-rag-wie-eine-frage-zur-antwort-wird)
- **Microsoft:** [Retrieval API](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api/ai-services/retrieval/overview), [Semantischer Index](https://learn.microsoft.com/en-us/microsoftsearch/semantic-index-for-copilot), [Foundry IQ FAQ](https://learn.microsoft.com/en-us/azure/foundry/agents/concepts/foundry-iq-faq), [SharePoint als indexierte Quelle](https://learn.microsoft.com/en-us/azure/search/agentic-knowledge-source-how-to-sharepoint-indexed), [SharePoint als entfernte Quelle](https://learn.microsoft.com/en-us/azure/search/agentic-knowledge-source-how-to-sharepoint-remote)
