---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# Kubernetes への継続的デプロイメント
{: #continuous-deployment-to-kubernetes}

このチュートリアルでは、{{site.data.keyword.containershort_notm}} で実行されているコンテナー化アプリケーションに対する継続的な統合と デリバリー・パイプラインをセットアップするプロセスについて説明します。ソース管理をセットアップし、コードを作成およびテストして、別のデプロイメント・ステージにデプロイする方法について学習します。次に、統合をセキュリティー・スキャナー、Slack 通知、分析などの他のサービスに追加します。

{:shortdesc}

## 達成目標
{: #objectives}

* 開発および実動 Kubernetes クラスターを作成します。
* スターター・アプリケーションを作成し、ローカルで実行して Git リポジトリーにプッシュします。
* Git リポジトリーに接続するように DevOps デリバリー・パイプラインを構成し、スターター・アプリを作成して開発/実動クラスターにデプロイします。
* アプリを探索して統合し、セキュリティー・スキャナー、Slack 通知、および分析を使用します。

## 使用するサービス
{: #services}

このチュートリアルでは、以下の {{site.data.keyword.Bluemix_notm}} サービスを使用します。

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**注意:** このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

![](images/solution21/Architecture.png)

1. コードをプライベート Git リポジトリーにプッシュします。
2. パイプラインによって Git 内の変更がピックアップされ、コンテナー・イメージが作成されます。
3. コンテナー・イメージが開発 Kubernetes クラスターにデプロイされたレジストリーにアップロードされます。
4. 変更を検証し、実動クラスターにデプロイします。
5. デプロイメント・アクティビティー用に Slack 通知がセットアップされます。


## 始める前に
{: #prereq}

* [{{site.data.keyword.dev_cli_notm}} をインストールします](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) (docker、kubectl、helm、ibmcloud cli、および必要なプラグインをインストールするスクリプト)。
* [{{site.data.keyword.registrylong_notm}} CLI およびレジストリー名前空間をセットアップします](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)。
* [Kubernetes の基礎を理解します](https://kubernetes.io/docs/tutorials/kubernetes-basics/)。

## 開発 Kubernetes クラスターの作成
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} は、Docker と Kubernetes テクノロジーを結合させた強力なツール、直観的なユーザー・エクスペリエンス、標準装備のセキュリティーと分離機能を提供します。これらの機能を使用することで、コンピュート・ホストから成るクラスター内でコンテナー化アプリのデプロイメント、操作、スケーリング、モニタリングを自動化することができます。

このチュートリアルを完了するには、**標準**タイプの**有料**クラスターを選択する必要があります。開発用と実動用の 2 つのクラスターをセットアップする必要があります。
{: shortdesc}

1. 最初の開発 Kubernetes クラスターは、[{{site.data.keyword.Bluemix}} カタログ](https://{DomainName}/containers-kubernetes/launch)から作成します。後で、これらのステップを繰り返して実動クラスターを作成する必要があります。

   使い勝手をよくするために、CPU の数、メモリー、取得するワーカー・ノードの数などの構成の詳細を確認してください。
   {:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. **「クラスター・タイプ」**を選択し、**「クラスターの作成」**をクリックして、Kubernetes クラスターをプロビジョンします。このチュートリアルでは、2 基の **CPU**、4 **GB RAM**、および 1 つの**ワーカー・ノード**を備えた最小限の**マシン・タイプ**で十分です。他のすべてのオプションは、デフォルトのままにしておいてかまいません。
3. **クラスター**と**ワーカー・ノード**の状況を確認し、**「準備完了 (Ready)」**になるまで待ちます。

**注:** ワーカーが「ready」になるまで次に進まないでください。

## スターター・アプリケーションの作成
{: #create_application}

{{site.data.keyword.containershort_notm}} はスターター・アプリケーションのセレクションを提供しており、これらのスターター・アプリケーションは、`ibmcloud dev create` コマンドまたは Web コンソールを使用して作成できます。このチュートリアルでは、Web コンソールを使用します。スターター・アプリケーションを使用すると、必要なすべてのボイラープレート、ビルド、および構成コードを備えたアプリケーション・スターターを生成することで開発時間が大幅に短縮されるため、ビジネス・ロジックのコーディングを速やかに開始できます。

1. [{{site.data.keyword.cloud_notm}} コンソール](https://{DomainName})で、左側のメニュー・オプションを使用して[「Web アプリ」](https://{DomainName}/developer/appservice/dashboard)を選択します。
2. **「Web から開始 (Start from the Web)」**セクションで、**「開始」**ボタンをクリックします。
3. `Node.js Web App with Express.js` タイルを選択し、`Create app` を選択して Node.js スターター・アプリケーションを作成します。
4. **名前**として `mynodestarter` を入力します。次に、**「作成」**をクリックします。

## DevOps デリバリー・パイプラインの構成
{: #create_devops}

1. スターター・アプリケーションが正常に作成されたので、**「アプリのデプロイ」**で**「クラウドにデプロイ」**ボタンをクリックします。
2. Kubernetes クラスター・デプロイメント方式を選択して、前に作成したクラスターを選択し、**「作成」**をクリックします。これにより、ツールチェーンとデリバリー・パイプラインが作成されます。![](images/solution21/BindCluster.png)
3. パイプラインが作成されたら、**「ツールチェーンの表示」**、**「Delivery Pipeline」**の順にクリックしてパイプラインを表示します。 ![](images/solution21/Delivery-pipeline.png)
4. デプロイ・ステージが完了したら、**「ログおよび履歴の表示」**をクリックしてログを表示します。
5. 表示された URL にアクセスして、アプリケーションにアクセスします (`http://worker-public-ip:portnumber/`)。![](images/solution21/Logs.png)
これで、アプリ・サービス UI を使用してスターター・アプリケーションを作成し、アプリケーションをビルドしてクラスターにデプロイするようにパイプラインを構成しました。

## ローカルでのアプリケーションの複製、ビルド、実行
{: #cloneandbuildapp}

このセクションでは、前のセクションで作成したスターター・アプリを使用し、ローカル・マシンに複製して、コードを変更してからローカルでビルドおよび実行します。
{: shortdesc}

### アプリケーションの複製
1. ツールチェーンの概要で、**「コード」**の下にある**「Git」**タイルを選択します。Git リポジトリー・ページにリダイレクトされ、ここでリポジトリーを複製できます。![HelloWorld](images/solution21/DevOps_Toolchain.png)

2. SSH 鍵をまだセットアップしていない場合は、上部の通知バーに説明が表示されます。新しいタブで**「SSH 鍵の追加」**リンクを開いて手順に従います。SSH の代わりに HTTPS を使用する場合は、**「パーソナル・アクセス・トークンの作成 (create a personal access token)」**をクリックして手順に従います。鍵またはトークンを将来参照できるように、必ず保存してください。
3. SSH または HTTPS を選択して git URL をコピーします。ソースをローカル・マシンに複製します。ユーザー名を指定するように求められた場合は、git ユーザー名を指定します。パスワードは、既存の **SSH 鍵**か**パーソナル・アクセス・トークン**を使用するか、前のステップで作成したものを使用します。

   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   {: codeblock}

4. 複製したリポジトリーを任意の IDE で開き、`public/index.html` にナビゲートします。"Congratulations!" を他のものに変更してコードを更新し、ファイルを保存します。

### ローカルでのアプリケーションのビルド
Java ローカル開発用の `mvn` またはノード開発用の `npm` を使用して、通常どおりにアプリケーションをビルドして実行できます。ローカルとクラウドで実行の一貫性を確保するために、Docker イメージをビルドし、コンテナー内のアプリケーションを実行することもできます。Docker イメージをビルドするには、以下の手順に従います。
{: shortdesc}

1. ローカルの Docker エンジンが開始されていることを確認し、以下のコマンドの実行を確認します。
   ```
   docker ps
   ```
   {: codeblock}
2. 複製された生成済みプロジェクト・ディレクトリーにナビゲートします。
   ```
   cd <project name>
   ```
   {: codeblock}
3. アプリケーションをローカルでビルドします。
   ```
   ibmcloud dev build
   ```
   {: codeblock}

   すべてのアプリケーション依存関係がダウンロードされ、Docker イメージ (アプリケーションと必要なすべての環境を含む) がビルドされるので、この実行には数分かかることがあります。

### ローカルでのアプリケーションの実行

1. コンテナーを実行します。
   ```
   ibmcloud dev run
   ```
   {: codeblock}

   これには、前のステップでビルドした Docker イメージを実行するためのローカルの Docker エンジンが使用されます。
2. コンテナーの開始後、http://localhost:3000/ に移動します。
   ![](images/solution21/node_starter_localhost.png)

## Git リポジトリーへのアプリケーションのプッシュ

このセクションでは、変更を Git リポジトリーにコミットします。パイプラインによってコミットがピックアップされ、変更が自動的にクラスターにプッシュされます。
1. 端末ウィンドウで、複製したリポジトリー内にいることを確認します。
2. 追加、コミット、およびプッシュという 3 つの簡単なステップで、変更内容をリポジトリーにプッシュします。
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
   {: codeblock}

3. 先ほど作成したツールチェーンに移動して、**「Delivery Pipeline」**タイルをクリックします。
4. **ビルド**・ステージと**デプロイ**・ステージが表示されることを確認します。
   ![](images/solution21/Delivery-pipeline.png)
5. **デプロイ**・ステージの完了を待ちます。
6. 最終実行の結果の下にあるアプリケーション **url** をクリックして、変更をライブで表示します。

アプリケーションの更新が表示されない場合は、パイプラインのデプロイ・ステージとビルド・ステージのログを確認します。

## 脆弱性アドバイザーを使用したセキュリティー
{: #vulnerability_advisor}

このステップでは、[脆弱性アドバイザー](https://{DomainName}/docs/services/va?topic=va-va_index#va_index)を探索します。脆弱性アドバイザーは、コンテナー・イメージのセキュリティー状況をデプロイメントの前に検査したり、実行中コンテナーの状況を検査したりするために使用します。

1. 先ほど作成したツールチェーンに移動して、**「Delivery Pipeline」**タイルをクリックします。
1. **「ステージの追加」**をクリックし、MyStage を**「検証ステージ (Validate Stage)」**に変更してから「ジョブ」>**「ジョブの追加」**をクリックします。

   1. ジョブ・タイプとして**「テスト」**を選択し、ボックスで**「テスト」**を**「脆弱性アドバイザー」**に変更します。
   1. 「テスター・タイプ」で**「脆弱性アドバイザー」**を選択します。他のすべてのフィールドは自動的に取り込まれます。
      コンテナー・レジストリー名前空間は、このツールチェーンの**ビルド・ステージ**で指定したものと同じにする必要があります。
      {:tip}
   1. **「テスト・スクリプト」**セクションを編集し、最後の行の `SAFE\ to\ deploy` を `NO\ ISSUES` に置き換えます。
   1. ステージを保存します。
1. **「検証ステージ (Validate Stage)」**をドラッグして中央に移動し、**「検証ステージ (Validate Stage)」**の**「実行」** ![](images/solution21/run.png) をクリックします。**検証ステージ**が失敗したことがわかります。

   ![](images/solution21/toolchain.png)

1. **「ログおよび履歴の表示」**をクリックして脆弱性の評価を確認します。ログの最後に、以下のように記録されています。

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   スキャンしたすべてのリポジトリーの詳細な脆弱性の評価は、[こちら](https://{DomainName}/containers-kubernetes/registry/private)を参照してください。
   {:tip}

   脆弱性のスキャンが 3 分を超える場合、ステージは、イメージが*スキャンされなかった* ことを示して失敗する可能性があります。このタイムアウトは、ジョブ・スクリプトを編集し、スキャン結果を待つ反復回数を増やすことによって変更できます。
   {:tip}

1. 修正アクションに従って、脆弱性を修正しましょう。複製したリポジトリーを IDE で開くか「Eclipse Orion Web IDE (Eclipse Orion Web IDE)」タイルを選択して、`Dockerfile` を開き、`EXPOSE 3000` の後に以下のコマンドを追加します。
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. 変更をコミットしてプッシュします。これにより、ツールチェーンがトリガーされ、**検証ステージ**が修正されます。

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## 実動 Kubernetes クラスターの作成

{: #deploytoproduction}

このセクションでは、Kubernetes アプリケーションを開発環境と実稼働環境にそれぞれデプロイしてデプロイメント・パイプラインを完了します。開発環境には自動デプロイメントをセットアップし、実稼働環境には手動デプロイメントをセットアップすると理想的です。その前に、これを実現できる 2 つの方法を検討しましょう。1 つのクラスターを開発環境と実稼働環境の両方に使用できます。ただし、開発用と実動用の 2 つの別個のクラスターを用意することをお勧めします。2 番目のクラスターを実動用にセットアップする方法を検討しましょう。
{: shortdesc}

1. [開発 Kubernetes クラスターの作成](#create_kube_cluster)セクションの手順に従って、新しいクラスターを作成します。このクラスターに `prod-cluster` という名前を付けます。
2. 先ほど作成したツールチェーンに移動して、**「Delivery Pipeline」**タイルをクリックします。
3. **「デプロイ・ステージ」**の名前を `Deploy dev` に変更します。そのためには、「設定」アイコン >**「ステージの構成」**をクリックします。
   ![](images/solution21/deploy_stage.png)
4. **「Deploy dev」**ステージを複製し (「設定」アイコン >「ステージの複製」)、複製したステージに `Deploy prod` という名前を付けます。
5. **「ステージ・トリガー」**を `Run jobs only when this stage is run manually` に変更します。 ![](images/solution21/prod-stage.png)
6. **「ジョブ」**タブで、クラスター名を新しく作成したクラスターに変更し、ステージを**保存**します。
7. ここで、完全なデプロイメント・セットアップが必要になります。開発から実動にデプロイするには、手動で `Deploy prod` ステージを実行して、実動にデプロイする必要があります。 ![](images/solution21/full-deploy.png)

これで、実動クラスターを作成し、更新を実動クラスターに手動でプッシュするようにパイプラインを構成しました。これは、拡張シナリオでの単純化プロセス・ステージであり、パイプラインの一部として単体テストと統合テストを含めます。

## Slack 通知のセットアップ
{: #setup_slack}

1. 戻って[ツールチェーン](https://{DomainName}/devops/toolchains)のリストを表示し、ツールチェーンを選択して**「ツールの追加」**をクリックします。
2. 検索ボックスで slack を検索するか、スクロールダウンして **Slack** を表示します。クリックして構成ページを開きます。
    ![](images/solution21/configure_slack.png)
3. **「Slack webhook」**については、この[リンク](https://my.slack.com/services/new/incoming-webhook/)の手順に従います。Slack 資格情報でログインし、既存のチャネル名を指定するか、新しいチャネル名を作成する必要があります。
4. 着信 Webhook 統合が追加されたら、**Webhook URL** をコピーして**「Slack webhook」**の下に貼り付けます。
5. Slack チャネルは、上記の Webhook 統合を作成するときに指定したチャネル名です。
6. **「Slack チーム名」**は、team-name.slack.com の team-name (最初の部分) です。例えば、kube は、kube.slack.com におけるチーム名です。
7. **「統合の作成 (Create Integration)」**をクリックします。 新しいタイルがツールチェーンに追加されます。
    ![](images/solution21/toolchain_slack.png)
8. 以降はツールチェーンが実行されるたびに、構成したチャネルに Slack 通知が表示されます。
    ![](images/solution21/slack_channel.png)

## リソースの削除
{: #removeresources}

このステップでは、リソースをクリーンアップして、上記で作成したものを削除します。

- Git リポジトリーを削除します。
- ツールチェーンを削除します。
- 2 つのクラスターを削除します。
- Slack チャネルを削除します。

## チュートリアルを発展させる
{: #expandTutorial}

より詳しく学習したいですか?ここでは、次に実行可能なことを示します。

- [LogDNA と Sysdig によるログの分析とアプリケーションの正常性のモニター](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)。
- テスト環境の追加および 3 つ目のクラスターへのデプロイ
- [複数のロケーションへの](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)実動クラスターのデプロイ。
- [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) を使用した追加の品質管理と分析によるパイプラインの強化

## 関連コンテンツ
{: #related}

* エンドツーエンドの Kubernetes ソリューション・ガイド、[Kubernetes への VM ベースのアプリの移動](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes)
* IBM Cloud Container Service の[セキュリティー](https://{DomainName}/docs/containers?topic=containers-security#cluster)
* ツールチェーン[統合](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations)
* [LogDNA および Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis) を使用したログの分析とアプリケーション・ヘルスのモニター


