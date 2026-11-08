# Copilot Studio im Test

Stand: 6. November 2026. Zurück zur [Startseite](../README.md). Alle Testdaten sind erfunden.

**Kurz gesagt:** Ein Copilot-Studio-Agent mit SharePoint als Wissensquelle beantwortet Einzelfragen deutlich besser als die Retrieval API, auch die Synonym-Frage. Mit dem Standardmodell nutzt er denselben Index wie Copilot: Scans und Text in Bildern findet er nicht. Listen gelingen, wenn ein Name in jeder Datei steht, und scheitern, wenn er selbst filtern oder rechnen muss. Mit GPT-5 Reasoning löst er fast alles, sogar den Scan, antwortet aber meist auf Englisch. Die Anweisung verbessert, was er aus Gefundenem macht, nicht was er findet. Die eingebaute Bewertung von Studio ist zu großzügig, um sich auf sie zu verlassen.

Der Agent läuft auf dem Standard-Harness. Was das bedeutet und wie sich der neuere Microsoft Copilot Harness unterscheidet, steht in [Harnesses im Vergleich](harnesses-im-vergleich.md) und [Copilot Harness im Praxistest](copilot-harness-praxistest.md).

## 1. Aufbau des Tests

| | |
|---|---|
| **Agent** | „Projektwissen Agent“, per Power Platform CLI (`pac`) angelegt und gepflegt |
| **Wissen** | Eine SharePoint-Site mit einem erfundenen Projektarchiv, Websuche aus |
| **Modell** | GPT-5 Chat, zum Vergleich einmal GPT-5 Reasoning |
| **Prüfweg** | Evaluation in Studio: Testsatz als CSV hochladen, Lauf starten, Antworten auslesen |
| **Testsätze** | 12 Vergleichsfragen, 12 Fragen zur Formatprobe, 7 Listenfragen (siehe [Testfragen](testfragen.md)) |
| **Bewertung** | Von Hand gegen die Sollantwort: **R** richtig, **T** teilweise, **F** falsch, **N** nichts gefunden |

Ein Lauf mit 12 Fragen dauert rund 4 Minuten.

## 2. Vergleichsfragen

Die zehn Standardfragen plus Z2 (Wert steht nur in einer Teams-Nachricht) und Z3 (Ort, der in den Quellen nicht vorkommt).

| # | Was geprüft wird | Lauf 1, alte Anweisung | Lauf 2, neue Anweisung | Lauf 3, Wiederholung | Lauf 4, GPT-5 Reasoning |
|---|---|---|---|---|---|
| 1 | Widerspruch beim Budget | R, bestreitet aber andere Fassungen | R, nennt 148.000 und 172.500 € | R, ebenso | R, ebenso |
| 2 | Plan gegen Ist | R | R | R | R |
| 3 | Synonym statt „Wiederanlauffrist“ | **R**, mit Hinweis auf v2 | R | R | R |
| 4 | Ende einer langen Datei | N | **R** | **R** | R |
| 5 | Excel-Tabelle | R | R | R | R |
| 6 | Tabelle im PDF | R | R | R | R |
| 7 | Text nur im Bild | N | N | N | N |
| 8 | Sprechernotiz | R | R | R | R |
| 9 | Scan ohne Textebene | T, nur „November“ | T | T | **R**, 14. November, 4 Restmängel |
| 10 | Mehrere Projekte | T, nur Velmora | T | T | **R**, Velmora und Tessaro |
| Z2 | Teams-Nachricht | F, Hessen | F | F | F |
| Z3 | Nicht in den Quellen | R | R | R | R |
| | **Summe der zehn Fragen** | **6 R, 2 T, 2 N** | **7 R, 2 T, 1 N** | **7 R, 2 T, 1 N** | **9 R, 1 N** |

**Was auffällt:**

