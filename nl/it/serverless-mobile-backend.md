---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: data-hd-operatingsystem="ios"}
{:android: data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Applicazione mobile con un backend senza server
{: #serverless-mobile-backend}

In questa esercitazione, imparerai come utilizzare {{site.data.keyword.openwhisk}} insieme ai servizi Cognitive e Data per creare un backend senza server per un'applicazione mobile.
{:shortdesc}

Non tutti gli sviluppatori di dispositivi mobili hanno esperienza nel gestire la logica del lato server o un server per iniziare. Preferiscono concentrare i loro sforzi sull'applicazione che stanno creando. Ora cosa succederebbe se potessero riutilizzare le loro capacità di sviluppo esistenti per scrivere il loro backend mobile? 

{{site.data.keyword.openwhisk_short}} è una piattaforma guidata dagli eventi senza server. Come [evidenziato in questo esempio](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp), le azioni che distribuisci possono essere facilmente trasformate in endpoint HTTP come *azioni web* per creare un'API di backend dell'applicazione web. Un'applicazione web che rappresenta un client nell'API REST, è facile fare un ulteriore passo avanti in questo esempio e applicare lo stesso approccio per creare un backend per un'applicazione mobile. E con {{site.data.keyword.openwhisk_short}}, gli sviluppatori di dispositivi mobili possono scrivere le azioni nello stesso linguaggio utilizzato per l'applicazione mobile, Java per Android e Swift per iOS.

Questa esercitazione è configurabile in base alla tua piattaforma di destinazione. Al momento stai esaminando la documentazione per la versione **iOS / Swift** di questa esercitazione. Utilizza la scheda nella parte superiore di questa documentazione per selezionare la versione **Android / Java** di questa esercitazione.
{: swift}

Questa esercitazione è configurabile in base alla tua piattaforma di destinazione. Al momento stai esaminando la documentazione per la versione **Android / Java** di questa esercitazione. Utilizza la scheda nella parte superiore di questa documentazione per selezionare la versione **iOS / Swift** di questa esercitazione.
{: java}

## Obiettivi
{: #objectives}

* Distribuire un backend mobile senza server con {{site.data.keyword.openwhisk_short}}.
* Aggiungere l'autenticazione utente a un'applicazione mobile con {{site.data.keyword.appid_short}}.
* Analizzare il feedback utente con {{site.data.keyword.toneanalyzershort}}.
* Inviare le notifiche con {{site.data.keyword.mobilepushshort}}.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura
{: #architecture}

L'applicazione mostrata in questa esercitazione è un'applicazione di feedback che analizza in modo intelligente il tono del testo di feedback e fa in modo che il cliente ne prenda atto in modo appropriato mediante {{site.data.keyword.mobilepushshort}}.

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. L'utente viene autenticato tramite [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID). {{site.data.keyword.appid_short}} fornisce i token di accesso e di identificazione. 
2. Ulteriori chiamate all'API di backend includono il token di accesso.
3. Il backend viene implementato con [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk). Le azioni senza server, esposte come Azioni web, prevedono che il token venga inviato nelle intestazioni della richiesta e che ne venga verificata la validità (firma e data di scadenza) prima di consentire l'accesso all'API effettiva.
4. Quando l'utente inoltra un feedback, questo viene archiviato in [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
5. Il testo di feedback viene elaborato con [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer).
6. In base al risultato dell'analisi, viene restituita all'utente una notifica tramite [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush).
7. L'utente riceve la notifica sul dispositivo. 

## Prima di iniziare
{: #prereqs}

Questa esercitazione utilizza lo strumento della riga di comando {{site.data.keyword.Bluemix_notm}} per eseguire il provisioning delle risorse e per distribuire il codice. Assicurati di installare lo strumento della riga di comando `ibmcloud`.

* [{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Script per installare la CLI ibmcloud e i plugin richiesti (Cloud Foundry e {{site.data.keyword.openwhisk_short}})

Inoltre, avrai bisogno del software e degli account riportati di seguito:

   1. Java 8
   2. Android Studio 2.3.3
   3. Account sviluppatore Google per configurare Firebase Cloud Messaging
   4. Shell bash, cURL
   {: java}


   1. Xcode
   2. Account sviluppatore Apple per configurare Apple Push Notification Service
   3. Shell bash, cURL
   {: swift}

In questa esercitazione configurerai le notifiche di push per l'applicazione. L'esercitazione presuppone che tu abbia completato l'esercitazione {{site.data.keyword.mobilepushshort}} di base per [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) o [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics) e che tu abbia familiarità con la configurazione di Firebase Cloud Messaging o Apple Push Notification Service.
{:tip}

Affinché gli utenti Windows 10 utilizzino le istruzioni della riga di comando, consigliamo di installare il sottosistema Windows per Linux e Ubuntu come descritto in [questo articolo](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10).
{: tip}
{: java}

## Ottieni il codice dell'applicazione

Il repository contiene sia l'applicazione mobile che il codice delle azioni {{site.data.keyword.openwhisk_short}}. 

1. Controlla il codice proveniente dal repository GitHub

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-android
   ```
   {: pre}
   {: java}

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-ios
   ```
   {: pre}
   {: swift}

2. Esamina la struttura del codice

| File                                     | Descrizione                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions) | Codice per le azioni {{site.data.keyword.openwhisk_short}} del backend mobile senza server |
| [**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android) | Codice per l'applicazione mobile           |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh) | Script helper per installare, disinstallare, aggiornare il trigger, le azioni e le regole {{site.data.keyword.openwhisk_short}} |
{: java}

| File                                     | Descrizione                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions) | Codice per le azioni {{site.data.keyword.openwhisk_short}} del backend mobile senza server |
| [**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp) | Codice per l'applicazione mobile           |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh) | Script helper per installare, disinstallare, aggiornare il trigger, le azioni e le regole {{site.data.keyword.openwhisk_short}} |
{: swift}

## Esegui il provisioning dei servizi per gestire l'autenticazione utente, la persistenza del feedback e l'analisi
{: #provision_services}

In questa sezione, eseguirai il provisioning dei servizi utilizzati dall'applicazione. Puoi scegliere di eseguire il provisioning dei servizi dal catalogo {{site.data.keyword.Bluemix_notm}} oppure utilizzando la riga di comando `ibmcloud`.

Ti consigliamo di creare un nuovo spazio per eseguire il provisioning dei servizi e per distribuire il backend senza server. Ciò ti aiuta a tenere insieme tutte le risorse. 

### Esegui il provisioning dei servizi dal catalogo {{site.data.keyword.Bluemix_notm}}

1. Vai al [catalogo {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/catalog/)
2. Crea un servizio [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) con il piano **Lite**. Imposta il nome su **serverlessfollowup-db**.
3. Crea un servizio [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) con il piano **Standard**. Imposta il nome su **serverlessfollowup-tone**.
4. Crea un servizio [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) con il piano di livello graduale (**Graduated tier**). Imposta il nome su **serverlessfollowup-appid**.
5. Crea un servizio [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) con il piano **Lite**. Imposta il nome su **serverlessfollowup-mobilepush**.

### Esegui il provisioning dei servizi dalla riga di comando

Con la riga di comando, esegui i comandi riportati di seguito per eseguire il provisioning dei servizi e per richiamare le loro credenziali:

   ```sh
   ibmcloud cf create-service cloudantNoSQLDB Lite serverlessfollowup-db
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service tone_analyzer standard serverlessfollowup-tone
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service AppID "Graduated tier" serverlessfollowup-appid
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service imfpush lite serverlessfollowup-mobilepush
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

## Configura {{site.data.keyword.mobilepushshort}}
{: #push_notifications}

Quando un utente inoltra un nuovo feedback, l'applicazione lo analizza e restituisce una notifica all'utente. L'utente può essere spostato su un'altra attività oppure può non avere l'applicazione mobile avviata, quindi l'utilizzo delle notifiche di push è un buon modo per comunicare con l'utente. Il servizio {{site.data.keyword.mobilepushshort}} rende possibile inviare notifiche agli utenti iOS o Android tramite un'API unificata. In questa sezione, configurerai il servizio {{site.data.keyword.mobilepushshort}} per la tua piattaforma di destinazione. 

### Configura FCM (Firebase Cloud Messaging)
{: java}

   1. Nella [console Firebase](https://console.firebase.google.com), crea un nuovo progetto. Imposta il nome su **serverlessfollowup**
   2. Passa a Project **Settings**
   3. Nella scheda **General**, aggiungi due applicazioni:
      1. una con il nome pacchetto impostato su: **com.ibm.mobilefirstplatform.clientsdk.android.push**
      2. e una con il nome pacchetto impostato su: **serverlessfollowup.app**
   4. Scarica il file `google-services.json` contenente le due applicazioni definite dalla console Firebase e inserisci questo file nella cartella `android/app` della directory di checkout.
   5. Trova l'ID mittente e la chiave server (detta anche chiave API in seguito) nella scheda **Cloud Messaging**.
   6. Nel dashboard del servizio Push Notifications, imposta il valore per l'ID mittente e la chiave API.
   {: java}

### Configura APNs (Apple Push Notifications Service)
{: swift}

1. Vai al portale [Apple Developer](https://developer.apple.com/) e registra un ID applicazione. 
2. Crea un certificato SSL di APNs di sviluppo e distribuzione. 
3. Crea un profilo di provisioning di sviluppo. 
4. Configura l'istanza del servizio {{site.data.keyword.mobilepushshort}} in {{site.data.keyword.Bluemix_notm}}. Fai riferimento a [Ottieni le credenziali APNs e configura il servizio {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-) per i passi dettagliati.
{: swift}

## Distribuisci un backend senza server
{: #serverless_backend}

Con tutti i servizi configurati, ora puoi distribuire il backend senza server. In questa sezione verranno create le seguenti risorse {{site.data.keyword.openwhisk_short}}:

| Risorsa                       | Tipo                           | Descrizione                              |
| ----------------------------- | ------------------------------ | ---------------------------------------- |
| `serverlessfollowup`          | Pacchetto                      | Un pacchetto per raggruppare le azioni e per conservare tutte le credenziali del servizio |
| `serverlessfollowup-cloudant` | Bind di pacchetto              | Collegato al pacchetto {{site.data.keyword.cloudant_short_notm}} integrato   |
| `serverlessfollowup-push`     | Bind di pacchetto              | Collegato al pacchetto {{site.data.keyword.mobilepushshort}}  |
| `auth-validate`               | Azione                         | Convalida i token di accesso e di identificazione |
| `users-add`                   | Azione                         | Archivia in modo persistente le informazioni utente (id, nome, email, immagine, id dispositivo) |
| `users-prepare-notify`        | Azione                         | Formatta un messaggio da utilizzare con {{site.data.keyword.mobilepushshort}} |
| `feedback-put`                | Azione                         | Archivia un feedback utente nel database   |
| `feedback-analyze`            | Azione                         | Analizza un feedback con {{site.data.keyword.toneanalyzershort}}   |
| `users-add-sequence`          | Sequenza esposta come azione web | `auth-validate` e `users-add`          |
| `feedback-put-sequence`       | Sequenza esposta come azione web | `auth-validate` e `feedback-put`       |
| `feedback-analyze-sequence`   | Sequenza                       | `read-document` da {{site.data.keyword.cloudant_short_notm}}, `feedback-analyze`, `users-prepare-notify` e `sendMessage` con {{site.data.keyword.mobilepushshort}} |
| `feedback-analyze-trigger`    | Trigger                        | Richiamata da {{site.data.keyword.openwhisk_short}} quando un feedback viene archiviato nel database |
| `feedback-analyze-rule`       | Regola                         | Collega il trigger `feedback-analyze-trigger` alla sequenza `feedback-analyze-sequence` |

### Compila il codice
{: java}
1. Dalla root della directory di checkout, compila il codice azioni
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### Configura e distribuisci le azioni
{: java}

2. Copia template.local.env in local.env

   ```sh
   cp template.local.env local.env
   ```
3. Ottieni le credenziali per i servizi {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}}
 e {{site.data.keyword.appid_short}} dal dashboard {{site.data.keyword.Bluemix_notm}} (o dall'output dei comandi ibmcloud che abbiamo eseguito in precedenza) e sostituisci i segnaposto in `local.env` con i valori corrispondenti. Queste proprietà verranno inserite in un pacchetto in modo che tutte le azioni possano ottenere l'accesso al database. 
4. Distribuisci le azioni in {{site.data.keyword.openwhisk_short}}. `deploy.sh` carica le credenziali da `local.env` per creare i database {{site.data.keyword.cloudant_short_notm}} (users, feedback e moods) e distribuire le risorse {{site.data.keyword.openwhisk_short}} per l'applicazione. 
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   Puoi utilizzare `./deploy.sh --uninstall` per rimuovere le risorse {{site.data.keyword.openwhisk_short}} una volta che hai completato l'esercitazione.
   {: tip}

### Configura e distribuisci le azioni
{: swift}

1. Copia template.local.env in local.env
   ```sh
   cp template.local.env local.env
   ```
2. Ottieni le credenziali per i servizi {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}}
 e {{site.data.keyword.appid_short}} dal dashboard {{site.data.keyword.Bluemix_notm}} (o dall'output dei comandi ibmcloud che abbiamo eseguito in precedenza) e sostituisci i segnaposto in `local.env` con i valori corrispondenti. Queste proprietà verranno inserite in un pacchetto in modo che tutte le azioni possano ottenere l'accesso al database. 
3. Distribuisci le azioni in {{site.data.keyword.openwhisk_short}}. `deploy.sh` carica le credenziali da `local.env` per creare i database {{site.data.keyword.cloudant_short_notm}} (users, feedback e moods) e distribuire le risorse {{site.data.keyword.openwhisk_short}} per l'applicazione. 

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   Puoi utilizzare `./deploy.sh --uninstall` per rimuovere le risorse {{site.data.keyword.openwhisk_short}} una volta che hai completato l'esercitazione.
   {: tip}

## Configura ed esegui un'applicazione mobile nativa per raccogliere il feedback utente
{: #mobile_app}

Le nostre azioni {{site.data.keyword.openwhisk_short}} sono pronte per la nostra applicazione mobile. Prima di eseguire l'applicazione mobile, devi configurare le sue impostazioni per specificare i servizi che hai creato. 

1. Con Android Studio, apri il progetto che si trova nella cartella `android` della tua directory di checkout.
2. Modifica `android/app/src/main/res/values/credentials.xml` e riempi gli spazi vuoti con i valori derivanti dalle credenziali. Avrai bisogno del `tenantId` di {{site.data.keyword.appid_short}}, dell'`appGuid` e del `clientSecret` di {{site.data.keyword.mobilepushshort}} e dei nomi organizzazione e spazio in cui è stato distribuito {{site.data.keyword.openwhisk_short}}. Per l'host API, avvia il terminale o il prompt dei comandi ed esegui il comando
   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
3. Apri `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` e `LoginAndRegistrationListener.java` e aggiorna la regione del servizio Push Notifications (BMSClient) e la regione AppID a seconda dell'ubicazione in cui sono state create le tue istanze del servizio.
4. Crea il progetto. 
5. Avvia l'applicazione su un dispositivo reale o su un emulatore.
   Affinché l'emulatore riceva le notifiche di push, assicurati di scegliere un'immagine con le API di Google e di accedere con un account Google all'interno dell'emulatore.
   {: tip}
6. Guarda {{site.data.keyword.openwhisk_short}} in background
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. Nell'applicazione, seleziona **Log in** per eseguire l'autenticazione con un account Facebook o Google. Una volta collegato, immetti un messaggio di feedback e premi il pulsante **Send Feedback**. Pochi secondi dopo aver inviato il feedback, dovresti ricevere una notifica di push sul dispositivo. Il testo della notifica può essere personalizzato modificando i documenti di template nel database `moods` nell'istanza del servizio {{site.data.keyword.cloudant_short_notm}}. Utilizza il pulsante **View token** per controllare i token di accesso e di identificazione generati da {{site.data.keyword.appid_short}} all'accesso.
{: java}


1. L'SDK client di push e altri SDK sono disponibili in CocoaPods e Carthage. Per questa soluzione, utilizziamo CocoaPods.
2. Apri il terminale e digita `cd ` alla cartella `followupapp`. Esegui il comando riportato di seguito per installare le dipendenze richieste. 
   ```sh
   pod install
   ```
   {: pre}
3. Apri il file con estensione  `.xcworkspace` presente nella cartella `followupapp` della tua directory di checkout per avviare il tuo codice in Xcode.
4. Modifica il file `BMSCredentials.plist` e riempi gli spazi vuoti con i valori derivanti dalle credenziali. Avrai bisogno del `tenantId` di {{site.data.keyword.appid_short}}, dell'`appGuid` e del `clientSecret` di {{site.data.keyword.mobilepushshort}} e dei nomi organizzazione e spazio in cui è stato distribuito {{site.data.keyword.openwhisk_short}}. Per l'host API, avvia il terminale o il prompt dei comandi ed esegui il comando

   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
5. Apri `AppDelegate.swift` e aggiorna la regione del servizio Push Notifications (BMSClient) e la regione AppID a seconda dell'ubicazione in cui sono state create le tue istanze del servizio. 
6. Crea il progetto. 
7. Avvia l'applicazione su un dispositivo reale o su un simulatore.
8. Guarda {{site.data.keyword.openwhisk_short}} in background eseguendo il comando riportato di seguito sul terminale. 
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. Nell'applicazione, seleziona **Log in** per eseguire l'autenticazione con un account Facebook o Google. Una volta collegato, immetti un messaggio di feedback e premi il pulsante **Send Feedback**. Pochi secondi dopo aver inviato il feedback, dovresti ricevere una notifica di push sul dispositivo. Il testo della notifica può essere personalizzato modificando i documenti di template nel database `moods` nell'istanza del servizio {{site.data.keyword.cloudant_short_notm}}. Utilizza il pulsante **View token** per controllare i token di accesso e di identificazione generati da {{site.data.keyword.appid_short}} all'accesso.
{: swift}

## Rimuovi le risorse

1. Utilizza `deploy.sh` per rimuovere le risorse {{site.data.keyword.openwhisk_short}}:

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. Elimina i servizi {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.appid_short}}, {{site.data.keyword.mobilepushshort}} e {{site.data.keyword.toneanalyzershort}} dalla console {{site.data.keyword.Bluemix_notm}}.

## Contenuto correlato

* {{site.data.keyword.appid_short}} fornisce una configurazione predefinita per aiutarti con la configurazione iniziale dei tuoi provider di identità. Prima di pubblicare la tua applicazione, [aggiorna la configurazione con le tue credenziali](https://{DomainName}/docs/services/appid?topic=appid-social#social). Sarai anche in grado di [personalizzare il widget di accesso](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget).


* Quando crei un'azione OpenWhisk Swift con un file di origine Swift (file .swift nella cartella `actions`), deve essere compilato in un file binario prima che venga eseguita l'azione. Una volta terminato, le chiamate successive all'azione sono molto più veloci finché il contenitore che include l'azione non viene eliminato. Questo ritardo è noto come ritardo di avvio a freddo.
  Per evitare il ritardo di avvio a freddo, puoi compilare il file Swift in un file binario e quindi caricarlo in OpenWhisk in un file zip. Poiché hai bisogno dello scaffolding OpenWhisk, il modo più semplice per creare il file binario è quello di costruirlo all'interno dello stesso ambiente in cui viene eseguito. Fai riferimento a [Assemblaggio di un'azione come eseguibile Swift](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions) per ulteriori passi.
{: swift}
