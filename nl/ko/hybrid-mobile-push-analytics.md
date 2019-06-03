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

# Push Notifications를 사용하는 하이브리드 모바일 애플리케이션
{: #hybrid-mobile-push-analytics}

{{site.data.keyword.Bluemix_notm}}의 {{site.data.keyword.mobilepushshort}}과 같은 고가치 모바일 서비스를 사용하여 하이브리드 Cordova 애플리케이션을 빠르게 작성하는 것이 얼마나 쉬운지 알아보십시오. 

Apache Cordova는 오픈 소스 모바일 개발 프레임워크입니다. 이는 크로스 플랫폼 개발에 표준 웹 기술(HTML5, CSS3 및 JavaScript)을 사용할 수 있게 해 줍니다. 애플리케이션은 각 플랫폼을 대상으로 하는 랩퍼에서 실행되며, 표준 준수 API 바인딩에 의존하여 센서, 데이터, 네트워크 상태와 같은 각 디바이스의 기능에 액세스합니다. 

이 튜토리얼에서는 Cordova 모바일 스타터 애플리케이션 작성, 모바일 서비스 추가, 클라이언트 SDK 설정, 스캐폴딩으로 작성된 코드 다운로드, 그리고 애플리케이션의 기능 확장을 안내합니다. 

## 목표

* {{site.data.keyword.mobilepushshort}} 서비스를 사용하는 모바일 프로젝트를 작성합니다. 
* APNs 및 FCM 인증 정보를 얻는 방법에 대해 학습합니다. 
* 코드를 다운로드하고 필수 설정을 완료합니다. 
* {{site.data.keyword.mobilepushshort}}을 구성하고, 전송하고 모니터합니다. 

 ![](images/solution15/Architecture.png)

## 제품

이 튜토리얼에서는 다음과 같은 제품을 사용합니다. 
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 시작하기 전에
{: #prereqs}

- Cordova 명령을 실행하는 데 필요한 Cordova [CLI](https://cordova.apache.org/docs/en/latest/guide/cli/)가 있어야 합니다. 
- Cordova-iOS [전제조건](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html) 및 Cordova-Android [전제조건](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html)을 만족해야 합니다. 
- 발신인 ID 및 서버 API 키를 얻기 위해 Firebase 콘솔에 로그인하는 데 필요한 Google 계정이 있어야 합니다. 
- {{site.data.keyword.Bluemix_notm}}의 {{site.data.keyword.mobilepushshort}} 서비스 인스턴스(제공자)에서 iOS 디바이스 및 애플리케이션으로 원격 알림을 전송하는 데 필요한 [Apple Developer](https://developer.apple.com/) 계정이 있어야 합니다. 
- 코드를 가져오고 확장하는 데 필요한 Xcode 및 Android Studio가 있어야 합니다. 

## 스타터 킷으로부터 Cordova 모바일 프로젝트 작성
{: #get_code}
{{site.data.keyword.Bluemix_notm}} 모바일 대시보드는 스타터 킷으로부터 프로젝트를 작성하여 모바일 앱 개발을 신속히 진행할 수 있게 해 줍니다. 
1. [모바일 대시보드](https://{DomainName}/developer/mobile/dashboard)로 이동하십시오. 
2. **스타터 킷**을 클릭하고 **앱 작성**을 클릭하십시오.
    ![](images/solution15/mobile_dashboard.png)
3. 프로젝트 이름을 입력하십시오. 이를 앱 이름으로 사용할 수도 있습니다. 
4. 플랫폼으로 **Cordova**를 선택하고 **작성**을 클릭하십시오. 

    ![](images/solution15/create_cordova_project.png)
5. **리소스 추가** > 모바일> **푸시 알림**을 클릭하고 서비스를 프로비저닝할 위치, 리소스 그룹 및 **Lite** 가격 플랜을 선택하십시오. 
6. **작성**을 클릭하여 {{site.data.keyword.mobilepushshort}} 서비스를 프로비저닝하십시오. **앱** 탭에서 새 앱이 작성됩니다. 

    **참고:** {{site.data.keyword.mobilepushshort}} 서비스가 비어 있는 스타터에 추가되어 있어야 합니다. 

다음 단계에서는 스캐폴딩으로 작성된 코드를 다운로드하고 필수 설정을 완료합니다. 

## 코드 다운로드 및 필수 설정 완료
{: #download_code}

코드를 아직 다운로드하지 않은 경우에는 {{site.data.keyword.Bluemix_notm}} 모바일 대시보드를 사용하여 프로젝트 > **사용자의 모바일 프로젝트**에서 **코드 다운로드**를 클릭함으로써 코드를 가져오십시오. 

1. 원하는 IDE에서 `/platforms/android/project.properties`로 이동하여 마지막 두 행(library.1 및 library.2)을 아래 행으로 대체하십시오. 

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
  위 변경사항은 Android에만 해당됩니다.
  {: tip}
2. 앱을 Android 에뮬레이터에서 실행하려면 터미널 또는 명령 프롬프트에서 아래 명령을 실행하십시오. 
```
 $ cordova build android
 $ cordova run android
```
 오류가 표시되는 경우에는 에뮬레이터를 실행하고 위 명령을 실행해 보십시오.
 {: tip}
3. 앱을 iOS 시뮬레이터에서 미리 보려면 터미널에서 아래 명령을 실행하십시오. 

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    `config.xml` 파일에 있는 프로젝트 이름은 `cordova info` 명령을 실행하여 찾을 수 있습니다.
    {: tip}

## FCM 및 APNs 인증 정보 얻기
{: #obtain_fcm_apns_credentials}

 ### FCM(Firebase Cloud Messaging) 구성

  1. [Firebase 콘솔](https://console.firebase.google.com)에서 새 프로젝트를 작성하십시오. 이름을 **hybridmobileapp**으로 설정하십시오. 
  2. 프로젝트 **설정**으로 이동하십시오. 
  3. **일반** 탭에서 두 애플리케이션을 추가하십시오. 
       1. 패키지 이름이 **com.ibm.mobilefirstplatform.clientsdk.android.push**로 설정된 애플리케이션
       2. 패키지 이름이 **io.cordova.hellocordovastarter**로 설정된 애플리케이션
  4. **클라우드 메시징** 탭에서 발신인 ID 및 서버 키(API 키라고도 함)를 찾으십시오. 
  5. {{site.data.keyword.mobilepushshort}} 서비스 대시보드에서 발신인 ID 및 API 키의 값을 설정하십시오. 


자세한 단계는 [FCM 인증 정보 얻기](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials)를 참조하십시오.
{: tip}

### Apple {{site.data.keyword.mobilepushshort}} service(APNs) 구성

  1. [Apple Developer](https://developer.apple.com/) 포털로 이동하여 App ID를 등록하십시오. 
  2. 개발 및 배포 APNs SSL 인증서를 작성하십시오. 
  3. 개발 프로비저닝 프로파일을 작성하십시오. 
  4. {{site.data.keyword.Bluemix_notm}}의 {{site.data.keyword.mobilepushshort}} 서비스 인스턴스를 구성하십시오. 

자세한 단계는 [APNs 인증 정보 얻기 및 {{site.data.keyword.mobilepushshort}} 서비스 구성](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials)을 참조하십시오.
{: tip}

## {{site.data.keyword.mobilepushshort}}의 구성, 전송 및 모니터링
{: #configure_push}

1. index.js의 `onDeviceReady` 함수에서 값 `{pushAppGuid}` 및

   `{pushClientSecret}`을 푸시 서비스 **인증 정보**(*appGuid* 및 *clientSecret*)로 대체하십시오. 

2. 모바일 대시보드 > 프로젝트 > Cordova 프로젝트로 이동하십시오. {{site.data.keyword.mobilepushshort}} 서비스를 클릭하고 아래 단계를 따르십시오. 

### APNs - 서비스 인스턴스 구성

알림을 전송하기 위해 {{site.data.keyword.mobilepushshort}} 서비스를 사용하려면 위 단계에서 작성한 .p12 인증서를 업로드하십시오. 이 인증서는 애플리케이션을 빌드하고 공개하는 데 필요한 개인 키 및 SSL 인증서를 포함합니다. 

**참고:** `.cer` 파일이 Keychain Access에 저장되고 나면 이를 자신의 컴퓨터로 내보내 `.p12` 인증서를 작성하십시오. 

1. 서비스 섹션의 `{{site.data.keyword.mobilepushshort}}`을 클릭하거나 {{site.data.keyword.mobilepushshort}} 서비스 옆에 세로로 나열된 세 개의 점을 클릭하고 `대시보드 열기`를 선택하십시오. 
2. {{site.data.keyword.mobilepushshort}} 대시보드의 `관리 > 알림 전송`에서 `구성` 옵션을 볼 수 있습니다. 

`푸시 알림 서비스` 콘솔에서 APNs를 설정하려면 다음 단계를 완료하십시오. 

1. 푸시 알림 서비스 대시보드에서 `구성`을 선택하십시오. 
2. `모바일 옵션`을 선택하여 APNs 푸시 인증 정보 양식의 정보를 업데이트하십시오. 
3. `샌드박스(개발)` 또는 `프로덕션(배포)` 중 해당되는 것을 선택한 후 작성한 `p.12` 인증서를 업로드하십시오. 
4. 비밀번호 필드에 .p12 인증서와 연관된 비밀번호를 입력한 후 저장을 클릭하십시오. 

### FCM - 서비스 인스턴스 구성

1. **모바일**을 선택한 후 GCM/FCM 푸시 인증 정보 탭을 Firebase 콘솔에서 처음에 작성한 발신인 ID/프로젝트 번호 및 API 키(서버 키)로 업데이트하십시오. 
2. **저장**을 클릭하십시오. {{site.data.keyword.mobilepushshort}} 서비스가 구성됩니다. 

### {{site.data.keyword.mobilepushshort}} 전송

1. **알림 전송**을 선택하고 전송 옵션을 선택하여 메시지를 작성하십시오. 지원되는 옵션은 태그별 디바이스, 디바이스 ID별 디바이스, 사용자 ID별 디바이스, Android 디바이스, IOS 디바이스, 웹 알림 및 모든 디바이스입니다.
   **참고:** **모든 디바이스** 옵션을 선택하면 {{site.data.keyword.mobilepushshort}}을 구독하는 모든 디바이스가 알림을 수신합니다. 

2. **메시지** 필드에서 메시지를 작성하십시오. 필요에 따라 선택적 설정을 구성하십시오. 
3. **전송**을 클릭하고 실제 디바이스가 알림을 수신했는지 확인하십시오. 

### 전송한 알림 모니터링

**모니터링** 섹션으로 이동하여 전송한 알림을 모니터할 수 있습니다.
IBM {{site.data.keyword.mobilepushshort}} 서비스는 이제 사용자 데이터로부터 그래프를 생성하여 푸시 성능을 모니터할 수 있도록 기능이 확장되었습니다. 사용자는 이 유틸리티를 사용하여 전송된 모든 {{site.data.keyword.mobilepushshort}}을 나열하거나, 모든 등록된 디바이스를 나열하거나, 일별, 주별 또는 월별로 정보를 분석할 수 있습니다.
 ![](images/solution6/monitoring_messages.png)

## 관련 컨텐츠
{: #related_content}

- [태그 기반 알림](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}}의 보안](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

