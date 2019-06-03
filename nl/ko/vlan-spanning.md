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

# IBM 네트워크를 통해 보안 사설 네트워크 연결
{: #vlan-spanning}

웹 애플리케이션에 대한 글로벌 접속 및 상시 작동 요구가 증가함에 따라 여러 클라우드 데이터 센터에서 서비스를 호스팅하라는 요구가 증가하고 있습니다. 데이터 센터가 여러 위치에 있으면 지리적 장애 발생 시 복원성을 제공하고 워크로드를 전 세계에 분산된 사용자에게 더 가까운 위치로 가져와서 대기 시간도 감소하고 인지되는 성능도 향상됩니다. [{{site.data.keyword.Bluemix_notm}} 네트워크](https://www.ibm.com/cloud/data-centers/)를 사용하면 사용자가 여러 데이터 센터 및 위치의 보안 사설 네트워크에서 호스팅되는 워크로드를 연결할 수 있습니다. 

이 튜토리얼에서는 서로 다른 데이터 센터에서 호스팅되는 두 개의 보안 사설 네트워크 사이에서 {{site.data.keyword.Bluemix_notm}} 사설 네트워크를 통해 비공개로 라우팅된 IP 연결의 설정을 제공합니다. 모든 리소스는 하나의 {{site.data.keyword.Bluemix_notm}} 계정이 소유합니다. 이는 [보안 사설 네트워크를 사용하여 워크로드 격리]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) 튜토리얼을 사용하여 [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning) 서비스를 사용하여 {{site.data.keyword.Bluemix_notm}} 사설 네트워크를 통해 안전하게 연결되는 두 개의 사설 네트워크를 배치합니다.
{:shortdesc}

## 목표
{: #objectives}

- {{site.data.keyword.Bluemix_notm}} IaaS 계정 내에서 보안 네트워크 연결
- 사이트 간 액세스를 위한 방화벽 규칙 설정 
- 사이트 간 라우팅 구성

## 사용되는 서비스
{: #products}

이 튜토리얼에서는 다음과 같은 {{site.data.keyword.Bluemix_notm}} 서비스를 사용합니다.  
* [가상 라우터 어플라이언스](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

이 튜토리얼에서는 비용이 발생할 수 있습니다. VRA는 월별 가격 플랜에서만 사용할 수 있습니다. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

  ![아키텍처](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. 보안 사설 네트워크 배치
2. VLAN Spanning 구성
3. 사설 네트워크 간 IP 라우팅 구성
4. 원격 사이트 액세스를 위한 방화벽 규칙 구성

## 시작하기 전에
{: #prereqs}

이 튜토리얼은 [보안 사설 네트워크를 사용하여 워크로드 격리]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 튜토리얼을 기반으로 합니다. 시작하기 전에 이 튜토리얼 및 해당 전제조건을 검토해야 합니다.  

## 보안 사설 네트워크 사이트 구성
{: #private_network}

[보안 사설 네트워크를 사용하여 워크로드 격리]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 튜토리얼은 두 개의 다른 데이터 센터에서 사설 네트워크를 구현하기 위해 두 번 활용됩니다. 사이트 사이에서 통신할 트래픽 또는 워크로드에 대한 대기 시간의 영향에 유의하는 것 외에 두 데이터 센터 중 활용할 수 있는 데이터 센터에 대한 제한은 없습니다.  

[보안 사설 네트워크를 사용하여 워크로드 격리]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 튜토리얼은 각각의 선택된 데이터 센터에 대해 변경 없이 수행할 수 있으며 이후 단계를 위해 다음과 같은 정보를 기록합니다.  

| 항목   | Datacenter1 | Datacenter2 |
|:------ |:--- | :--- |
| 데이터 센터 |  |  |
| VRA 공인 IP 주소 | <DC1 VRA 공인 IP 주소> | <DC2 VRA 공인 IP 주소> |
| VRA 사설 IP 주소 | <DC1 VRA 사설 IP 주소> | <DC2 VRA 사설 IP 주소> |
| VRA 사설 서브넷 및 CIDR |  |  |
| 사설 VLAN ID | &lt;DC1 사설 VLAN ID&gt;  | &lt;DC2 사설 VLAN ID&gt; |
| VSI 사설 IP 주소 | <DC1 VSI 사설 IP 주소> | <DC2 VSI 사설 IP 주소> |
| APP 구역 서브넷 및 CIDR | <DC1 APP 구역 서브넷/CIDR> | <DC2 APP 구역 서브넷/CIDR> |

1. [게이트웨이 어플라이언스](https://{DomainName}/classic/network/gatewayappliances) 페이지를 통해 각 VRA에 대한 게이트웨이 세부사항 페이지로 진행하십시오.   
2. 게이트웨이 VLAN 섹션을 찾은 후 **사설** 네트워크의 게이트웨이 [VLAN]( https://{DomainName}/classic/network/vlans)을 클릭하여 VLAN 세부사항을 보십시오. 이름은 ID('백엔드 고객 라우터'를 의미하는 `bcrxxx`)를 포함하고 있어야 하고 형식이 `nnnxx.bcrxxx.xxxx`여야 합니다. 
3. 프로비저닝된 VRA는 **디바이스* 섹션 아래에 표시됩니다. **서브넷** 섹션 아래에서 VRA 사설 서브넷 IP 주소 및 CIDR(/26)을 기록해 두십시오. 서브넷의 유형은 64개의 IP를 가진 기본 유형입니다. 이 세부사항은 나중에 라우팅 구성을 위해 필요합니다.  
4. 다시 게이트웨이 세부사항 페이지에서 **연관된 VLAN** 섹션을 찾은 후 보안 네트워크 및 APP 구역을 작성하기 위해 연관된 **사설** 네트워크의 [VLAN]( https://{DomainName}/classic/network/vlans)을 클릭하십시오.  
5. 프로비저닝된 VSI는 **디바이스* 섹션 아래에 표시됩니다. 라우팅 구성을 위해 필요하므로 **서브넷** 섹션 아래에서 VSI 서브넷 IP 주소 및 CIDR(/26)을 기록해 두십시오. 이 VLAN 및 서브넷은 두 VRA 방화벽 구성 모두에서 APP 구역으로 식별되고 &lt;APP 구역 서브넷/CIDR&gt;로 기록됩니다. 


## VLAN Spanning 구성 
{: #configure-vlan-spanning}

기본적으로 서로 다른 VLAN 및 데이터 센터에 있는 서버(및 VRA)는 사설 네트워크를 통해 서로 통신할 수 없습니다. 이 튜토리얼에서는 단일 데이터 센터 내에서 VRA를 사용하여 VLAN 및 서브넷을 클래식 IP 라우팅 및 방화벽과 연결하여 여러 VLAN에 걸친 서버 통신을 위한 사설 네트워크를 작성합니다. 이들은 동일한 데이터 센터 내에서 통신할 수 있지만 이 구성에서는 동일한 {{site.data.keyword.Bluemix_notm}} 계정에 속하는 서버가 여러 데이터 센터에 걸쳐 통신할 수 없습니다.  

[VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning) 서비스는 VRA와 연관되지 **않은** 서브넷과 VLAN 사이의 통신에 대한 이러한 제한을 해제합니다. VLAN Spanning이 사용으로 설정된 경우에도 VRA와 연관된 VLAN은 VRA 방화벽 및 라우팅 구성에 의해 결정된 대로 연관된 VRA를 통해서만 통신할 수 있습니다. VLAN Spanning이 사용으로 설정된 경우 {{site.data.keyword.Bluemix_notm}} 계정이 소유하는 VRA는 사설 네트워크를 통해 연결되어 통신할 수 있습니다.  

VRA에 의해 작성된 보안 사설 네트워크와 연관되지 않은 VLAN은 '확장'되어 여러 데이터 센터에 걸쳐 이 '연관되지 않은' VLAN 간 연결을 허용합니다. 여기에는 서로 다른 데이터 센터에 있는 동일한 IBM Cloud 계정에 속하는 VRA 게이트웨이(통과) VLAN이 포함되어 있습니다. 따라서 VLAN Spanning이 사용으로 설정된 경우 VRA가 여러 데이터 센터에 걸쳐 통신할 수 있게 합니다. VRA 간 연결이 설정되면 VRA 방화벽 및 라우팅 구성에서 보안 네트워크 내 서버가 연결될 수 있게 합니다.  

VLAN Spanning 사용: 

1. [VLAN]( https://{DomainName}/classic/network/vlans) 페이지로 진행하십시오. 
2. 페이지의 맨 위에서 **스팬** 탭을 선택하십시오. 
3. VLAN Spanning ‘켜짐’ 단일 선택 단추를 선택하십시오. 네트워크 변경이 완료되는 데 몇 분이 걸립니다. 
4. 이제 두 VRA가 통신할 수 있는지 확인하십시오. 

   데이터 센터 1 VRA에 로그인한 후 데이터 센터 2 VRA에 대해 ping을 실행하십시오. 

   ```
   SSH vyatta@<DC1 VRA Private IP Address>
   ping <DC2 VRA Private IP Address>
   ```
   {: codeblock}

   데이터 센터 2 VRA에 로그인한 후 데이터 센터 1 VRA에 대해 ping을 실행하십시오. 
   ```
   SSH vyatta@<DC2 VRA Private IP Address>
   ping <DC1 VRA Private IP Address>
   ```
   {: codeblock}

## VRA IP 라우팅 구성 
{: #vra_routing}

각각의 데이터 센터에서 VRA 라우팅을 작성하여 두 데이터 센터 모두의 APP 구역에 있는 VSI가 통신할 수 있게 하십시오.  

1. VRA 편집 모드에서 데이터 센터 2에 있는 APP 구역 사설 서브넷에 대한 정적 라우트를 데이터 센터 1에서 작성하십시오. 
   ```
   ssh vyatta@<DC1 VRA Private IP Address>
   conf
   set protocols static route <DC2 APP zone subnet/CIDR>  next-hop <DC2 VRA Private IP>
   commit
   ```
   {: codeblock}
2. VRA 편집 모드에서 데이터 센터 1에 있는 APP 구역 사설 서브넷에 대한 정적 라우트를 데이터 센터 2에서 작성하십시오. 
   ```
   ssh vyatta@<DC2 VRA Private IP Address>
   conf
   set protocols static route <DC1 APP zone subnet/CIDR>  next-hop <DC1 VRA Private IP>
   commit
   ```
   {: codeblock}
2. VRA 명령행에서 VRA 라우팅 테이블을 검토하십시오. 두 APP 구역 서브넷 간 트래픽을 허용하는 데 필요한 APP 구역 방화벽 규칙이 없기 때문에 지금은 VSI가 통신할 수 없습니다. 어느 쪽에서 시작되든 트래픽에는 방화벽 규칙이 필요합니다. 
   ```
   show ip route
   ```
   {: codeblock}

이제 IBM 사설 네트워크를 통해 APP 구역이 통신하도록 허용하는 새 라우트가 표시됩니다.  

## VRA 방화벽 구성
{: #vra_firewall}

기존 APP 구역 방화벽 규칙은 {{site.data.keyword.Bluemix_notm}} 사설 네트워크의 {{site.data.keyword.Bluemix_notm}} 서비스에 대한 이 서브넷의 트래픽을 허용하기 위해 NAT를 통한 공용 인터넷 액세스를 위해서만 구성되어 있습니다. 다른 데이터 센터에 있거나 이 VRA의 VSI와 연관된 다른 서브넷은 차단됩니다. 다음 단계는 다른 데이터 센터에 있는 서브넷에 대한 명시적 액세스를 허용하도록 APP-TO-INSIDE 방화벽 규칙과 연관된 `ibmprivate` 리소스 그룹을 업데이트하는 것입니다.  

1. 데이터 센터 1 VRA 편집 명령 모드에서 <DC2 APP 구역 서브넷>/CIDR을 `ibmprivate` 리소스 그룹에 추가하십시오. 
   ```
   set resources group address-group ibmprivate address <DC2 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
2. 데이터 센터 2 VRA 편집 명령 모드에서 <DC1 APP 구역 서브넷>/CIDR을 `ibmprivate` 리소스 그룹에 추가하십시오. 
   ```
   set resources group address-group ibmprivate address <DC1 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
3. 이제 두 데이터 센터에 있는 VSI가 통신할 수 있는지 확인하십시오. 
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}

   VSI가 통신할 수 없는 경우 인터페이스의 트래픽을 모니터링하고 방화벽 로그를 검토하기 위해 [보안 사설 네트워크를 사용하여 워크로드 격리]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) 튜토리얼의 지시사항을 따르십시오.  

## 리소스 제거
{: #removeresources}

이 튜토리얼에서 작성된 리소스를 제거하기 위한 단계입니다.  

VRA는 월별 유료 플랜을 기반으로 합니다. 취소해도 환불이 되지 않습니다. 다음 달에 이 VRA가 다시 필요하지 않은 경우에만 취소하는 것이 좋습니다. 이중 VRA 고가용성 클러스터가 필요한 경우 [게이트웨이 세부사항](https://{DomainName}/classic/network/gatewayappliances) 페이지에서 이 단일 VRA를 업그레이드할 수 있습니다.
{:tip}

1. 모든 Virtual Server 또는 Bare Metal Server를 취소하십시오. 
2. VRA를 취소하십시오. 
3. 지원 티켓을 사용하여 추가적인 VLAN을 취소하십시오.  


## 튜토리얼 확장

이 튜토리얼은
[VPN을 통해 보안 사설 네트워크에 연결](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network) 튜토리얼과 함께 사용하여 IPSec VPN을 통해 두 보안 네트워크를 사용자 원격 네트워크에 연결할 수 있습니다. {{site.data.keyword.Bluemix_notm}} IaaS 플랫폼에 대한 액세스의 복원성을 높이기 위해 두 보안 네트워크에 대한 VPN 링크를 설정할 수 있습니다. IBM은 IBM 사설 네트워크를 통한 클라이언트 데이터 센터 간 사용자 트래픽 라우팅을 허용하지 않습니다. 네트워크 루프를 방지하는 라우팅 구성은 이 튜토리얼의 범위를 벗어납니다.  


## 관련 자료
{:related}

1. VRF(Virtual Routing and Forwarding)는 여러 {{site.data.keyword.Bluemix_notm}} 계정에 걸쳐 네트워크를 연결하기 위해 VLAN Spanning 대신 사용합니다. [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)를 사용하는 모든 클라이언트의 경우 VRF가 필수입니다. [IBM Cloud의 VRF(Virtual Routing and Forwarding) 개요](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [IBM Cloud 네트워크](https://www.ibm.com/cloud/data-centers/)
