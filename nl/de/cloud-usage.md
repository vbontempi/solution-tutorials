---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# {{site.data.keyword.cloud_notm}}-Services, -Ressourcen und -Nutzung überprüfen
{: #cloud-usage}
Mit dem zunehmenden Einsatz von Cloud-Technologien müssen IT- und Finanzmanager die Nutzung dieser Technologien im Kontext von Innovation und Kostenkontrolle verstehen lernen. Fragen wie "Welche Services werden von Teams verwendet?", "Wie viel kostet es, eine Lösung zu betreiben?" und "Wie kann ich große Datenmengen unter Kontrolle halten?" können durch die Untersuchung verfügbarer Daten beantwortet werden. In diesem Lernprogramm erfahren Sie, wie Sie solche Informationen durchsuchen und allgemeine nutzungsbezogene Fragen beantworten können.
{:shortdesc}

## Lernziele
{: #objectives}
* {{site.data.keyword.cloud_notm}}-Artefakte einzeln aufführen: Cloud Foundry-Apps und -Service, Identity and Access Management-Ressourcen und Infrastruktureinheiten
* {{site.data.keyword.cloud_notm}}-Artefakte mit Nutzungs- und Abrechnungsdaten verknüpfen
* Beziehungen zwischen {{site.data.keyword.cloud_notm}}-Artefakten und Entwicklungsteams definieren
* Anhand von Nutzungs- und Abrechnungsdaten Datasets für Abrechnungszwecke erstellen

## Architektur
{: #architecture}

![Architektur](images/solution38/Architecture.png)

## Vorbereitende Schritte
{: #prereqs}

* Installieren Sie die [{{site.data.keyword.cloud_notm}}-CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
* Installieren Sie [cURL](https://curl.haxx.se/).
* Installieren Sie [Node.js](https://nodejs.org/).
* Installieren Sie [json2csv](https://www.npmjs.com/package/json2csv) mithilfe des Befehls `npm install -g json2csv`.
* Installieren Sie [jq](https://stedolan.github.io/jq/).

## Hintergrund
{: #background}

Vor der Ausführung von Befehlen, mit denen die {{site.data.keyword.cloud_notm}}-Nutzung inventarisiert und detailliert aufgeführt werden kann, ist es hilfreich, über ein gewisses Hintergrundwissen zu den groben Nutzungskategorien und deren Funktion zu verfügen. Schlüsselbegriffe, die später im Lernprogramm verwendet werden, sind fett dargestellt. Eine hilfreiche Darstellung der unten stehenden Artefakte finden Sie in der [Dokumentation zur Kontoverwaltung](https://{DomainName}/docs/account?topic=account-overview#overview).

### Cloud Foundry
Cloud Foundry ist eine Open-Source-Platform as a Service (PaaS) in {{site.data.keyword.cloud_notm}}, mit der Sie Anwendungen und **Services** bereitstellen und skalieren können, ohne dabei Server verwalten zu müssen. Cloud Foundry organisiert Anwendungen und Services in Organisationen und Bereichen. Eine **Organisation** ist ein Entwicklungskonto, das ein oder mehrere Benutzer besitzen und verwenden können. Eine Organisation kann mehrere Bereiche enthalten. Jeder **Bereich** stellt Benutzern Zugriff auf eine gemeinsam genutzte Position für die Anwendungsentwicklung, -implementierung und -wartung zur Verfügung.

### Identity and Access Management
IBM Cloud Identity and Access Management (IAM) ermöglicht es Ihnen, Benutzer für beide Plattformservices sicher zu authentifizieren und den Zugriff auf Ressourcen konsistent über die {{site.data.keyword.cloud_notm}}-Plattform zu steuern. Neuere Angebote und migrierte Cloud Foundry-Services stehen als **Ressourcen** zur Verfügung, die von {{site.data.keyword.cloud_notm}} Identity and Access Management verwaltet werden. Die Ressourcen sind in **Ressourcengruppen** organisiert und bieten die Zugriffssteuerung über Richtlinien und Rollen.

### Infrastruktur
Die Infrastruktur umfasst eine Vielzahl von Optionen für die Datenverarbeitung: Bare Metal-Server, virtuelle Serverinstanzen und Kubernetes-Knoten. Jede dieser Option wird als **Gerät** in der Konsole angezeigt.

### Konto
Die oben genannten Artefakte sind einem **Konto** für die Rechnungsstellung zugeordnet. **Benutzer** werden zu dem Konto eingeladen und haben Zugriff auf die verschiedenen Ressourcen innerhalb des Kontos.

## Berechtigungen zuordnen
Um den Cloud-Bestand und die Nutzung anzuzeigen, müssen die entsprechenden Rollen vom Kontoadministrator zugewiesen werden. Wenn Sie der Kontoadministrator sind, fahren Sie mit dem nächsten Abschnitt fort.

1. Der Kontoadministrator sollte sich bei {{site.data.keyword.cloud_notm}} anmelden und auf die Seite für die [**Identity & Access-Benutzer**](https://{DomainName}/iam/#/users) zugreifen.
2. Der Accountadministrator kann Ihren Namen aus der Liste der Benutzer in dem Konto auswählen, um die entsprechenden Berechtigungen zuzuweisen.
3. Klicken Sie auf der Registerkarte **Zugriffsrichtlinien** auf die Schaltfläche **Zugriff zuordnen** und nehmen Sie die folgenden Änderungen vor.
   1. Wählen Sie die Kachel **Zugriff in einer Ressourcengruppe zuweisen** aus. Wählen Sie die **Ressourcengruppe(n)** aus, auf die der Zugriff gewährt werden soll, und wenden Sie die Rolle **Anzeigeberechtigter** an. Um den Vorgang abzuschließen, klicken Sie auf die Schaltfläche **Zuweisen**. Diese Rolle ist für die Befehle `resource groups` und `billing resource-group-usage` erforderlich.
   2. Wählen Sie die Kachel **Zugriff mit Cloud Foundry zuweisen** aus. Wählen Sie das Überlaufmenü neben jeder **Organisation** aus, auf die der Zugriff erteilt werden soll. Wählen Sie im Menü die Option **Organisationsrolle bearbeiten** aus. Wählen Sie **Abrechnungsmanager** in der Liste **Organisationsrollen** aus. Um den Vorgang abzuschließen, klicken Sie auf die Schaltfläche **Rolle speichern**. Diese Rolle ist für den Befehl `billing org-usage` erforderlich.
   3. Wählen Sie die Kachel **Zugriff auf Ressourcen zuweisen** aus. Wählen Sie im Dropdown-Menü **Services** die Option **Alle Services mit aktiviertem Identity and Access Management** aus. Überprüfen Sie die Rolle **Editor** unter **Zugriffsrollen für Plattform zuweisen**. Diese Rolle ist für den Befehl `resource tag-attach` erforderlich.

## Ressourcen mit dem Suchbefehl suchen
{: #search}

Wenn Entwicklungsteams damit beginnen, Cloud-Service zu verwenden, ist es für Manager von Vorteil zu wissen, welche Services implementiert wurden. Implementierungsinformationen helfen bei der Beantwortung von Fragen im Zusammenhang mit dem Innovations- und Servicemanagement:
- Welche servicebezogenen Qualifikationen können von Teams gemeinsam genutzt werden, um andere Projekte zu erweitern?
- Was sind die Gemeinsamkeiten von Teams, die vielleicht in einigen fehlen?
- Welche Teams verwenden einen Service, der einen kritischen Fix erfordert oder bald nicht mehr unterstützt wird?
- Wie können Teams ihre Serviceinstanzen überprüfen, um große Datenmengen unter Kontrolle zu halten?

Die Suche ist nicht auf Services und Ressourcen beschränkt. Sie können auch Cloud-Artefakte wie Cloud Foundry-Organisationen und -Bereiche, Ressourcengruppen, Ressourcenbindungen, Aliasnamen usw. abfragen. Weitere Beispiele finden Sie in der Dokumentation zu [ibmcloud resource search](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search).
{:tip}

1. Melden Sie sich bei {{site.data.keyword.cloud_notm}} über die Befehlszeile an und geben Sie Ihr Cloud Foundry-Konto als Ziel an. Weitere Informationen finden Sie in der [Einführung zur CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh
    ibmcloud login
    ```
    {: pre}

    ```sh
    ibmcloud target --cf
    ```
    {: pre}

2. Inventarisieren Sie alle im Konto verwendeten Cloud Foundry-Services.
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. Boolesche Filter können angewendet werden, um Suchvorgänge zu erweitern oder einzuschränken. Suchen Sie zum Beispiel sowohl Cloud Foundry-Services und -Apps als auch IAM-Ressourcen mithilfe der unten stehenden Abfrage.
    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. Um Teams zu benachrichtigen, die einen bestimmten Servicetyp nutzen, verwenden Sie bei der Abfrage den Servicenamen. Ersetzen Sie `<name>` durch Text, z. B. `weather` oder `cloudant`. Rufen Sie dann den Namen der Organisation ab, indem Sie `<Organization ID>` durch die **Organisations-ID** ersetzen.
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

Das Tagging und die Suche können zusammen verwendet werden, um eine angepasste Kennzeichnung von Ressourcen bereitzustellen. Dazu zählt Folgendes: Anhängen des Tags an die Ressource(n) und Durchführen einer Suche anhand des Tagnamens. Erstellen Sie einen Tag mit dem Namen 'env:tutorial'.

1. Hängen Sie den Tag an eine Ressource an. Sie können einen Ressourcen-CRN über die Benutzerschnittstelle oder mit `ibmcloud resource service-instance <name|id>` anfordern. Cloudressourcennamen (CRNs) identifizieren IBM Cloud-Ressourcen eindeutig.
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <ressourcen-crn>
   ```
   {: pre}
1. Suchen Sie mit der unten angegebenen Abfrage nach den Cloud-Artefakten, die mit einem bestimmten Tag übereinstimmen.
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

Durch die Kombination von erweiterten Suchabfragen mit einem unternehmensweit vereinbarten Tagging-Schema können Manager und Teamleiter Cloud-Apps, -Ressourcen und -Services leichter erkennen und entsprechende Maßnahmen ergreifen.

## Nutzung im Nutzungsdashboard untersuchen
{: #dashboard}

Sobald das Management die von den Teams verwendeten Services kennt, lautet die nächste häufig gestellte Frage: "Was kostet es, diese Services zu betreiben?" Am einfachsten lassen sich die Nutzung und Kosten über das Nutzungsdashboard von {{site.data.keyword.cloud_notm}} ermitteln.

1. Melden Sie sich bei {{site.data.keyword.cloud_notm}} an und greifen Sie auf das [Nutzungsdashboard des Kontos](https://{DomainName}/account/usage) zu.
2. Wählen Sie im Dropdown-Menü **Gruppen** eine Cloud Foundry-Organisation aus, um die Servicenutzung anzuzeigen.
3. Klicken Sie für ein bestimmtes **Serviceangebot** auf **Instanzen anzeigen**, um die erstellten Serviceinstanzen anzuzeigen.
4. Wählen Sie auf der folgenden Seite eine Instanz aus und klicken Sie auf **Instanz anzeigen**. Die daraufhin angezeigte Seite enthält Details zu der Instanz, wie Organisation, Bereich und Standort, sowie einzelne Positionen, aus denen sich die Gesamtkosten zusammensetzen.
5. Rufen Sie über die Breadcrumb nochmals das [Nutzungsdashboard](https://{DomainName}/account/usage) auf.
6. Ändern Sie im Dropdown-Menü **Gruppen** den Selektor in **Ressourcengruppen** und wählen Sie eine Gruppe aus, z. B. **default**.
7. Führen Sie eine ähnliche Überprüfung der verfügbaren Instanzen durch.

## Nutzung mit der Befehlszeile untersuchen

In diesem Abschnitt erfahren Sie, wie Sie die Nutzung mit der Befehlszeilenschnittstelle untersuchen.

1. Listen Sie alle Cloud Foundry-Organisationen auf, die Ihnen zur Verfügung stehen, und legen Sie eine Umgebungsvariable fest, um eine dieser Organisationen zum Testen zu speichern.
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<organisationsname>
    ```
    {: pre}

2. Gliedern Sie die Abrechnung und Nutzung für eine bestimmte Organisation mit dem Befehl `billing` auf. Geben Sie einen bestimmten Monat mit dem Flag `-d` mit einem Datum im Format JJJJ-MM an.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Führen Sie die gleiche Untersuchung für Ressourcen durch. Jedes Konto verfügt über eine Ressourcengruppe des Typs `default`. Ersetzen Sie den Wert `<group>` durch eine im ersten Befehl aufgeführte Ressourcengruppe.
    ```sh
    ibmcloud resource groups
    ```
    {: pre}

    ```sh
    export RESOURCE_GROUP=<gruppe>
    ```
    {: pre}

    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP
    ```
    {: pre}

4. Sie können sowohl Cloud Foundry-Services als auch IAM-Ressourcen mit dem Befehl `resource-instances-usage` anzeigen. Führen Sie abhängig von Ihrer Berechtigungsstufe die entsprechenden Befehle aus.
    - Wenn Sie der Kontoadministrator sind oder Ihnen die Administratorrolle für alle Identity and Access-aktivierten Services zugeordnet wurde, führen Sie einfach den Befehl `resource-instances-usage` aus.
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  Wenn Sie nicht der Kontoadministrator sind, können Sie die folgenden Befehle verwenden, da am Anfang des Lernprogramms die Rolle des Anzeigeberechtigen festgelegt wurde.
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. Verwenden Sie den Befehl `sl`, um Infrastruktureinheiten anzuzeigen. Verwenden Sie dann den Befehl `vs`, um {{site.data.keyword.virtualmachinesshort}} zu überprüfen.
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## Nutzungsdaten mit der Befehlszeile exportieren
{: #usagecmd}

Für einige Personen in Managementpositionen ist es wichtig, Daten häufig in eine andere Anwendung exportieren zu können. Ein gängiges Beispiel ist der Export von Nutzungsdaten in ein Arbeitsblatt. In diesem Abschnitt exportieren Sie Nutzungsdaten in das CSV-Format (Comma Separated Value), das mit den meisten Tabellenkalkulationstabellen kompatibel ist.

In diesem Abschnitt werden zwei Tools von Drittanbietern verwendet: `jq` und `json2csv`. Jeder der unten stehenden Befehle setzt sich aus drei Schritten zusammen: Abrufen von Nutzungsdaten als JSON, Syntaxanalyse des JSON und Formatierung des Ergebnisses als CSV. Das Abrufen und Konvertieren von JSON-Daten ist nicht auf `json2csv` beschränkt; es können auch andere Tools genutzt werden.

Verwenden Sie die Option `-p`, um Ergebnisse im Pretty-Print-Format auszugeben. Wenn die Druckqualität der Daten unzureichend ist, entfernen Sie das Argument `-p`, um die unbearbeiteten CSV-Daten zu drucken.
{:tip}

1. Exportieren Sie die Nutzungsdaten einer Ressourcengruppe mit den erwarteten Kosten für jeden Ressourcentyp.
    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. Führen Sie die Instanzen für jeden Ressourcentyp in der Ressourcengruppe einzeln auf.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. Befolgen Sie den gleichen Ansatz für eine Cloud Foundry-Organisation.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. Fügen Sie den Daten mithilfe einer erweiterten `jq`-Abfrage erwartete Kosten hinzu. Dadurch werden mehr Zeilen erstellt, da einige Ressourcentypen mehrere Kostenmetriken aufweisen.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. Verwenden Sie die gleiche `jq`-Abfrage, um auch Cloud Foundry-Ressourcen mit den zugehörigen Kosten aufzuführen.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. Exportieren Sie die Daten in eine Datei, indem Sie die Option `-p` entfernen und die Ausgabe in eine Datei leiten. Die resultierende CSV-Datei kann mit einem Tabellenkalkulationsprogramm geöffnet werden.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## Nutzungsdaten mit APIs exportieren
{: #usageapi}

Während die `billing`-Befehle hilfreich sind, ist es mühsam, über die Befehlszeilenschnittstelle ein Gesamtbild zusammenzustellen. In ähnlicher Weise bietet das Nutzungsdashboard eine Übersicht über Organisationen und Ressourcengruppen, nicht jedoch unbedingt zur Nutzung eines Teams oder Projekts. In diesem Abschnitt untersuchen Sie einen eher datenbezogenen Ansatz zum Abrufen von Nutzungsdaten für benutzerdefinierte Anforderungen.

1. Setzen Sie im Terminal die Umgebungsvariable auf `IBMCLOUD_TRACE=true`, um API-Anforderungen und -Antworten zu drucken.
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. Führen Sie den Befehl `billing org-usage` erneut aus, um die API-Aufrufe anzuzeigen. Beachten Sie, dass mehrere Hosts und API-Routen für diesen einzelnen Befehl verwendet werden.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Rufen Sie ein OAuth-Token ab und führen Sie einen der API-Aufrufe aus dem Befehl `billing org-usage` aus. Beachten Sie, dass einige APIs das UAA-Token verwenden, während andere vielleicht ein IAM-Token als Trägerautorisierung verwenden.
    ```sh
    export IAM_TOKEN=`ibmcloud iam oauth-tokens | head -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    export UAA_TOKEN=`ibmcloud iam oauth-tokens | tail -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    curl -H "Authorization: Bearer $UAA_TOKEN" https://mccp.us-south.cf.cloud.ibm.com/v2/organizations?q=name:$ORG_NAME
    ```
    {: pre}

4. Um Infrastruktur-APIs auszuführen, müssen Sie Umgebungsvariablen für Ihren **API-Benutzernamen** und **Authentifizierungsschlüssel** aus Ihrem [Benutzerprofil](https://control.softlayer.com/account/user/profile) abrufen und festlegen. Wenn Sie keinen API-Benutzernamen und Authentifizierungsschlüssel sehen, können Sie entsprechend einen im Menü **Aktionen** neben Ihrem Namen in der [Benutzerliste](https://control.softlayer.com/account/users) erstellen.
    ```sh
    export IAAS_USERNAME=<api-benutzername>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<authentifizierungsschlüssel>
    ```
    {: pre}

5. Rufen Sie den Gesamtwert der Abrechnung für die Infrastruktur und die Rechnungspositionen mithilfe der folgenden APIs ab. Ähnliche APIs sind [hier](https://softlayer.github.io/reference/services/SoftLayer_Account/) dokumentiert.
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. Inaktivieren Sie die Tracefunktion.
    ```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

Während der datengesteuerte Ansatz die meiste Flexibilität bei der Untersuchung der Nutzungsdaten bietet, würde eine detailliertere Erläuterung über den Rahmen dieses Lernprogramms hinausgehen. Das GitHub-Projekt [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample) wurde erstellt und kombiniert {{site.data.keyword.cloud_notm}}-Services mit Cloud Functions, um eine Beispielimplementierung bereitzustellen.

## Lernprogramm erweitern
{: #expand}

Verwenden Sie die folgenden Vorschläge, um Ihre Untersuchung auf Bestandsdaten und nutzungsbezogene Daten zu erweitern.

- Untersuchen Sie die Befehle `ibmcloud billing` mit der Option `--output json`. Dadurch werden zusätzliche Dateneigenschaften angezeigt, die nicht im Rahmen des Lernprogramms erläutert werden.
- Lesen Sie den Blogeintrag zur [Verwendung der {{site.data.keyword.cloud_notm}}-Befehlszeile zum Suchen nach Ressourcen](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/). Dort finden Sie weitere Beispiele zu `ibmcloud resource search` und Informationen darüber, welche Eigenschaften Sie in Ihren Abfragen verwenden können.
- Überprüfen Sie die [Infrastruktur-Konto-APIs](https://softlayer.github.io/reference/services/SoftLayer_Account/) auf zusätzliche APIs zur Untersuchung der Infrastrukturnutzung.
- Überprüfen Sie das [jq-Handbuch](https://stedolan.github.io/jq/manual/) auf erweiterte Abfragen, um Nutzungsdaten zusammenzufassen.
