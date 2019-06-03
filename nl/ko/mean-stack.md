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


# MEAN 스택을 사용하는 현대적 웹 애플리케이션
{: #mean-stack}

이 튜토리얼에서는 널리 사용되는 MEAN 스택을 사용하는 웹 애플리케이션을 작성하는 단계를 안내합니다. 이는 **M**ongo DB, **E**xpress 웹 프레임워크, **A**ngular 프론트 엔드 프레임워크 및 Node.js 런타임으로 구성됩니다. 사용자는 MEAN 스타터를 로컬로 실행하고, 관리 DBaaS(Database-as-a-Service)를 작성 및 사용하고, 앱을 {{site.data.keyword.cloud_notm}}에 배치하고 모니터하는 방법을 학습합니다.   

## 목표

{: #objectives}

- 스타터 Node.js 앱을 로컬에서 작성하여 실행합니다. 
- 관리 DBaaS(Database-as-a-Service)를 작성합니다. 
- Node.js 앱을 클라우드에 배치합니다. 
- MongoDB 리소스를 스케일링합니다. 
- 애플리케이션 성능을 모니터하는 방법을 학습합니다. 

## 사용되는 서비스

{: #products}

이 튜토리얼에서는 다음과 같은 {{site.data.keyword.Bluemix_notm}} 서비스를 사용합니다. 

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**주의:** 이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처

{:#architecture}

<p style="text-align: center;">

![아키텍처 다이어그램](images/solution7/Architecture.png)</p>

1. 사용자가 웹 브라우저를 사용하여 애플리케이션에 액세스합니다. 
2. Node.js 앱이 {{site.data.keyword.composeForMongoDB}} 데이터베이스에서 데이터를 페치합니다. 

## 시작하기 전에

{: #prereqs}

1. [Git을 설치하십시오.](https://git-scm.com/) 
2. [{{site.data.keyword.Bluemix_notm}} CLI를 설치하십시오.](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) 


애플리케이션을 로컬에서 개발하고 실행하려면 다음 항목이 필요합니다. 
1. [Node.js 및 NPM을 설치하십시오.](https://nodejs.org/) 
2. [MongoDB Community Edition을 설치하고 실행하십시오.](https://docs.mongodb.com/manual/administration/install-community/) 

## MEAN 앱을 로컬에서 실행

{: #runapplocally}

이 절에서는 로컬 MongoDB 데이터베이스를 실행하고, MEAN 샘플 코드를 복제하고, 애플리케이션을 로컬에서 실행하여 로컬 MongoDB 데이터베이스를 사용합니다. 

{: shortdesc}

1. [여기](https://docs.mongodb.com/manual/administration/install-community/)에 있는 지시사항에 따라 MongoDB 데이터베이스 로컬에서 설치하고 실행하십시오. 설치가 완료되면 아래 명령을 사용하여 **mongod** 서버가 실행 중인지 확인하십시오. 다음 명령을 사용하여 데이터베이스가 실행 중인지 확인하십시오. 
  ```sh
  mongo
  ```
  {: codeblock}

2. MEAN 스타터 코드를 복제하십시오. 

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. 필수 패키지를 설치하십시오. 

  ```sh
  npm install
  ```
  {: codeblock}

4. .env.example 파일을 .env에 복사하십시오. 필요한 정보를 편집하십시오. 자신의 SESSION_SECRET은 반드시 추가해야 합니다. 

5. node server.js를 실행하여 앱을 시작하십시오. 
  ```sh
  node server.js
  ```
  {: codeblock}

6. 애플리케이션에 액세스하고 새 사용자를 작성한 후 로그인하십시오. 

## 클라우드에 MongoDB 데이터베이스의 인스턴스 작성

{: #createdatabase}

이 절에서는 클라우드에 {{site.data.keyword.composeForMongoDB}} 데이터베이스를 작성합니다. {{site.data.keyword.composeForMongoDB}}는 일반적으로 구성하기 더 쉬우며 기본 제공 백업 및 스케일링 기능을 제공하는 DBaaS(Database-as-a-Service)입니다. [IBM Cloud 카탈로그](https://{DomainName}/catalog/?category=data)에는 다양한 유형의 데이터베이스가 있는 것을 볼 수 있습니다. {{site.data.keyword.composeForMongoDB}}를 작성하려면 아래 단계를 따르십시오. 

{: shortdesc}

1. 명령행을 통해 {{site.data.keyword.cloud_notm}} 계정에 로그인한 후 자신의 {{site.data.keyword.cloud_notm}} 계정을 대상으로 설정하십시오.  

  ```sh
  ibmcloud login
  ibmcloud target --cf
  ```
  {: codeblock}

  더 많은 CLI 명령은 [여기](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)서 볼 수 있습니다. 

2. {{site.data.keyword.composeForMongoDB}}의 인스턴스를 작성하십시오. 이는 [콘솔 UI](https://{DomainName}/catalog/services/compose-for-mongodb)를 사용하여 수행할 수도 있습니다. 애플리케이션이 **mean-starter-mongodb**를 찾도록 구성되어 있으므로 서비스 이름을 이 이름으로 지정해야 합니다. 

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## 앱을 클라우드에 배치

{: #deployapp}

이 절에서는 관리 MongoDB 데이터베이스로 사용되는 {{site.data.keyword.cloud_notm}}에 node.js 앱을 배치합니다. 소스 코드는 이전에 작성된 "mongodb" 서비스를 사용하도록 구성된 [**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml) 파일을 포함합니다. 이 애플리케이션은 VCAP_SERVICES 환경 변수를 사용하여 Compose MongoDB 데이터베이스 인증 정보에 액세스합니다. 이는 [server.js 파일](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js)에서 볼 수 있습니다.  

{: shortdesc}

1. 코드를 클라우드에 푸시하십시오. 

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. 코드가 푸시되고 나면 앱을 브라우저에서 볼 수 있습니다. `https://mean-random-name.mybluemix.net`과 같은 형식의 무작위 호스트 이름이 생성됩니다. 애플리케이션 URL은 콘솔 대시보드 또는 명령행에서 얻을 수 있습니다. ![라이브 앱](images/solution7/live-app.png)


## MongoDB 데이터베이스 리소스 스케일링
{: #scaledatabase}

서비스에서 추가 스토리지를 필요로 하거나 서비스에 할당된 스토리지 양을 줄이려는 경우에는 리소스를 스케일링하여 이를 수행할 수 있습니다. 

{: shortdesc}

1. 콘솔 **대시보드**를 사용하여 **연결** 섹션으로 이동한 후 **MongoDB 인스턴스** 데이터베이스를 클릭하십시오. 
2. **배치 세부사항** 패널에서 **리소스 스케일링** 옵션을 클릭하십시오.
  ![](images/solution7/mongodb-scale-show.png)
3. **슬라이더**를 조정하여 {{site.data.keyword.composeForMongoDB}} 데이터베이스 서비스에 할당되는 스토리지를 늘리거나 줄이십시오. 
4. **배치 스케일링**을 클릭하여 스케일링을 트리거하고 대시보드 개요로 돌아가십시오. 스케일링이 진행 중임을 알리기 위해 '스케일링 시작됨' 메시지가 페이지 맨 위에 표시됩니다.
  ![](images/solution7/scaling-in-progress.png)


## 애플리케이션 성능 모니터링
{: #monitorapplication}

애플리케이션의 상태를 확인하기 위해 기본 제공 Availability Monitoring 서비스를 사용할 수 있습니다. Availability Monitoring 서비스는 클라우드의 애플리케이션에 자동으로 연결됩니다. Availability Monitoring 서비스는 항상 전 세계 모든 위치에서 종합적인 테스트를 실행하여 성능 문제를 사전에 발견하고 방문자들에게 영향을 주기 전에 수정합니다. 모니터링 대시보드로 이동하려면 아래 단계를 따르십시오. 

{: shortdesc}

1. 콘솔 대시보드를 사용하여 애플리케이션에서 **모니터링** 탭을 선택하십시오. 
2. **모든 테스트 보기**를 클릭하여 테스트를 보십시오.
   ![](images/solution7/alert_frequency.png)


## 관련 컨텐츠

{: #related}

- 소스 제어 및 [Continuous Delivery](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops) 설정
- [여러 지역](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp)에서 제공되는 웹 애플리케이션 보안
- [REST API](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis) 작성, 보안 및 관리
