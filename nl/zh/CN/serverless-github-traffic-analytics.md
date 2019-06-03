---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 将无服务器与 Cloud Foundry 组合用于数据检索和分析
{: #serverless-github-traffic-analytics}
在本教程中，您将创建一个应用程序，用于自动收集存储库的 GitHub 流量统计信息，并为流量分析奠定基础。GitHub 仅提供对最近 14 天的流量数据的访问。如果要分析更长时间段内的统计信息，您需要自行下载并存储这些数据。在本教程中，您将部署无服务器操作，以检索流量数据并将其存储在 SQL 数据库中。此外，将使用 Cloud Foundry 应用程序来管理存储库，并提供对统计数据的访问以进行数据分析。本教程中讨论的应用程序和无服务器操作实现的是支持多租户的解决方案，其初始功能集支持单租户方式。

![](images/solution24-github-traffic-analytics/Architecture.png)

## 目标

* 部署具有多租户支持和安全访问权的 Python 数据库应用程序
* 将 App ID 集成为基于 OpenID Connect 的认证服务提供者
* 设置 GitHub 流量统计信息的自动无服务器收集
* 集成 {{site.data.keyword.dynamdashbemb_short}} 以进行图形流量分析

## 产品
本教程使用以下产品：
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## 开始之前
{: #prereqs}

要完成本教程，您需要最新版本的 [IBM Cloud CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)，并且 {{site.data.keyword.openwhisk_short}} [插件已安装](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)。

## 服务和环境设置 (shell)
在此部分中，您将设置所需的服务并准备环境。所有这些操作都可以通过 shell 环境完成。

1. 克隆 [GitHub 存储库](https://github.com/IBM-Cloud/github-traffic-stats)，然后导航至克隆的目录及其 **backend** 子目录：
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. 使用 `ibmcloud login` 以交互方式登录到 {{site.data.keyword.Bluemix_short}}。可以通过运行 `ibmcloud target` 命令重新确认详细信息。您需要设置组织和空间。

3. 创建使用**入门**套餐的 {{site.data.keyword.dashdbshort}} 实例，并将其命名为 **ghstatsDB**：
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. 要在以后从 {{site.data.keyword.openwhisk_short}} 访问数据库服务，您需要授权。因此，请创建服务凭证，并将其标注为 **ghstatskey**：   
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. 创建 {{site.data.keyword.appid_short}} 服务实例。使用 **ghstatsAppID** 作为名称，并使用**累进层**套餐。
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
   随后，在 Cloud Foundry 空间中创建该新服务实例的别名。将 **YOURSPACE** 替换为要部署到的空间。
   ```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. 创建使用**轻量**套餐的 {{site.data.keyword.dynamdashbemb_short}} 服务实例。
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
   同样，创建该新服务实例的别名，并替换 **YOURSPACE**。
   ```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. 在 **backend** 目录中，将应用程序推送到 IBM Cloud。该命令会将随机路径用于应用程序。
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
   等待部署完成。此时应用程序文件已上传，运行时环境已创建，并且服务已绑定到应用程序。服务信息是从 `manifest.yml` 文件中获取的。如果使用了其他服务名称，那么需要更新该文件。该过程成功完成后，将显示应用程序 URI。

   上面的命令是将随机但唯一的路径用于应用程序。如果希望自己选取路径，请将该路径作为附加参数添加到命令，例如 `ibmcloud cf push your-app-name`。此外，还可以编辑 `manifest.yml` 文件，更改 **name**，并将 **random-route** 从 **true** 更改为 **false**。
   {:tip}

## App ID 和 GitHub 配置（浏览器）
以下步骤全部使用因特网浏览器执行。首先，将 {{site.data.keyword.appid_short}} 配置为使用 Cloud Directory 和 Python 应用程序。随后，创建 GitHub 访问令牌。部署的函数检索流量数据时需要此令牌。

1. 在 [{{site.data.keyword.Bluemix_short}} 资源列表](https://{DomainName}/resources)中，打开服务概述。在**服务**部分中，找到 {{site.data.keyword.appid_short}} 服务实例。单击其条目以打开详细信息。
2. 在服务仪表板中，单击左侧菜单中**身份提供者**下的**管理**。这将显示可用身份提供者的列表，例如 Facebook、Google、SAML 2.0 Federation 和 Cloud Directory。将 Cloud Directory 切换为**开启**，将其他所有提供者切换为**关闭**。
   
   您可能需要配置[多因子认证 (MFA)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) 和高级密码规则。但这些内容不在本教程中讨论。
   {:tip}

3. 在该页面底部是重定向 URL 的列表。输入应用程序的 **URL** + /redirect_uri。例如，`https://github-traffic-stats-random-word.mybluemix.net/redirect_uri`。

   要在本地测试应用程序，重定向 URL 为 `http://0.0.0.0:5000/redirect_uri`。可以配置多个重定向 URL。
   {:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. 在左侧的菜单中，单击**用户**。这将打开 Cloud Directory 中的用户列表。单击**添加用户**按钮将您自己添加为第一个用户。现在，您已完成配置 {{site.data.keyword.appid_short}} 服务。
5. 在浏览器中，访问 [Github.com](https://github.com/settings/tokens)，然后转至 **Settings -> Developer settings -> Personal access tokens**。单击 **Generate new token** 按钮。对于 **Token description**，输入 **GHStats Tutorial**。随后，启用 **repo** 类别下的 **public_repo**，以及 **admin:org** 下的 **read:org**。现在，在该页面的底部，单击 **Generate token**。新访问令牌会显示在下一页中。在随后的应用程序设置期间需要此令牌。
   ![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## 配置并测试 Python 应用程序
准备工作完成后，可以配置并测试应用程序。应用程序是使用常用的 [Flask](http://flask.pocoo.org/) 微框架以 Python 编写的。可以在统计信息收集中添加和除去存储库。流量数据可以采用表格视图进行访问。

1. 在浏览器中，打开已部署应用程序的 URI。您应该会看到欢迎页面。
   ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. 在浏览器中，将 `/admin/initialize-app` 添加到 URI 并访问该页面。该页面用于初始化应用程序及其数据。单击 **Start initialization** 按钮。这将使您转至受密码保护的配置页面。您用于登录的电子邮件地址会被视为系统管理员的标识。请使用您早先配置的电子邮件地址和密码。

3. 在配置页面中，输入姓名（用于问候）、GitHub 用户名和先前生成的访问令牌。单击 **Initialize**。这将创建数据库表，并插入一些配置值。最后，会为系统管理员和租户创建数据库记录。
   ![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. 完成后，您将转至受管存储库的列表。现在，可以通过提供 GitHub 帐户或组织的名称以及存储库的名称来添加存储库。输入数据后，单击 **Add repository**。存储库以及新分配的标识应该会显示在表中。可以通过输入存储库标识并单击 **Delete repository**，从系统中除去存储库。
![](images/solution24-github-traffic-analytics/RepositoryList.png)

## 部署 Cloud Functions 和触发器
实施管理应用程序后，请为 {{site.data.keyword.openwhisk_short}} 部署操作、触发器以及用于连接操作和触发器的规则。这些对象用于按指定的安排自动收集 GitHub 流量数据。操作会连接到数据库，迭代所有租户及其存储库，并获取每个存储库的视图和克隆数据。这些统计信息会合并到数据库中。

1. 切换到 **functions** 目录。
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. 创建新操作 **collectStats**。此操作将使用已包含必需数据库驱动程序的 [Python 3 环境](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments)。操作的源代码在 `ghstats.zip` 文件中提供。
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   如果修改了操作的源代码 (`__main__.py`)，那么可以再次使用 `zip -r ghstats.zip  __main__.py github.py` 来重新打包 zip 归档。有关详细信息，请参阅 `setup.sh` 文件。
   {:tip}
3. 将操作绑定到数据库服务。请使用在环境设置期间创建的实例和服务密钥。
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. 基于[警报包](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm)创建触发器。它支持以不同形式指定警报。请使用类似 [cron](https://en.wikipedia.org/wiki/Cron) 的样式。下面的触发器每天早上 6 点 (UTC) 触发，从 4 月 21 日开始，到 12 月 21 日结束。请确保开始日期是未来的日期。
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  可以通过应用 `"0 6 * * 0"` 将触发器从每日安排更改为每周安排。这将使触发器在每周日早上 6 点触发。
  {:tip}
5. 最后，创建 **myStatsRule** 规则，用于将 **myDaily** 触发器连接到 **collectStats** 操作。现在，触发器会按先前步骤中指定的安排执行。
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. 调用用于初始测试运行的操作。返回的 **repoCount** 应该会反映出早先配置的存储库数。
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
   输出将类似于以下内容：
   ```
   {
       "repoCount": 18
   }
   ```
7. 在包含应用程序页面的浏览器窗口中，现在可以访问存储库流量。缺省情况下，将显示 10 个条目。可以将其更改为其他值。还可以对表列进行排序，或使用搜索框来过滤特定存储库。可以输入日期和组织名称，然后按 viewcount 排序以列出特定日期得分最高者。
   ![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## 总结
在本教程中，您部署了无服务器操作以及相关的触发器和规则。通过这些对象，可以自动检索 GitHub 存储库的流量数据。有关这些存储库的信息（包括特定于租户的访问令牌）存储在 SQL 数据库 ({{site.data.keyword.dashdbshort}}) 中。Cloud Foundry 应用程序使用该数据库来管理用户和存储库，并在应用程序门户网站中显示流量统计信息。用户可以在可搜索的表中查看流量统计信息，也可以在嵌入式仪表板（{{site.data.keyword.dynamdashbemb_short}} 服务，请参阅下图）中显示这些统计信息。还可以将存储库列表和流量数据下载为 CSV 文件。

Cloud Foundry 应用程序通过连接到 {{site.data.keyword.appid_short}} 的 OpenID Connect 客户机来管理访问。
![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## 清除
要清除本教程使用的资源，可以按照与创建过程相反的顺序来删除相关服务和应用程序以及操作、触发器和规则：

1. 删除 {{site.data.keyword.openwhisk_short}} 规则、触发器和操作。
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. 删除 Python 应用程序及其服务。
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## 扩展教程
想要为本教程添加内容或更改本教程？下面是一些构想：
* 扩展应用程序以实现多租户支持。
* 集成用于数据的图表。
* 使用社交身份提供者。
* 将日期选取器添加到统计信息页面以过滤显示的数据。
* 将定制登录页面用于 {{site.data.keyword.appid_short}}。
* 使用 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 探索开发者之间的社交编码关系。

## 相关内容
下面是与本教程中涵盖的主题相关的其他信息的链接。

文档和 SDK：
* [{{site.data.keyword.openwhisk_short}} 文档](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* 文档：[IBM Knowledge Center 上有关 {{site.data.keyword.dashdbshort}} 的信息](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.appid_short}} 文档](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [IBM Cloud 上的 Python 运行时](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
