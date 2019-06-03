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


# 使用 MEAN 堆栈的现代 Web 应用程序
{: #mean-stack}

本教程将全程指导您使用热门 MEAN 堆栈创建 Web 应用程序。MEAN 堆栈由 **M**ongo DB、**E**xpress Web 框架、**A**ngular 前端框架和 Node.js 运行时组成。您将学习如何在本地运行 MEAN 入门模板，创建和使用受管数据库即服务 (DBasS)，将应用程序部署到 {{site.data.keyword.cloud_notm}} 以及监视应用程序。  

## 目标

{: #objectives}

- 在本地创建并运行入门模板 Node.js 应用程序。
- 创建受管数据库即服务 (DBasS)。
- 将 Node.js 应用程序部署到云。
- 缩放 MongoDB 资源。
- 了解如何监视应用程序性能。

## 使用的服务

{: #products}

本教程使用以下 {{site.data.keyword.Bluemix_notm}} 服务：

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**注意：**本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构

{:#architecture}

<p style="text-align: center;">

![体系结构图](images/solution7/Architecture.png)</p>

1. 用户使用 Web 浏览器访问应用程序。
2. Node.js 应用程序转至 {{site.data.keyword.composeForMongoDB}} 数据库来访存数据。

## 开始之前

{: #prereqs}

1. [安装 Git](https://git-scm.com/)
2. [安装 {{site.data.keyword.Bluemix_notm}} CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)


要在本地开发和运行应用程序：
1. [安装 Node.js 和 NPM](https://nodejs.org/)
2. [安装和运行 MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/)

## 本地运行 MEAN 应用程序

{: #runapplocally}

在此部分中，您将运行本地 MongoDB 数据库，克隆 MEAN 样本代码，并在本地运行应用程序以使用本地 MongoDB 数据库。

{: shortdesc}

1. 按照[此处](https://docs.mongodb.com/manual/administration/install-community/)的指示信息在本地安装并运行 MongoDB 数据库。安装完成后，使用以下命令来确认该数据库是否正在运行。
  ```sh
  mongo
  ```
  {: codeblock}

2. 克隆 MEAN 入门模板代码。

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. 安装必需的软件包。

  ```sh
  npm install
  ```
  {: codeblock}

4. 将 .env.example 文件复制到 .env。编辑所需的信息，至少添加您自己的 SESSION_SECRET。

5. 运行 node server.js 以启动应用程序。
  ```sh
  node server.js
  ```
  {: codeblock}

6. 访问应用程序，创建新用户并登录。

## 在云中创建 MongoDB 数据库的实例

{: #createdatabase}

在此部分中，您将在云中创建 {{site.data.keyword.composeForMongoDB}} 数据库。{{site.data.keyword.composeForMongoDB}} 是数据库即服务，通常更容易配置，并提供内置备份和缩放功能。可以在 [IBM Cloud“目录”](https://{DomainName}/catalog/?category=data)中找到许多不同类型的数据库。要创建 {{site.data.keyword.composeForMongoDB}}，请执行以下步骤。

{: shortdesc}

1. 通过命令行登录到您的 {{site.data.keyword.cloud_notm}} 帐户，并将您的 {{site.data.keyword.cloud_notm}} 帐户设定为目标。 

  ```sh
  ibmcloud login
  ibmcloud target --cf
  ```
  {: codeblock}

  可以在[此处](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)找到更多 CLI 命令。

2. 创建 {{site.data.keyword.composeForMongoDB}} 的实例。这还可使用[控制台 UI](https://{DomainName}/catalog/services/compose-for-mongodb) 来执行。服务名称必须命名为 **mean-starter-mongodb**，因为应用程序配置为使用此名称来查找此服务。

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## 将应用程序部署到云

{: #deployapp}

在此部分中，您要将 node.js 应用程序部署到使用受管 MongoDB 数据库的 {{site.data.keyword.cloud_notm}}。源代码包含 [**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml) 文件，该文件已配置为使用早先创建的“mongodb”服务。应用程序使用 VCAP_SERVICES 环境变量来访问 Compose MongoDB 数据库凭证。这可以在 [server.js 文件](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js)中进行查看。 

{: shortdesc}

1. 将代码推送到云。

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. 推送代码后，您应该可以在浏览器中查看应用程序。系统生成了随机主机名，可能类似于：`https://mean-random-name.mybluemix.net`。您可以通过控制台仪表板或命令行来获取应用程序 URL。![实时应用程序](images/solution7/live-app.png)


## 缩放 MongoDB 数据库资源
{: #scaledatabase}

如果您的服务需要额外存储空间，或者您要减少分配给服务的存储量，那么可以通过缩放资源来完成此操作。

{: shortdesc}

1. 使用控制台**仪表板**，转至**连接**部分，然后单击 **MongoDB 实例**数据库。
2. 在**部署详细信息**面板中，单击**缩放资源**选项。
  ![](images/solution7/mongodb-scale-show.png)
3. 调整**滑块**以增加或减少分配给 {{site.data.keyword.composeForMongoDB}} 数据库服务的存储量。
4. 单击**缩放部署**以触发重新缩放并返回到仪表板概述页面。这将在页面顶部显示“缩放已启动”消息，通知您正在执行重新缩放。
  ![](images/solution7/scaling-in-progress.png)


## 监视应用程序性能
{: #monitorapplication}

要检查应用程序的运行状况，可以使用内置的可用性监视服务。可用性监视服务会自动附加到云中的应用程序。可用性监视服务在全球各个位置全天候运行合成测试，以主动检测和修复性能问题，避免这些问题影响访问者。执行以下步骤来访问监视仪表板。

{: shortdesc}

1. 使用控制台仪表板，在您的应用程序下，选择**监视**选项卡。
2. 单击**查看所有测试**来查看测试。
   ![](images/solution7/alert_frequency.png)


## 相关内容

{: #related}

- 设置源代码控制和[持续交付](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops)。
- 在[多个位置](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp)中保护 Web 应用程序。
- 创建、保护和管理 [REST API](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis)。
