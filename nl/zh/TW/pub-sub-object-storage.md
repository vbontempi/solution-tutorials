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

# 使用 Object Storage 和發佈/訂閱傳訊進行非同步資料處理
{: #pub-sub-object-storage}
在本指導教學中，您將瞭解如何使用以 Apache Kafka 為基礎的傳訊服務，將長時間執行的工作負載編排到在 Kubernetes 叢集裡執行的應用程式。此模式用於使應用程式分離，以便能更進一步控制規模調整和效能。{{site.data.keyword.messagehub}} 可用於對要執行的工作進行佇列，而不會影響生產者應用程式，因此是用於長時間執行的作業的理想系統。 

{:shortdesc}

您將使用檔案處理範例來模擬此模式。請先建立一個使用者介面應用程式，用於將檔案上傳到 Object Storage，並產生指示要執行的工作的訊息。接下來，將建立一個個別的工作者應用程式，用於在收到訊息時，非同步處理使用者上傳的檔案。

## 目標
{: #objectives}

* 使用 {{site.data.keyword.messagehub}} 實作生產者/消費者模式
* 將服務連結到 Kubernetes 叢集

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

在本指導教學中，使用者介面應用程式是用 Node.js 撰寫的，而工作者應用程式是用 Java 撰寫的，這突顯出此模式的彈性。盡管在本指導教學中這兩個應用程式都在同一 Kubernetes 叢集中執行，但其中任一應用程式也可以實作為 Cloud Foundry 應用程式或無伺服器函數。

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. 使用者會利用使用者介面應用程式上傳檔案。
2. 檔案儲存在 {{site.data.keyword.cos_full_notm}} 中。
3. 訊息傳送到 {{site.data.keyword.messagehub}} 主題，指示新檔案正在等待處理。
4. 準備就緒後，工作者將接聽訊息並開始處理新檔案。

## 開始之前
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - 用於安裝 {{site.data.keyword.cloud_notm}} CLI、Kubernetes、Helm 和 docker 的工具。

## 建立 Kubernetes 叢集
{: #create_kube_cluster}

1. 透過[型錄](https://{DomainName}/containers-kubernetes/launch)，建立 Kubernetes 叢集。將其命名為 `mycluster`，以方便遵循此指導教學。此指導教學可以使用**免費**叢集完成。
   ![在 IBM Cloud 上建立 Kubernetes 叢集](images/solution25/KubernetesClusterCreation.png)
2. 檢查**叢集**和**工作者節點**的狀態，並等待它們成為**就緒**。

### 配置 kubectl

在此步驟中，您將會配置 kubectl，指向剛建立的叢集。[kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) 是一個指令行工具，您用來與 Kubernetes 叢集互動。

1. 使用 `ibmcloud login` 以互動方式登入。提供建立叢集所在的組織、位置及空間。您可以執行 `ibmcloud target` 指令，重新確認詳細資料。
2. 叢集準備就緒後，擷取叢集配置：
   ```bash
   ibmcloud cs cluster-config <cluster-name>
   ```
   {: pre}
3. 複製並貼上 **export** 指令，如指示設定 KUBECONFIG 環境變數。若要驗證 KUBECONFIG 環境變數是否設定適當，請執行下列指令：
  `echo $KUBECONFIG`
4. 確認 `kubectl` 指令已正確地配置
   ```bash
   kubectl cluster-info
   ```
  {: pre}
   ![](images/solution2/kubectl_cluster-info.png)

## 建立 {{site.data.keyword.messagehub}} 實例
 {: #create_messagehub}

{{site.data.keyword.messagehub}} 是一種以 Apache Kafka 為基礎的傳訊服務，具有快速、可調整和完全受管理的特點；Apache Kafka 是一種開放程式碼高傳輸量傳訊系統，為處理即時資料輸送提供了低延遲的平台。

 1. 在儀表板中，按一下[**建立資源**](https://{DomainName}/catalog/)，然後從「應用程式服務」區段中，選取 [**{{site.data.keyword.messagehub}}**](https://{DomainName}/catalog/services/event-streams)。
 2. 將服務命名為 `mymessagehub`，然後按一下**建立**。
 3. 將服務實例連結至 `default` Kubernetes 名稱空間，以將服務認證提供給您的叢集。
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

cluster-service-bind 指令會建立叢集密碼，用於以 JSON 格式保存服務實例的認證。使用 `kubectl get secrets` 可看到所產生名稱為 `binding-mymessagehub` 的密碼。如需相關資訊，請參閱[整合服務](https://{DomainName}/docs/containers?topic=containers-integrations#integrations)。

{:tip}

## 建立 Object Storage 實例

{: #create_cos}

{{site.data.keyword.cos_full_notm}} 經過加密，分散在多個地理位置，並使用 REST API 透過 HTTP 進行存取。{{site.data.keyword.cos_full_notm}} 為非結構化資料提供了有彈性、具成本效益且可擴充的雲端儲存空間。您將用這來儲存透過使用者介面上傳的檔案。

1. 在儀表板中，按一下[**建立資源**](https://{DomainName}/catalog/)，然後從「儲存空間」區段中，選取 [**{{site.data.keyword.cos_short}}**](https://{DomainName}/catalog/services/cloud-object-storage)。
2. 將服務命名為 `myobjectstorage`，然後按一下**建立**。
3. 按一下**建立儲存區**。
4. 將儲存區名稱設定為唯一名稱，例如 `username-mybucket`。
5. 選取**跨地區**備援，及選取 **us-geo**位置，然後按一下**建立**。
6. 將服務實例連結至 `default` Kubernetes 名稱空間，以將服務認證提供給您的叢集。
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## 將使用者介面應用程式部署到叢集

使用者介面應用程式是一個簡單的 Node.js Express Web 應用程式，容許使用者上傳檔案。此應用程式將檔案儲存在上面建立的 Object Storage 實例中，然後向 {{site.data.keyword.messagehub}} 主題 "work-topic" 傳送一條訊息，指示新檔案已準備好可供處理。

1. 在本端複製範例應用程式儲存庫，然後將目錄切換到 `pubsub-ui` 資料夾。
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. 開啟 `config.js`，並使用您的儲存區名稱更新 COSBucketName。
3. 建置並部署應用程式。deploy 指令會產生 docker 映像檔，將其推送到 {{site.data.keyword.registryshort_notm}}，然後建立 Kubernetes 部署。部署應用程式時，請按照互動式指示進行操作。
```sh
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
4. 造訪應用程式，並上傳 `sample-files` 資料夾中的檔案。上傳的檔案將儲存在 Object Storage 中，在工作者應用程式對這些檔案進行處理前，檔案的狀態將一直為「正在等待」。請使此瀏覽器視窗保持開啟。

   ![](images/solution25/files_uploaded.png)

## 將工作者應用程式部署到叢集

工作者應用程式是一個 Java 應用程式，用於接聽 {{site.data.keyword.messagehub}} Kafka "work-topic" 主題上的訊息。收到新訊息時，工作者將在訊息中擷取檔案的名稱，然後從 Object Storage 中取得檔案內容。接著，工作者將模擬對檔案的處理，並在完成時向 "result-work" 主題傳送另一條訊息。使用者介面應用程式將接聽此主題並更新狀態。

1. 將目錄切換到 `pubsub-worker` 目錄。
```sh
  cd ../pubsub-worker
```
2. 開啟 `resources/cos.properties`，並使用您的儲存區名稱更新 `bucket.name` 內容。
2. 建置並部署工作者應用程式。
```
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
3. 部署完成後，在瀏覽器視窗中再次檢查 Web 應用程式。請注意，現在每個檔案旁邊的狀態已變更為「已處理」。![](images/solution25/files_processed.png)

在本指導教學中，您瞭解了如何使用以 Kafka 為基礎的 {{site.data.keyword.messagehub}} 來實作生產者/消費者模式。這容許 Web 應用程式快速執行處理，並將繁重的處理卸載到其他應用程式。需要執行工作時，生產者（Web 應用程式）會建立訊息，並使工作在訂閱訊息的一個以上的工作者之間進行負載平衡。您是使用在 Kubernetes 上執行的 Java 應用程式來執行處理的，但這些應用程式也可以是 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing)。在 Kubernetes 上執行的應用程式非常適合處理長時間執行和密集型工作負載，而 {{site.data.keyword.openwhisk_short}} 更適合短時間執行的處理程序。

## 移除資源
{:removeresources}

導覽至[資源清單](https://{DomainName}/resources/)，然後
1. 刪除 Kubernetes 叢集 `mycluster`
2. 刪除 {{site.data.keyword.cos_full_notm}} `myobjectstorage`
3. 刪除 {{site.data.keyword.messagehub}} `mymessagehub`
4. 從左側功能表中選取 **Kubernetes**，選取**登錄**，然後刪除 `pubsub-xxx` 儲存庫。

## 相關內容
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [管理對 Object Storage 的存取](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [{{site.data.keyword.messagehub}} data processing with {{site.data.keyword.openwhisk_short}}](https://github.com/IBM/openwhisk-data-processing-message-hub)
