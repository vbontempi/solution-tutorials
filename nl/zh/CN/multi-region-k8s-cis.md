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

# 使用 Cloud Internet Services 的富有弹性且安全的多区域 Kubernetes 集群
{: #multi-region-k8s-cis}

在设计应用程序时若能考虑到弹性，用户就不太可能会遇到停机时间。使用 {{site.data.keyword.containershort_notm}} 实施解决方案时，可以利用各种内置功能（如负载均衡和隔离），提高应对主机、网络或应用程序潜在故障的弹性。通过创建多个集群，如果一个集群发生中断，用户仍然可以访问同时部署在另一个集群中的应用程序。利用位于不同位置的多个集群，用户还可以访问距离最近的集群，并缩短网络等待时间。为了获得更多弹性，还可以选择多专区集群，这意味着节点部署在一个位置的多个专区中。

Cloud Internet Services (CIS) 是一个统一平台，用于为因特网应用程序配置和管理域名系统 (DNS)、全局负载均衡 (GLB)、Web 应用程序防火墙 (WAF) 以及防御分布式拒绝服务 (DDoS)。本教程重点阐述了 CIS 如何与 Kubernetes 集群集成以支持此场景，并在多个位置提供安全、富有弹性的解决方案。

## 目标
{: #objectives}

* 在不同位置的多个 Kubernetes 集群上部署一个应用程序。
* 使用全局负载均衡器在多个集群之间分配流量。
* 将用户路由到距离最近的集群。
* 保护应用程序免受安全威胁。
* 通过高速缓存提高应用程序性能。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

<p style="text-align: center;">
  ![体系结构](images/solution32-multi-region-k8s-cis/Architecture.png)
</p>

1. 开发者为应用程序构建 Docker 映像。
2. 将映像推送到达拉斯和伦敦的 {{site.data.keyword.registryshort_notm}}。
3. 将应用程序部署到这两个位置的 Kubernetes 集群。
4. 最终用户访问该应用程序。
5. Cloud Internet Services 配置为拦截对应用程序发出的请求并在集群之间分发负载。此外，还启用了 DDoS 保护和 Web 应用程序防火墙，以保护应用程序免受常见威胁。（可选）对图像、CSS 文件等资产进行高速缓存。

## 开始之前
{: #prereqs}

* Cloud Internet Services 需要您拥有定制域，以便可将此域的 DNS 配置为指向 Cloud Internet Services 名称服务器。
* [安装 Git](https://git-scm.com/)。
* [安装 {{site.data.keyword.Bluemix_notm}} CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)。
* [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - 通过脚本安装 docker、kubectl、helm、ibmcloud cli 和必需的插件。
* [设置 {{site.data.keyword.registrylong_notm}} CLI 和注册表名称空间](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)。
* [学习 Kubernetes 基础知识](https://kubernetes.io/docs/tutorials/kubernetes-basics/)。

## 将应用程序部署到一个位置

本教程将一个 Kubernetes 应用程序部署到多个位置的集群。首先部署到一个位置 - 达拉斯，然后对伦敦位置重复这些步骤。

### 创建 Kubernetes 集群
{: #create_cluster}

要创建集群，请执行以下操作：
1. 从 [{{site.data.keyword.cloud_notm}}“目录”](https://{DomainName}/containers-kubernetes/catalog/cluster/create)中，选择 **{{site.data.keyword.containershort_notm}}**。
1. 将**位置**设置为**达拉斯**。
1. 选择**标准**集群。
1. 选择一个或多个专区作为**位置**。创建多专区集群可提高应用程序弹性。应用程序分布在多个专区时，用户不太可能会遇到停机时间。有关多专区域集群的更多信息可以在[此处](https://{DomainName}/docs/containers?topic=containers-plan_clusters#ha_clusters)找到。
1. 将**机器类型**设置为可用的最小配置 - **2 个 CPU** 和 **4 GB RAM** 足以满足本教程的需求。
1. 使用 **2** 个工作程序节点。
1. 将**集群名称**设置为 **my-us-cluster**。请使用命名模式 *`my-<location>-cluster`*，以便与本教程保持一致。

集群准备就绪后，接着将准备应用程序。

### 在 {{site.data.keyword.registryshort_notm}} 中创建名称空间

1. 将 {{site.data.keyword.Bluemix_notm}} CLI 的目标设定为达拉斯。
   ```bash
   ibmcloud target -r us-south
   ```
   {: pre}
2. 为应用程序创建名称空间。
   ```bash
   ibmcloud cr namespace-add <your_namespace>
   ```
   {: pre}

如果在该位置具有现有名称空间，那么还可以复用该名称空间。可以使用 `ibmcloud cr namespaces` 来列出现有名称空间。
{: tip}

### 构建应用程序

此步骤将应用程序构建成 Docker 映像。如果要配置第二个集群，那么可以跳过此步骤。这是简单的 HelloWorld 应用程序。

1. 将 [Hello world 应用程序](https://github.com/IBM/container-service-getting-started-wt){:new_windows}的源代码克隆到用户主目录。存储库在以 Lab 开头的各个文件夹中分别包含类似应用程序的不同版本。
   ```bash
    git clone https://github.com/IBM/container-service-getting-started-wt.git
    ```
   {: pre}
1. 浏览到 `Lab 1` 目录。
   ```bash
    cd 'container-service-getting-started-wt/Lab 1'
    ```
   {: pre}
1. 构建包含 `Lab 1` 目录中应用程序文件的 Docker 映像。
   ```bash
   docker build --tag multi-region-hello-world:1 .
   ```
   {: pre}

### 准备将映像推送到特定于位置的注册表

使用目标注册表标记映像：

   ```bash
   docker tag multi-region-hello-world:1 us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### 将映像推送到特定于位置的注册表

1. 确保本地 Docker 引擎可以推送到达拉斯的注册表。
   ```bash
    ibmcloud cr login
    ```
   {: pre}
2. 推送映像。
   ```bash
   docker push us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### 将应用程序部署到 Kubernetes 集群

在该阶段，集群应该已准备就绪。可以在 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/clusters) 控制台中查看其状态。

1. 检索集群的配置：
   ```bash
   ibmcloud cs cluster-config <us-cluster-name>
   ```
   {: pre}
1. 复制并粘贴输出，以设置 KUBECONFIG 环境变量。该变量由 `kubectl` 使用。
1. 使用两个副本在集群中运行应用程序：
   ```bash
   kubectl run hello-world-deployment --image=us.icr.io/<your_US-South_namespace>/hello-world:1 --replicas=2
   ```
   {: pre}
   示例输出：`deployment "hello-world-deployment" created`。
1. 使应用程序在集群中可访问
   ```bash
   kubectl expose deployment/hello-world-deployment --type=ClusterIP --port=80 --name=hello-world-service --target-port=8080
   ```
   {: pre}
   这将返回类似于 `service "hello-world-service" exposed` 的消息。

### 获取分配给集群的域名和 IP 地址
{: #CSALB_IP_subdomain}

创建 Kubernetes 集群时，会为其分配 Ingress 子域（例如，*my-us-cluster.us-south.containers.appdomain.cloud*）和公共应用程序负载均衡器 IP 地址。

1. 检索集群的 Ingress 子域：
   ```bash
   ibmcloud cs cluster-get <us-cluster-name>
   ```
   {: pre}
   查找 `Ingress Subdomain` 值。
1. 记下此信息以在稍后步骤中使用。

本教程使用 Ingress 子域来配置全局负载均衡器。还可以将该子域替换为公共应用程序负载均衡器 IP 地址 (`ibmcloud cs albs -cluster <us-cluster-name>`)。这两个选项都受支持。
{: tip}

## 然后对另一个位置执行操作

在伦敦位置重复上述步骤，其中：
* 将位置名**达拉斯**替换为**伦敦**；
* 将位置别名 **us-south** 替换为 **eu-gb**；
* 将注册表 *us.icr.io* 替换为 **uk.icr.io**；
* 将集群名称 *my-us-cluster* 替换为 **my-uk-cluster**。

## 配置多位置负载均衡

现在，应用程序在两个集群中运行，但缺少一个组件，供用户从单个入口点透明地访问任一集群。

在此部分中，您将配置 Cloud Internet Services (CIS) 以在两个集群之间分发负载。CIS 是一个一站式服务，提供全局负载均衡器 (GLB)、高速缓存、Web 应用程序防火墙 (WAF) 和页面规则来保护应用程序，同时确保云应用程序的可靠性和性能。

要配置全局负载均衡器，您将需要：
* 将定制域指向 CIS 名称服务器，
* 检索 Kubernetes 集群的 IP 地址或子域名，
* 配置运行状况检查，以验证应用程序的可用性，
* 以及定义指向集群的源池。

### 向 Cloud Internet Services 注册定制域
{: #create_cis_instance}

第一步是创建 CIS 实例，并将定制域指向 CIS 名称服务器。

1. 如果您没有域，可以向 [godaddy.com](http://godaddy.com) 等注册商购买域。
2. 导航至 {{site.data.keyword.Bluemix_notm}}“目录”中的 [Internet Services](https://{DomainName}/catalog/services/internet-services)。
3. 设置服务名称，然后单击**创建**以创建服务实例。
4. 供应服务实例后，设置域名，然后单击**添加域**。
5. 分配名称服务器后，配置注册商或域名提供商，以使用列出的名称服务器。
6. 配置注册商或 DNS 提供商后，最长可能需要 24 小时，更改才会生效。

   在“概述”页面上，域的状态从*暂挂*更改为*活动*时，可以使用 `dig <your_domain_name> ns` 命令来验证新的名称服务器是否已生效。
   {:tip}

### 为全局负载均衡器配置运行状况检查

运行状况检查有助于深入了解池的可用性，以便可以将流量路由到运行正常的池。这些检查定期发送 HTTP/HTTPS 请求并监视响应。

1. 在 Cloud Internet Services 仪表板中，导航至**可靠性** > **全局负载均衡器**，然后单击页面底部的**创建运行状况检查**。
1. 将**路径**设置为 **/**。
1. 将**监视类型**设置为 **HTTP**。
1. 单击**供应 1 个 实例**。

   构建您自己的应用程序时，可以定义一个专用的运行状况端点，例如 */heathz*，在其中可以报告应用程序状态。
   {:tip}

### 定义源池

池是一组源服务器，在池连接到 GLB 时，会将流量智能路由到池中。在英国和美国使用集群时，可以定义基于位置的池，并配置 CIS 以根据用户请求的地理位置，将用户重定向到距离最近的集群。

#### 一个池用于伦敦的集群
1. 单击**创建池**。
2. 将**名称**设置为 **UK**
3. 将**运行状况检查**设置为在先前部分中创建的检查
4. 将**运行状况检查区域**设置为**西欧**
5. 将**源名称**设置为 **uk-cluster**
6. 将**源地址**设置为伦敦集群的 Ingress 子域，例如 *my_uk_cluster.eu-gb.containers.appdomain.cloud*
7. 单击**供应 1 个 实例**。

#### 一个池用于达拉斯的集群
1. 单击**创建池**。
2. 将**名称**设置为 **US**
3. 将**运行状况检查**设置为在先前部分中创建的检查
4. 将**运行状况检查区域**设置为**北美洲西部地区**
5. 将**源名称**设置为 **us-cluster**
6. 将**源地址**设置为达拉斯集群的 Ingress 子域，例如 *my_us_cluster.us-south.containers.appdomain.cloud*
7. 单击**供应 1 个 实例**。

#### 一个池用于这两个集群
1. 单击**创建池**。
1. 将**名称**设置为 **All**
1. 将**运行状况检查**设置为在先前部分中创建的检查
1. 将**运行状况检查区域**设置为**北美洲东部地区**
1. 添加两个源：
   1. 一个源的**源名**设置为 **us-cluster**，**源地址**设置为达拉斯集群的 Ingress 子域
   2. 一个源的**源名**设置为 **uk-cluster**，**源地址**设置为伦敦集群的 Ingress 子域
2. 单击**供应 1 个 实例**。

### 创建全局负载均衡器

定义源池后，可以完成负载均衡器的配置。

1. 单击**创建负载均衡器**。
1. 在全局负载均衡器的**均衡器主机名**下输入名称。此名称还将是通用应用程序 URL (`http://<glb_name>.<your_domain_name>`) 的一部分，不管是哪个位置。
1. 在**缺省源池**下，单击**添加池**，然后添加名为 **All** 的池。
1. 展开**配置地理路径（可选）**的部分：
   1. 单击**添加路径**，选择**西欧**，然后单击**添加**。
   1. 单击**添加池**以选择 **UK** 池。
   1. 配置如下表中所示的其他路径。
   1. 单击**供应 1 个 实例**。

|区域|源池|
| :---------------:    | :---------: |
|西欧|UK|
|东欧|UK|
|东北亚|UK|
|东南亚|UK|
|北美洲西部地区|US|
|北美洲东部地区|US|

通过此配置，欧洲和亚洲的用户将重定向到伦敦的集群，美国的用户将重定向到达拉斯集群。请求与任何定义的路径都不匹配时，将重定向到**缺省源池**。

### 为每个位置的 Kubernetes 集群创建 Ingress 资源

全局负载均衡器现在已准备好处理请求。所有运行状况检查都应该为绿色。但是，Kubernetes 集群上需要最后一个配置步骤，才能正确回复来自全局负载均衡器的请求：需要定义一个 Ingress 资源，用于处理来自 GLB 域的请求。

1. 创建名为 **glb-ingress.yaml** 的 Ingress 资源文件
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: glb-ingress
   spec:
    rules:
      - host: <glb_name>.<your_domain_name>
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-world-service
              servicePort: 80
   ```
    {: pre}
    将 `<glb_name>.<your_domain_name>` 替换为在先前部署中定义的 URL。
1. 在为相应的位置集群设置 KUBECONFIG 变量之后，在伦敦和达拉斯集群中部署此资源：
   ```bash
   kubectl create -f glb-ingress.yaml
   ```
   {: pre}
   输出的消息为：`ingress.extension "glb-ingress" created`。

在此阶段，已成功配置全局负载均衡器用于多个位置中的 Kubernetes 集群。可以访问 GLB URL `http://<glb_name>.<your_domain_name>` 来查看应用程序。根据您的位置，会将您重定向到距离最近的集群 - 如果 CIS 无法将您的 IP 地址映射到特定位置，那么会将您重定向到缺省池中的集群。

## 保护应用程序
{: #secure_via_CIS}

### 开启 Web 应用程序防火墙

Web 应用程序防火墙 (WAF) 可保护 Web 应用程序免受 ISO 第 7 层攻击。通常，WAF 与分组规则集组合使用，这些规则集旨在过滤掉恶意流量，从而避免应用程序中有漏洞。

1. 在 Cloud Internet Services 仪表板中，导航至**安全性**，然后单击**管理**选项卡。
1. 在 **Web 应用程序防火墙**部分中，确保 WAF 已启用。
1. 单击**查看 OWASP 规则集**。在此页面中，可以查看 **OWASP 核心规则集**，并分别启用或禁用规则。规则启用后，如果入局请求触发了该规则，那么全局威胁分数将增大。**敏感度**设置将决定是否针对该请求触发**操作**。
   1. 保留缺省 OWASP 规则集不变。
   1. 将**敏感度**设置为`低`。
   1. 将**操作**设置为`模拟`以记录所有事件。
1. 单击**返回到安全性**。
1. 单击**查看 CIS 规则集**。此页面显示基于托管 Web 站点的常见技术堆栈构建的其他规则。

### 提高性能并防御拒绝服务攻击 
{: #proxy_setting}

分布式拒绝服务 ([DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)) 攻击是指通过在目标或其周围基础设施产生大量因特网流量，恶意尝试中断服务器、服务或网络的正常通信。CIS 可用于保护域免受 DDoS 攻击。

1. 在 CIS 仪表板中，选择**可靠性** > **全局负载均衡器**。
1. 在**负载均衡器**表中找到创建的 GLB。
1. 启用**代理**列中的“安全性和性能”功能：

   ![CIS 代理切换控件开启](images/solution32-multi-region-k8s-cis/cis-proxy.png)

**GLB 现在受到保护**。一个直接的好处就是，集群的源 IP 地址将对客户机隐藏。如果 CIS 检测到即将进行的请求存在威胁，那么在将用户重定向到应用程序之前，用户可能会看到类似下面的屏幕：

   ![正在验证 - DDoS 防御](images/solution32-multi-region-k8s-cis/cis-DDoS.png)

此外，现在可以控制 CIS 高速缓存的内容以及保持高速缓存的时间。转至**性能** > **高速缓存**以定义全局缓高速缓存级别和浏览器到期时间。可以使用**页面规则**来定制全局安全性和高速缓存规则。页面规则使用特定域路径来启用细颗粒度配置。例如，通过页面规则，可以决定将 **/assets** 下的所有内容高速缓存 **3 天**：

   ![页面规则](images/solution32-multi-region-k8s-cis/cis-pagerules.png)

## 除去资源
{:removeresources}

### 除去 Kubernetes 集群资源
1. 除去 Ingress。
1. 除去服务。
1. 除去部署。
1. 如果专门为此教程创建了集群，那么删除这些集群。

### 除去 CIS 资源
1. 除去 GLB。
1. 除去源池。
1. 除去运行状况检查。

## 相关内容
{:related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [管理 IBM CIS 以获取最佳安全性](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#best-practice-2-configure-your-security-level-selectively)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
* [{{site.data.keyword.registrylong_notm}} 基础知识](https://{DomainName}/docs/services/Registry?topic=registry-registry_overview#registry_planning)
* [将单实例应用程序部署到 Kubernetes 集群](https://{DomainName}/docs/containers?topic=containers-cs_apps_tutorial#cs_apps_tutorial_lesson1)
* [通过 CIS 保护流量和因特网应用程序的最佳实践](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#manage-your-ibm-cis-for-optimal-security)
* [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
