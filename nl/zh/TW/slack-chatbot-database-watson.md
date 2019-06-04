---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 建置資料庫驅動的 Slackbot
{: #slack-chatbot-database-watson}

在本指導教學中，您將建置一個 Slackbot，用於建立 Db2 資料庫項目並在其中搜尋事件和會議。Slackbot 由 {{site.data.keyword.conversationfull}} 服務提供支援。您將使用 Assistant 整合來整合 Slack 和 {{site.data.keyword.conversationfull}}。

Slack 整合在 Slack 和 {{site.data.keyword.conversationshort}} 之間傳遞訊息。在其中，一些伺服器端對話動作會對 Db2 資料庫執行 SQL 查詢。所有（但並不多）程式碼都是用 Node.js 撰寫。

## 目標
{: #objectives}

* 使用整合將 {{site.data.keyword.conversationfull}} 連接至 Slack
* 在 {{site.data.keyword.openwhisk_short}} 中建立、部署和連結 Node.js 動作
* 使用 Node.js 從 {{site.data.keyword.openwhisk_short}} 存取 Db2 資料庫

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}} ](https://{DomainName}/catalog/services/db2-warehouse) 或 [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

<p style="text-align: center;">

  ![架構](images/solution19/SlackbotArchitecture.png)
</p>

## 開始之前
{: #prereqs}

若要完成本指導教學，您需要最新版本的 [{{site.data.keyword.Bluemix_notm}} CLI](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)，並且 {{site.data.keyword.openwhisk_short}} [外掛程式已安裝](/docs/cli?topic=cloud-cli-plug-ins)。


## 服務及環境設定
在此區段中，您將設定所需的服務並準備環境。其中大部分操作可以從指令行介面 (CLI) 使用 Script 來完成。Script 可在 GitHub 上取得。

1. 複製 [GitHub 儲存庫](https://github.com/IBM-Cloud/slack-chatbot-database-watson)，然後導覽至複製的目錄：
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. 如果您尚未登入，請使用 `ibmcloud login` 以互動方式登入。
3. 使用下列指令將要在其中建立資料庫服務的組織和空間設定為目標：
   ```
ibmcloud target --cf
  ```
4. 建立 {{site.data.keyword.dashdbshort}} 實例，並將其命名為 **eventDB**：
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
   您還可以使用除**入門**套餐以外的套餐。
5. 若之後要從 {{site.data.keyword.openwhisk_short}} 存取資料庫服務，您需要授權。因此，請建立服務認證，並將其標示為 **slackbotkey**：   
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. 建立 {{site.data.keyword.conversationshort}} 服務實例。使用 **eventConversation** 作為名稱，並使用免費的精簡方案。
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. 接下來，將登錄 {{site.data.keyword.openwhisk_short}} 動作，並將服務認證連結到這些動作。某些動作啟用為 Web 動作，並且設定了密碼以阻止未經授權的呼叫。選擇一個密碼，並將其作為參數傳遞 - 相應地取代 **YOURSECRET**。

   其中一個動作被呼叫，以在 {{site.data.keyword.dashdbshort}} 中建立表格。藉由使用 {{site.data.keyword.openwhisk_short}} 的動作，您既不需要本端 Db2 驅動程式，也不必使用以瀏覽器為基礎的介面來手動建立表格。若要執行登錄和設定，請執行下面的行，這將執行包含所有動作的 **setup.sh** 檔案。如果系統不支援 Shell 指令，請複製 **setup.sh** 檔案中的每行，並分別執行各行。

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **附註：**依預設，Script 還會插入幾行資料範例。可以藉由在上面的 Script，將下列行設為註釋來停用此插入動作：`#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r`
8. 解壓縮已部署動作的名稱空間資訊。

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   記下 **/slackdemo/eventInsert** 前面的部分。這是已編碼的組織和空間。在下一區段中將需要此資訊。

## 載入技能/工作區
在本指導教學的此部分中，您將在 {{site.data.keyword.conversationshort}} 服務中載入預先定義的工作區或技能。
1. 在 [{{site.data.keyword.Bluemix_short}} 資源清單](https://{DomainName}/resources)中，開啟您的服務概觀。找到在先前區段中建立的 {{site.data.keyword.conversationshort}} 服務實例。按一下其項目，然後按一下服務別名以開啟服務詳細資料。
2. 按一下**啟動工具**以前往 {{site.data.keyword.conversationshort}} 工具。
3. 切換到**技能**，然後按一下**建立技能**，再按一下**匯入技能**。
4. 在對話框中，按一下**選取 JSON 檔案**後，從本端目錄中選取 **assistant-skill.json** 檔案。將匯入選項保留為**全部內容（目的、實體和對話）**，然後按一下**匯入**。這將建立名為 **TutorialSlackbot** 的新技能。
5. 按一下**對話**以查看對話節點。可以展開這些節點以查看結構，結構類似於下圖所示。

   該對話具有多個節點，用於處理尋求協助的問題和簡單的「順頌商祺」。**newEvent** 節點及其子節點收集必要的輸入，然後呼叫動作以將新的事件記錄插入到 Db2 中。

   **query events** 節點澄清事件是依 ID 還是日期進行搜尋。然後，**query events by shortname** 和 **query event by dates** 子節點會執行實際搜尋並收集所需資料。

   **credential_node** 設定用於對話動作的密碼以及有關 Cloud Foundry 組織的資訊。呼叫動作時需要後者。

  設定完所有內容後，後文將說明詳細資料。
  ![](images/solution19/SlackBot_Dialog.png)   
6. 按一下 **credential_node** 對話節點，藉由按一下**回應為：**右側的「功能表」圖示以開啟 JSON 編輯器。

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   將 **org_space** 取代為早先擷取到的已編碼組織和空間資訊。用 "%40" 替換所有 "@" 。接下來，將 **YOURSECRET** 變更為先前的實際密碼。再次按一下該圖示以關閉 JSON 編輯器。

## 建立助理並與 Slack 整合

現在，您將建立與先前技能關聯的助理，並將其與 Slack 整合。 
1. 按一下左上角的**技能**，然後選取**助理**。接下來，按一下**建立助理**。
2. 在對話框中，填寫 **TutorialAssistant** 作為名稱，然後按一下**建立助理**。在下一個畫面上，選擇**新增對話技能**。隨後，選擇**新增現有技能**，從清單中選取 **TutorialSlackbot**，並新增此項。
3. 新增技能後，按一下**新增整合**，然後從**受管整合**清單中，選取 **Slack**。
4. 按照指示將聊天機器人與 Slack 整合。

## 測試 Slackbot 並使其瞭解
開啟 Slack 工作區以試用聊天機器人。開始與機器人直接聊天。

1. 在傳訊表單中輸入 **help**。機器人應該會提供一些指引資訊。
2. 現在輸入 **new event** 以開始收集新事件記錄的資料。您將使用 {{site.data.keyword.conversationshort}} 插槽來收集所有必要的輸入。
3. 首先是事件 ID 或名稱。引號是必要的。使用引號可容許輸入更複雜的名稱。請輸入 **"Meetup: IBM Cloud"** 作為事件名稱。事件名稱會定義為以模式為基礎的實體 **eventName**。它支援在開頭和結尾使用不同類型的雙引號。
4. 接下來是事件位置。輸入是根據[系統實體 **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details)。但限制是只能使用 {{site.data.keyword.conversationshort}} 認可的縣/市。請嘗試使用 **Friedrichshafen** 這一縣/市。
5. 在下一步中將要求提供電子郵件位址或網站的 URI 等聯絡資訊。首先是 **https://www.ibm.com/events**。您將對該欄位使用以模式為基礎的實體。
6. 接下來的問題是收集開始和結束日期和時間。將使用容許不同輸入格式的 **sys-date** 和 **sys-time**。使用**下周四**作為開始日期，**下午 6 點**作為開始時間，使用下周四的確切日期和時間（例如，**2019-05-09** 和 **22:00**）作為結束日期及時間。
7. 最後，收集了所有資料後，系統將顯示摘要，並呼叫實作為 {{site.data.keyword.openwhisk_short}} 動作的伺服器動作，以將新記錄插入到 Db2 中。隨後，對話會切換到子節點，以藉由移除環境定義變數來清除處理環境。藉由輸入 **cancel**、**exit** 或類似操作，可以隨時取消整個輸入程序。在這種情況下，系統將確認使用者選擇，隨後會清除環境。
  ![](images/solution19/SlackSampleChat.png)   

現在有了一些取樣資料，可以進行搜尋了。
1. 鍵入 **show event information**。接下來，會提問是要依 ID 還是日期進行搜尋。輸入 **name**，然後輸入下一個問題 **"Think 2019"**。現在，聊天機器人應該會顯示有關該事件的資訊。該對話有多個回應可供選擇。
2. 使用 {{site.data.keyword.conversationshort}} 作為後端，可以輸入更複雜的片語，而跳過對話的某些部分。使用 **show event by the name "Think 2019"** 作為輸入。聊天機器人會直接傳回相應事件記錄。
3. 現在您要依日期進行搜尋。搜尋是由一對日期所定義，事件開始日期必須介於這對日期之間。將 **search conference by date in February 2019** 作為輸入，結果應該同樣為 **Think 2019** 事件。實體 **February** 會解釋為兩個日期 - 2 月 1 日和 2 月 28 日，因此為日期範圍的開始和結束日期提供輸入。[如果沒有指定年份 2019，則系統將識別未來的 2 月](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time)。 

執行了一些搜尋並有了新的事件項目後，您可以重新參閱會談歷程，並改進未來的對話。請遵循 [{{site.data.keyword.conversationshort}} 文件中有關**改進理解**](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro)的指示。


## 移除資源
{:removeresources}

在主目錄中執行清理 Script 會從 {{site.data.keyword.dashdbshort}} 中刪除事件表格，並從 {{site.data.keyword.openwhisk_short}} 中移除動作。開始修改和延伸程式碼時，此作業可能非常有用。清理 Script 不會變更 {{site.data.keyword.conversationshort}} 工作區或技能。   
```bash
sh cleanup.sh
```
{:codeblock}

在 [{{site.data.keyword.Bluemix_short}} 資源清單](https://{DomainName}/resources)中，開啟您的服務概觀。找到 {{site.data.keyword.conversationshort}} 服務實例，然後將其刪除。

## 擴展指導教學
想要新增或變更此指導教學？以下是一些構想：
1. 新增搜尋功能，例如萬用字元搜尋或搜尋事件持續時間 ("give me all events longer than 8 hours")。
2. 使用 {{site.data.keyword.databases-for-postgresql}}，而不使用 {{site.data.keyword.dashdbshort}}。[此 Slackbot 的 GitHub 儲存庫指導教學](https://github.com/IBM-Cloud/slack-chatbot-database-watson)已經有程式碼來支援 {{site.data.keyword.databases-for-postgresql}}。
3. 新增天氣服務並擷取事件日期和位置的預測資料。
4. 將事件資料匯出為 iCalendar **.ics** 檔案。
5. 藉由新增其他整合，將聊天機器人連接至 Facebook Messenger。
6. 向輸出新增互動式元素，例如按鈕。      


## 相關內容
{:related}

以下是本指導教學所涵蓋主題的其他資訊鏈結。

與聊天機器人相關的部落格文章：
* [Chatbots: Some tricks with slots in IBM Watson Conversation](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [Lively chatbots: Best Practices](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [Building chatbots: more tips and tricks](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

文件及 SDK：
* GitHub 儲存庫中的[在 IBM Watson Conversation 中處理變數的提示和技巧](https://github.com/IBM-Cloud/watson-conversation-variables)
* [{{site.data.keyword.openwhisk_short}} 文件](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* 文件：[IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Free Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions)（適用於開發人員）
* 文件：[ibm_db Node.js 驅動程式的 API 說明](https://github.com/ibmdb/node-ibm_db)
* [{{site.data.keyword.cloudantfull}} 文件](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
