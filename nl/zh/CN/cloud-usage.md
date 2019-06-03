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

# 查看 {{site.data.keyword.cloud_notm}} 服务、资源和使用情况
{: #cloud-usage}
随着云采用量的上升，IT 和财务管理人员需要掌握云的使用情况，以支持创新并控制成本。通过调查可用数据，可以回答诸如下面的问题：“团队在使用哪些服务？”、“运行一个解决方案需要多少成本？”以及“如何控制蔓生问题？”。本教程将介绍探索这些信息的方法，并回答与使用情况相关的常见问题。
{:shortdesc}

## 目标
{: #objectives}
* 逐项列记 {{site.data.keyword.cloud_notm}} 工件：Cloud Foundry 应用程序和服务、Identity and Access Management 资源以及基础架构设备
* 将 {{site.data.keyword.cloud_notm}} 工件与使用情况和计费相关联
* 定义 {{site.data.keyword.cloud_notm}} 工件与开发团队之间的关系
* 利用使用情况和计费数据来创建数据集以用于核算

## 体系结构
{: #architecture}

![体系结构](images/solution38/Architecture.png)

## 开始之前
{: #prereqs}

* 安装 [{{site.data.keyword.cloud_notm}} CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* 安装 [cURL](https://curl.haxx.se/)
* 安装 [Node.js](https://nodejs.org/)
* 使用 `npm install -g json2csv` 命令安装 [json2csv](https://www.npmjs.com/package/json2csv)
* 安装 [jq](https://stedolan.github.io/jq/)

## 背景
{: #background}

在执行对 {{site.data.keyword.cloud_notm}} 用途列出清单并进行详细描述的命令之前，先了解一些有关广泛类别的用途及其功能的背景知识会非常有用。本教程后面使用的关键术语均用粗体表示。在[管理帐户文档](https://{DomainName}/docs/account?topic=account-overview#overview)中可以找到以下工件的有用可视化。

### Cloud Foundry
Cloud Foundry 是 {{site.data.keyword.cloud_notm}} 上的一个开放式源代码平台即服务 (PaaS)，支持部署和缩放应用程序与**服务**，而无需管理服务器。Cloud Foundry 将应用程序和服务组织成组织或空间。**组织**是一个或多个用户可以拥有并使用的开发帐户。一个组织可以包含多个空间。每个**空间**为用户提供了对用于应用程序开发、部署和维护的共享位置的访问权。

### Identity and Access Management
通过 IBM Cloud Identity and Access Management (IAM)，您可以对这两个平台服务的用户进行安全认证，并在整个 {{site.data.keyword.cloud_notm}} 平台上以一致的方式来控制对资源的访问权。较新的产品和迁移的 Cloud Foundry 服务以**资源**形式存在，资源由 {{site.data.keyword.cloud_notm}} Identity and Access Management 进行管理。资源会组织成**资源组**，并通过策略和角色提供访问控制。

### 基础架构
基础架构包含各种计算选项：裸机服务器、虚拟服务器实例和 Kubernetes 节点。在控制台中，每个选项都被视为**设备**。

### 帐户
上述工件与**帐户**相关联，以便进行计费。可以邀请**用户**加入帐户并向其授予对帐户中不同资源的访问权。

## 分配许可权
要查看云清单和使用情况，您需要由帐户管理员分配的相应角色。如果您是帐户管理员，请继续执行下一部分。

1. 帐户管理员应该登录到 {{site.data.keyword.cloud_notm}}，然后访问 [**Identity & Access 用户**](https://{DomainName}/iam/#/users)页面。
2. 帐户管理员可以从帐户中的用户列表中选择您的姓名，以指定相应的许可权。
3. 在**访问策略**选项卡上，单击**分配访问权**按钮，然后执行以下更改。
   1. 选择**分配对资源组的访问权**。选择要向其授予访问权的**资源组**，然后应用**查看者**角色。通过单击**分配**按钮以完成操作。此角色对于 `resource groups` 和 `billing resource-group-usage` 命令是必需的。
   2. 选择**使用 Cloud Foundry 分配访问权**。选择要向其授予访问权的每个**组织**旁边的溢出菜单。从菜单中，选择**编辑组织角色**。从**组织角色**列表中，选择**记帐管理者**。通过单击**保存角色**按钮以完成操作。此角色对于 `billing org-usage` 命令是必需的。
   3. 选择**分配对资源的访问权**磁贴。在**服务**下拉菜单中，选择**所有启用“身份和访问权”的服务**。选中**分配平台访问角色**下的**编辑者**角色。此角色对于 `resource tag-attach` 命令是必需的。

## 使用 search 命令查找资源
{: #search}

随着开发团队开始使用云服务，管理人员可以通过了解部署了哪些服务而获益。部署信息有助于解答与创新和服务管理相关的问题：
- 可以跨团队共享哪些与服务相关的技能，从而改进其他项目？
- 不同团队有哪些共性可能在某些团队中缺乏？
- 哪些团队所使用的服务需要关键修复或者很快会变为不推荐使用？
- 团队如何复查自己的服务实例，以最大程度地控制蔓生问题？

搜索的范围不限于服务和资源。您还可以查询云工件，例如 Cloud Foundry 组织和空间、资源组、资源绑定、别名等。有关更多示例，请参阅 [ibmcloud resource search](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search) 文档。
{:tip}

1. 通过命令行登录到 {{site.data.keyword.cloud_notm}}，并将您的 Cloud Foundry 帐户设定为目标。请参阅 [CLI 入门](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)。
    ```sh
    ibmcloud login
    ```
    {: pre}

    ```sh
      ibmcloud target --cf
      ```
    {: pre}

2. 列出帐户中使用的所有 Cloud Foundry 服务的清单。
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. 可以应用布尔过滤器来扩大或缩小搜索范围。例如，使用以下查询来查找 Cloud Foundry 服务和应用程序以及 IAM 资源。

    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. 要通知使用特定服务类型的团队，请使用服务名称进行查询。将 `<name>` 替换为文本，例如 `weather` 或 `cloudant`。然后，通过将 `<Organization ID>` 替换为**组织标识**来获取组织名称。
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

标记和搜索可以一起使用，以提供资源的定制识别方式。这涉及：将标记附加到资源，然后使用标记名称进行搜索。请创建名为 env:tutorial 的标记。

1. 将该标记附加到资源。您可以通过用户界面或使用 `ibmcloud resource service-instance <name|id>` 来获取资源 CRN。云资源名称 (CRN) 用于唯一地标识 IBM Cloud 资源。
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <resource CRN>
   ```
   {: pre}
1. 使用以下查询来搜索与给定标记匹配的云工件。
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

通过将高级搜索查询与企业约定的标记模式组合使用，管理人员和团队负责人可以更轻松地识别云应用程序、资源和服务并对其执行操作。

## 通过使用情况仪表板探索使用情况
{: #dashboard}

管理层在了解了团队使用的服务后，下一个常问的问题就是“运营这些服务需要多少费用？”确定使用情况和成本的最直接方法是查看 {{site.data.keyword.cloud_notm}} 使用情况仪表板。

1. 登录到 {{site.data.keyword.cloud_notm}}，然后访问[帐户使用情况仪表板](https://{DomainName}/account/usage)。
2. 从**组**下拉菜单中，选择要查看其服务使用情况的 Cloud Foundry 组织。
3. 对于给定的**服务产品**，单击**查看实例**可查看已创建的服务实例。
4. 在以下页面上，选择一个实例，然后单击**查看实例**。生成的页面会提供有关该实例的详细信息，例如组织、空间和位置以及构成总成本的各个行项。
5. 使用面包屑来重新访问[使用情况仪表板](https://{DomainName}/account/usage)。
6. 从**组**下拉菜单中，将该选择器切换到**资源组**，然后选择一个组，例如 **default**。
7. 对可用实例执行类似的查看。

## 通过命令行探索使用情况

在此部分中，您将使用命令行界面来探索使用情况。

1. 列出可用的所有 Cloud Foundry 组织，然后设置一个环境变量以存储要测试的 Cloud Foundry 组织。
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<org name>
    ```
    {: pre}

2. 使用 `billing` 命令来逐项列记给定组织的计费和使用情况。使用 `-d` 标志与 YYYY-MM 格式的日期来指定特定月份。
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. 对资源执行相同的调查。每个帐户都有一个 `default` 资源组。使用值 `<group>` 替换第一个命令中列出的资源组。
    ```sh
    ibmcloud resource groups
    ```
    {: pre}

    ```sh
    export RESOURCE_GROUP=<group>
    ```
    {: pre}

    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP
    ```
    {: pre}

4. 可以使用 `resource-instances-usage` 命令来查看 Cloud Foundry 服务和 IAM 资源。请根据您的许可权级别，执行相应的命令。
    - 如果您是帐户管理员或已被授予对所有启用“身份和访问权”的服务的“管理员”角色，那么只需运行 `resource-instances-usage` 即可。
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  如果您不是帐户管理员，那么可以使用以下命令，因为在本教程开始部分已设置“查看者”角色。
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. 要查看基础架构设备，请使用 `sl` 命令。然后，使用 `vs` 命令来查看 {{site.data.keyword.virtualmachinesshort}}。
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## 通过命令行导出使用情况
{: #usagecmd}

管理层中有些人经常需要将数据导出到其他应用程序。常见的例子是将使用情况数据导出到电子表格中。在此部分中，您要将使用情况数据导出为逗号分隔值 (CSV) 格式，此格式与大多数电子表格应用程序兼容。

此部分将使用两个第三方工具：`jq` 和 `json2csv`。以下每个命令都由三个步骤组成：获取 JSON 格式的使用情况数据，解析 JSON，然后将结果的格式设置为 CSV。获取和转换 JSON 数据的过程不局限于使用 `json2csv` 来执行，还可以利用其他工具。

使用 `-p` 选项可采用精美格式显示结果。如果数据显示质量不佳，请除去 `-p` 自变量以显示原始 CSV 数据。
{:tip}

1. 导出资源组的使用情况，包括每种资源类型的预期成本。
    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. 逐项列记资源组中每种资源类型的实例。
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. 对 Cloud Foundry 组织采用相同的方法。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. 使用更高级的 `jq` 查询，向数据添加预期成本。这将创建更多行，因为某些资源类型具有多个成本度量值。
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. 使用相同的 `jq` 查询还可列出 Cloud Foundry 资源及关联的成本。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. 通过除去 `-p` 选项，并通过管道将输出传送到文件，可将数据导出到文件。生成的 CSV 文件可以使用电子表格应用程序打开。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## 通过 API 导出使用情况
{: #usageapi}

虽然 `billing` 命令很有用，但尝试使用命令行界面来组合“全局”视图会非常繁琐。与此类似，使用情况仪表板提供了组织和资源组的概述，但不一定是团队或项目的使用情况。在此部分中，您将开始探索数据驱动程度更高的方法，以根据定制需求来获取使用情况。

1. 在终端中，设置环境变量 `IBMCLOUD_TRACE=true` 来显示 API 请求和响应。
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. 重新运行 `billing org-usage` 命令以查看 API 调用。请注意，此单个命令可使用多个主机和 API 路径。
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. 获取 OAuth 令牌并执行在 `billing org-usage` 命令中显示的其中一个 API 调用。请注意，一些 API 使用 UAA 令牌，另一些 API 可能使用 IAM 令牌作为持有者授权。
    ```sh
    export IAM_TOKEN=`ibmcloud iam oauth-tokens | head -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    export UAA_TOKEN=`ibmcloud iam oauth-tokens | tail -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    curl -H "Authorization: Bearer $UAA_TOKEN" https://mccp.us-south.cf.cloud.ibm.com/v2/organizations?q=name:$ORG_NAME
    ```
    {: pre}

4. 要执行基础架构 API，请为[用户概要文件](https://control.softlayer.com/account/user/profile)中显示的 **API 用户名**和**认证密钥**设置环境变量。如果没有看到 API 用户名和认证密钥，那么可以通过[用户列表](https://control.softlayer.com/account/users)中您姓名旁边的**操作**菜单，创建 API 用户名和认证密钥。
    ```sh
    export IAAS_USERNAME=<API Username>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<Authentication Key>
    ```
    {: pre}

5. 使用以下 API 来获取基础架构计费总计和计费项目。[此处](https://softlayer.github.io/reference/services/SoftLayer_Account/)记录了类似的 API。
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. 禁用跟踪。
    ```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

虽然数据驱动方法在探索使用情况方面提供了最大的灵活性，但更详尽的说明已超出本教程的范围。创建了 GitHub 项目 [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample)，用于将 {{site.data.keyword.cloud_notm}} 服务与 Cloud Functions 组合使用，以提供样本实施。

## 扩展教程
{: #expand}

使用以下建议可将调查范围扩展到清单以及与使用情况相关的数据。

- 探索带 `--output json` 选项的 `ibmcloud billing` 命令。这将显示本教程中未涵盖的其他可用数据属性。
- 阅读博客帖子 [Using the {{site.data.keyword.cloud_notm}} command line to find resources](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/)，以了解有关 `ibmcloud resource search` 的更多示例，以及可以在查询中使用哪些属性。
- 查看 [Infrastructure Account APIs](https://softlayer.github.io/reference/services/SoftLayer_Account/)，以了解用于调查基础架构使用情况的其他 API。
- 查看 [jq Manual](https://stedolan.github.io/jq/manual/)，以了解用于聚集使用情况数据的高级查询。
