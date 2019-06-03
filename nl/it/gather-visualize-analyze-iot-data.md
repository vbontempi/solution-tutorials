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

# Raccogli, visualizza e analizza i dati IoT
{: #gather-visualize-analyze-iot-data}
Questa esercitazione ti guida nella configurazione di un dispositivo IoT, raccogliendo dati in {{site.data.keyword.iot_short_notm}}, esplorando i dati e creando visualizzazioni e utilizzando quindi i servizi di machine learning avanzati per analizzare i dati e rilevare anomalie nei dati cronologici.
{:shortdesc}

## Obiettivi
{: #objectives}

* Configurare il simulatore IoT.
* Inviare i dati della raccolta a {{site.data.keyword.iot_short_notm}}.
* Creare le visualizzazioni.
* Analizzare i dati generati dal dispositivo e rilevare le anomalie.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Applicazione Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience) con il servizio Spark e {{site.data.keyword.cos_full_notm}}
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* I dispositivi inviano i dati dei sensori a {{site.data.keyword.iot_full}} utilizzando il protocollo MQTT
* I dati cronologici vengono esportati in un database {{site.data.keyword.cloudant_short_notm}}
* {{site.data.keyword.DSX_short}} estrae i dati da questo database
* I dati vengono analizzati e visualizzati tramite un notebook Jupyter

## Prima di iniziare
{: #prereqs}

[{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - esegui lo script per installare la cli ibmcloud e i plugin richiesti

## Crea la IoT Platform
{: #iot_starter}

Per iniziare, creerai il servizio Internet of Things Platform, l'hub che può gestire i dispositivi, stabilire connessioni in modo protetto e **raccoglie dati** e rendere i dati cronologici disponibili per le visualizzazioni e le applicazioni.

1. Vai a [**{{site.data.keyword.Bluemix_notm}} Catalog**](https://{DomainName}/catalog/) e seleziona [**Internet of Things Platform**](https://{DomainName}/catalog/services/internet-of-things-platform) nella sezione **Internet of Things**.
2. Immetti `IoT demo hub` come nome servizio e fai clic su **Create** e avvia (**Launch**) il dashboard.
3. Dal menu laterale, seleziona **Security > Connection Security** e scegli **TLS Optional** in **Default Rule** > **Security Level** e fai clic su **Save**.
4. Dal menu laterale, seleziona **Devices** > **Device Types** e **+ Add Device Type**.
5. Immetti `simulator` come nome (**Name**) e fai clic su **Next** e **Done**.
6. Fai quindi clic su **Register Devices**
7. Scegli `simulator` per **Select Existing Device Type** e immetti quindi `phone` per **Device ID**.
8. Fai clic su **Next** finché non viene visualizzata la schermata **Device Security** (nella scheda Security).
9. Immetti un valore per il token di autenticazione (**Authentication Token**), ad esempio `myauthtoken`, e fai clic su **Next**.
10. Dopo che hai fatto clic su **Done**, vengono visualizzate le tue informazioni di connessione. Tieni questa scheda aperta.

La IoT Platform è ora configurata per iniziare a ricevere dati. I dispositivi dovranno inviare i loro dati alla IoT Platform con il tipo dispositivo (Device Type), l'ID e il token specificati.

## Crea il simulatore del dispositivo
{: #confignodered}
Distribuirai quindi un'applicazione web Node.js e la visiterai sul tuo telefono, stabilendo così una connessione all'IoT Platform e inviando a essa i dati di orientamento e dell'accelerometro del dispositivo.

1. Clona il repository Github:
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. Apri il codice in un IDE a tua scelta e modifica i valori `name` e `host` nel file **manifest.yml** impostandoli su un valore univoco.
3. Esegui il push dell'applicazione a {{site.data.keyword.Bluemix_notm}}.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. Nel giro di pochi minuti, la tua applicazione verrà distribuita e dovresti vedere un URL simile a `<UNIQUE_NAME>.mybluemix.net`
5. Visita questo URL sul tuo telefono utilizzando un browser.
6. Immetti le informazioni di connessione dalla tua scheda IoT Dashboard in **Device Credentials** e fai clic su **Connect**.
7. Il tuo telefono inizierà a trasmettere i dati. Tornando alla scheda **IBM {{site.data.keyword.iot_short_notm}}**, controlla l'eventuale presenza di nuove voci nella sezione **Recent Events**.
  ![](images/solution16/recent_events_with_phone.png)

## Visualizza i dati dal vivo in {{site.data.keyword.iot_short_notm}}
{: #createcards}
Creerai quindi una tabella e delle schede per visualizzare i dati del dispositivo nel dashboard. Per ulteriori informazioni su tabelle e schede, vedi [Visualizzazione dei dati in tempo reale utilizzando le tabelle e le schede](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html).

### Crea una tabella
{: #createboard}

1. Apri il dashboard **IBM {{site.data.keyword.iot_short_notm}}**.
2. Seleziona **Boards** dal menu a sinistra e fai quindi clic su **Create New Board**.
3. Immetti un nome per la tabella, `Simulators` ad esempio, e fai clic su **Next** e quindi su **Submit**.  
4. Seleziona la scheda che hai appena creato per aprirla.

### Visualizza i dati del dispositivo
{: #cardtemp}
1. Fai clic su **Add New Card** e seleziona quindi il tipo di scheda **Line Chart**, che si trova nella sezione Devices.
2. Seleziona il tuo dispositivo dall'elenco e fai clic su **Next**.
3. Fai clic su **Connect new data set**.
4. Nella pagina Create Value Card, seleziona o immetti i seguenti valori e fai clic su **Next**.
   - Event: sensorData
   - Property: ob
   - Name: OrientationBeta
   - Type: Float
   - Min: -180
   - Max: 180
5. Nella pagina Card Preview, seleziona **L** per la dimensione del grafico a linee e fai clic su **Next** > **Submit**
6. La scheda compare sul dashboard e include un grafico a linee dei dati di temperatura dal vivo.
7. Utilizza il browser del tuo telefono mobile per avviare nuovamente il simulatore e inclina lentamente il tuo telefono in avanti e all'indietro.
8. Tornando alla scheda **IBM {{site.data.keyword.iot_short_notm}}**, dovresti vedere il grafico che viene aggiornato.
   ![](images/solution16/board.png)

## Archivia i dati cronologici in {{site.data.keyword.cloudant_short_notm}}
1. Vai al [catalogo **{{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e crea un nuovo [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) denominato `iot-db`.
2. In **Connections**:
   1. Crea la connessione (**Create connection**)
   1. Seleziona l'ubicazione, l'organizzazione e lo spazio Cloud Foundry dove deve essere creato un alias per il servizio {{site.data.keyword.cloudant_short_notm}}.
   1. Espandi il nome dello spazio nella tabella **Connection Location** e utilizza il pulsante **Connect** accanto a **iot demo hub** per creare un alias per il servizio {{site.data.keyword.cloudant_short_notm}} in tale spazio. 
   1. Connetti e riprepara l'applicazione.
3. Apri il dashboard **IBM {{site.data.keyword.iot_short_notm}}**.
4. Seleziona **Extensions** dal menu a sinistra e fai quindi clic su **Setup** in **Historical Data Storage**.
5. Seleziona la scheda `iot-db` {{site.data.keyword.cloudant_short_notm}}.
6. Immetti `devicedata` per **Database Name** e fai clic su **Done**.
7. Dovrebbe essere caricata una nuova finestra che richiede l'autorizzazione. Se non vedi questa finestra, disabilita il tuo blocco pop-up e aggiorna la pagina.

I tuoi dati del dispositivo sono ora salvati in {{site.data.keyword.cloudant_short_notm}}. Dopo qualche minuto, avvia il dashboard {{site.data.keyword.cloudant_short_notm}} per vedere i tuoi dati.

![](images/solution16/cloudant.png)

## Rileva le anomalie utilizzando Machine Learning
{: #data_experience}

In questa sezione, utilizzerai il notebook Jupyter disponibile nel servizio {{site.data.keyword.DSX_short}} per caricare i tuoi dati mobili cronologici e rilevare le anomalie utilizzando lo scarto Z. Lo *scarto Z* è un punteggio standard che indica il numero di deviazioni standard di un elemento dalla sua media.

### Crea un nuovo progetto
1. Vai al [**catalogo {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e, in **AI**, seleziona [**{{site.data.keyword.DSX_short}}**](https://{DomainName}/catalog/services/data-science-experience).
2. Crea (**Create**) il servizio e avviane il dashboard facendo clic su **Get Started**
3. Crea un progetto > Seleziona **Data Science** >, fai clic su **Create project** e immetti `Detect Anomaly` come nome (**Name**) del progetto.
4. Lascia la casella di spunta **Restrict who can be a collaborator** deselezionata poiché non ci sono dati riservati.
5. In **Define Storage**, fai clic su **Add** e scegli un servizio **Cloud Object Storage** esistente oppure creane uno nuovo (seleziona **Lite** plan > Create). Premi **Refresh** per visualizzare il servizio creato.
6. Fai clic su **Create**. Il tuo nuovo progetto viene aperto e puoi iniziare ad aggiungere risorse ad esso.

### Connessione a {{site.data.keyword.cloudant_short_notm}} per i dati

1. Fai clic su **Assets** > **+ Add to Project** > **Connection**  
2. Seleziona **iot-db** {{site.data.keyword.cloudant_short_notm}} dove sono archiviati i dati del dispositivo.
3. Verifica le credenziali (**Credentials**) e fai quindi clic su **Create**.

### Crea un servizio Apache Spark

1. Fai clic su **Services** nella barra di navigazione superiore > Compute Services.
2. Fai clic su **Add service**.
   1. Fai clic su **Add** in **Apache Spark**.
   1. Seleziona il piano **Lite**.
   1. Fai clic su **Create**. 
3. Seleziona un'organizzazione e uno spazio, modifica il nome del servizio se lo desideri e conferma (**Confirm**).
1. Vai al progetto `Detect Anomaly` mediante **Projects**.
1. In **Settings**, scorri a **Associated services**.
1. Fai clic su **Add service** e seleziona **Spark**.
1. Seleziona l'istanza **Apache Spark** creata in precedenza.

### Crea un notebook Jupyter (ipynb)
1. Fai clic su **+ Add to Project** e aggiungi un nuovo **notebook**.
2. Immetti `Anomaly-detection-notebook` per il nome (**Name**).
3. Immetti `https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb` in **Notebook URL**.
4. Seleziona il servizio **Apache Spark** associato in precedenza come runtime.
5. Crea **Notebook**. Imposta `Python 3.5 with Spark 2.1` come tuo Kernel. Controlla che il notebook venga creato con i metadati e il codice, ![Jupyter Notebook Watson Studio](images/solution16/jupyter_notebook_watson_studio.png)
   Per eseguire l'aggiornamento, **Kernel** > Change kernel. Per indicare come attendibile (**Trust** il notebook, **File** > Trust Notebook.
   {:tip}

### Esegui il notebook e rileva le anomalie   
1. Seleziona la cella che inizia con `!pip install --upgrade pixiedust,` e fai quindi clic su **Run** o **Ctrl + Invio** per eseguire il codice.
2. Dopo che l'installazione è stata completata, riavvia il kernel Spark facendo clic sull'icona **Restart Kernel**.
3. Nella cella di codice successiva, importa le tue credenziali {{site.data.keyword.cloudant_short_notm}} in tale cella completando la seguente procedura:
   * Fai clic su ![](images/solution16/data_icon.png)
   * Seleziona la scheda **Connections**.
   * Fai clic su **Insert to code**. Viene creato un dizionario denominato _credentials_1_ con le tue credenziali {{site.data.keyword.cloudant_short_notm}}. Se il nome non è specificato come _credentials_1_, rinomina il dizionario come `credentials_1`. `credentials_1` viene utilizzato nelle restanti celle.
4. Nella cella con il nome database (`dbName`) immetti il nome del database {{site.data.keyword.cloudant_short_notm}} che è l'origine dei dati, ad esempio *iotp_yourWatsonIoTProgId_DBName_Year-month-day*. Per visualizzare i dati di dispositivi differenti, modifica i valori di `deviceId` e `deviceType` di conseguenza.
   Puoi trovare il database esatto andando alla tua istanza **iot-db** {{site.data.keyword.cloudant_short_notm}} che hai creato in precedenza > Avvia il dashboard.
   {:tip}
5. Salva il notebook ed esegui ciascuna cella di codice una dopo l'altra oppure esegui tutto (**Cell** > Run All) e, entro la fine del notebook, dovresti vedere le anomalie per i dati dei movimenti del dispositivo (oa,ob e og).
   Puoi modificare l'intervallo di tempo da prendere in considerazione impostandolo sull'ora del giorno che desideri. Cerca i valori `start` e `end`.
   {:tip}
   ![DSX notebook Jupyter](images/solution16/anomaly_detection_watson_studio.png)
6. Insieme al rilevamento delle anomalie, i rilevamenti e i punti chiave da questa sezione sono
    * L'utilizzo di Spark per preparare i dati per la visualizzazione.
    * L'utilizzo di Pandas per la visualizzazione dei dati
    * I grafici a barre e gli istogrammi per i dati del dispositivo.
    * La correlazione tra due sensori mediante la matrice di correlazione.
    * Un box plot per ciascun sensore del dispositivo, prodotto con la funzione di grafico di Pandas.
    * I grafici di densità tramite KDE (Kernel density estimation).
    ![](images/solution16/density_plots_sensor_data.png)

## Rimuovi le risorse
{:removeresources}

1. Vai a [Resource List](https://{DomainName}/resources/) > e scegli l'ubicazione (Location), l'organizzazione (Org) e lo spazio (Space) dove hai creato l'applicazione e i servizi. In **Cloud Foundry Apps**, elimina l'applicazione Node.JS che hai creato in precedenza.
2. In **Services**, elimina i servizi Internet of Things Platform, Apache Spark, {{site.data.keyword.cloudant_short_notm}} e {{site.data.keyword.cos_full_notm}} rispettivi che hai creato per questa esercitazione.

## Contenuto correlato
{:related}

* [Crea, distribuisci, verifica e riaddestra un modello di machine learning predittivo](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* Panoramica di [IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html)
* Rilevamento delle anomalie [Jupyter Notebook](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb)
* [Descrizione dello scarto Z](https://en.wikipedia.org/wiki/Standard_score)
* Sviluppo di soluzioni IoT cognitive per il rilevamento delle anomalie utilizzando il deep learning - [la serie di 5 post](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/)
