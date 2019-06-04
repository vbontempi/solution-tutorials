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

# 使用無伺服器後端的行動應用程式
{: #serverless-mobile-backend}

在本指導教學中，您將瞭解如何將 {{site.data.keyword.openwhisk}} 與認知和資料服務一起使用，為行動應用程式建置無伺服器後端。
{:shortdesc}

並非所有行動開發人員一開始都有管理伺服器端邏輯或伺服器的經驗。他們更願意將精力集中在所建置的應用程式上。現在，如果他們可以重複使用自己現有的開發技能來撰寫行動後端會怎樣？

{{site.data.keyword.openwhisk_short}} 是一種無伺服器事件驅動型平台。如[此範例中重點闡述](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp)的那樣，部署的動作可以輕鬆轉換為 HTTP 端點，作為 *Web 動作* 來建置 Web 應用程式後端 API。透過將 Web 應用程式作為 REST API 用戶端，可以輕鬆地進一步擴展此範例，以套用相同的方法為行動式應用程式建置後端。藉由 {{site.data.keyword.openwhisk_short}}，行動開發人員在撰寫動作時可以使用用於撰寫其行動式應用程式的語言 - 對於 Android，為 Java；對於 iOS，為 Swift。

本指導教學可根據您的目標平台進行配置。您目前檢視的是本指導教學的 **iOS / Swift** 版本的文件。使用本文件頂端的標籤可選取本指導教學的 **Android / Java** 版本。
{: swift}

本指導教學可根據您的目標平台進行配置。您目前檢視的是本指導教學的 **Android / Java** 版本的文件。使用本文件頂端的標籤可選取本指導教學的 **iOS / Swift** 版本。
{: java}

## 目標
{: #objectives}

* 透過 {{site.data.keyword.openwhisk_short}} 部署無伺服器行動後端。
* 透過 {{site.data.keyword.appid_short}} 將使用者鑑別新增到行動式應用程式。
* 透過 {{site.data.keyword.toneanalyzershort}} 分析使用者意見。
* 透過 {{site.data.keyword.mobilepushshort}} 傳送通知。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

本指導教學中顯示的應用程式是一個意見回饋應用程式，它會透過 {{site.data.keyword.mobilepushshort}} 聰明地分析意見文字的語氣，並適當地確認客戶。

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. 使用者透過 [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) 進行鑑別。{{site.data.keyword.appid_short}} 提供了存取記號和識別記號。
2. 對後端 API 的進一步呼叫包含存取記號。
3. 後端由 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) 實作。作為 Web 動作公開的無伺服器動作需要在要求標頭中傳送記號並驗證其有效性（簽章和到期日），然後才容許存取實際 API。
4. 使用者提交意見時，意見將儲存在 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 中。
5. 意見文字由 [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) 進行處理。
6. 根據分析結果，透過 [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) 將通知送回給使用者。
7. 使用者在裝置上收到通知。

## 開始之前
{: #prereqs}

本指導教學使用 {{site.data.keyword.Bluemix_notm}} 指令行工具來佈建資源和部署程式碼。確保安裝 `ibmcloud` 指令行工具。

* [{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - 用於安裝 ibmcloud CLI 和必要外掛程式（Cloud Foundry 和 {{site.data.keyword.openwhisk_short}}）的 Script

此外，您還需要下列軟體和帳戶：

   1. Java 8
   2. Android Studio 2.3.3
   3. Google 開發人員帳戶，用於配置 Firebase 雲端傳訊
   4. Bash Shell、cURL
   {: java}


   1. Xcode
   2. Apple 開發人員帳戶，用於配置 Apple 推送通知服務
   3. Bash Shell、cURL
   {: swift}

在本指導教學中，您將為應用程式配置推送通知。本指導教學假設您已完成 [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) 或 [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics) 的基本 {{site.data.keyword.mobilepushshort}} 指導教學，並且您熟悉 Firebase 雲端傳訊或 Apple 推送通知服務的配置。
{:tip}

要讓 Windows 10 使用者使用指令行指示，我們建議如[此文章](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10)中所述，安裝適用於 Linux 和 Ubuntu 的 Windows 子系統。
{: tip}
{: java}

## 取得應用程式碼

儲存庫包含行動應用程式和 {{site.data.keyword.openwhisk_short}} 動作碼。

1. 從 GitHub 儲存庫中移出程式碼

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

2. 檢閱程式碼結構

|檔案|說明|
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions) |無伺服器行動後端的 {{site.data.keyword.openwhisk_short}} 動作程式碼|
| [**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android) |行動應用程式的程式碼|
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh) |用來安裝、解除安裝、更新 {{site.data.keyword.openwhisk_short}} 觸發程式、動作、規則的 Helper Script|
{: java}

|檔案|說明|
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions) |無伺服器行動後端的 {{site.data.keyword.openwhisk_short}} 動作程式碼|
| [**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp) |行動應用程式的程式碼|
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh) |用來安裝、解除安裝、更新 {{site.data.keyword.openwhisk_short}} 觸發程式、動作、規則的 Helper Script|
{: swift}

