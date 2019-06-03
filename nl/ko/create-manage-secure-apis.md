---
copyright:
  years: 2017, 2019
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

# REST API 작성, 보안 및 관리
{: #create-manage-secure-apis}

이 튜토리얼은 LoopBack Node.js API 프레임워크를 사용하여 REST API를 작성하는 방법을 보여줍니다. Loopback을 사용하면 디바이스와 브라우저를 데이터 및 서비스에 연결하는 REST API를 신속하게 작성할 수 있습니다. 또한 {{site.data.keyword.apiconnect_long}}를 사용하여 API에 관리, 가시성, 보안 및 속도 제한 기능을 추가합니다.
{:shortdesc}

## 목표

* 매우 적은 코딩 작업으로 REST API를 빌드
* API를 {{site.data.keyword.Bluemix_notm}}에 공개하여 개발자들에게 제공
* 기존 API를 {{site.data.keyword.apiconnect_short}}로 가져오기
* SOR(System of Record)을 안전하게 노출하고 이에 대한 액세스를 제어

## 사용되는 서비스

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 

* [Loopback](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* [SDK for Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry 앱

## 아키텍처

![아키텍처](images/solution13/Architecture.png)

1. 개발자가 RESTful API를 정의합니다. 
2. 개발자가 API를 {{site.data.keyword.apiconnect_long}}에 공개합니다. 
3. 사용자와 애플리케이션이 API를 이용합니다. 

## 시작하기 전에

* [Node.js](https://nodejs.org/en/download/) 버전 6.x를 다운로드하여 설치하십시오(더 높은 Node.js 버전이 설치되어 있는 경우에는 [nvm](https://github.com/creationix/nvm) 등을 사용). 

## Node.js에서 REST API 작성

{: #create_api}
이 절에서는 [LoopBack](https://loopback.io/doc/index.html)을 사용하여 Node.js에서 API를 작성합니다. LoopBack은 매우 적은 코딩 작업으로 동적 엔드 투 엔드 REST API를 작성할 수 있게 해 주는 확장성 높은 오픈 소스 Node.js 프레임워크입니다. 

### 애플리케이션 작성

1. {{site.data.keyword.apiconnect_short}} 명령행 도구를 설치하십시오. {{site.data.keyword.apiconnect_short}} 설치에 문제가 있는 경우에는 명령 전에 `sudo`를 사용하거나 [여기](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit)에 있는 지시사항을 따르십시오.
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. 다음 명령을 실행하여 애플리케이션을 작성하십시오.
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. `Enter`를 눌러 **entries-api**를 **애플리케이션 이름**으로 사용하십시오. 
4. `Enter`를 눌러 **entries-api**를 **프로젝트를 포함할 디렉토리**로 사용하십시오. 
5. **3.x(현재)**를 **LoopBack 버전**으로 선택하십시오. 
6. **empty-server**를 **애플리케이션 유형**으로 선택하십시오. 

![APIC Loopback 스캐폴딩](images/solution13/apic_loopback.png)

### 데이터 소스 추가

데이터 소스는 데이터베이스, 외부 REST API, SOAP 웹 서비스 및 스토리지 서비스와 같은 백엔드 시스템을 나타냅니다. 데이터 소스는 일반적으로 작성, 검색, 업데이트 및 삭제(CRUD) 기능을 제공합니다. Loopback은 다양한 유형의 [데이터 소스](http://loopback.io/doc/en/lb3/Connectors-reference.html)를 지원하지만, 여기서는 작업을 간단하게 하기 위해 API에 인메모리 데이터 저장소를 사용합니다. 

1. 디렉토리를 새 프로젝트로 변경하고 API Designer를 실행하십시오.
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. 탐색에서 **데이터 소스**를 클릭하십시오. 그 후 **추가** 단추를 클릭하십시오. **새 LoopBack 데이터 소스** 대화 상자가 열립니다. 
3. **이름** 텍스트 필드에 `entriesDS`를 입력하고 **새로 작성** 단추를 클릭하십시오. 
4. **커넥터** 콤보 상자에서 **인메모리 DB**를 선택하십시오. 
5. 왼쪽 상단의 **모든 데이터 소스**를 클릭하십시오. 해당 데이터 소스가 데이터 소스 목록에 표시됩니다. 

   편집기는 새 데이터 소스에 대한 설정으로 server/datasources.json 파일을 자동으로 업데이트합니다.
   {:tip}

![API Designer 데이터 소스](images/solution13/datastore.png)

### 모델 추가

모델은 백엔드 시스템에 있는 데이터를 나타내는, Node.js와 REST API를 모두 사용하는 JavaScript 오브젝트입니다. 모델은 데이터 소스를 통해 이러한 시스템과 연결됩니다. 이 절에서는 데이터의 구조를 정의하고 이를 이전에 작성한 데이터 소스에 연결합니다. 

1. 탐색에서 **모델**을 클릭하십시오. 그 후 **추가** 단추를 클릭하십시오. **새 LoopBack 모델** 대화 상자가 열립니다. 
2. **이름** 텍스트 필드에 `entry`를 입력하고 **새로 작성** 단추를 클릭하십시오. 
3. **데이터 소스** 콤보 상자에서 **entriesDS**를 선택하십시오. 
4. **특성** 카드에서 **특성 추가** 아이콘을 사용하여 다음 특성을 추가하십시오. 
    1. **특성 이름**에 대해 `name`을 입력하고 **유형** 콤보 상자에서 **문자열**을 선택하십시오. 
    2. **특성 이름**에 대해 `email`을 입력하고 **유형** 콤보 상자에서 **문자열**을 선택하십시오. 
    3. **특성 이름**에 대해 `comment`를 입력하고 **유형** 콤보 상자에서 **문자열**을 선택하십시오. 
5. 오른쪽 상단에 있는 **저장** 아이콘을 클릭하여 모델을 저장하십시오. 

![모델 생성기](images/solution13/models.png)

## LoopBack 애플리케이션 테스트

이 절에서는 Loopback 애플리케이션의 로컬 인스턴스를 시작하고, API Designer를 사용해 데이터를 삽입하고 조회하여 API를 테스트합니다. API Designer의 탐색 도구는 적절한 보안 헤더 및 기타 요청 매개변수를 포함하므로 이를 사용하여 브라우저에서 REST 엔드포인트를 테스트해야 합니다. 

1. 왼쪽 하단에 있는 **시작** 아이콘을 클릭하여 로컬 서버를 시작하고 **실행 중** 메시지가 표시될 때까지 기다리십시오. 
2. 배너에서 **탐색** 링크를 클릭하여 API Designer의 탐색 도구를 보십시오. 사이드바에 API에서 LoopBack 모델에 대해 사용할 수 있는 REST 오퍼레이션이 표시됩니다. 
3. 왼쪽 분할창에서 **entry.create** 오퍼레이션을 클릭하여 엔드포인트를 표시하십시오. 가운데 분할창이 매개변수, 보안, 모델 인스턴스 데이터 및 응답 코드를 포함한 엔드포인트 관련 요약 정보를 표시합니다. 오른쪽 분할창은 cURL 명령과 Ruby, Python, Java 및 Node.js 등의 언어를 사용하여 엔드포인트를 호출하는 샘플 코드를 제공합니다. 
4. 오른쪽 분할창에서 **사용해 보기** 링크를 클릭하십시오. **매개변수**로 스크롤하여 **데이터** 텍스트 영역에 다음 내용을 입력하십시오.
    ```javascript
    {
      "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. **오퍼레이션 호출** 단추를 클릭하십시오.
  ![API Designer에서 API 테스트](images/solution13/data_entry_1.png)
6. **응답 코드: 200 OK**가 출력되었는지 확인하여 POST가 성공했는지 확인하십시오. localhost의 신뢰할 수 없는 인증서로 인해 오류 메시지가 표시되는 경우에는 API Designer 탐색 도구에서 오류 메시지에 제공된 링크를 클릭하여 인증서를 승인한 후 웹 브라우저에서 오퍼레이션을 호출하십시오. 정확한 프로시저는 사용 중인 웹 브라우저에 따라 달라집니다. 
7. cURL을 사용하여 다른 항목을 추가하십시오. 포트가 애플리케이션의 포트와 일치하는지 확인하십시오.
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. **entry.find** 오퍼레이션을 클릭한 후 **사용해 보기** 링크와 **오퍼레이션 호출** 단추를 차례로 클릭하십시오. 응답이 모든 항목을 표시합니다. **Jane Doe** 및 **John Doe**에 대한 JSON을 볼 수 있습니다.
  ![entry_find](images/solution13/find_response.png)

`entries-api` 디렉토리에서 `npm start` 명령을 실행하여 Loopback 애플리케이션을 수동으로 시작할 수도 있습니다. 사용자의 REST API는 [http://localhost:3000/api/entries](http://localhost:3000/api/entries)에서 사용 가능하지만 {{site.data.keyword.apiconnect_short}}에 의해 관리 및 보호되지는 않습니다.
{:tip}

## {{site.data.keyword.apiconnect_short}} 서비스 작성

다음 단계를 준비하기 위해 {{site.data.keyword.Bluemix_notm}}에 **{{site.data.keyword.apiconnect_short}}** 서비스를 작성합니다. {{site.data.keyword.apiconnect_short}}는 API에 대한 게이트웨이 역할을 수행하며 관리, 보안 및 속도 제한 기능 또한 제공합니다. 

1. {{site.data.keyword.Bluemix_notm}} [리소스 목록](https://{DomainName}/resources)을 실행하십시오. 
2. **카탈로그 > 통합 > {{site.data.keyword.apiconnect_short}}**로 이동한 후 **작성** 단추를 클릭하십시오. 

## {{site.data.keyword.Bluemix_notm}}에 API 공개

{: #publish}
여기서는 API Designer를 사용하여 애플리케이션을 {{site.data.keyword.Bluemix_notm}}에 Cloud Foundry 애플리케이션으로 배치하며 API 정의도 **{{site.data.keyword.apiconnect_short}}**에 공개합니다. 사용자의 로컬 툴킷은 API Designer입니다. 이를 닫은 경우에는 프로젝트 디렉토리에서 `apic edit`을 사용하여 다시 실행하십시오. 

애플리케이션은 `ibmcloud cf push` 명령을 사용하여 배치할 수도 있지만, 이 경우에는 보안되지 않습니다. {{site.data.keyword.apiconnect_short}}로 [API를 가져오려면](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing) `definitions` 폴더에 있는 OpenAPI 정의 파일을 사용하십시오. API Designer를 사용하여 배치하면 애플리케이션이 보안되며 정의를 자동으로 가져옵니다.
{:tip}

1. API Designer로 돌아와 배너의 **공개** 링크를 클릭하십시오. 그 후 **대상 추가 및 관리 > IBM Bluemix 대상 추가**를 클릭하십시오. 
2. 공개할 **지역** 및 **조직**을 선택하십시오. 
3. **샌드박스** 카탈로그를 선택하고 **다음**을 클릭하십시오. 
4. **새 애플리케이션 이름 입력** 텍스트 필드에 `entries-api-application`을 입력하고 **+** 아이콘을 클릭하십시오. 
5. 목록에서 **entries-api-application**을 클릭하고 **저장**을 클릭하십시오. 
6. 배너의 왼쪽 최상단에 있는 햄버거 **메뉴** 아이콘을 클릭하십시오. 그 후 **프로젝트** 및 **entries-api** 목록 항목을 클릭하십시오. 
7. API Designer UI에서 **API > entries-api > 어셈블** 링크를 클릭하십시오. 
8. 어셈블리 편집기에서 **필터 정책** 아이콘을 클릭하십시오. 
9. **DataPower Gateway 정책**을 선택하고 **저장**을 클릭하십시오. 
10. 맨 위 표시줄에서 **공개**를 클릭하고 대상을 선택하십시오. **애플리케이션 공개**를 선택하고 제품 스테이징 또는 공개 > **특정 제품** > **entries-api**를 선택하십시오. 
11. **공개**를 클릭하고 애플리케이션 공개가 완료되기를 기다리십시오.
    ![공개 대화 상자](images/solution13/publish.png)

    애플리케이션은 API와 관련된 Loopback 모델, 데이터 소스 및 코드를 포함합니다. 제품은 API가 어떤 방식으로 개발자에게 공개되는지 사용자가 선언할 수 있게 해 줍니다.
    {:tip}

API 애플리케이션이 {{site.data.keyword.Bluemix_notm}}에 Cloud Foundry 애플리케이션으로서 공개되었습니다. {{site.data.keyword.Bluemix_notm}} [리소스 목록](https://{DomainName}/resources)에 있는 Cloud Foundry 애플리케이션들을 살펴보면 이 애플리케이션을 볼 수 있지만, 보호되고 있으므로 URL을 사용한 직접 액세스는 불가능합니다. 다음 절에서는 관리 API에 액세스하는 방법을 보여줍니다. 

## API 게이트웨이

이제까지는 로컬에서 API를 디자인하고 테스트했습니다. 이 절에서는 {{site.data.keyword.apiconnect_short}}를 사용하여 {{site.data.keyword.Bluemix_notm}}에서 배치된 API를 테스트합니다. 

1. {{site.data.keyword.Bluemix_notm}} [리소스 목록](https://{DomainName}/resources)을 실행하십시오. 
2. **Cloud Foundry 서비스**에서 자신의 **{{site.data.keyword.apiconnect_short}}** 서비스를 찾아 선택하십시오. 
3. **탐색** 메뉴를 클릭한 후 **샌드박스** 링크를 클릭하십시오. 
4. **entry.create** 오퍼레이션을 클릭하십시오. 
5. 오른쪽 분할창에서 **사용해 보기**를 클릭하십시오. **매개변수**로 스크롤하여 **데이터** 텍스트 영역에 다음 내용을 입력하십시오.
    ```javascript
    {
      "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. **오퍼레이션 호출** 단추를 클릭하십시오. 성공을 나타내는 **코드: 200** 응답이 표시됩니다. 

![게이트웨이](images/solution13/gateway.png)

안전한 관리 API URL이 각 오퍼레이션 옆에 표시되며 이는 `https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries`와 같아야 합니다.
{: tip}

## 속도 제한

속도 제한을 설정하면 API 및 API의 특정 오퍼레이션에 대한 네트워크 트래픽을 관리할 수 있습니다. 속도 제한은 특정 시간 간격에 허용되는 최대 호출 수입니다. 

1. API Designer로 돌아가 **제품 > entries-api**를 클릭하십시오. 
2. 왼쪽에서 **기본 플랜**을 선택하십시오. 
3. **기본 플랜**을 펼치고 **속도 제한** 필드로 스크롤하십시오. 
4. 이 필드를 **10**개 호출 / **1****분**으로 설정하십시오. 
5. **엄격한 제한 적용**을 선택하고 **저장** 아이콘을 클릭하십시오.
  ![속도 제한 페이지](images/solution13/rate_limit.png)
6. [{{site.data.keyword.Bluemix_notm}}에 API 공개](#publish) 절의 단계에 따라 API를 다시 공개하십시오. 

이제 API가 1분당 10개의 요청으로 제한됩니다. **사용해 보기** 기능을 사용하여 한계까지 도달하십시오. [속도 제한 설정](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits)에 대한 자세한 정보를 보거나 API Designer를 탐색하여 사용 가능한 모든 관리 기능을 확인하십시오. 

## 튜토리얼 확장

이제까지의 작업으로 안전한 관리 API를 빌드했습니다. 아래 항목은 이 API를 개선하는 데 대한 추가 제안사항입니다. 

* [{{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant) LoopBack 커넥터를 사용하여 지속성 추가
* API Designer를 사용하여 API 관리에 사용할 수 있는 [추가 설정 보기](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0)
* {{site.data.keyword.apiconnect_short}}에서 [사용 가능한](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics) API **분석** 및 **시각화** 검토

## 관련 컨텐츠

* [Loopback 문서](https://loopback.io/doc/index.html)
* [{{site.data.keyword.apiconnect_long}} 시작하기](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
