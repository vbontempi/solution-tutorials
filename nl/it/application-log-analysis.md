---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Analizza i log e monitora l'integrità dell'applicazione con LogDNA e Sysdig
{: #application-log-analysis}

Questa esercitazione mostra come utilizzare il servizio [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/) per configurare e accedere ai log di un'applicazione Kubernetes distribuita su {{site.data.keyword.Bluemix_notm}}. Distribuirai un'applicazione Python su un cluster di cui è stato eseguito il provisioning su {{site.data.keyword.containerlong_notm}}, configurerai un agent LogDNA, genererai diversi livelli di log dell'applicazione e accederai ai log dei nodi di lavoro, ai log dei pod o ai log di rete. Quindi, cercherai, filtrerai e visualizzerai questi log tramite l'interfaccia utente web di {{site.data.keyword.la_short}}.

Inoltre, installerai anche il servizio [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/) e configurerai l'agent Sysdig per monitorare le prestazioni e l'integrità della tua applicazione e del tuo cluster {{site.data.keyword.containerlong_notm}}.
{:shortdesc}

## Obiettivi
{: #objectives}
* Distribuire un'applicazione a un cluster Kubernetes per generare le voci di log.
* Accedere e analizzare i diversi tipi di log per risolvere e anticipare i problemi.
* Ottenere visibilità operativa sulle prestazioni e sull'integrità della tua applicazione e del cluster che esegue la tua applicazione.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

  ![](images/solution12/Architecture.png)

1. L'utente si connette all'applicazione e genera le voci di log.
1. L'applicazione viene eseguita in un cluster Kubernetes da un'immagine archiviata in {{site.data.keyword.registryshort_notm}}.
1. L'utente configurerà l'agent del servizio {{site.data.keyword.la_full_notm}} per accedere ai log dell'applicazione e a livello di cluster.
1. L'utente configurerà l'agent del servizio {{site.data.keyword.mon_full_notm}} per monitorare l'integrità e le prestazioni del cluster {{site.data.keyword.containerlong_notm}} e dell'applicazione distribuita nel cluster.

## Prerequisiti
{: #prereq}

* [Installa {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - Script per installare docker, kubectl, helm, CLI ibmcloud e i plugin richiesti.
* [Configura la CLI {{site.data.keyword.registrylong_notm}} e il tuo spazio dei nomi del registro](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Concedi le autorizzazioni a un utente per visualizzare i log in LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna)
* [Concedi le autorizzazioni a un utente per visualizzare le metriche in Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig)

## Crea un cluster Kubernetes
{: #create_cluster}

{{site.data.keyword.containershort_notm}} fornisce un ambiente per distribuire applicazioni altamente disponibili nei contenitori Docker eseguiti nei cluster Kubernetes.

Salta questa sezione se hai un cluster **Standard** esistente e vuoi riutilizzarlo con questa esercitazione.
{: tip}

1. Crea **un nuovo cluster** dal [catalogo {{site.data.keyword.Bluemix}}](https://{DomainName}/kubernetes/catalog/cluster/create) e scegli il cluster **Standard**.
   L'inoltro del log *non* è abilitato per il cluster **Gratuito**.
   {:tip}
1. Seleziona un gruppo di risorse e un'area geografica.
1. Per praticità, utilizza il nome `mycluster` per essere coerente con questa esercitazione.
1. Seleziona una zona di lavoro (**Worker Zone**) e seleziona il tipo di macchina (**Machine type**) più piccolo con 2 **CPU** e 4 **GB di RAM** in quanto è sufficiente per questa esercitazione.
1. Seleziona 1 nodo di lavoro (**Worker node**) e lascia tutte le altre opzioni impostate sui valori predefiniti. Fai clic su **Create Cluster**.
1. Controlla lo stato del tuo **Cluster** e **Worker Node** e attendi che diventi **ready**.

## Esegui il provisioning di un'istanza {{site.data.keyword.la_short}}
{: #provision_logna_instance}

Le applicazioni distribuite su un cluster {{site.data.keyword.containerlong_notm}} in {{site.data.keyword.Bluemix_notm}} probabilmente genereranno un qualche livello di output diagnostico, ovvero i log. In qualità di sviluppatore o di operatore, potresti voler accedere e analizzare i diversi tipi di log, come i log dei nodi di lavoro, dei pod, dell'applicazione o di rete, per risolvere e anticipare i problemi.

Utilizzando il servizio {{site.data.keyword.la_short}}, è possibile aggregare i log da varie origini e conservarli per tutto il tempo necessario. Ciò consente di analizzare il "quadro generale" quando richiesto e di risolvere situazioni più complesse.

Per eseguire il provisioning di un servizio {{site.data.keyword.la_short}},

1. Vai alla pagina [Observability](https://{DomainName}/observe/) e, sotto **Logging**, fai clic su **Create instance**.
1. Fornisci un nome servizio (**Service name**) univoco.
1. Scegli una regione/ubicazione e seleziona un gruppo di risorse.
1. Seleziona **7 day Log Search** come tuo tipo di piano e fai clic su **Create**.

Il servizio fornisce un sistema di gestione dei log centralizzato in cui i dati di log sono ospitati su IBM Cloud.

## Distribuisci e configura un'applicazione Kubernetes per inoltrare i log
{: #deploy_configure_kubernetes_app}

Il [codice pronto per l'esecuzione per l'applicazione di registrazione si trova in questo repository GitHub](https://github.com/IBM-Cloud/application-log-analysis). L'applicazione viene scritta utilizzando [Django](https://www.djangoproject.com/), un popolare framework web lato server di Python. Clona o scarica il repository, quindi distribuisci l'applicazione al {{site.data.keyword.containershort_notm}} su {{site.data.keyword.Bluemix_notm}}.

### Distribuisci l'applicazione Python

Su un terminale:
1. Clona il repository GitHub:
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. Passa alla directory dell'applicazione
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. Crea un'immagine Docker con il [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile) in {{site.data.keyword.registryshort_notm}}.
   - Trova il **Container Registry** con `ibmcloud cr info`, ad esempio us.icr.io o uk.icr.io.
   - Crea uno spazio dei nomi per archiviare l'immagine del contenitore.
      ```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - Sostituisci `<CONTAINER_REGISTRY>` con il tuo valore di registro del contenitore e utilizza **app-log-analysis** come nome dell'immagine.
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - Sostituisci il valore **image** nel file `app-log-analysis.yaml` con la tag dell'immagine `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest`
1. Immetti il seguente comando per richiamare la configurazione del cluster e impostare la variabile di ambiente `KUBECONFIG`:
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. Distribuisci l'applicazione:
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. Per accedere all'applicazione, hai bisogno dell'IP pubblico (`public IP`) del nodo di lavoro e della `NodePort`
   - Per l'IP pubblico, immetti il seguente comando:
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - Per la NodePort che sarà di 5 cifre (ad esempio, 3xxxx), immetti il seguente comando:
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   Puoi ora accedere all'applicazione all'indirizzo `http://worker-ip-address:portnumber`

### Configura il cluster per inviare i log alla tua istanza LogDNA

Per configurare il cluster Kubernetes in modo che invii i log alla tua istanza {{site.data.keyword.la_full_notm}}, devi installare un pod *logdna-agent* su ogni nodo del tuo cluster. L'agent LogDNA legge i file di log dal pod su cui è installato e inoltra i dati di log alla tua istanza LogDNA.

1. Vai alla pagina [Observability](https://{DomainName}/observe/) e fai clic su **Logging**.
1. Fai clic su **Edit log resources** accanto al servizio che hai creato in precedenza e seleziona **Kubernetes**.
1. Copia ed esegui il primo comando su un terminale in cui hai impostato la variabile di ambiente `KUBECONFIG` per creare un segreto Kubernetes con la chiave di inserimento LogDNA per la tua istanza del servizio.
1. Copia ed esegui il secondo comando per distribuire un agent LogDNA su ogni nodo di lavoro del tuo cluster Kubernetes. L'agent LogDNA raccoglie i log con l'estensione **.log* e i file senza estensione che si trovano nella directory */var/log* del tuo pod. Per impostazione predefinita, i log vengono raccolti da tutti gli spazi dei nomi, incluso kube-system, e vengono inoltrati automaticamente al servizio {{site.data.keyword.la_full_notm}}.
1. Dopo aver configurato un'origine log, avvia l'interfaccia utente LogDNA facendo clic su **View LogDNA**. Potrebbe essere necessario attendere qualche minuto prima di poter visualizzare i log.

## Genera e accedi ai log dell'applicazione
{: generate_application_logs}

In questa sezione, genererai i log dell'applicazione e li esaminerai in LogDNA.

### Genera i log dell'applicazione

L'applicazione distribuita nei passi precedenti ti consente di registrare un messaggio a un livello di log scelto. I livelli di log disponibili sono **critical**, **error**, **warn**, **info** e **debug**. L'infrastruttura di registrazione dell'applicazione è configurata per consentire il passaggio solo delle voci di log che si trovano al livello impostato o a un livello superiore. Inizialmente, il livello del logger è impostato su **warn**. Pertanto, un messaggio registrato al livello **info** con un'impostazione server di **warn** non viene visualizzato nell'output diagnostico.

Dai un'occhiata al codice nel file [**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py). Il codice contiene istruzioni di **print** e chiamate alle funzioni del **logger**. I messaggi stampati vengono scritti nel flusso **stdout** (output regolare, console dell'applicazione/terminale), mentre i messaggi del logger sono inclusi nel flusso **stderr** (log degli errori).

1. Visita la pagina dell'applicazione web all'indirizzo `http://worker-ip-address:portnumber`.
1. Genera più voci di log inoltrando i messaggi a diversi livelli. L'interfaccia utente consente anche di modificare l'impostazione del logger per il livello di log del server. Modifica il livello di log sul lato server in un punto intermedio per renderlo più interessante. Ad esempio, puoi registrare un messaggio "500 internal server error" come **error** o "This is my first log entry" come **info**.

### Accedi ai log dell'applicazione

Puoi accedere al log specifico per l'applicazione nell'interfaccia utente di LogDNA utilizzando i filtri.

1. Nella barra superiore, fai clic su **All Apps**.
1. In Containers, seleziona **app-log-analysis**. Viene visualizzata una nuova vista non salvata con i log dell'applicazione di tutti i livelli.
1. Per visualizzare i log di specifici livelli, fai clic su **All Levels** e seleziona più livelli come error, info, warning ecc.

## Ricerca e filtra i log
{: #search_filter_logs}

Per impostazione predefinita, l'interfaccia utente di {{site.data.keyword.la_short}} mostra tutte le voci di log disponibili (Everything). Le voci più recenti vengono mostrate in basso tramite un aggiornamento automatico.
In questa sezione, modificherai cosa e quanto viene visualizzato e lo salverai come vista (**View**) per uso futuro.

### Ricerca i log

1. Nella casella di input **Search** che si trova nella parte inferiore della pagina nell'interfaccia utente di LogDNA,
   - puoi ricercare le righe che contengono un testo specifico come **"This is my first log entry"** o **500 internal server error**
   - o uno specifico livello di log immettendo `level:info` dove level è un campo che accetta il valore di stringa.

   Per ulteriori campi di ricerca e assistenza, fai clic sull'icona della guida alla sintassi accanto alla casella di input della ricerca
   {:tip}
1. Per passare a un intervallo di tempo specifico, immetti **5 mins ago** nella casella di input **Jump to timeframe**. Fai clic sull'icona accanto alla casella di input per trovare gli altri formati di ora nel tuo periodo di conservazione.
1. Per evidenziare i termini, fai clic sull'icona **Toggle Viewer Tools**.
1. Immetti **error** come termine di evidenziazione nella prima casella di input, **container** come termine di evidenziazione nella seconda casella di input e controlla le righe evidenziate con i termini.
1. Fai clic sull'icona **Toggle Timeline** per visualizzare le righe con i log in una specifica ora del giorno.

### Filtra i log

Puoi filtrare i log in base a tag, origini, applicazioni o livelli.

1. Nella barra superiore, fai clic su **All Tags** e seleziona la casella di spunta **k8s** per visualizzare i log correlati a Kubernetes.
1. Fai clic su **All Sources** e seleziona il nome dell'host (nodo di lavoro) di cui vuoi controllare i log. Ciò è utile se hai più nodi di lavoro nel tuo cluster.
1. Per controllare i log del contenitore o dei file, fai clic su **All Apps** e seleziona le caselle di spunta che ti interessano per visualizzare i log. 

### Crea una vista

Le viste sono dei collegamenti salvati per una specifica serie di filtri e query di ricerca.

Non appena cerchi o filtri i log, dovresti vedere **Unsaved View** nella barra superiore. Per salvare questo come vista:
1. Fai clic su **All Apps** e seleziona la casella di spunta accanto a **app-log-analysis**
1. Fai clic su **Unsaved View** > quindi su **save as new view / alert** e denomina la vista come **app-log-analysis-view**. Lascia vuoto il campo **Category**.
1. Fai clic su **Save View** e la nuova vista dovrebbe essere visualizzata nel riquadro di sinistra che mostra i log per l'applicazione.

### Visualizza i log con grafici e suddivisioni

In questa sezione, creerai una scheda e quindi aggiungerai un grafico con una suddivisione per visualizzare i dati a livello di applicazione. Una scheda è una raccolta di grafici e suddivisioni.

1. Nel riquadro di sinistra, fai clic sull'icona **board** (sopra l'icona delle impostazioni) > e fai clic su **NEW BOARD**.
1. Fai clic su **Edit** nella barra superiore e denomina la scheda come **app-log-analysis-board**. Fai clic su **Save**.
1. Fai clic su **Add Graph**:
   - Immetti **app** come campo nella prima casella di input e premi Invio.
   - Scegli **app-log-analysis** come tuo valore di campo.
   - Fai clic su **Add Graph**.
1. Seleziona **Counts** come metrica per visualizzare il numero di righe in ogni intervallo nelle ultime 24 ore.
1. Per aggiungere una suddivisione, fai clic sulla freccia sotto il grafico:
   - Scegli **Histogram** come tipo di suddivisione.
   - Scegli **level** come tipo di campo.
   - Fai clic su **Add Breakdown** per visualizzare una suddivisione con tutti i livelli che hai registrato per l'applicazione.

## Aggiungi {{site.data.keyword.mon_full_notm}} e monitora il tuo cluster
{: #monitor_cluster_sysdig}

Nella seguente sezione, aggiungerai {{site.data.keyword.mon_full_notm}} all'applicazione. Il servizio controlla regolarmente la disponibilità e il tempo di risposta dell'applicazione.

1. Vai alla pagina [Observability](https://{DomainName}/observe/) e, sotto **Monitoring**, fai clic su **Create instance**.
1. Fornisci un nome servizio (**Service name**) univoco.
1. Scegli una regione/ubicazione e seleziona un gruppo di risorse.
1. Seleziona **Graduated Tier** come tuo tipo di piano e fai clic su **Create**.
1. Fai clic su **Edit log resources** accanto al servizio che hai creato in precedenza e seleziona **Kubernetes**.
1. Copia ed esegui il comando sotto **Install Sysdig Agent to your cluster** su un terminale in cui hai impostato la variabile di ambiente `KUBECONFIG` per distribuire l'agent Sysdig nel tuo cluster. Attendi il completamento della distribuzione.

### Configura {{site.data.keyword.mon_short}}

Per configurare Sysdig per il monitoraggio dell'integrità e delle prestazioni del tuo cluster:
1. Fai clic su **View Sysdig** per visualizzare l'interfaccia utente di monitoraggio Sysdig. Nella pagina di benvenuto, fai clic su **Next**.
1. Scegli **Kubernetes** come metodo di installazione nella pagina di configurazione dell'ambiente.
1. Fai clic su **Go to Next step** accanto al messaggio di esito positivo della configurazione dell'agent e fai clic su **Let's Get started** nella pagina successiva.
1. Fai clic su **Next** e quindi su **Complete onboarding** per visualizzare la scheda `Explore` dell'interfaccia utente di Sysdig.

### Monitora il tuo cluster

Per controllare l'integrità e le prestazioni della tua applicazione e del tuo cluster:
1. Torna all'applicazione in esecuzione in `http://worker-ip-address:portnumber` e genera diverse voci di log.
1. Espandi **mycluster** nel riquadro di sinistra > espandi lo spazio dei nomi **default** > fai clic su **app-log-analysis-deployment** per vedere il conteggio delle richieste, il tempo di risposta ecc. nella procedura guidata di monitoraggio Sysdig.
1. Per controllare i codici di richiesta-risposta HTTP, fai clic sulla freccia accanto a **Kubernetes Pod Health** nella barra superiore e seleziona **HTTP** sotto **Applications**. Modifica l'intervallo in **10 M** nella barra inferiore dell'interfaccia utente di Sysdig.
1. Per monitorare la latenza dell'applicazione:
   - Dalla scheda Explore, seleziona **Deployments and Pods**.
   - Fai clic sulla freccia accanto a `HTTP` e quindi seleziona Metrics > Network.
   - Seleziona **net.http.request.time**.
   - Seleziona Time: **Sum** e Group: **Average**.
   - Fai clic su **More options** e quindi sul'icona **Topology**.
   - Fai clic su **Done** e fai doppio clic sulla casella per espandere la vista.
1. Per monitorare lo spazio dei nomi Kubernetes in cui è in esecuzione l'applicazione,
   - Dalla scheda Explore, seleziona **Deployments and Pods**.
   - Fai clic sulla freccia accanto a `net.http.request.time`.
   - Seleziona Default Dashboards > Kubernetes.
   - Seleziona Kubernetes State > Kubernetes State Overview.

### Crea un dashboard personalizzato

Insieme ai dashboard predefiniti, puoi creare il tuo dashboard personalizzato per visualizzare le viste e le metriche più utili/pertinenti per i contenitori che eseguono la tua applicazione in un'unica posizione. Ogni dashboard è composto da una serie di pannelli configurati per visualizzare specifici dati in numerosi formati diversi.

Per creare un dashboard:
1. Fai clic su **Dashboards** nel riquadro più a sinistra > fai clic su **Add Dashboard**.
1. Fai clic su **Blank Dashboard** > denomina il dashboard come **Container Request Overview** > fai clic su **Create Dashboard**.
1. Seleziona **Top List** come nuovo pannello e fornisci un nome per il pannello come **Request time per container**
   - In **Metrics**, immetti **net.http.request.time**.
   - Ambito: fai clic su **Override Dashboard Scope** > seleziona **container.image** > seleziona **is** > seleziona _l'immagine dell'applicazione_
   - Segmenta per **container.id** e dovresti vedere il tempo di richiesta netto di ogni contenitore.
   - Fai clic su **save**.
1. Per aggiungere un nuovo pannello, fai clic sull'icona **plus** e seleziona **Number(#)** come tipo di pannello
   - In **Metrics**, immetti **net.http.request.count** > modifica l'aggregazione temporale da **Average(Avg)** a **Sum**.
   - Ambito: fai clic su **Override Dashboard Scope** > seleziona **container.image** > seleziona **is** > seleziona _l'immagine dell'applicazione_
   - Confronta con **1 ora** fa e dovresti vedere il conteggio di richieste netto di ogni contenitore.
   - Fai clic su **save**.
1. Per modificare l'ambito di questo dashboard,
   - Fai clic su **Edit Scope** nel pannello del titolo.
   - Seleziona/immetti **Kubernetes.cluster.name** nel menu a discesa
   - Lascia vuoto il nome di visualizzazione e seleziona **is**.
   - Seleziona **mycluster** come valore e fai clic su **Save**.

## Rimuovi le risorse
{: #remove_resource}

- Rimuovi le istanze di LogDNA e Sysdig dalla pagina [Observability](https://{DomainName}/observe).
- Elimina il cluster incluso il nodo di lavoro, l'applicazione e i contenitori. Questa azione non potrà essere annullata.
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## Espandi l'esercitazione
{: #expand_tutorial}

- Utilizza il [servizio {{site.data.keyword.at_full}}](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started) per tracciare come le applicazioni interagiscono con i servizi IBM Cloud.
- [Aggiungi avvisi](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts) alla tua vista.
- [Esporta i log](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export) in un file locale.

## Contenuto correlato
{:related}
- [Reimpostazione della chiave di inserimento utilizzata da un cluster Kubernetes](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [Archiviazione dei log in IBM Cloud Object Storage](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [Configurazione degli avvisi in Sysdig](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [Utilizzo dei canali di notifica nell'interfaccia utente di Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
