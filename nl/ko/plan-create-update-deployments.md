---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# 배치 환경 계획, 작성 및 업데이트
{: #plan-create-update-deployments}

솔루션을 작성할 때는 다중 배치 환경이 일반적입니다. 이러한 배치 환경은 개발에서 프로덕션에 이르기까지 프로젝트의 라이프사이클을 반영합니다. 이 튜토리얼에서는 이 배치 환경의 작성 및 유지보수를 자동화하는 {{site.data.keyword.Bluemix_notm}} CLI 및 [Terraform](https://www.terraform.io/) 등의 도구를 소개합니다.
{:shortdesc}

개발자는 같은 내용을 두 번 쓰는 것을 좋아하지 않습니다. [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 원칙이 이 예 중 하나입니다. 마찬가지로 개발자는 환경을 설정하기 위해 사용자 인터페이스에서 많은 수의 클릭을 해야 하는 것을 좋아하지 않습니다. 따라서 시스템 관리자 및 개발자는 반복되고 오류가 발생하기 쉬운 재미없는 태스크를 자동화하기 위해 쉘 스크립트를 오랫동안 사용해 왔습니다. 

IaaS(Infrastructure as a Service), PaaS(Platform as a Service), CaaS(Container as a Service), FaaS(Functions as a Service)는 상위 레벨의 추상을 개발자에게 제공해 왔으며 베어메탈 서버, 관리 데이터베이스, 가상 머신, Kubernetes 클러스터 등의 리소스를 확보하는 것이 더 쉬워졌습니다. 하지만 이 리소스를 프로비저닝하고 나면 사용자 액세스를 구성하고 시간 경과에 따라 구성을 업데이트하는 등의 작업을 수행하기 위해 이들을 함께 연결해야 합니다. 이 모든 단계를 자동화하고 설치를 반복하는 기능은 오늘날 다양한 환경에서 구성을 수행하는 데 있어 필수적입니다. 

용량, 네트워킹, 인증 정보, 로그 상세도 등 환경 간 사소한 차이가 있는 개발 주기의 다양한 단계(Phase)를 지원하기 위해 프로젝트에서 다중 환경은 매우 일반적입니다. [이 다른 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)에서는 사용자, 팀 및 애플리케이션과 샘플 시나리오를 구성하는 우수 사례를 소개했습니다. 샘플 시나리오에서는 *개발*, *테스트* 및 *프로덕션*이라는 세 가지 환경을 고려합니다. 이 환경의 작성을 자동화하려면 어떻게 해야 합니까? 어떤 도구를 사용할 수 있습니까? 

## 목표
{: #objectives}

* 배치할 환경 세트 정의
* 이 환경의 배치를 자동화하기 위해 {{site.data.keyword.Bluemix_notm}} CLI 및 [Terraform](https://www.terraform.io/)을 사용하여 스크립트 작성
* 계정에 이 환경 배치

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 제품을 사용합니다. 
* [Terraform용 {{site.data.keyword.Bluemix_notm}} 제공자](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Identity and Access Management](https://{DomainName}/iam/#/users)
* [{{site.data.keyword.Bluemix_notm}} 명령행 인터페이스 - `ibmcloud` CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. 대상 인프라를 코드로 설명하기 위해 Terraform 파일 세트가 작성됩니다. 
1. 운영자가 `terraform apply`를 사용하여 환경을 프로비저닝합니다. 
1. 환경 구성을 완료하기 위해 쉘 스크립트가 작성됩니다. 
1. 운영자가 환경에 대해 스크립트를 실행합니다. 
1. 환경이 완전히 구성되어 사용할 준비가 됩니다. 

## 사용 가능한 도구 개요
{: #tools}

{{site.data.keyword.Bluemix_notm}}와 상호작용하고 반복 가능한 배치를 작성하는 첫 번째 도구는 [{{site.data.keyword.Bluemix_notm}} 명령행 인터페이스(`ibmcloud` CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli))입니다. `ibmcloud` 및 해당 플러그인을 사용하면 클라우드 리소스의 작성 및 구성을 자동화할 수 있습니다. {{site.data.keyword.virtualmachinesshort}}, Kubernetes 클러스터, {{site.data.keyword.openwhisk_short}}, Cloud Foundry 앱 및 서비스는 모두 명령행에서 프로비저닝할 수 있습니다. 

[이 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)에서 소개되는 또 다른 도구는 HashiCorp의 [Terraform](https://www.terraform.io/)입니다. HashiCorp에 따르면 *Terraform을 사용하면 인프라를 안전하고 예측 가능하게 작성, 변경 및 개선할 수 있습니다. Terraform은 팀 구성원 사이에 공유될 수 있고 코드로 처리될 수 있고 편집, 검토 및 버전화될 수 있는 선언적 구성 파일로 API를 코드화하는 오픈 소스 도구입니다. *Terraform은 코드로서의 인프라입니다. 사용자가 인프라의 모양을 작성하고 Terraform은 필요에 따라 클라우드 리소스를 작성, 업데이트 및 제거합니다. 

다중 클라우드 접근 방식을 지원하기 위해 Terraform은 제공자와 함께 작업합니다. 제공자는 API 상호작용을 이해하고 리소스를 노출시켜야 합니다. {{site.data.keyword.Bluemix_notm}}는 [Terraform용 제공자](https://github.com/IBM-Cloud/terraform-provider-ibm)를 가지고 있어 {{site.data.keyword.Bluemix_notm}}의 사용자가 Terraform을 사용하여 리소스를 관리할 수 있게 합니다. Terraform은 코드로서의 인프라로 분류되지만 IaaS(Infrastructure-As-A-Service) 리소스로 제한되지는 않습니다. Terraform용 {{site.data.keyword.Bluemix_notm}} 제공자는 IaaS(베머메탈, 가상 머신, 네트워크 서비스 등), CaaS({{site.data.keyword.containershort_notm}} 및 Kubernetes 클러스터), PaaS(Cloud Foundry 및 서비스) 및 FaaS({{site.data.keyword.openwhisk_short}}) 리소스를 지원합니다. 

## 배치를 자동화하는 스크립트 작성
{: #scripts}

코드로서의 인프라에 대한 설명을 시작할 때 작성하는 파일을 일반 코드로 처리하는 것이 중요하므로 해당 파일을 소스 제어 관리 시스템에서 저장합니다. 시간이 경과하면 이로 인해 소스 제어 검토 워크플로우를 사용하여 변경사항 적용 전에 변경사항의 유효성을 검증하고 지속적인 통합 파이프라인을 추가하여 인프라 변경사항을 자동으로 배치하는 등의 좋은 특성이 제공됩니다. 

[이 Git 저장소](https://github.com/IBM-Cloud/multiple-environments-as-code)에는 이전에 정의된 환경을 설정하기 위해 필요한 모든 구성 파일이 들어 있습니다. 저장소를 복제하여 파일의 컨텐츠를 자세히 설명하는 다음 절을 수행할 수 있습니다. 

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

저장소의 구조는 다음과 같습니다. 

| 디렉토리 |설명 |
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) | Terraform 파일의 홈 |
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) | 세 환경에 공통인 리소스를 프로비저닝하는 Terraform 파일 |
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) | 지정된 환경에 고유한 Terraform 파일 |
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) | 사용자 정책을 구성하는 Terraform 파일 |

### Terraform을 사용하여 힘든 작업 수행

*개발*, *테스트* 및 *프로덕션* 환경은 모양이 거의 동일합니다. 

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="하나의 배치 환경을 보여주는 다이어그램" />
</p>

이들 환경은 공통 조직 및 환경별 리소스를 공유합니다. 이들 환경은 할당된 용량 및 액세스 권한에 따라 차이가 있습니다. Terraform 파일은 Cloud Foundry 조직을 프로비저닝하는 ***글로벌*** 구성 및 Terraform 작업공간을 사용하여 환경별 리소스를 프로비저닝하는 ***환경별*** 구성을 사용하여 이를 반영합니다. 

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### 글로벌 구성

모든 환경은 공통 Cloud Foundry 조직을 공유하며 각각의 환경은 자체 영역을 가지고 있습니다. 

[terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) 디렉토리에서 이 조직을 프로비저닝하는 Terraform 스크립트를 찾습니다. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf)에는 조직에 대한 정의가 포함되어 있습니다. 

   ```sh
   # create a new organization for the project
   resource "ibm_org" "organization" {
     name             = "${var.org_name}"
     managers         = "${var.org_managers}"
     users            = "${var.org_users}"
     auditors         = "${var.org_auditors}"
     billing_managers = "${var.org_billing_managers}"
   }
   ```

이 리소스에서 모든 특성은 변수를 통해 구성됩니다. 다음 절에서는 이 변수를 설정하는 방법에 대해 학습합니다. 

환경을 완전히 배치하기 위해 Terraform과 {{site.data.keyword.Bluemix_notm}} CLI를 혼합하여 사용합니다. CLI를 사용하여 작성된 쉘 스크립트는 이 조직을 참조하거나 이름 또는 ID별 계정을 참조해야 할 수 있습니다. *global* 디렉토리에는 이 정보를 스크립트에서 재사용하기에 적합한 키/값으로 포함하고 있는 파일을 생성할 [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf)도 포함되어 있습니다. 

   ```sh
   # generate a property file suitable for shell scripts with useful variables relating to the environment
   resource "local_file" "output" {
     content = <<EOF
ACCOUNT_GUID=${data.ibm_account.account.id}
ORG_GUID=${ibm_org.organization.id}
ORG_NAME=${var.org_name}
EOF

     filename = "../outputs/global.env"
   }
   ```

### 개별 환경

Terraform을 사용하여 다중 환경을 관리하는 다양한 접근 방식이 있습니다. Terraform 파일을 별도의 디렉토리(환경당 한 디렉토리) 아래에 복제할 수 있습니다. [Terraform 모듈](https://www.terraform.io/docs/modules/index.html)을 사용하면 공통 구성을 그룹화하고 여러 환경에 걸쳐 모듈을 재사용하여 코드 중복을 줄일 수 있습니다. 별도의 디렉토리는 *개발* 환경을 발전시켜 변경사항을 테스트한 후 변경사항을 다른 환경에 전파할 수 있음을 의미합니다. 이 경우 환경 파일에 있는 모듈의 특정 버전을 참조할 수 있도록 Terraform *모듈*도 자체 소스 코드 저장소에 가지고 있는 것이 일반적입니다. 

환경은 단순하고 비슷하다는 점을 고려하여 [작업공간](https://www.terraform.io/docs/state/workspaces.html#best-practices)이라는 또 다른 Terraform 개념을 사용합니다. 작업공간을 사용하면 다양한 환경에서 동일한 Terraform 파일(.tf)을 사용할 수 있습니다. 예에서 *개발*, *테스트* 및 *프로덕션*이 작업공간입니다. 이 작업공간은 동일한 Terraform 정의를 사용하지만 구성 변수는 다릅니다(다른 이름, 다른 용량). 

각각의 환경에는 다음 사항이 필요합니다. 
* 전용 Cloud Foundry 영역
* 전용 리소스 그룹
* Kubernetes 클러스터
* 데이터베이스
* 파일 스토리지

Cloud Foundry 영역은 이전 단계에서 작성된 조직에 연결됩니다. 환경 Terraform 파일은 이 조직을 참조해야 합니다. 여기서 [Terraform 원격 상태](https://www.terraform.io/docs/state/remote.html)가 도움이 됩니다. 이를 통해 읽기 전용 모드에서 기존 Terraform 상태를 참조할 수 있습니다. 이는 Terraform 구성을 더 작은 조각으로 분할하여 개별 조각에 대한 책임을 다른 팀에게 넘기는 매유 유용한 구성입니다. [backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf)에는 이전에 작성된 조직을 찾는 데 사용된 *글로벌* 원격 상태의 정의가 포함되어 있습니다. 

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

조직을 참조할 수 있게 되면 이 조직 내에서 영역을 간단하게 작성할 수 있습니다. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf)에는 환경에 대한 리소스의 정의가 포함되어 있습니다. 

   ```sh
   # a Cloud Foundry space per environment
   resource "ibm_space" "space" {
     name       = "${var.environment_name}"
     org        = "${data.terraform_remote_state.global.org_name}"
     managers   = "${var.space_managers}"
     auditors   = "${var.space_auditors}"
     developers = "${var.space_developers}"
   }
   ```

*글로벌* 원격 상태에서 조직 이름을 참조하는 방법에 유의하십시오. 기타 특성은 구성 변수에서 가져옵니다. 

다음으로 리소스 그룹입니다. 

   ```sh
   # a resource group
   resource "ibm_resource_group" "group" {
    name     = "${var.environment_name}"
    quota_id = "${data.ibm_resource_quota.quota.id}"
}

   data "ibm_resource_quota" "quota" {
	name = "${var.resource_quota}"
}
   ```

Kubernetes 클러스터가 이 리소스 그룹에서 작성됩니다. {{site.data.keyword.Bluemix_notm}} 제공자는 클러스터를 나타내는 Terraform 리소스를 가지고 있습니다. 

   ```sh
  # a cluster
  resource "ibm_container_cluster" "cluster" {
  	name              = "${var.environment_name}-cluster"
  	datacenter        = "${var.cluster_datacenter}"
  	org_guid          = "${data.terraform_remote_state.global.org_guid}"
  	space_guid        = "${ibm_space.space.id}"
  	account_guid      = "${data.terraform_remote_state.global.account_guid}"
  	hardware          = "${var.cluster_hardware}"
  	machine_type      = "${var.cluster_machine_type}"
  	public_vlan_id    = "${var.cluster_public_vlan_id}"
  	private_vlan_id   = "${var.cluster_private_vlan_id}"
  	resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool" "cluster_workerpool" {
  worker_pool_name  = "${var.environment_name}-pool"
  machine_type      = "${var.cluster_machine_type}"
  cluster           = "${ibm_container_cluster.cluster.id}"
  size_per_zone     = "${var.worker_num}"
  hardware          = "${var.cluster_hardware}"
  resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool_zone_attachment" "cluster_zone" {
  cluster           = "${ibm_container_cluster.cluster.id}"
  worker_pool       =  "${element(split("/",ibm_container_worker_pool.cluster_workerpool.id),1)}"
  zone              = "${var.cluster_datacenter}"
  public_vlan_id    = "${var.cluster_public_vlan_id}"
  private_vlan_id   = "${var.cluster_private_vlan_id}"
  resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

또한 대부분의 특성은 구성 변수에서 초기화됩니다. 데이터 센터, 작업자 수, 작업자 유형을 조정할 수 있습니다. 

{{site.data.keyword.cos_full_notm}} 및 {{site.data.keyword.cloudant_short_notm}} 등의 IAM 사용 서비스도 그룹 내에서 리소스로 작성됩니다. 

   ```sh
# a database
resource "ibm_resource_instance" "database" {
    name              = "database"
    service           = "cloudantnosqldb"
    plan              = "${var.cloudantnosqldb_plan}"
    location          = "${var.cloudantnosqldb_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
# a cloud object storage
resource "ibm_resource_instance" "objectstorage" {
    name              = "objectstorage"
    service           = "cloud-object-storage"
    plan              = "${var.cloudobjectstorage_plan}"
    location          = "${var.cloudobjectstorage_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

Kubernetes 바인딩(시크릿)을 추가하여 애플리케이션에서 서비스 인증 정보를 검색할 수 있습니다. 

   ```sh
   # bind the cloudant service to the cluster
   resource "ibm_container_bind_service" "bind_database" {
      cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	  service_instance_name       = "${ibm_resource_instance.database.name}"
      namespace_id                = "default"
      account_guid                = "${data.terraform_remote_state.global.account_guid}"
      org_guid                    = "${data.terraform_remote_state.global.org_guid}"
      space_guid                  = "${ibm_space.space.id}"
      resource_group_id           = "${ibm_resource_group.group.id}"
}

   # bind the cloud object storage service to the cluster
   resource "ibm_container_bind_service" "bind_objectstorage" {
  	cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	space_guid                  = "${ibm_space.space.id}"
  	service_instance_id         = "${ibm_resource_instance.objectstorage.name}"
  	namespace_id                = "default"
  	account_guid                = "${data.terraform_remote_state.global.account_guid}"
  	org_guid                    = "${data.terraform_remote_state.global.org_guid}"
  	space_guid                  = "${ibm_space.space.id}"
  	resource_group_id           = "${ibm_resource_group.group.id}"
}
   ```

## 계정에 이 환경 배치

### {{site.data.keyword.Bluemix_notm}} CLI 설치

1. [이 지시사항](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)을 수행하여 CLI를 설치하십시오. 
1. 다음을 실행하여 설치의 유효성을 검증하십시오. 
   ```sh
   ibmcloud
   ```
   {: codeblock}

### Terraform 및 Terraform용 {{site.data.keyword.Bluemix_notm}} 제공자 설치

1. [시스템에 맞는 Terraform을 다운로드하여 설치하십시오. ](https://www.terraform.io/intro/getting-started/install.html)
1. [{{site.data.keyword.Bluemix_notm}} 제공자에 맞는 Terraform 2진을 다운로드하십시오. ](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)
   {{site.data.keyword.Bluemix_notm}} 제공자와 함께 Terraform을 설정하려면 이 [링크](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup)를 참조하십시오.
   {:tip}
1. 홈 디렉토리에서 Terraform 2진을 가리키는 `.terraformrc` 파일을 작성하십시오. 다음 예에서 `/opt/provider/terraform-provider-ibm`은 디렉토리의 라우트입니다. 
   ```sh
   # ~/.terraformrc
   providers {
     ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### 코드 가져오기

튜토리얼 저장소를 아직 복제하지 않은 경우 복제하십시오. 

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### 플랫폼 API 키 설정

1. 아직 없는 경우 [플랫폼 API 키](https://{DomainName}/iam/#/apikeys)를 확보한 후 향후 참조를 위해 저장하십시오. 

   > 이후 단계에서 배치 환경을 호스팅할 새 Cloud Foundry 조직을 작성할 계획인 경우에는 계정의 소유자인지 확인하십시오. 
1. 아래 명령을 실행하여 [terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl)을 *terraform/credentials.tfvars*에 복사하십시오. 
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. `terraform/credentials.tfvars`를 편집하고 `ibmcloud_api_key`의 값을 확보한 플랫폼 API 키로 설정하십시오. 

### Cloud Foundry 조직 작성 또는 재사용

새 조직을 작성하거나 기존 조직을 재사용(가져오기)하도록 선택할 수 있습니다. 세 배치 환경의 상위 조직을 작성하려면 **계정 소유자여야 합니다**. 

#### 새 조직을 작성하려면 다음을 수행하십시오. 

1. `terraform/global` 디렉토리로 변경하십시오. 
1. [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl)을 `global.tfvars`에 복사하십시오. 
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. `global.tfvars`를 편집하십시오. 
   1. **org_name**을 작성할 조직 이름으로 설정하십시오. 
   1. **org_managers**를 조직 내에서 *관리자* 역할을 부여할 사용자 ID의 목록으로 설정하십시오. 조직을 작성하는 사용자는 자동으로 관리자가 되므로 목록에 추가해서는 안 됩니다. 
   1. **org_users**를 조직에 초대할 모든 사용자의 목록으로 설정하십시오. 향후 단계에서 사용자의 액세스를 구성하려는 경우 사용자를 여기에 추가해야 합니다. 

   ```sh
   org_name = "a-new-organization"
   org_managers = [ "user1@domain.com", "another-user@anotherdomain.com" ]
   org_users = [ "user1@domain.com", "another-user@anotherdomain.com", "more-user@domain.com" ]
   ```
   {: codeblock}
1. `terraform/global` 폴더에서 Terraform을 초기화하십시오. 
   ```sh
   terraform init
   ```
   {: codeblock}
1. Terraform 플랜을 확인하십시오. 
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. 변경사항을 적용하십시오. 
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Terraform이 완료되면 다음 사항이 작성된 것입니다. 
* 새 Cloud Foundry 조직
* 체크아웃의 `outputs` 디렉토리 아래에 `global.env` 파일. 이 파일은 사용자가 다른 스크립트에서 참조할 수 있는 환경 변수를 가지고 있습니다. 
* `terraform.tfstate` 파일

> 이 튜토리얼에서는 Terraform 상태를 위해 `local` 백엔드 제공자를 사용합니다. Terraform을 검색하거나 프로젝트에 대해 혼자 작업할 때 유용하지만 팀으로 작업하거나 더 큰 인프라에서 작업할 때도 Terraform은 상태를 원격 위치에 저장할 수 있도록 지원합니다. Terraform 상태가 Terraform 오퍼레이션에 중요하다는 점을 고려하면 Terraform 상태를 위해 원격 고가용성 복원성 스토리지를 사용하도록 권장됩니다. 사용 가능한 옵션의 목록은 [Terraform 백엔드 유형](https://www.terraform.io/docs/backends/types/index.html)을 참조하십시오. 일부 백엔드는 Terraform 상태의 버전화 및 잠금도 지원합니다. 

#### 관리하는 조직을 재사용하려면 다음을 수행하십시오. 

계정 소유자가 아니지만 계정에서 조직을 관리하는 경우에도 기존 조직을 Terraform으로 가져올 수 있습니다. 

1. 조직 GUID를 검색하십시오. 
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. `terraform/global` 디렉토리로 변경하십시오. 
1. [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl)을 `global.tfvars`에 복사하십시오. 
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Terraform을 초기화하십시오. 
   ```sh
   terraform init
   ```
   {: codeblock}
1. Terraform을 초기화한 후 조직을 Terraform 상태로 가져오십시오. 
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. 기존 조직 이름 및 구조와 일치하도록 `global.tfvars`를 조정하십시오. 
1. 변경사항을 적용하십시오. 
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### 환경별 영역, 클러스터 및 서비스 작성

이 절에서는 `개발` 환경에 초점을 둡니다. 다른 환경과 단계는 동일하며 변수에 대해 선택하는 값만 다릅니다. 

1. 체크아웃의 `terraform/per-environment` 폴더로 변경하십시오. 
1. 템플리트 `tfvars` 파일을 복사하십시오. 환경별로 하나의 파일이 있습니다. 
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. `development.tfvars`를 편집하십시오. 
   1. **environment_name**을 작성할 Cloud Foundry 영역의 이름으로 설정하십시오. 
   1. **space_developers**를 이 영역에 대한 개발자 목록으로 설정하십시오. **Terraform이 사용자 대신 서비스를 프로비저닝할 수 있도록 사용자의 이름을 목록에 추가해야 합니다. **
   1. **cluster_datacenter**를 클러스터를 작성할 위치로 설정하십시오. 다음을 사용하여 사용 가능한 위치를 찾으십시오.
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. 클러스터의 사설(**cluster_private_vlan_id**) 및 공용(**cluster_public_vlan_id**) VLAN을 설정하십시오. 다음을 사용하여 위치에 사용 가능한 VLAN을 찾으십시오.
      ```sh
      ibmcloud cs vlans <location>
      ```
      {: codeblock}
   1. **cluster_machine_type**을 설정하십시오. 다음을 사용하여 위치에 사용 가능한 시스템 유형 및 특성을 찾으십시오.
      ```sh
      ibmcloud cs machine-types <location>
      ```
      {: codeblock}

   1. **resource_quota**를 설정하십시오. 다음을 사용하여 사용 가능한 리소스 할당량 정의를 찾으십시오.
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. Terraform을 초기화하십시오. 
   ```sh
   terraform init
   ```
   {: codeblock}
1. *개발* 환경을 위한 새 Terraform 작업공간을 작성하십시오. 
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   나중에 환경 사이에서 전환하려면 다음을 사용하십시오.
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. Terraform 플랜을 확인하십시오. 
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   다음과 같이 표시되어야 합니다.
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. 변경사항을 적용하십시오. 
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Terraform이 완료되면 다음 사항이 작성된 것입니다. 
* 리소스 그룹
* Cloud Foundry 영역
* 작업자 풀 및 구역이 첨부된 Kubernetes 클러스터
* 데이터베이스
* 데이터베이스 인증 정보가 포함된 Kubernetes 시크릿
* 스토리지
* 스토리지 인증 정보가 포함된 Kubernetes 시크릿
* 로깅(LogDNA) 인스턴스
* 모니터링(Sysdig) 인스턴스
* 체크아웃의 `outputs` 디렉토리 아래에 `development.env` 파일. 이 파일은 사용자가 다른 스크립트에서 참조할 수 있는 환경 변수를 가지고 있습니다. 
* `terraform.tfstate.d/development` 아래에 환경별 `terraform.tfstate`

`테스트` 및 `프로덕션`에 대해 이 단계를 반복할 수 있습니다. 

### 사용자 정책 지정

이전 단계에서는 Terraform 제공자를 사용하여 Cloud Foundry 조직 및 영역에서 역할을 구성할 수 있었습니다. Kubernetes 클러스터 등의 다른 리소스에 대한 사용자 정책의 경우 복제된 저장소에 있는 [roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) 폴더를 사용합니다. 

[이 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)에 정의된 *개발* 환경의 경우 정의할 정책은 다음과 같습니다. 

|           | IAM 액세스 정책 |
| --------- | ----------- |
| 개발자    | <ul><li>리소스 그룹: *뷰어*</li><li>리소스 그룹에서 플랫폼 액세스 역할: *뷰어*</li><li>로깅 및 모니터링 서비스 역할: *작성자*</li></ul> |
| 테스터    | <ul><li>구성이 필요하지 않습니다. 테스터는 개발 환경이 아니라 배치된 애플리케이션에 액세스합니다. </li></ul> |
| 운영자    | <ul><li>리소스 그룹: *뷰어*</li><li>리소스 그룹에서 플랫폼 액세스 역할: *운영자*, *뷰어*</li><li>로깅 및 모니터링 서비스 역할: *작성자*</li></ul> |
| 파이프라인 기능 사용자 | <ul><li>리소스 그룹: *뷰어*</li><li>리소스 그룹에서 플랫폼 액세스 역할: *편집자*, *뷰어*</li></ul> |

하나의 팀이 여러 개발자로 구성될 수 있다는 점을 고려하면 [액세스 그룹 개념](https://{DomainName}/docs/iam?topic=iam-groups#groups)을 활용하여 사용자 정책의 구성을 단순화할 수 있습니다. 단일 정책으로 그룹 내 모든 엔티티에 동일한 액세스를 지정할 수 있도록 계정 소유자가 액세스 그룹을 작성할 수 있습니다. 

*개발* 환경에서 *개발자* 역할의 경우 이는 다음과 같이 변환됩니다. 

   ```sh
resource "ibm_iam_access_group" "developer_role" {
  name        = "${var.access_group_name_developer_role}"
  description = "${var.access_group_description}"
}
resource "ibm_iam_access_group_policy" "resourcepolicy_developer" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Viewer"]

  resources = [{
    resource_type = "resource-group"
    resource      = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_platform_accesspolicy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles        = ["Viewer"]

  resources = [{
    resource_group_id = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_logging_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "logdna"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.logdna_instance_id}"
  }]
}
resource "ibm_iam_access_group_policy" "developer_monitoring_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "sysdig-monitor"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.sysdig_instance_id}"
  }]
}

   ```

체크아웃의 [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) 파일은 정의된 *개발자*, *운영자*, *테스터* 및 *기능 사용자* 역할에 대해 이 리소스의 예를 가지고 있습니다. *개발* 환경에서 *개발자, 운영자, 테스트 및 기능 사용자* 역할을 가진 사용자에 대해 이전 절에서 정의된 대로 정책을 설정하려면 다음을 수행하십시오. 

1. `terraform/roles/development` 디렉토리로 변경하십시오. 
2. 템플리트 `tfvars` 파일을 복사하십시오. 환경별로 하나의 파일이 있습니다(`roles` 디렉토리의 각 해당 폴더 아래에서 `production` 및 `testing` 템플리트를 찾을 수 있음). 

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. `development.tfvars`를 편집하십시오. 

   - **iam_access_members_developers**를 액세스 권한을 부여할 개발자의 목록으로 설정하십시오. 
   - **iam_access_members_operators**를 운영자의 목록으로 설정하고 이런 방식으로 계속 다른 항목을 설정하십시오. 
4. Terraform을 초기화하십시오. 
   ```sh
   terraform init
   ```
   {: codeblock}

5. Terraform 플랜을 확인하십시오. 
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   다음과 같이 표시되어야 합니다.
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. 변경사항을 적용하십시오. 
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
`테스트` 및 `프로덕션`에 대해 이 단계를 반복할 수 있습니다. 

## 리소스 제거

1. `roles` 아래의 `development` 폴더로 이동하십시오. 
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. 액세스 그룹 및 액세스 정책을 영구 삭제하십시오. 
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. `development` 작업공간을 활성화하십시오. 
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. 리소스 그룹, 영역, 서비스, 클러스터를 영구 삭제하십시오. 
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. `testing` 및 `production` 작업공간에 대해 이 단계를 반복하십시오. 
1. 이를 작성한 경우에는 조직을 영구 삭제하십시오. 
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## 관련 컨텐츠

* [Terraform 튜토리얼](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Terraform 제공자](https://ibm-cloud.github.io/tf-ibm-docs/)
* [Terraform용 {{site.data.keyword.Bluemix_notm}} 제공자 사용 예](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
