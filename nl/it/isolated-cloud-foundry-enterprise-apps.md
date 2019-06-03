---
copyright:
  years: 2019
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

# Applicazioni Cloud Foundry Enterprise isolate
{: #isolated-cloud-foundry-enterprise-apps}

Con {{site.data.keyword.cfee_full_notm}} (CFEE) puoi creare, su richiesta, più piattaforme Cloud Foundry di livello enterprise e isolate. Con questo, ottieni un'istanza Cloud Foundry privata distribuita su un cluster Kubernetes isolato. A differenza del Cloud pubblico, avrai il pieno controllo su ambiente, controllo dell'accesso, capacità, versione, utilizzo delle risorse e monitoraggio. {{site.data.keyword.cfee_full_notm}} fornisce la velocità e l'innovazione di una PaaS (platform-as-a-service) con la proprietà dell'infrastruttura che si trova nell'IT aziendale.

Un caso d'uso per {{site.data.keyword.cfee_full_notm}} è una piattaforma di innovazione appartenente all'azienda. Tu, in quanto uno sviluppatore in un'azienda, puoi creare nuovi microservizi o migrare le applicazioni legacy a CFEE. I microservizi possono essere quindi pubblicati per ulteriori sviluppatori utilizzando il marketplace Cloud Foundry. Da lì, gli sviluppatori nella tua istanza CFEE possono consumare i servizi nell'applicazione proprio come si fa oggi sul Cloud pubblico.

L'esercitazione ti guiderà nel processo di creazione e configurazione di un {{site.data.keyword.cfee_full_notm}}, configurando il controllo dell'accesso e distribuendo applicazioni e servizi. Riesaminerai anche la relazione tra CFEE e [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) distribuendo un broker di servizi personalizzato che integra i servizi personalizzati con CFEE.

## Obiettivi

{: #objectives}

- Comparare e raffrontare CFEE con Cloud Foundry pubblico
- Distribuire le applicazioni e i servizi all'interno di CFEE
- Comprendere la relazione tra Cloud Foundry e {{site.data.keyword.containershort_notm}}
- Analizzare la rete {{site.data.keyword.containershort_notm}} e Cloud Foundry di base

## Servizi utilizzati

{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura

{: #architecture}

![Architettura](./images/solution45-CFEE-apps/Architecture.png)

1. L'amministratore crea un'istanza CFEE e aggiunge gli utenti con l'accesso di sviluppatore
2. Lo sviluppatore esegue il push di un'applicazione starter Node.js a CFEE.
3. L'applicazione starter Node.js utilizza [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) per archiviare i dati.
4. Lo sviluppatore aggiunge un nuovo servizio di messaggio di benvenuto.
5. L'applicazione starter Node.js esegue il bind del nuovo servizio da un broker di servizi personalizzato.
6. L'applicazione starter Node.js visualizza il benvenuto in diverse lingue dal servizio.

## Prerequisiti

{: #prereq}

- [CLI {{site.data.keyword.cloud_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [CLI Cloud Foundry](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## Esegui il provisioning di {{site.data.keyword.cfee_full_notm}}

{:provision_cfee}

In questa sezione, creerai un'istanza di {{site.data.keyword.cfee_full_notm}} distribuita ai nodi di lavoro Kubernetes da {{site.data.keyword.containershort_notm}}.

1. [Prepara il tuo account {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare) per garantire la creazione delle risorse dell'infrastruttura necessarie.
2. Dal catalogo {{site.data.keyword.cloud_notm}}, crea un'istanza del servizio di [{{site.data.keyword.cfee_full_notm}} ](https://{DomainName}/cfadmin/create).
3. Configura CFEE fornendo quanto segue:
   - Seleziona il piano **Standard**.
   - Immetti un nome (**Name**) per l'istanza del servizio.
   - Seleziona un gruppo di risorse (**Resource group**) in cui viene creato l'ambiente. Avrai bisogno dell'autorizzazione per accedere ad almeno un gruppo di risorse nell'account per poter creare un'istanza CFEE.
   - Seleziona un'area geografica (**Geography**) e un'ubicazione (**Location**) dove viene distribuita l'istanza. Vedi l'elenco di [ubicazioni e data center di provisioning disponibili](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets).
   - Seleziona un'organizzazione (**Organization**) e uno spazio (**Space**) di Cloud Foundry pubblico dove verrà distribuito **{{site.data.keyword.composeForPostgreSQL}}**.
   - Seleziona il numero di celle (**Number of cells**) per l'ambiente Cloud Foundry. Una cella esegue le applicazioni Diego e Cloud Foundry. Sono necessarie almeno **2** celle per le applicazioni altamente disponibili.
   - Seleziona il tipo di macchina (**Machine type**), che determina la dimensione delle celle Cloud Foundry (CPU e memoria).
4. Esamina la sezione **Infrastructure** per visualizzare le proprietà del cluster Kubernetes che supporta CFEE. Il numero di nodi di lavoro (**Number of worker nodes**) equivale al numero di celle più 2. Due dei nodi di lavoro Kubernetes di cui è stato eseguito il provisioning fungono da piano di controllo CFEE. Il cluster Kubernetes su cui è distribuito l'ambiente sarà visualizzato nel dashboard {{site.data.keyword.cloud_notm}} [Clusters](https://{DomainName}/containers-kubernetes/clusters).
5. Fai clic sul pulsante **Create** per iniziare la distribuzione automatizzata.

La distribuzione automatizzata impiega circa 90-120 minuti. Una volta eseguita correttamente l'operazione di creazione, riceverai una email che conferma il provisioning di CFEE e dei servizi di supporto.

### Crea organizzazioni e spazi

Dopo che hai creato {{site.data.keyword.cfee_full_notm}}, vedi il documento relativo alla [creazione di organizzazioni e spazi](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs) per informazioni su come strutturare l'ambiente mediante organizzazioni e spazi. L'ambito delle applicazioni in un {{site.data.keyword.cfee_full_notm}} viene definito all'interno di spazi specifici. In modo analogo, esiste uno spazio all'interno del contesto di un'organizzazione. I membri di un'organizzazione condividono un piano di quota, le applicazioni, le istanze dei servizi e i domini personalizzati.

Nota: il menu **Manage > Account > Cloud Foundry orgs** che si trova nell'intestazione {{site.data.keyword.cloud_notm}} superiore è concepito esclusivamente per le organizzazioni {{site.data.keyword.cloud_notm}} pubbliche. Le organizzazioni CFEE vengono gestite all'interno della pagina **organizations** di un'istanza CFEE.

Attieniti alla seguente procedura per creare un'organizzazione e uno spazio CFEE.

1. Dal [dashboard Cloud Foundry](https://{DomainName}/dashboard/cloudfoundry/overview) seleziona **Environments** in **Enterprise**.
2. Seleziona la tua istanza CFEE e seleziona quindi **Organizations**.
3. Fai clic sul pulsante **Create Organizations**, fornisci `tutorial` come nome dell'organizzazione (**Organization Name**) e seleziona un **Quota Plan**. Termina facendo clic su **Add**.
4. Fai clic sull'organizzazione `tutorial` appena creata, seleziona la scheda **Spaces** e fai clic sul pulsante **Create Space**.
5. Fornisci `dev` come nome dello spazio (**Space Name**) e fai clic su **Add**.

### Aggiungi utenti a organizzazioni e spazi

In CFEE, puoi assegnare i ruoli che controllano l'accesso utente ma, per farlo, l'utente deve essere invitato al tuo account {{site.data.keyword.cloud_notm}} nella pagina **Identity & Access** in **Manage > Users** nell'intestazione {{site.data.keyword.cloud_notm}}.

Dopo che l'utente è stato invitato, attieniti alla procedura di seguito indicata per aggiungerlo all'organizzazione `tutorial` creata. 

1. Seleziona la tua [istanza CFEE](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) e seleziona quindi nuovamente **Organizations**.
2. Seleziona l'organizzazione `tutorial` creata dall'elenco.
3. Fai clic sulla scheda **Members** per visualizzare e aggiungere un nuovo utente.
4. Fai clic sul pulsante **Add members**, cerca il nome utente, seleziona i ruoli dell'organizzazione (**Organization Roles**) appropriati e fai clic su **Add**.

Ulteriori informazioni sull'aggiunta di utenti a organizzazioni e spazi CFEE sono disponibili [qui](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users).

## Distribuisci, configura ed esegui applicazioni CFEE

{:deploycfeeapps}

In questa sezione, distribuirai un'applicazione Node.js a CFEE. Dopo che ne è stata eseguita la distribuzione, procederai ad associarvi mediante bind un {{site.data.keyword.cloudant_short_notm}} e abiliterai la persistenza di controllo e registrazione. Verrà aggiunta anche la console Stratos per gestire l'applicazione.

### Distribuisci l'applicazione a CFEE

1. Dal tuo terminale, clona l'applicazione di esempio [get-started-node](https://github.com/IBM-Cloud/get-started-node).
   ```sh
   git clone https://github.com/IBM-Cloud/get-started-node
   ```
   {: codeblock}
2. Esegui l'applicazione in locale per assicurarti che venga creata e avviata correttamente. Procedi alla conferma accedendo a `http://localhost:3000/` nel tuo browser.
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. Accedi a {{site.data.keyword.cloud_notm}} e indica come destinazione la tua istanza CFEE. Un prompt interattivo ti aiuterà a selezionare la tua nuova istanza CFEE. Poiché esistono solo una singola organizzazione e un singolo spazio CFEE, saranno la destinazione predefinita. Puoi eseguire `ibmcloud target -o tutorial -s dev` se hai aggiunto più di una singola organizzazione o un singolo spazio.
   ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
   {: codeblock}
4. Esegui il push dell'applicazione **get-started-node** a CFEE.
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. L'endpoint della tua applicazione verrà visualizzato nell'output finale accanto alla proprietà `routes` . Apri l'URL nel tuo browser per confermare che l'applicazione è in esecuzione.

### Crea il database Cloudant ed eseguine il bind all'applicazione

Per eseguire il bind dei servizi {{site.data.keyword.cloud_notm}} all'applicazione **get-started-node**, dovrai prima creare un servizio nel tuo account {{site.data.keyword.cloud_notm}}.

1. Crea un [servizio {{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant). Fornisci il **nome servizio** `cfee-cloudant` e scegli la stessa ubicazione dove è stata creata l'istanza CFEE.
2. Aggiungi l'istanza del servizio {{site.data.keyword.cloudant_short_notm}} appena creata a CFEE.
   1. Torna all'organizzazione (**Organization**) `tutorial`  Fai clic sulla scheda **Spaces** e seleziona lo spazio `dev`.
   2. Seleziona la scheda **Services** e fai clic sul pulsante **Add Service**.
   3. Immetti `cfee-cloudant` nella casella di testo di ricerca e seleziona il risultato. Termina facendo clic su **Add**.Il servizio è ora disponibile alle applicazioni CFEE; tuttavia, continua a trovarsi in {{site.data.keyword.cloud_notm}} pubblico.
3. Nel menu di overflow dell'istanza del servizio visualizzata, seleziona **Bind to application**.
4. Seleziona l'applicazione **GetStartedNode** di cui hai eseguito il push in precedenza e fai clic su **Restage application after binding**. Infine, fai clic sul pulsante **Bind**. Attendi che l'applicazione venga ripreparata. Puoi controllare lo stato di avanzamento con il comando `ibmcloud app show GetStartedNode`.
5. Nel tuo browser, accedi all'applicazione, aggiungi il tuo nome e premi `Invio`. Il tuo nome verrà aggiunto a un database {{site.data.keyword.cloudant_short_notm}}.
6. Conferma selezionando l'istanza `tutorial` dall'elenco nella scheda **Services** . In questo modo si aprirà la pagina dei dettagli dell'istanza del servizio in {{site.data.keyword.cloud_notm}} pubblico.
7. Fai clic su **Launch Cloudant Dashboard** e seleziona il database `mydb`. Dovrebbe esistere un documento JSON con il tuo nome.

### Abilita la persistenza del controllo e della registrazione

Il controllo consente agli amministratori CFEE di tenere traccia delle attività Cloud Foundry come l'accesso, la creazione di organizzazioni e spazi, le assegnazioni di ruoli e appartenenze agli utenti, le distribuzioni di applicazioni, i bind di servizi e la configurazione di domini. Il controllo è supportato tramite l'integrazione con il servizio {{site.data.keyword.cloudaccesstrailshort}}.

I log delle applicazioni Cloud Foundry possono essere archiviati integrando {{site.data.keyword.loganalysisshort_notm}}. L'istanza del servizio {{site.data.keyword.loganalysisshort_notm}} selezionata da un amministratore CFEE è configurata automaticamente per ricevere e conservare in modo persistente gli eventi di registrazione Cloud Foundry generati dall'istanza CFEE.

Per abilitare la persistenza del controllo e della registrazione CFEE, attieniti a [questa procedura](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging).

### Installa la console Stratos per gestire l'applicazione

La console Stratos è uno strumento open source basato sul web per lavorare con Cloud Foundry. L'applicazione console Stratos può essere installata facoltativamente e utilizzata in un ambiente CFEE specifico per gestire organizzazioni, spazi e applicazioni.

Per installare l'applicazione console Stratos:

1. Apri l'istanza CFEE in cui vuoi installare la console Stratos.
2. Nella pagina **Overview**, fai clic su **Install Stratos Console**. Il pulsante è visibile solo agli utenti con autorizzazioni di amministratore o editor.
3. Nella finestra di dialogo Install Stratos Console, seleziona un'opzione di installazione. Puoi installare l'applicazione della console Stratos sul piano di controllo CFEE o in una delle celle. Seleziona una versione della console Stratos e il numero di istanze dell'applicazione da installare. Se installi l'applicazione della console Stratos in una cella, ti vengono richiesti l'organizzazione e lo spazio in cui distribuire l'applicazione.
4. Fai clic su **Install**.

Per installare l'applicazione sono necessari circa 5 minuti. Una volta completata l'installazione, viene visualizzato un pulsante **Stratos Console** al posto del pulsante **Install Stratos Console** nella pagina di panoramica. Ulteriori informazioni sulla console Stratos sono disponibili [qui](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app).

## Relazione tra CFEE e Kubernetes

Come piattaforma per applicazioni, CFEE viene eseguito su una qualche forma di infrastruttura virtuale dedicata o condivisa. Per molti anni, gli sviluppatori si sono preoccupati poco della piattaforma Cloud Foundry sottostante perché IBM l'ha gestita per loro. Con CFEE, non sei solo uno sviluppatore che scrive le applicazioni Cloud Foundry ma anche un operatore della piattaforma Cloud Foundry. Questo perché CFEE è distribuito su un cluster Kubernetes che tu controlli.

Anche se gli sviluppatori Cloud Foundry possono avere poca dimestichezza con Kubernetes, sono molti i concetti che entrambi hanno in comune. Come Cloud Foundry, Kubernetes isola le applicazioni in contenitori, che vengono eseguiti all'interno di un costrutto Kubernetes chiamato pod. Analogamente alle istanze dell'applicazione, i pod possono avere più copie (chiamate set di repliche) con il bilanciamento del carico dell'applicazione fornito da Kubernetes. L'applicazione `GetStartedNode` Cloud Foundry che hai distribuito in precedenza viene eseguita all'interno del pod `diego-cell-0`. Per supportare l'alta disponibilità, un altro pod `diego-cell-1` sarà eseguito su un nodo di lavoro Kubernetes separato. Poiché queste applicazioni Cloud Foundry vengono eseguite "all'interno" di Kubernetes, puoi anche comunicare con altri microservizi Kubernetes utilizzando la rete basata su Kubernetes. Le seguenti sezioni aiuteranno ad illustrare in modo più dettagliato le relazioni tra CFEE e Kubernetes.

## Distribuisci un broker di servizi Kubernetes

In questa sezione, distribuirai un microservizio a Kubernetes che funge da broker di servizi per CFEE. I [broker di servizi](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md) forniscono i dettagli sui servizi disponibili nonché il bind e il provisioning del supporto alla tua applicazione Cloud Foundry. Hai utilizzato un broker di servizi {{site.data.keyword.cloud_notm}} integrato per aggiungere il servizio {{site.data.keyword.cloudant_short_notm}}. Ora ne distribuirai, e utilizzerai, uno personalizzato. Modificherai quindi l'applicazione `GetStartedNode` per utilizzare il broker di servizi, che restituirà un messaggio di benvenuto in diverse lingue.

1. Tornando al tuo terminale, clona i progetti che forniscono i file di distribuzione Kubernetes e l'implementazione del broker di servizi.

   ```sh
   git clone https://github.com/IBM-Cloud/cfee-service-broker-kubernetes.git
   ```
   {: codeblock}

   ```sh
   cd cfee-service-broker-kubernetes
   ```
   {: codeblock}

   ```sh
   git clone https://github.com/IBM/sample-resource-service-brokers.git
   ```
   {: codeblock}

2. Crea e archivia l'immagine Docker che contiene il broker di servizi su {{site.data.keyword.registryshort_notm}}. Utilizza il comando `ibmcloud cr info` per richiamare manualmente l'URL del registro o automaticamente con il comando `export REGISTRY` riportato di seguito. Il comando `cr namespace - add` creerà uno spazio dei nomi per archiviare l'immagine docker.

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   Crea uno spazio dei nomi denominato `cfee-tutorial`.

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   Modifica il file `./cfee-service-broker-kubernetes/deployment.yml`. Aggiorna l'attributo `image` per riflettere il tuo URL del registro del contenitore.
3. Distribuisci l'immagine contenitore al cluster Kubernetes di CFEE. Il cluster del tuo CFEE esiste nel gruppo di risorse `default`, che deve essere indicato come destinazione, se non è stato già fatto. Utilizzando il nome del tuo cluster, esporta la variabile KUBECONFIG utilizzando il comando `cluster-config`. Crea quindi la distribuzione.
   ```sh
   ibmcloud target -g default
   ```
   {: codeblock}

   ```sh
   ibmcloud ks clusters
   ```
   {: codeblock}

   ```sh
   $(ibmcloud ks cluster-config <your-cfee-cluster-name> --export)
   ```
   {: codeblock}

   ```sh
   kubectl apply -f deployment.yml
   ```
   {: codeblock}
4. Verifica che i pod abbiano `Running` come STATUS. Potrebbe volerci qualche istante perché Kubernetes esegua il pull dell'immagine e avvii i contenitori. Nota che hai due pod perché `deployment.yml` ha richiesto 2 repliche (`replicas`).
   ```sh
   kubectl get pods
   ```
   {: codeblock}

## Verifica che il broker di servizi sia distribuito

Ora che hai distribuito il broker di servizi, conferma che funzioni correttamente. Eseguirai tale operazione in diversi modi: innanzitutto utilizzando il dashboard Kubernetes, poi accedendo al broker da un'applicazione Cloud Foundry e infine eseguendo effettivamente il bind di un servizio dal broker.

### Visualizza i tuoi pod dal dashboard Kubernetes

Questa sezione confermerà che le risorse Kubernetes sono configurate utilizzando il dashboard {{site.data.keyword.containershort_notm}}.

1. Dalla pagina [Kubernetes Clusters](https://{DomainName}/containers-kubernetes/clusters), accedi al tuo cluster CFEE facendo clic sull'elemento della riga che inizia con il nome del tuo servizio CFEE e che finisce con **-cluster**.
2. Apri il **dashboard Kubernetes** facendo clic sul pulsante corrispondente.
3. Fai clic sul collegamento **Services** dal menu a sinistra e seleziona **tutorial-broker-service**. Questo servizio è stato distribuito quando hai eseguito `kubectl apply` nei passi precedenti.
4. Nel dashboard risultante, nota quanto segue:
   - Al servizio è stato fornito un indirizzo IP privato (172.x.x.x) che è risolvibile solo all'interno del cluster Kubernetes.
   - Il servizio ha due endpoint, che corrispondono ai due pod che hanno i contenitori di broker di servizi in esecuzione.

Dopo aver confermato che il servizio è disponibile e sta fungendo da proxy ai pod di broker di servizi, puoi verificare che il broker risponda con le informazioni relative ai servizi disponibili.

Puoi visualizzare le risorse correlate a Cloud Foundry dal dashboard Kubernetes. Per vederle, fai clic su **Namespaces** per visualizzare tutti gli spazi dei nomi con le risorse, compreso lo spazio dei nomi di Cloud Foundry `cf`.
{: tip}

### Accedi al broker da un contenitore Cloud Foundry

Per provare le comunicazioni tra Cloud Foundry e Kubernetes, connetterai il broker di servizi direttamente da un'applicazione Cloud Foundry.

1. Tornando al tuo terminale, conferma che sei ancora connesso all'organizzazione `tutorial` del tuo CFEE e allo spazio `dev` utilizzando `ibmcloud target`. Se necessario, indica nuovamente come destinazione CFEE.
   ```sh
   ibmcloud target --cf
   ```
   {: codeblock}
2. Per impostazione predefinita, SSH è disabilitato negli spazi. Questo è diverso dal cloud pubblico, quindi abilita SSH nel tuo spazio `CFEE `dev`.
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. Utilizza il comando `kubectl` per visualizzare lo stesso ClusterIP che avevi visualizzato nel dashboard Kubernetes. Esegui quindi l'SSH nell'applicazione `GetStartedNode` e richiama i dati dal broker di servizi utilizzando l'indirizzo IP. Tieni presente che l'ultimo comando genererà un errore che verrà risolto dal prossimo passo.
   ```sh
   kubectl get service tutorial-broker-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf ssh GetStartedNode
   ```
   {: codeblock}

   ```sh
   export CLUSTER_IP=<CLUSTER-IP address>
   ```
   {: codeblock}

   ```sh
   wget --user TestServiceBrokerUser --password TestServiceBrokerPassword -O- http://$CLUSTER_IP/v2/catalog
   ```
   {: codeblock}
4. È probabile che tu abbia ricevuto un errore **Connection refused** . Ciò è dovuto agli [ASG (application security group)](https://docs.cloudfoundry.org/concepts/asg.html) predefiniti di CFEE. Un ASG (application security group) definisce l'intervallo di IP consentito per il traffico in uscita da un contenitore Cloud Foundry. Poiché `GetStartedNode` esiste fuori dall'intervallo predefinito, si verifica l'errore. Esci dalla sessione SSH e scarica l'ASG `public_networks`.
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. Modifica il file `public_networks.json` e verifica che l'indirizzo ClusterIP utilizzato non rientri nelle regole esistenti. Ad esempio, l'intervallo `172.32.0.0-192.167.255.255` probabilmente non include il ClusterIP e deve essere aggiornato. Se, ad esempio, il tuo indirizzo ClusterIP è qualcosa come `172.21.107.63` dovrai modificare il file per avere qualcosa tipo `172.0.0.0-255.255.255.255`.
6. Modifica la regola `destination` dell'ASG per includere l'indirizzo IP del servizio Kubernetes. Riduci il file per includere solo i dati JSON, che iniziano e finiscono con parentesi. Carica quindi il nuovo ASG.
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. Ripeti il passo 3, che dovrebbe ora riuscire con i dati del catalogo di simulazione. Termina chiudendo la sessione SSH.
   ```sh
   exit
   ```
   {: codeblock}

### Registra il broker di servizi presso CFEE

Per consentire agli sviluppatori di eseguire provisioning e bind dei servizi dal broker dei servizi, ne eseguirai la registrazione presso CFEE. In precedenza, hai lavorato con il broker utilizzando un indirizzo IP. Questo è però problematico. Se viene riavviato, il broker dei servizi riceve un nuovo indirizzo IP, il che richiede l'aggiornamento di CFEE. Per risolvere questo problema, utilizzerai un'altra funzione Kubernetes denominata KubeDNS che fornisce un nome dominio completo (FQDN Fully Qualified Domain Name) al broker dei servizi.

1. Registra il broker dei servizi presso CFEE utilizzando il nome dominio completo (FQDN) del servizio `tutorial-service-broker`. Ripetiamo, questo instradamento è interno al tuo cluster Kubernetes CFEE.
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. Aggiungi quindi i servizi offerti dal broker. Poiché il broker di esempio ha solo un servizio di simulazione, è necessario un singolo comando.
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. Nel tuo browser, accedi alla tua istanza CFEE dalla pagina [**Environments**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) e vai a `organizations -> spaces` e seleziona il tuo spazio `dev`.
4. Seleziona la scheda **Services** e il pulsante **Create Service**.
5. Nella casella di testo di ricerca, cerca **Test**. Verrà visualizzato il servizio di simulazione **Test Node Resource Service Broker Display Name** dal broker.
6. Seleziona il servizio, fai clic sul pulsante **Create** e fornisci il nome `welcome-service` per creare un'istanza del servizio. Diventerà chiaro nella prossima sezione perché viene denominato welcome-service. Esegui quindi il bind del servizio all'applicazione `GetStartedNode` utilizzando l'elemento **Bind to application** nel menu di overflow.
7. Per visualizzare il servizio collegato mediante bind, esegui il comando `cf env`. L'applicazione `GetStartedNode` può ora avvalersi dei dati nell'oggetto `credentials` in modo analogo a come utilizza attualmente i dati da `cloudantNoSQLDB`.
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## Aggiungi il servizio alla tua applicazione

Fino a questo punto, abbiamo distribuito un broker dei servizi ma non un servizio effettivo. Mentre il broker dei servizi fornisce i dati di bind a `GetStartedNode`, non è stata aggiunta alcuna funzionalità effettiva dal servizio stesso. Per correggere ciò, è stato creato un servizio di simulazione che fornisce un messaggio di benvenuto a `GetStartedNode` in diverse lingue. Distribuirai questo servizio a CFEE e aggiornerai il broker e l'applicazione per utilizzarlo.

1. Esegui il push dell'implementazione `welcome-service` a CFEE per consentire a ulteriori team di sviluppo di avvalersene come un'API.
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. Accedi al servizio utilizzando il tuo browser. L'instradamento (`route`) al servizio verrà visualizzato come parte dell'output del comando `push`.  L'URL risultante sarà simile a `https://welcome.<your-cfee-cluster-domain>`. Aggiorna la pagina per visualizzare le lingue alternative.
3. Ritorna alla cartella `sample-resource-service-brokers` e modifica l'implementazione di esempio Node.js `sample-resource-service-brokers/node-resource-service-broker/testresourceservicebroker.js`, vai alla riga 854 e aggiungi il
campo url all'oggetto credentials, sostituendo il segnaposto con il tuo nome dominio del cluster: 

   ```javascript
   // TODO - Do your actual work here

   var generatedUserid   = uuid();
   var generatedPassword = uuid();

   result = {
     credentials: {
       userid   : generatedUserid,
       password : generatedPassword,
       url: 'https://welcome.<your-cfee-cluster-domain>'
     }
   };
   ```
   {: codeblock}
4. Crea e distribuisci il broker dei servizi aggiornato. Ciò garantirà che la proprietà URL verrà fornita alle applicazioni che eseguono il bind del servizio.
   ```sh
   cd ..
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   ```sh
   kubectl patch deployment tutorial-servicebroker-deployment -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"version\":\"2\"}}}}}"
   ```
   {: codeblock}
5. Nel terminale, torna alla cartella dell'applicazione `get-started-node`. 
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. Modifica il file `get-started-node/server.js` in `get-stared-node` per includere il seguente middleware. Inseriscilo appena prima della chiamata `app.listen()`.
   ```javascript
   // Use the welcome service
   const request = require('request');
   const testService = appEnv.services['testnoderesourceservicebrokername'];

   if (testService) {
     const { credentials: { url} } = testService[0];
     app.get("/api/welcome", (req, res) => request(url, (e, r, b) => res.send(b)));
   } else {
     app.get("/api/welcome", (req, res) => res.send('Welcome'));
   }
   ```
   {: codeblock}
7. Effettua il refactoring della pagina `get-started-node/views/index.html`. Modifica `data-i18n="welcome"` in `id="welcome"` nella tag `h1`. Aggiungi quindi una chiamata al servizio alla fine della funzione `$(document).ready()`.
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. Aggiorna l'applicazione `GetStartedNode`. Includi la dipendenza del pacchetto `request` che era stata aggiunta a `server.js`, esegui nuovamente il bind del `welcome-service` per rilevare la nuova proprietà `url` e, per finire, esegui il push del nuovo codice dell'applicazione.
   ```sh
   npm i request -S
   ```
   {: codeblock}

   ```sh
   ibmcloud cf unbind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf bind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}

Hai finito. Visita ora l'applicazione e aggiorna la pagina diverse volte per visualizzare il messaggio di benvenuto in diverse lingue.

![Risposta del broker dei servizi](./images/solution45-CFEE-apps/service-broker.png)

Se continui a vedere solo **Welcome** e nessun'altra lingua, esegui il comando `ibmcloud cf env GetStartedNode` e conferma la presenza dell'`url` al servizio. In caso contrario, ripercorri i tuoi passi, aggiorna e ridistribuisci il broker. Se è presente il valore `url`, conferma che restituisce un messaggio nel tuo browser. Esegui quindi nuovamente i comandi `unbind-service` e `bind-service` seguiti da `ibmcloud cf restart GetStartedNode`.
{: tip}

Mentre il servizio di benvenuto utilizza Cloud Foundry come sua implementazione, potresti con la stessa facilità utilizzare Kubernetes. la differenza principale è che l'URL al servizio sarà simile a `welcome-service.default.svc.cluster.local`. Utilizzare l'URL KubeDNS URL offre il vantaggio aggiuntivo di tenere il traffico di rete ai servizi interno al cluster Kubernetes.

Con {{site.data.keyword.cfee_full_notm}} questi approcci alternativi sono ora possibili.

## Distribuisci questa esercitazione della soluzione utilizzando una toolchain  

Facoltativamente, puoi distribuire l'esercitazione della soluzione completa utilizzando una toolchain. Attieniti alle [istruzioni della toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes) per distribuire tutto quanto sopra indicato utilizzando una toolchain. 

Nota alcuni prerequisiti quando utilizzi una toolchain: devi aver creato un'istanza CFEE e un'organizzazione e uno spazio CFEE. Delle istruzioni dettagliate sono fornite nel readme delle [istruzioni della toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes). 

## Contenuto correlato
{:related}

* [Distribuzione di applicazioni nei cluster Kubernetes](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Componenti e architettura di Cloud Foundry Diego](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [Broker dei servizi CFEE su Kubernetes con una toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

