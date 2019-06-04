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


# 無伺服器 Web 應用程式和 API
{: #serverless-api-webapp}

在本指導教學中，您將透過在 GitHub 頁面上管理靜態網站內容，並使用 {{site.data.keyword.openwhisk}} 實作應用程式後端來建立無伺服器 Web 應用程式。

{{site.data.keyword.openwhisk_short}} 作為一個事件驅動型平台，支援[各種使用案例](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)。建置 Web 應用程式和 API 就是其中一例。對於 Web 應用程式，事件是指 Web 瀏覽器（或 REST 用戶端）和 Web 應用程式與 HTTP 要求之間的互動。您可以不佈建虛擬機器、容器或 Cloud Foundry 運行環境來部署後端，而改為使用無伺服器平台實作後端 API。這會是一個不錯的解決方案，可以避免為閒置時間付款，並讓平台根據需要調整規模。

{{site.data.keyword.openwhisk_short}} 中的任何動作（或函數）都可以轉換為可供 Web 用戶端使用的現成 HTTP 端點。已啟用 Web 時，這些動作稱為 *Web 動作*。擁有 Web 動作後，可以透過 API 閘道將這些動作組合成全功能 API。API 閘道是 {{site.data.keyword.openwhisk_short}} 的一個元件，用於公開 API。它具有安全、OAuth 支援、速率限制、自訂網域支援等功能。

## 目標

* 部署無伺服器後端和資料庫
* 公開 REST API
* 管理靜態網站
* 選用：將自訂網域用於 REST API

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

本指導教學中說明的應用程式是一個簡單的留言板網站，使用者可以在其中公佈訊息。

<p style="text-align: center;">

   ![架構](./images/solution8/Architecture.png)
</p>

1. 使用者存取在 GitHub Pages 中管理的應用程式。
2. 該 Web 應用程式呼叫後端 API。
3. 後端 API 在 API 閘道中進行定義。
4. API 閘道將要求轉遞給 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)。
5. {{site.data.keyword.openwhisk_short}} 動作使用 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 來儲存和擷取留言板項目。

