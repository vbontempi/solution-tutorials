---
copyright:
  years: 2019
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

# 격리된 Cloud Foundry 엔터프라이즈 앱
{: #isolated-cloud-foundry-enterprise-apps}

{{site.data.keyword.cfee_full_notm}}(CFEE)를 사용하면 여러 격리된 엔터프라이즈용 Cloud Foundry 플랫폼을 요청 시 작성할 수 있습니다. 이를 사용하면 격리된 Kubernetes 클러스터에 배치된 개인용 Cloud Foundry 인스턴스를 얻을 수 있습니다. 퍼블릭 클라우드와 달리, 사용자는 환경(액세스 제어, 용량, 버전, 리소스 사용 및 모니터링)을 완전히 제어할 수 있습니다. {{site.data.keyword.cfee_full_notm}}는 PaaS(Platform as a Service)의 속도 및 혁신을 엔터프라이즈 IT에서 볼 수 있는 인프라 소유권과 함께 제공합니다. 

{{site.data.keyword.cfee_full_notm}}에 대한 유스 케이스는 엔터프라이즈 소유의 혁신 플랫폼입니다. 엔터프라이즈 내 개발자는 새 마이크로서비스를 작성하거나 레거시 애플리케이션을 CFEE로 마이그레이션할 수 있습니다. 그 후에는 Cloud Foundry 마켓플레이스를 사용하여 다른 개발자에게 마이크로서비스를 공개할 수 있습니다. 이렇게 되면 CFEE 인스턴스의 다른 개발자들이 현재 퍼블릭 클라우드에서와 마찬가지로 애플리케이션 내에서 서비스를 이용할 수 있습니다. 

이 튜토리얼에서는 {{site.data.keyword.cfee_full_notm}}를 작성 및 구성하고, 액세스 제어를 설정하고, 애플리케이션 및 서비스를 배치하는 프로세스를 안내합니다. 또한 사용자 정의 서비스를 CFEE와 통합하는 사용자 정의 서비스 브로커를 배치함으로써 CFEE와 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) 간의 관계를 검토합니다. 

## 목표

