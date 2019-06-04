---
copyright:
  years: 2019
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

#                 隔離的 Cloud Foundry Enterprise 應用程式
            
{: #isolated-cloud-foundry-enterprise-apps}

使用 {{site.data.keyword.cfee_full_notm}} (CFEE)，您可以依需求建立多個隔離的企業級 Cloud Foundry 平台。借助這些平台，可在隔離的 Kubernetes 叢集上部署專用 Cloud Foundry 實例。與公用雲端不同的是，您將擁有對環境的完全控制權：存取控制、容量、版本、資源使用和監視。{{site.data.keyword.cfee_full_notm}} 使用在企業 IT 中找到的基礎架構所有權來提供平台即服務層次的速度和創新。

企業擁有的創新平台是 {{site.data.keyword.cfee_full_notm}} 的一個使用案例。您作為企業中的開發人員，可以建立新的微服務，也可以將舊式應用程式移轉到 CFEE。然後，可以使用 Cloud Foundry 市場向其他開發人員發佈微服務。發佈後，CFEE 實例中的其他開發人員可以在應用程式中使用服務，就像如今在為公用雲端上所做的一樣。

本指導教學會引導您完成下列處理程序：建立和配置 {{site.data.keyword.cfee_full_notm}}，並設定存取控制，然後部署應用程式和服務。您還將藉由部署自訂服務分配管理系統來整合自訂服務與 CFEE，以檢閱 CFEE 與 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) 之間的關係。

## 目標

