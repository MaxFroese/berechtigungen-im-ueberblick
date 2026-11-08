# Glossar

Begriffe aus dem Buch [Berechtigungen im Überblick](README.md), alphabetisch.

**Abonnement**: Vertrag über eine Anzahl Lizenzen eines Produkts in Microsoft 365, kurz Abo, nicht zu verwechseln mit einer Subscription in Azure.

**Abrechnungsrichtlinie**: Verknüpfung einer Power-Platform-Umgebung mit einer Azure-Subscription für nutzungsbasierte Abrechnung (billing policy).

**Abrechnungsrolle**: Rolle wie Billing account owner auf einer Ebene der Abrechnungshierarchie, die über Kaufen, Bezahlen und Unterschreiben entscheidet und von Entra-Rollen getrennt ist.

**Abuse Monitoring**: Automatische Missbrauchsprüfung von Prompts und Ausgaben in Foundry, bei der markierte Inhalte gespeichert und von Menschen geprüft werden können.

**Access Token**: Token für genau einen Zieldienst, das Rechte in `scp` oder `roles` trägt.

**Admin Consent**: Zustimmung eines Admins für den ganzen Tenant.

**Admin-Consent-Workflow**: Einstellung in Entra ID, mit der Nutzer für eine App eine Freigabe beantragen, die benannte Prüfer erteilen oder ablehnen.

**Agent Builder**: Werkzeug in Microsoft 365 Copilot für einfache Declarative Agents ohne eigene Aktionen.

**Anwendungsberechtigung**: App-Recht ohne Nutzer, im Token als `roles`.

**App Service**: Azure-Dienst, der Web-Apps auf verwalteten Servern betreibt.

**App-Registrierung**: Globaler Steckbrief einer App mit Name, Redirect-URIs, angeforderten Rechten und Anmeldedaten.

**Application User**: Nutzerdatensatz in Dataverse, der an eine App-Registrierung oder Managed Identity gebunden ist und Sicherheitsrollen ohne Lizenz erhält.

**Audience**: Claim `aud`: der Dienst, für den ein Token ausgestellt ist.

**Audit-Log**: Protokoll administrativer Änderungen und relevanter Aktivitäten.

**Auftragsverarbeitung**: Verarbeitung personenbezogener Daten durch einen Dienstleister nach Weisung des Verantwortlichen, geregelt in einem AVV.

**Ausgabenlimit**: Einstellung einer Azure-Subscription mit Guthaben, die bei `On` die Ressourcen stoppt, sobald das Guthaben aufgebraucht ist.

**AVV**: Vertrag zur Auftragsverarbeitung nach Art. 28 DSGVO.

**Azure**: Cloud-Plattform für eigene Ressourcen wie Web-Apps, Datenbanken und Modelle.

**Azure RBAC**: Rechtesystem für Azure-Ressourcen aus Principal, Rolle und Scope.

**Berechtigungsobjekt**: Art eines SAP-Rechts mit Feldern wie Buchungskreis und Aktivität; eine Berechtigung ist eine konkrete Belegung dieser Felder.

**Bibliothek**: Dokumentablage innerhalb einer Site.

**Branch-Schutz**: Regeln, die direkte Änderungen an einem Branch einschränken.

**Break-Glass-Konto**: Notfallkonto mit hohen Rechten, das von üblichen Zugriffsregeln gezielt ausgenommen ist.

**Buchungskreis**: Rechtlich selbständige Einheit der Buchhaltung in SAP und häufiger Organisationswert in Berechtigungen.

**Budget**: Schwellenwert in Azure Cost Management, der Warnungen auslöst, aber nichts stoppt.

**Business Unit**: Sicherheitsgrenze in Dataverse als Baum unter einer Root; jede Person und jede Zeile gehört genau einer an.

**Business-Tarif**: Firmentarif eines KI-Anbieters unter gewerblichen Bedingungen, einschließlich API-Zugang.

