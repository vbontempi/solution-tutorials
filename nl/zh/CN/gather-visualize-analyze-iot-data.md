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

# 收集、显示和分析 IoT 数据
{: #gather-visualize-analyze-iot-data}
本教程将全程指导您设置 IoT 设备，在 {{site.data.keyword.iot_short_notm}} 中收集数据，探索数据和创建可视化，然后使用高级机器学习服务来分析数据并检测历史数据中的异常。
{:shortdesc}

## 目标
{: #objectives}

* 设置 IoT 模拟器。
* 将集合数据发送到 {{site.data.keyword.iot_short_notm}}。
* 创建可视化。
* 分析设备生成的数据并检测异常。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Node.js 应用程序](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* 使用 Spark 服务和 {{site.data.keyword.cos_full_notm}} 的 [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* 设备使用 MQTT 协议将传感器数据发送到 {{site.data.keyword.iot_full}}
* 历史数据将导出到 {{site.data.keyword.cloudant_short_notm}} 数据库中
* {{site.data.keyword.DSX_short}} 从此数据库中拉取数据
* 通过 Jupyter 笔记本分析和显示数据

## 开始之前
{: #prereqs}

[{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - 运行脚本以安装 ibmcloud cli 和必需的插件

## 创建 IoT Platform
{: #iot_starter}

首先，将创建 Internet of Things Platform 服务 - 这是一个主数据中心，可以管理设备，安全地建立连接和**收集数据**，以及使历史数据可供可视化和应用程序使用。

1. 转至 [**{{site.data.keyword.Bluemix_notm}}“目录”**](https://{DomainName}/catalog/)，然后在**物联网**部分下，选择 [**Internet of Things Platform**](https://{DomainName}/catalog/services/internet-of-things-platform)。
2. 输入 `IoT demo hub` 作为服务名称，单击**创建**，然后**启动**仪表板。
3. 从侧边菜单中，选择**安全性 > 连接安全性**，在**缺省规则** > **安全级别**下选择 **TLS（可选）**，然后单击**保存**。
4. 从侧边菜单中，选择**设备** > **设备类型**和 **+ 添加设备类型**。
5. 输入 `simulator` 作为**名称**，然后单击**下一步**和**完成**。
6. 接下来，单击**注册设备**。
7. 对于**选择现有设备类型**，选择 `simulator`，然后对于**设备标识**，输入 `phone`。
8. 单击**下一步**，直到显示**设备安全性**（在“安全性”选项卡下）屏幕。
9. 为**认证令牌**输入值，例如：`myauthtoken`，然后单击**下一步**。
10. 单击**完成**后，将显示连接信息。请使此选项卡保持打开。

现在，IoT Platform 已配置，可开始接收数据。设备需要使用指定的设备类型、标识和令牌将其数据发送到 IoT Platform。

## 创建设备模拟器
{: #confignodered}
接下来，将部署 Node.js Web 应用程序并在手机上对其进行访问，该 Web 应用程序将连接到 IoT Platform，并向其发送设备加速计和方向数据。

1. 克隆 GitHub 存储库：
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. 在所选的 IDE 中打开代码，然后将 **manifest.yml** 文件中的 `name` 和 `host` 值更改为唯一值。
3. 将应用程序推送到 {{site.data.keyword.Bluemix_notm}}。
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. 应用程序会在几分钟后部署完成，您应该会看到类似于 `<UNIQUE_NAME>.mybluemix.net` 的 URL。
5. 在手机上使用浏览器访问此 URL。
6. 在**设备凭证**下，输入“IoT 仪表板”选项卡中的连接信息，然后单击**连接**。
7. 手机将开始传输数据。返回到 **IBM {{site.data.keyword.iot_short_notm}} 选项卡**，检查**最近事件**部分中是否有新条目。
  ![](images/solution16/recent_events_with_phone.png)

## 在 IBM {{site.data.keyword.iot_short_notm}} 中显示实时数据
{: #createcards}
接下来，您将创建一个板和多个卡，以在仪表板中显示设备数据。有关板和卡的更多信息，请参阅[使用板和卡显示实时数据](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html)。

### 创建板
{: #createboard}

1. 打开 **IBM {{site.data.keyword.iot_short_notm}} 仪表板**。
2. 从左侧菜单中选择**板**，然后单击**新建板**。
3. 输入板的名称，例如 `Simulators`，然后单击**下一步**，再单击**提交**。  
4. 选择刚才创建的板以将其打开。

### 显示设备数据
{: #cardtemp}
1. 单击**添加新卡**，然后选择**折线图**卡类型（位于“设备”部分中）。
2. 从列表中选择设备，然后单击**下一步**。
3. 单击**连接新数据集**。
4. 在“创建值卡”页面中，选择或输入以下值，然后单击**下一步**。
   - 事件：sensorData
   - 属性：ob
   - 名称：OrientationBeta
   - 类型：Float
   - 最小值：-180
   - 最大值：180
5. 在“卡预览”页面中，对于折线图大小，选择 **L**，然后单击**下一步** > **提交**。
6. 该卡会显示在仪表板中，并包含实时温度数据的折线图。
7. 使用手机浏览器重新启动模拟器，然后慢慢向前和向后倾斜手机。
8. 返回 **IBM {{site.data.keyword.iot_short_notm}} 选项卡**，您应该会看到图表已更新。
   ![](images/solution16/board.png)

## 将历史数据存储在 {{site.data.keyword.cloudant_short_notm}} 中
1. 转至 [**{{site.data.keyword.Bluemix_notm}}“目录”**](https://{DomainName}/catalog/)，然后创建名为 `iot-db` 的新 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)。
2. 在**连接**下：
   1. **创建连接**
   1. 选择 Cloud Foundry 位置、组织和空间，在其中应该创建 {{site.data.keyword.cloudant_short_notm}} 服务的别名。
   1. 展开**连接位置**表中的空间名称，然后使用 **iot demo hub** 旁边的**连接**按钮为该空间中的 {{site.data.keyword.cloudant_short_notm}} 服务创建别名。 
   1. 连接应用程序并对其重新编译打包。
3. 打开 **IBM {{site.data.keyword.iot_short_notm}} 仪表板**。
4. 从左侧菜单中选择**扩展**，然后单击**历史数据存储**下的**设置**。
5. 选择 `iot-db` {{site.data.keyword.cloudant_short_notm}} 数据库。
6. 对于**数据库名称**，输入 `devicedata`，然后单击**完成**。
7. 新窗口应该会装入要求授权的提示。如果没有看到此窗口，请禁用弹出窗口阻止程序并刷新页面。

现在，设备数据已保存在 {{site.data.keyword.cloudant_short_notm}} 中。几分钟后，启动 {{site.data.keyword.cloudant_short_notm}} 仪表板即可看到您的数据。

![](images/solution16/cloudant.png)

## 使用机器学习检测异常
{: #data_experience}

在此部分中，您将使用 IBM {{site.data.keyword.DSX_short}} 服务中提供的 Jupyter Notebook 来装入历史移动数据，并使用 Z 得分来检测异常。*Z 得分*是一种标准分数，指示元素与平均值的标准差。

### 创建新项目
1. 转至 [**{{site.data.keyword.Bluemix_notm}}“目录”**](https://{DomainName}/catalog/)，然后在 **AI** 下，选择 [**{{site.data.keyword.DSX_short}}**](https://{DomainName}/catalog/services/data-science-experience)。
2. 通过单击**开始使用**，**创建**服务并启动其仪表板。
3. 创建项目 > 选择**数据研究** > 单击**创建项目**，然后输入 `Detect Anomaly` 作为项目的**名称**。
4. 将**限制合作者资格**复选框保持未选中状态，因为没有保密数据。
5. 在**定义存储器**下，单击**添加**，然后选择现有 **Cloud Object Storage** 服务或创建新的 Cloud Object Storage 服务（选择**轻量**套餐 > 创建）。点击**刷新**可看到创建的服务。
6. 单击**创建**。新项目将打开，可以开始向其添加资源。

### 连接到 {{site.data.keyword.cloudant_short_notm}} 以获取数据

1. 单击**资产** > **+ 添加到项目** > **连接**。  
2. 选择存储了设备数据的 **iot-db** {{site.data.keyword.cloudant_short_notm}}。
3. 交叉检查**凭证**，然后单击**创建**。

### 创建 Apache Spark 服务

1. 单击顶部导航栏上的**服务** > 计算服务。
2. 单击**添加服务**。
   1. 在 **Apache Spark** 上单击**添加**。
   1. 选择**轻量**套餐。
   1. 单击**创建**。
3. 选择组织和空间，根据需要更改服务名称，然后**确认**。
1. 在**项目**中导航至 `Detect Anomaly` 项目。
1. 在**设置**中，滚动到**关联的服务**。
1. 单击**添加服务**，然后选择 **Spark**。
1. 选择先前安装的 **Apache Spark** 实例。

### 创建 Jupyter (ipynb) 笔记本
1. 单击 **+ 添加到项目**，然后添加新的**笔记本**。
2. 对于**名称**，输入 `Anomaly-detection-notebook`。
3. 在**笔记本 URL** 中，输入 `https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb`。
4. 选择先前作为运行时关联的 **Apache Spark** 服务。
5. 创建**笔记本**。将 `Python 3.5 with Spark 2.1` 设置为内核。检查笔记本是否是使用元数据和代码创建的。
   ![Jupyter Notebook Watson Studio](images/solution16/jupyter_notebook_watson_studio.png)
   要进行更新，请单击**内核** > 更改内核。要**信任**该笔记本，请单击**文件** > 信任笔记本。
   {:tip}

### 运行笔记本并检测异常   
1. 选择以 `!pip install --upgrade pixiedust,` 开头的单元格，然后单击**运行**或按 **Ctrl+Enter** 键来执行代码。
2. 安装完成后，通过单击**重新启动内核**图标，重新启动 Spark 内核。
3. 在下一个代码单元格中，通过完成以下步骤，将 {{site.data.keyword.cloudant_short_notm}} 凭证导入到该单元格中：
   * 单击 ![](images/solution16/data_icon.png)
   * 选择**连接**选项卡。
   * 单击**插入到代码**。这将使用 {{site.data.keyword.cloudant_short_notm}} 凭证创建名为 _credentials_1_ 的字典。如果该字典的名称未指定为 _credentials_1_，请将其重命名为 `credentials_1`。`credentials_1` 将在其余单元格中使用。
4. 在具有数据库名称 (`dbName`) 的单元格中，输入作为数据源的 {{site.data.keyword.cloudant_short_notm}} 数据库的名称，例如 *iotp_yourWatsonIoTProgId_DBName_Year-month-day*。要显示不同设备的数据，请相应地更改 `deviceId` 和 `deviceType` 的值。可以通过导航至先前创建的 **iot-db** {{site.data.keyword.cloudant_short_notm}} 实例 > 启动仪表板来查找确切的数据库。
   {:tip}
5. 保存笔记本，然后逐一执行每个代码单元格或运行所有单元格（**单元格** > 全部运行），在笔记本结束时，应该会看到设备流动数据（oa、ob 和 og）中的异常。可以将关注的时间间隔更改为一天中所需的时间。请查找 `start` 和 `end` 值。
   {:tip}
   ![Jupyter Notebook DSX](images/solution16/anomaly_detection_watson_studio.png)
6. 除了异常检测外，此部分中的主要发现结果或要点如下：
    * 使用 Spark 准备要进行可视化的数据。
    * 使用 Pandas 进行数据可视化。
    * 条形图和直方图用于表示设备数据。
    * 通过相关性矩阵体现两个传感器之间的相关性。
    * 每个设备传感器一个箱图，使用 Pandas 绘图功能生成。
    * 密度图通过内核密度估算 (KDE) 生成。
    ![](images/solution16/density_plots_sensor_data.png)

## 除去资源
{:removeresources}

1. 导航至[资源列表](https://{DomainName}/resources/) > 选择在其中创建应用程序和服务的位置、组织和空间。在 **Cloud Foundry 应用程序**下，删除上面创建的 Node.js 应用程序。
2. 在**服务**下，删除为本教程创建的各个 Internet of Things Platform、Apache Spark、{{site.data.keyword.cloudant_short_notm}} 和 {{site.data.keyword.cos_full_notm}} 服务。

## 相关内容
{:related}

* [构建、部署、测试和重新训练预测性机器学习模型](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* [IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html) 概述
* [Jupyter Notebook](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb) 异常检测
* [了解 Z 得分](https://en.wikipedia.org/wiki/Standard_score)
* 使用深度学习开发用于异常检测的认知 IoT 解决方案 - [5 个帖子系列](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/)
