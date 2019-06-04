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

# 具有推送通知的 Android 原生行動應用程式
{: #android-mobile-push-analytics}

瞭解如何輕鬆地在 {{site.data.keyword.Bluemix_notm}} 上快速建立具有 {{site.data.keyword.mobilepushshort}} 之類的高價值行動服務的原生 Android 應用程式。

本指導教學會引導您建立行動入門範本應用程式、新增行動服務、設定用戶端 SDK、將程式碼匯入 Android Studio，然後進一步增強應用程式。

## 目標
{: #objectives}

* 建立具有 {{site.data.keyword.mobilepushshort}} 服務的行動應用程式。
* 取得 FCM 認證。
* 下載程式碼並完成必要的設定。
* 配置、傳送並監視 {{site.data.keyword.mobilepushshort}}。

![](images/solution9/Architecture.png)

## 產品
{: #products}

本指導教學使用下列產品：
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 開始之前
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html)，用來匯入及加強程式碼。
- Google 帳戶，用來登入 Firebase 主控台以取得傳送端 ID 和伺服器 API 金鑰。

## 從入門範本套件建立 Android 行動應用程式
{: #get_code}
{{site.data.keyword.Bluemix_notm}} 行動儀表板容許您藉由從入門範本套件建立應用程式，來加快行動應用程式開發。
1. 導覽至[行動儀表板](https://{DomainName}/developer/mobile/dashboard)
2. 按一下**入門範本套件**和**建立應用程式**。![](images/solution9/mobile_dashboard.png)
3. 輸入應用程式名稱，這也可以是您的 Android 專案名稱。
4. 選取 **Android** 作為平台，並按一下**建立**。

    ![](images/solution9/create_mobile_project.png)
5. 按一下**新增資源** > 行動 > **Push Notifications**，然後選取您要佈建服務的位置、資源群組及**精簡**定價方案。
6. 按一下**建立**以佈建 {{site.data.keyword.mobilepushshort}} 服務。新的應用程式將建立在**應用程式**標籤下。

    **附註：**應該使用「空白入門範本」新增 {{site.data.keyword.mobilepushshort}} 服務。在下一步中，您將取得 Firebase Cloud Messaging (FCM) 認證。

在下一步中，您將下載支架程式碼，並設定 Push Android SDK。

## 下載程式碼並完成必要的設定
{: #download_code}

如果您尚未下載程式碼，則藉由按一下「應用程式」> **您的行動應用程式**下的**下載程式碼**按鈕，以使用 {{site.data.keyword.Bluemix_notm}} 行動儀表板取得程式碼。所下載的程式碼中包括 **{{site.data.keyword.mobilepushshort}}** 用戶端 SDK。用戶端 SDK 在 Gradle 和 Maven 上可用。對於本指導教學，您將使用 **Gradle**。

1. 啟動 Android Studio > **開啟現有 Android Studio 專案**，並指向下載的程式碼。
2. **Gradle** 建置將自動觸發，並將下載所有相依關係。
3. 將 **Google Play 服務**相依關係新增到模組層次 `build.gradle (Module: app)` 檔案結尾，即 `dependencies{.....}` 後面
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. 複製您已建立並下載到 Android 應用程式模組根目錄的 `google-services.json` 檔案。請注意，`google-service.json` 檔案包含新增的套件名稱。
5. 所需的許可權都包含在 `AndroidManifest.xml` 檔案和相依關係中。Push 和 Analytics 包含在 **build.gradle (Module: app)** 中。
6. `RECEIVE` 和 `REGISTRATION` 事件通知的 **Firebase Cloud Messaging (FCM)** 目的服務和目的過濾器包含在 `AndroidManifest.xml` 中

## 取得 FCM 認證
{: #obtain_fcm_credentials}

Firebase Cloud Messaging (FCM) 是用於將 {{site.data.keyword.mobilepushshort}} 交付到 Android 裝置、Google Chrome 瀏覽器以及 Chrome Apps & Extensions 的閘道。若要在主控台上設定 {{site.data.keyword.mobilepushshort}} 服務，您需要取得 FCM 認證（傳送端 ID 和 API 金鑰）。

API 金鑰由 {{site.data.keyword.mobilepushshort}} 服務安全地進行儲存和使用以連接至 FCM 伺服器，而傳送端 ID（專案號碼）由 Android SDK 以及用於 Google Chrome 和 Mozilla Firefox 的 JS SDK 在用戶端使用。若要設定 FCM 並取得認證，請完成以下步驟：

1. 造訪 [Firebase 主控台](https://console.firebase.google.com/?pli=1) - 需要 Google 使用者帳戶。
2. 選取**新增專案**。
3. 在**建立專案**視窗中，提供專案名稱，並選擇國家/地區，然後按一下**建立專案**。
4. 在左側的導覽窗格中，選取**設定**（按一下**概觀**旁的「設定」圖示）> **專案設定**。
5. 選擇「雲端傳訊」標籤，以取得專案認證 -「伺服器 API 金鑰」和「傳送端 ID」。**附註：**FCM 中列出的伺服器金鑰與伺服器 API 金鑰相同。![](images/solution9/fcm_console.png)

您還需要產生 `google-services.json` 檔案。請完成下列步驟：

1. 在 Firebase 主控台中，按一下上面建立的專案下的**專案設定**圖示 > **一般**標籤，並選取**將 Firebase 新增到您的 Android 應用程式**

    ![](images/solution9/firebase_project_settings.png)
2. 在**將 Firebase 新增到您的 Android 應用程式**限制模式視窗中，將 **com.ibm.mobilefirstplatform.clientsdk.android.push** 新增為套件名稱以登錄 {{site.data.keyword.mobilepushshort}} Android SDK。應用程式暱稱和 SHA-1 欄位為選用性欄位。按一下**登錄應用程式** > **繼續** > **完成**。

    ![](images/solution9/add_firebase_to_your_app.png)

3. 按一下**新增應用程式** > **將 Firebase 新增到您的應用程式**。藉由輸入套件名稱 **com.ibm.mysampleapp** 來包含應用程式的套件名稱，然後繼續將 Firebase 新增您的 Android 應用程式視窗。應用程式暱稱和 SHA-1 欄位為選用性欄位。按一下**登錄應用程式** >「繼續」>「完成」。**附註：**下載程式碼後，您可以在 `AndroidManifest.xml` 檔案中找到應用程式的套件名稱。
4. 在**您的應用程式**下，下載最新的配置檔 `google-services.json`。

    ![](images/solution9/google_services.png)

    **附註**：FCM 是 Google Cloud Messaging (GCM) 的新版本。確保對新應用程式使用 FCM 認證。現有應用程式將繼續以 GCM 配置運作。

*步驟和 Firebase 主控台使用者介面可能會有變更，如果需要，請參閱 Google 文件的 Firebase 部分*

## 配置、傳送並監視 {{site.data.keyword.mobilepushshort}}

{: #configure_push}

1. {{site.data.keyword.mobilepushshort}} SDK 已經匯入應用程式中，可以在 `MainActivity.java` 檔案中找到 Push 起始設定碼。

    **附註：**服務認證是 `/res/values/credentials.xml` 檔案的一部分。
2. 通知登錄在 `MainActivity.java` 中進行。（選用）提供唯一 USER_ID。
3. 在實體裝置或模擬器上執行應用程式以接收通知。
4. 在 {{site.data.keyword.Bluemix_notm}} 行動儀表板的**行動服務** > **現有服務**下開啟 {{site.data.keyword.mobilepushshort}} 服務，若要傳送基本 {{site.data.keyword.mobilepushshort}}，請完成下列步驟：
   - 按一下**管理** > **配置**。
   - 選取**行動**然後使用您一開始在 Firebase 主控台上建立的傳送端 ID/專案號碼和 API 金鑰（伺服器金鑰）來更新「GCM/FCM 推送認證」標籤。

     ![](images/solution9/configure_push_notifications.png)
   - 按一下**儲存**。{{site.data.keyword.mobilepushshort}} 服務現在已配置好。
   - 選取**傳送通知**，然後選擇傳送選項來編寫訊息。支援的選項包括「依標籤的裝置」、「裝置 ID」、「使用者 ID」、「Android 裝置」、「IOS 裝置」、「Web 通知」和「所有裝置」。**附註**：當您選取**所有裝置**選項時，所有已訂閱 {{site.data.keyword.mobilepushshort}} 的裝置都會收到通知。
   - 在**訊息**欄位中，編寫訊息。視需要選擇配置選用設定。
   - 按一下**傳送**，並驗證您的實體裝置已收到通知。

     ![](images/solution9/android_send_notifications.png)
5. 您應該在 Android 裝置上看到通知。

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. 您可以藉由導覽至 {{site.data.keyword.mobilepushshort}} 服務上的**監視**來監視傳送的通知。IBM {{site.data.keyword.mobilepushshort}} 服務現在已擴充功能，可藉由從您的使用者資料產生圖形，來監視推送效能。您可以使用公用程式來列出所有傳送的 {{site.data.keyword.mobilepushshort}}，或是列出所有已登錄裝置，及每日、每週或每月分析資訊。
 ![](images/solution6/monitoring_messages.png)

## 相關內容
{: #related_content}
- [自訂 {{site.data.keyword.mobilepushshort}} 設定](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [以標籤為基礎的通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}} 中的安全](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

