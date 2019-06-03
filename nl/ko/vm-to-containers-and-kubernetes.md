---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-15"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# VM 기반 앱을 Kubernetes로 이동
{: #vm-to-containers-and-kubernetes}

이 튜토리얼에서는 {{site.data.keyword.containershort_notm}}를 사용하여 VM 기반 앱을 Kubernetes 클러스터로 이동하는 프로세스에 대해 설명합니다. [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)는 Docker 및 Kubernetes 기술, 직관적인 사용자 경험, 기본 제공 보안 및 격리를 결합하여 컴퓨팅 호스트의 클러스터에서 컨테이너화된 앱의 배치, 오퍼레이션, 스케일링 및 모니터링을 자동화하는 강력한 도구를 제공합니다.
{: shortdesc}

이 튜토리얼에는 기존 앱을 가져오고 앱을 컨테이너화하고 앱을 Kubernetes 클러스터에 배치하는 방법에 대한 개념이 포함되어 있습니다. VM 기반 앱을 컨테이너화하기 위해 다음과 같은 옵션 중에서 선택할 수 있습니다. 

1. 자체 마이크로서비스로 분리될 수 있는 대형 단일 앱의 컴포넌트를 식별하십시오. 이 마이크로서비스를 컨테이너화한 후 Kubernetes 클러스터에 배치할 수 있습니다. 
2. 전체 앱을 컨테이너화한 후 앱을 Kubernetes 클러스터에 배치하십시오. 

보유한 앱의 유형에 따라 앱을 마이그레이션하는 단계가 다릅니다. 이 튜토리얼을 사용하여 수행해야 하는 일반적인 단계와 앱을 마이그레이션하기 전에 고려해야 할 사항에 대해 학습할 수 있습니다. 

## 목표
{: #objectives}

- VM 기반 앱에서 마이크로서비스를 식별하는 방법을 이해하고 VM과 Kubernetes 사이에서 컴포넌트를 맵핑하는 방법을 학습합니다. 
- VM 기반 앱을 컨테이너화하는 방법에 대해 학습합니다. 
- 컨테이너를 {{site.data.keyword.containershort_notm}}의 Kubernetes 클러스터에 배치하는 방법에 대해 학습합니다. 
- 학습한 모든 내용을 실습하고 클러스터에서 **JPetStore** 앱을 실행합니다. 

## 사용되는 서비스
{: #products}

이 튜토리얼에서는 다음과 같은 {{site.data.keyword.Bluemix_notm}} 서비스를 사용합니다. 

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**주의:** 이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{:#architecture}

### VM을 사용하는 전통적인 앱 아키텍처

다음 다이어그램은 가상 머신을 기반으로 하는 전통적인 앱 아키텍처의 예를 보여줍니다. 

<p style="text-align: center;">
![아키텍처 다이어그램](images/solution30/traditional_architecture.png)

</p>

1. 사용자가 요청을 Java 앱의 공용 엔드포인트에 전송합니다. 공용 엔드포인트는 사용 가능한 앱 서버 인스턴스 간 수신 네트워크 트래픽에 대해 로드 밸런싱을 수행하는 로드 밸런서 서비스에 의해 표시됩니다. 
2. 로드 밸런서는 VM에서 실행되는 정상 앱 서버 인스턴스 중 하나를 선택한 후 요청을 전달합니다. 
3. 앱 서버는 VM에서 실행되는 MySQL 데이터베이스에 앱 데이터를 저장합니다. 앱 서버 인스턴스는 Java 앱을 호스팅하고 VM에서 실행됩니다. 앱 코드, 구성 파일 및 종속 항목 등의 앱 파일은 VM에 저장됩니다. 

### 컨테이너화된 아키텍처

다음 다이어그램은 Kubernetes 클러스터에서 실행되는 현대 컨테이너 아키텍처의 예를 보여줍니다. 

<p style="text-align: center;">
![아키텍처 다이어그램](images/solution30/modern_architecture.png)
</p>

1. 사용자가 요청을 Java 앱의 공용 엔드포인트에 전송합니다. 공용 엔드포인트는 클러스터의 앱 팟(Pod) 사이에서 수신 네트워크 트래픽에 대해 로드 밸런싱을 수행하는 Ingress 애플리케이션 로드 밸런서(ALB)에 의해 표시됩니다. ALB는 공개적으로 노출된 앱에 대한 인바운드 네트워크 트래픽을 허용하는 규칙의 콜렉션입니다. 
2. ALB가 요청을 클러스터의 사용 가능한 앱 팟(Pod) 중 하나에 전달합니다. 앱 팟(Pod)은 가상 또는 실제 머신일 수 있는 작업자 노드에서 실행됩니다. 
3. 앱 팟(Pod)이 데이터를 지속적 볼륨에 저장합니다. 지속적 볼륨을 사용하여 앱 인스턴스 또는 작업자 노드 사이에서 데이터를 공유할 수 있습니다. 
4. 앱 팟(Pod)이 데이터를 {{site.data.keyword.Bluemix_notm}} 데이터베이스 서비스에 저장합니다. Kubernetes 클러스터 내에서 자체 데이터베이스를 실행할 수 있지만 관리 DBaaS(Database-as-a-Service)를 사용하면 일반적으로 더 쉽게 기본 제공 백업 및 스케일링을 구성하고 제공할 수 있습니다. [IBM 클라우드 카탈로그](https://{DomainName}/catalog/?category=data)에서 다양한 유형의 데이터베이스를 찾을 수 있습니다. 

### VM, 컨테이너 및 Kubernetes

{{site.data.keyword.containershort_notm}}는 컨테이너화된 앱을 Kubernetes 클러스터에서 실행하는 기능을 제공하고 다음과 같은 도구 및 기능을 전달합니다. 

- 직관적인 사용자 경험 및 강력한 도구
- 보안 애플리케이션을 신속하게 전달할 수 있는 기본 제공 보안 및 격리
- IBM® Watson™의 코그너티브 기능을 포함하는 클라우드 서비스
- Stateless 애플리케이션과 Stateful 워크로드 모두에 대한 전용 클러스터 리소스를 관리하는 기능

#### 가상 머신 대 컨테이너

**VM**은 기본 하드웨어에서 실행되는 전통적인 앱입니다. 단일 앱은 일반적으로 단일 컴퓨팅 호스트의 전체 리소스를 사용하지 않습니다. 대부분의 조직은 리소스 낭비를 방지하기 위해 단일 컴퓨팅 호스트에서 여러 앱을 실행합니다. 동일한 앱의 여러 사본을 실행할 수 있지만 격리를 제공하기 위해 VM을 사용하여 동일한 하드웨어에서 여러 앱 인스턴스(VM)를 실행할 수 있습니다. 이 VM은 전체 운영 체제 스택을 가지고 있어 런타임 시와 디스크에서 중복으로 인해 상대적으로 크고 비효율적입니다. 

**컨테이너**는 환경 사이에서 앱을 원활하게 이동시킬 수 있도록 앱 및 앱의 모든 종속 항목을 패키지하는 표준 방법입니다. 가상 머신과는 달리 컨테이너는 운영 체제를 번들로 포함하지 않습니다. 앱 코드, 런타임, 시스템 도구, 라이브러리 및 설정만 컨테이너에 패키지됩니다. 컨테이너는 가상 머신보다 가볍고 휴대하기 쉬우며 효율적입니다. 

또한 컨테이너를 사용하면 호스트 OS를 공유할 수 있습니다. 이를 통해 격리는 계속 제공하면서 중복이 줄어듭니다. 컨테이너를 사용하면 시스템 라이브러리 및 2진 등의 불필요한 파일을 삭제하여 공간을 절약하고 공격 표면을 줄일 수 있습니다. [여기서](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html) 가상 머신 및 컨테이너에 대한 자세한 내용을 읽으십시오. 

#### Kubernetes 오케스트레이션

[Kubernetes](http://kubernetes.io/)는 작업자 노드의 클러스터에서 컨테이너화된 앱의 라이프사이클을 관리하는 컨테이너 오케스트레이터입니다. 앱을 실행하려면 많은 다른 리소스가 필요합니다(예: 보안 키 및 다른 클라우드 서비스에 연결하는 데 도움이 되는 시크릿, 네트워크 및 볼륨). Kubernetes를 사용하면 이 리소스를 앱에 추가할 수 있습니다. Kubernetes의 주요 패러다임은 선언적 모델입니다. 사용자는 원하는 상태를 제공하고 Kubernetes는 준수를 시도한 후 설명된 상태를 유지합니다. 

이 [단계별 맞춤 워크샵](https://github.com/IBM/kube101/blob/master/workshop/README.md)을 사용하면 Kubernetes에 대한 첫 번째 실습 경험을 얻을 수 있습니다. 또한 Kubernetes [개념](https://kubernetes.io/docs/concepts/) 문서 페이지를 확인하여 Kubernetes의 개념에 대해 자세히 살펴보십시오. 

### IBM이 사용자를 위해 제공하는 사항

Kubernetes 클러스터를 {{site.data.keyword.containerlong_notm}}와 함께 사용하면 다음과 같은 이점이 있습니다. 

- 클러스터를 배치할 수 있는 다중 데이터 센터
- Ingress 및 로드 밸런서 네트워킹 옵션에 대한 지원
- 동적 지속적 볼륨 지원
- 고가용성 IBM 관리 Kubernetes 마스터

## 클러스터 크기 조정
{: #sizing_clusters}

클러스터 아키텍처를 설계할 때 가용성, 신뢰성, 복잡도 및 복구와 비용 간 균형을 유지하길 원합니다. {{site.data.keyword.containerlong_notm}}에서 Kubernetes 클러스터는 앱의 요구에 따라 아키텍처 옵션을 제공합니다. 약간의 플랜이 있으면 과도한 아키텍팅 또는 과도한 지출 없이 클라우드 리소스를 최대한 활용할 수 있습니다. 더 많게 추정하거나 더 적게 추정하는 경우에도 추가적인 작업자 노드 또는 더 큰 작업자 노드를 사용하여 쉽게 클러스터를 확장하거나 축소할 수 있습니다. 

Kubernetes를 사용하여 클라우드에서 프로덕션 앱을 실행하려면 다음과 같은 항목을 고려하십시오. 

1. 특정 지리적 위치에서 트래픽을 예상하고 있습니까? 예상하고 있는 경우 최상의 성능을 위해 사용자와 물리적으로 가장 가까운 위치를 선택하십시오. 
2. 고가용성을 위해 몇 개의 클러스터 복제본을 원합니까? 3개의 클러스터(개발용으로 하나, 테스트용으로 하나, 프로덕션용으로 하나)로 시작하는 것이 좋습니다. [사용자, 팀, 애플리케이션 구성을 위한 우수 사례](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments) 솔루션 안내서에서 다중 환경 작성에 대해 확인하십시오. 
3. 작업자 노드를 위해 필요한 [하드웨어](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes)는 무엇입니까? 가상 머신입니까 아니면 베어메탈입니까? 
4. 몇 개의 작업자 노드가 필요합니까? 이는 앱 스케일에 따라 다르며 노드가 많을수록 앱의 복원성이 높아집니다. 
5. 고가용성을 위해 몇 개의 복제본을 보유해야 합니까? 앱의 가용성을 높이고 위치 장애로 인해 앱이 작동 중단되지 않도록 보호하기 위해 복제본 클러스터를 여러 위치에 배치하십시오. 
6. 앱을 시작하기 위해 필요한 최소 리소스 세트는 무엇입니까? 실행하는 데 필요한 CPU 및 메모리의 양에 대해 앱을 테스트할 수 있습니다. 그러면 작업자 노드가 앱을 배치하고 시작할 수 있는 충분한 리소스를 가지게 됩니다. 그런 다음 팟(Pod) 스펙의 일부로 리소스 할당량을 설정해야 합니다. 이 설정은 요청을 지원할 수 있는 충분한 용량을 가진 작업자 노드를 선택(또는 스케줄)하기 위해 Kubernetes가 사용하는 것입니다. 작업자 노드에서 실행될 팟(Pod) 수와 해당 팟(Pod)에 대한 리소스 요구사항을 추정하십시오. 최소한 작업자 노드는 앱에 대해 하나의 팟(Pod)을 지원할 수 있을 정도로 커야 합니다. 
7. 작업자 노드 수는 언제 늘려야 합니까? 클러스터 사용량을 모니터하여 필요할 때 노드를 늘릴 수 있습니다. 이 튜토리얼을 확인하여 [로그를 분석하고 Kubernetes 애플리케이션의 상태를 모니터](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana)하는 방법에 대해 이해하십시오. 
8. 신뢰할 수 있는 중복 스토리지가 필요합니까? 필요한 경우 NFS 스토리지에 대한 지속적 볼륨 청구를 작성하거나 IBM Cloud 데이터베이스 서비스를 팟(Pod)에 바인드하십시오. 

위 내용을 더 구체적으로 설명하기 위해 클라우드에서 프로덕션 웹 애플리케이션을 실행하려고 하고 중간 수준에서 높은 수준까지의 트래픽 로드를 예상한다고 가정해 봅니다. 필요한 리소스에 대해 알아봅니다. 

1. 3개의 클러스터(개발용으로 하나, 테스트용으로 하나, 프로덕션용으로 하나)를 설정하십시오. 
2. 개발 및 테스트 클러스터는 최소 RAM 및 CPU 옵션(예: 각 클러스터에 대해 2개의 CPU, 4GB의 RAM 및 하나의 작업자 노드)으로 시작할 수 있습니다. 
3. 프로덕션 클러스터의 경우 성능, 고가용성 및 복원성을 위해 더 많은 리소스를 원할 수 있습니다. 전용 또는 베어메탈 옵션을 선택할 수 있고 최소 4개의 CPU, 16GB의 RAM 및 2개의 작업자 노드를 가질 수 있습니다. 

## 사용할 데이터베이스 옵션 결정
{: #database_options}

Kubernetes를 사용하는 경우 데이터베이스 처리를 위한 두 가지 옵션이 있습니다. 

1. Kubernetes 클러스터 내에서 데이터베이스를 실행할 수 있습니다. 이를 위해서는 데이터베이스를 실행할 마이크로서비스를 작성해야 합니다. MySQL 데이터베이스 예제를 사용하는 경우에는 다음을 수행해야 합니다. 
   - MySQL Dockerfile을 작성하십시오. 여기서 예제 [MySQL Dockerfile](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile)을 참조하십시오. 
   - 시크릿을 사용하여 데이터베이스 인증 정보를 저장해야 합니다. [여기서](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret) 예제를 확인하십시오. 
   - Kubernetes에 배치된 데이터베이스의 구성을 가진 deployment.yaml 파일이 필요합니다. [여기서](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml) 예제를 확인하십시오. 
2. 두 번째 옵션은 관리 DBaaS(Database-as-a-Service) 옵션을 사용하는 것입니다. 이 옵션은 일반적으로 구성하기가 더 쉽고 기본 제공 백업 및 스케일링을 제공합니다. [IBM Cloud 카탈로그](https://{DomainName}/catalog/?category=data)에서 다양한 유형의 데이터베이스를 찾을 수 있습니다. 이 옵션을 사용하려면 다음을 수행해야 합니다. 
   - [IBM 클라우드 카탈로그](https://{DomainName}/catalog/?category=data)에서 관리 DBaaS(Database-as-a-Service)를 작성하십시오. 
   - 시크릿 내에 데이터베이스 인증 정보를 저장하십시오. "Kubernetes 시크릿에 인증 정보 저장" 절에서 시크릿에 대해 자세히 살펴봅니다. 
   - 애플리케이션에서 DBaaS(Database-as-a-Service)를 사용하십시오. 

## 애플리케이션 파일을 저장할 위치 결정
{: #decide_where_to_store_data}

{{site.data.keyword.containershort_notm}}는 여러 팟(Pod)에서 데이터를 저장하고 공유하는 몇몇 옵션을 제공합니다. 일부 스토리지 옵션은 재해 상황에서 동일한 레벨의 지속성 및 가용성을 제공하지 않습니다.
{: shortdesc}

### 비지속적 데이터 스토리지

컨테이너 및 팟(Pod)은 설계상 수명이 짧으며 예상치 않게 실패할 수 있습니다. 컨테이너의 로컬 파일 시스템에 데이터를 저장할 수 있습니다. 컨테이너 내부의 데이터는 다른 컨테이너 또는 팟(Pod)과 공유할 수 없으며 컨테이너가 손상되거나 제거되면 유실됩니다. 

### 앱을 위해 지속적 데이터 스토리지를 작성하는 방법 학습

기본 Kubernetes 지속적 볼륨을 사용하여 [NFS 파일 스토리지](https://www.ibm.com/cloud/file-storage/details) 또는 [블록 스토리지](https://www.ibm.com/cloud/block-storage)에서 앱 데이터 및 컨테이너 데이터를 지속시킬 수 있습니다.
{: shortdesc}

NFS 파일 스토리지 또는 블록 스토리지를 프로비저닝하려면 지속적 볼륨 청구(PVC)를 작성하여 팟(Pod)에 대한 스토리지를 요청해야 합니다. PVC에서 스토리지의 읽기 및 쓰기 권한, 스토리지 유형, 스토리지 크기(GB), IOPS 및 데이터 보존 정책을 정의하는 사전 정의된 스토리지 클래스 중에서 선택할 수 있습니다. PVC는 {{site.data.keyword.Bluemix_notm}}에서 실제 스토리지 디바이스를 나타내는 지속적 볼륨(PV)을 동적으로 프로비저닝합니다. PVC를 팟(Pod)에 마운트하여 PV에서 읽고 PV에 쓸 수 있습니다. PV에 저장되는 데이터는 컨테이너가 손상되거나 팟(Pod)이 다시 스케줄되는 경우에도 사용할 수 있습니다. PV를 지원하는 NFS 파일 스토리지 및 블록 스토리지는 데이터에 대한 고가용성을 제공하기 위해 IBM에 의해 클러스터됩니다. 

PVC 작성 방법을 학습하려면 [{{site.data.keyword.containershort_notm}} 스토리지 문서](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage)에 설명된 단계를 수행하십시오. 

### 기존 데이터를 지속적 스토리지로 이동하는 방법 학습

로컬 시스템의 데이터를 지속적 스토리지에 복사하려면 PVC를 팟(Pod)에 마운트해야 합니다. 그런 다음 로컬 시스템의 데이터를 팟(Pod)의 지속적 볼륨에 복사할 수 있습니다.
{: shortdesc}

1. 날짜를 복사하려면 먼저 다음과 비슷한 구성을 작성해야 합니다. 

   ```bash
   kind: Pod
   apiVersion: v1
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
          claimName: mypvc
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/mnt/data"
             name: task-pv-storage
   ```
   {: codeblock}

2. 그런 다음 로컬 시스템의 데이터를 팟(Pod)에 복사하기 위해 다음과 같은 명령을 사용합니다. 
   ```
    kubectl cp <local_filepath>/<filename> <namespace>/<pod>:<pod_filepath>
   ```
   {: pre}

3. 클러스터의 팟(Pod)에서 로컬 시스템으로 데이터를 복사하십시오. 
   ```
   kubectl cp <namespace>/<pod>:<pod_filepath>/<filename> <local_filepath>/<filename>
   ```
   {: pre}


### 지속적 스토리지를 위한 백업 설정

파일 공유 및 블록 스토리지는 클러스터와 동일한 위치에 프로비저닝됩니다. 고가용성을 제공하기 위해 스토리지 자체는 IBM에 의해 클러스터된 서버에서 호스팅됩니다. 하지만 파일 공유 및 블록 스토리지는 자동으로 백업되지 않으므로 전체 위치가 실패하는 경우 액세스할 수 없습니다. 데이터가 유실되거나 손상되지 않도록 보호하기 위해 필요한 경우 데이터를 복원하는 데 사용할 수 있는 주기적인 백업을 설정할 수 있습니다. 

자세한 정보는 NFS 파일 스토리지 및 블록 스토리지에 대한 [백업 및 복원](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) 옵션을 참조하십시오. 

## 코드 준비
{: #prepare_code}

### 12개 요인 원칙 적용

[12개 요인 앱](https://12factor.net/)은 클라우드 기본 앱을 빌드하는 데 필요한 방법론입니다. 앱을 컨테이너화하고 이 앱을 클라우드에 이동시키고 앱을 Kubernetes와 오케스트레이션하길 원하는 경우 이 원칙 중 일부를 이해하고 적용하는 것이 중요합니다. 이 원칙 중 일부는 {{site.data.keyword.Bluemix_notm}}에서 필수입니다.
{: shortdesc}

필요한 주요 원칙 중 일부는 다음과 같습니다. 

- **코드 베이스** - 버전 제어 시스템(예: GIT 저장소) 내에서 모든 소스 코드 및 구성 파일이 추적되며 이는 배치를 위해 DevOps 파이프라인을 사용하는 경우 필요합니다. 
- **빌드, 릴리스, 실행** - 12개 요인 앱에서는 빌드, 릴리스 및 실행 단계 사이에 엄격한 분리를 사용합니다. 이는 통합 DevOps 딜리버리 파이프라인을 사용하여 자동화되어 앱을 클러스터에 배치하기 전에 앱을 빌드하고 테스트할 수 있습니다. [Kubernetes에 대한 지속적 배치 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)을 확인하여 지속적 통합 및 딜리버리 파이프라인을 설정하는 방법을 학습하십시오. 여기서는 소스 제어, 빌드, 테스트 및 배치 단계의 설정을 설명하고 보안 스캐너, 알림 및 분석 등의 통합을 추가하는 방법을 보여줍니다. 
- **구성** - 모든 구성 정보는 환경 변수에 저장됩니다. 앱 코드 내에 서비스 인증 정보가 하드코딩되지 않습니다. 인증 정보를 저장하기 위해 Kubernetes 시크릿을 사용할 수 있습니다. 인증 정보에 대한 자세한 내용이 아래에 설명되어 있습니다. 

### Kubernetes 시크릿에 인증 정보 저장
{: secrets}

인증 정보를 앱 코드에 저장하는 것은 바람직하지 않습니다. 대신 Kubernetes는 비밀번호, OAuth 토큰 또는 ssh 키 등의 민감한 정보를 보유하는 **["시크릿"](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)**이라는 것을 제공합니다. Kubernetes 시크릿은 기본적으로 암호화되어 이 데이터 축약어를 `pod` 정의 또는 Docker 이미지에 저장하는 것보다 시크릿을 민감한 데이터를 저장하는 더 안전하고 유연한 옵션으로 만듭니다. 

Kubernetes에서 시크릿을 사용하는 한 가지 방법은 다음과 같습니다. 

1. 파일을 작성하고 파일 내에 서비스 인증 정보를 저장하십시오. 
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api_key>
   }
   ```
   {: codeblock}

2. 그런 다음 아래 명령을 실행하여 Kubernetes 시크릿을 작성하고 아래 명령을 실행한 후 `kubectl get secrets`를 사용하여 시크릿이 작성되었는지 확인하십시오. 

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## 앱 컨테이너화
{: #build_docker_images}

앱을 컨테이너화하려면 Docker 이미지를 작성해야 합니다.
{: shortdesc}

이미지를 빌드하는 데 필요한 명령어 및 명령이 포함된 파일인 [Dockerfile](https://docs.docker.com/engine/reference/builder/#usage)에서 이미지가 작성됩니다. Dockerfile은 앱, 앱의 구성 및 해당 종속 항목 등 별도로 저장되는 명령어의 빌드 아티팩트를 참조합니다. 

기존 앱에 대해 자체 Dockerfile을 빌드하기 위해 다음과 같은 명령을 사용할 수 있습니다. 

- FROM - 컨테이너 런타임을 정의할 상위 이미지를 선택합니다. 
- ADD/COPY - 디렉토리의 컨텐츠를 컨테이너에 복사합니다. 
- WORKDIR - 컨테이너 내에서 작업 중인 디렉토리를 설정합니다. 
- RUN - 런타임 중에 앱에 필요한 소프트웨어 패키지를 설치합니다. 
- EXPOSE - 컨테이너 외부에서 포트를 사용할 수 있게 합니다. 
- ENV NAME - 환경 변수를 정의합니다. 
- CMD - 컨테이너가 실행될 때 실행되는 명령을 정의합니다. 

이미지는 일반적으로 공용(공용 레지스트리)으로 액세스할 수 있거나 소규모 사용자 그룹(개인용 레지스트리)의 액세스를 제한하여 설정될 수 있는 레지스트리에 저장됩니다. 공용 레지스트리(예: Docker Hub)를 사용하여 Docker 및 Kubernetes를 시작하여 클러스터에서 첫 번째 컨테이너화된 앱을 작성할 수 있습니다. 하지만 엔터프라이즈 앱의 경우 권한 없는 사용자가 이미지를 사용하고 변경하지 못하도록 보호하기 위해 {{site.data.keyword.registrylong_notm}}에 제공된 것과 같은 개인용 레지스트리를 사용하십시오. 

앱을 컨테이너화하여 {{site.data.keyword.registrylong_notm}}에 저장하려면 다음을 수행하십시오. 

1. Dockerfile을 작성해야 합니다. Dockerfile의 예제는 다음과 같습니다. 
   ```
   # Build JPetStore war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # Use WebSphere Liberty base image from the Docker Store
   FROM websphere-liberty:latest

   # Copy war from build stage and server.xml into image
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. Dockerfile이 작성되면 다음으로 컨테이너 이미지를 빌드하여 {{site.data.keyword.registrylong_notm}}에 푸시해야 합니다. 다음과 같은 명령을 사용하여 컨테이너를 빌드할 수 있습니다. 
   ```
   ibmcloud cr build -t <image_name> <directory_of_Dockerfile>
   ```
   {: pre}

## Kubernetes 클러스터에 앱 배치
{: #deploy_to_kubernetes}

컨테이너 이미지를 빌드하여 클라우드에 배치한 후에는 Kubernetes 클러스터에 배치해야 합니다. 이를 위해 deployment.yaml 파일을 작성해야 합니다.
{: shortdesc}

### Kubernetes 배치 yaml 파일을 작성하는 방법 학습

Kubernetes deployment.yaml 파일을 작성하려면 다음과 같은 작업을 수행해야 합니다. 

1. deployment.yaml 파일을 작성하십시오. [배치 YAML](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml) 파일의 예가 여기에 있습니다. 

2. deployment.yaml 파일에서 컨테이너에 대한 [리소스 할당량](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)을 정의하여 각 컨테이너를 제대로 시작하기 위해 필요한 CPU 및 메모리의 양을 지정할 수 있습니다. 컨테이너에 리소스 할당량이 지정된 경우 Kubernetes 스케줄러는 작업자 노드에서 팟(Pod)을 배치할 위치를 결정하는 데 도움이 될 수 있습니다. 

3. 다음으로 아래 명령을 사용하여 배치 및 서비스를 작성하고 볼 수 있습니다. 

   ```
   kubectl create -f <filepath/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## 요약
{: #summary}

이 튜토리얼에서는 다음과 같은 내용을 학습했습니다. 

- VM, 컨테이너 및 Kubernetes 간 차이점
- 다양한 환경 유형(개발, 테스트 및 프로덕션)에 대해 클러스터를 정의하는 방법
- 데이터 스토리지 처리 방법 및 지속적 데이터 스토리지의 중요성
- 앱에 12개 요인 원칙 적용 및 Kubernetes에서 인증 정보를 위한 시크릿 사용
- Docker 이미지를 작성하여 {{site.data.keyword.registrylong_notm}}에 푸시
- Kubernetes 배치 파일 작성 및 Kubernetes에 Docker 이미지 배치

## 학습한 모든 내용을 실습하고 클러스터에서 JPetStore 앱 실행
{: #runthejpetstore}

학습한 모든 내용을 실습하려면 [데모](https://github.com/ibm-cloud/ModernizeDemo/)를 수행하여 클러스터에서 **JPetStore** 앱을 실행하고 학습한 개념을 적용하십시오. JPetStore 앱은 별도의 마이크로서비스로 실행되는 IBM Watson 서비스를 통해 Kubernetes에서 앱을 확장할 수 있는 몇몇 확장된 기능을 가지고 있습니다. 

## 관련 컨텐츠
{: #related}

- Kubernetes 및 {{site.data.keyword.containershort_notm}} [시작하기](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/)
- [GitHub](https://github.com/IBM/container-service-getting-started-wt)의 {{site.data.keyword.containershort_notm}} 연구소
- Kubernetes 기본 [문서](http://kubernetes.io/)
- {{site.data.keyword.containershort_notm}}의 [지속적 스토리지](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning)
- 사용자, 팀 및 앱을 구성하는 데 필요한 [우수 사례 솔루션 안내서](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
- [LogDNA 및 Sysdig를 사용하여 로그 분석 및 애플리케이션 상태 모니터](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
- Kubernetes에서 실행되는 컨테이너화된 앱을 위해 [지속적 통합 및 딜리버리 파이프라인](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) 설정
- [여러 위치에](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp) 프로덕션 클러스터 배치
- 고가용성을 위해 [여러 위치에서 여러 클러스터](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations) 사용
