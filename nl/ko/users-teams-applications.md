---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# 사용자, 팀, 애플리케이션 구성을 위한 우수 사례
{: #users-teams-applications}

이 튜토리얼에서는 Identity and Access Management를 관리하기 위해 {{site.data.keyword.cloud_notm}}에서 사용 가능한 개념과 이러한 개념을 구현하여 애플리케이션의 여러 개발 단계를 지원하는 방법에 대한 개요를 제공합니다.
{:shortdesc}

애플리케이션을 빌드할 때 개발자가 코드를 커미트하는 시기부터 애플리케이션 코드를 일반 사용자가 사용할 수 있게 되는 시기까지 프로젝트의 개발 라이프사이클을 반영하는 여러 환경을 정의하는 것이 매우 일반적입니다. 이러한 환경의 일반적인 이름은 *샌드박스*, *테스트*, *스테이징*, *UAT*(User Acceptance Testing), *사전 프로덕션*, *프로덕션*입니다. 

이러한 별도의 환경을 작성하는 이유로는 기본 리소스 격리, 관리 및 액세스 정책 구현, 프로덕션 워크로드 보호, 프로덕션에 푸시하기 전에 변경사항의 유효성 검증 등이 있습니다. 

## 목표
{: #objectives}

* {{site.data.keyword.iamlong}} 및 Cloud Foundry 액세스 모델에 대해 학습합니다. 
* 역할과 환경을 분리하여 프로젝트를 구성합니다. 
* 지속적 통합을 설정합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 프로젝트 정의

다음과 같은 컴포넌트를 가진 샘플 프로젝트가 있다고 간주해 봅니다. 
* {{site.data.keyword.containershort_notm}}에 배치된 몇몇 마이크로서비스
* 데이터베이스
* 파일 스토리지 버킷

이 프로젝트에서는 세 가지 환경을 정의합니다. 
* *개발* - 이 환경은 커미트, 단위 테스트 및 스모크 테스트가 실행될 때마다 지속적으로 업데이트됩니다. 이 환경은 프로젝트의 최신, 최대 배치에 대한 액세스를 제공합니다. 
* *테스트* - 이 환경은 코드의 안정적인 분기 또는 태그 뒤에 빌드됩니다. 이 환경에서 사용자 승인 테스트가 수행됩니다. 이 환경의 구성은 프로덕션 환경과 비슷합니다. 이 환경은 실제 데이터(예: 익명화된 프로덕션 데이터)를 사용하여 로드됩니다. 
* *프로덕션* - 이 환경은 이전 환경에서 유효성 검증된 버전을 사용하여 업데이트됩니다. 

**딜리버리 파이프라인은 환경을 통한 빌드의 진행을 관리합니다. **이는 완전히 자동화되거나 환경 간 승인된 빌드를 승격시키기 위해 수동 유효성 검증 게이트를 포함할 수 있습니다. 이는 실제로 열려 있으며 회사의 우수 사례 및 워크플로우와 일치하도록 설정되어야 합니다. 

빌드 파이프라인의 실행을 지원하기 위해 **기능 사용자**를 소개합니다. 기능 사용자는 일반적인 {{site.data.keyword.cloud_notm}} 사용자이지만 실제 세계에 실제 ID를 가지고 있지 않은 팀 구성원입니다. 이 기능 사용자는 강력한 소유권이 필요한 딜리버리 파이프라인 및 기타 클라우드 리소스를 소유합니다. 이 접근 방식은 팀 구성원이 회사를 떠나거나 다른 프로젝트로 이동하는 경우에 도움이 됩니다. 기능 사용자는 사용자의 프로젝트에 전념하며 프로젝트의 수명 동안 변경되지 않습니다. 다음으로 작성하려고 하는 것은 이 기능 사용자에 대한 [API 키](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey)입니다. 기능 사용자를 위장하기 위해 DevOps 파이프라인을 설정하거나 자동화 스크립트를 실행하려고 할 때 이 API 키를 선택합니다. 

프로젝트 팀 구성원에게 책임을 지정하는 것과 관련해서는 다음과 같은 역할 및 관련 권한을 정의해 봅니다. 

|           | 개발        | 테스트  | 프로덕션   |
| --------- | ----------- | ------- | ---------- |
| 개발자    | <ul><li>코드 컨트리뷰션</li><li>로그 파일에 액세스할 수 있음</li><li>앱 및 서비스 구성을 볼 수 있음</li><li>배치된 애플리케이션 사용</li></ul> | <ul><li>로그 파일에 액세스할 수 있음</li><li>앱 및 서비스 구성을 볼 수 있음</li><li>배치된 애플리케이션 사용</li></ul> | <ul><li>액세스 없음</li></ul> |
| 테스터    | <ul><li>배치된 애플리케이션 사용</li></ul> | <ul><li>배치된 애플리케이션 사용</li></ul> | <ul><li>액세스 없음</li></ul> |
| 운영자    | <ul><li>로그 파일에 액세스할 수 있음</li><li>앱 및 서비스 구성을 보고 설정할 수 있음</li></ul> | <ul><li>로그 파일에 액세스할 수 있음</li><li>앱 및 서비스 구성을 보고 설정할 수 있음</li></ul> | <ul><li>로그 파일에 액세스할 수 있음</li><li>앱 및 서비스 구성을 보고 설정할 수 있음</li></ul> |
| 파이프라인 기능 사용자 | <ul><li>애플리케이션을 배치하고 배치 해제할 수 있음</li><li>앱 및 서비스 구성을 보고 설정할 수 있음</li></ul> | <ul><li>애플리케이션을 배치하고 배치 해제할 수 있음</li><li>앱 및 서비스 구성을 보고 설정할 수 있음</li></ul> | <ul><li>애플리케이션을 배치하고 배치 해제할 수 있음</li><li>앱 및 서비스 구성을 보고 설정할 수 있음</li></ul> |

## IAM(Identity and Access Management)
{: #first_objective}

{{site.data.keyword.iamshort}}(IAM)를 사용하면 플랫폼과 인프라 서비스 모두에 대해 사용자를 안전하게 인증하고 여러 {{site.data.keyword.cloud_notm}} 플랫폼에서 일관되게 **리소스**에 대한 액세스를 제어할 수 있습니다. {{site.data.keyword.cloud_notm}} 서비스 세트가 액세스 제어를 위해 Cloud IAM을 사용할 수 있도록 설정되고 **계정** 내의 **리소스 그룹**으로 구성되어 **사용자**가 한 번에 둘 이상의 리소스에 쉽고 빠르게 액세스할 수 있게 합니다. 계정 내의 리소스에 대한 사용자 및 서비스 ID 액세스를 지정하기 위해 Cloud IAM 액세스 **정책**이 사용됩니다. 

**정책**은 액세스 범위를 정의하는 속성 조합과 함께 하나 이상의 **역할**을 사용자 또는 서비스 ID에 지정합니다. 정책은 인스턴스 레벨까지 단일 서비스에 대한 액세스를 제공하거나 리소스 그룹에서 함께 구성된 리소스 세트에 적용될 수 있습니다. 지정하는 사용자 역할에 따라 특정 유형의 API 호출을 수행하거나 UI를 사용하여 서비스에 액세스하거나 플랫폼 관리 태스크를 완료하기 위해 다양한 레벨의 액세스가 사용자 또는 서비스 ID에 허용됩니다. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="IAM 모델의 다이어그램" />
</p>

현재는 IAM을 사용하여 {{site.data.keyword.cloud_notm}} 카탈로그의 일부 서비스를 관리할 수 없습니다. 이 서비스의 경우 허용되는 액세스 레벨을 정의하기 위해 Cloud Foundry 역할이 지정된 인스턴스가 속하는 조직 및 영역에 대한 액세스를 사용자에게 제공하여 Cloud Foundry를 계속 사용할 수 있습니다. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Cloud Foundry 모델의 다이어그램" />
</p>

## 하나의 환경에 대한 리소스 작성

이 샘플 프로젝트에 필요한 세 가지 환경에는 서로 다른 액세스 권한이 필요하고 서로 다른 용량을 할당해야 할 수 있지만 이들 환경은 공통 아키텍처 패턴을 공유합니다. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="하나의 환경을 보여주는 아키텍처 다이어그램" />
</p>

개발 환경을 빌드하는 것부터 시작해 봅니다. 

1. 환경을 배치할 [{{site.data.keyword.cloud_notm}} 위치를 선택하십시오. ](https://{DomainName})
1. Cloud Foundry 서비스 및 앱의 경우: 
   1. [프로젝트에 대한 조직을 작성하십시오](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg). 
   1. [환경에 대한 Cloud Foundry 영역을 작성하십시오](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo). 
   1. 이 영역 아래에서 프로젝트가 사용하는 Cloud Foundry 서비스를 작성하십시오. 
1. [환경에 대한 리소스 그룹을 작성하십시오](https://{DomainName}/account/resource-groups). 
1. 이 그룹에서 {{site.data.keyword.cos_full_notm}}, {{site.data.keyword.la_full_notm}}, {{site.data.keyword.mon_full_notm}}, {{site.data.keyword.cloudant_short_notm}} 등의 리소스 그룹과 호환되는 서비스를 작성하십시오. 
1. {{site.data.keyword.containershort_notm}}에서 [새 Kubernetes 클러스터를 작성](https://{DomainName}/containers-kubernetes/catalog/cluster)한 후 위에서 작성된 리소스 그룹을 선택해야 합니다. 
1. 로그를 전송하고 클러스터를 모니터하도록 {{site.data.keyword.la_full_notm}} 및 {{site.data.keyword.mon_full_notm}}를 구성하십시오. 

다음 다이어그램은 계정 아래에서 프로젝트 리소스가 작성되는 위치를 보여줍니다. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="프로젝트 리소스를 보여주는 다이어그램" />
</p>

## 환경 내에서 역할 지정

1. 사용자를 계정으로 초대하십시오. 
1. 정책을 사용자에게 지정하여 리소스 그룹, 그룹 내 서비스 및 {{site.data.keyword.containershort_notm}} 인스턴스에 액세스할 수 있는 사용자와 이 사용자의 권한을 제어하십시오. [액세스 정책 정의](https://{DomainName}/docs/containers?topic=containers-users#access_policies)를 참조하여 환경에서 사용자에게 맞는 정책을 선택하십시오. 동일한 정책 세트를 가진 사용자는 [동일한 액세스 그룹](https://{DomainName}/docs/iam?topic=iam-groups#groups)에 배치될 수 있습니다. 그러면 정책이 액세스 그룹에 지정되고 그룹의 모든 사용자가 이 정책을 상속하기 때문에 사용자 관리가 단순화됩니다. 
1. 환경 내에서 필요에 따라 사용자의 Cloud Foundry 조직 및 영역 역할을 구성하십시오. [역할 정의](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess)를 참조하여 환경을 기반으로 올바른 역할을 지정하십시오. 

서비스의 문서를 참조하여 서비스가 IAM 및 Cloud Foundry 역할을 특정 조치에 맵핑하는 방법을 이해하십시오. 예를 들어, [{{site.data.keyword.mon_full_notm}} 서비스가 IAM 역할을 조치에 맵핑하는 방법](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam)을 참조하십시오. 

올바른 역할을 사용자에게 지정하려면 몇몇 반복 및 세분화가 필요합니다. 권한은 리소스 그룹 레벨에서 제어되거나 그룹의 모든 리소스를 위한 것이거나 특정 서비스 인스턴스까지 세분화될 수 있다는 점을 고려하면 시간 경과에 따라 프로젝트에 이상적인 액세스 정책이 무엇인지 발견합니다. 

올바른 방법은 최소 권한 세트로 시작한 후 필요에 따라 신중하게 확장하는 것입니다. Kubernetes의 경우 해당 [RBAC(Role-Based Access Control)](https://kubernetes.io/docs/admin/authorization/rbac/)를 확인하여 클러스터 내 권한을 구성할 수 있습니다. 

개발 환경의 경우 이전에 정의된 사용자 책임이 다음과 같이 변환될 수 있습니다. 

|           | IAM 액세스 정책 | Cloud Foundry |
| --------- | ----------- | ------- |
| 개발자    | <ul><li>리소스 그룹: *뷰어*</li><li>리소스 그룹에서 플랫폼 액세스 역할: *뷰어*</li><li>로깅 및 모니터링 서비스 역할: *작성자*</li></ul> | <ul><li>조직 역할: *감사자*</li><li>영역 역할: *감사자*</li></ul> |
| 테스터    | <ul><li>구성이 필요하지 않습니다. 테스터는 개발 환경이 아니라 배치된 애플리케이션에 액세스합니다. </li></ul> | <ul><li>구성이 필요하지 않습니다. </li></ul> |
| 운영자    | <ul><li>리소스 그룹: *뷰어*</li><li>리소스 그룹에서 플랫폼 액세스 역할: *운영자*, *뷰어*</li><li>로깅 및 모니터링 서비스 역할: *작성자*</li></ul> | <ul><li>조직 역할: *감사자*</li><li>영역 역할: *개발자*</li></ul> |
| 파이프라인 기능 사용자 | <ul><li>리소스 그룹: *뷰어*</li><li>리소스 그룹에서 플랫폼 액세스 역할: *편집자*, *뷰어*</li></ul> | <ul><li>조직 역할: *감사자*</li><li>영역 역할: *개발자*</li></ul> |

IAM 액세스 정책 및 Cloud Foundry 역할은 [Identify and Access Management 사용자 인터페이스](https://{DomainName}/iam/#/users)에서 정의됩니다. 

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="개발자 역할에 대한 권한의 구성" />
</p>

## 여러 환경을 위해 복제

여기서 비슷한 단계를 복제하여 다른 환경을 빌드할 수 있습니다. 

1. 환경당 하나의 리소스 그룹을 작성하십시오. 
1. 환경당 하나의 클러스터 및 필수 서비스 인스턴스를 작성하십시오. 
1. 환경당 하나의 Cloud Foundry 영역을 작성하십시오. 
1. 각각의 영역에서 필수 서비스 인스턴스를 작성하십시오. 

<p style="text-align: center;">
  <img title="별도의 클러스터를 사용하여 환경 격리" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="환경을 격리하는 별도의 클러스터를 보여주는 다이어그램" />
</p>

[{{site.data.keyword.cloud_notm}} `ibmcloud` CLI](https://github.com/IBM-Cloud/ibm-cloud-developer-tools), [HashiCorp의 `terraform`](https://www.terraform.io/), [Terraform용 {{site.data.keyword.cloud_notm}} 제공자](https://github.com/IBM-Cloud/terraform-provider-ibm), Kubernetes CLI `kubectl` 등의 도구 조합을 사용하여 이러한 환경의 작성을 스크립트하고 자동화할 수 있습니다. 

환경에 대해 별도의 Kubernetes 클러스터를 사용하면 다음과 같은 좋은 특성이 제공됩니다. 
* 환경에 관계없이 모든 클러스터가 동일한 모양을 가지게 됩니다. 
* 특정 클러스터에 대한 액세스를 가지는 사용자를 더 쉽게 제어할 수 있습니다. 
* 배치 및 기본 리소스에 대한 업데이트 주기에 유연성을 제공합니다. 새 Kubernetes 버전이 있는 경우 먼저 개발 클러스터를 업데이트하고 애플리케이션의 유효성을 검증한 후 다른 환경을 업데이트하는 옵션을 제공합니다. 
* 서로 영향을 줄 수 있는 서로 다른 워크로드의 혼합을 방지합니다(예: 프로덕션 배치를 다른 항목과 격리). 

또 다른 접근 방식은 [Kubernetes 리소스 할당량](https://kubernetes.io/docs/concepts/policy/resource-quotas/)과 함께 [Kubernetes 네임스페이스](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)를 사용하여 환경을 격리하고 리소스 이용을 제어하는 것입니다. 

<p style="text-align: center;">
  <img title="별도의 네임스페이스를 사용하여 환경 격리" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="환경을 격리하는 별도의 네임스페이스를 보여주는 다이어그램" />
</p>

LogDNA UI의 `Search` 입력 상자에서 `namespace: `라는 필드를 사용하여 네임스페이스를 기반으로 로그를 필터링하십시오.
{: tip}

## 딜리버리 파이프라인 설정

서로 다른 환경에 배치하는 것과 관련해서 전체 프로세스를 구동하기 위해 지속적 통합/지속적 딜리버리 파이프라인을 설정할 수 있습니다. 
* 전용 클러스터에서 단위 테스트 및 통합 테스트를 실행하여 `development` 분기에서 최신, 최대 코드로 `Development` 환경을 지속적으로 업데이트하십시오. 
* 자동으로(이전 단계의 모든 테스트가 정상인 경우) 또는 수동 승격 프로세스를 통해 개발 빌드를 `Testing` 환경으로 승격시키십시오. 예를 들어, 일부 팀은 여기서도 서로 다른 분기를 사용하여 작업 중인 개발 상태를 `stable` 분기에 병합합니다. 
* 비슷한 프로세스를 반복하여 `Production` 환경으로 이동하십시오. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="빌드에서 배치까지의 CI/CD 파이프라인" />
</p>

DevOps 파이프라인 구성 시 기능 사용자의 API 키를 사용해야 합니다. 기능 사용자만 앱을 클러스터에 배치하기 위해 필요한 권한을 가지고 있어야 합니다. 

빌드 단계(Phase)에서는 Docker 이미지의 버전을 올바르게 지정하는 것이 중요합니다. Git 커미트 개정을 이미지 태그의 일부로 사용하거나 DevOps 도구 체인에서 제공하는 고유 ID를 사용하거나 이미지에 포함된 실제 빌드 및 소스 코드에 이미지를 쉽게 맵핑할 수 있게 하는 ID를 사용할 수 있습니다. 

Kubernetes에 익숙해지면 Kubernetes의 패키지 관리자인 [Helm](https://helm.sh/)을 사용하여 편리하게 애플리케이션을 버전화하고 어셈블하고 배치할 수 있습니다. [이 샘플 DevOps 도구 체인](https://github.com/open-toolchain/simple-helm-toolchain)은 좋은 시작점이며 Kubernetes 클러스터에 대한 지속적 딜리버리를 위해 사전 구성되어 있습니다. 프로젝트가 여러 마이크로서비스로 성장함에 따라 [Helm 우산 차트](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies)는 애플리케이션을 작성하는 적절한 솔루션을 제공합니다. 

## 튜토리얼 확장

축하합니다. 이제 개발에서 프로덕션까지 애플리케이션을 안전하게 배치할 수 있습니다. 애플리케이션 전달을 개선하는 추가적인 제안사항은 아래와 같습니다. 

* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)를 파이프라인에 추가하여 배치 중에 품질 제어를 수행하십시오. 
* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)를 사용하여 팀 구성원 코딩 컨트리뷰션 및 개발자 간 상호작용을 검토하십시오. 
* [배치 환경 계획, 작성 및 업데이트](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments) 튜토리얼을 수행하여 환경의 배치를 자동화하십시오. 

## 관련 정보

* [{{site.data.keyword.iamshort}} 시작하기](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [리소스 그룹에서 리소스 구성을 위한 우수 사례](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [LogDNA 및 Sysdig를 사용하여 로그 분석 및 상태 모니터](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [Kubernetes에 지속적 배치](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Helm 도구 체인](https://github.com/open-toolchain/simple-helm-toolchain)
* [Kubernetes 및 Helm을 사용하여 마이크로서비스 애플리케이션 개발](https://github.com/open-toolchain/microservices-helm-toolchain)
* [사용자에게 LogDNA에서 로그를 볼 수 있는 권한 부여](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [사용자에게 Sysdig에서 메트릭을 볼 수 있는 권한 부여](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
