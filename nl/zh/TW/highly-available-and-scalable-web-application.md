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

# 使用虛擬伺服器建置具備高可用性和高可擴充的 Web 應用程式
{: #highly-available-and-scalable-web-application}

處理額外負載的常見模式是向應用程式新增更多伺服器。另一個提高應用程式可用性和備援的關鍵方面是使用資料抄寫和負載平衡，將應用程式部署到多個區域或位置。

本指導教學會引導您實現建立以下內容的情境：

- 位於達拉斯的兩個 Web 應用程式伺服器
- 雲端負載平衡器，用於對一個位置中的兩部伺服器進行資料流量負載平衡。
- 一部 MySQL 資料庫伺服器。
- 一個可延續檔案儲存空間，用於儲存應用程式檔案和備份。
- 使用與第一個位置相同的配置來配置第二個位置，然後新增 Cloud Internet Services，以在一個副本失敗時將資料流量指向性能良好的位置。

## 目標
{: #objectives}

* 建立 {{site.data.keyword.virtualmachinesshort}} 來安裝 PHP 和 MySQL
* 使用 {{site.data.keyword.filestorage_short}} 來持續保存應用程式檔案和資料庫備份
* 佈建 {{site.data.keyword.loadbalancer_short}} 以將要求分散給應用程式伺服器
* 藉由新增第二個位置來延伸解決方案，以提高備援和可用性

## 使用的服務
{: #services}

本指導教學使用下列運行環境及服務：
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{: #architecture}

應用程式是一個簡單的 PHP 前端（Wordpress 部落格），使用的是 MySQL 資料庫。有多部前端伺服器可處理要求。

<p style="text-align: center;">
  ![架構圖](images/solution14/Architecture.png)
</p>

1. 使用者連接至應用程式。
2. {{site.data.keyword.loadbalancer_short}} 選取其中一個性能良好的伺服器來處理要求。
3. 所選伺服器存取儲存空間在共用檔案儲存空間上的應用程式檔案。
4. 該伺服器還從資料庫中取回資訊，並最終將頁面呈現給使用者。
5. 資料庫內容將定期備份。主伺服器故障後，即可使用備用資料庫伺服器。

## 開始之前
{: #prereqs}

### 配置 VPN 存取

在本指導教學中，負載平衡器是面向應用程式使用者的前端。在公用網際網路上不需要看到 {{site.data.keyword.virtualmachinesshort}}。因此，可以僅為其佈建專用 IP 位址，您將使用 VPN 連線在伺服器上工作。

1. [確定 VPN 存取已啟用](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     您應該是**主要使用者**，才能啟用 VPN 存取，或是請與主要使用者聯絡以取得存取權。
     {:tip}
2. 選取[使用者清單](https://{DomainName}/iam#/users)中的使用者，以取得 VPN 存取認證。
3. 透過 [Web 介面](https://www.softlayer.com/VPN-Access)登入 VPN，或使用 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 或 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 的 VPN 用戶端。

您可以選擇跳過此步驟，並使所有伺服器均可在公用網際網路上查看（但使其保持專用可提供額外的安全層次）。若要使其可供大眾使用，請在佈建 {{site.data.keyword.virtualmachinesshort}} 時選取**公用和專用網路上行鏈路**。
{: tip}

### 檢查帳戶許可權

請與您的基礎架構主要使用者聯絡，以取得下列許可權：
- **網路**許可權，以便可以建立使用**公用和專用網路上行鏈路**的 {{site.data.keyword.virtualmachinesshort}}（如果使用 VPN 連接至伺服器，則無需此許可權）

## 為資料庫佈建一部伺服器
{: #database_server}

在本節中，您將配置一部伺服器以充當主資料庫。

1. 移至 {{site.data.keyword.Bluemix}} 主控台中的「型錄」，然後從「基礎架構」區段中選取 [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)。
2. 選取**公用虛擬伺服器**，然後按一下**建立**。
3. 使用下列各項配置伺服器：
   - 將**名稱**設定為 **db1**。
   - 選取要佈建伺服器的位置。**本指導教學中建立的其他所有伺服器和資源都需要在同一位置中進行建立。**
   - 選取 **Ubuntu Minimal** 映像檔。
   - 保留預設運算特性。本指導教學已使用最小特性進行過測試，但應該能夠適用於任何特性。
   - 在**連接的儲存空間磁碟**下，選取 25 GB 開機磁碟。
   - 在**網路介面**下，選取 **100Mbps 專用網路上行鏈路**選項。

     如果您未配置 VPN 存取，請選取 **100Mbps 公用及專用網路上行鏈路**選項。
     {: tip}
   - 檢閱其他配置選項，然後按一下**佈建**以佈建伺服器。

      ![配置虛擬伺服器](images/solution14/db-server.png)

   附註：佈建處理程序可能需要 2 到 5 分鐘，伺服器才可供使用。建立伺服器後，您可在**裝置 > 裝置清單**下的伺服器詳細資料頁面中找到伺服器認證。若要透過 SSH 進入伺服器，需要伺服器專用或公用 IP 位址、使用者名稱和密碼（按一下裝置名稱旁邊的箭頭）。
   {: tip}

## 安裝和配置 MySQL
{: #mysql}

伺服器未隨附資料庫。在本節中，您將在伺服器上安裝 MySQL。

### 安裝 MySQL

1. 使用 SSH 連接至伺服器：
   ```sh
   ssh root@<Private-OR-Public-IP-Address>
   ```

   請務必根據虛擬伺服器的**位置**，使用正確的[網址](https://www.softlayer.com/VPN-Access)連接至 VPN 用戶端。
   {:tip}
2. 安裝 MySQL：
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   系統可能會提示您輸入密碼。請閱讀主控台上顯示的指示。
   {:tip}
3. 執行下列 Script 來協助保護 MySQL 資料庫安全：
   ```sh
   mysql_secure_installation
   ```

   系統可能會提示您從幾個選項中進行選擇。請根據需求進行明智的選擇。
   {:tip}

### 為應用程式建立資料庫

1. 登入 MySQL，然後建立名為 `wordpress` 的資料庫：
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. 授權對該資料庫的存取權，將 database-username 和 database-password 取代為您稍早設定的內容。

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. 使用下列指令檢查是否建立該資料庫：

   ```sh
   show databases;
   ```

4. 使用下列指令從該資料庫結束：

   ```sh
   exit
   ```

5. 記下資料庫名稱、使用者和密碼。配置應用程式伺服器時，將需要這些資訊。

### 讓網路上的其他伺服器可以看到 MySQL 伺服器

依預設，MySQL 僅接聽本端介面。應用程式伺服器將需要連接至資料庫，因此 MySQL 配置需要變更為接聽專用網路介面。

1. 使用 `nano /etc/mysql/my.cnf` 編輯 my.cnf 檔案，並新增下列各行：
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. 使用 Ctrl+X 結束並儲存檔案。

3. 重新啟動 MySQL：

   ```sh
   systemctl restart mysql
   ```

4. 藉由執行下列指令，確認 MySQL 是否正在接聽所有介面：
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## 建立用於資料庫備份的檔案儲存空間
{: #database_backup}

對於 MySQL，有許多方法可以執行和儲存備份。本指導教學使用 crontab 項目將資料庫內容傾出到磁碟。備份檔將儲存空間在檔案儲存空間中。顯然，這是一個簡單的備份機制。如果您計劃在正式作業環境中管理自己的 MySQL 資料庫伺服器，則需要[實作 MySQL 文件中說明的其中一個備份策略](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html)。

### 建立檔案儲存空間
{: #create_for_backup}

1. 在 {{site.data.keyword.Bluemix}} 主控台移至型錄，然後選取 [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)。
2. 按一下**建立**。
3. 使用下列各項配置服務：
   - 將**儲存空間類型**設定為**耐久性**。
   - 選取在其中建立資料庫伺服器的**位置**
   - 選取計費方法。
   - 在**儲存空間套件**下，選取 **0.25 IOPS/GB**
   - 在**儲存空間大小**下，選取 **20GB**
   - 將 **Snapshot 空間大小**保留為 **0GB**
   - 按一下「繼續」，以建立服務。

### 授權資料庫伺服器使用檔案儲存空間

虛擬伺服器需要先獲得授權，才能裝載檔案儲存空間。

1. 從[現有項目清單](https://{DomainName}/classic/storage/file)中選取新建立的檔案儲存空間。
2. 在**已授權主機**下，按一下**授權主機**，然後選取虛擬（資料庫）伺服器（選取**裝置**> 將「虛擬伺服器」作為「裝置類型」> 鍵入伺服器的名稱）。

### 裝載資料庫備份的檔案儲存空間

「檔案儲存空間」可以作為 NFS 磁碟機裝載到虛擬伺服器中。

1. 安裝 NFS 用戶端程式庫：
   ```sh
   apt-get -y install nfs-common
   ```

2. 藉由執行下列指令，建立名為 `/etc/systemd/system/mnt-database.mount` 的檔案：
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. 使用下列指令編輯 mnt-database.mount：
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. 將以下內容新增到 mnt-database.mount 檔案中，並將 `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` 的值 `What` 取代為檔案儲存空間的**裝載點**（例如，*fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*）。可以在建立的 File Storage 服務下取得**裝載點** URL。
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
   使用 Ctrl+X 來儲存並結束 nano 視窗
   {: tip}

5. 建立裝載點
  ```sh
  mkdir /mnt/database
  ```

6. 裝載儲存空間
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. 檢查裝載是否已順利完成
   ```sh
   mount
   ```
   最後幾行應該列出 File Storage 裝載。若不是如此，請使用 `journalctl -xe` 進行裝載作業的除錯。
   {: tip}

### 設定定期備份

1. 使用下列指令，藉由將 `CHANGE_ME` 取代為稍早指定的資料庫密碼，建立 `/root/dbbackup.sh` Shell Script（使用 `touch` 和 `nano`）：
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip > /mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. 確保該檔案是執行檔
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. 編輯 crontab
   ```sh
   crontab -e
   ```
4. 若要在每天晚上 11 點執行備份，請將內容設定為如下內容，然後儲存檔案並關閉編輯器
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## 為 PHP 應用程式佈建兩部伺服器
{: #app_servers}

在本節中，您將建立兩個 Web 應用程式伺服器。

1. 移至 {{site.data.keyword.Bluemix}} 主控台中的「型錄」，然後從「基礎架構」區段中選取 [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) 服務。
2. 選取**公用虛擬伺服器**，然後按一下**建立**。
3. 使用下列各項配置伺服器：
   - 將**名稱**設定為 **app1**。
   - 選取在其中佈建資料庫伺服器的位置。
   - 選取 **Ubuntu Minimal** 映像檔。
   - 保留預設運算特性。
   - 在**連接的儲存空間磁碟**下，選取 25 GB 作為開機磁碟。
   - 在**網路介面**下，選取 **100Mbps 專用網路上行鏈路**選項。

     如果您未配置 VPN 存取，請選取 **100Mbps 公用及專用網路上行鏈路**選項。
     {: tip}
   - 檢閱其他配置選項，然後按一下**佈建**以佈建伺服器。
     ![配置虛擬伺服器](images/solution14/db-server.png)
4. 重複步驟 1-3 以佈建另一部名為 **app2** 的虛擬伺服器。

## 建立檔案儲存空間以用於在應用程式伺服器之間共用檔案
{: shared_storage}

此檔案儲存空間用於在 *app1* 與 *app2* 伺服器之間共用應用程式檔案。

### 建立檔案儲存空間
{: #create_for_sharing}

1. 在 {{site.data.keyword.Bluemix}} 主控台移至型錄，然後選取 [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)。
2. 按一下**建立**。
3. 使用下列各項配置服務：
   - 將**儲存空間類型**設定為**耐久性**。
   - 選取在其中建立應用程式伺服器的**位置**
   - 選取計費方法。
   - 在**儲存空間套件**下，選取 **2 IOPS/GB**
   - 在**儲存空間大小**下，選取 **20GB**
   - 在 **Snapshot 空間大小**下，選取 **20GB**
   - 按一下「繼續」，以建立服務。

### 配置定期 Snapshot

[Snapshot](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) 是一個方便的選項，可用於保護資料，而不會影響效能。此外，還可以將 Snapshot 抄寫到其他資料中心。

1. 從[現有項目清單](https://{DomainName}/classic/storage/file)中選取檔案儲存空間。
2. 在**Snapshot 排程**下，編輯 Snapshot 排程。排程可以如下所示進行定義：
   1. 新增每小時 Snapshot、將分鐘設定為 30，並保留最後 24 個 Snapshot
   2. 新增每日 Snapshot、將時間設定為晚上 11 點，並保留最後 7 個 Snapshot
   3. 新增每週 Snapshot、將時間設定為凌晨 1 點，並保留最後 4 個 Snapshot，然後按一下「儲存」。
      ![備份 Snapshot](images/solution14/snapshots.png)

### 授權應用程式伺服器使用檔案儲存空間

1. 在**已授權主機**下，按一下**授權主機**以授權應用程式伺服器（app1 和 app2）使用此檔案儲存空間。

### 裝載檔案儲存空間

對每部應用程式伺服器（app1 和 app2）重複下列步驟：

1. 安裝 NFS 用戶端程式庫
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. 使用 `touch /etc/systemd/system/mnt-www.mount` 建立檔案，然後使用 `nano /etc/systemd/system/mnt-www.mount` 編輯檔案中的下列內容，將 `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` 的值 `What` 取代為檔案儲存空間的**裝載點**（例如，*fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*）。可以在[檔案儲存空間磁區清單](https://{DomainName}/classic/storage/file)下找到裝載點
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
3. 建立裝載點
   ```sh
   mkdir /mnt/www
   ```
4. 裝載儲存空間
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. 檢查裝載是否已順利完成
   ```sh
   mount
   ```
   最後幾行應該列出 File Storage 裝載。若不是如此，請使用 `journalctl -xe` 進行裝載作業的除錯。
   {: tip}

最終，與伺服器配置相關的所有步驟都可使用[佈建 Script](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script)或藉由[擷取映像檔](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template)來自動化。
{: tip}

## 在應用程式伺服器上安裝和配置 PHP 應用程式
{: #php_application}

本指導教學將設定 Wordpress 部落格。所有 Wordpress 檔案都將安裝在共用檔案儲存空間上，以便兩部應用程式伺服器都可以存取這些檔案。安裝 Wordpress 之前，需要配置 Web 伺服器和 PHP 運行環境。

### 安裝 Nginx 和 PHP

對每部應用程式伺服器重複下列步驟：

1. 安裝 Nginx
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. 安裝 PHP 和 MySQL 用戶端
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. 停止 PHP 服務和 Nginx
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. 使用 `nano /etc/nginx/sites-available/default` 將相關內容取代為下列內容：
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
5. 使用下列指令在兩部應用程式伺服器之一的 `/mnt/www` 目錄內建立 `html` 資料夾
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### 安裝及配置 WordPress

由於 Wordpress 將安裝在檔案儲存空間裝載上，因此只需在其中一部伺服器上執行下列步驟即可。我們選取的是 **app1**。

1. 擷取 Wordpress 安裝檔案

   如果應用程式伺服器具有公用網路鏈結，則可以直接從虛擬伺服器中下載 Wordpress 檔案：

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   如果虛擬伺服器只有專用網路鏈結，則需要從具有網際網路存取權的其他機器來擷取安裝檔案，並將其複製到虛擬伺服器。假設已從 https://wordpress.org/latest.tar.gz 擷取到 Wordpress 安裝檔案，則可以使用 `scp` 將其複製到虛擬伺服器：

   ```sh
   scp latest.tar.gz root@PRIVATE_IP_ADDRESS_OF_THE_SERVER:/tmp
   ```
   將 `latest` 取代為從 Wordpress 網站下載的檔名。
   {: tip}

   然後，透過 SSH 進入虛擬伺服器，並切換到 `tmp` 目錄

   ```sh
   cd /tmp
   ```

2. 解壓縮安裝檔案

   ```
   tar xzvf latest.tar.gz
   ```

3. 準備 Wordpress 檔案
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. 將檔案複製到共用檔案儲存空間
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. 設定許可權
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -執行程式 chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. 呼叫下列 Web 服務，並使用 `nano` 將結果注入 `/mnt/www/html/wp-config.php` 中
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   如果虛擬伺服器沒有公用網路鏈結，則只需在 Web 瀏覽器中開啟 https://api.wordpress.org/secret-key/1.1/salt/ 即可。

7. 使用 `nano /mnt/www/html/wp-config.php` 設定資料庫認證，然後更新這些資料庫認證：

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   Wordpress 已配置。若要完成安裝，您需要存取 Wordpress 使用者介面。

在兩部應用程式伺服器上，啟動 Web 伺服器和 PHP 運行環境：
7. 藉由執行下列指令啟動服務：

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

使用專用 IP 位址（如果將透過 VPN 連線進行存取）或者使用 *app1* 或 *app2* 的公用 IP 位址，在 `http://YourAppServerIPAddress/` 處存取 Wordpress 安裝。
![配置虛擬伺服器](images/solution14/wordpress.png)

如果將應用程式伺服器配置為僅使用專用網路鏈結，則無法直接在 Wordpress 管理主控台中安裝 Wordpress 外掛程式、佈景主題或升級。您需要透過 Wordpress 使用者介面來上傳檔案。
{: tip}

## 在應用程式伺服器前端佈建一部負載平衡器伺服器
{: #load_balancer}

目前，我們有兩部應用程式伺服器，使用的是不同的 IP 位址。如果選擇僅佈建專用網路上行鏈路，則在公用網際網路上甚至可能看不到這兩部伺服器。在這兩部伺服器的前端新增負載平衡器將使應用程式可供大眾使用。負載平衡器還將對使用者隱藏基礎的基礎架構。負載平衡器將監視應用程式伺服器的性能，並將送入的要求分派給性能良好的伺服器。

1. 移至「型錄」以建立 [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer)
2. 在**方案**步驟中，選取 *app1* 和 *app2* 所在的資料中心
3. 在**網路設定**中，
   1. 選取在其中佈建 *app1* 和 *app2* 的子網路
   2. 將預設 IBM 系統儲存區用於負載平衡器公用 IP。
4. 在**基本**中，
   1. 命名負載平衡器，例如 **app-lb-1**
   2. 保留預設通訊協定配置 - 依預設，負載平衡器是針對 HTTP 所配置。您自己的憑證支援 SSL 通訊協定。請參閱[在負載平衡器中匯入 SSL 憑證](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates)
      {: tip}
5. 在**伺服器實例**中，新增 *app1* 和 *app2* 伺服器
6. 檢閱並建立以完成精靈。

### 變更 Wordpress 配置以使用負載平衡器 URL

需要變更 Wordpress 配置才能使用負載平衡器位址。實際上，Wordpress 會保留對[部落格 URL 的參照，並在頁面中注入此位置](https://codex.wordpress.org/Settings_General_Screen)。如果未變更此設定，Wordpress 會直接將使用者重新導向到後端伺服器，因此繞過負載平衡器，或者如果伺服器只有專用 IP 位址，則根本不起作用。

1. 在負載平衡器詳細資料頁面中，找到負載平衡器位址。可以在[網路/負載平衡/本端](https://{DomainName}/classic/network/loadbalancing/cloud)下找到建立的負載平衡器。

   還可以藉由新增指向 DNS 配置中負載平衡器位址的 CNAME 記錄，將您自己的網域名稱用於負載平衡器。
   {: tip}
2. 透過 *app1* 或 *app2* URL 以管理者身分登入 Wordpress 部落格
3. 在「設定」/「一般」下，將「Wordpress 位址 (URL)」和「網址 (URL)」都設定為負載平衡器位址
4. 儲存這些設定。Wordpress 應該會重新導向到負載平衡器位址。由於 DNS 傳播，負載平衡器位址變為作用中狀態可能需要一些時間。
   {: tip}

### 測試負載平衡器行為

負載平衡器配置為檢查伺服器的性能，並將使用者僅重新導向到性能良好的伺服器。若要瞭解負載平衡器的運作方式，可以：

1. 使用下列指令監看 *app1* 和 *app2* 上的 Nginx 日誌：
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   您應該已經看到從負載平衡器定期執行的用於檢查伺服器性能狀態的 Ping 作業。
   {: tip}
2. 透過負載平衡器位址存取 Wordpress，並確保強制對頁面執行硬重新載入。請注意，在 Nginx 日誌中，*app1* 和 *app2* 都會提供該頁面的內容。負載平衡器會根據預期將資料流量重新導向到這兩部伺服器。

3. 在 *app1* 上停止 Nginx
   ```sh
   systemctl nginx stop
   ```

4. 片刻之後，重新載入 Wordpress 頁面，請注意所有相符項目都會移至 *app2*。

5. 在 *app2* 上停止 Nginx

6. 重新載入 Wordpress 頁面。由於沒有性能良好的伺服器，因此負載平衡器會傳回錯誤。

7. 在 *app1* 上重新啟動 Nginx
   ```sh
   systemctl nginx start
   ```

8. 負載平衡器偵測到 *app1* 正常運作時，會將資料流量重新導向到此伺服器。

## 使用第二個位置延伸解決方案（選用）
{: #secondregion}

為了提高備援和可用性，可以使用第二個位置來延伸基礎架構設定，並使應用程式在兩個位置執行。

對於第二個位置部署，架構將類似下圖。

<p style="text-align: center;">

  ![架構圖](images/solution14/Architecture2.png)
</p>

1. 使用者透過 IBM Cloud Internet Services (CIS) 存取應用程式。
2. CIS 將資料流量遞送到一個正常運作的位置。
3. 在位置中，負載平衡器會將資料流量重新導向至伺服器。
4. 應用程式存取資料庫。
5. 應用程式在檔案儲存空間中儲存和擷取媒體資產。

若要實作此架構，需要在第二個位置執行下列作業：

- 在新位置中重複上述所有先前步驟。
- 在兩個位置中的兩部 MySQL 伺服器之間設定資料庫抄寫。
- 配置 IBM Cloud Internet Services，以將位置之間的資料流量分散給性能良好的伺服器，如[其他指導教學](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)中所述。

## 移除資源
{: #removeresources}

1. 刪除負載平衡器
2. 取消 *db1*、*app1* 及 *app2*
3. 刪除兩個 File Storage 服務
4. 如果配置第二個位置，則刪除所有資源和 Cloud Internet Services。

## 相關內容
{: #related}

- 應用程式處理的靜態內容可利用負載平衡器前端的 Content Delivery Network，而減輕後端伺服器上的負載。如需實作 Content Delivery Network 的指導教學，請參閱[使用 CDN 加速交付靜態檔案 - Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)。
- 在本指導教學中，我們佈建兩部伺服器，可以自動新增更多伺服器來處理額外負載。使用 [Auto Scale](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale)，能夠將與新增或移除虛擬伺服器相關聯的手動調整處理程序自動化，而支援商業應用程式。
- 若要提高可用性和災難回復選項，可以將檔案儲存空間配置為對內容執行[自動定期 Snapshot](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots)以及[抄寫](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication)到其他資料中心。
