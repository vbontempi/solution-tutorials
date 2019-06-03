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

# 스트리밍 분석 및 SQL을 사용하는 빅데이터 로그
{: #big-data-log-analytics}

이 튜토리얼에서는 규제 요구사항을 지원하고 정보 발견을 돕기 위해 로그 레코드를 수집하고, 저장하고 분석하도록 디자인된 로그 분석 파이프라인을 빌드합니다. 이 솔루션은 {{site.data.keyword.cloud_notm}}에서 사용 가능한 몇 가지 서비스({{site.data.keyword.messagehub}}, {{site.data.keyword.cos_short}}, SQL Query 및 {{site.data.keyword.streaminganalyticsshort}})를 이용합니다. 정적 파일에서 {{site.data.keyword.messagehub}}로의 웹 서버 로그 메시지 전송을 시뮬레이션하여 사용자를 돕는 프로그램이 포함되어 있습니다. 

{{site.data.keyword.messagehub}}를 사용하면 다양한 작성자가 작성하는 수백만 개의 로그 레코드를 수신하기 위해 파이프라인이 스케일링됩니다. {{site.data.keyword.streaminganalyticsshort}}를 적용하면 로그 데이터를 실시간으로 검사하여 비즈니스 프로세스를 통합할 수 있습니다. 개발자, 지원 담당자 및 감사자가 SQL Query를 사용하여 데이터에 대해 직접 작업할 수 있는 {{site.data.keyword.cos_short}}를 사용하여 로그 메시지를 손쉽게 장기 스토리지로 경로 재지정할 수도 있습니다. 

이 튜토리얼은 로그 분석에 초점을 맞추고 있지만, 이를 다른 시나리오에 적용할 수도 있습니다. 스토리지가 제한된 IoT 디바이스 또한 {{site.data.keyword.cos_short}}에 메시지를 스트리밍할 수 있으며, 마케팅 전문가는 SQL Query를 사용하여 여러 디지털 상점의 고객 이벤트를 구분하고 분석할 수 있습니다.
{:shortdesc}

## 목표

{: #objectives}

* Apache Kafka 발행/구독 메시징 이해
* 감사 및 정부 규제 준수를 위해 로그 데이터 저장
* 예외 처리 프로세스 작성을 위해 로그 모니터
* 로그 데이터에 대한 포렌식 및 통계 분석 수행

## 사용되는 서비스

{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처

{: #architecture}

<p style="text-align: center;">

  ![아키텍처](images/solution31/Architecture.png)
</p>

1. 애플리케이션이 {{site.data.keyword.messagehub}}에 로그 이벤트를 생성함
2. 로그 이벤트가 {{site.data.keyword.streaminganalyticsshort}}에 의해 인터셉트되어 분석됨
3. 로그 이벤트가 {{site.data.keyword.cos_short}}에 있는 CSV 파일에 추가됨
4. 감사자 또는 지원 담당자가 SQL 작업을 실행함
5. SQL Query가 {{site.data.keyword.cos_short}}의 로그 파일에 대해 실행됨
6. 결과 세트가 {{site.data.keyword.cos_short}}에 저장되고 감사자 및 지원 담당자에게 전달됨

## 시작하기 전에

{: #prereqs}

* [Git 설치](https://git-scm.com/)
* [{{site.data.keyword.Bluemix_notm}} CLI 설치](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* [Node.js 설치](https://nodejs.org)
* [Kafka 0.10.2.X 클라이언트 다운로드](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)

## 서비스 작성

{: #setup}

이 절에서는 애플리케이션에 의해 생성된 로그 이벤트를 분석하는 데 필요한 서비스를 작성합니다. 

이 절에서는 명령행을 사용하여 서비스 인스턴스를 작성합니다. 또는 제공된 링크를 사용하여 카탈로그의 서비스 페이지에서 동일한 작업을 수행할 수 있습니다.
{: tip}

1. 명령행을 통해 {{site.data.keyword.cloud_notm}}에 로그인한 후 자신의 Cloud Foundry 계정을 대상으로 설정하십시오. [CLI 시작하기](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)를 참조하십시오.
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)의 Lite 인스턴스를 작성하십시오.
    ```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. [SQL Query](https://{DomainName}/catalog/services/sql-query)의 Lite 인스턴스를 작성하십시오.
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)의 표준 인스턴스를 작성하십시오.
    ```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## 메시징 토픽 및 {{site.data.keyword.cos_short}} 버킷 작성

{: #topics}

{{site.data.keyword.messagehub}} 토픽 및 {{site.data.keyword.cos_short}} 버킷을 작성하여 작업을 시작하십시오. 주제는 발행/구독 메시징 시스템에서 애플리케이션이 메시지를 전달하는 위치를 정의합니다. 메시지는 수신되어 처리된 후 {{site.data.keyword.cos_short}} 버킷에 있는 파일에 저장됩니다. 

1. 브라우저에서 [리소스](https://{DomainName}/resources?search=log-analysis)의 `log-analysis-hub` 서비스 인스턴스에 액세스하십시오. 
2. **+** 단추를 클릭하여 토픽을 작성하십시오. 
3. **토픽 이름**에 `webserver`를 입력하고 **토픽 작성** 단추를 클릭하십시오. 
4. **서비스 인증 정보** 및 **새 인증 정보** 단추를 클릭하십시오. 
5. 결과 대화 상자에서 `webserver-flow`를 **이름**으로 입력하고 **추가** 단추를 클릭하십시오. 
6. **인증 정보 보기**를 클릭하고 안전한 위치에 이 정보를 복사하십시오. 이는 다음 절에서 사용됩니다. 
7. [리소스 목록](https://{DomainName}/resources?search=log-analysis)으로 돌아가 `log-analysis-cos` 서비스 인스턴스를 선택하십시오. 
8. **버킷 작성**을 클릭하십시오. 
    * 버킷의 고유 **이름**을 입력하십시오. 
    * **복원성**에 대해 **지역 간**을 선택하십시오. 
    * **us-geo**를 **위치**로 선택하십시오. 
    * **버킷 작성**을 클릭하십시오. 

## Streams 플로우 소스 작성

{: #streamsflow}

이 절에서는 로그 메시지를 수신하는 Streams 플로우를 구성합니다. {{site.data.keyword.streaminganalyticsshort}} 서비스는 {{site.data.keyword.streamsshort}}에 의해 제공되며, 초당 수백만 개의 이벤트를 분석하여 밀리초에 가까운 응답 시간과 즉각적인 의사결정을 가능하게 합니다. 

1. 브라우저에서 [Watson Data Platform](https://dataplatform.ibm.com)에 액세스하십시오. 
2. **새 프로젝트** 단추 또는 타일을 선택한 후 **기본** 타일을 선택하고 **확인**을 클릭하십시오. 
    * **이름**을 `webserver-logs`로 입력하십시오. 
    * **스토리지** 옵션은 `log-analysis-cos`로 설정해야 합니다. 그렇지 않은 경우에는 서비스 인스턴스를 선택하십시오. 
    * **작성** 단추를 클릭하십시오. 
3. 결과 페이지에서 **설정** 탭을 선택하고 **도구**에서 **Streams Designer**를 선택하십시오. **저장** 단추를 클릭하여 완료하십시오. 
4. 맨 위 탐색줄에서 **프로젝트에 추가** 단추를 클릭한 후 **Streams 플로우**를 클릭하십시오. 
    * **IBM Streaming Analytics 인스턴스를 컨테이너 기반 플랜에 연관**을 클릭하십시오. 
    * **Lite** 단일 선택 단추를 선택하고 **작성**을 클릭하여 새 {{site.data.keyword.streaminganalyticsshort}} 인스턴스를 작성하십시오. Lite VM을 선택하지 마십시오. 
    * **서비스 이름**으로 `log-analysis-sa`를 제공하고 **확인**을 클릭하십시오. 
    * Streams 플로우 **이름**을 `webserver-flow`로 입력하십시오. 
    * **작성**을 클릭하여 완료하십시오. 
5. 결과 페이지에서 **{{site.data.keyword.messagehub}}** 타일을 선택하십시오. 
    * **연결 추가**를 클릭하고 `log-analysis-hub` {{site.data.keyword.messagehub}} 인스턴스를 선택하십시오. 인스턴스가 나열되어 있지 않은 경우에는 **IBM {{site.data.keyword.messagehub}}** 옵션을 선택하십시오. 이전 절에서 **서비스 인증 정보**로부터 얻은 **연결 세부사항**을 수동으로 입력하십시오. 연결의 **이름**을 `webserver-flow`로 지정하십시오. 
    * **작성**을 클릭하여 연결을 작성하십시오. 
    * **주제** 드롭 다운에서 `webserver`를 선택하십시오. 
    * **초기 오프셋** 드롭 다운에서 **첫 번째 새 메시지부터 시작**을 선택하십시오. 
    * **계속**을 클릭하십시오. 
6. **데이터 미리보기** 페이지는 열어두십시오. 이는 다음 절에서 사용됩니다. 

## Kafka 콘솔 도구를 {{site.data.keyword.messagehub}}와 함께 사용

{: #kafkatools}

`webserver-flow`는 현재 유휴 상태이며 메시지를 기다리고 있습니다. 이 절에서는 {{site.data.keyword.messagehub}}와 함께 작동하도록 Kafka 콘솔 도구를 구성합니다. Kafka 콘솔 도구는 터미널에서 임의의 메시지를 생성하고 이를 `webserver-flow`를 트리거하는 {{site.data.keyword.messagehub}}로 전송할 수 있게 해 줍니다. 

1. [Kafka 0.10.2.X 클라이언트](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)를 다운로드하여 압축을 푸십시오. 
2. `bin` 디렉토리로 변경하여 다음 컨텐츠로 `message-hub.config`라는 텍스트 파일을 작성하십시오.
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. `message-hub.config` 파일의 `USER` 및 `PASSWORD`를 이전 절의 **서비스 인증 정보**에서 본 `user` 및 `password`로 대체하십시오. `message-hub.config`를 저장하십시오. 
4. `bin` 디렉토리에서 다음 명령을 실행하십시오. `KAFKA_BROKERS_SASL`은 **서비스 인증 정보**에서 본 `kafka_brokers_sasl`로 대체하십시오. 예가 제공되어 있습니다.
    ```sh
    ./kafka-console-producer.sh --broker-list KAFKA_BROKERS_SASL \
    --producer.config message-hub.config --topic webserver
    ```
    {: pre}
    ```sh
    ./kafka-console-producer.sh --broker-list \
    kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093 \
    --producer.config message-hub.config --topic webserver
    ```
5. Kafka 콘솔 도구가 입력을 기다리고 있습니다. 아래 로그 메시지를 복사하여 터미널에 붙여넣으십시오. `enter`를 눌러 로그 메시지를 {{site.data.keyword.messagehub}}에 전송하십시오. 전송된 메시지가 `webserver-flow` **데이터 미리보기** 페이지에도 표시되는 것을 볼 수 있습니다.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![미리보기 페이지](images/solution31/preview_data.png)

## Streams 플로우 대상 작성

{: #streamstarget}

이 절에서는 대상을 정의하여 Streams 플로우 구성을 완료합니다. 이 대상은 이전에 작성된 {{site.data.keyword.cos_short}} 버킷에 수신 로그 메시지를 저장하는 데 사용됩니다. 수신 로그 메시지를 저장하고 파일에 추가하는 프로세스는 {{site.data.keyword.streaminganalyticsshort}}에서 자동으로 수행합니다. 

1. `webserver-flow` **미리보기 페이지**에서 **계속** 단추를 클릭하십시오. 
2. **{{site.data.keyword.cos_full_notm}}** 타일을 대상으로 선택하십시오. 
    * **연결 추가**를 클릭하고 `log-analysis-cos`를 선택하십시오. 
    * **작성**을 클릭하십시오. 
    * **파일 경로** `/YOUR_BUCKET_NAME/http-logs_%TIME.csv` 를 입력하십시오. `YOUR_BUCKET_NAME`을 첫 번째 절에서 사용된 것으로 대체하십시오. 
    * **형식** 드롭 다운에서 **csv**를 선택하십시오. 
    * **열 헤더 행** 선택란을 선택하십시오. 
    * **파일 작성 정책** 드롭 다운에서 **파일 크기**를 선택하십시오. 
    * **파일 크기(KB)** 텍스트 상자에 `102400`을 입력하여 한계를 100MB로 설정하십시오. 
    * **계속**을 클릭하십시오. 
3. **저장**을 클릭하십시오. 
4. **>** 재생 단추를 클릭하여 **Streams 플로우를 시작**하십시오. 
5. 플로우가 시작되고 나면 Kafka 콘솔 도구에서 다시 여러 로그 메시지를 전송하십시오. Streams Designer에서 `webserver-flow`를 보면 메시지가 도착하는 것을 볼 수 있습니다.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. {{site.data.keyword.cos_short}}의 버킷으로 돌아가십시오. 플로우에 충분한 메시지가 입력되거나 플로우가 다시 시작되고 나면 새 `log.csv` 파일이 작성됩니다. 

![webserver-flow](images/solution31/flow.png)

## Streams 플로우에 조건부 작동 추가

{: #streamslogic}

이제까지의 Streams 플로우는 {{site.data.keyword.messagehub}}에서 {{site.data.keyword.cos_short}}로 메시지를 이동하는 단순한 파이프였습니다. 대부분 팀에서는 원하는 이벤트를 실시간으로 파악하고자 합니다. 예를 들면, 개별 팀은 HTTP 500(애플리케이션 오류) 이벤트가 발생하는 경우 경보가 수신되면 작업에 도움이 됩니다. 이 절에서는 HTTP 200(정상) 코드와 HTTP 200이 아닌 코드를 식별하기 위해 플로우에 조건부 로직을 추가합니다. 

1. 연필 단추를 사용하여 **Streams 플로우를 편집**하십시오. 
2. HTTP 200 응답을 처리하는 필터 노드를 작성하십시오. 
    * **노드** 팔레트에서 **처리 및 분석**의 **필터** 노드를 캔버스로 끌어오십시오. 
    * 현재 단어 `Filter`를 포함하고 있는 이름 텍스트 상자에 `OK`를 입력하십시오. 
    * **조건식** 텍스트 영역에 다음 명령문을 입력하십시오.
      ```sh
      responseCode == 200
      ```
      {: pre}
    * 마우스를 사용하여 **{{site.data.keyword.messagehub}}** 노드의 출력(오른쪽)에서 **OK** 노드의 입력(왼쪽)으로 선을 그리십시오. 
    * **노드** 팔레트에서 **대상**의 **디버그** 노드를 캔버스로 끌어오십시오. 
    * **디버그** 노드와 **OK** 노드 간에 선을 그려 둘을 연결하십시오. 
3. 동일한 노드 및 다음 조건문을 사용해 프로세스를 반복하여 `Not OK` 필터를 작성하십시오.
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. 재생 단추를 클릭하여 **Streams 플로우를 저장하고 실행**하십시오. 
5. 프롬프트가 표시되면 **새 버전 실행**에 대한 링크를 클릭하십시오. 

![플로우 디자이너](images/solution31/flow_design.png)

## 메시지 로드 늘리기

{: #streamsload}

여기서는 Streams 플로우의 조건부 처리를 확인하기 위해 {{site.data.keyword.messagehub}}에 전송되는 메시지 볼륨을 늘립니다. 제공된 Node.js 프로그램은 webserver에 대한 트래픽을 기반으로 {{site.data.keyword.messagehub}}에 대한 실제 메시지 플로우를 시뮬레이션합니다. 여기서는 {{site.data.keyword.messagehub}} 및 {{site.data.keyword.streaminganalyticsshort}}의 확장성을 보여주기 위해 로그 메시지의 처리량을 늘립니다. 

이 절에서는 [node-rdkafka](https://www.npmjs.com/package/node-rdkafka)를 사용합니다. 시뮬레이터 설치가 실패하는 경우에는 npmjs 페이지에 있는 문제점 해결 지시사항을 참조하십시오. 문제점이 지속되면 다음 절로 건너뛰고 수동으로 데이터를 업로드할 수 있습니다. 

1. NASA에서 [Jul 01 to Jul 31, ASCII format, 20.7 MB gzip compressed](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz) 로그 파일을 다운로드하십시오. 
2. [GitHub의 IBM Cloud](https://github.com/IBM-Cloud/kafka-log-simulator)에서 로그 시뮬레이터를 복제하고 설치하십시오.
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. 시뮬레이터의 디렉토리로 변경하고 다음 명령을 실행하여 시뮬레이터를 설정하고 로그 이벤트 메시지를 생성하십시오. `LOGFILE`을 다운로드한 파일로 대체하십시오. `BROKERLIST` 및 `APIKEY`를 이전에 사용한 해당 **서비스 인증 정보**로 대체하십시오. 예가 제공되어 있습니다.
    ```sh
    npm install
    ```
    ```sh
    npm run build
    ```
    ```sh
    node dist/index.js --file LOGFILE --parser httpd --broker-list BROKERLIST \
    --api-key APIKEY --topic webserver --rate 100
    ```
    {: pre}
    ```sh
    node dist/index.js --file /Users/ibmcloud/Downloads/NASA_access_log_Jul95 \
    --parser httpd --broker-list \
    "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093" \
    --api-key Np15YZKN3SCdABUsOpJYtpue6jgJ7CwYgsoCWaPbuyFbdM4R \
    --topic webserver --rate 100
    ```
4. 시뮬레이터가 메시지 생성을 시작하면 브라우저에서 `webserver-flow`로 돌아오십시오. 
5. 원하는 수의 메시지가 조건부 분기를 통과하고 나면 `control+C`를 사용하여 시뮬레이터를 중지하십시오. 
6. `--rate` 값을 늘리거나 줄여 {{site.data.keyword.messagehub}} 스케일링을 시험해 보십시오. 

시뮬레이터는 webserver 로그의 경과 시간에 따라 다음 메시지의 전송을 지연시킵니다. `--rate 1`을 설정하면 실시간으로 이벤트를 전송합니다. `--rate 100` 설정은 webserver 로그의 경과 시간 1초마다 메시지 간에 10ms의 지연시간을 사용함을 의미합니다.
{: tip}

![10으로 설정된 플로우 로드](images/solution31/flow_load_10.png)

## SQL Query를 사용한 로그 파일 조사

{: #sqlquery}

시뮬레이터가 전송하는 메시지 수에 따라 {{site.data.keyword.cos_short}}의 파일 크기는 증가할 수밖에 없습니다. 사용자는 이제 SQL Query와 로그 파일을 결합하여 감사 또는 규제 준수 관련 질문에 답변하는 조사관 역할을 수행합니다. SQL Query를 사용하여 얻을 수 있는 이점은 추가 변환이나 데이터베이스 서버가 필요하지 않고 직접 로그 파일에 액세스할 수 있다는 것입니다. 

시뮬레이터가 모든 로그 메시지를 전송할 때까지 기다리지 않으려는 경우에는 [전체 CSV 파일](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp)을 {{site.data.keyword.cos_short}}에 업로드하여 작업을 즉시 시작할 수 있습니다.
{: tip}

1. [리소스 목록](https://{DomainName}/resources?search=log-analysis)에서 `log-analysis-sql` 서비스 인스턴스에 액세스하십시오. **UI 열기**를 선택하여 SQL Query를 실행하십시오. 
2. 다음 SQL을 **Type SQL here ...** 텍스트 영역에 입력하십시오.
    ```sql
    -- What are the top 10 web pages on NASA from July 1995?
    -- Which mission might be significant?
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. 로그 파일에서 Object SQL URL을 검색하십시오. 
    * [리소스 목록](https://{DomainName}/resources?search=log-analysis)에서 `log-analysis-cos` 서비스 인스턴스를 선택하십시오. 
    * 이전에 작성한 버킷을 선택하십시오. 
    * `http-logs_TIME.csv` 파일의 오버플로우 메뉴를 클릭하고 **Object SQL URL**을 선택하십시오. 
    * URL을 클립보드에 **복사**하십시오. 
4. `FROM` 절을 Object SQL URL로 업데이트하고 **실행**을 클릭하십시오. 
5. **결과** 탭에서 결과를 볼 수 있습니다. Kennedy Space Center 홈 페이지와 같은 일부 페이지에서는 당시에 한 임무가 상당히 인기있었을 것으로 예상할 수 있습니다. 
6. **조회 세부사항** 탭을 선택하여 {{site.data.keyword.cos_short}}에서 결과가 저장된 위치와 같은 추가 정보를 보십시오. 
7. 다음 질문 및 답변 쌍을 개별적으로 **여기에 SQL 입력** 텍스트 영역에 추가하여 이들의 결과를 보십시오.
    ```sql
    -- Who are the top 5 viewers?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- Which viewer has suspicious activity based on application failures?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- Which requests showed a page not found error to the user?
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- What are the top 10 largest files?
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- What is the distribution of total traffic by hour?
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- Why did the previous result return an empty hour?
    -- Hint, find the malformed hostname.
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

FROM 절은 하나의 파일로 제한되지 않습니다. 버킷의 모든 파일에 대해 SQL 조회를 실행하려면 `cos://us-geo/YOUR_BUCKET_NAME/`을 사용하십시오.
{: tip}

## 튜토리얼 확장

{: #expand}

이제까지 {{site.data.keyword.cloud_notm}}와의 로그 분석 파이프라인을 빌드했습니다. 아래 항목은 이 솔루션을 확장하여 적용하는 데 대한 추가 제안사항입니다. 

* Streams Designer에서 추가 대상을 사용하여 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)에 데이터를 저장하거나 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)에서 코드 실행
* [Object Storage를 사용한 데이터 레이크 빌드](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage) 튜토리얼에 따라 로그 데이터에 대시보드 추가
* [{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect)를 사용하여 추가 시스템을 {{site.data.keyword.messagehub}}와 통합

## 서비스 제거

{: #removal}

[리소스 목록](https://{DomainName}/resources?search=log-analysis)에서 오버플로우 메뉴의 **삭제** 또는 **서비스 삭제** 메뉴 항목을 사용하여 다음 서비스 인스턴스를 제거하십시오. 

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## 관련 컨텐츠

{:related}

* [Apache Kafka](https://kafka.apache.org/)
