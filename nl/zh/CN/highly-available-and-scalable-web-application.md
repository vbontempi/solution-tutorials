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

# 使用虚拟服务器构建具备高可用性和高可缩放性的 Web 应用程序
{: #highly-available-and-scalable-web-application}

处理额外负载的常见模式是向应用程序添加更多服务器。另一个提高应用程序可用性和弹性的关键方面是通过数据复制和负载均衡，将应用程序部署到多个专区或位置。

本教程将全程指导您实现创建以下内容的场景：

- 位于达拉斯的两个 Web 应用程序服务器
- 云负载均衡器，用于对一个位置中的两个服务器进行流量负载均衡。
- 一个 MySQL 数据库服务器。
- 一个耐用性文件存储器，用于存储应用程序文件和备份。
- 使用与第一个位置相同的配置来配置第二个位置，然后添加 Cloud Internet Services，以在一个副本失败时将流量指向正常运行的位置。

## 目标
{: #objectives}

* 创建 {{site.data.keyword.virtualmachinesshort}} 来安装 PHP 和 MySQL
* 使用 {{site.data.keyword.filestorage_short}} 来持久存储应用程序文件和数据库备份
* 供应 {{site.data.keyword.loadbalancer_short}} 以将请求分发给应用程序服务器
* 通过添加第二个位置来扩展解决方案，以提高弹性和可用性

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

应用程序是一个简单的 PHP 前端（Wordpress 博客），使用的是 MySQL 数据库。多个前端服务器用于处理请求。

<p style="text-align: center;">
  ![体系结构图](images/solution14/Architecture.png)
</p>

1. 用户连接到应用程序。
2. {{site.data.keyword.loadbalancer_short}} 选择其中一个正常运行的服务器来处理请求。
3. 所选服务器访问存储在共享文件存储器上的应用程序文件。
4. 该服务器还从数据库中拉取信息，并最终将页面呈现给用户。
5. 数据库内容将定期备份。万一主服务器出现故障，可以使用备用数据库服务器。

## 开始之前
{: #prereqs}

### 配置 VPN 访问

在本教程中，负载均衡器是面向应用程序用户的前端。{{site.data.keyword.virtualmachinesshort}} 无需在公用因特网上可查看。因此，可以仅为其供应专用 IP 地址，您将使用 VPN 连接在服务器上工作。

1. [确保 VPN 访问已启用](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     您应该是**主用户**才能启用 VPN 访问，或者联系主用户来获取访问权。
     {:tip}
2. 通过在[用户列表](https://{DomainName}/iam#/users)中选择用户来获取 VPN 访问凭证。
3. 通过 [Web 界面](https://www.softlayer.com/VPN-Access)登录到 VPN，或者使用适用于 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 或 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 的 VPN 客户机。

您可以选择跳过此步骤，并使所有服务器均可在公用因特网上查看（但使其保持专用可提供额外的安全级别）。要使其可供公共访问，请在供应 {{site.data.keyword.virtualmachinesshort}} 时选择**公用和专用网络上行链路**。
{: tip}

### 检查帐户许可权

请联系基础架构主用户以获取以下许可权：
- **网络**许可权，以便可以创建使用**公用和专用网络上行链路**的 {{site.data.keyword.virtualmachinesshort}}（如果使用 VPN 连接到服务器，那么无需此许可权）

## 为数据库供应一个服务器
{: #database_server}

在此部分中，您将配置一个服务器以充当主数据库。

1. 转至 {{site.data.keyword.Bluemix}} 控制台中的“目录”，然后从“基础架构”部分中，选择 [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)。
2. 选择 **Public Virtual Server**，然后单击**创建**。
3. 为该服务器配置以下各项：
   - 将**名称**设置为 **db1**。
   - 选择要供应服务器的位置。**本教程中创建的其他所有服务器和资源都需要在同一位置中进行创建。**
   - 选择 **Ubuntu Minimal** 映像。
   - 保留缺省计算类型模板。本教程已使用最小类型模板进行过测试，但应该能够适用于任何类型模板。
   - 在**连接的存储磁盘**下，选择 25 GB 引导磁盘。
   - 在**网络接口**下，选择 **100 Mbps 专用网络上行链路**选项。

     如果未配置 VPN 访问，请选择 **100 Mbps 公用和专用网络上行链路**选项。
     {: tip}
   - 复查其他配置选项，然后单击**供应**以供应服务器。

      ![配置虚拟服务器](images/solution14/db-server.png)

   注：供应过程可能需要 2 到 5 分钟，服务器才可供使用。创建服务器后，您可在**设备 > 设备列表**下的服务器详细信息页面中找到服务器凭证。要通过 SSH 连接到服务器，需要服务器专用或公共 IP 地址、用户名和密码（单击设备名称旁边的箭头）。
   {: tip}

## 安装和配置 MySQL
{: #mysql}

服务器不随附数据库。在此部分中，您将在服务器上安装 MySQL。

### 安装 MySQL

1. 使用 SSH 连接到服务器：
   ```sh
   ssh root@<Private-OR-Public-IP-Address>
   ```

   请务必根据虚拟服务器的**位置**，使用正确的[站点地址](https://www.softlayer.com/VPN-Access)连接到 VPN 客户机。
   {:tip}
2. 安装 MySQL：
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   系统可能会提示您输入密码。请通读控制台上显示的指示信息。
   {:tip}
3. 运行以下脚本来帮助保护 MySQL 数据库：
   ```sh
   mysql_secure_installation
   ```

   系统可能会提示您从几个选项中进行选择。请根据需求进行明智的选择。
   {:tip}

### 为应用程序创建数据库

1. 登录到 MySQL，然后创建名为 `wordpress` 的数据库：
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. 授予对该数据库的访问权，将 database-username 和 database-password 替换为您早先设置的内容。

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. 使用以下命令检查是否创建了该数据库：

   ```sh
   show databases;
   ```

4. 使用以下命令从该数据库退出：

   ```sh
   exit
   ```

5. 记下数据库名称、用户和密码。配置应用程序服务器时，将需要这些信息。

### 使 MySQL 服务器对网络上的其他服务器可视

缺省情况下，MySQL 仅侦听本地接口。应用程序服务器将需要连接到数据库，因此 MySQL 配置需要更改为侦听专用网络接口。

1. 使用 `nano /etc/mysql/my.cnf` 编辑 my.cnf 文件，并添加以下行：
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. 使用 Ctrl+X 退出并保存文件。

3. 重新启动 MySQL：

   ```sh
   systemctl restart mysql
   ```

4. 通过运行以下命令，确认 MySQL 是否在侦听所有接口：
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## 创建用于存储数据库备份的文件存储器
{: #database_backup}

对于 MySQL，有许多方法可以执行和存储备份。本教程使用 crontab 条目将数据库内容转储到磁盘。备份文件将存储在文件存储器中。显然，这是一个简单的备份机制。如果您计划在生产环境中管理自己的 MySQL 数据库服务器，那么需要[实现 MySQL 文档中描述的其中一个备份策略](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html)。

### 创建文件存储器
{: #create_for_backup}

1. 转至 {{site.data.keyword.Bluemix}} 控制台中的“目录”，然后选择 [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)。
2. 单击**创建**。
3. 为该服务配置以下各项：
   - 将**存储类型**设置为**耐久性**
   - 选择在其中创建了数据库服务器的**位置**
   - 选择计费方式
   - 在**存储包**下，选择 **0.25 IOPS/GB**
   - 在**存储器大小**下，选择 **20 GB**
   - 使**快照空间大小**保留为 **0 GB**
   - 单击“继续”以创建服务。

### 授权数据库服务器使用文件存储器

要使虚拟服务器能够安装文件存储器，需要先对虚拟服务器进行授权。

1. 从[现有项列表](https://{DomainName}/classic/storage/file)中选择新创建的文件存储器。
2. 在**已授权主机**下，单击**授权主机**，然后选择虚拟（数据库）服务器（选择**设备**> 将“虚拟服务器”作为“设备类型”> 输入服务器的名称）。

### 安装用于存储数据库备份的文件存储器

文件存储器可以作为 NFS 驱动器安装到虚拟服务器中。

1. 安装 NFS 客户机库：
   ```sh
   apt-get -y install nfs-common
   ```

2. 通过运行以下命令，创建名为 `/etc/systemd/system/mnt-database.mount` 的文件：
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. 使用以下命令编辑 mnt-database.mount：
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. 将以下内容添加到 mnt-database.mount 文件中，并将 `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` 的值 `What` 替换为文件存储器的**安装点**（例如，*fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*）。可以在创建的 File Storage 服务下获取**安装点** URL。
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
   使用 Ctrl+X 来保存并退出 nano 窗口
   {: tip}

5. 创建安装点
  ```sh
  mkdir /mnt/database
  ```

6. 安装存储器
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. 检查安装是否已成功执行
   ```sh
   安装
   ```
   最后几行应列出文件存储器安装。如果未列出，请使用 `journalctl -xe` 来调试安装操作。
   {: tip}

### 设置定期备份

1. 使用以下命令，通过将 `CHANGE_ME` 替换为早先指定的数据库密码，创建 `/root/dbbackup.sh` shell 脚本（使用 `touch` 和 `nano`）：
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip > /mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. 确保该文件是可执行文件
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. 编辑 crontab
   ```sh
   crontab -e
   ```
4. 要在每天晚上 11 点执行备份，请将内容设置为如下内容，然后保存文件并关闭编辑器
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## 为 PHP 应用程序供应两个服务器
{: #app_servers}

在此部分中，您将创建两个 Web 应用程序服务器。

1. 转至 {{site.data.keyword.Bluemix}} 控制台中的“目录”，然后从“基础架构”部分中，选择 [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) 服务。
2. 选择 **Public Virtual Server**，然后单击**创建**。
3. 为该服务器配置以下各项：
   - 将**名称**设置为 **app1**。
   - 选择在其中供应数据库服务器的位置。
   - 选择 **Ubuntu Minimal** 映像。
   - 保留缺省计算类型模板。
   - 在**连接的存储磁盘**下，选择 25 GB 作为引导磁盘。
   - 在**网络接口**下，选择 **100 Mbps 专用网络上行链路**选项。

     如果未配置 VPN 访问，请选择 **100 Mbps 公用和专用网络上行链路**选项。
     {: tip}
   - 复查其他配置选项，然后单击**供应**以供应服务器。
     ![配置虚拟服务器](images/solution14/db-server.png)
4. 重复步骤 1-3 以供应另一个名为 **app2** 的虚拟服务器。

## 创建文件存储器以用于在应用程序服务器之间共享文件
{: shared_storage}

此文件存储器用于在 *app1* 和 *app2* 服务器之间共享应用程序文件。

### 创建文件存储器
{: #create_for_sharing}

1. 转至 {{site.data.keyword.Bluemix}} 控制台中的“目录”，然后选择 [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)。
2. 单击**创建**。
3. 为该服务配置以下各项：
   - 将**存储类型**设置为**耐久性**
   - 选择在其中创建了应用程序服务器的**位置**
   - 选择计费方式
   - 在**存储包**下，选择 **2 IOPS/GB**
   - 在**存储器大小**下，选择 **20 GB**
   - 在**快照空间大小**下，选择 **20 GB**
   - 单击“继续”以创建服务。

### 配置定期快照

[快照](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots)是一个方便的选项，可用于保护数据，而不会影响性能。此外，还可以将快照复制到其他数据中心。

1. 从[现有项列表](https://{DomainName}/classic/storage/file)中选择文件存储器。
2. 在**快照安排**下，编辑快照安排。安排可以如下所示进行定义：
   1. 添加每小时快照，将分钟设置为 30，并保留最后 24 个快照
   2. 添加每日快照，将时间设置为晚上 11 点，并保留最后 7 个快照
   3. 添加每周快照，将时间设置为凌晨 1 点，并保留最后 4 个快照，然后单击“保存”。
      ![备份快照](images/solution14/snapshots.png)

### 授权应用程序服务器使用文件存储器

1. 在**已授权主机**下，单击**授权主机**以授权应用程序服务器（app1 和 app2）使用此文件存储器。

### 安装文件存储器

对每个应用程序服务器（app1 和 app2）重复以下步骤：

1. 安装 NFS 客户机库
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. 使用 `touch /etc/systemd/system/mnt-www.mount` 创建文件，然后使用 `nano /etc/systemd/system/mnt-www.mount` 编辑文件中的以下内容，将 `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` 的值 `What` 替换为文件存储器的**安装点**（例如，*fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*）。可以在[文件存储器卷列表](https://{DomainName}/classic/storage/file)下找到安装点
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
3. 创建安装点
   ```sh
   mkdir /mnt/www
   ```
4. 安装存储器
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. 检查安装是否已成功执行
   ```sh
   安装
   ```
   最后几行应列出文件存储器安装。如果未列出，请使用 `journalctl -xe` 来调试安装操作。
   {: tip}

最终，与服务器配置相关的所有步骤都可使用[供应脚本](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script)或通过[捕获映像](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template)来自动执行。
{: tip}

## 在应用程序服务器上安装和配置 PHP 应用程序
{: #php_application}

本教程将设置 Wordpress 博客。所有 Wordpress 文件都将安装在共享文件存储器上，以便两个应用程序服务器都可以访问这些文件。安装 Wordpress 之前，需要配置 Web 服务器和 PHP 运行时。

### 安装 Nginx 和 PHP

对每个应用程序服务器重复以下步骤：

1. 安装 Nginx
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. 安装 PHP 和 MySQL 客户机
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. 停止 PHP 服务和 Nginx
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. 使用 `nano /etc/nginx/sites-available/default` 将相关内容替换为以下内容：
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
5. 使用以下命令在两个应用程序服务器之一上的 `/mnt/www` 目录内创建 `html` 文件夹
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### 安装和配置 WordPress

由于 Wordpress 将安装在文件存储器安装上，因此只需在其中一个服务器上执行以下步骤即可。我们选取的是 **app1**。

1. 检索 Wordpress 安装文件

   如果应用程序服务器具有公用网络链路，那么可以直接从虚拟服务器中下载 Wordpress 文件：

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   如果虚拟服务器只有专用网络链路，那么需要通过具有因特网访问权的其他机器来检索安装文件，并将其复制到虚拟服务器。假定已从 https://wordpress.org/latest.tar.gz 检索到 Wordpress 安装文件，那么可以使用 `scp` 将其复制到虚拟服务器：

   ```sh
   scp latest.tar.gz root@PRIVATE_IP_ADDRESS_OF_THE_SERVER:/tmp
   ```
   将 `latest` 替换为从 Wordpress Web 站点下载的文件名。
   {: tip}

   然后，通过 SSH 登录到虚拟服务器，并切换到 `tmp` 目录

   ```sh
   cd /tmp
   ```

2. 解压缩安装文件

   ```
   tar xzvf latest.tar.gz
   ```

3. 准备 Wordpress 文件
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. 将文件复制到共享文件存储器
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. 设置许可权
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -exec chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. 调用以下 Web service 并使用 `nano` 将结果注入到 `/mnt/www/html/wp-config.php` 中
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   如果虚拟服务器没有公用网络链路，那么只需在 Web 浏览器中打开 https://api.wordpress.org/secret-key/1.1/salt/ 即可。

7. 使用 `nano /mnt/www/html/wp-config.php` 设置数据库凭证，然后更新这些数据库凭证：

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   Wordpress 已配置。要完成安装，您需要访问 Wordpress 用户界面。

在两个应用程序服务器上，启动 Web 服务器和 PHP 运行时：
7. 通过运行以下命令启动服务：

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

使用专用 IP 地址（如果将通过 VPN 连接进行访问）或者使用 *app1* 或 *app2* 的公共 IP 地址，在 `http://YourAppServerIPAddress/` 处访问 Wordpress 安装。
![配置虚拟服务器](images/solution14/wordpress.png)

如果将应用程序服务器配置为仅使用专用网络链路，那么无法直接在 Wordpress 管理控制台中安装 Wordpress 插件、主题或升级。您需要通过 Wordpress 用户界面来上传文件。
{: tip}

## 在应用程序服务器前端供应一个负载均衡器服务器
{: #load_balancer}

目前，我们有两个应用程序服务器，使用的是不同的 IP 地址。如果选择仅供应专用网络上行链路，那么在公用因特网上甚至可能看不到这两个服务器。在这两个服务器的前端添加负载均衡器将使应用程序可供公共访问。负载均衡器还将对用户隐藏底层基础架构。负载均衡器将监视应用程序服务器的运行状况，并将入局请求分派给正常运行的服务器。

1. 转至“目录”以创建 [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer)
2. 在**套餐**步骤中，选择 *app1* 和 *app2* 所在的数据中心
3. 在**网络设置**中，
   1. 选择在其中供应了 *app1* 和 *app2* 的子网
   2. 将缺省 IBM 系统池用于负载均衡器公共 IP。
4. 在**基本**中，
   1. 命名负载均衡器，例如 **app-lb-1**
   2. 保留缺省协议配置 - 缺省情况下，负载均衡器配置的是 HTTP。您自己的证书支持 SSL 协议。请参阅[在负载均衡器中导入 SSL 证书](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates)
      {: tip}
5. 在**服务器实例**中，添加 *app1* 和 *app2* 服务器
6. 复查并创建以完成向导。

### 更改 Wordpress 配置以使用负载均衡器 URL

需要更改 Wordpress 配置才能使用负载均衡器地址。实际上，Wordpress 会保留对[博客 URL 的引用，并在页面中注入此位置](https://codex.wordpress.org/Settings_General_Screen)。如果不更改此设置，Wordpress 会直接将用户重定向到后端服务器，从而绕过负载均衡器，或者如果服务器只有专用 IP 地址，那么根本不起作用。

1. 在负载均衡器详细信息页面中，找到负载均衡器地址。可以在[网络 - 负载均衡 - 本地](https://{DomainName}/classic/network/loadbalancing/cloud)下找到创建的负载均衡器。

   还可以通过添加指向 DNS 配置中负载均衡器地址的 CNAME 记录，将您自己的域名用于负载均衡器。
   {: tip}
2. 通过 *app1* 或 *app2* URL 以管理员身份登录到 Wordpress 博客
3. 在“设置”-“常规”下，将“Wordpress 地址 (URL)”和“站点地址 (URL)”都设置为负载均衡器地址
4. 保存这些设置。Wordpress 应该会重定向到负载均衡器地址。由于 DNS 传播，负载均衡器地址变为活动状态可能需要一些时间。
   {: tip}

### 测试负载均衡器行为

负载均衡器配置为检查服务器的运行状况，并将用户仅重定向到正常运行的服务器。要了解负载均衡器的运作方式，可以：

1. 使用以下命令监控 *app1* 和 *app2* 上的 Nginx 日志：
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   您应该已经看到通过负载均衡器定期执行的用于检查服务器运行状况的 ping 操作。
   {: tip}
2. 通过负载均衡器地址访问 Wordpress，并确保强制对页面执行硬重新装入。请注意，在 Nginx 日志中，*app1* 和 *app2* 都在处理该页面的内容。负载均衡器会按预期将流量重定向到这两个服务器。

3. 在 *app1* 上停止 Nginx
   ```sh
   systemctl nginx stop
   ```

4. 片刻之后，重新装入 Wordpress 页面，请注意所有命中都会转至 *app2*。

5. 在 *app2* 上停止 Nginx

6. 重新装入 Wordpress 页面。由于没有正常运行的服务器，因此负载均衡器会返回错误。

7. 在 *app1* 上重新启动 Nginx
   ```sh
   systemctl nginx start
   ```

8. 负载均衡器检测到 *app1* 正常运行时，会将流量重定向到此服务器。

## 使用第二个位置扩展解决方案（可选）
{: #secondregion}

为了提高弹性和可用性，可以使用第二个位置来扩展基础架构设置，并使应用程序在两个位置运行。

对于第二个位置部署，体系结构将类似下图。

<p style="text-align: center;">

  ![体系结构图](images/solution14/Architecture2.png)
</p>

1. 用户通过 IBM Cloud Internet Services (CIS) 访问应用程序。
2. CIS 将流量路由到一个正常运行的位置。
3. 在一个位置中，负载均衡器将流量重定向到服务器。
4. 应用程序访问数据库。
5. 应用程序在文件存储器中存储和检索媒体资产。

要实现此体系结构，需要在第二个位置执行以下操作：

- 在新位置中重复上述所有先前步骤。
- 在两个位置中的两个 MySQL 服务器之间设置数据库复制。
- 配置 IBM Cloud Internet Services，以将位置之间的流量分发给正常运行的服务器，如[其他教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)中所述。

## 除去资源
{: #removeresources}

1. 删除负载均衡器
2. 取消 *db1*、*app1* 和 *app2*
3. 删除两个 File Storage 服务
4. 如果配置了第二个位置，那么删除所有资源和 Cloud Internet Services。

## 相关内容
{: #related}

- 应用程序处理的静态内容可利用负载均衡器前端的 Content Delivery Network，从而减少后端服务器上的负载。有关实施 Content Delivery Network 的教程，请参阅[使用 CDN 加速交付静态文件 - Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)。
- 在本教程中，我们供应了两个服务器，可以自动添加更多服务器来处理额外负载。通过 [Auto Scale](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale)，能够自动执行与添加或除去虚拟服务器关联的手动缩放过程，从而支持业务应用程序。
- 要提高可用性和灾难恢复选项，可以将文件存储器配置为对内容执行[自动定期快照](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots)以及[复制](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication)到其他数据中心。
