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

# VPN をセキュア・プライベート・ネットワークへ
{: #configuring-IPSEC-VPN}

一般的な要件として、リモート・ネットワーク環境と {{site.data.keyword.Bluemix_notm}} のプライベート・ネットワーク上のサーバーの間のプライベート接続を作成する必要があります。通常、この接続では、{{site.data.keyword.Bluemix_notm}} 上のシステムのハイブリッド・ワークロード、データ転送、プライベート・ワークロード、または管理がサポートされます。サイト間仮想プライベート・ネットワーク (VPN) トンネルは、ネットワーク間の接続を保護するための一般的なアプローチです。 

{{site.data.keyword.Bluemix_notm}} では、パブリック・インターネット上の VPN を使用するか、またはプライベート専用ネットワーク接続を経由して、サイト間のデータ・センター接続のオプションを多数提供しています。 

{{site.data.keyword.Bluemix_notm}} への専用セキュア・ネットワーク・リンクについて詳しくは、
[{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
 を参照してください。パブリック・インターネット上の VPN には帯域幅が保証されない低コストのオプションがあります。 

{{site.data.keyword.Bluemix_notm}} 上のプロビジョン済みサーバーへのパブリック・インターネットを介した接続には、適した VPN オプションが 2 つあります。

-	[IPSEC VPN]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

このチュートリアルでは、クライアント・データ・センターのサブネットを {{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク上のセキュア・サブネットに接続するために、仮想ルーター・アプライアンス (VRA) を使用してサイト間 IPSec VPN をセットアップする方法を示します。 

この例は、[セキュア・プライベート・ネットワークを使用してワークロードを分離する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)チュートリアルに基づいて構築されています。サイト間 IPSec
VPN、GRE トンネル、および静的ルーティングを使用します。動的ルーティング (BGP など) と VTI トンネルを使用する、より複雑な VPN 構成は、[VRA 補足資料](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)で参照できます。
{:shortdesc}

## 達成目標
{: #objectives}

- IPSec VPN の構成パラメーターの文書化
- 仮想ルーター・アプライアンス上の IPSec VPN の構成
- GRE トンネル経由のトラフィックのルーティング

## 使用するサービス
{: #products}

このチュートリアルでは、以下の {{site.data.keyword.Bluemix_notm}} サービスを使用します。 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

このチュートリアルでは、費用が発生する場合があります。VRA は月額料金プランでのみ利用可能です。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	VPN 構成の文書化
2.	VRA 上の IPSec VPN の作成
3.	データ・センターの VPN とトンネルの構成
4.	GRE トンネルの作成
5.	静的 IP 経路の作成
6.	ファイアウォールの構成 

## 始める前に
{: #prereqs}

このチュートリアルでは、[セキュア・プライベート・ネットワークを使用してワークロードを分離する](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)チュートリアルのセキュア・プライベート・エンクロージャーをデータ・センターに接続します。そのチュートリアルを先に完了する必要があります。

## VPN 構成の文書化
{: #Document_VPN}

データ・センターと {{site.data.keyword.Bluemix_notm}} の間に IPSec VPN サイト間リンクを構成するには、多くの構成パラメーター、トンネルのタイプ、および IP ルーティング情報の決定について、オンサイトのネットワーキング・チームと調整する必要があります。VPN 接続が動作するには、パラメーターが正確に対応している必要があります。通常は、オンサイトのネットワーキング・チームが合意された企業標準に合わせて構成を指定し、データ・センターの VPN ゲートウェイの必須 IP アドレスおよびアクセス可能なサブネット・アドレス範囲を提供します。

VPN のセットアップを開始する前に、VPN ゲートウェイの IP アドレスと IP ネットワーク・サブネット範囲を決定し、データ・センターの VPN 構成および {{site.data.keyword.Bluemix_notm}} のセキュア・プライベート・ネットワーク・エンクロージャーで使用できるようにする必要があります。これらを以下の図に示します。セキュア・エンクロージャーの APP ゾーンは、IPSec トンネル経由でクライアント・データ・センターの「DC IP サブネット」内のシステムに接続されます。   

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

VPN を構成する {{site.data.keyword.Bluemix_notm}} ユーザーとクライアント・データ・センターのネットワーキング・チームの間で以下のパラメーターについて合意し、文書化する必要があります。この例では、リモート・トンネルとローカル・トンネルの IP アドレスはそれぞれ 192.168.10.1 と 192.168.10.2 に設定されています。オンサイトのネットワーキング・チームの同意があれば、任意のサブネットを使用できます。 

| 項目  | 説明 |
|:------ |:--- | 
| &lt;ike group name&gt; | IKE グループに付けられた接続用の名前。|
| &lt;ike encryption&gt; | {{site.data.keyword.Bluemix_notm}} とクライアント・データ・センターの間で使用することが合意された IKE 暗号化標準で、通常は *aes256*。|
| &lt;ike hash&gt; | {{site.data.keyword.Bluemix_notm}} とクライアント・データ・センターの間で合意された IKE ハッシュで、通常は *sha1*。|
| &lt;ike-lifetime&gt; | クライアント・データ・センターからの IKE 存続期間で、通常は *3600*。|
| &lt;esp group name&gt; | ESP グループに付けられた接続用の名前。|
| &lt;esp encryption&gt; | {{site.data.keyword.Bluemix_notm}} とクライアント・データ・センターの間で合意された ESP 暗号化標準で、通常は *aes256*。|
| &lt;esp hash&gt; | {{site.data.keyword.Bluemix_notm}} とクライアント・データ・センターの間で合意された ESP ハッシュで、通常は *sha1*。|
| &lt;esp-lifetime&gt; | クライアント・データ・センターからの ESP 存続期間で、通常は *1800*。|
| &lt;DC VPN Public IP&gt;  |クライアント・データ・センターの VPN ゲートウェイのインターネットに接続するパブリック IP アドレス。| 
| &lt;VRA Public IP&gt; | VRA のパブリック IP アドレス。|
| &lt;Remote tunnel IP\/24&gt; | IPSec トンネルのリモート・エンドに割り当てられた IP アドレス。IP クラウドまたはクライアント・データ・センターと競合しない範囲内の IP アドレスのペア。|
| &lt;Local tunnel IP\/24&gt; | IPSec トンネルのローカル・エンドに割り当てられた IP アドレス。|
| &lt;DC Subnet/CIDR&gt; | クライアント・データ・センターおよび CIDR でアクセスされるサブネットの IP アドレス。|
| &lt;App Zone subnet/CIDR&gt; | VRA 作成チュートリアルからの APP ゾーン・サブネットのネットワーク IP アドレスおよび CIDR。| 
| &lt;Shared-Secret&gt; | {{site.data.keyword.Bluemix_notm}} とクライアント・データ・センターの間で使用される共有暗号鍵。|

## VRA 上の IPSec VPN の構成
{: #Configure_VRA_VPN}

{{site.data.keyword.Bluemix_notm}} 上に VPN を作成するためのコマンドおよび変更する必要があるすべての変数を &lt; &gt; で囲んで強調表示したものを以下に示します。変更する必要がある各行では、変更が行単位で識別されます。値は表から
取得されます。 

1. SSH で VRA に接続し、`[edit]` モードに入ります。
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2. Internet Key Exchange (IKE) グループを作成します。
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike encryption>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike hash>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lifetime>
   ```
   {: codeblock}
3. カプセル化セキュリティー・ペイロード (ESP) グループを作成します。
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp encryption>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp hash>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lifetime>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. サイト間接続を定義します。
   ```
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication mode pre-shared-secret
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication pre-shared-secret <Shared-Secret>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  connection-type initiate
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  ike-group <ike group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  local-address <VRA Public IP>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  default-esp-group <esp group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1 protocol gre
   commit
   save
   ```
   {: codeblock}

## データ・センターの VPN とトンネルの構成
{: #Configure_DC_VPN}

1. クライアント・データ・センターのネットワーク・チームは、同じ IPSec VPN 接続を構成しますが、&lt;DC VPN Public IP&gt; と &lt;VRA Public IP&gt; をスワップし、ローカル・トンネルとリモート・トンネルのアドレスを指定し、&lt;DC Subnet/CIDR&gt; パラメーターと &lt;App Zone subnet/CIDR&gt; パラメーターもスワップします。クライアント・データ・センターの特定の構成コマンドは、VPN のベンダーによって異なります。
1. 次に進む前に、DC VPN ゲートウェイのパブリック IP アドレスにインターネットを介してアクセス可能であることを確認します。
   ```
   ping <DC VPN Public IP>
   ```
   {: codeblock}
2. データ・センターの VPN 構成が完了すると、IPSec リンクが自動的に確立されます。リンクが確立され、ステータスが 1 つ以上のアクティブ IPsec トンネルの存在を示していることを確認します。データ・センターで VPN の両端がアクティブ IPsec トンネルを示していることを確認します。
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. リンクが作成されていない場合は、デバッグ・コマンドを使用して、ローカル・アドレスとリモート・アドレスが正しく指定され、他のパラメーターが期待どおりであることを確認します。
   ``` bash
   show vpn debug
   ```
   {: codeblock}

出力に、`peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......` 行が表示されます。この行が存在しない場合、または「CONNECTING」が表示される場合は、VPN 構成にエラーがあります。  

## GRE トンネルの定義 
{: #Define_Tunnel}

1. VRA 編集モードで GRE トンネルを作成します。
   ```
   set interfaces tunnel tun0 address <Local tunnel IP/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <VRA Public IP>
   set interfaces tunnel tun0 remote-ip <DC VPN Public IP>
   commit
   ```
   {: codeblock}
2. トンネルの両端が構成されると、トンネルは自動的に確立されます。VRA コマンド・ラインからトンネルの動作状態を確認します。
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   最初のコマンドは、トンネルの状態とリンクが `u/u` (UP/UP) であることを示します。2 番目のコマンドは、トンネルに関する詳細とトラフィックが送受信されていることを示します。
3. トラフィックがトンネルを通過することを確認します。
   ```bash
   ping <Remote tunnel IP>
   ```
   {: codeblock}
   `show interfaces tunnel tun0` の TX と RX のカウントは、`ping` トラフィックがある間は増加を示します。
4. トラフィックが流れていない場合は、`monitor interface` コマンドを使用して、各インターフェース上で見られるトラフィックを監視できます。インターフェース `tun0` は、トンネル上の内部トラフィックを示します。インターフェース `dp0bond1` は、リモート VPN ゲートウェイとの間のカプセル化されたトラフィック・フローを示します。
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

戻りのトラフィックがない場合は、データ・センターのネットワーキング・チームがリモート・サイトの VPN およびトンネル・インターフェースにおけるトラフィック・フローをモニターして、問題を特定する必要があります。 

## 静的 IP 経路の作成
{: #Define_Routing}

トラフィックをトンネル経由でリモート・サブネットに送信する VRA ルーティングを作成します。

1. VRA 編集モードで静的ルートを作成します。
   ```
   set protocols static route <DC Subnet/CIDR>  next-hop <Remote tunnel IP>
   ```
   {: codeblock}   
2. VRA コマンド・ラインで VRA ルーティング・テーブルを確認します。現時点では、トンネル経由のトラフィックを許可するファイアウォール・ルールが存在しないため、トラフィックは経路を横断しません。ファイアウォール・ルールは、いずれの側で開始されたトラフィックに対しても必要になります。
   ```bash
   show ip route
   ```
   {: codeblock}

## ファイアウォールの構成
{: #Configure_firewall}

1. 許可された icmp トラフィックと tcp ポートのリソース・グループを作成します。 
   ```
   set res group icmp-group icmpgrp type 8
   set res group icmp-group icmpgrp type 11
   set res group icmp-group icmpgrp type 3

   set res group port tcpports port 22
   set res group port tcpports port 80
   set res group port tcpports port 443
   commit
   ```
   {: codeblock}
2. VRA 編集モードで、リモート・サブネットへのトラフィックのファイアウォール・ルールを作成します。
   ```
   set security firewall name APP-TO-TUNNEL default-action drop
   set security firewall name APP-TO-TUNNEL default-log

   set security firewall name APP-TO-TUNNEL rule 100 action accept
   set security firewall name APP-TO-TUNNEL rule 100 protocol tcp
   set security firewall name APP-TO-TUNNEL rule 100 destination port tcpports

   set security firewall name APP-TO-TUNNEL rule 200 protocol icmp
   set security firewall name APP-TO-TUNNEL rule 200 icmp group icmpgrp
   set security firewall name APP-TO-TUNNEL rule 200 action accept
   commit
   ```
   {: codeblock}

   ```
   set security firewall name TUNNEL-TO-APP default-action drop
   set security firewall name TUNNEL-TO-APP default-log

   set security firewall name TUNNEL-TO-APP rule 100 action accept
   set security firewall name TUNNEL-TO-APP rule 100 protocol tcp
   set security firewall name TUNNEL-TO-APP rule 100 destination port tcpports

   set security firewall name TUNNEL-TO-APP rule 200 protocol icmp
   set security firewall name TUNNEL-TO-APP rule 200 icmp group icmpgrp
   set security firewall name TUNNEL-TO-APP rule 200 action accept 
   commit
   ```
   {: codeblock}
2. トンネルのゾーンを作成し、いずれかのゾーンで開始されたトラフィックのファイアウォールを関連付けます。
   ```
   set security zone-policy zone TUNNEL description "GRE Tunnel"
   set security zone-policy zone TUNNEL default-action drop
   set security zone-policy zone TUNNEL interface tun0

   set security zone-policy zone TUNNEL to APP firewall TUNNEL-TO-APP
   set security zone-policy zone APP to TUNNEL firewall APP-TO-TUNNEL
   commit
   save
   ```
   {: codeblock}
3. 両端のファイアウォールとルーティングが正しく構成され、ICMP トラフィックと TCP トラフィックを許可していることを確認するには、まず VRA コマンド・ラインからリモート・サブネットのゲートウェイ・アドレスを ping します。成功した場合は VSI にログインします。
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}
4. VRA コマンド・ラインからの ping が失敗した場合は、トンネル・インターフェースに対する ping 要求への ping 応答があることを確認します。
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
   無応答の場合は、データ・センターのファイアウォール・ルールまたはルーティングに問題があります。モニター出力に応答が表示されるが、ping コマンドがタイムアウトする場合は、ローカル VRA ファイアウォール・ルールの構成を確認します。
5. VSI からの ping が失敗した場合は、VRA ファイアウォール・ルール、VRA 構成のルーティング、または VSI 構成のルーティングに問題があります。前のステップを完了して、要求がデータ・センターに送信され、データ・センターから応答があることを確認します。ローカル VLAN 上のトラフィックをモニターし、ファイアウォール・ログを調べると、問題をルーティングまたはファイアウォールに切り分けるのに役立ちます。
   ```
   monitor interfaces bonding dp0bond0.<VLAN ID>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

これで、セキュア・プライベート・ネットワーク・エンクロージャーからの VPN のセットアップは完了です。このシリーズの他のチュートリアルで、エンクロージャーからパブリック・インターネット上のサービスにアクセスする方法を示します。

## リソースの削除
{:removeresources}

このチュートリアルで作成したリソースを削除する手順は以下のとおりです。

VRA は月額料金プランで提供されます。取り消しても払い戻しはありません。取り消しは、この VRA が翌月に再び必要とならない場合にのみ行うことをお勧めします。デュアル VRA 高可用性クラスターが必要な場合は、この単一の VRA を[「ゲートウェイの詳細 (Gateway Details)」](https://{DomainName}/classic/network/gatewayappliances)ページでアップグレードできます。
{:tip}  

1. 仮想サーバーまたはベアメタル・サーバーをすべて取り消します
2. VRA を取り消します
3. サポート・チケットにより追加の VLAN をすべて取り消します。 

## 関連コンテンツ
{:related}
- [IBM 仮想ルーター・アプライアンス](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [静的およびポータブル IP サブネット](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Vyattaの資料](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS IPsec Site-to-Site VPN Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS Tunnels Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
