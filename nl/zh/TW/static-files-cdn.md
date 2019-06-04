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

# 使用 CDN 加速交付靜態檔案
{: #static-files-cdn}

本指導教學將全程指導您如何在 {{site.data.keyword.cos_full_notm}} 中管理和提供網站資產（影像、視訊和文件）和使用者產生的內容，以及如何使用 [{{site.data.keyword.cdn_full}} (CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai) 快速、安全地向世界各地的使用者交付內容。

Web 應用程式具有不同類型的內容：HTML 內容、影像、視訊、階式樣式表、JavaScript 檔案以及使用者產生的內容。有些內容經常變化，有些內容則變化不大，有些內容經常被很多使用者存取，有些則是偶爾才會被存取。隨著應用程式的受眾增長，您可能希望將提供這些內容的工作卸載到其他元件，以便能為主要應用程式釋放資源。您可能還希望從距離應用程式使用者很近的位置來提供這些內容，無論使用者身處全球哪個位置。

為什麼要在這些情境中使用 Content Delivery Network 的原因有很多：
* CDN 將對內容進行快取，並且僅當內容在其快取中無法使用或已到期時，才會從原點（您的伺服器）中取回內容；
* 利用遍佈全球的多個資料中心，CDN 將為使用者提供距離最近的位置中的快取內容；
* 瀏覽器在不同於主要應用程式的網域上執行，因此能夠平行載入其他內容 - 大多數瀏覽器對每個主機名稱的連線數有限制。

## 目標
{: #objectives}

* 將檔案上傳到 {{site.data.keyword.cos_full_notm}} 儲存區。
* 透過 Content Delivery Network (CDN) 使內容全球可用。
* 使用 Cloud Foundry Web 應用程式公開檔案。

## 使用的服務
{: #services}

本指導教學使用下列產品：
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構

<p style="text-align: center;">
![架構](images/solution3/Architecture.png)
</p>

1. 使用者存取應用程式
2. 應用程式包含透過 Content Delivery Network 配送的內容
3. 如果內容在 CDN 中無法使用或已到期，CDN 將從原點中取回內容。

## 開始之前
{: #prereqs}

請聯絡基礎架構帳戶的主要使用者來取得下列許可權：
   * 管理 CDN 帳戶
   * 管理儲存空間
   * API 金鑰

需要這些許可權才能檢視和使用儲存空間和 CDN 服務。

## 取得 Web 應用程式碼
{: #get_code}

假設有一個簡單的 Web 應用程式，具有不同類型的內容，如影像、視訊和級聯樣式表。您將內容儲存在儲存空間儲存區中，並將 CDN 配置為使用該儲存區作為其原始儲存區。

請先擷取應用程式碼：

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## 建立 Object Storage
{: #create_cos}

{{site.data.keyword.cos_full_notm}} 為非結構化資料提供了有彈性、具成本效益且可擴充的雲端儲存空間。

1. 移至主控台中的[型錄](https://{DomainName}/catalog/)，然後從「儲存空間」區段中，選取 [**Object Storage**](https://{DomainName}/catalog/services/cloud-object-storage)。
2. 建立新的 {{site.data.keyword.cos_full_notm}} 實例。
4. 在服務儀表板中，按一下**建立儲存區**。
5. 設定唯一儲存區名稱，如 `username-mywebsite`，然後按一下**建立**。避免在儲存區名稱中使用點 (.)。

## 將檔案上傳到儲存區
{: #upload}

在此區段中，會使用指令行工具 **curl** 將檔案上傳到儲存區。

1. 在 CLI 中登入到 {{site.data.keyword.Bluemix_notm}}，然後從 IBM Cloud IAM 中取得記號。
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. 複製上一步指令輸出中的記號。
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. 將記號和儲存區名稱的值設定為環境變數，以方便存取。
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. 從先前下載的 Web 應用程式碼的內容目錄中，上傳名為 `a-css-file.css`、`a-picture.png` 和 `a-video.mp4` 的檔案。將這些檔案上傳到儲存區的根目錄。
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. 在儀表板中檢視這些檔案。
   ![](images/solution3/Buckets.png)
6. 透過瀏覽器使用類似於下列範例的鏈結來存取這些檔案：

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## 透過 CDN 使檔案全球可用

在此區段中，您將建立 CDN 服務。CDN 服務將內容配送到需要該內容的位置。首次收到內容要求時，該服務會將內容從主機伺服器（{{site.data.keyword.cos_full_notm}} 中的儲存區）中取回到網路並將其保留在網路中，以供其他使用者快速存取，而不會有再次存取主機伺服器而產生的網路延遲。

### 建立 CDN 實例

1. 移至主控台中的型錄，然後從「網路」區段中，選取 **Content Delivery Network**。此 CDN 採用 Akamai 技術。按一下**建立**。
2. 在下一個對話框中，將 CDN 的**主機名稱**設定為自訂網域。雖然設定的是自訂網域，但仍可以透過 IBM 提供的 CNAME 來存取 CDN 內容。因此，如果不打算使用自訂網域，您可以設定任意名稱。
3. 將**自訂 CNAME** 字首設定為唯一值。
4. 接下來，在**配置原點**下，選取 **Object Storage** 以便為 COS 配置 CDN。
5. 將**端點**設定為儲存區 API 端點，例如 *s3-api.us-geo.objectstorage.softlayer.net*。
6. 將**主機標頭**和**路徑**保留為空。將**儲存區名稱**設定為 *your-bucket-name*。
7. 啟用 HTTP (80) 和 HTTPS (443) 埠。
8. 對於 **SSL 憑證**，如果要使用自訂網域，請選取 *DV SAN 憑證*。或者，若要透過 CNAME 存取儲存空間，請選取「萬用字元憑證」選項。
9. 按一下**建立**。

### 透過 CDN CNAME 存取內容

1. [在清單中](https://{DomainName}/classic/network/cdn)選取 CDN 實例。
2. 如果先前選取了 *DV SAN 憑證*，則在起始設定完成後，系統將提示您進行網域驗證。請遵循按一下**檢視網域驗證**時顯示的步驟。
3. **詳細資料**畫面顯示了 CDN 的**主機名稱**和 **CNAME**。
4. 使用 `https://your-cdn-cname.cdnedge.bluemix.net/a-picture.png` 存取檔案，或者如果使用的是自訂網域，請使用 `https://your-cdn-hostname/a-picture.png` 進行存取。如果省略檔名，您應該會看到 S3 ListBucketResult。

## 部署 Cloud Foundry 應用程式

應用程式包含 public/index.html 網頁，其中含有對現在 {{site.data.keyword.cos_full_notm}} 中所管理檔案的參照。後端 `app.js` 提供此網頁，並將位置保留元取代為 CDN 的實際位置。這樣，該網頁使用的所有資產都將由 CDN 提供。

1. 從終端機，進入移出程式碼的目錄。
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. 推送應用程式而不啟動它。
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. 配置 CDN_NAME 環境變數，以便應用程式可以參照 CDN 內容。
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. 啟動應用程式。
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. 使用 Web 瀏覽器存取應用程式，頁面樣式表、圖片和視訊將從 CDN 載入。

![](images/solution3/Application.png)

將 CDN 與 {{site.data.keyword.cos_full_notm}} 搭配使用是一種功能強大的組合，可讓您管理檔案，並將其提供給世界各地的使用者。此外，還可以使用 {{site.data.keyword.cos_full_notm}} 來儲存使用者上傳到應用程式的任何檔案。

## 移除資源

* 刪除 Cloud Foundry 應用程式
* 刪除 Content Delivery Network 服務
* 刪除 {{site.data.keyword.cos_full_notm}} 服務或儲存區

## 相關內容

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[管理對 {{site.data.keyword.cos_full_notm}} 的存取](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[CDN 開始使用](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
