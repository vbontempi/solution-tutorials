---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# 构建支持声音的 Android 聊天机器人
{: #android-watson-chatbot}

了解如何使用 {{site.data.keyword.Bluemix_short}} 上的 {{site.data.keyword.conversationshort}}、{{site.data.keyword.texttospeechshort}} 和 {{site.data.keyword.speechtotextshort}} 服务，轻而易举地快速创建支持声音的 Android 本机聊天机器人。

本教程将全程指导您完成定义意向和实体，以及为聊天机器人构建对话流以响应客户查询的过程。您将了解如何启用 {{site.data.keyword.speechtotextshort}} 和 {{site.data.keyword.texttospeechshort}} 服务以与 Android 应用程序进行轻松交互。
{:shortdesc}

## 目标
{: #objectives}

- 使用 {{site.data.keyword.conversationshort}} 定制和部署聊天机器人。
- 允许最终用户使用声音和音频与聊天机器人进行交互。
- 配置和运行 Android 应用程序。

## 使用的服务
{: #services}

本教程使用以下产品：

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## 体系结构
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* 用户使用自己的声音与移动应用程序进行交互。
* 音频通过 {{site.data.keyword.speechtotextfull}} 转录为文本。
* 文本传递到 {{site.data.keyword.conversationfull}}。
* 来自 {{site.data.keyword.conversationfull}} 的回复由 {{site.data.keyword.texttospeechfull}} 转换为音频，然后结果会发送回移动应用程序。

## 开始之前
{: #prereqs}

- 下载并安装 [Android Studio](https://developer.android.com/studio/index.html)。

## 创建服务
{: #setup}

在此部分中，您将创建教程所需的服务，首先创建 {{site.data.keyword.conversationshort}}，此服务用于构建可为客户提供帮助的认知虚拟助手。

1. 转至 [**{{site.data.keyword.Bluemix_notm}} 目录**](https://{DomainName}/catalog/)，然后选择 [{{site.data.keyword.conversationshort}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation) 服务 > **轻量**套餐：
   1. 将**名称**设置为 **android-chatbot-assistant**。
   1. **创建**。
2. 单击左侧窗格中的**服务凭证**，然后单击**新建凭证**。
   1. 将**名称**设置为 **for-android-app**。
   1. **添加**。
3. 单击**查看凭证**以查看凭证。记下 **API 密钥**和 **URL**，您将需要这些信息用于移动应用程序。

{{site.data.keyword.speechtotextshort}} 服务用于将人声转换为书面文字，然后可以将其作为输入发送到 {{site.data.keyword.Bluemix_short}} 上的 {{site.data.keyword.conversationshort}} 服务。

1. 转至 [**{{site.data.keyword.Bluemix_notm}} 目录**](https://{DomainName}/catalog/)，然后选择 [{{site.data.keyword.speechtotextshort}}](https://{DomainName}/catalog/services/speech-to-text) 服务 > **轻量**套餐。
   1. 将**名称**设置为 **android-chatbot-stt**。
   1. **创建**。
2. 单击左侧窗格中的**服务凭证**，然后单击**新建凭证**以添加新凭证。
   1. 将**名称**设置为 **for-android-app**。
   1. **添加**。
3. 单击**查看凭证**以查看凭证。记下 **API 密钥**和 **URL**，您将需要这些信息用于移动应用程序。

{{site.data.keyword.texttospeechshort}} 服务用于处理文本和自然语言，以生成具有相应节奏和语调的合成音频输出。此服务提供多种声音，可以在 Android 应用程序中进行配置。

1. 转至 [**{{site.data.keyword.Bluemix_notm}} 目录**](https://{DomainName}/catalog/)，然后选择 [{{site.data.keyword.texttospeechshort}}](https://{DomainName}/catalog/services/text-to-speech) 服务 > **轻量**套餐。
   1. 将**名称**设置为 **android-chatbot-tts**。
   1. **创建**。
2. 单击左侧窗格中的**服务凭证**，然后单击**新建凭证**以添加新凭证。
   1. 将**名称**设置为 **for-android-app**。
   1. **添加**。
3. 单击**查看凭证**以查看凭证。记下 **API 密钥**和 **URL**，您将需要这些信息用于移动应用程序。

## 创建技能
{: #create_workspace}

技能是一种容器，用于包含定义会话流的工件。

在本教程中，您将保存并使用 [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) 文件，其中包含预定义的意向、实体和与机器的对话流。

1. 在 {{site.data.keyword.conversationshort}} 服务详细信息页面中，导航至左侧窗格中的**管理**，单击**启动工具**以查看 {{site.data.keyword.conversationshort}} 仪表板。
1. 单击**技能**选项卡。
1. 单击**新建**，然后单击**导入技能**，并选择上面下载的 JSON 文件。
1. 选择**全部内容**选项，然后单击**导入**。这将使用预定义的意向、实体和对话流创建新技能。
1. 返回到“技能”列表。选择 `Ana` 技能上的操作菜单以**查看 API 详细信息**。

### 定义意向
{:#define_intent}

意向表示用户输入的目的，例如回答问题或处理帐单支付。您可为希望应用程序支持的每种类型的用户请求定义一个意向。通过识别用户输入中表达的意向，{{site.data.keyword.conversationshort}} 服务可以选择正确的对话流来进行响应。在工具中，意向的名称始终以 `#` 字符为前缀。

简单来说，意向就是最终用户的意图。下面是意向名称的示例。
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. 单击新创建的技能 - **Ana**。

   Ana 是一个保险机器人，用户可以通过该机器人来查询自己的健康福利和提出理赔。
{:tip}
2. 单击第一个选项卡来查看所有**意向**。
3. 单击**添加意向**来创建新意向。在 `#` 后输入 `Cancel_Policy` 作为意向名称，并提供可选的描述。
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. 单击**创建意向**。
5. 添加请求取消保单的用户示例
   - `我要取消保单`
   - `立即将我的保单作废`
   - `我想停止为保单付费。`
6. 逐个添加上述用户示例，然后单击**添加示例**。对于所有其他用户示例，重复此步骤。

   请记住至少添加 5 个用户示例，以便更好地训练机器人。
   {:tip}

7. 单击意向名称旁边的**关闭** ![](images/solution28-watson-chatbot-android/close_icon.png) 按钮以保存意向。
8. 单击**内容目录**，然后选择**常规**。单击**添加到技能**。

   内容目录通过添加现有意向（银行、客户服务、保险、电话公司、电子商务等），可帮助您更快入门。这些意向已基于用户可能会问的常见问题进行了训练。
{:tip}

### 定义实体
{:#define_entity}

实体表示与意向相关，并为意向提供特定上下文的术语或对象。您可为每个实体和用户可能输入的同义词列出可能的值。通过识别用户输入中提及的实体，{{site.data.keyword.conversationshort}} 服务可以选择执行特定操作来实现意向。在工具中，实体的名称始终以 `@` 字符为前缀。

下面是实体名称的示例。
 - `@location`
 - `@menu_item`
 - `@product`

1. 单击**实体**选项卡来查看现有实体。
2. 单击**添加实体**，然后在 `@` 后面输入实体的名称 `location`。单击**创建实体**。
3. 输入 `address` 作为值名称，然后选择**同义词**。
4. 添加 `place` 作为同义词，然后单击 ![](images/solution28-watson-chatbot-android/plus_icon.png) 图标。对同义词 `office`、`centre`、`branch` 等重复此操作，然后单击**添加值**。
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. 单击**关闭** ![](images/solution28-watson-chatbot-android/close_icon.png) 以保存更改。
6. 单击**系统实体**选项卡来查看 IBM 创建的可在任何用例中使用的公共实体。

   可以使用系统实体来识别它们所表示对象类型的范围广泛的一系列值。例如，`@sys-number` 系统实体与任何数字值匹配，包括整数、十进制小数，甚至包括以文字形式写出的数字。
{:tip}
7. 对于 @sys-person 和 @sys-location 系统实体，将**状态**从“关闭”切换到`开启`。

### 构建对话流
{:#build_dialog}

对话是一种分支会话流，用于定义应用程序在识别到定义的意向和实体时如何进行响应。使用工具中的对话构建器可创建与用户的会话，并根据在其输入中识别到的意向和实体来提供响应。

1. 单击**对话**选项卡来查看具有意向和实体的现有对话流。
2. 单击**添加节点**向对话添加新节点。
3. 在**如果助手识别到**下，输入 `#Cancel_Policy`。
4. 在**那么响应为**下，输入响应：`此机构未提供在线服务。请到我们最近的分支机构取消您的保单。`
5. 单击 ![](images/solution28-watson-chatbot-android/save_node.png) 以关闭并保存该节点。
6. 滚动至看到 `#greeting` 节点。单击该节点来查看详细信息。
   ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. 单击 ![](images/solution28-watson-chatbot-android/add_condition.png) 图标来**添加新条件**。从下拉列表中选择`或`，然后输入 `#General_Greetings` 作为意向。**那么响应为**部分会显示助手在用户问候时的响应。
   ![](images/solution28-watson-chatbot-android/apply_condition.png)

   上下文变量是在节点中定义的变量，可选择为其指定缺省值。其他节点或应用程序逻辑可以随后设置或更改上下文变量的值。应用程序可以将信息传递到对话，对话可以更新此信息，然后将其传递回应用程序，或者传递到后续节点。对话使用上下文变量来执行此操作。
   {:tip}

8. 使用**试用**按钮来测试对话流。

## 将技能链接到助手

**助手**是一种认知机器人，您可以根据自己的业务需要进行定制，并跨多个渠道进行部署，以随时随地根据客户需要向客户提供帮助。可以通过向助手添加满足客户目标所需的**技能**来定制助手。

1. 在 {{site.data.keyword.conversationshort}} 工具中，切换到**助手**，然后使用**新建**。
   1. 将**名称**设置为 **android-chatbot-assistant**。
   1. **创建**
1. 使用**添加对话技能**以选择先前部分中创建的技能。
   1. **添加现有技能**
   1. 选择 **Ana**
1. 选择“助手”> **设置** > **API 详细信息**中的操作菜单，记下**助手标识**，在移动应用程序中需要引用此标识（在 Android 应用程序的 `config.xml` 文件中）。

## 配置和运行 Android 应用程序
{:#configure_run_android_app}

存储库包含具有必需的 Gradle 依赖项的 Android 应用程序代码。

1. 运行以下命令来克隆 [GitHub 存储库](https://github.com/IBM-Cloud/chatbot-watson-android)：
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. 启动 Android Studio > **打开现有 Android Studio 项目**，然后指向已下载的代码。这将自动触发 **Gradle** 构建，并下载所有依赖项。
3. 打开 `app/src/main/res/values/config.xml` 以查看服务凭证的占位符 (`ASSISTANT_ID_HERE`)。在相应的占位符中输入服务凭证（早先保存的服务凭证），然后保存该文件。
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <!--Watson Assistant service credentials-->
       <!-- REPLACE `ASSISTANT_ID_HERE` with ID of the Assistant to use -->
       <string name="assistant_id">ASSISTANT_ID_HERE</string>

       <!-- REPLACE `ASSISTANT_API_KEY_HERE` with Watson Assistant service API Key-->
       <string name="assistant_apikey">ASSISTANT_API_KEY_HERE</string>

       <!-- REPLACE `ASSISTANT_URL_HERE` with Watson Assistant service URL-->
       <string name="assistant_url">ASSISTANT_URL_HERE</string>

       <!--Watson Speech To Text(STT) service credentials-->
       <!-- REPLACE `STT_API_KEY_HERE` with Watson Speech to Text service API Key-->
       <string name="STT_apikey">STT_API_KEY_HERE</string>

       <!-- REPLACE `STT_URL_HERE` with Watson Speech to Text service URL-->
       <string name="STT_url">STT_URL_HERE</string>

       <!--Watson Text To Speech(TTS) service credentials-->
       <!-- REPLACE `TTS_API_KEY_HERE` with Watson Text to Speech service API Key-->
       <string name="TTS_apikey">TTS_API_KEY_HERE</string>

       <!-- REPLACE `TTS_URL_HERE` with Watson Text to Speech service URL-->
       <string name="TTS_url">TTS_URL_HERE</string>
   </resources>
   ```
4. 构建项目并在真实设备上或使用模拟器启动应用程序。
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. 在下面提供的空白处**输入查询**，然后单击箭头图标以将查询发送到 {{site.data.keyword.conversationshort}} 服务。
6. 响应将传递到 {{site.data.keyword.texttospeechshort}} 服务，并且您应该会听到有声音在读出响应。
7. 单击应用程序左下角的**麦克风**图标，以输入转换为文本的语音，然后单击箭头图标将其发送到 {{site.data.keyword.conversationshort}} 服务。


## 除去资源
{:removeresources}

1. 导航至[资源列表](https://{DomainName}/resources/)。
1. 删除已创建的服务：
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## 相关内容
{:related}

- [创建实体、同义词和系统实体](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [上下文变量](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [构建复杂对话](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [使用槽收集信息](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [部署选项](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [改进技能](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
