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

# Crea un data lake utilizzando Object Storage
{: #smart-data-lake}

Le definizioni del termine data lake variano, ma nel contesto di questa esercitazione, un data lake è un approccio per archiviare i dati nel loro formato nativo per uso organizzativo. A tal fine, creerai un data lake per la tua organizzazione utilizzando {{site.data.keyword.cos_short}}. Combinando {{site.data.keyword.cos_short}} e SQL Query, gli analisti di dati possono eseguire la query dei dati nell'ubicazione in cui si trovano utilizzando SQL. Ti avvarrai anche del servizio SQL Query in un notebook Jupyter per condurre una semplice analisi. Una volta terminato, consenti agli utenti non tecnici di rilevare le loro informazioni approfondite utilizzando {{site.data.keyword.dynamdashbemb_notm}}.

## Obiettivi

- Utilizzare {{site.data.keyword.cos_short}} per archiviare file di dati non elaborati
- Eseguire la query dei dati direttamente da {{site.data.keyword.cos_short}} utilizzando SQL Query
- Rifinire e analizzare i dati {{site.data.keyword.DSX_full}}
- Condividere i dati nella tua organizzazione con {{site.data.keyword.dynamdashbemb_notm}}

## Servizi utilizzati

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [SQL Query](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Architettura

![Architettura](images/solution29/architecture.png)

1. I dati non elaborati vengono archiviati in {{site.data.keyword.cos_short}}
2. I dati vengono ridotti, migliorati o rifiniti con SQL Query
3. L'analisi dei dati viene eseguita in {{site.data.keyword.DSX}}
4. La LOB (Line of business) accede a un'applicazione web
5. I dati rifiniti vengono estratti da {{site.data.keyword.cos_short}}
6. I grafici LOB vengono creati utilizzando {{site.data.keyword.dynamdashbemb_notm}}

## Prima di iniziare

- [Installa Git](https://git-scm.com/)
- [Installa la CLI {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
- [Installa Aspera Connect](http://downloads.asperasoft.com/connect2/)
- [Installa Node.js e NPM](https://nodejs.org)

## Crea i servizi

In questa sezione, creerai i servizi richiesti per creare il tuo data lake.

Questa sezione utilizza la riga di comando per creare le istanze del servizio. In alternativa, puoi eseguire la stessa operazione dalla pagina del servizio nel catalogo utilizzando i link forniti.
{: tip}

1. Accedi a {{site.data.keyword.cloud_notm}} tramite la riga di comando e specifica il tuo account Cloud Foundry. Vedi [Introduzione alla CLI]https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Crea un'istanza di [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) con un alias Cloud Foundry. Se hai già un'istanza del servizio, esegui il comando `service-alias-create` con il nome servizio esistente.
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. Crea un'istanza di [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. Crea un'istanza di [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio).
    ```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. Crea un'istanza di [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded) con un alias Cloud Foundry.
    ```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. Passa alla directory di lavoro ed esegui il seguente comando per clonare un [repository GitHub](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard) dell'applicazione del dashboard. Quindi, esegui il push dell'applicazione nella tua organizzazione Cloud Foundry. Verrà eseguito automaticamente il bind dell'applicazione ai servizi richiesti sopra riportati utilizzando il relativo file [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml).
    ```sh
    git clone https://github.com/IBM-Cloud/nodejs-data-lake-dashboard.git
    ```
    {: pre}
    ```sh
    cd nodejs-data-lake-dashboard
    ```
    {: pre}
    ```sh
    npm install
    ```
    {: pre}
    ```sh
    npm run push
    ```
    {: pre}

    Dopo la distribuzione, l'applicazione sarà pubblica e in ascolto su un nome host casuale. Puoi andare alla pagina dell'[elenco delle risorse](https://{DomainName}/resources), selezionare l'applicazione in Cloud Foundry Apps e visualizzare l'URL o eseguire il comando `ibmcloud cf app dashboard-nodejs routes` per vedere gli instradamenti.
    {: tip}

7. Conferma che l'applicazione è attiva accedendo al suo URL pubblico nel browser.

![Pagina di destinazione del dashboard](images/solution29/dashboard-start.png)

## Caricamento dei dati

In questa sezione, caricherai i dati in un bucket {{site.data.keyword.cos_short}} utilizzando {{site.data.keyword.CHSTSshort}} integrato. {{site.data.keyword.CHSTSshort}} protegge i dati mentre vengono caricati nel bucket e [può ridurre in modo significativo i tempi di trasferimento](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/).

1. Scarica il file CSV [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD). La dimensione del file è di 81MB e il download potrebbe richiedere alcuni minuti. 
2. Nel tuo browser, accedi all'istanza del servizio **data-lake-cos** dall'[elenco delle risorse](https://{DomainName}/resources).
3. Crea un nuovo bucket per archiviare i dati.
    - Fai clic sul pulsante **Create a bucket**.
    - Seleziona **Regional** dall'elenco a discesa **Resiliency**.
    - Seleziona **us-south** da **Location**. Al momento, {{site.data.keyword.CHSTSshort}} è disponibile solo per i bucket creati nell'ubicazione `us-south`. In alternativa, scegli un'altra ubicazione e utilizza il tipo di trasferimento **Standard** nella prossima sezione. 
    - Specifica un **nome** per il bucket e fai clic su **Create**. Se ricevi un errore *AccessDenied*, prova ad utilizzare un nome bucket più specifico.
4. Carica il file CSV in {{site.data.keyword.cos_short}}.
    - Dal tuo bucket, fai clic sul pulsante **Add objects**.
    - Seleziona il pulsante di opzione **Aspera high-speed transfer**.
    - Fai clic sul pulsante **Add files**. Verrà aperto, in una finestra separata, il plugin Aspera - possibilmente dietro la finestra del tuo browser.
    - Vai al file CSV precedentemente scaricato e selezionalo.

![Bucket con file CSV](images/solution29/cos-bucket.png)

## Utilizzo dei dati

In questa sezione, convertirai la serie di dati non elaborati originale in una coorte mirata basata sugli attributi time e age. Ciò è utile per i consumatori del data lake che hanno interessi specifici o che hanno a che fare con serie di dati molto grandi. 

Utilizzerai SQL Query per manipolare i dati nell'ubicazione in cui si trovano in {{site.data.keyword.cos_short}} utilizzando istruzioni SQL conosciute. SQL Query ha un supporto integrato per CSV, JSON e Parquet - non sono necessari ulteriori servizi di calcolo o processi di estrazione-trasformazione-caricamento. 

1. Accedi all'istanza del servizio SQL Query **data-lake-sql** dal tuo [elenco delle risorse](https://{DomainName}/resources).
2. Seleziona **Open UI**.
3. Crea una nuova serie di dati eseguendo SQL direttamente nel file CSV caricato in precedenza.
    - Immetti il seguente SQL nell'area di testo **Type SQL here ...**.
        ```
        SELECT
        `Dr Number` AS id,
        `Date Occurred` AS date,
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
        FROM cos://us-south/<your-bucket-name>/Traffic_Collision_Data_from_2010_to_Present.csv
        WHERE
        `Time Occurred` >= 1700 AND
        `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND
        `Victim Age` <= 35
        ```
        {: codeblock}
    - Sostituisci l'URL nella clausola `FROM` con il nome del tuo bucket.
4. **Target** creerà automaticamente un bucket {{site.data.keyword.cos_short}} per conservare il risultato. Modifica **Target** con `cos://us-south/<your-bucket-name>/results`.
5. Fai clic sul pulsante **Run**. I risultati appariranno di seguito. 
6. Nella scheda **Query Details**, fai clic sull'icona **Launch** subito dopo l'URL **Result Location** per visualizzare la serie di dati intermedia che ora viene archiviata anche in {{site.data.keyword.cos_short}}.

![Notebook](images/solution29/sql-query.png)

## Combina i notebook Jupyter con SQL Query

In questa sezione, utilizzerai il client SQL Query all'interno di un notebook Jupyter. Questo riutilizza i dati archiviati in {{site.data.keyword.cos_short}} all'interno di uno strumento di analisi dei dati. La combinazione crea anche serie di dati che vengono archiviate automaticamente in {{site.data.keyword.cos_short}} che possono essere poi utilizzate con {{site.data.keyword.dynamdashbemb_notm}}.

1. Crea un nuovo notebook Jupyter in {{site.data.keyword.DSX}}.
    - In un browser, apri [{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true).
    - Fai clic sul tile **Create a Project** seguito da **Data Science**.
    - Fai clic su **Create project** e poi specifica un **Project name**.
    - Assicurati che **Storage** sia impostato su **data-lake-cos**.
    - Fai clic su **Create**.
    - Nel progetto risultante, fai clic su **Add to project** e **Notebook**.
    - Dalla scheda **Blank**, immetti un **Notebook name**.
    - Lascia **Language** e **Runtime** impostati sui valori predefiniti; fai clic su **Create notebook**.
2. Dal Notebook, installa e importa PixieDust e ibmcloudsql aggiungendo i seguenti comandi nel prompt di input **In [ ]: ** e quindi **esegui**.
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. Aggiungi una chiave API {{site.data.keyword.cos_short}} al Notebook. Ciò consentirà di archiviare i risultati di SQL Query in {{site.data.keyword.cos_short}}.
    - Aggiungi quanto segue nel prompt **In [ ]: ** successivo e poi **esegui**.
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - Dal terminale, crea una chiave API.
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - Copia la **chiave API** negli appunti. 
    - Incolla la chiave API nella casella di testo nel Notebook e premi il tasto `invio`.
    - Devi anche archiviare la chiave API in un'ubicazione permanente e sicura; Notebook non archivia la chiave API.
4. Aggiungi il CRN (Cloud Resource Name) dell'istanza SQL Query nel Notebook.
    - Nel prompt **In [ ]: ** successivo, assegna il CRN a una variabile nel tuo Notebook.
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - Dal terminale, copia il CRN dalla proprietà **ID** nei tuoi appunti.
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - Incolla il CRN tra virgolette singole ed **esegui**.
5. Aggiungi un'altra variabile a Notebook per specificare il bucket {{site.data.keyword.cos_short}} ed **esegui**.
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. Esegui i seguenti comandi in un altro prompt **In [ ]: ** ed **esegui** per visualizzare la serie di risultati. Al tuo bucket verrà anche aggiunto un nuovo file `accidents/jobid=<id>/<part>.csv*` che includerà il risultato di `SELECT`.
    ```python
    sqlClient = ibmcloudsql.SQLQuery(cloud_api_key, sql_crn, sql_cos_endpoint + '/accidents')

    data_source = sql_cos_endpoint + "/Traffic_Collision_Data_from_2010_to_Present.csv"

    query = """
    SELECT
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
    FROM  {}
    WHERE
        `Time Occurred` >= 1700 AND `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND `Victim Age` <= 35
    """.format(data_source)

    traffic_collisions = sqlClient.run_sql(query)
    traffic_collisions.head()
    ```
    {: codeblock}

## Visualizza i dati utilizzando PixieDust

In questa sezione, visualizzerai la serie di risultati precedente utilizzando PixieDust e Mapbox per identificare meglio i pattern o gli hot spot per gli incidenti di traffico. 

1. Crea un'espressione di tabella comune per convertire la colonna `location` per separare le colonne `latitude` e `longitude`. **Esegui** quanto riportato di seguito dal prompt del Notebook.
    ```python
    query = """
    WITH location AS (
        SELECT
            id,
            cast(split(coordinates, ',')[0] as float) as latitude,
            cast(split(coordinates, ',')[1] as float) as longitude
        FROM (SELECT
                `Dr Number` as id,
                regexp_replace(Location, '[()]', '') as coordinates
            FROM {0}
        )
    )
    SELECT
        d.`Dr Number` as id,
        d.`Date Occurred` as date,
        d.`Time Occurred` AS time,
        d.`Area Name` AS area,
        d.`Victim Age` AS age,
        d.`Victim Sex` AS sex,
        l.latitude,
        l.longitude
    FROM {0} AS d
        JOIN
        location AS l
        ON l.id = d.`Dr Number`
    WHERE
        d.`Time Occurred` >= 1700 AND
        d.`Time Occurred` <= 2000 AND
        d.`Victim Age` >= 20 AND
        d.`Victim Age` <= 35 AND
        l.latitude != 0.0000 AND
        l.latitude != 0.0000
    """.format(data_source)

    traffic_location = sqlClient.run_sql(query)
    traffic_location.head()
    ```
    {: codeblock}
2. Nel prompt **In [ ]: ** successivo, **esegui** il comando `display` per visualizzare il risultato utilizzando PixieDust.
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. Seleziona il pulsante a discesa del grafico; quindi seleziona **Map**.
4. Aggiungi `latitude` e `longitude` a **Keys**. Aggiungi `id` e `age` a **Values**. Fai clic su **OK** per visualizzare la mappa. 
5. Fai clic sull'icona **Save** per salvare il tuo Notebook in {{site.data.keyword.cos_short}}.

![Notebook](images/solution29/notebook-mapbox.png)

## Condividi la tua serie di dati con l'organizzazione

Non tutti gli utenti del data lake sono data scientist. Puoi consentire agli utenti non tecnici di acquisire le informazioni approfondite provenienti dal data lake utilizzando {{site.data.keyword.dynamdashbemb_notm}}. Analogamente a SQL Query, {{site.data.keyword.dynamdashbemb_notm}} può leggere i dati direttamente da {{site.data.keyword.cos_short}} utilizzando dashboard precostruiti. Questa sezione presenta una soluzione che consente a qualsiasi utente di accedere al data lake e di creare un dashboard personalizzato.

1. Accedi all'URL pubblico dell'applicazione dashboard di cui precedentemente hai eseguito il push in {{site.data.keyword.Bluemix_notm}}.
2. Seleziona un template che corrisponda al layout che hai previsto. (I seguenti passi utilizzano il secondo layout nella prima riga.)
3. Utilizza il pulsante `Add a source` che compare nella sezione laterale `Selected sources`, espandi il `nome bucket` accoridan e fai clic su una delle voci di tabella `accidents/jobid=...`. Chiudi la finestra di dialogo utilizzando l'icona X nell'angolo in alto a destra. 
4. A sinistra, fai clic sull'icona `Visualizations` e poi fai clic su **Summary**.
5. Seleziona l'origine `accidents/jobid=...`, espandi `Table` e crea un grafico. 
    - Trascina e rilascia l'`id` sulla riga **Value**.
    - Comprimi il grafico utilizzando l'icona nell'angolo superiore. 
6. Di nuovo, da `Visualizations`, crea un grafico di tipo **Tree map**:
    - Trascina e rilascia l'`area` sulla riga **Area hierarchy**.
    - Trascina e rilascia l'`id` sulla riga **Size**.
    - Comprimi il grafico per visualizzare il risultato. 

![Grafico del dashboard](images/solution29/dashboard-chart.png)

## Esplora il tuo dashboard

In questa sezione, eseguirai alcuni passi aggiuntivi per esplorare le funzioni dell'applicazione dashboard e {{site.data.keyword.dynamdashbemb_notm}}.

1. Fai clic sul pulsante **Mode** nella barra degli strumenti dell'applicazione dashboard di esempio per modificare la modalità di visualizzazione `VIEW`.
2. Fai clic su uno qualsiasi dei tile colorati nella parte inferiore del grafico oppure sui valori dell'`area` nella legenda del grafico. Tale operazione applica un filtro locale alla scheda, facendo in modo che gli altri grafici mostrino dati specifici per il filtro. 
3. Fai clic sul pulsante **Save** nella barra degli strumenti. 
    - Immetti il nome del tuo dashboard nel campo di input corrispondente. 
    - Seleziona la scheda **Spec** per visualizzare la specifica di questo dashboard. Una spec è il formato file nativo per {{site.data.keyword.dynamdashbemb_notm}}. In esso troverai le informazioni sui grafici che hai creato come pure l'origine dati {{site.data.keyword.cos_short}} utilizzata.
    - Salva il tuo dashboard nell'archiviazione locale del tuo browser utilizzando il pulsante **Save** della finestra di dialogo.
4. Fai clic sul pulsante **New** della barra degli strumenti per creare un nuovo dashboard. Per aprire un dashboard salvato, fai clic sul pulsante **Open**. Per eliminare un dashboard, utilizza l'icona **Delete** nella finestra di dialogo Open Dashboard.

Nelle applicazioni di produzione, crittografa le informazioni come URL, nomi utente e password per impedire che vengano mostrati agli utenti finali. Vedi [Encrypting data source information](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information).
{: tip}

## Espandi l'esercitazione

Congratulazioni, hai creato un data lake utilizzando {{site.data.keyword.cos_short}}. Di seguito sono riportati altri suggerimenti per migliorare il tuo data lake. 

- Fai delle prove con altre serie di date utilizzando SQL Query
- Trasmetti i dati da più origini nel tuo data lake completando [Log di big data con Streaming Analytics e SQL](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics)
- Modifica il codice dell'applicazione dashboard per archiviare le specifiche del dashboard in [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) o {{site.data.keyword.cos_short}}
- Crea un'istanza del servizio [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id) per abilitare la sicurezza nell'applicazione dashboard

## Rimuovi le risorse

Esegui i seguenti comandi per rimuovere i servizi, le applicazioni e le chiavi utilizzate. 

```sh
ibmcloud resource service-binding-delete dashboard-nodejs-dde dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-binding-delete dashboard-nodejs-cos dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-dde
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-cos
```
{: pre}
```sh
ibmcloud iam api-key-delete data-lake-cos-key
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-dde
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-cos
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-sql
```
{: pre}
```sh
ibmcloud service delete data-lake-studio
```
{: pre}
```sh
ibmcloud app delete dashboard-nodejs
```
{: pre}

## Contenuto correlato

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Notebook Jupyter](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
