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

# Apache Spark를 사용한 오픈 데이터 분석 및 시각화
{: #big-data-analytics-spark}

이 튜토리얼에서는 {{site.data.keyword.DSX_full}}, Jupyter Notebook 및 Apache Spark를 사용하여 오픈 데이터 세트를 분석하고 시각화합니다. 사용자는 먼저 인구 증가, 기대 수명 및 국가 ISO 코드를 나타내는 데이터를 하나의 데이터 프레임으로 결합하여 작업을 시작합니다. 그 후에는 인사이트를 발견하기 위해 Pixiedust라는 Python 라이브러리를 사용하여 데이터를 다양한 방식으로 조회하고 시각화합니다. 

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## 목표
{: #objectives}

* Apache Spark 및 {{site.data.keyword.DSX_short}}를 IBM Cloud에 배치
* Jupyter Notebook 및 Python 커널을 사용하여 작업
* 데이터 세트 가져오기, 변환, 분석 및 시각화

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 서비스 및 환경 설정
이 튜토리얼에서 사용되는 서비스를 프로비저닝하고 {{site.data.keyword.DSX_short}}에 프로젝트를 작성하여 작업을 시작하십시오. 

{{site.data.keyword.Bluemix_short}}를 위한 서비스는 [리소스 목록](https://{DomainName}/resources) 및 [카탈로그](https://{DomainName}/catalog/)에서 프로비저닝할 수 있습니다. {{site.data.keyword.DSX_short}}의 대시보드 및 프로젝트 설정에서 데이터 및 분석 서비스를 작성하거나 기존 서비스를 추가할 수도 있습니다.
{:tip}

1. [{{site.data.keyword.Bluemix_short}} 카탈로그](https://{DomainName}/catalog)에서 **AI** 섹션으로 이동하십시오. **{{site.data.keyword.DSX_short}}** 서비스를 작성하십시오. **시작하기** 단추를 클릭하여 **{{site.data.keyword.DSX_short}}** 대시보드를 실행하십시오. 
2. 대시보드에서 **프로젝트 작성** 타일을 클릭하고 **표준** > 프로젝트 작성을 선택하십시오. **이름** 필드에 이름으로 `1stProject`를 입력하십시오. 설명은 비워둘 수 있습니다. 
3. 페이지의 오른쪽에서는 **스토리지를 정의**할 수 있습니다. 스토리지를 이미 프로비저닝한 경우에는 목록에서 인스턴스를 선택하십시오. 그렇지 않은 경우에는 **추가**를 클릭하고 새 브라우저 탭의 지시사항을 따르십시오. 서비스 작성을 완료하고 나면 **새로 고치기**를 클릭하여 새 서비스를 보십시오. 
4. **작성** 단추를 클릭하여 프로젝트를 작성하십시오. 프로젝트의 개요 페이지로 경로 재지정됩니다.   
   ![](images/solution23/NewProject.png)
5. 개요 페이지에서 **설정**을 클릭하십시오. 
6. **연관된 서비스** 절에서 **서비스 추가**를 클릭하고 메뉴에서 **Spark**를 선택하십시오. 결과 화면에서는 기존 Spark 서비스 인스턴스를 선택하거나 새 인스턴스를 선택할 수 있습니다. 

## 노트북 작성 및 준비
[Jupyter Notebook](http://jupyter.org/)은 라이브 코드, 방정식, 시각화 및 이야기 텍스트를 포함하는 문서를 작성하고 공유할 수 있게 해 주는 오픈 소스 웹 애플리케이션입니다. 프로젝트에서는 노트북 및 기타 리소스가 구성됩니다. 
1. **자산** 탭에서 **노트북** 섹션으로 스크롤하여 **새 노트북**을 클릭하십시오. 
2. **공백** 노트북을 사용하십시오. `MyNotebook`을 **이름**으로 입력하십시오. 
3. **런타임 선택** 메뉴에서, 프로젝트의 설정에 추가한 Spark 인스턴스를 선택하십시오. 기본 **언어**를 **Python 3.5**로 유지하십시오. 
4. **노트북 작성**을 클릭하여 프로세스를 완료하십시오. 
5. 텍스트 및 명령을 입력하는 필드를 **셀**이라고 합니다. 비어 있는 셀에 다음 코드를 복사하여 [**Pixiedust** 패키지](https://pixiedust.github.io/pixiedust/use.html)를 가져오십시오. 도구 모음의 **실행** 아이콘을 클릭하거나 키보드에서 **Shift+Enter**를 눌러 셀을 실행하십시오. 
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

Jupyter 노트북에 대해 작업해 본 적이 없는 경우에는 오른쪽 상단 메뉴에서 **문서** 아이콘을 클릭하십시오. **데이터 분석**으로 이동한 후 [**노트북** 섹션](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics)으로 이동하여 [노트북 및 이를 구성하는 부분](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true)에 대해 자세히 알아보십시오.
{:tip}

## 데이터 로드
그 다음에는 세 개의 오픈 데이터 세트를 로드하고 노트북에서 사용할 수 있도록 설정하십시오. **Pixiedust** 라이브러리는 사용자가 손쉽게 [URL을 사용하여 **CSV** 파일을 로드](https://pixiedust.github.io/pixiedust/loaddata.html)할 수 있게 해 줍니다. 

1.  다음 행을 노트북의 다음 비어 있는 셀에 복사하되 아직 실행하지 마십시오. 
   ```Python
   df_pop = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
2. 다른 브라우저 탭에서 [커뮤니티](https://dataplatform.ibm.com/community?context=analytics) 섹션으로 이동하십시오. **데이터 세트** 중에서 [**Total population by country**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e)를 검색하고 타일을 클릭하십시오. 오른쪽 상단에서 **링크** 아이콘을 클릭하여 액세스 URI를 얻으십시오. URI를 복사하고 노트북 셀의 텍스트 **YourAccessURI**를 이 링크로 대체하십시오. 도구 모음의 **실행** 아이콘을 클릭하거나 **Shift+Enter**를 누르십시오. 
3. 다른 데이터 세트에 대해 위 단계를 반복하십시오. 다음 행을 노트북의 다음 비어 있는 셀에 복사하십시오. 
   ```Python
   df_life = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
4. **데이터 세트**가 있는 다른 브라우저 탭에서 [**Life expectancy at birth by country in total years**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895)를 검색하십시오. 다시 한 번 링크를 얻고 이를 사용하여 노트북 셀의 **YourAccessURI**를 대체한 후 **실행**하여 로드 프로세스를 시작하십시오. 
5. 세 데이터 세트 중 마지막으로, Github에 있는 오픈 데이터 세트의 콜렉션 중에서 국가 이름 및 해당 ISO 코드의 목록을 로드하십시오. 코드를 다음 비어 있는 노트북 셀에 복사하고 이를 실행하십시오. 
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

국가 코드의 목록은 나중에 정확한 국가 이름 대신 국가 코드를 사용하여 데이터 선택을 간소화하는 데 사용됩니다. 

## 데이터 변환
데이터가 사용 가능하도록 설정된 후에는 이를 약간 변환하여 세 개의 세트를 하나의 데이터 프레임으로 결합하십시오. 
1. 다음 코드 블록은 인구 데이터의 데이터 프레임을 재정의합니다. 이는 열의 이름을 바꾸는 SQL문을 사용하여 수행됩니다. 그 후에는 뷰가 작성되고 스키마가 출력됩니다. 이 코드를 다음 비어 있는 셀에 복사하고 이를 실행하십시오. 
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. 기대 수명 데이터에 대해 같은 작업을 반복하십시오. 이 코드는 스키마를 출력하는 대신 처음 10행을 출력합니다.   
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. 국가 데이터에 대해 스키마 변환을 반복하십시오. 
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. 이제 열 이름이 더 간단해졌으며 데이터 세트 간에 동일하므로 하나의 데이터 프레임으로 결합할 수 있습니다. 기대 수명 및 인구 데이터에 대해 **외부** 조인을 수행하십시오. 그 후 동일한 명령문에서 **내부** 조인을 수행하여 국가 코드를 추가하십시오. 모든 항목이 국가 및 연도에 따라 순서 지정됩니다. 출력은 데이터 프레임 **df_all**을 정의합니다. 내부 조인을 이용함으로써, 결과 데이터는 ISO 목록에 있는 국가만 포함하게 됩니다. 이 프로세스는 데이터에서 지역 및 기타 항목을 제거합니다. 
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. **Year**의 데이터 유형을 정수로 변경하십시오. 
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

결합된 데이터를 분석할 준비가 완료됩니다. 

## 데이터 분석
이 부분에서는 [Pixiedust를 사용하여 데이터를 다양한 차트에서 시각화](https://pixiedust.github.io/pixiedust/displayapi.html)합니다. 몇몇 국가의 기대 수명을 비교하는 것으로 작업을 시작하십시오. 

1. 코드를 다음 비어 있는 셀에 복사하고 이를 실행하십시오. 
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. 스크롤 가능한 테이블이 표시됩니다. 코드 블록 바로 아래에 있는 차트 아이콘을 클릭하여 **Line Chart**를 선택하십시오. **Pixiedust: Line Chart Options**가 있는 팝업 대화 상자가 표시됩니다. "기대 수명 비교"와 같은 **Chart Title**을 입력하십시오. 제공된 **Fields** 중에서 **Year**를 **Keys** 상자로, **Life**를 **Values** 영역으로 끌어 오십시오. **# of Rows to Display**에 대해 **1000**을 입력하십시오. **OK**를 눌러 선형 차트를 작성하십시오. 오른쪽에서 **mapplotlib**가 **Renderer**로 선택되었는지 확인하십시오. **Cluster By** 선택기를 클릭하고 **Country**를 선택하십시오. 다음과 같은 차트가 표시됩니다.
   ![](images/solution23/LifeExpectancy.png)

3. 2010년에 초점을 맞춘 차트를 작성하십시오. 코드를 다음 비어 있는 셀에 복사하고 이를 실행하십시오. 
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. 차트 선택기에서 **Map**을 선택하십시오. 구성 대화 상자에서 **Country**를 **Keys** 영역으로 끌어 오십시오. **Life**를 **Values** 상자로 이동하십시오. 첫 번째 차트와 마찬가지로 **# of Rows to Display**를 **1000**으로 늘리십시오. **OK**를 눌러 지도를 작성하십시오. **brunel**을 **Renderer**로 선택하십시오. 기대 수명에 따라 색칠된 세계 지도가 표시됩니다. 마우스를 사용하여 지도를 확대/축소할 수 있습니다.
   ![](images/solution23/LifeExpectancyMap2010.png)

## 튜토리얼 확장
아래 항목은 이 튜토리얼을 확장하여 적용하는 데 대한 몇 가지 아이디어 및 제안사항입니다. 
* 원하는 국가의 인구 증가 대비 기대 수명을 보여주는 조회를 작성하고 시각화
* 국가별 인구 증가율을 계산하고 시계 지도에 시각화
* 데이터 세트의 카탈로그에 추가 데이터를 로드하고 통합
* 결합된 데이터 세트를 파일 또는 데이터베이스로 내보내기

## 관련 컨텐츠
{:related}
아래 제공된 항목은 이 튜토리얼에서 다룬 주제와 관련된 링크입니다. 
* [Watson Data Platform](https://dataplatform.ibm.com): Use Watson Data Platform to collaborate and build smarter applications. Quickly visualize and discover insights from your data and collaborate across teams. 
* [PixieDust](https://www.ibm.com/cloud/pixiedust): Open source productivity tool for Jupyter Notebooks
* [Cognitive Class.ai](https://cognitiveclass.ai/): Data Science and Cognitive Computing Courses
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/): Things we made with data, so you can too
* [Analytics Engine](https://{DomainName}/catalog/services/analytics-engine): Develop and deploy analytics applications using open source Apache Spark and Apache Hadoop. 
