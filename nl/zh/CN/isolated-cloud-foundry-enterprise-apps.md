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

# 隔离的 Cloud Foundry 企业应用程序
{: #isolated-cloud-foundry-enterprise-apps}

通过 {{site.data.keyword.cfee_full_notm}} (CFEE)，您可以按需创建多个隔离的企业级 Cloud Foundry 平台。借助这些平台，可在隔离的 Kubernetes 集群上部署专用 Cloud Foundry 实例。与公共云不同的是，您将拥有对环境的完全控制权：访问控制、容量、版本、资源使用和监视。{{site.data.keyword.cfee_full_notm}} 通过在企业 IT 中找到的基础架构所有权来提供平台即服务级别的速度和创新。

企业拥有的创新平台是 {{site.data.keyword.cfee_full_notm}} 的一个用例。您作为企业中的开发者，可以创建新的微服务，也可以将旧应用程序迁移到 CFEE。然后，可以使用 Cloud Foundry 市场向其他开发者发布微服务。发布后，CFEE 实例中的其他开发者可以在应用程序中使用服务，就像如今在公共云上所做的一样。

本教程将全程指导您完成创建和配置 {{site.data.keyword.cfee_full_notm}}，设置访问控制以及部署应用程序和服务的过程。您还将通过部署定制服务代理程序来集成定制服务与 CFEE，以查看 CFEE 和 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) 之间的关系。

## 目标

