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

# LAMP 堆栈上的 PHP Web 应用程序
{: #lamp-stack}

本教程将全程指导您创建使用 **A**pache Web 服务器、**M**ySQL 数据库和 **P**HP 脚本编制的 Ubuntu **L**inux 虚拟服务器。这种软件组合（通常称为 LAMP 堆栈）十分常用，通常用于交付 Web 站点和 Web 应用程序。使用 {{site.data.keyword.BluVirtServers}} 时，可通过内置监视和漏洞扫描功能来快速部署 LAMP 堆栈。要查看 LAMP 服务器的运行情况，可安装并配置免费的开放式源代码 [WordPress](https://wordpress.org/) 内容管理系统。

## 目标

* 在数分钟内供应 LAMP 服务器
* 应用最新的 Apache、MySQL 和 PHP 版本
* 通过安装并配置 WordPress 来托管 Web 站点或博客
* 利用监视功能来检测中断和性能缓慢问题
* 评估漏洞并阻止不必要的流量

## 使用的服务

本教程使用以下运行时和服务：

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构

![体系结构图](images/solution4/Architecture.png)

1. 最终用户使用 Web 浏览器来访问 LAMP 服务器和应用程序

## 开始之前

{: #prereqs}

1. 请联系基础架构管理员以获取以下许可权。
  * 完成**公用和专用网络上行链路**所需的网络许可权

### 配置 VPN 访问

1. [确保 VPN 访问已启用](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     您应该是**主用户**才能启用 VPN 访问，或者联系主用户来获取访问权。
     {:tip}
2. 在[用户列表下的用户页面](https://{DomainName}/iam#/users)中获取 VPN 访问凭证。
3. 通过 [Web 界面](https://www.softlayer.com/VPN-Access)登录到 VPN，或者使用适用于 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 或 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 的 VPN 客户机。

   对于 VPN 客户机，使用 [VPN Web 访问页面](https://www.softlayer.com/VPN-Access)中单个数据中心 VPN 访问点的 FQDN（格式为 *vpn.xxxnn.softlayer.com*）作为网关地址。
   {:tip}

## 创建服务

在此部分中，您将供应具有固定配置的公共虚拟服务器。{{site.data.keyword.BluVirtServers_short}} 可以在数分钟内通过特定地理位置的虚拟服务器映像进行部署。虚拟服务器通常用于满足峰值需求，在峰值过后可以将其暂停或关闭其电源，这样云环境能完美地适应您的基础架构需求。

1. 在浏览器中，访问 [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) 目录页面。
2. 选择 **Public Virtual Server**，然后单击**创建**。
3. 在**映像**下的 **Ubuntu** 下，选择 **LAMP** 最新版本。尽管预安装了 Apache、MySQL 和 PHP，也可以使用最新版本重新安装 PHP 和 MySQL。
4. 在**网络接口**下，选择**公用和专用网络上行链路**选项。
5. 复查其他配置选项，然后单击**供应**以创建虚拟服务器。
     ![配置虚拟服务器](images/solution4/ConfigureVirtualServer.png)

创建服务器后，您会看到服务器登录凭证。虽然可以使用服务器公共 IP 地址通过 SSH 进行连接，但建议通过专用网络来访问服务器，并禁用公用网络上的 SSH 访问。


1. 执行[这些步骤](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network)以保护虚拟机，并禁用公用网络上的 SSH 访问。
1. 使用您的用户名、密码和专用 IP 地址，通过 SSH 连接到服务器。
   ```sh
   sudo ssh root@<Private-IP-Address>
   ```
   {: pre}

  可以在仪表板中找到服务器的专用 IP 地址和密码。
  {:tip}

  ![已创建的虚拟服务器](images/solution4/VirtualServerCreated.png)

## 重新安装 Apache、MySQL 和 PHP

建议定期使用最新的安全补丁和错误修订来更新 LAMP 堆栈。在此部分中，您将运行命令来更新 Ubuntu 软件包源，并使用最新版本重新安装 Apache、MySQL 和 PHP。请注意命令末尾的插入标记 (^)。

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

另一种选择是使用 `sudo apt-get update && sudo apt-get dist-upgrade` 升级所有软件包。
{:tip}

## 验证安装和配置

在此部分中，您将验证 Apache、MySQL 和 PHP 是否为最新版本，并且正在 Ubuntu 映像上运行。您还将对 MySQL 实现建议的安全设置。

1. 通过在浏览器中打开公共 IP 地址来验证 Ubuntu。您应该会看到 Ubuntu 欢迎页面。
   ![验证 Ubuntu](images/solution4/VerifyUbuntu.png)
2. 通过运行以下命令，验证端口 80 是否可用于处理 Web 流量。
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![验证端口](images/solution4/VerifyPort.png)
3. 使用以下命令查看安装的 Apache、MySQL 和 PHP 版本。
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
4. 运行以下脚本来保护 MySQL 数据库。
  ```sh
  mysql_secure_installation
  ```
  {: pre}
5. 输入 MySQL root 用户密码，并配置环境的安全设置。完成后，输入 `\q` 以退出 mysql 提示符。
  ```sh
  mysql -u root -p
  ```
  {: pre}

  MySQL 的缺省用户名和密码均为 root。
  {:tip}
6. 此外，可以使用以下命令快速创建 PHP 信息页面。
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. 查看创建的 PHP 信息页面：打开浏览器并转至 `http://{YourPublicIPAddress}/info.php`。替换虚拟服务器的公共 IP 地址。它看起来类似于下图。

![PHP 信息](images/solution4/PHPInfo.png)

### 安装和配置 WordPress

通过安装应用程序来体验 LAMP 堆栈。以下步骤将安装开放式源代码 WordPress 平台，该平台通常用于创建 Web 站点和博客。有关用于生产安装的更多信息和设置，请参阅 [WordPress 文档](https://codex.wordpress.org/Main_Page)。

1. 运行以下命令安装 Wordpress。
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. 将 WordPress 配置为使用 MySQL 和 PHP。运行以下命令打开文本编辑器，并创建 `/etc/wordpress/config-localhost.php` 文件。
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. 将以下行复制到该文件中，并将 *yourPassword* 替换为您的 MySQL 数据库密码，其他值保持不变。使用 `Ctrl+X` 键保存并退出该文件。
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
4. 在工作目录中，创建文本文件 `wordpress.sql` 以配置 WordPress 数据库。
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. 添加以下命令，并将 *yourPassword* 替换为您的数据库密码，其他值保持不变。然后保存该文件。
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. 运行以下命令创建数据库。
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. 命令完成后，删除 `wordpress.sql` 文件。将 WordPress 安装移至 Web 服务器文档根目录。
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. 完成 WordPress 设置并在平台上发布。打开浏览器并转至 `http://{yourVMPublicIPAddress}/wordpress`。替换 VM 的公共 IP 地址。它看起来类似于下图。
   ![运行中的 WordPress 站点](images/solution4/WordPressSiteRunning.png)

## 配置域

要将现有域名用于 LAMP 服务器，请更新 A 记录以指向虚拟服务器的公共 IP 地址。可以在仪表板中查看服务器的公共 IP 地址。

## 服务器监视和使用情况

为了确保服务器可用性和最佳用户体验，应在每个生产服务器上启用监视。在此部分中，您将探索可用于监视虚拟服务器的选项，并了解在任何给定时间服务器的使用情况。

### 服务器监视

有两种基本监视类型可用：服务 Ping 和慢速 Ping。

* **服务 Ping** 用于检查服务器响应时间是否等于或短于 1 秒
* **慢速 Ping** 用于检查服务器响应时间是否等于或短于 5 秒

缺省情况下添加的是“服务 Ping”，因此请使用以下步骤添加“慢速 Ping”监视。

1. 在仪表板中，从设备列表中选择您的服务器，然后单击**监视**选项卡。
  ![“慢速 Ping”监视](images/solution4/SlowPing.png)
2. 单击**管理监视器**。
3. 添加**慢速 Ping** 监视选项，然后单击**添加监视器**。选择 IP 地址的公共 IP 地址。
  ![添加“慢速 Ping”监视](images/solution4/AddSlowPing.png)

  **注**：不允许使用具有相同配置的重复监视器。每个配置只能创建一个监视器。

如果在分配的时间范围内未收到响应，将向 {{site.data.keyword.Bluemix_notm}} 帐户上的电子邮件地址发送警报。
  ![两个监视](images/solution4/TwoMonitoring.png)

### 服务器使用情况

选择**使用情况**选项卡以了解当前服务器的内存和 CPU 使用情况。
  ![服务器使用情况](images/solution4/ServerUsage.png)

## 服务器安全性

{{site.data.keyword.BluVirtServers}} 提供了多种安全选项，例如漏洞扫描和附加组件防火墙。

### 漏洞扫描程序

漏洞扫描程序会扫描服务器，以查找与服务器相关的任何漏洞。要对服务器运行漏洞扫描，请执行以下步骤。

1. 在仪表板中，选择您的服务器，然后单击**安全性**选项卡。
2. 单击**扫描**以启动扫描。
3. 扫描完成后，单击**扫描完成**以查看扫描报告。
  ![两个监视](images/solution4/Vulnerabilities.png)
4. 查看报告的任何漏洞。
  ![两个监视](images/solution4/VulnerabilityResults.png)

### 防火墙

另一种保护服务器的方法是添加防火墙。防火墙提供了一个重要的安全层：阻止不需要的流量到达服务器，降低攻击可能性，并允许服务器资源专用于其目标用途。防火墙选项可随需应变进行供应，无需中断服务。

防火墙可作为基础架构公用网络上所有服务器的附加组件功能提供。在订购过程中，可以选择特定于设备的硬件或软件防火墙来提供保护。或者，可以将专用防火墙设备部署到环境，然后将虚拟服务器部署到受保护的 VLAN。有关更多信息，请参阅[防火墙](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started)。

## 除去资源

要除去虚拟服务器，请完成以下步骤。

1. 登录到 [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices)。
2. 从**设备**菜单中，选择**设备列表**。
3. 单击要除去的虚拟服务器的**操作**，然后选择**取消**。

## 相关内容

* [使用 Terraform 部署 LAMP 堆栈](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
