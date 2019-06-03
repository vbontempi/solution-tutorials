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

# 음성 인식 Android 챗봇 빌드
{: #android-watson-chatbot}

{{site.data.keyword.Bluemix_short}}의 {{site.data.keyword.conversationshort}}, {{site.data.keyword.texttospeechshort}} 및 {{site.data.keyword.speechtotextshort}} 서비스를 사용하여 음성 인식 Android 기반 챗봇을 얼마나 쉽고 빠르게 작성할 수 있는지 알아보십시오. 

이 튜토리얼에서는 인텐트 및 엔티티를 정의하고 챗봇이 고객의 질문에 응답하는 데 필요한 대화 플로우를 빌드하는 프로세스를 안내합니다. 사용자는 Android 앱과의 손쉬운 상호작용을 위해 {{site.data.keyword.speechtotextshort}} 및 {{site.data.keyword.texttospeechshort}} 서비스를 사용으로 설정하는 방법을 학습합니다.
{:shortdesc}

## 목표
{: #objectives}

- {{site.data.keyword.conversationshort}}를 사용하여 챗봇을 사용자 정의하고 배치합니다. 
- 일반 사용자가 음성 및 오디오를 사용하여 챗봇과 상호작용할 수 있도록 합니다. 
- Android 앱을 구성하고 실행합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 제품을 사용합니다. 

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## 아키텍처
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* 사용자가 자신의 음성을 사용하여 모바일 애플리케이션과 상호작용합니다. 
* 오디오가 {{site.data.keyword.speechtotextfull}}를 사용하여 텍스트로 변환됩니다. 
* 텍스트가 {{site.data.keyword.conversationfull}}에 전달됩니다. 
* {{site.data.keyword.conversationfull}}의 응답이 {{site.data.keyword.texttospeechfull}}에 의해 오디오로 변환되며 결과가 모바일 애플리케이션에 다시 전송됩니다. 

## 시작하기 전에
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html)를 다운로드하여 설치하십시오. 

## 서비스 작성
{: #setup}

이 절에서는 고객을 돕는 가상 코그너티브 어시스턴트를 빌드하는 데 필요한 {{site.data.keyword.conversationshort}}부터 시작하여 튜토리얼에 필요한 서비스를 작성합니다. 

1. [**{{site.data.keyword.Bluemix_notm}} 카탈로그**](https://{DomainName}/catalog/)로 이동하여 [{{site.data.keyword.conversationshort}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation) 서비스 > **Lite** 플랜을 선택하십시오. 
   1. **이름**을 **android-chatbot-assistant**로 설정하십시오. 
   1. **작성**하십시오. 
2. 왼쪽 분할창에서 **서비스 인증 정보**를 클릭하고 **새 인증 정보**를 클릭하십시오. 
   1. **이름**을 **for-android-app**으로 설정하십시오. 
   1. **추가**하십시오. 
3. **인증 정보 보기**를 클릭하여 인증 정보를 보십시오. 모바일 애플리케이션에 필요한 **API 키**와 **URL**을 기록하십시오. 

{{site.data.keyword.speechtotextshort}} 서비스는 사람의 음성을 {{site.data.keyword.Bluemix_short}}의 {{site.data.keyword.conversationshort}} 서비스에 입력으로서 전송할 수 있는 텍스트로 변환합니다. 

1. [**{{site.data.keyword.Bluemix_notm}} 카탈로그**](https://{DomainName}/catalog/)로 이동하여 [{{site.data.keyword.speechtotextshort}}](https://{DomainName}/catalog/services/speech-to-text) 서비스 > **Lite** 플랜을 선택하십시오. 
   1. **이름**을 **android-chatbot-stt**로 설정하십시오. 
   1. **작성**하십시오. 
2. 왼쪽 분할창에서 **서비스 인증 정보**를 클릭하고 **새 인증 정보**를 클릭하여 새 인증 정보를 추가하십시오. 
   1. **이름**을 **for-android-app**으로 설정하십시오. 
   1. **추가**하십시오. 
3. **인증 정보 보기**를 클릭하여 인증 정보를 보십시오. 모바일 애플리케이션에 필요한 **API 키**와 **URL**을 기록하십시오. 

{{site.data.keyword.texttospeechshort}} 서비스는 텍스트 및 자연어를 처리하여 적절한 억양 및 음조가 있는 합성된 오디오 출력을 생성합니다. 이 서비스는 몇 가지 음성을 제공하며 Android 앱에서 구성할 수 있습니다. 

1. [**{{site.data.keyword.Bluemix_notm}} 카탈로그**](https://{DomainName}/catalog/)로 이동하여 [{{site.data.keyword.texttospeechshort}}](https://{DomainName}/catalog/services/text-to-speech) 서비스 > **Lite** 플랜을 선택하십시오. 
   1. **이름**을 **android-chatbot-tts**로 설정하십시오. 
   1. **작성**하십시오. 
2. 왼쪽 분할창에서 **서비스 인증 정보**를 클릭하고 **새 인증 정보**를 클릭하여 새 인증 정보를 추가하십시오. 
   1. **이름**을 **for-android-app**으로 설정하십시오. 
   1. **추가**하십시오. 
3. **인증 정보 보기**를 클릭하여 인증 정보를 보십시오. 모바일 애플리케이션에 필요한 **API 키**와 **URL**을 기록하십시오. 

## 스킬 작성
{: #create_workspace}

스킬은 대화 플로우를 정의하는 아티팩트의 컨테이너입니다. 

이 튜토리얼에서는 사전 정의된 인텐트, 엔티티 및 대화 플로우를 포함하는 [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) 파일을 시스템에 저장하여 사용합니다. 

1. {{site.data.keyword.conversationshort}} 서비스 세부사항 페이지에서 왼쪽 분할창의 **관리**로 이동한 후 **도구 실행**을 클릭하여 {{site.data.keyword.conversationshort}} 대시보드를 보십시오. 
1. **스킬** 탭을 클릭하십시오. 
1. **새로 작성** 및 **스킬 가져오기**를 클릭한 후 위에서 다운로드한 JSON 파일을 선택하십시오. 
1. **모두** 옵션을 선택하고 **가져오기**를 클릭하십시오. 사전 정의된 인텐트, 엔티티 및 대화 플로우를 포함하는 새 스킬이 작성됩니다. 
1. 스킬 목록으로 되돌아가십시오. `Ana` 스킬에 대한 조치 메뉴를 선택하고 **API 세부사항 보기**를 선택하십시오. 

### 인텐트 정의
{:#define_intent}

인텐트는 질문에 대한 답변이나 청구서 지불 처리와 같은 사용자 입력의 목적을 나타냅니다. 사용자는 애플리케이션이 지원하도록 할 각 사용자 요청 유형에 대해 인텐트를 정의합니다. {{site.data.keyword.conversationshort}} 서비스는 사용자 입력에서 표현된 인텐트를 인식하여 이에 응답하기 위한 올바른 대화 플로우를 선택할 수 있습니다. 이 도구에서 인텐트의 이름에는 항상 접두부로 `#` 문자가 포함됩니다. 

간단히 말하면, 인텐트는 일반 사용자의 의도입니다. 인텐트 이름의 예는 다음과 같습니다. 
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. 새로 작성된 스킬인 **Ana**를 클릭하십시오. 

   Ana는 사용자가 의료 혜택에 대해 질문하고 보험료를 청구하는 데 사용하는 보험 관련 봇입니다.
   {:tip}
2. 첫 번째 탭을 클릭하여 모든 **인텐트**를 보십시오. 
3. **인텐트 추가**를 클릭하여 새 인텐트를 추가하십시오. `#` 뒤에 `Cancel_Policy`를 인텐트 이름으로 입력하고 선택적 설명을 제공하십시오.
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. **인텐트 작성**을 클릭하십시오. 
5. 보험 취소가 요청된 경우의 사용자 예를 추가하십시오. 
   - `I want to cancel my policy`
   - `Drop my policy now`
   - `I wish to stop making payments on my policy.`
6. 사용자 예를 한 번에 하나씩 추가하고 **예 추가**를 클릭하십시오. 다른 모든 사용자 예에 대해 이를 반복하십시오. 

   봇 훈련도를 높이려면 5개 이상의 사용자 예를 추가하는 것이 좋습니다.
   {:tip}

7. 인텐트 이름 옆에 있는 **닫기** ![](images/solution28-watson-chatbot-android/close_icon.png) 단추를 클릭하여 인텐트를 저장하십시오. 
8. **컨텐츠 카탈로그**를 클릭하고 **일반**을 선택하십시오. **스킬에 추가**를 클릭하십시오. 

   컨텐츠 카탈로그는 기존 인텐트(은행 업무, 고객 응대, 보험, 통신, 전자상거래 등)를 추가하여 작업을 더 빨리 시작할 수 있게 도와줍니다. 이러한 인텐트는 사용자가 물을 수 있는 일반적인 질문으로 훈련되어 있습니다.
   {:tip}

### 엔티티 정의
{:#define_entity}

엔티티는 인텐트와 관련되어 있으며 인텐트에 특정 컨텍스트를 제공하는 단어 또는 대상을 나타냅니다. 사용자는 각 엔티티에 대해 입력될 수 있는 가능한 값과 해당 동의어를 나열합니다. {{site.data.keyword.conversationshort}} 서비스는 사용자 입력에 언급된 엔티티를 인식하여, 인텐트를 이행하기 위해 취할 특정 조치를 선택할 수 있습니다. 이 도구에서 엔티티의 이름에는 항상 접두부로 `@` 문자가 포함됩니다. 

엔티티 이름의 예는 다음과 같습니다. 
 - `@location`
 - `@menu_item`
 - `@product`

1. **엔티티** 탭을 클릭하여 기존 엔티티를 보십시오. 
2. **엔티티 추가**를 클릭하고 `@` 뒤에 `location`을 엔티티 이름으로 입력하십시오. **엔티티 작성**을 클릭하십시오. 
3. `address`를 값 이름으로 입력하고 **동의어**를 선택하십시오. 
4. `place`를 동의어로 추가하고 ![](images/solution28-watson-chatbot-android/plus_icon.png) 아이콘을 클릭하십시오. 동의어 `office`, `centre`, `branch` 등에 대해 이를 반복하고 **값 추가**를 클릭하십시오.
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. **닫기** ![](images/solution28-watson-chatbot-android/close_icon.png)를 클릭하여 변경사항을 저장하십시오. 
6. **시스템 엔티티** 탭을 클릭하여 어느 유스 케이스에도 사용할 수 있는, IBM에서 작성한 일반 엔티티를 확인하십시오. 

   시스템 엔티티는 해당 엔티티가 나타내는 대상 유형의 값을 광범위하게 인식하기 위해 사용할 수 있습니다. 예를 들어, `@sys-number` 시스템 엔티티는 정수, 소수, 글로 씌여진 숫자를 포함한 모든 숫자 값과 대응합니다.
   {:tip}
7. @sys-person 및 @sys-location 시스템 엔티티에 대해 **상태**를 off에서 `on`으로 전환하십시오. 

### 대화 플로우 빌드
{:#build_dialog}

대화는 애플리케이션이 정의된 인텐트 및 엔티티를 인식하는 경우 응답하는 방식을 정의하는, 분기하는 대화 플로우입니다. 사용자는 도구의 대화 빌더를 사용하여 사용자 입력에서 인식된 인텐트 및 엔티티에 대한 응답을 제공함으로써 사용자와의 대화를 작성합니다. 

1. **대화 상자** 탭을 클릭하여 인텐트 및 엔티티를 포함하는 기존 대화 플로우를 보십시오. 
2. **노드 추가**를 클릭하여 대화 상자에 새 노드를 추가하십시오. 
3. **어시스턴트가 다음 항목을 인식하는 경우**에 `#Cancel_Policy`를 입력하십시오. 
4. **다음 항목으로 응답**에 응답 `This facility is not available online. Please visit our nearest branch to cancel your policy.`를 입력하십시오. 
5. ![](images/solution28-watson-chatbot-android/save_node.png)를 클릭하여 닫고 노드를 저장하십시오. 
6. 스크롤하여 `#greeting` 노드를 보십시오. 이 노드를 클릭하여 세부사항을 보십시오.
   ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. ![](images/solution28-watson-chatbot-android/add_condition.png) 아이콘을 클릭하여 **새 조건을 추가하십시오**. 드롭 다운에서 `or`를 선택하고 `#General_Greetings`를 인텐트로 입력하십시오. **다음 항목으로 응답** 섹션이 사용자가 인사하는 경우의 어시스턴트 응답을 표시합니다.
   ![](images/solution28-watson-chatbot-android/apply_condition.png)

   컨텍스트 변수는 노드에 정의하는 변수이며 선택적으로 기본값을 지정할 수 있습니다. 나중에 다른 노드 또는 애플리케이션 로직이 컨텍스트 변수의 값을 설정하거나 변경할 수 있습니다. 애플리케이션은 대화에 정보를 전달할 수 있으며, 대화는 이 정보를 업데이트하여 애플리케이션 또는 후속 노드에 다시 전달할 수 있습니다. 대화는 컨텍스트 변수를 사용하여 이를 수행합니다.
   {:tip}

8. **사용해 보기** 단추를 사용하여 대화 플로우를 테스트하십시오. 

## 스킬을 어시스턴트에 연결

**어시스턴트**는 사용자의 비즈니스 요구사항에 따라 사용자 정의하고, 고객이 필요한 시점에 필요한 항목에 대해 도움을 받을 수 있도록 여러 채널에 배치할 수 있는 코그너티브 봇입니다. 사용자는 고객의 의도를 만족시키는 데 필요한 **스킬**을 어시스턴트에 추가하여 이를 사용자 정의합니다. 

1. {{site.data.keyword.conversationshort}} 도구에서 **어시스턴트**로 전환하여 **새로 작성**을 사용하십시오. 
   1. **이름**을 **android-chatbot-assistant**로 설정하십시오. 
   1. **작성**하십시오. 
1. **대화 스킬 추가**를 사용하여 이전 절에서 작성된 스킬을 선택하십시오. 
   1. **기존 스킬을 추가**하십시오. 
   1. **Ana**를 선택하십시오. 
1. 어시스턴트의 조치 메뉴 > **설정** > **API 세부사항**을 선택하고 **어시스턴트 ID**를 기록하십시오. 모바일 애플리케이션(Android 앱의 `config.xml` 파일)에서 이를 참조해야 할 수 있습니다. 

## Android 앱 구성 및 실행
{:#configure_run_android_app}

저장소는 필요한 Gradle 종속 항목을 포함하는 Android 애플리케이션 코드를 포함하고 있습니다. 

1. 아래 명령을 실행하여 [GitHub 저장소](https://github.com/IBM-Cloud/chatbot-watson-android)를 복제하십시오. 
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. Android Studio > **기존 Android Studio 프로젝트 열기**를 실행하고 다운로드한 코드를 지정하십시오. **Gradle** 빌드가 자동으로 트리거되며 모든 종속 항목이 다운로드됩니다. 
3. `app/src/main/res/values/config.xml`을 열어 서비스 인증 정보에 대한 플레이스홀더(`ASSISTANT_ID_HERE`)를 보십시오. 이전에 저장한 서비스 인증 정보를 각 플레이스홀더에 입력하고 파일을 저장하십시오. 
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <!--Watson Assistant service credentials-->
       <!-- REPLACE `ASSISTANT_ID_HERE` with ID of the Assistant to use -->
       <string name="assistant_id">ASSISTANT_ID_HERE</string>

       <!-- REPLACE `ASSISTANT_API_KEY_HERE` with Watson Assistant service API Key-->
       <string name="assistant_apikey">ASSISTANT_API_KEY_HERE</string>

       <!-- REPLACE `ASSISTANT_URL_HERE` with Watson Assistant service URL-->
       <string name="assistant_url">ASSISTANT_URL_HERE</string>

       <!--Watson Speech To Text(STT) service credentials-->
       <!-- REPLACE `STT_API_KEY_HERE` with Watson Speech to Text service API Key-->
       <string name="STT_apikey">STT_API_KEY_HERE</string>

       <!-- REPLACE `STT_URL_HERE` with Watson Speech to Text service URL-->
       <string name="STT_url">STT_URL_HERE</string>

       <!--Watson Text To Speech(TTS) service credentials-->
       <!-- REPLACE `TTS_API_KEY_HERE` with Watson Text to Speech service API Key-->
       <string name="TTS_apikey">TTS_API_KEY_HERE</string>

       <!-- REPLACE `TTS_URL_HERE` with Watson Text to Speech service URL-->
       <string name="TTS_url">TTS_URL_HERE</string>
   </resources>
   ```
4. 프로젝트를 빌드하고 애플리케이션을 실제 디바이스 또는 시뮬레이터에서 시작하십시오. 
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. 아래에 제공된 공간에서 **질문을 입력**하고 화살표 아이콘을 클릭하여 질문을 {{site.data.keyword.conversationshort}} 서비스에 전송하십시오. 
6. 응답이 {{site.data.keyword.texttospeechshort}} 서비스에 전달되며 응답을 읽는 음성을 들을 수 있습니다. 
7. 앱의 왼쪽 아래에 있는 **마이크** 아이콘을 클릭하여 말을 입력하십시오. 화살표 아이콘을 클릭하면 이것이 텍스트로 변환되어 {{site.data.keyword.conversationshort}} 서비스에 전송됩니다. 


## 리소스 제거
{:removeresources}

1. [리소스 목록](https://{DomainName}/resources/)으로 이동하십시오. 
1. 작성한 서비스를 삭제하십시오. 
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## 관련 컨텐츠
{:related}

- [엔티티, 동의어, 시스템 엔티티 작성](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [컨텍스트 변수](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [복잡한 대화 빌드](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [슬롯을 사용하여 정보 수집](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [배치 옵션](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [스킬 개선](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
