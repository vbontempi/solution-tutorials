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

# 配置 NAT 以从专用网络访问因特网
{: #nat-config-private}

在如今基于 Web 的 IT 应用程序和服务遍布的世界，几乎没有应用程序是孤立存在的。开发者开始期望在因特网上访问服务，无论服务是开放式源代码应用程序代码和更新，还是通过 REST API 提供应用程序功能的“第三方”服务。为了保护从专用网络对因特网托管的服务的访问，一种常用的方法是网络地址转换 (NAT) masquerade。在 NAT masquerade 中，专用 IP 地址以多对一的关系转换为出站公共接口的 IP 地址，从而保护专用 IP 地址不被公共访问。  

本教程说明了如何在虚拟路由器设备 (VRA) 上设置网络地址转换 (NAT) masquerade，以连接 {{site.data.keyword.Bluemix_notm}} 专用网络上的安全子网。本教程基于[使用安全专用网络隔离工作负载](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)教程而构建，添加了源 NAT (SNAT) 配置，其中对源地址进行了模糊处理，并使用防火墙规则来保护出站流量。更复杂的 NAT 配置可以在[补充 VRA 文档]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)中找到。
{:shortdesc}

## 目标
{: #objectives}

-	在虚拟路由器设备 (VRA) 上设置源网络地址转换 (SNAT)
-	针对因特网访问设置防火墙规则

## 使用的服务
{: #products}

本教程使用以下 {{site.data.keyword.Bluemix_notm}} 服务： 

* [虚拟路由器设备 VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

本教程可能会发生成本。VRA 仅在按月支付的定价套餐中可用。

## 体系结构
{: #architecture}

<p style="text-align: center;">

  ![体系结构](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	记录必需的因特网服务。
2.	设置 NAT。
3.	创建因特网防火墙专区和规则。

## 开始之前
{: #prereqs}

本教程在通过[使用安全专用网络隔离工作负载](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)教程创建的安全专用网络隔离区中启用主机，以访问公用因特网服务。后一个教程必须先完成。 

## 记录因特网服务
{: #Document_outbound}

第一步是确定将在公用因特网上访问的服务，并记录必须为来自因特网的出站和相应入站流量启用的端口。在稍后的步骤中，防火墙规则将需要此端口列表。 

在此示例中，仅启用了 http 和 https 端口，因为这两种端口可满足大多数需求。DNS 和 NTP 服务通过 {{site.data.keyword.Bluemix_notm}} 专用网络提供。如果需要这两种服务及其他服务，例如 SMTP（端口 25）或 MySQL（端口 3306），那么需要其他防火墙规则。两个基本端口规则如下：

-	端口 80 (http)
-	端口 443 (https)

验证第三方服务是否支持源地址列入白名单。如果支持，那么需要 VRA 的公共 IP 地址来配置第三方服务，以限制对服务的访问。 


## 通过 NAT masquerade 访问因特网 
{: #NAT_Masquerade}

按照此处的指示信息，使用 NAT masquerade 为 APP 专区中的主机配置外部因特网访问。 

1.	通过 SSH 登录到 VRA，然后进入 \[编辑\]（配置）方式。
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2.	在 VRA 上创建 SNAT 规则，并指定与先前 VRA 供应教程中为 APP 专区子网/VLAN 确定的相同 `<Subnet Gateway IP>/<CIDR>`。 
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Subnet Gateway IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## 创建防火墙
{: #Create_firewalls}

1.	创建防火墙规则 
   ```
   set security firewall name APP-TO-OUTSIDE default-action drop
   set security firewall name APP-TO-OUTSIDE description 'APP traffic to the Internet'
   set security firewall name APP-TO-OUTSIDE default-log

   set security firewall name APP-TO-OUTSIDE rule 90 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 90 action accept
   set security firewall name APP-TO-OUTSIDE rule 90 destination port 80

   set security firewall name APP-TO-OUTSIDE rule 100 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 100 action accept
   set security firewall name APP-TO-OUTSIDE rule 100 destination port 443

   set security firewall name APP-TO-OUTSIDE rule 200 protocol icmp
   set security firewall name APP-TO-OUTSIDE rule 200 icmp type 8
   set security firewall name APP-TO-OUTSIDE rule 200 action accept
   commit
   ```
   {: codeblock}

## 创建专区并应用规则
{: #Create_zone}

1.	创建专区 OUTSIDE 来控制对外部因特网的访问。
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	分配防火墙以控制进出因特网的流量。
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE 
   commit
   save
   ```
   {: codeblock}
3.	验证 APP 专区中的 VSI 现在是否可以访问因特网上的服务。使用 SSH 登录到本地 VSI，使用 ping 和 curl 来验证是否可通过 ICMP 和 TCP 来访问因特网上的站点。  
   ```bash
   ssh root@<VSI Private IP>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## 除去资源
{:removeresources}
除去本教程中创建的资源所需执行的步骤。 

VRA 采用按月付费套餐。取消也不会退费。建议您仅在下个月不再需要此 VRA 时取消。如果需要双 VRA 高可用性集群，那么可以在[网关详细信息](https://{DomainName}/classic/network/gatewayappliances)页面上升级此单个 VRA。
{:tip}  

1. 取消任何虚拟服务器或裸机服务器
2. 取消 VRA
3. 通过支持凭单取消任何其他 VLAN。 

## 相关材料
{:related}

-	[VRA 网络地址转换]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[NAT masquerade]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[补充 VRA 文档]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)。

