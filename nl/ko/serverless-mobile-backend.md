---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: data-hd-operatingsystem="ios"}
{:android: data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 서버리스 백엔드를 사용하는 모바일 애플리케이션
{: #serverless-mobile-backend}

이 튜토리얼에서는 코그너티브 및 데이터 서비스와 함께 {{site.data.keyword.openwhisk}}를 사용하여 모바일 애플리케이션을 위해 서버리스 백엔드를 빌드하는 방법에 대해 학습합니다.
{:shortdesc}

일부 모바일 개발자는 서버 측 로직 또는 시작할 서버를 관리해 본 경험이 없습니다. 이러한 개발자는 빌드 중인 앱에 집중하길 선호합니다. 이러한 개발자가 기존 개발 스킬을 재사용하여 모바일 백엔드를 작성할 수 있다면 어떻겠습니까? 

{{site.data.keyword.openwhisk_short}}는 서버리스 이벤트 중심 플랫폼입니다. [이 예에서 강조하는 것과 같이](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp) 배치하는 조치는 *웹 조치*로서 HTTP 엔드포인트로 쉽게 전환되어 웹 애플리케이션 백엔드 API를 빌드할 수 있습니다. 웹 애플리케이션은 REST API에 대한 클라이언트이므로 쉽게 이 예를 한 단계 더 진행하여 모바일 앱을 위한 백엔드를 빌드하기 위해 동일한 접근 방식을 적용할 수 있습니다. 또한 {{site.data.keyword.openwhisk_short}}를 사용하여 모바일 개발자는 모바일 앱, Android용 Java 및 iOS용 Swift에 사용되는 언어와 동일한 언어로 조치를 작성할 수 있습니다. 

이 튜토리얼은 대상 플랫폼을 기반으로 구성할 수 있습니다. 이 튜토리얼의 **iOS/Swift** 버전에 대한 문서를 현재 보고 있습니다. 이 문서의 맨 위에 있는 탭을 사용하여 이 튜토리얼의 **Android/Java** 버전을 선택하십시오.
{: swift}

이 튜토리얼은 대상 플랫폼을 기반으로 구성할 수 있습니다. 이 튜토리얼의 **Android/Java** 버전에 대한 문서를 현재 보고 있습니다. 이 문서의 맨 위에 있는 탭을 사용하여 이 튜토리얼의 **iOS/Swift** 버전을 선택하십시오.
{: java}

## 목표
{: #objectives}

* {{site.data.keyword.openwhisk_short}}를 사용하여 서버리스 모바일 백엔드를 배치합니다. 
* {{site.data.keyword.appid_short}}를 사용하여 모바일 앱에 사용자 인증을 추가합니다. 
* {{site.data.keyword.toneanalyzershort}}를 사용하여 사용자 피드백을 분석합니다. 
* {{site.data.keyword.mobilepushshort}}를 사용하여 알림을 전송합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

이 튜토리얼에 표시된 애플리케이션은 피드백 텍스트의 어조를 정확하게 분석하고 {{site.data.keyword.mobilepushshort}}를 통해 고객에게 적절하게 알리는 피드백 앱입니다. 

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. 사용자가 [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)에 대해 인증합니다. {{site.data.keyword.appid_short}}는 액세스 및 식별 토큰을 제공합니다. 
2. 백엔드 API에 대한 추가 호출에는 액세스 토큰이 포함되어 있습니다. 
3. [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)를 사용하여 백엔드가 구현됩니다. 웹 조치로 노출된 서버리스 조치는 토큰이 요청 헤더에 전송될 것으로 예상하며 실제 API에 대한 액세스를 허용하기 전에 유효성(서명 및 만기 날짜)을 확인합니다. 
4. 사용자가 피드백을 제출하면 피드백이 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)에 저장됩니다. 
5. [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)를 사용하여 피드백 텍스트가 처리됩니다. 
6. 분석 결과를 기반으로 [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)를 사용하여 알림이 사용자에게 다시 전송됩니다. 
7. 사용자가 디바이스에 대한 알림을 수신합니다. 

## 시작하기 전에
{: #prereqs}

이 튜토리얼에서는 {{site.data.keyword.Bluemix_notm}} 명령행 도구를 사용하여 리소스를 프로비저닝하고 코드를 배치합니다. `ibmcloud` 명령행 도구를 설치했는지 확인하십시오. 

* [{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - ibmcloud CLI 및 필수 플러그인(Cloud Foundry 및 {{site.data.keyword.openwhisk_short}})을 설치하는 스크립트

또한 다음과 같은 소프트웨어 및 계정이 필요합니다. 

   1. Java 8
   2. Android Studio 2.3.3
   3. Firebase Cloud Messaging을 구성하는 데 필요한 Google Developer 계정
   4. Bash 쉘, cURL
   {: java}


   1. Xcode
   2. Apple Push Notification Service를 구성하는 데 필요한 Apple Developer 계정
   3. Bash 쉘, cURL
   {: swift}

이 튜토리얼에서는 애플리케이션에 대한 푸시 알림을 구성합니다. 이 튜토리얼에서는 [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) 또는 [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics)용 기본 {{site.data.keyword.mobilepushshort}} 튜토리얼을 완료했고 Firebase Cloud Messaging 또는 Apple Push Notification Service의 구성에 익숙하다고 가정합니다.
{:tip}

Windows 10 사용자가 명령행 명령어에 대해 작업할 수 있도록 [이 기사](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10)에 설명된 대로 Linux 및 Ubuntu용 Windows Subsystem을 설치하도록 권장합니다.
{: tip}
{: java}

## 애플리케이션 코드 가져오기

저장소에는 모바일 애플리케이션과 {{site.data.keyword.openwhisk_short}} 조치 코드가 둘 다 포함되어 있습니다. 

1. GitHub 저장소에서 코드를 체크아웃하십시오. 

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-android
   ```
   {: pre}
   {: java}

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-ios
   ```
   {: pre}
   {: swift}

2. 코드 구조를 검토하십시오. 

| 파일                                     |설명                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions) | 서버리스 모바일 백엔드의 {{site.data.keyword.openwhisk_short}} 조치에 대한 코드 |
| [**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android) | 모바일 애플리케이션에 대한 코드 |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh) | {{site.data.keyword.openwhisk_short}} 트리거, 조치 및 규칙을 설치, 설치 제거 및 업데이트하는 헬퍼 스크립트 |
{: java}

| 파일                                     |설명                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions) | 서버리스 모바일 백엔드의 {{site.data.keyword.openwhisk_short}} 조치에 대한 코드 |
| [**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp) | 모바일 애플리케이션에 대한 코드 |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh) | {{site.data.keyword.openwhisk_short}} 트리거, 조치 및 규칙을 설치, 설치 제거 및 업데이트하는 헬퍼 스크립트 |
{: swift}

## 서비스를 프로비저닝하여 사용자 인증, 피드백 지속성 및 분석 처리
{: #provision_services}

이 절에서는 애플리케이션에서 사용하는 서비스를 프로비저닝합니다. `ibmcloud` 명령행을 사용하거나 {{site.data.keyword.Bluemix_notm}} 카탈로그에서 서비스를 프로비저닝하도록 선택할 수 있습니다. 

새 영역을 작성하여 서비스를 프로비저닝하고 서버리스 백엔드를 배치하도록 권장합니다. 이는 모든 리소스를 함께 보관하는 데 도움이 됩니다. 

### {{site.data.keyword.Bluemix_notm}} 카탈로그에서 서비스 프로비저닝

1. [{{site.data.keyword.Bluemix_notm}} 카탈로그](https://{DomainName}/catalog/)로 이동하십시오. 
2. **Lite** 플랜을 사용하여 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 서비스를 작성하십시오. 이름을 **serverlessfollowup-db**로 설정하십시오. 
3. **표준** 플랜을 사용하여 [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) 서비스를 작성하십시오. 이름을 **serverlessfollowup-tone**으로 설정하십시오. 
4. **등급화된 계층** 플랜을 사용하여 [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) 서비스를 작성하십시오. 이름을 **serverlessfollowup-appid**로 설정하십시오. 
5. **Lite** 플랜을 사용하여 [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) 서비스를 작성하십시오. 이름을 **serverlessfollowup-mobilepush**로 설정하십시오. 

### 명령행에서 서비스 프로비저닝

명령행에서 다음 명령을 실행하여 서비스를 프로비저닝하고 해당 인증 정보를 검색하십시오. 

   ```sh
   ibmcloud cf create-service cloudantNoSQLDB Lite serverlessfollowup-db
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service tone_analyzer standard serverlessfollowup-tone
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service AppID "Graduated tier" serverlessfollowup-appid
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service imfpush lite serverlessfollowup-mobilepush
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

## {{site.data.keyword.mobilepushshort}} 구성
{: #push_notifications}

사용자가 새 피드백을 제출하면 애플리케이션이 이 피드백을 분석한 후 사용자에게 알림을 다시 전송합니다. 사용자가 다른 태스크로 이동했거나 모바일 앱을 시작하지 않았을 수 있으므로 푸시 알림을 사용하는 것이 사용자와 통신하는 적절한 방법입니다. {{site.data.keyword.mobilepushshort}} 서비스를 사용하면 하나의 통합된 API를 통해 iOS 또는 Android 사용자에게 알림을 전송할 수 있습니다. 이 절에서는 대상 플랫폼을 위해 {{site.data.keyword.mobilepushshort}} 서비스를 구성합니다. 

### FCM(Firebase Cloud Messaging) 구성
{: java}

   1. [Firebase 콘솔](https://console.firebase.google.com)에서 새 프로젝트를 작성하십시오. 이름을 **serverlessfollowup**으로 설정하십시오. 
   2. 프로젝트 **설정**으로 이동하십시오. 
   3. **일반** 탭 아래에서 두 가지 애플리케이션을 추가하십시오. 
      1. 하나는 패키지 이름이 **com.ibm.mobilefirstplatform.clientsdk.android.push**로 설정됨됩니다. 
      2. 다른 하나는 패키지 이름이 **serverlessfollowup.app**으로 설정됩니다. 
   4. Firebase 콘솔에서 두 개의 정의된 애플리케이션이 포함된 `google-services.json`을 다운로드한 후 이 파일을 체크아웃 디렉토리의 `android/app` 폴더에 저장하십시오. 
   5. **Cloud Messaging** 탭 아래에서 발신인 ID 및 서버 키(이후에 API 키라고도 함)를 찾으십시오. 
   6. Push Notifications 서비스 대시보드에서 발신인 ID 및 API 키의 값을 설정하십시오.
   {: java}

### APNs(Apple Push Notifications Service) 구성
{: swift}

1. [Apple Developer](https://developer.apple.com/) 포털로 이동하여 App ID를 등록하십시오. 
2. 개발 및 배포 APNs SSL 인증서를 작성하십시오. 
3. 개발 프로비저닝 프로파일을 작성하십시오. 
4. {{site.data.keyword.Bluemix_notm}}에서 {{site.data.keyword.mobilepushshort}} 서비스 인스턴스를 구성하십시오. 자세한 단계는 [APNs 인증 정보 확보 및 {{site.data.keyword.mobilepushshort}} 서비스 구성](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-)을 참조하십시오.
{: swift}

## 서버리스 백엔드 배치
{: #serverless_backend}

모든 서비스가 구성되어 이제 서버리스 백엔드를 배치할 수 있습니다. 이 절에서는 다음과 같은 {{site.data.keyword.openwhisk_short}} 아티팩트가 작성됩니다. 

| 아티팩트                      |유형                           |설명                              |
| ----------------------------- | ------------------------------ | ---------------------------------------- |
| `serverlessfollowup`          | 패키지                         | 조치를 그룹화하고 모든 서비스 인증 정보를 보관하는 패키지 |
| `serverlessfollowup-cloudant` | 패키지 바인딩                  | 기본 제공 {{site.data.keyword.cloudant_short_notm}} 패키지에 바인드됨 |
| `serverlessfollowup-push`     | 패키지 바인딩                  | {{site.data.keyword.mobilepushshort}} 패키지에 바인드됨 |
| `auth-validate`               | 조치                           | 액세스 및 식별 토큰을 유효성 검증함 |
| `users-add`                   | 조치                           | 사용자 정보(ID, 이름, 이메일, 사진, 디바이스 ID)를 지속시킴 |
| `users-prepare-notify`        | 조치                           | {{site.data.keyword.mobilepushshort}}와 함께 사용할 메시지를 형식화함 |
| `feedback-put`                | 조치                           | 사용자 피드백을 데이터베이스에 저장함 |
| `feedback-analyze`            | 조치                           | {{site.data.keyword.toneanalyzershort}}를 사용하여 피드백을 분석함 |
| `users-add-sequence`          | 웹 조치로 노출된 시퀀스        | `auth-validate` 및 `users-add` |
| `feedback-put-sequence`       | 웹 조치로 노출된 시퀀스        | `auth-validate` 및 `feedback-put` |
| `feedback-analyze-sequence`   | 시퀀스                         |{{site.data.keyword.cloudant_short_notm}}의 `read-document`, `feedback-analyze`, `users-prepare-notify` 및 {{site.data.keyword.mobilepushshort}}를 사용한 `sendMessage` |
| `feedback-analyze-trigger`    | 트리거                         | 피드백이 데이터베이스에 저장될 때 {{site.data.keyword.openwhisk_short}}에 의해 호출됨 |
| `feedback-analyze-rule`       | 규칙                           | 트리거 `feedback-analyze-trigger`를 시퀀스 `feedback-analyze-sequence`와 연결함 |

### 코드 컴파일
{: java}
1. 체크아웃 디렉토리의 루트에서 조치 코드를 컴파일하십시오.
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### 조치 구성 및 배치
{: java}

2. template.local.env를 local.env에 복사하십시오. 

   ```sh
   cp template.local.env local.env
   ```
3. {{site.data.keyword.Bluemix_notm}} 대시보드(또는 이전에 실행한 ibmcloud 명령의 출력)에서 {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}} 및
{{site.data.keyword.appid_short}} 서비스에 대한 인증 정보를 가져온 후 `local.env`의 플레이스홀더를 해당 값으로 대체하십시오. 모든 조치가 데이터베이스에 액세스할 수 있도록 이 특성은 패키지에 삽입됩니다. 
4. 조치를 {{site.data.keyword.openwhisk_short}}에 배치하십시오. `deploy.sh`는 `local.env`의 인증 정보를 로드하여 {{site.data.keyword.cloudant_short_notm}} 데이터베이스(사용자, 피드백 및 무드)를 작성하고 애플리케이션에 대한 {{site.data.keyword.openwhisk_short}} 아티팩트를 배치합니다. 
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   튜토리얼을 완료하고 나면 `./deploy.sh --uninstall`을 사용하여 {{site.data.keyword.openwhisk_short}} 아티팩트를 제거할 수 있습니다.
   {: tip}

### 조치 구성 및 배치
{: swift}

1. template.local.env를 local.env에 복사하십시오. 
   ```sh
   cp template.local.env local.env
   ```
2. {{site.data.keyword.Bluemix_notm}} 대시보드(또는 이전에 실행한 ibmcloud 명령의 출력)에서 {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}} 및
{{site.data.keyword.appid_short}} 서비스에 대한 인증 정보를 가져온 후 `local.env`의 플레이스홀더를 해당 값으로 대체하십시오. 모든 조치가 데이터베이스에 액세스할 수 있도록 이 특성은 패키지에 삽입됩니다. 
3. 조치를 {{site.data.keyword.openwhisk_short}}에 배치하십시오. `deploy.sh`는 `local.env`의 인증 정보를 로드하여 {{site.data.keyword.cloudant_short_notm}} 데이터베이스(사용자, 피드백 및 무드)를 작성하고 애플리케이션에 대한 {{site.data.keyword.openwhisk_short}} 아티팩트를 배치합니다. 

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   튜토리얼을 완료하고 나면 `./deploy.sh --uninstall`을 사용하여 {{site.data.keyword.openwhisk_short}} 아티팩트를 제거할 수 있습니다.
   {: tip}

## 기본 모바일 애플리케이션을 구성 및 실행하여 사용자 피드백 수집
{: #mobile_app}

{{site.data.keyword.openwhisk_short}} 조치가 모바일 앱을 위해 준비가 되어 있습니다. 모바일 앱을 실행하기 전에 작성된 서비스를 대상으로 하도록 설정을 구성해야 합니다. 

1. Android Studio를 사용하여 체크아웃 디렉토리의 `android` 폴더에 있는 프로젝트를 여십시오. 
2. `android/app/src/main/res/values/credentials.xml`을 편집하고 인증 정보의 값으로 공백을 채우십시오. {{site.data.keyword.appid_short}} `tenantId`, {{site.data.keyword.mobilepushshort}} `appGuid` 및 `clientSecret`과 {{site.data.keyword.openwhisk_short}}가 배치된 조직 및 영역 이름이 필요합니다. API 호스트의 경우 명령 프롬프트 또는 터미널을 실행하거나 명령을 실행하십시오. 
   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
3. `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` 및 `LoginAndRegistrationListener.java`를 열고 서비스 인스턴스가 작성되는 위치에 따라 푸시 알림 서비스(BMSClient) 지역 및 AppID 지역을 업데이트하십시오. 
4. 프로젝트를 빌드하십시오. 
5. 에뮬레이터를 사용하거나 실제 디바이스에서 애플리케이션을 시작하십시오.
   에뮬레이터가 푸시 알림을 수신할 수 있도록 Google API가 있는 이미지를 선택하고 에뮬레이터 내에서 Google 계정으로 로그인해야 합니다.
   {: tip}
6. 백그라운드에서 {{site.data.keyword.openwhisk_short}}를 감시하십시오. 
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. 애플리케이션에서 **로그인**을 선택하여 Facebook 또는 Google 계정으로 인증하십시오. 로그인하고 나면 피드백 메시지를 입력하고 **피드백 보내기** 단추를 누르십시오. 피드백을 보낸 후 몇 초 뒤에 디바이스에 대한 푸시 알림을 수신해야 합니다. {{site.data.keyword.cloudant_short_notm}} 서비스 인스턴스의 `moods` 데이터베이스에서 템플리트 문서를 수정하여 알림 텍스트를 사용자 정의할 수 있습니다. 로그인 시 {{site.data.keyword.appid_short}}에 의해 생성된 액세스 및 식별 토큰을 검사하려면 **토큰 보기** 단추를 사용하십시오.
{: java}


1. 푸시 클라이언트 SDK 및 기타 SDK를 CocoaPods 및 Carthage에서 사용할 수 있습니다. 이 솔루션의 경우에는 CocoaPods를 사용합니다. 
2. 터미널을 열고 `followupapp` 폴더로 `cd`를 실행하십시오. 아래 명령을 실행하여 필요한 종속 항목을 설치하십시오. 
   ```sh
   pod install
   ```
   {: pre}
3. 체크아웃 디렉토리의 `followupapp` 폴더 아래에 있는 확장자가 `.xcworkspace`인 파일을 열어 Xcode에서 코드를 실행하십시오. 
4. `BMSCredentials.plist` 파일을 편집하고 인증 정보의 값으로 공백을 채우십시오. {{site.data.keyword.appid_short}} `tenantId`, {{site.data.keyword.mobilepushshort}} `appGuid` 및 `clientSecret`과 {{site.data.keyword.openwhisk_short}}가 배치된 조직 및 영역 이름이 필요합니다. API 호스트의 경우 터미널을 실행하고 명령을 실행하십시오. 

   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
5. `AppDelegate.swift`를 열고 서비스 인스턴스가 작성되는 위치에 따라 푸시 알림 서비스(BMSClient) 지역 및 AppID 지역을 업데이트하십시오. 
6. 프로젝트를 빌드하십시오. 
7. 에뮬레이터를 사용하거나 실제 디바이스에서 애플리케이션을 시작하십시오. 
8. 터미널에서 아래 명령을 실행하여 백그라운드에서 {{site.data.keyword.openwhisk_short}}를 감시하십시오. 
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. 애플리케이션에서 **로그인**을 선택하여 Facebook 또는 Google 계정으로 인증하십시오. 로그인하고 나면 피드백 메시지를 입력하고 **피드백 보내기** 단추를 누르십시오. 피드백을 보낸 후 몇 초 뒤에 디바이스에 대한 푸시 알림을 수신해야 합니다. {{site.data.keyword.cloudant_short_notm}} 서비스 인스턴스의 `moods` 데이터베이스에서 템플리트 문서를 수정하여 알림 텍스트를 사용자 정의할 수 있습니다. 로그인 시 {{site.data.keyword.appid_short}}에 의해 생성된 액세스 및 식별 토큰을 검사하려면 **토큰 보기** 단추를 사용하십시오.
{: swift}

## 리소스 제거

1. {{site.data.keyword.openwhisk_short}} 아티팩트를 제거하려면 `deploy.sh`를 사용하십시오. 

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. {{site.data.keyword.Bluemix_notm}} 콘솔에서 {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.appid_short}}, {{site.data.keyword.mobilepushshort}} 및 {{site.data.keyword.toneanalyzershort}} 서비스를 삭제하십시오. 

## 관련 컨텐츠

* {{site.data.keyword.appid_short}}는 ID 제공자의 초기 설정에 도움이 되는 기본 구성을 제공합니다. 앱을 공개하기 전에 [구성을 자체 인증 정보로 업데이트](https://{DomainName}/docs/services/appid?topic=appid-social#social)하십시오. [로그인 위젯을 사용자 정의](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget)할 수도 있습니다. 


* Swift 소스 파일(`actions` 폴더 아래의 .swift 파일)을 사용하여 OpenWhisk Swift 조치를 작성하는 경우에는 조치가 실행되기 전에 조치를 2진으로 컴파일해야 합니다. 완료되면 조치를 보유하는 컨테이너가 영구 제거될 때까지 조치에 대한 후속 호출이 훨씬 빨라집니다. 이 지연은 콜드 스타트 지연으로 알려져 있습니다.
  콜드 스타트 지연을 방지하기 위해 Swift 파일을 2진으로 컴파일한 후 zip 파일로 OpenWhisk에 업로드할 수 있습니다. OpenWhisk 스캐폴딩이 필요하므로 2진을 작성하는 가장 쉬운 방법은 실행되는 환경과 동일한 환경에서 빌드하는 것입니다. 자세한 단계는 [조치를 Swift 실행 파일로 패키지](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions)를 참조하십시오.
{: swift}
