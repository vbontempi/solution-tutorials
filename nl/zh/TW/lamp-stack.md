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

# LAMP 堆疊上的 PHP Web 應用程式
{: #lamp-stack}

本指導教學會引導您建立使用 **A**pache Web 伺服器、**M**ySQL 資料庫和 **P**HP Script 的 Ubuntu **L**inux 虛擬伺服器。這種軟體組合（通常稱為 LAMP 堆疊）十分常用，通常用於交付網站和 Web 應用程式。使用 {{site.data.keyword.BluVirtServers}} 時，可利用內建監視和漏洞掃描功能來快速部署 LAMP 堆疊。若要查看 LAMP 伺服器的運作情況，可安裝並配置免費的開放程式碼 [WordPress](https://wordpress.org/) 內容管理系統。

## 目標

* 在數分鐘內佈建 LAMP 伺服器
* 套用最新的 Apache、MySQL 和 PHP 版本
* 藉由安裝並配置 WordPress 來管理網站或部落格
* 利用監視功能來偵測中斷和效能緩慢
* 評估漏洞並阻止不必要的資料流量

## 使用的服務

本指導教學使用下列運行環境及服務：

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構

![架構圖](images/solution4/Architecture.png)

1. 一般使用者使用 Web 瀏覽器來存取 LAMP 伺服器和應用程式

## 開始之前

{: #prereqs}

1. 請聯絡基礎架構管理者以取得下列許可權。
  * 完成**公用和專用網路上行鏈路**所需的網路許可權

### 配置 VPN 存取

1. [確定 VPN 存取已啟用](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)。

     您應該是**主要使用者**，才能啟用 VPN 存取，或是請與主要使用者聯絡以取得存取權。
     {:tip}
2. 在[使用者清單下的使用者頁面](https://{DomainName}/iam#/users)中取得 VPN 存取認證。
3. 透過 [Web 介面](https://www.softlayer.com/VPN-Access)登入 VPN，或使用 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections)、[macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 或 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7) 的 VPN 用戶端。

   針對 VPN 用戶端，從 [VPN Web 存取頁面](https://www.softlayer.com/VPN-Access)使用單一資料中心 VPN 存取點的 FQDN 作為閘道位址，格式為 *vpn.xxxnn.softlayer.com*。
   {:tip}

## 建立服務

在本節中，您將佈建具有固定配置的公用虛擬伺服器。{{site.data.keyword.BluVirtServers_short}} 可以在數分鐘內從特定地理位置的虛擬伺服器映像檔進行部署。虛擬伺服器通常用於滿足尖峰需求，在尖峰過後可以將其暫停或關閉其電源，這樣雲端環境能完美地適應您的基礎架構需求。

1. 在瀏覽器中，存取 [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) 型錄頁面。
2. 選取**公用虛擬伺服器**，然後按一下**建立**。
3. 在**映像檔**下的 **Ubuntu** 下，選取 **LAMP** 最新版本。即使預先安裝 Apache、MySQL 和 PHP，也可以使用最新版本重新安裝 PHP 和 MySQL。
4. 在**網路介面**下，選取**公用和專用網路上行鏈路**選項。
5. 檢閱其他配置選項，然後按一下**佈建**以建立虛擬伺服器。
     ![配置虛擬伺服器](images/solution4/ConfigureVirtualServer.png)

建立伺服器後，您會看到伺服器登入認證。雖然可以使用伺服器公用 IP 位址透過 SSH 進行連接，但建議透過專用網路來存取伺服器，並停用公用網路上的 SSH 存取。


1. 執行[這些步驟](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network)以保護虛擬機器，並停用公用網路上的 SSH 存取。
1. 使用您的使用者名稱、密碼和專用 IP 位址，利用 SSH 連接至伺服器。
   ```sh
   sudo ssh root@<Private-IP-Address>
   ```
   {: pre}

  可以在儀表板中找到伺服器的專用 IP 位址和密碼。
  {:tip}

  ![已建立的虛擬伺服器](images/solution4/VirtualServerCreated.png)

## 重新安裝 Apache、MySQL 和 PHP

建議定期使用最新的安全修補程式和錯誤修正程式來更新 LAMP 堆疊。在本節中，您將執行指令來更新 Ubuntu 套件來源，並使用最新版本重新安裝 Apache、MySQL 和 PHP。請注意指令結尾的脫字符號 (^)。

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

另一種選擇是使用 `sudo apt-get update && sudo apt-get dist-upgrade` 升級所有套件。
{:tip}

## 驗證安裝和配置

在本節中，您將驗證 Apache、MySQL 和 PHP 是否為最新版本，並且正在 Ubuntu 映像檔上執行。您還將對 MySQL 實作建議的安全設定。

1. 藉由在瀏覽器中開啟公用 IP 位址來驗證 Ubuntu。您應該會看到 Ubuntu 歡迎使用頁面。
   ![驗證 Ubuntu](images/solution4/VerifyUbuntu.png)
2. 藉由執行下列指令，驗證埠 80 是否可用於處理 Web 資料流量。
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![驗證埠](images/solution4/VerifyPort.png)
3. 使用下列指令檢閱安裝的 Apache、MySQL 和 PHP 版本。
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
4. 執行下列 Script 來保護 MySQL 資料庫。
  ```sh
   mysql_secure_installation
   ```
  {: pre}
5. 輸入 MySQL root 使用者密碼，並配置環境的安全設定。完成後，輸入 `\q` 以結束 mysql 提示字元。
  ```sh
  mysql -u root -p
  ```
  {: pre}

  MySQL 的預設使用者名稱和密碼均為 root。
  {:tip}
6. 此外，可以使用下列指令快速建立 PHP 資訊頁面。
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. 檢視建立的 PHP 資訊頁面：開啟瀏覽器，並移至 `http://{YourPublicIPAddress}/info.php`。替換虛擬伺服器的公用 IP 位址。它看起來類似於下圖。

![PHP 參考資訊](images/solution4/PHPInfo.png)

### 安裝及配置 WordPress

藉由安裝應用程式來體驗 LAMP 堆疊。下列步驟將安裝開放程式碼 WordPress 平台，該平台通常用於建立網站和部落格。如需用於正式作業安裝的相關資訊和設定，請參閱 [WordPress 文件](https://codex.wordpress.org/Main_Page)。

1. 執行下列指令安裝 Wordpress。
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. 將 WordPress 配置為使用 MySQL 和 PHP。執行下列指令開啟文字編輯器，並建立 `/etc/wordpress/config-localhost.php` 檔案。
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. 將下列各行複製到該檔案中，並將 *yourPassword* 替換為您的 MySQL 資料庫密碼，其他值保持不變。使用 `Ctrl+X` 鍵儲存並結束該檔案。
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
4. 在工作目錄中，建立文字檔 `wordpress.sql` 以配置 WordPress 資料庫。
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. 新增下列指令，並將 *yourPassword* 替換為您的資料庫密碼，其他值保持不變。然後儲存該檔案。
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. 執行下列指令建立資料庫。
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. 指令完成後，刪除 `wordpress.sql` 檔案。將 WordPress 安裝移至 Web 伺服器文件根目錄。
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. 完成 WordPress 設定，並在平台上發佈。開啟瀏覽器，並移至 `http://{yourVMPublicIPAddress}/wordpress`。替換 VM 的公用 IP 位址。它看起來類似於下圖。
   ![執行中 WordPress 網站](images/solution4/WordPressSiteRunning.png)

## 配置網域

若要將現有網域名稱用於 LAMP 伺服器，請更新 A 記錄以指向虛擬伺服器的公用 IP 地址。可以在儀表板中檢視伺服器的公用 IP 位址。

## 伺服器監視和使用情況

為了確保伺服器可用性和最佳使用者體驗，應在每個正式作業伺服器上已啟用監視。在本節中，您將探索可用於監視虛擬伺服器的選項，並瞭解伺服器在任何給定時間的使用情況。

### 伺服器監視

有兩種基本監視類型可用：服務 Ping 和慢速 Ping。

* **服務 Ping** 用於檢查伺服器回應時間是否等於或短於 1 秒
* **慢速 Ping** 用於檢查伺服器回應時間是否等於或短於 5 秒

依預設新增的是「服務 Ping」，因此請使用下列步驟新增「慢速 Ping」監視。

1. 在儀表板中，從裝置清單中選取您的伺服器，然後按一下**監視**標籤。
  ![慢速 Ping 監視](images/solution4/SlowPing.png)
2. 按一下**管理監視器**。
3. 新增**慢速 Ping** 監視選項，然後按一下**新增監視器**。選取 IP 位址的公用 IP 位址。
  ![新增「慢速 Ping」監視](images/solution4/AddSlowPing.png)

  **附註**：不容許使用具有相同配置的重複監視器。一個配置只能建立一個監視器。

如果在分配的時間範圍內未收到回應，將向 {{site.data.keyword.Bluemix_notm}} 帳戶上的電子郵件位址傳送警示。
  ![兩個監視](images/solution4/TwoMonitoring.png)

### 伺服器使用情況

選取**使用情況**標籤以瞭解現行伺服器的記憶體用量和 CPU 使用率。
  ![伺服器使用情況](images/solution4/ServerUsage.png)

## 伺服器安全

{{site.data.keyword.BluVirtServers}} 提供多種安全選項，例如漏洞掃描和附加程式防火牆。

### 漏洞掃描器

漏洞掃描器會掃描伺服器，以找出與伺服器相關的任何漏洞。若要對伺服器執行漏洞掃描，請執行以下步驟。

1. 在儀表板中，選取您的伺服器，然後按一下**安全**標籤。
2. 按一下**掃描**以啟動掃描。
3. 掃描完成後，按一下**掃描完成**以檢視掃描報告。
  ![兩個監視](images/solution4/Vulnerabilities.png)
4. 檢閱報告的任何漏洞。
  ![兩個監視](images/solution4/VulnerabilityResults.png)

### Firewalls

另一種保護伺服器的方法是新增防火牆。防火牆提供一個重要的安全層：阻止不需要的資料流量到達伺服器、降低攻擊可能性，並容許伺服器資源專用於其目標用途。防火牆選項可依需求進行佈建，不需要岔斷服務。

防火牆可作為「基礎架構」公用網路上所有伺服器的附加程式特性。在訂購處理程序中，您可以選取裝置特定硬體或軟體防火牆來提供保護。或者，您也可以將專用防火牆應用裝置部署至環境，並將虛擬伺服器部署至受保護的 VLAN。如需相關資訊，請參閱[防火牆](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started)。

## 移除資源

若要移除虛擬伺服器，請完成下列步驟。

1. 登入 [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices)。
2. 從**裝置**功能表中，選取**裝置清單**。
3. 按一下要移除的虛擬伺服器的**動作**，然後選取**取消**。

## 相關內容

* [使用 Terraform 部署 LAMP 堆疊](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
