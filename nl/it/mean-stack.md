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


# Applicazione web moderna utilizzando lo stack MEAN
{: #mean-stack}

Questa esercitazione ti guida nella creazione di un'applicazione web utilizzando il diffuso stack MEAN. È composto da un DB **M**ongo, un framework web **E**xpress, un framework front end **A**ngular e un runtime Node.js. Imparerai come eseguire uno starter MEAN localmente, creare e utilizzare un DBaaS (database-as-a-service) gestito, distribuire l'applicazione a {{site.data.keyword.cloud_notm}} e monitorare l'applicazione.  

## Obiettivi

{: #objectives}

- Creare ed eseguire un'applicazione Node.js starter in locale.
- Creare un DBaaS (database-as-a-service) gestito.
- Distribuire l'applicazione Node.js al cloud.
- Ridimensionare le risorse MongoDB.
- Imparare come monitorare le prestazioni dell'applicazione.

## Servizi utilizzati

{: #products}

Questa esercitazione utilizza i seguenti servizi {{site.data.keyword.Bluemix_notm}}:

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**Attenzione:** questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura

{:#architecture}

<p style="text-align: center;">

![Diagramma dell'architettura](images/solution7/Architecture.png)</p>

1. L'utente accede all'applicazione utilizzando un browser web.
2. L'applicazione Node.js va al database {{site.data.keyword.composeForMongoDB}} per recuperare i dati.

## Prima di iniziare

{: #prereqs}

1. [Installa Git](https://git-scm.com/)
2. [Installa la CLI {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)


E per sviluppare ed eseguire l'applicazione in locale:
1. [Installa Node.js e NPM](https://nodejs.org/)
2. [Installa ed segui MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/)

## Esegui l'applicazione MEAN in locale

{: #runapplocally}

In questa sezione, eseguirai un database MongoDB locale, clonerai un codice di esempio MEAN ed eseguirai l'applicazione in locale per utilizzare il database MongoDB locale.

{: shortdesc}

1. Attieniti alle istruzioni [qui](https://docs.mongodb.com/manual/administration/install-community/) per installare ed eseguire il database MongoDB in loco. Una volta completata l'installazione, utilizza questo comando per confermare che il server **mongod** sia in esecuzione.
  ```sh
  mongo
  ```
  {: codeblock}

2. Clona il codice starter MEAN.

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. Installa i pacchetti richiesti. 

  ```sh
  npm install
  ```
  {: codeblock}

4. Copia il file .env.example in .env. Modifica le informazioni necessarie; come minimo, aggiungi il tuo SESSION_SECRET.

5. Esegui il nodo server.js per avviare la tua applicazione
  ```sh
  node server.js
  ```
  {: codeblock}

6. Accedi alla tua applicazione, crea un nuovo utente ed esegui l'accesso

## Crea un'istanza del database MongoDB nel cloud

{: #createdatabase}

In questa sezione, creerai un database {{site.data.keyword.composeForMongoDB}} nel cloud. {{site.data.keyword.composeForMongoDB}} è un DBaaS (database-as-a-service) che di norma è più facile da configurare e fornisce backup e ridimensionamento integrati. Puoi trovare molti tipi diversi di database nel [catalogo IBM Cloud](https://{DomainName}/catalog/?category=data).  Per creare {{site.data.keyword.composeForMongoDB}}, attieniti alla seguente procedura.

{: shortdesc}

1. Esegui l'accesso al tuo account {{site.data.keyword.cloud_notm}} tramite la riga di comando e indica come destinazione il tuo account {{site.data.keyword.cloud_notm}}. 

  ```sh
  ibmcloud login
  ibmcloud target --cf
  ```
  {: codeblock}

  Puoi trovare ulteriori comandi CLI [qui](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

2. Crea un'istanza di {{site.data.keyword.composeForMongoDB}}. Questa operazione può anche essere eseguita utilizzando l'[IU console](https://{DomainName}/catalog/services/compose-for-mongodb). Il servizio deve essere denominato **mean-starter-mongodb** poiché l'applicazione è configurata per cercare questo servizio in base a questo nome.

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## Distribuisci l'applicazione al cloud

{: #deployapp}

In questa sezione, distribuirai l'applicazione node.js al {{site.data.keyword.cloud_notm}} che ha utilizzato il database MongoDB gestito. Il codice sorgente contiene un file [**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml) che è stato configurato per utilizzare il servizio "mongodb" creato in precedenza. L'applicazione utilizza la variabile di ambiente VCAP_SERVICES per accedere alle credenziali del database Compose MongoDB. Questo valore può essere visualizzato nel [file server.js](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js). 

{: shortdesc}

1. Esegui il push del codice al cloud.

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. Dopo che è stato eseguito il push del codice, dovresti essere in grado di visualizzare l'applicazione nel tuo browser. È stato generato un nome host casuale che può essere simile a: `https://mean-random-name.mybluemix.net`. Puoi ottenere il tuo URL dell'applicazione dal dashboard della console o dalla riga di comando. ![Live App](images/solution7/live-app.png)


## Ridimensionamento delle risorse di database MongoDB
{: #scaledatabase}

Se il tuo servizio necessita ulteriore memoria o desideri ridurre la quantità di memoria assegnata al tuo servizio, puoi eseguire queste operazioni per ridimensionare le risorse.

{: shortdesc}

1. Utilizzando il **dashboard** della console, vai alla sezione **connections** e fai clic sul database **MongoDB instance**.
2. Nel pannello **deployment details**, fai clic sull'opzione **scale resources**.
  ![](images/solution7/mongodb-scale-show.png)
3. Regola il **dispositivo di scorrimento** per aumentare o ridurre l'archiviazione assegnata al tuo servizio database {{site.data.keyword.composeForMongoDB}}.
4. Fai clic su **Scale Deployment** per attivare il ridimensionamento e ritornare alla panoramica del dashboard. Nella parte superiore della pagina viene visualizzato un messaggio 'Scaling initiated' per informarti che il ridimensionamento è in corso.
  ![](images/solution7/scaling-in-progress.png)


## Monitora le prestazioni dell'applicazione
{: #monitorapplication}

Per controllare l'integrità della tua applicazione, puoi utilizzare il servizio Availability Monitoring integrato. Il servizio Availability Monitoring viene collegato automaticamente alle tue applicazioni nel cloud. Il servizio availability Monitoring esegue dei test sintetici dalle ubicazioni in tutto il mondo e ininterrottamente per rilevare e correggere in modo proattivo i problemi di prestazioni prima che abbiano delle ripercussioni sui tuoi visitatori. Attieniti alla procedura di seguito indicata per ottenere il dashboard di monitoraggio.

{: shortdesc}

1. Utilizzando il dashboard della console, nella tua applicazione seleziona la scheda **Monitoring**.
2. Fai clic su **View All Tests** per visualizzare i test.
   ![](images/solution7/alert_frequency.png)


## Contenuto correlato

{: #related}

- Configura il controllo origine e la [fornitura continua](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops).
- Proteggi l'applicazione web in [più ubicazioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp).
- Crea, proteggi e gestisci le [API REST](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis).
