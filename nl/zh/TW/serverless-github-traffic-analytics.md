---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 將無伺服器與 Cloud Foundry 組合用於資料擷取和分析
{: #serverless-github-traffic-analytics}
在本指導教學中，您將建立一個應用程式，用於自動收集儲存庫的 GitHub 資料流量統計資料，並為資料流量分析奠定基礎。GitHub 僅提供對最近 14 天的資料流量的存取。如果要分析更長時間段內的統計資料，您需要自行下載並儲存這些資料。在本指導教學中，您將部署無伺服器動作，以擷取資料流量資料並將其儲存在 SQL 資料庫中。此外，將使用 Cloud Foundry 應用程式來管理儲存庫，並提供對統計資料的存取以進行資料分析。本指導教學中討論的應用程式和無伺服器動作實作的是支援多租戶的解決方案，其起始特性集支援單一承租戶模式。

![](images/solution24-github-traffic-analytics/Architecture.png)

## 目標

* 部署具有多方承租戶支援和安全存取權的 Python 資料庫應用程式
* 將 App ID 整合為以 OpenID Connect 為基礎的鑑別服務提供者
* 設定 GitHub 資料流量統計資料的自動無伺服器收集
* 整合 {{site.data.keyword.dynamdashbemb_short}} 以進行圖形資料流量分析

## 產品
本指導教學使用下列產品：
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## 開始之前
{: #prereqs}

若要完成本指導教學，您需要安裝最新版本的 [IBM Cloud CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) 及 {{site.data.keyword.openwhisk_short}} [外掛程式](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)。

## 服務和環境設定 (Shell)
在此區段中，您將設定所需的服務並準備環境。所有這些作業都可以透過 Shell 環境完成。

1. 複製 [GitHub 儲存庫](https://github.com/IBM-Cloud/github-traffic-stats)，然後導覽至複製的目錄及其 **backend** 子目錄：
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. 使用 `ibmcloud login` 以互動方式登入 {{site.data.keyword.Bluemix_short}}。您可以執行 `ibmcloud target` 指令，重新確認詳細資料。您需要設定組織和空間。

3. 建立使用**入門**方案的 {{site.data.keyword.dashdbshort}} 實例，並將其命名為 **ghstatsDB**：
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. 若之後要從 {{site.data.keyword.openwhisk_short}} 存取資料庫服務，您需要授權。因此，請建立服務認證，並將其標示為 **ghstatskey**：   
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. 建立 {{site.data.keyword.appid_short}} 服務實例。使用 **ghstatsAppID** 作為名稱，並使用**累進層級**方案。
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
   然後，在 Cloud Foundry 空間中建立該新服務實例的別名。將 **YOURSPACE** 取代為要部署到的空間。
   ```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. 建立使用**精簡**方案的 {{site.data.keyword.dynamdashbemb_short}} 服務實例。
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
   同樣，建立該新服務實例的別名，並取代 **YOURSPACE**。
   ```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. 在 **backend** 目錄中，將應用程式推送到 IBM Cloud。該指令會將隨機路徑用於應用程式。
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
   等待部署完成。此時應用程式檔案已上傳，執行時期環境已建立，並且服務已連結至應用程式。服務資訊是從 `manifest.yml` 檔案中取得的。如果使用了其他服務名稱，則需要更新該檔案。該程序順利完成後，將顯示應用程式 URI。

   上面的指令是將隨機但唯一的路徑用於應用程式。如果希望自己選取路徑，請將該路徑作為額外參數新增到指令，例如 `ibmcloud cf push your-app-name`。此外，還可以編輯 `manifest.yml` 檔案，變更 **name**，並將 **random-route** 從 **true** 變更為 **false**。
   {:tip}

## App ID 和 GitHub 配置（瀏覽器）
下列步驟全部使用網際網路瀏覽器執行。請先將 {{site.data.keyword.appid_short}} 配置為使用 Cloud Directory 和 Python 應用程式。然後，建立 GitHub 存取記號。部署的函數擷取資料流量資料時需要此存取記號。

1. 在 [{{site.data.keyword.Bluemix_short}} 資源清單](https://{DomainName}/resources)中，開啟您的服務概觀。在**服務**區段中，找到 {{site.data.keyword.appid_short}} 服務實例。按一下其項目以開啟詳細資料。
2. 在服務儀表板中，按一下左端功能表中**身分提供者**下的**管理**。這將顯示可用身分提供者的清單，例如 Facebook、Google、SAML 2.0 Federation 和 Cloud Directory。將 Cloud Directory 切換為**開啟**，將其他所有提供者切換為**關閉**。
   
   您可能需要配置[多因子鑑別 (MFA)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) 和進階密碼規則。但這些內容不在本指導教學中討論。
   {:tip}

3. 在該頁面底端是重新導向 URL 的清單。輸入應用程式的 **URL** + /redirect_uri。例如，`https://github-traffic-stats-random-word.mybluemix.net/redirect_uri`。

   要在本端測試應用程式，重新導向 URL 為 `http://0.0.0.0:5000/redirect_uri`。可以配置多個重新導向 URL。
   {:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. 在左側的功能表中，按一下**使用者**。這將開啟 Cloud Directory 中的使用者清單。按一下**新增使用者**按鈕將您自己新增為第一個使用者。現在，您已完成配置 {{site.data.keyword.appid_short}} 服務。
5. 在瀏覽器中，存取 [Github.com](https://github.com/settings/tokens)，然後移至 **Settings -> Developer settings -> Personal access tokens**。按一下 **Generate new token** 按鈕。對於**記號說明**，輸入 **GHStats 指導教學**。然後，啟用 **repo** 種類下的 **public_repo**，以及 **admin:org** 下的 **read:org**。現在，在該頁面的底端，按一下 **Generate token**。新存取記號會顯示在下一頁中。在接下來的應用程式設定期間，需要此存取記號。
   ![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## 配置並測試 Python 應用程式
準備工作完成後，可以配置並測試應用程式。應用程式是使用常用的 [Flask](http://flask.pocoo.org/) 微框架以 Python 撰寫的。可以在統計資料收集中新增和移除儲存庫。資料流量資料可以採用表狀視圖進行存取。

1. 在瀏覽器中，開啟已部署應用程式的 URI。您應該會看到歡迎使用頁面。
   ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. 在瀏覽器中，將 `/admin/initialize-app` 新增到 URI 並存取該頁面。該頁面用於起始設定應用程式及其資料。按一下 **Start initialization** 按鈕。這將使您前往受密碼保護的配置頁面。您用於登入的電子郵件位址會被視為系統管理者的識別。請使用您先前配置的電子郵件位址和密碼。

3. 在配置頁面中，輸入姓名（用於問候語）、GitHub 使用者名稱和先前產生的存取記號。按一下 **Initialize**。這將建立資料庫表格，並插入一些配置值。最後，會為系統管理者和租戶建立資料庫記錄。
   ![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. 完成後，您將前往受管理儲存庫的清單。現在，可以透過提供 GitHub 帳戶或組織的名稱以及儲存庫的名稱來新增儲存庫。輸入資料後，按一下**新增儲存庫**。儲存庫以及新指派的 ID 應該會顯示在表格中。可以透過輸入儲存庫 ID 並按一下**刪除儲存庫**，從系統中移除儲存庫。
![](images/solution24-github-traffic-analytics/RepositoryList.png)

## 部署 Cloud Functions 和觸發程式
實施管理應用程式後，請為 {{site.data.keyword.openwhisk_short}} 部署動作、觸發程式以及用於連接動作和觸發程式的規則。這些物件用於按指定的排定自動收集 GitHub 資料流量資料。動作會連接至資料庫，反覆運算所有租戶及其儲存庫，並取得每個儲存庫的視圖和複製作業資料。這些統計資料會合併到資料庫中。

1. 切換到 **functions** 目錄。
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. 建立新動作 **collectStats**。這將使用已包含必要資料庫驅動程式的 [Python 3 環境](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments)。動作的原始碼在 `ghstats.zip` 檔案中提供。
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   如果修改了動作的原始碼 (`__main__.py`)，可以再次使用 `zip -r ghstats.zip  __main__.py github.py` 來重新套裝 zip 保存。如需詳細資料，請參閱 `setup.sh` 檔案。
   {:tip}
3. 將動作連結到資料庫服務。請使用在環境設定期間建立的實例和服務金鑰。
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. 根據[警示套件](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm)建立觸發程式。它支援以不同形式指定警示。請使用類似 [cron](https://en.wikipedia.org/wiki/Cron) 的樣式。下面的觸發程式每天早上 6 點 (UTC) 觸發，從 4 月 21 日開始，到 12 月 21 日結束。請確保開始日期是未來的日期。
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  可以透過套用 `"0 6 * * 0"` 將觸發程式從每日排定變更為每週排定。這將使其在每周日早上 6 點發動。
  {:tip}
5. 最後，建立 **myStatsRule** 規則，用於將 **myDaily** 觸發程式連接至 **collectStats** 動作。現在，觸發程式會按先前步驟中指定的排定執行。
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. 呼叫用於起始測試執行的動作。傳回的 **repoCount** 應該會反映出先前配置的儲存庫數。
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
   輸出將類似於以下內容：
   ```
   {
       "repoCount": 18
   }
   ```
7. 在包含應用程式頁面的瀏覽器視窗中，現在可以造訪儲存庫資料流量。依預設，將顯示 10 個項目。可以將其變更為其他值。還可以對表格直欄進行排序，或使用搜尋方框來過濾特定儲存庫。可以輸入日期和組織名稱，然後依 viewcount 排序，列出特定日期得分最高者。
   ![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## 結論
在本指導教學中，您部署了無伺服器動作以及相關的觸發程式和規則。透過這些對象，可以自動擷取 GitHub 儲存庫的傳輸資料。有關這些儲存庫的資訊（包括租戶特定的存取記號）儲存在 SQL 資料庫 ({{site.data.keyword.dashdbshort}}) 中。Cloud Foundry 應用程式使用該資料庫來管理使用者和儲存庫，並在應用程式入口網站中顯示資料流量統計資料。使用者可以在可搜索的表格中查看資料流量統計資料，也可以在內嵌式儀表板（{{site.data.keyword.dynamdashbemb_short}} 服務，請參閱下圖）中顯示這些統計資料。還可以將儲存庫清單和資料流量資料下載為 CSV 檔案。

Cloud Foundry 應用程式透過連接至 {{site.data.keyword.appid_short}} 的 OpenID Connect 用戶端來管理存取。
![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## 清理
若要清除本指導教學使用的資源，可以按照與建立過程相反的順序來刪除相關服務和應用程式以及動作、觸發程式和規則：

1. 刪除 {{site.data.keyword.openwhisk_short}} 規則、觸發程式和動作。
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. 刪除 Python 應用程式及其服務。
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## 擴展指導教學
想要新增或變更此指導教學？以下是一些構想：
* 擴展應用程式以實現多方承租戶支援。
* 整合用於資料的圖表。
* 使用社交身分提供者。
* 將日期選取器新增到統計資料頁面以過濾顯示的資料。
* 將自訂登入頁面用於 {{site.data.keyword.appid_short}}。
* 使用 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 探索開發人員之間的社交編碼關係。

## 相關內容
以下是本指導教學所涵蓋主題的其他資訊鏈結。

文件及 SDK：
* [{{site.data.keyword.openwhisk_short}} 文件](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* 文件：[IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.appid_short}} 文件](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [IBM Cloud 上的 Python 運行環境](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
