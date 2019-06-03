---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: #ios data-hd-operatingsystem="ios"}
{:android: #android data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Crea, proteggi e gestisci le API REST
{: #create-manage-secure-apis}

Questa esercitazione dimostra come creare le API REST utilizzando il framework API Node.js LoopBack. Con Loopback puoi creare rapidamente API REST che connettono dispositivi e browser a dati e servizi. Aggiungerai inoltre la gestione, la visibilità, la sicurezza e la limitazione della frequenza alle tue API utilizzando {{site.data.keyword.apiconnect_long}}.
{:shortdesc}

## Obiettivi

* Creare un'API REST con poca o nessuna codifica
* Pubblicare la tua API su {{site.data.keyword.Bluemix_notm}} per raggiungere gli sviluppatori
* Portare le API esistenti in {{site.data.keyword.apiconnect_short}}
* Esporre e controllare in modo sicuro l'accesso ai system of record

## Servizi utilizzati

Questa esercitazione utilizza i seguenti runtime e servizi:

* [Loopback](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* Applicazione Cloud Foundry [SDK for Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs)

## Architettura

![Architettura](images/solution13/Architecture.png)

1. Lo sviluppatore definisce l'API RESTful
2. Lo sviluppatore pubblica l'API su {{site.data.keyword.apiconnect_long}}
3. Gli utenti e le applicazioni utilizzano l'API

## Prima di iniziare

* Scarica e installa [Node.js](https://nodejs.org/en/download/) versione 6.x (utilizza [nvm](https://github.com/creationix/nvm) o simile se hai già installato una versione più recente di Node.js)

## Crea un'API REST in Node.js

{: #create_api}
In questa sezione, creerai un'API in Node.js utilizzando [LoopBack](https://loopback.io/doc/index.html). LoopBack è un framework Node.js open-source altamente estensibile che ti consente di creare API REST end-to-end dinamiche con codifica minima o nulla.

### Crea l'applicazione

1. Installa lo strumento della riga di comando {{site.data.keyword.apiconnect_short}}. Se riscontri problemi durante l'installazione di {{site.data.keyword.apiconnect_short}}, utilizza `sudo` prima del comando o segui le istruzioni riportate [qui](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit).
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. Immetti il seguente comando per creare l'applicazione.
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. Premi `Invio` per utilizzare **entries-api** come **nome della tua applicazione**.
4. Premi `Invio` per utilizzare **entries-api** come **directory per contenere il progetto**.
5. Scegli **3.x (current)** come **versione di LoopBack**.
6. Scegli **empty-server** come **tipo di applicazione**.

![Scaffolding di APIC Loopback](images/solution13/apic_loopback.png)

### Aggiungi un'origine dati

Le origini dati rappresentano i sistemi di backend come database, API REST esterne, servizi web SOAP e servizi di archiviazione. In genere, le origini dati forniscono funzioni di creazione, recupero, aggiornamento ed eliminazione (CRUD). Sebbene Loopback supporti molti tipi di [origini dati](http://loopback.io/doc/en/lb3/Connectors-reference.html), per semplicità utilizzerai un archivio dati in memoria con la tua API.

1. Passa alla directory del nuovo progetto e avvia API Designer.
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. Dalla navigazione, fai clic su **Data Sources**. Quindi, fai clic sul pulsante **Add**. Si aprirà una finestra di dialogo **New LoopBack Data Source**.
3. Immetti `entriesDS` nel campo di testo **Name** e fai clic sul pulsante **New**.
4. Seleziona **In-memory db** dalla casella combinata **Connector**.
5. Fai clic su **All Data Sources** in alto a sinistra. L'origine dati verrà visualizzata nell'elenco delle origini dati.

   L'editor aggiorna automaticamente il file server/datasources.json con le impostazioni per la nuova origine dati.
   {:tip}

![Origini dati API Designer](images/solution13/datastore.png)

### Aggiungi un modello

Un modello è un oggetto JavaScript con entrambe le API Node e REST che rappresentano i dati nei sistemi di backend. I modelli sono connessi a questi sistemi tramite le origini dati. In questa sezione, definirai la struttura dei tuoi dati e la connetterai all'origine dati creata in precedenza.

1. Dalla navigazione, fai clic su **Models**. Quindi, fai clic sul pulsante **Add**. Si aprirà una finestra di dialogo **New LoopBack Model**.
2. Immetti `entry` nel campo di testo **Name** e fai clic sul pulsante **New**.
3. Seleziona **entriesDS** dalla casella combinata **Data Source**.
4. Dalla scheda **Properties**, aggiungi le seguenti proprietà utilizzando l'icona **Add property**.
    1. Immetti `name` per **Property Name** e seleziona **string** dalla casella combinata **Type**.
    2. Immetti `email` per **Property Name** e seleziona **string** dalla casella combinata **Type**.
    3. Immetti `comment` per **Property Name** e seleziona **string** dalla casella combinata **Type**.
5. Fai clic sull'icona **Save** nel lato superiore destro per salvare il modello.

![Generatore di modelli](images/solution13/models.png)

## Verifica la tua applicazione LoopBack

In questa sezione, avvierai un'istanza locale della tua applicazione Loopback e verificherai l'API inserendo ed eseguendo query di dati utilizzando l'API Designer. Devi utilizzare lo strumento Explore di API Designer per verificare gli endpoint REST nel tuo browser in quanto include le intestazioni di sicurezza corrette e altri parametri di richiesta.

1. Avvia il server locale facendo clic sull'icona **Start** nell'angolo inferiore sinistro e attendi il messaggio **Running**.
2. Dal banner, fai clic sul link **Explore** per visualizzare lo strumento Explore di API Designer. La barra laterale mostra le operazioni REST disponibili per i modelli LoopBack nell'API.
3. Fai clic sull'operazione **entry.create** nel riquadro di sinistra per visualizzare l'endpoint. Il riquadro centrale visualizza informazioni di riepilogo sull'endpoint, inclusi i relativi parametri, sicurezza, dati di istanza del modello e codici di risposta. Il riquadro di destra fornisce il codice di esempio per chiamare l'endpoint utilizzando il comando cURL e i linguaggi come Ruby, Python, Java e Node.
4. Nel riquadro di destra, fai clic sul link **Try it**. Scorri verso il basso fino a **Parameters** e immetti quanto segue nell'area di testo **data**:
    ```javascript
    {
      "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. Fai clic sul pulsante **Call operation**.
  ![Verifica di un'API da API Designer](images/solution13/data_entry_1.png)
6. Conferma che la richiesta POST abbia avuto esito positivo controllando il campo **Response Code: 200 OK**. Se visualizzi un messaggio di errore dovuto a un certificato non attendibile per l'host locale, fai clic sul link fornito nel messaggio di errore nello strumento Explore di API Designer per accettare il certificato e procedi con il richiamo delle operazioni nel tuo browser web. La procedura esatta dipende da quale browser web stai utilizzando.
7. Aggiungi un'altra voce utilizzando cURL. Conferma che la porta corrisponda alla porta della tua applicazione.
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. Fai clic sull'operazione **entry.find**, quindi sul link **Try It** e sul pulsante **Call operation**. La risposta visualizza tutte le voci. Dovresti visualizzare il JSON per **Jane Doe** e **John Doe**.
  ![entry_find](images/solution13/find_response.png)

Puoi anche avviare manualmente l'applicazione Loopback immettendo il comando `npm start` dalla directory `entries-api`. Le tue API REST saranno disponibili all'indirizzo [http://localhost:3000/api/entries](http://localhost:3000/api/entries); tuttavia, non saranno gestite e protette da {{site.data.keyword.apiconnect_short}}.
{:tip}

## Crea il servizio {{site.data.keyword.apiconnect_short}}

Per prepararti per i passi successivi, creerai un servizio **{{site.data.keyword.apiconnect_short}}** su {{site.data.keyword.Bluemix_notm}}. {{site.data.keyword.apiconnect_short}} funge da gateway per la tua API e fornisce anche funzioni di gestione, sicurezza e limiti di frequenza.

1. Avvia l'[elenco delle risorse](https://{DomainName}/resources) {{site.data.keyword.Bluemix_notm}}.
2. Vai a **Catalog > Integration > {{site.data.keyword.apiconnect_short}}** e fai clic sul pulsante **Create**.

## Pubblica un'API su {{site.data.keyword.Bluemix_notm}}

{: #publish}
Utilizzerai l'API Designer per distribuire la tua applicazione in {{site.data.keyword.Bluemix_notm}} come applicazione Cloud Foundry e pubblicherai inoltre la tua definizione API su **{{site.data.keyword.apiconnect_short}}**. L'API Designer è il tuo toolkit locale. Se lo chiudi, riavvialo con `apic edit` dalla directory del progetto.

L'applicazione può anche essere distribuita manualmente utilizzando il comando `ibmcloud cf push`; tuttavia, non sarà protetta. Per [importare l'API](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing) in {{site.data.keyword.apiconnect_short}}, utilizza il file di definizione OpenAPI disponibile nella cartella `definitions`. La distribuzione mediante API Designer protegge l'applicazione e importa automaticamente la definizione.
{:tip}

1. Torna all'API Designer e fai clic sul link **Publish** nel banner. Quindi, fai clic su **Add and Manage Targets > Add IBM Bluemix target**.
2. Seleziona la regione (**Region**) e l'organizzazione (**Organization**) in cui vuoi pubblicare.
3. Seleziona il catalogo **Sandbox** e fai clic su **Next**.
4. Immetti `entries-api-application` nel campo di testo **Type a new application name** e fai clic sull'icona **+**.
5. Fai clic su **entries-api-application** nell'elenco e quindi su **Save**.
6. Fai clic sull'icona di **Menu** a tre linee nell'angolo superiore sinistro del banner. Quindi, fai clic su **Projects** e sulla voce di elenco **entries-api**.
7. Nell'interfaccia utente di API Designer, fai clic sui link **APIs > entries-api > Assemble**.
8. Nell'editor di assemblaggio, fai clic sull'icona **Filter policies**.
9. Seleziona **DataPower Gateway policies** e fai clic su **Save**.
10. Fai clic su **Publish** nella barra superiore e seleziona la tua destinazione. Seleziona **Publish application** e Stage or Publish Products > seleziona **Specific products** > **entries-api**.
11. Fai clic su **Publish** e attendi che l'applicazione finisca la pubblicazione.
    ![Finestra di dialogo di pubblicazione](images/solution13/publish.png)

    Un'applicazione contiene i modelli Loopback, le origini dati e il codice relativi alla tua API. Un prodotto ti consente di dichiarare il modo in cui un'API viene resa disponibile agli sviluppatori.
    {:tip}

L'applicazione API è ora pubblicata su {{site.data.keyword.Bluemix_notm}} come applicazione Cloud Foundry. Puoi vederla controllando le applicazioni Cloud Foundry nell'[elenco delle risorse](https://{DomainName}/resources) {{site.data.keyword.Bluemix_notm}}, ma l'accesso diretto tramite l'URL non è possibile in quanto l'applicazione è protetta. La prossima sezione mostrerà come è possibile accedere alle API gestite.

## Gateway API

Fino ad ora, hai progettato e verificato la tua API in locale. In questa sezione, utilizzerai {{site.data.keyword.apiconnect_short}} per verificare la tua API distribuita su {{site.data.keyword.Bluemix_notm}}.

1. Avvia l'[elenco delle risorse](https://{DomainName}/resources) {{site.data.keyword.Bluemix_notm}}.
2. Trova e seleziona il tuo servizio **{{site.data.keyword.apiconnect_short}}** sotto **Cloud Foundry Services**.
3. Fai clic sul menu **Explore** e quindi sul link **Sandbox**.
4. Fai clic sull'operazione **entry.create**.
5. Nel riquadro di destra, fai clic su **Try it**. Scorri verso il basso fino a **Parameters** e immetti quanto segue nell'area di testo **data**.
    ```javascript
    {
      "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. Fai clic sul pulsante **Call operation**. Dovresti visualizzare una risposta **Code: 200** che indica l'esito positivo.

![gateway](images/solution13/gateway.png)

L'URL della tua API gestita e sicura viene visualizzato accanto a ciascuna operazione e dovrebbe essere simile a `https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries`.
{: tip}

## Limitazione della frequenza

Configurando i limiti di frequenza puoi gestire il traffico di rete delle tue API e per operazioni specifiche al loro interno. Un limite di frequenza è il numero massimo di chiamate consentite in un particolare intervallo di tempo.

1. Torna all'API Designer e fai clic su **Products > entries-api**.
2. Seleziona **Default Plan** sulla sinistra.
3. Espandi **Default Plan** e scorri verso il basso fino al campo **Rate limits**.
4. Imposta i campi su **10** chiamate / **1** **minuto**.
5. Seleziona **Enforce hard limit** e fai clic sull'icona **Save**.
  ![Pagina dei limiti di frequenza](images/solution13/rate_limit.png)
6. Segui i passi riportati nella sezione [Pubblica un'API su {{site.data.keyword.Bluemix_notm}}](#publish) per pubblicare di nuovo la tua API.

La tua API è ora limitata a 10 richieste al minuto. Utilizza la funzione **Try it** per provare il limite. Consulta ulteriori informazioni sulla [Configurazione dei limiti di frequenza](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits) o esplora l'API Designer per visualizzare tutte le funzioni di gestione disponibili.

## Espandi l'esercitazione

Congratulazioni, hai creato un'API gestita e sicura. Di seguito vengono forniti ulteriori suggerimenti per migliorare la tua API.

* Aggiungi la persistenza dei dati utilizzando il connettore LoopBack [{{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant)
* Utilizza l'API Designer per [visualizzare ulteriori impostazioni](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0) per gestire la tua API
* Esamina le **Analisi** e le **Visualizzazioni** API [disponibili](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics) in {{site.data.keyword.apiconnect_short}}

## Contenuto correlato

* [Documentazione di Loopback](https://loopback.io/doc/index.html)
* [Introduzione a {{site.data.keyword.apiconnect_long}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