## 佈建服務以處理使用者鑑別、意見持續性和分析
{: #provision_services}

在此區段中，您將佈建應用程式使用的服務。可以選擇透過 {{site.data.keyword.Bluemix_notm}} 型錄或使用 `ibmcloud` 指令行來佈建服務。

建議建立新的空間來佈建服務並部署無伺服器後端。這有助於將所有資源保存在一起。

### 透過 {{site.data.keyword.Bluemix_notm}} 型錄佈建服務

1. 移至 [{{site.data.keyword.Bluemix_notm}} 型錄](https://{DomainName}/catalog/)。
2. 使用**精簡**方案建立 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 服務。將「名稱」設定為 **serverlessfollowup-db**。
3. 使用**標準**方案建立 [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) 服務。將「名稱」設定為 **serverlessfollowup-tone**。
4. 使用**累進層級**方案建立 [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) 服務。將「名稱」設定為 **serverlessfollowup-appid**。
5. 使用**精簡**方案建立 [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) 服務。將「名稱」設定為 **serverlessfollowup-mobilepush**。

### 透過指令行佈建服務

透過指令行，執行下列指令來佈建服務並擷取其認證：

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

## 配置 {{site.data.keyword.mobilepushshort}}
{: #push_notifications}

使用者提交新的意見時，應用程式將分析此意見，並將通知送回給使用者。使用者可能已經移至其他作業，或者可能沒有啟動行動式應用程式，因此使用 Push Notifications 是與使用者進行通訊的一種好辦法。利用 {{site.data.keyword.mobilepushshort}} 服務，可以透過一個統一的 API 向 iOS 或 Android 使用者傳送通知。在此區段中，您將為目標平台配置 {{site.data.keyword.mobilepushshort}} 服務。

### 配置 Firebase Cloud Messaging (FCM)
{: java}

   1. 在 [Firebase 主控台](https://console.firebase.google.com)中，建立新的專案。將「名稱」設定為 **serverlessfollowup**。
   2. 導覽至專案**設定**
   3. 在**一般**標籤下，新增兩個應用程式：
      1. 一個將套件名稱設定為 **com.ibm.mobilefirstplatform.clientsdk.android.push**
      2. 一個應用程式的套件名稱設定為：**serverlessfollowup.app**
   4. 從 Firebase 主控台下載包含兩個已定義應用程式的 `google-services.json`，然後將此檔案放入移出目錄的 `android/app` 資料夾中。
   5. 在 **Cloud Messaging** 標籤下找到「傳送端 ID」和「伺服器金鑰」（後面也稱為 API 金鑰）。
   6. 在 Push Notifications 服務儀表板中，設定「傳送端 ID 」和「API 金鑰」的值。
   {: java}

### 配置 Apple 推送通知服務 (APNs)
{: swift}

1. 移至 [Apple Developer](https://developer.apple.com/) 入口網站，並登錄應用程式 ID。
2. 建立開發及配送 APNs SSL 憑證。
3. 建立開發佈建設定檔。
4. 在 {{site.data.keyword.Bluemix_notm}} 上配置 {{site.data.keyword.mobilepushshort}} 服務實例。如需詳細步驟，請參閱[取得 APNs 認證和配置 {{site.data.keyword.mobilepushshort}} 服務](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-)。
{: swift}

## 部署無伺服器後端
{: #serverless_backend}

配置完所有服務後，現在可以部署無伺服器後端。在此區段中，會建立下列 {{site.data.keyword.openwhisk_short}} 構件：

| 構件 |類型|說明|
| ----------------------------- | ------------------------------ | ---------------------------------------- |
| `serverlessfollowup`          |套件|用於對動作進行分組並保留所有服務認證的套件|
| `serverlessfollowup-cloudant` |套件連結|連結至內建 {{site.data.keyword.cloudant_short_notm}} 套件|
| `serverlessfollowup-push`     |套件連結|連結至 {{site.data.keyword.mobilepushshort}} 套件|
| `auth-validate`               |動作|驗證存取記號和識別記號|
| `users-add`                   |動作|持續保存使用者資訊（ID、姓名、電子郵件、照片、裝置 ID）|
| `users-prepare-notify`        |動作|設定訊息格式以用於 {{site.data.keyword.mobilepushshort}} |
| `feedback-put`                |動作|將使用者意見儲存在資料庫中|
| `feedback-analyze`            |動作|透過 {{site.data.keyword.toneanalyzershort}} 分析意見|
| `users-add-sequence`          |序列公開為 Web 動作|`auth-validate` 和 `users-add`|
| `feedback-put-sequence`       |序列公開為 Web 動作| `auth-validate` 及 `feedback-put`       |
| `feedback-analyze-sequence`   |序列|透過 {{site.data.keyword.cloudant_short_notm}} 執行 `read-document`，然後透過 {{site.data.keyword.mobilepushshort}} 執行 `feedback-analyze`、`users-prepare-notify` 和 `sendMessage`|
| `feedback-analyze-trigger`    |觸發程式|意見儲存在資料庫中時由 {{site.data.keyword.openwhisk_short}} 呼叫|
| `feedback-analyze-rule`       |規則|將 `feedback-analyze-trigger` 觸發程式與 `feedback-analyze-sequence` 序列鏈結在一起|

### 編譯程式碼
{: java}
1. 從移出目錄的根目錄，編譯動作碼
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### 配置及部署動作
{: java}

2. 將 template.local.env 複製到 local.env

   ```sh
   cp template.local.env local.env
   ```
3. 從 {{site.data.keyword.Bluemix_notm}} 儀表板（或我們之前執行之 ibmcloud 指令的輸出）取得 {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.toneanalyzershort}}、{{site.data.keyword.mobilepushshort}}
 及 {{site.data.keyword.appid_short}} 服務的認證，並將 `local.env` 中的位置保留元取代為對應值。這些內容將注入到套件中，以便所有動作可以取得資料庫的存取權。
4. 將動作部署至 {{site.data.keyword.openwhisk_short}}。`deploy.sh` 會載入 `local.env` 中的認證，以建立 {{site.data.keyword.cloudant_short_notm}} 資料庫（使用者、意見回饋和心情）並部署應用程式的 {{site.data.keyword.openwhisk_short}} 構件。
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   完成指導教學之後，您可以使用 `./deploy.sh --uninstall` 移除 {{site.data.keyword.openwhisk_short}} 構件。
   {: tip}

### 配置及部署動作
{: swift}

1. 將 template.local.env 複製到 local.env
   ```sh
   cp template.local.env local.env
   ```
2. 從 {{site.data.keyword.Bluemix_notm}} 儀表板（或我們之前執行之 ibmcloud 指令的輸出）取得 {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.toneanalyzershort}}、{{site.data.keyword.mobilepushshort}}
 及 {{site.data.keyword.appid_short}} 服務的認證，並將 `local.env` 中的位置保留元取代為對應值。這些內容將注入到套件中，以便所有動作可以取得資料庫的存取權。
3. 將動作部署至 {{site.data.keyword.openwhisk_short}}。`deploy.sh` 會載入 `local.env` 中的認證，以建立 {{site.data.keyword.cloudant_short_notm}} 資料庫（使用者、意見回饋和心情）並部署應用程式的 {{site.data.keyword.openwhisk_short}} 構件。

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   完成指導教學之後，您可以使用 `./deploy.sh --uninstall` 移除 {{site.data.keyword.openwhisk_short}} 構件。
   {: tip}

## 配置並執行原生行動應用程式以收集使用者意見
{: #mobile_app}

{{site.data.keyword.openwhisk_short}} 動作已準備好用於行動式應用程式。在執行行動式應用程式之前，您需要配置其設定，以將您建立的服務設定為目標。

1. 透過 Android Studio，開啟位於移出目錄的 `android` 資料夾中的專案。
2. 編輯 `android/app/src/main/res/values/credentials.xml`，並在空白處填入認證中的值。您將需要 {{site.data.keyword.appid_short}} `tenantId`、{{site.data.keyword.mobilepushshort}} `appGuid` 和 `clientSecret`，以及在其中部署了 {{site.data.keyword.openwhisk_short}} 的組織和空間的名稱。對於 API 主機，啟動命令提示字元或終端機，並執行以下指令：
   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
3. 開啟 `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` 和 `LoginAndRegistrationListener.java`，並根據在其中建立服務實例的位置，更新推送通知服務 (BMSClient) 地區和 AppID 地區。
4. 建置專案。
5. 在真實際裝置上或使用模擬器啟動應用程式。
   要使模擬器接收推送通知，請確保使用 Google API 來選取影像，然後在模擬器中使用 Google 帳戶登入。
   {: tip}
6. 在背景監控 {{site.data.keyword.openwhisk_short}}
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. 在應用程式中，選取**登入**，以向 Facebook 或 Google 帳戶進行鑑別。登入之後，鍵入意見回饋訊息，然後按下**傳送意見回饋**按鈕。傳送意見回饋之後幾秒鐘，您應該會在裝置上收到推送通知。通知文字可以加以自訂，方法是在 {{site.data.keyword.cloudant_short_notm}} 服務實例中的 `moods` 資料庫修改範本文件。請使用**檢視記號**按鈕，檢查登入時 {{site.data.keyword.appid_short}} 產生的存取及識別記號。
{: java}


1. Push 用戶端 SDK 和其他 SDK 在 CocoaPods 和 Carthage 上可用。對於此解決方案，將使用 CocoaPods。
2. 開啟終端機，並透過 `cd ` 進入 `followupapp` 資料夾。執行以下指令安裝必要的相依關係。
   ```sh
   pod install
   ```
   {: pre}
3. 開啟位於移出目錄的 `followupapp` 資料夾下副檔名為 `.xcworkspace` 的檔案，以在 Xcode 中啟動程式碼。
4. 編輯 `BMSCredentials.plist` 檔案，並在空白處填入認證中的值。您將需要 {{site.data.keyword.appid_short}} `tenantId`、{{site.data.keyword.mobilepushshort}} `appGuid` 和 `clientSecret`，以及在其中部署了 {{site.data.keyword.openwhisk_short}} 的組織和空間的名稱。對於 API 主機，啟動終端機並執行以下指令：

   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
5. 開啟 `AppDelegate.swift`，並根據在其中建立服務實例的位置，更新推送通知服務 (BMSClient) 地區和 AppID 地區。
6. 建置專案。
7. 在真實際裝置上或使用模擬器啟動應用程式。
8. 透過在終端機上執行以下指令，在背景監控 {{site.data.keyword.openwhisk_short}}。
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. 在應用程式中，選取**登入**，以向 Facebook 或 Google 帳戶進行鑑別。登入之後，鍵入意見回饋訊息，然後按下**傳送意見回饋**按鈕。傳送意見回饋之後幾秒鐘，您應該會在裝置上收到推送通知。通知文字可以加以自訂，方法是在 {{site.data.keyword.cloudant_short_notm}} 服務實例中的 `moods` 資料庫修改範本文件。請使用**檢視記號**按鈕，檢查登入時 {{site.data.keyword.appid_short}} 產生的存取及識別記號。
{: swift}

## 移除資源

1. 使用 `deploy.sh` 來移除 {{site.data.keyword.openwhisk_short}} 構件：

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. 透過 {{site.data.keyword.Bluemix_notm}} 主控台，刪除 {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.appid_short}}、{{site.data.keyword.mobilepushshort}} 和 {{site.data.keyword.toneanalyzershort}} 服務。

## 相關內容

* {{site.data.keyword.appid_short}} 提供了預設配置，可協助起始設定身分提供者。在發佈應用程式之前，請[將配置更新為您自己的認證](https://{DomainName}/docs/services/appid?topic=appid-social#social)。您還可以[自訂登入小組件](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget)。


* 使用 Swift 來源檔案（`actions` 資料夾下的 .swift 檔案）建立 OpenWhisk Swift 動作時，必須將其編譯為二進位檔後，才能執行該動作。 完成後，對該動作的後續呼叫會快得多，直到清除保存該動作的容器為止。此延遲稱為冷啟動延遲。
  若要避免冷啟動延遲，可以將 Swift 檔案編譯為二進位檔，然後將其以 zip 檔案格式上傳到 OpenWhisk。由於您需要 OpenWhisk 支架，因此建立二進位檔的最簡單方法是在其執行所在的相同環境中進行建置。如需進一步的步驟，請參閱[將動作包裝為 Swift 執行檔](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions)。
{: swift}
