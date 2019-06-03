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

# 데이터 검색 및 분석을 위해 서버리스 및 Cloud Foundry 결합
{: #serverless-github-traffic-analytics}
이 튜토리얼에서는 자동으로 저장소에 대한 GitHub 트래픽 통계를 수집하고 트래픽 분석에 대한 기반을 제공하는 애플리케이션을 작성합니다. GitHub는 최근 14일 동안의 트래픽 데이터에 대한 액세스만 제공합니다. 오랜 기간에 대한 통계를 분석하려는 경우에는 해당 데이터를 직접 다운로드하고 저장해야 합니다. 이 튜토리얼에서는 서버리스 조치를 배치하여 트래픽 데이터를 검색한 후 SQL 데이터베이스에 저장합니다. 또한 Cloud Foundry 앱을 사용하여 저장소를 관리하고 데이터 분석을 위한 통계에 대한 액세스를 제공합니다. 이 튜토리얼에서 설명된 앱 및 서버리스 조치는 싱글 테넌트 모드를 지원하는 초기 기능 세트를 사용하여 멀티 테넌트 준비 솔루션을 구현합니다. 

![](images/solution24-github-traffic-analytics/Architecture.png)

## 목표

* 멀티 테넌트 지원 및 보안 액세스를 사용하여 Python 데이터베이스 앱 배치
* App ID를 OpenID Connect 기반 인증 제공자로 통합
* GitHub 트래픽 통계의 자동화된 서버리스 콜렉션 설정
* 그래픽 트래픽 분석을 위해 {{site.data.keyword.dynamdashbemb_short}} 통합

## 제품
이 튜토리얼에서는 다음과 같은 제품을 사용합니다. 
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## 시작하기 전에
{: #prereqs}

이 튜토리얼을 완료하려면 [IBM Cloud CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) 및 {{site.data.keyword.openwhisk_short}} [플러그인](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)의 최신 버전이 설치되어 있어야 합니다. 

## 서비스 및 환경 설정(쉘)
이 절에서는 필요한 서비스를 설정하고 환경을 준비합니다. 이는 모두 쉘 환경에서 달성할 수 있습니다. 

1. [GitHub 저장소](https://github.com/IBM-Cloud/github-traffic-stats)를 복제한 후 복제된 디렉토리 및 해당 **backend** 서브디렉토리로 이동하십시오. 
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. `ibmcloud login`을 사용하여 {{site.data.keyword.Bluemix_short}}에 대화식으로 로그인하십시오. `ibmcloud target` 명령을 실행하여 세부사항을 다시 확인할 수 있습니다. 조직 및 영역이 설정되어 있어야 합니다. 

3. **엔트리** 플랜을 사용하여 {{site.data.keyword.dashdbshort}} 인스턴스를 작성한 후 이름을 **ghstatsDB**로 지정하십시오. 
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. 나중에 {{site.data.keyword.openwhisk_short}}에서 데이터베이스 서비스에 액세스하려면 권한이 필요합니다. 따라서 서비스 인증 정보를 작성한 후 레이블을 **ghstatskey**로 지정합니다.    
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. {{site.data.keyword.appid_short}} 서비스의 인스턴스를 작성하십시오. **ghstatsAppID**를 이름으로 사용하고 **등급화된 계층** 플랜을 사용하십시오. 
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
   그런 다음 Cloud Foundry 영역에서 해당 새 서비스 인스턴스의 별명을 작성하십시오. 배치하는 영역으로 **YOURSPACE**를 대체하십시오.
   ```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. **lite** 플랜을 사용하여 {{site.data.keyword.dynamdashbemb_short}}의 인스턴스를 작성하십시오. 
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
   다시 해당 새 서비스 인스턴스의 별명을 작성하고 **YOURSPACE**를 대체하십시오.
   ```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. **백엔드** 디렉토리에서 애플리케이션을 IBM Cloud에 푸시하십시오. 이 명령에서는 애플리케이션에 대한 랜덤 라우트를 사용합니다. 
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
   배치가 완료될 때까지 기다리십시오. 애플리케이션 파일이 업로드되고 런타임 환경이 작성되고 서비스가 애플리케이션에 바인드됩니다. `manifest.yml` 파일에서 서비스 정보를 가져옵니다. 다른 서비스 이름을 사용한 경우 해당 파일을 업데이트해야 합니다. 프로세스가 완료되고 나면 애플리케이션 URI가 표시됩니다. 

   위의 명령에서는 애플리케이션에 대해 랜덤이지만 고유한 라우트를 사용합니다. 직접 선택하려면 이를 추가적인 매개변수로 명령에 추가하십시오(예: `ibmcloud cf push your-app-name`). `manifest.yml` 파일을 편집하고 **이름**을 변경하고 **random-route**를 **true**에서 **false**로 변경할 수도 있습니다.
   {:tip}

## App ID 및 GitHub 구성(브라우저)
다음과 같은 단계는 모두 인터넷 브라우저를 사용하여 수행됩니다. 먼저 Cloud Directory를 사용하고 Python 앱에 대해 작업하도록 {{site.data.keyword.appid_short}}를 구성합니다. 그런 다음 GitHub 액세스 토큰을 작성합니다. 이는 배치된 기능이 트래픽 데이터를 검색하기 위해 필요합니다. 

1. [{{site.data.keyword.Bluemix_short}} 리소스 목록](https://{DomainName}/resources)에서 서비스의 개요를 여십시오. **서비스** 섹션에서 {{site.data.keyword.appid_short}} 서비스의 인스턴스를 찾으십시오. 해당 항목을 클릭하여 세부사항을 여십시오. 
2. 서비스 대시보드에서 왼쪽에 있는 메뉴의 **ID 제공자** 아래에서 **관리**를 클릭하십시오. 그러면 Facebook, Google, SAML 2.0 Federation 및 Cloud Directory 등의 사용 가능한 ID 제공자의 목록을 가져옵니다. Cloud Directory를 **켜짐**으로 전환하고 다른 모든 제공자는 **꺼짐**으로 전환하십시오. 
   
   [MFA(Multi-Factor Authentication)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) 및 고급 비밀번호 규칙을 구성할 수 있습니다. 이에 대해서는 이 튜토리얼의 일부로 다루지 않습니다.
   {:tip}

3. 해당 페이지의 맨 아래에 경로 재지정 URL의 목록이 있습니다. 애플리케이션의 **url**에 /redirect_uri를 붙여 입력하십시오. 예를 들어, `https://github-traffic-stats-random-word.mybluemix.net/redirect_uri`입니다. 

   앱을 로컬로 테스트하는 경우 경로 재지정 URL은 `http://0.0.0.0:5000/redirect_uri`입니다. 여러 경로 재지정 URL을 구성할 수 있습니다.
   {:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. 왼쪽의 메뉴에서 **사용자**를 클릭하십시오. 그러면 Cloud Directory에서 사용자의 목록이 열립니다. **사용자 추가** 단추를 클릭하여 자신을 첫 번째 사용자로 추가하십시오. 이제 {{site.data.keyword.appid_short}} 서비스 구성이 완료되었습니다. 
5. 브라우저에서 [Github.com](https://github.com/settings/tokens)을 방문하여 **설정 -> 개발자 설정 -> 개인 액세스 토큰**으로 이동하십시오. **새 토큰 생성** 단추를 클릭하십시오. **토큰 설명**에 대해 **GHStats 튜토리얼**을 입력하십시오. 그런 다음 **repo** 카테고리 아래의 **public_repo**와 **admin:org** 아래의 **read:org**를 사용으로 설정하십시오. 이제 해당 페이지의 맨 아래에서 **토큰 생성**을 클릭하십시오. 새 액세스 토큰이 다음 페이지에 표시됩니다. 다음과 같은 애플리케이션 설정 수행 중에 이 액세스 토큰이 필요합니다.
   ![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## Python 앱 구성 및 테스트
준비 후 앱을 구성하고 테스트합니다. 앱은 인기 있는 [Flask](http://flask.pocoo.org/) 마이크로 프레임워크를 사용하여 Python으로 작성됩니다. 저장소를 통계 콜렉션에서 추가하거나 제거할 수 있습니다. 표 형식 보기에서 트래픽 데이터에 액세스할 수 있습니다. 

1. 브라우저에서 배치된 앱의 URI를 여십시오. 시작 페이지가 표시됩니다.
   ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. 브라우저에서 `/admin/initialize-app`을 URI에 추가한 후 페이지에 액세스하십시오. 이는 애플리케이션 및 해당 데이터를 초기화하는 데 사용됩니다. **초기화 시작** 단추를 클릭하십시오. 그러면 비밀번호로 보호된 구성 페이지로 이동합니다. 로그인하는 데 사용되는 이메일 주소를 시스템 관리자의 ID로 사용합니다. 이전에 구성한 이메일 주소 및 비밀번호를 사용하십시오. 

3. 구성 페이지에서 이름(인사를 위해 사용됨), GitHub 사용자 이름 및 이전에 생성된 액세스 토큰을 입력하십시오. **초기화**를 클릭하십시오. 그러면 데이터베이스 테이블이 작성되고 일부 구성 값이 삽입됩니다. 마지막으로 시스템 관리자 및 테넌트에 대한 데이터베이스 레코드가 작성됩니다.
   ![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. 완료되면 관리 저장소 목록으로 이동합니다. 이제 GitHub 계정 또는 조직의 이름 및 저장소의 이름을 제공하여 저장소를 추가할 수 있습니다. 데이터를 입력한 후 **저장소 추가**를 클릭하십시오. 새로 지정된 ID와 함께 저장소가 테이블에 표시되어야 합니다. ID를 입력한 후 **저장소 삭제**를 클릭하여 시스템에서 저장소를 제거할 수 있습니다.
![](images/solution24-github-traffic-analytics/RepositoryList.png)

## Cloud Functions 및 트리거 배치
관리 앱이 준비되어 있을 때 조치 및 트리거와 {{site.data.keyword.openwhisk_short}}에 대해 이 둘을 연결하는 규칙을 배치하십시오. 이 오브젝트는 지정된 스케줄에 GitHub 트래픽 데이터를 자동으로 수집하는 데 사용됩니다. 조치는 데이터에 연결하고 모든 테넌트 및 해당 저장소에서 반복하고 각 저장소에 대한 보기 및 복제 데이터를 확보합니다. 해당 통계는 데이터베이스에 병합됩니다. 

1. **functions** 디렉토리로 변경하십시오. 
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. 새 조치인 **collectStats**를 작성하십시오. 이 조치는 이미 필요한 데이터베이스 드라이버를 포함하고 있는 [Python 3 환경](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments)을 사용합니다. 조치의 소스 코드는 `ghstats.zip` 파일에 제공됩니다. 
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   조치의 소스 코드(`__main__.py`)를 수정하는 경우 `zip -r ghstats.zip  __main__.py github.py`를 다시 사용하여 zip 아카이브를 다시 패키지할 수 있습니다. 세부사항은 `setup.sh` 파일을 참조하십시오.
   {:tip}
3. 조치를 데이터베이스 서비스에 바인드하십시오. 환경 설정 중에 작성한 인스턴스 및 서비스 키를 사용하십시오. 
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. [알람 패키지](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm)를 기반으로 트리거를 작성하십시오. 이를 통해 다양한 양식으로 알람을 지정할 수 있습니다. [cron](https://en.wikipedia.org/wiki/Cron) 유사 스타일을 사용하십시오. 4월 21일부터 12월 21일까지 트리거는 매일 오전 6시(UTC)에 실행됩니다. 시작 날짜가 미래인지 확인하십시오. 
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  `"0 6 * * 0"`을 적용하여 트리거를 매일에서 매주 스케줄로 변경할 수 있습니다. 그러면 매주 일요일 오전 6시에 실행됩니다.
  {:tip}
5. 마지막으로 트리거 **myDaily**를 **collectStats** 조치에 연결하는 **myStatsRule** 규칙을 작성합니다. 이제 트리거는 이전 단계에서 지정된 스케줄에 조치가 실행되게 합니다. 
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. 초기 테스트 실행에 대한 조치를 호출하십시오. 리턴된 **repoCount**는 이전에 구성한 저장소의 수를 반영해야 합니다. 
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
   출력은 다음과 비슷합니다.
   ```
   {
       "repoCount": 18
   }
   ```
7. 앱 페이지가 있는 브라우저 창에서 이제 저장소 트래픽을 방문할 수 있습니다. 기본적으로 10개의 항목이 표시됩니다. 이를 다른 값으로 변경할 수 있습니다. 테이블 열을 정렬하거나 검색 상자를 사용하여 특정 저장소에 대해 필터링할 수도 있습니다. 날짜 및 조직 이름을 입력한 후 뷰 카운트별로 정렬하여 특정 날짜의 최고 점수 항목을 나열할 수 있습니다.
   ![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## 결론
이 튜토리얼에서는 서버리스 조치와 관련 트리거 및 규칙을 배치했습니다. 이들을 통해 GitHub 저장소에 대한 트래픽 데이터를 자동으로 검색할 수 있습니다. 테넌트별 액세스 토큰을 포함하여 해당 저장소에 대한 정보는 SQL 데이터베이스({{site.data.keyword.dashdbshort}})에 저장됩니다. 이 데이터베이스는 앱 포털에 트래픽 통계를 제공하고 사용자 및 저장소를 관리하기 위해 Cloud Foundry 앱에 의해 사용됩니다. 사용자는 검색 가능한 테이블에 있거나 임베디드 대시보드({{site.data.keyword.dynamdashbemb_short}} 서비스(아래 이미지 참조))에 시각화된 트래픽 통계를 볼 수 있습니다. 저장소 목록 및 트래픽 데이터를 CSV 파일로 다운로드할 수도 있습니다. 

Cloud Foundry 앱은 {{site.data.keyword.appid_short}}에 연결되는 OpenID Connect 클라이언트를 통해 액세스를 관리합니다.
![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## 정리
이 튜토리얼에 사용된 리소스를 정리하기 위해 관련 서비스 및 앱을 삭제할 수 있고 조치, 트리거 및 규칙도 작성된 역순으로 삭제할 수 있습니다. 

1. {{site.data.keyword.openwhisk_short}} 규칙, 트리거 및 조치를 삭제하십시오. 
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. Python 앱 및 해당 서비스를 삭제하십시오. 
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## 튜토리얼 확장
이 튜토리얼에 추가하거나 이 튜토리얼을 변경하시겠습니까? 몇 가지 방법은 다음과 같습니다. 
* 멀티 테넌트 지원을 위해 앱을 확장하십시오. 
* 데이터에 대한 차트를 통합하십시오. 
* 소셜 ID 제공자를 사용하십시오. 
* 날짜 선택도구를 통계 페이지에 추가하여 표시된 데이터를 필터링하십시오. 
* {{site.data.keyword.appid_short}}에 대해 사용자 정의 로그인 페이지를 사용하십시오. 
* [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)를 사용하여 개발자 간 소셜 코딩 관계를 탐색하십시오. 

## 관련 컨텐츠
이 튜토리얼에서 다루는 주제에 대한 추가 정보에 대한 링크는 다음과 같습니다. 

문서 및 SDK:
* [{{site.data.keyword.openwhisk_short}} 문서](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* 문서: [{{site.data.keyword.dashdbshort}}용 IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.appid_short}} 문서](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [IBM Cloud의 Python 런타임](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
