# Grundlagen: Wie eine eigene App an Microsoft-365-Daten kommt

Stand: November 2026. Alle Daten im Testtenant sind erfunden. Ergebnisse und Kosten der Tests stehen in [Testergebnisse](testergebnisse.md), die Apps selbst in der [Übersicht der Apps](../apps/README.md).

**Kurz gesagt:** Dateien liegen in SharePoint, Microsoft indexiert sie, und eine App holt sich daraus Textstellen, mit den Rechten des angemeldeten Nutzers. Fast alles auf dieser Seite dreht sich um eine Frage: Wer darf gerade was, und woher weiß das System das?

Als durchgehendes Beispiel dient eine kleine Web-App „Projektwissen“, die Fragen zu einem Projektarchiv in SharePoint beantwortet, im Browser und als Tab in Teams. Die Kapitel bauen aufeinander auf und beschreiben jeweils die Idee und den Grund für die gewählte Lösung.

## 1. Die Grundbegriffe

| Begriff | Bedeutung | Im Testaufbau |
|---|---|---|
| **Tenant** | Abgeschlossener Bereich einer Organisation mit Nutzern, Daten und Regeln | Ein Testtenant mit wenigen Konten |
| **Entra ID** | Identitätsdienst des Tenants: Wer ist wer, welche App darf was | Dort stehen die Testnutzer und die Apps |
| **App-Registrierung** | Steckbrief einer App: Name, Rechte, erlaubte Rücksprungadressen | Eine Registrierung für die Web-App |
| **Service Principal** | Vertretung der App im Tenant, an ihr hängen Zustimmungen und Rollen | Entsteht nicht immer automatisch beim Registrieren, für eine App ohne Nutzer musste er eigens angelegt werden |
| **Token** | Signierter, zeitlich begrenzter Ausweis für genau einen Dienst | Graph, eigener Server und das Sprachmodell in Azure bekommen je ein eigenes Token |
| **Microsoft Graph** | Gemeinsame API für Dateien, Teams und Nutzer | `graph.microsoft.com` |
| **Azure-Rolle** | Rechte auf Azure-Ressourcen, getrennt von Graph-Rechten | Der Server hat die Rolle „Cognitive Services OpenAI User“ auf dem Modell |

**Merksatz:** Ein Zugriff braucht beides. Die App muss das Recht haben, und bei delegierten Rechten muss auch der Nutzer dürfen.

| Art | Wer handelt | Grenze | Beispiel |
|---|---|---|---|
| **Delegiert** (`scp` im Token) | App im Namen eines Nutzers | Schnittmenge aus App-Recht und Nutzerrecht | Die Web-App |
| **Als App** (`roles` im Token) | App ohne Nutzer | Nur das App-Recht, daher heikel | Ein Skript mit `Sites.Selected` |
| **Managed Identity** | Azure-Ressource als eigene Identität | Ihre Azure-Rollen, kein Secret nötig | Der Server ruft das Modell auf |

Ein Beispiel für die Schnittmenge: Fehlt der App das Recht `Mail.Read`, kann sie keine Mails lesen, auch wenn der angemeldete Nutzer sein Postfach natürlich öffnen darf.

## 2. Anmeldung: Wie die App an ein Token kommt

**Idee:** Die App sieht das Passwort nie. Sie schickt den Nutzer zu Microsoft und bekommt nach der Anmeldung ein Token zurück.

**Warum so:** Die Web-App nutzt die Bibliothek MSAL.js direkt im Browser und braucht dafür kein Client-Secret. Ein Browser könnte ohnehin kein Geheimnis bewahren. Der Ablauf hat drei Stufen:

1. Token still aus dem Cache holen.
2. Abgelaufenes Token still erneuern.
3. Erst wenn beides scheitert, ein Anmeldefenster öffnen.

Deshalb sieht man die Anmeldung in einem normalen Browserfenster mit bestehender Sitzung oft gar nicht, in einem privaten Fenster dagegen schon.

## 3. Graph: Wie die App an Daten kommt

**Idee:** Der Browser ruft Graph selbst auf und schickt das Token des Nutzers mit. Graph liefert nur, was dieser Nutzer sehen darf.

**Warum so:** Der Server braucht so keine Rechte auf Dateien. Wird er kompromittiert, sind die Dateien nicht direkt betroffen. Der Preis: Der Server kann nicht prüfen, welche Textstellen wirklich aus SharePoint kommen. Für einen Test ist das in Ordnung, für ein Produkt wäre der Abruf über den Server zu überlegen.

