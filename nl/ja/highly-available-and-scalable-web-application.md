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

# 仮想サーバーを使用してスケーラブルな高可用性 Web アプリを作成する
{: #highly-available-and-scalable-web-application}

より多くのサーバーをアプリケーションに追加することは、追加の負荷を処理するための一般的なパターンです。 アプリケーションの可用性と耐障害性を向上させるためのもう 1 つの主要な側面は、データ・レプリケーションとロード・バランシングにより、アプリケーションを複数のゾーンまたは場所にデプロイすることです。

このチュートリアルでは、以下の作成を含むシナリオについて説明します。

- ダラスの 2 つの Web アプリケーション・サーバー。
- 1 つの場所にある 2 つのサーバーのトラフィックのロード・バランシングを行うための Cloud Load Balancer。
- 1 つの MySQL データベース・サーバー。
- アプリケーション・ファイルとバックアップを保管するための永続ファイル・ストレージ。
- 最初と同じ構成を含む 2 番目の場所を構成した後、いずれかのコピーが失敗した場合に、トラフィックを指す Cloud Internet Services を正常な場所に追加します。

## 達成目標
{: #objectives}

* PHP および MySQL をインストールするための {{site.data.keyword.virtualmachinesshort}} の作成
* アプリケーション・ファイルおよびデータベース・バックアップを永続化するための {{site.data.keyword.filestorage_short}} の使用
* アプリケーション・サーバーへの要求を分散するための {{site.data.keyword.loadbalancer_short}} のプロビジョン
* 耐障害性および可用性を高めるために 2 番目の場所を追加することによるソリューションの拡張

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

このチュートリアルでは、費用が発生する場合があります。 [料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

このアプリケーションは、MySQL データベースを使用した単純な PHP フロントエンド (WordPress ブログ) です。 複数のフロントエンド・サーバーにより要求を処理します。

<p style="text-align: center;">
  ![アーキテクチャー図](images/solution14/Architecture.png)
</p>

1. ユーザーがアプリケーションに接続します。
2. 要求を処理するために、{{site.data.keyword.loadbalancer_short}} により正常なサーバーのいずれかが選択されます。
3. 選択されたサーバーが、共有ファイル・ストレージに保管されたアプリケーション・ファイルにアクセスします。
4. さらに、サーバーによりデータベースから情報がプルされ、最後にユーザーにページがレンダリングされます。
5. 一定の間隔で、データベース・コンテンツがバックアップされます。マスターで障害が発生したときには、スタンバイ・データベース・サーバーを使用できます。

## 始める前に
{: #prereqs}

### VPN アクセスの構成

このチュートリアルでは、ロード・バランサーがアプリケーション・ユーザーにとっての入り口となります。 {{site.data.keyword.virtualmachinesshort}} をパブリック・インターネット上で可視化する必要はありません。このため、これらはプライベート IP アドレスのみを使用してプロビジョンでき、ユーザーは VPN 接続を使用してサーバー上で作業します。

1. [VPN アクセスが有効になっていることを確認します](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     VPN アクセスを有効にするには**マスター・ユーザー**である必要があります。または、アクセスを有効にするようにマスター・ユーザーに連絡してください。
     {:tip}
2. [ユーザー・リスト](https://{DomainName}/iam#/users)でユーザーを選択して、VPN アクセスの資格情報を取得します。
3. [Web インターフェース](https://www.softlayer.com/VPN-Access)から VPN にログインするか、または [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx)、または [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 用の VPN クライアントを使用します。

このステップをスキップして、すべてのサーバーをパブリック・インターネット上で可視化することもできます (ただし、これらのサーバーをプライベートな状態に保持すると、追加のセキュリティー・レベルが提供されます)。これらのサーバーをパブリックにするには、{{site.data.keyword.virtualmachinesshort}} をプロビジョンする際に、**「パブリックおよびプライベートのネットワーク・アップリンク (Public and Private Network Uplink)」**を選択します。
{: tip}

### アカウントの許可の確認

インフラストラクチャー・マスター・ユーザーに連絡して、以下の許可を取得します。
- **ネットワーク**: これにより、**「パブリックおよびプライベートのネットワーク・アップリンク (Public and Private Network Uplink)」**を使用して {{site.data.keyword.virtualmachinesshort}} を作成できます (この許可は、サーバーへの接続に VPN を使用する場合には必要ありません)。

## データベース用の 1 つのサーバーのプロビジョン
{: #database_server}

このセクションでは、マスター・データベースとして動作する 1 つのサーバーを構成します。

1. {{site.data.keyword.Bluemix}} コンソールでカタログに移動し、「インフラストラクチャー」セクションから「[{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)」を選択します。
2. **「パブリック Virtual Server」**を選択してから、**「作成」**をクリックします。
3. 以下のようにサーバーを構成します。
   - **「名前」**を **db1** に設定します。
   - サーバーをプロビジョンする場所を選択します。 **このチュートリアルで作成されるその他すべてのサーバーおよびリソースは、同じ場所に作成する必要があります。**
   - **Ubuntu Minimal** イメージを選択します。
   - デフォルトのコンピュート・フレーバーを保持します。 このチュートリアルは最小フレーバーを使用してテストされていますが、任意のフレーバーで動作します。
   - **「取り付けられたストレージ・ディスク」**の下で、25GB のブート・ディスクを選択します。
   - **「ネットワーク・インターフェース」**の下で、**「100Mbps プライベート・ネットワーク・アップリンク (100Mbps Private Network Uplink)」**オプションを選択します。

     VPN アクセスを構成しなかった場合は、**「100Mbps パブリックおよびプライベートのネットワーク・アップリンク (100Mbps Public and Private Network Uplink)」**オプションを選択します。
     {: tip}
   - その他の構成オプションを確認し、**「プロビジョン」**をクリックしてサーバーをプロビジョンします。

      ![仮想サーバーの構成](images/solution14/db-server.png)

   注: プロビジョニング・プロセスでサーバーが使用できるようになるには、2 分から 5 分かかる場合があります。 サーバーの作成後、サーバー資格情報は、**「デバイス」>「デバイス・リスト (Device List)」**の下のサーバー詳細ページに配置されます。サーバーへの SSH 接続には、サーバーのプライベートまたはパブリックの IP アドレス、ユーザー名、およびパスワードが必要です (デバイス名の横にある矢印をクリックします)。
   {: tip}

## MySQL のインストールおよび構成
{: #mysql}

サーバーはデータベースに付属していません。 このセクションでは、サーバー上に MySQL をインストールします。

### MySQL のインストール

1. SSH を使用して、サーバーに接続します。
   ```sh
   ssh root@<Private-OR-Public-IP-Address>
   ```

   VPN クライアントに接続するときは、必ず、仮想サーバーの**場所**に基づいた正しい[サイト・アドレス](https://www.softlayer.com/VPN-Access)を使用してください。
   {:tip}
2. MySQL をインストールします。
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   パスワードの入力を求めるプロンプトが表示される場合があります。 表示されるコンソールの指示を最後まで読みます。
   {:tip}
3. 次のスクリプトを実行すると、セキュアな MySQL データベースに役立ちます。
   ```sh
   mysql_secure_installation
   ```

   いくつかのオプションについてプロンプトが表示される場合があります。 要件に基づいて、適切に選択してください。
   {:tip}

### アプリケーション用のデータベースの作成

1. MySQL にログインし、`wordpress` というデータベースを作成します。
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. データベースへのアクセス権限を付与し、database-username および database-password を以前に設定したもので置き換えます。

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. 次を使用して、データベースが作成されたかどうかを確認します。

   ```sh
   show databases;
   ```

4. 次を使用して、データベースを終了します。

   ```sh
   exit
   ```

5. データベース名、ユーザー、およびパスワードをメモします。 これらは、アプリケーション・サーバーの構成時に必要となります。

### ネットワーク上の他のサーバーからの MySQL サーバーの可視化

デフォルトでは、MySQL はローカル・インターフェースのみを listen します。アプリケーション・サーバーではデータベースに接続する必要があるため、プライベート・ネットワーク・インターフェースを listen するように、MySQL 構成を変更する必要があります。

1. `nano /etc/mysql/my.cnf` を使用して my.cnf ファイルを編集し、次の行を追加します。
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. Ctrl+X を使用して、ファイルを終了および保存します。

3. MySQL を再始動します。

   ```sh
   systemctl restart mysql
   ```

4. 次のコマンドを実行して、MySQL がすべてのインターフェースを listen していることを確認します。
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## データベース・バックアップ用のファイル・ストレージの作成
{: #database_backup}

MySQL に関しては、バックアップを完了して保管できる数多くの方法があります。 このチュートリアルでは、crontab エントリーを使用して、データベース・コンテンツをディスクにダンプします。 バックアップ・ファイルはファイル・ストレージに保管されます。 これは、明らかに単純なバックアップ・メカニズムです。独自の MySQL データベース・サーバーを実稼働環境で管理する予定の場合は、[MySQL 資料で説明されているいずれかのバックアップ戦略を実装する](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html)必要があります。

### ファイル・ストレージの作成
{: #create_for_backup}

1. {{site.data.keyword.Bluemix}} コンソールでカタログに移動し、[「{{site.data.keyword.filestorage_short}}」](https://{DomainName}/catalog/infrastructure/file-storage)を選択します。
2. **「作成」**をクリックします。
3. 以下のようにサービスを構成します。
   - **「ストレージ・タイプ」**を**「エンデュランス (Endurance)」**に設定します。
   - データベース・サーバーを作成した場所と同じになるように、**「場所」**を選択します。
   - 課金方法を選択します。
   - **「ストレージ・パッケージ (Storage Packages)」**の下で、**「0.25 IOPS/GB (0.25 IOPS/GB)」**を選択します。
   - **「ストレージ・サイズ」**の下で、**「20GB (20GB)」**を選択します。
   - **「スナップショット・スペースのサイズ (Snapshot Space Size)」**は**「0GB (0GB)」**のまま保持します。
   - 「続行」をクリックして、サービスを作成します。

### データベース・サーバーにファイル・ストレージの使用を許可する

仮想サーバーにファイル・ストレージをマウントするには、許可が必要です。

1. [既存の項目のリスト](https://{DomainName}/classic/storage/file)から、新しく作成したファイル・ストレージを選択します。
2. **「許可されたホスト (Authorized Hosts)」**の下で**「ホストの許可 (Authorize Host)」**をクリックし、仮想 (データベース) サーバーを選択します (**「デバイス」**を選択して、「デバイス・タイプ」として「仮想サーバー」を選択し、サーバーの名前を入力します)。

### データベース・バックアップ用のファイル・ストレージのマウント

ファイル・ストレージは、NFS ドライブとして仮想サーバーにマウントできます。

1. NFS クライアント・ライブラリーをインストールします。
   ```sh
   apt-get -y install nfs-common
   ```

2. 次のコマンドを実行して、`/etc/systemd/system/mnt-database.mount` というファイルを作成します。
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. 次を使用して、mnt-database.mount を編集します。
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. 次のコンテンツを mnt-database.mount ファイルに追加し、`What` の `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` をファイル・ストレージの**マウント・ポイント** (*fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01* など) で置き換えます。 **マウント・ポイント** url は、作成したファイル・ストレージ・サービスの下で取得できます。
   ```
   [Unit]
   Description = Mount for Container Storage

   [Mount]
   What=CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT
   Where=/mnt/database
   Type=nfs
   Options=vers=3,sec=sys,noauto

   [Install]
   WantedBy = multi-user.target
   ```
   Ctrl+X を使用して、ナノ・ウィンドウを保存および終了します。
   {: tip}

5. マウント・ポイントを作成します。
  ```sh
  mkdir /mnt/database
  ```

6. ストレージをマウントします。
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. マウントが正常に行われたかどうかを確認します。
   ```sh
   マウント
   ```
   最後の行に、ファイル・ストレージ・マウントがリストされます。 そうでない場合は、`journalctl -xe` を使用して、マウント操作をデバッグします。
   {: tip}

### 一定の間隔でバックアップをセットアップする

1. `CHANGE_ME` を以前に指定したデータベース・パスワードで置き換えて次のコマンドを使用し、`/root/dbbackup.sh` シェル・スクリプトを作成します (`touch` および `nano` を使用します)。
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip>/mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. ファイルが実行可能であることを確認してください。
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. crontab を編集します。
   ```sh
   crontab -e
   ```
4. バックアップが毎日午後 11 時に実行されるようにするには、コンテンツを次のように設定し、ファイルを保存してエディターを閉じます。
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## PHP アプリケーション用の 2 つのサーバーのプロビジョン
{: #app_servers}

このセクションでは、2 つの Web アプリケーション・サーバーを作成します。

1. {{site.data.keyword.Bluemix}} コンソールでカタログに移動し、「インフラストラクチャー」セクションから「[{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)」サービスを選択します。
2. **「パブリック Virtual Server」**を選択してから、**「作成」**をクリックします。
3. 以下のようにサーバーを構成します。
   - **「名前」**を **app1** に設定します。
   - データベース・サーバーをプロビジョンしたのと同じ場所を選択します。
   - **Ubuntu Minimal** イメージを選択します。
   - デフォルトのコンピュート・フレーバーを保持します。 
   - **「取り付けられたストレージ・ディスク」**の下で、ブート・ディスクとして「25GB (25GB)」を選択します。
   - **「ネットワーク・インターフェース」**の下で、**「100Mbps プライベート・ネットワーク・アップリンク (100Mbps Private Network Uplink)」**オプションを選択します。

     VPN アクセスを構成しなかった場合は、**「100Mbps パブリックおよびプライベートのネットワーク・アップリンク (100Mbps Public and Private Network Uplink)」**オプションを選択します。
     {: tip}
   - その他の構成オプションを確認し、**「プロビジョン」**をクリックしてサーバーをプロビジョンします。
     ![仮想サーバーの構成](images/solution14/db-server.png)
4. ステップ 1 から 3 を繰り返して、**app2** という別の仮想サーバーをプロビジョンします。

## アプリケーション・サーバー間でファイルを共有するためのファイル・ストレージの作成
{: shared_storage}

このファイル・ストレージを使用して、*app1* サーバーと *app2* サーバーの間でアプリケーション・ファイルを共有します。

### ファイル・ストレージの作成
{: #create_for_sharing}

1. {{site.data.keyword.Bluemix}} コンソールでカタログに移動し、[「{{site.data.keyword.filestorage_short}}」](https://{DomainName}/catalog/infrastructure/file-storage)を選択します。
2. **「作成」**をクリックします。
3. 以下のようにサービスを構成します。
   - **「ストレージ・タイプ」**を**「エンデュランス (Endurance)」**に設定します。
   - アプリケーション・サーバーを作成した場所と同じになるように、**「場所」**を選択します。
   - 課金方法を選択します。
   - **「ストレージ・パッケージ (Storage Packages)」**の下で、**「2 IOPS/GB (2 IOPS/GB)」**を選択します。
   - **「ストレージ・サイズ」**の下で、**「20GB (20GB)」**を選択します。
   - **「スナップショット・スペースのサイズ (Snapshot Space Size)」**の下で、**「20GB (20GB)」**を選択します。
   - 「続行」をクリックして、サービスを作成します。

### 定期的なスナップショットの構成

[スナップショット](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots)は、パフォーマンスへの影響なくデータを保護できる便利なオプションとなります。さらに、スナップショットを別のデータ・センターに複製することもできます。

1. [既存の項目のリスト](https://{DomainName}/classic/storage/file)から、ファイル・ストレージを選択します。
2. **「スナップショット・スケジュール (Snapshot Schedules)」**の下で、スナップショット・スケジュールを編集します。スケジュールは、以下のように定義できます。
   1. 毎時のスナップショットを追加し、分を 30 に設定して、最新の 24 個のスナップショットを保持します。
   2. 毎日のスナップショットを追加し、時間を午後 11 時に設定して、最新の 7 個のスナップショットを保持します。
   3. 毎週のスナップショットを追加し、時間を午前 1 字に設定して、最新の 4 個のスナップショットを保持して「保存」をクリックします。
      ![バックアップ・スナップショット](images/solution14/snapshots.png)

### アプリケーション・サーバーにファイル・ストレージの使用を許可する

1. **「許可されたホスト (Authorized Hosts)」**の下で**「ホストの許可 (Authorize Host)」**をクリックし、このファイル・ストレージを使用することをアプリケーション・サーバー (app1 および app2) に許可します。

### ファイル・ストレージのマウント

各アプリケーション・サーバー (app1 および app2) 上で、以下のステップを繰り返します。

1. NFS クライアント・ライブラリーをインストールします。
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. `touch /etc/systemd/system/mnt-www.mount` を使用してファイルを作成し、以下のコンテンツを持つ `nano /etc/systemd/system/mnt-www.mount` を使用して編集します。このとき、`What` の `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` をファイル・ストレージの**マウント・ポイント** (*fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01* など) に置き換えます。マウント・ポイントは、[ファイル・ストレージ・ボリュームのリスト](https://{DomainName}/classic/storage/file)にあります。
   ```
   [Unit]
   Description = Mount for Container Storage

   [Mount]
   What=CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT
   Where=/mnt/www
   Type=nfs
   Options=vers=3,sec=sys,noauto

   [Install]
   WantedBy = multi-user.target
   ```
3. マウント・ポイントを作成します。
   ```sh
   mkdir /mnt/www
   ```
4. ストレージをマウントします。
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. マウントが正常に行われたかどうかを確認します。
   ```sh
   マウント
   ```
   最後の行に、ファイル・ストレージ・マウントがリストされます。 そうでない場合は、`journalctl -xe` を使用して、マウント操作をデバッグします。
   {: tip}

最終的に、サーバーの構成に関連するすべてのステップは、[プロビジョニング・スクリプト](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script)を使用するか、または[イメージをキャプチャーする](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template)ことで自動化できます。
{: tip}

## アプリケーション・サーバー上の PHP アプリケーションのインストールおよび構成
{: #php_application}

このチュートリアルでは、WordPress ブログをセットアップします。 すべての WordPress ファイルは、両方のアプリケーション・サーバーからアクセスできるように、共有ファイル・ストレージ上にインストールされます。 WordPress をインストールする前に、Web サーバーおよび PHP ランタイムを構成する必要があります。

### nginx および PHP のインストール

各アプリケーション・サーバー上で、以下のステップを繰り返します。

1. nginx をインストールします。
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. PHP および mysql クライアントをインストールします。
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. PHP サービスおよび nginx を停止します。
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. `nano /etc/nginx/sites-available/default` を使用して、コンテンツを次のように置き換えます。
   ```sh
   server {
          listen 80 default_server;
          listen [::]:80 default_server;

          root /mnt/www/html;

          index index.php;

          server_name _;

          location = /favicon.ico {
                  log_not_found off;
                  access_log off;
          }

          location = /robots.txt {
                  allow all;
                  log_not_found off;
                  access_log off;
          }

          location / {
                  # following https://codex.wordpress.org/Nginx
                  try_files $uri $uri/ /index.php?$args;
          }

          # pass the PHP scripts to the local FastCGI server
          location ~ \.php$ {
                  include snippets/fastcgi-php.conf;
                  fastcgi_pass unix:/run/php/php7.2-fpm.sock;
          }

          location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                  expires max;
                  log_not_found off;
          }

          # deny access to .htaccess files, if Apache's document root
          # concurs with nginx's one
          location ~ /\.ht {
                  deny all;
          }
   }
   ```
5. 次を使用して、2 つのうちいずれかのアプリ・サーバー上の `/mnt/www` ディレクトリー内に、`html` フォルダーを作成します。
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### WordPress のインストールと構成

WordPress はファイル・ストレージ・マウント上にインストールされるため、いずれかのサーバー上で以下のステップを実行することのみが必要です。**app1** で実行してみましょう。

1. WordPress インストール・ファイルを取得します。

   アプリケーション・サーバーにパブリック・ネットワーク・リンクが含まれる場合は、仮想サーバー内から直接 WordPress ファイルをダウンロードできます。

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   仮想サーバーにプライベート・ネットワーク・リンクのみが含まれる場合は、インターネット・アクセスを使用して別のマシンからインストール・ファイルを取得し、これらのファイルを仮想サーバーにコピーする必要があります。 WordPress インストール・ファイルを https://wordpress.org/latest.tar.gz から取得した場合、これらのファイルを `scp` を使用して仮想サーバーにコピーできます。

   ```sh
   scp latest.tar.gz root@PRIVATE_IP_ADDRESS_OF_THE_SERVER:/tmp
   ```
   `latest` を Wordpress Web サイトからダウンロードしたファイル名で置き換えます。
   {: tip}

   次に、仮想サーバーに ssh で接続し、`tmp` ディレクトリーに変更します。

   ```sh
   cd /tmp
   ```

2. インストール・ファイルを抽出します。

   ```
   tar xzvf latest.tar.gz
   ```

3. WordPress ファイルを準備します。
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. ファイルを共有ファイル・ストレージにコピーします。
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. 許可を設定します。
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -exec chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. 次の Web サービスを呼び出して、`nano` を使用して結果を `/mnt/www/html/wp-config.php` に注入します。
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   仮想サーバーにパブリック・ネットワーク・リンクが含まれない場合は、単に、Web ブラウザーから https://api.wordpress.org/secret-key/1.1/salt/ を開くことができます。

7. `nano /mnt/www/html/wp-config.php` を使用してデータベース資格情報を設定し、データベース資格情報を更新します。

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   WordPress が構成されました。 インストールを完了するには、WordPress ユーザー・インターフェースにアクセスする必要があります。

両方のアプリケーション・サーバー上で Web サーバーおよび PHP ランタイムを開始します。
7. 次のコマンドを実行して、サービスを開始します。

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

WordPress インストール (`http://YourAppServerIPAddress/`) にプライベート IP アドレス (VPN 接続を介して接続している場合) を使用するか、*app1* または *app2* のパブリック IP アドレスを使用してアクセスします。
![仮想サーバーの構成](images/solution14/wordpress.png)

アプリケーション・サーバーをプライベート・ネットワーク・リンクのみを使用して構成した場合、WordPress のプラグイン、テーマ、またはアップグレードを WordPress 管理コンソールから直接インストールすることはできません。 WordPress ユーザー・インターフェースを介してファイルをアップロードする必要があります。
{: tip}

## アプリケーション・サーバーの前に 1 つのロード・バランサー・サーバーをプロビジョンする
{: #load_balancer}

この時点で、個別の IP アドレスを含む 2 つのアプリケーション・サーバーを設定しました。 プライベート・ネットワーク・アップリンクのみをプロビジョンすることを選択した場合、これらのサーバーは、パブリック・インターネット上で可視化されないことがあります。これらのサーバーの前にロード・バランサーを追加することで、アプリケーションはパブリックになります。また、ロード・バランサーにより、基礎となるインフラストラクチャーがユーザーから隠されます。ロード・バランサーにより、アプリケーション・サーバーの正常性がモニターされ、着信要求は正常なサーバーにディスパッチされます。

1. カタログに移動し、[{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer)を作成します。
2. **「プラン」**ステップで、*app1* および *app2* と同じデータ・センターを選択します。
3. **「ネットワーク設定 (Network Settings)」**で、以下のようにします。
   1. *app1* および *app2* がプロビジョンされているのと同じサブネットを選択します。
   2. ロード・バランサーのパブリック IP 用のデフォルトの IBM システム・プールを使用します。
4. **「基本」**で、以下のようにします。
   1. ロード・バランサーに名前 (**app-lb-1** など) を付けます。
   2. デフォルトのプロトコル構成を保持します。デフォルトでは、ロード・バランサーは HTTP 用に構成されています。
      SSL プロトコルは、独自の証明書でサポートされます。 [ロード・バランサーでの SSL 証明書のインポート](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates)を参照してください。
      {: tip}
5. **「サーバー・インスタンス」**で、*app1* および *app2* の各サーバーを追加します。
6. 確認して作成し、ウィザードを完了します。

### WordPress 構成をロード・バランサー URL を使用するように変更する

WordPress 構成を、ロード・バランサー・アドレスを使用するように変更する必要があります。 実際、WordPress で保持されている参照先は[ブログ URL であり、この場所がページ内に注入されています](https://codex.wordpress.org/Settings_General_Screen)。 この設定を変更しない場合、WordPress はユーザーをバックエンド・サーバーに直接リダイレクトしてロード・バランサーをバイパスするか、サーバーにプライベート IP アドレスのみが設定されている場合にはまったく機能しなくなります。

1. ロード・バランサー・アドレスをその詳細ページで見つけます。 作成したロード・バランサーは、[Network / Load Balancing / Local](https://{DomainName}/classic/network/loadbalancing/cloud) にあります。

   また、DNS 構成内のロード・バランサー・アドレスを指す CNAME レコードを追加して、独自のドメイン名をロード・バランサーで使用することもできます。
   {: tip}
2. *app1* または *app2* の URL を介して、WordPress ブログに管理者としてログインします。
3. 「Settings」/「General」で、「Wordpress Address (URL)」と「Site Address (URL)」の両方をロード・バランサー・アドレスに設定します。
4. 設定を保存します。WordPress はロード・バランサー・アドレスにリダイレクトされます。
   DNS 伝搬が原因で、ロード・バランサー・アドレスがアクティブになるにはしばらく時間がかかることがあります。
   {: tip}

### ロード・バランサーの動作のテスト

ロード・バランサーは、サーバーの正常性を検査し、ユーザーを正常なサーバーにのみリダイレクトするように構成されています。 ロード・バランサーが機能する仕組みを理解するために、以下のことを実行できます。

1. 次を使用して、*app1* と *app2* の両方で nginx ログを監視します。
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   サーバーの正常性を検査するために、ロード・バランサーから定期的な ping が既に行われています。
   {: tip}
2. ロード・バランサー・アドレスを介して WordPress にアクセスし、必ずページのハード再ロードを強制するようにしてください。 nginx ログで、*app1* と *app2* の両方により、ページのコンテンツが提供されていることに注意してください。 ロード・バランサーにより、トラフィックは予想どおりに両方のサーバーにリダイレクトされています。

3. *app1* で nginx を停止します。
   ```sh
   systemctl nginx stop
   ```

4. しばらくしてから WordPress ページを再ロードすると、すべてのヒットが *app2* に送信されていることがわかります。

5. *app2* で nginx を停止します。

6. WordPress ページを再ロードします。 正常なサーバーが存在しないため、ロード・バランサーはエラーを返します。

7. *app1* で nginx を再始動します。
   ```sh
   systemctl nginx start
   ```

8. ロード・バランサーで *app1* が正常であるとして検出されると、トラフィックはこのサーバーにリダイレクトされます。

## 2 番目の場所を使用したソリューションの拡張 (オプション)
{: #secondregion}

耐障害性および可用性を高めるために、2 番目の場所を使用してインフラストラクチャー・セットアップを拡張し、アプリケーションが 2 つの場所で実行されるようにすることができます。

2 番目の場所をデプロイすると、アーキテクチャーは次のようになります。

<p style="text-align: center;">

  ![アーキテクチャー図](images/solution14/Architecture2.png)
</p>

1. ユーザーが、IBM Cloud Internet Services (CIS) を介してアプリケーションにアクセスします。
2. CIS によってトラフィックが正常な場所にルーティングされます。
3. 場所内で、ロード・バランサーがトラフィックをサーバーにリダイレクトします。
4. アプリケーションがデータベースにアクセスします。
5. アプリケーションがメディア資産を保管し、ファイル・ストレージからメディア資産を取得します。

このアーキテクチャーを実装するには、場所 2 で以下を実行することが必要になる場合があります。

- 前述のすべてのステップを新しい場所で繰り返します。
- 場所間で、2 つの MySQL サーバー間のデータベース・レプリケーションをセットアップします。
- [他のこのチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)で説明されているように、場所間で正常なサーバーにトラフィックを分散するよう IBM Cloud Internet Services を構成します。

## リソースの削除
{: #removeresources}

1. ロード・バランサーを削除します。
2. *db1*、*app1*、および *app2* をキャンセルします。
3. 2 つのファイル・ストレージ・サービスを削除します。
4. 2 番目の場所を構成した場合は、すべてのリソースおよび Cloud Internet Services を削除します。

## 関連コンテンツ
{: #related}

- アプリケーションによって提供される静的コンテンツには、コンテンツ配信ネットワークをロード・バランサーの前に置き、バックエンド・サーバーへの負荷を軽減することが役立つ場合があります。コンテンツ配信ネットワークを実装するチュートリアルについては、[CDN を使用した静的ファイルの配信の高速化 - オブジェクト・ストレージ](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)を参照してください。
- このチュートリアルでは 2 つのサーバーをプロビジョンしますが、追加の負荷を処理するためにより多くのサーバーを自動的に追加できます。[オートスケール](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale)を使用すると、ビジネス・アプリケーションをサポートするために、仮想サーバーの追加または削除に関連する手動によるスケーリング・プロセスを自動化できます。
- 可用性および災害復旧のオプションを向上させるために、コンテンツの[定期的な自動スナップショット](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots)および別のデータ・センターへの[レプリケーション](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication)を実行するよう、ファイル・ストレージを構成できます。
