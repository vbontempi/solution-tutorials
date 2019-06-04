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

# 使用 Push Notifications 的混合式行動應用程式
{: #hybrid-mobile-push-analytics}

瞭解如何輕而易舉地快速建立使用 {{site.data.keyword.Bluemix_notm}} 上高價值行動服務（如 {{site.data.keyword.mobilepushshort}}）的混合式 Cordova 應用程式。

Apache Cordova 是一種開放程式碼行動開發架構。它讓您能將多種標準 Web 技術（HTML5、CSS3 和 JavaScript）用於跨平台開發。應用程式在針對每個平台的封套內執行，並依賴符合標準的 API 連結來存取每個裝置的功能，如感應器、資料、網路狀態等。

本指導教學會引導您建立 Cordova 行動入門範本應用程式、新增行動服務、設定用戶端 SDK、下載支架程式碼，然後進一步增強應用程式。

## 目標

* 建立使用 {{site.data.keyword.mobilepushshort}} 服務的行動專案。
* 瞭解如何取得 APN 和 FCM 認證。
* 下載程式碼並完成必要的設定。
* 配置、傳送並監視 {{site.data.keyword.mobilepushshort}}。

 ![](images/solution15/Architecture.png)

## 產品

本指導教學使用下列產品：
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 開始之前
{: #prereqs}

- Cordova [CLI](https://cordova.apache.org/docs/en/latest/guide/cli/)，用於執行 Cordova 指令。
- Cordova-iOS [必要條件](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html)和 Cordova-Android [必要條件](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html)
- Google 帳戶，用來登入 Firebase 主控台以取得傳送端 ID 和伺服器 API 金鑰。
- [Apple Developer](https://developer.apple.com/) 帳戶，用於從 {{site.data.keyword.Bluemix_notm}} 上的 {{site.data.keyword.mobilepushshort}} 服務實例（提供者）將遠端通知傳送到 iOS 裝置和應用程式。
- Xcode 和 Android Studio，用於匯入程式碼和進一步增強程式碼。

## 從入門範本套件建立 Cordova 行動專案
{: #get_code}
{{site.data.keyword.Bluemix_notm}} 行動儀表板容許您藉由從入門範本套件建立專案，來加快行動應用程式開發。
1. 導覽至[行動儀表板](https://{DomainName}/developer/mobile/dashboard)。
2. 按一下**入門範本套件**，然後按一下**建立應用程式**。
    ![](images/solution15/mobile_dashboard.png)
3. 輸入專案名稱，這也可以是應用程式名稱。
4. 選取 **Cordova** 作為平台，然後按一下**建立**。

    ![](images/solution15/create_cordova_project.png)
5. 按一下**新增資源** > 行動 > **Push Notifications**，然後選取您要佈建服務的位置、資源群組及**精簡**定價方案。
6. 按一下**建立**以佈建 {{site.data.keyword.mobilepushshort}} 服務。新的應用程式將建立在**應用程式**標籤下。

    **附註：** {{site.data.keyword.mobilepushshort}} 服務應該已新增到「空白入門範本」。

在下一步中，您將下載支架程式碼，並完成所需的設定。

## 下載程式碼並完成必要的設定
{: #download_code}

如果尚未下載程式碼，請使用 {{site.data.keyword.Bluemix_notm}} 行動儀表板藉由按一下「專案」> **您的行動專案**下的**下載程式碼**按鈕來取得程式碼。

1. 在所選的 IDE 中，導覽至 `/platforms/android/project.properties`，並將最後兩行（library.1 和 library.2）取代為以下各行：

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
  上面的變更是 Android 特有的。
  {: tip}
2. 若要在 Android 模擬器上啟動應用程式，請在終端機或命令提示字元中執行以下指令：
```
 $ cordova build android
 $ cordova run android
```
 如果看到錯誤，請啟動模擬器並嘗試執行上面的指令。
 {: tip}
3. 若要在 iOS 模擬器中預覽應用程式，請在終端機中執行以下指令：

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    可以藉由執行 `cordova info` 指令在 `config.xml` 檔案中尋找您的專案名稱。
    {: tip}

## 取得 FCM 和 APNs 認證
{: #obtain_fcm_apns_credentials}

 ### 配置 Firebase Cloud Messaging (FCM)

  1. 在 [Firebase 主控台](https://console.firebase.google.com)中，建立新的專案。將名稱設定為 **hybridmobileapp**
  2. 導覽至專案**設定**
  3. 在**一般**標籤下，新增兩個應用程式：
       1. 一個將套件名稱設定為 **com.ibm.mobilefirstplatform.clientsdk.android.push**
       2. 一個將套件名稱設定為 **io.cordova.hellocordovastarter**
  4. 在 **Cloud Messaging** 標籤下找到「傳送端 ID」和「伺服器金鑰」（後面也稱為 API 金鑰）。
  5. 在 {{site.data.keyword.mobilepushshort}} 服務儀表板中，設定「傳送端 ID 」和「API 金鑰」的值。


如何詳細步驟，請參閱[取得 FCM 認證](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials)。
{: tip}

### 配置 Apple {{site.data.keyword.mobilepushshort}} 服務 (APNs)

  1. 移至 [Apple Developer](https://developer.apple.com/) 入口網站，並登錄 App ID。
  2. 建立開發及配送 APNs SSL 憑證。
  3. 建立開發佈建設定檔。
  4. 在 {{site.data.keyword.Bluemix_notm}} 上配置 {{site.data.keyword.mobilepushshort}} 服務實例。

如何詳細步驟，請參閱[取得 APNs 認證和配置 {{site.data.keyword.mobilepushshort}} 服務](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials)。
{: tip}

## 配置、傳送並監視 {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. 在 index.js 的 `onDeviceReady` 函數下，將 `{pushAppGuid}` 和

   `{pushClientSecret}` 值取代為 Push 服務**認證** - *appGuid* 和 *clientSecret*。

2. 移至行動儀表板 >「專案」>「Cordova 專案」，並按一下 {{site.data.keyword.mobilepushshort}} 服務，然後執行以下步驟。

### APNs - 配置服務實例

若要使用 {{site.data.keyword.mobilepushshort}} 服務來傳送通知，請上傳您在上述步驟建立的 .p12 憑證。此憑證包含私密金鑰以及建置和發佈應用程式所需的 SSL 憑證。

**附註**：在 `.cer` 檔案出現在「金鑰鏈存取」中之後，請將其匯出到您的電腦，以建立 `.p12` 憑證。

1. 按一下「服務」區段下的 `{{site.data.keyword.mobilepushshort}}`，或按一下 {{site.data.keyword.mobilepushshort}} 服務旁邊的三個垂直點，然後選取`開啟儀表板`。
2. 在 {{site.data.keyword.mobilepushshort}} 儀表板上，您應該會在`管理 > 傳送通知`下看到`配置`選項。

若要在 `Push Notification 服務`主控台上設定 APNs，請完成以下步驟：

1. 在「Push Notification 服務」儀表板中，選取`配置`。
2. 選擇`行動`選項，以更新 APNs 推送認證表單中的資訊。
3. 根據需要選取`沙盤推演（開發）`或`正式作業（分散）`，然後上傳已建立的 `p.12` 憑證。
4. 在「密碼」欄位中，輸入與 .p12 憑證檔案相關聯的密碼，然後按一下「儲存」。

### FCM - 配置服務實例

1. 選取**行動**然後使用您一開始在 Firebase 主控台上建立的傳送端 ID/專案號碼和 API 金鑰（伺服器金鑰）來更新「GCM/FCM 推送認證」標籤。
2. 按一下**儲存**。{{site.data.keyword.mobilepushshort}} 服務現在已配置好。

### 傳送 {{site.data.keyword.mobilepushshort}}

1. 選取**傳送通知**，然後選擇傳送選項來編寫訊息。支援的選項包括「依標籤的裝置」、「裝置 ID」、「使用者 ID」、「Android 裝置」、「IOS 裝置」、「Web 通知」和「所有裝置」。
**附註**：當您選取**所有裝置**選項時，所有已訂閱 {{site.data.keyword.mobilepushshort}} 的裝置都會收到通知。

2. 在**訊息**欄位中，編寫訊息。視需要選擇配置選用設定。
3. 按一下**傳送**，並驗證您的實體裝置已收到通知。

### 監視傳送的通知

可以藉由導覽至**監視**區段來監視傳送的通知。IBM {{site.data.keyword.mobilepushshort}} 服務現在已擴充功能，可藉由從您的使用者資料產生圖形，來監視推送效能。您可以使用公用程式來列出所有傳送的 {{site.data.keyword.mobilepushshort}}，或是列出所有已登錄裝置，及每日、每週或每月分析資訊。
 ![](images/solution6/monitoring_messages.png)

## 相關內容
{: #related_content}

- [以標籤為基礎的通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}} 中的安全](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

