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

# 使用 Object Storage 构建数据湖
{: #smart-data-lake}

关于术语“数据湖”的定义多种多样，但在本教程的上下文中，数据湖是指一种以本机格式存储数据以供组织使用的方法。为此，您将使用 {{site.data.keyword.cos_short}} 为组织创建数据湖。通过将 {{site.data.keyword.cos_short}} 与 SQL Query 组合使用，数据分析人员可以使用 SQL 来查询数据所在的位置。此外，还将在 Jupyter Notebook 中利用 SQL Query 服务来执行简单分析。完成后，允许非技术用户使用 {{site.data.keyword.dynamdashbemb_notm}} 来发现自己的洞察。

## 目标

- 使用 {{site.data.keyword.cos_short}} 存储原始数据文件
- 使用 SQL Query 直接在 {{site.data.keyword.cos_short}} 中查询数据
- 在 {{site.data.keyword.DSX_full}} 中优化和分析数据
- 使用 {{site.data.keyword.dynamdashbemb_notm}} 在整个组织中共享数据

## 使用的服务

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [SQL Query](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## 体系结构

![体系结构](images/solution29/architecture.png)

1. 原始数据存储在 {{site.data.keyword.cos_short}} 中
2. 使用 SQL Query 来减少、增强或优化数据
3. 在 {{site.data.keyword.DSX}} 中执行数据分析
4. 业务线访问 Web 应用程序
5. 从 {{site.data.keyword.cos_short}} 中拉取优化的数据
6. 使用 {{site.data.keyword.dynamdashbemb_notm}} 构建业务线图表

## 开始之前

- [安装 Git](https://git-scm.com/)
- [安装 {{site.data.keyword.Bluemix_notm}} CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
- [安装 Aspera Connect](http://downloads.asperasoft.com/connect2/)
- [安装 Node.js 和 NPM](https://nodejs.org)

## 创建服务

在此部分中，您将创建构建数据湖所需的服务。

此部分使用命令行来创建服务实例。或者，您可以使用提供的链接通过目录中的服务页面来执行相同操作。
{: tip}

1. 通过命令行登录到 {{site.data.keyword.cloud_notm}}，并将您的 Cloud Foundry 帐户设定为目标。请参阅 [CLI 入门]https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)。
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. 使用 Cloud Foundry 别名创建 [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) 实例。如果已有服务实例，请使用现有服务名称来运行 `service-alias-create` 命令。
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. 创建 [SQL Query](https://{DomainName}/catalog/services/sql-query) 实例。
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. 创建 [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio) 实例。
    ```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. 使用 Cloud Foundry 别名创建 [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded) 实例。
    ```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. 切换到工作目录，并运行以下命令来克隆仪表板应用程序的 [GitHub 存储库](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard)。然后将应用程序推送到 Cloud Foundy 组织。应用程序将使用其 [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml) 文件自动绑定上面的必需服务。
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

    部署后，应用程序将成为公共应用程序，并侦听随机主机名。可以访问[资源列表](https://{DomainName}/resources)页面，在“Cloud Foundry 应用程序”下，选择应用程序并查看 URL，或者运行 `ibmcloud cf app dashboard-nodejs routes` 命令来查看路径。
    {: tip}

7. 通过在浏览器中访问应用程序的公共 URL 来确认该应用程序是否处于活动状态。

![仪表板登录页面](images/solution29/dashboard-start.png)

## 上传数据

在此部分中，您将使用内置 {{site.data.keyword.CHSTSshort}} 将数据上传到 {{site.data.keyword.cos_short}} 存储区。{{site.data.keyword.CHSTSshort}} 可在数据上传到存储区时对数据进行保护，并[可以大大缩短传输时间](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/)。

1. 下载 [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD) CSV 文件。该文件有 81 MB，可能需要几分钟才能下载完。
2. 在浏览器中，访问[资源列表](https://{DomainName}/resources)中的 **data-lake-cos** 服务实例。
3. 创建新存储区来存储数据。
    - 单击**创建存储区**按钮。
    - 从**弹性**下拉列表中，选择**区域**。
    - 从**位置**中，选择 **us-south**。目前，{{site.data.keyword.CHSTSshort}} 仅可用于在 `us-south` 位置创建的存储区。或者，选择其他位置，然后使用下一部分中的**标准**传输类型。
    - 提供存储区的**名称**，然后单击**创建**。如果收到 *AccessDenied* 错误，请尝试使用唯一性更强的存储区名称。
4. 将该 CSV 文件上传到 {{site.data.keyword.cos_short}}。
    - 在存储区中，单击**添加对象**按钮。
    - 选择 **Aspera 高速传输**单选按钮。
    - 单击**添加文件**按钮。这将打开 Aspera 插件，该插件将位于单独的窗口中 - 可能在浏览器窗口后面。
    - 浏览至先前下载的 CSV 文件并选择该文件。

![具有 CSV 文件的存储区](images/solution29/cos-bucket.png)

## 使用数据

在此部分中，您将基于时间和年龄属性，将最初的原始数据集转换为目标群组。这对于有特定的兴趣或者难以驾驭超大数据集的数据湖使用者来说非常有用。

您将使用 SQL Query 通过熟悉的 SQL 语句来处理位于 {{site.data.keyword.cos_short}} 中的数据。SQL Query 内置了对 CSV、JSON 和 Parquet 的支持 - 无需额外的计算服务或抽取、变换和装入。

1. 从[资源列表](https://{DomainName}/resources)访问 **data-lake-sql** SQL Query 服务实例。
2. 选择**打开 UI**。
3. 通过直接对先前上传的 CSV 文件执行 SQL 来创建新数据集。
    - 在**在此输入 SQL...** 文本区域中，输入以下 SQL。
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
    - 将 `FROM` 子句中的 URL 替换为您的存储区名称。
4. **目标**将自动创建 {{site.data.keyword.cos_short}} 存储区来保存结果。将**目标**更改为 `cos://us-south/<your-bucket-name>/results`。
5. 单击**运行**按钮。结果如下所示。
6. 在**查询详细信息**选项卡上，单击紧跟在**结果位置** URL 后面的**启动**图标可查看中间数据集，现在此数据集也存储在 {{site.data.keyword.cos_short}} 上。

![笔记本](images/solution29/sql-query.png)

## 组合使用 Jupyter Notebook 和 SQL Query

在此部分中，您将在 Jupyter Notebook 中使用 SQL Query 客户机。这将在数据分析工具内复用存储在 {{site.data.keyword.cos_short}} 上的数据。此组合还将创建数据集，这些数据集会自动存储在 {{site.data.keyword.cos_short}} 上，然后可以用于 {{site.data.keyword.dynamdashbemb_notm}}。

1. 在 {{site.data.keyword.DSX}} 中创建新的 Jupyter 笔记本。
    - 在浏览器中，打开 [{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true)。
    - 单击**创建项目**磁贴，然后单击**数据研究**。
    - 单击**创建项目**，然后提供**项目名称**。
    - 确保**存储器**设置为 **data-lake-cos**。
    - 单击**创建**。
    - 在生成的项目中，单击**添加到项目**和**笔记本**。
    - 在**空白**选项卡中，输入**笔记本名称**。
    - 使**语言**和**运行时**保留为缺省值；单击**创建笔记本**。
2. 在笔记本中，通过将以下命令添加到 **In [ ]:** 输入提示符，以安装并导入 PixieDust 和 ibmcloudsql，
然后**运行**。
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. 将 {{site.data.keyword.cos_short}} API 密钥添加到笔记本。这将允许 SQL Query 结果存储在 {{site.data.keyword.cos_short}} 中。
    - 在下一个 **In [ ]:** 提示符中添加以下命令，然后**运行**。
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - 在终端中，创建 API 密钥。
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - 将 **API 密钥**复制到剪贴板。
    - 将 API 密钥粘贴到笔记本中的相应文本框中，然后按 `Enter` 键。
    - 您还应该将 API 密钥存储到安全的永久位置；笔记本不会存储 API 密钥。
4. 将 SQL Query 实例的 CRN（云资源名称）添加到笔记本。
    - 在下一个 **In [ ]:** 提示符中，将 CRN 分配给笔记本中的一个变量。
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - 在终端中，将**标识**属性中的 CRN 复制到剪贴板。
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - 将 CRN 粘贴在单引号之间，然后**运行**。
5. 将另一个变量添加到笔记本，以指定 {{site.data.keyword.cos_short}} 存储区，然后**运行**。
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. 在另一个 **In [ ]:** 提示符中执行以下命令，然后**运行**以查看结果集。您还将有新的 `accidents/jobid=<id>/<part>.csv*` 文件添加到存储区，其中包含 `SELECT` 的结果。
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

## 使用 PixieDust 显示数据

在此部分中，您将使用 PixieDust 和 Mapbox 显示先前的结果集，以更好地识别交通事故的模式或热点。

1. 创建公共表表达式，用于将 `location` 列转换为单独的 `latitude` 和 `longitude` 列。在笔记本的提示符中，**运行**以下命令。
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
2. 在下一个 **In [ ]:** 提示符中，**运行** `display` 命令以使用 PixieDust 查看结果。
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. 选择“图表”下拉列表按钮；然后选择**地图**。
4. 将 `latitude` 和 `longitude` 添加到**键**。将 `id` 和 `age` 添加到**值**。单击**确定**以查看地图。
5. 单击**保存**图标以将笔记本保存到 {{site.data.keyword.cos_short}}。

![笔记本](images/solution29/notebook-mapbox.png)

## 与组织共享数据集

并非数据湖的所有用户都是数据研究员。您可以允许非技术用户使用 {{site.data.keyword.dynamdashbemb_notm}} 来从数据湖中获取洞察。与 SQL Query 类似，{{site.data.keyword.dynamdashbemb_notm}} 可以使用预构建的仪表板，直接从 {{site.data.keyword.cos_short}} 中读取数据。此部分说明的解决方案允许任何用户访问数据湖并构建定制仪表板。

1. 访问先前推送到 {{site.data.keyword.Bluemix_notm}} 的仪表板应用程序的公共 URL。
2. 选择与您的目标布局匹配的模板。（以下步骤使用的是第一行中的第二个布局。）
3. 使用`所选源`侧边栏中显示的`添加源`按钮，展开`存储区名称`手风琴图标，然后单击其中一个 `accidents/jobid=...` 表条目。使用右上角的 X 图标以关闭对话框。
4. 在左侧，单击`可视化`图标，然后单击**摘要**。
5. 选择 `accidents/jobid=...` 源，展开`表`，然后创建图表。
    - 将 `id` 拖放到**值**行上。
    - 使用上角的图标折叠图表。
6. 回到`可视化`中，创建**树状图**图表：
    - 将 `area` 拖放到**区域层次结构**行上。
    - 将 `id` 拖放到**大小**行上。
    - 折叠图表以查看结果。

![仪表板图表](images/solution29/dashboard-chart.png)

## 探索仪表板

在此部分中，您将执行一些额外步骤来探索仪表板应用程序和 {{site.data.keyword.dynamdashbemb_notm}} 的功能。

1. 单击样本仪表板应用程序工具栏中的**方式**按钮，以将方式视图更改为 `VIEW`。
2. 单击图表下部的任何彩色磁贴，或单击图表图注中的 `area` 值。这会将一个本地过滤器应用于该选项卡，从而导致其他图表显示特定于该过滤器的数据。
3. 单击工具栏中的**保存**按钮。
    - 在相应的输入字段中，输入仪表板的名称。
    - 选择**规范**选项卡，以查看此仪表板的规范。规范是 {{site.data.keyword.dynamdashbemb_notm}} 的本机文件格式。在其中，可找到有关已创建的图表以及所使用的 {{site.data.keyword.cos_short}} 数据源的信息。
    - 使用对话框的**保存**按钮将仪表板保存到浏览器的本地存储器中。
4. 单击工具栏的**新建**按钮以创建新仪表板。要打开已保存的仪表板，请单击**打开**按钮。要删除仪表板，请使用“打开仪表板”对话框中的**删除**图标。

在生产应用程序中，请对 URL、用户名和密码等信息加密，以阻止最终用户看到这些信息。请参阅[加密数据源信息](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information)。
{: tip}

## 扩展教程

祝贺您，您已使用 {{site.data.keyword.cos_short}} 构建了数据湖。下面是增强数据湖的其他建议。

- 使用 SQL Query 试用其他数据集
- 完成[通过流式分析和 SQL 使用大数据日志](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics)，以将来自多个源的数据流式传输到数据湖中
- 编辑仪表板应用程序的代码，以将仪表板规范存储到 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) 或 {{site.data.keyword.cos_short}}
- 创建 [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id) 服务实例以在仪表板应用程序中启用安全性

## 除去资源

运行以下命令除去使用的服务、应用程序和密钥。

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

## 相关内容

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Jupyter Notebook](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
