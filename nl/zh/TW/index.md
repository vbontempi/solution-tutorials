---
copyright:
  years: 2017, 2018
lastupdated: "2019-05-06"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# 解決方案指導教學
{: #tutorials}

瞭解如何在 IBM Cloud 上建置、部署和調整現實世界解決方案。這些手冊就如何根據最佳作法和成熟技術使用 IBM Cloud 來實作常用模式，提供了逐步指示。
<style>
<!--
    #tutorials { /* hide the page header */
        display: none !important
    }
    p.last-updated { /* hide the last updated */
        display: none !important;
    }
    .doesNotExist, #doc-content, #single-content { /* use full width */
        width: calc(100% - 8%) !important;
        max-width: calc(100% - 8%) !important;
    }
    aside.side-nav, #topic-toc-wrapper { /* no need for side-nav */
        display: none !important;
    }
    .detailContentArea { /* use full width */
        max-width: 100% !important;
    }
    .allCategories {
        display: flex !important;
        flex-direction: row !important;
        flex-wrap: wrap !important;
    }
    .categoryBox {
        flex-grow: 1 !important;
        width: calc(33% - 20px) !important;
        text-decoration: none !important;
        margin: 0 10px 20px 0 !important;
        padding: 20px !important;
        border: 1px #dfe6eb solid !important;
        box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.2) !important;
        text-align: center !important;
        text-overflow: ellipsis !important;
        overflow: hidden !important;
    }
    .solutionBoxContainer {}
    .solutionBoxContainer a {
        text-decoration: none !important;
        border: none !important;
    }
    .solutionBox {
        display: inline-block !important;
        width: 100% !important;
        margin: 0 10px 20px 0 !important;
        padding: 10px !important;
        border: 1px #dfe6eb solid !important;
        box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.2) !important;
    }
    @media screen and (min-width: 960px) {
        .solutionBox {
        width: calc(50% - 3%) !important;
        }
        .solutionBox.solutionBoxFeatured {
        width: calc(50% - 3%) !important;
        }
        .solutionBoxContent {
        height: 270px !important;
        }
    }
    @media screen and (min-width: 1298px) {
        .solutionBox {
        width: calc(33% - 2%) !important;
        }
        .solutionBoxContent {
        min-height: 270px !important;
        }
    }
    .solutionBox:hover {
        border-color: rgb(136, 151, 162) !important;
    }
    .solutionBoxContent {
        display: flex !important;
        flex-direction: column !important;
    }
    .solutionBoxTitle {
        margin: 0rem !important;
        margin-bottom: 5px !important;
        font-size: 14px !important;
        font-weight: 700 !important;
        line-height: 16px !important;
        height: 37px !important;
        text-overflow: ellipsis !important;
        overflow: hidden !important;
        display: -webkit-box !important;
        -webkit-line-clamp: 2 !important;
        -webkit-box-orient: vertical !important;
    }
    .solutionBoxDescription {
        flex-grow: 1 !important;
        display: flex !important;
        flex-direction: column !important;
    }
    .descriptionContainer {
    }
    .descriptionContainer p {
        margin: 0 !important;
        overflow: hidden !important;
        display: -webkit-box !important;
        -webkit-line-clamp: 4 !important;
        -webkit-box-orient: vertical !important;
        font-size: 12px !important;
        font-weight: 400 !important;
        line-height: 1.5 !important;
        letter-spacing: 0 !important;
        max-height: 70px !important;
    }
    .architectureDiagramContainer {
        flex-grow: 1 !important;
        min-width: 250px !important;
        padding: 0 10px !important;
        text-align: center !important;
        display: flex !important;
        flex-direction: column !important;
        justify-content: center !important;
    }
    .architectureDiagram {
        max-height: 175px !important;
        padding: 5px !important;
        margin: 0 auto !important;
    }
-->
</style>

