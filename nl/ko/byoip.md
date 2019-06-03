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

# BYOIP(Bring Your Own IP Address)
{: #byoip}

BYOIP(Bring Your Own IP Address)는 기존 클라이언트 네트워크를 {{site.data.keyword.Bluemix_notm}}에서 프로비저닝되는 인프라에 연결하려는 경우 빈번히 요구됩니다. 목적은 일반적으로 클라이언트의 기존 IP 주소 지정 체계를 기반으로 하는 단일 IP 주소 공간으로의 전환을 통해 클라이언트 네트워크 라우팅 구성 및 작동의 변경을 최소화하는 것입니다. 

이 튜토리얼에서는 {{site.data.keyword.Bluemix_notm}}에 사용할 수 있는 BYOIP 구현 패턴에 대한 간략한 개요와 [안전한 사설 네트워크를 사용한 워크로드 격리](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 튜토리얼에 설명되어 있는 안전한 격납장치를 구현하는 경우 적절한 패턴을 식별하는 데 필요한 의사결정 트리를 제공합니다. 설정에 현장의 네트워크 팀, {{site.data.keyword.Bluemix_notm}} 기술 지원 또는 IBM Services의 의견이 필요할 수 있습니다. 

{:shortdesc}

## 목표
{: #objectives}

* BYOIP 구현 패턴을 이해합니다. 
* {{site.data.keyword.Bluemix_notm}}을 위한 구현 패턴을 선택합니다. 

## {{site.data.keyword.Bluemix_notm}} IP 주소 지정

{{site.data.keyword.Bluemix_notm}}에서는 여러 사설 주소 범위(구체적으로 예를 들면 10.0.0.0/8)를 이용하며, 일부 경우에는 이러한 범위가 기존 클라이언트 네트워크와 충돌할 수 있습니다. 주소 충돌이 있는 경우 IBM Cloud 네트워크와의 상호 운용을 가능하게 하기 위해 BYOIP를 지원하는 몇 가지 패턴이 있습니다. 

-	네트워크 주소 변환
-	GRE(Generic Routing Encapsulation) 터널링
-	IP 별명을 사용한 GRE 터널링
-	가상 오버레이 네트워크

패턴 선택은 {{site.data.keyword.Bluemix_notm}}에서 호스팅되는 애플리케이션에 의해 결정됩니다. 주된 요소로는 패턴 구현에 대한 애플리케이션 민감도, 그리고 클라이언트 네트워크와 {{site.data.keyword.Bluemix_notm}} 간 주소 범위 겹침 정도의 두 가지가 있습니다. {{site.data.keyword.Bluemix_notm}}에 대해 전용 사설 네트워크 연결을 사용하려는 경우에는 추가 고려사항 또한 적용됩니다. BYOIP를 고려하는 사용자는 [{{site.data.keyword.BluDirectLink}}
](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)에 대한 문서를 읽어보는 것이 좋습니다. {{site.data.keyword.BluDirectLink}} 사용자는 여기에 제공된 정보에 따라 연관된 안내를 따라야 합니다. 

## 구현 패턴 개요
{: #patterns_overview}

1. **NAT**: 온프레미스 클라이언트 라우터에서 NAT을 수행합니다. 온프레미스 NAT을 수행하여 클라이언트 주소 지정 체계를 {{site.data.keyword.Bluemix_notm}}가 프로비저닝된 IaaS 서비스에 지정한 IP 주소로 변환합니다.   
2. **GRE 터널링**: {{site.data.keyword.Bluemix_notm}}와 온프레미스 네트워크 간의 GRE 터널을 통해 IP 트래픽을 라우팅(일반적으로 VPN 사용)하여 주소 지정 체계를 통합합니다. 이는 [이 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)에 설명되어 있는 시나리오입니다.  

   주소 공간 겹침 가능성에 따라 두 가지 하위 패턴이 있습니다. 
     * 주소 겹침 없음: 주소 범위의 겹침 및 네트워크 간 충돌 위험이 없습니다. 
     * 부분적 주소 겹침: 클라이언트와 {{site.data.keyword.Bluemix_notm}} IP 주소 공간이 동일한 주소 범위를 사용하며 겹침 및 충돌 가능성이 있습니다. 이 경우에는 {{site.data.keyword.Bluemix_notm}} 사설 네트워크와 겹치지 않는 클라이언트 서브넷 주소가 선택됩니다. 

3. GRE 터널링 + IP 별명
온프레미스 네트워크와 {{site.data.keyword.Bluemix_notm}}에 있는 서버에 지정된 별명 IP 주소 간의 GRE 터널을 통해 IP 트래픽을 라우팅하여 주소 지정 체계를 통합합니다. 이는 [이 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)에 설명되어 있는 시나리오의 특수한 경우입니다. VRA의 적절한 라우팅 구성으로 지원되는, {{site.data.keyword.Bluemix_notm}}에 프로비저닝된 Virtual Server 및 Bare Metal Server에 호환 가능 IP 서브넷에 대한 추가 인터페이스 및 IP 별명이 작성됩니다. 

4. 가상 오버레이 네트워크
[{{site.data.keyword.Bluemix_notm}} Virtual Private Cloud(VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure)는 {{site.data.keyword.Bluemix_notm}}의 완전 가상 환경에 대해 BYOIP를 지원합니다. 이는 [이 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure)에 설명되어 있는 안전한 사설 네트워크 격납장치에 대한 대안으로 고려될 수 있습니다. 

또는 {{site.data.keyword.Bluemix_notm}} 네트워크의 계층에 가상 오버레이 네트워크를 구현하는 VMware NSX와 같은 솔루션을 고려하십시오. 가상 오버레이의 모든 BYOIP 주소는 {{site.data.keyword.Bluemix_notm}} 네트워크 주소 범위로부터 독립적입니다. VMware 및 [{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud) 시작하기를 참조하십시오. 

## 패턴 의사결정 트리
{: #decision_tree}

여기 제공된 의사결정 트리는 적절한 구현 패턴을 결정하는 데 사용할 수 있습니다.  

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

다음 참고는 추가적인 안내를 제공합니다. 

### NAT이 애플리케이션에 문제를 일으킬 수 있습니까?
{: #nat_consideration}

NAT이 문제가 될 수 있는 두 가지 특별한 경우는 다음과 같습니다. 이러한 경우에는 NAT을 사용하지 않아야 합니다.  

- Microsoft AD 도메인 통신과 같은 일부 애플리케이션, 그리고 P2P 애플리케이션에는 NAT과 관련된 기술적 문제가 있습니다. 
- 알 수 없는 서버가 {{site.data.keyword.Bluemix_notm}}와 통신해야 하는 경우 또는 {{site.data.keyword.Bluemix_notm}}와 온프레미스 서버 간에 수백 개의 양방향 연결이 필요한 경우. 이 경우에는 맵핑을 사전에 식별할 수 없으므로 클라이언트 라우터/NAT 테이블에서 모든 맵핑을 구성할 수 없습니다. 


### 주소 겹침 없음
{: #no-overlap}

온프레미스 네트워크에서 10.0.0.0/8이 사용되고 있습니까? 온프레미스 네트워크와 {{site.data.keyword.Bluemix_notm}} 사설 네트워크 간에 주소 겹침이 없는 경우에는 NAT이 필요하지 않도록 온프레미스 네트워크와 IBM Cloud 간에 [이 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN)에 설명되어 있는 바와 같은 GRE 터널링을 사용할 수 있습니다. 이를 위해서는 현장의 네트워크 팀과 함께 네트워크 주소 사용을 검토해야 합니다.  

### 부분적인 주소 겹침
{: #partial_overlap}

10.0.0.0/8 범위 중 무엇이든 온프레미스 네트워크에서 사용 중이라면, {{site.data.keyword.Bluemix_notm}} 네트워크에 사용 가능한 겹치지 않는 서브넷이 있습니까? 현장의 네트워크 팀과 함께 네트워크 주소 사용을 검토하고 {{site.data.keyword.Bluemix_notm}} 기술 영업 부서에 문의하여 사용 가능한 겹치지 않는 네트워크를 식별하십시오.  

### IP 별명 지정이 문제를 일으킬 수 있습니까?
{: #ip_alias}

겹치지 않는 주소가 존재하지 않는 경우에는 안전한 사설 네트워크 격납장치에 배치된 Virtual Server 및 Bare Metal Server에 IP 별명 지정을 구현할 수 있습니다. IP 별명 지정은 각 서버에 있는 하나 이상의 네트워크 인터페이스에 여러 서브넷 주소를 지정합니다.  

IP 별명 지정은 흔히 사용되는 기법이지만, 서버 및 애플리케이션 구성을 검토하여 이것이 멀티홈 및 IP 별명 구성에서 잘 작동하는지 판별하는 것이 좋습니다.   

BYOIP 서브넷에 대해 동적 라우트(예: BGP) 또는 정적 라우트를 작성하려면 VRA에서의 추가 라우팅 구성이 필요합니다.  

## 관련 컨텐츠
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [Virtual Private Cloud(VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
