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

# 收集、視覺化和分析 IoT 資料
{: #gather-visualize-analyze-iot-data}
本指導教學會引導您設定 IoT 裝置、在 {{site.data.keyword.iot_short_notm}} 中收集資料、探索資料和建立視覺化，然後使用進階 Machine Learning 服務來分析資料並偵測歷程資料中的異常。
{:shortdesc}

## 目標
{: #objectives}

* 設定 IoT 模擬器。
* 將收集資料傳送到 {{site.data.keyword.iot_short_notm}}。
* 建立視覺化。
* 分析裝置產生的資料並偵測異常。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Node.js 應用程式](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* 使用 Spark 服務和 {{site.data.keyword.cos_full_notm}} 的 [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* 裝置使用 MQTT 通訊協定將感應器資料傳送到 {{site.data.keyword.iot_full}}
* 歷程資料將匯出到 {{site.data.keyword.cloudant_short_notm}} 資料庫中
* {{site.data.keyword.DSX_short}} 從此資料庫取回資料
* 透過 Jupyter Notebook 分析和視覺化資料

## 開始之前
{: #prereqs}

[{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - 執行 Script 以安裝 ibmcloud cli 和必要的外掛程式

## 建立 IoT 平台
{: #iot_starter}

首先，將建立 Internet of Things 平台服務 - 此中心可以管理裝置、安全地連接和**收集資料**，以及使歷程資料可供視覺化和應用程式使用。

1. 移至 [**{{site.data.keyword.Bluemix_notm}} 型錄**](https://{DomainName}/catalog/)，然後在 **Internet of Things** 區段下，選取 [**Internet of Things 平台**](https://{DomainName}/catalog/services/internet-of-things-platform)。
2. 輸入 `IoT demo hub` 作為服務名稱，並按一下**建立**，然後**啟動**儀表板。
3. 從側邊功能表中，選取**安全 > 連線安全**，並在**預設規則** > **安全層次**下選取 **TLS（選用）**，然後按一下**儲存**。
4. 從側邊功能表中，選取**裝置** > **裝置類型**和 **+ 新增裝置類型**。
5. 輸入 `simulator` 作為**名稱**，然後按**下一步**和**完成**。
6. 接下來，按一下**登錄裝置**。
7. 針對**選取現有項目裝置類型**選擇 `simulator`，然後針對**裝置 ID** 輸入 `phone`。
8. 按**下一步**，直到顯示**裝置安全**（在「安全」標籤下）畫面。
9. 為**鑑別記號**輸入值，例如：`myauthtoken`，然後按**下一步**。
10. 按一下**完成**後，將顯示連線資訊。請將此標籤保持開啟。

現在，IoT 平台已配置，可開始接收資料。裝置需要使用指定的裝置類型、ID 和記號將其資料傳送到 IoT 平台。

## 建立裝置模擬器
{: #confignodered}
接下來，將部署 Node.js Web 應用程式並在行動電話上對其進行造訪，該 Web 應用程式將連接至 IoT 平台，並向其傳送裝置加速感應器和方向資料。

1. 複製 Github 儲存庫：
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. 在所選的 IDE 中開啟程式碼，然後將 **manifest.yml** 檔案中的 `name` 和 `host` 值變更為唯一值。
3. 將應用程式推送到 {{site.data.keyword.Bluemix_notm}}。
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. 應用程式會在幾分鐘後部署完成，您應該會看到類似於 `<UNIQUE_NAME>.mybluemix.net` 的 URL。
5. 在行動電話上使用瀏覽器造訪此 URL。
6. 在**裝置認證**下，輸入「IoT 儀表板」標籤中的連線資訊，然後按一下**連線**。
7. 行動電話將開始傳輸資料。回到 **IBM {{site.data.keyword.iot_short_notm}} 標籤**，檢查**最近事件**區段中是否有新項目。
  ![](images/solution16/recent_events_with_phone.png)

## 在 IBM {{site.data.keyword.iot_short_notm}} 中顯示即時資料
{: #createcards}
接下來，您將建立一個板和多張卡片，以在儀表板中顯示裝置資料。如需板和卡片的相關資訊，請參閱[使用板和卡片顯示即時資料](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html)。

### 建立板
{: #createboard}

1. 開啟 **IBM {{site.data.keyword.iot_short_notm}} 儀表板**。
2. 從左側功能表中選取**板**，然後按一下**建立新的板**。
3. 輸入板的名稱，例如 `Simulators`，然後按**下一步**，再按一下**提交**。  
4. 選取剛才建立的板以將其開啟。

### 顯示裝置資料
{: #cardtemp}
1. 按一下**新增卡片**，然後選取**折線圖**卡片類型（位於「裝置」區段中）。
2. 從清單中選取裝置，然後按**下一步**。
3. 按一下**連接新資料集**。
4. 在「建立值卡片」頁面中，選取或輸入下列值，然後按**下一步**。
   - 事件：sensorData
   - 內容：ob
   - 名稱：OrientationBeta
   - 類型：Float
   - 下限：-180
   - 上限：180
5. 在「卡片預覽」頁面中，對於折線圖大小，選取 **L**，然後按**下一步** > **提交**。
6. 該卡片會顯示在儀表板中，並包含即時溫度資料的折線圖。
7. 使用行動電話瀏覽器重新啟動模擬器，然後慢慢向前和向後傾斜行動電話。
8. 回到 **IBM {{site.data.keyword.iot_short_notm}} 標籤**，您應該會看到圖表已更新。
   ![](images/solution16/board.png)

## 將歷程資料儲存在 {{site.data.keyword.cloudant_short_notm}} 中
1. 移至 [**{{site.data.keyword.Bluemix_notm}} 型錄**](https://{DomainName}/catalog/)，然後建立名為 `iot-db` 的新 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)。
2. 在**連線**下：
   1. **建立連線**
   1. 選取 Cloud Foundry 位置、組織和空間，在其中應該建立 {{site.data.keyword.cloudant_short_notm}} 服務的別名。
   1. 展開**連線位置**表格中的空間名稱，然後使用 **iot demo hub** 旁邊的**連線**按鈕為該空間中的 {{site.data.keyword.cloudant_short_notm}} 服務建立別名。 
   1. 連接應用程式並對其重新編譯打包。
3. 開啟 **IBM {{site.data.keyword.iot_short_notm}} 儀表板**。
4. 從左側功能表中選取**延伸**，然後按一下**歷程資料儲存空間**下的**設定**。
5. 選取 `iot-db` {{site.data.keyword.cloudant_short_notm}} 資料庫。
6. 對於**資料庫名稱**，輸入 `devicedata`，然後按一下**完成**。
7. 新視窗應該會載入要求授權的提示。如果看不到此視窗，請停用蹦現畫面封鎖程式並重新整理頁面。

現在，裝置資料已儲存在 {{site.data.keyword.cloudant_short_notm}} 中。幾分鐘後，啟動 {{site.data.keyword.cloudant_short_notm}} 儀表板即可看到您的資料。

![](images/solution16/cloudant.png)

## 使用機器學習偵測異常
{: #data_experience}

在本節中，您將使用 IBM {{site.data.keyword.DSX_short}} 服務中提供的 Jupyter Notebook 來載入曆程行動資料，並使用 z 評分來偵測異常。*Z 評分* 是一種標準評分，指出元素與平均值的標準差。

### 建立新專案
1. 移至 [**{{site.data.keyword.Bluemix_notm}} 型錄**](https://{DomainName}/catalog/)，然後在 **AI** 下，選取 [**{{site.data.keyword.DSX_short}}**](https://{DomainName}/catalog/services/data-science-experience)。
2. 藉由按一下**開始使用**，**建立**服務並啟動其儀表板。
3. 「建立專案」> 選取**資料科學** > 按一下**建立專案**，然後輸入 `Detect Anomaly` 作為專案的**名稱**。
4. 維持不勾選**限制誰可以為合作人員**勾選框，因為沒有機密資料。
5. 在**定義儲存空間**下，按一下**新增**，然後選取現有 **Cloud Object Storage** 服務或建立新的 Cloud Object Storage 服務（選取**精簡**方案 >「建立」）。按一下**重新整理**，以查看建立的服務。
6. 按一下**建立**。您的新專案隨即開啟，可以開始新增資源到其中。

### 連接到 {{site.data.keyword.cloudant_short_notm}} 以取得資料

1. 按一下**資產** > **+ 新增到專案** > **連線**。  
2. 選取裝置資料儲存所在的 **iot-db** {{site.data.keyword.cloudant_short_notm}}。
3. 交叉檢查**認證**，然後按一下**建立**。

### 建立 Apache Spark 服務

1. 按一下頂端導覽列上的**服務** >「運算服務」。
2. 按一下**新增服務**。
   1. 在 **Apache Spark** 上按一下**新增**。
   1. 選取**精簡**方案。
   1. 按一下**建立**。
3. 選取組織和空間，並根據需要變更服務名稱，然後**確認**。
1. 在**專案**中，導覽至 `Detect Anomaly` 專案。
1. 在**設定**中，捲動到**關聯的服務**。
1. 按一下**新增服務**，然後選取 **Spark**。
1. 選取先前安裝的 **Apache Spark** 實例。

### 建立 Jupyter (ipynb) 記事本
1. 按一下 **+ 新增到專案**，然後新增**記事本**。
2. 對於**名稱**，輸入 `Anomaly-detection-notebook`。
3. 在**記事本 URL** 中，輸入 `https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb`。
4. 選取先前作為運行環境關聯的 **Apache Spark** 服務。
5. 建立**記事本**。將 `Python 3.5 with Spark 2.1` 設定為核心。檢查是否使用 meta 資料和程式碼建立記事本。
   ![Jupyter Notebook Watson Studio](images/solution16/jupyter_notebook_watson_studio.png)
   若要進行更新，請按一下**核心** >「變更核心」。若要**信任**該記事本，請按一下**檔案** >「信任記事本」。
   {:tip}

### 執行記事本並偵測異常   
1. 選取以 `!pip install --upgrade pixiedust,` 開頭的資料格，然後按一下**執行**或按 **Ctrl+Enter** 鍵來執行程式碼。
2. 安裝完成後，藉由按一下**重新啟動核心**圖示，重新啟動 Spark 核心。
3. 在下一個程式碼資料格中，藉由完成下列步驟，將 {{site.data.keyword.cloudant_short_notm}} 認證匯入到該資料格中：
   * 按一下 ![](images/solution16/data_icon.png)
   * 選取**連線**標籤。
   * 按一下**插入到程式碼**。這將使用 {{site.data.keyword.cloudant_short_notm}} 認證建立名為 _credentials_1_ 的字典。如果該字典的名稱未指定為 _credentials_1_，請將其重新命名為 `credentials_1`。`credentials_1` 將在其餘資料格中使用。
4. 在具有資料庫名稱 (`dbName`) 的資料格中，輸入作為資料來源的 {{site.data.keyword.cloudant_short_notm}} 資料庫名稱，例如 *iotp_yourWatsonIoTProgId_DBName_Year-month-day*。若要視覺化不同裝置的資料，請相應地變更 `deviceId` 和 `deviceType` 的值。可以藉由導覽至先前建立的 **iot-db** {{site.data.keyword.cloudant_short_notm}} 實例 >「啟動儀表板」來尋找確切的資料庫。
   {:tip}
5. 儲存記事本，然後逐一執行每個程式碼資料格或執行所有資料格（**資料格** >「全部執行」），在記事本結束時，應該會看到裝置移動資料（oa、ob 和 og）中的異常。可以將關注的時間間隔變更為一天中所需的時間。請尋找 `start` 和 `end` 值。
   {:tip}
   ![Jupyter Notebook DSX](images/solution16/anomaly_detection_watson_studio.png)
6. 除了異常偵測外，本節中的主要發現項目或要點如下：
    * 使用 Spark 準備要進行視覺化的資料。
    * 使用 Pandas 進行資料視覺化。
    * 長條圖和直方圖用於表示裝置資料。
    * 透過「相關性」矩陣體現兩個感應器之間的相關性。
    * 一個裝置感應器一個箱型圖，使用 Pandas 繪圖功能產生。
    * 密度圖透過核心密度預估 (KDE) 產生。
    ![](images/solution16/density_plots_sensor_data.png)

## 移除資源
{:removeresources}

1. 導覽至[資源清單](https://{DomainName}/resources/) > 選擇在其中建立應用程式和服務的位置、組織和空間。在 **Cloud Foundry 應用程式**下，刪除上面建立的 Node.js 應用程式。
2. 在**服務**下，刪除為本指導教學建立的各個 Internet of Things 平台、Apache Spark、{{site.data.keyword.cloudant_short_notm}} 和 {{site.data.keyword.cos_full_notm}} 服務。

## 相關內容
{:related}

* [建置、部署、測試和重新訓練預測的機器學習模型。](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* [IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html) 概觀
* [Jupyter Notebook](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb) 異常偵測
* [瞭解 z 評分](https://en.wikipedia.org/wiki/Standard_score)
* 使用深度學習開發用於異常偵測的認知 IoT 解決方案 - [5 個帖子系列](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/)
