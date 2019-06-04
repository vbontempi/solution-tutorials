---
copyright:
  years: 2019
lastupdated: "2019-05-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}
{:important: .important}

# 使用 VPC/VPN 閘道對雲端資源進行安全的專用內部部署存取
{: #vpc-site2site-vpn}

IBM 將接受限量客戶參與 2019 年 4 月初開辦的 VPC「搶先體驗」計劃，並在接下來幾個月開放擴大使用。如果您的組織希望體驗「IBM 虛擬專用雲端」，請填寫此[申請表](https://{DomainName}/vpc){: new_window}，IBM 業務代表將聯絡您進行下一步。
{: important}

IBM 提供了多種方法來使用 IBM Cloud 中的資源安全地延伸內部部署電腦網路。這樣，您可以靈活機動地在需要伺服器時佈建伺服器並在不再需要時移除伺服器。此外，您可以輕鬆、安全地將內部部署功能連接至 {{site.data.keyword.cloud_notm}} 服務。

本指導教學將全程指導將內部部署虛擬專用網路 (VPN) 閘道連接至 VPC（VPC/VPN 閘道）中建立的雲端 VPN。首先，您將建立新的 {{site.data.keyword.vpc_full}} (VPC) 以及相關聯的資源，例如子網路、網路存取控制清單 (ACL)、安全群組以及虛擬伺服器實例 (VSI)。VPC/VPN 閘道將對內部部署 VPN 閘道建立 [IPsec](https://en.wikipedia.org/wiki/IPsec) 站台對站台鏈路。IPsec 和 [Internet Key Exchange](https://en.wikipedia.org/wiki/Internet_Key_Exchange) (IKE) 通訊協定是成熟的安全通訊開放標準。

為了進一步示範安全和專用存取，您將在 VSI 上部署微服務，以存取 {{site.data.keyword.cos_short}} (COS)，後者代表事業線應用程式。COS 服務具有直接端點，在所有存取權位於 {{site.data.keyword.cloud_notm}} 的同一地區中時，可將此端點用於專用的無成本 Ingress/Egress。內部部署電腦將存取 COS 微服務。所有資料流量將流經 VPN，並因此將以專用方式流經 {{site.data.keyword.cloud_notm}}。

對於站台對站台閘道，有許多熱門的內部部署 VPN 解決方案可用。本指導教學利用 [strongSwan](https://www.strongswan.org/) VPN 閘道來連接到 VPC/VPN 閘道。若要模擬內部部署資料中心，請在 {{site.data.keyword.cloud_notm}} 的 VSI 上安裝 strongSwan 閘道。

{:shortdesc}
簡言之，使用 VPC，您可以

- 將內部部署系統連接至 {{site.data.keyword.cloud_notm}} 中執行的服務和工作負載，
- 確保 COS 的專用和低成本連線功能，
- 將以雲端為基礎的系統連接至以內部部署方式執行的服務和工作負載。

## 目標
{: #objectives}

* 從內部部署資料中心或者（虛擬）專用雲端存取虛擬專用雲端環境。
* 使用專用服務端點安全地呼叫到雲端服務。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。雖然本指導教學中從微服務存取 COS 沒有網路計費，但是存取 VPC 會發生標準網路計費。

## 架構
{: #architecture}

下圖顯示包含應用程式伺服器的虛擬專用雲端。應用程式伺服器會管理與 {{site.data.keyword.cos_short}} 服務互動的微服務。（模擬的）內部部署網路和虛擬雲端環境透過 VPN 閘道進行連接。

![架構](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. 使用提供的 Script 來設定基礎架構（VPC、子網路、具有規則的安全群組、網路 ACL 和 VSI）。
2. 微服務透過直接端點與 {{site.data.keyword.cos_short}} 互動。
3. 佈建 VPC/VPN 閘道，以將虛擬專用雲端環境公開給內部部署網路。
4. Strongswan 開放程式碼 IPsec 閘道軟體以內部部署方式用於建立與雲端環境的 VPN 連線。

## 開始之前
{: #prereqs}

- [遵循下列步驟](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)以安裝所有必要的指令行 (CLI) 工具。您需要選用的 CLI 基礎架構外掛程式。
- 透過指令行登入到 {{site.data.keyword.cloud_notm}}。有關詳細資料，請參閱 [CLI 開始使用](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli)。
- 檢查使用者許可權。您的使用者帳戶務必要有足夠的許可權以便建立及管理 VPC 資源。如需必要許可權的清單，請參閱[授與 VPC 使用者所需的許可權](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)。
- 您需要 SSH 金鑰才能連接至虛擬伺服器。如果您沒有 SSH 金鑰，請參閱[建立金鑰的指示](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)。
- 安裝 [**jq**](https://stedolan.github.io/jq/download/)。所提供的 Script 將其用於處理 JSON 輸出。

## 在虛擬專用雲端中部署虛擬應用程式伺服器
接下來，您將下載 Script 以設定微服務的基準線 VPC 環境和程式碼，以與 {{site.data.keyword.cos_short}} 互動。因此，您將佈建 {{site.data.keyword.cos_short}} 服務並設定基準線。

### 取得程式碼
{: #setup}
本指導教學在建立 VPN 閘道之前使用 Script 來部署基礎架構資源的基準線。微服務的這些 Script 和程式碼在 GitHub 儲存庫中提供。

1. 取得應用程式的程式碼：
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. 藉由切換到 **vpc-tutorials**，然後切換到 **vpc-site2site-vpn**，以移至本指導教學的 Script：
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### 建立服務
在本區段中，您將登入 CLI 上的 {{site.data.keyword.cloud_notm}}，並建立 {{site.data.keyword.cos_short}} 的實例。

1. 驗證您是否已執行登錄的必備步驟：
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. 使用**標準**或**精簡**方案建立 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) 的實例。
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   請注意，每個帳戶只可以建立一個精簡實例。如果您已經擁有 {{site.data.keyword.cos_short}} 實例，可以加以重複使用。
   {: tip}

3. 使用角色**讀者**建立服務金鑰：
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. 以 JSON 格式取得服務金鑰詳細資料，並將其儲存在子目錄 **vpc-app-cos** 中的新檔案 **credentials.json** 中。應用程式將在稍後使用該檔案。
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### 建立 Virtual Private Cloud 基準線資源
{: #create-vpc}
本指導教學提供 Script 以建立本指導教學所需的基準線資源，即起始環境。該 Script 可以在現有 VPC 中產生該環境，或者建立新的 VPC。

接下來，藉由配置設定 Script，然後執行該 Script，從而建立這些資源。該 Script 包含防禦主機的設定，如[使用防禦主機安全地存取遠端實例](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)中所述。

1. 將配置範例檔複製到要使用的檔案中：

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. 編輯檔案 **config.sh**，並根據環境調整設定。您需要將 **SSHKEYNAME** 的值變更為 SSH 金鑰的名稱或多個名稱的逗號分隔清單（請參閱「準備工作」）。修改不同的**區域**設定以符合您的雲端地區。所有其他變數可以按原樣保留。
3. 若要在新的 VPC 中建立資源，請如下所示執行 Script：

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   若要重複使用現有 VPC，請以此方式將其名稱傳遞給 Script。將 **YOUR_EXISTING_VPC** 取代為實際的 VPC 名稱。
   ```sh
   REUSE_VPC=YOUR_EXISTING_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. 這樣將導致建立下列資源，包括與防禦相關的資源：
   - 1 個 VPC （選用）
   - 1 個公用閘道
   - VPC 中的 3 個子網路
   - 具有 Ingress 和 Egress 規則的 4 個安全群組
   - 3 個 VSI：vpns2s-onprem-vsi（浮動 ip 為 ONPREM_IP），vpns2s-cloud-vsi（浮動 ip 為 VSI_CLOUD_IP），以及 vpns2s-bastion（浮動 ip 為 BASTION_IP_ADDRESS）

   記下 **BASTION_IP_ADDRESS**、**VSI_CLOUD_IP**、**ONPREM_IP**、**CLOUD_CIDR** 和 **ONPREM_CIDR** 的回覆值以供稍後使用。輸出還會儲存在檔案
**network_config.sh** 中。該檔案可以用於自動設定。

### 建立虛擬專用網路閘道和連線
接下來，您將新增 VPN 閘道以及與應用程式 VSI 的子網路的關聯連線。

1. 導覽至 [VPC 概觀](https://{DomainName}/vpc/overview)頁面，然後按一下導覽標籤中的 **VPN**，以及對話框中的**新建 VPN 閘道**。在表單**新建 VPC 的 VPN 閘道**中，輸入 **vpns2s-gateway** 作為名稱。確保選取正確的 VPC、資源群組以及子網路 **vpns2s-cloud-subnet**。
2. 使**新建 VPC 的 VPN 連線**保持啟動狀態。輸入 **vpns2s-gateway-conn** 作為名稱。
3. 對於**同層級閘道位址**，使用 **vpns2s-onprem-vsi** 的浮動 IP 位址 (ONPREM_IP)。鍵入 **20_PRESHARED_KEY_KEEP_SECRET_19** 作為**預先共用金鑰**。
4. 對於**本端子網路**，使用為 **CLOUD_CIDR** 提供的資訊，對於**同層級子網路**，使用為 **ONPREM_CIDR** 提供的資訊。
5. 按原樣保留**無回應對等節點偵測**中的設定。按一下**建立 VPN 閘道**以建立閘道和關聯的連線。
6. 等待 VPN 閘道變為可用（您可能需要重新整理畫面）。
7. 記下作為 **GW_CLOUD_IP** 指派的**閘道 IP** 位址。 

### 建立內部部署虛擬專用網路閘道
接下來，您將在模擬的內部部署環境中的其他網站上建立 VPN 閘道。您將使用以開放程式碼為基礎的 IPSec 軟體 [strongSwan](https://strongswan.org/)。

1. 使用 ssh 連接至「內部部署」VSI **vpns2s-onprem-vsi**。執行下列指令，並將 **ONPREM_IP** 取代為先前傳回的 IP 位址。

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   根據您的環境，您可能需要使用 `ssh -i <path to your private key file> root@ONPREMP_IP`。
   {:tip}

2. 接下來，在機器 **vpns2s-onprem-vsi** 上，執行下列指令以更新套裝軟體管理程式並安裝 strongSwan 軟體。

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. 藉由在檔案 **/etc/sysctl.conf** 結尾新增三行來配置該檔案。複製下列內容並執行該文件：

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. 接下來，編輯檔案 **/etc/ipsec.secrets**。新增下行以配置來源和目的地 IP 位址以及先前配置的預先共用金鑰。將 **ONPREM_IP** 取代為 vpns2s-onprem-vsi 的浮動 IP 的已知值。將 **GW_CLOUD_IP** 取代為 VPC VPN 閘道的已知 IP 位址。

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. 您需要配置的最後一個檔案是 **/etc/ipsec.conf**。將下列程式碼區塊新增到該檔案結尾。將 **ONPREM_IP**、**ONPREM_CIDR**、**GW_CLOUD_IP** 和 **CLOUD_CIDR** 取代為相應的已知值。

   ```sh
   # basic configuration
   config setup
      charondebug="all"
      uniqueids=yes
      strictcrlpolicy=no
   
   # connection to vpc/vpn datacenter 
   # left=onprem / right=vpc
   conn tutorial-site2site-onprem-to-cloud
      authby=secret
      left=%defaultroute
      leftid=ONPREM_IP
      leftsubnet=ONPREM_CIDR
      right=GW_CLOUD_IP
      rightsubnet=CLOUD_CIDR
      ike=aes256-sha2_256-modp1024!
      esp=aes256-sha2_256!
      keyingtries=0
      ikelifetime=1h
      lifetime=8h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start
   ```
   {:pre}

6. 重新啟動 VPN 閘道，然後藉由執行 ipsec restart 來檢查其狀態

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   這時它應該回報連線已建立。保持此機器的終端機和 ssh 連線為開啟狀態。

## 測試連線功能
您可以藉由使用 SSH 或者部署與 {{site.data.keyword.cos_short}} 互動的微服務，來測試站台對站台的 VPN 連線。

### 使用 SSH 測試
若要測試 VPN 連線是否已順利建立，請使用模擬的內部部署環境作為 Proxy，以登入到以雲端為基礎的應用程式伺服器。 

1. 在新的終端機中，取代值之後，執行下列指令。該指令使用 strongSwan 主機作為跳躍主機，透過 VPN 連接至應用程式伺服器的專用 IP 位址。

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. 一旦順利連線，關閉 SSH 連線。

3. 在「內部部署」VSI 終端機中，停止 VPN 閘道：
   ```sh
   ipsec stop
   ```
   {:pre}
4. 在步驟 1) 的指令視窗中，嘗試再次建立連線：

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
   此指令應該不會成功，因為 VPN 連線未處於作用中狀態，因此模擬的內部部署和雲端環境之間沒有直接鏈路。

   請注意，根據部署詳細資料，此連線實際上仍會成功。原因是在各區域之間支援 VPC 內部連線功能。如果您會將模擬的內部部署 VSI 部署到其他 VPC 或 [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers)，將需要 VPN 才能成功存取。
   {:tip}
   
5. 在「內部部署」VSI 終端機中，再次啟動 VPN 閘道：
   ```sh
   ipsec start
   ```
   {:pre}
 

### 使用微服務測試
您可以從內部部署 VSI 藉由存取雲端 VSI 上的微服務來測試運作中的 VPN 連線。

1. 將微服務應用程式的程式碼從本端機器複製到雲端 VSI。此指令使用防禦主機作為雲端 VSI 的跳躍主機。相應地取代 **BASTION_IP_ADDRESS** 和 **VSI_CLOUD_IP**。
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. 再次使用防禦主機作為跳躍主機連接至雲端 VSI。
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. 在雲端 VSI 上，切換到程式碼目錄：
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. 安裝 Python 和 Python 套裝軟體管理程式 PIP。
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. 使用 **pip** 安裝必需的 Python 套裝軟體。
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. 啟動應用程式：
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. 在「內部部署」VSI 終端機中，存取該服務。相應地取代 VSI_CLOUD_IP。
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
   指令應該傳回 JSON 物件。

## 移除資源
{: #remove-resources}

1. 在 VPC 管理主控台，按一下 **VPN**。在 VPN 閘道上的動作功能表中，選取**刪除**以移除閘道。
2. 接下來，按一下導覽中的**浮動 IP**，然後按一下您的 VSI 的 IP 位址。在動作功能表中，選取**發佈**。確認您想要釋放 IP 位址。
3. 接下來，切換至**虛擬伺服器實例**，然後**刪除**您的實例。實例將會刪除，它們的狀態會維持在**正在刪除**一陣子。請務必偶爾重新整理瀏覽器。
4. VSI 消失之後，切換至**子網路**。如果子網路有連接的公用閘道，則請按一下子網路名稱。在子網路詳細資料中，分離公用閘道。可以從概觀頁面刪除沒有公用閘道的子網路。刪除您的子網路。
5. 刪除子網路後，切換到 **VPC**，並刪除您的 VPC。

使用主控台時，您可能需要重新整理瀏覽器，才能在刪除資源之後看到更新的狀態資訊。
{:tip}

## 擴展指導教學 
{: #expand-tutorial}

想要新增或延伸此指導教學？以下是一些構想：

- 新增[負載平衡器](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc)以跨多個實例分散入埠微服務資料流量。
- [在公用伺服器上部署應用程式，在專用主機上部署您的資料和服務](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend)。


## 相關內容
{: #related}

- [VPC 名詞解釋](/docs/vpc?topic=vpc-vpc-glossary)
- [VPC 的 IBM Cloud CLI 外掛程式參考](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [使用 REST API 的 VPC](/docs/infrastructure/vpc/example-code.html)
- 解決方案指導教學：[使用防禦主機安全地存取遠端實例](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