- **Die Synonym-Frage gelingt.** Copilot Chat und die Retrieval API scheiterten daran, siehe [Index und Retrieval](index-und-retrieval.md). Der Agent formuliert die Suche offenbar selbst um, bevor er den Index fragt.
- **Er hält sich an die Quellen.** Wo nichts steht, sagt er das. Erfunden hat er in keinem Lauf etwas.
- **GPT-5 Reasoning liest den Scan.** Mit GPT-5 Chat war die Datei in drei Läufen „ohne auslesbaren Text“, mit Reasoning nannte der Agent Datum und Zahl der Restmängel aus genau diesem Protokoll. Naheliegend ist, dass dieses Modell die Datei selbst öffnet, statt nur Textstellen aus dem Index zu nehmen. Den Text im Bild bei Frage 7 fand es trotzdem nicht. Bis auf Frage 3 und 6 antwortete es auf Englisch.
- **Bei Frage 10 und Z2 behauptet er Vollständigkeit.** „In keinem anderen Projekt“ und „keine abweichenden Regionen“ stimmen beide nicht. Die Tessaro-Datei findet er nie, und die Teams-Nachricht ist keine Wissensquelle.

## 3. Was die Anweisung bewirkt

Die erste Anweisung war ein Satz. Die zweite verlangt ausdrücklich, Fassungen zu vergleichen, nie Vollständigkeit zu behaupten und bei Listen mehrfach zu suchen.

| Wirkung | Beleg |
|---|---|
| **Besser bei dem, was er schon gefunden hat** | Frage 1 nennt jetzt die alten Angebotssummen |
| **Findet mehr, wenn er öfter sucht** | Frage 4 in zwei Läufen richtig statt nichts gefunden |
| **Hilft nicht gegen die Suche** | Frage 10 findet Tessaro weiterhin nicht |
| **Wird nur halb befolgt** | Er schreibt weiter „taucht nicht auf“, hängt aber einen Vorbehalt an |
| **Nebenwirkung** | Fast jede Antwort endet mit „Die Liste kann unvollständig sein“, auch bei einem einzelnen Wert |

Der dritte Lauf ergab dieselben Bewertungen wie der zweite. Die Verbesserung ist also kein Zufall.

## 4. Formatprobe

Gefragt wurde jeweils nach dem Wartungscode einer Pumpstation. Jeder Code steht nur an einer Stelle.

| Station | Format | Agent |
|---|---|---|
| Nord, Süd | Word, Fließtext und Tabelle | Gefunden |
| Ost, West | PDF, Fließtext und Tabelle | Gefunden |
| Mühle, Damm | Excel, erstes und drittes Blatt | Gefunden |
| Brücke, Wehr | PowerPoint, Folientext und Sprechernotiz | Gefunden |
| Kanal, Schleuse | Markdown und Text | Gefunden |
| Hafen | PDF als Scan | **Nicht gefunden** |
| Turm | PowerPoint, Text nur im Bild | **Nicht gefunden** |

Das sind **10 von 12**, genau dasselbe Muster wie bei der Messung des Index. Mit GPT-5 Chat liest der Agent also nicht selbst in die Dateien, sondern bekommt Textstellen aus dem gleichen Index wie Copilot. Die Formatprobe lief nicht mit GPT-5 Reasoning. Später las ein Agent auf dem Microsoft Copilot Harness den Scan „Hafen“ und den Text im Bild, siehe [Copilot Harness im Praxistest](copilot-harness-praxistest.md).

## 5. Listenfragen

Sieben Fragen über 24 erfundene Projekte, mit vorher festgelegten Sollwerten.

| Frage | Soll | GPT-5 Chat | GPT-5 Reasoning |
|---|---|---|---|
| Projekte mit Mesaro Data | 8 | T, 7 von 8 | T, 7 von 8, aber auf Englisch |
| Projekte von Pia Lorenzen | 5 | **R**, 5 von 5 | T, 4 von 5 |
| Wie viele mit Trevanto Systems | 7 | **R** | **R** |
| Höchstes aktuelles Budget | Zellerau, 268.000 € | **R** | F, nimmt Karvia und wertet abgeschlossene Projekte nicht als „aktuell“ |
| Ist-Kosten über Plan | 5 Projekte | **F**, prüft 2 und sagt „keins“ | **R**, alle 5 mit Beträgen |
| Go-live 2023 | 9 Projekte | **F**, 2 von 9 | T, 8 von 9 |
| Palvera: Leitung und Partner | Wieskamp, Kolbrand IT | R | R |

