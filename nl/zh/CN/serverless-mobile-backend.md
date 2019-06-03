---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: data-hd-operatingsystem="ios"}
{:android: data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 使用无服务器后端的移动应用程序
{: #serverless-mobile-backend}

在本教程中，您将学习如何将 {{site.data.keyword.openwhisk}} 与认知和数据服务一起使用，为移动应用程序构建无服务器后端。
{:shortdesc}

并非所有移动开发者一开始都有管理服务器端逻辑或服务器的经验。他们更愿意将精力集中在所构建的应用程序上。现在，如果他们可以复用自己现有的开发技能来编写移动后端会怎样？

{{site.data.keyword.openwhisk_short}} 是一种无服务器事件驱动型平台。如[此示例中重点阐述](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp)的那样，部署的操作可以轻松转换为 HTTP 端点，作为 *Web 操作*来构建 Web 应用程序后端 API。通过将 Web 应用程序作为 REST API 客户机，可以轻松地进一步扩展此示例，以应用相同的方法为移动应用程序构建后端。借助 {{site.data.keyword.openwhisk_short}}，移动开发者在编写操作时可以使用用于编写其移动应用程序的语言 - 对于 Android，为 Java；对于 iOS，为 Swift。

本教程可根据目标平台进行配置。您目前查看的是本教程的 **iOS / Swift** 版本的文档。使用本文档顶部的选项卡可选择本教程的 **Android / Java** 版本。
{: swift}

本教程可根据目标平台进行配置。您目前查看的是本教程的 **Android / Java** 版本的文档。使用本文档顶部的选项卡可选择本教程的 **iOS / Swift** 版本。
{: java}

## 目标
{: #objectives}

* 通过 {{site.data.keyword.openwhisk_short}} 部署无服务器移动后端。
* 通过 {{site.data.keyword.appid_short}} 将用户认证添加到移动应用程序。
* 通过 {{site.data.keyword.toneanalyzershort}} 分析用户反馈。
* 通过 {{site.data.keyword.mobilepushshort}} 发送通知。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

本教程中显示的应用程序是一个反馈应用程序，通过 {{site.data.keyword.mobilepushshort}} 智能地分析反馈文本的语气，并使客户获得相应的了解。

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. 用户通过 [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) 进行认证。{{site.data.keyword.appid_short}} 提供了访问令牌和标识令牌。
2. 对后端 API 的进一步调用包含访问令牌。
3. 后端通过 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) 实现。作为 Web 操作公开的无服务器操作需要在请求头中发送令牌并验证其有效性（签名和到期日期），然后才允许访问实际 API。
4. 用户提交反馈时，反馈将存储在 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 中。
5. 反馈文本通过 [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) 进行处理。
6. 根据分析结果，通过 [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) 将通知发回用户。
7. 用户在设备上收到通知。

## 开始之前
{: #prereqs}

本教程使用 {{site.data.keyword.Bluemix_notm}} 命令行工具来供应资源和部署代码。确保安装 `ibmcloud` 命令行工具。

* [{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - 用于安装 ibmcloud CLI 和必需插件（Cloud Foundry 和 {{site.data.keyword.openwhisk_short}}）的脚本

此外，您还需要以下软件和帐户：

   1. Java 8
   2. Android Studio 2.3.3
   3. Google 开发者帐户，用于配置 Firebase 云消息传递
   4. Bash shell 和 cURL
   {: java}


   1. Xcode
   2. Apple 开发者帐户，用于配置 Apple 推送通知服务
   3. Bash shell 和 cURL
   {: swift}

在本教程中，您将为应用程序配置推送通知。本教程假定您已完成 [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) 或 [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics) 的基本 {{site.data.keyword.mobilepushshort}} 教程，并且您熟悉 Firebase 云消息传递或 Apple 推送通知服务的配置。
{:tip}

要让 Windows 10 用户使用命令行指示信息，我们建议如[此文章](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10)中所述，安装适用于 Linux 和 Ubuntu 的 Windows 子系统。
{: tip}
{: java}

## 获取应用程序代码

存储库包含移动应用程序和 {{site.data.keyword.openwhisk_short}} 操作代码。

1. 从 GitHub 存储库中检出代码

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-android
   ```
   {: pre}
   {: java}

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-ios
   ```
   {: pre}
   {: swift}

2. 查看代码结构

|File|描述|
| ---------------------------------------- | ---------------------------------------- |
|[**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions)|用于无服务器移动后端的 {{site.data.keyword.openwhisk_short}} 操作的代码|
|[**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android)|用于移动应用程序的代码|
|[**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh)|用于安装、卸载和更新 {{site.data.keyword.openwhisk_short}} 触发器、操作和规则的辅助脚本|
{: java}

|File|描述|
| ---------------------------------------- | ---------------------------------------- |
|[**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions)|用于无服务器移动后端的 {{site.data.keyword.openwhisk_short}} 操作的代码|
|[**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp)|用于移动应用程序的代码|
|[**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh)|用于安装、卸载和更新 {{site.data.keyword.openwhisk_short}} 触发器、操作和规则的辅助脚本|
{: swift}

## 供应服务以处理用户认证、反馈持久性和分析
{: #provision_services}

在此部分中，您将供应应用程序使用的服务。可以选择通过 {{site.data.keyword.Bluemix_notm}}“目录”或使用 `ibmcloud` 命令行来供应服务。

建议创建新空间来供应服务并部署无服务器后端。这有助于将所有资源保存在一起。

### 通过 {{site.data.keyword.Bluemix_notm}} 目录供应服务

1. 转至 [{{site.data.keyword.Bluemix_notm}}“目录”](https://{DomainName}/catalog/)。
2. 使用**轻量**套餐创建 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 服务。将“名称”设置为 **serverlessfollowup-db**。
3. 使用**标准**套餐创建 [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) 服务。将“名称”设置为 **serverlessfollowup-tone**。
4. 使用**累进层**套餐创建 [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) 服务。将“名称”设置为 **serverlessfollowup-appid**。
5. 使用**轻量**套餐创建 [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) 服务。将“名称”设置为 **serverlessfollowup-mobilepush**。

### 通过命令行供应服务

通过命令行，运行以下命令来供应服务并检索其凭证：

   ```sh
   ibmcloud cf create-service cloudantNoSQLDB Lite serverlessfollowup-db
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service tone_analyzer standard serverlessfollowup-tone
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service AppID "Graduated tier" serverlessfollowup-appid
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service imfpush lite serverlessfollowup-mobilepush
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

## 配置 {{site.data.keyword.mobilepushshort}}
{: #push_notifications}

用户提交新的反馈时，应用程序将分析此反馈，并将通知发回用户。用户可能已经移至其他任务，或者可能没有启动移动应用程序，因此使用 Push Notifications 是与用户进行通信的一种好办法。利用 {{site.data.keyword.mobilepushshort}} 服务，可以通过一个统一的 API 向 iOS 或 Android 用户发送通知。在此部分中，您将为目标平台配置 {{site.data.keyword.mobilepushshort}} 服务。

### 配置 Firebase 云消息传递 (FCM)
{: java}

   1. 在 [Firebase 控制台](https://console.firebase.google.com)中，创建新项目。将“名称”设置为 **serverlessfollowup**。
   2. 导航至项目**设置**。
   3. 在**常规**选项卡下，添加两个应用程序：
      1. 一个应用程序的软件包名称设置为：**com.ibm.mobilefirstplatform.clientsdk.android.push**
      2. 一个应用程序的软件包名称设置为：**serverlessfollowup.app**
   4. 从 Firebase 控制台下载包含两个已定义应用程序的 `google-services.json`，然后将此文件放入检出目录的 `android/app` 文件夹中。
   5. 在**云消息传递**选项卡下，找到“发件人标识”和“服务器密钥”（后文中也称为“API 密钥”）。
   6. 在 Push Notifications 服务仪表板中，设置“发件人标识”和“API 密钥”的值。
   {: java}

### 配置 Apple 推送通知服务 (APNs)
{: swift}

1. 转至 [Apple Developer](https://developer.apple.com/) 门户网站，并注册 App ID。
2. 创建开发和分发 APNs SSL 证书。
3. 创建开发供应概要文件。
4. 在 {{site.data.keyword.Bluemix_notm}} 上配置 {{site.data.keyword.mobilepushshort}} 服务实例。有关详细步骤，请参阅[获取 APNs 凭证和配置 {{site.data.keyword.mobilepushshort}} 服务](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-)。
{: swift}

## 部署无服务器后端
{: #serverless_backend}

配置完所有服务后，现在可以部署无服务器后端。在此部分中，将创建以下 {{site.data.keyword.openwhisk_short}} 工件：

|工件|类型|描述|
| ----------------------------- | ------------------------------ | ---------------------------------------- |
|`serverlessfollowup`|包|用于对操作进行分组并保留所有服务凭证的包|
|`serverlessfollowup-cloudant`|包绑定|绑定到内置 {{site.data.keyword.cloudant_short_notm}} 包|
|`serverlessfollowup-push`|包绑定|绑定到 {{site.data.keyword.mobilepushshort}} 包|
|`auth-validate`|操作|验证访问令牌和标识令牌|
|`users-add`|操作|持久存储用户信息（标识、姓名、电子邮件、照片、设备标识）|
|`users-prepare-notify`|操作|设置消息格式以用于 {{site.data.keyword.mobilepushshort}} |
|`feedback-put`|操作|将用户反馈存储在数据库中|
|`feedback-analyze`|操作|通过 {{site.data.keyword.toneanalyzershort}} 分析反馈|
|`users-add-sequence`|公开为 Web 操作的序列|`auth-validate` 和 `users-add`|
|`feedback-put-sequence`|公开为 Web 操作的序列|`auth-validate` 和 `feedback-put`|
|`feedback-analyze-sequence`|序列|通过 {{site.data.keyword.cloudant_short_notm}} 执行 `read-document`，然后通过 {{site.data.keyword.mobilepushshort}} 执行 `feedback-analyze`、`users-prepare-notify` 和 `sendMessage`|
|`feedback-analyze-trigger`|触发器|反馈存储在数据库中时由 {{site.data.keyword.openwhisk_short}} 调用|
|`feedback-analyze-rule`|规则|将 `feedback-analyze-trigger` 触发器与 `feedback-analyze-sequence` 序列链接在一起|

### 编译代码
{: java}
1. 从检出目录的根目录，编译操作代码
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### 配置并部署操作
{: java}

2. 将 template.local.env 复制到 local.env

   ```sh
   cp template.local.env local.env
   ```
3. 在 {{site.data.keyword.Bluemix_notm}} 仪表板中（或在先前运行的 ibmcloud 命令的输出中），获取 {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.toneanalyzershort}}、{{site.data.keyword.mobilepushshort}} 和 {{site.data.keyword.appid_short}} 服务的凭证，然后将 `local.env` 中的占位符替换为相应值。这些属性将注入到包中，以便所有操作都可以访问数据库。
4. 将操作部署到 {{site.data.keyword.openwhisk_short}}。`deploy.sh` 从 `local.env` 装入凭证，以创建 {{site.data.keyword.cloudant_short_notm}} 数据库（users、feedback 和 moods）并部署应用程序的 {{site.data.keyword.openwhisk_short}} 工件。
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   完成本教程后，可以使用 `./deploy.sh --uninstall` 来除去 {{site.data.keyword.openwhisk_short}} 工件。
   {: tip}

### 配置并部署操作
{: swift}

1. 将 template.local.env 复制到 local.env
   ```sh
   cp template.local.env local.env
   ```
2. 在 {{site.data.keyword.Bluemix_notm}} 仪表板中（或在先前运行的 ibmcloud 命令的输出中），获取 {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.toneanalyzershort}}、{{site.data.keyword.mobilepushshort}} 和 {{site.data.keyword.appid_short}} 服务的凭证，然后将 `local.env` 中的占位符替换为相应值。这些属性将注入到包中，以便所有操作都可以访问数据库。
3. 将操作部署到 {{site.data.keyword.openwhisk_short}}。`deploy.sh` 从 `local.env` 装入凭证，以创建 {{site.data.keyword.cloudant_short_notm}} 数据库（users、feedback 和 moods）并部署应用程序的 {{site.data.keyword.openwhisk_short}} 工件。

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   完成本教程后，可以使用 `./deploy.sh --uninstall` 来除去 {{site.data.keyword.openwhisk_short}} 工件。
   {: tip}

## 配置并运行本机移动应用程序以收集用户反馈
{: #mobile_app}

{{site.data.keyword.openwhisk_short}} 操作已准备好用于移动应用程序。在运行移动应用程序之前，您需要配置其设置，以将您创建的服务设定为目标。

1. 通过 Android Studio，打开位于检出目录的 `android` 文件夹中的项目。
2. 编辑 `android/app/src/main/res/values/credentials.xml`，并使用凭证中的值填写空白设置：您将需要 {{site.data.keyword.appid_short}} `tenantId`、{{site.data.keyword.mobilepushshort}} `appGuid` 和 `clientSecret`，以及在其中部署了 {{site.data.keyword.openwhisk_short}} 的组织和空间的名称。对于 API 主机，启动命令提示符或终端，并运行以下命令：
   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
3. 打开 `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` 和 `LoginAndRegistrationListener.java`，并根据在其中创建服务实例的位置，更新推送通知服务 (BMSClient) 区域和 AppID 区域。
4. 构建项目。
5. 在真实设备上或使用仿真器启动应用程序。
   要使仿真器接收推送通知，请确保使用 Google API 来选取图像，然后在仿真器中使用 Google 帐户登录。
   {: tip}
6. 在后台监控 {{site.data.keyword.openwhisk_short}}
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. 在应用程序中，选择**登录**以使用 Facebook 或 Google 帐户进行认证。登录后，输入反馈消息，然后按**发送反馈**按钮。发送反馈后几秒钟内，您应该会在设备上收到推送通知。通知文本可以通过修改 {{site.data.keyword.cloudant_short_notm}} 服务实例中 `moods` 数据库中的模板文档进行定制。使用**查看令牌**按钮可检查登录时 {{site.data.keyword.appid_short}} 生成的访问令牌和标识令牌。
{: java}


1. Push 客户机 SDK 和其他 SDK 在 CocoaPods 和 Carthage 上可用。对于此解决方案，将使用 CocoaPods。
2. 打开终端，并通过 `cd ` 进入 `followupapp` 文件夹。运行以下命令安装必需的依赖项。
   ```sh
   pod install
   ```
   {: pre}
3. 打开位于检出目录的 `followupapp` 文件夹下扩展名为 `.xcworkspace` 的文件，以在 Xcode 中启动代码。
4. 编辑 `BMSCredentials.plist` 文件，并使用凭证中的值填写空白设置。您将需要 {{site.data.keyword.appid_short}} `tenantId`、{{site.data.keyword.mobilepushshort}} `appGuid` 和 `clientSecret`，以及在其中部署了 {{site.data.keyword.openwhisk_short}} 的组织和空间的名称。对于 API 主机，启动终端并运行以下命令：

   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
5. 打开 `AppDelegate.swift`，并根据在其中创建服务实例的位置，更新推送通知服务 (BMSClient) 区域和 AppID 区域。
6. 构建项目。
7. 在真实设备上或使用仿真器启动应用程序。
8. 通过在终端上运行以下命令，在后台监控 {{site.data.keyword.openwhisk_short}}。
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. 在应用程序中，选择**登录**以使用 Facebook 或 Google 帐户进行认证。登录后，输入反馈消息，然后按**发送反馈**按钮。发送反馈后几秒钟内，您应该会在设备上收到推送通知。通知文本可以通过修改 {{site.data.keyword.cloudant_short_notm}} 服务实例中 `moods` 数据库中的模板文档进行定制。使用**查看令牌**按钮可检查登录时 {{site.data.keyword.appid_short}} 生成的访问令牌和标识令牌。
{: swift}

## 除去资源

1. 使用 `deploy.sh` 来除去 {{site.data.keyword.openwhisk_short}} 工件：

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. 通过 {{site.data.keyword.Bluemix_notm}} 控制台，删除 {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.appid_short}}、{{site.data.keyword.mobilepushshort}} 和 {{site.data.keyword.toneanalyzershort}} 服务。

## 相关内容

* {{site.data.keyword.appid_short}} 提供了缺省配置，可帮助初始设置身份提供者。在发布应用程序之前，请[将配置更新为您自己的凭证](https://{DomainName}/docs/services/appid?topic=appid-social#social)。您还可以[定制登录窗口小部件](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget)。


* 使用 Swift 源文件（`actions` 文件夹下的 .swift 文件）创建 OpenWhisk Swift 操作时，必须将其编译为二进制文件后，才能运行该操作。 完成后，对该操作的后续调用会快得多，直到清除保存该操作的容器为止。此延迟称为冷启动延迟。
  要避免冷启动延迟，可以将 Swift 文件编译为二进制文件，然后将其以 zip 文件格式上传到 OpenWhisk。由于您需要 OpenWhisk 脚手架，因此创建二进制文件的最简单方法是在其运行环境中进行构建。有关进一步的步骤，请参阅[将操作打包为 Swift 可执行文件](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions)。
{: swift}
