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

# プライベート・ネットワークからのインターネット・アクセスのための NAT の構成
{: #nat-config-private}

現在の Web ベースの IT アプリケーションおよびサービスの世界では、分離した状態で存在するアプリケーションは稀です。開発者はインターネット上のサービスへのアクセスを想定するようになり、そのサービスはオープン・ソースのアプリケーション・コードやアップデートである場合や、REST API 経由でアプリケーション機能を提供する「サード・パーティー」サービスである場合があります。ネットワーク・アドレス変換 (NAT) マスカレードは、プライベート・ネットワークからインターネットでホストされたサービスへのアクセスを保護するために、一般的に使用されるアプローチです。NAT マスカレードでは、プライベート IP アドレスをパブリック・アクセスからシールディングするために、プライベート IP アドレスは多対 1 の関係でアウトバウンド・パブリック・インターフェースの IP アドレスに変換されます。  

このチュートリアルでは、{{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク上でセキュア・サブネットを接続するために、仮想ルーター・アプライアンス (VRA) 上でネットワーク・アドレス変換 (NAT) マスカレードをセットアップする方法を示します。これは、[セキュア・プライベート・ネットワークを使用してワークロードを分離する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)チュートリアルに基づいてソース NAT (SNAT) 構成を追加し、ソース・アドレスが難読化され、ファイアウォール・ルールを使用してアウトバウンド・トラフィックが保護されるようにします。より複雑な NAT 構成は、[VRA 補足資料]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)で参照できます。
{:shortdesc}

## 達成目標
{: #objectives}

-	仮想ルーター・アプライアンス (VRA) 上でのソース・ネットワーク・アドレス変換 (SNAT) のセットアップ
-	インターネット・アクセスのためのファイアウォール・ルールのセットアップ

## 使用するサービス
{: #products}

このチュートリアルでは、以下の {{site.data.keyword.Bluemix_notm}} サービスを使用します。 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

このチュートリアルでは、費用が発生する場合があります。VRA は月額料金プランでのみ利用可能です。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	必要なインターネット・サービスを文書化します。
2.	NAT をセットアップします。
3.	インターネット・ファイアウォール・ゾーンとルールを作成します。

## 始める前に
{: #prereqs}

このチュートリアルでは、[セキュア・プライベート・ネットワークを使用してワークロードを分離する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)チュートリアルによって作成されたセキュア・プライベート・ネットワーク・エンクロージャー内のホストが、パブリック・インターネットのサービスにアクセスできるようにします。そのチュートリアルを先に完了する必要があります。 

## インターネット・サービスの文書化
{: #Document_outbound}

最初のステップとして、パブリック・インターネット上でアクセスするサービスを識別して、アウトバウンド・トラフィックおよびインターネットからの対応するインバウンド・トラフィックを有効にする必要があるポートを文書化します。このポートのリストは、後のステップで、ファイアウォール・ルールのために必要になります。 

この例では http と https の各ポートのみを有効にします。これらによって要件の大部分に対処できるためです。DNS サービスと NTP サービスは、{{site.data.keyword.Bluemix_notm}} プライベート・ネットワークから提供されます。これらと、SMTP (ポート 25) や MySQL (ポート 3306) などの他のサービスが要求される場合は、追加のファイアウォール・ルールが必要になります。以下の 2 つの基本的なポート・ルールがあります。

-	ポート 80 (http)
-	ポート 443 (https)

サード・パーティーのサービスでソース・アドレスのホワイトリスティングがサポートされているかどうかを確認します。サポートされている場合は、サード・パーティーのサービスでサービスへのアクセス制限を構成するために、VRA のパブリック IP アドレスが必要になります。 


## インターネットへの NAT マスカレード 
{: #NAT_Masquerade}

こちらの指示に従って、NAT マスカレードを使用して APP ゾーンのホストの外部インターネット・アクセスを構成します。 

1.	SSH で VRA に接続し、\[edit\] (config) モードに入ります。
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2.	VRA の SNAT ルールを作成します。このとき、前の VRA プロビジョニングのチュートリアルで APP ゾーンのサブネット/VLAN に対して決定されたものと同じ `<Subnet Gateway IP>/<CIDR>` を指定します。 
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Subnet Gateway IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## ファイアウォールの作成
{: #Create_firewalls}

1.	ファイアウォール・ルールの作成 
   ```
   set security firewall name APP-TO-OUTSIDE default-action drop
   set security firewall name APP-TO-OUTSIDE description 'APP traffic to the Internet'
   set security firewall name APP-TO-OUTSIDE default-log

   set security firewall name APP-TO-OUTSIDE rule 90 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 90 action accept
   set security firewall name APP-TO-OUTSIDE rule 90 destination port 80

   set security firewall name APP-TO-OUTSIDE rule 100 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 100 action accept
   set security firewall name APP-TO-OUTSIDE rule 100 destination port 443

   set security firewall name APP-TO-OUTSIDE rule 200 protocol icmp
   set security firewall name APP-TO-OUTSIDE rule 200 icmp type 8
   set security firewall name APP-TO-OUTSIDE rule 200 action accept
   commit
   ```
   {: codeblock}

## ゾーンの作成とルールの適用
{: #Create_zone}

1.	外部インターネットへのアクセスを制御するために、OUTSIDE というゾーンを作成します。
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	インターネットとの間のトラフィックを制御するために、ファイアウォールを割り当てます。
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE
   commit
   save
   ```
   {: codeblock}
3.	APP ゾーンの VSI がインターネット上のサービスにアクセスできるようになったことを確認します。SSH を使用してローカル VSI にログインし、ping と curl を使用して、インターネット上のサイトに対する icmp および tcp アクセスを確認します。  
   ```bash
   ssh root@<VSI Private IP>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## リソースの削除
{:removeresources}
このチュートリアルで作成したリソースを削除する手順は以下のとおりです。 

VRA は月額料金プランで提供されます。取り消しても払い戻しはありません。取り消しは、この VRA が翌月に再び必要とならない場合にのみ行うことをお勧めします。デュアル VRA 高可用性クラスターが必要な場合は、この単一の VRA を[「ゲートウェイの詳細 (Gateway Details)」](https://{DomainName}/classic/network/gatewayappliances)ページでアップグレードできます。
{:tip}  

1. 仮想サーバーまたはベアメタル・サーバーをすべて取り消します
2. VRA を取り消します
3. サポート・チケットにより追加の VLAN をすべて取り消します。 

## 関連する資料
{:related}

-	[VRA ネットワーク・アドレス変換]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[NAT マスカレード]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[VRA の補足資料]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).

