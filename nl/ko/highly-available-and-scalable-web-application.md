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

# Virtual Server를 사용하여 스케일링 가능한 고가용성 웹 앱 빌드
{: #highly-available-and-scalable-web-application}

애플리케이션에 서버를 추가하는 것은 추가 로드를 처리하기 위한 일반적인 패턴입니다. 애플리케이션 가용성 및 복원성 향상을 위한 또 다른 주요 요소는 데이터 복제 및 로드 밸런싱을 사용하여 애플리케이션을 여러 구역 또는 위치에 배치하는 것입니다. 

이 튜토리얼은 다음 항목을 작성하는 시나리오를 안내합니다. 

- 댈러스의 두 웹 애플리케이션 서버
- 한 위치에 있는 두 개의 서버 간에 트래픽을 로드 밸런싱하기 위한 Cloud Load Balancer
- 하나의 MySQL 데이터베이스 서버
- 애플리케이션 파일 및 백업을 저장하기 위한 내구성 있는 File Storage
- 동일한 구성으로 두 번째 위치를 구성한 후, 한 사본에 장애가 발생하면 정상 위치로 트래픽을 유도할 수 있도록 Cloud Internet Services 추가

## 목표
{: #objectives}

* PHP 및 MySQL 설치를 위한 {{site.data.keyword.virtualmachinesshort}} 작성
* 애플리케이션 파일 및 데이터베이스 백업 보존을 위해 {{site.data.keyword.filestorage_short}} 사용
* 애플리케이션 서버에 요청을 분배하기 위해 {{site.data.keyword.loadbalancer_short}} 프로비저닝
* 복원성 향상 및 고가용성을 위한 두 번째 위치를 추가하여 솔루션 확장

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

이 애플리케이션은 MySQL 데이터베이스를 사용하는 단순한 PHP 프론트 엔드(WordPress 블로그)입니다. 몇 개의 프론트 엔드 서버가 요청을 처리합니다. 

<p style="text-align: center;">
  ![아키텍처 다이어그램](images/solution14/Architecture.png)
</p>

1. 사용자가 애플리케이션에 연결합니다. 
2. {{site.data.keyword.loadbalancer_short}}가 요청을 처리할 정상 서버 중 하나를 선택합니다. 
3. 선택된 서버가 공유 File Storage에 저장된 애플리케이션 파일에 액세스합니다. 
4. 해당 서버가 데이터베이스로부터 정보를 가져와 사용자에게 페이지를 렌더링합니다. 
5. 정기 간격마다 데이터베이스 컨텐츠가 백업됩니다. 마스터 서버에 장애가 발생하는 경우에 대비하여 대기 데이터베이스 서버가 사용 가능 상태를 유지합니다. 

## 시작하기 전에
{: #prereqs}

### VPN 액세스 구성

이 튜토리얼에서, 애플리케이션 사용자는 먼저 Load Balancer를 통해야 데이터에 액세스할 수 있습니다. {{site.data.keyword.virtualmachinesshort}}가 공용 인터넷에 표시될 필요는 없습니다. 따라서 이는 사설 IP 주소만으로 프로비저닝할 수 있으며 사용자는 VPN 연결을 사용하여 서버에서 작업할 수 있습니다. 

1. [VPN 액세스가 사용으로 설정되어 있는지 확인하십시오.](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking) 

     VPN 액세스를 사용으로 설정하려면 **마스터 사용자**여야 하며, 그렇지 않은 경우에는 마스터 사용자에게 액세스를 요청하십시오.
     {:tip}
2. [사용자 목록](https://{DomainName}/iam#/users)에서 사용자를 선택하여 VPN 액세스 인증 정보를 얻으십시오. 
3. [웹 인터페이스](https://www.softlayer.com/VPN-Access)를 통하거나 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 또는 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7)용 VPN 클라이언트를 사용하여 VPN에 로그인하십시오. 

이 단계를 건너뛰고 모든 서버가 공용 인터넷에 표시되도록 선택할 수도 있습니다(그러나 이를 사설로 유지하면 추가적인 보안 레벨을 제공할 수 있음). 이를 공개하려면 {{site.data.keyword.virtualmachinesshort}}를 프로비저닝할 때 **공용 및 사설 네트워크 업링크**를 선택하십시오.
{: tip}

### 계정 권한 확인

인프라 마스터 사용자에게 문의하여 다음 권한을 얻으십시오. 
- **공용 및 사설 네트워크 업링크**를 사용하는 {{site.data.keyword.virtualmachinesshort}}를 작성할 수 있도록 하는 **네트워크**(이 권한은 서버에 연결하는 데 VPN을 사용하는 경우 필요하지 않음)

## 하나의 데이터베이스용 서버 프로비저닝
{: #database_server}

이 절에서는 마스터 데이터베이스 역할을 수행할 하나의 서버를 구성합니다. 

1. {{site.data.keyword.Bluemix}} 콘솔의 카탈로그로 이동하여 인프라 섹션에서 [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)를 선택하십시오. 
2. **공용 Virtual Server**를 선택한 후 **작성**을 클릭하십시오. 
3. 다음 항목으로 서버를 구성하십시오. 
   - **이름**을 **db1**로 설정하십시오. 
   - 서버를 프로비저닝할 위치를 선택하십시오. **이 튜토리얼에서 작성되는 모든 기타 서버 및 리소스는 동일한 위치에서 작성해야 합니다. **
   - **Ubuntu Minimal** 이미지를 선택하십시오. 
   - 기본 컴퓨팅 flavor를 유지하십시오. 이 튜토리얼은 가장 작은 flavor로 테스트되었지만 모든 flavor에서 작동합니다. 
   - **연결된 스토리지 디스크**에서 25GB 부트 디스크를 선택하십시오. 
   - **네트워크 인터페이스**에서 **100Mbps 사설 네트워크 업링크** 옵션을 선택하십시오. 

     VPN 액세스를 구성하지 않은 경우에는 **100Mbps 공용 및 사설 네트워크 업링크** 옵션을 선택하십시오.
     {: tip}
   - 다른 구성 옵션을 검토하고 **프로비저닝**을 클릭하여 서버를 프로비저닝하십시오. 

      ![Virtual Server 구성](images/solution14/db-server.png)

   참고: 프로비저닝 프로세스에서 서버가 사용할 수 있도록 준비되는 데는 2 - 5분이 소요됩니다. 서버가 작성되고 나면 **디바이스 > 디바이스 목록**의 서버 세부사항 페이지에서 서버 인증 정보를 찾을 수 있습니다. 서버에 SSH로 접속하려면 서버의 사설 또는 공인 IP 주소, 사용자 이름 및 비밀번호가 필요합니다(디바이스 이름 옆에 있는 화살표 클릭).
   {: tip}

## MySQL 설치 및 구성
{: #mysql}

서버가 데이터베이스와 함께 제공되지는 않습니다. 이 절에서는 서버에 MySQL을 설치합니다. 

### MySQL 설치

1. SSH를 사용하여 서버에 연결하십시오. 
   ```sh
   ssh root@<Private-OR-Public-IP-Address>
   ```

   Virtual Server의 **위치**에 따라 올바른 [사이트 주소](https://www.softlayer.com/VPN-Access)로 VPN 클라이언트에 연결해야 된다는 점을 기억하십시오.
   {:tip}
2. MySQL을 설치하십시오. 
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   비밀번호를 입력하라는 프롬프트가 표시될 수 있습니다. 표시된 콘솔의 지시사항을 읽으십시오.
   {:tip}
3. MySQL 데이터베이스 보안에 도움을 주려면 다음 스크립트를 실행하십시오. 
   ```sh
   mysql_secure_installation
   ```

   몇 가지 옵션에 대한 프롬프트가 표시될 수 있습니다. 자신의 요구사항에 따라 신중하게 선택하십시오.
   {:tip}

### 애플리케이션을 위한 데이터베이스 작성

1. MySQL에 로그인하여 `wordpress`라는 데이터베이스를 작성하십시오. 
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. 데이터베이스에 대한 액세스 권한을 부여하고, database-username 및 database-password를 이전에 설정한 값으로 대체하십시오. 

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. 다음 명령을 사용하여 데이터베이스가 작성되었는지 확인하십시오. 

   ```sh
   show databases;
   ```

4. 다음 명령을 사용하여 데이터베이스를 종료하십시오. 

   ```sh
   exit
   ```

5. 데이터베이스 이름 및 비밀번호를 기록하십시오. 이는 애플리케이션 서버를 구성할 때 필요합니다. 

### 네트워크의 다른 서버에서 볼 수 있도록 MySQL 설정

기본적으로 MySQL은 로컬 인터페이스만 청취합니다. 애플리케이션 서버가 데이터베이스에 연결해야 하므로 사설 네트워크 인터페이스를 청취하도록 MySQL 구성을 변경해야 합니다. 

1. `nano /etc/mysql/my.cnf`를 사용하여 my.cnf 파일을 편집하고 다음 행을 추가하십시오. 
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. Ctrl+X를 사용하여 파일을 저장하고 종료하십시오. 

3. MySQL을 다시 시작하십시오. 

   ```sh
   systemctl restart mysql
   ```

4. 다음 명령을 실행하여 MySQL이 모든 인터페이스를 청취하고 있는지 확인하십시오. 
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## 데이터베이스 백업을 위한 File Storage 작성
{: #database_backup}

MySQL의 경우에는 백업을 수행하고 저장하는 데 다양한 방법이 있습니다. 이 튜토리얼에서는 crontab 항목을 사용하여 데이터베이스 컨텐츠를 디스크에 덤프합니다. 백업 파일은 File Storage에 저장됩니다. 이는 단순한 백업 메커니즘입니다. 프로덕션 환경에서 자신의 MySQL 데이터베이스 서버를 관리하려는 경우에는 [MySQL 문서에 설명되어 있는 백업 전략 중 하나를 구현](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html)하는 것이 좋습니다. 

### File Storage 작성
{: #create_for_backup}

1. {{site.data.keyword.Bluemix}} 콘솔의 카탈로그로 이동하여 [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)를 선택하십시오. 
2. **작성**을 클릭하십시오. 
3. 다음 항목으로 서비스를 구성하십시오. 
   - **스토리지 유형**를 **내구성**으로 설정하십시오. 
   - 데이터베이스 서버를 작성한 위치와 동일한 **위치**를 선택하십시오. 
   - 청구 방법을 선택하십시오. 
   - **스토리지 패키지**에서 **0.25IOPS/GB**를 선택하십시오. 
   - **스토리지 크기**에서 **20GB**를 선택하십시오. 
   - **스냅샷 공간 크기**를 **0GB**로 유지하십시오. 
   - 계속을 클릭하여 서비스를 작성하십시오. 

### File Storage를 사용할 수 있도록 데이터베이스 서버에 권한 부여

Virtual Server가 File Storage를 마운트하려면 권한이 부여되어야 합니다. 

1. [기존 항목의 목록](https://{DomainName}/classic/storage/file)에서 새로 작성된 File Storage를 선택하십시오. 
2. **권한 부여된 호스트**에서 **호스트 권한 부여**를 클릭하고 Virtual Server(데이터베이스)를 선택하십시오(**디바이스** > 디바이스 유형으로서의 Virtual Server > 서버의 이름 입력). 

### 데이터베이스 백업을 위한 File Storage 마운트

File Storage는 Virtual Server에 NFS 드라이브로서 마운트할 수 있습니다. 

1. NFS 클라이언트 라이브러리를 설치하십시오. 
   ```sh
   apt-get -y install nfs-common
   ```

2. 다음 명령을 실행하여 `/etc/systemd/system/mnt-database.mount`라는 파일을 작성하십시오. 
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. 다음 명령을 사용하여 mnt-database.mount를 편집하십시오. 
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. 아래 컨텐츠를 mnt-database.mount 파일에 추가하고 `What`의 `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT`를 File Storage의 **마운트 지점**(예: *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*)으로 대체하십시오. **마운트 지점** URL은 작성된 File Storage 서비스에서 가져올 수 있습니다. 
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
   Ctrl+X를 사용하여 저장 후 nano 창을 종료하십시오.
   {: tip}

5. 마운트 지점을 작성하십시오. 
  ```sh
  mkdir /mnt/database
  ```

6. 스토리지를 마운트하십시오. 
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. 마운트가 성공적으로 수행되었는지 확인하십시오. 
   ```sh
   mount
   ```
   마지막 행은 File Storage 마운트를 나열해야 합니다. 그렇지 않은 경우에는 `journalctl -xe`를 사용하여 마운트 오퍼레이션을 디버그하십시오.
   {: tip}

### 정기 간격의 백업 설정

1. 다음 명령에서 `CHANGE_ME`를 이전에 지정한 데이터베이스 비밀번호로 대체하여 `/root/dbbackup.sh` 쉘 스크립트를 작성(`touch` 및 `nano` 사용)하십시오. 
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip > /mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. 해당 파일이 실행 가능한지 확인하십시오. 
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. crontab을 편집하십시오. 
   ```sh
   crontab -e
   ```
4. 백업이 매일 오후 11시에 수행되도록 하려면 컨텐츠를 다음과 같이 설정하고 파일을 저장한 후 편집기를 닫으십시오. 
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## PHP 애플리케이션을 위한 두 개의 서버 프로비저닝
{: #app_servers}

이 절에서는 두 개의 애플리케이션 서버를 작성합니다. 

1. {{site.data.keyword.Bluemix}} 콘솔의 카탈로그로 이동하여 인프라 섹션에서 [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) 서비스를 선택하십시오. 
2. **공용 Virtual Server**를 선택한 후 **작성**을 클릭하십시오. 
3. 다음 항목으로 서버를 구성하십시오. 
   - **이름**을 **app1**로 설정하십시오. 
   - 데이터베이스 서버를 프로비저닝한 위치와 동일한 위치를 선택하십시오. 
   - **Ubuntu Minimal** 이미지를 선택하십시오. 
   - 기본 컴퓨팅 flavor를 유지하십시오. 
   - **연결된 스토리지 디스크**에서 25GB를 부트 디스크로 선택하십시오. 
   - **네트워크 인터페이스**에서 **100Mbps 사설 네트워크 업링크** 옵션을 선택하십시오. 

     VPN 액세스를 구성하지 않은 경우에는 **100Mbps 공용 및 사설 네트워크 업링크** 옵션을 선택하십시오.
     {: tip}
   - 다른 구성 옵션을 검토하고 **프로비저닝**을 클릭하여 서버를 프로비저닝하십시오.
     ![Virtual Server 구성](images/solution14/db-server.png)
4. 1 - 3단계를 반복하여 이름이 **app2**인 다른 Virtual Server를 프로비저닝하십시오. 

## 애플리케이션 서버 간에 파일을 공유하는 데 필요한 File Storage 작성
{: shared_storage}

이 File Storage는 *app1* 및 *app2* 서버 간에 애플리케이션 파일을 공유하는 데 사용됩니다. 

### File Storage 작성
{: #create_for_sharing}

1. {{site.data.keyword.Bluemix}} 콘솔의 카탈로그로 이동하여 [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)를 선택하십시오. 
2. **작성**을 클릭하십시오. 
3. 다음 항목으로 서비스를 구성하십시오. 
   - **스토리지 유형**를 **내구성**으로 설정하십시오. 
   - 애플리케이션 서버를 작성한 위치와 동일한 **위치**를 선택하십시오. 
   - 청구 방법을 선택하십시오. 
   - **스토리지 패키지**에서 **2IOPS/GB**를 선택하십시오. 
   - **스토리지 크기**에서 **20GB**를 선택하십시오. 
   - **스냅샷 크기**에서 **20GB**를 선택하십시오. 
   - 계속을 클릭하여 서비스를 작성하십시오. 

### 정기 스냅샷 구성

[스냅샷](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots)은 사용자에게 성능에 영향을 주지 않으면서 데이터를 보호하는 편리한 방법을 제공합니다. 스냅샷을 다른 데이터 센터에 복제할 수도 있습니다. 

1. [기존 항목의 목록](https://{DomainName}/classic/storage/file)에서 File Storage를 선택하십시오. 
2. **스냅샷 스케줄**에서 스냅샷 스케줄을 편집하십시오. 스케줄은 다음과 같이 정의할 수 있습니다. 
   1. 분을 30으로 설정하고 마지막 24개 스냅샷을 보존하도록 하여 시간별 스냅샷을 추가하십시오. 
   2. 시간을 오후 11시로 설정하고 마지막 7개 스냅샷을 보존하도록 하여 일별 스냅샷을 추가하십시오. 
   3. 시간을 오전 1시로 설정하고 마지막 4개 스냅샷을 보존하도록 하여 주별 스냅샷을 추가하고 저장을 클릭하십시오.
      ![백업 스냅샷](images/solution14/snapshots.png)

### File Storage를 사용할 수 있도록 애플리케이션 서버에 권한 부여

1. **권한 부여된 호스트**에서 **호스트 권한 부여**를 클릭하여 애플리케이션 서버(app1 및 app2)에 이 File Storage를 사용할 수 있도록 권한을 부여하십시오. 

### File Storage 마운트

각 애플리케이션 서버(app1 및 app2)에서 다음 단계를 반복하십시오. 

1. NFS 클라이언트 라이브러리를 설치하십시오. 
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. `touch /etc/systemd/system/mnt-www.mount`를 사용하여 파일을 작성하고 `nano /etc/systemd/system/mnt-www.mount`를 사용하여 다음 컨텐츠로 편집하면서 `What`의 `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT`를 File Storage의 **마운트 지점**(예: *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*)으로 대체하십시오. 마운트 지점은 [File Storage 볼륨의 목록](https://{DomainName}/classic/storage/file)에서 찾을 수 있습니다. 
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
3. 마운트 지점을 작성하십시오. 
   ```sh
   mkdir /mnt/www
   ```
4. 스토리지를 마운트하십시오. 
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. 마운트가 성공적으로 수행되었는지 확인하십시오. 
   ```sh
   mount
   ```
   마지막 행은 File Storage 마운트를 나열해야 합니다. 그렇지 않은 경우에는 `journalctl -xe`를 사용하여 마운트 오퍼레이션을 디버그하십시오.
   {: tip}

최종적으로는 [프로비저닝 스크립트](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script)를 사용하거나 [이미지 캡처](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template)를 통해 서버 구성과 관련된 모든 단계를 자동화할 수 있습니다.
{: tip}

## 애플리케이션 서버에서 PHP 애플리케이션 설치 및 구성
{: #php_application}

이 튜토리얼에서는 WordPress 블로그를 설정합니다. 모든 WordPress 파일은 두 애플리케이션 서버가 모두 액세스할 수 있도록 공유 File Storage에 설치됩니다. WordPress를 설치하기 전에 먼저 웹 서버 및 PHP 런타임을 구성해야 합니다. 

### nginx 및 PHP 설치

각 애플리케이션 서버에서 다음 단계를 반복하십시오. 

1. nginx를 설치하십시오. 
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. PHP 및 MySQL 클라이언트를 설치하십시오. 
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. PHP 서비스 및 nginx를 중지하십시오. 
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. `nano /etc/nginx/sites-available/default`를 다음 내용과 함께 사용하여 컨텐츠를 대체하십시오. 
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
5. 두 개의 앱 서버 중 하나의 `/mnt/www` 디렉토리에서 다음 명령을 사용하여 `html` 폴더를 작성하십시오. 
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### WordPress 설치 및 구성

WordPress 가 File Storage 마운트에 설치될 것이므로, 사용자는 서버 중 하나에서 다음 단계만 수행하면 됩니다. 여기서는 **app1**이 선택되었습니다. 

1. WordPress 설치 파일을 검색하십시오. 

   애플리케이션 서버에 공용 네트워크 링크가 있는 경우에는 WordPress 파일을 Virtual Server에서 직접 다운로드할 수 있습니다. 

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   Virtual Server에 사설 네트워크 링크만 있는 경우에는 인터넷 액세스가 가능한 다른 시스템에서 설치 파일을 검색한 후 이를 Virtual Server로 복사해야 합니다. WordPress 설치 파일을 https://wordpress.org/latest.tar.gz에서 검색한 경우에는 `scp`를 사용하여 이를 Virtual Server로 복사할 수 있습니다. 

   ```sh
   scp latest.tar.gz root@PRIVATE_IP_ADDRESS_OF_THE_SERVER:/tmp
   ```
   `latest`를 WordPress 웹 사이트에서 다운로드한 파일 이름으로 대체하십시오.
   {: tip}

   SSH로 Virtual Server에 접속한 후 `tmp` 디렉토리로 변경하십시오. 

   ```sh
   cd /tmp
   ```

2. 설치 파일의 압축을 푸십시오. 

   ```
   tar xzvf latest.tar.gz
   ```

3. WordPress 파일을 준비하십시오. 
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. 공유 File Storage에 파일을 복사하십시오. 
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. 권한을 설정하십시오. 
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -exec chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. 다음 웹 서비스를 호출하고 `nano`를 사용하여 결과를 `/mnt/www/html/wp-config.php`에 삽입하십시오. 
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   Virtual Server에 공용 네트워크 링크가 없는 경우에는 웹 브라우저에서 https://api.wordpress.org/secret-key/1.1/salt/를 열 수 있습니다. 

7. `nano /mnt/www/html/wp-config.php`를 사용하여 데이터베이스 인증 정보를 설정하고 이를 업데이트하십시오. 

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   WordPress가 구성되었습니다. 설치를 완료하려면 WordPress 사용자 인터페이스에 액세스해야 합니다. 

두 애플리케이션 서버에서 웹 서버 및 PHP 런타임을 시작하십시오. 
7. 다음 명령을 실행하여 서비스를 시작하십시오. 

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

*app1* 또는 *app2*의 사설 IP 주소(VPN 연결을 사용하는 경우) 또는 공인 IP 주소를 사용하여 `http://YourAppServerIPAddress/`에서 WordPress 설치에 액세스하십시오.
![Virtual Server 구성](images/solution14/wordpress.png)

사설 네트워크 링크만 사용하여 애플리케이션 서버를 구성한 경우에는 WordPress 관리자 콘솔에서 직접 WordPress 플러그인, 테마 또는 업그레이드를 설치할 수 없습니다. 이 경우에는 WordPress 사용자 인터페이스를 통해 파일을 업로드해야 합니다.
{: tip}

## 애플리케이션 서버 앞에 하나의 Load Balancer 서버 프로비저닝
{: #load_balancer}

이 시점에서, 사용자에게는 IP 주소가 서로 다른 두 개의 애플리케이션 서버가 있습니다. 사설 네트워크 업링크만 프로비저닝하도록 선택하는 경우에는 이들이 공용 인터넷에도 표시되지 않을 수 있습니다. 이러한 서버 앞에 Load Balancer를 추가하면 애플리케이션이 공개됩니다. 이 Load Balancer 또한 기반 인프라는 사용자에게서 숨깁니다. 이 Load Balancer는 애플리케이션의 상태를 모니터하고 수신 요청을 정상 서버에 디스패치합니다. 

1. 카탈로그로 이동하여 [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer)를 작성하십시오. 
2. **플랜** 단계에서 *app1* 및 *app2*와 동일한 데이터 센터를 선택하십시오. 
3. **네트워크 설정**에서 다음 작업을 수행하십시오. 
   1. *app1* 및 *app2*가 프로비저닝된 서브넷과 동일한 서브넷을 선택하십시오. 
   2. Load Balancer 공인 IP에 대해 기본 IBM 시스템 풀을 사용하십시오. 
4. **기본**에서 다음 작업을 수행하십시오. 
   1. Load Balancer의 이름을 지정하십시오(예: **app-lb-1**). 
   2. 기본 프로토콜 구성을 유지하십시오(기본적으로 Load Balancer는 HTTP에 대해 구성됨).
      SSL 프로토콜은 사용자의 고유 인증서를 사용하여 지원됩니다. [Load Balancer로 SSL 인증서 가져오기](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates)를 참조하십시오.
      {: tip}
5. **서버 인스턴스**에서 *app1* 및 *app2* 서버를 추가하십시오. 
6. 검토하고 작성하여 마법사를 완료하십시오. 

### Load Balancer URL을 사용하도록 WordPress 구성 변경

Load Balancer 주소를 사용하도록 WordPress 구성을 변경해야 합니다. WordPress는 [블로그 URL에 대한 참조를 유지하고 이 위치를 페이지에 삽입](https://codex.wordpress.org/Settings_General_Screen)합니다. 이 설정을 변경하지 않으면 WordPress가 사용자를 백엔드 서버로 직접 경로 재지정함으로써 Load Balancer를 우회하며, 서버에 사설 IP 주소만 있는 경우에는 아예 작동하지 않습니다. 

1. Load Balancer의 세부사항 페이지에서 주소를 찾으십시오. 작성한 Load Balancer는 [네트워크 / 로드 밸런싱 / 로컬](https://{DomainName}/classic/network/loadbalancing/cloud)에서 찾을 수 있습니다. 

   DNS 구성에서 Load Balancer 주소를 가리키는 CNAME 레코드를 추가하여 Load Balancer에 고유 도메인 이름을 사용할 수도 있습니다.
   {: tip}
2. *app1* 또는 *app2* URL을 통해 WordPress 블로그에 관리자로 로그인하십시오. 
3. 설정 / 일반에서 WordPress 주소(URL) 및 사이트 주소(URL)를 모두 Load Balancer 주소로 설정하십시오. 
4. 설정을 저장하십시오. WordPress가 Load Balancer 주소로의 경로 재지정을 시작합니다.
   DNS 전파로 인해 Load Balancer 주소가 활성 상태가 되기까지는 어느 정도 시간이 소요될 수 있습니다.
   {: tip}

### Load Balancer 작동 테스트

Load Balancer는 서버의 상태를 확인하고 정상 서버로만 사용자를 경로 재지정하도록 구성됩니다. Load Balancer가 어떻게 작동하고 있는지 파악하기 위해 다음 작업을 수행할 수 있습니다. 

1. 다음 명령을 사용하여 *app1* 및 *app2* 둘 다에서 nginx 로그를 감시하십시오. 
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   이미 서버 상태를 확인하기 위한 Load Balancer의 정기 ping이 표시되고 있어야 합니다.
   {: tip}
2. Load Balancer 주소를 통해 WordPress에 액세스하고 페이지를 다시 로드하십시오. nginx 로그에서 *app1* 및 *app2*이 모두 페이지를 위한 컨텐츠를 서비스하고 있음을 확인하십시오. Load Balancer가 예상대로 트래픽을 두 서버로 경로 재지정하고 있습니다. 

3. *app1*에서 nginx를 중지하십시오. 
   ```sh
   systemctl nginx stop
   ```

4. 잠시 후 WordPress 페이지를 다시 로드하십시오. 모든 히트가 *app2*로 전달되는 것을 볼 수 있습니다. 

5. *app2*에서 nginx를 중지하십시오. 

6. WordPress 페이지를 다시 로드하십시오. 정상 서버가 없어 Load Balancer가 오류를 리턴합니다. 

7. *app1*에서 nginx를 다시 시작하십시오. 
   ```sh
   systemctl nginx start
   ```

8. Load Balancer가 *app1*이 정상 상태인 것을 발견하면 트래픽을 이 서버로 경로 재지정합니다. 

## 두 번째 위치로 솔루션 확장(선택사항)
{: #secondregion}

복원성 및 가용성을 향상시키기 위해, 두 번째 위치를 사용하여 인프라 설정을 확장하고 애플리케이션이 두 개의 위치에서 실행되도록 할 수 있습니다. 

두 번째 위치 배치를 사용하는 경우 아키텍처는 다음과 같습니다. 

<p style="text-align: center;">

  ![아키텍처 다이어그램](images/solution14/Architecture2.png)
</p>

1. 사용자가 IBM Cloud Internet Services(CIS)를 통해 애플리케이션에 액세스합니다. 
2. CIS가 트래픽을 정상 위치로 라우팅합니다. 
3. 하나의 위치에서 Load Balancer가 트래픽을 서버로 경로 재지정합니다. 
4. 애플리케이션이 데이터베이스에 액세스합니다. 
5. 애플리케이션이 File Storage에서 미디어 자산을 저장하고 검색합니다. 

이 아키텍처를 구현하려면 두 번째 위치에서 다음 작업을 수행해야 합니다. 

- 새 위치에서 이전 단계의 모든 작업을 반복하십시오. 
- 각 위치의 두 MySQL 서버 사이에 데이터 복제를 설정하십시오. 
- [다른 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)에 설명되어 있는 바와 같이 각 위치의 정상 서버 간에 트래픽을 분배하기 위해 IBM Cloud Internet Services를 구성하십시오. 

## 리소스 제거
{: #removeresources}

1. Load Balancer를 삭제하십시오. 
2. *db1*, *app1* 및 *app2*를 취소하십시오. 
3. 두 File Storage 서비스를 삭제하십시오. 
4. 두 번째 위치가 구성된 경우에는 모든 리소스 및 Cloud Internet Services를 삭제하십시오. 

## 관련 컨텐츠
{: #related}

- Load Balancer 앞에 Content Delivery Network를 사용하면 백엔드 서버의 로드를 줄여 애플리케이션이 서비스하는 정적 컨텐츠가 혜택을 볼 수 있습니다. Content Delivery Network를 구현하는 튜토리얼은 [Object Storage 및 CDN을 사용한 정적 파일 전달 가속화](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)를 참조하십시오. 
- 이 튜토리얼에서는 두 개의 서버를 프로비저닝했으나, 추가 로드를 처리하기 위해 서버를 자동으로 추가할 수도 있습니다. [자동 스케일링](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale)은 비즈니스 애플리케이션을 지원하기 위해 Virtual Server를 추가 또는 제거하는 작업과 연관된 수동 스케일링 프로세스를 자동화하는 기능을 제공합니다. 
- 가용성을 향상시키고 재해 복구 옵션을 늘리기 위해, File Storage는 컨텐츠의 [자동 정기 스냅샷 작성](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots)과 다른 데이터 센터로의 [복제](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication)를 수행하도록 구성될 수 있습니다. 
