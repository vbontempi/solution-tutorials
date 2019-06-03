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

# IoT 데이터 수집, 시각화 및 분석
{: #gather-visualize-analyze-iot-data}
이 튜토리얼은 IoT 디바이스를 설정하고, {{site.data.keyword.iot_short_notm}}에서 데이터를 수집하고, 데이터를 탐색하며 시각화를 작성하고, 그 후 고급 기계 학습 서비스를 사용하여 데이터를 분석하고 히스토리 데이터에서 이상 항목을 발견하는 작업을 안내합니다.
{:shortdesc}

## 목표
{: #objectives}

* IoT 시뮬레이터를 설정합니다. 
* 콜렉션 데이터를 {{site.data.keyword.iot_short_notm}}에 전송합니다. 
* 시각화를 작성합니다. 
* 디바이스에서 생성한 데이터를 분석하고 이상 항목을 발견합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Node.js 애플리케이션](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* Spark 서비스 및 {{site.data.keyword.cos_full_notm}}를 사용하는 [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* 디바이스가 MQTT 프로토콜을 사용하여 센서 데이터를 {{site.data.keyword.iot_full}}에 전송합니다. 
* 히스토리 데이터를 {{site.data.keyword.cloudant_short_notm}} 데이터베이스로 내보냅니다. 
* {{site.data.keyword.DSX_short}}가 이 데이터베이스에서 데이터를 가져옵니다. 
* 데이터가 Jupyter 노트북을 통해 분석되고 시각화됩니다. 

## 시작하기 전에
{: #prereqs}

[{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - 스크립트를 실행하여 IBM Cloud CLI 및 필수 플러그인을 설치하십시오. 

## IoT 플랫폼 작성
{: #iot_starter}

먼저 디바이스를 관리하고, 안전하게 연결하여 **데이터를 수집**하고, 시각화 및 애플리케이션을 위해 히스토리 데이터를 사용할 수 있는 상태로 만드는 허브인 Internet of Things Platform 서비스를 작성합니다. 

1. [**{{site.data.keyword.Bluemix_notm}} 카탈로그**](https://{DomainName}/catalog/)로 이동하여 **Internet of Things** 섹션에 있는 [**Internet of Things Platform**](https://{DomainName}/catalog/services/internet-of-things-platform)을 선택하십시오. 
2. 서비스 이름으로 `IoT demo hub`를 입력하고 **작성**을 클릭한 후 대시보드를 **실행**하십시오. 
3. 사이드 메뉴에서 **보안 > 연결 보안**을 선택하고 **기본 규칙** > **보안 레벨**에서 **선택적 TLS**를 선택한 후 **저장**을 클릭하십시오. 
4. 사이드 메뉴에서 **디바이스** > **디바이스 유형**을 선택한 후 **+ 디바이스 유형 추가**를 선택하십시오. 
5. **이름**으로 `simulator`를 입력하고 **다음** 및 **완료**를 클릭하십시오. 
6. 그 다음 **디바이스 등록**을 클릭하십시오. 
7. **기존 디바이스 유형 선택**에 대해 `simulator`를 선택한 후 **디바이스 ID**에 대해 `phone`을 입력하십시오. 
8. **디바이스 보안**(보안 탭에 있음) 화면이 표시될 때까지 **다음**을 클릭하십시오. 
9. **인증 토큰**의 값(예: `myauthtoken`)을 입력하고 **다음**을 클릭하십시오. 
10. **완료**를 클릭하면 사용자의 연결 정보가 표시됩니다. 이 탭을 열린 상태로 두십시오. 

IoT Platform이 데이터 수신을 시작하도록 구성되었습니다. 디바이스는 디바이스 유형, ID 및 토큰을 지정하여 IoT Platform에 데이터를 전송해야 합니다. 

## 디바이스 시뮬레이터 작성
{: #confignodered}
그 다음에는 Node.js 웹 애플리케이션을 배치하고 이를 자신의 휴대전화에서 방문합니다. 이렇게 하면 IoT Platform에 연결되어 디바이스 가속도계 및 방향 데이터가 여기에 전송됩니다. 

1. GitHub 저장소를 복제하십시오. 
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. 코드를 원하는 IDE에서 열고 **manifest.yml** 파일의 `name` 및 `host` 값을 고유 값으로 변경하십시오. 
3. 애플리케이션을 {{site.data.keyword.Bluemix_notm}}에 푸시하십시오. 
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. 몇 분 안에 애플리케이션이 배치되고 `<UNIQUE_NAME>.mybluemix.net`과 같은 URL을 볼 수 있습니다. 
5. 휴대전화에서 브라우저를 사용하여 이 URL에 방문하십시오. 
6. IoT 대시보드 탭의 **디바이스 인증 정보**에 있는 연결 정보를 입력하고 **연결**을 클릭하십시오. 
7. 휴대전화가 데이터 전송을 시작합니다. **IBM {{site.data.keyword.iot_short_notm}} 탭**으로 돌아와 **최근 이벤트** 섹션에 새 항목이 있는지 확인하십시오.
  ![](images/solution16/recent_events_with_phone.png)

## IBM {{site.data.keyword.iot_short_notm}}에서 실시간 데이터 표시
{: #createcards}
그 다음에는 대시보드에서 디바이스 데이터를 표시하는 데 필요한 보드 및 카드를 작성합니다. 보드 및 카드에 대한 자세한 정보는 [보드 및 카드를 사용하여 실시간 데이터 시각화](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html)를 참조하십시오. 

### 보드 작성
{: #createboard}

1. **IBM {{site.data.keyword.iot_short_notm}} 대시보드**를 여십시오. 
2. 왼쪽 메뉴에서 **보드**를 선택한 후 **새 보드 작성**을 클릭하십시오. 
3. 보드의 이름(예: `Simulators`)을 입력하고 **다음**을 클릭한 후 **제출**을 클릭하십시오.   
4. 방금 작성한 보드를 선택하여 여십시오. 

### 디바이스 데이터 표시
{: #cardtemp}
1. **새 카드 추가**를 클릭한 후 디바이스 섹션에 있는 **선형 차트** 카드 유형을 선택하십시오. 
2. 목록에서 디바이스를 선택한 후 **다음**을 클릭하십시오. 
3. **새 데이터 세트 연결**을 클릭하십시오. 
4. 값 카드 작성 페이지에서 다음 값을 선택하거나 입력하고 **다음**을 클릭하십시오. 
   - 이벤트: sensorData
   - 특성: ob
   - 이름: OrientationBeta
   - 유형: Float
   - 최소: -180
   - 최대: 180
5. 카드 미리보기 페이지에서 선형 차트 크기에 대해 **L**을 선택하고 **다음** > **제출**을 클릭하십시오. 
6. 카드가 대시보드에 표시되며 실시간 온도 데이터의 선형 차트를 포함합니다. 
7. 휴대전화 브라우저를 사용해 시뮬레이터를 다시 실행하여 휴대전화를 앞뒤로 천천히 기울여 보십시오. 
8. **IBM {{site.data.keyword.iot_short_notm}} 탭**으로 돌아오면 차트가 업데이트되는 것을 볼 수 있습니다.
   ![](images/solution16/board.png)

## {{site.data.keyword.cloudant_short_notm}}에 히스토리 데이터 저장
1. [**{{site.data.keyword.Bluemix_notm}} 카탈로그**](https://{DomainName}/catalog/)로 이동하여 이름이 `iot-db`인 새 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)를 작성하십시오. 
2. **연결**에서 다음 작업을 수행하십시오. 
   1. **연결을 작성하십시오**. 
   1. {{site.data.keyword.cloudant_short_notm}} 서비스에 대한 별명을 작성할 Cloud Foundry 위치, 조직 및 영역을 선택하십시오. 
   1. **연결 위치** 테이블에 있는 영역 이름을 펼치고 **iot demo hub** 옆에 있는 **연결** 단추를 사용하여 해당 영역에 {{site.data.keyword.cloudant_short_notm}} 서비스에 대한 별명을 작성하십시오.  
   1. 앱을 연결하고 다시 스테이징하십시오. 
3. **IBM {{site.data.keyword.iot_short_notm}} 대시보드**를 여십시오. 
4. 왼쪽 메뉴에서 **확장**을 선택한 후 **히스토리 데이터 스토리지**에서 **설정**을 클릭하십시오. 
5. `iot-db` {{site.data.keyword.cloudant_short_notm}} 데이터베이스를 선택하십시오. 
6. **데이터베이스 이름**으로 `devicedata`를 입력하고 **완료**를 클릭하십시오. 
7. 권한 부여를 요구하는 새 창이 로드됩니다. 이 창이 표시되지 않는 경우에는 팝업 차단을 사용 안함으로 설정하고 페이지를 새로 고치십시오. 

이제 디바이스 데이터가 {{site.data.keyword.cloudant_short_notm}}에 저장됩니다. 몇 분 뒤에 {{site.data.keyword.cloudant_short_notm}} 대시보드를 실행하여 데이터를 보십시오. 

![](images/solution16/cloudant.png)

## 기계 학습을 사용하여 이상 항목 발견
{: #data_experience}

이 절에서는 IBM {{site.data.keyword.DSX_short}} 서비스에서 사용할 수 있는 Jupyter Notebook을 사용하여 히스토리 모바일 데이터를 로드하고 z 점수를 사용하여 이상 항목을 발견합니다. *z 점수*는 특정 요소가 평균과 얼마나 떨어져있는지 나타내는 표준 점수입니다. 

### 새 프로젝트 작성
1. [**{{site.data.keyword.Bluemix_notm}} 카탈로그**](https://{DomainName}/catalog/)로 이동하여 **AI** 섹션에서 [**{{site.data.keyword.DSX_short}}**](https://{DomainName}/catalog/services/data-science-experience)를 선택하십시오. 
2. 서비스를 **작성**하고 **시작하기**를 클릭하여 해당 대시보드를 실행하십시오. 
3. 프로젝트 작성 > **데이터 사이언스**를 선택하고 **프로젝트 작성**을 클릭한 후 프로젝트 **이름**으로 `Detect Anomaly`를 입력하십시오. 
4. 기밀 데이터가 없으므로 **협업자가 될 수 있는 사용자 제한** 선택란을 선택하지 않은 상태로 두십시오. 
5. **스토리지 정의**에서 **추가**를 클릭하고 기존 **Cloud Object Storage** 서비스를 선택하거나 새 서비스를 작성하십시오(**Lite** 플랜 > 작성 선택). **새로 고치기**를 눌러 작성된 서비스를 보십시오. 
6. **작성**을 클릭하십시오. 새 프로젝트가 열리고 리소스 추가를 시작할 수 있습니다. 

### 데이터를 위해 {{site.data.keyword.cloudant_short_notm}}에 연결

1. **자산** > **+ 프로젝트에 추가** > **연결**을 클릭하십시오.   
2. 디바이스 데이터가 저장된 **iot-db** {{site.data.keyword.cloudant_short_notm}}를 선택하십시오. 
3. **인증 정보**를 교차 확인한 후 **작성**을 클릭하십시오. 

### Apache Spark 서비스 작성

1. 최상위 탐색줄에서 **서비스**를 클릭하고 컴퓨팅 서비스를 클릭하십시오. 
2. **서비스 추가**를 클릭하십시오. 
   1. **Apache Spark**에서 **추가**를 클릭하십시오. 
   1. **Lite** 플랜을 선택하십시오. 
   1. **작성**을 클릭하십시오. 
3. 조직 및 영역을 선택하고, 원하는 경우에는 서비스 이름을 변경한 후 **확인**하십시오. 
1. **프로젝트**를 통해 `Detect Anomaly` 프로젝트로 이동하십시오. 
1. **설정**에서 **연관된 서비스**로 스크롤하십시오. 
1. **서비스 추가**를 클릭하고 **Spark**를 선택하십시오. 
1. 이전에 작성한 **Apache Spark** 인스턴스를 선택하십시오. 

### Jupyter(ipynb) 노트북 작성
1. **+ 프로젝트에 추가**를 클릭하고 새 **노트북**을 추가하십시오. 
2. `Anomaly-detection-notebook`을 **이름**으로 입력하십시오. 
3. `https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb`를 **노트북 URL**에 입력하십시오. 
4. 이전에 연관된 **Apache Spark** 서비스를 런타임으로 선택하십시오. 
5. **노트북**을 작성하십시오. `Python 3.5 with Spark 2.1`을 커널로 설정하십시오. 메타데이터 및 코드를 사용하여 노트북이 작성되었는지 확인하십시오.
   ![Watson Studio Jupyter 노트북](images/solution16/jupyter_notebook_watson_studio.png)
   업데이트하려면 **커널** > 커널 변경을 선택하십시오. 노트북을 **신뢰**하려면 **파일** > 노트북 신뢰를 선택하십시오.
   {:tip}

### 노트북을 실행하고 이상 항목 발견   
1. `!pip install --upgrade pixiedust,`로 시작하는 셀을 선택한 후 **실행**을 클릭하거나 **Ctrl+Enter**를 눌러 코드를 실행하십시오. 
2. 설치가 완료되면 **커널 다시 시작** 아이콘을 클릭하여 Spark 커널을 다시 시작하십시오. 
3. 다음 코드 셀에서 다음 단계를 완료하여 {{site.data.keyword.cloudant_short_notm}} 인증 정보를 이 셀로 가져오십시오. 
   * ![](images/solution16/data_icon.png)를 클릭하십시오. 
   * **연결** 탭을 선택하십시오. 
   * **코드에 삽입**을 클릭하십시오. {{site.data.keyword.cloudant_short_notm}} 인증 정보를 포함하는 _credentials_1_이라는 사전이 작성됩니다. 이 사전의 이름이 _credentials_1_로 지정되지 않은 경우에는 사전 이름을 `credentials_1`로 바꾸십시오. 나머지 셀에서는 `credentials_1`이 사용됩니다. 
4. 데이터베이스 이름이 있는 셀(`dbName`)에서 데이터 소스인 {{site.data.keyword.cloudant_short_notm}} 데이터베이스의 이름(예: *iotp_yourWatsonIoTProgId_DBName_Year-month-day*)을 입력하십시오. 다른 디바이스의 데이터를 시각화하려면 `deviceId` 및 `deviceType`의 값을 적절히 변경하십시오.
   이전에 작성한 **iot-db** {{site.data.keyword.cloudant_short_notm}} 인스턴스로 이동해 대시보드를 실행하여 정확한 데이터베이스를 찾을 수 있습니다.
   {:tip}
5. 노트북을 저장하고 각 코드 셀을 하나씩 실행하거나 모두 실행(**셀** > 모두 실행)하면 노트북 끝에 디바이스 이동 데이터(oa,ob 및 og)의 이상 항목을 볼 수 있습니다.
   시간 간격을 원하는 시간으로 변경할 수 있습니다. `start` 및 `end` 값을 찾으십시오.
   {:tip}
   ![DSX Jupyter 노트북](images/solution16/anomaly_detection_watson_studio.png)
6. 이 절에서 이상 항목 발견 외에 기억해야 하는 중요한 결과 또는 요점은 다음과 같습니다. 
    * 시각화를 위해 데이터를 준비하는 과정에서의 Spark 사용
    * 데이터 시각화를 위한 Pandas 사용
    * 디바이스 데이터에 대한 막대형 차트, 히스토그램
    * 상관 행렬을 통한 두 센서 간의 상관
    * Pandas plot 함수로 생성된 각 디바이스 센서의 상자 도표
    * 커널 밀도 추정(KDE)을 통한 밀도 도표
    ![](images/solution16/density_plots_sensor_data.png)

## 리소스 제거
{:removeresources}

1. [리소스 목록](https://{DomainName}/resources/)으로 이동하여 서비스를 작성한 위치, 조직 및 영역을 선택하십시오. **Cloud Foundry 앱**에서, 위에서 작성한 Node.js 앱을 삭제하십시오. 
2. **서비스**에서 이 튜토리얼을 위해 작성한 각 Internet of Things Platform, Apache Spark, {{site.data.keyword.cloudant_short_notm}} 및 {{site.data.keyword.cos_full_notm}} 서비스를 삭제하십시오. 

## 관련 컨텐츠
{:related}

* [예측 기계 학습 모델 빌드, 배치, 테스트 및 재훈련](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* [IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html)의 개요
* 이상 항목 발견 [Jupyter 노트북](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb)
* [z 점수 이해하기](https://en.wikipedia.org/wiki/Standard_score)
* Developing cognitive IoT solutions for anomaly detection by using deep learning - [5개의 연속 게시물](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/)
