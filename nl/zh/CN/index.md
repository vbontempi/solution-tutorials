---
copyright:
  years: 2017, 2018
lastupdated: "2019-05-06"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# 解决方案教程
{: #tutorials}

了解如何在 IBM Cloud 上构建、部署和缩放现实世界解决方案。这些指南就如何根据最佳实践和成熟技术使用 IBM Cloud 来实施常用模式，提供了分步骤指示信息。
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

## 特色教程
<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                将端到端安全性应用于云应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>创建安全的云应用程序，其数据使用您自己的密钥加密，并通过用户认证和安全审计进行保护。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="“将端到端安全性应用于云应用程序”解决方案的体系结构图" />
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                隔离的 Cloud Foundry 企业应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>通过使用 Cloud Foundry Enterprise Environment 部署隔离的企业级 Cloud Foundry 平台，为组织提供创新平台。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="“隔离的 Cloud Foundry 企业用程序”解决方案的体系结构图" />
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Web 站点和 Web 应用程序
{: #websites }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-scalable-webapp-kubernetes#scalable-webapp-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Kubernetes 上可缩放的 Web 应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>搭建 Java Web 应用程序脚手架，在容器中本地运行该应用程序，然后将其部署到 Kubernetes 集群。此外，绑定定制域，监视环境的运行状况和进行缩放。
</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution2/Architecture.png" alt="“Kubernetes 上可缩放的 Web 应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                将基于 VM 的应用程序移动到 Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>采用基于 VM 的应用程序，对其进行容器化，然后将其部署到 Kubernetes 集群。请将这些步骤用作其他应用程序的一般指南。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution30/modern_architecture.png" alt="“将基于 VM 的应用程序移动到 Kubernetes”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-strategies-for-resilient-applications#strategies-for-resilient-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                弹性应用程序的策略
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>无论计算选项是什么：Kubernetes、Cloud Foundry、Cloud Functions 或虚拟服务器，企业都在寻求最大限度地减少停机时间，并创建可实现最大可用性的弹性体系结构。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution39/Architecture.png" alt="“弹性应用程序的策略”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                持续部署到 Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>为 Kubernetes 中运行的容器化应用程序设置持续集成和交付管道。向安全性扫描程序、Slack 通知和分析等其他服务添加集成。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution21/Architecture.png" alt="“持续部署到 Kubernetes”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Object Storage 和 CDN 加速交付静态文件
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在 Cloud Object Storage 中托管和提供 Web 站点资产（图像、视频和文档）和用户生成的内容，以及使用 Content Delivery Network (CDN) 快速、安全地向世界各地的用户交付内容。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution3/Architecture.png" alt="“使用 Object Storage 和 CDN 加速交付静态文件”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-pub-sub-object-storage#pub-sub-object-storage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Object Storage 和发布/预订消息传递进行异步数据处理
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用基于 Apache Kafka 的消息中心来协调在 Kubernetes 集群中运行的微服务之间的工作负载，并将数据存储在 Object Storage 中。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution25/Architecture.png" alt="“使用 Object Storage 和发布/预订消息传递进行异步数据处理”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-lamp-stack#lamp-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                LAMP 堆栈上的 Web 应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>创建使用 Apache Web 服务器、MySQL 和 PHP 的 Ubuntu Linux 虚拟服务器。然后在 LAMP 堆栈上安装并配置 WordPress 开放式源代码应用程序。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution4/Architecture.png" alt="“LAMP 堆栈上的 Web 应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Terraform 部署 LAMP 堆栈
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 Terraform 供应使用 Apache Web 服务器、MySQL、PHP 和 IBM Cloud Object Storage 服务的 Linux 虚拟服务器。更新配置以缩放资源并调整环境。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution10/architecture-2.png" alt="“使用 Terraform 部署 LAMP 堆栈”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                规划、创建和更新部署环境
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 IBM Cloud CLI 和 Terraform 自动创建和维护多个部署环境。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution26-plan-create-update-deployments/architecture.png" alt="“规划、创建和更新部署环境”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用虚拟服务器构建具备高可用性和高可缩放性的 Web 应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>创建一个负载均衡器、两个应用服务器（在安装了 NGINX 和 PHP 的 Ubuntu 上运行）、一个 MySQL 数据库服务器以及一个持久文件存储器（用于存储应用程序文件和备份）。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution14/Architecture.png" alt="“使用虚拟服务器构建具备高可用性和高可缩放性的 Web 应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-mean-stack#mean-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 MEAN 堆栈的现代 Web 应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用常用的 MEAN 堆栈（Mongo DB、Express、Angular 和 Node.js）构建 Web 应用程序。在本地运行应用程序，创建和使用数据库即服务，部署应用程序以及监视应用程序。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution7/Architecture.png" alt="“使用 MEAN 堆栈的现代 Web 应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-sql-database#sql-database">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                用于云数据的 SQL 数据库
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>供应 SQL 关系数据库服务，创建表以及将大型数据集装入到数据库中。部署 Web 应用程序，以利用这些数据并显示如何访问云数据库。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution5/Architecture.png" alt="“用于云数据的 SQL 数据库”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                无服务器 Web 应用程序和 API
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>通过在 GitHub Pages 上托管静态 Web 站点内容，并使用 Cloud Functions 实现应用程序后端来创建无服务器 Web 应用程序。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution8/Architecture.png" alt="“无服务器 Web 应用程序和 API”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                跨多个区域部署无服务器应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 Cloud Functions 和 Internet Services 构建全球可用的安全无服务器应用程序。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution44-multi-region-serverless/Architecture.png" alt="“跨多个区域部署无服务器应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 LogDNA 和 Sysdig 分析日志并监视应用程序运行状况
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 IBM Log Analysis with LogDNA 了解应用程序活动并对其进行诊断。使用 IBM Cloud Monitoring with Sysdig 监视应用程序。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution12/Architecture.png" alt="“使用 LogDNA 和 Sysdig 分析日志并监视应用程序运行状况”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                隔离的 Cloud Foundry 企业应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>通过使用 Cloud Foundry Enterprise Environment 部署隔离的企业级 Cloud Foundry 平台，为组织提供创新平台。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="“隔离的 Cloud Foundry 企业用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 聊天机器人
{: #chatbots }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-slack-chatbot-database-watson#slack-chatbot-database-watson">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                构建数据库驱动的 Slack 机器人
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 IBM Watson Assistant、Cloudant 和 IBM Cloud Functions 构建数据库驱动的 Slack 机器人。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution19/SlackbotArchitecture.png" alt="“构建数据库驱动的 Slack 机器人”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-watson-chatbot#android-watson-chatbot">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                构建支持声音的 Android 聊天机器人
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>定义意向和实体，并为聊天机器人构建对话流以响应客户的查询。启用 Speech to Text 和 Text to Speech 服务以与 Android 应用程序进行轻松交互。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution28-watson-chatbot-android/architecture.png" alt="“构建支持声音的 Android 聊天机器人”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 安全性
{: #security }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                跨多个区域保护 Web 应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用持续交付管道跨多个区域对 Web 应用程序进行创建、保护和部署和负载均衡。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution1/Architecture.png" alt="“跨多个区域保护 Web 应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                富有弹性且安全的多区域 Kubernetes 集群
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>将 Cloud Internet Services 与 Kubernetes 集群集成，以交付跨多个区域的富有弹性且安全的解决方案。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution32-multi-region-k8s-cis/Architecture.png" alt="“富有弹性且安全的多区域 Kubernetes 集群”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                创建、保护和管理 REST API
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 LoopBack Node.js API 框架创建新的 REST API。使用 IBM Cloud 上的 API Connect 服务向 API 添加管理、可视性、安全性和速率限制。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution13/Architecture.png" alt="“创建、保护和管理 REST API”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                将端到端安全性应用于云应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>创建安全的云应用程序，其数据使用您自己的密钥加密，并通过用户认证和安全审计进行保护。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="“将端到端安全性应用于云应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 移动
{: #mobile }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Push Notifications 的 iOS 移动应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在 IBM Cloud 上创建使用 Push Notifications 的 iOS Swift 应用程序。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution6/Architecture.png" alt="“使用 Push Notifications 的 iOS 移动应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Push Notifications 的 Android 本机移动应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在 IBM Cloud 上编写使用 Push Notifications 的 Android 本机应用程序。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution9/Architecture.png" alt="“使用 Push Notifications 的 Android 本机移动应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-hybrid-mobile-push-analytics#hybrid-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Push Notifications 的混合移动应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在 IBM Cloud 上开发使用 Push Notifications 的混合 Cordova 应用程序。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution15/Architecture.png" alt="“使用 Push Notifications 的混合移动应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-mobile-backend#serverless-mobile-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用无服务器后端的移动应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>将 Cloud Functions 与认知和数据服务一起使用，为移动应用程序构建无服务器后端。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution11/Architecture.png" alt="“使用无服务器后端的移动应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 机器学习和分析
{: #ml }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                通过流式分析和 SQL 使用大数据日志
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>收集、存储和分析日志记录，以满足法规要求并帮助发现信息。通过发布/预订消息传递，将解决方案扩展到数百万条记录，然后使用熟悉的 SQL 对持久存储的日志执行分析。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution31/Architecture.png" alt="“通过流式分析和 SQL 使用大数据日志”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-smart-data-lake#smart-data-lake">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Object Storage 构建数据湖
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>为数据研究员提供多种工具，用于通过 SQL Query 查询数据，并在 Watson Studio 中执行分析。通过交互式图表分享数据和洞察。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution29/architecture.png" alt="“使用 Object Storage 构建数据湖”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-github-traffic-analytics#serverless-github-traffic-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                将无服务器与 Cloud Foundry 组合用于数据检索和分析
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>自动收集存储库的 GitHub 流量统计信息，将其存储在 SQL 数据库中，然后开始使用流量分析。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution24-github-traffic-analytics/Architecture.png" alt="“将无服务器与 Cloud Foundry 组合用于数据检索和分析”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#create-deploy-retrain-machine-learning-model">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                构建、部署、测试和重新训练预测性机器学习模型
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>构建预测性机器学习模型，将其部署为 API，测试模型并使用反馈数据重新训练模型。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution22-build-machine-learning-model/architecture_diagram.png" alt="“构建、部署、测试和重新训练预测性机器学习模型”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-analytics-spark#big-data-analytics-spark">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 Apache Spark 分析和显示开放数据
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 Jupyter Notebook 分析和显示开放数据集。将 Apache Spark 服务与 IBM Watson Studio 和 Pixiedust 配合使用以生成图形。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution23/Architecture.png" alt="“使用 Apache Spark 分析和显示开放数据”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Internet of Things
{: #iot }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#gather-visualize-analyze-iot-data">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                收集、显示和分析 IoT 数据
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>设置 IoT 设备，在 Watson IoT Platform 中收集大量数据，使用机器学习分析数据，然后创建可视化。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution16/Architecture.png" alt="“收集、显示和分析 IoT 数据”解决方案的体系结构图"/>
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
                关于组织用户、团队、应用程序的最佳实践
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>概要介绍 IBM Cloud 中提供的用于管理身份和访问权管理的概念，以及如何实现这些概念以支持应用程序的多个开发阶段。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution20-users-teams-applications/architecture.png" alt="“关于组织用户、团队、应用程序的最佳实践”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-usage#cloud-usage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                查看 IBM Cloud 服务、资源和使用情况
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>介绍用于回答常见使用情况相关问题的各种方法。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution38/Architecture.png" alt="“查看 IBM Cloud 服务、资源和使用情况”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 网络
{: #Network }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Virtual Private Cloud 中的专用和公共子网
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用子网和实例创建虚拟私有云。通过附加安全组来保护资源，并且只允许最低访问权。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution40-vpc-public-app-private-backend/Architecture.png" alt="“Virtual Private Cloud 中的专用和公共子网”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-multi-region#vpc-multi-region">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                跨多个位置和专区部署隔离工作负载
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在跨多个专区和区域的虚拟私有云中部署工作负载。使用本地和全局负载均衡器在各专区之间分发流量。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution41-vpc-multi-region/Architecture.png" alt="“跨多个位置和专区部署隔离工作负载”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-site2site-vpn#vpc-site2site-vpn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用 VPC/VPN 网关对云资源进行安全的专用内部部署访问
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>通过安全的虚拟专用网将虚拟私有云连接到其他计算环境并使用 IBM Cloud 服务。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution46-vpc-vpn/ArchitectureDiagram.png" alt="“使用 VPC/VPN 网关对云资源进行安全的专用内部部署访问”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server#vpc-secure-management-bastion-server">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用防御主机安全地访问远程实例
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>部署防御主机以安全地访问虚拟私有云中的远程实例。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png" alt="“使用防御主机安全地访问远程实例”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                使用安全专用网络隔离工作负载
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>配置虚拟路由器设备以创建安全隔离区。关联 VLAN，供应服务器，设置 IP 路由和防火墙。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution33-secure-network-enclosure/Secure-priv-enc.png" alt="“使用安全专用网络隔离工作负载”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-nat-config-private#nat-config-private">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                配置 NAT 以从专用网络访问因特网
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用 NAT masquerade 将专用 IP 地址转换为出站公共接口。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution35-nat-config-private/vra-nat.png" alt="“配置 NAT 以从专用网络访问因特网”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-byoip#byoip">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                自带 IP 地址
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>概要介绍 BYOIP 实施模式并提供确定相应模式的指南。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution37-byoip/byoipdecision.png" alt="“自带 IP 地址”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                通过 VPN 连接到安全专用网络中
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>在远程网络环境和 IBM Cloud 专用网络上的服务器之间创建专用连接。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png" alt="“通过 VPN 连接到安全专用网络中”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vlan-spanning#vlan-spanning">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                通过 IBM 网络链接安全专用网络
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>部署两个专用网络，这两个专用网络使用 VLAN 生成服务通过 IBM Cloud 专用网络进行安全链接。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution43-vlan-spanning/vlan-spanning.png" alt="“通过 IBM 网络链接安全专用网络”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-web-app-private-network#web-app-private-network">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                从安全专用网络提供服务的 Web 应用程序
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>创建一个可缩放且安全的面向因特网的 Web 应用程序，该应用程序托管在通过虚拟路由器设备 (VRA)、VLAN、NAT 和防火墙进行保护的专用网络中。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution42-web-app-private-network/web-app-private.png" alt="“从安全专用网络提供服务的 Web 应用程序”解决方案的体系结构图"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

