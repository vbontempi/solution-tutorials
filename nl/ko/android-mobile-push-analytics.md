---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 푸시 알림을 사용하는 Android 기반 모바일 애플리케이션
{: #android-mobile-push-analytics}

{{site.data.keyword.Bluemix_notm}}의 {{site.data.keyword.mobilepushshort}}과 같은 고가치 모바일 서비스를 사용하여 Android 기반 애플리케이션을 빠르게 작성하는 것이 얼마나 쉬운지 알아보십시오. 

이 튜토리얼에서는 모바일 스타터 애플리케이션 작성, 모바일 서비스 추가, 클라이언트 SDK 설정, Android Studio로 코드 가져오기, 그리고 애플리케이션의 기능 확장을 안내합니다. 

## 목표
{: #objectives}

* {{site.data.keyword.mobilepushshort}} 서비스를 사용하는 모바일 앱을 작성합니다. 
* FCM 인증 정보을 얻습니다. 
* 코드를 다운로드하고 필수 설정을 완료합니다. 
* {{site.data.keyword.mobilepushshort}}을 구성하고, 전송하고 모니터합니다. 

![](images/solution9/Architecture.png)

## 제품
{: #products}

이 튜토리얼에서는 다음과 같은 제품을 사용합니다. 
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 시작하기 전에
{: #prereqs}

- 코드를 가져오고 확장하는 데 필요한 [Android Studio](https://developer.android.com/studio/index.html)가 있어야 합니다. 
- 발신인 ID 및 서버 API 키를 얻기 위해 Firebase 콘솔에 로그인하는 데 필요한 Google 계정이 있어야 합니다. 

## 스타터 킷으로부터 Android 모바일 앱 작성
{: #get_code}
{{site.data.keyword.Bluemix_notm}} 모바일 대시보드는 스타터 킷으로부터 앱을 작성하여 모바일 앱 개발을 신속히 진행할 수 있게 해 줍니다. 
1. [모바일 대시보드](https://{DomainName}/developer/mobile/dashboard)로 이동하십시오. 
2. **스타터 킷**을 클릭하고 **앱 작성**을 클릭하십시오.
    ![](images/solution9/mobile_dashboard.png)
3. 앱 이름을 입력하십시오. 이를 Android 프로젝트 이름으로 사용할 수도 있습니다. 
4. 플랫폼으로 **Android**를 선택하고 **작성**을 클릭하십시오. 

    ![](images/solution9/create_mobile_project.png)
5. **리소스 추가** > 모바일> **푸시 알림**을 클릭하고 서비스를 프로비저닝할 위치, 리소스 그룹 및 **Lite** 가격 플랜을 선택하십시오. 
6. **작성**을 클릭하여 {{site.data.keyword.mobilepushshort}} 서비스를 프로비저닝하십시오. **앱** 탭에서 새 앱이 작성됩니다. 

    **참고:** {{site.data.keyword.mobilepushshort}} 서비스가 비어 있는 스타터에 추가되어 있어야 합니다.
    다음 단계에서는 Firebase Cloud Messaging(FCM) 인증 정보를 얻습니다. 

다음 단계에서는 스캐폴딩으로 작성된 코드를 다운로드하고 푸시 Android SDK를 설정합니다. 

## 코드 다운로드 및 필수 설정 완료
{: #download_code}

코드를 아직 다운로드하지 않은 경우에는 {{site.data.keyword.Bluemix_notm}} 모바일 대시보드를 사용하여 앱 > **사용자의 모바일 앱**에서 **코드 다운로드**를 클릭함으로써 코드를 가져오십시오.
다운로드한 코드에는 **{{site.data.keyword.mobilepushshort}}** 클라이언트 SDK가 포함되어 있습니다. 이 클라이언트 SDK는 Gradle 및 Maven에서 사용할 수 있습니다. 이 튜토리얼에서는 **Gradle**을 사용합니다. 

1. Android Studio를 실행하고 **기존 Android Studio 프로젝트 열기**를 사용하여 다운로드한 코드를 지정하십시오. 
2. **Gradle** 빌드가 자동으로 트리거되며 모든 종속 항목이 다운로드됩니다. 
3. **Google Play 서비스** 종속성을 모듈 레벨 `build.gradle (Module: app)` 파일의 끝에 있는 `dependencies{.....}` 뒤에 추가하십시오. 
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. 작성한 `google-services.json` 파일을 복사하고 Android 애플리케이션 모듈 루트 디렉토리로 다운로드하십시오. `google-service.json` 파일이 추가된 패키지 이름을 포함하는 것을 볼 수 있습니다. 
5. 필수 권한은 `AndroidManifest.xml` 파일 및 종속 항목에 모두 포함되어 있습니다. 푸시 및 분석은 **build.gradle (Module: app)**에 포함되어 있습니다. 
6. `RECEIVE` 및 `REGISTRATION` 이벤트 알림에 대한 **Firebase Cloud Messaging(FCM)** 인텐트 서비스 및 인텐트 필터는 `AndroidManifest.xml`에 포함되어 있습니다. 

## FCM 인증 정보 얻기
{: #obtain_fcm_credentials}

Firebase Cloud Messaging(FCM)은 {{site.data.keyword.mobilepushshort}}을 Android 디바이스, Google Chrome 브라우저, Chrome 앱 & 확장 프로그램에 전달하는 데 사용되는 게이트웨이입니다. 콘솔에서 {{site.data.keyword.mobilepushshort}} 서비스를 설정하려면 FCM 인증 정보(발신인 ID 및 API 키)를 얻어야 합니다. 

API 키는 안전하게 저장되어 있고 {{site.data.keyword.mobilepushshort}} 서비스가 FCM 서버에 연결하는 데 사용하며, 발신인 ID(프로젝트 번호)는 클라이언트 측에서 Google Chrome용 Android SDK 및 JS SDK와 Mozilla Firefox가 사용합니다. FCM을 설정하고 인증 정보를 얻으려면 다음 단계를 완료하십시오. 

1. [Firebase 콘솔](https://console.firebase.google.com/?pli=1)에 방문하십시오. Google 사용자 계정이 필요합니다. 
2. **프로젝트 추가**를 선택하십시오. 
3. **프로젝트 작성** 창에서 프로젝트 이름을 제공하고 국가/지역을 선택한 후 **프로젝트 작성**을 클릭하십시오. 
4. 왼쪽 탐색 분할창에서 **설정**(**개요** 옆에 있는 설정 아이콘을 클릭) > **프로젝트 설정**을 선택하십시오. 
5. Cloud Messaging 탭을 선택하여 프로젝트 인증 정보(서버 API 키 및 발신인 ID)를 얻으십시오.
    **참고:**  FCM에서 나열되는 서버 키는 서버 API 키와 동일합니다.
    ![](images/solution9/fcm_console.png)

`google-services.json` 파일도 생성해야 합니다. 다음 단계를 완료하십시오. 

1. Firebase 콘솔에서 **프로젝트 설정** 아이콘을 클릭하고 위에서 작성한 프로젝트의 **일반** 탭에서 **Android 앱에 Firebase 추가**를 선택하십시오. 

    ![](images/solution9/firebase_project_settings.png)
2. **Android 앱에 Firebase 추가** 모달 창에서 {{site.data.keyword.mobilepushshort}} Android SDK 등록을 위한 패키지 이름으로 **com.ibm.mobilefirstplatform.clientsdk.android.push**를 추가하십시오. 앱 닉네임 및 SHA-1 필드는 선택사항입니다. **앱 등록** > **계속** > **완료**를 클릭하십시오. 

    ![](images/solution9/add_firebase_to_your_app.png)

3. **앱 추가** > **앱에 Firebase 추가**를 클릭하십시오. 패키지 이름 **com.ibm.mysampleapp**을 입력하여 애플리케이션의 패키지 이름을 포함시킨 후 Android 앱에 Firebase 추가 창으로 진행하십시오. 앱 닉네임 및 SHA-1 필드는 선택사항입니다. **앱 등록** > 계속 > 완료를 클릭하십시오.
     **참고:** 코드를 다운로드한 후에는 `AndroidManifest.xml` 파일에서 애플리케이션의 패키지 이름을 찾을 수 있습니다. 
4. **내 앱**에서 최신 구성 파일 `google-services.json`을 다운로드하십시오. 

    ![](images/solution9/google_services.png)

    **참고**: FCM은 Google Cloud Messaging(GCM)의 새 버전입니다. 새 앱에는 FCM 인증 정보를 사용해야 합니다. 기존 앱은 계속해서 GCM 구성으로 작동합니다. 

*단계 및 Firebase 콘솔 UI는 변경될 수 있습니다. 필요한 경우에는 Google 문서의 Firebase 부분을 참조하십시오. *

## {{site.data.keyword.mobilepushshort}}의 구성, 전송 및 모니터링

{: #configure_push}

1. {{site.data.keyword.mobilepushshort}} SDK를 이미 앱으로 가져왔으며, 푸시 초기화 코드를 `MainActivity.java` 파일에서 찾을 수 있습니다. 

    **참고:** 서비스 인증 정보는 `/res/values/credentials.xml` 파일의 일부입니다.
2. 알림 등록은 `MainActivity.java`에서 이뤄집니다. (선택사항) 고유 USER_ID를 제공하십시오. 
3. 앱을 실제 디바이스 또는 에뮬레이터에서 실행하여 알림을 수신하십시오. 
4. {{site.data.keyword.Bluemix_notm}} 모바일 대시보드의 **모바일 서비스** > **기존 서비스**에서 {{site.data.keyword.mobilepushshort}} 서비스를 열고 기본 {{site.data.keyword.mobilepushshort}}을 전송하려면 다음 단계를 완료하십시오. 
   - **관리** > **구성**을 클릭하십시오. 
   - **모바일**을 선택한 후 GCM/FCM 푸시 인증 정보 탭을 Firebase 콘솔에서 처음에 작성한 발신인 ID/프로젝트 번호 및 API 키(서버 키)로 업데이트하십시오. 

     ![](images/solution9/configure_push_notifications.png)
   - **저장**을 클릭하십시오. {{site.data.keyword.mobilepushshort}} 서비스가 구성됩니다. 
   - **알림 전송**을 선택하고 전송 옵션을 선택하여 메시지를 작성하십시오. 지원되는 옵션은 태그별 디바이스, 디바이스 ID별 디바이스, 사용자 ID별 디바이스, Android 디바이스, IOS 디바이스, 웹 알림 및 모든 디바이스입니다.
     **참고:** **모든 디바이스** 옵션을 선택하면 {{site.data.keyword.mobilepushshort}}을 구독하는 모든 디바이스가 알림을 수신합니다. 
   - **메시지** 필드에서 메시지를 작성하십시오. 필요에 따라 선택적 설정을 구성하십시오. 
   - **전송**을 클릭하고 실제 디바이스가 알림을 수신했는지 확인하십시오. 

     ![](images/solution9/android_send_notifications.png)
5. 사용자의 Android 디바이스에 알림이 표시됩니다. 

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. {{site.data.keyword.mobilepushshort}} 서비스의 **모니터링**으로 이동하여 전송한 알림을 모니터할 수 있습니다.
     IBM {{site.data.keyword.mobilepushshort}} 서비스는 이제 사용자 데이터로부터 그래프를 생성하여 푸시 성능을 모니터할 수 있도록 기능이 확장되었습니다. 사용자는 이 유틸리티를 사용하여 전송된 모든 {{site.data.keyword.mobilepushshort}}을 나열하거나, 모든 등록된 디바이스를 나열하거나, 일별, 주별 또는 월별로 정보를 분석할 수 있습니다.
      ![](images/solution6/monitoring_messages.png)

## 관련 컨텐츠
{: #related_content}
- [{{site.data.keyword.mobilepushshort}} 설정 사용자 정의](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [태그 기반 알림](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}}의 보안](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