**Canvas App**: App der Power Platform, deren Oberfläche frei gestaltet wird und die Daten über Connectors aus beliebigen Quellen holt.

**Client Secret**: Passwort einer App, das sich dauerhaft speichern lässt und deshalb abfließen kann.

**CLOUD Act**: US-Gesetz, nach dem US-Anbieter auch im Ausland gespeicherte Daten herausgeben müssen können.

**Cloud Connector**: SAP-Komponente im lokalen Netz, die die BTP mit internen SAP-Systemen verbindet und bei Principal Propagation kurzlebige Zertifikate erzeugt.

**Cloud Flow**: Flow, der in Microsofts Cloud läuft und automatisiert, sofort oder geplant ausgelöst wird.

**Compliance**: Von Intune gemeldeter Status, ob ein Gerät die Firmenregeln erfüllt.

**Conditional Access**: Regeln in Entra ID, die anhand von Nutzer, App, Gerät und Risiko über Anmeldungen entscheiden.

**Connector**: Baustein, über den ein Flow einen Dienst wie SharePoint oder Dataverse anspricht.

**Consent**: Zustimmung eines Nutzers oder Admins, dass eine App bestimmte Rechte erhält.

**Consumer-Tarif**: Privattarif eines KI-Anbieters unter Verbraucherbedingungen.

**Control Plane**: Verwaltungsebene einer Ressource: anlegen, konfigurieren, löschen.

**Copilot Credits**: Verbrauchseinheit, nach der Copilot Studio und Agenten nutzungsbasiert abgerechnet werden.

**CSP**: Cloud Solution Provider: Programm, in dem ein Partner Lizenzen und Azure-Verbrauch im eigenen Namen weiterverkauft und selbst abrechnet.

**Custom Connector**: Eigener Connector für eine API auf Basis einer OpenAPI-Beschreibung, nutzbar in Flows, Apps und Agenten.

**Custom Engine Agent**: Agent mit eigener Orchestrierung und eigenem Modell, für dessen Compliance der Betreiber verantwortlich ist.

**Data Plane**: Ebene der eigentlichen Inhalte einer Ressource oder eines Dienstes: Dateien, Geheimnisse, Modellaufrufe.

**Dataverse**: Datenbank der Power Platform, in der auch Flows als Datensätze liegen.

**Declarative Agent**: Agent, der die Orchestrierung und Modelle von Microsoft 365 Copilot nutzt und nur Anweisungen, Wissen und Aktionen ergänzt.

**Delegation**: Ausführung einer Abfrage einer Canvas App durch die Datenquelle selbst statt lokal auf einer begrenzten Zahl von Zeilen.

**Delegierte Berechtigung**: App-Recht im Namen eines Nutzers, wirksam als Schnittmenge aus App- und Nutzerrecht, im Token als `scp`.

**Deployment**: Bereitgestelltes Modell mit eigenem Namen an einer Foundry-Ressource.

**Deployment-Identität**: Identität, die eine neue Version ausrollt.

**Deployment-Typ**: Einstellung eines Deployments in Foundry, die festlegt, wo die Inferenz läuft: Global, Data Zone oder Standard.

**Desktop Flow**: Automatisierung von Bedienschritten auf einem Windows-Rechner (RPA), beaufsichtigt oder unbeaufsichtigt.

**DLP-Richtlinie**: Regel der Power Platform, die Connectors in geschäftlich, nicht geschäftlich und blockiert einteilt; nicht zu verwechseln mit DLP in Purview.

**DPA**: Microsoft Products and Services Data Protection Addendum: Datenschutzvertrag, der über die Product Terms für Microsofts Online-Dienste gilt.

**DSFA**: Datenschutz-Folgenabschätzung nach Art. 35 DSGVO: dokumentierte Risikoprüfung vor einer Verarbeitung mit hohem Risiko.

**Enterprise Application**: Portalname für den Service Principal einer App im Tenant.

**Entra ID**: Identitätsdienst des Tenants, der Nutzer, Gruppen und Apps kennt und Tokens ausstellt.

