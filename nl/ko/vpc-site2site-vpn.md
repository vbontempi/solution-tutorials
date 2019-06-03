---
copyright:
  years: 2019
lastupdated: "2019-05-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}
{:important: .important}

# 클라우드 리소스에 대한 안전한 개인용 온프레미스 액세스를 위해 VPC/VPN 게이트웨이 사용
{: #vpc-site2site-vpn}

IBM은 2019년 4월 초부터(다음 달에 연장 사용이 개방됨) 제한된 수의 고객만 VPC에 대한 조기 액세스 프로그램에 참여하도록 승인합니다. 조직에서 IBM Virtual Private Cloud에 대한 액세스를 확보하길 원하는 경우에는 이 [추천 양식](https://{DomainName}/vpc){: new_window}에 기입을 완료하면 IBM 담당자가 다음 단계와 관련하여 연락을 줍니다.
{: important}

IBM은 IBM 클라우드의 리소스를 사용하여 온프레미스 컴퓨터 네트워크를 안전하게 확장하는 다양한 방법을 제공합니다. 이를 통해 필요할 때 서버를 프로비저닝하고 더 이상 필요하지 않을 때 서버를 제거하는 탄력성을 얻을 수 있습니다. 또한 온프레미스 기능을 {{site.data.keyword.cloud_notm}} 서비스에 쉽고 안전하게 연결할 수 있습니다. 

이 튜토리얼에서는 VPC(VPC/VPN 게이트웨이) 내에서 작성된 클라우드 VPN에 온프레미스 가상 사설망(VPN) 게이트웨이를 연결하는 방법을 설명합니다. 먼저 새 {{site.data.keyword.vpc_full}}(VPC) 및 연관된 리소스(예: 서브넷, 네트워크 액세스 제어 목록(ACL), 보안 그룹 및 가상 서버 인스턴스(VSI))를 작성합니다.
VPC/VPN 게이트웨이는 온프레미스 VPN 게이트웨이에 대한 [IPsec](https://en.wikipedia.org/wiki/IPsec) 사이트 간 링크를 설정합니다. IPsec 및 [인터넷 키 교환](https://en.wikipedia.org/wiki/Internet_Key_Exchange)(IKE) 프로토콜은 보안 통신을 위해 입증된 개방형 표준입니다. 

안전한 개인용 액세스를 자세히 보여주기 위해 마이크로서비스를 VSI에 배치하여 {{site.data.keyword.cos_short}}(COS)에 액세스하여 비즈니스 애플리케이션의 라인을 나타냅니다.
COS 서비스에는 모든 액세스가 {{site.data.keyword.cloud_notm}}의 동일한 지역 내에 있을 때 개인용 무료 ingress/egress에 사용할 수 있는 직접 엔드포인트가 있습니다. 온프레미스 컴퓨터는 COS 마이크로서비스에 액세스합니다. 모든 트래픽은 VPN을 통해 플로우되므로 {{site.data.keyword.cloud_notm}}를 통해 비공개로 플로우됩니다. 

사이트 간 게이트웨이에 대한 많은 인기 있는 온프레미스 VPN 솔루션을 사용할 수 있습니다. 이 튜토리얼에서는 [strongSwan](https://www.strongswan.org/) VPN 게이트웨이를 활용하여 VPC/VPN 게이트웨이와 연결합니다. 온프레미스 데이터 센터를 시뮬레이션하기 위해 {{site.data.keyword.cloud_notm}}의 VSI에 strongSwan 게이트웨이를 설치합니다. 

{:shortdesc}
간단히 말하면 VPC를 사용하여 다음과 같은 작업을 수행할 수 있습니다. 

- {{site.data.keyword.cloud_notm}}에서 실행되는 서비스 및 워크로드에 온프레미스 시스템 연결
- COS에 대한 개인적인 저비용 연결 보장
- 온프레미스에서 실행되는 서비스 및 워크로드에 클라우드 기반 시스템 연결

## 목표
{: #objectives}

* 온프레미스 데이터 센터 또는 (가상) 프라이빗 클라우드에서 가상 프라이빗 클라우드 환경에 액세스합니다. 
* 개인 서비스 엔드포인트를 사용하여 클라우드 서비스에 안전하게 도달합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 이 튜토리얼에서는 마이크로서비스에서 COS에 액세스하는 데 네트워크 비용이 부과되지 않지만 VPC에 액세스하는 데는 표준 네트워크 비용이 발생합니다. 

## 아키텍처
{: #architecture}

다음 다이어그램에서는 앱 서버가 포함된 가상 프라이빗 클라우드를 보여줍니다. 앱 서버는 {{site.data.keyword.cos_short}} 서비스와 인터페이스로 접속하는 마이크로서비스를 호스팅합니다. (시뮬레이션된) 온프레미스 네트워크 및 가상 클라우드 환경은 VPN 게이트웨이를 통해 연결됩니다. 

![아키텍처](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. 제공된 스크립트를 사용하여 인프라(VPC, 서브넷, 규칙이 있는 보안 그룹, 네트워크 ACL 및 VSI)가 설정됩니다. 
2. 마이크로서비스가 직접 엔드포인트를 통해 {{site.data.keyword.cos_short}}와 인터페이스로 접속합니다. 
3. 가상 프라이빗 클라우드 환경을 온프레미스 네트워크에 노출하기 위해 VPC/VPN 게이트웨이가 프로비저닝됩니다. 
4. 클라우드 환경과 VPN 연결을 설정하기 위해 온프레미스에서 Strongswan 오픈 소스 IPsec 게이트웨이 소프트웨어가 사용됩니다. 

## 시작하기 전에
{: #prereqs}

- [다음과 같은 단계를 수행하여](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) 필요한 모든 명령행(CLI) 도구를 설치하십시오. 선택적 CLI 인프라 플러그인이 필요합니다. 
- 명령행을 통해 {{site.data.keyword.cloud_notm}}에 로그인하십시오. 자세한 내용은 [CLI 시작하기](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli)를 참조하십시오. 
- 사용자 권한을 확인하십시오. 사용자 계정에 VPC 리소스를 작성하고 관리할 수 있는 충분한 권한이 있는지 확인하십시오. 필요한 권한의 목록은 [VPC 사용자를 위해 필요한 권한 부여](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)를 참조하십시오. 
- 가상 서버에 연결하기 위해 SSH 키가 필요합니다. SSH 키가 없으면 [키 작성을 위한 지시사항](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites)을 참조하십시오. 
- [**jq**](https://stedolan.github.io/jq/download/)를 설치하십시오. 이는 JSON 출력을 처리하기 위해 제공된 스크립트에서 사용합니다. 

## 가상 프라이빗 클라우드에 가상 앱 서버 배치
아래에서는 마이크로서비스가 {{site.data.keyword.cos_short}}와 인터페이스로 접속하는 데 필요한 기준선 VPC 환경 및 코드를 설정하는 스크립트를 다운로드합니다. 그런 다음 {{site.data.keyword.cos_short}} 서비스를 프로비저닝하고 기준선을 설정합니다. 

### 코드 가져오기
{: #setup}
이 튜토리얼에서는 VPN 게이트웨이를 작성하기 전에 인프라 리소스의 기준선을 배치하기 위해 스크립트를 사용합니다. 마이크로서비스에 대한 이 스크립트 및 코드를 GitHub 저장소에서 사용할 수 있습니다. 

1. 애플리케이션의 코드를 가져오십시오. 
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. **vpc-tutorials**로 변경한 후 **vpc-site2site-vpn**으로 변경하여 이 튜토리얼에 대한 스크립트로 이동하십시오. 
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### 서비스 작성
이 절에서는 CLI에서 {{site.data.keyword.cloud_notm}}에 로그인한 후 {{site.data.keyword.cos_short}}의 인스턴스를 작성합니다. 

1. 로그인의 전제조건 단계를 수행했는지 확인하십시오.
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. **표준** 또는 **lite** 플랜을 사용하여 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)의 인스턴스를 작성하십시오. 
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   계정당 하나의 lite 인스턴스만 작성할 수 있습니다. {{site.data.keyword.cos_short}}의 인스턴스가 이미 있는 경우에는 해당 인스턴스를 재사용할 수 있습니다.
   {: tip}

3. 역할이 **독자**인 서비스 키를 작성하십시오. 
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. JSON 형식의 서비스 키 세부사항을 확보하여 서브디렉토리 **vpc-app-cos**의 새 파일 **credentials.json**에 저장하십시오. 이 파일은 나중에 앱에서 사용합니다. 
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### 가상 프라이빗 클라우드 기준선 리소스 작성
{: #create-vpc}
이 튜토리얼에서는 이 튜토리얼에 필요한 기준선 리소스(예: 시작 환경)를 작성하는 스크립트를 제공합니다. 이 스크립트는 기존 VPC에서 해당 환경을 생성하거나 새 VPC를 작성할 수 있습니다. 

아래에서는 설정 스크립트를 구성한 후 실행하여 이 리소스를 작성합니다. 이 스크립트는 [배스천 호스트를 사용하여 원격 인스턴스에 안전하게 액세스](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)에 설명된 대로 배스천 호스트의 설정을 통합합니다. 

1. 샘플 구성 파일을 사용할 파일 위에 복사하십시오. 

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. **config.sh** 파일을 편집하고 환경에 맞게 설정을 조정하십시오. **SSHKEYNAME**의 값을 SSH 키의 이름 또는 쉼표로 구분된 이름 목록으로 변경해야 합니다("시작하기 전에" 참조). 클라우드 지역과 일치하도록 다양한 **ZONE** 설정을 수정하십시오. 다른 모든 변수는 그대로 유지될 수 있습니다. 
3. 새 VPC에서 리소스를 작성하려면 다음과 같이 스크립트를 실행하십시오. 

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   기존 VPC를 재사용하려면 이 방식으로 해당 이름을 스크립트에 전달하십시오. **YOUR_EXISTING_VPC**를 실제 VPC 이름으로 대체하십시오. 
   ```sh
   REUSE_VPC=YOUR_EXISTING_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. 그러면 배스천 관련 리소스를 포함한 다음과 같은 리소스가 작성됩니다. 
   - 1개의 VPC(선택사항)
   - 1개의 퍼블릭 게이트웨이
   - VPC 내에 3개의 서브넷
   - ingress 및 egress 규칙을 가진 4개의 보안 그룹
   - 3개의 VSI: vpns2s-onprem-vsi(유동 IP는 ONPREM_IP임), vpns2s-cloud-vsi(유동 IP는 VSI_CLOUD_IP임) 및 vpns2s-bastion(유동 IP는 BASTION_IP_ADDRESS임)

   나중에 사용할 수 있도록 **BASTION_IP_ADDRESS**, **VSI_CLOUD_IP**, **ONPREM_IP**, **CLOUD_CIDR** 및 **ONPREM_CIDR**에 대해 리턴되는 값을 기록해 두십시오. 출력도 **network_config.sh** 파일에 저장됩니다. 이 파일은 자동화된 설정을 위해 사용할 수 있습니다. 

### 가상 사설망(VPN) 게이트웨이 및 연결 작성
아래에서는 VPN 게이트웨이 및 애플리케이션 VSI를 가진 서브넷에 대한 연관된 연결을 추가합니다. 

1. [VPC 개요](https://{DomainName}/vpc/overview) 페이지로 이동한 후 탐색 탭에서 **VPN**을 클릭하고 대화 상자에서 **새 VPN 게이트웨이**를 클릭하십시오. **VPC용 새 VPN 게이트웨이** 양식에 **vpns2s-gateway**를 이름으로 입력하십시오. 올바른 VPC, 리소스 그룹 및 **vpns2s-cloud-subnet**을 서브넷으로 선택했는지 확인하십시오. 
2. **VPC용 새 VPN 연결**은 활성화된 상태로 두십시오. **vpns2s-gateway-conn**을 이름으로 입력하십시오. 
3. **피어 게이트웨이 주소**에 대해 **vpns2s-onprem-vsi**의 유동 IP 주소(ONPREM_IP)를 사용하십시오. **20_PRESHARED_KEY_KEEP_SECRET_19**를 **사전 공유된 키**로 입력하십시오. 
4. **로컬 서브넷**의 경우 **CLOUD_CIDR**에 대해 제공된 정보를 사용하고 **피어 서브넷**의 경우 **ONPREM_CIDR**에 대해 제공된 정보를 사용하십시오. 
5. **데드 피어 발견**의 설정은 그대로 두십시오. 게이트웨이 및 연관된 연결을 작성하려면 **VPN 게이트웨이 작성**을 클릭하십시오. 
6. VPN 게이트웨이를 사용할 수 있게 될 때까지 기다리십시오(화면을 새로 고쳐야 할 수 있음). 
7. 지정된 **게이트웨이 IP** 주소를 **GW_CLOUD_IP**로 기록해 두십시오.  

### 온프레미스 가상 사설망(VPN) 게이트웨이 작성
다음으로 시뮬레이션된 온프레미스 환경의 다른 사이트에서 VPN 게이트웨이를 작성합니다. 오픈 소스 기반 IPsec 소프트웨어 [strongSwan](https://strongswan.org/)을 사용합니다. 

1. SSH를 사용하여 "온프레미스" VSI **vpns2s-onprem-vsi**에 연결하십시오. 다음을 실행하고 **ONPREM_IP**를 이전에 리턴된 IP 주소로 대체하십시오. 

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   환경에 따라 `ssh -i <path to your private key file> root@ONPREMP_IP`를 사용해야 할 수 있습니다.
   {:tip}

2. 다음으로 시스템 **vpns2s-onprem-vsi**에서 다음 명령을 실행하여 패키지 관리자를 업데이트하고 strongSwan 소프트웨어를 설치하십시오. 

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. 3개 행을 끝에 추가하여 **/etc/sysctl.conf** 파일을 구성하십시오. 다음을 위에 복사한 후 실행하십시오. 

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. 다음으로 **/etc/ipsec.secrets** 파일을 편집하십시오. 다음 행을 추가하여 소스 및 대상 IP 주소 및 이전에 구성된 사전 공유된 키를 구성하십시오. vpns2s-onprem-vsi의 유동 ID의 알려진 값으로 **ONPREM_IP**를 대체하십시오. VPC VPN 게이트웨이의 알려진 IP 주소로 **GW_CLOUD_IP**를 대체하십시오. 

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. 구성해야 하는 마지막 파일은 **/etc/ipsec.conf**입니다. 해당 파일의 끝에 다음 코드 블록을 추가하십시오. **ONPREM_IP**, **ONPREM_CIDR**, **GW_CLOUD_IP** 및 **CLOUD_CIDR**을 각각의 알려진 값으로 대체하십시오. 

   ```sh
   # basic configuration
   config setup
      charondebug="all"
      uniqueids=yes
      strictcrlpolicy=no
   
   # connection to vpc/vpn datacenter 
   # left=onprem / right=vpc
   conn tutorial-site2site-onprem-to-cloud
      authby=secret
      left=%defaultroute
      leftid=ONPREM_IP
      leftsubnet=ONPREM_CIDR
      right=GW_CLOUD_IP
      rightsubnet=CLOUD_CIDR
      ike=aes256-sha2_256-modp1024!
      esp=aes256-sha2_256!
      keyingtries=0
      ikelifetime=1h
      lifetime=8h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start
   ```
   {:pre}

6. ipsec restart를 실행하여 VPN 게이트웨이를 다시 시작한 후 상태를 확인하십시오. 

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   연결이 설정되었다고 표시되어야 합니다. 이 시스템에 대한 터미널 및 SSH 연결을 열린 상태로 유지하십시오. 

## 연결 테스트
SSH를 사용하거나 {{site.data.keyword.cos_short}}와 인터페이스로 접속하는 마이크로서비스를 배치하여 사이트 간 VPN 연결을 테스트할 수 있습니다. 

### SSH를 사용하여 테스트
VPN 연결이 설정되었는지 테스트하려면 시뮬레이션된 온프레미스 환경을 프록시로 사용하여 클라우드 기반 애플리케이션 서버에 로그인하십시오.  

1. 새 터미널에서 값을 대체한 후 다음 명령을 실행하십시오. 여기서는 strongSwan 호스트를 점프 호스트로 사용하여 VPN을 통해 애플리케이션 서버의 사설 IP 주소에 연결합니다. 

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. 연결되고 나면 SSH 연결을 닫으십시오. 

3. "온프레미스" VSI 터미널에서 VPN 게이트웨이를 중지하십시오. 
   ```sh
   ipsec stop
   ```
   {:pre}
4. 1)단계의 명령 창에서 연결을 다시 설정하십시오. 

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
   VPN 연결이 활성 상태가 아니어서 시뮬레이션된 온프레미스 환경과 클라우드 환경 사이에 직접 링크가 없기 때문에 이 명령은 성공해서는 안 됩니다. 

   배치 세부사항에 따라 이 연결은 실제로 계속 성공합니다. 왜냐하면 VPC 내 연결이 구역 전체에서 지원되기 때문입니다. 시뮬레이션된 온프레미스 VSI를 다른 VPC 또는 [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers)에 배치하는 경우에는 성공적인 액세스를 위해 VPN이 필요합니다.
   {:tip}
   
5. "온프레미스" VSI 터미널에서 VPN 게이트웨이를 다시 시작하십시오. 
   ```sh
   ipsec start
   ```
   {:pre}
 

### 마이크로서비스를 사용하여 테스트
온프레미스 VSI에서 클라우드 VSI의 마이크로서비스에 액세스하여 작동 중인 VPN 연결을 테스트할 수 있습니다. 

1. 로컬 시스템의 마이크로서비스 앱에 대한 코드를 클라우드 VSI 위에 복사하십시오. 이 명령은 배스천을 클라우드 VSI에 대한 점프 호스트로 사용합니다. **BASTION_IP_ADDRESS** 및 **VSI_CLOUD_IP**를 적절하게 대체하십시오. 
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. 다시 배스천을 점프 호스트로 사용하여 클라우드 VSI에 연결하십시오. 
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. 클라우드 VSI에서 코드 디렉토리로 변경하십시오. 
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. Python 및 Python 패키지 관리자 PIP를 설치하십시오. 
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. **pip**를 사용하여 필요한 Python 패키지를 설치하십시오. 
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. 앱을 시작하십시오. 
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. "온프레미스" VSI 터미널에서 서비스에 액세스하십시오. VSI_CLOUD_IP를 적절하게 대체하십시오. 
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
   이 명령은 JSON 오브젝트를 리턴해야 합니다. 

## 리소스 제거
{: #remove-resources}

1. VPC 관리 콘솔에서 **VPN**을 클릭하십시오. VPN 게이트웨이의 조치 메뉴에서 **삭제**를 선택하여 게이트웨이를 제거하십시오. 
2. 다음으로 탐색에서 **유동 IP**를 클릭한 후 VSI의 IP 주소를 클릭하십시오. 조치 메뉴에서 **해제**를 선택하십시오. IP 주소를 해제할지 확인하십시오. 
3. 다음으로 **가상 서버 인스턴스**로 전환한 후 인스턴스를 **삭제**하십시오. 인스턴스가 삭제되며 인스턴스의 상태가 잠시 **삭제 중**으로 유지됩니다. 종종 브라우저를 새로 고쳐야 합니다. 
4. VSI가 삭제되면 **서브넷**으로 전환하십시오. 서브넷에 접속된 퍼블릭 게이트웨이가 있는 경우 서브넷 이름을 클릭하십시오. 서브넷 세부사항에서 퍼블릭 게이트웨이를 분리하십시오. 퍼블릭 게이트웨이가 없는 서브넷은 개요 페이지에서 삭제할 수 있습니다. 서브넷을 삭제하십시오. 
5. 서브넷이 삭제된 후 **VPC**로 전환하고 VPC를 삭제하십시오. 

콘솔 사용 시 리소스 삭제 후 업데이트된 상태 정보를 보려면 브라우저를 새로 고쳐야 할 수 있습니다.
{:tip}

## 튜토리얼 확장 
{: #expand-tutorial}

이 튜토리얼에 추가하거나 이 튜토리얼을 확장하시겠습니까? 몇 가지 방법은 다음과 같습니다. 

- [로드 밸런서](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc)를 추가하여 여러 인스턴스에 인바운드 마이크로서비스 트래픽을 분배하십시오. 
- [애플리케이션을 공용 서버에 배치하고 데이터 및 서비스를 개인용 호스트에 배치하십시오. ](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend)


## 관련 컨텐츠
{: #related}

- [VPC 용어집](/docs/vpc?topic=vpc-vpc-glossary)
- [VPC용 IBM Cloud CLI 플러그인 참조서](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [REST API를 사용하는 VPC](/docs/infrastructure/vpc/example-code.html)
- 솔루션 튜토리얼: [배스천 호스트를 사용하여 원격 인스턴스에 안전하게 액세스](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
