---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
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

# 建立、保護和管理 REST API
{: #create-manage-secure-apis}

本指導教學示範如何使用 LoopBack Node.js API Framework 建立 REST API。使用 Loopback，您可以快速建立將裝置和瀏覽器連接至資料和服務的 REST API。您還可以使用 {{site.data.keyword.apiconnect_long}} 向 API 新增管理、可見性、安全和速率限制。
{:shortdesc}

## 目標

* 在幾乎無需編碼的情況下建置 REST API
* 在 {{site.data.keyword.Bluemix_notm}} 上發佈 API 以供開發人員使用
* 將現有的 API 引入 {{site.data.keyword.apiconnect_short}}
* 安全地公開和控制記錄系統的存取權

## 使用的服務

本指導教學使用下列運行環境及服務：

* [Loopback](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* [SDK for Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry 應用程式

## 架構

![架構](images/solution13/Architecture.png)

1. 開發人員定義 RESTful API
2. 開發人員將 API 發佈到 {{site.data.keyword.apiconnect_long}}
3. 使用者和應用程式使用 API

## 開始之前

* 下載和安裝 [Node.js](https://nodejs.org/en/download/) 6.x 版（如果您已經安裝更新版本的 Node.js，請使用 [nvm](https://github.com/creationix/nvm) 或類似工具）

## 在 Node.js 中建立 REST API

{: #create_api}
在本節中，您將使用 [LoopBack](https://loopback.io/doc/index.html) 在 Node.js 中建立 API。LoopBack 是高度可擴展的開放程式碼 Node.js 架構，讓您幾乎不需要編碼就可以建立動態的端對端 REST API。

### 建立應用程式

1. 安裝 {{site.data.keyword.apiconnect_short}} 指令行工具。如果您安裝 {{site.data.keyword.apiconnect_short}} 時遇到問題，請在該指令之前使用 `sudo` 或者遵循[這裡](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit)的指示。
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. 輸入下列指令以建立應用程式。
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. 按 `Enter` 鍵以使用 **entries-api** 作為**應用程式的名稱**。
4. 按 `Enter` 鍵以使用 **entries-api** 作為**包含專案的目錄**。
5. 選擇 **3.x（現行）**作為 **LoopBack 的版本**。
6. 選擇 **empty-server** 作為**應用程式的類型**。

![APIC Loopback 支架作業](images/solution13/apic_loopback.png)

### 新增資料來源

資料來源代表後端系統，例如資料庫、外部 REST API、SOAP Web 服務和儲存空間服務。資料來源通常提供建立、擷取、更新和刪除 (CRUD) 功能。Loopback 支援許多類型的[資料來源](http://loopback.io/doc/en/lb3/Connectors-reference.html)，為求簡化，您將利用 API 來使用記憶體內的資料儲存庫。

1. 將目錄切換到新的專案，並啟動 API Designer。
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. 從導覽中，按一下**資料來源**。然後，按一下**新增**按鈕。此時將開啟**新建 LoopBack 資料來源**對話框。
3. 在**名稱**文字欄位中，鍵入 `entriesDS`，並按一下**新建**按鈕。
4. 從**連接器**組合框中，選取**記憶體內資料庫**。
5. 按一下左上方的**所有資料來源**。資料來源將在資料來源清單中顯示。

   編輯器將使用新資料來源的設定自動更新 server/datasources.json 檔案。
   {:tip}

![API Designer 資料來源](images/solution13/datastore.png)

### 新增模型

模型是具有 Node 和 REST API 的 JavaScript 物件，代表後端系統中的資料。模型透過資料來源連接至這些系統。在本節中，您將定義資料結構，並將其連接至先前建立的資料來源。

1. 從導覽中，按一下**模型**。然後，按一下**新增**按鈕。此時將開啟**新建 LoopBack 模型**對話框。
2. 在**名稱**文字欄位中，鍵入 `entry`，並按一下**新建**按鈕。
3. 從**資料來源**組合框中，選取 **entriesDS**。
4. 從**內容**卡片中，使用**新增內容**圖示新增下列內容。
    1. 針對**內容名稱**鍵入 `name`，並從**類型**組合框中選取 **string**。
    2. 針對**內容名稱**鍵入 `email`，並從**類型**組合框中選取 **string**。
    3. 針對**內容名稱**鍵入 `comment`，並從**類型**組合框中選取 **string**。
5. 按一下右上角的**儲存**圖示以儲存模型。

![模型產生器](images/solution13/models.png)

## 測試 LoopBack 應用程式

在本節中，您將啟動 Loopback 應用程式的本端實例，藉由使用 API Designer 插入和查詢資料來測試 API。您必須使用 API Designer 的「探索」工具測試瀏覽器中的 REST 端點，因為它包括正確的安全標頭和其他要求參數。

1. 藉由按一下左下角的**開始**圖示來啟動本端伺服器，並等待顯示**執行中**訊息。
2. 從橫幅中，按一下**探索**鏈結以顯示 API Designer 的「探索」工具。資訊看板顯示可用於 API 中的 LoopBack 模型的 REST 作業。
3. 按一下左側窗格的 **entry.create** 作業以顯示端點。中心窗格顯示有關端點的摘要資訊，包括其參數、安全、模型實例資料和回應碼。右側窗格提供範例程式碼，以使用 cURL 指令和語言（如 Ruby、Python、Java 和 Node）呼叫該端點。
4. 在右側窗格中，按一下**試用**鏈結。向下捲動到**參數**，並在**資料**文字區中輸入下列內容：
    ```javascript
    { "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. 按一下**呼叫作業**按鈕。![從 API Designer 測試 API](images/solution13/data_entry_1.png)
6. 藉由查看 **Response Code: 200 OK** 來確認 POST 已成功。如果您看到由於 localhost 的未受信任憑證而發出的錯誤訊息，請按一下 API Designer「探索」工具的錯誤訊息中所提供的鏈結來接受憑證，然後繼續在 Web 瀏覽器中呼叫作業。確切的程序取決於您使用的 Web 瀏覽器。
7. 使用 cURL 新增其他項目。確認埠與應用程式埠相符。
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. 按一下 **entry.find** 作業，然後按一下後面有**呼叫作業**按鈕的**試用**鏈結。回應顯示所有項目。您應該會看到 **Jane Doe** 和 **John Doe** 的 JSON。![entry_find](images/solution13/find_response.png)

您還可以從 `entries-api` 目錄發出 `npm start` 指令來手動啟動 Loopback 應用程式。您的 REST API 將在 [http://localhost:3000/API/entries](http://localhost:3000/API/entries) 中可用，但是，{{site.data.keyword.apiconnect_short}} 不會對其進行管理和保護。
{:tip}

## 建立 {{site.data.keyword.apiconnect_short}} 服務

為了針對後續步驟做準備，您將在 {{site.data.keyword.Bluemix_notm}} 上建立 **{{site.data.keyword.apiconnect_short}}** 服務。{{site.data.keyword.apiconnect_short}} 充當 API 的閘道，還提供管理、安全和速率限制。

1. 啟動 {{site.data.keyword.Bluemix_notm}} [資源清單](https://{DomainName}/resources)。
2. 導覽至**型錄 > 整合 > {{site.data.keyword.apiconnect_short}}**，並按一下**建立**按鈕。

## 將 API 發佈到 {{site.data.keyword.Bluemix_notm}}

{: #publish}
您將使用 API Designer 以將您的應用程式作為 Cloud Foundry 應用程式部署到 {{site.data.keyword.Bluemix_notm}}，並且還會將 API 定義發佈到 **{{site.data.keyword.apiconnect_short}}**。API Designer 是您的本端工具箱。如果您已將其關閉，請從專案目錄使用 `apic edit` 將其重新啟動。

還可以使用 `ibmcloud cf push` 指令手動部署應用程式，但是，它不會受到安全保護。若要[匯入 API](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing) 到 {{site.data.keyword.apiconnect_short}}，請使用 `definitions` 資料夾中提供的 OpenAPI 定義檔。使用 API Designer 進行部署將確保應用程式的安全，並自動匯入定義。
{:tip}

1. 回到 API Designer，按一下橫幅中的**發佈**鏈結。然後按一下**新增和管理目標 > 新增 IBM Bluemix 目標**。
2. 選取您要發佈到其中的**地區**和**組織**。
3. 選取**沙盤推演型錄**，然後按**下一步**。
4. 在**鍵入新的應用程式名稱**文字欄位中，鍵入 `entries-api-application`，並按一下 **+** 圖示。
5. 按一下清單中的 **entries-api-application**，並按一下**儲存**。
6. 按一下橫幅最左上角的漢堡**功能表**圖示。然後按一下**專案**和 **entries-api** 清單項目。
7. 在 API Designer 使用者介面中，按一下 **API > entries-API > 組合**鏈結。
8. 在「組合」編輯器中，按一下**過濾器原則**圖示。
9. 選取 **DataPower Gateway 原則**，並按一下**儲存**。
10. 按一下頂端列的**發佈**，並選取您的目標。選取**發佈應用程式**和「編譯打包或發佈產品」> 選取**特定產品** > **entries-api**。
11. 按一下**發佈**，並等待應用程式完成發佈。![「發佈」對話框](images/solution13/publish.png)

    應用程式包含與 API 相關的 Loopback 模型、資料來源和程式碼。產品容許您宣告 API 如何提供給開發人員。
    {:tip}

API 應用程式現在已作為 Cloud Foundry 應用程式發佈到 {{site.data.keyword.Bluemix_notm}}。您可以藉由查看 {{site.data.keyword.Bluemix_notm}} [資源清單](https://{DomainName}/resources)下的 Cloud Foundry 應用程式來看到該應用程式，但是由於該應用程式受到保護，因此無法使用 URL 進行直接存取。下節將向您顯示如何存取受管理 API。

## API 閘道

到現在為止，您一直在設計並本端測試您的 API。在本節中，您將使用 {{site.data.keyword.apiconnect_short}} 在 {{site.data.keyword.Bluemix_notm}} 上測試部署的 API。

1. 啟動 {{site.data.keyword.Bluemix_notm}} [資源清單](https://{DomainName}/resources)。
2. 找到並選取 **Cloud Foundry 服務**下的 **{{site.data.keyword.apiconnect_short}}** 服務。
3. 按一下**探索**功能表，然後按一下**沙盤推演**鏈結。
4. 按一下 **entry.create** 作業。
5. 在右窗格上，按一下**試用**。向下捲動到**參數**，並在**資料**文字區中輸入下列內容。
    ```javascript
    { "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. 按一下**呼叫作業**按鈕。應該會顯示 **Code: 200** 回應，指出作業成功。

![閘道](images/solution13/gateway.png)

受管理且安全的 API URL 將顯示在每個作業旁，應該類似於 `https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries`。
{: tip}

## 速率限制

設定速率限制讓您能管理 API 的網路資料流量，以及 API 內特定作業的網路資料流量。速率限制為特定時間間隔容許的呼叫數目上限。

1. 回到 API Designer，按一下**產品 > entries-api**。
2. 選取左側的**預設方案**。
3. 展開**預設方案**，並向下捲動到**速率限制**欄位。
4. 將欄位設定為 **10** 次呼叫/**1** **分鐘**。
5. 選取**強制硬性限制**，並按一下**儲存**圖示。![「速率限制」頁面](images/solution13/rate_limit.png)
6. 遵循[將 API 發佈到 {{site.data.keyword.Bluemix_notm}}](#publish) 小節中的步驟，以重新發佈 API。

您的 API 現在限制為每分鐘 10 個要求。使用**試用**特性達到限制。請參閱有關[設定速率限制](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits)的進一步資訊，或者探索 API Designer 以瞭解可用的所有管理特性。

## 擴展指導教學

恭喜，您已經建置受管理且安全的 API。以下是增強您的 API 的其他建議。

* 使用 [{{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant) LoopBack 連接器新增資料持續性
* 使用 API Designer [檢視其他設定](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0)以管理您的 API
* 檢閱 {{site.data.keyword.apiconnect_short}} [提供的](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics) API **分析**和**視覺化**

## 相關內容

* [Loopback 文件](https://loopback.io/doc/index.html)
* [{{site.data.keyword.apiconnect_long}} 開始使用](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
