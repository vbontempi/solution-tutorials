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


# 无服务器 Web 应用程序和 API
{: #serverless-api-webapp}

在本教程中，您将通过在 GitHub 页面上托管静态 Web 站点内容，并使用 {{site.data.keyword.openwhisk}} 实现应用程序后端来创建无服务器 Web 应用程序。

{{site.data.keyword.openwhisk_short}} 作为一个事件驱动型平台，支持[各种用例](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)。构建 Web 应用程序和 API 就是其中一例。对于 Web 应用程序，事件是指 Web 浏览器（或 REST 客户机）和 Web 应用程序与 HTTP 请求之间的交互。您可以不供应虚拟机、容器或 Cloud Foundry 运行时来部署后端，而改为使用无服务器平台实现后端 API。这会是一个不错的解决方案，可以避免为空闲时间付费，并支持平台根据需要进行缩放。

{{site.data.keyword.openwhisk_short}} 中的任何操作（或函数）都可以转换为可供 Web 客户机使用的现成 HTTP 端点。针对 Web 启用时，这些操作称为 *Web 操作*。拥有 Web 操作后，可以通过 API 网关将这些操作组合成全功能 API。API 网关是 {{site.data.keyword.openwhisk_short}} 的一个组件，用于公开 API。此组件随附安全性、OAuth 支持、速率限制、定制域支持等功能。

## 目标

* 部署无服务器后端和数据库
* 公开 REST API
* 托管静态 Web 站点
* 可选：将定制域用于 REST API

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

本教程中说明的应用程序是一个简单的留言板 Web 站点，用户可以在其中发布消息。

<p style="text-align: center;">

   ![体系结构](./images/solution8/Architecture.png)
</p>

1. 用户访问在 GitHub Pages 中托管的应用程序。
2. 该 Web 应用程序调用后端 API。
3. 后端 API 在 API 网关中进行定义。
4. API 网关将请求转发给 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)。
5. {{site.data.keyword.openwhisk_short}} 操作使用 [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) 来存储和检索留言板条目。

