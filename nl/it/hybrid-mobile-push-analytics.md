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

# Applicazione mobile ibrida con Push Notifications
{: #hybrid-mobile-push-analytics}

Scopri come è facile creare rapidamente un'applicazione Cordova ibrida con un servizio mobile di alto valore come {{site.data.keyword.mobilepushshort}} su {{site.data.keyword.Bluemix_notm}}.

Apache Cordova è un framework di sviluppo mobile open-source. Ti consente di utilizzare tecnologie web standard, HTML5, CSS3 e JavaScript, per lo sviluppo multipiattaforma. Le applicazioni vengono eseguite all'interno di wrapper destinati a ciascuna piattaforma e fanno affidamento sui bind API conformi agli standard per accedere alle funzionalità di ciascun dispositivo come ad esempio i sensori, i dati, lo stato di rete ecc.

Questa esercitazione ti guida nella creazione di un'applicazione starter mobile Cordova, aggiungendo un servizio mobile, configurando l'SDK client, scaricando il codice di cui è stato eseguito lo scaffolding e quindi migliorando ulteriormente l'applicazione.

## Obiettivi

* Creare un progetto mobile con il servizio {{site.data.keyword.mobilepushshort}}.
* Imparare come ottenere le credenziali FCM e APN.
* Scaricare il codice e completare la configurazione richiesta.
* Configurare, inviare e monitorare {{site.data.keyword.mobilepushshort}}.

 ![](images/solution15/Architecture.png)

## Prodotti

Questa esercitazione utilizza i seguenti prodotti:
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Prima di iniziare
{: #prereqs}

- [CLI](https://cordova.apache.org/docs/en/latest/guide/cli/) Cordova per eseguire i comandi Cordova.
- [Prerequisiti](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html) di Cordova-IOS e [Prerequisiti](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html) di Cordova-Android
- Account Google per accedere alla console Firebase per ottenere l'ID mittente e la chiave API server.
- Account di [sviluppatori Apple](https://developer.apple.com/) per inviare notifiche remote dall'istanza del servizio {{site.data.keyword.mobilepushshort}} su {{site.data.keyword.Bluemix_notm}} (il provider) ad applicazioni e dispositivi iOS:
- Xcode e Android Studio per importare e migliorare ulteriormente il tuo codice.

## Crea il progetto mobile Cordova dal kit starter
{: #get_code}
Il {{site.data.keyword.Bluemix_notm}} Mobile Dashboard ti consente di tracciare rapidamente il tuo sviluppo di applicazioni mobili creando il tuo progetto da un kit starter.
1. Vai a [Mobile Dashboard](https://{DomainName}/developer/mobile/dashboard).
2. Fai clic su **Starter Kits** e fai clic su **Create App**.
    ![](images/solution15/mobile_dashboard.png)
3. Immetti un nome progetto; può essere anche il tuo nome applicazione.
4. Seleziona **Cordova** come tua piattaforma e fai clic su **Create**.

    ![](images/solution15/create_cordova_project.png)
5. Fai clic su **Add Resource** > Mobile > **Push Notifications** e seleziona l'ubicazione in cui desideri eseguire il provisioning del servizio, del gruppo di risorse e del piano prezzi **Lite**.
6. Fai clic su **Create** per eseguire il provisioning del servizio {{site.data.keyword.mobilepushshort}}. Verrà creata una nuova applicazione nella scheda **Apps**.

    **Nota:** il servizio {{site.data.keyword.mobilepushshort}} deve essere aggiunto allo Starter vuoto.

Nel passo successivo, scaricherai il codice di cui è stato eseguito lo scaffolding e completerai la configurazione richiesta.

## Scarica il codice e completa la configurazione richiesta
{: #download_code}

Se non hai ancora scaricato il codice, usa il {{site.data.keyword.Bluemix_notm}} Mobile Dashboard per ottenere il codice facendo clic sul pulsante **Download Code** in Projects > **Your Mobile Project**.

1. In un IDE a tua scelta, vai a `/platforms/android/project.properties` e sostituisci le ultime due righe (library.1 e library.2) con le righe di seguito indicate

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
  Le modifiche di cui sopra sono specifiche per Android
  {: tip}
2. Per avviare l'applicazione su un emulatore Android, esegui questo comando in un terminale o in un prompt di comandi
```
 $ cordova build android
 $ cordova run android
```
 Se vedi un errore, avvia un emulatore e prova ad eseguire il comando di cui sopra.
 {: tip}
3. Per visualizzare in anteprima l'applicazione nel simulatore iOS, esegui questi comandi in un terminale.

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    Puoi trovare il tuo nome progetto nel file `config.xml` eseguendo il comando `cordova info`.
    {: tip}

## Ottieni le credenziali FCM e APNs
{: #obtain_fcm_apns_credentials}

 ### Configura FCM (Firebase Cloud Messaging)

  1. Nella [console Firebase](https://console.firebase.google.com), crea un nuovo progetto. Imposta il nome su **hybridmobileapp**
  2. Passa a Project **Settings**
  3. Nella scheda **General**, aggiungi due applicazioni:
       1. una con il nome pacchetto impostato su: **com.ibm.mobilefirstplatform.clientsdk.android.push**
       2. e una con il nome pacchetto impostato su: **io.cordova.hellocordovastarter**
  4. Trova l'ID mittente e la chiave server (detta anche chiave API in seguito) nella scheda **Cloud Messaging**.
  5. Nel dashboard del servizio {{site.data.keyword.mobilepushshort}}. imposta il valore per l'ID mittente e la chiave API 


Per una procedura dettagliata, fai riferimento a [Ottieni le credenziali FCM](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials).
{: tip}

### Configura Apple {{site.data.keyword.mobilepushshort}} Service (APNs)

  1. Vai al portale [Apple Developer](https://developer.apple.com/) e registra un ID applicazione.
  2. Crea un certificato SSL di APNs di sviluppo e distribuzione.
  3. Crea un profilo di provisioning di sviluppo. 
  4. Configura l'istanza del servizio {{site.data.keyword.mobilepushshort}} in {{site.data.keyword.Bluemix_notm}}.

Per una procedura dettagliata, fai riferimento a [Ottieni le credenziali APNs e configura il servizio {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials).
{: tip}

## Configura, invia e monitora {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. In index.js, nella funzione `onDeviceReady`, sostituisci i valori `{pushAppGuid}` e

   `{pushClientSecret} `con le **credenziali** del servizio di push - *appGuid* e *clientSecret*.

2. Vai al tuo Mobile dashboard > Projects > Cordova Project, fai clic sul servizio {{site.data.keyword.mobilepushshort}} e attieniti alla seguente procedura.

### APNs - configura l'istanza del servizio

Per utilizzare il servizio {{site.data.keyword.mobilepushshort}} per inviare notifiche, carica i certificati .p12 che avevi creato nel passo precedente. Questo certificato contiene la chiave privata e i certificati SSL necessari per creare e pubblicare la tua applicazione.

**Nota:** dopo che il file `.cer` è presente nel tuo accesso alla catena di chiavi, eseguine l'esportazione sul tuo computer per creare un certificato `.p12`.

1. Fai clic su `{{site.data.keyword.mobilepushshort}}` nella sezione Services oppure fai clic sui tre punti verticali accanto al servizio {{site.data.keyword.mobilepushshort}} e seleziona `Open dashboard`.
2. Nel dashboard {{site.data.keyword.mobilepushshort}}, dovresti vedere l'opzione `Configure` in `Manage > Send Notifications`.

Per configurare APNs sulla console `Push Notification services`, completa la procedura.

1. Seleziona `Configure` dal dashboard Push Notification services.
2. Scegli la `Mobile option` per aggiornare le informazioni nel formato APNs Push Credentials.
3. Seleziona `Sandbox (development)` o `Production (distribution)` come appropriato e carica quindi il certificato `p.12` che hai creato.
4. Nel campo Password, immetti la password associata al file di certificato p12 e fai quindi clic su Save.

### FCM - configura l'istanza del servizio

1. Seleziona **Mobile** e quindi aggiorna la scheda delle credenziali di push GCM/FCM con l'ID mittente/numero progetto e la chiave API (chiave server) che hai inizialmente creato sulla console Firebase.
2. Fai clic su **Save**. Il servizio {{site.data.keyword.mobilepushshort}} è ora configurato.

### Invia {{site.data.keyword.mobilepushshort}}

1. Seleziona **Send Notifications** e componi un messaggio scegliendo un'opzione di invio. Le opzioni supportate sono Device by Tag, Device Id, User Id, Android devices, IOS devices, Web Notifications e All Devices.
   **Nota:** quando selezioni l'opzione **All Devices**, tutti i dispositivi che hanno sottoscritto {{site.data.keyword.mobilepushshort}} riceveranno le notifiche.

2. Nel campo **Message**, componi il messaggio. Scegli di configurare le impostazioni facoltative come richiesto.
3. Fai clic su **Send** e verifica che il tuo dispositivo fisico abbia ricevuto la notifica.

### Monitora le notifiche inviate

Puoi monitorare le tue notifiche inviate andando alla sezione **Monitoring**.
Il servizio IBM {{site.data.keyword.mobilepushshort}} ora estende le funzionalità di monitoraggio delle prestazioni di push generando dei grafici dai tuoi dati utente. Puoi utilizzare il programma di utilità per elencare tutte le {{site.data.keyword.mobilepushshort}} inviate o per elencare i dispositivi registrati e analizzare le informazioni su base giornaliera, settimanale o mensile.
 ![](images/solution6/monitoring_messages.png)

## Contenuto correlato
{: #related_content}

- [Notifiche basate sulle tag](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sicurezza in {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

