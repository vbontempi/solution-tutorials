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

# 使用 Object Storage 建置 Data Lake
{: #smart-data-lake}

關於 Data Lake 一詞的定義有許多，但在本指導教學的環境定義中，Data Lake 是指一種以原生格式儲存資料以供組織使用的方法。為此，您將使用 {{site.data.keyword.cos_short}} 為組織建立 Data Lake。透過將 {{site.data.keyword.cos_short}} 與 SQL Query 組合使用，資料分析師可以使用 SQL 來查詢資料所在的位置。此外，還將在 Jupyter Notebook 中利用 SQL Query 服務來處理簡單分析。完成後，容許非技術使用者使用 {{site.data.keyword.dynamdashbemb_notm}} 來探索自己的洞察。

## 目標

- 使用 {{site.data.keyword.cos_short}} 儲存原始資料檔
- 使用 SQL Query 直接在 {{site.data.keyword.cos_short}} 中查詢資料
- 在 {{site.data.keyword.DSX_full}} 中優化和分析資料
- 使用 {{site.data.keyword.dynamdashbemb_notm}} 在整個組織中共用資料

## 使用的服務

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [SQL Query](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## 架構

![架構](images/solution29/architecture.png)

1. 原始資料儲存在 {{site.data.keyword.cos_short}} 中
2. 使用 SQL Query 來減少、增強或優化資料
3. 在 {{site.data.keyword.DSX}} 中執行資料分析
4. 事業線存取 Web 應用程式
5. 從 {{site.data.keyword.cos_short}} 中取回優化的資料
6. 使用 {{site.data.keyword.dynamdashbemb_notm}} 建置事業線圖表

## 開始之前

- [安裝 Git](https://git-scm.com/)
- [安裝 {{site.data.keyword.Bluemix_notm}} CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
- [安裝 Aspera Connect](http://downloads.asperasoft.com/connect2/)
- [安裝 Node.js 和 NPM](https://nodejs.org)

## 建立服務

在此區段中，您將建立建置 Data Lake 所需的服務。

此節使用指令行來建立服務實例。或者，您可以從型錄的服務頁面使用所提供的鏈結執行相同作業。
{: tip}

1. 透過指令行登入 {{site.data.keyword.cloud_notm}}，並設定 Cloud Foundry 帳戶的目標。請參閱 [CLI 入門](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)。
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. 使用 Cloud Foundry 別名建立 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) 實例。如果已有服務實例，請使用現有服務名稱來執行 `service-alias-create` 指令。
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. 建立 [SQL Query](https://{DomainName}/catalog/services/sql-query) 實例。
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. 建立 [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio) 實例。
    ```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. 使用 Cloud Foundry 別名建立 [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded) 實例。
    ```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. 切換到工作目錄，並執行下列指令來複製儀表板應用程式的 [GitHub 儲存庫](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard)。然後將應用程式推送到 Cloud Foundy 組織。應用程式將使用其 [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml) 檔案自動連結上面的必要服務。
    ```sh
    git clone https://github.com/IBM-Cloud/nodejs-data-lake-dashboard.git
    ```
    {: pre}
    ```sh
    cd nodejs-data-lake-dashboard
    ```
    {: pre}
    ```sh
    npm install
    ```
    {: pre}
    ```sh
    npm run push
    ```
    {: pre}

    部署之後，應用程式將成為公用應用程式，並於隨機主機名稱上接聽。可以存取[資源清單](https://{DomainName}/resources)頁面，在「Cloud Foundry 應用程式」下，選取應用程式並檢視 URL，或者執行 `ibmcloud cf app dashboard-nodejs routes` 指令來檢視路徑。
    {: tip}

7. 透過在瀏覽器中存取應用程式的為公用 URL 來確認該應用程式是否處於作用中狀態。

![儀表板登入頁面](images/solution29/dashboard-start.png)

## 上傳資料

在此區段中，您將使用內建 {{site.data.keyword.CHSTSshort}} 將資料上傳到 {{site.data.keyword.cos_short}} 儲存區。{{site.data.keyword.CHSTSshort}} 可在資料上傳到儲存區時對資料進行保護，並[可以大幅縮短傳輸時間](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/)。

1. 下載 [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD) CSV 檔案。該檔案有 81 MB，可能需要幾分鐘才能下載完。
2. 在瀏覽器中，存取[資源清單](https://{DomainName}/resources)中的 **data-lake-cos** 服務實例。
3. 建立新儲存區來儲存資料。
    - 按一下**建立儲存區**按鈕。
    - 從**備援**下拉清單中，選取**地區**。
    - 從**位置**中，選取 **us-south**。目前，{{site.data.keyword.CHSTSshort}} 僅可用於在 `us-south` 位置建立的儲存區。或者，選擇其他位置，然後使用下一區段中的**標準**傳輸類型。
    - 提供儲存區的**名稱**，然後按一下**建立**。如果收到 *AccessDenied* 錯誤，請嘗試使用更獨特的儲存區名稱。
4. 將該 CSV 檔案上傳到 {{site.data.keyword.cos_short}}。
    - 在儲存區中，按一下**新增物件**按鈕。
    - 選取 **Aspera 高速傳輸**圓鈕。
    - 按一下**新增檔案**按鈕。這將開啟 Aspera 外掛程式，該外掛程式將位於個別的視窗中 - 可能在瀏覽器視窗後面。
    - 瀏覽至先前下載的 CSV 檔案並選取該檔案。

![具有 CSV 檔案的儲存區](images/solution29/cos-bucket.png)

## 使用資料

在此區段中，您將根據時間和年齡屬性，將最初的原始資料集轉換為目標群組。這對於有特定的興趣或者難以駕馭超大資料集的 Data Lake 消費者來說非常有用。

您將使用 SQL Query 透過熟悉的 SQL 陳述式來操作位於 {{site.data.keyword.cos_short}} 中的資料。SQL Query 內建了對 CSV、JSON 和 Parquet 的支援 - 無需額外的計算服務或擷取、轉換、載入。

1. 從[資源清單](https://{DomainName}/resources)存取 **data-lake-sql** SQL Query 服務實例。
2. 選取**開啟使用者介面**。
3. 透過直接對先前上傳的 CSV 檔案執行 SQL 來建立新資料集。
    - 將下列 SQL 輸入到**在這裡鍵入 SQL ...** 文字區。
    ```
選取
`Dr Number` AS id,
        `Date Occurred` AS date,
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
        FROM cos://us-south/<your-bucket-name>/Traffic_Collision_Data_from_2010_to_Present.csv
        WHERE
        `Time Occurred` >= 1700 AND
        `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND
        `Victim Age` <= 35
        ```
        {: codeblock}
    - 將 `FROM` 子句中的 URL 取代為您的儲存區名稱。
4. **目標**將自動建立 {{site.data.keyword.cos_short}} 儲存區來保存結果。將**目標**變更為 `cos://us-south/<your-bucket-name>/results`。
5. 按一下**執行**按鈕。結果如下所示。
6. 在**查詢詳細資料**標籤上，按一下緊跟在**結果位置** URL 後面的**啟動**圖示可檢視中間資料集，現在此資料集也儲存在 {{site.data.keyword.cos_short}} 上。

![記事本](images/solution29/sql-query.png)

## 組合使用 Jupyter Notebook 和 SQL Query

在此區段中，您將在 Jupyter Notebook 中使用 SQL Query 用戶端。這將在資料分析工具內重複使用儲存在 {{site.data.keyword.cos_short}} 上的資料。此組合還將建立資料集，這些資料集會自動儲存在 {{site.data.keyword.cos_short}} 上，然後可以用於 {{site.data.keyword.dynamdashbemb_notm}}。

1. 在 {{site.data.keyword.DSX}} 中建立新的 Jupyter 記事本。
    - 在瀏覽器中，開啟 [{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true)。
    - 按一下**建立專案**磚，然後按一下**資料研究**。
    - 按一下**建立專案**，然後提供**專案名稱**。
    - 確保**儲存空間**設定為 **data-lake-cos**。
    - 按一下**建立**。
    - 在產生的專案中，按一下**新增到專案**和**記事本**。
    - 在**空白**標籤中，輸入**記事本名稱**。
    - 使**語言**和**運行環境**保留為預設；按一下**建立記事本**。
2. 在記事本中，透過將下列指令新增到 **In [ ]:** 輸入提示字元，以安裝並匯入 PixieDust 和 ibmcloudsql，然後**執行**。
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. 將 {{site.data.keyword.cos_short}} API 金鑰新增到記事本。這將容許 SQL Query 結果儲存在 {{site.data.keyword.cos_short}} 中。
    - 在下一個 **In [ ]:** 提示字元中新增下列指令，然後**執行**。
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - 在終端機中，建立 API 金鑰。
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - 將 **API 金鑰**複製到剪貼簿。
    - 將 API 金鑰貼上到記事本中的相應文字框中，然後按 `Enter` 鍵。
    - 您還應該將 API 金鑰儲存到安全的永久位置；記事本不會儲存 API 金鑰。
4. 將 SQL Query 實例的 CRN（雲端資源名稱）新增到記事本。
    - 在下一個 **In [ ]:** 提示字元中，將 CRN 指派給記事本中的一個變數。
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - 在終端機中，將**ID** 內容中的 CRN 複製到剪貼板。
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - 將 CRN 貼上在單引號之間，然後**執行**。
5. 將另一個變數新增到記事本，以指定 {{site.data.keyword.cos_short}} 儲存區，然後**執行**。
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. 在另一個 **In [ ]:** 提示字元中執行下列指令，然後**執行**以檢視結果集。您還將有新的 `accidents/jobid=<id>/<part>.csv*` 檔案新增到儲存區，其中包含 `SELECT` 的結果。
    ```python
    sqlClient = ibmcloudsql.SQLQuery(cloud_api_key, sql_crn, sql_cos_endpoint + '/accidents')

    data_source = sql_cos_endpoint + "/Traffic_Collision_Data_from_2010_to_Present.csv"

    query = """
    SELECT
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
    FROM  {}
    WHERE
        `Time Occurred` >= 1700 AND `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND `Victim Age` <= 35
    """.format(data_source)

    traffic_collisions = sqlClient.run_sql(query)
    traffic_collisions.head()
    ```
    {: codeblock}

## 使用 PixieDust 顯示資料

在此區段中，您將使用 PixieDust 和 Mapbox 顯示先前的結果集，以便能更明確地識別出交通事故的模式或熱點。

1. 建立一般表格表示式，用於將 `location` 直欄轉換為個別的 `latitude` 和 `longitude` 直欄。在記事本的提示字元中，**執行**下列指令。
    ```python
    query = """
    WITH location AS (
        SELECT
            id,
            cast(split(coordinates, ',')[0] as float) as latitude,
            cast(split(coordinates, ',')[1] as float) as longitude
        FROM (SELECT
                `Dr Number` as id,
                regexp_replace(Location, '[()]', '') as coordinates
            FROM {0}
        )
    )
    SELECT
        d.`Dr Number` as id,
        d.`Date Occurred` as date,
        d.`Time Occurred` AS time,
        d.`Area Name` AS area,
        d.`Victim Age` AS age,
        d.`Victim Sex` AS sex,
        l.latitude,
        l.longitude
    FROM {0} AS d
        JOIN
        location AS l
        ON l.id = d.`Dr Number`
    WHERE
        d.`Time Occurred` >= 1700 AND
        d.`Time Occurred` <= 2000 AND
        d.`Victim Age` >= 20 AND
        d.`Victim Age` <= 35 AND
        l.latitude != 0.0000 AND
        l.latitude != 0.0000
    """.format(data_source)

    traffic_location = sqlClient.run_sql(query)
    traffic_location.head()
    ```
    {: codeblock}
2. 在下一個 **In [ ]:** 提示字元中，**執行** `display` 指令以使用 PixieDust 檢視結果。
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. 選取「圖表」下拉清單按鈕；然後選取**地圖**。
4. 將 `latitude` 和 `longitude` 新增到**索引鍵**。將 `id` 和 `age` 新增到**值**。按一下**確定**以檢視地圖。
5. 按一下**儲存**圖示以將記事本儲存到 {{site.data.keyword.cos_short}}。

![記事本](images/solution29/notebook-mapbox.png)

## 與組織共用資料集

並非 Data Lake 的所有使用者都是資料科學家。您可以容許非技術使用者使用 {{site.data.keyword.dynamdashbemb_notm}} 來從 Data Lake 取得洞察。與 SQL Query 類似，{{site.data.keyword.dynamdashbemb_notm}} 可以使用預先建置的儀表板，直接從 {{site.data.keyword.cos_short}} 中讀取資料。此區段說明的解決方案容許任何使用者存取 Data Lake 並建置自訂儀表板。

1. 存取先前推送到 {{site.data.keyword.Bluemix_notm}} 的儀表板應用程式的公用 URL。
2. 選取與您的目標佈置符合的範本。（下列步驟使用的是第一行中的第二個佈置。）
3. 使用`所選來源`端邊欄中顯示的`新增來源`按鈕，展開`儲存區名稱`手風琴圖標，然後按一下其中一個 `accidents/jobid=...` 表格項目。使用右上角的 X 圖示以關閉對話框。
4. 在左側，按一下`視覺化`圖示，然後按一下**摘要**。
5. 選取 `accidents/jobid=...` 來源，展開`表格`，然後建立圖表。
    - 將 `id` 拖放到**值**行上。
    - 使用上角的圖示收合圖表。
6. 回到`視覺化`中，建立**樹狀對映**圖表：
    - 將 `area` 拖放到**區域階層**行上。
    - 將 `id` 拖放到**大小**行上。
    - 收合圖表以檢視結果。

![儀表板圖表](images/solution29/dashboard-chart.png)

## 探索儀表板

在此區段中，您將執行一些額外步驟來探索儀表板應用程式和 {{site.data.keyword.dynamdashbemb_notm}} 的特性。

1. 按一下樣本儀表板應用程式工具列中的**模式**按鈕，以將模式視圖變更為 `VIEW`。
2. 按一下圖表下部的任何彩色磚，或按一下圖表圖註中的 `area` 值。這會將一個本端過濾器套用於該標籤，以便能導致其他圖表顯示該過濾器特定的資料。
3. 按一下工具列中的**儲存**按鈕。
    - 在對應的輸入欄位中，輸入儀表板的名稱。
    - 選取**規格**標籤，以檢視此儀表板的規格。規格是 {{site.data.keyword.dynamdashbemb_notm}} 的原生檔案格式。在其中，可找到有關已建立的圖表以及所使用的 {{site.data.keyword.cos_short}} 資料來源的資訊。
    - 使用對話框的**儲存**按鈕將儀表板儲存到瀏覽器的本端儲存空間中。
4. 按一下工具列的**建立新的項目**按鈕以建立新儀表板。若要開啟已儲存的儀表板，請按一下**開啟**按鈕。若要刪除儀表板，請使用「開啟儀表板」對話框中的**刪除**圖示。

在正式作業應用程式中，請對 URL、使用者名稱和密碼等資訊加密，以阻止一般使用者看到這些資訊。請參閱[加密資料來源資訊](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information)。
{: tip}

## 擴展指導教學

恭喜，您已使用 {{site.data.keyword.cos_short}} 建置了 Data Lake。以下是增強 Data Lake 的其他建議。

- 使用 SQL Query 試用其他資料集
- 完成[透過串流式分析和 SQL 使用海量資料日誌](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics)，以將來自多個來源的串流資料傳輸到 Data Lake 中
- 編輯儀表板應用程式的程式碼，以將儀表板規格儲存到 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 或 {{site.data.keyword.cos_short}}
- 建立 [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id) 服務實例以在儀表板應用程式中啟用安全

## 移除資源

執行下列指令移除使用的服務、應用程式和金鑰。

```sh
ibmcloud resource service-binding-delete dashboard-nodejs-dde dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-binding-delete dashboard-nodejs-cos dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-dde
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-cos
```
{: pre}
```sh
ibmcloud iam api-key-delete data-lake-cos-key
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-dde
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-cos
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-sql
```
{: pre}
```sh
ibmcloud service delete data-lake-studio
```
{: pre}
```sh
ibmcloud app delete dashboard-nodejs
```
{: pre}

## 相關內容

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Jupyter Notebook](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
