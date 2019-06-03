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

# 自带 IP 地址
{: #byoip}

自带 IP (BYOIP) 是常见需求，用于将现有客户机网络连接到 {{site.data.keyword.Bluemix_notm}} 上供应的基础架构。其目的通常是通过采用基于客户机现有 IP 寻址方案的单个 IP 地址空间，最大程度减少对客户机网络路由配置和操作的更改。

本教程简要介绍 BYOIP 实施模式的概况，在实现[使用安全专用网络隔离工作负载](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)教程中所述的安全隔离区时，可以将这些实施模式与 {{site.data.keyword.Bluemix_notm}} 和决策树一起用于识别相应模式。设置时，可能需要现场网络团队、{{site.data.keyword.Bluemix_notm}} 技术支持或 IBM Services 进行其他输入。

{:shortdesc}

## 目标
{: #objectives}

* 了解 BYOIP 实施模式
* 选择 {{site.data.keyword.Bluemix_notm}} 的实施模式。

## {{site.data.keyword.Bluemix_notm}} IP 寻址

{{site.data.keyword.Bluemix_notm}} 利用若干专用地址范围，通常是 10.0.0.0/8，在有些情况下，这些范围可能与现有客户机网络冲突。在地址发生冲突时，有若干模式支持 BYOIP 允许与 IBM Cloud 网络进行互操作。

-	网络地址转换
-	GRE（通用路由封装）隧道
-	使用 IP 别名的 GRE 隧道 
-	虚拟覆盖网络

模式的选择由计划在 {{site.data.keyword.Bluemix_notm}} 上托管的应用程序确定。有两个关键方面，应用程序对模式实施的敏感度，以及客户机网络和 {{site.data.keyword.Bluemix_notm}} 之间地址范围的重叠程度。如果打算使用与 {{site.data.keyword.Bluemix_notm}} 的专用私有网络连接，还将有更多注意事项适用。建议考虑使用 BYOIP 的所有用户阅读 [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link) 文档。对于 {{site.data.keyword.BluDirectLink}} 用户，应遵循关联的指南以遵从此处提供的信息。

## 实施模式概述
{: #patterns_overview}

1. **NAT**：内部部署客户机路由器中的 NAT 地址转换。执行内部部署 NAT 以将客户机寻址方案转换到 {{site.data.keyword.Bluemix_notm}} 为供应的 IaaS 服务分配的 IP 地址。  
2. **GRE 隧道**：在 {{site.data.keyword.Bluemix_notm}} 和内部部署网络之间通过 GRE 隧道路由 IP 流量（通常使用 VPN），可统一寻址方案。这是[本教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)中演示的场景。 

   存在两个子模式，取决于地址空间重叠的可能性。
     * 无地址重叠：地址范围没有地址重叠，网络之间没有冲突风险。
     * 部分地址重叠：客户机和 {{site.data.keyword.Bluemix_notm}} IP 地址空间使用相同的地址范围，有出现重叠和冲突的可能性。在这种情况下，将选择 {{site.data.keyword.Bluemix_notm}} 专用网络中不重叠的客户机子网地址。

3. GRE 隧道 + IP 别名
通过在内部部署网络和分配给 {{site.data.keyword.Bluemix_notm}} 上服务器的别名 IP 地址之间的 GRE 隧道来路由 IP 流量，可统一寻址方案。这是[本教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)中演示的场景的特殊情况。在 {{site.data.keyword.Bluemix_notm}} 上供应的虚拟和裸机服务器上创建了兼容 IP 子网的其他接口和 IP 别名，由 VRA 上相应的路由配置提供支持。

4. 虚拟覆盖网络 [{{site.data.keyword.Bluemix_notm}} Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) 支持对 {{site.data.keyword.Bluemix_notm}} 上的完全虚拟环境使用 BYOIP。可以将其视为
[本教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)中描述的安全专用网络隔离区的替代方法。

还可以考虑 VMware NSX 之类的解决方案，该解决方案通过 {{site.data.keyword.Bluemix_notm}} 网络在一个层中实施虚拟覆盖网络。虚拟覆盖中的所有 BYOIP 地址独立于 {{site.data.keyword.Bluemix_notm}} 网络地址范围。请参阅 [VMware 和[{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud) 使用入门。

## 模式决策树
{: #decision_tree}

此处的决策树可用于确定相应的实施模式。 

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

以下注释提供进一步指导：

### 您的应用程序使用 NAT 有问题吗？
{: #nat_consideration}

在以下两种不同的情况下，NAT 可能会产生问题。在这些情况下，不应使用 NAT。 

- Microsoft AD 域通信之类的一些应用程序和 P2P 应用程序可能在使用 NAT 时遇到技术问题。
- 在未知服务器需要与 {{site.data.keyword.Bluemix_notm}} 通信或者 {{site.data.keyword.Bluemix_notm}} 和内部部署服务器之间需要几百个双向连接时。在这种情况下，由于无法事先识别映射，所以无法在客户机路由器/NAT 表格上配置所有映射。


### 无地址重叠
{: #no-overlap}

内部部署网络中使用 10.0.0.0/8 吗？内部部署和 {{site.data.keyword.Bluemix_notm}} 专用网络之间没有地址重叠时，可以在内部部署和 IBM Cloud 之间使用[本教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)中所描述的 GRE 隧道，从而无需进行 NAT 转换。这需要与现场网络团队一起复查网络地址使用情况。 

### 部分地址重叠
{: #partial_overlap}

如果在内部部署网络中使用了 10.0.0.0/8 范围中的任何值，[{{site.data.keyword.Bluemix_notm}} 网络上是否提供了非重叠子网？与现场网络团队一起复查现有网络地址使用情况，并联系 [{{site.data.keyword.Bluemix_notm}} 技术销售人员以识别可用的非重叠网络。 

### IP 别名判别是否有问题？
{: #ip_alias}

如果不存在不会重叠的地址，那么可以在安全专用网络隔离区中部署的虚拟和裸机服务器上实施 IP 别名判别。IP 别名判别在每个服务器的一个或多个网络接口上分配多个子网地址。 

IP 别名判别是常用的技巧，尽管建议复查服务器和应用程序配置来确定这些配置在多宿主和 IP 别名配置下是否运行良好。  

将需要使用 VRA 上的其他路由配置来为 BYOIP 子网创建动态路由（例如 BGP）或静态路由。 

## 相关内容
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
