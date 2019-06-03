---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-08"
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

# 使用安全专用网络隔离工作负载
{: #secure-network-enclosure}

在公共云上，针对隔离且安全的专用网络环境的需要对于 IaaS 应用程序部署模型来说至关重要。防火墙、VLAN、路由和 VPN 都是创建隔离的专用环境所必需的组件。通过这种隔离，虚拟机和裸机服务器能够安全地部署在复杂的多层应用程序拓扑中，同时能防范公用因特网上的风险。  

本教程重点阐述了如何在 {{site.data.keyword.Bluemix_notm}} 上配置[虚拟路由器设备](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-) (VRA)，以创建安全专用网络（隔离区）。VRA 网关设备在一个自我管理的软件包中提供防火墙、VPN 网关、网络地址转换 (NAT) 和企业级路由。在本教程中，VRA 用于显示如何在 {{site.data.keyword.Bluemix_notm}} 上创建封闭、隔离的网络环境。在此隔离区中，可以使用为人熟悉且众所周知的 IP 路由、VLAN、IP 子网、防火墙规则、虚拟服务器和裸机服务器技术来创建应用程序拓扑。  

{:shortdesc}

本教程是在 {{site.data.keyword.Bluemix_notm}} 上进行经典联网的起点，不应将其视为按原样使用的生产功能。可以考虑的其他功能包括：
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [硬件防火墙设备](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* [IPSec VPN](https://{DomainName}/catalog/infrastructure/ipsec-vpn)，用于安全连接到数据中心。
* 高可用性，通过集群的 VRA 和双上行链路实现。
* 对安全性事件的日志记录和审计。

## 目标 
{: #objectives}

* 部署虚拟路由器设备 (VRA)
* 定义 VLAN 和 IP 子网，以部署虚拟机和裸机服务器
* 使用防火墙规则保护 VRA 和隔离区

## 使用的服务
{: #products}

本教程使用以下 {{site.data.keyword.Bluemix_notm}} 服务：
* [Virtual Router Appliance](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)

本教程可能会发生成本。VRA 仅在按月支付的定价套餐中可用。

## 体系结构
{: #architecture}

<p style="text-align: center;">

  ![体系结构](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. 配置 VPN
2. 部署 VRA 
3. 创建虚拟服务器
4. 通过 VRA 路由访问
5. 配置隔离区防火墙
6. 定义 APP 专区
7. 定义 INSIDE 专区

## 开始之前
{: #prereqs}

### 配置 VPN 访问

在本教程中，创建的网络隔离区不会在公用因特网上显示。VRA 和任何服务器只可通过专用网络进行访问，并且将使用 VPN 进行连接。 

1. [确保 VPN 访问已启用](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     您应该是**主用户**才能启用 VPN 访问，或者联系主用户来获取访问权。
     {:tip}
2. 通过在[用户列表](https://{DomainName}/iam#/users)中选择用户来获取 VPN 访问凭证。
3. 通过 [Web 界面](https://www.softlayer.com/VPN-Access)登录到 VPN，或者使用适用于 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 或 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 的 VPN 客户机。

   对于 VPN 客户机，使用 [VPN Web 访问页面](https://www.softlayer.com/VPN-Access)中单个数据中心 VPN 访问点的 FQDN（格式为 *vpn.xxxnn.softlayer.com*）作为网关地址。
   {:tip}

### 检查帐户许可权

请联系基础架构主用户以获取以下许可权：
- **快速许可权** - 基本用户
- **网络**许可权，用于创建和配置隔离区；需要所有网络许可权。 
- **服务**许可权，用于管理 SSH 密钥

### 上传 SSH 密钥

通过门户网站[上传 SSH 公用密钥](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial)，此密钥将用于访问和管理 VRA 和专用网络。  

### 设定目标数据中心

选择要部署安全专用网络的 {{site.data.keyword.Bluemix_notm}} 数据中心。 

### 订购 VLAN

要在目标数据中心内创建专用隔离区，必须首先分配服务器所需的专用 VLAN。第一个专用 VLAN 和第一个公用 VLAN 是免费的。如果需要更多 VLAN 用于支持多层应用程序拓扑，这些 VLAN 会收费。 

为了确保在同一数据中心路由器上有足够的 VLAN 可用并且可以与 VRA 相关联，建议订购 VLAN。请参阅[订购 VLAN](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans)。

## 供应虚拟路由器设备
{: #VRA}

第一步是部署 VRA；VRA 为专用网络隔离区提供 IP 路由和防火墙。{{site.data.keyword.Bluemix_notm}} 提供的面向公共的传输 VLAN 可从隔离区访问因特网，借助网关和可选的硬件防火墙，可以创建从公用 VLAN 到安全专用隔离区 VLAN 的连接。在此解决方案教程中，虚拟路由器设备 (VRA) 提供了此网关和防火墙周界。 

1. 在“目录”中，选择[网关设备](https://{DomainName}/gen1/infrastructure/provision/gateway)。
3. 在**网关供应商**部分中，选择 AT&T。可以选择“最高 20 Gbps”或“最高 2 Gbps”上行速度。
4. 在**主机名**部分中，输入新 VRA 的主机名和域。
5. 如果选中**高可用性**复选框，那么将使用 VRRP 使两个 VRA 设备以活动/备份设置工作。
6. 在**位置**部分中，选择需要 VRA 的“位置”和 **Pod**。
7. 选择“单处理器”或“双处理器”。您将获得服务器的列表。通过单击相应单选按钮来选择服务器。 
8. 选择 **RAM** 量。对于生产环境，建议使用至少 64 GB RAM。对于测试环境，至少为 8 GB。
9. 选择 **SSH 密钥**（可选）。此 SSH 密钥将安装在 VRA 上，因此可以使用用户 vyatta 通过此密钥来访问 VRA。
10. 硬盘驱动器。请保留缺省值。
11. 在**上行端口速度**部分中，选择满足您需求的速度、冗余和专用和/或公共接口的组合。
12. 在**附加组件**部分中，保留缺省值。如果要在公共接口上使用 IPv6，请选择 IPv6 地址。

在右侧，可以看到**订单摘要**。选中_我已阅读并同意下面列出的第三方服务协议：_复选框，然后单击**创建**按钮。这将部署网关。

[设备列表](https://{DomainName}/classic/devices)几乎会立即显示该 VRA，并带有**时钟**符号，指示此设备上正在进行事务处理。在 VRA 创建完成之前，**时钟**符号会一直存在，除了能查看设备详细信息外，无法对设备执行任何配置操作。
{:tip}

### 查看部署的 VRA

1. 检查新的 VRA。在[基础架构仪表板](https://{DomainName}/classic)上，选择左侧窗格中的**网络**，然后选择**网关设备**以转至[网关设备](https://{DomainName}/classic/network/gatewayappliances)页面。在**网关**列中选择新创建的 VRA 的名称，以转至“网关详细信息”页面。 ![](images/solution33-secure-network-enclosure/Gateway-detail.png)

2. 记下 VRA 的`专用`和`公共` IP 地址，以供未来使用。

## 初始 VRA 设置
{: #initial_VRA_setup}

1. 在工作站中，使用缺省 **vyatta** 帐户通过 SSL VPN 登录到 VRA，并接受 SSH 安全提示。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   如果 SSH 提示输入密码，说明 SSH 密钥未包含在构建中。请通过 [Web 浏览器](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui)使用 `VRA 专用 IP 地址`来访问 VRA。密码位于[软件密码](https://{DomainName}/classic/devices/passwords)页面中。在**配置**选项卡上，选择“系统/登录/vyatta”分支，并添加所需的 SSH 密钥。
   {:tip}

   设置 VRA 需要使用 `configure` 命令将 VRA 置于 \[编辑\] 方式。在`编辑`方式下，提示符会从 `$` 更改为 `#`。成功更改 VRA 配置后，可以使用 `compare` 命令查看更改，并使用 `validate` 命令检查更改。通过使用 `commit` 命令提交更改，更改将应用于正在运行的配置，并自动保存到启动配置中。


   {:tip}
2. 通过仅允许 SSH 登录来增强安全性。既然已通过专用网络成功执行 SSH 登录，那就可利用用户标识/密码认证来禁用访问。 
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
   在本教程中，现在假定在输入 `configure` 后，在 `edit` 提示符处输入了所有 VRA 命令。
3. 查看初始配置。
   ```
   show
   ```
   {: codeblock}

   VRA 已针对 {{site.data.keyword.Bluemix_notm}} IaaS 环境进行预配置。这包括以下内容：
   - NTP 服务器
   - 名称服务器
   - SSH
   - HTTPS Web 服务器 
   - 缺省时区 - 美国/芝加哥
4. 根据需要设置本地时区。使用 Tab 键进行自动补全将列出潜在的时区值。
   ```
   set system time-zone <timezone>
   ```
   {: codeblock}
5. 设置 ping 行为。为了帮助路由和防火墙故障诊断，未禁用 ping。 
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. 启用有状态防火墙操作。缺省情况下，VRA 防火墙是无状态的。 
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. 落实并自动保存对启动配置的更改。 
   ```
   commit
   ```
   {: codeblock}

## 订购第一个虚拟服务器
{: #order_virtualserver}

现在创建的虚拟服务器用于帮助诊断 VRA 配置错误。验证是否能通过 {{site.data.keyword.Bluemix_notm}} 专用网络对 VSI 进行成功访问，然后在稍后的步骤中通过 VRA 路由对该 VSI 的访问。 

1. 订购[虚拟服务器](https://{DomainName}/catalog/infrastructure/virtual-server-group)。  
2. 选择**公共虚拟服务器**，然后继续。
3. 在订购页面上：
   - 将**计费**设置为**按小时**。
   - 设置 *VSI 主机名*和*域名*。此域名不用于路由和 DNS，但应与网络命名标准保持一致。 
   - 将**位置**设置为与 VRA 相同。
   - 将**概要文件**设置为 **C1.1x1**。
   - 添加早先指定的 **SSH 密钥**。
   - 将**操作系统**设置为 **CentOS 7.x - 最低**。
   - 在**上行端口速度**中，必须将网络接口从缺省值*公共和专用*更改为仅指定**专用网络上行链路**。这可确保新服务器无法直接访问因特网，并且访问由 VRA 上的路由和防火墙规则进行控制。
   - 将**专用 VLAN** 设置为早先订购的专用 VLAN 的 VLAN 标识。
4. 单击勾选框以接受“第三方”服务协议，然后单击**创建**。
5. 在[设备](https://{DomainName}/classic/devices)页面上或通过电子邮件监视完成情况。 
6. 记下 VSI 的*专用 IP 地址*以用于后续步骤，并在**设备详细信息**页面的**网络**部分下，确认 VSI 是否分配给正确的 VLAN。如果没有，请删除此 VSI，然后在正确的 VLAN 上创建新的 VSI。 
7. 在本地工作站中，通过 VPN 使用 ping 和 SSH 来验证是否可以通过 {{site.data.keyword.Bluemix_notm}} 专用网络成功访问 VSI。
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## 通过 VRA 路由 VLAN 访问
{: #routing_vlan_via_vra}

此时虚拟服务器的专用 VLAN 就已经由 {{site.data.keyword.Bluemix_notm}} 管理系统关联到此 VRA。在此阶段，仍可通过 {{site.data.keyword.Bluemix_notm}} 专用网络上的 IP 路由来访问 VSI。现在，将通过 VRA 路由子网来创建安全专用网络，并通过确认现在无法访问 VSI 来进行验证。 

1. 通过[网关设备](https://{DomainName}/classic/network/gatewayappliances)页面继续设置 VRA 的“网关详细信息”，并在该页面的下半部分找到**关联的 VLAN** 部分。关联的 VLAN 将在此处列出。 
2. 如果此时需要添加其他 VLAN，请导航至**关联 VLAN** 部分。应启用下拉框*选择 VLAN*，并可以选择其他供应的 VLAN。![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png)

   如果未显示符合条件的 VLAN，说明在与 VRA 相同的路由器上没有可用的 VLAN。这将需要提交[支持凭单](https://{DomainName}/unifiedsupport/cases/add)，以在与 VRA 相同的路由器上请求专用 VLAN。
   {:tip}
5. 选择要与 VRA 关联的 VLAN，然后单击“保存”。初始 VLAN 关联可能需要几分钟才能完成。完成后，该 VLAN 应该会显示在**关联的 VLAN** 标题下。 

在此阶段，VLAN 和关联的子网不通过 VRA 进行保护或路由，并且 VSI 可通过 {{site.data.keyword.Bluemix_notm}} 专用网络进行访问。VLAN 的状态将显示为*已绕过*。

4. 选择右侧列中的**操作**，然后选择**路由 VLAN** 以通过 VRA 路由 VLAN/子网。这将需要几分钟时间。刷新屏幕后，该 VLAN 会显示为*已路由*。 
5. 选择 [VLAN 名称](https://{DomainName}/classic/network/vlans/)以查看 VLAN 详细信息。可以看到供应的 VSI 以及分配的主 IP 子网。记下“专用 VLAN 标识 \<nnnn\>”（在此示例中为 1199），因为在后面的步骤中将用到此标识。 
6. 选择[子网](https://{DomainName}/classic/network/subnets)以查看 IP 子网详细信息。记下子网网络、网关地址和 CIDR (/26)，因为进一步 VRA 配置需要这些信息。在专用网络上供应有 64 个主 IP 地址，要找到网关地址，可能需要选择第 2 页或第 3 页。 
7. 验证子网/VLAN 是否路由到 VRA，以及是否**无法**在工作站中通过管理网络使用 `ping` 来访问 VSI。 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}

这就通过 {{site.data.keyword.Bluemix_notm}} 控制台完成了 VRA 的设置。现在，通过 SSH 直接在 VRA 上执行配置隔离区和 IP 路由的其他工作。 

## 配置 IP 路由和安全隔离区
{: #vra_setup}

落实 VRA 配置后，将更改运行配置，并且更改会自动保存到启动配置中。

如果希望返回到先前的有效配置，缺省情况下可以查看、比较或复原最后 20 个落实点。有关落实和保存配置的更多详细信息，请参阅 [Vyatta 网络操作系统基本系统配置指南](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)。
   ```bash
   show system commit 
   rollback n
   compare
   ```
   {: codeblock}

### 配置 VRA IP 路由

配置 VRA 虚拟网络接口以从 {{site.data.keyword.Bluemix_notm}} 专用网络路由到新子网。  

1. 通过 SSH 登录到 VRA。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}
2. 使用早先步骤中记录的专用 VLAN 标识、子网网关 IP 地址和 CIDR 来创建新的虚拟接口。CIDR 通常为 /26。 
   ```
   set interfaces bonding dp0bond0 vif <VLAN ID> address <Subnet Gateway IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   使用 **`<Subnet Gateway IP>`** 地址至关重要。这通常比子网地址的起始地址大 1。输入无效的网关地址将导致错误：`配置路径：接口绑定 dp0bond0 vif xxxx 地址 [x.x.x.x] 无效`。请更正命令并重新输入。可以在“网络”>“IP 管理”>“子网”中查找该地址。单击需要知道网关地址的子网。“列表”中带有描述**网关**的第二个条目就是要作为 <Subnet Gateway IP>/<CIDR> 输入的 IP 地址。
   {: tip}

3. 列出新的虚拟接口 (vif)： 
   ```
   show interfaces
   ```
   {: codeblock}

   下面是示例接口配置，显示了 vif 1199 和子网网关地址。
   ![](images/solution33-secure-network-enclosure/show_interfaces.png)
4. 在工作站中通过管理网络验证 VSI 是否再次可访问。 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
   
   如果无法访问 VSI，请检查 VRA IP 路由表是否按预期进行了配置。如果需要，请删除并重新创建路由。要在配置方式下运行 show 命令，可以使用 run 命令    
   ```bash
    run show ip route <Subnet Gateway IP>
   ```
   {: codeblock}

这就完成了 IP 路由配置。

### 配置安全隔离区

通过配置专区和防火墙规则，可创建安全专用网络隔离区。在继续之前，请查看有关[防火墙配置](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls)的 VRA 文档。 

将定义两个专区：
   - INSIDE：IBM 专用和管理网络
   - APP：专用网络隔离区中的用户 VLAN 和子网		

1. 定义防火墙和缺省值。
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   如果意外运行了 set 命令两次，您将收到一条消息：*“配置路径 xxxxxxxx 无效。节点已存在”*。可以将其忽略。要更改不正确的参数，需要首先使用“delete security xxxxx xxxx xxxxxx”来删除该节点。
   {:tip}
2. 创建 {{site.data.keyword.Bluemix_notm}} 专用网络资源组。此地址组定义可以访问隔离区的 {{site.data.keyword.Bluemix_notm}} 专用网络以及可从隔离区访问的网络。对安全隔离区进行访问和从安全隔离区进行访问需要两组 IP 地址，这两组地址是 SSL VPN 数据中心和 {{site.data.keyword.Bluemix_notm}} 服务网络（后端/专用网络）。[{{site.data.keyword.Bluemix_notm}} IP 范围](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges)提供了需要允许的 IP 范围的完整列表。 
   - 定义要用于 VPN 访问的数据中心的 SSL VPN 地址。从 {{site.data.keyword.Bluemix_notm}} IP 范围的 SSL VPN 部分中，选择用于数据中心或 DC 集群的 VPN 访问点。下面的示例显示了 {{site.data.keyword.Bluemix_notm}} 伦敦数据中心的 VPN 地址范围。
     ```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - 为 WDC04、DAL01 和目标数据中心定义 {{site.data.keyword.Bluemix_notm}}“服务网络（在后端/专用网络上）”的地址范围。下面的示例是 WDC04（两个地址）、DAL01 和 LON06。
     ```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. 为用户 VLAN 和子网创建 APP 专区，并为 {{site.data.keyword.Bluemix_notm}} 专用网络创建 INSIDE 专区。分配先前创建的防火墙。专区定义使用 VRA 网络接口名称来标识与每个 VLAN 关联的专区。用于创建 APP 专区的命令需要指定与早先 VRA 关联的 VLAN 的 VLAN 标识。这在下面突出显示为 `<VLAN ID>`。
   ```
   set security zone-policy zone INSIDE description "IBM Internal network"
   set security zone-policy zone INSIDE default-action drop
   set security zone-policy zone INSIDE interface dp0bond0
   set security zone-policy zone INSIDE to APP firewall INSIDE-TO-APP 

   set security zone-policy zone APP description "Application network"
   set security zone-policy zone APP default-action drop

   set security zone-policy zone APP interface dp0bond0.<VLAN ID> 
   set security zone-policy zone APP to INSIDE firewall APP-TO-INSIDE 
   ```
   {: codeblock}
4. 落实配置，然后在工作站中，使用 ping 来验证防火墙现在是否拒绝通过 VRA 流至 VSI 的流量： 
   ```
   commit
   ```
   {: codeblock}

   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
5. 针对 UDP、TCP 和 ICMP 定义防火墙访问规则。
   ```
   set security firewall name INSIDE-TO-APP rule 200 protocol icmp
   set security firewall name INSIDE-TO-APP rule 200 icmp type 8
   set security firewall name INSIDE-TO-APP rule 200 action accept 
   set security firewall name INSIDE-TO-APP rule 200 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 100 action accept 
   set security firewall name INSIDE-TO-APP rule 100 protocol tcp
   set security firewall name INSIDE-TO-APP rule 100 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 110 action accept 
   set security firewall name INSIDE-TO-APP rule 110 protocol udp
   set security firewall name INSIDE-TO-APP rule 110 source address ibmprivate
   commit

   set security firewall name APP-TO-INSIDE rule 200 protocol icmp
   set security firewall name APP-TO-INSIDE rule 200 icmp type 8
   set security firewall name APP-TO-INSIDE rule 200 action accept 
   set security firewall name APP-TO-INSIDE rule 200 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 100 action accept 
   set security firewall name APP-TO-INSIDE rule 100 protocol tcp
   set security firewall name APP-TO-INSIDE rule 100 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 110 action accept 
   set security firewall name APP-TO-INSIDE rule 110 protocol udp
   set security firewall name APP-TO-INSIDE rule 110 destination address ibmprivate
   commit
   ```
   {: codeblock}
6. 验证防火墙访问权。 
   - 确认 INSIDE 到 APP 防火墙现在是否允许来自本地计算机的 ICMP 和 UDP/TCP 流量。
     ```bash
     ping <VSI Private IP Address>
     SSH root@<VSI Private IP Address>
     ```
     {: codeblock}
   - 确认 APP 到 INSIDE 防火墙是否允许 ICMP 和 UDP/TCP 流量。使用 SSH 登录到 VSI，然后在 10.0.80.11 和 10.0.80.12 上，对其中一个 {{site.data.keyword.Bluemix_notm}} 名称服务器执行 ping 操作。
     ```bash
     SSH root@<VSI Private IP Address>
     [root@vsi  ~]# ping 10.0.80.11 
     ```
     {: codeblock}
7. 在工作站中通过 SSH 来验证是否能持续访问 VRA 管理接口。如果可持续访问，请复查并保存配置。否则，重新引导 VRA 后将恢复为某个有效配置。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   ```
   show security  
   ```
   {: codeblock}

### 调试防火墙规则

可以通过 VRA 操作命令提示符来查看防火墙日志。在此配置中，仅记录每个专区丢弃的流量，以帮助诊断防火墙配置错误。  

1. 查看防火墙日志，以了解被拒绝的流量。定期查看日志可确定 APP 专区中的服务器是以有效方式还是以错误方式在尝试联系 IBM 网络上的服务。 
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. 如果服务或服务器不可联系，且防火墙日志中未显示任何相关内容，请使用早先的 `<VLAN ID>` 来验证 VRA 网络接口上是否有来自 {{site.data.keyword.Bluemix_notm}} 专用网络的预期 ping/SSH IP 流量，或者在 VRA 接口上是否有流至 VLAN 的预期 ping/SSH IP 流量。
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN ID> traffic
   ```

## 保护 VRA
{: #securing_the_vra}

1. 应用 VRA 安全策略。缺省情况下，基于策略的防火墙分区并不保护对 VRA 本身的访问。这是通过控制平面管制 (CPP) 配置的。VRA 将基本 CPP 规则集作为模板提供。请将其合并到配置中：
   ```bash
   merge /opt/vyatta/etc/cpp.conf 
   ```
   {: codeblock}

这将创建名为 `CPP` 的新防火墙规则集，查看其他规则，并以 \[编辑\] 方式落实。 
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. 保护公共 SSH 访问。由于 Vyatta 固件目前存在待解决的问题，因此建议不要使用 `set service SSH listen-address x.x.x.x` 来限制通过公用网络进行的 SSH 管理访问。或者，可以通过 CPP 防火墙阻止 VRA 公共接口使用的公共 IP 地址范围进行外部访问。此处使用的 `<VRA Public IP Subnet>` 与 `<VRA Public IP Address>` 相同，其中最后一个八位元为零 (x.x.x.0)。 
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <VRA Public IP Subnet>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit 
   ```
   {: codeblock}
3. 通过 IBM 内部网络验证 VRA SSH 管理访问权。如果在执行落实后无法通过 SSH 来访问 VRA，那么可以通过 VRA“设备详细信息”页面上提供的 KVM 控制台中的“操作”下拉菜单来访问 VRA。

这就完成了安全专用网络隔离区的设置，通过隔离区，可保护包含 VLAN 和子网的单个防火墙专区。可以按照相同的指示信息来添加其他防火墙专区、规则、虚拟服务器和裸机服务器、VLAN 以及子网。 

## 除去资源
{: #removeresources}

在此步骤中，您将清理资源以除去上面创建的内容。

VRA 采用按月付费套餐。取消也不会退费。建议您仅在下个月不再需要此 VRA 时取消。如果需要双 VRA 高可用性集群，那么可以在[网关详细信息](https://{DomainName}/classic/network/gatewayappliances/)页面上升级此单个 VRA。
{:tip}  

- 取消任何虚拟服务器或裸机服务器
- 取消 VRA
- 通过支持凭单取消任何其他 VLAN。 

## 相关内容
{: #related}

- [IBM 虚拟路由器设备](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [静态和可移植 IP 子网](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence Platform](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Vyatta 文档](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
