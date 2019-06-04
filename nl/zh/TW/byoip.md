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

# 自帶 IP 位址
{: #byoip}

自帶 IP (BYOIP) 是常見需求，用於將現有用戶端網路連接至 {{site.data.keyword.Bluemix_notm}} 上佈建的基礎架構。其目的通常是採用根據用戶端現有 IP 定址方法的單一 IP 位址空間，最大程度減少對用戶端網路遞送配置和作業的變更。

本指導教學簡要概述 BYOIP 實作模式，在實作[使用安全專用網路隔離工作負載](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)指導教學中所述的安全隔離區時，可以將這些實作模式與 {{site.data.keyword.Bluemix_notm}} 和決策樹狀結構一起用於識別適當的模式。設定時，可能需要現場網路團隊、{{site.data.keyword.Bluemix_notm}} 技術支援或 IBM Services 進行其他輸入。

{:shortdesc}

## 目標
{: #objectives}

* 瞭解 BYOIP 實作模式
* 選取 {{site.data.keyword.Bluemix_notm}} 的實作模式。

## {{site.data.keyword.Bluemix_notm}} IP 定址

{{site.data.keyword.Bluemix_notm}} 利用多個專用位址範圍，通常是 10.0.0.0/8，在有些情況下，這些範圍可能與現有用戶端網路衝突。在位址發生衝突時，有多個模式支援 BYOIP 容許與 IBM Cloud 網路進行互操作。

-	網址轉換
-	GRE（通用遞送封裝）通道
-	使用 IP 別名的 GRE 通道 
-	虛擬層疊網路

模式的選擇由計劃在 {{site.data.keyword.Bluemix_notm}} 上管理的應用程式確定。有兩個關鍵方面，應用程式對模式實作的靈敏度，以及用戶端網路和 {{site.data.keyword.Bluemix_notm}} 之間位址範圍的重疊程度。如果打算使用與 {{site.data.keyword.Bluemix_notm}} 的專用的專用網路連線，則還需要額外的考量。建議考慮使用 BYOIP 的所有使用者閱讀 [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link) 文件。對於 {{site.data.keyword.BluDirectLink}} 使用者，應遵循關聯的指引以遵守這裡提供的資訊。

## 實作模式概觀
{: #patterns_overview}

1. **NAT**：內部部署用戶端路由器中的 NAT 位址轉換。執行內部部署 NAT 以將用戶端定址方法轉換到 {{site.data.keyword.Bluemix_notm}} 為佈建的 IaaS 服務指派的 IP 位址。  
2. **GRE 通道**：在 {{site.data.keyword.Bluemix_notm}} 與內部部署網路之間藉由 GRE 通道遞送 IP 資料流量（通常使用 VPN），可統一定址方法。這是[本指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)中說明的情境。 

   存在兩個子模式，取決於位址空間重疊的可能性。
     * 無位址重疊：位址範圍沒有位址重疊，網路之間沒有衝突風險。
     * 局部位址重疊：用戶端和 {{site.data.keyword.Bluemix_notm}} IP 位址空間使用相同的位址範圍，有出現重疊和衝突的可能性。在這種情況下，將選擇 {{site.data.keyword.Bluemix_notm}} 專用網路中不重疊的用戶端子網路位址。

3. GRE 通道 + IP 別名藉由在內部部署網路與指派給 {{site.data.keyword.Bluemix_notm}} 上伺服器的別名 IP 位址之間的 GRE 通道來遞送 IP 資料流量，可統一定址方法。這是[本指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)中說明的情境的特殊情況。在 {{site.data.keyword.Bluemix_notm}} 上佈建的虛擬和裸機伺服器上建立相容 IP 子網路的其他介面和 IP 別名，由 VRA 上適當的遞送配置提供支援。

4. 虛擬層疊網路 [{{site.data.keyword.Bluemix_notm}} Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) 支援對 {{site.data.keyword.Bluemix_notm}} 上的完全虛擬環境使用 BYOIP。可以將其視為[本指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)中說明的安全專用網路隔離區的替代方案。

還可以考慮 VMware NSX 之類的解決方案，該解決方案透過 {{site.data.keyword.Bluemix_notm}} 網路在一個層級中實作虛擬層疊網路。虛擬層疊中的所有 BYOIP 位址獨立於 {{site.data.keyword.Bluemix_notm}} 網址範圍。請參閱「開始使用 VMware 和 [{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud)」。

## 模式決策樹狀結構
{: #decision_tree}

這裡的決策樹狀結構可用於確定適當地實作模式。 

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

下列附註提供進一步指引：

### 您的應用程式使用 NAT 有問題嗎？
{: #nat_consideration}

在下列兩種不同的情況下，NAT 可能會產生問題。在這些情況下，不應使用 NAT。 

- Microsoft AD 網域通訊之類的一些應用程式和 P2P 應用程式可能在使用 NAT 時遇到技術問題。
- 在不明伺服器需要與 {{site.data.keyword.Bluemix_notm}} 通訊或者 {{site.data.keyword.Bluemix_notm}} 與內部部署伺服器之間需要幾百個雙向連線時。在這種情況下，由於無法事先識別對映，所以無法在用戶端路由器/NAT 表格上配置所有對映。


### 無位址重疊
{: #no-overlap}

內部部署網路中使用 10.0.0.0/8 嗎？內部部署與 {{site.data.keyword.Bluemix_notm}} 專用網路之間沒有位址重疊時，可以在內部部署與 IBM Cloud 之間使用[本指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)中所說明的 GRE 通道，而無需進行 NAT 轉換。這需要與現場網路團隊一起檢閱網址使用情況。 

### 局部位址重疊
{: #partial_overlap}

如果在內部部署網路中使用 10.0.0.0/8 範圍中的任何值，{{site.data.keyword.Bluemix_notm}} 網路上是否提供非重疊子網路？與現場網路團隊一起檢閱現有網址使用情況，並聯絡 {{site.data.keyword.Bluemix_notm}} 技術業務人員以識別可用的非重疊網路。 

### IP 別名化是否有問題？
{: #ip_alias}

如果沒有重疊安全位址，則可以在安全專用網路隔離區中部署的虛擬和裸機伺服器上實作 IP 別名化。IP 別名化在每部伺服器的一個以上網路介面上指派多個子網路位址。 

IP 別名化是常用的技巧，不過建議檢閱伺服器和應用程式配置，以確定這些配置在多重主目錄和 IP 別名配置下是否運作良好。  

將需要使用 VRA 上的其他遞送配置來為 BYOIP 子網路建立動態路徑（例如 BGP）或靜態路徑。 

## 相關內容
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
