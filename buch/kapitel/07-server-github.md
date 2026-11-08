# 7. Server und GitHub: Vom Commit zur laufenden App

Zwischen einer Codeänderung und der laufenden App in Azure liegen mehrere Stationen, und an jeder handelt eine andere Identität. Ein Mensch schiebt Code nach GitHub, ein Automatisierungsjob baut und rollt aus, und später greift die App selbst auf Daten und Geheimnisse zu. Wer diese Identitäten auseinanderhält, findet für jede Frage nach einem Recht die richtige Stelle.

![Der Weg vom Commit über GitHub und den Runner bis zur laufenden App in Azure, mit der handelnden Identität an jedem Pfeil. Auf dem Weg handeln drei verschiedene Identitäten aus zwei getrennten Rechtewelten, und die Rechte der späteren hängen faktisch davon ab, wer die früheren kontrolliert.](../bilder/kap07-commit-zur-app.svg)

*Der Weg vom Commit über GitHub und den Runner bis zur laufenden App in Azure, mit der handelnden Identität an jedem Pfeil. Auf dem Weg handeln drei verschiedene Identitäten aus zwei getrennten Rechtewelten, und die Rechte der späteren hängen faktisch davon ab, wer die früheren kontrolliert.*

## GitHub-Rechte: eine eigene Welt außerhalb von Entra ID

GitHub ist kein Teil des Tenants. Ein GitHub-Konto ist zunächst ein eigenes Konto bei einem anderen Anbieter, und keine Entra-Rolle oder Azure-Rollenzuweisung gibt ein Recht in GitHub. Das hat eine praktische Folge: Wird ein Nutzerkonto in Entra ID deaktiviert, behält dieselbe Person ihren GitHub-Zugang, solange niemand sie dort entfernt. Größere GitHub-Tarife können die Anmeldung an Entra ID koppeln, zum Beispiel über SAML. Die Rechte selbst werden trotzdem in GitHub vergeben.

Eine **GitHub Organization** ist der gemeinsame Firmenbereich auf GitHub, dem Repositories und Mitglieder zugeordnet sind. Ein Owner darf dort alles, ein Mitglied der GitHub Organization bekommt nur das eingestellte Basisrecht und das, was Teams und Repositories zusätzlich vergeben. Die eigentliche Arbeit regeln die Repository-Rollen:

| Repository-Rolle | Darf im Kern | Relevanz für den Deployment-Weg |
|---|---|---|
| Read | Code lesen und klonen | Sieht auch alle Workflow-Dateien |
| Triage | Zusätzlich Issues und Pull Requests verwalten | Kein Schreibzugriff auf Code |
| Write | Branches pushen, Pull Requests mergen, sofern nichts dagegen spricht | Kann Workflows ändern und dadurch Code auf dem Runner ausführen |
| Maintain | Zusätzlich Teile der Repository-Einstellungen verwalten | Wie Write, mit mehr Verwaltung |
| Admin | Alle Einstellungen, einschließlich Secrets und Schutzregeln | Kann Schutzregeln ändern oder abschalten |

Besonders unterschätzt wird Write. Die Rolle klingt nach „darf Code ändern“, bedeutet aber auch: Die Person kann in einem eigenen Branch eine Workflow-Datei anlegen, die beliebige Befehle ausführt. Was dieser Workflow erreicht, hängt davon ab, welche Geheimnisse und Vertrauensregeln ihm offenstehen.

**Branch-Schutz** sind Regeln, die direkte Änderungen an einem Branch einschränken. Typisch für `main` sind: Änderungen nur über Pull Requests, mindestens ein Review durch eine andere Person, erfolgreiche Tests vor dem Merge und kein Force-Push. Wichtig ist die Einstellung, ob auch Admins an die Regeln gebunden sind. Einzelne Regeln können bei privaten Repositories einen bezahlten Tarif voraussetzen. Eine Sicherheitsgrenze ist Branch-Schutz allerdings nur dann, wenn auch das Deployment an den geschützten Branch gebunden ist.

