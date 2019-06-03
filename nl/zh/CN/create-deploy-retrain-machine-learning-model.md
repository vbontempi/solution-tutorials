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

# 构建、部署、测试和重新训练预测性机器学习模型
{: #create-deploy-retrain-machine-learning-model}
本教程引导您逐步完成构建预测性机器学习模型，将其部署为 API 以用于应用程序，测试模型并使用反馈数据重新训练模型的过程。所有这些操作都在 IBM Cloud 的集成、统一的自助服务体验中进行。

在本教程中，将使用 **Iris flower data set** 来创建机器学习模型以对花的物种进行分类。

在机器学习术语中，分类被视为受监督的学习的实例，即了解已正确识别的观察结果的训练集在何处可用。
{:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## 目标
{: #objectives}

* 将数据导入项目。
* 构建机器学习模型。
* 部署模型并试用 API。
* 测试机器学习模型。
* 为连续学习和模型评估创建反馈数据连接。
* 重新训练您的模型。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## 开始之前
{: #prereqs}
* IBM Watson Studio 和 Watson Knowledge Catalog 应用程序属于 IBM Watson 的一部分。要创建 IBM Watson 帐户，请首先注册这两个应用程序之一或全部。

   转至[试用 IBM Watson](https://dataplatform.ibm.com/registration/stepone) 并注册 IBM Watson 应用程序。

## 将数据导入项目

{:#import_data_project}

项目是您组织资源以实现特定目标的方式。您的项目资源可以包括数据、合作者和分析工具，例如 Jupyter 笔记本和机器学习模型。

您可以创建项目以添加数据，并在数据优化器中打开数据资产以进行数据清理和塑形。

**创建项目：**

1. 转至 [{{site.data.keyword.Bluemix_short}} 目录](https://{DomainName}/catalog)并选择 **AI** 部分下的 [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services)。**创建**服务。单击**开始使用**按钮以启动 **{{site.data.keyword.DSX_short}}** 仪表板。
2. 创建**项目** > 单击**标准**磁贴上的**创建项目**。添加项目的名称（如 `iris_project`）和可选的描述。
3. 将**限制合作者资格**复选框保持未选中状态，因为没有保密数据。
4. 在**定义存储器**下，单击**添加**，并选择现有的 Cloud Object Storage 服务，或者创建新的服务（选择**轻量**套餐 > 创建）。点击**刷新**可看到创建的服务。
5. 单击**创建**。新项目将打开，可以开始向其添加资源。

**导入数据：**

如先前所提及那样，您将使用 **Iris 数据集**。Iris 数据集在 R.A. Fisher 1936 年发表的经典论文 [The Use of Multiple Measurements in Taxonomic Problems](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf) 中使用过，并且在 [UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/) 中也可以找到。这个小型数据集常常用于测试机器学习算法和可视化。其目标是根据萼片和花瓣的长度和宽度，将鸢尾花分类为 3 个物种（Setosa、Versicolor 或 Virginica）。此 iris 数据集包含各有 50 个实例的 3 个类别，其中每个类别对应一个鸢尾花种类。![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


**下载** [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv)，其中包含每个类别的 40 个实例。您将使用每个类别的其余 10 个实例来重新训练模型。

1. 在项目的**资产**下，单击**查找和添加数据**图标 ![显示“查找数据”图标。](images/solution22-build-machine-learning-model/data_icon.png)。
2. 在**装入**下，单击**浏览**并上传已下载的 `iris_initial.csv`。![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. 一旦添加，您应该会在项目的**数据资产**部分下看到 `iris_initial.csv`。单击名称以查看数据集的内容。

## 关联服务
{:#associate_services}
1. 在**设置**下，滚动到**关联的服务** > 单击**添加服务** > 选择 **Spark**。![](images/solution22-build-machine-learning-model/associate_services.png)
2. 选择**轻量**套餐，并单击**创建**。使用缺省值，并单击**确认**。
3. 再次单击**添加服务**，并选择 **Watson**。单击 **Machine Learning** 磁贴上的**添加** > 选择**轻量**套餐 > 单击**创建**。
4. 保留缺省值，并单击**确认**以供应 Machine Learning 服务。

## 构建机器学习模型

{:#build_model}

1. 单击**添加到项目**并选择 **Watson Machine Learning 模型**。在对话框中，添加 **iris_model** 作为名称，并添加可选的描述。
2. 在**机器学习服务**部分下，您应该会看到在上述步骤中关联的 Machine Learning 服务。![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. 选择**模型构建器**作为模型类型，并在 **Spark 服务或环境**部分下选择先前创建的 spark 服务
4. 选择**手动**以手动创建模型。单击**创建**。

   对于自动方法，您完全依赖于自动数据准备 (ADP)。对于手动方法，除了 ADP 变换器处理的一些功能外，您还可以添加和配置自己的估算工具，即在分析中使用的算法。
   {:tip}

5. 在下一页上，选择 `iris_initial.csv` 作为数据集，并单击**下一步**。
6. 在**选择技巧**页面上，根据所添加的数据集，将预先填充“标签列”和“特征列”。选择 **species (String)** 作为**标签列**，**petal_length (Decimal)** 和 **petal_width (Decimal)** 作为**特征列**。
7. 选择**多类分类**作为建议的技巧。![](images/solution22-build-machine-learning-model/model_technique.png)
8. 对于**验证拆分**，配置以下设置：

   **训练：**50%，
   **测试：**25%，
   **维持：**25%

9. 单击**添加估算工具**，选择**决策树分类器**，然后选择**添加**。

   您一次可以评估多个估算工具。例如，您可以添加**决策树分类器**和**随机林分类器**作为估算工具以训练模型，并根据评估输出选择最佳匹配。
   {:tip}

10. 单击**下一步**以训练模型。看到状态为**已训练 & 已评估**后，单击**保存**。![](images/solution22-build-machine-learning-model/trained_model.png)

11. 单击**概述**以检查模型的详细信息。

## 部署模型并试用 API

{:#deploy_model}

1. 在已创建模型下，单击**部署** > **添加部署**。
2. 选择 **Web Service**。添加名称（如 `iris_deployment`）和可选的描述。
3. 单击**保存**。在概述页面上，单击新的 Web Service 的名称。当状态为 **DEPLOY_SUCCESS** 时，您可以在**实施**下查看评分端点、各种编程语言的代码片段以及 API 规范。
4. 单击**查看 API 规范**以查看和测试 {{site.data.keyword.pm_short}} API 端点。![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   要开始使用 API，您需要使用 [{{site.data.keyword.Bluemix_short}} 资源列表](https://{DomainName}/resources/)下 {{site.data.keyword.pm_short}} 服务实例的**服务凭证**选项卡上可用的**用户名**和**密码**来生成**访问令牌**。遵循 API 规范页面上提及的指示信息以生成**访问令牌**。
   {:tip}
5. 要进行在线预测，请使用 `POST /online` API 调用。
   * 可以在 [{{site.data.keyword.Bluemix_short}} 资源列表](https://{DomainName}/resources/)下的 {{site.data.keyword.pm_short}} 服务的**服务凭证**选项卡上找到 `instance_id`。
   * `deployment_id` 和 `published_model_id` 在部署的**概述**下。
   *  对于 `online_prediction_input`，使用下面的 JSON

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * 单击**试用**查看 JSON 输出。

6. 使用 API 端点，您现在可以从任何应用程序调用此模型。

## 测试您的模型

{:#test_model}

1. 在**测试**下，您应该会看到自动填充了输入数据（特征数据）。
2. 单击**预测**，您应该会看到图表中包含**物种的预测值**。![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. 对于 JSON 输入和输出，单击活动的输入和输出旁的图标。
4. 您可以更改输入数据，并继续测试模型。

## 创建反馈数据连接

{:#create_feedback_connection}

1. 对于连续的学习和模型评估，您需要将新数据存储在某个位置。创建 [{{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse) 服务 > **入门**套餐，以充当反馈数据连接。
2. 在 {{site.data.keyword.dashdbshort}} **管理**页面上，单击**打开**。在顶部导航上，选择**装入**。
3. 单击**我的计算机**下的**浏览文件**，并上传 `iris_initial.csv`。单击**下一步**。
4. 选择 **DASHXXXX**（例如 DASH1234）作为**模式**，然后单击**新建表格**。将其命名为 `IRIS_FEEDBACK`，并单击**下一步**。
5. 将自动检测数据类型。单击**下一步**，然后单击**开始装入**。![](images/solution22-build-machine-learning-model/define_table.png)
6. 将创建新目标 **DASHXXXX.IRIS_FEEDBACK**。

   在下一步重新训练模型以实现更好的性能和精度时，将使用此表格。

## 重新训练模型

{:#retrain_model}

1. 返回到 [{{site.data.keyword.Bluemix_short}} 资源列表](https://{DomainName}/resources)，在您一直使用的 {{site.data.keyword.DSX_short}} 服务下，单击**项目** > iris_project >  **iris-model**（“资产”下）> 评估。
2. 在**性能监视器**下，单击**配置性能监视**。
3. 在“配置性能监视”页面上：
   * 选择 Spark 服务。应该会自动填充预测类型。
   * 选择 **weightedPrecision** 作为度量值，并设置 `0.98` 作为可选的阈值。
   * 单击**新建连接**以指向在上面部分中创建的云上 IBM Db2 Warehouse。
   * 选择 Db2 仓库连接，一旦填充连接详细信息后，单击**创建**。![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * 单击**选择反馈数据引用**并指向 IRIS_FEEDBACK 表格，然后单击**选择**。![](images/solution22-build-machine-learning-model/select_source.png)
   * 在**重新评估所需的记录计数**框中，输入触发重新训练所需最小数量的新记录。使用 **10** 或留空以使用缺省值 1000。
   * 在**自动重新训练**框中，选择以下某个选项：
     - 要在模型性能低于所设置的阈值时启动自动重新训练，请选择**模型性能低于阈值时**。对于本教程，将在精度低于阈值 (.98) 时选择此选项。
     - 要禁止自动重新训练，请选择**永不**。
     - 要启动自动重新训练而不管性能如何，请选择**始终**。
   * 在**自动部署**框中，选择以下某个选项：
     - 要在每当模型性能高于先前版本时启动自动部署，请选择**模型性能高于先前版本时**。对于本教程，选择此选项是因为我们的目标是持续提高模型性能。
     - 要禁止自动部署，请选择**永不**。
     - 要启动自动部署而不管性能如何，请选择**始终**。
   * 单击**保存**。![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. 下载文件 [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv)。此后，单击**添加反馈数据**，选择下载的 CSV 文件，然后单击**打开**。
5. 单击**新建评估**以开始。![](images/solution22-build-machine-learning-model/retraining_model.png)
6. 一旦评估完成后，您可以查看**上次评估结果**部分以获取改进的 **WeightedPrecision** 值。

## 除去资源
{:removeresources}

1. 导航至 [{{site.data.keyword.Bluemix_short}} 资源列表](https://{DomainName}/resources/) > 选择您在其中创建服务的位置、组织和空间。
2. 删除您为本教程创建的相应 {{site.data.keyword.DSX_short}}、{{site.data.keyword.sparks}}、{{site.data.keyword.pm_short}}、{{site.data.keyword.dashdbshort}} 和 {{site.data.keyword.cos_short}} 服务。

## 相关内容
{:related}

- [Watson Studio 概述](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [使用 Machine Learning 检测异常](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [自动模型创建](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [机器学习和人工智能](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
