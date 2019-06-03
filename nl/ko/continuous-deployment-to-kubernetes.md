---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# Kubernetes로의 지속적 배치
{: #continuous-deployment-to-kubernetes}

이 튜토리얼에서는 {{site.data.keyword.containershort_notm}}에서 실행되는 컨테이너화된 애플리케이션의 지속적 통합 및 딜리버리 파이프라인을 설정하는 프로세스를 안내합니다. 사용자는 코드의 소스 제어를 설정하고, 코드를 빌드하고, 테스트한 후 다른 배치 단계로 배치하는 방법을 학습합니다. 그 다음에는 보안 스캐너, Slack 알림, 분석과 같은 기타 서비스와의 통합을 추가합니다. 

{:shortdesc}

## 목표
{: #objectives}

* 개발 및 프로덕션 Kubernetes 클러스터를 작성합니다. 
* 스타터 애플리케이션 작성하고, 로컬로 실행하고, Git 저장소로 푸시합니다. 
* Git 저장소에 연결하고 스타터 앱을 개발/프로덕션 클러스터에 빌드 및 배치하기 위한 DevOps Delivery Pipeline을 구성합니다. 
* 보안 스캐너, Slack 알림 및 분석을 사용하기 위해 앱을 탐색하고 통합합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 {{site.data.keyword.Bluemix_notm}} 서비스를 사용합니다. 

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**주의:** 이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

![](images/solution21/Architecture.png)

1. 개인용 Git 저장소에 코드를 푸시합니다. 
2. 파이프라인이 Git의 변경사항을 발견하고 컨테이너 이미지를 빌드합니다. 
3. 레지스트리에 업로드된 컨테이너 이미지가 개발 Kubernetes 클러스터에 배치됩니다. 
4. 변경사항을 유효성 검증하고 프로덕션 클러스터에 배치합니다. 
5. 배치 활동에 대한 Slack 알림을 설정합니다. 


## 시작하기 전에
{: #prereq}

* [{{site.data.keyword.dev_cli_notm}} 설치](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - 스크립트를 실행하여 Docker, kubectl, Helm, IBM Cloud CLI 및 필수 플러그인을 설치하십시오. 
* [{{site.data.keyword.registrylong_notm}} CLI 및 레지스트리 네임스페이스를 설정하십시오](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace). 
* [Kubernetes의 기본 사항을 이해하십시오](https://kubernetes.io/docs/tutorials/kubernetes-basics/). 

## 개발 Kubernetes 클러스터 작성
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}}는 Docker와 Kubernetes 기술을 결합하여 컴퓨팅 호스트 클러스터에서의 컨테이너화된 앱 배치, 운영, 스케일링 및 모니터링을 자동화하기 위한 강력한 도구, 직관적인 사용자 경험, 기본 제공 보안 및 격리 기능을 제공합니다. 

이 튜토리얼을 완료하려면 **표준** 유형의 **유료** 클러스터를 선택해야 합니다. 사용자는 두 개의 클러스터(하나는 개발용, 하나는 프로덕션용)를 설정해야 합니다.
{: shortdesc}

1. [{{site.data.keyword.Bluemix}} 카탈로그](https://{DomainName}/containers-kubernetes/launch)에서 첫 번째 개발 Kubernetes 클러스터를 작성하십시오. 나중에 이 단계를 반복하여 프로덕션 클러스터를 작성해야 합니다. 

   사용 편의를 위해 확보하는 CPU 수, 메모리 및 작업자 노드 수와 같은 구성 세부사항을 확인하십시오.
   {:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. **클러스터 유형**을 선택한 후 **클러스터 작성**을 클릭하여 Kubernetes 클러스터를 프로비저닝하십시오. 이 튜토리얼을 수행하는 데는 2개의 **CPU**, 4**GB RAM**, 1개의 **작업자 노드**를 포함하는 가장 작은 **시스템 유형**이면 충분합니다. 다른 모든 옵션은 기본값으로 둘 수 있습니다. 
3. **클러스터** 및 **작업자 노드**의 상태를 확인하고 **준비**가 될 때까지 기다리십시오. 

**참고:** 작업자 노드가 준비될 때까지 진행하지 마십시오. 

## 스타터 애플리케이션 작성
{: #create_application}

{{site.data.keyword.containershort_notm}}는 다양한 스타터 애플리케이션을 제공하며, 이러한 스타터 애플리케이션은 `ibmcloud dev create` 명령 또는 웹 콘솔을 사용하여 작성할 수 있습니다. 이 튜토리얼에서는 웹 콘솔을 사용합니다. 스타터 애플리케이션은 필요한 모든 표준 유형, 빌드 및 구성 코드를 포함하는 애플리케이션 스타터를 생성하여 더 빨리 비즈니스 로직 코딩을 시작할 수 있도록 함으로써 개발 시간을 크게 단축시킵니다. 

1. [{{site.data.keyword.cloud_notm}} 콘솔](https://{DomainName})에서 왼쪽 메뉴를 사용하여 [웹 앱](https://{DomainName}/developer/appservice/dashboard)을 선택하십시오. 
2. **웹에서 시작** 섹션에서 **시작하기** 단추를 클릭하십시오. 
3. `Node.js Web App with Express.js` 타일을 선택한 후 `Create app`을 선택하여 Node.js 스타터 애플리케이션을 작성하십시오. 
4. **이름**에 `mynodestarter`를 입력하십시오. 그 후 **작성**을 클릭하십시오. 

## DevOps Delivery Pipeline
{: #create_devops}

1. 스타터 애플리케이션을 작성했으면 **앱 배치**에서 **클라우드에 배치** 단추를 클릭하십시오. 
2. Kubernetes 클러스터 배치 방법을 선택하고, 이전에 작성한 클러스터를 선택한 후 **작성**을 클릭하십시오. 이렇게 하면 도구 체인 및 Delivery Pipeline이 작성됩니다. ![](images/solution21/BindCluster.png)
3. 파이프라인이 작성되고 나면 **도구 체인 보기**를 클릭한 후 **Delivery Pipeline**을 클릭하여 파이프라인을 보십시오. ![](images/solution21/Delivery-pipeline.png)
4. 배치 단계가 완료되고 나면 **로그 및 히스토리 보기**를 클릭하여 로그를 보십시오. 
5. 표시된 URL에 방문하여 애플리케이션에 액세스하십시오(`http://worker-public-ip:portnumber/`). ![](images/solution21/Logs.png)
작업을 완료했습니다. 이제까지 앱 서비스 UI를 사용하여 스타터 애플리케이션을 작성하고, 애플리케이션을 빌드하여 클러스터에 배치하는 데 필요한 파이프라인을 구성했습니다. 

## 애플리케이션을 로컬에서 복제, 빌드 및 실행
{: #cloneandbuildapp}

이 절에서는 이전 절에서 작성한 스타터 앱을 사용하여 이를 로컬 시스템에 복제하고, 코드를 수정한 후 로컬에서 빌드/실행합니다.
{: shortdesc}

### 애플리케이션 복제
1. 도구 체인 개요에서 **코드**의 **Git** 타일을 선택하십시오. 저장소를 복제할 수 있는 Git 저장소 페이지로 경로 재지정됩니다. ![Hello world](images/solution21/DevOps_Toolchain.png)

2. 아직 SSH 키를 설정하지 않은 경우에는 맨 위에 알림 표시줄이 지시사항과 함께 표시됩니다. 새 표시줄의 **SSH 키** 링크를 열어 단계를 따르고, SSH 대신 HTTPS를 사용하려는 경우에는 **개인용 액세스 토큰 작성**을 클릭하여 단계를 따르십시오. 나중에 참조할 수 있도록 키 또는 토큰을 저장하십시오. 
3. SSH 또는 HTTPS를 선택한 후 git URL을 복사하십시오. 소스를 로컬 시스템에 복제하십시오. 사용자 이름에 대한 프롬프트가 표시되면 Git 사용자 이름을 제공하십시오. 비밀번호는 기존 **SSH 키** 또는 **개인용 액세스 토큰**이나 이전 단계에서 작성한 것을 사용하십시오. 

   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   {: codeblock}

4. 선택한 IDE에서 복제된 저장소를 열고 `public/index.html`로 이동하십시오. "Congratulations!"를 다른 내용으로 변경하여 코드를 업데이트하고 파일을 저장하십시오. 

### 애플리케이션을 로컬에서 빌드
Java 로컬 개발의 경우 `mvn`, Node.js 개발의 경우 `npm`을 사용하는 것과 마찬가지로 애플리케이션을 빌드하고 실행할 수 있습니다. 로컬과 클라우드에서의 일관된 실행을 보장하기 위해 Docker 이미지를 빌드하고 애플리케이션을 컨테이너에서 실행할 수도 있습니다. Docker 이미지를 빌드하려면 다음 단계를 사용하십시오.
{: shortdesc}

1. 아래 명령을 실행하여 로컬 Docker 엔진이 시작되어 있는지 확인하십시오. 
   ```
   docker ps
   ```
   {: codeblock}
2. 복제하여 생성된 프로젝트 디렉토리로 이동하십시오. 
   ```
   cd <project name>
   ```
   {: codeblock}
3. 애플리케이션을 로컬에서 빌드하십시오. 
   ```
   ibmcloud dev build
   ```
   {: codeblock}

   모든 애플리케이션 종속 항목이 다운로드되며 애플리케이션 및 모든 필수 환경을 포함하는 Docker 이미지가 빌드되므로 이 작업이 완료되려면 몇 분 정도 소요됩니다. 

### 애플리케이션을 로컬에서 실행

1. 컨테이너를 실행하십시오. 
   ```
   ibmcloud dev run
   ```
   {: codeblock}

   이는 사용자의 로컬 Docker 엔진을 사용하여 이전 단계에서 빌드한 Docker 이미지를 실행합니다. 
2. 컨테이너가 시작되고 나면 http://localhost:3000/으로 이동하십시오.
   ![](images/solution21/node_starter_localhost.png)

## 애플리케이션을 Git 저장소에 푸시

이 절에서는 변경사항을 Git 저장소에 커미트합니다. 파이프라인은 커미트를 감지하고 변경사항을 클러스터에 자동으로 푸시합니다. 
1. 터미널 창에서, 현재 위치가 복제한 저장소 내인지 확인하십시오. 
2. 간단한 세 단계(추가, 커미트, 푸시)로 저장소에 변경사항을 푸시하십시오. 
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
   {: codeblock}

3. 이전에 작성한 도구 체인으로 이동하여 **Delivery Pipeline** 타일을 클릭하십시오. 
4. **빌드** 및 **배치** 단계가 표시되는지 확인하십시오.
   ![](images/solution21/Delivery-pipeline.png)
5. **배치** 단계가 완료될 때까지 기다리십시오. 
6. 마지막 실행 결과의 애플리케이션 **url**을 클릭하여 변경사항을 실시간으로 보십시오. 

애플리케이션이 업데이트되지 않는 경우에는 파이프라인의 배치 및 빌드 단계에 대한 로그를 확인하십시오. 

## Vulnerability Advisor를 사용한 보안
{: #vulnerability_advisor}

이 단계에서는 [Vulnerability Advisor](https://{DomainName}/docs/services/va?topic=va-va_index#va_index)에 대해 알아봅니다. Vulnerability Advisor는 배치 전에 컨테이너 이미지의 보안 상태를 확인하는 데 사용되며, 실행 중인 컨테이너의 상태 또한 확인합니다. 

1. 이전에 작성한 도구 체인으로 이동하여 **Delivery Pipeline** 타일을 클릭하십시오. 
1. **추가 단계**를 클릭하고 MyStage를 **유효성 검증 단계**로 변경한 후 작업 > **작업 추가**를 클릭하십시오. 

   1. 작업 유형으로 **테스트**를 선택하고 상자에서 **테스트**를 **Vulnerability Advisor**로 변경하십시오. 
   1. 테스터 유형에서 **Vulnerability Advisor**를 선택하십시오. 다른 모든 필드는 자동으로 채워집니다.
      컨테이너 레지스트리 네임스페이스는 이 도구 체인의 **빌드 단계**에서 언급된 것과 동일해야 합니다.
      {:tip}
   1. **테스트 스크립트** 섹션을 편집하여 마지막 행에 있는 `SAFE\ to\ deploy`를 `NO\ ISSUES`로 대체하십시오. 
   1. 단계를 저장하십시오. 
1. **유효성 검증 단계**를 중간으로 끌어 와 **유효성 검증 단계**에서 **실행** ![](images/solution21/run.png)을 클릭하십시오. **유효성 검증 단계**가 실패하는 것을 볼 수 있습니다. 

   ![](images/solution21/toolchain.png)

1. **로그 및 히스토리 보기**를 클릭하여 취약성 평가를 보십시오. 로그의 마지막에 다음과 같은 내용이 있습니다. 

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   모든 스캔된 저장소에 대한 자세한 취약성 평가는 [여기](https://{DomainName}/containers-kubernetes/registry/private)에서 볼 수 있습니다.
   {:tip}

   취약성 스캔에 3분 이상 소요되는 경우에는 이미지가 *스캔되지 않음*이라는 메시지와 함께 단계가 실패할 수 있습니다. 이 제한시간은 작업 스크립트를 편집하여 스캔 결과를 기다리는 반복 수를 늘림으로써 변경할 수 있습니다.
   {:tip}

1. 정정 조치에 따라 취약성을 수정하십시오. IDE에서 복제된 저장소를 열거나, Eclipse Orion 웹 IDE 타일을 선택한 후 `Dockerfile`을 열고 `EXPOSE 3000` 뒤에 아래 명령을 추가하십시오. 
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. 변경사항을 커미트하고 푸시하십시오. 이렇게 하면 도구 체인이 트리거되고 **유효성 검증 단계**가 수정됩니다. 

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## 프로덕션 Kubernetes 클러스터 작성

{: #deploytoproduction}

이 절에서는 Kubernetes 애플리케이션을 개발 및 프로덕션 환경에 각각 배치하여 배치 파이프라인을 완료합니다. 가능하면 개발 환경에 대해서는 자동 배치를 설정하고 프로덕션 환경에 대해서는 수동 배치를 설정할 것입니다. 이를 수행하기 전에, 먼저 이를 달성할 수 있는 두 가지 방법을 살펴보도록 합시다. 개발 환경과 프로덕션 환경에 하나의 클러스터를 사용할 수도 있습니다. 그러나 권장사항은 별도의 두 클러스터(하나는 개발용, 하나는 프로덕션용)를 사용하는 것입니다.
여기서는 프로덕션용 두 번째 클러스터를 설정하는 작업에 대해 알아보겠습니다.
{: shortdesc}

1. [개발 Kubernetes 클러스터 작성](#create_kube_cluster) 절의 지시사항에 따라 새 클러스터를 작성하십시오. 이 클러스터의 이름을 `prod-cluster`로 지정하십시오. 
2. 이전에 작성한 도구 체인으로 이동하여 **Delivery Pipeline** 타일을 클릭하십시오. 
3. 이름 **배치 단계**를 `Deploy dev`로 바꾸십시오. 설정 아이콘을 클릭한 후 **단계 구성**을 클릭하여 이를 수행할 수 있습니다.
   ![](images/solution21/deploy_stage.png)
4. **Deploy dev** 단계를 복제(설정 아이콘 > 단계 복제)하여 복제된 단계의 이름을 `Deploy prod`로 지정하십시오. 
5. **단계 트리거**를 `Run jobs only when this stage is run manually`로 변경하십시오. ![](images/solution21/prod-stage.png)
6. **작업** 탭에서 클러스터 이름을 새로 작성된 클러스터로 변경한 후 단계를 **저장**하십시오. 
7. 전체 배치 설정이 완료되었습니다. 개발에서 프로덕션으로 배치하려면 `Deploy prod` 단계를 수동으로 실행하여 프로덕션에 배치해야 합니다. ![](images/solution21/full-deploy.png)

프로덕션 클러스터를 작성하고 프로덕션 클러스터로 업데이트를 수동으로 푸시하는 파이프라인을 구성하는 작업을 완료했습니다. 이는 단위 테스트 및 통합 테스트를 파이프라인의 일부로 포함시키는 고급 시나리오의 프로세스 단계를 간소화한 것입니다. 

## Slack 알림 설정
{: #setup_slack}

1. 돌아가 [도구 체인](https://{DomainName}/devops/toolchains)의 목록을 보고 자신의 도구 체인을 선택한 후 **도구 추가**를 클릭하십시오. 
2. 검색 상자에서 slack을 검색하거나 아래로 스크롤하여 **Slack**을 찾으십시오. 클릭하여 구성 페이지를 보십시오.
    ![](images/solution21/configure_slack.png)
3. **Slack 웹훅**의 경우에는 이 [링크](https://my.slack.com/services/new/incoming-webhook/)의 단계를 따르십시오. 자신의 Slack 인증 정보로 로그인하여 기존 채널 이름을 제공하거나 새 채널 이름을 작성해야 합니다. 
4. 수신 웹훅 통합이 추가되고 나면 **웹훅 URL**을 복사하여 **Slack 웹훅**에 붙여넣으십시오. 
5. Slack 채널은 위에서 웹훅 통합을 작성할 때 제공한 채널 이름입니다. 
6. **Slack 팀 이름**은 team-name.slack.com의 team-name(첫 번째 부분)입니다. 예를 들어, kube.slack.com에서 팀 이름은 kube입니다. 
7. **통합 작성**을 클릭하십시오. 도구 체인에 새 타일이 추가됩니다.
    ![](images/solution21/toolchain_slack.png)
8. 지금부터는 도구 체인이 실행될 때마다 작성한 채널에 Slack 알림이 표시됩니다.
    ![](images/solution21/slack_channel.png)

## 리소스 제거
{: #removeresources}

이 단계에서는 리소스를 정리하여 위에서 작성한 리소스를 제거합니다. 

- Git 저장소를 삭제하십시오. 
- 도구 체인을 삭제하십시오. 
- 두 개의 클러스터를 삭제하십시오. 
- Slack 채널을 삭제하십시오. 

## 튜토리얼 확장
{: #expandTutorial}

더 자세히 알아보시겠습니까? 다음에 수행할 수 있는 작업에 대한 몇 가지 제안사항은 다음과 같습니다. 

- [LogDNA 및 Sysdig를 사용하여 로그를 분석하고 애플리케이션 상태를 모니터링합니다](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis). 
- 테스트 환경을 추가하고 이를 세 번째 클러스터에 배치합니다. 
- [여러 위치](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)에 프로덕션 클러스터를 배치합니다. 
- [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)를 사용하여 추가 품질 제어 및 분석 기능으로 파이프라인을 개선합니다. 

## 관련 컨텐츠
{: #related}

* 엔드 투 엔드 Kubernetes 솔루션 안내서인 [VM 기반 앱을 Kubernetes로 이동](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes)
* IBM Cloud 컨테이너 서비스에 대한 [보안](https://{DomainName}/docs/containers?topic=containers-security#cluster)
* 도구 체인 [통합](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations)
* [LogDNA 및 Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)를 사용한 로그 분석 및 애플리케이션 상태 모니터링


