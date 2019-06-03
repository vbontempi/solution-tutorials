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

# 예측 기계 학습 모델 빌드, 배치, 테스트 및 재훈련
{: #create-deploy-retrain-machine-learning-model}
이 튜토리얼에서는 예측 기계 학습 모델을 빌드하고, 이를 애플리케이션에서 사용할 수 있도록 API로 배치하고, 모델을 테스트하고, 피드백 데이터로 모델을 재훈련시키는 프로세스를 안내합니다. 이 모든 작업은 IBM Cloud의 통합되어 일체화된 셀프 서비스 경험 내에서 수행됩니다. 

이 튜토리얼에서는 **붓꽃 데이터 세트**를 사용하여 꽃의 품종을 분류하는 기계 학습 모델을 작성합니다. 

기계 학습 용어에서 분류는 지도 학습(올바르게 식별된 관찰에 대한 훈련 세트가 사용 가능한 학습)으로 간주됩니다.
{:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## 목표
{: #objectives}

* 데이터를 프로젝트로 가져옵니다. 
* 기계 학습 모델을 빌드합니다. 
* 모델을 배치하고 API를 사용해 봅니다. 
* 기계 학습 모델을 테스트합니다. 
* 지속적 학습 및 모델 평가를 위한 피드백 데이터 연결을 작성합니다. 
* 모델을 재훈련시킵니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## 시작하기 전에
{: #prereqs}
* IBM Watson Studio 및 Watson Knowledge Catalog는 IBM Watson의 일부인 애플리케이션입니다. IBM Watson 계정을 작성하려면 먼저 이들 애플리케이션 중 하나 또는 둘 다에 가입하십시오. 

   [IBM Watson 사용해 보기](https://dataplatform.ibm.com/registration/stepone)로 이동하여 IBM Watson 앱에 가입하십시오. 

## 데이터를 프로젝트로 가져오기

{:#import_data_project}

프로젝트는 특정 목표를 달성하기 위해 리소스를 구성하는 수단입니다. 프로젝트 리소스에는 데이터, 협업자, 그리고 Jupyter 노트북 및 기계 학습 모델과 같은 분석 도구가 포함될 수 있습니다. 

사용자는 데이터를 추가할 프로젝트를 작성하고 데이터 정제기에서 데이터 자산을 열어 데이터를 정리하고 구체화할 수 있습니다. 

**프로젝트 작성:**

1. [{{site.data.keyword.Bluemix_short}} 카탈로그](https://{DomainName}/catalog)로 이동하여 **AI** 섹션에서 [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services)를 선택하십시오. 서비스를 **작성**하십시오. **시작하기** 단추를 클릭하여 **{{site.data.keyword.DSX_short}}** 대시보드를 실행하십시오. 
2. **프로젝트** 작성 > **표준** 타일에서 **프로젝트 작성**을 클릭하십시오. 프로젝트에 대해 `iris_project`와 같은 이름과 선택적 설명을 추가하십시오. 
3. 기밀 데이터가 없으므로 **협업자가 될 수 있는 사용자 제한** 선택란을 선택하지 않은 상태로 두십시오. 
4. **스토리지 정의**에서 **추가**를 클릭하고 기존 Cloud Object Storage 서비스를 선택하거나 새 서비스를 작성하십시오(**Lite** 플랜 > 작성 선택). **새로 고치기**를 눌러 작성된 서비스를 보십시오. 
5. **작성**을 클릭하십시오. 새 프로젝트가 열리고 리소스 추가를 시작할 수 있습니다. 

**데이터 가져오기:**

앞서 언급한 바와 같이 여기서는 **붓꽃 데이터 세트**를 사용합니다. 이 붓꽃 데이터 세트는 1936년에 발표된 R.A. Fisher의 유명 논문인 [The Use of Multiple Measurements in Taxonomic Problems](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf)에서 사용되었으며 [UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/)에서도 찾아볼 수 있습니다. 이 작은 데이터 세트는 기계 학습 알고리즘 및 시각화를 테스트하는 데 종종 사용됩니다. 목표는 꽃받침과 꽃잎의 길이 및 너비 측정치로부터 붓꽃을 세 가지 품종(Setosa, Versicolor 또는 Virginica)으로 분류하는 것입니다. 붓꽃 데이터 세트는 각각 50개의 인스턴스가 있는 3개의 클래스를 포함하며, 각 클래스는 붓꽃의 유형에 해당합니다.
![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


각 클래스의 40개 인스턴스로 구성된 [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv)를 **다운로드**하십시오. 각 클래스의 나머지 10개 인스턴스는 모델을 재훈련하는 데 사용합니다. 

1. 프로젝트의 **자산**에서 **데이터 찾기 및 추가** 아이콘 ![데이터 찾기 아이콘을 표시합니다.](images/solution22-build-machine-learning-model/data_icon.png)을 클릭하십시오. 
2. **로드**에서 **찾아보기**를 클릭하고 다운로드한 `iris_initial.csv`를 업로드하십시오.
      ![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. `iris_initial.csv`가 추가되고 나면 이를 프로젝트의 **데이터 자산** 섹션 아래에서 볼 수 있습니다. 데이터 세트의 컨텐츠를 보려면 이름을 클릭하십시오. 

## 서비스 연관
{:#associate_services}
1. **설정**에서 **연관된 서비스**로 스크롤하여 **서비스 추가**를 클릭하고 **Spark**를 선택하십시오.
   ![](images/solution22-build-machine-learning-model/associate_services.png)
2. **Lite** 플랜을 선택하고 **작성**을 클릭하십시오. 기본값을 사용하고 **확인**을 클릭하십시오. 
3. **서비스 추가**를 다시 클릭하고 **Watson**을 선택하십시오. **Machine Learning** 타일에서 **추가**를 클릭하고 **Lite** 플랜을 선택한 후 **작성**을 클릭하십시오. 
4. 기본값을 그대로 두고 **확인**을 클릭하여 Machine Learning 서비스를 프로비저닝하십시오. 

## 기계 학습 모델 빌드

{:#build_model}

1. **프로젝트에 추가**를 클릭하고 **Watson Machine Learning 모델**을 선택하십시오. 이 대화 상자에서 **iris_model**을 이름으로 추가하고 선택적 설명을 추가하십시오. 
2. **Machine Learning 서비스** 섹션에서, 위 단계에서 연관시킨 Machine Learning 서비스를 볼 수 있습니다.
   ![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. **모델 빌더**를 모델 유형으로 선택하고 **Spark 서비스 또는 환경** 섹션에서 이전에 작성한 Spark 서비스를 선택하십시오. 
4. **수동**을 선택하여 모델을 수동으로 작성하십시오. **작성**을 클릭하십시오. 

   자동 방법의 경우에는 자동 데이터 준비(ADP)에 완전히 의존합니다. 수동 방법의 경우에는 ADP 변환기가 처리하는 일부 기능 외에 고유한 추정자(분석에서 사용되는 알고리즘)를 추가하고 구성할 수 있습니다.
   {:tip}

5. 다음 페이지에서 `iris_initial.csv`를 데이터 세트로 선택하고 **다음**을 클릭하십시오. 
6. **기법 선택** 페이지에서는 추가된 데이터 세트에 따라 레이블 열 및 특성 열이 미리 채워집니다. **species (String)**를 **레이블 열**로, **petal_length (Decimal)** 및 **petal_width (Decimal)**을 **특성 열**로 선택하십시오. 
7. **다중 클래스 분류**를 제안된 기법으로 선택하십시오.
   ![](images/solution22-build-machine-learning-model/model_technique.png)
8. **유효성 검증 분할**에 대해 다음 설정을 구성하십시오. 

   **훈련:** 50%,
   **테스트:** 25%,
   **예비:** 25%

9. **추정자 추가**를 클릭하고 **의사결정 트리 분류자**를 선택한 후 **추가**를 선택하십시오. 

   여러 추정자를 한 번에 평가할 수 있습니다. 예를 들면, 모델 훈련을 위한 추정자로 **의사결정 트리 분류자** 및 **랜덤 포레스트 분류자**를 추가하고 평가 출력에 따라 가장 적합한 것을 선택할 수 있습니다.
   {:tip}

10. **다음**을 클릭하여 모델을 훈련시키십시오. 상태가 **훈련 & 평가됨**으로 표시되면 **저장**을 클릭하십시오.
   ![](images/solution22-build-machine-learning-model/trained_model.png)

11. **개요**를 클릭하여 모델의 세부사항을 확인하십시오. 

## 모델 배치 및 API 사용해 보기

{:#deploy_model}

1. 작성된 모델에서 **배치** > **배치 추가**를 클릭하십시오. 
2. **웹 서비스**를 선택하십시오. `iris_deployment`와 같은 이름과 선택적 설명을 추가하십시오. 
3. **저장**을 클릭하십시오. 개요 페이지에서 새 웹 서비스의 이름을 클릭하십시오. 상태가 **DEPLOY_SUCCESS**가 되면 **구현**에서 스코어링 엔드포인트, 다양한 프로그래밍 언어로 된 코드 스니펫, API 스펙을 확인할 수 있습니다. 
4. **API 스펙 보기**를 클릭하여 {{site.data.keyword.pm_short}} API 엔드포인트를 보고 테스트하십시오.
   ![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   API에 대한 작업을 시작하려면 [{{site.data.keyword.Bluemix_short}} 리소스 목록](https://{DomainName}/resources/)에 있는 {{site.data.keyword.pm_short}} 서비스 인스턴스의 **서비스 인증 정보**에 있는 **사용자 이름** 및 **비밀번호**를 사용하여 **액세스 토큰**을 생성해야 합니다. **액세스 토큰**을 생성하려면 API 스펙 페이지에 있는 지시사항을 따르십시오.
   {:tip}
5. 온라인 예측을 수행하려면 `POST /online` API 호출을 사용하십시오. 
   * `instance_id`는 [{{site.data.keyword.Bluemix_short}} 리소스 목록](https://{DomainName}/resources/)에 있는 {{site.data.keyword.pm_short}}의 **서비스 인증 정보** 탭에서 찾을 수 있습니다. 
   * `deployment_id` 및 `published_model_id`는 배치의 **개요**에 있습니다. 
   *  `online_prediction_input`에 대해 아래 JSON을 사용하십시오. 

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * JSON 출력을 보려면 **사용해 보기**를 클릭하십시오. 

6. 이제 API 엔드포인트를 사용하여 모든 애플리케이션에서 이 모델을 호출할 수 있습니다. 

## 모델 테스트

{:#test_model}

1. **테스트**에서는 입력 데이터(특성 데이터)가 자동으로 채워지는 것을 볼 수 있습니다. 
2. **예측**을 클릭하면 차트에 **품종에 대한 예측 값**이 표시됩니다.
   ![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. JSON 입력 및 출력을 위해서는 활성 입력 및 출력 옆에 있는 아이콘을 클릭하십시오. 
4. 입력 데이터를 변경하고 모델 테스트를 계속할 수 있습니다. 

## 피드백 데이터 연결 작성

{:#create_feedback_connection}

1. 지속적 학습 및 모델 평가를 위해서는 새 데이터를 다른 위치에 저장해야 합니다. 피드백 데이터 연결 역할을 수행할 [{{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse) 서비스를 **엔트리** 플랜으로 작성하십시오. 
2. {{site.data.keyword.dashdbshort}} **관리** 페이지에서 **열기**를 클릭하십시오. 최상위 탐색에서 **로드**를 선택하십시오. 
3. **내 컴퓨터**에서 **파일 찾아보기**를 클릭하고 `iris_initial.csv`를 업로드하십시오. **다음**을 클릭하십시오. 
4. **DASHXXXX**(예: DASH1234)를 **스키마**로 선택한 후 **새 테이블**을 클릭하십시오. 이름을 `IRIS_FEEDBACK`으로 지정하고 **다음**을 클릭하십시오. 
5. 데이터 유형이 자동으로 발견됩니다. **다음**을 클릭한 후 **로드 시작**을 클릭하십시오.
   ![](images/solution22-build-machine-learning-model/define_table.png)
6. 새 대상 **DASHXXXX.IRIS_FEEDBACK**이 작성됩니다. 

   이 대상은 더 나은 성능 및 정밀도를 위해 모델을 재훈련시키는 다음 단계에서 사용합니다. 

## 모델 재훈련

{:#retrain_model}

1. [{{site.data.keyword.Bluemix_short}} 리소스 목록](https://{DomainName}/resources)으로 돌아와 이제까지 사용한 {{site.data.keyword.DSX_short}} 서비스에서 **프로젝트** > iris_project > **iris-model**(자산 아래에 있음) > 평가를 클릭하십시오. 
2. **성능 모니터링**에서 **성능 모니터링 구성**을 클릭하십시오. 
3. 성능 모니터링 구성 페이지에서 다음 작업을 수행하십시오. 
   * Spark 서비스를 선택하십시오. 예측 유형이 자동으로 채워집니다. 
   * **weightedPrecision**을 메트릭으로 선택하고 선택적 임계값으로 `0.98`을 설정하십시오. 
   * **새 연결 작성**을 클릭하여 위 절에서 작성한 IBM Db2 Warehouse on Cloud를 가리키십시오. 
   * Db2 Warehouse 연결을 선택하고 연결 세부사항이 채워지면 **작성**을 클릭하십시오.
     ![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * **피드백 데이터 참조 선택**을 클릭하고 IRIS_FEEDBACK 테이블을 가리킨 후 **선택**을 클릭하십시오.
     ![](images/solution22-build-machine-learning-model/select_source.png)
   * **재평가를 위해 필요한 레코드 개수** 상자에서 재훈련을 트리거하는 최소 레코드 수를 입력하십시오. **10**을 사용하거나 공백으로 두어 기본값인 1000을 사용하십시오. 
   * **자동 재훈련** 상자에서 다음 옵션 중 하나를 선택하십시오. 
     - 모델 성능이 설정한 임계값 밑으로 떨어질 때마다 자동 재훈련을 시작하려면 **모델 성능이 임계값 아래인 경우**를 선택하십시오. 이 튜토리얼에서는 정밀도가 임계값(.98)보다 낮으므로 이 옵션을 선택합니다. 
     - 자동 재훈련을 금지하려면 **수행 안함**을 선택하십시오. 
     - 성능에 관계없이 자동 재훈련을 시작하려면 **항상**을 선택하십시오. 
   * **자동 배치** 상자에서 다음 옵션 중 하나를 선택하십시오. 
     - 모델 성능이 이전 버전보다 좋을 때마다 자동 배치를 시작하려면 **모델 성능이 이전 버전보다 좋은 경우**를 선택하십시오. 이 튜토리얼에서는 모델 성능을 지속적으로 개선하는 것이 목표이므로 이 옵션을 선택합니다. 
     - 자동 배치를 금지하려면 **수행 안함**을 선택하십시오. 
     - 성능에 관계없이 자동 배치를 시작하려면 **항상**을 선택하십시오. 
   * **저장**을 클릭하십시오.
     ![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv) 파일을 다운로드하십시오. 그 후 **피드백 데이터 추가**를 클릭하고 다운로드한 csv 파일을 선택한 후 **열기**를 클릭하십시오. 
5. **새 평가**를 클릭하여 시작하십시오.
     ![](images/solution22-build-machine-learning-model/retraining_model.png)
6. 평가가 완료되면, **마지막 평가 결과** 섹션에서 개선된 **WeightedPrecision** 값을 확인할 수 있습니다. 

## 리소스 제거
{:removeresources}

1. [{{site.data.keyword.Bluemix_short}} 리소스 목록](https://{DomainName}/resources/)으로 이동하여 서비스를 작성한 위치, 조직 및 영역을 선택하십시오. 
2. 이 튜토리얼을 위해 작성한 각 {{site.data.keyword.DSX_short}}, {{site.data.keyword.sparks}}, {{site.data.keyword.pm_short}}, {{site.data.keyword.dashdbshort}} 및 {{site.data.keyword.cos_short}} 서비스를 삭제하십시오. 

## 관련 컨텐츠
{:related}

- [Watson Studio overview](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [기계 학습을 사용하여 이상 항목 발견](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [자동 모델 작성](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [Machine learning & AI in Watson Studio](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
