---
copyright:
  years: 2017, 2019
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

# REST API の作成、保護および管理
{: #create-manage-secure-apis}

このチュートリアルでは、LoopBack Node.js API フレームワークを使用して REST API を作成する方法について説明します。Loopback を使用すると、デバイスとブラウザーをデータやサービスに接続する REST API を迅速に作成できます。また、{{site.data.keyword.apiconnect_long}} を使用して、管理、可視性、セキュリティー、および速度制限を API に追加します。
{:shortdesc}

## 達成目標

* コーディングをまったく、またはほとんどせずに REST API を作成します。
* {{site.data.keyword.Bluemix_notm}} 上で API を公開して開発者が利用できるようにします。
* 既存の API を {{site.data.keyword.apiconnect_short}} に追加します。
* SoR (Systems of Record、定型業務処理システム) へのアクセスを安全に公開し、制御します。

## 使用するサービス

このチュートリアルでは、以下のランタイムおよびサービスを使用します。

* [Loopback](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* [SDK for Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry アプリ

## アーキテクチャー

![アーキテクチャー](images/solution13/Architecture.png)

1. 開発者が RESTful API を定義
2. 開発者が API を {{site.data.keyword.apiconnect_long}} に公開
3. ユーザーとアプリケーションが API を利用

## 始める前に

* [Node.js](https://nodejs.org/en/download/) バージョン 6.x をダウンロードしてインストールします (より新しいバージョンの Node.js が既にインストールされている場合は、[nvm](https://github.com/creationix/nvm) などを使用します)。

## Node.js での REST API の作成

{: #create_api}
このセクションでは、[LoopBack](https://loopback.io/doc/index.html) を使用して Node.js で API を作成します。LoopBack は、拡張性の高いオープン・ソースの Node.js フレームワークで、コーディングをまったく、またはほとんどせずに動的なエンドツーエンドの REST API を作成できます。

### アプリケーションの作成

1. {{site.data.keyword.apiconnect_short}} コマンド・ライン・ツールをインストールします。{{site.data.keyword.apiconnect_short}} のインストールに問題がある場合は、コマンドの前に `sudo` を使用するか、[こちらの](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit)指示に従います。
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. 以下のコマンドを入力して、アプリケーションを作成します。
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. `Enter` キーを押して、**entries-api** を**アプリケーションの名前**として使用します。
4. `Enter` キーを押して、**entries-api** を**プロジェクトを格納するディレクトリー**として使用します。
5. **LoopBack のバージョン**として **3.x (current)** を選択します。
6. **アプリケーションの種類**として **empty-server** を選択します。

![APIC Loopback スキャフォールド](images/solution13/apic_loopback.png)

### データ・ソースの追加

データ・ソースは、データベース、外部 REST API、SOAP Web サービス、ストレージ・サービスなどのバックエンド・システムを表します。一般的には、データ・ソースは作成、取得、更新、および削除 (CRUD) 機能を提供します。Loopback は多くのタイプの[データ・ソース](http://loopback.io/doc/en/lb3/Connectors-reference.html)をサポートしていますが、簡単にするために、メモリー内のデータ・ストアを API で使用します。

1. ディレクトリーを新規プロジェクトに変更し、API Designer を起動します。
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. ナビゲーションで**「データ・ソース」**をクリックします。次に、**「追加 (Add)」**ボタンをクリックします。**「新規ループバック・データ・ソース」**ダイアログが開きます。
3. **「名前」**テキスト・フィールドに `entriesDS` と入力し、**「新規」**ボタンをクリックします。
4. **「コネクター」**コンボ・ボックスから**「メモリー内データベース」**を選択します。
5. 左上で**「すべてのデータ・ソース」**をクリックします。データ・ソースがデータ・ソースのリストに表示されます。

   エディターにより、server/datasources.json ファイルが新しいデータ・ソースの設定で自動的に更新されます。
   {:tip}

![API Designer データ・ソース](images/solution13/datastore.png)

### モデルの追加

モデルは、ノードと REST API の両方を含む JavaScript オブジェクトで、バックエンド・システム内のデータを表します。モデルは、データ・ソース経由でこれらのシステムに接続されます。このセクションでは、データの構造を定義し、前に作成したデータ・ソースに接続します。

1. ナビゲーションから、**「モデル」**をクリックします。次に、**「追加 (Add)」**ボタンをクリックします。**「新規ループバック・モデル」**ダイアログが開きます。
2. **「名前」**テキスト・フィールドに `entry` と入力し、**「新規」**ボタンをクリックします。
3. **「データ・ソース」**コンボ・ボックスから**「entriesDS」**を選択します。
4. **「プロパティー」**カードから、**「プロパティーの追加」**アイコンを使用して以下のプロパティーを追加します。
    1. **「プロパティー名」**に `name` と入力し、**「タイプ」**コンボ・ボックスから**「ストリング」**を選択します。
    2. **「プロパティー名」**に `email` と入力し、**「タイプ」**コンボ・ボックスから**「ストリング」**を選択します。
    3. **「プロパティー名」**に `comment` と入力し、**「タイプ」**コンボ・ボックスから**「ストリング」**を選択します。
5. 右上にある**「保存」**アイコンをクリックしてモデルを保存します。

![モデル・ジェネレーター](images/solution13/models.png)

## LoopBack アプリケーションのテスト

このセクションでは、Loopback アプリケーションのローカル・インスタンスを開始し、API Designer を使用してデータを挿入および照会することで API をテストします。ブラウザーで REST エンドポイントをテストするには、API Designer の「探索」ツールを使用する必要があります。これは、ここに適切なセキュリティー・ヘッダーや他の要求パラメーターが含まれているためです。

1. 左下隅にある**「開始」**アイコンをクリックしてローカル・サーバーを始動し、**「実行中」**メッセージを待ちます。
2. バナーの**「探索」**リンクをクリックして、API Designer の「探索」ツールを表示します。サイドバーに、API 内の LoopBack モデルに使用できる REST 操作が表示されます。
3. 左側のペインにある**「entry.create」**操作をクリックして、エンドポイントを表示します。中央のペインには、エンドポイントの概要 (パラメーター、セキュリティー、モデル・インスタンス・データ、応答コードなど) が表示されます。右側のペインには、cURL コマンドを使用してエンドポイントを呼び出すためのサンプル・コードと、Ruby、Python、Java、Node などの言語が表示されます。
4. 右側のペインで**「試す (Try it)」**リンクをクリックします。**「パラメーター」**までスクロールダウンし、**「data」**テキスト域に以下を入力します。
    ```javascript
    {
      "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. **「操作の呼び出し」**ボタンをクリックします。
  ![API Designer からの API のテスト](images/solution13/data_entry_1.png)
6. **「応答コード: 200 OK (Response Code: 200 OK)」**を確認して、POST が成功したことを確認します。localhost の信頼できない証明書が原因でエラー・メッセージが表示された場合は、API Designer の「探索」ツールで、エラー・メッセージに記載されているリンクをクリックし、証明書を受け入れてください。そうすれば、Web ブラウザーでの操作の呼び出しを続行できます。 正確な手順は、ご使用の Web ブラウザーによって異なります。
7. cURL を使用して別のエントリーを追加します。ポートがアプリケーションのポートに一致していることを確認します。
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. **「entry.find」**操作をクリックしてから**「試す (Try It)」**リンクをクリックすると、**「操作の呼び出し」**ボタンが表示されます。応答によってすべてのエントリーが表示されます。**Jane Doe** と **John Doe** の JSON が表示されます。
  ![entry_find](images/solution13/find_response.png)

`entries-api` ディレクトリーから `npm start` コマンドを発行して、Loopback アプリケーションを手動で開始することもできます。REST API は [http://localhost:3000/api/entries](http://localhost:3000/api/entries) で使用できます。ただし、{{site.data.keyword.apiconnect_short}} によって管理および保護されません。
{:tip}

## {{site.data.keyword.apiconnect_short}} サービスの作成

次のステップの準備をするために、{{site.data.keyword.Bluemix_notm}} 上に **{{site.data.keyword.apiconnect_short}}** サービスを作成します。{{site.data.keyword.apiconnect_short}} は、API へのゲートウェイとして機能し、管理、セキュリティー、およびレート制限も提供します。

1. {{site.data.keyword.Bluemix_notm}} [ リソース・リスト](https://{DomainName}/resources)を起動します。
2. **「カタログ」>「統合」>「{{site.data.keyword.apiconnect_short}}**」にナビゲートし、**「作成」**ボタンをクリックします。

## {{site.data.keyword.Bluemix_notm}} への API の公開

{: #publish}
API Designer を使用してアプリケーションを Cloud Foundry アプリケーションとして {{site.data.keyword.Bluemix_notm}} にデプロイし、API 定義を **{{site.data.keyword.apiconnect_short}}** に公開します。API Designer はローカル・ツールキットです。閉じた場合は、プロジェクト・ディレクトリーから `apic edit` で再起動します。

アプリケーションは、`ibmcloud cf push` コマンドを使用して手動でデプロイすることもできますが、その場合は保護されません。{{site.data.keyword.apiconnect_short}} に [API をインポート](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing)するには、`definitions` フォルダーで使用可能な OpenAPI 定義ファイルを使用します。API Designer を使用してデプロイすると、アプリケーションは保護され、定義が自動的にインポートされます。
{:tip}

1. API Designer に戻り、バナーの**「公開」**リンクをクリックします。次に、**「ターゲットの追加および管理」>「IBM Bluemix ターゲットの追加」**をクリックします。
2. 公開先の**地域**および**組織**を選択します。
3. **「サンドボックス」**カタログを選択し、**「次へ」**をクリックします。
4. **「新規アプリケーション名の入力 (Type a new application name)」**テキスト・フィールドに `entries-api-application` と入力し、**「+」**アイコンをクリックします。
5. リストの**「entries-api-application」**をクリックし、**「保存」**をクリックします。
6. バナーの左上隅にあるハンバーガーの**「メニュー」**アイコンをクリックします。次に、**「プロジェクト」**および**「entries-api」**リスト項目をクリックします。
7. API Designer UI で、**「API」>「entries-api」>「アセンブル」**リンクをクリックします。
8. アセンブリー・エディターで、**「ポリシーのフィルター」**アイコンをクリックします。
9. **「DataPower Gateway ポリシー」**を選択し、**「保存」**をクリックします。
10. 上部のバーにある**「公開」**をクリックし、ターゲットを選択します。**「アプリケーションの公開」**を選択し、「製品のステージングまたは公開」>「**特定の製品**を選択」>**「entries-api」**を選択します。
11. **「公開」**をクリックし、アプリケーションの公開が終了するまで待ちます。
    ![公開ダイアログ](images/solution13/publish.png)

    アプリケーションには、API に関連する Loopback モデル、データ・ソース、およびコードが含まれています。製品では、API を開発者が使用できるようにする方法を宣言できます。
    {:tip}

API アプリケーションが Cloud Foundry アプリケーションとして {{site.data.keyword.Bluemix_notm}} に公開されました。このアプリケーションは、{{site.data.keyword.Bluemix_notm}} [リソース・リスト](https://{DomainName}/resources)で Cloud Foundry アプリケーションを参照することによって確認できますが、アプリケーションは保護されているため、URL を使用して直接アクセスすることはできません。次のセクションでは、管理対象 API にアクセスする方法を示します。

## API ゲートウェイ

これまでは、API をローカルで設計およびテストしました。このセクションでは、{{site.data.keyword.apiconnect_short}} を使用して、{{site.data.keyword.Bluemix_notm}} 上でデプロイ済み API をテストします。

1. {{site.data.keyword.Bluemix_notm}} [リソース・リスト](https://{DomainName}/resources)を起動します。
2. **「Cloud Foundry サービス」**で **{{site.data.keyword.apiconnect_short}}** サービスを検索して選択します。
3. **「探索」**メニューをクリックし、**「サンドボックス」**リンクをクリックします。
4. **「entry.create」**操作をクリックします。
5. 右側のペインで**「試す (Try it)」**をクリックします。 **「パラメーター」**までスクロールダウンし、**「data」**テキスト域に以下を入力します。
    ```javascript
    {
      "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. **「操作の呼び出し」**ボタンをクリックします。 成功を示す**コード: 200** 応答が表示されます。

![](images/solution13/gateway.png)

管理対象のセキュアな API URL は、`https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries` のように、各操作の横に表示されます。
{: tip}

## 速度制限

レート制限を設定すると、API や API 内の特定の操作のネットワーク・トラフィックを管理できます。 レート制限とは、特定の時間間隔内に実行できる呼び出しの最大数です。

1. API Designer に戻り、**「製品」>「entries-api」**をクリックします。
2. 左側で**「デフォルトのプラン」**を選択します。
3. **「デフォルトのプラン」**を展開し、**「レート制限」**フィールドまでスクロールダウンします。
4. フィールドを **10** 呼び出し数/**1** **分**に設定します。
5. **「ハード制限の強制」**を選択し、**「保存」**アイコンをクリックします。
  ![レート制限ページ](images/solution13/rate_limit.png)
6. [{{site.data.keyword.Bluemix_notm}} への API の公開](#publish)セクションの手順に従って、API を再公開します。

API が分当たり 10 要求に制限されました。**「試す (Try it)」**機能を使用して制限に達します。[レート制限のセットアップ](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits)に関する詳細を参照するか、API Designer を探索して使用可能なすべての管理機能を確認してください。

## チュートリアルを発展させる

これで完了です。管理対象のセキュアな API を構築しました。API を拡張するためのその他の推奨事項を以下に示します。

* [{{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant) LoopBack コネクターを使用してデータの永続性を追加します。
* API Designer を使用して、API を管理するための[追加設定を表示](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0)します。
* {{site.data.keyword.apiconnect_short}} で[使用可能な](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics) API の**分析**および**視覚化**を確認します。

## 関連コンテンツ

* [Loopback Documentation](https://loopback.io/doc/index.html)
* [{{site.data.keyword.apiconnect_long}} サービスの概説](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
