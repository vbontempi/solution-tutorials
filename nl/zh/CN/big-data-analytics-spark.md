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

# 使用 Apache Spark 分析和显示开放数据
{: #big-data-analytics-spark}

在本教程中，您将使用 {{site.data.keyword.DSX_full}}、Jupyter Notebook 和 Apache Spark 分析和显示开放数据集。首先，将描述人口增长、预期寿命和国家/地区 ISO 代码的数据组合到单个数据帧中。然后，为了发现洞察，将使用名为 Pixiedust 的 Python 库以各种方式查询和显示数据。

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## 目标
{: #objectives}

* 在 IBM Cloud 上部署 Apache Spark 和 {{site.data.keyword.DSX_short}}
* 使用 Jupyter Notebook 和 Python 内核
* 导入、变换、分析和显示数据集

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 服务和环境设置
首先供应本教程中使用的服务，并在 {{site.data.keyword.DSX_short}} 中创建项目。

可以通过[资源列表](https://{DomainName}/resources)和[目录](https://{DomainName}/catalog/)，供应 {{site.data.keyword.Bluemix_short}} 的各种服务。或者，通过 {{site.data.keyword.DSX_short}} 的仪表板和项目设置，可以创建或添加现有“数据和分析”服务。
{:tip}

1. 在 [{{site.data.keyword.Bluemix_short}}“目录”](https://{DomainName}/catalog)中，导航至 **AI** 部分。创建 **{{site.data.keyword.DSX_short}}** 服务。单击**开始使用**按钮以启动 **{{site.data.keyword.DSX_short}}** 仪表板。
2. 在仪表板中，单击**创建项目**磁贴 > 选择**标准** > 创建项目。在**名称**字段中，输入 `1stProject` 作为名称。您可以将描述保留为空。
3. 在页面右侧，可以**定义存储器**。如果已供应存储器，请从列表中选择一个实例。如果未供应，请单击**添加**，然后按照新浏览器选项卡中的指示信息进行操作。完成服务创建后，单击**刷新**以查看新服务。
4. 单击**创建**按钮以单击项目。这会将您重定向到项目的概述页面。  
   ![](images/solution23/NewProject.png)
5. 在概述页面上，单击**设置**。
6. 在**关联的服务**部分中，单击**添加服务**，然后从菜单中选择 **Spark**。在生成的屏幕中，可以选择现有的 Spark 服务实例，也可以创建新实例。

## 创建和准备笔记本
[Jupyter Notebook](http://jupyter.org/) 是一种开放式源代码 Web 应用程序，允许创建和共享包含实时代码、公式、可视化和描述文本的文档。笔记本和其他资源在项目中进行组织。
1. 单击**资产**选项卡，向下滚动至**笔记本**部分，然后单击**新建笔记本**。
2. 使用**空白**笔记本。输入 `MyNotebook` 作为**名称**。
3. 在**选择运行时**菜单中，选择已添加到项目设置的 Spark 实例。使缺省**语言**保留为 **Python 3.5**。
4. 单击**创建笔记本**以完成该过程。
5. 在其中输入文本和命令的字段称为**单元格**。将以下代码复制到空单元格中，以导入 [**Pixiedust** 包](https://pixiedust.github.io/pixiedust/use.html)。通过单击工具栏中的**运行**图标，或按键盘上的 **Shift+Enter** 键来执行单元格。
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

如果您从未使用过 Jupyter Notebook，请单击右上方菜单中的**文档**图标。导航至**分析数据**，然后导航至[**笔记本**部分](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics)，以了解有关[笔记本及其各部分](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true)的更多信息。
{:tip}

## 装入数据
接下来装入三个开放数据集，并使其在笔记本中可用。通过 **Pixiedust** 库，可以轻松地[使用 URL 装入 **CSV** 文件](https://pixiedust.github.io/pixiedust/loaddata.html)。

1.  将以下行复制到笔记本中的下一个空单元格中，但现在不要运行。
   ```Python
   df_pop = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
2. 在另一个浏览器选项卡中，转至[社区](https://dataplatform.ibm.com/community?context=analytics)部分。在**数据集**下，搜索 [**Total population by country**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e)，然后单击相关磁贴。单击右上方的**链接**图标以获取访问 URI。复制该 URI，然后将笔记本单元格中的文本 **YourAccessURI** 替换为该链接。单击工具栏中的**运行**图标，或按 **Shift+Enter** 键。
3. 对另一个数据集重复此步骤。将以下行复制到笔记本中的下一个空单元格中。
   ```Python
   df_life = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
4. 在带有**数据集**的另一个浏览器选项卡中，搜索 [**Life expectancy at birth by country in total years**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895)。再次获取相应链接，并使用该链接替换笔记本单元格中的 **YourAccessURI**，然后单击**运行**以启动装入过程。
5. 对于这三个数据集的最后一个数据集，从 Github 上的一组开放数据集装入国家/地区名称及其 ISO 代码列表。将以下代码复制到下一个空白笔记本单元格中并运行该代码。
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

稍后将使用国家/地区代码列表来简化数据选择，因为选择数据时将使用国家/地区代码，而不是确切的国家/地区名称。

## 变换数据
数据可用后，对其略做变换，然后可将这三个集组合成单个数据帧。
1. 以下代码块将重新定义用于人口数据的数据帧。这是通过对列重命名的 SQL 语句完成的。然后，会创建视图并显示模式。将以下代码复制到下一个空白单元格中并运行该代码。
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. 对预期寿命数据重复相同操作。此代码不会显示模式，而会改为显示前 10 行。  
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. 对国家/地区数据重复模式变换。
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. 现在，在可以组合成单个数据帧的各数据集内，列名更简单并保持一致。对预期寿命和人口数据执行**外**连接。然后，在同一语句中，通过**内**连接添加国家/地区代码。所有内容都将按国家/地区和年份排序。输出会定义数据帧 **df_all**。通过使用内连接，生成的数据仅包含可在 ISO 列表中找到的国家/地区。此过程将从数据中除去区域条目和其他条目。
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. 将**年份**的数据类型更改为整数。
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

现在组合的数据已准备好供分析。

## 分析数据
在此部分中，请[使用 Pixiedust 将数据显示为不同的图表](https://pixiedust.github.io/pixiedust/displayapi.html)。首先，比较一些国家/地区的预期寿命。

1. 将以下代码复制到下一个空白单元格中并运行该代码。
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. 这将显示一个可滚动的表。单击代码块正下方的“图表”图标，然后选择**折线图**。这将显示一个弹出对话框，其中包含 **Pixiedust：折线图选项**。输入**图表标题**，如“预期寿命比较”。从提供的**字段**中，将**年份**拖入**键**框，将**寿命**拖入**值**区域。对于**要显示的行数**，输入 **1000**。按**确定**以绘制折线图。在右侧，确保选择 **mapplotlib** 作为**渲染器**。单击**聚类依据**选择器，然后选择**国家/地区**。这将显示类似于下面的图表。
   ![](images/solution23/LifeExpectancy.png)

3. 创建焦点位于 2010 年的图表。将以下代码复制到下一个空白单元格中并运行该代码。
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. 在图表选择器中，选择**地图**。在配置对话框中，将**国家/地区**拖入**键**区域。将**寿命**拖入**值**框。与第一个图表类似，将**要显示的行数**增大到 **1000**。按**确定**以绘制地图。选择 **brunel** 作为**渲染器**。这将显示相对于预期寿命着色的世界地图。您可以使用鼠标来放大地图。
   ![](images/solution23/LifeExpectancyMap2010.png)

## 扩展教程
下面是一些增强本教程的想法和建议。
* 创建并显示查询，以展示相对于您所选国家/地区的人口增长的预期寿命比率
* 在世界地图上计算并显示每个国家/地区的人口增长率
* 从数据集目录中装入和集成其他数据
* 将组合数据导出到文件或数据库

## 相关内容
{:related}
下面提供了与本教程中涵盖的主题相关的链接。
* [Watson Data Platform](https://dataplatform.ibm.com)：使用 Watson Data Platform 可协作和构建更智能的应用程序。您可快速显示数据，从数据中发现洞察，并在不同团队之间进行协作。
* [PixieDust](https://www.ibm.com/cloud/pixiedust)：用于 Jupyter Notebook 的开放式源代码生产力工具
* [Cognitive Class.ai](https://cognitiveclass.ai/)：数据研究和认知计算课程
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/)：我们利用数据实现的一切，您也能实现
* [Analytics Engine 服务](https://{DomainName}/catalog/services/analytics-engine)：使用开放式源代码 Apache Spark 和 Apache Hadoop 开发和部署分析应用程序
