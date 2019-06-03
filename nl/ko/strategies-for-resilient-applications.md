---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-11"
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

# 복원성 애플리케이션에 대한 전략
{: #strategies-for-resilient-applications}

컴퓨팅 옵션(Kubernetes, Cloud Foundry, Cloud Functions 또는 Virtual Servers)에 관계없이 엔터프라이즈에서는 작동 중단 시간을 최소화하고 최대 가용성을 달성하는 복원성 아키텍처를 작성하려고 합니다. 이 튜토리얼에서는 복원성 솔루션을 빌드하고 그 과정에서 다음과 같은 질문에 답하는 IBM Cloud의 기능을 강조합니다. 

- 솔루션을 글로벌하게 사용할 수 있도록 준비할 때 고려해야 할 사항은 무엇입니까? 
- 사용 가능한 컴퓨팅 옵션이 다중 지역 애플리케이션을 전달하는 데 어떤 도움이 됩니까? 
- 애플리케이션 또는 서비스 아티팩트를 추가적인 지역으로 가져오려면 어떻게 해야 합니까? 
- 데이터베이스를 여러 위치에서 복제하려면 어떻게 해야 합니까? 
- Block Storage, File Storage, Object Storage, 데이터베이스 중 어떤 지원 서비스를 사용해야 합니까? 
- 서비스별 고려사항이 있습니까? 

## 목표
{: #objectives}

* 복원성 애플리케이션을 빌드할 때 관련된 아키텍처 개념에 대해 학습합니다. 
* 이러한 개념이 IBM Cloud 컴퓨팅 및 서비스 오퍼링에 맵핑되는 방식에 대해 이해합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처 및 개념

{: #architecture}

복원성 아키텍처를 설계하려면 솔루션의 개별 블록 및 해당 특정 기능을 고려해야 합니다.  

다중 지역 설정에 존재할 수 있는 다양한 컴포넌트를 보여주는 다중 지역 아키텍처가 아래에 있습니다. ![아키텍처](images/solution39/Architecture.png)

위 아키텍처 다이어그램은 컴퓨팅 옵션에 따라 다를 수 있습니다. 이후 절의 각 컴퓨팅 옵션 아래에서 특정 아키텍처 다이어그램을 볼 수 있습니다.  

### 2개 지역에 대한 재해 복구 

재해 복구를 활용하기 위해 두 개의 광범위하게 승인되는 아키텍처가 사용됩니다(**능동/능동** 및 **능동/수동**). 각 아키텍처는 복구에 소요되는 시간 및 노력과 관련된 자체 비용 및 이점을 가지고 있습니다. 

#### 능동-능동 구성

능동/능동 아키텍처에서는 두 위치가 모두 둘 사이에서 트래픽을 분배하는 로드 밸런서를 가진 동일한 능동 인스턴스를 가지고 있습니다. 이 접근 방식을 사용하는 경우에는 실시간으로 두 지역 간 데이터를 동기화하기 위해 데이터 복제가 수행되어야 합니다. 

![능동/능동](images/solution39/Active-active.png)

이 구성에서는 능동/수동 아키텍처보다 적은 수동 조치방안으로 더 높은 가용성을 제공합니다. 요청은 두 데이터 센터 모두에서 제공됩니다. 적절한 제한시간과 함께 에지 서비스(로드 밸런서)를 구성하고 첫 번째 데이터 센터 환경에서 장애가 발생하는 경우 요청을 두 번째 데이터 센터로 자동으로 라우트하는 로직을 재시도해야 합니다. 

능동/능동 시나리오에서 **복구 지점 목표**(RPO)를 고려할 때 원활한 요청 플로우를 허용하기 위해 두 능동 데이터 센터 간 데이터 동기화는 매우 시간을 잘 맞춰야 합니다. 

#### 능동-수동 구성

능동/수동 아키텍처는 하나의 능동 지역과 백업으로 사용되는 두 번째(수동) 지역에 의존합니다. 능동 지역에서 가동 중단이 발생하는 경우 수동 지역이 능동 상태가 됩니다. 데이터베이스 또는 파일 스토리지가 애플리케이션 및 사용자에 맞도록 하기 위해 수동 개입이 필요할 수 있습니다.  

![능동/능동](images/solution39/Active-passive.png)

요청은 능동 사이트에서 제공됩니다. 가동 중단 또는 애플리케이션 장애 발생 시 대기 데이터 센터가 요청을 제공할 준비가 되어 있도록 사전 애플리케이션 작업이 수행됩니다. 능동에서 수동 데이터 센터로 전환하는 것은 시간이 걸리는 작업입니다. **복구 시간 목표**(RTO)와 **복구 지점 목표**(RPO)는 능동/능동 구성보다 높습니다. 

### 3개 지역에 대한 재해 복구

작동 중단 시간을 허용하지 않는 오늘날의 "상시 접속된" 서비스 시대에 고객은 모든 비즈니스 서비스가 전 세계 어디서든 항상 액세스 가능하길 기대하고 있습니다. 엔터프라이즈에 대한 비용 효율이 높은 전략에는 재해 복구 인프라를 빌드하는 대신 지속적 가용성을 위한 인프라를 구성하는 것이 포함되어 있습니다. 

3개의 데이터 센터를 사용하면 2개를 사용할 때보다 높은 복원성 및 가용성이 제공됩니다. 또한 데이터 센터에 균등하게 로드를 분산시켜 성능을 향상시킬 수 있습니다. 엔터프라이즈에 2개의 데이터 센터만 있는 경우 2개의 애플리케이션을 하나의 데이터 센터에 배치하고 세 번째 애플리케이션은 두 번째 데이터 센터에 배치하는 방식으로 변형됩니다. 또는 비즈니스 로직 및 프리젠테이션 계층을 3 능동 토폴로지에 배치하고 데이터 계층을 2 능동 토폴로지에 배치할 수 있습니다. 

#### 능동-능동-능동(3 능동) 구성

![](images/solution39/Active-active-active.png)

요청은 3개 능동 데이터 센터에서 실행 중인 애플리케이션에서 제공됩니다. IBM.com 웹 사이트에 대한 사례 연구에서는 3 능동에는 클러스터당 50%의 컴퓨팅, 메모리 및 네트워크 용량만 필요하지만 2 능동에서는 클러스터당 100%가 필요함을 보여줍니다. 데이터 계층은 비용 차이가 두드러지는 위치입니다. 자세한 내용을 보려면 [*Always On: Assess, Design, Implement, and Manage Continuous Availability*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf)를 읽으십시오. 

#### 능동-능동-수동 구성

![](images/solution39/Active-active-passive.png)

이 시나리오에서는 기본 및 보조 데이터 센터에 있는 두 능동 애플리케이션 중 하나가 가동 중단되면 세 번째 데이터 센터에 있는 대기 애플리케이션이 활성화됩니다. 정상 상태를 복원하여 고객 요청을 처리하기 위해 2개 데이터 센터 시나리오에서 설명된 재해 복구 프로시저가 수행됩니다. 세 번째 데이터 센터에 있는 대기 애플리케이션은 핫 또는 콜드 스탠바이 구성에서 설정될 수 있습니다. 

재해 복구에 대한 자세한 내용은 [이 안내서](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/)를 참조하십시오. 

### 다중 지역 아키텍처

다중 지역 아키텍처에서는 각각의 지역이 애플리케이션의 동일한 사본을 실행하는 다양한 위치에 애플리케이션이 배치됩니다.  

지역은 앱, 서비스 및 기타 {{site.data.keyword.cloud_notm}} 리소스를 배치할 수 있는 특정 지리적 위치입니다. [{{site.data.keyword.cloud_notm}} 지역](https://{DomainName}/docs/containers?topic=containers-regions-and-zones)은 컴퓨팅, 네트워크 및 스토리지 리소스를 호스팅하는 실제 데이터 센터와 서비스 및 애플리케이션을 호스팅하는 관련 냉각 및 전원인 하나 이상의 구역으로 구성됩니다. 구역은 서로 격리되어 있어 장애가 공유되지 않습니다. 

또한 다중 지역 아키텍처에서는 지역 사이에 트래픽을 분배하기 위해 글로벌 로드 밸런서(예: [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services))가 필요합니다. 

여러 지역에 솔루션을 배치하면 다음과 같은 이점이 있습니다. 
- 일반 사용자의 대기 시간 향상 - 속도가 중요합니다. 백엔드 원본이 일반 사용자와 더 가까울수록 사용자의 경험이 향상되고 속도도 빨라집니다. 
- 재해 복구 - 능동 지역이 실패하면 신속하게 복구할 백업 지역이 있습니다. 
- 비즈니스 요구사항 - 일부 경우에는 수백 킬로미터 떨어진 별개의 지역에 데이터를 저장해야 합니다. 따라서 이 경우에는 여러 지역에 데이터를 저장해야 합니다.  

### 지역 내 다중 구역 아키텍처

다중 구역 지역 애플리케이션을 빌드하는 것은 지역 내 여러 구역에 애플리케이션을 배치하는 것을 의미하며 지역이 둘 또는 셋 있을 수도 있습니다.  

다중 구역 지역 아키텍처를 사용하는 경우에는 로컬 로드 밸런서가 지역의 구역 사이에서 로컬로 트래픽을 분배하도록 요구하며 두 번째 지역이 설정된 경우에는 글로벌 로드 밸런서가 지역 사이에서 트래픽을 분배합니다.  

[여기서](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones) 지역 및 구역에 대한 자세한 정보를 볼 수 있습니다. 

## 컴퓨팅 옵션 

이 절에서는 {{site.data.keyword.cloud_notm}}에서 사용 가능한 컴퓨팅 옵션을 검토합니다. 각각의 컴퓨팅 옵션에 대해 아키텍처 배치 방법에 대한 튜토리얼과 함께 아키텍처 다이어그램이 제공됩니다. 

참고: 일부 컴퓨팅 옵션 아키텍처에는 데이터베이스 또는 기타 서비스가 포함되어 있지 않습니다. 이들은 선택된 컴퓨팅 옵션을 위해 2개 지역에 앱을 배치하는 데만 중점을 둡니다. 다중 지역 컴퓨팅 옵션 예제를 배치하고 나면 다음 논리적 단계는 데이터베이스 및 기타 서비스를 추가하는 것입니다. 이 솔루션 튜토리얼의 이후 절에서 [데이터베이스](#databaseservices) 및 [비데이터베이스 서비스](#nondatabaseservices)에 대해 다룹니다. 

### Cloud Foundry 

Cloud Foundry는 다중 지역 아키텍처의 배치를 달성하는 기능을 제공하며 [지속적 딜리버리](https://{DomainName}/catalog/services/continuous-delivery) 파이프라인 서비스를 사용하면 여러 지역에 애플리케이션을 배치할 수 있습니다. Cloud Foundry 다중 지역에 대한 아키텍처의 모양은 다음과 비슷합니다. 

![CF 아키텍처](images/solution39/CF2-Architecture.png)

동일한 애플리케이션이 여러 지역에 배치되고 글로벌 로드 밸런서가 트래픽을 가장 가까운 정상 지역으로 라우트합니다. [**여러 지역에서 웹 애플리케이션 보안**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp) 튜토리얼에서 비슷한 아키텍처의 배치에 대해 설명합니다. 

### {{site.data.keyword.cfee_full_notm}}

**{{site.data.keyword.cfee_full_notm}}(CFEE)**는 추가적인 기능과 함께 퍼블릭 Cloud Foundry와 동일한 모든 기능을 제공합니다. 

**{{site.data.keyword.cfee_full_notm}}**를 사용하면 요청 시 여러 개의 격리된 엔터프라이즈급 Cloud Foundry 플랫폼을 인스턴스화할 수 있습니다. CFEE의 인스턴스는 [{{site.data.keyword.cloud_notm}}](http://ibm.com/cloud)에서
사용자의 자체 계정에서 실행됩니다. 환경은 [{{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service) 위에 격리된 하드웨어에 배치됩니다. 액세스 제어, 용량 관리, 변경 관리, 모니터링 및 서비스를 포함하여 환경을 완전히 제어할 수 있습니다. 

{{site.data.keyword.cfee_full_notm}}를 사용하는 다중 지역 아키텍처가 아래에 있습니다. 

![아키텍처](images/solution39/CFEE-Architecture.png)

이 아키텍처를 배치하려면 다음과 같은 작업을 수행해야 합니다.  

- 2개의 CFEE 인스턴스(각 지역에 하나씩)를 설정하십시오. 
- 서비스를 작성하여 CFEE 계정에 바인드하십시오.  
- CFEE API 엔드포인트를 대상으로 하여 앱을 푸시하십시오.  
- 퍼블릭 Cloud Foundry의 경우와 마찬가지로 데이터베이스 복제를 설정하십시오.  

또한 단계별 안내서 [Cloud Foundry Enterprise Environment(CFEE)에 물류 마법사 배치](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md)를 확인하십시오. 이 안내서에서는 마이크로서비스 기반 애플리케이션을 CFEE에 배치하는 것에 대해 설명합니다. 하나의 CFEE 인스턴스에 배치되고 나면 프로시저를 두 번째 지역에 복제하고 [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)를 두 CFEE 인스턴스 앞에 접속시켜 트래픽의 로드 밸런싱을 수행할 수 있습니다.  

자세한 내용은 [{{site.data.keyword.cfee_full_notm}} 문서](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about)를 참조하십시오. 

### Kubernetes

Kubernetes를 사용하면 지역 아키텍처 내에서 다중 구역을 달성할 수 있으며 이는 능동/능동 유스 케이스일 수 있습니다. {{site.data.keyword.containershort_notm}}를 사용하여 솔루션을 구현하면 로드 밸런싱 및 격리, 호스트, 네트워크 또는 앱의 잠재적 장애에 대한 복원성 향상 등의 기본 제공 기능의 혜택을 받습니다. 다중 클러스터를 작성하면 한 클러스터에서 가동 중단이 발생해도 사용자는 또 다른 클러스터에도 배치되는 앱에 여전히 액세스할 수 있습니다. 서로 다른 지역에 다중 클러스터가 있는 경우 사용자는 가장 가까운 클러스터에 액세스하여 네트워크 대기 시간도 줄일 수 있습니다. 추가적인 복원성을 위해 다중 구역 클러스터를 선택하여 지역 내 여러 구역에 노드를 배치하는 옵션도 가지고 있습니다.  

Kubernetes 다중 지역 아키텍처의 모양은 다음과 비슷합니다. 

![Kubernetes](images/solution39/Kub-Architecture.png)

1. 개발자가 애플리케이션에 대한 Docker 이미지를 빌드합니다. 
2. 이미지가 두 개의 다른 위치로 {{site.data.keyword.registryshort_notm}}에 푸시됩니다. 
3. 애플리케이션이 해당 두 위치의 Kubernetes 클러스터에 배치됩니다.
4. 일반 사용자가 애플리케이션에 액세스합니다. 
5. Cloud Internet Services는 애플리케이션에 대한 요청을 인터셉트하고 로드를 클러스터 전체에 분배하도록 구성됩니다. 또한 공통 위협으로부터 애플리케이션을 보호하기 위해 DDoS 보호 및 웹 애플리케이션 방화벽이 사용으로 설정됩니다. 선택적으로 이미지, CSS 파일 등의 자산이 캐시에 저장됩니다. 

[**Cloud Internet Services를 사용하는 안전한 복원성 다중 지역 Kubernetes 클러스터**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis) 튜토리엘에서는 이러한 아키텍처를 배치하는 단계에 대해 설명합니다. 

### {{site.data.keyword.openwhisk_short}}

{{site.data.keyword.openwhisk_short}}는 여러 {{site.data.keyword.cloud_notm}} 위치에서 사용할 수 있습니다. 복원성을 향상시키고 네트워크 대기 시간을 줄이기 위해 애플리케이션은 여러 위치에 백엔드를 배치할 수 있습니다. 그런 다음 IBM Cloud Internet Services(CIS)를 사용하여 개발자가 트래픽을 가장 가까운 정상 백엔드에 트래픽을 분배하는 시작점을 노출할 수 있습니다. {{site.data.keyword.openwhisk_short}} 다중 지역에 대한 아키텍처의 모양은 다음과 비슷합니다. 

 ![Functions 아키텍처](images/solution39/Functions-Architecture.png)

1. 사용자가 애플리케이션에 액세스합니다. 요청이 Internet Services를 통해 전달됩니다.
2. Internet Services가 가장 가까운 정상 API 백엔드로 사용자의 경로를 재지정합니다. 
3. Certificate Manager가 API에 SSL 인증서를 제공합니다. 트래픽은 처음부터 끝까지 암호화됩니다. 
4. Cloud Functions를 사용하여 API가 구현됩니다. 

[**여러 지역에 서버리스 앱 배치**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless) 튜토리얼을 수행하여 이 아키텍처를 배치하는 방법을 확인하십시오. 

### {{site.data.keyword.baremetal_short}} 및 {{site.data.keyword.virtualmachinesshort}}

{{site.data.keyword.virtualmachinesshort}} 및 {{site.data.keyword.baremetal_short}}는 다중 지역 아키텍처를 달성하는 기능을 제공합니다. {{site.data.keyword.cloud_notm}}의 사용 가능한 많은 위치에 서버를 프로비저닝할 수 있습니다. 

![서버 위치](images/solution39/ServersLocation.png)

{{site.data.keyword.virtualmachinesshort}} 및 {{site.data.keyword.baremetal_short}}를 사용하여 이러한 아키텍처를 준비하는 경우 파일 스토리지, 백업, 복구 및 데이터베이스(서비스로서의 데이터베이스와 가상 서버에 데이터베이스 설치 중에서 선택)를 고려하십시오.  

아래 아키텍처에서는 하나의 지역이 능동 상태이고 두 번째 지역이 수동 상태인 능동/수동 아키텍처에서 {{site.data.keyword.virtualmachinesshort}}를 사용한 다중 지역 아키텍처의 배치를 보여줍니다.  

![VM 아키텍처](images/solution39/vm-Architecture2.png)

이러한 아키텍처에 필요한 컴포넌트:  

1. 사용자가 IBM Cloud Internet Services(CIS)를 통해 애플리케이션에 액세스합니다. 
2. CIS가 트래픽을 능동 위치로 라우트합니다. 
3. 위치 내에서 로드 밸런서가 서버로 트래픽의 경로를 재지정합니다. 
4. 가상 서버에 배치된 데이터베이스(데이터베이스를 구성하고 지역 사이에서 복제 및 백업을 설정함). 대안은 튜토리얼에서 나중에 설명하는 주제인 서비스로서의 데이터베이스를 사용하는 것입니다. 
5. 애플리케이션 이미지 및 파일을 저장할 파일 스토리지. 파일 스토리지는 지정된 날짜 및 시간에 스냅샷을 찍을 수 있는 기능을 제공하며 이 스냅샷은 다른 지역 내에서 재사용할 수 있고 이는 수동으로 수행할 수 있습니다.  

[**Virtual Servers를 사용하여 확장 가능한 고가용성 웹 앱 빌드**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application) 튜토리얼에서 이 아키텍처를 구현합니다. 

## 데이터베이스 및 애플리케이션 파일
{: #databaseservices}

{{site.data.keyword.cloud_notm}}는 비즈니스 요구에 따라 관계형 데이터베이스와 비관계형 데이터베이스를 모두 가진 엄선된 [서비스로서의 데이터베이스](https://{DomainName}/catalog/?category=databases)를 제공합니다. [DBaaS(Database-as-a-Service)](https://www.ibm.com/cloud/learn/what-is-cloud-database)는 많은 장점을 제공합니다. {{site.data.keyword.cloudant}} 등의 DBaaS를 사용하면 다중 지역 지원을 활용하여 서로 다른 지역의 두 데이터베이스 서비스 간 실시간 복제를 수행하고 백업을 수행하고 스케일링 및 최대 가동 시간을 가질 수 있습니다.  

**주요 기능:** 

- 클라우드 플랫폼을 통해 빌드되고 액세스되는 데이터베이스 서비스
- 엔터프라이즈 사용자가 전용 하드웨어를 구입하지 않고도 데이터베이스를 호스팅할 수 있게 합니다. 
- 사용자가 관리하거나 서비스로 제공되어 제공자가 관리할 수 있습니다. 
- SQL 또는 NoSQL 데이터베이스를 지원할 수 있습니다. 
- 웹 인터페이스 또는 공급업체가 제공한 API를 통해 엑세스됩니다. 

**다중 지역 아키텍처에 대한 준비:**

- 데이터베이스 서비스의 복원성 옵션은 무엇입니까? 
- 여러 지역의 여러 데이터베이스 서비스 사이에서 복제를 처리하는 방법은 무엇입니까? 
- 데이터가 어떻게 백업되고 있습니까? 
- 각각에 대한 재해 복구 접근 방식은 무엇입니까? 

### {{site.data.keyword.cloudant}}

{{site.data.keyword.cloudant}}는 빠르게 성장하는 대규모 웹 및 모바일 앱에서 일반적인 무거운 워크로드를 처리하기 위해 최적화된 분산 데이터베이스입니다. 완전히 관리되는 SLA 지원 {{site.data.keyword.Bluemix_notm}} 서비스로 사용 가능한 {{site.data.keyword.cloudant}}는 독립적으로 처리량과 스토리지를 탄력적으로 스케일링합니다. {{site.data.keyword.cloudant}}는 다운로드 가능한 온프레미스 설치로도 사용 가능하며 해당 API 및 강력한 복제 프로토콜은 가장 인기가 있는 웹 및 모바일 개발 스택에 대한 라이브러리, CouchDB 및 PouchDB를 포함하는 오픈 소스 에코시스템과 호환됩니다. 

{{site.data.keyword.cloudant}}는 여러 위치의 여러 인스턴스 간 [복제](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation)를 지원합니다. 소스 데이터베이스에서 발생한 변경사항은 모두 대상 데이터베이스에서 재생성됩니다. 연속적으로 또는 '일회성' 태스크로 수에 관계없이 데이터베이스 사이에서 복제를 작성할 수 있습니다. 다음 다이어그램에서는 2개의 {{site.data.keyword.cloudant}} 인스턴스(각 지역에 하나씩)를 사용하는 일반적인 구성을 보여줍니다. 

![능동-능동](images/solution39/Active-active.png)

[이 지시사항](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery)을 참조하여 {{site.data.keyword.cloudant}} 인스턴스 간 복제를 구성하십시오. 이 서비스는 [데이터를 백업 및 복원](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery)하는 데 필요한 지시사항 및 도구도 제공합니다. 

### {{site.data.keyword.Db2_on_Cloud_short}}, {{site.data.keyword.dashdbshort_notm}} 및 {{site.data.keyword.Db2Hosted_notm}}

{{site.data.keyword.cloud_notm}}는 몇 가지 [Db2 데이터베이스 서비스](https://{DomainName}/catalog/?search=db2h)를 제공합니다. 이는 다음과 같습니다. 

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2): 일반적인 작동 가능 OLTP 유사 워크로드에 대해 완전히 관리되는 클라우드 SQL 데이터베이스입니다. 
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse): 고성능 페타바이트 스케일 분석 워크로드에 대해 완전히 관리되는 클라우드 데이터 웨어하우스 서비스입니다. SMP 서비스 플랜과 MPP 서비스 플랜을 모두 제공하며 최적화된 종횡배열 데이터 저장소 및 인메모리 처리를 활용합니다. 
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted): IBM에 의해 호스팅되고 사용자 데이터베이스 시스템에 의해 관리됩니다. Db2에 클라우드 인프라에 대한 전체 관리 액세스를 제공하여 사용자의 자체 인프라 관리의 비용, 복잡도 및 위험을 제거합니다. 

다음에서는 작동 가능 워크로드에 대해 DBaaS로서의 {{site.data.keyword.Db2_on_Cloud_short}}에 중점을 둡니다. 이 워크로드는 이 튜토리얼에서 설명하는 애플리케이션의 경우 일반적입니다. 

#### {{site.data.keyword.Db2_on_Cloud_short}}에 대한 다중 지역 지원

{{site.data.keyword.Db2_on_Cloud_short}}는 [HADR(High Availability and Disaster Recovery)을 달성하기 위한 몇몇 옵션](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview)을 제공합니다. 새 서비스를 작성할 때 고가용성 옵션을 선택할 수 있습니다. 나중에 인스턴스 대시보드를 통해 [지리적으로 복제된 재해 복구 노드를 추가](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha)할 수 있습니다. 오프사이트 DR 노드 옵션을 사용하면 선택한 오프사이트 {{site.data.keyword.cloud_notm}} 데이터 센터에서 데이터를 실시간으로 데이터베이스 노드와 동기화할 수 있습니다. 

자세한 정보는 [고가용성 문서](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha)에서 확인할 수 있습니다. 

#### 백업 및 복원

{{site.data.keyword.Db2_on_Cloud_short}}에는 유료 플랜에 대한 일일 백업이 포함되어 있습니다. 일반적으로 백업은 {{site.data.keyword.cos_short}}를 사용하여 저장되므로 보유한 데이터의 가용성을 증가시키기 위해 3개의 데이터 센터를 활용합니다. 백업은 14일 동안 보관됩니다. 백업을 사용하여 특정 시점 복구를 수행할 수 있습니다. [백업 및 복원 문서](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br)에서는 데이터를 원하는 날짜 및 시간으로 복원하는 방법에 대한 세부사항을 제공합니다. 

### {{site.data.keyword.databases-for}}

{{site.data.keyword.databases-for}}는 몇몇 오픈 소스 데이터베이스 시스템을 완전히 관리되는 서비스로 제공합니다. 이는 다음과 같습니다.   
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

이 서비스는 모두 동일한 특성을 공유합니다.    
* 고가용성을 위해 클러스터에 배치됩니다. 각 서비스의 문서에서 세부사항을 찾을 수 있습니다. 
  - [PostgreSQL](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch ](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* 각 클러스터가 여러 구역에 분산되어 있습니다. 
* 데이터가 여러 구역에 걸쳐 복제됩니다. 
* 사용자가 인스턴스에 대한 스토리지 및 메모리 리소스를 확장할 수 있습니다. 세부사항은 [스케일링에 대한 문서(예: {{site.data.keyword.databases-for-redis}})](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings)를 참조하십시오. 
* 매일 또는 요구 시 백업이 수행됩니다. 각 서비스에 대한 세부사항이 문서화되어 있습니다. [백업 문서(예: {{site.data.keyword.databases-for-postgresql}})](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups)가 여기에 있습니다. 
* 저장 데이터, 백업 및 네트워크 트래픽이 암호화됩니다. 
* [{{site.data.keyword.databases-for}} CLI 플러그인을 사용하여 각각의 서비스를 관리할 수 있습니다. ](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in)


### {{site.data.keyword.cos_full_notm}}

{{site.data.keyword.cos_full_notm}}(COS)는 지속 가능하고 안전한 비용 효율적인 클라우드 스토리지를 제공합니다. {{site.data.keyword.cos_full_notm}}를 사용하여 저장된 정보는 암호화되어 여러 지리적 위치에 분산됩니다. COS 인스턴스 내에서 스토리지 버킷을 작성할 때 버킷을 작성해야 하는 위치와 사용할 복원성 옵션을 결정합니다. 

세 가지 유형의 버킷 복원성이 있습니다. 
   - **교차 지역** 복원성은 데이터를 몇몇 대도시 영역에 분산시킵니다. 이는 다중 지역 옵션으로 표시될 수 있습니다. 교차 지역 버킷에 저장된 컨텐츠에 액세스하는 경우 COS는 정상 지역에서 컨텐츠를 검색할 수 있는 특수 엔드포인트를 제공합니다. 
   - **지역** 복원성은 데이터를 단일 대도시 영역에 분산시킵니다. 이는 지역 구성 내의 다중 구역으로 표시될 수 있습니다. 
   - **단일 데이터 센터** 복원성은 데이터를 단일 데이터 센터 내의 여러 어플라이언스에 분산시킵니다. 

{{site.data.keyword.cos_full_notm}} 복원성 옵션에 대한 자세한 설명은 [이 문서](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints)를 참조하십시오. 

### {{site.data.keyword.filestorage_full_notm}}

{{site.data.keyword.filestorage_full_notm}}는 빠르고 지속적이고 유연한 네트워크에 접속된 NFS 기반 파일 스토리지입니다. 이 NAS(Network-Attached Storage) 환경에서는 파일 공유 기능 및 성능을 완전히 제어할 수 있습니다. 복원성을 위해 라우팅된 TCP/IP 연결을 통해 최대 64개의 권한 부여된 디바이스에 {{site.data.keyword.filestorage_short}} 공유를 연결할 수 있습니다. 

파일 스토리지 기능 중 일부로는 _스냅샷_, _복제_, _동시 액세스_가 있습니다. 전체 기능 목록은 [문서](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage)를 참조하십시오. 

서버에 접속되면 {{site.data.keyword.filestorage_short}} 서비스를 사용하여 데이터 백업 및 애플리케이션 파일(예: 이미지 및 동영상)을 쉽게 저장할 수 있으며 동일한 지역의 다양한 서버에서 이 이미지 및 파일을 사용할 수 있습니다. 

두 번째 지역을 추가할 때는 {{site.data.keyword.filestorage_short}}의 스냅샷 기능을 사용하여 자동 또는 수동으로 스냅샷을 찍은 후 두 번째 수동 지역에서 재사용할 수 있습니다.  

스냅샷을 원격 데이터 센터의 대상 볼륨에 자동으로 복사하기 위해 복제를 스케줄할 수 있습니다. 치명적인 이벤트가 발생하거나 데이터가 손상되는 경우 원격 사이트에서 사본을 복구할 수 있습니다. File Storage 스냅샷에 대한 자세한 내용은 [여기서](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots) 찾을 수 있습니다. 

## 비데이터베이스 서비스
{: #nondatabaseservices}

{{site.data.keyword.cloud_notm}}는 엄선된 비데이터베이스 [서비스](https://{DomainName}/catalog)를 제공하며 이 서비스는 IBM 서비스이기도 하고 서드파티 서비스이기도합니다. 다중 지역 아키텍처에 대한 플랜을 작성하는 경우 서비스(예: Watson 서비스)가 다중 지역 설정에서 작동하는 방법에 대해 이해해야 합니다. 

### {{site.data.keyword.conversationfull}}

[{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about)는 개발자와 비기술 사용자가 대화형 AI 기반 어시스턴트를 빌드하기 위해 협업할 수 있게 하는 플랫폼입니다. 

어시스턴트는 비즈니스 요구에 맞게 사용자 정의하고 고객이 필요로 할 때 고객에게 도움말을 제공하기 위해 여러 채널에 배치할 수 있는 코그너티브 봇입니다. 어시스턴트에는 하나 이상의 스킬이 포함되어 있습니다. 대화 스킬에는 어시스턴트가 고객을 도울 수 있게 하는 교육 데이터 및 로직이 포함되어 있습니다. 

{{site.data.keyword.conversationshort}} V1은 Stateless라는 점에 유의해야 합니다. {{site.data.keyword.conversationshort}}은 99.5%의 가동 시간을 제공하지만 여러 지역에 있는 고가용성 애플리케이션을 위해 여전히 여러 지역에 이 서비스의 여러 인스턴스를 가지고 있길 원할 수 있습니다.  

여러 위치에서 인스턴스를 작성했으면 {{site.data.keyword.conversationshort}} 도구를 사용하여 한 인스턴스에서 기존 작업공간(의도, 엔티티 및 대화 상자)을 내보내십시오. 그런 다음 이 작업공간을 다른 위치로 가져오십시오. 

## 요약

| 오퍼링   | 복원성 옵션        |
| -------- | ------------------ |
| Cloud Foundry | <ul><li>여러 위치에 애플리케이션 배치</li><li>Cloud Internet Services를 사용하여 여러 위치에서의 요청 제공</li><li>Cloud Foundry API를 사용하여 조직 및 영역을 구성하고 여러 위치에 앱 푸시</li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>여러 위치에 애플리케이션 배치</li><li>Cloud Internet Services를 사용하여 여러 위치에서의 요청 제공</li><li>Cloud Foundry API를 사용하여 조직 및 영역을 구성하고 여러 위치에 앱 푸시</li><li>Kubernetes 서비스에서 빌드</li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>다중 구역 클러스터를 지원하는 설계에 의한 복원성</li><li>Cloud Internet Services를 사용하여 여러 위치에 분산된 클러스터로부터의 요청 제공</li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>여러 위치에서 사용 가능</li><li>Cloud Internet Services를 사용하여 여러 위치에서의 요청 제공</li><li>Cloud Functions API를 사용하여 여러 위치에 조치 배치</li></ul> |
| {{site.data.keyword.baremetal_short}} 및 {{site.data.keyword.virtualmachinesshort}} | <ul><li>여러 위치에 서버 프로비저닝</li><li>동일한 위치에 있는 서버를 로컬 로드 밸런서에 접속</li><li>Cloud Internet Services를 사용하여 여러 위치에서의 요청 제공</li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>데이터베이스 간 일회성 및 연속 복제</li><li>지역 내 자동 데이터 중복성</li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>실시간 데이터 동기화를 위해 지리적으로 복제된 재해 복구 노드 프로비저닝</li><li>유료 플랜을 사용하여 일일 백업</li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>다중 구역 Kubernetes 클러스터에서 빌드</li><li>교차 지역에서 읽은 복제본</li><li>일일 및 요구 시 백업</li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>단일 데이터 센터, 지역 및 교차 지역 복원성</li><li>API를 사용하여 스토리지 버킷에서 컨텐츠 동기화</li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>스냅샷을 사용하여 원격 데이터 센터에 있는 대상에 대한 컨텐츠 자동 캡처</li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>Watson API를 사용하여 여러 위치의 여러 인스턴스 사이에서 작업공간 스펙 내보내기 및 가져오기</li></ul> |

## 관련 컨텐츠

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [다중 구역 클러스터를 사용하여 앱 가용성 향상](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry, 여러 지역에서 웹 애플리케이션 보안](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [Cloud Functions, 다중 지역에 서버리스 앱 배치](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes, Cloud Internet Services를 사용하는 안전한 복원성 다중 지역 Kubernetes 클러스터](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [Virtual Servers, 고가용성 확장 가능 웹 앱 빌드](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
