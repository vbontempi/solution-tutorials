---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# 關於組織使用者、團隊、應用程式的最佳作法
{: #users-teams-applications}

本指導教學概要介紹了 {{site.data.keyword.cloud_notm}} 中提供的用於管理身分和存取權管理的概念，以及如何實作這些概念以支援應用程式的多個開發階段。
{:shortdesc}

在建置應用程式時，通常會定義多個環境以反映從開發人員確定程式碼到應用程式碼可用於一般使用者的整個專案開發生命週期。這些環境通常會使用如下名稱：*Sandbox*、*test*、*staging*、*UAT*（使用者驗收測試）、*pre-production*、*production*。

隔離基礎資源、實作監管和存取原則、保護正式作業工作負載、驗證變更後再將其推送到正式作業中，這些都是您需要建立這些個別環境的一些原因。

## 目標
{: #objectives}

* 瞭解 {{site.data.keyword.iamlong}} 和 Cloud Foundry 存取模型
* 配置專案並隔離各個角色和環境
* 設定持續整合

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 定義專案

讓我們考慮一下具有下列元件的專案範例：
* 在 {{site.data.keyword.containershort_notm}} 中部署的幾個微服務、
* 資料庫、
* 檔案儲存空間儲存區。

在此專案中，我們定義了三個環境：
* *開發* - 此環境隨著每次執行確定、單元測試、煙霧測試而持續更新。透過該環境，可以存取最新、最大的專案部署。
* *測試* - 此環境是依據程式碼的穩定分支或標籤來建置的。在這裡進行使用者驗收測試。其配置類似於正式作業環境。其中載入的是真實資料（以匿名的正式作業資料為例）。
* *正式作業* - 此環境使用先前環境中驗證的版本進行更新。

**交付管線透過該環境來管理建置的進展。**它既可以完全自動化，也可以包含手動驗證關卡以促進各環境之間的核准建置 - 這是真正開放式的，並且應該設定為與公司的最佳作法和工作流程相符合。

為了支援執行建置管道，我們引入了**功能使用者**，這是一個一般的 {{site.data.keyword.cloud_notm}} 使用者，但也是在現實世界中沒有真實身分的團隊成員。此功能使用者將擁有交付管線以及需要強所有權的其他任何雲端資源。當團隊成員離開公司或者調到其他專案時，此方法會很有幫助。功能使用者將專用於您的專案，在專案生命期限內不會變更。接下來您需要為此功能使用者建立 [API 金鑰](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey)。在設定 DevOps 管線時或者在您要執行自動化 Script 時，您將選取此 API 金鑰，以模擬功能使用者。

在為專案團隊成員指派職責方面，讓我們定義下列角色和相關許可權：

|           |開發|測試|正式作業|
| --------- | ----------- | ------- | ---------- |
|開發人員| <ul><li>提出程式碼</li><li>可以存取日誌檔</li><li>可以檢視應用程式及服務配置</li><li>使用已部署的應用程式</li></ul> | <ul><li>可以存取日誌檔</li><li>可以檢視應用程式及服務配置</li><li>使用已部署的應用程式</li></ul> | <ul><li>無存取權</li></ul> |
|測試人員| <ul><li>使用已部署的應用程式</li></ul> | <ul><li>使用已部署的應用程式</li></ul> | <ul><li>無存取權</li></ul> |
|操作員 | <ul><li>可以存取日誌檔</li><li>可以檢視/設定應用程式及服務配置</li></ul> | <ul><li>可以存取日誌檔</li><li>可以檢視/設定應用程式及服務配置</li></ul> | <ul><li>可以存取日誌檔</li><li>可以檢視/設定應用程式及服務配置</li></ul> |
|管線功能使用者| <ul><li>可以部署/取消部署應用程式</li><li>可以檢視/設定應用程式及服務配置</li></ul> | <ul><li>可以部署/取消部署應用程式</li><li>可以檢視/設定應用程式及服務配置</li></ul> | <ul><li>可以部署/取消部署應用程式</li><li>可以檢視/設定應用程式及服務配置</li></ul> |

## Identity and Access Management (IAM)
{: #first_objective}

透過 {{site.data.keyword.iamshort}} (IAM)，您可以安全地對平台服務和基礎架構服務的使用者進行鑑別，並在整個 {{site.data.keyword.cloud_notm}} 平台上以一致的方式控制對**資源**的存取權。讓一組 {{site.data.keyword.cloud_notm}} 服務能夠使用 Cloud IAM 進行存取控制，這些服務在您的**帳戶**中組織成**資源群組**，讓**使用者**能夠快速、輕鬆地一次存取多個資源。Cloud IAM 存取**原則**用於為使用者和服務 ID 指派對帳戶中資源的存取權。

**原則**透過屬性組合來定義存取權的範圍，以便為使用者或服務 ID 指派一個以上的**角色**。原則可以提供對單一服務下至實例層次的存取權，原則也可以套用於群組織成資源群組的一群組資源。根據指派的使用者角色，容許使用者或服務 ID 具有不同層次的存取權，以透過使用使用者介面或執行特定類型的 API 呼叫來完成平台管理作業或存取服務。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="IAM 模型圖" />
</p>

目前還不能使用 IAM 來管理 {{site.data.keyword.cloud_notm}} 型錄中的所有服務。對於這些服務，您可以繼續使用 Cloud Foundry，方法是透過指派 Cloud Foundry 角色來定義容許的存取層次，為使用者提供對實例所屬組織和空間的存取權。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Cloud Foundry 模型圖" />
</p>

## 為一個環境建立資源

雖然此專案範例所需的三種環境要求不同的存取權，還可能需要配置不同的容量，但它們共用同一個架構模式。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="顯示一個環境的架構圖" />
</p>

我們先來建置開發環境。

1. [選取一個要用於部署環境的 {{site.data.keyword.cloud_notm}} 位置](https://{DomainName})。
1. 對於 Cloud Foundry 服務和 APP：
   1. [為專案建立組織](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg)。
   1. [為環境建立 Cloud Foundry 空間](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo)。
   1. 在此空間下建立專案要使用的 Cloud Foundry 服務
1. [為環境建立資源群組](https://{DomainName}/account/resource-groups)。
1. 在此群組中建立與資源群組相容的服務，如 {{site.data.keyword.cos_full_notm}}、{{site.data.keyword.la_full_notm}}、{{site.data.keyword.mon_full_notm}} 和 {{site.data.keyword.cloudant_short_notm}}。
1. 在 {{site.data.keyword.containershort_notm}} 中[建立新的 Kubernetes 叢集](https://{DomainName}/containers-kubernetes/catalog/cluster)，確保選取上面建立的資源群組。
1. 配置 {{site.data.keyword.la_full_notm}} 和 {{site.data.keyword.mon_full_notm}} 以傳送日誌並監視叢集。

下圖顯示在帳戶下建立專案資源的位置：

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="顯示專案資源的圖" />
</p>

## 在環境中指派角色

1. 邀請使用者加入帳戶
1. 為使用者指派原則以控制誰可以存取資源群組、群組中的服務以及 {{site.data.keyword.containershort_notm}} 實例及其許可權。請參閱[存取原則定義](https://{DomainName}/docs/containers?topic=containers-users#access_policies)來為環境中的使用者選取正確的原則。可以將具有相同原則集的使用者放入[相同的存取群組](https://{DomainName}/docs/iam?topic=iam-groups#groups)中。這樣就簡化了使用者管理，因為原則將指派給存取群組，並由群組中的所有使用者繼承。
1. 在環境中根據需求來配置 Cloud Foundry 組織和空間角色。請參閱[角色定義](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess)來根據環境指派正確的角色。

請參閱服務文件，瞭解服務如何將 IAM 和 Cloud Foundry 角色對映到特定動作。請參閱範例 [{{site.data.keyword.mon_full_notm}} 服務如何將 IAM 角色對映到動作](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam)。

為使用者指派正確的角色需要進行多次反覆運算和精確。給定的許可權可以在資源群組層次進行控制，適用於一個群組中的所有資源，或者細化到服務的特定實例，經過一段時間之後，您會探索哪些存取原則對於您的專案比較理想。

好的做法是從先從一組最少的許可權開始，然後再根據需要慎重擴展。對於 Kubernetes，您應該看一下它的[角色型存取控制 (RBAC)](https://kubernetes.io/docs/admin/authorization/rbac/) 以配置叢集內授權。

對於開發環境，先前定義的使用者責任可以轉換為下列內容：

|           | IAM 存取原則|Cloud Foundry|
| --------- | ----------- | ------- |
|開發人員| <ul><li>資源群組：*檢視者*</li><li>資源群組中的平台存取角色：*檢視者*</li><li>記載及監視服務角色：*撰寫者*</li></ul> | <ul><li>組織角色：*審核員*</li><li>空間角色：*審核員*</li></ul> |
|測試人員| <ul><li>不需要配置。測試人員會存取已部署的應用程式，而非開發環境</li></ul> | <ul><li>無需配置</li></ul> |
|操作員 | <ul><li>資源群組：*檢視者*</li><li>資源群組中的平台存取角色：*操作員*、*檢視者*</li><li>記載及監視服務角色：*撰寫者*</li></ul> | <ul><li>組織角色：*審核員*</li><li>空間角色：*開發人員*</li></ul> |
|管線功能使用者| <ul><li>資源群組：*檢視者*</li><li>資源群組中的平台存取角色：*編輯者*、*檢視者*</li></ul> | <ul><li>組織角色：*審核員*</li><li>空間角色：*開發人員*</li></ul> |

IAM 存取原則和 Cloud Foundry 角色是在 [Identify and Access Management 使用者介面](https://{DomainName}/iam/#/users)中定義的：

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="針對開發人員角色配置許可權" />
</p>

## 抄寫多個環境

在該處，您可以重複類似步驟以建置其他環境。

1. 每個環境建立一個資源群組。
1. 每個環境建立一個叢集和需要的服務實例。
1. 每個環境建立一個 Cloud Foundry 空間。
1. 在每個空間中建立需要的服務實例。

<p style="text-align: center;">
  <img title="使用個別的叢集以隔離環境" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="顯示個別叢集以隔離環境的圖" />
</p>

使用工具組合（如 [{{site.data.keyword.cloud_notm}} `ibmcloud` CLI](https://github.com/IBM-Cloud/ibm-cloud-developer-tools)、[HashiCorp 的 `terraform`](https://www.terraform.io/)、[用於 Terraform 的 {{site.data.keyword.cloud_notm}} 提供者](https://github.com/IBM-Cloud/terraform-provider-ibm)、Kubernetes CLI `kubectl`），您可以透過編制 Script 來自動建立這些環境。

環境的個別 Kubernetes 叢集隨附一些有用的內容：
* 無論環境如何，所有叢集的外觀都差不多；
* 更容易控制誰有權存取特定叢集；
* 部署和基礎資源的更新週期更靈活；有新的 Kubernetes 版本時，您可以選擇先更新開發叢集，驗證應用程式，然後再更新其他環境；
* 避免了將可能會相互影響的不同工作負載混合在一起，如將正式作業部署與其他部署隔離開。

還有一種方法是使用 [Kubernetes 名稱空間](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)以及 [Kubernetes 資源配額](https://kubernetes.io/docs/concepts/policy/resource-quotas/)來隔離環境並控制資源消耗。

<p style="text-align: center;">
  <img title="使用個別的名稱空間來隔離環境" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="顯示個別名稱空間以隔離環境的圖" />
</p>

在 LogDNA 使用者介面的`搜尋`輸入方框中，使用`名稱空間：`欄位來根據名稱空間過濾日誌。
{: tip}

## 設定交付管線

在部署到其他環境中時，可以將您的持續整合/持續交付管線設定為驅動整個程序：
* 使用來自 `development` 分支的最新、最好的程式碼持續更新`開發`環境，並在專用叢集上執行單元測試和整合測試。
* 將開發建置升級到`測試`環境，可以是自動完成（如果先前各階段中的所有測試都沒有問題），也可以是透過手動升級流程來完成。有些團隊在這裡也會使用不同的分支，將有效的開發狀態合併到 `stable` 分支作為範例；
* 重複類似的程序以移動到`正式作業`環境。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="從建置到部署的 CI/CD 管道" />
</p>

在配置 DevOps 管道時，確保使用功能使用者的 API 金鑰。只有功能使用者應該需要具有將應用程式部署到叢集所需的權限。

在建置階段，請務必正確地對 docker 映像檔進行版本控制。您可以使用 Git 確定修訂作為映像檔標籤的一部分，也可以使用 DevOps 工具鏈提供的唯一 ID；只要有助於輕鬆將映像檔對映到該映像檔中包含的實際建置和原始碼，任何 ID 都可以。

在熟悉 Kubernetes 之後，Kubernetes 的套裝軟體管理程式 [Helm](https://helm.sh/) 就會成為對應用程式進行版本控制、組合和部署的便捷工具。[此 DevOps 工具鏈範例](https://github.com/open-toolchain/simple-helm-toolchain)很適合作為起點，並且預先配置了針對 Kubernetes 叢集的持續交付。隨著專案增長為多個微服務，[Helm 傘形 chart](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies) 將為編寫應用程式提供不錯的解決方案。

## 擴展指導教學

祝賀您，您的應用程式現在可以安全地從開發部署到正式作業。下面是改進應用程式交付的其他建議。

* 將 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 新增到管道以在部署期間執行品質控制。
* 使用 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 檢閱團隊成員的編碼貢獻情況以及開發人員之間的互動。
* 按照指導教學[計劃、建立並更新部署環境](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments)以實現環境部署自動化。

## 相關資訊

* [{{site.data.keyword.iamshort}} 開始使用](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [用資源群組組織資源的最佳作法](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [使用 LogDNA 和 Sysdig 分析日誌並監視性能](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [持續部署至 Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Hello Helm 工具鏈](https://github.com/open-toolchain/simple-helm-toolchain)
* [使用 Kubernetes 和 Helm 開發微服務應用程式](https://github.com/open-toolchain/microservices-helm-toolchain)
* [授與使用者檢視 LogDNA 中日誌的許可權](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [授與使用者檢視 Sysdig 中度量的許可權](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
