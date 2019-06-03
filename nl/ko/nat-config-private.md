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

# 사설 너트워크에서의 인터넷 액세스를 위한 NAT 구성
{: #nat-config-private}

웹 기반 IT 애플리케이션 및 서비스가 주류가 되면서, 이제 격리되어 존재하는 애플리케이션은 거의 없습니다. 개발자들은 인터넷 상의 서비스(이것이 오픈 소스 애플리케이션 코드 및 업데이트이든, 또는 REST API를 통해 애플리케이션 기능을 제공하는 서드파티 서비스든 관계없이)에 대한 액세스를 필수적인 것으로 생각하게 되었습니다. 네트워크 주소 변환(NAT) 위장은 사설 네트워크에서 인터넷 호스팅되는 서비스에 대한 액세스를 보안하는 데 일반적으로 사용되는 접근법입니다. NAT 위장에서 사설 IP 주소는 다대일 관계의 아웃바운드 공용 인터페이스의 IP 주소로 변환되어 사설 IP 주소를 공용 액세스로부터 보호합니다.   

이 튜토리얼에서는 {{site.data.keyword.Bluemix_notm}} 사설 네트워크의 보안된 서브넷에 연결하는 Virtual Router Appliance(VRA)에서의 네트워크 주소 변환(NAT) 위장 설정을 제공합니다. 이는 [안전한 사설 네트워크를 사용한 워크로드 격리](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) 튜토리얼을 기반으로 하며, 여기에 아웃바운드 트래픽을 보안하기 위해 소스 주소가 난독화되고 방화벽 규칙이 사용되는 소스 NAT(SNAT) 구성이 추가됩니다. 더 복잡한 NAT 구성은 [보충 VRA 문서]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)에 있습니다.
{:shortdesc}

## 목표
{: #objectives}

-	Virtual Router Appliance(VRA)에서 소스 네트워크 주소 변환(SNAT) 설정
-	인터넷 액세스를 위한 방화벽 규칙 설정

## 사용되는 서비스
{: #products}

이 튜토리얼에서는 다음과 같은 {{site.data.keyword.Bluemix_notm}} 서비스를 사용합니다.  

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

이 튜토리얼에서는 비용이 발생할 수 있습니다. VRA는 월별 가격 플랜에서만 사용할 수 있습니다. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

  ![아키텍처](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	필요한 인터넷 서비스를 기록합니다. 
2.	NAT을 설정합니다. 
3.	인터넷 방화벽 구역 및 규칙을 작성합니다. 

## 시작하기 전에
{: #prereqs}

이 튜토리얼은 [안전한 사설 네트워크를 사용한 워크로드 격리](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) 튜토리얼에서 작성된 안전한 사설 네트워크 격납장치의 호스트가 공용 인터넷 서비스에 액세스할 수 있도록 합니다. 이 튜토리얼을 먼저 완료해야 합니다.  

## 필요한 인터넷 서비스 기록
{: #Document_outbound}

첫 번째 단계는 공용 인터넷에서 액세스할 서비스를 식별하고 아웃바운드 트래픽과 이에 대응하는 인터넷에서의 인바운드 트래픽을 위해 사용으로 설정해야 하는 포트를 기록하는 것입니다. 이 포트 목록은 이후 단계의 방화벽 규칙에 필요합니다.  

http 및 https 포트가 대부분의 요구사항을 만족시키므로, 이 예에서는 이들 포트만 사용으로 설정됩니다. DNS 및 NTP 서비스는 {{site.data.keyword.Bluemix_notm}} 사설 네트워크로부터 제공됩니다. 이러한 서비스, 그리고 SMTP(포트 25) 또는 MySQL(포트 3306)과 같은 기타 서비스가 필요한 경우에는 추가 방화벽 규칙이 필요합니다. 두 기본 포트 규칙은 다음과 같습니다. 

-	포트 80(http)
-	포트 443(https)

서드파티 서비스가 소스 주소의 화이트리스트 지정을 지원하는지 확인하십시오. 지원하는 경우에는 서비스에 대한 액세스를 제한하도록 서드파티 서비스를 구성하는 데 VRA의 공인 IP 주소가 필요합니다.  


## 인터넷에 대한 NAT 위장 
{: #NAT_Masquerade}

NAT 위장을 사용하여 APP 구역에 있는 호스트의 외부 인터넷 액세스를 구성하려면 다음 지시사항을 따르십시오.  

1.	VRA에 SSH로 접속하여 \[edit\](구성)모드로 전환하십시오. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2.	이전 VRA 프로비저닝 튜토리얼에서 APP 구역 서브넷/VLAN에 대해 판별된 것과 동일한 `<Subnet Gateway IP>/<CIDR>`을 지정하여 VRA에 SNAT 규칙을 작성하십시오.  
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Subnet Gateway IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## 방화벽 작성
{: #Create_firewalls}

1.	방화벽 규칙을 작성하십시오.  
   ```
   set security firewall name APP-TO-OUTSIDE default-action drop
   set security firewall name APP-TO-OUTSIDE description 'APP traffic to the Internet'
   set security firewall name APP-TO-OUTSIDE default-log

   set security firewall name APP-TO-OUTSIDE rule 90 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 90 action accept
   set security firewall name APP-TO-OUTSIDE rule 90 destination port 80

   set security firewall name APP-TO-OUTSIDE rule 100 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 100 action accept
   set security firewall name APP-TO-OUTSIDE rule 100 destination port 443

   set security firewall name APP-TO-OUTSIDE rule 200 protocol icmp
   set security firewall name APP-TO-OUTSIDE rule 200 icmp type 8
   set security firewall name APP-TO-OUTSIDE rule 200 action accept
   commit
   ```
   {: codeblock}

## 구역 작성 및 규칙 적용
{: #Create_zone}

1.	외부 인터넷에 대한 액세스를 제어하는 데 필요한 구역인 OUTSIDE를 작성하십시오. 
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	방화벽을 지정하여 인터넷과의 트래픽을 제어하십시오. 
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE 
   commit
   save
   ```
   {: codeblock}
3.	APP 구역의 VSI가 이제 인터넷 상의 서비스에 액세스할 수 있는지 유효성 검증하십시오. SSH를 사용하여 로컬 VSI에 로그인하고, ping 및 curl을 사용하여 인터넷 사이트에 대한 ICMP 및 TCP 액세스를 유효성 검증하십시오.   
   ```bash
   ssh root@<VSI Private IP>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## 리소스 제거
{:removeresources}
이 튜토리얼에서 작성된 리소스를 제거하기 위한 단계입니다.  

VRA는 월별 유료 플랜을 기반으로 합니다. 취소해도 환불이 되지 않습니다. 다음 달에 이 VRA가 다시 필요하지 않은 경우에만 취소하는 것이 좋습니다. 듀얼 VRA 고가용성 클러스터가 필요한 경우에는 [게이트웨이 세부사항](https://{DomainName}/classic/network/gatewayappliances) 페이지에서 이 싱글 VRA를 업그레이드할 수 있습니다.
{:tip}  

1. 모든 Virtual Server 또는 Bare Metal Server를 취소하십시오. 
2. VRA를 취소하십시오. 
3. 지원 티켓을 통해 모든 추가 VLAN을 취소하십시오.  

## 관련 자료
{:related}

-	[VRA 네트워크 주소 변환]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[NAT 위장]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[보충 VRA 문서]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)

