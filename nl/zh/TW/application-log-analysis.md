---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 使用 LogDNA 和 Sysdig 分析日誌並監視應用程式性能
{: #application-log-analysis}

本指導教學說明可以如何使用 [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/) 服務來配置和存取部署在 {{site.data.keyword.Bluemix_notm}} 上的 Kubernetes 應用程式的日誌。您要將 Python 應用程式部署到 {{site.data.keyword.containerlong_notm}} 上佈建的叢集、配置 LogDNA 代理程式、產生不同層次的應用程式日誌，以及存取工作者節點日誌、Pod 日誌或網路日誌。然後，將透過 {{site.data.keyword.la_short}} Web 使用者介面來搜尋、過濾和視覺化這些日誌。

此外，您還將設定 [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/) 服務，並配置 Sysdig 代理程式以監視應用程式和 {{site.data.keyword.containerlong_notm}} 叢集的效能和性能。
{:shortdesc}

## 目標
{: #objectives}
* 將應用程式部署到 Kubernetes 叢集以產生日誌項目。
* 存取和分析不同類型的日誌，以對問題進行疑難排解，並搶先一步預防問題。
* 從營運角度瞭解應用程式以及執行應用程式的叢集的效能和性能。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

  ![](images/solution12/Architecture.png)

1. 使用者連接至應用程式並產生日誌項目。
1. 應用程式會根據儲存在 {{site.data.keyword.registryshort_notm}} 的映像檔，在 Kubernetes 叢集中執行。
1. 使用者將配置 {{site.data.keyword.la_full_notm}} 服務代理程式，以存取應用程式和叢集層次的日誌。
1. 使用者將配置 {{site.data.keyword.mon_full_notm}} 服務代理程式，以監視 {{site.data.keyword.containerlong_notm}} 叢集以及部署到叢集的應用程式的性能和效能。

## 必要條件
{: #prereq}

* [安裝 {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - 使用 Script 安裝 docker、kubectl、helm、ibmcloud cli 和必要的外掛程式。
* [設定 {{site.data.keyword.registrylong_notm}} CLI 及登錄名稱空間](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)。
* [授權使用者檢視 LogDNA 中日誌的許可權](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna)
* [授權使用者檢視 Sysdig 中度量值的許可權](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig)

## 建立 Kubernetes 叢集
{: #create_cluster}

{{site.data.keyword.containershort_notm}} 提供環境以在 Kubernetes 叢集裡執行的 Docker 容器，部署高可用性的應用程式。

如果您有現有的**標準**叢集，並想要在本指導教學中重複使用該叢集，請跳過本節。
{: tip}

1. 從 [{{site.data.keyword.Bluemix}} 型錄](https://{DomainName}/kubernetes/catalog/cluster/create)建立**新叢集**，然後選擇**標準**叢集。對於**免費**叢集，*不會* 啟用日誌轉遞。
   {:tip}
1. 選取資源群組和地理位置。
1. 為方便起見，請使用名稱 `mycluster`，以便與本指導教學保持一致。
1. 選取**工作者節點區域**，然後選取具有 2 個 **CPU** 和 4 GB **RAM** 的最小**機型**，此類型足以滿足本指導教學的需求。
1. 選取 1 個**工作者節點**，並使其他所有選項都設定為預設值。按一下**建立叢集**。
1. 檢查**叢集**和**工作者節點**的狀態，並等待狀態變為**準備就緒**。

## 佈建 {{site.data.keyword.la_short}} 實例
{: #provision_logna_instance}

部署到 {{site.data.keyword.Bluemix_notm}} 中 {{site.data.keyword.containerlong_notm}} 叢集的應用程式可能會產生某種層次的診斷輸出，即日誌。作為開發人員或操作員，您可能希望存取和分析不同類型的日誌，例如工作者節點日誌、Pod 日誌、應用程式日誌或網路日誌，以對問題進行疑難排解，並搶先一步預防問題。

藉由使用 {{site.data.keyword.la_short}} 服務，可以從各種來源聚集日誌，並根據需要保留任意長的時間。這容許在需要時分析「全局」，並對更複雜的狀況進行疑難排解。

若要佈建 {{site.data.keyword.la_short}} 服務，請執行下列動作：

1. 導覽至[觀察](https://{DomainName}/observe/)頁面，在**記載**下，按一下**建立實例**。
1. 提供唯一的**服務名稱**。
1. 選取地區/位置，然後選取資源群組。
1. 選取 **7 天日誌搜尋**作為方案，然後按一下**建立**。

服務提供一個集中日誌管理系統，其中日誌資料在 IBM Cloud 上管理。

## 部署 Kubernetes 應用程式並將其配置為轉遞日誌
{: #deploy_configure_kubernetes_app}

記載應用程式的可隨時執行程式碼位於[此 GitHub 儲存庫中](https://github.com/IBM-Cloud/application-log-analysis)。該應用程式是使用 [Django](https://www.djangoproject.com/) 撰寫的，這是一種常用的 Python 伺服器端 Web 架構。請複製或下載此儲存庫，然後將應用程式部署到 {{site.data.keyword.Bluemix_notm}} 上的 {{site.data.keyword.containershort_notm}}。

### 部署 Python 應用程式

在終端機上：
1. 複製 GitHub 儲存庫：
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. 切換到應用程式目錄：
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. 在 {{site.data.keyword.registryshort_notm}} 中使用 [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile) 建置 Docker 映像檔。
   - 使用 `ibmcloud cr info` 來尋找**容器登錄**，例如 us.icr.io 或 uk.icr.io。
   - 建立名稱空間以儲存容器映像檔。
      ```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - 將 `<CONTAINER_REGISTRY>` 取代為容器登錄值，並將 **app-log-analysis** 用作映像檔名稱。
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - 將 `app-log-analysis.yaml` 檔案中的 **image** 值取代為映像檔標籤 `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest`
1. 執行以下指令來擷取叢集配置，並設定 `KUBECONFIG` 環境變數：
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. 部署應用程式：
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. 若要存取應用程式，您需要工作者節點的`公用 IP` 和 `NodePort`
   - 若要取得公用 IP，請執行下列指令：
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - 若要取得 NodePort（這將為 5 位數，例如 3xxxx），請執行以下指令：
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   現在，可以在 `http://worker-ip-address:portnumber` 上存取應用程式

### 配置叢集以將日誌傳送到 LogDNA 實例

若要將您的 Kubernetes 叢集配置為傳送日誌到您的 {{site.data.keyword.la_full_notm}} 實例，您必須在叢集的每個節點上安裝 *logdna-agent* Pod。LogDNA 代理程式會從安裝它的 Pod 中讀取日誌檔，並將日誌資料轉遞至 LogDNA 實例。

1. 導覽至[觀察](https://{DomainName}/observe/)頁面，然後按一下**記載**。
1. 在您稍早建立的服務旁按一下**編輯日誌資源**，然後選取 **Kubernetes**。
1. 在已設定 `KUBECONFIG` 環境變數的終端機上複製並執行第一個指令，以使用服務實例的 LogDNA 汲取金鑰來建立 Kubernetes 密碼。
1. 複製並執行第二個指令，以在 Kubernetes 叢集的每個工作者節點上部署 LogDNA 代理程式。LogDNA 代理程式使用副檔名 **.log* 和無副檔名的檔案來收集日誌，這些檔案儲存在 Pod 的 */var/log* 目錄中。依預設，將從所有名稱空間（包括 kube-system）收集日誌，並自動將日誌轉遞到 {{site.data.keyword.la_full_notm}} 服務。
1. 配置日誌來源後，藉由按一下**檢視 LogDNA** 來啟動 LogDNA 使用者介面。您可能需要等待幾分鐘後才能開始看到日誌。

## 產生和存取應用程式日誌
{: generate_application_logs}

在本節中，您將產生應用程式日誌，並在 LogDNA 中進行檢閱。

### 產生應用程式日誌

在先前步驟中部署的應用程式容許以所選記載層次記載訊息。可用的記載層次為**嚴重**、**錯誤**、**警告**、**參考**和**除錯**。應用程式的記載基礎架構配置為僅容許傳遞等於或高於設定層次的日誌項目。最初，日誌程式層次設定為**警告**。因此，在伺服器設定為**警告**時，以**參考**層次記載的訊息不會顯示在診斷輸出中。

查看 [**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py) 檔案中的程式碼。程式碼包含 **print** 陳述式以及對 **logger** 函數的呼叫。顯示的訊息會寫入 **stdout** 串流（常規輸出，應用程式主控台/終端機），而日誌程式訊息會顯示在 **stderr** 串流（錯誤日誌）中。

1. 造訪 Web 應用程式：`http://worker-ip-address:portnumber`。
1. 藉由提交不同層次的訊息，可產生多個日誌項目。使用者介面還容許變更伺服器記載層次的日誌程式設定。將伺服器端記載層次變更為中間的值以使其更相關。例如，可以將 "500 internal server error" 記載為**錯誤**，或將 "This is my first log entry" 記載為**參考**。

### 存取應用程式日誌

在 LogDNA 使用者介面中，可以使用過濾器來存取應用程式特定的日誌。

1. 在頂端列，按一下**所有應用程式**。
1. 在「容器」下，勾選 **app-log-analysis**。這將顯示新的未儲存視圖，其中包含所有層次的應用程式日誌。
1. 若要查看特定記載層次的日誌，請按一下**所有層次**，然後選取多個層次，如「錯誤」、「參考」、「警告」等。

## 搜尋和過濾日誌
{: #search_filter_logs}

依預設，{{site.data.keyword.la_short}} 使用者介面會顯示所有可用的日誌項目（全部）。透過自動重新整理，最新的項目會顯示在底部。在本節中，您將修改顯示哪些內容以及顯示多少內容，然後將其另存為**視圖**以供未來使用。

### 搜尋日誌

1. 在位於 LogDNA 使用者介面中頁面底部的**搜尋**輸入框中，
   - 可以搜尋包含特定文字的行，如 "**This is my first log entry**" 或 **500 internal server error**。
   - 或者藉由輸入 `level:info`（其中 level 是接受字串值的欄位）來搜尋特定的記載層次。

   如需其他搜尋欄位和說明，請按一下搜尋輸入框旁邊的「語法說明」圖示
   {:tip}
1. 若要跳至特定時間範圍，請在**跳至時間範圍**輸入框中，輸入 **5 mins ago**。按一下輸入框旁邊的圖示可尋找保留期間內的其他時間格式。
1. 若要強調顯示項目，請按一下**切換檢視器工具**圖示。
1. 在第一個輸入框中輸入 **error** 作為強調顯示項目，並在第二個輸入框中輸入 **container** 作為強調顯示項目，然後查看包含這兩項的強調顯示行。
1. 按一下**切換時間表**圖示以查看包含特定時間的行。

### 過濾日誌

您可以依標籤、來源、應用程式或層次來過濾日誌。

1. 在頂端列，按一下**所有標籤**，然後選取 **k8s** 勾選框可查看與 Kubernetes 相關的日誌。
1. 按一下**所有來源**，然後選取要查看其日誌的主機（工作者節點）的名稱。這非常適用於您的叢集中有多個工作者節點的情況。
1. 若要查看容器或檔案日誌，請按一下**所有應用程式**，然後選取要查看其日誌的勾選框。

### 建立視圖

視圖是一組特定過濾器和搜尋查詢的已儲存快速鍵。

只要搜尋或過濾日誌，就應該會在頂端列看到**未儲存的視圖**。若要將其另存為視圖，請執行下列動作：
1. 按一下**所有應用程式**，然後選取 **app-log-analysis** 旁邊的勾選框。
1. 按一下**未儲存的視圖** > 按一下**另存為新視圖/警示**，然後將視圖命名為 **app-log-analysis-view**。將**種類**保留空白。
1. 按一下**儲存視圖**，隨後新視圖應該會出現在左側窗格中，並顯示應用程式的日誌。

### 使用圖形和分解視覺化日誌

在本節中，您將建立一個板，然後在其中新增一個具有分解的圖形，以視覺化應用程式層次資料。板是圖形和分解的集合。

1. 在左側窗格中，按一下**板**圖示（在「設定」圖示上方）> 按一下**新建板**。
1. 按一下頂端列的**編輯**，然後將其命名為 **app-log-analysis-board**。按一下**儲存**。
1. 按一下**新增圖形**：
   - 在第一個輸入框中輸入 **app** 作為欄位，然後按 Enter 鍵。
   - 選擇 **app-log-analysis** 作為欄位值。
   - 按一下**新增圖形**。
1. 選取**計數**作為度量值，以查看過去 24 小時內每個時間間隔的行數。
1. 若要新增分解，請按一下圖形下方的箭頭：
   - 選擇**直方圖**作為分解類型。
   - 選擇**層次**作為欄位類型。
   - 按一下**新增分解**以查看包含為應用程式記載的所有層次的分解。

## 新增 {{site.data.keyword.mon_full_notm}} 並監視叢集
{: #monitor_cluster_sysdig}

下面，您將向應用程式新增 {{site.data.keyword.mon_full_notm}}。該服務會定期檢查應用程式的可用性和回應時間。

1. 導覽至[觀察](https://{DomainName}/observe/)頁面，在**監視**下，按一下**建立實例**。
1. 提供唯一的**服務名稱**。
1. 選取地區/位置，然後選取資源群組。
1. 選取**累進層級**作為方案，然後按一下**建立**。
1. 在您稍早建立的服務旁按一下**編輯日誌資源**，然後選取 **Kubernetes**。
1. 在已設定 `KUBECONFIG` 環境變數的終端機上，複製並執行**將 Sysdig 代理程式安裝到叢集**下的指令，以在叢集中部署 Sysdig 代理程式。請等待部署完成。

### 配置 {{site.data.keyword.mon_short}}

若要配置 Sysdig 以監視叢集的性能和效能，請執行下列動作：
1. 按一下**檢視 Sysdig**，您應該會看到 Sysdig 監視器使用者介面。在歡迎使用頁面上，按**下一步**。
1. 在設定環境下，選擇 **Kubernetes** 作為安裝方法。
1. 按一下代理程式配置成功訊息旁邊的**移至下一步**，然後在下一頁上，按一下**開始使用**。
1. 按**下一步**，然後按一下**完成上線**以顯示 Sysdig 使用者介面中的`探索`標籤。

### 監視叢集

若要檢查應用程式和叢集的性能和效能，請執行下列動作：
1. 回到在 `http://worker-ip-address:portnumber` 上執行的應用程式，產生多個日誌項目。
1. 在 Sysdig 監視器精靈上，展開左側窗格中的 **mycluster** > 展開 **default** 名稱空間 > 按一下 **app-log-analysis-deployment** 以查看「要求計數」、「回應時間」等。
1. 若要查看 HTTP 要求/回應碼，請按一下頂端列上 **Kubernetes Pod 性能**旁邊的箭頭，然後在**應用程式**下，選取 **HTTP**。在 Sysdig 使用者介面的底端列上，將時間間隔變更為 **10 分**。
1. 若要監視應用程式的延遲，請執行下列動作：
   - 從「探索」標籤，選取**部署及 Pod**。
   - 按一下 `HTTP` 旁邊的箭頭，然後選取「度量值」>「網路」。
   - 選取 **net.http.request.time**。
   - 選取時間：**總和**以及選取群組：**平均**。
   - 按一下**其他選項**，然後按一下**拓蹼**圖示。
   - 按一下**完成**，然後按兩下方框以展開視圖。
1. 若要監視應用程式執行所在的 Kubernetes 名稱空間，請執行下列動作：
   - 從「探索」標籤，選取**部署及 Pod**。
   - 按一下 `net.http.request.time` 旁邊的箭頭。
   - 選取「預設儀表板」> Kubernetes。
   - 選取「Kubernetes 狀態」>「Kubernetes 狀態概觀」。

### 建立自訂儀表板

除了預先定義的儀表板之外，您還可以建立自己的自訂儀表板，以在單一位置顯示執行應用程式的容器的最有用/最相關的視圖和度量值。每個儀表板由一系列畫面組成，這些畫面配置為以多種不同的格式顯示特定資料。

若要建立儀表板，請執行下列動作：
1. 按一下最左側窗格中的**儀表板** > 按一下**新增儀表板**。
1. 按一下**空白儀表板** > 將儀表板命名為 **Container Request Overview** > 按一下**建立儀表板**。
1. 選取**排行榜**作為新畫面，並將該畫面命名為 **Request time per container**
   - 在**度量值**下，鍵入 **net.http.request.time**。
   - 範圍：按一下**置換儀表板範圍** > 選取 **container.image** > 選取 **is** > 選取_應用程式映像檔_
   - 依 **container.id** 進行分段，您應該會看到每個容器的淨要求時間。
   - 按一下**儲存**。
1. 若要新增新畫面，請按一下**加號**圖示，然後選取**數字 (#)** 作為畫面類型
   - 在**度量值**下，鍵入 **net.http.request.count** > 將時間聚集從**平均 (Avg)** 變更為**總和**。
   - 範圍：按一下**置換儀表板範圍** > 選取 **container.image** > 選取 **is** > 選取_應用程式映像檔_
   - 與 **1 小時**前進行比較，您應該會看到每個容器的淨要求計數。
   - 按一下**儲存**。
1. 若要編輯此儀表板的範圍，請執行下列動作：
   - 按一下標題畫面中的**編輯範圍**。
   - 在下拉清單中，選取/鍵入 **Kubernetes.cluster.name**
   - 將顯示名稱空白，然後選取 **is**。
   - 選取 **mycluster** 作為值，然後按一下**儲存**。

## 移除資源
{: #remove_resource}

- 從[觀察](https://{DomainName}/observe)頁面中移除 LogDNA 和 Sysdig 實例。
- 刪除包含工作者節點、應用程式和容器的叢集。這個動作無法復原。
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## 擴展指導教學
{: #expand_tutorial}

- 使用 [{{site.data.keyword.at_full}} 服務](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started)追蹤應用程式與 IBM Cloud 服務互動的情形。
- [新增警示](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts)至您的視圖。
- [匯出日誌](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export)至本端檔案。

## 相關內容
{:related}
- [重設 Kubernetes 叢集使用的汲取金鑰](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [將日誌保存至 IBM Cloud Object Storage](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [在 Sysdig 中配置警示](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [在 Sysdig 使用者介面中使用通知頻道](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