{: #objectives}

- CFEE와 공용 Cloud Foundry의 비교 및 대조
- CFEE 내에 앱 및 서비스 배치
- Cloud Foundry와 {{site.data.keyword.containershort_notm}} 간의 관계 이해
- 기본 Cloud Foundry 및 {{site.data.keyword.containershort_notm}} 네트워킹 살펴보기

## 사용되는 서비스

{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처

{: #architecture}

![아키텍처](./images/solution45-CFEE-apps/Architecture.png)

1. 관리자가 CFEE 인스턴스를 작성하고 개발자 액세스 권한이 있는 사용자를 추가합니다. 
2. 개발자가 Node.js 스타터 앱을 CFEE에 푸시합니다. 
3. Node.js 스타터 앱이 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)를 사용하여 데이터를 저장합니다. 
4. 개발자가 새 "환영 메시지" 서비스를 추가합니다. 
5. Node.js 스타터 앱이 사용자 정의 서비스 브로커의 새 서비스를 바인드합니다. 
6. Node.js 스타터 앱이 서비스에서 "환영합니다"를 다양한 언어로 표시합니다. 

## 선행 조건

{: #prereq}

- [{{site.data.keyword.cloud_notm}} CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [Cloud Foundry CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## {{site.data.keyword.cfee_full_notm}} 프로비저닝

{:provision_cfee}

이 절에서는 {{site.data.keyword.containershort_notm}}로부터 Kubernetes 작업자 노드에 배치되는 {{site.data.keyword.cfee_full_notm}}의 인스턴스를 작성합니다. 

1. 필수 인프라 리소스를 작성할 수 있도록 [{{site.data.keyword.cloud_notm}} 계정을 준비](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare)하십시오. 
2. {{site.data.keyword.cloud_notm}} 카탈로그에서 [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)의 인스턴스를 작성하십시오. 
3. 다음 항목을 제공하여 CFEE를 구성하십시오. 
   - **표준** 플랜을 선택하십시오. 
   - 서비스 인스턴스의 **이름**을 입력하십시오. 
   - 환경이 작성되는 **리소스 그룹**을 선택하십시오. CFEE 인스턴스를 작성하려면 계정에 있는 하나 이상의 리소스 그룹에 대한 액세스 권한이 필요합니다. 
   - 인스턴스가 배치되는 **지역** 및 **위치**를 선택하십시오. [사용 가능한 프로비저닝 위치 및 데이터 센터](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets)의 목록을 참조하십시오. 
   - **{{site.data.keyword.composeForPostgreSQL}}**이 배치될 공용 Cloud Foundry **조직** 및 **영역**을 선택하십시오. 
   - Cloud Foundry 환경의 **셀 수**를 선택하십시오. 하나의 셀은 Diego 및 Cloud Foundry 애플리케이션을 실행합니다. 고가용성 애플리케이션을 위해서는 **2**개 이상의 셀이 필요합니다. 
   - Cloud Foundry 셀의 크기(CPU 및 메모리)를 결정하는 **시스템 유형**을 선택하십시오. 
4. **인프라** 섹션을 검토하여 CFEE를 지원하는 Kubernetes 클러스터의 특성을 보십시오. **작업자 노드 수**는 셀 수에 2를 더한 값과 같습니다. 프로비저닝된 Kubernetes 작업자 노드 중 두 개는 CFEE 제어 플레인 역할을 수행합니다. 환경이 배치된 Kubernetes 클러스터는 {{site.data.keyword.cloud_notm}} [클러스터](https://{DomainName}/containers-kubernetes/clusters) 대시보드에 표시됩니다. 
5. **작성** 단추를 클릭하여 자동화된 배치를 시작하십시오. 

자동화된 배치에는 약 90 - 120분이 소요됩니다. 작성되고 나면 CFEE 및 지원 서비스의 프로비저닝을 확인하는 이메일을 수신합니다. 

### 조직 및 영역 작성

{{site.data.keyword.cfee_full_notm}}를 작성한 후에는 [조직 및 영역 작성](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs)에서 조직 및 영역을 통해 환경을 구성하는 방법에 대한 정보를 참조하십시오. {{site.data.keyword.cfee_full_notm}}의 앱은 특정 영역에 속합니다. 마찬가지로 영역은 조직 내에 존재합니다. 조직의 구성원은 할당량 플랜, 앱, 서비스 인스턴스 및 사용자 정의 도메인을 공유합니다. 

참고: 맨 위 {{site.data.keyword.cloud_notm}} 헤더에 있는 **관리 > 계정 > Cloud Foundry 조직** 메뉴는 전적으로 공용 {{site.data.keyword.cloud_notm}} 조직을 위한 것입니다. CFEE 조직은 CFEE 인스턴스의 **조직** 페이지에서 관리됩니다. 

아래 단계에 따라 CFEE 조직 및 영역을 작성하십시오. 

1. [Cloud Foundry 대시보드](https://{DomainName}/dashboard/cloudfoundry/overview)의 **엔터프라이즈**에서 **환경**을 선택하십시오. 
2. 자신의 CFEE 인스턴스를 선택한 후 **조직**을 선택하십시오. 
3. **조직 작성** 단추를 클릭하고 **조직 이름**으로 `tutorial`을 제공한 후 **할당량 플랜**을 선택하십시오. **추가**를 클릭하여 완료하십시오. 
4. 새로 작성된 조직인 `tutorial`을 클릭한 후 **영역** 탭을 선택하고 **영역 작성** 단추를 클릭하십시오. 
5. **영역 이름**으로 `dev`를 제공하고 **추가**를 클릭하십시오. 

### 조직 및 영역에 사용자 추가

CFEE에서는 사용자 액세스를 제어하는 역할을 지정할 수 있지만, 이를 수행하려면 **Identity & Access** 페이지에 있는 {{site.data.keyword.cloud_notm}} 헤더의 **관리 > 사용자**에서 {{site.data.keyword.cloud_notm}} 계정으로 사용자를 초대해야 합니다. 

사용자가 초대되었으면 아래 단계에 따라 작성한 `tutorial` 조직에 사용자를 추가하십시오.  

1. [CFEE 인스턴스](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments)를 선택한 후 **조직**을 다시 선택하십시오. 
2. 목록에서 작성된 `tutorial` 조직을 선택하십시오. 
3. **구성원** 탭을 클릭하여 새 사용자를 보고 추가하십시오. 
4. **구성원 추가** 단추를 클릭하고, 사용자 이름을 검색하고, 적절한 **조직 역할**을 선택한 후 **추가**를 클릭하십시오. 

CFEE 조직 및 영역에 사용자를 추가하는 데 대한 자세한 정보는 [여기](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users)에 있습니다. 

## CFEE 앱 배치, 구성 및 실행

{:deploycfeeapps}

이 절에서는 CFEE에 Node.js 애플리케이션을 배치합니다. 배치된 후에는 여기에 {{site.data.keyword.cloudant_short_notm}}를 바인드하고 감사 및 로깅 보존을 사용으로 설정합니다. 애플리케이션 관리를 위해 Stratos 콘솔 또한 추가됩니다. 

### CFEE에 애플리케이션 배치

1. 터미널에서 [get-started-node](https://github.com/IBM-Cloud/get-started-node) 샘플 애플리케이션을 복제하십시오. 
   ```sh
   git clone https://github.com/IBM-Cloud/get-started-node
   ```
   {: codeblock}
2. 앱을 로컬로 실행하여 올바르게 빌드되고 시작되는지 확인하십시오. 브라우저에서 `http://localhost:3000/`에 액세스하여 확인하십시오. 
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. {{site.data.keyword.cloud_notm}}에 로그인하여 CFEE 인스턴스를 대상으로 설정하십시오. 대화식 프롬프트가 새 CFEE 인스턴스를 선택하는 데 도움을 줍니다. 하나의 CFEE 조직 및 영역만 존재하므로 이들 항목이 기본 대상이 됩니다. 둘 이상의 조직 또는 영역을 추가한 경우에는 `ibmcloud target -o tutorial -s dev`를 실행할 수 있습니다. 
   ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
   {: codeblock}
4. **get-started-node** 앱을 CFEE에 푸시하십시오. 
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. 최종 출력의 `routes` 특성 옆에 앱의 엔드포인트가 표시됩니다. 해당 URL을 브라우저에서 열어 애플리케이션이 실행 중인지 확인하십시오. 

### Cloudant 데이터베이스를 작성하고 앱에 바인드

{{site.data.keyword.cloud_notm}} 서비스를 **get-started-node** 애플리케이션에 바인드하려면 먼저 {{site.data.keyword.cloud_notm}} 계정에 서비스를 작성해야 합니다. 

1. [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 서비스를 작성하십시오. **서비스 이름**에 `cfee-cloudant`를 제공하고 CFEE 인스턴스가 작성된 위치와 동일한 위치를 선택하십시오. 
2. 새로 작성된 {{site.data.keyword.cloudant_short_notm}} 서비스 인스턴스를 CFEE에 추가하십시오. 
   1. `tutorial` **조직**으로 다시 이동하십시오. **영역** 탭을 클릭하고 `dev` 영역을 선택하십시오. 
   2. **서비스** 탭을 선택하고 **서비스 추가** 단추를 클릭하십시오. 
   3. 검색 텍스트 상자에 `cfee-cloudant`를 입력하고 결과를 선택하십시오. **추가**를 클릭하여 완료하십시오. 이제 CFEE 애플리케이션에서 해당 서비스를 사용할 수 있습니다. 그러나 이는 여전히 공용 {{site.data.keyword.cloud_notm}}에 있습니다. 
3. 표시된 서비스 인스턴스의 오버플로우 메뉴에서 **애플리케이션에 바인드**를 선택하십시오. 
4. 이전에 푸시한 **GetStartedNode** 애플리케이션을 선택하고 **바인드 후 애플리케이션 다시 스테이징**을 클릭하십시오. 마지막으로 **바인드** 단추를 클릭하십시오. 애플리케이션이 다시 스테이징될 때까지 기다리십시오. `ibmcloud app show GetStartedNode` 명령으로 진행상태를 확인할 수 있습니다. 
5. 브라우저에서 애플리케이션에 액세스하고 사용자의 이름을 추가한 후 `enter`를 누르십시오. 사용자의 이름이 {{site.data.keyword.cloudant_short_notm}} 데이터베이스에 추가됩니다. 
6. **서비스** 탭의 목록에서 `tutorial` 인스턴스를 선택하여 확인하십시오. 이렇게 하면 공용 {{site.data.keyword.cloud_notm}}의 서비스 인스턴스 세부사항 페이지가 열립니다. 
7. **Cloudant 대시보드 실행**을 클릭하고 `mydb` 데이터베이스를 선택하십시오. 사용자의 이름을 사용하는 JSON 문서가 있습니다. 

### 감사 및 로깅 보존 사용

CFEE 관리자는 감사를 통해 로그인, 조직 및 영역 작성, 사용자 멤버십 및 역할 지정, 애플리케이션 배치, 서비스 바인딩, 도메인 구성과 같은 Cloud Foundry 활동을 추적할 수 있습니다. 감사는 {{site.data.keyword.cloudaccesstrailshort}} 서비스와의 통합을 통해 지원됩니다. 

Cloud Foundry 애플리케이션 로그는 {{site.data.keyword.loganalysisshort_notm}}와 통합하여 저장할 수 있습니다. CFEE 관리자가 선택하는 {{site.data.keyword.loganalysisshort_notm}} 서비스 인스턴스는 CFEE 인스턴스에서 생성된 Cloud Foundry 로깅 이벤트를 수신하고 보존하도록 자동으로 구성됩니다. 

CFEE 감사 및 로깅 보존을 사용으로 설정하려면 [이 단계](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging)를 따르십시오. 

### 앱 관리를 위한 Stratos 콘솔 설치

Stratos 콘솔은 Cloud Foundry 관련 작업을 위한 웹 기반 오픈 소스 도구입니다. Stratos 콘솔 애플리케이션은 선택적으로 설치되어 특정 CFEE 환경에서 조직, 영역 및 애플리케이션을 관리하는 데 사용될 수 있습니다. 

Stratos 콘솔 애플리케이션을 설치하려면 다음 작업을 수행하십시오. 

1. Stratos 콘솔을 설치할 CFEE 인스턴스를 여십시오. 
2. **개요** 페이지에서 **Stratos 콘솔 설치**를 클릭하십시오. 이 단추는 관리자 또는 편집자 권한이 있는 사용자에게만 표시됩니다. 
3. Stratos 콘솔 설치 대화 상자에서 설치 옵션을 선택하십시오. Stratos 콘솔 애플리케이션은 CFEE 제어 플레인, 또는 셀 중 하나에 설치할 수 있습니다. Stratos 콘솔의 버전과 설치할 애플리케이션 인스턴스 수를 선택하십시오. Stratos 콘솔 앱을 셀에 설치하는 경우에는 애플리케이션을 배치할 조직 및 영역에 대한 프롬프트가 표시됩니다. 
4. **설치**를 클릭하십시오. 

애플리케이션 설치에는 약 5분이 소요됩니다. 설치가 완료되고 나면 개요 페이지에 **Stratos 콘솔 설치** 단추 대신 **Stratos 콘솔** 단추가 표시됩니다. Stratos 콘솔에 대한 자세한 내용은 [여기](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app)에 있습니다. 

## CFEE와 Kubernetes 간의 관계

CFEE는 애플리케이션 플랫폼으로서 임의의 형태의 전용 또는 공유 가상 인프라에서 실행됩니다. 기반 Cloud Foundry 플랫폼은 IBM이 대신 관리했으므로 오랫동안 개발자들은 이에 대해 크게 신경쓰지 않았습니다. CFEE를 사용하는 경우, 사용자는 Cloud Foundry 애플리케이션을 작성하는 개발자일뿐만 아니라 Cloud Foundry 플랫폼의 운영자이기도 합니다. 이는 CFEE가 사용자가 제어하는 Kubernetes 클러스터에 배치되기 때문입니다. 

Cloud Foundry 개발자에게 있어서 Kubernetes가 생소할 수도 있지만, 이 둘은 공유하는 개념이 많습니다. Cloud Foundry와 마찬가지로, Kubernetes는 팟(Pod)이라고 하는 Kubernetes 구조에서 실행되는 컨테이너에 애플리케이션을 격리합니다. 애플리케이션 인스턴스와 마찬가지로, 팟(Pod)은 여러 사본(ReplicaSet)을 보유할 수 있으며 Kubernetes에서 제공하는 애플리케이션 로드 밸런싱을 사용합니다. 이전에 배치한 Cloud Foundry `GetStartedNode` 애플리케이션은 `diego-cell-0` 팟(Pod)에서 실행됩니다. 고가용성을 지원하기 위해, 또 다른 팟(Pod) `diego-cell-1`은 별도의 Kubernetes 작업자 노드에서 실행됩니다. 이러한 Cloud Foundry 앱은 Kubernetes "내부"에서 실행되므로, 사용자는 Kubernetes 기반 네트워킹을 사용하여 다른 Kubernetes 마이크로서비스와 통신할 수도 있습니다. 다음 절은 CFEE와 Kubernetes 간의 관계를 좀 더 자세히 설명하는 데 도움을 줍니다. 

## Kubernetes 서비스 브로커 배치

이 절에서는 Kubernetes에 CFEE의 서비스 브로커 역할을 수행하는 마이크로서비스를 배치합니다. [서비스 브로커](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md)는 사용 가능한 서비스에 대한 세부사항, 그리고 Cloud Foundry 애플리케이션에 대한 바인딩 및 프로비저닝 지원을 제공합니다. {{site.data.keyword.cloudant_short_notm}} 서비스를 추가하는 데는 기본 제공 {{site.data.keyword.cloud_notm}} 서비스 브로커를 사용했었습니다. 여기서는 사용자 정의 서비스 브로커를 배치하고 사용할 것입니다. 그 후에는 몇 가지 언어로 "환영합니다" 메시지를 리턴하는 `GetStartedNode` 앱을 이 서비스 브로커를 사용하도록 수정합니다. 

1. 터미널로 돌아가 Kubernetes 배치 파일과 서비스 브로커 구현을 제공하는 프로젝트를 복제하십시오. 

   ```sh
   git clone https://github.com/IBM-Cloud/cfee-service-broker-kubernetes.git
   ```
   {: codeblock}

   ```sh
   cd cfee-service-broker-kubernetes
   ```
   {: codeblock}

   ```sh
   git clone https://github.com/IBM/sample-resource-service-brokers.git
   ```
   {: codeblock}

2. {{site.data.keyword.registryshort_notm}}에서 서비스 브로커를 포함하는 Docker 이미지를 빌드하고 저장하십시오. `ibmcloud cr info` 명령을 사용하여 레지스트리 URL을 수동으로 검색하거나 아래 `export REGISTRY` 명령을 사용하여 자동으로 검색하십시오. `cr namespace-add` 명령은 Docker 이미지를 저장하기 위한 네임스페이스를 작성합니다. 

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   `cfee-tutorial`이라는 네임스페이스를 작성하십시오. 

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   `./cfee-service-broker-kubernetes/deployment.yml` 파일을 편집하십시오. 사용자의 컨테이너 레지스트리 URL을 반영하도록 `image` 속성을 업데이트하십시오. 
3. 컨테이너 이미지를 CFEE의 Kubernetes 클러스터에 배치하십시오. 사용자 CFEE의 클러스터는 `default` 리소스 그룹에 있으며, 이는 아직 대상으로 설정되지 않은 경우 대상으로 설정해야 합니다. 사용자 클러스터의 이름을 사용하여, `cluster-config` 명령을 사용해 KUBECONFIG 변수를 내보내십시오. 그 후 배치를 작성하십시오. 
   ```sh
   ibmcloud target -g default
   ```
   {: codeblock}

   ```sh
   ibmcloud ks clusters
   ```
   {: codeblock}

   ```sh
   $(ibmcloud ks cluster-config <your-cfee-cluster-name> --export)
   ```
   {: codeblock}

   ```sh
   kubectl apply -f deployment.yml
   ```
   {: codeblock}
4. 팟(Pod)의 상태가 `Running`인지 확인하십시오. Kubernetes가 이미지를 가져와 컨테이너를 시작하기까지는 잠시 기다려야 할 수 있습니다. `deployment.yml`에서 2개의 `복제본`을 요청했으므로 팟(Pod)이 두 개인 것을 볼 수 있습니다. 
   ```sh
   kubectl get pods
   ```
   {: codeblock}

## 서비스 브로커의 배치 여부 확인

서비스 브로커를 배치했으므로, 이제 올바르게 작동하는지 확인하십시오. 여기서는 이를 몇 가지 방법(첫 번째는 Kubernetes 대시보드를 사용하여, 그 다음에는 Cloud Foundry 앱에서 브로커에 액세스하여, 마지막으로는 브로커에서 실제로 서비스를 바인드하여)으로 수행합니다. 

### Kubernetes 대시보드에서 팟(Pod) 보기

이 절에서는 {{site.data.keyword.containershort_notm}} 대시보드를 사용하여 Kubernetes 아티팩트가 구성되었는지 확인합니다. 

1. [Kubernetes 클러스터](https://{DomainName}/containers-kubernetes/clusters) 페이지에서 자신의 CFEE 서비스 이름으로 시작하고 **-cluster**로 끝나는 행 항목을 클릭하여 자신의 CFEE 클러스터에 액세스하십시오. 
2. 해당 단추를 클릭하여 **Kubernetes 대시보드**를 여십시오. 
3. 왼쪽 메뉴에서 **서비스** 링크를 클릭하고 **tutorial-broker-service**를 선택하십시오. 이 서비스는 이전 단계에서 `kubectl apply`를 실행했을 때 배치되었습니다. 
4. 결과 대시보드에서 다음 항목을 확인하십시오. 
   - 서비스에 Kubernetes 클러스터 내에서만 해석 가능한 오버레이 IP 주소(172.x.x.x)가 제공되었습니다. 
   - 서비스에 실행 중인 서비스 브로커 컨테이너가 있는 두 팟(Pod)에 해당하는 두 엔드포인트가 있습니다. 

서비스가 사용 가능하며 서비스 브로커 팟(Pod)을 중개하고 있는 것을 확인하고 나면 브로커가 사용 가능한 서비스에 대한 정보로 응답하는지 확인할 수 있습니다. 

Kubernetes 대시보드에서 Cloud Foundry 관련 아티팩트를 볼 수 있습니다. 이를 보려면 **네임스페이스**를 클릭하여 아티팩트가 있는 모든 네임스페이스(`cf` Cloud Foundry 네임스페이스 포함)를 보십시오.
{: tip}

### Cloud Foundry 컨테이너에서 브로커에 액세스

여기서는 Cloud Foundry 대 Kubernetes 통신을 보여주기 위해 Cloud Foundry 애플리케이션에서 직접 서비스 브로커에 연결합니다. 

1. 터미널로 돌아가 `ibmcloud target`을 사용하여 CFEE `tutorial` 조직 및 `dev` 영역에 여전히 연결되어 있는지 확인하십시오. 필요한 경우에는 CFEE를 다시 대상으로 설정하십시오. 
   ```sh
   ibmcloud target --cf
   ```
   {: codeblock}
2. 기본적으로 영역에서는 SSH가 사용 안함으로 설정됩니다. 이는 퍼블릭 클라우드와 다르므로 CFEE `dev` 영역에서 SSH를 사용으로 설정하십시오. 
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. `kubectl` 명령을 사용하여 Kubernetes 대시보드에 표시되었던 것과 동일한 클러스터 IP를 표시하십시오. 그 후 `GetStartedNode` 애플리케이션에 SSH로 접속해 해당 IP 주소를 사용하여 서비스 브로커에서 데이터를 검색하십시오. 마지막 명령에서 오류가 발생할 수 있다는 점을 참고하십시오. 이는 다음 단계에서 해결됩니다. 
   ```sh
   kubectl get service tutorial-broker-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf ssh GetStartedNode
   ```
   {: codeblock}

   ```sh
   export CLUSTER_IP=<CLUSTER-IP address>
   ```
   {: codeblock}

   ```sh
   wget --user TestServiceBrokerUser --password TestServiceBrokerPassword -O- http://$CLUSTER_IP/v2/catalog
   ```
   {: codeblock}
4. 아마도 **Connection refused** 오류가 수신되었을 것입니다. 이는 CFEE의 기본 [애플리케이션 보안 그룹](https://docs.cloudfoundry.org/concepts/asg.html)으로 인한 것입니다. 애플리케이션 보안 그룹(ASG)은 Cloud Foundry 컨테이너에서의 egress 트래픽에 대해 허용 가능한 IP 범위를 정의합니다. `GetStartedNode`는 기본 범위를 벗어나므로 이 오류가 발생합니다. SSH 세션을 종료하고 `public_networks` ASG를 다운로드하십시오. 
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. `public_networks.json` 파일을 편집하고 사용되는 ClusterIP 주소가 기존 규칙을 벗어나는지 확인하십시오. 예를 들어, 범위 `172.32.0.0-192.167.255.255`는 해당 ClusterIP를 포함하지 않을 가능성이 높으며 이러한 경우에는 이를 업데이트해야 합니다. 예를 들어, 사용자의 ClusterIP 주소가 `172.21.107.63`인 경우에는 범위가 `172.0.0.0-255.255.255.255`와 같도록 파일을 편집해야 합니다. 
6. Kubernetes 서비스의 IP 주소를 포함하도록 ASG `destination` 규칙을 조정하십시오. JSON 데이터(대괄호로 시작하고 종료됨)만 포함하도록 파일을 잘라내십시오. 그 후 새 ASG를 업로드하십시오. 
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. 3단계를 반복하십시오. 이제 이 단계가 성공하며 샘플 카탈로그 데이터가 검색됩니다. SSH 세션을 종료하여 작업을 완료하십시오. 
   ```sh
   exit
   ```
   {: codeblock}

### CFEE에 서비스 브로커 등록

여기서는 개발자가 서비스 브로커에서 서비스를 프로비저닝하고 바인드할 수 있도록 이를 CFEE에 등록합니다. 이전에는 IP 주소를 사용하여 브로커에 대해 작업했습니다. 여기에는 문제가 있습니다. 서비스 브로커가 다시 시작되면 브로커가 새 IP 주소를 수신하므로 CFEE를 업데이트해야 하기 때문입니다. 이 문제를 해결하기 위해, 여기서는 서비스 브로커에 대한 완전한 도메인 이름(FQDN)을 제공하는 KubeDNS라는 또 다른 Kubernetes 기능을 사용합니다. 

1. `tutorial-service-broker` 서비스의 FQDN을 사용하여 서비스 브로커를 CFEE에 등록하십시오. 이 라우트 또한 CFEE Kubernetes 클러스터 내부의 라우트입니다. 
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. 그 후 브로커에서 제공하는 서비스를 추가하십시오. 샘플 브로커에는 하나의 샘플 서비스밖에 없으므로 하나의 명령만 필요합니다. 
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. 브라우저에서, [**환경**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) 페이지로부터 CFEE 인스턴스에 액세스하여 `조직 -> 영역`으로 이동한 후 `dev` 영역을 선택하십시오. 
4. **서비스** 탭을 선택하고 **서비스 작성** 단추를 선택하십시오. 
5. 검색 텍스트 상자에서 **Test**를 검색하십시오. 브로커의 **Test Node Resource Service Broker Display Name** 샘플 서비스가 표시됩니다. 
6. 이 서비스를 선택하고 **작성** 단추를 클릭한 후 `welcome-service` 이름을 제공하여 서비스 인스턴스를 작성하십시오. 이름이 왜 welcome-service로 지정되었는지는 다음 절에서 알 수 있습니다. 그 후 오버플로우 메뉴의 **애플리케이션에 바인드** 항목을 사용하여 이 서비스를 `GetStartedNode` 앱에 바인드하십시오. 
7. 바인드된 서비스를 보려면 `cf env` 명령을 실행하십시오. 이제 `GetStartedNode` 애플리케이션이 현재 `cloudantNoSQLDB`의 데이터를 사용하는 것과 마찬가지로 `credentials` 오브젝트의 데이터를 이용할 수 있습니다. 
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## 애플리케이션에 서비스 추가

이제까지는 서비스 브로커를 배치했지만 실제 서비스는 배치하지 않았습니다. 서비스 브로커는 `GetStartedNode`에 바인딩 데이터를 제공하지만, 서비스 자체의 실제 기능은 추가되지 않았습니다. 이를 정정하기 위해, `GetStartedNode`에 다양한 언어로 "환영합니다" 메시지를 제공하는 샘플 서비스가 작성되었습니다. 사용자는 이 서비스를 배치하고 이를 사용하도록 브로커 및 애플리케이션을 업데이트합니다. 

1. 다른 개발 팀이 `welcome-service` 구현을 API로 이용할 수 있도록 이를 CFEE에 푸시하십시오. 
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. 브라우저를 사용하여 서비스에 액세스하십시오. 서비스에 대한 `route`는 `push` 명령 출력의 일부로서 표시됩니다. 결과 URL은 `https://welcome.<your-cfee-cluster-domain>`과 같습니다. 다른 언어를 보려면 페이지를 새로 고치십시오. 
3. `sample-resource-service-brokers` 폴더로 돌아와 Node.js 샘플 구현 `sample-resource-service-brokers/node-resource-service-broker/testresourceservicebroker.js`를 편집하여 854행 뒤에 클러스터 URL을 추가하십시오.  

   ```javascript
   // TODO - Do your actual work here
    
   var generatedUserid   = uuid();
   var generatedPassword = uuid();
    
   result = {
     credentials: {
       userid   : generatedUserid,
       password : generatedPassword,
       url: 'https://welcome.<your-cfee-cluster-domain>'
     }
   };
   ```
   {: codeblock}
4. 업데이트된 서비스 브로커를 빌드하고 배치하십시오. 이는 서비스를 바인드하는 앱에 URL 특성이 제공되도록 합니다. 
   ```sh
   cd ..
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   ```sh
   kubectl patch deployment tutorial-servicebroker-deployment -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"version\":\"2\"}}}}}"
   ```
   {: codeblock}
5. 터미널에서 `get-started-node` 애플리케이션 폴더로 다시 이동하십시오.  
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. `get-stared-node`의 `get-started-node/server.js` 파일을 편집하여 다음 미들웨어를 포함시키십시오. 이를 `app.listen()` 호출 바로 앞에 삽입하십시오. 
   ```javascript
   // Use the welcome service
   const request = require('request');
   const testService = appEnv.services['testnoderesourceservicebrokername'];

   if (testService) {
     const { credentials: { url} } = testService[0];
     app.get("/api/welcome", (req, res) => request(url, (e, r, b) => res.send(b)));
   } else {
     app.get("/api/welcome", (req, res) => res.send('Welcome'));
   }
   ```
   {: codeblock}
7. `get-started-node/views/index.html` 페이지를 리팩터하십시오. `h1` 태그에서 `data-i18n="welcome"`을 `id="welcome"`으로 변경하십시오. 그 후 `$(document).ready()` 함수의 끝에 서비스에 대한 호출을 추가하십시오. 
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. `GetStartedNode` 앱을 업데이트하십시오. `server.js`에 추가된 `request` 패키지 종속성을 포함시키고, 새 `url` 특성을 사용하도록 `welcome-service`를 리바인드한 후 마지막으로 앱의 새 코드를 푸시하십시오. 
   ```sh
   npm i request -S
   ```
   {: codeblock}

   ```sh
   ibmcloud cf unbind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf bind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}

모든 작업을 완료했습니다. 이제 애플리케이션으로 이동하여 페이지를 몇 번 새로 고치면서 다양한 언어로 된 환영 메시지를 보십시오. 

![서비스 브로커 응답](./images/solution45-CFEE-apps/service-broker.png)

**Welcome**만 표시되고 다른 언어는 표시되지 않는 경우에는 `ibmcloud cf env GetStartedNode` 명령을 실행하여 서비스의 `url`이 있는지 확인하십시오. 그렇지 않은 경우에는 위 단계를 다시 수행하고 업데이트한 후 브로커를 다시 배치하십시오. `url` 값이 있는 경우에는 이 URL이 브라우저에서 메시지를 리턴하는지 확인하십시오. 그 후 `unbind-service` 및 `bind-service` 명령을 다시 실행하고 `ibmcloud cf restart GetStartedNode`를 다시 실행하십시오.
{: tip}

welcome-service는 Cloud Foundry를 구현으로 사용하지만, Kubernetes 또한 손쉽게 이와 같이 사용할 수 있습니다. 주된 차이점은 서비스에 대한 URL이 `welcome-service.default.svc.cluster.local`이라는 것입니다. KubeDNS URL을 사용하면 서비스에 대한 네트워크 트래픽을 Kubernetes 클러스터 내부에서 유지하는 장점이 있습니다. 

{{site.data.keyword.cfee_full_notm}}를 사용하면 이러한 대체 접근법을 사용할 수 있습니다. 

## 도구 체인을 사용하여 이 솔루션 튜토리얼 배치  

선택적으로, 도구 체인을 사용하여 전체 솔루션 튜토리얼을 배치할 수도 있습니다. 도구 체인을 사용하여 위 항목 모두를 배치하려면 [도구 체인 지시사항](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)을 따르십시오.  

참고: 도구 체인을 사용하는 경우에는 CFEE 인스턴스를 작성하고 CFEE 조직 및 CFEE 영역을 작성한 상태여야 합니다. 자세한 지시사항은 [도구 체인 지시사항](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes) readme에 설명되어 있습니다.  

## 관련 컨텐츠
{:related}

* [Kubernetes 클러스터에 앱 배치](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Diego Components and Architecture](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [CFEE Service Broker on Kubernetes with a Toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

