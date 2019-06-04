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

# 從安全專用網路提供服務的 Web 應用程式
{: #web-app-private-network}

對於為公用雲端來說，管理 Web 應用程式是一種常見的部署模式，在其中您可以根據需要來調整資源以符合您的短期和長期使用需求。應用程式工作負載的安全是基本的先決條件，用於補充為公用雲端所提供的備援能力和可調整性。 

此指導教學會指導您建立一個可調整且安全的面向網際網路的 Web 應用程式，該應用程式將在透過虛擬路由器應用裝置 (VRA)、VLAN、NAT 和防火牆進行保護的專用網路中加以管理。該應用程式由一個負載平衡器、兩個 Web 應用程式伺服器以及一個 MySQL 資料庫伺服器組成。它組合了三個指導教學，用於說明可以如何使用標準網路連線功能將 Web 應用程式安全部署在 {{site.data.keyword.Bluemix_notm}} IaaS 平台上。
{:shortdesc}

## 目標
{: #objectives}

- 建立虛擬伺服器以安裝 PHP 和 MySQL
- 佈建一個負載平衡器以將要求配送給應用程式伺服器
- 部署虛擬路由器應用裝置 (VRA) 以建立安全網路
- 定義 VLAN 和 IP 子網路 
- 透過防火牆規則來保護網路
- 針對應用程式部署進行來源網址轉換 (SNAT)

## 使用的服務
{: #products}

本指導教學使用下列 {{site.data.keyword.Bluemix_notm}} 服務： 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [ 負載平衡器 ]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

本指導教學可能會產生成本。VRA 只提供於按月定價方案。

## 架構
{: #architecture}

<p style="text-align: center;">

  ![架構](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	配置安全專用網路
2.	配置 NAT 存取權以進行應用程式部署
3.	部署可擴充的 Web 應用程式及負載平衡器

## 開始之前
{: #prereqs}

此指導教學將利用三個現有指導教學，依序部署。開始之前，請先檢閱全部三個指導教學：

-	[使用安全專用網路隔離工作負載]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[配置 NAT 以便從安全網路存取網際網路]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[使用虛擬伺服器建置高可用性且可擴充的 Web 應用程式]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## 配置安全專用網路
{: #private_network}

在公用雲端上，隔離且安全的專用網路環境對於 IaaS 應用程式安全模型來說至關重要。在建立隔離的專用環境時，防火牆、VLAN、遞送及 VPN 全都是必要的元件。第一步是建立要在其中部署 Web 應用程式的安全專用網路隔離區。  

- [使用安全專用網路隔離工作負載](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

可以遵循本指導教學而不需變更。在稍後的步驟中，會將三個虛擬機器作為 Nginx Web 伺服器和 MySQL 資料庫部署到 APP 區域中。 

## 配置 NAT 以安全地部署應用程式
{: #nat_config}

若要安裝開放式來源程式碼應用程式，您需要具有對網際網路的安全存取權才能存取來源儲存庫。為了防止安全專用網路中的伺服器在公用網際網路中公開，將使用來源 NAT，其中對來源端位址進行了模糊處理，並使用防火牆規則來保護出埠應用程式儲存庫要求。所有入埠要求都會被拒絕。 

- [配置 NAT 以便從安全網路存取網際網路]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

可以遵循本指導教學而不需變更。在下一步中，NAT 配置將用於存取所需的 Nginx 和 MySQL 模組。  


## 部署可擴充的 Web 應用程式及負載平衡器
{: #scalable_app}

Nginx 和 MySQL 上的 Wordpress 安裝，具有一個負載平衡器，用於說明如何在安全專用網路中部署可調整且具復原力的 Web 應用程式 

本指導教學將指導您完成此方案，即建立一個 {{site.data.keyword.Bluemix_notm}} 負載平衡器、兩個 Web 應用程式伺服器以及一個 MySQL 資料庫伺服器。這些伺服器會部署到安全專用網路中的 APP 區域，以將防火牆與其他工作負載和公用網路相分離。 

- [使用虛擬伺服器建置高可用性且可擴充的 Web 應用程式]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

本指導教學中有以下三個變更：

1.	本指導教學中使用的虛擬伺服器，會部署到受 VRA 後面的 APP 防火牆區域保護的 VLAN 和子網路上。
2. 訂購虛擬伺服器時，請指定 &lt;Private VLAN ID&gt;。有關訂購虛擬伺服器時如何指定 &lt;Private VLAN ID&gt; 的詳細資料，請參閱[使用安全專用網路隔離工作負載]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)指導教學中的[訂購第一個虛擬伺服器](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver)指示。此外，請務必選取您先前在該指導教學中上傳的 SSH 金鑰，以容許存取虛擬伺服器。 
3. 因為將 Wordpress 檔案複製到共用儲存空間的重新同步效能較差，所以強烈建議**不要**在本指導教學中使用檔案儲存服務。這不會影響整個指導教學。對於應用程式伺服器和資料庫，可以忽略有關建立檔案儲存空間和設定裝載的步驟。或者，您也可以在這兩個應用程式伺服器上執行所有[在應用程式伺服器上安裝和配置 PHP 應用程式](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application)的步驟。在完成[在應用程式伺服器上安裝和配置 PHP 應用程式](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application)中的步驟之前，請先在這兩個應用程式伺服器上建立目錄 `/mnt/www/`。此目錄最初是在檔案儲存空間區段中建立的，這區段現已被移除。 

   ```sh
   mkdir /mnt/www
   ```

完成此步驟後，負載平衡器應該會處於性能良好狀態，並且可透過網際網路存取 Wordpress 網站。組成 Web 應用程式的虛擬伺服器受 VRA 防火牆保護，以防止透過網際網路進行外部存取，因此只能透過負載平衡器進行存取。對於正式作業環境，還應考慮由 [{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services) 提供的 DDoS 保護和 Web 應用程式防火牆 (WAF)。


## 移除資源
{: #removeresources}

移除本指導教學所建立資源時要採取的步驟。 

VRA 屬於按月付費方案。取消不會退款。建議只有在下個月不會再度需要此 VRA 時才取消。
{:tip}  

1. 取消全部虛擬伺服器或裸機伺服器
2. 取消 VRA
3. 以支援問題單取消任何其他 VLAN
4. 刪除負載平衡器
5. 刪除檔案儲存服務

## 擴展指導教學 

1. 本指導教學中，最初只佈建了兩個虛擬伺服器作為應用程式層級，可以自動新增更多伺服器以處理額外的負載。透過[自動調整]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group)，能夠自動化與新增或移除虛擬伺服器相關聯的手動調整程序，以支援您的商業應用程式。

2. 透過向 VRA 新增第二個專用 VLAN 和 IP 子網路來個別保護使用者資料，以建立用於管理 MySQL 資料庫伺服器的 DATA 區域。將防火牆規則配置為僅容許埠 3306 上從 APP 區域到 DATA 區域的入埠 MySQL IP 資料流量。 

