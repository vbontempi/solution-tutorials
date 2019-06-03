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


# 서버리스 웹 애플리케이션 및 API
{: #serverless-api-webapp}

이 튜토리얼에서는 GitHub Pages에서 정정 웹 사이트 컨텐츠를 호스팅하고 {{site.data.keyword.openwhisk}}를 사용하여 애플리케이션 백엔드를 구현하여 서버리스 웹 애플리케이션을 작성합니다. 

이벤트 중심 플랫폼으로서 {{site.data.keyword.openwhisk_short}}는 [다양한 유스 케이스](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)를 지원합니다. 웹 애플리케이션 및 API 빌드가 이 중 하나입니다. 웹 앱의 경우 이벤트는 웹 브라우저(또는 REST 클라이언트)와 웹 앱(HTTP 요청) 간 상호작용입니다. 가상 머신, 컨테이너 또는 Cloud Foundry 런타임을 프로비저닝하여 백엔드를 배치하는 대신 서버리스 플랫폼을 사용하여 백엔드 API를 구현할 수 있습니다. 이는 유휴 시간에 대한 지불을 방지하고 필요에 따라 플랫폼을 확장하기 위해 적절한 솔루션일 수 있습니다. 

{{site.data.keyword.openwhisk_short}}의 액션(기능)은 웹 클라이언트가 이용할 준비가 된 HTTP 엔드포인트로 전환될 수 있습니다. 웹에 대해 사용으로 설정된 경우 이 액션을 *웹 액션*이라고 합니다. 웹 조치가 확보되면 웹 조치를 API 게이트웨이가 있는 모든 기능을 갖춘 API로 어셈블할 수 있습니다. API 게이트웨이는 API를 노출할 {{site.data.keyword.openwhisk_short}}의 컴포넌트입니다. 이는 보안, OAuth 지원, 비율 제한, 사용자 정의 도메인 지원과 함께 제공됩니다. 

## 목표

* 서버리스 백엔드 및 데이터베이스 배치
* REST API 노출
* 정적 웹 사이트 호스팅
* 선택사항: REST API에 대해 사용자 정의 도메인 사용

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

이 튜토리얼에 표시된 애플리케이션은 사용자가 메시지를 게시할 수 있는 단순 방명록 웹 사이트입니다. 

<p style="text-align: center;">

   ![아키텍처](./images/solution8/Architecture.png)
</p>

1. 사용자가 GitHub Pages에서 호스팅되는 애플리케이션에 액세스합니다. 
2. 웹 애플리케이션이 백엔드 API를 호출합니다. 
3. 백엔드 API가 API 게이트웨이에서 정의됩니다. 
4. API 게이트웨이가 요청을 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)에 전달합니다. 
5. {{site.data.keyword.openwhisk_short}} 액션이 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)를 사용하여 방명록 항목을 저장하고 검색합니다. 

