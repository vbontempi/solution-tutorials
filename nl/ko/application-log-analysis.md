---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# LogDNA 및 Sysdig를 사용한 로그 분석 및 애플리케이션 상태 모니터링
{: #application-log-analysis}

이 튜토리얼에서는 [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/) 서비스를 사용하여 {{site.data.keyword.Bluemix_notm}}에 배치된 Kubernetes 애플리케이션의 로그를 구성하고 여기에 액세스하는 방법을 보여줍니다. 사용자는 {{site.data.keyword.containerlong_notm}}에 프로비저닝된 클러스터에 Python 애플리케이션을 배치하고, LogDNA 에이전트를 구성하고, 다양한 레벨의 애플리케이션 로그, 액세스 작업자 로그, 팟(Pod) 로그 또는 네트워크 로그를 생성합니다. 그 후에는 {{site.data.keyword.la_short}} 웹 UI를 통해 이러한 로그를 검색하고, 필터링하고 시각화합니다. 

또한 [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/) 서비스를 설정하고 애플리케이션 및 {{site.data.keyword.containerlong_notm}} 클러스터의 성능 및 상태를 모니터하도록 Sysdig 에이전트를 구성합니다.
{:shortdesc}

## 목표
{: #objectives}
* 로그 항목을 생성하기 위해 애플리케이션을 Kubernetes 클러스터에 배치합니다. 
* 문제점을 해결하고 사전에 방지하기 위해 다양한 유형의 로그에 액세스하여 이를 분석합니다. 
* 앱과 앱을 실행하는 클러스터의 성능 및 상태에 대한 운영 관련 가시성을 확보합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

  ![](images/solution12/Architecture.png)

1. 사용자가 애플리케이션에 연결하여 로그 항목을 생성합니다. 
1. 애플리케이션은 {{site.data.keyword.registryshort_notm}}에 저장된 이미지의 Kubernetes 클러스터에서 실행됩니다. 
1. 사용자가 애플리케이션 및 클러스터 레벨 로그에 액세스하기 위해 {{site.data.keyword.la_full_notm}} 서비스 에이전트를 구성합니다. 
1. 사용자가 {{site.data.keyword.containerlong_notm}} 클러스터와 해당 클러스터에 배치된 앱의 상태 및 성능을 모니터하기 위해 {{site.data.keyword.mon_full_notm}} 서비스 에이전트를 구성합니다. 

## 선행 조건
{: #prereq}

* [{{site.data.keyword.dev_cli_notm}} 설치](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - 스크립트를 실행하여 Docker, kubectl, Helm, IBM Cloud CLI 및 필수 플러그인을 설치하십시오. 
* [{{site.data.keyword.registrylong_notm}} CLI 및 레지스트리 네임스페이스를 설정하십시오](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace). 
* [사용자에게 LogDNA에서 로그를 볼 수 있도록 권한을 부여하십시오](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna). 
* [사용자에게 Sysdig에서 메트릭을 볼 수 있도록 권한을 부여하십시오](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig). 

## Kubernetes 클러스터 작성
{: #create_cluster}

{{site.data.keyword.containershort_notm}}에서는 Kubernetes 클러스터에서 실행되는 Docker 컨테이너에 고가용성 앱을 배치하는 데 필요한 환경을 제공합니다. 

기존 **표준** 클러스터가 있으며 이 튜토리얼을 위해 이를 재사용하려는 경우에는 이 절을 건너뛰십시오.
{: tip}

1. [{{site.data.keyword.Bluemix}} 카탈로그](https://{DomainName}/kubernetes/catalog/cluster/create)로부터 **새 클러스터**를 작성하고 **표준** 클러스터를 선택하십시오.
   **무료** 클러스터에 대해서는 로그 전달이 사용으로 설정되지 *않습니다*.
   {:tip}
1. 리소스 그룹 및 지역을 선택하십시오. 
1. 편의를 위해 이 튜토리얼에서는 모두 `mycluster`라는 이름을 사용하십시오. 
1. **작업자 구역**을 선택하고, 2개의 **CPU**와 4**GB RAM**을 포함하는 가장 작은 **시스템 유형**을 선택하십시오. 이 튜토리얼을 수행하는 데는 이 유형으로 충분합니다. 
1. **작업자 노드**에 대해 1을 선택하고 다른 모든 옵션은 기본값으로 설정하십시오. **클러스터 작성**을 클릭하십시오. 
1. **클러스터** 및 **작업자 노드**의 상태를 확인하고 **준비**가 될 때까지 기다리십시오. 

## {{site.data.keyword.la_short}} 인스턴스 프로비저닝
{: #provision_logna_instance}

{{site.data.keyword.Bluemix_notm}}의 {{site.data.keyword.containerlong_notm}} 클러스터에 배치된 애플리케이션은 로그와 같은 일정 레벨의 진단 출력을 생성합니다. 개발자 또는 운영자는 문제점을 해결하고 사전에 방지하기 위해 작업자 로그, 팟(Pod) 로그, 앱 로그 또는 네트워크 로그와 같은 다양한 유형의 로그에 액세스하여 이를 분석하고자 할 수 있습니다. 

{{site.data.keyword.la_short}} 서비스를 사용하면 다양한 소스로부터 로그를 수집하여 필요한 만큼 보유할 수 있습니다. 이는 필요한 경우, 그리고 더 복잡한 상황을 해결해야 하는 경우를 위해 "큰 그림"을 분석할 수 있게 해 줍니다. 

{{site.data.keyword.la_short}} 서비스를 프로비저닝하려면 다음 작업을 수행하십시오. 

1. [관찰 가능성](https://{DomainName}/observe/) 페이지로 이동하여 **로깅**에서 **인스턴스 작성**을 클릭하십시오. 
1. 고유 **서비스 이름**을 제공하십시오. 
1. 지역/위치를 선택하고 리소스 그룹을 선택하십시오. 
1. 플랜으로 **7일 로그 검색**을 선택하고 **작성**을 클릭하십시오. 

이 서비스는 로그 데이터가 IBM Cloud에서 호스팅되는 중앙 집중식 로그 관리 시스템을 제공합니다. 

## Kubernetes 앱의 배치 및 구성과 로그 전달
{: #deploy_configure_kubernetes_app}

[이 GitHub 저장소](https://github.com/IBM-Cloud/application-log-analysis)에는 로깅 앱을 위한 즉시 실행 가능한 코드가 있습니다. 이 애플리케이션은 널리 사용되는 Python 서버 측 웹 프레임워크인 [Django](https://www.djangoproject.com/)로 작성되었습니다. 저장소를 복제하거나 다운로드한 후 앱을 {{site.data.keyword.Bluemix_notm}}의 {{site.data.keyword.containershort_notm}}에 배치하십시오. 

### Python 애플리케이션 배치

터미널에서 다음 작업을 수행하십시오. 
1. GitHub 저장소를 복제하십시오. 
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. 애플리케이션 디렉토리로 변경하십시오. 
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile)을 사용하여 {{site.data.keyword.registryshort_notm}}에 Docker 이미지를 빌드하십시오. 
   - `ibmcloud cr info`로 us.icr.io 또는 uk.icr.io와 같은 **컨테이너 레지스트리**를 찾으십시오. 
   - 컨테이너 이미지를 저장할 네임스페이스를 작성하십시오.
      ```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - `<CONTAINER_REGISTRY>`를 자신의 컨테이너 레지스트리 값으로 대체하고 **app-log-analysis**를 이미지 이름으로 사용하십시오.
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - `app-log-analysis.yaml` 파일의 **이미지** 값을 이미지 태그 `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest`로 대체하십시오. 
1. 아래 명령을 실행하여 클러스터 구성을 검색하고 `KUBECONFIG` 환경 변수를 설정하십시오. 
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. 앱을 배치하십시오. 
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. 애플리케이션에 액세스하려면 작업자 노드의 `public IP`와 `NodePort`가 필요합니다. 
   - 공인 IP를 얻으려면 다음 명령을 실행하십시오.
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - 5자리 숫자인 NodePort를 얻으려면 아래 명령을 실행하십시오.
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   이제 `http://worker-ip-address:portnumber`에서 애플리케이션에 액세스할 수 있습니다. 

### LogDNA 인스턴스에 로그를 전송하도록 클러스터 구성

로그를 {{site.data.keyword.la_full_notm}} 인스턴스에 전송하도록 Kubernetes 클러스터를 구성하려면 클러스터의 각 노드에 *logdna-agent* 팟(Pod)을 설치해야 합니다. LogDNA 에이전트는 설치된 팟(Pod)에서 로그 파일을 읽고 로그 데이터를 LogDNA 인스턴스에 전달합니다. 

1. [관찰 가능성](https://{DomainName}/observe/) 페이지로 이동하여 **로깅**을 클릭하십시오. 
1. 이전에 작성한 서비스 옆에 있는 **로그 리소스 편집**을 클릭하고 **Kubernetes**를 선택하십시오. 
1. `KUBECONFIG` 환경 변수를 설정한 터미널에서 첫 번째 명령을 복사하고 실행하여 서비스 인스턴스에 대한 LogDNA 수집 키로 Kubernetes 시크릿을 작성하십시오. 
1. 두 번째 명령을 복사하고 실행하여 Kubernetes 클러스터의 모든 작업자 노드에 LogDNA 에이전트를 배치하십시오. LogDNA 에이전트는 확장자가 **.log인 로그와 팟(Pod)의 /var/log* 디렉토리에 저장된 *확장자가 없는 파일*을 수집합니다. 기본적으로 로그는 kube-system을 포함한 모든 네임스페이스에서 수집되어 자동으로 {{site.data.keyword.la_full_notm}} 서비스에 전달됩니다. 
1. 로그 소스를 구성한 후에는 **LogDNA 보기**를 클릭하여 LogDNA UI를 실행하십시오. 로그가 표시되기까지는 몇 분 정도 소요될 수 있습니다. 

## 애플리케이션 로그 생성 및 액세스
{: generate_application_logs}

이 절에서는 애플리케이션 로그를 생성하여 LogDNA에서 검토합니다. 

### 애플리케이션 로그 생성

이전 단계에서 배치된 애플리케이션은 선택된 로그 레벨에서 메시지를 로그할 수 있게 해 줍니다. 사용 가능한 로그 레벨은 **치명적**, **오류**, **경고**, **정보** 및 **디버그**입니다. 애플리케이션의 로깅 인프라는 설정된 레벨 이상의 로그 항목만 전달되도록 구성되어 있습니다. 처음에는 로거 레벨이 **경고**로 설정되어 있습니다. 따라서 서버 설정이 **경고**인 경우 **정보**에서 로그된 메시지는 진단 출력에 표시되지 않습니다. 

[**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py) 파일의 코드를 살펴보십시오. 이 코드는 **print**문과 **logger** 함수에 대한 호출을 포함합니다. 출력된 메시지는 **stdout** 스트림(일반 출력, 애플리케이션 콘솔/터미널)에 기록되고 로거 메시지는 **stderr** 스트림(오류 로그)에 표시됩니다. 

1. 웹 앱(`http://worker-ip-address:portnumber`)에 방문하십시오. 
1. 다양한 레벨에서 메시지를 제출하여 몇 개의 로그 항목을 생성하십시오. UI를 사용하면 서버 로그 레벨의 로거 설정도 변경할 수 있습니다. 중간에 서버 측 로그 레벨도 변경하여 데이터에 변화를 주십시오. 예를 들면, "500 internal server error"를 **오류**로 로그하거나 "This is my first log entry"를 **정보**로 로그할 수 있습니다. 

### 애플리케이션 로그 액세스

필터를 사용하여 LogDNA UI에서 특정 애플리케이션 로그에 액세스할 수 있습니다. 

1. 맨 위 표시줄에서 **모든 앱**을 클릭하십시오. 
1. 컨테이너에서 **app-log-analysis**를 선택하십시오. 모든 레벨의 애플리케이션 로그를 포함하는 저장되지 않은 새 보기가 표시됩니다. 
1. 특정 로그 레벨의 로그를 보려면 **모든 레벨**을 클릭하고 오류, 정보, 경고 등과 같은 여러 레벨을 선택하십시오. 

## 로그 검색 및 필터링
{: #search_filter_logs}

{{site.data.keyword.la_short}} UI는 기본적으로 모든 사용 가능한 로그 항목(모두)을 표시합니다. 가장 최신 항목은 자동 새로 고치기를 통해 맨 아래에 표시됩니다.
이 절에서는 표시되는 항목 및 수량을 수정하고 이를 나중에 사용할 수 있도록 **보기**로 저장합니다. 

### 로그 검색

1. LogDNA UI의 페이지 맨 아래에 있는 **검색** 입력 상자에서는 다음 작업을 수행할 수 있습니다. 
   - **"This is my first log entry"** 또는 **500 internal server error**와 같은 특정 텍스트를 포함하는 행을 검색합니다. 
   - `level:info`(여기서 level은 문자열 값을 허용하는 필드)를 입력하여 특정 로그 레벨을 검색합니다. 

   더 많은 검색 필드 및 도움말을 보려면 검색 입력 상자 옆에 있는 구문 도움말 아이콘을 클릭하십시오.
   {:tip}
1. 특정 시간 범위로 이동하려면 **시간 범위로 이동** 입력 상자에 **5분 전**을 입력하십시오. 보존 기간 내의 다른 시간 형식을 찾아보려면 입력 상자 옆에 있는 아이콘을 클릭하십시오. 
1. 용어를 강조표시하려면 **뷰어 도구 전환** 아이콘을 클릭하십시오. 
1. **오류**를 첫 번째 입력 상자에 강조표시 용어로 입력하고 **컨테이너**를 두 번째 입력 상자에 강조표시 용어로 입력한 후 해당 용어를 포함하는 강조표시된 행을 확인하십시오. 
1. 특정 시간의 로그를 포함하는 행을 보려면 **타임라인 전환** 아이콘을 클릭하십시오. 

### 로그 필터링

태그, 소스, 앱 또는 레벨을 기준으로 로그를 필터링할 수 있습니다. 

1. 맨 위 표시줄에서 **모든 태그**를 클릭하고 **k8s** 선택란을 선택하여 Kubernetes 관련 로그를 보십시오. 
1. **모든 소스**를 클릭하고 로그를 확인할 호스트(작업자 노드)의 이름을 선택하십시오. 이는 클러스터에 여러 작업자 노드가 있는 경우 유용합니다. 
1. 컨테이너 또는 파일 로그를 확인하려면 **모든 앱**을 클릭하고 로그를 볼 항목에 대한 선택란을 선택하십시오. 

### 보기 작성

보기는 특정 필터 세트 및 검색 조회에 대해 저장된 바로 가기입니다. 

로그를 검색하거나 필터링하는 즉시 맨 위 표시줄에 **저장되지 않은 보기**가 표시됩니다. 이를 보기로 저장하려면 다음 작업을 수행하십시오. 
1. **모든 앱**을 클릭하고 **app-log-analysis** 옆에 있는 선택란을 선택하십시오. 
1. **저장되지 않은 보기**를 클릭하고 **새 보기/경보로 저장**을 클릭한 후 해당 보기의 이름을 **app-log-analysis-view**로 지정하십시오. **카테고리**는 비워 두십시오. 
1. **보기 저장**을 클릭하면 앱의 로그를 표시하는 왼쪽 분할창에 새 보기가 표시됩니다. 

### 그래프 및 분석을 사용하여 로그 시각화

이 절에서는 앱 레벨 데이터를 시각화하기 위해 보드를 작성한 후 분석을 포함하는 그래프를 추가합니다. 보드는 그래프와 분석의 집합입니다. 

1. 왼쪽 분할창에서 **보드** 아이콘(설정 아이콘 위에 있음)을 클릭하고 **새 보드**를 클릭하십시오. 
1. 맨 위 표시줄에서 **편집**을 클릭하고 이 **app-log-analysis-board**의 이름을 지정하십시오. **저장**을 클릭하십시오. 
1. **그래프 추가**를 클릭하십시오. 
   - 첫 번째 입력 상자에 **앱**을 필드로서 입력하고 Enter를 누르십시오. 
   - **app-log-analysis**를 필드 값으로 선택하십시오. 
   - **그래프 추가**를 클릭하십시오. 
1. 지난 24시간 동안의 각 간격 내 행 숫자를 보기 위해 **개수**를 메트릭으로 선택하십시오. 
1. 분석을 추가하려면 그래프 아래에 있는 화살표를 클릭하십시오. 
   - 분석 유형으로 **히스토그램**을 선택하십시오. 
   - **레벨**을 필드 유형으로 선택하십시오. 
   - **분석 추가**를 클릭하여 앱에 대해 로그한 모든 레벨과 함께 분석을 보십시오. 

## {{site.data.keyword.mon_full_notm}} 추가 및 클러스터 모니터링
{: #monitor_cluster_sysdig}

다음 작업에서는 애플리케이션에 {{site.data.keyword.mon_full_notm}}를 추가합니다. 이 서비스는 앱의 가용성 및 응답 시간을 정기적으로 확인합니다. 

1. [관찰 가능성](https://{DomainName}/observe/) 페이지로 이동하여 **모니터링**에서 **인스턴스 작성**을 클릭하십시오. 
1. 고유 **서비스 이름**을 제공하십시오. 
1. 지역/위치를 선택하고 리소스 그룹을 선택하십시오. 
1. **누진 계층**을 플랜으로 선택하고 **작성**을 클릭하십시오. 
1. 이전에 작성한 서비스 옆에 있는 **로그 리소스 편집**을 클릭하고 **Kubernetes**를 선택하십시오. 
1. `KUBECONFIG` 환경 변수를 설정한 터미널에서 **클러스터에 Sysdig 에이전트 설치**의 명령을 복사하고 실행하여 클러스터에 Sysdig 에이전트를 배치하십시오. 배치가 완료될 때까지 기다리십시오. 

### {{site.data.keyword.mon_short}} 구성

클러스터의 상태 및 성능을 모니터하기 위해 Sysdig를 구성하려면 다음 작업을 수행하십시오. 
1. **Sysdig 보기**를 클릭하면 Sysdig 모니터 UI가 표시됩니다. 시작 페이지에서 **다음**을 클릭하십시오. 
1. 환경 설정 아래에서 설치 방법으로 **Kubernetes**를 선택하십시오. 
1. 에이전트 구성 성공 메시지 옆에 있는 **다음 단계로 이동**을 클릭하고 다음 페이지에서 **시작하기**를 클릭하십시오. 
1. **다음**을 클릭한 후 **온보딩 완료**를 클릭하여 Sysdig UI의 `Explore` 탭을 표시하십시오. 

### 클러스터 모니터링

앱과 클러스터의 상태 및 성능을 확인하려면 다음 작업을 수행하십시오. 
1. `http://worker-ip-address:portnumber`에서 실행 중인 애플리케이션으로 돌아가 몇 가지 로그 항목을 생성하십시오. 
1. 왼쪽 분할창의 **mycluster**를 펼치고 **기본** 네임스페이스를 펼친 후 **app-log-analysis-deployment**를 클릭하여 Sysdig 모니터 마법사에서 요청 개수, 응답 시간 등을 보십시오. 
1. HTTP 요청-응답 코드를 확인하려면 맨 위 표시줄의 **Kubernetes 팟(Pod) 상태** 옆에 있는 화살표를 클릭하고 **애플리케이션** 아래에서 **HTTP**를 선택하십시오. Sysdig UI의 맨 아래 표시줄에서 간격을 **10M**으로 변경하십시오. 
1. 애플리케이션의 대기 시간을 모니터하려면 다음 작업을 수행하십시오. 
   - 탐색 탭에서 **배치 및 팟(Pod)**을 선택하십시오. 
   - `HTTP` 옆에 있는 화살표를 클릭한 후 메트릭 > 네트워크를 선택하십시오. 
   - **net.http.request.time**을 선택하십시오. 
   - 시간: **합계** 및 그룹: **평균**을 선택하십시오. 
   - **추가 옵션**을 클릭한 후 **토폴로지** 아이콘을 클릭하십시오. 
   - **완료**를 클릭하고 상자를 두 번 클릭하여 보기를 펼치십시오. 
1. 애플리케이션이 실행 중인 Kubernetes 네임스페이스를 모니터하려면 다음 작업을 수행하십시오. 
   - 탐색 탭에서 **배치 및 팟(Pod)**을 선택하십시오. 
   - `net.http.request.time` 옆에 있는 화살표를 클릭하십시오. 
   - 기본 대시보드 > Kubernetes를 선택하십시오. 
   - Kubernetes 상태 > Kubernetes 상태 개요를 선택하십시오. 

### 사용자 정의 대시보드 작성

사전 정의된 대시보드 외에, 고유 사용자 정의 대시보드를 작성하여 사용자의 앱을 실행하고 있는 컨테이너에 대해 가장 유용하거나 중요한 보기 및 메트릭을 하나의 위치에서 표시할 수 있습니다. 각 대시보드는 특정 데이터를 다양한 형식으로 표시하도록 구성된 일련의 패널로 구성됩니다. 

대시보드를 작성하려면 다음 작업을 수행하십시오. 
1. 가장 왼쪽에 있는 분할창에서 **대시보드**를 클릭하고 **대시보드 추가**를 클릭하십시오. 
1. **비어 있는 대시보드**를 클릭하고 이름을 **컨테이너 요청 개요**로 지정한 후 **대시보드 작성**을 클릭하십시오. 
1. **최상위 목록**을 새 패널로 선택하고 패널의 이름을 **컨테이너당 요청 시간**으로 지정하십시오. 
   - **메트릭**에 **net.http.request.time**을 입력하십시오. 
   - 범위: **대시보드 범위 대체**를 클릭하고, **container.image**를 선택하고, **is**를 선택한 후 _애플리케이션 이미지_를 선택하십시오. 
   - **container.id**로 구분하면 각 컨테이너의 실제 요청 시간이 표시됩니다. 
   - **저장**을 클릭하십시오. 
1. 새 패널을 추가하려면 **더하기** 아이콘을 클릭하고 **숫자(#)**를 패널 유형으로 선택하십시오. 
   - **메트릭**에서 **net.http.request.count**를 입력하고 시간 집계를 **평균**에서 **합계**로 변경하십시오. 
   - 범위: **대시보드 범위 대체**를 클릭하고, **container.image**를 선택하고, **is**를 선택한 후 _애플리케이션 이미지_를 선택하십시오. 
   - **1시간** 전과 비교하면 각 컨테이너의 실제 요청 개수가 표시됩니다. 
   - **저장**을 클릭하십시오. 
1. 이 대시보드의 범위를 편집하려면 다음 작업을 수행하십시오. 
   - 제목 패널에서 **범위 편집**을 클릭하십시오. 
   - 드롭 다운에서 **Kubernetes.cluster.name**을 선택/입력하십시오. 
   - 표시 이름을 비워 두고 **is**를 선택하십시오. 
   - **mycluster**를 값으로 선택하고 **저장**을 클릭하십시오. 

## 리소스 제거
{: #remove_resource}

- [관찰 가능성](https://{DomainName}/observe) 페이지에서 LogDNA 및 Sysdig 인스턴스를 제거하십시오. 
- 작업자 노드, 앱 및 컨테이너를 포함하여 클러스터를 삭제하십시오. 이 조치는 실행 취소할 수 없습니다. 
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## 튜토리얼 확장
{: #expand_tutorial}

- [{{site.data.keyword.at_full}} 서비스](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started)를 사용하여 애플리케이션이 IBM Cloud 서비스와 어떻게 상호작용하는지 추적하십시오. 
- 보기에 [경보를 추가하십시오](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts). 
- 로컬 파일에 [로그를 내보내십시오](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export). 

## 관련 컨텐츠
{:related}
- [Kubernetes 클러스터에서 사용되는 수집 키 재설정](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [IBM Cloud Object Storage에 로그 아카이브](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [Sysdig에서 경보 구성](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [Sysdig UI에서의 알림 채널 관련 작업](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
