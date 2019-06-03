---
copyright:
  years: 2018, 2019
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

# 跨多个区域部署无服务器应用程序
{: #multi-region-serverless}

本教程说明了如何配置 IBM Cloud Internet Services 和 {{site.data.keyword.openwhisk_short}}，以跨多个区域部署无服务器应用程序。

通过无服务器计算平台，开发者能快速构建 API，而无需服务器。{{site.data.keyword.openwhisk}} 支持为操作自动生成 REST API，将操作转换为 HTTP 端点，并能够启用安全 API 认证。此功能不仅有助于向外部使用者公开 API，还可帮助构建微服务应用程序。

{{site.data.keyword.openwhisk_short}} 在多个 {{site.data.keyword.cloud_notm}} 位置可用。为了提高弹性并缩短网络等待时间，应用程序可以将其后端部署在多个位置。然后，通过 IBM Cloud Internet Services (CIS)，开发者可以公开单个入口点，用于负责将流量分发给距离最近且正常运行的后端。

## 目标
{: #objectives}

* 部署 {{site.data.keyword.openwhisk_short}} 操作。
* 使用定制域通过 {{site.data.keyword.APIM}} 公开操作。
* 使用 Cloud Internet Services 跨多个位置分发流量。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

本教程讨论的是使用 {{site.data.keyword.openwhisk_short}} 实现后端的公共 Web 应用程序。为了缩短网络等待时间并防止中断，应用程序将部署在多个位置。本教程中配置了两个位置。

<p style="text-align: center;">

  ![体系结构](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. 用户访问该应用程序。请求通过 Internet Services。
2. Internet Services 将用户重定向到距离最近且正常运行的 API 后端。
3. {{site.data.keyword.cloudcerts_short}} 为 API 提供其 SSL 证书。流量是端到端加密的。
4. API 通过 {{site.data.keyword.openwhisk_short}} 实现。

## 开始之前
{: #prereqs}

1. Cloud Internet Services 需要您拥有定制域，以便可将此域的 DNS 配置为指向 Cloud Internet Services 名称服务器。如果您没有域，可以向 [godaddy.com](http://godaddy.com) 等注册商购买域。
1. 通过[执行以下步骤](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)，安装所有必需的命令行 (CLI) 工具。

## 配置定制域

第一步是创建 IBM Cloud Internet Services (CIS) 实例，并将定制域指向 CIS 名称服务器。

1. 导航至 {{site.data.keyword.Bluemix_notm}}“目录”中的 [Internet Services](https://{DomainName}/catalog/services/internet-services)。
1. 设置服务名称，然后单击**创建**以创建服务实例。对于本教程，您可以使用任何定价套餐。
1. 供应服务实例后，通过单击**开始使用**来设置域名，然后单击**添加域**。
1. 单击**下一步**。分配名称服务器后，配置注册商或域名提供商，以使用列出的名称服务器。
1. 配置注册商或 DNS 提供商后，最长可能需要 24 小时，更改才会生效。

   在“概述”页面上，域的状态从*暂挂*更改为*活动*时，可以使用 `dig <your_domain_name> ns` 命令来验证新的名称服务器是否已生效。
   {:tip}

### 获取定制域的证书

要通过定制域公开 {{site.data.keyword.openwhisk_short}} 操作，需要安全的 HTTPS 连接。您应该获取计划用于无服务器后端的域和子域的 SSL 证书。假定有类似 *mydomain.com* 的域名，那么可以在 *api.mydomain.com* 上托管操作。需要签发用于 *api.mydomain.com* 的证书。

可以从 [Let's Encrypt](https://letsencrypt.org/) 获取免费的 SSL 证书。在此过程中，可能需要在 Cloud Internet Services 的 DNS 接口中配置 TXT 类型的 DNS 记录，以证明您是域的所有者。
{:tip}

获得域的 SSL 证书和专用密钥后，请务必将其转换为 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 格式。

1. 将证书转换为 PEM 格式：
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. 将专用密钥转换为 PEM 格式：
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### 将证书导入到中央存储库

1. 在支持的位置中创建 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 实例。
1. 在服务仪表板中，使用**导入证书**：
   * 将**名称**设置为定制子域和域，例如 *api.mydomain.com*。
   * 浏览以找到 PEM 格式的**证书文件**。
   * 浏览以找到 PEM 格式的**专用密钥文件**。
   * **导入**。

## 在多个位置部署操作

在此部分中，您将创建操作，将其公开为 API，然后使用存储在 {{site.data.keyword.cloudcerts_short}} 中的 SSL 证书将定制域映射到 API。

<p style="text-align: center;">

  ![API 体系结构](images/solution44-multi-region-serverless/api-architecture.png)
</p>

**doWork** 操作可实现其中一个 API 操作。**healthz** 将稍后用于检查 API 是否运行正常。检查结果可能只返回 *OK*，不包含任何操作，也可能会执行更复杂的检查，例如对数据库或对 API 所需的其他关键服务执行 ping 操作。

对于要托管应用程序后端的每个位置，需要重复以下三个部分。在本教程中，可以选取*达拉斯 (us-south)* 和*伦敦 (eu-gb)* 作为目标。

### 定义操作

1. 转至 [{{site.data.keyword.openwhisk_short}} / 操作](https://{DomainName}/openwhisk/actions)。
1. 切换到目标位置，然后选择要部署操作的组织和空间。
1. 创建操作
   1. 将**名称**设置为 **doWork**。
   1. 将**封装包**设置为 **default**。
   1. 将**运行时**设置为最近版本的 **Node.js**。
   1. **创建**。
1. 将操作代码更改为：
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. **保存**
1. 创建另一个操作以用作 API 的运行状况检查：
   1. 将**名称**设置为 **healthz**。
   1. 将**封装包**设置为 **default**。
   1. 将**运行时**设置为最近的 **Node.js**。
   1. **创建**。
1. 将操作代码更改为：
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. **保存**

### 使用受管 API 公开操作

下一步涉及创建受管 API 来公开操作。

1. 转至 [{{site.data.keyword.openwhisk_short}} / API](https://{DomainName}/openwhisk/apimanagement)。
1. 创建新的受管 {{site.data.keyword.openwhisk_short}} API：
   1. 将 **API 名称**设置为 **App API**。
   1. 将**基本路径**设置为 **/api**。
1. 创建操作：
   1. 将**路径**设置为 **/do**。
   1. 将**动词**设置为 **GET**。
   1. 将**包**设置为 **default**。
   1. 将**操作**设置为 **doWork**。
   1. **创建**
1. 创建另一个操作：
   1. 将**路径**设置为 **/healthz**。
   1. 将**动词**设置为 **GET**。
   1. 将**包**设置为 **default**。
   1. 将**操作**设置为 **healthz**。
   1. **创建**
1. **保存**该 API

### 为受管 API 配置定制域

通过创建受管 API，您可获得缺省端点，如 `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`。在此部分中，您将配置此端点，使其能够处理来自于定制子域的请求，该域稍后将在 IBM Cloud Internet Services 中进行配置。

1. 转至 [API / 定制域](https://{DomainName}/apis/domains)。
1. 在**区域**选择器中，选择目标位置。
1. 找到链接到在其中创建了操作和受管 API 的组织和空间的定制域。单击“操作”菜单中的**更改设置**。
1. 记下**缺省域 / 别名**值。
1. 选中**应用定制域**
   1. 将**域名**设置为将用于 CIS 全局负载均衡器的域，例如 *api.mydomain.com*。
   1. 选择持有证书的 {{site.data.keyword.cloudcerts_short}} 实例。
   1. 从域中选择证书。
1. 转至 **Cloud Internet Services** 的仪表板，在**可靠性 / DNS** 下，创建新的 **DNS TXT 记录**：
   1. 将**名称**设置为定制子域，例如 **api**。
   1. 将**内容**设置为**缺省域 / 别名**。
   1. 保存记录。
1. 保存定制域设置。该对话框将检查是否存在 DNS TXT 记录。

   如果找不到 TXT 记录，那么可能需要等待该记录传播，然后重试保存设置。应用设置后，可以除去 DNS TXT 记录。
   {: tip}

重复以上各部分以配置更多位置。

## 在各位置之间分发流量

**在此阶段，您已在多个位置中设置操作**，但没有单个入口点可以对其进行访问。在此部分中，您将配置全局负载均衡器 (GLB) 以在各位置之间分发流量。

<p style="text-align: center;">

  ![全局负载均衡器的体系结构](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### 创建运行状况检查

Internet Services 将定期调用此端点以检查后端的运行状况。

1. 转至 IBM Cloud Internet Services 实例的仪表板。
1. 在**可靠性 / 全局负载均衡器**下，创建运行状况检查：
   1. 将**监视类型**设置为 **HTTPS**。
   1. 将**路径**设置为 **/api/healthz**。
   1. **供应资源**。

### 创建源池

通过为每个位置创建一个池，可以稍后在全局负载均衡器中配置地理路径，以将用户重定向到距离最近的位置。另一个选项是创建包含所有位置的单个池，并使负载均衡器循环使用池中的各个源。

对于每个位置：
1. 创建源池。
1. 将**名称**设置为 **app-&lt;location&gt;**，例如 _app-Dallas_。
1. 选择先前创建的运行状况检查。
1. 将**运行状况检查区域**设置为距离部署 {{site.data.keyword.openwhisk_short}} 的位置最近的区域。
1. 将**源名称**设置为 **app-&lt;location&gt;**。
1. 将**源地址**设置为受管 API 的缺省域/别名（例如，_5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_）。
1. **供应资源**。

### 创建全局负载均衡器

1. 创建负载均衡器。
1. 将**均衡器主机名**设置为 **api.mydomain.com**。
1. 添加区域源池。
1. **供应资源**。

片刻之后，转至 `https://api.mydomain.com/api/do?name=John&place=Earth`。这应该会回复第一个正常运行的池中所运行的函数。

### 测试故障转移

要测试故障转移，池运行状况检查必须失败，这样 GLB 才会重定向到下一个正常运行的池。要模拟故障，可以修改运行状况检查函数以使其失败。

1. 转至 [{{site.data.keyword.openwhisk_short}} / 操作](https://{DomainName}/openwhisk/actions)。
1. 选择 GLB 中配置的第一个位置。
1. 编辑 `healthz` 函数，将其实现更改为 `throw new Error()`。
1. 保存。
1. 等待对此源池运行运行状况检查。
1. 再次获取 `https://api.mydomain.com/api/do?name=John&place=Earth`，它现在应该重定向到另一个正常运行的源。
1. 还原代码更改，使其恢复为正常运行的源。

## 除去资源
{: #removeresources}

### 除去 CIS 资源

1. 除去 GLB。
1. 除去源池。
1. 除去运行状况检查。

### 除去操作

1. 除去 [API](https://{DomainName}/openwhisk/apimanagement)
1. 除去[操作](https://{DomainName}/openwhisk/actions)

## 相关内容
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [使用 Cloud Internet Services 的富有弹性且安全的多区域 Kubernetes 集群](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