## 시작하기 전에
{: #prereqs}

이 안내서에서는 GitHub Pages를 사용하여 정적 웹 사이트를 호스팅합니다. 공용 GitHub 계정을 가지고 있는지 확인하십시오. 

## 방명록 데이터베이스 작성

{{site.data.keyword.cloudant_short_notm}}를 작성하는 것부터 시작해 봅니다. {{site.data.keyword.cloudant_short_notm}}는 유연한 JSON 스키마를 활용하는 현대 웹 및 모바일 애플리케이션을 위해 설계된 완전히 관리되는 데이터 계층입니다. {{site.data.keyword.cloudant_short_notm}}는 Apache CouchDB를 기반으로 하여 이와 호환되며 보안 HTTPS API를 통해 액세스 가능하여 애플리케이션이 성장함에 따라 확장됩니다. 

1. 카탈로그에서 **Cloudant**를 선택하십시오. 
2. 서비스 이름을 **guestbook-db**로 설정하고 **레거시 인증 정보와 IAM 모두 사용**을 인증 방법으로 선택하고 **작성**을 클릭하십시오. 
3. 다시 리소스 목록의 이름 열 아래에서 ***guestbook-db** 항목을 클릭하십시오. 참고: 서비스가 프로비저닝될 때까지 기다려야 할 수 있습니다.  
4. 서비스 세부사항 화면에서 ***Cloudant 대시보드 실행***을 클릭하여 다른 브라우저 탭에서 여십시오. 참고: Cloudant 인스턴스에 로그인해야 할 수 있습니다.  
5. ***데이터베이스 작성***을 클릭하고 ***방명록***이라는 데이터베이스를 작성하십시오. 

  ![](images/solution8/Create_Database.png)

6. 서비스 세부사항 탭으로 돌아가서 **서비스 인증 정보** 아래에서
   1. **새 인증 정보**를 작성하고 기본값을 승인한 후 **추가**를 클릭하십시오. 
   2. 액션 아래에서 **인증 정보 보기**를 클릭하십시오. 나중에 Cloud Functions 액션이 Cloudant 서비스에 대해 읽기/쓰기를 수행할 수 있도록 하기 위해 이 인증 정보가 필요합니다. 

## 서버리스 액션 작성

이 절에서는 서버리스 액션(일반적으로 Functions라고 함)을 작성합니다. {{site.data.keyword.openwhisk}}(Apache OpenWhisk 기반)는 수신 이벤트에 대한 응답으로 기능을 실행하고 사용하지 않을 때는 비용이 들지 않는 FaaS(Function-as-a-Service) 플랫폼입니다. 

![](images/solution8/Functions.png)

### 방명록 항목을 저장하는 조치의 시퀀스

한 조치의 출력이 다음 조치의 입력 역할을 수행하는 방식의 조치 체인인 **시퀀스**를 작성합니다. 작성할 첫 번째 시퀀스는 게스트 메시지를 지속시키는 데 사용됩니다. 이름, 이메일 ID 및 주석이 제공된 경우 시퀀스에서는 다음을 수행합니다. 
   * 지속될 문서를 작성합니다. 
   * {{site.data.keyword.cloudant_short_notm}} 데이터베이스에 문서를 저장합니다. 

첫 번째 조치를 작성하는 것에서부터 시작하십시오. 

1. **기능** https://{DomainName}/openwhisk로 전환하십시오. 
2. 왼쪽 분할창에서 **조치**를 클릭한 후 **작성**을 클릭하십시오. 
3. 이름 `prepare-entry-for-save`를 사용하여 **조치를 작성**하고 **Node.js**를 런타임으로 선택하십시오(참고: 최신 버전 선택). 
4. 기존 코드를 아래의 코드 스니펫으로 대체하십시오. 
   ```js
   /**
    * Prepare the guestbook entry to be persisted
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. **저장**하십시오. 

그런 다음 조치를 시퀀스에 추가하십시오. 

1. **엔클로징 시퀀스**를 클릭한 후 **시퀀스에 추가**를 클릭하십시오. 
1. 시퀀스 이름에 대해 `save-guestbook-entry-sequence`를 입력한 후 **작성 및 추가**를 클릭하십시오. 

마지막으로 두 번째 조치를 시퀀스에 추가하십시오. 

1. **save-guestbook-entry-sequence**를 클릭한 후 **추가**를 클릭하십시오. 
1. **공용 사용**, **Cloudant**를 선택한 후 **조치** 아래에서 **create-document**를 선택하십시오. 
1. **새 바인딩**을 작성하십시오.  
1. 이름에 대해 `binding-for-guestbook`을 입력하십시오. 
1. Cloudant 인스턴스에 대해 `Input your own credentials`를 선택하고 cloudant 서비스에 대해 캡처된 인증 정보로 사용자 이름, 비밀번호, 호스트 및 데이터베이스 = `guestbook` 필드를 채우고 **추가**를 클릭한 후 **저장**을 클릭하십시오.
   {: tip}
1. 이를 테스트하기 위해 **입력 변경**을 클릭한 후 아래 JSON을 입력하십시오.
    ```json
    {
      "name": "John Smith",
      "email": "john@smith.com",
      "comment": "this is my comment"
    }
    ```
    {: codeblock}
1. **적용**한 후 **호출**하십시오.
    ![](images/solution8/Save_Entry_Invoke.png)

### 항목을 검색하는 조치의 시퀀스

두 번째 시퀀스는 기존 방명록 항목을 검색하는 데 사용됩니다. 이 시퀀스는 다음을 수행합니다. 
   * 데이터베이스의 모든 문서를 나열합니다. 
   * 문서를 형식화한 후 리턴합니다. 

1. **기능** 아래에서 **조치**를 클릭한 후 새 Node.js 조치를 **작성**하여 이름을 `set-read-input`으로 지정하십시오. 
2. 기존 코드를 아래의 코드 스니펫으로 대체하십시오. 이 조치는 적절한 매개변수를 다음 조치에 전달합니다. 
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. **저장**하십시오. 

조치를 시퀀스에 추가하십시오. 

1. **엔클로징 시퀀스**, **시퀀스에 추가** 및 **새로 작성**을 클릭하십시오. 
1. **조치 이름**에 대해 `read-guestbook-entries-sequence`를 입력한 후 **작성 및 추가**를 클릭하십시오. 

시퀀스를 완료하십시오. 

1. **read-guestbook-entries-sequence** 시퀀스를 클릭한 후 **추가**를 클릭하여 두 번째 조치를 작성 및 추가하여 Cloudant에서 문서를 가져오십시오. 
1. **공용 사용** 아래에서 **{{site.data.keyword.cloudant_short_notm}}**를 선택한 후 **list-documents**를 선택하십시오. 
1. **내 바인딩** 아래에서 **binding-for-guestbook** 및 **추가**를 선택하여 이 공용 조치를 작성하여 시퀀스에 추가하십시오. 
1. **추가**를 다시 클릭하여 {{site.data.keyword.cloudant_short_notm}}의 문서를 형식화할 세 번째 조치를 작성 및 추가하십시오. 
1. **새로 작성** 아래에서 이름에 대해 `format-entries`를 입력한 후 **작성 및 추가**를 클릭하십시오. 
1. **format-entries**를 클릭한 후 코드를 아래 코드로 대체하십시오. 
   ```js
   const md5 = require('spark-md5');

   function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. **저장**하십시오. 
1. **조치**를 클릭한 후 **read-guestbook-entries-sequence**를 클릭하여 시퀀스를 선택하십시오. 
1. **저장**을 클릭한 후 **호출**을 클릭하십시오. 출력은 다음과 비슷합니다.
   ![](images/solution8/Read_Entries_Invoke.png)

## API 작성

![](images/solution8/Cloud_Functions_API.png)

1. 조치 https://{DomainName}/openwhisk/actions로 이동하십시오. 
2. **read-guestbook-entries-sequence** 시퀀스를 선택하십시오. 이름 옆에서 **웹 조치**를 클릭한 후 **웹 조치 사용** 및 **저장**을 선택하십시오. 
3. **save-guestbook-entry-sequence** 시퀀스에 대해 동일한 작업을 수행하십시오. 
4. API https://{DomainName}/openwhisk/apimanagement로 이동한 후 **{{site.data.keyword.openwhisk_short}} API를 작성**하십시오. 
5. 이름을 `guestbook`으로 설정하고 기본 경로를 `/guestbook`으로 설정하십시오. 
6. **오퍼레이션 작성**을 클릭한 후 방명록 항목을 검색할 오퍼레이션을 작성하십시오. 
   1. **경로**를 `/entries`로 설정하십시오. 
   2. **verb**를 `GET*`로 설정하십시오. 
   3. **read-guestbook-entries-sequence** 조치를 선택하십시오. 
7. **오퍼레이션 작성**을 클릭한 후 방명록 항목을 지속시킬 오퍼레이션을 작성하십시오. 
   1. **경로**를 `/entries`로 설정하십시오. 
   2. **verb**를 `PUT`으로 설정하십시오. 
   3. **save-guestbook-entry-sequence** 조치를 선택하십시오. 
8. API를 저장한 후 노출하십시오. 

## 웹 앱 배치

1. Guestbook 사용자 인터페이스 저장소 https://github.com/IBM-Cloud/serverless-guestbook을 공용 GitHub에 대해 분기 실행하십시오. 
2. **docs/guestbook.js**를 수정하고 **apiUrl**의 값을 API 게이트웨이에서 제공하는 라우트로 대체하십시오. 
3. 수정된 파일을 분기 실행된 저장소에 대해 커미트하십시오. 
4. 저장소의 설정 페이지에서 **GitHub Pages**로 스크롤하여 소스를 **마스터 분기/문서 폴더**로 변경한 후 저장하십시오. 
5. 저장소에 대한 공용 페이지에 액세스하십시오. 
6. 이전에 작성된 "테스트" 방명록 항목이 표시되어야 합니다. 
7. 새 항목을 추가하십시오. 

![](images/solution8/Guestbook.png)

## 선택사항: API에 대해 자체 도메인 사용

관리 API를 작성하면 `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`과 같은 기본 엔드포인트가 제공됩니다. 이 절에서는 사용자 정의 하위 도메인에서 제공되는 요청을 처리할 수 있도록 이 엔드포인트를 구성합니다. 

### 사용자 정의 도메인에 대한 인증서 얻기

사용자 정의 도메인을 통해 {{site.data.keyword.openwhisk_short}} 액션을 노출하려면 안전한 HTTPS 연결이 필요합니다. 서버리스 백엔드와 함께 사용하려고 하는 도메인 및 하위 도메인에 대한 SSL 인증서를 확보해야 합니다. *mydomain.com*이라는 도메인을 사용한다고 가정하면 *guestbook-api.mydomain.com*에서 액션이 호스팅될 수 있습니다. *guestbook-api.mydomain.com*(또는 **.mydomain.com*)에 대한 인증서가 발행되어야 합니다. 

[Let's Encrypt](https://letsencrypt.org/)에서 무료 SSL 인증서를 얻을 수 있습니다. 이 프로세스 동안 도메인의 소유자임을 증명하기 위해 DNS 인터페이스에서 TXT 유형의 DNS 레코드를 구성해야 할 수 있습니다.
{:tip}

도메인에 대한 SSL 인증서 및 개인 키를 확보하고 나면 이를 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 형식으로 변환해야 합니다. 

1. 인증서를 PEM 형식으로 변환하려면 다음 명령을 실행하십시오. 
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. 개인 키를 PEM 형식으로 변환하려면 다음 명령을 실행하십시오. 
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### 인증서를 중앙 저장소로 가져오기

1. 지원되는 위치에서 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 인스턴스를 작성하십시오. 
1. 서비스 대시보드에서 **인증서 가져오기**를 사용하십시오. 
   * **이름**을 사용자 정의 하위 도메인 및 도메인(예: `guestbook-api.mydomain.com`)으로 설정하십시오. 
   * PEM 형식의 **인증서 파일**을 찾아보십시오. 
   * PEM 형식의 **개인 키 파일**을 찾아보십시오. 
   * **가져오기**를 수행하십시오. 

### 관리 API에 대해 사용자 정의 도메인 구성

1. [API/사용자 정의 도메인](https://{DomainName}/apis/domains)으로 이동하십시오. 
1. 지역 선택기에서 조치를 배치한 위치를 선택하십시오. 
1. 조치 및 관리 API를 작성한 조직 및 영역에 연결된 사용자 정의 도메인을 찾으십시오. 조치 메뉴에서 **설정 변경**을 클릭하십시오. 
1. **기본 도메인/별명** 값을 기록해 두십시오. 
1. **사용자 정의 도메인 적용**을 선택하십시오. 
   1. **도메인 이름**을 사용할 도메인(예: `guestbook-api.mydomain.com`)으로 설정하십시오. 
   1. 인증서가 있는 {{site.data.keyword.cloudcerts_short}} 인스턴스를 선택하십시오. 
   1. 도메인에 대한 인증서를 선택하십시오. 
1. DNS 제공자로 이동하여 새 **DNS TXT 레코드**를 작성하여 도메인을 API 기본 도메인/별명에 맵핑하십시오. 설정이 적용되고 나면 DNS TXT 레코드를 제거할 수 있습니다. 
1. 사용자 정의 도메인 설정을 저장하십시오. 대화 상자에서 DNS TXT 레코드의 존재를 확인합니다. 
1. 마지막으로 DNS 제공자의 설정으로 돌아가서 사용자 정의 도메인(예: guestbook-api.mydomain.com)이 기본 도메인/별명을 가리키는 CNAME 레코드를 작성하십시오. 그러면 사용자 정의 도메인을 통한 트래픽이 백엔드 API로 라우팅됩니다. 

DNS 변경사항이 전파되고 나면 https://guestbook-api.mydomain.com/guestbook에서 방명록 API에 액세스할 수 있습니다. 

1. **docs/guestbook.js**를 편집하고 https://guestbook-api.mydomain.com/guestbook으로 **apiUrl**의 값을 업데이트하십시오. 
1. 수정된 파일을 커미트하십시오. 
1. 이제 애플리케이션이 사용자 정의 도메인을 통해 API에 액세스합니다. 

## 리소스 제거

* {{site.data.keyword.cloudant_short_notm}} 서비스 삭제
* {{site.data.keyword.openwhisk_short}}에서 API 삭제
* {{site.data.keyword.openwhisk_short}}에서 조치 삭제

## 관련 컨텐츠
* [서버리스에 대한 추가 안내서 및 샘플](https://developer.ibm.com/code/journey/category/serverless/)
* [{{site.data.keyword.openwhisk}} 시작하기](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [{{site.data.keyword.openwhisk}} 공통 유스 케이스](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [조치에서 REST API 작성](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
