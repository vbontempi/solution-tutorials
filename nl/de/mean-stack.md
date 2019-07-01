---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}


# Moderne Webanwendung mit dem MEAN-Stack erstellen
{: #mean-stack}

In diesem Lernprogramm erfahren Sie, wie Sie eine Webanwendung mithilfe des gängigen MEAN-Stacks erstellen können. Der Stack setzt sich aus einer **M**ongoDB-Instanz, einem **E**xpress Web-Framework, einem **A**ngular-Front-End-Framework und einer Node.js-Laufzeit zusammen. Sie lernen, wie Sie einen MEAN-Starter lokal ausführen, eine verwaltete Database as a Service-Instanz (DBasS) erstellen und verwenden, die App in {{site.data.keyword.cloud_notm}} implementieren und die Anwendung überwachen.  

## Lernziele

{: #objectives}

- Node.js-Starter-App lokal erstellen und ausführen
- Verwalteten Database as a Service (DBasS) verwenden
- Node.js-App in der Cloud bereitstellen
- MongoDB-Ressourcen skalieren
- Erfahren, wie die Anwendungsleistung überwacht werden kann

## Verwendete Services

{: #products}

In diesem Lernprogramm werden die folgenden {{site.data.keyword.Bluemix_notm}}-Services verwendet:

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**Achtung:** Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur

{:#architecture}

<p style="text-align: center;">

![Diagramm der Architektur](images/solution7/Architecture.png)</p>

1. Benutzer greifen auf die Anwendung über einen Web-Browser zu.
2. Die Node.js-App ruft Daten aus der {{site.data.keyword.composeForMongoDB}}-Datenbank ab.

## Vorbereitende Schritte

{: #prereqs}

1. [Installieren Sie Git](https://git-scm.com/).
2. [Installieren Sie die {{site.data.keyword.Bluemix_notm}}-CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli).


Führen Sie außerdem zum lokalen Entwickeln und Ausführen der Anwendung die folgenden Schritte aus:
1. [Installieren Sie Node.js und NPM](https://nodejs.org/).
2. [Installieren Sie MongoDB Community Edition und führen Sie sie aus](https://docs.mongodb.com/manual/administration/install-community/).

## MEAN-App lokal ausführen

{: #runapplocally}

In diesem Abschnitt führen Sie eine lokale MongoDB-Datenbank aus, klonen einen MEAN-Beispielcode und führen die Anwendung lokal aus, um die lokale MongoDB-Datenbank zu verwenden.

{: shortdesc}

1. Folgen Sie den [hier](https://docs.mongodb.com/manual/administration/install-community/) aufgeführten Anweisungen, um die MongoDB-Datenbank lokal zu installieren und auszuführen. Sobald die Installation abgeschlossen ist, überprüfen Sie mit dem nachfolgenden Befehl, dass der **mongod**-Server ausgeführt wird.
  ```sh
  mongo
  ```
  {: codeblock}

2. Klonen Sie den MEAN-Starter-Code.

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. Installieren Sie die erforderlichen Pakete.

  ```sh
  npm install
  ```
  {: codeblock}

4. Kopieren Sie Datei '.env.example' in .env. Bearbeiten Sie die benötigten Informationen; fügen Sie dabei mindestens Ihren eigenen geheimen Sitzungsschlüssel (SESSION_SECRET) hinzu.

5. Führen Sie 'node server.js' aus, um Ihre App zu starten.
  ```sh
  node server.js
  ```
  {: codeblock}

6. Greifen Sie auf Ihre Anwendung zu, erstellen Sie einen neuen Benutzer und melden Sie sich an.

## Instanz der MongoDB-Datenbank in der Cloud erstellen

{: #createdatabase}

In diesem Abschnitt erstellen Sie eine {{site.data.keyword.composeForMongoDB}}-Datenbank in der Cloud. {{site.data.keyword.composeForMongoDB}} ist ein Database as a Service, der in der Regel leichter zu konfigurieren ist und integrierte Sicherungs- und Skalierungsfunktionen bietet. Der [IBM cloud-Katalog](https://{DomainName}/catalog/?category=data) enthält viele verschiedene Typen von Datenbanken. Führen Sie die folgenden Schritte aus, um {{site.data.keyword.composeForMongoDB}} zu erstellen.

{: shortdesc}

1. Melden Sie sich über die Befehlszeile bei Ihrem {{site.data.keyword.cloud_notm}}-Konto an und wählen Sie Ihr {{site.data.keyword.cloud_notm}}-Konto als Ziel aus. 

  ```sh
  ibmcloud login
  ibmcloud target --cf
  ```
  {: codeblock}

  Weitere CLI-Befehle finden Sie [hier.](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

2. Erstellen Sie eine Instanz von {{site.data.keyword.composeForMongoDB}}. Dieser Schritt kann auch über die [Konsolenbenutzerschnittstelle](https://{DomainName}/catalog/services/compose-for-mongodb) erfolgen. Der Servicename muss **mean-starter-mongodb** lauten, da die Anwendung so konfiguriert ist, dass sie anhand des Namens nach diesem Service sucht.

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## App in der Cloud implementieren

{: #deployapp}

In diesem Abschnitt implementieren Sie die Node.js-App in der {{site.data.keyword.cloud_notm}}-Instanz, die die verwaltete MongoDB-Datenbank verwendet hat. Der Quellcode enthält eine Datei [**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml), die für die Verwendung des zuvor erstellten 'mongodb'-Service konfiguriert wurde. Die Anwendung verwendet die Umgebungsvariable VCAP_SERVICES, um auf die Compose MongoDB-Datenbankberechtigungsnachweise zuzugreifen. Dies kann in der Datei [server.js](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js) angezeigt werden. 

{: shortdesc}

1. Übertragen Sie den Code mit einer Push-Operation an die Cloud.

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. Sobald der Code mit einer Push-Operation übertragen wurde, sollten Sie die App in Ihrem Browser anzeigen können. Es wird ein beliebiger Hostname generiert, der in etwa wie folgt aussehen kann: `https://mean-random-name.mybluemix.net`. Sie können Ihre Anwendungs-URL über das Konsolendashboard oder die Befehlszeile abrufen.![Live App](images/solution7/live-app.png)


## MongoDB-Datenbankressourcen skalieren
{: #scaledatabase}

Wenn Ihr Service zusätzlichen Speicher benötigt oder Sie den Umfang des Speichers, der Ihrem Service zugeordnet ist, reduzieren möchten, können Sie dies durch das Skalieren von Ressourcen erreichen.

{: shortdesc}

1. Wechseln Sie im **Dashboard** der Konsole zum Abschnitt **Verbindungen** und klicken Sie auf die Datenbank **MongoDB-Instanz**.
2. Klicken Sie in der Anzeige mit den **Implementierungsdetails** auf die Option **Ressourcen skalieren**.
  ![](images/solution7/mongodb-scale-show.png)
3. Passen Sie den **Schieberegler** an, um den Speicher, der Ihrem {{site.data.keyword.composeForMongoDB}}-Datenbankservice zugeordnet ist, zu erhöhen oder zu reduzieren.
4. Klicken Sie auf **Implementierung skalieren**, um die Größenänderung des Speichers vorzunehmen und zur Dashboardübersicht zurückzukehren. Oben auf der Seite wird eine Nachricht darüber angezeigt, dass die Skalierung initiiert wurde, damit Sie wissen, dass die erneute Skalierung durchgeführt wird.
  ![](images/solution7/scaling-in-progress.png)


## Anwendungsleistung überwachen
{: #monitorapplication}

Wenn Sie den Status Ihrer Anwendung überprüfen möchten, können Sie den integrierten Availability Monitoring-Service zur Verfügbarkeitsüberwachung verwenden. Der Availability Monitoring-Service wird automatisch an Ihre Anwendungen in der Cloud angehängt. Der Availability Monitoring-Service führt synthetische Tests von Standorten auf der ganzen Welt rund um die Uhr aus, um Leistungsprobleme proaktiv zu erkennen und zu beheben, bevor diese sich auf die Benutzer Ihrer Sites auswirken. Führen Sie die folgenden Schritte aus, um zum Überwachungsdashboard zu gelangen.

{: shortdesc}

1. Wählen Sie im Konsolendashboard unter Ihrer Anwendung die Registerkarte **Überwachung** aus.
2. Klicken Sie auf **Alle Tests anzeigen**, um die Tests anzuzeigen.
   ![](images/solution7/alert_frequency.png)


## Zugehöriger Inhalt

{: #related}

- Quellcodeverwaltung und [Continuous Delivery](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops) einrichten
- Webanwendungen an [mehreren Standorten](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp) sichern
- [REST-APIs erstellen, sichern und verwalten](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis)
