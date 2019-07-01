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

# Mobile iOS-Anwendung mit Push Notifications
{: #ios-mobile-push-analytics}

Erfahren Sie, wie einfach es ist, schnell eine iOS-Swift-Anwendung mit einem hochwertigen mobilen Service wie {{site.data.keyword.mobilepushshort}} on {{site.data.keyword.Bluemix_short}} zu erstellen.

Dieses Lernprogramm führt Sie durch die Erstellung einer mobilen Starteranwendung, das Hinzufügen eines mobilen Service, das Einrichten eines Client-SDK, das Importieren des Codes in Xcode und das anschließende weitere Verbessern der Anwendung.
{:shortdesc: .shortdesc}

## Lernziele
{:#objectives}

- Mobile App mit {{site.data.keyword.mobilepushshort}}- und {{site.data.keyword.mobileanalytics_short}}-Services über das Swift-Basis-Starter-Kit erstellen
- APNs-Berechtigungsnachweise abrufen und eine {{site.data.keyword.mobilepushshort}}-Serviceinstanz konfigurieren
- Code herunterladen und Client-SDK einrichten
- {{site.data.keyword.mobilepushshort}} senden und überwachen

  ![](images/solution6/Architecture.png)

## Produkte
{:#products}

Dieses Lernprogramm verwendet die folgenden Produkte:
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Vorbereitende Schritte
{: #prereqs}

1. Sie benötigen ein [Apple Developers](https://developer.apple.com/)-Konto zum Senden ferner Benachrichtigungen von der {{site.data.keyword.mobilepushshort}}-Serviceinstanz in {{site.data.keyword.Bluemix_short}} (dem Provider) an iOS-Geräte und -Anwendungen.
2. Sie benötigen Xcode für das Importieren und Erweitern des Codes.

## Mobile App über das Swift-Basis-Starter-Kit erstellen
{: #get_code}

1. Navigieren Sie zum [mobilen Dashboard](https://{DomainName}/developer/mobile/dashboard), um Ihre `App` über vordefinierte `Starter-Kits` zu erstellen.
2. Klicken Sie auf **Starter-Kits** und blättern Sie nach unten, um das **Basis**-Starter-Kit auszuwählen.
    ![](images/solution6/mobile_dashboard.png)
3. Geben Sie einen App-Namen ein, der gleichzeitig das Xcode-Projekt und der App-Name sein wird.
4. Wählen Sie `iOS Swift` als Plattform aus und klicken Sie auf **Erstellen**.
    ![](images/solution6/create_mobile_project.png)
5. Klicken Sie auf **Ressource hinzufügen** > Mobil > **Push Notifications** und wählen Sie die Position aus, die Sie für den Service, die Ressourcengruppe und den **Lite**-Preisplan bereitstellen möchten.
6. Klicken Sie auf **Erstellen**, um den {{site.data.keyword.mobilepushshort}}-Service bereitzustellen. Auf der Registerkarte **Apps** wird eine neue App erstellt.

**Hinweis:** Der {{site.data.keyword.mobilepushshort}}-Service sollte bereits mit dem leeren Starter hinzugefügt werden.

## Code herunterladen und Client-SDKs einrichten
{: #download_code}

Wenn Sie den Code noch nicht heruntergeladen haben, klicken Sie unter 'Apps' > `Ihre mobile App` auf `Code herunterladen`.
Der heruntergeladene Code enthält das **{{site.data.keyword.mobilepushshort}}**-Client-SDK. Das Client SDK ist auf CocoaPods und Carthage verfügbar. Für diese Lösung verwenden Sie CocoaPods.

1. Um CocoaPods auf Ihrer Maschine zu installieren, öffnen Sie den `Terminal` und führen Sie den folgenden Befehl aus.
   ```
   sudo gem install cocoapods
   ```
   {: pre:}
2. Dekomprimieren Sie den heruntergeladenen Code und verwenden Sie den Terminal, um zum dekomprimierten Ordner zu navigieren.

   ```
   cd <projektname>
   ```
   {: pre:}
3. Der Ordner enthält bereits eine `Podfile` mit erforderlichen Abhängigkeiten. Führen Sie den folgenden Befehl aus, um die Abhängigkeiten (Client-SDKs) zu installieren, und die erforderlichen Abhängigkeiten werden installiert.

  ```
  pod install
  ```
  {: pre:}

## APNs-Berechtigungsnachweise abrufen und eine {{site.data.keyword.mobilepushshort}}-Serviceinstanz konfigurieren
{: #obtain_apns_credentials}

   Bei iOS-Geräten und -Anwendungen können Anwendungsentwickler mit dem Apple Push Notification Service (APNs) ferne Benachrichtigungen von der {{site.data.keyword.mobilepushshort}}-Serviceinstanz in {{site.data.keyword.Bluemix_short}} (dem Provider) an iOS-Geräte und -Anwendungen senden. Nachrichten werden an eine Zielanwendung auf dem Gerät gesendet.

   Sie müssen die APNs-Berechtigungsnachweise anfordern und konfigurieren. Die APNs-Zertifikate werden sicher vom {{site.data.keyword.mobilepushshort}}-Service verwaltet und verwendet, um eine Verbindung zum APNs-Server als Provider herzustellen.

### App-ID registrieren

   Die App-ID (die Bundle-ID) ist eine eindeutige Kennung, die eine bestimmte Anwendung kennzeichnet. Jede Anwendung benötigt eine App-ID. Services wie der {{site.data.keyword.mobilepushshort}}-Service werden für die App-ID konfiguriert.
   Stellen Sie sicher, dass Sie über ein [Apple Developer](https://developer.apple.com/)-Konto verfügen. Dies ist eine obligatorische Voraussetzung.

   1. Wechseln Sie zum [Apple Developer](https://developer.apple.com/)-Portal, klicken Sie auf das `Mitgliedscenter` und wählen Sie `Zertifikate, IDs & Profile` aus.
   2. Wechseln Sie zum Abschnitt `IDs` > 'App-IDs'.
   3. Geben Sie auf der Seite zum `Registrieren von App-IDs` den App-Namen im Feld für den beschreibenden Namen der App-ID ein. Beispiel: ACME {{site.data.keyword.mobilepushshort}}. Geben Sie eine Zeichenfolge für das App-ID-Präfix an.
   4. Wählen Sie für das Suffix der App-ID die Option `Explizite Anwendungs-ID` aus und geben Sie einen Wert für die Bundle-ID an. Es wird empfohlen, eine Zeichenfolge im Stil eines Reverse-DNS-Eintrags zu verwenden. Beispiel: com.ACME.push.
   5. Aktivieren Sie das Kontrollkästchen `{{site.data.keyword.mobilepushshort}}` und klicken Sie auf `Weiter`.
   6. Sehen Sie sich die Einstellungen an und klicken Sie auf `Registrieren` > `Fertig`.
     Ihre App-ID ist jetzt registriert.

     ![](images/solution6/push_ios_register_appid.png)

### APNs-SSL-Zertifikat für die Entwicklung und Verteilung erstellen
   Bevor Sie ein APNs-Zertifikat abrufen, müssen Sie zuerst eine Zertifikatssignieranforderung (CSR) generieren und sie an Apple, die Zertifizierungsstelle (CA), übergeben. Die Zertifikatssignieranforderung enthält Informationen, die Ihr Unternehmen und Ihren öffentlichen und privaten Schlüssel identifizieren, die Sie zum Signieren für Ihren Apple {{site.data.keyword.mobilepushshort}} verwenden. Generieren Sie dann das SSL-Zertifikat im iOS Developer-Portal. Das Zertifikat wird zusammen mit dem öffentlichen und dem privaten Schlüssel in der Schlüsselbundverwaltung gespeichert.
   Sie können APNs in zwei Modi verwenden:

- Sandbox-Modus für Entwicklung und Tests.
- Produktionsmodus bei der Verteilung von Anwendungen über den App Store (oder andere Unternehmensverteilungsmechanismen).

   Sie müssen separate Zertifikate für Ihre Entwicklungs- und Verteilungsumgebungen abrufen. Die Zertifikate sind einer App-ID für die App zugeordnet, die der Empfänger ferner Benachrichtigungen ist. Für die Produktion können Sie bis zu zwei Zertifikate erstellen. {{site.data.keyword.Bluemix_short}} verwendet die Zertifikate zum Herstellen einer SSL-Verbindung mit APNs.

   1. Rufen Sie die Apple Developer-Website auf, klicken Sie auf das **Mitgliedscenter** und wählen Sie **Zertifikate, IDs & Profile** aus.
   2. Klicken Sie im Bereich **IDs** auf **App-IDs**.
   3. Wählen Sie in der Liste der App-IDs Ihre App-ID und anschließend `Bearbeiten` aus.
   4. Aktivieren Sie das Kontrollkästchen **{{site.data.keyword.mobilepushshort}}** und klicken Sie dann im Teilfenster **SSL-Zertifikat für Entwicklung** auf **Zertifikat erstellen**.

     ![SSL-Zertifikat für Push Notifications](images/solution6/certificate_createssl.png)

   5. Wenn die Anzeige zum **Erstellen einer Zertifikatssignieranforderung (CSR)** angezeigt wird, befolgen Sie die angezeigten Anweisungen zum Generieren einer CSR-Datei. Geben Sie einen aussagekräftigen Namen an, mit dem Sie feststellen können, ob es sich um ein Zertifikat für die Entwicklung (Sandbox) oder die Verteilung (Produktion) handelt, z. B. 'sandbox-apns-certificate' oder 'production-apns-certificate'.
   6. Klicken Sie auf **Weiter** und klicken Sie in der Anzeige 'CSR-Datei hochladen' auf **Datei auswählen**. Wählen Sie dann die Datei **CertificateSigningRequest.certSigningRequest** aus, die Sie soeben erstellt haben. Klicken Sie auf **Weiter**.
   7. Klicken Sie im Teilfenster zum Herunterladen, Installieren und Sichern auf 'Herunterladen'. Die Datei **aps_development.cer** wird heruntergeladen.
        ![Zertifikat herunterladen](images/solution6/push_certificate_download.png)

        ![Zertifikat generieren](images/solution6/generate_certificate.png)
   8. Öffnen Sie in Ihrem Mac die **Schlüsselverwaltung**, klicken Sie auf **Datei**, **Importieren** und wählen Sie die heruntergeladene .cer-Datei aus, um sie zu installieren.
   9. Klicken Sie mit der rechten Maustaste auf das neue Zertifikat und den privaten Schlüssel, wählen Sie dann **Exportieren** aus und ändern Sie das **Dateiformat** in das Personal Information Exchange-Format (`.p12`-Format).
     ![Zertifikate und Schlüssel exportieren](images/solution6/keychain_export_key.png)
   10. Geben Sie im Feld **Sichern als** dem Zertifikat einen aussagekräftigen Namen, bespielsweise `sandbox_apns.p12` oder **production_apns.p12** und klicken Sie auf 'Sichern'.
     ![Zertifikat und Schlüssel exportieren](images/solution6/certificate_p12v2.png)
   11. Geben Sie in das Feld **Kennwort eingeben** ein Kennwort ein, um die exportierten Elemente zu schützen, und klicken Sie anschließend auf 'OK'. Sie können dieses Kennwort verwenden, um die APNs-Einstellungen in der {{site.data.keyword.mobilepushshort}}-Servicekonsole zu konfigurieren.
       ![Zertifikate und Schlüssel exportieren](images/solution6/export_p12.png)
   12. Die **Key Access.app** fordert Sie auf, Ihren Schlüssel aus der Anzeige **Schlüsselbund** zu exportieren. Geben Sie Ihr Verwaltungskennwort für Ihren Mac ein, damit Ihr System diese Elemente exportieren kann, und wählen Sie dann die Option `Immer zulassen` aus. Auf Ihrem Desktop wird ein `.p12`-Zertifikat generiert.

      Klicken Sie im Teilfenster **Produktions-SSL-Zertifikat** für das Produktions-SSL auf **Zertifikat erstellen** und wiederholen Sie die Schritte 5 bis 12.
      {:tip}

### Bereitstellungsprofil für die Entwicklung erstellen
   Gemeinsam mit der App-ID kann mit dem Bereitstellungsprofil ermittelt werden, auf welchen Geräte Ihre App installiert und ausgeführt werden kann und auf welche Services sie Zugriff hat. Für jede App-ID erstellen Sie zwei Bereitstellungsprofile: eines für die Entwicklung und das andere für die Verteilung. Xcode verwendet das Bereitstellungsprofil für die Entwicklung, um festzustellen, welche Entwickler die Anwendung erstellen und welche Geräte in der Anwendung getestet werden dürfen.

   Stellen Sie sicher, dass Sie eine App-ID registriert, diese für den {{site.data.keyword.mobilepushshort}}-Service aktiviert und für die Verwendung eines APNs-SSL-Zertifikats für die Entwicklung und Produktion konfiguriert haben.

   Erstellen Sie wie folgt ein Bereitstellungsprofil für die Entwicklung:

   1. Wechseln Sie zum [Apple Developer](https://developer.apple.com/)-Portal, klicken Sie auf das `Mitgliedscenter` und wählen Sie `Zertifikate, IDs & Profile` aus.
   2. Wechseln Sie in die [Mac Developer-Bibliothek](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site), blättern Sie nach unten zum Abschnitt zum `Erstellen von Bereitstellungsprofilen für die Entwicklung` und befolgen Sie die Anweisungen, um ein Entwicklungsprofil zu erstellen.
     **Hinweis:** Wenn Sie ein Bereitstellungsprofil für die Entwicklung erstellen, wählen Sie die folgenden Optionen aus:

     - **iOS-App-Entwicklung**
     - **Für iOS- und watchOS-Apps**

### Bereitstellungsprofil für die Store-Verteilung erstellen
   Verwenden Sie das Bereitstellungsprofil für die Store-Verteilung, um Ihre App für die Verteilung an den App Store zu übergeben.

   1. Wechseln Sie zum [Apple Developer](https://developer.apple.com/)-Portal, klicken Sie auf das `Mitgliedscenter` und wählen Sie `Zertifikate, IDs & Profile` aus.
   2. Doppelklicken Sie auf das heruntergeladene Bereitstellungsprofil, um es in Xcode zu installieren.
     Nachdem Sie die Berechtigungsnachweise abgerufen haben, ist der nächste Schritt die [Konfiguration einer Serviceinstanz](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2).

### Serviceinstanz konfigurieren

   Um den {{site.data.keyword.mobilepushshort}}-Service zum Senden von Benachrichtigungen zu verwenden, laden Sie das .p12-Zertifikat herunter, das Sie im obigen Schritt erstellt haben. Dieses Zertifikat enthält den privaten Schlüssel und die SSL-Zertifikate, die zum Erstellen und Veröffentlichen Ihrer Anwendung erforderlich sind.

   **Hinweis:** Sobald die `.cer`-Datei in der Schlüsselbundverwaltung enthalten ist, exportieren Sie sie auf Ihren Computer, um ein `.p12`-Zertifikat zu erstellen.

1. Klicken Sie unter dem Abschnitt 'Services' auf {{site.data.keyword.mobilepushshort}} oder neben dem {{site.data.keyword.mobilepushshort}}-Service auf die drei vertikalen Punkte und wählen Sie `Dashboard öffnen` aus.
2. Im {{site.data.keyword.mobilepushshort}}-Dashboard sollte die Option `Konfigurieren` unter `Verwalten` angezeigt werden.

Führen Sie die folgenden Schritte aus, um APNs in der Konsole für `Push Notifications-Services` zu konfigurieren:

1. Wählen Sie die Option `Mobile` aus, um die Informationen im Formular für die APNs-Push-Berechtigungsnachweise zu aktualisieren.
2. Wählen Sie `Sandbox/APNs-Server für Entwicklung` oder `APNs-Server für Produktion` nach Bedarf aus und laden Sie dann das erstellte `.p12`-Zertifikat hoch.
3. Geben Sie in das Feld 'Kennwort' das Kennwort ein, das der .p12-Zertifikatsdatei zugeordnet ist, und klicken Sie anschließend auf 'Speichern'.

![](images/solution6/Mobile_push_configure.png)

## {{site.data.keyword.mobilepushshort}} konfigurieren, senden und überwachen
{: #configure_push}

1. Der Push-Initialisierungscode (unter `func application`) und der Benachrichtungsregistrierungscode befinden sich in `AppDelegate.swift`. Geben Sie eine eindeutige USER_ID (optional) an.
2. Führen Sie die App auf einem physischen Gerät aus, da Benachrichtigungen nicht an einen iPhone-Simulator gesendet werden können.
3. Öffnen Sie den {{site.data.keyword.mobilepushshort}}-Service unter `Mobile Services` > **Vorhandene Services** im {{site.data.keyword.Bluemix_short}} Mobile-Dashboard. Führen Sie die folgenden Schritte aus, um grundlegende {{site.data.keyword.mobilepushshort}} zu senden:
  * Wählen Sie `Nachrichten` aus und verfassen Sie eine Nachricht, indem Sie eine Sendeoption auswählen. Die unterstützten Optionen sind 'Geräte nach Tag', 'Geräte-ID', 'Benutzer-ID', 'Android-Geräte', 'iOS-Geräte', 'Webbenachrichtigungen' und 'Alle Geräte' und andere Browser.

       **Hinweis:** Wenn Sie die Option 'Alle Geräte' auswählen, erhalten alle Geräte Benachrichtigungen, die {{site.data.keyword.mobilepushshort}} subskribiert hat.
  * Erstellen Sie im Feld `Nachricht` Ihre Nachricht. Wählen Sie aus, dass die optionalen Einstellungen nach Bedarf konfiguriert werden sollen.
  * Klicken Sie auf `Senden` und überprüfen Sie, ob die physischen Geräte die Benachrichtigung erhalten haben.
    ![](images/solution6/send_notifications.png)

4. Sie sollten eine Benachrichtigung auf Ihrem iPhone sehen.

   ![](images/solution6/iphone_notification.png)

5. Sie können Ihre gesendeten Benachrichtigungen überwachen, indem Sie zu `Überwachung` im {{site.data.keyword.mobilepushshort}}-Service navigieren.

Der IBM {{site.data.keyword.mobilepushshort}}-Service weist jetzt eine erweiterte Funktionalität auf, um die Push-Leistung zu überwachen, indem anhand Ihrer Benutzerdaten Diagramme generiert werden. Sie können mit dem Dienstprogramm alle gesendeten {{site.data.keyword.mobilepushshort}} oder alle registrierten Geräte auflisten und Informationen auf täglicher, wöchentlicher oder monatlicher Basis analysieren.
      ![](images/solution6/monitoring_messages.png)

## Zugehöriger Inhalt
{: #related_content}

- [Tag-basierte Benachrichtigungen](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}}-APIs](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sicherheit in {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)
