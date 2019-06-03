---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# 持续部署到 Kubernetes
{: #continuous-deployment-to-kubernetes}

本教程将全程指导您完成为在 {{site.data.keyword.containershort_notm}} 上运行的容器化应用程序设置持续集成和交付管道的过程。您将学习如何设置源代码控制，然后构建和测试代码，并将代码部署到不同的部署阶段。接下来，将向安全性扫描程序、Slack 通知和分析等其他服务添加集成。

{:shortdesc}

## 目标
{: #objectives}

* 创建开发和生产 Kubernetes 集群。
* 创建入门模板应用程序，在本地运行并将其推送到 Git 存储库。
* 配置 DevOps 交付管道，以连接到 Git 存储库，构建入门模板应用程序并将其部署到开发/生产集群。
* 探索并集成应用程序，以使用安全性扫描程序、Slack 通知和分析。

## 使用的服务
{: #services}

本教程使用以下 {{site.data.keyword.Bluemix_notm}} 服务：

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**注意：**本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

![](images/solution21/Architecture.png)

1. 将代码推送到专用 Git 存储库。
2. 管道选取 Git 中的更改并构建容器映像。
3. 将上传到注册表的容器映像部署到开发 Kubernetes 集群。
4. 验证更改并部署到生产集群。
5. 设置有关部署活动的 Slack 通知。


## 开始之前
{: #prereq}

* [安装 {{site.data.keyword.dev_cli_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - 通过脚本安装 docker、kubectl、helm、ibmcloud cli 和必需的插件。
* [设置 {{site.data.keyword.registrylong_notm}} CLI 和注册表名称空间](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)。
* [学习 Kubernetes 基础知识](https://kubernetes.io/docs/tutorials/kubernetes-basics/)。

## 创建开发 Kubernetes 集群
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} 通过组合 Docker 和 Kubernetes 技术、直观的用户体验以及内置安全性和隔离，提供功能强大的工具来自动对计算主机集群中的容器化应用程序进行部署、操作、缩放和监视。


要完成本教程，需要选择**标准**类型的**付费**集群。您需要设置两个集群，一个用于开发，一个用于生产。
{: shortdesc}

1. 通过 [{{site.data.keyword.Bluemix}}“目录”](https://{DomainName}/containers-kubernetes/launch)创建第一个开发 Kubernetes 集群。稍后需要重复这些步骤并创建生产集群。

   为了方便使用，请检查配置详细信息，如 CPU 数量、内存和获得的工作程序节点数。
   {:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. 选择**集群类型**，然后单击**创建集群**以供应 Kubernetes 集群。对于本教程，选择具有 2 个 **CPU** 和 4 GB **RAM** 的最小**机器类型**以及 1 个**工作程序节点**足以满足需求。其他所有选项都可以保留其缺省值。
3. 检查**集群**和**工作程序节点**的状态，并等待它们变为**准备就绪**。

**注：**不要在工作程序准备就绪之前继续操作。

## 创建入门模板应用程序
{: #create_application}

{{site.data.keyword.containershort_notm}} 提供了一批精选的入门模板应用程序，可以使用 `ibmcloud dev create` 命令或 Web 控制台来创建这些入门模板应用程序。在本教程中将使用 Web 控制台。通过生成具有所有必需的样板、构建和配置代码的应用程序入门模板，大大缩短了入门模板应用程序的开发时间，从而可以更快开始对业务逻辑进行编码。

1. 在 [{{site.data.keyword.cloud_notm}} 控制台](https://{DomainName})中，使用左侧的菜单选项，并选择 [Web 应用程序](https://{DomainName}/developer/appservice/dashboard)。
2. 在**从 Web 开始**部分下，单击**开始使用**按钮。
3. 选择 `Node.js Web App with Express.js` 磁贴，然后选择`创建应用程序`以创建 Node.js 入门模板应用程序。
4. 输入**名称** `mynodestarter`。然后，单击**创建**。

## 配置 DevOps 交付管道
{: #create_devops}

1. 既然您已成功创建了入门模板应用程序，请在**部署应用程序**下，单击**部署到云**按钮。
2. 选择 Kubernetes 集群部署方法，选择早先创建的集群，然后单击**创建**。这将创建工具链和交付管道。![](images/solution21/BindCluster.png)
3. 创建管道后，单击**查看工具链**，然后单击**交付管道**以查看管道。![](images/solution21/Delivery-pipeline.png)
4. 部署阶段完成后，单击**查看日志和历史记录**以查看日志。
5. 访问显示的 URL 以访问应用程序 (`http://worker-public-ip:portnumber/`)。 ![](images/solution21/Logs.png)
完成，您已使用 App Service UI 创建了入门模板应用程序，并配置了管道用于构建应用程序并将其部署到集群。

## 克隆应用程序并在本地构建和运行应用程序
{: #cloneandbuildapp}

在此部分中，您将使用早先部分中创建的入门模板应用程序，将其克隆到本地计算机，修改代码，然后在本地构建/运行该应用程序。
{: shortdesc}

### 克隆应用程序
1. 在工具链概述中，选择**代码**下的 **Git** 磁贴。这会将您重定向到 Git 存储库页面，在其中可以克隆存储库。![HelloWorld](images/solution21/DevOps_Toolchain.png)

2. 如果尚未设置 SSH 密钥，那么在顶部应该会看到包含相关指示信息的通知栏。通过在新选项卡中打开**添加 SSH 密钥**链接来执行相关步骤，或者如果要使用 HTTPS 而不使用 SSH，请通过单击**创建个人访问令牌**来执行相关步骤。请记住保存密钥或令牌以供未来引用。
3. 选择 SSH 或 HTTPS，然后复制 Git URL。将源克隆到本地计算机。如果系统提示您输入用户名，请提供您的 Git 用户名。对于密码，请使用现有的 **SSH 密钥**或**个人访问令牌**，或者使用在上一步中创建的令牌。

   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   {: codeblock}

4. 在所选的 IDE 中打开克隆的存储库，然后导航至 `public/index.html`。通过尝试将“Congratulations!”更改为其他内容来更新代码，然后保存该文件。

### 本地构建应用程序
您可以像通常使用 `mvn` 进行 Java 本地开发或使用 `npm` 进行节点开发一样来构建和运行应用程序。此外，还可以构建 Docker 映像并在容器中运行应用程序，以确保在本地和在云上的执行一致。使用以下步骤构建 Docker 映像。
{: shortdesc}

1. 确保本地 Docker 引擎已启动；要进行检查，请运行以下命令：
   ```
   docker ps
   ```
   {: codeblock}
2. 导航至克隆的所生成项目目录。
   ```
   cd <project name>
   ```
   {: codeblock}
3. 本地构建应用程序。
   ```
ibmcloud dev build
```
   {: codeblock}

   这可能需要几分钟才能运行，因为将下载所有应用程序依赖项，并且会构建包含应用程序和所有必需环境的 Docker 映像。

### 在本地运行应用程序

1. 运行容器。
   ```
ibmcloud dev run
```
   {: codeblock}

   这将使用本地 Docker 引擎来运行在上一步中构建的 Docker 映像。
2. 容器启动后，转至 http://localhost:3000/
   ![](images/solution21/node_starter_localhost.png)

## 将应用程序推送到 Git 存储库

在此部分中，您要将更改提交到 Git 存储库。管道将选取该提交，并自动将更改推送到集群。
1. 在终端窗口中，确保位于克隆的存储库中。
2. 通过三个简单步骤将更改推送到存储库：添加、提交和推送。
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
   {: codeblock}

3. 转至先前创建的工具链，然后单击**交付管道**磁贴。
4. 确认是否显示有**构建**和**部署**阶段。
   ![](images/solution21/Delivery-pipeline.png)
5. 等待**部署**阶段完成。
6. 单击“最后一次执行结果”下的应用程序 **URL** 可实时查看更改。

如果没有看到应用程序在更新，请检查管道的“部署”和“构建”阶段的日志。

## 使用漏洞顾问程序确保安全性
{: #vulnerability_advisor}

在此步骤中，您将探索[漏洞顾问程序](https://{DomainName}/docs/services/va?topic=va-va_index#va_index)。漏洞顾问程序用于在部署之前检查容器映像的安全状态，另外还会检查正在运行的容器的状态。

1. 转至先前创建的工具链，然后单击**交付管道**磁贴。
1. 单击**添加阶段**，将 MyStage 更改为**验证阶段**，然后单击“作业”> **添加作业**。

   1. 选择**测试**作为“作业类型”，然后将框中的**测试**更改为**漏洞顾问程序**。
   1. 在“测试类型”下，选择**漏洞顾问程序**。其他所有字段应该会自动填充。容器注册表名称空间应该与此工具链的**构建阶段**中提及的名称空间相同。
      {:tip}
   1. 编辑**测试脚本**部分，并将最后一行中的 `SAFE\ to\ deploy` 替换为 `NO\ ISSUES`
   1. 保存该阶段
1. 将**验证阶段**拖至中间位置，然后在**验证阶段**上单击**运行** ![](images/solution21/run.png)。您将看到**验证阶段**失败。

   ![](images/solution21/toolchain.png)

1. 单击**查看日志和历史记录**来查看漏洞评估。日志的末尾显示：

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   可以在[此处](https://{DomainName}/containers-kubernetes/registry/private)查看所有已扫描存储库的详细漏洞评估
   {:tip}

   如果扫描漏洞所花的时间超过 3 分钟，阶段可能会失败，称该映像*尚未扫描*。通过编辑作业脚本并增大等待扫描结果的迭代次数，可以更改此超时。
   {:tip}

1. 下面将通过执行纠正措施来修复漏洞。在 IDE 中打开克隆的存储库，或选择 Eclipse Orion Web IDE 磁贴，打开 `Dockerfile`，并在 `EXPOSE 3000` 后面添加以下命令：
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. 提交并推送更改。这应该会触发工具链，并修复**验证阶段**。

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## 创建生产 Kubernetes 集群

{: #deploytoproduction}

在此部分中，您将通过将 Kubernetes 应用程序分别部署到开发和生产环境来完成部署管道。理想情况下，我们希望为开发环境设置自动部署，而为生产环境设置手动部署。在此之前，我们先来探索可以交付此类部署的两种方式。可以将一个集群同时用于开发和生产环境。但是，建议采用两个单独的集群，一个用于开发，一个用于生产。下面将探索为生产设置第二个集群。
{: shortdesc}

1. 按照[创建开发 Kubernetes 集群](#create_kube_cluster)部分中的指示信息来创建新集群。将此集群命名为 `prod-cluster`。
2. 转至先前创建的工具链，然后单击**交付管道**磁贴。
3. 将**部署阶段**重命名为 `Deploy dev`，这可以通过单击“设置”图标 > **配置阶段**来执行。
   ![](images/solution21/deploy_stage.png)
4. 克隆 **Deploy dev** 阶段（“设置”图标 >“克隆阶段”），然后将克隆的阶段命名为 `Deploy prod`。
5. 将**阶段触发器**更改为`仅当手动运行此阶段时才运行作业`。 ![](images/solution21/prod-stage.png)
6. 在**作业**选项卡下，将集群名称更改为新创建的集群，然后**保存**该阶段。
7. 现在，您应该拥有完整的部署设置，要从开发环境部署到生产环境，必须手动运行 `Deploy prod` 阶段以部署到生产环境。![](images/solution21/full-deploy.png)

完成，现在您已创建生产集群，并将管道配置为手动将更新推送到生产集群。这是一个简化过程阶段，在更高级的场景中，您会将单元测试和集成测试包含为管道的一部分。

## 设置 Slack 通知
{: #setup_slack}

1. 返回以查看[工具链](https://{DomainName}/devops/toolchains)列表，选择您的工具链，然后单击**添加工具**。
2. 在搜索框中搜索 Slack，或向下滚动以显示 **Slack**。单击以查看配置页面。
    ![](images/solution21/configure_slack.png)
3. 对于 **Slack Webhook**，请执行此[链接](https://my.slack.com/services/new/incoming-webhook/)中的步骤。您需要使用 Slack 凭证进行登录，并提供现有通道名称或创建新通道名称。
4. 添加“入局 Webhook 集成”后，复制该 **Webhook URL**，并将其粘贴到 **Slack Webhook** 下。
5. Slack 通道是在上面创建 Webhook 集成时提供的通道名称。
6. **Slack 团队名称**是 team-name.slack.com 中的 team-name（第一部分）。例如，在 kube.slack.com 中，kube 是团队名称。
7. 单击**创建集成**。新的磁贴将添加到您的工具链中。
    ![](images/solution21/toolchain_slack.png)
8. 从现在开始，每当您的工具链执行时，您都应该会在配置的通道中看到 Slack 通知。
    ![](images/solution21/slack_channel.png)

## 除去资源
{: #removeresources}

在此步骤中，您将清理资源以除去上面创建的内容。

- 删除 Git 存储库。
- 删除工具链。
- 删除两个集群。
- 删除 Slack 通道。

## 扩展教程
{: #expandTutorial}

想要了解更多信息？下面是您接下来可以执行的操作的一些构想：

- [使用 LogDNA 和 Sysdig 分析日志并监视应用程序运行状况](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)。
- 添加测试环境并将其部署到第三个集群。
- [跨多个位置](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)部署生产集群。
- 使用 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 通过其他质量控制和分析功能来增强管道。

## 相关内容
{: #related}

* 端到端 Kubernetes 解决方案指南：[将基于 VM 的应用程序移至 Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes)。
* IBM Cloud Container Service [安全性](https://{DomainName}/docs/containers?topic=containers-security#cluster)。
* 工具链[集成](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations)。
* 使用 [LogDNA 和 Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis) 分析日志并监视应用程序运行状况。


