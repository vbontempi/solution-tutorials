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

# 使用 Push Notifications 的混合移动应用程序
{: #hybrid-mobile-push-analytics}

了解如何轻而易举地快速创建使用 {{site.data.keyword.Bluemix_notm}} 上高价值移动服务（如 {{site.data.keyword.mobilepushshort}}）的混合 Cordova 应用程序。

Apache Cordova 是一种开放式源代码移动开发框架。此框架支持将多种标准 Web 技术（HTML5、CSS3 和 JavaScript）用于跨平台开发。应用程序在针对每个平台的包装器内执行，并依赖符合标准的 API 绑定来访问每个设备的功能，如传感器、数据、网络状态等。

本教程将全程指导您创建 Cordova 移动入门模板应用程序，添加移动服务，设置客户机 SDK，下载脚手架代码，然后进一步增强应用程序。

## 目标

* 创建使用 {{site.data.keyword.mobilepushshort}} 服务的移动项目。
* 了解如何获取 APN 和 FCM 凭证。
* 下载代码并完成所需的设置。
* 配置、发送和监视 {{site.data.keyword.mobilepushshort}}。

 ![](images/solution15/Architecture.png)

## 产品

本教程使用以下产品：
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 开始之前
{: #prereqs}

- Cordova [CLI](https://cordova.apache.org/docs/en/latest/guide/cli/)，用于执行 Cordova 命令。
- Cordova-iOS [必备软件](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html)和 Cordova-Android [必备软件](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html)
- Google 帐户，用于登录 Firebase 控制台以获取发送方标识和服务器 API 密钥。
- [Apple Developer](https://developer.apple.com/) 帐户，用于从 {{site.data.keyword.Bluemix_notm}} 上的 {{site.data.keyword.mobilepushshort}} 服务实例（提供者）将远程通知发送到 iOS 设备和应用程序。
- Xcode 和 Android Studio，用于导入代码和进一步增强代码。

## 通过入门模板工具包创建 Cordova 移动项目
{: #get_code}
在 {{site.data.keyword.Bluemix_notm}} Mobile 仪表板中，可以通过入门模板工具包来创建项目，从而快速跟踪移动应用程序开发。
1. 导航至 [Mobile 仪表板](https://{DomainName}/developer/mobile/dashboard)。
2. 单击**入门模板工具包**，然后单击**创建应用程序**。
    ![](images/solution15/mobile_dashboard.png)
3. 输入项目名称，这也可以是应用程序名称。
4. 选择 **Cordova** 作为平台，然后单击**创建**。

    ![](images/solution15/create_cordova_project.png)
5. 单击**添加资源** > 移动 > **推送通知**并选择您想要供应服务的位置、资源组以及**轻量**定价套餐。
6. 单击**创建**以供应 {{site.data.keyword.mobilepushshort}} 服务。在**应用程序**选项卡下将创建新的应用程序。

    **注：** {{site.data.keyword.mobilepushshort}} 服务应该已添加到“空入门模板”。

在下一步中，您将下载脚手架代码并完成所需的设置。

## 下载代码并完成所需的设置
{: #download_code}

如果尚未下载代码，请使用 {{site.data.keyword.Bluemix_notm}} Mobile 仪表板通过单击“项目”> **您的移动项目**下的**下载代码**按钮来获取代码。

1. 在所选的 IDE 中，导航至 `/platforms/android/project.properties`，并将最后两行（library.1 和 library.2）替换为以下各行：

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
  上面的更改是特定于 Android 的。
  {: tip}
2. 要在 Android 仿真器上启动应用程序，请在终端或命令提示符中运行以下命令：
```
 $ cordova build android
 $ cordova run android
```
 如果看到错误，请启动仿真器并尝试运行上面的命令。
 {: tip}
3. 要在 iOS 模拟器中预览应用程序，请在终端中运行以下命令：

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    可以通过运行 `cordova info` 命令在 `config.xml` 文件中查找您的项目名称。
    {: tip}

## 获取 FCM 和 APNs 凭证
{: #obtain_fcm_apns_credentials}

 ### 配置 Firebase 云消息传递 (FCM)

  1. 在 [Firebase 控制台](https://console.firebase.google.com)中，创建新项目。将“名称”设置为 **hybridmobileapp**。
  2. 导航至项目**设置**。
  3. 在**常规**选项卡下，添加两个应用程序：
       1. 一个应用程序的软件包名称设置为：**com.ibm.mobilefirstplatform.clientsdk.android.push**
       2. 一个应用程序的软件包名称设置为：**io.cordova.hellocordovastarter**
  4. 在**云消息传递**选项卡下，找到“发件人标识”和“服务器密钥”（后文中也称为“API 密钥”）。
  5. 在 {{site.data.keyword.mobilepushshort}} 服务仪表板中，设置“发件人标识”和“API 密钥”的值。


有关详细步骤，请参阅[获取 FCM 凭证](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials)。
{: tip}

### 配置 Apple {{site.data.keyword.mobilepushshort}} 服务 (APNs)

  1. 转至 [Apple Developer](https://developer.apple.com/) 门户网站，并注册 App ID。
  2. 创建开发和分发 APNs SSL 证书。
  3. 创建开发供应概要文件。
  4. 在 {{site.data.keyword.Bluemix_notm}} 上配置 {{site.data.keyword.mobilepushshort}} 服务实例。

有关详细步骤，请参阅[获取 APNs 凭证和配置 {{site.data.keyword.mobilepushshort}} 服务](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials)。
{: tip}

## 配置、发送和监视 {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. 在 index.js 中的 `onDeviceReady` 函数下，将 `{pushAppGuid}` 和

   `{pushClientSecret}` 值替换为 Push 服务**凭证** - *appGuid* 和 *clientSecret*。

2. 转至 Mobile 仪表板 >“项目”>“Cordova 项目”，单击 {{site.data.keyword.mobilepushshort}} 服务，然后执行以下步骤。

### APNs - 配置服务实例

要使用 {{site.data.keyword.mobilepushshort}} 服务来发送通知，请上传在上面的步骤中已创建的 .p12 证书。此证书包含构建和发布应用程序所需的专用密钥和 SSL 证书。

**注**：在 `.cer` 文件出现在“钥匙串访问”中之后，请将其导出到您的计算机，以创建 `.p12` 证书。

1. 单击“服务”部分下的 `{{site.data.keyword.mobilepushshort}}`，或单击 {{site.data.keyword.mobilepushshort}} 服务旁边的三个垂直点，然后选择`打开仪表板`。
2. 在 {{site.data.keyword.mobilepushshort}} 仪表板上，您应该会在`管理 > 发送通知`下看到`配置`选项。

要在 `Push Notification 服务`控制台上设置 APNs，请完成以下步骤：

1. 在“Push Notification 服务仪表板”中，选择`配置`。
2. 选择`移动`选项，以更新“APNs 推送凭证”表单上的信息。
3. 根据需要选择`沙箱（开发）`或`生产（分发）`，然后上传已创建的 `p.12` 证书。
4. 在“密码”字段中，输入与 .p12 证书文件关联的密码，然后单击“保存”。

### FCM - 配置服务实例

1. 选择**移动**，然后使用您在 Firebase 控制台上初始创建的发送方标识/项目号和 API 密钥（服务器密钥）来更新“GCM/FCM 推送凭证”选项卡。
2. 单击**保存**。现在，{{site.data.keyword.mobilepushshort}} 服务已配置。

### 发送 {{site.data.keyword.mobilepushshort}}

1. 选择**发送通知**，然后通过选择发送选项来编写消息。支持的选项包括“带标记的设备”、“设备标识”、“用户标识”、“Android 设备”、“IOS 设备”、“Web 通知”和“所有设备”。
**注**：选择**所有设备**选项时，预订了 {{site.data.keyword.mobilepushshort}} 的所有设备都会收到通知。

2. 在**消息**字段中，编写您的消息。根据需要，选择配置可选设置。
3. 单击**发送**，验证您的物理设备是否已收到通知。

### 监视发送的通知

可以通过导航至**监视**部分来监视发送的通知。IBM {{site.data.keyword.mobilepushshort}} 服务现在扩展了功能，可以通过用户数据生成图形，从而监视推送性能。您可以使用实用程序列出所有已发送的 {{site.data.keyword.mobilepushshort}}，或者列出所有已注册的设备，并每日、每周或每月分析信息。![](images/solution6/monitoring_messages.png)

## 相关内容
{: #related_content}

- [基于标记的通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}} 中的安全性](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

