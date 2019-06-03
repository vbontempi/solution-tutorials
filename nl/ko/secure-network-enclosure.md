---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-08"
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

# 보안 사설 네트워크를 사용하여 워크로드 격리
{: #secure-network-enclosure}

격리된 보안 사설 네트워크 환경에 대한 필요성이 퍼블릭 클라우드에 IaaS 애플리케이션 배치의 중심입니다. 방화벽, VLAN, 라우팅 및 VPN은 모두 격리된 사설 환경 작성 시 필요한 컴포넌트입니다. 이러한 격리를 통해 가상 머신 및 베어메탈 서버가 복잡한 다중 계층 애플리케이션 토폴로지에 안전하게 배치되어 공용 인터넷의 위험으로부터 보호할 수 있습니다.   

이 튜토리얼에서는 보안 사설 네트워크(격납장치)를 작성하기 위해 {{site.data.keyword.Bluemix_notm}}에서 [VRA(Virtual Router Appliance)](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-)를 구성하는 방법을 강조합니다. VRA 게이트웨이 어플라이언스는 단일 자체 관리 패키지, 방화벽, VPN 게이트웨이, NAT(Network Address Translation) 및 엔터프라이즈급 라우팅에 제공됩니다. 이 튜토리얼에서는 VRA를 사용하여 {{site.data.keyword.Bluemix_notm}}에서 엔클로징된 격리된 네트워크 환경을 작성하는 방법을 보여줍니다. 이 격납장치 내에서 익숙하고 잘 알려진 기술인 IP 라우팅, VLAN, IP 서브넷, 방화벽 규칙, 가상 및 베어메탈 서버를 사용하여 애플리케이션 토폴로지를 작성할 수 있습니다.   

{:shortdesc}

이 튜토리얼은 {{site.data.keyword.Bluemix_notm}}에서 클래식 네트워킹의 시작점이므로 그대로 프로덕션 기능으로 간주해서는 안 됩니다. 고려할 수 있는 추가 기능은 다음과 같습니다. 
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [하드웨어 방화벽 어플라이언스](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* 데이터 센터에 대한 보안 연결을 위한 [IPSec VPN](https://{DomainName}/catalog/infrastructure/ipsec-vpn)
* 클러스터된 VRA 및 이중 업링크를 사용한 고가용성
* 보안 이벤트의 로깅 및 감사

## 목표 
{: #objectives}

* VRA(Virtual Router Appliance) 배치
* 가상 머신 및 베어메탈 서버를 배치할 VLAN 및 IP 서브넷 정의
* 방화벽 규칙을 사용하여 VRA 및 격납장치 보안

## 사용되는 서비스
{: #products}

이 튜토리얼에서는 다음과 같은 {{site.data.keyword.Bluemix_notm}} 서비스를 사용합니다. 
* [가상 라우터 어플라이언스](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)

이 튜토리얼에서는 비용이 발생할 수 있습니다. VRA는 월별 가격 플랜에서만 사용할 수 있습니다. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

  ![아키텍처](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. VPN 구성
2. VRA 배치 
3. Virtual Server 작성
4. VRA를 통한 라우트 액세스
5. 격납장치 방화벽 구성
6. APP 구역 정의
7. INSIDE 구역 정의

## 시작하기 전에
{: #prereqs}

### VPN 액세스 구성

이 튜토리얼에서는 작성된 네트워크 격납장치가 공용 인터넷에 표시되지 않습니다. VRA 및 서버는 사설 네트워크를 통해서만 액세스할 수 있으므로 연결을 위해 VPN을 사용합니다.  

1. [VPN 액세스가 사용으로 설정되어 있는지 확인하십시오. ](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking)

     VPN 액세스를 사용으로 설정하기 위해 **마스터 사용자**여야 합니다. 그렇지 않으면 액세스를 위해 마스터 사용자에게 문의하십시오.
     {:tip}
2. [사용자 목록](https://{DomainName}/iam#/users)에서 사용자를 선택하여 VPN 액세스 인증 정보를 얻으십시오. 
3. [웹 인터페이스](https://www.softlayer.com/VPN-Access)를 통해 VPN에 로그인하거나 [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) 또는 [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7)용 VPN 클라이언트를 사용하십시오. 

   VPN 클라이언트의 경우 [VPN 웹 액세스 페이지](https://www.softlayer.com/VPN-Access)에서 *vpn.xxxnn.softlayer.com* 양식의 단일 데이터 센터 VPN 액세스 지점의 FQDN을 게이트웨이 주소로 사용하십시오.
   {:tip}

### 계정 권한 확인

인프라 마스터 사용자에게 문의하여 다음 권한을 얻으십시오. 
- **빠른 권한** - 기본 사용자
- **네트워크** - 격납장치를 작성하고 구성할 수 있도록 모든 네트워크 권한이 필요합니다.  
- **서비스** - SSH 키를 관리합니다. 

### SSH 키 업로드

포털을 통해 VRA 및 사설 네트워크에 액세스하고 이를 관리하는 데 사용될 포털 [SSH 공개 키를 업로드](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial)하십시오.   

### 대상 데이터 센터

보안 사설 네트워크를 배치할 {{site.data.keyword.Bluemix_notm}} 데이터 센터를 선택하십시오.  

### VLAN 주문

대상 데이터 센터에서 사설 격납장치를 작성하려면 먼저 서버에 필요한 사설 VLAN을 지정해야 합니다. 첫 번째 사설 및 첫 번째 공용 VLAN에는 요금이 부과되지 않습니다. 다중 계층 애플리케이션 토폴로지를 지원하기 위한 추가 VLAN에는 요금이 부과됩니다.  

동일한 데이터 센터 라우터에서 충분한 VLAN을 사용하고 VRA와 연관시키려면 VLAN을 주문하는 것이 좋습니다. [VLAN 주문](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans)을 참조하십시오. 

## 가상 라우터 어플라이언스 프로비저닝
{: #VRA}

첫 번째 단계는 사설 네트워크 격납장치에 대한 방화벽 및 IP 라우팅을 제공할 VRA를 배치하는 것입니다. {{site.data.keyword.Bluemix_notm}} 제공 공용 페이싱 통과 VLAN을 사용하여 격납장치에서 인터넷에 액세스합니다. 게이트웨이 및 선택적으로 하드웨어 방화벽은 공용 VLAN으로부터 보안 사설 격납장치 VLAN으로의 연결을 작성합니다. 이 솔루션 튜토리얼에서는 VRA(Virtual Router Appliance)가 이 게이트웨이 및 방화벽 경계를 제공합니다.  

1. 카탈로그에서 [게이트웨이 어플라이언스](https://{DomainName}/gen1/infrastructure/provision/gateway)를 선택하십시오. 
3. **게이트웨이 공급업체** 섹션에서 AT&T를 선택하십시오. 업링크 속도 "최대 20Gbps"와 "최대 2Gbps" 사이에서 선택할 수 있습니다. 
4. **호스트 이름** 섹션에서 새 VRA의 호스트 이름 및 도메인을 입력하십시오. 
5. **고가용성** 선택란을 선택하는 경우 VRRP를 사용하여 활성/백업 설정에서 작동 중인 두 개의 VRA 디바이스를 얻습니다. 
6. **위치** 섹션에서 VRA가 필요한 **팟(Pod)** 및 위치를 선택하십시오. 
7. 단일 프로세서 또는 듀얼 프로세서를 선택하십시오. 서버 목록을 얻습니다. 단일 선택 단추를 클릭하여 서버를 선택하십시오.  
8. **RAM** 크기를 선택하십시오. 프로덕션 환경의 경우 최소 64GB의 RAM을 사용하는 것이 좋습니다. 테스트 환경의 경우 최소 8GB입니다. 
9. **SSH 키**를 선택하십시오(선택사항). 이 SSH 키는 VRA에 설치되므로 사용자 vyatta를 사용하여 이 키로 VRA에 액세스할 수 있습니다. 
10. 하드 드라이브. 기본값을 유지하십시오. 
11. **업링크 포트 속도** 섹션에서 사용자의 요구를 충족하는 속도, 중복성 및 사설 및/또는 공용 인터페이스의 조합을 선택하십시오. 
12. **추가 기능** 섹션에서 기본값을 유지하십시오. 공용 인터페이스에서 IPv6를 사용하려면 IPv6 주소를 선택하십시오. 

오른쪽에서 **주문 요약**을 볼 수 있습니다. _아래에 나열된 서드파티 서비스 계약을 읽고 이에 동의합니다._ 선택란을 선택한 후 **작성** 단추를 클릭하십시오. 게이트웨이가 배치됩니다. 

이 디바이스에서 트랜잭션이 진행 중임을 나타내는 **시계** 기호와 함께 VRA가 거의 즉시 [디바이스 목록](https://{DomainName}/classic/devices)에 표시됩니다. VRA 작성이 완료될 때까지 **시계** 기호가 남아 있으며 세부사항을 보는 것 외에는 디바이스에 대한 어떤 구성 조치도 수행할 수 없습니다.
{:tip}

### 배치된 VRA 검토

1. 새 VRA를 검사하십시오. [인프라 대시보드](https://{DomainName}/classic)의 왼쪽 분할창에서 **네트워크**를 선택한 후 **게이트웨이 어플라이언스**를 선택하여 [게이트웨이 어플라이언스](https://{DomainName}/classic/network/gatewayappliances) 페이지로 이동하십시오. **게이트웨이** 열에서 새로 작성된 VRA의 이름을 선택하여 게이트웨이 세부사항 페이지로 진행하십시오. ![](images/solution33-secure-network-enclosure/Gateway-detail.png)

2. 향후 사용을 위해 VRA의 `사설` 및 `공인` IP 주소를 기록해 두십시오. 

## 초기 VRA 설정
{: #initial_VRA_setup}

1. 워크스테이션에서 SSL VPN을 통해 기본 **vyatta** 계정을 사용하여 VRA에 로그인하여 SSH 보안 프롬프트를 승인하십시오.  
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   SSH에서 비밀번호 입력 프롬프트를 표시하는 경우에는 SSH 키가 빌드에 포함되지 않은 것입니다. `VRA 사설 IP 주소`를 사용하여 [웹 브라우저](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui)를 통해 VRA에 액세스하십시오. 비밀번호는 [소프트웨어 비밀번호](https://{DomainName}/classic/devices/passwords) 페이지에서 제공됩니다. **구성** 탭에서 시스템/로그인/vyatta 분기를 선택한 후 원하는 SSH 키를 추가하십시오.
   {:tip}

   VRA를 설정하려면 `configure` 명령을 사용하여 VRA를 \[edit\] 모드에 배치해야 합니다. `edit` 모드에 있으면 프롬프트가 `$`에서 `#`로 변경됩니다. VRA 구성 변경에 성공하면 `compare` 명령을 사용하여 변경사항을 보고 `validate` 명령을 사용하여 변경사항을 확인할 수 있습니다. `commit` 명령을 사용하여 변경사항을 커미트하면 실행 중인 구성에 적용되고 시작 구성에 자동응로 저장됩니다. 


   {:tip}
2. SSH 로그인만 허용하여 보안을 강화하십시오. 사설 네트워크를 통해 SSH 로그인에 성공했으므로 사용자 ID/비밀번호 인증을 통해 액세스를 사용 안함으로 설정하십시오.  
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
   `configure` 입력 후 이 튜토리얼의 이 지점에서는 모든 VRA 명령이 `edit` 프롬프트에서 입력된다고 가정합니다.
3. 초기 구성을 검토하십시오. 
   ```
   show
   ```
   {: codeblock}

   {{site.data.keyword.Bluemix_notm}} IaaS 환경의 경우 VRA가 사전 구성되어 있습니다. 여기에는 다음 사항이 포함되어 있습니다. 
   - NTP 서버
   - 이름 서버
   - SSH
   - HTTPS 웹 서버 
   - 기본 시간대 미국/시카고
4. 필요에 따라 로컬 시간대를 설정하십시오. 탭 키를 사용하여 자동 완성하면 잠재적 시간대 값이 나열됩니다. 
   ```
   set system time-zone <timezone>
   ```
   {: codeblock}
5. Ping 동작을 설정하십시오. 라우팅 및 방화벽 문제점 해결을 지원하기 위해 Ping은 사용 안함으로 설정되지 않습니다.  
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. Stateful 방화벽 오퍼레이션을 사용으로 설정하십시오. 기본적으로 VRA 방화벽은 Stateless입니다.  
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. 변경사항을 커미트하고 시작 구성에 자동으로 저장하십시오.  
   ```
   commit
   ```
   {: codeblock}

## 첫 번째 가상 서버 주문
{: #order_virtualserver}

VRA 구성 오류의 진단을 지원하기 위해 이 지점에서 가상 서버가 작성됩니다. VSI에 대한 액세스는 이후 단계에서 VRA를 통해 VSI에 대한 액세스가 라우팅되기 전에 {{site.data.keyword.Bluemix_notm}} 사설 네트워크를 통해 유효성 검증됩니다.  

1. [가상 서버](https://{DomainName}/catalog/infrastructure/virtual-server-group)를 주문하십시오.   
2. **공용 가상 서버**를 선택하고 계속하십시오. 
3. 주문 페이지에서 다음을 수행하십시오. 
   - **청구**를 **시간별**로 설정하십시오. 
   - *VSI 호스트 이름* 및 *도메인 이름*을 설정하십시오. 이 도메인 이름은 라우팅 및 DNS에 사용되지 않지만 네트워크 이름 지정 표준과 맞아야 합니다.  
   - **위치**를 VRA와 동일하게 설정하십시오. 
   - **프로파일**을 **C1.1x1**로 설정하십시오. 
   - 이전에 지정한 **SSH 키**를 추가하십시오. 
   - **운영 체제**를 **CentOS 7.x - 최소**로 설정하십시오. 
   - **업링크 포트 속도**에서 **사설 네트워크 업링크**만 지정하도록 네트워크 인터페이스를 기본값인 *공용 및 사설*에서 변경해야 합니다. 그러면 새 서버가 인터넷에 직접 액세스되지 않으며 VRA에 대한 라우팅 및 방화벽 규칙에 의해 액세스가 제어됩니다. 
   - **사설 VLAN**을 이전에 주문한 사설 VLAN의 VLAN ID로 설정하십시오. 
4. 선택 상자를 클릭하여 '서드파티' 서비스 계약에 동의한 후 **작성**을 클릭하십시오. 
5. 이메일을 통해 또는 [디바이스](https://{DomainName}/classic/devices) 페이지에서 완료를 모니터하십시오.  
6. 이후 단계를 위해 VSI의 *사설 IP 주소*와 올바른 VLAN에 VSI가 지정되는 **디바이스 세부사항** 페이지의 **네트워크** 섹션 아래에 있는 항목을 기록해 두십시오. 그렇지 않으면 이 VSI를 삭제하고 올바른 VLAN에서 새 VSI를 작성하십시오.  
7. VPN을 통해 로컬 워크스테이션에서 Ping 및 SSH를 사용하여 {{site.data.keyword.Bluemix_notm}} 사설 네트워크를 통해 VSI에 대한 액세스를 확인하십시오. 
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## VRA를 통해 VLAN 액세스 라우트
{: #routing_vlan_via_vra}

가상 서버의 사설 VLAN이 {{site.data.keyword.Bluemix_notm}} 관리 시스템에 의해 이 VRA에 연관됩니다. 이 단계에서 VSI는 여전히 {{site.data.keyword.Bluemix_notm}} 사설 네트워크의 IP 라우팅을 통해 액세스할 수 있습니다. 이제 VRA를 통해 서브넷을 라우팅하여 보안 사설 네트워크를 작성하고 이제 VSI에 액세스할 수 없음을 확인하여 유효성 검증합니다.  

1. [게이트웨이 어플라이언스](https://{DomainName}/classic/network/gatewayappliances) 페이지를 통해 VRA에 대한 게이트웨이 세부사항으로 진행하여 페이지의 하단부에서 **연관된 VLAN** 섹션을 찾으십시오. 연관된 VLAN이 여기에 나열됩니다.  
2. 지금 추가적인 VLAN을 추가하는 것이 바람직한 경우에는 **VLAN 연관** 섹션으로 이동하십시오. *VLAN 선택* 드롭 다운 상자가 사용으로 설정되어야 하고 기타 프로비저닝된 VLAN을 선택할 수 있습니다. ![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png)

   적격 VLAN이 표시되지 않는 경우에는 VRA와 동일한 라우터에서 VLAN을 사용할 수 없습니다. 이로 인해 VRA와 동일한 라우터에서 사설 VLAN을 요청하기 위해 [지원 티켓](https://{DomainName}/unifiedsupport/cases/add)을 발생시켜야 합니다.
   {:tip}
5. VRA와 연관시킬 VLAN을 선택한 후 저장을 클릭하십시오. 초기 VLAN 연관을 완료하는 데는 몇 분이 걸릴 수 있습니다. 완료되고 나면 **연관된 VLAN** 표제 아래에 VLAN이 표시되어야 합니다.  

이 단계에서 VLAN 및 연관된 서브넷은 보호되거나 VRA를 통해 라우팅되지 않으므로 {{site.data.keyword.Bluemix_notm}} 사설 네트워크를 통해 VSI에 액세스할 수 있습니다. VLAN의 상태는 *무시됨*으로 표시됩니다. 

4. 오른쪽 열에서 **조치**를 선택한 후 **VLAN 라우트**를 선택하여 VRA를 통해 VLAN/서브넷을 라우팅하십시오. 이 작업을 수행하는 데 몇 분 정도 걸립니다. 화면을 새로 고치면 *라우팅됨*으로 표시됩니다.  
5. [VLAN 이름](https://{DomainName}/classic/network/vlans/)을 선택하여 VLAN 세부사항을 보십시오. 프로비저닝된 VSI 및 지정된 기본 IP 서브넷이 표시될 수 있습니다. 이후 단계에서 사용되므로 사설 VLAN ID \<nnnn\>(이 예에서는 1199)을 기록해 두십시오.  
6. [서브넷](https://{DomainName}/classic/network/subnets)을 선택하여 IP 서브넷 세부사항을 확인하십시오. 향후 VRA 구성에 필요하므로 서브넷 네트워크, 게이트웨이 주소 및 CIDR(/26)을 기록해 두십시오. 64개의 기본 IP 주소가 사설 네트워크에 프로비저닝되므로 게이트웨이 주소를 찾으려면 페이지 2 또는 3을 선택해야 할 수 있습니다.  
7. 서브넷/VLAN이 VRA로 라우팅되며 `ping`을 사용하여 워크스테이션에서 관리 네트워크를 통해 VSI에 액세스 **불가능**한지 유효성 검증하십시오.  
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}

이는 {{site.data.keyword.Bluemix_notm}} 콘솔을 통한 VRA 설정을 완료합니다. 격납장치 및 IP 라우팅을 구성하는 추가 작업은 이제 SSH를 통해 VRA에서 직접 수행됩니다.  

## IP 라우팅 및 보안 격납장치 구성
{: #vra_setup}

VRA 구성이 커미트되면 실행 중인 구성이 변경되고 변경사항이 시작 구성에 자동으로 저장됩니다. 

이전의 작동 중인 구성으로 돌아가도록 권장되는 경우에는 기본적으로 마지막 20개 커미트 지점을 보거나 비교하거나 복원할 수 있습니다. 구성 커미트 및 저장에 대한 자세한 내용은 [Vyatta 네트워크 OS 기본 시스템 구성 안내서](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)를 참조하십시오. 
   ```bash
   show system commit 
   rollback n
   compare
   ```
   {: codeblock}

### VRA IP 라우팅 구성

{{site.data.keyword.Bluemix_notm}} 사설 네트워크에서 새 서브넷으로 라우팅되도록 VRA 가상 네트워크 인터페이스를 구성하십시오.   

1. SSH를 사용하여 VRA에 로그인하십시오.  
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}
2. 이전 단계에서 기록된 사설 VLAN ID, 서브넷 게이트웨이 IP 주소 및 CIDR을 사용하여 새 가상 인터페이스를 작성하십시오. CIDR은 일반적으로 /26입니다.  
   ```
   set interfaces bonding dp0bond0 vif <VLAN ID> address <Subnet Gateway IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   **`<Subnet Gateway IP>`** 주소를 사용하는 것이 중요합니다. 이는 일반적으로 서브넷 주소 시작 주소보다 1이 더 많습니다. 올바르지 않은 게이트웨이 주소를 입력하면 `Configuration path: interfaces bonding dp0bond0 vif xxxx address [x.x.x.x] is not valid` 오류가 발생합니다. 명령을 정정한 후 다시 입력하십시오. 네트워크 > IP 관리 > 서브넷에서 이를 찾을 수 있습니다. 게이트웨이 주소를 알아야 하는 서브넷을 클릭하십시오. 설명 **게이트웨이**를 가진 목록의 두 번째 항목은 <서브넷 게이트웨이 IP>/<CIDR>로 입력할 IP 주소입니다.
   {: tip}

3. 새 가상 인터페이스(vif)를 나열하십시오.  
   ```
   show interfaces
   ```
   {: codeblock}

   이는 vif 1199 및 서브넷 게이트웨이 주소를 보여주는 예제 인터페이스 구성입니다.
   ![](images/solution33-secure-network-enclosure/show_interfaces.png)
4. 워크스테이션에서 관리 네트워크를 통해 VSI에 다시 한번 액세스할 수 있는지 유효성 검증하십시오.  
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
   
   VSI에 액세스할 수 없는 경우 VRA IP 라우팅 테이블이 예상대로 구성되어 있는지 확인하십시오. 필요한 경우 라우트를 삭제한 후 다시 작성하십시오. 구성 모드에서 show 명령을 실행하기 위해 run 명령을 사용할 수 있습니다.     
   ```bash
    run show ip route <Subnet Gateway IP>
   ```
   {: codeblock}

이는 IP 라우팅 구성을 완료합니다. 

### 보안 격납장치 구성

구역 및 방화벽 규칙의 구성을 통해 보안 사설 네트워크 격납장치가 작성됩니다. 진행하기 전에 [방화벽 구성](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls)에 대한 VRA 문서를 검토하십시오.  

두 개의 구역이 정의됩니다. 
   - INSIDE: IBM 사설 및 관리 네트워크
   - APP: 사설 네트워크 격납장치 내의 사용자 VLAN 및 서브넷		

1. 방화벽 및 기본값을 정의하십시오. 
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   set 명령이 실수로 두 번 실행되는 경우 *'Configuration path xxxxxxxx is not valid. Node exists'*라는 메시지가 수신됩니다. 이는 무시할 수 있습니다. 올바르지 않은 매개변수를 변경하려면 먼저 'delete security xxxxx xxxx xxxxx'를 사용하여 노드를 삭제해야 합니다.
   {:tip}
2. {{site.data.keyword.Bluemix_notm}} 사설 네트워크 리소스 그룹을 작성하십시오. 이 주소 그룹은 격납장치 및 격납장치에서 도달할 수 있는 네트워크에 액세스할 수 있는 {{site.data.keyword.Bluemix_notm}} 사설 네트워크를 정의합니다. 두 개의 IP 주소 세트와 보안 격납장치 간 액세스가 필요하며 이 IP 주소 세트는 SSL VPN 데이터 센터와 {{site.data.keyword.Bluemix_notm}} 서비스 네트워크(백엔드/사설 네트워크)입니다. [{{site.data.keyword.Bluemix_notm}} IP 범위](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges)는 허용되어야 하는 IP 범위의 전체 목록을 제공합니다.  
   - VPN 액세스를 위해 사용하는 데이터 센터의 SSL VPN 주소를 정의하십시오. {{site.data.keyword.Bluemix_notm}} IP 범위의 SSL VPN 섹션에서 데이터 센터 또는 DC 클러스터에 대한 VPN 액세스 지점을 선택하십시오. 이 예제에서는 {{site.data.keyword.Bluemix_notm}} 런던 데이터 센터의 VPN 주소 범위를 보여줍니다.
     ```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - 대상 데이터 센터, WDC04 및 DAL01에 대한 {{site.data.keyword.Bluemix_notm}} ‘서비스 네트워크(백엔드/사설 네트워크)’의 주소 범위를 정의하십시오. 여기서 예는 WDC04(2개의 주소), DAL01 및 LON06입니다.
     ```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. 사용자 VLAN 및 서브넷에 대한 APP 구역과 {{site.data.keyword.Bluemix_notm}} 사설 네트워크에 대한 INSIDE 구역을 작성하십시오. 이전에 작성한 방화벽을 지정하십시오. 구역 정의에서는 VRA 네트워크 인터페이스 이름을 사용하여 각 VLAN과 연관된 구역을 식별합니다. APP 구역을 작성하는 명령을 사용하려면 VRA와 연관된 VLAN의 VLAN ID를 이전에 지정해야 합니다. 이는 아래에서 `<VLAN ID>`로 강조표시되어 있습니다. 
   ```
   set security zone-policy zone INSIDE description "IBM Internal network"
   set security zone-policy zone INSIDE default-action drop
   set security zone-policy zone INSIDE interface dp0bond0
   set security zone-policy zone INSIDE to APP firewall INSIDE-TO-APP 

   set security zone-policy zone APP description "Application network"
   set security zone-policy zone APP default-action drop

   set security zone-policy zone APP interface dp0bond0.<VLAN ID> 
   set security zone-policy zone APP to INSIDE firewall APP-TO-INSIDE 
   ```
   {: codeblock}
4. 구성을 커미트하고 워크스테이션에서 ping을 사용하여 방화벽이 VRA를 통한 VSI로의 트래픽을 거부하고 있는지 확인하십시오.  
   ```
   commit
   ```
   {: codeblock}

   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
5. udp, tcp 및 icmp에 대한 방화벽 액세스 규칙을 정의하십시오. 
   ```
   set security firewall name INSIDE-TO-APP rule 200 protocol icmp
   set security firewall name INSIDE-TO-APP rule 200 icmp type 8
   set security firewall name INSIDE-TO-APP rule 200 action accept 
   set security firewall name INSIDE-TO-APP rule 200 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 100 action accept 
   set security firewall name INSIDE-TO-APP rule 100 protocol tcp
   set security firewall name INSIDE-TO-APP rule 100 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 110 action accept 
   set security firewall name INSIDE-TO-APP rule 110 protocol udp
   set security firewall name INSIDE-TO-APP rule 110 source address ibmprivate
   commit

   set security firewall name APP-TO-INSIDE rule 200 protocol icmp
   set security firewall name APP-TO-INSIDE rule 200 icmp type 8
   set security firewall name APP-TO-INSIDE rule 200 action accept 
   set security firewall name APP-TO-INSIDE rule 200 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 100 action accept 
   set security firewall name APP-TO-INSIDE rule 100 protocol tcp
   set security firewall name APP-TO-INSIDE rule 100 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 110 action accept 
   set security firewall name APP-TO-INSIDE rule 110 protocol udp
   set security firewall name APP-TO-INSIDE rule 110 destination address ibmprivate
   commit
   ```
   {: codeblock}
6. 방화벽 액세스의 유효성을 검증하십시오.  
   - INSIDE-TO-APP 방화벽이 이제 로컬 시스템으로부터의 ICMP 및 udp/tcp 트래픽을 허용하는지 확인하십시오.
     ```bash
     ping <VSI Private IP Address>
     SSH root@<VSI Private IP Address>
     ```
     {: codeblock}
   - APP-TO-INSIDE 방화벽이 ICMP 및 udp/tcp 트래픽을 허용하는지 확인하십시오. SSH를 사용하여 VSI에 로그인한 후 10.0.80.11 및 10.0.80.12에서 {{site.data.keyword.Bluemix_notm}} 이름 서버 중 하나에 대해 ping을 실행하십시오.
     ```bash
     SSH root@<VSI Private IP Address>
     [root@vsi  ~]# ping 10.0.80.11 
     ```
     {: codeblock}
7. 워크스테이션에서 SSH를 통해 VRA 관리 인터페이스에 대한 지속적 액세스의 유효성을 검증하십시오. 액세스가 유지되는 경우 구성을 검토하고 저장하십시오. 그렇지 않으면 VRA를 다시 부팅하면 작동 중인 구성으로 다시 돌아갑니다.  
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   ```
   show security  
   ```
   {: codeblock}

### 방화벽 규칙 디버깅

VRA 작동 가능 명령 프롬프트에서 방화벽 로그를 볼 수 있습니다. 이 구성에서는 올바르지 않은 방화벽 구성의 진단을 지원하기 위해 각 구역에 대해 삭제된 트래픽만 로그됩니다.   

1. 방화벽 로그에서 거부된 트래픽을 검토하십시오. 로그를 주기적으로 검토하면 APP 구역에 있는 서버가 IBM 네트워크에서 서비스에 접속을 올바르게 시도하고 있는지 아니면 잘못 시도하고 있는지 식별됩니다.  
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. 서비스 또는 서버에 접속할 수 없는 경우 방화벽 로그에 아무것도 표시되지 않습니다. 이전의 `<VLAN ID>`를 사용하여 VLAN에 대한 VRA 인터페이스 또는 {{site.data.keyword.Bluemix_notm}} 사설 네트워크로부터의 VRA 네트워크 인터페이스에 예상 ping/ssh IP 트래픽이 있는지 확인하십시오. 
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN ID> traffic
   ```

## VRA 보안
{: #securing_the_vra}

1. VRA 보안 정책을 적용하십시오. 기본적으로 정책 기반 방화벽 구역화는 VRA 자체에 대한 액세스에 보안을 설정하지 않습니다. 이는 CPP(Control Plane Policing)를 통해 구성됩니다. VRA는 기본 CPP 규칙 세트를 템플리트로 제공합니다. 이를 구성에 병합하십시오. 
   ```bash
   merge /opt/vyatta/etc/cpp.conf 
   ```
   {: codeblock}

이는 `CPP`라는 새 방화벽 규칙 세트를 작성하고 추가 규칙을 보고 \[edit\] 모드에서 커미트합니다.  
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. 공용 SSH 액세스에 보안을 설정하십시오. 이번에는 Vyatta 펌웨어에 대한 미해결 문제로 인해 공용 네트워크를 통한 SSH 관리 액세스를 제한하기 위해 `set service SSH listen-address x.x.x.x`를 사용하는 것이 권장되지 않습니다. 또는 VRA 공용 인터페이스에서 사용하는 공인 IP 주소의 범위에 대한 CPP 방화벽을 통해 외부 액세스를 차단할 수 있습니다. 여기서 사용된 `<VRA Public IP Subnet>`은 `<VRA Public IP Address>`와 동일하며 마지막 옥텟은 0입니다(x.x.x.0).  
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <VRA Public IP Subnet>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit 
   ```
   {: codeblock}
3. IBM 내부 네트워크를 통한 VRA SSH 관리 액세스의 유효성을 검증하십시오. 커미트를 수행한 후 SSH를 통한 VRA에 대한 액세스가 유실되는 경우 조치 드롭 다운 메뉴를 통해 VRA의 디바이스 세부사항 페이지에서 사용 가능한 KVM 콘솔을 통해 VRA에 액세스할 수 있습니다. 

이는 VLAN 및 서브넷이 포함된 단일 방화벽 구역을 보호하는 보안 사설 네트워크 격납장치의 설정을 완료합니다. 동일한 지시사항을 수행하여 추가적인 방화벽 구역, 규칙, 가상 및 베어메탈 서버, VLAN 및 서브넷을 추가할 수 있습니다.  

## 리소스 제거
{: #removeresources}

이 단계에서는 위에서 작성한 사항을 제거하기 위해 리소스를 정리합니다. 

VRA는 월별 유료 플랜을 기반으로 합니다. 취소해도 환불이 되지 않습니다. 다음 달에 이 VRA가 다시 필요하지 않은 경우에만 취소하는 것이 좋습니다. 이중 VRA 고가용성 클러스터가 필요한 경우 [게이트웨이 세부사항](https://{DomainName}/classic/network/gatewayappliances/) 페이지에서 이 단일 VRA를 업그레이드할 수 있습니다.
{:tip}  

- 모든 Virtual Server 또는 Bare Metal Server를 취소하십시오. 
- VRA를 취소하십시오. 
- 지원 티켓을 사용하여 추가적인 VLAN을 취소하십시오.  

## 관련 컨텐츠
{: #related}

- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [정적 및 이동 가능 IP 서브넷](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence Platform](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Vyatta 문서](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
