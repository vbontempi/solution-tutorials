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

# Log di big data con Streaming Analytics e SQL
{: #big-data-log-analytics}

In questa esercitazione, creerai una pipeline di analisi dei log progettata per raccogliere, archiviare e analizzare i record di log per supportare i requisiti normativi o facilitare il rilevamento delle informazioni. Questa soluzione sfrutta diversi servizi disponibili in {{site.data.keyword.cloud_notm}}: {{site.data.keyword.messagehub}}, {{site.data.keyword.cos_short}}, SQL Query e {{site.data.keyword.streaminganalyticsshort}}. Un programma ti assisterà simulando la trasmissione dei messaggi di log del server web da un file statico a {{site.data.keyword.messagehub}}.

Con {{site.data.keyword.messagehub}}, la pipeline viene ridimensionata per ricevere milioni di record di log da vari produttori. Applicando {{site.data.keyword.streaminganalyticsshort}}, i dati di log possono essere esaminati in tempo reale per integrare i processi di business. I messaggi di log possono anche essere facilmente reindirizzati all'archiviazione a lungo termine utilizzando {{site.data.keyword.cos_short}} in cui sviluppatori, personale di supporto e revisori possono lavorare direttamente con i dati utilizzando SQL Query.

Sebbene questa esercitazione si concentri sull'analisi dei log, è applicabile ad altri scenari: i dispositivi IoT limitati all'archiviazione possono analogamente inviare in streaming i messaggi a {{site.data.keyword.cos_short}} o i professionisti del marketing possono segmentare e analizzare gli eventi dei clienti attraverso le proprietà digitali con SQL Query.
{:shortdesc}

## Obiettivi

{: #objectives}

* Comprendere la messaggistica di pubblicazione-sottoscrizione di Apache Kafka
* Archiviare i dati di log per i requisiti di controllo e conformità
* Monitorare i log per creare processi di gestione delle eccezioni
* Condurre analisi forensi e statistiche sui dati di log

## Servizi utilizzati

{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura

{: #architecture}

<p style="text-align: center;">

  ![Architettura](images/solution31/Architecture.png)
</p>

1. L'applicazione genera eventi di log in {{site.data.keyword.messagehub}}
2. L'evento di log viene intercettato e analizzato da {{site.data.keyword.streaminganalyticsshort}}
3. L'evento di log viene aggiunto a un file CSV che si trova in {{site.data.keyword.cos_short}}
4. Il revisore o il personale di supporto emette un lavoro SQL
5. SQL Query viene eseguito sul file di log in {{site.data.keyword.cos_short}}
6. La serie di risultati viene archiviata in {{site.data.keyword.cos_short}} e consegnata al revisore e al personale di supporto

## Prima di iniziare

{: #prereqs}

* [Installa Git](https://git-scm.com/)
* [Installa la CLI {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* [Installa Node.js](https://nodejs.org)
* [Scarica il client Kafka 0.10.2.X](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)

## Crea i servizi

{: #setup}

In questa sezione, creerai i servizi richiesti per eseguire l'analisi degli eventi di log generati dalle tue applicazioni.

Questa sezione utilizza la riga di comando per creare le istanze del servizio. In alternativa, puoi eseguire la stessa operazione dalla pagina del servizio nel catalogo utilizzando i link forniti.
{: tip}

1. Accedi a {{site.data.keyword.cloud_notm}} tramite la riga di comando e specifica il tuo account Cloud Foundry. Vedi [Introduzione alla CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Crea un'istanza Lite di [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
    ```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. Crea un'istanza Lite di [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. Crea un'istanza Standard di [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams).
    ```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## Crea un argomento di messaggistica e un bucket {{site.data.keyword.cos_short}}

{: #topics}

Inizia creando un argomento {{site.data.keyword.messagehub}} e un bucket {{site.data.keyword.cos_short}}. Gli argomenti definiscono dove le applicazioni recapitano i messaggi nei sistemi di messaggistica di pubblicazione-sottoscrizione. Dopo che i messaggi sono stati ricevuti ed elaborati, saranno archiviati in un file che si trova in un bucket {{site.data.keyword.cos_short}}.

1. Nel tuo browser, accedi all'istanza del servizio `log-analysis-hub` da [Resources](https://{DomainName}/resources?search=log-analysis).
2. Fai clic sul pulsante **+** per creare un argomento.
3. In **Topic Name** immetti `webserver` e fai clic sul pulsante **Create topic**.
4. Fai clic su **Service Credentials** e quindi sul pulsante **New Credential**.
5. Nella finestra di dialogo risultante, immetti `webserver-flow` nel campo **Name** e fai clic sul pulsante **Add**.
6. Fai clic su **View Credentials** e copia le informazioni in un luogo sicuro. Verranno utilizzate nella prossima sezione.
7. Torna all'[elenco delle risorse](https://{DomainName}/resources?search=log-analysis) e seleziona l'istanza del servizio `log-analysis-cos`.
8. Fai clic su **Create bucket**.
    * In **Name** immetti un nome univoco per il bucket.
    * Seleziona **Cross Region** per **Resiliency**.
    * Seleziona **us-geo** come **Location**.
    * Fai clic su **Create bucket**.

## Crea un'origine del flusso di stream

{: #streamsflow}

In questa sezione, inizierai a configurare un flusso di stream che riceve i messaggi di log. Il servizio {{site.data.keyword.streaminganalyticsshort}} si avvale della tecnologia {{site.data.keyword.streamsshort}}, che può analizzare milioni di eventi al secondo, consentendo tempi di risposta inferiori al millisecondo e processi decisionali immediati.

1. Nel tuo browser, accedi a [Watson Data Platform](https://dataplatform.ibm.com).
2. Seleziona il pulsante o il tile **New project**, quindi il tile **Basic** e fai clic su **OK**.
    * In **Name**, immetti `webserver-logs`.
    * L'opzione **Storage** dovrebbe essere impostata su `log-analysis-cos`. In caso contrario, seleziona l'istanza del servizio.
    * Fai clic sul pulsante **Create**.
3. Nella pagina risultante, fai clic sulla scheda **Settings** e seleziona **Streams Designer** in **Tools**. Termina facendo clic sul pulsante **Save**.
4. Fai clic sul pulsante **Add to project** e quindi su **Streams flow** dalla barra di navigazione in alto.
    * Fai clic su **Associate an IBM Streaming Analytics instance with a container-based plan**.
    * Crea una nuova istanza di {{site.data.keyword.streaminganalyticsshort}} selezionando il pulsante di opzione **Lite** e facendo clic su **Create**. Non selezionare la VM Lite.
    * Per **Service name** immetti `log-analysis-sa` e fai clic su **Confirm**.
    * Per il campo **Name** del flusso di stream, immetti `webserver-flow`.
    * Termina facendo clic su **Create**.
5. Nella pagina risultante, seleziona il tile **{{site.data.keyword.messagehub}}**.
    * Fai clic su **Add Connection** e seleziona la tua istanza {{site.data.keyword.messagehub}} `log-analysis-hub`. Se non vedi elencata la tua istanza, seleziona l'opzione **IBM {{site.data.keyword.messagehub}}**. Immetti manualmente i **dettagli di connessione** che hai ottenuto nel campo **Service credentials** nella sezione precedente. **Denomina** la connessione `webserver-flow`.
    * Fai clic su **Create** per creare la connessione.
    * Seleziona `webserver` dal menu a discesa **Topic**.
    * Seleziona **Start with the first new message** dal menu a discesa **Initial Offset**.
    * Fai clic su **Continue**.
6. Lascia aperta la pagina **Preview Data**; verrà utilizzata nella prossima sezione.

## Utilizzo degli strumenti della console Kafka con {{site.data.keyword.messagehub}}

{: #kafkatools}

Il `webserver-flow` è attualmente inattivo e in attesa di messaggi. In questa sezione, configurerai gli strumenti della console Kafka per lavorare con {{site.data.keyword.messagehub}}. Gli strumenti della console Kafka ti consentono di produrre messaggi arbitrari dal terminale e di inviarli a {{site.data.keyword.messagehub}}, che attiverà il `webserver-flow`.

1. Scarica e decomprimi il [client Kafka 0.10.2.X](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz).
2. Imposta la directory su `bin` e crea un file di testo denominato `message-hub.config` con i seguenti contenuti.
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. Sostituisci `USER` e `PASSWORD` nel tuo file `message-hub.config` con i valori `user` e `password` visualizzati in **Service Credentials** nella sezione precedente. Salva `message-hub.config`.
4. Dalla directory `bin`, immetti il seguente comando. Sostituisci `KAFKA_BROKERS_SASL` con il valore `kafka_brokers_sasl` visualizzato in **Service Credentials**. Viene fornito un esempio.
    ```sh
    ./kafka-console-producer.sh --broker-list KAFKA_BROKERS_SASL \
    --producer.config message-hub.config --topic webserver
    ```
    {: pre}
    ```sh
    ./kafka-console-producer.sh --broker-list \
    kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093 \
    --producer.config message-hub.config --topic webserver
    ```
5. Lo strumento della console Kafka è in attesa di input. Copia e incolla il seguente messaggio di log nel terminale. Premi `invio` per inviare il messaggio di log a {{site.data.keyword.messagehub}}. Nota che i messaggi inviati vengono visualizzati anche nella pagina **Preview Data** di `webserver-flow`.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![Preview page](images/solution31/preview_data.png)

## Crea una destinazione del flusso di stream

{: #streamstarget}

In questa sezione, completerai la configurazione del flusso di stream definendo una destinazione. La destinazione verrà utilizzata per archiviare i messaggi di log in entrata nel bucket {{site.data.keyword.cos_short}} creato in precedenza. Il processo di archiviazione e aggiunta dei messaggi di log in entrata in un file verrà eseguito automaticamente da {{site.data.keyword.streaminganalyticsshort}}.

1. Nella pagina di anteprima (**Preview Page**) di `webserver-flow`, fai clic sul pulsante **Continue**.
2. Seleziona il tile **{{site.data.keyword.cos_full_notm}}** come destinazione.
    * Fai clic su **Add Connection** e seleziona `log-analysis-cos`.
    * Fai clic su **Create**.
    * Per **File path** immetti `/YOUR_BUCKET_NAME/http-logs_%TIME.csv`. Sostituisci `YOUR_BUCKET_NAME` con il valore utilizzato nella prima sezione.
    * Seleziona **csv** nel menu a discesa **Format**.
    * Seleziona la casella di spunta **Column header row**.
    * Seleziona **File Size** nel menu a discesa **File Creation Policy**.
    * Imposta il limite su 100 MB immettendo `102400` nella casella di testo **File Size (KB)**.
    * Fai clic su **Continue**.
3. Fai clic su **Save**.
4. Fai clic sul pulsante di riproduzione **>** per **avviare il flusso di stream**.
5. Dopo che il flusso è stato avviato, invia di nuovo più messaggi di log dallo strumento della console Kafka. Puoi controllare come arrivano i messaggi visualizzando il `webserver-flow` in Streams Designer.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. Torna al tuo bucket in {{site.data.keyword.cos_short}}. Sarà presente un nuovo file `log.csv` una volta che un numero sufficiente di messaggi è entrato nel flusso o il flusso viene riavviato.

![webserver-flow](images/solution31/flow.png)

## Aggiungi un comportamento condizionale ai flussi di stream

{: #streamslogic}

Fino ad ora, il flusso di stream è un semplice condotto che sposta i messaggi da {{site.data.keyword.messagehub}} a {{site.data.keyword.cos_short}}. Molto probabilmente, i team vorranno conoscere eventi di interesse in tempo reale. Ad esempio, i singoli team potrebbero usufruire degli avvisi quando si verificano eventi HTTP 500 (errore dell'applicazione). In questa sezione, aggiungerai la logica condizionale al flusso per identificare i codici HTTP 200 (OK) e non HTTP 200.

1. Utilizza il pulsante matita per **modificare il flusso di stream**.
2. Crea un nodo filtro che gestisca le risposte HTTP 200.
    * Dalla tavolozza **Nodes**, trascina il nodo **Filter** da **PROCESSING AND ANALYTICS** nel canvas.
    * Immetti `OK` nella casella di testo del nome, che attualmente contiene la parola `Filter`.
    * Immetti la seguente istruzione nell'area di testo **Condition Expression**.
      ```sh
      responseCode == 200
      ```
      {: pre}
    * Con il mouse, traccia una linea dall'output del nodo **{{site.data.keyword.messagehub}}** (lato destro) all'input del tuo nodo **OK** (lato sinistro).
    * Dalla tavolozza **Nodes**, trascina il nodo **Debug** che si trova sotto **TARGETS** nel canvas.
    * Connetti il nodo **Debug** al nodo **OK** tracciando una linea tra questi due.
3. Ripeti il processo per creare un filtro `Not OK` utilizzando gli stessi nodi e la seguente istruzione di condizione.
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. Fai clic sul pulsante di riproduzione per **salvare ed eseguire il flusso di stream**.
5. Se richiesto, fai clic sul link per **eseguire la nuova versione**.

![Flow designer](images/solution31/flow_design.png)

## Aumento del carico di messaggi

{: #streamsload}

Per visualizzare la gestione condizionale nel flusso di stream, aumenterai il volume di messaggi inviato a {{site.data.keyword.messagehub}}. Il programma Node.js fornito simula un flusso realistico di messaggi su {{site.data.keyword.messagehub}} in base al traffico verso il server web. Per dimostrare la scalabilità di {{site.data.keyword.messagehub}} e {{site.data.keyword.streaminganalyticsshort}}, aumenterai la velocità effettiva dei messaggi di log.

Questa sezione utilizza [node-rdkafka](https://www.npmjs.com/package/node-rdkafka). Se l'installazione del simulatore non riesce, consulta la pagina npmjs per istruzioni sulla risoluzione dei problemi. Se i problemi persistono, puoi passare alla sezione successiva e caricare manualmente i dati.

1. Scarica e decomprimi il file di log [Jul 01 to Jul 31, ASCII format, 20.7 MB gzip compressed](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz) dalla NASA.
2. Clona e installa il simulatore di log da [IBM-Cloud su GitHub](https://github.com/IBM-Cloud/kafka-log-simulator).
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. Passa alla directory del simulatore e immetti i seguenti comandi per configurare il simulatore e produrre messaggi di eventi di log. Sostituisci `LOGFILE` con il file che hai scaricato. Sostituisci `BROKERLIST` e `APIKEY` con i valori corrispondenti in **Service Credentials** che hai utilizzato in precedenza. Viene fornito un esempio.
    ```sh
    npm install
    ```
    ```sh
    npm run build
    ```
    ```sh
    node dist/index.js --file LOGFILE --parser httpd --broker-list BROKERLIST \
    --api-key APIKEY --topic webserver --rate 100
    ```
    {: pre}
    ```sh
    node dist/index.js --file /Users/ibmcloud/Downloads/NASA_access_log_Jul95 \
    --parser httpd --broker-list \
    "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093" \
    --api-key Np15YZKN3SCdABUsOpJYtpue6jgJ7CwYgsoCWaPbuyFbdM4R \
    --topic webserver --rate 100
    ```
4. Nel browser, torna al tuo `webserver-flow` dopo che il simulatore ha iniziato a produrre messaggi.
5. Arresta il simulatore dopo che un numero desiderato di messaggi ha attraversato i rami condizionali utilizzando `control+C`.
6. Prova a utilizzare il ridimensionamento di {{site.data.keyword.messagehub}} aumentando o riducendo il valore `--rate`.

Il simulatore ritarderà l'invio del messaggio successivo in base al tempo trascorso nel log del server web. L'impostazione `--rate 1` invia gli eventi in tempo reale. L'impostazione `--rate 100` indica che per ogni 1 secondo di tempo trascorso nel log del server web viene utilizzato un ritardo di 10ms tra i messaggi.
{: tip}

![Carico del flusso impostato su 10](images/solution31/flow_load_10.png)

## Analisi dei dati di log tramite SQL Query

{: #sqlquery}

A seconda del numero di messaggi inviati dal simulatore, le dimensioni del file di log su {{site.data.keyword.cos_short}} sono sicuramente aumentate. Ora agirai da investigatore rispondendo a domande di controllo o conformità combinando SQL Query con il tuo file di log. Il vantaggio dell'utilizzo di SQL Query è che il file di log è direttamente accessibile: non sono necessari ulteriori trasformazioni o server di database.

Se preferisci non aspettare che il simulatore invii tutti i messaggi di log, carica il [file CSV completo](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp) su {{site.data.keyword.cos_short}} per iniziare immediatamente.
{: tip}

1. Accedi all'istanza del servizio `log-analysis-sql` dall'[elenco delle risorse](https://{DomainName}/resources?search=log-analysis). Seleziona **Open UI** per avviare SQL Query.
2. Immetti il seguente SQL nell'area di testo **Type SQL here ...**.
    ```sql
    -- What are the top 10 web pages on NASA from July 1995?
    -- Which mission might be significant?
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. Richiama l'URL SQL dell'oggetto dal file di log.
    * Dall'[elenco delle risorse](https://{DomainName}/resources?search=log-analysis), seleziona l'istanza del servizio `log-analysis-cos`.
    * Seleziona il bucket che hai creato in precedenza.
    * Fai clic sul menu di overflow sul file `http-logs_TIME.csv` e seleziona **Object SQL URL**.
    * **Copia** l'URL negli appunti.
4. Aggiorna la clausola `FROM` con l'URL SQL dell'oggetto e fai clic su **Run**.
5. Il risultato può essere visualizzato nella scheda **Result**. Mentre alcune pagine, come la home page del Kennedy Space Center, sono previste, una missione è molto popolare al momento.
6. Seleziona la scheda **Query Details** per visualizzare ulteriori informazioni come la posizione in cui è stato archiviato il risultato su {{site.data.keyword.cos_short}}.
7. Prova le seguenti coppie di domande e risposte aggiungendole singolarmente all'area di testo **Type SQL here ...**.
    ```sql
    -- Who are the top 5 viewers?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- Which viewer has suspicious activity based on application failures?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- Which requests showed a page not found error to the user?
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- What are the top 10 largest files?
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- What is the distribution of total traffic by hour?
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- Why did the previous result return an empty hour?
    -- Hint, find the malformed hostname.
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

Le clausole FROM non sono limitate a un singolo file. Utilizza `cos://us-geo/YOUR_BUCKET_NAME/` per eseguire query SQL su tutti i file nel bucket.
{: tip}

## Espandi l'esercitazione

{: #expand}

Congratulazioni, hai creato una pipeline di analisi dei log con {{site.data.keyword.cloud_notm}}. Di seguito vengono forniti ulteriori suggerimenti per migliorare la tua soluzione.

* Utilizza ulteriori destinazioni in Streams Designer per archiviare i dati in [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) o esegui il codice in [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* Segui l'esercitazione [Crea un data lake utilizzando Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage) per aggiungere un dashboard ai dati di log
* Integra sistemi aggiuntivi con {{site.data.keyword.messagehub}} utilizzando [{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect).

## Rimuovi i servizi

{: #removal}

Dall'[elenco delle risorse](https://{DomainName}/resources?search=log-analysis), utilizza la voce di menu **Delete** o **Delete service** nel menu di overflow per rimuovere le seguenti istanze del servizio.

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## Contenuto correlato

{:related}

* [Apache Kafka](https://kafka.apache.org/)
