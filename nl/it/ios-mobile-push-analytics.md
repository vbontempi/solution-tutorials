---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-08"
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

# Applicazione mobile iOS con Push Notifications
{: #ios-mobile-push-analytics}

Scopri come è facile creare rapidamente un'applicazione iOS Swift con un servizio mobile di alto valore {{site.data.keyword.mobilepushshort}} su {{site.data.keyword.Bluemix_short}}.

Questa esercitazione ti guida nella creazione di un'applicazione starter mobile, aggiungendo un servizio mobile, configurando l'SDK client, scaricando il codice in Xcode e quindi migliorando ulteriormente l'applicazione.
{:shortdesc: .shortdesc}

## Obiettivi
{:#objectives}

- Creare un'applicazione mobile con i servizi {{site.data.keyword.mobilepushshort}} e {{site.data.keyword.mobileanalytics_short}} dal kit starter Swift di base.
- Ottenere le credenziali APNs e configurare l'istanza del servizio {{site.data.keyword.mobilepushshort}}.
- Scaricare il codice e configurare l'SDK client.
- Inviare e monitorare {{site.data.keyword.mobilepushshort}}.

  ![](images/solution6/Architecture.png)

## Prodotti
{:#products}

Questa esercitazione utilizza i seguenti prodotti:
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Prima di iniziare
{: #prereqs}

1. Account di [sviluppatori Apple](https://developer.apple.com/) per inviare notifiche remote dall'istanza del servizio {{site.data.keyword.mobilepushshort}} su {{site.data.keyword.Bluemix_short}} (il provider) ad applicazioni e dispositivi iOS:
2. Xcode per importare e migliorare il tuo codice.

## Crea un'applicazione mobile dal kit starter Swift di base
{: #get_code}

1. Vai a [Mobile Dashboard](https://{DomainName}/developer/mobile/dashboard) per creare la tua applicazione (`App`) dai kit starter (`Starter Kits`) predefiniti.
2. Fai clic su **Starter Kits** e scorri verso il basso per selezionare il kit starter **Basic**.
    ![](images/solution6/mobile_dashboard.png)
3. Immetti un nome dell'applicazione che sarà anche il nome dell'applicazione e del progetto Xcode.
4. Seleziona `iOS Swift` come tua piattaforma e fai clic su **Create**.
    ![](images/solution6/create_mobile_project.png)
5. Fai clic su **Add Resource** > Mobile > **Push Notifications** e seleziona l'ubicazione in cui desideri eseguire il provisioning del servizio, del gruppo di risorse e del piano prezzi **Lite**.
6. Fai clic su **Create** per eseguire il provisioning del servizio {{site.data.keyword.mobilepushshort}}. Verrà creata una nuova applicazione nella scheda **Apps**.

​      **Nota:** {{site.data.keyword.mobilepushshort}} il servizio dovrebbe già essere stato aggiunto con lo Starter vuoto.

## Scarica il codice e configura gli SDK client
{: #download_code}

Se non hai ancora scaricato il codice, fai clic su `Download Code` in Apps > `Your Mobile App`
Il codice scaricato viene fornito con l'SDK client **{{site.data.keyword.mobilepushshort}}** incluso. L'SDK client è disponibile su CocoaPods e Carthage. Per questa soluzione, utilizzerai CocoaPods.

1. Per installare CocoaPods sulla tua macchina, apri il terminale (`Terminal`) ed esegui questo comando.
   ```
   sudo gem install cocoapods
   ```
   {: pre:}
2. Decomprimi il codice scaricato e, utilizzando il terminale, vai alla cartella decompressa.

   ```
   cd <Name of Project>
   ```
   {: pre:}
3. La cartella include già un `podfile` con le dipendenze richieste. Esegui questo comando per installare le dipendenze (SDK client) e le dipendenze richieste verranno installate.

  ```
  pod install
  ```
  {: pre:}

## Ottieni le credenziali APNs e configura l'istanza del servizio {{site.data.keyword.mobilepushshort}}.
{: #obtain_apns_credentials}

   Per i dispositivi e le applicazioni iOS, APNs (Apple Push Notification Service) consente agli sviluppatori di applicazioni di inviare notifiche remote dall'istanza del servizio {{site.data.keyword.mobilepushshort}} su {{site.data.keyword.Bluemix_short}} (il provider) a dispositivi e applicazioni iOS: I messaggi sono inviati a un'applicazione di destinazione sul dispositivo.

   Devi ottenere e configurare le tue credenziali APNs. I certificati APNs sono gestiti in modo sicuro dal servizio {{site.data.keyword.mobilepushshort}} e utilizzati per stabilire una connessione al server APNs come un provider.

### Registrazione di un ID applicazione

   L'ID applicazione (l'identificativo del bundle) è un identificativo univoco che identifica una specifica applicazione. Ogni applicazione richiede un ID applicazione. I servizi come {{site.data.keyword.mobilepushshort}} sono configurati sull'ID applicazione.
   Assicurati di avere un account di [sviluppatore Apple](https://developer.apple.com/). Questo è un prerequisito obbligatorio.

   1. Vai al portale [Apple Developer](https://developer.apple.com/), fai clic su `Member Center` e seleziona `Certificates, IDs & Profiles`.
   2. Vai alla sezione `Identifiers` > App IDs.
   3. Nella pagina `Registering App IDs`, fornisci il nome dell'applicazione nel campo del nome di descrizione dell'ID dell'applicazione (App ID Description Name). Ad esempio: ACME {{site.data.keyword.mobilepushshort}}. Fornisci una stringa per il prefisso dell'ID applicazione.
   4. Per un suffisso dell'ID applicazione, scegli `Explicit App ID` e fornisci un valore ID bundle. Ti consigliamo di fornire una stringa di stile nome dominio inversa. Ad esempio: com.ACME.push.
   5. Seleziona la casella di spunta `{{site.data.keyword.mobilepushshort}}` e fai clic su `Continue`.
   6. Completa le impostazioni e fai clic su `Register` > `Done`. Il tuo ID applicazione è ora registrato.

     ![](images/solution6/push_ios_register_appid.png)

### Crea un certificato SSL di APNs di sviluppo e distribuzione
   Prima di ottenere un certificato APNs, devi generare una CSR (certificate signing request) e inoltrarla ad Apple, l'autorità di certificazione (CA). La CSR contiene le informazioni che identificano la tua azienda e la tua chiave pubblica e privata che utilizzi per sottoscrivere il tuo Apple {{site.data.keyword.mobilepushshort}}. Genera quindi il certificato SSL
nel portale per sviluppatori iOS. Il certificato, insieme alla sua chiave pubblica e a quella privata, viene archiviato in Keychain Access.
   Puoi utilizzare le APNs in due modi:

- Modalità sandbox per lo sviluppo e il test.
- Modalità di produzione quando si distribuiscono le applicazioni tramite l'App Store (o altri meccanismi di distribuzione aziendali).

   Devi ottenere dei certificati separati per i tuoi ambienti di sviluppo e distribuzione. I certificati sono associati a un ID applicazione per l'applicazione destinataria delle notifiche remote. per la produzione, è possibile creare fino a due certificati.{{site.data.keyword.Bluemix_short}} utilizza i certificati per stabilire una connessione SSL con APNs.

   1. Vai al sito web di Apple Developer, fai clic su **Member Center** e seleziona **Certificates, IDs & Profiles**.
   2. Nell'area **Identifiers**, fai clic su **App IDs**.
   3. Dal tuo elenco di ID applicazione, seleziona il tuo ID applicazione e seleziona quindi `Edit`.
   4. Seleziona la casella di spunta **{{site.data.keyword.mobilepushshort}}** e quindi, sul riquadro **Development SSL certificate**, fai clic su **Create Certificate**.

     ![Certificati SSL di Push Notification](images/solution6/certificate_createssl.png)

   5. Quando viene visualizzata la **schermata About Creating a Certificate Signing Request (CSR)**, attieniti alle istruzioni visualizzate per creare un file CSR (Certificate Signing Request). Fornisci un nome significativo che ti aiuta a identificare se si tratta di un certificato per lo sviluppo (sandbox) o la distribuzione (produzione); ad esempio, sandbox-apns-certificate o production-apns-certificate.
   6. Fai clic su **Continue** e, nella schermata Upload CSR file, fai clic su **Choose File** e seleziona il file **CertificateSigningRequest.certSigningRequest** che hai appena creato. Fai clic su **Continue**.
   7. Nel pannello Download, Install and Backup, fai clic su Download. Il file **aps_development.cer** viene scaricato.
        ![Scarica il certificato](images/solution6/push_certificate_download.png)

        ![Genera il certificato](images/solution6/generate_certificate.png)
   8. Sul tuo mac, apri **Keychain Access**, **File**, **Import** e seleziona il file .cer scaricato per installarlo.
   9. Fai clic con il pulsante destro del mouse sul nuovo certificato e sulla chiave privata, seleziona quindi **Export** e modifica il formato del file (**File Format**) in un formato di scambio di informazioni personali (Personal information exchange format) (formato `.p12`).
     ![Esporta certificati e chiavi](images/solution6/keychain_export_key.png)
   10. Nel campo **Save As**, dai al certificato un nome significativo. Degli esempi possono essere `sandbox_apns.p12` o **production_apns.p12**; fai quindi clic su Save.
     ![Esporta certificati e chiavi](images/solution6/certificate_p12v2.png)
   11. Nel campo **Enter a password**, immetti una password per proteggere gli elementi esportati e fai quindi clic su OK. Puoi utilizzare questa password per configurare le tue impostazioni APNs sulla console del servizio {{site.data.keyword.mobilepushshort}}.
       ![Esporta certificato e chiavi](images/solution6/export_p12.png)
   12. **Key Access.app** ti chiede di esportare la tua chiave dalla schermata **Keychain**. Immetti la tua password amministrativa per il tuo Mac per consentire al tuo sistema di esportare questi elementi e seleziona quindi l'opzione `Always Allow`. Sul tuo desktop viene generato un certificato `.p12`.

      Per l'SSL di produzione, nel riquadro **Production SSL certificate**, fai clic su **Create Certificate** e ripeti i passi da 5 a 12 sopra indicati.
      {:tip}

### Creazione di profilo di provisioning di sviluppo
   Il profilo di provisioning utilizza l'ID applicazione per determinare quali sono i dispositivi su cui può essere installata ed eseguita la tua applicazione e quali sono i servizi a cui la tua applicazione può accedere. Per ogni ID applicazione, crei due profili di provisioning: uno per lo sviluppo e l'altro per la distribuzione. Xcode utilizza il profilo di provisioning di sviluppo per determinare quali sono gli sviluppatori a cui è consentito creare l'applicazione e quali sono i dispositivi di cui è consentita l'esecuzione di test sull'applicazione.

   Assicurati di aver registrato un ID applicazione, di averlo abilitato per il servizio {{site.data.keyword.mobilepushshort}} e di averlo configurato per utilizzare un certificato SSL APNs di sviluppo e produzione.

   Crea un profilo di provisioning di sviluppo nel seguente modo:

   1. Vai al portale [Apple Developer](https://developer.apple.com/), fai clic su `Member Center` e seleziona `Certificates, IDs & Profiles`.
   2. Vai alla [Mac Developer Library](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site), scorri la sezione `Creating Development Provisioning Profiles` e attieniti alle istruzioni per creare un profilo di sviluppo.
     **Nota:** quando configuri un profilo di provisioning di sviluppo, seleziona le seguenti opzioni:

     - **iOS App Development**
     - **For iOS and watchOS apps**

### Creazione di un profilo di provisioning di distribuzione di archivio
   Utilizza il profilo di provisioning di archivio per inoltrare la tua applicazione per la distribuzione all'App Store.

   1. Vai a [Apple Developer](https://developer.apple.com/), fai clic su `Member Center` e seleziona `Certificates, IDs & Profiles`.
   2. Fai doppio clic sul file di provisioning scaricato per installarlo in Xcode.
     Dopo aver ottenuto le credenziali FCM, il passo successivo è [Configura un'istanza del servizio](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2).

### Configura l'istanza del servizio

   Per utilizzare il servizio {{site.data.keyword.mobilepushshort}} per inviare notifiche, carica i certificati .p12 che avevi creato nel passo precedente. Questo certificato contiene la chiave privata e i certificati SSL necessari per creare e pubblicare la tua applicazione.

   **Nota**: dopo che il file `.cer` è presente nel tuo accesso alla catena di chiavi, eseguine l'esportazione sul tuo computer per creare un certificato `.p12`.

1. Fai clic su {{site.data.keyword.mobilepushshort}} nella sezione Services oppure fai clic sui tre punti verticali accanto al servizio {{site.data.keyword.mobilepushshort}} e seleziona `Open dashboard`.
2. Nel dashboard {{site.data.keyword.mobilepushshort}}, dovresti vedere l'opzione `Configure` in `Manage`.

Per configurare APNs sulla console `Push Notification services`, completa la procedura.

1. Scegli la `Mobile option` per aggiornare le informazioni nel formato APNs Push Credentials.
2. Seleziona `Sandbox/Development APNs Server` o `Production APNs Server` come appropriato e carica quindi il certificato `.p12` che hai creato.
3. Nel campo Password, immetti la password associata al file di certificato p12 e fai quindi clic su Save.

![](images/solution6/Mobile_push_configure.png)

## Configura, invia e monitora {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. Il codice di inizializzazione del push (sotto `func application`) e il codice di registrazione della notifica possono essere trovati in `AppDelegate.swift`. Fornisci uno USER_ID univoco (facoltativo).
2. Esegui l'applicazione su un dispositivo fisico poiché le notifiche non possono essere inviate a un simulatore iPhone.
3. Apri il servizio {{site.data.keyword.mobilepushshort}} in `Mobile Services` > **Existing services** sul dashboard di {{site.data.keyword.Bluemix_short}} Mobile e, per inviare {{site.data.keyword.mobilepushshort}} di base, completa la seguente procedura:
  * Seleziona `Messages` e componi un messaggio scegliendo un'opzione Send to. Le opzioni supportate sono Device by Tag, Device Id, User Id, Android devices, iOS devices, Web Notifications, All Devices and other browsers.

       **Nota:** quando selezioni l'opzione All Devices, tutti i dispositivi che hanno sottoscritto {{site.data.keyword.mobilepushshort}} riceveranno le notifiche.
  * Nel campo `Message`, componi il messaggio. Scegli di configurare le impostazioni facoltative come richiesto.
  * Fai clic su `Send` e verifica che il tuo dispositivo fisico abbia ricevuto la notifica.
    ![](images/solution6/send_notifications.png)

4. Dovresti vedere una notifica sul tuo iPhone.

   ![](images/solution6/iphone_notification.png)

5. Puoi monitorare le tue notifiche inviate andando a `Monitoring` sul {{site.data.keyword.mobilepushshort}} Service.

Il servizio IBM {{site.data.keyword.mobilepushshort}} ora estende le funzionalità di monitoraggio delle prestazioni di push generando dei grafici dai tuoi dati utente. Puoi utilizzare il programma di utilità per elencare tutte le {{site.data.keyword.mobilepushshort}} inviate o per elencare i dispositivi registrati e analizzare le informazioni su base giornaliera, settimanale o mensile.
![](images/solution6/monitoring_messages.png)

## Contenuto correlato
{: #related_content}

- [Notifiche basate sulle tag](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sicurezza in {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)
