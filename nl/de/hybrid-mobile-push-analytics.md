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

# Mobile Hybrid-Anwendungen mit Push Notifications
{: #hybrid-mobile-push-analytics}

Erfahren Sie, wie einfach es ist, schnell eine Hybrid-Cordova-Anwendung mit einem hochwertigen mobilen Service wie {{site.data.keyword.mobilepushshort}} on {{site.data.keyword.Bluemix_notm}} zu erstellen.

Apache Cordova ist ein mobiles Open-Source-Entwicklungsframework. Es ermöglicht Ihnen die Verwendung von Standard-Webtechnologien, wie HTML5, CSS3 und JavaScript, für eine plattformübergreifende Entwicklung. Anwendungen werden in Wrappern für jede Plattform ausgeführt und basieren auf standardkonformen API-Bindungen, um auf die Funktionen der einzelnen Geräte zuzugreifen, wie z. B. Sensoren, Daten, Netzstatus usw.

Dieses Lernprogramm führt Sie durch die Erstellung einer mobilen Cordova-Startanwendung, das Hinzufügen eines mobilen Service, das Einrichten des Client-SDK, das Herunterladen des Gerüstcodes und das anschließende weitere Verbessern der Anwendung.

## Lernziele

* Mobiles Projekt mit dem {{site.data.keyword.mobilepushshort}}-Service erstellen
* Vorgehensweise zum Abrufen von APNs und FCM-Berechtigungsnachweisen kennenlernen
* Code herunterladen und die erforderliche Konfiguration durchführen
* {{site.data.keyword.mobilepushshort}}-Instanz konfigurieren, senden und überwachen

 ![](images/solution15/Architecture.png)

## Produkte

Dieses Lernprogramm verwendet die folgenden Produkte:
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Vorbereitende Schritte
{: #prereqs}

- Sie benötigen die Cordova-[CLI](https://cordova.apache.org/docs/en/latest/guide/cli/) zum Ausführen von Cordova-Befehlen.
- Sie müssen die [Voraussetzungen](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html) für Cordova-iOS und die [Voraussetzungen](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html) für Cordova-Android erfüllen.
- Sie benötigen ein Google-Konto für die Anmeldung bei der Firebase-Konsole, um die Absender-ID und den Server-API-Schlüssel abzurufen.
- Sie benötigen ein [Apple Developer](https://developer.apple.com/)-Konto zum Senden ferner Benachrichtigungen von der {{site.data.keyword.mobilepushshort}}-Serviceinstanz in {{site.data.keyword.Bluemix_notm}} (dem Provider) an iOS-Geräte und -Anwendungen.
- Sie benötigen Xcode und Android Studio zum Importieren und Erweitern Ihres Codes.

## Mobiles Cordova-Projekt über das Starter-Kit erstellen
{: #get_code}
Mit dem mobilen Dashboard in {{site.data.keyword.Bluemix_notm}} können Sie die Entwicklung mobiler Apps beschleunigen, indem Sie Ihr Projekt über ein Starter-Kit erstellen.
1. Navigieren Sie zum [mobilen Dashboard](https://{DomainName}/developer/mobile/dashboard).
2. Klicken Sie auf **Starter-Kits** und dann auf **App erstellen**.
    ![](images/solution15/mobile_dashboard.png)
3. Geben Sie einen Projektnamen ein. Dies kann auch der App-Name sein.
4. Wählen Sie **Cordova** als Plattform aus und klicken Sie auf **Erstellen**.

    ![](images/solution15/create_cordova_project.png)
5. Klicken Sie auf **Ressource hinzufügen** > Mobil > **Push Notifications** und wählen Sie die Position aus, die Sie für den Service, die Ressourcengruppe und den **Lite**-Preisplan bereitstellen möchten.
6. Klicken Sie auf **Erstellen**, um den {{site.data.keyword.mobilepushshort}}-Service bereitzustellen. Auf der Registerkarte **Apps** wird eine neue App erstellt.

    **Hinweis:**{{site.data.keyword.mobilepushshort}} Der Service sollte bereits mit dem leeren Starter hinzugefügt werden.

Im nächsten Schritt laden Sie den Gerüstcode herunter und führen die erforderliche Konfiguration durch.

## Code herunterladen und die erforderliche Konfiguration durchführen
{: #download_code}

Wenn Sie den Code noch nicht heruntergeladen haben, verwenden Sie das mobile {{site.data.keyword.Bluemix_notm}}-Dashboard, um den Code zu erhalten, indem Sie unter 'Projekte > **Ihr mobiles Projekt** auf die Schaltfläche **Code herunterladen** klicken.

1. Navigieren Sie in einer IDE Ihrer Wahl zu `/platforms/android/project.properties` und ersetzen Sie die letzten beiden Zeilen (library.1 und library.2) durch die Zeilen unten.

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
  Die oben genannten Änderungen gelten speziell für Android.
  {: tip}
2. Um die App auf einem Android-Emulator zu starten, führen Sie den folgenden Befehl in einem Terminal oder einer Eingabeaufforderung aus:
```
 $ cordova build android
 $ cordova run android
```
 Wenn Sie einen Fehler sehen, starten Sie einen Emulator und versuchen Sie, den oben genannten Befehl auszuführen.
 {: tip}
3. Um eine Vorschau der App im iOS-Simulator anzuzeigen, führen Sie die folgenden Befehle in einem Terminal aus:

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    Sie können den Projektnamen durch das Ausführen des Befehls `cordova info` aus der Datei `config.xml` abrufen.
    {: tip}

## FCM- und APNs-Berechtigungsnachweise abrufen
{: #obtain_fcm_apns_credentials}

 ### Firebase Cloud Messaging (FCM) konfigurieren

  1. Erstellen Sie in der [Firebase-Konsole](https://console.firebase.google.com) ein neues Projekt. Setzen Sie den Namen auf **hybridmobileapp**.
  2. Navigieren Sie zu den **Einstellungen** des Projekts.
  3. Fügen Sie auf der Registerkarte **Allgemein** zwei Anwendungen hinzu:
       1. Eine mit dem Paketnamen **com.ibm.mobilefirstplatform.clientsdk.android.push**
       2. und eine mit dem Paketnamen **io.cordova.hellocordovastarter**
  4. Suchen Sie auf der Registerkarte **Cloud Messaging** nach der Absender-ID und dem Serverschlüssel (im späteren Verlauf auch als API-Schlüssel bezeichnet).
  5. Geben Sie im Dashboard des {{site.data.keyword.mobilepushshort}}-Service den Wert für die Absender-ID und den API-Schlüssel an.


Detaillierte Schritte finden Sie unter [FCM-Berechtigungsnachweise abrufen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials).
{: tip}

### Apple {{site.data.keyword.mobilepushshort}}-Service (APNs) konfigurieren

  1. Rufen Sie das [Apple Developer](https://developer.apple.com/)-Portal auf und registrieren Sie eine App-ID.
  2. Erstellen Sie ein APNs-SSL-Zertifikat für die Entwicklung und Verteilung.
  3. Erstellen Sie ein Bereitstellungsprofil für die Entwicklung.
  4. Konfigurieren Sie die {{site.data.keyword.mobilepushshort}}-Serviceinstanz in {{site.data.keyword.Bluemix_notm}}.

Detaillierte Schritte finden Sie unter [APNs-Berechtigungsnachweise abrufen und den {{site.data.keyword.mobilepushshort}}-Service konfigurieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials).
{: tip}

## {{site.data.keyword.mobilepushshort}} konfigurieren, senden und überwachen
{: #configure_push}

1. Ersetzen Sie in der Datei 'index.js' unter der Funktion `onDeviceReady` die Werte für `{pushAppGuid}` und

   `{pushClientSecret} `mit **Berechtigungsnachweisen** für den Push-Service - *appGuid* und *clientSecret*.

2. Wechseln Sie zu 'Mobiles Dashboard > Projekte > Cordova-Projekt, klicken Sie auf den {{site.data.keyword.mobilepushshort}}-Service und führen Sie die nachfolgenden Schritte aus.

### APNs - Serviceinstanz konfigurieren

Um den {{site.data.keyword.mobilepushshort}}-Service zum Senden von Benachrichtigungen zu verwenden, laden Sie das .p12-Zertifikat herunter, das Sie im obigen Schritt erstellt haben. Dieses Zertifikat enthält den privaten Schlüssel und die SSL-Zertifikate, die zum Erstellen und Veröffentlichen Ihrer Anwendung erforderlich sind.

**Hinweis:** Sobald die Datei `.cer` in Ihrer Schlüsselbundverwaltung enthalten ist, exportieren Sie sie auf Ihren Computer, um ein `.p12`-Zertifikat zu erstellen.

1. Klicken Sie unter dem Abschnitt 'Services' auf `{{site.data.keyword.mobilepushshort}}` oder neben dem {{site.data.keyword.mobilepushshort}}-Service auf die drei vertikalen Punkte und wählen Sie `Dashboard öffnen` aus.
2. Im {{site.data.keyword.mobilepushshort}}-Dashboard sollte die Option `Konfigurieren` unter `Verwalten > Benachrichtigungen senden` angezeigt werden.

Führen Sie die folgenden Schritte aus, um APNs in der Konsole für `Push Notifications-Services` zu konfigurieren:

1. Wählen Sie im Dashbaord für Push Notificiations-Services die Option `Konfigurieren` aus.
2. Wählen Sie die Option `Mobil` aus, um die Informationen im Formular für die APNs-Push-Berechtigungsnachweise zu aktualisieren.
3. Wählen Sie entsprechend nach Bedarf `Sandbox (Entwicklung)` oder `Produktion (Verteilung)` aus und laden Sie dann das erstellte `p.12`-Zertifikat hoch.
4. Geben Sie in das Feld 'Kennwort' das Kennwort ein, das der .p12-Zertifikatsdatei zugeordnet ist, und klicken Sie anschließend auf 'Speichern'.

### FCM - Serviceinstanz konfigurieren

1. Wählen Sie **Mobil** aus und aktualisieren Sie dann die Registerkarte 'GCM/FCM-Push-Berechtigungsnachweise' mit der Absender-ID/Projektnummer und dem API-Schlüssel (Serverschlüssel), die Sie ursprünglich in der Firebase-Konsole erstellt haben.
2. Klicken Sie auf **Speichern**. Der {{site.data.keyword.mobilepushshort}}-Service ist jetzt konfiguriert.

### {{site.data.keyword.mobilepushshort}} senden

1. Wählen Sie **Benachrichtigungen senden** aus und verfassen Sie eine Nachricht, indem Sie eine Sendeoption auswählen. Die unterstützten Optionen sind 'Geräte nach Tag', 'Geräte-ID', 'Benutzer-ID', 'Android-Geräte', 'iOS-Geräte', 'Webbenachrichtigungen' und 'Alle Geräte'.
     **Hinweis:** Wenn Sie die Option **Alle Geräte** auswählen, erhalten alle Geräte Benachrichtigungen, die für {{site.data.keyword.mobilepushshort}} subskribiert sind.

2. Erstellen Sie im Feld **Nachricht** Ihre Nachricht. Wählen Sie aus, dass die optionalen Einstellungen nach Bedarf konfiguriert werden sollen.
3. Klicken Sie auf **Senden** und überprüfen Sie, ob die physische Einheit die Benachrichtigung erhalten hat.

### Gesendete Benachrichtigungen überwachen

Sie können Ihre gesendeten Benachrichtigungen überwachen, indem Sie zum Abschnitt **Überwachung** navigieren.
Der IBM {{site.data.keyword.mobilepushshort}}-Service weist jetzt eine erweiterte Funktionalität auf, um die Push-Leistung zu überwachen, indem anhand Ihrer Benutzerdaten Diagramme generiert werden. Sie können mit dem Dienstprogramm alle gesendeten {{site.data.keyword.mobilepushshort}} oder alle registrierten Geräte auflisten und Informationen auf täglicher, wöchentlicher oder monatlicher Basis analysieren.
      ![](images/solution6/monitoring_messages.png)

## Zugehöriger Inhalt
{: #related_content}

- [Tag-basierte Benachrichtigungen](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}}-APIs](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sicherheit in {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

