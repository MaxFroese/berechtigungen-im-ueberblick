# Sieben Test-Apps: was sich mit Microsoft 365, Azure und einem Modell bauen lässt

Stand: November 2026. Alle Daten sind erfunden.

**Kurz gesagt:** Sieben kleine Apps probieren allgemeine KI-Anwendungsfälle aus dem Büroalltag aus. Was zuverlässig klappt, ist Struktur aus Dokumenten holen. Offen bleiben der Betrieb und der Test mit echten Daten.

## Worum es geht

Im Büroalltag gibt es viele Aufgaben, bei denen ein Sprachmodell helfen könnte: Wissen in vorhandenen Dokumenten finden, Abläufe aus verteilten Unterlagen ableiten, Belege prüfen, Freigaben kontrollieren, Aufgaben aus Besprechungen ziehen, Anträge genehmigen lassen. Jede App greift einen dieser allgemeinen Fälle auf und misst ihn in einem Testtenant mit erfundenen Daten.

**Wichtig zur Einordnung:** Die meisten Apps liefen nicht live gegen Microsoft 365. Die Daten wurden einmal erzeugt oder einmal aus dem Testtenant geladen und lokal ausgewertet. Die öffentliche Seite zeigte danach nur vorberechnete Ergebnisse, ohne Anmeldung, ohne Modellaufruf und ohne Schreibweg. Solche Apps sind eher ein Prototyp oder Fahrplan: Sie zeigen, was möglich wäre und wo es hakt, nicht, dass es im Betrieb schon läuft. Wirklich live mit Anmeldung, Graph und Modell liefen nur **Projektwissen** und der **Genehmigungsbot**, in Teilen auch der **Protokollpilot**.

