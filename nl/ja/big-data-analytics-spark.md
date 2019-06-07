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

# Apache Spark によるオープン・データの分析と視覚化
{: #big-data-analytics-spark}

このチュートリアルでは、{{site.data.keyword.DSX_full}}、Jupyter Notebook および Apache Spark を使用して、オープン・データ・セットを分析して視覚化します。人口増加、平均余命および国別 ISO コードを記載したデータを単一のデータ・フレームに結合することから開始します。その後、洞察を得るために、Pixiedust と呼ばれる Python ライブラリーを使用して、さまざまな方法でデータを照会して視覚化します。

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## 達成目標
{: #objectives}

* IBM Cloud での Apache Spark および {{site.data.keyword.DSX_short}} のデプロイ
* Jupyter Notebook と Python カーネルを使用した作業
* データ・セットのインポート、変換、分析および視覚化

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## サービスと環境のセットアップ
このチュートリアルで使用するサービスのプロビジョニングから開始して、{{site.data.keyword.DSX_short}} 内でプロジェクトを作成します。

{{site.data.keyword.Bluemix_short}} のサービスは、[「リソース・リスト」](https://{DomainName}/resources)および[「カタログ」](https://{DomainName}/catalog/)からプロビジョンできます。また、{{site.data.keyword.DSX_short}} を使用すると、ダッシュボードおよびプロジェクト設定から Data & Analytics サービスを作成したり既存のものを追加したりできます。
{:tip}

1. [{{site.data.keyword.Bluemix_short}} カタログ](https://{DomainName}/catalog)で**「AI」**セクションにナビゲートします。**{{site.data.keyword.DSX_short}}** サービスを作成します。**「開始」**ボタンをクリックして、**{{site.data.keyword.DSX_short}}** ダッシュボードを起動します。
2. ダッシュボードで、**「プロジェクトの作成 (Create a project)」**タイルをクリックし、**「標準」**>「プロジェクトの作成 (Create Project)」の順に選択します。**「名前」**フィールドに、名前として `1stProject` を入力します。説明は空のままでかまいません。
3. ページの右側で、**ストレージを定義**できます。ストレージを既にプロビジョンしている場合は、リストからインスタンスを選択します。そうでない場合は、**「追加 (Add)」**をクリックして、新規ブラウザー・タブの手順に従います。サービスの作成が完了したら、**「最新表示 (Refresh)」**をクリックして、新規サービスを表示します。
4. **「作成」**ボタンをクリックして、プロジェクトを作成します。プロジェクトの概要ページにリダイレクトされます。  
   ![](images/solution23/NewProject.png)
5. 概要ページで、**「設定 (Settings)」**をクリックします。
6. **「関連サービス (Associated services)」**セクションで、**「サービスの追加 (Add Service)」**をクリックして、メニューから**「Spark」**を選択します。結果画面で、既存の Spark サービス・インスタンスを選択するか、新規に作成することができます。

## ノートブックの作成と準備
[Jupyter Notebook](http://jupyter.org/) は、ライブ・コード、式、視覚化および説明テキストを含む文書を作成して共有できるオープン・ソースの Web アプリケーションです。ノートブックやその他のリソースはプロジェクトに編成されます。
1. **「アセット (Assets)」**タブをクリックして、**「ノートブック (Notebooks)」**セクションまでスクロールダウンし、**「新規ノートブック (New notebook)」**をクリックします。
2. **「空白 (Blank)」**のノートブックを使用します。**「名前」**に `MyNotebook` を入力します。
3. **「ランタイムの選択 (Select runtime)」**メニューから、プロジェクトの設定に追加した Spark インスタンスを選択します。**「言語 (Language)」**はデフォルトの **Python 3.5** のままにします。
4. **「ノートブックの作成 (Create Notebook)」**をクリックして処理を完了します。
5. テキストやコマンドを入力するフィールドは、**セル**と呼ばれます。以下のコードを空のセルにコピーして、[**Pixiedust** パッケージをインポートします](https://pixiedust.github.io/pixiedust/use.html)。ツールバーの**「実行 (Run)」**アイコンをクリックするか、キーボードで **Shift と Enter** キーを押して、セルを実行します。
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

Jupyter Notebooks で作業したことがない場合は、メニュー右上の**「資料 (Docs)」**アイコンをクリックします。**「データの分析 (Analyze data)」**の[**「ノートブック (Notebooks)」**セクション](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics)にナビゲートし、[ノートブックやそのパーツ](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true)について参照してください。
{:tip}

## データのロード
次に、3 つのオープン・データ・セットをロードして、ノートブック内で使用できるようにします。**Pixiedust** ライブラリーを使用すると、容易に [URL を使用して **CSV** ファイルをロード](https://pixiedust.github.io/pixiedust/loaddata.html)できます。

1.  以下の行をノートブックの次の空のセルにコピーしますが、まだ実行しません。
   ```Python
   df_pop = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
2. 別のブラウザー・タブで、[「コミュニティー (Community)」](https://dataplatform.ibm.com/community?context=analytics)セクションに移動します。**「データ・セット (Data Sets)」**で、[**「国別総人口 (Total population by country)」**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e)を検索して、そのタイルをクリックします。右上の**「リンク (link)」**アイコンをクリックして、アクセス URI を取得します。URI をコピーして、ノートブック・セルのテキスト **YourAccessURI** をこのリンクに置き換えます。ツールバーの**「実行 (Run)」**アイコンをクリックするか、**Shift と Enter** キーを押します。
3. 別のデータ・セットについてもこの手順を繰り返します。以下の行をノートブックの次の空のセルにコピーします。
   ```Python
   df_life = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
4. 別のブラウザー・タブの**「データ・セット (Data Sets)」**で、[**「年間の国別出生時平均余命 (Life expectancy at birth by country in total years)」**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895)を検索します。同様にリンクを取得し、それを使用してノートブック・セルの **YourAccessURI** を置き換えて**実行**し、ロード処理を開始します。
5. 3 つのうち最後のデータ・セットについては、Github のオープン・データ・セットの集合から国名とその ISO コードのリストをロードします。コードを次の空のノートブック・セルにコピーして実行します。
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

後で、国別コードのリストを使用して、正確な国名の代わりに国別コードを使用することによって、データの選択が簡素化されます。

## データの変換
データが使用可能になったら、わずかな変換を行い、3 つのセットを単一のデータ・フレームに結合します。
1. 以下のコード・ブロックは、人口データのデータ・フレームを再定義します。これは列の名前を変更する SQL ステートメントで実行されます。ビューが作成され、スキーマが出力されます。以下のコードを次の空のセルにコピーして実行します。
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. 平均余命データについても同様に繰り返します。以下のコードでは、スキーマ出力の代わりに、最初の 10 行が出力されます。  
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. 国別データについてもスキーマの変換を繰り返します。
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. これで、列名が単純化されてデータ・セット全体で同じになるため、データ・セットを単一のデータ・フレームに結合できます。平均余命データと人口データに**外部**結合を実行します。その後、同じステートメントで、**内部**結合を実行して国別コードを追加します。すべてが国別および年別で順序付けされます。出力にはデータ・フレーム **df_all** を定義します。内部結合を使用することによって、結果データには ISO リストにある国のみが含まれます。この処理により、地域やその他の項目がデータから削除されます。
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. **Year** のデータ・タイプを整数に変更します。
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

結合されたデータを分析する準備ができました。

## データの分析
ここでは、[データをさまざまなグラフで視覚化するために Pixiedust](https://pixiedust.github.io/pixiedust/displayapi.html) を使用します。まず、いくつかの国の平均余命を比較します。

1. コードを次の空のセルにコピーして実行します。
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. スクロール可能な表が表示されます。コード・ブロックのすぐ下にあるグラフ・アイコンをクリックして、**「折れ線グラフ (Line Chart)」**を選択します。ポップアップ・ダイアログ**「Pixiedust: 折れ線グラフのオプション (Line Chart Options)」**が表示されます。**「グラフのタイトル (Chart Title)」**に「平均余命の比較 (Comparison of Life Expectancy)」などを入力します。提供された**「フィールド (Fields)」**から、**「年 (Year)」**を**「キー (Keys)」**ボックスに、**「寿命 (Life)」**を**「値 (Values)」**エリアにドラッグします。**「表示する行数 (# of Rows to Display)」**に **1000** を入力します。**「OK」**を押すと、折れ線グラフが作図されます。右側にある**「レンダラー (Renderer)」**で、**「mapplotlib」**が選択されていることを確認します。**「クラスター基準 (Cluster By)」**のセレクターをクリックして、**「国 (Country)」**を選択します。以下のようなグラフが表示されます。
   ![](images/solution23/LifeExpectancy.png)

3. 2010 年に焦点を当てたグラフを作成します。コードを次の空のセルにコピーして実行します。
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. グラフのセレクターで、**「地図 (Map)」**を選択します。構成ダイアログで、**「国 (Country)」**を**「キー (Keys)」**エリアにドラッグします。**「寿命 (Life)」**を**「値 (Values)」**ボックスに移動します。最初のグラフと同様に、**「表示する行数 (# of Rows to Display)」**を **1000** に増やします。**「OK」**を押すと、地図が作図されます。**「レンダラー (Renderer)」**として**「brunel」**を選択します。平均余命に関連付けて色分けされた世界地図が表示されます。マウスを使用して地図を拡大できます。
   ![](images/solution23/LifeExpectancyMap2010.png)

## チュートリアルを発展させる
このチュートリアルを強化するアイデアや提案を以下にいくつか示します。
* 選択した国について、人口増加に対して相対的な平均余命率を示す照会を作成して視覚化する
* 国ごとの人口増加率を計算して世界地図で視覚化する
* データ・セットのカタログから追加データを読み込み統合する
* 結合されたデータをファイルまたはデータベースにエクスポートする

## 関連コンテンツ
{:related}
このチュートリアルで取り上げたトピックに関連したリンクを以下に示します。
* [Watson Data Platform](https://dataplatform.ibm.com): Watson Data Platform を使用して、共同作業でよりスマートなアプリケーションを作成します。データの迅速な視覚化により洞察を得ることができ、チーム全体でのコラボレーションが可能です。
* [PixieDust](https://www.ibm.com/cloud/pixiedust): Jupyter Notebooks のオープン・ソースの生産性向上ツール
* [Cognitive Class.ai](https://cognitiveclass.ai/): データ・サイエンスとコグニティブ・コンピューティングについてのコース
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/): データを使ってできること
* [Analytics Engine サービス](https://{DomainName}/catalog/services/analytics-engine): オープン・ソースの Apache Spark および Apache Hadoop を使用して分析アプリケーションを開発してデプロイします