## GitHub Actions, Workflow und Runner

**GitHub Actions** ist die in GitHub eingebaute Automatisierung. Ein **Workflow** ist eine YAML-Datei unter `.github/workflows`, die festlegt, wann welche Schritte laufen, zum Beispiel bei jedem Push auf `main`. Er besteht aus Jobs, und jeder Job läuft auf einem eigenen Rechner, dem **Runner**. Es gibt zwei Arten:

- **GitHub-gehostete Runner** sind frische virtuelle Maschinen bei GitHub. Sie werden für einen Job erzeugt und danach verworfen.
- **Self-hosted Runner** sind eigene Rechner, die sich bei GitHub registrieren. Sie können ins interne Netz reichen, behalten aber je nach Einrichtung Zustand zwischen Jobs. Ein bösartiger Job kann dort Spuren hinterlassen, die den nächsten Job betreffen.

Der Code läuft also nicht abstrakt „in GitHub“, sondern als gewöhnlicher Prozess auf einem Linux- oder Windows-Rechner. Eine Action ist ein wiederverwendbarer, meist fremder Baustein eines Workflows. Jede Abhängigkeit, die `npm install` nachlädt, und jede fremde Action, die der Workflow einbindet, läuft dort mit und sieht dieselben Umgebungsvariablen und Dateien wie der Rest des Jobs. Außerdem bekommt jeder Job automatisch den `GITHUB_TOKEN`, ein kurzlebiges Token für Zugriffe auf GitHub selbst. Es hat nichts mit Azure zu tun, und der Block `permissions` im Workflow legt fest, was es darf.

## Deployment-Identität und Laufzeit-Identität

Die zentrale Trennung dieses Themas: Die **Deployment-Identität** rollt eine neue Version aus. Die **Laufzeit-Identität** ist die Identität, mit der die laufende App auf andere Dienste zugreift. Sie haben unterschiedliche Aufgaben und sollten nie dieselbe Identität sein.

| | Deployment-Identität | Laufzeit-Identität |
|---|---|---|
| Handelt | Beim Ausrollen, auf dem Runner | Dauerhaft, in der App |
| Typische Form | Service Principal einer App-Registrierung oder user-assigned Managed Identity, jeweils mit Federated Credential | Managed Identity der Ressource |
| Typisches Recht | „Website Contributor“ auf genau einer Web-App (Control Plane) | „Key Vault Secrets User“ auf einem Key Vault (Data Plane) |
| Wirkung bei Missbrauch | Kann Code austauschen | Kann die Daten lesen, auf die die App Zugriff hat |

Die Trennung hat eine Grenze. Wer neuen Code ausrollen darf, bestimmt, was mit der Laufzeit-Identität geschieht. Die Deployment-Identität liest zwar selbst kein Geheimnis aus dem Key Vault, kann aber Code ausrollen, der es liest. Rechte wirken hier transitiv:

> Write auf `main` → Deployment-Identität → Code in der App → Rechte der Laufzeit-Identität

![Die Kette von Write auf main über die Deployment-Identität und den ausgerollten Code bis zu den Rechten der Laufzeit-Identität. Die Trennung verhindert den direkten Griff nach den Daten, aber wer ausrollen darf, erbt mittelbar die Rechte der Laufzeit-Identität.](../bilder/kap07-transitive-rechte.svg)

*Die Kette von Write auf main über die Deployment-Identität und den ausgerollten Code bis zu den Rechten der Laufzeit-Identität. Die Trennung verhindert den direkten Griff nach den Daten, aber wer ausrollen darf, erbt mittelbar die Rechte der Laufzeit-Identität.*

Nützlich bleibt die Trennung trotzdem. Sie verhindert den direkten Griff nach den Daten, macht einen Angriff im Deployment-Verlauf sichtbar und begrenzt, was ein abgeflossenes Runner-Token erreicht. Der eigentliche Schutz dieser Kette liegt vorne: bei Branch-Schutz, Reviews und der Bindung des Deployments an einen bestimmten Kontext.

