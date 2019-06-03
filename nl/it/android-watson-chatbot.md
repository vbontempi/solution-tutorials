---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# Crea un chatbot Android abilitato alla funzione vocale
{: #android-watson-chatbot}

Scopri come è facile creare rapidamente un chatbot nativo Android abilitato alla funzione vocale con i servizi {{site.data.keyword.conversationshort}}, {{site.data.keyword.texttospeechshort}} e {{site.data.keyword.speechtotextshort}} su {{site.data.keyword.Bluemix_short}}.

Questa esercitazione ti guida nel processo di definizione di intenti ed entità e di creazione di un flusso di dialogo per il tuo chatbot per rispondere alle domande dei clienti. Imparerai come abilitare i servizi {{site.data.keyword.speechtotextshort}} e {{site.data.keyword.texttospeechshort}} per una facile interazione con l'applicazione Android.
{:shortdesc}

## Obiettivi
{: #objectives}

- Utilizzare {{site.data.keyword.conversationshort}} per personalizzare e distribuire un chatbot.
- Consentire agli utenti finali di interagire con il chatbot utilizzando voce e audio.
- Configurare ed eseguire l'applicazione Android.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti prodotti:

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## Architettura
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* Gli utenti interagiscono con un'applicazione mobile utilizzando la loro voce.
* L'audio viene trascritto in testo con {{site.data.keyword.speechtotextfull}}.
* Il testo viene passato a {{site.data.keyword.conversationfull}}.
* La risposta di {{site.data.keyword.conversationfull}} viene convertita in audio da {{site.data.keyword.texttospeechfull}} e il risultato viene rinviato all'applicazione mobile.

## Prima di iniziare
{: #prereqs}

- Scarica e installa [Android Studio](https://developer.android.com/studio/index.html).

## Crea i servizi
{: #setup}

In questa sezione, creerai i servizi richiesti dall'esercitazione iniziando con {{site.data.keyword.conversationshort}} per creare assistenti virtuali cognitivi che aiutano i tuoi clienti.

1. Vai al [catalogo **{{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e seleziona [{{site.data.keyword.conversationshort}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation) service > **Lite** plan:
   1. Imposta **Name** su **android-chatbot-assistant**
   1. Seleziona **Create**.
2. Fai clic su **Service credentials** nel riquadro di sinistra e seleziona **New credential**.
   1. Imposta **Name** su **for-android-app**.
   1. **Add**.
3. Fai clic su **View Credentials** per visualizzare le credenziali. Prendi nota dei valori **API Key** e **URL**, che ti serviranno per l'applicazione mobile.

Il servizio {{site.data.keyword.speechtotextshort}} converte la voce umana nella parola scritta che può essere inviata come input al servizio {{site.data.keyword.conversationshort}} su {{site.data.keyword.Bluemix_short}}.

1. Vai al [catalogo **{{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e seleziona [{{site.data.keyword.speechtotextshort}}](https://{DomainName}/catalog/services/speech-to-text) service > **Lite** plan.
   1. Imposta **Name** su **android-chatbot-stt**.
   1. Seleziona **Create**.
2. Fai clic su **Service credentials** nel riquadro di sinistra e seleziona **New credential** per aggiungere una nuova credenziale.
   1. Imposta **Name** su **for-android-app**.
   1. **Add**.
3. Fai clic su **View Credentials** per visualizzare le credenziali. Prendi nota dei valori **API Key** e **URL**, che ti serviranno per l'applicazione mobile.

Il servizio {{site.data.keyword.texttospeechshort}} elabora il testo e il linguaggio naturale per generare un output audio sintetizzato completo di cadenza e intonazione appropriate. Il servizio fornisce diverse voci e può essere configurato nell'applicazione Android.

1. Vai al [catalogo **{{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e seleziona [{{site.data.keyword.texttospeechshort}}](https://{DomainName}/catalog/services/text-to-speech) service > **Lite** plan.
   1. Imposta **Name** su **android-chatbot-tts**.
   1. Seleziona **Create**.
2. Fai clic su **Service credentials** nel riquadro di sinistra e seleziona **New credential** per aggiungere una nuova credenziale.
   1. Imposta **Name** su **for-android-app**.
   1. **Add**.
3. Fai clic su **View Credentials** per visualizzare le credenziali. Prendi nota dei valori **API Key** e **URL**, che ti serviranno per l'applicazione mobile.

## Crea una capacità
{: #create_workspace}

Una capacità è un contenitore per le risorse che definiscono il flusso di conversazione.

Per questa esercitazione, salverai e utilizzerai il file [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) con intenti, entità e flusso di dialogo predefiniti sulla tua macchina.

1. Nella pagina dei dettagli del servizio {{site.data.keyword.conversationshort}}, passa a **Manage** nel riquadro di sinistra e fai clic su **Launch tool** per visualizzare il dashboard {{site.data.keyword.conversationshort}}.
1. Fai clic sulla scheda **Skills**.
1. Fai clic su **Create new**, quindi su **Import skill** e scegli il file JSON scaricato in precedenza.
1. Seleziona l'opzione **Everything** e fai clic su **Import**. Viene creata una nuova capacità con intenti, entità e flusso di dialogo predefiniti.
1. Torna all'elenco Skills. Seleziona il menu di azioni sulla capacità `Ana` e fai clic su **View API Details**.

### Definisci un intento
{:#define_intent}

Un intento rappresenta lo scopo dell'input di un utente, come rispondere a una domanda o elaborare il pagamento di una fattura. Puoi definire un intento per ogni tipo di richiesta utente che desideri sia supportata dalla tua applicazione. Riconoscendo l'intenzione espressa nell'input di un utente, il servizio {{site.data.keyword.conversationshort}} può scegliere il flusso di dialogo corretto per rispondere. Nello strumento, il nome di un intento ha sempre come prefisso il carattere `#`.

In poche parole, gli intenti sono le intenzioni dell'utente finale. Di seguito sono riportati degli esempi di nomi intento.
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. Fai clic sulla capacità appena creata - **Ana**.

   Ana è un bot assicurativo che consente agli utenti di eseguire query sulle loro agevolazioni sanitarie e richieste di file.
   {:tip}
2. Fai clic sulla prima scheda per visualizzare tutti gli intenti (**Intents**).
3. Fai clic su **Add intent** per creare un nuovo intento. Immetti `Cancel_Policy` come tuo nome intento dopo `#` e fornisci una descrizione facoltativa.
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. Fai clic su **Create intent**.
5. Aggiungi degli esempi utente quando richiesto per annullare una politica
   - `I want to cancel my policy`
   - `Drop my policy now`
   - `I wish to stop making payments on my policy.`
6. Aggiungi gli esempi utente uno dopo l'altro e fai clic su **add example**. Ripeti questo passo per tutti gli altri esempi utente.

   Ricorda di aggiungere almeno 5 esempi utente per addestrare al meglio il tuo bot.
   {:tip}

7. Fai clic sul pulsate **close** ![](images/solution28-watson-chatbot-android/close_icon.png) accanto al nome intento per salvare l'intento.
8. Fai clic su **Content Catalog** e seleziona **General**. Fai clic su **Add to skill**.

   Il catalogo di contenuto ti aiuta a iniziare più velocemente aggiungendo degli intenti esistenti (servizi bancari, servizio clienti, assicurazioni, telco, e-commerce e molti altri). Questi intenti vengono addestrati su domande comuni che gli utenti potrebbero chiedere.
   {:tip}

### Definisci un'entità
{:#define_entity}

Un'entità rappresenta un termine o un oggetto che è rilevante per i tuoi intenti e che fornisce un contesto specifico per un intento. Elenchi i possibili valori per ciascuna entità e i sinonimi che gli utenti potrebbero immettere. Riconoscendo le entità che sono indicate nell'input dell'utente, il servizio {{site.data.keyword.conversationshort}} può scegliere le azioni specifiche da intraprendere per soddisfare un intento. Nello strumento, il nome di un'entità ha sempre come prefisso il carattere `@`.

Di seguito sono riportati degli esempi di nomi di entità
 - `@location`
 - `@menu_item`
 - `@product`

1. Fai clic sulla scheda **Entities** per visualizzare le entità esistenti.
2. Fai clic su **Add entity** e immetti il nome dell'entità come `location` dopo `@`. Fai clic su **Create entity**.
3. Immetti `address` come nome valore e seleziona **Synonyms**.
4. Aggiungi `place` come sinonimo e fai clic sull'icona ![](images/solution28-watson-chatbot-android/plus_icon.png). Ripeti con i sinonimi `office`, `centre`, `branch` ecc. e fai clic su **Add Value**.
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. Fai clic su **close** ![](images/solution28-watson-chatbot-android/close_icon.png) per salvare le modifiche.
6. Fai clic sulla scheda **System entities** per controllare le entità comuni create da IBM che potrebbero essere utilizzate in qualsiasi caso di utilizzo.

   Le entità di sistema possono essere utilizzate per riconoscere un'ampia gamma di valori per i tipi di oggetti che rappresentano. Ad esempio, l'entità di sistema `@sys-number` corrisponde a qualsiasi valore numerico, compresi numeri interi, frazioni decimali o anche numeri scritti come parole.
   {:tip}
7. Cambia lo **Status** da off a `on` per le entità di sistema @sys-person e @sys-location.

### Crea il flusso di dialogo
{:#build_dialog}

Un dialogo è un flusso di conversazione ramificato che definisce come l'applicazione risponde quando riconosce gli intenti e le entità che hai definito. Utilizza il builder di dialoghi nello strumento per creare conversazioni con gli utenti, fornendo risposte basate sugli intenti e sulle entità che riconosci nel loro input. 

1. Fai clic sulla scheda **Dialog** per visualizzare il flusso di dialogo esistente con gli intenti e le entità.
2. Fai clic su **Add node** per aggiungere un nuovo nodo al dialogo.
3. In **if assistant recognizes**, immetti `#Cancel_Policy`.
4. In **Then respond with**, immetti la risposta `This facility is not available online. Please visit our nearest branch to cancel your policy.`
5. Fai clic su ![](images/solution28-watson-chatbot-android/save_node.png) per chiudere e salvare il nodo.
6. Scorri per visualizzare il nodo `#greeting`. Fai clic sul nodo per visualizzare i dettagli.
   ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. Fai clic sull'icona ![](images/solution28-watson-chatbot-android/add_condition.png) per **aggiungere una nuova condizione**. Seleziona `or` dal menu a discesa e immetti `#General_Greetings` come intento. **La sezione Then respond with** mostra la risposta dell'assistente quando viene salutato dall'utente.
   ![](images/solution28-watson-chatbot-android/apply_condition.png)

   Una variabile di contesto è una variabile che definisci in un nodo e per la quale puoi facoltativamente specificare un valore predefinito. Altri nodi o la logica di applicazione possono successivamente impostare o modificare il valore della variabile di contesto. L'applicazione può passare le informazioni al dialogo e il dialogo può aggiornare queste informazioni e ritrasmetterle all'applicazione o a un nodo successivo. Il dialogo esegue tali operazioni utilizzando le variabili di contesto.
   {:tip}

8. Verifica il flusso di dialogo utilizzando il pulsante **Try it**.

## Collega la capacità a un assistente

Un **assistente** è un bot cognitivo che puoi personalizzare per le tue esigenze aziendali e distribuire attraverso più canali per fornire supporto ai tuoi clienti dove e quando ne hanno bisogno. Personalizzi l'assistente aggiungendo ad esso le **capacità** di cui ha bisogno per soddisfare gli obiettivi dei tuoi clienti.

1. Nello strumento {{site.data.keyword.conversationshort}}, passa a **Assistants** e utilizza **Create new**.
   1. Imposta **Name** su **android-chatbot-assistant**
   1. Seleziona **Create**
1. Utilizza **Add Dialog skill** per selezionare la capacità creata nelle sezioni precedenti.
   1. Fai clic su **Add existing skill**
   1. Seleziona **Ana**
1. Seleziona il menu di azioni su Assistant > **Settings** > **API Details**, quindi prendi nota dell'**Assistant ID** a cui dovrai fare riferimento dall'applicazione mobile (nel file `config.xml` dell'applicazione Android).

## Configura ed esegui l'applicazione Android
{:#configure_run_android_app}

Il repository contiene il codice applicativo Android con le dipendenze Gradle richieste.

1. Immetti il seguente comando per clonare il [repository GitHub](https://github.com/IBM-Cloud/chatbot-watson-android):
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. Avvia Android Studio > **Open an existing Android Studio project** e punta al codice scaricato. La build di **Gradle** verrà attivata automaticamente e verranno scaricate tutte le dipendenze.
3. Apri `app/src/main/res/values/config.xml` per visualizzare i segnaposto (`ASSISTANT_ID_HERE`) per le credenziali del servizio. Immetti le credenziali del servizio (che hai salvato in precedenza) nei rispettivi segnaposto e salva il file.
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <!--Watson Assistant service credentials-->
       <!-- REPLACE `ASSISTANT_ID_HERE` with ID of the Assistant to use -->
       <string name="assistant_id">ASSISTANT_ID_HERE</string>

       <!-- REPLACE `ASSISTANT_API_KEY_HERE` with Watson Assistant service API Key-->
       <string name="assistant_apikey">ASSISTANT_API_KEY_HERE</string>

       <!-- REPLACE `ASSISTANT_URL_HERE` with Watson Assistant service URL-->
       <string name="assistant_url">ASSISTANT_URL_HERE</string>

       <!--Watson Speech To Text(STT) service credentials-->
       <!-- REPLACE `STT_API_KEY_HERE` with Watson Speech to Text service API Key-->
       <string name="STT_apikey">STT_API_KEY_HERE</string>

       <!-- REPLACE `STT_URL_HERE` with Watson Speech to Text service URL-->
       <string name="STT_url">STT_URL_HERE</string>

       <!--Watson Text To Speech(TTS) service credentials-->
       <!-- REPLACE `TTS_API_KEY_HERE` with Watson Text to Speech service API Key-->
       <string name="TTS_apikey">TTS_API_KEY_HERE</string>

       <!-- REPLACE `TTS_URL_HERE` with Watson Text to Speech service URL-->
       <string name="TTS_url">TTS_URL_HERE</string>
   </resources>
   ```
4. Crea il progetto e avvia l'applicazione su un dispositivo reale o con un simulatore.
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. **Immetti la tua query ** nello spazio fornito in basso e fai clic sull'icona della freccia per inviare la query al servizio {{site.data.keyword.conversationshort}}.
6. La risposta verrà passata al servizio {{site.data.keyword.texttospeechshort}} e dovresti sentire una voce che legge la risposta.
7. Fai clic sull'icona del **microfono** nell'angolo inferiore destro dell'applicazione per inserire il discorso che viene convertito in testo e che può quindi essere inviato al servizio {{site.data.keyword.conversationshort}} facendo clic sull'icona della freccia.


## Rimuovi le risorse
{:removeresources}

1. Passa all'[elenco delle risorse](https://{DomainName}/resources/)
1. Elimina i servizi che hai creato:
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## Contenuto correlato
{:related}

- [Creazione di entità, sinonimi ed entità di sistema](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [Variabili di contesto](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [Creazione di un dialogo complesso](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [Raccolta di informazioni con gli slot](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [Opzioni di distribuzione](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [Migliora la tua capacità](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