**Das Muster:**

- **Listen nach einem Namen klappen.** Steht „Mesaro Data“ oder „Pia Lorenzen“ in jeder passenden Datei, findet die Suche sie fast alle. Die Grenze von 25 Dateien greift hier noch nicht.
- **Listen nach einer Bedingung scheitern mit GPT-5 Chat.** Für „Ist über Plan“ müsste er jeden Abschlussbericht öffnen und rechnen. Er schaut zwei an und schließt daraus, es gebe keine Überschreitung.
- **GPT-5 Reasoning löst genau diese Fälle**, sucht also gründlicher. Dafür antwortet es trotz deutscher Anweisung oft auf Englisch und legt Begriffe eigenwillig aus.

Für eine verlässliche Liste über alle Projekte bleibt eine strukturierte Ablage besser: einmal auslesen, dann rechnen. Diesen Weg geht die Projektkartei, beschrieben in der [Übersicht der Apps](../apps/README.md).

## 6. Modellwahl

Zur Auswahl standen GPT-5 Chat, GPT-5 Auto, GPT-5 Reasoning, GPT-4.1 und zwei „GPT Chat Latest“. Claude Sonnet und Opus waren ausgegraut, weil Anthropic-Modelle im Tenant erst ein Admin freigeben muss.

| | GPT-5 Chat | GPT-5 Reasoning |
|---|---|---|
| Vergleichsfragen | 7 R, 2 T, 1 N | **9 R, 1 N**, liest auch den Scan |
| Listen nach Name | Sehr gut | Etwas schlechter |
| Listen nach Bedingung | Scheitert | Gut |
| Sprache | Deutsch | Oft Englisch |
| Status | Allgemein verfügbar | Preview |

GPT-5 Reasoning ist inhaltlich klar besser. Dagegen sprechen derzeit die englischen Antworten trotz deutscher Anweisung und der Preview-Status. Ein naheliegender nächster Versuch ist, die Sprache in der Anweisung schärfer zu fordern. Auswertungen über alle Projekte sollte man auch mit diesem Modell nicht dem Agenten überlassen.

## 7. Die eingebaute Bewertung

Studio bewertet jede Antwort selbst. Die Methoden taugen unterschiedlich gut:

| Methode | Was sie prüft | Erfahrung |
|---|---|---|
| **Answer quality** | Relevant, vollständig, durch Quellen belegt | Vergleicht nicht mit der Sollantwort. Gab Frage 10 und Z2 ein „Pass“, obwohl beide falsch waren, und wertete die richtige Enthaltung bei Z3 als „Fail“ |
| **Meaning match** | Bedeutung wie die Sollantwort, Schwelle 50 von 100 | Wertete „2 von 9 Projekten“ als bestanden. Bei englischen Antworten 0 Punkte, auch wenn sie richtig waren |
| **Keyword match** | Sollbegriff kommt vor | Bei allen 12 Wartungscodes „Invalid“. Vermutlich schreibt das Modell den Code mit einem geschützten Bindestrich (`ORKAVIL‑1147`), der nicht als Treffer zählt |

Die Bewertung von Hand bleibt also nötig. Hilfreich ist die Evaluation trotzdem: Sie stellt die Fragen jedes Mal gleich und speichert alle Antworten mit Quellen.

## 8. Kosten und Abrechnung

Abgerechnet wurde über einen **Pay-as-you-go-Plan** im Power Platform Admin Center, angelegt am 30. Oktober 2026:

