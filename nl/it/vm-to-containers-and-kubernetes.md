---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-15"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Spostamento di un'applicazione basata sulla VM in Kubernetes
{: #vm-to-containers-and-kubernetes}

Questa esercitazione ti guida nel processo di spostamento di un'applicazione basata sulla VM in un cluster Kubernetes utilizzando il {{site.data.keyword.containershort_notm}}. [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) offre potenti strumenti combinando le tecnologie Docker e Kubernetes, un'esperienza utente intuitiva e la sicurezza e l'isolamento integrati per automatizzare la distribuzione, il funzionamento, il ridimensionamento e il monitoraggio di applicazioni inserite in un contenitore in un cluster di host di calcolo.
{: shortdesc}

Le lezioni presenti in questa esercitazione includono i concetti su come prendere un'applicazione esistente, inserirla in un contenitore e distribuirla in un cluster Kubernetes. Per inserire in un contenitore la tua applicazione basata sulla VM, puoi scegliere tra le seguenti opzioni. 

1. Identifica i componenti di un'applicazione monolitica di grandi dimensioni che possono essere separati nel loro microservizio. Puoi inserire in un contenitore questi microservizi e distribuirli in un cluster Kubernetes.
2. Inserisci in un contenitore l'intera applicazione e distribuiscila in un cluster Kubernetes.

A seconda del tipo di applicazione di cui disponi, i passi per eseguire la migrazione della tua applicazione possono variare. Puoi utilizzare questa esercitazione per acquisire informazioni sui passi generali che devi eseguire e sugli elementi che devi tenere in considerazione prima di migrare la tua applicazione.

## Obiettivi
{: #objectives}

- Comprendere in che modo identificare i microservizi in un'applicazione basata sulla VM e in che modo associare i componenti tra le VM e Kubernetes.
- Apprendere come inserire in un contenitore un'applicazione basata sulla VM. 
- Apprendere come distribuire il contenitore in un cluster Kubernetes in {{site.data.keyword.containershort_notm}}.
- Mettere in pratica quanto appreso, eseguire l'applicazione **JPetStore** nel tuo cluster.

## Servizi utilizzati
{: #products}

Questa esercitazione utilizza i seguenti servizi {{site.data.keyword.Bluemix_notm}}:

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**Attenzione:** questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura
{:#architecture}

### Architettura dell'applicazione tradizionale con le VM

Il seguente diagramma mostra un esempio di architettura dell'applicazione tradizionale che si basa sulle VM. 

<p style="text-align: center;">
![Diagramma architettura](images/solution30/traditional_architecture.png)

</p>

1. L'utente invia una richiesta all'endpoint pubblico dell'applicazione Java. L'endpoint pubblico viene rappresentato da un servizio del programma di bilanciamento del carico che bilancia il carico del traffico in entrata tra le istanze del server delle applicazioni disponibili. 
2. Il programma di bilanciamento del carico seleziona una delle istanze del server delle applicazioni integre che vengono eseguite su una VM e inoltra la richiesta. 
3. Il server delle applicazioni archivia i dati dell'applicazione in un database MySQL che viene eseguito su una VM. Le istanze del server delle applicazioni ospitano l'applicazione Java e vengono eseguite su una VM. I file dell'applicazione, ad esempio il codice dell'applicazione, i file di configurazione e le dipendenze, vengono archiviati nella VM. 

### Architettura inserita in un contenitore

Il seguente diagramma mostra un esempio di architettura moderna di un contenitore che viene eseguita in un cluster Kubernetes.

<p style="text-align: center;">
![Diagramma architettura](images/solution30/modern_architecture.png)

</p>

1. L'utente invia una richiesta all'endpoint pubblico dell'applicazione Java. L'endpoint pubblico viene rappresentato da un programma di bilanciamento del carico dell'applicazione (ALB) Ingress che bilancia il carico del traffico di rete in entrata tra i pod dell'applicazione nel cluster. L'ALB è una raccolta di regole che consentono il traffico di rete in entrata a un'applicazione esposta pubblicamente. 
2. L'ALB inoltra la richiesta a uno dei pod dell'applicazione disponibili nel cluster. I pod dell'applicazione vengono eseguiti sui nodi di lavoro che possono essere una VM o una macchina fisica. 
3. I pod dell'applicazione archiviano i dati i volumi persistenti. I volumi persistenti possono essere utilizzati per condividere i dati tra le istanze dell'applicazione o i nodi di lavoro. 
4. I pod dell'applicazione archiviano i dati in un servizio database {{site.data.keyword.Bluemix_notm}}. Puoi eseguire il tuo database all'interno del cluster Kubernetes, ma, di solito, utilizzando un DBaaS (database-as-a-service) gestito è più facile configurare e fornire il ridimensionamento e i backup integrati. Puoi trovare molti tipi diversi di database nel [catalogo IBM Cloud](https://{DomainName}/catalog/?category=data). 

### VM, contenitori e Kubernetes

{{site.data.keyword.containershort_notm}} fornisce la capacità di eseguire le applicazioni inserite in un contenitore nei cluster Kubernetes e offre i seguenti strumenti e le seguenti funzioni: 

- Esperienza utente intuitiva e potenti strumenti
- Sicurezza e isolamento integrati per consentire la distribuzione rapida delle applicazioni sicure
- Servizi Cloud che includono le capacità cognitive di IBM® Watson™
- Capacità di gestire le risorse cluster dedicate sia per le applicazioni senza stato che per i carichi di lavoro con stato

#### Messa a confronto di VM e contenitori

Le **VM**, applicazioni tradizionali che vengono eseguite sull'hardware nativo. Di norma, una singola applicazione non utilizza le risorse complete di un singolo host di calcolo. La maggior parte delle organizzazioni prova a eseguire più applicazioni su un singolo host di calcolo per evitare di sprecare risorse. Puoi eseguire più copie della stessa applicazione, ma per fornire l'isolamento, puoi utilizzare le VM per eseguire più istanze dell'applicazione (VM) sullo stesso hardware. Queste VM hanno stack del sistema operativo completi che le rendono relativamente grandi e inefficienti a causa della duplicazione sia nel runtime che sul disco.

I **contenitori** rappresentano un modo standard per assemblare le applicazioni e tutte le relative dipendenze in modo che tu possa spostare facilmente le applicazioni tra gli ambienti. A differenza delle macchine virtuali, i contenitori non includono il sistema operativo. Nei contenitori vengono riuniti solo il codice dell'applicazione, il runtime, gli strumenti di sistema, le librerie e le impostazioni. I contenitori sono più leggeri, portatili ed efficienti rispetto alle macchine virtuali. 

Inoltre, i contenitori ti consentono di condividere il sistema operativo dell'host. Ciò riduce la duplicazione mentre fornisce ancora l'isolamento. I contenitori ti consentono di eliminare i file non necessari come le librerie di sistema e i file binari per risparmiare spazio e ridurre la tua superficie di attacco. Leggi ulteriori informazioni sulle VM e sui contenitori [qui](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html).

#### Orchestrazione Kubernetes

[Kubernetes](http://kubernetes.io/) è un orchestratore del contenitore per gestire il ciclo di vita delle applicazioni inserite in un contenitore in un cluster di nodi di lavoro. Le tue applicazioni potrebbero aver bisogno di altre risorse per l'esecuzione, ad esempio, i volumi, le reti e i segreti che ti aiuteranno a connetterti ad altri servizi cloud, e le chiavi protette. Kubernetes ti aiuta ad aggiungere queste risorse alla tua applicazione. Il paradigma chiave di Kubernetes è il suo modello dichiarativo. L'utente fornisce lo stato desiderato e Kubernetes tenta di conformarsi allo stato descritto e poi lo mantiene. 

Questo [workshop di autoapprendimento](https://github.com/IBM/kube101/blob/master/workshop/README.md) può aiutarti nella tua prima esperienza pratica con Kubernetes. Inoltre, consulta la pagina della documentazione sui [concetti](https://kubernetes.io/docs/concepts/) di Kubernetes per acquisire ulteriori informazioni sui concetti di Kubernetes.

### Cosa sta facendo IBM per te

Utilizzando i cluster Kubernetes con {{site.data.keyword.containerlong_notm}}, usufruisci dei seguenti vantaggi: 

- Più data center in cui puoi distribuire i tuoi cluster.
- Supporto per le opzioni di rete ingress e del programma di bilanciamento del carico. 
- Supporto del volume persistente dinamico. 
- Master Kubernetes gestiti da IBM altamente disponibili. 

## Dimensionamento dei cluster
{: #sizing_clusters}

Quando progetti la tua architettura cluster, vuoi bilanciare i costi in rapporto a disponibilità, affidabilità, complessità e ripristino. I cluster Kubernetes in {{site.data.keyword.containerlong_notm}} forniscono opzioni di architettura basate sulle esigenze delle tue applicazioni. Con un po' di pianificazione, puoi ottenere il massimo dalle tue risorse cloud senza eccedere nella progettazione dell'architettura o nella spesa. Anche se dovessi sbagliare la stima, puoi facilmente ampliare o ridurre il tuo cluster con altri nodi di lavoro o nodi di lavoro più grandi. 

Per eseguire un'applicazione di produzione nel cloud utilizzando Kubernetes, tieni in considerazione i seguenti elementi:

1. Ti aspetti del traffico da una specifica ubicazione geografica? Se sì, seleziona l'ubicazione più vicina a te per prestazioni migliori. 
2. Quante repliche del tuo cluster vuoi per una disponibilità più elevata? Un buon punto di partenza potrebbe essere tre cluster, uno per lo sviluppo, uno per il test e uno per la produzione. Consulta la guida alla soluzione [Procedure ottimali per organizzare gli utenti, i team e le applicazioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments) per la creazione di più ambienti.
3. Di quale [hardware](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes) hai bisogno per i nodi di lavoro? VM o bare metal?
4. Di quanti nodi di lavoro hai bisogno? Ciò dipende molto dal ridimensionamento delle applicazioni, più nodi hai più resiliente sarà la tua applicazione. 
5. Quante repliche devi avere per una disponibilità più elevata? Distribuisci i cluster di replica in più ubicazioni per rendere più disponibile la tua applicazione e per proteggerla dall'inattività causata da un malfunzionamento dell'ubicazione. 
6. Qual è la serie minima di risorse di cui ha bisogno la tua applicazione per iniziare? Potresti voler verificare la tua applicazione per la quantità di memoria e di CPU di cui necessita per l'esecuzione. Il tuo nodo di lavoro deve avere risorse sufficienti per distribuire e avviare l'applicazione. Assicurati poi di impostare le quote di risorse come parte delle specifiche del pod. Questa impostazione è quella che Kubernetes utilizza per selezionare (o pianificare) un nodo di lavoro che ha capacità sufficiente per supportare la richiesta. Stima quanti pod verranno eseguiti sul nodo di lavoro e i requisiti di risorsa per tali pod. Come minimo, il tuo nodo di lavoro deve essere sufficientemente ampio da supportare un pod per l'applicazione. 
7. Quando aumentare il numero di nodi di lavoro? Puoi monitorare l'utilizzo del cluster e aumentare i nodi quando necessario. Vedi questa esercitazione per comprendere come [analizzare i log e monitorare l'integrità delle applicazioni Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana).
8. Hai bisogno di archiviazione ridondante e affidabile? Se sì, crea un'attestazione del volume persistente per l'archiviazione NFS oppure esegui il bind di un servizio database IBM Cloud al tuo pod.

Per rendere più specifico quanto riportato sopra, presupponiamo che tu voglia eseguire un'applicazione web di produzione nel cloud e che prevedi un carico di lavoro da medio a elevato. Esploriamo quali risorse ti servirebbero: 

1. Configura tre cluster, uno per lo sviluppo, uno per il test e uno per la produzione. 
2. I cluster di sviluppo e di test possono iniziare con un'opzione minima di RAM e di CPU (ad esempio, 2 CPU, 4GB di RAM e un nodo di lavoro per ciascun cluster).
3. Per il cluster di produzione, potresti voler avere più risorse per le prestazioni, l'alta disponibilità e la resilienza. Potremmo scegliere un'opzione dedicata o addirittura un bare metal e avere almeno 4 CPU, 16GB di RAM e due nodi di lavoro. 

## Decidi quale opzione database utilizzare
{: #database_options}

Con Kubernetes, hai due opzioni per gestire i database:

1. Puoi eseguire il tuo database all'interno del cluster Kubernetes, per farlo, dovresti creare un microservizio per eseguire il database. Se stai utilizzando l'esempio di database MySQL, devi effettuare quanto segue: 
   - Crea un Dockerfile MySQL, vedi un [Dockerfile MySQL](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile) di esempio qui.
   - Devi utilizzare i segreti per archiviare la credenziale del database. Vedine un esempio [qui](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret).
   - Hai bisogno di un file deployment.yaml con la configurazione del tuo database distribuito in Kubernetes. Vedine un esempio [qui](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml).
2. La seconda opzione sarebbe quella di utilizzare l'opzione DBaaS (database-as-a-service) gestito. Questa opzione di solito è più semplice da configurare e fornisce backup e ridimensionamento integrati. Puoi trovare molti tipi diversi di database nel [catalogo IBM Cloud](https://{DomainName}/catalog/?category=data). Per utilizzare questa opzione, devi effettuare quanto segue: 
   - Crea un DBaaS (database-as-a-service) gestito dal [catalogo IBM Cloud](https://{DomainName}/catalog/?category=data).
   - Archivia le credenziali del database all'interno di un segreto. Apprenderai ulteriori informazioni sui segreti nella sezione "Archivia le credenziali nei segreti Kubernetes". 
   - Utilizza il DBaaS (database-as-a-service) nella tua applicazione. 

## Decidi dove archiviare i file dell'applicazione
{: #decide_where_to_store_data}

{{site.data.keyword.containershort_notm}} fornisce diverse opzioni per archiviare e condividere i dati tra i pod. Non tutte le opzioni di archiviazione offrono lo stesso livello di persistenza e disponibilità in situazioni di emergenza.
{: shortdesc}

### Archiviazione dati non persistente

I contenitori e i pod sono, come progettati, di breve durata e possono avere un malfunzionamento imprevisto. Puoi archiviare i dati nel file system locale di un contenitore. I dati all'interno di un contenitore non possono essere condivisi con altri contenitori o pod e vengono persi quando il contenitore ha un arresto anomalo o viene rimosso. 

### Apprendi come creare l'archiviazione dati persistente per la tua applicazione

Puoi rendere persistenti i dati dell'applicazione e quelli del contenitore in un'[archiviazione file NFS](https://www.ibm.com/cloud/file-storage/details) o in un'[archiviazione blocchi](https://www.ibm.com/cloud/block-storage) utilizzando i volumi persistenti Kubernetes nativi.
{: shortdesc}

Per eseguire il provisioning di un'archiviazione file NFS o di un'archiviazione blocchi, devi richiedere l'archiviazione per il tuo pod creando un'attestazione del volume persistente (PVC o persistent volume claim). Nella tua PVC, puoi scegliere tra le classi di archiviazione predefinite che definiscono il tipo di archiviazione, la dimensione dell'archiviazione in gigabyte, l'IOPS, la politica di conservazione dei dati e le autorizzazioni di lettura e scrittura per la tua archiviazione. Una PVC esegue, in modo dinamico, il provisioning di un volume persistente che rappresenta il dispositivo di archiviazione effettivo in {{site.data.keyword.Bluemix_notm}}. Puoi montare la PVC nel tuo pod per leggere e scrivere nel volume persistente. I dati archiviati nei volumi persistenti sono disponibili anche se il contenitore si arresta in modo anomalo o il pod viene ripianificato. L'archiviazione file NFS e l'archiviazione blocchi che supportano il volume persistente vengono organizzate in cluster da IBM per fornire un'elevata disponibilità per i tuoi dati. 

Per informazioni su come creare una PVC, segui i passi inclusi nella [documentazione dell'archiviazione del {{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage).

### Apprendi come spostare i dati esistenti nell'archiviazione persistente

Per copiare i dati dalla tua macchina locale alla tua archiviazione persistente, devi montare la PVC in un pod. Poi, puoi copiare i dati dalla tua macchina locale a un volume persistente nel tuo pod.
{: shortdesc}

1. Per copiare i dati, devi creare innanzitutto una configurazione simile alla seguente: 

   ```bash
   kind: Pod
   apiVersion: v1
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
          claimName: mypvc
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/mnt/data"
             name: task-pv-storage
   ```
   {: codeblock}

2. Poi, per copiare i dati dalla tua macchina locale al pod, utilizza un comando simile a questo:
   ```
    kubectl cp <local_filepath>/<filename> <namespace>/<pod>:<pod_filepath>
   ```
   {: pre}

3. Copia i dati da un pod nel tuo cluster alla tua macchina locale:
   ```
   kubectl cp <namespace>/<pod>:<pod_filepath>/<filename> <local_filepath>/<filename>
   ```
   {: pre}


### Configura i backup per l'archiviazione persistente

Viene eseguito il provisioning delle condivisioni file e dell'archiviazione blocchi nella stessa ubicazione del tuo cluster. L'archiviazione stessa è ospitata sui server organizzati in cluster da IBM per fornire un'elevata disponibilità. Tuttavia, il backup delle condivisioni file e dell'archiviazione blocchi non viene eseguito automaticamente e potrebbero essere inaccessibili in caso di malfunzionamento dell'intera ubicazione. Per proteggere i tuoi dati da perdite o danneggiamenti, puoi configurare backup periodici che puoi utilizzare per ripristinare i tuoi dati quando necessario. 

Per ulteriori informazioni, vedi le opzioni di [backup e ripristino](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) per l'archiviazione file NFS e l'archiviazione blocchi.

## Prepara il tuo codice
{: #prepare_code}

### Applica i principi a 12 fattori

L'[applicazione a dodici fattori](https://12factor.net/) è una metodologia per creare le applicazioni native cloud. Quando vuoi inserire in un contenitore un'applicazione, sposta questa applicazione nel cloud e orchestrala con Kubernetes, è importante comprendere e applicare alcuni di questi principi. Alcuni di questi principi sono richiesti in {{site.data.keyword.Bluemix_notm}}.
{: shortdesc}

Ecco alcuni dei principi chiave richiesti: 

- **Codebase** - Tutto il codice sorgente e i file di configurazione vengono tracciati all'interno di un sistema di controllo della versione (ad esempio, un repository GIT), ciò è necessario se stai utilizzando una pipeline DevOps per la distribuzione. 
- **Build, release, esecuzione** - L'applicazione a 12 fattori utilizza una rigida separazione tra le fasi di build (creazione), release (rilascio) ed esecuzione. Ciò può essere automatizzato con una pipeline di fornitura DevOps integrata per creare e verificare l'applicazione prima di distribuirla nel cluster. Consulta l'[esercitazione Distribuzione continua su Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) per apprendere come configurare una pipeline di fornitura e di integrazione continua. Tratta la configurazione delle fasi di controllo, creazione, test e distribuzione di origine e mostra come aggiungere le integrazioni come gli scanner della sicurezza, le notifiche e le analisi. 
- **Configurazione** - Tutte le informazioni sulla configurazione sono archiviate in variabili di ambiente. Nessuna credenziale del servizio viene codificata all'interno del codice dell'applicazione. Per archiviare le credenziali, puoi utilizzare i segreti Kubernetes. Di seguito vengono riportate altre informazioni sulle credenziali. 

### Archivia le credenziali nei segreti Kubernetes
{: secrets}

Non consigliamo mai di archiviare le credenziali all'interno del codice dell'applicazione. Kubernetes, invece, fornisce i cosiddetti **["segreti"](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)** che contengono informazioni sensibili come ad esempio password, token OAuth o chiavi ssh. I segreti Kubernetes sono crittografati per impostazione predefinita, ciò li rende un'opzione più sicura e più flessibile per archiviare i dati sensibili rispetto ad archiviare questi dati testualmente in una definizione `pod` o in un'immagine docker.

Un modo per utilizzare i segreti in Kubernetes è eseguire una procedura simile alla seguente: 

1. Crea un file e archivia le credenziali del servizio al suo interno. 
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api_key>
   }
   ```
   {: codeblock}

2. Poi, crea un segreto Kubernetes eseguendo il comando riportato di seguito e verifica che il segreto venga creato utilizzando `kubectl get secrets` dopo aver eseguito il comando riportato di seguito: 

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## Inserisci in un contenitore la tua applicazione
{: #build_docker_images}

Per inserire in un contenitore la tua applicazione, devi creare un'immagine Docker.
{: shortdesc}

Viene creata un'immagine da un [Dockerfile](https://docs.docker.com/engine/reference/builder/#usage) che è un file che contiene le istruzioni e i comandi per creare l'immagine. Nelle sue istruzioni, un Dockerfile potrebbe fare riferimento alle risorse di build che vengono archiviate separatamente, quali ad esempio un'applicazione, la configurazione dell'applicazione e le relative dipendenze. 

Per creare il tuo Dockerfile per la tua applicazione esistente, potresti utilizzare i seguenti comandi:

- FROM - Scegli un'immagine principale per definire il runtime del contenitore. 
- ADD/COPY - Copia il contenuto di una directory nel contenitore.
- WORKDIR - Imposta la directory di lavoro all'interno del contenitore. 
- RUN - Installa i pacchetti software di cui hanno bisogno le applicazioni durante il runtime.
- EXPOSE - Rendi disponibile una porta all'esterno del contenitore. 
- ENV NAME - Definisci le variabili di ambiente. 
- CMD - Definisci i comandi che vengono eseguiti quando avvii il contenitore. 

Le immagini normalmente vengono archiviate in un registro ed è possibile accedervi pubblicamente (registro pubblico) o configurarle con un accesso limitato a un piccolo gruppo di utenti (registro privato). I registri pubblici, come Docker Hub possono essere utilizzati per iniziare a utilizzare Docker e Kubernetes per creare la tua prima applicazione inserita in un contenitore in un cluster. Ma quando si tratta di applicazioni aziendali, utilizza un registro privato, come quello fornito in {{site.data.keyword.registrylong_notm}} per impedire che le tue immagini vengano utilizzate o modificate da utenti non autorizzati. 

Per inserire in un contenitore un'applicazione e archiviarla in {{site.data.keyword.registrylong_notm}}:

1. Dovresti creare un Dockerfile, di seguito viene riportato un esempio di Dockerfile.
   ```
   # Build JPetStore war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # Use WebSphere Liberty base image from the Docker Store
   FROM websphere-liberty:latest

   # Copy war from build stage and server.xml into image
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. Una volta creato un Dockerfile, devi creare un'immagine del contenitore ed eseguirne il push in {{site.data.keyword.registrylong_notm}}. Puoi creare un contenitore utilizzando un comando come il seguente: 
   ```
   ibmcloud cr build -t <image_name> <directory_of_Dockerfile>
   ```
   {: pre}

## Distribuisci la tua applicazione in un cluster Kubernetes
{: #deploy_to_kubernetes}

Dopo aver creato ed eseguito il push di un'immagine del contenitore nel cloud, devi eseguire la distribuzione nel tuo cluster Kubernetes. Per eseguire tale azione, devi creare un file deployment.yaml.
{: shortdesc}

### Apprendi come creare un file deployment.yaml di Kubernetes

Per creare file deployment.yaml di Kubernetes, devi eseguire una procedura simile alla seguente: 

1. Crea un file deployment.yaml, qui troverai un esempio di file [deployment YAML](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml).

2. Nel tuo file deployment.yaml, puoi definire le [quote di risorse](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) per i tuoi contenitori per specificare la quantità di CPU e di memoria di cui ha bisogno ogni contenitore per iniziare correttamente. Se i contenitori hanno quote di risorse specificate, il programma di pianificazione Kubernetes può prendere decisioni migliori sul nodo di lavoro in cui collocare i tuoi pod. 

3. Puoi quindi utilizzare i comandi riportati di seguito per creare e visualizzare la distribuzione e i servizi creati: 

   ```
   kubectl create -f <filepath/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## Riepilogo
{: #summary}

In questa esercitazione, hai appreso quanto segue: 

- Le differenze tra VM, contenitori e Kubernetes.
- Come definire i cluster per i diversi tipi di ambiente (sviluppo, test e produzione).
- Come gestire l'archiviazione dati e l'importanza dell'archiviazione dati persistente. 
- Applicare i principi a 12 fattori alla tua applicazione e utilizzare i segreti per le credenziali in Kubernetes.
- Creare le immagini docker ed eseguirne il push in {{site.data.keyword.registrylong_notm}}.
- Creare file di distribuzione Kubernetes e distribuire l'immagine Docker in Kubernetes.

## Metti in pratica quanto appreso, esegui l'applicazione JPetStore nel tuo cluster
{: #runthejpetstore}

Per mettere in pratica quanto appreso, segui la [demo](https://github.com/ibm-cloud/ModernizeDemo/) per eseguire l'applicazione **JPetStore** nel tuo cluster e per applicare i concetti appresi. L'applicazione JPetStore ha alcune funzionalità estese per consentirti di estendere un'applicazione in Kubernetes tramite i servizi IBM Watson in esecuzione come un microservizio separato.

## Contenuto correlato
{: #related}

- [Introduzione a](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/) con Kubernetes e il {{site.data.keyword.containershort_notm}}.
- Laboratori {{site.data.keyword.containershort_notm}} su [GitHub](https://github.com/IBM/container-service-getting-started-wt).
- [Documentazione](http://kubernetes.io/) principale di Kubernetes.
- [Archiviazione persistente](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) nel {{site.data.keyword.containershort_notm}}.
- La [guida alle soluzioni per le procedure ottimali](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications) per organizzare utenti, team e applicazioni. 
- [Analizza i log e monitora l'integrità dell'applicazione con LogDNA e Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Configura [pipeline di fornitura e integrazione continue](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) per le applicazioni inserite in un contenitore eseguite in Kubernetes.
- Distribuisci il cluster di produzione [in più ubicazioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Utilizza [più cluster in più ubicazioni](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations) per un'elevata disponibilità. 
