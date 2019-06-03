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

# CDN을 사용하여 정적 파일 전달 가속화
{: #static-files-cdn}

이 튜토리얼에서는 {{site.data.keyword.cos_full_notm}}에서 사용자가 생성한 컨텐츠 및 웹 사이트 자산(이미지, 동영상, 문서)을 호스팅 및 제공하는 방법과 전 세계 사용자에게 빠르고 안전하게 전달하기 위해 [{{site.data.keyword.cdn_full}}(CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)를 사용하는 방법을 설명합니다. 

웹 애플리케이션은 HTML 컨텐츠, 이미지, 동영상, 캐스케이딩 스타일시트, JavaScript 파일, 사용자가 생성한 컨텐츠 등 다양한 유형의 컨텐츠를 가지고 있습니다. 일부 컨텐츠는 자주 변경되고 다른 컨텐츠는 자주 변경되지 않고 일부 컨텐츠는 많은 사용자가 매우 자주 액세스하고 다른 컨텐츠는 가끔 액세스됩니다. 애플리케이션의 대상이 증가함에 따라 이 컨텐츠 제공 작업을 다른 컴포넌트에 넘겨 기본 애플리케이션을 위한 리소스를 확보하길 원할 수 있습니다. 애플리케이션 사용자가 전 세계 어디에 있든 애플리케이션 사용자와 가까운 위치에서 이 컨텐츠를 제공하길 원할 수도 있습니다. 

이러한 상황에서 Content Delivery Network를 사용하는 데는 여러 가지 이유가 있습니다. 
* CDN은 컨텐츠를 캐시에 저장하여 캐시에서 사용 불가능하거나 만료된 경우에만 컨텐츠를 원본(서버)에서 가져옵니다. 
* 전 세계에 있는 복수의 데이터 센터를 사용하여 CDN은 사용자와 가장 가까운 위치에서 캐시에 저장된 컨텐츠를 제공합니다. 
* 기본 애플리케이션과 다른 도메인에서 실행되면 브라우저가 더 많은 컨텐츠를 병렬로 로드할 수 있습니다. 대부분의 브라우저는 호스트 이름당 연결 수에 제한이 있습니다. 

## 목표
{: #objectives}

* {{site.data.keyword.cos_full_notm}} 버킷에 파일을 업로드합니다. 
* CDN(Content Delivery Network)을 사용하여 컨텐츠를 글로벌로 사용 가능하게 합니다. 
* Cloud Foundry 웹 애플리케이션을 사용하여 파일을 노출합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 제품을 사용합니다. 
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처

<p style="text-align: center;">
![아키텍처](images/solution3/Architecture.png)
</p>

1. 사용자가 애플리케이션에 액세스합니다. 
2. 애플리케이션은 Content Delivery Network를 통해 분배된 컨텐츠를 포함하고 있습니다. 
3. 컨텐츠가 CDN에서 사용 불가능하거나 만료된 경우 CDN은 원본에서 컨텐츠를 가져옵니다. 

## 시작하기 전에
{: #prereqs}

인프라 계정의 마스터 사용자에게 문의하여 다음과 같은 권한을 얻으십시오. 
   * CDN 계정 관리
   * 스토리지 관리
   * API 키

스토리지 및 CDN 서비스를 보고 사용하려면 이러한 권한이 필수입니다. 

## 웹 애플리케이션 코드 가져오기
{: #get_code}

이미지, 동영상, 캐스케이딩 스타일시트 등 다양한 유형의 컨텐츠를 가진 단순 웹 애플리케이션이 있다고 가정해 봅니다. 사용자는 스토리지 버킷에 컨텐츠를 저장한 후 버킷을 원본으로 사용하도록 CDN을 구성합니다. 

시작하려면 다음 애플리케이션 코드를 검색하십시오. 

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## Object Storage 작성
{: #create_cos}

{{site.data.keyword.cos_full_notm}}는 구조화되지 않은 데이터를 위해 유연하고 비용 효율이 높고 확장 가능한 클라우드 스토리지를 제공합니다. 

1. 콘솔에서 [카탈로그](https://{DomainName}/catalog/)로 이동한 후 스토리지 섹션에서 [**Object Storage**](https://{DomainName}/catalog/services/cloud-object-storage)를 선택하십시오. 
2. {{site.data.keyword.cos_full_notm}}의 새 인스턴스를 작성하십시오. 
4. 서비스 대시보드에서 **버킷 작성**을 클릭하십시오. 
5. 고유 버킷 이름(예: `username-mywebsite`)을 설정한 후 **작성**을 클릭하십시오. 버킷 이름에는 점(.)을 사용하지 마십시오. 

## 버킷에 파일 업로드
{: #upload}

이 절에서는 명령행 도구 **curl**을 사용하여 파일을 버킷에 업로드합니다. 

1. CLI에서 {{site.data.keyword.Bluemix_notm}}에 로그인한 후 IBM Cloud IAM에서 토큰을 가져오십시오. 
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. 이전 단계의 명령 출력에서 토큰을 복사하십시오. 
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. 쉽게 액세스할 수 있도록 토큰 및 버킷 이름의 값을 환경 변수로 설정하십시오. 
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. 이전에 다운로드한 웹 애플리케이션 코드의 컨텐츠 디렉토리에서 `a-css-file.css`, `a-picture.png` 및 `a-video.mp4`라는 파일을 업로드하십시오. 파일을 버킷의 루트에 업로드하십시오. 
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. 대시보드에서 파일을 보십시오.
   ![](images/solution3/Buckets.png)
6. 다음 예와 비슷한 링크를 사용하여 브라우저를 통해 파일에 액세스하십시오. 

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## CDN을 사용하여 파일을 글로벌로 사용 가능하게 만들기

이 절에서는 CDN 서비스를 작성합니다. CDN 서비스는 필요한 위치에 컨텐츠를 분배합니다. 처음으로 컨텐츠가 요청되면 컨텐츠는 호스트 서버({{site.data.keyword.cos_full_notm}}에서 사용자의 버킷)에서 네트워크로 가져온 후 다른 사용자가 호스트 서버에 다시 도달하기 위해 네트워크 대기 시간 없이 신속하게 액세스할 수 있도록 해당 네트워크에 남아 있습니다. 

### CDN 인스턴스 작성

1. 콘솔의 카탈로그로 이동한 후 네트워크 섹션에서 **Content Delivery Network**를 선택하십시오. 이 CDN은 Akamai를 기반으로 합니다. **작성**을 클릭하십시오. 
2. 다음 대화 상자에서 CDN의 **호스트 이름**을 사용자 정의 도메인으로 설정하십시오. 사용자 정의 도메인을 설정했지만 여전히 IBM이 제공한 CNAME을 통해 CDN 컨텐츠에 액세스할 수 있습니다. 따라서 사용자 정의 도메인을 사용하지 않으려는 경우에는 임의의 이름을 설정할 수 있습니다. 
3. **사용자 정의 CNAME** 접두부를 고유 값으로 설정하십시오. 
4. 다음으로 **원본 구성** 아래에서 **Object Storage**를 선택하여 COS용 CDN을 구성하십시오. 
5. **엔드포인트**를 버킷 API 엔드포인트(예: *s3-api.us-geo.objectstorage.softlayer.net*)로 설정하십시오. 
6. **호스트 헤더** 및 **경로**는 공백으로 두십시오. **버킷 이름**을 *your-bucket-name*으로 설정하십시오. 
7. HTTP(80) 포트와 HTTPS(443) 포트를 둘 다 사용으로 설정하십시오. 
8. **SSL 인증서**에 대해 *DV SAN 인증서*를 선택하십시오(사용자 정의 도메인을 사용하려는 경우). 그렇지 않으면 CNAME을 통해 스토리지에 액세스할 수 있도록 **와일드카드 인증서* 옵션을 선택하십시오. 
9. **작성**을 클릭하십시오. 

### CDN CNAME을 통해 컨텐츠에 액세스

1. [목록](https://{DomainName}/classic/network/cdn)에서 CDN 인스턴스를 선택하십시오. 
2. 이전에 *DV SAN 인증서*를 선택한 경우에는 초기 설정이 완료되면 도메인 유효성 검증 프롬프트가 표시됩니다. **도메인 유효성 검증 보기**를 클릭하면 표시되는 단계를 수행하십시오. 
3. **세부사항** 패널에는 CDN의 **CNAME**과 **호스트 이름**이 둘 다 표시됩니다. 
4. `https://your-cdn-cname.cdnedge.bluemix.net/a-picture.png` 또는 `https://your-cdn-hostname/a-picture.png`(사용자 정의 도메인을 사용하는 경우)를 사용하여 파일에 액세스하십시오. 파일 이름을 생략하는 경우에는 S3 ListBucketResult가 대신 표시됩니다. 

## Cloud Foundry 애플리케이션 배치

애플리케이션에는 이제 {{site.data.keyword.cos_full_notm}}에서 호스팅되는 파일에 대한 참조가 포함된 public/index.html 웹 페이지가 포함되어 있습니다. 백엔드 `app.js`는 이 웹 페이지를 제공하고 플레이스홀더를 CDN의 실제 위치로 대체합니다. 이러한 방식으로 웹 페이지에서 사용하는 모든 자산이 CDN에 의해 제공됩니다. 

1. 터미널에서 코드를 체크아웃한 디렉토리로 이동하십시오. 
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. 애플리케이션을 시작하지 않고 푸시하십시오. 
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. 앱이 CDN 컨텐츠를 참조할 수 있도록 CDN_NAME 환경 변수를 구성하십시오. 
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. 앱을 시작하십시오. 
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. 웹 브라우저를 사용하여 앱에 액세스하면 페이지 스타일시트, 사진 및 동영상이 CDN에서 로드됩니다. 

![](images/solution3/Application.png)

CDN을 {{site.data.keyword.cos_full_notm}}와 함께 사용하는 것은 파일을 호스팅하고 전 세계 사용자에게 제공할 수 있는 강력한 조합입니다. 또한 {{site.data.keyword.cos_full_notm}}를 사용하면 사용자가 애플리케이션에 업로드한 모든 파일을 저장할 수 있습니다. 

## 리소스 제거

* Cloud Foundry 애플리케이션 삭제
* Content Delivery Network 서비스 삭제
* {{site.data.keyword.cos_full_notm}} 서비스 또는 버킷 삭제

## 관련 컨텐츠

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[{{site.data.keyword.cos_full_notm}}에 대한 액세스 관리](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[CDN 시작하기](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
