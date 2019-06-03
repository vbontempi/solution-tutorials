---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Crea uno Slackbot controllato dal database
{: #slack-chatbot-database-watson}

In questa esercitazione, creerai uno Slackbot per generare e ricercare eventi e conferenze nelle voci del database Db2. Slackbot è supportato dal servizio {{site.data.keyword.conversationfull}}. Integrerai Slack e {{site.data.keyword.conversationfull}} utilizzando un'integrazione Assistant.

L'integrazione Slack incanala i messaggi tra Slack e {{site.data.keyword.conversationshort}}. Lì, alcune azioni del lato server eseguono query SQL su un database Db2. Tutto il codice (effettivamente non tantissimo) è scritto in Node.js.

## Obiettivi
{: #objectives}

* Collegare {{site.data.keyword.conversationfull}} a Slack utilizzando un'integrazione
* Creare, distribuire ed eseguire il bind delle azioni Node.js in {{site.data.keyword.openwhisk_short}}
* Accedere a un database Db2 da {{site.data.keyword.openwhisk_short}} utilizzando Node.js

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}} ](https://{DomainName}/catalog/services/db2-warehouse) o [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura
{: #architecture}

<p style="text-align: center;">

  ![Architettura](images/solution19/SlackbotArchitecture.png)
</p>

## Prima di iniziare
{: #prereqs}

Per completare questa esercitazione, devi disporre della versione più recente della [CLI {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) e devi avere [installato il plugin](/docs/cli?topic=cloud-cli-plug-ins) di {{site.data.keyword.openwhisk_short}}.


## Configurazione del servizio e dell'ambiente
In questa sezione, configurerai i servizi necessari e preparerai l'ambiente. La maggior parte di queste operazioni può essere eseguita dalla CLI utilizzando gli script. Sono disponibili su GitHub.

1. Clona il [repository GitHub](https://github.com/IBM-Cloud/slack-chatbot-database-watson) e passa alla directory clonata:
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. Se non sei collegato, utilizza `ibmcloud login` per accedere in modo interattivo.
3. Stabilisci l'organizzazione e lo spazio con cui creare il servizio database:
   ```
   ibmcloud target --cf
   ```
4. Crea un'istanza {{site.data.keyword.dashdbshort}} e denominala **eventDB**:
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
   Puoi anche utilizzare un piano diverso da **Entry**.
5. Per accedere al servizio database da {{site.data.keyword.openwhisk_short}} in seguito, hai bisogno dell'autorizzazione. Pertanto, crea le credenziali del servizio ed etichettale **slackbotkey**:   
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. Crea un'istanza del servizio {{site.data.keyword.conversationshort}}. Utilizza **eventConversation** come nome e il piano Lite gratuito.
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. Registrerai quindi le azioni per {{site.data.keyword.openwhisk_short}} ed eseguirai il bind delle credenziali del servizio a tali azioni. Alcune delle azioni sono abilitate come azioni web e viene impostato un segreto per impedire richiami non autorizzati. Scegli un segreto e passalo come parametro - sostituisci **YOURSECRET** di conseguenza.

   Una delle azioni viene richiamata per creare una tabella in {{site.data.keyword.dashdbshort}}. Utilizzando un'azione di {{site.data.keyword.openwhisk_short}}, non hai bisogno di un driver Db2 locale né di utilizzare l'interfaccia basata sul browser per creare manualmente la tabella. Per eseguire la registrazione e la configurazione, esegui la riga riportata di seguito che eseguirà il file **setup.sh** che contiene tutte le azioni. Se il tuo sistema non supporta i comandi shell, copia ogni riga dal file **setup.sh** ed eseguila singolarmente. 

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **Nota:** per impostazione predefinita, lo script inserisce anche alcune righe di dati di esempio. Puoi disabilitare ciò eliminando il commento della seguente riga nello script sopra riportato: `#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r`
8. Estrai le informazioni sullo spazio dei nomi per le azioni distribuite. 

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   Annota la parte prima di **/slackdemo/eventInsert**. Si tratta dell'organizzazione e dello spazio codificati. Ne avrai bisogno nella prossima sezione. 

## Carica la capacità/lo spazio di lavoro
In questa parte dell'esercitazione, caricherai uno spazio di lavoro predefinito o una capacità predefinita nel servizio {{site.data.keyword.conversationshort}}.
1. Nell'[elenco delle risorse {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), apri la panoramica dei tuoi servizi. Individua l'istanza del servizio {{site.data.keyword.conversationshort}} creata nella sezione precedente. Fai clic sulla sua voce e poi sull'alias del servizio per aprire i dettagli del servizio. 
2. Fai clic su **Launch Tool** per richiamare lo strumento {{site.data.keyword.conversationshort}}.
3. Passa a **Skills**, quindi fai clic su **Create skill** e poi su **Import skill**.
4. Nella finestra di dialogo, dopo aver fatto clic su **Choose JSON file**, seleziona il file **assistant-skill.json** dalla directory locale. Lascia l'opzione di importazione impostata su **Everything (Intents, Entities, and Dialog)**, quindi fai clic su **Import**. Ciò crea una nuova capacità denominata **TutorialSlackbot**.
5. Fai clic su **Dialog** per vedere i nodi di dialogo. Puoi espanderli per vedere una struttura come quella riportata di seguito. 

   La finestra di dialogo contiene dei nodi per gestire le domande per il supporto e una semplice frase di ringraziamento ("Thank You"). Il nodo **newEvent** e il suo elemento secondario raccolgono l'input necessario e poi richiamano un'azione per inserire un nuovo record dell'evento in Db2.

   Il nodo **query events** chiarisce se gli eventi vengono ricercati in base al loro identificativo o alla data. La ricerca effettiva e la raccolta dei dati necessari vengono poi eseguite dai nodi secondari **query events by shortname** e **query event by dates**.

   **credential_node** configura il segreto per le azioni di dialogo e le informazioni sull'organizzazione Cloud Foundry. Queste ultime sono necessarie per richiamare le azioni. 

  I dettagli verranno spiegati in seguito, una volta completata la configurazione.
  ![](images/solution19/SlackBot_Dialog.png)   
6. Fai clic sul nodo di dialogo **credential_node**, apri l'editor JSON facendo clic sull'icona del menu alla destra di **Then respond with**.

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   Sostituisci **org_space** con le informazioni codificate sull'organizzazione e sullo spazio che hai recuperato in precedenza. Sostituisci "@" con "%40". Cambia quindi **YOURSECRET** con il tuo segreto effettivo generato in precedenza. Chiudi l'editor JSON facendo di nuovo clic sull'icona. 

## Crea un assistente e integralo con Slack

Ora, creerai un assistente associato alla capacità di prima e lo integrerai con Slack. 
1. Fai clic su **Skills** in alto a sinistra, quindi seleziona **Assistants**. Poi, fai clic su **Create assistant**.
2. Nella finestra di dialogo, utilizza **TutorialAssistant** come nome, quindi fai clic su **Create assistant**. Nella schermata successiva, scegli **Add dialog skill**. Dopodiché, scegli **Add existing skill**, seleziona **TutorialSlackbot** dall'elenco e aggiungilo. 
3. Una volta aggiunta la capacità, fai clic su **Add integration**, poi, dall'elenco delle integrazioni gestite (**Managed integrations**), seleziona **Slack**.
4. Segui le istruzioni per integrare il tuo chatbot con Slack.

## Verifica lo Slackbot e acquisisci informazioni
Apri il tuo spazio di lavoro Slack per un test drive del chatbot. Avvia una chat diretta con il bot.

1. Immetti **help** nel modulo di messaggistica. Il bot dovrebbe rispondere con alcune indicazioni. 
2. Ora immetti **new event** per iniziare a raccogliere i dati per un nuovo record dell'evento. Utilizzerai gli slot {{site.data.keyword.conversationshort}} per raccogliere tutto l'input necessario. 
3. Il primo è il nome o l'identificativo dell'evento. Le virgolette sono obbligatorie. Ti consentono di immettere nomi più complessi. Immetti **"Meetup: IBM Cloud"** come nome dell'evento. Il nome dell'evento viene definito come un'entità basata sul pattern **eventName**. Supporta diversi tipi di virgolette all'inizio e alla fine. 
4. Il successivo è l'ubicazione dell'evento. L'input si basa sull'[entità di sistema **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details). Come limitazione, possono essere utilizzate solo le città riconosciute da {{site.data.keyword.conversationshort}}. Prova ad utilizzare **Friedrichshafen** come città. 
5. Le informazioni di contatto, come un indirizzo email o l'URI per un sito web, vengono richieste nel prossimo passo. Inizia con **https://www.ibm.com/events**. Utilizzerai un'entità basata sul pattern per tale campo. 
6. Le prossime domande stanno raccogliendo la data e l'ora per l'inizio e la fine. Vengono utilizzati **sys-date** e **sys-time** che consentono formati di input diversi. Utilizza **next Thursday** come data di inizio, **6 pm** per l'ora, utilizza la data esatta di giovedì prossimo, ad esempio, **2019-05-09** e **22:00** per la data e l'ora di fine. 
7. Infine, con tutti i dati raccolti, viene stampato un riepilogo e viene richiamata un'azione server, implementata come azione {{site.data.keyword.openwhisk_short}}, per inserire un nuovo record in Db2. Dopodiché, la finestra di dialogo passa a un nodo secondario per ripulire l'ambiente di elaborazione eliminando le variabili di contesto. L'intero processo di input può essere annullato in qualsiasi momento immettendo **cancel**, **exit** o simili. In tal caso, viene riconosciuta la scelta utente e l'ambiente viene ripulito.
  ![](images/solution19/SlackSampleChat.png)   

Con alcuni dati di esempio al suo interno, è il momento di effettuare la ricerca. 
1. Immetti **show event information**. Segue una domanda per stabilire se effettuare la ricerca in base all'identificativo e alla data. Immetti un **nome** e per la domanda successiva **"Think 2019"**. Ora, il chatbot dovrebbe visualizzare le informazioni relative a tale evento. Il dialogo ha più risposte tra cui scegliere. 
2. Con {{site.data.keyword.conversationshort}} come backend, è possibile immettere frasi più complesse e quindi ignorare parti del dialogo. Utilizza **show event by the name "Think 2019"** come input. Il chatbot restituisce direttamente il record dell'evento. 
3. Ora eseguirai la ricerca in base alla data. Una ricerca viene definita da una coppia di date, la data di inizio dell'evento deve essere compresa tra esse. Con **search conference by date in February 2019** come input, il risultato dovrebbe essere di nuovo l'evento **Think 2019**. L'entità **February** viene interpretata come due date, 1° febbraio e 28 febbraio, fornendo quindi l'input per l'intervallo di date di inizio e di fine. [Se non viene specificato l'anno 2019, verrà identificato un febbraio futuro](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time). 

Dopo altre ricerche e nuove voci evento, puoi rivisitare la cronologia della chat e migliorare il dialogo futuro. Segui le istruzioni presenti nella [documentazione di {{site.data.keyword.conversationshort}} su come **migliorare la comprensione**](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro).


## Rimuovi le risorse
{:removeresources}

Eseguendo lo script di ripulitura nella directory principale, elimini la tabella degli eventi da {{site.data.keyword.dashdbshort}} e rimuovi le azioni da {{site.data.keyword.openwhisk_short}}. Ciò potrebbe essere utile quando inizi a modificare e a estendere il codice. Lo script di ripulitura non modifica la capacità o lo spazio di lavoro {{site.data.keyword.conversationshort}}.    
```bash
sh cleanup.sh
```
{:codeblock}

Nell'[elenco delle risorse {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), apri la panoramica dei tuoi servizi. Individua l'istanza del servizio {{site.data.keyword.conversationshort}}, quindi eliminala. 

## Espandi l'esercitazione
Vuoi aggiungere qualcosa o modificare questa esercitazione? Ecco alcune idee:
1. Aggiungi capacità di ricerca, ad esempio, a una ricerca con caratteri jolly o a una ricerca sulle durate degli eventi ("give me all events longer than 8 hours").
2. Utilizza {{site.data.keyword.databases-for-postgresql}} invece di {{site.data.keyword.dashdbshort}}. Il [repository GitHub per questa esercitazione Slackbot](https://github.com/IBM-Cloud/slack-chatbot-database-watson) contiene già il codice per supportare {{site.data.keyword.databases-for-postgresql}}.
3. Aggiungi un servizio meteo e richiama i dati sulle previsioni per la data dell'evento e l'ubicazione. 
4. Esporta i dati dell'evento come un file iCalendar **.ics**.
5. Connetti il chatbot a Facebook Messenger aggiungendo un'altra integrazione. 
6. Aggiungi elementi interattivi, ad esempio pulsanti, all'output.      


## Contenuto correlato
{:related}

Ecco i link a informazioni aggiuntive sugli argomenti trattati in questa esercitazione. 

Post dei blog correlati a chatbot:
* [Chatbots: Some tricks with slots in IBM Watson Conversation](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [Lively chatbots: Best Practices](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [Building chatbots: more tips and tricks](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

Documentazione e SDK:
* Repository GitHub con [suggerimenti e indicazioni per la gestione delle variabili in IBM Watson Conversation](https://github.com/IBM-Cloud/watson-conversation-variables)
* [Documentazione di {{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Documentazione: [IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Free Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) per sviluppatori
* Documentazione: [API Description of the ibm_db Node.js driver](https://github.com/ibmdb/node-ibm_db)
* [Documentazione di {{site.data.keyword.cloudantfull}}](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
