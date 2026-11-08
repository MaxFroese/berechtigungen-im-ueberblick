# Umsetzungsplan: Projektagent in Microsoft 365

Stand: November 2026. Architekturentwurf für eine kleine Beratungsfirma (Systemhaus) mit einigen Dutzend Mitarbeitenden, die mit Microsoft 365 arbeitet und einen externen IT-Dienstleister für den Tenant hat. Als Ganzes ist der Entwurf nicht gebaut. Einzelne Bausteine wurden im Testtenant ausprobiert, siehe [Was davon im Test geprüft wurde](#was-davon-im-test-geprüft-wurde).

**Kennzeichnung der Aussagen:**

- **[Doku]**: in der Microsoft-Dokumentation nachgelesen
- **[Prüfen]**: aus allgemeinem Wissen oder Community-Quellen, vor dem Bauen im Tenant testen
- **[Annahme]**: Vorschlag oder Beispiel, muss an die jeweilige Firma angepasst werden

---

## 1. Das Wichtigste in Kürze

**Die Mitarbeitenden arbeiten nur in Teams, Planner, Outlook und Office.** Ein kleiner Dienst im Hintergrund (Worker) liest neue Ereignisse, vergleicht sie mit einem Playbook und schlägt Änderungen am Projektstand vor. Ein Mensch bestätigt per Klick.

Vier Grundsätze tragen den Entwurf:

1. **SharePoint ist der Speicher.** Der Worker hält keine eigenen Kundendaten dauerhaft.
2. **Power Automate ist der Klebstoff, nicht die Intelligenz.** Es löst aus, zeigt Karten und schreibt nach Planner.
3. **Das Modell bekommt keine Werkzeuge.** Es liest vorbereiteten Text und liefert nur JSON. Geschrieben wird allein von geprüftem Code.
4. **Jedes Ereignis läuft gleich ab.** Ein Meeting oder ein fertiges Dokument führt immer zu Vorschlag, Entscheidung und Update.

```text
  Ereignis                 Worker                         Mensch
  ---------                ------                         ------
  Transkript fertig  --->  Kontext laden
  Dokument "fertig"  --->  Modell aufrufen, JSON prüfen
  Termin steht an    --->  Vorschlag in SP-Liste  --->    Karte in Teams
                                                             |
                                                  Annehmen / Ändern / Ablehnen
                                                             |
  Planner, Projektverlauf  <---  Power Automate  <-----------+
```

---

## 2. Was läuft wo?

| Ebene | Technik | Aufgabe | Enthält KI? |
|---|---|---|---|
| **Daten** | SharePoint-Listen und -Bibliotheken, Planner | Projektstand, Playbook, Dokumente, Aufgaben | Nein |
| **Klebstoff** | Power Automate | Karten posten, Antworten verarbeiten, Planner schreiben | Nein |
| **Worker** | Kleines Programm (etwa Python), zuerst lokal, später als geplanter Job in der Cloud | Ereignisse holen, Kontext bauen, Modell aufrufen, Ergebnis prüfen | Ruft sie auf |
| **Modell** | Claude oder GPT, je nach Vorgabe zum Datenstandort | Auswertung, strukturierte Ausgabe | Ja |
| **Anzeige** | Teams-Kanal, SharePoint-Seite | Karten und Projektübersicht | Nein |
| **Chat (später)** | Copilot Studio oder SharePoint-Agent | Fragen an Playbook und Wissenssammlung | Ja |

### Wo der Worker läuft

| Stufe | Ort | Geeignet für |
|---|---|---|
| **Test** | Firmenrechner, von Hand gestartet | Stufen 0 bis 2 |
| **Betrieb** | [Azure Container Apps Job](https://learn.microsoft.com/en-us/azure/container-apps/jobs) mit Cron-Zeitplan **[Doku]** | Regelmäßige Läufe, zum Beispiel alle 15 Minuten |
| **Nur falls nötig** | Azure Function als Webhook-Empfänger | Reaktion in Echtzeit |

Für den Anfang reicht ein Cron-Job. Ein paar Minuten Verzögerung stören hier nicht, und ein Job ist einfacher zu betreiben als ein dauerhaft erreichbarer Webhook.

### Welches Modell

Das Modell bleibt austauschbar, weil der Worker es nur über eine dünne Schnittstelle anspricht: Prompt und JSON-Schema hinein, JSON heraus.

| Vorgabe | Option | Anmerkung |
|---|---|---|
| **EU nicht zwingend** | Claude oder GPT direkt per API | Am einfachsten |
| **EU zwingend, Claude** | Claude über Bedrock oder Vertex in einer EU-Region | Laut Anthropic in Europa verfügbar, in Foundry noch nicht |
| **EU zwingend, möglichst Azure** | GPT über Azure mit Data Zone EU | Modell und Region im Foundry-Portal prüfen **[Prüfen]** |

---

## 3. Die Datenstruktur

### 3.1 Aufteilung

```text
Zentrale Site "KI-Projektwissen"
├── Liste: Projektregister       (welche Projekte, wo, welche Regeln)
├── Liste: Playbook-Phasen
├── Liste: Playbook-Schritte
├── Liste: Playbook-Risiken
├── Liste: Playbook-Änderungen   (Vorschläge mit Freigabe)
└── Bibliothek: Wissenssammlung  (freigegebene Steckbriefe)

Pro Projekt: Team und zugehörige SharePoint-Site
├── Bibliothek: Dokumente        (mit Spalten Status, Phase, Typ)
├── Liste: Steckbrief
├── Liste: Verlauf               (Entscheidung, Frage, Risiko, Abweichung)
├── Liste: Vorschläge            (Eingang für die Karten)
├── Seite: Übersicht             (als Tab im Kanal)
└── Planner-Plan
```

Die Projektlisten liegen bewusst in der Projekt-Site. So gelten die bestehenden Rechte auch für den Output des Agenten, und beim Löschen des Projekts verschwinden sie mit. **[Annahme]** Die Struktur muss zur vorhandenen Projektablage passen und wird sonst angepasst.

### 3.2 Die wichtigsten Listen

**Playbook-Schritte (zentral)**

| Spalte | Typ | Beispiel |
|---|---|---|
| `PlaybookTyp` | Auswahl | Systemeinführung |
| `Phase` | Nachschlagen | Ist-Analyse |
| `Schritt` | Text | Bestehende Systeme und Schnittstellen erfassen |
| `Rolle` | Auswahl | Berater |
| `Ergebnis` | Text | Abschnitt in der Ist-Analyse |
| `DauerTypisch` | Zahl (Tage) | 3 |
| `Bedingung` | Text | Nur wenn Schnittstellen im Umfang |
| `Beleg` | Text | 12 von 15 Projekten |
| `Version` | Text | 0.3 |
| `Status` | Auswahl | Entwurf, geprüft, veraltet |

**Verlauf (pro Projekt)**

| Spalte | Typ | Inhalt |
|---|---|---|
| `Art` | Auswahl | Entscheidung, offene Frage, Risiko, Abweichung |
| `Titel` | Text | Kurzform, höchstens eine Zeile |
| `Begründung` | Text | Warum, kurz |
| `Folge` | Text | Auswirkung auf Zeitplan oder Umfang |
| `Verantwortlich` | Person | Zuständig |
| `Quelle` | Link | Transkript oder Dokument |
| `Status` | Auswahl | Offen, erledigt, verworfen |
| `VorschlagID` | Zahl | Rückverweis zum Vorschlag |

**Vorschläge (pro Projekt)**

| Spalte | Typ | Inhalt |
|---|---|---|
| `Typ` | Auswahl | Aufgabe, Abweichung, Risiko, Phasenwechsel, Agenda |
| `Titel`, `Begründung`, `Folge` | Text | Text der Karte |
| `Payload` | Mehrzeilig | JSON mit den auszuführenden Änderungen |
| `Konfidenz` | Zahl | 0 bis 1 |
| `QuelleLink` | Link | Beleg |
| `EreignisID` | Text | Verhindert doppelte Vorschläge |
| `Status` | Auswahl | Neu, angenommen, geändert, abgelehnt, abgelaufen |
| `EntschiedenVon` | Person | Für die spätere Auswertung |

Diese Liste ist zugleich die Datenquelle für die Annahmequote pro Vorschlagstyp.

---

## 4. Wie das Playbook entsteht

Das ist der wichtigste und unsicherste Teil. Die Reihenfolge lautet: **erst Daten extrahieren, dann verdichten, dann von Menschen korrigieren lassen, dann testen.**

> **Gibt es schon ein internes Vorgehensmodell, ist es der Startpunkt.** Dann schlägt die KI nicht die Phasen selbst vor, sondern füllt das Modell mit Details, Dauern und Risiken. Das ist genauer und wird eher akzeptiert.

### Schritt 1: Projekte auswählen

- 10 bis 20 abgeschlossene Projekte **eines** Typs **[Annahme]**, zum Beispiel Systemeinführungen der letzten zwei Jahre
- Projekte mit sehr anderem Kundentyp oder deutlich ältere Projekte als eigene Gruppe, nicht vermischen
- Projekte mit strengen Geheimhaltungsvereinbarungen ausschließen
- 2 bis 3 Projekte für den Backtest zurückhalten

### Schritt 2: Dokumente vorbereiten

1. Dateien pro Projekt über Graph oder einen synchronisierten Ordner auflisten.
2. Wichtige Dateien nach Name, Typ und Datum wählen: Angebot, Ist-Analyse, Konzept, Lenkungskreis-Folien, Abschlussbericht. Bei Unklarheit ordnet ein günstiges Modell zu.
3. Dateien in Text umwandeln, zum Beispiel mit MarkItDown, und mit Dateiname und Seite versehen.
4. Kundennamen vor dem Modellaufruf durch Platzhalter ersetzen, soweit möglich. Das senkt das Risiko, ersetzt aber nicht die menschliche Prüfung.

### Schritt 3: Pro Projekt getrennt extrahieren

Für jedes Projekt läuft ein eigener Aufruf ohne Wissen über andere Projekte. Das Ergebnis ist eine Datei wie diese:

```json
{
  "projekt_id": "P-017",
  "profil": { "kundentyp": "Industrie", "altsysteme": 2, "umfang": ["Schnittstellen"] },
  "phasen": [
    {
      "name": "Ist-Analyse",
      "beginn": "2025-03",
      "ende": "2025-05",
      "ergebnisse": ["Ist-Analyse.docx"],
      "ereignisse": [
        {
          "art": "problem",
          "text": "Zweites Altsystem erst spät bekannt",
          "folge": "Phase verlängert",
          "quelle": "LK-Folien-April.pptx, Folie 4"
        }
      ]
    }
  ],
  "unsicher": ["Enddatum Pilotphase nicht belegt"]
}
```

Regel: Ohne Quelle kein Eintrag. Lieber `unsicher` als geraten.

### Schritt 4: Stichprobe prüfen

Drei Projektleitungen prüfen jeweils ihr eigenes Projekt. Fehler führen zur Anpassung von Prompt oder Dokumentauswahl, danach läuft die Extraktion noch einmal. Erst dann geht es weiter.

### Schritt 5: Verdichten

Das Modell bekommt **nur die JSON-Dateien**, nicht die Rohdokumente, und erzeugt:

- Phasen mit typischer Reihenfolge und Dauerspanne
- Schritte mit Rollen und Ergebnissen
- Varianten nach Profil, zum Beispiel ein oder mehrere Altsysteme
- Risiken mit Häufigkeit und Gegenmaßnahme
- pro Aussage einen Beleg wie „12 von 15 Projekten“

Dünn belegte Aussagen werden markiert. Ein Sprachmodell findet immer eine Struktur, deshalb muss der Beleg sichtbar bleiben.

### Schritt 6: Workshop mit erfahrenen Beratern

Zwei Stunden mit zwei oder drei erfahrenen Beratern, weil in Dokumenten vieles fehlt:

- Stimmt der Ablauf? Was steht nur auf dem Papier?
- Woran merkt man früh, dass ein Projekt kippt?
- Was fehlt völlig?

Danach gibt eine benannte Person Version 0.1 frei, und die Inhalte kommen in die Playbook-Listen.

### Schritt 7: Backtest

Für jedes zurückgehaltene Projekt geht der Test Ereignis für Ereignis durch:

| Messung | Frage |
|---|---|
| **Abdeckung** | Wie viele der echten Schritte standen im Playbook? |
| **Risiken** | Wie viele echte Probleme hätte es vorher genannt? |
| **Zeit** | Lagen die echten Dauern innerhalb der Spanne? |
| **Nutzen** | Hätte es der damaligen Projektleitung geholfen? |

**Der Backtest entscheidet über das gesamte Vorhaben.** Bei schwachem Ergebnis wird nicht weitergebaut, sondern zuerst Projekttyp, Datenbasis oder Ansatz geändert.

### Das Playbook aktuell halten

1. Am Projektende vergleicht der Worker den Verlauf mit dem Playbook.
2. Wiederkehrende Abweichungen werden zu Einträgen in `Playbook-Änderungen`.
3. Die verantwortliche Person gibt sie frei.
4. Die Version steigt. Laufende Projekte bleiben auf ihrer Version, bis die Projektleitung wechselt.

Das Playbook ändert sich also nie automatisch.

---

## 5. Wie Projekte und Seiten angelegt werden

**Die Übersichtsseite sollte nicht von der KI erzeugt werden.** Sie wird einmal von Hand als Vorlage gebaut und zeigt danach die Listen live an.

Die [Graph-API für Seiten](https://learn.microsoft.com/en-us/graph/api/sitepage-create?view=graph-rest-1.0) ist dafür schlecht geeignet **[Doku]**:

- Sie unterstützt nur eine feste Liste an Web Parts, und das Listen-Web-Part steht nicht darauf.
- Laut Doku ist `Sites.ReadWrite.All` nötig, was weit über den gewünschten Zugriff hinausgeht.

### Die Übersichtsseite

1. In der Pilot-Site von Hand eine SharePoint-Seite bauen.
2. Listen-Web-Parts mit gefilterten Ansichten einfügen, etwa „offene Fragen“ oder „neue Abweichungen“.
3. Die Seite als Tab im Teams-Kanal anheften.

Ändern sich die Listen, ändert sich die Seite von selbst.

### Neues Projekt anlegen

| Stufe | Vorgehen |
|---|---|
| **Pilot** | Alles von Hand, einmalig |
| **Wenige Projekte** | Checkliste und vorbereitete Vorlage |
| **Viele Projekte** | Automatisch wie unten beschrieben |

Automatisch läuft das in diesen Schritten ab:

1. Die Projektleitung füllt einen Eintrag im `Projektregister` aus.
2. Ein Flow legt Team und Planner-Plan an **[Prüfen]**.
3. Die Site-Struktur kommt per [Site Template und Site Script](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/site-design-overview) oder per [PnP Provisioning](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/site-design-pnp-provisioning) **[Doku]**. Listen und Spalten gehen gut mit Site Scripts. Für Seiten mit Web Parts ist PnP wohl die bessere Wahl, laut [Community-Antworten](https://learn.microsoft.com/en-us/answers/questions/5515009/how-to-create-a-sharepoint-site-template-that-incl) wird dabei aber nicht alles vollständig übernommen **[Prüfen]**.
4. Ein Administrator oder ein eng begrenzter Admin-Prozess gibt dem Worker Zugriff auf die neue Site.
5. Der Worker liest den Steckbrief, wählt die Playbook-Variante und schreibt einen Vorschlag für Phasen und Startaufgaben.
6. Nach „Annehmen“ legt der Flow die Aufgaben im Plan an.

Vorhandene Projektvorlagen oder Automatisierungen werden weiterverwendet, statt sie zu ersetzen.

---

## 6. Wie alles angestoßen wird

### 6.1 Ereignisse

| Ereignis | Erkennung | Danach |
|---|---|---|
| **Meeting-Transkript fertig** | Worker fragt regelmäßig ab oder nutzt eine [Graph-Subscription](https://learn.microsoft.com/en-us/graph/teams-changenotifications-callrecording-and-calltranscript) **[Doku]** | Auswertung und Vorschläge |
| **Dokument auf „fertig“** | Power-Automate-Trigger „When a file is created or modified (properties only)“ schreibt einen Auftrag **[Prüfen]** | Abgleich mit dem Projektstand |
| **Aufgabe erledigt** | Tägliche Abfrage des Plans oder Planner-Trigger in Power Automate **[Prüfen]** | Prüfung des Phasenfortschritts |
| **Meeting steht an** | Täglicher Lauf prüft den Kalender der nächsten 24 Stunden | Agenda-Vorschlag |
| **Frist in Gefahr** | Täglicher Lauf ohne Modell, nur einfache Regeln | Karte an die Projektleitung |
| **Projektende** | Status im `Projektregister` | Steckbrief-Entwurf und Playbook-Vergleich |

**Zuordnung von Meeting zu Projekt:** Die Projekt-ID steht im Betreff, zum Beispiel `[P-2026-014] Lenkungskreis`. **[Annahme]** Das ist einfach und robust. Meetings ohne ID ignoriert der Worker, die Auswertung wird also bewusst pro Meeting aktiviert.

**Details zum Transkript:**

- Berechtigung laut Doku: `OnlineMeetingTranscript.Read.All` für Meeting-Transkripte **[Doku]**. Nicht zu verwechseln mit `CallTranscripts.Read.All`, das laut derselben Doku für Ad-hoc-Anrufe gilt.
- Eine Subscription muss **vor Beginn der Transkription** bestehen, sonst kommt keine Benachrichtigung **[Doku]**.
- Sie geht auch pro Organisator (`users/{userId}/onlineMeetings/getAllTranscripts`). Das ist für einen Pilot enger als der ganze Tenant **[Doku]**.
- Meetings in privaten Kanälen werden derzeit nicht unterstützt **[Doku]**.
- Ein Tenant-Admin kann den Graph-Zugriff auf Transkripte sperren **[Doku]**.
- Ob eine Application Access Policy nötig ist oder zusätzliche Kosten anfallen, ist zu klären **[Prüfen]**.

### 6.2 Der Lauf eines Ereignisses

```text
1. Auftrag holen           Noch nicht verarbeitet? Sonst abbrechen.
2. Projekt bestimmen       Aus Projekt-ID, sonst ignorieren.
3. Regeln prüfen           Darf dieses Projekt ausgewertet werden?
4. Kontext laden           Steckbrief, relevante Phase, offene Punkte, Ereignis
5. Modell aufrufen         Festes JSON-Schema, keine Werkzeuge
6. Ergebnis prüfen         Schema gültig? Quelle vorhanden? Höchstens 5 Vorschläge?
7. Vorschläge schreiben    In die Liste "Vorschläge" des Projekts
8. Ereignis abhaken        Damit nichts doppelt läuft
```

Beispiel für den Vertrag zwischen Modell und Code:

```json
{
  "vorschlaege": [
    {
      "typ": "abweichung",
      "titel": "Zwei Altsysteme statt einem",
      "begruendung": "Im Termin wurde ein zweites Altsystem genannt.",
      "folge": "Ist-Analyse ca. 1 Woche länger",
      "aktionen": [
        { "art": "aufgabe_anlegen", "titel": "Altsystem 2 prüfen", "rolle": "Berater" },
        { "art": "meilenstein_verschieben", "tage": 7 }
      ],
      "quelle": "Transkript, 12:40",
      "konfidenz": 0.8
    }
  ]
}
```

Vorschläge unter einer Konfidenzschwelle werden nicht gezeigt. Die Schwelle wird im Test eingestellt, und das Modell darf auch eine leere Liste liefern.

### 6.3 Die Karte in Teams

Power Automate übernimmt die Anzeige:

1. **Trigger:** neuer Eintrag in `Vorschläge` mit Status „Neu“.
2. **Aktion:** [„Post an adaptive card to a Teams channel and wait for a response“](https://learn.microsoft.com/en-us/power-automate/create-adaptive-cards) **[Doku]**.
3. **Bedingung je nach Antwort:**
   - **Annehmen:** Aufgabe anlegen, Eintrag in `Verlauf`, Status ändern.
   - **Ändern:** Felder aus der Karte übernehmen, dann wie Annehmen.
   - **Ablehnen:** nur den Status ändern, optional mit Grund.

Der Weg über die Liste hat zwei Vorteile: Worker und Flow sind nur über SharePoint verbunden, und vermutlich braucht man keine Premium-Connectoren. Der [HTTP-Request-Trigger ist nämlich Premium](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/site-design-trigger-flow-tutorial) **[Doku]**. Lizenzen und Laufzeitgrenzen wartender Flows sind trotzdem zu prüfen **[Prüfen]**.

Eine Karte lässt sich laut Doku nur einmal absenden, und für den Flow muss die Workflows-App in Teams installiert sein **[Doku]**.

---

## 7. Zugriff und Sicherheit

### Berechtigungen

| Bedarf | Empfehlung |
|---|---|
| **Listen und Dokumente** | [`Sites.Selected`](https://learn.microsoft.com/en-us/graph/permissions-selected-overview) mit expliziter Freigabe pro Site **[Doku]** |
| **Noch engerer Zugriff** | `Lists.SelectedOperations.Selected` für einzelne Listen **[Doku]** |
| **Transkripte** | Zunächst nur für Pilot-Organisatoren |
| **Planner** | Der Flow schreibt im Namen eines Dienstkontos oder einer Person **[Prüfen]** |

Ohne explizite Freigabe hat die App trotz erteilter Zustimmung **keinen** Zugriff, und Administratoren können ihn pro Ressource oder insgesamt entziehen **[Doku]**. Das Freigeben selbst erfordert hohe Rechte. Es gehört deshalb zum Admin, typischerweise zum IT-Dienstleister, und nie in den Worker.

### Schutz vor Manipulation

Dokumente, Mails und Transkripte können versteckte Anweisungen enthalten (Prompt Injection). Deshalb:

- **Keine Werkzeuge für das Modell.** Selbst ein manipuliertes Ergebnis ist nur Text.
- **Eingaben klar trennen.** Dokumentinhalte werden als Daten markiert, nicht als Anweisung.
- **Aktionen per Positivliste.** Der Code führt nur bekannte Aktionstypen aus.
- **Kein automatischer Versand nach außen.** Kundenkommunikation bleibt ein Entwurf.
- **Projekttrennung.** Jeder Lauf bekommt nur Daten eines Projekts plus das freigegebene Playbook.

### Daten

- Vor echten Läufen klären: Kundenverträge, Geheimhaltungsvereinbarungen, Auftragsverarbeitung, Datenstandort, Aufbewahrung beim Modellanbieter.
- Zustimmung zur Transkription einholen und Mitarbeitende informieren. Gibt es einen Betriebsrat, ihn früh einbinden.
- Worker-Logs ohne Inhalte, nur IDs, Status und Tokenzahlen.
- Temporäre Texte nach dem Lauf löschen.
- Die Ausgaben sind Projektvorschläge, keine Rechts- oder Steuerberatung.

---

## 8. Stufenplan

Jede Stufe hat ein klares Kriterium. Ist es nicht erfüllt, geht es nicht weiter.

| Stufe | Inhalt | Technik | Weiter, wenn |
|---|---|---|---|
| **0. Klären** | Datenfreigabe, Projektablage, Vorgehensmodell, Lizenzen, Ansprechpartner | Gespräche | Schriftliche Freigabe für Testdaten liegt vor |
| **1. Handtest** | Ein Transkript und ein Dokument von Hand auswerten | Skript auf dem Rechner | Teilnehmende finden die Vorschläge überwiegend brauchbar |
| **2. Playbook** | Schritte aus Kapitel 4 für einen Projekttyp | Skripte, JSON, Workshop | Backtest überzeugt die Berater |
| **3. Struktur** | Listen und Übersicht in einem Pilotprojekt, von Hand gepflegt | SharePoint | Projektleitung nutzt die Übersicht wirklich |
| **4. Erster Kreislauf** | Transkript → Vorschlag → Karte → Planner, Worker von Hand gestartet | Graph, Power Automate | Annahmequote und Aufwand sind vertretbar |
| **5. Betrieb** | Geplanter Job, Monitoring, Kostengrenzen, Dokumentation | Azure | Läuft stabil über mehrere Wochen |
| **6. Ausbau** | Weitere Ereignisse, Agenda, Fristen, Steckbrief | Worker erweitern | Pro Ereignistyp einzeln bewerten |
| **7. Rollout** | Automatische Projektanlage, weitere Projekttypen, Chat | PnP, Copilot Studio | Dauerhafte Betreuung ist geklärt |

**Start:** Stufe 0 und 1 parallel. Beides braucht keine Infrastruktur und liefert schnell etwas zum Zeigen.

---

## 9. Was gemessen wird

| Kennzahl | Quelle | Bedeutung |
|---|---|---|
| **Annahmequote pro Typ** | Liste `Vorschläge` | Zeigt, welche Schritte später automatischer laufen könnten |
| **Änderungsquote** | Liste `Vorschläge` | Hoch bedeutet: richtige Richtung, schwache Details |
| **Prüfzeit pro Karte** | Stichprobe oder Befragung | Sollte unter einer Minute liegen |
| **Stichprobe angenommener Vorschläge** | Manuelle Prüfung | Verhindert, dass blindes Durchklicken gut aussieht |
| **Verpasste Punkte** | Vergleich mit der Projektleitung | Was hat der Agent nicht erkannt? |
| **Kosten pro Lauf und Projekt** | Worker-Logs | Grundlage für jede Skalierung |
| **Echte Zeitersparnis** | Vorher-nachher-Vergleich | Gefühlte Ersparnis täuscht oft |

---

## 10. Vor dem Bauen zu prüfen

**Organisatorisch:**

- [ ] Dürfen Kundendaten verarbeitet werden, und bei welchem Anbieter?
- [ ] Ist EU-Verarbeitung zwingend?
- [ ] Wie sind Projekte heute in Teams und SharePoint organisiert?
- [ ] Gibt es ein internes Vorgehensmodell?
- [ ] Gibt es genug vergleichbare abgeschlossene Projekte eines Typs?
- [ ] Wer verwaltet den Tenant und erteilt App-Berechtigungen (eigene IT oder Dienstleister)?
- [ ] Sind Transkription und Graph-Zugriff auf Transkripte aktiv?
- [ ] Gibt es ein Azure-Abonnement und ein Budget?
- [ ] Wer betreut das System dauerhaft?

**Technisch im Tenant:**

- [ ] Transkript-Abruf mit `OnlineMeetingTranscript.Read.All`, Policy und mögliche Kosten
- [ ] App-Zugriff auf Planner, Rechte und Verhalten
- [ ] Trigger auf Statusänderung in der Bibliothek
- [ ] Karten-Flow ohne Premium-Connector, Laufzeitgrenze wartender Flows
- [ ] `Sites.Selected` für Listen und Dokumente mit der Worker-App
- [ ] Übersichtsseite als Tab, Listenansichten in Teams
- [ ] Modell, Region und strukturierte Ausgabe beim gewählten Anbieter

---

## Was davon im Test geprüft wurde

Im Testtenant mit erfundenen Daten, Stand November 2026 (Einzelheiten in [Testergebnisse](testergebnisse.md) und [Apps im Überblick](../apps/README.md)):

- **`Sites.Selected`** funktioniert wie beschrieben: Vor der Freigabe lieferten beide Test-Sites 403, nach der Freigabe einer Site nur diese 200. Zum Freigeben war `Sites.FullControl.All` nötig.
- **Karte in Teams mit Rückmeldung in eine SharePoint-Liste** lief in einem Testaufbau mit einem eigenen Teams-Bot, zum Vergleich auch als Power-Automate-Genehmigung: Karte nach Sekunden, Klick bis zum Listeneintrag. Mit zwei echten Personen ist das nicht getestet.
- **Aufgaben aus einem Meeting nach Planner** wurden real angelegt. Auch der Abruf eines echten Transkripts über die API und das Anlegen in To Do liefen im Nachtest real.
- Das Playbook selbst (Kapitel 4) und der Backtest sind nicht erprobt, weil dafür echte abgeschlossene Projekte nötig sind.

---

## Quellen

**Microsoft:**

- [Create a page in a SharePoint site (Graph)](https://learn.microsoft.com/en-us/graph/api/sitepage-create?view=graph-rest-1.0)
- [Selected permissions in OneDrive and SharePoint](https://learn.microsoft.com/en-us/graph/permissions-selected-overview)
- [Change notifications for transcripts and recordings](https://learn.microsoft.com/en-us/graph/teams-changenotifications-callrecording-and-calltranscript)
- [Fetch meeting transcripts and recordings](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/meeting-transcripts/overview-transcripts)
- [plannerTask resource](https://learn.microsoft.com/en-us/graph/api/resources/plannertask?view=graph-rest-1.0)
- [Adaptive Cards in Power Automate](https://learn.microsoft.com/en-us/power-automate/create-adaptive-cards)
- [SharePoint site templates and site scripts](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/site-design-overview)
- [PnP provisioning from a site script](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/site-design-pnp-provisioning)
- [Site design with Power Automate](https://learn.microsoft.com/en-us/sharepoint/dev/declarative-customization/site-design-trigger-flow-tutorial)
- [Community-Antwort zu Site-Vorlagen mit Seiten und Planner](https://learn.microsoft.com/en-us/answers/questions/5515009/how-to-create-a-sharepoint-site-template-that-incl)
- [Jobs in Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/jobs)

**Modelle und Daten:**

- [Regional Compliance (Anthropic)](https://claude.com/regional-compliance)
- [GPT-6.1 Terra in Microsoft Foundry (Microsoft), nur als Suchzusammenfassung gelesen](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/gpt-6-1-terra-now-generally-available-in-microsoft-foundry-build-agents-that-see/4567769)

**Kontext:**

- [The Enterprise AI Playbook (Stanford)](https://digitaleconomy.stanford.edu/app/uploads/2026/03/EnterpriseAIPlaybook_PereiraGraylinBrynjolfsson.pdf)
