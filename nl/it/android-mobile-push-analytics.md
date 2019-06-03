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

# Applicazione mobile nativa Android con Push Notifications
{: #android-mobile-push-analytics}

Scopri come è facile creare rapidamente un'applicazione Android nativa con un servizio mobile di alto valore come {{site.data.keyword.mobilepushshort}} su {{site.data.keyword.Bluemix_notm}}.

Questa esercitazione ti guida nella creazione di un'applicazione starter mobile, aggiungendo un servizio mobile, configurando l'SDK client, importando il codice su Android Studio e quindi migliorando ulteriormente l'applicazione.

## Obiettivi
{: #objectives}

* Creare un'applicazione mobile con il servizio {{site.data.keyword.mobilepushshort}}.
* Ottenere le credenziali FCM.
* Scaricare il codice e completare la configurazione richiesta.
* Configurare, inviare e monitorare {{site.data.keyword.mobilepushshort}}.

![](images/solution9/Architecture.png)

## Prodotti
{: #products}

Questa esercitazione utilizza i seguenti prodotti:
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Prima di iniziare
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html) per l'importazione e il miglioramento del tuo codice.
- Account Google per accedere alla console Firebase per ottenere l'ID mittente e la chiave API server.

## Crea l'applicazione mobile Android dal kit starter
{: #get_code}
Il dashboard di {{site.data.keyword.Bluemix_notm}} Mobile ti consente di accelerare lo sviluppo della tua applicazione mobile creandola da un kit starter.
1. Passa al [dashboard Mobile](https://{DomainName}/developer/mobile/dashboard)
2. Fai clic su **Starter Kits** e quindi su **Create App**.
    ![](images/solution9/mobile_dashboard.png)
3. Immetti un nome applicazione; questo può essere anche il nome del tuo progetto Android.
4. Seleziona **Android** come piattaforma e fai clic su **Create**.

    ![](images/solution9/create_mobile_project.png)
5. Fai clic su **Add Resource** > Mobile > **Push Notifications** e seleziona l'ubicazione in cui desideri eseguire il provisioning del servizio, del gruppo di risorse e del piano prezzi **Lite**.
6. Fai clic su **Create** per eseguire il provisioning del servizio {{site.data.keyword.mobilepushshort}}. Verrà creata una nuova applicazione nella scheda **Apps**.

    **Nota:** il servizio {{site.data.keyword.mobilepushshort}} deve essere aggiunto con lo starter vuoto.
    Nel passo successivo, otterrai le credenziali FCM (Firebase Cloud Messaging).

Nel passo successivo, scaricherai il codice di scaffolding e configurerai l'SDK Push Android.

## Scarica il codice e completa la configurazione richiesta
{: #download_code}

Se non hai ancora scaricato il codice, utilizza il dashboard di {{site.data.keyword.Bluemix_notm}} Mobile per ottenere il codice facendo clic sul pulsante **Download Code** in Apps > **Your Mobile App**.
Il codice scaricato viene fornito con incluso l'SDK client di **{{site.data.keyword.mobilepushshort}}**. L'SDK client è disponibile su Gradle e Maven. Per questa esercitazione, utilizzerai **Gradle**.

1. Avvia Android Studio > **Open an existing Android Studio project** e punta al codice scaricato.
2. La build di **Gradle** verrà attivata automaticamente e verranno scaricate tutte le dipendenze.
3. Aggiungi la dipendenza **Google Play services** alla fine del tuo file `build.gradle (Module: app)` a livello di modulo, dopo `dependencies{.....}`
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. Copia il file `google-services.json` che hai creato e scaricato nella directory root del modulo dell'applicazione Android. Nota che il file `google-service.json` include i nomi dei pacchetti aggiunti.
5. Le autorizzazioni richieste sono tutte all'interno del file `AndroidManifest.xml` e delle dipendenze. Push e Analytics sono inclusi in **build.gradle (Module: app)**.
6. Il servizio e i filtri di intento di **Firebase Cloud Messaging (FCM)** per le notifiche evento `RECEIVE` e `REGISTRATION` sono inclusi in `AndroidManifest.xml`

## Ottieni le credenziali FCM
{: #obtain_fcm_credentials}

FCM (Firebase Cloud Messaging) è il gateway utilizzato per fornire {{site.data.keyword.mobilepushshort}} ai dispositivi Android, al browser Google Chrome e alle applicazioni ed estensioni di Chrome. Per configurare il servizio {{site.data.keyword.mobilepushshort}} sulla console, devi ottenere le tue credenziali FCM (ID mittente e chiave API).

La chiave API viene memorizzata in modo sicuro e utilizzata dal servizio {{site.data.keyword.mobilepushshort}} per connettersi al server FCM e l'ID mittente (numero progetto) viene utilizzato dall'SDK Android e dall'SDK JS per Google Chrome e Mozilla Firefox sul lato client. Per configurare FCM e ottenere le tue credenziali, completa la seguente procedura:

1. Visita il sito della [console Firebase](https://console.firebase.google.com/?pli=1)  -  È richiesto un account utente di Google.
2. Seleziona **Add project**.
3. Nella finestra **Create a project**, fornisci un nome progetto, scegli una regione/paese e fai clic su **Create project**.
4. Nel riquadro di navigazione a sinistra, seleziona **Settings** (fai clic sull'icona Settings accanto a **Overview**)> **Project settings**.
5. Scegli la scheda Cloud Messaging per ottenere le tue credenziali del progetto - la chiave API server e un ID mittente.
    **Nota:** la chiave del server elencata in FCM è uguale alla chiave API server.
    ![](images/solution9/fcm_console.png)

Potresti anche aver bisogno di generare il file `google-services.json`. Completa la seguente procedura:

1. Nella console Firebase, fai clic sull'icona **Project Settings** > scheda **General** sotto il progetto che hai creato in precedenza e seleziona **Add Firebase to your Android App**

    ![](images/solution9/firebase_project_settings.png)
2. Nella finestra modale **Add Firebase to your Android app**, aggiungi **com.ibm.mobilefirstplatform.clientsdk.android.push** come nome pacchetto per registrare l'SDK Android {{site.data.keyword.mobilepushshort}}. I campi App nickname e SHA-1 sono facoltativi. Fai clic su **REGISTER APP** > **Continue** > **Finish**.

    ![](images/solution9/add_firebase_to_your_app.png)

3. Fai clic su **ADD APP** > **Add Firebase to your app**.  Includi il nome del pacchetto della tua applicazione immettendo il nome pacchetto **com.ibm.mysampleapp**, quindi procedi alla finestra di aggiunta di Firebase alla tua applicazione Android. I campi App nickname e SHA-1 sono facoltativi. Fai clic su **REGISTER APP** > Continue > Finish.
     **Nota:** puoi trovare il nome del pacchetto della tua applicazione nel file `AndroidManifest.xml` una volta scaricato il codice.
4. Scarica il file di configurazione più recente `google-services.json` in **Your apps**.

    ![](images/solution9/google_services.png)

    **Nota**: FCM è la nuova versione di GCM (Google Cloud Messaging). Assicurati di utilizzare le credenziali FCM per le nuove applicazioni. Le applicazioni esistenti continueranno a funzionare con le configurazioni GCM.

*I passi e l'interfaccia utente della console Firebase sono soggetti a modifiche; se necessario, consulta la documentazione di Google per la parte relativa a Firebase*

## Configura, invia e monitora {{site.data.keyword.mobilepushshort}}

{: #configure_push}

1. L'SDK {{site.data.keyword.mobilepushshort}} è già importato nell'applicazione e il codice di inizializzazione push è disponibile nel file `MainActivity.java`.

    **Nota:** le credenziali del servizio fanno parte del file `/res/values/credentials.xml`.
2. La registrazione per le notifiche avviene in `MainActivity.java`.  (Facoltativo) Fornisci un USER_ID univoco.
3. Esegui l'applicazione su un dispositivo fisico o su un emulatore per ricevere le notifiche.
4. Apri il servizio {{site.data.keyword.mobilepushshort}} in **Mobile Services** > **Existing services** sul dashboard di {{site.data.keyword.Bluemix_notm}} Mobile e, per inviare {{site.data.keyword.mobilepushshort}} di base, completa la seguente procedura:
   - Fai clic su **Manage** > **Configure**.
   - Seleziona **Mobile** e quindi aggiorna la scheda delle credenziali di push GCM/FCM con l'ID mittente/numero progetto e la chiave API (chiave server) che hai inizialmente creato sulla console Firebase.

     ![](images/solution9/configure_push_notifications.png)
   - Fai clic su **Save**. Il servizio {{site.data.keyword.mobilepushshort}} è ora configurato.
   - Seleziona **Send Notifications** e componi un messaggio scegliendo un'opzione di invio. Le opzioni supportate sono Device by Tag, Device Id, User Id, Android devices, IOS devices, Web Notifications e All Devices.
     **Nota:** quando selezioni l'opzione **All Devices**, tutti i dispositivi che hanno sottoscritto {{site.data.keyword.mobilepushshort}} riceveranno le notifiche.
   - Nel campo **Message**, componi il messaggio. Scegli di configurare le impostazioni facoltative come richiesto.
   - Fai clic su **Send** e verifica che il tuo dispositivo fisico abbia ricevuto la notifica.

     ![](images/solution9/android_send_notifications.png)
5. Dovresti visualizzare una notifica sul tuo dispositivo Android.

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. Puoi monitorare le tue notifiche inviate passando alla sezione **Monitoring** sul servizio {{site.data.keyword.mobilepushshort}}.
     Il servizio IBM {{site.data.keyword.mobilepushshort}} ora estende le funzionalità di monitoraggio delle prestazioni di push generando dei grafici dai tuoi dati utente. Puoi utilizzare il programma di utilità per elencare tutte le {{site.data.keyword.mobilepushshort}} inviate o per elencare i dispositivi registrati e analizzare le informazioni su base giornaliera, settimanale o mensile.
      ![](images/solution6/monitoring_messages.png)

## Contenuto correlato
{: #related_content}
- [Personalizza le impostazioni di {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [Notifiche basate sulle tag](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sicurezza in {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

