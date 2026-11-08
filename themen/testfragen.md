# Testfragen für die Copilot-Suche

Alle Firmen, Personen und Zahlen in den Testdaten sind erfunden. Die Testdaten bestehen aus einigen erfundenen Kundenprojekten mit Word-, Excel-, PowerPoint-, PDF-, BPMN-, Markdown- und Textdateien. Sie sind absichtlich so gebaut, dass typische Schwächen der Suche sichtbar werden: widersprüchliche Fassungen, Angaben tief in langen Dateien, Text nur im Bild oder im Scan, Sprechernotizen und Fragen über mehrere Projekte.

**Methodischer Hinweis:** Die Sollantworten dürfen nicht im Tenant liegen, sonst findet Copilot sie und das Ergebnis ist wertlos.

## Ablage im Tenant

| Testdatensatz | Ablage | Zweck |
|---|---|---|
| Projekt Velmora Logistik | Team „Projektarchiv Test“, Kanal „Velmora“ | Dateien aus einem Teams-Kanal |
| Stadtwerke Quarzbach, Tessaro Versicherung, Formatprobe | Dokumentbibliothek desselben Teams | Normale SharePoint-Ablage |
| Brunnhalde (vertraulich) | Eigene private Site, zwei Besitzer | Rechteprüfung |
| Latenzprobe | Erst hochladen, wenn die anderen liegen, Uhrzeit festhalten | Dauer bis zur Indexierung |

Laut Doku werden neue Dateien auf Sites mit mindestens zwei berechtigten Nutzern täglich indexiert. Fragen ab 24 Stunden nach dem Upload sind deshalb aussagekräftig, frühere Ergebnisse gelten nur als Zwischenstand.

Im Testtenant gab es nur eine Copilot-Lizenz. Das zweite Konto hatte keine Lizenz für SharePoint, Teams oder Office und meldete sich für die Tests nicht an. Mitgemessen wird, ob ein Konto ohne Lizenz als zweiter Berechtigter zählt.

In den Kanal „Velmora“ wird von Hand eine Nachricht gepostet, die inhaltlich zu Tessaro gehört. So wird mitgeprüft, ob die Suche Kanalnachrichten findet und richtig zuordnet:

> Kurze Info zu Tessaro: Der Kunde hat gestern die Pilotregion geändert. Statt Hessen läuft der Pilot jetzt in Thüringen.

## Die zehn Vergleichsfragen

| # | Frage | Sollantwort | Fundstelle | Was geprüft wird |
|---|---|---|---|---|
| 1 | Wie hoch ist das aktuelle genehmigte Budget im Projekt Velmora? | 189.000 € (CR-03) | Lenkungsausschuss-Folien, Projektplan (Excel) | Widerspruch: 148.000 € in der Fassung „final“, 172.500 € in „v2“ |
| 2 | Wann ging Velmora tatsächlich live? | 7. Juli 2025 | Abschlussbericht (PDF) | Plan gegen Ist über vier Terminstände |
| 3 | Wie lange darf das Zählerportal nach einem Ausfall höchstens nicht verfügbar sein? | 38 Minuten | Fachkonzept v3, etwa bei 63 % der Datei | Synonym statt „Wiederanlauffrist“, tiefe Stelle, Widerspruch zu v2 (120 Minuten) |
| 4 | Wie lange werden die Ablesedaten archiviert? | 11 Jahre | Fachkonzept v3, Anhang am Dateiende | Ende einer langen Datei, Widerspruch zu v2 (6 Jahre) |
| 5 | Welches Risiko hat bei Quarzbach den höchsten Score? | R-03, fehlende Barrierefreiheit, Score 16 | Risikoregister (Excel) | Auswertung einer Tabelle |
| 6 | Welche Sparte hat bei Tessaro die höchste Fehlerquote? | Haftpflicht, 6,8 % | Studie (PDF), Seite 5 | Tabelle in einem PDF |
| 7 | Ab welcher Schadenhöhe wird bei Tessaro direkt an den Regulierer eskaliert? | 7.500 € | Workshop-Folien, Folie 1 | Text nur im Bild |
| 8 | Baut Quarzbach auch eine App? | Nein, nur das Webportal | Lenkungsausschuss-Folien, Notiz auf Folie 2 | Sprechernotiz |
| 9 | Wann wurde das Zählerportal abgenommen, und wie viele Restmängel gab es? | 14. November 2025, 4 Restmängel | Abnahmeprotokoll als Scan (PDF) | Scan ohne Textebene |
| 10 | In welchen Projekten wurde die Lumeno Datenservice GmbH eingesetzt? | Velmora und Tessaro | Angebot (Word), Projektplan (Excel), Kickoff-Protokoll (Word) | Mehrere Projekte und Formate |

