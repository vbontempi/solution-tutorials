---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 클라우드 애플리케이션에 엔드 투 엔드 보안 적용
{: #cloud-e2e-security}

잠재적인 보안 위험과 이러한 위험에 대한 보호 방법을 명확히 이해하지 않으면 완벽한 애플리케이션 아키텍처를 구축할 수 없습니다. 애플리케이션 데이터는 유실되거나, 침해되거나 도난되어서는 안 되는 중요한 리소스입니다. 또한 데이터는 암호화 기술을 통해 저장 또는 전송 중에 보호되어야 합니다. 저장 데이터를 암호화하면 데이터가 유실되거나 도난당한 후에도 정보가 노출되지 않도록 보호할 수 있습니다. HTTPS, SSL 및 TLS과 같은 방법을 통해 전송(예: 인터넷을 통해) 데이터를 암호화하면 도청을 방지하여 중간자 공격을 막을 수 있습니다. 

특정 리소스에 대한 사용자의 액세스를 인증하고 권한 부여하는 것은 많은 애플리케이션의 또 다른 일반적인 요구사항입니다. 소셜 ID를 사용하는 고객 및 공급자, 클라우드 호스팅 디렉토리를 사용하는 파트너, 조직의 ID 제공자를 사용하는 직원 등으로 인해 다양한 인증 체계를 지원해야 할 수 있습니다. 

이 튜토리얼에서는 {{site.data.keyword.cloud}} 카탈로그에서 사용 가능한 주요 보안 서비스와 이를 사용하는 방법을 함께 안내합니다. 파일 공유를 제공하는 애플리케이션은 보안 개념을 실제 상황에 적용합니다.
{:shortdesc}

## 목표
{: #objectives}

* 고유 암호화 키로 스토리지 버킷의 컨텐츠를 암호화
* 사용자가 애플리케이션에 액세스하기 전에 인증하도록 요구
* 클라우드 서비스 간 보안 관련 API 호출 및 기타 조치를 모니터하고 감사

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* 선택사항: [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

이 튜토리얼은 [비Lite 계정](https://{DomainName}/docs/account?topic=account-accounts#accounts)을 필요로 하며 비용을 발생시킬 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

이 튜토리얼은 사용자 그룹이 공통 스토리지 풀에 파일을 업로드할 수 있도록 하고 공유 가능한 링크를 통해 이러한 파일에 대한 액세스를 제공하는 샘플 애플리케이션을 제공합니다. 이 애플리케이션은 Node.js로 작성되며 {{site.data.keyword.containershort_notm}}에 Docker 컨테이너로서 배치됩니다. 이는 애플리케이션의 보안 상태를 향상시키기 위해 몇 가지 보안 관련 서비스 및 기능을 이용합니다. 

<p style="text-align: center;">

  ![아키텍처](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. 사용자가 애플리케이션에 연결합니다. 
2. 사용자 정의 도메인 및 TLS 인증서를 사용하는 경우 이 인증서는 {{site.data.keyword.cloudcerts_short}}에서 배치하고 관리합니다. 
3. {{site.data.keyword.appid_short}}에서는 애플리케이션을 보안하고 사용자를 인증 페이지로 경로 재지정합니다. 사용자는 여기서 가입할 수도 있습니다. 
4. 애플리케이션이 Kubernetes 클러스터의 {{site.data.keyword.registryshort_notm}}에 저장된 이미지로부터 실행됩니다. 이 이미지의 취약성 여부가 자동으로 스캔됩니다. 
5. 업로드된 파일이 {{site.data.keyword.cos_short}}에 저장되고 해당 메타데이터는 {{site.data.keyword.cloudant_short_notm}}에 저장됩니다. 
6. 파일 스토리지 버킷이 사용자 제공 키를 이용하여 데이터를 암호화합니다. 
7. 애플리케이션 관리 활동이 {{site.data.keyword.cloudaccesstrailshort}}에 의해 로그됩니다. 

## 시작하기 전에
{: #prereqs}

1. [이러한 단계](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)에 따라 모든 필요한 명령행 인터페이스(CLI)를 설치하십시오. 
2. 이 튜토리얼에서 사용하는 최신 버전의 플러그인이 있는지 확인하십시오. 업그레이드하려면 `ibmcloud plugin update --all`을 사용하십시오. 

## 서비스 작성
{: #setup}

### 애플리케이션 배치 위치 결정

1. 애플리케이션 및 해당 리소스를 배치할 **위치**, **Cloud Foundry 조직 및 영역**, **리소스 그룹**을 식별하십시오. 

### 사용자 및 애플리케이션 활동 캡처 
{: #activity-tracker }

{{site.data.keyword.cloudaccesstrailshort}} 서비스는 {{site.data.keyword.Bluemix_notm}}에서 서비스의 상태를 변경하는, 사용자가 시작한 활동을 기록합니다. 이 튜토리얼의 끝에서는 튜토리얼의 단계를 완료하여 생성된 이벤트를 검토합니다. 

1. {{site.data.keyword.Bluemix_notm}} 카탈로그에 액세스하여 [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)의 인스턴스를 작성하십시오. Cloud Foundry 영역당 하나의 {{site.data.keyword.cloudaccesstrailshort}} 인스턴스만 있을 수 있다는 점을 참고하십시오. 
2. 인스턴스가 작성되고 나면 이름을 **secure-file-storage-activity-tracker**로 변경하십시오. 
3. 모든 Activity Tracker 이벤트를 보기 위해 필요한 권한이 지정되어 있는지 확인하려면 다음 작업을 수행하십시오. 
   1. [Identity & Access > 사용자](https://{DomainName}/iam/#/users)로 이동하십시오. 
   2. 목록에서 자신의 사용자 이름을 선택하십시오. 
   3. **액세스 정책**에서, 서비스가 작성된 위치에 **뷰어** 역할을 사용하여 {{site.data.keyword.loganalysisshort_notm}} 서비스에 대한 정책을 작성하십시오(이 정책이 없는 경우). 
   4. **Cloud Foundry 액세스 권한**에서, 자신에게 {{site.data.keyword.cloudaccesstrailshort}}가 프로비저닝된 Cloud Foundry 영역에 대한 **개발자** 역할이 있는지 확인하십시오. 그렇지 않은 경우에는 조직의 Cloud Foundry 관리자와 협력하여 이 역할을 지정하십시오. 

권한 설정에 대한 자세한 지시사항은 [{{site.data.keyword.cloudaccesstrailshort}} 문서](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy)를 참조하십시오.
{: tip}

### 애플리케이션을 위한 클러스터 작성

{{site.data.keyword.containershort_notm}}에서는 Kubernetes 클러스터에서 실행되는 Docker 컨테이너에 고가용성 앱을 배치하는 데 필요한 환경을 제공합니다. 

기존 **표준** 클러스터가 있으며 이 튜토리얼을 위해 이를 재사용하려는 경우에는 이 절을 건너뛰십시오.
{: tip}

1. [클러스터 작성 페이지](https://{DomainName}/containers-kubernetes/catalog/cluster/create)에 액세스하십시오. 
   1. **위치**를 이전 단계에서 사용된 위치로 설정하십시오. 
   2. **클러스터 유형**을 **표준**으로 설정하십시오. 
   3. **가용성**을 **단일 구역**으로 설정하십시오. 
   4. **마스터 구역**을 선택하십시오. 
2. 기본 **Kubernetes 버전 ** 및 **하드웨어 격리**를 유지하십시오. 
3. 이 클러스터에서 이 튜토리얼만 배치하려는 경우에는 **작업자 노드**를 **1**로 설정하십시오. 
4. **클러스터 이름**을 **secure-file-storage-cluster**로 설정하십시오. 
5. **클러스터 작성** 단추를 클릭하십시오. 

클러스터가 프로비저닝되는 중에, 이 튜토리얼에 필요한 다른 서비스를 작성하십시오. 

### 고유 암호화 키 사용

{{site.data.keyword.keymanagementserviceshort}}는 여러 {{site.data.keyword.Bluemix_notm}} 서비스에 사용되는 암호화된 키를 프로비저닝하는 데 도움을 줍니다. {{site.data.keyword.keymanagementserviceshort}}와 {{site.data.keyword.cos_full_notm}}는 [저장 데이터를 보호하기 위해 함께 작동합니다](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos). 이 절에서는 스토리지 버킷을 위한 하나의 루트 키를 작성합니다. 

1. [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms)의 인스턴스를 작성하십시오. 
   * 이름을 **secure-file-storage-kp**로 설정하십시오. 
   * 서비스 인스턴스를 작성할 리소스 그룹을 선택하십시오. 
2. **관리**에서 **키 추가** 단추를 클릭하여 새 루트 키를 작성하십시오. 이는 스토리지 버킷 컨텐츠를 암호화하는 데 사용됩니다. 
   * 이름을 **secure-file-storage-root-enckey**로 설정하십시오. 
   * 키 유형을 **루트 키**로 설정하십시오. 
   * 그 후 **키 생성**을 수행하십시오. 

[기존 루트 키 가져오기](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys)를 통해 BYOK(Bring Your Own Key)를 수행하십시오.
{: tip}

### 사용자 파일의 스토리지 설정

파일 공유 애플리케이션은 파일을 {{site.data.keyword.cos_short}} 버킷에 저장합니다. 파일과 사용자 간의 관계는 {{site.data.keyword.cloudant_short_notm}} 데이터베이스에 메타데이터로 저장됩니다. 이 절에서는 이러한 서비스를 작성하고 구성합니다. 

#### 컨텐츠를 위한 버킷

1. [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)의 인스턴스를 작성하십시오. 
   * **이름**을 **secure-file-storage-cos**로 설정하십시오. 
   * 이전 서비스와 동일한 **리소스 그룹**을 사용하십시오. 
2. **서비스 인증 정보**에서 *새 인증 정보*를 작성하십시오. 
   * **이름**을 **secure-file-storage-cos-acckey**로 설정하십시오. 
   * **역할**을 **작성자**로 설정하십시오. 
   * **서비스 ID**를 지정하지 마십시오. 
   * **인라인 구성 매개변수**를 **{"HMAC":true}**로 설정하십시오. 이는 사전 서명된 URL을 생성하는 데 필요합니다. 
   * **추가**를 클릭하십시오. 
   * **인증 정보 보기**를 클릭하여 인증 정보를 기록하십시오. 이는 나중 단계에서 필요합니다. 
3. 메뉴에서 **엔드포인트**를 클릭하십시오. **복원성**을 **지역**으로 설정하고 **위치**를 대상 위치로 설정하십시오. **공용** 서비스 엔드포인트를 복사하십시오. 이는 나중에 애플리케이션 구성에서 사용됩니다. 

버킷을 작성하기 전에, **secure-file-storage-kp**에 저장된 루트 키에 대한 액세스 권한을 **secure-file-storage-cos**에 부여하십시오. 

1. {{site.data.keyword.cloud_notm}} 콘솔에서 [Identity & Access > 권한 부여](https://{DomainName}/iam/#/authorizations)로 이동하십시오. 
2. **작성** 단추를 클릭하십시오. 
3. **소스 서비스** 메뉴에서 **Cloud Object Storage**를 선택하십시오. 
4. **소스 서비스 인스턴스** 메뉴에서 이전에 작성한 **secure-file-storage-cos** 서비스를 선택하십시오. 
5. **대상 서비스** 메뉴에서 **Key Protect**를 선택하십시오. 
6. **대상 서비스 인스턴스** 메뉴에서 권한 부여할 **secure-file-storage-kp** 서비스를 선택하십시오. 
7. **독자** 역할을 사용으로 설정하십시오. 
8. **권한 부여** 단추를 클릭하십시오. 

마지막으로 버킷을 작성하십시오. 

1. [리소스 목록](https://{DomainName}/resources)에서 **secure-file-storage-cos** 서비스 인스턴스에 액세스하십시오. 
2. **버킷 작성**을 클릭하십시오. 
   1. **이름**을 **&lt;your-initials&gt;-secure-file-upload**와 같이 고유 값으로 설정하십시오. 
   2. **복원성**을 **지역**으로 설정하십시오. 
   3. **위치**를 **secure-file-storage-kp** 서비스를 작성한 위치와 동일하게 설정하십시오. 
   4. **스토리지 클래스**를 **표준**으로 설정하십시오. 
3. 선택란을 **Key Protect 키 추가**로 선택하십시오. 
   1. **secure-file-storage-kp** 서비스를 선택하십시오. 
   2. **secure-file-storage-root-enckey**를 키로 선택하십시오. 
4. **버킷 작성**을 클릭하십시오. 

#### 사용자와 파일 간의 데이터베이스 맵 관계

{{site.data.keyword.cloudant_short_notm}} 데이터베이스는 애플리케이션에서 업로드된 모든 파일에 대한 메타데이터를 포함합니다. 

1. [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)의 인스턴스를 작성하십시오. 
   * **이름**을 **secure-file-storage-cloudant**로 설정하십시오. 
   * 위치를 설정하십시오. 
   * 이전 서비스와 동일한 **리소스 그룹**을 사용하십시오. 
   * **사용 가능한 인증 방법**을 **IAM만 사용**으로 설정하십시오. 
2. **서비스 인증 정보**에서 *새 인증 정보*를 작성하십시오. 
   * **이름**을 **secure-file-storage-cloudant-acckey**로 설정하십시오. 
   * **역할**을 **관리자**로 설정하십시오. 
   * *선택사항* 필드의 기본값을 유지하십시오. 
   * **추가**하십시오. 
3. **인증 정보 보기**를 클릭하여 인증 정보를 기록하십시오. 이는 나중 단계에서 필요합니다. 
4. **관리**에서 Cloudant 대시보드를 실행하십시오. 
5. **데이터베이스 작성**을 클릭하여 **secure-file-storage-metadata**라는 데이터베이스를 작성하십시오. 

### 사용자 인증

{{site.data.keyword.appid_short}}를 사용하면 리소스를 보안하고 애플리케이션에 인증을 추가할 수 있습니다. {{site.data.keyword.appid_short}}는 {{site.data.keyword.containershort_notm}}와 [통합](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth)되어 클러스터에 배치된 애플리케이션에 액세스하는 사용자를 인증합니다. 

1. [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)의 인스턴스를 작성하십시오. 
   * **서비스 이름**을 **secure-file-storage-appid**로 설정하십시오. 
   * 이전 서비스와 동일한 **위치** 및 **리소스 그룹**을 사용하십시오. 
2. **ID 제공자 / 관리**의 **인증 설정** 탭에서 애플리케이션에 사용할 도메인을 가리키는 **웹 경로 재지정 URL**을 추가하십시오. 예를 들어, 클러스터 Ingress 하위 도메인이
`<cluster-name>.us-south.containers.appdomain.cloud`인 경우 경로 재지정 URL은 `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback`입니다. {{site.data.keyword.appid_short}}의 경우 웹 경로 재지정 URL은 **https**여야 합니다.  Ingress 하위 도메인은 클러스터 대시보드에서 보거나 `ibmcloud ks cluster-get <cluster-name>`을 사용하여 볼 수 있습니다. 

{{site.data.keyword.appid_short}} 대시보드에서는 사용되는 ID 관리자, 그리고 로그인 및 사용자 관리 경험을 사용자 정의해야 합니다. 이 튜토리얼에서는 작업을 간단하게 하기 위해 기본값을 사용합니다. 프로덕션 환경에서는 다요소 인증(MFA) 및 고급 비밀번호 규칙을 사용하는 것을 고려하십시오.
{: tip}

## 앱 배치

모든 서비스가 구성되었습니다. 이 절에서는 클러스터에 튜토리얼 애플리케이션을 배치합니다. 

### 코드 가져오기

1. 애플리케이션의 코드를 가져오십시오. 
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. **secure-file-storage** 디렉토리로 이동하십시오. 
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### Docker 이미지 빌드

1. {{site.data.keyword.registryshort_notm}}에서 [Docker 이미지를 빌드하십시오](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating). 
   - `ibmcloud cr info`로 **us**.icr.io 또는 **uk**.icr.io와 같은 레지스트리 엔드포인트를 찾으십시오. 
   - 컨테이너 이미지를 저장할 네임스페이스를 작성하십시오.
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - **secure-file-storage**를 이미지 이름으로 사용하십시오. 

   ```sh
   ibmcloud cr build -t <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### 인증 정보 및 구성 설정 채우기

1. `credentials.template.env`를 `credentials.env`에 복사하십시오. 
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. `credentials.env`를 편집하고 공백을 다음 값으로 채우십시오. 
   * {{site.data.keyword.cos_short}} 서비스 지역 엔드포인트, 버킷 이름, **secure-file-storage-cos**에 대해 작성된 인증 정보
   * **secure-file-storage-cloudant**에 대한 인증 정보
3. `secure-file-storage.template.yaml`을 `secure-file-storage.yaml`로 복사하십시오. 
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. `secure-file-storage.yaml`을 편집하고 플레이스홀더(`$IMAGE_PULL_SECRET`, `$REGISTRY_URL`, `$REGISTRY_NAMESPACE`, `$IMAGE_NAME`, `$TARGET_NAMESPACE`, `$INGRESS_SUBDOMAIN`, `$INGRESS_SECRET`)를 올바른 값으로 대체하십시오. 예를 들어, 애플리케이션이 _default_ Kubernetes 네임스페이스에 배치된다고 가정해 보십시오. 

| 변수 |값 |설명 |
| -------- | ----- | ----------- |
| `$IMAGE_PULL_SECRET` |.yaml에서 주석 처리된 행 유지 | 레지스트리에 액세스하는 데 필요한 시크릿입니다.  |
| `$REGISTRY_URL` | *us.icr.io* | 이전 절에서 이미지가 빌드된 레지스트리입니다. |
| `$REGISTRY_NAMESPACE` | *secure-file-storage-namespace* | 이전 절에서 이미지가 빌드된 네임스페이스 레지스트리입니다. |
| `$IMAGE_NAME` | *secure-file-storage* | Docker 이미지의 이름입니다. |
| `$TARGET_NAMESPACE` | *default* | 앱이 푸시될 Kubernetes 네임스페이스입니다. |
| `$INGRESS_SUBDOMAIN` | *secure-file-stora-123456.us-south.containers.appdomain.cloud* | 클러스터 개요 페이지 또는 `ibmcloud ks cluster-get secure-file-storage-cluster`에서 검색하십시오. |
| `$INGRESS_SECRET` | *secure-file-stora-123456* | 클러스터 개요 페이지 또는 `ibmcloud ks cluster-get secure-file-storage-cluster`에서 검색하십시오. |

default가 아닌 Kubernetes 네임스페이스를 사용하려는 경우에는 `$IMAGE_PULL_SECRET`만 필요합니다. 이는 추가 Kubernetes 구성(예: [새 네임스페이스에 Docker 레지스트리 시크릿 작성](https://{DomainName}/docs/containers?topic=containers-images#other))을 필요로 합니다.
{: tip}

### 클러스터 배치

1. 클러스터 구성을 검색하고 KUBECONFIG 환경 변수를 설정하십시오. 
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. 애플리케이션에서 서비스 인증 정보를 얻는 데 사용하는 시크릿을 작성하십시오. 
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. {{site.data.keyword.appid_short_notm}} 서비스 인스턴스를 클러스터에 바인드하십시오. 
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
   동일한 이름의 서비스가 여럿 있는 경우에는 명령이 실패합니다. 이 경우에는 이름 대신 서비스 GUID를 전달해야 합니다. 서비스의 GUID를 찾으려면 `ibmcloud resource service-instance secure-file-storage-appid`를 사용하십시오.
   {: tip}
4. 앱을 배치하십시오. 
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## 애플리케이션 테스트

애플리케이션은 `https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/`에서 액세스할 수 있습니다. 

1. 애플리케이션의 홈 페이지로 이동하십시오. {{site.data.keyword.appid_short_notm}} 기본 로그인 페이지로 경로 재지정됩니다. 
2. 유효한 이메일 주소를 사용하여 새 계정으로 가입하십시오. 
3. 받은 편지함에서 계정 확인을 위한 이메일이 수신되기를 기다리십시오. 
4. 로그인하십시오. 
5. 업로드할 파일을 선택하십시오. **업로드**를 클릭하십시오. 
6. 파일에 대해 **공유** 조치를 사용하여 다른 사용자가 파일에 액세스할 수 있도록 공유 가능한, 사전 서명된 URL을 생성하십시오. 이 링크는 5분 후 만료되도록 설정됩니다. 

인증된 사용자는 파일을 저장할 수 있는 고유 공간을 갖습니다. 다른 사용자의 파일은 볼 수 없지만, 사전 서명된 URL을 생성하여 특정 파일에 대한 임시 액세스 권한을 부여하는 것은 가능합니다. 

애플리케이션에 대한 추가 세부사항은 [소스 코드 저장소](https://github.com/IBM-Cloud/secure-file-storage)에서 찾을 수 있습니다. 

## 보안 이벤트 검토
이제 애플리케이션 및 해당 서비스가 배치되었으므로, 이 프로세스에서 생성된 보안 이벤트를 검토할 수 있습니다. 모든 이벤트는 {{site.data.keyword.cloudaccesstrailshort}} 인스턴스에서 사용 가능하며 [그래픽 UI(Kibana), CLI 또는 API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status)를 통해 액세스할 수 있습니다. 

1. [{{site.data.keyword.Bluemix_notm}} 리소스 목록](https://{DomainName}/resources)에서 {{site.data.keyword.cloudaccesstrailshort}} 인스턴스 **secure-file-storage-activity-tracker**를 찾고 해당 대시보드를 여십시오. 
2. 기본적으로 **관리** 탭은 **영역 로그**를 표시합니다. **로그 보기** 옆에 있는 선택기를 클릭하여 **계정 로그**로 전환하십시오. 여기에는 몇 가지 이벤트가 표시됩니다. 
3. **Kibana에서 보기**를 클릭하여 전체 이벤트 뷰어를 여십시오. 
4. **검색**을 클릭하여 이벤트 세부사항을 검토하십시오. 
5. **사용 가능한 필드**에서 **action_str** 및 **initiator.name_str**을 추가하십시오. 
6. 삼각형 아이콘을 클릭하여 원하는 항목을 펼친 후 테이블 또는 JSON 형식을 선택하십시오. 

자동 새로 고치기 및 표시되는 시간 범위에 대한 설정을 변경하여 분석되는 데이터와 분석 방식을 변경할 수 있습니다.
{: tip}

## 선택사항: 사용자 정의 도메인 사용 및 네트워크 트래픽 암호화
기본적으로 애플리케이션은 하위 도메인 `containers.appdomain.cloud`의 일반 호스트 이름에서 액세스할 수 있습니다. 그러나 배치된 앱에 사용자 정의 도메인을 사용할 수도 있습니다. **https**, 암호화된 네트워크 트래픽을 사용한 액세스를 계속 지원하기 위해서는 원하는 호스트 이름에 대한 인증서 또는 와일드카드 인증서를 제공해야 합니다. 다음 절에서는 기존 인증서를 {{site.data.keyword.cloudcerts_short}}에 업로드하고 이를 클러스터에 배치합니다. 또한 사용자 정의 도메인을 사용하도록 앱 구성을 업데이트합니다. 

[Let's Encrypt](https://letsencrypt.org/)에서 인증서를 얻는 방법에 대한 예는 다음 [{{site.data.keyword.cloud}} 블로그](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)에 설명되어 있습니다.
{: tip}

1. [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)의 인스턴스 작성
   * 이름을 **secure-file-storage-certmgr**로 설정하십시오. 
   * 다른 서비스와 동일한 **위치** 및 **리소스 그룹**을 사용하십시오. 
2. **인증서 가져오기**를 클릭하여 인증서를 가져오십시오. 
   * 이름을 **SecFileStorage**로 설정하고 설명을 **e2e 보안 튜토리얼을 위한 인증서**로 설정하십시오. 
   * **찾아보기** 단추를 사용하여 인증서 파일을 업로드하십시오. 
   * **가져오기**를 클릭하여 가져오기 프로세스를 완료하십시오. 
3. 가져온 인증서에 대한 항목을 찾고 펼치십시오. 
   * 인증서 항목을 확인하십시오(예: 도메인 이름과 사용자 정의 도메인의 일치 여부). 와일드카드 인증서를 업로드한 경우에는 도메인 이름에 별표가 포함됩니다. 
   * 인증서의 **crn** 옆에 있는 **복사** 기호를 클릭하십시오. 
4. 명령행으로 전환하여 인증서 정보를 시크릿으로서 클러스터에 배치하십시오. 이전 단계에서 crn을 복사한 후에는 다음 명령을 실행하십시오. 
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <the copied crn from previous step>
   ```
   {: codeblock}
   다음 명령을 실행하여 클러스터가 해당 인증서를 인식하고 있는지 확인하십시오.
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. 파일 `secure-file-storage.yaml`을 편집하십시오. 
   * **Ingress**에 대한 섹션을 찾으십시오. 
   * 사용자 정의 도메인 관련 행을 주석 해제하고 편집하여 자신의 도메인 및 호스트 이름을 채우십시오.
   사용자 정의 도메인에 대한 CNAME 항목은 클러스터를 가리켜야 합니다. 세부사항은 문서에 있는 [사용자 정의 도메인 맵핑에 대한 안내](https://{DomainName}/docs/containers?topic=containers-ingress#private_3)를 참조하십시오.
   {: tip}
6. 배치된 항목에 구성 변경사항을 적용하십시오. 
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. 브라우저로 다시 전환하십시오. [{{site.data.keyword.Bluemix_notm}} 리소스 목록](https://{DomainName}/resources)에서 이전에 작성하여 구성한 {{site.data.keyword.appid_short}} 서비스를 찾고 해당 관리 대시보드를 실행하십시오. 
   * **ID 제공자**의 **관리**로 이동한 후 **설정**으로 이동하십시오. 
   * **웹 경로 재지정 URL 추가** 양식에서 `https://secure-file-storage.<your custom domain>/appid_callback`을 다른 URL로 추가하십시오. 
8. 이제 모든 항목의 설정이 완료되었습니다. 구성된 사용자 정의 도메인 `https://secure-file-storage.<your custom domain>`에 액세스하여 앱을 테스트하십시오. 

## 튜토리얼 확장

보안은 절대 완벽하지 않습니다. 아래 제안사항을 수행하여 애플리케이션의 보안을 강화해 보십시오. 

* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 사용을 통한 정적 및 동적 코드 스캔 수행
* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)에 대한 정책 및 규칙을 사용하여 양질의 코드만 릴리스되도록 함

## 리소스 제거
{:removeresources}

리소스를 제거하려면 배치된 컨테이너를 삭제한 후 프로비저닝된 서비스를 삭제하십시오. 

1. 배치된 컨테이너를 삭제하십시오. 
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. 배치에 대한 시크릿을 삭제하십시오. 
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. 컨테이너 레지스트리에서 Docker 이미지를 제거하십시오. 
   ```sh
   ibmcloud cr image-rm <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. [{{site.data.keyword.Bluemix_notm}} 리소스 목록](https://{DomainName}/resources)에서 이 튜토리얼을 위해 작성된 리소스를 찾으십시오. 검색 상자를 사용하고 **secure-file-storage**를 패턴으로 사용하십시오. 각 서비스 옆에 있는 컨텍스트 메뉴를 클릭하고 **서비스 삭제**를 선택하여 서비스를 삭제하십시오. {{site.data.keyword.keymanagementserviceshort}} 서비스는 키가 삭제된 후에만 제거할 수 있다는 점을 참고하십시오. 관련 대시보드로 이동하여 키를 삭제하려면 서비스 인스턴스를 클릭하십시오. 

다른 사용자와 계정을 공유하는 경우에는 항상 자신의 자원만 삭제해야 합니다.
{: tip}

## 관련 컨텐츠
{:related}

* [{{site.data.keyword.security-advisor_short}} 문서](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [Security to safeguard and monitor your cloud apps](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [{{site.data.keyword.Bluemix_notm}} 플랫폼 보안](https://{DomainName}/docs/overview?topic=overview-security#security)
* [Security in the IBM Cloud](https://www.ibm.com/cloud/security)
* [튜토리얼: 사용자, 팀, 애플리케이션 구성에 대한 우수 사례](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [Secure Apps on IBM Cloud with Wildcard Certificates](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
