---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-23"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 使用 Terraform 部署 LAMP 堆疊
{: #infrastructure-as-code-terraform}

[Terraform](https://www.terraform.io/) 讓您能以安全且可預測的方式建立、變更和改進基礎架構。Terraform 是一個開放程式碼工具，用於將 API 編碼為宣告的配置檔，這些檔案可以在團隊成員之間共用，視為程式碼，進行編輯、檢閱和版本化。

在本指導教學中，您將使用範例配置來佈建 **L**inux 虛擬伺服器（使用 **A**pache Web 伺服器、**M**ySQL 和 **P**HP 伺服器），這稱為 **LAMP** 堆疊。然後，將更新配置以新增 {{site.data.keyword.cos_full_notm}} 服務並調整資源以調整環境（記憶體、CPU 和磁碟大小）。最後將刪除配置所建立的所有資源。

## 目標
{: #objectives}

* 配置 Terraform 和 {{site.data.keyword.Bluemix_notm}} Provider for Terraform。
* 使用 Terraform 建立、更新、調整並最終破壞 LAMP 堆疊配置。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

<p style="text-align: center;">

  ![架構圖](images/solution10/architecture-2.png)
</p>

1. 建立一組 Terraform 檔案來說明 LAMP 堆疊配置。
1. 從配置目錄中呼叫 `terraform`。
1. `terraform` 呼叫 {{site.data.keyword.cloud_notm}} API 來佈建資源。

## 開始之前
{: #prereqs}

請與您的基礎架構主要使用者聯絡，以取得下列許可權：
- 網路（用於新增**公用和專用網路上行鏈路**）
- API 金鑰

## 必要條件

{: #prereq}

藉由[安裝程式](https://www.terraform.io/intro/getting-started/install.html)來安裝 **Terraform**，或者在 MacOS 上使用 [Homebrew](https://brew.sh/) 執行指令 `brew install terraform` 來安裝 Terraform。

在 **Windows** 上，執行以下步驟完成 Terraform 設定：
   1. 將下載的 zip 中的檔案複製到 `C:\terraform`（建立 `terraform` 資料夾）。
   2. 以管理者身分開啟命令提示字元，然後將 PATH 設定為使用 Terraform 二進位檔。

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## 配置 {{site.data.keyword.Bluemix_notm}} Provider for Terraform
{: #setup}

為了支援多雲端方法，Terraform 會與提供者合作。提供者負責瞭解 API 互動及公開資源。在本節中，您將配置 CLI 以指定 {{site.data.keyword.Bluemix_notm}} 提供者的位置。

1. 藉由在終端機或命令提示字元視窗中執行 `terraform`，檢查 Terraform 安裝。您應該會看到`共用指令`的清單。

  ```
  terraform
  ```

2. 下載系統的適當 [{{site.data.keyword.Bluemix_notm}} Provider](https://github.com/IBM-Cloud/terraform-provider-ibm/releases) 外掛程式，並解壓縮保存檔。您應該會看到 `terraform-provider-ibm_VERSION` 二進位外掛程式檔案。

3. 對於非 Windows 系統，請在使用者的起始目錄中建立 `.terraform.d/plugins` 目錄，並將二進位檔放入其中。使用下列指令可取得參考資訊。

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    在 **Windows** 上，該檔案需要放入使用者的 "Application Data" 目錄下的 `terraform.d/plugins` 中。

  - 在命令提示字元處，執行以下指令來建立[提供者配置](https://www.terraform.io/docs/configuration/providers.html)
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - 啟動 **Windows Powershell**（開始 + R > Powershell），然後執行以下指令來建立 `terraform.rc` 檔案
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   在第一個提示字元處，輸入以下內容
   ```
    # ~/.terraformrc
    providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        PATH_TO_YOUR_APPDATA_PLUGINS 應該是包含正斜線 (/) 的絕對路徑。例如，`C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe`
        {: tip}

  - 按一下 Enter 鍵以結束提示字元。

## 準備 Terraform 配置

{: #terraformconfig}

在本節中，您將使用 {{site.data.keyword.Bluemix_notm}} 提供的範例 Terraform 配置來瞭解 Terraform 配置的基礎知識。

1. 造訪 https://github.com/IBM-Cloud/LAMP-terraform-ibm，並將您自己的副本**分出**到您的帳戶。
2. 在本端複製分出：
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. 檢查配置檔
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml) - 包含伺服器安裝配置，在此可以新增與伺服器安裝相關以及與要在伺服器上安裝的內容相關的所有 Script。請參閱注入到此檔案中的 `phpinfo();`。
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf) - 包含與提供者相關的變數，其中需要提供者的使用者名稱和 API 金鑰。
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) - 包含使用指定變數部署 VM 的伺服器配置。
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) - 包含 **SoftLayer** 使用者名稱和 API 金鑰、{{site.data.keyword.Bluemix_notm}} API 金鑰以及您的空間/組織名稱。可以將這些認證新增到此檔案中以取得最佳作法，避免每次部署伺服器時，都要從指令行重新輸入這些認證。附註：不要使用您的認證發佈此檔案。
4. 在所選的 IDE 中開啟 [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) 檔案，然後藉由新增**公用 SSH** 金鑰來修改該檔案。此金鑰將用於存取此配置建立的 VM。如需建立 SSH 金鑰的相關資訊，請遵循[此鏈結](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html)中的指示進行作業。若要將公開金鑰複製到剪貼簿，可以在終端機中執行以下指令。

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     此指令將 SSH 複製到剪貼簿，然後可以將其貼到 [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) 中大約第 69 行的 `ssh_key` 預設變數下。

    在 **Windows** 上，下載、安裝並啟動 [Git Bash](https://git-scm.com/download/win)，然後執行以下指令，以將公用 SSH 金鑰複製到剪貼簿。

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. 套用 IDE 開啟 [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) 檔案，藉由新增列出的所有認證來修改該檔案；在該檔案中新增這些認證意味著以後每次執行 terraform apply 時，無需重新輸入這些認證。為了完成本指導教學的其餘部分，您必須新增 [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) 檔案中列出的所有五個認證。

## 從 Terraform 配置建立 LAMP 堆疊伺服器
{: #Createserver}
在本節中，您將瞭解如何從 Terraform 配置範例來建立 LAMP 堆疊伺服器。該配置用於佈建虛擬機器實例，並將 **A**pache、**M**ySQL (**M**ariaDB) 和 **P**HP 安裝在該實例上。

1. 導覽至複製的儲存庫所在的資料夾。
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. 起始設定 Terraform 配置。這還將安裝 `terraform-provider-ibm_VERSION` 外掛程式。
   ````bash
    terraform init
   ````
   {: pre}
3. 套用 Terraform 配置。這將建立在配置中定義的資源。
   ```
    terraform apply
   ```
   {: pre}
   您應該會看到類似於下面的輸出。![來源控制 URL](images/solution10/created.png)
4. 接下來，前往[基礎架構裝置清單](https://{DomainName}/classic/devices)以驗證伺服器是否已建立。![來源控制 URL](images/solution10/configuration.png)

## 新增 {{site.data.keyword.cos_full_notm}} 服務和調整資源

{: #modify}

在本節中，您將瞭解如何調整虛擬伺服器資源，並將 [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) 服務新增到基礎架構環境。

1. 編輯 `vm.tf` 檔案以增加下列各項的值，然後儲存該檔案。
 - 將 CPU 核心數目增加到 4 個核心
 - 將 RAM（記憶體）增加到 4096
 - 將磁碟大小增加到 100 GB

2. 接下來，新增 [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) 服務，為此，請建立一個新檔案，並將其命名為 **ibm-cloud-object-storage.tf**。將以下程式碼 Snippet 新增到新建立的檔案中。這些程式碼 Snippet 為組織名稱和空間名稱建立變數名稱，然後這兩個變數名稱用於擷取建立服務所需的空間 GUID。程式碼將 {{site.data.keyword.cos_full_notm}} 服務名稱設定為 `lamp_objectstorage`，然後您需要空間 GUID、服務完整名稱和方案類型。以下程式碼將建立超值方案，這是隨收隨付制方案。您還可以使用精簡方案，但請注意，精簡方案限制為一個帳戶僅一個服務。

   ```
   variable "org_name" {
     description = "Enter your IBM Cloud org name, you can get your org name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   variable "space_name" {
     description = "Enter your IBM Cloud space name, you can get your space name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   data "ibm_space" "space" {
     space = "${var.space_name}"
     org   = "${var.org_name}"
   }

   # Cloud Object Storage
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # you can only have one Lite plan per account so let's use the Premium - it is pay-as-you-go
     plan = "Premium"
   }
   ```
   {: pre}
   **附註：**我們稍後會在日誌中尋找標籤 "lamp_objectstorage""，以確保 {{site.data.keyword.cos_full_notm}} 已順利建立。

3. 藉由執行下列指令，再次起始設定 Terraform 配置：

   ```bash
    terraform init
   ```
   {: pre}

4. 藉由執行下列指令，套用 Terraform 變更：
   ```bash
    terraform apply
   ```
   {: pre}
   **附註：**順利執行 terraform apply 指令後，您應該會看到新的 `terraform.tfstate` 檔案已新增到目錄中。此檔案包含完整的部署確認，可追蹤上次套用的內容以及未來對配置的任何修改。如果此檔案被移除或遺失，則您將遺失 Terraform 部署配置。

## 驗證 VM 和 {{site.data.keyword.cos_short}}
{: #verifyvm}

在本節中，您將驗證 VM 和 {{site.data.keyword.cos_short}} 以確保已順利進行建立。

**驗證 VM**

1. 在左側功能表中，按一下**基礎架構**以檢視虛擬伺服器裝置的清單。
2. 按一下**裝置** -> **裝置清單**以尋找建立的伺服器。您應該會看到您的伺服器裝置列出。
3. 按一下伺服器以檢視有關伺服器配置的相關資訊。請查看下面的擷取畫面，在其中可以看到伺服器已順利建立。![來源控制 URL](images/solution10/configuration.png)
4. 接下來，將在 Web 瀏覽器中測試伺服器。在 Web 瀏覽器中開啟伺服器公用 IP 位址。您應該會看到類似於下圖的伺服器預設安裝頁面。![來源控制 URL](images/solution10/LAMP.png)


**驗證 {{site.data.keyword.cos_full_notm}}**

1. 在 **{{site.data.keyword.Bluemix_notm}} 儀表板**中，您應該會看到 {{site.data.keyword.cos_full_notm}} 服務實例已建立並可供使用。![object-storage](images/solution10/ibm-cloud-object-storage.png)

   可以在[這裡](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)找到有關 {{site.data.keyword.cos_full_notm}} 的進一步資訊。

## 移除資源
{: #deleteresources}

使用下列指令刪除資源：
   ```bash
   terraform destroy
   ```
   {: pre}

**附註：**若要刪除資源，您需要基礎架構管理許可權。如果您沒有管理超級使用者帳戶，請使用基礎架構儀表板來要求取消資源。可以在基礎架構儀表板中的「裝置」下要求取消裝置。![object-storage](images/solution10/rm.png)

## 相關內容

- [Terraform ](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [使用 CDN 加速交付靜態檔案 - {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

