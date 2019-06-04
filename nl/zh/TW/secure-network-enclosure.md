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

# 使用安全專用網路隔離工作負載
{: #secure-network-enclosure}

在公用雲端上，對於 IaaS 應用程式部署模型來說，非常需要隔離且安全的專用網路環境。在建立隔離的專用環境時，防火牆、VLAN、遞送及 VPN 全都是必要的元件。透過這種隔離，虛擬機器和裸機伺服器能夠安全地部署在複雜的多層應用程式拓蹼中，同時能防範公用網際網路上的風險。  

本指導教學主要闡述如何在 {{site.data.keyword.Bluemix_notm}} 上配置[虛擬路由器應用裝置](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-) (VRA)，以建立安全專用網路（隔離區）。VRA 閘道應用裝置在一個自我管理的套裝軟體中提供防火牆、VPN 閘道、網址轉換 (NAT) 和企業級遞送。在本指導教學中，VRA 用於顯示如何在 {{site.data.keyword.Bluemix_notm}} 上建立封閉、隔離的網路環境。在此隔離區中，可以使用為人熟悉且眾所周知的 IP 遞送、VLAN、IP 子網路、防火牆規則、虛擬伺服器和裸機伺服器技術來建立應用程式拓蹼。  

{:shortdesc}

本指導教學是在 {{site.data.keyword.Bluemix_notm}} 上進行標準網路連線功能的起點，不應將其視為依現狀使用的正式作業功能。可以考慮的其他功能包括：
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [硬體防火牆應用裝置](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* [IPSec VPN](https://{DomainName}/catalog/infrastructure/ipsec-vpn)，用於與資料中心進行安全連線。
* 高可用性，透過叢集的 VRA 和雙重上行鏈路來達成。
* 對安全事件的記載和審核。

## 目標 
{: #objectives}

* 部署虛擬路由器應用裝置 (VRA)
* 定義 VLAN 和 IP 子網路，以部署虛擬機器和裸機伺服器
* 使用防火牆規則保護 VRA 和隔離區

## 使用的服務
{: #products}

本指導教學使用下列 {{site.data.keyword.Bluemix_notm}} 服務：
* [Virtual Router Appliance](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)

本指導教學可能會產生成本。VRA 只提供於按月定價方案。

## 架構
{: #architecture}

<p style="text-align: center;">

  ![架構](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. 配置 VPN
2. 部署 VRA 
3. 建立虛擬伺服器
4. 透過 VRA 進行遞送存取
5. 配置隔離區防火牆
6. 定義 APP 區域
7. 定義 INSIDE 區域

## 開始之前
{: #prereqs}

### 配置 VPN 存取

在本指導教學中，建立的網路隔離區不會在公用網際網路上顯示。VRA 和任何伺服器只可透過專用網路進行存取，並且將使用 VPN 進行連線功能。 

1. [確定 VPN 存取已啟用](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     您應該是**主要使用者**，才能啟用 VPN 存取，或是請與主要使用者聯絡以取得存取權。
     {:tip}
2. 選取[使用者清單](https://{DomainName}/iam#/users)中的使用者，以取得 VPN 存取認證。
3. 透過 [Web 介面](https://www.softlayer.com/VPN-Access)登入 VPN，或使用 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 或 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 的 VPN 用戶端。

   針對 VPN 用戶端，從 [VPN Web 存取頁面](https://www.softlayer.com/VPN-Access)使用單一資料中心 VPN 存取點的 FQDN 作為閘道位址，格式為 *vpn.xxxnn.softlayer.com*。
   {:tip}

### 檢查帳戶許可權

請與您的基礎架構主要使用者聯絡，以取得下列許可權：
- **快速許可權** - 基本使用者
- **網路**許可權，用於建立和配置隔離區；需要所有網路許可權。 
- **服務**許可權，用於管理 SSH 金鑰

### 上傳 SSH 金鑰

透過入口網站[上傳 SSH 公開金鑰](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial)，此金鑰將用於存取和管理 VRA 和專用網路。  

### 設定目標資料中心

選擇要部署安全專用網路的 {{site.data.keyword.Bluemix_notm}} 資料中心。 

### 訂購 VLAN

若要在目標資料中心內建立專用隔離區，必須先指派伺服器所需的專用 VLAN。第一個專用 VLAN 和第一個公用 VLAN 是免費的。如果需要更多 VLAN 用於支援多層應用程式拓蹼，這些 VLAN 會收費。 

為了確保在同一資料中心路由器上有足夠的 VLAN 可用並且可以與 VRA 相關聯，建議訂購 VLAN。請參閱[訂購 VLAN](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans)。

## 佈建虛擬路由器應用裝置
{: #VRA}

第一步是部署 VRA；VRA 為專用網路隔離區提供 IP 遞送和防火牆。{{site.data.keyword.Bluemix_notm}} 提供的面向為公用的傳輸 VLAN 可從隔離區存取網際網路，藉由閘道和選用的硬體防火牆，可以建立從公用 VLAN 到安全專用隔離區 VLAN 的連線功能。在此解決方案指導教學中，虛擬路由器應用裝置 (VRA) 提供了此閘道和防火牆周邊。 

1. 在「型錄」中，選取[閘道應用裝置](https://{DomainName}/gen1/infrastructure/provision/gateway)。
3. 在**閘道供應商**區段中，選取 AT&T。可以選擇「最高 20 Gbps」或「最高 2 Gbps」上行鏈路速度。
4. 在**主機名稱**區段中，輸入新 VRA 的主機名稱和網域。
5. 如果勾選**高可用性**勾選框，則您會有兩台使用 VRRP 以作用中/備份設定方式工作的 VRA 裝置。
6. 在**位置**區段中，選取需要 VRA 的「位置」和 **Pod**。
7. 選取「單處理器」或「雙重處理器」。您將獲得伺服器的清單。透過按一下相應圓鈕來選擇伺服器。 
8. 選取 **RAM** 數量。對於正式作業環境，建議使用至少 64 GB RAM。對於測試環境，至少使用 8 GB。
9. 選取 **SSH 金鑰**（選用）。此 SSH 金鑰將安裝在 VRA 上，因此可以使用使用者 vyatta 透過此金鑰來存取 VRA。
10. 硬碟磁碟機。請保留預設值。
11. 在**上行鏈路埠速度**區段中，選取符合您需求的速度、備援和專用和/或公用介面的組合。
12. 在**附加程式**區段中，保留預設值。如果要在公用介面上使用 IPv6，請選取 IPv6 位址。

在右端，可以看到**訂單摘要**。選取_我已閱讀並同意下面列出的協力廠商服務協議：_勾選框，然後按一下**建立**按鈕。將會部署閘道。

[裝置清單](https://{DomainName}/classic/devices)幾乎會立即顯示該 VRA，並具有**時鐘**符號，指示此裝置上正在進行交易處理。在 VRA 建立完成之前，**時鐘**符號會一直存在，除了能檢視裝置詳細資料外，無法對裝置執行任何配置動作。
{:tip}

### 檢閱部署的 VRA

1. 檢查新的 VRA。在[基礎架構儀表板](https://{DomainName}/classic)上，選取左側窗格中的**網路**，然後選取**閘道應用裝置**以移至[閘道應用裝置](https://{DomainName}/classic/network/gatewayappliances)頁面。在**閘道**直欄中選取新建立的 VRA 的名稱，以前往「閘道詳細資料」頁面。 ![](images/solution33-secure-network-enclosure/Gateway-detail.png)

2. 記下 VRA 的`專用`和`公用` IP 位址，以供未來使用。

## 起始 VRA 設定
{: #initial_VRA_setup}

1. 在工作站中，使用預設 **vyatta** 帳戶透過 SSL VPN 登入到 VRA，並接受 SSH 安全提示。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   如果 SSH 提示輸入密碼，說明 SSH 金鑰未包含在建置中。請透過 [Web 瀏覽器](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui)使用 `VRA 專用 IP 位址`來存取 VRA。密碼位於[軟體密碼](https://{DomainName}/classic/devices/passwords)頁面中。在**配置**標籤上，選取「系統/登錄/vyatta 」分支，並新增所需的 SSH 金鑰。
   {:tip}

   設定 VRA 需要使用 `configure` 指令將 VRA 置於 \[編輯\] 模式。在`編輯`模式下，提示字元會從 `$` 變更為 `#`。成功變更 VRA 配置後，可以使用 `compare` 指令檢視變更，並使用 `validate` 指令檢查變更。透過 `commit` 指令確定變更，變更將套用於執行中的配置，並自動儲存到啟動配置中。


   {:tip}
2. 透過僅容許 SSH 登入來增強安全。既然已透過專用網路成功執行 SSH 登入，那就可利用使用者 ID /密碼鑑別來停用存取。 
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
   在本指導教學中，從此處開始，假設在輸入 `configure` 後，在 `edit` 提示字元處輸入了所有 VRA 指令。
3. 檢閱起始配置。
   ```
   show
   ```
   {: codeblock}

   VRA 已針對 {{site.data.keyword.Bluemix_notm}} IaaS 環境進行預先配置。這包括下列內容：
   - NTP 伺服器
   - 名稱伺服器
   - SSH
   - HTTPS Web 伺服器 
   - 預設時區 - 美國/芝加哥
4. 根據需要設定本端時區。使用 Tab 鍵進行自動完成將列出潛在的時區值。
   ```
   set system time-zone <timezone>
   ```
   {: codeblock}
5. 設定 ping 行為。為了輔助遞送和防火牆疑難排解，未停用 ping。 
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. 啟用有狀態防火牆作業。依預設，VRA 防火牆是無狀態的。 
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. 確定並自動儲存對啟動配置的變更。 
   ```
commit
```
   {: codeblock}

## 訂購第一個虛擬伺服器
{: #order_virtualserver}

現在建立的虛擬伺服器用於輔助診斷 VRA 配置錯誤。會先驗證是否能透過 {{site.data.keyword.Bluemix_notm}} 專用網路順利存取 VSI，才在稍後的步驟中透過 VRA 遞送存取該 VSI。
 

1. 訂購[虛擬伺服器](https://{DomainName}/catalog/infrastructure/virtual-server-group)。  
2. 選取**公用虛擬伺服器**，然後繼續。
3. 在訂購頁面上：
   - 將**計費**設定為**按小時**。
   - 設定 *VSI 主機名稱*和*網域名稱*。此網域名稱不用於遞送和 DNS，但應與網路命名標準保持一致。 
   - 將**位置**設定為與 VRA 相同。
   - 將**設定檔**設定為 **C1.1x1**。
   - 新增先前指定的 **SSH 金鑰**。
   - 將**作業系統**設定為 **CentOS 7.x - 最低**。
   - 在**上行鏈路埠速度**中，必須將網路介面從預設*公用和專用* 變更為僅指定**專用網路上行鏈路鏈路**。這可確保新伺服器無法直接存取網際網路，並且存取由 VRA 上的遞送和防火牆規則進行控制。
   - 將**專用 VLAN** 設定為先前訂購的專用 VLAN 的 VLAN ID。
4. 按一下勾選方框以接受「協力廠商」服務合約，然後按一下**建立**。
5. 在[裝置](https://{DomainName}/classic/devices)頁面上或透過電子郵件監視完成情況。 
6. 記下 VSI 的*專用 IP 位址* 以用於後續步驟，並在**裝置詳細資料**頁面的**網路**區段下，確認 VSI 是否指派給正確的 VLAN。如果沒有，請刪除此 VSI，然後在正確的 VLAN 上建立新的 VSI。 
7. 在本端工作站中，透過 VPN 使用 ping 和 SSH 來驗證是否可以透過 {{site.data.keyword.Bluemix_notm}} 專用網路成功存取 VSI。
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## 透過 VRA 遞送 VLAN 存取
{: #routing_vlan_via_vra}

此時虛擬伺服器的專用 VLAN 就已經由 {{site.data.keyword.Bluemix_notm}} 管理系統關聯到此 VRA。在此階段，仍可透過 {{site.data.keyword.Bluemix_notm}} 專用網路上的 IP 遞送來存取 VSI。現在，將透過 VRA 遞送子網路來建立安全專用網路，並透過確認現在無法存取 VSI 來進行驗證。 

1. 透過[閘道應用裝置](https://{DomainName}/classic/network/gatewayappliances)頁面前往 VRA 的「閘道詳細資料」，並在該頁面的下半區段找到**關聯的 VLAN** 區段。關聯的 VLAN 將在這裡列出。 
2. 如果此時需要新增其他 VLAN，請導覽至**關聯 VLAN** 區段。應已啟用下拉方框*選取 VLAN*，並可以選取其他佈建的 VLAN。![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png)

   如果未顯示符合條件的 VLAN，說明在與 VRA 相同的路由器上沒有可用的 VLAN。這將需要提交[支援問題單](https://{DomainName}/unifiedsupport/cases/add)，以在與 VRA 相同的路由器上要求專用 VLAN。
   {:tip}
5. 選取要與 VRA 關聯的 VLAN，然後按一下「儲存」。起始 VLAN 關聯可能需要幾分鐘才能完成。完成後，該 VLAN 應該會顯示在**關聯的 VLAN** 標題下。 

在此階段，VLAN 和關聯的子網路不透過 VRA 進行保護或遞送，並且 VSI 可透過 {{site.data.keyword.Bluemix_notm}} 專用網路進行存取。VLAN 的狀態將顯示為*略過*。

4. 選取右側直欄中的**動作**，然後選取**遞送 VLAN** 以透過 VRA 遞送 VLAN/子網路。這將需要幾分鐘時間。重新整理畫面後，該 VLAN 會顯示為*已遞送*。 
5. 選取 [VLAN 名稱](https://{DomainName}/classic/network/vlans/)以檢視 VLAN 詳細資料。可以看到佈建的 VSI 以及指派的主要 IP 子網路。記下「專用 VLAN ID \<nnnn\>」（在此範例中為 1199），因為在後面的步驟中將用到此 ID。 
6. 選取[子網路](https://{DomainName}/classic/network/subnets)以查看 IP 子網路詳細資料。記下子網路、閘道位址和 CIDR (/26)，因為進一步 VRA 配置需要這些資訊。在專用網路上已佈建 64 個主要 IP 位址，要找到閘道位址，可能需要選取第 2 頁或第 3 頁。 
7. 驗證子網路/VLAN 是否遞送到 VRA，以及是否**無法**在工作站中透過管理網路使用 `ping` 來存取 VSI。 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}

這就透過 {{site.data.keyword.Bluemix_notm}} 主控台完成了 VRA 的設定。現在，透過 SSH 直接在 VRA 上執行配置隔離區和 IP 遞送的其他工作。 

## 配置 IP 遞送和安全隔離區
{: #vra_setup}

確定 VRA 配置後，將變更執行配置，並且變更會自動儲存到啟動配置中。

如果希望回到先前的有效配置，依預設可以檢視、比較或還原最後 20 個確定點。如需確定和儲存配置的相關詳細資料，請參閱 [Vyatta Network OS Basic System Configuration Guide](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)。
   ```bash
   show system commit 
   rollback n
   compare
   ```
   {: codeblock}

### 配置 VRA IP 遞送

配置 VRA 虛擬網路介面以從 {{site.data.keyword.Bluemix_notm}} 專用網路遞送到新子網路。  

1. 透過 SSH 登入到 VRA。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}
2. 使用先前步驟中記錄的專用 VLAN ID、子網路閘道 IP 位址和 CIDR 來建立新的虛擬介面。CIDR 通常為 /26。 
   ```
   set interfaces bonding dp0bond0 vif <VLAN ID> address <Subnet Gateway IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   使用 **`<Subnet Gateway IP>`** 位址至關重要。這通常比子網路位址的起始位址大 1。輸入無效的閘道位址將導致錯誤：`配置路徑：介面綁定 dp0bond0 vif xxxx 位址 [x.x.x.x] 無效`。請更正指令並重新輸入。可以在「網路」>「IP 管理」>「子網路」中查閱該位址。按一下需要知道閘道位址的子網路。「清單」中具有說明**閘道**的第二個項目就是要作為 <Subnet Gateway IP>/<CIDR> 輸入的 IP 位址。
   {: tip}

3. 列出新的虛擬介面 (vif)： 
   ```
   show interfaces
   ```
   {: codeblock}

   下面是範例介面配置，顯示了 vif 1199 和子網路閘道位址。
   ![](images/solution33-secure-network-enclosure/show_interfaces.png)
4. 在工作站中透過管理網路驗證 VSI 是否再次可存取。 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
   
   如果無法存取 VSI，請檢查 VRA IP 遞送表是否按預期進行了配置。如果需要，請刪除並重建路徑。若要在配置模式下執行 show 指令，可以使用 run 指令    
   ```bash
    run show ip route <Subnet Gateway IP>
   ```
   {: codeblock}

這就完成了 IP 遞送配置。

### 配置安全隔離區

透過配置區域和防火牆規則，可建立安全專用網路隔離區。在繼續之前，請檢閱有關[防火牆配置](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls)的 VRA 文件。 

將定義兩個區域：
   - INSIDE：IBM 專用和管理網路
   - APP：專用網路隔離區中的使用者 VLAN 和子網路		

1. 定義防火牆和預設值。
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   如果意外執行了 set 指令兩次，您將收到一條訊息：*「配置路徑 xxxxxxxx 無效。節點已存在」*。可以將其忽略。若要變更不正確的參數，需要先使用 "delete security xxxxx xxxx xxxxxx" 來刪除該節點。
   {:tip}
2. 建立 {{site.data.keyword.Bluemix_notm}} 專用網路資源群組。此位址群組定義可以存取隔離區的 {{site.data.keyword.Bluemix_notm}} 專用網路以及可從隔離區存取的網路。對安全隔離區進行存取和從安全隔離區進行存取需要兩組 IP 位址，這兩組位址是 SSL VPN 資料中心和 {{site.data.keyword.Bluemix_notm}} 服務網路（後端/專用網路）。[{{site.data.keyword.Bluemix_notm}} IP 範圍](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges)提供了需要容許的 IP 範圍的完整清單。 
   - 定義要用於 VPN 存取的資料中心的 SSL VPN 位址。從 {{site.data.keyword.Bluemix_notm}} IP 範圍的 SSL VPN 區段中，選取用於資料中心或 DC 叢集的 VPN 存取點。下面的範例顯示了 {{site.data.keyword.Bluemix_notm}} 倫敦資料中心的 VPN 位址範圍。
     ```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - 為 WDC04、DAL01 和目標資料中心定義 {{site.data.keyword.Bluemix_notm}}「服務網路（在後端/專用網路上）」的位址範圍。下面的範例是 WDC04（兩個位址）、DAL01 和 LON06。
     ```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. 為使用者 VLAN 和子網路建立 APP 區域，並為 {{site.data.keyword.Bluemix_notm}} 專用網路建立 INSIDE 區域。指派先前建立的防火牆。區域定義使用 VRA 網路介面名稱來識別與每個 VLAN 關聯的區域。用於建立 APP 區域的指令，需要指定與先前 VRA 關聯的 VLAN 的 VLAN ID。這在下面強調顯示為 `<VLAN ID>`。
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
4. 確定配置，然後在工作站中，使用 ping 來驗證防火牆現在是否拒絕透過 VRA 流至 VSI 的資料流量： 
   ```
commit
```
   {: codeblock}

   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
5. 針對 UDP、TCP 和 ICMP 定義防火牆存取規則。
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
6. 驗證防火牆存取權。 
   - 確認 INSIDE 到 APP 防火牆現在是否容許來自本端機器的 ICMP 和 UDP/TCP 資料流量。
     ```bash
     ping <VSI Private IP Address>
     SSH root@<VSI Private IP Address>
     ```
     {: codeblock}
   - 確認 APP 到 INSIDE 防火牆是否容許 ICMP 和 UDP/TCP 資料流量。使用 SSH 登入到 VSI，然後在 10.0.80.11 和 10.0.80.12 上，對其中一個 {{site.data.keyword.Bluemix_notm}} 名稱伺服器執行 ping 操作。
     ```bash
     SSH root@<VSI Private IP Address>
     [root@vsi  ~]# ping 10.0.80.11 
     ```
     {: codeblock}
7. 在工作站中透過 SSH 來驗證是否能持續存取 VRA 管理介面。如果可持續存取，請檢閱並儲存配置。否則，重新開機 VRA 後將恢復為某個有效配置。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   ```
   show security  
   ```
   {: codeblock}

### 除錯防火牆規則

可以透過 VRA 操作命令提示字元來檢視防火牆日誌。在此配置中，僅記錄每個區域捨棄的資料流量，以輔助診斷防火牆配置錯誤。  

1. 檢閱防火牆日誌，以瞭解被拒絕的資料流量。定期檢閱日誌，可識別 APP 區域中的伺服器是有效或錯誤地試圖聯絡 IBM 網路上的服務。 
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. 如果服務或伺服器不可聯繫，且防火牆日誌中未顯示任何相關內容，請使用先前的 `<VLAN ID>` 來驗證 VRA 網路介面上是否有來自 {{site.data.keyword.Bluemix_notm}} 專用網路的預期 ping/SSH IP 資料流量，或者在 VRA 介面上是否有流至 VLAN 的預期 ping/SSH IP 資料流量。
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN ID> traffic
   ```

## 保護 VRA
{: #securing_the_vra}

1. 套用 VRA 安全原則。依預設，以原則為基礎的防火牆分區並不會保護對 VRA 本身的存取。這是透過控制平面原則 (CPP) 配置的。VRA 將基本 CPP 規則集作為範本提供。請將其合併到配置中：
   ```bash
   merge /opt/vyatta/etc/cpp.conf 
   ```
   {: codeblock}

這將建立名為 `CPP` 的新防火牆規則集，檢視其他規則，並以 \[編輯\] 模式確定。 
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. 保護公用 SSH 存取。由於 Vyatta 韌體目前存在待解決的問題，因此建議不要使用 `set service SSH listen-address x.x.x.x` 來限制透過公用網路進行的 SSH 管理存取。或者，可以透過 CPP 防火牆阻止對 VRA 公用介面使用的公用 IP 位址範圍進行外部存取。這裡使用的 `<VRA Public IP Subnet>` 與 `<VRA Public IP Address>` 相同，其中最後一個八位元組為零 (x.x.x.0)。 
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <VRA Public IP Subnet>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit 
   ```
   {: codeblock}
3. 透過 IBM 內部網路驗證 VRA SSH 管理存取權。如果在執行確定後無法透過 SSH 來存取 VRA，可以透過 VRA 「裝置詳細資料」頁面上提供的 KVM 主控台中的「動作」下拉功能表來存取 VRA。

這就完成了安全專用網路隔離區的設定，透過隔離區，可保護包含 VLAN 和子網路的單一防火牆區域。可以按照相同的指示來新增其他防火牆區域、規則、虛擬伺服器和裸機伺服器、VLAN 以及子網路。 

## 移除資源
{: #removeresources}

在此步驟中，您將清除資源，以移除以上建立的內容。

VRA 屬於按月付費方案。取消不會退款。建議只有在下個月不會再度需要此 VRA 時才取消。如果需要雙 VRA 高可用性叢集，這個單一 VRA 可以在[閘道詳細資料](https://{DomainName}/classic/network/gatewayappliances/)頁面上升級。
{:tip}  

- 取消全部虛擬伺服器或裸機伺服器
- 取消 VRA
- 以支援問題單取消任何其他 VLAN 

## 相關內容
{: #related}

- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [靜態和可攜式 IP 子網路](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence 平台](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Vyatta 文件](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
