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

# オブジェクト・ストレージを使用したデータレイクの構築
{: #smart-data-lake}

データレイクという用語の定義はさまざまですが、このチュートリアルでは、組織で使用するためにデータをそのままの形式で保管する方法のことです。そのために、{{site.data.keyword.cos_short}} を使用して組織用のデータレイクを作成します。{{site.data.keyword.cos_short}} と SQL Query を組み合わせることで、データ・アナリストは、データを置いたその場所で SQL を使用してデータを照会することができます。Jupyter Notebook の SQL Query サービスを活用して、単純な分析を行うこともできます。完了すれば、技術者でないユーザーも {{site.data.keyword.dynamdashbemb_notm}} を使用して独自の洞察を得られるようになります。

## 達成目標

- {{site.data.keyword.cos_short}} を使用して、生データ・ファイルを保管する
- SQL Query を使用して {{site.data.keyword.cos_short}} のデータを直接照会する
- {{site.data.keyword.DSX_full}} でデータを精製して分析する
- {{site.data.keyword.dynamdashbemb_notm}} を使用して組織全体でデータを共有する

## 使用するサービス

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [ SQL Query ](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## アーキテクチャー

![アーキテクチャー](images/solution29/architecture.png)

1. {{site.data.keyword.cos_short}} に生データを保管します
2. SQL Query を使用して、データを要約、拡張、精製します
3. {{site.data.keyword.DSX}} でデータ分析を行います
4. エンド・ユーザーが Web アプリケーションにアクセスします
5. 精製したデータを {{site.data.keyword.cos_short}} からプルします
6. {{site.data.keyword.dynamdashbemb_notm}} を使用して、基幹業務チャートを作成します

## 始める前に

- [Git をインストール](https://git-scm.com/)します
- [{{site.data.keyword.Bluemix_notm}} CLI をインストール](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)します
- [Aspera Connect をインストール](http://downloads.asperasoft.com/connect2/)します
- [Node.js と NPM をインストール](https://nodejs.org)します

## サービスを作成する

このセクションでは、データレイクの構築に必要なサービスを作成します。

このセクションでは、コマンド・ラインを使用してサービス・インスタンスを作成します。また、提供されているリンクを使用して、カタログのサービス・ページから同様の操作をすることもできます。
{: tip}

1. コマンド・ラインを使用して {{site.data.keyword.cloud_notm}} にログインし、ターゲットの Cloud Foundry アカウントを設定します。[CLI の概説]https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) を参照してください。
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Cloud Foundry 別名を持つ [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) のインスタンスを作成します。既にサービス・インスタンスがある場合は、既存のサービス名を指定して `service-alias-create` コマンドを実行します。
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. [SQL Query](https://{DomainName}/catalog/services/sql-query) のインスタンスを作成します。
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio) のインスタンスを作成します。
    ```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. Cloud Foundry 別名を持つ [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded) のインスタンスを作成します。
    ```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. 作業ディレクトリーに移動し、次のコマンドを実行して、このダッシュボード・アプリケーションの [GitHub リポジトリー](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard)を複製します。次に、Cloud Foundy 組織にアプリケーションをプッシュします。アプリケーションは、その [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml) ファイルを使用して、前述の必要なサービスを自動的にバインドします。
    ```sh
    git clone https://github.com/IBM-Cloud/nodejs-data-lake-dashboard.git
    ```
    {: pre}
    ```sh
    cd nodejs-data-lake-dashboard
    ```
    {: pre}
    ```sh
    npm install
    ```
    {: pre}
    ```sh
    npm run push
    ```
    {: pre}

    デプロイメント後、アプリケーションは公開され、ランダムなホスト名で listen します。[「リソース・リスト」](https://{DomainName}/resources)ページに移動し、「Cloud Foundry アプリ」でアプリを選択して URL を表示できます。あるいは、コマンド `ibmcloud cf app dashboard-nodejs routes` を実行して経路を確認できます
    {: tip}

7. ブラウザーでそのパブリック URL にアクセスして、アプリケーションがアクティブであることを確認します。

![ダッシュボードのランディング・ページ](images/solution29/dashboard-start.png)

## データをアップロードする

このセクションでは、組み込みの {{site.data.keyword.CHSTSshort}} を使用して、{{site.data.keyword.cos_short}} バケットにデータをアップロードします。{{site.data.keyword.CHSTSshort}} を使用すると、バケットにアップロード中のデータを保護できるうえに、[転送時間を大幅に短縮できます](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/)。

1. [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD) という CSV ファイルをダウンロードします。このファイルは 81 MB あるので、ダウンロードには数分かかる可能性があります。
2. ブラウザーで、[「リソース・リスト」](https://{DomainName}/resources)の **data-lake-cos** サービス・インスタンスにアクセスします。
3. データを保管する新規バケットを作成します。
    - **「バケットの作成」**ボタンをクリックします。
    - **「耐障害性 (Resiliency)」**ドロップダウンから**「地域 (Regional)」**を選択します。
    - **「ロケーション」**から**「us-south」**を選択します。現時点では、{{site.data.keyword.CHSTSshort}} は、`米国南部`で作成されたバケットでのみ使用できます。代わりに別のロケーションを選択し、次のセクションで**「標準」**転送タイプを使用することもできます。
    - バケットの**名前**を指定し、**「作成」**をクリックします。*AccessDenied* というエラーを受け取った場合は、より固有なバケット名で試行してください。
4. CSV ファイルを {{site.data.keyword.cos_short}} にアップロードします。
    - バケットから、**「オブジェクトの追加」**ボタンをクリックします。
    - **「Aspera 高速転送」**ラジオ・ボタンを選択します。
    - **「ファイルの追加」**ボタンをクリックします。これにより、Aspera プラグインが別のウィンドウで開かれます。通常、ブラウザー・ウィンドウの後方で開かれます。
    - 前にダウンロードした CSV ファイルを参照して選択します。

![CSV ファイルが含まれているバケット](images/solution29/cos-bucket.png)

## データを処理する

このセクションでは、時刻と経過時間の属性に基づいて、元の生データ・セットをターゲットのデータ群に変換します。データレイクの利用者が特定の関心を持っていたり、非常に大きなデータ・セットに頭を悩ませていたりする場合は、このような処理が役に立ちます。

SQL Query を使用して、{{site.data.keyword.cos_short}} にデータを置いたまま、使い慣れた SQL ステートメントでデータを操作します。SQL Query は、CSV、JSON および Parquet を標準でサポートするので、追加のコンピュート・サービスも抽出変換ロード処理も必要ありません。

1. [「リソース・リスト」](https://{DomainName}/resources)から **data-lake-sql** SQL Query サービス・インスタンスにアクセスします。
2. **「UI を開く (Open UI)」**を選択します。
3. 前にアップロードした CSV ファイルに対して直接 SQL を実行して、新規データ・セットを作成します。
    - **「ここに SQL を入力します... (Type SQL here ...)」**テキスト域に、以下の SQL を入力します。
    ```
        SELECT
        `Dr Number` AS id,
        `Date Occurred` AS date,
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
        FROM cos://us-south/<your-bucket-name>/Traffic_Collision_Data_from_2010_to_Present.csv
        WHERE
        `Time Occurred` >= 1700 AND
        `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND
        `Victim Age` <= 35
        ```
        {: codeblock}
    - `FROM` 節の URL はバケット名に置き換えてください。
4. **ターゲット**に、結果を保持する {{site.data.keyword.cos_short}} バケットが自動的に作成されます。**ターゲット**を `cos://us-south/<your-bucket-name>/results` に変更します。
5. **「実行」**ボタンをクリックします。下部に結果が表示されます。
6. **「照会の詳細 (Query Details)」**タブで、**「結果の場所 (Result Location)」** URL の後にある**「起動 (Launch)」**アイコンをクリックして、中間データ・セット (このデータ・セットも {{site.data.keyword.cos_short}} に保管されています) を表示します。

![Notebook](images/solution29/sql-query.png)

## Jupyter Notebook と SQL Query を組み合わせる

このセクションでは、Jupyter Notebook の内部で SQL Query クライアントを使用します。この場合、{{site.data.keyword.cos_short}} に保管されているデータ・ストアをデータ分析ツール内で再使用します。この組み合わせでもデータ・セットが作成されます。このデータ・セットは、{{site.data.keyword.cos_short}} に自動的に保管され、{{site.data.keyword.dynamdashbemb_notm}} で使用できるようになります。

1. {{site.data.keyword.DSX}} で新規 Jupyter Notebook を作成します。
    - ブラウザーで、[{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true) を開きます。
    - **「プロジェクトの作成 (Create a Project)」**タイルをクリックしてから、**「データ・サイエンス (Data Science)」**をクリックします。
    - **「プロジェクトの作成 (Create project)」**をクリックしてから、**「プロジェクト名 (Project name)」**を指定します。
    - **「ストレージ」**が **data-lake-cos** に設定されていることを確認してください。
    - **「作成」**をクリックします。
    - 作成されたプロジェクトで、**「プロジェクトに追加 (Add to project)」**および**「ノートブック (Notebook)」**をクリックします。
    - **「ブランク (Blank)」**タブで、**「ノートブック名 (Notebook name)」**を入力します。
    - **「言語」**および**「ランタイム」**はデフォルトのままにして、**「ノートブックの作成 (Create notebook)」**をクリックします。
2. Notebook から、以下のコマンドを **In [ ]: ** 入力プロンプトに追加して PixieDust と ibmcloudsql をインストール、インポートし、**「実行」**をクリックします。
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. {{site.data.keyword.cos_short}} API キーを Notebook に追加します。これにより、SQL Query の結果が {{site.data.keyword.cos_short}} に保管されます。
    - 以下を次の **In [ ]: ** プロンプトに追加して、**「実行」**をクリックします。
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - 端末から、API キーを作成します。
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - **API キー**をクリップボードにコピーします。
    - API キーを Notebook 内のテキスト・ボックスに貼り付けて、`Enter` キーを押します。
    - また、API キーをセキュアで永続的な場所に保管する必要もあります。Notebook には API キーは保管されません。
4. SQL Query インスタンスの CRN (クラウド・リソース名) を Notebook に追加します。
    - 次の **In [ ]: ** プロンプトで、CRN を Notebook 内の変数に割り当てます。
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - 端末で、**ID** プロパティーの CRN をクリップボードにコピーします。
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - 単一引用符の間に CRN を貼り付け、**「実行」**をクリックします。
5. もう 1 つ、{{site.data.keyword.cos_short}} バケットを指定した変数を Notebook に追加し、**「実行」**をクリックします。
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. 別の **In [ ]: ** プロンプトで以下のコマンドを入力し、**「実行」**をクリックして結果を表示します。バケットにも、`SELECT` の結果を含む新しいファイル `accidents/jobid=<id>/<part>.csv*` が追加されています。
    ```python
    sqlClient = ibmcloudsql.SQLQuery(cloud_api_key, sql_crn, sql_cos_endpoint + '/accidents')

    data_source = sql_cos_endpoint + "/Traffic_Collision_Data_from_2010_to_Present.csv"

    query = """
    SELECT
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
    FROM  {}
    WHERE
        `Time Occurred` >= 1700 AND `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND `Victim Age` <= 35
    """.format(data_source)

    traffic_collisions = sqlClient.run_sql(query)
    traffic_collisions.head()
    ```
    {: codeblock}

## PixieDust を使用してデータを視覚化する

このセクションでは、PixieDust および Mapbox を使用して前の結果セットを視覚化し、交通事故のパターンやホット・スポットをより詳しく分析します。

1. 共通表式を作成して、`location` 列を別々の `latitude` 列と `longitude` 列に変換します。Notebook のプロンプトから以下を**実行**します。
    ```python
    query = """
    WITH location AS (
        SELECT
            id,
            cast(split(coordinates, ',')[0] as float) as latitude,
            cast(split(coordinates, ',')[1] as float) as longitude
        FROM (SELECT
                `Dr Number` as id,
                regexp_replace(Location, '[()]', '') as coordinates
            FROM {0}
        )
    )
    SELECT
        d.`Dr Number` as id,
        d.`Date Occurred` as date,
        d.`Time Occurred` AS time,
        d.`Area Name` AS area,
        d.`Victim Age` AS age,
        d.`Victim Sex` AS sex,
        l.latitude,
        l.longitude
    FROM {0} AS d
        JOIN
        location AS l
        ON l.id = d.`Dr Number`
    WHERE
        d.`Time Occurred` >= 1700 AND
        d.`Time Occurred` <= 2000 AND
        d.`Victim Age` >= 20 AND
        d.`Victim Age` <= 35 AND
        l.latitude != 0.0000 AND
        l.latitude != 0.0000
    """.format(data_source)

    traffic_location = sqlClient.run_sql(query)
    traffic_location.head()
    ```
    {: codeblock}
2. 次の **In [ ]: ** プロンプトで、`display` コマンドを**実行**し、PixieDust を使用して結果を表示します。
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. グラフのドロップダウン・ボタンを選択して、**「地図 (Map)」**を選択します。
4. `latitude` と `longitude` を**「キー (Keys)」**に追加します。`ID` と `age` を**「値」**に追加します。**「OK」**をクリックしてマップを表示します。
5. **「保存」**アイコンをクリックして、Notebook を {{site.data.keyword.cos_short}} に保存します。

![Notebook](images/solution29/notebook-mapbox.png)

## 組織でデータ・セットを共有する

データレイクの利用者が全員データ・サイエンティストであるとは限りません。技術者でないユーザーでも {{site.data.keyword.dynamdashbemb_notm}} を使用してデータレイクから洞察を得られるようにすることができます。SQL Query と同様に、{{site.data.keyword.dynamdashbemb_notm}} も、事前に作成されたダッシュボードを使用して、{{site.data.keyword.cos_short}} から直接データを読み取ることができます。このセクションでは、ユーザーがデータレイクにアクセスしてカスタム・ダッシュボードを作成できるようにするソリューションを紹介します。

1. 前に {{site.data.keyword.Bluemix_notm}} にプッシュしたダッシュボード・アプリケーションのパブリック URL にアクセスします。
2. 目的のレイアウトに対応するテンプレートを選択します (以下の手順では、最初の行の 2 番目のレイアウトを使用します)。
3. `「選択されたソース (Selected sources)」`サイド・シェルフに表示される`「ソースの追加 (Add a source)」`ボタンを使用して、`bucket name` のアコーディオンを展開し、いずれかの `accidents/jobid=...` テーブル・エントリーをクリックします。右上の X アイコンを使用して、ダイアログを閉じます。
4. 左側にある`「視覚化 (Visualizations)」`アイコンをクリックし、**「サマリー」**をクリックします。
5. `accidents/jobid=...` ソースを選択し、`テーブル`を展開してグラフを作成します。
    - `ID` を**「値」**行にドラッグ・アンド・ドロップします。
    - 上の隅にあるアイコンを使用して、グラフを省略します。
6. 以下のようにして、再度、`「視覚化 (Visualizations)」`から**ツリー・マップ**・グラフを作成します。
    - `area` を**「エリアの階層 (Area hierarchy)」**行にドラッグ・アンド・ドロップします。
    - `ID` を**「サイズ」**行にドラッグ・アンド・ドロップします。
    - グラフを省略して結果を表示します。

![ダッシュボード・グラフ](images/solution29/dashboard-chart.png)

## ダッシュボードを探索する

このセクションでは、追加の手順をいくつか実行して、ダッシュボード・アプリケーションと {{site.data.keyword.dynamdashbemb_notm}} の機能を探索します。

1. サンプル・ダッシュボード・アプリケーションのツールバーにある**「モード」**ボタンをクリックして、モード・ビュー `VIEW` を変更します。
2. 下部にあるグラフの色付きのタイルのいずれか、またはグラフの凡例にある `area` 値をクリックします。これにより、タブにローカル・フィルターが適用され、他のグラフにそのフィルターで限定されたデータが表示されるようになります。
3. ツールバーの**「保存」**ボタンをクリックします。
    - 対応する入力フィールドにダッシュボードの名前を入力します。
    - **「仕様 (Spec)」**タブを選択して、このダッシュボードの仕様を表示します。仕様は、{{site.data.keyword.dynamdashbemb_notm}} の固有のファイル形式です。この中には、作成したグラフと使用した {{site.data.keyword.cos_short}} データ・ソースに関する情報が入っています。
    - ダイアログの**「保存」**ボタンを使用して、ブラウザーのローカル・ストレージにダッシュボードを保存します。
4. 新規ダッシュボードを作成するには、ツールバーの**「新規」**ボタンをクリックします。保存したダッシュボードを開くには、**「開く」**ボタンをクリックします。ダッシュボードを削除するには、「ダッシュボードを開く (Open Dashboard)」ダイアログの**「削除」**アイコンを使用します。

実動アプリケーションでは、URL、ユーザー名、パスワードなどの情報を暗号化して、エンド・ユーザーから見えないようにしてください。[データ・ソース情報の暗号化](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information)を参照してください。
{: tip}

## チュートリアルを発展させる

おめでとうございます。{{site.data.keyword.cos_short}} を使用したデータレイクの構築が完了しました。以下は、データレイクをさらに機能拡張するためのヒントです。

- SQL Query を使用して追加のデータ・セットを試します。
- [ストリーミング分析と SQL によるビッグデータ・ログ](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics)を実行して、複数のソースからデータレイクにデータをストリームします。
- ダッシュボード・アプリケーションのコードを編集して、ダッシュボードの仕様を [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) または {{site.data.keyword.cos_short}} に保存します。
- [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id) サービス・インスタンスを作成して、ダッシュボード・アプリケーションのセキュリティーを有効にします。

## リソースの削除

以下のコマンドを実行して、使用したサービス、アプリケーション、キーを削除します。

```sh
ibmcloud resource service-binding-delete dashboard-nodejs-dde dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-binding-delete dashboard-nodejs-cos dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-dde
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-cos
```
{: pre}
```sh
ibmcloud iam api-key-delete data-lake-cos-key
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-dde
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-cos
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-sql
```
{: pre}
```sh
ibmcloud service delete data-lake-studio
```
{: pre}
```sh
ibmcloud app delete dashboard-nodejs
```
{: pre}

## 関連コンテンツ

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Jupyter Notebook](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
