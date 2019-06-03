---
copyright:
  years: 2019
lastupdated: "2019-04-02"
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
{:important: .important}

# 跨多个位置和专区部署隔离工作负载
{: #vpc-multi-region}

从 2019 年 4 月初开始，IBM 将接受有限数量的客户参与 VPC 的“抢先体验”计划，并在接下来的几个月内开放更大的使用量。如果您的组织希望获取 IBM Virtual Private Cloud 的访问权，请填写此[报名表](https://{DomainName}/vpc){: new_window}，IBM 代表将联系您，就后续步骤与您进行沟通。
{: important}

本教程将引导您通过在不同的 IBM Cloud 区域供应 VPC 来设置隔离的工作负载。具有子网和虚拟服务器实例 (VSI) 的区域。这些 VSI 在一个区域的多个专区中创建，通过配置带有后端池、前端侦听器和正确运行状况检查的负载均衡器，增加区域内以及全局范围的弹性。

对于全局负载均衡器，您将从目录供应 IBM Cloud Internet Services (CIS) 服务，对于管理所有入局 HTTPS 请求的 SSL 证书，将创建 {{site.data.keyword.cloudcerts_long_notm}} 目录服务，并将导入该证书以及专用密钥。

{:shortdesc}

## 目标
{: #objectives}

* 通过虚拟私有云可用的基础架构对象了解工作负载隔离。
* 在一个区域的不同专区之间使用负载均衡器在虚拟服务器之间分发流量。
* 在区域之间使用全局负载均衡器，以提高弹性并缩短等待时间。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.loadbalancer_full}}](https://{DomainName}/vpc/provision/loadBalancer)
- IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
- [{{site.data.keyword.cloudcerts_long_notm}}](https://{DomainName}/catalog/services/cloudcerts)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/estimator/review)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

  ![体系结构](images/solution41-vpc-multi-region/Architecture.png)

1. 管理员 (DevOps) 在区域 1 中的 VPC 内两个不同专区下的子网中供应 VSI，并在区域 2 中创建的 VPC 内重复相同的操作。
2. 管理员使用区域 1 中不同专区内子网的后端服务器池和一个前端侦听器来创建负载均衡器。在区域 2 中重复相同的操作。
3. 管理员利用关联的定制域供应 cloud internet services 服务，并创建一个指向在两个不同 VPC 中创建的负载均衡器的全局负载均衡器。
4. 管理员通过将域 SSL 证书添加到证书管理器服务来启用 HTTPS 加密。
5. 因特网用户发出 HTTP/HTTPS 请求，全局负载均衡器处理该请求。
6. 请求路由到全局和本地的负载均衡器。然后，可用的服务器实例履行请求。

## 开始之前
{: #prereqs}

- 检查用户许可权。确保您的用户帐户具有创建和管理 VPC 资源所需的足够许可权。有关所需的许可权的列表，请参阅[授予 VPC 用户所需的许可权](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources)。
- 您需要 SSH 密钥来连接到虚拟服务器。如果您没有 SSH 密钥，请参阅[创建密钥的指示信息](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)。
- Cloud Internet Services 需要您拥有定制域，以便可将此域的 DNS 配置为指向 Cloud Internet Services 名称服务器。如果您未拥有域，那么您可以从 [godaddy.com](http://godaddy.com/) 之类的注册商购买一个域。

## 创建 VPC、子网和 VSI
{: #create-infrastructure}

在本部分中，您将在区域 1 中创建您自己的 VPC，并在区域 1 的两个不同专区中创建子网，然后供应 VSI。

要在区域 1 创建您自己的 {{site.data.keyword.vpc_short}}：

1. 导航至 [VPC 概述](https://{DomainName}/vpc/overview)页面，并单击**创建 VPC**。
2. 在**新建虚拟私有云**部分下：
   * 输入 **vpc-region1** 作为 VPC 的名称。
   * 选择**资源组**。
   * （可选）添加**标记**以组织您的资源。
3. 选择**新建缺省值（全部允许）**作为 VPC 缺省访问控制表 (ACL)。
4. 取消选中 SSH，从**缺省安全组**执行 ping 操作，并将**经典访问**保留为取消选中状态。
5. 在 **VPC 的新子网**下：
   * 输入 **vpc-region1-zone1-subnet** 作为唯一名称。
   * 选择位置（例如，达拉斯），不妨称为**区域 1**，在区域 1 中选择一个专区（例如，达拉斯 1），不妨称为**专区 1**。
   * 以 CIDR 表示法输入子网的 IP 范围，即 **10.xxx.0.0/24**。按原样保留此**地址前缀**，并选择 256 作为**地址数**。
6. 对子网访问控制表 (ACL) 选择**使用 VPC 缺省值**。您可以稍后配置入站和出站规则。
7. 考虑到子网中所有虚拟服务器实例将附加有浮动 IP，不需要为子网启用公共网关。虚拟服务器实例将通过其浮动 IP 连接到因特网。
8. 单击**创建虚拟私有云**以供应实例。

要确认子网的创建，请单击左侧窗格上的**子网**，并等待状态更改为**可用**。您可以在**子网**下创建新的子网。

### 在专区 2 中创建子网

1. 单击**新建子网**，输入 **vpc-region1-zone2-subnet** 作为子网的唯一名称，并选择 **vpc-region1** 作为 VPC。
2. 选择我们在上面称为区域 1 的位置（例如，达拉斯），并选择区域 1 中不同的专区（例如，达拉斯 2），不妨将所选专区称为**专区 2**。
3. 以 CIDR 表示法输入子网的 IP 范围，即 **10.xxx.64.0/24**。按原样保留此**地址前缀**，并选择 256 作为**地址数**。
4. 对子网访问控制表 (ACL) 选择**使用 VPC 缺省值**。

### 供应 VSI
在子网状态更改为**可用**后，

1. 单击 **vpc-region1-zone1-subnet**，再单击**附加的实例**，然后单击**新建实例**。
2. 输入唯一名称，并选取 **vpc-region1-zone1-vsi**。然后，选择先前创建的 VPC，并选择**位置**以及**专区**，如前所述。
3. 选择任何 **Ubuntu Linux** 映像，单击**所有概要文件**，在**计算**下，选择带有 2 个 vCPU 和 4 GB RAM 的 **c-2x4**。
4. 对于 **SSH 密钥**，选取您初始创建的 SSH 密钥。
5. 在**网络接口**下，单击“安全组”旁边的**编辑**图标。
   * 检查是否选择了 **vpc-region1-zone1-subnet** 作为子网。如果没有，请进行选择。
   * 单击**保存**。
   * 单击**创建虚拟服务器实例**。
6.  等待 VSI 状态更改为**已打开电源**。然后，选择 VSI **vpc-region1-zone1-vsi**，滚动到**网络接口**并单击**浮动 IP** 下的**预留**以将公共 IP 地址关联到您的 VSI。将关联的的 IP 地址保存到剪贴板，以供未来参考。
7. **重复**步骤 1-6 以在**区域 1** 的**专区 2**中供应 VSI。

导航至左侧窗格的**网络**下的 **VPC** 和**子网**，并**重复**上述步骤以遵循上述相同命名约定来使用**区域 2** 中的子网和 VSI 供应新的 VPC。

## 安装和配置 VSI 上的 Web 服务器
{: #install-configure-web-server-vsis}

遵循[使用防御主机安全地访问远程实例](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)中提及的步骤，使用充当 `jump` 服务器和维护安全组的防御主机来安全地维护服务器。
{:tip}

通过 SSH 成功登录到区域 1 内专区 1 的子网中供应的服务器后：

1. 在提示符处，运行下面的命令以将 Nginx 作为 Web 服务器进行安装：
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   {:codeblock}
2. 使用下面的命令检查 Nginx 服务的状态：
   ```
   sudo systemctl status nginx
   ```
   {:pre}
   输出应显示 Nginx 服务处于**活动**状态并且正在运行。
3. 您将需要打开 **HTTP (80)** 和 **HTTPS (443)** 端口以接收流量（请求）。您可以通过 [UFW](https://help.ubuntu.com/community/UFW) - `sudo ufw enable` 调整防火墙并启用包括两个端口的规则的“Nginx Full”概要文件来完成该操作：
   ```
   sudo ufw allow 'Nginx Full'
   ```
   {:pre}
4. 要验证 Nginx 是否按预期运行，请在所选的浏览器中打开 `http://FLOATING_IP`，此时您将看到缺省的 Nginx 欢迎页面。
5. 要使用区域和专区详细信息更新 html 页面，请运行下面的命令：
   ```
   nano /var/www/html/index.nginx-debian.html
   ```
   {:pre}
   将区域和专区标语_在**区域 1 的专区 1** 中运行的服务器_附加到显示`欢迎使用 nginx！`的 `h1` 标记，并保存更改。
6. 重新启动 nginx 服务器以反映更改
   ```
   sudo systemctl restart nginx
   ```
   {:pre}

**重复**步骤 1-6 以安装和配置所有专区的子网中 VSI 上的 Web 服务器，并务必使用相应的专区信息更新 html。


## 使用负载均衡器在专区之间分发流量
{: #distribute-traffic-with-load-balancers}

在此部分中，您将创建两个负载均衡器。每个区域一个负载均衡器，以在不同专区内相应子网下的多个服务器实例之间分发流量。

### 配置负载均衡器

1. 导航至**负载均衡器**，并单击**新建负载均衡器**。
2. 指定 **vpc-lb-region1** 作为唯一名称，并选择 **vpc-region1** 作为虚拟私有云，后跟创建 VPC 所在的资源组，输入：**公共**，并输入**区域 1** 作为区域。
3. 为**区域 1** 的**专区 1** 和**专区 2** 选择专用 IP。
4. 创建 VSI 的新后端池，以充当同等同级来共享路由到该池的流量。使用下面的值设置参数，并单击**创建**。
	- **名称**：region1-pool
	- **协议**：HTTP
	- **方法**：循环法
	- **会话粘性**：无
	- **运行状况检查路径**：/
	- **运行状况协议**：HTTP
	- **时间间隔（秒）**：15
	- **超时（秒）**：2
	- **最大重试次数**：2
5. 单击**附加**以将服务器实例添加到 region1-pool
   - 选择 **vpc-region1-zone1-subnet** 的专用 IP，选择您创建的实例，并将端口设置为 80。
   - 单击**添加**，这次选择 **vpc-region1-zone2-subnet** 的专用 IP，选择实例并将端口设置为 80。
   - 单击**附加**以完成后端池的创建。
6. 单击**新建侦听器**以创建新的前端侦听器；侦听器是检查连接请求的进程。
   - **协议**：HTTP
   - **端口**：80
   - **后端池**：region1-pool
   - **最大连接数**：留空并单击**创建**。
7. 单击**创建负载均衡器**以供应负载均衡器。

### 测试负载均衡器

1. 等待负载均衡器状态更改为**活动**。
2. 在 Web 浏览器中打开**地址**。
3. 刷新页面若干次，并注意每次刷新后，负载均衡器会命中不同的服务器。
4. **保存**地址以供将来参考。

如果您进行观察，将发现请求未加密，并仅支持 HTTP。在下一部分中，您将配置 SSL 证书和启用 HTTPS。

在**区域 2** 中**重复**上述步骤 1-7。

## 使用 HTTPS 保护 VPC 中的流量
{: #secure_https}

在添加 HTTPS 侦听器之前，您需要生成 SSL 证书，验证定制域的真实性，定制域是存放证书并将其映射到基础架构服务的地方。

### 供应 CIS 服务，并配置定制域。

在本部分中，您将创建 IBM Cloud Internet Services (CIS) 服务，通过将定制域指向 CIS 名称服务器进行配置，并稍后配置全局负载均衡器。

1. 导航至 {{site.data.keyword.Bluemix_notm}}“目录”中的 [Internet Services](https://{DomainName}/catalog/services/internet-services)。
2. 设置服务名称，然后单击**创建**以创建服务实例。对于本教程，您可以使用任何定价套餐。
3. 供应服务实例后，通过单击**开始使用**来设置域名，然后单击**添加域**。
4. 单击**下一步**。分配名称服务器后，配置注册商或域名提供商，以使用列出的名称服务器。
5. 配置注册商或 DNS 提供商后，最长可能需要 24 小时，更改才会生效。

   在“概述”页面上，域的状态从*暂挂*更改为*活动*时，可以使用 `dig <YOUR_DOMAIN_NAME> ns` 命令来验证新的名称服务器是否已生效。
   {:tip}

您应获得您计划用于全局负载均衡器的域和子域的 SSL 证书。假定有类似 mydomain.com 的域，那么全局负载均衡器可以在 `lb.mydomain.com` 上托管。需要为 lb.mydomain.com 发放该证书。

可以从 [Let's Encrypt](https://letsencrypt.org/) 获取免费的 SSL 证书。在此过程中，可能需要在 Cloud Internet Services 的 DNS 接口中配置 TXT 类型的 DNS 记录，以证明您是域的所有者。
{:tip}

获得域的 SSL 证书和专用密钥后，请务必将其转换为 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 格式。

1. 将证书转换为 PEM 格式：
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
2. 将专用密钥转换为 PEM 格式：
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### 导入证书，并授权负载均衡器服务

您可以通过 IBM Certificate Manager 管理 SSL 证书。

1. 在支持的位置中创建 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 实例。
2. 在服务仪表板中，使用**导入证书**：
   * 将**名称**设置为定制子域和域，例如 *lb.mydomain.com*。
   * 浏览以找到 PEM 格式的**证书文件**。
   * 浏览以找到 PEM 格式的**专用密钥文件**。
   * **导入**。
3. 创建授权，以向负载均衡器服务实例授予对包含 SSL 证书的证书管理器实例的访问权。您可以通过[身份和访问权授权](https://{DomainName}/iam#/authorizations)来管理此类授权。
  - 单击**创建**并选择**基础架构服务**作为源服务
  - **VPC 的负载均衡器**作为资源类型
  - **证书管理器**作为目标服务
  - 分配**写入者**服务访问角色。
  - 要创建负载均衡器，您必须对源资源实例授予“全部”资源实例权限。目标服务实例可能是**全部实例**，或者也可能是具体的证书管理器资源实例。

### 创建 HTTPS 侦听器

现在，导航至[负载均衡器](https://{DomainName}/vpc/network/loadBalancers)

1. 选择 **vpc-lb-region1**
2. 在**前端侦听器**下，单击**新建侦听器**

   -  **协议**：HTTPS
   -  **端口**：443
   -  **后端池**：同一区域中的池
   -  为 **lb.YOUR-DOMAIN-NAME** 选择 SSL 证书

3. 单击**创建**以配置 HTTPS 侦听器

在**区域 2** 的负载均衡器中**重复**相同的操作。

## 配置全局负载均衡器
{: #global-load-balancer}

在此部分中，您会配置将入局流量分发给不同 {{site.data.keyword.Bluemix_notm}} 区域中配置的本地负载均衡器的全局负载均衡器 (GLB)。

### 通过全局负载均衡器在区域之间分发流量
通过导航至服务下的[资源列表](https://{DomainName}/resources)，打开所创建的 CIS 服务。

1. 导航至**可靠性**下的**全局负载均衡器**，并单击**创建负载均衡器**。
2. 输入 **lb.YOUR-DOMAIN-NAME** 作为主机名，并将 TTL 设置为 60 秒。
3. 单击**添加池**以定义缺省源池
   - **名称**：lb-region1
   - **运行状况检查**：CREATE A NEW HEALTH CHECK
     - **监视器类型**：HTTP
     - **路径**：/
     - **端口**：80
   - **运行状况检查区域**：Eastern North America
   - **源**
     - **名称**：region1
     - **地址**：ADDRESS OF **REGION1** LOCAL LOAD BALANCER
     - **权重**：1
     - 单击**添加**

4. 再**添加**一个指向**西欧**区域中**区域 2** 负载均衡器的**源池**，并单击**供应 1 个资源**，以供应全局负载均衡器。

等待**运行状况**检查状态更改为**正常运行**。打开所选浏览器中的链接 **lb.YOUR-DOMAIN-NAME** 以查看操作中的全局负载均衡器。

### 故障转移测试
到现在为止，您应该看到大多数情况下，您命中**区域 1** 中的服务器，因为向其分配了相较于**区域 2** 中的服务器更高的权重。接下来在**区域 1** 源池中引入运行状况检查故障。

1. 导航至[虚拟服务器实例](https://{DomainName}/vpc/compute/vs)。
2. 单击在**区域 1** 中**专区 1** 内运行的服务器旁边的**三个点 (...)**，并单击**停止**。
3. 对于**区域 1** 的**专区 2** 中运行的服务器，**重复**相同的操作。
4. 返回到 CIS 服务下的 GLB，并等待运行状态更改为**严重**。
5. 现在，刷新域 URL 时，您应该会始终命中**区域 2** 中的服务器。

不要忘记**启动**区域 1 的专区 1 和专区 2 中的服务器
{:tip}

## 除去资源
{: #removeresources}

- 除去 CIS 服务下的全局负载均衡器、源池和运行状况检查。
- 除去证书管理器服务中的证书。
- 除去负载均衡器、VSI、子网和 VPC。
- 在[资源列表](https://{DomainName}/resources)下，删除本教程中使用的服务。


## 相关内容
{: #related}

* [在 IBM Cloud VPC 中使用负载均衡器](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc#--beta-using-load-balancers-in-ibm-cloud-vpc)
