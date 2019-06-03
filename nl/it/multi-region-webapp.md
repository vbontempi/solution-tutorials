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

# Applicazione web protetta in più regioni
{: #multi-region-webapp}

Questa esercitazione ti guida nella creazione, protezione, distribuzione e bilanciamento del carico di un'applicazione Cloud Foundry in più regioni utilizzando una [pipeline {{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery).

Si verificheranno delle interruzioni delle applicazioni o di loro parti, e questo è un dato di fatto. Può trattarsi di un problema nel tuo codice, una manutenzione pianificata che si ripercuote sulle risorse utilizzate dalla tua applicazione, un malfunzionamento hardware che arresta una zona, un'ubicazione o un data center dove è ospitata la tua applicazione. Sono eventi che si verificheranno e devi essere preparato. Con {{site.data.keyword.Bluemix_notm}}, puoi distribuire la tua applicazione a [più ubicazioni](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg) per aumentare la resilienza della tua applicazione. Con la tua applicazione ora in esecuzione in più ubicazioni, inoltre, puoi anche reindirizzare il traffico utente all'ubicazione più vicina per ridurre la latenza.

## Obiettivi

* Distribuire un'applicazione Cloud Foundry a più ubicazioni con {{site.data.keyword.contdelivery_short}}.
* Associare un dominio personalizzato all'applicazione.
* Configurare il GLB (global load balancing) alla tua applicazione multi-ubicazione.
* Eseguire il bind di un certificato SSL alla tua applicazione.
* Monitorare le prestazioni dell'applicazione.

## Servizi utilizzati

Questa esercitazione utilizza i seguenti runtime e servizi:
* [Applicazione {{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry
* [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) per DevOps
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura

Questa esercitazione implica uno scenario attivo/attivo in cui due copie dell'applicazione vengono distribuite in due ubicazioni differenti e le due copie si occupano delle richieste utente in modalità round-robin. Se si verifica il malfunzionamento di una copia, la configurazione DNS punta automaticamente all'ubicazione integra.

<p style="text-align: center;">

   ![Architettura](./images/solution1/Architecture.png)
</p>

## Crea un'applicazione Node.js
{: #create}

Inizia creando un'applicazione starter Node.js che viene eseguita in un ambiente Cloud Foundry.

1. Fai clic su **[Catalog](https://{DomainName}/catalog/)** nella console {{site.data.keyword.Bluemix_notm}}.
2. Fai clic su **Cloud Foundry Apps** nella categoria **Platform** e seleziona **[{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)** .
     ![](images/solution1/SDKforNodejs.png)
3. Immetti un **nome univoco** per la tua applicazione, che sarà anche il tuo nome host, ad esempio myusername-nodeapp. Fai quindi clic su **Create**.
4.  Dopo l'avvio dell'applicazione, fai clic sul link **Visit URL** nella pagina **Overview** per visualizzare la tua applicazione LIVE su una nuova scheda.

![HelloWorld](images/solution1/HelloWorld.png)

Ottimo inizio. Hai una tua applicazione starter Node.js in esecuzione in {{site.data.keyword.Bluemix_notm}}.

Eseguiamo ora il push del codice sorgente della tua applicazione a un repository e distribuiamo le tue modifiche automaticamente.

## Configura il controllo origine e {{site.data.keyword.contdelivery_short}}
{: #devops}

In questo passo, configuri un repository di controllo origine git per archiviare il tuo codice e crei quindi una pipeline, che distribuisce le eventuali modifiche del codice automaticamente.

1. Nel riquadro di sinistra della tua applicazione che hai appena creato, seleziona **Overview** e scorri per trovare **{{site.data.keyword.contdelivery_short}}**. Fai clic su **Enable**.

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. Mantieni le opzioni predefinite e fai clic su **Create**. Dovresti ora disporre di una **toolchain** predefinita che è stata creata.

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. Seleziona il tile **Git** in **Code**. Vieni quindi reindirizzato alla tua pagina di repository git.
4. Se non hai ancora configurato le chiavi SSH, dovresti vedere una barra di notifica in alto con le istruzioni. Segui la procedura aprendo il link **Add an SSH key** in una nuova scheda o, se vuoi utilizzare HTTPS anziché SSH, segui la procedura facendo clic su **Create a personal access token**. Ricordati di salvare la chiave o il token per riferimenti futuri.
5. Seleziona SSH o HTTPS e copia l'URL git. Clona l'origine alla tua macchina locale.
   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   **Nota:** se ti viene richiesto un nome utente, fornisci il tuo nome utente git. Per la password, utilizza una **chiave SSH** o un **token di accesso personale** esistente o quello che hai creato nel passo precedente.
6. Apri il repository clonato in un IDE di tua scelta e passa a `public/index.html`. Aggiorniamo ora il codice. Prova a modificare "Hello World" in qualcos'altro.
7. Esegui l'applicazione in locale eseguendo i comandi uno dopo l'altro
  `npm install`, `npm build`,  `npm start ` e visita `localhost:<port_number>` nel tuo browser.
  **<port_number>** come visualizzato nella console.
8. Esegui il push della modifica al tuo repository con tre semplici passi: aggiunta, commit e push.
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
9. Vai alla toolchain che hai creato in precedenza e fai clic sul tile **Delivery Pipeline**.
10. Conferma che vedi le fasi **BUILD** e **DEPLOY**.
  ![HelloWorld](images/solution1/DevOps_Pipeline.png)
11. Attendi il completamento della fase **DEPLOY**.
12. Fai clic sull'**url** dell'applicazione sotto il risultato dell'ultima esecuzione per visualizzare le modifiche in tempo reale.

Continua ad apportare ulteriori modifiche alla tua applicazione ed esegui periodicamente il commit delle tue modifiche al tuo repository git. Se non vedi l'aggiornamento della tua applicazione, controlla i log delle fasi DEPLOY e BUILD della tua pipeline.

## Distribuisci a un'altra ubicazione
{: #deploy_another_region}

Distribuirai quindi la stessa applicazione a un'altra ubicazione {{site.data.keyword.Bluemix_notm}}. Possiamo utilizzare la stessa toolchain ma aggiungeremo un'altra fase DEPLOY per gestire la distribuzione dell'applicazione a un'altra ubicazione.

1. Vai alla panoramica (**Overview**) dell'applicazione e scorri per trovare **View toolchain**.
2. Seleziona **Delivery Pipeline** da Deliver.
3. Fai clic sull'**icona di ingranaggio** nella fase **DEPLOY** e seleziona **Clone Stage**.
   ![HelloWorld](images/solution1/CloneStage.png)
4. Rinomina la fase come "Deploy to UK" e seleziona **JOBS**.
5. Modifica **IBM Cloud region** in **London - https://api.eu-gb.bluemix.net**. Crea uno **spazio** se non ne hai uno.
6. Modifica **Deploy script** in `cf push "${CF_APP}" -d eu-gb.mybluemix.net`

   ![HelloWorld](images/solution1/DeployToUK.png)
7. Fai clic su **Save** ed esegui la nuova fase facendo clic sul **pulsante Play**.

## Registra un dominio personalizzato presso IBM Cloud Internet Services

{: #domain_cis}

IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) è una piattaforma uniforme per configurare e gestire DNS (Domain Name System), GLB (Global Load Balancing), WAF (Web Application Firewall) e la protezione da attacchi di tipo DDoS (Distributed Denial of Service) per le applicazioni web, Fornisce un servizio internet veloce, ad elevate prestazioni, affidabile e sicuro per i clienti che svolgono la loro attività commerciale su IBM Cloud con tre funzionalità principali per migliorare il flusso di lavoro; la sicurezza, l'affidabilità e le prestazioni.  

Quando distribuisci un'applicazione reale, probabilmente vorrai utilizzare un tuo dominio invece del dominio mybluemix.net fornito da IBM. In questo passo, una volta che disponi di un dominio personalizzato, puoi utilizzare i server DNS forniti da IBM Cloud Internet Services.

1. Acquista un dominio da un registrar come [http://godaddy.com](http://godaddy.com).
2. Passa a [Internet Services](https://{DomainName}/catalog/services/internet-services) nel catalogo {{site.data.keyword.Bluemix_notm}}.
2. Immetti un nome servizio e fai clic su **Create** per creare un'istanza del servizio.
3. Quando viene eseguito il provisioning dell'istanza del servizio, imposta il tuo nome dominio e fai clic su **Add domain**.
4. Quando vengono assegnati i server dei nomi, configura il tuo registrar o il provider del nome del dominio in modo che utilizzi i server dei nomi elencati.
5. Una volta configurato il tuo registrar o il provider DNS, potresti dover attendere fino a 24 ore affinché le modifiche diventino effettive. Quando lo stato del dominio nella pagina Overview cambia da *Pending* a *Active*, puoi utilizzare il comando `dig <your_domain_name> ns` per verificare che i server dei nomi IBM Cloud siano diventati effettivi.
  {:tip}

## Aggiungi il bilanciamento del carico di lavoro globale all'applicazione

{: #add_glb}

In questa sezione, utilizzerai il GLB (Global Load Balancer) in IBM Cloud Internet Services per gestire il traffico in più ubicazioni. Il GLB utilizza un pool di origine che consente la distribuzione del traffico a più origini.

### Prima di creare un GLB, crea un controllo dell'integrità per il GLB.

1. Nell'applicazione Cloud Internet Services, vai a **Reliability** > **Global Load Balancer** e, nella parte inferiore della pagina, fai clic su **Create health check**.
2. Immetti il percorso che desideri monitorare, ad esempio `/`, e seleziona un tipo (HTTP o HTTPS). Di norma, puoi creare un endpoint di integrità dedicato. Fai clic su **Provision 1 Instance**.
   ![Controllo dell'integrità](images/solution1/health_check.png)

### Dopo di che, crea un pool di origine con due origini.

1. Fai clic su **Create Pool**.
2. Immetti un nome per il pool, seleziona il controllo dell'integrità che hai appena creato e una regione che è vicina all'ubicazione della tua applicazione node.js.
3. Immetti un nome per la prima origine e il nome host per l'applicazione a Dallas `<your_app>.mybluemix.net`.
4. In modo analogo, aggiungi un'altra origine con l'indirizzo di origine che punta all'applicazione a Londra `<your_app>.eu-gb.mybluemix.net`.
5. Fai clic su **Provision 1 Instance**.
   ![Pool di origine](images/solution1/origin_pool.png)

### Crea un GLB (Global Load Balancer). 

1. Fai clic su **Create Load Balancer**.
2. Immetti un nome per il GLB (Global Load Balancer). Questo nome farà anche parte dell'URL applicazione universale (`http://<glb_name>.<your_domain_name>`), indipendentemente dall'ubicazione.
3. Fai clic su **Add pool** e seleziona il pool di origine che hai appena creato.
4. Fai clic su **Provision 1 Instance**.
   ![Global Load Balancer](images/solution1/load_balancer.png)

A questo punto, il GLB è configurato ma le applicazioni Cloud Foundry non sono ancora pronte a rispondere alle richieste dal nome dominio GLB configurato. Per completare la configurazione, aggiornerai le applicazioni con gli instradamenti utilizzando il dominio personalizzato.

## Configura il dominio personalizzato e gli instradamenti alla tua applicazione

{: #add_domain}

In questo passo, assocerai il nome dominio personalizzato all'endpoint sicuro per l'ubicazione {{site.data.keyword.Bluemix_notm}} dove è in esecuzione la tua applicazione.

1. Dalla barra dei menu, fai clic su **Manage** e quindi su **Account**: [Account](https://{DomainName}/account).
2. Nella pagina dell'account, vai all'applicazione **Cloud Foundry Orgs** e seleziona **Domains** dalla colonna Actions.
3. Fai clic su **Add a domain** e immetti il tuo nome dominio personalizzato acquisito dal registrar.
4. Seleziona l'ubicazione corretta e fai clic su **Save**.
5. In modo analogo, aggiungi il nome dominio personalizzato a Londra.
6. Ritorna all'elenco risorse ([Resource List](https://{DomainName}/resources)) di {{site.data.keyword.Bluemix_notm}}, vai a **Cloud Foundry Apps** e fai clic sull'applicazione a Dallas, fai clic su **Route** > **Edit Routes** e fai clic su **Add Route**.
   ![Add a route](images/solution1/ApplicationRoutes.png)
7. Immetti il nome host GLB che hai configurato in precedenza nel campo **Enter host (optional)** e seleziona il dominio personalizzato che hai appena aggiunto. Fai clic su **Save**.
8. In modo analogo, configura il dominio e gli instradamenti per l'applicazione a Londra.

A questo punto, puoi visitare la tua applicazione con l'URL `<glb_name>.<your_domain_name>` e il Global Load Balancer distribuisce automaticamente il traffico per le tue applicazioni multi-ubicazione. Puoi verificare ciò arrestando la tua applicazione a Dallas, tenendo l'applicazione a Londra attiva e accedendo all'applicazione tramite il Global Load Balancer.

Anche se al momento questa operazione funziona, poiché abbiamo configurato la fornitura continua nei passi precedenti, la configurazione può essere sovrascritta se viene attivata un'altra build. Per rendere persistenti queste modifiche, torna alle toolchain e modifica il file *manifest.yml*.

1. Nell'elenco risorse ([Resource List](https://{DomainName}/resources)) {{site.data.keyword.Bluemix_notm}}, vai a **Cloud Foundry Apps** e fai clic sull'applicazione a Dallas, vai alla panoramica (**Overview**) dell'applicazione e scorri per trovare **View toolchain**.
2. Seleziona il tile Git in Code.
3. Seleziona *manifest.yml*.
4. Fai clic su **Edit** e aggiungi gli instradamenti personalizzati. Sostituisci le configurazioni originali di dominio e host con solo `Routes`.

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <glb_name>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. Esegui il commit delle modifiche e assicurati che le build per entrambe le ubicazioni riescano.  

## Alternativa: associa il dominio personalizzato al dominio di sistema IBM Cloud

È possibile che tu non voglia utilizzare un Global Load Balancer davanti alle tue applicazioni multi-ubicazione ma devi associare il nome dominio personalizzato all'endpoint sicuro per l'ubicazione {{site.data.keyword.Bluemix_notm}} dove è in esecuzione la tua applicazione.

Con l'applicazione Cloud Internet Services, attieniti alla seguente procedura per configurare i record `CNAME` per la tua applicazione:

1. Nell'applicazione Cloud Internet Services, vai a **Reliability** > **DNS**.
2. Seleziona **CNAME** dall'elenco a discesa **Type**, immetti un alias per la tua applicazione nel campo name e l'URL applicazione nel campo del nome del dominio. L'applicazione `<your_app>.mybluemix.net` a Dallas può essere associata a una `<your_app>`.
3. Fai clic su **Add Record**. Posiziona l'interruttore PROXY su ON per migliorare la sicurezza della tua applicazione.
4. In modo analogo, imposta il record `CNAME` per l'endpoint Londra.
   ![Record CNAME](images/solution1/cnames.png)

Quando utilizzi un dominio predefinito diverso da `mybluemix.net`, come ad esempio `cf.appdomain.cloud` o `cf.cloud.ibm.com`, assicurati di utilizzare il [rispettivo dominio di sistema](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain).
{:tip}

Se stai utilizzando un provider DNS differente, la procedura per configurare il record CNAME varia a seconda del tuo provider DNS. Ad esempio, se stai utilizzando GoDaddy, ti attieni alle linee guida di [Domains Help](https://www.godaddy.com/help/add-a-cname-record-19236) fornite da GoDaddy.

Perché le tue applicazioni Cloud Foundry siano raggiungibili tramite il dominio personalizzato, dovrai aggiungere quest'ultimo all'[elenco di domini nell'organizzazione Cloud Foundry dove sono distribuite le applicazioni](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#updatingapps). Una volta eseguita questa operazione, puoi aggiungere gli instradamenti ai manifest dell'applicazione:

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <your_app>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}

## Esegui il bind del certificato SSL alla tua applicazione
{: #ssl}

1. Ottieni un certificato SSL. Ad esempio, puoi eseguire un acquisto da https://www.godaddy.com/web-security/ssl-certificate o generarne uno gratuito all'indirizzo https://letsencrypt.org/.
2. Vai alla panoramica (**Overview**) > **Routes** > **Manage Domains** dell'applicazione.
3. Fai clic sul pulsante di caricamento del certificato SSL e carica il certificato.
5. Accedi alla tua applicazione con https invece di http.

## Monitora le prestazioni dell'applicazione
{: #monitor}

Controlliamo l'integrità della tua applicazione multi-ubicazione.

1. Nel dashboard dell'applicazione, seleziona **Monitoring**.
2. Fai clic su **View All Tests**
   ![](images/solution1/alert_frequency.png)

Availability Monitoring esegue dei test sintetici dalle ubicazioni in tutto il mondo e ininterrottamente per rilevare e correggere in modo proattivo i problemi di prestazioni prima che abbiano delle ripercussioni sugli utenti. Se hai configurato un instradamento personalizzato per la tua applicazione, modifica la definizione di test per accedere alla tua applicazione tramite il suo dominio personalizzato.

## Rimuovi le risorse

* Elimina la toolchain
* Elimina le due applicazioni Cloud Foundry distribuite nelle due ubicazioni
* Elimina il GLB, i pool di origine e il controllo dell'integrità
* Elimina la configurazione DNS
* Elimina l'istanza Internet Services

## Contenuto correlato

[Aggiunta di un database Cloudant](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[Ridimensionamento automatico delle applicazioni Cloud Foundry](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