{: #objectives}

- 比较和对比 CFEE 与公共 Cloud Foundry
- 在 CFEE 中部署应用程序和服务
- 了解 Cloud Foundry 与 {{site.data.keyword.containershort_notm}} 的关系
- 调查基本 Cloud Foundry 和 {{site.data.keyword.containershort_notm}} 联网情况

## 使用的服务

{: #services}

本教程使用以下运行时和服务：

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构

{: #architecture}

![体系结构](./images/solution45-CFEE-apps/Architecture.png)

1. 管理员创建 CFEE 实例，并添加具有开发者访问权的用户。
2. 开发者将 Node.js 入门模板应用程序推送到 CFEE。
3. Node.js 入门模板应用程序使用 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 来存储数据。
4. 开发者添加新的“欢迎消息”服务。
5. Node.js 入门模板应用程序通过定制服务代理程序绑定新服务。
6. Node.js 入门模板应用程序通过服务以不同语言显示“欢迎”。

## 先决条件

{: #prereq}

- [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [Cloud Foundry CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## 供应 {{site.data.keyword.cfee_full_notm}}

{:provision_cfee}

在此部分中，您将创建 {{site.data.keyword.cfee_full_notm}} 实例，并通过 {{site.data.keyword.containershort_notm}} 部署到 Kubernetes 工作程序节点。

1. [准备 {{site.data.keyword.cloud_notm}} 帐户](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare)，以确保创建必需的基础架构资源。
2. 在 {{site.data.keyword.cloud_notm}}“目录”中，创建 [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create) 的服务实例。
3. 通过提供以下内容来配置 CFEE：
   - 选择**标准**套餐。
   - 输入服务实例的**名称**。
   - 选择在其中创建环境的**资源组**。您需要可访问帐户中至少一个资源组的许可权，才能创建 CFEE 实例。
   - 选择在其中部署实例的**地理位置**和**位置**。请参阅[可用供应位置和数据中心](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets)的列表。
   - 选择将在其中部署 **{{site.data.keyword.composeForPostgreSQL}}** 的公共 Cloud Foundry **组织**和**空间**。
   - 选择用于 Cloud Foundry 环境的**单元数**。一个单元可运行多个 Diego 和 Cloud Foundry 应用程序。对于高可用性应用程序，至少需要 **2** 个单元。
   - 选择**机器类型**，这将确定 Cloud Foundry 单元的大小（CPU 和内存）。
4. 复查**基础架构**部分，以查看支持 CFEE 的 Kubernetes 集群的属性。**工作程序节点数**等于单元数加 2。其中供应的两个 Kubernetes 工作程序节点充当 CFEE 控制平面。部署了环境的 Kubernetes 集群将显示在 {{site.data.keyword.cloud_notm}} [集群](https://{DomainName}/containers-kubernetes/clusters)仪表板中。
5. 单击**创建**按钮以开始自动部署。

自动部署需要大约 90 到 120 分钟。成功创建后，您将收到一封电子邮件，用于确认供应 CFEE 和支持服务。

### 创建组织和空间

创建了 {{site.data.keyword.cfee_full_notm}} 后，请参阅[创建组织和空间](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs)，以获取有关如何通过组织和空间构造环境的信息。{{site.data.keyword.cfee_full_notm}} 中的应用程序的作用域限定在特定空间内。与此类似，空间存在于组织中。组织的成员共享配额套餐、应用程序、服务实例和定制域。

注：位于 {{site.data.keyword.cloud_notm}} 顶部标题中的**管理 > 帐户 > Cloud Foundry 组织**菜单专门用于公共 {{site.data.keyword.cloud_notm}} 组织。CFEE 组织在 CFEE 实例的**组织**页面中进行管理。

执行以下步骤创建 CFEE 组织和空间。

1. 在 [Cloud Foundry 仪表板](https://{DomainName}/dashboard/cloudfoundry/overview)中的**企业**下，选择**环境**。
2. 选择您的 CFEE 实例，然后选择**组织**。
3. 单击**创建组织**按钮，提供 `tutorial` 作为**组织名称**，然后选择**配额套餐**。通过单击**添加**以完成操作。
4. 单击新创建的 `tutorial` 组织，选择**空间**选项卡，然后单击**新建空间**按钮。
5. 提供 `dev` 作为**空间名称**，然后单击**添加**。

### 向组织和空间添加用户

在 CFEE 中，可以分配用于控制用户访问权的角色，但要执行此操作，必须在 {{site.data.keyword.cloud_notm}} 标题中**管理 > 用户**下的**身份和访问权**页面中，邀请用户加入您的 {{site.data.keyword.cloud_notm}} 帐户。

邀请用户后，执行以下步骤将用户添加到创建的 `tutorial` 组织。 

1. 选择您的 [CFEE 实例](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments)，然后再次选择**组织**。
2. 从列表中选择创建的 `tutorial` 组织。
3. 单击**成员**选项卡来查看用户和添加新用户。
4. 单击**添加成员**按钮，搜索用户名，选择相应的**组织角色**，然后单击**添加**。

有关向 CFEE 组织和空间添加用户的更多信息，可在[此处](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users)找到。

## 部署、配置和运行 CFEE 应用程序

{:deploycfeeapps}

在此部分中，您要将 Node.js 应用程序部署到 CFEE。部署后，将 {{site.data.keyword.cloudant_short_notm}} 绑定到该应用程序，然后启用审计和日志记录持久性。此外，还将添加 Stratos Console 来管理应用程序。

### 将应用程序部署到 CFEE

1. 在终端中，克隆 [get-started-node](https://github.com/IBM-Cloud/get-started-node) 样本应用程序。
   ```sh
   git clone https://github.com/IBM-Cloud/get-started-node
   ```
   {: codeblock}
2. 在本地运行该应用程序，以确保它正确构建并启动。请通过在浏览器中访问 `http://localhost:3000/` 进行确认。
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. 登录到 {{site.data.keyword.cloud_notm}}，并将您的 CFEE 实例设定为目标。交互式提示将帮助您选择新的 CFEE 实例。由于只存在一个 CFEE 组织和一个空间，因此会将其作为缺省目标。如果添加了多个组织或空间，那么可以运行 `ibmcloud target -o tutorial -s dev`。
   ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
   {: codeblock}
4. 将 **get-started-node** 应用程序推送到 CFEE。
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. 应用程序的端点将显示在最终输出中的 `routes` 属性旁边。在浏览器中打开 URL 以确认应用程序是否正在运行。

### 创建 Cloudant 数据库并将其绑定到应用程序

要将 {{site.data.keyword.cloud_notm}} 服务绑定到 **get-started-node** 应用程序，首先需要在 {{site.data.keyword.cloud_notm}} 帐户中创建服务。

1. 创建 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 服务。提供**服务名称** `cfee-cloudant`，然后选择在其中创建了 CFEE 实例的位置。
2. 将新创建的 {{site.data.keyword.cloudant_short_notm}} 服务实例添加到 CFEE。
   1. 导航回 `tutorial` **组织**。单击**空间**选项卡，然后选择 `dev` 空间。
   2. 选择**服务**选项卡，然后单击**添加服务**按钮。
   3. 在搜索文本框中输入 `cfee-cloudant`，然后选择结果。通过单击**添加**以完成操作。现在，服务可用于 CFEE 应用程序；但是，该服务仍位于公共 {{site.data.keyword.cloud_notm}} 中。
3. 在显示的服务实例的溢出菜单上，选择**绑定到应用程序**。
4. 选择早先推送的 **GetStartedNode** 应用程序，然后单击**绑定后重新编译打包应用程序**。最后，单击**绑定**按钮。等待应用程序重新编译打包。可以使用 `ibmcloud app show GetStartedNode` 命令来检查进度。
5. 在浏览器中，访问应用程序，添加您的姓名，然后点击 `Enter` 键。您的姓名将添加到 {{site.data.keyword.cloudant_short_notm}} 数据库。
6. 通过从**服务**选项卡上的列表中选择 `tutorial` 实例进行确认。这将在公共 {{site.data.keyword.cloud_notm}} 中打开服务实例的详细信息页面。
7. 单击**启动 Cloudant 仪表板**，然后选择 `mydb` 数据库。应该存在带有您姓名的 JSON 文档。

### 启用审计和日志记录持久性

通过审计，CFEE 管理员可以跟踪 Cloud Foundry 活动，例如登录、组织和空间创建、用户成员资格和角色分配、应用程序部署、服务绑定和域配置。审计通过与 {{site.data.keyword.cloudaccesstrailshort}} 服务集成进行支持。

可以通过集成 {{site.data.keyword.loganalysisshort_notm}} 来存储 Cloud Foundry 应用程序日志。由 CFEE 管理员选择的 {{site.data.keyword.loganalysisshort_notm}} 服务实例会自动配置为接收并持久存储从 CFEE 实例生成的 Cloud Foundry 日志记录事件。

要启用 CFEE 审计和日志记录持久性，请执行[此处的步骤](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging)。

### 安装 Stratos Console 来管理应用程序

Stratos Console 是一个基于 Web 的开放式源代码工具，用于对 Cloud Foundry 进行操作。可以选择在特定的 CFEE 环境中安装和使用 Stratos Console 应用程序，以管理组织、空间和应用程序。

要安装 Stratos Console 应用程序，请执行以下操作：

1. 打开要安装 Stratos Console 的 CFEE 实例。
2. 在**概述**页面上，单击**安装 Stratos Console**。此按钮仅对具有管理员或编辑者许可权的用户显示。
3. 在“安装 Stratos Console”对话框中，选择安装选项。您可以在 CFEE 控制平面上或其中一个单元中安装 Stratos Console 应用程序。选择 Stratos Console 的版本以及要安装的应用程序实例数。如果是在单元中安装 Stratos Console 应用程序，那么系统将提示您提供部署应用程序的组织和空间。
4. 单击**安装**。

安装该应用程序需要大约 5 分钟时间。安装完成后，会显示 **Stratos Console** 按钮，以取代“概述”页面上的**安装 Stratos Console** 按钮。有关 Stratos Console 的更多信息，可以在[此处](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app)找到。

## CFEE 与 Kubernetes 之间的关系

CFEE 作为一个应用程序平台，可以在某种形式的专用或共享虚拟基础架构上运行。多年来，开发者很少会考虑底层 Cloud Foundry 平台，因为是 IBM 在管理此平台。通过 CFEE，您不仅是编写 Cloud Foundry 应用程序的开发者，也是 Cloud Foundry 平台的运营者。这是因为 CFEE 部署在您所控制的 Kubernetes 集群上。

虽然 Cloud Foundry 开发者可能对 Kubernete 并不熟悉，但这两个产品共有的概念很多。与 Cloud Foundry 一样，Kubernetes 也是将应用程序隔离到容器中，这些容器在称为 pod 的 Kubernetes 构造内运行。与应用程序实例类似，pod 也可以具有多个副本（称为副本集），使用由 Kubernetes 提供的应用程序负载均衡。您早先部署的 Cloud Foundry `GetStartedNode` 应用程序在 `diego-cell-0` pod 内运行。为了支持高可用性，另一个 pod `diego-cell-1` 将在单独的 Kubernetes 工作程序节点上运行。由于这些 Cloud Foundry 应用程序是在 Kubernetes“内部”运行，因此还可以使用基于 Kubernetes 的联网与其他 Kubernetes 微服务进行通信。以下各部分将有助于更详细地说明 CFEE 与 Kubernetes 之间的关系。

## 部署 Kubernetes 服务代理程序

在此部分中，您要将微服务部署到 Kubernetes 以充当 CFEE 的服务代理程序。[服务代理程序](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md)提供有关可用服务的详细信息，以及有关向 Cloud Foundry 应用程序绑定和供应支持的详细信息。您已使用内置 {{site.data.keyword.cloud_notm}} 服务代理程序添加了 {{site.data.keyword.cloudant_short_notm}} 服务。现在，将部署并使用定制服务代理程序。然后，将修改 `GetStartedNode` 应用程序以使用该服务代理程序，这将返回多种语言的“欢迎”消息。

1. 返回到终端，克隆用于提供 Kubernetes 部署文件和服务代理程序实现的项目。

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

2. 在 {{site.data.keyword.registryshort_notm}} 上构建并存储包含服务代理程序的 Docker 映像。使用 `ibmcloud cr info` 命令手动检索注册表 URL，或使用下面的 `export REGISTRY` 命令自动检索注册表 URL。`cr namespace-add` 命令将创建一个名称空间来存储 Docker 映像。

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   创建名为 `cfee-tutorial` 的名称空间。

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   编辑 `./cfee-service-broker-kubernetes/deployment.yml` 文件。更新 `image` 属性以反映出容器注册表 URL。
3. 将容器映像部署到 CFEE 的 Kubernetes 集群。CFEE 集群存在于 `default` 资源组中，如果尚未将其设定为目标，请执行此操作。使用您的集群名称，通过 `cluster-config` 命令导出 KUBECONFIG 变量。然后创建部署。
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
4. 验证 pod 的 STATUS 是否为 `Running`。Kubernetes 可能需要一些时间来拉取映像并启动容器。请注意，您有两个 pod，因为 `deployment.yml` 已请求 2 个 `replicas`。
   ```sh
  kubectl get pods
  ```
   {: codeblock}

## 验证服务代理程序是否已部署

既然已部署了服务代理程序，请确认它是否在正常运行。您将通过多种方式来确认：首先是使用 Kubernetes 仪表板，然后是通过 Cloud Foundry 应用程序访问代理程序，最后是通过代理程序实际绑定服务。

### 在 Kubernetes 仪表板中查看 pod

此部分将使用 {{site.data.keyword.containershort_notm}} 仪表板来确认 Kubernetes 工件是否已配置。

1. 在 [Kubernetes 集群](https://{DomainName}/containers-kubernetes/clusters)页面中，通过单击以 CFEE 服务名称开头且以 **-cluster** 结尾的行项来访问 CFEE 集群。
2. 通过单击相应的按钮来打开 **Kubernetes 仪表板**。
3. 单击左侧菜单中的**服务**链接，然后选择 **tutorial-broker-service**。在早先步骤中运行 `kubectl apply` 时，已部署此服务。
4. 在生成的仪表板中，注意以下内容：
   - 服务已提供覆盖 IP 地址 (172.x.x.x)，该地址仅在 Kubernetes 集群中可解析。
   - 服务有两个端点，对应于运行服务代理程序容器的两个 pod。

在确认服务可用并正在代理服务代理程序 pod 之后，可以验证代理程序是否使用有关可用服务的信息进行响应。

可以在 Kubernetes 仪表板中查看 Cloud Foundry 相关工件。要查看这些工件，请单击**名称空间**以查看具有包含 `cf` Cloud Foundry 名称空间的工件的所有名称空间。
{: tip}

### 通过 Cloud Foundry 容器访问代理程序

为了演示 Cloud Foundry 到 Kubernetes 的通信，将直接从 Cloud Foundry 应用程序连接到服务代理程序。

1. 返回到终端，使用 `ibmcloud target` 确认是否仍连接到 CFEE `tutorial` 组织和 `dev` 空间。如果需要，请将 CFEE 重新设定为目标。
   ```sh
      ibmcloud target --cf
      ```
   {: codeblock}
2. 缺省情况下，空间中已禁用 SSH。这与公共云不同，因此请在 CFEE `dev` 空间中启用 SSH。
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. 使用 `kubectl` 命令显示在 Kubenetes 仪表板中看到的相同 ClusterIP。然后通过 SSH 登录到 `GetStartedNode` 应用程序，并使用该 IP 地址在服务代理程序中检索数据。请注意，最后一个命令可能会导致错误，此错误将在下一步中予以解决。
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
4. 您可能收到了 **Connection refused** 错误。导致此错误的原因是 CFEE 的缺省[应用程序安全组](https://docs.cloudfoundry.org/concepts/asg.html)。应用程序安全组 (ASG) 定义用于 Cloud Foundry 容器中流出流量的允许 IP 范围。由于 `GetStartedNode` 不属于缺省范围，因此会发生此错误。请退出 SSH 会话，然后下载 `public_networks` ASG。
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. 编辑 `public_networks.json` 文件，并验证正在使用的 ClusterIP 地址是否超出现有规则范围。例如，范围 `172.32.0.0-192.167.255.255` 可能不包含 ClusterIP，因此需要更新。例如，如果 ClusterIP 地址类似于 `172.21.107.63`，那么需要编辑该文件，使其具有类似于 `172.0.0.0-255.255.255.255` 的内容。
6. 调整 ASG `destination` 规则，以包含 Kubernetes Service 的 IP 地址。修剪该文件以仅包含 JSON 数据，此数据以括号开头和结尾。然后上传新的 ASG。
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. 重复步骤 3，现在应该可以成功模拟目录数据。通过退出 SSH 会话以完成操作。
   ```sh
   exit
   ```
   {: codeblock}

### 向 CFEE 注册服务代理程序

要允许开发者通过服务代理程序来供应和绑定服务，请向 CFEE 注册服务代理程序。先前，您已通过 IP 地址使用代理程序。但这会产生一个问题：服务代理程序重新启动时，会收到新的 IP 地址，而这需要更新 CFEE。要解决此问题，将使用另一个名为 KubeDNS 的 Kubernetes 功能，此功能为服务代理程序提供标准域名 (FQDN)。

1. 使用 `tutorial-service-broker` 服务的 FQDN 向 CFEE 注册服务代理程序。同样，这是 CFEE Kubernetes 集群的内部路径。
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. 然后，添加代理程序提供的服务。由于样本代理程序只有一个模拟服务，因此只需要一个命令。
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. 在浏览器中，从[**环境**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments)页面访问 CFEE 实例，然后导航至`组织 -> 空间`，并选择 `dev` 空间。
4. 选择**服务**选项卡，然后选择**创建服务**按钮。
5. 在搜索文本框中，搜索**测试**。这将显示代理程序中的**测试节点资源服务代理程序显示名称**模拟服务。
6. 选择该服务，单击**创建**按钮，然后提供名称 `welcome-service` 来创建服务实例。在下一部分中您会明白为什么将其命名为 welcome-service。然后使用溢出菜单中的**绑定到应用程序**项，将该服务绑定到 `GetStartedNode` 应用程序。
7. 要查看绑定的服务，请运行 `cf env` 命令。现在，`GetStartedNode` 应用程序可以像当前使用 `cloudantNoSQLDB` 中的数据一样，利用 `credentials` 对象中的数据。
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## 向应用程序添加服务

到目前为止，您部署的是服务代理程序，但并不是实际服务。虽然服务代理程序向 `GetStartedNode` 提供绑定数据，但未添加服务本身的实际功能。为了解决这个问题，创建了一个模拟服务，用于向 `GetStartedNode` 提供各种语言的“欢迎”消息。您要将此服务部署到 CFEE，并将代理程序和应用程序更新为使用此服务。

1. 将 `welcome-service` 实现推送到 CFEE，以允许其他开发团队将其用作 API。
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. 使用浏览器访问该服务。服务的 `route` 将显示在 `push` 命令的输出中。生成的 URL 将类似于 `https://welcome.<your-cfee-cluster-domain>`。刷新页面可查看交替显示的语言。
3. 返回到 `sample-resource-service-brokers` 文件夹并编辑 Node.js 样本实现 `sample-resource-service-brokers/node-resource-service-broker/testresourceservicebroker.js`，在第 854 行之后添加 url，其值为集群 URL。 

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
4. 构建并部署更新后的服务代理程序。这可确保将 URL 属性提供给绑定了该服务的应用程序。
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
5. 在终端中，导航回 `get-started-node` 应用程序文件夹。 
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. 编辑 `get-stared-node` 中的 `get-started-node/server.js` 文件，以包含以下中间件。将其插入到 `app.listen()` 调用的正前方。
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
7. 重构 `get-started-node/views/index.html` 页面。将 `h1` 标记中的 `data-i18n="welcome"` 更改为 `id="welcome"`。然后，在 `$(document).ready()` 函数末尾添加对该服务的调用。
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. 更新 `GetStartedNode` 应用程序。包含已添加到 `server.js` 的 `request` 软件包依赖项，重新绑定 `welcome-service` 以选取新的 `url` 属性，最后推送应用程序的新代码。
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

完成了！现在，访问该应用程序并多次刷新页面可查看不同语言的欢迎消息。

![服务代理程序响应](./images/solution45-CFEE-apps/service-broker.png)

如果仍然只看到 **Welcome**，而未显示其他语言，请运行 `ibmcloud cf env GetStartedNode` 命令，并确认该服务的 `url` 是否存在。如果不存在，请重新跟踪执行的步骤，进行更新，并重新部署代理程序。如果存在 `url` 值，请确认它是否在浏览器中返回消息。然后，重新运行 `unbind-service` 和 `bind-service` 命令，接着运行 `ibmcloud cf restart GetStartedNode`。
{: tip}

虽然欢迎服务使用 Cloud Foundry 作为其实现，但使用 Kubernetes 作为实现也很容易。主要区别在于服务的 URL 可能是 `welcome-service.default.svc.cluster.local`。使用 KubeDNS URL 还有一个额外的好处，即可以使网络流量保持流至 Kubernetes 集群内部的服务。

通过 {{site.data.keyword.cfee_full_notm}}，现在可以使用这些替代方法。

## 使用工具链部署此解决方案教程  

（可选）可以使用工具链来部署完整的解决方案教程。请按照[工具链指示信息](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)使用工具链来部署以上所有内容。 

注：使用工具链时有一些先决条件，您必须已创建 CFEE 实例，并已创建 CFEE 组织和 CFEE 空间。[工具链指示信息](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)自述文件中概述了详细的指示信息。 

## 相关内容
{:related}

* [在 Kubernetes 集群中部署应用程序](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Cloud Foundry Diego Components and Architecture](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [CFEE Service Broker on Kubernetes with a Toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