**Entra Join**: Gerät ist als Firmengerät direkt mit Entra ID verbunden.

**Entra-Gruppen-Team**: Team in Dataverse, dessen Mitglieder aus einer Gruppe in Entra ID abgeleitet werden und dem Sicherheitsrollen zugewiesen sind.

**Entra-Registrierung**: Gerät ist Entra ID bekannt, bleibt aber im Wesentlichen privat verwaltet.

**Entra-Rolle**: Administrative Rolle im Tenant, zum Beispiel Global Administrator.

**EU Data Boundary**: Microsofts Zusage, Kundendaten in EU und EFTA zu speichern und zu verarbeiten, mit dokumentierten Ausnahmen.

**Federated Credential**: Konkrete Vertrauensregel an einer Identität, die Issuer, Subject und Audience festlegt.

**Fiori**: Web-Oberfläche von SAP, deren Apps im Hintergrund OData-Services aufrufen.

**Firefighter**: Notfallzugang in SAP mit weitreichenden Rechten, befristet, begründet, protokolliert und nachträglich geprüft.

**Flex Routing**: Copilot-Einstellung, die bei Lastspitzen Inferenz außerhalb der EU Data Boundary erlaubt.

**Flow**: Automatisierter Ablauf in Power Automate aus einem Auslöser und Aktionen.

**Foundry**: Microsofts Azure-Dienst für Modelle und Agenten, früher Azure AI Foundry; Portal `ai.azure.com`.

**Foundry-Ressource**: Azure-Ressource vom Typ `Microsoft.CognitiveServices/accounts` mit eigenem Endpunkt, an der Deployments und Projekte hängen.

**Freigabelink**: Link, der Zugriff auf ein Element gewährt und je nach Art bestimmte Personen, die Organisation oder jeden umfasst.

**Funktionstrennung**: Regel, dass bestimmte Kombinationen von Rechten nicht bei derselben Person liegen dürfen (Segregation of Duties), etwa Lieferant anlegen und Zahlung freigeben.

**Gast**: Externes Konto, das per Einladung als eigenes Objekt im Tenant vertreten ist.

**GDAP**: Granular Delegated Admin Privileges: vom Kunden bestätigte, befristete Beziehung, über die ein Partner ausgewählte Entra-Rollen im Kunden-Tenant hält.

**Geschäftsgeheimnis**: Geheime Information mit wirtschaftlichem Wert, die durch angemessene Maßnahmen geschützt wird.

**GitHub Actions**: In GitHub eingebaute Automatisierung über Workflow-Dateien.

**GitHub Organization**: Gemeinsamer Firmenbereich auf GitHub mit eigenen Mitgliedern und Rechten.

**GitHub Secret**: Verschlüsselt bei GitHub gespeicherter Wert, den Workflows nutzen können.

**Graph API**: Gemeinsame REST-API `graph.microsoft.com` für Daten aus Microsoft 365 und Entra ID.

**IAS**: Identity Authentication Service der SAP Cloud Identity Services: Identitätsanbieter oder Vermittler vor Entra ID für SAP-Anwendungen.

**ID-Token**: Token, das der App sagt, wer sich angemeldet hat.

**Identity Provisioning**: Dienst der SAP Cloud Identity Services, der Benutzer und Gruppen in SAP-Systeme verteilt.

**Intune**: Dienst zur Geräteverwaltung und Compliance-Prüfung.

**IT-Dienstleister**: Externe Firma, oft Managed Service Provider (MSP) genannt, die Tenant, Geräte, Lizenzen und Support einer Firma betreibt, ohne über Inhalte und Zugriffe fachlich zu entscheiden.

**Key Vault**: Azure-Dienst für Geheimnisse, Schlüssel und Zertifikate.

**KI-Agent**: Programm wie Claude Code oder Codex, das mit einem Modell plant und selbst Dateien ändert und Befehle ausführt.

**Laufzeit-Identität**: Identität, mit der die laufende App auf andere Dienste zugreift.

