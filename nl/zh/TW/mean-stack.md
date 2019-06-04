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


# 使用 MEAN 堆疊的現代 Web 應用程式
{: #mean-stack}

本指導教學將全程指導您使用熱門 MEAN 堆疊建立 Web 應用程式。MEAN 堆疊由 **M**ongo DB、**E**xpress Web 架構、**A**ngular 前端架構和 Node.js 運行環境組成。您將瞭解如何在本端執行 MEAN 入門範本，建立和使用受管理資料庫即服務 (DBasS)，將應用程式部署到 {{site.data.keyword.cloud_notm}} 以及監視應用程式。  

## 目標

{: #objectives}

- 在本端建立並執行入門範本 Node.js 應用程式。
- 建立受管理資料庫即服務 (DBasS)。
- 將 Node.js 應用程式部署到雲端。
- 調整 MongoDB 資源。
- 瞭解如何監視應用程式效能。

## 使用的服務

{: #products}

本指導教學使用下列 {{site.data.keyword.Bluemix_notm}} 服務：

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**注意：**本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構

{:#architecture}

<p style="text-align: center;">

![架構圖](images/solution7/Architecture.png)</p>

1. 使用者使用 Web 瀏覽器存取應用程式。
2. Node.js 應用程式移至 {{site.data.keyword.composeForMongoDB}} 資料庫來提取資料。

## 開始之前

{: #prereqs}

1. [安裝 Git](https://git-scm.com/)
2. [安裝 {{site.data.keyword.Bluemix_notm}} CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)


若要在本端開發和執行應用程式：
1. [安裝 Node.js 和 NPM](https://nodejs.org/)
2. [安裝和執行 MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/)

## 本端執行 MEAN 應用程式

{: #runapplocally}

在此區段中，您將執行本端 MongoDB 資料庫，複製 MEAN 範例程式碼，並在本端執行應用程式以使用本端 MongoDB 資料庫。

{: shortdesc}

1. 按照[這裡](https://docs.mongodb.com/manual/administration/install-community/)的指示在本端安裝並執行 MongoDB 資料庫。安裝完成後，使用下列指令來確認該 **mongod** 伺服器及您的資料庫是否執行中。
  ```sh
  mongo
  ```
  {: codeblock}

2. 複製 MEAN 入門範本程式碼。

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. 安裝必要的套裝軟體。

  ```sh
npm install
```
  {: codeblock}

4. 將 .env.example 檔案複製到 .env。編輯所需的資訊，至少新增您自己的 SESSION_SECRET。

5. 執行 node server.js 以啟動應用程式。
  ```sh
  node server.js
  ```
  {: codeblock}

6. 存取應用程式，建立新使用者並登入。

## 在雲端中建立 MongoDB 資料庫的實例

{: #createdatabase}

在此區段中，您將在雲端中建立 {{site.data.keyword.composeForMongoDB}} 資料庫。{{site.data.keyword.composeForMongoDB}} 是資料庫即服務，通常更容易配置，並提供內建備份和調整功能。您可以在 [IBM Cloud 型錄](https://{DomainName}/catalog/?category=data)找到許多不同類型的資料庫。若要建立 {{site.data.keyword.composeForMongoDB}}，請遵循底下的步驟。

{: shortdesc}

1. 透過指令行登入到您的 {{site.data.keyword.cloud_notm}} 帳戶，並將您的 {{site.data.keyword.cloud_notm}} 帳戶設定為目標。 

  ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
  {: codeblock}

  可以在[這裡](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)找到更多 CLI 指令。

2. 建立 {{site.data.keyword.composeForMongoDB}} 實例。這還可使用[主控台使用者介面](https://{DomainName}/catalog/services/compose-for-mongodb)來執行。服務名稱必須命名為 **mean-starter-mongodb**，因為應用程式配置為使用此名稱來尋找此服務。

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## 將應用程式部署到雲端

{: #deployapp}

在此區段中，您要將 node.js 應用程式部署到使用受管理 MongoDB 資料庫的 {{site.data.keyword.cloud_notm}}。原始碼包含 [**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml) 檔案，該檔案已配置為使用先前建立的 "mongodb" 服務。應用程式使用 VCAP_SERVICES 環境變數來存取 Compose MongoDB 資料庫認證。這可以在 [server.js 檔案](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js)中進行檢視。 

{: shortdesc}

1. 將程式碼推送到雲端。

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. 推送程式碼後，您應該可以在瀏覽器中檢視應用程式。系統產生了隨機主機名稱，可能類似於：`https://mean-random-name.mybluemix.net`。您可以透過主控台儀表板或指令行來取得應用程式 URL。![即時應用程式](images/solution7/live-app.png)


## 調整 MongoDB 資料庫資源
{: #scaledatabase}

如果您的服務需要額外儲存空間空間，或者您要減少配置給服務的儲存空間數量，可以透過調整資源來完成此動作。

{: shortdesc}

1. 使用主控台**儀表板**，移至**連線**區段，然後按一下 **MongoDB 實例**資料庫。
2. 在**部署詳細資料**畫面中，按一下**調整資源**選項。
  ![](images/solution7/mongodb-scale-show.png)
3. 調整**調節器**以增加或減少配置給 {{site.data.keyword.composeForMongoDB}} 資料庫服務的儲存空間量。
4. 按一下**調整部署**以觸發重新調整並回到儀表板概觀頁面。這將在頁面頂端顯示「調整已起始」訊息，通知您正在執行重新調整。
  ![](images/solution7/scaling-in-progress.png)


## 監視應用程式效能
{: #monitorapplication}

若要檢查應用程式的性能，可以使用內建的可用性監視服務。可用性監視服務會自動連接到雲端中的應用程式。可用性監視服務在全球各個位置全天候執行合成測試，以主動偵測和修正效能問題，避免這些問題影響訪客。執行以下步驟來存取監視儀表板。

{: shortdesc}

1. 使用主控台儀表板，在您的應用程式下，選取**監視**標籤。
2. 按一下**檢視所有測試**來檢視測試。
   ![](images/solution7/alert_frequency.png)


## 相關內容

{: #related}

- 設定來源控制和[持續交付](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops)。
- 在[多個位置](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp)中保護 Web 應用程式。
- 建立、保護和管理 [REST API](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis)。