## OIDC und Federated Credential im Detail

Der Runner muss sich bei Entra ID ausweisen, ohne dass irgendwo ein dauerhaftes Passwort liegt. Das Mittel dafür ist **Workload Identity Federation**: eine Vertrauensregel, mit der ein externes Token gegen ein Entra-Token getauscht wird. **OIDC** ist dabei der Standard, mit dem GitHub dem Runner einen signierten Herkunftsnachweis ausstellt. Darin steht nicht „dieser Job darf etwas“, sondern nur „dieser Job läuft in diesem Repository in diesem Kontext“. Was daraus folgt, entscheidet allein Entra ID anhand einer vorab hinterlegten Regel.

Diese Regel heißt **Federated Credential** und legt drei Werte fest:

- **Issuer**: Wer muss den Nachweis ausgestellt haben? Für GitHub Actions ist das `https://token.actions.githubusercontent.com`.
- **Subject**: Für wen muss der Nachweis gelten? Bei GitHub stecken darin das Repository und der Kontext des Jobs.
- **Audience**: Für wen ist der Nachweis bestimmt? Der Standardwert für den Tausch bei Entra ID ist `api://AzureADTokenExchange`.

Zum Kontext eines Jobs kann ein GitHub Environment gehören. Das ist ein benanntes Ziel wie `production`, an das sich Schutzregeln hängen lassen, etwa eine Pflichtfreigabe oder eine Beschränkung auf bestimmte Branches. Auch das hängt bei privaten Repositories vom Tarif ab.

Das OIDC-Token des Runners ist ein gewöhnliches JWT. Ein Ausschnitt seiner Claims:

```json
{
  "iss": "https://token.actions.githubusercontent.com",
  "sub": "repo:contoso/meine-app:environment:production",
  "aud": "api://AzureADTokenExchange",
  "ref": "refs/heads/main"
}
```

An `sub` hängt die Sicherheit. Der Aufbau richtet sich nach dem Kontext des Jobs:

| Kontext des Jobs | Subject |
|---|---|
| Push auf einen Branch | `repo:contoso/meine-app:ref:refs/heads/main` |
| Job mit Environment | `repo:contoso/meine-app:environment:production` |
| Pull Request | `repo:contoso/meine-app:pull_request` |

Entra ID vergleicht das Subject exakt mit der Regel. Hier liegt die häufigste Falle beim Einrichten: Sobald ein Job ein `environment` angibt, lautet das Subject auf das Environment und nicht mehr auf den Branch. Eine auf `ref:refs/heads/main` eingerichtete Regel passt dann nicht. Die Fehlermeldung nennt meist das tatsächlich gelieferte Subject.

![Der Runner holt ein OIDC-Token von GitHub, tauscht es bei Entra ID gegen ein Access Token und rollt damit nach Azure aus. GitHub liefert nur den Herkunftsnachweis. Entschieden wird zweimal auf Microsoft-Seite: erst die Vertrauensregel, wer die Identität annehmen darf, dann die Rollenzuweisung, was sie darf.](../bilder/kap07-oidc-tausch.svg)

*Der Runner holt ein OIDC-Token von GitHub, tauscht es bei Entra ID gegen ein Access Token und rollt damit nach Azure aus. GitHub liefert nur den Herkunftsnachweis. Entschieden wird zweimal auf Microsoft-Seite: erst die Vertrauensregel, wer die Identität annehmen darf, dann die Rollenzuweisung, was sie darf.*