**Zwei verschiedene Wege zu Dateien:**

| Weg | Was passiert | Verhalten bei neuen Dateien |
|---|---|---|
| **Aufzählung** | Ordner durchgehen, keine Suche | Eine frisch hochgeladene Datei ist sofort da |
| **Suche** (`/search/query`) | Abfrage gegen den Index | Die Datei erscheint erst, wenn der Index nachgezogen hat |

## 4. Index und RAG: Wie eine Frage zur Antwort wird

**Idee:** RAG heißt: erst passende Textstellen suchen, dann das Modell nur daraus antworten lassen.

```text
Frage
  → Retrieval API: bis zu 25 Dateien mit Textstellen, gefiltert nach den Rechten des Nutzers
  → eigener Server: prüft das Token
  → Modell: formuliert die Antwort und nennt Quellen
  → Browser: zeigt Antwort und Quellen
```

**Warum so:** Microsoft übernimmt Crawling, Indexierung und Rechte. Gebaut wird nur die Antwortschicht. Dafür gibt es wenig Einfluss darauf, was gefunden wird, und die App ist an die Grenzen des Dienstes gebunden.

**Prompt-Injection:** Der Systemprompt erklärt die gelieferten Textstellen ausdrücklich zu Daten, weil darin versteckte Anweisungen stehen könnten. Das senkt das Risiko, schließt es aber nicht aus.

**Was die Messungen zeigen:**

| Beobachtung | Was daraus folgt |
|---|---|
| Stichwörter wurden nach unter 35 Minuten gefunden | Der Stichwortweg ist schnell |
| Ganze Fragen fanden auch nach rund 18,5 Stunden nichts | Der Grund ist offen, siehe [Index und Retrieval](index-und-retrieval.md) |
| „budget“ lieferte erst 19 und später 24 von 24 Projekten, „Festpreis“ nur 18 | Eine vollständig wirkende Liste kann unvollständig sein, und das Ergebnis schwankt |
| Aus den Dateien mit Widersprüchen kam der aktuelle Wert | Gutes Einzelergebnis, noch keine Aussage über alle Fälle |

Die Ergebnisse für ganze Fragen hält [Testfragen](testfragen.md) fest.

## 5. Der eigene Server: Warum es ihn überhaupt gibt

**Idee:** Der Browser darf das Modell nicht direkt aufrufen, weil er dafür Rechte in Azure bräuchte. Der Server ist die Schleuse.

**Zwei Prüfungen, zwei Identitäten:**

| Richtung | Geprüft wird |
|---|---|
| Browser → Server | Token: Signatur, Tenant, Zielgruppe (Audience), Scope `access_as_user` |
| Server → Modell | Managed Identity mit Azure-Rolle, kein Schlüssel |

Dafür hat die App einen eigenen Scope (`api://<client-id>/access_as_user`). Ein Graph-Token lehnt der Server ab, denn es ist für einen anderen Dienst ausgestellt. Ein Aufruf ganz ohne Token bekommt 401.

## 6. Teams: Dieselbe Seite in einem Rahmen

**Idee:** Ein Teams-Tab ist eine Webseite in einem Rahmen. Das hochgeladene App-Paket (Manifest) sagt Teams nur, welche Adresse es anzeigen soll.

**Warum so:** Mit Nested App Authentication übernimmt Teams die Anmeldung. Ein Token-Tausch auf dem Server wie beim älteren On-Behalf-Of-Weg entfällt. Dafür braucht die App-Registrierung eine spezielle Rücksprungadresse (`brk-multihub://…`), und der Server muss erlauben, dass Teams die Seite einrahmt. Im Tab meldet sich die App still an, der Nutzer sieht keine zweite Anmeldung.

## 7. `Sites.Selected`: Zugriff ohne Nutzer begrenzen

**Idee:** Ein Skript ohne Nutzer hätte mit `Sites.Read.All` Zugriff auf alle Sites. `Sites.Selected` erlaubt zunächst nichts, jede Site wird einzeln freigegeben.

**Das Ergebnis des Tests:**

| Zeitpunkt | Projektarchiv | Vertrauliche Site |
|---|---|---|
| Vor der Freigabe | 403 | 403 |
| Nach der Freigabe nur für das Projektarchiv | 200 | 403 |

