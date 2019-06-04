---
copyright:
  years: 2018, 2019
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

# 建置、部署、測試和重新訓練預測的機器學習模型。
{: #create-deploy-retrain-machine-learning-model}
本指導教學會引導您完成下列處理程序：建置預測性機器學習模型、將其部署為 API 以用於應用程式、測試模型，並使用意見資料重新訓練模型。所有這些作業都在 IBM Cloud 的整合、統一的自助服務體驗中進行。

在本指導教學中，將使用 **Iris flower data set** 來建立機器學習模型以對花的物種進行分類。

在機器學習術語中，分類會視為受監督學習的一個例子，亦即有正確識別之觀察訓練集可用的學習方式。
{:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## 目標
{: #objectives}

* 將資料匯入專案。
* 建置機器學習模型。
* 部署模型並試用 API。
* 測試機器學習模型。
* 為持續學習和模型評估建立意見資料連線。
* 重新訓練您的模型。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## 開始之前
{: #prereqs}
* IBM Watson Studio 和 Watson Knowledge Catalog 應用程式屬於 IBM Watson 的一部分。若要建立 IBM Watson 帳戶，請先註冊這兩個應用程式之一，或兩者都註冊。

   移至[試用 IBM Watson](https://dataplatform.ibm.com/registration/stepone) 並註冊 IBM Watson 應用程式。

## 將資料匯入專案

{:#import_data_project}

專案是您組織資源以實現特定目標的方式。您的專案資源可以包括資料、合作人員和分析工具，例如 Jupyter Notebook 和機器學習模型。

您可以建立專案以新增資料，並在資料修正器中開啟資料資產以進行資料清理和整理。

**建立專案：**

1. 移至 [{{site.data.keyword.Bluemix_short}} 型錄](https://{DomainName}/catalog)並選取 **AI** 區段下的 [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services)。**建立**服務。按一下**開始使用**按鈕以啟動 **{{site.data.keyword.DSX_short}}** 儀表板。
2. 建立**專案** > 按一下**標準**磚上的**建立專案**。新增專案的名稱（如 `iris_project`）和選用的說明。
3. 維持不勾選**限制誰可以為合作人員**勾選框，因為沒有機密資料。
4. 在**定義儲存空間**下，按一下**新增**，並選取現有的 Cloud Object Storage 服務，或者建立新的服務（選取**精簡**方案 >「建立」）。按一下**重新整理**，以查看建立的服務。
5. 按一下**建立**。您的新專案隨即開啟，可以開始新增資源到其中。

**匯入資料：**

如先前所提及，您將使用 **Iris 資料集**。Iris 資料集在 R.A. Fisher 1936 年發表的經典論文 [The Use of Multiple Measurements in Taxonomic Problems](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf) 中使用過，並且在 [UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/) 中也可以找到。這個小型資料集常常用於測試機器學習演算法和視覺化。其目標是根據萼片和花瓣的長度和寬度，將鳶尾花分類為 3 個物種（Setosa、Versicolor 或 Virginica）。此 iris 資料集包含各有 50 個實例的 3 個類別，其中每個類別對應一個鳶尾花種類別。![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


**下載** [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv)，其中包含每個類別的 40 個實例。您將使用每個類別的其餘 10 個實例來重新訓練模型。

1. 在專案的**資產**下，按一下**尋找和新增資料**圖示 ![顯示「尋找資料」圖示。](images/solution22-build-machine-learning-model/data_icon.png)。
2. 在**載入**下，按一下**瀏覽**並上傳已下載的 `iris_initial.csv`。![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. 一旦新增，您應該會在專案的**資料資產**區段下看到 `iris_initial.csv`。按一下名稱以查看資料集的內容。

## 關聯服務
{:#associate_services}
1. 在**設定**下，捲動到**關聯的服務** > 按一下**新增服務** > 選擇 **Spark**。![](images/solution22-build-machine-learning-model/associate_services.png)
2. 選取**精簡**方案，並按一下**建立**。使用預設值，並按一下**確認**。
3. 再次按一下**新增服務**，並選擇 **Watson**。按一下 **Machine Learning** 磚上的**新增** > 選擇**精簡**方案 > 按一下**建立**。
4. 保留預設值，並按一下**確認**以佈建 Machine Learning 服務。

## 建置機器學習模型

{:#build_model}

1. 按一下**新增到專案**，並選取 **Watson Machine Learning 模型**。在對話框中，新增 **iris_model** 作為名稱，並新增選用的說明。
2. 在 **Machine Learning 服務**區段下，您應該會看到在上述步驟中關聯的 Machine Learning 服務。![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. 選取**模型建置器**作為模型類型，並在 **Spark 服務或環境**區段下選取先前建立的 Spark 服務
4. 選取**手動**以手動建立模型。按一下**建立**。

   對於自動方法，您完全依賴於自動資料準備 (ADP)。對於手動方法，除了 ADP 轉換器處理的一些功能外，您還可以新增和配置自己的預估器，即在分析中使用的演算法。
   {:tip}

5. 在下一頁，選取 `iris_initial.csv` 作為資料集，並按**下一步**。
6. 在**選取技巧**頁面上，根據所新增的資料集，將預先移入「標籤直欄」和「特性直欄」。選取 **species (String)** 作為**標籤直欄**，**petal_length (Decimal)** 和 **petal_width (Decimal)** 作為**特性直欄**。
7. 選擇**多類別分類**作為建議的技巧。![](images/solution22-build-machine-learning-model/model_technique.png)
8. 對於**驗證分割**，配置下列設定：

   **訓練：** 50%、
   **測試：**25%、
   **保留：**25%

9. 按一下**新增預估器**，並選取**決策樹狀結構分類器**，然後選取**新增**。

   您一次可以評估多個預估器。例如，您可以新增**決策樹狀結構分類器**和**隨機森林分類器**作為預估器以訓練模型，並根據評估輸出選擇最適合項目。
{:tip}

10. 按**下一步**以訓練模型。看到狀態為**已訓練及已評估**後，按一下**儲存**。![](images/solution22-build-machine-learning-model/trained_model.png)

11. 按一下**概觀**以檢查模型的詳細資料。

## 部署模型並試用 API

{:#deploy_model}

1. 在已建立的模型下，按一下**部署** > **新增部署**。
2. 選擇 **Web 服務**。新增名稱（如 `iris_deployment`）和選用的說明。
3. 按一下**儲存**。在概觀頁面上，按一下新的 Web 服務名稱。當狀態為 **DEPLOY_SUCCESS** 時，您可以在**實作**下查看評分端點、各種程式設計語言的程式碼 Snippet 以及 API 規格。
4. 按一下**檢視 API 規格**以檢視和測試 {{site.data.keyword.pm_short}} API 端點。![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   若要開始使用 API，您需要使用 [{{site.data.keyword.Bluemix_short}} 資源清單](https://{DomainName}/resources/)下 {{site.data.keyword.pm_short}} 服務實例的**服務認證**標籤上可用的**使用者名稱**和**密碼**來產生**存取記號**。遵循 API 規格頁面上提及的指示，以產生**存取記號**。
   {:tip}
5. 若要進行線上預測，請使用 `POST /online` API 呼叫。
   * 可以在 [{{site.data.keyword.Bluemix_short}} 資源清單](https://{DomainName}/resources/)下的 {{site.data.keyword.pm_short}} 服務的**服務認證**標籤上找到 `instance_id`。
   * `deployment_id` 和 `published_model_id` 在部署的**概觀**下。
   *  對於 `online_prediction_input`，使用以下 JSON

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * 按一下**試用**查看 JSON 輸出。

6. 使用 API 端點，您現在可以從任何應用程式呼叫此模型。

## 測試您的模型

{:#test_model}

1. 在**測試**下，您應該會看到自動移入輸入資料（特性資料）。
2. 按一下**預測**，您應該會看到圖表中包含**物種的預測值**。![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. 對於 JSON 輸入和輸出，按一下作用中輸入和輸出旁的圖示。
4. 您可以變更輸入資料，並繼續測試模型。

## 建立意見資料連線

{:#create_feedback_connection}

1. 對於持續學習和模型評估，您需要將新資料儲存在某個位置。建立 [{{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse) 服務 > **入門**方案，以充當意見資料連線。
2. 在 {{site.data.keyword.dashdbshort}} **管理**頁面上，按一下**開啟**。在頂端導覽上，選取**載入**。
3. 按一下**我的電腦**下的**瀏覽檔案**，並上傳 `iris_initial.csv`。按**下一步**。
4. 選取 **DASHXXXX**（例如 DASH1234）作為**綱目**，然後按一下**新建表格**。將其命名為 `IRIS_FEEDBACK`，並按**下一步**。
5. 將自動偵測資料類型。按**下一步**，然後按一下**開始載入**。![](images/solution22-build-machine-learning-model/define_table.png)
6. 將建立新目標 **DASHXXXX.IRIS_FEEDBACK**。

   在下一步重新訓練模型以實現更好的效能和精準度時，將使用此表格。

## 重新訓練模型

{:#retrain_model}

1. 回到 [{{site.data.keyword.Bluemix_short}} 資源清單](https://{DomainName}/resources)，在您一直使用的 {{site.data.keyword.DSX_short}} 服務下，按一下**專案** > iris_project >  **iris-model**（「資產」下）> 評估。
2. 在**效能監視器**下，按一下**配置效能監視**。
3. 在「配置效能監視」頁面上：
   * 選取 Spark 服務。應該會自動移入預測類型。
   * 選擇 **weightedPrecision** 作為度量值，並設定 `0.98` 作為選用的臨界值。
   * 按一下**建立新的項目連線**以指向在上節中建立的雲端上 IBM Db2 Warehouse。
   * 選取 Db2 倉儲連線，一旦移入連線詳細資料後，按一下**建立**。![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * 按一下**選取意見資料參照**並指向 IRIS_FEEDBACK 表格，然後按一下**選取**。![](images/solution22-build-machine-learning-model/select_source.png)
   * 在**重新評估所需的記錄計數**方框中，鍵入觸發重新訓練所需最小數目的新記錄。使用 **10** 或留空以使用預設值 1000。
   * 在**自動重新訓練**方框中，選取下列某個選項：
     - 若要在模型效能低於所設定的臨界值時啟動自動重新訓練，請選取**模型效能低於臨界值時**。對於本指導教學，將在精準度低於臨界值 (.98) 時選擇此選項。
     - 若要禁止自動重新訓練，請選取**永不**。
     - 若要啟動自動重新訓練而不管效能如何，請選取**始終**。
   * 在**自動部署**方框中，選取下列某個選項：
     - 若要在每當模型效能高於先前版本時啟動自動部署，請選取**模型效能高於先前版本時**。對於本指導教學，選擇此選項是因為我們的目標是持續提高模型效能。
     - 若要禁止自動部署，請選取**永不**。
     - 若要啟動自動部署而不管效能如何，請選取**始終**。
   * 按一下**儲存**。![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. 下載檔案 [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv)。此後，按一下**新增意見資料**，並選取下載的 CSV 檔案，然後按一下**開啟**。
5. 按一下**新建評估**以開始。![](images/solution22-build-machine-learning-model/retraining_model.png)
6. 一旦評估完成後，您可以檢查**最後評估結果**區段以取得改進的 **WeightedPrecision** 值。

## 移除資源
{:removeresources}

1. 導覽至 [{{site.data.keyword.Bluemix_short}} 資源清單](https://{DomainName}/resources/) > 選擇您在其中建立服務的位置、組織和空間。
2. 刪除您為本指導教學建立的個別 {{site.data.keyword.DSX_short}}、{{site.data.keyword.sparks}}、{{site.data.keyword.pm_short}}、{{site.data.keyword.dashdbshort}} 和 {{site.data.keyword.cos_short}} 服務。

## 相關內容
{:related}

- [Watson Studio 概觀](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [使用 Machine Learning 偵測異常](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [自動模型建立](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [機器學習和 AI](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