1. Der Job fordert beim OIDC-Dienst von GitHub ein Token an. Das geht nur, wenn der Workflow das Recht `id-token: write` gesetzt hat.
2. GitHub signiert ein kurzlebiges Token mit den Claims des Jobs.
3. Der Runner schickt dieses Token an Entra ID und nennt dabei Client-ID und Tenant der gewünschten Identität.
4. Entra ID lädt die öffentlichen Schlüssel des Issuers, prüft die Signatur und vergleicht Issuer, Subject und Audience mit dem Federated Credential.
5. Passt alles, stellt Entra ID ein kurz gültiges Access Token für die Azure-Verwaltung aus. Der nächste Job holt sich ein neues.
6. Mit diesem Token ruft der Runner Azure auf. Azure RBAC prüft nun wie bei jedem anderen Principal die Rollenzuweisung.

Die Einrichtung in Azure besteht aus zwei Teilen, der Vertrauensregel und dem eigentlichen Recht. Am Beispiel einer App-Registrierung:

```bash
az ad app federated-credential create --id <client-id> --parameters '{"name": "github-production", "issuer": "https://token.actions.githubusercontent.com", "subject": "repo:contoso/meine-app:environment:production", "audiences": ["api://AzureADTokenExchange"]}'

az role assignment create --assignee <client-id> --role "Website Contributor" \
  --scope "/subscriptions/<subscription-id>/resourceGroups/meine-rg/providers/Microsoft.Web/sites/meine-app"
```

`--assignee` akzeptiert hier auch die Client-ID und löst sie zum Service Principal auf, der dafür schon existieren muss. Der passende Workflow braucht kein Passwort:

```yaml
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    permissions:
      contents: read
      id-token: write          # erlaubt das Anfordern des OIDC-Tokens
    steps:
      - uses: actions/checkout@v4
      - uses: azure/login@v2
        with:
          client-id: ${{ vars.AZURE_CLIENT_ID }}
          tenant-id: ${{ vars.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
      - uses: azure/webapps-deploy@v3
        with:
          app-name: meine-app
          package: .
```

Die drei Kennungen liegen als normale Variablen (`vars`) vor. Sie sind keine Geheimnisse, denn ohne passendes OIDC-Token nützen sie niemandem. Das Recht `id-token: write` gehört nur an den Deploy-Job. Laufen Bau und Tests in einem eigenen Job ohne dieses Recht, kann ein Build-Skript mit nachgeladenen Abhängigkeiten gar kein OIDC-Token anfordern.

Damit schließt sich auch die Kette zum Branch-Schutz. Das Subject sorgt dafür, dass nur ein Job aus dem genannten Kontext die Identität bekommt. Branch-Schutz und Environment-Regeln sorgen dafür, dass in diesen Kontext nur geprüfter Code gelangt. Bei einer Regel mit `ref:refs/heads/main` läuft ein Workflow aus einem beliebigen Feature-Branch zwar, sein Subject passt aber nicht zur Regel. Bei einer Environment-Regel kann ein Feature-Branch das Environment ebenfalls angeben und liefert dann genau das erwartete Subject. Die eigentliche Grenze ist dort deshalb die Branch-Beschränkung des Environments.

## GitHub Secret gegen OIDC

Die ältere Variante speichert einen dauerhaften Zugang als **GitHub Secret**, also als verschlüsselt bei GitHub abgelegten Wert, den Workflows nutzen können. Typisch ist ein **Client Secret** einer App-Registrierung oder ein Publish Profile, eine Datei mit Zugangsdaten für das Deployment einer einzelnen Web-App.

Ein GitHub Secret lässt sich nach dem Speichern über Oberfläche und API nicht mehr im Klartext auslesen, und in Logs maskiert GitHub bekannte Werte. Das schützt aber nur vor dem versehentlichen Ausgeben. Ein Workflow, der den Wert kodiert oder an einen fremden Server schickt, kommt daran vorbei. Secrets auf Repository-Ebene stehen außerdem Workflows auf jedem Branch zur Verfügung. Wer Write hat, kann sie also mit einem Workflow im eigenen Branch abgreifen, an Branch-Schutz vorbei. Environment-Secrets lassen sich enger binden.

