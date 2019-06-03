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

# Cloud 데이터에 대한 SQL 데이터베이스
{: #sql-database}

이 튜토리얼에서는 SQL(관계형) 데이터베이스 서비스를 프로비저닝하고 테이블을 작성하고 대형 데이터 세트(도시 정보)를 데이터베이스에 로드하는 방법을 보여줍니다. 그런 다음 웹 앱 "worldcities"를 배치하여 해당 데이터를 활용하고 클라우드 데이터베이스에 액세스하는 방법을 보여줍니다. 이 앱은 [Flask 프레임워크](http://flask.pocoo.org/)를 사용하여 Python으로 작성됩니다. 

![](images/solution5/Architecture.png)

## 목표

* SQL 데이터베이스 프로비저닝
* 데이터베이스 스키마(테이블) 작성
* 데이터 로드
* 앱 및 데이터베이스 서비스 연결(인증 정보 공유)
* 모니터링, 보안, 백업 및 복구

## 제품

이 튜토리얼에서는 다음과 같은 제품을 사용합니다. 
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## 시작하기 전에
{: #prereqs}

[GeoNames](http://www.geonames.org/)로 이동한 후 [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip) 파일을 다운로드하여 추출하십시오. 이 파일은 인구가 1000명 이상인 도시에 대한 정보를 보유합니다. 이를 데이터 세트로 사용합니다. 

## SQL 데이터베이스 프로비저닝
**[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)** 서비스의 인스턴스를 작성하는 것에서부터 시작하십시오. 

![](images/solution5/Catalog.png)

1. [{{site.data.keyword.Bluemix_short}} 대시보드](https://{DomainName})를 방문하십시오. 상단 탐색줄에서 **카탈로그**를 클릭하십시오. 
2. 왼쪽 분할창의 플랫폼 아래에서 **데이터 및 분석**을 클릭한 후 **{{site.data.keyword.dashdbshort_notm}}**를 선택하십시오. 
3. **엔트리** 플랜을 선택한 후 제안된 서비스 이름을 "sqldatabase"(나중에 이 이름을 사용함)로 변경하십시오. 데이터베이스 배치를 위한 위치를 선택한 후 올바른 조직 및 영역이 선택되었는지 확인하십시오. 
4. **작성**을 클릭하십시오. 잠시 후 성공 알림이 수신되어야 합니다. 
5. **리소스 목록**에서 새로 작성된 {{site.data.keyword.dashdbshort_notm}} 서비스에 대한 항목을 클릭하십시오. 
6. **열기**를 클릭하여 데이터베이스 콘솔을 실행하십시오. 콘솔을 처음으로 사용하는 경우에는 둘러보기를 수행하는 것이 좋습니다. 

## 테이블 작성
샘플 데이터를 보유할 테이블이 필요합니다. 콘솔을 사용하여 테이블을 작성하십시오. 

1. {{site.data.keyword.dashdbshort_notm}}에 대한 콘솔의 탐색줄에서 **탐색**을 클릭하십시오. 데이터베이스에 있는 기존 스키마 목록으로 이동합니다. 
2. "DASH"로 시작하는 스키마를 찾아서 클릭하십시오. 
3. **"+ 새 테이블"**을 클릭하여 테이블 이름 및 해당 열에 대한 양식을 가져오십시오. 
4. "cities"를 테이블 이름으로 입력하십시오. [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) 파일에서 열 정의를 복사한 후 열 및 데이터 유형에 대한 상자에 붙여넣으십시오. 
5. **작성**을 클릭하여 새 테이블을 정의하십시오.    
   ![](images/solution5/TableCitiesCreated.png)

## 데이터 로드
"cities" 테이블이 작성되었으므로 이 테이블에 데이터를 로드합니다. 이는 [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift) 마이그레이션 서비스를 활용하여 Swift 또는 Amazon S3 인터페이스를 가진 클라우드 오브젝트 스토리지(COS)에서 수행하거나 로컬 시스템에서 수행하는 등 다양한 방법으로 수행할 수 있습니다. 이 튜토리얼의 경우 사용자의 시스템에서 데이터를 업로드합니다. 이 프로세스 동안 파일 컨텐츠와 완전히 일치하도록 테이블 구조 및 데이터를 조정합니다. 

1. 상단 탐색에서 **로드**를 클릭하십시오. 그런 다음 **파일 선택** 아래에서 **파일 찾아보기**를 클릭하여 이 안내서의 첫 번째 절에서 다운로드한 "cities1000.txt" 파일을 찾아서 선택하십시오. 
2. **다음**을 클릭하여 스키마 개요로 이동하십시오. 다시 "DASH"로 시작하는 스키마를 선택한 후 "CITIES" 테이블을 선택하십시오. 다시 **다음**을 클릭하십시오.    

   테이블이 비어 있으므로 기존 데이터에 추가하거나 기존 데이터를 겹쳐쓸 차이점이 작성되지 않습니다.
   {:tip }
3. 이제 로드 프로세스 중에 "cities1000.txt" 파일의 데이터를 해석하는 방법을 사용자 정의하십시오. 파일에는 데이터만 포함되어 있으므로 먼저 "첫 번째 행에 헤더"를 사용 안함으로 설정하십시오. 다음으로 "0x09"를 구분자로 입력하십시오. 이는 파일에 있는 값이 탭(도표 작성자)으로 구분됨을 의미합니다. 마지막으로 "YYYY-MM-DD"를 날짜 형식으로 선택하십시오. 이제 모든 항목이 이 스크린샷과 비슷한 모양입니다.     
  ![](images/solution5/LoadTabSeparator.png)
4. **다음**을 클릭하면 로드 설정을 검토하도록 제안됩니다. 동의한 후 **로드 시작**을 클릭하여 "CITIES" 테이블에 데이터 로드를 시작하십시오. 진행상태가 표시됩니다. 데이터가 업로드되고 나면 몇 초 후에 로드가 완료되고 일부 통계가 제공됩니다.    
   ![](images/solution5/LoadProgressSteps.png)

## SQL을 사용하여 로드된 데이터 확인
데이터가 관계형 데이터베이스에 로드되었습니다. 오류는 없었지만 어쨌든 몇 가지 빠른 테스트를 실행해야 합니다. 기본 제공 SQL 편집기를 사용하여 일부 SQL문을 입력하고 실행하십시오. 

1. 상단 탐색에서 **SQL 실행**을 클릭하십시오.
   기본 제공 SQL 편집기 대신 {{site.data.keyword.dashdbshort_notm}}가 있는 서버 시스템 또는 데스크탑에서 전통적인 클라우드 기반 SQL 도구를 사용할 수 있습니다. 설정 메뉴에서 연결 정보를 찾을 수 있습니다. "서적" 아이콘(문서 및 도움말을 의미함) 뒤에 제공된 메뉴의 "다운로드" 섹션에 다운로드를 위한 일부 도구가 제공됩니다.
    {:tip }
2. "SQL 편집기"에서 다음과 같은 조회를 입력하거나 복사하십시오.    
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   그런 다음 **모두 실행** 단추를 누르십시오. 결과 섹션에서 로드 프로세스에 의해 보고된 것과 동일한 수의 행이 표시되어야 합니다.    
3. "SQL 편집기"에서 새 행에 다음 명령문을 입력하십시오. 
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. 편집기에서 위 명령문의 텍스트를 선택하십시오. **선택된 항목 실행** 단추를 클릭하십시오. 이 명령문만 지금 실행되어 결과 섹션에 몇몇 국가별 통계를 리턴해야 합니다. 

## 애플리케이션 코드 배치
[데이터베이스 앱에 대해 실행 준비가 된 코드가 이 Github 저장소에 있습니다. ](https://github.com/IBM-Cloud/cloud-sql-database)저장소를 복제하거나 다운로드한 후 IBM Cloud에 푸시하십시오. 

1. Github 저장소를 복제하십시오. 
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. 애플리케이션을 IBM Cloud에 푸시하십시오. 데이터베이스가 프로비저닝된 위치, 조직 및 영역에 로그인되어 있어야 합니다. 한 번에 한 행씩 이 명령을 복사하여 붙여넣으십시오. 
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. 푸시 프로세스가 완료되고 나면 앱에 액세스할 수 있어야 합니다. 추가 구성은 필요하지 않습니다. `manifest.yml` 파일은 앱과 "sqldatabase"라는 데이터베이스 서비스를 함께 바인드하도록 IBM Cloud에 지시합니다. 

## 보안, 백업 및 복구, 모니터링
{{site.data.keyword.dashdbshort_notm}}는 관리 서비스입니다. IBM은 환경 보안, 일일 백업 및 시스템 모니터링을 처리합니다. 엔트리 플랜에서 데이터베이스 환경은 관리가 감소하고 사용자에 대한 옵션이 구성된 멀티 테넌트 설정입니다. 하지만 엔터프라이즈 플랜 중 하나를 사용하는 경우에는 [사용자를 관리하고 추가 데이터베이스 보안을 구성하고](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html) [데이터베이스를 모니터하는](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html) 몇 가지 옵션이 있습니다.    

전통적인 관리 옵션 외에 [{{site.data.keyword.dashdbshort_notm}} 서비스가 모니터링, 사용자 관리, 유틸리티, 로드, 스토리지 액세스 등을 위한 REST API도 제공합니다. ](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html)해당 API의 실행 가능 Swagger 인터페이스는 "RES API" 아래의 "서적" 아이콘 뒤에 있는 메뉴에서 액세스할 수 있습니다. 모니터링 등을 위해 사용할 수 있는 일부 도구(예: IBM Data Server Manager)도 동일한 해당 메뉴에 있는 "다운로드" 섹션 아래에서 다운로드할 수 있습니다. 

## 앱 테스트
로드된 데이터 세트를 기반으로 도시 정보를 표시하는 앱이 최소값으로 감소합니다. 이는 검색 양식을 제공하여 도시 이름 및 몇몇 사전 구성된 도시를 지정합니다. 이들은 `/search?name=cityname`(검색 양식) 또는 `/city/cityname`(직접 지정된 도시)으로 변환됩니다. 두 요청 모두 백그라운드의 동일한 코드 행에서 제공됩니다. 도시 이름은 보안상 이유로 매개변수 마커를 사용하여 준비된 SQL문에 값으로 전달됩니다. 행은 데이터베이스에서 페치되고 렌더링을 위해 HTML 템플리트에 전달됩니다. 

## 정리
튜토리얼에서 사용하는 리소스를 정리하려면 다음과 같은 단계를 수행하십시오. 
1. [{{site.data.keyword.Bluemix_short}} 리소스 목록](https://{DomainName}/resources)을 방문하십시오. 앱을 찾으십시오. 
2. 앱의 메뉴 아이콘을 클릭한 후 **앱 삭제**를 선택하십시오. 대화 상자 창에서 관련 {{site.data.keyword.dashdbshort_notm}} 서비스를 삭제할 체크 표시를 선택하십시오. 
3. **삭제** 단추를 클릭하십시오. 앱 및 데이터베이스 서비스가 제거되고 다시 리소스 목록으로 이동합니다. 

## 튜토리얼 확장
이 앱을 확장하시겠습니까? 몇 가지 방법은 다음과 같습니다. 
1. 대체 이름에 대한 와일드카드 검색을 제공하십시오. 
2. 특정 인구 값 내에서만 특정 국가의 도시를 검색하십시오. 
3. CSS 스타일을 대체하고 템플리트를 확장하여 페이지 레이아웃을 변경하십시오. 
4. 양식 기반 새 도시 정보 작성을 허용하거나 기존 데이터(예: 인구)에 대한 업데이트를 허용하십시오. 

## 관련 컨텐츠
* 문서: [{{site.data.keyword.dashdbshort_notm}}용 IBM Knowledge Center](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* 관리 서비스, 데이터 백업, 데이터 암호화 및 보안 등과 관련된 질문에 답하는 [{{site.data.keyword.Db2_on_Cloud_long_notm}} 및 {{site.data.keyword.dashdblong_notm}}에 대해 자주 묻는 질문(FAQ)](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html)
* 개발자용 [무료 Db2 개발자 커뮤니티 에디션](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions)
* 문서: [ibm_db Python 드라이버에 대한 API 설명](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
