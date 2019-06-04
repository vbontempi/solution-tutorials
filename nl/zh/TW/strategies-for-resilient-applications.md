---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-11"
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

# 回復型應用程式的策略
{: #strategies-for-resilient-applications}

無論是運算選項、Kubernetes、Cloud Foundry、Cloud Functions 還是虛擬伺服器，企業都在尋求能將關閉時間降至最低，並建立可實現最大可用性的回復型架構。本指導教學重點闡述了 IBM Cloud 建置回復型解決方案的功能，並在此過程中回答了下列問題。

- 在準備要在全球可用的解決方案時，應該考慮什麼？
- 可用的運算選項如何協助交付多地區應用程式？
- 如何將應用程式或服務構件匯入到其他地區？
- 資料庫如何在不同位置之間進行抄寫？
- 應該使用哪些後端服務：Block Storage、File Storage、Object Storage 或 Databases？
- 有任何服務特定的考慮因素嗎？

## 目標
{: #objectives}

* 瞭解建置回復型應用程式時涉及的架構概念
* 瞭解此類概念如何對映到 IBM Cloud 運算和服務供應項目

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構和概念

{: #architecture}

若要設計回復型架構，您需要考慮解決方案的各個構成區塊及其特定功能。 

下面是一個多地區架構，展示了多地區設定中可能存在的不同元件。![架構](images/solution39/Architecture.png)

上面的架構圖可能根據運算選項而有所不同。您將在後面的各區段中看到每個運算選項下的特定架構圖。 

### 具有兩個地區的災難回復 

為了方便進行災難回復，使用了兩種廣泛接受的架構：**主動/主動**和**主動/被動**。每種架構都有自己的成本，並在回復期間的時間和工作量方面各有優點。

#### 主動/主動配置

在主動/主動架構中，兩個位置都具有完全相同的主動實例，透過負載平衡器在兩者之間配送資料流量。使用此方法時，必須實施資料抄寫，以即時同步兩個地區之間的資料。

![主動/主動](images/solution39/Active-active.png)

與主動/被動架構相比，此配置提供了更高的可用性，需要的手動補救工作更少。要求透過兩個資料中心進行處理。應該為邊緣服務（負載平衡器）配置適當的逾時和重試邏輯，以便在第一個資料中心環境中發生故障時，自動將要求遞送到第二個資料中心。

在主動/主動方案中考慮**回復點目標** (RPO) 時，兩個主動資料中心之間的資料同步化必須非常及時，以容許無縫要求流程。

#### 主動/被動配置

主動/被動架構依賴於一個主動地區和用作備份的另一個（被動）地區。如果主動地區發生中斷，被動地區會變為主動地區。可能需要人為介入，以確保資料庫或檔案儲存空間與應用程式和使用者需求保持同步。 

![主動/主動](images/solution39/Active-passive.png)

要求透過主動網站進行處理。如果發生中斷或應用程式故障，將執行應用程式準備工作，以使備用資料中心準備好處理要求。從主動資料中心切換到被動資料中心是一項很耗時的作業。相比主動/主動配置，**回復時間目標** (RTO) 和**回復點目標** (RPO) 都要更長。

### 具有三個地區的災難回復

在如今對關閉時間零容忍、服務「永遠可用」的時代，客戶希望每個商業服務在世界範圍內隨時隨地保持可存取。對企業而言，有成本效益的策略涉及構造實現持續可用性的基礎架構，而不是建置災難回復基礎架構。

使用三個資料中心提供的備援能力和可用性要高於兩個資料中心。此外，還可以透過在各資料中心之間更均勻地分散負載，以便能提供更佳效能。如果企業只有兩個資料中心，這種部署的變式是在一個資料中心部署兩個應用程式，在另一個資料中心部署第三個應用程式。或者，可以採用三主動拓蹼形式部署商業邏輯和呈現層，以雙主動拓蹼形式部署資料層。

#### 主動/主動/主動（三主動）配置

![](images/solution39/Active-active-active.png)

要求由在三個主動資料中心中的任何一個中心執行的應用程式進行處理。IBM.com 網站上的一個案例研究表明，三主動只需要佔用每個叢集 50% 的運算、記憶體和網路容量，但雙主動需要佔用每個叢集 100% 的相應資源。資料層的成本差異非常明顯。如需進一步的詳細資料，請閱讀 [*AlwaysOn: Assess, Design, Implement, and Manage Continuous Availability*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf)。

#### 主動/主動/被動配置

![](images/solution39/Active-active-passive.png)

在此情境中，主資料中心和輔助資料中心內的兩個主動應用程式之一發生中斷時，第三個資料中心的備用應用程式會啟動。將遵循雙資料中心情境中說明的災難回復程序，以還原為正常處理客戶要求。第三個資料中心的備用應用程式可以設定為快速待命或冷備用配置。

如需災難回復的更多資訊，請參閱[此手冊](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/)。

### 多地區架構

在多地區架構中，應用程式會部署到不同的位置，在這些位置中，每個地區執行完全相同的應用程式副本。 

地區是可以部署 APP、服務和其他 {{site.data.keyword.cloud_notm}} 資源的特定地理位置。[{{site.data.keyword.cloud_notm}} 地區](https://{DomainName}/docs/containers?topic=containers-regions-and-zones)由一個以上的區域組成，區域是實體資料中心，用於管理運算、網路和儲存空間資源以及相關冷卻系統和電源，服務和應用程式透過這些資源管理。區域彼此隔離，以確保不會共用單一失敗點。


此外，在多地區架構中，需要廣域負載平衡器（如 [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)），以便在地區之間配送資料流量。

跨多個地區部署解決方案有下列優點：
- 縮短一般使用者的延遲 - 速度是關鍵，後端原點離一般使用者越近，使用者體驗越好，速度越快。
- 災難回復 - 主動地區發生故障時，可以使用備份地區快速回復。
- 業務需求 - 在某些情況下，需要將資料儲存在相隔數百公里的不同地區中。因此，在這種情況下，您必須將資料儲存在多個地區中。 

### 地區架構內有多個區域

建置多區域地區應用程式意味著將應用程式部署在一個地區的不同區域中，而且之後您還可能有兩個或三個地區。 

使用多區域地區架構，您需要有本端負載平衡器在一個地區的不同區域之間本端分散資料流量，而如果之後設定了第二個地區，則廣域負載平衡器會在這些地區之間分散資料流量。 

可以在[這裡](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones)瞭解有關地區和區域的更多資訊。

## 運算選項 

此區段再次探討 {{site.data.keyword.cloud_notm}} 中可用的運算選項。對於每個運算選項，提供了相應的架構圖以及有關如何部署此類架構的指導教學。

附註：所有運算選項的架構都沒有包含資料庫或其他服務，只專注於將應用程式部署到所選運算選項的兩個地區。但部署了任何多地區運算選項範例後，下一個邏輯步驟就是新增資料庫和其他服務。本解決方案指導教學的後面區段將闡述[資料庫](#databaseservices)和[非資料庫服務](#nondatabaseservices)。

### Cloud Foundry 

Cloud Foundry 提供了實現多地區架構部署的功能，而使用[持續交付](https://{DomainName}/catalog/services/continuous-delivery)管道服務也容許您在多個地區中部署應用程式。Cloud Foundry 多地區的架構類似於下圖：

![CF 架構](images/solution39/CF2-Architecture.png)

相同的應用程式部署在多個地區中，廣域負載平衡器將資料流量遞送到距離最近且性能良好的地區。[**跨多個地區保護 Web 應用程式**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)指導教學將指導您完成類似架構的部署。

### {{site.data.keyword.cfee_full_notm}}

**{{site.data.keyword.cfee_full_notm}} (CFEE)** 提供與公用 Cloud Foundry 相同的所有功能，以及其他特性。

透過 **{{site.data.keyword.cfee_full_notm}}**，您可以依需求實例化多個隔離的企業級 Cloud Foundry 平台。在 [{{site.data.keyword.cloud_notm}}](http://ibm.com/cloud) 中，CFEE 的實例會在您自己的帳戶中執行。環境部署在基於 [{{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service) 的隔離硬體上。您擁有對環境的完全控制權，包括存取控制、容量管理、變更管理、監視和服務。

下面是使用 {{site.data.keyword.cfee_full_notm}} 的多地區架構。

![架構](images/solution39/CFEE-Architecture.png)

部署此架構需要下列操作： 

- 設定兩個 CFEE 實例 - 每個地區一個實例。
- 建立服務並將其連結到 CFEE 帳戶。 
- 推送以 CFEE API 端點為目標的 APP。 
- 設定資料庫抄寫，就像在公用 Cloud Foundry 上執行一樣。 

此外，請查看逐步手冊：[Deploy Logistics Wizard to Cloud Foundry Enterprise Environment (CFEE)](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md)。此指南將引導您逐步將以微服務為基礎的應用程式部署到 CFEE。部署到一個 CFEE 實例後，可以將該程序抄寫到第二個地區，並在兩個 CFEE 實例的前端連接 [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)，以對資料流量進行負載平衡。 

如需其他詳細資料，請參閱 [{{site.data.keyword.cfee_full_notm}} 文件](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about)。

### Kubernetes

透過 Kubernetes，可以在地區架構內有多個區域，這可以是主動/主動使用案例。使用 {{site.data.keyword.containershort_notm}} 實作解決方案時，可以利用各種內建功能（如負載平衡和隔離），提高對於主機、網路或應用程式潛在故障的備援能力。藉由建立多個叢集，以及一個叢集發生停機的情況下，使用者仍然可以存取也部署在另一個叢集裡的應用程式。利用位於不同地區的多個叢集，使用者還可以存取距離最近的叢集，縮短網路延遲。為了獲得更多備援能力，還可以選取多區域叢集，這意味著節點部署在一個地區的多個區域中。 

Kubernetes 多地區架構類似於下圖。

![Kubernetes](images/solution39/Kub-Architecture.png)

1. 開發人員建置應用程式的 Docker 映像檔。
2. 將映像檔推送到兩個不同位置的 {{site.data.keyword.registryshort_notm}}。
3. 應用程式部署到兩個位置的 Kubernetes 叢集。
4. 一般使用者存取應用程式。
5. Cloud Internet Services 配置為截取對應用程式的要求，以及在叢集之間分散負載。此外，已啟用 DDoS 保護和 Web 應用程式防火牆，保護應用程式抵禦常見威脅。選擇性地快取資產，例如影像、CSS 檔。

[**使用 Cloud Internet Services 的具復原力且安全的多地區 Kubernetes 叢集**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)指導教學可全程指導您完成部署此類架構的步驟。

### {{site.data.keyword.openwhisk_short}}

{{site.data.keyword.openwhisk_short}} 在多個 {{site.data.keyword.cloud_notm}} 位置都有提供。為了提高備援及縮短網路延遲，應用程式可以在多個位置部署其後端。使用 IBM Cloud Internet Services (CIS)，開發人員可以公開單一進入點，來負責將資料流量分散至最近的性能健全後端。{{site.data.keyword.openwhisk_short}} 多地區的架構類似於下圖。

 ![Functions 架構](images/solution39/Functions-Architecture.png)

1. 使用者存取應用程式。要求會透過 Internet Services。
2. Internet Services 將使用者重新導向到最近的性能健全 API 後端。
3. Certificate Manager 為 API 提供其 SSL 憑證。資料流量會全程加密。
4. API 透過 Cloud Functions 實作。

透過遵循[**跨多個地區部署無伺服器 APP**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless)指導教學，瞭解如何部署此架構。

### {{site.data.keyword.baremetal_short}} 和 {{site.data.keyword.virtualmachinesshort}}

{{site.data.keyword.virtualmachinesshort}} 和 {{site.data.keyword.baremetal_short}} 提供了實現多地區架構的功能。可以在 {{site.data.keyword.cloud_notm}} 上的多個可用位置佈建伺服器。

![伺服器位置](images/solution39/ServersLocation.png)

在使用 {{site.data.keyword.virtualmachinesshort}} 和 {{site.data.keyword.baremetal_short}} 準備此類架構時，請考慮下列事項：檔案儲存空間、備份、回復和資料庫，在資料庫即服務與在虛擬伺服器上安裝資料庫這兩個選項中做出選擇。 

以下架構示範了如何在主動/被動架構（其中，一個地區為主動，另一個地區為被動）中使用 {{site.data.keyword.virtualmachinesshort}} 部署多地區架構。 

![VM 架構](images/solution39/vm-Architecture2.png)

此類架構需要的元件： 

1. 使用者透過 IBM Cloud Internet Services (CIS) 存取應用程式。
2. CIS 將資料流量遞送到主動位置。
3. 在位置中，負載平衡器會將資料流量重新導向至伺服器。
4. 部署在虛擬伺服器上的資料庫，這意味著將配置資料庫，並設定地區之間的抄寫和備份。替代方案是使用資料庫即服務，此主題將在本指導教學後面進行討論。
5. 用於儲存應用程式映像檔和檔案的檔案儲存空間，檔案儲存空間提供了在給定時間和日期建立 Snapshot 的功能，然後此 Snapshot 可以在另一個地區中重複使用，這需要手動執行。 

[**使用虛擬伺服器建置具備高可用性和高可調整性的 Web 應用程式**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application)指導教學實作此架構。

## 資料庫和應用程式檔案
{: #databaseservices}

{{site.data.keyword.cloud_notm}} 提供了一組精選的[資料庫即服務](https://{DomainName}/catalog/?category=databases)，包括關聯式資料庫和非關聯式資料庫，您可根據自己的業務需求選擇使用。[資料庫即服務 (DBaaS)](https://www.ibm.com/cloud/learn/what-is-cloud-database) 有許多優點。藉由 DBaaS（如 {{site.data.keyword.cloudant}}），可以利用多地區支援，以容許您在不同地區的兩個資料庫服務之間進行即時抄寫、執行備份、調整規模，以及提供最長的執行時間。 

**主要功能：** 

- 提供透過雲端平台建置和存取的資料庫服務
- 可讓企業使用者管理資料庫，而無需購買專用硬體
- 可以由使用者管理或由供應商以即服務方式提供並進行管理
- 可以支援 SQL 或 NoSQL 資料庫
- 可以透過 Web 介面或供應商提供的 API 進行存取

**準備多地區架構：**

- 資料庫服務有哪些備援選項？
- 如何處理不同地區中多個資料庫服務之間的抄寫？
- 如何備份資料？
- 針對每種情況有哪些災難回復方法？

### {{site.data.keyword.cloudant}}

{{site.data.keyword.cloudant}} 是一種分散式資料庫，經過最佳化可以處理快速增長的大型 Web 及行動應用程式所典型具備的繁重工作負載。{{site.data.keyword.cloudant}} 提供作為 SLA 支援的完全受管理的 {{site.data.keyword.Bluemix_notm}} 服務，可以獨立地彈性調整傳輸量和儲存空間量。{{site.data.keyword.cloudant}} 還作為可下載的內部部署安裝提供，其 API 和強大的抄寫通訊協定與開放程式碼生態系統相容，該生態系統包括 CouchDB、PouchDB 以及適用於最常用 Web 及行動開發堆疊的程式庫。

{{site.data.keyword.cloudant}} 支援在不同位置的多個實例之間進行[抄寫](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation)。來源資料庫中發生的任何變更都將在目標資料庫中重新產生。可以在任意數目的資料庫之間建立抄寫，抄寫方式可以是連續的，也可以是「一次性」作業。下圖顯示了使用兩個 {{site.data.keyword.cloudant}} 實例的典型配置，每個地區一個實例：

![主動/主動](images/solution39/Active-active.png)

請參閱[這些指示](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery)來配置 {{site.data.keyword.cloudant}} 實例之間的抄寫。該服務還提供了用於[備份和還原資料](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery)的指示和工具。

### {{site.data.keyword.Db2_on_Cloud_short}}、{{site.data.keyword.dashdbshort_notm}} 和 {{site.data.keyword.Db2Hosted_notm}}

{{site.data.keyword.cloud_notm}} 提供了多個 [Db2 資料庫服務](https://{DomainName}/catalog/?search=db2h)。包括：

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2)：完全受管理的雲端 SQL 資料庫，適用於典型的類似 OLTP 的操作工作負載。
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse)：完全受管理的雲端資料倉儲服務，適用於高效能、 PB 級分析工作量。此服務提供 SMP 和 MPP 服務方案，並利用經過最佳化的柱狀資料儲存庫和記憶體內處理。
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted)：由 IBM 管理並由使用者資料庫系統進行管理。此服務為 Db2 提供了對雲端基礎架構的完全管理存取權，因此消除了與管理您自己的基礎架構相關的成本、複雜性和風險。

我們將在下面說明用於操作工作負載的作為 DBaaS 的 {{site.data.keyword.Db2_on_Cloud_short}}。這些工作負載是本指導教學中討論的應用程式的典型工作負載。

#### {{site.data.keyword.Db2_on_Cloud_short}} 的多地區支援

{{site.data.keyword.Db2_on_Cloud_short}} 提供了多個[用於實現高可用性和災難回復 (HADR) 的選項](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview)。可以在建立新服務時選擇「高可用性」選項。也可以日後透過實例儀表板[新增地理複製的災難回復節點](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha)。藉由非現場 DR 節點選項，可以將資料即時同步到所選的非現場 {{site.data.keyword.cloud_notm}} 資料中心的資料庫節點。

相關資訊可在[高可用性文件](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha)中取得。

#### 備份及還原

{{site.data.keyword.Db2_on_Cloud_short}} 包含針對付費方案的每日備份。通常，備份會使用 {{site.data.keyword.cos_short}} 進行儲存，以便能利用三個資料中心來提高保留資料的可用性。備份會保留 14 天。可以使用備份來執行復原點回復。[備份和還原文件](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br)提供了有關如何將資料還原到所需日期和時間的詳細資料。

### {{site.data.keyword.databases-for}}

{{site.data.keyword.databases-for}} 提供了多個開放程式碼資料庫系統作為完全受管理服務。包括：  
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

所有這些服務都具有相同的性質：   
* 為了實現高可用性，這些服務會部署在叢集中。詳細資料可在各個服務的文件中找到：
  - [PostgreSQL ](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* 每個叢集分散在多個區域中。
* 資料在各區域之間進行抄寫。
* 使用者可以擴展實例的儲存空間和記憶體資源。如需詳細資料，請參閱[有關調整的文件，例如 {{site.data.keyword.databases-for-redis}}](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings)。
* 備份每天或依需求執行。將記錄各個服務的詳細資料。這裡是 [{{site.data.keyword.databases-for-postgresql}} 等的備份文件](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups)。
* 靜態資料、備份和網路資料流量都會加密。
* 各個[服務可以使用 {{site.data.keyword.databases-for}} CLI 外掛程式來管理](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in)


### {{site.data.keyword.cos_full_notm}}

{{site.data.keyword.cos_full_notm}} (COS) 提供耐久、安全且有成本效益的雲端儲存空間。儲存在 {{site.data.keyword.cos_full_notm}} 中的資訊加密並分散在多個地理位置。在 COS 實例中建立儲存空間儲存區時，可以決定應在哪個位置建立儲存空間儲存區以及使用哪個備援選項。

有三種類型的儲存區備援：
   - **跨地區**備援將在多個大城市地區中分散資料。可以將此項視為多地區選項。存取儲存在「跨地區」儲存區中的內容時，COS 提供了一個特殊端點，能夠從性能良好的地區中擷取內容。
   - **地區**備援將在單一大城市地區中分散資料。可以將此項視為一個地內有多個區域的配置。
   - **單一資料中心**備援將在單一資料中心內的多個應用裝置中分散資料。

如需 {{site.data.keyword.cos_full_notm}} 備援選項的詳細說明，請參閱[此文件](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints)。

### {{site.data.keyword.filestorage_full_notm}}

{{site.data.keyword.filestorage_full_notm}} 是一種網路連接、以 NFS 為基礎的檔案儲存空間，具有持久、快速、靈活的特點。在這個網路連接儲存空間 (NAS) 環境中，您可以完全控制檔案共用功能及效能。{{site.data.keyword.filestorage_short}} 共用可透過已遞送的 TCP/IP 連線來連接至最多 64 個已授權裝置，以便能實現備援能力。

一些檔案儲存空間特性包括_Snapshot_、_抄寫_和_並行存取_。如需完整的特性清單，請參閱[文件](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage)。

連接到伺服器後，可以輕鬆地使用 {{site.data.keyword.filestorage_short}} 服務來儲存資料備份以及影像和視訊等應用程式檔案，然後可以在同一地區的不同伺服器中使用這些影像和檔案。

新增第二個地區時，可以使用 {{site.data.keyword.filestorage_short}} 的 Snapshot 特性自動或手動建立 Snapshot，然後在第二個被動地區內重複使用此 Snapshot。 

可以排定抄寫以自動將 Snapshot 抄寫到遠端資料中心的目的地磁區。如果發生災難性事件，或者您的資料毀損，則可以在遠端網站中回復副本。有關 File Storage Snapshot 的更多資訊，可以在[這裡](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots)找到。

## 非資料庫服務
{: #nondatabaseservices}

{{site.data.keyword.cloud_notm}} 提供了一組精選的非資料庫[服務](https://{DomainName}/catalog)，包括 IBM 服務和第三方服務。在規劃多地區架構時，需要瞭解各種服務（如 Watson 服務）如何在多地區設定中工作。

### {{site.data.keyword.conversationfull}}

[{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about) 是一個平台，容許開發人員和非技術使用者進行分工合作，共同建置基於 AI 技術的交談式助理。

助理是一種認知機器人，您可以針對商業需求予以自訂，並且跨多個頻道進行部署，以隨時隨地提供客戶所需的協助。
助理包含一項或多項技能。對話技能包含可讓助理協助您客戶的訓練資料和邏輯。


值得注意的是，{{site.data.keyword.conversationshort}} V1 是無狀態的。透過 {{site.data.keyword.conversationshort}}，可實現 99.5% 的執行時間，但對於跨多個地區的高可用性應用程式，您可能還是希望在不同地區中擁有此服務的多個實例。 

在多個位置建立實例後，可使用工具 {{site.data.keyword.conversationshort}} 從一個實例中匯出現有工作區，包括目的、實體和對話。然後在其他位置中匯入此工作區。

## 摘要

|供應項目|備援選項|
| -------- | ------------------ |
|Cloud Foundry| <ul><li>將應用程式部署至多個位置</li><li>從多個位置使用 Cloud Internet Services 服務要求</li><li>使用 Cloud Foundry API 來配置組織、空間，以及將應用程式推送至多個位置</li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>將應用程式部署至多個位置</li><li>從多個位置使用 Cloud Internet Services 服務要求</li><li>使用 Cloud Foundry API 來配置組織、空間，以及將應用程式推送至多個位置</li><li>基於 Kubernetes Service 建置</li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>精心設計的備援能力，支援多區域叢集</li><li>使用 Cloud Internet Services 處理來自分散在多個位置的叢集的要求</li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>在多個位置中可用</li><li>從多個位置使用 Cloud Internet Services 服務要求</li><li>使用 Cloud Functions API 在多個位置部署動作</li></ul> |
| {{site.data.keyword.baremetal_short}} 和 {{site.data.keyword.virtualmachinesshort}} | <ul><li>在多個位置中佈建伺服器</li><li>將同一位置的伺服器連接到本端負載平衡器</li><li>從多個位置使用 Cloud Internet Services 服務要求</li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>在資料庫之間進行一次性抄寫和持續抄寫</li><li>地區內自動資料冗餘</li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>佈建地理複製的災難回復節點，以實現即時資料同步化</li><li>每日備份（付費方案中提供）</li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>基於多區域 Kubernetes 叢集建置</li><li>跨地區讀取抄本</li><li>每日備份和依需求備份</li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>單一資料中心、地區和跨地區備援</li><li>使用 API 在各儲存空間儲存區之間同步內容</li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>使用 Snapshot 自動將內容擷取到遠端資料中心內的目的地</li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>使用 Watson API 在不同位置的多個實例之間匯出和匯入工作區規格</li></ul> |

## 相關內容

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry - 跨多個地區保護 Web 應用程式](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [CloudFunctions - 跨多個地區部署無伺服器 APP](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes - 使用 Cloud Internet Services 的具復原力且安全的多地區 Kubernetes 叢集](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [虛擬伺服器 - 建置具備高可用性和高可調整性的 Web 應用程式](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