| Frage | GitHub Secret mit Client Secret | OIDC mit Federated Credential |
|---|---|---|
| Was liegt in GitHub? | Ein dauerhaft gültiges Passwort | Nur nicht geheime Kennungen |
| Was liegt in Entra ID? | Das Client Secret an der App-Registrierung | Die Regel mit Issuer, Subject, Audience |
| Abgeflossener Wert nutzbar | Von überall, bis zum Ablauf oder Widerruf | Nur kurz, im Rahmen der Rollenzuweisung |
| Bindung an Branch oder Environment | Nur über GitHub-Konfiguration | Zusätzlich von Entra ID geprüft |
| Rotation | Manuell, jemand muss den Ablauf im Blick haben | Entfällt |

OIDC macht GitHub Secrets nicht überflüssig. Für Dienste, die keine Federation anbieten, bleibt ein gespeicherter API-Schlüssel oft die einzige Option. Für den Weg von GitHub nach Azure ist OIDC aber der Normalfall, ein Client Secret die Ausnahme.

## App Service, Managed Identity und Key Vault

Das Ziel des Deployments ist hier **App Service**. Die App bekommt ihre Konfiguration über App Settings, die zur Laufzeit als Umgebungsvariablen erscheinen. Für Geheimnisse im Klartext sind sie kein guter Ort: Wer die Konfiguration über die Control Plane lesen darf, sieht die Werte, und das schließt „Website Contributor“ und damit die Deployment-Identität ein. Dafür gibt es **Key Vault**.

Die Laufzeit-Identität ist im Normalfall eine **Managed Identity** (siehe Kapitel 2, Identitäten und Tokens). Die App holt ihr Token über einen lokalen Endpunkt, den die Plattform nur innerhalb der Ressource bereitstellt, im Code meist über die SDK-Klasse `DefaultAzureCredential`.

```bash
az webapp identity assign --name meine-app --resource-group meine-rg
az role assignment create --assignee <principal-id> --role "Key Vault Secrets User" --scope "<key-vault-id>"
```

Die Rolle „Key Vault Secrets User“ ist eine Data-Plane-Rolle und erlaubt nur das Lesen von Geheimnissen. Sie setzt einen Key Vault voraus, dessen Zugriff über Azure RBAC gesteuert wird. Der Wert kommt dann auf einem von zwei Wegen in die App:

- **Direkt im Code**: Die App ruft Key Vault mit ihrer Managed Identity auf und liest das Geheimnis.
- **Key-Vault-Referenz**: Ein App Setting enthält statt des Werts einen Verweis wie `@Microsoft.KeyVault(VaultName=kv-meine-app;SecretName=api-key)`, den App Service mit der Managed Identity auflöst. Die App sieht eine normale Umgebungsvariable, die Control Plane nur den Verweis.

Noch besser ist es, wenn gar kein Geheimnis nötig ist. Viele Azure-Dienste akzeptieren direkt das Token der Managed Identity, und die Rollenzuweisung auf dem Zieldienst ist dann das einzige Recht. Auch Anwendungsberechtigungen für die Graph API lassen sich an den Service Principal einer Managed Identity hängen (siehe Kapitel 3, Die drei Rechtesysteme).

Eine Managed Identity löst allerdings nur das Problem gespeicherter Passwörter. Läuft fremder Code in der App, etwa durch eine Sicherheitslücke, kann auch er beim lokalen Endpunkt Tokens holen. Deshalb gilt Least Privilege auch für die Laufzeit-Identität: Rollenzuweisungen auf einzelne Ressourcen statt auf die ganze Subscription.

## Was liegt wo?

