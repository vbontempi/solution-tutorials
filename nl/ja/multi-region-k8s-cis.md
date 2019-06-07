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

# Cloud Internet Services による耐障害性のあるセキュアな複数地域 Kubernetes クラスター
{: #multi-region-k8s-cis}

耐障害性を考慮してアプリケーションを設計すると、ユーザーがダウン時間を経験する可能性が低くなります。{{site.data.keyword.containershort_notm}} を使用してソリューションを実装する場合は、ロード・バランシングや分離などの組み込み機能を利用して、ホスト、ネットワーク、アプリで想定される障害に対する耐障害性を高めることができます。
複数のクラスターを作成すれば、1 つのクラスターで障害が発生しても、ユーザーは別のクラスターにデプロイされているアプリにまだアクセスできます。また、さまざまなロケーションに複数のクラスターを作成すれば、ユーザーは一番近いクラスターにアクセスできるので、ネットワーク待ち時間も短縮されます。さらに耐障害性を高めるために、複数ゾーンのクラスターを選択する方法もあります。つまり、1 つのロケーションの中の複数のゾーンにノードをデプロイします。

このチュートリアルでは、ドメイン・ネーム・システム (DNS)、グローバル・ロード・バランシング (GLB)、Web アプリケーション・ファイアウォール (WAF)、
インターネット・アプリケーションの分散型サービス妨害 (DDoS) 対策を構成して管理するための統一プラットフォームである Cloud Internet Services (CIS)を Kubernetes クラスターと統合して、
このシナリオをサポートし、安全で耐障害性のあるソリューションを複数のロケーションをまたいで提供する方法に焦点を当てます。

## 達成目標
{: #objectives}

* 1 つのアプリケーションをさまざまなロケーションにある複数の Kubernetes クラスターにデプロイする。
* グローバル・ロード・バランサーを使用して複数のクラスターにトラフィックを分散させる。
* ユーザーを一番近いクラスターに転送する。
* アプリケーションをセキュリティー上の脅威から保護する。
* キャッシュを使用してアプリケーションのパフォーマンスを向上させる。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">
  ![アーキテクチャー](images/solution32-multi-region-k8s-cis/Architecture.png)
</p>

1. 開発者がアプリケーション用の Docker イメージを構築します。
2. イメージがダラスとロンドンの {{site.data.keyword.registryshort_notm}} にプッシュされます。
3. アプリケーションが両方のロケーションの Kubernetes クラスターにデプロイされます。
4. エンド・ユーザーがアプリケーションにアクセスします。
5. Cloud Internet Services は、アプリケーションへの要求を代行受信して、負荷を複数のクラスター間に分散させるように構成します。さらに、アプリケーションを一般的な脅威から保護するために、DDoS 対策および Web アプリケーション・ファイアウォールを有効にします。オプションで、画像や CSS ファイルなどの資産をキャッシュに入れます。

## 始める前に
{: #prereqs}

* Cloud Internet Services を使用するには、カスタム・ドメインを所有している必要があります。Cloud Internet Services のネーム・サーバーを参照するようにそのドメインの DNS を構成するためです。
* [Git をインストールします](https://git-scm.com/)。
* [{{site.data.keyword.Bluemix_notm}} CLI をインストールします](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)。
* [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - docker、kubectl、helm、ibmcloud cli、および必須プラグインをインストールするスクリプト。
* [{{site.data.keyword.registrylong_notm}} CLI およびレジストリー名前空間をセットアップします](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)。
* [Kubernetes の基礎を理解します](https://kubernetes.io/docs/tutorials/kubernetes-basics/)。

## 1 つのロケーションにアプリケーションをデプロイする

このチュートリアルでは、Kubernetes アプリケーションを複数のロケーションにあるクラスターにデプロイします。1 つのロケーション、ダラスから開始して、次の手順をロンドンについても繰り返します。

### Kubernetes クラスターを作成する
{: #create_cluster}

クラスターを作成するには、以下のようにします。
1. [{{site.data.keyword.cloud_notm}} カタログ](https://{DomainName}/containers-kubernetes/catalog/cluster/create)から **{{site.data.keyword.containershort_notm}}** を選択します。
1. **「ロケーション」**を**「ダラス」**に設定します。
1. **「標準」**クラスターを選択します。
1. **「ロケーション」**として 1 つ以上のゾーンを選択します。複数ゾーンのクラスターを作成すると、アプリケーションの耐障害性が向上します。アプリを複数のゾーンに分散させると、ユーザーがダウン時間を経験する可能性が大幅に低くなります。複数ゾーンのクラスターについて詳しくは、[こちら](https://{DomainName}/docs/containers?topic=containers-plan_clusters#ha_clusters)を参照してください。
1. **「マシン・タイプ」**を選択可能な最小値に設定します - このチュートリアルには **2 つの CPU** と **4GB RAM** で十分です。
1. **2** つのワーカー・ノードを使用します。
1. **「クラスター名」**を **my-us-cluster** に設定します。このチュートリアルでは、命名パターンを *`my-<location>-cluster`* に統一します。

クラスターが準備中の間に、アプリケーションを準備します。

### {{site.data.keyword.registryshort_notm}} に名前空間を作成する

1. {{site.data.keyword.Bluemix_notm}} CLI のターゲットをダラスに設定します。
   ```bash
   ibmcloud target -r us-south
   ```
   {: pre}
2. アプリケーションの名前空間を作成します。
   ```bash
   ibmcloud cr namespace-add <your_namespace>
   ```
   {: pre}

このロケーションに名前空間がある場合は、既存のものを再使用することもできます。`ibmcloud cr namespaces` で既存の名前空間をリストできます。
{: tip}

### アプリケーションをビルドする

この手順では、アプリケーションを Docker イメージとしてビルドします。2 つ目のクラスターを構成するときにはこの手順は省略できます。これは単純な HelloWorld アプリです

1. [Hello world アプリ](https://github.com/IBM/container-service-getting-started-wt){:new_windows}のソース・コードをユーザーのホーム・ディレクトリーに複製します。このリポジトリーには、同じアプリのさまざまなバージョンが、Lab で始まる各フォルダーに入っています。
   ```bash
   git clone https://github.com/IBM/container-service-getting-started-wt.git
   ```
   {: pre}
1. `Lab 1` ディレクトリーに移動します。
   ```bash
   cd 'container-service-getting-started-wt/Lab 1'
   ```
   {: pre}
1. `Lab 1` ディレクトリーのアプリ・ファイルを組み込んだ Docker イメージをビルドします。
   ```bash
   docker build --tag multi-region-hello-world:1 .
   ```
   {: pre}

### ロケーションに固有のレジストリーにプッシュするイメージを準備する

イメージにターゲット・レジストリーのタグを付けます。

   ```bash
   docker tag multi-region-hello-world:1 us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### ロケーションに固有のレジストリーにイメージをプッシュする

1. ローカル Docker エンジンがダラスのレジストリーにプッシュできることを確認します。
   ```bash
   ibmcloud cr login
   ```
   {: pre}
2. イメージをプッシュします。
   ```bash
   docker push us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### Kubernetes クラスターにアプリケーションをデプロイする

この段階では、クラスターの準備が完了していなければなりません。[{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/clusters) コンソールでその状況を確認できます。

1. クラスターの構成を取得します。
   ```bash
   ibmcloud cs cluster-config <us-cluster-name>
   ```
   {: pre}
1. 出力をコピーして貼り付け、KUBECONFIG 環境変数を設定します。この変数は `kubectl` で使用されます。
1. 2 つのレプリカを使用してクラスターでアプリケーションを実行します。
   ```bash
   kubectl run hello-world-deployment --image=us.icr.io/<your_US-South_namespace>/hello-world:1 --replicas=2
   ```
   {: pre}
   出力例: `deployment "hello-world-deployment" created`。
1. クラスター内のアプリケーションを公開します。
   ```bash
   kubectl expose deployment/hello-world-deployment --type=ClusterIP --port=80 --name=hello-world-service --target-port=8080
   ```
   {: pre}
   `service "hello-world-service" exposed` のようなメッセージが返されます。

### クラスターに割り当てられたドメイン名と IP アドレスを取得する
{: #CSALB_IP_subdomain}

Kubernetes クラスターを作成すると、Ingress サブドメイン (*my-us-cluster.us-south.containers.appdomain.cloud* など) およびアプリケーション・ロード・バランサーのパブリック IP アドレスが割り当てられます。

1. クラスターの Ingress サブドメインを取得します。
   ```bash
   ibmcloud cs cluster-get <us-cluster-name>
   ```
   {: pre}
   `Ingress Subdomain` の値を見つけます。
1. 後の手順で使用するために、この情報をメモします。

このチュートリアルでは、Ingress サブドメインを使用してグローバル・ロード・バランサーを構成します。このサブドメインの代わりに、アプリケーション・ロード・バランサーのパブリック IP アドレス (`ibmcloud cs albs -cluster <us-cluster-name>`) を使用することもできます。どちらの方法もサポートされています。
{: tip}

## 次は別のロケーションで

これまでの手順を以下のように置き換え、ロンドンについて繰り返します。
* ロケーション名 **Dallas** を **London** に置き換える。
* ロケーションの別名 **us-south** を **eu-gb** に置き換える。
* レジストリー *us.icr.io* を **uk.icr.io** に置き換える。
* クラスター名 *my-us-cluster* を **my-uk-cluster** に置き換える。

## 複数のロケーションのロード・バランシングを構成する

これでアプリケーションが 2 つのクラスターで実行されるようになりましたが、ユーザーが単一のエントリー・ポイントからいずれかのクラスターに透過的にアクセスできるようにする、あるコンポーネントが欠落しています。

このセクションでは、2 つのクラスター間に負荷を分散させるために Cloud Internet Services (CIS) を構成します。CIS は、
アプリケーションを保護しながら Cloud アプリケーションの信頼性とパフォーマンスを確保するためにグローバル・ロード・バランサー (GLB)、
キャッシング、Web アプリケーション・ファイアウォール (WAF)、およびページ・ルールを提供するワンストップ・ショップのサービスです。

グローバル・ロード・バランサーを構成するには、以下を行う必要があります。
* カスタム・ドメインに CIS ネーム・サーバーを参照させる
* Kubernetes クラスターの IP アドレスまたはサブドメイン名を取得する
* アプリケーションの使用可否を検証するヘルス・チェックを構成する
* クラスターを指定したオリジン・プールを定義する

### Cloud Internet Services にカスタム・ドメインを登録する
{: #create_cis_instance}

最初の手順は、CIS のインスタンスを作成し、カスタム・ドメインに CIS ネーム・サーバーを参照させることです。

1. ドメインを所有していない場合は、[godaddy.com](http://godaddy.com) などのレジストラーから購入できます。
2. {{site.data.keyword.Bluemix_notm}} カタログの [Internet Services](https://{DomainName}/catalog/services/internet-services) にナビゲートします。
3. サービス名を設定し、**「作成」**をクリックしてサービスのインスタンスを作成します。
4. サービス・インスタンスがプロビジョンされたら、ドメイン名を設定して**「ドメインの追加」**をクリックします。
5. ネーム・サーバーが割り当てられたら、そのリストされたネーム・サーバーを使用するようにレジストラーまたはドメイン・ネーム・プロバイダーを構成します。
6. レジストラーまたは DNS プロバイダーを構成してから、その変更が反映されるまでに最大で 24 時間かかる場合があります。

   「概要」ページのドメインの状況が*「保留中」*から*「アクティブ」*に変わったら、`dig <your_domain_name> ns` コマンドを使用して、新しいネーム・サーバーが有効になったことを確認できます。
{:tip}

### グローバル・ロード・バランサーのヘルス・チェックを構成する

ヘルス・チェックを使用すると、プールが使用可能かどうかを調べて、正常なプールにトラフィックを転送できます。 これらのチェックは、定期的に HTTP/HTTPS 要求を送信して応答をモニターします。

1. Cloud Internet Services ダッシュボードで、**「信頼性」**>**「グローバル・ロード・バランサー」**にナビゲートし、ページの下部にある**「ヘルス・チェックの作成」**をクリックします。
1. **「パス」**を**「/」**に設定します。
1. **「モニター・タイプ」**を**「HTTP」**に設定します。
1. **「1 インスタンスをプロビジョン (Provision 1 Instance)」**をクリックします。

   独自のアプリケーションを構築するときに、アプリケーション状態を報告する */heathz* などの専用ヘルス・エンドポイントを定義できます。
   {:tip}

### オリジン・プールを定義する

プールとは、GLB に接続するとトラフィックがインテリジェントに転送されるオリジン・サーバーのグループのことです。 英国と米国にクラスターがあるので、ロケーション・ベースのプールを定義し、ユーザー要求の地理的位置に基づいて一番近いクラスターにユーザーをリダイレクトするように CIS を構成できます。

#### ロンドンのクラスター用のプール
1. **「プールの作成」**をクリックします。
2. **「名前」**を**「UK」**に設定します
3. **「ヘルス・チェック」**を直前のセクションで作成したものに設定します
4. **「ヘルス・チェック地域」**を**「西ヨーロッパ」**に設定します
5. **「起点名」**を**「uk-cluster」**に設定します
6. **「起点アドレス」**をロンドンのクラスターの Ingress サブドメイン (*my_uk_cluster.eu-gb.containers.appdomain.cloud* など) に設定します
7. **「1 インスタンスをプロビジョン (Provision 1 Instance)」**をクリックします。

#### ダラスのクラスター用のプール
1. **「プールの作成」**をクリックします。
2. **「名前」**を**「US」**に設定します
3. **「ヘルス・チェック」**を直前のセクションで作成したものに設定します
4. **「ヘルス・チェック地域」**を**「北アメリカ西部」**に設定します
5. **「起点名」**を**「us-cluster」**に設定します
6. **「起点アドレス」**をダラスのクラスターの Ingress サブドメイン (*my_us_cluster.us-south.containers.appdomain.cloud* など) に設定します
7. **「1 インスタンスをプロビジョン (Provision 1 Instance)」**をクリックします。

#### そして、両方のクラスター用のプール
1. **「プールの作成」**をクリックします。
1. **「名前」**を**「All」**に設定します
1. **「ヘルス・チェック」**を直前のセクションで作成したものに設定します
1. **「ヘルス・チェック地域」**を**「北アメリカ東部」**に設定します
1. 次の 2 つのオリジンを追加します。
   1. **「起点名」**を**「us-cluster」**に設定し、**「起点アドレス」**をダラスのクラスターの Ingress サブドメインに設定したもの
   2. **「起点名」**を**「uk-cluster」**に設定し、**「起点アドレス」**をロンドンのクラスターの Ingress サブドメインに設定したもの
2. **「1 インスタンスをプロビジョン (Provision 1 Instance)」**をクリックします。

### グローバル・ロード・バランサーを作成する

オリジン・プールを定義したら、ロード・バランサーの構成を実行できます。

1. **「ロード・バランサーの作成」**をクリックします。
1. **「バランサー・ホスト名」**の下に、グローバル・ロード・バランサーの名前を入力します。この名前は、ロケーションに関係なく、ユニバーサル・アプリケーション URL (`http://<glb_name>.<your_domain_name>`) の一部ともなります。
1. **「デフォルトのオリジン・プール (Default origin pools)」**の下で、**「プールの追加」**をクリックし、**All** という名前のプールを追加します。
1. **「地理的経路の構成 (Configure geo routes)」**のセクションを展開します (オプション)。
   1. **「経路の追加」**をクリックし、**「西ヨーロッパ」**を選択して、**「追加」**をクリックします。
   1. **「プールの追加」**をクリックして、**「UK」**プールを選択します。
   1. 以下の表に示す追加の経路を構成します。
   1. **「1 インスタンスをプロビジョン (Provision 1 Instance)」**をクリックします。

| 地域               | オリジン・プール |
| :---------------:    | :---------: |
|西ヨーロッパ        |     UK      |
|東ヨーロッパ        |     UK      |
|北東アジア        |     UK      |
|東南アジア        |     UK      |
|北アメリカ西部 |     US      |
|北アメリカ東部 |     US      |

この構成により、ヨーロッパとアジアのユーザーはロンドンのクラスターにリダイレクトされ、米国のユーザーはダラスのクラスターにリダイレクトされます。定義されている経路に一致しない要求は、**「デフォルトのオリジン・プール (Default origin pools)」**にリダイレクトされます。

### ロケーションごとに Kubernetes クラスターの Ingress リソースを作成する

グローバル・ロード・バランサーが要求を処理できる状態になりました。すべてのヘルス・チェックが緑色になっているはずです。ただし、グローバル・ロード・バランサーから送られてくる要求に正しく応答するために、Kubernetes クラスターで最後の構成手順を行う必要があります。具体的には、GLB ドメインからの要求を処理する Ingress リソースを定義する必要があります。

1. **glb-ingress.yaml** という名前の Ingress リソース・ファイルを作成します
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: glb-ingress
   spec:
    rules:
      - host: <glb_name>.<your_domain_name>
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-world-service
              servicePort: 80
   ```
    {: pre}
    `<glb_name>.<your_domain_name>` は直前のセクションで定義した URL に置き換えてください。
1. このリソースをロンドンおよびダラスの両方のクラスターにデプロイします。デプロイするときには、対応するロケーションのクラスターの KUBECONFIG 変数を事前に設定します。
   ```bash
   kubectl create -f glb-ingress.yaml
   ```
   {: pre}
   これにより、メッセージ `ingress.extension "glb-ingress" created` が出力されます。

この段階で、複数のロケーションの Kubernetes クラスターに対するグローバル・ロード・バランサーを正常に構成できました。GLB の URL `http://<glb_name>.<your_domain_name>` にアクセスして、アプリケーションを参照できます。ユーザーの場所に応じて一番近いクラスターにリダイレクトされます。CIS がユーザーの IP アドレスを特定の場所にマップできない場合には、デフォルト・プールにリダイレクトされます。

## アプリケーションを保護する
{: #secure_via_CIS}

### Web アプリケーション・ファイアウォールをオンにする

Web アプリケーション・ファイアウォール (WAF) は、Web アプリケーションを ISO レイヤー 7 攻撃から保護します。これは通常、グループ化されたルール・セットと組み合わされます。これらのルール・セットは、不正なトラフィックを除外してアプリケーションの脆弱性を保護するためのものです。

1. Cloud Internet Services ダッシュボードで、**「セキュリティー」**、**「管理」**タブにナビゲートします。
1. **「Web アプリケーション・ファイアウォール」**セクションで、WAF が有効であることを確認します。
1. **「OWASP ルール・セットの表示」**をクリックします。このページで、**「OWASP コア・ルール・セット」**を確認し、ルールを個別に有効/無効にすることができます。ルールを有効にすると、着信要求でルールがトリガーされた場合に、グローバルな脅威のスコアが高くなります。要求に対して**「アクション」**がトリガーされるかどうかは、**「機密度」**設定によって決まります。
   1. デフォルトの OWASP ルール・セットはそのままにしておきます。
   1. **「機密度」**を`「低 (Low)」`に設定します。
   1. **「アクション」**を`「シミュレート」` に設定して、すべてのイベントをログに記録します。
1. **「セキュリティーに戻る」**をクリックします。
1. **「CIS ルール・セットの表示」**をクリックします。このページには、Web サイトをホストするための一般的な技術スタックを基に構築された追加のルールが表示されます。

### パフォーマンスの向上およびサービス拒否攻撃への防御を強化する 
{: #proxy_setting}

分散型サービス拒否 ([DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)) 攻撃とは、ターゲットやその周辺のインフラストラクチャーに大量のインターネット・トラフィックを送り付けて圧倒し、サーバー、サービス、またはネットワークの通常のトラフィックを中断させようとする悪意のある行為です。 CIS はドメインを DDoS から保護する機能を備えています。

1. CIS ダッシュボードで、**「信頼性」**>**「グローバル・ロード・バランサー」**を選択します。
1. 作成した GLB を**「ロード・バランサー」**テーブルで見つけます。
1. **「プロキシー」**列でセキュリティーとパフォーマンスの機能を有効にします。

   ![CIS プロキシーをオンに切り替える](images/solution32-multi-region-k8s-cis/cis-proxy.png)

**これで GLB が保護されるようになりました。**。すぐに享受できるメリットとして、クラスターのオリジン IP アドレスがクライアントに表示されなくなります。着信要求に脅威があることを CIS が検出すると、一般には、アプリケーションにリダイレクトされる前に、ユーザーに次のような画面が表示されます。

   ![検査中 - DDoS 対策](images/solution32-multi-region-k8s-cis/cis-DDoS.png)

さらに、CIS でキャッシュに入れるコンテンツと、キャッシュに入れておく期間も制御できるようになりました。**「パフォーマンス」**>**「キャッシュ」**に移動して、グローバル・キャッシュ・レベルおよびブラウザー有効期限を定義してください。**「ページ・ルール」**によって、グローバルなセキュリティーおよびキャッシュのルールをカスタマイズできます。ページ・ルールにより、特定のドメイン・パスを使用した、きめの細かい構成が可能になります。ページ・ルールを使用する例として、**/assets** の下のすべてのコンテンツをキャッシュに **3 日間**入れるように指定できます。

   ![ページ・ルール](images/solution32-multi-region-k8s-cis/cis-pagerules.png)

## リソースの削除
{:removeresources}

### Kubernetes クラスター・リソースの削除
1. Ingress を削除します。
1. サービスを削除します。
1. デプロイメントを削除します。
1. このチュートリアル用に特別にクラスターを作成した場合は、そのクラスターを削除します。

### CIS リソースの削除
1. GLB を削除します。
1. 起点プールを削除します。
1. ヘルス・チェックを削除します。

## 関連コンテンツ
{:related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Manage your IBM CIS for optimal security](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#best-practice-2-configure-your-security-level-selectively)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
* [{{site.data.keyword.registrylong_notm}} の基礎](https://{DomainName}/docs/services/Registry?topic=registry-registry_overview#registry_planning)
* [アプリの 1 つのインスタンスを Kubernetes クラスターにデプロイする](https://{DomainName}/docs/containers?topic=containers-cs_apps_tutorial#cs_apps_tutorial_lesson1)
* [トラフィックとインターネット・アプリケーションを CIS で保護するためのベスト・プラクティス](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#manage-your-ibm-cis-for-optimal-security)
* [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