## Zusatzfragen

| # | Frage | Sollantwort | Was geprüft wird |
|---|---|---|---|
| Z1 | Was folgt im aktuellen Wareneingang bei Velmora auf die Qualitätsprüfung? | Entscheidung „Ware in Ordnung?“: bei ja einlagern, bei nein Reklamation an den Lieferanten | `.bpmn`-Datei als Quelle, die ältere Fassung kennt den Schritt nicht |
| Z2 | In welcher Region läuft der Tessaro-Pilot aktuell? | Thüringen, laut Kanalnachricht | Teams-Nachricht gegen Dokumente, die Hessen nennen |
| Z3 | Welches ERP-System empfiehlt Brunnhalde? | Korvantis ERP, nur mit Zugriff auf die Site | Rechte |
| Z4 | Was ist das Projektkennwort von Brunnhalde? | Morwenthal-Quelle, nur mit Zugriff auf die Site | Rechte, Kunstwort |
| Z5 | Was bedeutet Zyntharok? | Depot, Kühlkette reißt über acht Grad | Latenz: Zeitpunkt des ersten Treffers |
| Z6 | Bei welcher Temperatur wird die Kühlung im Depot kritisch? | Über acht Grad | Latenz der semantischen Suche, ohne das Kunstwort |

## Formatprobe

Jeweils dieselbe Frage: „Wie lautet der Wartungscode für die Pumpstation …?“ Jeder Code steckt in einem anderen Träger.

| Station | Code | Träger |
|---|---|---|
| Nord | ORKAVIL-1147 | Word, Fließtext |
| Süd | BELMIRAN-2283 | Word, Tabelle |
| Ost | TUVERANT-3391 | PDF, Fließtext |
| West | SOLVENIK-4406 | PDF, Tabelle |
| Hafen | QUINDARO-5512 | PDF, Scan |
| Mühle | FENORAK-6624 | Excel, erstes Blatt |
| Damm | GRISTOLUN-7735 | Excel, drittes Blatt nach langen Blättern |
| Brücke | HAVOLINE-8841 | PowerPoint, Folientext |
| Wehr | MIRAVELT-9952 | PowerPoint, Sprechernotiz |
| Turm | PELLANDRO-0958 | PowerPoint, Bild |
| Kanal | NOVARIK-1063 | Markdown |
| Schleuse | ZANTHEROS-1174 | Text |

## Rechteprüfung

Weil nur ein Konto eine Copilot-Lizenz hat, läuft die Prüfung über dessen Zugriff.

1. Das Konto mit Copilot-Lizenz legt die private Site an, lädt die Brunnhalde-Dateien hoch und trägt das zweite Konto als zweiten Besitzer ein. Sobald der Inhalt indexiert ist, werden Z3 und Z4 gestellt. Erwartung: Treffer. Bleibt er aus, zählt ein Konto ohne Lizenz vielleicht nicht als zweiter Berechtigter.
2. Das Konto mit Copilot-Lizenz entfernt sich im SharePoint Admin Center aus der Site, das zweite Konto bleibt Besitzer. Gemessen wird, wann der Treffer verschwindet. Erwartung: kein Treffer, auch nicht als globaler Admin.
3. Das Konto fügt sich wieder hinzu. Gemessen wird, wann der Treffer zurückkommt.

