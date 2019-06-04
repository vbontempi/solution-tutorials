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

# 使用 Push Notifications 的 iOS 行動應用程式
{: #ios-mobile-push-analytics}

瞭解如何輕而易舉地快速建立使用 {{site.data.keyword.Bluemix_short}} 上高價值行動服務 {{site.data.keyword.mobilepushshort}} 的 iOS Swift 應用程式。

本指導教學會引導您建立行動入門範本應用程式、新增行動服務、設定用戶端 SDK、將程式碼匯入到 Xcode，然後進一步增強應用程式。
{:shortdesc: .shortdesc}

## 目標
{:#objectives}

- 從基本 Swift 入門範本套件建立使用 {{site.data.keyword.mobilepushshort}} 和 {{site.data.keyword.mobileanalytics_short}} 服務的行動應用程式。
- 取得 APNs 認證並配置 {{site.data.keyword.mobilepushshort}} 服務實例。
- 下載程式碼並設定用戶端 SDK。
- 傳送和監視 {{site.data.keyword.mobilepushshort}}。

  ![](images/solution6/Architecture.png)

## 產品
{:#products}

本指導教學使用下列產品：
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 開始之前
{: #prereqs}

1. [Apple Developer](https://developer.apple.com/) 帳戶，用於從 {{site.data.keyword.Bluemix_short}} 上的 {{site.data.keyword.mobilepushshort}} 服務實例（提供者）將遠端通知傳送到 iOS 裝置和應用程式。
2. Xcode，用於匯入程式碼和增強程式碼。

## 從基本 Swift 入門範本套件建立行動應用程式
{: #get_code}

1. 導覽至[行動儀表板](https://{DomainName}/developer/mobile/dashboard)以從預先定義的`入門範本套件`建立`應用程式`。
2. 按一下**入門範本套件**，然後向下捲動以選取**基本**入門範本套件。
    ![](images/solution6/mobile_dashboard.png)
3. 輸入應用程式名稱，這也將是 Xcode 專案和應用程式名稱。
4. 選取 `iOS Swift` 作為平台，然後按一下**建立**。
    ![](images/solution6/create_mobile_project.png)
5. 按一下**新增資源** > 行動 > **Push Notifications**，然後選取您要佈建服務的位置、資源群組及**精簡**定價方案。
6. 按一下**建立**以佈建 {{site.data.keyword.mobilepushshort}} 服務。新的應用程式將建立在**應用程式**標籤下。

​      **附註：** {{site.data.keyword.mobilepushshort}} 服務應該已新增有「空白入門範本」。

## 下載程式碼並設定用戶端 SDK
{: #download_code}

如果尚未下載程式碼，請按一下「應用程式」> `您的行動應用程式`下的`下載程式碼`。下載的程式碼隨附包含 **{{site.data.keyword.mobilepushshort}}** 用戶端 SDK。用戶端 SDK 在 CocoaPods 和 Carthage 上可用。對於此解決方案，將使用 CocoaPods。

1. 若要在機器上安裝 CocoaPods，請開啟`終端機`，然後執行以下指令。
   ```
  sudo gem install cocoapods
  ```
   {: pre:}
2. 解壓縮下載的程式碼，然後使用終端機，導覽至解壓縮的資料夾。

   ```
  cd <Name of Project>
  ```
   {: pre:}
3. 該資料夾已經包含 `podfile` 及其必要的相依關係。執行以下指令安裝相依關係（用戶端 SDK），這將安裝必要的相依關係。

  ```
  pod install
  ```
  {: pre:}

## 取得 APNs 認證並配置 {{site.data.keyword.mobilepushshort}} 服務實例。
{: #obtain_apns_credentials}

   對於 iOS 裝置和應用程式，Apple Push Notification Service (APNs) 容許應用程式開發人員從 {{site.data.keyword.Bluemix_short}} 上的 {{site.data.keyword.mobilepushshort}} 服務實例（提供者）將遠端通知傳送到 iOS 裝置和應用程式。訊息會傳送到裝置上的目標應用程式。

   您需要取得並配置 APNs 認證。APNs 憑證由 {{site.data.keyword.mobilepushshort}} 服務安全地進行管理，並用於連接至 APNs 伺服器（提供者）。

### 登錄 App ID

   App ID（組合 ID）是用於識別特定應用程式的唯一 ID。每個應用程式都需要 App ID。{{site.data.keyword.mobilepushshort}} 服務這類服務是配置給 App ID。確保您具有 [Apple Developer](https://developer.apple.com/) 帳戶。這是必備的先決條件。

   1. 移至 [Apple Developer](https://developer.apple.com/) 入口網站，並按一下 `Member Center`，然後選取 `Certificates, IDs & Profiles`。
   2. 移至 `Identifiers` > App IDs 區段。
   3. 在 `Registering App IDs` 頁面中，於 App ID Description 的 Name 欄位中，提供應用程式名稱。例如：ACME {{site.data.keyword.mobilepushshort}}。為 App ID Prefix 提供字串。
   4. 對於 App ID Suffix，選擇 `Explicit App ID`，並提供 Bundle ID 值。建議提供反向網域名稱樣式字串。例如：com.ACME.push。
   5. 選取 `{{site.data.keyword.mobilepushshort}}` 勾選框，然後按一下`Continue`。
   6. 仔細檢查設定，然後按一下 `Register` > `Done`。現在，您的 App ID 已登錄。

     ![](images/solution6/push_ios_register_appid.png)

### 建立開發和分散 APNs SSL 憑證
   在取得 APNs 憑證之前，必須先產生一個憑證簽署要求 (CSR)，然後將其提交給 Apple（憑證管理中心 (CA)）。CSR 包含用於識別您公司的資訊以及用於對 Apple {{site.data.keyword.mobilepushshort}} 簽署的公開金鑰和私密金鑰。然後，在 iOS 開發人員入口網站上產生 SSL 憑證。該憑證與其公開金鑰和私密金鑰一起儲存在「金鑰鏈存取」中。您可以在兩種模式下使用 APNs：

- 沙盤推演模式，用於開發和測試。
- 正式作業模式，用於透過 App Store（或其他企業分散機制）分散應用程式。

   必須分別針對開發環境和分散環境取得不同的憑證。憑證與接收遠端通知的應用程式的 App ID 相關聯。對於正式作業環境，最多可建立兩個憑證。{{site.data.keyword.Bluemix_short}} 使用憑證與 APNs 建立 SSL 連線。

   1. 移至 Apple Developer 網站，並按一下 **Member Center**，然後選取 **Certificates, IDs & Profiles**。
   2. 在 **Identifiers** 區域中，按一下 **App IDs**。
   3. 從 App ID 清單中，選取您的 App ID，然後選取 `Edit`。
   4. 選取 **{{site.data.keyword.mobilepushshort}}** 勾選框，然後在 **Development SSL certificate** 窗格上，按一下 **Create Certificate**。

     ![Push Notification SSL 憑證](images/solution6/certificate_createssl.png)

   5. 顯示 **About Creating a Certificate Signing Request (CSR)** 畫面時，遵循顯示的指示建立憑證簽署要求 (CSR) 檔案。提供有意義的名稱來協助您識別這是開發（沙盤推演）憑證還是配送（正式作業）憑證；例如，sandbox-apns-certificate 或 production-apns-certificate。
   6. 按一下 **Continue**，在 Upload CSR file 畫面上，按一下 **Choose File**，然後選取剛才建立的 **CertificateSigningRequest.certSigningRequest** 檔案。按一下**繼續**。
   7. 在 Download, Install and Backup 窗格中，按一下 Download。這將下載 **aps_development.cer** 檔案。
        ![下載憑證](images/solution6/push_certificate_download.png)

        ![產生憑證](images/solution6/generate_certificate.png)
   8. 在 Mac 上，開啟 **Keychain Access**、**File**、**Import**，選取下載的 .cer 檔案以進行安裝。
   9. 以滑鼠右鍵按一下新的憑證和私密金鑰，然後選取 **Export**，並將 **File Format** 變更為「個人資訊交換」格式（`.p12` 格式）。
     ![匯出憑證和金鑰](images/solution6/keychain_export_key.png)
   10. 在 **Save As** 欄位中，為憑證提供有意義的名稱。例如，`sandbox_apns.p12` 或 **production_apns.p12**，然後按一下 Save。
     ![匯出憑證和金鑰](images/solution6/certificate_p12v2.png)
   11. 在 **Enter a password** 欄位中，輸入用於保護匯出項目的密碼，然後按一下 OK。可以使用此密碼在 {{site.data.keyword.mobilepushshort}} 服務控制台上配置 APNs 設定。
       ![匯出憑證和金鑰](images/solution6/export_p12.png)
   12. **Key Access.app** 會提示您從 **Keychain** 畫面匯出金鑰。輸入 Mac 的管理員密碼，以容許系統匯出這些項目，然後選取 `Always Allow` 選項。這將在桌面上產生一個 `.p12` 憑證。

      對於「正式作業 SSL」，在 **Production SSL certificate** 窗格中，按一下 **Create Certificate**，然後重複上面的步驟 5 到 12。
      {:tip}

### 建立開發佈建設定檔
   佈建設定檔與 App ID 一起用於確定哪些裝置可以安裝並執行您的應用程式，以及您的應用程式可以存取哪些服務。對於每個 App ID，都可建立兩個佈建設定檔：一個用於開發，另一個用於分散。Xcode 使用開發佈建設定檔來確定容許哪些開發人員建置應用程式，以及容許在應用程式上測試哪些裝置。

   確保您已執行下列作業：登錄 App ID，針對 {{site.data.keyword.mobilepushshort}} 服務已啟用該 ID，然後將其配置為使用開發和正式作業 APNs SSL 憑證。

   建立開發佈建設定檔，如下所示：

   1. 移至 [Apple Developer](https://developer.apple.com/) 入口網站，並按一下 `Member Center`，然後選取 `Certificates, IDs & Profiles`。
   2. 移至 [Mac Developer Library](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site)，並捲動到 `Creating Development Provisioning Profiles` 區段，然後遵循指示建立開發設定檔。
     **附註**：配置開發佈建設定檔時，請選取下列選項：

     - **iOS App Development**
     - **For iOS and watchOS apps**

### 建立市集分散佈建設定檔
   使用市集佈建設定檔可提交應用程式以分散到 App Store。

   1. 移至 [Apple Developer](https://developer.apple.com/)，並按一下 `Member Center`，然後選取 `Certificates, IDs & Profiles`。
   2. 按兩下下載的佈建設定檔，以將其安裝到 Xcode 中。取得認證後，下一步是[配置服務實例](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2)。

### 配置服務實例

   若要使用 {{site.data.keyword.mobilepushshort}} 服務來傳送通知，請上傳您在上述步驟建立的 .p12 憑證。此憑證包含私密金鑰以及建置和發佈應用程式所需的 SSL 憑證。

   **附註**：在 `.cer` 檔案出現在「金鑰鏈存取」中之後，請將其匯出到您的電腦，以建立 `.p12` 憑證。

1. 按一下「服務」區段下的 {{site.data.keyword.mobilepushshort}}，或按一下 {{site.data.keyword.mobilepushshort}} 服務旁邊的三個垂直點，然後選取`開啟儀表板`。
2. 在 {{site.data.keyword.mobilepushshort}} 儀表板上，您應該會在`管理`下看到`配置`選項。

若要在 `Push Notification 服務`主控台上設定 APNs，請完成以下步驟：

1. 選擇`行動`選項，以更新 APNs 推送認證表單中的資訊。
2. 根據需要選取`沙盤推演/開發 APNs 伺服器`或`正式作業 APNs 伺服器`，然後上傳已建立的 `.p12` 憑證。
3. 在「密碼」欄位中，輸入與 .p12 憑證檔案相關聯的密碼，然後按一下「儲存」。

![](images/solution6/Mobile_push_configure.png)

## 配置、傳送並監視 {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. 可以在 `AppDelegate.swift` 中找到 Push 起始設定碼（在`函數應用程式`下）和通知登錄程式碼。提供唯一的 USER_ID（選用）。
2. 在實體裝置上執行應用程式，因為通知無法傳送到 iPhone 模擬器。
3. 在 {{site.data.keyword.Bluemix_short}} 行動儀表板的`行動服務` > **現有服務**下開啟 {{site.data.keyword.mobilepushshort}} 服務，若要傳送基本 {{site.data.keyword.mobilepushshort}}，請完成下列步驟：
  * 選取`訊息`，然後選擇「傳送至」選項來編寫訊息。支援的選項包括「依標籤的裝置」、「裝置 ID」、「使用者 ID」、「Android 裝置」、「iOS 裝置」、「Web 通知」、「所有裝置」和「其他瀏覽器」。

       **附註**：當您選取「所有裝置」選項時，所有已訂閱 {{site.data.keyword.mobilepushshort}} 的裝置都會收到通知。
  * 在`訊息`欄位中，編寫訊息。視需要選擇配置選用設定。
  * 按一下`傳送`，然後驗證實體裝置是否收到通知。
    ![](images/solution6/send_notifications.png)

4. 您應該會在 iPhone 上看到通知。

   ![](images/solution6/iphone_notification.png)

5. 可以藉由在 {{site.data.keyword.mobilepushshort}} 服務上導覽至`監視`來監視傳送的通知。

IBM {{site.data.keyword.mobilepushshort}} 服務現在已擴充功能，可藉由從您的使用者資料產生圖形，來監視推送效能。您可以使用公用程式來列出所有傳送的 {{site.data.keyword.mobilepushshort}}，或是列出所有已登錄裝置，及每日、每週或每月分析資訊。
 ![](images/solution6/monitoring_messages.png)

## 相關內容
{: #related_content}

- [以標籤為基礎的通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}} 中的安全](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)
