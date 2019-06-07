---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# ユーザー、チーム、アプリケーションを編成するためのベスト・プラクティス
{: #users-teams-applications}

このチュートリアルでは、ID とアクセスの管理を実現するための {{site.data.keyword.cloud_notm}} の概念と、アプリケーション開発の複数のステージをサポートするためにそれらの概念を実装する方法について概要を説明します。{:shortdesc}

アプリケーションを作成するときには、開発者がコードをコミットする段階から、アプリケーション・コードをエンド・ユーザーに公開する段階までのプロジェクトの開発ライフサイクルを反映して複数の環境を定義するのが非常に一般的です。*サンドボックス*、*テスト*、*ステージング*、*UAT* (ユーザー受け入れテスト)、*実動前*、*実動*などが、そうした環境を表す典型的な名称です。

そうした環境を別々に作成する理由としては、基礎リソースの分離、ガバナンスやアクセス・ポリシーの実装、実動ワークロードの保護、実動環境に移す前の変更内容の検証などの理由があります。

## 達成目標
{: #objectives}

* {{site.data.keyword.iamlong}} と Cloud Foundry の各アクセス・モデルについて学習する。
* 役割と環境を分離してプロジェクトを構成する。
* 継続的統合をセットアップする。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## プロジェクトを定義する

以下のコンポーネントを使用するサンプル・プロジェクトについて考えていきましょう。
* {{site.data.keyword.containershort_notm}} にデプロイする複数のマイクロサービス
* データベース
* ファイル・ストレージ・バケット

このプロジェクトでは、3 つの環境を定義します。
* *開発* - この環境はコミットするたびに更新され、単体テストとスモーク・テストが実行されます。プロジェクトの最新/最高のデプロイメントにアクセスできる環境になります。
* *テスト* - この環境は安定したコードのブランチやタグの後に作成します。ユーザー受け入れテストを実行する場所になります。構成は実動環境によく似ています。現実的なデータ (実動データをサンプルとして匿名化したもの) をロードします。
* *実動* - この環境は上記の環境で確認したバージョンに更新されます。

**デリバリー・パイプラインで、ビルドの環境間の進行を管理します。**完全に自動化することも、承認したビルドを次の環境に進めるための手動の検証ゲートを組み込むこともできます。非常に自由に構成できるので、自社のベスト・プラクティスやワークフローに合わせてセットアップしてください。

ビルド・パイプラインの実行に役立つように、**機能ユーザー**という考えを採用することにします。機能ユーザーとは、通常の {{site.data.keyword.cloud_notm}} ユーザーですが、現実世界には実在しないチーム・メンバーです。この機能ユーザーが、デリバリー・パイプラインや、強い所有権を必要とする他のクラウド・リソースの所有者になります。このアプローチは、チーム・メンバーが退職したり別のプロジェクトに移ったりした場合に役立ちます。機能ユーザーはプロジェクト専用のユーザーであり、プロジェクトが続く限り変わりません。そして次は、その機能ユーザーの [API キー](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey)を作成します。DevOps パイプラインをセットアップしたり自動化スクリプトを実行したりするときには、この API キーを選択して機能ユーザーを偽装します。

プロジェクト・チームのメンバーへの役割の割り当てについては、以下の役割および関連する権限を定義しましょう。

|           | 開発 | テスト | 実動 |
| --------- | ----------- | ------- | ---------- |
| 開発者 | <ul><li>コードを共同作成する</li><li>ログ・ファイルにアクセスできる</li><li>アプリやサービスの構成を表示できる</li><li>デプロイ済みのアプリケーションを使用する</li></ul> | <ul><li>ログ・ファイルにアクセスできる</li><li>アプリやサービスの構成を表示できる</li><li>デプロイ済みのアプリケーションを使用する</li></ul> | <ul><li>アクセスなし</li></ul> |
| テスター    | <ul><li>デプロイ済みのアプリケーションを使用する</li></ul> | <ul><li>デプロイ済みのアプリケーションを使用する</li></ul> | <ul><li>アクセスなし</li></ul> |
| オペレーター  | <ul><li>ログ・ファイルにアクセスできる</li><li>アプリやサービスの構成を表示/設定できる</li></ul> | <ul><li>ログ・ファイルにアクセスできる</li><li>アプリやサービスの構成を表示/設定できる</li></ul> | <ul><li>ログ・ファイルにアクセスできる</li><li>アプリやサービスの構成を表示/設定できる</li></ul> |
| パイプラインの機能ユーザー  | <ul><li>アプリケーションをデプロイ/アンデプロイできる</li><li>アプリやサービスの構成を表示/設定できる</li></ul> | <ul><li>アプリケーションをデプロイ/アンデプロイできる</li><li>アプリやサービスの構成を表示/設定できる</li></ul> | <ul><li>アプリケーションをデプロイ/アンデプロイできる</li><li>アプリやサービスの構成を表示/設定できる</li></ul> |

## Identity and Access Management (IAM)
{: #first_objective}

{{site.data.keyword.iamshort}} (IAM) を使用すれば、プラットフォーム・サービスとインフラストラクチャー・サービスの両方でセキュアにユーザー認証を行い、{{site.data.keyword.cloud_notm}} プラットフォーム全体で一貫した方法によって**リソース**へのアクセスを制御できます。一連の {{site.data.keyword.cloud_notm}} サービスが、Cloud IAM を使用したアクセス制御に対応しています。そうしたサービスを**アカウント**で**リソース・グループ**として編成すれば、複数のリソースへのアクセス権を素早く簡単に**ユーザー**に一括で付与することができます。アカウント内のリソースへのアクセス権限をユーザーとサービス ID に割り当てるためには、Cloud IAM アクセス・**ポリシー**を使用します。

1 つの**ポリシー**で、1 人のユーザーまたは 1 つのサービス ID に 1 つ以上の**役割**を割り当て、アクセス権限の有効範囲を定義する属性の組み合わせを指定します。ポリシーはインスタンス・レベルまで細分化した単一サービスへのアクセス権限を指定できます。また、まとめて 1 つのリソース・グループに編成されたリソースの集合にポリシーを適用することもできます。 割り当てたユーザー役割に基づいて、さまざまなレベルのアクセス権限がユーザーまたはサービス ID に与えられます。プラットフォーム管理タスクを実行できるアクセス権限や、UI を使用したり特定のタイプの API 呼び出しを実行したりしてサービスを利用できるアクセス権限などがあります。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="IAM モデルの図" />
</p>

現時点では、{{site.data.keyword.cloud_notm}} カタログのすべてのサービスが IAM で管理可能なわけではありません。そのようなサービスについては、引き続き Cloud Foundry を使用できます。許可するアクセス権限のレベルを定義するために割り当てる Cloud Foundry 役割を指定して、インスタンスが属している組織やスペースへのアクセス権限をユーザーに付与します。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Cloud Foundry モデルの図" />
</p>

## 1 つの環境用のリソースを作成する

このサンプル・プロジェクトに必要な 3 つの環境は、アクセス権限の要件が違います。容量の割り振りもおそらく違いますが、アーキテクチャーのパターンは同じです。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="1 つの環境のアーキテクチャー図" />
</p>

まずは、開発環境を作成しましょう。

1. [環境をデプロイする {{site.data.keyword.cloud_notm}} ロケーション](https://{DomainName})を選択します。
1. Cloud Foundry のサービスとアプリの場合:
   1. [プロジェクトの組織を作成します](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg)。
   1. [この環境用の Cloud Foundry スペースを作成します](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo)。
   1. そのスペースで、プロジェクトに使用する Cloud Foundry サービスを作成します。
1. [この環境用のリソース・グループを作成します](https://{DomainName}/account/resource-groups)。
1. リソース・グループに対応するサービスを作成します。ここでは、{{site.data.keyword.cos_full_notm}}、{{site.data.keyword.la_full_notm}}、{{site.data.keyword.mon_full_notm}}、{{site.data.keyword.cloudant_short_notm}} をグループにします。
1. {{site.data.keyword.containershort_notm}} で[新しい Kubernetes クラスターを作成](https://{DomainName}/containers-kubernetes/catalog/cluster)し、先ほど作成したリソース・グループを選択します。
1. ログを送信してクラスターをモニターするために、{{site.data.keyword.la_full_notm}} と {{site.data.keyword.mon_full_notm}} を構成します。

アカウント下に作成する各プロジェクト・リソースの場所を以下の図に示します。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="プロジェクト・リソースを表す図" />
</p>

## 1 つの環境内で役割を割り当てる

1. ユーザーをアカウントに招待します。
1. ユーザーにポリシーを割り当てます。これによって、リソース・グループ、そのグループ内のサービス、{{site.data.keyword.containershort_notm}} インスタンスにアクセスできる人およびその人の権限を制御します。[アクセス・ポリシーの定義](https://{DomainName}/docs/containers?topic=containers-users#access_policies)を参照して、環境内のユーザーに適したポリシーを選択してください。同じポリシー・セットを割り当てるユーザーは、[同じアクセス・グループ](https://{DomainName}/docs/iam?topic=iam-groups#groups)にすることができます。そうすれば、ユーザー管理が簡単になります。ポリシーをアクセス・グループに割り当てれば、そのグループ内のすべてのユーザーがそのポリシーを継承するからです。
1. 環境のニーズに基づいて、Cloud Foundry の組織とスペースの役割を構成します。[役割の定義](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess)を参照し、環境に応じて適切な役割を割り当ててください。

各サービスで行われる IAM 役割および Cloud Foundry 役割と具体的なアクションのマッピングについては、サービスの資料を参照してください。例えば、[{{site.data.keyword.mon_full_notm}} サービスでの IAM の役割とアクションのマッピング](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam)を参照してください。

ユーザーに適切な役割を割り当てるには、何度も割り当てを繰り返して精度を高めていく必要があります。付与する権限はリソース・グループ・レベルでも制御できます (リソース・グループ内のすべてのリソースが対象になります) が、特定のサービス・インスタンスごとにきめ細かく制御することもできます。どのようなアクセス・ポリシーがプロジェクトにとって理想的であるかは徐々に明らかになるはずです。

まずは最小セットの権限から始め、必要に応じて慎重に範囲を広げていくことをお勧めします。Kubernetes のクラスター内の権限の構成については、[Role-Based Access Control (RBAC)](https://kubernetes.io/docs/admin/authorization/rbac/) を参照してください。

開発環境では、最初に定義したユーザーの役割を以下のように表すことができます。

|           | IAM アクセス・ポリシー | Cloud Foundry |
| --------- | ----------- | ------- |
| 開発者 | <ul><li>リソース・グループ: *ビューアー*</li><li>リソース・グループのプラットフォーム・アクセス役割: *ビューアー*</li><li>ロギング & モニタリング・サービスの役割: *ライター*</li></ul> | <ul><li>組織の役割: *監査員*</li><li>スペースの役割: *監査員*</li></ul> |
| テスター    | <ul><li>構成不要。テスターは、開発環境ではなくデプロイ済みのアプリケーションにアクセスする</li></ul> | <ul><li>構成不要。</li></ul> |
| オペレーター  | <ul><li>リソース・グループ: *ビューアー*</li><li>リソース・グループのプラットフォーム・アクセス役割: *オペレーター*、*ビューアー*</li><li>ロギング & モニタリング・サービスの役割: *ライター*</li></ul> | <ul><li>組織の役割: *監査員*</li><li>スペースの役割: *開発者*</li></ul> |
| パイプラインの機能ユーザー | <ul><li>リソース・グループ: *ビューアー*</li><li>リソース・グループのプラットフォーム・アクセス役割: *エディター*、*ビューアー*</li></ul> | <ul><li>組織の役割: *監査員*</li><li>スペースの役割: *開発者*</li></ul> |

IAM のアクセス・ポリシーと Cloud Foundry 役割は、[Identify and Access Management ユーザー・インターフェース](https://{DomainName}/iam/#/users)で定義します。

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="開発者役割の権限の構成" />
</p>

## 同じような手順で複数の環境を作成する

ここからは、同じような手順を繰り返して他の環境を作成します。

1. 環境ごとにリソース・グループを 1 つ作成します。
1. 環境ごとにクラスターを 1 つ作成し、必要なサービス・インスタンスも作成します。
1. 環境ごとに Cloud Foundry スペースを 1 つ作成します。
1. 各スペースで必要なサービス・インスタンスを作成します。

<p style="text-align: center;">
  <img title="別々のクラスターを使用して環境を分離する" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="別々のクラスターで環境を分離した図" />
</p>

[{{site.data.keyword.cloud_notm}} `ibmcloud` CLI](https://github.com/IBM-Cloud/ibm-cloud-developer-tools)、[HashiCorp `terraform`](https://www.terraform.io/)、[{{site.data.keyword.cloud_notm}} provider for Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm)、Kubernetes CLI `kubectl` などのツールを組み合わせて、こうした環境の作成をスクリプト化して自動化できます。

環境ごとに Kubernetes クラスターを別にすると、さまざまなメリットがあります。
* 環境に関わりなく、すべてのクラスターが同じようなものになります。
* 各クラスターへのユーザー・アクセスを簡単に制御できます。
* デプロイメントと基礎リソースの更新サイクルが柔軟になります。Kubernetes の新しいバージョンが出たら、まずは開発環境のクラスターを更新し、アプリケーションを検証してから他の環境を更新するという方法を取ることができます。
* ワークロードが混在するために他の環境に影響が出ることを防止できます。例えば、実動環境のデプロイメントを他の環境から分離したりします。

別の方法として、[Kubernetes 名前空間](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)と [Kubernetes リソース割り当て量](https://kubernetes.io/docs/concepts/policy/resource-quotas/)を組み合わせることで環境を分離し、リソース消費量を制御することもできます。

<p style="text-align: center;">
  <img title="別々の名前空間を使用して環境を分離する" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="別々の名前空間で環境を分離した図" />
</p>

LogDNA UI の`「Search」`入力ボックスで`「namespace:」`フィールドを使用すれば、名前空間に基づいてログをフィルタリングできます。
{: tip}

## デリバリー・パイプラインをセットアップする

各環境へのデプロイについては、継続的統合/継続的デリバリーのパイプラインをセットアップしてプロセス全体を促進できます。
* `開発`環境は、`開発`ブランチの最新/最高のコードで継続的に更新し、専用クラスターで単体テストと統合テストを実行します。
* `テスト`環境に、開発ビルドをプロモートします。それまでのステージのすべてのテストが OK であれば自動的にプロモートできます。手動のプロモーション・プロセスでプロモートすることもできます。チームによっては、ここで別のブランチを使用することもあります。例えば、作業中の開発状態を`安定`ブランチにマージしたりします。
* 同じようなプロセスを繰り返して、`実動`環境に進みます。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="ビルドからデプロイまでの CI/CD パイプライン" />
</p>

DevOps パイプラインを構成するときには、必ず機能ユーザーの API キーを使用してください。クラスターにアプリをデプロイするのに必要な権限は、機能ユーザーだけが持つようにしてください。

ビルド・フェーズでは、Docker イメージのバージョン管理を適切に行うことが重要です。イメージ・タグの一部として Git コミット・リビジョンや、DevOps ツールチェーンで提供される固有 ID を使用すると良いでしょう。何であれ、イメージをそのイメージの中に含まれている実際のビルドやソース・コードと簡単に対応付けられる識別子を使用してください。

Kubernetes に慣れてきたら [Helm](https://helm.sh/) (Kubernetes のパッケージ・マネージャー) を使用すると便利です。このツールで、アプリケーションのバージョン管理、アセンブル、デプロイを行えます。そのための開始点として、[このサンプル DevOps ツールチェーン](https://github.com/open-toolchain/simple-helm-toolchain)をお勧めします。Kubernetes クラスターへの継続的デリバリーがあらかじめ構成されています。多数のマイクロサービスを含むプロジェクトになったら、[Helm アンブレラ・チャート](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies)が、アプリケーションを組み立てるための有効なソリューションになります。

## チュートリアルを発展させる

おめでとうございます。これで、開発環境から実動環境にアプリケーションを安全にデプロイできるようになりました。以下は、アプリケーション・デリバリーをさらに改善するためのヒントです。

* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) をパイプラインに追加して、デプロイメント時に品質管理を行います。
* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) を使用して、チーム・メンバーのコーディングの貢献度や開発者間のやり取りを審査します。
* 環境のデプロイメントを自動化するために、[デプロイメント環境の計画、作成、更新](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments)のチュートリアルを実行します。

## 関連情報

* [{{site.data.keyword.iamshort}} の入門チュートリアル](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [リソースをリソース・グループに編成するためのベスト・プラクティス](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [LogDNA と Sysdig によるログの分析と正常性のモニター](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [Kubernetes への継続的デプロイメント](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Hello Helm ツールチェーン](https://github.com/open-toolchain/simple-helm-toolchain)
* [Develop a microservices application with Kubernetes and Helm](https://github.com/open-toolchain/microservices-helm-toolchain)
* [LogDNA でログを表示する権限をユーザーに与える](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [Sysdig でメトリックを表示する権限をユーザーに与える](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