## 特色指導教學
<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                將端對端安全套用至雲端應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>建立安全的雲端應用程式，其特色是以您自己的金鑰加密資料、使用者鑑別及安全審核。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="「將端到端安全套用於雲端應用程式」解決方案的架構圖" />
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                隔離的 Cloud Foundry Enterprise 應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>藉由使用 Cloud Foundry Enterprise Environment，部署隔離的企業級 Cloud Foundry 平台，為您的組織提供創新平台。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="「隔離的 Cloud Foundry 企業用程序」解決方案的架構圖" />
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 網站和 Web 應用程式
{: #websites }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-scalable-webapp-kubernetes#scalable-webapp-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
Kubernetes 上的可擴充 Web 應用程式</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>搭建 Java Web 應用程式支架、在容器中本端執行該應用程式，然後將其部署到 Kubernetes 叢集。此外，連結自訂網域，監視環境的性能和進行調整。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution2/Architecture.png" alt="「Kubernetes 上可調整的 Web 應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                將以 VM 為基礎的應用程式移動到 Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>採用以 VM 為基礎的應用程式、對其進行容器化，然後將其部署到 Kubernetes 叢集。請將這些步驟用作其他應用程式的一般指引。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution30/modern_architecture.png" alt="「將以 VM 為基礎的應用程式移動到 Kubernetes 」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-strategies-for-resilient-applications#strategies-for-resilient-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                備援應用程式的策略
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>無論運算選項是什麼：Kubernetes、Cloud Foundry、Cloud Functions 或虛擬伺服器，企業都在尋求最短的關閉時間，並建立可實現最高可用性的備援架構。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution39/Architecture.png" alt="「備援應用程式的策略」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                持續部署到 Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>為 Kubernetes 中執行的容器化應用程式設定持續整合和交付管道。向安全掃描器、Slack 通知和分析等其他服務新增整合。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution21/Architecture.png" alt="「持續部署到 Kubernetes 」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Object Storage 和 CDN 加速交付靜態檔案
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在 Cloud Object Storage 中管理和提供網站資產（影像、視訊和文件）和使用者產生的內容，以及使用 Content Delivery Network (CDN) 快速、安全地向世界各地的使用者交付內容。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution3/Architecture.png" alt="「使用 Object Storage 和 CDN 加速交付靜態檔案」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-pub-sub-object-storage#pub-sub-object-storage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Object Storage 和發佈/訂閱傳訊進行非同步資料處理
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用以 Apache Kafka 為基礎的訊息中心來協調 Kubernetes 叢集裡執行的微服務之間的工作負載，並將資料儲存在 Object Storage 中。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution25/Architecture.png" alt="「使用 Object Storage 和發佈/訂閱傳訊進行非同步資料處理」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-lamp-stack#lamp-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                LAMP 堆疊上的 Web 應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>建立使用 Apache Web 伺服器、MySQL 和 PHP 的 Ubuntu Linux 虛擬伺服器。然後在 LAMP 堆疊上安裝並配置 WordPress 開放程式碼應用程式。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution4/Architecture.png" alt="「LAMP 堆疊上的 Web 應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Terraform 部署 LAMP 堆疊
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 Terraform 佈建使用 Apache Web 伺服器、MySQL、PHP 和 IBM Cloud Object Storage 服務的 Linux 虛擬伺服器。更新配置以調整資源並調整環境。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution10/architecture-2.png" alt="「使用 Terraform 部署 LAMP 堆疊」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                規劃、建立和更新部署環境
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 IBM Cloud CLI 和 Terraform 自動建立和維護多個部署環境。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution26-plan-create-update-deployments/architecture.png" alt="「規劃、建立和更新部署環境」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用虛擬伺服器建置具備高可用性和高可調整性的 Web 應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>建立一個負載平衡器、兩個應用程式伺服器（在安裝了 NGINX 和 PHP 的 Ubuntu 上執行）、一個 MySQL 資料庫伺服器以及一個持久檔案儲存空間（用於儲存空間應用程式檔案和備份）。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution14/Architecture.png" alt="「使用虛擬伺服器建置具備高可用性和高可調整性的 Web 應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-mean-stack#mean-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
使用 MEAN 堆疊的現代 Web 應用程式</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用常用的 MEAN 堆疊（Mongo DB、Express、Angular 和 Node.js）建置 Web 應用程式。在本端執行應用程式，建立和使用資料庫即服務，部署應用程式以及監視應用程式。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution7/Architecture.png" alt="「使用 MEAN 堆疊的現代 Web 應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-sql-database#sql-database">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                SQL Database for Cloud Data
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>佈建 SQL 關聯式資料庫服務，建立表格以及將大型資料集載入到資料庫中。部署 Web 應用程式，以利用這些資料並顯示如何存取雲端資料庫。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution5/Architecture.png" alt="SQL Database for Cloud Data 解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
無伺服器 Web 應用程式和 API</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>藉由在 GitHub Pages 上管理靜態網站內容，並使用 Cloud Functions 實作應用程式後端來建立無伺服器 Web 應用程式。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution8/Architecture.png" alt="「無伺服器 Web 應用程式和 API」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                跨多個地區部署無伺服器應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 Cloud Functions 和 Internet Services 建置全球可用的安全無伺服器應用程式。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution44-multi-region-serverless/Architecture.png" alt="「跨多個地區部署無伺服器應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 LogDNA 和 Sysdig 分析日誌並監視應用程式性能
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 IBM Log Analysis with LogDNA 瞭解應用程式活動並對其進行診斷。使用 IBM Cloud Monitoring with Sysdig 監視應用程式。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution12/Architecture.png" alt="「使用 LogDNA 和 Sysdig 分析日誌並監視應用程式性能」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                隔離的 Cloud Foundry Enterprise 應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>藉由使用 Cloud Foundry Enterprise Environment，部署隔離的企業級 Cloud Foundry 平台，為您的組織提供創新平台。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="「隔離的 Cloud Foundry Enterprise 應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 聊天機器人
{: #chatbots }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-slack-chatbot-database-watson#slack-chatbot-database-watson">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                建置資料庫驅動的 Slackbot
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 IBM Watson Assistant、Cloudant 和 IBM Cloud Functions 建置資料庫驅動的 Slackbot。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution19/SlackbotArchitecture.png" alt="「建置資料庫驅動的 Slackbot」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-watson-chatbot#android-watson-chatbot">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                建置具語音功能的 Android 聊天機器人
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>定義目的和實體，並為聊天機器人建置對話流程以回應客戶的查詢。啟用 Speech to Text 和 Text to Speech 服務以與 Android 應用程式進行輕鬆互動。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution28-watson-chatbot-android/architecture.png" alt="「建置具語音功能的 Android 聊天機器人」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 安全
{: #security }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                跨多個區域保護 Web 應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用持續交付管線跨多個地區對 Web 應用程式進行建立、保護和部署和負載平衡。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution1/Architecture.png" alt="「跨多個地區保護 Web 應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
具彈性且安全的多地區 Kubernetes 叢集</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>將 Cloud Internet Services 與 Kubernetes 叢集整合，以交付跨多個地區的富有彈性且安全的解決方案。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution32-multi-region-k8s-cis/Architecture.png" alt="「富有彈性且安全的多區域 Kubernetes 叢集」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                建立、保護和管理 REST API
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 LoopBack Node.js API 架構建立新的 REST API。使用 IBM Cloud 上的 API Connect 服務向 API 新增管理、可見性、安全和速率限制。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution13/Architecture.png" alt="「建立、保護和管理 REST API 」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                將端對端安全套用至雲端應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>建立安全的雲端應用程式，其特色是以您自己的金鑰加密資料、使用者鑑別及安全審核。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="「將端到端安全套用於雲端應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 行動
{: #mobile }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Push Notifications 的 iOS 行動應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在 IBM Cloud 上建立使用 Push Notifications 的 iOS Swift 應用程式。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution6/Architecture.png" alt="「使用 Push Notifications 的 iOS 行動應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Push Notifications 的 Android 原生行動應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在 IBM Cloud 上撰寫使用 Push Notifications 的 Android 原生應用程式。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution9/Architecture.png" alt="「使用 Push Notifications 的 Android 原生行動應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-hybrid-mobile-push-analytics#hybrid-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Push Notifications 的混合行動應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在 IBM Cloud 上開發使用 Push Notifications 的混合 Cordova 應用程式。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution15/Architecture.png" alt="「使用 Push Notifications 的混合行動應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-mobile-backend#serverless-mobile-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用無伺服器後端的行動應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>將 Cloud Functions 與認知和資料服務一起使用，為行動應用程式建置無伺服器後端。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution11/Architecture.png" alt="「使用無伺服器後端的行動應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 機器學習和分析
{: #ml }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用串流式分析和 SQL 的海量資料日誌
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>收集、儲存和分析日誌記錄，以滿足法規要求並輔助資訊探索。使用發佈/訂閱傳訊，將解決方案擴充到數百萬筆記錄，然後使用熟悉的 SQL 對持續保存的日誌執行分析。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution31/Architecture.png" alt="「使用串流式分析和 SQL 的海量資料日誌」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-smart-data-lake#smart-data-lake">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Object Storage 建置 Data Lake
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>為資料科學家提供多種工具，用於使用 SQL Query 查詢資料，並在 Watson Studio 中處理分析。透過互動式圖表分享資料和洞察。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution29/architecture.png" alt="「使用 Object Storage 建置 Data Lake」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-github-traffic-analytics#serverless-github-traffic-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                將無伺服器與 Cloud Foundry 結合用於資料擷取和分析
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>自動收集儲存庫的 GitHub 資料流量統計資料，將其儲存在 SQL 資料庫中，然後開始使用資料流量分析。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution24-github-traffic-analytics/Architecture.png" alt="「將無伺服器與 Cloud Foundry 結合用於資料擷取和分析」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#create-deploy-retrain-machine-learning-model">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
建置、部署、測試和重新訓練預測的機器學習模型。</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>建置預測性機器學習模型，將其部署為 API，測試模型並使用意見資料重新訓練模型。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution22-build-machine-learning-model/architecture_diagram.png" alt="「建置、部署、測試和重新訓練預測性機器學習模型」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-analytics-spark#big-data-analytics-spark">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Apache Spark 分析和視覺化開放資料
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 Jupyter Notebook 分析和視覺化開放資料集。將 Apache Spark 服務與 IBM Watson Studio 和 Pixiedust 配合使用以產生圖形。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution23/Architecture.png" alt="「使用 Apache Spark 分析和視覺化開放資料」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 物聯網
{: #iot }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#gather-visualize-analyze-iot-data">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                收集、視覺化和分析 IoT 資料
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>設定 IoT 裝置、在 Watson IoT 平台收集大數量資料、使用機器學習分析資料，然後建立視覺化。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution16/Architecture.png" alt="「收集、視覺化和分析 IoT 資料」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Identity and Access Management
{: #iam }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                關於組織使用者、團隊、應用程式的最佳作法
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>概要介紹 IBM Cloud 中提供的用於管理身分和存取權管理的概念，以及如何實作這些概念以支援應用程式的多個開發階段。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution20-users-teams-applications/architecture.png" alt="「關於組織使用者、團隊、應用程式的最佳作法」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-usage#cloud-usage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                檢閱 IBM Cloud 服務、資源和使用情況
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>介紹用於回答常見使用情況相關問題的各種方法。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution38/Architecture.png" alt="「檢閱 IBM Cloud 服務、資源和使用情況」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 網路
{: #Network }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Virtual Private Cloud 中的專用和公用子網路
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用子網路和實例建立虛擬專用雲端。藉由連接安全群組並且只容許最低存取權，來保護資源。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution40-vpc-public-app-private-backend/Architecture.png" alt="「Virtual Private Cloud 中的專用和公用子網路」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-multi-region#vpc-multi-region">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                跨多個位置和區域部署隔離工作負載
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在跨多個區域和地區的虛擬專用雲端中部署工作負載。使用本端和廣域負載平衡器在各區域之間分散資料流量。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution41-vpc-multi-region/Architecture.png" alt="「跨多個位置和區域部署隔離工作負載」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-site2site-vpn#vpc-site2site-vpn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 VPC/VPN 閘道對雲端資源進行安全的專用內部部署存取
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>透過安全的虛擬專用網路將虛擬專用雲端連接至其他運算環境，並使用 IBM Cloud 服務。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution46-vpc-vpn/ArchitectureDiagram.png" alt="「使用 VPC/VPN 閘道對雲端資源進行安全的專用內部部署存取」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server#vpc-secure-management-bastion-server">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用防禦主機安全地存取遠端實例
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>部署防禦主機以安全地存取虛擬專用雲端中的遠端實例。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png" alt="「使用防禦主機安全地存取遠端實例」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用安全專用網路隔離工作負載
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>配置虛擬路由器應用裝置以建立安全隔離區。關聯 VLAN、佈建伺服器、設定 IP 遞送和防火牆。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution33-secure-network-enclosure/Secure-priv-enc.png" alt="「使用安全專用網路隔離工作負載」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-nat-config-private#nat-config-private">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                配置 NAT 以從專用網路存取網際網路
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 NAT 假冒將專用 IP 位址轉換為出埠公用介面。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution35-nat-config-private/vra-nat.png" alt="「配置 NAT 以從專用網路存取網際網路」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-byoip#byoip">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
自帶 IP 位址</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>BYOIP 實作模式的概觀，以及識別適當模式的指引。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution37-byoip/byoipdecision.png" alt="「自帶 IP 位址」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                透過 VPN 連接到安全專用網路中
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在遠端網路環境和 IBM Cloud 專用網路上的伺服器之間建立專用連線。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png" alt="「透過 VPN 連接到安全專用網路中」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vlan-spanning#vlan-spanning">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                透過 IBM 網路鏈結安全專用網路
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>部署兩個專用網路，這兩個專用網路使用 VLAN Spanning 服務透過 IBM Cloud 專用網路進行安全鏈結。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution43-vlan-spanning/vlan-spanning.png" alt="「透過 IBM 網路鏈結安全專用網路」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-web-app-private-network#web-app-private-network">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                從安全專用網路提供服務的 Web 應用程式
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>建立一個可調整且安全的面向網際網路的 Web 應用程式，該應用程式管理在使用虛擬路由器應用裝置 (VRA)、VLAN、NAT 和防火牆進行保護的專用網路中。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution42-web-app-private-network/web-app-private.png" alt="「從安全專用網路提供服務的 Web 應用程式」解決方案的架構圖"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

