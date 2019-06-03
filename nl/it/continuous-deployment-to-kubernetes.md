---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# Distribuzione continua su Kubernetes
{: #continuous-deployment-to-kubernetes}

Questa esercitazione ti guida nel processo di configurazione di una pipeline di fornitura e integrazione continue per le applicazioni inserite nei contenitori in esecuzione su {{site.data.keyword.containershort_notm}}.  Imparerai come configurare il controllo origine, quindi come creare, verificare e distribuire il codice in diverse fasi di distribuzione. Successivamente, aggiungerai integrazioni ad altri servizi come scanner della sicurezza, notifiche Slack e analisi.

{:shortdesc}

## Obiettivi
{: #objectives}

* Creare cluster Kubernetes di sviluppo e produzione.
* Creare un'applicazione starter, eseguirla in locale ed eseguirne il push in un repository Git.
* Configurare la pipeline di fornitura DevOps per la connessione al tuo repository Git, creare e distribuire l'applicazione starter sui cluster di sviluppo/produzione.
* Esplorare e integrare l'applicazione per utilizzare scanner della sicurezza, notifiche Slack e analisi.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti servizi {{site.data.keyword.Bluemix_notm}}:

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**Attenzione:** questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

![](images/solution21/Architecture.png)

1. Esegui il push del codice a un repository Git privato.
2. La pipeline raccoglie le modifiche in Git e crea l'immagine del contenitore.
3. L'immagine del contenitore viene caricata nel registro distribuito in un cluster Kubernetes di sviluppo.
4. Convalidare le modifiche ed eseguire la distribuzione nel cluster di produzione.
5. Configurazione delle notifiche Slack per le attività di distribuzione.


## Prima di iniziare
{: #prereq}

* [Installa {{site.data.keyword.dev_cli_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - Script per installare docker, kubectl, helm, CLI ibmcloud e i plugin richiesti.
* [Configura la CLI {{site.data.keyword.registrylong_notm}} e il tuo spazio dei nomi del registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Comprendi i principi di base di Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Crea il cluster Kubernetes di sviluppo
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} offre potenti strumenti combinando le tecnologie Docker e Kubernetes, un'esperienza utente intuitiva e la sicurezza e l'isolamento integrati per automatizzare la distribuzione, il funzionamento, il ridimensionamento e il monitoraggio di applicazioni inserite in un contenitore in un cluster di host di calcolo.

Per completare questa esercitazione, dovrai selezionare il cluster **a pagamento** di tipo **Standard**. Ti verrà richiesto di configurare due cluster, uno per lo sviluppo e uno per la produzione.
{: shortdesc}

1. Crea il primo cluster di sviluppo Kubernetes dal [catalogo {{site.data.keyword.Bluemix}}](https://{DomainName}/containers-kubernetes/launch). In seguito, ti verrà richiesto di ripetere questi passi e creare un cluster di produzione.

   Per facilità di utilizzo, controlla i dettagli di configurazione come il numero di CPU, la memoria e il numero di nodi di lavoro che ottieni.
   {:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. Seleziona il tipo di cluster (**Cluster type**) e fai clic su **Create Cluster** per eseguire il provisioning di un cluster Kubernetes. Il **tipo di macchina** più piccolo con 2 **CPU**, 4 **GB di RAM** e 1 **nodo di lavoro** è sufficiente per questa esercitazione. Tutte le altre opzioni possono essere lasciate sui loro valori predefiniti.
3. Controlla lo stato del tuo **cluster** e **nodo di lavoro** e attendi che diventi **ready**.

**Nota:** non procedere finché i tuoi nodi di lavoro non sono pronti.

## Crea un'applicazione starter
{: #create_application}

{{site.data.keyword.containershort_notm}} offre una selezione di applicazioni starter; queste applicazioni starter possono essere create utilizzando il comando `ibmcloud dev create` o la console web. In questa esercitazione, utilizzeremo la console web. L'applicazione starter riduce notevolmente i tempi di sviluppo generando starter dell'applicazione con tutto il codice necessario per contenitore tipo, creazione e configurazione in modo che tu possa iniziare a codificare la logica di business più velocemente.

1. Dalla [console {{site.data.keyword.cloud_notm}}](https://{DomainName}), utilizza l'opzione di menu a sinistra e seleziona [Web Apps](https://{DomainName}/developer/appservice/dashboard).
2. Nella sezione **Start from the Web**, fai clic sul pulsante **Get Started**.
3. Seleziona il tile `Node.js Web App with Express.js` e quindi `Create app` per creare un'applicazione starter Node.js.
4. Immetti il **nome** `mynodestarter`. Quindi, fai clic su **Create**.

## Configura la pipeline di fornitura DevOps
{: #create_devops}

1. Ora che hai creato correttamente l'applicazione starter, in **Deploy your App**, fai clic sul pulsante **Deploy to Cloud**.
2. Selezionando il metodo di distribuzione del cluster Kubernetes, seleziona il cluster creato in precedenza e fai quindi clic su **Create**. Ciò creerà una pipeline di strumenti e di fornitura per te. ![](images/solution21/BindCluster.png)
3. Una volta creata la pipeline, fai clic su **View Toolchain** e quindi su **Delivery Pipeline** per visualizzare la pipeline. ![](images/solution21/Delivery-pipeline.png)
4. Dopo aver completato le fasi di distribuzione, fai clic su **View logs and history** per visualizzare i log.
5. Visita l'URL visualizzato per accedere all'applicazione (`http://worker-public-ip:portnumber/`). ![](images/solution21/Logs.png)
Hai utilizzato l'interfaccia utente App Service per creare le applicazioni starter e hai configurato la pipeline per creare e distribuire l'applicazione sul tuo cluster.

## Clona, crea ed esegui l'applicazione in locale
{: #cloneandbuildapp}

In questa sezione, utilizzerai l'applicazione starter creata nella sezione precedente, la clonerai nella tua macchina locale, modificherai il codice e quindi la creerai/eseguirai in locale.
{: shortdesc}

### Clona l'applicazione
1. Dalla panoramica della toolchain, seleziona il tile **Git** in **Code**. Verrai reindirizzato alla tua pagina di repository git dove potrai clonare il repository.![HelloWorld](images/solution21/DevOps_Toolchain.png)

2. Se non hai ancora configurato le chiavi SSH, dovresti vedere una barra di notifica in alto con le istruzioni. Segui la procedura aprendo il link **Add an SSH key** in una nuova scheda o, se vuoi utilizzare HTTPS anziché SSH, segui la procedura facendo clic su **Create a personal access token**. Ricordati di salvare la chiave o il token per riferimenti futuri.
3. Seleziona SSH o HTTPS e copia l'URL git. Clona l'origine alla tua macchina locale. Se ti viene richiesto un nome utente, fornisci il tuo nome utente git. Per la password, utilizza una **chiave SSH** o un **token di accesso personale** esistente o quello che hai creato nel passo precedente.

   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   {: codeblock}

4. Apri il repository clonato in un IDE di tua scelta e passa a `public/index.html`. Aggiorna il codice provando a modificare "Congratulations!" con qualcos'altro e salva quindi il file.

### Crea l'applicazione in locale
Puoi creare ed eseguire l'applicazione come faresti normalmente con `mvn` per lo sviluppo locale java o con `npm` per lo sviluppo dei nodi.  Puoi anche creare un'immagine docker ed eseguire l'applicazione in un contenitore per garantire l'esecuzione coerente in locale e nel cloud. Per creare la tua immagine docker, utilizza la seguente procedura.
{: shortdesc}

1. Assicurati che il tuo motore Docker locale sia avviato, per controllare immetti il seguente comando:
   ```
   docker ps
   ```
   {: codeblock}
2. Passa alla directory del progetto generato clonata.
   ```
   cd <project name>
   ```
   {: codeblock}
3. Crea l'applicazione in locale.
   ```
   ibmcloud dev build
   ```
   {: codeblock}

   L'esecuzione di questo comando potrebbe richiedere alcuni minuti poiché vengono scaricate tutte le dipendenze dell'applicazione e viene creata un'immagine Docker, che contiene la tua applicazione e tutto l'ambiente richiesto.

### Esegui l'applicazione in locale

1. Esegui il contenitore.
   ```
   ibmcloud dev run
   ```
   {: codeblock}

   Questo utilizza il tuo motore Docker locale per eseguire l'immagine docker che hai creato nel passo precedente.
2. Dopo l'avvio del tuo contenitore, vai all'indirizzo http://localhost:3000/
   ![](images/solution21/node_starter_localhost.png)

## Esegui il push dell'applicazione al tuo repository Git

In questa sezione, eseguirai il commit della tua modifica al repository Git. La pipeline raccoglierà il commit ed eseguirà automaticamente il push delle modifiche al tuo cluster.
1. Nella finestra del terminale, assicurati di essere all'interno del repository che hai clonato.
2. Esegui il push della modifica al tuo repository con tre semplici passi: aggiunta, commit e push.
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
   {: codeblock}

3. Vai alla toolchain che hai creato in precedenza e fai clic sul tile **Delivery Pipeline**.
4. Conferma di vedere le fasi **BUILD** e **DEPLOY**.
   ![](images/solution21/Delivery-pipeline.png)
5. Attendi il completamento della fase **DEPLOY**.
6. Fai clic sull'**url** dell'applicazione sotto il risultato dell'ultima esecuzione per visualizzare le modifiche in tempo reale.

Se non vedi l'aggiornamento della tua applicazione, controlla i log delle fasi DEPLOY e BUILD della tua pipeline.

## Sicurezza tramite Vulnerability Advisor
{: #vulnerability_advisor}

In questo passo, esplorerai [Vulnerability Advisor](https://{DomainName}/docs/services/va?topic=va-va_index#va_index). Il controllo delle vulnerabilità viene utilizzato per controllare lo stato di sicurezza delle immagini del contenitore prima della distribuzione e controlla anche lo stato dei contenitori in esecuzione.

1. Vai alla toolchain che hai creato in precedenza e fai clic sul tile **Delivery Pipeline**.
1. Fai clic su **Add Stage** e modifica MyStage in **Validate Stage**, quindi fai clic su JOBS  > **ADD JOB**.

   1. Seleziona **Test** come tipo di lavoro e modifica **Test** in **Vulnerability advisor** nella casella.
   1. In Tester type, seleziona **Vulnerability Advisor**. Tutti gli altri campi dovrebbero venire popolati automaticamente.
      Lo spazio dei nomi Container Registry dovrebbe essere uguale a quello menzionato in **Build Stage** di questa toolchain.
      {:tip}
   1. Modifica la sezione **Test script** e sostituisci `SAFE\ to\ deploy` nell'ultima riga con `NO\ ISSUES`
   1. Salva la fase
1. Trascina e sposta la **Validate Stage** al centro, quindi fai clic su **Run** ![](images/solution21/run.png) nella **Validate Stage**. Vedrai che la **Validate stage** non riesce.

   ![](images/solution21/toolchain.png)

1. Fai clic su **View logs and history** per visualizzare la valutazione delle vulnerabilità. La fine del log indica:

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   Puoi vedere le valutazioni di vulnerabilità dettagliate di tutti i repository scansionati facendo clic [qui](https://{DomainName}/containers-kubernetes/registry/private)
   {:tip}

   La fase potrebbe non riuscire indicando che l'immagine *non è stata scansionata* se la scansione per il rilevamento delle vulnerabilità impiega più di 3 minuti. Questo timeout può essere cambiato modificando lo script del lavoro e aumentando il numero di iterazioni da attendere per i risultati della scansione.
   {:tip}

1. Correggi le vulnerabilità seguendo l'azione correttiva. Apri il repository clonato in un IDE o seleziona il tile Eclipse Orion web IDE, apri `Dockerfile` e aggiungi il seguente comando dopo `EXPOSE 3000`
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. Esegui il commit e il push delle modifiche. Questo dovrebbe attivare la toolchain e correggere la **Validate Stage**.

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## Crea il cluster Kubernetes di produzione

{: #deploytoproduction}

In questa sezione, completerai la pipeline di distribuzione distribuendo l'applicazione Kubernetes rispettivamente agli ambienti di sviluppo e di produzione. Idealmente, vogliamo configurare una distribuzione automatica per l'ambiente di sviluppo e una distribuzione manuale per l'ambiente di produzione. Prima di farlo, esploriamo i due modi in cui puoi ottenere tutto questo. È possibile utilizzare un cluster per entrambi gli ambienti di sviluppo e produzione. Tuttavia, si consiglia di avere due cluster separati, uno per lo sviluppo e uno per la produzione. Esploriamo la configurazione di un secondo cluster per la produzione.
{: shortdesc}

1. Segui le istruzioni riportate nella sezione [Crea il cluster Kubernetes di sviluppo](#create_kube_cluster) e crea un nuovo cluster. Denomina questo cluster `prod-cluster`.
2. Vai alla toolchain che hai creato in precedenza e fai clic sul tile **Delivery Pipeline**.
3. Rinomina la **Deploy Stage** come `Deploy dev`, puoi farlo facendo clic sull'icona delle impostazioni >  **Configure Stage**.
   ![](images/solution21/deploy_stage.png)
4. Clona la fase **Deploy dev** (icona impostazioni > Clone Stage) e denomina la fase clonata come `Deploy prod`.
5. Modifica **stage trigger** in `Run jobs only when this stage is run manually`. ![](images/solution21/prod-stage.png)
6. Nella scheda **Job**, modifica il nome del cluster nel cluster appena creato e quindi **salva** la fase.
7. Ora dovresti avere la configurazione di distribuzione completa; per distribuire dallo sviluppo alla produzione devi eseguire manualmente la fase `Deploy prod` per la distribuzione in produzione. ![](images/solution21/full-deploy.png)

A questo punto, hai creato un cluster di produzione e hai configurato la pipeline per eseguire manualmente il push degli aggiornamenti al tuo cluster di produzione. Questa è una fase del processo di semplificazione in uno scenario più avanzato in cui dover includere test di unità e test di integrazione come parte della pipeline.

## Configura le notifiche Slack
{: #setup_slack}

1. Torna indietro per visualizzare l'elenco di [toolchain](https://{DomainName}/devops/toolchains), seleziona la tua toolchain e fai clic su **Add a Tool**.
2. Cerca slack nella casella di ricerca o scorri verso il basso per vedere **Slack**. Fai clic per visualizzare la pagina di configurazione.
    ![](images/solution21/configure_slack.png)
3. Per **Slack webhook**, segui la procedura riportata in questo [link](https://my.slack.com/services/new/incoming-webhook/). Devi accedere con le tue credenziali Slack e fornire un nome di canale esistente o crearne uno nuovo.
4. Una volta aggiunta l'integrazione del webhook in entrata, copia l'**URL Webhook** e incollalo in **Slack webhook**.
5. Il canale slack è il nome del canale che hai fornito durante la precedente creazione di un'integrazione webhook.
6. **Slack team name** è il nome team (prima parte) di team-name.slack.com. Ad esempio, kube è il nome team in kube.slack.com
7. Fai clic su **Create Integration**. Verrà aggiunto un nuovo tile alla tua toolchain.
    ![](images/solution21/toolchain_slack.png)
8. D'ora in poi, ogni volta che la tua toolchain viene eseguita, dovresti vedere le notifiche Slack nel canale che hai configurato.
    ![](images/solution21/slack_channel.png)

## Rimuovi le risorse
{: #removeresources}

In questo passo, ripulirai le risorse per rimuovere ciò che hai creato in precedenza.

- Elimina il repository Git.
- Elimina la toolchain.
- Elimina i due cluster.
- Elimina il canale Slack.

## Espandi l'esercitazione
{: #expandTutorial}

Vuoi saperne di più? Ecco alcune idee su cosa puoi fare successivamente:

- [Analizza i log e monitora l'integrità dell'applicazione con LogDNA e Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Aggiungi un ambiente di test e distribuiscilo in un terzo cluster.
- Distribuisci il cluster di produzione [in più ubicazioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Migliora la tua pipeline con ulteriori controlli di qualità e analisi utilizzando [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Contenuto correlato
{: #related}

* Guida alla soluzione Kubernetes end-to-end, [spostamento di applicazioni basate su VM in Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes).
* [Sicurezza](https://{DomainName}/docs/containers?topic=containers-security#cluster) per IBM Cloud Container Service.
* [Integrazioni](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations) della toolchain.
* Analizza i log e monitora l'integrità dell'applicazione con [LogDNA e Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).


