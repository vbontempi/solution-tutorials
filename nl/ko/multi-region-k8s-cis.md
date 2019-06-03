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

# Cloud Internet Services를 사용하여 강한 복원성과 보안을 갖춘 다중 지역 Kubernetes 클러스터
{: #multi-region-k8s-cis}

복원성을 고려하여 애플리케이션이 디자인된 경우에는 사용자가 작동중단시간을 경험할 가능성이 적어집니다. {{site.data.keyword.containershort_notm}}를 사용하여 솔루션을 구현하면 로드 밸런싱 및 격리와 같은 기본 제공 기능의 이점을 누리며 호스트, 네트워크 또는 앱의 잠재적 장애에 대한 복원성을 증가시킬 수 있습니다. 여러 클러스터를 작성하면 한 클러스터에서 가동 중단이 발생하는 경우에도 사용자가 다른 클러스터에 배치된 앱에 여전히 액세스할 수 있습니다. 여러 위치의 여러 클러스터를 사용하는 경우에는 가장 가까운 클러스터에 액세스하여 네트워크 대기 시간을 줄일 수도 있습니다. 추가 복원성을 위해서는 하나의 위치에 있는 여러 구역에 노드가 배치됨을 의미하는 다중 구역 클러스터를 선택할 수도 있습니다. 

이 튜토리얼에서는 이 시나리오를 지원하기 위해 도메인 이름 시스템(DNS), 글로벌 로드 밸런싱(GLB), WAF(Web Application Firewall), 인터넷 애플리케이션에 대한 분산 서비스 거부(DDoS) 보호를 구성하고 관리하는 통합 플랫폼인 Cloud Internet Services(CIS)를 Kubernetes 클러스터와 통합하는 방법, 그리고 여러 위치에서 안전하며 복원성 있는 솔루션을 제공하는 방법에 대해 자세히 설명합니다. 

## 목표
{: #objectives}

* 다양한 위치의 여러 Kubernetes 클러스터에 애플리케이션을 배치합니다. 
* 글로벌 로드 밸런서를 사용하여 여러 클러스터에 트래픽을 분배합니다. 
* 가장 가까운 클러스터로 사용자를 라우팅합니다. 
* 보안 위협으로부터 애플리케이션을 보호합니다. 
* 캐싱을 사용하여 애플리케이션 성능을 향상시킵니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">
  ![아키텍처](images/solution32-multi-region-k8s-cis/Architecture.png)
</p>

1. 개발자가 애플리케이션의 Docker 이미지를 빌드합니다. 
2. 이 이미지가 댈러스 및 런던의 {{site.data.keyword.registryshort_notm}}에 푸시됩니다. 
3. 애플리케이션이 해당 두 위치의 Kubernetes 클러스터에 배치됩니다. 
4. 일반 사용자가 애플리케이션에 액세스합니다. 
5. Cloud Internet Services는 애플리케이션에 대한 요청을 인터셉트하고 로드를 클러스터 전체에 분배하도록 구성됩니다. 일반적인 위협으로부터 애플리케이션을 보호하기 위해 DDoS 보호 및 Web Application Firewall 또한 사용으로 설정됩니다. 선택적으로 이미지, CSS 파일과 같은 자산이 캐시됩니다. 

## 시작하기 전에
{: #prereqs}

* Cloud Internet Services를 사용하려면 사용자 정의 도메인이 Cloud Internet Services 이름 서버를 가리키도록 DNS를 구성하기 위해 사용자 정의 도메인을 소유해야 합니다. 
* [Git을 설치하십시오](https://git-scm.com/). 
* [{{site.data.keyword.Bluemix_notm}} CLI를 설치하십시오](/docs/cli?topic=cloud-cli-install-ibmcloud-cli). 
* [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - 스크립트를 실행하여 Docker, kubectl, Helm, IBM Cloud CLI 및 필수 플러그인을 설치하십시오. 
* [{{site.data.keyword.registrylong_notm}} CLI 및 레지스트리 네임스페이스를 설정하십시오](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace). 
* [Kubernetes의 기본 사항을 이해하십시오](https://kubernetes.io/docs/tutorials/kubernetes-basics/). 

## 하나의 위치에 애플리케이션 배치

이 튜토리얼에서는 Kubernetes 애플리케이션을 여러 위치의 클러스터에 배치합니다. 먼저 댈러스에서 배치를 시작한 후 런던에 대해 이러한 단계를 반복합니다. 

### Kubernetes 클러스터 작성
{: #create_cluster}

클러스터를 작성하려면 다음 작업을 수행하십시오.  
1. [{{site.data.keyword.cloud_notm}} 카탈로그](https://{DomainName}/containers-kubernetes/catalog/cluster/create)에서 **{{site.data.keyword.containershort_notm}}**를 선탁하십시오. 
1. **위치**를 **댈러스**로 설정하십시오. 
1. **표준** 클러스터를 선택하십시오. 
1. 하나 이상의 구역을 **위치**로 선택하십시오. 다중 구역 클러스터를 작성하면 애플리케이션 복원성이 향상됩니다. 앱이 여러 구역에 배치되어 있으면 사용자가 작동중단시간을 경험할 가능성이 훨씬 적어집니다. 다중 구역 클러스터에 대한 자세한 내용은 [여기](https://{DomainName}/docs/containers?topic=containers-plan_clusters#ha_clusters)에 있습니다. 
1. **시스템 유형**을 사용 가능한 가장 작은 유형으로 설정하십시오. 이 튜토리얼을 수행하는 데는 **2개 CPU** 및 **4GB RAM**이면 충분합니다. 
1. **2**개의 작업자 노드를 사용하십시오. 
1. **클러스터 이름**을 **my-us-cluster**로 설정하십시오. 이 튜토리얼에서는 일관되게 *`my-<location>-cluster`* 이름 패턴을 사용하십시오. 

클러스터가 준비되는 동안 애플리케이션을 준비하십시오. 

### {{site.data.keyword.registryshort_notm}}에 네임스페이스 작성

1. {{site.data.keyword.Bluemix_notm}} CLI의 대상을 댈러스로 설정하십시오. 
   ```bash
   ibmcloud target -r us-south
   ```
   {: pre}
2. 애플리케이션을 위한 네임스페이스를 작성하십시오. 
   ```bash
   ibmcloud cr namespace-add <your_namespace>
   ```
   {: pre}

기존 네임스페이스가 해당 위치에 있는 경우에는 이를 재사용할 수도 있습니다. `ibmcloud cr namespaces`로 기존 네임스페이스를 나열할 수 있습니다.
{: tip}

### 애플리케이션 빌드

이 단계에서는 애플리케이션을 Docker 이미지로 빌드합니다. 두 번째 클러스터를 구성하는 경우에는 이 단계를 건너뛸 수 있습니다. 이는 단순한 Hello world 앱입니다. 

1. [Hello world 앱](https://github.com/IBM/container-service-getting-started-wt){:new_windows}의 소스 코드를 사용자 홈 디렉토리로 복제하십시오. 이 저장소는 Lab으로 시작하는 각 폴더에 유사한 앱의 서로 다른 버전을 포함하고 있습니다. 
   ```bash
   git clone https://github.com/IBM/container-service-getting-started-wt.git
   ```
   {: pre}
1. `Lab 1` 디렉토리로 이동하십시오. 
   ```bash
   cd 'container-service-getting-started-wt/Lab 1'
   ```
   {: pre}
1. `Lab 1` 디렉토리의 앱 파일을 포함하는 Docker 이미지를 빌드하십시오. 
   ```bash
   docker build --tag multi-region-hello-world:1 .
   ```
   {: pre}

### 특정 위치 레지스트리에 푸시될 이미지 준비

대상 레지스트리로 이미지에 태그를 지정하십시오. 

   ```bash
   docker tag multi-region-hello-world:1 us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### 이미지를 특정 위치 레지스트리에 푸시

1. 로컬 Docker 엔진이 댈러스의 레지스트리로 푸시할 수 있는지 확인하십시오. 
   ```bash
   ibmcloud cr login
   ```
   {: pre}
2. 이미지를 푸시하십시오. 
   ```bash
   docker push us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### 애플리케이션을 Kubernetes 클러스터에 배치

이 단계에 도달하면 클러스터가 준비되어 있습니다. 해당 상태는 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/clusters) 콘솔에서 확인할 수 있습니다. 

1. 클러스터의 구성을 검색하십시오. 
   ```bash
   ibmcloud cs cluster-config <us-cluster-name>
   ```
   {: pre}
1. 출력을 복사하고 붙여넣어 KUBECONFIG 환경 변수를 설정하십시오. 이 변수는 `kubectl`에 의해 사용됩니다. 
1. 복제본을 둘로 설정하여 클러스터의 애플리케이션을 실행하십시오. 
   ```bash
   kubectl run hello-world-deployment --image=us.icr.io/<your_US-South_namespace>/hello-world:1 --replicas=2
   ```
   {: pre}
   출력 예: `deployment "hello-world-deployment" created`.
1. 클러스터에서 애플리케이션에 액세스할 수 있도록 설정하십시오. 
   ```bash
   kubectl expose deployment/hello-world-deployment --type=ClusterIP --port=80 --name=hello-world-service --target-port=8080
   ```
   {: pre}
   이는 `service "hello-world-service" exposed`와 같은 메시지를 리턴합니다. 

### 클러스터에 지정된 도메인 이름 및 IP 주소 가져오기
{: #CSALB_IP_subdomain}

Kubernetes 클러스터가 작성되면 Ingress 하위 도메인(예: *my-us-cluster.us-south.containers.appdomain.cloud*) 및 공용 애플리케이션 로드 밸런서 IP 주소가 지정됩니다. 

1. 클러스터의 Ingress 하위 도메인을 검색하십시오. 
   ```bash
   ibmcloud cs cluster-get <us-cluster-name>
   ```
   {: pre}
   `Ingress Subdomain` 값을 찾으십시오.
1. 나중 단계를 위해 이 정보를 기록하십시오. 

이 튜토리얼에서는 Ingress 하위 도메인을 사용하여 글로벌 로드 밸런서를 구성합니다. 하위 도메인을 공용 애플리케이션 로드 밸런서 IP 주소(`ibmcloud cs albs -cluster <us-cluster-name>`)로 바꿀 수도 있습니다. 두 방법은 모두 지원됩니다.
{: tip}

## 다른 위치 구성

다음 항목을 대체하여 이전 단계를 런던에서 반복하십시오. 
* 위치 이름 **댈러스**를 **런던**으로
* 위치 별명 **us-south**를 **eu-gb**로
* 레지스트리 *us.icr.io*를 **uk.icr.io**로
* 클러스터 이름 *my-us-cluster*를 **my-uk-cluster**로

## 다중 위치 로드 밸런싱 구성

애플리케이션이 이제 두 클러스터에서 실행되지만 사용자가 하나의 시작점에서 두 클러스터에 투명하게 액세스하기 위해서는 한 컴포넌트가 누락되어 있습니다. 

이 절에서는 Cloud Internet Services(CIS)를 구성하여 두 클러스터 간에 로드를 분배합니다. CIS는 Cloud 애플리케이션을 보안하면서 복원성 및 성능을 보장하기 위해 글로벌 로드 밸런서(GLB), 캐싱, WAF(Web Application Firewall) 및 페이지 규칙을 제공하는 일체형 서비스입니다. 

글로벌 로드 밸런서를 구성하려면 다음 항목이 필요합니다. 
* 사용자 정의 도메인이 CIS 이름 서버를 가리켜야 함
* Kubernetes 클러스터의 IP 주소 또는 하위 도메인을 검색해야 함
* 애플리케이션의 가용성을 유효성 검증하기 위한 상태 확인을 구성해야 함
* 클러스터를 가리키는 오리진 풀을 정의해야 함

### Cloud Internet Services를 사용하여 사용자 정의 도메인 등록
{: #create_cis_instance}

첫 번째 단계는 CIS의 인스턴스를 작성하고 사용자 정의 도메인이 CIS 이름 서버를 가리키도록 하는 것입니다. 

1. 소유하는 도메인이 없는 경우에는 [godaddy.com](http://godaddy.com)와 같은 등록 담당자로부터 도메인을 구입할 수 있습니다. 
2. {{site.data.keyword.Bluemix_notm}} 카탈로그의 [Internet Services](https://{DomainName}/catalog/services/internet-services)로 이동하십시오. 
3. 서비스 이름을 설정하고 **작성**을 클릭하여 서비스의 인스턴스를 작성하십시오. 
4. 서비스 인스턴스가 프로비저닝되면 도메인 이름을 설정하고 **도메인 추가**를 클릭하십시오. 
5. 이름 서버가 지정되면 나열된 이름 서버를 사용하도록 등록 담당자 또는 도메인 이름 제공자를 구성하십시오. 
6. 등록 담당자 또는 DNS 제공자를 구성한 후 이것이 적용되기까지는 최대 24시간이 소요될 수 있습니다. 

   개요 페이지의 도메인 상태가 *보류 중*에서 *활성*으로 변경되면 `dig <your_domain_name> ns` 명령을 사용하여 새 이름 서버가 적용되었는지 확인할 수 있습니다.
   {:tip}

### 글로벌 로드 밸런서에 대한 상태 확인 구성

상태 확인은 정상 풀에 트래픽을 라우팅할 수 있도록 풀의 가용성을 파악하는 데 도움을 줍니다. 이러한 확인은 주기적으로 HTTP/HTTPS 요청을 전송하고 응답을 모니터합니다. 

1. Cloud Internet Services 대시보드에서 **신뢰성** > **글로벌 로드 밸런서**로 이동한 후 페이지 맨 아래에서 **상태 확인 작성**을 클릭하십시오. 
1. **경로**를 **/**로 설정하십시오. 
1. **모니터 유형**을 **HTTP**로 설정하십시오. 
1. **1개 인스턴스 프로비저닝**을 클릭하십시오. 

   자신의 고유 애플리케이션을 빌드할 때는 애플리케이션 상태를 보고하는 */heathz*와 같은 전용 상태 엔드포인트를 정의할 수 있습니다.
   {:tip}

### 오리진 풀 정의

풀은 GLB에 연결된 경우 트래픽이 지능적으로 라우팅되는 오리진 서버의 그룹입니다. 영국과 미국의 클러스터를 사용하는 경우에는 위치 기반 풀을 정의하고 사용자 요청의 지리적 위치에 따라 가장 가까운 클러스터로 사용자를 경로 재지정하도록 CIS를 구성할 수 있습니다. 

#### 런던에 있는 클러스터에 대한 하나의 풀 작성
1. **풀 작성**을 클릭하십시오. 
2. **이름**을 **UK**로 설정하십시오. 
3. **상태 확인**을 이전 절에서 작성한 것으로 설정하십시오. 
4. **상태 확인 지역**을 **서부 유럽**으로 설정하십시오. 
5. **오리진 이름**을 **uk-cluster**로 설정하십시오. 
6. **오리진 주소**를 런던에 있는 클러스터의 Ingress 하위 도메인(예: *my_uk_cluster.eu-gb.containers.appdomain.cloud*)으로 설정하십시오. 
7. **1개 인스턴스 프로비저닝**을 클릭하십시오. 

#### 댈러스에 있는 클러스터에 대한 하나의 풀 작성
1. **풀 작성**을 클릭하십시오. 
2. **이름**을 **US**로 설정하십시오. 
3. **상태 확인**을 이전 절에서 작성한 것으로 설정하십시오. 
4. **상태 확인 지역**을 **서부 북미**로 설정하십시오. 
5. **오리진 이름**을 **us-cluster**로 설정하십시오. 
6. **오리진 주소**를 댈러스에 있는 클러스터의 Ingress 하위 도메인(예: *my_us_cluster.us-south.containers.appdomain.cloud*)으로 설정하십시오. 
7. **1개 인스턴스 프로비저닝**을 클릭하십시오. 

#### 두 클러스터 모두에 대한 하나의 풀 작성
1. **풀 작성**을 클릭하십시오. 
1. **이름**을 **All**로 설정하십시오. 
1. **상태 확인**을 이전 절에서 작성한 것으로 설정하십시오. 
1. **상태 확인 지역**을 **동부 북미**로 설정하십시오. 
1. 두 오리진을 추가하십시오. 
   1. **오리진 이름**이 **us-cluster**로 설정되었으며 **오리진 주소**가 댈러스에 있는 클러스터의 Ingress 하위 도메인으로 설정된 오리진
   2. **오리진 이름**이 **uk-cluster**로 설정되었으며 **오리진 주소**가 런던에 있는 클러스터의 Ingress 하위 도메인으로 설정된 오리진
2. **1개 인스턴스 프로비저닝**을 클릭하십시오. 

### 글로벌 로드 밸랜서 작성

오리진 풀이 정의되었으면 로드 밸런서의 구성을 완료할 수 있습니다. 

1. **로드 밸런서 작성**을 클릭하십시오. 
1. **밸런서 호스트 이름**에 글로벌 로드 밸런서의 이름을 입력하십시오. 이 이름은 또한 위치에 상관없이 유니버셜 애플리케이션 URL(`http://<glb_name>.<your_domain_name>`)의 일부가 됩니다. 
1. **기본 오리진 풀**에서 **풀 추가**를 클릭하고 **All**이라는 풀을 추가하십시오. 
1. **지리적 라우트 구성(선택사항)** 섹션을 펼치십시오. 
   1. **라우트 추가**를 클릭하고 **서부 유럽**을 선택한 후 **추가**를 클릭하십시오. 
   1. **풀 추가**를 클릭하여 **UK** 풀을 선택하십시오. 
   1. 다음 표에 표시되어 있는 바와 같이 추가 라우트를 구성하십시오. 
   1. **1개 인스턴스 프로비저닝**을 클릭하십시오. 

| 지역                 |  오리진 풀  |
| :---------------:    | :---------: |
|서부 유럽             |     UK      |
|동부 유럽             |     UK      |
|북동 아시아           |     UK      |
|남동 아시아           |     UK      |
|서부 북미             |     US      |
|동부 북미             |     US      |

이 구성에서 유럽 및 아시아의 사용자는 런던의 클러스터로 경로 재지정되고 미국의 사용자는 댈러스 클러스터로 경로 재지정됩니다. 요청이 정의된 라우트와 일치하지 않는 경우에는 **기본 오리진 풀**로 경로 재지정됩니다. 

### 각 위치의 Kubernetes 클러스터를 위한 Ingress 리소스 작성

이제 글로벌 로드 밸런서가 요청을 서비스할 준비가 되었습니다. 모든 상태 확인이 정상입니다. 하지만 글로벌 로드 밸런서로부터 수신되는 요청에 올바르게 응답하기 위해서는 Kubernetes 클러스터에서 한 가지 마지막 구성 단계를 수행해야 하며, 이는 GLB 도메인으로부터의 요청을 처리하는 데 필요한 Ingress 리소스를 정의하는 것입니다. 

1. 이름이 **glb-ingress.yaml**인 Ingress 리소스 파일을 작성하십시오. 
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: glb-ingress
   spec:
    rules:
      - host: <glb_name>.<your_domain_name>
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-world-service
              servicePort: 80
   ```
    {: pre}
    `<glb_name>.<your_domain_name>`을 이전 절에서 정의한 URL로 대체하십시오.
1. 각 위치 클러스터의 KUBECONFIG 변수를 설정한 후 이 리소스를 런던 및 댈러스 클러스터에 모두 배치하십시오. 
   ```bash
   kubectl create -f glb-ingress.yaml
   ```
   {: pre}
   이는 메시지 `ingress.extension "glb-ingress" created`를 출력합니다. 

이 단계에 도달하면 여러 위치의 Kubernetes 클러스터에 글로벌 로드 밸런서가 구성되어 있습니다. 사용자는 GLB URL `http://<glb_name>.<your_domain_name>`에 액세스하여 애플리케이션을 볼 수 있습니다. 사용자는 자신의 위치에 따라 가장 가까운 클러스터로 경로 재지정되며, CIS가 사용자의 IP 주소를 특정 위치로 맵핑할 수 없는 경우에는 기본 풀의 클러스터로 경로 재지정됩니다. 

## 애플리케이션 보안
{: #secure_via_CIS}

### Web Application Firewall 켜기

WAF(Web Application Firewall)는 ISO 계층 7 공격에 대해 웹 애플리케이션을 보호합니다. 보통 이는 그룹화된 규칙 세트와 결합되며, 이러한 규칙 세트는 악성 트래픽을 필터링하여 애플리케이션을 취약성으로부터 보호하는 것을 목표로 합니다. 

1. Cloud Internet Services 대시보드에서 **보안**으로 이동한 후 **관리** 탭으로 이동하십시오. 
1. **Web Application Firewall** 섹션에서 WAF가 사용으로 설정되어 있는지 확인하십시오. 
1. **OWASP 규칙 세트 보기**를 클릭하십시오. 이 페이지에서는 **OWASP 핵심 규칙 세트**를 검토하고 각 규칙을 개별적으로 사용 또는 사용 안함으로 설정할 수 있습니다. 특정 규칙이 사용으로 설정되면 수신 요청이 해당 규칙을 트리거하는 경우 글로벌 위협 스코어가 증가합니다. 요청에 대한 **조치**의 트리거 여부는 **민감도** 설정에 의해 결정됩니다. 
   1. 기본 OWASP 규칙 세트를 그대로 두십시오. 
   1. **민감도**를 `Low`로 설정하십시오. 
   1. **조치**를 `Simulate`로 설정하여 모든 이벤트를 로그하십시오. 
1. **보안으로 돌아가기**를 클릭하십시오. 
1. **CIS 규칙 세트 보기**를 클릭하십시오. 이 페이지는 웹 사이트 호스팅을 위한 일반적인 기술 스택에 빌드된 추가 규칙을 보여줍니다. 

### 성능 향상 및 서비스 거부(DoS) 공격에 대한 보호 
{: #proxy_setting}

분산 서비스 거부([DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)) 공격은 대상 또는 주변 인프라를 대량의 인터넷 트래픽으로 압도하여 서버, 서비스 또는 네트워크에 대한 정상 트래픽을 방해하는 악의적인 시도입니다. CIS에는 DDoS 공격으로부터 도메인을 보호하는 기능이 갖춰져 있습니다. 

1. CIS 대시보드에서 **신뢰성** > **글로벌 로드 밸런서**를 선택하십시오. 
1. **로드 밸런서** 테이블에서 작성한 GLB를 찾으십시오. 
1. **프록시** 열에서 보안 및 성능 기능을 사용으로 설정하십시오. 

   ![CIS 프록시 토글 켜기](images/solution32-multi-region-k8s-cis/cis-proxy.png)

**사용자의 GLB가 이제 보호됩니다**. 즉각적인 이점은 클러스터의 오리진 IP 주소가 클라이언트로부터 숨겨진다는 것입니다. CIS가 수신 요청에 대한 위협을 발견하는 경우 사용자가 애플리케이션으로 경로 재지정되기 전에 사용자에게 다음과 같은 화면이 표시될 수 있습니다. 

   ![확인 중 - DDoS 보호](images/solution32-multi-region-k8s-cis/cis-DDoS.png)

또한 이제 CIS가 캐시하는 컨텐츠와 캐시 기간을 제어할 수 있습니다. 글로벌 캐싱 레벨 및 브라우저 만기를 정의하려면 **성능** > **캐싱**으로 이동하십시오. **페이지 규칙**으로 글로벌 보안 및 캐싱 규칙을 사용자 정의할 수 있습니다. 페이지 규칙은 특정 도메인 경로를 사용하여 세분화된 구성을 가능하게 합니다. 페이지 규칙의 예를 들면, 다음과 같이 **/assets** 아래에 있는 모든 컨텐츠를 **3일** 동안 캐시하도록 할 수 있습니다. 

   ![페이지 규칙](images/solution32-multi-region-k8s-cis/cis-pagerules.png)

## 리소스 제거
{:removeresources}

### Kubernetes 클러스터 리소스 제거
1. Ingress를 제거하십시오. 
1. 서비스를 제거하십시오. 
1. 배치를 제거하십시오. 
1. 이 튜토리얼을 위해 클러스터를 작성한 경우에는 이를 삭제하십시오. 

### CIS 리소스 제거
1. GLB를 제거하십시오. 
1. 오리진 풀을 제거하십시오. 
1. 상태 확인을 제거하십시오. 

## 관련 컨텐츠
{:related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [최적의 보안을 위한 IBM CIS 관리](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#best-practice-2-configure-your-security-level-selectively)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
* [{{site.data.keyword.registrylong_notm}} 기본 사항](https://{DomainName}/docs/services/Registry?topic=registry-registry_overview#registry_planning)
* [Kubernetes 클러스터에 단일 인스턴스 앱 배치](https://{DomainName}/docs/containers?topic=containers-cs_apps_tutorial#cs_apps_tutorial_lesson1)
* [CIS를 통한 트래픽 및 인터넷 애플리케이션 보안의 우수 사례](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#manage-your-ibm-cis-for-optimal-security)
* [Improving App Availability with Multizone Clusters](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
