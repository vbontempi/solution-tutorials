---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# LAMP スタック上の PHP Web アプリケーション
{: #lamp-stack}

このチュートリアルでは、Ubuntu **L**inux 仮想サーバー (**A**pache Web サーバーを含む)、**M**ySQL データベース、および **P**HP スクリプトの作成について説明します。このソフトウェアの組み合わせは、より一般的には LAMP スタックと呼ばれ、通常、Web サイトおよび Web アプリケーションの配信に使用されます。{{site.data.keyword.BluVirtServers}} を使用して、組み込みモニタリングおよび脆弱性スキャンとともに LAMP スタックを素早くデプロイします。 稼働中の LAMP サーバーを確認するために、無料でオープン・ソースの [WordPress](https://wordpress.org/) コンテンツ管理システムをインストールして構成します。

## 達成目標

* 数分での LAMP サーバーのプロビジョン
* 最新バージョンの Apache、MySQL、および PHP の適用
* WordPress のインストールおよび構成による Web サイトまたはブログのホスト
* モニタリングを使用した障害およびパフォーマンスの遅延の検出
* 脆弱性の評価と望ましくないトラフィックからの保護

## 使用するサービス

このチュートリアルでは、以下のランタイムおよびサービスを使用します。

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

このチュートリアルでは、費用が発生する場合があります。 [料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー

![アーキテクチャー図](images/solution4/Architecture.png)

1. エンド・ユーザーが Web ブラウザーを使用して LAMP サーバーおよびアプリケーションにアクセスします

## 始める前に

{: #prereqs}

1. 以下の権限を割り当てるように、インフラストラクチャー管理者に依頼してください。
  * **パブリックおよびプライベートのネットワーク・アップリンク**の完了に必要なネットワーク許可

### VPN アクセスの構成

1. [VPN アクセスが有効になっていることを確認します](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     VPN アクセスを有効にするには**マスター・ユーザー**である必要があります。または、アクセスを有効にするようにマスター・ユーザーに連絡してください。
     {:tip}
2. [「ユーザー」リストの下の自分のユーザー・ページ](https://{DomainName}/iam#/users)で VPN アクセスの資格情報を取得します。
3. [Web インターフェース](https://www.softlayer.com/VPN-Access)から VPN にログインするか、または [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx)、または [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 用の VPN クライアントを使用します。

   VPN クライアントの場合は、[VPN Web アクセス・ページ](https://www.softlayer.com/VPN-Access)にある単一のデータ・センター VPN アクセス・ポイントの完全修飾ドメイン名 (FQDN) をゲートウェイ・アドレスとして使用し、*vpn.xxxnn.softlayer.com* という形式で指定します。
{:tip}

## サービスの作成

このセクションでは、パブリック仮想サーバーに修正された構成をプロビジョンします。{{site.data.keyword.BluVirtServers_short}}は、特定の地理的位置の仮想サーバー・イメージから数分でデプロイできます。仮想サーバーは、通常、需要のピークに対応した後、中断または電源遮断されることがあり、これにより、クラウド環境がインフラストラクチャーのニーズに適合するようになっています。

1. ご使用のブラウザーで[{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)のカタログ・ページにアクセスします。
2. **「パブリック Virtual Server」**を選択してから、**「作成」**をクリックします。
3. **「イメージ」**の**「Ubuntu」**で最新バージョンの **LAMP** を選択します。 これは Apache、MySQL、および PHP ではプリインストールされていますが、最新バージョンを使用して PHP および MySQL を再インストールします。
4. **「ネットワーク・インターフェース」**で**「パブリックおよびプライベートのネットワーク・アップリンク (Public and Private Network Uplink)」**オプションを選択します。
5. 他の構成オプションをレビューし、**「プロビジョン」**をクリックして、仮想サーバーを作成します。
  ![仮想サーバーの構成](images/solution4/ConfigureVirtualServer.png)

サーバーの作成後に、サーバー・ログイン資格情報が表示されます。 サーバーのパブリック IP アドレスを使用して SSH を介して接続できますが、サーバーにはプライベート・ネットワークを介してアクセスし、パブリック・ネットワークでの SSH アクセスを無効にすることをお勧めします。

1. [こちらのステップ](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network)に従って、仮想マシンを保護し、パブリック・ネットワークでの SSH アクセスを無効にします。
1. ユーザー名、パスワード、およびプライベート IP アドレスを使用して、SSH でサーバーに接続します。
   ```sh
   sudo ssh root@<Private-IP-Address>
   ```
   {: pre}

  サーバーのプライベート IP アドレスおよびパスワードはダッシュボードにあります。
  {:tip}

  ![作成された仮想サーバー](images/solution4/VirtualServerCreated.png)

## Apache、MySQL、および PHP の再インストール

LAMP スタックを最新のセキュリティー・パッチおよびバグ修正で定期的に更新することをお勧めします。 このセクションでは、コマンドを実行して、Ubuntu パッケージ・ソースを更新し、最新バージョンの Apache、MySQL、および PHP を再インストールします。 コマンドの末尾にキャレット (^) を付けます。

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

代替オプションとして、`sudo apt-get update && sudo apt-get dist-upgrade` を使用してすべてのパッケージをアップグレードできます。
{:tip}

## インストールおよび構成の確認

このセクションでは、Apache、MySQL、および PHP が最新であり、Ubuntu イメージで実行されていることを確認します。 また、MySQL の推奨セキュリティー設定を実装します。

1. ブラウザーでパブリック IP アドレスを開いて、Ubuntu を確認します。 Ubuntu のウェルカム・ページが表示されます。
   ![Ubuntu の確認](images/solution4/VerifyUbuntu.png)
2. 以下のコマンドを実行して、ポート 80 を Web トラフィックに使用できることを確認します。
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![ポートの確認](images/solution4/VerifyPort.png)
3. 以下のコマンドを使用して、インストールされている Apache、MySQL、および PHP のバージョンをレビューします。
  ```sh
  apache2 -v
  ```
  {: pre}
  ```sh
  mysql -V
  ```
  {: pre}
  ```sh
   php -v
  ```
   {: pre}
4. 以下のスクリプトを実行して、MySQL データベースを保護します。
  ```sh
  mysql_secure_installation
  ```
  {: pre}
5. MySQL ルート・パスワードを入力し、環境のセキュリティー設定を構成します。 完了したら、`\q` と入力して mysql プロンプトを終了します。
  ```sh
  mysql -u root -p
  ```
  {: pre}

  デフォルトの MySQL ユーザー名およびパスワードは、root および root です。
  {:tip}
6. さらに、以下のコマンドを使用して PHP 情報ページを素早く作成します。
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. 作成した PHP 情報ページを表示します。ブラウザーを開き、`http://{YourPublicIPAddress}/info.php` に移動します。 ご使用の仮想サーバーのパブリック IP アドレスで置き換えます。 以下のイメージのようになります。

![PHP 情報](images/solution4/PHPInfo.png)

### WordPress のインストールと構成

アプリケーションをインストールして LAMP スタックを体験します。 以下のステップでは、通常、Web サイトやブログの作成に使用される、オープン・ソースの WordPress プラットフォームをインストールします。 実動インストールの詳細と設定は、[WordPress の資料](https://codex.wordpress.org/Main_Page)を参照してください。

1. 以下のコマンドを実行して、WordPress をインストールします。
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. MySQL および PHP を使用するように WordPress を構成します。 以下のコマンドを実行して、テキスト・エディターを開き、ファイル `/etc/wordpress/config-localhost.php` を作成します。
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. 以下の行をファイルにコピーし、*yourPassword* をご使用の MySQL データベース・パスワードに置き換えます。他の値は変更せずに使用します。 その後、`Ctrl+X` を使用してファイルを保存して終了します。
   ```php
   <?php
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'wordpress');
   define('DB_PASSWORD', 'yourPassword');
   define('DB_HOST', 'localhost');
   define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
   ?>
   ```
   {: pre}
4. 作業ディレクトリーでテキスト・ファイル `wordpress.sql` を作成して、WordPress データベースを構成します。
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. 以下のコマンドを追加して、*yourPassword* をご使用のデータベース・パスワードで置き換えます。他の値は変更せずに使用します。 その後、ファイルを保存します。
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. 以下のコマンドを実行して、データベースを作成します。
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. コマンドの完了後に、ファイル `wordpress.sql` を削除します。 WordPress インストールを Web サーバーの文書ルートに移動します。
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. WordPress セットアップを完了し、プラットフォームで公開します。 ブラウザーを開き、`http://{yourVMPublicIPAddress}/wordpress` に移動します。 ご使用の VM のパブリック IP アドレスで置き換えます。 以下のイメージのようになります。
   ![実行中の WordPress サイト](images/solution4/WordPressSiteRunning.png)

## ドメインの構成

LAMP サーバーで既存のドメイン名を使用するには、仮想サーバーのパブリック IP アドレスを指すように A レコードを更新します。
サーバーのパブリック IP アドレスは、ダッシュボードで確認できます。

## サーバーのモニターおよび使用量

サーバーの可用性と最適なユーザー・エクスペリエンスを確保するために、すべての実動サーバーでモニタリングが有効になっている必要があります。 このセクションでは、仮想サーバーをモニターするために使用できるオプションを確認し、指定された時刻のサーバーの使用量を調査します。

### サーバー・モニタリング

2 つの基本的なモニタリング・タイプ SERVICE PING および SLOW PING を使用できます。

* **SERVICE PING** は、サーバーの応答時間が 1 秒以下であることを確認します
* **SLOW PING** は、サーバーの応答時間が 5 秒以下であることを確認します

SERVICE PING はデフォルトで追加されているため、以下のステップに従って、SLOW PING モニタリングを追加します。

1. ダッシュボードでデバイスのリストからご使用のサーバーを選択し、**「モニタリング」**タブをクリックします。
  ![Slow Ping モニタリング](images/solution4/SlowPing.png)
2. **「モニターの管理」**をクリックします。
3. **SLOW PING** モニタリング・オプションを追加し、**「モニターの追加 (Add Monitor)」**をクリックします。IP アドレスのパブリック IP アドレスを選択します。
  ![Slow Ping モニタリングの追加](images/solution4/AddSlowPing.png)

  **注**: 構成が同じ重複モニターは許可されていません。 構成ごとに 1 つのモニターのみを作成できます。

割り当てられた時間フレームで応答を受信しなかった場合は、{{site.data.keyword.Bluemix_notm}} アカウントの E メール・アドレスにアラートが送信されます。
  ![2 つのモニタリング](images/solution4/TwoMonitoring.png)

### サーバーの使用量

**「使用量」**タブを選択して、現在のサーバーのメモリーおよび CPU の使用量を確認します。
  ![サーバーの使用量](images/solution4/ServerUsage.png)

## サーバー・セキュリティー

{{site.data.keyword.BluVirtServers}} は、脆弱性スキャンやアドオン・ファイアウォールなど、複数のセキュリティー・オプションを提供します。

### 脆弱性スキャナー

脆弱性スキャナーは、サーバーをスキャンして、サーバーに関連する脆弱性を見つけます。 サーバーに対して脆弱性スキャンを実行するには、以下のステップに従います。

1. ダッシュボードでサーバーを選択し、**「セキュリティー」**タブをクリックします。
2. **「スキャン」**をクリックして、スキャンを開始します。
3. スキャンの完了後に、**「スキャンの完了 (Scan Complete)」**をクリックして、スキャン・レポートを表示します。
  ![2 つのモニタリング](images/solution4/Vulnerabilities.png)
4. 報告された脆弱性をレビューします。
  ![2 つのモニタリング](images/solution4/VulnerabilityResults.png)

### ファイアウォール

サーバーを保護する別の方法としてファイアウォールの追加があります。 ファイアウォールは、必要不可欠なセキュリティー・レイヤーを提供します。望ましくないトラフィックがサーバーに到達しないようにして、攻撃の可能性を減らし、意図した使用目的専用にサーバー・リソースを使用できるようにします。 ファイアウォール・オプションは、サービスを中断せずに、オンデマンドでプロビジョンされます。

ファイアウォールは、Infrastructure パブリック・ネットワーク上のすべてのサーバーでアドオン機能として使用可能です。 オーダー・プロセスの一部として、デバイス固有のハードウェア・ファイアウォールまたはソフトウェア・ファイアウォールを選択して保護を提供できます。 あるいは、専用のファイアウォール・アプライアンスを環境にデプロイし、保護された VLAN に仮想サーバーをデプロイすることもできます。 詳しくは、[ファイアウォール](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started)を参照してください。

## リソースの削除

仮想サーバーを取り外すには、以下の手順で行います。

1. [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices) にログインします。
2. **「デバイス」**メニューから**「デバイス・リスト (Device List)」**を選択します。
3. 取り外す仮想サーバーの**「アクション」**をクリックして、**「Cancel」**を選択します。

## 関連コンテンツ

* [Terraform を使用した LAMP スタックのデプロイ](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
