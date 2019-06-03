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

# LAMP 스택의 PHP 웹 애플리케이션
{: #lamp-stack}

이 튜토리얼에서는 Ubuntu **L**inux Virtual Server(**A**pache 웹 서버, **M**ySQL 데이터베이스 및 **P**HP 스크립팅 포함)를 작성하는 단계를 안내합니다. 이 소프트웨어 조합(일반적으로 LAMP 스택이라고 함)은 대중적이며 웹 사이트 및 웹 애플리케이션을 전달하는 데 빈번히 사용됩니다. 여기서는 {{site.data.keyword.BluVirtServers}}를 사용하여 기본 제공 모니터링 및 취약성 스캔 기능을 포함하는 LAMP 스택을 빠르게 배치합니다. 또한 LAMP 서버가 작동하는 모습을 보기 위해 무료 오픈 소스인 [WordPress](https://wordpress.org/) 컨텐츠 관리 시스템을 설치하고 구성합니다. 

## 목표

* 몇 분 내에 LAMP 서버를 프로비저닝
* 최신 Apache, MySQL 및 PHP 버전 적용
* WordPress를 설치하고 구성하여 웹 사이트 또는 블로그를 호스팅
* 모니터링을 이용하여 가동 중단 및 느린 성능을 발견
* 취약성을 평가하고 원치 않는 트래픽으로부터 보호

## 사용되는 서비스

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처

![아키텍처 다이어그램](images/solution4/Architecture.png)

1. 일반 사용자가 웹 브라우저를 사용하여 LAMP 서버 및 애플리케이션에 액세스합니다. 

## 시작하기 전에

{: #prereqs}

1. 인프라 관리자에게 문의하여 다음 권한을 얻으십시오. 
  * **공용 및 사설 네트워크 업링크** 설정을 위한 네트워크 권한

### VPN 액세스 구성

1. [VPN 액세스가 사용으로 설정되어 있는지 확인하십시오](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking). 

     VPN 액세스를 사용으로 설정하려면 **마스터 사용자**여야 하며, 그렇지 않은 경우에는 마스터 사용자에게 액세스를 요청하십시오.
     {:tip}
2. [사용자 목록에 있는 사용자 페이지](https://{DomainName}/iam#/users)에서 VPN 액세스 인증 정보를 얻으십시오. 
3. [웹 인터페이스](https://www.softlayer.com/VPN-Access)를 통하거나 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 또는 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7)용 VPN 클라이언트를 사용하여 VPN에 로그인하십시오. 

   VPN 클라이언트의 경우에는 [VPN 웹 액세스 페이지](https://www.softlayer.com/VPN-Access)에 있는 단일 데이터 센터 VPN 액세스 지점의 FQDN(*vpn.xxxnn.softlayer.com* 양식)을 게이트웨이 주소로 사용하십시오.
   {:tip}

## 서비스 작성

이 절에서는 고정된 구성을 사용하여 공용 Virtual Server를 프로비저닝합니다. {{site.data.keyword.BluVirtServers_short}}는 특정 지리적 위치에 있는 Virtual Server 이미지로부터 수 분 내에 배치될 수 있습니다. Virtual Server는 클라우드 환경이 사용자의 인프라 요구사항에 완벽히 부합할 수 있도록, 일반적으로 수요가 가장 많은 경우를 처리하고 그 후에는 일시중단하거나 전원을 끌 수 있습니다. 

1. 브라우저에서 [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) 카탈로그 페이지에 액세스하십시오. 
2. **공용 Virtual Server**를 선택하고 **작성**을 클릭하십시오. 
3. **이미지**에서 **Ubuntu** 아래에 있는 **LAMP** 최신 버전을 선택하십시오. 이 이미지에는 Apache, MySQL 및 PHP가 사전 설치되어 있지만 여기서는 Apache, MySQL 및 PHP를 최신 버전으로 다시 설치합니다. 
4. **네트워크 인터페이스**에서 **공용 및 사설 네트워크 업링크** 옵션을 선택하십시오. 
5. 다른 구성 옵션을 검토하고 **프로비저닝**을 클릭하여 Virtual Server를 작성하십시오.
  ![Virtual Server 구성](images/solution4/ConfigureVirtualServer.png)

서버가 작성되고 나면 서버 로그인 인증 정보가 표시됩니다. 서버 공인 IP 주소를 사용하여 SSH를 통해 연결할 수도 있지만, 사설 네트워크를 통해 서버에 액세스하고 공용 네트워크에서의 SSH 액세스는 사용 안함으로 설정하는 것이 좋습니다. 

1. 가상 머신을 보안하고 공용 네트워크에서의 SSH 액세스를 사용 안함으로 설정하려면 [이러한 단계](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network)를 따르십시오. 
1. 사용자 이름, 비밀번호 및 사설 IP 주소를 사용하여, SSH를 통해 서버에 연결하십시오. 
   ```sh
   sudo ssh root@<Private-IP-Address>
   ```
   {: pre}

  서버의 사설 IP 주소 및 비밀번호는 대시보드에서 찾을 수 있습니다.
  {:tip}

  ![작성된 Virtual Server](images/solution4/VirtualServerCreated.png)

## Apache, MySQL 및 PHP 다시 설치

정기적으로 최신 보안 패치 및 버그 수정으로 LAMP stack 스택을 업데이트하는 것이 좋습니다. 이 절에서는 Ubuntu 패키지 소스를 업데이트하고 Apache, MySQL 및 PHP를 최신 버전으로 다시 설치하는 명령을 실행합니다. 명령 마지막에 캐럿(^)이 있다는 점을 참고하십시오. 

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

다른 방법은 `sudo apt-get update && sudo apt-get dist-upgrade`로 모든 패키지를 업그레이드하는 것입니다.
{:tip}

## 설치 및 구성 확인

이 절에서는 Apache, MySQL 및 PHP가 최신 버전이며 Ubuntu 이미지에서 실행 중인지 확인합니다. 또한 MySQL에 대한 권장 보안 설정을 구현합니다. 

1. 브라우저에서 공인 IP 주소를 열어 Ubuntu를 확인하십시오. Ubuntu 시작 페이지가 표시됩니다.
   ![Ubuntu 확인](images/solution4/VerifyUbuntu.png)
2. 다음 명령을 실행하여 웹 트래픽에 포트 80을 사용할 수 있는지 확인하십시오. 
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![포트 확인](images/solution4/VerifyPort.png)
3. 다음 명령을 사용하여 설치된 Apache, MySQL 및 PHP 버전을 검토하십시오. 
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
4. 다음 스크립트를 실행하여 MySQL 데이터베이스를 보안하십시오. 
  ```sh
  mysql_secure_installation
  ```
  {: pre}
5. MySQL 루트 비밀번호를 입력하고 환경에 대한 보안 설정을 구성하십시오. 구성을 완료한 후에는 `\q`를 입력하여 mysql 프롬프트를 종료하십시오. 
  ```sh
  mysql -u root -p
  ```
  {: pre}

  MySQL 기본 사용자 이름 및 비밀번호는 각각 root와 root입니다.
  {:tip}
6. 또한 다음 명령을 사용하여 PHP 정보 페이지를 신속히 작성할 수 있습니다. 
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. 작성한 PHP 정보 페이지를 보십시오. 브라우저를 열고 `http://{YourPublicIPAddress}/info.php`로 이동하십시오. Virtual Server의 공인 IP 주소를 대체하십시오. 이는 다음 이미지와 유사합니다. 

![PHP 정보](images/solution4/PHPInfo.png)

### WordPress 설치 및 구성

애플리케이션을 설치하여 LAMP 스택을 실제로 사용해 보십시오. 다음 단계는 일반적으로 웹 사이트 및 블로그를 작성하는 데 사용되는 오픈 소스 WordPress 플랫폼을 설치합니다. 자세한 정보와 프로덕션 설치의 설정은 [WordPress 문서](https://codex.wordpress.org/Main_Page)를 참조하십시오. 

1. 다음 명령을 실행하여 WordPress를 설치하십시오. 
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. MySQL 및 PHP를 사용하도록 WordPress를 구성하십시오. 다음 명령을 실행하여 텍스트 편집기를 열고 `/etc/wordpress/config-localhost.php` 파일을 작성하십시오. 
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. *yourPassword*를 자신의 MySQL 데이터베이스 비밀번호로 대체하고 다른 값은 그대로 두어 다음 행을 파일에 복사하십시오. `Ctrl+X`를 사용하여 파일을 저장하고 종료하십시오. 
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
4. 작업 디렉토리에서 WordPress 데이터베이스 구성을 위한 텍스트 파일 `wordpress.sql`을 작성하십시오. 
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. *yourPassword*를 자신의 데이터베이스 비밀번호로 대체하고 다른 값은 그대로 두어 다음 행을 추가하십시오. 그 후 파일을 저장하십시오. 
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. 다음 명령을 실행하여 데이터베이스를 작성하십시오. 
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. 명령이 완료되고 나면 `wordpress.sql` 파일을 삭제하십시오. WordPress 설치를 웹 서버 문서 루트로 이동하십시오. 
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. WordPress 설정을 완료하고 플랫폼에 공개하십시오. 브라우저를 열고 `http://{yourVMPublicIPAddress}/wordpress`로 이동하십시오. VM의 공인 IP 주소를 대체하십시오. 이는 다음 이미지와 유사합니다.
   ![실행 중인 WordPress 사이트](images/solution4/WordPressSiteRunning.png)

## 도메인 구성

LAMP 서버에 기존 도메인 이름을 사용하려면 Virtual Server의 공인 IP 주소를 가리키도록 A 레코드를 업데이트하십시오.
서버의 공인 IP 주소는 대시보드에서 볼 수 있습니다. 

## 서버 모니터링 및 사용량

서버 가용성과 최상의 사용자 경험을 보장하기 위해, 모든 프로덕션 서버에서는 모니터링을 사용으로 설정해야 합니다. 이 절에서는 Virtual Server를 모니터하는 데 사용할 수 있는 옵션을 살펴보고 임의의 시점의 서버 사용량에 대해 알아봅니다. 

### 서버 모니터링

두 가지 기본 모니터링 유형(SERVICE PING 및 SLOW PING)을 사용할 수 있습니다. 

* **SERVICE PING**은 서버 응답 시간이 1초 이하인지 확인합니다. 
* **SLOW PING**은 서버 응답 시간이 5초 이하인지 확인합니다. 

SERVICE PING은 기본적으로 추가되어 있으므로, 다음 단계를 사용하여 SLOW PING을 추가하십시오. 

1. 대시보드의 디바이스 목록에서 서버를 선택한 후 **모니터링** 탭을 클릭하십시오.
  ![SLOW PING 모니터링](images/solution4/SlowPing.png)
2. **모니터 관리**를 클릭하십시오. 
3. **SLOW PING** 모니터링 옵션을 추가하고 **모니터 추가**를 클릭하십시오. IP 주소에 대해 자신의 공인 IP 주소를 선택하십시오.
  ![SLOW PING 모니터링 추가](images/solution4/AddSlowPing.png)

  **참고**: 동일한 구성으로 모니터를 복제하는 것은 허용되지 않습니다. 구성당 하나의 모니터만 작성할 수 있습니다. 

할당된 시간 범위 내에 응답이 수신되지 않는 경우에는 {{site.data.keyword.Bluemix_notm}} 계정의 이메일 주소로 경보가 전송됩니다.
  ![두 가지 모니터링](images/solution4/TwoMonitoring.png)

### 서버 사용량

현재 서버의 메모리 및 CPU 사용량을 알아보려면 **사용량** 탭을 선택하십시오.
  ![서버 사용량](images/solution4/ServerUsage.png)

## 서버 보안

{{site.data.keyword.BluVirtServers}}는 취약성 스캔 및 추가 방화벽과 같은 몇 가지 보안 옵션을 제공합니다. 

### 취약성 스캐너

취약성 스캐너는 서버와 관련된 취약성에 대해 서버를 스캔합니다. 서버에 대해 취약성 스캔을 실행하려면 아래 단계를 따르십시오. 

1. 대시보드에서 서버를 선택한 후 **보안** 탭을 클릭하십시오. 
2. **스캔**을 클릭하여 스캔을 시작하십시오. 
3. 스캔이 완료되고 나면 **스캔 완료**를 클릭하여 스캔 보고서를 보십시오.
  ![두 가지 모니터링](images/solution4/Vulnerabilities.png)
4. 보고된 취약성을 검토하십시오.
  ![두 가지 모니터링](images/solution4/VulnerabilityResults.png)

### 방화벽

서버를 보안하는 다른 방법은 방화벽을 추가하는 것입니다. 방화벽은 필수 보안 계층을 제공하며 원치 않는 트래픽이 서버에 도달하는 것을 방지하고, 공격 가능성을 줄이며, 서버 리소스가 의도한 대로 온전히 사용될 수 있게 해 줍니다. 방화벽 옵션은 요청 시 서비스 중단 없이 프로비저닝됩니다. 

방화벽은 인프라 공용 네트워크에 있는 모든 서버에서 추가 기능으로서 사용할 수 있습니다. 사용자는 주문 프로세스 중에 특정 디바이스에 고유한 하드웨어 또는 소프트웨어 방화벽을 선택하여 보호를 제공할 수 있습니다. 또는 환경에 전용 방화벽 어플라이언스를 배치하고 보호된 VLAN에 Virtual Server를 배치할 수도 있습니다. 자세한 정보는 [방화벽](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started)을 참조하십시오. 

## 리소스 제거

Virtual Server를 제거하려면 다음 단계를 완료하십시오. 

1. [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices)에 로그인하십시오. 
2. **디바이스** 메뉴에서 **디바이스 목록**을 선택하십시오. 
3. 제거할 Virtual Server에 대해 **조치**를 클릭하고 **취소**를 선택하십시오. 

## 관련 컨텐츠

* [Terraform을 사용한 LAMP 스택 배치](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
