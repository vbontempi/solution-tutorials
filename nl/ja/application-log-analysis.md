---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# LogDNA と Sysdig によるログの分析とアプリケーションの正常性のモニター
{: #application-log-analysis}

このチュートリアルでは、[{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/) サービスを使用して、{{site.data.keyword.Bluemix_notm}} にデプロイされている Kubernetes アプリケーションのログを構成し、アクセスする方法を示します。Python アプリケーションを {{site.data.keyword.containerlong_notm}} でプロビジョンされたクラスターにデプロイし、LogDNA エージェントを構成し、さまざまなレベルのアプリケーション・ログを生成して、ワーカー・ログ、ポッド・ログ、またはネットワーク・ログにアクセスします。 その後、{{site.data.keyword.la_short}} Web UI を使用してこれらのログを検索、フィルタリング、および視覚化します。

さらに、[{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/) サービスをセットアップし、Sysdig エージェントを構成して、アプリケーションおよび {{site.data.keyword.containerlong_notm}} クラスターのパフォーマンスと正常性をモニターします。
{:shortdesc}

## 達成目標
{: #objectives}
* アプリケーションを Kubernetes クラスターにデプロイして、ログ・エントリーを生成します。
* さまざまなタイプのログにアクセスして分析し、問題をトラブルシューティングしたり、問題に先手を打ったりします。
* アプリおよびアプリが実行されているクラスターのパフォーマンスと正常性を可視化して運用に役立てることができます。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

このチュートリアルでは、費用が発生する場合があります。 [料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

  ![](images/solution12/Architecture.png)

1. ユーザーがアプリケーションに接続し、ログ・エントリーを生成します。
1. アプリケーションが、{{site.data.keyword.registryshort_notm}} に保管されたイメージから Kubernetes クラスターで実行されます。
1. ユーザーが、アプリケーション・レベルおよびクラスター・レベルのログにアクセスするための {{site.data.keyword.la_full_notm}} サービス・エージェントを構成します。
1. ユーザーが、{{site.data.keyword.containerlong_notm}} クラスターおよびこのクラスターにデプロイされているアプリのパフォーマンスと正常性をモニターするための {{site.data.keyword.mon_full_notm}} サービス・エージェントを構成します。

## 前提条件
{: #prereq}

* [{{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) をインストールします (docker、kubectl、helm、ibmcloud cli、および必要なプラグインをインストールするスクリプト)。
* [{{site.data.keyword.registrylong_notm}} CLI およびレジストリー名前空間](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)をセットアップします。
* [LogDNA でログを表示するための許可をユーザーに付与します](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna)
* [Sysdig でメトリックを表示するための許可をユーザーに付与します](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig)

## Kubernetes クラスターの作成
{: #create_cluster}

{{site.data.keyword.containershort_notm}} は、Kubernetes クラスターで実行される Docker コンテナーに高可用性のアプリをデプロイする環境を提供します。

既存の**標準**クラスターをこのチュートリアルで再利用する場合は、このセクションをスキップしてください。
{: tip}

1. [{{site.data.keyword.Bluemix}} カタログ](https://{DomainName}/kubernetes/catalog/cluster/create)から**新規クラスター**を作成し、**標準**クラスターを選択します。
   ログ転送は、**フリー**クラスターでは有効では*ありません*。
   {:tip}
1. リソース・グループおよびジオグラフィーを選択します。
1. このチュートリアルでの一貫性を保つために、便宜上、`mycluster` という名前を使用します。
1. **「ワーカー・ゾーン (Worker Zone)」**を選択し、**CPU** が 2 つで 4 **GB RAM** の最も小さい**マシン・タイプ**を選択します。このチュートリアルには、これで十分です。
1. **ワーカー・ノード**を 1 つ選択し、その他すべてのオプションはデフォルトのままにしておきます。 **「クラスターの作成」**をクリックします。
1. **クラスター**と**ワーカー・ノード**の状況を確認し、**「ready」**になるまで待機します。

## {{site.data.keyword.la_short}} インスタンスのプロビジョン
{: #provision_logna_instance}

{{site.data.keyword.Bluemix_notm}} の {{site.data.keyword.containerlong_notm}} クラスターにデプロイされたアプリケーションでは、あるレベルの診断出力 (ログ) が生成される場合があります。 開発者またはオペレーターは、ワーカー・ログ、ポッド・ログ、アプリ・ログ、ネットワーク・ログなど、さまざまなタイプのログにアクセスして分析し、問題をトラブルシューティングしたり、問題に先手を打ったりできます。

{{site.data.keyword.la_short}} サービスを使用すると、さまざまなソースからのログを集約して、必要な期間だけ保持できます。 これにより、必要に応じて大局的に分析し、より複雑な状況をトラブルシューティングできるようになります。

{{site.data.keyword.la_short}} サービスをプロビジョンするには、以下のようにします。

1. [プログラム識別情報](https://{DomainName}/observe/)ページにナビゲートし、**「ロギング」**で**「インスタンスの作成 (Create instance)」**をクリックします。
1. 一意の**サービス名**を指定します。
1. 地域/場所を選択し、リソース・グループを選択します。
1. プランとして**「7 日間ログ検索 (7 day Log Search)」**を選択し、**「作成」**をクリックします。

このサービスでは、ログ・データが IBM Cloud でホストされる中央ログ管理システムが提供されます。

## ログを転送する Kubernetes アプリのデプロイと構成
{: #deploy_configure_kubernetes_app}

[ロギング・アプリでそのまま実行できるコードは、この GitHub リポジトリーにあります](https://github.com/IBM-Cloud/application-log-analysis)。 アプリケーションは、よく利用される Python サーバー・サイド Web フレームワーク [Django](https://www.djangoproject.com/) を使用して作成されます。 リポジトリーを複製またはダウンロードして、アプリを {{site.data.keyword.Bluemix_notm}} の {{site.data.keyword.containershort_notm}} にデプロイします。

### Python アプリケーションのデプロイ

端末:
1. 以下のようにして、Github リポジトリーを複製します。
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. アプリケーション・ディレクトリーに移動します
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. {{site.data.keyword.registryshort_notm}} で [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile) を使用して Docker イメージをビルドします。
   - `ibmcloud cr info` を使用して、us.icr.io や uk.icr.io などの**コンテナー・レジストリー**を見つけます。
   - コンテナー・イメージを保管する名前空間を作成します。
      ```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - `<CONTAINER_REGISTRY>` をコンテナー・レジストリー値で置き換え、イメージ名として **app-log-analysis** を使用します。
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - `app-log-analysis.yaml` ファイルの**イメージ**値をイメージ・タグ `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest` で置き換えます
1. 以下のコマンドを実行して、クラスター構成を取得し、`KUBECONFIG` 環境変数を設定します。
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. アプリをデプロイします。
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. アプリケーションにアクセスするには、ワーカー・ノードの `public IP` と `NodePort` が必要です
   - パブリック IP について、以下のコマンドを実行します。
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - 5 桁 (3xxxx など) の NodePort について、以下のコマンドを実行します。
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   これで、`http://worker-ip-address:portnumber` でアプリケーションにアクセスできます

### LogDNA インスタンスにログを送信するようにクラスターを構成する

ログを {{site.data.keyword.la_full_notm}} インスタンスに送信するように Kubernetes クラスターを構成するには、クラスターの各ノード上に *logdna-agent* ポッドをインストールしなければなりません。 LogDNA エージェントはそれがインストールされたポッドからログ・ファイルを読み取り、LogDNA インスタンスにログ・データを転送します。

1. [「プログラム識別情報」](https://{DomainName}/observe/)ページにナビゲートし、**「ロギング」**をクリックします。
1. 以前に作成したサービスの横にある**「ログ・リソースの編集 (Edit log resources)」**をクリックし、**「Kubernetes (Kubernetes)」**を選択します。
1. `KUBECONFIG` 環境変数を設定してサービス・インスタンスの LogDNA 取り込み鍵で kubernetes シークレットを作成した端末で、最初のコマンドをコピーして実行します。
1. 2 番目のコマンドをコピーして実行し、Kubernetes クラスターの各ワーカー・ノードに LogDNA エージェントをデプロイします。 LogDNA エージェントは、ポッドの /var/log* ディレクトリーに*保管されている **.log 拡張子のファイルと拡張子のないファイル* を使用してログを収集します。デフォルトでは、kube-system を含めすべての名前空間からログが収集され、{{site.data.keyword.la_full_notm}} サービスに自動的に転送されます。
1. ログ・ソースを構成したら、**「LogDNA の表示 (View LogDNA)」**をクリックして LogDNA UI を起動します。ログの表示が開始されるまでに数分かかる場合があります。

## アプリケーション・ログの生成とアクセス
{: generate_application_logs}

このセクションでは、アプリケーション・ログを生成して、LogDNA でレビューします。

### アプリケーション・ログの生成

前のステップでデプロイしたアプリケーションでは、選択したログ・レベルでメッセージをログに記録できます。 使用可能なログ・レベルは、**重大**、**エラー**、**警告**、**情報**、および**デバッグ**です。 アプリケーションのロギング・インフラストラクチャーは、設定レベル以上のログ・エントリーのみを渡すことができるように構成されています。 最初にロガー・レベルは、**警告**に設定されています。 このため、サーバー設定が**警告**になっている場合、**情報**でログに記録されたメッセージは診断出力に表示されません。

[**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py) ファイルのコードを確認してください。 このコードには、**print** ステートメントおよび **logger** 関数の呼び出しが含まれています。 出力されたメッセージは **stdout** ストリーム (標準出力、アプリケーション・コンソール/端末) に書き込まれ、ロガー・メッセージは **stderr** ストリーム (エラー・ログ) に表示されます。

1. `http://worker-ip-address:portnumber` の Web アプリにアクセスします。
1. 異なるレベルのメッセージを処理依頼して、複数のログ・エントリーを生成します。 この UI では、サーバー・ログ・レベルのロガー設定を変更することもできます。 サーバー・サイドのログ・レベルを中間に変更して、より興味深くします。 例えば、「500 internal server error」を**エラー**としてログに記録したり、「This is my first log entry」を**情報**としてログに記録したりできます。

### アプリケーション・ログへのアクセス

フィルターを使用して、LogDNA UI でアプリケーションに固有のログにアクセスできます

1. トップ・バーで**「すべてのアプリ」**をクリックします。
1. コンテナーの下の **app-log-analysis** を確認します。 保存されていない新規ビューがすべてのレベルのアプリケーション・ログとともに表示されます。
1. 特定のログ・レベルのログを表示するには、**「すべてのレベル (All Levels)」**をクリックし、「エラー」、「情報」、「警告」など、複数のレベルを選択します。

## ログの検索およびフィルター
{: #search_filter_logs}

{{site.data.keyword.la_short}} UI では、デフォルトで、使用可能なすべてのログ・エントリーが表示されます (すべて (Everything))。 自動リフレッシュによって、最新のエントリーが一番下に表示されます。
このセクションでは、表示内容および表示量を変更して、今後使用する**ビュー**として保存します。

### ログを検索する

1. LogDNA UI のページの最下部にある**「検索」**入力ボックスでは、以下のことができます。
   - **"This is my first log entry"** や **500 internal server error** など、特定のテキストを含む行を検索できます。
   - `level:info` を入力して特定のログ・レベルを検索できます。ここで、level は、ストリング値を受け入れるフィールドとなります。

   追加の検索フィールドおよびヘルプは、検索入力ボックスの横の構文ヘルプ・アイコンをクリックしてください。
   {:tip}
1. 特定の時間フレームに移動するには、**「時間フレームに移動 (Jump to timeframe)」**入力ボックスに **5 mins ago** と入力します。入力ボックスの横のアイコンをクリックして、保存期間内の他の時刻形式を参照します。
1. 用語を強調表示するには、**「ビューアー・ツールの切り替え (Toggle Viewer Tools)」**アイコンをクリックします。
1. 最初の入力ボックスに強調表示用語として **error** を入力し、2 番目の入力ボックスに強調表示用語として **container** を入力して、これらの用語を含む強調表示された行を確認します。
1. **「タイムラインの切り替え (Toggle Timeline)」**アイコンをクリックして、特定の時刻のログを含む行を表示します。

### ログをフィルタリングする

タグ、ソース、アプリ、またはレベルでログをフィルターに掛けることができます。

1. トップバーで**「すべてのタグ (All Tags)」**をクリックし、**「k8s (k8s)」**チェック・ボックスを選択して、Kubernetes 関連のログを表示します。
1. **「すべてのソース (All Sources)」**をクリックし、ログを確認するホスト (ワーカー・ノード) の名前を選択します。 クラスター内に複数のワーカー・ノードがある場合に役立ちます。
1. コンテナーまたはファイル・ログを確認するには、**「すべてのアプリ」**をクリックし、ログを表示するチェック・ボックスを選択します。

### ビューの作成

ビューは、特定のフィルター・セットおよび検索照会への保存済みショートカットです。

ログを検索したり、ログにフィルターを掛けたりすると、すぐにトップ・バーに**「未保存ビュー (Unsaved View)」**が表示されます。 これをビューとして保存するには、以下のようにします。
1. **「すべてのアプリ」**をクリックし、**app-log-analysis** の横のチェック・ボックスを選択します
1. **「未保存ビュー (Unsaved View)」**をクリックし、**save as new view / alert** をクリックして、ビューに **app-log-analysis-view** という名前を付けます。**「カテゴリー」**は空のままにします。
1. **「ビューの保存 (Save View)」**をクリックすると、アプリのログを示す新しいビューが左側のペインに表示されます。

### グラフと明細によるログの視覚化

このセクションでは、ボードを作成し、明細を含むグラフを追加して、アプリ・レベル・データを視覚化します。 ボードは、グラフと明細のコレクションです。

1. 左側のペインで **board** アイコン (設定アイコンの上) をクリックし、**「新規ボード (NEW BOARD)」**をクリックします。
1. トップ・バーの**「編集」**をクリックして、**app-log-analysis-board** という名前を付けます。 **「保存」**をクリックします。
1. **「グラフの追加 (Add Graph)」**をクリックします。
   - 最初の入力ボックスのフィールドに **app** と入力して、Enter を押します。
   - フィールド値として**「app-log-analysis」**を選択します。
   - **「グラフの追加 (Add Graph)」**をクリックします。
1. メトリックとして**「count」**を選択して、過去 24 時間の各間隔の行数を確認します。
1. 明細を追加するには、グラフの下の矢印をクリックします。
   - 明細タイプとして**「ヒストグラム (Histogram)」**を選択します。
   - フィールド・タイプとして**「レベル (level)」**を選択します。
   - **「明細の追加 (Add Breakdown)」**をクリックして、アプリに関してログに記録したすべてのレベルの明細を表示します。

## {{site.data.keyword.mon_full_notm}} の追加およびクラスターのモニター
{: #monitor_cluster_sysdig}

ここでは、アプリケーションに {{site.data.keyword.mon_full_notm}} を追加します。 サービスによって、アプリの可用性と応答時間が定期的に確認されます。

1. [プログラム識別情報](https://{DomainName}/observe/)ページにナビゲートし、**「モニタリング」**で**「インスタンスの作成 (Create instance)」**をクリックします。
1. 一意の**サービス名**を指定します。
1. 地域/場所を選択し、リソース・グループを選択します。
1. プランとして**「段階的な層 (Graduated Tier)」**を選択し、**「作成」**をクリックします。
1. 以前に作成したサービスの横にある**「ログ・リソースの編集 (Edit log resources)」**をクリックし、**「Kubernetes (Kubernetes)」**を選択します。
1. `KUBECONFIG` 環境変数を設定してクラスターに Sysdig エージェントをデプロイした端末で、**「Sysdig エージェントのクラスターへのインストール (Install Sysdig Agent to your cluster)」**のコマンドをコピーして実行します。デプロイメントの完了を待ちます。

### {{site.data.keyword.mon_short}} の構成

クラスターの正常性とパフォーマンスをモニターするように Sysdig を構成するには、以下のようにします。
1. **「Sysdig の表示 (View Sysdig)」**をクリックすると、sysdig モニター UI が表示されます。 ウェルカム・ページで、**「Next」**をクリックします。
1. セットアップ環境でのインストール方法として**「Kubernetes」**を選択します。
1. エージェント構成成功メッセージの横の**「Go to Next step」**をクリックし、次のページで**「Let's Get started」**をクリックします。
1. **「Next」**、**「Complete onboarding」**の順にクリックして、Sysdig UI の`「Explore」`タブを表示します。

### クラスターのモニター

アプリおよびクラスターの正常性とパフォーマンスを確認するには、以下のようにします。
1. `http://worker-ip-address:portnumber` でのアプリケーションの実行で複数のログ・エントリーを生成します。
1. 左側のペインで **mycluster** を展開し、**default** 名前空間を展開して、**app-log-analysis-deployment** をクリックし、Sysdig モニター・ウィザードで「Request count」、「Response Time」などを確認します。
1. HTTP 要求/応答コードを確認するには、トップ・バーの**「Kubernetes Pod Health」**の横の矢印をクリックし、**「Applications」**で**「HTTP」**を選択します。Sysdig UI のボトム・バーで間隔を **10 M** に変更します。
1. アプリケーションの待ち時間をモニターするには、以下のようにします。
   - 「Explore」タブで**「Deployments and Pods」**を選択します。
   - `HTTP` の横の矢印をクリックして、「Metrics」>「Network」を選択します。
   - **「net.http.request.time」**を選択します。
   - 「Time:」で**「Sum」**を選択し、「Group:」で**「Average」**を選択します。
   - **「More options」**をクリックし、**「Topology」**アイコンをクリックします。
   - **「Done」**をクリックし、ボックスをダブルクリックしてビューを展開します。
1. アプリケーションが実行されている Kubernetes 名前空間をモニターするには、以下のようにします。
   - 「Explore」タブで**「Deployments and Pods」**を選択します。
   - `net.http.request.time` の横の矢印をクリックします。
   - 「Default Dashboards」>「Kubernetes」を選択します。
   - 「Kubernetes State」>「Kubernetes State Overview」を選択します。

### カスタム・ダッシュボードの作成

事前定義されたダッシュボード以外に、独自のカスタム・ダッシュボードを作成して、単一の場所でアプリを実行しているコンテナーの最も役立つ関連ビューおよびメトリックを表示できます。各ダッシュボードは、複数の異なる形式で特定のデータを表示するように構成された一連のパネルで構成されます。

ダッシュボードを作成するには、以下のようにします。
1. 左端のペインで**「Dashboards」**をクリックし、**「Add Dashboard」**をクリックします。
1. **「Blank Dashboard」**をクリックし、ダッシュボードに **Container Request Overview** という名前を付けて、**「Create Dashboard」**をクリックします。
1. 新規パネルとして**「Top List」**を選択し、パネルに **Request time per container** という名前を付けます
   - **「Metrics」**に、**net.http.request.time** と入力します。
   - スコープ: **「Override Dashboard Scope」**をクリックし、**「container.image」**>**「is」**> _アプリケーション・イメージ_ を選択します
   - **container.id** によってセグメント化すると、各コンテナーの正味要求時間が表示されます。
   - **「保存」**をクリックします。
1. 新規パネルを追加するには、**正符号**アイコンをクリックし、パネル・タイプとして**「Number(#)」**を選択します
   - **「Metrics」**で **net.http.request.count** と入力し、時間集計を**「Avg」**から**「Sum」**に変更します。
   - スコープ: **「Override Dashboard Scope」**をクリックし、**「container.image」**>**「is」**> _アプリケーション・イメージ_ を選択します
   - **1 時間**前と比較すると、各コンテナーの正味要求数が表示されます。
   - **「保存」**をクリックします。
1. このダッシュボードのスコープを編集するには、以下のようにします。
   - タイトル・パネルの**「Edit Scope」**をクリックします。
   - ドロップダウンで**「Kubernetes.cluster.name」**を選択するか、入力します
   - 表示名を空のままにし、**「is」**を選択します。
   - 値として**「mycluster」**を選択し、**「Save」**をクリックします。

## リソースの削除
{: #remove_resource}

- [「プログラム識別情報」](https://{DomainName}/observe)ページから LogDNA インスタンスと Sysdig インスタンスを削除します。
- ワーカー・ノード、アプリ、およびコンテナーを含むクラスターを削除します。 このアクションは元に戻せません。
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## チュートリアルを発展させる
{: #expand_tutorial}

- [{{site.data.keyword.at_full}} サービス](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started)を使用して、アプリケーションが IBM Cloud サービスとどのように対話するかをトラッキングします。
- ビューに[アラートを追加](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts)します。
- ローカル・ファイルに[ログをエクスポート](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export)します。

## 関連コンテンツ
{:related}
- [Kubernetes クラスターによって使用される取り込み鍵の再設定](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [IBM Cloud Object Storage へのログのアーカイブ](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [Sysdig でのアラートの構成](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [Sysdig UI での通知チャネルの処理](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