**Lizenz**: Einzelner Platz aus einem Abonnement, der einem Konto zugewiesen wird und dort Dienste freischaltet, aber keine Rechte auf Daten gibt.

**Maker-Portal**: Bauwerkzeug der Power Platform wie `make.powerapps.com`, das immer in einer gewählten Umgebung arbeitet.

**Managed Environment**: Bündel von Verwaltungsfunktionen, das auf einer Umgebung eingeschaltet wird und Premium-Lizenzen für deren Nutzung voraussetzt.

**Managed Identity**: Von Azure verwaltete Identität einer Ressource, die ohne gespeichertes Secret auskommt.

**Mandant**: Dreistellige Nummer, die Benutzer und Geschäftsdaten innerhalb eines SAP-Systems trennt, während Code und Tabellendefinitionen geteilt sind; kein Tenant.

**MCA**: Microsoft Customer Agreement: Direktvertrag zwischen Microsoft und einem Käufer, an dem die Abrechnungshierarchie hängt.

**MCP**: Model Context Protocol: Standard, über den ein KI-Agent externe Werkzeuge und Datenquellen anspricht.

**Member**: Reguläres internes Nutzerkonto des Tenants.

**MFA**: Anmeldung mit mehr als einem Faktor.

**Microsoft 365**: Sammlung der Dienste für Zusammenarbeit wie SharePoint, OneDrive, Teams und Exchange Online.

**Microsoft-365-Gruppe**: Gruppe, an der gemeinsame Dienste wie Team, Site und Postfach hängen.

**Model-driven App**: App der Power Platform, deren Oberfläche aus dem Datenmodell in Dataverse erzeugt wird und deren Rechte aus Sicherheitsrollen kommen.

**MSAL**: Microsofts Bibliothek, mit der Apps Tokens holen und zwischenspeichern.

**OIDC**: Standard, mit dem GitHub dem Runner einen signierten Herkunftsnachweis ausstellt.

**On-Premises Data Gateway**: Dienst im lokalen Netz, über den Power Platform und Logic Apps ausgehend interne Systeme wie SAP erreichen.

**Orchestrierung**: Laufzeit eines Agenten, die entscheidet, welche Wissensquellen, Tools und Topics bei einer Anfrage benutzt werden.

**Oversharing**: Zustand, in dem mehr Personen Zugriff haben als fachlich gewollt.

**pac CLI**: Kommandozeile der Power Platform für Anmeldung, Solutions, Dataverse und Automatisierung.

**Personenbezogene Daten**: Alle Informationen, die sich auf eine identifizierte oder identifizierbare Person beziehen.

**PFCG-Rolle**: Rolle im SAP-Profilgenerator aus Menü und Berechtigungsdaten, aus der ein Profil erzeugt wird; als Einzel-, Sammel- oder abgeleitete Rolle.

**PIM**: Privileged Identity Management: Admin-Rechte werden nur bei Bedarf und befristet aktiviert.

**Pipeline (Power Platform)**: Eingebaute Auslieferung einer Solution als managed über festgelegte Stufen, auf Wunsch delegiert über einen Service Principal.

**Power Automate**: Automatisierungsdienst der Power Platform, in dem Flows laufen.

**Power Pages**: Baustein der Power Platform für Websites, deren Besucher Kontakte in Dataverse statt Konten im Tenant sind.

**Power Platform**: Microsofts Low-Code-Plattform, unter anderem mit Power Apps, Power Automate und Copilot Studio.

**Power Platform Admin Center**: Verwaltungsort für Umgebungen, Kapazität, DLP-Richtlinien und Abrechnungsrichtlinien.

**Premium-Connector**: Connector oder Baustein wie HTTP, Dataverse oder AI Builder, der eine eigene Power-Automate-Lizenz erfordert.

**Principal**: Oberbegriff für alles, dem Rechte zugewiesen werden können: Nutzer, Gruppe, Service Principal, Managed Identity.

