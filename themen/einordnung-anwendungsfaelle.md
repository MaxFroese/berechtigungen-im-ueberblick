# Einordnung von KI-Anwendungsfällen

Stand: November 2026. Ein allgemeines Raster, um KI-Wünsche in einer kleinen Firma einzuordnen, bevor etwas gebaut wird. Es ist eine Einschätzung, keine Messung. Ausprobierte Beispiele mit erfundenen Daten stehen in [Apps im Überblick](../apps/README.md) und [Testergebnisse](testergebnisse.md).

**Kurz gesagt:** KI-Wünsche lassen sich in vier Gruppen trennen, die jeweils anders angegangen werden. Am meisten Erkenntnis bringt meist zuerst die Aufbereitung des vorhandenen Wissens. Alles, was laufend mitläuft und die Zusammenarbeit verändert, ist das größte Vorhaben und kommt besser später.

## Die vier Gruppen

| Gruppe | Was sie ausmacht | Vorgehen |
|---|---|---|
| **Arbeitsweise und Skills** | Personenzentriert, kein fester Ablauf, kaum Abstimmung mit anderen | Die Person baut es sich mit einem Coding-Agenten (etwa Claude Code oder Codex) und hält bewährtes Vorgehen als Skill fest |
| **Feste Abläufe** | Klare, wiederkehrende Schritte, oft mit mehreren Beteiligten | Erst Aufwand messen, dann eventuell automatisieren |
| **Datenbasis** | Einmal aufbereiten, danach für viele nutzbar | „Brechstange“: ein Agent arbeitet sich einmal durch den Bestand |
| **Laufende Arbeit** | Ein mitlaufender Stand, der sich mit jedem Ereignis ändert und die Teamarbeit verändert | Zurückstellen, bis die Datenbasis steht (Entwurf: [Umsetzungsplan Projektagent](umsetzungsplan-projektagent.md)) |

## Einen Fall einordnen

1. **Verändert es laufend die Zusammenarbeit im Team?** Dann gehört es zur laufenden Arbeit.
2. **Sind es immer dieselben Schritte?** Dann ist es ein fester Ablauf.
3. **Macht es im Kern eine Person für ihre eigene Arbeit?** Dann ist es Arbeitsweise oder Skill.
4. **Geht es darum, vorhandenes Wissen aufzubereiten?** Dann ist es die Datenbasis.

Derselbe Wunsch kann je nach Umsetzung in verschiedene Gruppen fallen. Macht jede Person etwas für sich, ist es Arbeitsweise. Läuft es automatisch für alle, ist es ein fester Ablauf. Erst wenn daraus ein mitlaufender Stand wird, ist es laufende Arbeit.

## Allgemeine Beobachtungen

- **Weniger KI als gedacht:** Viele feste Abläufe lassen sich größtenteils mit festen Regeln prüfen. KI hilft vor allem dort, wo unstrukturierte Eingaben gelesen werden müssen.
- **Fertiges zuerst prüfen:** Oft deckt ein vorhandenes Werkzeug (Copilot in Office, eine fertige Fachanwendung) den Fall schon ab. Ein Eigenbau lohnt sich erst, wenn klar ist, was dort fehlt.
- **Skills statt Erklärtexte:** Für personenzentrierte Aufgaben wirken eigene Regeln und zwei, drei gute Beispiele mehr als lange Anleitungen. Ergebnisse in festen Formaten sollten automatisch geprüft werden.
- **Personenbezug früh klären:** Sobald Kontaktdaten, Gesundheitsdaten oder Aufzeichnungen von Gesprächen im Spiel sind, sind Datenschutz und Einwilligung vor jeder Technik zu klären. Gesundheitsdaten brauchen in der Regel gar keine KI.
- **Firmengerät und Firmenkonto:** Auch persönliche Skripte sollten nicht auf privaten Geräten mit Firmendaten laufen.

## Wann sich etwas lohnt

„Nur eine Person nutzt es“ ist kein gutes Ausschlusskriterium. Ein Werkzeug für eine Person ist sinnvoll, wenn es billig gebaut ist und viel Zeit spart. Entscheidend ist die gesparte Zeit gegen Bau und Pflege, und das lässt sich messen: Vorgänge pro Monat mal gesparte Minuten.

## Eine mögliche Reihenfolge

1. **Datenfreigabe klären und das vorhandene Wissen einmal aufbereiten:** bringt meist die meiste Erkenntnis für alle anderen Fälle.
2. **Schnell sichtbare Skills:** einzelne Personen bauen sich Werkzeuge für ihre eigene Arbeit.
3. **Feste Abläufe:** erst messen, dann entscheiden.
4. **Laufende Arbeit:** erst, wenn die Datenbasis steht.

## Vorher zu klären

- Welche KI-Werkzeuge und Tarife sind für Kundendaten freigegeben, und wo werden die Daten verarbeitet (EU oder nicht)?
- Welche Bestandsdaten sind vorhanden und dürfen ausgewertet werden?
- Wie oft fallen die Vorgänge an, die automatisiert werden sollen, und wie lange dauern sie heute?
