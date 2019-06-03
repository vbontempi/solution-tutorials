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

# 使用 CDN 加速交付静态文件
{: #static-files-cdn}

本教程将全程指导您如何在 {{site.data.keyword.cos_full_notm}} 中托管和提供 Web 站点资产（图像、视频和文档）和用户生成的内容，以及如何使用 [{{site.data.keyword.cdn_full}} (CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai) 快速、安全地向世界各地的用户交付内容。

Web 应用程序具有不同类型的内容：HTML 内容、图像、视频、级联样式表、JavaScript 文件以及用户生成的内容。有些内容经常变化，有些内容则变化不大，有些内容经常被很多用户访问，有些则是偶尔才会被访问。随着应用程序的受众增长，您可能希望将提供这些内容的工作卸载到其他组件，从而为主应用程序释放资源。您可能还希望从距离应用程序用户很近的位置来提供这些内容，无论用户身处全球哪个位置。

为什么要在这些情境中使用 Content Delivery Network 的原因有很多：
* CDN 将对内容进行高速缓存，并且仅当内容在其高速缓存中不可用或已到期时，才会从源（您的服务器）中拉取内容；
* 凭借遍布全球的多个数据中心，CDN 将为用户提供距离最近的位置中的高速缓存内容；
* 浏览器在不同于主应用程序的域上运行，因此能够并行装入更多内容 - 大多数浏览器对每个主机名的连接数有限制。

## 目标
{: #objectives}

* 将文件上传到 {{site.data.keyword.cos_full_notm}} 存储区。
* 通过 Content Delivery Network (CDN) 使内容全球可用。
* 使用 Cloud Foundry Web 应用程序公开文件。

## 使用的服务
{: #services}

本教程使用以下产品：
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构

<p style="text-align: center;">
![体系结构](images/solution3/Architecture.png)
</p>

1. 用户访问应用程序
2. 应用程序包含通过 Content Delivery Network 分发的内容
3. 如果内容在 CDN 中不可用或已到期，CDN 将从源中拉取内容。

## 开始之前
{: #prereqs}

请联系基础架构帐户的主用户来获取以下许可权：
   * 管理 CDN 帐户
   * 管理存储器
   * API 密钥

需要这些许可权才能查看和使用存储器和 CDN 服务。

## 获取 Web 应用程序代码
{: #get_code}

假设有一个简单的 Web 应用程序，具有不同类型的内容，如图像、视频和级联样式表。您将内容存储在存储区中，并将 CDN 配置为使用该存储区作为其源。

首先，检索应用程序代码：

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## 创建 Object Storage
{: #create_cos}

{{site.data.keyword.cos_full_notm}} 为非结构化数据提供了灵活、有成本效益且可缩放的云存储器。

1. 转至控制台中的[目录](https://{DomainName}/catalog/)，然后从“存储”部分中，选择 [**Object Storage**](https://{DomainName}/catalog/services/cloud-object-storage)。
2. 创建新的 {{site.data.keyword.cos_full_notm}} 实例。
4. 在服务仪表板中，单击**创建存储区**。
5. 设置唯一存储区名称，如 `username-mywebsite`，然后单击**创建**。避免在存储区名称中使用点 (.)。

## 将文件上传到存储区
{: #upload}

在此部分中，将使用命令行工具 **curl** 将文件上传到存储区。

1. 在 CLI 中登录到 {{site.data.keyword.Bluemix_notm}}，然后从 IBM Cloud IAM 中获取令牌。
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. 复制上一步命令输出中的令牌。
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. 将令牌和存储区名称的值设置为环境变量，以方便访问。
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. 从先前下载的 Web 应用程序代码的内容目录中，上传名为 `a-css-file.css`、`a-picture.png` 和 `a-video.mp4` 的文件。将这些文件上传到存储区的根目录。
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. 在仪表板中查看这些文件。
   ![](images/solution3/Buckets.png)
6. 通过浏览器使用类似于以下示例的链接来访问这些文件：

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## 通过 CDN 使文件全球可用

在此部分中，您将创建 CDN 服务。CDN 服务将内容分发到需要该内容的位置。首次收到内容请求时，该服务会将内容从主机服务器（{{site.data.keyword.cos_full_notm}} 中的存储区）中拉取到网络并将其保留在网络中，以供其他用户快速访问，而不会有再次访问主机服务器而产生的网络等待时间。

### 创建 CDN 实例

1. 转至控制台中的目录，然后从“网络”部分中，选择 **Content Delivery Network**。此 CDN 基于 Akamai 技术。单击**创建**。
2. 在下一个对话框中，将 CDN 的**主机名**设置为定制域。虽然设置的是定制域，但仍可以通过 IBM 提供的 CNAME 来访问 CDN 内容。因此，如果不打算使用定制域，那么可以设置任意名称。
3. 将**定制 CNAME** 前缀设置为唯一值。
4. 接下来，在**配置源**下，选择 **Object Storage** 以便为 COS 配置 CDN。
5. 将**端点**设置为存储区 API 端点，例如 *s3-api.us-geo.objectstorage.softlayer.net*。
6. 将**主机头**和**路径**保留为空。将**存储区名称**设置为 *your-bucket-name*。
7. 启用 HTTP (80) 和 HTTPS (443) 端口。
8. 对于 **SSL 证书**，如果要使用定制域，请选择 *DV SAN 证书*。或者，要通过 CNAME 访问存储器，请选取**通配符证书*选项。
9. 单击**创建**。

### 通过 CDN CNAME 访问内容

1. [在列表中](https://{DomainName}/classic/network/cdn)选择 CDN 实例。
2. 如果早先选取了 *DV SAN 证书*，那么在初始设置完成后，系统将提示您进行域验证。请执行单击**查看域验证**时显示的步骤。
3. **详细信息**面板显示了 CDN 的**主机名**和 **CNAME**。
4. 使用 `https://your-cdn-cname.cdnedge.bluemix.net/a-picture.png` 访问文件，或者如果使用的是定制域，请使用 `https://your-cdn-hostname/a-picture.png` 进行访问。如果省略文件名，那么您应该会看到 S3 ListBucketResult。

## 部署 Cloud Foundry 应用程序

应用程序包含 public/index.html Web 页面，其中含有对现在 {{site.data.keyword.cos_full_notm}} 中所托管文件的引用。后端 `app.js` 提供此 Web 页面，并将占位符替换为 CDN 的实际位置。这样，该 Web 页面使用的所有资产都将由 CDN 提供。

1. 在终端中，进入检出了代码的目录。
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. 推送应用程序，而不启动该应用程序。
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. 配置 CDN_NAME 环境变量，以便应用程序可以引用 CDN 内容。
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. 启动应用程序。
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. 使用 Web 浏览器访问应用程序，页面样式表、图片和视频将从 CDN 装入。

![](images/solution3/Application.png)

将 CDN 与 {{site.data.keyword.cos_full_notm}} 配合使用是一种功能强大的组合，支持托管文件，并将其提供给世界各地的用户。此外，还可以使用 {{site.data.keyword.cos_full_notm}} 来存储用户上传到应用程序的任何文件。

## 除去资源

* 删除 Cloud Foundry 应用程序
* 删除 Content Delivery Network 服务
* 删除 {{site.data.keyword.cos_full_notm}} 服务或存储区

## 相关内容

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[管理对 {{site.data.keyword.cos_full_notm}} 的访问](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[CDN 入门](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
