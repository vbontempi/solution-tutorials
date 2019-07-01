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

# Mobile Anwendung mit serverunabhängigem Back-End
{: #serverless-mobile-backend}

In diesem Lernprogramm erfahren Sie, wie Sie mit {{site.data.keyword.openwhisk}} in Kombination mit kognitiven Services und Datenservices ein serverunabhängiges Back-End für eine mobile Anwendung erstellen können.
{:shortdesc}

Nicht alle Entwickler mobiler Anwendungen verfügen über Vorkenntnisse in der Verwaltung der serverseitigen Logik oder eines Servers. Solche Entwickler möchten sich bei ihrer Arbeit vor allem auf die zu erstellende App konzentrieren. Doch wie wäre es, wenn sie ihr vorhandenes Entwicklungs-Knowhow auch beim Schreiben des gewünschten mobilen Back-Ends nutzen könnten?

{{site.data.keyword.openwhisk_short}} ist eine serverunabhängige, ereignisgesteuerte Plattform. Wie [in diesem Beispiel dargestellt](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp) können die von Ihnen bereitgestellten Aktionen schnell und einfach in HTTP-Endpunkte in *Webaktionen* umgewandelt werden, um die Back-End-API für eine Webanwendung zu erstellen. Da eine Webanwendung ein Client für die REST-API ist, kann der Ansatz aus dem vorliegenden Beispiel auch auf die Erstellung eines Back-Ends für eine mobile App übertragen werden. Mithilfe von {{site.data.keyword.openwhisk_short}} können Entwickler mobiler Apps die Aktionen in derselben Sprache schreiben, die auch für die mobile App verwendet wird (Java für Android und Swift für iOS).

Dieses Lernprogramm kann entsprechend Ihrer Zielplattform konfiguriert werden. Momentan ist die Dokumentation der Version für **iOS / Swift** dieses Lernprogramms geöffnet. Über die Registerkarte am oberen Rand dieser Dokumentation können Sie die Version dieses Lernprogramms für **Android / Java** auswählen.
{: swift}

Dieses Lernprogramm kann entsprechend Ihrer Zielplattform konfiguriert werden. Momentan ist die Dokumentation der Version für **Android / Java** dieses Lernprogramms geöffnet. Über die Registerkarte am oberen Rand dieser Dokumentation können Sie die Version dieses Lernprogramms für **iOS / Swift** auswählen.
{: java}

## Ziele
{: #objectives}

* Serverunabhängiges mobiles Back-End mit {{site.data.keyword.openwhisk_short}} bereitstellen
* Benutzerauthentifizierung für eine mobile App mit {{site.data.keyword.appid_short}} hinzufügen
* Benutzerfeedback mit {{site.data.keyword.toneanalyzershort}} analysieren
* Benachrichtigungen mit {{site.data.keyword.mobilepushshort}} senden

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur
{: #architecture}

Die in diesem Lernprogramm dargestellte Anwendung ist eine Feedback-App zum intelligenten Analysieren der Tendenz des Feedback-Texts, um dem Kunden eine entsprechende Bestätigung durch eine {{site.data.keyword.mobilepushshort}} zu senden.

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. Der Benutzer authentifiziert sich bei [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID). {{site.data.keyword.appid_short}} stellt Zugriffs- und Authentifizierungstoken bereit.
2. Nachfolgende Aufrufe der Back-End-API enthalten das Zugriffstoken.
3. Das Back-End wird mit [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) implementiert. Die serverunabhängigen Aktionen sind als Webaktionen zugänglich und erwarten, dass das Token in den Anforderungsheadern übergeben wird. Sie prüfen die Gültigkeit des Tokens (Signatur und Ablaufdatum), bevor der Zugang zur eigentlichen API gewährt wird.
4. Wenn der Benutzer Feedback abschickt, wird das Feedback in [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) gespeichert.
5. Der Feedbacktext wird mit [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) verarbeitet.
6. Je nach Analyseergebnis wird mit [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) eine Benachrichtigung zurück an den Benutzer gesendet.
7. Der Benutzer empfängt die Benachrichtigung auf dem Gerät.

## Vorbereitende Schritte
{: #prereqs}

Im vorliegenden Lernprogramm wird das {{site.data.keyword.Bluemix_notm}}-Befehlszeilentool verwendet, um Ressourcen bereitzustellen und Code zu implementieren. Stellen Sie sicher, dass das Befehlszeilentool `ibmcloud` installiert ist.

* [{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Script zum Installieren der Befehlszeilenschnittstelle 'ibmcloud' und der erforderlichen Plug-ins (Cloud Foundry und {{site.data.keyword.openwhisk_short}})

Außerdem benötigen Sie die folgende Software und die folgenden Konten:

   1. Java 8
   2. Android Studio 2.3.3
   3. Google Developer-Konto zum Konfigurieren von Firebase Cloud Messaging
   4. Bash-Shell, cURL
   {: java}


   1. Xcode
   2. Apple Developer-Konto zum Konfigurieren von Apple Push Notification Service
   3. Bash-Shell, cURL
   {: swift}

In diesem Lernprogramm konfigurieren Sie Push-Benachrichtigungen für die Anwendung. Im Lernprogramm wird vorausgesetzt, dass Sie das grundlegende {{site.data.keyword.mobilepushshort}}-Lernprogramm für [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) oder für [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics) durchgearbeitet haben und mit der Konfiguration von Firebase Cloud Messaging oder Apple Push Notification Service vertraut sind.
{:tip}

Für Windows 10-Benutzer, die mit den Befehlszeilenanweisungen arbeiten möchten, wird empfohlen, das Windows-Subsystem für Linux und Ubuntu zu installieren, wie in [diesem Artikel](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10) beschrieben.
{: tip}
{: java}

## Anwendungscode abrufen

Das Repository enthält sowohl den Code der mobilen Anwendung als auch den {{site.data.keyword.openwhisk_short}}-Aktionscode.

1. Rufen Sie den Code aus dem GitHub-Repository ab: 

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

2. Überprüfen Sie die Codestruktur.

| Datei                                    | Beschreibung                             |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions) | Code für die {{site.data.keyword.openwhisk_short}}-Aktionen des serverunabhängigen mobilen Back-Ends |
| [**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android) | Code für die mobile Anwendung          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh) | Helper-Script zum Installieren, Deinstallieren und Aktualisieren der {{site.data.keyword.openwhisk_short}}-Auslöser, -Aktionen und -Regeln |
{: java}

| Datei                                     | Beschreibung                            |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions) | Code für die {{site.data.keyword.openwhisk_short}}-Aktionen des serverunabhängigen mobilen Back-Ends |
| [**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp) | Code für die mobile Anwendung          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh) | Helper-Script zum Installieren, Deinstallieren und Aktualisieren der {{site.data.keyword.openwhisk_short}}-Auslöser, -Aktionen und -Regeln |
{: swift}

## Services zum Verarbeiten von Benutzerauthentifizierung, Feedback-Persistenz und Analyse bereitstellen
{: #provision_services}

In diesem Abschnitt stellen Sie die von der Anwendung verwendeten Services bereit. Sie können die Services wahlweise über den {{site.data.keyword.Bluemix_notm}}-Katalog oder mithilfe der `ibmcloud`-Befehlszeile bereitstellen.

Es wird empfohlen, einen neuen Bereich zum Bereitstellen der Services und zum Implementieren des serverunabhängigen Back-Ends zu erstellen. Auf diese Weise können alle Ressourcen gebündelt werden.

### Services aus dem {{site.data.keyword.Bluemix_notm}}-Katalog bereitstellen

1. Rufen Sie den [{{site.data.keyword.Bluemix_notm}}-Katalog](https://{DomainName}/catalog/) auf.
2. Erstellen Sie einen [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)-Service mit dem **Lite**-Plan. Geben Sie den Namen **serverlessfollowup-db** an.
3. Erstellen Sie einen [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)-Service mit dem **Standard**-Plan. Geben Sie den Namen **serverlessfollowup-tone** an.
4. Erstellen Sie einen [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)-Service mit dem Plan für **gestaffelte Preisstufe**. Geben Sie den Namen **serverlessfollowup-appid** an.
5. Erstellen Sie einen [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)-Service mit dem **Lite**-Plan. Geben Sie den Namen **serverlessfollowup-mobilepush** an.

### Services über die Befehlszeile bereitstellen

Führen Sie in der Befehlszeile die folgenden Befehle aus, um die Services bereitzustellen und die zugehörigen Berechtigungsnachweise abzurufen:

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

## {{site.data.keyword.mobilepushshort}} konfigurieren
{: #push_notifications}

Wenn ein Benutzer neues Feedback abschickt, wird das Feedback von der Anwendung analysiert und eine Benachrichtigung an den Benutzer zurück gesendet. Der Benutzer hat inzwischen möglicherweise eine andere Task begonnen oder die mobile App beendet. Daher ist das Senden einer Push-Benachrichtigung eine gute Methode zum Kommunizieren mit dem Benutzer. Der {{site.data.keyword.mobilepushshort}}-Service ermöglicht das Senden von Benachrichtigungen an iOS oder Android über eine einheitliche API. In diesem Abschnitt konfigurieren Sie den {{site.data.keyword.mobilepushshort}}-Service für Ihre Zielplattform.

### Firebase Cloud Messaging (FCM) konfigurieren
{: java}

   1. Erstellen Sie in der [Firebase-Konsole](https://console.firebase.google.com) ein neues Projekt. Geben Sie den Namen **serverlessfollowup** an.
   2. Navigieren Sie zu den **Einstellungen** für das Projekt.
   3. Fügen Sie auf der Registerkarte **Allgemein** zwei Anwendungen hinzu:
      1. Eine Anwendung mit dem Paketnamen **com.ibm.mobilefirstplatform.clientsdk.android.push**
      2. Eine Anwendung mit dem Paketnamen **serverlessfollowup.app**
   4. Laden Sie die Datei `google-services.json` mit den beiden definierten Anwendungen aus der Firebase-Konsole herunter und speichern Sie sie im Ordner `android/app` des Verzeichnisses 'checkout'.
   5. Suchen Sie die Absender-ID und den Serverschlüssel (im Folgenden auch 'API-Schlüssel' genannt) auf der Registerkarte **Cloud Messaging**.
   6. Geben Sie im Service-Dashboard für Push-Benachrichtigungen den Wert für Absender-ID und API-Schlüssel an.
   {: java}

### Apple Push Notifications Service (APNs) konfigurieren
{: swift}

1. Rufen Sie das Portal [Apple Developer](https://developer.apple.com/) auf und registrieren Sie eine App-ID.
2. Erstellen Sie ein APNs-SSL-Zertifikat für Entwicklung und Verteilung.
3. Erstellen Sie ein Profil für Entwicklungsbereitstellung.
4. Konfigurieren Sie die {{site.data.keyword.mobilepushshort}}-Serviceinstanz in {{site.data.keyword.Bluemix_notm}}. Eine ausführliche Anleitung finden Sie unter [APNs-Berechtigungsnachweise abrufen und {{site.data.keyword.mobilepushshort}}-Service konfigurieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-).
{: swift}

## Serverunabhängiges Back-End implementieren
{: #serverless_backend}

Nachdem alle Services konfiguriert sind, können Sie nun das serverunabhängige Back-End implementieren. In diesem Abschnitt werden die folgenden {{site.data.keyword.openwhisk_short}}-Artefakte erstellt:

| Artefakt                      | Typ                            | Beschreibung                             |
| ----------------------------- | ------------------------------ | ---------------------------------------- |
| `serverlessfollowup`          | Paket                        | Ein Paket zum Gruppieren der Aktionen und zum Aufbewahren aller Serviceberechtigungsnachweise |
| `serverlessfollowup-cloudant` | Paketbindung                | An das integrierte {{site.data.keyword.cloudant_short_notm}}-Paket gebunden   |
| `serverlessfollowup-push`     | Paketbindung                | An das {{site.data.keyword.mobilepushshort}}-Paket gebunden  |
| `auth-validate`               | Aktion                        | Prüft die Zugriffs- und Identifikationstokens |
| `users-add`                   | Aktion                        | Speichert die Benutzerinformationen (ID, Name, E-Mail, Bild, Geräde-ID) |
| `users-prepare-notify`        | Aktion                        | Formatiert eine Nachricht für die Verwendung mit {{site.data.keyword.mobilepushshort}} |
| `feedback-put`                | Aktion                        | Speichert ein Benutzerfeedback in der Datenbank   |
| `feedback-analyze`            | Aktion                        | Analysiert ein Feedback mit {{site.data.keyword.toneanalyzershort}}   |
| `users-add-sequence`          | Als Webaktion verfügbare Folge | `auth-validate` und `users-add`          |
| `feedback-put-sequence`       | Als Webaktion verfügbare Folge | `auth-validate` und `feedback-put`       |
| `feedback-analyze-sequence`   | Folge                           | `read-document` aus {{site.data.keyword.cloudant_short_notm}}, `feedback-analyze`, `users-prepare-notify` und `sendMessage` mit {{site.data.keyword.mobilepushshort}} |
| `feedback-analyze-trigger`    | Auslöser                         | Wird von {{site.data.keyword.openwhisk_short}} aufgerufen, wenn ein Feedback in der Datenbank gespeichert wird |
| `feedback-analyze-rule`       | Regel                          | Verknüpft den Auslöser `feedback-analyze-trigger` mit der Folge `feedback-analyze-sequence` |

### Code kompilieren
{: java}
1. Kompilieren Sie den Aktionscode vom Stammverzeichnis des Verzeichnisses 'checkout' aus:
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### Aktionen konfigurieren und bereitstellen
{: java}

2. Kopieren Sie 'template.local.env' in 'local.env':

   ```sh
   cp template.local.env local.env
   ```
3. Rufen Sie die Berechtigungsnachweise für {{site.data.keyword.cloudant_short_notm}}-, {{site.data.keyword.toneanalyzershort}}-, {{site.data.keyword.mobilepushshort}}-
 und {{site.data.keyword.appid_short}}-Services aus dem {{site.data.keyword.Bluemix_notm}}-Dashboard ab (oder die Ausgabe der zuvor ausgeführten ibmcloud-Befehle) und ersetzen Sie die Platzhalter in `local.env` durch entsprechende Werte. Diese Eigenschaften werden in ein Paket eingefügt, damit alle Aktionen Zugriff auf die Datenbank bekommen können.
4. Stellen Sie die Aktionen in {{site.data.keyword.openwhisk_short}} bereit. `deploy.sh` lädt die Berechtigungsnachweise aus `local.env`, um die {{site.data.keyword.cloudant_short_notm}}-Datenbanken (für Benutzer, Feedback und Modus) zu erstellen und die {{site.data.keyword.openwhisk_short}}-Artefakte für die Anwendung bereitzustellen.
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   Sie können `./deploy.sh --uninstall` verwenden, um die {{site.data.keyword.openwhisk_short}}-Artefakte zu entfernen, nachdem Sie das Lernprogramm abgeschlossen haben.
   {: tip}

### Aktionen konfigurieren und bereitstellen
{: swift}

1. Kopieren Sie 'template.local.env' in 'local.env':
   ```sh
   cp template.local.env local.env
   ```
2. Rufen Sie die Berechtigungsnachweise für {{site.data.keyword.cloudant_short_notm}}-, {{site.data.keyword.toneanalyzershort}}-, {{site.data.keyword.mobilepushshort}}-
 und {{site.data.keyword.appid_short}}-Services aus dem {{site.data.keyword.Bluemix_notm}}-Dashboard ab (oder die Ausgabe der zuvor ausgeführten ibmcloud-Befehle) und ersetzen Sie die Platzhalter in `local.env` durch entsprechende Werte. Diese Eigenschaften werden in ein Paket eingefügt, damit alle Aktionen Zugriff auf die Datenbank bekommen können.
3. Stellen Sie die Aktionen in {{site.data.keyword.openwhisk_short}} bereit. `deploy.sh` lädt die Berechtigungsnachweise aus `local.env`, um die {{site.data.keyword.cloudant_short_notm}}-Datenbanken (für Benutzer, Feedback und Modus) zu erstellen und die {{site.data.keyword.openwhisk_short}}-Artefakte für die Anwendung bereitzustellen.

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   Sie können `./deploy.sh --uninstall` verwenden, um die {{site.data.keyword.openwhisk_short}}-Artefakte zu entfernen, nachdem Sie das Lernprogramm abgeschlossen haben.
   {: tip}

## Native mobile Anwendung zum Erfassen von Benutzerfeedback konfigurieren und ausführen
{: #mobile_app}

Die {{site.data.keyword.openwhisk_short}}-Aktionen sind jetzt für die mobile App vorbereitet. Vor dem Ausführen der mobilen App müssen Sie die zugehörigen Einstellungen konfigurieren, um die von Ihnen erstellen Services als Ziel anzugeben.

1. Öffnen Sie in Android Studio das Projekt im Ordner `android` Ihres Verzeichnisses 'checkout'.
2. Bearbeiten Sie die Datei `android/app/src/main/res/values/credentials.xml` und füllen Sie die Leerstellen mit Werten aus Berechtigungsnachweisen. Sie benötigen {{site.data.keyword.appid_short}} `tenantId`, {{site.data.keyword.mobilepushshort}} `appGuid` und `clientSecret` sowie die Namen der Organisation und des Bereichs, in dem {{site.data.keyword.openwhisk_short}} implementiert wurde. Starten Sie für den API-Host die Eingabeaufforderung oder das Terminal und führen Sie den folgenden Befehl aus:
   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
3. Öffnen Sie die Datei `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` und `LoginAndRegistrationListener.java` und passen Sie für Push-Benachrichtigungen 'service(BMSClient)region' und 'AppID region' an den Standort an, an dem Ihre Serviceinstanzen erstellt wurden.
4. Erstellen Sie den Build des Projekts.
5. Starten Sie die Anwendung auf einem realen Gerät oder mit einem Emulator.
   Damit der Emulator Push-Benachrichtigungen erhält, müssen Sie ein Image mit den Google-APIs auswählen und sich im Emulator mit einem Google-Konto anmelden.
   {: tip}
6. Überwachen Sie {{site.data.keyword.openwhisk_short}} im Hintergrund:
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. Wählen Sie in der Anwendung **Anmelden** aus, um die Authentifizierung über ein Facebook- oder Google-Konto vorzunehmen. Geben Sie nach erfolgter Anmeldung eine Feedbacknachricht ein und aktivieren Sie die Schaltfläche **Feedback senden**. Wenige Sekunden nach dem Senden des Feedbacks, müssten Sie eine Push-Benachrichtigung auf dem Gerät empfangen. Der Benachrichtigungstext kann durch Bearbeiten des Vorlagendokuments in der Datenbank `moods` in der {{site.data.keyword.cloudant_short_notm}}-Serviceinstanz angepasst werden. Mithilfe der Schaltfläche **Token anzeigen** können Sie die von {{site.data.keyword.appid_short}} generierten Zugriffs- und Identifikationstoken bei der Anmeldung überprüfen.
{: java}


1. Das Push-Client-SDK und andere SDKs sind in CocoaPods und Carthage verfügbar. Für die vorliegende Lösung soll CocoaPods verwendet werden.
2. Öffnen Sie ein Terminal und wechseln Sie mit dem Befehl `cd ` in den Ordner `followupapp`. Führen Sie den folgenden Befehl aus, um die erforderlichen Abhängigkeiten zu installieren:
   ```sh
   pod install
   ```
   {: pre}
3. Öffnen Sie die Datei mit der Erweiterung `.xcworkspace` im Ordner `followupapp` Ihres Verzeichnisses 'checkout', um Ihren Code in Xcode zu starten.
4. Bearbeiten Sie die Datei `BMSCredentials.plist` und füllen Sie die Leerstellen mit Werten aus Berechtigungsnachweisen. Sie benötigen die {{site.data.keyword.appid_short}}-`tenantId`, {{site.data.keyword.mobilepushshort}}-`appGuid` und `clientSecret` sowie den Namen der Organisation und des Bereichs, in dem {{site.data.keyword.openwhisk_short}} implementiert wurde. Starten Sie für den API-Host das Terminal und führen Sie den folgenden Befehl aus:

   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
5. Öffnen Sie die Datei `AppDelegate.swift` und passen Sie die Push-Benachrichtigungen 'service(BMSClient) region' und 'AppID region' an den Standort an, an dem Ihre Serviceinstanzen erstellt werden.
6. Erstellen Sie den Build des Projekts.
7. Starten Sie die Anwendung auf einem realen Gerät oder mit einem Simulator.
8. Zeigen Sie {{site.data.keyword.openwhisk_short}} im Hintergrund an, indem Sie den folgenden Befehl in einem Terminal ausführen:
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. Wählen Sie in der Anwendung **Anmelden** aus, damit die Authentifizierung über ein Facebook- oder Google-Konto erfolgt. Geben Sie nach erfolgter Anmeldung eine Feedbacknachricht ein und aktivieren Sie die Schaltfläche **Feedback senden**. Wenige Sekunden nach dem Senden des Feedbacks, müssten Sie eine Push-Benachrichtigung auf dem Gerät empfangen. Der Benachrichtigungstext kann durch Bearbeiten des Vorlagendokuments in der Datenbank `moods` in der {{site.data.keyword.cloudant_short_notm}}-Serviceinstanz angepasst werden. Mithilfe der Schaltfläche **Token anzeigen** können Sie die von {{site.data.keyword.appid_short}} generierten Zugriffs- und Identifikationstoken bei der Anmeldung überprüfen.
{: swift}

## Ressourcen entfernen

1. Verwenden Sie `deploy.sh`, um die {{site.data.keyword.openwhisk_short}}-Artefakte zu entfernen:

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. Löschen Sie die Services {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.appid_short}}, {{site.data.keyword.mobilepushshort}} und {{site.data.keyword.toneanalyzershort}} in der {{site.data.keyword.Bluemix_notm}}-Konsole.

## Zugehörige Inhalte

* {{site.data.keyword.appid_short}} stellt eine Standardkonfiguration bereit, die Sie bei der Erstkonfiguration Ihrer Identitätsprovider unterstützt. Vor dem Publizieren Ihrer App müssen Sie [in der Konfiguration Ihre eigenen Berechtigungsnachweise angeben](https://{DomainName}/docs/services/appid?topic=appid-social#social). Außerdem können Sie [das Anmeldewidget anpassen](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget).


* Wenn Sie eine OpenWhisk-Swift-Aktion mit einer Swift-Quellendatei erstellen (.swift-Dateien sind im Ordner `actions` enthalten), muss die Datei in eine Binärdatei kompiliert werden, bevor die Aktion ausgeführt wird. Nach dieser Kompilierung können nachfolgende Aufrufe der Aktion deutlich schneller ausgeführt werden, bis der Container, der Ihre Aktion enthält, gelöscht wird. Diese anfängliche Verzögerung wird als Kaltstartverzögerung bezeichnet. Zur Vermeidung der Kaltstartverzögerung können Sie Ihre Swift-Datei in eine Binärdatei kompilieren und anschließend als ZIP-Datei in OpenWhisk hochladen. Da Sie das OpenWhisk-Gerüst benötigen, besteht die einfachste Methode zum Erstellen der Binärdatei darin, die Datei in derselben Umgebung zu erstellen, in der sie ausgeführt wird. Weitere Schritte finden Sie im Abschnitt [Aktion als ausführbare Swift-Datei paketieren](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions).
{: swift}
