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

# Accelera la fornitura di file statici utilizzando una CDN
{: #static-files-cdn}

Questa esercitazione ti guida nella procedura per ospitare e servire le risorse del sito web (immagini, video, documenti) e il contenuto generato dall'utente in un {{site.data.keyword.cos_full_notm}} e per utilizzare una [{{site.data.keyword.cdn_full}} (CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai) per la distribuzione veloce e sicura agli utenti nel mondo.

Le applicazioni web hanno tipi diversi di contenuto: contenuto HTML, immagini, video, CSS (cascading style sheet), file JavaScript, contenuto generato dall'utente. Alcuni contenuti cambiano spesso, altri non così tanto, ad alcuni gli utenti accedono molto spesso, ad altri occasionalmente. Man mano che crescono i destinatari dell'applicazione, potresti voler scaricare la fornitura di questi contenuti su un altro componente, liberando le risorse per la tua applicazione principale. Potresti anche volere che questi contenuti vengano serviti da un'ubicazione vicina agli utenti della tua applicazione, ovunque siano nel mondo. 

Ci sono molti motivi per cui utilizzare una CDN in queste situazioni: 
* la CDN memorizzerà nella cache il contenuto, estraendo il contenuto dall'origine (i tuoi server) solo se non è disponibile nella sua cache oppure se è scaduto;
* con più data center in tutto il mondo, la CDN servirà il contenuto memorizzato nella cache dall'ubicazione più vicina per i tuoi utenti;
* essendo in esecuzione in un dominio diverso da quello della tua applicazione principale, il browser sarà in grado di caricare più contenuti in parallelo - la maggior parte dei browser ha un limite per il numero di connessioni per nome host. 

## Obiettivi
{: #objectives}

* Caricare i file in un bucket {{site.data.keyword.cos_full_notm}}.
* Rendere il contenuto globalmente disponibile con una CDN (Content Delivery Network).
* Esporre i file utilizzando un'applicazione web Cloud Foundry. 

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti prodotti:
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura

<p style="text-align: center;">
![Architettura](images/solution3/Architecture.png)
</p>

1. L'utente accede all'applicazione
2. L'applicazione include il contenuto distribuito tramite una CDN (Content Delivery Network)
3. Se il contenuto non è disponibile nella CDN o è scaduto, la CDN estrae il contenuto dall'origine.

## Prima di iniziare
{: #prereqs}

Contatta l'utente master del tuo account dell'infrastruttura per ottenere le seguenti autorizzazioni: 
   * Gestisci account CDN
   * Gestisci archiviazione
   * Chiave API

Queste autorizzazioni sono necessarie per poter visualizzare e utilizzare i servizi Archiviazione e CDN.

## Ottieni il codice dell'applicazione web
{: #get_code}

Consideriamo una semplice applicazione web con diversi tipi di contenuto come immagini, video e CSS (cascading style sheet). Archivierai il contenuto in un bucket di archiviazione e configurerai la CDN per utilizzare il bucket come sua origine.

Per iniziare, richiama il codice dell'applicazione:

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## Crea un Object Storage
{: #create_cos}

{{site.data.keyword.cos_full_notm}} offre un'archiviazione cloud flessibile, conveniente e scalabile per dati non strutturati. 

1. Vai al [catalogo](https://{DomainName}/catalog/) nella console e seleziona [**Object Storage**](https://{DomainName}/catalog/services/cloud-object-storage) dalla sezione Storage.
2. Crea una nuova istanza di {{site.data.keyword.cos_full_notm}}
4. Nel dashboard del servizio, fai clic su **Create Bucket**.
5. Imposta un nome bucket univoco come ad esempio `username-mywebsite` e fai clic su **Create**. Evita di utilizzare i punti (.) nel nome bucket.

## Carica i file in un bucket
{: #upload}

In questa sezione, utilizzerai lo strumento della riga di comando **curl** per caricare i file nel bucket.

1. Accedi a {{site.data.keyword.Bluemix_notm}} dalla CLI e ottieni un token da IBM Cloud IAM.
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. Copia il token dall'output del comando nel passo precedente. 
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. Imposta il valore del token e il nome bucket in una variabile di ambiente per un facile accesso. 
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. Carica i file denominati `a-css-file.css`, `a-picture.png` e `a-video.mp4` dalla directory del contenuto del codice dell'applicazione web che hai scaricato in precedenza. Carica i file nella root del bucket.
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. Visualizza i tuoi file dal tuo dashboard.
   ![](images/solution3/Buckets.png)
6. Accedi ai file tramite il tuo browser utilizzando un link simile al seguente esempio:

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## Assicurati che i file siano globalmente disponibili con una CDN

In questa sezione, creerai un servizio CDN. Il servizio CDN distribuisce il contenuto dove è necessario. La prima volta che viene richiesto il contenuto, viene estratto dal server host (il tuo bucket in {{site.data.keyword.cos_full_notm}}) nella rete e rimane lì affinché gli altri utenti possano accedervi velocemente senza la latenza di rete per raggiungere nuovamente il server host. 

### Crea un'istanza CDN

1. Vai al catalogo nella console e seleziona **Content Delivery Network** dalla sezione Network. Questa CDN si avvale di Akamai. Fai clic su **Create**.
2. Nella finestra di dialogo successiva, imposta il valore di **Hostname** per la CDN sul tuo dominio personalizzato. Anche se imposti un dominio personalizzato, puoi ancora accedere al contenuto della CDN tramite il CNAME fornito da IBM. Pertanto, se non intendi utilizzare il dominio personalizzato, puoi impostare un nome arbitrario. 
3. Imposta il prefisso **Custom CNAME** su un nome univoco. 
4. Poi, in **Configure your origin**, seleziona **Object Storage** per configurare la CDN per COS.
5. Imposta **Endpoint** sul tuo endpoint API del bucket, ad esempio *s3-api.us-geo.objectstorage.softlayer.net*.
6. Lascia vuoti **Host header** e **Path**. Imposta **Bucket name** su *your-bucket-name*.
7. Abilita entrambe le porte HTTP (80) e HTTPS (443).
8. Per **SSL certificate**, seleziona *DV SAN Certificate* se vuoi utilizzare un dominio personalizzato. Altrimenti, per accedere all'archiviazione tramite CNAME, seleziona l'opzione **Wildcard Certificate*.
9. Fai clic su **Create**.

### Accedi al tuo contenuto tramite il CNAME CDN

1. Seleziona l'istanza CDN [nell'elenco](https://{DomainName}/classic/network/cdn).
2. Se in precedenza hai selezionato *DV SAN Certificate*, ti verrà richiesta la convalida del dominio una volta completata la configurazione iniziale. Segui i passi mostrati quando fai clic su **View domain validation**.
3. Il pannello **Details** mostra sia il valore di **Hostname** che di **CNAME** per la tua CDN.
4. Accedi al tuo file con `https://your-cdn-cname.cdnedge.bluemix.net/a-picture.png` oppure, se stai utilizzando un dominio personalizzato, `https://your-cdn-hostname/a-picture.png`. Se ometti il nome file, dovresti vedere invece S3 ListBucketResult.

## Distribuisci l'applicazione Cloud Foundry

L'applicazione contiene una pagina web public/index.html che include i riferimenti ai file ora ospitati in {{site.data.keyword.cos_full_notm}}. L'`app.js` di backend serve questa pagina web e sostituisce un segnaposto con l'ubicazione effettiva della tua CDN. In questo modo, tutte le risorse utilizzate dalla pagina web vengono servite dalla CDN.

1. Da un terminale, vai alla directory in cui hai estratto il codice. 
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. Esegui il push dell'applicazione senza avviarla. 
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. Configura la variabile di ambiente CDN_NAME in modo che l'applicazione possa fare riferimento al contenuto della CDN.
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. Avvia l'applicazione. 
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. Accedi all'applicazione dal tuo browser web, il foglio di stile della pagina, un'immagine e un video vengono caricati dalla CDN. 

![](images/solution3/Application.png)

L'utilizzo di una CDN con {{site.data.keyword.cos_full_notm}} è una combinazione potente che ti consente di ospitare i file e di servirli agli utenti di tutto il mondo. Puoi anche utilizzare {{site.data.keyword.cos_full_notm}} per archiviare i file che i tuoi utenti caricano nella tua applicazione. 

## Rimuovi le risorse

* Elimina l'applicazione Cloud Foundry
* Elimina il servizio CDN (Content Delivery Network)
* Elimina il bucket o il servizio {{site.data.keyword.cos_full_notm}}

## Contenuto correlato

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[Gestisci l'accesso a {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[Introduzione a CDN](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
