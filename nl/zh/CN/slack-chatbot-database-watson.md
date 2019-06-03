---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 构建数据库驱动的 Slack 机器人
{: #slack-chatbot-database-watson}

在本教程中，您将构建一个 Slack 机器人，用于创建 Db2 数据库条目并在其中搜索事件和会议。Slack 机器人由 {{site.data.keyword.conversationfull}} 服务提供支持。您将使用 Assistant 集成来集成 Slack 和 {{site.data.keyword.conversationfull}}。

Slack 集成在 Slack 和 {{site.data.keyword.conversationshort}} 之间传递消息。在其中，一些服务器端对话操作会对 Db2 数据库执行 SQL 查询。所有（但并不多）代码都是用 Node.js 编写的。

## 目标
{: #objectives}

* 使用集成将 {{site.data.keyword.conversationfull}} 连接到 Slack
* 在 {{site.data.keyword.openwhisk_short}} 中创建、部署和绑定 Node.js 操作
* 使用 Node.js 从 {{site.data.keyword.openwhisk_short}} 访问 Db2 数据库

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}} ](https://{DomainName}/catalog/services/db2-warehouse) 或 [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

<p style="text-align: center;">

  ![体系结构](images/solution19/SlackbotArchitecture.png)
</p>

## 开始之前
{: #prereqs}

要完成本教程，您需要最新版本的 [{{site.data.keyword.Bluemix_notm}} CLI](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)，并且 {{site.data.keyword.openwhisk_short}} [插件已安装](/docs/cli?topic=cloud-cli-plug-ins)。


## 服务和环境设置
在此部分中，您将设置所需的服务并准备环境。其中大部分操作可以通过命令行界面 (CLI) 使用脚本来完成。脚本可在 GitHub 上获取。

1. 克隆 [GitHub 存储库](https://github.com/IBM-Cloud/slack-chatbot-database-watson)，然后导航至克隆的目录：
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. 如果您尚未登录，请使用 `ibmcloud login` 以交互方式登录。
3. 使用以下命令将要在其中创建数据库服务的组织和空间设定为目标：
   ```
      ibmcloud target --cf
      ```
4. 创建 {{site.data.keyword.dashdbshort}} 实例，并将其命名为 **eventDB**：
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
   您还可以使用除**入门**套餐以外的套餐。
5. 要在以后从 {{site.data.keyword.openwhisk_short}} 访问数据库服务，您需要授权。因此，请创建服务凭证，并将其标注为 **slackbotkey**：   
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. 创建 {{site.data.keyword.conversationshort}} 服务实例。使用 **eventConversation** 作为名称，并使用免费的轻量套餐。
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. 接下来，将注册 {{site.data.keyword.openwhisk_short}} 操作，并将服务凭证绑定到这些操作。某些操作作为 Web 操作启用，并且设置了私钥以阻止未经授权的调用。选择一个私钥，并将其作为参数传递 - 相应地替换 **YOURSECRET**。

   其中一个操作被调用，以在 {{site.data.keyword.dashdbshort}} 中创建表。通过使用 {{site.data.keyword.openwhisk_short}} 的操作，您既不需要本地 Db2 驱动程序，也不必使用基于浏览器的界面来手动创建表。要执行注册和设置，请运行下面的行，这将执行包含所有操作的 **setup.sh** 文件。如果系统不支持 shell 命令，请复制 **setup.sh** 文件中的每行，并分别执行各行。

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **注：**缺省情况下，脚本还会插入几行样本数据。可以通过在上面的脚本中注释掉以下行来禁用此插入操作：`#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r`
8. 抽取已部署操作的名称空间信息。

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   记下 **/slackdemo/eventInsert** 前面的部分。这是已编码的组织和空间。在下一部分中将需要此信息。

## 装入技能/工作空间
在本教程的此部分中，您将在 {{site.data.keyword.conversationshort}} 服务中装入预定义的工作空间或技能。
1. 在 [{{site.data.keyword.Bluemix_short}} 资源列表](https://{DomainName}/resources)中，打开服务概述。找到在先前部分中创建的 {{site.data.keyword.conversationshort}} 服务实例。单击其条目，然后单击服务别名以打开服务详细信息。
2. 单击**启动工具**以访问 {{site.data.keyword.conversationshort}} 工具。
3. 切换到**技能**，然后单击**创建技能**，再单击**导入技能**。
4. 在对话框中，单击**选择 JSON 文件**后，从本地目录中选择 **assistant-skill.json** 文件。将导入选项保留为**全部内容（意向、实体和对话）**，然后单击**导入**。这将创建名为 **TutorialSlackbot** 的新技能。
5. 单击**对话**以查看对话节点。可以展开这些节点以查看结构，结构类似于下图所示。

   该对话具有多个节点，用于处理有关帮助的问题和简单的“谢谢”。**newEvent** 节点及其子节点收集必要的输入，然后调用操作以将新的事件记录插入到 Db2 中。

   **query events** 节点澄清事件是按标识还是按日期进行搜索。然后，**query events by shortname** 和 **query event by dates** 子节点会执行实际搜索并收集所需数据。

   **credential_node** 设置用于对话操作的私钥以及有关 Cloud Foundry 组织的信息。调用操作时需要后者。

  设置完所有内容后，后文将说明详细信息。
  ![](images/solution19/SlackBot_Dialog.png)   
6. 单击 **credential_node** 对话节点，通过单击**响应为：**右侧的“菜单”图标以打开 JSON 编辑器。

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   将 **org_space** 替换为早先检索到的已编码组织和空间信息。用“%40”替换所有“@”。接下来，将 **YOURSECRET** 更改为先前的实际私钥。再次单击该图标以关闭 JSON 编辑器。

## 创建助手并与 Slack 集成

现在，您将创建与先前技能关联的助手，并将其与 Slack 集成。 
1. 单击左上角的**技能**，然后选择**助手**。接下来，单击**创建助手**。
2. 在对话框中，填写 **TutorialAssistant** 作为名称，然后单击**创建助手**。在下一个屏幕上，选择**添加对话技能**。随后，选择**添加现有技能**，从列表中选取 **TutorialSlackbot**，并添加此项。
3. 添加技能后，单击**添加集成**，然后从**受管集成**列表中，选择 **Slack**。
4. 按照指示信息将聊天机器人与 Slack 集成。

## 测试 Slack 机器人并使其学习
打开 Slack 工作空间以试用聊天机器人。开始与机器人直接聊天。

1. 在消息传递表单中输入 **help**。机器人应该会提供一些指导信息。
2. 现在输入 **new event** 以开始收集新事件记录的数据。您将使用 {{site.data.keyword.conversationshort}} 槽来收集所有必要的输入。
3. 首先是事件标识或名称。引号是必需的。使用引号可允许输入更复杂的名称。请输入 **"Meetup: IBM Cloud"** 作为事件名称。事件名称会定义为基于模式的实体 **eventName**。它支持在开头和结尾使用不同类型的双引号。
4. 接下来是事件位置。输入基于[系统实体 **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details)。但限制是只能使用 {{site.data.keyword.conversationshort}} 认可的城市。请尝试使用 **Friedrichshafen** 这一城市。
5. 在下一步中将要求提供电子邮件地址或 Web 站点的 URI 等联系信息。首先是 **https://www.ibm.com/events**。您将对该字段使用基于模式的实体。
6. 接下来的问题是收集开始和结束日期和时间。将使用允许不同输入格式的 **sys-date** 和 **sys-time**。使用**下周四**作为开始日期，**下午 6 点**作为开始时间，使用下周四的确切日期和时间（例如，**2019-05-09** 和 **22:00**）作为结束日期和时间。
7. 最后，收集了所有数据后，系统将显示摘要，并调用实现为 {{site.data.keyword.openwhisk_short}} 操作的服务器操作，以将新记录插入到 Db2 中。随后，对话会切换到子节点，以通过除去上下文变量来清除处理环境。通过输入 **cancel**、**exit** 或类似操作，可以随时取消整个输入过程。在这种情况下，系统将确认用户选择，随后会清除环境。
  ![](images/solution19/SlackSampleChat.png)   

现在有了一些样本数据，可以进行搜索了。
1. 输入 **show event information**。接下来，会提问是要按标识还是按日期进行搜索。输入 **name**，然后输入下一个问题 **"Think 2019"**。现在，聊天机器人应该会显示有关该事件的信息。该对话有多个响应可供选择。
2. 通过将 {{site.data.keyword.conversationshort}} 作为后端，可以输入更复杂的短语，从而跳过对话的某些部分。使用 **show event by the name "Think 2019"** 作为输入。聊天机器人会直接返回相应事件记录。
3. 现在您要按日期进行搜索。搜索由一对日期进行定义，事件开始日期必须介于这对日期之间。将 **search conference by date in February 2019** 作为输入，结果应该同样为 **Think 2019** 事件。实体 **February** 会解释为两个日期 - 2 月 1 日和 2 月 28 日，从而为日期范围的开始和结束日期提供输入。[如果没有指定年份 2019，那么系统将识别未来的 2 月](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time)。 

执行了一些搜索并有了新的事件条目后，可以重新访问交谈历史记录，并改进未来的对话。请按照 [{{site.data.keyword.conversationshort}} 文档中有关**改进理解**](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro)的指示信息进行操作。


## 除去资源
{:removeresources}

在主目录中执行清除脚本会从 {{site.data.keyword.dashdbshort}} 中删除事件表，并从 {{site.data.keyword.openwhisk_short}} 中除去操作。开始修改和扩展代码时，此操作可能非常有用。清除脚本不会更改 {{site.data.keyword.conversationshort}} 工作空间或技能。   
```bash
sh cleanup.sh
```
{:codeblock}

在 [{{site.data.keyword.Bluemix_short}} 资源列表](https://{DomainName}/resources)中，打开服务概述。找到 {{site.data.keyword.conversationshort}} 服务实例，然后将其删除。

## 扩展教程
想要为本教程添加内容或更改本教程？下面是一些构想：
1. 添加搜索功能，例如通配符搜索或搜索事件持续时间 ("give me all events longer than 8 hours")。
2. 使用 {{site.data.keyword.databases-for-postgresql}}，而不使用 {{site.data.keyword.dashdbshort}}。
[此 Slack 机器人的 GitHub 存储库](https://github.com/IBM-Cloud/slack-chatbot-database-watson)教程已经有代码来支持 {{site.data.keyword.databases-for-postgresql}}。
3. 添加天气服务并检索事件日期和位置的预测数据。
4. 将事件数据导出为 iCalendar **.ics** 文件。
5. 通过添加其他集成，将聊天机器人连接到 Facebook Messenger。
6. 向输出添加交互式元素，例如按钮。      


## 相关内容
{:related}

下面是与本教程中涵盖的主题相关的其他信息的链接。

与聊天机器人相关的博客帖子：
* [Chatbots: Some tricks with slots in IBM Watson Conversation](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [Lively chatbots: Best Practices](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [Building chatbots: more tips and tricks](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

文档和 SDK：
* GitHub 存储库中的[在 IBM Watson Conversation 中处理变量的提示和技巧](https://github.com/IBM-Cloud/watson-conversation-variables)
* [{{site.data.keyword.openwhisk_short}} 文档](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* 文档：[IBM Knowledge Center 上有关 {{site.data.keyword.dashdbshort}} 的信息](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* 面向开发者的[免费 Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions)
* 文档：[ibm_db Node.js 驱动程序的 API 描述](https://github.com/ibmdb/node-ibm_db)
* [{{site.data.keyword.cloudantfull}} 文档](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
