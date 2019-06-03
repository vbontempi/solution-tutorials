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

# Virtual Private Cloud 中的专用和公共子网
{: #vpc-public-app-private-backend}

从 2019 年 4 月初开始，IBM 将接受有限数量的客户参与 VPC 的“抢先体验”计划，并在接下来的几个月内开放更大的使用量。如果您的组织希望获取 IBM Virtual Private Cloud 的访问权，请填写此[报名表](https://{DomainName}/vpc){: new_window}，IBM 代表将联系您，就后续步骤与您进行沟通。
{: important}

本教程引导您使用公共和专用子网以及每个子网中的虚拟服务器实例 (VSI) 创建您自己的 {{site.data.keyword.vpc_full}} (VPC)。VPC 是共享云基础架构上与其他虚拟网络逻辑隔离的您自己的私有云。

[子网](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#subnet)是 IP 地址范围。子网必须绑定到单个专区，不能跨多个专区或区域。对于 VPC 的用途，子网的重要特征是子网可以彼此隔离，也可以按通常方式互连。子网隔离可以通过网络[访问控制表](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#access-control-list) (ACL) 来实现，ACL 充当防火墙，用于控制子网之间的数据包流。同样地，[安全组](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#security-group) (SG) 充当虚拟防火墙，用于控制进出单个 VSI 的数据包流。

公共子网用于必须公开给外部世界的资源。访问受限而绝不能由外部世界直接访问的资源必须放在专用子网中。这种子网上的实例可以是后端数据库或者您不希望可公开访问的一些秘密存储。您将定义 SG 以允许或者拒绝进入 VSI 的流量。
{:shortdesc}

简言之，使用 VPC，您可以

- 创建软件定义的网络 (SDN)，
- 隔离工作负载，
- 精细控制入站和出站流量。

## 目标

{: #objectives}

- 了解虚拟私有云可用的基础架构对象
- 了解如何创建虚拟私有云、子网和服务器实例
- 了解如何应用安全组以保护对服务器的访问

## 使用的服务

{: #services}

本教程使用以下运行时和服务：

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

![体系结构](images/solution40-vpc-public-app-private-backend/Architecture.png)


1. 管理员 (DevOps) 在云中设置所需的基础架构（VPC、子网、带规则的安全组、VSI）。
2. 因特网用户向前端的 Web 服务器发出 HTTP/HTTPS 请求。
3. 前端从安全的后端请求专用资源，并将结果提供给用户。

## 开始之前

{: #prereqs}

- 检查用户许可权。确保您的用户帐户具有创建和管理 VPC 资源所需的足够许可权。有关所需的许可权的列表，请参阅[授予 VPC 用户所需的许可权](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources)。

- 您需要 SSH 密钥来连接到虚拟服务器。如果您没有 SSH 密钥，请参阅[创建密钥的指示信息](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)。

## 创建 Virtual Private Cloud
{: #create-vpc}

创建您自己的 {{site.data.keyword.vpc_short}}：

1. 导航至 [VPC 概述](https://{DomainName}/vpc/overview)页面，并单击**创建 VPC**。
2. 在**新建虚拟私有云**部分下：
   * 输入 **vpc-pubpriv** 作为 VPC 的名称。
   * 选择**资源组**。
   * （可选）添加**标记**以组织您的资源。
3. 选择**新建缺省值（全部允许）**作为 VPC 缺省访问控制表 (ACL)。
1. 取消选中 SSH，从**缺省安全组**执行 ping 操作。
4. 在 **VPC 的新子网**下：
   * 输入 **vpc-pubpriv-backend-subnet** 作为唯一名称。
   * 选择位置。
   * 以 CIDR 表示法输入子网的 IP 范围，即 **10.xxx.0.0/24**。按原样保留此**地址前缀**，并选择 256 作为**地址数**。
5. 对子网访问控制表 (ACL) 选择**使用 VPC 缺省值**。您可以稍后配置入站和出站规则。
6. 单击**创建虚拟私有云**以供应实例。

如果连接到专用子网的 VSI 需要访问因特网以装入软件，请将公共网关切换到**已连接**，因为连接公共网关将允许所有连接的资源与公共因特网通信。一旦 VSI 拥有所需的所有软件，请将公共网关返回到**拆离**状态，以便该子网无法访问公用因特网。
{: important}

要确认子网的创建，请单击左侧窗格上的**子网**，并等待状态更改为**可用**。您可以在**子网**下创建新的子网。

## 创建后端安全组和 VSI
{: #backend-subnet-vsi}

在本部分中，您将为后端创建安全组和虚拟服务器实例。

### 创建后端安全组

缺省情况下，安全组与 VPC 一起创建，以允许进入所连接实例的所有 SSH（TCP 端口 22）和 Ping（ICMP 类型 8）流量。

要为后端创建新的安全组：  
1. 单击**网络**下的**安全组**，然后单击**新建安全组**。  
2. 输入 **vpc-pubpriv-backend-sg** 作为名称，然后选择先前创建的 VPC。  
3. 单击**创建安全组**。

您将稍后编辑安全组以添加入站和出站规则。

### 创建后端虚拟服务器实例

在新创建的子网中创建虚拟服务器实例：

1. 单击**子网**下的后端子网。
2. 单击**连接的实例**，然后单击**新建实例**。
3. 输入唯一名称，并选取 **vpc-pubpriv-backend-vsi**。然后，选择先前创建的 VPC 和**位置**，如前所述。
4. 选择 **Ubuntu Linux** 映像，单击**所有概要文件**，在**计算**下，选择带有 2 个 vCPU 和 4 GB RAM 的 **c-2x4**。
5. 对于 **SSH 密钥**，选择您先前创建的 SSH 密钥。
6. 在**网络接口**下，单击“安全组”旁边的**编辑**图标。
   * 选择 **vpc-pubpriv-backend-subnet** 作为子网。
   * 取消选中缺省安全组，并将 **vpc-pubpriv-backend-sg** 勾选为活动。
   * 单击**保存**。
7. 单击**创建虚拟服务器实例**。

## 创建前端子网、安全组和 VSI
{: #frontend-subnet-vsi}

与后端类似，您将创建带有虚拟服务器实例和安全组的前端子网。

### 创建前端子网

为前端创建新的子网：

1. 单击左侧窗格的**网络**下的**子网** > **新建子网**。
   * 输入 **vpc-pubpriv-frontend-subnet** 作为名称，然后选择您创建的 VPC。
   * 选择位置。
   * 以 CIDR 表示法输入子网的 IP 范围，即 **10.xxx.1.0/24**。按原样保留此**地址前缀**，并选择 256 作为**地址数**。
1. 对子网访问控制表 (ACL) 选择 **VPC 缺省值**。您可以稍后配置入站和出站规则。
1. 考虑到子网中所有虚拟服务器实例将附加有浮动 IP，不需要为子网启用公共网关。虚拟服务器实例将通过其浮动 IP 连接到因特网。
1. 单击**创建子网**以进行供应。

### 创建前端安全组

为前端创建新的安全组：
1. 单击“网络”下的**安全组**，然后单击**新建安全组**。
2. 输入 **vpc-pubpriv-frontend-sg** 作为名称，然后选择先前创建的 VPC。
3. 单击**创建安全组**。

### 创建前端虚拟服务器实例

在新创建的子网中创建虚拟服务器实例：

1. 单击**子网**下的前端子网。
2. 单击**连接的实例**，然后单击**新建实例**。
3. 输入唯一名称 **vpc-pubpriv-frontend-vsi**，选择先前创建的 VPC，然后选择与之前相同的**位置**。
4. 选择 **Ubuntu Linux** 映像，单击**所有概要文件**，在**计算**下，选择带有 2 个 vCPU 和 4 GB RAM 的 **c-2x4**。
5. 对于 **SSH 密钥**，选择您先前创建的 SSH 密钥。
6. 在**网络接口**下，单击“安全组”旁边的**编辑**图标。
   * 选择 **vpc-pubpriv-frontend-subnet** 作为子网。
   * 取消选中缺省安全组，并激活 **vpc-pubpriv-frontend-sg**。
   * 单击**保存**。
   * 单击**创建虚拟服务器实例**。
7. 等待 VSI 状态更改为**已打开电源**。然后，选择前端 VSI **vpc-pubpriv-frontend-vsi**，滚动到**网络接口**并单击**浮动 IP** 下的**预留**以将公共 IP 地址关联到您的前端 VSI。将关联的的 IP 地址保存到剪贴板，以供未来参考。

## 设置前端与后端之间的连接
{: #setup-connectivity-frontend-backend}

所有服务器就位后，在本部分中，你将设置连接以允许前端和后端服务器之间进行常规操作。

### 配置前端安全组

1. 导航至**网络**部分的**安全组**，然后单击 **vpc-pubpriv-frontend-sg**。
2. 首先，使用**添加规则**添加以下**入站**规则。这些规则允许入局 HTTP 请求和 Ping (ICMP)。

	<table>
   <thead>
      <tr>
         <td><strong>源</strong></td>
         <td><strong>协议</strong></td>
         <td><strong>值</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>任何 - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>从：<strong>80</strong> 到 <strong>80</strong></td>
      </tr>
      <tr>
         <td>任何 - 0.0.0.0/0</td>
         <td>TCP</td>
         <td>从：<strong>443</strong> 到 <strong>443</strong></td>
      </tr>
      <tr>
         <td>任何 - 0.0.0.0/0</td>
	      <td>ICMP</td>
	      <td>类型：<strong>8</strong>，代码：<strong>留空</strong></td>
      </tr>
   </tbody>
   </table>

3. 接下来，添加这些**出站**规则。

   <table>
   <thead>
      <tr>
         <td><strong>目标</strong></td>
         <td><strong>协议</strong></td>
         <td><strong>值</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>类型：<strong>安全组</strong> - 名称：<strong>vpc-pubpriv-backend-sg</strong></td>
         <td>TCP</td>
         <td>后端服务器的端口，参见提示</td>
      </tr>
   </tbody>
   </table>

这里有典型后端服务的端口。MySQL 使用端口 3306，PostgreSQL 使用端口 5432。Db2 在端口 50000 或 50001 上进行访问。缺省情况下，Microsoft SQL Server 使用端口 1433。
[常用端口的许多列表之一可在 Wikipedia 上找到](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)。
{:tip }

### 配置后端安全组
以类似于前端的方式，为后端配置安全组。

1. 导航至**网络**部分的**安全组**，然后单击 **vpc-pubpriv-backend-sg**。
2. 使用**添加规则**添加以下**入站**规则。该规则允许连接到后端服务。

   <table>
   <thead>
      <tr>
         <td><strong>源</strong></td>
         <td><strong>协议</strong></td>
         <td><strong>值</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>类型：<strong>安全组</strong> - 名称：<strong>vpc-pubpriv-frontend-sg</strong></td>
         <td>TCP</td>
         <td>后端服务器的端口</td>
      </tr>
   </tbody>
   </table>


## 安装软件并执行维护任务
{: #install-software-maintenance-tasks}

遵循[使用防御主机安全地访问远程实例](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)中提及的步骤，使用充当 `jump` 服务器和维护安全组的防御主机来安全地维护服务器。


## 除去资源
{: #remove-resources}

1. 在 VPC 管理控制台中，单击**浮动 IP**，再单击 VSI 的 IP 地址，然后在“操作”菜单中选择**发布**。确认您想要发布此 IP 地址。
2. 接下来，切换到**虚拟服务器实例**并**删除**实例。实例将被删除，其状态将在一段时间内保留为**正在删除**。确保时不时地刷新浏览器。
3. 删除 VSI 之后，切换到**子网**。如果子网具有连接的公共网关，那么单击子网名称。在子网详细信息中，拆离公共网关。可以从概述页面删除没有公共网关的子网。删除子网。
4. 删除子网后，切换到 **VPC** 选项卡，并删除您的 VPC。

使用控制台时，您可能需要刷新浏览器以在删除资源后查看更新后的状态信息。
{:tip}

## 扩展教程
{: #expand-tutorial}

想要补充或扩展本教程吗？下面是一些构想：


- 添加[负载均衡器](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-load-balancer)以跨多个实例分发入站流量。
- 创建[虚拟专用网](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn) (VPN)，以便您的 VPC 可以安全地连接到其他专用网络，例如内部部署网络或其他 VPC。


## 相关内容
{: #related}

- [VPC 词汇表](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)
- [使用 IBM Cloud CLI 的 VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli#creating-a-vpc-using-the-ibm-cloud-cli)
- [使用 REST API 的 VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis#creating-a-vpc-using-the-rest-apis)
