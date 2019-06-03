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

# 通过流式分析和 SQL 使用大数据日志
{: #big-data-log-analytics}

在本教程中，您将构建一个日志分析管道，用于收集、存储和分析日志记录，以满足法规要求或帮助发现信息。此解决方案将利用 {{site.data.keyword.cloud_notm}} 中提供的多个服务：{{site.data.keyword.messagehub}}、{{site.data.keyword.cos_short}}、SQL Query 和 {{site.data.keyword.streaminganalyticsshort}}。程序将通过模拟 Web 服务器日志消息从静态文件传输到 {{site.data.keyword.messagehub}} 的过程来为您提供帮助。

通过 {{site.data.keyword.messagehub}}，管道可以进行缩放，以接收来自各种生产者的数百万条日志记录。通过应用 {{site.data.keyword.streaminganalyticsshort}}，可以实时检查日志数据以集成业务流程。使用 {{site.data.keyword.cos_short}} 还可以轻松地将日志消息重定向到长期存储器，供开发者、支持人员和审计员通过 SQL Query 直接使用其中的数据。

虽然本教程专注于说明日志分析，但也适用于其他场景：存储器受限的物联网设备可以采用类似方式将消息流式传输到 {{site.data.keyword.cos_short}} ，或者市场营销专员可以使用 SQL Query 来对不同数字资产中的客户事件进行分段并分析。
{:shortdesc}

## 目标

{: #objectives}

* 了解 Apache Kafka 发布/预订消息传递
* 存储日志数据，以满足审计和合规性要求
* 监视日志，以创建异常处理过程
* 对日志数据执行取证和统计学分析

## 使用的服务

{: #services}

本教程使用以下运行时和服务：

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构

{: #architecture}

<p style="text-align: center;">

  ![体系结构](images/solution31/Architecture.png)
</p>

1. 应用程序生成日志事件并发送到 {{site.data.keyword.messagehub}}
2. {{site.data.keyword.streaminganalyticsshort}} 拦截并分析日志事件
3. 日志事件附加到位于 {{site.data.keyword.cos_short}} 中的 CSV 文件
4. 审计员或支持人员发布 SQL 作业
5. 通过 SQL Query 对 {{site.data.keyword.cos_short}} 中的日志文件执行查询
6. 结果集存储在 {{site.data.keyword.cos_short}} 中，并传递给审计员和支持人员

## 开始之前

{: #prereqs}

* [安装 Git](https://git-scm.com/)
* [安装 {{site.data.keyword.Bluemix_notm}} CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* [安装 Node.js](https://nodejs.org)
* [下载 Kafka 0.10.2.X 客户机](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)

## 创建服务

{: #setup}

在此部分中，您将创建对应用程序生成的日志事件执行分析所需的服务。

此部分使用命令行来创建服务实例。或者，您可以使用提供的链接通过目录中的服务页面来执行相同操作。
{: tip}

1. 通过命令行登录到 {{site.data.keyword.cloud_notm}}，并将您的 Cloud Foundry 帐户设定为目标。请参阅 [CLI 入门](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)。
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. 创建 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) 的轻量实例。
    ```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. 创建 [SQL Query](https://{DomainName}/catalog/services/sql-query) 的轻量实例。
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. 创建 [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams) 的标准实例。
    ```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## 创建消息传递主题和 {{site.data.keyword.cos_short}} 存储区

{: #topics}

首先创建 {{site.data.keyword.messagehub}} 主题和 {{site.data.keyword.cos_short}} 存储区。主题定义应用程序在发布/预订消息传递系统中传递消息的位置。系统收到并处理消息后，会将消息存储在位于 {{site.data.keyword.cos_short}} 存储区的文件中。

1. 在浏览器中，访问[资源](https://{DomainName}/resources?search=log-analysis)中的 `log-analysis-hub` 服务实例。
2. 单击 **+** 按钮以创建主题。
3. 输入**主题名称** `webserver`，然后单击**创建主题**按钮。
4. 单击**服务凭证**，再单击**新建凭证**按钮。
5. 在生成的对话框中，输入 `webserver-flow` 作为**名称**，然后单击**添加**按钮。
6. 单击**查看凭证**，然后将其中的信息复制到安全位置。在下一部分中将用到这些信息。
7. 返回到[资源列表](https://{DomainName}/resources?search=log-analysis)，选择 `log-analysis-cos` 服务实例。
8. 单击**创建存储区**。
    * 输入存储区的唯一**名称**。
    * 对于**弹性**，选择**跨区域**。
    * 选择 **us-geo** 作为**位置**。
    * 单击**创建存储区**。

## 创建 Streams 流源

{: #streamsflow}

在此部分中，您将开始配置接收日志消息的 Streams 流。{{site.data.keyword.streaminganalyticsshort}} 服务基于 {{site.data.keyword.streamsshort}} 技术，该技术每秒可以分析数百万个事件，支持亚毫秒级的响应时间和即时决策。

1. 在浏览器中，访问 [Watson Data Platform](https://dataplatform.ibm.com)。
2. 选择**新建项目**按钮或磁贴，选择**基本**磁贴，然后单击**确定**。
    * 输入**名称** `webserver-logs`。
    * **存储**选项应该设置为 `log-analysis-cos`。如果未设置为此服务实例，请选择此服务实例。
    * 单击**创建**按钮。
3. 在生成的页面上，选择**设置**选项卡，然后选中**工具**中的 **Streams Designer**。通过单击**保存**按钮以完成操作。
4. 单击**添加到项目**按钮，然后单击顶部导航栏中的 **Streams 流**。
    * 单击**将 IBM Streaming Analytics 实例与基于容器的套餐相关联**。
    * 通过选择**轻量**单选按钮，然后单击**创建**，以创建新的 {{site.data.keyword.streaminganalyticsshort}} 实例。不要选择轻量 VM。
    * 对于**服务名称**，提供 `log-analysis-sa`，然后单击**确认**。
    * 对于 Streams 流**名称**，输入 `webserver-flow`。
    * 通过单击**创建**以完成操作。
5. 在生成的页面上，选择 **{{site.data.keyword.messagehub}}** 磁贴。
    * 单击**添加连接**，然后选择 `log-analysis-hub` {{site.data.keyword.messagehub}} 实例。如果未看到该实例列出，请选择 **IBM {{site.data.keyword.messagehub}}** 选项。手动输入从先前部分的**服务凭证**中获取的**连接详细信息**。将连接的**名称**指定为 `webserver-flow`。
    * 单击**创建**以创建连接。
    * 从**主题**下拉列表中，选择 `webserver`。
    * 从**初始偏移量**下拉列表中，选择**从第一个新消息开始**。
    * 单击**继续**。
6. 使**预览数据**页面保持打开；在下一部分中将用到此页面。

## 将 Kafka 控制台工具与 {{site.data.keyword.messagehub}} 配合使用

{: #kafkatools}

`webserver-flow` 当前处于空闲状态，正在等待消息。在此部分中，您将配置 Kafka 控制台工具以使用 {{site.data.keyword.messagehub}}。通过 Kafka 控制台工具，可以从终端生成任意消息，并将这些消息发送到 {{site.data.keyword.messagehub}}，这将触发 `webserver-flow`。

1. 下载并解压缩 [Kafka 0.10.2.X 客户机](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)。
2. 将目录切换至 `bin`，然后创建包含以下内容的名为 `message-hub.config` 的文本文件。
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. 将 `message-hub.config` 文件中的 `USER` 和 `PASSWORD` 替换为先前部分的**服务凭证**中显示的 `user` 和 `password` 值。保存 `message-hub.config`。
4. 从 `bin` 目录中，运行以下命令。将 `KAFKA_BROKERS_SASL` 替换为**服务凭证**中显示的 `kafka_brokers_sasl` 值。下面提供了示例。
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
5. Kafka 控制台工具正在等待输入。将下面的日志消息复制并粘贴到终端中。按 `Enter` 键将日志消息发送到 {{site.data.keyword.messagehub}}。请注意，已发送的消息还会显示在 `webserver-flow` **预览数据**页面上。
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![“预览”页面](images/solution31/preview_data.png)

## 创建 Streams 流目标

{: #streamstarget}

在此部分中，您将通过定义目标来完成 Streams 流配置。目标将用于在早先创建的 {{site.data.keyword.cos_short}} 存储区中存储传入日志消息。将传入日志消息存储并附加到文件的过程将由 {{site.data.keyword.streaminganalyticsshort}} 自动完成。

1. 在 `webserver-flow` **预览**页面上，单击**继续**按钮。
2. 选择 **{{site.data.keyword.cos_full_notm}}** 磁贴作为目标。
    * 单击**添加连接**，然后选择 `log-analysis-cos`。
    * 单击**创建**。
    * 输入**文件路径** `/YOUR_BUCKET_NAME/http-logs_%TIME.csv`。将 `YOUR_BUCKET_NAME` 替换为第一部分中使用的相应名称。
    * 在**格式**下拉列表中，选择 **csv**。
    * 选中**列标题行**复选框。
    * 在**文件创建策略**下拉列表中，选择**文件大小**。
    * 通过在**文件大小 (KB)** 文本框中输入 `102400`，将限制设置为 100 MB。
    * 单击**继续**。
3. 单击**保存**。
4. 单击“播放”按钮 **>** 以**启动 Streams 流**。
5. 在启动流后，再次从 Kafka 控制台工具发送多条日志消息。可以通过在 Streams Designer 中查看 `webserver-flow` 来监控消息到达。
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. 返回到 {{site.data.keyword.cos_short}} 中的存储区。在足够的消息进入流或重新启动该流后，将存在新的 `log.csv` 文件。

![webserver-flow](images/solution31/flow.png)

## 向 Streams 流添加条件行为

{: #streamslogic}

到目前为止，Streams 流是一个简单的管道，用于将消息从 {{site.data.keyword.messagehub}} 移至 {{site.data.keyword.cos_short}}。更有可能的情况是，团队希望实时了解关注的事件。例如，发生 HTTP 500（应用程序错误）事件时，发出的警报对各个团队都有利。在此部分中，您将向流添加条件逻辑，以识别 HTTP 200（正常）和非 HTTP 200 代码。

1. 使用“画笔”按钮来**编辑 Streams 流**。
2. 创建一个过滤器节点来处理 HTTP 200 响应。
    * 在**节点**选用板中，将该**过滤器**节点从**处理和分析**拖至画布。
    * 在“名称”文本框中，输入`正常`，当前此框中包含的文字是`过滤器`。
    * 在**条件表达式**文本区域中，输入以下语句。
      ```sh
      responseCode == 200
      ```
      {: pre}
    * 使用鼠标从 **{{site.data.keyword.messagehub}}** 节点的输出（右侧）到**正常**节点的输入（左侧）之间绘制一条直线。
    * 在**节点**选用板中，将**目标**下找到的**调试**节点拖至画布。
    * 通过在**调试**节点与**正常**节点之间绘制一条直线，以连接这两个节点。
3. 重复此过程以创建使用相同节点和以下条件语句的`不正常`过滤器。
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. 单击“播放”按钮以**保存并运行 Streams 流**。
5. 如果系统提示，请单击**运行新版本**的链接。

![流设计器](images/solution31/flow_design.png)

## 增加消息负载

{: #streamsload}

为了在 Streams 流中查看条件处理，您将增加发送到 {{site.data.keyword.messagehub}} 的消息量。提供的 Node.js 程序将模拟根据流至 Web 服务器的流量发送到 {{site.data.keyword.messagehub}} 的实际消息流。为了演示 {{site.data.keyword.messagehub}} 和 {{site.data.keyword.streaminganalyticsshort}} 的可缩放性，您将增加日志消息的吞吐量。

此部分使用的是 [node-rdkafka](https://www.npmjs.com/package/node-rdkafka)。如果模拟器安装失败，请参阅 npmjs 页面以获取故障诊断指示信息。如果问题仍然存在，您可以跳至下一部分并手动上传数据。

1. 下载并解压缩来自 NASA 的 [Jul 01 to Jul 31, ASCII format, 20.7 MB gzip compressed](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz) 日志文件。
2. 从 [GitHub 上的 IBM Cloud](https://github.com/IBM-Cloud/kafka-log-simulator) 克隆日志模拟器并进行安装。
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. 切换至模拟器的目录，然后运行以下命令来设置模拟器，并生成日志事件消息。将 `LOGFILE` 替换为下载的文件。将 `BROKERLIST` 和 `APIKEY` 替换为早先使用的相应**服务凭证**。下面提供了示例。
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
4. 在浏览器中，在模拟器开始生成消息后返回到 `webserver-flow`。
5. 在所需数量的消息通过条件分支后，使用 `Ctrl+C` 键停止模拟器。
6. 通过增大或减小 `--rate` 值来对 {{site.data.keyword.messagehub}} 缩放进行试验。

模拟器将根据 Web 服务器日志中的耗用时间来延迟发送下一条消息。设置 `--rate 1` 将实时发送事件。设置 `--rate 100` 表示对于 Web 服务器日志中的每 1 秒耗用时间，将在消息之间使用 10 毫秒延迟。
{: tip}

![流负载设置为 10](images/solution31/flow_load_10.png)

## 使用 SQL Query 调查日志数据

{: #sqlquery}

根据模拟器发送的消息数，{{site.data.keyword.cos_short}} 上日志文件的文件大小明确增加了。现在，您将通过组合使用 SQL Query 与日志文件，充当调查员来回答审计或合规性问题。使用 SQL Query 的优点是可以直接访问日志文件 - 不需要额外的变换或数据库服务器。

如果您不想等待模拟器发送所有日志消息，请将[完整的 CSV 文件](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp)上传到 {{site.data.keyword.cos_short}} 以立即开始。
{: tip}

1. 访问[资源列表](https://{DomainName}/resources?search=log-analysis)中的 `log-analysis-sql` 服务实例。选择**打开 UI** 以启动 SQL Query。
2. 在**在此输入 SQL...** 文本区域中，输入以下 SQL。
    ```sql
    -- 自 1995 年 7 月以来，NASA 最热门的十大 Web 页面是什么？
    -- 哪项任务可能非常重要？
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. 在日志文件中检索对象 SQL URL。
    * 从[资源列表](https://{DomainName}/resources?search=log-analysis)中，选择 `log-analysis-cos` 服务实例。
    * 选择先前创建的存储区。
    * 单击 `http-logs_TIME.csv` 文件上的溢出菜单，然后选择**对象 SQL URL**。
    * 将该 URL **复制**到剪贴板。
4. 使用“对象 SQL URL”更新 `FROM` 子句，然后单击**运行**。
5. 结果可以在**结果**选项卡上查看。虽然应该会显示某些页面（如肯尼迪航天中心主页），但有一项任务在当前十分热门。
6. 选择**查询详细信息**选项卡来查看其他信息，例如结果在 {{site.data.keyword.cos_short}} 上的存储位置。
7. 通过在**在此输入 SQL...** 文本区域中分别添加以下问题和回答对，以尝试这些对。
    ```sql
    -- 排名靠前的五大浏览者是谁？
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- 根据应用程序故障，哪个浏览者有可疑活动？
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- 哪些请求向用户显示“找不到页面”错误？
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- 最大的 10 个文件是什么？
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- 按小时计算的总流量分布情况如何？
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- 为什么先前的结果返回的小时值为空？
    -- 提示：请查找格式不正确的主机名。
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

FROM 子句不限于单个文件。使用 `cos://us-geo/YOUR_BUCKET_NAME/` 可对存储区中的所有文件运行 SQL 查询。
{: tip}

## 扩展教程

{: #expand}

祝贺您，您已使用 {{site.data.keyword.cloud_notm}} 构建了日志分析管道。下面是增强解决方案的其他建议。

* 在 Streams Designer 中使用其他目标在 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 中存储数据或在 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) 中执行代码
* 遵循[使用 Object Storage 构建数据湖](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage)教程，以添加用于记录数据的仪表板
* 使用 [{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect) 将其他系统与 {{site.data.keyword.messagehub}} 相集成

## 除去服务

{: #removal}

在[资源列表](https://{DomainName}/resources?search=log-analysis)中，使用溢出菜单中的**删除**或**删除服务**菜单项来除去以下服务实例。

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## 相关内容

{:related}

* [Apache Kafka](https://kafka.apache.org/)
