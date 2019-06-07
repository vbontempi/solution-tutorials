---
copyright:
  years: 2018, 2019
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

# 複数の地域にわたるサーバーレス・アプリのデプロイ
{: #multi-region-serverless}

このチュートリアルでは、IBM Cloud Internet Services と {{site.data.keyword.openwhisk_short}} を構成して複数の地域にわたってサーバーレス・アプリをデプロイする方法を示します。

サーバーレス・コンピューティング・プラットフォームは、サーバーなしで API を素早く作成する方法を開発者に提供します。 {{site.data.keyword.openwhisk}} は、アクションに対応した REST API の自動生成、HTTP エンドポイントへのアクションの転換、セキュア API 認証の使用可能化をサポートします。こうした機能は、外部利用者への API の公開だけでなく、マイクロサービス・アプリケーションの構築にも役立ちます。

{{site.data.keyword.openwhisk_short}} は、{{site.data.keyword.cloud_notm}} の複数の場所で使用できます。耐障害性を高め、ネットワーク待ち時間を短縮するために、アプリケーションはそのバックエンドを複数の場所にデプロイできます。その後、開発者は IBM Cloud Internet Services (CIS) を使用して、正常に機能している最も近いバックエンドにトラフィックを分散する責任を持つ単一のエントリー・ポイントを公開できます。

## 達成目標
{: #objectives}

* {{site.data.keyword.openwhisk_short}} アクションをデプロイする。
* {{site.data.keyword.APIM}} を介してカスタム・ドメインでアクションを公開する。
* Cloud Internet Services を使用して複数のロケーション間でトラフィックを分配する。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

このチュートリアルでは、{{site.data.keyword.openwhisk_short}} を使用してバックエンドが実装されるパブリック Web アプリケーションを取り上げます。ネットワーク待ち時間を短縮し、停止を回避するために、複数のロケーションにアプリケーションがデプロイされます。このチュートリアルでは、2 つのロケーションが構成されます。

<p style="text-align: center;">

  ![アーキテクチャー](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. ユーザーがアプリケーションにアクセスします。要求が Internet Services 経由で送信されます。
2. Internet Services によって、正常に機能している最も近い API バックエンドにユーザーが転送されます。
3. {{site.data.keyword.cloudcerts_short}} が API にその SSL 証明書を提供します。トラフィックはエンドツーエンドで暗号化されます。
4. {{site.data.keyword.openwhisk_short}} を使用して API が実装されます。

## 始める前に
{: #prereqs}

1. Cloud Internet Services を使用するには、カスタム・ドメインを所有している必要があります。Cloud Internet Services のネーム・サーバーを参照するようにそのドメインの DNS を構成するためです。ドメインを所有していない場合は、[godaddy.com](http://godaddy.com) などのレジストラーから購入できます。
1. [これらのステップに従って](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)、必要なコマンド・ライン (CLI) ツールをすべてインストールします。

## カスタム・ドメインの構成

最初のステップは、IBM Cloud Internet Services (CIS) のインスタンスを作成し、カスタム・ドメインが CIS ネーム・サーバーを指すようにすることです。

1. {{site.data.keyword.Bluemix_notm}} カタログの [Internet Services](https://{DomainName}/catalog/services/internet-services) にナビゲートします。
1. サービス名を設定し、**「作成」**をクリックして、サービスのインスタンスを作成します。このチュートリアルでは、任意の料金プランを使用できます。
1. サービス・インスタンスがプロビジョンされたら、**「始める」**をクリックしてドメイン・ネームを設定し、**「ドメインの追加」**をクリックします。
1. **「次のステップ」**をクリックします。ネーム・サーバーが割り当てられたら、そのリストされたネーム・サーバーを使用するようにレジストラーまたはドメイン・ネーム・プロバイダーを構成します。 
1. レジストラーまたは DNS プロバイダーを構成してから、その変更が反映されるまでに最大で 24 時間かかる場合があります。

   「概要」ページのドメインの状況が*「保留中」*から*「アクティブ」*に変わったら、`dig <your_domain_name> ns` コマンドを使用して、新しいネーム・サーバーが有効になったことを確認できます。
   {:tip}

### カスタム・ドメインの証明書の取得

カスタム・ドメインを介して {{site.data.keyword.openwhisk_short}} アクションを公開するためには、セキュア HTTPS 接続が必要になります。サーバーレス・バックエンドで使用する予定のドメインとサブドメインの SSL 証明書を取得する必要があります。*mydomain.com* のようなドメインと仮定すれば、アクションは *api.mydomain.com* でホストされることになります。*api.mydomain.com* の証明書が発行される必要があります。

[Let's Encrypt](https://letsencrypt.org/) から無料の SSL 証明書を入手できます。このプロセス中に、ドメインの所有者であることを証明するために、Cloud Internet Services の DNS インターフェースに TXT タイプの DNS レコードを構成する必要がある場合があります。
{:tip}

ドメインの SSL 証明書と秘密鍵を取得したら、それらを [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) フォーマットに必ず変換してください。

1. 証明書を PEM フォーマットに変換するには、次のようにします。
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. 秘密鍵を PEM フォーマットに変換するには、次のようにします。
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### 中央リポジトリーへの証明書のインポート

1. サポートされている場所で [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) インスタンスを作成します。
1. サービス・ダッシュボードで、**「証明書のインポート (Import Certificate)」**を次のように使用します。
   * **「名前」**をカスタムのサブドメインとドメイン (例えば *api.mydomain.com*) に設定します。
   * PEM フォーマットの**証明書ファイル**を参照します。
   * PEM フォーマットの**秘密鍵ファイル**を参照します。
   * **「インポート」**を実行します。

## 複数のロケーションへのアクションのデプロイ

このセクションでは、アクションを作成し、それらを API として公開して、その API にカスタム・ドメインをマップします。SSL 証明書は {{site.data.keyword.cloudcerts_short}} に保管されています。

<p style="text-align: center;">

  ![API アーキテクチャー](images/solution44-multi-region-serverless/api-architecture.png)
</p>

アクション **doWork** では、API 操作の 1 つが実装されます。アクション **healthz** は、後で API が正常かどうかを検査する際に使用されます。単に *OK* を返すだけのノーオペレーションである場合もあれば、API が必要とするデータベースなどの重要なサービスの ping のようなより複雑な検査を行う場合もあります。

アプリケーション・バックエンドをホストするロケーションごとに、以下の 3 つのセクションを繰り返す必要があります。このチュートリアルでは、ターゲットとして *Dallas (us-south)* と *London (eu-gb)* を選択できます。

### アクションの定義

1. [「{{site.data.keyword.openwhisk_short}}」/「アクション」](https://{DomainName}/openwhisk/actions)にアクセスします。
1. ターゲット・ロケーションに切り替え、アクションをデプロイする組織とスペースを選択します。
1. アクションを作成します。
   1. **「名前」**を **doWork** に設定します。
   1. **「パッケージを囲む」**を**「デフォルト」**に設定します。
   1. **「ランタイム」**を最新バージョンの **Node.js** に設定します。
   1. **作成**します。
1. アクション・コードを次のコードに変更します。
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. **保存**します。
1. API のヘルス・チェックとして使用されるもう 1 つのアクションを作成します。
   1. **「名前」**を **healthz** に設定します。
   1. **「パッケージを囲む」**を**「デフォルト」**に設定します。
   1. **「ランタイム」**を最新の **Node.js** に設定します。
   1. **作成**します。
1. アクション・コードを次のコードに変更します。
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. **保存**します。

### 管理 API でのアクションの公開

次のステップは、アクションを公開するための管理 API を作成することです。

1. [「{{site.data.keyword.openwhisk_short}}」/「API」](https://{DomainName}/openwhisk/apimanagement)にアクセスします。
1. 新しい管理対象 {{site.data.keyword.openwhisk_short}} API を作成します。
   1. **「API 名 (API name)」**を **App API** に設定します。
   1. **「基本パス (Base path)」**を **/api** に設定します。
1. 操作を作成します。
   1. **「パス」**を **/do** に設定します。
   1. **「Verb」**を **GET** に設定します。
   1. **「パッケージ」**を**「デフォルト」**に設定します。
   1. **「アクション」**を **doWork** に設定します。
   1. **作成**します。
1. もう 1 つの操作を作成します。
   1. **「パス」**を **/healthz** に設定します。
   1. **「Verb」**を **GET** に設定します。
   1. **「パッケージ」**を**「デフォルト」**に設定します。
   1. **「アクション」**を **healthz** に設定します。
   1. **作成**します。
1. API を**保存**します。

### 管理対象 API のカスタム・ドメインの構成

管理対象 API を作成すると、デフォルトのエンドポイント (例: `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`) が設定されます。このセクションでは、カスタム・サブドメイン (後で IBM Cloud Internet Services で構成されるドメイン) からの要求に対応できるようにこのエンドポイントを構成します。

1. [「API」/「カスタム・ドメイン」](https://{DomainName}/apis/domains)に移動します。
1. **「地域」**セレクターで、ターゲット・ロケーションを選択します。
1. アクションと管理対象 API を作成した組織とスペースにリンクされているカスタム・ドメインを見つけます。アクション・メニューで**「設定の変更 (Change Settings)」**をクリックします。
1. **デフォルト・ドメイン/別名**の値をメモします。
1. **「カスタム・ドメインの適用 (Apply custom domain)」**にチェック・マークを付けます。
   1. **「ドメイン・ネーム」**を、CIS グローバル・ロード・バランサーで使用するドメイン (例えば *api.mydomain.com*) に設定します。
   1. 証明書を保持している {{site.data.keyword.cloudcerts_short}} インスタンスを選択します。
   1. ドメインの証明書を選択します。
1. **Cloud Internet Services** のインスタンスのダッシュボードにアクセスして、**「信頼性」/「DNS」**で、新しい **DNS TXT レコード**を作成します。
   1. **「名前」**をカスタム・サブドメイン (例えば **api**) に設定します。
   1. **「コンテンツ」**を**デフォルト・ドメイン/別名**に設定します。
   1. レコードを保存します。
1. カスタム・ドメイン設定を保存します。ダイアログにより、DNS TXT レコードが存在するかどうかがチェックされます。

   TXT レコードが見つからない場合は、TXT レコードが反映されるまで待ってから、設定の保存を再試行することが必要な場合があります。設定が適用されたら、DNS TXT レコードを削除できます。
   {: tip}

上記のセクションを繰り返して、さらにロケーションを構成します。

## ロケーション間のトラフィックの分配

**この段階で複数のロケーションにアクションをセットアップしました**が、それらに到達するための単一のエントリー・ポイントがありません。このセクションでは、ロケーション間でトラフィックを分配するためのグローバル・ロード・バランサー (GLB) を構成します。

<p style="text-align: center;">

  ![グローバル・ロード・バランサーのアーキテクチャー](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### ヘルス・チェックの作成

Internet Services は、このエンドポイントを定期的に呼び出して、バックエンドの正常性を検査します。

1. IBM Cloud Internet Services インスタンスのダッシュボードにアクセスします。
1. **「信頼性」/「グローバル・ロード・バランサー」**で、ヘルス・チェックを作成します。
   1. **「モニター・タイプ」**を **HTTPS** に設定します。
   1. **「パス」**を **/api/healthz** に設定します。
   1. **リソースをプロビジョンします**。

### 起点プールの作成

ロケーションごとにプールを 1 つ作成することにより、後で、最も近いロケーションにユーザーをリダイレクトするようにグローバル・ロード・バランサーに地理的経路を構成できるようになります。別の選択肢は、すべてのロケーションを含む単一のプールを作成し、ロード・バランサーにプール内の起点を循環させることです。

ロケーションごとに以下のようにします。
1. 起点プールを作成します。
1. **「名前」**を **app-&lt;location&gt;** (例えば _app-Dallas_) に設定します。
1. 既に作成したヘルス・チェックを選択します。
1. **「ヘルス・チェック地域」**を、{{site.data.keyword.openwhisk_short}} がデプロイされたロケーションに近い地域に設定します。
1. **「起点名」**を **app-&lt;location&gt;** に設定します。
1. **「起点アドレス」**を管理 API のデフォルト・ドメイン/別名 (例えば _5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_) に設定します。
1. **リソースをプロビジョンします**。

### グローバル・ロード・バランサーの作成

1. ロード・バランサーを作成します。
1. **「バランサー・ホスト名」**を **api.mydomain.com** に設定します。
1. 地域の起点プールを追加します。
1. **リソースをプロビジョンします**。

しばらくしてから、`https://api.mydomain.com/api/do?name=John&place=Earth` にアクセスします。最初の正常なプールで実行されている機能に関する応答が返されます。

### フェイルオーバーのテスト

フェイルオーバーをテストするには、GLB が次の正常なプールにリダイレクトするように、プールのヘルス・チェックが失敗しなければなりません。障害をシミュレートするには、ヘルス・チェック機能を変更して失敗させます。

1. [「{{site.data.keyword.openwhisk_short}}」/「アクション」](https://{DomainName}/openwhisk/actions)にアクセスします。
1. GLB で最初に構成したロケーションを選択します。
1. `healthz` 機能を編集して、その実装を `throw new Error()` に変更します。
1. 保存します。
1. この起点プールに対してヘルス・チェックが実行されるまで待ちます。
1. 再度 `https://api.mydomain.com/api/do?name=John&place=Earth` にアクセスすると、今度は別の正常な起点にリダイレクトされます。
1. コード変更を元に戻して正常な起点に戻るようにします。

## リソースの削除
{: #removeresources}

### CIS リソースの削除

1. GLB を削除します。
1. 起点プールを削除します。
1. ヘルス・チェックを削除します。

### アクションの削除

1. [API](https://{DomainName}/openwhisk/apimanagement) を削除します。
1. [アクション](https://{DomainName}/openwhisk/actions)を削除します。

## 関連コンテンツ
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Cloud Internet Services による耐障害性のあるセキュアな複数地域 Kubernetes クラスター](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
