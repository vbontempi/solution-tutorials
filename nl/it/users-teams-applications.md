---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# Procedure ottimali per organizzare gli utenti, i team e le applicazioni
{: #users-teams-applications}

Questa esercitazione fornisce una panoramica dei concetti disponibili {{site.data.keyword.cloud_notm}} per gestire IAM (Identity and Access Management) e il modo in cui possono essere implementati per supportare le diverse fasi di sviluppo di un'applicazione.
{:shortdesc}

Quando crei un'applicazione, è molto comune definire più ambienti che riflettono il ciclo di vita di sviluppo di un progetto, da uno sviluppatore che esegue il commit del codice al codice dell'applicazione reso disponibile agli utenti finali. *Sandbox*, *test*, *preparazione*, *UAT* (user acceptance testing), *preproduzione*, *produzione* sono nomi tipici per questi ambienti. 

L'isolamento delle risorse sottostanti, l'implementazione delle politiche di governance e di accesso, la protezione di un carico di lavoro di produzione, la convalida delle modifiche prima di eseguirne il push per la produzione, sono alcuni dei motivi per cui vorresti creare questi ambienti separati. 

## Obiettivi
{: #objectives}

* Acquisire informazioni sui modelli di accesso {{site.data.keyword.iamlong}} e Cloud Foundry
* Configurare un progetto con una separazione tra ruoli e ambienti
* Configurare l'integrazione continua

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Definisci un progetto

Consideriamo un progetto di esempio con i seguenti componenti: 
* diversi microservizi distribuiti nel {{site.data.keyword.containershort_notm}},
* database,
* bucket di archiviazione file.

In questo progetto, definiamo tre ambienti:
* *Development* (Sviluppo) - questo ambiente viene aggiornato continuamente con ogni commit, test di unità e smoke test eseguito. Fornisce l'accesso alla distribuzione migliore e più recente del progetto. 
* *Testing* (Test) - questo ambiente viene creato dopo un ramo o una tag del codice stabile. È dove viene eseguito il test di accettazione dell'utente. La sua configurazione è simile all'ambiente di produzione. Viene caricato con dati realistici (ad esempio, dati di produzione anonimi).
* *Production* (Produzione) - questo ambiente viene aggiornato con la versione convalidata nell'ambiente precedente. 

**Una pipeline di fornitura gestisce l'avanzamento di una build nell'ambiente.** Può essere completamente automatizzata oppure includere dei gate di convalida manuale per promuovere le build approvate tra gli ambienti - è molto aperta e deve essere configurata in modo che corrisponda alle procedure ottimali e ai flussi di lavoro aziendali. 

Per supportare l'esecuzione della pipeline di creazione, introduciamo **un utente funzionale** - un utente {{site.data.keyword.cloud_notm}} regolare, ma un membro del team senza una reale identità nel mondo fisico. Questo utente funzionale sarà proprietario delle pipeline di fornitura e di qualsiasi altra risorsa cloud che richiede una proprietà potente. Questo approccio ti aiuta nel caso in cui un membro del team lasci l'azienda oppure venga spostato su un altro progetto. L'utente funzionale sarà dedicato al tuo progetto e non cambierà nel corso del ciclo di vita del progetto. La prossima cosa che vorrai creare è [una chiave API](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey) per questo utente funzionale. Selezionerai questa chiave API quando configurerai le pipeline DevOps oppure quando vorrai eseguire gli script di automazione, per impersonare l'utente funzionale. 

Quando si tratta di assegnare le responsabilità ai membri del team del progetto, definiamo i seguenti ruoli e le autorizzazioni correlate: 

|           | Sviluppo    | Test    | Produzione |
| --------- | ----------- | ------- | ---------- |
| Developer (Sviluppatore) | <ul><li>contribuisce al codice</li><li>può accedere ai file di log</li><li>può esaminare la configurazione dell'applicazione e del servizio</li><li>utilizza le applicazioni distribuite</li></ul> | <ul><li>può accedere ai file di log</li><li>può esaminare la configurazione dell'applicazione e del servizio</li><li>utilizza le applicazioni distribuite</li></ul> | <ul><li>nessun accesso</li></ul> |
| Tester    | <ul><li>utilizza le applicazioni distribuite</li></ul> | <ul><li>utilizza le applicazioni distribuite</li></ul> | <ul><li>nessun accesso</li></ul> |
| Operator (Operatore) | <ul><li>può accedere ai file di log</li><li>può visualizzare/impostare la configurazione dell'applicazione e del servizio</li></ul> | <ul><li>può accedere ai file di log</li><li>può visualizzare/impostare la configurazione dell'applicazione e del servizio</li></ul> | <ul><li>può accedere ai file di log</li><li>può visualizzare/impostare la configurazione dell'applicazione e del servizio</li></ul> |
| Pipeline Functional User (Utente funzionale della pipeline)  | <ul><li>può distribuire/annullare la distribuzione delle applicazioni</li><li>può visualizzare/impostare la configurazione dell'applicazione e del servizio</li></ul> | <ul><li>può distribuire/annullare la distribuzione delle applicazioni</li><li>può visualizzare/impostare la configurazione dell'applicazione e del servizio</li></ul> | <ul><li>può distribuire/annullare la distribuzione delle applicazioni</li><li>può visualizzare/impostare la configurazione dell'applicazione e del servizio</li></ul> |

## IAM (Identity and Access Management)
{: #first_objective}

{{site.data.keyword.iamshort}} (IAM) ti consente di autenticare in modo sicuro gli utenti per i servizi di piattaforma e di infrastruttura e di controllare l'accesso alle **risorse** in modo coerente attraverso la piattaforma {{site.data.keyword.cloud_notm}}. Viene abilitata una serie di servizi {{site.data.keyword.cloud_notm}} per utilizzare Cloud IAM per il controllo dell'accesso e questi servizi sono organizzati in **gruppi di risorse** all'interno del tuo **account** per consentire agli **utenti** un accesso rapido e semplice a più di una risorsa alla volta. Le **politiche** di accesso Cloud IAM vengono utilizzate per assegnare agli utenti e agli ID servizio l'accesso alle risorse all'interno del tuo account. 

Una **politica** assegna a un utente o a un ID servizio uno o più **ruoli** con una combinazione di attributi che definiscono l'ambito di accesso. La politica può fornire l'accesso a un singolo servizio fino al livello di istanza o può essere applicata a una serie di risorse organizzate insieme in un gruppo di risorse. A seconda dei ruoli utente che assegni, all'ID utente o servizio vengono consentiti diversi livelli di accesso per completare attività di gestione della piattaforma o per accedere a un servizio utilizzando l'interfaccia utente o per eseguire specifici tipi di chiamate API. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="Diagramma del modello IAM" />
</p>

In questo momento, non tutti i servizi nel catalogo {{site.data.keyword.cloud_notm}} possono essere gestiti utilizzando IAM. Per questi servizi, puoi continuare a utilizzare Cloud Foundry fornendo agli utenti l'accesso all'organizzazione e allo spazio a cui appartiene l'istanza con un ruolo Cloud Foundry assegnato per definire il livello di accesso consentito. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Diagramma del modello Cloud Foundry" />
</p>

## Crea le risorse per un ambiente

Sebbene i tre ambienti necessari per questo progetto di esempio richiedano diritti di accesso diversi e possono aver bisogno di assegnazioni di capacità diverse, condividono un pattern di architettura comune. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="Diagramma architettura che mostra un ambiente" />
</p>

Iniziamo creando l'ambiente Development (Sviluppo). 

1. [Seleziona un'ubicazione {{site.data.keyword.cloud_notm}}](https://{DomainName}) in cui distribuire l'ambiente. 
1. Per le applicazioni e i servizi Cloud Foundry: 
   1. [Crea un'organizzazione per il progetto](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg).
   1. [Crea uno spazio Cloud Foundry per l'ambiente](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo).
   1. Crea i servizi Cloud Foundry utilizzati dal progetto in questo spazio
1. [Crea un gruppo di risorse per l'ambiente](https://{DomainName}/account/resource-groups).
1. Crea i servizi compatibili con un gruppo di risorse come {{site.data.keyword.cos_full_notm}}, {{site.data.keyword.la_full_notm}}, {{site.data.keyword.mon_full_notm}} e {{site.data.keyword.cloudant_short_notm}} in questo gruppo.
1. [Crea un nuovo cluster Kubernetes](https://{DomainName}/containers-kubernetes/catalog/cluster) in {{site.data.keyword.containershort_notm}}, assicurati di selezionare il gruppo di risorse creato prima. 
1. Configura {{site.data.keyword.la_full_notm}} e {{site.data.keyword.mon_full_notm}} per inviare i log e per monitorare il cluster.

Il seguente diagramma mostra dove vengono create le risorse del progetto nell'account:

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="Diagramma che mostra le risorse del progetto" />
</p>

## Assegna i ruoli all'interno dell'ambiente

1. Invita gli utenti al tuo account
1. Assegna le politiche agli utenti per controllare chi può accedere al gruppo di risorse, ai servizi all'interno del gruppo e all'istanza {{site.data.keyword.containershort_notm}} e le relative autorizzazioni. Fai riferimento alla [definizione di politica di accesso](https://{DomainName}/docs/containers?topic=containers-users#access_policies) per selezionare le politiche corrette per un utente nell'ambiente. Gli utenti con la stessa serie di politiche possono essere collocati nello [stesso gruppo di accesso](https://{DomainName}/docs/iam?topic=iam-groups#groups). Semplifica la gestione degli utenti in quanto le politiche verranno assegnate al gruppo di accesso ed ereditate da tutti gli utenti nel gruppo. 
1. Configura i loro ruoli organizzazione e spazio Cloud Foundry in base alle loro esigenze all'interno dell'ambiente. Fai riferimento alla [definizione di ruolo](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess) per assegnare i ruoli corretti in base all'ambiente. 

Fai riferimento alla documentazione dei servizi per comprendere in che modo un servizio sta associando i ruoli IAM e Cloud Foundry a specifiche azioni. Vedi, ad esempio, [in che modo il servizio {{site.data.keyword.mon_full_notm}} associa i ruoli IAM alle azioni](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam).

L'assegnazione dei ruoli corretti agli utenti richiederà diverse interazioni e perfezionamenti. Determinate autorizzazioni possono essere controllate a livello di gruppo di risorse, per tutte le risorse in un gruppo oppure possono essere dettagliate fino a una specifica istanza di un servizio, scoprirai nel tempo quali sono le politiche di accesso ideali per il tuo progetto. 

Ti raccomandiamo di iniziare con una serie minima di autorizzazioni e poi espanderla in base alle esigenze. Per Kubernetes, vorrai esaminare il suo [Role-Based Access Control (RBAC)](https://kubernetes.io/docs/admin/authorization/rbac/) per configurare le autorizzazioni in cluster.

Per l'ambiente Development (Sviluppo), le responsabilità utente definite in precedenza potrebbero essere tradotte in quanto segue: 

|           | Politiche di accesso IAM | Cloud Foundry |
| --------- | ----------- | ------- |
| Developer (Sviluppatore) | <ul><li>Gruppo di risorse: *Viewer* (Visualizzatore)</li><li>Ruoli di accesso alla piattaforma nel gruppo di risorse: *Viewer* (Visualizzatore)</li><li>Ruolo del servizio Logging & Monitoring (Registrazione e monitoraggio): *Writer* (Scrittore)</li></ul> | <ul><li>Ruolo organizzazione: *Auditor* (Revisore)</li><li>Ruolo spazio: *Auditor* (Revisore)</li></ul> |
| Tester    | <ul><li>Non è necessaria alcuna configurazione. Il tester accede all'applicazione distribuita, non agli ambienti di sviluppo</li></ul> | <ul><li>Non è necessaria alcuna configurazione</li></ul> |
| Operator (Operatore) | <ul><li>Gruppo di risorse: *Viewer* (Visualizzatore)</li><li>Ruoli di accesso alla piattaforma nel gruppo di risorse: *Operator* (Operatore), *Viewer* (Visualizzatore)</li><li>Ruolo del servizio Logging & Monitoring (Registrazione e monitoraggio): *Writer* (Scrittore)</li></ul> | <ul><li>Ruolo organizzazione: *Auditor* (Revisore)</li><li>Spazio ruolo: *Developer* (Sviluppatore)</li></ul> |
| Pipeline Functional User (Utente funzionale della pipeline)  | <ul><li>Gruppo di risorse: *Viewer* (Visualizzatore)</li><li>Ruoli di accesso alla piattaforma nel gruppo di risorse: *Editor*, *Viewer* (Visualizzatore)</li></ul> | <ul><li>Ruolo organizzazione: *Auditor* (Revisore)</li><li>Spazio ruolo: *Developer* (Sviluppatore)</li></ul> |

Le politiche di accesso IAM e i ruoli Cloud Foundry sono definiti nell'[interfaccia utente IAM (Identify and Access Management)](https://{DomainName}/iam/#/users):

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="Configurazione delle autorizzazioni per il ruolo sviluppatore" />
</p>

## Replica per più ambienti

Da qui, puoi replicare passi simili per creare gli altri ambienti. 

1. Crea un gruppo di risorse per ambiente. 
1. Crea un cluster e le istanze del servizio richieste per ambiente. 
1. Crea uno spazio Cloud Foundry per ambiente. 
1. Crea le istanze del servizio richieste in ogni spazio. 

<p style="text-align: center;">
  <img title="Utilizzo di cluster separati per isolare gli ambienti" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="Diagramma che mostra cluster separati per isolare gli ambienti" />
</p>

Utilizzando una combinazione di strumenti come la [CLI {{site.data.keyword.cloud_notm}} `ibmcloud`](https://github.com/IBM-Cloud/ibm-cloud-developer-tools), [`terraform` di HashiCorp](https://www.terraform.io/), [{{site.data.keyword.cloud_notm}} Provider for Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm), la CLI Kubernetes `kubectl`, puoi creare script e automatizzare la creazione di questi ambienti. 

Separa i cluster Kubernetes per gli ambienti con buone proprietà: 
* indipendentemente dall'ambiente, i cluster tendono a sembrare tutti uguali; 
* è più semplice controllare chi ha accesso a uno specifico cluster;
* consente la flessibilità nei cicli di aggiornamento per le distribuzioni e le risorse sottostanti; quando è disponibile una nuova versione Kubernetes, ti dà la possibilità di aggiornare innanzitutto il cluster Development (Sviluppo), di convalidare la tua applicazione e poi di aggiornare l'altro ambiente;
* evita di combinare carichi di lavoro che potrebbero influenzarsi a vicenda, ad esempio isolando la distribuzione della produzione dagli altri. 

Un altro approccio consiste nell'utilizzare gli [spazi dei nomi Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) insieme alle [quote di risorse Kubernetes](https://kubernetes.io/docs/concepts/policy/resource-quotas/) per isolare gli ambienti e controllare l'utilizzo delle risorse. 

<p style="text-align: center;">
  <img title="Utilizzo di spazi dei nomi separati per isolare gli ambienti" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="Diagramma che mostra spazi dei nomi separati per isolare gli ambienti" />
</p>

Nella casella di input `Search` dell'interfaccia utente LogDNA, utilizza il campo `namespace: ` per filtrare i log in base allo spazio dei nomi.
{: tip}

## Configura la pipeline di fornitura

Quando si tratta di eseguire la distribuzione in diversi ambienti, la tua pipeline di integrazione continua/fornitura continua può essere configurata per gestire l'interno processo:
* aggiorna in modo continuo l'ambiente `Development` (Sviluppo) con il codice migliore e più recente proveniente dal ramo `development`, eseguendo test di unità e test di integrazione sul cluster dedicato;
* promuove le build di sviluppo nell'ambiente `Testing` (Test), automaticamente se tutti i test delle fasi precedenti hanno avuto esito positivo oppure tramite il processo di promozione manuale. Alcuni team utilizzeranno anche diversi rami qui, integrando lo stato di sviluppo del lavoro con un ramo `stable`, ad esempio;
* Ripete un processo simile per passare all'ambiente `Production` (Produzione).

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="Una pipeline CI/CD dalla build da distribuire" />
</p>

Quando configuri la pipeline DevOps, assicurati di utilizzare la chiave API di un utente funzionale. Solo l'utente funzionale deve disporre dei diritti necessari per distribuire le applicazioni nei tuoi cluster. 

Durante la fase di creazione, è importante controllare la versione corretta delle immagini Docker. Puoi utilizzare la revisione di commit Git come parte della tag dell'immagine oppure un identificativo univoco fornito dalla tua toolchain DevOps; qualsiasi identificativo che ti semplificherà l'associazione dell'immagine alla tua build effettiva e al codice sorgente contenuto nell'immagine. 

Man mano che acquisisci familiarità con Kubernetes, [Helm](https://helm.sh/), il gestore pacchetti per Kubernetes, diventerà uno strumento pratico per controllare la versione, assemblare e distribuire la tua applicazione. [Questa toolchain DevOps di esempio](https://github.com/open-toolchain/simple-helm-toolchain) è un buon punto di partenza ed è preconfigurata per la fornitura continua a un cluster Kubernetes. Man mano che il tuo progetto cresce in più microservizi, il [diagramma a ombrello Helm](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies) fornirà una buona soluzione per comporre la tua applicazione. 

## Espandi l'esercitazione

Congratulazioni, la tua applicazione può essere ora distribuita in modo sicuro dallo sviluppo alla produzione. Di seguito troverai altri suggerimenti per migliorare la distribuzione dell'applicazione. 

* Aggiungi [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) alla tua pipeline per eseguire il controllo di qualità durante le distribuzioni.
* Esamina i contributi di codifica dei membri del team e le interazioni tra gli sviluppatori con [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).
* Segui l'esercitazione [Pianifica, crea e aggiorna ambienti di distribuzione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments) per automatizzare la distribuzione dei tuoi ambienti. 

## Informazioni correlate

* [Introduzione a {{site.data.keyword.iamshort}}](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [Procedure ottimali per organizzare le risorse in un gruppo di risorse](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [Analizza i log e monitora l'integrità dell'applicazione con LogDNA e Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [Distribuzione continua su Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Hello Helm toolchain](https://github.com/open-toolchain/simple-helm-toolchain)
* [Develop a microservices application with Kubernetes and Helm](https://github.com/open-toolchain/microservices-helm-toolchain)
* [Concessione delle autorizzazioni a un utente per visualizzare i log in LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [Concessione delle autorizzazioni a un utente per visualizzare le metriche in Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
