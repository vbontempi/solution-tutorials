---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-11"
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

# Strategie per le applicazioni resilienti
{: #strategies-for-resilient-applications}

Indipendentemente dall'opzione di calcolo, Kubernetes, Cloud Foundry, Cloud Functions o Server virtuali, le aziende cercano di ridurre il tempo di inattività e di creare architetture resilienti che raggiungono la massima disponibilità. Questa esercitazione evidenzia le capacità di IBM Cloud di creare soluzioni resilienti e, nel farlo, risponde alle seguenti domande. 

- Cosa devo tenere in considerazione quando preparo una soluzione che deve essere globalmente disponibile?
- In che modo le opzioni di calcolo disponibili ti aiutano a distribuire le applicazioni multi-regione?
- In che modo importo le risorse dell'applicazione o del servizio in ulteriori regioni? 
- In che modo i database possono essere replicati nelle ubicazioni?
- Quali servizi di supporto devono essere utilizzati: Block Storage, File Storage, Object Storage, Databases?
- Ci sono delle considerazioni specifiche del servizio?

## Obiettivi
{: #objectives}

* Apprendere i concetti di architettura coinvolti quando si creano applicazioni resilienti. 
* Comprendere in che modo tali concetti si associano alle offerte di calcolo e di servizi di IBM Cloud

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura e concetti

{: #architecture}

Per progettare un'architettura resiliente, devi considerare i singoli blocchi della tua soluzione e le loro specifiche capacità.  

Di seguito viene riportata un'architettura multi-regione che illustra i diversi componenti che possono esistere in una configurazione multi-regione. ![Architettura](images/solution39/Architecture.png)

Il diagramma dell'architettura sopra riportato può essere diverso a seconda dell'opzione di calcolo. Vedrai specifici diagrammi dell'architettura in ogni opzione di calcolo nelle sezioni successive.  

### Ripristino di emergenza con due regioni 

Per facilitare il ripristino di emergenza, vengono utilizzate due architetture ampiamente accettate: **attiva/attiva** e **attiva/passiva**. Ciascuna architettura presenta costi e benefici correlati al tempo e allo sforzo durante il ripristino. 

#### Configurazione attiva-attiva

In un'architettura attiva/attiva, entrambe le ubicazioni hanno istanze attive identiche con un programma di bilanciamento del carico che distribuisce il traffico tra di loro. Utilizzando questo approccio, è necessario che la replica dei dati sia attiva per sincronizzare i dati tra le due regioni in tempo reale. 

![Attiva/Attiva](images/solution39/Active-active.png)

Questa configurazione fornisce una disponibilità più elevata con una minore correzione manuale rispetto all'architettura attiva/passiva. Le richieste vengono servite da entrambi i data center. Devi configurare i servizi edge (programma di bilanciamento del carico) con il timeout appropriato e una logica di nuovi tentativi per instradare automaticamente la richiesta al secondo data center se si verifica un malfunzionamento nel primo ambiente data center.

Quando consideri un **obiettivo del punto di ripristino** (RPO) nello scenario attivo/attivo, la sincronizzazione dei dati tra i due data center attivi deve essere estremamente tempestiva per consentire un flusso della richiesta senza interruzioni.

#### Configurazione attiva-passiva

Un'architettura attiva/passiva si basa su una regione attiva e su una seconda regione (passiva) utilizzata come un backup. Nel caso di un'interruzione nella regione attiva, la regione passiva diventa attiva. Potrebbe essere necessario un intervento manuale per garantire che i database o l'archiviazione file siano in linea con le necessità dell'applicazione e dell'utente.  

![Attiva/Attiva](images/solution39/Active-passive.png)

Le richieste vengono servite dal sito attivo. Nel caso di un'interruzione o di un malfunzionamento dell'applicazione, viene eseguito un lavoro di pre-applicazione per rendere il data center in standby pronto per servire la richiesta. Il passaggio dal data center attivo a quello passivo è un'operazione che richiede molto tempo. Sia l'**obiettivo del tempo di ripristino** (RTO) che l'**obiettivo del punto di ripristino** (RPO) sono più elevati rispetto alla configurazione attiva/attiva. 

### Ripristino di emergenza con tre regioni

Nell'era attuale dei servizi "Always On" con tolleranza zero per i tempi di inattività, i clienti si aspettano che ogni servizio aziendale rimanga accessibile 24 ore su 24 in qualsiasi parte del mondo. Una strategia conveniente per le aziende implica la progettazione di un'architettura per la tua infrastruttura per la disponibilità continua piuttosto che la creazione di infrastrutture per il ripristino di emergenza. 

L'utilizzo di tre data center offre una maggiore resilienza e disponibilità rispetto a due. Può anche offrire prestazioni migliori distribuendo il carico in modo più uniforme tra i data center. Se l'azienda dispone solo di due data center, una variante a ciò consiste nel distribuire due applicazioni in un data center e la terza nel secondo data center. In alternativa, puoi distribuire i livelli di presentazione e di logica aziendale nella topologia 3 attivi e distribuire il livello di dati nella topologia 2 attivi.

#### Configurazione attiva-attiva-attiva (3 attivi)

![](images/solution39/Active-active-active.png)

Le richieste vengono servite dall'applicazione in esecuzione in uno qualsiasi dei tre data center attivi. Un caso di studio sul sito web IBM.com indica che la configurazione 3 attivi richiede solo il 50% della capacità di calcolo, di memoria e di rete per cluster, ma la configurazione 2 attivi richiede il 100% per cluster. Il livello di dati è il punto in cui risalta la differenza di costi. Per ulteriori dettagli, leggi [*Always On: Assess, Design, Implement, and Manage Continuous Availability*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf).

#### Configurazione attiva-attiva-passiva

![](images/solution39/Active-active-passive.png)

In questo scenario, quando una delle due applicazioni attive nei data center primario e secondario risente di un'interruzione, viene attivata l'applicazione in standby nel terzo data center. Per ripristinare la normalità per elaborare le richieste del cliente, viene seguita la procedura di ripristino di emergenza descritta nello scenario di due data center. L'applicazione in standby nel terzo data center può essere impostata in una configurazione hot standby o cold standby. 

Fai riferimento a [questa guida](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/) per ulteriori informazioni sul ripristino di emergenza. 

### Architetture multi-regioni

In un'architettura multi-regione, un'applicazione viene distribuita in diverse ubicazioni in cui ogni regione esegue una copia identica dell'applicazione.  

Una regione è una specifica ubicazione geografica in cui puoi distribuire applicazioni, servizi e altre risorse {{site.data.keyword.cloud_notm}}. Le [regioni {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/containers?topic=containers-regions-and-zones) sono composte da una o più zone, che sono i data center fisici che ospitano le risorse di calcolo, di rete e di archiviazione e il raffreddamento e la potenza correlati che ospitano i servizi e le applicazioni. Le zone sono isolate l'una dall'altra, il che garantisce che non ci sia alcun singolo punto di malfunzionamento condiviso. 

Inoltre, in un'architettura multi-regione, un programma di bilanciamento del carico globale come [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services) è obbligatorio per poter distribuire il traffico tra le regioni. 

La distribuzione di una soluzione tra più regioni presenta i seguenti vantaggi:
- Migliora la latenza per gli utenti finali - la velocità è la chiave, più la tua origine di backend è vicina agli utenti finali, migliore e più veloce sarà l'esperienza per gli utenti. 
- Ripristino di emergenza - quando nella regione attiva si verifica un malfunzionamento, hai una regione di backup per un rapido ripristino. 
- Requisiti aziendali - in alcuni casi, hai bisogno di archiviare i dati in regioni distinte, separati da diverse centinaia di chilometri. Pertanto, in tal caso, devi archiviare i dati in più regioni.  

### Architetture multizona all'interno delle regioni

La creazione di applicazioni per regioni multizona vuol dire avere la tua applicazione distribuita nelle zone di una regione e quindi potresti avere anche due o tre regioni.  

Con l'architettura della regione multizona, devi avere un programma di bilanciamento del carico locale per distribuire il traffico localmente tra le zone in una regione e quindi, se viene configurata una seconda regione, un programma di bilanciamento del carico globale distribuisce il traffico tra le regioni.  

Puoi acquisire ulteriori informazioni sulle regioni e sulle zone [qui](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones).

## Opzioni di calcolo 

Questa sezione esamina le opzioni di calcolo disponibili in {{site.data.keyword.cloud_notm}}. Per ogni opzione di calcolo, viene fornito un diagramma dell'architettura insieme a un'esercitazione che illustra come distribuire tale architettura. 

Nota: tutte le architetture delle opzioni di calcolo non hanno database o altri servizi inclusi, si concentrano solo sulla distribuzione di un'applicazione in due regioni per l'opzione di calcolo selezionata. Una volta distribuito uno qualsiasi degli esempi delle opzioni di calcolo multi-regione, il passo logico successivo sarebbe aggiungere i database e altri servizi. Le sezioni successive di questa esercitazione della soluzione riguarderanno i [database](#databaseservices) e i [servizi non database](#nondatabaseservices).

### Cloud Foundry 

Cloud Foundry offre la possibilità di realizzare la distribuzione di un'architettura multi-regione, utilizzando anche i servizi di pipeline di [fornitura continua](https://{DomainName}/catalog/services/continuous-delivery) che ti consentono di distribuire la tua applicazione tra più regioni. L'architettura per Cloud Foundry multi-regione è simile alla seguente: 

![Architettura-CF](images/solution39/CF2-Architecture.png)

La stessa applicazione viene distribuita in più regioni e un programma di bilanciamento del carico globale instrada il traffico alla regione più vicina e integra. L'esercitazione [**Applicazione web protetta in più regioni**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp) ti guida nella distribuzione di un'architettura simile. 

### {{site.data.keyword.cfee_full_notm}}

**{{site.data.keyword.cfee_full_notm}} (CFEE)** offre tutte le stesse funzionalità del Cloud Foundry pubblico insieme ad altre funzioni.

**{{site.data.keyword.cfee_full_notm}}** ti consente di istanziare, su richiesta, più piattaforme Cloud Foundry di livello enterprise e isolate. Le istanze di CFEE vengono eseguite all'interno del tuo account in [{{site.data.keyword.cloud_notm}}
](http://ibm.com/cloud). L'ambiente viene distribuito in un hardware isolato sul [ {{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service). Hai il controllo completo sull'ambiente, incluso il controllo dell'accesso, la gestione delle capacità, la gestione delle modifiche, il monitoraggio e i servizi.

Di seguito viene riportata un'architettura multi-regione che utilizza {{site.data.keyword.cfee_full_notm}}.

![Architettura](images/solution39/CFEE-Architecture.png)

La distribuzione di questa architettura richiede quanto segue: 

- Configura due istanze CFEE - una in ogni regione.
- Crea ed esegui il bind dei servizi all'account CFEE. 
- Esegui il push delle applicazioni specifiche per l'endpoint API CFEE.  
- Configura la replica del database, proprio come faresti su un Cloud Foundry pubblico. 

Inoltre, consulta la guida dettagliata [Deploy Logistics Wizard to Cloud Foundry Enterprise Environment (CFEE)](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md). Ti guiderà nella distribuzione di un'applicazione basata sul microservizio in CFEE. Una volta distribuita in un'istanza CFEE, puoi replicare la procedura in una seconda regione e collegare [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) davanti alle due istanze CFEE per bilanciare il carico del traffico. 

Per ulteriori dettagli, fai riferimento alla [documentazione {{site.data.keyword.cfee_full_notm}}](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about). 

### Kubernetes

Con Kubernetes, puoi realizzare un'architettura multizona all'interno delle regioni, questo può essere un caso di utilizzo attivo/attivo. Quando implementi una soluzione con {{site.data.keyword.containershort_notm}}, usufruisci delle capacità integrate come il bilanciamento del carico e l'isolamento, la resilienza aumentata contro potenziali malfunzionamenti con host, reti o applicazioni. Creando più cluster e se si verifica un'interruzione con un cluster, gli utenti possono ancora accedere a un'applicazione che viene distribuita anche in un altro cluster. Con più cluster in regioni diverse, gli utenti possono accedere anche al cluster più vicino con una latenza di rete ridotta. Per una resilienza aggiuntiva, hai anche la possibilità di selezionare i cluster multizona, ciò significa che i tuoi nodi vengono distribuiti tra più zone all'interno di una regione.  

L'architettura multi-regione Kubernetes è simile a quella riportata di seguito.

![Kubernetes](images/solution39/Kub-Architecture.png)

1. Lo sviluppatore crea le immagini Docker per l'applicazione. 
2. Viene eseguito il push delle immagini in {{site.data.keyword.registryshort_notm}} in due ubicazioni diverse.
3. L'applicazione viene distribuita ai cluster Kubernetes in entrambe le ubicazioni. 
4. Gli utenti finali accedono all'applicazione. 
5. Cloud Internet Services è configurato per intercettare le richieste all'applicazione e per distribuire il carico tra i cluster. Sono inoltre abilitati la Protezione DDoS e WAF (Web Application Firewall) per proteggere l'applicazione dalle comuni minacce. Facoltativamente, le risorse come immagini e file CSS vengono memorizzate nella cache. 

L'esercitazione [**Cluster Kubernetes multi-regione resilienti e protetti con Cloud Internet Services**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis) ti guida nella distribuzione di tale architettura.

### {{site.data.keyword.openwhisk_short}}

{{site.data.keyword.openwhisk_short}} è disponibile in più ubicazioni {{site.data.keyword.cloud_notm}}. Per aumentare la resilienza e ridurre la latenza di rete, le applicazioni possono distribuire il loro backend in più ubicazioni. Quindi, con IBM CIS (Cloud Internet Services), gli sviluppatori possono esporre un singolo punto di ingresso incaricato di distribuire il traffico al back-end integro più vicino. L'architettura per la multi-regione {{site.data.keyword.openwhisk_short}} è simile alla seguente. 

 ![Architettura-Functions](images/solution39/Functions-Architecture.png)

1. Gli utenti accedono all'applicazione. La richiesta passa attraverso Internet Services.
2. Internet Services reindirizza gli utenti al backend API integro più vicino. 
3. Certificate Manager fornisce l'API con il suo certificato SSL. Il traffico viene crittografato end-to-end.
4. L'API viene implementata con Cloud Functions.

Scopri come distribuire questa architettura seguendo l'esercitazione [**Distribuzione di applicazioni senza server in più regioni**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless).

### {{site.data.keyword.baremetal_short}} e {{site.data.keyword.virtualmachinesshort}}

{{site.data.keyword.virtualmachinesshort}} e {{site.data.keyword.baremetal_short}} offrono la possibilità di realizzare un'architettura multi-regione. Puoi eseguire il provisioning dei server in molte ubicazioni disponibili su {{site.data.keyword.cloud_notm}}.

![ubicazioni server](images/solution39/ServersLocation.png)

Quando prepari tale architettura utilizzando {{site.data.keyword.virtualmachinesshort}} e {{site.data.keyword.baremetal_short}}, considera quanto segue: archiviazione file, backup, ripristino e database, scelta tra un database come servizio o installazione di un database su un server virtuale. 

L'architettura riportata di seguito illustra la distribuzione di un'architettura multi-regione utilizzando {{site.data.keyword.virtualmachinesshort}} in un'architettura attiva/passiva in cui una regione è attiva e la seconda regione è passiva. 

![Architettura-VM](images/solution39/vm-Architecture2.png)

I componenti richiesti per tale architettura: 

1. Gli utenti accedono all'applicazione tramite IBM CIS (Cloud Internet Services).
2. CIS instrada il traffico all'ubicazione attiva.
3. All'interno di un'ubicazione, un programma di bilanciamento del carico reindirizza il traffico a un server.
4. I database vengono distribuiti in un server virtuale, ciò significa che configurerai il database e le repliche e i backup tra le regioni. L'alternativa consisterebbe nell'utilizzare un DBaaS (database-as-service), un argomento trattato più avanti nell'esercitazione. 
5. L'archiviazione file per archiviare le immagini e i file dell'applicazione, File Storage offre la possibilità di eseguire un'istantanea in un determinato giorno e a una determinata ora, questa istantanea può essere poi riutilizzata all'interno di un'altra regione, operazione che eseguiresti manualmente.  

L'esercitazione [**Utilizza i server virtuali per creare un'applicazione web altamente disponibile e scalabile**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application) implementa questa architettura.

## Database e file dell'applicazione
{: #databaseservices}

{{site.data.keyword.cloud_notm}} offre una selezione di [DBaaS (database as a service)](https://{DomainName}/catalog/?category=databases) con database relazionali e non relazionali a seconda delle tue esigenze aziendali. [DBaaS (Database-as-a-service)](https://www.ibm.com/cloud/learn/what-is-cloud-database) offre molti vantaggi. Utilizzando un DBaaS come {{site.data.keyword.cloudant}}, puoi usufruire del supporto multi-regione che ti consente di eseguire la replica dal vivo tra due servizi database che si trovano in regioni diverse, di eseguire i backup e di avere un ridimensionamento e una massima operatività. 

**Funzioni chiave:** 

- Un servizio database integrato e accessibile tramite una piattaforma cloud
- Consente agli utenti aziendali di ospitare i database senza dover acquistare hardware dedicato
- Può essere gestito dall'utente oppure offerto come un servizio e gestito da un provider
- Può supportare i database SQL o NoSQL
- È accessibile tramite un'interfaccia web o un'API fornita dal fornitore

**Durante la preparazione per l'architettura multi-regione:**

- Quali sono le opzioni di resilienza del servizio database?
- Come viene gestita la replica tra più servizi database tra le regioni?
- Come viene eseguito il backup dei dati?
- Quali sono gli approcci di ripristino di emergenza per ciascun caso?

### {{site.data.keyword.cloudant}}

{{site.data.keyword.cloudant}} è un database distribuito ottimizzato per gestire carichi di lavoro pesanti tipici di grandi applicazioni web e mobili in rapida crescita. Disponibile come un servizio {{site.data.keyword.Bluemix_notm}} completamente gestito e supportato da SLA, {{site.data.keyword.cloudant}} ridimensiona elasticamente in modo indipendente la velocità effettiva e l'archiviazione. {{site.data.keyword.cloudant}} è disponibile anche come un'installazione in loco scaricabile e la sua API e il suo potente protocollo di replica sono compatibili con un ecosistema open source che include CouchDB, PouchDB e le librerie per gli stack di sviluppo mobile e web più popolari. 

{{site.data.keyword.cloudant}} supporta la [replica](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation) tra più istanze tra le ubicazioni. Qualsiasi modifica eseguita nel database di origine viene riprodotta nel database di destinazione. Puoi creare repliche tra un qualsiasi numero di database, in modo continuo o come un'attività una tantum. Il seguente diagramma mostra una configurazione tipica che utilizza due istanze {{site.data.keyword.cloudant}}, una in ogni regione: 

![attiva-attiva](images/solution39/Active-active.png)

Fai riferimento a [queste istruzioni](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery) per configurare la replica tra le istanze {{site.data.keyword.cloudant}}. Il servizio fornisce anche le istruzioni e gli strumenti per [eseguire il backup e il ripristino dei dati](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery).

### {{site.data.keyword.Db2_on_Cloud_short}}, {{site.data.keyword.dashdbshort_notm}} e {{site.data.keyword.Db2Hosted_notm}}

{{site.data.keyword.cloud_notm}} offre diversi [servizi database Db2](https://{DomainName}/catalog/?search=db2h). Sono:

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2): un database SQL cloud completamente gestito per i tipici carichi di lavoro operativi come OLTP.
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse): un servizio di data warehouse cloud completamente gestito per carichi lavoro analitici a livello di petabyte ad alte prestazioni. Offre i piani di servizio SMP e MPP e utilizza un archivio dati a colonne ottimizzato e l'elaborazione nella memoria.
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted): un database ospitato da IBM e gestito dal sistema di database dell'utente. Fornisce a Db2 l'accesso amministrativo completo all'infrastruttura cloud, eliminando in tal modo il costo, la complessità e il rischio di gestione della tua infrastruttura. 

Nella seguente sezione, ci concentreremo su {{site.data.keyword.Db2_on_Cloud_short}} come DBaaS per i carichi di lavoro operativi. Questi carichi di lavoro sono tipici delle applicazioni trattate in questa esercitazione. 

#### Supporto multi-regione per {{site.data.keyword.Db2_on_Cloud_short}}

{{site.data.keyword.Db2_on_Cloud_short}} offre diverse [opzioni per realizzare HADR (High Availability and Disaster Recovery)](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview). Puoi scegliere l'opzione High Availability (HA - Alta disponibilità) quando crei un nuovo servizio. In seguito, puoi [aggiungere un nodo di ripristino di emergenza (DR) con replica geografica](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha) tramite il dashboard dell'istanza. L'opzione del nodo DR offsite ti consente di sincronizzare i tuoi dati in tempo reale in un nodo database in un data center {{site.data.keyword.cloud_notm}} offsite di tua scelta. 

Ulteriori informazioni sono disponibili nella [documentazione di Elevata disponibilità (HA)](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha).

#### Backup e ripristino

{{site.data.keyword.Db2_on_Cloud_short}} include backup giornalieri per piani a pagamento. Di norma, i backup vengono archiviati utilizzando {{site.data.keyword.cos_short}} e utilizzando così tre data center per una maggiore disponibilità dei dati conservati. I backup vengono conservati per 14 giorni. Puoi utilizzarli per eseguire un ripristino a punto temporale. La [documentazione su backup e ripristino](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br) fornisce i dettagli su come puoi ripristinare i dati alla data e all'ora che desideri. 

### {{site.data.keyword.databases-for}}

{{site.data.keyword.databases-for}} offre diversi sistemi di database open source come servizi completamente gestiti. Sono:  
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

Tutti questi servizi condividono le stesse caratteristiche:    
* Per l'alta disponibilità, vengono distribuiti in cluster. Puoi trovare i dettagli nella documentazione di ciascun servizio: 
  - [PostgreSQL](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* Ciascun cluster si estende su più zone. 
* I dati vengono replicati tra le zone. 
* Gli utenti possono ampliare le risorse di archiviazione e di memoria per un'istanza. Per dettagli, vedi la [documentazione sul ridimensionamento di, ad esempio, {{site.data.keyword.databases-for-redis}}](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings).
* I backup vengono eseguiti giornalmente o su richiesta. I dettagli sono documentati per ciascun servizio. Questa è [la documentazione sul backup di, ad esempio, {{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups).
* I dati inattivi, i backup e il traffico di rete sono crittografati. 
* Ciascun [servizio può essere gestito utilizzando il plugin CLI {{site.data.keyword.databases-for}}](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in)


### {{site.data.keyword.cos_full_notm}}

{{site.data.keyword.cos_full_notm}} (COS) offre un'archiviazione durevole, protetta e conveniente. Le informazioni archiviate con {{site.data.keyword.cos_full_notm}} vengono crittografate e diffuse tra più ubicazioni geografiche. Quando crei i bucket di archiviazione all'interno di un'istanza COS, decidi in quale ubicazione deve essere creato il bucket e quale opzione di resilienza utilizzare. 

Ci sono tre tipi di resilienza del bucket:
   - La resilienza interregionale (**Cross Region**) distribuisce i tuoi dati in diverse aree metropolitane. Può essere vista come un'opzione multi-regione. Quando accedi al contenuto memorizzato in un bucket interregionale (Cross Region), COS offre un endpoint speciale in grado di richiamare il contenuto da una regione integra. 
   - La resilienza regionale (**Regional**) distribuisce i dati in una singola area metropolitana. Può essere vista come una configurazione multizona all'interno di una regione. 
   - La resilienza a singolo data center (**Single Data Center**) distribuisce i dati in più dispositivi all'interno di un singolo data center. 

Fai riferimento a [questa documentazione](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints) per una spiegazione dettagliata delle opzioni di resilienza di {{site.data.keyword.cos_full_notm}}.

### {{site.data.keyword.filestorage_full_notm}}

{{site.data.keyword.filestorage_full_notm}} è un'archiviazione file basata di NFS, collegata alla rete flessibile, veloce e persistente. In questo ambiente NAS (Network Attached Storage), hai il controllo totale sulle prestazioni e la funzione di condivisioni dei tuoi file. Le condivisioni di {{site.data.keyword.filestorage_short}} possono essere connesse a un massimo di 64 dispositivi autorizzati su connessioni TCP/IP instradate per la resilienza.

Alcune delle funzioni dell'archiviazione file sono _Istantanee_, _Replica_, _Accesso simultaneo_. Fai riferimento alla [documentazione](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage) per un elenco completo delle funzioni. 

Una volta collegato ai tuoi server, un servizio {{site.data.keyword.filestorage_short}} può essere utilizzato facilmente per archiviare i backup dei dati, i file delle applicazioni come immagini e video, queste immagini e questi file possono essere poi utilizzati all'interno di server diversi nella stessa regione. 

Quando aggiungi una seconda regione, puoi utilizzare la funzione delle istantanee di {{site.data.keyword.filestorage_short}} per eseguire automaticamente o manualmente un'istantanea e poi riutilizzarla all'interno della seconda regione passiva. 

La replica può essere pianificata per copiare automaticamente le istantanee in un volume di destinazione in un data center remoto. Le copie possono essere ripristinate nel sito remoto nel caso si verifichi un evento catastrofico o un danneggiamento dei dati. Puoi trovare ulteriori informazioni sulle istantanee [qui](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots).

## Servizi non database
{: #nondatabaseservices}

{{site.data.keyword.cloud_notm}} offre una selezione di [servizi](https://{DomainName}/catalog) non database, questi sono sia servizi IBM che servizi di terze parti. Quando pianifichi l'architettura multi-regione, devi comprendere in che modo i servizi come i servizi Watson possono funzionare in una configurazione multi-regione.

### {{site.data.keyword.conversationfull}}

[{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about) è una piattaforma che consente agli sviluppatori e agli utenti non tecnici di collaborare alla creazione di assistenti conversazionali con tecnologia di Intelligenza Artificiale. 

Un assistente è un bot cognitivo che puoi personalizzare per le tue esigenze aziendali e distribuire in più canali per supportare i tuoi clienti dove e quando ne hanno bisogno. L'assistente include una o più capacità. Una capacità di dialogo contiene i dati di addestramento e la logica che consente a un assistente di aiutare i tuoi clienti. 

È importante tenere presente che {{site.data.keyword.conversationshort}} V1 è senza stato. {{site.data.keyword.conversationshort}} offre un tempo di attività del 99.5% ma, per le applicazioni altamente disponibili tra più regioni, potresti anche volere più istanze di questo servizio tra le regioni.  

Una volta che hai creato le istanze in più ubicazioni, utilizza lo strumento {{site.data.keyword.conversationshort}} per esportare, da un'istanza, uno spazio di lavoro esistente, inclusi gli intenti, le entità e il dialogo. Quindi importa questo spazio di lavoro in altre ubicazioni. 

## Riepilogo

| Offerta  | Opzioni di resilienza |
| -------- | ------------------ |
| Cloud Foundry | <ul><li>Distribuisci le applicazioni in più ubicazioni</li><li>Servi le richieste da più ubicazioni con Cloud Internet Services</li><li>Utilizza le API Cloud Foundry per configurare le organizzazioni, gli spazi ed eseguire il push delle applicazioni in più ubicazioni</li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>Distribuisci le applicazioni in più ubicazioni</li><li>Servi le richieste da più ubicazioni con Cloud Internet Services</li><li>Utilizza le API Cloud Foundry per configurare le organizzazioni, gli spazi ed eseguire il push delle applicazioni in più ubicazioni</li><li>Basato sul servizio Kubernetes</li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>Resilienza tramite la progettazione con il supporto per i cluster multizona</li><li>Servi le richieste dai cluster estesi in più ubicazioni con Cloud Internet Services</li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>Disponibile in più ubicazioni</li><li>Servi le richieste da più ubicazioni con Cloud Internet Services</li><li>Utilizza l'API Cloud Functions per distribuire le azioni in più ubicazioni</li></ul> |
| {{site.data.keyword.baremetal_short}} e {{site.data.keyword.virtualmachinesshort}} | <ul><li>Esegui il provisioning dei server in più ubicazioni</li><li>Collega i server nella stessa ubicazione a un programma di bilanciamento del carico locale</li><li>Servi le richieste da più ubicazioni con Cloud Internet Services</li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>Replica singola e continua tra i database</li><li>Ridondanza dei dati automatica all'interno di una regione</li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>Esegui il provisioning di un nodo di ripristino di emergenza con replica geografica per la sincronizzazione dei dati in tempo reale</li><li>Backup giornaliero con i piani a pagamento</li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>Basato sui cluster Kubernetes multizona</li><li>Repliche di lettura in più regioni</li><li>Backup giornalieri e su richiesta</li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>Resilienza a singolo data center, regionale e interregionale</li><li>Utilizza l'API per sincronizzare il contenuto tra i bucket di archiviazione</li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>Utilizza le istantanee per acquisire automaticamente il contenuto in una destinazione in un data center remoto</li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>Utilizza l'API Watson per esportare e importare la specifica dello spazio di lavoro tra più istanze nelle ubicazioni</li></ul> |

## Contenuto correlato

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry, Applicazione web protetta in più regioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [Cloud Functions, Distribuzione di applicazioni senza server in più regioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes, Cluster Kubernetes multi-regione resilienti e protetti con Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [Server virtuali, Crea un'applicazione web altamente disponibile e scalabile](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
