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

# 将端到端安全性应用于云应用程序
{: #cloud-e2e-security}

要构建完备的应用程序体系结构，必须对潜在的安全风险以及如何防范此类威胁有清楚的了解。应用程序数据是一种不能丢失、泄露或被盗的关键资源。此外，静态和动态数据应通过加密技术进行保护。加密静态数据可以防止信息泄露，就算信息丢失或被盗也不会外泄。通过 HTTPS、SSL 和 TLS 等方法加密动态数据（例如，在因特网上传输的数据）可防止窃听和所谓的中间人攻击。

在许多应用程序中，另一个常见要求是针对用户对特定资源的访问权进行认证和授权。这可能需要支持不同的认证方案：使用社交身份的客户和供应商、来自云托管目录的合作伙伴以及来自组织的身份提供者的员工。

本教程将全程指导您使用 {{site.data.keyword.cloud}}“目录”中提供的关键安全服务，并说明如何将这些服务一起使用。提供文件共享的应用程序将落实安全性概念。
{:shortdesc}

## 目标
{: #objectives}

* 使用您自己的加密密钥来加密存储区中的内容
* 要求用户在访问应用程序之前进行认证
* 监视和审计云服务上与安全相关的 API 调用和其他操作

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* 可选：[{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

本教程需要[非轻量帐户](https://{DomainName}/docs/account?topic=account-accounts#accounts)，因此可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

本教程提供了一个样本应用程序，支持多组用户将文件上传到公共存储池，并通过可共享链接提供对这些文件的访问。该应用程序是用 Node.js 编写的，并作为 Docker 容器部署到 {{site.data.keyword.containershort_notm}}。它利用多种与安全相关的服务和功能来改善应用程序的安全状况。

<p style="text-align: center;">

  ![体系结构](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. 用户连接到应用程序。
2. 如果使用的是定制域和 TLS 证书，那么证书由 {{site.data.keyword.cloudcerts_short}} 进行管理和部署。
3. {{site.data.keyword.appid_short}} 可保护应用程序并将用户重定向到认证页面。用户还可以进行注册。
4. 应用程序通过存储在 {{site.data.keyword.registryshort_notm}} 中的映像在 Kubernetes 集群中运行。系统会自动扫描此映像以查找漏洞。
5. 已上传的文件存储在 {{site.data.keyword.cos_short}} 中，随附的元数据存储在 {{site.data.keyword.cloudant_short_notm}} 中。
6. 文件存储区利用用户提供的密钥来加密数据。
7. 应用程序管理活动由 {{site.data.keyword.cloudaccesstrailshort}} 进行记录。

## 开始之前
{: #prereqs}

1. 通过[执行以下步骤](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)，安装所有必需的命令行 (CLI) 工具。
2. 确保您拥有本教程中所使用插件的最新版本；使用 `ibmcloud plugin update --all` 来进行升级。

## 创建服务
{: #setup}

### 决定应用程序的部署位置

1. 确定将部署应用程序及其资源的**位置**、**Cloud Foundry 组织和空间**以及**资源组**。

### 捕获用户和应用程序活动 
{: #activity-tracker }

{{site.data.keyword.cloudaccesstrailshort}} 服务将记录用户启动的会更改 {{site.data.keyword.Bluemix_notm}} 中服务状态的活动。在本教程结束时，您可查看通过完成本教程的各步骤生成的事件。

1. 访问 {{site.data.keyword.Bluemix_notm}}“目录”并创建 [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker) 实例。请注意，每个 Cloud Foundry 空间只能有一个 {{site.data.keyword.cloudaccesstrailshort}} 实例。
2. 创建实例后，将其名称更改为 **secure-file-storage-activity-tracker**。
3. 要查看所有 Activity Tracker 事件，请确保您已分配有相应许可权：
   1. 转至[身份和访问权 > 用户](https://{DomainName}/iam/#/users)。
   2. 在列表中选择您的用户名。
   3. 在**访问策略**下，如果没有相应的策略，请在创建了 {{site.data.keyword.loganalysisshort_notm}} 服务的位置，为该服务创建具有**查看者**角色的策略。
   4. 在 **Cloud Foundry 访问权**下，确保您在供应了 {{site.data.keyword.cloudaccesstrailshort}} 的 Cloud Foundry 空间中具有**开发者**角色。如果没有，请使用组织的 Cloud Foundry 管理者来分配该角色。

在 [{{site.data.keyword.cloudaccesstrailshort}} 文档](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy)中可找到有关设置许可权的详细指示信息。
{: tip}

### 为应用程序创建集群

{{site.data.keyword.containershort_notm}} 提供了可在 Kubernetes 集群中运行的 Docker 容器中部署高可用性应用程序的环境。

如果您有现有的**标准**集群，希望在本教程中复用该集群，请跳过此部分。
{: tip}

1. 访问[集群创建页面](https://{DomainName}/containers-kubernetes/catalog/cluster/create)。
   1. 将**位置**设置为先前步骤中使用的位置。
   2. 将**集群类型**设置为**标准**。
   3. 将**可用性**设置为**单专区**。
   4. 选择**主专区**。
2. 保留缺省 **Kubernetes 版本**和**硬件隔离**。
3. 如果计划在此集群上仅部署本教程，请将**工作程序节点数**设置为 **1**。
4. 将**集群名称**设置为 **secure-file-storage-cluster**。
5. 单击**创建集群**按钮。

在供应集群期间，您将创建本教程所需的其他服务。

### 使用您自己的加密密钥

{{site.data.keyword.keymanagementserviceshort}} 可帮助您在各个 {{site.data.keyword.Bluemix_notm}} 服务中为应用程序供应加密的密钥。{{site.data.keyword.keymanagementserviceshort}} 和 {{site.data.keyword.cos_full_notm}} [一起使用可保护静态数据](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos)。在此部分中，您将为存储区创建一个根密钥。

1. 创建 [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms) 实例。
   * 将“名称”设置为 **secure-file-storage-kp**。
   * 选择要在其中创建服务实例的资源组。
2. 在**管理**下，单击**添加密钥**按钮以创建新的根密钥。此密钥将用于加密存储区内容。
   * 将“名称”设置为 **secure-file-storage-root-enckey**。
   * 将“密钥类型”设置为**根密钥**。
   * 然后，**生成密钥**。

自带密钥 (BYOK) 可通过[导入现有根密钥](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys)来提供。
{: tip}

### 设置用于用户文件的存储空间

文件共享应用程序会将文件保存到 {{site.data.keyword.cos_short}} 存储区。文件和用户之间的关系作为元数据存储在 {{site.data.keyword.cloudant_short_notm}} 数据库中。在此部分中，您将创建和配置这些服务。

#### 用于保存内容的存储区

1. 创建 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) 实例。
   * 将**名称**设置为 **secure-file-storage-cos**。
   * 使用先前服务所用的相同**资源组**。
2. 在**服务凭证**下，*新建凭证*。
   * 将**名称**设置为 **secure-file-storage-cos-acckey**。
   * 将**角色**设置为**写入者**。
   * 不要指定**服务标识**。
   * 将**内联配置参数**设置为 **{"HMAC":true}**。生成预签名的 URL 时需要此参数。
   * 单击**添加**。
   * 通过单击**查看凭证**来记下凭证。在稍后步骤中您需要这些凭证。
3. 单击菜单中的**端点**：将**弹性**设置为**区域**，将**位置**设置为目标位置。复制**公共**服务端点。稍后在应用程序配置中将使用此端点。

在创建存储区之前，将授予 **secure-file-storage-cos** 对存储在 **secure-file-storage-kp** 中的根密钥的访问权。

1. 转至 {{site.data.keyword.cloud_notm}} 控制台中的[身份和访问权 > 授权](https://{DomainName}/iam/#/authorizations)。
2. 单击**创建**按钮。
3. 在**源服务**菜单中，选择 **Cloud Object Storage**。
4. 在**源服务实例**菜单中，选择先前创建的 **secure-file-storage-cos** 服务。
5. 在**目标服务**菜单中，选择 **Key Protect**。
6. 在**目标服务实例**菜单中，选择要对其授权的 **secure-file-storage-kp** 服务。
7. 启用**读取者**角色。
8. 单击**授权**按钮。

最后创建存储区。

1. 访问[资源列表](https://{DomainName}/resources)中的 **secure-file-storage-cos** 服务实例。
2. 单击**创建存储区**。
   1. 将**名称**设置为唯一值，例如 **&lt;your-initials&gt;-secure-file-upload**。
   2. 将**弹性**设置为**区域**。
   3. 将**位置**设置为在其中创建了 **secure-file-storage-kp** 服务的位置。
   4. 将**存储类**设置为**标准**。
3. 选中**添加 Key Protect 密钥**旁的复选框。
   1. 选择 **secure-file-storage-kp** 服务。
   2. 选择 **secure-file-storage-root-enckey** 作为密钥。
4. 单击**创建存储区**。

#### 用户与其文件之间的数据库映射关系

{{site.data.keyword.cloudant_short_notm}} 数据库将包含从应用程序上传的所有文件的元数据。

1. 创建 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 实例。
   * 将**名称**设置为 **secure-file-storage-cloudant**。
   * 设置位置。
   * 使用先前服务所用的相同**资源组**。
   * 将**可用认证方法**设置为**仅使用 IAM**。
2. 在**服务凭证**下，*新建凭证*。
   * 将**名称**设置为 **secure-file-storage-cloudant-acckey**。
   * 将**角色**设置为**管理者**。
   * 保留*可选*字段的缺省值。
   * **添加**。
3. 通过单击**查看凭证**来记下凭证。在稍后步骤中您需要这些凭证。
4. 在**管理**下，启动 Cloudant 仪表板。
5. 单击**创建数据库**以创建名为 **secure-file-storage-metadata** 的数据库。

### 认证用户

通过 {{site.data.keyword.appid_short}}，可以保护资源并向应用程序添加认证。{{site.data.keyword.appid_short}} 与 {{site.data.keyword.containershort_notm}} [集成](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth)，以对访问集群中所部署应用程序的用户进行认证。

1. 创建 [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) 实例。
   * 将**服务名称**设置为 **secure-file-storage-appid**。
   * 使用先前服务所用的相同**位置**和**资源组**。
2. 在**认证设置**选项卡的**身份提供者/管理**下，添加指向将用于应用程序的域的 **Web 重定向 URL**。例如，如果集群 Ingress 子域为 `<cluster-name>.us-south.containers.appdomain.cloud`，那么重定向 URL 将为 `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback`。{{site.data.keyword.appid_short}} 需要 Web 重定向 URL 为 **https**。您可以在集群仪表板中或使用 `ibmcloud ks cluster-get <cluster-name>` 来查看 Ingress 子域。

您应该在 {{site.data.keyword.appid_short}} 仪表板中定制使用的身份提供者以及登录和用户管理体验。为简单起见，本教程使用的是缺省值。对于生产环境，请考虑使用多因子认证 (MFA) 和高级密码规则。
{: tip}

## 部署应用程序

现在所有服务都已配置。在此部分中，您要将教程应用程序部署到集群。

### 获取代码

1. 获取应用程序的代码：
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. 转至 **secure-file-storage** 目录：
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### 构建 Docker 映像

1. 在 {{site.data.keyword.registryshort_notm}} 中[构建 Docker 映像](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating)。
   - 使用 `ibmcloud cr info` 来查找注册表端点，例如 **us**.icr.io 或 **uk**.icr.io。
   - 创建用于存储容器映像的名称空间。
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - 使用 **secure-file-storage** 作为映像名称。

   ```sh
   ibmcloud cr build -t <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### 填写凭证和配置设置

1. 将 `credentials.template.env` 复制到 `credentials.env`：
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. 编辑 `credentials.env`，并使用以下值填写空白设置：
   * {{site.data.keyword.cos_short}} 服务区域端点、存储区名称、为 **secure-file-storage-cos** 创建的凭证
   * 以及 **secure-file-storage-cloudant** 的凭证。
3. 将 `secure-file-storage.template.yaml` 复制到 `secure-file-storage.yaml`：
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. 编辑 `secure-file-storage.yaml`，并将占位符（`$IMAGE_PULL_SECRET`、`$REGISTRY_URL`、`$REGISTRY_NAMESPACE`、`$IMAGE_NAME`、`$TARGET_NAMESPACE`、`$INGRESS_SUBDOMAIN` 和 `$INGRESS_SECRET`）替换为正确的值。例如，假定应用程序部署到 _default_ Kubernetes 名称空间：

|变量|值|描述|
| -------- | ----- | ----------- |
|`$IMAGE_PULL_SECRET`|在 .yaml 中使相关行保持注释掉。|用于访问注册表的私钥。|
|`$REGISTRY_URL`|*us.icr.io*|在先前部分中，在其中构建了映像的注册表。|
|`$REGISTRY_NAMESPACE`|*secure-file-storage-namespace*|在先前部分中，在其中构建了映像的注册表名称空间。|
|`$IMAGE_NAME`|*secure-file-storage*|Docker 映像的名称。|
|`$TARGET_NAMESPACE`|*default*|将在其中推送应用程序的 Kubernetes 名称空间。|
|`$INGRESS_SUBDOMAIN`|*secure-file-stora-123456.us-south.containers.appdomain.cloud*|在集群概述页面中或使用 `ibmcloud ks cluster-get secure-file-storage-cluster` 进行检索。|
|`$INGRESS_SECRET`|*secure-file-stora-123456*|在集群概述页面中或使用 `ibmcloud ks cluster-get secure-file-storage-cluster` 进行检索。|

仅当要使用非缺省 Kubernetes 名称空间时，才需要 `$IMAGE_PULL_SECRET`。这需要额外的 Kubernetes 配置（例如，[在新名称空间中创建 Docker 注册表私钥](https://{DomainName}/docs/containers?topic=containers-images#other)）。
{: tip}

### 部署到集群

1. 检索集群配置，并设置 KUBECONFIG 环境变量。
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. 创建应用程序用于获取服务凭证的私钥：
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. 将 {{site.data.keyword.appid_short_notm}} 服务实例绑定到集群。
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
   如果您有多个同名服务，那么命令会失败。您应该传递服务 GUID，而不是服务名称。要查找服务的 GUID，请使用 `ibmcloud resource service-instance secure-file-storage-appid`。
   {: tip}
4. 部署应用程序。
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## 测试应用程序

可以在 `https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/` 上访问应用程序。

1. 转至应用程序的主页。这会将您重定向到 {{site.data.keyword.appid_short_notm}} 缺省登录页面。
2. 使用有效电子邮件地址注册新帐户。
3. 等待收件箱收到用于验证帐户的电子邮件。
4. 登录。
5. 选择要上传的文件。单击**上传**。
6. 对文件使用**共享**操作，以生成预签名 URL，此 URL 可以与要访问该文件的其他人共享。链接设置为在 5 分钟后到期。

已认证的用户有自己的空间可存储文件。虽然他们无法查看彼此的文件，但可以生成预签名 URL 来授予对特定文件的临时访问权。

可以在[源代码存储库](https://github.com/IBM-Cloud/secure-file-storage)中找到有关应用程序的更多详细信息。

## 查看安全性事件
既然已成功部署应用程序及其服务，现在可以查看该过程生成的安全性事件。所有事件都在 {{site.data.keyword.cloudaccesstrailshort}} 实例中集中提供，可以通过[图形 UI (Kibana)、CLI 或 API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status) 进行访问。

1. 在 [{{site.data.keyword.Bluemix_notm}} 资源列表](https://{DomainName}/resources)中，找到 {{site.data.keyword.cloudaccesstrailshort}} 实例 **secure-file-storage-activity-tracker**，并打开其仪表板。
2. 缺省情况下，**管理**选项卡会显示**空间日志**。通过单击**查看日志**旁边的选择器，可切换到**帐户日志**。其中应该会显示若干事件。
3. 单击**在 Kibana 中查看**以打开完全的事件查看器。
4. 通过单击**发现**，查看事件详细信息。
5. 在**可用字段**中，添加 **action_str** 和 **initiator.name_str**。
6. 通过单击三角形图标，然后选择表或 JSON 格式，展开感兴趣的条目。

可以更改自动刷新的设置和显示的时间范围，从而更改数据分析方式以及所分析的数据内容。
{: tip}

## 可选：使用定制域并加密网络流量
缺省情况下，可以在 `containers.appdomain.cloud` 子域的通用主机名上访问应用程序。但是，也可以将定制域用于部署的应用程序。要持续支持 **https**，需要提供对加密网络流量的访问权（通过所需主机名的证书或通配符证书）。在以下部分中，您要将现有证书上传到 {{site.data.keyword.cloudcerts_short}}，然后将其部署到集群。此外，还将更新应用程序配置以使用定制域。

在 [{{site.data.keyword.cloud}} 博客](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)中描述了如何从 [Let's Encrypt](https://letsencrypt.org/) 获取证书的示例。
{: tip}

1. 创建 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager) 实例。
   * 将“名称”设置为 **secure-file-storage-certmgr**。
   * 使用其他服务所用的相同**位置**和**资源组**。
2. 单击**导入证书**以导入您的证书。
   * 将“名称”设置为 **SecFileStorage**，将“描述”设置为 **e2e 安全性教程的证书**。
   * 使用**浏览**按钮上传该证书文件。
   * 单击**导入**以完成导入过程。
3. 找到已导入证书的条目，并将其展开。
   * 验证证书条目，例如验证域名是否与您的定制域相匹配。如果上传的是通配符证书，那么域名中会包含星号。
   * 单击证书 **crn** 旁边的**复制**符号。
4. 切换到命令行，以将证书信息作为私钥部署到集群。在上一步中复制了 crn 中的内容后，执行以下命令。
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <the copied crn from previous step>
   ```
   {: codeblock}
   通过执行以下命令，验证集群是否知道有关该证书的信息。
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. 编辑 `secure-file-storage.yaml` 文件。
   * 找到与 **Ingress** 相关的部分。
   * 取消注释并编辑涵盖定制域的行，填写您的域名和主机名。
   定制域的 CNAME 条目需要指向集群。有关详细信息，请查看本文档中的[关于映射定制域的指南](https://{DomainName}/docs/containers?topic=containers-ingress#private_3)。
   {: tip}
6. 将配置更改应用于已部署的配置：
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. 切换回浏览器。在 [{{site.data.keyword.Bluemix_notm}} 资源列表](https://{DomainName}/resources)中，找到先前创建并配置的 {{site.data.keyword.appid_short}} 服务，然后启动其管理仪表板。
   * 转至**身份提供者**下的**管理**，然后转至**设置**。
   * 在**添加 Web 重定向 URL** 表单中，将 `https://secure-file-storage.<your custom domain>/appid_callback` 添加为另一个 URL。
8. 现在，一切应该都已就位。请通过在配置的定制域 `https://secure-file-storage.<your custom domain>` 上访问应用程序来测试应用程序。

## 扩展教程

安全永无止境。请尝试以下建议来增强应用程序的安全性。

* 使用 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 执行静态和动态代码扫描
* 通过将策略和规则与 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 配合使用，确保仅发布高质量代码

## 除去资源
{:removeresources}

要除去资源，请删除部署的容器，然后删除供应的服务。

1. 删除部署的容器：
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. 删除用于部署的私钥：
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. 从容器注册表中除去 Docker 映像：
   ```sh
   ibmcloud cr image-rm <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. 在 [{{site.data.keyword.Bluemix_notm}} 资源列表](https://{DomainName}/resources)中，找到为此教程创建的资源。使用搜索框和 **secure-file-storage** 作为模式。通过单击每个服务旁边的上下文菜单，并选择**删除服务**来删除每个服务。请注意，{{site.data.keyword.keymanagementserviceshort}} 服务只有在删除密钥后才能除去。单击该服务实例以访问相关仪表板并删除密钥。

如果您与其他用户共享帐户，请始终确保仅删除您自己的资源。
{: tip}

## 相关内容
{:related}

* [{{site.data.keyword.security-advisor_short}} 文档](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [Security to safeguard and monitor your cloud apps](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [{{site.data.keyword.Bluemix_notm}} Platform 安全性](https://{DomainName}/docs/overview?topic=overview-security#security)
* [Security in the IBM Cloud](https://www.ibm.com/cloud/security)
* [教程：组织用户、团队和应用程序的最佳实践](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [Secure Apps on IBM Cloud with Wildcard Certificates](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