**Warum das wichtig ist:** Es sind zwei Schritte in zwei Systemen. Das App-Recht liegt in Entra, die Freigabe auf der Site. Wer die Freigabe setzt, braucht selbst weitergehende Rechte, hier `Sites.FullControl.All`. Im Test waren das deshalb zwei verschiedene Identitäten: eine verwaltende, die freigibt, und eine lesende, die nur das Freigegebene sieht.

## 8. Copilot Studio und Power Automate als Code

**Idee:** Auch was wie reine Klickoberfläche aussieht, liegt als Daten vor. Ein Agent in Copilot Studio besteht aus YAML-Dateien, ein Flow ist ein Datensatz in Dataverse mit einer JSON-Definition.

**Agent:** Die Dateien enthalten Anweisungen, Modell und Websuche, die Wissensquellen (etwa SharePoint) und vorgefertigte Gesprächsthemen. Änderungen laufen über die Power Platform CLI in zwei Schritten, Hochladen und Veröffentlichen:

```bash
pac copilot push      # lokale Agent-Dateien hochladen
pac copilot publish   # hochgeladenen Stand veröffentlichen
```

**Flow:** Ein Genehmigungstest bestand aus einer SharePoint-Liste und einem Flow mit der Regel „über 500 € Genehmigung nötig“. Das ist ein einfacher Ausdruck ohne KI. Das passt zu der Beobachtung, dass feste Abläufe oft weniger KI brauchen als gedacht.

**Grenze:** Verbindungen wie die zu SharePoint gehören einem Nutzer und werden im Browser erteilt. Das bleibt Handarbeit. Dass der Testflow lief, lag an einer bereits vorhandenen Verbindung.

## 9. Fehler, aus denen sich etwas lernen lässt

| Fehler | Ursache | Was das lehrt |
|---|---|---|
| Entra-Fehler 530035 bei der Code-Anmeldung | Security Defaults blockieren den Device-Code-Weg | Nicht die Sicherheit abschalten, sondern einen erlaubten Weg nehmen: normale Browser-Anmeldung |
| Die Azure CLI lädt keine SharePoint-Dateien | Sie bekommt nur feste Graph-Rechte | Admin sein reicht nicht, auch die App braucht das Recht |
| Nachschlagen eines anderen Nutzers scheitert | `Group.ReadWrite.All` erlaubt nicht, andere Nutzer zu lesen | Rechte sind enger geschnitten, als sie klingen |
| Graph-Suche: 400 bei `region` | Der Parameter gilt nicht für delegierte Aufrufe | Delegiert und als App verhalten sich verschieden |
| Messung fiel aus: „Nicht angemeldet“ | Ein Recht war ergänzt, dem noch niemand zugestimmt hatte | Stilles Erneuern bekommt nur Rechte mit Zustimmung |
| Option `supportsNestedAppAuth` existiert nicht | In MSAL v5 entfallen, die Doku zeigte noch die alte Form | Bei Fehlern auch die installierte Version prüfen |
| Knopf-Flow per API startet ohne Eingaben | Der Startaufruf gab die Werte nicht weiter | Für den Test kurz ein HTTP-Auslöser, danach zurückgestellt |
| `pac`: „You must install .NET“ | Installationspfad unbekannt, `DOTNET_ROOT` fehlte | Installiert heißt noch nicht auffindbar |

## 10. Was das für kleine Organisationen bedeuten könnte

Das sind vorläufige Schlüsse aus einem kleinen Test mit erfundenen Daten.

- **Einzelne Fragen:** Über die Retrieval API funktionieren im Test bisher nur Stichwörter, ganze Sätze finden nichts. Das ist vor einem Einsatz zu klären, siehe [Index und Retrieval](index-und-retrieval.md).
- **Listen über alle Projekte:** Dafür reicht dieser Weg nicht. Sinnvoller ist, alte Projekte einmalig komplett auszuwerten und die Ergebnisse strukturiert abzulegen.
- **Rechte:** Delegierte Rechte nehmen viel Arbeit ab, denn die App muss sie nicht nachbauen.
- **Belege und Genehmigungen:** Eine Liste mit Regel ist schnell gebaut. Noch ungeprüft sind Belegerkennung, Genehmigung und Kosten.

## Offene Punkte

- Vergleich mit Copilot Chat und den Agenten, siehe [Copilot Studio](copilot-studio.md) und [Harnesses im Vergleich](harnesses-im-vergleich.md).
- Rechteprüfung auf einer privaten Site mit einem zweiten Konto.
- Scans und Bilder findet der Index bisher nicht.
