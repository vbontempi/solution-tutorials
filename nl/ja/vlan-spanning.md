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

# IBM ネットワークを介したセキュアなプライベート・ネットワークのリンク
{: #vlan-spanning}

Web アプリケーションを年中無休で世界中に提供するというニーズが高まるにつれて、複数のクラウド・データ・センターでサービスをホストするニーズも高まっています。複数のロケーションにデータ・センターを設置すれば、1 つの地域で障害が発生した場合の耐障害性を確保できます。また、世界中に分散しているユーザーの近くにワークロードを持ってくることで、待ち時間の削減と体感パフォーマンスの向上にもつながります。[{{site.data.keyword.Bluemix_notm}} ネットワーク](https://www.ibm.com/cloud/data-centers/)を使用すれば、複数のセキュアなプライベート・ネットワークでホストしているワークロードを、データ・センターやロケーションをまたいでリンクすることができます。

このチュートリアルでは、{{site.data.keyword.Bluemix_notm}} プライベート・ネットワークを使用して、別々のデータ・センターでホストされている 2 つのセキュア・プライベート・ネットワークの間にプライベート・ルーティングの IP 接続をセットアップします。すべてのリソースは 1 つの {{site.data.keyword.Bluemix_notm}} アカウントが所有するものです。[セキュアなプライベート・ネットワークによるワークロードの分離]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)のチュートリアルを基にデプロイした 2 つのプライベート・ネットワークを、[VLAN スパンニング]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)・サービスを使用して {{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク経由でセキュアにリンクします。
{:shortdesc}

## 達成目標
{: #objectives}

- {{site.data.keyword.Bluemix_notm}} IaaS アカウント内でセキュアなネットワークをリンクする
- サイト間アクセスのためにファイアウォール・ルールをセットアップする 
- サイト間のルーティングを構成する

## 使用するサービス
{: #products}

このチュートリアルでは、以下の {{site.data.keyword.Bluemix_notm}} サービスを使用します。 
* [仮想ルーター・アプライアンス](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [VLAN スパンニング]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

このチュートリアルでは、費用が発生する場合があります。VRA は月額料金プランでのみ利用可能です。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. セキュアなプライベート・ネットワークをデプロイします。
2. VLAN スパンニングを構成します。
3. プライベート・ネットワーク間の IP ルーティングを構成します。
4. リモート・サイトにアクセスするためのファイアウォール・ルールを構成します。

## 始める前に
{: #prereqs}

このチュートリアルは、[セキュアなプライベート・ネットワークによるワークロードの分離]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)のチュートリアルが前提になっています。始める前に、前提チュートリアルとその必須条件を確認してください。 

## セキュアなプライベート・ネットワーク・サイトを構成する
{: #private_network}

[セキュアなプライベート・ネットワークによるワークロードの分離]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)のチュートリアルを 2 回使用して、2 つのデータ・センターにプライベート・ネットワークを実装します。2 つのデータ・センターとして利用するものを選択するときに特に制限はありません。ただし、サイト間通信のトラフィックやワークロードの待ち時間への影響は考慮する必要があります。 

選択した各データ・センターで、[セキュアなプライベート・ネットワークによるワークロードの分離]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)のチュートリアルを実行します。変更は不要です。後の手順で使用する以下の情報を記録してください。 

| 項目  | データ・センター 1 | データ・センター 2 |
|:------ |:--- | :--- |
| データ・センター |  |  |
| VRA パブリック IP アドレス | <DC1 VRA Public IP Address> | <DC2 VRA Public IP Address> |
| VRA プライベート IP アドレス | <DC1 VRA Private IP Address> | <DC2 VRA Private IP Address> |
| VRA プライベート・サブネット & CIDR |  |  |
| プライベート VLAN ID | &lt;DC1 Private VLAN ID&gt;  | &lt;DC2 Private VLAN ID&gt; |
| VSI プライベート IP アドレス | <DC1 VSI Private IP Address> | <DC2 VSI Private IP Address> |
| APP ゾーン・サブネット & CIDR | <DC1 APP zone subnet/CIDR> | <DC2 APP zone subnet/CIDR> |

1. [「ゲートウェイ・アプライアンス」](https://{DomainName}/classic/network/gatewayappliances)ページから、各 VRA の「ゲートウェイの詳細」ページに進みます。  
2. 「ゲートウェイ VLAN」セクションを見つけて、**プライベート**・ネットワークにあるゲートウェイ [VLAN]( https://{DomainName}/classic/network/vlans) をクリックし、VLAN の詳細情報を表示します。名前に `bcrxxx` という ID が含まれています。bcr は「バックエンド・カスタマー・ルーター」の略で、`nnnxx.bcrxxx.xxxx` という形式になります。
3. プロビジョンされている VRA が**「デバイス」*セクションに表示されます。**「サブネット*」*セクションにある VRA プライベート・サブネットの IP アドレスと CIDR (/26) をメモしてください。そのサブネットが 64 個の IP を含むプライマリー・タイプになります。後でルーティングを構成するときに、この詳細情報が必要になります。 
4. 次に、「ゲートウェイの詳細」ページで**「関連付けられた VLAN」**セクションを見つけて、セキュアなネットワークと APP ゾーンを作成するときに関連付けた**プライベート**・ネットワークの [VLAN]( https://{DomainName}/classic/network/vlans) をクリックします。 
5. プロビジョンされている VSI が**「デバイス」*セクションに表示されます。**「サブネット*」*セクションにある VSI サブネットの IP アドレスと CIDR (/26) をメモしてください。ルーティングを構成するときに必要になります。この VLAN とサブネットを両方の VRA ファイアウォール構成で APP ゾーンとして指定し、&lt;APP Zone subnet/CIDR&gt; として記録します。


## VLAN スパンニングを構成する 
{: #configure-vlan-spanning}

デフォルトでは、別々の VLAN とデータ・センターにあるサーバー (および VRA) は、プライベート・ネットワーク上で相互に通信できません。これらのチュートリアルでは、1 つのデータ・センターの中で VLAN 間のサーバー通信用のプライベート・ネットワークを作成するために、複数の VRA で従来の IP ルーティングとファイアウォールを使用して VLAN とサブネットをリンクします。同じデータ・センター内の通信は可能ですが、この構成では、同じ {{site.data.keyword.Bluemix_notm}} アカウントに属するサーバー同士がデータ・センターをまたいで通信することはできません。 

[VLAN スパンニング]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)・サービスは、VRA に関連付けられて**いない** VLAN とサブネットの間の通信に対するこの制限を取り払うことができます。ただし、VLAN スパンニングを有効にしても、VRA に関連付けられている VLAN は、その関連付けられている VRA を経由し、VRA のファイアウォールとルーティングの構成に指定されたとおりの通信しかできないので注意してください。VLAN スパンニングを有効にすると、{{site.data.keyword.Bluemix_notm}} アカウントが所有する VRA 同士がプライベート・ネットワーク経由で接続され、通信可能になります。 

VRA で作成したセキュアなプライベート・ネットワークに関連付けられていない VLAN が「スパンニング」され、「関連付けられていない」VLAN の間で、データ・センターをまたいだ相互接続が可能になります。同じ IBM Cloud アカウントに属し、別々のデータ・センターにある VRA ゲートウェイ (中継) VLAN も対象になります。したがって、VLAN スパンニングを有効にすると、データ・センター間の VRA の通信が可能になります。VRA 間の接続によって、VRA のファイアウォールとルーティングの構成を使用して、セキュアなネットワーク上のサーバー同士を接続することができます。 

VLAN スパンニングを有効にします。

1. [「VLAN」]( https://{DomainName}/classic/network/vlans)ページに進みます。
2. ページ上部にある**「スパン」**タブを選択します。
3. VLAN スパンニングの「オン」ラジオ・ボタンを選択します。ネットワークの変更が完了するまでに数分かかります。
4. 2 つの VRA 同士が通信できるようになったことを確認します。

   データ・センター 1 の VRA にログインして、データ・センター 2 の VRA に ping を実行します。

   ```
   SSH vyatta@<DC1 VRA Private IP Address>
   ping <DC2 VRA Private IP Address>
   ```
   {: codeblock}

   データ・センター 2 の VRA にログインして、データ・センター 1 の VRA に ping を実行します。
   ```
   SSH vyatta@<DC2 VRA Private IP Address>
   ping <DC1 VRA Private IP Address>
   ```
   {: codeblock}

## VRA の IP ルーティングを構成する 
{: #vra_routing}

各データ・センターで VRA ルーティングを作成し、両方のデータ・センターの APP ゾーンにある VSI 間の通信を可能にします。 

1. データ・センター 1 で、データ・センター 2 の APP ゾーン・プライベート・サブネットへの静的ルートを VRA 編集モードで作成します。
   ```
   ssh vyatta@<DC1 VRA Private IP Address>
   conf
   set protocols static route <DC2 APP zone subnet/CIDR>  next-hop <DC2 VRA Private IP>
   commit
   ```
   {: codeblock}
2. データ・センター 2 で、データ・センター 1 の APP ゾーン・プライベート・サブネットへの静的ルートを VRA 編集モードで作成します。
   ```
   ssh vyatta@<DC2 VRA Private IP Address>
   conf
   set protocols static route <DC1 APP zone subnet/CIDR>  next-hop <DC1 VRA Private IP>
   commit
   ```
   {: codeblock}
2. VRA コマンド・ラインで VRA ルーティング・テーブルを確認します。この時点では、VSI 同士の通信はできません。2 つの APP ゾーン・サブネット間のトラフィックを許可する APP ゾーン・ファイアウォール・ルールが存在しないからです。ファイアウォール・ルールは、いずれの側で開始されたトラフィックに対しても必要になります。
   ```
   show ip route
   ```
   {: codeblock}

IBM プライベート・ネットワーク経由の通信を APP ゾーンに許可する新しいルートが表示されます。 

## VRA ファイアウォールの構成
{: #vra_firewall}

既存の APP ゾーン・ファイアウォール・ルールの構成では、このサブネットと {{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク上の {{site.data.keyword.Bluemix_notm}} サービスとの間のトラフィックと、NAT によるパブリック・インターネット・アクセスだけが許可されています。この VRA で VSI に関連付けられている他のサブネットや他のデータ・センターのサブネットはブロックされます。次の手順で、APP-TO-INSIDE ファイアウォール・ルールに関連付けられている `ibmprivate` リソース・グループを更新して、他のデータ・センターのサブネットへの明示的なアクセスを許可します。 

1. データ・センター 1 の VRA 編集コマンド・モードで、<DC2 APP zone subnet>/CIDR を `ibmprivate` リソース・グループに追加します。
   ```
   set resources group address-group ibmprivate address <DC2 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
2. データ・センター 2 の VRA 編集コマンド・モードで、<DC1 APP zone subnet>/CIDR を `ibmprivate` リソース・グループに追加します。
   ```
   set resources group address-group ibmprivate address <DC1 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
3. 両方のデータ・センターの VSI 同士が通信できることを確認します。
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}

   VSI 同士の通信ができない場合は、[セキュアなプライベート・ネットワークによるワークロードの分離]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)のチュートリアルに、インターフェース上のトラフィックをモニターしたりファイアウォールのログを確認したりする手順を記載しているので実行してください。 

## リソースの削除
{: #removeresources}

このチュートリアルで作成したリソースを削除する手順は以下のとおりです。 

VRA は月額料金プランで提供されます。取り消しても払い戻しはありません。取り消しは、この VRA が翌月に再び必要とならない場合にのみ行うことをお勧めします。デュアル VRA 高可用性クラスターが必要な場合は、この単一の VRA を[「ゲートウェイの詳細 (Gateway Details)」](https://{DomainName}/classic/network/gatewayappliances)ページでアップグレードできます。
{:tip}

1. 仮想サーバーまたはベアメタル・サーバーをすべて取り消します
2. VRA を取り消します
3. サポート・チケットにより追加の VLAN をすべて取り消します。 


## チュートリアルを発展させる

このチュートリアルと [VPN をセキュア・プライベート・ネットワークへ](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network)のチュートリアルを併用すれば、両方のセキュア・ネットワークを IPSec VPN 経由でユーザーのリモート・ネットワークにリンクできます。両方のセキュア・ネットワークへの VPN リンクを確立すれば、{{site.data.keyword.Bluemix_notm}} IaaS プラットフォームへのアクセスの耐障害性が高まります。ただし、IBM は、IBM プライベート・ネットワークを経由したクライアント・データ・センター間のユーザー・トラフィックのルーティングを許可していません。ネットワーク・ループを回避するためのルーティング構成は、このチュートリアルの範囲を超えています。 


## 関連資料
{:related}

1. {{site.data.keyword.Bluemix_notm}} アカウントのすべてのネットワークを接続する VLAN スパンニングを使用する代わりに、Virtual Routing and Forwarding (VRF) を使用することもできます。VRF は、[{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)を使用するすべてのクライアントにとって必須の機能です。[Virtual Routing and Forwarding (VRF) on IBM Cloud の概要](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [IBM Cloud のネットワーク](https://www.ibm.com/cloud/data-centers/)
