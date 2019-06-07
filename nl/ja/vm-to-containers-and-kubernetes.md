---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-15"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# VM ベース・アプリの Kubernetes への移動
{: #vm-to-containers-and-kubernetes}

このチュートリアルでは、{{site.data.keyword.containershort_notm}} を使用して VM ベース・アプリを Kubernetes クラスターに移動するプロセスについて学びます。[{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) は、Docker と Kubernetes テクノロジーを結合させた強力なツール、直観的なユーザー・エクスペリエンス、標準装備のセキュリティーと分離機能を提供します。これらの機能を使用することで、コンピュート・ホストから成るクラスター内でコンテナー化アプリのデプロイメント、操作、スケーリング、モニタリングを自動化することができます。
{: shortdesc}

このチュートリアルには、既存のアプリをコンテナー化して Kubernetes クラスターにデプロイする方法の概念を理解するレッスンも含まれています。VM ベースのアプリをコンテナー化する方法としては、次の選択肢があります。

1. モノリシックな大きなアプリを構成する要素のうち、単体のマイクロサービスとして切り離せるものを見極めます。それらのマイクロサービスをコンテナー化して Kubernetes クラスターにデプロイします。
2. アプリ全体をコンテナー化して Kubernetes クラスターにデプロイします。

アプリのタイプによって、アプリのマイグレーション手順は多岐にわたります。このチュートリアルでは、一般的な手順と、アプリをマイグレーションする前に考えるべき注意点を取り上げます。

## 達成目標
{: #objectives}

- VM ベースのアプリからマイクロサービスを見極める方法と、VM と Kubernetes の間でコンポーネントをマッピングする方法を理解する。
- VM ベースのアプリをコンテナー化する方法を学ぶ。
- {{site.data.keyword.containershort_notm}} でコンテナーを Kubernetes クラスターにデプロイする方法を学ぶ。
- すべての学習内容を実践し、**JPetStore** アプリをクラスターで実行する。

## 使用するサービス
{: #products}

このチュートリアルでは、以下の {{site.data.keyword.Bluemix_notm}} サービスを使用します。

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**注意:** このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{:#architecture}

### VM に基づく従来型のアプリ・アーキテクチャー

仮想マシンに基づく従来型のアプリ・アーキテクチャーの例を以下の図に示します。

<p style="text-align: center;">
![アーキテクチャー図](images/solution30/traditional_architecture.png)

</p>

1. ユーザーが Java アプリのパブリック・エンドポイントに要求を送信します。パブリック・エンドポイントには、ロード・バランサー・サービスがあり、使用可能なアプリ・サーバー・インスタンス間で着信ネットワーク・トラフィックのロード・バランシングを行います。
2. ロード・バランサーが、VM で正常に稼働しているアプリ・サーバー・インスタンスの中から選択したインスタンスに要求を転送します。
3. アプリ・サーバーが、VM で稼働している MySQL データベースにアプリ・データを保管します。アプリ・サーバー・インスタンスは VM で稼働して Java アプリをホストします。アプリ・ファイル (アプリ・コード、構成ファイル、依存関係など) はその VM に保管されます。

### コンテナー化アーキテクチャー

Kubernetes クラスターで実行する最新のコンテナー・アーキテクチャーの例を以下の図に示します。

<p style="text-align: center;">
![アーキテクチャー図](images/solution30/modern_architecture.png)

</p>

1. ユーザーが Java アプリのパブリック・エンドポイントに要求を送信します。パブリック・エンドポイントには、Ingress アプリケーション・ロード・バランサー (ALB) があり、これがクラスター内のアプリ・ポッド間で着信ネットワーク・トラフィックのロード・バランシングを行います。ALB は、公開されているアプリへの着信ネットワーク・トラフィックを許可するためのルールの集合です。
2. ALB がクラスター内の使用可能なアプリ・ポッドの 1 つに要求を転送します。アプリ・ポッドが稼働するワーカー・ノードは、仮想マシンの場合もあれば物理マシンの場合もあります。
3. アプリ・ポッドが永続ボリュームにデータを保管します。永続ボリュームを使用して、アプリ・インスタンス間またはワーカー・ノード間でデータを共有できます。
4. アプリ・ポッドが {{site.data.keyword.Bluemix_notm}} データベース・サービスにデータを保管します。Kubernetes クラスター内で独自のデータベースを実行することも可能ですが、通常は、マネージド・サービスである Database as a Service (DBasS) を使用するほうが簡単に構成できるうえに、バックアップやスケーリングの組み込み機能も利用できます。[IBM Cloud カタログ](https://{DomainName}/catalog/?category=data)で多種多様なデータベースが提供されています。

### VM、コンテナー、Kubernetes

{{site.data.keyword.containershort_notm}} は、Kubernetes クラスターでコンテナー化アプリを実行できるだけでなく、以下のツールや機能も利用できます。

- 直感的なユーザー・エクスペリエンスと強力なツール
- セキュア・アプリケーションの迅速なデリバリーを可能にする標準装備のセキュリティー機能と分離機能
- IBM® Watson™ のコグニティブ機能を含むクラウド・サービス
- ステートレス・アプリケーションとステートフル・ワークロードの両方の専用クラスター・リソースを管理する機能

#### 仮想マシンとコンテナーの比較

**VM** という従来型のアプリはネイティブ・ハードウェア上で稼働します。1 つのアプリが 1 つのコンピュート・ホストの全リソースを使用することは通常ありません。ほとんどの組織では、リソースの無駄を回避するために、1 つのコンピュート・ホストで複数のアプリを実行しようとします。同じアプリのコピーをいくつも実行する場合もありますが、分離を確保する目的で、同じハードウェア上で複数のアプリ・インスタンス (VM) を実行することもあります。このような場合、VM にはオペレーティング・システムのすべてのスタックが含まれるのでサイズが比較的大きくなるうえに、ランタイムとディスクの両方で重複が生じるのであまり効率的ではありません。

**コンテナー**は、環境間でアプリをシームレスに移動できるようにアプリとそのすべての依存関係をパッケージ化するための標準的な手段です。仮想マシンとは異なり、コンテナーはオペレーティング・システムをバンドルしません。 アプリのコード、ランタイム、システム・ツール、ライブラリー、設定のみがコンテナー内にパッケージされます。 コンテナーは、仮想マシンより軽量で移植しやすく、効率的です。

さらに、コンテナーの場合はホスト OS を共有できます。その結果、重複を減らしつつも分離を実現できます。また、コンテナーの場合はシステム・ライブラリーやシステム・バイナリーなどの不要なファイルを除けるので、スペースの節約ならびに攻撃可能面の縮小にもなります。仮想マシンとコンテナーの詳細については、[こちら](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html)を参照してください。

#### Kubernetes のオーケストレーション

[Kubernetes](http://kubernetes.io/) は、ワーカー・ノードのクラスターでコンテナー化アプリのライフサイクルを管理するためのコンテナー・オーケストレーターです。通常、アプリを実行するためには他にも多くのリソースが必要になります。例えば、他のクラウド・サービスに接続したり鍵を保護したりするために、ボリューム、ネットワーク、シークレットが必要になります。Kubernetes で、そうしたリソースをアプリに追加することができます。Kubernetes の鍵となるパラダイムはその宣言モデルにあります。ユーザーが理想の状態を指定すると、Kubernetes がそれに準拠しようとし、理想の状態を維持します。

まずは実際に Kubernetes を操作したいという場合は、こちらの[自己学習用ワークショップ](https://github.com/IBM/kube101/blob/master/workshop/README.md)が役に立つでしょう。また、Kubernetes の[概念](https://kubernetes.io/docs/concepts/)についての資料のページも参照して、Kubernetes の概念を理解してください。

### IBM を利用するメリット

{{site.data.keyword.containerlong_notm}} の Kubernetes クラスターを使用すると、以下のメリットがあります。

- 複数のデータ・センターにクラスターをデプロイできます。
- Ingress とロード・バランサーのネットワーク・オプションを利用できます。
- 動的永続ボリュームを利用できます。
- IBM マネージドの高可用性の Kubernetes マスターを利用できます。

## クラスターのサイズ変更
{: #sizing_clusters}

クラスターのアーキテクチャーを設計するときには、可用性、信頼性、複雑性、リカバリーなどとコストを比較する必要があります。{{site.data.keyword.containerlong_notm}} の Kubernetes クラスターは、アプリの要件に応じてアーキテクチャーを選択できるようになっています。少し計画を立てるだけで、過剰なアーキテクチャーや過剰な投資を避けてクラウド・リソースを最大限に活用できるようになります。見積もりと現実がずれていても、ワーカー・ノードの数やサイズを調整して簡単にクラスターをスケールアップ/スケールダウンできます。

Kubernetes を使用してクラウドで実動アプリを実行するときには、以下の点を検討してください。

1. 特定の 1 つの地理的ロケーションからのトラフィックだけを予想していますか。その場合は、最高のパフォーマンスを得るために、物理的に最も近いロケーションを選択してください。
2. 高可用性のためにクラスターのレプリカはいくつ必要ですか。まずは、開発用、テスト用、実動用の 3 つのクラスターから始めるのが良いでしょう。複数の環境を作成する方法については、[ユーザー、チーム、アプリケーションを編成するためのベスト・プラクティス](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments)のソリューション・ガイドを確認してください。
3. ワーカー・ノードにはどのような[ハードウェア](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes)が必要ですか。仮想マシンですか、それともベア・メタルですか?
4. ワーカー・ノードは何台必要ですか。これはアプリのスケールに大きく左右されます。ノードの数が多いほど、アプリの耐障害性は高くなります。
5. 高可用性のためにいくつのレプリカが必要ですか。レプリカ・クラスターを複数のロケーションにデプロイすれば、アプリの可用性が高まり、1 つのロケーションの障害によるアプリの停止を避けられます。
6. アプリの起動に必要な最小限のリソースはどれくらいですか。アプリの実行に必要なメモリー量と CPU を調べるためにアプリをテストをする必要があるでしょう。そして、アプリをデプロイして起動できるだけの十分なリソースをワーカー・ノードに用意する必要があります。それから、ポッド仕様の一部としてリソースの割り当て量を指定します。Kubernetes はその設定に基づいて、要求に対応できるだけの十分な容量を有するワーカー・ノードを選択 (スケジュール) します。ワーカー・ノードで実行するポッドの数と、その数のポッドに対応するリソース所要量を見積もってください。少なくとも、アプリのポッドを 1 つ実行できる容量のワーカー・ノードにしなければなりません。
7. どのような場合にワーカー・ノード数を増やしますか。クラスターの使用量をモニターし、必要に応じてノード数を増やすことができます。こちらのチュートリアルで[Kubernetesアプリケーションのログの分析と正常性のモニター](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana)の方法を理解してください。
8. 信頼性の高い冗長ストレージが必要ですか。その場合は、NFS ストレージの永続ボリューム請求を作成するか、IBM Cloud データベース・サービスをポッドにバインドします。

上記の点を具体的に考えるために、クラウドで実動 Web アプリケーションを実行するという状況を想定しましょう。トラフィックの負荷としては中程度から高い負荷を想定します。必要になるリソースについて検討してみましょう。

1. 開発用、テスト用、実動用の 3 つのクラスターをセットアップします。
2. 開発クラスターとテスト・クラスターは、RAM と CPU の最小オプションで開始します (例: 1 クラスターあたり CPU 2 つ、RAM 4 GB、ワーカー・ノード 1 台)。
3. 実動クラスターでは、パフォーマンス、高可用性、耐障害性のために、より多くのリソースが必要になるでしょう。専用マシンやベア・メタルにするという選択肢もあります。少なくとも CPU 4 つ、RAM 16 GB、ワーカー・ノード 2 台にします。

## データベースを使用する方法を決定する
{: #database_options}

Kubernetes には、データベースを使用する方法として次の 2 つの選択肢があります。

1. Kubernetes クラスター内で独自のデータベースを実行できます。そのためには、データベースを実行するマイクロサービスを作成する必要があります。MySQL データベースのサンプルを使用する場合は、以下のようにします。
   - MySQL Dockerfile を作成します。[MySQL Dockerfile](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile) のサンプルを参照してください。
   - データベースの資格情報を保管するためにシークレットを使用する必要があります。[こちら](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret)のサンプルを参照してください。
   - Kubernetes にデプロイするデータベースの構成を指定した deployment.yaml ファイルが必要です。[こちら](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml)のサンプルを参照してください。
2. 2 つ目の選択肢は、マネージド・サービスである Database as a Service (DBasS) を使用するという方法です。通常は、このオプションのほうが簡単に構成できるうえに、バックアップやスケーリングの組み込み機能も利用できます。[IBM Cloud カタログ](https://{DomainName}/catalog/?category=data)で多種多様なデータベースが提供されています。この選択肢を使用する場合は、以下のようにします。
   - [IBM クラウド・カタログ](https://{DomainName}/catalog/?category=data)からマネージド・サービスである Database as a Service (DBasS) を作成します。
   - データベースの資格情報をシークレットに保管します。シークレットの詳細については、『Kubernetes シークレットに資格情報を保管する』のセクションを参照してください。
   - アプリケーションで Database as a Service (DBasS) を使用します。

## アプリケーション・ファイルの保管場所を決定する
{: #decide_where_to_store_data}

{{site.data.keyword.containershort_notm}} には、データを保管してポッド間で共有する方法としていくつかの選択肢があります。どの保管方法でも災害時に同じレベルの持続性と可用性を保持できるわけではありません。
{: shortdesc}

### 非永続データ・ストレージ

設計上、コンテナーやポッドの存続期間は短く、予期せぬ障害が起こることがあります。 コンテナーのローカル・ファイル・システムにデータを保管できます。コンテナー内部のデータは、他のコンテナーやポッドと共有できず、コンテナーがクラッシュしたり削除されたりすると失われます。

### アプリの永続データ・ストレージを作成する方法

Kubernetes のネイティブ永続ボリュームを使用して、[NFS ファイル・ストレージ](https://www.ibm.com/cloud/file-storage/details)または[ブロック・ストレージ](https://www.ibm.com/cloud/block-storage)にアプリのデータやコンテナーのデータを永続的に保管できます。
{: shortdesc}

NFS ファイル・ストレージまたはブロック・ストレージをプロビジョンするには、永続ボリューム請求 (PVC) を作成してポッドのストレージを要求する必要があります。PVC では、事前定義されているストレージ・クラスの中から選択できます。ストレージ・クラスごとに、ストレージのタイプ、ストレージのサイズ (ギガバイト)、IOPS、データ保存ポリシー、読み取りと書き込みの権限が規定されています。PVC に基づいて、実際のストレージ・デバイスに相当する永続ボリューム (PV) が {{site.data.keyword.Bluemix_notm}} に動的にプロビジョンされます。PVC をポッドにマウントすると、PV の読み取り/書き込みが可能になります。PV に保管されているデータは、コンテナーがクラッシュしたりポッドのスケジュールが変更されたりした場合でも利用できます。PV の基礎になる NFS ファイル・ストレージとブロック・ストレージは、データの高可用性を実現するために IBM がクラスター化しています。

PVC を作成する場合は、[{{site.data.keyword.containershort_notm}} ストレージの資料](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage)で説明している手順に従ってください。

### 既存のデータを永続ストレージに移動する方法

ローカル・マシンから永続ストレージにデータをコピーするには、PVC をポッドにマウントする必要があります。そうすると、ローカル・マシンからポッドの永続ボリュームにデータをコピーできます。
{: shortdesc}

1. データをコピーするには、まず以下のような構成を作成する必要があります。

   ```bash
   kind: Pod
   apiVersion: v1
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
          claimName: mypvc
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/mnt/data"
             name: task-pv-storage
   ```
   {: codeblock}

2. 次に、ローカル・マシンからポッドにデータをコピーするために、以下のようなコマンドを使用します。
   ```
    kubectl cp <local_filepath>/<filename> <namespace>/<pod>:<pod_filepath>
   ```
   {: pre}

3. クラスター内のポッドからローカル・マシンにデータをコピーします。
   ```
   kubectl cp <namespace>/<pod>:<pod_filepath>/<filename> <local_filepath>/<filename>
   ```
   {: pre}


### 永続ストレージのバックアップをセットアップする

ファイル共有とブロック・ストレージは、クラスターと同じロケーションにプロビジョンされます。ストレージ自体も、高可用性を確保するために、IBM がクラスター化したサーバーでホストされています。ただし、ファイル共有とブロック・ストレージは自動的にはバックアップされないので、ロケーション全体で障害が発生した場合はアクセス不能になる可能性があります。データの損失や損傷を防止するためには、定期バックアップをセットアップし、必要時にバックアップを使用してデータをリストアできるようにします。

詳細については、NFS ファイル・ストレージとブロック・ストレージの[バックアップとリストア](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning)のオプションを参照してください。

## コードを準備する
{: #prepare_code}

### 12 の基本原則を適用する

クラウド・ネイティブなアプリを作成するための方法論として、[Twelve-Factor App](https://12factor.net/) があります。アプリをコンテナー化してクラウドに移行し、Kubernetes でオーケストレーションする場合は、この基本原則を理解して適用することが重要です。{{site.data.keyword.Bluemix_notm}} では、この基本原則の一部が必要になります。
{: shortdesc}

必要になる主な基本原則を以下に示します。

- **コードベース** - すべてのソース・コードと構成ファイルをバージョン管理システム (GIT リポジトリーなど) で管理します。DevOps パイプラインを使用してデプロイメントを行う場合は、これが必要です。
- **ビルド、リリース、実行** - 12-Factor App では、ビルド、リリース、実行という各ステージを厳密に区別します。統合 DevOps デリバリー・パイプラインでこれを自動化すれば、アプリをビルドし、テストしてから、クラスターにデプロイできます。継続的統合/継続的デリバリーのパイプラインをセットアップする方法については、[Kubernetes への継続的デプロイメントのチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)を参照してください。ソース管理、ビルド、テスト、デプロイの各ステージのセットアップ方法や、セキュリティー・スキャナー、通知、分析などの統合機能を追加する方法を学べます。
- **構成** - 構成情報はすべて環境変数に保管します。サービスの資格情報をアプリ・コードの中にハードコーディングしてはいけません。資格情報を保管するには、Kubernetes シークレットを使用します。資格情報の詳細については、以下で説明します。

### Kubernetes シークレットに資格情報を保管する
{: secrets}

アプリ・コードの中に資格情報を保管するのは決して良いやり方ではありません。代わりに、パスワード、OAuth トークン、SSH 鍵などの機密情報を保管するために**[「シークレット」](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)**というものが Kubernetes には用意されています。Kubernetes のシークレットはデフォルトで暗号化されるので、`ポッド`定義や Docker イメージに機密データをそのまま保管するよりも安全で柔軟な保管方法になります。

Kubernetes でシークレットを使用する 1 つの方法を以下に示します。

1. ファイルを作成し、以下のサービスの資格情報をそのファイルの中に保管します。
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api_key>
   }
   ```
   {: codeblock}

2. 以下のコマンドを実行して Kubernetes のシークレットを作成します。コマンドを実行したら、`kubectl get secrets` を使用して、シークレットが作成されたことを確認します。

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## アプリをコンテナー化する
{: #build_docker_images}

アプリをコンテナー化するには、Docker イメージを作成する必要があります。
{: shortdesc}

イメージは [Dockerfile](https://docs.docker.com/engine/reference/builder/#usage) (イメージをビルドするための命令およびコマンドが入ったファイル ) から作成されます。Dockerfile では、命令の中で、別個に保管したビルド成果物 (アプリ、アプリの構成、依存関係など) を参照できます。

既存のアプリの Dockerfile を作成するときには、以下のコマンドを使用できます。

- FROM - コンテナーのランタイムを定義するための親イメージを選択します。
- ADD/COPY - 1 つのディレクトリーの内容をコンテナーにコピーします。
- WORKDIR - コンテナー内に作業ディレクトリーを設定します。
- RUN - アプリの実行時に必要なソフトウェア・パッケージをインストールします。
- EXPOSE - ポートをコンテナー外に公開します。
- ENV NAME - 環境変数を定義します。
- CMD - コンテナーの起動時に実行するコマンドを定義します。

イメージは通常、レジストリーに保管されます。だれでもアクセスできるレジストリー (パブリック・レジストリー) を使用することも、小さなグループのユーザーだけにアクセスを限定したレジストリー (専用レジストリー) をセットアップすることもできます。 Docker Hub などのパブリック・レジストリーは、Docker および Kubernetes の入門として最初のコンテナー化アプリをクラスター内に作成する時に使用できます。 しかし、エンタープライズ・アプリを作成する場合は、不正なユーザーがイメージを使用したり変更したりできないように、{{site.data.keyword.registrylong_notm}} で提供されているようなプライベート・レジストリーを使用してください。

アプリをコンテナー化して {{site.data.keyword.registrylong_notm}} に保管するには、以下のようにします。

1. Dockerfile を作成する必要があります。Dockerfile の例を以下に示します。
   ```
   # Build JPetStore war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # Use WebSphere Liberty base image from the Docker Store
   FROM websphere-liberty:latest

   # Copy war from build stage and server.xml into image
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. Dockerfile を作成したら、次はコンテナー・イメージをビルドして {{site.data.keyword.registrylong_notm}} にプッシュする必要があります。以下のようなコマンドでコンテナーをビルドできます。
   ```
   ibmcloud cr build -t <image_name> <directory_of_Dockerfile>
   ```
   {: pre}

## アプリを Kubernetes クラスターにデプロイする
{: #deploy_to_kubernetes}

コンテナー・イメージをビルドしてクラウドにプッシュしたら、次は Kubernetes クラスターにデプロイする必要があります。そのためには、deployment.yaml ファイルを作成しなければなりません。
{: shortdesc}

### Kubernetes のデプロイメント YAML ファイルを作成する方法

Kubernetes の deployment.yaml ファイルを作成するには、以下のようにします。

1. deployment.yaml ファイルを作成します。こちらに[デプロイメント YAML](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml) ファイルのサンプルがあります。

2. deployment.yaml ファイルで、コンテナーの[リソース割り当て量](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)を定義して、各コンテナーを正常に起動するのに必要な CPU とメモリーの量を指定できます。コンテナーのリソース割り当て量を指定すると、Kubernetes スケジューラーがポッドの配置先にするワーカー・ノードを適切に決定できるようになります。

3. 次に、以下のコマンドを使用して、デプロイメントとサービスを作成し、作成したデプロイメントとサービスを表示できます。

   ```
   kubectl create -f <filepath/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## サマリー
{: #summary}

このチュートリアルでは、以下の内容を学びました。

- VM、コンテナー、Kubernetes の違い。
- 環境の種類 (開発、テスト、実動) 別にクラスターを定義する方法。
- データ・ストレージの使用方法と、永続データ・ストレージの重要性。
- 12 の基本原則をアプリに適用する方法と、資格情報のために Kubernetes のシークレットを使用する方法。
- Docker イメージをビルドして {{site.data.keyword.registrylong_notm}} にプッシュする方法。
- Kubernetes のデプロイメント・ファイルを作成して Docker イメージを Kubernetes にデプロイする方法。

## すべての学習内容を実践して JPetStore アプリをクラスターで実行する
{: #runthejpetstore}

学習内容を実践するために、[デモ](https://github.com/ibm-cloud/ModernizeDemo/)を参考にしながらクラスターで **JPetStore** アプリを実行し、学習した概念を応用します。JPetStore アプリは、Kubernetes で IBM Watson サービスを別個のマイクロサービスとして実行して機能を拡張できるようになっています。

## 関連コンテンツ
{: #related}

- Kubernetes および {{site.data.keyword.containershort_notm}} の[概説](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/)。
- [GitHub](https://github.com/IBM/container-service-getting-started-wt) の {{site.data.keyword.containershort_notm}} ラボ。
- Kubernetes のメインの[資料](http://kubernetes.io/)。
- {{site.data.keyword.containershort_notm}} の [永続ストレージ](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning)。
- ユーザー、チーム、アプリを編成するための[ベスト・プラクティスのソリューション・ガイド](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)。
- [LogDNA と Sysdig によるログの分析とアプリケーションの正常性のモニター](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)。
- Kubernetes で実行するコンテナー化アプリの[継続的統合/継続的デリバリーのパイプライン](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)のセットアップ。
- [複数のロケーションへの](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)実動クラスターのデプロイ。
- [複数のロケーションで複数のクラスター](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations)を使用することで実現する高可用性。
