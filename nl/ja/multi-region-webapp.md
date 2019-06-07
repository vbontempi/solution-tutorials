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

# 複数の領域にわたるセキュアな Web アプリケーション
{: #multi-region-webapp}

このチュートリアルでは、[{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) パイプラインを使用して、複数の地域をまたぐ Cloud Foundry アプリケーションの作成、保護、デプロイ、ロード・バランシングを行う方法を説明します。

アプリまたはアプリの一部が停止することがあります。これが現実です。原因としては、コードに問題がある、アプリで使用するリソースが計画保守の影響を受ける、ハードウェア障害のためにアプリをホストしているゾーン、ロケーション、データ・センターの機能が停止する、などがあります。どの状況も発生する可能性があるので、準備しておく必要があります。{{site.data.keyword.Bluemix_notm}} では、アプリケーションを[複数のロケーション](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg)にデプロイして、アプリケーションの耐障害性を高めることができます。また、アプリケーションを複数のロケーションで実行すれば、ユーザー・トラフィックを一番近いロケーションにリダイレクトして待ち時間を短縮することもできます。

## 達成目標

* {{site.data.keyword.contdelivery_short}} を使用して Cloud Foundry アプリケーションを複数のロケーションにデプロイする。
* カスタム・ドメインをアプリケーションにマップする。
* 複数ロケーション対応アプリケーションのグローバル・ロード・バランシングを構成する。
* SSL 証明書をアプリケーションにバインドする。
* アプリケーション・パフォーマンスをモニターする。

## 使用するサービス

このチュートリアルでは、以下のランタイムとサービスを使用します。
* [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry アプリ
* [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) for DevOps
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー

このチュートリアルでは、アプリケーションのコピー 2 つを 2 つの別々のロケーションにデプロイし、その 2 つのコピーがラウンドロビン方式でカスタマー要求を処理するというアクティブ/アクティブのシナリオを使用します。一方のコピーに障害が起きた場合は、DNS 構成によって自動的に正常なロケーションに向かわせられます。

<p style="text-align: center;">

   ![アーキテクチャー](./images/solution1/Architecture.png)
</p>

## Node.js アプリケーションの作成
{: #create}

まずは、Cloud Foundry 環境で実行する Node.js スターター・アプリケーションを作成します。

1. {{site.data.keyword.Bluemix_notm}} コンソールで**[「カタログ」](https://{DomainName}/catalog/)**をクリックします。
2. **「プラットフォーム」**カテゴリーの下の**「Cloud Foundry アプリ」**をクリックして、
**[「{{site.data.keyword.runtime_nodejs_notm}}」](https://{DomainName}/catalog/starters/sdk-for-nodejs)**を選択します。
     ![](images/solution1/SDKforNodejs.png)
3. アプリケーションの**「固有の名前 (unique name)」**を入力します (例: myusername-nodeapp)。これはホスト名にもなります。その後、**「作成」**をクリックします。
4.  アプリケーションが起動したら、**「概要」**ページの**「URL にアクセスする (Visit URL)」**リンクをクリックして、新しいタブでアプリケーションを実際に確認します。

![HelloWorld](images/solution1/HelloWorld.png)

素晴らしいスタートです。 自分自身の Node.js スターター・アプリケーションが {{site.data.keyword.Bluemix_notm}} で稼働しています。

次に、アプリケーションのソース・コードをリポジトリーにプッシュして、変更が自動的にデプロイされるようにします。

## ソース・コントロールおよび {{site.data.keyword.contdelivery_short}} のセットアップ
{: #devops}

この手順では、コードを保管するための git ソース管理リポジトリーをセットアップしてから、コードの変更を自動的にデプロイするパイプラインを作成します。

1. 先ほど作成したアプリケーションの左側のペインで、**「概要」**を選択してスクロールし、**「{{site.data.keyword.contdelivery_short}}」**を見つけます。**「Enable」**をクリックします。

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. デフォルト・オプションをそのままにして、**「作成」**をクリックします。これで、デフォルトの**ツールチェーン**が作成されたはずです。

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. **「コード」**の下で**「Git」**タイルを選択します。そうすると、git リポジトリー・ページが表示されます。
4. SSH 鍵をまだセットアップしていない場合は、上部の通知バーに説明が表示されます。新しいタブで**「SSH 鍵の追加」**リンクを開いて手順に従います。SSH の代わりに HTTPS を使用する場合は、**「パーソナル・アクセス・トークンの作成 (create a personal access token)」**をクリックして手順に従います。鍵またはトークンを将来参照できるように、必ず保存してください。
5. SSH または HTTPS を選択して git URL をコピーします。ソースをローカル・マシンに複製します。
   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   **注:** ユーザー名を求めるプロンプトが出された場合は、git ユーザー名を入力します。パスワードは、既存の **SSH 鍵**か**パーソナル・アクセス・トークン**を使用するか、前のステップで作成したものを使用します。
6. 複製したリポジトリーを任意の IDE で開き、`public/index.html` にナビゲートします。では、コードを更新しましょう。「Hello World」を他の内容に変更してみてください。
7. コマンド `npm install`、`npm build`、`npm start` を順次実行してアプリケーションをローカルで実行し、ブラウザーで `localhost:<port_number>` にアクセスします。**<port_number>** はコンソールに表示されます。
8. 追加、コミット、およびプッシュという 3 つの簡単なステップで、変更内容をリポジトリーにプッシュします。
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
9. 先ほど作成したツールチェーンに移動して、**「Delivery Pipeline」**タイルをクリックします。
10. **ビルド**および**デプロイ**のステージが表示されていることを確認します。 ![HelloWorld](images/solution1/DevOps_Pipeline.png)
11. **デプロイ**・ステージの完了を待ちます。
12. 最終実行の結果の下にあるアプリケーション **url** をクリックして、変更をライブで表示します。

アプリケーションの変更を続け、定期的に変更内容を git リポジトリーにコミットします。アプリケーションの更新が表示されない場合は、パイプラインのデプロイ・ステージおよびビルド・ステージのログを確認します。

## 別のロケーションへのデプロイ
{: #deploy_another_region}

次は、同じアプリケーションを別の {{site.data.keyword.Bluemix_notm}} ロケーションにデプロイします。同じツールチェーンを使用できます。ただし、別のロケーションへのアプリケーションのデプロイを処理するために、別のデプロイ・ステージを追加します。

1. アプリケーションの**「概要」**にナビゲートし、スクロールして**「ツールチェーンの表示」**を見つけます。
2. Deliver で**「Delivery Pipeline」**を選択します。
3. **デプロイ**・ステージの**歯車アイコン**をクリックし、**「ステージの複製」**を選択します。
   ![HelloWorld](images/solution1/CloneStage.png)
4. ステージの名前を「Deploy to UK」に変更して、**「ジョブ」**を選択します。
5. **「IBM Cloud リージョン」**を**「ロンドン - https://api.eu-gb.bluemix.net」**に変更します。**スペース**がない場合は作成します。
6. **「デプロイ・スクリプト」**を `cf push "${CF_APP}" -d eu-gb.mybluemix.net` に変更します。

   ![HelloWorld](images/solution1/DeployToUK.png)
7. **「保存」**をクリックし、**再生ボタン**をクリックして新しいステージを実行します。

## IBM Cloud Internet Services へのカスタム・ドメインの登録

{: #domain_cis}

IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) は、ドメイン・ネーム・システム (DNS)、グローバル・ロード・バランシング (GLB)、Web アプリケーション・ファイアウォール (WAF)、および Web アプリケーションの分散型サービス妨害 (DDoS) 対策を構成して管理するための統一プラットフォームです。これは、IBM Cloud でビジネスを運営するお客様向けに、高速、高性能かつ信頼できる安全なインターネット・サービスを提供し、3 つの主要機能 (セキュリティー、信頼性、パフォーマンス) でワークフローを強化できるようにします。  

現実にアプリケーションをデプロイする場合には、IBM 提供の mybluemix.net ドメインではなく、独自のドメインを使用するのが一般的です。この手順では、カスタム・ドメインを取得した後に、IBM Cloud Internet Services から提供される DNS サーバーを使用できます。

1. [http://godaddy.com](http://godaddy.com) などのレジストラーからドメインを購入します。
2. {{site.data.keyword.Bluemix_notm}} カタログの [Internet Services](https://{DomainName}/catalog/services/internet-services) にナビゲートします。
2. サービス名を入力し、**「作成」**をクリックしてサービスのインスタンスを作成します。
3. サービス・インスタンスがプロビジョンされたら、ドメイン名を設定して**「ドメインの追加」**をクリックします。
4. ネーム・サーバーが割り当てられたら、そのリストされたネーム・サーバーを使用するようにレジストラーまたはドメイン・ネーム・プロバイダーを構成します。
5. レジストラーまたは DNS プロバイダーを構成してから、その変更が反映されるまでに最大で 24 時間かかる場合があります。「概要」ページのドメインの状況が*「保留中」*から*「アクティブ」*に変わったら、`dig <your_domain_name> ns` コマンドを使用して、IBM Cloud ネーム・サーバーが有効になったことを確認できます。
{:tip}

## グローバル・ロード・バランシングをアプリケーションに追加します。

{: #add_glb}

このセクションでは、IBM Cloud Internet Services のグローバル・ロード・バランサー (GLB) を使用して、複数のロケーションのトラフィックを管理します。GLB では、複数のオリジンへのトラフィック分散を可能にするオリジン・プールを使用します。

### GLB を作成する前に、GLB のヘルス・チェックを作成します。

1. Cloud Internet Services アプリケーションで、**「信頼性」**>**「グローバル・ロード・バランサー」**にナビゲートし、ページの下部にある**「ヘルス・チェックの作成」**をクリックします。
2. モニターするパス (`/` など) を入力して、タイプ (HTTP または HTTPS) を選択します。通常は、専用のヘルス・エンドポイントを作成できます。**「1 インスタンスをプロビジョン (Provision 1 Instance)」**をクリックします。
   ![ヘルス・チェック](images/solution1/health_check.png)

### その後、2 つのオリジンで構成されるオリジン・プールを作成します。

1. **「プールの作成」**をクリックします。
2. プールの名前を入力し、先ほど作成したヘルス・チェックを選択し、node.js アプリケーションのロケーションに近い領域を選択します。
3. 1 つ目のオリジンの名前と、ダラスのアプリケーション `<your_app>.mybluemix.net` のホスト名を入力します。
4. 同様に、もう 1 つのオリジンを追加します。オリジンのアドレスにはロンドンのアプリケーション `<your_app>.eu-gb.mybluemix.net` を指定します。
5. **「1 インスタンスをプロビジョン (Provision 1 Instance)」**をクリックします。
   ![オリジン・プール](images/solution1/origin_pool.png)

### グローバル・ロード・バランサー (GLB) の作成 

1. **「ロード・バランサーの作成」**をクリックします。
2. グローバル・ロード・バランサーの名前を入力します。この名前は、ロケーションに関係なく、ユニバーサル・アプリケーション URL (`http://<glb_name>.<your_domain_name>`) の一部ともなります。
3. **「プールの追加」**をクリックし、先ほど作成したオリジン・プールを選択します。
4. **「1 インスタンスをプロビジョン (Provision 1 Instance)」**をクリックします。
   ![グローバル・ロード・バランサー](images/solution1/load_balancer.png)

この段階で、GLB は構成されていますが、Cloud Foundry アプリケーションは、構成された GLB ドメイン名からの要求に応答する準備がまだできていません。構成を完了するには、カスタム・ドメインを使用する経路を指定してアプリケーションを更新します。

## アプリケーションへのカスタム・ドメインおよび経路の構成

{: #add_domain}

この手順では、アプリケーションを実行する {{site.data.keyword.Bluemix_notm}} のロケーションのセキュア・エンドポイントに、カスタム・ドメイン名を対応付けます。

1. メニュー・バーで、**「管理」**をクリックしてから**「アカウント」**をクリックします ([アカウント](https://{DomainName}/account))。
2. アカウント・ページで、アプリケーション**「Cloud Foundry の組織」**にナビゲートして、「アクション」列から**「ドメイン」**を選択します。
3. **「ドメインの追加」**をクリックして、レジストラーから取得したカスタム・ドメイン名を入力します。
4. 正しいロケーションを選択して、**「保存」**をクリックします。
5. 同様に、カスタム・ドメイン名を追加します。
6. {{site.data.keyword.Bluemix_notm}} の[「リソース・リスト」](https://{DomainName}/resources)に戻り、**「Cloud Foundry アプリ」**にナビゲートして、
ダラスのアプリケーションをクリックし、**「経路」**>**「経路の編集」**とクリックして、**「経路の追加」**をクリックします。
   ![経路の追加](images/solution1/ApplicationRoutes.png)
7. 前に構成した GLB ホスト名を**「ホストの入力 (オプション)」**フィールドに入力し、先ほど追加したカスタム・ドメインを選択します。**「保存」**をクリックします。
8. 同様に、ロンドンのアプリケーションのドメインおよび経路を構成します。

この時点で、URL `<glb_name>.<your_domain_name>` を使用してアプリケーションにアクセスできます。グローバル・ロード・バランサーが、複数ロケーション対応アプリケーションのトラフィックを自動的に分散させます。ロンドンのアプリケーションはアクティブにしたままダラスのアプリケーションを停止し、グローバル・ロード・バランサーを介してアプリケーションにアクセスすれば、これを検証できます。

現時点でこれは成功しますが、前の手順で継続的デリバリーを構成したので、別のビルドがトリガーされると構成が上書きされる可能性があります。これらの変更内容を持続させるには、ツールチェーンに戻って以下のように *manifest.yml* ファイルを変更します。

1. {{site.data.keyword.Bluemix_notm}} の[「リソース・リスト」](https://{DomainName}/resources)で**「Cloud Foundry アプリ」**にナビゲートし、ダラスのアプリケーションをクリックし、アプリケーションの**「概要」**にナビゲートし、スクロールして**「ツールチェーンの表示」**を見つけます。
2. 「コード」の下で Git タイルを選択します。
3. *manifest.yml* を選択します。
4. **「編集」**をクリックしてカスタム経路を追加します。元のドメインとホストの構成を`経路`のみに置き換えます。

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <glb_name>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. 変更内容をコミットして、両方のロケーションのビルドが成功したことを確認します。  

## 代替方法: カスタム・ドメインを IBM Cloud システム・ドメインにマップする

複数ロケーション対応アプリケーションのフロントでグローバル・ロード・バランサーを使用したくはないけれども、アプリケーションを実行する {{site.data.keyword.Bluemix_notm}} ロケーションのセキュア・エンドポイントにカスタム・ドメイン名を対応付けたいという場合があります。

Cloud Intenet Services アプリケーションでは、以下の手順でアプリケーションの `CNAME` レコードをセットアップできます。

1. Cloud Internet Services アプリケーションで、**「信頼性」**>**「DNS」**にナビゲートします。
2. **「タイプ」**ドロップダウン・リストから**「CNAME」**を選択し、「名前」フィールドにアプリケーションの別名を入力して、「ドメイン名」フィールドにアプリケーション URL を入力します。ダラスのアプリケーション `<your_app>.mybluemix.net` を CNAME `<your_app>` にマップできます。
3. **「レコードの追加」**をクリックします。プロキシーのトグルをオンに切り替えて、アプリケーションのセキュリティーを強化します。
4. 同様に、ロンドンのエンドポイントの `CNAME` レコードを設定します。
   ![CNAME レコード](images/solution1/cnames.png)

`mybluemix.net` 以外のデフォルト・ドメイン (`cf.appdomain.cloud` や `cf.cloud.ibm.com` など) を使用している場合は、[対応するシステム・ドメイン](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain)を使用してください。
{:tip}

異なる DNS プロバイダーを使用している場合、CNAME レコードのセットアップ手順は、使用している DNS プロバイダーによって異なります。例えば、GoDaddy を使用している場合、GoDaddy の[ドメインのヘルプ](https://www.godaddy.com/help/add-a-cname-record-19236)にあるガイダンスに従ってください。

カスタム・ドメインで Cloud Foundry アプリケーションにアクセスできるようにするには、[アプリケーションをデプロイした Cloud Foundry 組織のドメインのリスト](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#updatingapps)に、そのカスタム・ドメインを追加する必要があります。追加したら、経路をアプリケーション・マニフェストに追加できます。

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <your_app>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}

## SSL 証明書のアプリケーションへのバインド
{: #ssl}

1. SSL 証明書を取得します。例えば、https://www.godaddy.com/web-security/ssl-certificate から購入したり https://letsencrypt.org/ で無料版を生成したりできます。
2. アプリケーションの**「概要」**>**「経路」**>**「ドメインの管理」**にナビゲートします。
3. SSL 証明書のアップロード・ボタンをクリックして、証明書をアップロードします。
5. http ではなく https を使用してアプリケーションにアクセスします。

## アプリケーション・パフォーマンスのモニター
{: #monitor}

複数ロケーション対応アプリケーションの正常性を検査してみましょう。

1. アプリケーション・ダッシュボードで、**「モニタリング」**を選択します。
2. **「すべてのテストの表示」**をクリックします。
   ![](images/solution1/alert_frequency.png)

Availability Monitoring は、世界中のロケーションから合成テストを絶えず実行し、ユーザーが影響を受ける前に、パフォーマンスの問題を積極的に検出して解決しています。アプリケーションのカスタム経路を構成した場合は、アプリケーションにカスタム・ドメインを使用してアクセスするようにテスト定義を変更してください。

## リソースの削除

* ツールチェーンを削除します
* 2 つのロケーションにデプロイした 2 つの Cloud Foundry アプリケーションを削除します
* GLB、オリジン・プール、およびヘルス・チェックを削除します
* DNS 構成を削除します
* Internet Services インスタンスを削除します

## 関連コンテンツ

[Cloudant データベースの追加](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[Auto-Scaling Cloud Foundry applications](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
