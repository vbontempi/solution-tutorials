---
copyright:
  years: 2019
lastupdated: "2019-05-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}
{:important: .important}

# 使用 VPC/VPN 网关对云资源进行安全的专用内部部署访问
{: #vpc-site2site-vpn}

从 2019 年 4 月初开始，IBM 将接受有限数量的客户参与 VPC 的“抢先体验”计划，并在接下来的几个月内开放更大的使用量。如果您的组织希望获取 IBM Virtual Private Cloud 的访问权，请填写此[报名表](https://{DomainName}/vpc){: new_window}，IBM 代表将联系您，就后续步骤与您进行沟通。
{: important}

IBM 提供了多种方法来使用 IBM Cloud 中的资源安全地扩展内部部署计算机网络。这样，您可以灵活机动地在需要服务器时供应服务器并在不再需要时除去服务器。此外，您可以轻松、安全地将内部部署功能连接到 {{site.data.keyword.cloud_notm}} 服务。

本教程将全程指导将内部部署虚拟专用网 (VPN) 网关连接到 VPC（VPC/VPN 网关）中创建的云 VPN。首先，您将创建新的 {{site.data.keyword.vpc_full}} (VPC) 以及关联的资源，例如子网、网络访问控制表 (ACL)、安全组以及虚拟服务器实例 (VSI)。
VPC/VPN 网关将对内部部署 VPN 网关建立 [IPsec](https://en.wikipedia.org/wiki/IPsec) 站点到站点链路。IPsec 和 [Internet Key Exchange](https://en.wikipedia.org/wiki/Internet_Key_Exchange) (IKE) 协议是成熟的安全通信开放标准。

要进一步演示安全和专用访问，您将在 VSI 上部署微服务，以访问 {{site.data.keyword.cos_short}} (COS)，后者表示业务线应用程序。
COS 服务具有直接端点，在所有访问权位于 {{site.data.keyword.cloud_notm}} 的同一区域中时，可将此端点用于专用的无成本入口/出口。内部部署计算机将访问 COS 微服务。所有流量将流经 VPN，并因此将以专用方式流经 {{site.data.keyword.cloud_notm}}。

对于站点到站点网关，有许多热门的内部部署 VPN 解决方案可用。本教程利用 [strongSwan](https://www.strongswan.org/) VPN 网关来连接到 VPC/VPN 网关。要模拟内部部署数据中心，您将在 {{site.data.keyword.cloud_notm}} 的 VSI 上安装 strongSwan 网关。

{:shortdesc}
简言之，使用 VPC，您可以

- 将内部部署系统连接到 {{site.data.keyword.cloud_notm}} 中运行的服务和工作负载，
- 确保 COS 的专用和低成本连接，
- 将基于云的系统连接到以内部部署方式运行的服务和工作负载。

## 目标
{: #objectives}

* 从内部部署数据中心或者（虚拟）私有云访问虚拟私有云环境。
* 使用专用服务端点安全地访问云服务。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。虽然本教程中从微服务访问 COS 没有网络费用，但是访问 VPC 会发生标准网络费用。

## 体系结构
{: #architecture}

下图显示包含应用程序服务器的虚拟私有云。应用程序服务器托管与 {{site.data.keyword.cos_short}} 服务交互的微服务。（模拟的）内部部署网络和虚拟云环境通过 VPN 网关进行连接。

![体系结构](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. 使用提供的脚本来设置基础架构（VPC、子网，带规则的安全组、网络 ACL 和 VSI）。
2. 微服务通过直接端点与 {{site.data.keyword.cos_short}} 交互。
3. 供应 VPC/VPN 网关，以将虚拟私有云环境公开给内部部署网络。
4. Strongswan 开放式源代码 IPsec 网关软件以内部部署方式用于建立与云环境的 VPN 连接。

## 开始之前
{: #prereqs}

- 通过[执行以下步骤](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)，安装所有必需的命令行 (CLI) 工具。您需要可选的 CLI 基础架构插件。
- 通过命令行登录到 {{site.data.keyword.cloud_notm}}。有关详细信息，请参阅 [CLI 入门](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli)。
- 检查用户许可权。确保您的用户帐户具有创建和管理 VPC 资源所需的足够许可权。有关所需的许可权的列表，请参阅[授予 VPC 用户所需的许可权](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)。
- 您需要 SSH 密钥来连接到虚拟服务器。如果您没有 SSH 密钥，请参阅[创建密钥的指示信息](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)。
- 安装 [**jq**](https://stedolan.github.io/jq/download/)。所提供的脚本将其用于处理 JSON 输出。

## 在虚拟私有云中部署虚拟应用程序服务器
接下来，您将下载脚本以设置微服务的基线 VPC 环境和代码，以与 {{site.data.keyword.cos_short}} 交互。因此，您将供应 {{site.data.keyword.cos_short}} 服务并设置基线。

### 获取代码
{: #setup}
本教程在创建 VPN 网关之前使用脚本来部署基础架构资源的基线。微服务的这些脚本和代码在 GitHub 存储库中提供。

1. 获取应用程序的代码：
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. 通过切换到 **vpc-tutorials**，然后切换到 **vpc-site2site-vpn**，以转至本教程的脚本：
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### 创建服务
在本部分中，您将登录 CLI 上的 {{site.data.keyword.cloud_notm}}，并创建 {{site.data.keyword.cos_short}} 的实例。

1. 验证您是否已执行登录的必备步骤：
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. 使用**标准**或**轻量**套餐创建 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) 的实例。
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   请注意，每个帐户只可以创建一个轻量实例。如果您已经拥有 {{site.data.keyword.cos_short}} 实例，可以加以复用。
   {: tip}

3. 使用角色**读取者**创建服务密钥：
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. 以 JSON 格式获取服务密钥详细信息，并将其存储在子目录 **vpc-app-cos** 中的新文件 **credentials.json** 中。应用程序将在稍后使用该文件。
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### 创建 Virtual Private Cloud 基线资源
{: #create-vpc}
本教程提供脚本以创建本教程所需的基线资源，即起始环境。该脚本可以在现有 VPC 中生成该环境，或者创建新的 VPC。

接下来，通过配置设置脚本，然后运行该脚本，从而创建这些资源。该脚本包含防御主机的设置，如[使用防御主机安全地访问远程实例](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)中所述。

1. 将样本配置文件复制到要使用的文件中：

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. 编辑文件 **config.sh**，并根据环境调整设置。您需要将 **SSHKEYNAME** 的值更改为 SSH 密钥的名称或多个名称的逗号分隔列表（请参阅“准备工作”）。修改不同的**专区**设置以匹配您的云区域。所有其他变量可以按原样保留。
3. 要在新的 VPC 中创建资源，请按如下所示运行脚本：

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   要复用现有 VPC，请按此方式将其名称传递给脚本。将 **YOUR_EXISTING_VPC** 替换为实际的 VPC 名称。
   ```sh
   REUSE_VPC=YOUR_EXISTING_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. 这样将导致创建以下资源，包括与防御相关的资源：
   - 1 个 VPC（可选）
   - 1 个公共网关
   - VPC 中的 3 个子网
   - 带入口和出口规则的 4 个安全组
   - 3 个 VSI：vpns2s-onprem-vsi（浮动 ip 为 ONPREM_IP），vpns2s-cloud-vsi（浮动 ip 为 VSI_CLOUD_IP），以及 vpns2s-bastion（浮动 ip 为 BASTION_IP_ADDRESS）

   记下 **BASTION_IP_ADDRESS**、**VSI_CLOUD_IP**、**ONPREM_IP**、**CLOUD_CIDR** 和 **ONPREM_CIDR** 的返回值以供稍后使用。输出还会存储在文件
**network_config.sh** 中。该文件可以用于自动设置。

### 创建虚拟专用网网关和连接
接下来，您将添加 VPN 网关以及与应用程序 VSI 的子网的关联连接。

1. 导航至 [VPC 概述](https://{DomainName}/vpc/overview)页面，然后单击导航选项卡中的 **VPN**，以及对话框中的**新建 VPN 网关**。在表单**新建 VPC 的 VPN 网关**中，输入 **vpns2s-gateway** 作为名称。确保选择正确的 VPC、资源组以及子网 **vpns2s-cloud-subnet**。
2. 使**新建 VPC 的 VPN 连接**保持激活状态。输入 **vpns2s-gateway-conn** 作为名称。
3. 对于**同级网关地址**，使用 **vpns2s-onprem-vsi** 的浮动 IP 地址 (ONPREM_IP)。输入 **20_PRESHARED_KEY_KEEP_SECRET_19** 作为**预共享密钥**。
4. 对于**本地子网**，使用为 **CLOUD_CIDR** 提供的信息，对于**同级子网**，使用为 **ONPREM_CIDR** 提供的信息。
5. 按原样保留**失效同级检测**中的设置。单击**创建 VPN 网关**以创建网关和关联的连接。
6. 等待 VPN 网关变为可用（您可能需要刷新屏幕）。
7. 记下作为 **GW_CLOUD_IP** 分配的**网关 IP** 地址。 

### 创建内部部署虚拟专用网网关
接下来，您将在模拟的内部部署环境中的其他站点上创建 VPN 网关。您将使用基于开放式源代码的 IPSec 软件 [strongSwan](https://strongswan.org/)。

1. 使用 ssh 连接到“内部部署”VSI **vpns2s-onprem-vsi**。执行以下命令，并将 **ONPREM_IP** 替换为先前返回的 IP 地址。

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   根据您的环境，您可能需要使用 `ssh -i <path to your private key file> root@ONPREMP_IP`。
   {:tip}

2. 接下来，在机器 **vpns2s-onprem-vsi** 上，执行以下命令以更新软件包管理器并安装 strongSwan 软件。

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. 通过在文件 **/etc/sysctl.conf** 末尾添加三行来配置该文件。复制以下内容并运行该文件：

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. 接下来，编辑文件 **/etc/ipsec.secrets**。添加以下行以配置源和目标 IP 地址以及先前配置的预共享密钥。将 **ONPREM_IP** 替换为 vpns2s-onprem-vsi 的浮动 IP 的已知值。将 **GW_CLOUD_IP** 替换为 VPC VPN 网关的已知 IP 地址。

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. 您需要配置的最后一个文件是 **/etc/ipsec.conf**。将以下代码块添加到该文件末尾。将 **ONPREM_IP**、**ONPREM_CIDR**、**GW_CLOUD_IP** 和 **CLOUD_CIDR** 替换为相应的已知值。

   ```sh
   # basic configuration
   config setup
      charondebug="all"
      uniqueids=yes
      strictcrlpolicy=no
   
   # connection to vpc/vpn datacenter 
   # left=onprem / right=vpc
   conn tutorial-site2site-onprem-to-cloud
      authby=secret
      left=%defaultroute
      leftid=ONPREM_IP
      leftsubnet=ONPREM_CIDR
      right=GW_CLOUD_IP
      rightsubnet=CLOUD_CIDR
      ike=aes256-sha2_256-modp1024!
      esp=aes256-sha2_256!
      keyingtries=0
      ikelifetime=1h
      lifetime=8h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start
   ```
   {:pre}

6. 重新启动 VPN 网关，然后通过运行 ipsec restart 来检查其状态

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   这时应报告连接已建立。保持此机器的终端和 ssh 连接为打开状态。

## 测试连接
您可以通过使用 SSH 或者通过部署与 {{site.data.keyword.cos_short}} 交互的微服务来测试站点到站点的 VPN 连接。

### 使用 SSH 测试
要测试 VPN 连接是否已成功建立，请使用模拟的内部部署环境作为代理，以登录到基于云的应用程序服务器。 

1. 在新的终端中，替换值之后，执行以下命令。该命令使用 strongSwan 主机作为跳板机，通过 VPN 连接到应用程序服务器的专用 IP 地址。

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. 一旦成功连接，关闭 SSH 连接。

3. 在“内部部署”VSI 终端中，停止 VPN 网关：
   ```sh
   ipsec stop
   ```
   {:pre}
4. 在步骤 1) 的命令窗口中，尝试再次建立连接：

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
   此命令应该不会成功，因为 VPN 连接未处于活动状态，因此模拟的内部部署和云环境之间没有直接链路。

   请注意，根据部署详细信息，此连接实际上仍会成功。原因是在各专区之间支持 VPC 内部连接。如果您会将模拟的内部部署 VSI 部署到其他 VPC 或 [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers)，那么将需要 VPN 才能成功访问。
   {:tip}
   
5. 在“内部部署”VSI 终端中，再次启动 VPN 网关：
   ```sh
   ipsec start
   ```
   {:pre}
 

### 使用微服务测试
您可以从内部部署 VSI 通过访问云 VSI 上的微服务来测试运行中的 VPN 连接。

1. 将微服务应用程序的代码从本地计算机复制到云 VSI。此命令使用防御主机作为云 VSI 的跳板机。相应地替换 **BASTION_IP_ADDRESS** 和 **VSI_CLOUD_IP**。
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. 再次使用防御主机作为跳板机连接到云 VSI。
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. 在云 VSI 上，切换到代码目录：
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. 安装 Python 和 Python 软件包管理器 PIP。
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. 使用 **pip** 安装必需的 Python 软件包。
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. 启动应用程序：
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. 在“内部部署”VSI 终端中，访问该服务。相应地替换 VSI_CLOUD_IP。
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
此命令应返回 JSON 对象。

## 除去资源
{: #remove-resources}

1. 在 VPC 管理控制台，单击 **VPN**。在 VPN 网关上的操作菜单中，选择**删除**以除去网关。
2. 接下来，单击导航中的**浮动 IP**，然后单击您的 VSI 的 IP 地址。在操作菜单中，选择**发布**。确认您想要发布此 IP 地址。
3. 接下来，切换到**虚拟服务器实例**并**删除**实例。实例将被删除，其状态将在一段时间内保留为**正在删除**。确保时不时地刷新浏览器。
4. 删除 VSI 之后，切换到**子网**。如果子网具有连接的公共网关，那么单击子网名称。在子网详细信息中，拆离公共网关。可以从概述页面删除没有公共网关的子网。删除子网。
5. 删除子网后，切换到 **VPC**，并删除您的 VPC。

使用控制台时，您可能需要刷新浏览器以在删除资源后查看更新后的状态信息。
{:tip}

## 扩展教程 
{: #expand-tutorial}

想要补充或扩展本教程吗？下面是一些构想：


- 添加[负载均衡器](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc)以跨多个实例分发入站微服务流量。
- [在公共服务器上部署应用程序，在专用主机上部署您的数据和服务](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend)。


## 相关内容
{: #related}

- [VPC 词汇表](/docs/vpc?topic=vpc-vpc-glossary)
- [VPC 的 IBM Cloud CLI 插件参考](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [使用 REST API 的 VPC](/docs/infrastructure/vpc/example-code.html)
- 解决方案教程：[使用防御主机安全地访问远程实例](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
