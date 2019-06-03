---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 跨多个区域保护 Web 应用程序
{: #multi-region-webapp}

本教程将全程指导您使用 [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) 管道跨多个区域对 Cloud Foundry 应用程序进行创建、保护、部署和负载均衡。

应用程序或应用程序的某些部分会发生中断 - 这是事实。原因可能是代码中存在问题，计划维护影响应用程序使用的资源，硬件故障导致托管应用程序的专区、位置或数据中心停止运行。其中任何问题都有可能会发生，因此您必须做好准备应对。借助 {{site.data.keyword.Bluemix_notm}}，可以将应用程序部署到[多个位置](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg)，以提高应用程序的弹性。现在，应用程序在多个位置运行时，还可以将用户流量重定向到距离最近的位置以缩短等待时间。

## 目标

* 使用 {{site.data.keyword.contdelivery_short}} 将 Cloud Foundry 应用程序部署到多个位置。
* 将定制域映射到应用程序。
* 为多位置应用程序配置全局负载均衡。
* 将 SSL 证书绑定到应用程序。
* 监视应用程序性能。

## 使用的服务

本教程使用以下运行时和服务：
* [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry 应用程序
* [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) for DevOps
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构

本教程涉及主动/主动场景，其中应用程序的两个副本部署在两个不同的位置，这两个副本以循环方式处理客户请求。如果一个副本失败，DNS 配置会自动指向正常运行的位置。

<p style="text-align: center;">

   ![体系结构](./images/solution1/Architecture.png)
</p>

## 创建 Node.js 应用程序
{: #create}

首先创建在 Cloud Foundry 环境中运行的 Node.js 入门模板应用程序。

1. 单击 {{site.data.keyword.Bluemix_notm}} 控制台中的**[目录](https://{DomainName}/catalog/)**。
2. 单击**平台**类别下的 **Cloud Foundry 应用程序**，然后选择 **[{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)**。
     ![](images/solution1/SDKforNodejs.png)
3. 输入应用程序的**唯一名称**，这也将是您的主机名，例如：myusername-nodeapp。然后，单击**创建**。
4.  应用程序启动后，单击**概述**页面上的**访问 URL** 链接，以在新选项卡上实时查看应用程序。

![HelloWorld](images/solution1/HelloWorld.png)

很好！您已拥有自己的在 {{site.data.keyword.Bluemix_notm}} 中运行的 Node.js 入门模板应用程序。

接下来，将应用程序的源代码推送到存储库并自动部署更改。

## 设置源代码控制和 {{site.data.keyword.contdelivery_short}}
{: #devops}

在此步骤中，将设置 Git 源代码控制存储库来存储代码，然后创建管道，用于自动部署任何代码更改。

1. 在刚才创建的应用程序的左侧窗格中，选择**概述**，然后滚动以找到 **{{site.data.keyword.contdelivery_short}}**。单击**启用**。

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. 保留缺省选项，然后单击**创建**。现在，您应该已创建缺省**工具链**。

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. 选择**代码**下的 **Git** 磁贴。然后，系统会将您定向到 Git 存储库页面。
4. 如果尚未设置 SSH 密钥，那么在顶部应该会看到包含相关指示信息的通知栏。通过在新选项卡中打开**添加 SSH 密钥**链接来执行相关步骤，或者如果要使用 HTTPS 而不使用 SSH，请通过单击**创建个人访问令牌**来执行相关步骤。请记住保存密钥或令牌以供未来引用。
5. 选择 SSH 或 HTTPS，然后复制 Git URL。将源克隆到本地计算机。
   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   **注：**如果系统提示您输入用户名，请提供您的 Git 用户名。对于密码，请使用现有的 **SSH 密钥**或**个人访问令牌**，或者使用在上一步中创建的令牌。
6. 在所选的 IDE 中打开克隆的存储库，然后导航至 `public/index.html`。现在，将更新代码。请尝试将“Hello World”更改为其他内容。
7. 通过依次运行 `npm install`、`npm build`、`npm start ` 命令来本地运行应用程序，然后在浏览器中访问 `localhost:<port_number>`。
  **<port_number>** 如控制台中所显示。
8. 通过三个简单步骤将更改推送到存储库：添加、提交和推送。
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
9. 转至先前创建的工具链，然后单击**交付管道**磁贴。
10. 确认是否显示有**构建**和**部署**阶段。
   ![HelloWorld](images/solution1/DevOps_Pipeline.png)
11. 等待**部署**阶段完成。
12. 单击“最后一次执行结果”下的应用程序 **URL** 可实时查看更改。

继续对应用程序做进一步更改，并定期将更改落实到 Git 存储库。如果没有看到应用程序在更新，请检查管道的“部署”和“构建”阶段的日志。

## 部署到其他位置
{: #deploy_another_region}

接下来，要将同一应用程序部署到其他 {{site.data.keyword.Bluemix_notm}} 位置。我们可以使用相同的工具链，但会添加另一个“部署”阶段来处理将应用程序部署到其他位置的过程。

1. 导航至应用程序**概述**，然后滚动以找到**查看工具链**。
2. 从“交付”中，选择**交付管道**。
3. 单击**部署**阶段上的**齿轮**图标，然后选择**克隆阶段**。
   ![HelloWorld](images/solution1/CloneStage.png)
4. 将阶段重命名为“部署到 UK”，然后选择**作业**。
5. 将 **IBM Cloud 区域**更改为**伦敦 - https://api.eu-gb.bluemix.net**。如果还没有空间，请创建**空间**。
6. 将**部署脚本**更改为 `cf push "${CF_APP}" -d eu-gb.mybluemix.net`。

   ![HelloWorld](images/solution1/DeployToUK.png)
7. 单击**保存**，然后通过单击**播放**按钮来运行新阶段。

## 向 IBM Cloud Internet Services 注册定制域

{: #domain_cis}

IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) 是一个统一平台，用于为 Web 应用程序配置和管理域名系统 (DNS)、全局负载均衡 (GLB)、Web 应用程序防火墙 (WAF) 以及防御分布式拒绝服务 (DDoS)。此平台为在 IBM Cloud 上运行业务的客户提供快速、高性能、可靠且安全的因特网服务，其中有三个主要功能可增强工作流程：安全性、可靠性和性能。  

部署真实世界的应用程序时，您可能希望使用自己的域，而不是 IBM 提供的 mybluemix.net 域。在此步骤中，拥有定制域后，可以使用 IBM Cloud Internet Services 提供的 DNS 服务器。

1. 向注册商（例如，[http://godaddy.com](http://godaddy.com)）购买域。
2. 导航至 {{site.data.keyword.Bluemix_notm}}“目录”中的 [Internet Services](https://{DomainName}/catalog/services/internet-services)。
2. 输入服务名称，然后单击**创建**以创建服务实例。
3. 供应服务实例后，设置域名，然后单击**添加域**。
4. 分配名称服务器后，配置注册商或域名提供商，以使用列出的名称服务器。
5. 配置注册商或 DNS 提供商后，最长可能需要 24 小时，更改才会生效。在“概述”页面上，域的状态从*暂挂*更改为*活动*时，可以使用 `dig <your_domain_name> ns` 命令来验证 IBM Cloud 名称服务器是否已生效。
  {:tip}

## 向应用程序添加全局负载均衡

{: #add_glb}

在此部分中，您将使用 IBM Cloud Internet Services 中的全局负载均衡器 (GLB) 来管理多个位置的流量。GLB 利用了支持将流量分发给多个源的源池。

### 创建 GLB 之前，请为 GLB 创建运行状况检查。

1. 在 Cloud Internet Services 应用程序中，导航至**可靠性** > **全局负载均衡器**，然后单击页面底部的**创建运行状况检查**。
2. 输入要监视的路径，例如 `/`，然后选择类型（HTTP 或 HTTPS）。通常，可以创建专用的运行状况端点。单击**供应 1 个 实例**。
   ![运行状况检查](images/solution1/health_check.png)

### 在此之后，创建具有两个源的源池。

1. 单击**创建池**。
2. 输入池的名称，选择刚才创建的运行状况检查，以及靠近 Node.js 应用程序所在位置的区域。
3. 输入位于达拉斯的第一个源的名称和应用程序的主机名：`<your_app>.mybluemix.net`。
4. 与此类似，添加另一个源，其中源地址指向位于伦敦的应用程序：`<your_app>.eu-gb.mybluemix.net`。
5. 单击**供应 1 个 实例**。
   ![源池](images/solution1/origin_pool.png)

### 创建全局负载均衡器 (GLB)。 

1. 单击**创建负载均衡器**。
2. 输入全局负载均衡器的名称。此名称还将是通用应用程序 URL (`http://<glb_name>.<your_domain_name>`) 的一部分，不管是哪个位置。
3. 单击**添加池**，然后选择刚才创建的源池。
4. 单击**供应 1 个 实例**。
   ![全局负载均衡器](images/solution1/load_balancer.png)

在此阶段，GLB 已配置，但 Cloud Foundry 应用程序尚未准备好回复来自所配置 GLB 域名的请求。要完成配置，将使用定制域和路径来更新应用程序。

## 配置应用程序的定制域和路径

{: #add_domain}

在此步骤中，要将定制域名映射到运行应用程序的 {{site.data.keyword.Bluemix_notm}} 位置的安全端点。

1. 在菜单栏中，单击**管理**，然后单击**帐户**：[帐户](https://{DomainName}/account)。
2. 在“帐户”页面上，导航至应用程序的 **Cloud Foundry 组织**，然后从“操作”列中，选择**域**。
3. 单击**添加域**，然后输入从注册商那里获取的定制域名。
4. 选择正确的位置，然后单击**保存**。
5. 与此类似，将定制域名添加到“伦敦”。
6. 返回到 {{site.data.keyword.Bluemix_notm}} [资源列表](https://{DomainName}/resources)，导航至 **Cloud Foundry 应用程序**，单击位于达拉斯的应用程序，单击**路径** > **编辑路径**，然后单击**添加路径**。
   ![添加路径](images/solution1/ApplicationRoutes.png)
7. 输入先前在**输入主机（可选）**字段中配置的 GLB 主机名，然后选择刚才添加的定制域。单击**保存**。
8. 与此类似，为位于伦敦的应用程序配置域和路径。

此时，可以使用 URL `<glb_name>.<your_domain_name>` 来访问应用程序，并且全局负载均衡器会自动为多位置应用程序分发流量。要对此进行验证，可以停止位于达拉斯的应用程序，使位于伦敦的应用程序保持活动状态，然后通过全局负载均衡器来访问应用程序。

虽然此时上述操作有效，但由于在先前的步骤中配置了持续交付，因此在触发另一个构建时，可能会覆盖配置。要使这些更改持久存储，请返回到工具链并修改 *manifest.yml* 文件：

1. 在 {{site.data.keyword.Bluemix_notm}} [资源列表](https://{DomainName}/resources)中，导航至 **Cloud Foundry 应用程序**，单击位于达拉斯的应用程序，导航至应用程序**概述**，然后滚动以找到**查看工具链**。
2. 选择“代码”下的 Git 磁贴。
3. 选择 *manifest.yml*。
4. 单击**编辑**，然后添加定制路径。仅替换 `Routes` 中的原始域和主机配置。

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <glb_name>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. 落实更改，并确保两个位置的构建都成功。  

## 替代方法：将定制域映射到 IBM Cloud 系统域

您可能不希望在多位置应用程序前端使用全局负载均衡器，但需要将定制域名映射到运行应用程序的 {{site.data.keyword.Bluemix_notm}} 位置的安全端点。

使用 Cloud Intenet Services 应用程序时，请执行以下步骤为应用程序设置 `CNAME` 记录：

1. 在 Cloud Internet Services 应用程序中，导航至**可靠性** > **DNS**。
2. 从**类型**下拉列表中，选择 **CNAME**，在“名称”字段中，输入应用程序的别名，然后在“域名”字段中，输入应用程序 URL。位于达拉斯的应用程序 `<your_app>.mybluemix.net` 可以映射到 CNAME `<your_app>`。
3. 单击**添加记录**。将“代理”切换控件切换到“开启”可增强应用程序的安全性。
4. 与此类似，为“伦敦”端点设置 `CNAME` 记录。
   ![CNAME 记录](images/solution1/cnames.png)

使用除 `mybluemix.net` 之外的其他缺省域（例如，`cf.appdomain.cloud` 或 `cf.cloud.ibm.com`）时，请确保使用[相应的系统域](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain)。
{:tip}

如果使用的是其他 DNS 提供程序，那么设置 CNAME 记录的步骤因 DNS 提供程序而有所不同。例如，如果使用的是 GoDaddy，请遵循 GoDaddy 中的 [Domains Help](https://www.godaddy.com/help/add-a-cname-record-19236) 指南。

要使 Cloud Foundry 应用程序可通过定制域访问，需要将定制域添加到[部署了应用程序的 Cloud Foundry 组织中的域列表](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#updatingapps)。完成后，可以将路径添加到应用程序清单中：

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <your_app>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}

## 将 SSL 证书绑定到应用程序
{: #ssl}

1. 获取 SSL 证书。例如，可以从 https://www.godaddy.com/web-security/ssl-certificate 购买 SSL 证书，也可以在 https://letsencrypt.org/ 上免费生成 SSL 证书。
2. 导航至应用程序**概述** > **路径** > **管理域**。
3. 单击“SSL 证书上传”按钮并上传证书。
5. 使用 https（而不是 http）来访问应用程序。

## 监视应用程序性能
{: #monitor}

下面将检查多位置应用程序的运行状况。

1. 在应用程序仪表板中，选择**监视**。
2. 单击**查看所有测试**
   ![](images/solution1/alert_frequency.png)。

Availability Monitoring 在全球各个位置全天候运行合成测试，以主动检测和修复性能问题，避免这些问题影响用户。如果为应用程序配置了定制路径，请更改测试定义，以通过其定制域来访问应用程序。

## 除去资源

* 删除工具链
* 删除在两个位置部署的两个 Cloud Foundry 应用程序
* 删除 GLB、源池和运行状况检查
* 删除 DNS 配置
* 删除 Internet Services 实例

## 相关内容

[添加 Cloudant 数据库](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[自动缩放 Cloud Foundry 应用程序](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
