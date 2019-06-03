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

# Kubernetes 上的可缩放 Web 应用程序
{: #scalable-webapp-kubernetes}

本教程将全程指导您如何搭建 Web 应用程序脚手架，在容器中本地运行 Web 应用程序，然后将其部署到使用 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster) 创建的 Kubernetes 集群。此外，你将学习如何绑定定制域、监视环境的运行状况和缩放应用程序。
{:shortdesc}

容器是打包应用程序及其所有依赖项的标准方法，以便您可以无缝地在环境之间移动应用程序。与虚拟机不同，容器不会捆绑操作系统。在容器中只会打包应用程序代码、运行时、系统工具、库和设置。容器比虚拟机更轻巧、可移植性更强、更高效。

对于希望启动自己项目的开发者，{{site.data.keyword.dev_cli_notm}} CLI 通过生成模板应用程序，以供您立即运行或定制为自己的解决方案的入门模板，支持快速开发和部署应用程序。除了生成入门模板应用程序代码、Docker 容器映像和 CloudFoundry 资产外，开发 CLI 和 Web 控制台使用的代码生成器还可生成文件，以帮助部署到 [Kubernetes](https://kubernetes.io/) 环境中。这些模板生成 [Helm](https://github.com/kubernetes/helm) chart，用于描述应用程序初始 Kubernetes 部署配置，并可根据需要轻松扩展以创建多映像或复杂部署。

## 目标
{: #objectives}

* 搭建入门模板应用程序脚手架。
* 将应用程序部署到 Kubernetes 集群。
* 绑定定制域。
* 监视集群的日志和运行状况。
* 缩放 Kubernetes pod。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

<p style="text-align: center;">

  ![体系结构](images/solution2/Architecture.png)
</p>

1. 开发者使用 {{site.data.keyword.dev_cli_notm}} 生成入门模板应用程序。
1. 通过构建应用程序，生成 Docker 容器映像。
1. 映像被推送到 {{site.data.keyword.containershort_notm}} 中的名称空间。
1. 应用程序部署到 Kubernetes 集群。
1. 用户访问该应用程序。

## 开始之前
{: #prereqs}

* [设置 {{site.data.keyword.registrylong_notm}} CLI 和注册表名称空间](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [安装 {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - 用于安装 docker、kubectl、helm、ibmcloud cli 和必需插件的脚本
* [学习 Kubernetes 基础知识](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## 创建 Kubernetes 集群
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} 通过组合 Docker 和 Kubernetes 技术、直观的用户体验以及内置安全性和隔离，提供功能强大的工具来自动对计算主机集群中的容器化应用程序进行部署、操作、缩放和监视。

本教程的主要部分可以使用**免费**集群来完成。与 Kubernetes Ingress 和定制域相关的两个可选部分需要**标准**类型的**付费**集群。

1. 通过 [{{site.data.keyword.Bluemix}}“目录”](https://{DomainName}/containers-kubernetes/launch)，创建 Kubernetes 集群。

   为了方便使用，请检查配置详细信息，如使用轻量套餐和标准套餐时获得的 CPU 数、内存和工作程序节点数。
   {:tip}

   ![在 IBM Cloud 上创建 Kubernetes 集群](images/solution2/KubernetesClusterCreation.png)
2. 选择**集群类型**，然后单击**创建集群**以供应 Kubernetes 集群。
3.  检查**集群**和**工作程序节点**的状态，并等待它们变为**准备就绪**。

### 配置 kubectl

在此步骤中，您将配置 kubectl 以指向新创建的集群。[kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) 是一个命令行工具，用于与 Kubernetes 集群进行交互。

1. 使用 `ibmcloud login` 以交互方式登录。提供在其中创建集群的组织、位置和空间。可以通过运行 `ibmcloud target` 命令重新确认详细信息。
2. 集群准备就绪后，通过将 MYCLUSTER 环境变量设置为您的集群名称来检索集群配置：
   ```bash
   export MYCLUSTER=<CLUSTER_NAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
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


## 创建入门模板应用程序
{: #create_application}

`ibmcloud dev` 工具通过生成具有所有必需的样板、构建和配置代码的应用程序入门模板，大大缩短了开发时间，从而可以更快开始对业务逻辑进行编码。

1. 启动 `ibmcloud dev` 向导。
   ```
ibmcloud dev create
```
   {: pre}

1. 选择`后端服务 / Web 应用程序` > `Java - MicroProfile / JavaEE` > `Java Web App with Eclipse MicroProfile and Java EE（Web 应用程序）`来创建 Java 入门模板。（要改为创建 Node.js 入门模板，请使用`后端服务 / Web 应用程序` > `Node`> `Node.js Web App with Express.js（Web 应用程序）`）
1. 输入应用程序的**名称**。
1. 选择要在其中部署此应用程序的资源组。
1. 不要添加其他服务。
1. 不要添加 DevOps 工具链，请选择**手动部署**。

这将生成一个入门模板应用程序，其中包含代码和所有必要的配置文件，用于在 Cloud Foundry 或 Kubernetes 上进行本地开发和部署到云。 

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### 构建应用程序

您可以像通常使用 `mvn` 进行 Java 本地开发或使用 `npm` 进行节点开发一样来构建和运行应用程序。此外，还可以构建 Docker 映像并在容器中运行应用程序，以确保在本地和在云上的执行一致。使用以下步骤构建 Docker 映像。


1. 确保本地 Docker 引擎已启动。
   ```
   docker ps
   ```
   {: pre}
2. 切换到生成的项目目录。
   ```
   cd <project name>
   ```
   {: pre}
3. 构建应用程序。
   ```
ibmcloud dev build
```
   {: pre}

   这可能需要几分钟才能运行，因为将下载所有应用程序依赖项，并且会构建包含应用程序和所有必需环境的 Docker 映像。

### 在本地运行应用程序

1. 运行容器。
   ```
ibmcloud dev run
```
   {: pre}

   这将使用本地 Docker 引擎来运行在上一步中构建的 Docker 映像。
2. 容器启动后，转至 `http://localhost:9080/`。如果创建的是 Node.js 应用程序，请转至 `http://localhost:3000/`。
  ![](images/solution2/LibertyLocal.png)

## 使用 Helm chart 将应用程序部署到集群
{: #deploy}

在此部分中，首先将 Docker 映像推送到 IBM Cloud 专用容器注册表，然后创建指向该映像的 Kubernetes 部署。

1. 通过列出注册表中的所有名称空间来找到您的**名称空间**。
   ```sh
ibmcloud cr namespaces
```
   {: pre}
   如果您有名称空间，请记下其名称以供稍后使用。如果没有，请创建名称空间。
   ```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. 将 MYNAMESPACE 和 MYPROJECT 环境变量分别设置为您的名称空间和项目名称。

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. 通过运行 `ibmcloud cr info` 来识别您的**容器注册表**（例如，us.icr.io）。
4. 将 MYREGISTRY 环境变量设置为您的注册表。
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. 构建 Docker 映像并对其进行标记 (`-t`)。
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. 将 Docker 映像推送到 IBM Cloud 上的容器注册表。
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. 在 IDE 上，导航至 `chart\YOUR PROJECT NAME` 下的 **values.yaml**，并更新**映像存储库**值，以指向 IBM Cloud 容器注册表上您的映像。**保存**该文件。

   要获取映像存储库详细信息，请运行 `echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}`。

8. [Helm](https://helm.sh/) 通过 Helm chart 帮助管理 Kubernetes 应用程序，它可帮助定义、安装和升级最复杂的 Kubernetes 应用程序。通过导航至 `chart\YOUR PROJECT NAME`，并在集群中运行以下命令来初始化 Helm：

   ```bash
   helm init
   ```
   {: pre}
   要升级 Helm，请运行以下命令：`helm init --upgrade`
   {:tip}

9. 要安装 Helm chart，请切换到 `chart\YOUR PROJECT NAME` 目录并运行以下命令：
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. 使用 `kubectl get service ${MYPROJECT}-service`（对于 Java 应用程序）和 `kubectl get service ${MYPROJECT}-application-service`（对于 Node.js 应用程序）来识别服务在侦听的公共端口。端口为 5 位数（例如，31569），位于 `PORT(S)` 下。
11. 要获取工作程序节点的公共 IP，请运行以下命令：
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. 在 `http://worker-ip-address:portnumber/` 上访问应用程序。

## 将 IBM 提供的域用于集群
{: #ibm_domain}

在先前的步骤中，应用程序是使用非标准端口进行访问的。服务通过 Kubernetes NodePort 功能公开。

付费集群随附 IBM 提供的域。这为您提供了一个更好的选择，可以使用正确的 URL 在标准 HTTP/S 端口上公开应用程序。

使用 Ingress 设置集群到服务的入站连接。

![Ingress](images/solution2/Ingress.png)

1. 识别 IBM 提供的 **Ingress 域**
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   以找到
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. 创建 Ingress 文件 `ingress-ibmdomain.yml`，以指向您的支持 HTTP 和 HTTPS 的域。使用以下文件作为模板，并将所有用 <> 括起的值替换为上面输出中的相应值。**service-name** 是上面步骤中 `==> v1/Service` 下的名称，或者运行 `kubectl get svc` 来查找类型为 **NodePort** 的服务名称。
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-ibmdomain-http-and-https
   spec:
     tls:
     - hosts:
       -  <nameofproject>.<ingress-sub-domain>
       secretName: <ingress-secret>
     rules:
     - host: <nameofproject>.<ingress-sub-domain>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: codeblock}
3. 部署 Ingress
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. 在 `https://<nameofproject>.<ingress-sub-domain>/` 上访问应用程序

## 使用您自己的定制域
{: #custom_domain}

要使用定制域，需要使用指向 IBM 提供的域的 CNAME 记录，或指向 IBM 提供的 Ingress 的可移植公共 IP 地址的 A 记录来更新 DNS 记录。考虑到付费集群随附固定 IP 地址，因此 A 记录是一个不错的选择。

有关更多信息，请参阅[将 Ingress 控制器用于定制域](https://{DomainName}/docs/containers?topic=containers-ingress#ingress)。

### 使用 HTTP

1. 创建 Ingress 文件 `ingress-customdomain-http.yml`，以指向您的域：
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-customdomain-http
   spec:
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
2. 部署 Ingress
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. 在 `http://<customdomain>/` 上访问应用程序

### 使用 HTTPS

如果此时要尝试使用 HTTPS 在 `https://<customdomain>/` 上访问应用程序，那么可能会在 Web 浏览器中收到安全警告称连接不是专用的。此外，您还会收到 404 错误，因为刚才配置的 Ingress 并不知道如何定向 HTTPS 流量。

1. 为域获取可信的 SSL 证书。您将需要证书和密钥：
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3
   可以使用 [Let's Encrypt](https://letsencrypt.org/) 来生成可信证书。
2. 将证书和密钥保存在 base64 ASCII 格式的文件中。
3. 创建 TLS 私钥以存储证书和密钥：
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<custom-domain.cert> --key=<custom-domain.key>
   ```
   {: pre}
4. 创建 Ingress 文件 `ingress-customdomain-https.yml`，以指向您的域：
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-customdomain-https
   spec:
     tls:
     - hosts:
       - <my-custom-domain.com>
       secretName: <my-custom-domain-secret-name>
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
5. 部署 Ingress：
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. 在 `https://<customdomain>/` 上访问应用程序。

## 监视应用程序运行状况
{: #monitor_application}

1. 要检查应用程序的运行状况，请导航至[集群](https://{DomainName}/containers-kubernetes/clusters)以查看集群列表，然后单击在上面创建的集群。
2. 单击 **Kubernetes 仪表板**以在新选项卡中启动该仪表板。
   ![](images/solution2/launch_kubernetes_dashboard.png)
3. 选择左侧窗格上的**节点**，单击节点的**名称**，然后查看**分配资源**可了解节点的运行状况。
   ![](images/solution2/KubernetesDashboard.png)
4. 要查看容器中的应用程序日志，请选择 **pod** > **pod-name**，然后选择**日志**。
5. 要通过 **SSH** 登录到容器，请识别上一步中您的 pod 名称，然后运行该 pod：
   ```sh
   kubectl exec -it <pod-name> -- bash
   ```
   {: pre}

## 缩放 Kubernetes pod
{: #scale_cluster}

随着应用程序负载的增加，可以手动增加部署中的 pod 副本数。副本由 [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) 进行管理。要将应用程序扩展为两个副本，请运行以下命令：

   ```sh
 kubectl scale deployment <nameofproject>-deployment --replicas=2
   ```
   {: pre}

片刻之后，您将在 Kubernetes 仪表板中（或通过 `kubectl get pods`）看到用于您应用程序的两个 pod。集群中的 Ingress 控制器将处理两个副本之间的负载均衡。此外，还可以使水平缩放自动执行。

有关手动和自动缩放的信息，请参阅 Kubernetes 文档：

   * [Scaling a deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## 除去资源

* 删除集群，或者如果计划复用集群，请仅删除为应用程序创建的 Kubernetes 工件。

## 相关内容

* [IBM Cloud Kubernetes Service](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [持续部署到 Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
