---
copyright:
  years: 2017, 2018
lastupdated: "2019-05-06"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# 솔루션 튜토리얼
{: #tutorials}

IBM Cloud에서 실제 솔루션을 빌드하고, 배치하고, 스케일링하는 방법을 알아보십시오. 이러한 안내서는 우수 사례 및 검증된 기술을 기반으로 일반적인 패턴을 구현하기 위해 IBM Cloud를 사용하는 방법에 대한 단계별 지시사항을 제공합니다. 
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

## 제공되는 튜토리얼
<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                클라우드 애플리케이션에 엔드 투 엔드 보안 적용
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>사용자의 고유 키로 암호화된 데이터, 사용자 인증 및 보안 감사를 제공하는 안전한 클라우드 애플리케이션을 작성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="클라우드 애플리케이션에 엔드 투 엔드 보안 적용 솔루션에 대한 아키텍처 다이어그램" />
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                격리된 Cloud Foundry 엔터프라이즈 앱
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Foundry Enterprise Environment를 사용하여 격리된 엔터프라이즈용 Cloud Foundry 플랫폼을 배치함으로써 조직에 혁신 플랫폼을 제공합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="격리된 Cloud Foundry 엔터프라이즈 앱 솔루션에 대한 아키텍처 다이어그램" />
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 웹 사이트 및 웹 앱
{: #websites }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-scalable-webapp-kubernetes#scalable-webapp-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Kubernetes의 스케일링 가능한 웹 앱
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>스캐폴딩으로 Java 웹 애플리케이션을 작성하고, 컨테이너에서 로컬로 실행한 후 Kubernetes 클러스터에 배치합니다. 또한 사용자 정의 도메인을 바인드하고, 환경의 상태 및 범위를 모니터합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution2/Architecture.png" alt="Kubernetes의 스케일링 가능한 웹 앱 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                VM 기반 애플리케이션을 Kubernetes로 이동
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>VM 기반 애플리케이션을 컨테이너화한 후 Kubernetes 클러스터에 배치합니다. 이 튜토리얼의 단계를 다른 애플리케이션에 대한 일반적인 안내로 사용합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution30/modern_architecture.png" alt="VM 기반 애플리케이션을 Kubernetes로 이동 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-strategies-for-resilient-applications#strategies-for-resilient-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                복원성 애플리케이션에 대한 전략
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>컴퓨팅 옵션(Kubernetes, Cloud Foundry, Cloud Functions 또는 Virtual Server)에 관계없이, 엔터프라이즈에서는 가동 중단 시간을 최소화하고 최대의 가용성을 달성하는 복원성 아키텍처를 작성하는 것을 목표로 합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution39/Architecture.png" alt="복원성 애플리케이션을 위한 전략 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Kubernetes로의 지속적 배치
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Kubernetes 클러스터에서 실행되는 컨테이너화된 애플리케이션의 지속적 통합 및 지속적 전달 파이프라인을 설정합니다. 보안 스캐너, Slack 알림, 분석과 같은 기타 서비스와의 통합을 추가합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution21/Architecture.png" alt="Kubernetes로의 지속적 배치 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Object Storage 및 CDN을 사용한 정적 파일 전달 가속화
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Object Storage에서 웹 사이트 자산(이미지, 동영상, 문서) 및 사용자 생성 컨텐츠를 호스팅 및 서비스하고, 전 세계의 사용자에게 빠르고 안전하게 전달할 수 있도록 Content Delivery Network(CDN)를 사용합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution3/Architecture.png" alt="Object Storage 및 CDN을 사용한 정적 파일 전달 가속화 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-pub-sub-object-storage#pub-sub-object-storage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                오브젝트 스토리지 및 발행/구독 메시징을 사용하여 비동기 데이터 처리
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Apache Kafka 기반 Message Hub를 사용하여 Kubernetes 클러스터에서 실행 중인 마이크로서비스 간에 워크로드를 조정하고 데이터를 Object Storage에 저장합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution25/Architecture.png" alt="오브젝트 스토리지 및 발행/구독 메시징을 사용하여 비동기 데이터 처리 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-lamp-stack#lamp-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                LAMP 스택의 웹 애플리케이션
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Apache 웹 서버, MySQL 및 PHP를 포함하는 Ubuntu Linux Virtual Server를 작성합니다. 그 후 LAMP 스택에 WordPress 오픈 소스 애플리케이션을 설치하고 구성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution4/Architecture.png" alt="LAMP 스택의 웹 애플리케이션 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Terraform을 사용한 LAMP 스택 배치
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Terraform을 사용하여 Apache 웹 서버, MySQL, PHP 및 IBM Cloud Object Storage 서비스를 포함하는 Linux Virtual Server를 프로비저닝합니다. 구성을 업데이트하여 리소스를 스케일링하고 환경을 조정합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution10/architecture-2.png" alt="Terraform을 사용한 LAMP 스택 배치 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                배치 환경 계획, 작성 및 업데이트
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud CLI 및 Terraform을 사용하여 여러 배치 환경의 작성 및 유지보수를 자동화합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution26-plan-create-update-deployments/architecture.png" alt="배치 환경 계획, 작성 및 업데이트 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Virtual Server를 사용하여 스케일링 가능한 고가용성 웹 앱 빌드
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>하나의 Load Balancer, NGINX 및 PHP가 설치된 Ubuntu에서 실행되는 두 애플리케이션 서버, 하나의 MySQL 데이터베이스 서버, 그리고 애플리케이션 파일 및 백업을 저장할 내구성 있는 File Storage를 작성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution14/Architecture.png" alt="Virtual Server를 사용하여 스케일링 가능한 고가용성 웹 앱 빌드 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-mean-stack#mean-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                MEAN 스택을 사용하는 현대적 웹 애플리케이션
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>널리 사용되는 MEAN 스택인 Mongo DB, Express, Angular, Node.js를 사용하는 웹 애플리케이션을 빌드합니다. 앱을 로컬로 실행하고, DBaaS(Database-as-a-Service)를 작성 및 사용하고, 앱을 배치하고 모니터합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution7/Architecture.png" alt="MEAN 스택을 사용하는 현대적 웹 애플리케이션 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-sql-database#sql-database">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                클라우드 데이터를 위한 SQL 데이터베이스
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>SQL 관계형 데이터베이스 서비스를 프로비저닝하고, 테이블을 작성하고, 데이터베이스에 대형 데이터 세트를 로드합니다. 이러한 데이터를 사용하며 클라우드 데이터베이스에 액세스하는 방법을 보여주는 웹 앱을 배치합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution5/Architecture.png" alt="클라우드 데이터를 위한 SQL 데이터베이스 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                서버리스 웹 애플리케이션 및 API
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>GitHub Pages에서 정적 웹 사이트 컨텐츠를 호스팅하고 Cloud Functions를 사용해 웹 애플리케이션 백엔드를 구현하여 서버리스 웹 애플리케이션을 작성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution8/Architecture.png" alt="서버리스 웹 애플리케이션 및 API 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                여러 지역에 서버리스 앱 배치
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Functions 및 Internet Services를 사용하여 전 세계에서 사용 가능하며 안전한 서버리스 애플리케이션을 빌드합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution44-multi-region-serverless/Architecture.png" alt="여러 지역에 서버리스 앱 배치 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                LogDNA 및 Sysdig를 사용한 로그 분석 및 애플리케이션 상태 모니터링
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Log Analysis with LogDNA를 사용하여 애플리케이션 활동을 파악하고 진단합니다. IBM Cloud Monitoring with Sysdig를 사용하여 애플리케이션을 모니터합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution12/Architecture.png" alt="LogDNA 및 Sysdig를 사용한 로그 분석 및 애플리케이션 상태 모니터링 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                격리된 Cloud Foundry 엔터프라이즈 앱
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Foundry Enterprise Environment를 사용하여 격리된 엔터프라이즈용 Cloud Foundry 플랫폼을 배치함으로써 조직에 혁신 플랫폼을 제공합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="격리된 Cloud Foundry 엔터프라이즈 앱 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 챗봇
{: #chatbots }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-slack-chatbot-database-watson#slack-chatbot-database-watson">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                데이터베이스 중심의 Slackbot 빌드
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Watson Assistant, Cloudant 및 IBM Cloud Functions를 사용하여 데이터베이스 중심의 Slackbot을 빌드합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution19/SlackbotArchitecture.png" alt="데이터베이스 중심의 Slackbot 빌드 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-watson-chatbot#android-watson-chatbot">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                음성 인식 Android 챗봇 빌드
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>인텐트, 엔티티를 정의하고 고객의 질문에 응답하는 챗봇의 대화 플로우를 빌드합니다. Android 앱과의 손쉬운 상호작용을 위해 Speech to Text 및 Text to Speech 서비스를 사용합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution28-watson-chatbot-android/architecture.png" alt="음성 인식 Android 챗봇 빌드 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 보안
{: #security }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                여러 지역에서 웹 애플리케이션 보안
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>지속적 딜리버리 파이프라인을 사용하여 여러 지역에서 제공되는 웹 애플리케이션을 작성하고, 보안하고, 배치하고, 이에 대한 로드 밸런싱을 수행합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution1/Architecture.png" alt="여러 지역에서 제공되는 웹 애플리케이션 보안 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                안전한 복원성 다중 지역 Kubernetes 클러스터
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Internet Services를 Kubernetes 클러스터와 통합하여 여러 지역에 안전한 복원성 솔루션을 전달합니다.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution32-multi-region-k8s-cis/Architecture.png" alt="강한 복원력과 보안을 갖춘 다중 지역 Kubernetes 클러스터 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                REST API 작성, 보안 및 관리
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>LoopBack Node.js API 프레임워크를 사용하여 새 REST API를 작성합니다. IBM Cloud의 API Connect 서비스를 사용하여 API에 관리, 가시성, 보안 및 속도 제한 기능을 추가합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution13/Architecture.png" alt="AREST API 작성, 보안 및 관리 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                클라우드 애플리케이션에 엔드 투 엔드 보안 적용
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>사용자의 고유 키로 암호화된 데이터, 사용자 인증 및 보안 감사를 제공하는 안전한 클라우드 애플리케이션을 작성합니다.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="클라우드 애플리케이션에 엔드 투 엔드 보안 적용 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 모바일
{: #mobile }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                푸시 알림을 사용하는 iOS 모바일 앱
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud의 푸시 알림을 사용하는 iOS Swift 애플리케이션을 작성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution6/Architecture.png" alt="푸시 알림을 사용하는 iOS 모바일 앱 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                푸시 알림을 사용하는 Android 기반 모바일 앱
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud의 푸시 알림을 사용하는 Android 기반 애플리케이션을 작성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution9/Architecture.png" alt="푸시 알림을 사용하는 Android 기반 모바일 앱 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-hybrid-mobile-push-analytics#hybrid-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Push Notifications를 사용하는 하이브리드 모바일 애플리케이션
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud의 푸시 알림을 사용하는 하이브리드 Cordova 애플리케이션을 개발합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution15/Architecture.png" alt="푸시 알림을 사용하는 하이브리드 모바일 앱 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-mobile-backend#serverless-mobile-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                서버리스 백엔드를 사용하는 모바일 애플리케이션
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Functions를 코그너티브 및 데이터 서비스와 함께 사용하여 모바일 애플리케이션을 위한 서버리스 백엔드를 빌드합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution11/Architecture.png" alt="서버리스 백엔드를 사용하는 모바일 애플리케이션 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 기계 학습 및 분석
{: #ml }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                스트리밍 분석 및 SQL을 사용하는 빅데이터 로그
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>규제 요구사항을 지원하고 정보 발견을 돕기 위해 로그 레코드를 수집하고, 저장하고 분석합니다. 발행/구독 메시징을 사용하여 솔루션을 수백만 개의 레코드로 스케일링한 후 익숙한 SQL을 사용하여 보존된 로그를 분석합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution31/Architecture.png" alt="스트리밍 분석 및 SQL을 사용하는 빅데이터 로그 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-smart-data-lake#smart-data-lake">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Object Storage를 사용한 데이터 레이크 빌드
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>데이터 과학자에게 SQL Query를 사용하여 데이터를 조회하고 Watson Studio에서 분석을 수행하는 도구를 제공합니다. 대화식 차트를 통해 데이터와 인사이트를 공유합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution29/architecture.png" alt="Object Storage를 사용한 데이터 레이크 빌드 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-github-traffic-analytics#serverless-github-traffic-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                데이터 검색 및 분석에서 서버리스와 Cloud Foundry 결합
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>저장소에 대한 GitHub 트래픽 통계를 자동으로 수집하고, 이를 SQL 데이터베이스에 저장한 후 트래픽 분석을 시작합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution24-github-traffic-analytics/Architecture.png" alt="데이터 검색 및 분석에서 서버리스와 Cloud Foundry 결합 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#create-deploy-retrain-machine-learning-model">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                예측 기계 학습 모델 빌드, 배치, 테스트 및 재훈련
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>예측 기계 학습 모델을 빌드하고 API로 배치한 후, 피드백 데이터로 모델을 테스트하고 재훈련시키십시오. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution22-build-machine-learning-model/architecture_diagram.png" alt="예측 기계 학습 모델 빌드, 배치, 테스트 및 재훈련 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-analytics-spark#big-data-analytics-spark">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Apache Spark를 사용한 오픈 데이터 분석 및 시각화
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Jupyter Notebook을 사용하여 오픈 데이터 세트를 분석하고 시각화합니다. Apache Spark 서비스를 IBM Watson Studio 및 Pixiedust와 사용하여 그래픽을 생성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution23/Architecture.png" alt="Apache Spark를 사용한 오픈 데이터 분석 및 시각화 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## IoT(Internet of Things)
{: #iot }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#gather-visualize-analyze-iot-data">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                IoT 데이터 수집, 시각화 및 분석
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IoT 디바이스를 설정하고, Watson IoT Platform에서 대량의 데이터를 수집하고, 기계 학습을 사용하여 데이터를 분석하고, 시각화를 작성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution16/Architecture.png" alt="IoT 데이터 수집, 시각화 및 분석 솔루션에 대한 아키텍처 다이어그램"/>
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
                사용자, 팀 및 애플리케이션 구성에 대한 우수 사례
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud에서 ID 및 액세스 관리를 수행하는 데 사용할 수 있는 개념과 애플리케이션의 여러 개발 단계를 지원하기 위해 이를 구현하는 방법에 대한 개요입니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution20-users-teams-applications/architecture.png" alt="사용자, 팀 및 애플리케이션 구성에 대한 우수 사례 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-usage#cloud-usage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                IBM Cloud 서비스, 리소스 및 사용량 검토
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>일반적인 사용 관련 질문에 답변하는 데 사용되는 다양한 접근법에 대한 소개입니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution38/Architecture.png" alt="IBM Cloud 서비스, 리소스 및 사용 검토 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 네트워크
{: #Network }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Virtual Private Cloud의 공용 및 사설 서브넷
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>서브넷 및 인스턴스를 사용하여 Virtual Private Cloud를 작성합니다. 보안 그룹을 연결하여 리소스를 보안하고 최소한의 액세스만 허용합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution40-vpc-public-app-private-backend/Architecture.png" alt="Virtual Private Cloud의 공용 및 사설 서브넷 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-multi-region#vpc-multi-region">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                여러 지역 및 구역에 격리된 워크로드 배치
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>여러 구역 및 지역의 Virtual Private Cloud에 워크로드를 배치합니다. 로컬 및 글로벌 로드 밸런서를 사용하여 구역 간에 트래픽을 분배합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution41-vpc-multi-region/Architecture.png" alt="여러 지역 및 구역에 격리된 워크로드 배치 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-site2site-vpn#vpc-site2site-vpn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                클라우드 리소스에 대한 안전한 개인용 온프레미스 액세스를 위해 VPC/VPN 게이트웨이 사용
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>안전한 가상 사설망(VPN)을 통해 Virtual Private Cloud를 다른 컴퓨팅 환경에 연결하여 IBM Cloud 서비스를 이용합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution46-vpc-vpn/ArchitectureDiagram.png" alt="클라우드 리소스에 대한 안전한 개인용 온프레미스 액세스를 위해 VPC/VPN 게이트웨이 사용 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server#vpc-secure-management-bastion-server">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                배스천 호스트로 원격 인스턴스에 안전하게 액세스
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>배스천 호스트를 배치하여 Virtual Private Cloud 내 원격 인스턴스에 안전하게 액세스합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png" alt="배스천 호스트로 원격 인스턴스에 안전하게 액세스 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                보안 사설 네트워크를 사용하여 워크로드 격리
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Virtual Router Appliance를 구성하여 안전한 격납장치를 작성합니다. VLAN을 연관시키고, 서버를 프로비저닝하고, IP 라우팅 및 방화벽을 설정합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution33-secure-network-enclosure/Secure-priv-enc.png" alt="안전한 사설 네트워크를 사용한 워크로드 격리 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-nat-config-private#nat-config-private">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                사설 너트워크에서의 인터넷 액세스를 위한 NAT 구성
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>NAT 위장을 사용하여 사설 IP 주소를 아웃바운드 공용 인터페이스로 변환합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution35-nat-config-private/vra-nat.png" alt="사설 너트워크에서의 인터넷 액세스를 위한 NAT 구성 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-byoip#byoip">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                BYOIP(Bring Your Own IP Address)
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>BYOIP 구현 패턴에 대한 개용와 적절한 패턴을 식별하는 방법에 대한 안내입니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution37-byoip/byoipdecision.png" alt="BYOIP(Bring Your Own IP Address) 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                안전한 사설 네트워크로의 VPN
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>원격 네트워크 환경과 IBM Cloud의 사설 네트워크 간에 개인용 연결을 작성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png" alt="안전한 사설 네트워크로의 VPN 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vlan-spanning#vlan-spanning">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                IBM 네트워크를 통한, 안전한 사설 네트워크 간의 연결
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>VLAN Spanning 서비스를 사용하여 IBM Cloud 사설 네트워크를 통해 안전하게 연결된 두 사설 네트워크를 배치합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution43-vlan-spanning/vlan-spanning.png" alt="IBM 네트워크를 통한, 안전한 사설 네트워크 간의 연결 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-web-app-private-network#web-app-private-network">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                안전한 사설 네트워크로부터 서비스되는 웹 애플리케이션
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Virtual Router Appliance(VRA), VLAN, NAT 및 방화벽을 사용하여 보안되는 사설 네트워크에서 호스팅되는, 스케일링 가능하며 안전한 인터넷 연결 웹 애플리케이션을 작성합니다. </p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution42-web-app-private-network/web-app-private.png" alt="안전한 사설 네트워크로부터 서비스되는 웹 애플리케이션 솔루션에 대한 아키텍처 다이어그램"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