## 開始之前
{: #prereqs}

本手冊使用 GitHub Pages 來管理靜態網站。因此，請確保您有公用 GitHub 帳戶。

## 建立 Guestbook 資料庫

請先建立 {{site.data.keyword.cloudant_short_notm}}。{{site.data.keyword.cloudant_short_notm}} 是一個完全受管理的資料層，專為現代 Web 及行動應用程式而設計，採用了靈活的 JSON 綱目。{{site.data.keyword.cloudant_short_notm}} 是根據 Apache CouchDB 而建置並與其相容，可透過安全的 HTTPS API 進行存取，如此可隨著應用程式的增長而調整規模。

1. 在「型錄」中，選取 **Cloudant**。
2. 將服務名稱設定為 **guestbook-db**，選取**使用舊認證和 IAM** 作為鑑別方法，然後按一下**建立**。
3. 回到「資源清單」，按一下「名稱」直欄下的 ***guestbook-db** 項目。附註：您可能需要等待服務佈建完畢。 
4. 在服務詳細資料畫面中，按一下***啟動 Cloudant 儀表板***，該儀表板將在另一個瀏覽器標籤中開啟。附註：Cloudant 實例可能需要登入。 
5. 按一下***建立資料庫***，然後建立名為 ***guestbook*** 的資料庫。

  ![](images/solution8/Create_Database.png)

6. 回到服務詳細資料標籤，在**服務認證**下
   1. 建立**新認證**，接受預設值，然後按一下**新增**。
   2. 按一下「動作」下的**檢視認證**。稍後將需要這些認證來容許 Cloud Functions 動作對 Cloudant 服務執行讀取/寫入動作。

## 建立無伺服器動作

在此區段中，您將建立無伺服器動作（通常稱為「函數」）。{{site.data.keyword.openwhisk}}（以 Apache OpenWhisk 為基礎）是一種函數即服務 (FaaS) 平台，用於根據送入事件執行函數，並且在不使用時，不會產生任何成本。


![](images/solution8/Functions.png)

### 儲存留言板項目的動作序列

您將建立**序列**；序列是一個動作鏈，一個動作的輸出會當作下一個動作的輸入，依此類推。將建立的第一個序列用於持續保存訪客訊息。如果提供了姓名、電子郵件 ID 和註解，則序列將為：
   * 建立要持續保存的文件。
   * 將文件儲存在 {{site.data.keyword.cloudant_short_notm}} 資料庫中。

請先建立第一個動作：

1. 切換到 **Functions** https://{DomainName}/openwhisk。
2. 在左側窗格中，按一下**動作**，然後按一下**建立**。
3. **建立動作**，名稱為 `prepare-entry-for-save`，然後選取 **Node.js** 作為「運行環境」（附註：請選取最新的版本）。
4. 將現有程式碼取代為以下程式碼 Snippet：
   ```js
   /**
    * Prepare the guestbook entry to be persisted
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. **儲存**

然後，將該動作新增至序列：

1. 按一下**包含序列**，然後按一下**新增至序列**。
1. 對於序列名稱，輸入 `save-guestbook-entry-sequence`，然後按一下**建立並新增**。

然後，將第二個動作新增至序列：

1. 按一下 **save-guestbook-entry-sequence**，然後按一下**新增**。
1. 選取**使用公用** > **Cloudant**，然後選取**動作**下的 **create-document**。
1. **新建連結**。 
1. 對於「名稱」，輸入 `binding-for-guestbook`。
1. 對於「Cloudant 實例」，選取`輸入您自己的認證`，並使用為 Cloudant 實例擷取的認證資訊填寫「使用者名稱」、「密碼」、「主機」和「資料庫」（為 `guestbook`）欄位，然後按一下**新增**，再按一下**儲存**。
   {: tip}
1. 若要測試，請按一下**變更輸入**，然後輸入以下 JSON：
    ```json
    {
         "name": "John Smith",
      "email": "john@smith.com",
      "comment": "this is my comment"
    }
    ```
    {: codeblock}
1. 按一下**套用**，然後按一下**呼叫**。
    ![](images/solution8/Save_Entry_Invoke.png)

### 擷取項目的動作序列

第二個序列用於擷取現有的留言板項目。此序列將：
   * 列出資料庫中的所有文件。
   * 設定文件格式，然後傳回這些文件。

1. 在 **Functions** 下，按一下**動作**，然後**建立**新的 Node.js 動作，並將其命名為 `set-read-input`。
2. 將現有程式碼取代為以下程式碼 Snippet。此動作會將適當的參數傳遞給下一個動作。
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. **儲存**

將該動作新增至序列：

1. 按一下**包含序列** > **新增至序列**，然後按一下**建立新的項目**。
1. 對於**動作名稱**，輸入 `read-guestbook-entries-sequence`，然後按一下**建立並新增**。

完成序列：

1. 按一下 **read-guestbook-entries-sequence** 序列，然後按一下**新增**以建立並新增第二個動作，此動作用於從 Cloudant 取得文件。
1. 在**使用公用**下，選擇 **{{site.data.keyword.cloudant_short_notm}}**，然後選擇 **list-documents**。
1. 在**我的連結**下，選擇 **binding-for-guestbook**，然後選擇**新增**以建立此公用動作，並將其新增至序列。
1. 再次按一下**新增**以建立並新增第三個動作，此動作將用於設定來自 {{site.data.keyword.cloudant_short_notm}} 之文件的格式。
1. 在**建立新的項目**下，對於「名稱」，輸入 `format-entries`，然後按一下**建立並新增**。
1. 按一下 **format-entries**，然後將程式碼取代為以下內容：
   ```js
   const md5 = require('spark-md5');

      function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. **儲存**
1. 透過按一下**動作**，然後按一下 **read-guestbook-entries-sequence** 來選擇序列。
1. 按一下**儲存**，然後按一下**呼叫**。輸出應該類似於下列內容：
   ![](images/solution8/Read_Entries_Invoke.png)

## 建立 API

![](images/solution8/Cloud_Functions_API.png)

1. 移至「動作」：https://{DomainName}/openwhisk/actions。
2. 選取 **read-guestbook-entries-sequence** 序列。在名稱旁邊，按一下 **Web 動作**，選取**啟用 Web 動作**，然後按一下**儲存**。
3. 對 **save-guestbook-entry-sequence** 序列執行相同的操作。
4. 移至 API (https://{DomainName}/openwhisk/apimanagement)，然後按一下**建立 {{site.data.keyword.openwhisk_short}} API**。
5. 將「名稱」設定為 `guestbook`，將「基本路徑」設定為 `/guestbook`。
6. 按一下**建立作業**，然後建立用於擷取留言板項目的作業：
   1. 將**路徑**設定為 `/entries`。
   2. 將**動詞**設定為 `GET*`。
   3. 選取 **read-guestbook-entries-sequence** 動作
7. 按一下**建立作業**，然後建立用於持續保存留言板項目的作業：
   1. 將**路徑**設定為 `/entries`。
   2. 將**動詞**設定為 `PUT`
   3. 選取 **save-guestbook-entry-sequence** 動作
8. 儲存並公開該 API。

## 部署 Web 應用程式

1. 將 Guestbook 使用者介面儲存庫 (https://github.com/IBM-Cloud/serverless-guestbook) 分出到公用 GitHub。
2. 修改 **docs/guestbook.js**，並將 **apiUrl** 的值取代為 API 閘道給定的路徑。
3. 將修改後的檔案提交到分出的儲存庫。
4. 在儲存庫的「設定」頁面中，捲動到 **GitHub Pages**，將來源變更為**主分支 /docs 資料夾**，然後按一下「儲存」。
5. 存取儲存庫的公用頁面。
6. 您應該會看到先前建立的「測試」留言板項目。
7. 新增新項目。

![](images/solution8/Guestbook.png)

## 選用：將您自己的網域用於 API

建立受管理 API 會提供您預設端點，例如 `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`。在此區段中，您將配置此端點，使其能夠處理來自於自訂子網域的要求。

### 取得自訂網域的憑證

透過自訂網域公開 {{site.data.keyword.openwhisk_short}} 動作，將需要安全的 HTTPS 連線。您應該為計劃搭配無伺服器後端使用的網域及子網域，取得 SSL 憑證。假設有類似 *mydomain.com* 的網域，可以在 *guestbook-api.mydomain.com* 上管理動作。需要簽發用於 *guestbook-api.mydomain.com*（或 **.mydomain.com*）的憑證。

您可以從 [Let's Encrypt](https://letsencrypt.org/) 取得免費的 SSL 憑證。在此程序中，可能需要在 DNS 介面中配置 TXT 類型的 DNS 記錄，以證明您是域的擁有者。
{:tip}

取得網域的 SSL 憑證及私密金鑰之後，請務必將它們轉換成 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 格式。

1. 將憑證轉換為 PEM 格式：
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. 將私密金鑰轉換為 PEM 格式：
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### 將憑證匯入至中央儲存庫

1. 在支援的位置建立 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 實例。
1. 在服務儀表板中，使用**匯入憑證**：
   * 將**名稱**設定為自訂子網域和網域，例如 `guestbook-api.mydomain.com`。
   * 瀏覽 PEM 格式的**憑證檔案**。
   * 瀏覽 PEM 格式的**私密金鑰檔案**。
   * **匯入**。

### 配置受管理 API 的自訂網域

1. 移至 [API / 自訂網域](https://{DomainName}/apis/domains)。
1. 在「地區」選取器中，選取部署了動作的位置。
1. 找到鏈結至您建立動作和受管理 API 之組織及空間的自訂網域。在動作功能表中按一下**變更設定**。
1. 記下**預設網域 / 別名**值。
1. 勾選**套用自訂網域**
   1. 將**網域名稱**設定為將使用的網域，例如 `guestbook-api.mydomain.com`。
   1. 選取持有憑證的 {{site.data.keyword.cloudcerts_short}} 實例。
   1. 選取網域的憑證。
1. 移至 DNS 提供者，然後建立新的 **DNS TXT 記錄**，用於將您的網域對映到 API 預設網域/別名。設定套用之後便可以移除 DNS TXT 記錄。
   
1. 儲存自訂網域設定。對話框會檢查 DNS TXT 記錄是否存在。
1. 最後，回到 DNS 提供者的設定，然後建立 CNAME 記錄，用於將自訂網域（例如，guestbook-api.mydomain.com）指向預設網域/別名。這會導致透過您自訂網域的資料流量遞送至您的後端 API。

傳播 DNS 變更後，即能夠存取留言板 API：https://guestbook-api.mydomain.com/guestbook。

1. 編輯 **docs/guestbook.js**，並將 **apiUrl** 值更新為 https://guestbook-api.mydomain.com/guestbook。
1. 確定修改的檔案。
1. 現在，應用程式可透過自訂網域存取該 API。

## 移除資源

* 刪除 {{site.data.keyword.cloudant_short_notm}} 服務
* 從 {{site.data.keyword.openwhisk_short}} 中刪除 API
* 從 {{site.data.keyword.openwhisk_short}} 中刪除動作

## 相關內容
* [More guides and samples on serverless](https://developer.ibm.com/code/journey/category/serverless/)
* [開始使用 {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [{{site.data.keyword.openwhisk}} 常見使用案例](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [透過動作建立 REST API](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
