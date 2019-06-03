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

# 여러 지역에서 웹 애플리케이션 보안
{: #multi-region-webapp}

이 튜토리얼에서는 [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) 파이프라인을 사용하여 여러 지역에서 Cloud Foundry 애플리케이션을 작성하고 보안을 설정하고 배치하고 로드 밴런싱을 수행하는 것에 대해 설명합니다. 

앱 또는 앱의 일부가 가동 중단되는 것이 사실입니다. 이는 코드, 앱이 사용하는 리소스에 영향을 주는 계획된 유지보수, 구역을 작동 중단시키는 하드웨어 장애, 앱이 호스팅되는 데이터 센터에서의 문제일 수 있습니다. 이러한 문제가 발생하므로 준비를 해야 합니다. {{site.data.keyword.Bluemix_notm}}를 사용하면 애플리케이션을 [여러 위치](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg)에 배치하여 애플리케이션 복원성을 높일 수 있습니다. 또한 이제 애플리케이션이 여러 위치에서 실행되므로 사용자 트래픽의 경로를 가장 가까운 위치로 재지정하여 대기 시간을 줄일 수도 있습니다. 

## 목표

* {{site.data.keyword.contdelivery_short}}를 사용하여 Cloud Foundry 애플리케이션을 여러 위치에 배치합니다. 
* 사용자 정의 도메인을 애플리케이션에 맵핑합니다. 
* 다중 위치 애플리케이션에 대한 글로벌 로드 밸런싱을 구성합니다. 
* SSL 인증서를 애플리케이션에 바인드합니다. 
* 애플리케이션 성능을 모니터합니다. 

## 사용되는 서비스

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry 앱
* DevOps용 [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처

이 튜토리얼에는 두 개의 애플리케이션 사본이 두 개의 다른 위치에 배치되고 두 개의 사본이 라운드 로빈 방식으로 고객 요청을 제공하는 능동/능동 시나리오가 포함되어 있습니다. 하나의 사본이 실패하는 경우 DNS 구성은 자동으로 정상 위치를 가리킵니다. 

<p style="text-align: center;">

   ![아키텍처](./images/solution1/Architecture.png)
</p>

## Node.js 애플리케이션 작성
{: #create}

Cloud Foundry 환경에서 실행되는 Node.js 스타터 애플리케이션을 작성하는 것에서부터 시작하십시오. 

1. {{site.data.keyword.Bluemix_notm}} 콘솔에서 **[카탈로그](https://{DomainName}/catalog/)**를 클릭하십시오. 
2. **플랫폼** 카테고리 아래에서 **Cloud Foundry 앱**을 클릭한 후 **[{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)**을 선택하십시오.
     ![](images/solution1/SDKforNodejs.png)
3. 애플리케이션에 대해 **고유 이름**(호스트 이름이기도 함)을 입력하십시오(예: myusername-nodeapp). 그리고 **작성**을 클릭하십시오. 
4.  애플리케이션이 시작되면 **개요** 페이지에서 **방문 URL** 링크를 클릭하여 새 탭에서 애플리케이션을 라이브로 확인하십시오. 

![HelloWorld](images/solution1/HelloWorld.png)

출발이 좋습니다. 자체 Node.js 스타터 애플리케이션이 {{site.data.keyword.Bluemix_notm}}에서 실행 중입니다. 

다음으로 애플리케이션의 소스 코드를 저장소에 푸시한 후 변경사항을 자동으로 배치해 봅니다. 

## 소스 제어 및 {{site.data.keyword.contdelivery_short}} 설정
{: #devops}

이 단계에서는 코드를 저장할 git 소스 제어 저장소를 설정한 후 파이프라인을 작성하여 코드 변경사항을 자동으로 배치합니다. 

1. 방금 작성한 애플리케이션의 왼쪽 분할창에서 **개요**를 선택한 후 스크롤하여 **{{site.data.keyword.contdelivery_short}}**를 찾으십시오. **사용**을 클릭하십시오. 

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. 기본 옵션을 유지하고 **작성**을 클릭하십시오. 이제 기본 **도구 체인**이 작성되었습니다. 

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. **코드** 아래에서 **Git** 타일을 선택하십시오. 그러면 git 저장소 페이지로 이동합니다. 
4. 아직 SSH 키를 설정하지 않은 경우에는 맨 위에 알림 표시줄이 지시사항과 함께 표시됩니다. 새 탭에서 **SSH 키 추가** 링크를 열어 단계를 수행하십시오. SSH 대신 HTTPS를 사용하려면 **개인 액세스 토큰 작성**을 클릭하여 단계를 수행하십시오. 나중에 참조할 수 있도록 키 또는 토큰을 저장하십시오.
5. SSH 또는 HTTPS를 선택한 후 git URL을 복사하십시오. 소스를 로컬 시스템에 복제하십시오. 
   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   **참고:** 사용자 이름 입력 프롬프트가 표시되면 git 사용자 이름을 제공하십시오. 비밀번호는 기존 **SSH 키** 또는 **개인용 액세스 토큰**이나 이전 단계에서 작성한 것을 사용하십시오.
6. 선택한 IDE에서 복제된 저장소를 열고 `public/index.html`로 이동하십시오. 이제 코드를 업데이트합니다. "Hello World"를 다른 것으로 바꿔 보십시오. 
7. `npm install`, `npm build`, `npm start` 명령을 순서대로 실행하여 로컬로 애플리케이션을 실행한 후
  브라우저에서 콘솔에 표시된 대로 `localhost:<port_number>`**<port_number>**를
  방문하십시오. 
8. 추가, 커미트 및 푸시라는 세 가지 단순한 단계를 사용하여 변경사항을 저장소에 푸시하십시오. 
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
9. 이전에 작성한 도구 체인으로 이동하여 **Delivery Pipeline** 타일을 클릭하십시오. 
10. **빌드** 및 **배치** 단계가 표시되는지 확인하십시오.
  ![HelloWorld](images/solution1/DevOps_Pipeline.png)
11. **배치** 단계가 완료될 때까지 기다리십시오. 
12. 마지막 실행 결과의 애플리케이션 **url**을 클릭하여 변경사항을 실시간으로 보십시오. 

애플리케이션에 대한 추가 변경사항을 계속 작성하고 git 저장소에 대해 변경사항을 주기적으로 커미트하십시오. 애플리케이션이 업데이트되지 않으면 파이프라인의 배치 및 빌드 단계의 로그를 확인하십시오. 

## 다른 위치에 배치
{: #deploy_another_region}

다음으로 동일한 애플리케이션을 다른 {{site.data.keyword.Bluemix_notm}} 위치에 배치합니다. 동일한 도구 체인을 사용할 수 있지만 또 다른 배치 단계를 추가하여 또 다른 위치에 대한 애플리케이션 배치를 처리합니다. 

1. 애플리케이션 **개요**로 이동한 후 스크롤하여 **도구 체인 보기**를 찾으십시오. 
2. 전달에서 **딜리버리 파이프라인**을 선택하십시오. 
3. **배치** 단계에서 **기어 아이콘**을 클릭한 후 **복제 단계**를 선택하십시오.
   ![HelloWorld](images/solution1/CloneStage.png)
4. 단계의 이름을 "영국에 배치"로 바꾼 후 **작업**을 선택하십시오. 
5. **IBM Cloud 지역**을 **런던 - https://api.eu-gb.bluemix.net**으로 변경하십시오. **영역**이 없으면 영역을 작성하십시오. 
6. **배치 스크립트**를 `cf push "${CF_APP}" -d eu-gb.mybluemix.net`으로 변경하십시오. 

   ![HelloWorld](images/solution1/DeployToUK.png)
7. **저장**을 클릭한 후 **재생 단추**를 클릭하여 새 단계를 실행하십시오. 

## IBM Cloud Internet Services에 사용자 정의 도메인 등록

{: #domain_cis}

IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)는 DNS(Domain Name System), GLB(Global Load Balancing), WAF(Web Application Firewall) 및 웹 애플리케이션을 위해 DDoS(Distributed Denial of Service)에 대한 보호를 구성하고 관리하는 균등 플랫폼입니다. 이는 워크플로우를 향상시키는 세 가지 기본 기능(보안, 신뢰성 및 성능)을 사용하여 IBM Cloud에서 비즈니스를 운영하는 고객을 위해 빠르고 신뢰할 수 있고 안전한 고성능 인터넷 서비스를 제공합니다.   

실제 애플리케이션을 배치할 때는 IBM이 제공한 mybluemix.net 도메인 대신 자체 도메인을 사용하려고 합니다. 이 단계에서는 사용자 정의 도메인을 확보한 후 IBM Cloud Internet Services에서 제공하는 DNS 서버를 사용할 수 있습니다. 

1. 등록 담당자(예: [http://godaddy.com](http://godaddy.com))로부터 도메인을 구입하십시오. 
2. {{site.data.keyword.Bluemix_notm}} 카탈로그의 [Internet Services](https://{DomainName}/catalog/services/internet-services)로 이동하십시오. 
2. 서비스 이름을 입력한 후 **작성**을 클릭하여 서비스의 인스턴스를 작성하십시오. 
3. 서비스 인스턴스가 프로비저닝되면 도메인 이름을 설정한 후 **도메인 추가**를 클릭하십시오. 
4. 이름 서버가 지정되면 나열된 이름 서버를 사용하도록 등록 담당자 또는 도메인 이름 제공자를 구성하십시오. 
5. 등록 담당자 또는 DNS 제공자를 구성한 후 변경사항이 적용되는 데 최대 24시간이 걸릴 수 있습니다.
  개요 페이지에서 도메인의 상태가 *보류 중*에서 *활성*으로 변경되면 `dig <your_domain_name> ns` 명령을 사용하여 IBM Cloud 이름 서버가 적용되었는지 확인할 수 있습니다.
  {:tip}

## 애플리케이션에 글로벌 로드 밸런싱 추가

{: #add_glb}

이 절에서는 IBM Cloud Internet Services에서 GLB(Global Load Balancer)를 사용하여 여러 위치의 트래픽을 관리합니다. GLB는 여러 원본에 트래픽이 분배되도록 허용하는 원본 풀을 활용합니다. 

### GLB를 작성하기 전에 GLB에 대한 상태 검사를 작성하십시오. 

1. Cloud Internet Services 애플리케이션에서 **신뢰성** > **글로벌 로드 밸런서**로 이동한 후 페이지의 맨 아래에서 **상태 검사 작성**을 클릭하십시오. 
2. 모니터할 경로(예: `/`)를 입력한 후 유형(HTTP 또는 HTTPS)을 선택하십시오. 일반적으로 전용 상태 엔드포인트를 작성할 수 있습니다. **1개 인스턴스 프로비저닝**을 클릭하십시오.
   ![상태 검사](images/solution1/health_check.png)

### 그런 다음 두 개의 원본을 가진 원본 풀을 작성하십시오. 

1. **풀 작성**을 클릭하십시오. 
2. 풀의 이름을 입력하고 방금 작성한 상태 검사를 선택한 후 node.js 애플리케이션의 위치와 가까운 지역을 선택하십시오. 
3. 댈러스 `<your_app>.mybluemix.net`에서 애플리케이션의 호스트 이름 및 첫 번째 원본의 이름을 입력하십시오. 
4. 마찬가지로 런던 `<your_app>.eu-gb.mybluemix.net`에서 애플리케이션을 가리키는 원본 주소를 가진 다른 원본을 추가하십시오. 
5. **1개 인스턴스 프로비저닝**을 클릭하십시오.
   ![원본 풀](images/solution1/origin_pool.png)

### GLB(Global Load Balancer)를 작성하십시오.  

1. **로드 밸런서 작성**을 클릭하십시오. 
2. 글로벌 로드 밸런서의 이름을 입력하십시오. 이 이름은 위치에 관계없이 유니버셜 애플리케이션 URL(`http://<glb_name>.<your_domain_name>`)의 일부도 됩니다. 
3. **풀 추가**를 클릭한 후 방금 작성한 원본 풀을 선택하십시오. 
4. **1개 인스턴스 프로비저닝**을 클릭하십시오.
   ![글로벌 로드 밸런서](images/solution1/load_balancer.png)

이 단계에서는 GLB가 구성되어 있지만 Cloud Foundry 애플리케이션은 아직 구성된 GLB 도메인 이름의 요청에 응답할 준비가 되어 있지 않습니다. 구성을 완료하기 위해 사용자 정의 도메인을 사용하는 라우트로 애플리케이션을 업데이트합니다. 

## 애플리케이션에 대한 사용자 정의 도메인 및 라우트 정의

{: #add_domain}

이 단계에서는 애플리케이션이 실행 중인 {{site.data.keyword.Bluemix_notm}} 위치에 대한 보안 엔드포인트에 사용자 정의 도메인 이름을 맵핑합니다. 

1. 메뉴 표시줄에서 **관리**를 클릭한 후 **계정**: [계정](https://{DomainName}/account)을 클릭하십시오. 
2. 계정 페이지에서 애플리케이션 **Cloud Foundry 조직**으로 이동한 후 조치 열에서 **도메인**을 선택하십시오. 
3. **도메인 추가**를 클릭한 후 등록 담당자로부터 얻은 사용자 정의 도메인 이름을 입력하십시오. 
4. 올바른 위치를 선택한 후 **저장**을 클릭하십시오. 
5. 마찬가지로 사용자 정의 도메인 이름을 런던에 추가하십시오. 
6. {{site.data.keyword.Bluemix_notm}} [리소스 목록](https://{DomainName}/resources)으로 돌아가서 **Cloud Foundry 앱**으로 이동한 후 댈러스에서 애플리케이션을 클릭하고 **라우트** > **라우트 편집**을 클릭한 후 **라우트 추가**를 클릭하십시오.
   ![라우트 추가](images/solution1/ApplicationRoutes.png)
7. **호스트 입력(선택사항)** 필드에 이전에 구성한 GLB 호스트 이름을 입력한 후 방금 추가한 사용자 정의 도메인을 선택하십시오. **저장**을 클릭하십시오. 
8. 마찬가지로 런던에서 애플리케이션에 대한 도메인 및 라우트를 구성하십시오. 

이 지점에서 URL `<glb_name>.<your_domain_name>`을 사용하여 애플리케이션을 방문할 수 있으며 글로벌 로드 밸런서는 다중 위치 애플리케이션에 대한 트래픽을 자동으로 분배합니다. 댈러스에서 애플리케이션을 중지하고 런던에서 애플리케이션을 활성 상태로 유지하고 글로벌 로드 밸런서를 통해 애플리케이션에 액세스하여 이를 확인할 수 있습니다. 

지금은 이 작업이 작동하지만 이전 단계에서 지속적 딜리버리를 구성했기 때문에 다른 빌드가 트리거될 때 구성이 겹쳐써질 수 있습니다. 이 변경사항을 지속시키려면 도구 체인으로 되돌아가서 *manifest.yml* 파일을 수정하십시오. 

1. {{site.data.keyword.Bluemix_notm}} [리소스 목록](https://{DomainName}/resources)에서 **Cloud Foundry 앱**으로 이동한 후 댈러스에서 애플리케이션을 클릭하고 애플리케이션 **개요**로 이동한 후 스크롤하여 **도구 체인 보기**를 찾으십시오. 
2. 코드 아래에서 Git 타일을 선택하십시오. 
3. *manifest.yml*을 선택하십시오. 
4. **편집**을 클릭한 후 사용자 정의 라우트를 추가하십시오. 원래 도메인 및 호스트 구성을 `Routes`로만 바꾸십시오. 

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <glb_name>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. 변경사항을 커미트하고 두 위치 모두에 대한 빌드가 성공했는지 확인하십시오.   

## 대안: IBM Cloud 시스템 도메인에 사용자 정의 도메인 맵핑

다중 위치 애플리케이션 앞에서 글로벌 로드 밸런서를 활용하길 원하지 않지만 애플리케이션이 실행 중인 {{site.data.keyword.Bluemix_notm}} 위치에 대한 보안 엔드포인트에 사용자 정의 도메인 이름을 맵핑해야 할 수 있습니다. 

Cloud Intenet Services 애플리케이션을 사용하여 다음과 같은 단계를 수행하여 애플리케이션에 대한 `CNAME` 레코드를 설정하십시오. 

1. Cloud Internet Services 애플리케이션에서 **신뢰성** > **DNS**로 이동하십시오. 
2. **유형** 드롭 다운 목록에서 **CNAME**을 선택하고 이름 필드에 애플리케이션의 별명을 입력한 후 도메인 이름 필드에 애플리케이션 URL을 입력하십시오. 댈러스에 있는 애플리케이션 `<your_app>.mybluemix.net`은 CNAME `<your_app>`에 맵핑될 수 있습니다. 
3. **레코드 추가**를 클릭하십시오. 프록시 토글을 켜짐으로 전환하여 애플리케이션의 보안을 강화하십시오. 
4. 마찬가지로 런던 엔드포인트에 대한 `CNAME` 레코드를 설정하십시오.
   ![CNAME 레코드](images/solution1/cnames.png)

`mybluemix.net`이 아닌 다른 기본 도메인(예: `cf.appdomain.cloud` 또는 `cf.cloud.ibm.com`)을 사용하는 경우 [각각의 해당 시스템 도메인](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain)을 사용해야 합니다.
{:tip}

다른 DNS 제공자를 사용하는 경우 CNAME 레코드를 설정하는 단계는 DNS 제공자에 따라 다릅니다. 예를 들어, GoDaddy를 사용하는 경우 GoDaddy의 [도메인 도움말](https://www.godaddy.com/help/add-a-cname-record-19236) 안내를 따르십시오. 

사용자 정의 도메인을 통해 Cloud Foundry 애플리케이션에 도달하려면 [애플리케이션이 배치되는 Cloud Foundry 조직에 있는 도메인의 목록](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#updatingapps)에 사용자 정의 도메인을 추가해야 합니다. 완료되면 라우트를 애플리케이션 Manifest에 추가할 수 있습니다. 

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <your_app>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}

## 애플리케이션에 SSL 인증서 바인드
{: #ssl}

1. SSL 인증서를 확보하십시오. 예를 들어, https://www.godaddy.com/web-security/ssl-certificate에서 구입하거나 https://letsencrypt.org/에서 무료로 생성할 수 있습니다. 
2. 애플리케이션 **개요** > **라우트** > **도메인 관리**로 이동하십시오. 
3. SSL 인증서 업로드 단추를 클릭한 후 인증서를 업로드하십시오. 
5. HTTP 대신 HTTPS를 사용하여 애플리케이션에 액세스하십시오. 

## 애플리케이션 성능 모니터링
{: #monitor}

다중 위치 애플리케이션의 상태를 확인해 봅니다. 

1. 애플리케이션 대시보드에서 **모니터링**을 선택하십시오. 
2. **모든 테스트 보기**를 클릭하십시오.
   ![](images/solution1/alert_frequency.png)

가용성 모니터링은 사용자가 영향을 받기 전에 성능 문제를 미리 발견하여 수정하기 위해 전 세계 여러 위치에서 24시간 내내 합성 테스트를 실행합니다. 애플리케이션을 위해 사용자 정의 라우트를 구성한 경우 테스트 정의를 변경하여 해당 사용자 정의 도메인을 통해 애플리케이션에 액세스하십시오. 

## 리소스 제거

* 도구 체인 삭제
* 두 위치에 배치된 두 개의 Cloud Foundry 애플리케이션 삭제
* GLB, 원본 풀 및 상태 검사 삭제
* DNS 구성 삭제
* Internet Services 인스턴스 삭제

## 관련 컨텐츠

[Cloudant 데이터베이스 추가](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[Cloud Foundry 애플리케이션 자동 스케일링](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
