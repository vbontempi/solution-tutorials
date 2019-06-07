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

# 独自の IP アドレスの持ち込み
{: #byoip}

独自の IP アドレスの持ち込み (BYOIP) は、{{site.data.keyword.Bluemix_notm}} でプロビジョンされるインフラストラクチャーに既存のクライアント・ネットワークを接続する場合によくある要件です。その意図は、通常、クライアント・ネットワークのルーティング構成や、クライアントの既存の IP アドレッシング・スキームに基づく単一 IP アドレス・スペースを採用した操作への変更を最小限にすることです。

このチュートリアルでは、{{site.data.keyword.Bluemix_notm}} で使用可能な BYOIP 実装パターンについての簡単な概要と、[セキュア・プライベート・ネットワークを使用してワークロードを分離する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)チュートリアルで説明されているセキュア・エンクロージャーを実現する場合に適切なパターンを識別するためのデシジョン・ツリーを示します。セットアップでは、オンサイト・ネットワーク・チーム、{{site.data.keyword.Bluemix_notm}} テクニカル・サポート、または IBM サービスからの追加入力データが必要になる場合があります。

{:shortdesc}

## 達成目標
{: #objectives}

* BYOIP 実装パターンの理解
* {{site.data.keyword.Bluemix_notm}} の実装パターンの選択

## {{site.data.keyword.Bluemix_notm}} IP アドレッシング

{{site.data.keyword.Bluemix_notm}} では多くのプライベート・アドレス範囲、具体的には 10.0.0.0/8 が使用され、場合によっては、これらの範囲は既存のクライアント・ネットワークと競合することがあります。アドレス競合が存在する場合に、IBM Cloud ネットワークとの相互運用を可能にする BYOIP をサポートする多くのパターンがあります。

-	ネットワーク・アドレス変換
-	GRE (Generic Routing Encapsulation) トンネリング
-	IP 別名を使用した GRE トンネリング
-	仮想オーバーレイ・ネットワーク

パターンの選択は、{{site.data.keyword.Bluemix_notm}} でホストされる予定のアプリケーションによって決まります。パターンの実装がアプリケーションに及ぼす影響、および、クライアント・ネットワークと {{site.data.keyword.Bluemix_notm}} の間のアドレス範囲のオーバーラップの程度という、2 つの重要な側面があります。また、{{site.data.keyword.Bluemix_notm}} に専用プライベート・ネットワーク接続を使用する予定の場合は、さらに考慮が必要です。BYOIP を考慮しているすべてのユーザーに対して、[{{site.data.keyword.BluDirectLink}}
](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link) の資料を読むことが推奨されます。{{site.data.keyword.BluDirectLink}} ユーザーの場合、ここで提供される情報に従って、関連ガイダンスに進んでください。

## 実装パターンの概要
{: #patterns_overview}

1. **NAT**: オンプレミス・クライアント・ルーターでの NAT アドレス変換です。オンプレミス NAT を実行して、プロビジョンされる IaaS サービスに {{site.data.keyword.Bluemix_notm}} によって割り当てられる IP アドレスに、クライアント・アドレッシング・スキームを変換します。  
2. **GRE トンネリング**: {{site.data.keyword.Bluemix_notm}} とオンプレミス・ネットワークの間の GRE トンネル、通常は VPN を介した、ルーティング IP トラフィックによって、アドレッシング・スキームが統合されます。このシナリオについては、[こちらのチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)を参照してください。 

   アドレス・スペース・オーバーラップの可能性に応じた 2 つのサブパターンがあります。
     * アドレス・オーバーラップなし: アドレス範囲のアドレス・オーバーラップおよびネットワーク間の競合リスクがない場合。
     * 部分的なアドレス・オーバーラップ: クライアントと {{site.data.keyword.Bluemix_notm}} IP アドレス・スペースで同じアドレス範囲が使用されており、オーバーラップおよび競合の可能性がある場合。この場合は、{{site.data.keyword.Bluemix_notm}} プライベート・ネットワークでオーバーラップしないクライアント・サブネット・アドレスが選択されます。

3. GRE トンネリング + IP 別名
オンプレミス・ネットワークと {{site.data.keyword.Bluemix_notm}} のサーバーに割り当てられる別名 IP アドレスとの間の GRE トンネルを介した、ルーティング IP トラフィックによって、アドレッシング・スキームが統合されます。この特殊な事例のシナリオについては、[こちらのチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)を参照してください。互換 IP サブネットの追加インターフェースと IP 別名が、{{site.data.keyword.Bluemix_notm}} でプロビジョンされる仮想サーバーおよびベアメタル・サーバーで作成され、VRA での適切なルーティング構成によってサポートされます。

4. 仮想オーバーレイ・ネットワーク
[{{site.data.keyword.Bluemix_notm}} 仮想プライベート・クラウド (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) では、{{site.data.keyword.Bluemix_notm}} の完全な仮想環境での BYOIP がサポートされます。これは、[こちらのチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)で説明されているセキュア・プライベート・ネットワーク・エンクロージャーの代替と見なすことができます。

また、{{site.data.keyword.Bluemix_notm}} ネットワークのレイヤーで仮想オーバーレイ・ネットワークを実装する VMware NSX などのソリューションも考慮してください。仮想オーバーレイのすべての BYOIP アドレスは、{{site.data.keyword.Bluemix_notm}} ネットワーク・アドレス範囲から独立しています。[VMware および {{site.data.keyword.Bluemix_notm}} 入門](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud)を参照してください。

## パターンのデシジョン・ツリー
{: #decision_tree}

適切な実装パターンを判別するために、以下のデシジョン・ツリーを使用できます。 

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

追加のガイダンスを以下に示します。

### アプリケーションで NAT が問題になりますか?
{: #nat_consideration}

以下の 2 つは、NAT が問題になる可能性がある明確な事例です。これらの事例では、NAT を使用しないでください。 

- Microsoft AD ドメイン通信などの一部のアプリケーションや、P2P アプリケーションでは、NAT を使用すると技術的な問題が発生する可能性があります。
- 不明サーバーが {{site.data.keyword.Bluemix_notm}} と通信する必要がある場合、または {{site.data.keyword.Bluemix_notm}} とオンプレミス・サーバーの間で何百もの双方向接続が必要な場合。この場合は、事前にマッピングを識別できないため、すべてのマッピングをクライアントのルーター/NAT テーブルで構成することはできません。


### アドレス・オーバーラップなし
{: #no-overlap}

オンプレミス・ネットワークで、10.0.0.0/8 が使用されていますか? オンプレミスと {{site.data.keyword.Bluemix_notm}} プライベート・ネットワークの間でアドレス・オーバーラップが存在しない場合は、[こちらのチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)で説明されている GRE トンネリングをオンプレミスと IBM Cloud の間で使用でき、NAT 変換の必要がなくなります。このためには、ネットワーク・アドレスの使用についてオンサイト・ネットワーク・チームと検討することが必要になります。 

### 部分的なアドレス・オーバーラップ
{: #partial_overlap}

オンプレミス・ネットワークで 10.0.0.0/8 範囲の一部が使用されている場合、{{site.data.keyword.Bluemix_notm}} ネットワークでオーバーラップしていない使用可能なサブネットはありますか? 既存のネットワーク・アドレスの使用法についてオンサイト・ネットワーク・チームと検討し、{{site.data.keyword.Bluemix_notm}} テクニカル販売部門に連絡を取り、オーバーラップしていない使用可能なネットワークを識別します。 

### IP 別名割り当てが問題になりますか?
{: #ip_alias}

オーバーラップしても支障がないアドレスが存在しない場合は、セキュア・プライベート・ネットワーク・エンクロージャーにデプロイされている仮想サーバーおよびベアメタル・サーバーで IP 別名割り当てを実装できます。IP 別名割り当てによって、各サーバーの 1 つ以上のネットワーク・インターフェースに、複数のサブネット・アドレスが割り当てられます。 

IP 別名割り当てはよく使用される手法ですが、マルチホーム構成および IP 別名構成の下でサーバーおよびアプリケーションが正常に機能するかどうかを判別するために、それらの構成を検討することをお勧めします。  

BYOIP サブネットの動的ルート (BGP など) または静的ルートを作成するために、VRA での追加のルーティング構成が必要です。 

## 関連コンテンツ
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [仮想プライベート・クラウド (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
