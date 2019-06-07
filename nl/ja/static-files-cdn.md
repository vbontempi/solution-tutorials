---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# CDN を使用した静的ファイルの配信の高速化
{: #static-files-cdn}

このチュートリアルでは、Web サイトの資産 (画像、ビデオ、ドキュメント) やユーザー生成コンテンツを {{site.data.keyword.cos_full_notm}} でホストして提供する方法と、[{{site.data.keyword.cdn_full}} (CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai) を使用して世界中のユーザーに高速かつセキュアに配信する方法について説明します。

Web アプリケーションには、HTML コンテンツ、画像、ビデオ、カスケーディング・スタイル・シート、JavaScript ファイル、ユーザー生成コンテンツなど、さまざまなタイプのコンテンツがあります。頻繁に変更されるコンテンツもあれば、あまり変更されないコンテンツもあります。多くのユーザーから非常に頻繁にアクセスされるコンテンツもあれば、まれにしかアクセスされないコンテンツもあります。アプリケーションのオーディエンスが増加したら、このようなコンテンツの提供を別のコンポーネントにオフロードして、メインのアプリケーションのためにリソースを解放することが必要になるはずです。また、アプリケーション・ユーザーが世界のどこにいようと、そのユーザーに近いロケーションからコンテンツを提供することも必要になるでしょう。

そのような状況では、次のような多くの理由から、コンテンツ配信ネットワークが使用されます。
* CDN はコンテンツをキャッシュに入れ、コンテンツがキャッシュにない場合や有効期限が切れている場合にのみ、コンテンツをオリジン (サーバー) からプルします。
* CDN は、世界中のいくつものデータ・センターを使用して、キャッシュに入れたコンテンツをユーザーの最も近いロケーションから提供します。
* メインのアプリケーションとは別のドメインで実行されるので、ブラウザーで多くのコンテンツを並行してロードできます (ほとんどのブラウザーには、ホスト名あたりの接続数に制限があります)。

## 達成目標
{: #objectives}

* {{site.data.keyword.cos_full_notm}} バケットにファイルをアップロードする。
* コンテンツ・デリバリー・ネットワーク (CDN) を使用してコンテンツをグローバルに提供する。
* Cloud Foundry Web アプリケーションを使用してファイルを公開する。

## 使用するサービス
{: #services}

このチュートリアルでは、以下の製品を使用します。
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー

<p style="text-align: center;">
![アーキテクチャー](images/solution3/Architecture.png)
</p>

1. ユーザーがアプリケーションにアクセスします
2. アプリケーションには、コンテンツ・デリバリー・ネットワークで配信されるコンテンツが含まれています
3. コンテンツが CDN にない場合、または有効期限が切れている場合は、CDN がオリジンからコンテンツをプルします

## 始める前に
{: #prereqs}

インフラストラクチャー・アカウントのマスター・ユーザーに連絡して、以下の許可を取得します。
   * CDN アカウントの管理
   * ストレージの管理
   * API キー

ストレージ・サービスおよび CDN サービスを表示および使用するために、これらの許可が必要です。

## Web アプリケーション・コードを取得する
{: #get_code}

画像、ビデオ、カスケーディング・スタイル・シートなどのさまざまなタイプのコンテンツを使用する単純な Web アプリケーションについて考えてみましょう。コンテンツをストレージ・バケットに保管して、バケットをオリジンとして使用するように CDN を構成します。

まずは、アプリケーション・コードを取得します。

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## オブジェクト・ストレージを作成する
{: #create_cos}

{{site.data.keyword.cos_full_notm}} は、非構造化データのための柔軟でコスト効率が高いスケーラブルなクラウド・ストレージです。

1. コンソールで[カタログ](https://{DomainName}/catalog/)に移動し、「ストレージ」セクションから[**「オブジェクト・ストレージ (Object Storage)」**](https://{DomainName}/catalog/services/cloud-object-storage)を選択します。
2. {{site.data.keyword.cos_full_notm}} の新規インスタンスを作成します。
4. サービス・ダッシュボードで、**「バケットの作成」**をクリックします。
5. `username-mywebsite` などの固有のバケット名を設定して、**「作成」**をクリックします。バケット名にドット (.) は使用しないでください。

## バケットにファイルをアップロードする
{: #upload}

このセクションでは、コマンド・ライン・ツール **curl** を使用して、バケットにファイルをアップロードします。

1. CLI から {{site.data.keyword.Bluemix_notm}} にログインして、IBM Cloud IAM からトークンを取得します。
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. 前の手順のコマンドの出力からトークンをコピーします。
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. トークンの値とバケット名を環境変数に設定すると、簡単にアクセスできるようになります。
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. 前にダウンロードした Web アプリケーション・コードの content ディレクトリーから、`a-css-file.css`、`a-picture.png`、および `a-video.mp4` という名前のファイルをアップロードします。ファイルは、バケットのルートにアップロードします。
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. ダッシュボードでファイルを確認します。
   ![](images/solution3/Buckets.png)
6. 次の例のようなリンクを使用して、ブラウザーでファイルにアクセスします。

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## CDN を使用してファイルをグローバルに提供する

このセクションでは、CDN サービスを作成します。CDN サービスは、必要とされる場所にコンテンツを配信します。コンテンツは、初めて要求されたときにホスト・サーバー ({{site.data.keyword.cos_full_notm}} 内のバケット) からネットワークにプルされた後、ネットワーク上に保持されるので、他のユーザーはすぐにアクセスできます。もう一度ホスト・サーバーにアクセスしてネットワーク待機時間が発生することがありません。

### CDN インスタンスを作成する

1. コンソールでカタログに移動し、「ネットワーク」セクションから**「Content Delivery Network」**を選択します。この CDN には Akamai が採用されています。**「作成」**をクリックします。
2. 次のダイアログで、CDN の**「ホスト名」**にカスタム・ドメインを設定します。カスタム・ドメインを設定しても、IBM 提供の CNAME を介して CDN コンテンツにアクセスすることができます。そのため、カスタム・ドメインを使用しない場合は、任意の名前を設定できます。
3. **「カスタム CNAME (Custom CNAME)」**接頭部に固有値を設定します。
4. 次は、**「オリジンの構成 (Configure your origin)」**の下で、**「オブジェクト・ストレージ (Object Storage)」**を選択して、CDN に COS を構成します。
5. **「エンドポイント」**にバケットの API エンドポイント (*s3-api.us-geo.objectstorage.softlayer.net* など) を設定します。
6. **「ホスト・ヘッダー (Host header)」**および**「パス」**は空のままにします。**「バケット名 (Bucket name)」**に *your-bucket-name* を設定します。
7. HTTP (80) ポートと HTTPS (443) ポートの両方を有効にします。
8. カスタム・ドメインを使用する場合は、**「SSL 証明書 (SSL certificate)」**で*「DV SAN 証明書 (DV SAN Certificate)」*を選択します。あるいは、CNAME を介してストレージにアクセスする場合は、「**ワイルドカード証明書* (**Wildcard Certificate*)」オプションを選択します。
9. **「作成」**をクリックします。

### CDN CNAME を介してコンテンツにアクセスする

1. [リストで](https://{DomainName}/classic/network/cdn)、CDN インスタンスを選択します。
2. 先ほど *DV SAN 証明書* を選択した場合は、初期セットアップの完了後に、ドメインの検証を求めるプロンプトが出されます。**「ドメイン検証の表示 (View domain validation)」**をクリックして表示された手順に従ってください。
3. **「詳細」**パネルに、CDN の**「ホスト名」**と**「CNAME」**の両方が表示されます。
4. `https://your-cdn-cname.cdnedge.bluemix.net/a-picture.png`、または、カスタム・ドメインを使用している場合は `https://your-cdn-hostname/a-picture.png` を使用してファイルにアクセスします。ファイル名を省略すると、代わりに S3 ListBucketResult が表示されます。

## Cloud Foundry アプリケーションをデプロイする

このアプリケーションの public/index.html の Web ページには、現在 {{site.data.keyword.cos_full_notm}} でホストしているファイルの参照が含まれています。バックエンドの `app.js` が、この Web ページを処理し、プレースホルダーを CDN の実際のロケーションに置き換えます。このようにして、Web ページで使用するすべての資産が CDN から提供されます。

1. 端末から、コードをチェックアウトしたディレクトリーに移動します。
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. アプリケーションを起動せずに、プッシュします。
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. アプリが CDN コンテンツを参照できるように、CDN_NAME 環境変数を構成します。
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. アプリを起動します。
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. Web ブラウザーでアプリにアクセスします。ページ・スタイルシート、画像、およびビデオが CDN からロードされます。

![](images/solution3/Application.png)

CDN と {{site.data.keyword.cos_full_notm}} の併用は、ファイルをホストして世界中のユーザーに提供できる強力な組み合わせです。{{site.data.keyword.cos_full_notm}} を使用して、ユーザーがアプリケーションにアップロードしたファイルを保管することもできます。

## リソースの削除

* Cloud Foundry アプリケーションを削除する
* Content Delivery Network サービスを削除する
* {{site.data.keyword.cos_full_notm}} サービスまたはバケットを削除する

## 関連コンテンツ

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[{{site.data.keyword.cos_full_notm}} へのアクセスの管理](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[CDN の概説](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
