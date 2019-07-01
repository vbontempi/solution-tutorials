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

# Native mobile Android-Anwendung mit Push Notifications
{: #android-mobile-push-analytics}

Erfahren Sie, wie einfach es ist, schnell eine native Android-Anwendung mit einem hochwertigen mobilen Service wie {{site.data.keyword.mobilepushshort}} on {{site.data.keyword.Bluemix_notm}} zu erstellen.

Dieses Lernprogramm führt Sie durch die Erstellung einer mobilen Starteranwendung, das Hinzufügen eines mobilen Service, das Einrichten eines Client-SDK, das Importieren des Codes in Android Studio und das anschließende weitere Verbessern der Anwendung.

## Lernziele
{: #objectives}

* Mobile App mit dem {{site.data.keyword.mobilepushshort}}-Service erstellen
* FCM-Berechtigungsnachweise abrufen
* Code herunterladen und die erforderliche Konfiguration durchführen
* {{site.data.keyword.mobilepushshort}}-Instanz konfigurieren, senden und überwachen

![](images/solution9/Architecture.png)

## Produkte
{: #products}

Dieses Lernprogramm verwendet die folgenden Produkte:
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Vorbereitende Schritte
{: #prereqs}

- Sie benötigen [Android Studio](https://developer.android.com/studio/index.html) zum Importieren und Verbessern des Codes.
- Sie benötigen ein Google-Konto für die Anmeldung bei der Firebase-Konsole, um die Absender-ID und den Server-API-Schlüssel abzurufen.

## Mobile Android-App über das Starter-Kit erstellen
{: #get_code}
Mit dem mobilen Dashboard in {{site.data.keyword.Bluemix_notm}} können Sie die Entwicklung mobiler Apps beschleunigen, indem Sie Ihre App über ein Starter-Kit erstellen.
1. Navigieren Sie zum [mobilen Dashboard](https://{DomainName}/developer/mobile/dashboard).
2. Klicken Sie auf **Starter-Kits** und dann auf **App erstellen**.
    ![](images/solution9/mobile_dashboard.png)
3. Geben Sie einen App-Namen ein; dies kann auch der Name Ihres Android-Projekts sein.
4. Wählen Sie **Android** als Plattform aus und klicken Sie auf **Erstellen**.

    ![](images/solution9/create_mobile_project.png)
5. Klicken Sie auf **Ressource hinzufügen** > Mobil > **Push Notifications** und wählen Sie die Position aus, die Sie für den Service, die Ressourcengruppe und den **Lite**-Preisplan bereitstellen möchten.
6. Klicken Sie auf **Erstellen**, um den {{site.data.keyword.mobilepushshort}}-Service bereitzustellen. Auf der Registerkarte **Apps** wird eine neue App erstellt.

    **Hinweis:** Der {{site.data.keyword.mobilepushshort}}-Service sollte bereits mit dem leeren Starter hinzugefügt werden.
    Im nächsten Schritt rufen Sie die Berechtigungsnachweise von Firebase Cloud Messaging (FCM) ab.

Im nächsten Schritt laden Sie den Gerüstcode herunter und konfigurieren das Push-Android-SDK.

## Code herunterladen und die erforderliche Konfiguration durchführen
{: #download_code}

Wenn Sie den Code noch nicht heruntergeladen haben, verwenden Sie das mobile Dashboard von {{site.data.keyword.Bluemix_notm}} zum Abrufen des Codes, indem Sie unter 'Apps > **Ihre mobile App** auf die Schaltfläche **Code herunterladen** klicken.
Der heruntergeladene Code enthält das **{{site.data.keyword.mobilepushshort}}**-Client-SDK. Das Client-SDK ist auf Gradle und Maven verfügbar. Für dieses Lernprogramm verwenden Sie **Gradle**.

1. Starten Sie Android Studio. > Wählen Sie **Vorhandenes Android Studio-Projekt öffnen** aus und zeigen Sie auf den heruntergeladenen Code.
2. Es wird automatisch ein **Gradle**-Build ausgelöst und alle Abhängigkeiten werden heruntergeladen.
3. Fügen Sie die Abhängigkeit **Google Play-Services** zur Datei `build.gradle (Module: app)` auf Modulebene am Ende hinter `dependencies{.....}` hinzu.
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. Kopieren Sie die Datei `google-services.json`, die Sie erstellt und heruntergeladen haben, in das Stammverzeichnis Ihres Android-Anwendungsmoduls. Beachten Sie, dass die Datei `google-service.json` die hinzugefügten Paketnamen enthält.
5. Die erforderlichen Berechtigungen befinden sich alle in der Datei `AndroidManifest.xml` und in den Abhängigkeiten. Push und Analytics sind in der Datei **build.gradle (Modul:app)** enthalten.
6. Der Intent-Service und Intent-Filter von **Firebase Cloud Messaging (FCM)** für die Ereignisbenachrichtigungen `RECEIVE` und `REGISTRATION` sind in der Datei `AndroidManifest.xml` enthalten.

## FCM-Berechtigungsnachweise anfordern
{: #obtain_fcm_credentials}

Firebase Cloud Messaging (FCM) ist das Gateway, das für die Bereitstellung von {{site.data.keyword.mobilepushshort}} für Android-Geräte, den Google Chrome-Browser und Chrome-Apps & -Erweiterungen verwendet wird. Wenn Sie den {{site.data.keyword.mobilepushshort}}-Service in der Konsole konfigurieren möchten, müssen Sie Ihre FCM-Berechtigungsnachweise (Absender-ID und API-Schlüssel) abrufen.

Der API-Schlüssel wird sicher gespeichert und vom {{site.data.keyword.mobilepushshort}}-Service verwendet, um eine Verbindung zum FCM-Server herzustellen; die Absender-ID (Projektnummer) wird vom Android-SDK und JS-SDK für Google Chrome und Mozilla Firefox auf der Clientseite verwendet. Führen Sie die folgenden Schritte aus, um FCM einzurichten und Ihre Berechtigungsnachweise abzurufen:

1. Rufen Sie die [Firebase-Konsole](https://console.firebase.google.com/?pli=1) auf -  hierfür ist ein Google-Benutzerkonto erforderlich.
2. Wählen Sie **Projekt hinzufügen** aus.
3. Geben Sie im Fenster **Projekt erstellen** einen Projektnamen an, wählen Sie ein Land/eine Region aus und klicken Sie auf **Projekt erstellen**.
4. Wählen Sie im linken Navigationsbereich die Option **Einstellungen** (klicken Sie dazu neben **Übersicht** auf das Symbol für Einstellungen) > **Projekteinstellungen** aus.
5. Wählen Sie die Registerkarte für das Cloud-Messaging aus, um die Berechtigungsnachweise für Ihr Projekt - den Server-API-Schlüssel und eine Absender-ID - abzurufen.
    **Hinweis:** Der in FCM aufgelistete Serverschlüssel ist mit dem Server-API-Schlüssel identisch.
    ![](images/solution9/fcm_console.png)

Außerdem müssen Sie die Datei `google-services.json` generieren. Führen Sie die folgenden Schritte aus:

1. Klicken Sie in der Firebase-Konsole auf das Symbol **Projekteinstellungen** > Registerkarte **Allgemein** unter dem Projekt, das Sie oben erstellt haben, und wählen Sie **Firebase zu Ihrer Android-App hinzufügen** aus.

    ![](images/solution9/firebase_project_settings.png)
2. Fügen Sie im Modalfenster **Firebase zu Ihrer Android-App hinzufügen** den Namen **com.ibm.mobilefirstplatform.clientsdk.android.push** als Paketnamen hinzu, um das Android-SDK von {{site.data.keyword.mobilepushshort}} zu registrieren. Die Felder 'Kurzname der App' und 'SHA-1' sind optional. Klicken Sie auf **App registrieren** > **Weiter** > **Fertigstellen**.

    ![](images/solution9/add_firebase_to_your_app.png)

3. Klicken Sie auf **App hinzufügen** > **Firebase zu Ihrer App hinzufügen**. Geben Sie den Paketnamen Ihrer Anwendung ein, indem Sie den Paketnamen **com.ibm.mysampleapp** eingeben und dann mit dem Hinzufügen von Firebase zu Ihrem Android-App-Fenster fortfahren. Die Felder 'Kurzname der App' und 'SHA-1' sind optional. Klicken Sie auf **App registrieren** > Weiter > Fertigstellen.
     **Hinweis:** Wenn Sie den Code heruntergeladen haben, finden Sie den Paketnamen der Anwendung in der Datei `AndroidManifest.xml`.
4. Laden Sie die neueste Konfigurationsdatei `google-services.json` unter **Ihre Apps** herunter.

    ![](images/solution9/google_services.png)

    **Hinweis**: FCM ist die neue Version von Google Cloud Messaging (GCM). Stellen Sie sicher, dass Sie FCM-Berechtigungsnachweise für neue Apps verwenden. Vorhandene Apps funktionieren weiterhin mit GCM-Konfigurationen.

*Die Schritte und die Benutzerschnittstelle der Firebase-Konsole können jederzeit geändert werden. Weitere Informationen finden Sie bei Bedarf in den Abschnitten zu Firebase in der Google-Dokumentation.*

## {{site.data.keyword.mobilepushshort}} konfigurieren, senden und überwachen

{: #configure_push}

1. Das {{site.data.keyword.mobilepushshort}}-SDK wurde bereits in die App importiert und Push-Initialisierungscode befindet sich in der Datei `MainActivity.java`.

    **Hinweis:** Die Serviceberechtigungsnachweise sind Teil der Datei `/res/values/credentials.xml`.
2. Die Registrierung für Benachrichtigungen erfolgt in `MainActivity.java`.  (Optional) Geben Sie eine eindeutige Benutzer-ID (USER_ID) an.
3. Führen Sie die App auf einer physischen Einheit oder einem Emulator aus, um Benachrichtigungen zu erhalten.
4. Öffnen Sie den {{site.data.keyword.mobilepushshort}}-Service unter **Mobile Services** > **Vorhandene Services** im mobilen {{site.data.keyword.Bluemix_notm}}-Dashboard. Führen Sie die folgenden Schritte aus, um grundlegende {{site.data.keyword.mobilepushshort}} zu senden:
   - Klicken Sie auf **Verwalten** > **Konfigurieren**.
   - Wählen Sie **Mobil** aus und aktualisieren Sie dann die Registerkarte 'GCM/FCM-Push-Berechtigungsnachweise' mit der Absender-ID/Projektnummer und dem API-Schlüssel (Serverschlüssel), die Sie ursprünglich in der Firebase-Konsole erstellt haben.

     ![](images/solution9/configure_push_notifications.png)
   - Klicken Sie auf **Speichern**. Der {{site.data.keyword.mobilepushshort}}-Service ist jetzt konfiguriert.
   - Wählen Sie **Benachrichtigungen senden** aus und verfassen Sie eine Nachricht, indem Sie eine Sendeoption auswählen. Die unterstützten Optionen sind 'Geräte nach Tag', 'Geräte-ID', 'Benutzer-ID', 'Android-Geräte', 'iOS-Geräte', 'Webbenachrichtigungen' und 'Alle Geräte'.
     **Hinweis:** Wenn Sie die Option **Alle Geräte** auswählen, erhalten alle Geräte Benachrichtigungen, die für {{site.data.keyword.mobilepushshort}} subskribiert sind.
   - Erstellen Sie im Feld **Nachricht** Ihre Nachricht. Wählen Sie aus, dass die optionalen Einstellungen nach Bedarf konfiguriert werden sollen.
   - Klicken Sie auf **Senden** und überprüfen Sie, ob die physische Einheit die Benachrichtigung erhalten hat.

     ![](images/solution9/android_send_notifications.png)
5. Sie sollten eine Benachrichtigung auf Ihrem Android-Gerät sehen.

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. Sie können Ihre gesendeten Benachrichtigungen überwachen, indem Sie zu **Überwachung** im {{site.data.keyword.mobilepushshort}}-Service navigieren.
     Der IBM {{site.data.keyword.mobilepushshort}}-Service weist jetzt eine erweiterte Funktionalität auf, um die Push-Leistung zu überwachen, indem anhand Ihrer Benutzerdaten Diagramme generiert werden. Sie können mit dem Dienstprogramm alle gesendeten {{site.data.keyword.mobilepushshort}} oder alle registrierten Geräte auflisten und Informationen auf täglicher, wöchentlicher oder monatlicher Basis analysieren.
      ![](images/solution6/monitoring_messages.png)

## Zugehöriger Inhalt
{: #related_content}
- [{{site.data.keyword.mobilepushshort}}-Einstellungen anpassen](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [Tag-basierte Benachrichtigungen](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}}-APIs](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sicherheit in {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

