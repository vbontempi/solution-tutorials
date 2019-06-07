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

# クラウド・アプリケーションへのエンドツーエンドのセキュリティーの適用
{: #cloud-e2e-security}

可能性のあるセキュリティー・リスクを認識し、このような脅威から保護する方法を明確に理解することなしに、アプリケーション・アーキテクチャーは完成しません。アプリケーション・データはクリティカル・リソースとなり、逸失、漏えい、盗難は許容されません。また、データは保存時および転送中に暗号化技法によって保護する必要があります。保存されたデータを暗号化することによって、そのデータが失われたり、盗まれたりした場合でも情報が開示されないように保護されます。転送中 (インターネットを介してなど) のデータを HTTPS、SSL、TLS などの方法で暗号化することによって、盗聴や中間者攻撃が回避されます。

特定のリソースへのユーザーのアクセスの認証と許可も、多くのアプリケーションに共通の要件となります。ソーシャル ID を使用する顧客およびサプライヤー、クラウドでホストされるディレクトリーからのパートナー、組織の ID 提供者からの従業員など、さまざまな認証スキームがサポートされる必要がある場合があります。

このチュートリアルでは、{{site.data.keyword.cloud}} カタログで使用できる主要なセキュリティー・サービスと、それらを一緒に使用する方法について説明します。ファイル共有を提供するアプリケーションによって、セキュリティー概念が実践されます。
{:shortdesc}

## 達成目標
{: #objectives}

* 独自の暗号鍵を使用してストレージ・バケットのコンテンツを暗号化する
* アプリケーションにアクセスする前に、ユーザーに認証を要求する
* クラウド・サービスでのセキュリティー関連の API 呼び出しやその他のアクションをモニターおよび監査する

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* オプション: [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

このチュートリアルでは、[ライト以外のアカウント](https://{DomainName}/docs/account?topic=account-accounts#accounts)が必要です。また、コストが発生する場合があります。 [料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

このチュートリアルでは、ユーザーのグループが共通のストレージ・プールにファイルをアップロードし、共有可能なリンクによってこれらのファイルへのアクセスを提供できるサンプル・アプリケーションを取り上げます。 このアプリケーションは Node.js で作成され、Docker コンテナーとして {{site.data.keyword.containershort_notm}} にデプロイされます。 複数のセキュリティー関連のサービスおよび機能を活用して、アプリケーションのセキュリティーが改善されます。

<p style="text-align: center;">

  ![アーキテクチャー](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. ユーザーがアプリケーションに接続します。
2. カスタム・ドメインおよび TLS 証明書を使用する場合は、この証明書が {{site.data.keyword.cloudcerts_short}} によって管理され、ここからデプロイされます。
3. {{site.data.keyword.appid_short}} によってアプリケーションが保護され、ユーザーが確認ページにリダイレクトされます。ユーザーは登録することもできます。
4. アプリケーションが、{{site.data.keyword.registryshort_notm}} に保管されたイメージから Kubernetes クラスターで実行されます。 このイメージは脆弱性がないか自動的にスキャンされます。
5. {{site.data.keyword.cloudant_short_notm}} に保管されたメタデータとともにアップロードされたファイルが {{site.data.keyword.cos_short}} に保管されます。
6. ファイル・ストレージ・バケットでは、ユーザー提供の鍵を活用してデータが暗号化されます。
7. アプリケーション管理アクティビティーが {{site.data.keyword.cloudaccesstrailshort}} によってログに記録されます。

## 始める前に
{: #prereqs}

1. [これらのステップに従って](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)、必要なコマンド・ライン (CLI) ツールをすべてインストールします。
2. このチュートリアルで最新バージョンのプラグインを使用していることを確認します。アップグレードするには、`ibmcloud plugin update --all` を使用します。

## サービスの作成
{: #setup}

### アプリケーションのデプロイ先の決定

1. アプリケーションとそのリソースをデプロイする**場所**、**Cloud Foundry 組織およびスペース**、および**リソース・グループ**を識別します。

### ユーザーおよびアプリケーションのアクティビティーの収集 
{: #activity-tracker }

{{site.data.keyword.cloudaccesstrailshort}} サービスは、{{site.data.keyword.Bluemix_notm}} 内のサービスの状態を変更するユーザー開始アクティビティーを記録します。 このチュートリアルの最後に、チュートリアルのステップを完了して生成されたイベントをレビューします。

1. {{site.data.keyword.Bluemix_notm}} カタログにアクセスし、[{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker) のインスタンスを作成します。 Cloud Foundry スペースごとの {{site.data.keyword.cloudaccesstrailshort}} のインスタンスは 1 つのみです。
2. インスタンスが作成されたら、その名前を **secure-file-storage-activity-tracker** に変更します。
3. すべてのアクティビティー・トラッカー・イベントを表示するには、許可が割り当てられていることを確認します。
   1. [「ID およびアクセス (Identity & Access)」>「ユーザー」](https://{DomainName}/iam/#/users)に移動します。
   2. リストからユーザー名を選択します。
   3. **「アクセス・ポリシー」**で、{{site.data.keyword.loganalysisshort_notm}} サービスが作成された場所の**ビューアー**役割を持つこのサービスのポリシーを作成します (このポリシーがない場合)。
   4. **「Cloud Foundry アクセス権限」**で、{{site.data.keyword.cloudaccesstrailshort}} がプロビジョンされた Cloud Foundry スペースの**開発者**役割があることを確認します。 この役割がない場合は、組織の Cloud Foundry 管理者と協力して役割を割り当てます。

許可のセットアップ手順の詳細は、[{{site.data.keyword.cloudaccesstrailshort}} の資料](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy)を参照してください。
{: tip}

### アプリケーション用のクラスターの作成

{{site.data.keyword.containershort_notm}} は、Kubernetes クラスターで実行される Docker コンテナーに高可用性のアプリをデプロイする環境を提供します。

既存の**標準**クラスターをこのチュートリアルで再利用する場合は、このセクションをスキップしてください。
{: tip}

1. [クラスター作成ページ](https://{DomainName}/containers-kubernetes/catalog/cluster/create)にアクセスします。
   1. **「場所」**を前のステップで使用した場所に設定します。
   2. **「クラスター・タイプ (Cluster type)」**を**「標準」**に設定します。
   3. **「可用性」**を**「単一ゾーン (Single Zone)」**に設定します。
   4. **「マスター・ゾーン (Master Zone)」**を選択します。
2. デフォルトの**「Kubernetes バージョン」**および**「ハードウェアの分離」**を保持します。
3. このクラスターでこのチュートリアルのみをデプロイする予定の場合は、**「ワーカー・ノード」**を**「1」**に設定します。
4. **「クラスター名 (Cluster name)」**を **secure-file-storage-cluster** に設定します。
5. **「クラスターの作成 (Create Cluster)」**ボタンをクリックします。

クラスターがプロビジョンされている間に、チュートリアルで必要となる他のサービスを作成します。

### 自社所有の暗号鍵の使用

{{site.data.keyword.keymanagementserviceshort}} は、{{site.data.keyword.Bluemix_notm}} サービス上のアプリの暗号鍵をプロビジョンするときに役立ちます。 {{site.data.keyword.keymanagementserviceshort}} および {{site.data.keyword.cos_full_notm}} は、[と連携して、保存されたデータを保護します](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos)。このセクションでは、ストレージ・バケット用のルート・キーを 1 つ作成します。

1. [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms) のインスタンスを作成します。
   * 名前を **secure-file-storage-kp** に設定します。
   * サービス・インスタンスを作成するリソース・グループを選択します。
2. **「管理」**で**「キーの追加」**ボタンをクリックして、新しいルート・キーを作成します。 このキーは、ストレージ・バケットのコンテンツの暗号化に使用します。
   * 名前を **secure-file-storage-root-enckey** に設定します。
   * キー・タイプを**「ルート・キー (Root key)」**に設定します。
   * 次に**「キーの生成 (Generate key)」**を選択します。

独自のキーを使用する (BYOK) には、[既存のルート・キーをインポートします](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys)。
{: tip}

### ユーザー・ファイルのストレージのセットアップ

ファイル共有アプリケーションは、ファイルを {{site.data.keyword.cos_short}} バケットに保存します。 ファイルとユーザーの関係は、メタデータとして {{site.data.keyword.cloudant_short_notm}} データベースに保管されます。 このセクションでは、以下のサービスを作成および構成します。

#### コンテンツ用のバケット

1. [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) のインスタンスを作成します。
   * **「名前」**を **secure-file-storage-cos** に設定します。
   * 前のサービスと同じ**リソース・グループ**を使用します。
2. **「サービス資格情報」**で、*新規資格情報* を作成します。
   * **「名前」**を **secure-file-storage-cos-acckey** に設定します。
   * **「役割」**を**「ライター」**に設定します。
   * **「サービス ID」**は指定しないでください。
   * **「インラインの構成パラメーター (Inline Configuration Parameters)」**を **{"HMAC":true}** に設定します。 これは署名付き URL の生成に必要です。
   * **「追加」**をクリックします。
   * **「資格情報の表示」**をクリックして、資格情報をメモします。 後のステップで必要になります。
3. メニューから**「エンドポイント」**をクリックします。**「耐障害性 (Resiliency)」**を**「地域 (Regional)」**に設定し、**「場所」**をターゲットの場所に設定します。 **パブリック**・サービス・エンドポイントをコピーします。 後のアプリケーションの構成で使用されます。

バケットを作成する前に、**secure-file-storage-kp** に保管されているルート・キーに **secure-file-storage-cos** アクセス権限を付与します。

1. {{site.data.keyword.cloud_notm}} コンソールの[「ID およびアクセス (Identity & Access)」>「許可」](https://{DomainName}/iam/#/authorizations)に移動します。
2. **「作成」**ボタンをクリックします。
3. **「ソース・サービス」**メニューで**「Cloud Object Storage (Cloud Object Storage)」**を選択します。
4. **「ソース・サービス・インスタンス (Source service instance)」**メニューで、前に作成した **secure-file-storage-cos** サービスを選択します。
5. **「ターゲット・サービス」**メニューで**「鍵の保護」**を選択します。
6. **「ターゲット・サービス・インスタンス」**メニューで、前に作成した **secure-file-storage-kp** サービスを選択して許可します。
7. **リーダー**の役割を有効にします。
8. **「許可」**ボタンをクリックします。

最後にバケットを作成します。

1. [「リソース・リスト」](https://{DomainName}/resources)から **secure-file-storage-cos** サービス・インスタンスにアクセスします。
2. **「バケットの作成 (Create bucket)」**をクリックします。 
   1. **「名前」**を **&lt;your-initials&gt;-secure-file-upload** などの固有値に設定します。
   2. **「耐障害性 (Resiliency)」**を**「地域 (Regional)」**に設定します。
   3. **「場所」**を **secure-file-storage-kp** サービスを作成した場所を同じ場所に設定します。
   4. **「ストレージ・クラス (Storage class)」**を**「標準」**に設定します
3. **「キーに保護キーを追加する (Add Key Protect Keys)」**チェック・ボックスを選択します。
   1. **secure-file-storage-kp** サービスを選択します。
   2. キーとして **secure-file-storage-root-enckey** を選択します。
4. **「バケットの作成 (Create bucket)」**をクリックします。 

#### ユーザーとそのファイルとの間のデータベース・マップ関係

{{site.data.keyword.cloudant_short_notm}} データベースには、アプリケーションからアップロードされたすべてのファイルのメタデータが含まれます。

1. [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) のインスタンスを作成します。
   * **「名前」**を **secure-file-storage-cloudant** に設定します。
   * 場所を設定します。
   * 前のサービスと同じ**リソース・グループ**を使用します。
   * **「使用可能な認証方式 (Available authentication methods)」**を**「IAM のみを使用 (Use only IAM)」**に設定します。
2. **「サービス資格情報」**で、*新規資格情報* を作成します。
   * **「名前」**を **secure-file-storage-cloudant-acckey** に設定します。
   * **「役割」**を**「管理者」**に設定します
   * *オプション* のフィールドは、デフォルト値のままにしておきます
   * **「追加」**。
3. **「資格情報の表示」**をクリックして、資格情報をメモします。 後のステップで必要になります。
4. **「管理」**で Cloudant ダッシュボードを起動します。
5. **「データベースの作成 (Create Database)」**をクリックして、**secure-file-storage-metadata** という名前のデータベースを作成します。

### ユーザー認証

{{site.data.keyword.appid_short}} を使用すると、リソースを保護し、アプリケーションに認証を追加できます。 {{site.data.keyword.appid_short}} は、{{site.data.keyword.containershort_notm}} と[統合](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth)され、クラスターにデプロイされたアプリケーションへのユーザーのアクセスを認証します。

1. [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) のインスタンスを作成します。
   * **「サービス名」**を **secure-file-storage-appid** に設定します。
   * 前のサービスと同じ**場所**と**リソース・グループ**を使用します。
2. **「認証設定 (Authentication Settings)」**タブの**「ID プロバイダー/管理 (Identity Providers / Manage)」**で、アプリケーションに使用するドメインを指す **Web リダイレクト URL** を追加します。 例えば、クラスター Ingress サブドメインが
`<cluster-name>.us-south.containers.appdomain.cloud` である場合、リダイレクト URL は `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback` になります。 {{site.data.keyword.appid_short}} では、Web リダイレクト URL が **https** である必要があります。 クラスター・ダッシュボードまたは `ibmcloud ks cluster-get <cluster-name>` を使用して、Ingress サブドメインを参照できます。

{{site.data.keyword.appid_short}} ダッシュボードで、使用されている ID プロバイダーとログインおよびユーザー管理エクスペリエンスをカスタマイズする必要があります。 このチュートリアルでは、わかりやすくするためにデフォルトを使用します。 実稼働環境では、多要素認証 (MFA) および高度なパスワード・ルールを使用することを検討してください。
{: tip}

## アプリのデプロイ

すべてのサービスが構成されました。 このセクションでは、チュートリアル・アプリケーションをクラスターにデプロイします。

### コードの取得

1. アプリケーションのコードを入手します。
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. 以下の **secure-file-storage** ディレクトリーに移動します。
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### Docker イメージのビルド

1. {{site.data.keyword.registryshort_notm}} 内で [Docker イメージをビルド](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating)します。
   - `ibmcloud cr info` を使用して、**us**.icr.io や **uk**.icr.io などのレジストリー・エンドポイントを見つけます。
   - コンテナー・イメージを保管する名前空間を作成します。
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - イメージ名には **secure-file-storage** を使用します。

   ```sh
   ibmcloud cr build -t <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### 資格情報および構成設定の入力

1. `credentials.template.env` を `credentials.env` にコピーします。
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. `credentials.env` を編集し、空白に以下の値を入力します。
   * {{site.data.keyword.cos_short}} サービスの地域エンドポイント、バケット名、**secure-file-storage-cos** 用に作成された資格情報。
   * **secure-file-storage-cloudant** の資格情報。
3. `secure-file-storage.template.yaml` を `secure-file-storage.yaml` にコピーします。
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. `secure-file-storage.yaml` を編集し、プレースホルダー (`$IMAGE_PULL_SECRET`、`$REGISTRY_URL`、`$REGISTRY_NAMESPACE`、`$IMAGE_NAME`、`$TARGET_NAMESPACE`、`$INGRESS_SUBDOMAIN`、`$INGRESS_SECRET`) を適切な値で置き換えます。 例として、アプリケーションが_デフォルト_ の Kubernetes 名前空間にデプロイされるとします。

| 変数 | 値 | 説明 |
| -------- | ----- | ----------- |
| `$IMAGE_PULL_SECRET` | .yaml でコメントされている行を保持します | レジストリーにアクセスするためのシークレット。  |
| `$REGISTRY_URL` | *us.icr.io* | 前のセクションでイメージがビルドされたレジストリー。 |
| `$REGISTRY_NAMESPACE` | *secure-file-storage-namespace* |前のセクションでイメージがビルドされたレジストリーの名前空間。 |
| `$IMAGE_NAME` | *secure-file-storage* | Docker イメージの名前。 |
| `$TARGET_NAMESPACE` | *default* | アプリがプッシュされる Kubernetes 名前空間。 |
| `$INGRESS_SUBDOMAIN` | *secure-file-stora-123456.us-south.containers.appdomain.cloud* | クラスター概要ページまたは `ibmcloud ks cluster-get secure-file-storage-cluster` を使用して取得します。 |
| `$INGRESS_SECRET` | *secure-file-stora-123456* | クラスター概要ページまたは `ibmcloud ks cluster-get secure-file-storage-cluster` を使用して取得します。 |

`$IMAGE_PULL_SECRET` は、デフォルトではなく別の Kubernetes 名前空間を使用する場合にのみ必要です。 この場合、追加の Kubernetes 構成が必要になります ([新しい名前空間で Docker レジストリー・シークレットを作成する](https://{DomainName}/docs/containers?topic=containers-images#other)など)。
{: tip}

### クラスターへのデプロイ

1. クラスター構成を取得し、KUBECONFIG 環境変数を設定します。
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. アプリケーションがサービス資格情報を取得するために使用するシークレットを作成します。
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. {{site.data.keyword.appid_short_notm}} サービス・インスタンスをクラスターにバインドします。
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
   同じ名前のサービスが複数ある場合、このコマンドは失敗します。 サービスの名前ではなく GUID を渡す必要があります。 サービスの GUID を見つけるには、`ibmcloud resource service-instance secure-file-storage-appid` を使用します。
   {: tip}
4. アプリをデプロイします。
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## アプリケーションのテスト

アプリケーションには、`https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/` でアクセスできます。

1. アプリケーションのホーム・ページに移動します。 {{site.data.keyword.appid_short_notm}} のデフォルトのログイン・ページにリダイレクトされます。
2. 有効な E メール・アドレスを使用して新しいアカウントを登録します。
3. 受信ボックスにアカウントを確認する E メール・アドレスが届くまで待機します。
4. ログインします。
5. アップロードするファイルを選択します。 **「アップロード」**をクリックします。
6. ファイルに対して**「共有」**アクションを使用して、ファイルにアクセスする他のユーザーと共有できる署名付き URL を生成します。 このリンクは 5 分後に期限切れになるように設定されています。

認証されたユーザーには、ファイルを保管するための専用のスペースがあります。 ユーザーは、お互いのファイルを表示することはできませんが、署名付き URL を生成して、特定のファイルへの一時的なアクセス権限を付与できます。

アプリケーションについて詳しくは、[ソース・コード・リポジトリー](https://github.com/IBM-Cloud/secure-file-storage)を参照してください。

## セキュリティー・イベントのレビュー
アプリケーションとそのサービスが正常にデプロイされたため、そのプロセスによって生成されたセキュリティー・イベントをレビューできるようになりました。 すべてのイベントは、{{site.data.keyword.cloudaccesstrailshort}} インスタンスでのみ使用でき、[グラフィカル UI (Kibana)、CLI、または API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status) を介してアクセスできます。

1. [{{site.data.keyword.Bluemix_notm}} リソース・リスト](https://{DomainName}/resources)で {{site.data.keyword.cloudaccesstrailshort}} インスタンス **secure-file-storage-activity-tracker** を見つけ、そのダッシュボードを開きます。
2. デフォルトでは、**「管理」**タブには**「スペース・ログ (Space logs)」**が表示されます。**「ログの表示 (View logs)」**の横にあるセレクターをクリックして、**「アカウント・ログ (Account logs)」**に切り替えます。 複数のイベントが表示されます。
3. **「Kibana で表示 (View in Kibana)」**をクリックして、完全なイベント・ビューアーを開きます。
4. **「検出 (Discover)」**をクリックして、イベントの詳細をレビューします。
5. **「使用可能なフィールド (Available Fields)」**で **action_str** および **initiator.name_str** を追加します。
6. 三角形アイコンをクリックし、表形式または JSON 形式を選択して、必要なエントリーを展開します。

自動リフレッシュおよび表示される時刻範囲の設定を変更して、分析されるデータと分析方法を変更できます。
{: tip}

## オプション: カスタム・ドメインの使用およびネットワーク・トラフィックの暗号化
デフォルトでは、`containers.appdomain.cloud` のサブドメインの汎用ホスト名でアプリケーションにアクセスできます。 ただし、デプロイされたアプリとともにカスタム・ドメインを使用することもできます。 **https** の継続的なサポートについては、暗号化されたネットワーク・トラフィックを使用してアクセスし、必要なホスト名の証明書またはワイルドカード証明書を提供する必要があります。以下のセクションでは、既存の証明書を {{site.data.keyword.cloudcerts_short}} にアップロードし、クラスターにデプロイします。また、カスタム・ドメインを使用するアプリ構成を更新します。

[Let's Encrypt](https://letsencrypt.org/) から証明書を取得する方法の例は、この [{{site.data.keyword.cloud}} ブログ](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)で説明されています。
{: tip}

1. [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager) のインスタンスを作成します。
   * 名前を **secure-file-storage-certmgr** に設定します。
   * 他のサービスと同じ**場所**と**リソース・グループ**を使用します。
2. **「証明書のインポート (Import Certificate)」**をクリックして、証明書をインポートします。
   * 名前を **SecFileStorage** に設定し、説明を **Certificate for e2e security tutorial** に設定します。
   * **「参照」**ボタンを使用して、証明書ファイルをアップロードします。
   * **「インポート」**をクリックして、インポート処理を完了します。
3. インポートされた証明書のエントリーを見つけて、展開します。
   * ドメイン名がカスタム・ドメインと一致するかなど、証明書エントリーを確認します。 ワイルドカード証明書をアップロードした場合は、ドメイン名にアスタリスクが含まれます。
   * 証明書の **crn** の横の**コピー**・シンボルをクリックします。
4. コマンド・ラインに切り替えて、証明書の情報をシークレットとしてクラスターにデプロイします。 前のステップの crn でのコピーの後に、以下のコマンドを実行します。
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <the copied crn from previous step>
   ```
   {: codeblock}
   以下のコマンドを実行して、クラスターが証明書を認識していることを確認します。
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. ファイル `secure-file-storage.yaml` を編集します。
   * **Ingress** のセクションを見つけます。
   * カスタム・ドメインに関する行をアンコメントおよび編集し、ドメインおよびホスト名を入力します。
   カスタム・ドメインの CNAME エントリーは、クラスターを指している必要があります。 詳しくは、この資料の[カスタム・ドメインのマッピングに関するガイド](https://{DomainName}/docs/containers?topic=containers-ingress#private_3)を参照してください。
   {: tip}
6. 構成の変更をデプロイメントに適用します。
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. 再度ブラウザーに切り替えます。 [{{site.data.keyword.Bluemix_notm}} リソース・リスト](https://{DomainName}/resources)で、前に作成および構成した{{site.data.keyword.appid_short}} サービスを見つけ、その管理ダッシュボードを起動します。
   * **「ID プロバイダー」**の**「管理」**に移動し、**「設定」**に移動します。
   * **「Web リダイレクト URL の追加 (Add web redirect URLs)」**フォームで別の URL として `https://secure-file-storage.<your custom domain>/appid_callback` を追加します。
8. これですべてが配置されました。 構成したカスタム・ドメイン `https://secure-file-storage.<your custom domain>` でアプリにアクセスして、アプリをテストします。

## チュートリアルを発展させる

セキュリティーに終わりはありません。 以下の提案を試して、アプリケーションのセキュリティーを強化してください。

* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) を使用して、静的および動的なコード・スキャンを実行します
* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) のポリシーと規則を使用して、良質なコードのみがリリースされるようにします

## リソースの削除
{:removeresources}

リソースを削除するには、デプロイされたコンテナーを削除してから、プロビジョンされたサービスを削除します。

1. デプロイされたコンテナーを削除します。
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. デプロイメントのシークレットを削除します。
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. コンテナー・レジストリーから Docker イメージを削除します。
   ```sh
   ibmcloud cr image-rm <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. [{{site.data.keyword.Bluemix_notm}} リソース・リスト](https://{DomainName}/resources)で、このチュートリアルのために作成されたリソースを見つけます。検索ボックスおよび **secure-file-storage** をパターンとして使用します。各サービスの横のコンテキスト・メニューをクリックし、**「サービスの削除」**を選択して、各サービスを削除します。{{site.data.keyword.keymanagementserviceshort}} サービスは、キーが削除された後にのみ削除できます。サービス・インスタンスをクリックして、関連ダッシュボードを取得し、キーを削除します。

他のユーザーとアカウントを共有している場合は、常に、自分のリソースのみを削除するようにします。
{: tip}

## 関連コンテンツ
{:related}

* [{{site.data.keyword.security-advisor_short}} の資料](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [クラウド・アプリを保護およびモニターするためのセキュリティー](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [{{site.data.keyword.Bluemix_notm}} プラットフォーム・セキュリティー](https://{DomainName}/docs/overview?topic=overview-security#security)
* [IBM Cloud でのセキュリティー](https://www.ibm.com/cloud/security)
* [チュートリアル: ユーザー、チーム、アプリケーションを編成するためのベスト・プラクティス](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [ワイルドカード証明書を使用した IBM Cloud でのアプリの保護](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