{: #objectives}

- 比較和對照 CFEE 與公用 Cloud Foundry
- 在 CFEE 中部署應用程式和服務
- 瞭解 Cloud Foundry 與 {{site.data.keyword.containershort_notm}} 的關係
- 調查基本 Cloud Foundry 和 {{site.data.keyword.containershort_notm}} 網路

## 使用的服務

{: #services}

本指導教學使用下列運行環境及服務：

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構

{: #architecture}

![架構](./images/solution45-CFEE-apps/Architecture.png)

1. 管理建立 CFEE 實例，並新增具有開發人員存取權的使用者。
2. 開發人員將 Node.js 入門範本應用程式推送到 CFEE。
3. Node.js 入門範本應用程式使用 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 來儲存資料。
4. 開發人員新增「歡迎使用訊息」服務。
5. Node.js 入門範本應用程式從自訂服務分配管理系統連結新服務。
6. Node.js 入門範本應用程式從服務以不同語言顯示「歡迎使用」。

## 必要條件

{: #prereq}

- [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [Cloud Foundry CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## 佈建 {{site.data.keyword.cfee_full_notm}}

{:provision_cfee}

在本節中，您將建立 {{site.data.keyword.cfee_full_notm}} 實例，並從 {{site.data.keyword.containershort_notm}} 部署到 Kubernetes 工作者節點。

1. [準備 {{site.data.keyword.cloud_notm}} 帳戶](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare)，以確保建立必要的基礎架構資源。
2. 在 {{site.data.keyword.cloud_notm}}「型錄」中，建立 [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create) 的服務實例。
3. 藉由提供下列內容來配置 CFEE：
   - 選取**標準**方案。
   - 輸入服務實例的**名稱**。
   - 選取在其中建立環境的**資源群組**。您需要可存取帳戶中至少一個資源群組的許可權，才能建立 CFEE 實例。
   - 選取在其中部署實例的**地理位置**和**位置**。請參閱[可用佈建位置和資料中心](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets)的清單。
   - 選取將在其中部署 **{{site.data.keyword.composeForPostgreSQL}}** 的公用 Cloud Foundry **組織**和**空間**。
   - 選取用於 Cloud Foundry 環境的**資料格數目**。一個資料格可執行多個 Diego 和 Cloud Foundry 應用程式。對於高可用性應用程式，至少需要 **2** 個資料格。
   - 選取**機型**，這將確定 Cloud Foundry 資料格的大小（CPU 和記憶體）。
4. 檢閱**基礎架構**區段，以檢閱支援 CFEE 的 Kubernetes 叢集的內容。**工作者節點數目**等於資料格數目加 2。其中佈建的兩個 Kubernetes 工作者節點充當 CFEE 控制平面。部署環境的 Kubernetes 叢集將顯示在 {{site.data.keyword.cloud_notm}} [叢集](https://{DomainName}/containers-kubernetes/clusters)儀表板中。
5. 按一下**建立**按鈕以開始自動部署。

自動部署需要大約 90 到 120 分鐘。順利建立後，您將收到一封電子郵件，用於確認佈建 CFEE 和支援服務。

### 建立組織和空間

建立 {{site.data.keyword.cfee_full_notm}} 後，請參閱[建立組織和空間](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs)，以取得如何透過組織和空間建構環境的資訊。{{site.data.keyword.cfee_full_notm}} 中的應用程式是以特定空間為範圍。與此類似，空間存在於組織中。組織的成員會共用配額方案、應用程式、服務實例及自訂網域。

附註：位於 {{site.data.keyword.cloud_notm}} 頂端標頭中的**管理 > 帳戶 > Cloud Foundry 組織**功能表專門用於為公用 {{site.data.keyword.cloud_notm}} 組織。CFEE 組織在 CFEE 實例的**組織**頁面中進行管理。

執行以下步驟建立 CFEE 組織和空間。

1. 在 [Cloud Foundry 儀表板](https://{DomainName}/dashboard/cloudfoundry/overview)中的**企業**下，選取**環境**。
2. 選取您的 CFEE 實例，然後選取**組織**。
3. 按一下**建立組織**按鈕，並提供 `tutorial` 作為**組織名稱**，然後選取**配額方案**。最後按一下**新增**以便完成作業。
4. 按一下新建立的 `tutorial` 組織，並選取**空間**標籤，然後按一下**新建空間**按鈕。
5. 提供 `dev` 作為**空間名稱**，然後按一下**新增**。

### 向組織和空間新增使用者

在 CFEE 中，可以指派用於控制使用者存取權的角色，但要執行此作業，必須在 {{site.data.keyword.cloud_notm}} 標頭的**管理 > 使用者**下的**身分及存取**頁面中，邀請使用者加入您的 {{site.data.keyword.cloud_notm}} 帳戶。

邀請使用者後，執行以下步驟將使用者新增到建立的 `tutorial` 組織。 

1. 選取您的 [CFEE 實例](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments)，然後再次選取**組織**。
2. 從清單中選取建立的 `tutorial` 組織。
3. 按一下**成員**標籤來檢視使用者和新增使用者。
4. 按一下**新增成員**按鈕、搜尋使用者名稱、選取適當的**組織角色**，然後按一下**新增**。

可以在[這裡](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users)找到有關向 CFEE 組織和空間新增使用者的相關資訊。

## 部署、配置和執行 CFEE 應用程式

{:deploycfeeapps}

在本節中，您要將 Node.js 應用程式部署到 CFEE。部署後，將 {{site.data.keyword.cloudant_short_notm}} 連結到該應用程式，然後啟用審核和記載持續性。此外，還將新增「Stratos 主控台」來管理應用程式。

### 將應用程式部署到 CFEE

1. 在終端機中，複製 [get-started-node](https://github.com/IBM-Cloud/get-started-node) 範例應用程式。
   ```sh
git clone https://github.com/IBM-Cloud/get-started-node
  ```
   {: codeblock}
2. 在本端執行該應用程式，以確保它正確建置並啟動。請藉由在瀏覽器中存取 `http://localhost:3000/` 進行確認。
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. 登入 {{site.data.keyword.cloud_notm}}，並將您的 CFEE 實例設定為目標。互動式提示將協助您選取新的 CFEE 實例。由於只存在一個 CFEE 組織和一個空間，因此會將其作為預設目標。如果新增多個組織或空間，則可以執行 `ibmcloud target -o tutorial -s dev`。
   ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
   {: codeblock}
4. 將 **get-started-node** 應用程式推送到 CFEE。
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. 應用程式的端點將顯示在最終輸出中的 `routes` 內容旁邊。在瀏覽器中開啟 URL 以確認應用程式是否正在執行。

### 建立 Cloudant 資料庫並將其連結到應用程式

若要將 {{site.data.keyword.cloud_notm}} 服務連結到 **get-started-node** 應用程式，需要先在 {{site.data.keyword.cloud_notm}} 帳戶中建立服務。

1. 建立 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 服務。提供**服務名稱** `cfee-cloudant`，然後選擇在其中建立 CFEE 實例的位置。
2. 將新建立的 {{site.data.keyword.cloudant_short_notm}} 服務實例新增至 CFEE。
   1. 導覽回 `tutorial` **組織**。按一下**空間**標籤，然後選取 `dev` 空間。
   2. 選取**服務**標籤，然後按一下**新增服務**按鈕。
   3. 在搜尋文字框中鍵入 `cfee-cloudant`，然後選取結果。最後按一下**新增**以便完成作業。現在，服務可用於 CFEE 應用程式；但是，該服務仍位於為公用 {{site.data.keyword.cloud_notm}} 中。
3. 在顯示的服務實例的溢位功能表上，選取**連結至應用程式**。
4. 選取稍早推送的 **GetStartedNode** 應用程式，然後按一下**連結之後重新編譯打包應用程式**。最後，按一下**連結**按鈕。等待應用程式重新編譯打包。可以使用 `ibmcloud app show GetStartedNode` 指令來檢查進度。
5. 在瀏覽器中，存取應用程式、新增您的姓名，然後按 `Enter` 鍵。您的姓名將新增到 {{site.data.keyword.cloudant_short_notm}} 資料庫。
6. 藉由從**服務**標籤上的清單中選取 `tutorial` 實例進行確認。這將在公用 {{site.data.keyword.cloud_notm}} 中開啟服務實例的詳細資料頁面。
7. 按一下**啟動 Cloudant 儀表板**，然後選取 `mydb` 資料庫。應該存在具有您姓名的 JSON 文件。

### 啟用審核和記載持續性

透過審核，CFEE 管理者可以追蹤 Cloud Foundry 活動，例如登入、組織和空間建立、使用者成員資格和角色指派、應用程式部署、服務連結和網域配置。審核透過與 {{site.data.keyword.cloudaccesstrailshort}} 服務整合進行支援。

可以藉由整合 {{site.data.keyword.loganalysisshort_notm}} 來儲存 Cloud Foundry 應用程式日誌。由 CFEE 管理者選取的 {{site.data.keyword.loganalysisshort_notm}} 服務實例會自動配置為接收並持續保存從 CFEE 實例產生的 Cloud Foundry 記載事件。

若要啟用 CFEE 審核和記載持續性，請執行[這裡的步驟](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging)。

### 安裝「Stratos 主控台」來管理應用程式

「Stratos 主控台」是一個以 Web 為基礎的開放程式碼工具，用於對 Cloud Foundry 進行作業。可以選擇在特定的 CFEE 環境中安裝和使用「Stratos 主控台」應用程式，以管理組織、空間和應用程式。

若要安裝「Stratos 主控台」應用程式，請執行下列動作：

1. 開啟您要安裝 Stratos 主控台的 CFEE 實例。
2. 在**概觀**頁面上，按一下**安裝 Stratos 主控台**。此按鈕僅對具有管理者或編輯者權限的使用者顯示。
3. 在「安裝 Stratos 主控台」對話框中，選取安裝選項。您可以在 CFEE 控制平面上或其中一個資料格中安裝「Stratos 主控台」應用程式。選取「Stratos 主控台」版本，以及要安裝之應用程式的實例數。如果是在資料格中安裝「Stratos 主控台」應用程式，則系統將提示您提供部署應用程式的組織和空間。
4. 按一下**安裝**。

應用程式需要大約 5 分鐘才能安裝。安裝完成後，會顯示 **Stratos 主控台**按鈕，以取代「概觀」頁面上的**安裝 Stratos 主控台**按鈕。可以在[這裡](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app)找到有關「Stratos 主控台」的相關資訊。

## CFEE 與 Kubernetes 之間的關係

CFEE 作為一個應用程式平台，可以在某種形式的專用或共用虛擬基礎架構上執行。多年來，開發人員很少會考慮基礎 Cloud Foundry 平台，因為是 IBM 在管理此平台。使用 CFEE，您不僅是撰寫 Cloud Foundry 應用程式的開發人員，也是 Cloud Foundry 平台的營運者。這是因為 CFEE 部署在您所控制的 Kubernetes 叢集上。

雖然 Cloud Foundry 開發人員可能對 Kubernetes 並不熟悉，但這兩個產品共有的概念很多。與 Cloud Foundry 一樣，Kubernetes 也是將應用程式隔離到容器中，這些容器在稱為 Pod 的 Kubernetes 建構內執行。與應用程式實例類似，Pod 也可以具有多個抄本（稱為抄本集），使用由 Kubernetes 提供的應用程式負載平衡。您稍早部署的 Cloud Foundry `GetStartedNode` 應用程式在 `diego-cell-0` Pod 內執行。為了支援高可用性，另一個 Pod `diego-cell-1` 將在個別的 Kubernetes 工作者節點上執行。由於這些 Cloud Foundry 應用程式是在 Kubernetes「內部」執行，因此還可以使用以 Kubernetes 基礎的網路連線功能與其他 Kubernetes 微服務進行通訊。下列各節將有助於更詳細地說明 CFEE 與 Kubernetes 之間的關係。

## 部署 Kubernetes 服務分配管理系統

在本節中，您要將微服務部署到 Kubernetes 以充當 CFEE 的服務分配管理系統。[服務分配管理系統](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md)提供有關可用服務的詳細資料，以及有關向 Cloud Foundry 應用程式連結和佈建支援的詳細資料。您已使用內建 {{site.data.keyword.cloud_notm}} 服務分配管理系統新增 {{site.data.keyword.cloudant_short_notm}} 服務。現在，將部署並使用自訂服務。然後，將修改 `GetStartedNode` 應用程式以使用該服務分配管理系統，這將傳回多種語言的「歡迎使用」訊息。

1. 回到終端機，複製提供 Kubernetes 部署檔案和服務分配管理系統實作的專案。

   ```sh
   git clone https://github.com/IBM-Cloud/cfee-service-broker-kubernetes.git
   ```
   {: codeblock}

   ```sh
   cd cfee-service-broker-kubernetes
   ```
   {: codeblock}

   ```sh
   git clone https://github.com/IBM/sample-resource-service-brokers.git
   ```
   {: codeblock}

2. 在 {{site.data.keyword.registryshort_notm}} 上建置並儲存包含服務分配管理系統的 Docker 映像檔。使用 `ibmcloud cr info` 指令手動擷取登錄 URL，或使用下面的 `export REGISTRY` 指令自動擷取登錄 URL。`cr namespace-add` 指令將建立一個名稱空間來儲存 Docker 映像檔。

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   建立名為 `cfee-tutorial` 的名稱空間。

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   編輯 `./cfee-service-broker-kubernetes/deployment.yml` 檔案。更新 `image` 屬性以反映出容器登錄 URL。
3. 將容器映像檔部署到 CFEE 的 Kubernetes 叢集。CFEE 叢集存在於 `default` 資源群組中，如果尚未將其設定為目標，請執行此作業。使用您的叢集名稱，利用 `cluster-config` 指令匯出 KUBECONFIG 變數。然後建立部署。
   ```sh
   ibmcloud target -g default
   ```
   {: codeblock}

   ```sh
  ibmcloud ks clusters
  ```
   {: codeblock}

   ```sh
   $(ibmcloud ks cluster-config <your-cfee-cluster-name> --export)
   ```
   {: codeblock}

   ```sh
   kubectl apply -f deployment.yml
   ```
   {: codeblock}
4. 驗證 Pod 的 STATUS 是否為 `Running`。Kubernetes 可能需要一些時間來取回映像檔並啟動容器。請注意，您有兩個 pod，因為 `deployment.yml` 已要求 2 個`抄本`。
   ```sh
    kubectl get pods
    ```
   {: codeblock}

## 驗證服務分配管理系統是否已部署

既然已部署服務分配管理系統，請確認它是否正常運作。您將藉由多種方式來確認：先使用 Kubernetes 儀表板，然後藉由 Cloud Foundry 應用程式存取分配管理系統，最後是藉由分配管理系統實際連結服務。

### 在 Kubernetes 儀表板中檢視 pod

本節將使用 {{site.data.keyword.containershort_notm}} 儀表板來確認 Kubernetes 構件是否已配置。

1. 在 [Kubernetes 叢集](https://{DomainName}/containers-kubernetes/clusters)頁面中，藉由按一下以 CFEE 服務名稱開頭且以 **-cluster** 結尾的行項目來存取 CFEE 叢集。
2. 藉由按一下對應的按鈕來開啟 **Kubernetes 儀表板**。
3. 按一下左側功能表中的**服務**鏈結，然後選取 **tutorial-broker-service**。在稍早步驟中執行 `kubectl apply` 時，已部署此服務。
4. 在產生的儀表板中，注意下列內容：
   - 服務已提供層疊 IP 位址 (172.x.x.x)，該位址僅在 Kubernetes 叢集內才可解析。
   - 服務有兩個端點，對應於執行服務分配管理系統容器的兩個 pod。

在確認服務可用並正在代理服務分配管理系統 Pod 之後，可以驗證分配管理系統是否使用有關可用服務的資訊進行回應。

可以在 Kubernetes 儀表板中檢視 Cloud Foundry 相關構件。若要檢視這些構件，請按一下**名稱空間**以檢視具有包含 `cf` Cloud Foundry 名稱空間的構件的所有名稱空間。
{: tip}

### 從 Cloud Foundry 容器存取分配管理系統

為了示範 Cloud Foundry 到 Kubernetes 的通訊，將直接從 Cloud Foundry 應用程式連接至服務分配管理系統。

1. 回到終端機，使用 `ibmcloud target` 確認是否仍連接至 CFEE `tutorial` 組織和 `dev` 空間。如果需要，請將 CFEE 重新設定為目標。
   ```sh
   ibmcloud target --cf
   ```
   {: codeblock}
2. 依預設，空間中已停用 SSH。這與公用雲端不同，因此請在 CFEE `dev` 空間中啟用 SSH。
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. 使用 `kubectl` 指令顯示在 Kubernetes 儀表板中看到的相同 ClusterIP。然後透過 SSH 進入 `GetStartedNode` 應用程式，並使用該 IP 位址從服務分配管理系統擷取資料。請注意，最後一個指令可能會導致錯誤，此錯誤將在下一步中予以解析。
   ```sh
   kubectl get service tutorial-broker-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf ssh GetStartedNode
   ```
   {: codeblock}

   ```sh
   export CLUSTER_IP=<CLUSTER-IP address>
   ```
   {: codeblock}

   ```sh
   wget --user TestServiceBrokerUser --password TestServiceBrokerPassword -O- http://$CLUSTER_IP/v2/catalog
   ```
   {: codeblock}
4. 您可能收到 **Connection refused** 錯誤。導致此錯誤的原因是 CFEE 的預設[應用程式安全群組](https://docs.cloudfoundry.org/concepts/asg.html)。應用程式安全群組 (ASG) 定義用於 Cloud Foundry 容器中輸出資料流量的容許 IP 範圍。由於 `GetStartedNode` 不屬於預設範圍，因此會發生此錯誤。請結束 SSH 階段作業，然後下載 `public_networks` ASG。
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. 編輯 `public_networks.json` 檔案，並驗證正在使用的 ClusterIP 位址是否超出現有規則範圍。例如，範圍 `172.32.0.0-192.167.255.255` 可能不包含 ClusterIP，因此需要更新。例如，如果 ClusterIP 位址類似於 `172.21.107.63`，則需要編輯該檔案，使其具有類似於 `172.0.0.0-255.255.255.255` 的內容。
6. 調整 ASG `destination` 規則，以包含 Kubernetes 服務的 IP 位址。修整該檔案以僅包含 JSON 資料，此資料以括號開頭和結尾。然後上傳新的 ASG。
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. 重複步驟 3，現在應該可以順利模擬型錄資料。結束 SSH 階段作業以完成作業。
   ```sh
   exit
   ```
   {: codeblock}

### 向 CFEE 登錄服務分配管理系統

若要容許開發人員從服務分配管理系統來佈建和連結服務，請向 CFEE 登錄服務分配管理系統。先前，您已使用 IP 位址處理分配管理系統。但這會產生一個問題：服務分配管理系統重新啟動時，會收到新的 IP 位址，而這需要更新 CFEE。若要解決此問題，將使用另一個名為 KubeDNS 的 Kubernetes 特性，此特性為服務分配管理系統提供完整網域名稱 (FQDN)。

1. 使用 `tutorial-service-broker` 服務的 FQDN 向 CFEE 登錄服務分配管理系統。同樣，這是 CFEE Kubernetes 叢集的內部路徑。
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. 然後，新增分配管理系統提供的服務。由於範例分配管理系統只有一個模擬服務，因此只需要一個指令。
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. 在瀏覽器中，從[**環境**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments)頁面存取 CFEE 實例，然後導覽至`組織 -> 空間`，並選取 `dev` 空間。
4. 選取**服務**標籤，然後選取**建立服務**按鈕。
5. 在搜尋文字框中，搜尋**測試**。這將顯示分配管理系統中的**測試節點資源服務分配管理系統顯示名稱**模擬服務。
6. 選取該服務，並按一下**建立**按鈕，然後提供名稱 `welcome-service` 來建立服務實例。在下節中您會明白為什麼將其命名為 welcome-service。然後使用溢位功能表中的**連結到應用程式**項目，將該服務連結到 `GetStartedNode` 應用程式。
7. 若要檢視連結服務，請執行 `cf env` 指令。現在，`GetStartedNode` 應用程式可以像目前使用 `cloudantNoSQLDB` 中的資料一樣，利用 `credentials` 物件中的資料。
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## 向應用程式新增服務

到目前為止，您部署的是服務分配管理系統，但並不是實際服務。雖然服務分配管理系統向 `GetStartedNode` 提供連結資料，但未新增服務本身的實際功能。為瞭解決這個問題，建立一個模擬服務，用於向 `GetStartedNode` 提供各種語言的「歡迎使用」訊息。您要將此服務部署到 CFEE，並將分配管理系統和應用程式更新為使用此服務。

1. 將 `welcome-service` 實作推送到 CFEE，以容許其他開發團隊將其用作 API。
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. 使用瀏覽器存取該服務。服務的 `route` 將顯示在 `push` 指令的輸出中。產生的 URL 將類似於 `https://welcome.<your-cfee-cluster-domain>`。重新整理頁面可查看替代語言。
3. 回到 `sample-resource-service-brokers` 資料夾並編輯 Node.js 實作範例 `sample-resource-service-brokers/node-resource-service-broker/testresourceservicebroker.js`，在第 854 行之後新增 url，其值為叢集 URL。 

   ```javascript
   // TODO - Do your actual work here

   var generatedUserid   = uuid();
   var generatedPassword = uuid();
    
   result = {
     credentials: {
       userid   : generatedUserid,
       password : generatedPassword,
       url: 'https://welcome.<your-cfee-cluster-domain>'
     }
   };
   ```
   {: codeblock}
4. 建置並部署更新後的服務分配管理系統。這可確保將 URL 內容提供給連結該服務的應用程式。
   ```sh
   cd ..
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   ```sh
   kubectl patch deployment tutorial-servicebroker-deployment -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"version\":\"2\"}}}}}"
   ```
   {: codeblock}
5. 在終端機中，導覽回 `get-started-node` 應用程式資料夾。 
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. 編輯 `get-stared-node` 中的 `get-started-node/server.js` 檔案，以包含下列中介軟體。將其插入到 `app.listen()` 呼叫的正前方。
   ```javascript
   // Use the welcome service
   const request = require('request');
   const testService = appEnv.services['testnoderesourceservicebrokername'];

   if (testService) {
     const { credentials: { url} } = testService[0];
     app.get("/api/welcome", (req, res) => request(url, (e, r, b) => res.send(b)));
   } else {
     app.get("/api/welcome", (req, res) => res.send('Welcome'));
   }
   ```
   {: codeblock}
7. 重構 `get-started-node/views/index.html` 頁面。將 `h1` 標籤中的 `data-i18n="welcome"` 變更為 `id="welcome"`。然後，在 `$(document).ready()` 函數結尾新增對該服務的呼叫。
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. 更新 `GetStartedNode` 應用程式。包含已新增到 `server.js` 的 `request` 套件相依關係，重新連結 `welcome-service` 以選取新的 `url` 內容，最後推送應用程式的新程式碼。
   ```sh
   npm i request -S
   ```
   {: codeblock}

   ```sh
   ibmcloud cf unbind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf bind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}

完成了！現在，造訪該應用程式，並多次重新整理頁面以查看不同語言的歡迎使用訊息。

![服務分配管理系統回應](./images/solution45-CFEE-apps/service-broker.png)

如果仍然只看到 **Welcome**，而未顯示其他語言，請執行 `ibmcloud cf env GetStartedNode` 指令，並確認該服務的 `url` 是否存在。如果不存在，請重新追蹤步驟，並進行更新，然後重新部署分配管理系統。如果存在 `url` 值，請確認它是否在瀏覽器中傳回訊息。然後，重新執行 `unbind-service` 和 `bind-service` 指令，接著執行 `ibmcloud cf restart GetStartedNode`。
{: tip}

雖然歡迎使用服務使用 Cloud Foundry 作為其實作，但使用 Kubernetes 作為實作也很容易。主要區別在於服務的 URL 可能是 `welcome-service.default.svc.cluster.local`。使用 KubeDNS URL 還有一個額外的好處，即可讓網路資料流量流向至 Kubernetes 叢集內部的服務。

使用 {{site.data.keyword.cfee_full_notm}}，現在可以使用這些替代方法。

## 使用工具鏈部署此解決方案指導教學  

（選用）可以使用工具鏈來部署完整的解決方案指導教學。請遵循[工具鏈指示](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)使用工具鏈來部署以上所有內容。 

附註：使用工具鏈時有一些必要條件，您必須已建立 CFEE 實例，並已建立 CFEE 組織和 CFEE 空間。[工具鏈指示](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)ReadMe 中概述詳細的指示。 

## 相關內容
{:related}

* [在 Kubernetes 叢集中部署應用程式](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Cloud Foundry Diego Components and Architecture](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [CFEE Service Broker on Kubernetes with a Toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

