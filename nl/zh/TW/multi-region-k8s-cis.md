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

# 使用 Cloud Internet Services 的具復原力且安全的多地區 Kubernetes 叢集
{: #multi-region-k8s-cis}

在設計應用程式時若能考慮到備援能力，使用者就不太可能會遇到關閉時間。使用 {{site.data.keyword.containershort_notm}} 實作解決方案時，可以利用各種內建功能（如負載平衡和隔離），提高對於主機、網路或應用程式潛在故障的備援能力。藉由建立多個叢集，以及一個叢集發生停機的情況下，使用者仍然可以存取也部署在另一個叢集裡的應用程式。利用位於不同位置的多個叢集，使用者還可以存取距離最近的叢集，並縮短網路延遲。為了獲得更多備援能力，還可以選取多區域叢集，這意味著節點部署在一個位置的多個區域中。

Cloud Internet Services (CIS) 是一個統一平台，用於為網際網路應用程式配置和管理網域名稱系統 (DNS)、廣域負載平衡 (GLB)、Web 應用程式防火牆 (WAF) 以及防止分散式阻斷服務 (DDoS)。本指導教學重點闡述了 CIS 如何與 Kubernetes 叢集整合以支援此情境，並在多個位置提供安全、具復原力的解決方案。

## 目標
{: #objectives}

* 在不同位置的多個 Kubernetes 叢集上部署一個應用程式。
* 使用廣域負載平衡器在多個叢集之間配送資料流量。
* 將使用者遞送到距離最近的叢集。
* 保護應用程式免受安全威脅。
* 透過快取提高應用程式效能。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

<p style="text-align: center;">
  ![架構](images/solution32-multi-region-k8s-cis/Architecture.png)
</p>

1. 開發人員建置應用程式的 Docker 映像檔。
2. 將映像檔推送到達拉斯和倫敦的 {{site.data.keyword.registryshort_notm}}。
3. 應用程式部署到兩個位置的 Kubernetes 叢集。
4. 一般使用者存取應用程式。
5. Cloud Internet Services 配置為截取對應用程式的要求，以及在叢集之間分散負載。此外，已啟用 DDoS 保護和 Web 應用程式防火牆，保護應用程式抵禦常見威脅。選擇性地快取資產，例如影像、CSS 檔。

## 開始之前
{: #prereqs}

* Cloud Internet Services 需要您擁有自訂網域，因此您可以設定此網域的 DNS，指向 Cloud Internet Services 名稱伺服器。
* [安裝 Git](https://git-scm.com/)。
* [安裝 {{site.data.keyword.Bluemix_notm}} CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)。
* [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - 透過 Script 安裝 docker、kubectl、helm、ibmcloud cli 和必要的外掛程式。
* [設定 {{site.data.keyword.registrylong_notm}} CLI 及登錄名稱空間](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)。
* [瞭解 Kubernetes 的基本觀念](https://kubernetes.io/docs/tutorials/kubernetes-basics/)。

## 將應用程式部署到一個位置

本指導教學將一個 Kubernetes 應用程式部署到多個位置的叢集。先部署到一個位置 - 達拉斯，然後對倫敦位置重複這些步驟。

### 建立 Kubernetes 叢集
{: #create_cluster}

若要建立叢集，請執行下列動作：
1. 從 [{{site.data.keyword.cloud_notm}} 型錄](https://{DomainName}/containers-kubernetes/catalog/cluster/create)中，選取 **{{site.data.keyword.containershort_notm}}**。
1. 將**位置**設定為**達拉斯**。
1. 選取**標準**叢集。
1. 選取一個以上的區域作為**位置**。建立多區域叢集可提高應用程式備援能力。應用程式分散在多個區域時，使用者不太可能會遇到關閉時間。有關多區域叢集的更多資訊可以在[這裡](https://{DomainName}/docs/containers?topic=containers-plan_clusters#ha_clusters)找到。
1. 將**機型**設定為可用的最小配置 - **2 個 CPU** 和 **4 GB RAM** 足以滿足本指導教學的需求。
1. 使用 **2** 個工作者節點。
1. 將**叢集名稱**設定為 **my-us-cluster**。請使用命名型樣 *`my-<location>-cluster`*，以便與本指導教學保持一致。

叢集準備就緒後，接著將準備應用程式。

### 在 {{site.data.keyword.registryshort_notm}} 中建立名稱空間

1. 將 {{site.data.keyword.Bluemix_notm}} CLI 的目標設定為達拉斯。
   ```bash
   ibmcloud target -r us-south
   ```
   {: pre}
2. 為應用程式建立名稱空間。
   ```bash
   ibmcloud cr namespace-add <your_namespace>
   ```
   {: pre}

如果在該位置具有現有名稱空間，還可以重複使用該名稱空間。您可以使用 `ibmcloud cr namespaces` 來列出現有名稱空間。
{: tip}

### 建置應用程式

此步驟將應用程式建置成 docker 映像檔。如果要配置第二個叢集，您可以跳過此步驟。這是簡單的 HelloWorld 應用程式。

1. 將 [Hello world 應用程式](https://github.com/IBM/container-service-getting-started-wt){:new_windows}的原始碼複製到使用者起始目錄。儲存庫在以 Lab 開頭的各個資料夾中分別包含類似應用程式的不同版本。
   ```bash
    git clone https://github.com/IBM/container-service-getting-started-wt.git
    ```
   {: pre}
1. 瀏覽到 `Lab 1` 目錄。
   ```bash
    cd 'container-service-getting-started-wt/Lab 1'
    ```
   {: pre}
1. 建置包含 `Lab 1` 目錄中應用程式檔案的 docker 映像檔。
   ```bash
   docker build --tag multi-region-hello-world:1 .
   ```
   {: pre}

### 準備將映像檔推送到位置特定的登錄

使用目標登錄標籤映像檔：

   ```bash
   docker tag multi-region-hello-world:1 us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### 將映像檔推送到位置特定的登錄

1. 確保本端 docker 引擎可以推送到達拉斯的登錄。
   ```bash
   ibmcloud cr login
   ```
   {: pre}
2. 推送映像檔。
   ```bash
   docker push us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### 將應用程式部署到 Kubernetes 叢集

在該階段，叢集應該已準備就緒。可以在 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/clusters) 主控台中查看其狀態。

1. 擷取叢集的配置：
   ```bash
   ibmcloud cs cluster-config <us-cluster-name>
   ```
   {: pre}
1. 複製並貼上輸出，以設定 KUBECONFIG 環境變數。該變數由 `kubectl` 使用。
1. 使用兩個抄本在叢集中執行應用程式：
   ```bash
   kubectl run hello-world-deployment --image=us.icr.io/<your_US-South_namespace>/hello-world:1 --replicas=2
   ```
   {: pre}
   範例輸出：`deployment "hello-world-deployment" created`。
1. 使應用程式在叢集中可存取
   ```bash
   kubectl expose deployment/hello-world-deployment --type=ClusterIP --port=80 --name=hello-world-service --target-port=8080
   ```
   {: pre}
   這將傳回類似於 `service "hello-world-service" exposed` 的訊息。

### 取得指派給叢集的網域名稱和 IP 位址
{: #CSALB_IP_subdomain}

建立 Kubernetes 叢集時，會為其指派 Ingress 子網域（例如，*my-us-cluster.us-south.containers.appdomain.cloud*）和公用應用程式負載平衡器 IP 位址。

1. 擷取叢集的 Ingress 子網域：
   ```bash
   ibmcloud cs cluster-get <us-cluster-name>
   ```
   {: pre}
   尋找 `Ingress Subdomain` 值。
1. 記下此資訊以在稍後步驟中使用。

本指導教學使用 Ingress 子網域來配置廣域負載平衡器。還可以將該子網域交換為公用應用程式負載平衡器 IP 位址 (`ibmcloud cs albs -cluster <us-cluster-name>`)。這兩個選項都受支援。
{: tip}

## 然後對另一個位置執行操作

在倫敦位置重複上述步驟，其中：
* 將位置名稱**達拉斯**取代為**倫敦**；
* 將位置別名 **us-south** 取代為 **eu-gb**；
* 將登錄 *us.icr.io* 取代為 **uk.icr.io**；
* 將叢集名稱 *my-us-cluster* 取代為 **my-uk-cluster**。

## 配置多位置負載平衡

現在，應用程式在兩個叢集中執行，但缺少一個元件，供使用者從單一進入點透明地存取任一叢集。

在此區段中，您將配置 Cloud Internet Services (CIS) 以在兩個叢集之間配送負載。CIS 是一個一站式服務，提供廣域負載平衡器 (GLB)、快取、Web 應用程式防火牆 (WAF) 和頁面規則來保護應用程式，同時確保雲端應用程式的可靠性和效能。

若要配置廣域負載平衡器，您將需要：
* 將自訂網域指向 CIS 名稱伺服器，
* 擷取 Kubernetes 叢集的 IP 位址或子網域名稱，
* 配置性能檢查，以驗證應用程式的可用性，
* 以及定義指向叢集的原始儲存區。

### 向 Cloud Internet Services 登錄自訂網域
{: #create_cis_instance}

第一步是建立 CIS 實例，並將自訂網域指向 CIS 名稱伺服器。

1. 如果您未擁有網域，可以從例如 [godaddy.com](http://godaddy.com) 的註冊商購買一個。
2. 導覽至 {{site.data.keyword.Bluemix_notm}} 型錄中的 [Internet Services](https://{DomainName}/catalog/services/internet-services)。
3. 設定服務名稱，然後按一下**建立**以建立服務的實例。
4. 佈建服務實例時，設定您的網域名稱，然後按一下**新增網域**。
5. 指派名稱伺服器時，請配置登記員或網域名稱提供者以便使用列出的名稱伺服器。
6. 配置登記員或 DNS 提供者之後，可能需要最多 24 小時，變更才會生效。

   當「概觀」頁面上的網域狀態從 *Pending* 變更為 *Active* 時，您可以使用 `dig <your_domain_name> ns` 指令驗證新的名稱伺服器已生效。
   {:tip}

### 為廣域負載平衡器配置性能檢查

性能檢查有助於深入瞭解儲存區的可用性，以將資料流量遞送至性能良好儲存區。這些檢查會定期傳送 HTTP/HTTPS 要求，以及監視回應。

1. 在 Cloud Internet Services 儀表板中，導覽至**可靠性** > **廣域負載平衡器**，然後按一下頁面底端的**建立性能檢查**。
1. 將**路徑**設定為 **/**。
1. 將**監視類型**設定為 **HTTP**。
1. 按一下**佈建 1 個實例**。

   建置您自己的應用程式時，可以定義一個專用的性能端點，例如 */heathz*，在其中可以報告應用程式狀態。
   {:tip}

### 定義原始儲存區

儲存區是在連接至 GLB 時將資料流量聰明地遞送至其中的原點伺服器群組。在英國和美國使用叢集時，可以定義以位置為基礎的儲存區，並配置 CIS 以根據使用者要求的地理位置，將使用者重新導向到距離最近的叢集。

#### 一個儲存區用於倫敦的叢集
1. 按一下**建立儲存區**。
2. 將**名稱**設定為 **UK**
3. 將**性能檢查**設定為前一節建立的性能檢查
4. 將**性能檢查地區**設定為**西歐**
5. 將**原點名稱**設定為 **uk-cluster**
6. 將**原點位址**設定為倫敦叢集的 Ingress 子網域，例如 *my_uk_cluster.eu-gb.containers.appdomain.cloud*
7. 按一下**佈建 1 個實例**。

#### 一個儲存區用於達拉斯的叢集
1. 按一下**建立儲存區**。
2. 將**名稱**設定為 **US**
3. 將**性能檢查**設定為前一節建立的性能檢查
4. 將**性能檢查地區**設定為**北美洲西部地區**
5. 將**原點名稱**設定為 **us-cluster**
6. 將**原點位址**設定為達拉斯叢集的 Ingress 子網域，例如 *my_us_cluster.us-south.containers.appdomain.cloud*
7. 按一下**佈建 1 個實例**。

#### 一個儲存區用於這兩個叢集
1. 按一下**建立儲存區**。
1. 將**名稱**設定為 **All**
1. 將**性能檢查**設定為前一節建立的性能檢查
1. 將**性能檢查地區**設定為**北美洲東部地區**
1. 新增兩個原點：
   1. 一個原點的**原點名稱**設定為 **us-cluster**，**原點位址**設定為達拉斯叢集的 Ingress 子網域
   2. 一個原點的**原點名稱**設定為 **uk-cluster**，**原點位址**設定為倫敦叢集的 Ingress 子網域
2. 按一下**佈建 1 個實例**。

### 建立廣域負載平衡器

定義原始儲存區後，可以完成負載平衡器的配置。

1. 按一下**建立負載平衡器**。
1. 在**平衡器主機名稱**下輸入廣域負載平衡器的名稱。這個名稱也將是您通用應用程式 URL (`http://<glb_name>.<your_domain_name>`) 的一部分，而不論其位置在何處。
1. 在**預設原始儲存區**下，按一下**新增儲存區**，然後新增名為 **All** 的儲存區。
1. 展開**配置地理路徑（可選）**的區段：
   1. 按一下**新增路徑**，選取**西歐**，然後按一下**新增**。
   1. 按一下**新增儲存區**以選取 **UK** 儲存區。
   1. 配置如下表中所示的其他路徑。
   1. 按一下**佈建 1 個實例**。

|地區|原始儲存區|
| :---------------:    | :---------: |
|西歐|英國|
|東歐|英國|
|東北亞|英國|
|東南亞|英國|
|北美洲西部|美國|
|北美洲東部|美國|

透過此配置，歐洲和亞洲的使用者將重新導向到倫敦的叢集，美國的使用者將重新導向到達拉斯叢集。要求與任何定義的路徑都不符合時，將重新導向到**預設原始儲存區**。

### 為每個位置的 Kubernetes 叢集建立 Ingress 資源

廣域負載平衡器現在已準備好處理要求。所有性能檢查都應該為綠色。但是，Kubernetes 叢集上需要最後一個配置步驟，才能正確回覆來自廣域負載平衡器的要求：需要定義一個 Ingress 資源，用於處理來自 GLB 網域的要求。

1. 建立名為 **glb-ingress.yaml** 的 Ingress 資源檔案
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: glb-ingress
   spec:
    rules:
      - host: <glb_name>.<your_domain_name>
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-world-service
              servicePort: 80
   ```
    {: pre}
    將 `<glb_name>.<your_domain_name>` 取代為在先前區段中定義的 URL。
1. 在為相應的位置叢集設定 KUBECONFIG 變數之後，在倫敦和達拉斯叢集中部署此資源：
   ```bash
   kubectl create -f glb-ingress.yaml
   ```
   {: pre}
   輸出的訊息為：`ingress.extension "glb-ingress" created`。

在此階段，已順利配置廣域負載平衡器搭配使用跨多個位置的 Kubernetes 叢集。可以存取 GLB URL `http://<glb_name>.<your_domain_name>` 來檢視應用程式。根據您的位置，會將您重新導向到距離最近的叢集 - 如果 CIS 無法將您的 IP 位址對映到特定位置，則會將您重新導向到預設儲存區中的叢集。

## 保護應用程式
{: #secure_via_CIS}

### 開啟 Web 應用程式防火牆

Web 應用程式防火牆 (WAF) 可保護 Web 應用程式免受 ISO 第 7 層攻擊。通常，WAF 與分組規則集組合使用，這些規則集旨在過濾掉惡意資料流量，以避免應用程式中有漏洞。

1. 在 Cloud Internet Services 儀表板中，導覽至**安全**，然後按一下**管理**標籤。
1. 在 **Web 應用程式防火牆**區段中，確保 WAF 已啟用。
1. 按一下**檢視 OWASP 規則集**。在此頁面中，可以檢閱 **OWASP 核心規則集**，並分別啟用或停用規則。規則已啟用後，如果進入的要求觸發了該規則，廣域威脅分數將增加。**靈敏度**設定將決定是否針對該要求觸發**動作**。
   1. 保留預設 OWASP 規則集不變。
   1. 將**靈敏度**設定為`低`。
   1. 將**動作**設定為`模擬`以記錄所有事件。
1. 按一下**回到安全**。
1. 按一下**檢視 CIS 規則集**。此頁面顯示關於管理網站之常見技術堆疊而建置的其他規則。

### 提高效能並防止阻斷服務攻擊 
{: #proxy_setting}

分散式阻斷服務 ([DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)) 攻擊是指透過在目標或其周圍基礎設施產生大量網際網路資料流量，惡意試圖中斷伺服器、服務或網路的正常通信。CIS 可用於保護網域免受 DDoS 攻擊。

1. 在 CIS 儀表板中，選取**可靠性** > **廣域負載平衡器**。
1. 在**負載平衡器**表格中找到建立的 GLB。
1. 啟用**Proxy**直欄中的「安全和效能」特性：

   ![CIS Proxy 切換開關開啟](images/solution32-multi-region-k8s-cis/cis-proxy.png)

**GLB 現在已受到保護**。直接的好處就是將會對用戶端隱藏叢集的原點 IP 位址。如果 CIS 偵測到未來的要求存在威脅，則在將使用者重新導向到應用程式之前，使用者可能會看到類似下面的畫面：

   ![正在驗證 - DDoS 防禦](images/solution32-multi-region-k8s-cis/cis-DDoS.png)

此外，現在可以控制 CIS 快取的內容以及保持快取的時間。移至**效能** > **快取**以定義廣域快取層次和瀏覽器有效期限。可以使用**頁面規則**來自訂廣域安全和快取規則。頁面規則使用特定網域路徑來啟用精細配置。例如，透過頁面規則，可以決定將 **/assets** 下的所有內容快取 **3 天**：

   ![頁面規則](images/solution32-multi-region-k8s-cis/cis-pagerules.png)

## 移除資源
{:removeresources}

### 移除 Kubernetes 叢集資源
1. 移除 Ingress。
1. 移除服務。
1. 移除部署。
1. 如果專門為此指導教學建立了叢集，則刪除這些叢集。

### 移除 CIS 資源
1. 移除 GLB。
1. 移除原點儲存區。
1. 移除性能檢查。

## 相關內容
{:related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [管理 IBM CIS 以取得最佳安全](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#best-practice-2-configure-your-security-level-selectively)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
* [{{site.data.keyword.registrylong_notm}} 基礎知識](https://{DomainName}/docs/services/Registry?topic=registry-registry_overview#registry_planning)
* [將單一實例應用程式部署到 Kubernetes 叢集](https://{DomainName}/docs/containers?topic=containers-cs_apps_tutorial#cs_apps_tutorial_lesson1)
* [透過 CIS 保護資料流量和網際網路應用程式的最佳作法](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#manage-your-ibm-cis-for-optimal-security)
* [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
