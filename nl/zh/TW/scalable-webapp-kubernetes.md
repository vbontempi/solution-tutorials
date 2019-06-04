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

# Kubernetes 上的可調整 Web 應用程式
{: #scalable-webapp-kubernetes}

本指導教學將全程指導您如何搭建 Web 應用程式支架、在容器中本端執行 Web 應用程式，然後將其部署到使用 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster) 建立的 Kubernetes 叢集。此外，您將瞭解如何連結自訂網域、監視環境的性能和調整應用程式。
{:shortdesc}

容器是包裝應用程式及其所有相依關係的標準方式，而此方式可讓您在環境之間無縫移動應用程式。容器與虛擬機器不同，容器不會搭載作業系統。只有應用程式碼、運行環境、系統工具、程式庫及設定才會包裝在容器中。容器比虛擬機器更輕量、可攜性更高且更有效率。


對於希望啟動自己專案的開發人員，{{site.data.keyword.dev_cli_notm}} CLI 透過產生範本應用程式，讓您立即執行或自訂為自己的解決方案的入門範本，以便能快速開發和部署應用程式。除了產生入門範本應用程式碼、docker 容器映像檔和 CloudFoundry 資產外，開發 CLI 和 Web 主控台使用的程式碼產生器還可產生檔案，以輔助部署到 [Kubernetes](https://kubernetes.io/) 環境中。這些範本產生 [Helm](https://github.com/kubernetes/helm) 圖表，用於說明應用程式的起始 Kubernetes 部署配置，並可根據需要輕鬆延伸以建立多映像檔或複雜的部署。

## 目標
{: #objectives}

* 搭建入門範本應用程式支架。
* 將應用程式部署到 Kubernetes 叢集。
* 連結自訂網域。
* 監視叢集的日誌和性能。
* 調整 Kubernetes pod。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

<p style="text-align: center;">

  ![架構](images/solution2/Architecture.png)
</p>

1. 開發人員使用 {{site.data.keyword.dev_cli_notm}} 產生入門範本應用程式。
1. 透過建置應用程式，產生 docker 容器映像檔。
1. 映像檔被推送到 {{site.data.keyword.containershort_notm}} 中的名稱空間。
1. 應用程式部署到 Kubernetes 叢集。
1. 使用者存取應用程式。

## 開始之前
{: #prereqs}

* [設定 {{site.data.keyword.registrylong_notm}} CLI 和登錄名稱空間](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [安裝 {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - 用於安裝 docker、kubectl、helm、ibmcloud cli 和必要外掛程式的 Script
* [學習 Kubernetes 基礎知識](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## 建立 Kubernetes 叢集
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} 藉由結合 Docker 和 Kubernetes 技術、直覺式使用者體驗以及內建安全和隔離來提供功能強大的工具，以在運算主機的叢集中自動部署、操作、擴充及監視容器化應用程式。


本指導教學的主要部分可以使用**免費**叢集來完成。與 Kubernetes Ingress 和自訂網域相關的兩個選用區段需要**標準**類型的**付費**叢集。

1. 透過 [{{site.data.keyword.Bluemix}} 型錄](https://{DomainName}/containers-kubernetes/launch)，建立 Kubernetes 叢集。

   為了方便使用，請檢查配置詳細資料，如使用精簡方案和標準方案時獲得的個 CPU 數、記憶體和工作者節點數。
   {:tip}

   ![在 IBM Cloud 上建立 Kubernetes 叢集](images/solution2/KubernetesClusterCreation.png)
2. 選取**叢集類型**，然後按一下**建立叢集**來佈建 Kubernetes 叢集。
3.  檢查**叢集**和**工作者節點**的狀態，並等待它們成為**就緒**。

### 配置 kubectl

在此步驟中，您將會配置 kubectl，指向剛建立的叢集。[kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) 是一個指令行工具，您用來與 Kubernetes 叢集互動。

1. 使用 `ibmcloud login` 以互動方式登入。提供建立叢集所在的組織、位置及空間。您可以執行 `ibmcloud target` 指令，重新確認詳細資料。
2. 叢集準備就緒後，透過將 MYCLUSTER 環境變數設定為您的叢集名稱來擷取叢集配置：
   ```bash
   export MYCLUSTER=<CLUSTER_NAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
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


## 建立入門範本應用程式
{: #create_application}

`ibmcloud dev` 工具透過產生具有所有必需的樣板、建置和配置程式碼的應用程式入門範本，大幅縮短開發時間，以便能更快開始對商業邏輯進行編碼。

1. 啟動 `ibmcloud dev` 精靈。
   ```
   ibmcloud dev create
   ```
   {: pre}

1. 選取`後端服務 / Web 應用程式` > `Java - MicroProfile / JavaEE` > `Java Web App with Eclipse MicroProfile and Java EE（Web 應用程式）`來建立 Java 入門範本。（若要改為建立 Node.js 入門範本，請使用`後端服務 / Web 應用程式` > `Node`> `Node.js Web App with Express.js（Web 應用程式）`）
1. 輸入應用程式的**名稱**。
1. 選取要在其中部署此應用程式的資源群組。
1. 不要新增其他服務。
1. 不要新增 DevOps 工具鏈，請選取**手動部署**。

這將產生一個入門範本應用程式，其中包含程式碼和所有必要的配置檔，用於在 Cloud Foundry 或 Kubernetes 上進行本端開發和部署到雲端。 

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### 建置應用程式

您可以如常建置並執行應用程式，使用 `mvn` 進行 java 本端開發或使用 `npm` 進行 node 開發。您也可以建置 Docker 映像檔並在容器中執行應用程式，以確定在本端和在雲端能一致地執行。請使用下列步驟來建置您的 Docker 映像檔。


1. 確保本端 docker 引擎已啟動。
   ```
   docker ps
   ```
   {: pre}
2. 切換到產生的專案目錄。
   ```
   cd <project name>
   ```
   {: pre}
3. 建置應用程式。
   ```
  ibmcloud dev build
  ```
   {: pre}

   這可能會需要幾分鐘來執行，因為會下載所有應用程式相依關係，且會建置 Docker 映像檔，其中包含您的應用程式和所有必要環境。

### 在本端執行應用程式

1. 執行容器。
   ```
  ibmcloud dev run
  ```
   {: pre}

   這會使用您的本端 Docker 引擎，來執行您在先前步驟中建置的 Docker 映像檔。
2. 容器啟動後，移至 `http://localhost:9080/`。如果建立的是 Node.js 應用程式，請移至 `http://localhost:3000/`。
  ![](images/solution2/LibertyLocal.png)

## 使用 Helm 圖表將應用程式部署到叢集
{: #deploy}

在此區段中，先將 docker 映像檔推送到 IBM Cloud 專用容器登錄，然後建立指向該映像檔的 Kubernetes 部署。

1. 透過列出登錄中的所有名稱空間來找到您的**名稱空間**。
   ```sh
ibmcloud cr namespaces
```
   {: pre}
   如果您有名稱空間，請記下其名稱以供稍後使用。如果沒有，請建立名稱空間。
   ```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. 將 MYNAMESPACE 和 MYPROJECT 環境變數分別設定為您的名稱空間和專案名稱。

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. 透過執行 `ibmcloud cr info` 來識別您的**容器登錄**（例如，us.icr.io）。
4. 將 MYREGISTRY 環境變量設定為您的登錄。
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. 建置 docker 映像檔並對其進行標示 (`-t`)。
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. 將 docker 映像檔推送到 IBM Cloud 上的容器登錄。
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. 在 IDE 上，導覽至 `chart\YOUR PROJECT NAME` 下的 **values.yaml**，並更新**映像檔儲存庫**值，以指向 IBM Cloud 容器登錄上您的映像檔。**儲存**該檔案。

   若要取得映像檔儲存庫詳細資料，請執行 `echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}`。

8. [Helm](https://helm.sh/) 透過 Helm 圖表協助管理 Kubernetes 應用程式，它可協助定義、安裝和升級最複雜的 Kubernetes 應用程式。透過導覽至 `chart\YOUR PROJECT NAME`，並在叢集中執行以下指令來起始設定 Helm：

   ```bash
   helm init
   ```
   {: pre}
   若要升級 Helm，請執行以下指令：`helm init --upgrade`
   {:tip}

9. 若要安裝 Helm 圖表，請切換到 `chart\YOUR PROJECT NAME` 目錄並執行以下指令：
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. 使用 `kubectl get service ${MYPROJECT}-service`（對於 Java 應用程式）和 `kubectl get service ${MYPROJECT}-application-service`（對於 Node.js 應用程式）來識別服務要接聽的公用埠。埠為 5 位數（例如，31569），位於 `PORT(S)` 下。
11. 要取得工作者節點的公用 IP，請執行以下指令：
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. 在 `http://worker-ip-address:portnumber/` 上存取應用程式。

## 將 IBM 提供的網域用於叢集
{: #ibm_domain}

在先前的步驟中，應用程式是使用非標準埠進行存取的。服務透過 Kubernetes NodePort 特性公開。

付費叢集隨附 IBM 提供的網域。這為您提供了一個更好的選擇，可以使用正確的 URL 在標準 HTTP/S 埠上公開應用程式。

使用 Ingress 設定叢集到服務的入埠連線。

![Ingress](images/solution2/Ingress.png)

1. 識別 IBM 提供的 **Ingress 網域**
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   以找到
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. 建立 Ingress 檔案 `ingress-ibmdomain.yml`，以指向您的支援 HTTP 和 HTTPS 的網域。使用下列檔案作為範本，並將所有用 <> 括起的值取代為上面輸出中適當的值。**service-name** 是上面步驟中 `==> v1/Service` 下的名稱，或者執行 `kubectl get svc` 來尋找類型為 **NodePort** 的服務名稱。
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-ibmdomain-http-and-https
   spec:
     tls:
     - hosts:
       -  <nameofproject>.<ingress-sub-domain>
       secretName: <ingress-secret>
     rules:
     - host: <nameofproject>.<ingress-sub-domain>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: codeblock}
3. 部署 Ingress
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. 在 `https://<nameofproject>.<ingress-sub-domain>/` 上存取應用程式

## 使用您自己的自訂網域
{: #custom_domain}

若要使用自訂網域，需要使用指向 IBM 提供的網域的 CNAME 記錄，或指向 IBM 提供的 Ingress 的可攜式公用 IP 位址的 A 記錄來更新 DNS 記錄。考慮到付費叢集隨附固定 IP 位址，因此 A 記錄是一個不錯的選擇。

如需相關資訊，請參閱[將 Ingress 控制器用於自訂網域](https://{DomainName}/docs/containers?topic=containers-ingress#ingress)。

### 使用 HTTP

1. 建立 Ingress 檔案 `ingress-customdomain-http.yml`，以指向您的網域：
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-customdomain-http
   spec:
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
2. 部署 Ingress
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. 在 `http://<customdomain>/` 上存取應用程式

### 使用 HTTPS

如果此時要嘗試使用 HTTPS 在 `https://<customdomain>/` 上存取應用程式，可能會在 Web 瀏覽器中收到安全警告，指出連線不是專用的。此外，您還會收到 404 錯誤，因為剛才配置的 Ingress 並不知道如何導向 HTTPS 資料流量。

1. 為網域取得可信的 SSL 憑證。您將需要憑證和金鑰：
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3
   可以使用 [Let's Encrypt](https://letsencrypt.org/) 來產生可信憑證。
2. 將憑證和金鑰儲存在 base64 ASCII 格式的檔案中。
3. 建立 TLS 密碼以儲存憑證和金鑰：
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<custom-domain.cert> --key=<custom-domain.key>
   ```
   {: pre}
4. 建立 Ingress 檔案 `ingress-customdomain-https.yml`，以指向您的網域：
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-customdomain-https
   spec:
     tls:
     - hosts:
       - <my-custom-domain.com>
       secretName: <my-custom-domain-secret-name>
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
5. 部署 Ingress：
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. 在 `https://<customdomain>/` 上存取應用程式。

## 監視應用程式性能
{: #monitor_application}

1. 若要檢查應用程式的性能，請導覽至[叢集](https://{DomainName}/containers-kubernetes/clusters)以查看叢集清單，然後按一下在上面建立的叢集。
2. 按一下 **Kubernetes 儀表板**以在新標籤中啟動該儀表板。
   ![](images/solution2/launch_kubernetes_dashboard.png)
3. 選取左側窗格上的**節點**，按一下節點的**名稱**，然後查看**配置資源**可瞭解節點的性能。
   ![](images/solution2/KubernetesDashboard.png)
4. 若要檢閱容器中的應用程式日誌，請選取 **pod** > **pod-name**，然後選取**日誌**。
5. 若要透過 **SSH** 登錄到容器，請識別上一步中您的 pod 名稱，然後執行該 pod：
   ```sh
   kubectl 執行程式-it <pod-name> -- bash
   ```
   {: pre}

## 調整 Kubernetes pod
{: #scale_cluster}

隨著應用程式負載的增加，可以手動增加部署中的 pod 抄本數。抄本由 [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) 進行管理。若要將應用程式擴展為兩個抄本，請執行下列指令：

   ```sh
 kubectl scale deployment <nameofproject>-deployment --replicas=2
   ```
   {: pre}

片刻之後，您將在 Kubernetes 儀表板中（或透過 `kubectl get pods`）看到用於您應用程式的兩個 pod。叢集中的 Ingress 控制器將處理兩個抄本之間的負載平衡。此外，還可以使水平調整自動執行。

如需手動和自動調整的資訊，請參閱 Kubernetes 文件：

   * [Scaling a deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## 移除資源

* 刪除叢集，或者如果計劃重複使用叢集，請僅刪除為應用程式建立的 Kubernetes 構件。

## 相關內容

* [IBM Cloud Kubernetes Service](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [持續部署至 Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
