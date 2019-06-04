---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 使用 Streaming Analytics 和 SQL 處理海量資料日誌
{: #big-data-log-analytics}

在本指導教學中，您將建置一個日誌分析管線，用於收集、儲存和分析日誌記錄，以滿足法規要求或輔助資訊探索。此解決方案將利用 {{site.data.keyword.cloud_notm}} 中提供的多個服務：{{site.data.keyword.messagehub}}、{{site.data.keyword.cos_short}}、SQL Query 和 {{site.data.keyword.streaminganalyticsshort}}。程式將藉由模擬 Web 伺服器日誌訊息從靜態檔案傳輸到 {{site.data.keyword.messagehub}} 的處理程序，來為您提供協助。

使用 {{site.data.keyword.messagehub}}，管線可以進行調整，以接收來自各種生產者的數百萬條日誌記錄。藉由套用 {{site.data.keyword.streaminganalyticsshort}}，可以即時檢查日誌資料以整合商業程序。使用 {{site.data.keyword.cos_short}} 還可以輕鬆地將日誌訊息重新導向到長期儲存空間，供開發人員、支援人員和審核員使用 SQL Query 直接使用資料。

雖然本指導教學專注於說明日誌分析，但也適用於其他情境：儲存空間受限的 IoT 裝置可以採用類似方式將訊息串流式傳輸到 {{site.data.keyword.cos_short}}"，或者行銷專員可以使用 SQL Query 來對不同數位資產中的客戶事件進行分段並分析。
{:shortdesc}

## 目標

{: #objectives}

* 瞭解 Apache Kafka 發佈/訂閱傳訊
* 儲存日誌資料，以滿足審核和規範要求
* 監視日誌，以建立異常狀況處理程序
* 對日誌資料處理取證和統計分析

## 使用的服務

{: #services}

本指導教學使用下列運行環境及服務：

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構

{: #architecture}

<p style="text-align: center;">

  ![架構](images/solution31/Architecture.png)
</p>

1. 應用程式產生日誌事件並傳送到 {{site.data.keyword.messagehub}}
2. {{site.data.keyword.streaminganalyticsshort}} 攔截並分析日誌事件
3. 日誌事件附加到位於 {{site.data.keyword.cos_short}} 中的 CSV 檔案
4. 審核員或支援人員發出 SQL 工作
5. 使用 SQL Query 對 {{site.data.keyword.cos_short}} 中的日誌檔執行查詢
6. 結果集儲存在 {{site.data.keyword.cos_short}} 中，並傳遞給審核員和支援人員

## 開始之前

{: #prereqs}

* [安裝 Git](https://git-scm.com/)
* [安裝 {{site.data.keyword.Bluemix_notm}} CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* [安裝 Node.js](https://nodejs.org)
* [下載 Kafka 0.10.2.X 用戶端](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)

## 建立服務

{: #setup}

在本節中，您將建立對應用程式產生的日誌事件執行分析所需的服務。

此節使用指令行來建立服務實例。或者，您可以從型錄的服務頁面使用所提供的鏈結執行相同作業。
{: tip}

1. 透過指令行登入 {{site.data.keyword.cloud_notm}}，並設定 Cloud Foundry 帳戶的目標。請參閱 [CLI 開始使用](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)。
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. 建立 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) 的精簡實例。
    ```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. 建立 [SQL Query](https://{DomainName}/catalog/services/sql-query) 的精簡實例。
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. 建立 [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams) 的標準實例。
    ```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## 建立傳訊主題和 {{site.data.keyword.cos_short}} 儲存區

{: #topics}

首先建立 {{site.data.keyword.messagehub}} 主題和 {{site.data.keyword.cos_short}} 儲存區。主題定義應用程式在發佈/訂閱傳訊系統中傳遞訊息的位置。系統收到並處理訊息後，會將訊息儲存在位於 {{site.data.keyword.cos_short}} 儲存區的檔案中。

1. 在瀏覽器中，存取[資源](https://{DomainName}/resources?search=log-analysis)中的 `log-analysis-hub` 服務實例。
2. 按一下 **+** 按鈕以建立主題。
3. 輸入**主題名稱** `webserver`，然後按一下**建立主題**按鈕。
4. 按一下**服務認證**，再按一下**新建認證**按鈕。
5. 在產生的對話框中，鍵入 `webserver-flow` 作為**名稱**，然後按一下**新增**按鈕。
6. 按一下**檢視認證**，然後將其中的資訊複製到安全位置。在下節中將用到這些資訊。
7. 回到[資源清單](https://{DomainName}/resources?search=log-analysis)，選取 `log-analysis-cos` 服務實例。
8. 按一下**建立儲存區**。
    * 輸入儲存區的唯一**名稱**。
    * 對於**備援**，選取**跨地區**。
    * 選取 **us-geo** 作為**位置**。
    * 按一下**建立儲存區**。

## 建立 Streams 流程來源

{: #streamsflow}

在本節中，您將開始配置接收日誌訊息的 Streams 流程。{{site.data.keyword.streaminganalyticsshort}} 服務採用 {{site.data.keyword.streamsshort}} 技術，該技術每秒可以分析數百萬個事件，而能達到亞毫秒級的回應時間和即時決策。

1. 在瀏覽器中，存取 [Watson Data 平台](https://dataplatform.ibm.com)。
2. 選取**新建專案**按鈕或磚，並選取**基本**磚，然後按一下**確定**。
    * 輸入**名稱** `webserver-logs`。
    * **儲存空間**選項應該設定為 `log-analysis-cos`。如果未設定為此服務實例，請選取此服務實例。
    * 按一下**建立**按鈕。
3. 在產生的頁面上，選取**設定**標籤，然後勾選**工具**中的 **Streams Designer**。藉由按一下**儲存**按鈕以完成作業。
4. 按一下**新增到專案**按鈕，然後按一下頂端導覽列中的 **Streams 流程**。
    * 按一下**將 IBM Streaming Analytics 實例與以容器為基礎的方案相關聯**。
    * 藉由選取**精簡**圓鈕，然後按一下**建立**，以建立新的 {{site.data.keyword.streaminganalyticsshort}} 實例。請不要選取精簡 VM。
    * 對於**服務名稱**，提供 `log-analysis-sa`，然後按一下**確認**。
    * 對於 Streams 流程**名稱**，鍵入 `webserver-flow`。
    * 藉由按一下**建立**以完成作業。
5. 在產生的頁面上，選取 **{{site.data.keyword.messagehub}}** 磚。
    * 按一下**新增連線**，然後選取 `log-analysis-hub` {{site.data.keyword.messagehub}} 實例。如果看不到列出該實例，請選取 **IBM {{site.data.keyword.messagehub}}** 選項。手動輸入從上節的**服務認證**中取得的**連線詳細資料**。將連線的**名稱**指定為 `webserver-flow`。
    * 按一下**建立**以建立連線。
    * 從**主題**下拉清單中，選取 `webserver`。
    * 從**初始偏移量**下拉清單中，選取**從第一個新訊息開始**。
    * 按一下**繼續**。
6. 使**預覽資料**頁面保持開啟；在下節中將用到此頁面。

## 使用 Kafka 主控台工具搭配 {{site.data.keyword.messagehub}}

{: #kafkatools}

`webserver-flow` 目前處於閒置狀態，正在等待訊息。在本節中，您將配置 Kafka 主控台工具以使用 {{site.data.keyword.messagehub}}。使用 Kafka 主控台工具，可以從終端機產生任意訊息，並將這些訊息傳送到 {{site.data.keyword.messagehub}}，這將觸發 `webserver-flow`。

1. 下載並解壓縮 [Kafka 0.10.2.X 用戶端](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)。
2. 將目錄切換至 `bin`，然後建立包含下列內容且名為 `message-hub.config` 的文字檔。
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. 將 `message-hub.config` 檔案中的 `USER` 和 `PASSWORD` 取代為上節的**服務認證**中顯示的 `user` 和 `password` 值。儲存 `message-hub.config`。
4. 從 `bin` 目錄中，執行下列指令。將 `KAFKA_BROKERS_SASL` 取代為**服務認證**中顯示的 `kafka_brokers_sasl` 值。提供的範例如下。
    ```sh
    ./kafka-console-producer.sh --broker-list KAFKA_BROKERS_SASL \
    --producer.config message-hub.config --topic webserver
    ```
    {: pre}
    ```sh
    ./kafka-console-producer.sh --broker-list \
    kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093 \
    --producer.config message-hub.config --topic webserver
    ```
5. Kafka 主控台工具正在等待輸入。將下面的日誌訊息複製並貼入終端機中。按 `Enter` 鍵將日誌訊息傳送到 {{site.data.keyword.messagehub}}。請注意，已傳送的訊息還會顯示在 `webserver-flow` **預覽資料**頁面上。
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![「預覽」頁面](images/solution31/preview_data.png)

## 建立 Streams 流程目標

{: #streamstarget}

在本節中，您將藉由定義目標來完成 Streams 流程配置。目標將用於在稍早建立的 {{site.data.keyword.cos_short}} 儲存區中儲存送入的日誌訊息。將送入的日誌訊息儲存並附加到檔案的處理程序將由 {{site.data.keyword.streaminganalyticsshort}} 自動完成。

1. 在 `webserver-flow` **預覽**頁面上，按一下**繼續**按鈕。
2. 選取 **{{site.data.keyword.cos_full_notm}}** 磚作為目標。
    * 按一下**新增連線**，然後選取 `log-analysis-cos`。
    * 按一下**建立**。
    * 輸入**檔案路徑** `/YOUR_BUCKET_NAME/http-logs_%TIME.csv`。將 `YOUR_BUCKET_NAME` 取代為第一節中使用的名稱。
    * 在**格式**下拉清單中，選取 **csv**。
    * 勾選**直欄標頭行**勾選框。
    * 在**檔案建立原則**下拉清單中，選取**檔案大小**。
    * 藉由在**檔案大小 (KB)** 文字框中輸入 `102400`，將限制設定為 100 MB。
    * 按一下**繼續**。
3. 按一下**儲存**。
4. 按一下「播放」按鈕 **>** 以**啟動 Streams 流程**。
5. 在啟動流程後，再次從 Kafka 主控台工具傳送多條日誌訊息。可以藉由在 Streams Designer 中檢視 `webserver-flow` 來監看訊息到達。
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. 回到 {{site.data.keyword.cos_short}} 中的儲存區。在足夠的訊息進入流程或重新啟動該流程後，將有新的 `log.csv` 檔案存在。

![webserver-flow](images/solution31/flow.png)

## 向 Streams 流程新增條件行為

{: #streamslogic}

到目前為止，Streams 流程是一個簡單的管線，用於將訊息從 {{site.data.keyword.messagehub}} 移至 {{site.data.keyword.cos_short}}。更有可能的情況是，團隊希望即時瞭解關注的事件。例如，發生 HTTP 500（應用程式錯誤）事件時，發出的警示對各個團隊都有利。在本節中，您將向流程新增條件邏輯，以識別 HTTP 200（正常）和非 HTTP 200 程式碼。

1. 使用「鉛筆」按鈕來**編輯 Streams 流程**。
2. 建立一個過濾器節點來處理 HTTP 200 回應。
    * 在**節點**選用區中，將**過濾器**節點從**處理和分析**拖曳至畫布。
    * 在「名稱」文字框中，鍵入`正常`，目前此方框中包含的字組是`正常`。
    * 在**條件表示式**文字區中，輸入下列陳述式。
      ```sh
      responseCode == 200
      ```
      {: pre}
    * 使用滑鼠從 **{{site.data.keyword.messagehub}}** 節點的輸出（右側）到**正常**節點的輸入（左側）之間繪製一條直線。
    * 在**節點**選用區中，將**目標**下找到的**除錯**節點拖曳至畫布。
    * 藉由在**除錯**節點與**正常**節點之間繪製一條直線，以連接這兩個節點。
3. 重複此處理程序以建立使用相同節點和下列條件陳述式的 `Not OK` 過濾器。
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. 按一下「播放」按鈕以**儲存並執行 Streams 流程**。
5. 如果系統提示，請按一下**執行新版本**的鏈結。

![流程設計程式](images/solution31/flow_design.png)

## 增加訊息負載

{: #streamsload}

為了在 Streams 流程中檢視條件處理，您將增加傳送到 {{site.data.keyword.messagehub}} 的訊息量。提供的 Node.js 程式將模擬根據流向 Web 伺服器的資料流量傳送到 {{site.data.keyword.messagehub}} 的實際訊息流程。為了示範 {{site.data.keyword.messagehub}} 和 {{site.data.keyword.streaminganalyticsshort}} 的可調整性，您將增加日誌訊息的傳輸量。

本節使用的是 [node-rdkafka](https://www.npmjs.com/package/node-rdkafka)。如果模擬器安裝失敗，請參閱 npmjs 頁面以取得疑難排解指示。如果問題仍然存在，您可以跳至下節並手動上傳資料。

1. 下載並解壓縮來自 NASA 的 [Jul 01 to Jul 31, ASCII format, 20.7 MB gzip compressed](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz) 日誌檔。
2. 從 [GitHub 上的 IBM Cloud](https://github.com/IBM-Cloud/kafka-log-simulator) 複製日誌模擬器並進行安裝。
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. 切換至模擬器的目錄，然後執行下列指令來設定模擬器，並產生日誌事件訊息。將 `LOGFILE` 取代為下載的檔案。將 `BROKERLIST` 和 `APIKEY` 取代為稍早使用的對應**服務認證**。提供的範例如下。
    ```sh
    npm install
    ```
    ```sh
    npm run build
    ```
    ```sh
    node dist/index.js --file LOGFILE --parser httpd --broker-list BROKERLIST \
    --api-key APIKEY --topic webserver --rate 100
    ```
    {: pre}
    ```sh
    node dist/index.js --file /Users/ibmcloud/Downloads/NASA_access_log_Jul95 \
    --parser httpd --broker-list \
    "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093" \
    --api-key Np15YZKN3SCdABUsOpJYtpue6jgJ7CwYgsoCWaPbuyFbdM4R \
    --topic webserver --rate 100
    ```
4. 在瀏覽器中，在模擬器開始產生訊息後回到 `webserver-flow`。
5. 在所需數目的訊息透過條件分支後，使用 `Ctrl+C` 鍵停止模擬器。
6. 藉由增加或減少 `--rate` 值來對 {{site.data.keyword.messagehub}} 調整進行試驗。

模擬器將根據 Web 伺服器日誌中的經歷時間來延遲傳送下一條訊息。設定 `--rate 1` 將即時傳送事件。設定 `--rate 100` 表示對於 Web 伺服器日誌中的每 1 秒經歷時間，將在訊息之間使用 10 毫秒延遲。
{: tip}

![流程負載設定為 10](images/solution31/flow_load_10.png)

## 使用 SQL Query 調查日誌資料

{: #sqlquery}

根據模擬器傳送的訊息數，{{site.data.keyword.cos_short}} 上日誌檔的檔案大小明確增加。現在，您將藉由將 SQL Query 與日誌檔結合使用，充當調查員來回答審核或規範問題。使用 SQL Query 的優點是可以直接存取日誌檔 - 不需要額外的轉換或資料庫伺服器。

如果您不想等待模擬器傳送所有日誌訊息，請將[完整的 CSV 檔案](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp)上傳到 {{site.data.keyword.cos_short}} 以立即開始。
{: tip}

1. 存取[資源清單](https://{DomainName}/resources?search=log-analysis)中的 `log-analysis-sql` 服務實例。選取**開啟使用者介面** 以啟動 SQL Query。
2. 將下列 SQL 輸入到**在這裡鍵入 SQL ...** 文字區。
    ```sql
    -- What are the top 10 web pages on NASA from July 1995?
    -- Which mission might be significant?
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. 在日誌檔中擷取物件 SQL URL。
    * 從[資源清單](https://{DomainName}/resources?search=log-analysis)中，選取 `log-analysis-cos` 服務實例。
    * 選取先前建立的儲存區。
    * 按一下 `http-logs_TIME.csv` 檔案上的溢位功能表，然後選取**物件 SQL URL**。
    * 將該 URL **複製**到剪貼簿。
4. 使用「物件 SQL URL」更新 `FROM` 子句，然後按一下**執行**。
5. 結果可以在**結果**標籤上查看。雖然應該會顯示某些頁面（如甘迺迪太空中心首頁），但有一項任務在當前十分熱門。
6. 選取**查詢詳細資料**標籤來檢視其他資訊，例如結果在 {{site.data.keyword.cos_short}} 上的儲存位置。
7. 藉由在**在這裡鍵入 SQL ...** 文字區中分別新增下列問題和回答配對，以嘗試這些配對。
    ```sql
    -- Who are the top 5 viewers?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- Which viewer has suspicious activity based on application failures?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- Which requests showed a page not found error to the user?
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- What are the top 10 largest files?
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- What is the distribution of total traffic by hour?
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- Why did the previous result return an empty hour?
    -- Hint, find the malformed hostname.
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

FROM 子句不限於單一檔案。使用 `cos://us-geo/YOUR_BUCKET_NAME/` 可對儲存區中的所有檔案執行 SQL 查詢。
{: tip}

## 擴展指導教學

{: #expand}

恭喜您，您已使用 {{site.data.keyword.cloud_notm}} 建置日誌分析管線。下面是增強解決方案的其他建議。

* 在 Streams Designer 中使用其他目標在 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 中儲存資料或在 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) 中執行程式碼
* 遵循[使用 Object Storage 建置 Data Lake](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage) 指導教學，以新增用於記載資料的儀表板
* 使用 [{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect) 將其他系統與 {{site.data.keyword.messagehub}} 整合

## 移除服務

{: #removal}

在[資源清單](https://{DomainName}/resources?search=log-analysis)中，使用溢位功能表中的**刪除**或**刪除服務**功能表項目來移除下列服務實例。

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## 相關內容

{:related}

* [Apache Kafka](https://kafka.apache.org/)
