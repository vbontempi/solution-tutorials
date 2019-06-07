---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-11"
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

# 耐障害性のあるアプリケーションのための戦略
{: #strategies-for-resilient-applications}

Kubernetes、Cloud Foundry、Cloud Functions、Virtual Servers などのコンピュート・オプションに関係なく、企業はダウン時間を最小限に抑え、最大限の可用性を実現する耐障害性のあるアーキテクチャーの構築を目指しています。このチュートリアルでは、耐障害性のあるソリューションを構築するための IBM Cloud の機能を中心に説明し、その過程で以下の質問に答えます。

- グローバルに使用可能になるようにソリューションを準備するときに検討すべき事項は何ですか?
- 複数地域アプリケーションの提供に役立つコンピュート・オプションはどのように入手すればよいですか?
- アプリケーションまたはサービス成果物を追加の地域にインポートするにはどうすればよいですか?
- 複数の場所でデータベースを複製するにはどうすればよいですか?
- ブロック・ストレージ、ファイル・ストレージ、オブジェクト・ストレージ、データベースのうち、どのバッキング・サービスを使用すべきですか?
- サービス固有の考慮事項はありますか?

## 達成目標
{: #objectives}

* 耐障害性のあるアプリケーションを構築する際に必要となるアーキテクチャーの概念を学びます。
* そのような概念が IBM Cloud のコンピュート・オファリングとサービス・オファリングにどのように対応するかを理解します。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャーと概念

{: #architecture}

耐障害性のあるアーキテクチャーを設計するには、ソリューションの個々の構成単位とそれらの特定の機能を検討する必要があります。 

以下の図は、複数地域セットアップに存在する可能性があるさまざまなコンポーネントを示す複数地域アーキテクチャーです。 ![アーキテクチャー](images/solution39/Architecture.png)

上に示すアーキテクチャー図は、コンピュート・オプションによって異なる場合があります。以降のセクションで、各コンピュート・オプションの固有のアーキテクチャー図を示します。 

### 2 つの地域による災害復旧 

災害復旧を促進するために、広く受け入れられている 2 つのアーキテクチャーである**アクティブ/アクティブ**と**アクティブ/パッシブ**を使用します。各アーキテクチャーには、復旧時の時間と作業に関連した独自のコストと利点があります。

#### アクティブ-アクティブ構成

アクティブ/アクティブのアーキテクチャーでは、両方のロケーションで同一のアクティブ・インスタンスを保有し、その 2 つのロケーション間でトラフィックを分散するロード・バランサーが使用されます。この方法を使用する場合、データのレプリケーションは、両方の地域間でデータをリアルタイムで同期させるように設定する必要があります。

![アクティブ/アクティブ](images/solution39/Active-active.png)

この構成は、アクティブ/パッシブ・アーキテクチャーよりも少ない手動修復で高可用性を実現します。要求は両方のデータ・センターで処理されます。最初のデータ・センター環境で障害が発生した場合は、適切なタイムアウトと再試行ロジックを使用してエッジ・サービス (ロード・バランサー) を構成し、要求を 2 番目のデータ・センターに自動的にルーティングする必要があります。

アクティブ/アクティブのシナリオで**目標復旧時点** (RPO) を検討する場合は、シームレスな要求フローを可能にするために、2 つのアクティブなデータ・センター間でのデータの同期を極めて正確なタイミングで行う必要があります。

#### アクティブ/パッシブ構成

アクティブ/パッシブのアーキテクチャーは、1 つのアクティブな地域と、バックアップとして使用される 2 番目の (パッシブな) 地域を使用します。アクティブ地域で障害が発生した場合、パッシブ地域がアクティブになります。データベースまたはファイル・ストレージがアプリケーションおよびユーザーの現在のニーズに合ったものなるように、手操作による介入が必要になることがあります。 

![アクティブ/パッシブ](images/solution39/Active-passive.png)

要求はアクティブ・サイトで処理されます。故障またはアプリケーション障害が発生した場合、スタンバイ・データ・センターが要求を処理できるようにするために、適用前の作業が実行されます。アクティブなデータ・センターからパッシブなデータ・センターへの切り替えは時間のかかる作業です。**目標復旧時間** (RTO) と**目標復旧時点** (RPO) は両方とも、アクティブ/アクティブ構成と比べて高くなります。

### 3 つの地域による災害復旧

今日のダウン時間をまったく許容しない「常時稼働」サービスの時代では、お客様はすべてのビジネス・サービスに世界中どこからでも 24 時間常にアクセスできることを期待しています。企業にとって費用対効果の高い戦略を策定するには、災害復旧インフラストラクチャーを構築するのではなく、継続的可用性を実現するためのインフラストラクチャーの設計が必要です。

データ・センターを 3 つ使用すると、2 つの場合よりも優れた耐障害性と可用性を得ることができます。また、データ・センター間で負荷をより均等に分散させることで、パフォーマンスを向上させることもできます。企業にデータ・センターが 2 つしかない場合は、このトポロジーの変形として、2 つのアプリケーションを 1 番目のデータ・センターにデプロイし、3 番目のアプリケーションを 2 番目のデータ・センターにデプロイします。または、ビジネス・ロジック層とプレゼンテーション層を 3 アクティブのトポロジーにデプロイし、データ層を 2 アクティブのトポロジーにデプロイすることもできます。

#### アクティブ-アクティブ-アクティブ (3 つのアクティブ) 構成

![](images/solution39/Active-active-active.png)

要求は、3 つのアクティブ・データ・センターのいずれかで実行されているアプリケーションによって処理されます。IBM.com Web サイトのケース・スタディーによると、3 アクティブではクラスターごとに 50% のコンピュート容量、メモリー容量、およびネットワーク容量しか必要になりませんが、2 アクティブではクラスターごとに 100% 必要になります。データ層では、大きなコスト差異が発生します。詳細については、[*Always On: Assess, Design, Implement, and Manage Continuous Availability*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf) を参照してください。

#### アクティブ-アクティブ-パッシブ構成

![](images/solution39/Active-active-passive.png)

このシナリオでは、1 次データ・センターと 2 次データ・センターの 2 つのアクティブ・アプリケーションのいずれかで障害が発生した場合、3 番目のデータ・センターのスタンバイ・アプリケーションがアクティブになります。正常性を回復してお客様の要求を処理するために、2 つのデータ・センターのシナリオで説明した災害復旧手順が実行されます。3 つ目のデータ・センターのスタンバイ・アプリケーションは、ホット・スタンバイ構成またはコールド・スタンバイ構成のいずれかでセットアップできます。

災害復旧について詳しくは、[このガイド](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/)を参照してください。

### 複数地域アーキテクチャー

複数地域アーキテクチャーでは、各地域でアプリケーションの同一コピーが実行されるさまざまな場所にアプリケーションがデプロイされます。 

地域は、アプリ、サービス、およびその他の {{site.data.keyword.cloud_notm}} リソースをデプロイできる特定の地理的な場所です。 [{{site.data.keyword.cloud_notm}} 地域](https://{DomainName}/docs/containers?topic=containers-regions-and-zones)は、1 つ以上のゾーンで構成されます。ゾーンは物理データ・センターであり、サービスとアプリケーションをホストするためのコンピュート・リソース、ネットワーク・リソース、ストレージ・リソース、関連冷却機器および電源機器をホストしています。単一障害点を共有しないようにゾーンは互いに独立しています。

また、複数地域アーキテクチャーでは、地域間でトラフィックを分散するために、[Cloud Internet Services](https://{DomainName}/catalog/services/internet-services) などのグローバル・ロード・バランサーが必要になります。

複数の地域にソリューションをデプロイすることによって、以下の利点が得られます。
- エンド・ユーザーの待ち時間を改善する - 速度が重要になります。バックエンドの起点がエンド・ユーザーに近いほど、ユーザーの体験がより向上し、高速化されます。
- 災害復旧 - アクティブな地域で障害が発生したときに、バックアップ地域が存在するため、迅速にリカバリーできます。
- ビジネス要件 - 場合によっては、数 100 キロメートル離れた異なる地域にデータを保管する必要があります。そのような場合には、複数の地域にデータを保管する必要があります。 

### 地域内の複数ゾーン・アーキテクチャー

複数ゾーン地域のアプリケーションを構築することは、アプリケーションを地域内の複数のゾーンにまたがってデプロイすることを意味し、また、地域が 2 つまたは 3 つ存在する場合もあります。 

複数ゾーン地域アーキテクチャーでは、1 つの地域のゾーン間でトラフィックをローカルに分散するためにローカルのロード・バランサーが必要になります。また、2 番目の地域がセットアップされている場合は、グローバル・ロード・バランサーによってトラフィックが地域間で分散されます。 

地域およびゾーンについて詳しくは、[こちら](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones)を参照してください。

## コンピュート・オプション 

このセクションでは、{{site.data.keyword.cloud_notm}} で使用可能なコンピュート・オプションについて検討します。各コンピュート・オプションについて、アーキテクチャー図とそのアーキテクチャーのデプロイ方法に関するチュートリアルが提供されています。

注: どのコンピュート・オプション・アーキテクチャーにもデータベースなどのサービスは含まれていません。これらのアーキテクチャーは、選択したコンピュート・オプション用の 2 つの地域にアプリをデプロイすることのみに焦点を当てたものです。複数地域のコンピュート・オプションのいずれかの例をデプロイした後で、次の論理ステップとしてデータベースおよびその他のサービスを追加します。このソリューション・チュートリアルの後半のセクションで、[データベース](#databaseservices)および[データベース以外のサービス](#nondatabaseservices)について扱います。

### Cloud Foundry 

Cloud Foundry は、複数地域アーキテクチャーのデプロイメントを実行する機能を提供するとともに、[継続的デリバリー](https://{DomainName}/catalog/services/continuous-delivery)・パイプライン・サービスを使用することによって、アプリケーションを複数の地域にデプロイできるようにします。Cloud Foundry 複数地域のアーキテクチャーは次のようになります。

![CF-アーキテクチャー](images/solution39/CF2-Architecture.png)

同じアプリケーションが複数の地域にデプロイされ、グローバル・ロード・バランサーによって正常に機能している最も近い地域にトラフィックがルーティングされます。[**複数の地域にわたるセキュアな Web アプリケーション**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)・チュートリアルでは、同様のアーキテクチャーのデプロイメント手順について説明しています。

### {{site.data.keyword.cfee_full_notm}}

**{{site.data.keyword.cfee_full_notm}} (CFEE)** は、パブリック Cloud Foundry と同じ機能をすべて提供するとともに、追加機能も備えています。

**{{site.data.keyword.cfee_full_notm}}** を使用して、オンデマンドで複数の分離したエンタープライズ・クラスの Cloud Foundry プラットフォームのインスタンスを生成できます。CFEE のインスタンスは、[{{site.data.keyword.cloud_notm}}](http://ibm.com/cloud)
の自分アカウント内で実行されます。この環境は、[{{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service) 上の分離されたハードウェアにデプロイされます。アクセス制御、キャパシティー管理、変更管理、モニター、サービスなど、環境を完全に制御できます。

{{site.data.keyword.cfee_full_notm}} を使用した複数地域アーキテクチャーは次のとおりです。

![アーキテクチャー](images/solution39/CFEE-Architecture.png)

このアーキテクチャーをデプロイするには、以下を実行する必要があります。 

- 2 つの CFEE インスタンスをセットアップします (地域ごとに 1 つ)。
- サービスを作成して、CFEE アカウントにバインドします。 
- CFEE API エンドポイントをターゲットとするアプリをプッシュします。 
- パブリック Cloud Foundry の場合と同じ手順で、データベース・レプリケーションをセットアップします。 

さらに、ステップ・バイ・ステップ・ガイドの [Deploy Logistics Wizard to Cloud Foundry Enterprise Environment (CFEE)](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md) も確認してください。このガイドでは、マイクロサービス・ベースのアプリケーションを CFEE にデプロイする手順について説明しています。1 つ目の CFEE インスタンスにデプロイしたら、2 番目の地域で同じ手順を繰り返し、[インターネット・サービス](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)を 2 つの CFEE インスタンスの前に接続してトラフィックをロード・バランシングすることができます。 

詳細については、[{{site.data.keyword.cfee_full_notm}} の資料](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about)を参照してください。

### Kubernetes

Kubernetes を使用して、地域内の複数ゾーン・アーキテクチャーを構築できます。これは、アクティブ/アクティブのユース・ケースとなります。{{site.data.keyword.containershort_notm}} を使用してソリューションを実装する場合、ロード・バランシングや負荷の分離などの組み込み機能により、ホスト、ネットワーク、アプリで想定される障害に対する耐障害性を強化できるという利点が得られます。複数のクラスターを作成すれば、1 つのクラスターで障害が発生しても、ユーザーは別のクラスターにデプロイされているアプリにまだアクセスできます。また、異なる地域に複数のクラスターがあるため、ユーザーは最も近いクラスターにアクセスすることができ、ネットワーク待ち時間も短縮されます。耐障害性をさらに高めるために、複数ゾーン・クラスターを選択するオプションもあります。複数ゾーン・クラスターでは、ノードが地域内の複数のゾーンにデプロイされます。 

Kubernetes の複数地域アーキテクチャーは以下のようになります。

![Kubernetes](images/solution39/Kub-Architecture.png)

1. 開発者がアプリケーション用の Docker イメージを構築します。
2. これらのイメージが 2 つの異なる場所にある {{site.data.keyword.registryshort_notm}} にプッシュされます。
3. アプリケーションが両方のロケーションの Kubernetes クラスターにデプロイされます。
4. エンド・ユーザーがアプリケーションにアクセスします。
5. Cloud Internet Services は、アプリケーションへの要求を代行受信して、負荷を複数のクラスター間に分散させるように構成します。さらに、アプリケーションを一般的な脅威から保護するために、DDoS 対策および Web アプリケーション・ファイアウォールを有効にします。オプションで、画像や CSS ファイルなどの資産をキャッシュに入れます。

[**Cloud Internet Services による耐障害性のあるセキュアな複数地域 Kubernetes クラスター**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)・チュートリアルでは、このようなアーキテクチャーをデプロイするためのステップを順を追って説明しています。

### {{site.data.keyword.openwhisk_short}}

{{site.data.keyword.openwhisk_short}} は、{{site.data.keyword.cloud_notm}} の複数の場所で使用できます。耐障害性を高め、ネットワーク待ち時間を短縮するために、アプリケーションはそのバックエンドを複数の場所にデプロイできます。その後、開発者は IBM Cloud Internet Services (CIS) を使用して、正常に機能している最も近いバックエンドにトラフィックを分散する責任を持つ単一のエントリー・ポイントを公開できます。{{site.data.keyword.openwhisk_short}} 複数地域のアーキテクチャーは次のようになります。

 ![Functions のアーキテクチャー](images/solution39/Functions-Architecture.png)

1. ユーザーがアプリケーションにアクセスします。要求が Internet Services 経由で送信されます。
2. Internet Services によって、正常に機能している最も近い API バックエンドにユーザーが転送されます。
3. Certificate Manager によって API に SSL 証明書が提供されます。トラフィックはエンドツーエンドで暗号化されます。
4. API が Cloud Functions によって実装されます。

[**複数の地域にわたるサーバーレス・アプリのデプロイ**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless)・チュートリアルを参照して、このアーキテクチャーのデプロイ方法を確認してください。

### {{site.data.keyword.baremetal_short}} と {{site.data.keyword.virtualmachinesshort}}

{{site.data.keyword.virtualmachinesshort}} と {{site.data.keyword.baremetal_short}} は、複数地域アーキテクチャーを実現する機能を提供します。{{site.data.keyword.cloud_notm}} 上で使用可能な多くの場所にサーバーをプロビジョンできます。

![サーバー・ロケーション](images/solution39/ServersLocation.png)

{{site.data.keyword.virtualmachinesshort}} と {{site.data.keyword.baremetal_short}} を使用してこのようなアーキテクチャーを準備するときは、ファイル・ストレージ、バックアップ、リカバリーおよびデータベースについて検討するとともに、Database as a Service を使用するか、仮想サーバー上にデータベースをインストールするかを選択する必要があります。 

以下のアーキテクチャーは、一方の地域がアクティブで、もう一方の地域がパッシブであるアクティブ/パッシブのアーキテクチャーで、{{site.data.keyword.virtualmachinesshort}} を使用して複数地域アーキテクチャーをデプロイする方法の例を示しています。 

![VM アーキテクチャー](images/solution39/vm-Architecture2.png)

このアーキテクチャーに必要な構成要素は以下のとおりです。 

1. ユーザーが、IBM Cloud Internet Services (CIS) を介してアプリケーションにアクセスします。
2. CIS がトラフィックをアクティブな場所にルーティングします。
3. 場所内で、ロード・バランサーがトラフィックをサーバーにリダイレクトします。
4. データベースを仮想サーバーにデプロイします。つまり、データベースを構成し、地域間のレプリケーションとバックアップをセットアップします。または、Database as a Service を使用します。これについては、このチュートリアルの後半で説明します。
5. アプリケーション・イメージとファイルを保管するファイル・ストレージ。ファイル・ストレージは、指定された日時にスナップショットを取る機能を提供します。このスナップショットは、別の地域で手動などによる再利用が可能です。 

[**仮想サーバーを使用してスケーラブルな高可用性 Web アプリを作成する**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application)のチュートリアルで、このアーキテクチャーを実装します。

## データベースおよびアプリケーション・ファイル
{: #databaseservices}

{{site.data.keyword.cloud_notm}} では、[Database as a Service](https://{DomainName}/catalog/?category=databases) を複数提供しており、ビジネス・ニーズに応じてリレーショナル・データベースまたはリレーショナル以外のデータベースを選択できます。[Database-as-a-service (DBaaS)](https://www.ibm.com/cloud/learn/what-is-cloud-database) には多くの利点があります。{{site.data.keyword.cloudant}} のような DBaaS を使用すると、複数地域サポートを利用できるため、異なる地域の 2 つのデータベース・サービス間でライブ・レプリケーションを行ったり、バックアップを実行したりできるとともに、拡張性と最大のアップタイムを確保できます。 

**主要機能:** 

- クラウド・プラットフォームで構築され、アクセスされるデータベース・サービスです。
- ユーザーは、専用のハードウェアを購入することなく、企業で利用するデータベースをホストできます。
- ユーザーが管理することも、プロバイダーが管理するサービスとしても利用できます。
- SQL データベースまたは NoSQL データベースをサポートできます。
- Web インターフェースまたはベンダー提供の API からアクセスします。

**複数地域アーキテクチャーの準備:**

- データベース・サービスの耐障害性オプションはどのようなものですか?
- 地域にまたがる複数のデータベース・サービス間でレプリケーションはどのように処理されますか?
- データはどのようにバックアップされますか?
- 各地域の災害復旧方法はどのようなものですか?

### {{site.data.keyword.cloudant}}

{{site.data.keyword.cloudant}} は、大規模でかつ急成長する Web アプリケーションやモバイル・アプリケーションに特有の大量のワークロードを処理するために最適化された分散データベースです。SLA に裏付けられたフル・マネージド型の {{site.data.keyword.Bluemix_notm}} サービスとして利用できる {{site.data.keyword.cloudant}} は、スループットとストレージを単独で柔軟に拡張します。{{site.data.keyword.cloudant}} は、ダウンロード可能なオンプレミスのインストール済み環境としても利用でき、その API と強力な複製プロトコルは、CouchDB および PouchDB や一般的な Web およびモバイル開発スタック用ライブラリーなどの、オープン・ソース・エコシステムと互換性があります。

{{site.data.keyword.cloudant}} は、複数の場所にまたがる複数のインスタンス間の[レプリケーション](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation)をサポートしています。ソース・データベースで行われたすべての変更がターゲット・データベースで複製されます。継続的に、または「1 回限りの」タスクとして、任意の数のデータベース間でレプリケーションを作成できます。以下の図は、2 つの {{site.data.keyword.cloudant}} インスタンス (各地域に 1 つずつ) を使用する標準的な構成を示しています。

![アクティブ-アクティブ](images/solution39/Active-active.png)

{{site.data.keyword.cloudant}} インスタンス間のレプリケーションを構成するには、[この説明](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery)を参照してください。このサービスには、[データをバックアップしてリストアする](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery)ための説明とツールも用意されています。

### {{site.data.keyword.Db2_on_Cloud_short}}、{{site.data.keyword.dashdbshort_notm}}、および {{site.data.keyword.Db2Hosted_notm}}

{{site.data.keyword.cloud_notm}} では、複数の [Db2 データベース・サービス](https://{DomainName}/catalog/?search=db2h)を提供しています。これらのサービスは次のとおりです。

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2): OLTP のような典型的な運用ワークロード向けのフル・マネージド型クラウド SQL データベース。
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse): 高性能なペタバイト規模の分析ワークロード向けのフル・マネージド型クラウド・データウェアハウス・サービス。SMP と MPP の両方のサービス・プランを提供し、最適化された縦欄データ・ストアとメモリー内処理を使用します。
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted): IBM によってホストされ、ユーザーのデータベース・システムによって管理されます。このサービスは、Db2 にクラウド・インフラストラクチャー上のフル管理アクセスを提供し、独自のインフラストラクチャーを管理することによるコスト、複雑さ、およびリスクを排除します。

以下では、運用ワークロード用の DBaaS として {{site.data.keyword.Db2_on_Cloud_short}} を中心に説明します。これらのワークロードは、このチュートリアルで説明するアプリケーションに典型的なワークロードです。

#### {{site.data.keyword.Db2_on_Cloud_short}} の複数地域サポート

{{site.data.keyword.Db2_on_Cloud_short}} は、[高可用性および災害復旧 (HADR) を実現するためのオプション](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview)を複数提供しています。新規サービスを作成するときに高可用性オプションを選択できます。後で、インスタンス・ダッシュボードを使用して、[Geo レプリケートされた災害復旧ノードを追加](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha)することができます。オフサイトの DR ノード・オプションを使用すると、選択したオフサイト {{site.data.keyword.cloud_notm}} データ・センターのデータベース・ノードにデータをリアルタイムで同期させることができます。

詳細情報については、[高可用性の資料](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha)を参照してください。

#### バックアップとリストア

{{site.data.keyword.Db2_on_Cloud_short}} には、有料プランの日次バックアップが含まれています。通常、バックアップは {{site.data.keyword.cos_short}} を使用して保管されるため、保存データの可用性を高めるために 3 つのデータ・センターを使用します。バックアップの保存期間は 14 日です。これらのバックアップを使用して、ポイント・イン・タイム・リカバリーを実行できます。[バックアップとリストアの資料](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br)に、希望の日時にデータをリストアする方法についての詳しい説明が記載されています。

### {{site.data.keyword.databases-for}}

{{site.data.keyword.databases-for}} は、複数のオープン・ソース・データベース・システムをフル・マネージド型のサービスとして提供します。それらは以下のとおりです。  
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

これらのサービスはすべて、以下のような同じ特性を共有しています。   
* 高可用性を実現するために、クラスターにデプロイされます。詳細については、以下の各サービスについての資料を参照してください。
  - [PostgreSQL](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* 各クラスターは複数のゾーンにまたがって分散されています。
* データはゾーン間で複製されます。
* ユーザーは、インスタンスのストレージ・リソースとメモリー・リソースを増大できます。詳細については、[例えば、{{site.data.keyword.databases-for-redis}} のスケーリングに関する資料](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings)を参照してください。
* バックアップは毎日または要求に応じて行われます。サービスごとに詳細が文書化されます。[例えば、{{site.data.keyword.databases-for-postgresql}} についてのバックアップに関する資料](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups)を参照してください。
* Data at Rest (保存されたデータ)、バックアップ、およびネットワーク・トラフィックは暗号化されます。
* 各[サービスは、{{site.data.keyword.databases-for}} CLI プラグインを使用して管理できます](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in)。


### {{site.data.keyword.cos_full_notm}}

{{site.data.keyword.cos_full_notm}} (COS) は、永続的でセキュアなコスト効果の高いクラウド・ストレージを提供します。{{site.data.keyword.cos_full_notm}} で保管された情報は暗号化され、複数の地理的位置に分散されます。COS インスタンス内にストレージ・バケットを作成するときは、バケットを作成する場所、およびどの耐障害性オプションを使用するかを決定します。

バケットの耐障害性には、以下の 3 つのタイプがあります。
   - 耐障害性が**クロス・リージョン**の場合、データが複数の都市圏の間に分散されます。これは複数地域オプションとみなすことができます。クロス・リージョンのバケットに保管されているコンテンツにアクセスする際に、COS は正常に機能している地域からコンテンツを取得できる特殊なエンドポイントを提供します。
   - 耐障害性が**地域**の場合、データが単一の都市圏全体に分散されます。これは、地域構成内の複数ゾーンとみなすことができます。
   - 耐障害性が**単一データ・センター**の場合、単一のデータ・センター内の複数のアプライアンスの間にデータが分散されます。

{{site.data.keyword.cos_full_notm}} の耐障害性オプションの詳しい説明については、[この資料](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints)を参照してください。

### {{site.data.keyword.filestorage_full_notm}}

{{site.data.keyword.filestorage_full_notm}} は、永続的で高速かつ柔軟な、ネットワーク接続された NFS ベースのファイル・ストレージです。この Network Attached Storage (NAS) 環境では、ファイル共有機能とパフォーマンスを完全に制御できます。 {{site.data.keyword.filestorage_short}}共有は、耐障害性のために経路指定された TCP/IP 接続を介して、最大 64 個の許可デバイスに接続できます。

ファイル・ストレージ機能には、_スナップショット_、_レプリケーション_、_同時アクセス_などがあります。機能の全リストについては、[この資料](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage)を参照してください。

サーバーに接続すると、{{site.data.keyword.filestorage_short}} サービスを使用して、データのバックアップ、およびイメージやビデオなどのアプリケーション・ファイルを容易に保管できます。その後、これらのイメージやファイルは、同じ地域内の複数の異なるサーバーで使用できます。

2 番目の地域を追加する場合、{{site.data.keyword.filestorage_short}} のスナップショット機能を使用して、自動または手動でスナップショットを取り、2 番目のパッシブ地域内で再使用できます。 

スナップショットをリモート・データ・センターの宛先ボリュームに自動的にコピーするようにレプリケーションをスケジュールできます。壊滅的なイベントが発生した場合やデータが破損した場合、コピーをリモート・サイトでリカバリーすることができます。ファイル・ストレージ・スナップショットの詳細については、[こちら](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots)を参照してください。

## データベース以外のサービス
{: #nondatabaseservices}

{{site.data.keyword.cloud_notm}} は、データベース以外の複数の[サービス](https://{DomainName}/catalog)を提供しています。これらのサービスには、IBM サービスとサード・パーティー製のサービスの両方があります。複数地域アーキテクチャーを計画するときは、Watson サービスのようなサービスが複数地域セットアップでどのように機能するかを理解する必要があります。

### {{site.data.keyword.conversationfull}}

[{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about) は、会話型の AI 活用アシスタントの構築で、開発者と技術者以外のユーザーがコラボレーションできるようにするプラットフォームです。

アシスタントは、ビジネス・ニーズに合わせてカスタマイズし、必要な場面で顧客にヘルプを提供するために複数のチャネルに展開できるコグニティブ・ボットです。 アシスタントには 1 つまたは複数のスキルが含まれています。ダイアログ・スキルには、アシスタントでのお客様の支援を可能にする、トレーニング・データとロジックが含まれます。

{{site.data.keyword.conversationshort}} V1 はステートレスであることを知っておくことが重要です。{{site.data.keyword.conversationshort}} では、99.5% のアップタイムを実現しますが、複数の地域にまたがる高可用性アプリケーションでは、このサービスの複数のインスタンスを複数の地域に配置することをお勧めします。 

複数の場所にインスタンスを作成したら、{{site.data.keyword.conversationshort}} ツールを使用して、インテント、エンティティー、およびダイアログを含む既存のワークスペースを 1 つのインスタンスからエクスポートします。その後、このワークスペースを他の場所でインポートします。

## サマリー

| Offering | 耐障害性のオプション|
| -------- | ------------------ |
| Cloud Foundry | <ul><li>複数の場所にアプリケーションをデプロイします</li><li>Cloud Internet Services を使用して複数の場所からの要求を処理します</li><li>Cloud Foundry API を使用して、組織およびスペースを構成し、アプリを複数の場所にプッシュします</li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>複数の場所にアプリケーションをデプロイします</li><li>Cloud Internet Services を使用して複数の場所からの要求を処理します</li><li>Cloud Foundry API を使用して、組織およびスペースを構成し、アプリを複数の場所にプッシュします</li><li>Kubernetes サービス上に構築されます</li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>複数ゾーン・クラスターをサポートする設計による耐障害性</li><li>Cloud Internet Services を使用して複数の場所に分散されたクラスターからの要求を処理します</li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>複数の場所で利用可能です</li><li>Cloud Internet Services を使用して複数の場所からの要求を処理します</li><li>Cloud Functions API を使用して、複数の場所にアクションをデプロイします</li></ul> |
| {{site.data.keyword.baremetal_short}} と {{site.data.keyword.virtualmachinesshort}} | <ul><li>複数の場所にサーバーをプロビジョンします</li><li>同じ場所にある複数のサーバーをローカルのロード・バランサーに接続します</li><li>Cloud Internet Services を使用して複数の場所からの要求を処理します</li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>データベース間の 1 回限りのレプリケーションおよび継続的レプリケーション</li><li>地域内の自動データ冗長性</li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>リアルタイムのデータ同期のために Geo レプリケートされた災害復旧ノードをプロビジョンします</li><li>有料プランによる日次バックアップ</li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>複数ゾーン Kubernetes クラスター上に構築されます</li><li>クロス・リージョン・リード・レプリカ</li><li>日次バックアップと必要に応じたバックアップ</li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>単一データ・センター、地域、およびクロス地域の耐障害性</li><li>API を使用して、複数のストレージ・バケット間でコンテンツを同期化します</li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>スナップショットを使用して、リモート・データ・センターの宛先にコンテンツを自動的に取り込みます</li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>Watson API を使用して、複数の場所の複数のインスタンス間でワークスペース仕様をエクスポートおよびインポートします</li></ul> |

## 関連コンテンツ

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry、複数の地域にわたるセキュアな Web アプリケーション](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [Cloud Functions、複数の地域にわたるサーバーレス・アプリのデプロイ](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes、Cloud Internet Services による耐障害性のあるセキュアな複数地域 Kubernetes クラスター](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [仮想サーバー、高可用性かつスケーラブルな Web アプリの構築](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
