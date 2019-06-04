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

#                 將端對端安全套用至雲端應用程式
            
{: #cloud-e2e-security}

若要建置完整的應用程式架構，必須清楚瞭解潛在的安全風險以及如何防範此類威脅。應用程式資料是一種不能遺失、洩露或被盜的關鍵資源。此外，靜態和動態資料應透過加密技術進行保護。加密靜態資料可以防止資訊洩露，就算資訊遺失或被盜也不會外洩。透過 HTTPS、SSL 和 TLS 等方法加密動態資料（例如，在網際網路上傳輸的資料）可防止竊聽和所謂的攔截式攻擊。

在許多應用程式中，另一個常見要求是針對使用者對特定資源的存取權進行鑑別和授權。這可能需要支援不同的鑑別方法：使用社交身分的客戶和供應商、來自雲端管理的目錄的夥伴以及來自組織的身分提供者的員工。

本指導教學會引導您使用 {{site.data.keyword.cloud}}「型錄」中提供的關鍵安全服務，並說明如何將這些服務一起使用。提供檔案共用的應用程式將落實安全概念。
{:shortdesc}

## 目標
{: #objectives}

* 使用您自己的加密金鑰來加密儲存空間儲存區中的內容
* 要求使用者在存取應用程式之前進行鑑別
* 監視和審核雲端服務上與安全相關的 API 呼叫和其他動作

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* 選用項目：[{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

本指導教學需要[非精簡帳戶](https://{DomainName}/docs/account?topic=account-accounts#accounts)且可能產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

本指導教學提供一個範例應用程式，讓使用者群組將檔案上傳到共用儲存區，並透過可共用鏈結提供對這些檔案的存取。該應用程式是使用 Node.js 所撰寫，並作為 Docker 容器部署到 {{site.data.keyword.containershort_notm}}。它利用多種與安全相關的服務和特性來改善應用程式的安全狀態。

<p style="text-align: center;">

  ![架構](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. 使用者連接至應用程式。
2. 如果使用的是自訂網域和 TLS 憑證，則憑證由 {{site.data.keyword.cloudcerts_short}} 進行管理和部署。
3. {{site.data.keyword.appid_short}} 可保護應用程式並將使用者重新導向到鑑別頁面。使用者還可以進行註冊。
4. 應用程式會根據儲存在 {{site.data.keyword.registryshort_notm}} 的映像檔，在 Kubernetes 叢集中執行。系統會自動掃描此映像檔以找出漏洞。
5. 已上傳的檔案儲存在 {{site.data.keyword.cos_short}} 中，隨附的 meta 資料儲存在 {{site.data.keyword.cloudant_short_notm}} 中。
6. 檔案儲存空間儲存區利用使用者提供的金鑰來加密資料。
7. 應用程式管理活動由 {{site.data.keyword.cloudaccesstrailshort}} 進行記載。

## 開始之前
{: #prereqs}

1. [遵循下列步驟](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)以安裝所有必要的指令行 (CLI) 工具。
2. 確保您擁有本指導教學中所使用外掛程式的最新版本；使用 `ibmcloud plugin update --all` 來進行升級。

## 建立服務
{: #setup}

### 決定應用程式的部署位置

1. 確定將部署應用程式及其資源的**位置**、**Cloud Foundry 組織和空間**以及**資源群組**。

### 擷取使用者和應用程式活動 
{: #activity-tracker }

{{site.data.keyword.cloudaccesstrailshort}} 服務將記錄由使用者啟動且變更 {{site.data.keyword.Bluemix_notm}} 中服務狀態的活動。在本指導教學結束時，您可檢閱藉由完成本指導教學的各步驟產生的事件。

1. 存取 {{site.data.keyword.Bluemix_notm}}「型錄」並建立 [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker) 實例。請注意，每個 Cloud Foundry 空間只能有一個 {{site.data.keyword.cloudaccesstrailshort}} 實例。
2. 建立實例後，將其名稱變更為 **secure-file-storage-activity-tracker**。
3. 若要檢視所有 Activity Tracker 事件，請確保您已獲指派許可權：
   1. 移至[身分及存取 > 使用者](https://{DomainName}/iam/#/users)。
   2. 在清單中選取您的使用者名稱。
   3. 在**存取原則**下，如果沒有原則，請在建立 {{site.data.keyword.loganalysisshort_notm}} 服務的位置，為該服務建立具有**檢視者**角色的原則。
   4. 在 **Cloud Foundry 存取權**下，確保您在佈建 {{site.data.keyword.cloudaccesstrailshort}} 的 Cloud Foundry 空間中具有**開發人員**角色。如果沒有，請使用組織的 Cloud Foundry 管理員來指派該角色。

在 [{{site.data.keyword.cloudaccesstrailshort}} 文件](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy)中可找到有關設定許可權的詳細指示。
{: tip}

### 為應用程式建立叢集

{{site.data.keyword.containershort_notm}} 提供環境以在 Kubernetes 叢集裡執行的 Docker 容器，部署高可用性的應用程式。

如果您有現有的**標準**叢集，想要在本指導教學中重複使用該叢集，請跳過本節。
{: tip}

1. 存取[叢集建立頁面](https://{DomainName}/containers-kubernetes/catalog/cluster/create)。
   1. 將**位置**設定為先前步驟中使用的位置。
   2. 將**叢集類型**設定為**標準**。
   3. 將**可用性**設定為**單一區域**。
   4. 選取**主節點區域**。
2. 保留預設 **Kubernetes 版本**和**硬體隔離**。
3. 如果計劃在此叢集上僅部署本指導教學，請將**工作者節點**設定為 **1**。
4. 將**叢集名稱**設定為 **secure-file-storage-cluster**。
5. 按一下**建立叢集**按鈕。

在佈建叢集期間，您將建立本指導教學所需的其他服務。

### 使用您自己的加密金鑰

{{site.data.keyword.keymanagementserviceshort}} 可協助您在 {{site.data.keyword.Bluemix_notm}} 服務之間為應用程式佈建加密金鑰。{{site.data.keyword.keymanagementserviceshort}} 與 {{site.data.keyword.cos_full_notm}} [一起使用可保護靜態資料](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos)。在本節中，您將為儲存空間儲存區建立一個根金鑰。

1. 建立 [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms) 實例。
   * 將「名稱」設定為 **secure-file-storage-kp**。
   * 選取要在其中建立服務實例的資源群組。
2. 在**管理**下，按一下**新增金鑰**按鈕以建立新的根金鑰。此金鑰將用於加密儲存空間儲存區內容。
   * 將「名稱」設定為 **secure-file-storage-root-enckey**。
   * 將「金鑰類型」設定為**根金鑰**。
   * 然後，**產生金鑰**。

自帶金鑰 (BYOK) 可藉由[匯入現有根金鑰](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys)來提供。
{: tip}

### 設定用於使用者檔案的儲存空間

檔案共用應用程式會將檔案儲存到 {{site.data.keyword.cos_short}} 儲存區。檔案與使用者之間的關係作為 meta 資料儲存在 {{site.data.keyword.cloudant_short_notm}} 資料庫中。在本節中，您將建立和配置這些服務。

#### 內容的儲存區

1. 建立 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) 實例。
   * 將**名稱**設定為 **secure-file-storage-cos**。
   * 使用與先前服務相同的**資源群組**。
2. 在**服務認證**下，*新建認證*。
   * 將**名稱**設定為 **secure-file-storage-cos-acckey**。
   * 將**角色**設定為**撰寫者**。
   * 不要指定**服務 ID**。
   * 將**線型配置參數**設定為 **{"HMAC":true}**。產生預先簽署的 URL 時需要此參數。
   * 按一下**新增 **。
   * 按一下**檢視認證**以記下認證。您將在之後的步驟中需要它們。
3. 按一下功能表中的**端點**：將**備援**設定為**地區**，將**位置**設定為目標位置。複製**公用**服務端點。稍後在應用程式配置中將使用此端點。

在建立儲存區之前，將授權 **secure-file-storage-cos** 對儲存在 **secure-file-storage-kp** 中的根金鑰的存取權。

1. 移至 {{site.data.keyword.cloud_notm}} 主控台中的[身分及存取 > 授權](https://{DomainName}/iam/#/authorizations)。
2. 按一下**建立**按鈕。
3. 在**來源服務**功能表中，選取 **Cloud Object Storage**。
4. 在**來源服務實例**功能表中，選取先前建立的 **secure-file-storage-cos** 服務。
5. 在**目標服務**功能表中，選取 **Key Protect**。
6. 在**目標服務實例**功能表中，選取要對其授權的 **secure-file-storage-kp** 服務。
7. 啟用**讀者**角色。
8. 按一下**授權**按鈕。

最後建立儲存區。

1. 存取[資源清單](https://{DomainName}/resources)中的 **secure-file-storage-cos** 服務實例。
2. 按一下**建立儲存區**。
   1. 將**名稱**設定為唯一值，例如 **&lt;your-initials&gt;-secure-file-upload**。
   2. 將**備援**設定為**地區**。
   3. 將**位置**設定為在其中建立 **secure-file-storage-kp** 服務的位置。
   4. 將**儲存空間類別**設定為**標準**。
3. 選取**新增 Key Protect 金鑰**旁邊的勾選框。
   1. 選取 **secure-file-storage-kp** 服務。
   2. 選取 **secure-file-storage-root-enckey** 作為金鑰。
4. 按一下**建立儲存區**。

#### 使用者與其檔案之間的資料庫對映關係

{{site.data.keyword.cloudant_short_notm}} 資料庫將包含從應用程式上傳的所有檔案的 meta 資料。

1. 建立 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 實例。
   * 將**名稱**設定為 **secure-file-storage-cloudant**。
   * 設定位置。
   * 使用與先前服務相同的**資源群組**。
   * 將**可用鑑別方法**設定為**僅使用 IAM**。
2. 在**服務認證**下，*新建認證*。
   * 將**名稱**設定為 **secure-file-storage-cloudant-acckey**。
   * 將**角色**設定為**管理員**。
   * 保留*選用* 欄位的預設值。
   * **新增**。
3. 按一下**檢視認證**以記下認證。您將在之後的步驟中需要它們。
4. 在**管理**下，啟動 Cloudant 儀表板。
5. 按一下**建立資料庫**以建立名為 **secure-file-storage-metadata** 的資料庫。

### 鑑別使用者

使用 {{site.data.keyword.appid_short}}，可以保護資源並向應用程式新增鑑別。{{site.data.keyword.appid_short}} 與 {{site.data.keyword.containershort_notm}} [整合](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth)，以對存取叢集中所部署應用程式的使用者進行鑑別。

1. 建立 [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) 實例。
   * 將**服務名稱**設定為 **secure-file-storage-appid**。
   * 使用先前服務所用的相同**位置**和**資源群組**。
2. 在**鑑別設定**標籤的**身分提供者/管理**下，新增指向將用於應用程式的網域的 **Web 重新導向 URL**。例如，如果叢集 Ingress 子網域為 `<cluster-name>.us-south.containers.appdomain.cloud`，則重新導向 URL 將為 `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback`。{{site.data.keyword.appid_short}} 需要 Web 重新導向 URL 為 **https**。您可以在叢集儀表板中或使用 `ibmcloud ks cluster-get <cluster-name>` 來檢視 Ingress 子網域。

您應該在 {{site.data.keyword.appid_short}} 儀表板中自訂使用的身分提供者以及登入和使用者管理體驗。為求簡化，本指導教學使用的是預設值。對於正式作業環境，請考慮使用多因子鑑別 (MFA) 和進階密碼規則。
{: tip}

## 部署應用程式

現在所有服務都已配置。在本節中，您要將指導教學應用程式部署到叢集。

### 取得程式碼

1. 取得應用程式的程式碼：
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. 移至 **secure-file-storage** 目錄：
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### 建置 Docker 映像檔

1. 在 {{site.data.keyword.registryshort_notm}} 中[建置 Docker 映像檔](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating)。
   - 使用 `ibmcloud cr info` 來尋找登錄端點，例如 **us**.icr.io 或 **uk**.icr.io。
   - 建立名稱空間以儲存容器映像檔。
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - 使用 **secure-file-storage** 作為映像檔名稱。

   ```sh
   ibmcloud cr build -t <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### 填寫認證和配置設定

1. 將 `credentials.template.env` 複製到 `credentials.env`：
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. 編輯 `credentials.env`，並使用以下值填寫空白：
   * {{site.data.keyword.cos_short}} 服務地區端點、儲存區名稱、為 **secure-file-storage-cos** 建立的認證
   * 以及 **secure-file-storage-cloudant** 的認證。
3. 將 `secure-file-storage.template.yaml` 複製到 `secure-file-storage.yaml`：
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. 編輯 `secure-file-storage.yaml`，並將位置保留元（`$IMAGE_PULL_SECRET`、`$REGISTRY_URL`、`$REGISTRY_NAMESPACE`、`$IMAGE_NAME`、`$TARGET_NAMESPACE`、`$INGRESS_SUBDOMAIN` 和 `$INGRESS_SECRET`）取代為正確的值。例如，假設應用程式部署到 _default_ Kubernetes 名稱空間：

|變數|值|說明|
| -------- | ----- | ----------- |
| `$IMAGE_PULL_SECRET` |保留在 .yaml 中已設為註解的行。|用於存取登錄的密碼。|
| `$REGISTRY_URL` | *us.icr.io* |在上節中，在其中建置映像檔的登錄。|
| `$REGISTRY_NAMESPACE` | *secure-file-storage-namespace* |在上節中，在其中建置映像檔的登錄名稱空間。|
| `$IMAGE_NAME` | *secure-file-storage* |Docker 映像檔的名稱。|
| `$TARGET_NAMESPACE` | *default* |將在其中推送應用程式的 Kubernetes 名稱空間。|
| `$INGRESS_SUBDOMAIN` | *secure-file-stora-123456.us-south.containers.appdomain.cloud* |從叢集概觀頁面或使用 `ibmcloud ks cluster-get secure-file-storage-cluster` 擷取。|
| `$INGRESS_SECRET` | *secure-file-stora-123456* |從叢集概觀頁面或使用 `ibmcloud ks cluster-get secure-file-storage-cluster` 擷取。|

僅當要使用非預設 Kubernetes 名稱空間時，才需要 `$IMAGE_PULL_SECRET`。這需要額外的 Kubernetes 配置（例如，[在新名稱空間中建立 Docker 登錄密碼](https://{DomainName}/docs/containers?topic=containers-images#other)）。
{: tip}

### 部署到叢集

1. 擷取叢集配置，並設定 KUBECONFIG 環境變數。
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. 建立應用程式用於取得服務認證的密碼：
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. 將 {{site.data.keyword.appid_short_notm}} 服務實例連結到叢集。
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
   如果您有多個同名服務，則指令會失敗。您應該傳遞服務 GUID，而不是服務名稱。若要尋找服務的 GUID，請使用 `ibmcloud resource service-instance secure-file-storage-appid`。
   {: tip}
4. 部署應用程式。
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## 測試應用程式

可以在 `https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/` 上存取應用程式。

1. 移至應用程式的首頁。這會將您重新導向到 {{site.data.keyword.appid_short_notm}} 預設登入頁面。
2. 使用有效電子郵件位址註冊新帳戶。
3. 等待信箱收到用於驗證帳戶的電子郵件。
4. 登入。
5. 選擇要上傳的檔案。按一下**上傳**。
6. 對檔案使用**共用**動作，以產生預先簽署的 URL，此 URL 可以與要存取該檔案的其他人共用。鏈結設定為在 5 分鐘後到期。

已鑑別使用者有自己的空間可儲存檔案。雖然他們無法查看彼此的檔案，但可以產生預先簽署的 URL 來授權對特定檔案的暫時存取權。

可以在[原始碼儲存庫](https://github.com/IBM-Cloud/secure-file-storage)中找到有關應用程式的更多詳細資料。

## 檢閱安全事件
既然已順利部署應用程式及其服務，現在可以檢閱該處理程序產生的安全事件。所有事件都在 {{site.data.keyword.cloudaccesstrailshort}} 實例中集中提供，可以透過[圖形使用者介面 (Kibana)、CLI 或 API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status) 進行存取。

1. 在 [{{site.data.keyword.Bluemix_notm}} 資源清單](https://{DomainName}/resources)中，找到 {{site.data.keyword.cloudaccesstrailshort}} 實例 **secure-file-storage-activity-tracker**，並開啟其儀表板。
2. 依預設，**管理**標籤會顯示**空間日誌**。藉由按一下**檢視日誌**旁邊的選取器，可切換到**帳戶日誌**。其中應該會顯示數個事件。
3. 按一下**在 Kibana 中檢視**以開啟完整事件檢視器。
4. 藉由按一下**探索**，檢閱事件詳細資料。
5. 在**可用欄位**中，新增 **action_str** 和 **initiator.name_str**。
6. 藉由按一下三角形圖示，然後選擇表格或 JSON 格式，展開感興趣的項目。

可以變更自動重新整理的設定和顯示的時間範圍，而變更資料分析方式以及所分析的資料內容。
{: tip}

## 選用：使用自訂網域並加密網路資料流量
依預設，可以在 `containers.appdomain.cloud` 子網域的通用主機名稱上存取應用程式。但是，也可以將自訂網域用於部署的應用程式。若要持續支援 **https**，需要提供對加密網路資料流量的存取權（所需主機名稱的憑證或萬用字元憑證）。在下節中，您要將現有憑證上傳到 {{site.data.keyword.cloudcerts_short}}，然後將其部署到叢集。此外，還將更新應用程式配置以使用自訂網域。

在 [{{site.data.keyword.cloud}} 部落格](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)中說明如何從 [Let's Encrypt](https://letsencrypt.org/) 取得憑證的範例。
{: tip}

1. 建立 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager) 的實例
   * 將「名稱」設定為 **secure-file-storage-certmgr**。
   * 使用其他服務所用的相同**位置**和**資源群組**。
2. 按一下**匯入憑證**以匯入您的憑證。
   * 將「名稱」設定為 **SecFileStorage**，將「說明」設定為 **e2e 安全指導教學的憑證**。
   * 使用**瀏覽**按鈕上傳該憑證檔。
   * 按一下**匯入**以完成匯入處理程序。
3. 找到已匯入憑證的項目，並將其展開。
   * 驗證憑證項目，例如驗證網域名稱是否與您的自訂網域相符。如果上傳的是萬用字元憑證，則網域名稱中會包含星號。
   * 按一下憑證 **crn** 旁邊的**複製**符號。
4. 切換到指令行，以將憑證資訊作為密碼部署到叢集。在上一步中複製 crn 中的內容後，執行下列指令。
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <the copied crn from previous step>
   ```
   {: codeblock}
   藉由執行下列指令，驗證叢集是否知道有關該憑證的資訊。
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. 編輯 `secure-file-storage.yaml` 檔案。
   * 找到與 **Ingress** 相關的小節。
   * 解除註解並編輯涵蓋自訂網域的行，然後填寫您的網域名稱和主機名稱。
   自訂網域的 CNAME 項目需要指向叢集。如需詳細資料，請查看本文件中的[關於對映自訂網域的手冊](https://{DomainName}/docs/containers?topic=containers-ingress#private_3)。
   {: tip}
6. 將配置變更套用於已部署的配置：
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. 切換回瀏覽器。在 [{{site.data.keyword.Bluemix_notm}} 資源清單](https://{DomainName}/resources)中，找到先前建立並配置的 {{site.data.keyword.appid_short}} 服務，然後啟動其管理儀表板。
   * 移至**身分提供者**下的**管理**，然後移至**設定**。
   * 在**新增 Web 重新導向 URL** 表單中，將 `https://secure-file-storage.<your custom domain>/appid_callback` 新增為另一個 URL。
8. 現在，一切應該都已就緒。請藉由在配置的自訂網域 `https://secure-file-storage.<your custom domain>` 上存取應用程式來測試應用程式。

## 擴展指導教學

安全永無止境。請嘗試以下建議來增強應用程式的安全。

* 使用 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 執行靜態和動態碼掃描
* 藉由將原則和規則與 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 搭配使用，確保僅發佈高品質程式碼

## 移除資源
{:removeresources}

若要移除資源，請刪除部署的容器，然後刪除佈建的服務。

1. 刪除部署的容器：
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. 刪除用於部署的密碼：
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. 從容器登錄中移除 Docker 映像檔：
   ```sh
   ibmcloud cr image-rm <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. 在 [{{site.data.keyword.Bluemix_notm}} 資源清單](https://{DomainName}/resources)中，找到為此指導教學建立的資源。使用搜尋方框和 **secure-file-storage** 作為模式。藉由按一下每個服務旁邊的快速功能表，並選擇**刪除服務**來刪除每個服務。請注意，{{site.data.keyword.keymanagementserviceshort}} 服務只有在刪除金鑰後才能移除。按一下該服務實例以前往相關儀表板，以及刪除金鑰。

如果您與其他使用者共用帳戶，請務必僅刪除您自己的資源。
{: tip}

## 相關內容
{:related}

* [{{site.data.keyword.security-advisor_short}} 文件](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [Security to safeguard and monitor your cloud apps](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [{{site.data.keyword.Bluemix_notm}} 平台安全](https://{DomainName}/docs/overview?topic=overview-security#security)
* [Security in the IBM Cloud](https://www.ibm.com/cloud/security)
* [指導教學：組織使用者、團隊和應用程式的最佳作法](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [Secure Apps on IBM Cloud with Wildcard Certificates](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
