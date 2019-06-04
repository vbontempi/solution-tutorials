---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# 持續部署至 Kubernetes
{: #continuous-deployment-to-kubernetes}

本指導教學會引導您為在 {{site.data.keyword.containershort_notm}} 上執行的容器化應用程式設定持續整合和交付管線的處理程序。您將瞭解如何設定來源控制，並建置和測試程式碼，然後將程式碼部署到不同的部署階段。接下來，將向安全掃描器、Slack 通知和分析這類其他服務新增整合。

{:shortdesc}

## 目標
{: #objectives}

* 建立開發和正式作業 Kubernetes 叢集。
* 建立入門範本應用程式，在本端執行並將其推送到 Git 儲存庫。
* 配置 DevOps 交付管線，以連接至 Git 儲存庫，並建置入門範本應用程式，然後將其部署到開發/正式作業叢集。
* 探索並整合應用程式，以使用安全掃描器、Slack 通知和分析。

## 使用的服務
{: #services}

本指導教學使用下列 {{site.data.keyword.Bluemix_notm}} 服務：

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**注意：**本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

![](images/solution21/Architecture.png)

1. 將程式碼推送到專用 Git 儲存庫。
2. 管線會反映 Git 中的變更，並建置容器映像檔。
3. 將上傳到登錄的容器映像檔部署到開發 Kubernetes 叢集。
4. 驗證變更並部署到正式作業叢集。
5. 設定有關部署活動的 Slack 通知。


## 開始之前
{: #prereq}

* [安裝 {{site.data.keyword.dev_cli_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - 使用 Script 安裝 docker、kubectl、helm、ibmcloud cli 和必要的外掛程式。
* [設定 {{site.data.keyword.registrylong_notm}} CLI 及登錄名稱空間](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)。
* [瞭解 Kubernetes 的基本觀念](https://kubernetes.io/docs/tutorials/kubernetes-basics/)。

## 建立開發 Kubernetes 叢集
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} 藉由結合 Docker 和 Kubernetes 技術、直覺式使用者體驗以及內建安全和隔離來提供功能強大的工具，以在運算主機的叢集中自動部署、操作、擴充及監視容器化應用程式。


若要完成本指導教學，需要選取**標準**類型的**付費**叢集。您需要設定兩個叢集，一個用於開發，一個用於正式作業。
{: shortdesc}

1. 從 [{{site.data.keyword.Bluemix}} 型錄](https://{DomainName}/containers-kubernetes/launch)建立第一個開發 Kubernetes 叢集。稍後需要重複這些步驟並建立正式作業叢集。

   為了方便使用，請檢查配置詳細資料，例如 CPU 數目、記憶體和取得的工作者節點數目。
   {:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. 選取**叢集類型**，然後按一下**建立叢集**來佈建 Kubernetes 叢集。對於本指導教學，選擇具有 2 個 **CPU** 和 4 GB **RAM** 的最小**機型**以及 1 個**工作者節點**足以滿足需求。其他所有選項都可以保留其預設值。
3. 檢查**叢集**和**工作者節點**的狀態，並等待它們成為**就緒**。

**附註：**不要在工作者節點準備就緒之前繼續作業。

## 建立入門範本應用程式
{: #create_application}

{{site.data.keyword.containershort_notm}} 提供一組精選的入門範本應用程式，可以使用 `ibmcloud dev create` 指令或 Web 主控台來建立這些入門範本應用程式。在本指導教學中將使用 Web 主控台。藉由產生具有所有必要樣板、建置和配置程式碼的應用程式入門範本，大幅縮短入門範本應用程式的開發時間，從而可以更快開始對商業邏輯進行編碼。

1. 在 [{{site.data.keyword.cloud_notm}} 主控台](https://{DomainName})中，使用左側的功能表選項，並選取 [Web 應用程式](https://{DomainName}/developer/appservice/dashboard)。
2. 在**從 Web 開始**區段下，按一下**開始使用**按鈕。
3. 選取`搭配 Express.js 的 Node.js Web 應用程式`磚，然後選取`建立應用程式`以建立 Node.js 入門範本應用程式。
4. 輸入**名稱** `mynodestarter`。然後，按一下**建立**。

## 配置 DevOps 交付管線
{: #create_devops}

1. 既然您已順利建立入門範本應用程式，請在**部署應用程式**下，按一下**部署到雲端**按鈕。
2. 選取 Kubernetes 叢集部署方法，並選取稍早建立的叢集，然後按一下**建立**。這將建立工具鏈和交付管線。![](images/solution21/BindCluster.png)
3. 建立管線後，按一下**檢視工具鏈**，然後按一下**交付管線**以檢視管線。![](images/solution21/Delivery-pipeline.png)
4. 部署階段完成後，按一下**檢視日誌和歷程**以檢視日誌。
5. 存取顯示的 URL 以存取應用程式 (`http://worker-public-ip:portnumber/`)。 ![](images/solution21/Logs.png)
完成，您已使用 App Service 使用者介面建立入門範本應用程式，並配置管線用於建置應用程式並將其部署到叢集。

## 複製應用程式並在本端建置和執行應用程式
{: #cloneandbuildapp}

在本節中，您將使用稍早章節中建立的入門範本應用程式，並將其複製到本端機器，再修改程式碼，然後在本端建置/執行該應用程式。
{: shortdesc}

### 複製應用程式
1. 在「工具鏈概觀」中，選取**程式碼**下的 **Git** 磚。這會將您重新導向到 Git 儲存庫頁面，在其中可以複製儲存庫。![HelloWorld](images/solution21/DevOps_Toolchain.png)

2. 如果您尚未設定 SSH 金鑰，應該會在頂端看到含有指示的通知列。請在新的分頁開啟**新增 SSH 金鑰**鏈結並遵循步驟，或是如果您想要改用 HTTPS 而非 SSH 的話，請按一下**建立個人存取記號**並遵循步驟。請記得儲存金鑰或記號，以便未來參考。
3. 選取 SSH 或 HTTPS 並複製 Git URL。將原始檔複製到您的本端機器。如果提示您輸入使用者名稱，請提供您的 Git 使用者名稱。針對密碼，請使用現有的 **SSH 金鑰**或**個人存取記號**，或是您在先前步驟中建立的 SSH 金鑰或個人存取記號。

   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   {: codeblock}

4. 在您選擇的 IDE 中開啟複製的儲存庫，並導覽至 `public/index.html`。藉由嘗試將 "Congratulations!" 變更為其他內容來更新程式碼，然後儲存該檔案。

### 本端建置應用程式
您可以如常建置並執行應用程式，使用 `mvn` 進行 java 本端開發或使用 `npm` 進行 node 開發。您也可以建置 Docker 映像檔並在容器中執行應用程式，以確定在本端和在雲端能一致地執行。請使用下列步驟來建置您的 Docker 映像檔。
{: shortdesc}

1. 確保本端 Docker 引擎已啟動；若要進行檢查，請執行以下指令：
   ```
   docker ps
   ```
   {: codeblock}
2. 導覽至產生的複製專案目錄。
   ```
   cd <project name>
   ```
   {: codeblock}
3. 在本端建置應用程式。
   ```
  ibmcloud dev build
  ```
   {: codeblock}

   這可能會需要幾分鐘來執行，因為會下載所有應用程式相依關係，且會建置 Docker 映像檔，其中包含您的應用程式和所有必要環境。

### 在本端執行應用程式

1. 執行容器。
   ```
  ibmcloud dev run
  ```
   {: codeblock}

   這會使用您的本端 Docker 引擎，來執行您在先前步驟中建置的 Docker 映像檔。
2. 容器啟動後，移至 http://localhost:3000/
   ![](images/solution21/node_starter_localhost.png)

## 將應用程式推送到 Git 儲存庫

在本節中，您要將變更提交到 Git 儲存庫。管線將選取該提交，並自動將變更推送到叢集。
1. 在終端機視窗中，確保位於複製的儲存庫中。
2. 以三個簡單的步驟將變更推送至儲存庫：新增、確定、推送。
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
   {: codeblock}

3. 移至您先前建立的工具鏈，然後按一下 **Delivery Pipeline** 磚。
4. 確認您看到**建置**及**部署**階段。
   ![](images/solution21/Delivery-pipeline.png)
5. 等待**部署**階段完成。
6. 在前次執行結果下，按一下應用程式 **url** 以即時檢視變更。

如果您看不到應用程式更新，請檢查管線的部署及建置階段的日誌。

## 使用 Vulnerability Advisor 的安全
{: #vulnerability_advisor}

在此步驟中，您將探索 [Vulnerability Advisor](https://{DomainName}/docs/services/va?topic=va-va_index#va_index)。Vulnerability Advisor 用於在部署之前檢查容器映像檔的安全狀態，另外還會檢查執行中容器的狀態。

1. 移至您先前建立的工具鏈，然後按一下 **Delivery Pipeline** 磚。
1. 按一下**新增階段**，並將 MyStage 變更為**驗證階段**，然後按一下「工作」> **新增工作**。

   1. 選取**測試**作為「工作類型」，然後將方框中的**測試**變更為 **Vulnerability Advisor**。
   1. 在「測試類型」下，選取 **Vulnerability Advisor**。應該會自動移入其他所有欄位。容器登錄名稱空間應該與此工具鏈的**建置階段**中提及的名稱空間相同。
      {:tip}
   1. 編輯**測試 Script** 區段，並將最後一行中的 `SAFE\ to\ deploy` 取代為 `NO\ ISSUES`
   1. 儲存該階段
1. 將**驗證階段**拖曳並移至中間位置，然後在**驗證階段**上按一下**執行** ![](images/solution21/run.png)。您將看到**驗證階段**失敗。

   ![](images/solution21/toolchain.png)

1. 按一下**檢視日誌和歷程**來檢視漏洞評估。日誌的結尾顯示：

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   可以在[這裡](https://{DomainName}/containers-kubernetes/registry/private)查看所有已掃描儲存庫的詳細漏洞評量
   {:tip}

   如果掃描漏洞所花的時間超過 3 分鐘，階段可能會失敗，並指出該映像檔*尚未掃描*。藉由編輯工作 Script 並增加等待掃描結果的反覆運算次數，可以變更此逾時。
   {:tip}

1. 遵循更正動作，以修正漏洞。在 IDE 中開啟複製的儲存庫，或選取 Eclipse Orion Web IDE 磚，並開啟 `Dockerfile`，然後在 `EXPOSE 3000` 後面新增以下指令：
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. 提交並推送變更。這應該會觸發工具鏈，並修正**驗證階段**。

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## 建立正式作業 Kubernetes 叢集

{: #deploytoproduction}

在本節中，您將藉由將 Kubernetes 應用程式分別部署到開發和正式作業環境來完成部署管線。理想情況下，我們希望為開發環境設定自動部署，而為正式作業環境設定手動部署。在此之前，我們先來探索可以交付此類部署的兩種方式。可以將一個叢集同時用於開發和正式作業環境。但是，建議採用兩個個別的叢集，一個用於開發，一個用於正式作業。下面將探索為正式作業設定第二個叢集。
{: shortdesc}

1. 遵循[建立開發 Kubernetes 叢集](#create_kube_cluster)小節中的指示來建立新叢集。將此叢集命名為 `prod-cluster`。
2. 移至您先前建立的工具鏈，然後按一下 **Delivery Pipeline** 磚。
3. 將**部署階段**重新命名為 `Deploy dev`，這可以藉由按一下「設定」圖示 > **配置階段**來執行。
   ![](images/solution21/deploy_stage.png)
4. 複製 **Deploy dev** 階段（「設定」圖示 >「複製階段」），然後將複製的階段命名為 `Deploy prod`。
5. 將**階段觸發程式**變更為`僅當手動執行此階段時才執行工作`。 ![](images/solution21/prod-stage.png)
6. 在**工作**標籤下，將叢集名稱變更為新建立的叢集，然後**儲存**該階段。
7. 您現在應該擁有完整的部署設定，若要從開發環境部署到正式作業環境，必須手動執行 `Deploy prod` 階段以部署到正式作業環境。![](images/solution21/full-deploy.png)

完成，現在您已建立正式作業叢集，並將管線配置為手動將更新推送到正式作業叢集。這是一個簡化處理程序階段，在更進階的情境中，您會將單元測試和整合測試包含為管線的一部分。

## 設定 Slack 通知
{: #setup_slack}

1. 返回以檢視[工具鏈](https://{DomainName}/devops/toolchains)清單，並選取您的工具鏈，然後按一下**新增工具**。
2. 在搜尋方框中搜尋 Slack，或向下捲動以顯示 **Slack**。按一下以查看配置頁面。
    ![](images/solution21/configure_slack.png)
3. 對於 **Slack Webhook**，請執行此[鏈結](https://my.slack.com/services/new/incoming-Webhook/)中的步驟。您需要使用 Slack 認證進行登入，並提供現有頻道名稱或建立新頻道名稱。
4. 新增「送入 Webhook 整合」後，複製該 **Webhook URL**，並將其貼入 **Slack Webhook** 下。
5. Slack 頻道是在上面建立 Webhook 整合時提供的頻道名稱。
6. **Slack 團隊名稱**是 team-name.slack.com 中的 team-name（第一部分）。例如，在 kube.slack.com 中，kube 是團隊名稱。
7. 按一下**建立整合**。新的磚將新增到您的工具鏈中。
    ![](images/solution21/toolchain_slack.png)
8. 從現在開始，每當您的工具鏈執行時，您都應該會在配置的頻道中看到 Slack 通知。
    ![](images/solution21/slack_channel.png)

## 移除資源
{: #removeresources}

在此步驟中，您將清除資源，以移除以上建立的內容。

- 刪除 Git 儲存庫。
- 刪除工具鏈。
- 刪除兩個叢集。
- 刪除 Slack 頻道。

## 擴展指導教學
{: #expandTutorial}

想要進一步瞭解嗎？下面是您接下來可以執行的作業的一些構想：

- [使用 LogDNA 及 Sysdig分析日誌並監視應用程式性能](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)。
- 新增測試環境並將其部署到第三個叢集。
- [在多個位置](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)部署正式作業叢集。
- 使用利用 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 的其他品質控制和分析功能來增強管線。

## 相關內容
{: #related}

* 端到端 Kubernetes 解決方案手冊：[將以 VM 為基礎的應用程式移至 Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes)。
* IBM Cloud Container Service [安全](https://{DomainName}/docs/containers?topic=containers-security#cluster)。
* 工具鏈[整合](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations)。
* 使用 [LogDNA 和 Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis) 分析日誌並監視應用程式性能。


