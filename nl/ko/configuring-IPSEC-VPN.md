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

# 안전한 사설 네트워크로의 VPN
{: #configuring-IPSEC-VPN}

원격 네트워크 환경과 {{site.data.keyword.Bluemix_notm}}의 사설 네트워크에 있는 서버 간에 개인용 연결을 작성하는 것은 일반적인 요구사항입니다. 보통 이 연결은 {{site.data.keyword.Bluemix_notm}}의 하이브리드 워크로드, 데이터 전송, 개인용 워크로드 또는 시스템 관리를 지원합니다. 사이트 간 가상 사설 네트워크(VPN)는 네트워크 간 연결을 보안하는 일반적인 접근법입니다.  

{{site.data.keyword.Bluemix_notm}}에서는 사이트 간 데이터 센터 연결을 위한 몇 가지 옵션(공용 인터넷을 통한 VPN 또는 사설 전용 네트워크 연결)을 제공합니다.  

{{site.data.keyword.Bluemix_notm}}에 대한 안전한 전용 네트워크와 관련된 세부사항은 [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)를
참조하십시오. 공용 인터넷을 통한 VPN은 비용이 더 저렴하지만 대역폭을 보장하지 않습니다.  

{{site.data.keyword.Bluemix_notm}}에 프로비저닝된 서버에 대한, 공용 인터넷을 통한 연결에는 두 가지 적절한 VPN 옵션이 있습니다. 

-	[IPSec VPN]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

이 튜토리얼에서는 클라이언트 데이터 센터의 서브넷을 {{site.data.keyword.Bluemix_notm}} 사설 네트워크의 보안된 서브넷에 연결하는, VRA(Virtual Router Appliance)를 사용한 사이트 간 IPSec VPN의 설정을 제공합니다.  

이 예는 [안전한 사설 네트워크를 사용한 워크로드 격리](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) 튜토리얼을 기반으로 합니다. 이 튜토리얼은 사이트 간 IPSec
VPN, GRE 터널 및 정적 라우팅을 사용합니다. 동적 라우팅(BGP 등)을 사용하는 더 복잡한 VPN 구성은 [보충 VRA 문서](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)에 있습니다.
{:shortdesc}

## 목표
{: #objectives}

- IPSec VPN에 필요한 구성 매개변수 기록
- Virtual Router Appliance에 IPSec VPN 구성
- GRE 터널을 통해 트래픽 라우팅

## 사용되는 서비스
{: #products}

이 튜토리얼에서는 다음과 같은 {{site.data.keyword.Bluemix_notm}} 서비스를 사용합니다.  

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

이 튜토리얼에서는 비용이 발생할 수 있습니다. VRA는 월별 가격 플랜에서만 사용할 수 있습니다. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

  ![아키텍처](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	VPN 구성 기록
2.	VRA에 IPSec VPN 작성
3.	데이터 센터 VPN 및 터널 구성
4.	GRE 터널 작성
5.	정적 IP 라우트 작성
6.	방화벽 구성 

## 시작하기 전에
{: #prereqs}

이 튜토리얼에서는 [안전한 사설 네트워크를 사용한 워크로드 격리](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) 튜토리얼의 안전한 사설 격납장치를 사용자의 데이터 센터에 연결합니다. 이 튜토리얼을 먼저 완료해야 합니다. 

## VPN 구성 기록
{: #Document_VPN}

사용자의 데이터 센터와 {{site.data.keyword.Bluemix_notm}} 간에 IPSec VPN 사이트 간 링크를 구성하려면 현장의 네트워킹 팀과 협력하여 많은 구성 매개변수, 터널 유형 및 IP 라우팅 정보를 판별해야 합니다. 이러한 매개변수는 작동하는 VPN 연결과 정확히 일치해야 합니다. 일반적으로는 현장의 네트워킹 팀이 회사 표준을 만족시키는 구성을 지정하고, 필요한 데이터 센터 VPN 게이트웨이 IP 주소와 액세스할 수 있는 서브넷 주소 범위를 제공합니다. 

VPN 설정을 진행하려면 먼저 VPN 게이트웨이의 IP 주소와 IP 네트워크 서브넷 범위가 판별되어 데이터 센터 VPN 구성 및 {{site.data.keyword.Bluemix_notm}}의 안전한 사설 네트워크 격납장치에서 사용 가능해야 합니다. 이들은 다음 그림에 표시되어 있으며, 여기서는 안전한 격납장치의 APP 구역이 IPSec 터널을 통해 클라이언트 데이터 센터의 'DC IP Subnet'에 있는 시스템에 연결됩니다.    

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

VPN을 구성하는 {{site.data.keyword.Bluemix_notm}} 사용자와 클라이언트 데이터 센터의 네트워킹 팀은 다음 매개변수에 대해 동의하고 이를 기록해야 합니다. 이 예에서 원격 및 로컬 터널 IP 주소는 192.168.10.1 및 192.168.10.2로 설정됩니다. 현장의 네트워킹 팀이 동의하는 경우에는 임의의 서브넷을 사용할 수 있습니다.  

| 항목 |설명 |
|:------ |:--- | 
| &lt;ike group name&gt; | 연결의 IKE 그룹에 지정된 이름입니다. |
| &lt;ike encryption&gt; | {{site.data.keyword.Bluemix_notm}}와 클라이언트 데이터 센터 간에 사용하기로 동의한 IKE 암호화 표준입니다(일반적으로 *aes256*). |
| &lt;ike hash&gt; | {{site.data.keyword.Bluemix_notm}}와 클라이언트 데이터 센터 간에 사용하기로 동의한 IKE 해시입니다(일반적으로 *sha1*). |
| &lt;ike-lifetime&gt; | 클라이언트 데이터 센터의 IKE 수명입니다(일반적으로 *3600*). |
| &lt;esp group name&gt; | 연결의 ESP 그룹에 지정된 이름입니다. |
| &lt;esp encryption&gt; | {{site.data.keyword.Bluemix_notm}}와 클라이언트 데이터 센터 간에 사용하기로 동의한 ESP 암호화 표준입니다(일반적으로 *aes256*). |
| &lt;esp hash&gt; | {{site.data.keyword.Bluemix_notm}}와 클라이언트 데이터 센터 간에 사용하기로 동의한 ESP 해시입니다(일반적으로 *sha1*). |
| &lt;esp-lifetime&gt; | 클라이언트 데이터 센터의 ESP 수명입니다(일반적으로 *1800*). |
| &lt;DC VPN Public IP&gt;  | 클라이언트 데이터 센터에 있는 VPN 게이트웨이의 인터넷 연결 공인 IP 주소입니다. | 
| &lt;VRA Public IP&gt; | VRA의 공인 IP 주소입니다. |
| &lt;Remote tunnel IP\/24&gt; | IPSec 터널의 원격 단말에 지정된 IP 주소입니다. IBM Cloud 또는 클라이언트 데이터 센터와 충돌하지 않는 범위 내 IP 주소 쌍입니다. |
| &lt;Local tunnel IP\/24&gt; | IPSec 터널의 로컬 단말에 지정된 IP 주소입니다. |
| &lt;DC Subnet/CIDR&gt; | 클라이언트 데이터 센터 및 CIDR에서 액세스될 서브넷의 IP 주소입니다. |
| &lt;App Zone subnet/CIDR&gt; | VRA 작성 튜토리얼에 있는 APP 구역 서브넷의 네트워크 IP 주소 및 CIDR입니다. | 
| &lt;Shared-Secret&gt; | {{site.data.keyword.Bluemix_notm}}와 클라이언트 데이터 센터 간에 사용될 공유 암호화 키입니다. |

## VRA에 IPSec VPN 구성
{: #Configure_VRA_VPN}

{{site.data.keyword.Bluemix_notm}}에 VPN을 작성하기 위해 변경해야 하는 명령 및 모든 변수는 아래에 &lt; &gt;로 강조표시되어 있습니다. 각 행이 변경되어야 하므로 이러한 변경사항은 행별로 식별됩니다. 값은 테이블에서
제공됩니다.  

1. VRA에 SSH로 접속하여 `[edit]` 모드로 전환하십시오. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2. IKE(Internet Key Exchange) 그룹을 작성하십시오. 
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike encryption>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike hash>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lifetime>
   ```
   {: codeblock}
3. ESP(Encapsulating Security Payload) 그룹을 작성하십시오. 
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp encryption>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp hash>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lifetime>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. 사이트 간 연결을 정의하십시오. 
   ```
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication mode pre-shared-secret
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication pre-shared-secret <Shared-Secret>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  connection-type initiate
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  ike-group <ike group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  local-address <VRA Public IP>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  default-esp-group <esp group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1 protocol gre
   commit
   save
   ```
   {: codeblock}

## 데이터 센터 VPN 및 터널 구성
{: #Configure_DC_VPN}

1. 클라이언트 데이터 센터의 네트워크 팀은 &lt;DC VPN Public IP&gt;와 &lt;VRA Public IP&gt;을 서로 바꾸고, 로컬 터널 주소와 원격 터널 주소를 바꾸고, &lt;DC Subnet/CIDR&gt;과 &lt;App Zone subnet/CIDR&gt; 매개변수 또한 서로 바꾸어 동일한 IPSec VPN 연결을 작성합니다. 클라이언트 데이터 센터의 특성 구성 명령은 VPN의 벤더에 따라 달라집니다. 
1. 진행하기 전에 인터넷을 통해 DC VPN 게이트웨이의 공인 IP 주소에 액세스할 수 있는지 유효성 검증하십시오. 
   ```
   ping <DC VPN Public IP>
   ```
   {: codeblock}
2. 데이터 센터 VPN 구성이 완료되면 IPSec 링크가 자동으로 작동을 시작합니다. 링크가 설정되었으며 상태가 하나 이상의 활성 IPSec 터널이 있음을 나타내는지 확인하십시오. 데이터 센터에서 VPN의 양 단말이 모두 활성 IPSec 터널이 있음을 나타내고 있는지 확인하십시오. 
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. 링크가 작성되지 않은 경우에는 debug 명령을 사용하여 로컬 및 원격 주소가 올바르게 지정되었으며 기타 매개변수가 올바른 상태인지 유효성 검증하십시오. 
   ``` bash
   show vpn debug
   ```
   {: codeblock}

행 `peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......`이 출력에 있어야 합니다. 이 행이 없거나 'CONNECTING'으로 표시되는 경우에는 VPN 구성에 오류가 있는 것입니다.   

## GRE 터널 정의 
{: #Define_Tunnel}

1. VRA 편집 모드에서 GRE 터널을 작성하십시오. 
   ```
   set interfaces tunnel tun0 address <Local tunnel IP/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <VRA Public IP>
   set interfaces tunnel tun0 remote-ip <DC VPN Public IP>
   commit
   ```
   {: codeblock}
2. 터널의 양 단말이 구성되고 나면 터널이 자동으로 작동을 시작합니다. VRA 명령행에서 터널의 작동 상태를 확인하십시오. 
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   첫 번째 명령은 상태 및 링크가 `u/u`(작동 중/작동 중)인 터널을 표시해야 합니다. 두 번째 명령은 터널에 대한 추가 세부사항과, 트래픽이 전송 및 수신되고 있음을 표시합니다.
3. 트래픽이 터널을 통해 플로우되는지 유효성 검증하십시오. 
   ```bash
   ping <Remote tunnel IP>
   ```
   {: codeblock}
   `ping` 트래픽이 있는 동안에는 `show interfaces tunnel tun0`의 TX 및 RX 개수가 증가해야 합니다.
4. 트래픽이 플로우되지 않는 경우에는 `monitor interface` 명령을 사용하여 각 인터페이스에서 관찰되는 트래픽을 볼 수 있습니다. 인터페이스 `tun0`은 터널을 통한 내부 트래픽을 표시합니다. 인터페이스 `dp0bond1`은 원격 VPN 게이트웨이와의 사이에 플로우되는 캡슐화된 트래픽을 표시합니다. 
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

아무 트래픽도 관찰되지 않는 경우에는 데이터 센터 네트워킹 팀이 원격 사이트의 VPN 및 터널 인터페이스에서 플로우되는 트래픽을 모니터하여 문제를 로컬에서 찾아야 합니다.  

## 정적 IP 라우트 작성
{: #Define_Routing}

터널을 통해 원격 서브넷으로 트래픽을 경로 지정하도록 VRA 라우팅을 작성하십시오. 

1. VRA 편집 모드에서 정적 라우트를 작성하십시오. 
   ```
   set protocols static route <DC Subnet/CIDR>  next-hop <Remote tunnel IP>
   ```
   {: codeblock}   
2. VRA 명령행에서 VRA 라우팅 테이블을 검토하십시오. 이 시점에는 터널을 통한 트래픽을 허용하는 방화벽 규칙이 없으므로 라우트를 통해 전송되는 트래픽이 없습니다. 어느 쪽에서든 트래픽을 시작하려면 방화벽 규칙이 필요합니다. 
   ```bash
   show ip route
   ```
   {: codeblock}

## 방화벽 구성
{: #Configure_firewall}

1. 허용되는 ICMP 트래픽 및 TCP 포트에 대한 리소스 그룹을 작성하십시오.  
   ```
   set res group icmp-group icmpgrp type 8
   set res group icmp-group icmpgrp type 11
   set res group icmp-group icmpgrp type 3

   set res group port tcpports port 22
   set res group port tcpports port 80
   set res group port tcpports port 443
   commit
   ```
   {: codeblock}
2. VRA 편집 모드에서 원격 서브넷으로의 트래픽에 대한 방화벽 규칙을 작성하십시오. 
   ```
   set security firewall name APP-TO-TUNNEL default-action drop
   set security firewall name APP-TO-TUNNEL default-log

   set security firewall name APP-TO-TUNNEL rule 100 action accept
   set security firewall name APP-TO-TUNNEL rule 100 protocol tcp
   set security firewall name APP-TO-TUNNEL rule 100 destination port tcpports

   set security firewall name APP-TO-TUNNEL rule 200 protocol icmp
   set security firewall name APP-TO-TUNNEL rule 200 icmp group icmpgrp
   set security firewall name APP-TO-TUNNEL rule 200 action accept
   commit
   ```
   {: codeblock}

   ```
   set security firewall name TUNNEL-TO-APP default-action drop
   set security firewall name TUNNEL-TO-APP default-log

   set security firewall name TUNNEL-TO-APP rule 100 action accept
   set security firewall name TUNNEL-TO-APP rule 100 protocol tcp
   set security firewall name TUNNEL-TO-APP rule 100 destination port tcpports

   set security firewall name TUNNEL-TO-APP rule 200 protocol icmp
   set security firewall name TUNNEL-TO-APP rule 200 icmp group icmpgrp
   set security firewall name TUNNEL-TO-APP rule 200 action accept 
   commit
   ```
   {: codeblock}
2. 터널의 구역을 작성하고 양 구역에서 시작되는 트래픽에 대해 방화벽을 연관시키십시오. 
   ```
   set security zone-policy zone TUNNEL description "GRE Tunnel"
   set security zone-policy zone TUNNEL default-action drop
   set security zone-policy zone TUNNEL interface tun0

   set security zone-policy zone TUNNEL to APP firewall TUNNEL-TO-APP
   set security zone-policy zone APP to TUNNEL firewall APP-TO-TUNNEL
   commit
   save
   ```
   {: codeblock}
3. 양 단말의 방화벽 및 라우팅이 올바르게 구성되었으며 이제 ICMP 및 TCP 트래픽을 허용하는지 유효성 검증하는 방법은 먼저 VRA 명령행에서 원격 서브넷의 게이트웨이 주소에 대해 ping을 실행하고, 성공하는 경우에는 VSI에 로그인하는 것입니다. 
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}
4. VRA 명령행의 ping 실행이 실패하는 경우에는 터널 인터페이스에서 ping 요청에 대한 응답으로 ping 응답이 관찰되는지 유효성 검증하십시오. 
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
   응답이 없는 경우에는 데이터 센터의 방화벽 규칙 또는 라우팅에 문제가 있음을 나타냅니다. 모니터 출력에서 응답이 관찰되지만 ping 명령이 제한시간을 초과하는 경우에는 로컬 VRA 방화벽 규칙의 구성을 확인하십시오.
5. VSI에서의 ping 실행이 실패하는 경우에는 VRA 또는 VSI 구성의 VRA 방화벽 규칙 또는 라우팅에 문제가 있음을 나타냅니다. 이전 단계를 완료하여 요청이 전송되며 데이터 센터에서 응답이 관찰되는지 확인하십시오. 로컬 VLAN의 트래픽을 모니터링하고 방화벽 로그를 검사하면 문제를 라우팅 또는 방화벽으로 한정하는 데 도움이 됩니다. 
   ```
   monitor interfaces bonding dp0bond0.<VLAN ID>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

이렇게 하면 안전한 사설 네트워크 격납장치의 VPN 설정이 완료됩니다. 이 계열의 다른 튜토리얼은 격납장치가 공용 인터넷의 서비스에 액세스하는 방법을 보여줍니다. 

## 리소스 제거
{:removeresources}

이 튜토리얼에서 작성된 리소스를 제거하기 위한 단계입니다. 

VRA는 월별 유료 플랜을 기반으로 합니다. 취소해도 환불이 되지 않습니다. 다음 달에 이 VRA가 다시 필요하지 않은 경우에만 취소하는 것이 좋습니다. 듀얼 VRA 고가용성 클러스터가 필요한 경우에는 [게이트웨이 세부사항](https://{DomainName}/classic/network/gatewayappliances) 페이지에서 이 싱글 VRA를 업그레이드할 수 있습니다.
{:tip}  

1. 모든 Virtual Server 또는 Bare Metal Server를 취소하십시오. 
2. VRA를 취소하십시오. 
3. 지원 티켓을 통해 모든 추가 VLAN을 취소하십시오.  

## 관련 컨텐츠
{:related}
- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [정적 및 이동 가능 IP 서브넷](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Vyatta 문서](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS IPsec Site-to-Site VPN Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS Tunnels Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