| Was | Wo es liegt | Wer herankommt |
|---|---|---|
| Quellcode, Workflows | GitHub-Repository; Kopien auf Laptops und kurz auf dem Runner | Repository-Rollen; Read genügt zum Lesen |
| GitHub Secrets | GitHub, verschlüsselt; zur Laufzeit im Runner | Setzen durch Admins; nutzbar für Workflows, praktisch also auch für Write |
| Federated Credential | An der App-Registrierung oder der Managed Identity | Besitzer der App-Registrierung, passende Entra- oder Azure-Rollen |
| Tokens der Deployment-Identität | Auf dem Runner, kurz gültig | Der Job und alles, was in ihm läuft |
| Tokens der Laufzeit-Identität | Im Speicher der App, von der Plattform ausgestellt | Die App und ihr Prozess |
| App Settings | App Service | Rollen mit Konfigurationsrechten auf der Web-App |
| Geheimnisse | Key Vault | Data-Plane-Rollen auf dem Key Vault |
| Rollenzuweisungen | Azure RBAC, am jeweiligen Scope (Azure) | Rollen wie Owner oder User Access Administrator |
| Logs | Workflow-Läufe bei GitHub; Activity Log für Control-Plane-Vorgänge in Azure; Sign-in-Log für Anmeldungen der Identitäten; Key-Vault-Protokolle, sofern aktiviert | Je nach System eigene Rollen |

Ein verdächtiges Deployment lässt sich damit aus zwei Richtungen verfolgen: im Workflow-Lauf bei GitHub und im Activity Log bei Azure. Kein System allein liefert das ganze Bild.

## Angriffsszenarien: Repo, Runner, Laptop

| Szenario | Was der Angreifer bekommt | Was er nicht bekommt | Erste Reaktion |
|---|---|---|---|
| Repository wird nur gelesen, etwa weil es öffentlich wurde | Code, Workflows, Kennungen und alles aus der Git-Historie, etwa eine alte `.env` | GitHub Secrets; kein OIDC-Token, weil er keinen Job im Repository starten kann | Alle je eingecheckten Geheimnisse rotieren; Löschen aus der Historie allein reicht nicht |
| Write-Zugang zum Repository, etwa über ein gestohlenes Konto | Workflows im eigenen Branch, damit Repository-Secrets | Bei OIDC mit passendem Subject keine Azure-Rechte, solange `main` geschützt und das Environment auf `main` beschränkt ist | Konto sperren, Workflow-Läufe prüfen, erreichbare Secrets rotieren |
| Runner wird im Job übernommen, etwa durch eine bösartige Abhängigkeit | Alles im Job: Quellcode, Secrets des Jobs, das aktuelle Azure-Token für dessen Gültigkeit | Bei OIDC kein dauerhaftes Anmeldegeheimnis der Deployment-Identität; eine ausgerollte Hintertür kann aber Zugang erhalten und die Rechte der Laufzeit-Identität nutzen | Ausgerollte Version kontrollieren, Abhängigkeit entfernen, Secrets des Jobs rotieren |
| Laptop einer Entwicklerin wird übernommen | Lokale Git-Zugänge, `az`-Token-Cache, `.env`-Dateien, also deren persönliche Rechte (siehe Kapitel 6, Geräte und Zugriffsregeln) | Bei Pflicht-Review keine alleinige Änderung an `main`; bei Least Privilege keinen breiten Azure-Zugriff | Sitzungen in Entra ID und GitHub widerrufen, Schlüssel tauschen, lokale Geheimnisse rotieren |

Daraus folgen einige konkrete Regeln:

- **Subject eng fassen.** Pro Umgebung eine Regel, gebunden an ein geschütztes Environment oder einen geschützten Branch.
- **Rollenzuweisung der Deployment-Identität auf die einzelne App begrenzen.** Ein Contributor auf der ganzen Subscription macht aus einem Runner-Vorfall einen Subscription-Vorfall.
- **Fremde Actions auf einen festen Commit-Hash pinnen.** Ein Versions-Tag kann nachträglich auf anderen Code zeigen, ein Hash nicht.
- **Self-hosted Runner nicht für öffentliche Repositories nutzen.** Sonst können Fremde über Pull Requests unter Umständen Code auf einem eigenen Rechner ausführen.
