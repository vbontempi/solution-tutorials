---
copyright:
  years: 2018, 2019
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

# 여러 지역에 서버리스 앱 배치
{: #multi-region-serverless}

이 튜토리얼에서는 여러 지역에 서버리스 앱을 배치하기 위해 IBM Cloud Internet Services 및 {{site.data.keyword.openwhisk_short}}를 구성하는 방법을 보여줍니다. 

서버리스 컴퓨팅 플랫폼은 개발자에게 서버 없이 신속하게 API를 빌드하는 방법을 제공합니다. {{site.data.keyword.openwhisk}}는 액션에 대한 REST API 자동 생성, 액션의 HTTP 엔드포인트 변환, 안전한 API 인증을 가능하게 하는 기능을 지원합니다. 이 기능은 API를 외부 이용자에게 노출하는 데 도움을 주며 마이크로서비스 빌드에도 도움을 줍니다. 

{{site.data.keyword.openwhisk_short}}는 여러 {{site.data.keyword.cloud_notm}} 위치에서 사용 가능합니다. 복원성을 향상시키고 네트워크 대기 시간을 줄이기 위해, 애플리케이션은 자신의 백엔드를 여러 위치에 배치할 수 있습니다. 그 후에는 개발자가 IBM Cloud Internet Services(CIS)를 사용하여 가장 가까운 정상 백엔드에 트래픽을 분배하는 하나의 시작점을 노출할 수 있습니다. 

## 목표
{: #objectives}

* {{site.data.keyword.openwhisk_short}} 액션을 배치합니다. 
* 사용자 정의 도메인을 포함하는 {{site.data.keyword.APIM}}를 통해 액션을 노출합니다. 
* Cloud Internet Services를 사용하여 여러 위치에 트래픽을 분배합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

이 튜토리얼에서는 {{site.data.keyword.openwhisk_short}}로 구현된 백엔드가 있는 공용 웹 애플리케이션을 가정합니다. 이 애플리케이션은 네트워크 대기 시간을 줄이고 가동 중단을 방지하기 위해 여러 위치에 배치됩니다. 이 튜토리얼에서는 두 가지 위치가 구성됩니다. 

<p style="text-align: center;">

  ![아키텍처](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. 사용자가 애플리케이션에 액세스합니다. 요청이 Internet Services를 통해 전달됩니다. 
2. Internet Services가 사용자를 가장 가까운 정상 API 백엔드로 경로 재지정합니다. 
3. {{site.data.keyword.cloudcerts_short}}가 해당 SSL 인증서를 사용하여 API를 제공합니다. 트래픽이 엔드 투 엔드로 암호화됩니다. 
4. API가 {{site.data.keyword.openwhisk_short}}를 사용하여 구현됩니다. 

## 시작하기 전에
{: #prereqs}

1. Cloud Internet Services의 사용자가 사용자 정의 도메인을 소유해야 합니다. 이를 소유해야 Cloud Internet Services 이름 서버를 가리키도록 이 도메인의 DNS를 구성할 수 있습니다. 소유하는 도메인이 없는 경우에는 [godaddy.com](http://godaddy.com)와 같은 등록 담당자로부터 도메인을 구입할 수 있습니다. 
1. [이러한 단계](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)에 따라 모든 필요한 명령행 인터페이스(CLI)를 설치하십시오. 

## 사용자 정의 도메인 구성

첫 번째 단계는 IBM Cloud Internet Services(CIS)의 인스턴스를 작성하고 사용자 정의 도메인이 CIS 이름 서버를 가리키도록 하는 것입니다. 

1. {{site.data.keyword.Bluemix_notm}} 카탈로그의 [Internet Services](https://{DomainName}/catalog/services/internet-services)로 이동하십시오. 
1. 서비스 이름을 설정하고 **작성**을 클릭하여 서비스의 인스턴스를 작성하십시오. 이 튜토리얼에서는 모든 가격 플랜을 사용할 수 있습니다. 
1. 서비스 인스턴스가 프로비저닝되면 **시작하기**를 클릭하여 도메인 이름을 설정하고 **도메인 추가**를 클릭하십시오. 
1. **다음 단계**를 클릭하십시오. 이름 서버가 지정되면 나열된 이름 서버를 사용하도록 등록 담당자 또는 도메인 이름 제공자를 구성하십시오. 
1. 등록 담당자 또는 DNS 제공자를 구성한 후 이것이 적용되기까지는 최대 24시간이 소요될 수 있습니다. 

   개요 페이지의 도메인 상태가 *보류 중*에서 *활성*으로 변경되면 `dig <your_domain_name> ns` 명령을 사용하여 새 이름 서버가 적용되었는지 확인할 수 있습니다.
   {:tip}

### 사용자 정의 도메인에 대한 인증서 얻기

사용자 정의 도메인을 통해 {{site.data.keyword.openwhisk_short}} 액션을 노출하려면 안전한 HTTPS 연결이 필요합니다. 사용자는 서버리스 백엔드에 사용할 도메인 및 하위 도메인에 대한 SSL 인증서를 얻어야 합니다. 도메인이 *mydomain.com*인 경우에는 *api.mydomain.com*에서 액션을 호스팅할 수 있습니다. 이 경우에는 *api.mydomain.com*에 대해 인증서가 발행되어야 합니다. 

[Let's Encrypt](https://letsencrypt.org/)에서 무료 SSL 인증서를 얻을 수 있습니다. 이 프로세스 중에, 자신이 도메인 소유자임을 증명하기 위해 Cloud Internet Services의 DNS 인터페이스에서 유형이 TXT인 DNS 레코드를 구성해야 할 수 있습니다.
{:tip}

도메인에 대한 SSL 인증서 및 개인 키를 얻은 후에는 이를 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 형식으로 변환해야 합니다. 

1. 인증서를 PEM 형식으로 변환하려면 다음 명령을 실행하십시오. 
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. 개인 키를 PEM 형식으로 변환하려면 다음 명령을 실행하십시오. 
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### 인증서를 중앙 저장소로 가져오기

1. 지원되는 위치에 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 인스턴스를 작성하십시오. 
1. 서비스 대시보드에서 **인증서 가져오기**를 사용하십시오. 
   * **이름**을 *api.mydomain.com*과 같은 사용자 정의 하위 도메인 및 도메인으로 설정하십시오. 
   * PEM 형식의 **인증서 파일**을 찾아보십시오. 
   * PEM 형식의 **개인 키 파일**을 찾아보십시오. 
   * **가져오십시오**. 

## 여러 위치에 액션 배치

이 절에서는 액션을 작성하고, 이를 API로서 노출하고, {{site.data.keyword.cloudcerts_short}}에 저장된 SSL 인증서를 사용하여 사용자 정의 도메인을 API에 맵핑합니다. 

<p style="text-align: center;">

  ![API 아키텍처](images/solution44-multi-region-serverless/api-architecture.png)
</p>

액션 **doWork**는 사용자의 API 오퍼레이션 중 하나를 구현합니다. 액션 **healthz**는 나중에 API의 정상 여부 확인에 사용됩니다. 이는 *정상*만 리턴하고 아무 오퍼레이션도 수행하지 않거나 데이터베이스, 또는 API에서 필요로 하는 기타 중요 서비스에 대한 ping 실행과 같이 더 복잡한 확인을 수행할 수도 있습니다. 

다음 세 절은 애플리케이션 백엔드를 호스팅할 모든 위치에서 반복해야 합니다. 이 튜토리얼에서는 *댈러스(us-south)* 및 *런던(eu-gb)*을 대상으로 선택할 수 있습니다. 

### 액션 정의

1. [{{site.data.keyword.openwhisk_short}} / 액션](https://{DomainName}/openwhisk/actions)으로 이동하십시오. 
1. 대상 위치로 전환하고 액션을 배치할 조직 및 영역을 선택하십시오. 
1. 액션을 작성하십시오. 
   1. **이름**을 **doWork**로 설정하십시오. 
   1. **엔클로징 패키지**를 **기본**으로 설정하십시오. 
   1. **런타임**을 최신 **Node.js** 버전으로 설정하십시오. 
   1. **작성**하십시오. 
1. 액션 코드를 변경하십시오. 
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. **저장**하십시오. 
1. API의 상태 확인에 사용할 다른 액션을 작성하십시오. 
   1. **이름**을 **healthz**로 설정하십시오. 
   1. **엔클로징 패키지**를 **기본**으로 설정하십시오. 
   1. **런타임**을 최신 **Node.js** 버전으로 설정하십시오. 
   1. **작성**하십시오. 
1. 액션 코드를 변경하십시오. 
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. **저장**하십시오. 

### 관리 API를 사용하여 액션 노출

다음 단계는 액션 노출에 필요한 관리 API를 작성하는 작업을 포함합니다. 

1. [{{site.data.keyword.openwhisk_short}} / API](https://{DomainName}/openwhisk/apimanagement)로 이동하십시오. 
1. 새 관리 {{site.data.keyword.openwhisk_short}} API를 작성하십시오. 
   1. **API 이름**을 **앱 API**로 설정하십시오. 
   1. **기본 경로**를 **/api**로 설정하십시오. 
1. 오퍼레이션을 작성하십시오. 
   1. **경로**를 **/do**로 설정하십시오. 
   1. **verb**를 **GET**으로 설정하십시오. 
   1. **패키지**를 **기본**으로 설정하십시오. 
   1. **액션**을 **doWork**로 설정하십시오. 
   1. **작성**하십시오. 
1. 다른 오퍼레이션을 작성하십시오. 
   1. **경로**를 **/healthz**로 설정하십시오. 
   1. **verb**를 **GET**으로 설정하십시오. 
   1. **패키지**를 **기본**으로 설정하십시오. 
   1. **액션**을 **healthz**로 설정하십시오. 
   1. **작성**하십시오. 
1. API를 **저장**하십시오. 

### 관리 API의 사용자 정의 도메인 구성

관리 API를 작성하면 `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`과 같은 기본 엔드포인트가 제공됩니다. 이 절에서는 사용자 정의 하위 도메인에서 수신되는 요청을 처리할 수 있도록 이 엔드포인트를 구성하며, 이 도메인은 나중에 IBM Cloud Internet Services에서 구성됩니다. 

1. [API / 사용자 정의 도메인](https://{DomainName}/apis/domains)으로 이동하십시오. 
1. **지역** 선택기에서 대상 지역을 선택하십시오. 
1. 조치 및 관리 API를 작성한 조직 및 영역과 연결된 사용자 정의 도메인을 찾으십시오. 조치 메뉴에서 **설정 변경**을 클릭하십시오. 
1. **기본 도메인 / 별명** 값을 기록하십시오. 
1. **사용자 정의 도메인 적용**을 선택하십시오. 
   1. **도메인 이름**을 CIS 글로벌 로드 밸런서와 함께 사용할 도메인(예: *api.mydomain.com*)으로 설정하십시오. 
   1. 인증서가 있는 {{site.data.keyword.cloudcerts_short}} 인스턴스를 선택하십시오. 
   1. 도메인에 대한 인증서를 선택하십시오. 
1. 자신의 **Cloud Internet Services** 인스턴스의 대시보드로 이동하여 **신뢰성 / DNS**에서 새 **DNS TXT 레코드**를 작성하십시오. 
   1. **이름**을 사용자 정의 하위 도메인(예: **api**)으로 설정하십시오. 
   1. **컨텐츠**를 **기본 도메인 / 별명**으로 설정하십시오. 
   1. 레코드를 저장하십시오. 
1. 사용자 정의 도메인 설정을 저장하십시오. 대화 상자가 해당 DNS TXT 레코드의 존재 여부를 확인합니다. 

   해당 TXT 레코드를 찾을 수 없는 경우에는 이것이 전파될 때까지 기다린 후 설정 저장을 재시도해야 합니다. 설정이 적용된 후에는 DNS TXT 레코드를 제거할 수 있습니다.
   {: tip}

이전 절을 반복하여 추가 위치를 구성하십시오. 

## 위치 간에 트래픽 분배

**이 단계에 도달하면, 여러 위치에 액션을 설정**했지만 이들에 도달할 시작점은 없는 상태입니다. 이 절에서는 위치 간에 트래픽을 분배하는 글로벌 로드 밸런서(GLB)를 구성합니다. 

<p style="text-align: center;">

  ![글로벌 로드 밸런서의 아키텍처](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### 상태 확인 작성

Internet Services는 백엔드의 상태를 확인하기 위해 정기적으로 이 엔드포인트를 호출할 수 있습니다. 

1. IBM Cloud Internet Services 인스턴스의 대시보드로 이동하십시오. 
1. **신뢰성 / 글로벌 로드 밸런서**에서 상태 확인을 작성하십시오. 
   1. **모니터 유형**을 **HTTPS**로 설정하십시오. 
   1. **경로**를 **/api/healthz**로 설정하십시오. 
   1. **리소스를 프로비저닝**하십시오. 

### 오리진 풀 작성

위치당 하나의 풀을 작성하면 나중에 글로벌 로드 밸런서에서 지리적 라우트를 구성하여 사용자를 가장 가까운 위치로 경로 재지정할 수 있습니다. 또 다른 옵션은 모든 위치를 포함하는 하나의 풀을 작성하고 로드 밸런서가 풀에 있는 오리진을 순환하도록 하는 것입니다. 

모든 위치에 대해 다음 작업을 수행하십시오. 
1. 오리진 풀을 작성하십시오. 
1. **이름**을 **app-&lt;위치&gt;**(예: _app-Dallas_)로 설정하십시오. 
1. 이전에 작성한 상태 확인을 선택하십시오. 
1. **상태 확인 지역**을 {{site.data.keyword.openwhisk_short}}가 배치된 위치와 가까운 지역으로 설정하십시오. 
1. **오리진 이름**을 **app-&lt;위치&gt;**로 설정하십시오. 
1. **오리진 주소**를 관리 API의 기본 도메인 / 별명(예: _5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_)으로 설정하십시오. 
1. **리소스를 프로비저닝**하십시오. 

### 글로벌 로드 밸런서 작성

1. 로드 밸런서를 작성하십시오. 
1. **밸런서 호스트 이름**을 **api.mydomain.com**으로 설정하십시오. 
1. 지역 오리진 풀을 추가하십시오. 
1. **리소스를 프로비저닝**하십시오. 

잠시 후 `https://api.mydomain.com/api/do?name=John&place=Earth`로 이동하십시오. 이렇게 하면 첫 번째 정상 풀에서 실행 중인 기능이 응답으로 제공됩니다. 

### 장애 조치 테스트

장애 조치를 테스트하려면, GLB가 다음 정상 풀로 트래픽을 경로 재지정하도록 하기 위해 풀 상태 확인이 실패해야 합니다. 장애를 시뮬레이션하기 위해 상태 검사 기능이 실패하도록 수정할 수 있습니다. 

1. [{{site.data.keyword.openwhisk_short}} / 액션](https://{DomainName}/openwhisk/actions)으로 이동하십시오. 
1. GLB에 구성된 첫 번째 위치를 선택하십시오. 
1. `healthz` 기능을 편집하여 해당 구현을 `throw new Error()`로 변경하십시오. 
1. 저장하십시오. 
1. 이 오리진 풀에 대해 상태 확인이 실행되기를 기다리십시오. 
1. `https://api.mydomain.com/api/do?name=John&place=Earth`로 다시 이동하십시오. 이번에는 다른 정상 오리진으로 경로 재지정됩니다. 
1. 코드 변경사항을 되돌려 해당 위치를 정상 오리진으로 되돌리십시오. 

## 리소스 제거
{: #removeresources}

### CIS 리소스 제거

1. GLB를 제거하십시오. 
1. 오리진 풀을 제거하십시오. 
1. 상태 확인을 제거하십시오. 

### 액션 제거

1. [API](https://{DomainName}/openwhisk/apimanagement)를 제거하십시오. 
1. [액션](https://{DomainName}/openwhisk/actions)을 제거하십시오. 

## 관련 컨텐츠
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Cloud Internet Services를 사용하여 강한 복원성과 보안을 갖춘 다중 지역 Kubernetes 클러스터](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
