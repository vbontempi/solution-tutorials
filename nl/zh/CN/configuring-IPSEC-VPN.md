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

# 通过 VPN 连接到安全专用网络中
{: #configuring-IPSEC-VPN}

我们常常需要在远程网络环境和 {{site.data.keyword.Bluemix_notm}} 专用网络上的服务器之间创建专用连接。通常情况下，此连接支持 {{site.data.keyword.Bluemix_notm}} 上的混合工作负载、数据传输、专用工作负载或系统管理。站点到站点的虚拟专用网 (VPN) 隧道是保护网络之间连接安全的常用方法。 

{{site.data.keyword.Bluemix_notm}} 提供了用于站点到站点的数据中心连接的若干选项，可以是使用公用因特网上的 VPN 或者通过私有专用网络连接。 

请参阅 [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
以获取有关 {{site.data.keyword.Bluemix_notm}} 的专用安全网络链路的更多详细信息。公用因特网上的 VPN 提供了成本更低的选项，只是没有带宽保证。 

有两个合适的 VPN 选项，可用于通过公用因特网连接到 {{site.data.keyword.Bluemix_notm}} 上供应的服务器：

-	[IPSEC VPN]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[虚拟路由器设备 VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

本教程演示了使用虚拟路由器设备 (VRA) 将客户机数据中心的子网连接到 {{site.data.keyword.Bluemix_notm}} 专用网络上安全子网的站点到站点 IPSec VPN 的设置。 

此示例基于[使用安全专用网络隔离工作负载](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)教程进行构建。它使用站点到站点 IPSec VPN、GRE 隧道和静态路由。使用动态路由（BGP 等）和 VTI 隧道的更复杂 VPN 配置可以在[补充 VRA 文档](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)中找到。
{:shortdesc}

## 目标
{: #objectives}

- 记录 IPSec VPN 的配置参数
- 在虚拟路由器设备上配置 IPSec VPN
- 通过 GRE 隧道路由流量

## 使用的服务
{: #products}

本教程使用以下 {{site.data.keyword.Bluemix_notm}} 服务： 

* [虚拟路由器设备 VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

本教程可能会发生成本。VRA 仅在按月支付的定价套餐中可用。

## 体系结构
{: #architecture}

<p style="text-align: center;">

  ![体系结构](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	记录 VPN 配置
2.	在 VRA 上创建 IPSec VPN
3.	配置数据中心 VPN 和隧道
4.	创建 GRE 隧道
5.	创建静态 IP 路由
6.	配置防火墙 

## 开始之前
{: #prereqs}

本教程将[使用安全专用网络隔离工作负载](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)教程中的安全专用隔离区连接到您的数据中心。后一个教程必须先完成。

## 记录 VPN 配置
{: #Document_VPN}

配置数据中心和 {{site.data.keyword.Bluemix_notm}} 之间的 IPSec VPN 站点到站点链路需要与您的现场网络团队协作，以确定许多配置参数、隧道类型以及 IP 路由信息。此参数必须与正常运作的 VPN 连接完全一致。通常，您的现场网络团队将指定配置以匹配协定的公司标准，并为您提供数据中心 VPN 网关的必需 IP 地址以及可以访问的子网地址范围。

在开始设置 VPN 之前，VPN 网关的 IP 地址和 IP 网络子网范围必须加以确定，并可用于数据中心 VPN 配置和 {{site.data.keyword.Bluemix_notm}} 中的安全专用网络隔离区。下图进行了说明，其中安全隔离区中的 APP 专区将通过 IPSec 隧道连接到客户机数据中心的“DC IP 子网”中的系统。   

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

以下参数必须在配置 VPN 的 {{site.data.keyword.Bluemix_notm}} 用户和客户机数据中心的网络团队之间进行协定并记录下来。在此示例中，远程和本地隧道 IP 地址分别设置为 192.168.10.1 和 192.168.10.2。经现场网络团队同意，可以使用任意子网。 

|项|描述|
|:------ |:--- | 
| &lt;ike group name&gt; | 指定给用于连接的 IKE 组的名称。|
| &lt;ike encryption&gt; | {{site.data.keyword.Bluemix_notm}} 和客户机数据中心之间要使用的协定 IKE 加密标准，通常为 *aes256*。|
| &lt;ike hash&gt; | {{site.data.keyword.Bluemix_notm}} 和客户机数据中心之间协定的 IKE 散列，通常为 *sha1*。|
| &lt;ike-lifetime&gt; | 客户机数据中心的 IKE 生命周期，通常为 *3600*。|
| &lt;esp group name&gt; | 指定给用于连接的 ESP 组的名称。|
| &lt;esp encryption&gt; | {{site.data.keyword.Bluemix_notm}} 和客户机数据中心之间协定的 ESP 加密标准，通常为 *aes256*。|
| &lt;esp hash&gt; | {{site.data.keyword.Bluemix_notm}} 和客户机数据中心之间协定的 ESP 散列，通常为 *sha1*。|
| &lt;esp-lifetime&gt; | 客户机数据中心的 ESP 生命周期，通常为 *1800*。|
| &lt;DC VPN Public IP&gt;  | 客户机数据中心内 VPN 网关 的面向因特网的公共 IP 地址。| 
| &lt;VRA Public IP&gt; |VRA 的公共 IP 地址。|
| &lt;Remote tunnel IP\/24&gt; |分配给 IPSec 隧道远程端的 IP 地址。范围中与 IP Cloud 或客户机数据中心不冲突的 IP 地址对。|
| &lt;Local tunnel IP\/24&gt; |分配给 IPSec 隧道本地端的 IP 地址。|
| &lt;DC Subnet/CIDR&gt; |要在客户机数据中心和 CIDR 中访问的子网的 IP 地址。|
| &lt;App Zone subnet/CIDR&gt; |VRA 创建教程中的 APP 专区子网的网络 IP 地址和 CIDR。| 
| &lt;Shared-Secret&gt; |要在 {{site.data.keyword.Bluemix_notm}} 和客户机数据中心之间使用的共享加密密钥。|

## 在 VRA 上配置 IPSec VPN
{: #Configure_VRA_VPN}

要在 {{site.data.keyword.Bluemix_notm}} 上创建 VPN，命令和需要更改的所有变量都在下面使用 &lt; &gt; 进行突出显示。对于需要更改的每一行，将按行识别更改。值来自表格。 

1. 通过 SSH 登录到 VRA 并进入 `[edit]` 方式。
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2. 创建因特网密钥交换 (IKE) 组。
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike encryption>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike hash>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lifetime>
   ```
   {: codeblock}
3. 创建封装安全性有效负载 (ESP) 组
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp encryption>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp hash>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lifetime>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. 定义站点到站点连接
   ```
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication mode pre-shared-secret
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication pre-shared-secret <Shared-Secret>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  connection-type initiate
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  ike-group <ike group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  local-address <VRA Public IP>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  default-esp-group <esp group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1 protocol gre
   commit
   save
   ```
   {: codeblock}

## 配置数据中心 VPN 和隧道
{: #Configure_DC_VPN}

1. 客户机数据中心的网络团队将配置相同的 IPSec VPN 连接，但会交换 &lt;DC VPN Public IP&gt; 和 &lt;VRA Public IP&gt;，交换本地和远程隧道地址，并交换 &lt;DC Subnet/CIDR&gt; 和 &lt;App Zone subnet/CIDR&gt; 参数。客户机数据中心内的具体配置命令将取决于 VPN 的供应商。
1. 在继续之前，验证 DC VPN 网关的公共 IP 地址是否可通过因特网访问：
   ```
   ping <DC VPN Public IP>
   ```
   {: codeblock}
2. 完成数据中心的 VPN 配置时，IPSec 链路应该会自动出现。验证该链路是否已建立，以及状态是否显示存在一个或更多活动的 IPSec 隧道。通过数据中心验证 VPN 的两端是否都显示活动的 IPSec 隧道。
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. 如果未创建该链路，请使用 debug 命令验证本地和远程地址是否已正确指定，以及其他参数是否符合预期：
   ``` bash
   show vpn debug
   ```
   {: codeblock}

输出中应该会显示行 `peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......`。如果此行不存在，或者显示“CONNECTING”，那么 VPN 配置中存在错误。  

## 定义 GRE 隧道 
{: #Define_Tunnel}

1. 在 VRA 编辑方式下创建 GRE 隧道。
   ```
   set interfaces tunnel tun0 address <Local tunnel IP/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <VRA Public IP>
   set interfaces tunnel tun0 remote-ip <DC VPN Public IP>
   commit
   ```
   {: codeblock}
2. 隧道的两端已配置后，隧道应该会自动出现。通过 VRA 命令行检查隧道的运行状态。
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   第一个命令应该会显示隧道的状态和链路为 `u/u` (UP/UP)。第二个命令将显示有关隧道的更多详细信息，并显示已传输和接收流量。
3. 验证流量是否通过隧道流动
   ```bash
   ping <Remote tunnel IP>
   ```
   {: codeblock}
  存在 `ping` 流量时，应该会看到 `show interfaces tunnel tun0` 上的 TX 和 RX 计数递增。
4. 如果流量未流动，那么可使用 `monitor interface` 命令观察每个接口上看到的流量。接口 `tun0` 显示通过隧道的内部流量。接口 `dp0bond1` 将显示进出远程 VPN 网关的封装流量流。
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

如果未看到返回流量，那么数据中心网络团队将需要监视 VPN 中的流量流和远程站点上的隧道接口，以将该问题控制在本地。 

## 创建静态 IP 路由
{: #Define_Routing}

创建 VRA 路由以通过隧道将流量定向到远程子网。

1. 在 VRA 编辑方式下创建静态路由。
   ```
   set protocols static route <DC Subnet/CIDR>  next-hop <Remote tunnel IP>
   ```
   {: codeblock}   
2. 通过 VRA 命令行查看 VRA 路由表。此时，由于没有防火墙规则来允许流量通过隧道传输，所以没有流量流经路由。在任一端启动的流量都需要防火墙规则。
   ```bash
   show ip route
   ```
   {: codeblock}

## 配置防火墙
{: #Configure_firewall}

1. 为允许的 icmp 流量和 tcp 端口创建资源组。 
   ```
   set res group icmp-group icmpgrp type 8
   set res group icmp-group icmpgrp type 11
   set res group icmp-group icmpgrp type 3

   set res group port tcpports port 22
   set res group port tcpports port 80
   set res group port tcpports port 443
   commit
   ```
   {: codeblock}
2. 在 VRA 编辑方式下为进入远程子网的流量创建防火墙规则。
   ```
   set security firewall name APP-TO-TUNNEL default-action drop
   set security firewall name APP-TO-TUNNEL default-log

   set security firewall name APP-TO-TUNNEL rule 100 action accept
   set security firewall name APP-TO-TUNNEL rule 100 protocol tcp
   set security firewall name APP-TO-TUNNEL rule 100 destination port tcpports

   set security firewall name APP-TO-TUNNEL rule 200 protocol icmp
   set security firewall name APP-TO-TUNNEL rule 200 icmp group icmpgrp
   set security firewall name APP-TO-TUNNEL rule 200 action accept
   commit
   ```
   {: codeblock}

   ```
   set security firewall name TUNNEL-TO-APP default-action drop
   set security firewall name TUNNEL-TO-APP default-log

   set security firewall name TUNNEL-TO-APP rule 100 action accept
   set security firewall name TUNNEL-TO-APP rule 100 protocol tcp
   set security firewall name TUNNEL-TO-APP rule 100 destination port tcpports

   set security firewall name TUNNEL-TO-APP rule 200 protocol icmp
   set security firewall name TUNNEL-TO-APP rule 200 icmp group icmpgrp
   set security firewall name TUNNEL-TO-APP rule 200 action accept 
   commit
   ```
   {: codeblock}
2. 为隧道创建专区，并为任一专区中启动的流量关联防火墙。
   ```
   set security zone-policy zone TUNNEL description "GRE Tunnel"
   set security zone-policy zone TUNNEL default-action drop
   set security zone-policy zone TUNNEL interface tun0

   set security zone-policy zone TUNNEL to APP firewall TUNNEL-TO-APP
   set security zone-policy zone APP to TUNNEL firewall APP-TO-TUNNEL
   commit
   save
   ```
   {: codeblock}
3. 要验证两端的防火墙和路由是否配置正确，以及现在是否允许 ICMP 和 TCP 流量，请对远程子网的网关地址执行 ping 操作，先通过 VRA 命令行执行，如果成功，再通过登录到 VSI 来执行。
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}
4. 如果通过 VRA 命令行执行 ping 操作失败，请验证在响应隧道接口上的 ping 请求时是否看到 ping 回复。
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
   没有响应指示防火墙规则或数据中心的路由出现问题。如果在监视器输出中出现回复，但是 ping 命令超时，请检查本地 VRA 防火墙规则的配置。
5. 如果通过 VSI 执行 ping 操作失败，这表明 VRA 防火墙规则、VRA 或 VSI 配置中的路由出现问题。完成先前的步骤，以确保请求已发送到数据中心，并且从数据中心看到响应。监视本地 VLAN 上的流量和检查防火墙日志有助于确定问题是出在路由还是防火墙上。
   ```
   monitor interfaces bonding dp0bond0.<VLAN ID>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

这样完成了从安全专用网络隔离区设置 VPN 的过程。本系列的其他教程阐述隔离区可以如何访问公用因特网上的服务。

## 除去资源
{:removeresources}

除去本教程中创建的资源所需执行的步骤。

VRA 采用按月付费套餐。取消也不会退费。建议您仅在下个月不再需要此 VRA 时取消。如果需要双 VRA 高可用性集群，那么可以在[网关详细信息](https://{DomainName}/classic/network/gatewayappliances)页面上升级此单个 VRA。
{:tip}  

1. 取消任何虚拟服务器或裸机服务器
2. 取消 VRA
3. 通过支持凭单取消任何其他 VLAN。 

## 相关内容
{:related}
- [IBM 虚拟路由器设备](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [静态和可移植 IP 子网](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Vyatta 文档](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS IPsec Site-to-Site VPN Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS Tunnels Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
