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

# ストリーミング分析と SQL によるビッグデータ・ログ
{: #big-data-log-analytics}

このチュートリアルでは、規制要件のサポートと情報開示の支援のために、ログ・レコードを収集、保管および分析するように設計されたログ分析パイプラインを作成します。このソリューションでは、{{site.data.keyword.cloud_notm}} で使用可能なサービスである、{{site.data.keyword.messagehub}}、{{site.data.keyword.cos_short}}、SQL Query および {{site.data.keyword.streaminganalyticsshort}} を活用します。Web サーバーのログ・メッセージを静的ファイルから {{site.data.keyword.messagehub}} に送信するシミュレーションをプログラムによって支援します。

{{site.data.keyword.messagehub}} を使用すると、パイプラインは数百万のログ・レコードをさまざまなプロデューサーから受信するようにスケーリングされます。{{site.data.keyword.streaminganalyticsshort}} を適用することによって、ログ・データをリアルタイムで検査でき、ビジネス・プロセスが統合されます。{{site.data.keyword.cos_short}} を使用して、ログ・メッセージを長期保管のために容易に転送することもでき、それによって、開発者、サポート・スタッフ、監査員は SQL Query を使用してデータを直接処理できます。

このチュートリアルはログ分析に焦点を当てていますが、その他のシナリオ、例えば、ストレージの限られた IoT デバイスで同様にメッセージを {{site.data.keyword.cos_short}} にストリームする場合や、マーケティングの専門家が SQL Query を使用してデジタルなプロパティーを横断して顧客イベントをセグメント化および分析する場合に適用できます。
{:shortdesc}

## 達成目標

{: #objectives}

* Apache Kafka のメッセージングのパブリッシュ/サブスクライブの理解
* 監査およびコンプライアンス要件のためのログ・データの保管
* 例外処理プロセス作成のためのログのモニター
* ログ・データでのフォレンジック分析や統計分析の実施

## 使用するサービス

{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー

{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー](images/solution31/Architecture.png)
</p>

1. アプリケーションが {{site.data.keyword.messagehub}} に対するログ・イベントを生成します
2. ログ・イベントが {{site.data.keyword.streaminganalyticsshort}} によって傍受および分析されます
3. ログ・イベントが {{site.data.keyword.cos_short}} 内にある CSV ファイルに追加されます
4. 監査員やサポート・スタッフが SQL ジョブを実行します
5. SQL Query が {{site.data.keyword.cos_short}} 内のログ・ファイルに対して実行されます
6. 結果セットが {{site.data.keyword.cos_short}} に保管され、監査員やサポート・スタッフに送信されます

## 始める前に

{: #prereqs}

* [Git のインストール](https://git-scm.com/)
* [{{site.data.keyword.Bluemix_notm}} CLI のインストール](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* [Node.js のインストール](https://nodejs.org)
* [Kafka 0.10.2.X クライアントのダウンロード](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)

## サービスの作成

{: #setup}

このセクションでは、アプリケーションによって生成されるログ・イベントの分析を実行するために必要なサービスを作成します。

このセクションでは、コマンド・ラインを使用して、サービス・インスタンスを作成します。また、提供されているリンクを使用して、カタログのサービス・ページから同様の操作をすることもできます。
{: tip}

1. コマンド・ラインを使用して {{site.data.keyword.cloud_notm}} にログインし、ターゲットの Cloud Foundry アカウントを設定します。[CLI の概説](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)を参照してください。
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) のライト・インスタンスを作成します。
    ```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. [SQL Query](https://{DomainName}/catalog/services/sql-query) のライト・インスタンスを作成します。
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams) の標準インスタンスを作成します。
    ```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## メッセージング・トピックおよび {{site.data.keyword.cos_short}} バケットの作成

{: #topics}

{{site.data.keyword.messagehub}} トピックと {{site.data.keyword.cos_short}} バケットの作成から始めます。トピックは、パブリッシュ/サブスクライブ・メッセージング・システムで、アプリケーションがどこにメッセージを送信するかを定義します。メッセージは、受信されて処理された後、{{site.data.keyword.cos_short}} バケットにあるファイル内に保管されます。

1. ブラウザーで、[「リソース」](https://{DomainName}/resources?search=log-analysis)から `log-analysis-hub` サービス・インスタンスにアクセスします。
2. **+** ボタンをクリックして、トピックを作成します。
3. **「トピック名」**に `webserver` を入力し、**「トピックの作成」**ボタンをクリックします。
4. **「サービス資格情報」**および**「新規資格情報」**ボタンをクリックします。
5. 結果ダイアログで、**「名前」**に `webserver-flow` を入力して、**「追加」**ボタンをクリックします。
6. **「資格情報の表示」**をクリックして、安全な場所に情報をコピーします。これは次のセクションで使用します。
7. [「リソース・リスト」](https://{DomainName}/resources?search=log-analysis)に戻り、`log-analysis-cos` サービス・インスタンスを選択します。
8. **「バケットの作成 (Create bucket)」**をクリックします。 
    * バケットの固有の**「名前」**を入力します。
    * **「耐障害性 (Resiliency)」**で**「クロス・リージョン (Cross Region)」**を選択します。
    * **「場所」**で**「us-geo」**を選択します。
    * **「バケットの作成 (Create bucket)」**をクリックします。 

## ストリーム・フロー・ソースの作成

{: #streamsflow}

このセクションでは、ログ・メッセージを受信するストリーム・フローの構成を開始します。{{site.data.keyword.streaminganalyticsshort}} サービスは {{site.data.keyword.streamsshort}} を採用し、1 秒当たり数百万のイベントを分析して 1 ミリ秒未満の応答時間と即時の意思決定を実現できます。

1. ブラウザーで、[Watson Data Platform](https://dataplatform.ibm.com) にアクセスします。
2. **「新規プロジェクト (New project)」**ボタンまたはタイル、次に**「基本 (Basic)」**タイルを選択して、**「OK」**をクリックします。
    * **「名前」**に `webserver-logs` を入力します。
    * **「ストレージ (Storage)」**オプションは `log-analysis-cos` に設定されている必要があります。そうでない場合は、このサービス・インスタンスを選択してください。
    * **「作成」**ボタンをクリックします。
3. 結果ページで、**「設定 (Settings)」**タブを選択して、**「ツール (Tools)」**の**「Streams Designer」**にチェック・マークを付けます。**「保存 (Save)」**ボタンをクリックして終了します。
4. 上部ナビゲーション・バーで、**「プロジェクトに追加 (Add to project)」**ボタン、次に**「ストリーム・フロー (Streams flow)」**をクリックします。
    * **「IBM Streaming Analytics インスタンスとコンテナー・ベース・プランの関連付け (Associate an IBM Streaming Analytics instance with a container-based plan)」**をクリックします。
    * **「ライト (Lite)」**ラジオ・ボタンを選択して**「作成 (Create)」**をクリックし、新規 {{site.data.keyword.streaminganalyticsshort}} インスタンスを作成します。「ライト VM (Lite VM)」は選択しないでください。
    * **「サービス名 (Service name)」**に `log-analysis-sa` を指定して**「確認 (Confirm)」**をクリックします。
    * ストリーム・フローの**「名前」**に `webserver-flow` を入力します。
    * **「作成」**をクリックして終了します。
5. 結果ページで、**{{site.data.keyword.messagehub}}** タイルを選択します。
    * **「接続の追加 (Add Connection)」**をクリックして、`log-analysis-hub` {{site.data.keyword.messagehub}} インスタンスを選択します。該当インスタンスがリストされていない場合は、**IBM {{site.data.keyword.messagehub}}** オプションを選択してください。前のセクションの**「サービス資格情報」**で取得した**接続の詳細**を手動で入力します。接続の**名前**を `webserver-flow` と指定します。
    * **「作成」**をクリックして接続を作成します。
    * **「トピック (Topic)」**ドロップダウンから `webserver` を選択します。
    * **「初期オフセット (Initial Offset)」**ドロップダウンから**「最初の新規メッセージを開始 (Start with the first new message)」**を選択します。
    * **「続行 (Continue)」**をクリックします。
6. **「データのプレビュー (Preview Data)」**ページは開いたままにします。これは次のセクションで使用します。

## {{site.data.keyword.messagehub}} での Kafka コンソール・ツールの使用

{: #kafkatools}

`webserver-flow` は現在、活動停止中で、メッセージを待機しています。このセクションでは、{{site.data.keyword.messagehub}} を処理するために Kafka コンソール・ツールを構成します。Kafka コンソール・ツールを使用すると、ターミナルで任意のメッセージを作成して {{site.data.keyword.messagehub}} に送信することができ、それによって `webserver-flow` がトリガーされます。

1. [Kafka 0.10.2.X クライアント](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)をダウンロードして unzip します。
2. ディレクトリー `bin` に移動して、以下の内容を持つ `message-hub.config` という名前のテキスト・ファイルを作成します。
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. `message-hub.config` ファイルの `USER` と `PASSWORD` は、前のセクションの**「サービス資格情報」**で表示された `user` と `password` の値に置き換えます。`message-hub.config` を保存します。
4. `bin` ディレクトリーから以下のコマンドを実行します。`KAFKA_BROKERS_SASL` は、前のセクションの**「サービス資格情報」**で表示された `kafka_brokers_sasl` 値に置き換えます。以下に例を示します。
    ```sh
    ./kafka-console-producer.sh --broker-list KAFKA_BROKERS_SASL \
    --producer.config message-hub.config --topic webserver
    ```
    {: pre}
    ```sh
    ./kafka-console-producer.sh --broker-list \
    kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093 \
    --producer.config message-hub.config --topic webserver
    ```
5. Kafka コンソール・ツールは入力を待機しています。以下のログ・メッセージをコピーしてターミナルに貼り付けます。`enter` を押して、ログ・メッセージを {{site.data.keyword.messagehub}} に送信します。送信したメッセージは、`webserver-flow` の**「データのプレビュー (Preview Data)」**ページにも表示されることに注意してください。
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![プレビュー・ページ](images/solution31/preview_data.png)

## ストリーム・フロー・ターゲットの作成

{: #streamstarget}

このセクションでは、ターゲットを定義して、ストリーム・フローの構成を完了します。ターゲットは、前に作成した {{site.data.keyword.cos_short}} バケットに着信ログ・メッセージを保管するために使用します。着信ログ・メッセージのファイルへの保管と追加の処理は、{{site.data.keyword.streaminganalyticsshort}} によって自動的に実行されます。

1. `webserver-flow` **プレビュー・ページ**で、**「続行 (Continue)」**ボタンをクリックします。
2. ターゲットとして **{{site.data.keyword.cos_full_notm}}** タイルを選択します。
    * **「接続の追加 (Add Connection)」**をクリックして、`log-analysis-cos` を選択します。
    * **「作成」**をクリックします。
    * **「ファイル・パス (File path)」**に `/YOUR_BUCKET_NAME/http-logs_%TIME.csv` を入力します。`YOUR_BUCKET_NAME` は、最初のセクションで使用したものに置き換えます。
    * **「フォーマット (Format)」**ドロップダウンで**「csv」**を選択します。
    * **「列見出し行 (Column header row)」**チェック・ボックスにチェック・マークを付けます。
    * **「ファイル作成ポリシー (File Creation Policy)」**ドロップダウンで**「ファイル・サイズ (File Size)」**を選択します。
    * **「ファイル・サイズ (KB) (File Size (KB))」**テキスト・ボックスに `102400` を入力して、制限が 100 MB になるように設定します。
    * **「続行 (Continue)」**をクリックします。
3. **「保存」**をクリックします。
4. **>** 再生ボタンをクリックして、**ストリーム・フローを開始します**。
5. フローが開始された後、Kafka コンソール・ツールから複数のログ・メッセージを再度送信します。Streams Designer で `webserver-flow` を表示すると、メッセージが到着するのを見ることができます。
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. {{site.data.keyword.cos_short}} のバケットに戻ります。十分なメッセージがフローに入った後、またはフローが再開された後、新規 `log.csv` ファイルが存在しています。

![webserver-flow](images/solution31/flow.png)

## ストリーム・フローへの条件動作の追加

{: #streamslogic}

ここまでは、ストリーム・フローは、単純なパイプ、つまり、メッセージが {{site.data.keyword.messagehub}} から {{site.data.keyword.cos_short}} に移動するだけのものでした。おそらく、チームでは、関心のあるイベントについてリアルタイムで知りたいでしょう。例えば、個々のチームにとって、HTTP 500 (アプリケーション・エラー) イベントが発生したときのアラートが役立つ場合があります。このセクションでは、HTTP 200 (OK) コードと非 HTTP 200 コードを識別するため、フローに条件ロジックを追加します。

1. 鉛筆ボタンを使用して、**ストリーム・フローを編集します**。
2. HTTP 200 応答を処理するフィルター・ノードを作成します。
    * **「ノード (Nodes)」**パレットの**「処理と分析 (PROCESSING AND ANALYTICS)」**から**「フィルター (Filter)」**ノードをキャンバスにドラッグします。
    * 名前テキスト・ボックスに `OK` を入力します。ここには現在、単語 `Filter` が入っています。
    * **「条件式 (Condition Expression)」**テキスト域に、以下のステートメントを入力します。
      ```sh
      responseCode == 200
      ```
      {: pre}
    * マウスを使用して、**{{site.data.keyword.messagehub}}** ノードの出力 (右側) から**「OK」**ノードの入力 (左側) に線を引きます。
    * **「ノード (Nodes)」**パレットの**「ターゲット (TARGETS)」**の下にある**「デバッグ (Debug)」**ノードをキャンバスにドラッグします。
    * **「デバッグ (Debug)」**ノードと**「OK」**ノードの間に線を引いて、この 2 つを接続します。
3. 同じノードと以下の条件ステートメントを使用して、この処理を繰り返し、`Not OK` フィルターを作成します。
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. 再生ボタンをクリックして、**ストリーム・フローを保存して実行します**。
5. プロンプトが出された場合、リンクをクリックして、**新規バージョンを実行します**。

![フロー設計機能](images/solution31/flow_design.png)

## メッセージ負荷の増加

{: #streamsload}

ストリーム・フローの条件処理を確認するには、{{site.data.keyword.messagehub}} に送信されるメッセージ量を増やします。提供される Node.js プログラムでは、Web サーバーへのトラフィックに基づき {{site.data.keyword.messagehub}} へのメッセージの現実的なフローがシミュレートされます。{{site.data.keyword.messagehub}} と {{site.data.keyword.streaminganalyticsshort}} のスケーラビリティーをデモンストレーションするには、ログ・メッセージのスループットを増やします。

このセクションでは、[node-rdkafka](https://www.npmjs.com/package/node-rdkafka) を使用します。シミュレーターのインストールが失敗する場合は、npmjs ページのトラブルシューティング手順を参照してください。問題が解決しない場合は、次のセクションにスキップして、手動でデータをアップロードしてもかまいません。

1. NASA から [7 月 1 日から 7 月 31 日までの ASCII フォーマット、20.7 MB の gzip 圧縮された](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz)ログ・ファイルをダウンロードして unzip します。
2. [GitHub の IBM-Cloud](https://github.com/IBM-Cloud/kafka-log-simulator) からログ・シミュレーターを複製してインストールします。
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. シミュレーターのディレクトリーに移動して、以下のコマンドを実行し、シミュレーターをセットアップしてログ・イベント・メッセージを作成します。`LOGFILE` は、ダウンロードしたファイルに置き換えます。`BROKERLIST` と `APIKEY` は、前に使用した対応する**サービス資格情報**に置き換えます。以下に例を示します。
    ```sh
    npm install
    ```
    ```sh
    npm run build
    ```
    ```sh
    node dist/index.js --file LOGFILE --parser httpd --broker-list BROKERLIST \
    --api-key APIKEY --topic webserver --rate 100
    ```
    {: pre}
    ```sh
    node dist/index.js --file /Users/ibmcloud/Downloads/NASA_access_log_Jul95 \
    --parser httpd --broker-list \
    "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093" \
    --api-key Np15YZKN3SCdABUsOpJYtpue6jgJ7CwYgsoCWaPbuyFbdM4R \
    --topic webserver --rate 100
    ```
4. シミュレーターがメッセージを作成し始めたら、ブラウザーの `webserver-flow` に戻ります。
5. 必要な数のメッセージが条件付きブランチを通過したら、`control+C` を使用してシミュレーターを停止します。
6. `--rate` 値を増やしたり減らしたりして、{{site.data.keyword.messagehub}} のスケーリングを試験します。

シミュレーターは、Web サーバー・ログの経過時間に基づいて、次のメッセージの送信を遅延させます。`--rate 1` を設定すると、イベントはリアルタイムで送信されます。`--rate 100` を設定すると、Web サーバー・ログの経過時間の 1 秒ごとに、メッセージ間で 10 ms の遅延が使用されます。
{: tip}

![10 に設定されたフロー負荷](images/solution31/flow_load_10.png)

## SQL Query を使用したログ・データの調査

{: #sqlquery}

シミュレーターによって送信されるメッセージの数に応じて、{{site.data.keyword.cos_short}} のログ・ファイルのファイル・サイズは確実に増大します。ここでは、SQL Query をログ・ファイルに結合して、監査やコンプライアンスの質問に応答する調査員として作業します。SQL Query を使用する利点は、ログ・ファイルに直接アクセス可能である、つまり、追加の変換やデータベース・サーバーが必要ないことです。

シミュレーターによってすべてのログ・メッセージが送信されるまで待つ必要をなくすには、[完全な CSV ファイル](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp)を {{site.data.keyword.cos_short}} にアップロードすると、即時に開始できます。
{: tip}

1. [「リソース・リスト」](https://{DomainName}/resources?search=log-analysis)から `log-analysis-sql` サービス・インスタンスにアクセスします。**「UI を開く (Open UI)」**を選択して、SQL Query を起動します。
2. **「ここに SQL を入力します... (Type SQL here ...)」**テキスト域に、以下の SQL を入力します。
    ```sql
    -- What are the top 10 web pages on NASA from July 1995?
    -- Which mission might be significant?
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. ログ・ファイルからオブジェクト SQL URL を取得します。
    * [「リソース・リスト」](https://{DomainName}/resources?search=log-analysis)から、`log-analysis-cos` サービス・インスタンスを選択します。
    * 前に作成したバケットを選択します。
    * `http-logs_TIME.csv` ファイルのオーバーフロー・メニューをクリックして、**「オブジェクト SQL URL」**を選択します。
    * クリップボードに URL を**コピー**します。
4. `FROM` 節をオブジェクト SQL URL で更新して、**「実行 (Run)」**をクリックします。
5. 結果は、**「結果 (Result)」**タブに表示されます。複数のページ (ケネディ宇宙センターのホーム・ページなど) が予想されますが、その時点で 1 つのミッションの人気が非常に高くなっています。
6. **「照会の詳細 (Query Details)」**タブを選択して、{{site.data.keyword.cos_short}} で結果が保管された場所などの追加情報を表示します。
7. 以下の質問と回答のペアを試行するには、**「ここに SQL を入力します... (Type SQL here ...)」**テキスト域に、個別にそれらを追加します。
    ```sql
    -- Who are the top 5 viewers?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- Which viewer has suspicious activity based on application failures?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- Which requests showed a page not found error to the user?
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- What are the top 10 largest files?
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- What is the distribution of total traffic by hour?
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- Why did the previous result return an empty hour?
    -- Hint, find the malformed hostname.
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

FROM 節は単一ファイルに限定されているわけではありません。バケット内のすべてのファイルで SQL 照会を実行するには、`cos://us-geo/YOUR_BUCKET_NAME/` を使用します。
{: tip}

## チュートリアルを発展させる

{: #expand}

{{site.data.keyword.cloud_notm}} でログ分析パイプラインを作成できました。ソリューションを向上させるための追加の提案を以下に示します。

* Streams Designer で追加ターゲットを使用して、[{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) にデータを保管したり、[{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) でコードを実行したりします。
* [オブジェクト・ストレージを使用したデータレイクの構築](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage)チュートリアルに従って、ログ・データにダッシュボードを追加します。
* [{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect) を使用して、{{site.data.keyword.messagehub}} に追加システムを統合します。

## サービスの削除

{: #removal}

[「リソース・リスト」](https://{DomainName}/resources?search=log-analysis)で、オーバーフロー・メニューの**「削除」**または**「サービスの削除」**メニュー項目を使用して、以下のサービス・インスタンスを削除します。

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## 関連コンテンツ

{:related}

* [Apache Kafka](https://kafka.apache.org/)
