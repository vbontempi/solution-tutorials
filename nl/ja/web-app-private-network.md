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

# セキュア・プライベート・ネットワークで提供する Web アプリケーション
{: #web-app-private-network}

Web アプリケーションのホスティングは、短期的および長期的な使用量の需要に応じてリソースをオンデマンドでスケーリングできるパブリック・クラウドの一般的なデプロイメント・パターンです。パブリック・クラウドで提供される耐障害性および拡張性を補完するには、アプリケーションのワークロードに対するセキュリティーが基本的な前提条件になります。 

このチュートリアルでは、インターネット向けのスケーラブルでセキュアな Web アプリケーションを作成し、仮想ルーター・アプライアンス (VRA)、VLAN、NAT、ファイアウォールで保護したプライベート・ネットワークでホストする方法について説明します。アプリケーションは、ロード・バランサー、2 つの Web アプリケーション・サーバー、および MySQL データベース・サーバーで構成されます。3 つのチュートリアルを組み合わせて、従来のネットワーキングを使用して Web アプリケーションを {{site.data.keyword.Bluemix_notm}} IaaS プラットフォームにセキュアにデプロイする方法を説明します。
{:shortdesc}

## 達成目標
{: #objectives}

- 仮想サーバーを作成して PHP および MySQL をインストールする
- ロード・バランサーをプロビジョンして、アプリケーション・サーバーに要求を分散させる
- 仮想ルーター・アプライアンス (VRA) をデプロイしてセキュアなネットワークを構築する
- VLAN および IP サブネットを定義する 
- ファイアウォール・ルールを使用してネットワークを保護する
- アプリケーション・デプロイメントに送信元ネットワーク・アドレス変換 (SNAT) を使用する

## 使用するサービス
{: #products}

このチュートリアルでは、以下の {{site.data.keyword.Bluemix_notm}} サービスを使用します。 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [ロード・バランサー]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

このチュートリアルでは、費用が発生する場合があります。VRA は月額料金プランでのみ利用可能です。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	セキュアなプライベート・ネットワークを構成します。
2.	アプリケーション・デプロイメントのために NAT アクセスを構成します。
3.	スケーラブルな Web アプリケーションとロード・バランサーをデプロイします。

## 始める前に
{: #prereqs}

このチュートリアルでは、3 つの既存のチュートリアルを使用します。これらは順番に実施します。始める前に、3 つのすべてのチュートリアルを確認する必要があります。

-	[セキュア・プライベート・ネットワークを使用してワークロードを分離する]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[セキュアなネットワークからのインターネット・アクセスのために NAT を構成する]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[仮想サーバーを使用してスケーラブルな高可用性 Web アプリを作成する]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## セキュアなプライベート・ネットワークを構成する
{: #private_network}

独立したセキュアなプライベート・ネットワーク環境は、パブリック・クラウド上の IaaS アプリケーションのセキュリティー・モデルの中核です。ファイアウォール、VLAN、ルーティング、および VPN はすべて、分離されたプライベート環境の構築に必要なコンポーネントです。最初の手順は、Web アプリケーションをデプロイするセキュアなプライベート・ネットワーク・エンクロージャーの作成です。  

- [セキュア・プライベート・ネットワークを使用してワークロードを分離する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

このチュートリアルは変更せずに進めることができます。後の手順で、Nginx Web サーバーおよび MySQL データベースとして、3 つの仮想マシンを APP ゾーンにデプロイします。 

## セキュアなアプリケーション・デプロイメントのために NAT を構成する
{: #nat_config}

オープン・ソースのアプリケーションをインストールするには、ソース・リポジトリーにアクセスするためにセキュアなインターネット・アクセスが必要です。セキュアなプライベート・ネットワーク内のサーバーがパブリック・インターネットから見えないように、送信元 NAT を使用して送信元アドレスを隠し、ファイアウォール・ルールを使用してアウトバウンドのアプリケーション・リポジトリー要求を保護します。インバウンド要求はすべて拒否します。 

- [セキュアなネットワークからのインターネット・アクセスのために NAT を構成する]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

このチュートリアルは変更せずに進めることができます。次の手順では、必要な Nginx および MySQL モジュールにアクセスするために NAT 構成を使用します。  


## スケーラブルな Web アプリケーションとロード・バランサーをデプロイします。
{: #scalable_app}

拡張性と耐障害性を備えた Web アプリケーションをセキュアなプライベート・ネットワークにデプロイする方法を説明するために、Nginx および MySQL を使用する Wordpress インストール環境とロード・バランサーを使用します。 

このチュートリアルでは、このシナリオに沿って、{{site.data.keyword.Bluemix_notm}} ロード・バランサー、2 つの Web アプリケーション・サーバー、および 1 つの MySQL データベース・サーバーを作成します。サーバーはセキュアなプライベート・ネットワーク内の APP ゾーンにデプロイし、ファイアウォールを使用して他のワークロードやパブリック・ネットワークから分離します。 

- [仮想サーバーを使用してスケーラブルな高可用性 Web アプリを作成する]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

このチュートリアルは次の 3 点を変更します。

1.	このチュートリアルで使用する仮想サーバーは、VRA の内側の APP ファイアウォール・ゾーンで保護された VLAN とサブネットにデプロイします。
2. 仮想サーバーを注文する際には、&lt;プライベート VLAN ID&gt; を指定します。仮想サーバーを注文する際に &lt;プライベート VLAN ID&gt; を指定する方法の詳細については、[セキュア・プライベート・ネットワークを使用してワークロードを分離する]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)のチュートリアルの[最初の仮想サーバーを注文する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver)手順を参照してください。また、仮想サーバーにアクセスできるように、チュートリアルの最初のほうでアップロードした SSH 鍵を忘れずに選択してください。 
3. Wordpress ファイルを共有ストレージにコピーするときの rsync のパフォーマンスが低いので、このチュートリアルではファイル・ストレージ・サービスを**使用しない**ことを強くお勧めします。これはチュートリアル全体には影響しません。ファイル・ストレージの作成とマウントのセットアップに関連する手順は、アプリケーション・サーバーとデータベースでは無視できます。代わりに、[アプリケーション・サーバーに PHP アプリケーションをインストールして構成する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application)手順のすべてを、両方のアプリケーション・サーバーで実行する必要があります。[アプリケーション・サーバーに PHP アプリケーションをインストールして構成する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application)手順を実行する前に、まず両方のアプリケーション・サーバーにディレクトリー `/mnt/www/` を作成します。このディレクトリーは、削除することにしたファイル・ストレージ・セクションで作成していたものです。 

   ```sh
   mkdir /mnt/www
   ```

この手順の完了時には、ロード・バランサーが正常な状態で、Wordpress サイトがインターネットでアクセスできる状態になっています。Web アプリケーションを構成する仮想サーバーは、VRA ファイアウォールによってインターネット経由の外部アクセスから保護されており、ロード・バランサー経由でのみアクセスできます。実動環境の場合は、[{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services)で提供される DDoS 対策および Web アプリケーション・ファイアウォール (WAF) も検討する必要があります。


## リソースの削除
{: #removeresources}

このチュートリアルで作成したリソースを削除する手順は以下のとおりです。 

VRA は月額料金プランで提供されます。取り消しても払い戻しはありません。取り消しは、この VRA が翌月に再び必要とならない場合にのみ行うことをお勧めします。{:tip}  

1. 仮想サーバーまたはベアメタル・サーバーをすべて取り消します
2. VRA を取り消します
3. サポート・チケットにより追加の VLAN をすべて取り消します。
4. ロード・バランサーを削除します。
5. ファイル・ストレージ・サービスを削除します。

## チュートリアルを発展させる 

1. このチュートリアルでは、アプリケーション層として 2 つの仮想サーバーのみを最初にプロビジョンしましたが、負荷の増加に対処するために自動的にサーバーを追加することもできます。[オートスケール]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group)を使用すると、ビジネス・アプリケーションをサポートするための仮想サーバーの追加/削除に関連する手動のスケーリング・プロセスを自動化できます。

2. 2 つ目のプライベート VLAN と IP サブネットを VRA に追加し、MySQL データベース・サーバーをホストするための DATA ゾーンを作成してユーザー・データを分離して保護します。APP ゾーンから DATA ゾーンへのインバウンド方向のポート 3306 の MySQL IP トラフィックのみを許可するようにファイアウォール・ルールを構成します。 

