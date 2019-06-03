---
copyright:
  years: 2018, 2019
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

# Elaborazione dei dati asincrona utilizzando l'archiviazione oggetti e la messaggistica di pubblicazione/sottoscrizione
{: #pub-sub-object-storage}
In questa esercitazione, imparerai come utilizzare un servizio di messaggistica basato su Apache Kafka per orchestrare i carichi di lavoro a lunga esecuzione per le applicazioni in esecuzione in un cluster Kubernetes. Questo pattern viene utilizzato per disaccoppiare la tua applicazione, consentendo un maggior controllo sulla scalabilità e sulle prestazioni. {{site.data.keyword.messagehub}} può essere utilizzato per accodare il lavoro da eseguire senza ripercussioni sulle applicazioni produttore, rendendolo un sistema ideale per le attività a lunga durata. 

{:shortdesc}

Simulerai questo pattern utilizzando un esempio di elaborazione file. Crea innanzitutto un'applicazione IU che sarà utilizzata per caricare i file nell'archiviazione oggetti e generare messaggi che indicano il lavoro da eseguire. Creerai quindi un'applicazione nodo di lavoro separata che elaborerà in modo asincrono i file caricati dall'utente quando riceve i messaggi.

## Obiettivi
{: #objectives}

* Implementare un pattern produttore-consumatore con {{site.data.keyword.messagehub}}
* Eseguire il bind dei servizi a un cluster Kubernetes

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

In questa esercitazione, l'applicazione IU è scritta in Node.js e l'applicazione nodo di lavoro è scritta in Java, evidenziando la flessibilità di questo pattern. Anche se entrambe le applicazioni sono in esecuzione nello stesso cluster Kubernetes in questa esercitazione, entrambe potrebbero essere state implementate come una funzione senza server o un'applicazione Cloud Foundry.

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. L'utente carica il file utilizzando l'applicazione IU
2. Il file viene salvato in {{site.data.keyword.cos_full_notm}}
3. Viene inviato un messaggio all'argomento {{site.data.keyword.messagehub}} che indica che il nuovo file è in attesa di elaborazione.
4. Quando sono pronti, i nodi di lavoro sono in ascolto per i messaggi e iniziano a elaborare il nuovo file.

## Prima di iniziare
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - strumento per installare la CLI {{site.data.keyword.cloud_notm}}, Kubernetes, Helm e Docker.

## Crea un cluster Kubernetes
{: #create_kube_cluster}

1. Crea un cluster Kubernetes dal [Catalogo](https://{DomainName}/containers-kubernetes/launch). Denominalo `mycluster` per rendere più facile seguire questa esercitazione. Questa esercitazione può essere eseguita con un cluster **Free**.
   ![Creazione del cluster Kubernetes su IBM Cloud](images/solution25/KubernetesClusterCreation.png)
2. Controlla lo stato del tuo **cluster** e **nodo di lavoro** e attendi che diventi **ready**.

### Configura kubectl

In questo passo, configurerai kubectl per puntare al tuo cluster appena creato per il futuro. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) è uno strumento di riga di comando che utilizzi per interagire con un cluster Kubernetes.

1. Utilizza `ibmcloud login` per eseguire l'accesso in modo interattivo. Fornisci l'organizzazione (org), l'ubicazione e lo spazio in cui viene creato il cluster. Puoi riconfermare i dettagli eseguendo il comando `ibmcloud target`.
2. Quando il cluster è pronto, richiamane la configurazione.
   ```bash
   ibmcloud cs cluster-config <cluster-name>
   ```
   {: pre}
3. Copia e incolla il comando **export** per impostare la variabile di ambiente KUBECONFIG come indicato. Per verificare se la variabile di ambiente è impostata correttamente o meno, esegui questo comando:
`echo $KUBECONFIG`
4. Controlla che il comando `kubectl` sia configurato correttamente
   ```bash
   kubectl cluster-info
   ```
  {: pre}
   ![](images/solution2/kubectl_cluster-info.png)

## Crea un'istanza {{site.data.keyword.messagehub}}
 {: #create_messagehub}

{{site.data.keyword.messagehub}} è un servizio di messaggistica veloce, scalabile e completamente gestito basato su Apache Kafka, un sistema di messaggistica open-source ad alta velocità che fornisce una piattaforma a bassa latenza per gestire i feed di dati in tempo reale.

 1. Dal dashboard, fai clic su [**Create resource**](https://{DomainName}/catalog/) e seleziona [**{{site.data.keyword.messagehub}}**](https://{DomainName}/catalog/services/event-streams) dalla sezione Application Services.
 2. Denomina il servizio `mymessagehub` e fai clic su **Create**.
 3. Fornisci le credenziali del servizio al tuo cluster eseguendo il bind dell'istanza del servizio al tuo spazio dei nomi Kubernetes `default`.
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

Il comando cluster-service-bind crea un segreto del cluster che contiene le credenziali della tua istanza del servizio in formato JSON. Utilizza `kubectl get secrets ` per visualizzare il segreto generato con il nome `binding-mymessagehub`. Per ulteriori dettagli, vedi il documento relativo all'[integrazione dei servizi](https://{DomainName}/docs/containers?topic=containers-integrations#integrations).

{:tip}

## Crea un'istanza Object Storage

{: #create_cos}

{{site.data.keyword.cos_full_notm}} è crittografato e diffuso in più ubicazioni geografiche e si accede a esso su HTTP utilizzando un'API REST. {{site.data.keyword.cos_full_notm}} offre un'archiviazione cloud flessibile, conveniente e scalabile per dati non strutturati. Te ne servirai per archiviare i file caricati dall'IU.

1. Dal dashboard, fai clic su [**Create resource**](https://{DomainName}/catalog/) e seleziona [**{{site.data.keyword.cos_short}}**](https://{DomainName}/catalog/services/cloud-object-storage) dalla sezione Storage.
2. Denomina il servizio `myobjectstorage` e fai clic su **Create**.
3. Fai clic su **Create Bucket**.
4. Imposta il nome del bucket su un nome univoco come `username-mybucket`.
5. Seleziona **Cross Region** come resilienza (Resiliency) e **us-geo** come ubicazione (Location) e fai clic su **Create**
6. Fornisci le credenziali del servizio al tuo cluster eseguendo il bind dell'istanza del servizio al tuo spazio dei nomi Kubernetes `default`.
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## Distribuisci l'applicazione IU al cluster

L'applicazione IU è una semplice applicazione web Express Node.js che consente all'utente di caricare i file. Archivia i file nell'istanza Object Storage creata in precedenza e invia quindi un messaggio all'argomento {{site.data.keyword.messagehub}} "work-topic" che indica che un nuovo file è pronto per essere elaborato.

1. Clona il repository dell'applicazione di esempio localmente e cambia di directory passando alla cartella `pubsub-ui`.
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. Apri `config.js` e aggiorna COSBucketName con il tuo nome bucket.
3. Crea e distribuisci l'applicazione. Il comando deploy genera un'immagine docker, ne esegue il push al tuo {{site.data.keyword.registryshort_notm}} e crea quindi una distribuzione Kubernetes. Attieniti alle istruzioni interattive durante la distribuzione dell'applicazione.
```sh
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
4. Visita l'applicazione e carica i file dalla cartella `sample-files`. I file caricati saranno archiviati in Object Storage e lo stato sarà "awaiting" finché non verranno elaborati dall'applicazione nodo di lavoro. Lascia aperta questa finestra del browser.

   ![](images/solution25/files_uploaded.png)

## Distribuisci l'applicazione nodo di lavoro al cluster

L'applicazione nodo di lavoro è un'applicazione Java che è in ascolto per l'argomento "work-topic" di Kafka {{site.data.keyword.messagehub}} per i messaggi. Su un nuovo messaggio, il nodo di lavoro richiamerà il nome del file dal messaggio e otterrà quindi il contenuto del file da Object Storage. Simulerà quindi l'elaborazione del file e, una volta completata l'operazione, invierà un altro messaggio all'argomento "result-work". L'applicazione IU sarà in ascolto per questo argomento e aggiornerà lo stato.

1. Passa alla directory `pubsub-worker`
```sh
  cd ../pubsub-worker
```
2. Apri `resources/cos.properties` e aggiorna la proprietà `bucket.name` con il tuo nome bucket.
2. Crea e distribuisci l'applicazione nodo di lavoro.
```
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
3. Una volta completata la distribuzione, controlla nuovamente la finestra del browser con la tua applicazione web. Nota che lo stato accanto a ciascun file è stato modificato in "processed".
![](images/solution25/files_processed.png)

In questa esercitazione, hai imparato come puoi utilizzare {{site.data.keyword.messagehub}} basato su Kafka per implementare un pattern produttore-consumatore. Ciò consente all'applicazione web di essere rapida e di scaricare l'elaborazione pesante su altre applicazioni. Quando è necessario svolgere del lavoro, il produttore (applicazione web) crea dei messaggi e il carico del lavoro viene bilanciato tra uno o più nodi di lavoro che sottoscrivono i messaggi. Hai utilizzato un'applicazione Java in esecuzione su Kubernetes per gestire l'elaborazione, ma tali applicazioni possono anche essere [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing). Le applicazioni in esecuzione su Kubernetes sono ideali per carichi di lavoro intensivi e a lunga esecuzione, laddove {{site.data.keyword.openwhisk_short}} risulta essere una scelta migliore per i processi di breve durata.

## Rimuovi le risorse
{:removeresources}

Vai a [Resource List](https://{DomainName}/resources/) e
1. elimina il cluster Kubernetes `mycluster`
2. elimina {{site.data.keyword.cos_full_notm}} `myobjectstorage`
3. elimina {{site.data.keyword.messagehub}} `mymessagehub`
4. seleziona **Kubernetes** dal menu a sinistra, **Registry** ed elimina quindi i repository `pubsub-xxx`.

## Contenuto correlato
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [Gestisci l'accesso a Object Storage](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [Elaborazione di dati {{site.data.keyword.messagehub}} con {{site.data.keyword.openwhisk_short}}](https://github.com/IBM/openwhisk-data-processing-message-hub)
