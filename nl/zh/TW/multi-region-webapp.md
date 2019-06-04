---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 跨多個地區保護 Web 應用程式
{: #multi-region-webapp}

本指導教學將全程指導您使用 [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) 管道跨多個地區對 Cloud Foundry 應用程式進行建立、保護、部署和負載平衡。

應用程式或應用程式的某些部分會發生中斷 - 這是事實。原因可能是程式碼中存在問題，計劃維護影響應用程式使用的資源，硬體故障導致管理應用程式的區域、位置或資料中心運作中斷。其中任何問題都有可能會發生，因此您必須做好準備應對。藉由 {{site.data.keyword.Bluemix_notm}}，可以將應用程式部署到[多個位置](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg)，以提高應用程式的備援能力。現在，應用程式在多個位置執行時，還可以將使用者資料流量重新導向到距離最近的位置以縮短延遲。

## 目標

* 使用 {{site.data.keyword.contdelivery_short}} 將 Cloud Foundry 應用程式部署到多個位置。
* 將自訂網域對映到應用程式。
* 為多位置應用程式配置廣域負載平衡。
* 將 SSL 憑證連結至應用程式。
* 監視應用程式效能。

## 使用的服務

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry 應用程式
* [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) for DevOps
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構

本指導教學涉及主動/主動情境，其中應用程式的兩個副本部署在兩個不同的位置，這兩個副本以循環方式處理客戶要求。如果一個副本失敗，DNS 配置會自動指向性能良好的位置。

<p style="text-align: center;">

   ![架構](./images/solution1/Architecture.png)
</p>

## 建立 Node.js 應用程式
{: #create}

先建立在 Cloud Foundry 環境中執行的 Node.js 入門範本應用程式。

1. 按一下 {{site.data.keyword.Bluemix_notm}} 主控台中的**[型錄](https://{DomainName}/catalog/)**。
2. 按一下**平台**種類下的 **Cloud Foundry 應用程式**，然後選取 **[{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)**。
     ![](images/solution1/SDKforNodejs.png)
3. 輸入應用程式的**唯一名稱**，這也將是您的主機名稱，例如：myusername-nodeapp。然後，按一下**建立**。
4.  應用程式啟動後，按一下**概觀**頁面上的**造訪 URL** 鏈結，以在新標籤上即時查看應用程式。

![HelloWorld](images/solution1/HelloWorld.png)

很棒的開始！您已擁有自己的在 {{site.data.keyword.Bluemix_notm}} 中執行的 Node.js 入門範本應用程式。

接下來，將應用程式的原始碼推送到儲存庫並自動部署變更。

## 設定來源控制和 {{site.data.keyword.contdelivery_short}}
{: #devops}

在此步驟中，將設定 Git 原始碼控制儲存庫來儲存程式碼，然後建立管道，用於自動部署任何程式碼變更。

1. 在剛才建立的應用程式的左側窗格中，選取**概觀**，然後捲動以找到 **{{site.data.keyword.contdelivery_short}}**。按一下**啟用**。

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. 保留預設選項，然後按一下**建立**。現在，您應該已建立預設**工具鏈**。

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. 選取**程式碼**下的 **Git** 磚。然後，系統會將您導向到 Git 儲存庫頁面。
4. 如果您尚未設定 SSH 金鑰，應該會在頂端看到含有指示的通知列。請在新的分頁開啟**新增 SSH 金鑰**鏈結並遵循步驟，或是如果您想要改用 HTTPS 而非 SSH 的話，請按一下**建立個人存取記號**並遵循步驟。請記得儲存金鑰或記號，以便未來參考。
5. 選取 SSH 或 HTTPS 並複製 Git URL。將原始檔複製到您的本端機器。
   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   **附註：**如果系統提示您輸入使用者名稱，請提供您的 Git 使用者名稱。針對密碼，請使用現有的 **SSH 金鑰**或**個人存取記號**，或是您在先前步驟中建立的 SSH 金鑰或個人存取記號。
6. 在您選擇的 IDE 中開啟複製的儲存庫，並導覽至 `public/index.html`。現在，將更新程式碼。請嘗試將 "Hello World" 變更為其他內容。
7. 透過依次執行 `npm install`、`npm build`、`npm start ` 指令來本端執行應用程式，然後在瀏覽器中造訪 `localhost:<port_number>`。
  **<port_number>** 如主控台中所顯示。
8. 以三個簡單的步驟將變更推送至儲存庫：新增、確定、推送。
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
9. 移至您先前建立的工具鏈，然後按一下 **Delivery Pipeline** 磚。
10. 確認是否顯示有**建置**和**部署**階段。
   ![HelloWorld](images/solution1/DevOps_Pipeline.png)
11. 等待**部署**階段完成。
12. 在前次執行結果下，按一下應用程式 **url** 以即時檢視變更。

繼續對應用程式做進一步變更，並定期將變更確定到 Git 儲存庫。如果您看不到應用程式更新，請檢查管線的部署及建置階段的日誌。

## 部署到其他位置
{: #deploy_another_region}

接下來，要將同一應用程式部署到其他 {{site.data.keyword.Bluemix_notm}} 位置。我們可以使用相同的工具鏈，但會新增另一個「部署」階段來處理將應用程式部署到其他位置的過程。

1. 導覽至應用程式**概觀**，然後捲動以找到**檢視工具鏈**。
2. 從「交付」中，選取**交付管道**。
3. 按一下**部署**階段上的**齒輪**圖示，然後選取**複製階段**。
   ![HelloWorld](images/solution1/CloneStage.png)
4. 將階段重新命名為「部署到 UK 」，然後選取**工作**。
5. 將 **IBM Cloud 地區**變更為**倫敦 - https://api.eu-gb.bluemix.net**。如果還沒有空間，請建立**空間**。
6. 將**部署 Script**變更為 `cf push "${CF_APP}" -d eu-gb.mybluemix.net`。

   ![HelloWorld](images/solution1/DeployToUK.png)
7. 按一下**儲存**，然後透過按一下**播放**按鈕來執行新階段。

## 向 IBM Cloud Internet Services 登錄自訂網域

{: #domain_cis}

IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) 是一個統一平台，用於為 Web 應用程式配置和管理網域名稱系統 (DNS)、廣域負載平衡 (GLB)、Web 應用程式防火牆 (WAF) 以及防止分散式阻斷服務 (DDoS)。此平台為在 IBM Cloud 上執行業務的客戶提供快速、高效能、可靠且安全的網際網路服務，其中有三個主要功能可增強工作流程：安全、可靠性和效能。  

部署真實世界的應用程式時，您可能希望使用自己的網域，而不是 IBM 提供的 mybluemix.net 網域。在此步驟中，擁有自訂網域後，可以使用 IBM Cloud Internet Services 提供的 DNS 伺服器。

1. 向註冊商（例如，[http://godaddy.com](http://godaddy.com)）購買網域。
2. 導覽至 {{site.data.keyword.Bluemix_notm}} 型錄中的 [Internet Services](https://{DomainName}/catalog/services/internet-services)。
2. 輸入服務名稱，然後按一下**建立**以建立服務實例。
3. 佈建服務實例時，設定您的網域名稱，然後按一下**新增網域**。
4. 指派名稱伺服器時，請配置登記員或網域名稱提供者以便使用列出的名稱伺服器。
5. 配置登記員或 DNS 提供者之後，可能需要最多 24 小時，變更才會生效。在「概觀」頁面上，網域的狀態從*擱置* 變更為*作用中* 時，可以使用 `dig <your_domain_name> ns` 指令來驗證 IBM Cloud 名稱伺服器是否已生效。
  {:tip}

## 向應用程式新增廣域負載平衡

{: #add_glb}

在此區段中，您將使用 IBM Cloud Internet Services 中的廣域負載平衡器 (GLB) 來管理多個位置的資料流量。GLB 利用了可將資料流量配送給多個原點的原始儲存區。

### 建立 GLB 之前，請為 GLB 建立性能檢查。

1. 在 Cloud Internet Services 應用程式中，導覽至**可靠性** > **廣域負載平衡器**，然後按一下頁面底端的**建立性能檢查**。
2. 輸入要監視的路徑，例如 `/`，然後選取類型（HTTP 或 HTTPS）。通常，可以建立專用的性能端點。按一下**佈建 1 個實例**。
   ![性能檢查](images/solution1/health_check.png)

### 在此之後，建立具有兩個原點的原始儲存區。

1. 按一下**建立儲存區**。
2. 輸入儲存區的名稱，選取剛才建立的性能檢查，以及靠近 Node.js 應用程式所在位置的地區。
3. 輸入位於達拉斯的第一個原點的名稱和應用程式的主機名稱：`<your_app>.mybluemix.net`。
4. 與此類似，新增另一個原點，其中原點位址指向位於倫敦的應用程式：`<your_app>.eu-gb.mybluemix.net`。
5. 按一下**佈建 1 個實例**。
   ![原始儲存區](images/solution1/origin_pool.png)

### 建立廣域負載平衡器 (GLB)。 

1. 按一下**建立負載平衡器**。
2. 輸入廣域負載平衡器的名稱。這個名稱也將是您通用應用程式 URL (`http://<glb_name>.<your_domain_name>`) 的一部分，而不論其位置在何處。
3. 按一下**新增儲存區**，然後選取剛才建立的原始儲存區。
4. 按一下**佈建 1 個實例**。
   ![廣域負載平衡器](images/solution1/load_balancer.png)

在此階段，GLB 已配置，但 Cloud Foundry 應用程式尚未準備好回覆來自所配置 GLB 網域名稱的要求。若要完成配置，將使用自訂網域和路徑來更新應用程式。

## 配置應用程式的自訂網域和路徑

{: #add_domain}

在此步驟中，要將自訂網域名稱對映到執行應用程式的 {{site.data.keyword.Bluemix_notm}} 位置的安全端點。

1. 在功能表列中，按一下**管理**，然後按一下**帳戶**：[帳戶](https://{DomainName}/account)。
2. 在「帳戶」頁面上，導覽至應用程式的 **Cloud Foundry 組織**，然後從「動作」直欄中，選取**網域**。
3. 按一下**新增網域**，然後輸入從註冊商那裡獲得的自訂網域名稱。
4. 選取正確的位置，然後按一下**儲存**。
5. 與此類似，將自訂網域名稱新增到「倫敦」。
6. 回到 {{site.data.keyword.Bluemix_notm}} [資源清單](https://{DomainName}/resources)，導覽至 **Cloud Foundry 應用程式**，按一下位於達拉斯的應用程式，按一下**路徑** > **編輯路徑**，然後按一下**新增路徑**。
   ![新增路徑](images/solution1/ApplicationRoutes.png)
7. 輸入先前在**輸入主機（選用）**欄位中配置的 GLB 主機名稱，然後選取剛才新增的自訂網域。按一下**儲存**。
8. 與此類似，為位於倫敦的應用程式配置網域和路徑。

此時，可以使用 URL `<glb_name>.<your_domain_name>` 來造訪應用程式，並且廣域負載平衡器會自動為多位置應用程式配送資料流量。要對此進行驗證，可以停止位於達拉斯的應用程式，使位於倫敦的應用程式保持作用中狀態，然後透過廣域負載平衡器來存取應用程式。

雖然此時上述操作有效，但由於在先前的步驟中配置了持續交付，因此在觸發另一個建置時，可能會改寫配置。若要使這些變更持續存在，請回到工具鏈並修改 *manifest.yml* 檔案：

1. 在 {{site.data.keyword.Bluemix_notm}} [資源清單](https://{DomainName}/resources)中，導覽至 **Cloud Foundry 應用程式**，按一下位於達拉斯的應用程式，導覽至應用程式**概觀**，然後捲動以找到**檢視工具鏈**。
2. 選取「程式碼」下的 Git 磚。
3. 選取 *manifest.yml*。
4. 按一下**編輯**，然後新增自訂路徑。僅取代 `Routes` 中的原始網域和主機配置。

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <glb_name>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. 確定變更，並確保兩個位置的建置都成功。  

## 替代方案：將自訂網域對映到 IBM Cloud 系統網域

您可能不希望在多位置應用程式前端使用廣域負載平衡器，但需要將自訂網域名稱對映到執行應用程式的 {{site.data.keyword.Bluemix_notm}} 位置的安全端點。

使用 Cloud Intenet Services 應用程式時，請執行下列步驟為應用程式設定 `CNAME` 記錄：

1. 在 Cloud Internet Services 應用程式中，導覽至**可靠性** > **DNS**。
2. 從**類型**下拉清單中，選取 **CNAME**，在「名稱」欄位中，鍵入應用程式的別名，然後在「網域名稱」欄位中，鍵入應用程式 URL。位於達拉斯的應用程式 `<your_app>.mybluemix.net` 可以對映到 CNAME `<your_app>`。
3. 按一下**新增記錄**。將 "Proxy" 切換開關切換到「開啟」可增強應用程式的安全。
4. 與此類似，為「倫敦」端點設定 `CNAME` 記錄。
   ![CNAME 記錄](images/solution1/cnames.png)

使用除 `mybluemix.net` 之外的其他預設網域（例如，`cf.appdomain.cloud` 或 `cf.cloud.ibm.com`）時，請確保使用[相應的系統網域](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain)。
{:tip}

如果使用的是其他 DNS 提供者，則設定 CNAME 記錄的步驟會因 DNS 提供者而有所不同。例如，如果使用的是 GoDaddy，請遵循 GoDaddy 中的 [Domains Help](https://www.godaddy.com/help/add-a-cname-record-19236) 指引。

要使 Cloud Foundry 應用程式可透過自訂網域存取，需要將自訂網域新增到[部署了應用程式的 Cloud Foundry 組織中的網域清單](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#updatingapps)。完成後，可以將路徑新增至應用程式資訊清單中：

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <your_app>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}

## 將 SSL 憑證連結至應用程式
{: #ssl}

1. 取得 SSL 憑證。例如，可以從 https://www.godaddy.com/web-security/ssl-certificate 購買 SSL 憑證，也可以在 https://letsencrypt.org/ 上免費產生 SSL 憑證。
2. 導覽至應用程式**概觀** > **路徑** > **管理網域**。
3. 按一下「SSL 憑證上傳」按鈕並上傳憑證。
5. 使用 https（而不是 http）來存取應用程式。

## 監視應用程式效能
{: #monitor}

下面將檢查多位置應用程式的性能。

1. 在應用程式儀表板中，選取**監視**。
2. 按一下**檢視所有測試**
   ![](images/solution1/alert_frequency.png)。

Availability Monitoring 在全球各個位置全天候執行合成測試，以主動偵測和修正效能問題，避免這些問題影響使用者。如果為應用程式配置了自訂路徑，請變更測試定義，以透過其自訂網域來存取應用程式。

## 移除資源

* 刪除工具鏈
* 刪除在兩個位置部署的兩個 Cloud Foundry 應用程式
* 刪除 GLB、原始儲存區和性能檢查
* 刪除 DNS 配置
* 刪除 Internet Services 實例

## 相關內容

[新增 Cloudant 資料庫](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[自動調整 Cloud Foundry 應用程式](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
