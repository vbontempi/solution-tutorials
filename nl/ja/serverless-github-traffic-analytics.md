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

# サーバーレスと Cloud Foundry の組み合わせによるデータの取得と分析
{: #serverless-github-traffic-analytics}
このチュートリアルでは、リポジトリーの GitHub トラフィック統計情報を自動的に収集し、トラフィック分析の基盤を提供するアプリケーションを作成します。GitHub では、過去 14 日間のトラフィック・データのみにアクセスできます。これよりも長い期間の統計情報を分析する場合は、データを自分自身でダウンロードして保管する必要があります。このチュートリアルでは、トラフィック・データを取得して SQL データベースに保管するサーバーレス・アクションをデプロイします。さらに、リポジトリーを管理し、データ分析のために統計情報にアクセスできるようにするため、Cloud Foundry アプリを使用しています。このチュートリアルで説明するアプリとサーバーレス・アクションは、シングル・テナント・モードをサポートする初期機能セットを使用してマルチテナント対応ソリューションを実装します。

![](images/solution24-github-traffic-analytics/Architecture.png)

## 達成目標

* マルチテナントに対応し、保護されたアクセスを備えた Python データベース・アプリをデプロイする
* アプリ ID を OpenID Connect ベースの認証プロバイダーとして統合する
* GitHub トラフィック統計情報の自動サーバーレス収集をセットアップする
* グラフィカル・トラフィック分析のために {{site.data.keyword.dynamdashbemb_short}} を統合する

## 製品
このチュートリアルでは、以下の製品を使用します。
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## 始める前に
{: #prereqs}

このチュートリアルを実行するには、最新バージョンの [IBM Cloud CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) と {{site.data.keyword.openwhisk_short}} [ プラグインがインストールされている必要があります](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)。

## サービスと環境のセットアップ (シェル)
このセクションでは、必要なサービスを設定し、環境を準備します。すべての作業はシェル環境から行うことができます。

1. [GitHub リポジトリー](https://github.com/IBM-Cloud/github-traffic-stats)を複製し、複製したディレクトリーとその **backend** サブディレクトリーに移動します。
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. `ibmcloud login` を使用して {{site.data.keyword.Bluemix_short}} に対話式にログインします。詳細を再確認するには、`ibmcloud target` コマンドを実行します。組織とスペースを設定している必要があります。

3. **エントリー (Entry)** プランを使用して {{site.data.keyword.dashdbshort}} インスタンスを作成し、**ghstatsDB** という名前を付けます。
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. 後で {{site.data.keyword.openwhisk_short}} からデータベース・サービスにアクセスするための許可が必要です。このためサービス資格情報を作成し、**ghstatskey** というラベルを付けます。   
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. {{site.data.keyword.appid_short}} サービスのインスタンスを作成します。名前として **ghstatsAppID** を使用し、**段階課金制 (Graduated tier)** プランを使用します。
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
次に、Cloud Foundry スペースで新しいサービス・インスタンスの別名を作成します。**YOURSPACE** をデプロイ先スペースに置き換えます。
```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. **ライト (lite)** プランを使用して {{site.data.keyword.dynamdashbemb_short}} サービスのインスタンスを作成します。
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
新しいサービス・インスタンスの別名を作成し、**YOURSPACE** を置き換えます。
```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. **backend** ディレクトリーでアプリケーションを IBM Cloud にプッシュします。このコマンドはユーザーのアプリケーションに対してランダムな経路を使用します。
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
デプロイメントが終了するまで待ちます。アプリケーション・ファイルがアップロードされ、ランタイム環境が作成され、サービスがアプリケーションにバインドされます。サービス情報は `manifest.yml` ファイルから取得されます。他のサービス名を使用した場合は、このファイルを更新する必要があります。プロセスが正常に完了すると、アプリケーション URI が表示されます。

   上記のコマンドはアプリケーションに対してランダムで一意の経路を使用します。経路を自分自身で選択する場合は、コマンドの追加パラメーターとして経路を追加します (例: `ibmcloud cf push your-app-name`)。また、`manifest.yml` ファイルを編集し、**name** を変更し、**random-route** を **true** から **false** に変更することもできます。
{:tip}

## アプリ ID と GitHub 構成 (ブラウザー)
以下のステップはすべて、インターネット・ブラウザーを使用して行います。最初に {{site.data.keyword.appid_short}} を構成して、Cloud Directory を使用し、Python アプリと連携するようにします。その後、GitHub アクセス・トークンを作成します。デプロイされている機能がトラフィック・データを取得する必要があります。

1. [{{site.data.keyword.Bluemix_short}} リソース・リスト](https://{DomainName}/resources)で、サービスの概要を開きます。**「サービス」**セクションで {{site.data.keyword.appid_short}} サービスのインスタンスを見つけます。そのエントリーをクリックして詳細を開きます。
2. サービス・ダッシュボードの左側にあるメニューの**「ID プロバイダー」**の下にある**「管理」**をクリックします。使用可能な ID プロバイダー (Facebook、Google、SAML 2.0 Federation、Cloud Directory など) のリストが表示されます。Cloud Directory を**「オン」**に切り替え、その他のプロバイダーを**「オフ」**に切り替えます。
   
   [多要素認証 (MFA)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) と高度なパスワード・ルールを構成することがあります。これについては、このチュートリアルでは説明しません。
{:tip}

3. ページ下部にリダイレクト URL のリストがあります。アプリケーションの **url** と /redirect_uri を組み合わせて入力します。例えば `https://github-traffic-stats-random-word.mybluemix.net/redirect_uri` などです。

   アプリをローカルでテストする場合のリダイレクト URL は `http://0.0.0.0:5000/redirect_uri` です。複数のリダイレクト URL を構成できます。
{:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. 左側のメニューで**「ユーザー」**をクリックします。Cloud Directory でユーザー・リストが開きます。**「ユーザーの追加」**ボタンをクリックし、ユーザー自身を最初のユーザーとして追加します。これで、{{site.data.keyword.appid_short}} サービスの構成が完了しました。
5. ブラウザーで [Github.com](https://github.com/settings/tokens) にアクセスし、**「Settings」->「Developer settings」->「Personal access tokens」**の順に移動します。**「Generate new token」**ボタンをクリックします。**「Token description」**に**「GHStats Tutorial」**と入力します。その後、**repo** カテゴリーの下の **public_repo** と、**admin:org** の下の **read:org** を有効にします。次に、そのページ下部にある**「Generate token」**をクリックします。新しいアクセス・トークンが次のページに表示されます。これは、以降のアプリケーション設定で必要になります。
![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## Python アプリの構成とテスト
準備が完了したら、アプリを構成してテストします。アプリは、人気の高い [Flask](http://flask.pocoo.org/) マイクロフレームワークを使用して Python で作成されています。統計情報収集でリポジトリーを追加または削除できます。トラフィック・データには表形式のビューでアクセスできます。

1. ブラウザーで、デプロイされているアプリの URI を開きます。ウェルカム・ページが表示されます。
   ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. ブラウザーで URI に `/admin/initialize-app` を追加し、そのページにアクセスします。これは、アプリケーションとそのデータを初期化するために使用されます。**「Start initialization」**ボタンをクリックします。これにより、パスワードで保護されている構成ページが表示されます。ログインに使用した E メール・アドレスがシステム管理者の ID として取得されます。以前に構成した E メール・アドレスとパスワードを使用します。

3. 構成ページで、名前 (あいさつで使用されている場合)、GitHub ユーザー名、および以前に生成したアクセス・トークンを入力します。**「Initialize」**をクリックします。データベース表が作成され、一部の構成値が挿入されます。最後に、システム管理者とテナントのデータベース・レコードが作成されます。
![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. 完了すると、管理対象リポジトリーのリストが表示されます。リポジトリーを追加できます。追加するには、GitHub アカウントまたは組織の名前とリポジトリーの名前を指定します。データの入力が完了したら、**「Add repository」**をクリックします。リポジトリーと新規に割り当てられた ID が表に表示されます。リポジトリーをシステムから削除できます。削除するには、リポジトリーの ID を入力して**「Delete repository」**をクリックします。
![](images/solution24-github-traffic-analytics/RepositoryList.png)

## Cloud の機能とトリガーのデプロイ
管理アプリが導入された状態で、{{site.data.keyword.openwhisk_short}} でこの 2 つを接続するためのアクション、トリガー、およびルールをデプロイします。これらのオブジェクトは、指定されたスケジュールで GitHub トラフィック・データを自動的に収集するために使用されます。このアクションはデータベースに接続し、すべてのテナントとそのリポジトリーで処理を繰り返し、各リポジトリーのビューと複製データを取得します。これらの統計情報がデータベースにマージされます。

1. **functions** ディレクトリーに移動します。
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. 新しいアクション **collectStats** を作成します。このアクションは、必要なデータベース・ドライバーが既に含まれている [Python 3 環境](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments)を使用します。アクションのソース・コードはファイル `ghstats.zip` に含まれています。
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   アクションのソース・コード (`__main__.py`) を変更した場合は、`zip -r ghstats.zip  __main__.py github.py` を再度使用して zip アーカイブを再パッケージできます。詳しくは、ファイル `setup.sh` を参照してください。
{:tip}
3. アクションをデータベース・サービスにバインドします。環境のセットアップ中に作成したサービス・キーとインスタンスを使用します。
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. [アラーム・パッケージ](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm)に基づいてトリガーを作成します。アラームはさまざまな形式で指定できます。[cron](https://en.wikipedia.org/wiki/Cron) のようなスタイルを使用します。トリガーは、4 月 21 日から 12 月 21 日まで、毎日 UTC 時間で午前 6 時に発生します。必ず、将来の開始日を指定してください。
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  トリガーを日次スケジュールから週次スケジュールに変更するには、`"0 6 * * 0"` を適用します。これにより、トリガーは毎週日曜日午前 6 時に発生します。
{:tip}
5. 最後に、トリガー **myDaily** を **collectStats** アクションに接続するルール **myStatsRule** を作成します。これで、トリガーにより、以前のステップで指定したスケジュールでアクションが実行されます。
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. 初期テスト実行のため、アクションを呼び出します。返される **repoCount** には、以前に構成したリポジトリーの数が反映されています。
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
出力は次のようになります。
```
   {
       "repoCount": 18
   }
   ```
7. アプリ・ページが表示されているブラウザー・ウィンドウで、リポジトリー・トラフィックを確認できます。デフォルトでは 10 件のエントリーが表示されます。この数を変更できます。また、表の列をソートしたり、検索ボックスを使用してフィルタリングし、特定のリポジトリーに絞り込んだりすることもできます。日付と組織名を入力して、viewcount に基づいてソートし、特定の日付の上位スコア獲得リポジトリーをリストできます。
![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## まとめ
このチュートリアルでは、サーバーレス・アクションと関連トリガーおよびルールをデプロイしました。これにより、GitHub リポジトリーのトラフィック・データを自動的に取得できます。これらのリポジトリーに関する情報 (テナント固有のアクセス・トークンなど) は SQL データベースに保管されます ({{site.data.keyword.dashdbshort}})。このデータベースは、ユーザーとリポジトリーの管理と、アプリ・ポータルでのトラフィック統計情報の表示のために Cloud Foundry アプリにより使用されます。トラフィック統計情報は、検索可能な表に表示されるか、または組み込みダッシュボードで視覚化されます ({{site.data.keyword.dynamdashbemb_short}} サービス、以下の画像を参照)。また、リポジトリーのリストとトラフィック・データを CSV ファイルとしてダウンロードできます。

Cloud Foundry アプリは、{{site.data.keyword.appid_short}} に接続している OpenID Connect クライアントを使用したアクセスを管理します。
![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## クリーンアップ
このチュートリアルで使用したリソースをクリーンアップするには、関連サービスとアプリ、アクション、トリガー、ルールを、作成順とは逆の順序で削除できます。

1. {{site.data.keyword.openwhisk_short}} ルール、トリガー、およびアクションを削除します。
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. Python アプリとそのサービスを削除します。
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## チュートリアルを発展させる
このチュートリアルに追加またはチュートリアルを変更する必要がありますか?以下にいくつかのアイデアを示します。
* アプリを拡張してマルチテナント対応にする。
* データのグラフを統合する。
* ソーシャル・アイデンティティー・プロバイダーを使用する。
* 表示データをフィルタリングできるように、日付ピッカーを統計情報ページに追加する。
* {{site.data.keyword.appid_short}} のカスタム・ログイン・ページを使用する。
* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) を使用して、開発者間のソーシャル・コーディングの関係を広げる。

## 関連コンテンツ
このチュートリアルで扱ったトピックに関する追加情報へのリンクを以下に示します。

資料と SDK:
* [{{site.data.keyword.openwhisk_short}} の資料](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* 資料: [IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.appid_short}} の資料](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [IBM Cloud の Python ランタイム](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
