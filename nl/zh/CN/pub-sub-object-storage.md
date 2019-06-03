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

# 使用 Object Storage 和发布/预订消息传递进行异步数据处理
{: #pub-sub-object-storage}
在本教程中，您将学习如何使用基于 Apache Kafka 的消息传递服务，将长时间运行的工作负载编排到在 Kubernetes 集群中运行的应用程序。此模式用于使应用程序分离，从而可以更好地控制缩放和性能。{{site.data.keyword.messagehub}} 可用于对要执行的工作进行排队，而不会影响生产者应用程序，因此是用于长时间运行的任务的理想系统。 

{:shortdesc}

您将使用文件处理示例来模拟此模式。首先创建一个 UI 应用程序，用于将文件上传到 Object Storage，并生成指示要执行的工作的消息。接下来，将创建一个单独的工作程序应用程序，用于在收到消息时，异步处理用户上传的文件。

## 目标
{: #objectives}

* 使用 {{site.data.keyword.messagehub}} 实现生产者/使用者模式
* 将服务绑定到 Kubernetes 集群

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

在本教程中，UI 应用程序是用 Node.js 编写的，而工作程序应用程序是用 Java 编写的，这突出了此模式的灵活性。尽管在本教程中这两个应用程序都在同一 Kubernetes 集群中运行，但其中任一应用程序也可以实现为 Cloud Foundry 应用程序或无服务器函数。

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. 用户使用 UI 应用程序上传文件。
2. 文件保存在 {{site.data.keyword.cos_full_notm}} 中。
3. 消息发送到 {{site.data.keyword.messagehub}} 主题，指示新文件正在等待处理。
4. 准备就绪后，工作程序将侦听消息并开始处理新文件。

## 开始之前
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - 用于安装 {{site.data.keyword.cloud_notm}} CLI、Kubernetes、Helm 和 Docker 的工具。

## 创建 Kubernetes 集群
{: #create_kube_cluster}

1. 通过[目录](https://{DomainName}/containers-kubernetes/launch)，创建 Kubernetes 集群。将其命名为 `mycluster`，以方便遵循此教程。此教程可以使用**免费**集群完成。
   ![在 IBM Cloud 上创建 Kubernetes 集群](images/solution25/KubernetesClusterCreation.png)
2. 检查**集群**和**工作程序节点**的状态，并等待它们变为**准备就绪**。

### 配置 kubectl

在此步骤中，您将配置 kubectl 以指向新创建的集群。[kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) 是一个命令行工具，用于与 Kubernetes 集群进行交互。

1. 使用 `ibmcloud login` 以交互方式登录。提供在其中创建集群的组织、位置和空间。可以通过运行 `ibmcloud target` 命令重新确认详细信息。
2. 集群准备就绪后，检索集群配置：
   ```bash
ibmcloud cs cluster-config <cluster-name>
```
   {: pre}
3. 复制并粘贴 **export** 命令，以如指示设置 KUBECONFIG 环境变量。要验证 KUBECONFIG 环境变量是否设置正确，请运行以下命令：
  `echo $KUBECONFIG`
4. 检查 `kubectl` 命令是否配置正确
   ```bash
    kubectl cluster-info
    ```
  {: pre}
   ![](images/solution2/kubectl_cluster-info.png)

## 创建 {{site.data.keyword.messagehub}} 实例
 {: #create_messagehub}

{{site.data.keyword.messagehub}} 是一种基于 Apache Kafka 的消息传递服务，具有快速、可缩放和完全受管的特点；Apache Kafka 是一种开放式源代码高吞吐量消息传递系统，为处理实时数据输送提供了低等待时间的平台。

 1. 在仪表板中，单击[**创建资源**](https://{DomainName}/catalog/)，然后从“应用程序服务”部分中，选择 [**{{site.data.keyword.messagehub}}**](https://{DomainName}/catalog/services/event-streams)。
 2. 将服务命名为 `mymessagehub`，然后单击**创建**。
 3. 通过将服务实例绑定到 `default` Kubernetes 名称空间，为集群提供服务凭证。
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

cluster-service-bind 命令会创建集群私钥，用于以 JSON 格式保存服务实例的凭证。使用 `kubectl get secrets` 可看到生成的名称为 `binding-mymessagehub` 的私钥。有关更多信息，请参阅[集成服务](https://{DomainName}/docs/containers?topic=containers-integrations#integrations)。

{:tip}

## 创建 Object Storage 实例

{: #create_cos}

{{site.data.keyword.cos_full_notm}} 经过加密，分散在多个地理位置，并使用 REST API 通过 HTTP 进行访问。{{site.data.keyword.cos_full_notm}} 为非结构化数据提供了灵活、有成本效益且可缩放的云存储器。您将使用这种存储器来存储通过 UI 上传的文件。

1. 在仪表板中，单击[**创建资源**](https://{DomainName}/catalog/)，然后从“存储”部分中，选择 [**{{site.data.keyword.cos_short}}**](https://{DomainName}/catalog/services/cloud-object-storage)。
2. 将服务命名为 `myobjectstorage`，然后单击**创建**。
3. 单击**创建存储区**。
4. 将存储区名称设置为唯一名称，例如 `username-mybucket`。
5. 对于“弹性”，选择**跨区域**，对于“位置”，选择 **us-geo**，然后单击**创建**。
6. 通过将服务实例绑定到 `default` Kubernetes 名称空间，为集群提供服务凭证。
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## 将 UI 应用程序部署到集群

UI 应用程序是一个简单的 Node.js Express Web 应用程序，允许用户上传文件。此应用程序将文件存储在上面创建的 Object Storage 实例中，然后向 {{site.data.keyword.messagehub}} 主题“work-topic”发送一条消息，指示新文件已准备好供处理。

1. 在本地克隆样本应用程序存储库，然后将目录切换到 `pubsub-ui` 文件夹。
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. 打开 `config.js`，并使用您的存储区名称更新 COSBucketName。
3. 构建并部署应用程序。deploy 命令会生成 Docker 映像，将其推送到 {{site.data.keyword.registryshort_notm}}，然后创建 Kubernetes 部署。部署应用程序时，请按照交互式指示信息进行操作。
```sh
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
4. 访问应用程序，并上传 `sample-files` 文件夹中的文件。上传的文件将存储在 Object Storage 中，在工作程序应用程序对这些文件进行处理前，文件的状态将一直为“正在等待”。请使此浏览器窗口保持打开。

   ![](images/solution25/files_uploaded.png)

## 将工作程序应用程序部署到集群

工作程序应用程序是一个 Java 应用程序，用于侦听 {{site.data.keyword.messagehub}} Kafka“work-topic”主题上的消息。收到新消息时，工作程序将在消息中检索文件的名称，然后从 Object Storage 中获取文件内容。接着，工作程序将模拟对文件的处理，并在完成时向“result-work”主题发送另一条消息。UI 应用程序将侦听此主题并更新状态。

1. 将目录切换到 `pubsub-worker` 目录。
```sh
  cd ../pubsub-worker
```
2. 打开 `resources/cos.properties`，并使用您的存储区名称更新 `bucket.name` 属性。
2. 构建并部署工作程序应用程序。
```
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
3. 部署完成后，在浏览器窗口中再次检查 Web 应用程序。请注意，现在每个文件旁边的状态已更改为“已处理”。![](images/solution25/files_processed.png)

在本教程中，您学习了如何使用基于 Kafka 的 {{site.data.keyword.messagehub}} 来实现生产者/使用者模式。这允许 Web 应用程序快速执行处理，并将繁重的处理卸载到其他应用程序。需要执行工作时，生产者（Web 应用程序）会创建消息，并使工作在订阅消息的一个或多个工作程序之间进行负载均衡。您是使用在 Kubernetes 上运行的 Java 应用程序来执行处理的，但这些应用程序还可以是 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing)。在 Kubernetes 上运行的应用程序非常适合处理长时间运行和密集型工作负载，而 {{site.data.keyword.openwhisk_short}} 更适合短时间运行的进程。

## 除去资源
{:removeresources}

导航至[资源列表](https://{DomainName}/resources/)，然后
1. 删除 Kubernetes 集群 `mycluster`
2. 删除 {{site.data.keyword.cos_full_notm}} `myobjectstorage`
3. 删除 {{site.data.keyword.messagehub}} `mymessagehub`
4. 从左侧菜单中选择 **Kubernetes**，选择**注册表**，然后删除 `pubsub-xxx` 存储库。

## 相关内容
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [管理对 Object Storage 的访问](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [{{site.data.keyword.messagehub}} data processing with {{site.data.keyword.openwhisk_short}}](https://github.com/IBM/openwhisk-data-processing-message-hub)
