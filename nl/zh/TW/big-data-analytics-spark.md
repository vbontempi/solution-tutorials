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

# 使用 Apache Spark 分析和視覺化開放資料
{: #big-data-analytics-spark}

在本指導教學中，您將使用 {{site.data.keyword.DSX_full}}、Jupyter Notebook 和 Apache Spark 分析和視覺化開放資料集。首先，將說明人口增長、預期壽命和國家/地區 ISO 代碼的資料結合到單一資料訊框中。然後，為了探索見解，將使用名為 Pixiedust 的 Python 程式庫以各種方式查詢和視覺化資料。

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## 目標
{: #objectives}

* 在 IBM Cloud 上部署 Apache Spark 和 {{site.data.keyword.DSX_short}}
* 使用 Jupyter Notebook 和 Python 核心
* 匯入、轉換、分析和視覺化資料集

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 服務及環境設定
首先佈建本指導教學中使用的服務，並在 {{site.data.keyword.DSX_short}} 中建立專案。

可以從[資源清單](https://{DomainName}/resources)和[型錄](https://{DomainName}/catalog/)，佈建 {{site.data.keyword.Bluemix_short}} 的各種服務。或者，從 {{site.data.keyword.DSX_short}} 的儀表板和專案設定，可以建立或新增現有「資料及分析」服務。
{:tip}

1. 在 [{{site.data.keyword.Bluemix_short}} 型錄](https://{DomainName}/catalog)中，導覽至 **AI** 區段。建立 **{{site.data.keyword.DSX_short}}** 服務。按一下**開始使用**按鈕以啟動 **{{site.data.keyword.DSX_short}}** 儀表板。
2. 在儀表板中，按一下**建立專案**磚 > 選取**標準** >「建立專案」。在**名稱**欄位中，輸入 `1stProject` 作為名稱。您可以將說明保留空白。
3. 在頁面右側，可以**定義儲存空間**。如果已佈建儲存空間，請從清單中選取一個實例。如果未佈建，請按一下**新增**，然後遵循新瀏覽器標籤中的指示進行作業。完成服務建立後，按一下**重新整理**以查看新服務。
4. 按一下**建立**按鈕以按一下專案。這會將您重新導向到專案的概觀頁面。  
   ![](images/solution23/NewProject.png)
5. 在概觀頁面上，按一下**設定**。
6. 在**關聯的服務**區段中，按一下**新增服務**，然後從功能表中選取 **Spark**。在產生的畫面中，可以選擇現有的 Spark 服務實例，也可以建立新實例。

## 建立和準備記事本
[Jupyter Notebook](http://jupyter.org/) 是一種開放程式碼 Web 應用程式，容許建立和共用包含即時程式碼、方程式、視覺化和敘述文字的文件。記事本和其他資源會組織成專案。
1. 按一下**資產**標籤，並向下捲動至**記事本**區段，然後按一下**新建記事本**。
2. 使用**空白**記事本。輸入 `MyNotebook` 作為**名稱**。
3. 在**選取運行環境**功能表中，選擇已新增到專案設定的 Spark 實例。使預設**語言**保留為 **Python 3.5**。
4. 按一下**建立記事本**以完成該處理程序。
5. 在其中輸入文字和指令的欄位稱為**資料格**。將下列程式碼複製到空的資料格中，以匯入 [**Pixiedust** 套件](https://pixiedust.github.io/pixiedust/use.html)。藉由按一下工具列中的**執行**圖示，或按鍵盤上的 **Shift+Enter** 鍵來執行資料格。
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

如果您從未使用過 Jupyter Notebook，請按一下右上方功能表中的**文件**圖示。導覽至**分析資料**，然後導覽至[**記事本**區段](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics)，以進一步瞭解[記事本和其各部分](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true)。
{:tip}

## 載入資料
接下來載入三個開放資料集，並使其在記事本中可用。**Pixiedust** 程式庫讓您能輕鬆地[使用 URL 載入 **CSV** 檔案](https://pixiedust.github.io/pixiedust/loaddata.html)。

1.  將下行複製到記事本中的下個空的資料格中，但現在不要執行。
   ```Python
   df_pop = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
2. 在另一個瀏覽器標籤中，移至[社群](https://dataplatform.ibm.com/community?context=analytics)區段。在**資料集**下，搜尋 [**Total population by country**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e)，然後按一下相關磚。按一下右上方的**鏈結**圖示以取得存取 URI。複製該 URI，然後將記事本資料格中的文字 **YourAccessURI** 取代為該鏈結。按一下工具列中的**執行**圖示，或按 **Shift+Enter** 鍵。
3. 對另一個資料集重複此步驟。將下行複製到記事本中的下個空的資料格中。
   ```Python
   df_life = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
4. 在具有**資料集**的另一個瀏覽器標籤中，搜尋 [**Life expectancy at birth by country in total years**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895)。再次取得鏈結，並使用該鏈結取代記事本資料格中的 **YourAccessURI**，然後按一下**執行**以啟動載入處理程序。
5. 對於這三個資料集的最後一個資料集，從 Github 上的一組開放資料集載入國家/地區名稱及其 ISO 代碼清單。將以下程式碼複製到下一個空白記事本資料格，並執行該程式碼。
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

稍後將使用國家/地區碼清單來簡化資料選擇，因為選擇資料時將使用國家/地區碼，而不是確切的國家/地區名稱。

## 轉換資料
資料可用後，對其略做轉換，然後可將這三個集合結合成單一資料訊框。
1. 下列程式碼區塊將重新定義用於人口資料的資料訊框。這是使用對直欄重新命名的 SQL 陳述式所完成。然後，會建立視圖並列印綱目。將程式碼複製到下個空的資料格，然後執行它。
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. 對預期壽命資料重複相同作業。此程式碼不會顯示綱目，而會改為顯示前 10 行。  
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. 對國家/地區資料重複綱目轉換。
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. 現在，在可以結合成單一資料訊框的各資料集內，直欄名稱更簡單並保持一致。對預期壽命和人口資料執行**外部**結合。然後，在同一陳述式中，使用**內部**結合新增國家/地區碼。所有內容都將依國家/地區和年份排序。輸出會定義資料訊框 **df_all**。藉由使用內部結合，產生的資料僅包含可在 ISO 清單中找到的國家/地區。此處理程序將從資料中移除地區項目和其他項目。
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. 將**年份**的資料類型變更為整數。
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

現在結合的資料已準備好供分析。

## 分析資料
在此部分中，請[使用 Pixiedust 將資料視覺化為不同的圖表](https://pixiedust.github.io/pixiedust/displayapi.html)。首先，比較一些國家/地區的預期壽命。

1. 將程式碼複製到下個空的資料格，然後執行它。
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. 這將顯示一個可捲動的表格。按一下程式碼區塊正下方的「圖表」圖示，然後選取**折線圖**。這將顯示一個蹦現對話框，其中包含 **Pixiedust：折線圖選項**。輸入**圖表標題**，如「預期壽命比較」。從提供的**欄位**中，將**年份**拖曳至**索引鍵**方框，將**壽命**拖曳至**值**區域。對於**要顯示的行數**，輸入 **1000**。按**確定**以繪製折線圖。在右側，確保選取 **mapplotlib** 作為**展現器**。按一下**叢集依據**選取器，然後選擇**國家/地區**。這將顯示類似於下面的圖表。
   ![](images/solution23/LifeExpectancy.png)

3. 建立焦點位於 2010 年的圖表。將程式碼複製到下個空的資料格，然後執行它。
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. 在圖表選取器中，選擇**地圖**。在配置對話框中，將**國家/地區**拖曳至**索引鍵**區域。將**壽命**拖曳至**值**方框。與第一個圖表類似，將**要顯示的行數**增加到 **1000**。按**確定**以繪製地圖。選擇 **brunel** 作為**展現器**。這將顯示關於預期壽命而著色的世界地圖。您可以使用滑鼠來放大地圖。
   ![](images/solution23/LifeExpectancyMap2010.png)

## 擴展指導教學
下面是一些增強本指導教學的想法和建議。
* 建立並視覺化查詢，以顯示與您所選國家/地區人口增長相關的預期壽命率
* 在世界地圖上運算並視覺化每個國家/地區的人口增長率
* 從資料集型錄中載入和整合其他資料
* 將結合的資料匯出到檔案或資料庫

## 相關內容
{:related}
下面提供與本指導教學中涵蓋的主題相關的鏈結。
* [Watson Data 平台](https://dataplatform.ibm.com)：使用 Watson Data 平台可分工合作和建置更聰明的應用程式。您可快速視覺化資料、從資料中探索見解，並在不同團隊之間進行分工合作。
* [PixieDust](https://www.ibm.com/cloud/pixiedust)：用於 Jupyter Notebook 的開放程式碼生產力工具
* [Cognitive Class.ai](https://cognitiveclass.ai/)：資料科學和認知運算課程
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/)：我們利用資料實現的一切，您也能實現
* [Analytics Engine 服務](https://{DomainName}/catalog/services/analytics-engine)：使用開放程式碼 Apache Spark 和 Apache Hadoop 開發和部署分析應用程式
