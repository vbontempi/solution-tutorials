---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-15"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 将基于 VM 的应用程序移动到 Kubernetes
{: #vm-to-containers-and-kubernetes}

本教程将全程指导您完成使用 {{site.data.keyword.containershort_notm}} 将基于 VM 的应用程序移动到 Kubernetes 集群的过程。[{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) 通过组合 Docker 和 Kubernetes 技术、直观的用户体验以及内置安全性和隔离，提供功能强大的工具来自动对计算主机集群中的容器化应用程序进行部署、操作、缩放和监视。
{: shortdesc}

本教程中的课程包括关于如何选取现有应用程序、将应用程序容器化以及将应用程序部署到 Kubernetes 集群的概念。要将基于 VM 的应用程序容器化，可以在下列两个选项中进行选择。

1. 识别大型单一应用程序中可分隔为独立微服务的组件。可以对这些微服务进行容器化，然后将其部署到 Kubernetes 集群。
2. 将整个应用程序容器化，然后将应用程序部署到 Kubernetes 集群。

根据您拥有的应用程序的类型不同，迁移应用程序的步骤也可能不同。您可以使用此教程来了解在迁移应用程序之前必须执行的一般步骤和必须考虑的事项。

## 目标
{: #objectives}

- 了解如何识别基于 VM 的应用程序中的微服务，并了解如何在 VM 和 Kubernetes 之间映射组件。
- 了解如何将基于 VM 的应用程序容器化。
- 了解如何在 {{site.data.keyword.containershort_notm}} 中将容器部署到 Kubernetes 集群中。
- 将学到的所有做法付诸实践，在集群中运行 **JPetStore** 应用程序。

## 使用的服务
{: #products}

本教程使用以下 {{site.data.keyword.Bluemix_notm}} 服务：

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**注意：**本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{:#architecture}

### 使用 VM 的传统应用程序体系结构

下图显示了基于虚拟机的传统应用程序体系结构的示例。

<p style="text-align: center;">
![体系结构图](images/solution30/traditional_architecture.png)
</p>

1. 用户将请求发送到 Java 应用程序的公共端点。公共端点由负载均衡器服务表示，该负载均衡器对各个可用应用程序服务器实例之间的入局网络流量进行负载均衡。
2. 负载均衡器选择一个在 VM 上运行的运行状况良好的应用程序服务器实例，并转发请求。
3. 应用程序服务器将应用程序数据存储于在 VM 上运行的 MySQL 数据库中。应用程序服务器实例托管 Java 应用程序并在 VM 上运行。应用程序文件（如应用程序代码、配置文件和依赖项）存储在 VM 上。

### 容器化的体系结构

下图显示了在 Kubernetes 集群中运行的现代容器体系结构的示例。

<p style="text-align: center;">
![体系结构图](images/solution30/modern_architecture.png)
</p>

1. 用户将请求发送到 Java 应用程序的公共端点。公共端点由 Ingress 应用程序负载均衡器 (ALB) 表示，该负载均衡器对集群中各个应用程序 pod 之间的入局网络流量进行负载均衡。ALB 是一组规则，用于允许进入以公共方式公开的应用程序的入站网络流量。
2. ALB 将请求转发到集群中某个可用的应用程序 pod。应用程序 pod 在工作程序节点上运行，这些节点可以是虚拟机，也可以是物理机器。
3. 应用程序 pod 将数据存储在持久卷中。持久卷可用于在应用程序实例或工作程序节点之间共享数据。
4. 应用程序 pod 将数据存储在 {{site.data.keyword.Bluemix_notm}} 数据库服务中。您可以在 Kubernetes 集群中运行自己的数据库，但使用受管的数据库即服务 (DBasS) 通常更容易进行配置，并且提供了内置的备份和缩放。可以在 [IBM Cloud“目录”](https://{DomainName}/catalog/?category=data)中找到许多不同类型的数据库。

### VM、容器和 Kubernetes

{{site.data.keyword.containershort_notm}} 提供了在 Kubernetes 集群中运行容器化应用程序的功能，并交付了以下工具和功能：

- 直观的用户体验和强大的工具
- 内置安全性和隔离，支持快速交付安全应用程序
- 包含来自 IBM® Watson™ 的认知能力的云服务
- 能够管理无状态应用程序和有状态工作负载的专用集群资源

#### 虚拟机与容器

**VM**，传统应用程序在本机硬件上运行。单个应用程序通常不使用单个计算主机的完整资源。大多数组织都尝试在单个计算主机上运行多个应用程序以避免浪费资源。您可以运行同一应用程序的多个副本，但要提供隔离，可以使用 VM 在相同的硬件上运行多个应用程序实例 (VM)。这些 VM 有完整的操作系统堆栈，由于在运行时和在磁盘上都有重复，所以它们相对较大并且效率低下。

**容器**是打包应用程序及其所有依赖项的标准方法，以便您可以无缝地在环境之间移动应用程序。与虚拟机不同，容器不会捆绑操作系统。容器仅会打包应用程序代码、运行时、系统工具、库和设置。容器比虚拟机更轻巧、可移植性更强、更高效。

此外，使用容器可以共享主机操作系统。这样就能在减少重复的情况下仍提供隔离。使用容器还可以丢弃不需要的文件（如系统库和二进制文件）以节省空间并减少攻击面。在[此处](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html)阅读有关虚拟机和容器的更多信息。

#### Kubernetes 编排

[Kubernetes](http://kubernetes.io/) 是一种容器编排器，可以在工作程序节点的集群中管理容器化应用程序的生命周期。您的应用程序可能需要其他许多资源才能运行，如卷、网络、可帮助您连接其他云服务的私钥，以及安全密钥。Kubernetes 帮助您将这些资源添加到应用程序。Kubernetes 的密钥范例是其声明式模型。用户提供所需的状态，Kubernetes 尝试满足描述的状态，然后保持该状态。

此[自定步调学习班](https://github.com/IBM/kube101/blob/master/workshop/README.md)可以帮助您获得使用 Kubernetes 的第一次实际经验。此外，请查看 Kubernetes [Concepts](https://kubernetes.io/docs/concepts/) 文档页面以了解有关 Kubernetes 概念的更多信息。

### IBM 能为您做什么

通过将 Kubernetes 集群与 {{site.data.keyword.containerlong_notm}} 一起使用，您可获得以下优点：

- 可在其中部署集群的多个数据中心。
- 对 Ingress 和负载均衡器联网选项的支持。
- 动态持久卷支持。
- IBM 管理的高可用性 Kubernetes 主节点。

## 调整集群大小
{: #sizing_clusters}

在设计集群体系结构时，您希望针对可用性、可靠性、复杂性和恢复来平衡成本。{{site.data.keyword.containerlong_notm}} 中的 Kubernetes 集群根据应用程序的需求来提供体系结构选项。只需进行少量规划，即可最充分地利用云资源，而不会过度设计或过度花费。即使您高估或低估了实际情况，也可以通过更多工作程序节点或更大的工作程序节点来轻松扩展或缩减集群。

要使用 Kubernetes 在云中运行生产应用程序，请考虑下列各项：

1. 您预期会有来自特定地理位置的流量吗？如果有，请选择实际离您最近的位置，以获得最佳性能。
2. 为了实现更高的可用性，需要有多少集群副本？比较合适的起点是三个集群，一个用于开发，一个用于测试，一个用于生产。查看有关创建多个环境的[组织用户、团队、应用程序的最佳实践](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments)解决方案指南。
3. 工作程序节点需要什么[硬件](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes)？虚拟机还是裸机？
4. 需要多少个工作程序节点？这主要取决于应用程序规模，您拥有的节点数量越多，应用程序的弹性越高。
5. 为了实现更高可用性，应该具备多少个副本？在多个位置部署副本集群以提高应用程序可用性，并防止应用程序因某个位置故障而停止运行。
6. 应用程序至少需要哪些资源才能启动？您可能要测试应用程序运行所需的内存量和 CPU。然后，您的工作程序节点应该有足够的资源来部署并启动应用程序。接着，作为 pod 规范的一部分，确保设置资源配额。Kubernetes 正是使用此设置来选择（或安排）有足够容量来支持请求的工作程序节点。估算有多少个 pod 会在工作程序节点上运行，以及这些 pod 的资源需求。工作程序节点至少要大到足以支持应用程序的一个 pod。
7. 何时增加工作程序节点数？您可以监视集群使用情况，并根据需要增加节点数。请参阅本教程以了解如何[分析日志并监视 Kubernetes 应用程序的运行状况](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana)。
8. 是否需要冗余的可靠存储？如果是，请创建 NFS 存储的持久卷声明，或者将 IBM Cloud 数据库服务绑定到 pod。

要使上述内容更具体，我们可以假定，您要在云中运行生产 Web 应用程序，并预期有中高水平的流量负载。让我们看一下您将需要哪些资源：

1. 设置三个集群，一个用于开发，一个用于测试，一个用于生产。
2. 开发和测试集群开始时可以使用最低 RAM 和 CPU 选项（例如，每个集群 2 个 CPU、4GB RAM 和一个工作程序节点）。
3. 对于生产集群，可能需要有更多资源才能保证性能、高可用性和弹性。我们可能选择专用甚至裸机选项，并有至少 4 个 CPU、16GB RAM 和两个工作程序节点。

## 确定要使用的数据库选项
{: #database_options}

利用 Kubernetes，您可以使用两个选项来处理数据库：

1. 您可以在 Kubernetes 集群中运行数据库，以执行创建微服务来运行数据库所需的操作。如果使用 MySQL 数据库示例，那么需要执行下列操作：
   - 创建 MySQL Dockerfile，请在此处查看示例 [MySQL Dockerfile](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile)。
   - 您需要使用私钥来存储数据库凭证。请在[此处](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret)查看相应的示例。
   - 您需要有 deployment.yaml 文件，其中包含要部署到 Kubernetes 的数据库的配置。请在[此处](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml)查看相应的示例。
2. 第二个选项是使用受管的数据库即服务 (DBasS) 选项。此选项通常更容易配置，并提供内置备份和缩放功能。可以在 [IBM Cloud“目录”](https://{DomainName}/catalog/?category=data)中找到许多不同类型的数据库。要使用此选项，需要执行下列操作：
   - 从 [IBM Cloud“目录”](https://{DomainName}/catalog/?category=data)创建受管数据库即服务 (DBasS)。
   - 将数据库凭证存储在私钥中。在“在 Kubernetes 私钥中存储凭证”部分中，您将了解有关私钥的更多信息。
   - 在应用程序中使用数据库即服务 (DBasS)。

## 确定存储应用程序文件的位置
{: #decide_where_to_store_data}

{{site.data.keyword.containershort_notm}} 提供了用于在不同 pod 中存储和共享数据的几个选项。在灾难情况下，并非所有存储选项提供的持久性和可用性级别都是相同的。
{: shortdesc}

### 非持久性数据存储

根据设计，容器和 pod 的生存时间短，并且可能会意外发生故障。您可以在容器的本地文件系统中存储数据。容器内的数据不能与其他容器或 pod 共享，并且在容器崩溃或被除去时会丢失。

### 了解如何为应用程序创建持久数据存储

您可以使用本机 Kubernetes 持久卷在 [NFS 文件存储器](https://www.ibm.com/cloud/file-storage/details)或[块存储器](https://www.ibm.com/cloud/block-storage)中持久存储应用程序数据和容器数据。
{: shortdesc}

要供应 NFS 文件存储器或块存储器，必须通过创建持久卷声明 (PVC) 来为 pod 请求存储器。在 PVC 中，可以从预定义的存储类中进行选择，以用于定义存储器类型、存储器大小（以千兆字节为单位）、IOPS、数据保留时间策略以及对存储器的读和写许可权。PVC 动态供应持久卷 (PV)，用于表示 {{site.data.keyword.Bluemix_notm}} 中的实际存储设备。您可以将 PVC 安装到 pod 中以在 PV 中进行读写。存储在 PV 中的数据即使在容器崩溃或者 pod 重新安排的情况下也是可用的。支持 PV 的 NFS 文件存储器和块存储器由 IBM 建立集群，以便为数据提供高可用性。

要了解如何创建 PVC，请按照 [{{site.data.keyword.containershort_notm}} 存储文档](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage)中涵盖的步骤执行操作。

### 了解如何将现有数据移动到持久存储

要将数据从本地计算机复制到持久存储中，必须将 PVC 安装到 pod。然后可以将数据从本地计算机复制到 pod 中的持久卷。
{: shortdesc}

1. 要复制数据，需要先创建类似于以下内容的配置：

   ```bash
   kind: Pod
   apiVersion: v1
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
          claimName: mypvc
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/mnt/data"
             name: task-pv-storage
   ```
   {: codeblock}

2. 然后，要将数据从本地计算机复制到 pod，需要使用如下命令：
   ```
    kubectl cp <local_filepath>/<filename> <namespace>/<pod>:<pod_filepath>
   ```
   {: pre}

3. 将数据从集群中的 pod 复制到本地计算机：
   ```
   kubectl cp <namespace>/<pod>:<pod_filepath>/<filename> <local_filepath>/<filename>
   ```
   {: pre}


### 为持久存储设置备份

文件共享和块存储器会供应到集群所在的位置。存储器本身由 IBM 在集群服务器上托管以提供高可用性。但是，文件共享和块存储器不会自动进行备份，因此它们在整个位置发生故障时可能无法进行访问。为了防止数据丢失或损坏，可以设置定期备份，以便在需要时可用于复原数据。


有关更多信息，请参阅有关 NFS 文件存储器和块存储器的[备份和复原](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning)选项。

## 准备代码
{: #prepare_code}

### 应用 12 因子原则

[十二因子应用程序](https://12factor.net/)是用于构建云本机应用程序的方法。当您要将应用程序容器化，将此应用程序移动到云，以及使用 Kubernetes 编排应用程序时，请务必理解并应用其中一些原则。{{site.data.keyword.Bluemix_notm}} 中要求使用其中一些原则。
{: shortdesc}

下面是需要的一些关键原则：

- **代码库** - 在版本控制系统（如 Git 存储库）中跟踪所有源代码和配置文件，这在使用 DevOps 管道进行部署时是必需的。
- **构建、发布、运行** - 12 因子应用程序在构建、发布和运行阶段之间使用严格分离。这可以使用集成的 DevOps Delivery Pipeline 自动执行，以在将应用程序部署到集群之前先构建和测试应用程序。请查看[持续部署到 Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) 教程，以了解如何设置持续集成和 Delivery Pipeline。其中涵盖源代码控制、构建、测试和部署阶段的设置，并展示了如何添加安全性扫描程序、通知和分析等集成。
- **配置** - 所有配置信息都存储在环境变量中。任何服务凭证都不会在应用程序代码中进行硬编码。要存储凭证，可以使用 Kubernetes 私钥。下面介绍了有关凭证的更多信息。

### 将凭证存储在 Kubernetes 私钥中
{: secrets}

将凭证存储在应用程序代码中绝不是一个好的做法。相反，Kubernetes 提供了所谓的**[“私钥”](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)**，用于容纳敏感信息，如密码、OAuth 令牌或 SSH 密钥。Kubernetes 私钥在缺省情况下已加密，相较于将敏感数据逐字存储在 `pod` 定义或 Docker 映像中，使用 Kubernetes 私钥来存储此数据更安全，也更灵活。

在 Kubernetes 中使用私钥的一种方式如下：

1. 创建文件并将服务凭证存储在其中。
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api_key>
   }
   ```
   {: codeblock}

2. 然后，运行以下命令以创建 Kubernetes 私钥，并在运行以下命令之后使用 `kubectl get secrets` 验证是否已创建该私钥：

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## 将应用程序容器化
{: #build_docker_images}

要将应用程序容器化，必须创建 Docker 映像。
{: shortdesc}

映像是从 [Dockerfile](https://docs.docker.com/engine/reference/builder/#usage) 创建的，该文件中包含用于构建映像的指示信息和命令。Dockerfile 可能会在其单独存储的指令中引用构建工件，例如应用程序、应用程序配置及其依赖项。


要为现有应用程序构建自己的 Dockerfile，可使用以下命令：

- FROM - 选择用于定义容器运行时的父映像。
- ADD/COPY - 将目录内容复制到容器中。
- WORKDIR - 在容器中设置工作目录。
- RUN - 安装应用程序在运行时期间需要的软件包。
- EXPOSE - 使一个端口在容器外部可用。
- ENV NAME - 定义环境变量。
- CMD - 定义在容器启动时运行的命令。

映像通常存储在可由公共（公共注册表）访问的注册表中，或者存储在针对一小组用户设置有限访问权（专用注册表）的注册表中。
开始使用 Docker 和 Kubernetes 在集群中创建第一个容器化应用程序时，可以使用公共注册表（如 Docker Hub）。但是对于企业应用程序，请使用专用注册表（如在 {{site.data.keyword.registrylong_notm}} 中提供的注册表），以保护映像不被未经授权的用户使用和更改。

要将应用程序容器化并将其存储在 {{site.data.keyword.registrylong_notm}} 中，请执行以下操作：

1. 您需要创建 Dockerfile，下面是 Dockerfile 的示例。
   ```
   # Build JPetStore war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # Use WebSphere Liberty base image from the Docker Store
   FROM websphere-liberty:latest

   # Copy war from build stage and server.xml into image
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. 创建 Dockerfile 之后，接着需要构建容器映像并将其推送到 {{site.data.keyword.registrylong_notm}}。您可以使用以下命令构建容器：
   ```
   ibmcloud cr build -t <image_name> <directory_of_Dockerfile>
   ```
   {: pre}

## 将应用程序部署到 Kubernetes 集群
{: #deploy_to_kubernetes}

在构建容器映像并将其推送到云之后，接下来需要将其部署到 Kubernetes 集群。要执行该操作，需要创建 deployment.yaml 文件。
{: shortdesc}

### 了解如何创建 Kubernetes deployment.yaml 文件

要创建 Kubernetes deployment.yaml 文件，需要执行以下操作：

1. 创建 deployment.yaml 文件，下面是 [deployment YAML](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml) 文件的示例。

2. 在 deployment.yaml 文件中，可以定义容器的[资源配额](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)以指定每个容器正常启动需要多少 CPU 和内存。如果容器已指定资源配额，那么 Kubernetes 调度程序在决定将 pod 放置在哪个工作程序节点上时，可以做出更好的选择。

3. 接下来，您可以使用下面的命令来创建并查看已创建的部署和服务：

   ```
   kubectl create -f <filepath/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## 摘要
{: #summary}

在本教程中，您学到了以下内容：

- VM、容器和 Kubernetes 之间的差别。
- 如何针对不同的环境类型（开发、测试、生产）定义集群。
- 如何处理数据存储以及持久数据存储的重要性。
- 对应用程序应用 12 因子原则并使用私钥作为 Kubernetes 中的凭证。
- 构建 Docker 映像并将其推送到 {{site.data.keyword.registrylong_notm}}。
- 创建 Kubernetes 部署文件并将 Docker 映像部署到 Kubernetes。

## 将学到的所有做法付诸实践，在集群中运行 JPetStore 应用程序。
{: #runthejpetstore}

将学到的所有做法付诸实践，按照[演示](https://github.com/ibm-cloud/ModernizeDemo/)在集群上运行 **JPetStore** 应用程序并应用所学的概念。JPetStore 应用程序有一些扩展的功能，您可以利用这些功能让 IBM Watson 服务作为单独的微服务运行以在 Kubernetes 中扩展应用程序。

## 相关内容
{: #related}

- Kubernetes 和 {{site.data.keyword.containershort_notm}} [入门](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/)
- [GitHub](https://github.com/IBM/container-service-getting-started-wt) 上的 {{site.data.keyword.containershort_notm}} 实验室。
- Kubernetes 主[文档](http://kubernetes.io/)。
- {{site.data.keyword.containershort_notm}} 中的[持久存储](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning)。
- 用于组织用户、团队和应用程序的[最佳实践解决方案指南](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)。
- [使用 LogDNA 和 Sysdig 分析日志并监视应用程序运行状况](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)。
- 为 Kubernetes 中运行的容器化应用程序设置[持续集成和 Delivery Pipeline](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)。
- [跨多个位置](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)部署生产集群。
- 使用[跨多个位置的多个集群](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations)以实现高可用性。
