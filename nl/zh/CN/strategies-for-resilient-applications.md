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

# 弹性应用程序的策略
{: #strategies-for-resilient-applications}

无论是计算选项、Kubernetes、Cloud Foundry、Cloud Functions 还是虚拟服务器，企业都在寻求最大限度地减少停机时间，并创建可实现最大可用性的弹性体系结构。本教程重点阐述了 IBM Cloud 构建弹性解决方案的功能，并在此过程中回答了以下问题。

- 在准备要在全球可用的解决方案时，应该考虑什么？
- 可用的计算选项如何帮助交付多区域应用程序？
- 如何将应用程序或服务工件导入到其他区域？
- 数据库如何在不同位置之间进行复制？
- 应该使用哪些后备服务：Block Storage、File Storage、Object Storage 或 Databases？
- 有任何特定于服务的考虑因素吗？

## 目标
{: #objectives}

* 了解构建弹性应用程序时涉及的体系结构概念
* 了解此类概念如何映射到 IBM Cloud 计算和服务产品

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构和概念

{: #architecture}

要设计弹性体系结构，您需要考虑解决方案的各个构成块及其特定功能。 

下面是一个多区域体系结构，展示了多区域设置中可能存在的不同组件。![体系结构](images/solution39/Architecture.png)

上面的体系结构图可能根据计算选项而有所不同。您将在后面的各部分中看到每个计算选项下的特定体系结构图。 

### 具有两个区域的灾难恢复 

为了方便进行灾难恢复，使用了两种广泛接受的体系结构：**主动/主动**和**主动/被动**。每种体系结构都有自己的成本，并在恢复期间的时间和工作量方面各有优点。

#### 主动/主动配置

在主动/主动体系结构中，两个位置都具有完全相同的主动实例，通过负载均衡器在两者之间分发流量。使用此方法时，必须实施数据复制，以实时同步两个区域之间的数据。

![主动/主动](images/solution39/Active-active.png)

与主动/被动体系结构相比，此配置提供了更高的可用性，需要的手动修复工作更少。请求通过两个数据中心进行处理。应该为边缘服务（负载均衡器）配置相应的超时和重试逻辑，以便在第一个数据中心环境中发生故障时，自动将请求路由到第二个数据中心。

在主动/主动方案中考虑**恢复点目标** (RPO) 时，两个主动数据中心之间的数据同步必须非常及时，以允许无缝请求流。

#### 主动/被动配置

主动/被动体系结构依赖于一个主动区域和用作备份的另一个（被动）区域。如果主动区域发生中断，被动区域会变为主动区域。可能需要手动干预，以确保数据库或文件存储器与应用程序和用户需求保持同步。 

![主动/主动](images/solution39/Active-passive.png)

请求通过主动站点进行处理。如果发生中断或应用程序故障，将执行应用程序准备工作，以使备用数据中心准备好处理请求。从主动数据中心切换到被动数据中心是一项很耗时的操作。相比主动/主动配置，**恢复时间目标** (RTO) 和**恢复点目标** (RPO) 都要更长。

### 具有三个区域的灾难恢复

在如今对停机时间零容忍、服务“永远可用”的时代，客户希望每个业务服务在世界范围内随时随地保持可访问。对企业而言，有成本效益的策略涉及构造实现持续可用性的基础架构，而不是构建灾难恢复基础架构。

使用三个数据中心提供的弹性和可用性要高于两个数据中心。此外，还可以通过在各数据中心之间更均匀地分布负载，从而提供更佳性能。如果企业只有两个数据中心，那么这种部署的变体是在一个数据中心部署两个应用程序，在另一个数据中心部署第三个应用程序。或者，可以采用三主动拓扑形式部署业务逻辑和表示层，以双主动拓扑形式部署数据层。

#### 主动/主动/主动（三主动）配置

![](images/solution39/Active-active-active.png)

请求由在三个主动数据中心中的任何一个中心运行的应用程序进行处理。IBM.com Web 站点上的一个案例研究表明，三主动只需要占用每个集群 50% 的计算、内存和网络容量，但双主动需要占用每个集群 100% 的相应资源。数据层的成本差异非常明显。有关进一步的详细信息，请阅读 [*AlwaysOn: Assess, Design, Implement, and Manage Continuous Availability*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf)。

#### 主动/主动/被动配置

![](images/solution39/Active-active-passive.png)

在此场景中，主数据中心和辅助数据中心内的两个主动应用程序之一发生中断时，第三个数据中心的备用应用程序会激活。将遵循双数据中心场景中描述的灾难恢复过程，以复原为正常处理客户请求。第三个数据中心的备用应用程序可以设置为热备用或冷备用配置。

有关灾难恢复的更多信息，请参阅[此指南](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/)。

### 多区域体系结构

在多区域体系结构中，应用程序会部署到不同的位置，在这些位置中，每个区域运行完全相同的应用程序副本。 

区域是可以部署应用程序、服务和其他 {{site.data.keyword.cloud_notm}} 资源的特定地理位置。[{{site.data.keyword.cloud_notm}} 区域](https://{DomainName}/docs/containers?topic=containers-regions-and-zones)由一个或多个专区组成，专区是物理数据中心，用于托管计算、网络和存储资源以及相关冷却系统和电源，服务和应用程序通过这些资源托管。专区彼此隔离，可确保没有共享的单点故障。


此外，在多区域体系结构中，需要全局负载均衡器（如 [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)），以便在区域之间分发流量。

跨多个区域部署解决方案可带来以下优点：
- 缩短最终用户的等待时间 - 速度是关键，后端源离最终用户越近，用户体验越好，速度越快。
- 灾难恢复 - 主动区域发生故障时，可以使用备份区域快速恢复。
- 业务需求 - 在某些情况下，需要将数据存储在相隔数百公里的不同区域中。因此，在这种情况下，您必须将数据存储在多个区域中。 

### 区域体系结构中的多专区

构建多专区区域应用程序意味着将应用程序部署在一个区域的不同专区中，随后您还可能具有两个或三个区域。 

对于多专区区域体系结构，需要有本地负载均衡器在一个区域的不同专区之间本地分发流量，如果随后设置了第二个区域，那么全局负载均衡器会在这些区域之间分发流量。 

可以在[此处](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones)了解有关区域和专区的更多信息。

## 计算选项 

此部分再次探讨 {{site.data.keyword.cloud_notm}} 中可用的计算选项。对于每个计算选项，提供了相应的体系结构图以及有关如何部署此类体系结构的教程。

注：所有计算选项的体系结构都没有包含数据库或其他服务，只专注于将应用程序部署到所选计算选项的两个区域。但部署了任何多区域计算选项示例后，下一个逻辑步骤就是添加数据库和其他服务。本解决方案教程的后面部分将阐述[数据库](#databaseservices)和[非数据库服务](#nondatabaseservices)。

### Cloud Foundry 

Cloud Foundry 提供了实现多区域体系结构部署的功能，而使用[持续交付](https://{DomainName}/catalog/services/continuous-delivery)管道服务也允许您在多个区域中部署应用程序。Cloud Foundry 多区域的体系结构类似于下图：

![CF 体系结构](images/solution39/CF2-Architecture.png)

相同的应用程序部署在多个区域中，全局负载均衡器将流量路由到距离最近且正常运行的区域。[**跨多个区域保护 Web 应用程序**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)教程将指导您完成类似体系结构的部署。

### {{site.data.keyword.cfee_full_notm}}

**{{site.data.keyword.cfee_full_notm}} (CFEE)** 提供与公共 Cloud Foundry 相同的所有功能，以及其他功能。

通过 **{{site.data.keyword.cfee_full_notm}}**，您可以按需实例化多个隔离的企业级 Cloud Foundry 平台。CFEE 的实例在 [{{site.data.keyword.cloud_notm}}](http://ibm.com/cloud) 中您自己的帐户中运行。环境部署在基于 [{{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service) 的隔离硬件上。您拥有对环境的完全控制权，包括访问控制、容量管理、变更管理、监视和服务。

下面是使用 {{site.data.keyword.cfee_full_notm}} 的多区域体系结构。

![体系结构](images/solution39/CFEE-Architecture.png)

部署此体系结构需要以下操作： 

- 设置两个 CFEE 实例 - 每个区域一个实例。
- 创建服务并将其绑定到 CFEE 帐户。 
- 推送以 CFEE API 端点为目标的应用程序。 
- 设置数据库复制，就像在公共 Cloud Foundry 上执行的操作一样。 

此外，请查看逐步指南：[Deploy Logistics Wizard to Cloud Foundry Enterprise Environment (CFEE)](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md)。此指南将引导您逐步将基于微服务的应用程序部署到 CFEE。部署到一个 CFEE 实例后，可以将该过程复制到第二个区域，并在两个 CFEE 实例的前端连接 [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)，以对流量进行负载均衡。 

有关其他详细信息，请参阅 [{{site.data.keyword.cfee_full_notm}} 文档](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about)。

### Kubernetes

通过 Kubernetes，可以在区域体系结构中实现多专区，这可以是主动/主动用例。使用 {{site.data.keyword.containershort_notm}} 实施解决方案时，可以利用各种内置功能（如负载均衡和隔离），提高应对主机、网络或应用程序潜在故障的弹性。通过创建多个集群，如果一个集群发生中断，用户仍然可以访问同时部署在另一个集群中的应用程序。利用位于不同区域的多个集群，用户还可以访问距离最近的集群，缩短网络等待时间。为了获得更多弹性，还可以选择多专区集群，这意味着节点部署在一个区域的多个专区中。 

Kubernetes 多区域体系结构类似于下图。

![Kubernetes](images/solution39/Kub-Architecture.png)

1. 开发者为应用程序构建 Docker 映像。
2. 将映像推送到两个不同位置的 {{site.data.keyword.registryshort_notm}}。
3. 将应用程序部署到这两个位置的 Kubernetes 集群。
4. 最终用户访问该应用程序。
5. Cloud Internet Services 配置为拦截对应用程序发出的请求并在集群之间分发负载。此外，还启用了 DDoS 保护和 Web 应用程序防火墙，以保护应用程序免受常见威胁。（可选）对图像、CSS 文件等资产进行高速缓存。

[**使用 Cloud Internet Services 的富有弹性且安全的多区域 Kubernetes 集群**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)教程可全程指导您完成部署此类体系结构的步骤。

### {{site.data.keyword.openwhisk_short}}

{{site.data.keyword.openwhisk_short}} 在多个 {{site.data.keyword.cloud_notm}} 位置可用。为了提高弹性并缩短网络等待时间，应用程序可以将其后端部署在多个位置。然后，通过 IBM Cloud Internet Services (CIS)，开发者可以公开单个入口点，用于负责将流量分发给距离最近且正常运行的后端。{{site.data.keyword.openwhisk_short}} 多区域的体系结构类似于下图。

 ![Functions 体系结构](images/solution39/Functions-Architecture.png)

1. 用户访问该应用程序。请求通过 Internet Services。
2. Internet Services 将用户重定向到距离最近且正常运行的 API 后端。
3. Certificate Manager 为 API 提供其 SSL 证书。流量是端到端加密的。
4. API 通过 Cloud Functions 实现。

通过遵循[**跨多个区域部署无服务器应用程序**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless)教程，了解如何部署此体系结构。

### {{site.data.keyword.baremetal_short}} 和 {{site.data.keyword.virtualmachinesshort}}

{{site.data.keyword.virtualmachinesshort}} 和 {{site.data.keyword.baremetal_short}} 提供了实现多区域体系结构的功能。可以在 {{site.data.keyword.cloud_notm}} 上的多个可用位置供应服务器。

![服务器位置](images/solution39/ServersLocation.png)

在使用 {{site.data.keyword.virtualmachinesshort}} 和 {{site.data.keyword.baremetal_short}} 准备此类体系结构时，请考虑以下事项：文件存储器、备份、恢复和数据库，在数据库即服务与在虚拟服务器上安装数据库这两个选项中做出选择。 

以下体系结构演示了如何在主动/被动体系结构（其中，一个区域为主动，另一个区域为被动）中使用 {{site.data.keyword.virtualmachinesshort}} 部署多区域体系结构。 

![VM 体系结构](images/solution39/vm-Architecture2.png)

此类体系结构需要的组件： 

1. 用户通过 IBM Cloud Internet Services (CIS) 访问应用程序。
2. CIS 将流量路由到主动位置。
3. 在一个位置中，负载均衡器将流量重定向到服务器。
4. 部署在虚拟服务器上的数据库，这意味着将配置数据库，并设置区域之间的复制和备份。替代方法是使用数据库即服务，此主题将在本教程后面进行讨论。
5. 用于存储应用程序映像和文件的文件存储器，文件存储器提供了在给定时间和日期创建快照的功能，然后此快照可以在另一个区域中复用，这需要手动执行。 

[**使用虚拟服务器构建具备高可用性和高可缩放性的 Web 应用程序**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application)教程实施此体系结构。

## 数据库和应用程序文件
{: #databaseservices}

{{site.data.keyword.cloud_notm}} 提供了一组精选的[数据库即服务](https://{DomainName}/catalog/?category=databases)，包括关系数据库和非关系数据库，您可根据自己的业务需求选择使用。[数据库即服务 (DBaaS)](https://www.ibm.com/cloud/learn/what-is-cloud-database) 可带来许多优点。借助 DBaaS（如 {{site.data.keyword.cloudant}}），可以利用多区域支持，以允许您在不同区域的两个数据库服务之间进行实时复制，执行备份，使用缩放，并且正常运行时间最长。 

**主要功能：** 

- 提供通过云平台构建和访问的数据库服务
- 支持企业用户托管数据库，无需购买专用硬件
- 可以由用户管理或由供应商以即服务方式提供并进行管理
- 可以支持 SQL 或 NoSQL 数据库
- 可以通过 Web 界面或供应商提供的 API 进行访问

**准备多区域体系结构：**

- 数据库服务有哪些弹性选项？
- 如何处理不同区域中多个数据库服务之间的复制？
- 如何备份数据？
- 针对每种情况有哪些灾难恢复方法？

### {{site.data.keyword.cloudant}}

{{site.data.keyword.cloudant}} 是一种分布式数据库，经过优化可以处理快速增长的大型 Web 和移动应用程序所典型具备的密集工作负载。{{site.data.keyword.cloudant}} 作为支持 SLA 的完全受管的 {{site.data.keyword.Bluemix_notm}} 服务提供，可以独立地弹性缩放吞吐量和存储量。{{site.data.keyword.cloudant}} 还作为可下载的本地安装提供，其 API 和强大的复制协议与开放式源代码生态系统兼容，该生态系统包括 CouchDB、PouchDB 以及针对最常用 Web 和移动开发堆栈的库。

{{site.data.keyword.cloudant}} 支持在不同位置的多个实例之间进行[复制](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation)。源数据库中发生的任何更改都将在目标数据库中重现。可以在任意数量的数据库之间创建复制，复制方式可以是连续的，也可以是“一次性”任务。下图显示了使用两个 {{site.data.keyword.cloudant}} 实例的典型配置，每个区域一个实例：

![主动/主动](images/solution39/Active-active.png)

请参阅[这些指示信息](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery)来配置 {{site.data.keyword.cloudant}} 实例之间的复制。该服务还提供了用于[备份和复原数据](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery)的指示信息和工具。

### {{site.data.keyword.Db2_on_Cloud_short}}、{{site.data.keyword.dashdbshort_notm}} 和 {{site.data.keyword.Db2Hosted_notm}}

{{site.data.keyword.cloud_notm}} 提供了多个 [Db2 数据库服务](https://{DomainName}/catalog/?search=db2h)。包括：

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2)：完全受管的云 SQL 数据库，适用于典型的类似 OLTP 的操作工作负载。
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse)：完全受管的云数据仓库服务，适用于高性能、拍字节级分析工作负载。此服务提供 SMP 和 MPP 服务套餐，并利用经过优化的柱状数据存储和内存内处理。
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted)：由 IBM 托管并由用户数据库系统进行管理。此服务为 Db2 提供了对云基础架构的完全管理访问权，因此消除了与管理您自己的基础架构相关的成本、复杂性和风险。

在下文中，我们将重点关注用于操作工作负载的作为 DBaaS 的 {{site.data.keyword.Db2_on_Cloud_short}}。这些工作负载是本教程中讨论的应用程序的典型工作负载。

#### {{site.data.keyword.Db2_on_Cloud_short}} 的多区域支持

{{site.data.keyword.Db2_on_Cloud_short}} 提供了多个[用于实现高可用性和灾难恢复 (HADR) 的选项](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview)。可以在创建新服务时选择“高可用性”选项。也可以日后通过实例仪表板[添加地理复制的灾难恢复节点](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha)。借助非现场 DR 节点选项，可以将数据实时同步到所选的非现场 {{site.data.keyword.cloud_notm}} 数据中心的数据库节点。

更多信息可在[高可用性文档](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha)中获取。

#### 备份和复原

{{site.data.keyword.Db2_on_Cloud_short}} 包括针对付费套餐的每日备份。通常，备份会使用 {{site.data.keyword.cos_short}} 进行存储，从而利用三个数据中心来提高保留数据的可用性。备份会保留 14 天。可以使用备份来执行时间点恢复。[备份和复原文档](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br)提供了有关如何将数据复原到所需日期和时间的详细信息。

### {{site.data.keyword.databases-for}}

{{site.data.keyword.databases-for}} 提供了多个开放式源代码数据库系统作为完全受管服务。包括：  
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

所有这些服务都具有相同的特征：   
* 为了实现高可用性，这些服务会部署在集群中。详细信息可在各个服务的文档中找到：
  - [PostgreSQL](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* 每个集群分布在多个专区中。
* 数据在各专区之间进行复制。
* 用户可以扩展实例的存储器和内存资源。有关详细信息，请参阅[有关缩放的文档，例如 {{site.data.keyword.databases-for-redis}}](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings)。
* 备份每天或按需执行。将记录各个服务的详细信息。此处是 [{{site.data.keyword.databases-for-postgresql}} 等的备份文档](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups)。
* 静态数据、备份和网络流量都会加密。
* 各个[服务可以使用 {{site.data.keyword.databases-for}} CLI 插件来管理](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in)


### {{site.data.keyword.cos_full_notm}}

{{site.data.keyword.cos_full_notm}} (COS) 提供耐久、安全且有成本效益的云存储器。存储在 {{site.data.keyword.cos_full_notm}} 中的信息已加密并分布在多个地理位置。在 COS 实例中创建存储区时，可以决定应在哪个位置创建存储区以及使用哪个弹性选项。

有三种类型的存储区弹性：
   - **跨区域**弹性将在多个大城市区域中分布数据。可以将此项视为多区域选项。访问存储在“跨区域”存储区中的内容时，COS 提供了一个特殊端点，能够从正常运行的区域中检索内容。
   - **区域**弹性将在单个大城市区域中分布数据。可以将此项视为一个区域中多专区的配置。
   - **单个数据中心**弹性将在单个数据中心内的多个设备中分布数据。

有关 {{site.data.keyword.cos_full_notm}} 弹性选项的详细说明，请参阅[此文档](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints)。

### {{site.data.keyword.filestorage_full_notm}}

{{site.data.keyword.filestorage_full_notm}} 是一种网络连接的基于 NFS 的文件存储器，具有持久、快速、灵活的特点。在此网络连接存储器 (NAS) 环境中，您对文件共享功能和性能具有完全控制权。{{site.data.keyword.filestorage_short}} 共享可通过路由 TCP/IP 连接来连接到最多 64 个已授权设备，从而实现弹性。

一些文件存储器功能包括_快照_、_复制_和_并行访问_。有关完整的功能列表，请参阅[文档](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage)。

连接到服务器后，可以轻松地使用 {{site.data.keyword.filestorage_short}} 服务来存储数据备份以及图像和视频等应用程序文件，然后可以在同一区域的不同服务器中使用这些图像和文件。

添加第二个区域时，可以使用 {{site.data.keyword.filestorage_short}} 的快照功能自动或手动创建快照，然后在第二个被动区域内复用此快照。 

可以安排复制以自动将快照复制到远程数据中心的目标卷。如果发生灾难性事件，或者数据变得损坏，那么可以在远程站点中恢复副本。有关 File Storage 快照的更多信息，可以在[此处](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots)找到。

## 非数据库服务
{: #nondatabaseservices}

{{site.data.keyword.cloud_notm}} 提供了一组精选的非数据库[服务](https://{DomainName}/catalog)，包括 IBM 服务和第三方服务。在规划多区域体系结构时，需要了解各种服务（如 Watson 服务）如何在多区域设置中工作。

### {{site.data.keyword.conversationfull}}

[{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about) 是一个平台，允许开发人员和非技术用户进行协作，共同构建基于 AI 技术的会话式助手。

助手是一种认知机器人，您可以根据自己的业务需要进行定制，并跨多个渠道进行部署，以随时随地根据客户需要向客户提供帮助。助手包含一项或多项技能。对话技能包含训练数据和逻辑，用于支持助手为客户提供帮助。

值得注意的是，{{site.data.keyword.conversationshort}} V1 是无状态的。通过 {{site.data.keyword.conversationshort}}，可实现 99.5% 的正常运行时间，但对于跨多个区域的高可用性应用程序，您可能还是希望在不同区域中拥有此服务的多个实例。 

在多个位置创建实例后，可使用工具 {{site.data.keyword.conversationshort}} 从一个实例中导出现有工作空间，包括意向、实体和对话。然后在其他位置中导入此工作空间。

## 摘要

|产品|弹性选项|
| -------- | ------------------ |
|Cloud Foundry| <ul><li>将应用程序部署到多个位置</li><li>使用 Cloud Internet Services 处理来自多个位置的请求</li><li>使用 Cloud Foundry API 来配置组织和空间，并将应用程序推送到多个位置</li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>将应用程序部署到多个位置</li><li>使用 Cloud Internet Services 处理来自多个位置的请求</li><li>使用 Cloud Foundry API 来配置组织和空间，并将应用程序推送到多个位置</li><li>基于 Kubernetes Service 构建</li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>精心设计的弹性，支持多专区集群</li><li>使用 Cloud Internet Services 处理来自分布在多个位置的集群的请求</li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>在多个位置中可用</li><li>使用 Cloud Internet Services 处理来自多个位置的请求</li><li>使用 Cloud Functions API 在多个位置部署操作</li></ul> |
| {{site.data.keyword.baremetal_short}} 和 {{site.data.keyword.virtualmachinesshort}} | <ul><li>在多个位置中供应服务器</li><li>将同一位置的服务器连接到本地负载均衡器</li><li>使用 Cloud Internet Services 处理来自多个位置的请求</li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>在数据库之间进行一次性复制和持续复制</li><li>区域内自动数据冗余</li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>供应地理复制的灾难恢复节点，以实现实时数据同步</li><li>每日备份（付费套餐中提供）</li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>基于多专区 Kubernetes 集群构建</li><li>跨区域读取副本</li><li>每日备份和按需备份</li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>单个数据中心、区域和跨区域弹性</li><li>使用 API 在各存储区之间同步内容</li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>使用快照自动将内容捕获到远程数据中心内的目标</li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>使用 Watson API 在不同位置的多个实例之间导出和导入工作空间规范</li></ul> |

## 相关内容

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry - 跨多个区域保护 Web 应用程序](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [CloudFunctions - 跨多个区域部署无服务器应用程序](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes - 使用 Cloud Internet Services 的富有弹性且安全的多区域 Kubernetes 集群](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [虚拟服务器 - 构建具备高可用性和高可缩放性的 Web 应用程序](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
