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


# サーバーレス Web アプリケーションと API
{: #serverless-api-webapp}

このチュートリアルでは、GitHub ページで静的 Web サイト・コンテンツをホスティングし、{{site.data.keyword.openwhisk}} を使用してアプリケーション・バックエンドを実装することでサーバーレス Web アプリケーションを作成します。

{{site.data.keyword.openwhisk_short}} はイベント・ドリブン・プラットフォームとして、[さまざまなユース・ケース](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)をサポートします。Web アプリケーションと API の構築はその 1 つです。Web アプリでは、イベントとは Web ブラウザー (または REST クライアント) と Web アプリの間のやりとり、つまり HTTP 要求です。バックエンドをデプロイするために仮想マシン、コンテナー、または Cloud Foundry ランタイムをプロビジョンする代わりに、サーバーレス・プラットフォームを使用してバックエンド API を実装できます。これは、アイドル時間にかかる費用の発生を防ぎ、プラットフォームを必要に応じてスケーリングするための適切な解決策となる可能性があります。

{{site.data.keyword.openwhisk_short}} のすべてのアクション (または機能) を、Web クライアントがすぐに利用できる HTTP エンドポイントに変換できます。これらのアクションが Web 対応になると、*Web アクション*と呼ばれます。Web アクションが準備できたら、これらの Web アクションをまとめて、API ゲートウェイを備えた全機能 API にすることができます。API ゲートウェイは、API を公開するための {{site.data.keyword.openwhisk_short}} のコンポーネントです。セキュリティー、OAuth サポート、速度制限、およびカスタム・ドメイン・サポートを備えています。

## 達成目標

* サーバーレス・バックエンドとデータベースをデプロイする
* REST API を公開する
* 静的 Web サイトをホスティングする
* オプション: REST API にカスタム・ドメインを使用する

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

このチュートリアルに示すアプリケーションは、ユーザーがメッセージを投稿できる単純なゲストブック Web サイトです。

<p style="text-align: center;">

   ![アーキテクチャー](./images/solution8/Architecture.png)
</p>

1. ユーザーが GitHub Pages でホストされているアプリケーションにアクセスします。
2. Web アプリケーションがバックエンド API を呼び出します。
3. バックエンド API が API ゲートウェイで定義されます。
4. API ゲートウェイにより要求が [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) に転送されます。
5. {{site.data.keyword.openwhisk_short}} アクションが [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) を使用してゲストブック・エントリーを保管および取得します。

## 始める前に
{: #prereqs}

このガイドでは、静的 Web サイトのホスティングに GitHub Pages を使用します。パブリック GitHub アカウントを所有していることを確認してください。

## Guestbook データベースの作成

最初に {{site.data.keyword.cloudant_short_notm}} を作成します。{{site.data.keyword.cloudant_short_notm}} は管理不要のデータ・レイヤーで、柔軟な JSON スキーマを活用する最新の Web およびモバイルのアプリケーション向けに設計されています。{{site.data.keyword.cloudant_short_notm}} は Apache CouchDB に基づいて構築され、Apache CouchDB と互換であり、アプリケーションの進化とともに拡大するセキュア HTTPS API を介してアクセス可能です。

1. カタログで**「Cloudant」**を選択します。
2. サービス名を **guestbook-db** に設定し、認証方式として**「レガシー資格情報と IAM の両方を使用する (Use both legacy credentials and IAM)」**を選択し、**「作成」**をクリックします。
3. リソース・リストに戻り、「名前」列の**「*guestbook-db」**エントリーをクリックします。注: 場合によってはサービスのプロビジョンが完了するまで待つ必要があります。 
4. サービス詳細画面で***「Cloudant ダッシュボードを起動 (Launch Cloudant Dashboard)」***をクリックします。このダッシュボードが別のブラウザー・タブで開きます。注: Cloudant インスタンスへのログインが必要な場合があります。 
5. ***「データベースの作成 (Create Database)」***をクリックし、***guestbook*** という名前のデータベースを作成します。

  ![](images/solution8/Create_Database.png)

6. サービス詳細タブに戻り、**「サービス資格情報」**で次の手順に従います。
   1. **新しい資格情報**を作成し、デフォルト値を受け入れ、**「追加」**をクリックします。
   2. 「アクション」の下の**「資格情報の表示」**をクリックします。これらの資格情報は、後で Cloud Functions のアクションに対し Cloudant サービスに対する読み取り/書き込みを許可するために必要です。

## サーバーレス・アクションの作成

このセクションでは、サーバーレス・アクション (一般に「機能」と呼ばれます) を作成します。{{site.data.keyword.openwhisk}} (Apache OpenWhisk に基づく) は、着信イベントに応じて機能を実行する Function-as-a-Service (FaaS) プラットフォームです。使用していないときは料金はかかりません。

![](images/solution8/Functions.png)

### ゲストブック・エントリーを保存するためのアクション・シーケンス

**シーケンス**を作成します。シーケンスとは、あるアクションの出力が次のアクションの入力となるアクションの連鎖です。作成する最初のシーケンスは、ゲスト・メッセージを保持するために使用されます。名前、メール ID、およびコメントが指定されると、このシーケンスでは次の処理が行われます。
   * 保持する文書を作成します。
   * 文書を {{site.data.keyword.cloudant_short_notm}} データベースに保管します。

最初に、1 番目のアクションを作成します。

1. **Functions** https://{DomainName}/openwhisk に切り替えます。
2. 左側のペインで**「アクション」**をクリックし、次に**「作成」**をクリックします。
3. `prepare-entry-for-save` という名前の**アクションを作成し**、ランタイムとして **Node.js** を選択します (注: 最新バージョンを選択してください)。
4. 既存のコードを、以下のコード・スニペットに置き換えます。
   ```js
   /**
    * Prepare the guestbook entry to be persisted
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. **保存**します。

次に、シーケンスにアクションを追加します。

1. **「エンクロージング・シーケンス」**をクリックしてから、**「シーケンスに追加」**をクリックします。
1. シーケンス名として `save-guestbook-entry-sequence` を入力し、**「作成して追加 (Create and Add)」**をクリックします。

最後に、シーケンスに 2 番目のアクションを追加します。

1. **save-guestbook-entry-sequence** をクリックしてから、**「追加」**をクリックします。
1. **「パブリックの使用」**、**「Cloudant」**を選択し、次に**「アクション」**の下の**「create-document」**を選択します。
1. **新しいバインディング**を作成します。 
1. 名前として `binding-for-guestbook` を入力します。
1. Cloudant インスタンスで`「自分の資格情報の入力」`を選択し、キャプチャーした Cloudant サービスの資格情報を「ユーザー名」、「パスワード」、「ホスト」に入力し、「データベース」を `guestbook` に設定します。次に**「追加」**をクリックしてから**「保存」**をクリックします。
   {: tip}
1. テストのため、**「入力の変更」**をクリックして以下の JSON を入力します。
```json
    {
      "name": "John Smith",
      "email": "john@smith.com",
      "comment": "this is my comment"
    }
    ```
    {: codeblock}
1. **「適用」**をクリックしてから、**「起動」**をクリックします。
    ![](images/solution8/Save_Entry_Invoke.png)

### エントリーを取得するためのアクション・シーケンス

2 番目のシーケンスは、既存のゲストブック・エントリーを取得するために使用されます。このシーケンスでは以下の操作が行われます。
   * データベースのすべての文書をリストします。
   * 文書をフォーマットして返します。

1. **「機能 (Functions)」**の下で**「アクション」**をクリックし、新しい Node.js アクションを**作成**し、`set-read-input` という名前を付けます。
2. 既存のコードを、以下のコード・スニペットに置き換えます。このアクションは、適切なパラメーターを次のアクションに渡します。
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. **保存**します。

アクションをシーケンスに追加します。

1. **「エンクロージング・シーケンス」**、**「シーケンスに追加」**、および**「新規作成」**の順にクリックします。
1. **「アクション名」**に `read-guestbook-entries-sequence` と入力し、**「作成して追加 (Create and Add)」**をクリックします。

シーケンスを完成させます。

1. Cloudant から文書を取得する 2 番目のアクションを作成して追加するため、**read-guestbook-entries-sequence** シーケンスをクリックし、**「追加」**をクリックします。
1. **「パブリックの使用」**で**「{{site.data.keyword.cloudant_short_notm}}」**を選択し、次に**「list-documents」**を選択します
1. **「マイ・バインディング」**で**「binding-for-guestbook」**を選択し、**「追加」**をクリックします。このパブリック・アクションが作成され、シーケンスに追加されます。
1. {{site.data.keyword.cloudant_short_notm}} の文書をフォーマットする 3 番目のアクションを作成して追加するため、**「追加」**をもう一度クリックします。
1. **「新規作成」**で名前として `format-entries` を入力し、**「作成して追加 (Create and Add)」**をクリックします。
1. **「format-entries」**をクリックし、コードを以下のコードに置き換えます。
   ```js
   const md5 = require('spark-md5');

   function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. **保存**します。
1. シーケンスを選択するため、**「アクション」**をクリックして**「read-guestbook-entries-sequence」**をクリックします。
1. **「保存」**をクリックしてから**「起動」**をクリックします。出力は以下のようになります。
   ![](images/solution8/Read_Entries_Invoke.png)

## API の作成

![](images/solution8/Cloud_Functions_API.png)

1. アクションに移動します (https://{DomainName}/openwhisk/actions)。
2. **read-guestbook-entries-sequence** シーケンスを選択します。名前の横にある**「Web アクション」**をクリックし、**「Web アクションを有効にする (Enable Web Action)」**にチェック・マークを付け、**「保存」**をクリックします。
3. **save-guestbook-entry-sequence** シーケンスでも同じ作業を行います。
4. API (https://{DomainName}/openwhisk/apimanagement) に移動し、**{{site.data.keyword.openwhisk_short}} API を作成します**。
5. 名前を `guestbook` に設定し、基本パスを `/guestbook` に設定します
6. **「操作の作成 (Create operation)」**をクリックし、ゲストブック・エントリーを取得する操作を作成します。
   1. **path** を `/entries` に設定します。
   2. **verb** を `GET*` に設定します。
   3. **read-guestbook-entries-sequence** アクションを選択します。
7. **「操作の作成 (Create operation)」**をクリックし、ゲストブック・エントリーを保持する操作を作成します。
   1. **path** を `/entries` に設定します。
   2. **verb** を `PUT` に設定します。
   3. **save-guestbook-entry-sequence** アクションを選択します。
8. API を保存して公開します。

## Web アプリのデプロイ

1. Guestbook ユーザー・インターフェース・リポジトリー (https://github.com/IBM-Cloud/serverless-guestbook) をパブリック GitHub にフォークします。
2. **docs/guestbook.js** を変更し、**apiUrl** の値を API ゲートウェイにより指定されるルートに置き換えます。
3. 変更したファイルを、フォークしたリポジトリーにコミットします。
4. リポジトリーの「設定」ページで**「GitHub ページ (GitHub Pages)」**にスクロールし、ソースを **master branch /docs フォルダー**に変更して保存します。
5. リポジトリーのパブリック・ページにアクセスします。
6. 前に作成した「test」ゲストブック・エントリーが表示されます。
7. 新しいエントリーを追加します。

![](images/solution8/Guestbook.png)

## オプション: API に独自のドメインを使用する

管理対象 API を作成すると、デフォルトのエンドポイント (例: `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`) が設定されます。このセクションでは、このエンドポイントを構成して、カスタム・サブドメインから発信される要求を処理できるようにします。

### カスタム・ドメインの証明書の取得

カスタム・ドメインを介して {{site.data.keyword.openwhisk_short}} アクションを公開するためには、セキュア HTTPS 接続が必要になります。サーバーレス・バックエンドで使用する予定のドメインとサブドメインの SSL 証明書を取得する必要があります。*mydomain.com* のようなドメインの場合、*guestbook-api.mydomain.com* でアクションをホストできる可能性があります。*guestbook-api.mydomain.com* (または **.mydomain.com*) の証明書を発行する必要があります。

[Let's Encrypt](https://letsencrypt.org/) から無料の SSL 証明書を入手できます。このプロセスでは、ドメインの所有者であることを証明するため、DNS インターフェースでタイプ TXT の DNS レコードを構成する必要がある場合があります。
{:tip}

ドメインの SSL 証明書と秘密鍵を取得したら、それらを [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) フォーマットに必ず変換してください。

1. 証明書を PEM フォーマットに変換するには、次のようにします。
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. 秘密鍵を PEM フォーマットに変換するには、次のようにします。
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### 中央リポジトリーへの証明書のインポート

1. サポートされている場所で [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) インスタンスを作成します。
1. サービス・ダッシュボードで、**「証明書のインポート (Import Certificate)」**を次のように使用します。
   * **「名前」**をカスタム・サブドメインとドメイン (例: `guestbook-api.mydomain.com`) に設定します。
   * PEM フォーマットの**証明書ファイル**を参照します。
   * PEM フォーマットの**秘密鍵ファイル**を参照します。
   * **「インポート」**を実行します。

### 管理対象 API のカスタム・ドメインの構成

1. [「API」/「カスタム・ドメイン」](https://{DomainName}/apis/domains)に移動します。
1. 「地域」セレクターで、アクションをデプロイするロケーションを選択します。
1. アクションと管理対象 API を作成した組織とスペースにリンクされているカスタム・ドメインを見つけます。アクション・メニューで**「設定の変更 (Change Settings)」**をクリックします。
1. **デフォルト・ドメイン/別名**の値をメモします。
1. **「カスタム・ドメインの適用 (Apply custom domain)」**にチェック・マークを付けます。
   1. **「ドメイン・ネーム」**に、使用するドメイン (例: `guestbook-api.mydomain.com`) を設定します。
   1. 証明書を保持している {{site.data.keyword.cloudcerts_short}} インスタンスを選択します。
   1. ドメインの証明書を選択します。
1. DNS プロバイダーに移動し、ドメインを API デフォルト・ドメイン / 別名にマップする新しい **DNS TXT レコード**を作成します。設定が適用されたら、DNS TXT レコードを削除できます。
1. カスタム・ドメイン設定を保存します。ダイアログにより、DNS TXT レコードが存在するかどうかがチェックされます。
1. 最後に、DNS プロバイダーの設定に戻り、カスタム・ドメイン (例:guestbook-api.mydomain.com) をデフォルト・ドメイン / 別名に指し示す CNAME レコードを作成します。これにより、カスタム・ドメインを通過するトラフィックがバックエンド API にルーティングされます。

DNS の変更が伝搬されたら、https://guestbook-api.mydomain.com/guestbook の Guestbook API にアクセスできます。

1. **docs/guestbook.js** を編集し、**apiUrl** の値を https://guestbook-api.mydomain.com/guestbook に更新します。
1. 変更したファイルをコミットします。
1. これで、アプリケーションはカスタム・ドメインを介して API にアクセスします。

## リソースの削除

* {{site.data.keyword.cloudant_short_notm}} サービスを削除します。
* {{site.data.keyword.openwhisk_short}} から API を削除します。
* {{site.data.keyword.openwhisk_short}} からアクションを削除します。

## 関連コンテンツ
* [サーバーレスに関するその他のガイドおよびサンプル](https://developer.ibm.com/code/journey/category/serverless/)
* [{{site.data.keyword.openwhisk}} の概説](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [{{site.data.keyword.openwhisk}} の一般的なユース・ケース](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [アクションからの REST API の作成](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
