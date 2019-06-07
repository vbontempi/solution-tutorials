---
copyright:
  years: 2019
lastupdated: "2019-05-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}
{:important: .important}

# オンプレミスからクラウド・リソースにセキュアかつプライベートにアクセスするために VPC/VPN ゲートウェイを使用する
{: #vpc-site2site-vpn}

IBM は、2019 年 4 月上旬に開始する VPC の早期アクセス・プログラムでは、参加を受け付けるお客様の数を限定しますが、数カ月後に利用者数を拡大する予定です。 IBM 仮想プライベート・クラウドの利用をご希望のお客様は、こちらの[応募フォーム](https://{DomainName}/vpc){: new_window} にご記入ください。IBM 担当員から次のステップについてお知らせいたします。
{: important}

IBM は、IBM Cloud のリソースを使用してオンプレミスのコンピューター・ネットワークをセキュアに拡張するためのさまざまな方法を提供しています。これにより、必要になったらサーバーをプロビジョンして不要になったら削除するという柔軟性を得ることができます。さらに、オンプレミスの機能を {{site.data.keyword.cloud_notm}} サービスに簡単かつ安全に接続できます。

このチュートリアルでは、オンプレミスの仮想プライベート・ネットワーク (VPN) ゲートウェイを、VPC 内に作成したクラウド VPN (VPC/VPN ゲートウェイ) に接続する方法について説明します。まずは、新しい {{site.data.keyword.vpc_full}} (VPC) と、サブネット、ネットワーク・アクセス制御リスト (ACL)、セキュリティー・グループ、仮想サーバー・インスタンス (VSI) などの関連リソースを作成します。
VPC/VPN ゲートウェイは、オンプレミスの VPN ゲートウェイへの [IPsec](https://en.wikipedia.org/wiki/IPsec) サイト間リンクを確立します。IPsec および [Internet Key Exchange](https://en.wikipedia.org/wiki/Internet_Key_Exchange) (IKE) プロトコルは、セキュア通信のための実績のあるオープン・スタンダードです。

セキュアでプライベートなアクセスであることをさらに実証するために、基幹業務アプリケーションを表すマイクロサービスを VSI にデプロイして {{site.data.keyword.cos_short}} (COS) にアクセスします。
COS サービスには、無料でプライベートに送受信するために使用できるダイレクト・エンドポイントがあります (ただし、{{site.data.keyword.cloud_notm}} の同じ領域内でのアクセスに限られます)。オンプレミスのコンピューターがこの COS マイクロサービスにアクセスします。すべてのトラフィックが VPN を経由するので、プライベートに {{site.data.keyword.cloud_notm}} に送信されます。

サイト間ゲートウェイとしてよく使用されるオンプレミス VPN ソリューションがいくつもあります。このチュートリアルでは、[strongSwan](https://www.strongswan.org/) の VPN ゲートウェイを使用して VPC/VPN ゲートウェイと接続します。オンプレミスのデータ・センターをシミュレートするために、{{site.data.keyword.cloud_notm}} の VSI に strongSwan ゲートウェイをインストールします。

{:shortdesc}
要約すると、VPC を使用してできることは次のとおりです。

- オンプレミス・システムを、{{site.data.keyword.cloud_notm}} で実行されているサービスおよびワークロードに接続する
- COS に低コストでプライベート接続する
- クラウド・ベースのシステムを、オンプレミスで実行しているサービスおよびワークロードに接続する

## 達成目標
{: #objectives}

* オンプレミスのデータ・センターまたは (仮想) プライベート・クラウドから仮想プライベート・クラウド環境にアクセスする。
* プライベート・サービス・エンドポイントを使用してクラウド・サービスにセキュアにアクセスする。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。このチュートリアルのマイクロサービスから COS へのアクセスにはネットワーク料金はかかりませんが、VPC へのアクセスには標準のネットワーク料金がかかります。

## アーキテクチャー
{: #architecture}

次の図は、アプリ・サーバーを含む仮想プライベート・クラウドを示しています。このアプリ・サーバーは、{{site.data.keyword.cos_short}} サービスと対話するマイクロサービスをホストします。(シミュレートされた) オンプレミス・ネットワークと仮想クラウド環境は、VPN ゲートウェイを介して接続されています。

![アーキテクチャー](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. 提供されたスクリプトを使用して、インフラストラクチャー (VPC、サブネット、ルールを指定したセキュリティー・グループ、ネットワーク ACL、および VSI) をセットアップします。
2. マイクロサービスがダイレクト・エンドポイントを介して {{site.data.keyword.cos_short}} と対話します。
3. VPC/VPN ゲートウェイをプロビジョンして、仮想プライベート・クラウド環境をオンプレミス・ネットワークに公開します。
4. オンプレミスで Strongswan のオープン・ソースの IPsec ゲートウェイ・ソフトウェアを使用して、クラウド環境との VPN 接続を確立します。

## 始める前に
{: #prereqs}

- [これらのステップに従って](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)、必要なコマンド・ライン (CLI) ツールをすべてインストールします。オプションの CLI インフラストラクチャー・プラグインが必要です。
- コマンド・ラインを使用して {{site.data.keyword.cloud_notm}} にログインします。詳しくは、[CLI の概説](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli)を参照してください。
- ユーザー許可を確認してください。VPC リソースを作成および管理するための十分な許可がユーザー・アカウントに付与されていることを確認してください。必要な許可のリストについては、[VPC ユーザーに必要な許可の付与](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)を参照してください。
- 仮想サーバーに接続するには SSH 鍵が必要です。SSH 鍵がない場合は、[鍵を作成するためのステップ](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)を参照してください。
- [**jq**](https://stedolan.github.io/jq/download/) をインストールします。提供されているスクリプトでは、これを使用して JSON 出力を処理します。

## 仮想プライベート・クラウドに仮想アプリケーション・サーバーをデプロイする
以下では、ベースライン VPC 環境をセットアップするためのスクリプトと、 {{site.data.keyword.cos_short}} と対話するマイクロサービスのコードをダウンロードします。その後、{{site.data.keyword.cos_short}} サービスをプロビジョンして、ベースラインをセットアップします。

### コードを取得する
{: #setup}
このチュートリアルでは、VPN ゲートウェイを作成する前に、スクリプトを使用してベースラインのインフラストラクチャー・リソースをデプロイします。これらのスクリプトとマイクロサービスのコードは、GitHub リポジトリーにあります。

1. アプリケーションのコードを入手します。
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. **vpc-tutorials** に移動してから **vpc-site2site-vpn** に移動して、このチュートリアル用のスクリプトの場所に変更します。
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### サービスを作成する
このセクションでは、CLI で {{site.data.keyword.cloud_notm}} にログインし、{{site.data.keyword.cos_short}} のインスタンスを作成します。

1. 前提条件としてログインの手順を実行したことを確認します。
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. **標準**プランまたは**ライト**・プランを使用して [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) のインスタンスを作成します。
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   ライト・インスタンスはアカウントごとに 1 つしか作成できないので注意してください。{{site.data.keyword.cos_short}} のインスタンスが既にある場合は、それを再使用できます。{: tip}

3. **リーダー**の役割を指定してサービス・キーを作成します。
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. サービス・キーの詳細情報を JSON 形式で取得し、サブディレクトリー **vpc-app-cos** 内の新しいファイル **credentials.json** に格納します。このファイルを後でアプリケーションで使用します。
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### 仮想プライベート・クラウドのベースライン・リソースを作成する
{: #create-vpc}
このチュートリアルでは、チュートリアルに必要なベースライン・リソース、つまり開始環境を作成するためのスクリプトを用意しています。このスクリプトは、既存の VPC に環境を生成することも、新規の VPC を作成することもできます。

以下では、セットアップ・スクリプトを構成し、実行してこれらのリソースを作成します。スクリプトには、[要塞ホストを使用してリモート・インスタンスにセキュアにアクセスする](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)で説明している要塞ホストのセットアップが組み込まれています。

1. サンプル構成ファイルを使用するためにコピーします。

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. **config.sh** ファイルを編集して、環境に合わせて設定を調整します。**SSHKEYNAME** の値を SSH 鍵の名前または SSH 鍵の名前をコンマで区切ったリストに変更する必要があります (『始める前に』を参照)。クラウドの地域に合わせて、**ZONE** のさまざまな設定を変更します。他の変数はすべてそのままでかまいません。
3. 新しい VPC にリソースを作成するには、次のようにスクリプトを実行します。

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   既存の VPC を再使用するには、次のようにして名前をスクリプトに渡します。**YOUR_EXISTING_VPC** を実際の VPC 名に置き換えてください。
   ```sh
   REUSE_VPC=YOUR_EXISTING_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. これにより、要塞関連のリソースを含む以下のリソースが作成されます。
   - VPC x 1 (オプション)
   - パブリック・ゲートウェイ x 1
   - VPC 内のサブネット x 3
   - 受信ルールおよび送信ルールを指定したセキュリティー・グループ x 4
   - VSI x 3: vpns2s-onprem-vsi (floating-ip は ONPREM_IP)、vpns2s-cloud-vsi (floating-ip は VSI_CLOUD_IP)、vpns2s-bastion (floating-ip は BASTION_IP_ADDRESS)

   後で使用できるように、**BASTION_IP_ADDRESS**、**VSI_CLOUD_IP**、**ONPREM_IP**、**CLOUD_CIDR**、および **ONPREM_CIDR** の戻り値をメモします。出力は **network_config.sh** ファイルにも格納されます。このファイルは自動セットアップに使用できます。

### 仮想プライベート・ネットワークのゲートウェイと接続を作成する
以下では、VPN ゲートウェイを追加し、アプリケーション VSI が存在するサブネットへの関連接続を追加します。

1. [VPC の概要](https://{DomainName}/vpc/overview)ページに移動し、ナビゲーション・タブの**「VPN」**をクリックし、ダイアログの**「新規 VPN ゲートウェイ (New VPN gateway)」**をクリックします。**「VPC の新規 VPN ゲートウェイ (New VPN gateway for VPC)」**フォームに、名前として **vpns2s-gateway** と入力します。正しい VPC、リソース・グループ、およびサブネット (**vpns2s-cloud-subnet**) が選択されていることを確認します。
2. **「VPC の新規 VPN 接続 (New VPN gateway for VPC)」**をアクティブのままにします。名前として **vpns2s-gateway-conn** と入力します。
3. **「ピア・ゲートウェイ・アドレス」**には、**vpns2s-onprem-vsi** の浮動 IP アドレス (ONPREM_IP) を使用します。**「事前共有鍵」**として **20_PRESHARED_KEY_KEEP_SECRET_19** と入力します。
4. **「ローカル・サブネット」**には **CLOUD_CIDR** として返された情報を使用し、**「ピア・サブネット」**には **ONPREM_CIDR** として返された情報を使用します。
5. **「デッド・ピア検出」**の設定はそのままにします。**「VPN ゲートウェイの作成」**をクリックして、ゲートウェイおよび関連接続を作成します。
6. VPN ゲートウェイが使用可能になるまで待ちます (画面を再表示しなければならない場合があります)。
7. 割り当てられた**ゲートウェイ IP** アドレスを **GW_CLOUD_IP** としてメモします。 

### オンプレミスの仮想プライベート・ネットワーク・ゲートウェイを作成する
次は、もう一方のサイトに VPN ゲートウェイを作成します。つまり、シミュレートしたオンプレミス環境です。オープン・ソース・ベースの IPsec ソフトウェア [strongSwan](https://strongswan.org/) を使用します。

1. ssh を使用して「オンプレミス」の VSI **vpns2s-onprem-vsi** に接続します。以下を実行します。**ONPREM_IP** は、前に返された IP アドレスに置き換えます。

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   環境によっては、`ssh -i <path to your private key file> root@ONPREMP_IP` を使用する必要があります。
   {:tip}

2. 次は、マシン **vpns2s-onprem-vsi** で、次のコマンドを実行してパッケージ・マネージャーを更新し、strongSwan ソフトウェアをインストールします。

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. **/etc/sysctl.conf** ファイルの最後に 3 行追加して構成します。以下をコピーして実行します。

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. 次は、**/etc/ipsec.secrets** ファイルを編集します。次の行を追加して、送信元と宛先の IP アドレス、および前に構成した事前共有鍵を構成します。**ONPREM_IP** を、vpns2s-onprem-vsi の浮動 IP の既知の値に置き換えます。**GW_CLOUD_IP** は VPC の VPN ゲートウェイの既知の IP アドレスに置き換えます。

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. 最後に構成する必要があるファイルは **/etc/ipsec.conf** です。ファイルの最後に次のコード・ブロックを追加します。**ONPREM_IP**、**ONPREM_CIDR**、**GW_CLOUD_IP**、および **CLOUD_CIDR** はそれぞれ既知の値に置き換えます。

   ```sh
   # basic configuration
   config setup
      charondebug="all"
      uniqueids=yes
      strictcrlpolicy=no
   
   # connection to vpc/vpn datacenter 
   # left=onprem / right=vpc
   conn tutorial-site2site-onprem-to-cloud
      authby=secret
      left=%defaultroute
      leftid=ONPREM_IP
      leftsubnet=ONPREM_CIDR
      right=GW_CLOUD_IP
      rightsubnet=CLOUD_CIDR
      ike=aes256-sha2_256-modp1024!
      esp=aes256-sha2_256!
      keyingtries=0
      ikelifetime=1h
      lifetime=8h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start
   ```
   {:pre}

6. ipsec restart を実行して VPN ゲートウェイを再起動し、そのステータスを確認します。

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   接続が確立されたことが報告されます。端末とこのマシンへの SSH 接続は開いたままにします。

## 接続をテストする
サイト間 VPN 接続をテストするには、SSH を使用するか、{{site.data.keyword.cos_short}} と対話するマイクロサービスをデプロイします。

### ssh を使用してテストする
VPN 接続が正常に確立されたことをテストするために、シミュレートしたオンプレミス環境をプロキシーとして使用して、クラウド・ベースのアプリケーション・サーバーにログインします。 

1. 新しい端末で、次のコマンドの値を置き換えて実行します。これは、strongSwan ホストを踏み台ホストとして使用して、VPN 経由でアプリケーション・サーバーのプライベート IP アドレスに接続します。

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. 正常に接続されたら、SSH 接続を閉じます。

3. 「オンプレミス」の VSI 端末で、VPN ゲートウェイを停止します。
   ```sh
   ipsec stop
   ```
   {:pre}
4. 手順 1) のコマンド・ウィンドウで、もう一度接続してみます。

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
   コマンドは成功しないはずです。VPN 接続がアクティブではないので、シミュレートしたオンプレミス環境とクラウド環境の間に直接リンクが存在しないからです。

   デプロイメントの詳細によっては、この接続でも実際に成功することがあるので注意してください。理由は、ゾーンをまたぐ VPC 内部接続がサポートされているからです。シミュレートしたオンプレミス VSI を別の VPC または [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers) にデプロイした場合は、VPN がないとアクセスできません。{:tip}
   
5. 「オンプレミス」の VSI 端末で、VPN ゲートウェイをもう一度起動します。
   ```sh
   ipsec start
   ```
   {:pre}
 

### マイクロサービスを使用してテストする
オンプレミス VSI からクラウド VSI 上のマイクロサービスにアクセスして、VPN 接続が機能していることをテストできます。

1. マイクロサービス・アプリケーションのコードをローカル・マシンからクラウド VSI にコピーします。このコマンドは、要塞をクラウド VSI への踏み台ホストとして使用します。**BASTION_IP_ADDRESS** および **VSI_CLOUD_IP** を適切な値に置き換えます。
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. もう一度、要塞を踏み台ホストとして使用して、クラウド VSI に接続します。
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. クラウド VSI で、コードのディレクトリーに移動します。
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. Python と Python パッケージ・マネージャー PIP をインストールします。
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. **pip** を使用して、必要な Python パッケージをインストールします。
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. アプリを起動します。
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. 「オンプレミス」の VSI 端末で、サービスにアクセスします。VSI_CLOUD_IP を適切な値に置き換えます。
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
   コマンドから JSON オブジェクトが返されるはずです。

## リソースの削除
{: #remove-resources}

1. VPC 管理コンソールで**「VPN」**をクリックします。VPN ゲートウェイのアクション・メニューで、**「削除」**を選択してゲートウェイを削除します。
2. 次は、ナビゲーションの**「浮動 IP」**をクリックしてから、VSI の IP アドレスをクリックします。アクション・メニューで**「解放」**を選択します。IP アドレスを解放することを確認します。
3. 次に、**「仮想サーバー・インスタンス」**に切り替え、インスタンスを**削除**します。インスタンスが削除され、しばらくの間、ステータスに**「削除中」**と表示されます。時々ブラウザーを最新表示するようにしてください。
4. VSI が表示されなくなったら、**「サブネット」**に切り替えます。サブネットにパブリック・ゲートウェイが接続されている場合は、サブネット名をクリックします。「サブネットの詳細」でパブリック・ゲートウェイを切り離します。パブリック・ゲートウェイのないサブネットは、概要ページで削除できます。サブネットを削除します。
5. サブネットが削除されたら、**「VPC」**に切り替えて VPC を削除します。

コンソールの使用時、リソース削除後の更新された状況情報を確認するためには、ブラウザーを最新表示しなければならない場合があります。
{:tip}

## チュートリアルを発展させる 
{: #expand-tutorial}

このチュートリアルに、追加や拡張が必要ですか。以下にいくつかのアイデアを示します。

- [ロード・バランサー](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc)を追加して、インバウンド・マイクロサービス・トラフィックを複数のインスタンスに分散させます。
- [アプリケーションをパブリック・サーバーにデプロイし、データとサービスをプライベート・ホストにデプロイ](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend)します。


## 関連コンテンツ
{: #related}

- [VPC の用語集](/docs/vpc?topic=vpc-vpc-glossary)
- [VPC のための IBM Cloud CLI プラグインのリファレンス](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [REST API を使用した VPC](/docs/infrastructure/vpc/example-code.html)
- ソリューションのチュートリアル: [要塞ホストを使用してリモート・インスタンスにセキュアにアクセスする](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