Ergebnis zu Schritt 1: Das Kunstwort der privaten Site tauchte 3,5 bis 4 Stunden nach dem Upload im Index auf. Ein Konto ohne Lizenz als zweiter Berechtigter hat die Indexierung also nicht verhindert (siehe [Testergebnisse](testergebnisse.md)).

## Ergebnisse im Copilot-Studio-Agenten

Bewertung: **R** richtig, **T** teilweise, **F** falsch, **N** nichts gefunden. Bei Widersprüchen wird auch festgehalten, ob die alte Fassung genannt oder der Widerspruch erwähnt wurde.

Ausgewertet wurde hier der Agent in Copilot Studio mit SharePoint-Link als Wissensquelle (Modell GPT-5 Chat, 6. November 2026). Die weiteren geplanten Varianten (Copilot-Chat, Agent Builder mit SharePoint oder Upload, Copilot Studio mit Upload) wurden in dieser Tabelle nicht ausgewertet.

| # | Lauf 1 | Anmerkung |
|---|---|---|
| 1 | R | 189.000 €, behauptet aber, es gebe keine abweichenden Fassungen |
| 2 | R | Nennt auch die Planstände 30.6. und 15.5. |
| 3 | R | 38 Minuten, nennt v2 mit 120 Minuten als ersetzt |
| 4 | N | Stelle am Dateiende nicht gefunden, zitiert fremde Abschlussberichte |
| 5 | R | |
| 6 | R | |
| 7 | N | Text im Bild nicht gefunden, sagt das klar |
| 8 | R | Sprechernotiz gefunden |
| 9 | T | Nur „November“ aus dem Plan, Scan nicht lesbar, sagt das klar |
| 10 | T | Nur Velmora, behauptet „in keinem anderen Projekt“ |

Zusatzfragen im selben Agenten: **Z2 F**, nennt Hessen und behauptet, es gebe keine widersprüchlichen Angaben. Die Teams-Nachricht ist keine Wissensquelle des Agenten. **Z3 R**, der Agent findet nichts, weil Brunnhalde nicht zu seinen Quellen gehört.

### Zweiter Lauf mit nachgeschärfter Anweisung

Die erste Anweisung des Agenten war ein Satz. Die zweite verlangt ausdrücklich, Fassungen zu vergleichen, nie Vollständigkeit zu behaupten und bei Listen mehrfach zu suchen. Ergebnis: **7 R, 2 T, 1 N** statt 6 R, 2 T, 2 N.

| # | Lauf 1 | Lauf 2 | Was sich geändert hat |
|---|---|---|---|
| 1 | R | R | Nennt jetzt 148.000 € und 172.500 € als ersetzte Fassungen |
| 4 | N | **R** | 11 Jahre, mit v2 (6 Jahre). Ein Lauf, also möglicherweise Zufall |
| 10 | T | T | Weiter nur Velmora, weiter „taucht nicht auf“, jetzt mit Hinweis auf Unvollständigkeit. Neu ist ein falscher Schluss: Lumeno sei „nicht fortgeführt“ worden, obwohl der Projektplan den Posten im Endbudget führt |
| Z2 | F | F | Weiter Hessen und „keine abweichenden Regionen“, jetzt mit Hinweis auf mögliche Teams-Nachrichten |

Die übrigen Antworten blieben richtig. Nebenwirkung: Fast jede Antwort endet jetzt mit einem Satz zur Unvollständigkeit, auch bei einfachen Einzelwerten.

Ein dritter Lauf mit derselben Anweisung bestätigte 7 R, 2 T, 1 N. Mit GPT-5 Reasoning statt GPT-5 Chat: **9 R, 1 N**, auch Frage 9 (Scan) und 10 (Tessaro) richtig, Z2 weiter falsch. Einzelheiten in [Copilot Studio im Test](copilot-studio.md).
