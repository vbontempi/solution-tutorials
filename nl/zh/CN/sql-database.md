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

# 用于云数据的 SQL 数据库
{: #sql-database}

本教程说明了如何供应 SQL（关系）数据库服务，创建表以及将大型数据集（城市信息）装入到数据库中。然后，将部署 Web 应用程序“worldcities”，以利用这些数据并显示如何访问云数据库。该应用程序是使用 [Flask 框架](http://flask.pocoo.org/)以 Python 编写的。

![](images/solution5/Architecture.png)

## 目标

* 供应 SQL 数据库
* 创建数据库模式（表）
* 装入数据
* 连接应用程序和数据库服务（共享凭证）
* 监视、安全性、备份和恢复

## 产品

本教程使用以下产品：
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## 开始之前
{: #prereqs}

转至 [GeoNames](http://www.geonames.org/)，然后下载并解压缩 [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip) 文件。此文件包含有关人口超过 1000 的城市的信息。您将使用此文件作为数据集。

## 供应 SQL 数据库
首先，创建 **[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)** 服务实例。

![](images/solution5/Catalog.png)

1. 访问 [{{site.data.keyword.Bluemix_short}}“仪表板”](https://{DomainName})。单击顶部导航栏中的**目录**。
2. 单击左侧窗格中“平台”下的**数据和分析**，然后选择 **{{site.data.keyword.dashdbshort_notm}}**。
3. 选取**入门**套餐，然后将建议的服务名称更改为“sqldatabase”（稍后您将使用该名称）。选取数据库的部署位置，并确保选择正确的组织和空间。
4. 单击**创建**。片刻之后，您应该会收到成功通知。
5. 在**资源列表**中，单击新创建的 {{site.data.keyword.dashdbshort_notm}} 服务的条目。
6. 单击**打开**以启动数据库控制台。如果这是第一次使用控制台，系统将向您提供教程。

## 创建表
您需要表来保存样本数据。请使用控制台来创建表。

1. 在 {{site.data.keyword.dashdbshort_notm}} 的控制台中，单击导航栏中的**探索**。这会将您转至数据库中现有模式的列表。
2. 找到并单击以“DASH”开头的模式。
3. 单击“**+ 新建表"**”以显示用于表名及其列的表单。
4. 输入“cities”作为表名。将 [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) 文件中的列定义复制并粘贴到列和数据类型的对应框中。
5. 单击**创建**以定义新表。   
   ![](images/solution5/TableCitiesCreated.png)

## 装入数据
既然已创建了“cities”表，现在要将数据装入到该表中。这可以采用不同的方式完成，例如从本地计算机装入、使用 Swift 或 AmazonS3 接口从 Cloud Object Storage (COS) 装入，或利用 [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift) 迁移服务装入。在本教程中，将从计算机上传数据。在此过程中，可以调整表结构和数据格式，以完全匹配文件内容。

1. 在顶部导航中，单击**装入**。然后，在**文件选择**下，单击**浏览文件**以找到并选取在本指南第一部分中下载的“cities1000.txt”文件。
2. 单击**下一步**以访问模式概述。再次选择以“DASH”开头的模式，然后选择“CITIES”表。再次单击**下一步**。   

   因为该表是空的，所以附加到现有数据或覆盖现有数据都是一样的效果。
   {:tip }
3. 现在，定制在装入过程中如何解释“cities1000.txt”文件中的数据。首先，禁用“第一行为标题”，因为该文件仅包含数据。接着，输入“0x09”作为分隔符。这意味着文件中的值将用制表符分隔。最后，选取“YYYY-MM-DD”作为日期格式。现在，所有内容应该类似于下面的屏幕快照。    
  ![](images/solution5/LoadTabSeparator.png)
4. 单击**下一步**，即可查看装入设置。同意并单击**开始装入**以开始将数据装入到“CITIES”表中。系统将显示进度。上传数据后，应该只需几秒钟就可完成装入，随即会显示一些统计信息。   
   ![](images/solution5/LoadProgressSteps.png)

## 使用 SQL 验证装入的数据
数据已装入到关系数据库中。未发生任何错误，但仍然应该执行一些快速测试。请使用内置 SQL 编辑器来输入并执行一些 SQL 语句。

1. 在顶部导航中，单击**运行 SQL**。
   您可以不使用内置 SQL 编辑器，而改为将桌面或服务器上基于云的和传统的 SQL 工具用于 {{site.data.keyword.dashdbshort_notm}}。连接信息可以在“设置”菜单中找到。某些工具甚至可以在“书籍”图标（代表文档和帮助）后面提供的菜单的“下载”部分中进行下载。
    {:tip }
2. 在“SQL 编辑器”中，输入或复制以下查询：   
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   然后按**全部运行**按钮。在“结果”部分中，显示的行数应该与装入过程所报告的行数相同。   
3. 在“SQL 编辑器”中，在新行上输入以下语句：
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. 在编辑器中，选择上述语句的文本。单击**运行所选内容**按钮。现在应该只会执行此语句，并在“结果”部分中按国家/地区统计信息返回某些信息。

## 部署应用程序代码
数据库应用程序的可随时运行的代码位于[此 Github 存储库中](https://github.com/IBM-Cloud/cloud-sql-database)。请克隆或下载此存储库，然后将其推送到 IBM Cloud。

1. 克隆 GitHub 存储库：
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. 将应用程序推送到 IBM Cloud。您需要登录到向其供应数据库的位置、组织和空间。复制并粘贴这些命令，一次一行。
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. 推送过程完成后，您应该能够访问该应用程序。无需进一步配置。`manifest.yml` 文件会指示 IBM Cloud 将应用程序与名为“sqldatabase”的数据库服务绑定在一起。

## 安全性、备份和恢复以及监视
{{site.data.keyword.dashdbshort_notm}} 是一项受管服务。IBM 负责确保环境安全、处理每日备份和系统监视。在入门套餐中，数据库环境是多租户设置，减少了针对用户的管理和选项配置工作。但是，如果您使用的是其中一个企业套餐，那么有[多个选项可管理用户，配置更多数据库安全性](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html)以及[监视数据库](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html)。   

除了传统的管理选项外，[{{site.data.keyword.dashdbshort_notm}} 服务还提供了支持监视、用户管理、实用程序、装入、存储访问等操作的 REST API](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html)。可以在“REST API”下“书籍”图标后面的菜单中，访问该 API 的可执行 Swagger 接口。一些可用于监视等功能的工具（例如，IBM Data Server Manager）甚至可以在该相同菜单的“下载”部分下进行下载。

## 测试应用程序
基于装入的数据集显示城市信息的应用程序已缩减至最小。此应用程序提供了一个搜索表单，用于指定城市名称和若干预配置的城市。这些项将转换为 `/search?name=cityname`（搜索表单）或 `/city/cityname`（直接指定的城市）。这两个请求都在后台通过相同的代码行进行处理。出于安全原因，将使用参数标记将 cityname 作为值传递给预编译 SQL 语句。从数据库中访存行，然后将其传递给 HTML 模板以进行呈现。

## 清除
要清除本教程使用的资源，请执行以下步骤：
1. 访问 [{{site.data.keyword.Bluemix_short}} 资源列表](https://{DomainName}/resources)。找到您的应用程序。
2. 单击应用程序的“菜单”图标，然后选择**删除应用程序**。在对话框窗口中，勾选要删除相关 {{site.data.keyword.dashdbshort_notm}} 服务的复选标记。
3. 单击**删除**按钮。这将除去应用程序和数据库服务，然后您将返回到资源列表。

## 扩展教程
要扩展此应用程序吗？下面是一些构想：
1. 提供对备用名称的通配符搜索。
2. 搜索特定国家/地区中仅特定人口值的城市。
3. 通过替换 CSS 样式和扩展模板，更改页面布局。
4. 允许基于表单创建新城市信息，或允许更新现有数据，例如人口。

## 相关内容
* 文档：[IBM Knowledge Center 上有关 {{site.data.keyword.dashdbshort_notm}} 的信息](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.Db2_on_Cloud_long_notm}} 和 {{site.data.keyword.dashdblong_notm}} 常见问题](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html)，解答了与受管服务、数据备份、数据加密和安全性等相关的问题。
* 面向开发者的[免费 Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions)
* 文档：[ibm_db Python 驱动程序的 API 描述](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
