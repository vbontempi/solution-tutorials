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

# 从安全专用网络提供服务的 Web 应用程序
{: #web-app-private-network}

对于公共云来说，托管 Web 应用程序是一种常见的部署模式，在其中您可以根据需要来缩放资源以满足您的短期和长期使用需求。应用程序工作负载的安全性是基本的先决条件，用于补充公共云所能提供的弹性和可伸缩性。 

此教程会指导您创建一个可缩放且安全的面向因特网的 Web 应用程序，该应用程序托管在通过虚拟路由器设备 (VRA)、VLAN、NAT 和防火墙进行保护的专用网络中。该应用程序由一个负载均衡器、两个 Web 应用程序服务器以及一个 MySQL 数据库服务器组成。它组合了三个教程，用于演示可以如何使用经典联网将 Web 应用程序安全部署在 {{site.data.keyword.Bluemix_notm}} IaaS 平台上。
{:shortdesc}

## 目标
{: #objectives}

- 创建虚拟服务器以安装 PHP 和 MySQL
- 供应一个负载均衡器以将请求分发给应用程序服务器
- 部署虚拟路由器设备 (VRA) 以创建安全网络
- 定义 VLAN 和 IP 子网 
- 通过防火墙规则来保护网络
- 针对应用程序部署进行源网络地址转换 (SNAT)

## 使用的服务
{: #products}

本教程使用以下 {{site.data.keyword.Bluemix_notm}} 服务： 

* [虚拟路由器设备 VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [负载均衡器]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

本教程可能会发生成本。VRA 仅在按月支付的定价套餐中可用。

## 体系结构
{: #architecture}

<p style="text-align: center;">

  ![体系结构](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	配置安全专用网络
2.	配置 NAT 访问权以进行应用程序部署
3.	部署可缩放的 Web 应用程序和负载均衡器

## 开始之前
{: #prereqs}

此教程将利用按以下顺序部署的三个现有教程。开始之前，应复查所有三个教程：

-	[使用安全专用网络隔离工作负载]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[配置 NAT 以从安全网络进行因特网访问]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[使用虚拟服务器来构建高可用性和高可伸缩性 Web 应用程序]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## 配置安全专用网络
{: #private_network}

在公共云上，隔离且安全的专用网络环境对于 IaaS 应用程序安全性模型来说至关重要。防火墙、VLAN、路由和 VPN 都是创建隔离的专用环境所必需的组件。第一步是创建要在其中部署 Web 应用程序的安全专用网络隔离区。  

- [使用安全专用网络隔离工作负载](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

可以遵循此教程而不进行更改。在稍后的步骤中，会将三个虚拟机作为 Nginx Web 服务器和 MySQL 数据库部署到 APP 专区中。 

## 配置 NAT 以安全地部署应用程序
{: #nat_config}

要安装开放式源代码应用程序，您需要具有对因特网的安全访问权才能访问源存储库。为了防止安全专用网络中的服务器在公用因特网中公开，将使用源 NAT，其中对源地址进行了模糊处理，并使用防火墙规则来保护出站应用程序存储库请求。所有入站请求都会被拒绝。 

- [配置 NAT 以从安全网络进行因特网访问]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

可以遵循此教程而不进行更改。在下一步中，NAT 配置将用于访问所需的 Nginx 和 MySQL 模块。  


## 部署可缩放的 Web 应用程序和负载均衡器
{: #scalable_app}

Nginx 和 MySQL 上的 Wordpress 安装以及一个负载均衡器用于演示可以如何在安全专用网络中部署可扩缩放的弹性 Web 应用程序 

本教程将指导您完成此方案，即创建一个 {{site.data.keyword.Bluemix_notm}} 负载均衡器、两个 Web 应用程序服务器以及一个 MySQL 数据库服务器。这些服务器会部署到安全专用网络中的 APP 专区，以将防火墙与其他工作负载和公用网络相分离。 

- [使用虚拟服务器来构建高可用性和高可伸缩性 Web 应用程序]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

本教程中有以下三个更改：

1.	本教程中使用的虚拟服务器部署到受 VRA 后面的 APP 防火墙专区保护的 VLAN 和子网上。
2. 订购虚拟服务器时，请指定 &lt;Private VLAN ID&gt;。有关订购虚拟服务器时如何指定 &lt;Private VLAN ID&gt; 的详细信息，请参阅[使用安全专用网络隔离工作负载]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)教程中的[订购第一个虚拟服务器](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver)指示信息。此外，请务必选择您在该教程前面部分中上传的 SSH 密钥，以允许访问虚拟服务器。 
3. 因为将 Wordpress 文件复制到共享存储器的 rsync 性能较差，所以强烈建议**不要**在本教程中使用 File Storage 服务。这不会影响整个教程。对于应用程序服务器和数据库，可以忽略有关创建 File Storage 和设置安装的步骤。或者，您也可以在这两个应用程序服务器上执行所有[在应用程序服务器上安装和配置 PHP 应用程序](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application)的步骤。在完成[在应用程序服务器上安装和配置 PHP 应用程序](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application)中的步骤之前，请先在这两个应用程序服务器上创建目录 `/mnt/www/`。此目录最初是在文件存储部分中创建的，这部分现已被除去。 

   ```sh
   mkdir /mnt/www
   ```

完成此步骤后，负载均衡器应该会处于正常运行状态，并且可通过因特网访问 Wordpress 站点。组成 Web 应用程序的虚拟服务器受 VRA 防火墙保护以防止通过因特网进行外部访问，因此只能通过负载均衡器进行访问。对于生产环境，还应考虑由 [{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services) 提供的 DDoS 保护和 Web 应用程序防火墙 (WAF)。


## 除去资源
{: #removeresources}

除去本教程中创建的资源所需执行的步骤。 

VRA 采用按月付费套餐。取消也不会退费。建议您仅在下个月不再需要此 VRA 时取消。
{:tip}  

1. 取消任何虚拟服务器或裸机服务器
2. 取消 VRA
3. 通过支持凭单取消任何其他 VLAN。
4. 删除负载均衡器
5. 删除 File Storage 服务

## 扩展教程 

1. 本教程中，最初只供应了两个虚拟服务器作为应用程序层，可以自动添加更多服务器以处理额外的负载。通过[自动缩放]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group)，能够自动执行与添加或除去虚拟服务器相关联的手动缩放过程，以支持您的业务应用程序。

2. 通过向 VRA 添加第二个专用 VLAN 和 IP 子网来单独保护用户数据，以创建用于托管 MySQL 数据库服务器的 DATA 专区。将防火墙规则配置为仅允许端口 3306 上从 APP 专区到 DATA 专区的入站 MySQL IP 流量。 

