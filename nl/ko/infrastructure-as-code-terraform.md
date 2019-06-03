---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-23"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Terraform을 사용한 LAMP 스택 배치
{: #infrastructure-as-code-terraform}

[Terraform](https://www.terraform.io/)은 안전하고 예측 가능하게 인프라를 작성하고, 변경하고, 개선할 수 있게 해 줍니다. 이는 API를 팀 구성원 간에 공유할 수 있으며 코드로 취급하여 편집, 검토 및 버전화할 수 있는 선언적 구성 파일로 코드화하는 오픈 소스 도구입니다. 

이 튜토리얼에서는 샘플 구성을 사용하여 **LAMP** 스택이라고 하는 **L**inux Virtual Server(**A**pache 웹 서버, **M**ySQL, **P**HP 서버 포함)를 프로비저닝합니다. 그 후에는 구성을 업데이트하여 {{site.data.keyword.cos_full_notm}} 서비스를 추가하고 리소스를 스케일링하여 환경을 조정합니다(메모리, CPU 및 디스크 크기). 마지막으로 구성에 의해 작성된 모든 리소스를 삭제하여 작업을 완료합니다. 

## 목표
{: #objectives}

* Terraform 및 {{site.data.keyword.Bluemix_notm}} Provider for Terraform을 구성합니다. 
* Terraform을 사용하여 LAMP 스택 구성을 작성, 업데이트 및 스케일링하고, 마지막으로 영구 삭제합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

  ![아키텍처 다이어그램](images/solution10/architecture-2.png)
</p>

1. LAMP 스택 구성을 나타내기 위해 Terraform 파일 세트가 작성됩니다. 
1. 구성 디렉토리에서 `terraform`이 호출됩니다. 
1. `terraform`이 {{site.data.keyword.cloud_notm}} API를 호출하여 리소스를 프로비저닝합니다. 

## 시작하기 전에
{: #prereqs}

인프라 마스터 사용자에게 문의하여 다음 권한을 얻으십시오. 
- 네트워크(**공용 및 사설 네트워크 업링크** 추가를 위한)
- API 키

## 선행 조건

{: #prereq}

[설치 프로그램](https://www.terraform.io/intro/getting-started/install.html)을 통해, 또는 macOS에서 [Homebrew](https://brew.sh/)를 사용해 `brew install terraform` 명령을 실행하여 **Terraform**을 설치하십시오. 

**Windows**에서 Terraform 설정을 완료하려면 다음 단계를 따르십시오. 
   1. 다운로드한 zip 파일에서 `C:\terraform`(`terraform` 폴더를 작성해야 함)으로 파일을 복사하십시오. 
   2. 관리자로서 명령 프롬프트를 열어 Terraform 바이너리를 사용할 수 있도록 PATH를 설정하십시오. 

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## {{site.data.keyword.Bluemix_notm}} Provider for Terraform 구성
{: #setup}

Terraform은 다중 클라우드 접근법을 지원하기 위해 여러 제공자와 협력합니다. 제공자는 API 상호작용을 이해하고 리소스를 노출시켜야 합니다. 이 절에서는 {{site.data.keyword.Bluemix_notm}} Provider의 위치를 지정하기 위해 CLI를 구성합니다. 

1. 터미널 또는 명령 프롬프트 창에서 `terraform`을 실행하여 Terraform 설치를 확인하십시오. `Common commands`의 목록이 표시되어야 합니다. 

  ```
  terraform
  ```

2. 자신의 시스템에 해당되는 [{{site.data.keyword.Bluemix_notm}} Provider](https://github.com/IBM-Cloud/terraform-provider-ibm/releases) 플러그인을 다운로드하여 아카이브의 압축을 푸십시오. `terraform-provider-ibm_VERSION` 2진 플러그 파일을 볼 수 있습니다. 

3. Windows가 아닌 시스템의 경우에는 사용자의 홈 디렉토리에 `.terraform.d/plugins` 디렉토리를 작성하고 2진 파일을 여기에 배치하십시오. 다음 명령을 참조하십시오. 

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    **Windows**에서는 이 파일을 사용자의 "Application Data" 디렉토리에 있는 `terraform.d/plugins`에 배치해야 합니다. 

  - 명령 프롬프트에서 아래 명령을 실행하십시오([제공자 구성](https://www.terraform.io/docs/configuration/providers.html)). 
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - **Windows PowerShell**(시작 키 + R > Powershell)을 실행하고 아래 명령을 실행하여 `terraform.rc` 파일을 작성하십시오. 
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   첫 번째 프롬프트에서 아래 컨텐츠를 입력하십시오.
   ```
    # ~/.terraformrc
    providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        PATH_TO_YOUR_APPDATA_PLUGINS는 슬래시(/)를 포함하는 절대 경로여야 합니다. 예를 들면 `C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe`와 같습니다.
        {: tip}

  - Enter를 눌러 프롬프트를 종료하십시오. 

## Terraform 구성 준비

{: #terraformconfig}

이 절에서는 {{site.data.keyword.Bluemix_notm}}에서 제공하는 샘플 Terraform 구성을 사용하여 Terraform 구성의 기본을 학습합니다. 

1. https://github.com/IBM-Cloud/LAMP-terraform-ibm에 방문하여 자신의 계정으로 사본을 **분기**하십시오. 
2. 분기를 로컬로 복제하십시오. 
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. 구성 파일을 살펴보십시오. 
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml) - 서버 설치 구성을 포함합니다. 이 파일이 서버에 설치할 항목을 지정하기 위해 서버 설치와 관련된 모든 스크립트를 추가할 수 있는 파일입니다. 이 파일에 삽입된 `phpinfo();`를 참조하십시오. 
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf) - 제공자 사용자 이름 및 API 키가 필요한 경우 제공자와 관련된 변수를 포함합니다. 
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) - 지정된 변수를 사용하여 VM을 배치하기 위한 서버 구성을 포함합니다. 
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) - **SoftLayer** 사용자 이름 및 API 키, {{site.data.keyword.Bluemix_notm}} API 키 및 사용자의 영역/조직 이름을 포함합니다. 이러한 인증 정보는 서버를 배치할 때마다 명령행에서 이러한 인증 정보를 다시 입력할 필요가 없도록 하는 우수 사례를 구현하기 위해 이 파일에 추가될 수 있습니다. 참고: 이 파일을 자신의 인증 정보를 사용하여 공개하지 마십시오. 
4. 원하는 IDE에서 [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) 파일을 열고 자신의 **공개 SSH** 키를 추가하여 파일을 편집하십시오. 이는 이 구성으로 작성된 VM에 액세스하는 데 사용됩니다. SSH 키 작성에 대한 정보를 얻으려면 [이 링크](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html)의 지시사항을 따르십시오. 공개 키를 클립보드에 복사하려는 경우에는 터미널에서 아래 명령을 실행할 수 있습니다. 

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     이 명령은 SSH 키를 클립보드에 복사하며, 사용자는 그 후 [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf)에서 69행 정도에 있는 `ssh_key` 기본 변수에 이를 전달할 수 있습니다. 

    **Windows**에서는 [Git Bash](https://git-scm.com/download/win)를 다운로드하고 설치하여 실행한 후 아래 명령을 실행하여 공개 SSH 키를 클립보드에 복사하십시오. 

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. 자신의 IDE를 사용하여 [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) 파일을 열고 나열된 모든 인증 정보를 추가하여 이 파일을 수정하십시오. 이 파일에 이러한 인증 정보를 추가하면 terraform apply를 실행할 때마다 이 인증 정보를 다시 입력할 필요가 없습니다. 이 튜토리얼의 나머지 부분을 완료하려면 [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) 파일에 나열된 다섯 개의 인증 정보를 모두 추가해야 합니다. 

## Terraform 구성으로부터 LAMP 스택 서버 작성
{: #Createserver}
이 절에서는 Terraform 구성 샘플로부터 LAMP 스택 서버를 작성하는 방법을 학습합니다. 이 구성은 가상 머신 인스턴스를 프로비저닝하고 **A**pache, **M**ySQL(**M**ariaDB) 및 **P**HP를 이 인스턴스에 설치하는 데 사용됩니다. 

1. 복제한 저장소의 폴더로 이동하십시오. 
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. Terraform 구성을 초기화하십시오. 이는 `terraform-provider-ibm_VERSION` 플러그인 또한 설치합니다. 
   ````bash
    terraform init
   ````
   {: pre}
3. Terraform 구성을 적용하십시오. 이는 구성에 정의된 리소스를 작성합니다. 
   ```
    terraform apply
   ```
   {: pre}
   아래와 같은 출력이 표시됩니다. ![소스 제어 URL](images/solution10/created.png)
4. 그 다음에는 [인프라 디바이스 목록](https://{DomainName}/classic/devices)으로 이동하여 서버가 작성되었는지 확인하십시오. ![소스 제어 URL](images/solution10/configuration.png)

## {{site.data.keyword.cos_full_notm}} 서비스 추가 및 리소스 스케일링

{: #modify}

이 절에서는 Virtual Server 리소스를 스케일링하고 인프라 환경에 [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) 서비스를 추가하는 방법을 살펴봅니다. 

1. `vm.tf` 파일을 편집하여 다음 항목을 증가시키고 파일을 저장하십시오. 
 - CPU 코어 수를 4개로 증가
 - RAM(메모리)을 4096으로 증가
 - 디스크 크기를 100GB로 증가

2. 그 다음 새 [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) 서비스를 추가하십시오. 이를 수행하기 위해 새 파일을 작성하고 이름을 **ibm-cloud-object-storage.tf**로 지정하십시오. 새로 작성된 파일에 아래 코드 스니펫을 추가하십시오. 아래 코드 스니펫은 조직 이름 및 영역 이름에 대한 변수 이름을 작성한 후 이들 두 변수 이름을 사용하여 서비스를 작성하는 데 필요한 영역 GUID를 검색합니다. 이는 {{site.data.keyword.cos_full_notm}} 서비스 이름을 `lamp_objectstorage`로 설정하며, 그 후 사용자에게는 영역 GUID, 서비스의 완전한 이름 및 플랜 유형이 필요합니다. 어차피 종량과금제를 사용하므로 아래 코드는 프리미엄 플랜을 작성합니다. Lite 플랜을 사용할 수도 있지만 Lite 플랜은 계정당 1개의 서비스로 제한된다는 점을 참고하십시오. 

   ```
   variable "org_name" {
     description = "Enter your IBM Cloud org name, you can get your org name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   variable "space_name" {
     description = "Enter your IBM Cloud space name, you can get your space name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   data "ibm_space" "space" {
     space = "${var.space_name}"
     org   = "${var.org_name}"
   }

   # a cloud object storage
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # you can only have one Lite plan per account so let's use the Premium - it is pay-as-you-go
     plan = "Premium"
   }
   ```
   {: pre}
   **참고:** 나중에 {{site.data.keyword.cos_full_notm}}가 성공적으로 작성되었는지 확인하기 위해 로그에서 레이블 "lamp_objectstorage"를 검색할 것입니다. 

3. 다음 명령을 실행하여 Terraform 구성을 초기화하십시오. 

   ```bash
    terraform init
   ```
   {: pre}

4. 다음 명령을 실행하여 Terraform 변경사항을 적용하십시오. 
   ```bash
    terraform apply
   ```
   {: pre}
   **참고:** terraform apply 명령을 성공적으로 실행하고 나면 새 `terraform.tfstate` 파일이 디렉토리에 추가된 것을 볼 수 있습니다. 이 파일은 구성에 마지막으로 적용된 수정사항과 미래에 적용될 수 있는 수정사항을 추적하기 위한 전체 배치 확인을 포함합니다. 이 파일이 제거되었거나 유실되면 Terraform 배치 구성을 잃게 됩니다. 

## VM 및 {{site.data.keyword.cos_short}} 확인
{: #verifyvm}

이 절에서는 VM 및 {{site.data.keyword.cos_short}}를 확인하여 성공적으로 작성되었는지 확인합니다. 

**VM 확인**

1. 왼쪽 메뉴에서 **인프라**를 클릭하여 Virtual Server 디바이스의 목록을 보십시오. 
2. **디바이스** -> **디바이스 목록**을 클릭하여 작성한 서버를 찾으십시오. 자신의 서버 디바이스가 나열된 것을 볼 수 있습니다. 
3. 서버를 클릭하여 서버 구성에 대한 자세한 정보를 보십시오. 아래 스크린샷을 보면 서버가 성공적으로 작성된 것을 볼 수 있습니다. ![Source Control URL](images/solution10/configuration.png)
4. 그 다음에는 웹 브라우저에서 서버를 테스트하십시오. 웹 브라우저에서 서버 공인 IP 주소를 여십시오. 아래와 같은 서버 기본 설치 페이지를 볼 수 있습니다. ![소스 제어 URL](images/solution10/LAMP.png)


**{{site.data.keyword.cos_full_notm}}** 확인

1. **{{site.data.keyword.Bluemix_notm}} 대시보드**에서 사용자를 위한 {{site.data.keyword.cos_full_notm}} 서비스 인스턴스가 작성되어 사용할 준비가 된 것을 볼 수 있습니다. ![object-storage](images/solution10/ibm-cloud-object-storage.png)

   {{site.data.keyword.cos_full_notm}}에 대한 자세한 정보는 [여기](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)에 있습니다. 

## 리소스 제거
{: #deleteresources}

다음 명령을 사용하여 리소스를 삭제하십시오. 
   ```bash
   terraform destroy
   ```
   {: pre}

**참고:** 리소스를 삭제하려면 인프라 관리자 권한이 필요합니다. 관리자 수퍼유저 계정이 없는 경우에는 인프라 대시보드를 사용하여 리소스를 취소하도록 요청하십시오. 인프라 대시보드의 디바이스에서 디바이스 취소를 요청할 수 있습니다. ![object-storage](images/solution10/rm.png)

## 관련 컨텐츠

- [Terraform](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [{{site.data.keyword.cos_full_notm}} 및 CDN을 사용한 정적 파일 전달 가속화](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

