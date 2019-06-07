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

# IoT データの収集、視覚化、および分析
{: #gather-visualize-analyze-iot-data}
このチュートリアルでは、IoT デバイスのセットアップ、{{site.data.keyword.iot_short_notm}}内のデータの収集、データの探索、および視覚化の作成を行った後、拡張された機械学習サービスを使用してデータを分析し、履歴データ内の異常を検出する方法について説明します。
{:shortdesc}

## 達成目標
{: #objectives}

* IoT シミュレーターをセットアップします。
* コレクション・データを {{site.data.keyword.iot_short_notm}}に送信します。
* 視覚化を作成します。
* デバイス生成データを分析して異常を検出します。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Node.js アプリケーション](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* Spark サービスおよび {{site.data.keyword.cos_full_notm}} を備えた [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


このチュートリアルでは、費用が発生する場合があります。 [料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* デバイスにより、センサー・データが MQTT プロトコルを使用して {{site.data.keyword.iot_full}} に送信されます。
* 履歴データが、{{site.data.keyword.cloudant_short_notm}} データベースにエクスポートされます。
* {{site.data.keyword.DSX_short}} により、このデータベースからデータがプルされます。
* データが、Jupyter Notebook によって分析され視覚化されます。

## 始める前に
{: #prereqs}

[{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - スクリプトを実行して、ibmcloud cli および必要なプラグインをインストールします。

## IoT プラットフォームの作成
{: #iot_starter}

開始するには、Internet of Things Platform サービス (デバイスを管理可能なハブ) を作成し、安全に接続して**データを収集**し、視覚化およびアプリケーションに使用可能な履歴データを作成します。

1. [**{{site.data.keyword.Bluemix_notm}} カタログ**](https://{DomainName}/catalog/)に移動して、**「IoT」**セクションの[**「Internet of Things Platform」**](https://{DomainName}/catalog/services/internet-of-things-platform)を選択します。
2. サービス名として `IoT demo hub` と入力し、**「作成」**をクリックしてダッシュボードを**起動します**。
3. サイド・メニューから、**「セキュリティー」>「接続セキュリティー」**を選択し、**「デフォルトの規則」**>**「セキュリティー・レベル」**の下の**「TLS (オプション)」**を選択して、**「保存」**をクリックします。
4. サイド・メニューから、**「デバイス」**>**「デバイス・タイプ」**を選択して、**「+ デバイス・タイプの追加」**を選択します。
5. **「名前」**として `simulator` と入力し、**「次へ」**および**「完了」**をクリックします。
6. 次に、**「デバイスの登録」**をクリックします。
7. **「既存のデバイス・タイプの選択」**に対して `simulator` を選択してから、**「デバイス ID」**に対して `phone` と入力します。
8. **「デバイス・セキュリティー」**画面が (「セキュリティー」タブの下に) 表示されるまで、**「次へ」**をクリックします。
9. **「認証トークン」**に値 (`myauthtoken` など) を入力し、**「次へ」**をクリックします。
10. **「完了」**をクリックすると、接続情報が表示されます。 このタブを開いたままにしておきます。

これで、IoT プラットフォームはデータの受信を開始するように構成されました。 デバイスにより、指定されたデバイス・タイプ、ID、およびトークンとともに、そのデータを IoT プラットフォームに送信する必要があります。

## デバイス・シミュレーターの作成
{: #confignodered}
次に、Node.js Web アプリケーションをデプロイして電話でアクセスすると、デバイス加速度センサーに接続され、方向データが IoT プラットフォームに送信されます。

1. 以下のようにして、Github リポジトリーを複製します。
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. 任意の IDE でコードを開き、**manifest.yml** ファイル内の `name` および `host` の各値を固有値に変更します。
3. アプリケーションを {{site.data.keyword.Bluemix_notm}} にプッシュします。
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. 数分でアプリケーションがデプロイされ、URL シミュレーターが `<UNIQUE_NAME>.mybluemix.net` に設定されます。
5. ブラウザーを使用し、電話でこの URL にアクセスします。
6. 「IoT ダッシュボード (IoT Dashboard)」タブの**「デバイスの資格情報」**から接続情報を入力し、**「接続」**をクリックします。
7. 電話でデータの送信が開始されます。 **「IBM {{site.data.keyword.iot_short_notm}}」タブ**に戻り、**「最新のイベント (Recent Events)」**セクションの新規エントリーを確認します。
  ![](images/solution16/recent_events_with_phone.png)

## IBM {{site.data.keyword.iot_short_notm}}でのライブ・データの表示
{: #createcards}
次に、ダッシュボードでデバイス・データを表示するためのボードとカードを作成します。 ボードとカードの詳細については、[ボードとカードを使用したリアルタイム・データの視覚化](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html)を参照してください。

### ボードの作成
{: #createboard}

1. **IBM {{site.data.keyword.iot_short_notm}}・ダッシュボード**を開きます。
2. 左側のメニューから**「ボード」**を選択してから、**「新規ボードの作成」**をクリックします。
3. ボードの名前 (`Simulators` など) を入力し、**「次へ」**、**「送信」**の順にクリックします。  
4. 作成したボードを選択して開きます。

### デバイス・データの表示
{: #cardtemp}
1. **「新しいカードの追加」**をクリックしてから、「デバイス」セクションにある**「線グラフ」**カード・タイプを選択します。
2. リストからデバイスを選択してから、**「次へ」**をクリックします。
3. **「新規データ・セットの接続」**をクリックします。
4. 「「値」カードの作成」ページで以下の値を選択/入力し、**「次へ」**をクリックします。
   - イベント: sensorData
   - プロパティー: ob
   - 名前: OrientationBeta
   - タイプ: 浮動小数点
   - 最小 (Min): -180
   - 最大: 180
5. 「カード・プレビュー」ページで、線グラフのサイズとして**「L」**を選択し、**「次へ」**>**「送信」**をクリックします。
6. ダッシュボードにカードが表示され、ライブの温度データの線グラフが組み込まれます。
7. 携帯電話のブラウザーを使用して再度シミュレーターを起動し、ゆっくりと電話を前後に傾けます。
8. **「IBM {{site.data.keyword.iot_short_notm}}」タブ**に戻ると、グラフが更新されることがわかります。
   ![](images/solution16/board.png)

## {{site.data.keyword.cloudant_short_notm}} での履歴データの保管
1. [**{{site.data.keyword.Bluemix_notm}} カタログ**](https://{DomainName}/catalog/)に移動し、`iot-db` という名前の新しい [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)を作成します。
2. **「接続」**で以下のようにします。
   1. **接続の作成**
   1. Cloud Foundry の場所、組織、および {{site.data.keyword.cloudant_short_notm}} サービスに対する別名を作成する必要があるスペースを選択します。
   1. **「接続場所」**テーブルでスペース名を展開し、**「iot demo hub」**の横にある**「接続」**ボタンを使用して、該当スペース内に {{site.data.keyword.cloudant_short_notm}} サービスに対する別名を作成します。 
   1. アプリを接続および再ステージングします。
3. **IBM {{site.data.keyword.iot_short_notm}}・ダッシュボード**を開きます。
4. 左側のメニューから**「拡張」**を選択してから、**「履歴データ・ストレージ」**の下で**「セットアップ」**をクリックします。
5. `iot-db` {{site.data.keyword.cloudant_short_notm}} データベースを選択します。
6. **「データベース名」**に `devicedata` と入力し、**「完了」**をクリックします。
7. 新しいウィンドウに、許可のためのプロンプトがロードされます。 このウィンドウが表示されない場合は、ポップアップ・ブロッカーを無効にして、ページを最新表示します。

これで、デバイス・データは {{site.data.keyword.cloudant_short_notm}} に保存されました。 数分後、{{site.data.keyword.cloudant_short_notm}} ダッシュボードを起動してデータを表示します。

![](images/solution16/cloudant.png)

## 機械学習を使用した異常の検出
{: #data_experience}

このセクションでは、IBM {{site.data.keyword.DSX_short}} サービスで使用可能な Jupyter Notebook を使用して、履歴モバイル・データをロードしたり、z スコアを使用して異常を検出したりします。 *z スコア* とは、エレメントが平均から離れている標準偏差の程度を示す標準スコアです。

### 新規プロジェクトの作成
1. [**{{site.data.keyword.Bluemix_notm}} カタログ**](https://{DomainName}/catalog/)に移動し、**「AI」**の下で、[**「{{site.data.keyword.DSX_short}}」**](https://{DomainName}/catalog/services/data-science-experience)を選択します。
2. サービスを**作成**し、**「開始」**をクリックしてそのダッシュボードを起動します。
3. プロジェクトを作成します。**「データ・サイエンス (Data Science)」**を選択し、**「プロジェクトの作成 (Create project)」**をクリックして、プロジェクトの**「名前」**として `Detect Anomaly` と入力します。
4. 機密データがないため、**「コラボレーターになれるユーザーを制限する (Restrict who can be a collaborator)」**チェック・ボックスのチェックを外したままにします。
5. **「ストレージの定義 (Define Storage)」**の下で**「追加 (Add)」**をクリックして、既存の **Cloud Object Storage** サービスを選択するか、または新しく作成します (**「ライト」**プラン>「作成」を選択)。**「最新表示」**を押して、作成したサービスを表示します。
6. **「作成」**をクリックします。 新規プロジェクトが開き、それに対するリソースの追加を開始できます。

### データ用の {{site.data.keyword.cloudant_short_notm}} への接続

1. **「資産 (Assets)」**>**「+ プロジェクトに追加 (Add to Project)」**>**「接続」**をクリックします。  
2. デバイス・データが保管されている **iot-db** {{site.data.keyword.cloudant_short_notm}} を選択します。
3. **「資格情報」**をクロスチェックしてから、**「作成」**をクリックします。

### Apache Spark サービスの作成

1. **「サービス」**(ナビゲーション・バー上) >「コンピュート・サービス (Compute Services)」をクリックします。
2. **「サービスの追加」**をクリックします。
   1. **Apache Spark** で**「追加 (Add)」**をクリックします。
   1. **「ライト」**プランを選択します。
   1. **「作成」**をクリックします。
3. 必要に応じて組織およびスペースを選択してサービス名を変更し、**確認**します。
1. **「プロジェクト」**から、`Detect Anomaly` プロジェクトにナビゲートします。
1. **「設定」**で、**「関連サービス (Associated services)」**にスクロールします。
1. **「サービスの追加」**をクリックして、**「Spark (Spark)」**を選択します。
1. 以前に作成した **Apache Spark** インスタンスを選択します。

### Jupyter (ipynb) Notebook の作成
1. **「+ プロジェクトに追加 (Add to Project)」**をクリックして、新しい**ノートブック**を追加します。
2. **「名前」**に `Anomaly-detection-notebook` と入力します。
3. **「ノートブック URL (Notebook URL)」**に、`https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb` と入力します。
4. ランタイムとして、以前に関連付けた **Apache Spark** サービスを選択します。
5. **ノートブック**を作成します。 カーネルとして `Python 3.5 with Spark 2.1` を設定します。 ノートブックがメタデータおよびコードを使用して作成されたことを確認します。
    ![Jupyter Notebook Watson Studio](images/solution16/jupyter_notebook_watson_studio.png)
   更新するには、**「カーネル (Kernel)」**>「カーネルの変更 (Change kernel)」をクリックします。 ノートブックを**信用する**には、**「ファイル」**>「ノートブックを信用する (Trust Notebook)」をクリックします。
   {:tip}

### ノートブックの実行および異常の検出   
1. `!pip install --upgrade pixiedust,` で開始するセルを選択してから、**「実行 (Run)」**をクリックするか、または **Ctrl キーと Enter キー**同時に押して、コードを実行します。
2. インストールが完了したら、**「カーネルの再始動 (Restart Kernel)」**アイコンをクリックして Spark カーネルを再始動します。
3. 次のコード・セルで、以下のステップを実行して、{{site.data.keyword.cloudant_short_notm}} 資格情報をそのセルにインポートします。
   * ![](images/solution16/data_icon.png) をクリックします。
   * **「接続」**タブを選択します。
   * **「コードに挿入 (Insert to code)」**をクリックします。 ご使用の {{site.data.keyword.cloudant_short_notm}} 資格情報で _credentials_1_ という辞書が作成されます。 名前が _credentials_1_ と指定されていない場合は、辞書の名前を `credentials_1` に変更します。`credentials_1` が残りのセルで使用されます。
4. データベース名 (`dbName`) が表示されているセルに、データのソースである {{site.data.keyword.cloudant_short_notm}} データベースの名前 (*iotp_yourWatsonIoTProgId_DBName_Year-month-day* など) を入力します。 さまざまなデバイスのデータを視覚化するには、`deviceId` および `deviceType` の各値を適宜変更します。
   正確なデータベースを検索するには、前に作成した **iot-db** {{site.data.keyword.cloudant_short_notm}} インスタンスにナビゲートして「ダッシュボードを起動」をクリックします。
   {:tip}
5. ノートブックを保存して各コード・セルを 1 つずつ実行するか、またはすべてを実行します (**「セル」**>「すべて実行 (Run All)」)。ノートブックの最後に、デバイス移動データの異常が表示されます (oa、ob、および og)。
   対象となる時間間隔を 1 日の必要な時刻に変更できます。`start` および `end` の各値を検索します。
   {:tip}
   ![Jupyter Notebook DSX](images/solution16/anomaly_detection_watson_studio.png)
6. 異常検出とともに、このセクションから得られる主要な結果または結論は以下のとおりです。
    * 視覚化用のデータを準備するための Spark の使用量
    * データ視覚化のための Panda の使用量
    * デバイス・データの棒グラフ、ヒストグラム
    * 相関行列による 2 つのセンサー間の相関
    * Panda のプロット機能により生成される、各デバイス・センサーのボックス・プロット
    * カーネル密度の推定 (KDE) による密度プロット
    ![](images/solution16/density_plots_sensor_data.png)

## リソースの削除
{:removeresources}

1. [「リソース・リスト」](https://{DomainName}/resources/)にナビゲートして、アプリおよびサービスを作成した「場所」、「組織」、および「スペース」を選択します。 **「Cloud Foundry アプリ」**の下で、上で作成した Node.js アプリを削除します。
2. **「サービス」**の下で、このチュートリアル用に作成した Internet of Things Platform、Apache Spark、{{site.data.keyword.cloudant_short_notm}}、および {{site.data.keyword.cos_full_notm}} の各サービスを削除します。

## 関連コンテンツ
{:related}

* [予測式機械学習モデルのビルド、デプロイ、テスト、リトレーニング](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* [IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html) の概要
* 異常検出 [Jupyter Notebook](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb)
* [z スコアの理解](https://en.wikipedia.org/wiki/Standard_score)
* ディープ・ラーニングを使用した異常検出のためのコグニティブ IoT ソリューションの開発 - [一連の 5 つの投稿](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/)
