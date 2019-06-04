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

# 跨多個地區部署無伺服器應用程式
{: #multi-region-serverless}

本指導教學說明瞭如何配置 IBM Cloud Internet Services 和 {{site.data.keyword.openwhisk_short}}，以跨多個地區部署無伺服器應用程式。

透過無伺服器運算平台，開發人員能快速建置 API，而無需伺服器。{{site.data.keyword.openwhisk}} 支援為動作自動產生 REST API、將動作轉換為 HTTP 端點，並能夠啟用安全 API 鑑別。此功能不僅有助於向外部消費者公開 API，還可協助建置微服務應用程式。

{{site.data.keyword.openwhisk_short}} 在多個 {{site.data.keyword.cloud_notm}} 位置都有提供。為了提高備援及縮短網路延遲，應用程式可以在多個位置部署其後端。使用 IBM Cloud Internet Services (CIS)，開發人員可以公開單一進入點，來負責將資料流量分散至最近的性能健全後端。

## 目標
{: #objectives}

* 部署 {{site.data.keyword.openwhisk_short}} 動作。
* 使用自訂網域透過 {{site.data.keyword.APIM}} 公開動作。
* 使用 Cloud Internet Services 跨多個位置配送資料流量。

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

本指導教學討論的是使用 {{site.data.keyword.openwhisk_short}} 實作後端的公用 Web 應用程式。為了縮短網路延遲並防止中斷，應用程式將部署在多個位置。本指導教學中配置了兩個位置。

<p style="text-align: center;">

  ![架構](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. 使用者存取應用程式。此要求是透過 Internet Services 來進行。
2. Internet Services 將使用者重新導向到最近的性能健全 API 後端。
3. {{site.data.keyword.cloudcerts_short}} 為 API 提供其 SSL 憑證。資料流量會全程加密。
4. API 透過 {{site.data.keyword.openwhisk_short}} 實作。

## 開始之前
{: #prereqs}

1. Cloud Internet Services 需要您擁有自訂網域，因此您可以設定此網域的 DNS，指向 Cloud Internet Services 名稱伺服器。如果您未擁有網域，可以從例如 [godaddy.com](http://godaddy.com) 的註冊商購買一個。
1. [遵循下列步驟](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)以安裝所有必要的指令行 (CLI) 工具。

## 配置自訂網域

第一步是建立 IBM Cloud Internet Services (CIS) 實例，並將自訂網域指向 CIS 名稱伺服器。

1. 導覽至 {{site.data.keyword.Bluemix_notm}} 型錄中的 [Internet Services](https://{DomainName}/catalog/services/internet-services)。
1. 設定服務名稱，然後按一下**建立**以建立服務的實例。您可以針對本指導教學使用任何定價方案。
1. 佈建服務實例時，藉由按一下**讓我們開始吧**設定您的網域名稱，然後按一下**新增網域**。
1. 按**下一步**。指派名稱伺服器時，請配置登記員或網域名稱提供者以便使用列出的名稱伺服器。
1. 配置登記員或 DNS 提供者之後，可能需要最多 24 小時，變更才會生效。

   當「概觀」頁面上的網域狀態從 *Pending* 變更為 *Active* 時，您可以使用 `dig <your_domain_name> ns` 指令驗證新的名稱伺服器已生效。
   {:tip}

### 取得自訂網域的憑證

透過自訂網域公開 {{site.data.keyword.openwhisk_short}} 動作，將需要安全的 HTTPS 連線。您應該為計劃搭配無伺服器後端使用的網域及子網域，取得 SSL 憑證。假設有類似 *mydomain.com* 的網域名稱，可以在 *api.mydomain.com* 上管理動作。需要簽發用於 *api.mydomain.com* 的憑證。

您可以從 [Let's Encrypt](https://letsencrypt.org/) 取得免費的 SSL 憑證。在過程中，您可能需要在 Cloud Internet Services 的 DNS 介面，配置 TXT 類型的 DNS 記錄，以證明您是網域的擁有者。
{:tip}

取得網域的 SSL 憑證及私密金鑰之後，請務必將它們轉換成 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 格式。

1. 將憑證轉換為 PEM 格式：
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. 將私密金鑰轉換為 PEM 格式：
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### 將憑證匯入至中央儲存庫

1. 在支援的位置建立 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 實例。
1. 在服務儀表板中，使用**匯入憑證**：
   * 將**名稱**設定為自訂子網域和網域，例如 *api.mydomain.com*。
   * 瀏覽 PEM 格式的**憑證檔案**。
   * 瀏覽 PEM 格式的**私密金鑰檔案**。
   * **匯入**。

## 在多個位置部署動作

在此區段中，您將建立動作，將其公開為 API，然後使用儲存在 {{site.data.keyword.cloudcerts_short}} 中的 SSL 憑證將自訂網域對映到 API。

<p style="text-align: center;">

  ![API 架構](images/solution44-multi-region-serverless/api-architecture.png)
</p>

**doWork** 動作可實作其中一個 API 作業。**healthz** 將稍後用於檢查 API 是否性能正常。檢查結果可能只傳回 *OK*，不包含任何操作，也可能會執行更複雜的檢查，例如對資料庫或對 API 所需的其他關鍵服務執行 ping 操作。

對於要管理應用程式後端的每個位置，需要重複下列三個區段。在本指導教學中，可以選取*達拉斯 (us-south)* 和*倫敦 (eu-gb)* 作為目標。

### 定義動作

1. 移至 [{{site.data.keyword.openwhisk_short}} / 動作](https://{DomainName}/openwhisk/actions)。
1. 切換到目標位置，然後選取要部署動作的組織和空間。
1. 建立動作
   1. 將**名稱**設定為 **doWork**。
   1. 將**含括套件**設定為 **default**。
   1. 將**運行環境**設定為最近版本的 **Node.js**。
   1. **建立**。
1. 將動作程式碼變更為：
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. **儲存**
1. 建立另一個動作以用作 API 的性能檢查：
   1. 將**名稱**設定為 **healthz**。
   1. 將**含括套件**設定為 **default**。
   1. 將**運行環境**設定為最近的 **Node.js**。
   1. **建立**。
1. 將動作程式碼變更為：
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. **儲存**

### 使用受管理 API 公開動作

下一步涉及建立受管理 API 來公開動作。

1. 移至 [{{site.data.keyword.openwhisk_short}} / API](https://{DomainName}/openwhisk/apimanagement)。
1. 建立新的受管理 {{site.data.keyword.openwhisk_short}} API：
   1. 將 **API 名稱**設定為 **App API**。
   1. 將**基本路徑**設定為 **/api**。
1. 建立作業：
   1. 將**路徑**設定為 **/do**。
   1. 將**動詞**設定為 **GET**。
   1. 將**套件**設定為 **default**。
   1. 將**動作**設定為 **doWork**。
   1. **建立**
1. 建立另一個作業：
   1. 將**路徑**設定為 **/healthz**。
   1. 將**動詞**設定為 **GET**。
   1. 將**套件**設定為 **default**。
   1. 將**動作**設定為 **healthz**。
   1. **建立**
1. **儲存**該 API

### 配置受管理 API 的自訂網域

建立受管理 API 會提供您預設端點，例如 `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`。在此區段中，您將配置此端點，使其能夠處理來自於自訂子網域的要求，該網域稍後將在 IBM Cloud Internet Services 中進行配置。

1. 移至 [API / 自訂網域](https://{DomainName}/apis/domains)。
1. 在**地區**選取器中，選取目標位置。
1. 找到鏈結至您建立動作和受管理 API 之組織及空間的自訂網域。在動作功能表中按一下**變更設定**。
1. 記下**預設網域 / 別名**值。
1. 勾選**套用自訂網域**
   1. 將**網域名稱**設定為將用於 CIS 廣域負載平衡器的網域，例如 *api.mydomain.com*。
   1. 選取持有憑證的 {{site.data.keyword.cloudcerts_short}} 實例。
   1. 選取網域的憑證。
1. 移至 **Cloud Internet Services** 的儀表板，在**可靠性 / DNS** 下，建立新的 **DNS TXT 記錄**：
   1. 將**名稱**設定為自訂子網域，例如 **api**。
   1. 將**內容**設定為**預設網域 / 別名**。
   1. 儲存記錄。
1. 儲存自訂網域設定。對話框會檢查 DNS TXT 記錄是否存在。

   如果找不到 TXT 記錄，可能需要等待該記錄傳播，然後重試儲存設定。設定套用之後便可以移除 DNS TXT 記錄。
   {: tip}

重複以上各區段以配置更多位置。

## 在各位置之間配送資料流量

**在此階段，您已在多個位置設定動作**，但沒有單一進入點可以對其進行存取。在此區段中，您將配置廣域負載平衡器 (GLB) 以在各位置之間分散資料流量。

<p style="text-align: center;">

  ![廣域負載平衡器的架構](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### 建立性能檢查

Internet Services 將定期呼叫此端點以檢查後端的性能。

1. 前往 IBM Cloud Internet Services 實例的儀表板。
1. 在**可靠性 / 廣域負載平衡器**下，建立性能檢查：
   1. 將**監視類型**設定為 **HTTPS**。
   1. 將**路徑**設定為 **/api/healthz**。
   1. **佈建資源**。

### 建立原點儲存區

透過為每個位置建立一個儲存區，可以稍後在廣域負載平衡器中配置地理路徑，以將使用者重新導向到距離最近的位置。另一個選項是建立包含所有位置的單一儲存區，並使負載平衡器循環使用儲存區中的各個原點。

對於每個位置：
1. 建立原始儲存區。
1. 將**名稱**設定為 **app-&lt;location&gt;**，例如 _app-Dallas_。
1. 選取先前建立的性能檢查。
1. 將**性能檢查地區**設定為距離部署 {{site.data.keyword.openwhisk_short}} 的位置最近的地區。
1. 將**原點名稱**設定為 **app-&lt;location&gt;**。
1. 將**原點位址**設定為受管理 API 的預設網域/別名（例如，_5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_）。
1. **佈建資源**。

### 建立廣域負載平衡器

1. 建立負載平衡器。
1. 將**平衡器主機名稱**設定為 **api.mydomain.com**。
1. 新增地區原始儲存區。
1. **佈建資源**。

片刻之後，移至 `https://api.mydomain.com/api/do?name=John&place=Earth`。這應該會回覆為第一個性能良好的儲存區中所執行的函數。

### 測試失效接手

若要測試失效接手，儲存區性能檢查結果必須為故障，這樣 GLB 才會重新導向到下一個性能良好的儲存區。若要模擬故障情況，可以修改性能檢查函數以使其發生故障。

1. 移至 [{{site.data.keyword.openwhisk_short}} / 動作](https://{DomainName}/openwhisk/actions)。
1. 選取 GLB 中配置的第一個位置。
1. 編輯 `healthz` 函數，將其實作變更為 `throw new Error()`。
1. 儲存。
1. 等待對此原始儲存區執行性能檢查。
1. 再次取得 `https://api.mydomain.com/api/do?name=John&place=Earth`，它現在應該重新導向到另一個性能良好的原點。
1. 回復程式碼變更，使其回復為性能良好的原點。

## 移除資源
{: #removeresources}

### 移除 CIS 資源

1. 移除 GLB。
1. 移除原點儲存區。
1. 移除性能檢查。

### 移除動作

1. 移除 [API](https://{DomainName}/openwhisk/apimanagement)
1. 移除[動作](https://{DomainName}/openwhisk/actions)

## 相關內容
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [使用 Cloud Internet Services 的具復原力且安全的多地區 Kubernetes 叢集](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
