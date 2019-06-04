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

# 透過 IBM 網路鏈結安全專用網路
{: #vlan-spanning}

隨著對 Web 應用程式全球聯繫範圍和全天候運作的需求增加，在多個雲端資料中心內管理服務的需求也隨之增加。跨多個位置的資料中心可以在出現地理位置故障時提供備援能力，還會使工作負載更靠近分散在全球的使用者，以便能減少延遲並及早發現。[{{site.data.keyword.Bluemix_notm}} 網路](https://www.ibm.com/cloud/data-centers/)讓使用者能夠跨資料中心和位置鏈結各安全專用網路中所管理的工作負載。

本指導教學將呈現，在不同資料中心內管理的兩個安全專用網路之間，如何設定透過 {{site.data.keyword.Bluemix_notm}} 專用網路的專用遞送 IP 連線。所有資源都由一個 {{site.data.keyword.Bluemix_notm}} 帳戶擁有。該帳戶使用[透過安全專用網路隔離工作負載]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)指導教學來部署兩個專用網路，這兩個專用網路使用 [VLAN Spanning ]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)服務透過 {{site.data.keyword.Bluemix_notm}} 專用網路進行安全鏈結。
{:shortdesc}

## 目標
{: #objectives}

- 在 {{site.data.keyword.Bluemix_notm}} IaaS 帳戶中鏈結安全網路
- 針對站台對站台存取設定防火牆規則 
- 配置站台間的遞送

## 使用的服務
{: #products}

本指導教學使用下列 {{site.data.keyword.Bluemix_notm}} 服務： 
* [Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

本指導教學可能會發生成本。VRA 只提供於按月定價方案。

## 架構
{: #architecture}

<p style="text-align: center;">

  ![架構](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. 部署安全專用網路
2. 配置 VLAN Spanning
3. 配置專用網路之間的 IP 遞送
4. 配置防火牆規則以存取遠端網站

## 開始之前
{: #prereqs}

本指導教學是根據指導教學[使用安全專用網路隔離工作負載]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)。開始之前，應檢閱該指導教學及其必要條件。 

## 配置安全專用網路網站
{: #private_network}

將利用指導教學[使用安全專用網路隔離工作負載]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)兩次，以在兩個不同的資料中心內實作專用網路。除了注意延遲對將在網站之間通訊的任何資料流量或工作負載的影響之外，對於可以利用哪兩個資料中心沒有任何限制。 

可以遵循[使用安全專用網路隔離工作負載]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)指導教學，而不變更每個所選資料中心，同時記錄下列資訊以供後續步驟使用。 

|項目| Datacenter1 | Datacenter2 |
|:------ |:--- | :--- |
|資料中心|  |  |
| VRA 公用 IP 位址 | <DC1 VRA Public IP Address> | <DC2 VRA Public IP Address> |
| VRA 專用 IP 位址| <DC1 VRA Private IP Address> | <DC2 VRA Private IP Address> |
| VRA 專用子網路 & CIDR |  |  |
| 專用 VLAN ID| &lt;DC1 Private VLAN ID&gt;  | &lt;DC2 Private VLAN ID&gt; |
| VSI 專用 IP 位址| <DC1 VSI Private IP Address> | <DC2 VSI Private IP Address> |
| APP 區域子網路 & CIDR | <DC1 APP zone subnet/CIDR> | <DC2 APP zone subnet/CIDR> |

1. 透過[閘道應用裝置](https://{DomainName}/classic/network/gatewayappliances)頁面，前往每個 VRA 的「閘道詳細資料」頁面。  
2. 找到「閘道 VLAN 」區段，然後按一下**專用**網路上的「閘道 [VLAN]( https://{DomainName}/classic/network/vlans)」以檢視 VLAN 詳細資料。名稱應包含 ID `bcrxxx`，代表「後端客戶路由器」，並且格式應為 `nnnxx.bcrxxx.xxxx`。
3. 將在**裝置**區段下顯示所佈建的 VRA。在**子網路**區段下，記下 VRA 專用子網路 IP 位址和 CIDR (/26)。該子網路的類型為 primary，具有 64 個 IP。稍後的遞送配置會需要這些詳細資料。 
4. 回到「閘道詳細資料」頁面，找到**關聯的 VLAN** 區段，然後在關聯的**專用**網路上按一下 [VLAN]( https://{DomainName}/classic/network/vlans) 以建立安全網路和 APP 區域。 
5. 將在**裝置**區段下顯示所佈建的 VSI。在**子網路**區段下，記下 VSI 子網路 IP 位址和 CIDR (/26)，因為遞送配置需要這些資訊。此 VLAN 和子網路會在兩個 VRA 防火牆配置中識別為 APP 區域，並記錄為 &lt;APP Zone subnet/CIDR&gt;。


## 配置 VLAN Spanning 
{: #configure-vlan-spanning}

依預設，不同 VLAN 和資料中心上的伺服器（和 VRA）無法透過專用網路彼此進行通訊。在這些指導教學中，在單一資料中心內，VRA 用於將 VLAN 和子網路鏈結到標準 IP 遞送和防火牆，以建立專用網路用於跨 VLAN 進行伺服器通訊。雖然它們可以在同一資料中心內進行通訊，但在此配置中，屬於同一 {{site.data.keyword.Bluemix_notm}} 帳戶的伺服器卻無法跨資料中心進行通訊。 

[VLAN Spanning ]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)服務解除了對**未**與 VRA 關聯的 VLAN 和子網路之間通訊的這一限制。請務必注意，即使已啟用了 VLAN Spanning，與 VRA 關聯的 VLAN 也只能透過其關聯的 VRA 進行通訊，如 VRA 防火牆和遞送配置所確定。已啟用 VLAN Spanning 後，{{site.data.keyword.Bluemix_notm}} 帳戶所擁有的 VRA 會透過專用網路進行連接，並可以進行通訊。 

未與 VRA 所建立的安全專用網路相關聯的 VLAN 是「跨距的」，容許這些「未關聯的」VLAN 跨資料中心互相連接。這包括不同資料中心內屬於同一 IBM Cloud 帳戶的 VRA 閘道（轉移）VLAN。因此，當 VLAN Spanning 已啟用後，會容許 VRA 跨資料中心進行通訊。透過 VRA 到 VRA 的連線功能，VRA 防火牆和遞送配置可讓各安全網路中的伺服器能夠連接。 

啟用 VLAN Spanning：

1. 前往 [VLAN]( https://{DomainName}/classic/network/vlans) 頁面。
2. 在頁面頂端選取**跨距**標籤
3. 為 VLAN Spanning 選取「啟動」圓鈕。該網路變更需要幾分鐘才能完成。
4. 確認這兩個 VRA 現在可以通訊：

   登入到資料中心 1 VRA，然後對資料中心 2 VRA 執行 ping 操作

   ```
   SSH vyatta@<DC1 VRA Private IP Address>
   ping <DC2 VRA Private IP Address>
   ```
   {: codeblock}

   登入到資料中心 2 VRA，然後對資料中心 1 VRA 執行 ping 操作
   ```
   SSH vyatta@<DC2 VRA Private IP Address>
   ping <DC1 VRA Private IP Address>
   ```
   {: codeblock}

## 配置 VRA IP 遞送 
{: #vra_routing}

在每個資料中心內建立 VRA 遞送，以讓兩個資料中心內 APP 區域中的 VSI 能夠進行通訊。 

1. 在 VRA 編輯模式下，在資料中心 1 中建立與資料中心 2 中的 APP 區域專用子網路的靜態路徑。
   ```
   ssh vyatta@<DC1 VRA Private IP Address>
   conf
   set protocols static route <DC2 APP zone subnet/CIDR>  next-hop <DC2 VRA Private IP>
   commit
   ```
   {: codeblock}
2. 在 VRA 編輯模式下，在資料中心 2 中建立與資料中心 1 中的 APP 區域專用子網路的靜態路徑。
   ```
   ssh vyatta@<DC2 VRA Private IP Address>
   conf
   set protocols static route <DC1 APP zone subnet/CIDR>  next-hop <DC1 VRA Private IP>
   commit
   ```
   {: codeblock}
2. 從 VRA 指令行檢閱 VRA 遞送表。此時 VSI 還不能進行通訊，因為不存在 APP 區域防火牆規則以容許兩個 APP 區域子網路之間的資料流量。在任一端起始的資料流量都需要防火牆規則。
   ```
   show ip route
   ```
   {: codeblock}

現在將顯示容許 APP 區域透過 IBM 專用網路進行通訊的新路徑。 

## VRA 防火牆配置
{: #vra_firewall}

現有的 APP 區域防火牆規則僅配置為容許此子網路與 {{site.data.keyword.Bluemix_notm}} 專用網路上的 {{site.data.keyword.Bluemix_notm}} 服務之間的資料流量，以及透過 NAT 進行公用網際網路存取的資料流量。與此 VRA 上或其他資料中心內的 VSI 關聯的其他子網路都會被阻止。下一步是更新與 APP-TO-INSIDE 防火牆規則相關聯的 `ibmprivate` 資源群組，以容許明確存取其他資料中心內的子網路。 

1. 在資料中心 1 VRA 編輯指令模式下，將 <DC2 APP zone subnet>/CIDR 新增到 `ibmprivate` 資源群組
   ```
   set resources group address-group ibmprivate address <DC2 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
2. 在資料中心 2 VRA 編輯指令模式下，將 <DC1 APP zone subnet>/CIDR 新增到 `ibmprivate` 資源群組
   ```
   set resources group address-group ibmprivate address <DC1 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
3. 驗證兩個資料中心內的 VSI 現在是否可以進行通訊
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}

   如果 VSI 無法進行通訊，請遵循[使用安全專用網路隔離工作負載]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)指導教學中的指示來監視介面上的資料流量並檢閱防火牆日誌。 

## 移除資源
{: #removeresources}

移除本指導教學所建立資源時要採取的步驟。 

VRA 屬於按月付費方案。取消不會退款。建議只有在下個月不會再度需要此 VRA 時才取消。如果需要雙 VRA 高可用性叢集，這個單一 VRA 可以在[閘道詳細資料](https://{DomainName}/classic/network/gatewayappliances)頁面上升級。
{:tip}

1. 取消全部虛擬伺服器或裸機伺服器
2. 取消 VRA
3. 以支援問題單取消任何其他 VLAN 


## 延伸指導教學

本指導教學可以與[透過 VPN 連接到安全專用網路中](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network)指導教學結合使用，以透過 IPSec VPN 將兩個安全網路鏈結到使用者的遠端網路。可以建立連接到這兩個安全網路的 VPN 鏈結，以在存取 {{site.data.keyword.Bluemix_notm}} IaaS 平台時提高備援能力。請注意，IBM 不容許透過 IBM 專用網路在用戶端資料中心之間遞送使用者資料流量。避免網路迴圈的遞送配置已超出了本指導教學的範圍。 


## 相關資料
{:related}

1.  「虛擬遞送和轉遞 (VRF)」是使用 VLAN Spanning 在整個 {{site.data.keyword.Bluemix_notm}} 帳戶中連接網路的替代方案。使用 [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview) 的所有用戶端都必須使用 VRF。[IBM Cloud 上的虛擬遞送及轉遞 (VRF) 概觀](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [IBM Cloud 網路](https://www.ibm.com/cloud/data-centers/)
