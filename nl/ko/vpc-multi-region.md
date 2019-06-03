---
copyright:
  years: 2019
lastupdated: "2019-04-02"
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
{:important: .important}

# 여러 위치 및 구역에 격리된 워크로드 배치
{: #vpc-multi-region}

IBM은 2019년 4월 초부터(다음 달에 연장 사용이 개방됨) 제한된 수의 고객만 VPC에 대한 조기 액세스 프로그램에 참여하도록 승인합니다. 조직에서 IBM Virtual Private Cloud에 대한 액세스를 확보하길 원하는 경우에는 이 [추천 양식](https://{DomainName}/vpc){: new_window}에 기입을 완료하면 IBM 담당자가 다음 단계와 관련하여 연락을 줍니다.
{: important}

이 튜토리얼에서는 다양한 IBM Cloud 지역에 VPC를 프로비저닝하여 격리된 워크로드를 설정하는 단계에 대해 설명합니다. 서브넷 및 가상 서버 인스턴스(VSI)가 있는 지역. 백엔드 풀, 프론트 엔드 리스너 및 적절한 상태 검사를 사용하여 로드 밸런서를 구성하여 지역 내에서 및 글로벌로 복원성을 증가시키기 위해 지역의 여러 구역에서 이 VSI가 작성됩니다. 

글로벌 로드 밸런서의 경우 카탈로그에서 IBM Cloud Internet Services(CIS) 서비스를 프로비저닝하며 모든 수신 HTTPS 요청에 대한 SSL 인증서를 관리하기 위해 {{site.data.keyword.cloudcerts_long_notm}} 카탈로그 서비스가 작성되고 개인 키와 함께 인증서를 가져옵니다. 

{:shortdesc}

## 목표
{: #objectives}

* 가상 프라이빗 클라우드에 사용 가능한 인프라 오브젝트를 통한 워크로드 격리에 대해 이해합니다. 
* 지역 내 구역 사이에서 로드 밸런서를 사용하여 가상 서버 사이에 트래픽을 분배합니다. 
* 지역 사이에서 글로벌 로드 밸런서를 사용하여 복원성을 늘리고 대기 시간을 줄입니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.loadbalancer_full}}](https://{DomainName}/vpc/provision/loadBalancer)
- IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
- [{{site.data.keyword.cloudcerts_long_notm}}](https://{DomainName}/catalog/services/cloudcerts)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/estimator/review)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

  ![아키텍처](images/solution41-vpc-multi-region/Architecture.png)

1. 관리자(DevOps)가 지역 1의 VPC에 있는 두 개의 다른 구역 아래의 서브넷에 VSI를 프로비저닝하고 지역 2에서 작성된 VPC에서 동일하게 반복합니다. 
2. 관리자가 프론트 엔드 리스너 및 지역 1의 서로 다른 구역에 있는 서브넷 서버의 백엔드 풀과 함께 로드 밸런서를 작성합니다. 지역 2에서 동일하게 반복합니다. 
3. 관리자가 클라우드 인터넷 서비스에 연관된 사용자 정의 도메인을 프로비저닝하고 두 개의 다른 VPC에서 작성된 로드 밸런서를 가리키는 글로벌 로드 밸런서를 작성합니다. 
4. 관리자가 도메인 SSL 인증서를 인증서 관리자 서비스에 추가하여 HTTPS 암호화를 사용으로 설정합니다. 
5. 인터넷 사용자가 HTTP/HTTPS 요청을 작성하고 글로벌 로드 밸런서가 요청을 처리합니다. 
6. 요청이 글로벌 및 로컬로 로드 밸런서에 라우팅됩니다. 그런 다음 사용 가능한 서버 인스턴스에 의해 요청이 이행됩니다. 

## 시작하기 전에
{: #prereqs}

- 사용자 권한을 확인하십시오. 사용자 계정에 VPC 리소스를 작성하고 관리할 수 있는 충분한 권한이 있는지 확인하십시오. 필요한 권한의 목록은 [VPC 사용자를 위해 필요한 권한 부여](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources)를 참조하십시오. 
- 가상 서버에 연결하기 위해 SSH 키가 필요합니다. SSH 키가 없으면 [키 작성을 위한 지시사항](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)을 참조하십시오. 
- Cloud Internet Services를 사용하려면 이 도메인에 대한 DNS가 Cloud Internet Services 이름 서버를 가리키도록 구성할 수 있도록 사용자 정의 도메인을 소유하고 있어야 합니다. 도메인을 소유하고 있지 않으면 등록 담당자(예: [godaddy.com](http://godaddy.com/))로부터 도메인을 구입할 수 있습니다. 

## VPC, 서브넷 및 VSI 작성
{: #create-infrastructure}

이 절에서는 지역 1의 두 개의 다른 구역에서 작성된 서브넷을 사용하여 지역 1에서 자체 VPC를 작성한 후 VSI를 프로비저닝합니다. 

지역 1에서 자체 {{site.data.keyword.vpc_short}}를 작성하려면 다음을 수행하십시오. 

1. [VPC 개요](https://{DomainName}/vpc/overview) 페이지로 이동하여 **VPC 작성**을 클릭하십시오. 
2. **새 가상 프라이빗 클라우드** 섹션 아래에서 다음을 수행하십시오. 
   * **vpc-region1**을 VPC의 이름으로 입력하십시오. 
   * **리소스 그룹**을 선택하십시오. 
   * 선택적으로 **태그**를 추가하여 리소스를 구성하십시오. 
3. **새 기본값 작성(모두 허용)**을 VPC 기본 액세스 제어 목록(ACL)으로 선택하십시오. 
4. SSH를 선택 취소하고 **기본 보안 그룹**에서 ping을 실행한 후 **클래식 액세스**를 선택 취소된 상태로 두십시오. 
5. **VPC에 대한 새 서브넷** 아래에서 다음을 수행하십시오. 
   * 고유 이름으로 **vpc-region1-zone1-subnet**을 입력하십시오. 
   * 위치(예: 댈러스)를 선택하여 이를 **지역 1**이라고 하고 지역 1에 있는 구역(예: 댈러스 1)을 선택하여 이를 **구역 1**이라고 해 봅니다. 
   * CIDR 표기법으로 서브넷의 IP 범위를 입력하십시오(예: **10.xxx.0.0/24**). **주소 접두부**는 그대로 두고 **주소 수**를 256으로 선택하십시오. 
6. 서브넷 액세스 제어 목록(ACL)에 대해 **VPC 기본값 사용**을 선택하십시오. 나중에 인바운드 및 아웃바운드 규칙을 구성할 수 있습니다. 
7. 서브넷에 있는 모든 가상 서버 인스턴스에 유동 IP가 접속된다는 점을 고려하면 서브넷에 대해 퍼블릭 게이트웨이를 사용으로 설정하지 않아도 됩니다. 가상 서버 인스턴스는 유동 IP를 통해 인터넷에 연결됩니다. 
8. **가상 프라이빗 클라우드 작성**을 클릭하여 인스턴스를 프로비저닝하십시오. 

서브넷 작성을 확인하려면 왼쪽 분할창에서 **서브넷**을 클릭한 후 상태가 **사용 가능**으로 변경될 때까지 기다리십시오. **서브넷** 아래에서 새 서브넷을 작성할 수 있습니다. 

### 구역 2에서 서브넷 작성

1. **새 서브넷**을 클릭한 후 **vpc-region1-zone2-subnet**을 서브넷의 고유 이름으로 입력하고 **vpc-region1**을 VPC로 선택하십시오. 
2. 위에서 지역 1이라고 부르는 위치(예: 댈러스)를 선택한 후 지역 1의 다른 구역(예: 댈러스 2)을 선택하여 선택한 이 구역을 **구역 2**라고 해 봅니다. 
3. CIDR 표기법으로 서브넷의 IP 범위를 입력하십시오(예: **10.xxx.64.0/24**). **주소 접두부**는 그대로 두고 **주소 수**를 256으로 선택하십시오. 
4. 서브넷 액세스 제어 목록(ACL)에 대해 **VPC 기본값 사용**을 선택하십시오. 

### VSI 프로비저닝
서브넷의 상태가 **사용 가능**으로 변경되고 나면 다음을 수행하십시오. 

1. **vpc-region1-zone1-subnet**을 클릭하고 **접속된 인스턴스**를 클릭한 후 **새 인스턴스**를 클릭하십시오. 
2. 고유 이름을 입력한 후 **vpc-region1-zone1-vsi**를 선택하십시오. 그런 다음 이전과 같이 **구역**과 함께 **위치** 및 이전에 작성한 VPC를 선택하십시오. 
3. **Ubuntu Linux** 이미지를 선택하고 **모든 프로파일**을 클릭한 후 **컴퓨팅** 아래에서 **c-2x4**(2개의 vCPU 및 4GB RAM)를 선택하십시오. 
4. **SSH 키**에 대해 초기에 작성한 SSH 키를 선택하십시오. 
5. **네트워크 인터페이스** 아래에서 보안 그룹 옆의 **편집** 아이콘을 클릭하십시오. 
   * **vpc-region1-zone1-subnet**이 서브넷으로 선택되어 있는지 확인하십시오. 선택되어 있지 않은 경우에는 이를 서브넷으로 선택하십시오. 
   * **저장**을 클릭하십시오. 
   * **가상 서버 인스턴스 작성**을 클릭하십시오. 
6.  VSI의 상태가 **전원 켜짐**으로 변경될 때까지 기다리십시오. 그런 다음 VSI **vpc-region1-zone1-vsi**를 선택하고 **네트워크 인터페이스**까지 스크롤하여 **유동 IP** 아래에서 **예약**을 클릭하여 공인 IP 주소를 VSI에 연관시키십시오. 향후 참조를 위해 연관된 IP 주소를 클립보드에 저장하십시오. 
7. 1단계 - 6단계를 **반복**하여 **지역 1**의 **구역 2**에 VSI를 프로비저닝하십시오. 

왼쪽 분할창의 **네트워크** 아래에 있는 **서브넷** 및 **VPC**로 이동한 후 위의 동일한 이름 지정 규칙을 따라 **region2**에 VSI 및 서브넷을 가진 새 VPC를 프로비저닝하기 위해 위의 단계를 **반복**하십시오. 

## VSI에서 웹 서버 설치 및 구성
{: #install-configure-web-server-vsis}

유지보수 보안 그룹 및 `점프` 서버 역할을 수행하는 배스천 호스트를 사용하여 서버를 안전하게 유지보수하기 위해 [배스천 호스트를 사용하여 원격 인스턴스에 안전하게 액세스](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)에 언급된 단계를 수행하십시오.
{:tip}

SSH를 통해 지역 1에 있는 구역 1의 서브넷에 프로비저닝된 서버에 연결되고 나면 다음을 수행하십시오. 

1. 프롬프트에서 아래 명령을 실행하여 Nginx를 웹 서버로 설치하십시오. 
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   {:codeblock}
2. 다음 명령을 사용하여 Nginx 서비스의 상태를 확인하십시오. 
   ```
   sudo systemctl status nginx
   ```
   {:pre}
   출력에는 Nginx 서비스가 **활성** 상태이고 실행 중이라고 표시되어야 합니다.
3. 트래픽(요청)을 수신하기 위해 **HTTP(80)** 및 **HTTPS(443)** 포트를 열어야 합니다. [UFW](https://help.ubuntu.com/community/UFW) - `sudo ufw enable`을 통해 방화벽을 조정하고 두 포트 모두에 대한 규칙이 포함된 ‘Nginx Full’ 프로파일을 사용으로 설정하여 이를 수행할 수 있습니다. 
   ```
   sudo ufw allow 'Nginx Full'
   ```
   {:pre}
4. Nginx가 예상대로 작동하는지 확인하기 위해 선택한 브라우저에서 `http://FLOATING_IP`를 열면 기본 Nginx 시작 페이지가 표시되어야 합니다. 
5. 지역 및 구역 세부사항으로 html 페이지를 업데이트하려면 아래 명령을 실행하십시오. 
   ```
   nano /var/www/html/index.nginx-debian.html
   ```
   {:pre}
   지역 및 구역(예: _**지역 1의 구역 1**에서 실행 중인 서버_)을 `Welcome to nginx!`라는 `h1` 태그에 추가하고 변경사항을 저장하십시오.
6. Nginx 서버를 다시 시작하여 변경사항을 반영하십시오. 
   ```
   sudo systemctl restart nginx
   ```
   {:pre}

1단계 - 6단계를 **반복**하여 모든 구역의 서브넷에 있는 VSI에서 웹 서버를 구성하고 각각의 해당 구역 정보로 html을 업데이트하는 것을 잊지 마십시오. 


## 로드 밸런서를 사용하여 구역 사이에서 트래픽 분배
{: #distribute-traffic-with-load-balancers}

이 절에서는 두 개의 로드 밸런서를 작성합니다. 각각의 지역에 하나씩 있는 이 로드 밸런서는 서로 다른 구역에 있는 각각의 해당 서브넷 아래의 여러 서버 인스턴스 사이에서 트래픽을 분배합니다. 

### 로드 밸런서 구성

1. **로드 밸런서**로 이동한 후 **새 로드 밸런서**를 클릭하십시오. 
2. **vpc-lb-region1**에 고유 이름을 제공하고 **vpc-region1**을 가상 프라이빗 클라우드로 선택한 후 VPC가 작성된 리소스 그룹을 선택하고 **공용**을 유형으로 선택하고 **region1**을 지역으로 선택하십시오. 
3. **지역 1**의 **구역 1** 및 **구역 2** 모두의 사설 IP를 선택하십시오. 
4. 풀로 라우팅된 트래픽을 공유하기 위해 동등 피어 역할을 수행하는 VSI의 새 백엔드 풀을 작성하십시오. 아래의 값을 사용하여 매개변수를 설정하고 **작성**을 클릭하십시오. 
	- **이름**: region1-pool
	- **프로토콜**: HTTP
	- **메소드**: 라운드 로빈
	- **세션 연결 유지**: 없음
	- **상태 검사 경로**: /
	- **상태 프로토콜**: HTTP
	- **간격(초)**: 15
	- **제한시간(초)**: 2
	- **최대 재시도 횟수**: 2
5. **첨부**를 클릭하여 서버 인스턴스를 region1-pool에 추가하십시오. 
   - **vpc-region1-zone1-subnet**의 사설 IP를 선택하고 작성한 인스턴스를 선택한 후 80을 포트로 설정하십시오. 
   - **추가**를 클릭한 후 이번에는 **vpc-region1-zone2-subnet**의 사설 IP를 선택하고 인스턴스를 선택한 후 80을 포트로 설정하십시오. 
   - **첨부**를 클릭하여 백엔드 풀 작성을 완료하십시오. 
6. **새 리스너**를 클릭하여 새 프론트 엔드 리스너를 작성하십시오. 리스너는 연결 요청을 확인하는 프로세스입니다. 
   - **프로토콜**: HTTP
   - **포트**: 80
   - **백엔드 풀**: region1-pool
   - **최대 연결 수**: 공백으로 두고 **작성**을 클릭하십시오. 
7. **로드 밸런서 작성**을 클릭하여 로드 밸런서를 프로비저닝하십시오. 

### 로드 밸런서 테스트

1. 로드 밸런서의 상태가 **활성**으로 변경될 때까지 기다리십시오. 
2. 웹 브라우저에서 **주소**를 여십시오. 
3. 페이지를 여러 번 새로 고쳐 새로 고칠 때마다 로드 밸런서가 다른 서버에 연결되는지 확인하십시오. 
4. 향후 참조를 위해 주소를 **저장**하십시오. 

관찰하는 경우 요청은 암호화되지 않으며 HTTP만 지원합니다. 다음 절에서 SSL 인증서를 구성하고 HTTPS를 사용으로 설정합니다. 

**지역 2**에서 위의 1단계 - 7단계를 **반복**하십시오. 

## HTTPS를 사용하여 VPC 내 트래픽 보안
{: #secure_https}

HTTPS 리스너를 추가하기 전에 SSL 인증서를 생성하고 인증서를 보유하여 인프라 서비스에 맵핑할 위치인 사용자 도메인의 진위를 확인하십시오. 

### CIS 서비스 프로비저닝 및 사용자 정의 도메인 구성

이 절에서는 IBM Cloud Internet Services(CIS) 서비스를 작성하고 CIS 이름 서버를 가리키도록 사용자 정의 도메인을 구성한 후 나중에 글로벌 로드 밸런서를 구성합니다. 

1. {{site.data.keyword.Bluemix_notm}} 카탈로그의 [Internet Services](https://{DomainName}/catalog/services/internet-services)로 이동하십시오. 
2. 서비스 이름을 설정한 후 **작성**을 클릭하여 서비스의 인스턴스를 작성하십시오. 이 튜토리얼에 대한 가격 플랜을 사용할 수 있습니다. 
3. 서비스 인스턴스가 프로비저닝되면 **시작합니다**를 클릭하여 도메인 이름을 설정하고 **도메인 추가**를 클릭하십시오. 
4. **다음 단계**를 클릭하십시오. 이름 서버가 지정되면 나열된 이름 서버를 사용하도록 등록 담당자 또는 도메인 이름 제공자를 구성하십시오. 
5. 등록 담당자 또는 DNS 제공자를 구성하고 나면 변경사항을 적용하는 데 최대 24시간이 필요할 수 있습니다. 

   개요 페이지에서 도메인의 상태가 *보류 중*에서 *활성*으로 변경되면 `dig <YOUR_DOMAIN_NAME> ns` 명령을 사용하여 새 이름 서버가 적용되었음을 확인할 수 있습니다.
   {:tip}

글로벌 로드 밸런서와 함께 사용하려고 하는 도메인 및 하위 도메인에 대한 SSL 인증서를 확보해야 합니다. mydomain.com이라는 도메인을 사용한다고 가정하면 `lb.mydomain.com`에서 글로벌 로드 밸런서가 호스팅될 수 있습니다. lb.mydomain.com에 대한 인증서가 발행되어야 합니다. 

[Let's Encrypt](https://letsencrypt.org/)에서 무료 SSL 인증서를 얻을 수 있습니다. 이 프로세스 동안 도메인의 소유자임을 증명하기 위해 Cloud Internet Services의 DNS 인터페이스에서 TXT 유형의 DNS 레코드를 구성해야 할 수 있습니다.
{:tip}

도메인에 대한 SSL 인증서 및 개인 키를 확보하고 나면 이를 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 형식으로 변환해야 합니다. 

1. 인증서를 PEM 형식으로 변환하려면 다음 명령을 실행하십시오. 
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
2. 개인 키를 PEM 형식으로 변환하려면 다음 명령을 실행하십시오. 
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### 인증서 가져오기 및 로드 밸런서 서비스에 권한 부여

IBM Certificate Manager를 통해 SSL 인증서를 관리할 수 있습니다. 

1. 지원되는 위치에서 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 인스턴스를 작성하십시오. 
2. 서비스 대시보드에서 **인증서 가져오기**를 사용하십시오. 
   * **이름**을 사용자 정의 하위 도메인 및 도메인(예: *lb.mydomain.com*)으로 설정하십시오. 
   * PEM 형식의 **인증서 파일**을 찾아보십시오. 
   * PEM 형식의 **개인 키 파일**을 찾아보십시오. 
   * **가져오기**를 수행하십시오. 
3. SSL 인증서가 포함된 인증서 관리자 인스턴스에 대한 액세스를 로드 밸런서 서비스에 제공하는 권한을 작성하십시오. [Identity and Access Authorizations](https://{DomainName}/iam#/authorizations)를 통해 이러한 권한을 관리할 수 있습니다. 
  - **작성**을 클릭한 후 **인프라 서비스**를 소스 서비스로 선택하십시오. 
  - **VPC용 로드 밸런서**를 리소스 유형으로 선택하십시오. 
  - **Certificate Manager**를 대상 서비스로 선택하십시오. 
  - **작성자** 서비스 액세스 역할을 지정하십시오. 
  - 로드 밸런서를 작성하려면 소스 리소스 인스턴스에 대한 모든 리소스 인스턴스 권한을 부여해야 합니다. 대상 서비스 인스턴스는 **모든 인스턴스**이거나 특정 인증서 관리자 리소스 인스턴스일 수 있습니다. 

### HTTPS 리스너 작성

이제 [로드 밸런서](https://{DomainName}/vpc/network/loadBalancers)로 이동하십시오. 

1. **vpc-lb-region1**을 선택하십시오. 
2. **프론트 엔드 리스너** 아래에서 **새 리스너**를 클릭하십시오. 

   -  **프로토콜**: HTTPS
   -  **포트**: 443
   -  **백엔드 풀**: 동일한 지역에 있는 풀
   -  **lb.YOUR-DOMAIN-NAME**에 대한 SSL 인증서를 선택하십시오. 

3. **작성**을 클릭하여 HTTPS 리스너를 구성하십시오. 

**지역 2**의 로드 밸런서에서 동일하게 **반복**하십시오. 

## 글로벌 로드 밸런서 구성
{: #global-load-balancer}

이 절에서는 서로 다른 {{site.data.keyword.Bluemix_notm}} 지역에서 구성된 로컬 로드 밸런서에 수신 트래픽을 분배하는 글로벌 로드 밸런서(GLB)를 구성합니다. 

### 글로벌 로드 밸런서를 사용하여 여러 지역에 트래픽 분배
서비스 아래의 [리소스 목록](https://{DomainName}/resources)으로 이동하여 작성된 CIS 서비스를 여십시오. 

1. **신뢰성** 아래의 **글로벌 로드 밸런서**로 이동하여 **로드 밸런서 작성**을 클릭하십시오. 
2. **lb.YOUR-DOMAIN-NAME**을 호스트 이름으로 입력하고 TTL을 60초로 입력하십시오. 
3. **풀 추가**를 클릭하여 기본 원본 풀을 정의하십시오. 
   - **이름**: lb-region1
   - **상태 검사**: 새 상태 검사 작성
     - **모니터 유형**: HTTP
     - **경로**: /
     - **포트**: 80
   - **상태 검사 지역**: 북미 동부지역
   - **원본**
     - **이름**: region1
     - **주소**: **REGION1** 로컬 로드 밸런서의 주소
     - **가중치**: 1
     - **추가**를 클릭하십시오. 

4. **서유럽** 지역에 있는 **region2** 로드 밸런서를 가리키는 하나 이상의 **원본 풀**을 **추가**한 후 **1개 리소스 프로비저닝**을 클릭하여 글로벌 로드 밸런서를 프로비저닝하십시오. 

**상태** 검사 상태가 **정상**으로 변경될 때까지 기다리십시오. 선택한 브라우저에서 **lb.YOUR-DOMAIN-NAME** 링크를 열어 작동 중인 글로벌 로드 밸런서를 확인하십시오. 

### 장애 복구 테스트
지금까지는 **지역 1**에 있는 서버가 **지역 2**에 있는 서버보다 가중치가 높았기 때문에 지역 1에 있는 서버에 연결되었습니다. 이제 **지역 1** 원본 풀에서의 상태 검사 장애를 소개합니다. 

1. [가상 서버 인스턴스](https://{DomainName}/vpc/compute/vs)로 이동하십시오. 
2. **지역 1**의 **구역 1**에서 실행 중인 서버 옆에 있는 **세 개의 점(...)**을 클릭한 후 **중지**를 클릭하십시오. 
3. **지역 1**의 **구역 2**에서 실행 중인 서버에 대해 동일하게 **반복**하십시오. 
4. CIS 서비스 아래의 GLB로 돌아가서 상태가 **위험**으로 변경될 때까지 기다리십시오. 
5. 이제 도메인 url을 새로 고치면 항상 **지역 2**에 있는 서버에 연결됩니다. 

지역 1의 구역 1 및 구역 2에 있는 서버를 **시작**하는 것을 잊지 마십시오.
{:tip}

## 리소스 제거
{: #removeresources}

- CIS 서비스 아래에서 글로벌 로드 밸런서, 원본 풀 및 상태 검사를 제거하십시오. 
- 인증서 관리자 서비스에서 인증서를 제거하십시오. 
- 로드 밸런서, VSI, 서브넷 및 VPC를 제거하십시오. 
- [리소스 목록](https://{DomainName}/resources) 아래에서 이 튜토리얼에 사용된 서비스를 삭제하십시오. 


## 관련 컨텐츠
{: #related}

* [IBM Cloud VPC에서 로드 밸런서 사용](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc#--beta-using-load-balancers-in-ibm-cloud-vpc)
