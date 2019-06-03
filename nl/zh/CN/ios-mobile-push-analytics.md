---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-08"
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

# 使用 Push Notifications 的 iOS 移动应用程序
{: #ios-mobile-push-analytics}

了解如何轻而易举地快速创建使用 {{site.data.keyword.Bluemix_short}} 上高价值移动服务 {{site.data.keyword.mobilepushshort}} 的 iOS Swift 应用程序。

本教程将全程指导您创建移动入门模板应用程序，添加移动服务，设置客户机 SDK，将代码导入到 Xcode，然后进一步增强应用程序。
{:shortdesc: .shortdesc}

## 目标
{:#objectives}

- 通过基本 Swift 入门模板工具包创建使用 {{site.data.keyword.mobilepushshort}} 和 {{site.data.keyword.mobileanalytics_short}} 服务的移动应用程序。
- 获取 APNs 凭证，并配置 {{site.data.keyword.mobilepushshort}} 服务实例。
- 下载代码并设置客户机 SDK。
- 发送和监视 {{site.data.keyword.mobilepushshort}}。

  ![](images/solution6/Architecture.png)

## 产品
{:#products}

本教程使用以下产品：
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 开始之前
{: #prereqs}

1. [Apple Developer](https://developer.apple.com/) 帐户，用于从 {{site.data.keyword.Bluemix_short}} 上的 {{site.data.keyword.mobilepushshort}} 服务实例（提供者）将远程通知发送到 iOS 设备和应用程序。
2. Xcode，用于导入代码和增强代码。

## 通过基本 Swift 入门模板工具包创建移动应用程序
{: #get_code}

1. 导航至 [Mobile 仪表板](https://{DomainName}/developer/mobile/dashboard)以通过预定义的`入门模板工具包`创建`应用程序`。
2. 单击**入门模板工具包**，然后向下滚动以选择**基本**入门模板工具包。
    ![](images/solution6/mobile_dashboard.png)
3. 输入应用程序名称，这也将是 Xcode 项目和应用程序名称。
4. 选择 `iOS Swift` 作为平台，然后单击**创建**。
    ![](images/solution6/create_mobile_project.png)
5. 单击**添加资源** > 移动 > **推送通知**并选择您想要供应服务的位置、资源组以及**轻量**定价套餐。
6. 单击**创建**以供应 {{site.data.keyword.mobilepushshort}} 服务。在**应用程序**选项卡下将创建新的应用程序。

​      **注：** {{site.data.keyword.mobilepushshort}} 服务应该已添加有“空入门模板”。

## 下载代码并设置客户机 SDK
{: #download_code}

如果尚未下载代码，请单击“应用程序”> `您的移动应用程序`下的`下载代码`。下载的代码随附包含 **{{site.data.keyword.mobilepushshort}}** 客户机 SDK。客户机 SDK 在 CocoaPods 和 Carthage 上可用。对于此解决方案，将使用 CocoaPods。

1. 要在机器上安装 CocoaPods，请打开`终端`，然后运行以下命令。
   ```
   sudo gem install cocoapods
   ```
   {: pre:}
2. 解压缩下载的代码，然后使用终端，导航至解压缩的文件夹。

   ```
   cd <Name of Project>
   ```
   {: pre:}
3. 该文件夹已经包含 `podfile` 及其必需的依赖项。运行以下命令安装依赖项（客户机 SDK），这将安装必需的依赖项。

  ```
  pod install
  ```
  {: pre:}

## 获取 APNs 凭证，并配置 {{site.data.keyword.mobilepushshort}} 服务实例。
{: #obtain_apns_credentials}

   对于 iOS 设备和应用程序，Apple 推送通知服务 (APNs) 允许应用程序开发者从 {{site.data.keyword.Bluemix_short}} 上的 {{site.data.keyword.mobilepushshort}} 服务实例（提供者）将远程通知发送到 iOS 设备和应用程序。消息会发送到设备上的目标应用程序。

   您需要获取并配置 APNs 凭证。APNs 证书由 {{site.data.keyword.mobilepushshort}} 服务安全地进行管理，并用于连接到 APNs 服务器（提供者）。

### 注册应用程序标识

   应用程序标识（捆绑标识）是用于标识特定应用程序的唯一标识。每个应用程序都需要应用程序标识。像 {{site.data.keyword.mobilepushshort}} 服务这样的服务是配置给应用程序标识的。确保您具有 [Apple Developer](https://developer.apple.com/) 帐户。这是必备的先决条件。

   1. 转至 [Apple Developer](https://developer.apple.com/) 门户网站，单击 `Member Center`，然后选择 `Certificates, IDs & Profiles`。
   2. 转至 `Identifiers` > App IDs 部分。
   3. 在 `Registering App IDs` 页面中 App ID Description 下的 Name 字段中，提供应用程序名称。例如：ACME {{site.data.keyword.mobilepushshort}}。为 App ID Prefix 提供字符串。
   4. 对于 App ID Suffix，选中 `Explicit App ID`，并提供 Bundle ID 值。建议提供逆向域名样式的字符串。例如：com.ACME.push。
   5. 选中 `{{site.data.keyword.mobilepushshort}}` 复选框，然后单击 `Continue`。
   6. 仔细检查设置，然后单击 `Register` > `Done`。现在，您的应用程序标识已注册。

     ![](images/solution6/push_ios_register_appid.png)

### 创建开发和分发 APNs SSL 证书
   在获取 APNs 证书之前，必须首先生成一个证书签名请求 (CSR)，然后将其提交给 Apple（认证中心 (CA)）。CSR 包含用于标识您公司的信息以及用于对 Apple {{site.data.keyword.mobilepushshort}} 签名的公用密钥和专用密钥。然后，在 iOS 开发者门户网站上生成 SSL 证书。该证书与其公用密钥和专用密钥一起存储在“钥匙串访问”中。您可以在两种模式下使用 APNs：

- 沙箱模式，用于开发和测试。
- 生产模式，用于通过 App Store（或其他企业分发机制）分发应用程序。

   必须分别针对开发环境和分发环境获取不同的证书。证书与接收远程通知的应用程序的应用程序标识相关联。对于生产环境，最多可创建两个证书。{{site.data.keyword.Bluemix_short}} 使用证书与 APNs 建立 SSL 连接。

   1. 转至 Apple Developer Web 站点，单击 **Member Center**，然后选择 **Certificates, IDs & Profiles**。
   2. 在 **Identifiers** 区域中，单击 **App IDs**。
   3. 从应用程序标识列表中，选择您的应用程序标识，然后选择 `Edit`。
   4. 选中 **{{site.data.keyword.mobilepushshort}}** 复选框，然后在 **Development SSL certificate** 窗格上，单击 **Create Certificate**。

     ![Push Notification SSL 证书](images/solution6/certificate_createssl.png)

   5. 显示 **About Creating a Certificate Signing Request (CSR)** 屏幕时，按照显示的指示信息创建证书签名请求 (CSR) 文件。提供有意义的名称来帮助您识别这是开发（沙箱）证书还是分发（生产）证书；例如，sandbox-apns-certificate 或 production-apns-certificate。
   6. 单击 **Continue**，在 Upload CSR file 屏幕上，单击 **Choose File**，然后选择刚才创建的 **CertificateSigningRequest.certSigningRequest** 文件。单击**继续**。
   7. 在 Download, Install and Backup 窗格中，单击 Download。这将下载 **aps_development.cer** 文件。
        ![下载证书](images/solution6/push_certificate_download.png)

        ![生成证书](images/solution6/generate_certificate.png)
   8. 在 Mac 上，打开**钥匙串访问** > **文件** > **导入**，选择下载的 .cer 文件以进行安装。
   9. 右键单击新的证书和专用密钥，然后选择**导出**，并将**文件格式**更改为“个人信息交换”格式（`.p12` 格式）。
     ![导出证书和密钥](images/solution6/keychain_export_key.png)
   10. 在**存储为**字段中，为证书提供有意义的名称。例如，`sandbox_apns.p12` 或 **production_apns.p12**，然后单击“存储”。
     ![导出证书和密钥](images/solution6/certificate_p12v2.png)
   11. 在**输入密码**字段中，输入用于保护导出项的密码，然后单击“好”。可以使用此密码在 {{site.data.keyword.mobilepushshort}} 服务控制台上配置 APNs 设置。
       ![导出证书和密钥](images/solution6/export_p12.png)
   12. **Key Access.app** 会提示您从**钥匙串**屏幕导出密钥。输入 Mac 的管理员密码，以允许系统导出这些项，然后选择`总是允许`选项。这将在桌面上生成一个 `.p12` 证书。

      对于“生产 SSL”，在 **Production SSL certificate** 窗格中，单击 **Create Certificate**，然后重复上面的步骤 5 到 12。
      {:tip}

### 创建开发供应概要文件
   供应概要文件与应用程序标识一起用于确定哪些设备可以安装并运行您的应用程序，以及您的应用程序可以访问哪些服务。对于每个应用程序标识，都可创建两个供应概要文件：一个用于开发，另一个用于分发。Xcode 使用开发供应概要文件来确定允许哪些开发者构建应用程序，以及允许在应用程序上测试哪些设备。

   确保您已执行以下操作：注册应用程序标识，针对 {{site.data.keyword.mobilepushshort}} 服务启用该标识，然后将其配置为使用开发和生产 APNs SSL 证书。

   创建开发供应概要文件，如下所示：

   1. 转至 [Apple Developer](https://developer.apple.com/) 门户网站，单击 `Member Center`，然后选择 `Certificates, IDs & Profiles`。
   2. 转至 [Mac Developer Library](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site)，滚动到 `Creating Development Provisioning Profiles` 部分，然后按照指示信息创建开发概要文件。
     **注**：配置开发供应概要文件时，请选择以下选项：

     - **iOS App Development**
     - **For iOS and watchOS apps**

### 创建应用商店分发供应概要文件
   使用应用商店供应概要文件可提交应用程序以分发到 App Store。

   1. 转至 [Apple Developer](https://developer.apple.com/)，单击 `Member Center`，然后选择 `Certificates, IDs & Profiles`。
   2. 双击下载的供应概要文件，以将其安装到 Xcode 中。获取凭证后，下一步是[配置服务实例](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2)。

### 配置服务实例

   要使用 {{site.data.keyword.mobilepushshort}} 服务来发送通知，请上传在上面的步骤中已创建的 .p12 证书。此证书包含构建和发布应用程序所需的专用密钥和 SSL 证书。

   **注**：在 `.cer` 文件出现在“钥匙串访问”中之后，请将其导出到您的计算机，以创建 `.p12` 证书。

1. 单击“服务”部分下的 {{site.data.keyword.mobilepushshort}}，或单击 {{site.data.keyword.mobilepushshort}} 服务旁边的三个垂直点，然后选择`打开仪表板`。
2. 在 {{site.data.keyword.mobilepushshort}} 仪表板上，您应该会在`管理`下看到`配置`选项。

要在 `Push Notification 服务`控制台上设置 APNs，请完成以下步骤：

1. 选择`移动`选项，以更新“APNs 推送凭证”表单上的信息。
2. 根据需要选择`沙箱/开发 APNs 服务器`或`生产 APNs 服务器`，然后上传已创建的 `.p12` 证书。
3. 在“密码”字段中，输入与 .p12 证书文件关联的密码，然后单击“保存”。

![](images/solution6/Mobile_push_configure.png)

## 配置、发送和监视 {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. 推送初始化代码（在`函数应用程序`下）和通知注册代码可在 `AppDelegate.swift` 中找到。提供唯一的 USER_ID（可选）。
2. 在物理设备上运行应用程序，因为通知无法发送到 iPhone 模拟器。
3. 在 {{site.data.keyword.Bluemix_short}} Mobile 仪表板的`移动服务` > **现有服务**下打开 {{site.data.keyword.mobilepushshort}} 服务，要发送基本 {{site.data.keyword.mobilepushshort}}，请完成以下步骤：
  * 选择`消息`，然后通过选择“收件人”选项来编写消息。支持的选项包括“带标记的设备”、“设备标识”、“用户标识”、“Android 设备”、“iOS 设备”、“Web 通知”、“所有设备”和“其他浏览器”。

       **注**：选择“所有设备”选项时，预订了 {{site.data.keyword.mobilepushshort}} 的所有设备都会收到通知。
  * 在`消息`字段中，编辑您的消息。根据需要，选择配置可选设置。
  * 单击`发送`，然后验证物理设备是否收到了通知。
    ![](images/solution6/send_notifications.png)

4. 您应该会在 iPhone 上看到通知。

   ![](images/solution6/iphone_notification.png)

5. 可以通过在 {{site.data.keyword.mobilepushshort}} 服务上导航至`监视`部分来监视发送的通知。

IBM {{site.data.keyword.mobilepushshort}} 服务现在扩展了功能，可以通过用户数据生成图形，从而监视推送性能。您可以使用实用程序列出所有已发送的 {{site.data.keyword.mobilepushshort}}，或者列出所有已注册的设备，并每日、每周或每月分析信息。![](images/solution6/monitoring_messages.png)

## 相关内容
{: #related_content}

- [基于标记的通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}} 中的安全性](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)
