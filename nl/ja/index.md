---
copyright:
  years: 2017, 2018
lastupdated: "2019-05-06"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# ソリューションのチュートリアル
{: #tutorials}

IBM Cloud 上で実際にソリューションをビルド、デプロイ、スケーリングする方法について説明します。以下のガイドでは、IBM Cloud を使用して、ベスト・プラクティスと実績あるテクノロジーに基づいた一般的なパターンを実装する方法について段階的に説明しています。
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

## 主要なチュートリアル
<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
クラウド・アプリケーションへのエンドツーエンドのセキュリティーの適用</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>独自の鍵によるデータ暗号化、ユーザー認証、セキュリティー監査の機能を備えたセキュアなクラウド・アプリケーションを作成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="クラウド・アプリケーションへのエンドツーエンド・セキュリティーの適用というソリューションを表すアーキテクチャー図" />
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                分離された Cloud Foundry エンタープライズ・アプリ
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Foundry エンタープライズ環境を使用して、分離されたエンタープライズ・グレードの Cloud Foundry プラットフォームをデプロイすることにより、組織にイノベーション・プラットフォームを提供します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="分離された Cloud Foundry エンタープライズ・アプリというソリューションを表すアーキテクチャー図" />
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Web サイトと Web アプリ
{: #websites }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-scalable-webapp-kubernetes#scalable-webapp-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Kubernetes のスケーラブル Web アプリケーション
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Java Web アプリケーションを構築し、ローカル環境のコンテナー内で実行してから、Kubernetes クラスターにデプロイします。また、カスタム・ドメインをバインドし、環境の正常性をモニターしてスケーリングします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution2/Architecture.png" alt="Kubernetes でのスケーラブルな Web アプリというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
VM ベース・アプリケーションの Kubernetes への移行</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>VM ベース・アプリケーションを取得し、コンテナー化して、Kubernetes クラスターにデプロイします。 他のアプリケーションでも、一般的なガイドとしてこの手順を使用できます。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution30/modern_architecture.png" alt="VM ベース・アプリケーションの Kubernetes への移動というソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-strategies-for-resilient-applications#strategies-for-resilient-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                耐障害性のあるアプリケーションのための戦略
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>コンピュート・オプション (Kubernetes、Cloud Foundry、Cloud Functions、または仮想サーバー) に関係なく、エンタープライズでは、ダウン時間の最小化および可用性を最大限に高める耐障害性のあるアーキテクチャーの作成が追求されます。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution39/Architecture.png" alt="耐障害性のあるアプリケーションのための戦略というソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Kubernetes への継続的デプロイメント
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Kubernetes クラスターで実行されているコンテナー化されたアプリケーションに対する継続的な統合およびデリバリー・パイプラインをセットアップします。 セキュリティー・スキャナー、Slack 通知、分析などの他のサービスとの統合を追加します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution21/Architecture.png" alt="Kubernetes への継続的デプロイメントというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                オブジェクト・ストレージと CDN を使用した静的ファイル・デリバリーの加速
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Web サイト資産 (イメージ、ビデオ、文書) およびユーザーによって生成されたコンテンツを Cloud Object Storage でホスティングおよび提供し、世界中のユーザーに迅速かつ安全に配信するためにコンテンツ配信ネットワーク (CDN) を使用します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution3/Architecture.png" alt="オブジェクト・ストレージと CDN を使用した静的ファイル・デリバリーの加速というソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-pub-sub-object-storage#pub-sub-object-storage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                オブジェクト・ストレージとパブリッシュ/サブスクライブのメッセージングによる非同期データ処理
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Apache Kafka ベースのメッセージ・ハブを使用して、Kubernetes クラスター内で実行するマイクロサービス間でワークロードを調整し、オブジェクト・ストレージにデータを保管します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution25/Architecture.png" alt="オブジェクト・ストレージとパブリッシュ/サブスクライブのメッセージングによる非同期データ処理のソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-lamp-stack#lamp-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                LAMP スタックの Web アプリケーション
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Apache Web サーバー、MySQL、および PHP で Ubuntu Linux 仮想サーバーを作成します。 その後、WordPress オープン・ソース・アプリケーションを LAMP スタック上にインストールおよび構成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution4/Architecture.png" alt="LAMP スタックの Web アプリケーションというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Terraform を使用した LAMP スタックのデプロイ
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Terraform を使用して、Apache Web サーバー、MySQL、PHP、IBM Cloud オブジェクト・ストレージ・サービスを装備した Linux 仮想サーバーをプロビジョンします。 構成を更新して、リソースを拡張したり環境を調整したりします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution10/architecture-2.png" alt="Terraform を使用した LAMP スタックのデプロイのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                デプロイメント環境の計画、作成、および更新
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud CLI と Terraform によって、複数のデプロイメント環境の作成と保守を自動化します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution26-plan-create-update-deployments/architecture.png" alt="デプロイメント環境の計画、作成、および更新というソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                仮想サーバーを使用してスケーラブルな高可用性 Web アプリを作成する
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>ロード・バランサー、NGINX と PHP がインストールされた Ubuntu で実行される 2 つのアプリケーション・サーバー、1 つの MySQL データベース・サーバー、およびアプリケーション・ファイルとバックアップを保管するための永続ファイル・ストレージを作成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution14/Architecture.png" alt="仮想サーバーを使用してスケーラブルな高可用性 Web アプリを作成するソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-mean-stack#mean-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                MEAN スタックを使用した最新の Web アプリケーション
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>一般的な MEAN スタック (Mongo DB、Express、Angular、Node.js) を使用して Web アプリケーションを作成します。ローカル環境でアプリを実行し、Database as a Service を作成して使用し、アプリをデプロイしてモニターします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution7/Architecture.png" alt="MEAN スタックを使用した最新の Web アプリケーションというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-sql-database#sql-database">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                クラウド・データ用の SQL データベース
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>SQL リレーショナル・データベース・サービスをプロビジョンし、テーブルを作成して、大規模なデータ・セットをデータベースにロードします。 このデータを活用するために Web アプリをデプロイし、クラウド・データベースにアクセスする方法を示します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution5/Architecture.png" alt="クラウド・データ用の SQL データベースというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                サーバーレス Web アプリケーションと API
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>GitHub ページで静的 Web サイト・コンテンツをホスティングし、Cloud Functions でアプリケーション・バックエンドを実装して、サーバーレス Web アプリケーションを作成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution8/Architecture.png" alt="サーバーレス Web アプリケーションと API というソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                複数の地域にわたるサーバーレス・アプリのデプロイ
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Functions および Cloud Internet Services を使用して、グローバルに使用可能でセキュアなサーバーレス・アプリケーションをビルドします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution44-multi-region-serverless/Architecture.png" alt="複数の地域にわたるサーバーレス・アプリのデプロイのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
LogDNA と Sysdig によるログの分析とアプリケーションの正常性のモニター</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Log Analysis with LogDNA を使用して、アプリケーション・アクティビティーを理解および診断します IBM Cloud Monitoring with Sysdig を使用してアプリケーションをモニターします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution12/Architecture.png" alt="LogDNA および Sysdig を使用したログの分析およびアプリケーション正常性のモニターというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                分離された Cloud Foundry エンタープライズ・アプリ
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Foundry エンタープライズ環境を使用して、分離されたエンタープライズ・グレードの Cloud Foundry プラットフォームをデプロイすることにより、組織にイノベーション・プラットフォームを提供します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="分離された Cloud Foundry エンタープライズ・アプリというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## チャットボット
{: #chatbots }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-slack-chatbot-database-watson#slack-chatbot-database-watson">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                データベース駆動型の Slackbot の作成
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Watson Assistant、Cloudant、IBM Cloud Functions を使用して、データベース駆動型の Slackbot を作成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution19/SlackbotArchitecture.png" alt="データベース駆動型 Slackbot の作成のソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-watson-chatbot#android-watson-chatbot">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                音声対応の Android チャットボットの作成
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>インテントとエンティティーを定義し、チャットボットがお客様の照会に対応するためのダイアログ・フローを作成します。Android アプリと簡単に対話できるように、Speech to Text および Text to Speech の各サービスを有効にします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution28-watson-chatbot-android/architecture.png" alt="音声が有効な Android チャットボットのビルドというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## セキュリティー
{: #security }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                複数の領域にわたるセキュアな Web アプリケーション
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>継続的デリバリー・パイプラインを使用して、複数の地域にわたり、Web アプリケーションの作成、保護、デプロイ、およびロード・バランシングを行います。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution1/Architecture.png" alt="複数の領域にわたるセキュアな Web アプリケーションのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                耐障害性のあるセキュアな複数地域 Kubernetes クラスター
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cloud Internet Services と Kubernetes クラスターを統合して、耐障害性のあるセキュアなソリューションを複数の地域をまたいで提供します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution32-multi-region-k8s-cis/Architecture.png" alt="耐障害性のあるセキュアな複数地域 Kubernetes クラスターのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                REST API の作成、保護および管理
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>LoopBack Node.js API フレームワークを使用して、新しい REST API を作成します。IBM Cloud で API Connect サービスを使用して、管理、可視性、セキュリティー、および速度制限を API に追加します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution13/Architecture.png" alt="REST API の作成、保護、および管理のソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
クラウド・アプリケーションへのエンドツーエンドのセキュリティーの適用</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>独自の鍵によるデータ暗号化、ユーザー認証、セキュリティー監査の機能を備えたセキュアなクラウド・アプリケーションを作成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="クラウド・アプリケーションへのエンドツーエンド・セキュリティーの適用というソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## モバイル
{: #mobile }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                 プッシュ通知を使用する iOS モバイル・アプリ
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud でプッシュ通知を使用する iOS Swift アプリケーションを作成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution6/Architecture.png" alt="プッシュ通知を使用する iOS モバイル・アプリのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                プッシュ通知を使用する Android ネイティブ・モバイル・アプリ
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud で、プッシュ通知を使用する Android ネイティブ・アプリケーションを記述します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution9/Architecture.png" alt="プッシュ通知を使用する Android ネイティブ・モバイル・アプリのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-hybrid-mobile-push-analytics#hybrid-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                プッシュ通知を使用するハイブリッド・モバイル・アプリケーション
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IBM Cloud で、プッシュ通知を使用するハイブリッド Cordova アプリケーションを開発します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution15/Architecture.png" alt="プッシュ通知を使用するハイブリッド・モバイル・アプリケーションというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-mobile-backend#serverless-mobile-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                サーバーレス・バックエンドを備えたモバイル・アプリケーション
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>コグニティブ・サービスおよびデータ・サービスで Cloud Functions を使用して、モバイル・アプリケーション用のサーバーレス・バックエンドをビルドします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution11/Architecture.png" alt="サーバーレス・バックエンドを備えたモバイル・アプリケーションのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## 機械学習および分析
{: #ml }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
ストリーミング分析と SQL によるビッグデータ・ログ</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>ログ・レコードを収集、保管、分析して、規制上の要件に対応し、情報開示を支援します。パブリッシュ/サブスクライブのメッセージングを使用し、ソリューションを数百万件のレコードに拡張してから、使い慣れた SQL で保存ログを分析します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution31/Architecture.png" alt="Streaming Analytics および SQL を使用したビッグデータ・ログというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-smart-data-lake#smart-data-lake">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                オブジェクト・ストレージを使用したデータレイクの作成
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>SQL Query でデータを照会したり Watson Studio で分析を実行したりできるツールをデータ・サイエンティストに提供します。対話式グラフを使用して、データおよび洞察を共有します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution29/architecture.png" alt="オブジェクト・ストレージを使用したデータレイクの作成のソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-github-traffic-analytics#serverless-github-traffic-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                サーバーレスと Cloud Foundry の組み合わせによるデータの取得と分析
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>リポジトリーの GitHub トラフィック統計を自動的に収集し、SQL データベースに保管して、トラフィック分析を開始します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution24-github-traffic-analytics/Architecture.png" alt="サーバーレスと Cloud Foundry の組み合わせによるデータの取得と分析のソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#create-deploy-retrain-machine-learning-model">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                予測式機械学習モデルのビルド、デプロイ、テスト、リトレーニング
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>予測機械学習モデルを作成し、API としてデプロイし、テストし、フィードバック・データでリトレーニングします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution22-build-machine-learning-model/architecture_diagram.png" alt="予測式機械学習モデルのビルド、デプロイ、テスト、リトレーニングというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-analytics-spark#big-data-analytics-spark">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
Apache Spark によるオープン・データの分析と視覚化</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Jupyter Notebook を使用して、オープン・データ・セットを分析し、視覚化します。Apache Spark サービスと IBM Watson Studio および Pixiedust を使用して、グラフィックスを生成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution23/Architecture.png" alt="Apache Spark を使用したオープン・データの分析および視覚化というソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## IoT
{: #iot }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#gather-visualize-analyze-iot-data">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                IoT データの収集、視覚化、および分析
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>IoT デバイスをセットアップし、Watson IoT Platform で大量のデータを収集し、機械学習によってデータを分析し、視覚表示を生成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution16/Architecture.png" alt="IoT データの収集、視覚化、および分析というソリューションを表すアーキテクチャー図"/>
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
                ユーザー、チーム、アプリケーションを編成するためのベスト・プラクティス
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>ID とアクセスの管理を実現するための IBM Cloud の概念と、アプリケーション開発の複数のステージをサポートするためにそれらの概念を実装する方法の概要。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution20-users-teams-applications/architecture.png" alt="ユーザー、チーム、アプリケーションを編成するためのベスト・プラクティスのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-usage#cloud-usage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                IBM Cloud のサービス、リソース、および使用量の確認
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>使用量に関するよくある質問に答えるためのさまざまなアプローチの概要。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution38/Architecture.png" alt="IBM Cloud のサービス、リソース、および使用量の確認のソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## ネットワーク
{: #Network }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                仮想プライベート・クラウド内のプライベート・サブネットとパブリック・サブネット
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>サブネットおよびインスタンスを使用して、仮想プライベート・クラウドを作成します。 セキュリティー・グループを追加し、最小限のアクセスだけを許可して、リソースを保護します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution40-vpc-public-app-private-backend/Architecture.png" alt="仮想プライベート・クラウド内のプライベート・サブネットとパブリック・サブネットのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-multi-region#vpc-multi-region">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                複数の場所およびゾーンにわたる分離されたワークロードのデプロイ
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>複数のゾーンおよび地域にわたり、仮想プライベート・クラウドにワークロードをデプロイします。 ローカルとグローバルのロード・バランサーを使用して、ゾーン間にトラフィックを分散させます。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution41-vpc-multi-region/Architecture.png" alt="複数の場所およびゾーンにわたる分離されたワークロードのデプロイのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-site2site-vpn#vpc-site2site-vpn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                オンプレミスからクラウド・リソースにセキュアかつプライベートにアクセスするために VPC/VPN ゲートウェイを使用する
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>セキュアな仮想プライベート・ネットワークを介して、仮想プライベート・クラウドを別のコンピューティング環境に接続し、IBM Cloud サービスを利用します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution46-vpc-vpn/ArchitectureDiagram.png" alt="オンプレミスからクラウド・リソースにセキュアかつプライベートにアクセスするために VPC/VPN ゲートウェイを使用するソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server#vpc-secure-management-bastion-server">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                要塞ホストを使用してリモート・インスタンスにセキュアにアクセスする
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>要塞ホストをデプロイして、仮想プライベート・クラウド内のリモート・インスタンスにセキュアにアクセスします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png" alt="要塞ホストを使用してリモート・インスタンスにセキュアにアクセスするソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
セキュアなプライベート・ネットワークを使用してワークロードを分離する</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>仮想ルーター・アプライアンスを構成して、セキュアなエンクロージャーを作成します。VLAN を関連付け、サーバーをプロビジョンし、IP ルーティングとファイアウォールをセットアップします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution33-secure-network-enclosure/Secure-priv-enc.png" alt="セキュア・プライベート・ネットワークを使用してワークロードを分離するというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-nat-config-private#nat-config-private">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                プライベート・ネットワークからのインターネット・アクセスのための NAT の構成
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>NAT マスカレードを使用して、プライベート IP アドレスをアウトバウンド・パブリック・インターフェースに変換します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution35-nat-config-private/vra-nat.png" alt="プライベート・ネットワークからのインターネット・アクセス用に NAT を構成するというソリューションを表すアーキテクチャー図k"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-byoip#byoip">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
独自の IP アドレスの持ち込み</h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>BYOIP 実装パターンの概要と、適切なパターンを見つけるためのガイド。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution37-byoip/byoipdecision.png" alt="独自の IP アドレスの持ち込というソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                VPN をセキュア・プライベート・ネットワークへ
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>リモート・ネットワーク環境と IBM Cloud のプライベート・ネットワーク上のサーバーの間に、プライベート接続を作成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png" alt="VPN をセキュア・プライベート・ネットワークへというソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vlan-spanning#vlan-spanning">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                IBM ネットワークを介したセキュアなプライベート・ネットワークのリンク
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>VLAN スパンニング・サービスを使用して、IBM Cloud プライベート・ネットワークを介して安全にリンクされた 2 つのプライベート・ネットワークをデプロイします。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution43-vlan-spanning/vlan-spanning.png" alt="IBM ネットワークを介したセキュアなプライベート・ネットワークのリンクのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-web-app-private-network#web-app-private-network">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                セキュア・プライベート・ネットワークで提供する Web アプリケーション
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>仮想ルーター・アプライアンス (VRA)、VLAN、NAT、およびファイアウォールを使用して保護されたプライベート・ネットワークでホスティングされる、スケーラブルでセキュアなインターネット接続 Web アプリケーションを作成します。</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution42-web-app-private-network/web-app-private.png" alt="セキュア・プライベート・ネットワークで提供する Web アプリケーションのソリューションを表すアーキテクチャー図"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

