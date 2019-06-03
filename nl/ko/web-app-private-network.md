---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
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

# 안전한 사설 네트워크로부터 서비스되는 웹 애플리케이션
{: #web-app-private-network}

단기 및 장기 사용 요구사항에 맞춰 리소스를 스케일링할 수 있는 웹 애플리케이션 호스팅은 퍼블릭 클라우드의 일반적인 배치 패턴입니다. 퍼블릭 클라우드에서 제공하는 복원성 및 확장성을 보완하기 위해, 애플리케이션 워크로드에 대한 보안은 기본적인 전제조건입니다.  

이 튜토리얼에서는 Virtual Router Appliance(VRA), VLAN, NAT 및 방화벽을 사용하여 보안되는 사설 네트워크에서 호스팅되는, 스케일링 가능하며 안전한 인터넷 연결 웹 애플리케이션의 작성을 안내합니다. 이 애플리케이션은 하나의 Load Balancer, 두 개의 웹 애플리케이션 및 하나의 MySQL 데이터베이스 서버로 구성됩니다. 이는 기존 네트워킹을 사용하여 {{site.data.keyword.Bluemix_notm}} IaaS 플랫폼에 웹 애플리케이션을 안전하게 배치하는 방법을 보여주기 위해 세 개의 튜토리얼을 결합합니다.
{:shortdesc}

## 목표
{: #objectives}

- PHP 및 MySQL 설치를 위한 Virtual Server 작성
- 애플리케이션 서버에 요청을 분배하기 위해 Load Balancer 프로비저닝
- 안전한 네트워크를 작성하기 위해 Virtual Router Appliance(VRA) 배치
- VLAN 및 IP 서브넷 정의 
- 방화벽 규칙을 사용하여 네트워크 보안
- 애플리케이션 배치를 위한 SNAT(Source Network Address Translation)

## 사용되는 서비스
{: #products}

이 튜토리얼에서는 다음과 같은 {{site.data.keyword.Bluemix_notm}} 서비스를 사용합니다.  

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [Load Balancer]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

이 튜토리얼에서는 비용이 발생할 수 있습니다. VRA는 월별 가격 플랜에서만 사용할 수 있습니다. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

  ![아키텍처](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	안전한 사설 네트워크 구성
2.	애플리케이션 배치를 위한 NAT 액세스 구성
3.	스케일링 가능한 웹 앱 및 Load Balancer 배치

## 시작하기 전에
{: #prereqs}

이 튜토리얼에서는 순서대로 배치되는 세 개의 기존 튜토리얼을 이용합니다. 진행하기 전에 먼저 이들 세 튜토리얼을 검토해야 합니다. 

-	[안전한 사설 네트워크를 사용한 네트워크 격리]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[사설 네트워크에서의 인터넷 액세스를 위한 NAT 구성]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[Virtual Server를 사용하여 스케일링 가능한 고가용성 웹 앱 빌드]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## 안전한 사설 네트워크 구성
{: #private_network}

격리된 안전한 사설 네트워크 환경은 퍼블릭 클라우드의 IaaS 애플리케이션 보안 모델의 중심입니다. 방화벽, VLAN, 라우팅 및 VPN은 모두 격리된 사설 환경을 작성하는 데 있어서 필요한 컴포넌트입니다.
첫 번째 단계는 웹 앱이 배치될 안전한 사설 네트워크 격납장치를 작성하는 것입니다.   

- [안전한 사설 네트워크를 사용한 네트워크 격리](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

이 튜토리얼은 변경하지 않고 따를 수 있습니다. 나중 단계에서는 세 가상 머신이 APP 구역에 Nginx 웹 서버 및 MySQL 데이터베이스로 배치됩니다.  

## 안전한 애플리케이션 배치를 위한 NAT 구성
{: #nat_config}

소스 애플리케이션을 설치하려면 소스 저장소 액세스를 위한 안전한 인터넷 액세스가 필요합니다. 안전한 사설 네트워크의 서버가 공용 인터넷에 노출되지 않도록 보호하기 위해, 소스 주소를 난독화하고 방화벽 규칙을 사용하여 아웃바운드 애플리케이션 저장소 요청을 보호하는 Source NAT이 사용됩니다. 모든 인바운드 요청은 거부됩니다.  

- [사설 네트워크에서의 인터넷 액세스를 위한 NAT 구성]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

이 튜토리얼은 변경하지 않고 따를 수 있습니다. 다음 단계에서 NAT 구성은 필수 Nginx 및 MySQL 모듈에 액세스하는 데 사용됩니다.   


## 스케일링 가능한 웹 앱 및 Load Balancer 배치
{: #scalable_app}

안전한 사설 네트워크에 스케일링 가능하며 복원성 있는 웹 애플리케이션을 배치하는 방법을 보여주기 위한 Nginx 및 MySQL에서의 WordPress 설치(Load Balancer가 함께 사용됨)입니다.  

이 튜토리얼에서는 하나의 {{site.data.keyword.Bluemix_notm}} Load Balancer, 두 개의 웹 애플리케이션 서버, 하나의 MySQL 데이터베이스 서버를 작성하는 시나리오를 안내합니다. 서버는 방화벽을 통한 다른 워크로드 및 공용 인터넷으로부터의 분리를 제공하기 위해 안전한 사설 네트워크의 APP 구역에 배치됩니다.  

- [Virtual Server를 사용하여 스케일링 가능한 고가용성 웹 앱 빌드]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

이 튜토리얼에는 세 가지 변경사항이 있습니다. 

1.	이 튜토리얼에서 사용되는 Virtual Server는 VRA 뒤의 APP 방화벽 구역으로 보호되는 VLAN 및 서브넷에 배치됩니다. 
2. Virtual Server를 주문할 때 &lt;개인용 VLAN ID&gt;를 지정하십시오. Virtual Server를 주문할 때 &lt;개인용 VLAN ID&gt;를 지정하는 방법에 대한 세부사항은 [안전한 사설 네트워크를 사용한 워크로드 격리]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 튜토리얼의 [첫 번째 Virtual Server 주문](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver) 지시사항을 참조하십시오. 또한 Virtual Server에 액세스할 수 있도록 튜토리얼 앞부분에서 업로드된 SSH 키를 선택하는 것을 잊지 마십시오.  
3. WordPress 파일을 공유 스토리지로 복사할 때의 불량한 rsync 성능으로 인해, 이 튜토리얼에서는 File Storage 서비스를 사용하지 **않는** 것이 좋습니다. 이는 전체 튜토리얼에 영향을 주지 않습니다. 앱 서버 및 DB에 대한 File Storage 작성 및 마운트 설정과 관련된 단계는 무시할 수 있습니다. 또는 두 앱 서버 모두에서 모든 [애플리케이션 서버에서 PHP 애플리케이션 설치 및 구성](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application) 단계를 수행해야 합니다.
   [애플리케이션 서버에서 PHP 애플리케이션 설치 및 구성](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application)의 단계를 완료하기 전에 먼저 디렉토리 `/mnt/www/`를 두 앱 모두에서 작성하십시오. 이 디렉토리는 원래 현재 제거된 File Storage 절에서 작성되었었습니다.  

   ```sh
   mkdir /mnt/www
   ```

이 단계가 끝나면 Load Balancer는 정상 상태이며 인터넷에서 WordPress 사이트에 액세스할 수 있어야 합니다. 웹 애플리케이션을 구성하는 Virtual Server는 VRA 방화벽으로 인터넷을 통한 외부 액세스로부터 보호되며 Load Balancer를 통해서만 액세스할 수 있습니다. 프로덕션 환경의 경우에는 [{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services)에서 제공하는 DDoS 보호나 WAF(Web Application Firewall) 또한 고려해야 합니다. 


## 리소스 제거
{: #removeresources}

이 튜토리얼에서 작성된 리소스를 제거하기 위한 단계입니다.  

VRA는 월별 유료 플랜을 기반으로 합니다. 취소해도 환불이 되지 않습니다. 다음 달에 이 VRA가 다시 필요하지 않은 경우에만 취소하는 것이 좋습니다.
{:tip}  

1. 모든 Virtual Server 또는 Bare Metal Server를 취소하십시오. 
2. VRA를 취소하십시오. 
3. 지원 티켓을 통해 모든 추가 VLAN을 취소하십시오. 
4. Load Balancer를 삭제하십시오. 
5. File Storage 서비스를 삭제하십시오. 

## 튜토리얼 확장 

1. 이 튜토리얼에서는 처음에 두 개의 Virtual Server를 앱 계층으로 프로비저닝했으나, 추가 로드를 처리하기 위해 서버를 자동으로 추가할 수도 있습니다. [자동 스케일링]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group)은 비즈니스 애플리케이션을 지원하기 위한 Virtual Server 추가 또는 제거와 연관된 수동 스케일링 프로세스를 자동화하는 기능을 제공합니다. 

2. 두 번째 사설 VLAN 및 IP 서브넷을 VRA에 추가하여 MySQL 데이터베이스 서버를 호스팅하기 위한 DATA 구역을 작성함으로써 사용자 데이터를 추가로 보호하십시오. APP 구역에서 DATA 구역으로의, 포트 3306을 통한 인바운드 MySQL IP 트래픽만 허용하도록 방화벽 규칙을 구성하십시오.  

