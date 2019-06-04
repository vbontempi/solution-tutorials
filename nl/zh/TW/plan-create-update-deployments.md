---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# 規劃、建立和更新部署環境
{: #plan-create-update-deployments}

建置解決方案時，多個部署環境十分常見。這些環境反映了從開發到正式作業的專案生命週期。本指導教學將介紹 {{site.data.keyword.Bluemix_notm}} CLI 和 [Terraform](https://www.terraform.io/) 等工具，以自動建立和維護這些部署環境。
{:shortdesc}

開發人員很討厭重複撰寫相同的東西。[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 原則就是其中一個例子。同樣地，開發人員也不喜歡必須在使用者介面中大量點擊才能設定環境。因此，系統管理者和開發人員長期以來一直是使用 Shell Script 將重複性、易出錯且無趣的作業自動化。

基礎架構即服務 (IaaS)、平台即服務 (PaaS)、容器即服務 (CaaS) 和函數即服務 (FaaS) 為開發人員提供了高層次的抽象概念，以便能更輕鬆地獲得各種資源，如 Bare Metal Server 伺服器、受管理資料庫、虛擬機器、Kubernetes 叢集等。但是，一旦佈建了這些資源後，就需要將其連接在一起，以便能配置使用者存取權及隨時間變化更新配置等等。在現今的環境中，能夠自動化所有這些步驟並在不同環境下重複執行安裝和配置是非常必要的。

在一個專案中有多個環境十分常見，這些環境可支援開發週期的不同階段，各個環境之間在容量、網路連線功能、認證、日誌詳細程度等方面略有差異。在[其他指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)中，我們已經介紹過組織使用者、團隊和應用程式的最佳作法以及一個範例情境。該範例情境考慮了三個環境：*開發*、*測試*和*正式作業*。如何自動建立這些環境？可以使用哪些工具？

## 目標
{: #objectives}

* 定義要部署的一組環境
* 使用 {{site.data.keyword.Bluemix_notm}} CLI 和 [Terraform](https://www.terraform.io/) 撰寫 Script 以自動部署這些環境
* 在帳戶中部署這些環境

## 使用的服務
{: #services}

本指導教學使用下列產品：
* [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Identity and Access Management](https://{DomainName}/iam/#/users)
* [{{site.data.keyword.Bluemix_notm}} 指令行介面 - `ibmcloud` CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. 建立一組 Terraform 檔案，用於說明目標基礎架構即程式碼。
1. 操作員套用 `terraform apply` 來佈建各個環境。
1. 撰寫 Shell Script 以完成環境配置。
1. 操作員針對環境執行 Script。
1. 環境已完全配置，可供使用。

## 可用工具概觀
{: #tools}

第一個與 {{site.data.keyword.Bluemix_notm}} 互動並建立可重複部署的工具是 [{{site.data.keyword.Bluemix_notm}} 指令行介面 - `ibmcloud` CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)。藉由 `ibmcloud` 及其外掛程式，可以自動建立和配置雲端資源。透過指令行，則可以佈建 {{site.data.keyword.virtualmachinesshort}}、Kubernetes 叢集、{{site.data.keyword.openwhisk_short}}、Cloud Foundry 應用程式和服務。

[本指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)中介紹的另一個工具是 HashiCorp 的產品 [Terraform](https://www.terraform.io/)。引用 HashiCorp 所述，*透過 Terraform，能夠以可預測的方式安全地建立、變更和改進基礎架構。Terraform 是一個開放程式碼工具，用於將 API 編碼為宣告的配置檔，這些檔案可以在團隊成員之間共用，視為程式碼，進行編輯、檢閱和版本化。*這屬於基礎架構即程式碼。您寫下所預期的基礎架構，然後 Terraform 將根據需要建立、更新和移除雲端資源。

為了支援多雲端方法，Terraform 會與提供者合作。提供者負責瞭解 API 互動及公開資源。{{site.data.keyword.Bluemix_notm}} 具有[自己的 Terraform 提供者](https://github.com/IBM-Cloud/terraform-provider-ibm)，可讓 {{site.data.keyword.Bluemix_notm}} 使用者使用 Terraform 來管理資源。雖然 Terraform 被分類為基礎架構即程式碼，但適用範圍並不局限於基礎架構即服務資源。{{site.data.keyword.Bluemix_notm}} Provider for Terraform 支援 IaaS（Bare Metal Server、虛擬機器、網路服務等）、CaaS（{{site.data.keyword.containershort_notm}} 和 Kubernetes 叢集）、PaaS（Cloud Foundry 和服務）和 FaaS ({{site.data.keyword.openwhisk_short}}) 資源。

## 撰寫自動部署 Script
{: #scripts}

開始說明基礎架構即程式碼時，請務必將建立的檔案視為正規程式碼，而將其儲存在來源控制管理系統中。這樣在一段時間後就能呈現出各種優點，例如先利用來源控制檢閱工作流程驗證變更之後再套用變更，以及新增持續整合管道以自動部署基礎架構變更。

[此 Git 儲存庫](https://github.com/IBM-Cloud/multiple-environments-as-code)具有設定先前定義之環境所需的所有配置檔。您可以複製此儲存庫，以按照詳細描述檔案內容的以下各節進行操作。

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

儲存庫的結構如下所示：

|目錄|說明|
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) |Terraform 檔案的起始目錄|
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) |用於佈建三種環境的通用資源的 Terraform 檔案|
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) |給定環境特定的 Terraform 檔案|
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) |用於配置使用者原則的 Terraform 檔案|

### 透過 Terraform 處理棘手任務

*開發*、*測試* 和*正式作業* 環境幾乎看起來都一樣。

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="顯示一個部署環境的圖" />
</p>

各個環境共用一個共同的組織和環境特定的資源。環境在配置的容量和存取權方面有所不同。這些特點反映在 Terraform 檔案中，***global*** 配置用於佈建 Cloud Foundry 組織，***per-environment*** 配置使用 Terraform 工作空間來佈建環境特定的資源：

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### 廣域配置

所有環境都共用一個共同的 Cloud Foundry 組織，每個環境都有其自己的空間。

在 [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) 目錄下，可以找到 Terraform Script 來佈建此組織。[main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf) 套件含組織的定義：

   ```sh
   # create a new organization for the project
   resource "ibm_org" "organization" {
     name             = "${var.org_name}"
     managers         = "${var.org_managers}"
     users            = "${var.org_users}"
     auditors         = "${var.org_auditors}"
     billing_managers = "${var.org_billing_managers}"
   }
   ```

在此資源中，所有內容均透過變數進行配置。在以下各區段中，您將瞭解如何設定這些變數。

若要完全部署環境，您將混合使用 Terraform 和 {{site.data.keyword.Bluemix_notm}} CLI。使用 CLI 撰寫的 Shell Script 可能需要依名稱或 ID 來參照此組織或帳戶。*global* 目錄還包含 [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf)，這可用來產生一個檔案，其中包含此資訊作為鍵/值，適合在編寫 Script 時重複使用：

   ```sh
   # generate a property file suitable for shell scripts with useful variables relating to the environment
   resource "local_file" "output" {
     content = <<EOF
ACCOUNT_GUID=${data.ibm_account.account.id}
ORG_GUID=${ibm_org.organization.id}
ORG_NAME=${var.org_name}
EOF

     filename = "../outputs/global.env"
   }
   ```

### 不同的環境

使用 Terraform 來管理多個環境有多種不同的方法。可以在個別的目錄下複製 Terraform 檔案，每個環境一個目錄。透過 [Terraform 模組](https://www.terraform.io/docs/modules/index.html)，可以將共用配置視為一個群組，並在不同環境中重複使用模組，以便減少程式碼重複。個別的目錄意味著可以逐漸改進*開發* 環境來測試變更，然後將變更傳播到其他環境。在這種情況下，Terraform *模組* 也常常位於其自己的原始碼儲存庫中，以便可以在環境檔案中參照模組的特定版本。

考慮到各個環境相當簡單且類似，您將使用另一個名為[工作空間](https://www.terraform.io/docs/state/workspaces.html#best-practices)的 Terraform 概念。透過工作空間，可以在不同環境中使用相同的 Terraform 檔案 (.tf)。在範例中，*development*、*testing* 和 *production* 是工作空間。它們將使用相同的 Terraform 定義，但具有不同的配置變數（不同的名稱和不同的容量）。

每個環境需要：
* 一個專用 Cloud Foundry 空間
* 一個專用資源群組
* 一個 Kubernetes 叢集
* 資料庫
* 檔案儲存空間

Cloud Foundry 空間已鏈結到上一步中建立的組織。環境 Terraform 檔案需要參照此組織。這正是 [Terraform 遠端狀態](https://www.terraform.io/docs/state/remote.html)的功用。它容許以唯讀模式參照現有 Terraform 狀態。這是一個非常有用的建構設計，用來將 Terraform 配置分割成較小的部分，而能將各個部分的責任分配給不同的團隊。[backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf) 套件含 *global* 遠端狀態的定義，用於尋找先前建立的組織：

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

可以參照組織後，就可以直接在此組織中建立空間。[main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf) 套件含針對環境的資源定義。

   ```sh
   # a Cloud Foundry space per environment
   resource "ibm_space" "space" {
     name       = "${var.environment_name}"
     org        = "${data.terraform_remote_state.global.org_name}"
     managers   = "${var.space_managers}"
     auditors   = "${var.space_auditors}"
     developers = "${var.space_developers}"
   }
   ```

請注意 *global* 遠端狀態中是如何參照組織名稱的。其他內容透過配置變數取得。

接下來是資源群組。

   ```sh
   # a resource group
   resource "ibm_resource_group" "group" {
    name     = "${var.environment_name}"
    quota_id = "${data.ibm_resource_quota.quota.id}"
}

   data "ibm_resource_quota" "quota" {
	name = "${var.resource_quota}"
}
   ```

Kubernetes 叢集在此資源群組中建立。{{site.data.keyword.Bluemix_notm}} 提供者具有 Terraform 資源來代表叢集：

   ```sh
  # a cluster
  resource "ibm_container_cluster" "cluster" {
  	name              = "${var.environment_name}-cluster"
  	datacenter        = "${var.cluster_datacenter}"
  	org_guid          = "${data.terraform_remote_state.global.org_guid}"
  	space_guid        = "${ibm_space.space.id}"
  	account_guid      = "${data.terraform_remote_state.global.account_guid}"
  	hardware          = "${var.cluster_hardware}"
  	machine_type      = "${var.cluster_machine_type}"
  	public_vlan_id    = "${var.cluster_public_vlan_id}"
  	private_vlan_id   = "${var.cluster_private_vlan_id}"
  	resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool" "cluster_workerpool" {
  worker_pool_name  = "${var.environment_name}-pool"
  machine_type      = "${var.cluster_machine_type}"
  cluster           = "${ibm_container_cluster.cluster.id}"
  size_per_zone     = "${var.worker_num}"
  hardware          = "${var.cluster_hardware}"
  resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool_zone_attachment" "cluster_zone" {
  cluster           = "${ibm_container_cluster.cluster.id}"
  worker_pool       =  "${element(split("/",ibm_container_worker_pool.cluster_workerpool.id),1)}"
  zone              = "${var.cluster_datacenter}"
  public_vlan_id    = "${var.cluster_public_vlan_id}"
  private_vlan_id   = "${var.cluster_private_vlan_id}"
  resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

同樣，大多數內容將透過配置變數進行起始設定。可以調整資料中心、工作者數和工作者類型。

{{site.data.keyword.cos_full_notm}} 和 {{site.data.keyword.cloudant_short_notm}} 等啟用 IAM 的服務也會建立為群組內的資源：

   ```sh
# a database
resource "ibm_resource_instance" "database" {
    name              = "database"
    service           = "cloudantnosqldb"
    plan              = "${var.cloudantnosqldb_plan}"
    location          = "${var.cloudantnosqldb_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
# a cloud object storage
resource "ibm_resource_instance" "objectstorage" {
    name              = "objectstorage"
    service           = "cloud-object-storage"
    plan              = "${var.cloudobjectstorage_plan}"
    location          = "${var.cloudobjectstorage_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

可以新增 Kubernetes 連結（密碼），以從應用程式中擷取服務認證：

   ```sh
   # bind the cloudant service to the cluster
   resource "ibm_container_bind_service" "bind_database" {
      cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	  service_instance_name       = "${ibm_resource_instance.database.name}"
      namespace_id                = "default"
      account_guid                = "${data.terraform_remote_state.global.account_guid}"
      org_guid                    = "${data.terraform_remote_state.global.org_guid}"
      space_guid                  = "${ibm_space.space.id}"
      resource_group_id           = "${ibm_resource_group.group.id}"
}

   # bind the cloud object storage service to the cluster
   resource "ibm_container_bind_service" "bind_objectstorage" {
  	cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	space_guid                  = "${ibm_space.space.id}"
  	service_instance_id         = "${ibm_resource_instance.objectstorage.name}"
  	namespace_id                = "default"
  	account_guid                = "${data.terraform_remote_state.global.account_guid}"
  	org_guid                    = "${data.terraform_remote_state.global.org_guid}"
  	space_guid                  = "${ibm_space.space.id}"
  	resource_group_id           = "${ibm_resource_group.group.id}"
}
   ```

## 在帳戶中部署此環境

### 安裝 {{site.data.keyword.Bluemix_notm}} CLI

1. 按照[這些指示](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)來安裝 CLI。
1. 透過執行下列指令驗證安裝：
   ```sh
   ibmcloud
   ```
   {: codeblock}

### 安裝 Terraform 和 {{site.data.keyword.Bluemix_notm}} Provider for Terraform

1. [下載並安裝適用於您系統的 Terraform](https://www.terraform.io/intro/getting-started/install.html)。
1. [下載適用於 {{site.data.keyword.Bluemix_notm}} 提供者的 Terraform 二進位檔](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)。
   若要使用 {{site.data.keyword.Bluemix_notm}} 提供者設定 Terraform，請參閱此[鏈結](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup)
   {:tip}
1. 在指向 Terraform 二進位檔的起始目錄中建立 `.terraformrc` 檔案。在下列範例中，`/opt/provider/terraform-provider-ibm` 是目錄的路徑。
   ```sh
    # ~/.terraformrc
    providers {
        ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### 取得程式碼

如果尚未複製指導教學儲存庫，請進行複製：

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### 設定平台 API 金鑰

1. 取得[平台 API 金鑰](https://{DomainName}/iam/#/apikeys)並儲存此 API 金鑰以供未來參考（如果尚無此金鑰）。

   > 如果在後續步驟中計劃建立新的 Cloud Foundry 組織來管理部署環境，請確保您是帳戶的擁有者。
1. 透過執行以下指令，將 [terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl) 複製到 *terraform/credentials.tfvars*：
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. 編輯 `terraform/credentials.tfvars`，並將 `ibmcloud_api_key` 的值設定為取得的平台 API 金鑰。

### 建立或重複使用 Cloud Foundry 組織

可以選擇建立新的組織或重複使用（匯入）現有組織。若要建立三個部署環境的母組織，**您需要是帳戶擁有者**。

#### 建立新的組織

1. 切換至 `terraform/global` 目錄
1. 將 [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) 複製到 `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. 編輯 `global.tfvars`
   1. 將 **org_name** 設定為要建立的組織名稱
   1. 將 **org_managers** 設定為要被授與組織中*管理員* 角色的使用者 ID 的清單 - 建立組織的使用者會自動成為管理員，因此不應新增到此清單中
   1. 將 **org_users** 設定為要邀請加入組織的所有使用者的清單 - 需要在此處新增要在後續步驟中配置其存取權的使用者

   ```sh
   org_name = "a-new-organization"
   org_managers = [ "user1@domain.com", "another-user@anotherdomain.com" ]
   org_users = [ "user1@domain.com", "another-user@anotherdomain.com", "more-user@domain.com" ]
   ```
   {: codeblock}
1. 從 `terraform/global` 資料夾起始設定 Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. 查看 Terraform 方案
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. 套用變更
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Terraform 完成之後，它會建立：
* 新的 Cloud Foundry 組織
* 移出程序中 `outputs` 目錄下的一個 `global.env` 檔案。這個檔案具有您可以在其他 Script 參考的環境變數
* `terraform.tfstate` 檔案

> 對於 Terraform 狀態，本指導教學使用的是 `local` 後端提供者。這在探索 Terraform 或單獨在專案上工作時很方便，但以團隊形式或在更大型基礎架構上工作時，Terraform 還支援將狀態儲存到遠端位置。由於 Terraform 狀態對於 Terraform 作業至關重要，因此建議使用遠端、高可用性、具復原力的儲存空間來儲存 Terraform 狀態。如需可用選項的清單，請參閱 [Terraform 後端類型](https://www.terraform.io/docs/backends/types/index.html)。有些後端甚至支援對 Terraform 狀態進行版本化和鎖定。

#### 重複使用所管理的組織

如果您不是帳戶擁有者，但負責管理帳戶中的組織，您也可以將現有組織匯入到 Terraform 中。

1. 擷取組織 GUID
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. 切換至 `terraform/global` 目錄
1. 將 [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) 複製到 `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. 起始設定 Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. 起始設定 Terraform 後，將組織匯入到 Terraform 狀態中
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. 調整 `global.tfvars` 以符合現有組織名稱和結構
1. 套用變更
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### 建立 per-environment 空間、叢集和服務

此區段主要說明 `development` 環境。對於其他環境，步驟是相同的，只是為變數選取的值有所不同。

1. 切換到移出程序的 `terraform/per-environment` 資料夾
1. 複製範本 `tfvars` 檔。每個環境都有一個 tfvars 文件：
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. 編輯 `development.tfvars`
   1. 將 **environment_name** 設定為要建立的 Cloud Foundry 空間的名稱
   1. 將 **space_developers** 設定為此空間的開發人員清單。**確保將您的姓名新增到此清單中，以便 Terraform 可以代表您佈建服務。**
   1. 將 **cluster_datacenter** 設定為要在其中建立叢集的位置。使用下列指令尋找可用的位置：
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. 為叢集設定專用 (**cluster_private_vlan_id**) 和公用 (**cluster_public_vlan_id**) VLAN。使用下列指令尋找位置可用的 VLAN：
      ```sh
      ibmcloud cs vlans <location>
      ```
      {: codeblock}
   1. 設定 **cluster_machine_type**。使用下列指令尋找位置的可用機型和性質：
      ```sh
      ibmcloud cs machine-types <location>
      ```
      {: codeblock}

   1. 設定 **resource_quota**。使用下列指令尋找可用的資源配額定義：
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. 起始設定 Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. 為 *development* 環境建立新的 Terraform 工作區
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   稍後可切換使用各個環境
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. 查看 Terraform 方案
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   它應該報告：
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. 套用變更
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Terraform 完成之後，它會建立：
* 資源群組
* Cloud Foundry 空間
* 一個 Kubernetes 叢集，其連接一個工作者儲存區和一個區域
* 資料庫
* 包含資料庫認證的一個 Kubernetes 密碼
* 一個儲存空間
* 包含儲存空間認證的一個 Kubernetes 密碼
* 一個記載 (LogDNA) 實例
* 一個監視 (Sysdig) 實例
* 移出程序中 `outputs` 目錄下的一個 `development.env` 檔案。這個檔案具有您可以在其他 Script 參考的環境變數
* `terraform.tfstate.d/development` 下環境特定的 `terraform.tfstate`。

您可以針對 `testing` 及 `production` 重複這些步驟。

### 指派使用者原則

在先前的步驟中，Cloud Foundry 組織和空間中的角色可以使用 Terraform 提供者進行配置。對於其他資源（如 Kubernetes 叢集）上的使用者原則，將使用複製的儲存庫中的 [roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) 資料夾。

對於[本指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)中定義的*開發* 環境，要定義的原則如下：

|           | IAM 存取原則|
| --------- | ----------- |
|開發人員| <ul><li>資源群組：*檢視者*</li><li>資源群組中的平台存取角色：*檢視者*</li><li>記載及監視服務角色：*撰寫者*</li></ul> |
|測試人員| <ul><li>不需要配置。測試人員會存取已部署的應用程式，而非開發環境</li></ul> |
|操作員 | <ul><li>資源群組：*檢視者*</li><li>資源群組中的平台存取角色：*操作員*、*檢視者*</li><li>記載及監視服務角色：*撰寫者*</li></ul> |
|管線功能使用者| <ul><li>資源群組：*檢視者*</li><li>資源群組中的平台存取角色：*編輯者*、*檢視者*</li></ul> |

考慮到一個團隊可能有多個開發人員和測試者，因此可以利用[存取群組概念](https://{DomainName}/docs/iam?topic=iam-groups#groups)來簡化使用者原則的配置。存取群組可以由帳戶擁有者建立，這樣就可以使用單一原則將相同的存取權指派給群組內的所有實體。

對於*開發* 環境中的*開發人員* 角色，這將轉換為：

   ```sh
resource "ibm_iam_access_group" "developer_role" {
  name        = "${var.access_group_name_developer_role}"
  description = "${var.access_group_description}"
}
resource "ibm_iam_access_group_policy" "resourcepolicy_developer" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Viewer"]

  resources = [{
    resource_type = "resource-group"
    resource      = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_platform_accesspolicy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles        = ["Viewer"]

  resources = [{
    resource_group_id = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_logging_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "logdna"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.logdna_instance_id}"
  }]
}
resource "ibm_iam_access_group_policy" "developer_monitoring_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "sysdig-monitor"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.sysdig_instance_id}"
  }]
}

   ```

移出程序中的 [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) 檔案包含針對已定義*開發人員*、*操作員*、*測試者* 和*功能使用者* 角色的這些資源的範例。若若要針對*開發* 環境中具有*開發人員、操作員、測試者和功能使用者* 角色的使用者，設定上一區段中定義的原則，請執行下列動作：

1. 切換到 `terraform/roles/development` 目錄
2. 複製範本 `tfvars` 檔。每個環境都有一個 tfvars 文件（可以在 `roles` 目錄中 `production` 和 `testing` 範本的各自資料夾下找到這兩個範本）

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. 編輯 `development.tfvars`

   - 將 **iam_access_members_developers** 設定為要被授與存取權的開發人員的清單。
   - 將 **iam_access_members_operators** 設定為操作員的清單，依此類推。
4. 起始設定 Terraform
   ```sh
   terraform init
   ```
   {: codeblock}

5. 查看 Terraform 方案
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   它應該報告：
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. 套用變更
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
您可以針對 `testing` 及 `production` 重複這些步驟。

## 移除資源

1. 導覽至 `roles` 下的 `development` 資料夾
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. 毀損存取群組和存取原則
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. 啟動 `development` 工作區
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. 毀損資源群組、空間、服務和叢集
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. 對 `testing` 和 `production` 工作空間重複這些步驟。
1. 如果已建立了組織，請將其毀損
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## 相關內容

* [Terraform 指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Terraform 提供者](https://ibm-cloud.github.io/tf-ibm-docs/)
* [Examples using {{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
