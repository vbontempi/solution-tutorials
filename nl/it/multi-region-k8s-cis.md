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

# Cluster Kubernetes multi-regione resilienti e protetti con Cloud Internet Services
{: #multi-region-k8s-cis}

Quando un'applicazione è progettata pensando alla resilienza, è meno probabile che gli utenti riscontrino dei tempi di inattività. Quando implementi una soluzione con {{site.data.keyword.containershort_notm}}, usufruisci delle capacità integrate come il bilanciamento del carico e l'isolamento e la resilienza aumentata contro potenziali malfunzionamenti con host, reti o applicazioni. Creando più cluster e se si verifica un'interruzione con un cluster, gli utenti possono ancora accedere a un'applicazione che viene distribuita anche in un altro cluster. Con più cluster in ubicazioni diverse, gli utenti possono accedere anche al cluster più vicino e ridurre la latenza di rete. Per una resilienza aggiuntiva, hai anche la possibilità di selezionare i cluster multizona, il che significa che i tuoi nodi vengono distribuiti tra più zone all'interno di un'ubicazione.

Questa esercitazione evidenzia in che modo CIS (Cloud Internet Services), una piattaforma uniforme per configurare e gestire DNS (Domain Name System), GLB (Global Load Balancing), WAF (Web Application Firewall) e la protezione da attacchi di tipo DDoS (Distributed Denial of Service) per le applicazioni internet, possa essere integrato con i cluster Kubernetes per supportare questo scenario e fornire una soluzione protetta e resiliente in più ubicazioni.

## Obiettivi
{: #objectives}

* Distribuire un'applicazione su più cluster Kubernetes in ubicazioni differenti.
* Distribuire il traffico su più cluster con un Global Load Balancer.
* Instradare gli utenti al cluster più vicino.
* Proteggere la tua applicazione dalle minacce alla sicurezza.
* Aumentare le prestazioni dell'applicazione con la memorizzazione nella cache.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* IBM Cloud [Internet services](https://{DomainName}/catalog/services/internet-services)
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

<p style="text-align: center;">
  ![Architettura](images/solution32-multi-region-k8s-cis/Architecture.png)
</p>

1. Lo sviluppatore crea le immagini Docker per l'applicazione.
2. Viene eseguito il push delle immagini a {{site.data.keyword.registryshort_notm}} a Dallas e Londra.
3. L'applicazione viene distribuita ai cluster Kubernetes in entrambe le ubicazioni.
4. Gli utenti finali accedono all'applicazione.
5. Cloud Internet Services è configurato per intercettare le richieste all'applicazione e per distribuire il carico tra i cluster. Sono inoltre abilitati la Protezione DDoS e WAF (Web Application Firewall) per proteggere l'applicazione dalle comuni minacce. Facoltativamente, le risorse come immagini e file CSS vengono memorizzate nella cache.

## Prima di iniziare
{: #prereqs}

* CIS (Cloud Internet Services) richiede che tu sia il proprietario di un dominio personalizzato in modo che tu possa configurare il DNS affinché questo dominio punti al server dei nomi CIS (Cloud Internet Services).
* [Installa Git](https://git-scm.com/).
* [Installa la CLI {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli).
* [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - script per installare docker, kubectl, helm, la cli ibmcloud e i plugin richiesti.
* [Configura la CLI {{site.data.keyword.registrylong_notm}} e il tuo spazio dei nomi del registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Comprendi i principi di base di Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Distribuisci un'applicazione in una singola ubicazione

Questa esercitazione distribuisce un'applicazione Kubernetes ai cluster in più ubicazioni. Inizierai con un'ubicazione, Dallas, e ripeterai quindi questa procedura per Londra.

### Crea un cluster Kubernetes
{: #create_cluster}

Per creare un cluster:
1. Seleziona **{{site.data.keyword.containershort_notm}}** dal [catalogo {{site.data.keyword.cloud_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster/create).
1. Imposta **Location** su **Dallas**.
1. Seleziona il cluster **Standard**.
1. Seleziona una o più zone come ubicazione (**Ubicazione**). La creazione di un cluster multizona aumenta la resilienza dell'applicazione. È molto meno probabile che gli utenti riscontrino dei tempi di inattività quando l'applicazione è distribuita in più zone. Ulteriori informazioni sui cluster multizona sono disponibili [qui](https://{DomainName}/docs/containers?topic=containers-plan_clusters#ha_clusters).
1. Imposta **Machine type** sul valore più piccolo disponibile; per questa esercitazione, sono sufficienti **2 CPUs** e **4GB RAM**.
1. Utilizza **2** nodi di lavoro.
1. Imposta **Cluster name** su **my-us-cluster**. Utilizza il pattern di denominazione *`my-<location>-cluster`* per essere congruente con questa esercitazione.

Mentre il cluster si sta preparando, tu preparerai l'applicazione.

### Crea uno spazio dei nomi in {{site.data.keyword.registryshort_notm}}

1. Indica come destinazione la CLI {{site.data.keyword.Bluemix_notm}} a Dallas.
   ```bash
   ibmcloud target -r us-south
   ```
   {: pre}
2. Crea uno spazio dei nomi per l'applicazione.
   ```bash
   ibmcloud cr namespace-add <your_namespace>
   ```
   {: pre}

Puoi anche riutilizzare uno spazio dei nomi esistente se ne hai uno nell'ubicazione. Puoi elencare gli spazi dei nomi esistenti con `ibmcloud cr namespaces`.
{: tip}

### Crea l'applicazione

Questo passo crea l'applicazione in un'immagine Docker. Puoi tralasciare questo passo se stai configurando il secondo cluster. Si tratta di una semplice applicazione Hello World.

1. Clona il codice sorgente per l'[applicazione Hello World](https://github.com/IBM/container-service-getting-started-wt){:new_windows} nella tua home directory. Il repository contiene diverse versioni di un'applicazione simile nelle cartelle, ognuna delle quali inizia con Lab.
   ```bash
   git clone https://github.com/IBM/container-service-getting-started-wt.git
   ```
   {: pre}
1. Passa alla directory `Lab 1`.
   ```bash
   cd 'container-service-getting-started-wt/Lab 1'
   ```
   {: pre}
1. Crea un'immagine Docker che include i file dell'applicazione della directory `Lab 1`. 
   ```bash
   docker build --tag multi-region-hello-world:1 .
   ```
   {: pre}

### Prepara l'immagine per l'esecuzione del push nel registro specifico per l'ubicazione

Contrassegna l'immagine con il registro di destinazione:

   ```bash
   docker tag multi-region-hello-world:1 us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### Esegui il push dell'immagine nel registro specifico per l'ubicazione

1. Assicurati che il tuo motore Docker locale possa eseguire il push al registro a Dallas.
   ```bash
   ibmcloud cr login
   ```
   {: pre}
2. Esegui il push dell'immagine.
   ```bash
   docker push us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### Distribuisci l'applicazione al cluster Kubernetes

A questo punto, il cluster dovrebbe essere pronto. Puoi controllarne lo stato nella [console {{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/clusters).

1. Richiama la configurazione del cluster:
   ```bash
   ibmcloud cs cluster-config <us-cluster-name>
   ```
   {: pre}
1. Copia e incolla l'output per impostare la variabile di ambiente KUBECONFIG. La variabile viene utilizzata da `kubectl`.
1. Esegui l'applicazione nel cluster con due repliche:
   ```bash
   kubectl run hello-world-deployment --image=us.icr.io/<your_US-South_namespace>/hello-world:1 --replicas=2
   ```
   {: pre}
   Output di esempio: `deployment "hello-world-deployment" created`.
1. Rendi l'applicazione accessibile all'interno del cluster
   ```bash
   kubectl expose deployment/hello-world-deployment --type=ClusterIP --port=80 --name=hello-world-service --target-port=8080
   ```
   {: pre}
   Restituisce un messaggio come `service "hello-world-service" exposed`.

### Ottieni il nome dominio e l'indirizzo IP assegnati al cluster
{: #CSALB_IP_subdomain}

Quando viene creato un cluster Kubernetes, gli viene assegnato un sottodominio Ingress (ad es. *my-us-cluster.us-south.containers.appdomain.cloud*) e un indirizzo IP di Application Load Balancer pubblico.

1. Richiama il sottodominio Ingress del cluster:
   ```bash
   ibmcloud cs cluster-get <us-cluster-name>
   ```
   {: pre}
   Cerca il valore `Ingress Subdomain`.
1. Prendi nota di queste informazioni per un passo successivo.

Questa esercitazione utilizza il sottodominio Ingress per configurare il Global Load Balancer. Puoi anche sostituire il sottodominio per l'indirizzo IP dell'Application Load Balancer (`ibmcloud cs albs -cluster <us-cluster-name>`). Sono supportate entrambe le opzioni.
{: tip}

## E quindi, in un'altra ubicazione

Ripeti i passi precedenti a Londra sostituendo:
* il nome ubicazione **Dallas** con **London**;
* l'alias ubicazione **us-south** con **eu-gb**;
* il registro *us.icr.io* con **uk.icr.io**;
* e il nome cluster *my-us-cluster* con **my-uk-cluster**.

## Configura il bilanciamento del carico multi-ubicazione

La tua applicazione è ora in esecuzione in due cluster ma manca un componente per consentire agli utenti di accedere ai cluster in modo trasparente da un singolo punto di accesso.

In questa sezione, configurerai CIS (Cloud Internet Services) per distribuire il carico tra i due cluster. CIS è un servizio a referente unico che fornisce la regola di pagina, GLB (Global Load Balancer), memorizzazione in cache e WAF (Web Application Firewall) per proteggere le tue applicazioni garantendo al tempo stesso l'affidabilità e le prestazioni per le tue applicazioni cloud.

Per configurare un Global Load Balancer, dovrai:
* puntare un dominio personalizzato ai server dei nomi CIS,
* richiamare gli indirizzi IP o i nomi di sottodominio dei cluster Kubernetes,
* configurare i controlli dell'integrità per convalidare la disponibilità della tua applicazione
* e definire i pool di origine che puntano ai cluster.

### Registra un dominio personalizzato presso Cloud Internet Services
{: #create_cis_instance}

Il primo passo consiste nel creare un'istanza di CIS e di puntare il tuo dominio personalizzato ai server dei nomi CIS.

1. Se non sei il proprietario di un dominio, puoi acquistarne uno da un registrar come ad esempio [godaddy.com](http://godaddy.com).
2. Passa a [Internet Services](https://{DomainName}/catalog/services/internet-services) nel catalogo {{site.data.keyword.Bluemix_notm}}.
3. Imposta il nome servizio e fai clic su **Create** per creare un'istanza del servizio.
4. Quando viene eseguito il provisioning dell'istanza del servizio, imposta il tuo nome dominio e fai clic su **Add domain**.
5. Quando vengono assegnati i server dei nomi, configura il tuo registrar o il provider del nome del dominio in modo che utilizzi i server dei nomi elencati.
6. Una volta configurato il tuo registrar o il provider DNS, potresti dover attendere fino a 24 ore affinché le modifiche diventino effettive.

   Quando lo stato del dominio nella pagina Overview cambia da *Pending* a *Active*, puoi utilizzare il comando `dig <your_domain_name> ns` per verificare che i nuovi server dei nomi siano stati applicati.
   {:tip}

### Configura il controllo dell'integrità per il Global Load Balancer

Un controllo dell'integrità consente di acquisire informazioni dettagliate sulla disponibilità dei pool in modo che il traffico possa essere instradato solo ai pool integri. Questi controlli inviano periodicamente le richieste HTTP/HTTPS e monitorano le risposte.

1. Nel dashboard Cloud Internet Services, vai a **Reliability** > **Global Load Balancer**e nella parte inferiore della pagina, fai clic su **Create health check**.
1. Imposta **Path** su **/**
1. Imposta **Monitor Type** su **HTTP**.
1. Fai clic su **Provision 1 Instance**.

   Quando crei le tue applicazioni, puoi definire un endpoint di controllo dell'integrità dedicato, come ad esempio */heathz*, dove notificherai lo stato dell'applicazione.
   {:tip}

### Definisci i pool di origine

Un pool è un gruppo di server di origine a cui viene instradato il traffico in modo intelligente quando collegato a un GLB. Con i cluster nel Regno Unito e negli Stati Uniti, puoi definire dei pool basati sull'ubicazione e configurare CIS per reindirizzare gli utenti ai cluster più vicini sulla base dell'ubicazione geografica delle richieste utente.

#### Un pool per il cluster a Londra
1. Fai clic su **Create Pool**.
2. Imposta **Name** su **UK**
3. Imposta **Health check** su quello creato nella sezione precedente
4. Imposta **Health Check Region** su **Western Europe**
5. Imposta **Origin Name** su **uk-cluster**
6. Imposta **Origin Address** sul sottodominio Ingress del cluster a Londra, ad es. *my_uk_cluster.eu-gb.containers.appdomain.cloud*
7. Fai clic su **Provision 1 Instance**.

#### Un pool per il cluster a Dallas
1. Fai clic su **Create Pool**.
2. Imposta **Name** su **US**
3. Imposta **Health check** su quello creato nella sezione precedente
4. Imposta **Health Check Region** su **Western North America**
5. Imposta **Origin Name** su **us-cluster**
6. Imposta **Origin Address** sul sottodominio Ingress del cluster a Dallas, ad es. *my_us_cluster.us-south.containers.appdomain.cloud*
7. Fai clic su **Provision 1 Instance**.

#### E un pool con entrambi i cluster
1. Fai clic su **Create Pool**.
1. Imposta **Name** su **All**
1. Imposta **Health check** su quello creato nella sezione precedente
1. Imposta **Health Check Region** su **Eastern North America**
1. Aggiungi due origini:
   1. una con il nome origine (**Origin Name**) impostato su **us-cluster** e l'indirizzo di origine (**Origin Address**) impostato sul sottodominio Ingress del cluster a Dallas
   2. una con il nome origine (**Origin Name**) impostato su **uk-cluster** e l'indirizzo di origine (**Origin Address**) impostato sul sottodominio Ingress del cluster a Londra
2. Fai clic su **Provision 1 Instance**.

### Crea il Global Load Balancer

Con i pool di origine definiti, puoi completare la configurazione del programma di bilanciamento del carico.

1. Fai clic su **Create Load Balancer**.
1. Immetti un nome in **Balancer hostname** per il Global Load Balancer. Questo nome farà anche parte dell'URL applicazione universale (`http://<glb_name>.<your_domain_name>`), indipendentemente dall'ubicazione.
1. In **Default origin pools**, fai clic su **Add pool** e aggiungi il pool denominato **All**.
1. Espandi la sezione di **Configure geo routes(optional)**:
   1. Fai clic su **Add route**, seleziona **Western Europe** e fai clic su **Add**.
   1. Fai clic su **Add pool** per selezionare il pool **UK**.
   1. Configura ulteriori instradamenti come mostrato nella seguente tabella.
   1. Fai clic su **Provision 1 Instance**.

| Regione              |Pool di origine |
| :---------------:    | :---------: |
|Western Europe        |     UK      |
|Eastern Europe        |     UK      |
|Northeast Asia        |     UK      |
|Southeast Asia        |     UK      |
|Western North America |     US      |
|Eastern North America |     US      |

Con questa configurazione, gli utenti in Europa e in Asia saranno reindirizzati al cluster a Londra, gli utenti negli Stati Uniti al cluster a Dallas. Quando una richiesta non corrisponde a nessuno degli instradamenti definiti, verrà reindirizzata ai pool di origine predefiniti (**Default origin pools**).

### Crea la risorsa Ingress per i cluster Kubernetes per ogni ubicazione

Il Global Load Balancer è ora pronto a soddisfare le richieste. Tutti i controlli dell'integrità dovrebbero dare esito positivo. C'è però almeno un altro passo di configurazione necessario sui cluster Kubernetes per rispondere correttamente alle richieste provenienti dal Global Load Balancer: devi definire una risorsa Ingress per gestire le richieste dal dominio GLB.

1. Crea un file di risorse Ingress denominato **glb-ingress.yaml**
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: glb-ingress
   spec:
    rules:
      - host: <glb_name>.<your_domain_name>
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-world-service
              servicePort: 80
   ```
    {: pre}
    Sostituisci `<glb_name>.<your_domain_name>` con l'URL che hai definito nella sezione precedente.
1. Distribuisci questa risorsa a entrambi i cluster di Londra e Dallas dopo aver configurato la variabile RECONFIG per i rispettivi cluster di ubicazione.
   ```bash
   kubectl create -f glb-ingress.yaml
   ```
   {: pre}
   Genera in output il messaggio `ingress.extension "glb-ingress" created`.

A questo punto, hai configurato correttamente un Global Load Balancer con i cluster Kubernetes in più ubicazioni. Puoi accedere all'URL del GLB `http://<glb_name>.<your_domain_name>` per visualizzare la tua applicazione. In base alla tua ubicazione, vieni reindirizzato al cluster più vicino o a un cluster dal pool predefinito se CIS non è stato in grado di associare il tuo indirizzo IP a una specifica ubicazione.

## Proteggi l'applicazione
{: #secure_via_CIS}

### Attiva WAF (Web Application Firewall)

Il WAF (Web Application Firewall) protegge la tua applicazione web da attacchi ISO livello 7. Di norma, viene combinato con set di regole raggruppati, che hanno come obiettivo il proteggere da vulnerabilità nell'applicazione escludendo mediante filtro il traffico malintenzionato.

1. Nel dashboard Cloud Internet Services, vai a **Security** e quindi nella scheda **Manage**.
1. Nella sezione **Web Application Firewall**, assicurati che WAF sia abilitato.
1. Fai clic su **View OWASP Rule Set**. Da questa pagina, puoi esaminare il set di regole di base OWASP (**OWASP Core Rule Set**) e abilitare o disabilitare singolarmente le regole. Quando una regola è abilitata, se una richiesta in entrata la attiva, il punteggio di minaccia globale verrà aumentato. L'impostazione **Sensitivity** deciderà se viene attivata un'azione (**Action**) per la richiesta.
   1. Non apportare alcuna modifica ai set di regole OWASP predefiniti.
   1. Imposta **Sensitivity** su `Low`.
   1. Imposta **Action** su `Simulate` per registrare tutti gli eventi.
1. Fai clic su **Back to Security**.
1. Fai clic su **View CIS Rule Set**. Questa pagina mostra regole aggiuntive basate sugli stack di tecnologia comuni per l'hosting di siti web.

### Aumenta le prestazioni e proteggi da attacchi DoS (Denial of Service) 
{: #proxy_setting}

Un attacco distributed denial of service ([DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)) è un tentativo doloso di interrompere il traffico normale di un server, un servizio o una rete sovraccaricando la destinazione o la relativa infrastruttura circostante con una grande quantità di traffico internet. CIS è equipaggiato per proteggere il tuo dominio da DDoS.

1. Nel dashboard CIS, seleziona **Reliability** > **Global Load Balancer**.
1. Individua il GLB che hai creato nella tabella **Load Balancers**.
1. Abilita le funzioni di sicurezza (Security) e prestazioni (Performance) nella colonna **Proxy**.

   ![Attivazione (impostazione su ON) del proxy CIS](images/solution32-multi-region-k8s-cis/cis-proxy.png)

**Il tuo GLB è ora protetto**. Un vantaggio immediato è che l'indirizzo IP di origine dei tuoi cluster sarà nascosto ai client. Se CIS rileva una minaccia per una richiesta in arrivo, potresti vedere una schermata come questa prima di essere reindirizzato alla tua applicazione:

   ![Verifica - protezione da DDoS](images/solution32-multi-region-k8s-cis/cis-DDoS.png)

Inoltre, puoi ora controllare quale contenuto viene memorizzato nella cache da CIS e per quanto tempo rimane memorizzato nella cache. Vai a **Performance** > **Caching** per definire il livello di memorizzazione nella cache globale e la scadenza del browser. Puoi personalizzare le regole di sicurezza globale e di memorizzazione nella cache con le regole della pagina (**Page Rules**). Le regole della pagina (Page Rules) abilitano una configurazione dettagliata utilizzando percorsi di dominio specifici. Per fare un esempio con le regole della pagina (Page Rules), puoi decidere di memorizzare nella cache tutto il contenuto in **/assets** per 3 giorni (**3 days**):

   ![Regole della pagina](images/solution32-multi-region-k8s-cis/cis-pagerules.png)

## Rimuovi le risorse
{:removeresources}

### Rimuovi le risorse del cluster Kubernetes
1. Rimuovi l'Ingress.
1. Rimuovi il servizio.
1. Rimuovi la distribuzione.
1. Elimina i cluster se li hai creati specificamente per questa esercitazione.

### Rimuovi le risorse CIS
1. Rimuovi il GLB.
1. Rimuovi i pool di origine.
1. Rimuovi i controlli dell'integrità.

## Contenuto correlato
{:related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Gestisci il tuo IBM CIS per la sicurezza ottimale](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#best-practice-2-configure-your-security-level-selectively)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
* [{{site.data.keyword.registrylong_notm}} di base](https://{DomainName}/docs/services/Registry?topic=registry-registry_overview#registry_planning)
* [Distribuzione di singole applicazioni dell'istanza ai cluster Kubernetes](https://{DomainName}/docs/containers?topic=containers-cs_apps_tutorial#cs_apps_tutorial_lesson1)
* [Prassi ottimale per proteggere il traffico e l'applicazione internet tramite CIS](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#manage-your-ibm-cis-for-optimal-security)
* [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
