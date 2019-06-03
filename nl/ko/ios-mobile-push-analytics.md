---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-08"
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

# 푸시 알림을 사용하는 iOS 모바일 애플리케이션
{: #ios-mobile-push-analytics}

{{site.data.keyword.Bluemix_short}}의 {{site.data.keyword.mobilepushshort}}과 같은 고가치 모바일 서비스를 사용하여 iOS Swift 애플리케이션을 빠르게 작성하는 것이 얼마나 쉬운지 알아보십시오. 

이 튜토리얼에서는 모바일 스타터 애플리케이션 작성, 모바일 서비스 추가, 클라이언트 SDK 설정, Xcode로 코드 가져오기, 그리고 애플리케이션의 기능 확장을 안내합니다.
{:shortdesc: .shortdesc}

## 목표
{:#objectives}

- 기본 Swift 스타터 킷으로부터 {{site.data.keyword.mobilepushshort}} 및 {{site.data.keyword.mobileanalytics_short}} 서비스를 사용하여 모바일 앱을 작성합니다. 
- APNs 인증 정보를 얻고 {{site.data.keyword.mobilepushshort}} 서비스 인스턴스를 구성합니다. 
- 코드를 다운로드하고 클라이언트 SDK를 설정합니다. 
- {{site.data.keyword.mobilepushshort}}을 전송하고 모니터합니다. 

  ![](images/solution6/Architecture.png)

## 제품
{:#products}

이 튜토리얼에서는 다음과 같은 제품을 사용합니다. 
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 시작하기 전에
{: #prereqs}

1. {{site.data.keyword.Bluemix_short}}의 {{site.data.keyword.mobilepushshort}} 서비스 인스턴스(제공자)에서 iOS 디바이스 및 애플리케이션으로 원격 알림을 전송하는 데 필요한 [Apple Developer](https://developer.apple.com/) 계정이 있어야 합니다. 
2. 코드를 가져오고 확장하는 데 필요한 Xcode가 있어야 합니다. 

## 기본 Swift 스타터 킷으로부터 모바일 앱 작성
{: #get_code}

1. [모바일 대시보드](https://{DomainName}/developer/mobile/dashboard)로 이동하여 사전 정의된 `스타터 킷`으로부터 `앱`을 작성하십시오. 
2. **스타터 킷**을 클릭하고 아래로 스크롤하여 **기본** 스타터 킷을 선택하십시오.
    ![](images/solution6/mobile_dashboard.png)
3. Xcode 프로젝트 및 앱 이름으로도 사용될 앱 이름을 입력하십시오. 
4. `iOS Swift`를 플랫폼으로 선택하고 **작성**을 클릭하십시오.
    ![](images/solution6/create_mobile_project.png)
5. **리소스 추가** > 모바일> **푸시 알림**을 클릭하고 서비스를 프로비저닝할 위치, 리소스 그룹 및 **Lite** 가격 플랜을 선택하십시오. 
6. **작성**을 클릭하여 {{site.data.keyword.mobilepushshort}} 서비스를 프로비저닝하십시오. **앱** 탭에서 새 앱이 작성됩니다. 

​      **참고:** {{site.data.keyword.mobilepushshort}} 서비스가 비어 있는 스타터에 이미 추가되어 있어야 합니다. 

## 코드 다운로드 및 클라이언트 SDK 설정
{: #download_code}

아직 코드를 다운로드하지 않은 경우에는 앱 > `사용자의 모바일 앱`에서 `코드 다운로드`를 클릭하십시오.
다운로드된 코드에는 **{{site.data.keyword.mobilepushshort}}** 클라이언트 SDK가 포함되어 있습니다. 이 클라이언트 SDK는 CocoaPods 및 Carthage에서 사용할 수 있습니다. 이 솔루션에서는 CocoaPods를 사용합니다. 

1. 시스템에 CocoaPods를 설치하려면 `터미널`을 열고 아래 명령을 실행하십시오. 
   ```
   sudo gem install cocoapods
   ```
   {: pre:}
2. 터미널을 사용하여 다운로드한 코드의 압축을 풀고 압축을 푼 폴더로 이동하십시오. 

   ```
   cd <Name of Project>
   ```
   {: pre:}
3. 이 폴더에는 필수 종속 항목을 포함하는 `podfile`이 이미 포함되어 있습니다. 종속 항목(클라이언트 SDK)을 설치하는 아래 명령을 실행하면 필수 종속 항목이 설치됩니다. 

  ```
  pod install
  ```
  {: pre:}

## APNs 인증 정보 얻기 및 {{site.data.keyword.mobilepushshort}} 서비스 인스턴스 구성
{: #obtain_apns_credentials}

   iOS 디바이스 및 애플리케이션의 경우에는 애플리케이션 개발자가 {{site.data.keyword.Bluemix_short}}의 {{site.data.keyword.mobilepushshort}} 서비스 인스턴스(제공자)에서 iOS 디바이스 및 애플리케이션으로 원격 알림을 전송하는 데 APNs(Apple Push Notification service)가 사용됩니다. 디바이스의 대상 애플리케이션으로 메시지가 전송됩니다. 

   사용자는 APNs 인증 정보를 얻고 구성해야 합니다. APNs 인증서는 {{site.data.keyword.mobilepushshort}} 서비스가 안전하게 관리하며 제공자로서 APNs 서버에 연결하는 데 사용됩니다. 

### 앱 ID 등록

   앱 ID(번들 ID)는 특정 애플리케이션을 식별하는 고유 ID입니다. 각 애플리케이션은 앱 ID를 필요로 합니다. {{site.data.keyword.mobilepushshort}} 서비스와 같은 서비스는 앱 ID를 대상으로 구성됩니다.
   자신에게 [Apple Developer](https://developer.apple.com/) 계정이 있는지 확인하십시오. 이는 필수 전제조건입니다. 

   1. [Apple Developer](https://developer.apple.com/) 포털로 이동하여 `Member Center`를 클릭하고 `Certificates, IDs & Profiles`를 선택하십시오. 
   2. `Identifiers` > App IDs 섹션으로 이동하십시오. 
   3. `Registering App IDs` 페이지에서 App ID Description Name 필드에 앱 이름을 제공하십시오. 예: ACME {{site.data.keyword.mobilepushshort}}. App ID 접두부를 위한 문자열을 제공하십시오. 
   4. App ID 접미부에 대해서는 `Explicit App ID`를 선택하고 번들 ID 값을 제공하십시오. 반전된 도메인 이름 스타일의 문자열을 제공하는 것이 좋습니다. 예: com.ACME.push. 
   5. `{{site.data.keyword.mobilepushshort}}` 선택란을 선택하고 `Continue`를 클릭하십시오. 
   6. 설정을 수행하고 `Register` > `Done`을 클릭하십시오.
     이제 앱 ID가 등록되었습니다. 

     ![](images/solution6/push_ios_register_appid.png)

### 개발 및 배포 APNs SSL 인증서 작성
   APNs 인증서를 획득하려면 먼저 인증서 서명 요청(CSR)을 생성하고 이를 인증 기관(CA)인 Apple에 제출해야 합니다. CSR은 사용자의 회사를 식별하는 정보와 사용자가 Apple {{site.data.keyword.mobilepushshort}}에 서명하는 데 사용하는 공개 및 개인 키를 포함합니다. 그런 다음 iOS 개발자 포털에서 SSL 인증서를 생성하십시오. 인증서와 해당 공개 및 개인 키는 Keychain Access에 저장됩니다.
   APNs는 두 가지 모드로 사용할 수 있습니다. 

- 개발 및 테스트를 위한 샌드박스 모드
- App Store(또는 기타 엔터프라이즈 배포 메커니즘)를 통해 애플리케이션을 배포하는 경우의 프로덕션 모드

   개발 환경과 배포 환경에 대해 각각 별도의 인증서를 획득해야 합니다. 인증서는 원격 알림의 수신인인 앱의 앱 ID와 연관됩니다. 프로덕션의 경우 최대 2개의 인증서를 작성할 수 있습니다. {{site.data.keyword.Bluemix_short}}는 인증서를 사용하여 APNs와의 SSL 연결을 설정합니다. 

   1. Apple Developer 웹 사이트로 이동하여 **Member Center**를 클릭하고 **Certificates, IDs & Profiles**를 선택하십시오. 
   2. **Identifiers** 영역에서 **App IDs**를 클릭하십시오. 
   3. 앱 ID 목록에서 자신의 앱 ID를 선택한 후 `Edit`를 선택하십시오. 
   4. **{{site.data.keyword.mobilepushshort}}** 선택란을 선택한 후 **Development SSL certificate** 분할창에서 **Create Certificate**을 클릭하십시오. 

     ![푸시 알림 SSL 인증서](images/solution6/certificate_createssl.png)

   5. **About Creating a Certificate Signing Request (CSR)** 화면이 표시되면 표시된 지시사항에 따라 인증서 서명 요청(CSR) 파일을 작성하십시오. 개발(샌드박스)용 인증서인지, 또는 배포(프로덕션)용 인증서인지 식별할 수 있도록 의미있는 이름을 지정하십시오(예: sandbox-apns-certificate 또는 production-apns-certificate). 
   6. **Continue**를 클릭하고 Upload CSR file 화면에서 **Choose File**을 클릭한 후 방금 작성한 **CertificateSigningRequest.certSigningRequest** 파일을 선택하십시오. **Continue**를 클릭하십시오. 
   7. Download, Install and Backup 분할창에서 Download를 클릭하십시오. **aps_development.cer** 파일이 다운로드됩니다.
        ![인증서 다운로드](images/solution6/push_certificate_download.png)

        ![인증서 생성](images/solution6/generate_certificate.png)
   8. 자신의 Mac에서 **Keychain Access**, **File**, **Import**를 열고 다운로드한 .cer 파일을 선택하여 설치하십시오. 
   9. 새 인증서 및 개인 키를 마우스 오른쪽 단추로 클릭한 후 **Export**를 선택하고 **File Format**을 Personal Information Exchange 형식(`.p12` 형식)으로 변경하십시오.
     ![인증서 및 키 내보내기](images/solution6/keychain_export_key.png)
   10. **Save As** 필드에서 인증서에 의미있는 이름을 제공하십시오. 예를 들면 `sandbox_apns.p12` 또는 **production_apns.p12**와 같습니다. 그 후 Save를 클릭하십시오.
     ![인증서 및 키 내보내기](images/solution6/certificate_p12v2.png)
   11. **비밀번호 입력** 필드에서 내보낸 항목을 보호하기 위한 비밀번호를 입력한 후 OK를 클릭하십시오. 이 비밀번호는 {{site.data.keyword.mobilepushshort}} 서비스 콘솔에서 APNs 설정을 구성하는 데 사용할 수 있습니다.
       ![인증서 및 키 내보내기](images/solution6/export_p12.png)
   12. **Key Access.app**이 **Keychain** 화면에서 키를 내보내도록 프롬프트를 표시합니다. 시스템이 해당 항목을 내보낼 수 있도록 Mac의 관리 비밀번호를 입력한 후 `Always Allow` 옵션을 선택하십시오. 데스크탑에 `.p12` 인증서가 생성됩니다. 

      프로덕션 SSL의 경우에는 **Production SSL certificate** 분할창에서 **Create Certificate**을 클릭하고 5 - 12단계를 반복하십시오.
      {:tip}

### 개발 프로비저닝 프로파일 작성
   프로비저닝 프로파일은 앱 ID와 함께 작동하여 사용자 앱을 설치하고 실행할 수 있는 디바이스 및 사용자 앱에서 액세스할 수 있는 서비스를 판별합니다. 사용자는 각 앱 ID에 대해 두 개(개발용 및 배포용)의 프로비저닝 프로파일을 작성합니다. Xcode는 개발 프로비저닝 프로파일을 사용하여 애플리케이션을 빌드할 수 있도록 허용된 개발자와 애플리케이션을 테스트할 수 있도록 허용된 디바이스를 판별합니다. 

   앱 ID를 등록하고, {{site.data.keyword.mobilepushshort}} 서비스를 사용할 수 있도록 설정하였으며 개발 및 프로덕션 APNs SSL 인증서를 사용하도록 구성했는지 확인하십시오. 

   개발 프로비저닝 프로파일을 다음과 같이 작성하십시오. 

   1. [Apple Developer](https://developer.apple.com/) 포털로 이동하여 `Member Center`를 클릭하고 `Certificates, IDs & Profiles`를 선택하십시오. 
   2. [Mac 개발자 라이브러리](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site)로 이동하여 `Creating Development Provisioning Profiles` 섹션으로 이동한 후 개발 프로파일 작성에 대한 지시사항을 따르십시오.
     **참고:** 개발 프로비저닝 프로파일을 구성할 때는 다음 옵션을 선택하십시오. 

     - **iOS App Development**
     - **For iOS and watchOS apps**

### 상점 배포 프로비저닝 프로파일 작성
   배포용 앱을 App Store에 제출하려면 상점 프로비저닝 프로파일을 사용하십시오. 

   1. [Apple Developer](https://developer.apple.com/)로 이동하여 `Member Center`를 클릭하고 `Certificates, IDs & Profiles`를 선택하십시오. 
   2. 다운로드한 프로비저닝 프로파일을 두 번 클릭하여 Xcode에 설치하십시오.
     인증 정보를 얻은 후 다음 단계는 [서비스 인스턴스를 구성](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2)하는 것입니다. 

### 서비스 인스턴스 구성

   알림을 전송하기 위해 {{site.data.keyword.mobilepushshort}} 서비스를 사용하려면 위 단계에서 작성한 .p12 인증서를 업로드하십시오. 이 인증서는 애플리케이션을 빌드하고 공개하는 데 필요한 개인 키 및 SSL 인증서를 포함합니다. 

   **참고:** `.cer` 파일이 Keychain Access에 저장되고 나면 이를 자신의 컴퓨터로 내보내 `.p12` 인증서를 작성하십시오. 

1. 서비스 섹션의 {{site.data.keyword.mobilepushshort}}을 클릭하거나 {{site.data.keyword.mobilepushshort}} 서비스 옆에 세로로 나열된 세 개의 점을 클릭하고 `대시보드 열기`를 선택하십시오. 
2. {{site.data.keyword.mobilepushshort}} 대시보드의 `관리`에서 `구성` 옵션을 볼 수 있습니다. 

`푸시 알림 서비스` 콘솔에서 APNs를 설정하려면 다음 단계를 완료하십시오. 

1. `모바일 옵션`을 선택하여 APNs 푸시 인증 정보 양식의 정보를 업데이트하십시오. 
2. `샌드박스/개발 APNs 서버` 또는 `프로덕션 APNs 서버` 중 해당되는 것을 선택한 후 작성한 `.p12` 인증서를 업로드하십시오. 
3. 비밀번호 필드에 .p12 인증서와 연관된 비밀번호를 입력한 후 저장을 클릭하십시오. 

![](images/solution6/Mobile_push_configure.png)

## {{site.data.keyword.mobilepushshort}}의 구성, 전송 및 모니터링
{: #configure_push}

1. 푸시 초기화 코드(`func application`에 있음) 및 알림 등록 코드는 `AppDelegate.swift`에 있습니다. 고유 USER_ID를 제공하십시오(선택사항). 
2. iPhone Simulator로는 알림을 전송할 수 없으므로 앱을 실제 디바이스에서 실행하십시오. 
3. {{site.data.keyword.Bluemix_short}} 모바일 대시보드의 `모바일 서비스` > **기존 서비스**에서 {{site.data.keyword.mobilepushshort}} 서비스를 열고 기본 {{site.data.keyword.mobilepushshort}}을 전송하려면 다음 단계를 완료하십시오. 
  * `메시지`를 선택하고 전송 옵션을 선택하여 메시지를 작성하십시오. 지원되는 옵션은 태그별 디바이스, 디바이스 ID별 디바이스, 사용자 ID별 디바이스, Android 디바이스, iOS 디바이스, 웹 알림, 모든 디바이스 및 기타 브라우저입니다. 

       **참고:** 모든 디바이스 옵션을 선택하면 {{site.data.keyword.mobilepushshort}}을 구독하는 모든 디바이스가 알림을 수신합니다.
  * `메시지` 필드에서 메시지를 작성하십시오. 필요에 따라 선택적 설정을 구성하십시오. 
  * `전송`을 클릭하고 실제 디바이스가 알림을 수신했는지 확인하십시오.
    ![](images/solution6/send_notifications.png)

4. 사용자의 iPhone에 알림이 표시됩니다. 

   ![](images/solution6/iphone_notification.png)
5. {{site.data.keyword.mobilepushshort}} 서비스의 `모니터링`으로 이동하여 전송한 알림을 모니터할 수 있습니다. 

IBM {{site.data.keyword.mobilepushshort}} 서비스는 이제 사용자 데이터로부터 그래프를 생성하여 푸시 성능을 모니터할 수 있도록 기능이 확장되었습니다. 사용자는 이 유틸리티를 사용하여 전송된 모든 {{site.data.keyword.mobilepushshort}}을 나열하거나, 모든 등록된 디바이스를 나열하거나, 일별, 주별 또는 월별로 정보를 분석할 수 있습니다.
![](images/solution6/monitoring_messages.png)

## 관련 컨텐츠
{: #related_content}

- [태그 기반 알림](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}}의 보안](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)
