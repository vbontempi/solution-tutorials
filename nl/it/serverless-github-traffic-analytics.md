---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Combinazione dell'azione senza server e di Cloud Foundry per il richiamo e l'analisi dei dati
{: #serverless-github-traffic-analytics}
In questa esercitazione, creerai un'applicazione per raccogliere automaticamente le statistiche del traffico GitHub per i repository e fornire la base per l'analisi del traffico. GitHub fornisce solo l'accesso ai dati del traffico per gli ultimi 14 giorni. Se vuoi analizzare le statistiche per un periodo di tempo più lungo, devi scaricare e archiviare tu stesso tali dati. In questa esercitazione, distribuirai un'azione senza server per richiamare i dati del traffico e archiviarli in un database SQL. Inoltre, viene utilizzata un'applicazione Cloud Foundry per gestire i repository e fornire l'accesso alle statistiche per l'analisi dei dati. L'applicazione e l'azione senza server trattate in questa esercitazione implementano una soluzione pronta a più tenant con la serie iniziale di funzioni che supporta la modalità a singolo tenant.

![](images/solution24-github-traffic-analytics/Architecture.png)

## Obiettivi

* Distribuire un'applicazione del database Python con il supporto a più tenant e l'accesso protetto
* Integrare l'ID applicazione come provider di autenticazione basato su OpenID Connect
* Configurare la raccolta senza server automatizzata delle statistiche del traffico GitHub
* Integrare {{site.data.keyword.dynamdashbemb_short}} per l'analisi grafica del traffico

## Prodotti
Questa esercitazione utilizza i seguenti prodotti:
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Prima di iniziare
{: #prereqs}

Per completare questa esercitazione, devi disporre della versione più recente della [CLI IBM Cloud](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) e devi avere [installato il plugin](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) di {{site.data.keyword.openwhisk_short}}.

## Configurazione del servizio e dell'ambiente (shell)
In questa sezione, configurerai i servizi necessari e preparerai l'ambiente. Queste operazioni possono essere eseguite dall'ambiente shell. 

1. Clona il [repository GitHub](https://github.com/IBM-Cloud/github-traffic-stats) e passa alla directory clonata e alla sua sottodirectory di **backend**:
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. Utilizza `ibmcloud login` per accedere in modo interattivo a {{site.data.keyword.Bluemix_short}}. Puoi riconfermare i dettagli eseguendo il comando `ibmcloud target`. Devi avere un'organizzazione e uno spazio configurati. 

3. Crea un'istanza {{site.data.keyword.dashdbshort}} con il piano d'ingresso (**Entry**) e denominala **ghstatsDB**:
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. Per accedere al servizio database da {{site.data.keyword.openwhisk_short}} in seguito, hai bisogno dell'autorizzazione. Pertanto, crea le credenziali del servizio ed etichettale **ghstatskey**:   
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. Crea un'istanza del servizio {{site.data.keyword.appid_short}}. Utilizza **ghstatsAppID** come nome e il piano di livello graduale (**Graduated tier**).
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
   Dopodiché, crea un alias di tale nuova istanza del servizio nello spazio Cloud Foundry. Sostituisci **YOURSPACE** con lo spazio in cui stai eseguendo la distribuzione.
   ```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. Crea un'istanza del servizio {{site.data.keyword.dynamdashbemb_short}} utilizzando il piano **lite**.
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
   Di nuovo, crea un alias di tale nuova istanza del servizio e sostituisci **YOURSPACE**.
   ```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. Nella directory **backend**, esegui il push dell'applicazione in IBM Cloud. Il comando utilizza un instradamento casuale per la tua applicazione. 
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
   Attendi il completamento della distribuzione. I file dell'applicazione vengono caricati, l'ambiente di runtime viene creato e i servizi vengono collegati all'applicazione. Le informazioni sul servizio vengono acquisite dal file `manifest.yml`. Devi aggiornare tale file se hai utilizzato altri nomi del servizio. Una volta completato correttamente il processo, viene visualizzato l'URI dell'applicazione. 

   Il comando sopra riportato utilizza un instradamento casuale ma univoco per l'applicazione. Se vuoi sceglierne uno tu stesso, aggiungilo come parametro aggiuntivo al comando, ad esempio `ibmcloud cf push your-app-name`. Puoi anche modificare il file `manifest.yml`, modificare il **nome** e modificare **random-route** da **true** a **false**.
   {:tip}

## Configurazione dell'ID applicazione e GitHub (browser)
I passi riportati di seguito sono tutti eseguiti utilizzando il tuo browser Internet. Innanzitutto, configura {{site.data.keyword.appid_short}} per utilizzare Cloud Directory e l'applicazione Python. Dopodiché, crea un token di accesso GitHub. È necessario per la funzione distribuita per richiamare i dati del traffico. 

1. Nell'[elenco delle risorse {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), apri la panoramica dei tuoi servizi. Individua l'istanza del servizio {{site.data.keyword.appid_short}} nella sezione **Services**. Fai clic sulla sua voce per aprire i dettagli. 
2. Nel dashboard del servizio, fai clic su **Manage** in **Identity Providers** nel menu a sinistra. Visualizza un elenco dei provider di identità disponibili, ad esempio Facebook, Google, SAML 2.0 Federation e Cloud Directory. Commuta Cloud Directory in **On**, tutti gli altri provider in **Off**.
   
   Potresti voler configurare [Multi-Factor Authentication (MFA)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) e le regole password avanzate. Questi argomenti non vengono trattati in questa esercitazione.
   {:tip}

3. Alla fine di quella pagina troverai l'elenco degli URL di reindirizzamento. Immetti l'**url** della tua applicazione + /redirect_uri. Ad esempio `https://github-traffic-stats-random-word.mybluemix.net/redirect_uri`.

   Per verificare l'applicazione in locale, l'URL di reindirizzamento è `http://0.0.0.0:5000/redirect_uri`. Puoi configurare più URL di reindirizzamento.
   {:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. Nel menu a sinistra, fai clic su **Users**. Apre un elenco degli utenti in Cloud Directory. Fai clic sul pulsante **Add User** per aggiungere te stesso come primo utente. Ora hai terminato la configurazione del servizio {{site.data.keyword.appid_short}}.
5. Nel browser, visita [Github.com](https://github.com/settings/tokens) e vai a **Settings -> Developer settings -> Personal access tokens**. Fai clic sul pulsante **Generate new token**. Immetti **GHStats Tutorial** per la **descrizione del token**. Dopodiché, abilita **public_repo** nella categoria **repo** e **read:org** in **admin:org**. Ora, alla fine di tale pagina, fai clic su **Generate token**. Il nuovo token di accesso viene visualizzato nella pagina successiva. Ne avrai bisogno durante la configurazione dell'applicazione che segue.
   ![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## Configura e verifica l'applicazione Python
Dopo la preparazione, configura e verifica l'applicazione. L'applicazione viene scritta in Python utilizzando il popolare microframework [Flask](http://flask.pocoo.org/). I repository possono essere aggiunti e rimossi dalla raccolta delle statistiche. Puoi accedere ai dati del traffico in una vista tabulare. 

1. In un browser, apri l'URI dell'applicazione distribuita. Dovresti vedere una pagina di benvenuto.
   ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. Nel browser, aggiungi `/admin/initialize-app` all'URI e accedi alla pagina. Viene utilizzato per inizializzare l'applicazione e i suoi dati. Fai clic sul pulsante **Start initialization**. Questa operazione ti porterà a una pagina di configurazione protetta da password. L'indirizzo email che hai utilizzato per l'accesso viene acquisito come identificazione per l'amministratore di sistema. Utilizza l'indirizzo email e la password che hai configurato in precedenza. 

3. Nella pagina di configurazione, immetti un nome (viene utilizzato per il messaggio iniziale), il tuo nome utente GitHub e il token di accesso che hai generato prima. Fai clic su **Initialize**. Ciò crea le tabelle database e inserisce alcuni valori di configurazione. Infine, crea record di database per l'amministratore di sistema e un tenant.
   ![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. Una volta terminato, vieni portato all'elenco dei repository gestiti. Ora puoi aggiungere i repository fornendo il nome dell'account GitHub o dell'organizzazione e il nome del repository. Dopo aver immesso i dati, fai clic su **Add repository**. Il repository, insieme a un identificativo di nuova assegnazione, deve comparire nella tabella. Puoi rimuovere i repository dal sistema immettendo il loro ID e facendo clic su **Delete repository**.
![](images/solution24-github-traffic-analytics/RepositoryList.png)

## Distribuisci Cloud Function e un trigger
Con l'applicazione di gestione implementata, distribuisci un'azione, un trigger e una regola per connettere i due per {{site.data.keyword.openwhisk_short}}. Questi oggetti vengono utilizzati per raccogliere automaticamente i dati del traffico GitHub nella pianificazione specificata. L'azione si connette al database, scorre tutti i tenant e i loro repository e ottiene la vista e i dati di clonazione per ciascun repository. Tali statistiche vengono integrate nel database.

1. Vai alla directory **functions**.
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. Crea una nuova azione **collectStats**. Viene utilizzato un [ambiente Python 3](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments) che include già il driver del database richiesto. Il codice sorgente per l'azione viene fornito nel file `ghstats.zip`.
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   Se modifichi il codice sorgente per l'azione (`__main__.py`), puoi reimpacchettare di nuovo l'archivio zip con `zip -r ghstats.zip  __main__.py github.py`. Per i dettagli, vedi il file `setup.sh`.
   {:tip}
3. Esegui il bind dell'azione al servizio database. Utilizza l'istanza e la chiave del servizio che hai creato durante la configurazione dell'ambiente. 
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. Crea un trigger basato sul [pacchetto allarmi](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm). Supporta diversi formati per specificare l'allarme. Utilizza lo stile come [cron](https://en.wikipedia.org/wiki/Cron). A partire dal 21 aprile e fino al 21 dicembre, il trigger si attiva ogni giorno alle 6am UTC. Assicurati di avere una data di inizio futura. 
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  Puoi modificare il trigger da una pianificazione giornaliera a una settimanale applicando `"0 6 * * 0"`. Questa modifica lo attiverà ogni domenica alle 6am.
  {:tip}
5. Infine, crea una regola **myStatsRule** che connetta il trigger **myDaily** all'azione **collectStats**. Ora, il trigger fa in modo che l'azione venga eseguita in base alla pianificazione specificata nel passo precedente. 
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. Richiama l'azione per l'esecuzione del test iniziale. Il valore di **repoCount** riportato dovrebbe riflettere il numero di repository che hai configurato in precedenza. 
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
   L'output sarà simile al seguente:
   ```
   {
       "repoCount": 18
   }
   ```
7. Ora, nella finestra del tuo browser con la pagina dell'applicazione, puoi visitare il traffico del repository. Per impostazione predefinita, vengono visualizzate 10 voci. Puoi modificarla con un valore diverso. Puoi anche ordinare le colonne della tabella oppure utilizzare la casella di ricerca per filtrare repository specifici. Puoi immettere una data e un nome organizzazione e poi eseguire l'ordinamento in base a viewcount per elencare i primi punteggi per un particolare giorno.
   ![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## Conclusioni
In questa esercitazione, hai distribuito un'azione senza server e un trigger e una regola correlati. Consentono di richiamare automaticamente i dati del traffico per i repository GitHub. Le informazioni su tali repository, incluso il token di accesso specifico del tenant, vengono archiviate in un database SQL ({{site.data.keyword.dashdbshort}}). Tale database viene utilizzato dall'applicazione Cloud Foundry per gestire gli utenti, i repository e per presentare le statistiche del traffico nel portale dell'applicazione. Gli utenti possono vedere le statistiche del traffico in tabelle ricercabili o visualizzate in un dashboard integrato (servizio {{site.data.keyword.dynamdashbemb_short}}, vedi immagine riportata di seguito). È anche possibile scaricare l'elenco dei repository e i dati del traffico come file CSV.

L'applicazione Cloud Foundry gestisce l'accesso tramite un client OpenID Connect connesso a {{site.data.keyword.appid_short}}.
![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## Ripulitura
Per ripulire le risorse utilizzate per questa esercitazione, puoi eliminare l'applicazione e i servizi correlati come pure l'azione, il trigger e la regola nell'ordine inverso in cui sono stati creati: 

1. Elimina la regola, il trigger e l'azione {{site.data.keyword.openwhisk_short}}.
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. Elimina l'applicazione Python e i suoi servizi.
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## Espandi l'esercitazione
Vuoi aggiungere qualcosa o modificare questa esercitazione? Ecco alcune idee:
* Espandi l'applicazione per il supporto a più tenant.
* Integra un grafico per i dati. 
* Utilizza i provider di identità social.
* Aggiungi un selettore di data alla pagina delle statistiche per filtrare i dati visualizzati. 
* Utilizza una pagina di accesso personalizzata per {{site.data.keyword.appid_short}}.
* Esplora le relazioni di social coding tra gli sviluppatori utilizzando [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Contenuto correlato
Ecco i link a informazioni aggiuntive sugli argomenti trattati in questa esercitazione. 

Documentazione e SDK:
* [Documentazione di {{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Documentazione: [IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Documentazione di {{site.data.keyword.appid_short}}](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [Runtime di Python su IBM Cloud](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
