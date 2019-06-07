---
copyright:
  years: 2019
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

#                 分離された Cloud Foundry エンタープライズ・アプリ
            
{: #isolated-cloud-foundry-enterprise-apps}

{{site.data.keyword.cfee_full_notm}} (CFEE) を使用して、オンデマンドで複数の分離したエンタープライズ・クラスの Cloud Foundry プラットフォームを作成できます。 これにより、分離した Kubernetes クラスターにデプロイされたプライベート Cloud Foundry インスタンスを取得します。 パブリック Cloud とは異なり、アクセス制御、キャパシティー、バージョン、リソース使用量とモニターなど、環境を完全に制御できます。 {{site.data.keyword.cfee_full_notm}} は、エンタープライズ IT でのインフラストラクチャー所有権によって、platform-as-a-service にスピードとイノベーションをもたらします。

{{site.data.keyword.cfee_full_notm}} のユース・ケースには、エンタープライズ所有のイノベーション・プラットフォームがあります。 エンタープライズの開発者は、新しいマイクロサービスを作成するか、またはレガシー・アプリケーションを CFEE にマイグレーションできます。その後、Cloud Foundry マーケットプレイスを使用して、マイクロサービスを追加の開発者に公開できます。 この場合、CFEE インスタンス内の他の開発者は、本日パブリック・クラウドで行ったのと同様にアプリケーション内でサービスを利用できます。

このチュートリアルでは、{{site.data.keyword.cfee_full_notm}} の作成と構成、アクセス制御のセットアップ、およびアプリとサービスのデプロイのプロセスを順に説明します。 また、カスタム・サービスと CFEE を統合するカスタム・サービス・ブローカーをデプロイして、CFEE と [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) の関係をレビューします。

## 達成目標

{: #objectives}

- CFEE とパブリック Cloud Foundry の比較
- CFEE 内でのアプリとサービスのデプロイ
- Cloud Foundry と {{site.data.keyword.containershort_notm}} の関係の理解
- Cloud Foundry と {{site.data.keyword.containershort_notm}} の基本的なネットワーキングの確認

## 使用するサービス

{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

このチュートリアルでは、費用が発生する場合があります。 [料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー

{: #architecture}

![アーキテクチャー](./images/solution45-CFEE-apps/Architecture.png)

1. 管理者が CFEE インスタンスを作成し、開発者アクセス権限を持つユーザーを追加します。
2. 開発者が Node.js スターター・アプリを CFEE にプッシュします。
3. Node.js スターター・アプリが [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) を使用して、データを保管します。
4. 開発者が新しいウェルカム・メッセージ・サービスを追加します。
5. Node.js スターター・アプリが、カスタム・サービス・ブローカーからの新しいサービスをバインドします。
6. Node.js スターター・アプリに、サービスからの「ウェルカム」が異なる言語で表示されます。

## 前提条件

{: #prereq}

- [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [Cloud Foundry CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## {{site.data.keyword.cfee_full_notm}} のプロビジョン

{:provision_cfee}

このセクションでは、{{site.data.keyword.containershort_notm}} から Kubernetes ワーカー・ノードにデプロイされている {{site.data.keyword.cfee_full_notm}} のインスタンスを作成します。

1. 必要なインフラストラクチャー・リソースが作成されるように、[{{site.data.keyword.cloud_notm}} アカウント](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare)を準備します。
2. {{site.data.keyword.cloud_notm}} カタログから [{{site.data.keyword.cfee_full_notm}} ](https://{DomainName}/cfadmin/create) のサービス・インスタンスを作成します。
3. 以下を指定して、CFEE を構成します。
   - **「標準」**プランを選択します。
   - サービス・インスタンスの**「名前」**を入力します。
   - 環境が作成される**「リソース・グループ」**を選択します。 CFEE インスタンスを作成するには、アカウントで少なくとも 1 つのリソース・グループにアクセスするための許可を備えている必要があります。
   - インスタンスがデプロイされる**「地域」**と**「場所」**を選択します。 [プロビジョンに使用可能な場所およびデータ・センター](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets)のリストを参照してください。
   - **{{site.data.keyword.composeForPostgreSQL}}** がデプロイされるパブリック Cloud Foundry の**「組織」**と**「Space」**を選択します。
   - Cloud Foundry 環境の**「セルの数」**を選択します。 セルでは、Diego および Cloud Foundry アプリケーションが実行されます。 可用性の高いアプリケーションには、**2** つ以上のセルが必要です。
   - **「マシン・タイプ (Machine type)」**を選択します。これにより、Cloud Foundry セルのサイズ (CPU およびメモリー) が決まります。
4. **「インフラストラクチャー」**セクションをレビューして、CFEE をサポートする Kubernetes クラスターのプロパティーを確認します。  **「ワーカー・ノード数 (Number of worker nodes)」**は、セル数に 2 を加えた値になります。プロビジョンされる Kubernetes ワーカー・ノードのうちの 2 つが CFEE コントロール・プレーンとして機能します。環境がデプロイされた Kubernetes クラスターは、{{site.data.keyword.cloud_notm}} [Clusters](https://{DomainName}/containers-kubernetes/clusters) ダッシュボードに表示されます。
5. **「作成」**ボタンをクリックして、自動デプロイメントを開始します。

自動デプロイメントには、90 分から 120 分ほどかかります。 正常に作成されたら、CFEE のプロビジョニングとサービスのサポートを確定する E メールを受信します。

### 組織およびスペースの作成

{{site.data.keyword.cfee_full_notm}} の作成後は、[組織およびスペースの作成](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs)を参照して、組織およびスペースによって環境を構造化する方法に関する情報を確認します。{{site.data.keyword.cfee_full_notm}} のアプリは、特定のスペース内にスコープ設定されます。 同様に、組織内にもスペースが存在します。 組織のメンバーは、割り当て量プラン、アプリ、サービス・インスタンス、およびカスタム・ドメインを共有します。

注: 上部の {{site.data.keyword.cloud_notm}} ヘッダーにある**「管理」>「アカウント」>「Cloud Foundry の組織」**メニューは、パブリック {{site.data.keyword.cloud_notm}} 組織専用です。CFEE 組織は、CFEE インスタンスの**「組織」**ページで管理します。

以下のステップに従って、CFEE 組織およびスペースを作成します。

1. [Cloud Foundry ダッシュボード](https://{DomainName}/dashboard/cloudfoundry/overview)で**「エンタープライズ」**の下の**「環境」**を選択します。
2. CFEE インスタンスを選択し、**「組織」**を選択します。
3. **「組織の作成 (Create Organizations)」**ボタンをクリックし、**「組織名」**に `tutorial` を指定して、**「割り当て量プラン」**を選択します。最後に**「追加 (Add)」**をクリックします。
4. 新しく作成した組織 `tutorial` をクリックし、**「スペース」**タブを選択して、**「スペースの作成」**ボタンをクリックします。
5. **「スペース名」**に `dev` を指定し、**「追加 (Add)」**をクリックします。

### 組織およびスペースへのユーザーの追加

CFEE では、ユーザー・アクセスを制御する役割を割り当てることができますが、これを行うには、{{site.data.keyword.cloud_notm}} ヘッダーの**「管理」>「ユーザー」**にある**「ID およびアクセス (Identity & Access)」**ページで、{{site.data.keyword.cloud_notm}} アカウントにユーザーを招待する必要があります。

ユーザーを招待したら、以下のステップに従って、作成した `tutorial` 組織にユーザーを追加します。 

1. [CFEE インスタンス](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments)を選択し、再度**「組織」**を選択します。
2. リストから作成した `tutorial` 組織を選択します。
3. **「メンバー」**タブをクリックして、新しいユーザーを表示および追加します。
4. **「メンバーの追加」**ボタンをクリックしてユーザー名を検索し、適切な**組織の役割**を選択して、**「追加 (Add)」**をクリックします。

CFEE 組織およびスペースへのユーザーの追加について詳しくは、[こちら](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users)を参照してください。

## CFEE アプリのデプロイ、構成、および実行

{:deploycfeeapps}

このセクションでは、Node.js アプリケーションを CFEE にデプロイします。 デプロイ後に、{{site.data.keyword.cloudant_short_notm}} をバインドし、監査およびロギング・パーシスタンスを有効にします。 アプリケーションを管理するために、Stratos コンソールも追加します。

### アプリケーションの CFEE へのデプロイ

1. ご使用の端末で [get-started-node](https://github.com/IBM-Cloud/get-started-node) サンプル・アプリケーションを複製します。
   ```sh
   git clone https://github.com/IBM-Cloud/get-started-node
   ```
   {: codeblock}
2. アプリをローカルで実行し、適切にビルドおよび開始されていることを確認します。 ご使用のブラウザーで `http://localhost:3000/` にアクセスして、確定します。
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. {{site.data.keyword.cloud_notm}} にログインし、自分の CFEE インスタンスをターゲットにします。 対話式のプロンプトに従って、新しい CFEE を選択します。 1 つの CFEE 組織およびスペースのみが存在するため、これらがデフォルトのターゲットになります。 複数の組織またはスペースを追加した場合は、`ibmcloud target -o tutorial -s dev` を実行します。
   ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
   {: codeblock}
4. **get-started-node** アプリを CFEE にプッシュします。
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. アプリのエンドポイントが、`routes` プロパティーの横の最終出力に表示されます。 ご使用のブラウザーで URL を開き、アプリケーションが実行されていることを確定します。

### Cloudant データベースの作成とアプリへのバインド

{{site.data.keyword.cloud_notm}} サービスを **get-started-node** アプリケーションにバインドするには、最初に {{site.data.keyword.cloud_notm}} アカウントでサービスを作成する必要があります。

1. [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) サービスを作成します。 **サービス名** `cfee-cloudant` を指定し、CFEE インスタンスが作成されたのと同じ場所を選択します。
2. 新しく作成した {{site.data.keyword.cloudant_short_notm}} サービス・インスタンスを CFEE に追加します。
   1. `tutorial` **組織**に戻ります。 **「スペース」**タブをクリックし、`dev` スペースを選択します。
   2. **「サービス」**タブを選択し、**「サービスの追加」**ボタンをクリックします。
   3. 検索テキスト・ボックスに `cfee-cloudant` と入力し、結果を選択します。 最後に**「追加 (Add)」**をクリックします。 これで、サービスが CFEE アプリケーションで使用できるようになりました。ただし、サービスはまだパブリック {{site.data.keyword.cloud_notm}} にあります。
3. 表示されたサービス・インスタンスのオーバーフロー・メニューで**「アプリケーションへのバインド」**を選択します。
4. 前にプッシュした **GetStartedNode** アプリケーションを選択し、**「バインディング後のアプリケーションの再ステージ」**をクリックします。 最後に**「バインド」**ボタンをクリックします。 アプリケーションが再ステージされるまで待ってください。 進行を確認するには、`ibmcloud app show GetStartedNode` コマンドを使用します。
5. ご使用のブラウザーでアプリケーションにアクセスし、名前を追加して、`Enter` を押します。 名前が {{site.data.keyword.cloudant_short_notm}} データベースに追加されます。
6. **「サービス」**タブでリストから `tutorial` インスタンスを選択して、確認します。 これにより、パブリック {{site.data.keyword.cloud_notm}} でサービス・インスタンスの詳細ページが開きます。
7. **「Cloudant ダッシュボードの起動 (Launch Cloudant Dashboard)」**をクリックし、`mydb` データベースを選択します。 自分の名前が付いた JSON 文書が存在します。

### 監査およびロギング・パーシスタンスの有効化

監査によって、CFEE 管理者は、ログイン、組織およびスペースの作成、ユーザーへのメンバーシップと役割の割り当て、アプリケーションのデプロイメント、サービスのバインド、ドメイン構成などの Cloud Foundry アクティビティーを追跡できます。 監査は、{{site.data.keyword.cloudaccesstrailshort}} サービスとの統合によってサポートされます。

Cloud Foundry アプリケーション・ログは、{{site.data.keyword.loganalysisshort_notm}} を統合することによって保管できます。 CFEE 管理者が選択した {{site.data.keyword.loganalysisshort_notm}} サービス・インスタンスは、CFEE インスタンスから生成された Cloud Foundry ロギング・イベントを受け取って永続化するように自動的に構成されます。

CFEE 監査およびロギング・パーシスタンスを有効にするには、[こちらのステップ](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging)に従います。

### アプリを管理する Stratos コンソールのインストール

Stratos コンソールは、Cloud Foundry で作業するためのオープン・ソースの Web ベース・ツールです。 Stratos コンソール・アプリケーションをオプションで特定の CFEE 環境にインストールして使用することで、組織、スペース、およびアプリケーションを管理できます。

Stratos コンソール・アプリケーションをインストールするには、以下のようにします。

1. Stratos コンソールをインストールする CFEE インスタンスを開きます。
2. **「概要」**ページで**「Stratos Console のインストール」**をクリックします。 このボタンは、管理者またはエディターの許可を備えているユーザーにのみ表示されます。
3. 「Stratos コンソールのインストール (Install Stratos Console)」ダイアログで、インストール・オプションを選択します。 Stratos コンソール・アプリケーションは、CFEE コントロール・プレーンまたはセルのいずれかにインストールできます。 Stratos コンソールのバージョン、およびインストールするアプリケーションのインスタンス数を選択します。 セルに Stratos コンソール・アプリをインストールする場合、アプリケーションをデプロイする組織およびスペースを指定するように求められます。
4. **「インストール」**をクリックします。

アプリケーションのインストールに約 5 分かかります。 インストールが完了すると、概要ページに**「Stratos Console のインストール」**ボタンの代わりに**「Stratos コンソール (Stratos Console)」**ボタンが表示されます。Stratos コンソールについて詳しくは、[こちら](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app)を参照してください。

## CFEE と Kubernetes の関係

CFEE はアプリケーション・プラットフォームとして、何らかの形式の専用または共有仮想インフラストラクチャーで実行されます。 長年にわたり、開発者は、基盤となる Cloud Foundry プラットフォームについてあまり考えてきませんでした。これは、IBM が開発者のために管理していたためです。 CFEE を使用すると、開発者だけでなく Cloud Foundry プラットフォームのオペレーターも、Cloud Foundry アプリケーションを作成できます。 これは、CFEE が、制御している Kubernetes クラスターにデプロイされるためです。

Cloud Foundry 開発者は Kubernetes に馴染みがないかもしれませんが、両方に共通する概念が数多くあります。Cloud Foundry と同様に、Kubernetes は、ポッドと呼ばれる Kubernetes 構成体内部で実行されるコンテナーにアプリケーションを分離します。 アプリケーション・インスタンスに似ているポッドには、Kubernetes によって提供されるアプリケーション・ロード・バランシングを使用して複数のコピー (レプリカ・セットと呼ばれる) を含めることができます。  前にデプロイした Cloud Foundry `GetStartedNode` アプリケーションは、`diego-cell-0` ポッド内部で実行されます。 高可用性をサポートするために、別のポッド `diego-cell-1` が別個の Kubernetes ワーク・ノードで実行されます。 Cloud Foundry アプリは「Kubernetes 内部」で実行されるため、Kubernetes ベースのネットワーキングを使用して他の Kubernetes マイクロサービスと通信することもできます。 以下の各セクションでは、CFEE と Kubernetes の関係をさらに詳しく示します。

## Kubernetes サービス・ブローカーのデプロイ

このセクションでは、CFEE のサービス・ブローカーとして機能するマイクロサービスを Kubernetes にデプロイします。 [サービス・ブローカー](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md)は、使用可能なサービスの詳細を提供し、Cloud Foundry アプリケーションにバインドおよびプロビジョニングのサポートを提供します。組み込まれた {{site.data.keyword.cloud_notm}} サービス・ブローカーを使用して、{{site.data.keyword.cloudant_short_notm}} サービスを追加します。ここでは、カスタム・サービスをデプロイして使用します。その後、サービス・ブローカーを使用するように `GetStartedNode` アプリを変更します。これにより、ウェルカム・メッセージが複数の言語で返されます。

1. ご使用の端末に戻り、Kubernetes デプロイメント・ファイルおよびサービス・ブローカーの実装を提供するプロジェクトを複製します。

   ```sh
   git clone https://github.com/IBM-Cloud/cfee-service-broker-kubernetes.git
   ```
   {: codeblock}

   ```sh
   cd cfee-service-broker-kubernetes
   ```
   {: codeblock}

   ```sh
   git clone https://github.com/IBM/sample-resource-service-brokers.git
   ```
   {: codeblock}

2. {{site.data.keyword.registryshort_notm}} でサービス・ブローカーを格納する Docker イメージをビルドおよび保管します。 `ibmcloud cr info` コマンドを使用して、レジストリー URL を手動で取得するか、以下の `export REGISTRY` コマンドを使用して自動的に取得します。 `cr namespace-add` コマンドによって、docker イメージを保管する名前空間が作成されます。

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   `cfee-tutorial` という名前の名前空間を作成します。

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   `./cfee-service-broker-kubernetes/deployment.yml` ファイルを編集します。 コンテナー・レジストリー URL を反映するように `image` 属性を更新します。
3. コンテナー・イメージを CFEE の Kubernetes クラスターにデプロイします。 CFEE のクラスターは、`default` リソース・グループに存在します。まだターゲットにしていない場合は、ターゲットにする必要があります。`cluster-config` コマンドでクラスターの名前を使用して、KUBECONFIG 変数をエクスポートします。 その後、デプロイメントを作成します。
   ```sh
   ibmcloud target -g default
   ```
   {: codeblock}

   ```sh
   ibmcloud ks clusters
   ```
   {: codeblock}

   ```sh
   $(ibmcloud ks cluster-config <your-cfee-cluster-name> --export)
   ```
   {: codeblock}

   ```sh
   kubectl apply -f deployment.yml
   ```
   {: codeblock}
4. ポッドの STATUS が `Running` であることを確認します。 Kubernetes がイメージをプルし、コンテナーを開始するまでに多少時間がかかることがあります。 `deployment.yml` が 2 つの `replicas` を要求したため、ポッドが 2 つあります。
   ```sh
   kubectl get pods
   ```
   {: codeblock}

## サービス・ブローカーがデプロイされていることの確認

サービス・ブローカーがデプロイされたため、その機能を適切に確認します。 これを行うには、複数の方法があります。最初に Kubernetes ダッシュボードを使用し、次に Cloud Foundry アプリからブローカーにアクセスし、最後にブローカーからサービスを実際にバインドします。

### Kubernetes ダッシュボードからのポッドの表示

このセクションでは、{{site.data.keyword.containershort_notm}} ダッシュボードを使用して Kubernetes 成果物が構成されていることを確認します。

1. [「Kubernetes クラスター」](https://{DomainName}/containers-kubernetes/clusters)ページで、CFEE サービスの名前で始まり、**-cluster** で終わる行項目をクリックして、CFEE クラスターにアクセスします。
2. 対応するボタンをクリックして、**Kubernetes ダッシュボード**を開きます。
3. 左側のメニューで**「サービス」**リンクをクリックし、**tutorial-broker-service** を選択します。 このサービスは、前のステップで `kubectl apply` を実行したときにデプロイされています。
4. 開いたダッシュボードで以下を確認します。
   - サービスが、Kubernetes クラスター内でのみ解決可能なオーバーレイ IP アドレス (172.x.x.x) を提供していること。
   - サービスに、サービス・ブローカー・コンテナーが実行されている 2 つのポッドに対応する 2 つのエンドポイントがあること。

サービスが使用可能であり、サービス・ブローカー・ポッドをプロキシングしていることを確認したら、使用可能なサービスに関する情報に対するブローカーの応答を検証できます。

Kubernetes ダッシュボードから Cloud Foundry 関連の成果物を表示できます。 これらを表示するには、**「名前空間」**をクリックして、`cf` Cloud Foundry 名前空間を含む成果物とともにすべての名前空間を表示します。
{: tip}

### Cloud Foundry コンテナーからブローカーへのアクセス

Cloud Foundry から Kubernetes への通信をデモンストレーションするために、Cloud Foundry アプリケーションから直接サービス・ブローカーに接続します。

1. 端末に戻り、`ibmcloud target` を使用して、CFEE `tutorial` 組織および `dev` スペースに接続されていることを確認します。必要に応じて、CFEE を再度ターゲットとして指定します。
   ```sh
   ibmcloud target --cf
   ```
   {: codeblock}
2. デフォルトでは、SSH はスペースで無効になっています。 これはパブリック・クラウドとは異なるため、CFEE `dev` スペースで SSH を有効化します。
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. `kubectl` コマンドを使用して、Kubenetes ダッシュボードに表示されていたものと同じ ClusterIP を表示します。 次に、SSH によって `GetStartedNode` アプリケーションにアクセスし、IP アドレスを使用して、サービス・ブローカーからデータを取得します。 最後のコマンドはエラーになる可能性があります。これは次のステップで解決します。
   ```sh
   kubectl get service tutorial-broker-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf ssh GetStartedNode
   ```
   {: codeblock}

   ```sh
   export CLUSTER_IP=<CLUSTER-IP address>
   ```
   {: codeblock}

   ```sh
   wget --user TestServiceBrokerUser --password TestServiceBrokerPassword -O- http://$CLUSTER_IP/v2/catalog
   ```
   {: codeblock}
4. **Connection refused** エラーを受け取る可能性があります。 これは、CFEE のデフォルトの[アプリケーション・セキュリティー・グループ](https://docs.cloudfoundry.org/concepts/asg.html)のためです。 アプリケーション・セキュリティー・グループ (ASG) は、Cloud Foundry コンテナーからの退出トラフィックに許可可能な IP 範囲を定義します。 `GetStartedNode` がデフォルトの範囲の外側に存在するため、エラーが発生します。SSH セッションを終了し、`public_networks` ASG をダウンロードします。
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. `public_networks.json` ファイルを編集し、使用されている ClusterIP アドレスが既存の規則の範囲外であることを確認します。 例えば、範囲 `172.32.0.0-192.167.255.255` には ClusterIP が含まれていない可能性があり、更新する必要があります。 例えば、ClusterIP アドレスが `172.21.107.63` である場合、`172.0.0.0-255.255.255.255` が含まれるようにファイルを編集する必要があります。
6. ASG `destination` 規則を調整して、Kubernetes サービスの IP アドレスが含まれるようにします。 先頭と末尾が大括弧の JSON データのみが含まれるようにファイルをトリムします。 その後、新しい ASG をアップロードします。
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. ステップ 3 を繰り返すと、今回はモック・カタログ・データで成功します。 最後に SSH セッションを終了します。
   ```sh
   exit
   ```
   {: codeblock}

### サービス・ブローカーの CFEE への登録

開発者にサービス・ブローカーからのサービスのプロビジョンとバインドを許可するには、サービス・ブローカーを CFEE に登録します。 以前に、IP アドレスを使用してブローカーを処理しました。 これには、問題があります。 サービス・ブローカーが再始動する場合、新しい IP アドレスを受信し、CFEE を更新する必要があります。 この問題に対応するには、KubeDNS と呼ばれる別の Kubernetes 機能を使用します。この機能は、サービス・ブローカーに完全修飾ドメイン名 (FQDN) を提供します。

1. `tutorial-service-broker` サービスの FQDN を使用して、サービス・ブローカーを CFEE に登録します。 このルートも、CFEE Kubernetes クラスターの内部です。
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. 次に、ブローカーによって提供されるサービスを追加します。 サンプル・ブローカーにはモック・サービスが 1 つあるだけであるため、単一のコマンドが必要です。
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. ご使用のブラウザーで、[**「環境」**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments)ページから CFEE インスタンスにアクセスし、`organizations -> spaces` にナビゲートして `dev` スペースを選択します。
4. **「サービス」**タブを選択し、**「サービスの作成」**ボタンを選択します。
5. 検索テキスト・ボックスで、**Test** を検索します。 ブローカーからの **Test Node Resource Service Broker Display Name** モック・サービスが表示されます。
6. このサービスを選択し、**「作成」**ボタンをクリックし、名前 `welcome-service` を指定して、サービス・インスタンスを作成します。名前が welcome-service である理由は次のセクションで明らかになります。 次に、オーバーフロー・メニューの**「アプリケーションへのバインド (Bind to appliction)」**項目を使用して、サービスを `GetStartedNode` アプリにバインドします。
7. バインドされたサービスを表示するには、`cf env` コマンドを実行します。 `GetStartedNode` アプリケーションは、現在 `cloudantNoSQLDB` のデータを使用しているのと同様に、`credentials` オブジェクトのデータを活用できるようになりました。
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## サービスのアプリケーションへの追加

この時点で、サービス・ブローカーをデプロイしていますが、実際のサービスはデプロイしていません。 サービス・ブローカーは、データの `GetStartedNode` へのバインドを提供しますが、サービス自体からの実際の機能は追加されていません。 これを修正するために、さまざまな言語のウェルカム・メッセージを `GetStartedNode` に提供するモック・サービスが作成されました。 このサービスを CFEE にデプロイし、これを使用するようにブローカーとアプリケーションを更新します。

1. `welcome-service` 実装を CFEE にプッシュして、追加の開発チームがこの実装を API として活用できるようにします。
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. ブラウザーを使用して、このサービスにアクセスします。 サービスへの `route` が、`push` コマンドの出力の一部として表示されます。 結果の URL は、`https://welcome.<your-cfee-cluster-domain>` のようになります。 別の言語を表示するには、ページを最新表示します。
3. `sample-resource-service-brokers` フォルダーに戻り、Node.js サンプル実装 `sample-resource-service-brokers/node-resource-service-broker/testresourceservicebroker.js` を編集して、行 854 の後に url を追加し、クラスター URL を追加します。 

   ```javascript
   // TODO - Do your actual work here
    
   var generatedUserid   = uuid();
   var generatedPassword = uuid();
    
   result = {
     credentials: {
       userid   : generatedUserid,
       password : generatedPassword,
       url: 'https://welcome.<your-cfee-cluster-domain>'
     }
   };
   ```
   {: codeblock}
4. 更新したサービス・ブローカーをビルドしてデプロイします。 これにより、URL プロパティーがサービスをバインドするアプリに提供されます。
   ```sh
   cd ..
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   ```sh
   kubectl patch deployment tutorial-servicebroker-deployment -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"version\":\"2\"}}}}}"
   ```
   {: codeblock}
5. 端末で `get-started-node` アプリケーション・フォルダーに戻ります。 
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. `get-stared-node` の `get-started-node/server.js` ファイルを編集して、以下のミドルウェアが含まれるようにします。 このファイルを `app.listen()` 呼び出しの直前に挿入します。
   ```javascript
   // Use the welcome service
   const request = require('request');
   const testService = appEnv.services['testnoderesourceservicebrokername'];

   if (testService) {
     const { credentials: { url} } = testService[0];
     app.get("/api/welcome", (req, res) => request(url, (e, r, b) => res.send(b)));
   } else {
     app.get("/api/welcome", (req, res) => res.send('Welcome'));
   }
   ```
   {: codeblock}
7. `get-started-node/views/index.html` ページをリファクタリングします。 `h1` タグで `data-i18n="welcome"` を `id="welcome"` に変更します。 その後、`$(document).ready()` 関数の最後でサービスに呼び出しを追加します。
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. `GetStartedNode` アプリを更新します。 `server.js` に追加されている `request` パッケージ依存関係を含め、`welcome-service` を再バインドして、新しい `url` プロパティーを取得し、最後にアプリの新しいコードをプッシュします。
   ```sh
   npm i request -S
   ```
   {: codeblock}

   ```sh
   ibmcloud cf unbind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf bind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}

これで完了です。 ここでアプリケーションにアクセスし、ページを複数回最新表示して、ウェルカム・メッセージをさまざまな言語で表示します。

![サービス・ブローカーの応答](./images/solution45-CFEE-apps/service-broker.png)

引き続き **Welcome** のみを表示し、他の言語は表示しない場合は、`ibmcloud cf env GetStartedNode` コマンドを実行し、サービスへの `url` が存在することを確認します。 それ以外の場合は、ステップを戻って、ブローカーを更新および再デプロイします。 `url` 値が存在する場合は、ブラウザーでメッセージが返されることを確認します。 その後、`unbind-service` コマンドと `bind-service` コマンドを再度実行し、`ibmcloud cf restart GetStartedNode` を実行します。
{: tip}

ウェルカム・サービスでは Cloud Foundry がその実装として使用されますが、同様に Kubernetes を簡単に使用することもできます。 主な相違点は、サービスへの URL が `welcome-service.default.svc.cluster.local` になる可能性が高い点です。 KubeDNS URL の使用には、サービスへのネットワーク・トラフィックを Kubernetes クラスターの内部に保持するという追加の利点があります。

{{site.data.keyword.cfee_full_notm}} では、これらの代替アプローチが可能です。

## ツールチェーンを使用したソリューション・チュートリアルのデプロイ  

オプションとして、ツールチェーンを使用して完全なソリューション・チュートリアルをデプロイできます。 [ツールチェーンの説明](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)に従い、ツールチェーンを使用して前出のすべてをデプロイします。 

注: ツールチェーンを使用する際の前提条件として、CFEE インスタンス、CFEE 組織、および CFEE スペースが作成されている必要があります。 詳細な手順は、[ツールチェーンの説明](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)の readme を参照してください。 

## 関連コンテンツ
{:related}

* [Kubernetes クラスターでのアプリのデプロイ](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Cloud Foundry の Diego コンポーネントおよびアーキテクチャー](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [ツールチェーンを使用する Kubernetes の CFEE サービス・ブローカー](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

