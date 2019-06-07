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

# クラウド・データ用の SQL データベース
{: #sql-database}

このチュートリアルでは、SQL (リレーショナル) データベース・サービスをプロビジョンし、表を作成して、データベースに大量のデータ・セット (市区町村情報) をロードする方法について説明します。その後、そのデータを利用する Web アプリ「worldcities」をデプロイし、クラウド・データベースにアクセスする方法を説明します。このアプリは、[Flask フレームワーク](http://flask.pocoo.org/)を使用して Python で作成されています。

![](images/solution5/Architecture.png)

## 達成目標

* SQL データベースをプロビジョンする
* データベース・スキーマ (表) を作成する
* データをロードする
* アプリとデータベース・サービスを接続する (資格情報を共有する)
* モニタリング、セキュリティー、バックアップ & リカバリー

## 製品

このチュートリアルでは、以下の製品を使用します。
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## 始める前に
{: #prereqs}

[GeoNames](http://www.geonames.org/) にアクセスし、ファイル [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip) をダウンロードして解凍します。この中には、人口が 1000 人を超える市区町村に関する情報が入っています。これをデータ・セットとして使用します。

## SQL データベースをプロビジョンする
まず、**[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)** サービスのインスタンスを作成します。

![](images/solution5/Catalog.png)

1. [{{site.data.keyword.Bluemix_short}} ダッシュボード](https://{DomainName})にアクセスします。上部ナビゲーション・バーの**「カタログ」**をクリックします。
2. 左側のペインの「プラットフォーム」下の**「データ & 分析」**をクリックして、**「{{site.data.keyword.dashdbshort_notm}}」**を選択します。
3. **「エントリー (Entry)」**プランを選択し、提示されたサービス名を「sqldatabase」に変更します (後でこの名前を使用します)。データベースのデプロイメントのロケーションを選択し、正しい組織とスペースが選択されていることを確認します。
4. **「作成」**をクリックします。間もなく、成功の通知を受け取ります。
5. **「リソース・リスト」**で、新規に作成した {{site.data.keyword.dashdbshort_notm}} サービスのエントリーをクリックします。
6. **「開く」**をクリックして、データベース・コンソールを起動します。コンソールを初めて使用するときには、ツアーの表示が出ます。

## 表を作成する
サンプル・データをデータを格納する表が必要です。コンソールを使用して表を作成します。

1. {{site.data.keyword.dashdbshort_notm}} のコンソールで、ナビゲーション・バーの**「探索」**をクリックします。データベース内にある既存のスキーマのリストが表示されます。
2. 「DASH」で始まるスキーマを見つけてクリックします。
3. **「+ 新規表 (+ New Table)」**をクリックして、表名とその列のフォームを表示します。
4. 表名として「cities」と入力します。ファイル [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) の列定義をコピーし、列とデータ・タイプのボックス内に貼り付けます。
5. **「作成」**をクリックして、新規表を定義します。   
   ![](images/solution5/TableCitiesCreated.png)

## データをロードする
表「cities」が作成されたので、その中にデータをロードします。これはさまざまな方法で行うことができます。例えば、ローカル・マシンやクラウド・オブジェクト・ストレージ (COS) から、Swift や Amazon S3 インターフェース、あるいは [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift) マイグレーション・サービスを利用してロードすることができます。このチュートリアルでは、マシンからデータをアップロードします。このプロセスの中で、表の構造とデータ形式をファイル内容に完全に対応するように合わせます。

1. 上部ナビゲーションで、**「ロード (Load)」**をクリックします。次に、**「ファイル選択 (File selection)」**の下で、**「ファイルを参照 (browse files)」**をクリックして、このガイドの最初のセクションでダウンロードしたファイル「cities1000.txt」を見つけて選択します。
2. **「次へ」**をクリックして、スキーマの概要に進みます。ここでも「DASH」で始まるスキーマを選択してから、表「CITIES」を選択します。もう一度**「次へ」**をクリックします。   

   この表は空ですので、既存のデータに追加するか既存のデータを上書きするかは関係ありません。
   {:tip }
3. 次は、ロード・プロセスでファイル「cities1000.txt」のデータを解釈する方法をカスタマイズします。まず、このファイルにはデータしか入っていないので、「最初の行をヘッダーとする (Header in first row)」を無効にします。次に、区切り文字として「0x09」と入力します。これは、ファイル内の値がタブ (タブ文字) で区切られていることを意味します。最後に、日付形式として「YYYY-MM-DD」を選択します。これで、すべてが以下のスクリーン・ショットのようになります。    
  ![](images/solution5/LoadTabSeparator.png)
4. **「次へ」**をクリックすると、ロード設定の確認画面が表示されます。同意して**「ロードの開始 (Begin Load)」**をクリックし、「CITIES」表へのデータのロードを開始します。進行状況が表示されます。データがアップロードされれば、ほんの数秒でロードは完了するはずです。いくつかの統計が表示されます。   
   ![](images/solution5/LoadProgressSteps.png)

## ロードされたデータを SQL を使用して確認する
データがリレーショナル・データベースにロードされました。エラーはありませんでしたが、いくつかの簡単なテストを実行する必要があります。組み込みの SQL エディターを使用して、いくつかの SQL ステートメントを入力し、実行します。

1. 上部ナビゲーションで**「SQL の実行 (Run SQL)」**をクリックします。
組み込みの SQL エディターの代わりに、デスクトップまたは {{site.data.keyword.dashdbshort_notm}} のサーバー・マシンで、クラウド・ベースや従来型の SQL ツールを使用することもできます。接続情報は、設定メニューにあります。(資料とヘルプを表す)「本」のアイコンから表示されるメニューの「ダウンロード」セクションで、ダウンロードできるツールもあります。
    {:tip }
2. 「SQL エディター」で、以下の照会を入力またはコピーします。   
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   次に、**「すべて実行 (Run All)」**ボタンを押します。結果セクションに、ロード・プロセスで報告された行数と同じ数の行が表示されます。   
3. 「SQL エディター」で、次のステートメントを新しい行に入力します。
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. エディターの中で、上記ステートメントのテキストを選択します。**「選択した項目を実行 (Run Selected)」**ボタンをクリックします。ここでは、このステートメントのみが実行され、結果セクションに国別の統計が返されます。

## アプリケーション・コードをデプロイする
すぐに実行できる[データベース・アプリのコードが、この Github リポジトリーにあります](https://github.com/IBM-Cloud/cloud-sql-database)。このリポジトリーを複製またはダウンロードして、IBM Cloud にプッシュします。

1. 以下のようにして、Github リポジトリーを複製します。
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. アプリケーションを IBM Cloud にプッシュします。データベースをプロビジョンしたロケーション、組織およびスペースにログインする必要があります。これらのコマンドを一度に 1 行ずつコピー・アンド・ペーストしてください。
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. プッシュ・プロセスが完了したら、アプリにアクセスできます。これ以上の構成は不要です。ファイル `manifest.yml` が、IBM Cloud に対して、このアプリを「sqldatabase」というデータベース・サービスと一緒にバインドするように命令します。

## セキュリティー、バックアップ & リカバリー、モニタリング
{{site.data.keyword.dashdbshort_notm}} は、マネージド・サービスです。環境の保護、日常的なバックアップおよびシステムのモニタリングは、IBM が行います。エントリー・プランの場合は、マルチテナント・セットアップのデータベース環境であり、ユーザーが選択できる管理オプションや構成オプションは少なくなっています。一方、エンタープライズ・プランのいずれかを使用する場合は、[ユーザーを管理し、追加のデータベース・セキュリティーを構成するためのいくつかのオプション](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html)と、[データベースをモニターする](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html)オプションを選択できます。   

従来型の管理オプションのほかに、[{{site.data.keyword.dashdbshort_notm}} サービスは、モニタリング、ユーザー管理、ユーティリティー、ロード、ストレージ・アクセスなどのための REST API も提供しています](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html)。そのような API の実行可能 Swagger インターフェースに、「Rest API」下の「本」のアイコンで表示されるメニューからアクセスすることができます。同じメニューの「ダウンロード」セクションで、モニタリングなどに使用可能ないくつかのツール (IBM Data Server Manager など) をダウンロードすることもできます。

## アプリをテストする
ロードしたデータ・セットに基づいて市区町村情報を表示するこのアプリは、最小限に切り詰められています。このアプリは、市区町村名 1 つといくつかの事前構成された市区町村を指定できる検索フォームを表示します。これらは、`/search?name=cityname` (検索フォーム) または `/city/cityname` (直接指定された市区町村) のいずれかに変換されます。どちらの要求もバックグラウンドでは同じコード行で処理されます。市区町村名は、セキュリティー上の理由から、パラメーター・マーカーを使用して、準備済みの SQL ステートメントに値として渡されます。行がデータベースから取り出され、HTML テンプレートに渡されてレンダリングされます。

## クリーンアップ
チュートリアルで使用したリソースをクリーンアップするには、以下の手順を実行します。
1. [{{site.data.keyword.Bluemix_short}} リソース・リスト](https://{DomainName}/resources)にアクセスします。アプリを見つけます。
2. アプリのメニュー・アイコンをクリックして、**「アプリの削除」**を選択します。ダイアログ・ウィンドウで、関連する {{site.data.keyword.dashdbshort_notm}} サービスを削除するためのチェック・マークを付けます。
3. **「削除」**ボタンをクリックします。アプリとデータベース・サービスが削除され、リソース・リストに戻ります。

## チュートリアルを発展させる
このアプリを機能拡張しますか? 以下にいくつかのアイデアを示します。
1. 代替名をワイルドカード検索できるようにする。
2. 特定の国および特定の人口値の市区町村のみを検索する。
3. CSS スタイルを置換し、テンプレートを拡張して、ページ・レイアウトを変更する。
4. フォームを使用して市区町村情報を新規作成できるようにする。または、既存のデータ (人口など) を更新できるようにする。

## 関連コンテンツ
* 資料: [IBM Knowledge Center for {{site.data.keyword.dashdbshort_notm}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.Db2_on_Cloud_long_notm}} および {{site.data.keyword.dashdblong_notm}} に関するよくある質問 (FAQ)](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html)。マネージド・サービス、データ・バックアップ、データ暗号化、セキュリティーなどに関する質問への回答があります。
* [無料の Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) (開発者向け)
* 資料: [ibm_db Python ドライバーの API の説明](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