**Principal Propagation**: Durchreichen der Identität einer angemeldeten Person bis in ein SAP-System, das sie einem eigenen SAP-Benutzer zuordnet; das SAP-Gegenstück zur delegierten Berechtigung.

**Product Terms**: Nutzungsbedingungen für Microsoft-Produkte, die unter dem Kundenvertrag gelten und das DPA einbeziehen.

**Projekt**: Arbeitsbereich für Agenten und zugehörige Inhalte als Unterressource einer Foundry-Ressource, mit eigener Managed Identity.

**Prompt Injection**: Anweisung in verarbeiteten Daten, die ein Modell fälschlich wie einen Auftrag befolgt.

**Publisher**: Herausgeber einer Solution, der das Präfix der technischen Namen aller neuen Komponenten festlegt.

**Rechnungsabschnitt**: Ebene unter dem Rechnungsprofil, die Kosten auf einer Rechnung gruppiert und der Abonnements und Subscriptions zugeordnet sind.

**Rechnungskonto**: Oberste Ebene der Abrechnung unter einer MCA, die den Vertrag und das Unterschriftsrecht trägt; Typ Business oder Individual.

**Rechnungsprofil**: Ebene unter dem Rechnungskonto, die eine Rechnung erzeugt und das Zahlungsmittel trägt.

**Refresh Token**: Länger gültiges Token, mit dem neue Access Tokens geholt werden.

**Region**: Rechenzentrumsstandort einer Ressource oder Umgebung, unabhängig vom Tenant.

**Resource Group**: Logischer Ordner für zusammengehörige Azure-Ressourcen.

**Ressource**: Einzelnes verwaltetes Objekt in Azure, etwa eine Web-App oder ein Key Vault.

**Role Collection**: Bündel von Rollen auf der SAP BTP, das Benutzern oder Gruppen eines Identitätsanbieters zugewiesen wird.

**Rollenzuweisung**: Verknüpfung aus Principal, Rolle und Scope (Azure).

**Runner**: Rechner, auf dem ein Workflow ausgeführt wird.

**Sandbox**: Vom Betriebssystem oder einer Laufzeitumgebung durchgesetzte Grenze dafür, welche Dateien und Netzziele ein Prozess erreicht.

**SAP BTP**: Business Technology Platform: SAPs Plattform für Integration, Erweiterungen, Daten, KI und Identität, gegliedert in Global Account und Subaccounts.

**SAP GRC**: SAP-Werkzeug für Anträge, Prüfung der Funktionstrennung, Notfallzugriff und Rezertifizierung in klassischen SAP-Systemen; für Cloud-Produkte ergänzt durch Cloud Identity Access Governance.

**Schicht**: Ebene im Stapel einer Komponente aus System, managed Solutions und unmanaged Anpassungen, von der zur Laufzeit die oberste gilt.

**Schlüssel**: Statisches Zugangsgeheimnis einer Foundry-Ressource, das die Data Plane ohne Azure RBAC öffnet und sich mit `disableLocalAuth` abschalten lässt.

**Scope (Azure)**: Ebene einer Azure-Rollenzuweisung, die nach unten vererbt wird: Management Group, Subscription, Resource Group, Ressource.

**Security Defaults**: Grundschutz-Regeln ohne Einzelkonfiguration, etwa Pflicht-MFA, typisch für kleine Tenants.

**Security Trimming**: Filterung von Such- und Abrufergebnissen auf das, was die anfragende Identität sehen darf.

**Service Plan**: Einzelner Dienstbaustein innerhalb einer SKU, etwa Exchange Online, der sich bei der Zuweisung abschalten lässt.

**Service Principal**: Lokale Vertretung einer App in einem Tenant, an der Zustimmungen und Rollen hängen.

**Sicherheitsgruppe**: Gruppe, die vor allem zum Zuweisen von Rechten dient.

**Sicherheitsrolle**: Rolle in Dataverse, die regelt, wer in einer Umgebung welche Daten und Bausteine sehen und ändern darf.

