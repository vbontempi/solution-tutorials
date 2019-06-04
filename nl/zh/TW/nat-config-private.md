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

# 配置 NAT 以從專用網路存取網際網路
{: #nat-config-private}

在如今以 Web 為基礎的 IT 應用程式和服務世界，幾乎沒有應用程式是孤立存在的。開發人員開始預期在網際網路上存取服務，無論服務是開放程式碼應用程式碼和更新，還是透過 REST API 提供應用程式功能的「協力廠商」服務。為了保護從專用網路對網際網路管理的服務的存取，一種常用的方法是網址轉換 (NAT) 假冒。在 NAT 假冒中，專用 IP 位址以多對一的關係轉換為出埠公用介面的 IP 位址，以便能保護專用 IP 位址不被公用存取。  

本指導教學說明瞭如何在虛擬路由器應用裝置 (VRA) 上設定網址轉換 (NAT) 假冒，以連接 {{site.data.keyword.Bluemix_notm}} 專用網路上的安全子網路。本指導教學是根據[使用安全專用網路隔離工作負載](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)指導教學而建置，並新增來源 NAT (SNAT) 配置，其中對來源端位址進行了模糊處理，並使用防火牆規則來保護出埠資料流量。更複雜的 NAT 配置可以在[補充 VRA 文件]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)中找到。
{:shortdesc}

## 目標
{: #objectives}

-	在虛擬路由器應用裝置 (VRA) 上設定來源網址轉換 (SNAT)
-	針對網際網路存取設定防火牆規則

## 使用的服務
{: #products}

本指導教學使用下列 {{site.data.keyword.Bluemix_notm}} 服務： 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

本指導教學可能會產生成本。VRA 只提供於按月定價方案。

## 架構
{: #architecture}

<p style="text-align: center;">

  ![架構](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	記錄必要的網際網路服務。
2.	設定 NAT。
3.	建立網際網路防火牆區域和規則。

## 開始之前
{: #prereqs}

本指導教學在透過[使用安全專用網路隔離工作負載](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)指導教學建立的安全專用網路隔離區中啟用主機，以存取公用網際網路服務。必須先完成該指導教學。 

## 記錄網際網路服務
{: #Document_outbound}

第一步是確定將在公用網際網路上存取的服務，並記錄必須為來自網際網路的出埠和相應入埠資料流量啟用的埠。在稍後的步驟中，防火牆規則將需要此埠清單。 

在此範例中，僅已啟用了 http 和 https 埠，因為這兩種埠可滿足大多數需求。DNS 和 NTP 服務透過 {{site.data.keyword.Bluemix_notm}} 專用網路提供。如果需要這兩種服務及其他服務，例如 SMTP（埠 25）或 MySQL（埠 3306），需要其他防火牆規則。兩個基本埠規則如下：

-	埠 80 (http)
-	埠 443 (https)

驗證協力廠商服務是否支援來源端位址列入白名單。如果支援，將需要 VRA 的公用 IP 位址來配置協力廠商服務，以限制對服務的存取。 


## 透過 NAT 假冒存取網際網路 
{: #NAT_Masquerade}

按照這裡的指示，使用 NAT 假冒為 APP 區域中的主機配置外部網際網路存取。 

1.	透過 SSH 登錄到 VRA，然後進入 \[編輯\]（配置）模式。
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2.	在 VRA 上建立 SNAT 規則，並指定與先前 VRA 佈建指導教學中為 APP 區域子網路/VLAN 確定的相同 `<Subnet Gateway IP>/<cidr>`。 
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Subnet Gateway IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## 建立防火牆
{: #Create_firewalls}

1.	建立防火牆規則 
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

## 建立區域並套用規則
{: #Create_zone}

1.	建立區域 OUTSIDE 來控制對外部網際網路的存取。
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	指派防火牆以控制進出網際網路的資料流量。
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE 
   commit
   save
   ```
   {: codeblock}
3.	驗證 APP 區域中的 VSI 現在是否可以存取網際網路上的服務。使用 SSH 登入到本端 VSI，使用 ping 和 curl 來驗證是否可透過 ICMP 和 TCP 來存取網際網路上的網站。  
   ```bash
   ssh root@<VSI Private IP>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## 移除資源
{:removeresources}
移除本指導教學所建立資源時要採取的步驟。 

VRA 屬於按月付費方案。取消不會退款。建議只有在下個月不會再度需要此 VRA 時才取消。如果需要雙 VRA 高可用性叢集，這個單一 VRA 可以在[閘道詳細資料](https://{DomainName}/classic/network/gatewayappliances)頁面上升級。
{:tip}  

1. 取消全部虛擬伺服器或裸機伺服器
2. 取消 VRA
3. 以支援問題單取消任何其他 VLAN 

## 相關資料
{:related}

-	[VRA 網址轉換]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[NAT 假冒]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[補充 VRA 文件]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)。

