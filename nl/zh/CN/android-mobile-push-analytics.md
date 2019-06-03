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

# 带推送通知的 Android 本机移动应用程序
{: #android-mobile-push-analytics}

了解如何轻松地在 {{site.data.keyword.Bluemix_notm}} 上快速创建带有 {{site.data.keyword.mobilepushshort}} 之类的高价值移动服务的本机 Android 应用程序。

此教程引导您创建移动入门模板应用程序，添加移动服务，设置客户机 SDK，将代码导入 Android Studio，然后进一步增强应用程序。

## 目标
{: #objectives}

* 创建带 {{site.data.keyword.mobilepushshort}} 服务的移动应用程序。
* 获取 FCM 凭证。
* 下载代码并完成所需的设置。
* 配置、发送和监视 {{site.data.keyword.mobilepushshort}}。

![](images/solution9/Architecture.png)

## 产品
{: #products}

本教程使用以下产品：
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 开始之前
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html)，用于导入和增强代码。
- Google 帐户，用于登录 Firebase 控制台以获取发送方标识和服务器 API 密钥。

## 从入门模板工具包创建 Android 移动应用程序
{: #get_code}
{{site.data.keyword.Bluemix_notm}} Mobile 仪表板允许您通过从入门模板工具包创建应用程序来快速跟踪您的移动应用程序开发过程。
1. 导航至 [Mobile 仪表板](https://{DomainName}/developer/mobile/dashboard)
2. 单击**入门模板工具包**和**创建应用程序**。![](images/solution9/mobile_dashboard.png)
3. 输入应用程序名称，这也可以是您的 Android 项目名称。
4. 选择 **Android** 作为平台并单击**创建**。

    ![](images/solution9/create_mobile_project.png)
5. 单击**添加资源** > 移动 > **推送通知**并选择您想要供应服务的位置、资源组以及**轻量**定价套餐。
6. 单击**创建**以供应 {{site.data.keyword.mobilepushshort}} 服务。在**应用程序**选项卡下将创建新的应用程序。

    **注：**应使用空入门模板添加 {{site.data.keyword.mobilepushshort}} 服务。在下一步中，您将获取 Firebase Cloud Messaging (FCM) 凭证。

在下一步中，您将下载有脚手架的代码，并设置 Push Android SDK。

## 下载代码并完成所需的设置
{: #download_code}

如果您尚未下载代码，那么通过单击“应用程序”>**您的移动应用程序**下的**下载代码**按钮来使用 {{site.data.keyword.Bluemix_notm}} Mobile 仪表板获取代码。
所下载的代码中包括 **{{site.data.keyword.mobilepushshort}}** 客户机 SDK。客户机 SDK 在 Gradle 和 Maven 上可用。对于本教程，您将使用 **Gradle**。

1. 启动 Android Studio > **打开现有 Android Studio 项目**，并指向下载的代码。
2. **Gradle** 构建将自动触发，并将下载所有依赖项。
3. 将 **Google Play 服务**依赖项添加到模块级别 `build.gradle (Module: app)` 文件末尾 `dependencies{.....}` 之后 
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. 复制您已创建并下载到 Android 应用程序模块根目录的 `google-services.json` 文件。请注意，`google-service.json` 文件包括添加的程序包名。
5. 所需的许可权都包含在 `AndroidManifest.xml` 文件和依赖项中。Push 和 Analytics 包含在 **build.gradle (Module: app)** 中。
6. `RECEIVE` 和 `REGISTRATION` 事件通知的 **Firebase Cloud Messaging (FCM)** 意向服务和意向过滤器包含在 `AndroidManifest.xml` 中

## 获取 FCM 凭证
{: #obtain_fcm_credentials}

Firebase Cloud Messaging (FCM) 是用于将 {{site.data.keyword.mobilepushshort}} 交付到 Android 设备、Google Chrome 浏览器以及 Chrome 应用程序和扩展的网关。要在控制台上设置 {{site.data.keyword.mobilepushshort}} 服务，您需要获取 FCM 凭证（发送方标识和 API 密钥）。

API 密钥由 {{site.data.keyword.mobilepushshort}} 服务安全地进行存储和使用以连接到 FCM 服务器，发送方标识（项目号）由 Android SDK 以及用于 Google Chrome 和 Mozilla Firefox 的 JS SDK 在客户机端使用。要设置 FCM 并获取凭证，请完成以下步骤：

1. 访问 [Firebase 控制台](https://console.firebase.google.com/?pli=1)  -  需要 Google 用户帐户。
2. 选择**添加项目**。
3. 在**创建项目**窗口中，提供项目名称，选择国家/地区并单击**创建项目**。
4. 在左侧的导航窗格中，选择**设置**（单击**概述**旁的“设置”图标）> **项目设置**。
5. 选择“云消息传递”选项卡，以获取项目凭证 -“服务器 API 密钥”和“发送方标识”。**注：**FCM 中列出的服务器密钥与服务器 API 密钥相同。![](images/solution9/fcm_console.png)

您还需要生成 `google-services.json` 文件。请完成以下步骤：

1. 在 Firebase 控制台中，单击上面创建的项目下的**项目设置**图标 > **常规**选项卡，并选择**将 Firebase 添加到您的 Android 应用程序**

    ![](images/solution9/firebase_project_settings.png)
2. 在**将 Firebase 添加到您的 Android** 应用程序模态窗口中，将 **com.ibm.mobilefirstplatform.clientsdk.android.push** 添加为软件包名称以注册 {{site.data.keyword.mobilepushshort}} Android SDK。应用程序昵称和 SHA-1 字段是可选的。单击**注册应用程序** > **继续** > **完成**。

    ![](images/solution9/add_firebase_to_your_app.png)

3. 单击**添加应用程序** > **将 Firebase 添加到您的应用程序**。通过输入软件包名称 **com.ibm.mysampleapp** 来包含应用程序的软件包名称，然后继续将 Firebase 添加您的 Android 应用程序窗口。应用程序昵称和 SHA-1 字段是可选的。单击**注册应用程序** > 继续 > 完成。**注：**下载代码后，您可以在 `AndroidManifest.xml` 文件中找到应用程序的软件包名称。
4. 在**您的应用程序**下，下载最新的配置文件 `google-services.json`。

    ![](images/solution9/google_services.png)

    **注**：FCM 是 Google Cloud Messaging (GCM) 的新版本。确保对新应用程序使用 FCM 凭证。现有应用程序将继续以 GCM 配置运作。

*步骤和 Firebase 控制台 UI 可能会有更改，如果需要，请参阅 Google 文档的 Firebase 部分*

## 配置、发送和监视 {{site.data.keyword.mobilepushshort}}

{: #configure_push}

1. {{site.data.keyword.mobilepushshort}} SDK 已经导入应用程序中，可以在 `MainActivity.java` 文件中找到 Push 初始化代码。

    **注：**服务凭证是 `/res/values/credentials.xml` 文件的一部分。
2. 通知注册在 `MainActivity.java` 中进行。（可选）提供唯一 USER_ID。
3. 在物理设备或仿真器上运行应用程序以接收通知。
4. 在 {{site.data.keyword.Bluemix_notm}} Mobile 仪表板的**移动服务** > **现有服务**下打开 {{site.data.keyword.mobilepushshort}} 服务，要发送基本的 {{site.data.keyword.mobilepushshort}}，请完成以下步骤：
   - 单击**管理** > **配置**。
   - 选择**移动**，然后使用您在 Firebase 控制台上初始创建的发送方标识/项目号和 API 密钥（服务器密钥）来更新“GCM/FCM 推送凭证”选项卡。

     ![](images/solution9/configure_push_notifications.png)
   - 单击**保存**。现在，{{site.data.keyword.mobilepushshort}} 服务已配置。
   - 选择**发送通知**，然后通过选择发送选项来编写消息。支持的选项包括：设备（按标记）、设备标识、用户标识、Android 设备、iOS 设备、Web 通知以及所有设备。**注**：选择**所有设备**选项时，预订了 {{site.data.keyword.mobilepushshort}} 的所有设备都会收到通知。
   - 在**消息**字段中，编写您的消息。根据需要，选择配置可选设置。
   - 单击**发送**，验证您的物理设备是否已收到通知。

     ![](images/solution9/android_send_notifications.png)
5. 您应该在 Android 设备上看到通知。

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. 您可以通过导航至 {{site.data.keyword.mobilepushshort}} 服务上的**监视**来监视发送的通知。IBM {{site.data.keyword.mobilepushshort}} 服务现在扩展了功能，可以通过用户数据生成图形，从而监视推送性能。您可以使用实用程序列出所有已发送的 {{site.data.keyword.mobilepushshort}}，或者列出所有已注册的设备，并每日、每周或每月分析信息。![](images/solution6/monitoring_messages.png)

## 相关内容
{: #related_content}
- [定制 {{site.data.keyword.mobilepushshort}} 设置](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [基于标记的通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}} 中的安全性](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

