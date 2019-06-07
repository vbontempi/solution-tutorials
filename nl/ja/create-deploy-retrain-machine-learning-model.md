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

#                 予測式機械学習モデルのビルド、デプロイ、テスト、リトレーニング
            
{: #create-deploy-retrain-machine-learning-model}
このチュートリアルでは、予測式機械学習モデルを構築して、アプリケーションで使用する API としてデプロイし、テストしてフィードバック・データでリトレーニングするプロセスについて説明します。このことはすべて、IBM Cloud 上の統合および統一されたセルフサービス・エクスペリエンスで行われます。

このチュートリアルでは、花の品種を分類する機械学習モデルの作成に**アイリス・フラワー・データ・セット**を使用します。

機械学習の用語では、分類は監視付き学習の一例と見なされます。例えば、正しく識別された観察のトレーニング・セットを利用できる学習があります。
{:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## 達成目標
{: #objectives}

* データをプロジェクトにインポートします。
* 機械学習モデルを構築します。
* モデルをデプロイし、API を試します。
* 機械学習モデルをテストします。
* 継続的な学習とモデル評価のためのフィードバック・データ接続を作成します。
* モデルをリトレーニングします。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## 始める前に
{: #prereqs}
* IBM Watson Studio および Watson Knowledge Catalog は、IBM Watson に含まれるアプリケーションです。IBM Watson アカウントを作成するには、これらのアプリケーションの一方または両方に登録して開始します。

   [IBM Watson の試用](https://dataplatform.ibm.com/registration/stepone)に移動し、IBM Watson アプリに登録します。

## プロジェクトへのデータのインポート

{:#import_data_project}

プロジェクトは、特定の目標を達成するためのリソースの編成方法です。プロジェクト・リソースには、データ、コラボレーター、Jupyter Notebook や機械学習モデルなどの分析ツールを含めることができます。

プロジェクトを作成してデータを追加し、データをクレンジングおよびシェーピングするためにデータ・リファイナーでデータ資産を開くことができます。

**プロジェクトの作成:**

1. [{{site.data.keyword.Bluemix_short}} カタログ](https://{DomainName}/catalog)に移動し、**「AI」**セクションにある「[{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services)」を選択します。サービスを**「作成」**します。**「開始」**ボタンをクリックして、**{{site.data.keyword.DSX_short}}** ダッシュボードを起動します。
2. **「プロジェクトの作成 (Create a project)」**で、**「標準」**タイルの**「プロジェクトの作成 (Create Project)」**をクリックします。`iris_project` という名前およびプロジェクトのオプションの説明を追加します。
3. 機密データがないため、**「コラボレーターになれるユーザーを制限する (Restrict who can be a collaborator)」**チェック・ボックスのチェックを外したままにします。
4. **「ストレージの定義 (Define Storage)」**の下で**「追加 (Add)」**をクリックして、既存の Cloud Object Storage サービスを選択するかまたは新しく作成します (**「ライト」**プラン>「作成」を選択)。**「最新表示」**を押して、作成したサービスを表示します。
5. **「作成」**をクリックします。 新規プロジェクトが開き、それに対するリソースの追加を開始できます。

**データのインポート:**

前述のように、**Iris データ・セット**を使用します。アイリス・データ・セットは、R.A. Fisher の著名な 1936 年の論文、[The Use of Multiple Measurements in Taxonomic Problems](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf) で使用され、[UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/) にもあります。この小さいデータ・セットは、機械学習アルゴリズムや可視化のテストによく使用されています。その目的は、がく片と花弁の長さと幅の測定により、アイリスの花を 3 つの品種 (Setosa、Versicolor、Virginica) に分類することです。アイリス・データ・セットには、それぞれ 50 インスタンスを持つ 3 つのクラスが含まれています。各クラスは、植物のアイリスの品種を指します。
![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


各クラス 40 インスタンスで構成される [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv) を**ダウンロード**します。各クラスの残りの 10 インスタンスを使用してモデルをリトレーニングします。

1. プロジェクトの**「資産 (Assets)」**で、**「データを検索して追加 (Find and Add Data)」**アイコン ![データ検索アイコンを示します。](images/solution22-build-machine-learning-model/data_icon.png) をクリックします。
2. **「ロード (Load)」**で**「参照 (browse)」**をクリックして、ダウンロードした `iris_initial.csv` をアップロードします。
      ![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. 追加すると、プロジェクトの**「データ資産 (Data assets)」**セクションに `iris_initial.csv` が表示されます。名前をクリックすると、データ・セットの内容が表示されます。

## サービスの関連付け
{:#associate_services}
1. **「設定」**で、**「関連サービス (Associated services)」**までスクロールし、**「サービスの追加」**をクリックし、**「Spark (Spark)」**を選択します。
   ![](images/solution22-build-machine-learning-model/associate_services.png)
2. **「ライト」**プランを選択し、**「作成」**をクリックします。デフォルト値を使用し、**「確認」**をクリックします。
3. **「サービスの追加」**を再度クリックし、**「Watson」**を選択します。**「機械学習 (Machine Learning)」**タイルの**「追加 (Add)」**をクリックし、**「ライト」**プランを選択して**「作成」**をクリックします。
4. デフォルト値をそのままにし、**「確認」**をクリックして、機械学習サービスをプロビジョンします。

## 機械学習モデルの構築

{:#build_model}

1. **「プロジェクトに追加 (Add to project)」**をクリックし、**「Watson 機械学習モデル (Watson Machine Learning model)」**を選択します。ダイアログで、**iris_model** という名前およびオプションの説明を追加します。
2. **「機械学習サービス (Machine Learning Service)」**セクションに、上記のステップで関連付けた機械学習サービスが表示されます。
   ![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. モデル・タイプとして**「モデル・ビルダー (Model builder)」**を選択し、**「Spark サービスまたは環境 (Spark Service or Environment)」**セクションで、前に作成した Spark サービスを選択します。
4. **「手動」**を選択して、モデルを手動で作成します。**「作成」**をクリックします。

   自動方式の場合は、自動データ準備 (ADP) に完全に依存します。手動方式の場合は、ADP 変換プログラムによって処理されるいくつかの機能に加えて、分析で使用されるアルゴリズムである独自の見積もりプログラムを追加および構成できます。
   {:tip}

5. 次のページで、データ・セットとして `iris_initial.csv` を選択し、**「次へ」**をクリックします。
6. **「手法の選択 (Select a technique)」**ページでは、追加したデータ・セットに基づいて、「ラベル」列と「フィーチャー」列にデータが取り込まれます。**「ラベル列 (Label Col)」**として**「species (String)」**を選択し、**「フィーチャー列 (Feature columns)」**として**「petal_length (Decimal)」**と**「petal_width (Decimal)」**を選択します。
7. 推奨手法として**「複数クラス分類 (Multiclass Classification)」**を選択します。
   ![](images/solution22-build-machine-learning-model/model_technique.png)
8. **「検証分割 (Validation Split)」**で、以下の設定を構成します。

   **トレーニング (Train):** 50%,
   **テスト:** 25%,
   **ホールドアウト (Holdout):** 25%

9. **「見積もりプログラムの追加 (Add Estimators)」**をクリックし、**「デシジョン・ツリー分類器 (Decision Tree Classifier)」**、**「追加 (Add)」**の順に選択します。

   一度に複数の見積もりプログラムを評価できます。例えば、見積もりプログラムとして**「デシジョン・ツリー分類器 (Decision Tree Classifier)」**と**「ランダム・フォレスト分類器 (Random Forest Classifier)」**を追加して、モデルをトレーニングし、評価出力に基づいて最適なモデルを選択できます。
   {:tip}

10. **「次へ」**をクリックしてモデルをトレーニングします。**「トレーニングおよび評価済み (Trained & Evaluated)」**ステータスが表示されたら、**「保存」**をクリックします。
   ![](images/solution22-build-machine-learning-model/trained_model.png)

11. **「概要」**をクリックして、モデルの詳細を確認します。

## モデルのデプロイおよび API の試行

{:#deploy_model}

1. 作成したモデルの下で、**「デプロイメント」**>**「デプロイメントの追加 (Add Deployment)」**をクリックします。
2. **「Web サービス (Web Service)」**を選択します。`iris_deployment` という名前およびオプションの説明を追加します。
3. **「保存」**をクリックします。概要ページで、新しい Web サービスの名前をクリックします。ステータスが **DEPLOY_SUCCESS** になったら、**「実装 (Implementation)」**で、スコアリング・エンドポイント、さまざまなプログラミング言語のコード・スニペット、および API 仕様を確認できます。
4. **「API 仕様の表示 (View API Specification)」**をクリックして、{{site.data.keyword.pm_short}} API エンドポイントを表示およびテストします。
   ![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   API の操作を開始するには、[{{site.data.keyword.Bluemix_short}} リソース・リスト](https://{DomainName}/resources/)の{{site.data.keyword.pm_short}}サービス・インスタンスの**「サービス資格情報」**タブで使用可能な**ユーザー名**と**パスワード**を使用して、**アクセス・トークン**を生成する必要があります。API 仕様ページで示される指示に従って、**アクセス・トークン**を生成します。
   {:tip}
5. オンライン予測を行うには、`POST /online` API 呼び出しを使用します。
   * `instance_id` は、[{{site.data.keyword.Bluemix_short}} リソース・リスト](https://{DomainName}/resources/)の{{site.data.keyword.pm_short}}サービスの**「サービス資格情報」**タブにあります。
   * `deployment_id` および `published_model_id` は、デプロイメントの**「概要」**の下にあります。
   *  `online_prediction_input` には、以下の JSON を使用します。

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * **「試す (Try it out)」**をクリックして、JSON 出力を表示します。

6. API エンドポイントを使用して、アプリケーションからこのモデルを呼び出すことができるようになりました。

## モデルのテスト

{:#test_model}

1. **「テスト」**の下に、自動的に取り込まれた入力データ (フィーチャー・データ) が表示されます。
2. **「予測 (Predict)」**をクリックすると、**「品種の予測値 (Predicted value for species)」**がグラフで表示されます。
   ![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. JSON 入出力については、アクティブな入出力の横にあるアイコンをクリックします。
4. 入力データを変更して、モデルのテストを続行できます。

## フィードバック・データ接続の作成

{:#create_feedback_connection}

1. 学習とモデル評価を継続的に行うには、新しいデータをどこかに格納する必要があります。フィードバック・データ接続として機能する [{{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse) サービスの**エントリー**・プランを作成します。
2. {{site.data.keyword.dashdbshort}} の**「管理」**ページで、**「開く」**をクリックします。上部のナビゲーションで、**「ロード (Load)」**を選択します。
3. **「マイ・コンピューター (My computer)」**で**「ファイルの参照 (browse files)」**をクリックし、`iris_initial.csv` をアップロードします。**「次へ」**をクリックします。
4. **「スキーマ」**として **DASHXXXX** (DASH1234 など) を選択し、**「新規テーブル (New Table)」**をクリックします。`IRIS_FEEDBACK` という名前を付けて**「次へ」**をクリックします。
5. データ型が自動的に検出されます。**「次へ」**、**「ロードの開始 (Begin Load)」**の順にクリックします。
   ![](images/solution22-build-machine-learning-model/define_table.png)
6. 新しいターゲット **DASHXXXX.IRIS_FEEDBACK** が作成されます。

   これを次のステップで使用して、パフォーマンスと精度が向上するようにモデルをリトレーニングします。

## モデルのリトレーニング

{:#retrain_model}

1. [{{site.data.keyword.Bluemix_short}} リソース・リスト](https://{DomainName}/resources)に戻り、使用している {{site.data.keyword.DSX_short}} サービスの下で、**「プロジェクト」**>「iris_project」>**「iris-model」**(資産の下) >「評価 (Evaluation)」をクリックします。
2. **「パフォーマンス・モニター (Performance Monitoring)」**で、**「パフォーマンス・モニターの構成 (Configure Performance Monitoring)」**をクリックします。
3. 「パフォーマンス・モニターの構成 (Configure Performance Monitoring)」ページで、以下を実行します。
   * Spark サービスを選択します。予測タイプが自動的に取り込まれます。
   * メトリックとして **weightedPrecision** を選択し、オプションのしきい値として `0.98` を設定します。
   * **「新規接続の作成 (Create new connection)」**をクリックして、上記のセクションで作成したクラウド上の IBM Db2 ウェアハウスを指すようにします。
   * Db2 ウェアハウス接続を選択します。接続の詳細が取り込まれたら、**「作成」**をクリックします。
     ![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * **「フィードバック・データ参照の選択」**をクリックし、IRIS_FEEDBACK テーブルを指して**「選択」**をクリックします。
     ![](images/solution22-build-machine-learning-model/select_source.png)
   * **「再評価に必要なレコード数 (Record count required for re-evaluation)」**ボックスに、リトレーニングをトリガーする新規レコードの最小数を入力します。**10** を使用するか、空白のままにしてデフォルト値の 1000 を使用します。
   * **「自動リトレーニング (Auto retrain)」**ボックスで、以下のいずれかのオプションを選択します。
     - モデルのパフォーマンスが設定したしきい値を下回るたびに自動リトレーニングを開始するには、**「モデルのパフォーマンスがしきい値を下回った場合 (when model performance is below threshold)」**を選択します。このチュートリアルでは、精度がしきい値未満であるため (.98)、このオプションを選択します。
     - 自動リトレーニングが実行されないようにするには、**「なし」**を選択します。
     - パフォーマンスに関係なく自動リトレーニングを開始するには、**「常に」**を選択します。
   * **「自動デプロイ (Auto deploy)」**ボックスで、以下のいずれかのオプションを選択します。
     - モデルのパフォーマンスが前バージョンより向上するたびに自動デプロイメントを開始するには、**「モデルのパフォーマンスが前バージョンより向上した場合 (when model performance is better than previous version)」**を選択します。このチュートリアルの目的は、モデルのパフォーマンスを継続的に高めることであるため、このオプションを選択します。
     - 自動デプロイメントが実行されないようにするには、**「なし」**を選択します。
     - パフォーマンスに関係なく自動デプロイメントを開始するには、**「常に」**を選択します。
   * **「保存」**をクリックします。
     ![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. ファイル [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv) をダウンロードします。その後、**「フィードバック・データの追加 (Add feedback data)」**をクリックし、ダウンロードした csv ファイルを選択して**「開く」**をクリックします。
5. **「新規評価 (New evaluation)」**をクリックして開始します。
     ![](images/solution22-build-machine-learning-model/retraining_model.png)
6. 評価が完了したら、**「最終評価結果 (Last Evalution Result)」**セクションで **WeightedPrecision** 値が向上しているか確認できます。

## リソースの削除
{:removeresources}

1. [{{site.data.keyword.Bluemix_short}} リソース・リスト](https://{DomainName}/resources/)にナビゲートし、サービスを作成した場所、組織、およびスペースを選択します。
2. それぞれの {{site.data.keyword.DSX_short}}、{{site.data.keyword.sparks}}、{{site.data.keyword.pm_short}}、{{site.data.keyword.dashdbshort}}、およびこのチュートリアルで作成した {{site.data.keyword.cos_short}} サービスを削除します。

## 関連コンテンツ
{:related}

- [Watson Studio の概要](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [機械学習を使用した異常の検出](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [自動モデル作成](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [機械学習と AI](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