| | |
|---|---|
| Umgebung | Die Standardumgebung des Testtenants, in der auch der Agent liegt |
| Produkte | Copilot Studio, Windows 365 for Agents |
| Abrechnung | Ein Azure-Abo (Testguthaben) mit eigener Ressourcengruppe für die Credits |

Die Nutzung des Agenten und vermutlich auch die Evaluationsläufe werden also über das Azure-Abo abgerechnet. Am 6. November um 14 Uhr zeigten Admin Center und Azure Cost Management noch 0 Credits und 0 € an. Beide buchen mit Verzögerung. Auf dem Standard-Harness kosten Testchat und Läufe auf dem Entwurf ohnehin nichts, gezählt wird erst nach dem Veröffentlichen (siehe [Harnesses im Vergleich](harnesses-im-vergleich.md)).

**Wichtig:** Ein solcher Plan bleibt an der Umgebung hängen. Endet ein Testguthaben, läuft die Agentennutzung auf das Abo weiter, bis der Plan von der Umgebung gelöst oder gelöscht ist.

Die Funktion *Apps (Preview)* in Studio lief dagegen nicht: Sie verlangt einen eigenen Credit-Topf, den der Tenant nicht hat.

## 9. Bedienung ohne Oberfläche

| Schritt | Per `pac` | Nur in der Oberfläche |
|---|---|---|
| Agent anlegen, ändern, veröffentlichen | `pac copilot push`, `publish` | |
| Testsatz anlegen | | CSV hochladen |
| Lauf starten | `pac copilot-studio run-maker-evaluation-test-set` | |
| Status und Bewertung abrufen | `get-maker-evaluation-test-run --json` | |
| Antworttexte lesen | | Ergebnisseite oder CSV-Export |

Stolpersteine:

- `pac` braucht eine passende .NET-Installation und findet sie unter macOS nicht immer von selbst (Umgebungsvariable `DOTNET_ROOT`).
- Ein Lauf braucht die Verbindungs-ID eines Benutzerprofils. Ohne Profil läuft er anonym und sieht SharePoint nicht.
- Ein Lauf auf der veröffentlichten Fassung brach mit HTTP 500 ab, auf dem Entwurf lief er.
- Die Antworttexte gibt die Schnittstelle nur mit eigenen Copilot-Studio-Rechten heraus, die die Azure CLI nicht hat.
- Nach rund einer Stunde Arbeit im Browser lud Studio nur noch weiße Seiten, die Konsole zeigte abgelehnte Anfragen.

## 10. Folgerungen für kleine Organisationen

- **Für Fragen an ein Projektarchiv ist ein Studio-Agent ein guter Start.** Kein Code, Rechte über die Anmeldung, und er beantwortet Fragen, an denen die rohe Suche scheitert.
- **Scans und Bilder brauchen einen Schritt davor**, etwa Texterkennung beim Ablegen. Sonst sind sie für jeden Agenten auf dem Standard-Harness unsichtbar.
- **Listen und Auswertungen gehören nicht in den Chat.** Das gilt für alle Wege auf dem Standard-Harness. Eine strukturierte Ablage ist verlässlicher.
- **Evaluation gehört zum Projekt.** Ein fester Testsatz zeigt, ob eine geänderte Anweisung hilft oder schadet. Die Bewertung muss aber jemand prüfen.
- **Abrechnung vorher klären.** Ein Pay-as-you-go-Plan rechnet still im Hintergrund ab.

## Offene Punkte

- Wie liest GPT-5 Reasoning den Scan? Die Aktivitätsansicht eines Laufs und eine Formatprobe mit diesem Modell könnten das zeigen.
- Antwortet GPT-5 Reasoning mit strengerer Sprachvorgabe auf Deutsch?
- Wie viele Credits verbraucht ein Lauf, sobald Admin Center oder Azure sie anzeigen?
- Wissen per Upload statt SharePoint-Link: Liest Dataverse Scans und Bilder?
- Kommt die Stärke bei Synonymen von der Umformulierung der Frage oder von einer anderen Suche?