## 开始之前
{: #prereqs}

本指南使用 GitHub Pages 来托管静态 Web 站点。因此，请确保您有公共 GitHub 帐户。

## 创建 Guestbook 数据库

首先创建 {{site.data.keyword.cloudant_short_notm}}。{{site.data.keyword.cloudant_short_notm}} 是一个完全受管的数据层，专为现代 Web 和移动应用程序而设计，采用了灵活的 JSON 模式。{{site.data.keyword.cloudant_short_notm}} 基于 Apache CouchDB 而构建并与之兼容，可通过安全的 HTTPS API 进行访问，支持随着应用程序的增长而缩放。

1. 在“目录”中，选择 **Cloudant**。
2. 将服务名称设置为 **guestbook-db**，选择**使用旧凭证和 IAM** 作为认证方法，然后单击**创建**。
3. 返回到“资源列表”，单击“名称”列下的 ***guestbook-db** 条目。注：您可能需要等待服务供应完毕。 
4. 在服务详细信息屏幕中，单击***启动 Cloudant 仪表板***，该仪表板将在另一个浏览器选项卡中打开。注：Cloudant 实例可能需要登录。 
5. 单击***创建数据库***，然后创建名为 ***guestbook*** 的数据库。

  ![](images/solution8/Create_Database.png)

6. 返回到服务详细信息选项卡，在**服务凭证**下
   1. **新建凭证**，接受缺省值，然后单击**添加**。
   2. 单击“操作”下的**查看凭证**。稍后将需要这些凭证来允许 Cloud Functions 操作对 Cloudant 服务执行读取/写入操作。

## 创建无服务器操作

在此部分中，您将创建无服务器操作（通常称为“函数”）。{{site.data.keyword.openwhisk}}（基于 Apache OpenWhisk）是一种函数即服务 (FaaS) 平台，用于根据入局事件执行函数，并且在不使用时，不会产生任何成本。


![](images/solution8/Functions.png)

### 保存留言板条目的操作序列

您将创建**序列**；序列是操作链，其中一个操作的输出充当下一个操作的输入，依此类推。将创建的第一个序列用于持久存储访客消息。如果提供了姓名、电子邮件标识和注释，那么序列将为：
   * 创建要持久存储的文档。
   * 将文档存储在 {{site.data.keyword.cloudant_short_notm}} 数据库中。

首先创建第一个操作：

1. 切换到 **Functions** https://{DomainName}/openwhisk。
2. 在左侧窗格中，单击**操作**，然后单击**创建**。
3. **创建操作**，名称为 `prepare-entry-for-save`，然后选择 **Node.js** 作为“运行时”（注：请选取最新的版本）。
4. 将现有代码替换为以下代码片段：
   ```js
   /**
    * Prepare the guestbook entry to be persisted
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. **保存**

然后，将该操作添加到序列：

1. 单击**闭包序列**，然后单击**添加到序列**。
1. 对于序列名称，输入 `save-guestbook-entry-sequence`，然后单击**创建并添加**。

然后，将第二个操作添加到序列：

1. 单击 **save-guestbook-entry-sequence**，然后单击**添加**。
1. 选择**使用公共** > **Cloudant**，然后选择**操作**下的 **create-document**。
1. **新建绑定**。 
1. 对于“名称”，输入 `binding-for-guestbook`。
1. 对于“Cloudant 实例”，选择`输入您自己的凭证`，并使用为 Cloudant 实例捕获的凭证信息填写“用户名”、“密码”、“主机”和“数据库”（为 `guestbook`）字段，然后单击**添加**，再单击**保存**。
   {: tip}
1. 要进行测试，请单击**更改输入**，然后输入以下 JSON：
    ```json
    {
      "name": "John Smith",
      "email": "john@smith.com",
      "comment": "this is my comment"
    }
    ```
    {: codeblock}
1. 单击**应用**，然后单击**调用**。
    ![](images/solution8/Save_Entry_Invoke.png)

### 检索条目的操作序列

第二个序列用于检索现有的留言板条目。此序列将：
   * 列出数据库中的所有文档。
   * 设置文档格式，然后返回这些文档。

1. 在 **Functions** 下，单击**操作**，然后**创建**新的 Node.js 操作，并将其命名为 `set-read-input`。
2. 将现有代码替换为以下代码片段。此操作会将相应的参数传递给下一个操作。
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. **保存**

将该操作添加到序列：

1. 单击**闭包序列** > **添加到序列**，然后单击**新建**。
1. 对于**操作名称**，输入 `read-guestbook-entries-sequence`，然后单击**创建并添加**。

完成序列：

1. 单击 **read-guestbook-entries-sequence** 序列，然后单击**添加**以创建并添加第二个操作，此操作用于从 Cloudant 获取文档。
1. 在**使用公共**下，选择 **{{site.data.keyword.cloudant_short_notm}}**，然后选择 **list-documents**。
1. 在**我的绑定**下，选择 **binding-for-guestbook**，然后选择**添加**以创建此公共操作，并将其添加到序列。
1. 再次单击**添加**以创建并添加第三个操作，此操作将用于设置来自 {{site.data.keyword.cloudant_short_notm}} 的文档的格式。
1. 在**新建**下，对于“名称”，输入 `format-entries`，然后单击**创建并添加**。
1. 单击 **format-entries**，然后将代码替换为以下内容：
   ```js
   const md5 = require('spark-md5');

   function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. **保存**
1. 通过单击**操作**，然后单击 **read-guestbook-entries-sequence** 来选择序列。
1. 单击**保存**，然后单击**调用**。输出应该类似于以下内容：
   ![](images/solution8/Read_Entries_Invoke.png)

## 创建 API

![](images/solution8/Cloud_Functions_API.png)

1. 转至“操作”：https://{DomainName}/openwhisk/actions。
2. 选择 **read-guestbook-entries-sequence** 序列。在名称旁边，单击 **Web 操作**，选中**启用 Web 操作**，然后单击**保存**。
3. 对 **save-guestbook-entry-sequence** 序列执行相同的操作。
4. 转至 API (https://{DomainName}/openwhisk/apimanagement)，然后单击**创建 {{site.data.keyword.openwhisk_short}} API**。
5. 将“名称”设置为 `guestbook`，将“基本路径”设置为 `/guestbook`。
6. 单击**创建操作**，然后创建用于检索留言板条目的操作：
   1. 将**路径**设置为 `/entries`
   2. 将**动词**设置为 `GET*`
   3. 选择 **read-guestbook-entries-sequence** 操作
7. 单击**创建操作**，然后创建用于持久存储留言板条目的操作：
   1. 将**路径**设置为 `/entries`
   2. 将**动词**设置为 `PUT`
   3. 选择 **save-guestbook-entry-sequence** 操作
8. 保存并公开该 API。

## 部署 Web 应用程序

1. 将 Guestbook 用户界面存储库 (https://github.com/IBM-Cloud/serverless-guestbook) 派生到公共 GitHub。
2. 修改 **docs/guestbook.js**，并将 **apiUrl** 的值替换为 API 网关给定的路径。
3. 将修改后的文件提交到派生的存储库。
4. 在存储库的“设置”页面中，滚动到 **GitHub Pages**，将源更改为**主分支 /docs 文件夹**，然后单击“保存”。
5. 访问存储库的公共页面。
6. 您应该会看到早先创建的“测试”留言板条目。
7. 添加新条目。

![](images/solution8/Guestbook.png)

## 可选：将您自己的域用于 API

通过创建受管 API，您可获得缺省端点，如 `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`。在此部分中，您将配置此端点，使其能够处理来自于定制子域的请求。

### 获取定制域的证书

要通过定制域公开 {{site.data.keyword.openwhisk_short}} 操作，需要安全的 HTTPS 连接。您应该获取计划用于无服务器后端的域和子域的 SSL 证书。假定有类似 *mydomain.com* 的域，那么可以在 *guestbook-api.mydomain.com* 上托管操作。需要签发用于 *guestbook-api.mydomain.com*（或 **.mydomain.com*）的证书。

可以从 [Let's Encrypt](https://letsencrypt.org/) 获取免费的 SSL 证书。在此过程中，可能需要在 DNS 界面中配置 TXT 类型的 DNS 记录，以证明您是域的所有者。
{:tip}

获得域的 SSL 证书和专用密钥后，请务必将其转换为 [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) 格式。

1. 将证书转换为 PEM 格式：
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. 将专用密钥转换为 PEM 格式：
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### 将证书导入到中央存储库

1. 在支持的位置中创建 [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) 实例。
1. 在服务仪表板中，使用**导入证书**：
   * 将**名称**设置为定制子域和域，例如 `guestbook-api.mydomain.com`。
   * 浏览以找到 PEM 格式的**证书文件**。
   * 浏览以找到 PEM 格式的**专用密钥文件**。
   * **导入**。

### 为受管 API 配置定制域

1. 转至 [API / 定制域](https://{DomainName}/apis/domains)。
1. 在“区域”选择器中，选择部署了操作的位置。
1. 找到链接到在其中创建了操作和受管 API 的组织和空间的定制域。单击“操作”菜单中的**更改设置**。
1. 记下**缺省域 / 别名**值。
1. 选中**应用定制域**
   1. 将**域名**设置为将使用的域，例如 `guestbook-api.mydomain.com`。
   1. 选择持有证书的 {{site.data.keyword.cloudcerts_short}} 实例。
   1. 从域中选择证书。
1. 转至 DNS 提供程序，然后创建新的 **DNS TXT 记录**，用于将您的域映射到 API 缺省域/别名。应用设置后，可以除去 DNS TXT 记录。
1. 保存定制域设置。该对话框将检查是否存在 DNS TXT 记录。
1. 最后，返回到 DNS 提供程序的设置，然后创建 CNAME 记录，用于将定制域（例如，guestbook-api.mydomain.com）指向缺省域/别名。这将使得流经定制域的流量被路由到后端 API。

传播 DNS 更改后，即能够访问留言板 API：https://guestbook-api.mydomain.com/guestbook。

1. 编辑 **docs/guestbook.js**，并将 **apiUrl** 值更新为 https://guestbook-api.mydomain.com/guestbook。
1. 落实修改的文件。
1. 现在，应用程序可通过定制域访问该 API。

## 除去资源

* 删除 {{site.data.keyword.cloudant_short_notm}} 服务
* 从 {{site.data.keyword.openwhisk_short}} 中删除 API
* 从 {{site.data.keyword.openwhisk_short}} 中删除操作

## 相关内容
* [有关无服务器的更多指南和样本](https://developer.ibm.com/code/journey/category/serverless/)
* [{{site.data.keyword.openwhisk}} 入门](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [{{site.data.keyword.openwhisk}} 常见用例](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [通过操作创建 REST API](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
