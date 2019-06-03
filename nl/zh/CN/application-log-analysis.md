---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 使用 LogDNA 和 Sysdig 分析日志并监视应用程序运行状况
{: #application-log-analysis}

本教程说明了可以如何使用 [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/) 服务来配置和访问部署在 {{site.data.keyword.Bluemix_notm}} 上的 Kubernetes 应用程序的日志。您要将 Python 应用程序部署到 {{site.data.keyword.containerlong_notm}} 上供应的集群，配置 LogDNA 代理程序，生成不同级别的应用程序日志以及访问工作程序日志、pod 日志或网络日志。然后，将通过 {{site.data.keyword.la_short}} Web UI 来搜索、过滤和显示这些日志。

此外，您还将设置 [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/) 服务，并配置 Sysdig 代理程序以监视应用程序和 {{site.data.keyword.containerlong_notm}} 集群的性能和运行状况。
{:shortdesc}

## 目标
{: #objectives}
* 将应用程序部署到 Kubernetes 集群以生成日志条目。
* 访问和分析不同类型的日志，以对问题进行故障诊断，并抢先一步预防问题。
* 从运营角度了解应用程序以及运行应用程序的集群的性能和运行状况。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

  ![](images/solution12/Architecture.png)

1. 用户连接到应用程序并生成日志条目。
1. 应用程序通过存储在 {{site.data.keyword.registryshort_notm}} 中的映像在 Kubernetes 集群中运行。
1. 用户将配置 {{site.data.keyword.la_full_notm}} 服务代理程序，以访问应用程序和集群级别的日志。
1. 用户将配置 {{site.data.keyword.mon_full_notm}} 服务代理程序，以监视 {{site.data.keyword.containerlong_notm}} 集群以及部署到集群的应用程序的运行状况和性能。

## 先决条件
{: #prereq}

* [安装 {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - 通过脚本安装 docker、kubectl、helm、ibmcloud cli 和必需的插件。
* [设置 {{site.data.keyword.registrylong_notm}} CLI 和注册表名称空间](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)。
* [授予用户查看 LogDNA 中日志的许可权](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna)
* [授予用户查看 Sysdig 中度量值的许可权](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig)

## 创建 Kubernetes 集群
{: #create_cluster}

{{site.data.keyword.containershort_notm}} 提供了可在 Kubernetes 集群中运行的 Docker 容器中部署高可用性应用程序的环境。

如果您有现有的**标准**集群，并希望在本教程中复用该集群，请跳过此部分。
{: tip}

1. 通过 [{{site.data.keyword.Bluemix}} 目录](https://{DomainName}/kubernetes/catalog/cluster/create)创建**新集群**，然后选择**标准**集群。对于**免费**集群，*不会*启用日志转发。
   {:tip}
1. 选择资源组和地理位置。
1. 为方便起见，请使用名称 `mycluster`，以便与本教程保持一致。
1. 选择**工作程序专区**，然后选择具有 2 个 **CPU** 和 4 GB **RAM** 的最小**机器类型**，此类型足以满足本教程的需求。
1. 选择 1 个**工作程序节点**，并使其他所有选项保持设置为缺省值。单击**创建集群**。
1. 检查**集群**和**工作程序节点**的状态，并等待状态变为**准备就绪**。

## 供应 {{site.data.keyword.la_short}} 实例
{: #provision_logna_instance}

部署到 {{site.data.keyword.Bluemix_notm}} 中 {{site.data.keyword.containerlong_notm}} 集群的应用程序可能会生成某种级别的诊断输出，即日志。作为开发者或操作员，您可能希望访问和分析不同类型的日志，例如工作程序日志、pod 日志、应用程序日志或网络日志，以对问题进行故障诊断，并抢先一步预防问题。

通过使用 {{site.data.keyword.la_short}} 服务，可以从各种源聚集日志，并根据需要保留任意长的时间。这允许在需要时分析“全局情况”，并对更复杂的情况进行故障诊断。

要供应 {{site.data.keyword.la_short}} 服务，请执行以下操作：

1. 导航至[可观察性](https://{DomainName}/observe/)页面，在**日志记录**下，单击**创建实例**。
1. 提供唯一的**服务名称**。
1. 选择区域/位置，然后选择资源组。
1. 选择 **7 天日志搜索**作为套餐，然后单击**创建**。

服务提供了一个集中日志管理系统，其中日志数据在 IBM Cloud 上托管。

## 部署 Kubernetes 应用程序并将其配置为转发日志
{: #deploy_configure_kubernetes_app}

日志记录应用程序的可随时运行的代码位于[此 GitHub 存储库中](https://github.com/IBM-Cloud/application-log-analysis)。该应用程序是使用 [Django](https://www.djangoproject.com/) 编写的，这是一种常用的 Python 服务器端 Web 框架。请克隆或下载此存储库，然后将应用程序部署到 {{site.data.keyword.Bluemix_notm}} 上的 {{site.data.keyword.containershort_notm}}。

### 部署 Python 应用程序

在终端上：
1. 克隆 GitHub 存储库：
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. 切换到应用程序目录：
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. 在 {{site.data.keyword.registryshort_notm}} 中使用 [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile) 构建 Docker 映像。
   - 使用 `ibmcloud cr info` 来查找**容器注册表**，例如 us.icr.io 或 uk.icr.io。
   - 创建用于存储容器映像的名称空间。
      ```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - 将 `<CONTAINER_REGISTRY>` 替换为容器注册表值，并将 **app-log-analysis** 用作映像名称。
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - 将 `app-log-analysis.yaml` 文件中的 **image** 值替换为映像标记 `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest`
1. 运行以下命令来检索集群配置，并设置 `KUBECONFIG` 环境变量：
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. 部署应用程序：
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. 要访问应用程序，您需要工作程序节点的`公共 IP` 以及 `NodePort`
   - 要获取公共 IP，请运行以下命令：
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - 要获取 NodePort（这将为 5 位数，例如 3xxxx），请运行以下命令：
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   现在，可以在 `http://worker-ip-address:portnumber` 上访问应用程序

### 配置集群以将日志发送到 LogDNA 实例

要配置 Kubernetes 集群以将日志发送到 {{site.data.keyword.la_full_notm}} 实例，必须在集群的每个节点上安装 *logdna-agent* pod。LogDNA 代理程序会从安装了该代理程序的 pod 中读取日志文件，并将日志数据转发到 LogDNA 实例。

1. 导航至[可观察性](https://{DomainName}/observe/)页面，然后单击**日志记录**。
1. 单击早先创建的服务旁边的**编辑日志资源**，然后选择 **Kubernetes**。
1. 在已设置 `KUBECONFIG` 环境变量的终端上复制并运行第一个命令，以使用服务实例的 LogDNA 摄入密钥来创建 Kubernetes 私钥。
1. 复制并运行第二个命令，以在 Kubernetes 集群的每个工作程序节点上部署 LogDNA 代理程序。LogDNA 代理程序使用扩展名 **.log* 和无扩展名的文件来收集日志，这些文件存储在 pod 的 */var/log* 目录中。缺省情况下，将从所有名称空间（包括 kube-system）收集日志，并自动将日志转发到 {{site.data.keyword.la_full_notm}} 服务。
1. 配置日志源后，通过单击**查看 LogDNA** 来启动 LogDNA UI。您可能需要等待几分钟后才能开始看到日志。

## 生成和访问应用程序日志
{: generate_application_logs}

在此部分中，您将生成应用程序日志，并在 LogDNA 中进行查看。

### 生成应用程序日志

在先前步骤中部署的应用程序允许以所选日志级别记录消息。可用的日志级别为**严重**、**错误**、**警告**、**参考**和**调试**。应用程序的日志记录基础架构配置为仅允许传递等于或高于设置级别的日志条目。最初，日志记录器级别设置为**警告**。因此，在服务器设置为**警告**时，以**参考**级别记录的消息不会显示在诊断输出中。

查看 [**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py) 文件中的代码。代码包含 **print** 语句以及对 **logger** 函数的调用。显示的消息会写入 **stdout** 流（常规输出，应用程序控制台/终端），而日志记录器消息会显示在 **stderr** 流（错误日志）中。

1. 访问 Web 应用程序：`http://worker-ip-address:portnumber`。
1. 通过提交不同级别的消息，可生成多个日志条目。UI 还允许更改服务器日志级别的日志记录器设置。将服务器端日志级别更改为中间的值以使其更相关。例如，可以将“500 internal server error”记录为**错误**，或将“This is my first log entry”记录为**参考**。

### 访问应用程序日志

在 LogDNA UI 中，可以使用过滤器来访问特定于应用程序的日志。

1. 在顶部栏中，单击**所有应用程序**。
1. 在“容器”下，选中 **app-log-analysis**。这将显示新的未保存视图，其中包含所有级别的应用程序日志。
1. 要查看特定日志级别的日志，请单击**所有级别**，然后选择多个级别，如“错误”、“参考”、“警告”等。

## 搜索和过滤日志
{: #search_filter_logs}

缺省情况下，{{site.data.keyword.la_short}} UI 会显示所有可用的日志条目（全部内容）。通过自动刷新，最新的条目会显示在底部。在此部分中，您将修改显示哪些内容以及显示多少内容，然后将其另存为**视图**以供未来使用。

### 搜索日志

1. 在位于 LogDNA UI 中页面底部的**搜索**输入框中，
   - 可以搜索包含特定文本的行，如“**This is my first log entry**”或 **500 internal server error**。
   - 或者通过输入 `level:info`（其中 level 是接受字符串值的字段）来搜索特定的日志级别。

   有关更多搜索字段和帮助，请单击搜索输入框旁边的“语法帮助”图标
   {:tip}
1. 要跳转至特定时间范围，请在**跳转至时间范围**输入框中，输入 **5 mins ago**。单击输入框旁边的图标可查找保留期内的其他时间格式。
1. 要突出显示搜索项，请单击**切换查看器工具**图标。
1. 在第一个输入框中输入 **error** 作为突出显示项，在第二个输入框中输入 **container** 作为突出显示项，然后查看包含这两项的突出显示行。
1. 单击**切换时间线**图标以查看包含特定时刻日志的行。

### 过滤日志

可以按标签、源、应用程序或级别来过滤日志。

1. 在顶部栏上，单击**所有标签**，然后选中 **k8s** 复选框可查看与 Kubernetes 相关的日志。
1. 单击**所有源**，然后选择要查看其日志的主机（工作程序节点）的名称。这非常适用于您的集群中有多个工作程序节点的情况。
1. 要查看容器或文件日志，请单击**所有应用程序**，然后选中要查看其日志的应用程序的相应复选框。

### 创建视图

视图是一组特定过滤器和搜索查询的已保存快捷方式。

只要搜索或过滤日志，就应该会在顶部栏中看到**未保存的视图**。要将其另存为视图，请执行以下操作：
1. 单击**所有应用程序**，然后选中 **app-log-analysis** 旁边的复选框。
1. 单击**未保存的视图** > 单击**另存为新视图/警报**，然后将视图命名为 **app-log-analysis-view**。将**类别**保留为空。
1. 单击**保存视图**，随后新视图应该会出现在左侧窗格中，并显示相应应用程序的日志。

### 使用图形和细分显示日志

在此部分中，您将创建一个仪表板，然后在其中添加一个带有细分的图形，以显示应用程序级别数据。仪表板是图形和细分的集合。

1. 在左侧窗格中，单击**仪表板**图标（在“设置”图标上方）> 单击**新建仪表板**。
1. 单击顶部栏上的**编辑**，然后将其命名为 **app-log-analysis-board**。单击**保存**。
1. 单击**添加图形**：
   - 在第一个输入框中输入 **app** 作为字段，然后按 Enter 键。
   - 选择 **app-log-analysis** 作为字段值。
   - 单击**添加图形**。
1. 选择**计数**作为度量值，以查看过去 24 小时内每个时间间隔的行数。
1. 要添加细分，请单击图形下方的箭头：
   - 选择**直方图**作为细分类型。
   - 选择**级别**作为字段类型。
   - 单击**添加细分**以查看包含为应用程序记录的所有级别的细分。

## 添加 {{site.data.keyword.mon_full_notm}} 并监视集群
{: #monitor_cluster_sysdig}

下面，您将向应用程序添加 {{site.data.keyword.mon_full_notm}}。该服务会定期检查应用程序的可用性和响应时间。

1. 导航至[可观察性](https://{DomainName}/observe/)页面，在**监视**下，单击**创建实例**。
1. 提供唯一的**服务名称**。
1. 选择区域/位置，然后选择资源组。
1. 选择**累进层**作为套餐，然后单击**创建**。
1. 单击早先创建的服务旁边的**编辑日志资源**，然后选择 **Kubernetes**。
1. 在已设置 `KUBECONFIG` 环境变量的终端上，复制并运行**将 Sysdig 代理程序安装到集群**下的命令，以在集群中部署 Sysdig 代理程序。请等待部署完成。

### 配置 {{site.data.keyword.mon_short}}

要配置 Sysdig 以监视集群的运行状况和性能，请执行以下操作：
1. 单击**查看 Sysdig**，您应该会看到 Sysdig 监视器 UI。在欢迎页面上，单击**下一步**。
1. 在设置环境下，选择 **Kubernetes** 作为安装方法。
1. 单击代理程序配置成功消息旁边的**转至下一步**，然后在下一页上，单击**开始使用**。
1. 单击**下一步**，然后单击**完成引导**以显示 Sysdig UI 中的`探索`选项卡。

### 监视集群

要检查应用程序和集群的运行状况和性能，请执行以下操作：
1. 返回到在 `http://worker-ip-address:portnumber` 上运行的应用程序，生成多个日志条目。
1. 在 Sysdig 监视器向导上，展开左侧窗格中的 **mycluster** > 展开**缺省**名称空间 > 单击 **app-log-analysis-deployment** 以查看“请求计数”、“响应时间”等。
1. 要查看 HTTP 请求-响应代码，请单击顶部栏上 **Kubernetes pod 运行状况**旁边的箭头，然后在**应用程序**下，选择 **HTTP**。在 Sysdig UI 的底部栏上，将时间间隔更改为 **10 分钟**。
1. 要监视应用程序的等待时间，
   - 在“探索”选项卡中，选择**部署和 pod**。
   - 单击 `HTTP` 旁边的箭头，然后选择“度量值”>“网络”。
   - 选择 **net.http.request.time**。
   - 选择时间：**总和**，选择组：**平均**。
   - 单击**更多选项**，然后单击**拓扑**图标。
   - 单击**完成**，然后双击框以展开视图。
1. 要监视运行应用程序的 Kubernetes 名称空间，
   - 在“探索”选项卡中，选择**部署和 pod**。
   - 单击 `net.http.request.time` 旁边的箭头。
   - 选择“缺省仪表板”> Kubernetes。
   - 选择“Kubernetes 状态”>“Kubernetes 状态概述”。

### 创建定制仪表板

除了预定义的仪表板，您还可以创建自己的定制仪表板，以便在单个位置显示运行应用程序的容器的最有用/最相关的视图和度量值。每个仪表板由一系列面板组成，这些面板配置为以多种不同的格式显示特定数据。

要创建仪表板，请执行以下操作：
1. 单击最左侧窗格中的**仪表板** > 单击**添加仪表板**。
1. 单击**空白仪表板** > 将仪表板命名为**容器请求概述** > 单击**创建仪表板**。
1. 选择**排行榜**作为新面板，并将该面板命名为**每个容器的请求时间**
   - 在**度量值**下，输入 **net.http.request.time**。
   - 作用域：单击**覆盖仪表板作用域** > 选择 **container.image** > 选择 **is** > 选择 _the application image_
   - 按 **container.id** 进行分段，您应该会看到每个容器的最终请求时间。
   - 单击**保存**。
1. 要添加新面板，请单击**加号**图标，然后选择**数字 (#)** 作为面板类型
   - 在**度量值**下，输入 **net.http.request.count** > 将时间聚集从**平均 (Avg)** 更改为**总和**。
   - 作用域：单击**覆盖仪表板作用域** > 选择 **container.image** > 选择 **is** > 选择 _the application image_
   - 与 **1 小时**前进行比较，您应该会看到每个容器的最终请求计数。
   - 单击**保存**。
1. 要编辑此仪表板的作用域，
   - 单击标题面板中的**编辑作用域**。
   - 在下拉列表中，选择/输入 **Kubernetes.cluster.name**。
   - 将显示名称保留为空，然后选择 **is**。
   - 选择 **mycluster** 作为值，然后单击**保存**。

## 除去资源
{: #remove_resource}

- 从[可观察性](https://{DomainName}/observe)页面中除去 LogDNA 和 Sysdig 实例。
- 删除包含工作程序节点、应用程序和容器的集群。此操作无法撤销。
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## 扩展教程
{: #expand_tutorial}

- 使用 [{{site.data.keyword.at_full}} 服务](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started)可跟踪应用程序如何与 IBM Cloud 服务进行交互。
- 向视图[添加警报](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts)。
- [导出日志](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export)至本地文件。

## 相关内容
{:related}
- [重置 Kubernetes 集群使用的摄入密钥](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [将日志归档到 IBM Cloud Object Storage](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [在 Sysdig 中配置警报](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [在 Sysdig UI 中使用通知通道](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
