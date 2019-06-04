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

# VPN 到安全專用網路
{: #configuring-IPSEC-VPN}

我們常常需要在遠端網路環境與 {{site.data.keyword.Bluemix_notm}} 專用網路的伺服器之間建立專用連線。通常情況下，此連線功能支援 {{site.data.keyword.Bluemix_notm}} 上的混合式工作負載、資料傳送、專用工作負載或系統管理。站台對站台虛擬專用網路 (VPN) 通道是保護網路之間連線功能安全的常用方法。 

{{site.data.keyword.Bluemix_notm}} 提供用於站台對站台資料中心連線功能的多個選項，可以是使用公用網際網路上的 VPN 或者透過專用的專用網路連線功能。 

請參閱 [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
以取得有關 {{site.data.keyword.Bluemix_notm}} 的專用安全網路鏈結的更多詳細資料。公用網際網路上的 VPN 提供成本更低的選項，只是沒有頻寬保證。 

有兩個適合的 VPN 選項，可用於透過公用網際網路連接至 {{site.data.keyword.Bluemix_notm}} 上佈建的伺服器：

-	[IPSEC VPN]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

本指導教學呈現使用虛擬路由器應用裝置 (VRA) 將用戶端資料中心的子網路連接至 {{site.data.keyword.Bluemix_notm}} 專用網路上安全子網路的站台對站台 IPSec VPN 的設定。 

此範例根據[使用安全專用網路隔離工作負載](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)指導教學進行建置。它使用站台對站台 IPSec VPN、GRE 通道和靜態遞送。可以在[補充 VRA 文件](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)中找到使用動態路徑（BGP 等）和 VTI 通道的更複雜 VPN 配置。
{:shortdesc}

## 目標
{: #objectives}

- 記錄 IPSec VPN 的配置參數
- 在虛擬路由器應用裝置上配置 IPSec VPN
- 透過 GRE 通道遞送資料流量

## 使用的服務
{: #products}

本指導教學使用下列 {{site.data.keyword.Bluemix_notm}} 服務： 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

本指導教學可能會產生成本。VRA 只提供於按月定價方案。

## 架構
{: #architecture}

<p style="text-align: center;">

  ![架構](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	記錄 VPN 配置
2.	在 VRA 上建立 IPSec VPN
3.	配置資料中心 VPN 和通道
4.	建立 GRE 通道
5.	建立靜態 IP 路徑
6.	配置防火牆 

## 開始之前
{: #prereqs}

本指導教學將[使用安全專用網路隔離工作負載](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)指導教學中的安全專用隔離區連接至您的資料中心。必須先完成該指導教學。

## 記錄 VPN 配置
{: #Document_VPN}

配置資料中心與 {{site.data.keyword.Bluemix_notm}} 之間的 IPSec VPN 站台對站台鏈結需要與您的現場網路團隊協作，以確定許多配置參數、通道類型和 IP 遞送資訊。此參數必須與正常運作的 VPN 連線完全一致。通常，您的現場網路團隊將指定配置以符合協議的組織標準，並為您提供資料中心 VPN 閘道的必要 IP 位址以及可存取的子網路位址範圍。

在開始設定 VPN 之前，VPN 閘道的 IP 位址和 IP 網路子網路範圍必須加以確定，並可用於資料中心 VPN 配置和 {{site.data.keyword.Bluemix_notm}} 中的安全專用網路隔離區。下圖進行說明，其中安全隔離區中的「應用程式」區域將透過 IPSec 通道連接至用戶端資料中心的「DC IP 子網路」中的系統。   

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

下列參數必須在配置 VPN 的 {{site.data.keyword.Bluemix_notm}} 使用者與用戶端資料中心的網路團隊之間進行協議並記錄下來。在此範例中，遠端和本端通道 IP 位址分別設定為 192.168.10.1 和 192.168.10.2。經現場網路團隊同意，可以使用任意子網路。 

|項目|說明|
|:------ |:--- | 
| &lt;ike group name&gt; | 指定給用於連線的 IKE 群組的名稱。|
| &lt;ike encryption&gt; | {{site.data.keyword.Bluemix_notm}} 與用戶端資料中心之間要使用的協議 IKE 加密標準，通常為 *aes256*。|
| &lt;ike hash&gt; | {{site.data.keyword.Bluemix_notm}} 與用戶端資料中心之間協議的 IKE 雜湊，通常為 *sha1*。|
| &lt;ike-lifetime&gt; | 用戶端資料中心的 IKE 生命期限，通常為 *3600*。|
| &lt;esp group name&gt; | 指定給用於連線的 ESP 群組的名稱。|
| &lt;esp encryption&gt; | {{site.data.keyword.Bluemix_notm}} 與用戶端資料中心之間協議的 ESP 加密標準，通常為 *aes256*。|
| &lt;esp hash&gt; | {{site.data.keyword.Bluemix_notm}} 與用戶端資料中心之間協議的 ESP 雜湊，通常為 *sha1*。|
| &lt;esp-lifetime&gt; | 用戶端資料中心的 ESP 生命期限，通常為 *1800*。|
| &lt;DC VPN Public IP&gt;  | 用戶端資料中心內 VPN 閘道的面向網際網路的公用 IP 位址。| 
| &lt;VRA Public IP&gt; |VRA 的公用 IP 位址。|
| &lt;Remote tunnel IP\/24&gt; |指派給 IPSec 通道遠端的 IP 位址。範圍中與 IP Cloud 或用戶端資料中心不衝突的 IP 位址配對。|
| &lt;Local tunnel IP\/24&gt; |指派給 IPSec 通道本端的 IP 位址。|
| &lt;DC Subnet/CIDR&gt; |要在用戶端資料中心和 CIDR 中存取的子網路的 IP 位址。|
| &lt;App Zone subnet/CIDR&gt; |VRA 建立指導教學中的「應用程式」區域子網路的網路 IP 位址和 CIDR。| 
| &lt;Shared-Secret&gt; |要在 {{site.data.keyword.Bluemix_notm}} 與用戶端資料中心之間使用的共用加密金鑰。|

## 在 VRA 上配置 IPSec VPN
{: #Configure_VRA_VPN}

若要在 {{site.data.keyword.Bluemix_notm}} 上建立 VPN，指令和需要變更的所有變數都在下面使用 &lt; &gt; 進行強調顯示。對於需要變更的每一行，將逐行識別變更。值來自表格。 

1. 透過 SSH 進入 VRA 並進入 `[edit]` 模式。
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2. 建立網際網路金鑰交換 (IKE) 群組。
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike encryption>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike hash>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lifetime>
   ```
   {: codeblock}
3. 建立封裝安全有效負載 (ESP) 群組
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp encryption>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp hash>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lifetime>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. 定義站台對站台連線
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

## 配置資料中心 VPN 和通道
{: #Configure_DC_VPN}

1. 用戶端資料中心的網路團隊將配置相同的 IPSec VPN 連線，但會交換 &lt;DC VPN Public IP&gt; 與 &lt;VRA Public IP&gt;、交換本端與遠端通道位址，並交換 &lt;DC Subnet/CIDR&gt; 與 &lt;App Zone subnet/CIDR&gt; 參數。用戶端資料中心內的特定配置指令將取決於 VPN 的供應商。
1. 在繼續之前，驗證 DC VPN 閘道的公用 IP 位址是否可透過網際網路存取：
   ```
   ping <DC VPN Public IP>
   ```
   {: codeblock}
2. 完成資料中心的 VPN 配置時，IPSec 鏈結應該會自動出現。驗證該鏈結是否已建立，以及狀態是否顯示存在一個或更多作用中的 IPSec 通道。使用資料中心驗證 VPN 的兩端是否都顯示作用中的 IPSec 通道。
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. 如果未建立該鏈結，請使用 debug 指令驗證是否已正確指定本端和遠端位址，以及其他參數是否符合預期：
   ``` bash
   show vpn debug
   ```
   {: codeblock}

輸出中應該會顯示行 `peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......`。如果此行不存在，或者顯示 "CONNECTING""，則 VPN 配置中已存在錯誤。  

## 定義 GRE 通道 
{: #Define_Tunnel}

1. 在 VRA 編輯模式下建立 GRE 通道。
   ```
   set interfaces tunnel tun0 address <Local tunnel IP/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <VRA Public IP>
   set interfaces tunnel tun0 remote-ip <DC VPN Public IP>
   commit
   ```
   {: codeblock}
2. 通道的兩端已配置後，通道應該會自動出現。從 VRA 指令行檢查通道的作業狀態。
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   第一個指令應該會顯示通道的狀態和鏈結為 `u/u` (UP/UP)。第二個指令將顯示有關通道的更多詳細資料，並顯示已傳輸和接收資料流量。
3. 驗證資料流量是否跨通道流動
   ```bash
   ping <Remote tunnel IP>
   ```
   {: codeblock}
  存在 `ping` 資料流量時，應該會看到 `show interfaces tunnel tun0` 上的 TX 和 RX 計數遞增。
4. 如果資料流量未流動，則可使用 `monitor interface` 指令觀察每個介面上看到的資料流量。介面 `tun0` 顯示透過通道的內部資料流量。介面 `dp0bond1` 將顯示進出遠端 VPN 閘道的封裝資料流量流程。
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

如果看不到返回資料流量，則資料中心網路團隊將需要監視 VPN 中的資料流量流程和遠端網站上的通道介面，以將該問題控制在本端。 

## 建立靜態 IP 路徑
{: #Define_Routing}

建立 VRA 遞送以透過通道將資料流量導向遠端子網路。

1. 在 VRA 編輯模式下建立靜態路徑。
   ```
   set protocols static route <DC Subnet/CIDR>  next-hop <Remote tunnel IP>
   ```
   {: codeblock}   
2. 從 VRA 指令行檢閱 VRA 遞送表。目前資料流量都不會遍訪路徑，因為沒有防火牆規則存在，無法允許資料流量透過通道。在任一端起始的資料流量都需要防火牆規則。
   ```bash
   show ip route
   ```
   {: codeblock}

## 配置防火牆
{: #Configure_firewall}

1. 針對允許的 icmp 資料流量及 tcp 埠建立資源群組。 
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
2. 針對 VRA 編輯模式中送往遠端子網路的資料流量建立防火牆規則。
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
2. 為通道建立區域，並為任一區域中起始的資料流量關聯防火牆。
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
3. 若要驗證兩端的防火牆和遞送是否配置正確，以及現在是否容許 ICMP 和 TCP 資料流量，請對遠端子網路的閘道位址執行 Ping 作業，先藉由 VRA 指令行執行，如果成功，再藉由登入 VSI 來執行。
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}
4. 如果從 VRA 指令行執行 Ping 作業失敗，請驗證在回應通道介面上的 Ping 要求時是否看到 Ping 回覆。
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
   「沒有回應」指出防火牆規則或資料中心的遞送發生問題。如果在監視器輸出中出現回覆，但是 Ping 指令逾時，請檢查本端 VRA 防火牆規則的配置。
5. 如果從 VSI 執行 Ping 作業失敗，這指出 VRA 防火牆規則、VRA 或 VSI 配置中的遞送發生問題。完成先前的步驟，以確保要求已傳送到資料中心，並且從資料中心看到回應。監視本端 VLAN 上的資料流量以及檢查防火牆日誌有助於確定發生遞送還是防火牆問題。
   ```
   monitor interfaces bonding dp0bond0.<VLAN ID>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

這樣就完成從安全專用網路隔離區設定 VPN。本系列的其他指導教學說明隔離區可以如何存取公用網際網路上的服務。

## 移除資源
{:removeresources}

移除本指導教學所建立資源時要採取的步驟。

VRA 屬於按月付費方案。取消不會退款。建議只有在下個月不會再度需要此 VRA 時才取消。如果需要雙 VRA 高可用性叢集，這個單一 VRA 可以在[閘道詳細資料](https://{DomainName}/classic/network/gatewayappliances)頁面上升級。
{:tip}  

1. 取消全部虛擬伺服器或裸機伺服器
2. 取消 VRA
3. 以支援問題單取消任何其他 VLAN 

## 相關內容
{:related}
- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [靜態和可攜式 IP 子網路](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Vyatta 文件](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS IPsec Site-to-Site VPN Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS Tunnels Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
