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

# 用於雲端資料的 SQL 資料庫
{: #sql-database}

本指導教學說明瞭如何佈建 SQL（關係）資料庫服務，建立表格以及將大型資料集（縣/市資訊）載入到資料庫中。然後，將部署 Web 應用程式 "worldcities"，以利用這些資料並顯示如何存取雲端資料庫。該應用程式是使用 [Flask 架構](http://flask.pocoo.org/)以 Python 撰寫。

![](images/solution5/Architecture.png)

## 目標

* 佈建 SQL 資料庫
* 建立資料庫綱目（表格）
* 載入資料
* 連接應用程式和資料庫服務（共用認證）
* 監視、安全、備份和回復

## 產品

本指導教學使用下列產品：
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## 開始之前
{: #prereqs}

移至 [GeoNames](http://www.geonames.org/)，然後下載並解壓縮 [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip) 檔案。此文件包含有關人口超過 1000 的縣/市的資訊。您將使用此文件作為資料集。

## 佈建 SQL 資料庫
首先，建立 **[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)** 服務實例。

![](images/solution5/Catalog.png)

1. 造訪 [{{site.data.keyword.Bluemix_short}}「儀表板」](https://{DomainName})。按一下頂端導覽列的**型錄**。
2. 按一下左側窗格中「平台」下的**資料及分析**，然後選取 **{{site.data.keyword.dashdbshort_notm}}**。
3. 選取**入門**方案，然後將建議的服務名稱變更為 "sqldatabase" （稍後您將使用該名稱）。選取資料庫的部署位置，並確保選取正確的組織和空間。
4. 按一下**建立**。片刻之後，您應該會收到成功通知。
5. 在**資源清單**中，按一下新建立的 {{site.data.keyword.dashdbshort_notm}} 服務的項目。
6. 按一下**開啟**以啟動資料庫主控台。如果這是第一次使用主控台，系統將向您提供導覽。

## 建立表格
您需要表格來存放資料範例。請使用主控台來建立。

1. 在 {{site.data.keyword.dashdbshort_notm}} 的主控台中，按一下導覽列中的**探索**。這會讓您前往資料庫中現有模式的清單。
2. 找到並按一下以 "DASH" 開頭的綱目。
3. 按一下「**+ 新建表格"**」以顯示用於表格名稱及其直欄的表單。
4. 輸入 "cities" 作為表格名稱。將 [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) 檔案中的直欄定義複製並貼上到直欄和資料類型的對應方框中。
5. 按一下**建立**以定義新表格。   
   ![](images/solution5/TableCitiesCreated.png)

## 載入資料
既然已建立了 "cities" 表格，現在要將資料載入到該表格中。這可以採用不同的方式完成，例如從本端機器載入、使用 Swift 或 AmazonS3 介面從 Cloud Object Storage (COS) 載入，或利用 [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift) 移轉服務載入。在本指導教學中，將從機器上傳資料。在此程序中，可以調整表格結構和資料格式，以完全符合檔案內容。

1. 在頂端導覽中，按一下**載入**。然後，在**檔案選擇**下，按一下**瀏覽檔案**以找到並選取在本手冊第一區段中下載的 "cities1000.txt" 檔案。
2. 按一下**下一步**以前往綱目概觀。再次選擇以 "DASH" 開頭的綱目，然後選擇 "CITIES" 表格。再次按**下一步**。   

   因為表格是空的，所以附加到現有資料或改寫現有資料都是一樣的效果。
   {:tip }
3. 現在，自訂在載入程序中如何解釋 "cities1000.txt" 檔案中的資料。首先，停用「第一行為標頭」，因為該檔案僅包含資料。接著，鍵入 "0x09" 作為分隔字元。這意味著檔案中的值將用制表符分隔。最後，選取 "YYYY-MM-DD" 作為日期格式。現在，所有內容應該類似於下面的擷取畫面 / 畫面。    
  ![](images/solution5/LoadTabSeparator.png)
4. 按一下**下一步**，即可檢閱載入設定。同意並按一下**開始載入**以開始將資料載入到 "CITIES" 表格中。系統將顯示進度。上傳資料後，應該只需幾秒鐘就可完成載入，隨即會顯示一些統計資料。   
   ![](images/solution5/LoadProgressSteps.png)

## 使用 SQL 驗證載入的資料
資料已載入到關聯式資料庫中。未發生任何錯誤，但仍然應該執行一些快速測試。請使用內建 SQL 編輯器來鍵入並執行一些 SQL 陳述式。

1. 在頂端導覽中，按一下**執行 SQL**。
   您可以不使用內建 SQL 編輯器，而改為藉由 {{site.data.keyword.dashdbshort_notm}}，在桌面或伺服器機器上使用以雲端為基礎和傳統的 SQL 工具。連線資訊可以在「設定」功能表中找到。某些工具甚至可以在「書籍」圖示（代表文件和說明）後面提供的功能表的「下載」區段中進行下載。
    {:tip }
2. 在「SQL 編輯器」中，鍵入或複製下列查詢：   
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   然後按**全部執行**按鈕。在「結果」區段中，顯示的行數應該與載入程序所報告的行數相同。   
3. 在「SQL 編輯器」中，在新行上輸入下列陳述式：
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. 在編輯器中，選取上述陳述式的文字。按一下**執行所選內容**按鈕。現在應該只會執行此陳述式，並在「結果」區段中，依國家/地區統計資料傳回某些資訊。

## 部署應用程式碼
資料庫應用程式的可隨時執行的程式碼位於[此 GitHub 儲存庫中](https://github.com/IBM-Cloud/cloud-sql-database)。請複製或下載此儲存庫，然後將其推送到 IBM Cloud。

1. 複製 Github 儲存庫：
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. 將應用程式推送到 IBM Cloud。您需要登入到向其佈建資料庫的位置、組織和空間。複製並貼上這些指令，一次一行。
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. 推送程序完成後，您應該能夠存取該應用程式。無需進一步配置。`manifest.yml` 檔案會指示 IBM Cloud 將應用程式與名為 "sqldatabase" 的資料庫服務連結在一起。

## 安全、備份和回復以及監視
{{site.data.keyword.dashdbshort_notm}} 是一項受管理服務。IBM 負責確保環境安全、處理每日備份和系統監視。在入門方案中，資料庫環境是多方承租戶設定，減少了針對使用者的管理和選項配置工作。但是，如果您使用的是其中一個企業方案，有[多個選項可管理使用者，配置更多資料庫安全](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html)以及[監視資料庫](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html)。   

除了傳統的管理選項外，[{{site.data.keyword.dashdbshort_notm}} 服務還提供了監視、使用者管理、公用程式、負載、儲存空間存取等的 REST API](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html)。可以在 "REST API" 下「書籍」圖示後面的功能表中，存取該 API 的可執行 Swagger 介面。一些可用於監視等功能的工具（例如，IBM Data Server Manager）甚至可以在該相同功能表的「下載」區段下進行下載。

## 測試應用程式
根據載入的資料集顯示縣/市資訊的應用程式已縮減至最小。此應用程式提供了一個搜尋表單，用於指定縣/市名稱和若干預先配置的縣/市。這些將轉換為 `/search?name=cityname`（搜尋表單）或 `/city/cityname`（直接指定的縣/市）。這兩個要求都在背景從相同的程式碼行進行處理。出於安全原因，將使用參數標記將 cityname 作為值傳遞給備妥的 SQL 陳述式。從資料庫中提取行，然後將其傳遞給 HTML 範本以進行呈現。

## 清理
若要清除本指導教學使用的資源，請執行以下步驟：
1. 造訪 [{{site.data.keyword.Bluemix_short}} 資源清單](https://{DomainName}/resources)。找到您的應用程式。
2. 按一下應用程式的「功能表」圖示，然後選擇**刪除應用程式**。在對話框視窗中，勾選要刪除相關 {{site.data.keyword.dashdbshort_notm}} 服務的勾號。
3. 按一下**刪除**按鈕。這將移除應用程式和資料庫服務，然後您將回到資源清單。

## 擴展指導教學
要延伸此應用程式嗎？以下是一些構想：
1. 提供對替代名稱的萬用字元搜尋。
2. 搜尋特定國家/地區中僅特定人口值的縣/市。
3. 藉由取代 CSS 樣式和延伸範本，變更頁面佈置。
4. 容許以表單為基礎建立新縣/市資訊，或容許更新現有資料，例如人口。

## 相關內容
* 文件：[IBM Knowledge Center for {{site.data.keyword.dashdbshort_notm}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.Db2_on_Cloud_long_notm}} 和 {{site.data.keyword.dashdblong_notm}} 常見問題集](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html)，解答了與受管理服務、資料備份、資料加密和安全等相關的問題。
* [Free Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions)（適用於開發人員）
* 文件：[ibm_db Python 驅動程式的 API 說明](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