**SID**: Dreistellige Kennung eines SAP-Systems, etwa PRD.

**Sign-in-Log**: Protokoll der Anmeldungen in Entra ID.

**Site**: SharePoint-Bereich mit Bibliotheken, Listen und eigenen Rechten.

**Sites.Selected**: Anwendungs- oder delegierte Berechtigung, die zunächst nichts erlaubt und jede Site einzeln freigibt.

**SKU**: Technische Kennung eines kaufbaren Microsoft-Produkts, in der Graph API als `skuId` und `skuPartNumber`.

**Solution**: Paket aus Komponenten der Power Platform ohne Daten, unmanaged als Quelle und managed als Artefakt für Test und Produktion.

**Spaltensicherheit**: Schutz einzelner Spalten in Dataverse über Sicherheitsprofile, der nur innerhalb vorhandenen Zeilenzugriffs wirkt.

**Standard-Connector**: Connector wie SharePoint oder Outlook, der in den meisten Microsoft-365-Lizenzen enthalten ist.

**Subscription**: Abrechnungs- und Rechtegrenze in Azure, die an genau einen Tenant gebunden ist.

**Technischer Benutzer**: System- oder Kommunikationsbenutzer in SAP mit festen Zugangsdaten für Schnittstellen; das SAP-Gegenstück zur Anwendungsberechtigung.

**Tenant**: Abgeschlossener Bereich einer Organisation in der Microsoft-Cloud mit eigenen Identitäten, Daten und Regeln.

**Testversion**: Kostenloses, befristetes Abonnement eines Produkts, das sich bei eingeschalteter wiederkehrender Abrechnung am Ende in ein bezahltes umwandelt.

**Token**: Signierter, zeitlich begrenzter Ausweis.

**Topic**: Fest gebauter Gesprächsablauf in Copilot Studio.

**Transportauftrag**: Sammlung von Änderungen in SAP, die in der Entwicklung freigegeben und über einen festen Weg nach Test und Produktion importiert wird.

**Umgebung**: Abgegrenzter Container der Power Platform mit eigener Region, eigenen Rechten und optional eigener Dataverse-Datenbank.

**Umgebungsrolle**: Rolle in einer Umgebung ohne Dataverse, etwa `Environment Admin` oder `Environment Maker`.

**Umgebungsvariable**: Solution-Komponente mit einer Definition und einem eigenen Wert je Umgebung, auch als Verweis auf ein Geheimnis im Key Vault.

**Unterauftragsverarbeiter**: Dienstleister, den ein Auftragsverarbeiter einsetzt und den der Verantwortliche genehmigen muss.

**Verantwortlicher**: Stelle, die nach Art. 4 Nr. 7 DSGVO über Zwecke und Mittel einer Verarbeitung entscheidet.

**Verbindung**: Gespeicherte Anmeldung einer bestimmten Identität bei einem Connector; ein Flow handelt mit den Rechten ihrer Besitzerin.

**Verbindungsreferenz**: Platzhalter in einer Solution, dem beim Import eine Verbindung der Zielumgebung zugeordnet wird.

**Vererbung**: Standardregel, dass Elemente die Rechte ihres übergeordneten Elements übernehmen.

**Vertraulichkeitsvereinbarung**: Vertrag, der die Nutzung und Weitergabe vertraulicher Informationen begrenzt.

**Wiederkehrende Abrechnung**: Einstellung eines Abonnements, ob es am Laufzeitende automatisch verlängert oder aus einer Testversion gekauft wird.

**Workflow**: YAML-Datei in `.github/workflows`, die festlegt, wann welche Schritte laufen.

**Workload Identity Federation**: Vertrauensregel, mit der ein externes Token, etwa von GitHub, gegen ein Entra-Token getauscht wird, ohne dass ein Secret nötig ist.

**Zugriffsebene**: Reichweite eines Privilegs in einer Sicherheitsrolle: User, Business Unit, Parent: Child Business Units oder Organization.
