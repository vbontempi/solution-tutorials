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

# 跨多個位置和區域部署隔離工作負載
{: #vpc-multi-region}

IBM 將接受限量客戶參與 2019 年 4 月初開辦的 VPC「搶先體驗」計劃，並在接下來幾個月開放擴大使用。如果您的組織希望體驗「IBM 虛擬專用雲端」，請填寫此[申請表](https://{DomainName}/vpc){: new_window}，IBM 業務代表將聯絡您進行下一步。
{: important}

本指導教學將引導您透過在不同的 IBM Cloud 地區佈建 VPC 來設定隔離的工作負載。具有子網路和虛擬伺服器實例 (VSI) 的地區。這些 VSI 在一個地區的多個區域中建立，透過配置具有後端儲存區、前端接聽器和正確性能檢查的負載平衡器，增加地區內以及廣域範圍的備援能力。

對於廣域負載平衡器，您將從型錄佈建 IBM Cloud Internet Services (CIS) 服務，對於管理所有送入 HTTPS 要求的 SSL 憑證，將建立 {{site.data.keyword.cloudcerts_long_notm}} 型錄服務，並將匯入該憑證以及私密金鑰。

{:shortdesc}

## 目標
{: #objectives}

* 透過虛擬專用雲端可用的基礎架構物件瞭解工作負載隔離。
* 在一個地區的不同區域之間使用負載平衡器在虛擬伺服器之間配送資料流量。
* 在地區之間使用廣域負載平衡器，以提高備援能力並縮短延遲。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.loadbalancer_full}}](https://{DomainName}/vpc/provision/loadBalancer)
- IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
- [{{site.data.keyword.cloudcerts_long_notm}}](https://{DomainName}/catalog/services/cloudcerts)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/estimator/review)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

  ![架構](images/solution41-vpc-multi-region/Architecture.png)

1. 管理 (DevOps) 在地區 1 中的 VPC 內兩個不同區域下的子網路中佈建 VSI，並在地區 2 中建立的 VPC 內重複相同的操作。
2. 管理使用地區 1 中不同區域內子網路的後端伺服器儲存區和一個前端接聽器來建立負載平衡器。在地區 2 中重複相同的操作。
3. 管理利用關聯的自訂網域佈建 Cloud Internet Services 服務，並建立一個指向在兩個不同 VPC 中建立的負載平衡器的廣域負載平衡器。
4. 管理透過將網域 SSL 憑證新增到 Certificate Manager 服務來啟用 HTTPS 加密。
5. 網際網路使用者發出 HTTP/HTTPS 要求，而廣域負載平衡器處理該要求。
6. 此要求將遞送至廣域和本端的負載平衡器。然後，由可用的伺服器實例履行此要求。

## 開始之前
{: #prereqs}

- 檢查使用者許可權。您的使用者帳戶務必要有足夠的許可權以便建立及管理 VPC 資源。如需必要許可權的清單，請參閱[授與 VPC 使用者所需的許可權](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources)。
- 您需要 SSH 金鑰才能連接至虛擬伺服器。如果您沒有 SSH 金鑰，請參閱[建立金鑰的指示](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)。
- Cloud Internet Services 需要您擁有自訂網域，因此您可以設定此網域的 DNS，指向 Cloud Internet Services 名稱伺服器。如果您未擁有網域，可以從例如 [godaddy.com](http://godaddy.com/) 的註冊商購買一個。

## 建立 VPC、子網路及 VSI
{: #create-infrastructure}

在本區段中，您將在地區 1 中建立您自己的 VPC，並在地區 1 的兩個不同區域中建立子網路，然後佈建 VSI。

若要在地區 1 建立您自己的 {{site.data.keyword.vpc_short}}：

1. 導覽至 [VPC 概觀](https://{DomainName}/vpc/overview)頁面，然後按一下**建立 VPC**。
2. 在**新建 Virtual Private Cloud** 區段下：
   * 輸入 **vpc-region1** 作為 VPC 的名稱。
   * 選取**資源群組**。
   * 選擇性地新增**標籤**以組織您的資源。
3. 選取**建立新的預設值（全部容許）**作為您的 VPC 預設存取控制清單 (ACL)。
4. 取消勾選 SSH，從**預設安全群組**執行 ping 操作，並將**經典存取**保持為不勾選狀態。
5. 在**新建 VPN 的子網路**下：
   * 輸入 **vpc-region1-zone1-subnet** 作為唯一名稱。
   * 選取位置（例如，達拉斯），不妨稱為**地區 1**，在地區 1（例如，達拉斯 1）中選取一個區域，不妨稱為**區域 1**。
   * 以 CIDR 表示法輸入子網路的 IP 範圍，亦即 **10.xxx.0.0/24**。保留**位址字首**原樣，並將**位址數目**選取為 256。
6. 為您的子網路存取控制清單 (ACL) 選取**使用 VPC 預設值**。您可以之後再配置入埠和出埠規則。
7. 假設子網路中的所有虛擬伺服器實例會連接浮動 IP，則不需要啟用子網路的公用閘道。虛擬伺服器實例會透過浮動 IP 而具有網際網路連線功能。
8. 按一下**建立 Virtual Private Cloud** 來佈建實例。

若要確認子網路的建立，請按一下左側窗格上的**子網路**，並等待狀態變更為**可用**。您可以在**子網路**下建立新的子網路。

### 在區域 2 中建立子網路

1. 按一下**新建子網路**，輸入 **vpc-region1-zone2-subnet** 作為子網路的唯一名稱，並選取 **vpc-region1** 作為 VPC。
2. 選取我們在上面稱為地區 1 的位置（例如，達拉斯），並選取地區 1 中不同的區域（例如，達拉斯 2），不妨將所選區域稱為**區域 2**。
3. 以 CIDR 表示法輸入子網路的 IP 範圍，即 **10.xxx.64.0/24**。保留**位址字首**原樣，並將**位址數目**選取為 256。
4. 為您的子網路存取控制清單 (ACL) 選取**使用 VPC 預設值**。

### 佈建 VSI
在子網路狀態變更為**可用**後，

1. 按一下 **vpc-region1-zone1-subnet**，再按一下**連接的實例**，然後按一下**新建實例**。
2. 輸入唯一名稱，並選取 **vpc-region1-zone1-vsi**。然後，選取先前建立的 VPC，並選取**位置**以及**區域**，如前所述。
3. 選擇任何 **Ubuntu Linux** 映像檔，按一下**所有設定檔**，在**運算**下，選擇具有 2 個 vCPU 和 4 GB RAM 的 **c-2x4**。
4. 對於 **SSH 金鑰**，選取您起始建立的 SSH 金鑰。
5. 在**網路介面**下，按一下安全群組旁的**編輯**圖示。
   * 檢查是否選取了 **vpc-region1-zone1-subnet** 作為子網路。如果沒有，請進行選取。
   * 按一下**儲存**。
   * 按一下**建立虛擬伺服器實例**。
6.  等到 VSI 的狀態變更為**已開啟電源**。然後，選取 VSI **vpc-region1-zone1-vsi**，捲動到**網路介面**並按一下**浮動 IP** 下的**預留**以將公用 IP 位址關聯到您的 VSI。將相關聯的 IP 位址儲存至剪貼簿，以便未來參考。
7. **重複**步驟 1-6 以在**地區 1** 的**區域 2** 中佈建 VSI。

導覽至左側窗格的**網路**下的 **VPC** 和**子網路**，並**重複**上述步驟以遵循上述相同命名慣例來使用**地區 2** 中的子網路和 VSI 佈建新的 VPC。

## 安裝和配置 VSI 上的 Web 伺服器
{: #install-configure-web-server-vsis}

遵循[使用防禦主機安全存取遠端實例](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)中提及的步驟，使用充當 `jump` 伺服器和維護安全組的防禦主機來安全地維護伺服器。
{:tip}

透過 SSH 順利登錄到地區 1 內區域 1 的子網路中佈建的伺服器後：

1. 在提示字元處，執行下面的指令以將 Nginx 作為 Web 伺服器進行安裝：
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   {:codeblock}
2. 使用下面的指令檢查 Nginx 服務的狀態：
   ```
   sudo systemctl status nginx
   ```
   {:pre}
   輸出應顯示 Nginx 服務處於**作用中**狀態並且正在執行。
3. 您將需要開啟 **HTTP (80)** 和 **HTTPS (443)** 埠以接收資料流量（要求）。您可以透過 [UFW](https://help.ubuntu.com/community/UFW) - `sudo ufw enable` 調整防火牆並啟用包括兩個埠的規則的 "Nginx Full" 設定檔來完成該動作：
   ```
   sudo ufw allow 'Nginx Full'
   ```
   {:pre}
4. 若要驗證 Nginx 是否按預期運作，請在所選的瀏覽器中開啟 `http://FLOATING_IP`，此時您將看到預設的 Nginx 歡迎使用頁面。
5. 若要使用地區和區域詳細資料更新 html 頁面，請執行下面的指令：
   ```
   nano /var/www/html/index.nginx-debian.html
   ```
   {:pre}
   將地區和區域標語_在**地區 1 的區域 1** 中執行的伺服器_ 附加到顯示`歡迎使用 nginx！`的 `h1` 標籤，並儲存更改。
6. 重新啟動 nginx 伺服器以反映變更
   ```
   sudo systemctl restart nginx
   ```
   {:pre}

**重複**步驟 1-6 以安裝和配置所有區域的子網路中 VSI 上的 Web 伺服器，並務必使用相應的區域資訊更新 html。


## 使用負載平衡器在區域之間配送資料流量
{: #distribute-traffic-with-load-balancers}

在此區段中，您將建立兩個負載平衡器。每個地區一個負載平衡器，以在不同區域內相應子網路下的多個伺服器實例之間配送資料流量。

### 配置負載平衡器

1. 導覽至**負載平衡器**，並按一下**新建負載平衡器**。
2. 指定 **vpc-lb-region1** 作為唯一名稱，並選取 **vpc-region1** 作為虛擬專用雲端，後跟建立 VPC 所在的資源群組，輸入：**公用**，並輸入**地區 1** 作為地區。
3. 為**地區 1** 的**區域 1** 和**區域 2** 選取專用 IP。
4. 建立 VSI 的新後端儲存區，以充當同等同層級來共用遞送到該儲存區的資料流量。使用下面的值設定參數，並按一下**建立**。
	- **名稱**：region1-pool
	- **通訊協定**：HTTP
	- **方法**：循環式
	- **階段作業黏著度**：無
	- **性能檢查路徑**：/
	- **性能通訊協定**：HTTP
	- **時間間隔（秒）**：15
	- **超時（秒）**：2
	- **最大重試次數**：2
5. 按一下**連接**以將伺服器實例新增到 region1-pool
   - 選取 **vpc-region1-zone1-subnet** 的專用 IP，選取您建立的實例，並將埠設定為 80。
   - 按一下**新增**，這次選取 **vpc-region1-zone2-subnet** 的專用 IP，選取實例並將埠設定為 80。
   - 按一下**連接**以完成後端儲存區的建立。
6. 按一下**建立新的項目接聽器**以建立新的前端接聽器；接聽器是檢查連線要求的處理程序。
   - **通訊協定**：HTTP
   - **埠**：80
   - **後端儲存區**：region1-pool
   - **最大連接數**：留空並按一下**建立**。
7. 按一下**建立負載平衡器**以佈建負載平衡器。

### 測試負載平衡器

1. 等待負載平衡器狀態變更為**作用中**。
2. 在 Web 瀏覽器中開啟**位址**。
3. 重新整理頁面若干次，並注意每次重新整理後，負載平衡器會命中不同的伺服器。
4. **儲存**位址以供將來參考。

如果您進行觀察，將發現要求未加密，並僅支援 HTTP。在下一區段中，您將配置 SSL 憑證和啟用 HTTPS。

在**地區 2** 中**重複**上述步驟 1-7。

## 使用 HTTPS 保護 VPC 中的資料流量
{: #secure_https}

在新增 HTTPS 接聽器之前，您需要產生 SSL 憑證，驗證自訂網域的真實性，自訂網域是存放憑證並將其對映到基礎架構服務的地方。

### 佈建 CIS 服務，並配置自訂網域。

在本區段中，您將建立 IBM Cloud Internet Services (CIS) 服務，透過將自訂網域指向 CIS 名稱伺服器進行配置，並稍後配置廣域負載平衡器。

1. 導覽至 {{site.data.keyword.Bluemix_notm}} 型錄中的 [Internet Services](https://{DomainName}/catalog/services/internet-services)。
2. 設定服務名稱，然後按一下**建立**以建立服務的實例。您可以針對本指導教學使用任何定價方案。
3. 佈建服務實例時，藉由按一下**讓我們開始吧**設定您的網域名稱，然後按一下**新增網域**。
4. 按**下一步**。指派名稱伺服器時，請配置登記員或網域名稱提供者以便使用列出的名稱伺服器。
5. 配置登記員或 DNS 提供者之後，可能需要最多 24 小時，變更才會生效。

   當「概觀」頁面上的網域狀態從 *Pending* 變更為 *Active* 時，您可以使用 `dig <YOUR_DOMAIN_NAME> ns` 指令驗證新的名稱伺服器已生效。
   {:tip}

您應獲得您計劃用於廣域負載平衡器的網域和子網域的 SSL 憑證。假設有類似 mydomain.com 的網域，廣域負載平衡器可以在 `lb.mydomain.com` 上管理。需要為 lb.mydomain.com 簽發該憑證。

您可以從 [Let's Encrypt](https://letsencrypt.org/) 取得免費的 SSL 憑證。在過程中，您可能需要在 Cloud Internet Services 的 DNS 介面，配置 TXT 類型的 DNS 記錄，以證明您是網域的擁有者。
{:tip}

取得網域的 SSL 憑證及私密金鑰之後，請務必將它們轉換成 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 格式。

1. 將憑證轉換為 PEM 格式：
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
2. 將私密金鑰轉換為 PEM 格式：
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### 匯入憑證，並授權負載平衡器服務

您可以透過 IBM Certificate Manager 管理 SSL 憑證。

1. 在支援的位置建立 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 實例。
2. 在服務儀表板中，使用**匯入憑證**：
   * 將**名稱**設定為自訂子網域和網域，例如 *lb.mydomain.com*。
   * 瀏覽 PEM 格式的**憑證檔案**。
   * 瀏覽 PEM 格式的**私密金鑰檔案**。
   * **匯入**。
3. 建立授權，以向 Load Balancer 服務實例授予對包含 SSL 憑證的 Certificate Manager 實例的存取權。您可以透過[身分和存取權授權](https://{DomainName}/iam#/authorizations)來管理此類授權。
  - 按一下**建立**並選擇 **Infrastructure Service** 作為來源服務
  - **Load Balancer for VPC** 作為資源類型
  - **Certificate Manager** 作為目標服務
  - 指派**撰寫者**服務存取角色。
  - 若要建立負載平衡器，您必須對來源資源實例授與「全部」資源實例權限。目標服務實例可能是**全部實例**，或者也可能是具體的 Certificate Manager 資源實例。

### 建立 HTTPS 接聽器

現在，導覽至[負載平衡器](https://{DomainName}/vpc/network/loadBalancers)

1. 選取 **vpc-lb-region1**
2. 在**前端接聽器**下，按一下**新建接聽器**

   -  **通訊協定**：HTTPS
   -  **埠**：443
   -  **後端儲存區**：同一地區中的儲存區
   -  為 **lb.YOUR-DOMAIN-NAME** 選擇 SSL 憑證

3. 按一下**建立**以配置 HTTPS 接聽器

在**地區 2** 的負載平衡器中**重複**相同的操作。

## 配置廣域負載平衡器
{: #global-load-balancer}

在此區段中，您會配置將送入資料流量配送給不同 {{site.data.keyword.Bluemix_notm}} 地區中配置的本端負載平衡器的廣域負載平衡器 (GLB)。

### 透過廣域負載平衡器在地區之間配送資料流量
透過導覽至服務下的[資源清單](https://{DomainName}/resources)，開啟所建立的 CIS 服務。

1. 導覽至**可靠性**下的**廣域負載平衡器**，並按一下**建立負載平衡器**。
2. 輸入 **lb.YOUR-DOMAIN-NAME** 作為主機名稱，並將 TTL 設定為 60 秒。
3. 按一下**新增儲存區**以定義預設原始儲存區
   - **名稱**：lb-region1
   - **性能檢查**：CREATE A NEW HEALTH CHECK
     - **監視器類型**：HTTP
     - **路徑**：/
     - **埠**：80
   - **性能檢查地區**：Eastern North America
   - **原點**
     - **名稱**：region1
     - **位址**：ADDRESS OF **REGION1** LOCAL LOAD BALANCER
     - **加權**：1
     - 按一下**新增**

4. 再**新增**一個指向**西歐**地區中**地區 2** 負載平衡器的**原始儲存區**，並按一下**佈建 1 個資源**，以佈建廣域負載平衡器。

等待**性能**檢查狀態變更為**性能良好**。開啟所選瀏覽器中的鏈結 **lb.YOUR-DOMAIN-NAME** 以查看動作中的廣域負載平衡器。

### 失效接手測試
到現在為止，您應該看到大多數情況下，您命中**地區 1** 中的伺服器，因為向其指派了相較於**地區 2** 中的伺服器更高的加權。接下來在**地區 1** 原始儲存區中引入性能檢查故障。

1. 導覽至[虛擬伺服器實例](https://{DomainName}/vpc/compute/vs)。
2. 按一下在**地區 1** 中**區域 1** 內執行的伺服器旁邊的**三個點 (...)**，並按一下**停止**。
3. 對於**地區 1** 的**區域 2** 中執行的伺服器，**重複**相同的操作。
4. 回到 CIS 服務下的 GLB，並等待性能狀態變更為**嚴重**。
5. 現在，重新整理網域 URL 時，您應該會始終命中**地區 2** 中的伺服器。

不要忘記**啟動**地區 1 的區域 1 和區域 2 中的伺服器
{:tip}

## 移除資源
{: #removeresources}

- 移除 CIS 服務下的廣域負載平衡器、原始儲存區和性能檢查。
- 移除 Certificate Manager 服務中的憑證。
- 移除負載平衡器、VSI、子網路和 VPC。
- 在[資源清單](https://{DomainName}/resources)下，刪除本指導教學中使用的服務。


## 相關內容
{: #related}

* [在 IBM Cloud VPC 中使用負載平衡器](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc#--beta-using-load-balancers-in-ibm-cloud-vpc)
