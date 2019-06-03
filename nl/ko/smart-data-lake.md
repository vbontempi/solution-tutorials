---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Object Storage를 사용하여 데이터 레이크 빌드
{: #smart-data-lake}

데이터 레이크라는 용어의 정의는 다양하지만 이 튜토리얼의 문맥에서 데이터 레이크는 조직적 사용을 위해 기본 형식으로 데이터를 저장하기 위한 접근 방식입니다. 이를 위해 {{site.data.keyword.cos_short}}를 사용하여 조직의 데이터 레이크를 작성합니다. {{site.data.keyword.cos_short}}와 SQL 조회를 결합하면 데이터 분석가가 SQL을 사용하여 데이터가 있는 위치에서 데이터를 조회할 수 있습니다. Jupyter 노트북의 SQL 조회 서비스를 활용하여 단순 분석도 수행합니다. 완료되면 비기술 사용자가 {{site.data.keyword.dynamdashbemb_notm}}를 사용하여 자체 인사이트를 검색하도록 허용하십시오. 

## 목표

- {{site.data.keyword.cos_short}}를 사용하여 원시 데이터 파일을 저장합니다. 
- SQL 조회를 사용하여 {{site.data.keyword.cos_short}}에서 직접 데이터를 조회합니다. 
- {{site.data.keyword.DSX_full}}에서 데이터를 세분화하고 분석합니다. 
- {{site.data.keyword.dynamdashbemb_notm}}를 사용하여 조직 전체에서 데이터를 공유합니다. 

## 사용되는 서비스

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [SQL 조회](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## 아키텍처

![아키텍처](images/solution29/architecture.png)

1. 원시 데이터가 {{site.data.keyword.cos_short}}에 저장됩니다. 
2. SQL 조회를 사용하여 데이터가 감소하거나 향상되거나 세분화됩니다. 
3. {{site.data.keyword.DSX}}에서 데이터 분석이 발생합니다. 
4. 비즈니스 라인이 웹 애플리케이션에 액세스합니다. 
5. {{site.data.keyword.cos_short}}에서 세분화된 데이터를 가져옵니다. 
6. {{site.data.keyword.dynamdashbemb_notm}}를 사용하여 비즈니스 라인 차트가 빌드됩니다. 

## 시작하기 전에

- [Git를 설치하십시오.](https://git-scm.com/)
- [{{site.data.keyword.Bluemix_notm}} CLI를 설치하십시오.](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
- [Aspera Connect를 설치하십시오.](http://downloads.asperasoft.com/connect2/)
- [Node.js 및 NPM을 설치하십시오.](https://nodejs.org)

## 서비스 작성

이 절에서는 데이터 레이크를 빌드하기 위해 필요한 서비스를 작성합니다. 

이 절에서는 명령행을 사용하여 서비스 인스턴스를 작성합니다. 또는 제공된 링크를 사용하여 카탈로그의 서비스 페이지에서 동일한 작업을 수행할 수 있습니다.
{: tip}

1. 명령행을 통해 {{site.data.keyword.cloud_notm}}에 로그인한 후 Cloud Foundry 계정을 대상으로 설정하십시오. [CLI 시작하기](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)를 참조하십시오.
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Cloud Foundry 별명을 사용하여 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)의 인스턴스를 작성하십시오. 이미 서비스 인스턴스가 있는 경우에는 기존 서비스 이름을 사용하여 `service-alias-create` 명령을 실행하십시오.
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. [SQL 조회](https://{DomainName}/catalog/services/sql-query)의 인스턴스를 작성하십시오.
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)의 인스턴스를 작성하십시오.
    ```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. Cloud Foundry 별명을 사용하여 [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)의 인스턴스를 작성하십시오.
    ```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. 작업 중 디렉토리로 변경한 후 다음 명령을 실행하여 대시보드 애플리케이션의 [GitHub 저장소](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard)를 복제하십시오. 그런 다음 애플리케이션을 Cloud Foundy 조직에 푸시하십시오. 애플리케이션은 [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml) 파일을 사용하여 위의 필수 서비스를 자동으로 바인드합니다.
    ```sh
    git clone https://github.com/IBM-Cloud/nodejs-data-lake-dashboard.git
    ```
    {: pre}
    ```sh
    cd nodejs-data-lake-dashboard
    ```
    {: pre}
    ```sh
    npm install
    ```
    {: pre}
    ```sh
    npm run push
    ```
    {: pre}

    배치 후 애플리케이션은 공용이며 랜덤 호스트 이름을 청취합니다. [리소스 목록](https://{DomainName}/resources) 페이지로 이동하여 Cloud Foundry 앱 아래에서 앱을 선택하고 URL을 보거나 `ibmcloud cf app dashboard-nodejs routes` 명령을 실행하여 라우트를 확인하십시오.
    {: tip}

7. 브라우저에서 공용 URL에 액세스하여 애플리케이션이 활성 상태인지 확인하십시오. 

![대시보드 랜딩 페이지](images/solution29/dashboard-start.png)

## 데이터 업로드

이 절에서는 기본 제공 {{site.data.keyword.CHSTSshort}}를 사용하여 {{site.data.keyword.cos_short}} 버킷에 데이터를 업로드합니다. {{site.data.keyword.CHSTSshort}}는 데이터가 버킷에 업로드될 때 데이터를 보호하며 [전송 시간을 상당히 줄일 수 있습니다](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/). 

1. [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD) CSV 파일을 다운로드하십시오. 이 파일은 81MB이므로 다운로드하는 데 몇 분이 걸릴 수 있습니다. 
2. 브라우저의 [리소스 목록](https://{DomainName}/resources)에서 **data-lake-cos** 서비스 인스턴스에 액세스하십시오. 
3. 데이터를 저장할 새 버킷을 작성하십시오. 
    - **버킷 작성** 단추를 클릭하십시오. 
    - **복원성** 드롭 다운에서 **지역별**을 선택하십시오. 
    - **위치**에서 **us-south**를 선택하십시오. 지금은 `us-south` 위치에서 작성된 버킷에만 {{site.data.keyword.CHSTSshort}}를 사용할 수 있습니다. 또는 또 다른 위치를 선택하고 다음 절에서 **표준** 전송 유형을 사용하십시오. 
    - 버킷 **이름**을 제공하고 **작성**을 클릭하십시오. *AccessDenied* 오류가 수신되면 더 고유한 버킷 이름으로 시도해 보십시오. 
4. {{site.data.keyword.cos_short}}에 CSV 파일을 업로드하십시오. 
    - 버킷에서 **오브젝트 추가** 단추를 클릭하십시오. 
    - **Aspera 고속 전송** 단일 선택 단추를 선택하십시오. 
    - **파일 추가** 단추를 클릭하십시오. 별도의 창(브라우저 창 뒤)에서 Aspera 플러그인이 열립니다. 
    - 이전에 다운로드한 CSV 파일을 찾아서 선택하십시오. 

![CSV 파일이 있는 버킷](images/solution29/cos-bucket.png)

## 데이터에 대한 작업

이 절에서는 시간 및 연령 속성을 기반으로 원래 원시 데이터 세트를 대상 코호트로 변환합니다. 이는 특정 관심사를 가지고 있거나 매우 큰 데이터 세트로 고생하는 데이터 레이크의 이용자에게 도움이 됩니다. 

SQL 조회를 사용하여 익숙한 SQL문으로 {{site.data.keyword.cos_short}}에서 데이터가 있는 위치에서 데이터를 조작합니다. SQL 조회는 CSV, JSON 및 Parquet에 대한 기본 제공 지원을 가지고 있습니다(추가 계산 서비스 또는 추출-변환-로드가 필요하지 않음). 

1. [리소스 목록](https://{DomainName}/resources)에서 **data-lake-sql** SQL 조회 서비스 인스턴스에 액세스하십시오. 
2. **UI 열기**를 선택하십시오. 
3. 이전에 업로드한 CSV 파일에서 직접 SQL을 실행하여 새 데이터 세트를 작성하십시오. 
    - 다음 SQL을 **Type SQL here ...** 텍스트 영역에 입력하십시오.
        ```
        SELECT
        `Dr Number` AS id,
        `Date Occurred` AS date,
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
        FROM cos://us-south/<your-bucket-name>/Traffic_Collision_Data_from_2010_to_Present.csv
        WHERE
        `Time Occurred` >= 1700 AND
        `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND
        `Victim Age` <= 35
        ```
        {: codeblock}
    - `FROM` 절에 있는 URL을 버킷의 이름으로 바꾸십시오. 
4. **대상**은 결과를 보유할 {{site.data.keyword.cos_short}} 버킷을 자동으로 작성합니다. **대상**을 `cos://us-south/<your-bucket-name>/results`로 변경하십시오. 
5. **실행** 단추를 클릭하십시오. 결과가 아래에 표시됩니다. 
6. **조회 세부사항** 탭에서 **결과 위치** URL 뒤에 있는 **실행** 아이콘을 클릭하여 중간 데이터 세트(이제 {{site.data.keyword.cos_short}}에도 저장됨)를 보십시오. 

![노트북](images/solution29/sql-query.png)

## SQL 조회와 Jupyter 노트북 결합

이 절에서는 Jupyter 노트북에서 SQL 조회 클라이언트를 사용합니다. 여기서는 데이터 분석 도구 내에서 {{site.data.keyword.cos_short}}에 저장된 데이터를 재사용합니다. 이 조합은 {{site.data.keyword.dynamdashbemb_notm}}와 함께 사용할 수 있는 {{site.data.keyword.cos_short}}에 자동으로 저장되는 데이터 세트도 작성합니다. 

1. {{site.data.keyword.DSX}}에서 새 Jupyter 노트북을 작성하십시오. 
    - 브라우저에서 [{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true)를 여십시오. 
    - **프로젝트 작성** 타일을 클릭한 후 **데이터 사이언스**를 클릭하십시오. 
    - **프로젝트 작성**을 클릭한 후 **프로젝트 이름**을 제공하십시오. 
    - **스토리지**가 **data-lake-cos**로 설정되어 있는지 확인하십시오. 
    - **작성**을 클릭하십시오. 
    - 결과로 생성되는 프로젝트에서 **프로젝트에 추가** 및 **노트북**을 클릭하십시오. 
    - **공백** 탭에서 **노트북 이름**을 입력하십시오. 
    - **언어** 및 **런타임**은 기본값으로 두십시오. **노트북 작성**을 클릭하십시오. 
2. 노트북에서 다음 명령을 **In [ ]: ** 입력 프롬프트에 추가하여 PixieDust 및 ibmcloudsql을 설치하고 가져온 후 **실행**하십시오.
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. {{site.data.keyword.cos_short}} API 키를 노트북에 추가하십시오. 그러면 SQL 조회 결과를 {{site.data.keyword.cos_short}}에 저장할 수 있습니다. 
    - 다음 **In [ ]: ** 프롬프트에서 다음을 추가한 후 **실행**하십시오.
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - 터미널에서 API 키를 작성하십시오.
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - **API 키**를 클립보드에 복사하십시오. 
    - API 키를 노트북의 텍스트 상자에 붙여넣고 `enter` 키를 누르십시오. 
    - 또한 API 키를 안전한 영구적인 위치에 저장해야 합니다. 노트북은 API 키를 저장하지 않습니다. 
4. SQL 조회 인스턴스의 CRN(Cloud Resource Name)을 노트북에 추가하십시오. 
    - 다음 **In [ ]: ** 프롬프트에서 CRN을 노트북의 변수에 지정하십시오.
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - 터미널에서 **ID** 특성의 CRN을 클립보드에 복사하십시오.
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - CRN을 작은따옴표 사이에 붙여넣은 후 **실행**하십시오. 
5. 또 다른 변수를 노트북에 추가하여 {{site.data.keyword.cos_short}} 버킷을 지정한 후 **실행**하십시오.
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. 또 다른 **In [ ]: ** 프롬프트에서 다음 명령을 실행한 후 **실행**하여 결과 세트를 확인하십시오. `SELECT`의 결과를 포함하는 버킷에 추가된 새 `accidents/jobid=<id>/<part>.csv*` 파일도 가지게 됩니다.
    ```python
    sqlClient = ibmcloudsql.SQLQuery(cloud_api_key, sql_crn, sql_cos_endpoint + '/accidents')

    data_source = sql_cos_endpoint + "/Traffic_Collision_Data_from_2010_to_Present.csv"

    query = """
    SELECT
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
    FROM  {}
    WHERE
        `Time Occurred` >= 1700 AND `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND `Victim Age` <= 35
    """.format(data_source)

    traffic_collisions = sqlClient.run_sql(query)
    traffic_collisions.head()
    ```
    {: codeblock}

## PixieDust를 사용하여 데이터 시각화

이 절에서는 트래픽 인시던트에 대한 핫 스팟 또는 패턴을 더 잘 식별하기 위해 PixieDust 및 Mapbox를 사용하여 이전 결과 세트를 시각화합니다. 

1. 공통 테이블 표현식을 작성하여 `location` 열을 별도의 `latitude` 및 `longitude` 열로 변환하십시오. 노트북의 프롬프트에서 다음을 **실행**하십시오.
    ```python
    query = """
    WITH location AS (
        SELECT
            id,
            cast(split(coordinates, ',')[0] as float) as latitude,
            cast(split(coordinates, ',')[1] as float) as longitude
        FROM (SELECT
                `Dr Number` as id,
                regexp_replace(Location, '[()]', '') as coordinates
            FROM {0}
        )
    )
    SELECT
        d.`Dr Number` as id,
        d.`Date Occurred` as date,
        d.`Time Occurred` AS time,
        d.`Area Name` AS area,
        d.`Victim Age` AS age,
        d.`Victim Sex` AS sex,
        l.latitude,
        l.longitude
    FROM {0} AS d
        JOIN
        location AS l
        ON l.id = d.`Dr Number`
    WHERE
        d.`Time Occurred` >= 1700 AND
        d.`Time Occurred` <= 2000 AND
        d.`Victim Age` >= 20 AND
        d.`Victim Age` <= 35 AND
        l.latitude != 0.0000 AND
        l.latitude != 0.0000
    """.format(data_source)

    traffic_location = sqlClient.run_sql(query)
    traffic_location.head()
    ```
    {: codeblock}
2. 다음 **In [ ]: ** 프롬프트에서 `display` 명령을 **실행**하여 PixieDust를 사용하여 결과를 보십시오.
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. 차트 드롭 다운 단추를 선택한 후 **맵**을 선택하십시오. 
4. `latitude` 및 `longitude`를 **키**에 추가하십시오. `id` 및 `age`를 **값**에 추가하십시오. **확인**을 클릭하여 맵을 보십시오. 
5. **저장** 아이콘을 클릭하여 노트북을 {{site.data.keyword.cos_short}}에 저장하십시오. 

![노트북](images/solution29/notebook-mapbox.png)

## 데이터 세트를 조직과 공유

데이터 레이크의 일부 사용자는 데이터 과학자가 아닙니다. {{site.data.keyword.dynamdashbemb_notm}}를 사용하여 비기술 사용자가 데이터 레이크에서 인사이트를 얻도록 허용할 수 있습니다. SQL 조회와 마찬가지로 {{site.data.keyword.dynamdashbemb_notm}}는 사전 빌드된 대시보드를 사용하여 {{site.data.keyword.cos_short}}에서 직접 데이터를 읽을 수 있습니다. 이 절에서는 모든 사용자가 데이터 레이크에 액세스하여 사용자 정의 대시보드를 빌드할 수 있게 하는 솔루션을 제공합니다. 

1. 이전에 {{site.data.keyword.Bluemix_notm}}에 푸시한 대시보드 애플리케이션의 공용 URL에 액세스하십시오. 
2. 의도한 레이아웃과 일치하는 템플리트를 선택하십시오. (다음 단계에서는 첫 번째 행의 두 번째 레이아웃을 사용합니다.)
3. `Selected sources` 사이드 쉘프에 표시되는 `Add a source` 단추를 사용하고 `bucket name` 아코디언을 펼친 후 `accidents/jobid=...` 테이블 항목 중 하나를 클릭하십시오. 오른쪽 상단의 X 아이콘을 사용하여 대화 상자를 닫으십시오. 
4. 왼쪽에서 `Visualizations` 아이콘을 클릭한 후 **요약**을 클릭하십시오. 
5. `accidents/jobid=...` 소스를 선택하고 `Table`을 펼친 후 차트를 작성하십시오. 
    - `id`를 끌어서 **값** 행에 놓으십시오. 
    - 상단의 아이콘을 사용하여 차트를 접으십시오. 
6. 다시 `Visualizations`에서 **트리 맵** 차트를 작성하십시오. 
    - `area`을 끌어서 **영역 계층 구조** 행에 놓으십시오. 
    - `id`를 끌어서 **크기** 행에 놓으십시오. 
    - 차트를 접고 결과를 보십시오. 

![대시보드 차트](images/solution29/dashboard-chart.png)

## 대시보드 탐색

이 절에서는 몇몇 추가 단계를 수행하여 대시보드 애플리케이션 및 {{site.data.keyword.dynamdashbemb_notm}}의 기능을 탐색합니다. 

1. 샘플 대시보드 애플리케이션의 도구 모음에 있는 **모드** 단추를 클릭하여 모드 보기 `VIEW`를 변경하십시오. 
2. 아래쪽 차트에 있는 색상 지정된 타일 또는 차트의 범례에 있는 `area` 값을 클릭하십시오. 그러면 로컬 필터가 탭에 적용되어 다른 차트가 필터에 해당되는 데이터를 표시합니다. 
3. 도구 모음에서 **저장** 단추를 클릭하십시오. 
    - 해당 입력 필드에 대시보드의 이름을 입력하십시오. 
    - **스펙** 탭을 선택하여 이 대시보드의 스펙을 보십시오. 스펙은 {{site.data.keyword.dynamdashbemb_notm}}의 기본 파일 형식입니다. 여기서 사용자가 작성한 차트 및 사용된 {{site.data.keyword.cos_short}} 데이터 소스에 대한 정보를 찾습니다. 
    - 대화 상자의 **저장** 단추를 사용하여 대시보드를 브라우저의 로컬 스토리지에 저장하십시오. 
4. 도구 모음의 **새로 작성** 단추를 클릭하여 새 대시보드를 작성하십시오. 저장된 대시보드를 열려면 **열기** 단추를 클릭하십시오. 대시보드를 삭제하려면 대시보드 열기 대화 상자에 있는 **삭제** 아이콘을 사용하십시오. 

프로덕션 애플리케이션에서는 URL, 사용자 이름 및 비밀번호 등의 정보를 암호화하여 일반 사용자가 볼 수 없게 하십시오. [데이터 소스 정보 암호화](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information)를 참조하십시오.
{: tip}

## 튜토리얼 확장

축하합니다. {{site.data.keyword.cos_short}}를 사용하여 데이터 레이크를 빌드했습니다. 데이터 레이크를 향상시키기 위한 추가적인 제안사항은 다음과 같습니다. 

- SQL 조회를 사용하여 추가적인 데이터 세트에 대해 실험
- [스트리밍 분석 및 SQL이 포함된 빅데이터 로그](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics)를 사용하여 다중 소스에서 데이터 레이크로 데이터 스트리밍
- 대시보드 애플리케이션의 코드를 편집하여 대시보드 스펙을 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 또는 {{site.data.keyword.cos_short}}에 저장
- 대시보드 애플리케이션에서 보안을 사용하기 위해 [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id) 서비스 인스턴스 작성

## 리소스 제거

사용된 서비스, 애플리케이션 및 키를 제거하려면 다음 명령을 실행하십시오. 

```sh
ibmcloud resource service-binding-delete dashboard-nodejs-dde dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-binding-delete dashboard-nodejs-cos dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-dde
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-cos
```
{: pre}
```sh
ibmcloud iam api-key-delete data-lake-cos-key
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-dde
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-cos
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-sql
```
{: pre}
```sh
ibmcloud service delete data-lake-studio
```
{: pre}
```sh
ibmcloud app delete dashboard-nodejs
```
{: pre}

## 관련 컨텐츠

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Jupyter 노트북](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