| App | Fall | Live oder offline | Wichtigstes Ergebnis |
|---|---|---|---|
| [Projektwissen](#projektwissen) | Fragen an Projektdateien, auch als Teams-Tab | Live | Stichwörter gehen, ganze Sätze über die Retrieval API nicht |
| [Prozessatlas](#prozessatlas) | Prozessentwurf aus verteilten Unterlagen | Offline, Live-Weg gebaut, nicht geprüft | Findet fast alle Schritte und erfand im Lückentest keinen |
| [Projektkartei](#projektkartei) | Vollständige Listen über alle Projekte | Offline, Daten einmal per Graph geladen | 24 von 24 Karten, 168 von 168 Feldern, ein Härtefall still falsch |
| [Beleg-Prüfer](#beleg-prüfer) | Belege lesen, dann Regeln prüfen | Offline, Belege erzeugt | These weitgehend bestätigt, Eigenbau lohnt bei wenigen Vorgängen nicht |
| [Freigabe-Röntgen](#freigabe-röntgen) | Zu breite Freigaben finden | Scan lokal gegen den Tenant, Demo offline | 20 von 20 Blind-Befunden, aber nur 7 von 11 Risikostufen richtig |
| [Protokollpilot](#protokollpilot) | Aufgaben aus Meeting-Transkripten | Messung offline, Abruf und Anlage teils real | Rund 99 % der Aufgaben, falsche Sprecherzeile führt still zu falscher Zuweisung |
| [Genehmigungsbot](#genehmigungsbot) | Genehmigungskarte in Teams | Live | Echter Klick bis in die Liste, 91 Tests, zwei echte Personen ungetestet |

**Zu den Modellnamen:** `gpt-5.4-mini` ist das günstige Modell. Das stärkste getestete Modell ist `gpt-6.1-sol`. Weitere Modelle (`gpt-6.1-terra`, `gpt-6-luna`, `gpt-6-astra`) tauchen nur in einzelnen Vergleichen auf.

## Was eine App mit Live-Daten aus Microsoft 365 braucht

Eine Seite, die nur vorberechnete Daten zeigt, ist ein Webserver mit ein paar Dateien. Sobald eine App im Namen eines Nutzers Dateien liest oder ein Modell aufruft, kommt eine ganze Kette dazu. Jeder Schritt hat einen Grund. Die Grundlagen stehen ausführlicher in [Grundlagen](../themen/grundlagen.md), die Rechte im [Buch „Berechtigungen“](../buch/index.html).

```text
Hosting (Azure Web-App)
  → App-Registrierung in Entra ID (Steckbrief: Name, Rechte, Rücksprungadressen)
  → Rechte wählen: delegiert oder als Anwendung
  → Zustimmung (Consent), für viele Rechte durch einen Admin
  → Anmeldung mit MSAL im Browser oder in Teams
  → Token für genau einen Dienst
  → Abruf über Microsoft Graph (Dateien, Suche, Listen, Meetings)
  → eigener Server prüft das Token und ruft das Modell auf (Azure / Foundry)
  → optional: Teams-Tab oder Bot
  → laufende Kosten: Plan, Lizenzen, Tokens, Pflege
```

| Schritt | Warum |
|---|---|
| **Hosting**, hier Azure App Service (Web-App) | Die Anmeldung braucht eine feste HTTPS-Adresse als Rücksprungziel, ein Bot oder Webhook eine öffentlich erreichbare. Alle sieben Apps teilten sich einen Plan der Stufe B1 (1,75 GB, Listenpreis rund 13 € im Monat, nicht auf der Preisseite geprüft) |
| **App-Registrierung in Entra ID** | Ohne sie gibt es kein Token. Sie legt fest, welche Rechte die App überhaupt anfragen darf und wohin Microsoft nach der Anmeldung zurückspringt |
| **Delegierte Rechte** (`scp` im Token) | Die App handelt im Namen des angemeldeten Nutzers und sieht nur die Schnittmenge aus App-Recht und Nutzerrecht. Die App muss Rechte nicht nachbauen, der Server braucht keinen eigenen Dateizugriff |
| **Anwendungsrechte** (`roles` im Token) | Die App handelt ohne Nutzer, etwa ein Bot oder ein nächtlicher Lauf. Heikel, weil nur das App-Recht gilt. Eingrenzen lässt es sich mit `Sites.Selected` oder `Lists.SelectedOperations.Selected`, dann gibt man jede Site oder Liste einzeln frei |
| **Consent / Admin Consent** | Ein Recht wirkt erst, wenn jemand zugestimmt hat. Viele Rechte, etwa auf Meeting-Transkripte, verlangen die Zustimmung eines Admins. Ein nachträglich ergänztes Recht ohne Zustimmung ließ die stille Anmeldung scheitern |
| **Anmeldung mit MSAL** | Die App sieht kein Passwort, sie leitet zu Microsoft und bekommt ein Token zurück. MSAL.js im Browser braucht kein Client-Secret, das ein Browser ohnehin nicht schützen könnte |
| **Token** | Ein signierter, zeitlich begrenzter Ausweis für genau einen Dienst. Graph, eigener Server und Modell bekommen je ein eigenes. Ein Graph-Token lehnt der eigene Server ab, weil die Zielgruppe nicht passt |
| **Graph-Abruf** | Eine gemeinsame API für Dateien, Suche, Listen, Planner, To Do und Meetings. Für Textstellen fürs Modell gibt es die Retrieval API, die eine Copilot-Lizenz oder Pay-as-you-go braucht |
| **Teams-Tab oder Bot** | Ein Tab ist dieselbe Webseite in einem Rahmen, die Anmeldung übernimmt Teams (Nested App Authentication). Ein Bot braucht zusätzlich eine Azure-Bot-Ressource, ein hochgeladenes App-Paket und muss im Kanal installiert sein, sonst darf er nichts schicken |
| **Modellzugang über Azure / Foundry** | Der Browser darf das Modell nicht direkt aufrufen. Der Server meldet sich mit einer verwalteten Identität (Managed Identity) und einer Azure-Rolle an, ohne Schlüssel |
| **Laufende Kosten** | Plan, Lizenzen (Copilot für die Retrieval API, Premium für manche Power-Automate-Auslöser), Tokens und vor allem die Pflege. Tokens waren in allen Tests der kleinste Posten |

### Woran es live hängen blieb

- **Transkripte:** Trotz erteilter Rechte kam zuerst `403 GraphAccessToTranscriptsDisabled`. Erst nachdem im Teams Admin Center der Graph-Zugriff auf Transkripte und die Sprecherzuordnung eingeschaltet waren, klappte der Abruf, nach rund 20 Minuten. Ohne diese Admin-Entscheidung gibt es keinen Abruf per API, egal welche Rechte die App hat.
- **Retrieval API nur mit Stichwörtern:** Ganze Sätze fanden auch nach rund 18,5 Stunden nichts, obwohl die Doku sie empfiehlt. Ein eigener Index fand dieselben Sätze. Neue Dateien tauchten in der Graph-Suche nach rund 30 Minuten auf, in der Retrieval API nach 53 Minuten noch gar nicht. Siehe [Index und Retrieval](../themen/index-und-retrieval.md) und [Eigener Index mit Foundry IQ](../themen/eigener-index-foundry-iq.md).
- **Bot nicht installiert:** Der Bot Connector antwortete mit `403 BotNotInConversationRoster`, solange das Teams-Paket nicht hochgeladen war. Der Upload ist ein Schritt in Teams, den die App nicht selbst erledigen kann.
- **Zustimmung fehlt:** Das vorhandene Token einer anderen App bekam bei To Do 403, die eigens angelegte App mit `Tasks.ReadWrite` wirkte erst nach der Zustimmung. Verbindungen in Power Automate gehören einem Nutzer und werden meist im Browser erteilt, eine Genehmigungsverbindung ließ sich überraschend auch ohne Browser anlegen.
- **Doku und Wirklichkeit:** Die Rechte-Tabellen der Doku waren mehrfach enger oder weiter als das Verhalten. `Lists.SelectedOperations.Selected` reichte für Schreiben und sogar für ein Webhook-Abo, wo die Doku `Sites.ReadWrite.All` und `Sites.Read.All` nennt. Verlassen sollte man sich darauf nicht.
- **Voller Plan:** Mit sieben Apps lag der B1-Plan bei rund 80 % Speicher. Zwei Deployments scheiterten an der Startprüfung von 230 Sekunden. Ohne Always On dauerte der erste Aufruf nach einer Pause bis zu 14 Sekunden.

## Die Apps im Einzelnen

### Projektwissen

- **Ziel:** Fragen an erfundene Projektdateien in SharePoint stellen, im Browser und als Teams-Tab.
- **Lösung:** Klassisches RAG über Microsofts Index. Der Browser holt mit dem Token des Nutzers bis zu 25 Dateien mit Textstellen über die Retrieval API. Der eigene Server prüft das Token und lässt das Modell nur aus diesen Stellen antworten, mit Quellen.
- **Live:** Ja, die einzige App mit der ganzen Kette aus Anmeldung (MSAL), delegierten Rechten, Graph, Server mit verwalteter Identität und Teams-Tab. In Teams genügte nach einer einmaligen Zustimmung die vorhandene Anmeldung.
- **Ergebnisse:** „budget velmora“ ergab 189.000 € mit Quellen, 501 Tokens. Stichwörter wurden nach unter 35 Minuten gefunden. Ganze Fragen fanden nichts. „budget“ lieferte erst 19 und später 24 von 24 Projekten, „Festpreis“ nur 18.
- **Erkenntnis:** Microsoft übernimmt Crawling, Index und Rechte, dafür hat man wenig Einfluss darauf, was gefunden wird. Eine vollständig wirkende Liste kann unvollständig sein, ohne Fehlermeldung.
- **Eigenbau oder Kauf:** Für Einzelfragen mit bekannten Begriffen ein schlanker Eigenbau auf fertigem Index. Für Listen über alle Projekte reicht der Weg nicht, siehe Projektkartei.

### Prozessatlas

- **Ziel:** Aus verteilten Unterlagen zu einem Ablauf (Anleitung, veraltete Fassung, Interview, Folien, Excel) einen Prozessentwurf mit Quellen erzeugen, als Diagramm und Simulation.
- **Lösung:** Das Modell liest alle Dokumente eines Prozesses und antwortet als JSON nach festem Schema. Danach arbeitet nur das Programm: Zitate wörtlich prüfen, Regeln prüfen, Diagramme zeichnen, ereignisgesteuert simulieren. Neue Dokumente erzeugen nur Änderungsvorschläge, ein Mensch nimmt sie an.
- **Offline:** Acht erfundene Prozesse, 43 Dateien, lokal ausgelesen. Die Seite zeigte vorberechnete Ergebnisse. Ein Live-Weg aus SharePoint mit Anmeldung ist gebaut, aber nicht mit echter Anmeldung geprüft.
- **Ergebnisse:** In rund einer Minute ein Entwurf, alle Modelle fanden 98 bis 100 % der Schritte. Im Lückentest erfand kein Modell einen der 16 fehlenden Schritte. Widersprüche zwischen zwei Interviews meldete das große Modell 8 von 8, `gpt-5.4-mini` nur 2 von 8. 25 fremde Dateien verschlechterten nichts, kosteten aber das Vierfache an Tokens. Eine eingeschleuste Anweisung im PDF wurde nicht befolgt.
- **Erkenntnis:** Das Interview trägt am meisten. Aus Folien oder Excel allein entsteht kein belastbarer Ablauf, es fehlen Verzweigungen, Ausnahmen und bei Folien die Rollen. Alle Zitate können stimmen und das Modell trotzdem falsch sein: Ohne eigene Regel machten große Modelle aus „dann wird telefoniert“ eigene Schritte.
- **Grenze:** Die Interviews entstanden aus dem Soll, echte sind lückenhafter. Dokumente zeigen den Soll-Zustand, nicht den gelebten Prozess.
- **Eigenbau oder Kauf:** Sinnvoll als Vorbereitung gegen die leere Seite. Für Versionierung, Freigaben und Teamarbeit sind Fachwerkzeuge weit voraus. Denkbar: hier den Entwurf erzeugen und in ein Modellierungswerkzeug übernehmen.

### Projektkartei

- **Ziel:** Fragen über alle Projekte vollständig beantworten, etwa „Budget aller Projekte“. Eine Suche liefert die besten Treffer, eine Liste braucht alle.
- **Lösung:** Jedes Projekt wird einmal ausgelesen. Das Modell sammelt zu jedem Feld alle Fundstellen, das Programm entscheidet nach der Regel „neuere Fassung schlägt ältere“ und meldet Widersprüche. Fragen laufen danach über feste Auswertungen ohne Modell, mit Angabe, welche Karten keinen Wert haben.
- **Offline:** Die Daten wurden lokal per Graph geladen (Delta-Abfrage) und ausgelesen. Die öffentliche App las nur zwei fertige JSON-Dateien, ohne Identität, ohne Rolle, nur GET.
- **Ergebnisse:** 24 von 24 Karten und 168 von 168 Feldern bei beiden Modellen, kein veralteter Wert als aktuell. Widersprüche: `gpt-6.1-sol` 48 von 48, `gpt-5.4-mini` 30 von 48. Erster Delta-Lauf 216 Aufrufe in 11,2 s, zweiter ohne Änderung 1 Aufruf in 0,5 s und 0 Tokens. Einmalig rund 165.000 Tokens, jede Frage danach 0.
- **Erkenntnis:** Im Härtetest war ein Budget still falsch: „240 T€“ plus später „erhöht sich um 20 T€“ wurde zu 240.000 € bzw. 20.000 € statt 260.000 €. Die Zitate stimmten, die Prüfung merkte nichts. Der Vorsprung gegenüber der Retrieval API ließ sich bei „budget“ nicht wiederholen (24 von 24), bei „Festpreis“ und ganzen Sätzen aber deutlich.
- **Grenze:** Nur vorgedachte Fragen. Gezahlt wird für Vollständigkeit, nicht für Ersparnis: Nach Tokens rechnet sich die Kartei erst nach rund 48 Budgetfragen.
- **Eigenbau oder Kauf:** Eigenbau, weil kein fertiger Dienst eine garantierte Vollständigkeit liefert. Für freie Fragen bleibt die Retrieval API.

### Beleg-Prüfer

- **Ziel:** Belege wie Quittungen und Rechnungen prüfen. These: Dubletten, fehlende Angaben und Richtlinienverstöße lassen sich größtenteils mit festen Regeln prüfen, KI braucht es vor allem beim Auslesen.
- **Lösung:** Ein Beleg wird gelesen (Sprachmodell mit Bild oder Azure Document Intelligence `prebuilt-receipt`), danach prüfen feste Regeln aus einer Richtlinie als Konfiguration, ohne Modell.
- **Offline:** 97 erfundene, teils absichtlich beschädigte Belege, einmal lokal ausgelesen. Die Seite zeigte nur vorberechnete Ergebnisse, kein Upload.
- **Ergebnisse:** Feldgenauigkeit `gpt-6.1-sol` 98,0 %, `gpt-5.4-mini` 96,3 %, Document Intelligence 87,0 % (stark bei Betrag und Datum, schwach bei Positionen und Teilnehmern). Fehlalarme durch Lesefehler: 5, 19 und 49. Bei stark beschädigten Belegen fiel `gpt-5.4-mini` auf 84,8 %. Rund 1.500 Tokens je Beleg, Lesekosten etwa 0,2 Cent.
- **Erkenntnis:** These weitgehend bestätigt. Gefährlich ist „stimmig falsch“: Das Lesen glättete teils genau die Abweichung, die eine Regel finden soll. Die Selbsteinschätzung „unsicher“ markierte nur einen Teil der Fehler. Zehn eingebaute Fallen zeigen Fälle, die Kontext oder Urteil brauchen, etwa ein handschriftlicher Genehmigungsvermerk.
- **Eigenbau oder Kauf:** In einem Rechenbeispiel mit erfundenen Annahmen (wenige Vorgänge im Monat) trägt sich weder Eigenbau noch fertige Standardsoftware rein über gesparte Minuten. Die fertige Lösung war günstiger als der Eigenbau. Tokens entschieden nichts, Arbeitszeit und Betrieb schon.

### Freigabe-Röntgen

- **Ziel:** Vor einem Copilot-Rollout zu breit freigegebene Dateien und Sites finden. Copilot nutzt alles, was eine Person öffnen darf.
- **Lösung:** Ein ausschließlich lesender Scan über Graph löst Rechte bis zur Person auf, mit Vererbung und verschachtelten Gruppen. Feste Regeln vergeben Punkte und Stufen, die Sensibilität kommt aus dem Dateinamen. Eine getrennte Variante lässt ein Modell den Inhalt einschätzen. Die App ändert nie Rechte.
- **Teils live:** Der Scan lief lokal gegen ein Testlabor im Tenant (55 Elemente). Die öffentliche Demo zeigte geschwärzte, vorberechnete Ergebnisse, weil ein Rechte-Scan selbst sensibel ist.
- **Ergebnisse:** Im nachträglich gebauten Blindsatz 20 von 20 Befunden, 0 Fehlalarme, aber nur 7 von 11 Risikostufen richtig (englische Namen, Synonyme, Ordnername). Ohne Vererbung und Gruppen übersah eine naive Variante fast die Hälfte. Mit Modell 10 von 11 Stufen, dafür eigene Fehler, und ein Satz im Dokument konnte die Einstufung senken. Labor-Scan mit `$batch` 4,5 s.
- **Grenze:** SharePoint-Gruppen sind über Graph nicht auflösbar, direkte Mitglieder etwa bei „Besucher“ bleiben unsichtbar. Nur zwei Konten, die Sicht einer Person ohne Recht ist nicht geprüft.
- **Eigenbau oder Kauf:** Nützlich für kleine Organisationen ohne eigene IT. Mit Copilot-Lizenz zuerst Purview und die Berichte von SharePoint Advanced Management prüfen, dieses Werkzeug ersetzt sie nicht.

### Protokollpilot

- **Ziel:** Aus gesprochener, unordentlicher Sprache Aufgaben mit Inhalt, Verantwortlichem und Termin ziehen und nach Freigabe in Planner oder To Do anlegen.
- **Lösung:** Das Modell liest das Transkript und liefert Aufgaben mit wörtlichem Zitat und Zeitmarke. Das Programm prüft Zitate und rechnet relative Termine nach. Ein Mensch setzt Häkchen, erst dann wird angelegt, ohne Dubletten beim zweiten Absenden.
- **Teils live:** Gemessen auf 46 erzeugten Meetings, offline. Real im Tenant liefen die Anlage in Planner und To Do (lokal, nicht über die öffentliche Seite) und der Abruf eines echten Transkripts mit delegierter Anmeldung, nach dem Admin-Schalter im Teams Admin Center.
- **Ergebnisse:** Aufgaben gefunden: `gpt-6.1-sol` 99,0 %, `gpt-6.1-terra` 99,3 %, `gpt-5.4-mini` 94,3 %. Termine nach Prüfung durch das Programm 98,5 bis 100 %, das Modell allein 70 bis 94 %. Rund 3.500 Tokens je kurzes Meeting. Im echten Meeting (2 Minuten, ein Sprecher, Wortfehlerrate 16,9 %) fanden Copilot und die eigenen Modelle je 5 von 5 Aufgaben. Termine als Datum und Zitate lieferte nur der Eigenbau.
- **Erkenntnis:** Stimmt die Sprecherzeile nicht und verrät der Text das nicht, geht die Aufgabe an die falsche Person, bei jedem Modell, ohne Warnung. Die Information fehlt im Transkript. Bei einem gemeinsamen Mikrofon im Raum wäre das der Normalfall.
- **Grenze:** Aufzeichnen und Transkribieren ist heikel: Einwilligung, Betriebsrat und DSGVO sind vor echtem Einsatz zu klären.
- **Eigenbau oder Kauf:** Für Zusammenfassungen eher Copilot, wenn die Lizenz ohnehin da ist (Copilot Business laut Microsoft 21 US-Dollar je Nutzer und Monat bei Jahreszahlung). Eigenbau nur für Prüfbarkeit, gerechnete Termine und Freigabe vor dem Schreiben, oder als Kombination: Copilots Aufgaben per API holen und nur prüfen und anlegen.

### Genehmigungsbot

- **Ziel:** Einen Antrag ab einem Betrag in Teams genehmigen lassen und mit Wer und Wann in eine SharePoint-Liste zurückschreiben. Der Kern sind die Fälle, an denen einfache Bots scheitern: Doppelklick, zwei gleichzeitige Entscheider, Vier-Augen-Regel, geänderter Betrag, Eskalation, Neustart.
- **Lösung:** Die Listenzeile ist die einzige Wahrheit, geschrieben wird mit `If-Match`. Ein Zeitgeber alle 60 Sekunden gleicht ab, ein Graph-Webhook beschleunigt nur. Adaptive Cards mit `Action.Execute`. Kein Modell, es gab keinen Grund für eines.
- **Live:** Ja. Bot in Azure (Azure Bot F0, Microsoft 365 Agents SDK), Anmeldung ohne Geheimnis über eine verwaltete Identität mit Föderation, Recht nur auf diese eine Liste.
- **Ergebnisse:** Ein erfundener Testantrag über 612 € kam als Karte in Teams, der Klick auf „Genehmigen“ schrieb Entscheider und Zeit in die Liste. Beim eigenen Antrag zeigte die Karte keine Knöpfe. 91 Tests. Webhook reagierte nach 21 bis 41 s, kaum schneller als der Zeitgeber. Gefälschte Aufrufe wurden alle abgewiesen. Ein verlorener paralleler Schreibvorgang lieferte einmal `500` statt des dokumentierten `412`.
- **Grenze:** Zwei echte Personen, Handy und Klick während eines Neustarts sind nicht getestet. Der Bot braucht Always On.
- **Eigenbau oder Kauf:** Eher Lernprojekt als Empfehlung. Ein Power-Automate-Nachbau lief, bringt Oberfläche, Verlauf und Betrieb mit, ließ aber eine Selbstgenehmigung zu. Vier Augen, Betragsstufen und Eskalation muss man dort selbst bauen.

## Was sich quer durch die Tests zeigt

- **Modell nur dort, wo es nötig ist:** Auslesen braucht ein Modell. Prüfen, Rechnen, Abfragen und Zeichnen gehen ohne und sind danach reproduzierbar und kostenlos.
- **Das günstige Modell reicht oft, aber nicht überall:** Bei Schritten und Feldern lag `gpt-5.4-mini` nah dran. Bei Widersprüchen und beschädigten Belegen war das große Modell klar besser.
- **Eigene Tests täuschen:** Mehrmals lag ein auffälliges Ergebnis am Testaufbau oder an einer einzelnen Stichprobe, etwa ein Leck im Generator, das wie erfundene Schritte aussah. Die sechs fehlenden Budget-Projekte traten bei einer späteren Messung nicht wieder auf: „budget“ lieferte 24 von 24, knapp im Limit von 25 Dateien, „Festpreis“ nur 18 von 24.
- **Stille Fehler sind die teuersten:** Eine falsch zugeordnete Aufgabe, ein still fehlendes Projekt oder ein falsches Budget sehen aus wie richtige Ergebnisse. Deshalb Belege anzeigen und einen Menschen freigeben lassen.
- **Saubere Daten schmeicheln:** Alle Testdaten stammen aus Generatoren. Die Zahlen sind eine obere Grenze, keine Prognose für echte Ordner.

## Folgerungen

Vorläufige Schlüsse aus Tests mit erfundenen Daten. Ein allgemeines Raster zur Einordnung steht in [Einordnung von KI-Anwendungsfällen](../themen/einordnung-anwendungsfaelle.md).

1. **Wissen aus alten Projekten:** Einmaliges Auslesen ist vollständiger als Suche. Für Listen über alle Projekte spricht das für einen einmaligen Lauf über das Projektarchiv statt für Suche zur Laufzeit.
2. **Prozessentwurf als Vorbereitung:** Interessant, solange ein Mensch freigibt.
3. **Freigaben prüfen:** Vor einem breiten Copilot-Einsatz wahrscheinlich nützlich, vor allem ohne eigene IT-Abteilung. Fertige Werkzeuge von Microsoft wären zuerst zu prüfen.
4. **Belegprüfung:** Bei wenigen Vorgängen im Monat eher nicht selbst bauen. Zuerst messen, wie viele Vorgänge anfallen und wie lange eine Prüfung heute dauert.
5. **Protokolle:** Für Zusammenfassungen eher Copilot, wenn die Lizenz ohnehin da ist. Ein Eigenbau lohnt nur für Belege, gerechnete Termine und Freigabe vor dem Schreiben. Die Qualität von Copilot ist nur an einem kurzen Meeting geprüft. Einwilligung und Betriebsrat sind vor echtem Einsatz zu klären.
6. **Genehmigungen:** Eher ein Lernprojekt als eine Empfehlung. Zuerst Power Automate prüfen, dort fehlt aber eine Vier-Augen-Prüfung von Haus aus.
