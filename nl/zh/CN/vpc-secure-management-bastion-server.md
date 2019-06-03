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

# 使用防御主机安全地访问远程实例
{: #vpc-secure-management-bastion-server}

从 2019 年 4 月初开始，IBM 将接受有限数量的客户参与 VPC 的“抢先体验”计划，并在接下来的几个月内开放更大的使用量。如果您的组织希望获取 IBM Virtual Private Cloud 的访问权，请填写此[报名表](https://{DomainName}/vpc){: new_window}，IBM 代表将联系您，就后续步骤与您进行沟通。
{: important}

本教程将全程指导您部署防御主机，以安全地访问虚拟私有云中的远程实例。防御主机是在公共子网中供应的实例，可以通过 SSH 访问。设置后，防御主机充当**跳板机**服务器，允许安全地连接到专用子网中供应的实例。

要降低 VPC 中服务器的敞口，您将创建和使用防御主机。单个服务器上的管理任务将使用 SSH 执行，通过防御主机进行代理。仅允许使用连接到服务器的特殊维护安全组来访问服务器以及从服务器定期访问因特网（例如，用于软件安装）。
{:shortdesc}

## 目标
{: #objectives}

* 了解如何设置防御主机和带规则的安全组
* 通过防御主机安全管理服务器

## 使用的服务
{: #services}

本教程使用以下运行时和服务：  

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)  
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

  ![体系结构](images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png)

1. 在云中设置所需的基础架构（子网、带规则的安全组、VSI）之后，管理员 (DevOps) 使用专用的 SSH 密钥连接 (SSH) 到防御主机。
2. 管理员分配带合适出站规则的维护安全组。
3. 管理员通过防御主机安全地连接 (SSH) 到实例的专用 IP 地址，以安装或更新任何所需软件，例如，Web 服务器
4. 因特网用户向 Web 服务器发出 HTTP/HTTPS 请求。

## 开始之前
{: #prereqs}

- 检查用户许可权。确保您的用户帐户具有创建和管理 VPC 资源所需的足够许可权。有关所需的许可权的列表，请参阅[授予 VPC 用户所需的许可权](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources)。
- 您需要 SSH 密钥来连接到虚拟服务器。如果您没有 SSH 密钥，请参阅[创建密钥的指示信息](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)。
- 本教程假定您要在现有[虚拟私有云](https://{DomainName}/vpc/network/vpcs)中添加防御主机。**如果您的帐户中没有虚拟私有云，请创建一个虚拟私有云，然后继续后续步骤。**

## 创建防御主机
{: #create-bastion-host}

在本部分中，您将创建和配置防御主机以及不同子网中的安全组。

### 创建子网
{: #create-bastion-subnet}

1. 单击左侧窗格的**网络**下的**子网**，然后单击**新建子网**。  
   * 输入 **vpc-secure-bastion-subnet** 作为名称，然后选择您创建的 VPC。  
   * 选择一个位置和专区。  
   * 以 CIDR 表示法输入子网的 IP 范围，即 **10.xxx.0.0/24**。按原样保留此**地址前缀**，并选择 256 作为**地址数**。
1. 对子网访问控制表 (ACL) 选择 **VPC 缺省值**。您可以稍后配置入站和出站规则。
1. 将**公共网关**切换到**已连接**。 
1. 单击**创建子网**以进行供应。

### 创建和配置防御安全组

接下来创建安全组，并配置防御 VSI 的入站规则。

1. 导航至**安全组**并单击**新建安全组**。输入 **vpc-secure-bastion-sg** 作为名称，并选择您的 VPC。 
2. 现在，通过在入站部分中单击**添加规则**来创建以下入站规则。这些规则允许 SSH 访问和 Ping (ICMP) 操作。
 
	**入站规则：**
	<table>
	   <thead>
	      <tr>
	         <td><strong>源</strong></td>
	         <td><strong>协议</strong></td>
	         <td><strong>值</strong></td>
	      </tr>
	   <tbody>
	      <tr>
	         <td>任何 - 0.0.0.0/0</td>
	         <td>TCP</td>
	         <td>从：<strong>22</strong> 到 <strong>22</strong></td>
	      </tr>
         <tr>
            <td>任何 - 0.0.0.0/0</td>
	         <td>ICMP</td>
	         <td>类型：<strong>8</strong>，代码：<strong>留空</strong></td>
         </tr>
	   </tbody>
	</table>

   要进一步增强安全性，入站流量可以限制为公司网络或者典型的家庭网络。您可以运行 `curl ipecho.net/plain ; echo` 以获得网络的外部 IP 地址，并改为使用该地址。
   {:tip }

### 创建防御实例
子网和安全组已经就位，接下来创建防御虚拟服务器实例。

1. 在左侧窗格的**子网**下，选择 **vpc-secure-bastion-subnet**。
2. 单击**连接的实例数**并在自己的 VPC 下供应名称为 **vpc-secure-vsi** 的**新实例**。选择 Ubuntu Linux 作为映像，**c-2x4**（2 个 vCPU 和 4 GB RAM）作为概要文件。
3. 选择**位置**，确保稍后使用相同的位置。
4. 要创建新的 **SSH 密钥**，请单击**新建密钥**
   * 输入 **vpc-ssh-key** 作为密钥名称。
   * 按原样保留**区域**。
   * 将现有本地 SSH 密钥的内容复制并粘贴到**公用密钥**下。  
   * 单击**添加 SSH 密钥**。
5. 在**网络接口**下，单击“安全组”旁边的**编辑**图标。 
   * 确保 **vpc-secure-subnet** 选为子网。
   * 取消选中缺省安全组，并标记 **vpc-secure-bastion-sg**。
   * 单击**保存**。
6. 单击**创建虚拟服务器实例**。
7. 打开该实例的电源后，单击 **vpc-secure-bastion-vsi** 并**保留**一个浮动 IP。

### 测试您的防御

防御的浮动 IP 地址处于活动状态后，尝试使用 **ssh** 连接到该地址：

   ```sh
   ssh -i ~/.ssh/<PRIVATE_KEY> root@<BASTION_FLOATING_IP_ADDRESS>
   ```
   {:pre}

## 使用维护访问规则配置安全组
{: #maintenance-security-group}

通过对运行中防御的访问权，继续创建用于安装和更新软件之类的维护任务的安全组。

1. 导航至**安全组**并使用下面的出站规则供应名称为 **vpc-secure-maintenance-sg** 的新安全组

   <table>
   <thead>
      <tr>
         <td><strong>目标</strong></td>
         <td><strong>协议</strong></td>
         <td><strong>值</strong> </td>
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
         <td>任何 - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>从：<strong>53</strong> 到 <strong>53</strong></td>
      </tr>
      <tr>
         <td>任何 - 0.0.0.0/0</td>
         <td>UDP</td>
         <td>从：<strong>53</strong> 到 <strong>53</strong></td>
      </tr>
   </tbody>
   </table>

   DNS 服务器请求在端口 53 上进行处理。DNS 使用 TCP 进行专区传输，使用 UDP 进行名称查询，查询可以是定期（主要）或逆向的。HTTP 请求位于端口 80 和 443 上。
   {:tip }

2. 接下来，添加此**入站**规则，以允许从防御主机进行 SSH 访问。

   <table>
	   <thead>
	      <tr>
	         <td><strong>源</strong></td>
	         <td><strong>协议</strong></td>
	         <td><strong>值</strong> </td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>类型：<strong>安全组</strong> - 名称：<strong>vpc-secure-bastion-sg</strong></td>
	         <td>TCP</td>
	         <td>从：<strong>22</strong> 到 <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>

3. 创建安全组。
4. 导航至 **VPC 的所有安全组**，然后选择 **vpc-secure-sg**。
5. 最后，编辑安全组，并添加以下**出站**规则。

   <table>
	   <thead>
	      <tr>
	         <td><strong>目标</strong></td>
	         <td><strong>协议</strong></td>
	         <td><strong>值</strong> </td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>类型：<strong>安全组</strong> - 名称：<strong>vpc-secure-maintenance-sg</strong></td>
	         <td>TCP</td>
	         <td>从：<strong>22</strong> 到 <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>


## 使用防御主机访问 VPC 中的其他实例 
{: #bastion-host-access-instances}

 在本部分中，您将使用虚拟服务器实例和安全组创建专用子网。缺省情况下，VPC 中创建的任何子网都是专用的。

如果您想要连接的 VPC 中已经拥有虚拟服务器实例，那么您可以跳过下面三个部分，从[将虚拟服务器实例添加到维护安全组](#add-vsi-to-maintenance)开始。

### 创建子网
{: #create-private-subnet}

新建子网：

1. 单击左侧窗格的**网络**下的**子网**，然后单击**新建子网**。  
   * 输入 **vpc-secure-private-subnet** 作为名称，然后选择您创建的 VPC。  
   * 选择位置。  
   * 以 CIDR 表示法输入子网的 IP 范围，即 **10.xxx.1.0/24**。按原样保留此**地址前缀**，并选择 256 作为**地址数**。
1. 对子网访问控制表 (ACL) 选择 **VPC 缺省值**。您可以稍后配置入站和出站规则。
1. 将**公共网关**切换到**已连接**。 
1. 单击**创建子网**以进行供应。

### 创建安全组

新建安全组：  
1. 单击“网络”下的**安全组**，然后单击**新建安全组**。  
2. 输入 **vpc-secure-private-sg** 作为名称，然后选择先前创建的 VPC。   
3. 单击**创建安全组**。  

### 创建虚拟服务器实例

在新创建的子网中创建虚拟服务器实例：

1. 单击**子网**下的专用子网。
2. 单击**连接的实例**，然后单击**新建实例**。
3. 输入唯一名称 **vpc-secure-private-vsi**，选择先前创建的 VPC，然后选择与之前相同的**位置**。
4. 选择 **Ubuntu Linux** 映像，单击**所有概要文件**，在**计算**下，选择带有 2 个 vCPU 和 4 GB RAM 的 **c-2x4**。
5. 对于 **SSH 密钥**，选取您先前为防御创建的 SSH 密钥。
6. 在**网络接口**下，单击“安全组”旁边的**编辑**图标。   
   * 选择 **vpc-secure-private-subnet** 作为子网。  
   * 取消选中缺省安全组，并激活 **vpc-secure-private-sg**。  
   * 单击**保存**。  
7. 单击**创建虚拟服务器实例**。  


### 将虚拟服务器添加到维护安全组
{: #add-vsi-to-maintenance}

对于服务器上的管理工作，您必须将特定虚拟服务器与维护安全组关联。接下来，您将启用维护，登录到专用服务器，更新软件包信息，然后再次解除关联安全组。

接下来为服务器启用维护安全组。

1. 导航至**安全组**，并选择 **vpc-secure-maintenance-sg** 安全组。  
2. 单击**连接的接口**，然后单击**编辑接口**。  
3. 展开虚拟服务器实例，并激活**接口**列中**主要**旁的选项。
4. 单击**保存**以应用更改。

### 连接到实例

要使用实例的**专用 IP** 通过 SSH 登录到该实例，您将使用防御主机作为**跳板机**。

1. 在**虚拟服务器实例**下获得虚拟服务器实例的专用 IP 地址。
2. 使用带 `-J` 的 ssh 命令，通过先前使用的防御**浮动 IP** 地址和**网络接口**下显示的服务器**专用 IP** 地址来登录到服务器。

   ```sh
   ssh -J root@<BASTION_FLOATING_IP_ADDRESS> root@<PRIVATE_IP_ADDRESS>
   ```
   {:pre}
   
   在 OpenSSH V7.3+ 中支持 `-J` 标志。在较旧版本中，`-J` 不可用。在此情况下，最安全、最直接的方式是使用 ssh 的 stdio 转发 (`-W`) 方式使连接“弹跳”通过防御主机。例如，`ssh -o ProxyCommand="ssh -W %h:%p root@<BASTION_FLOATING_IP_ADDRESS" root@<PRIVATE_IP_ADDRESS>`
   {:tip }

### 安装软件并执行维护任务

连接后，您可以在专用子网的虚拟服务器上安装软件，或者执行维护任务。

1. 首先，更新软件包信息：
   ```sh
   apt-get update
   ```
   {:pre}
2. 安装所需的软件，例如 Nginx、MySQL 或 IBM Db2。

完成后，使用 `exit` 命令断开服务器的连接。 

要允许因特网用户的 HTTP/HTTPS 请求，请将**浮动 IP** 分配给专用子网中的 VSI，并在专用 VSI 的安全组中通过入站规则打开所需端口（80 - HTTP 和 443 - HTTPS）。
{:tip}

### 禁用维护安全组

完成软件安装或维护后，您应从维护安全组除去虚拟服务器，以使其保持隔离。

1. 导航至**安全组**，并选择 **vpc-secure-maintenance-sg** 安全组。  
2. 单击**连接的接口**，然后单击**编辑接口**。  
3. 展开虚拟服务器实例，并取消选中**接口**列中**主要**旁的选项。
4. 单击**保存**以应用更改。

## 除去资源
{: #removeresources}

1. 切换到**虚拟服务器实例**并**删除**实例。实例将被删除，其状态将在一段时间内保留为**正在删除**。确保时不时地刷新浏览器。
2. 删除 VSI 之后，切换到**子网**并删除子网。
4. 删除子网后，切换到**虚拟私有云**选项卡，并删除您的 VPC。

使用控制台时，您可能需要刷新浏览器以在删除资源后查看更新后的状态信息。
{:tip}

## 相关内容
{: #related}

* [Virtual Private Cloud 中的专用和公共子网](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend)
