---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-08"
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

# セキュアなプライベート・ネットワークを使用してワークロードを分離する
{: #secure-network-enclosure}

分離されたセキュアなプライベート・ネットワーク環境に対するニーズは、パブリック・クラウド上の IaaS アプリケーション・デプロイメント・モデルの中心となります。ファイアウォール、VLAN、ルーティング、および VPN はすべて、分離されたプライベート環境の構築に必要なコンポーネントです。この分離により、仮想マシンとベアメタル・サーバーを複雑な複数層のアプリケーション・トポロジーに安全にデプロイできると同時に、公衆インターネット上のリスクから保護することができます。  

このチュートリアルでは、[仮想ルーター・アプライアンス](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-) (VRA) を {{site.data.keyword.Bluemix_notm}} 上に構成して、セキュア・プライベート・ネットワーク (エンクロージャー) を作成する方法について重点的に説明します。VRA は、ファイアウォール、VPN ゲートウェイ、ネットワーク・アドレス変換 (NAT)、およびエンタープライズ・レベルのルーティングを単一の自己管理パッケージで提供します。このチュートリアルでは、VRA を使用して、エンクローズされた分離ネットワーク環境を {{site.data.keyword.Bluemix_notm}} 上に作成する方法について説明します。このエンクロージャー内で、IP ルーティング、VLAN、IP サブネット、ファイアウォール・ルール、仮想サーバー、ベアメタル・サーバーといった広く使用される、よく知られたテクノロジーを使用して、アプリケーション・トポロジーを作成できます。  

{:shortdesc}

このチュートリアルは、{{site.data.keyword.Bluemix_notm}} での従来型ネットワーキングの開始点であり、このまま実動環境で使用できると考えることはできません。追加で使用することを検討できる機能を以下に示します。
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [ハードウェア・ファイアウォール・アプライアンス](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* データ・センターへのセキュア接続のための [IPSec VPN](https://{DomainName}/catalog/infrastructure/ipsec-vpn)。
* クラスター化された VRA とデュアル・アップリンクによる高可用性。
* セキュリティー・イベントのロギングおよび監査。

## 達成目標 
{: #objectives}

* 仮想ルーター・アプライアンス (VRA) をデプロイする
* 仮想マシンとベアメタル・サーバーをデプロイするために VLAN と IP サブネットを定義する
* ファイアウォール・ルールを使用して VRA とエンクロージャーを保護する

## 使用するサービス
{: #products}

このチュートリアルでは、以下の {{site.data.keyword.Bluemix_notm}} サービスを使用します。
* [仮想ルーター・アプライアンス](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)

このチュートリアルでは、費用が発生する場合があります。VRA は月額料金プランでのみ利用可能です。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. VPN の構成
2. VRA のデプロイ 
3. 仮想サーバーの作成
4. VRA を介したアクセスのルーティング
5. エンクロージャー・ファイアウォールの構成
6. APP ゾーンの定義
7. INSIDE ゾーンの定義

## 始めに
{: #prereqs}

### VPN アクセスを構成する

このチュートリアルでは、作成したネットワーク・エンクロージャーは公衆インターネット上では表示されません。VRA およびすべてのサーバーは、プライベート・ネットワークを介してのみ利用でき、接続時には VPN を使用します。 

1. [VPN アクセスが有効になっていることを確認します](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     VPN アクセスを有効にするには**マスター・ユーザー**である必要があります。または、アクセスを有効にするようにマスター・ユーザーに連絡してください。
     {:tip}
2. [ユーザー・リスト](https://{DomainName}/iam#/users)でユーザーを選択して、VPN アクセスの資格情報を取得します。
3. [Web インターフェース](https://www.softlayer.com/VPN-Access)から VPN にログインするか、または [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx)、または [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 用の VPN クライアントを使用します。

   VPN クライアントの場合は、[VPN Web アクセス・ページ](https://www.softlayer.com/VPN-Access)にある単一のデータ・センター VPN アクセス・ポイントの完全修飾ドメイン名 (FQDN) をゲートウェイ・アドレスとして使用し、*vpn.xxxnn.softlayer.com* という形式で指定します。
{:tip}

### アカウントの許可を確認する

インフラストラクチャー・マスター・ユーザーに連絡して、以下の許可を取得してください。
- **クイック許可 (Quick Permissions)** - 基本ユーザー
- **ネットワーク** -  エンクロージャーを作成して構成できるようにするには、すべてのネットワーク許可が必要になります。 
- **サービス** - SSH 鍵を管理します。

### SSH 鍵をアップロードする

ポータル経由で [SSH 公開鍵をアップロード](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial)します。SSH 公開鍵は、VRA およびプライベート・ネットワークのアクセスおよび管理に使用されます。  

### 対象のデータ・センターを選択する

セキュアなプライベート・ネットワークをデプロイする {{site.data.keyword.Bluemix_notm}} データ・センターを選択します。 

### VLAN を注文する

ターゲット・データ・センターのプライベート・エンクロージャーを作成するには、サーバーに必要なプライベート VLAN を最初に割り当てる必要があります。最初のプライベート VLAN と最初のパブリック VLAN には費用はかかりません。複数層アプリケーション・トポロジーをサポートするための追加の VLAN には費用が発生します。 

十分な VLAN が同じデータ・センター・ルーターで使用可能であり、VRA に関連付けることができるようにするために、VLAN を注文することをお勧めします。[VLAN の注文](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans)を参照してください。

## 仮想ルーター・アプライアンスをプロビジョンする
{: #VRA}

最初のステップは、プライベート・ネットワーク・エンクロージャー用の IP ルーティングとファイアウォールを提供する VRA をデプロイすることです。{{site.data.keyword.Bluemix_notm}} 提供の公衆網向けトランジット VLAN、ゲートウェイ、およびオプションでハードウェア・ファイアウォールによってエンクロージャーからのインターネット・アクセスが可能であり、パブリック VLAN からセキュアなプライベート・エンクロージャー VLAN への接続が作成されます。このソリューションのチュートリアルでは、仮想ルーター・アプライアンス (VRA) によって境界に対する当該ゲートウェイとファイアウォールが提供されます。 

1. カタログから[「ゲートウェイ・アプライアンス」](https://{DomainName}/gen1/infrastructure/provision/gateway)を選択します。
3. **「ゲートウェイ・ベンダー」**セクションで「AT&T」を選択します。「最大 20 Gbps」または「最大 2 Gbps」のアップリンク速度を選択できます。
4. **「ホスト名」**セクションで、新しい VRA のホスト名とドメインを入力します。
5. **「高可用性」**チェック・ボックスにチェック・マークを付けた場合は、VRRP を使用したアクティブ/バックアップ・セットアップで機能する 2 つの VRA デバイスを取得します。
6. **「場所」**セクションで、VRA が必要になる場所と**ポッド**を選択します。
7. 「シングル・プロセッサー」または「デュアル・プロセッサー」を選択します。サーバーのリストが表示されます。ラジオ・ボタンをクリックして、サーバーを選択します。 
8. **RAM** の容量を指定します。実稼働環境では、最小で 64GB の RAM を使用することをお勧めします。テスト環境の場合は、RAM の最小容量は 8GB です。
9. **「SSH 鍵」**(オプション) を選択します。この SSH 鍵は VRA にインストールされ、ユーザー「vyatta」を使用してこの鍵で VRA にアクセスできるようになります。
10. ハード・ディスク。デフォルトを保持してください。
11. **「アップリンク・ポート速度」**セクションで、速度、冗長性、およびプライベート・インターフェースまたはパブリック・インターフェース (あるいはその両方) の組み合わせのうち、ニーズを満たすものを選択します。
12. **「アドオン」**セクションで、デフォルトを保持します。パブリック・インターフェースで IPv6 を使用する場合は、IPv6 アドレスを選択します。

右側に**「発注要約」**が表示されます。_「以下に表示されるサード・パーティー・サービス契約を読み、同意します」_チェック・ボックスにチェック・マークを付けて、**「作成」**ボタンをクリックします。ゲートウェイがデプロイされます。

[「装置リスト (Device list)」](https://{DomainName}/classic/devices)に、**時計**の記号が付いた VRA がほぼすぐに表示されます。この記号は、トランザクションがこの装置で進行中であることを示しています。VRA の作成が完了するまで、**時計**記号は表示されたままになり、詳細の表示以上の構成アクションを装置に対して実行することはできません。
{:tip}

### デプロイされた VRA を確認する

1. 新しい VRA を検査します。[「インフラストラクチャー・ダッシュボード (Infrastructure Dashboard)」](https://{DomainName}/classic)の左側のペインで**「ネットワーク」**を選択してから**「ゲートウェイ・アプライアンス」**を選択し、[「ゲートウェイ・アプライアンス」](https://{DomainName}/classic/network/gatewayappliances)ページに進みます。**「ゲートウェイ」**列で新しく作成された VRA の名前を選択し、「ゲートウェイ詳細 (Gateway Details)」ページに進みます。![](images/solution33-secure-network-enclosure/Gateway-detail.png)

2. VRA の `Private` IP アドレスと `Public` IP アドレスを今後の使用のためにメモします。

## 初期 VRA セットアップ
{: #initial_VRA_setup}

1. ワークステーションから SSL VPN 経由で、デフォルトの **vyatta** アカウントを使用して VRA にログインし、SSH セキュリティー・プロンプトを受け入れます。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   SSH からパスワードを求めるプロンプトが出される場合、SSH 鍵はビルドに含まれていません。[Web ブラウザー](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui)で `VRA Private IP Address` を使用して VRA にアクセスしてください。パスワードは、[ソフトウェア・パスワード](https://{DomainName}/classic/devices/passwords)のページから取得します。**「構成」**タブで、システム/ログイン/vyatta ブランチを選択して、必要な SSH 鍵を追加します。
{:tip}

   VRA をセットアップするには、`configure` コマンドを使用して VRA を「編集」モードにする必要があります。`edit` モードでは、プロンプトは `$` から `#` に変わります。VRA 構成の変更が正常に行われた後、`compare` コマンドを使用して変更を表示し、`validate` コマンドで変更内容を確認できます。`commit` コマンドを使用して変更を確定すると、その変更は実行中の構成に適用され、開始構成に自動的に保存されます。


   {:tip}
2. SSH ログインのみ許可することによって、セキュリティーを強化します。プライベート・ネットワーク経由の SSH ログインが成功したため、ユーザー ID とパスワードによる認証を使用したアクセスを無効にします。 
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
   本チュートリアルのこの時点では、すべの VRA コマンドは、`configure` コマンドの入力後に `edit` プロンプトで入力されるものと想定されます。
3. 初期構成を確認します。
   ```
   show
   ```
   {: codeblock}

   VRA は {{site.data.keyword.Bluemix_notm}} IaaS 環境用に事前構成されています。これには、以下が含まれます。
   - NTP サーバー
   - ネーム・サーバー
   - SSH
   - HTTPS Web サーバー 
   - デフォルト時間帯の米国/シカゴ
4. 必要に応じて、ローカル・タイム・ゾーンを設定します。Tab キーを使用してオートコンプリートを行うと、可能なタイム・ゾーン値がリストされます。
   ```
   set system time-zone <timezone>
   ```
   {: codeblock}
5. ping 動作を設定します。ルーティングやファイアウォールのトラブルシューティングに役立つように ping は無効化されていません。 
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. ステートフルなファイアウォール操作を有効にします。デフォルトでは、VRA ファイアウォールはステートレスです。 
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. 開始構成に対する変更を確定して、自動的に保存します。 
   ```
   commit
   ```
   {: codeblock}

## 最初の仮想サーバーを注文する
{: #order_virtualserver}

この時点で、VRA 構成エラーの診断に役立つように仮想サーバーが作成されます。{{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク上で VSI への正常なアクセスが検証されてから、その後のステップで VSI へのアクセスが VRA 経由でルーティングされます。 

1. [仮想サーバー](https://{DomainName}/catalog/infrastructure/virtual-server-group)を注文します。  
2. **「パブリック Virtual Server」**を選択して先に進みます。
3. 注文ページで、以下のようにします。
   - **「請求処理」**を**「毎時」**に設定します。
   - *「VSI ホスト名 (VSI Hostname)」*と*「ドメイン・ネーム」*を設定します。このドメイン・ネームは、ルーティングおよび DNS では使用されませんが、ネットワーク・ネーミング標準に準拠している必要があります。 
   - **「場所」**を VRA と同一に設定します。
   - **「プロファイル」**を**「C1.1x1」**に設定します。
   - 上記で指定した**「SSH 鍵」**を追加します。
   - **「オペレーティング・システム」**を**「CentOS 7.x - Minimal」**に設定します。
   - **「アップリンク・ポート速度」**で、ネットワーク・インターフェースをデフォルトの*「パブリックおよびプライベート (public and private)」*から**「プライベート・ネットワーク・アップリンク (Private Network Uplink)」**のみ指定するように変更する必要があります。これにより、新規サーバーはインターネットに直接アクセスできず、アクセスは VRA 上のルーティング・ルールとファイアウォール・ルールにより制御されるようになります。
   - **「プライベート VLAN」**を前の手順で注文したプライベート VLAN の VLAN ID に設定します。
4. チェック・ボックスをクリックして、「サード・パーティー」サービス契約書を受け入れて、**「作成」**をクリックします。
5. [「デバイス」](https://{DomainName}/classic/devices)ページで処理が完了するのを確認するか、E メールで確認します。 
6. 後のステップで使用するために、VSI の*プライベート IP アドレス*をメモして、**「デバイス詳細」**ページの**「ネットワーク」**セクションで、VSI が正しい VLAN に割り当てられていることを確認します。正しい VLAN に割り当てられていない場合は、この VSI を削除して、正しい VLAN 上に新規 VSI を作成します。 
7. VPN 上のローカル・ワークステーションからの ping および SSH を使用して {{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク経由で VSI に正常にアクセスできることを確認します。
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## VRA を使用して VLAN アクセスをルーティングする
{: #routing_vlan_via_vra}

仮想サーバーのプライベート VLAN が {{site.data.keyword.Bluemix_notm}} 管理システムによってこの VRA に関連付けられます。この段階では、VSI は {{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク上の IP ルーティング経由でまだアクセス可能です。ここで、VRA を使用してサブネットをルーティングし、セキュアなプライベート・ネットワークを作成し、VSI にアクセスできなくなったことを確認して、ネットワークが保護されていることを検証します。 

1. [「ゲートウェイ・アプライアンス」](https://{DomainName}/classic/network/gatewayappliances)ページから VRA のゲートウェイ詳細に進み、ページの下半分にある**「関連する VLAN (Associated VLANs)」**セクションを見つけてください。関連する VLAN がここにリストされます。 
2. この時点で、その他の VLAN を追加する必要がある場合は、**「VLAN の関連付け (Associate a VLAN)」**セクションに移動します。ドロップダウン・ボックスの*「VLAN の選択 (Select VLAN)」*が有効になっているはずです。プロビジョンされたその他の VLAN を選択できます。![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png)

   適格な VLAN が示されていない場合、VRA と同じルーター上に使用可能な VLAN はありません。この場合、[サポート・チケット](https://{DomainName}/unifiedsupport/cases/add)を発行して、VRA と同じルーター上にプライベート VLAN を要求する必要があります。
{:tip}
5. VRA に関連付ける VLAN を選択して、「保存」をクリックします。最初の VLAN の関連付けが完了するまでに、数分かかることがあります。関連付けが完了すると、**「関連する VLAN (Associated VLANs)」**見出しの下に VLAN が表示されます。 

この段階では、VLAN および関連するサブネットは保護も VRA によるルーティングも行われておらず、VSI は {{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク経由でアクセスされます。VLAN の状況は*「バイパス (Bypassed)」*と表示されます。

4. 右側の列で**「アクション」**を選択し、次に**「VLAN のルーティング (Route VLAN)」**を選択して、VRA 経由で VLAN/サブネットをルーティングします。この処理には数分かかります。画面が最新表示されて、*ルーティング*されたことが示されます。 
5. [VLAN 名](https://{DomainName}/classic/network/vlans/)を選択して、VLAN の詳細を表示します。プロビジョン済み VSI、および割り当てられた 1 次 IP サブネットを確認できます。プライベート VLAN ID \<nnnn\> (この例では 1199) をメモします (後のステップで使用します)。 
6. [サブネット](https://{DomainName}/classic/network/subnets)を選択して、IP サブネットの詳細を確認します。サブネット・ネットワーク、ゲートウェイ・アドレス、および CIDR (/26) をメモします。これらは、さらに詳細な VRA の構成で必要になります。64 個の 1 次 IP アドレスがプライベート・ネットワーク上にプロビジョンされます。ゲートウェイ・アドレスを見つけるために、2 ページ目または 3 ページ目の選択が必要になる場合があります。 
7. サブネット/VLAN が VRA にルーティングされ、ワークステーションから管理ネットワーク経由で `ping` を使用して VSI にアクセス**できない**ことを確認します。 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}

これで、{{site.data.keyword.Bluemix_notm}} コンソールを使用した VRA のセットアップは完了です。続いて、エンクロージャーと IP ルーティングを構成するための追加の作業を SSH を使用して VRA 上で直接実行します。 

## IP ルーティングとセキュア・エンクロージャーを構成する
{: #vra_setup}

VRA の構成が確定されると、実行中の構成が変更され、その変更内容が自動的に開始構成に保存されます。

以前の作業構成に戻ることが望ましい場合は、デフォルトで、最後の 20 個のコミット・ポイントを表示、比較、または復元できます。構成のコミットおよび保存についての詳細は、[Vyatta Network OS Basic System Configuration Guide](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation) を参照してください。
   ```bash
   show system commit 
   rollback n
   compare
   ```
   {: codeblock}

### VRA の IP ルーティングを構成する

{{site.data.keyword.Bluemix_notm}} プライベート・ネットワークから新規サブネットにルーティングするための VRA 仮想ネットワーク・インターフェースを構成します。  

1. SSH を使用して VRA にログインします。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}
2. 上記のステップで記録しておいたプライベート VLAN ID、サブネット・ゲートウェイ IP アドレス、および CIDR を使用して新しい仮想インターフェースを作成します。通常、CIDR は /26 です。 
   ```
   set interfaces bonding dp0bond0 vif <VLAN ID> address <Subnet Gateway IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   **`<Subnet Gateway IP>`** アドレスが使用されていることが重要です。これは通常、サブネット範囲内の最初のアドレスの 1 つです。無効なゲートウェイ・アドレスを入力すると、「`Configuration path: interfaces bonding dp0bond0 vif xxxx address [x.x.x.x] is not valid`」というエラーが出されます。コマンドを訂正して再入力してください。このゲートウェイ・アドレスは、「ネットワーク」>「IP 管理」>「サブネット」で確認することができます。ゲートウェイ・アドレスを確認するために必要なサブネットをクリックします。**「ゲートウェイ」**という記述がある、リスト内の 2 番目の項目が <サブネット・ゲートウェイ IP>/<CIDR> として入力する IP アドレスです。
   {: tip}

3. 新規仮想インターフェース (vif) をリストします。 
   ```
   show interfaces
   ```
   {: codeblock}

   以下は、vif 1199 とサブネット・ゲートウェイ・アドレスが示されたインターフェース構成の例です。
![](images/solution33-secure-network-enclosure/show_interfaces.png)
4. ワークステーションから管理ネットワーク経由で VSI に再度アクセスできることを確認します。 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
   
   VSI にアクセスできない場合は、VRA IP ルーティング・テーブルが予想どおり構成されていることを確認してください。必要に応じて、ルートを削除して再作成します。構成モードで show コマンドを実行するには、run コマンドを使用します。    
   ```bash
    run show ip route <Subnet Gateway IP>
   ```
   {: codeblock}

これで、IP ルーティングの構成は完了です。

### セキュア・エンクロージャーを構成する

セキュアなプライベート・ネットワーク・エンクロージャーは、ゾーンとファイアウォール・ルールを構成することによって作成されます。手順を進める前に、[ファイアウォール構成](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls)に関する VRA の資料を確認してください。 

以下の 2 つのゾーンが定義されています。
   - INSIDE: IBM のプライベート・ネットワークおよび管理ネットワーク
   - APP: プライベート・ネットワーク・エンクロージャー内のユーザーの VLAN およびサブネット		

1. ファイアウォールとデフォルトを定義します。
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   set コマンドが間違って 2 回実行されると、「*Configuration path xxxxxxxx is not valid. Node exists*」というメッセージが表示されます。これは無視してかまいません。間違ったパラメーターを変更するには、「delete security xxxxx xxxx xxxxx」と表示されているノードを最初に削除する必要があります。
{:tip}
2. {{site.data.keyword.Bluemix_notm}} プライベート・ネットワーク・リソース・グループを作成します。このアドレス・グループは、エンクロージャー、およびエンクロージャーから到達可能なネットワークにアクセスできる {{site.data.keyword.Bluemix_notm}} プライベート・ネットワークを定義します。2 つのセットの IP アドレスがセキュアなエンクロージャーとの間のアクセスを必要とします。これらは、SSL VPN データ・センターと {{site.data.keyword.Bluemix_notm}} サービス・ネットワーク (バックエンド/プライベート・ネットワーク) です。[{{site.data.keyword.Bluemix_notm}} の IP 範囲](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges)に、許可が必要な IP 範囲の全リストが提供されています。 
   - VPN アクセスに使用しているデータ・センターの SSL VPN アドレスを定義します。「{{site.data.keyword.Bluemix_notm}} の IP 範囲」の SSL VPN のセクションから、データ・センターまたは DC クラスターの VPN アクセス・ポイントを選択します。以下の例では、{{site.data.keyword.Bluemix_notm}} ロンドンのデータ・センターの VPN アドレス範囲が示されています。
```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - {{site.data.keyword.Bluemix_notm}} の「サービス・ネットワーク (バックエンド/プライベート・ネットワーク上)」の WDC04、DAL01、および目的のデータ・センターのアドレス範囲を定義します。以下の例は、WDC04 (2 つのアドレス)、DAL01、および LON06 です。
```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. ユーザーの VLAN およびサブネットの APP ゾーン、および {{site.data.keyword.Bluemix_notm}} プライベート・ネットワークの INSIDE ゾーンを作成します。以前に作成したファイアウォールを割り当てます。ゾーン定義では、VRA ネットワーク・インターフェース名を使用して、各 VLAN に関連付けられているゾーンを識別します。APP ゾーンを作成するコマンドでは、以前に作成した VRA に関連する VLAN の VLAN ID を指定する必要があります。この ID は、以下の例では `<VLAN ID>` と強調表示されています。
   ```
   set security zone-policy zone INSIDE description "IBM Internal network"
   set security zone-policy zone INSIDE default-action drop
   set security zone-policy zone INSIDE interface dp0bond0
   set security zone-policy zone INSIDE to APP firewall INSIDE-TO-APP 

   set security zone-policy zone APP description "Application network"
   set security zone-policy zone APP default-action drop

   set security zone-policy zone APP interface dp0bond0.<VLAN ID> 
   set security zone-policy zone APP to INSIDE firewall APP-TO-INSIDE 
   ```
   {: codeblock}
4. 構成を確定し、ワークステーションから ping を使用して、ファイアウォールで VRA を経由した VSI へのトラフィックが拒否されていることを確認します。 
   ```
   commit
   ```
   {: codeblock}

   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
5. UDP、TCP、および ICMP のファイアウォール・アクセス・ルールを定義します。
   ```
   set security firewall name INSIDE-TO-APP rule 200 protocol icmp
   set security firewall name INSIDE-TO-APP rule 200 icmp type 8
   set security firewall name INSIDE-TO-APP rule 200 action accept 
   set security firewall name INSIDE-TO-APP rule 200 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 100 action accept 
   set security firewall name INSIDE-TO-APP rule 100 protocol tcp
   set security firewall name INSIDE-TO-APP rule 100 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 110 action accept 
   set security firewall name INSIDE-TO-APP rule 110 protocol udp
   set security firewall name INSIDE-TO-APP rule 110 source address ibmprivate
   commit

   set security firewall name APP-TO-INSIDE rule 200 protocol icmp
   set security firewall name APP-TO-INSIDE rule 200 icmp type 8
   set security firewall name APP-TO-INSIDE rule 200 action accept 
   set security firewall name APP-TO-INSIDE rule 200 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 100 action accept 
   set security firewall name APP-TO-INSIDE rule 100 protocol tcp
   set security firewall name APP-TO-INSIDE rule 100 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 110 action accept 
   set security firewall name APP-TO-INSIDE rule 110 protocol udp
   set security firewall name APP-TO-INSIDE rule 110 destination address ibmprivate
   commit
   ```
   {: codeblock}
6. ファイアウォール・アクセスを確認します。 
   - ローカル・マシンから INSIDE-TO-APP ファイアウォールで ICMP および UDP/TCP トラフィックが許可されていることを確認します。
```bash
     ping <VSI Private IP Address>
     SSH root@<VSI Private IP Address>
     ```
     {: codeblock}
   - APP-TO-INSIDE ファイアウォールで ICMP および UDP/TCP トラフィックが許可されていることを確認します。SSH を使用して VSI にログインし、10.0.80.11 および 10.0.80.12 の {{site.data.keyword.Bluemix_notm}} ネーム・サーバーのいずれかを ping します。
```bash
     SSH root@<VSI Private IP Address>
     [root@vsi  ~]# ping 10.0.80.11 
     ```
     {: codeblock}
7. ワークステーションから SSH 経由で VRA 管理インターフェースに継続してアクセスできることを確認します。アクセスが維持されている場合、構成を確認して保存します。そうでない場合は、VRA をリブートすることによって、作業構成に戻ります。 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   ```
   show security  
   ```
   {: codeblock}

### ファイアウォール・ルールをデバッグする

VRA 操作コマンド・プロンプトからファイアウォール・ログを表示できます。この構成では、ファイアウォールの間違った構成の診断に役立つように、各ゾーンのドロップされたトラフィックのみログに記録されます。  

1. 拒否されたトラフィックのファイアウォール・ログを確認します。ログの定期的な確認により、APP ゾーンのサーバーが IBM ネットワーク上のサービスに適切に接続を行っているのか、誤って接続しようとしているのかを識別できます。 
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. サービスにもサーバーにも接続できず、ファイアウォール・ログに何も表示されない場合は、{{site.data.keyword.Bluemix_notm}} プライベート・ネットワークからの VRA ネットワーク・インターフェース、または前述の `<VLAN ID>` を使用する VLAN への VRA インターフェース上に、予期される ping/SSH IP トラフィックが存在するかどうかを確認します。
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN ID> traffic
   ```

## VRA を保護する
{: #securing_the_vra}

1. VRA セキュリティー・ポリシーを適用します。デフォルトでは、ポリシー・ベースのファイアウォール・ゾーニングでは VRA 自体へのアクセスは保護されません。これは、Control Plane Policing (CPP) により構成されます。VRA は基本的な CPP ルール・セットをテンプレートとして提供します。次のように、このルール・セットを現行の構成にマージします。
   ```bash
   merge /opt/vyatta/etc/cpp.conf 
   ```
   {: codeblock}

これにより、`CPP` という名前の新規ファイアウォール・ルール・セットが作成されます。追加のルールを表示して、「編集」モードでコミットします。 
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. パブリック SSH アクセスを保護します。現時点では Vyatta ファームウェアに関する未解決の問題があるため、`set service SSH listen-address x.x.x.x` を使用して、パブリック・ネットワーク上の SSH 管理アクセスを制限することは推奨されません。代わりに、VRA パブリック・インターフェースで使用されるパブリック IP アドレス範囲に対して、CPP ファイアウォール経由で外部アクセスをブロックできます。ここで使用される `<VRA Public IP Subnet>` は、最後のオクテットがゼロ (x.x.x.0) の `<VRA Public IP Address>` と同一です。 
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <VRA Public IP Subnet>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit 
   ```
   {: codeblock}
3. IBM 内部ネットワーク上の VRA SSH 管理アクセスを確認します。コミットの実行後に SSH 経由の VRA へのアクセスが失われた場合、「アクション」ドロップダウン・メニューにある VRA の「デバイス詳細」ページから使用可能な KVM コンソールを介して VRA にアクセスできます。

これで、VLAN とサブネットを含む単一のファイアウォール・ゾーンを保護するセキュアなプライベート・ネットワーク・エンクロージャーのセットアップが完了しました。追加のファイアウォール・ゾーン、ルール、仮想サーバーとベアメタル・サーバー、VLAN、およびサブネットは、同様の手順で追加できます。 

## リソースの削除
{: #removeresources}

このステップでは、リソースをクリーンアップして、上記で作成したものを削除します。

VRA は月額料金プランで提供されます。取り消しても払い戻しはありません。取り消しは、この VRA が翌月に再び必要とならない場合にのみ行うことをお勧めします。二重の VRA 高可用性クラスターが必要な場合は、[ゲートウェイ詳細](https://{DomainName}/classic/network/gatewayappliances/)ページでこの単一 VRA をアップグレードできます。
{:tip}  

- 仮想サーバーまたはベアメタル・サーバーをすべて取り消します
- VRA を取り消します
- サポート・チケットにより追加の VLAN をすべて取り消します。 

## 関連コンテンツ
{: #related}

- [IBM 仮想ルーター・アプライアンス](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [静的 IP サブネットおよびポータブル IP サブネット](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence Platform](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Vyattaの資料](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
