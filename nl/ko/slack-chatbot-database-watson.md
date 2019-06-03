---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 데이터베이스 중심의 Slackbot 빌드
{: #slack-chatbot-database-watson}

이 튜토리얼에서는 Slackbot을 빌드하여 이벤트 및 컨퍼런스에 대한 Db2 데이터베이스 항목을 작성하고 검색합니다. Slackbot은 {{site.data.keyword.conversationfull}} 서비스에 의해 지원됩니다. 어시스턴트 통합을 사용하여 Slack과 {{site.data.keyword.conversationfull}}를 통합합니다. 

Slack 통합은 Slack과 {{site.data.keyword.conversationshort}} 사이에 메시지를 전달합니다. 여기서 일부 서버 측 대화 상자 조치는 Db2 데이터베이스에 대해 SQL 조회를 수행합니다. 많지는 않지만 모든 코드가 Node.js로 작성됩니다. 

## 목표
{: #objectives}

* 통합을 사용하여 {{site.data.keyword.conversationfull}}를 Slack에 연결
* {{site.data.keyword.openwhisk_short}}에서 Node.js 조치 작성, 배치 및 바인드
* Node.js를 사용하여 {{site.data.keyword.openwhisk_short}}에서 Db2 데이터베이스에 액세스

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse) 또는 [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

  ![아키텍처](images/solution19/SlackbotArchitecture.png)
</p>

## 시작하기 전에
{: #prereqs}

이 튜토리얼을 완료하려면 [{{site.data.keyword.Bluemix_notm}} CLI](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) 및 {{site.data.keyword.openwhisk_short}} [플러그인](/docs/cli?topic=cloud-cli-plug-ins)의 최신 버전이 설치되어 있어야 합니다. 


## 서비스 및 환경 설정
이 절에서는 필요한 서비스를 설정하고 환경을 준비합니다. 이 작업 중 대부분은 스크립트를 사용하여 명령행 인터페이스(CLI)에서 수행할 수 있습니다. 이들은 GitHub에서 사용할 수 있습니다. 

1. [GitHub 저장소](https://github.com/IBM-Cloud/slack-chatbot-database-watson)를 복제한 후 복제된 디렉토리로 이동하십시오. 
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. 로그인되어 있지 않은 경우에는 `ibmcloud login`을 사용하여 대화식으로 로그인하십시오. 
3. 다음을 사용하여 데이터베이스 서비스를 작성할 조직 및 영역을 대상으로 지정하십시오. 
   ```
   ibmcloud target --cf
   ```
4. {{site.data.keyword.dashdbshort}} 인스턴스를 작성한 후 이름을 **eventDB**로 지정하십시오. 
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
   **엔트리** 플랜 이외의 다른 플랜도 사용할 수 있습니다.
5. 나중에 {{site.data.keyword.openwhisk_short}}에서 데이터베이스 서비스에 액세스하려면 권한이 필요합니다. 따라서 서비스 인증 정보를 작성하여 **slackbotkey**로 레이블을 지정합니다.    
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. {{site.data.keyword.conversationshort}} 서비스의 인스턴스를 작성하십시오. **eventConversation**을 이름으로 사용하고 무료 Lite 플랜을 사용하십시오. 
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. 다음으로 {{site.data.keyword.openwhisk_short}}에 대한 조치를 등록하고 서비스 인증 정보를 해당 조치에 바인드합니다. 일부 조치는 웹 조치로 사용으로 설정되고 권한 없는 호출을 방지하기 위해 시크릿이 설정됩니다. 시크릿을 선택하여 매개변수로 전달하고 **YOURSECRET**을 적절하게 대체하십시오. 

   {{site.data.keyword.dashdbshort}}에서 테이블을 작성하기 위해 조치 중 하나가 호출됩니다. {{site.data.keyword.openwhisk_short}}의 조치를 사용하면 로컬 Db2 드라이버도 필요하지 않고 테이블을 수동으로 작성하기 위해 브라우저 기반 인터페이스를 사용하지 않아도 됩니다. 등록 및 설정을 수행하려면 아래 행을 실행하십시오. 모든 조치가 포함된 **setup.sh** 파일이 실행됩니다. 시스템에서 쉘 명령을 지원하지 않는 경우 **setup.sh** 파일에서 각각의 행을 복사한 후 개별적으로 실행하십시오. 

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **참고:** 기본적으로 스크립트는 샘플 데이터의 몇몇 행도 삽입합니다. 위 스크립트에서 `#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r` 행을 주석 해제하여 이를 사용 안함으로 설정할 수 있습니다. 
8. 배치된 조치에 대한 네임스페이스 정보를 추출하십시오. 

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   **/slackdemo/eventInsert** 앞에 있는 부분을 기록해 두십시오. 이는 인코딩된 조직 및 영역입니다. 다음 절에서 이 항목이 필요합니다. 

## 스킬/작업공간 로드
튜토리얼의 이 부분에서는 사전 정의된 작업공간 또는 스킬을 {{site.data.keyword.conversationshort}} 서비스에 로드합니다. 
1. [{{site.data.keyword.Bluemix_short}} 리소스 목록](https://{DomainName}/resources)에서 서비스의 개요를 여십시오. 이전 절에서 작성된 {{site.data.keyword.conversationshort}} 서비스의 인스턴스를 찾으십시오. 해당 항목을 클릭한 후 서비스 별명을 클릭하여 서비스 세부사항을 여십시오. 
2. **도구 실행**을 클릭하여 {{site.data.keyword.conversationshort}} 도구로 이동하십시오. 
3. **스킬**로 전환한 후 **스킬 작성**을 클릭한 후 **스킬 가져오기**를 클릭하십시오. 
4. 대화 상자에서 **JSON 파일 선택**을 클릭한 후 로컬 디렉토리에서 **assistant-skill.json** 파일을 선택하십시오. 가져오기 옵션을 **모든 항목(의도, 엔티티 및 대화 상자)**으로 두고 **가져오기**를 클릭하십시오. 그러면 **TutorialSlackbot**이라는 새 스킬이 작성됩니다. 
5. **대화 상자**를 클릭하여 대화 상자 노드를 확인하십시오. 이를 펼쳐 아래와 같은 구조를 볼 수 있습니다. 

   대화 상자에는 도움말과 간단한 "감사합니다"에 대한 질문을 처리하는 노드가 있습니다. **newEvent** 노드 및 해당 하위에서는 필요한 입력을 수집한 후 새 이벤트 레코드를 Db2에 삽입하는 조치를 호출합니다. 

   **이벤트 조회** 노드는 ID와 날짜 중 어느 것을 기준으로 이벤트를 검색하는지 명확하게 표시합니다. 필요한 데이터를 실제로 검색하고 수집하는 작업은 하위 노드 **짧은 이름별 이벤트 조회** 및 **날짜별 이벤트 조회**에 의해 수행됩니다. 

   **credential_node**는 대화 상자 조치에 대한 시크릿과 Cloud Foundry 조직에 대한 정보를 설정합니다. 후자는 조치를 호출하기 위해 필요합니다. 

  모든 항목이 설정되면 이후에 아래에서 세부사항에 대해 설명합니다.
  ![](images/solution19/SlackBot_Dialog.png)   
6. 대화 상자 노드 **credential_node**를 클릭하고 **Then respond with** 오른쪽에 있는 메뉴 아이콘을 클릭하여 JSON 편집기를 여십시오. 

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   **org_space**를 이전에 검색한 인코딩된 조직 및 영역 정보로 대체하십시오. "@"를 모두 "%40"으로 대체하십시오. 다음으로 **YOURSECRET**을 이전의 실제 시크릿으로 변경하십시오. 아이콘을 다시 클릭하여 JSON 편집기를 닫으십시오. 

## 어시스턴트 작성 및 Slack과 통합

이제 이전의 스킬과 연관된 어시스턴트를 작성한 후 Slack과 통합합니다.  
1. 왼쪽 상단에서 **스킬**을 클릭한 후 **어시스턴트**를 선택하십시오. 다음으로 **어시스턴트 작성**을 클릭하십시오. 
2. 대화 상자에서 **TutorialAssistant**를 이름으로 기입한 후 **어시스턴트 작성**을 클릭하십시오. 다음 화면에서 **대화 상자 스킬 추가**를 선택하십시오. 그런 다음 **기존 스킬 추가**를 선택한 후 목록에서 **TutorialSlackbot**을 선택하여 추가하십시오. 
3. 스킬을 추가한 후 **통합 추가**를 클릭한 다음 **관리 통합**의 목록에서 **Slack**을 선택하십시오. 
4. 지시사항을 따라 챗봇을 Slack과 통합시키십시오. 

## Slackbot 테스트 및 학습
챗봇의 테스트 드라이브에 대한 Slack 작업공간을 여십시오. 봇과 직접 대화를 시작하십시오. 

1. 메시징 양식에 **help**를 입력하십시오. 봇이 몇몇 지침으로 응답합니다. 
2. 이제 **new event**를 입력하여 새 이벤트 레코드에 대한 데이터를 수집하십시오. {{site.data.keyword.conversationshort}} 슬롯을 사용하여 필요한 모든 입력을 수집합니다. 
3. 먼저 이벤트 ID 또는 이름이 표시됩니다. 따옴표는 필수입니다. 이를 통해 더 복잡한 이름을 입력할 수 있습니다. **"Meetup: IBM Cloud"**를 이벤트 이름으로 입력하십시오. 이벤트 이름은 패턴 기반 엔티티 **eventName**으로 정의됩니다. 이를 통해 시작 및 끝 부분에 다양한 유형의 큰따옴표를 사용할 수 있습니다. 
4. 다음으로 이벤트 위치가 표시됩니다. 입력은 [시스템 엔티티 **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details)을 기반으로 합니다. {{site.data.keyword.conversationshort}}에 의해 인식되는 도시에만 사용할 수 있다는 제한이 있습니다. **Friedrichshafen**을 도시로 사용해 보십시오. 
5. 다음 단계에서는 웹 사이트의 URI 또는 이메일 주소 등의 연락처 정보가 요구됩니다. **https://www.ibm.com/events**에서 시작하십시오. 해당 필드에 대해 패턴 기반 엔티티를 사용합니다. 
6. 다음 질문에서는 시작 및 종료의 날짜 및 시간을 수집합니다. **sys-date** 및 **sys-time**을 사용하여 다양한 입력 형식을 허용할 수 있습니다. **다음 목요일**을 시작 날짜로 사용하고 **오후 6시**를 시간으로 사용하고 다음 목요일의 정확한 날짜(예: **2019-05-09** 및 **22:00**)를 종료 날짜 및 시간으로 사용하십시오. 
7. 마지막으로 수집된 모든 데이터를 가진 요약이 인쇄되고 {{site.data.keyword.openwhisk_short}} 조치로 구현된 서버 조치가 호출되어 새 레코드를 Db2에 삽입합니다. 그런 다음 대화 상자가 하위 노드로 전환되어 컨텍스트 변수를 제거하여 처리 환경을 정리합니다. **cancel**, **exit** 등을 입력하여 언제든지 전체 입력 프로세스를 취소할 수 있습니다. 이 경우 사용자 선택사항이 수신확인되고 환경이 정리됩니다.
  ![](images/solution19/SlackSampleChat.png)   

일부 샘플 데이터가 있으면 이제 검색할 시간입니다. 
1. **show event information**을 입력하십시오. 다음은 ID와 날짜 중 어느 것을 기준으로 검색할지 묻는 질문입니다. 다음 질문 **"Think 2019"**에 대해 **name**을 입력하십시오. 이제 챗봇이 해당 이벤트에 대한 정보를 표시합니다. 대화 상자에는 선택할 수 있는 여러 응답이 있습니다. 
2. {{site.data.keyword.conversationshort}}이 백엔드로 준비되면 더 복잡한 문구를 입력하여 대화 상자의 일부를 건너뛸 수 있습니다. **show event by the name "Think 2019"**를 입력으로 사용하십시오. 챗봇은 직접 이벤트 레코드를 리턴합니다. 
3. 이제 날짜별로 검색합니다. 검색은 날짜 쌍으로 정의되며 이벤트 시작 날짜는 이 날짜 쌍 사이에 있어야 합니다. **search conference by date in February 2019**를 입력으로 사용하면 결과는 다시 **Think 2019** 이벤트여야 합니다. **2월** 엔티티는 2월 1일과 2월 28일이라는 두 개의 날짜로 해석되어 날짜 범위의 시작 및 끝에 대한 입력을 제공합니다. [2019년이 지정되지 않는 경우 2월 이후가 식별됩니다. ](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time) 

몇몇 추가 검색 및 새 이벤트 항목 이후 대화 히스토리를 다시 방문하여 향후 대화 상자를 개선할 수 있습니다. [**이해 향상**에 대한 {{site.data.keyword.conversationshort}} 문서](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro)에 있는 지시사항을 따르십시오. 


## 리소스 제거
{:removeresources}

기본 디렉토리에서 정리 스크립트를 실행하면 {{site.data.keyword.dashdbshort}}에서 이벤트 테이블이 삭제되고 {{site.data.keyword.openwhisk_short}}에서 조치가 제거됩니다. 이는 코드 수정 및 확장을 시작할 때 유용할 수 있습니다. 정리 스크립트는 {{site.data.keyword.conversationshort}} 작업공간 또는 스킬을 변경하지 않습니다.    
```bash
sh cleanup.sh
```
{:codeblock}

[{{site.data.keyword.Bluemix_short}} 리소스 목록](https://{DomainName}/resources)에서 서비스의 개요를 여십시오. {{site.data.keyword.conversationshort}} 서비스의 인스턴스를 찾아서 삭제하십시오. 

## 튜토리얼 확장
이 튜토리얼에 추가하거나 이 튜토리얼을 변경하시겠습니까? 몇 가지 방법은 다음과 같습니다. 
1. 검색 기능을 추가하십시오(예: 와일드카드 검색 또는 이벤트 기간 검색("give me all events longer than 8 hours")). 
2. {{site.data.keyword.dashdbshort}} 대신 {{site.data.keyword.databases-for-postgresql}}을 사용하십시오. [이 Slackbot 튜토리얼에 대한 GitHub 저장소](https://github.com/IBM-Cloud/slack-chatbot-database-watson)에는 {{site.data.keyword.databases-for-postgresql}}을 지원하는 코드가 항상 포함되어 있습니다. 
3. 날씨 서비스를 추가하고 이벤트 날짜 및 위치에 대한 예측 데이터를 검색하십시오. 
4. 이벤트 데이터를 iCalendar **.ics** 파일로 내보내십시오. 
5. 또 다른 통합을 추가하여 챗봇을 Facebook Messenger에 연결하십시오. 
6. 대화식 요소(예: 단추)를 출력에 추가하십시오.       


## 관련 컨텐츠
{:related}

이 튜토리얼에서 다루는 주제에 대한 추가 정보에 대한 링크는 다음과 같습니다. 

챗봇 관련 블로그 게시물: 
* [챗봇: IBM Watson Conversation의 슬롯에 대한 몇몇 트릭](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [적극적인 챗봇: 우수 사례](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [챗봇 빌드: 추가 팁 및 트릭](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

문서 및 SDK:
* [IBM Watson Conversation에서 변수 처리를 위한 팁 및 트릭](https://github.com/IBM-Cloud/watson-conversation-variables)이 있는 GitHub 저장소
* [{{site.data.keyword.openwhisk_short}} 문서](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* 문서: [{{site.data.keyword.dashdbshort}}용 IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* 개발자용 [무료 Db2 개발자 커뮤니티 에디션](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions)
* 문서: [ibm_db Node.js 드라이버에 대한 API 설명](https://github.com/ibmdb/node-ibm_db)
* [{{site.data.keyword.cloudantfull}} 문서](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
