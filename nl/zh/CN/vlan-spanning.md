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

# 通过 IBM 网络链接安全专用网络
{: #vlan-spanning}

随着对 Web 应用程序全球覆盖和全天候运作的需求的增加，在多个云数据中心内托管服务的需求也随之增加。跨多个位置的数据中心可以在出现地理位置故障时提供弹性，还会使工作负载更靠近全球分布的用户，从而减少等待时间并提高感知的性能。[{{site.data.keyword.Bluemix_notm}} 网络](https://www.ibm.com/cloud/data-centers/)支持用户跨数据中心和位置链接到在安全专用网络中托管的工作负载。

本教程将演示如何设置不同数据中心内托管的两个安全专用网络之间通过 {{site.data.keyword.Bluemix_notm}} 专用网络建立的专用路由 IP 连接。所有资源都由一个 {{site.data.keyword.Bluemix_notm}} 帐户拥有。该帐户使用[通过安全专用网络隔离工作负载]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)教程来部署两个专用网络，这两个专用网络使用 [VLAN 生成]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)服务通过 {{site.data.keyword.Bluemix_notm}} 专用网络进行安全链接。
{:shortdesc}

## 目标
{: #objectives}

- 在 {{site.data.keyword.Bluemix_notm}} IaaS 帐户中链接安全网络
- 设置防火墙规则以进行站点到站点的访问 
- 配置站点间的路由

## 使用的服务
{: #products}

本教程使用以下 {{site.data.keyword.Bluemix_notm}} 服务： 
* [虚拟路由器设备](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [VLAN 生成]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

本教程可能会发生成本。VRA 仅在按月支付的定价套餐中可用。

## 体系结构
{: #architecture}

<p style="text-align: center;">

  ![体系结构](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. 部署安全专用网络
2. 配置“VLAN 生成”
3. 配置专用网络之间的 IP 路由
4. 配置防火墙规则以访问远程站点

## 开始之前
{: #prereqs}

本教程基于教程[使用安全专用网络隔离工作负载]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)。开始之前，应复查该教程及其先决条件。 

## 配置安全专用网络站点
{: #private_network}

将利用教程[使用安全专用网络隔离工作负载]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)两次，以在两个不同的数据中心内实施专用网络。除了注意等待时间对将在站点之间通信的任何流量或工作负载的影响之外，对于可以利用哪两个数据中心没有任何限制。 

可以遵循[使用安全专用网络隔离工作负载]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)教程，而不更改每个所选数据中心，同时记录以下信息以供后续步骤使用。 

| 项  | Datacenter1 | Datacenter2 |
|:------ |:--- | :--- |
| 数据中心 |  |  |
| VRA 公共 IP 地址 | <DC1 VRA Public IP Address> | <DC2 VRA Public IP Address> |
| VRA 专用 IP 地址| <DC1 VRA Private IP Address> | <DC2 VRA Private IP Address> |
| VRA 专用子网 & CIDR |  |  |
| 专用 VLAN 标识| &lt;DC1 Private VLAN ID&gt;  | &lt;DC2 Private VLAN ID&gt; |
| VSI 专用 IP 地址| <DC1 VSI Private IP Address> | <DC2 VSI Private IP Address> |
| APP 专区子网 & CIDR | <DC1 APP zone subnet/CIDR> | <DC2 APP zone subnet/CIDR> |

1. 通过[网关设备](https://{DomainName}/classic/network/gatewayappliances)页面，前往每个 VRA 的“网关详细信息”页面。  
2. 找到“网关 VLAN”部分，然后单击**专用**网络上的“网关 [VLAN]( https://{DomainName}/classic/network/vlans)”以查看 VLAN 详细信息。名称应包含标识 `bcrxxx`，代表“后端客户路由器”，并且格式应为 `nnnxx.bcrxxx.xxxx`。
3. 将在**设备*部分下显示所供应的 VRA。在**子网** 部分下，记下 VRA 专用子网 IP 地址和 CIDR (/26)。该子网的类型为 primary，具有 64 个 IP。稍后的路由配置会需要这些详细信息。 
4. 回到“网关详细信息”页面，找到**关联的 VLAN** 部分，然后在关联的**专用**网络上单击 [VLAN]( https://{DomainName}/classic/network/vlans) 以创建安全网络和 APP 专区。 
5. 将在**设备*部分下显示所供应的 VSI。在**子网** 部分下，记下 VSI 子网 IP 地址和 CIDR (/26)，因为路由配置需要这些信息。此 VLAN 和子网会在两个 VRA 防火墙配置中识别为 APP 专区，并记录为 &lt;APP Zone subnet/CIDR&gt;。


## 配置“VLAN 生成” 
{: #configure-vlan-spanning}

缺省情况下，不同 VLAN 和数据中心上的服务器（和 VRA）无法通过专用网络彼此进行通信。在这些教程中，在单个数据中心内，VRA 用于将 VLAN 和子网链接到经典 IP 路由和防火墙，以创建专用网络用于跨 VLAN 进行服务器通信。虽然它们可以在同一数据中心内进行通信，但在此配置中，属于同一 {{site.data.keyword.Bluemix_notm}} 帐户的服务器却无法跨数据中心进行通信。 

[VLAN 生成]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)服务解除了对**未**与 VRA 关联的 VLAN 和子网之间通信的这一限制。请务必注意，即使启用了 VLAN 生成，与 VRA 关联的 VLAN 也只能通过其关联的 VRA 进行通信，如 VRA 防火墙和路由配置所确定。启用 VLAN 生成后，{{site.data.keyword.Bluemix_notm}} 帐户所拥有的 VRA 会通过专用网络进行连接，并可以进行通信。 

将“生成”未与 VRA 所创建的安全专用网络关联的 VLAN，从而允许这些“未关联的”VLAN 跨数据中心进行互连。这包括不同数据中心内属于同一 IBM Cloud 帐户的 VRA 网关（转移）VLAN。因此，当“VLAN 生成”启用后，会允许 VRA 跨数据中心进行通信。通过 VRA 到 VRA 的连接，VRA 防火墙和路由配置将支持安全网络中的服务器进行连接。 

启用“VLAN 生成”：

1. 前往 [VLAN]( https://{DomainName}/classic/network/vlans) 页面。
2. 在页面顶部选择**生成**选项卡
3. 为“VLAN 生成”选择“启动”单选按钮。该网络更改需要几分钟才能完成。
4. 确认这两个 VRA 现在可以通信：

   登录到数据中心 1 VRA，然后对数据中心 2 VRA 执行 ping 操作

   ```
   SSH vyatta@<DC1 VRA Private IP Address>
   ping <DC2 VRA Private IP Address>
   ```
   {: codeblock}

   登录到数据中心 2 VRA，然后对数据中心 1 VRA 执行 ping 操作
   ```
   SSH vyatta@<DC2 VRA Private IP Address>
   ping <DC1 VRA Private IP Address>
   ```
   {: codeblock}

## 配置 VRA IP 路由 
{: #vra_routing}

在每个数据中心内创建 VRA 路由，以支持两个数据中心内 APP 专区中的 VSI 进行通信。 

1. 在 VRA 编辑方式下，在数据中心 1 中创建与数据中心 2 中的 APP 专区专用子网的静态路由。
   ```
   ssh vyatta@<DC1 VRA Private IP Address>
   conf
   set protocols static route <DC2 APP zone subnet/CIDR>  next-hop <DC2 VRA Private IP>
   commit
   ```
   {: codeblock}
2. 在 VRA 编辑方式下，在数据中心 2 中创建与数据中心 1 中的 APP 专区专用子网的静态路由。
   ```
   ssh vyatta@<DC2 VRA Private IP Address>
   conf
   set protocols static route <DC1 APP zone subnet/CIDR>  next-hop <DC1 VRA Private IP>
   commit
   ```
   {: codeblock}
2. 通过 VRA 命令行查看 VRA 路由表。此时 VSI 还不能进行通信，因为不存在 APP 专区防火墙规则以允许两个 APP 专区子网之间的流量。在任一端启动的流量都需要防火墙规则。
   ```
   show ip route
   ```
   {: codeblock}

现在将显示允许 APP 专区通过 IBM 专用网络进行通信的新路由。 

## VRA 防火墙配置
{: #vra_firewall}

现有的 APP 专区防火墙规则仅配置为允许此子网与 {{site.data.keyword.Bluemix_notm}} 专用网络上的 {{site.data.keyword.Bluemix_notm}} 服务之间的流量，以及通过 NAT 进行公用因特网访问。与此 VRA 上或其他数据中心内的 VSI 关联的其他子网都会被阻止。下一步是更新与 APP-TO-INSIDE 防火墙规则相关联的 `ibmprivate` 资源组，以允许显式访问其他数据中心内的子网。 

1. 在数据中心 1 VRA 编辑命令方式下，将 <DC2 APP zone subnet>/CIDR 添加到 `ibmprivate` 资源组
   ```
   set resources group address-group ibmprivate address <DC2 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
2. 在数据中心 2 VRA 编辑命令方式下，将 <DC1 APP zone subnet>/CIDR 添加到 `ibmprivate` 资源组
   ```
   set resources group address-group ibmprivate address <DC1 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
3. 验证两个数据中心内的 VSI 现在是否可以进行通信
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}

   如果 VSI 无法进行通信，请遵循[使用安全专用网络隔离工作负载]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)教程中的指示信息来监视接口上的流量并复查防火墙日志。 

## 除去资源
{: #removeresources}

除去本教程中创建的资源所需执行的步骤。 

VRA 采用按月付费套餐。取消也不会退费。建议您仅在下个月不再需要此 VRA 时取消。如果需要双 VRA 高可用性集群，那么可以在[网关详细信息](https://{DomainName}/classic/network/gatewayappliances)页面上升级此单个 VRA。
{:tip}

1. 取消任何虚拟服务器或裸机服务器
2. 取消 VRA
3. 通过支持凭单取消任何其他 VLAN。 


## 扩展教程

本教程可以与[通过 VPN 连接到安全专用网络中](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network)教程结合使用，以通过 IPSec VPN 将两个安全网络链接到用户的远程网络。可以建立与两个安全网络的 VPN 链路，以提高访问 {{site.data.keyword.Bluemix_notm}} IaaS 平台时的弹性。请注意，IBM 不允许通过 IBM 专用网络在客户机数据中心之间路由用户流量。避免网络循环的路由配置已超出了本教程的范围。 


## 相关材料
{:related}

1.  、虚拟路由和转发 (VRF) 是使用“VLAN 生成”在整个 {{site.data.keyword.Bluemix_notm}} 帐户中连接网络的替代方法。使用 [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview) 的所有客户机都必须使用 VRF。[IBM Cloud 上的虚拟路由和转发 (VRF) 的概述](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [IBM Cloud 网络](https://www.ibm.com/cloud/data-centers/)
