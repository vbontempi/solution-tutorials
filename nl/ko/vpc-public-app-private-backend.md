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

# 가상 프라이빗 클라우드의 사설 및 공인 서브넷
{: #vpc-public-app-private-backend}

IBM은 2019년 4월 초부터(다음 달에 연장 사용이 개방됨) 제한된 수의 고객만 VPC에 대한 조기 액세스 프로그램에 참여하도록 승인합니다. 조직에서 IBM Virtual Private Cloud에 대한 액세스를 확보하길 원하는 경우에는 이 [추천 양식](https://{DomainName}/vpc){: new_window}에 기입을 완료하면 IBM 담당자가 다음 단계와 관련하여 연락을 줍니다.
{: important}

이 튜토리얼에서는 공인 및 사설 서브넷 및 각 서브넷의 가상 서버 인스턴스(VSI)를 사용하여 자체 {{site.data.keyword.vpc_full}}(VPC)를 작성하는 것에 대해 설명합니다. VPC는 다른 가상 네트워크와 논리적으로 격리된 공유 클라우드 인프라의 사용자 자체 프라이빗 클라우드입니다. 

[서브넷](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#subnet)은 IP 주소 범위입니다. 서브넷은 단일 구역에 바인드되며 여러 구역 또는 지역에 걸쳐 있을 수 없습니다. VPC의 용도로 서브넷의 중요한 특성은 서브넷을 서로 격리할 수 있고 일반적인 방법으로 서로 연결할 수도 있다는 사실입니다. 서브넷 격리는 서브넷 간 데이터 패킷 플로우를 제어하기 위해 방화벽 역할을 수행하는 네트워크 [액세스 제어 목록](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#access-control-list)(ACL)에 의해 달성될 수 있습니다. 마찬가지로 [보안 그룹](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#security-group)(SG)은 개별 VSI의 데이터 패킷 플로우를 제어하기 위해 가상 방화벽 역할을 수행합니다. 

공인 서브넷은 외부 세계에 노출되어야 하는 리소스에 사용됩니다. 외부 세계에서 직접 액세스되어서는 안 되는 제한된 액세스를 가진 리소스는 사설 서브넷에 배치됩니다. 이러한 서브넷에 있는 인스턴스는 백엔드 데이터베이스이거나 공개적인 액세스를 원하지 않는 일부 비밀 저장소일 수 있습니다. SG를 정의하여 VSI에 대한 트래픽을 허용하거나 거부합니다.
{:shortdesc}

간단히 말하면 VPC를 사용하여 다음과 같은 작업을 수행할 수 있습니다. 

- 소프트웨어 정의 네트워크(SDN)를 작성합니다. 
- 워크로드를 격리합니다. 
- 인바운드 및 아웃바운드 트래픽을 미세 제어합니다. 

## 목표

{: #objectives}

- 가상 프라이빗 클라우드에 사용 가능한 인프라 오브젝트에 대해 이해
- 가상 프라이빗 클라우드, 서브넷 및 서버 인스턴스를 작성하는 방법에 대해 학습
- 보안 그룹을 적용하여 서버에 대한 액세스에 보안을 설정하는 방법 알아보기

## 사용되는 서비스

{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

![아키텍처](images/solution40-vpc-public-app-private-backend/Architecture.png)


1. 관리자(DevOps)가 클라우드에서 필요한 인프라(VPC, 서브넷, 규칙이 있는 보안 그룹, VSI)를 설정합니다. 
2. 인터넷 사용자가 프론트 엔드의 웹 서버에 대한 HTTP/HTTPS 요청을 작성합니다. 
3. 프론트 엔드가 보안 백엔드에서 개인용 리소스를 요청하고 결과를 사용자에게 제공합니다. 

## 시작하기 전에

{: #prereqs}

- 사용자 권한을 확인하십시오. 사용자 계정에 VPC 리소스를 작성하고 관리할 수 있는 충분한 권한이 있는지 확인하십시오. 필요한 권한의 목록은 [VPC 사용자를 위해 필요한 권한 부여](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources)를 참조하십시오. 

- 가상 서버에 연결하기 위해 SSH 키가 필요합니다. SSH 키가 없으면 [키 작성을 위한 지시사항](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)을 참조하십시오. 

## 가상 프라이빗 클라우드 작성
{: #create-vpc}

자체 {{site.data.keyword.vpc_short}}를 작성하려면 다음을 수행하십시오. 

1. [VPC 개요](https://{DomainName}/vpc/overview) 페이지로 이동하여 **VPC 작성**을 클릭하십시오. 
2. **새 가상 프라이빗 클라우드** 섹션 아래에서 다음을 수행하십시오. 
   * **vpc-pubpriv**를 VPC의 이름으로 입력하십시오. 
   * **리소스 그룹**을 선택하십시오. 
   * 선택적으로 **태그**를 추가하여 리소스를 구성하십시오. 
3. **새 기본값 작성(모두 허용)**을 VPC 기본 액세스 제어 목록(ACL)으로 선택하십시오. 
1. SSH를 선택 취소한 후 **기본 보안 그룹**에서 ping을 실행하십시오. 
4. **VPC에 대한 새 서브넷** 아래에서 다음을 수행하십시오. 
   * 고유 이름으로 **vpc-pubpriv-backend-subnet**을 입력하십시오. 
   * 위치를 선택하십시오. 
   * CIDR 표기법으로 서브넷의 IP 범위를 입력하십시오(예: **10.xxx.0.0/24**). **주소 접두부**는 그대로 두고 **주소 수**를 256으로 선택하십시오. 
5. 서브넷 액세스 제어 목록(ACL)에 대해 **VPC 기본값 사용**을 선택하십시오. 나중에 인바운드 및 아웃바운드 규칙을 구성할 수 있습니다. 
6. **가상 프라이빗 클라우드 작성**을 클릭하여 인스턴스를 프로비저닝하십시오. 

사설 서브넷에 접속된 VSI가 소프트웨어를 로드하기 위해 인터넷에 액세스해야 하는 경우에는 퍼블릭 게이트웨이에 접속하면 접속된 모든 리소스가 공용 인터넷과 통신할 수 있으므로 퍼블릭 게이트웨이를 **접속됨**으로 전환하십시오. VSI가 필요한 모든 소프트웨어를 보유하게 되면 서브넷이 공용 인터넷에 도달할 수 없도록 퍼블릭 게이트웨이를 **분리됨**으로 되돌리십시오.
{: important}

서브넷 작성을 확인하려면 왼쪽 분할창에서 **서브넷**을 클릭한 후 상태가 **사용 가능**으로 변경될 때까지 기다리십시오. **서브넷** 아래에서 새 서브넷을 작성할 수 있습니다. 

## 백엔드 보안 그룹 및 VSI 작성
{: #backend-subnet-vsi}

이 절에서는 백엔드에 대한 보안 그룹 및 가상 서버 인스턴스를 작성합니다. 

### 백엔드 보안 그룹 작성

기본적으로 보안 그룹은 VPC와 함께 작성되어 접속된 모든 인스턴스에 대한 모든 SSH(TCP 포트 22) 및 Ping(ICMP 유형 8) 트래픽을 허용합니다. 

백엔드에 대한 새 보안 그룹을 작성하려면 다음을 수행하십시오.   
1. **네트워크** 아래에서 **보안 그룹**을 클릭한 후 **새 보안 그룹**을 클릭하십시오.   
2. **vpc-pubpriv-backend-sg**를 이름으로 입력한 후 이전에 작성한 VPC를 선택하십시오.   
3. **보안 그룹 작성**을 클릭하십시오. 

나중에 보안 그룹을 편집하여 인바운드 및 아웃바운드 규칙을 추가합니다. 

### 백엔드 가상 서버 인스턴스 작성

새로 작성되는 서브넷에서 가상 서버 인스턴스를 작성하려면 다음을 수행하십시오. 

1. **서브넷** 아래에서 백엔드 서브넷을 클릭하십시오. 
2. **접속된 인스턴스**를 클릭한 후 **새 인스턴스**를 클릭하십시오. 
3. 고유 이름을 입력한 후 **vpc-pubpriv-backend-vsi**를 선택하십시오. 그런 다음 이전에 작성한 VPC를 선택하고 이전처럼 **위치**를 선택하십시오. 
4. **Ubuntu Linux** 이미지를 선택하고 **모든 프로파일**을 클릭한 후 **컴퓨팅** 아래에서 **c-2x4**(2개의 vCPU 및 4GB RAM)를 선택하십시오. 
5. **SSH 키**에 대해 이전에 작성한 SSH 키를 선택하십시오. 
6. **네트워크 인터페이스** 아래에서 보안 그룹 옆의 **편집** 아이콘을 클릭하십시오. 
   * **vpc-pubpriv-backend-subnet**을 서브넷으로 선택하십시오. 
   * 기본 보안 그룹을 선택 취소한 후 **vpc-pubpriv-backend-sg**를 활성으로 선택하십시오. 
   * **저장**을 클릭하십시오. 
7. **가상 서버 인스턴스 작성**을 클릭하십시오. 

## 프론트 엔드 서브넷, 보안 그룹 및 VSI 작성
{: #frontend-subnet-vsi}

백엔드와 마찬가지로 보안 그룹 및 가상 서버 인스턴스를 가진 프론트 엔드 서브넷을 작성합니다. 

### 프론트 엔드에 대한 서브넷 작성

프론트 엔드에 대해 새 서브넷을 작성하려면 다음을 수행하십시오. 

1. 왼쪽 분할창의 **네트워크** 아래에 있는 **서브넷**을 클릭한 후 **새 서브넷**을 클릭하십시오. 
   * **vpc-pubpriv-frontend-subnet**을 이름으로 입력한 후 작성한 VPC를 선택하십시오. 
   * 위치를 선택하십시오. 
   * CIDR 표기법으로 서브넷의 IP 범위를 입력하십시오(예: **10.xxx.1.0/24**). **주소 접두부**는 그대로 두고 **주소 수**를 256으로 선택하십시오. 
1. 서브넷 액세스 제어 목록(ACL)에 대해 **VPC 기본값**을 선택하십시오. 나중에 인바운드 및 아웃바운드 규칙을 구성할 수 있습니다. 
1. 서브넷에 있는 모든 가상 서버 인스턴스에 유동 IP가 접속된다는 점을 고려하면 서브넷에 대해 퍼블릭 게이트웨이를 사용으로 설정하지 않아도 됩니다. 가상 서버 인스턴스는 유동 IP를 통해 인터넷에 연결됩니다. 
1. **서브넷 작성**을 클릭하여 이를 프로비저닝하십시오. 

### 프론트 엔드 보안 그룹 작성

프론트 엔드에 대한 새 보안 그룹을 작성하려면 다음을 수행하십시오. 
1. 네트워크 아래에서 **보안 그룹**을 클릭한 후 **새 보안 그룹**을 클릭하십시오. 
2. **vpc-pubpriv-frontend-sg**를 이름으로 입력한 후 이전에 작성한 VPC를 선택하십시오. 
3. **보안 그룹 작성**을 클릭하십시오. 

### 프론트 엔드 가상 서버 인스턴스 작성

새로 작성되는 서브넷에서 가상 서버 인스턴스를 작성하려면 다음을 수행하십시오. 

1. **서브넷** 아래에서 프론트 엔드 서브넷을 클릭하십시오. 
2. **접속된 인스턴스**를 클릭한 후 **새 인스턴스**를 클릭하십시오. 
3. 고유 이름(**vpc-pubpriv-frontend-vsi**)을 입력하고 이전에 작성한 VPC를 선택한 후 이전과 동일한 **위치**를 선택하십시오. 
4. **Ubuntu Linux** 이미지를 선택하고 **모든 프로파일**을 클릭한 후 **컴퓨팅** 아래에서 **c-2x4**(2개의 vCPU 및 4GB RAM)를 선택하십시오. 
5. **SSH 키**에 대해 이전에 작성한 SSH 키를 선택하십시오. 
6. **네트워크 인터페이스** 아래에서 보안 그룹 옆의 **편집** 아이콘을 클릭하십시오. 
   * **vpc-pubpriv-frontend-subnet**을 서브넷으로 선택하십시오. 
   * 기본 보안 그룹을 선택 취소한 후 **vpc-pubpriv-frontend-sg**를 활성화하십시오. 
   * **저장**을 클릭하십시오. 
   * **가상 서버 인스턴스 작성**을 클릭하십시오. 
7. VSI의 상태가 **전원 켜짐**으로 변경될 때까지 기다리십시오. 그런 다음 프론트 엔드 VSI **vpc-pubpriv-frontend-vsi**를 선택하고 **네트워크 인터페이스**까지 스크롤하여 **유동 IP** 아래에서 **예약**을 클릭하여 공인 IP 주소를 프론트 엔드 VSI에 연관시키십시오. 향후 참조를 위해 연관된 IP 주소를 클립보드에 저장하십시오. 

## 프론트 엔드와 백엔드 간 연결 설정
{: #setup-connectivity-frontend-backend}

모든 서버가 준비된 상태에서 이 절에서는 프론트 엔드 서버와 백엔드 서버 사이에서 일반 오퍼레이션을 허용하기 위해 연결을 설정합니다. 

### 프론트 엔드 보안 그룹 구성

1. **네트워크** 섹션의 **보안 그룹**으로 이동한 후 **vpc-pubpriv-frontend-sg**를 클릭하십시오. 
2. 먼저 **규칙 추가**를 사용하여 다음과 같은 **인바운드** 규칙을 추가하십시오. 이 규칙은 수신 HTTP 요청 및 Ping(ICMP)을 허용합니다. 

	<table>
   <thead>
      <tr>
         <td><strong>소스</strong></td>
         <td><strong>프로토콜</strong></td>
         <td><strong>값</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>임의 - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td><strong>80</strong>부터 <strong>80</strong>까지</td>
      </tr>
      <tr>
         <td>임의 - 0.0.0.0/0</td>
         <td>TCP</td>
         <td><strong>443</strong>부터 <strong>443</strong>까지</td>
      </tr>
      <tr>
         <td>임의 - 0.0.0.0/0</td>
	      <td>ICMP</td>
	      <td>유형: <strong>8</strong>, 코드: <strong>공백으로 둠</strong></td>
      </tr>
   </tbody>
   </table>

3. 다음으로 이러한 **아웃바운드** 규칙을 추가하십시오. 

   <table>
   <thead>
      <tr>
         <td><strong>대상</strong></td>
         <td><strong>프로토콜</strong></td>
         <td><strong>값</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>유형: <strong>보안 그룹</strong> - 이름: <strong>vpc-pubpriv-backend-sg</strong></td>
         <td>TCP</td>
         <td>백엔드 서버의 포트, 팁 확인</td>
      </tr>
   </tbody>
   </table>

일반적인 백엔드 서비스에 대한 포트는 다음과 같습니다. MySQL은 포트 3306, PostgreSQL 포트 5432를 사용합니다. Db2는 포트 50000 또는 50001에서 액세스됩니다. Microsoft SQL Server는 기본적으로 포트 1433을 사용합니다. [공통 포트를 가지고 있는 다수의 목록 중 하나를 Wikipedia에서 찾을 수 있습니다. ](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers)
{:tip }

### 백엔드 보안 그룹 구성
프론트 엔드와 마찬가지로 백엔드에 대해 보안 그룹을 구성하십시오. 

1. **네트워크** 섹션의 **보안 그룹**으로 이동한 후 **vpc-pubpriv-backend-sg**를 클릭하십시오. 
2. **규칙 추가**를 사용하여 다음과 같은 **인바운드** 규칙을 추가하십시오. 이 규칙은 백엔드 서비스에 대한 연결을 허용합니다. 

   <table>
   <thead>
      <tr>
         <td><strong>소스</strong></td>
         <td><strong>프로토콜</strong></td>
         <td><strong>값</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>유형: <strong>보안 그룹</strong> - 이름: <strong>vpc-pubpriv-frontend-sg</strong></td>
         <td>TCP</td>
         <td>백엔드 서버의 포트</td>
      </tr>
   </tbody>
   </table>


## 소프트웨어 설치 및 유지보수 태스크 수행
{: #install-software-maintenance-tasks}

유지보수 보안 그룹 및 `점프` 서버 역할을 수행하는 배스천 호스트를 사용하여 서버를 안전하게 유지보수하기 위해 [배스천 호스트를 사용하여 원격 인스턴스에 안전하게 액세스](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)에 언급된 단계를 수행하십시오.


## 리소스 제거
{: #remove-resources}

1. VPC 관리 콘솔에서 **유동 IP**를 클릭하고 VSI에 대한 IP 주소를 클릭한 후 조치 메뉴에서 **해제**를 선택하십시오. IP 주소를 해제할지 확인하십시오. 
2. 다음으로 **가상 서버 인스턴스**로 전환한 후 인스턴스를 **삭제**하십시오. 인스턴스가 삭제되고 인스턴스의 상태가 잠시 **삭제 중**으로 유지됩니다. 종종 브라우저를 새로 고쳐야 합니다. 
3. VSI가 삭제되면 **서브넷**으로 전환하십시오. 서브넷에 접속된 퍼블릭 게이트웨이가 있는 경우 서브넷 이름을 클릭하십시오. 서브넷 세부사항에서 퍼블릭 게이트웨이를 분리하십시오. 퍼블릭 게이트웨이가 없는 서브넷은 개요 페이지에서 삭제할 수 있습니다. 서브넷을 삭제하십시오. 
4. 서브넷이 삭제된 후 **VPC** 탭으로 전환한 후 VPC를 삭제하십시오. 

콘솔 사용 시 리소스 삭제 후 업데이트된 상태 정보를 보려면 브라우저를 새로 고쳐야 할 수 있습니다.
{:tip}

## 튜토리얼 확장
{: #expand-tutorial}

이 튜토리얼에 추가하거나 이 튜토리얼을 확장하시겠습니까? 몇 가지 방법은 다음과 같습니다. 

- [로드 밸런서](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-load-balancer)를 추가하여 여러 인스턴스에 인바운드 트래픽을 분배하십시오. 
- VPC가 또 다른 가상 사설망(VPN)(예: 온프레미스 네트워크 또는 또 다른 VPC)에 안전하게 연결할 수 있도록 [가상 사설망](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn)(VPN)을 작성하십시오. 


## 관련 컨텐츠
{: #related}

- [VPC 용어집](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)
- [IBM Cloud CLI를 사용하는 VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli#creating-a-vpc-using-the-ibm-cloud-cli)
- [REST API를 사용하는 VPC](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis#creating-a-vpc-using-the-rest-apis)
