---
copyright:
  years: 2017, 2019
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

# 创建、保护和管理 REST API
{: #create-manage-secure-apis}

本教程演示如何使用 LoopBack Node.js API Framework 创建 REST API。使用 Loopback，您可以快速创建将设备和浏览器连接到数据和服务的 REST API。您还可以使用 {{site.data.keyword.apiconnect_long}} 向 API 添加管理、可视性、安全性和速率限制。
{:shortdesc}

## 目标

* 在几乎无需编码的情况下构建 REST API
* 在 {{site.data.keyword.Bluemix_notm}} 上发布 API 以供开发人员使用
* 将现有的 API 引入 {{site.data.keyword.apiconnect_short}}
* 安全地公开和控制记录系统的访问权限

## 使用的服务

本教程使用以下运行时和服务：

* [Loopback](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* [SDK for Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry 应用程序

## 体系结构

![体系结构](images/solution13/Architecture.png)

1. 开发人员定义 RESTful API
2. 开发人员将 API 发布到 {{site.data.keyword.apiconnect_long}}
3. 用户和应用程序使用 API

## 开始之前

* 下载和安装 [Node.js](https://nodejs.org/en/download/) V6.x（如果您已经安装了更新版本的 Node.js，请使用 [nvm](https://github.com/creationix/nvm) 或类似工具）

## 在 Node.js 中创建 REST API

{: #create_api}
在本部分中，您将使用 [LoopBack](https://loopback.io/doc/index.html) 在 Node.js 中创建 API。LoopBack 是高度可扩展的开放式源代码 Node.js 框架，使得您几乎无需编码就可以创建动态的端对端 REST API。

### 创建应用程序

1. 安装 {{site.data.keyword.apiconnect_short}} 命令行工具。如果您安装 {{site.data.keyword.apiconnect_short}} 时遇到问题，请在该命令之前使用 `sudo` 或者遵循[此处](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit)的指示信息。
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. 输入以下命令以创建应用程序。
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. 按 `Enter` 键以使用 **entries-api** 作为**应用程序的名称**。
4. 按 `Enter` 键以使用 **entries-api** 作为**包含项目的目录**。
5. 选择 **3.x（当前）**作为 **LoopBack 的版本**。
6. 选择 **empty-server** 作为**应用程序的类型**。

![APIC Loopback 脚手架](images/solution13/apic_loopback.png)

### 添加数据源

数据源表示后端系统，例如数据库、外部 REST API、SOAP Web Service 以及存储服务。数据源通常提供创建、检索、更新和删除 (CRUD) 功能。Loopback 支持许多类型的[数据源](http://loopback.io/doc/en/lb3/Connectors-reference.html)，为了简便起见，您将通过 API 使用内存中数据存储。

1. 将目录切换到新的项目，并启动 API Designer。
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. 从导航中，单击**数据源**。然后，单击**添加**按钮。此时将打开**新的 LoopBack 数据源**对话框。
3. 在**名称**文本字段中，输入 `entriesDS`，并单击**新建**按钮。
4. 从**连接器**组合框中选择**内存中数据库**。
5. 单击左上方的**所有数据源**。数据源将在数据源列表中显示。

   编辑器将使用新数据源的设置自动更新 server/datasources.json 文件。
   {:tip}

![API Designer 数据源](images/solution13/datastore.png)

### 添加模型

模型是带有 Node 和 REST API 的 JavaScript 对象，表示后端系统中的数据。模型通过数据源连接到这些系统。在本部分中，您将定义数据结构，并将其连接到先前创建的数据源。

1. 从导航中，单击**模型**。然后，单击**添加**按钮。此时将打开**新建 LoopBack 模型**对话框。
2. 在**名称**文本字段中，输入 `entry`，并单击**新建**按钮。
3. 从**数据源**组合框中选择 **entriesDS**。
4. 从**属性**卡中，使用**添加属性**图标添加以下属性。
    1. 针对**属性名称**输入 `name`，并从**类型**组合框中选择 **string**。
    2. 针对**属性名称**输入 `email`，并从**类型**组合框中选择 **string**。
    3. 针对**属性名称**输入 `comment`，并从**类型**组合框中选择 **string**。
5. 单击右上角的**保存**图标以保存模型。

![模型生成器](images/solution13/models.png)

## 测试 LoopBack 应用程序

在本部分中，您将启动 Loopback 应用程序的本地实例，通过使用 API Designer 插入和查询数据来测试 API。您必须使用 API Designer 的 Explore 工具测试浏览器中的 REST 端点，因为它包括正确的安全性标头和其他请求参数。

1. 通过单击左下角的**开始**图标来启动本地服务器，并等待显示**正在运行**消息。
2. 从条幅中，单击**探索**链接以显示 API Designer 的“探索”工具。侧边栏显示可用于 API 中的 LoopBack 模型的 REST 操作。
3. 单击左侧窗格的 **entry.create** 操作以显示端点。中心窗格显示有关端点的摘要信息，包括其参数、安全性、模型实例数据和响应代码。右侧窗格提供了样本代码，以使用 cURL 命令和语言（如 Ruby、Python、Java 和 Node）调用该端点。
4. 在右侧窗格中，单击**试用**链接。向下滚动到**参数**，并在**数据**文本区域中输入以下内容：
    ```javascript
    {
      "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. 单击**调用操作**按钮。
  ![从 API Designer 测试 API](images/solution13/data_entry_1.png)
6. 通过查看 **Response Code: 200 OK** 来确认 POST 已成功。如果看到由于本地主机的证书不可信而出现的错误消息，请在 API Designer“探索”工具中单击该错误消息内提供的链接以接受证书，然后继续在 Web 浏览器中调用操作。具体过程取决于您使用的 Web 浏览器。
7. 使用 cURL 添加其他条目。确认端口与应用程序端口匹配。
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. 单击 **entry.find** 操作，然后单击后跟**调用操作**按钮的**试用**链接。响应显示所有条目。您应该会看到 **Jane Doe** 和 **John Doe** 的 JSON。
  ![entry_find](images/solution13/find_response.png)

您还可以从 `entries-api` 目录发出 `npm start` 命令来手动启动 Loopback 应用程序。您的 REST API 将在 [http://localhost:3000/api/entries](http://localhost:3000/api/entries) 中可用，但是，{{site.data.keyword.apiconnect_short}} 不会对其进行管理和保护。
{:tip}

## 创建 {{site.data.keyword.apiconnect_short}} 服务

要为后续步骤做准备，您将在 {{site.data.keyword.Bluemix_notm}} 上创建 **{{site.data.keyword.apiconnect_short}}** 服务。{{site.data.keyword.apiconnect_short}} 充当 API 的网关，还提供管理、安全性和速率限制。

1. 启动 {{site.data.keyword.Bluemix_notm}} [资源列表](https://{DomainName}/resources)。
2. 导航至**目录 > 集成 > {{site.data.keyword.apiconnect_short}}**，并单击**创建**按钮。

## 将 API 发布到 {{site.data.keyword.Bluemix_notm}}

{: #publish}
您将使用 API Designer 以将您的应用程序作为 Cloud Foundry 应用程序部署到 {{site.data.keyword.Bluemix_notm}}，并且还会将 API 定义发布到 **{{site.data.keyword.apiconnect_short}}**。API Designer 是您的本地工具箱。如果您已将其关闭，请从项目目录使用 `apic edit` 将其重新启动。

还可以使用 `ibmcloud cf push` 命令手动部署应用程序，但是，它不会受到安全保护。要[将 API 导入](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing)到 {{site.data.keyword.apiconnect_short}}，请使用 `definitions` 文件夹中提供的 OpenAPI 定义文件。使用 API Designer 进行部署将确保应用程序的安全，并自动导入定义。
{:tip}

1. 返回 API Designer，单击条幅中的**发布**链接。然后单击**添加和管理目标 > 添加 IBM Bluemix 目标**。
2. 选择您想要发布到的**区域**和**组织**。
3. 选择**沙箱**目录，并单击**下一步**。
4. 在**输入新应用程序名称**文本字段中，输入 `entries-api-application`，并单击 **+** 图标。
5. 单击列表中的 **entries-api-application**，并单击**保存**。
6. 单击条幅最左上角的汉堡**菜单**图标。然后单击**项目**和 **entries-api** 列表项。
7. 在 API Designer UI 中，单击 **API > entries-api > 组合**链接。
8. 在组合件编辑器中，单击**过滤策略**图标。
9. 选择 **DataPower Gateway 策略**并单击**保存**。
10. 单击顶部栏上的**发布**，并选择您的目标。选择**发布应用程序**和“编译打包或发布产品”> 选择**特定产品** > **entries-api**。
11. 单击**发布**，并等待应用程序完成发布。
    ![“发布”对话框](images/solution13/publish.png)

    应用程序包含与 API 相关的 Loopback 模型、数据源和代码。产品允许您声明 API 如何提供给开发人员。
    {:tip}

API 应用程序现在已作为 Cloud Foundry 应用程序发布到 {{site.data.keyword.Bluemix_notm}}。您可以通过查看 {{site.data.keyword.Bluemix_notm}} [资源列表](https://{DomainName}/resources)下的 Cloud Foundry 应用程序来看到该应用程序，但是由于该应用程序受保护，所以无法使用 URL 进行直接访问。下面的部分将向您显示如何访问受管的 API。

## API 网关

到现在为止，您一直在设计并本地测试您的 API。在本部分中，您将使用 {{site.data.keyword.apiconnect_short}} 在 {{site.data.keyword.Bluemix_notm}} 上测试部署的 API。

1. 启动 {{site.data.keyword.Bluemix_notm}} [资源列表](https://{DomainName}/resources)。
2. 找到并选择 **Cloud Foundry 服务**下的 **{{site.data.keyword.apiconnect_short}}** 服务。
3. 单击**探索**菜单，然后单击**沙箱**链接。
4. 单击 **entry.create** 操作。
5. 在右侧窗格中，单击**试用**。向下滚动到**参数**，并在**数据**文本区域中输入以下内容。
    ```javascript
    {
      "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. 单击**调用操作**按钮。应该会显示 **Code: 200** 响应，指示操作成功。

![网关](images/solution13/gateway.png)

您的受管安全 API URL 将显示在每个操作旁，应该类似于 `https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries`。
{: tip}

## 速率限制

通过设置速率限制，可以管理 API 以及 API 内的特定操作的网络流量。速率限制为特定时间间隔中允许的最大调用数。

1. 返回 API Designer，单击**产品 > entries-api**。
2. 选择左侧的**缺省套餐**。
3. 展开**缺省套餐**，向下滚动到**速率限制**字段。
4. 将字段设置为 **10** 个调用/**1** **分钟**。
5. 选择**强制实施硬限制**，并单击**保存**图标。
  ![“速率限制”页面](images/solution13/rate_limit.png)
6. 遵循[将 API 发布到 {{site.data.keyword.Bluemix_notm}}](#publish) 部分中的步骤，以重新发布 API。

您的 API 现在限制为每分钟 10 个请求。使用**试用**功能达到限制。请参阅有关[设置速率限制](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits)的更多信息，或者探索 API Designer 以了解可用的所有管理功能。

## 扩展教程

恭喜，您已经构建了受管且安全的 API。以下是增强您的 API 的其他建议。

* 使用 [{{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant) LoopBack 连接器添加数据持久性
* 使用 API Designer [查看其他设置](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0)以管理您的 API
* 复查 API **分析**和**可视化**（在 {{site.data.keyword.apiconnect_short}} 中[可用](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics)）

## 相关内容

* [Loopback 文档](https://loopback.io/doc/index.html)
* [{{site.data.keyword.apiconnect_long}} 入门](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
