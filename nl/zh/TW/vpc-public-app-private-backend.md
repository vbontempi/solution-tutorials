---
copyright:
  years: 2019
lastupdated: "2019-04-02"
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
{:important: .important}

# Virtual Private Cloud 中的專用和公用子網路
{: #vpc-public-app-private-backend}

IBM 將接受限量客戶參與 2019 年 4 月初開辦的 VPC「搶先體驗」計劃，並在接下來幾個月開放擴大使用。如果您的組織希望體驗 IBM Virtual Private Cloud，請填寫此[申請表](https://{DomainName}/vpc){: new_window}，IBM 業務代表將與您聯絡，進行下一步。
{: important}

本指導教學引導您使用公用和專用子網路以及每個子網路中的虛擬伺服器實例 (VSI) 建立您自己的 {{site.data.keyword.vpc_full}} (VPC)。VPC 是共用雲端基礎架構上與其他虛擬網路邏輯隔離的您自己的專用雲端。

[子網路](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#subnet)是 IP 位址範圍。子網必須連結至單個區域，不能跨多個區域或地區。對於 VPC 的用途，子網路的重要性質是子網路可以彼此隔離，也可以按通常方式互連。子網路隔離可以藉由網路[存取控制清單](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#access-control-list) (ACL) 來實現，ACL 充當防火牆，用於控制子網路之間的資料封包流程。同樣地，[安全群組](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#security-group) (SG) 充當虛擬防火牆，用於控制進出個別 VSI 的資料封包流程。

公用子網路用於必須公開給外部世界的資源。存取受限而絕不能由外部世界直接存取的資源必須放在專用子網路中。這種子網路上的實例可以是後端資料庫或者您不希望可公開存取的一些密碼儲存庫。您將定義 SG 以容許或者拒絕進入 VSI 的資料流量。
{:shortdesc}

簡言之，使用 VPC，您可以

- 建立軟體定義的網路 (SDN)，
- 隔離工作負載，
- 精細控制入埠和出埠資料流量。

## 目標

{: #objectives}

- 瞭解虛擬專用雲端可用的基礎架構物件
- 瞭解如何建立虛擬專用雲端、子網路和伺服器實例
- 瞭解如何套用安全群組以保護對伺服器的存取

## 使用的服務

{: #services}

本指導教學使用下列運行環境及服務：

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

![架構](images/solution40-vpc-public-app-private-backend/Architecture.png)


1. 管理員 (DevOps) 在雲端中設定所需的基礎架構（VPC、子網路、具有規則的安全群組、VSI）。
2. 網際網路使用者向前端的 Web 伺服器發出 HTTP/HTTPS 要求。
3. 前端從安全的後端要求專用資源，並將結果提供給使用者。

## 開始之前

{: #prereqs}

- 檢查使用者許可權。您的使用者帳戶務必要有足夠的許可權以便建立及管理 VPC 資源。如需必要許可權的清單，請參閱[授與 VPC 使用者所需的許可權](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources)。

- 您需要 SSH 金鑰才能連接至虛擬伺服器。如果您沒有 SSH 金鑰，請參閱[建立金鑰的指示](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)。

## 建立 Virtual Private Cloud
{: #create-vpc}

建立您自己的 {{site.data.keyword.vpc_short}}：

1. 導覽至 [VPC 概觀](https://{DomainName}/vpc/overview)頁面，然後按一下**建立 VPC**。
2. 在**新建 Virtual Private Cloud** 區段下：
   * 輸入 **vpc-pubpriv** 作為 VPC 的名稱。
   * 選取**資源群組**。
   * 選擇性地新增**標籤**以組織您的資源。
3. 選取**建立新的預設值（全部容許）**作為您的 VPC 預設存取控制清單 (ACL)。
1. 取消勾選 SSH，從**預設安全群組**執行 ping 操作。
4. 在**新建 VPN 的子網路**下：
   * 輸入 **vpc-pubpriv-backend-subnet** 作為唯一名稱。
   * 選取位置。
   * 以 CIDR 表示法輸入子網路的 IP 範圍，亦即 **10.xxx.0.0/24**。保留**位址字首**原樣，並將**位址數目**選取為 256。
5. 為您的子網路存取控制清單 (ACL) 選取**使用 VPC 預設值**。您可以之後再配置入埠和出埠規則。
6. 按一下**建立 Virtual Private Cloud** 來佈建實例。

如果連接到專用子網路的 VSI 需要存取網際網路以載入軟體，請將公用閘道切換到**附加**，因為連接公用閘道將容許所有連接的資源與公用網際網路通訊。一旦 VSI 擁有所需的所有軟體，請讓公用閘道回到**分離**狀態，以便該子網路無法達到公用網際網路。
{: important}

若要確認子網路的建立，請按一下左側窗格上的**子網路**，並等待狀態變更為**可用**。您可以在**子網路**下建立新的子網路。

## 建立後端安全群組和 VSI
{: #backend-subnet-vsi}

在本區段中，您將為後端建立安全群組和虛擬伺服器實例。

### 建立後端安全群組

依預設，安全群組與 VPC 一起建立，以容許進入所連接實例的所有 SSH（TCP 埠 22）和 Ping（ICMP 類型 8）資料流量。

若要為後端建立新的安全群組，請執行下列動作：  
1. 按一下**網路**下的**安全群組**，然後按一下**新建安全群組**。  
2. 輸入 **vpc-pubpriv-backend-sg** 作為名稱，然後選取先前建立的 VPC。  
3. 按一下**建立安全群組**。

您稍後將編輯安全群組以新增入埠和出埠規則。

### 建立後端虛擬伺服器實例

若要在新建立的子網路中建立虛擬伺服器實例，請執行下列動作：

1. 按一下**子網路**下的後端子網路。
2. 按一下**連接的實例**，然後按一下**新建實例**。
3. 輸入唯一名稱，並選取 **vpc-pubpriv-backend-vsi**。然後，選取先前建立的 VPC 和**位置**，如前所述。
4. 選擇 **Ubuntu Linux** 映像檔，按一下**所有設定檔**，在**運算**下，選擇具有 2 個 vCPU 和 4 GB RAM 的 **c-2x4**。
5. 針對 **SSH 金鑰**，挑選您先前建立的 SSH 金鑰。
6. 在**網路介面**下，按一下安全群組旁的**編輯**圖示。
   * 選取 **vpc-pubpriv-backend-subnet** 作為子網路。
   * 取消勾選預設安全群組，並將 **vpc-pubpriv-backend-sg** 勾選為作用中。
   * 按一下**儲存**。
7. 按一下**建立虛擬伺服器實例**。

## 建立前端子網路、安全群組和 VSI
{: #frontend-subnet-vsi}

與後端類似，您將建立具有虛擬伺服器實例和安全群組的前端子網路。

### 建立前端子網路

為前端建立新的子網路：

1. 按一下左側窗格的**網路**下的**子網路** > **新建子網路**。
   * 輸入 **vpc-pubpriv-frontend-subnet** 作為名稱，然後選取您建立的 VPC。
   * 選取位置。
   * 以 CIDR 表示法輸入子網路的 IP 範圍，亦即 **10.xxx.1.0/24**。保留**位址字首**原樣，並將**位址數目**選取為 256。
1. 為您的子網路存取控制清單 (ACL) 選取 **VPC 預設值**。您可以之後再配置入埠和出埠規則。
1. 假設子網路中的所有虛擬伺服器實例會連接浮動 IP，則不需要啟用子網路的公用閘道。虛擬伺服器實例會透過浮動 IP 而具有網際網路連線功能。
1. 按一下**建立子網路**來佈建它。

### 建立前端安全群組

為前端建立新的安全群組：
1. 按一下「網路」下的**安全群組**，然後按一下**新建安全群組**。
2. 輸入 **vpc-pubpriv-frontend-sg** 作為名稱，並選取稍早之前建立的 VPC。
3. 按一下**建立安全群組**。

### 建立前端虛擬伺服器實例

若要在新建立的子網路中建立虛擬伺服器實例，請執行下列動作：

1. 按一下**子網路**下的前端子網路。
2. 按一下**連接的實例**，然後按一下**新建實例**。
3. 輸入唯一名稱 **vpc-pubpriv-frontend-vsi**，選取先前建立的 VPC，然後選取與之前相同的**位置**。
4. 選取 **Ubuntu Linux** 映像檔、按一下**所有設定檔**，然後在**運算**底下選擇 **c-2x4**：2 個 vCPU 和 4 GB RAM
5. 針對 **SSH 金鑰**，挑選您先前建立的 SSH 金鑰。
6. 在**網路介面**下，按一下安全群組旁的**編輯**圖示。
   * 選取 **vpc-pubpriv-frontend-subnet** 作為子網路。
   * 取消勾選預設安全群組，並啟動 **vpc-pubpriv-frontend-sg**。
   * 按一下**儲存**。
   * 按一下**建立虛擬伺服器實例**。
7. 等到 VSI 的狀態變更為**已開啟電源**。然後，選取前端 VSI **vpc-pubpriv-frontend-vsi**，捲動到**網路介面**並按一下**浮動 IP** 下的**預留**以將公用 IP 位址關聯到您的前端 VSI。將相關聯的 IP 位址儲存至剪貼簿，以便未來參考。

## 設定前端與後端之間的連線功能
{: #setup-connectivity-frontend-backend}

所有伺服器就位後，在本區段中，您將設定連線功能以容許前端和後端伺服器之間進行一般作業。

### 配置前端安全群組

1. 導覽至**網路**區段的**安全群組**，然後按一下 **vpc-pubpriv-frontend-sg**。
2. 首先，使用**新增規則**新增下列**入埠**規則。這些規則容許送入 HTTP 要求和 Ping (ICMP)。

	<table>
   <thead>
      <tr>
         <td><strong>來源</strong></td>
         <td><strong>通訊協定</strong></td>
         <td><strong>值</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>任何 - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>從：<strong>80</strong> 到 <strong>80</strong></td>
      </tr>
      <tr>
         <td>任何 - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>從：<strong>443</strong> 到 <strong>443</strong></td>
      </tr>
      <tr>
         <td>任何 - 0.0.0.0/0 </td>
	      <td>ICMP</td>
	      <td>類型：<strong>8</strong>，程式碼：<strong>留空</strong></td>
      </tr>
   </tbody>
   </table>

3. 接下來，新增這些**出埠**規則。

   <table>
   <thead>
      <tr>
         <td><strong>目的地</strong></td>
         <td><strong>通訊協定</strong></td>
         <td><strong>值</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>類型：<strong>安全群組</strong> - 名稱：<strong>vpc-pubpriv-backend-sg</strong></td>
         <td>TCP</td>
         <td>後端伺服器的埠，請參閱提示</td>
      </tr>
   </tbody>
   </table>

這裡有典型後端服務的埠。MySQL 使用埠 3306，PostgreSQL 使用埠 5432。Db2 在埠 50000 或 50001 上進行存取。依預設，Microsoft SQL Server 使用埠 1433。
[常用埠的許多清單之一可在 Wikipedia 上找到](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)。
{:tip }

### 配置後端安全群組
以類似於前端的方式，為後端配置安全群組。

1. 導覽至**網路**區段的**安全群組**，然後按一下 **vpc-pubpriv-backend-sg**。
2. 使用**新增規則**新增下列**入埠**規則。該規則容許連線到後端服務。

   <table>
   <thead>
      <tr>
         <td><strong>來源</strong></td>
         <td><strong>通訊協定</strong></td>
         <td><strong>值</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>類型：<strong>安全群組</strong> - 名稱：<strong>vpc-pubpriv-frontend-sg</strong></td>
         <td>TCP</td>
         <td>後端伺服器的埠</td>
      </tr>
   </tbody>
   </table>


## 安裝軟體及執行維護作業
{: #install-software-maintenance-tasks}

遵循[使用防禦主機安全地存取遠端實例](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)中提及的步驟，使用充當 `jump` 伺服器和維護安全群組的防禦主機來安全地維護伺服器。


## 移除資源
{: #remove-resources}

1. 在 VPC 管理主控台中，按一下**浮動 IP**，再按一下 VSI 的 IP 位址，然後在「動作」功能表中選取**發佈**。確認您想要釋放 IP 位址。
2. 接下來，切換至**虛擬伺服器實例**，然後**刪除**您的實例。實例將會刪除，它們的狀態會維持在**正在刪除**一陣子。請務必偶爾重新整理瀏覽器。
3. VSI 消失之後，切換至**子網路**。如果子網路有連接的公用閘道，則請按一下子網路名稱。在子網路詳細資料中，分離公用閘道。可以從概觀頁面刪除沒有公用閘道的子網路。刪除您的子網路。
4. 刪除子網路後，切換到 **VPC** 標籤，並刪除您的 VPC。

使用主控台時，您可能需要重新整理瀏覽器，才能在刪除資源之後看到更新的狀態資訊。
{:tip}

## 擴展指導教學
{: #expand-tutorial}

想要新增或延伸此指導教學？以下是一些構想：

- 新增[負載平衡器](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-load-balancer)以跨多個實例分散入埠資料流量。
- 建立[虛擬專用網路](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn) (VPN)，以便您的 VPC 可以安全地連接至其他專用網路，例如內部部署網路或其他 VPC。


## 相關內容
{: #related}

- [VPC 名詞解釋](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)
- [使用 IBM Cloud CLI 的 VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli#creating-a-vpc-using-the-ibm-cloud-cli)
- [使用 REST API 的 VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis#creating-a-vpc-using-the-rest-apis)
