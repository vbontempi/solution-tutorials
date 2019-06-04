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

# 使用防禦主機安全存取遠端實例
{: #vpc-secure-management-bastion-server}

IBM 將接受限量客戶參與 2019 年 4 月初開辦的 VPC「搶先體驗」計劃，並在接下來幾個月開放擴大使用。如果您的組織希望體驗「IBM 虛擬專用雲端」，請填寫此[申請表](https://{DomainName}/vpc){: new_window}，IBM 業務代表將聯絡您進行下一步。
{: important}

本指導教學將全程指導您部署防禦主機，以安全存取虛擬專用雲端中的遠端實例。防禦主機是在公用子網路中佈建的實例，可以透過 SSH 存取。設定後，防禦主機充當**跳躍**伺服器，容許安全地連線到專用子網路中佈建的實例。

若要降低 VPC 中伺服器的敞口，您將建立和使用防禦主機。個別伺服器上的管理作業將使用 SSH 執行，透過防禦主機進行代理。僅容許使用連接到伺服器的特殊維護安全群組來存取伺服器以及從伺服器定期存取網際網路（例如，用於軟體安裝）。
{:shortdesc}

## 目標
{: #objectives}

* 瞭解如何設定防禦主機和具有規則的安全群組
* 透過防禦主機安全管理伺服器

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：  

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)  
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

  ![架構](images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png)

1. 在雲端中設定所需的基礎架構（子網路、具有規則的安全群組、VSI）之後，管理 (DevOps) 使用專用的 SSH 金鑰連接 (SSH) 到防禦主機。
2. 管理指派具有合適出埠規則的維護安全群組。
3. 管理透過防禦主機安全地連接 (SSH) 到實例的專用 IP 位址，以安裝或更新任何所需軟體，例如，Web 伺服器
4. 網際網路使用者向 Web 伺服器發出 HTTP/HTTPS 要求。

## 開始之前
{: #prereqs}

- 檢查使用者許可權。您的使用者帳戶務必要有足夠的許可權以便建立及管理 VPC 資源。如需必要許可權的清單，請參閱[授與 VPC 使用者所需的許可權](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources)。
- 您需要 SSH 金鑰才能連接至虛擬伺服器。如果您沒有 SSH 金鑰，請參閱[建立金鑰的指示](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)。
- 本指導教學假設您要在現有[虛擬專用雲端](https://{DomainName}/vpc/network/vpcs)中新增防禦主機。**如果您的帳戶中沒有虛擬專用雲端，請建立一個虛擬專用雲端，然後繼續後續步驟。**

## 建立防禦主機
{: #create-bastion-host}

在本區段中，您將建立和配置防禦主機以及不同子網路中的安全群組。

### 建立子網路
{: #create-bastion-subnet}

1. 在左窗格的**網路**下，按一下**子網路**，然後按一下**新建子網路**。  
   * 輸入 **vpc-secure-bastion-subnet** 作為名稱，然後選取您建立的 VPC。  
   * 選取一個位置和區域。  
   * 以 CIDR 表示法輸入子網路的 IP 範圍，亦即 **10.xxx.0.0/24**。保留**位址字首**原樣，並將**位址數目**選取為 256。
1. 為您的子網路存取控制清單 (ACL) 選取 **VPC 預設值**。您可以之後再配置入埠和出埠規則。
1. 將**公用閘道**切換成**已連接**。 
1. 按一下**建立子網路**來佈建它。

### 建立和配置防禦安全群組

接下來建立安全群組，並配置防禦 VSI 的入埠規則。

1. 導覽至**安全群組**並按一下**新建安全群組**。輸入 **vpc-secure-bastion-sg** 作為名稱，並選取您的 VPC。 
2. 現在，透過在入埠區段中按一下**新增規則**來建立下列入埠規則。這些規則容許 SSH 存取和 Ping (ICMP) 操作。
 
	**入埠規則：**
	<table>
	   <thead>
	      <tr>
	         <td><strong>來源</strong></td>
	         <td><strong>通訊協定</strong></td>
	         <td><strong>值</strong></td>
	      </tr>
	   <tbody>
	      <tr>
	         <td>任何 - 0.0.0.0/0 </td>
	         <td>TCP</td>
	         <td>從：<strong>22</strong> 到 <strong>22</strong></td>
	      </tr>
         <tr>
            <td>任何 - 0.0.0.0/0 </td>
	         <td>ICMP</td>
	         <td>類型：<strong>8</strong>，程式碼：<strong>留空</strong></td>
         </tr>
	   </tbody>
	</table>

   若要進一步增強安全，入埠資料流量可以限制為公司網路或者典型的家庭網路。您可以執行 `curl ipecho.net/plain ; echo` 以獲得網路的外部 IP 位址，並改為使用該位址。{:tip }

### 建立防禦實例
子網路和安全群組已經就位，接下來建立防禦虛擬伺服器實例。

1. 在左側窗格的**子網路**下，選取 **vpc-secure-bastion-subnet**。
2. 按一下**連接的實例數**並在自己的 VPC 下佈建名稱為 **vpc-secure-vsi** 的**新實例**。選取 Ubuntu Linux 作為映像檔，**c-2x4**（2 個 vCPU 和 4 GB RAM）作為設定檔。
3. 選取**位置**，確保稍後使用相同的位置。
4. 若要建立新的 **SSH 金鑰**，請按一下**建立新的項目金鑰**
   * 輸入 **vpc-ssh-key** 作為索引鍵名稱。
   * 按原樣保留**地區**。
   * 將現有本端 SSH 金鑰的內容複製並貼上到**公開金鑰**下。  
   * 按一下**新增 SSH 金鑰**。
5. 在**網路介面**下，按一下安全群組旁的**編輯**圖示。 
   * 確保 **vpc-secure-subnet** 選為子網路。
   * 取消勾選預設安全群組，並標示 **vpc-secure-bastion-sg**。
   * 按一下**儲存**。
6. 按一下**建立虛擬伺服器實例**。
7. 打開該實例的電源後，按一下 **vpc-secure-bastion-vsi** 並**保留**一個浮動 IP。

### 測試您的防禦

防禦的浮動 IP 位址處於作用中狀態後，嘗試使用 **ssh** 連接至該位址：

   ```sh
   ssh -i ~/.ssh/<PRIVATE_KEY> root@<BASTION_FLOATING_IP_ADDRESS>
   ```
   {:pre}

## 使用維護存取規則配置安全群組
{: #maintenance-security-group}

透過對運作中防禦的存取權，繼續建立用於安裝和更新軟體之類的維護作業的安全群組。

1. 導覽至**安全群組**並使用下面的出埠規則佈建名稱為 **vpc-secure-maintenance-sg** 的新安全群組

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
         <td>TCP</td>
         <td>從：<strong>53</strong> 到 <strong>53</strong></td>
      </tr>
      <tr>
         <td>任何 - 0.0.0.0/0 </td>
         <td>UDP</td>
         <td>從：<strong>53</strong> 到 <strong>53</strong></td>
      </tr>
   </tbody>
   </table>

   DNS 伺服器要求在埠 53 上進行處理。DNS 使用 TCP 進行區域傳輸，使用 UDP 進行名稱查詢，查詢可以是定期（主要）或反向的。HTTP 要求位於埠 80 和 443 上。{:tip }

2. 接下來，新增此**入埠**規則，以容許從防禦主機進行 SSH 存取。

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
	         <td>類型：<strong>安全群組</strong> - 名稱：<strong>vpc-secure-bastion-sg</strong></td>
	         <td>TCP</td>
	         <td>從：<strong>22</strong> 到 <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>

3. 建立安全群組。
4. 導覽至 **VPC 的所有安全群組**，然後選取 **vpc-secure-sg**。
5. 最後，編輯安全群組，並新增下列**出埠**規則。

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
	         <td>類型：<strong>安全群組</strong> - 名稱：<strong>vpc-secure-maintenance-sg</strong></td>
	         <td>TCP</td>
	         <td>從：<strong>22</strong> 到 <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>


## 使用防禦主機存取 VPC 中的其他實例 
{: #bastion-host-access-instances}

 在本區段中，您將使用虛擬伺服器實例和安全群組建立專用子網路。依預設，VPC 中建立的任何子網路都是專用的。

如果您想要連接的 VPC 中已經擁有虛擬伺服器實例，可以跳過下面三個區段，然後從[將虛擬伺服器實例新增到維護安全群組](#add-vsi-to-maintenance)開始。

### 建立子網路
{: #create-private-subnet}

建立新的項目子網路：

1. 在左窗格的**網路**下，按一下**子網路**，然後按一下**新建子網路**。  
   * 輸入 **vpc-secure-private-subnet** 作為名稱，然後選取您建立的 VPC。  
   * 選取位置。  
   * 以 CIDR 表示法輸入子網路的 IP 範圍，亦即 **10.xxx.1.0/24**。保留**位址字首**原樣，並將**位址數目**選取為 256。
1. 為您的子網路存取控制清單 (ACL) 選取 **VPC 預設值**。您可以之後再配置入埠和出埠規則。
1. 將**公用閘道**切換成**已連接**。 
1. 按一下**建立子網路**來佈建它。

### 建立安全群組

建立新的項目安全群組：  
1. 按一下「網路」下的**安全群組**，然後按一下**新建安全群組**。  
2. 輸入 **vpc-secure-private-sg** 作為名稱，然後選取先前建立的 VPC。   
3. 按一下**建立安全群組**。  

### 建立虛擬伺服器實例

若要在新建立的子網路中建立虛擬伺服器實例，請執行下列動作：

1. 按一下**子網路**下的專用子網路。
2. 按一下**連接的實例**，然後按一下**新建實例**。
3. 輸入唯一名稱 **vpc-secure-private-vsi**，選取先前建立的 VPC，然後選取與之前相同的**位置**。
4. 選取 **Ubuntu Linux** 映像檔、按一下**所有設定檔**，然後在**運算**底下選擇 **c-2x4**：2 個 vCPU 和 4 GB RAM
5. 對於 **SSH 金鑰**，選取您先前為防禦建立的 SSH 金鑰。
6. 在**網路介面**下，按一下安全群組旁的**編輯**圖示。   
   * 選取 **vpc-secure-private-subnet** 作為子網路。  
   * 取消勾選預設安全群組，並啟動 **vpc-secure-private-sg**。  
   * 按一下**儲存**。  
7. 按一下**建立虛擬伺服器實例**。  


### 將虛擬伺服器新增到維護安全群組
{: #add-vsi-to-maintenance}

對於伺服器上的管理工作，您必須將特定虛擬伺服器與維護安全群組關聯。接下來，您將啟用維護，登入到專用伺服器，更新套裝軟體資訊，然後再次解除關聯安全群組。

接下來為伺服器啟用維護安全群組。

1. 導覽至**安全群組**，然後選取 **vpc-secure-maintenance-sg** 安全群組。  
2. 按一下**連接的介面**，然後按一下**編輯介面**。  
3. 展開虛擬伺服器實例，並啟動**介面**直欄中**主要**旁的選項。
4. 按一下**儲存**以便套用變更。

### 連接至實例

若要使用實例的**專用 IP** 透過 SSH 登錄到該實例，您將使用防禦主機作為**跳躍主機**。

1. 在**虛擬伺服器實例**下獲得虛擬伺服器實例的專用 IP 位址。
2. 使用具有 `-J` 的 ssh 指令，透過先前使用的防禦**浮動 IP** 位址和**網路介面**下顯示的伺服器**專用 IP** 位址來登入到伺服器。

   ```sh
   ssh -J root@<BASTION_FLOATING_IP_ADDRESS> root@<PRIVATE_IP_ADDRESS>
   ```
   {:pre}
   
   在 OpenSSH V7.3+ 中支援 `-J` 旗標。在較舊版本中，`-J` 無法使用。在此情況下，最安全、最直接的模式是使用 ssh 的 stdio 轉遞 (`-W`) 模式使連線「彈跳」透過防禦主機。例如，`ssh -o ProxyCommand="ssh -W %h:%p root@<BASTION_FLOATING_IP_ADDRESS" root@<PRIVATE_IP_ADDRESS>`
   {:tip }

### 安裝軟體及執行維護作業

連接後，您可以在專用子網路的虛擬伺服器上安裝軟體，或者執行維護作業。

1. 請先更新套裝軟體資訊：
   ```sh
        apt-get update
        ```
   {:pre}
2. 安裝所需的軟體，例如 Nginx、MySQL 或 IBM Db2。

完成後，使用 `exit` 指令中斷與伺服器的連線。 

若要容許網際網路使用者的 HTTP/HTTPS 要求，請將**浮動 IP** 指派給專用子網路中的 VSI，並在專用 VSI 的安全群組中透過入埠規則開啟所需埠（80 - HTTP 和 443 - HTTPS）。
{:tip}

### 停用維護安全群組

完成軟體安裝或維護後，您應從維護安全群組移除虛擬伺服器，以使其保持隔離。

1. 導覽至**安全群組**，然後選取 **vpc-secure-maintenance-sg** 安全群組。  
2. 按一下**連接的介面**，然後按一下**編輯介面**。  
3. 展開虛擬伺服器實例，並取消勾選**介面**直欄中**主要**旁的選項。
4. 按一下**儲存**以便套用變更。

## 移除資源
{: #removeresources}

1. 切換到**虛擬伺服器實例**並**刪除**實例。實例將會刪除，它們的狀態會維持在**正在刪除**一陣子。請務必偶爾重新整理瀏覽器。
2. 刪除 VSI 之後，切換到**子網路**並刪除子網路。
4. 刪除子網路後，切換到**虛擬專用雲端**標籤，並刪除您的 VPC。

使用主控台時，您可能需要重新整理瀏覽器，才能在刪除資源之後看到更新的狀態資訊。
{:tip}

## 相關內容
{: #related}

* [Virtual Private Cloud 中的專用和公用子網路](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend)
