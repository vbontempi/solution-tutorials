---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# 关于组织用户、团队、应用程序的最佳实践
{: #users-teams-applications}

本教程概要介绍了 {{site.data.keyword.cloud_notm}} 中提供的用于管理身份和访问权管理的概念，以及如何实现这些概念以支持应用程序的多个开发阶段。
{:shortdesc}

在构建应用程序时，通常会定义多个环境以反映从开发者落实代码到应用程序代码可用于最终用户的整个项目开发生命周期。这些环境通常会使用如下名称：*Sandbox*、*test*、*staging*、*UAT*（用户接受测试）、*pre-production*、*production*。

隔离底层的资源，实施监管和访问策略，保护生产工作负载，验证更改后再将其推送到生产中，这些都是您需要创建这些单独环境的一些原因。

## 目标
{: #objectives}

* 了解 {{site.data.keyword.iamlong}} 和 Cloud Foundry 访问模型
* 配置项目并隔离各个角色和环境
* 设置连续集成

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 定义项目

让我们考虑一下带下列组件的样本项目：
* 在 {{site.data.keyword.containershort_notm}} 中部署的几个微服务、
* 数据库、
* 文件存储区。

在此项目中，我们定义了三个环境：
* *开发* - 此环境随着每次执行落实、单元测试、冒烟测试而持续更新。通过该环境，可以访问最新、最大的项目部署。
* *测试* - 此环境是依据代码的稳定分支或标记来构建的。在这里进行用户接受测试。其配置类似于生产环境。其中装入的是真实数据（以匿名的生产数据为例）。
* *生产* - 此环境使用先前环境中验证的版本进行更新。

**Delivery Pipeline 通过该环境来管理构建的进展。**它既可以完全自动化，也可以包含手动验证关卡以推进各环境之间的批准构建 - 这是真正开放式的，并且应该设置为与公司的最佳实践和工作流程相匹配。

为了支持执行构建管道，我们引入了**功能用户**，这是一个常规的 {{site.data.keyword.cloud_notm}} 用户，但也是在现实世界中没有真实身份的团队成员。此功能用户将拥有 Delivery Pipeline 以及需要强所有权的其他任何云资源。当团队成员离开公司或者调到其他项目时，此方法会很有帮助。功能用户将专用于您的项目，在项目生命周期内不会更改。接下来您需要为此功能用户创建 [API 密钥](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey)。在设置 DevOps 管道时或者在您要运行自动化脚本时，您将选择此 API 密钥，以模拟功能用户。

在为项目团队成员分配职责方面，让我们定义以下角色和相关许可权：

|           |开发|测试|生产|
| --------- | ----------- | ------- | ---------- |
|开发者| <ul><li>贡献代码</li><li>可以访问日志文件</li><li>可以查看应用程序和服务配置</li><li>使用已部署的应用程序</li></ul> | <ul><li>可以访问日志文件</li><li>可以查看应用程序和服务配置</li><li>使用已部署的应用程序</li></ul> | <ul><li>无访问权</li></ul> |
|测试者| <ul><li>使用已部署的应用程序</li></ul> | <ul><li>使用已部署的应用程序</li></ul> | <ul><li>无访问权</li></ul> |
|操作员| <ul><li>可以访问日志文件</li><li>可以查看/设置应用程序和服务配置</li></ul> | <ul><li>可以访问日志文件</li><li>可以查看/设置应用程序和服务配置</li></ul> | <ul><li>可以访问日志文件</li><li>可以查看/设置应用程序和服务配置</li></ul> |
|管道功能用户| <ul><li>可以部署/取消部署应用程序</li><li>可以查看/设置应用程序和服务配置</li></ul> | <ul><li>可以部署/取消部署应用程序</li><li>可以查看/设置应用程序和服务配置</li></ul> | <ul><li>可以部署/取消部署应用程序</li><li>可以查看/设置应用程序和服务配置</li></ul> |

## Identity and Access Management (IAM)
{: #first_objective}

通过 {{site.data.keyword.iamshort}} (IAM)，您可以安全地对平台服务和基础架构服务的用户进行认证，并在整个 {{site.data.keyword.cloud_notm}} 平台上以一致的方式控制对**资源**的访问权。支持一组 {{site.data.keyword.cloud_notm}} 服务使用 Cloud IAM 进行访问控制，这些服务在您的**帐户**中组织成**资源组**，让**用户**能够快速、轻松地一次访问多个资源。Cloud IAM 访问**策略**用于为用户和服务标识分配对帐户中资源的访问权。

**策略**通过属性组合来定义访问权的作用域，从而为用户或服务标识分配一个或多个**角色**。策略可以提供对单个服务下至实例级别的访问权，策略也可以应用于组织成资源组的一组资源。根据分配的用户角色，允许用户或服务标识具有不同级别的访问权，以通过使用 UI 或执行特定类型的 API 调用来完成平台管理任务或访问服务。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="IAM 模型图" />
</p>

目前还不能使用 IAM 来管理 {{site.data.keyword.cloud_notm}} 目录中的所有服务。对于这些服务，您可以继续使用 Cloud Foundry，方法是通过分配 Cloud Foundry 角色来定义允许的访问级别，为用户提供对实例所属组织和空间的访问权。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Cloud Foundry 模型图" />
</p>

## 为一个环境创建资源

虽然此样本项目所需的三种环境要求不同的访问权，还可能需要分配不同的容量，但它们共用同一个体系结构模式。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="显示一个环境的体系结构图" />
</p>

我们先来构建开发环境。

1. [选择一个要用于部署环境的 {{site.data.keyword.cloud_notm}} 位置](https://{DomainName})。
1. 对于 Cloud Foundry 服务和应用程序：
   1. [为项目创建组织](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg)。
   1. [为环境创建 Cloud Foundry 空间](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo)。
   1. 在此空间下创建项目要使用的 Cloud Foundry 服务
1. [为环境创建资源组](https://{DomainName}/account/resource-groups)。
1. 在此组中创建与资源组兼容的服务，如 {{site.data.keyword.cos_full_notm}}、{{site.data.keyword.la_full_notm}}、{{site.data.keyword.mon_full_notm}} 和 {{site.data.keyword.cloudant_short_notm}}。
1. 在 {{site.data.keyword.containershort_notm}} 中[创建新的 Kubernetes 集群](https://{DomainName}/containers-kubernetes/catalog/cluster)，确保选择上面创建的资源组。
1. 配置 {{site.data.keyword.la_full_notm}} 和 {{site.data.keyword.mon_full_notm}} 以发送日志并监视集群。

下图显示在帐户下创建项目资源的位置：

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="显示项目资源的图" />
</p>

## 在环境中分配角色

1. 邀请用户加入帐户
1. 为用户分配策略以控制谁可以访问资源组、组中的服务以及 {{site.data.keyword.containershort_notm}} 实例及其许可权。请参阅[访问策略定义](https://{DomainName}/docs/containers?topic=containers-users#access_policies)来为环境中的用户选择正确的策略。可以将具有相同策略集的用户放入[相同的访问组](https://{DomainName}/docs/iam?topic=iam-groups#groups)中。这样就简化了用户管理，因为策略将分配给访问组，并由组中的所有用户继承。
1. 在环境中根据需求来配置 Cloud Foundry 组织和空间角色。请参阅[角色定义](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess)来根据环境分配正确的角色。

请参阅服务文档，了解服务如何将 IAM 和 Cloud Foundry 角色映射到特定操作。请参阅示例 [{{site.data.keyword.mon_full_notm}} 服务如何将 IAM 角色映射到操作](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam)。

为用户分配正确的角色需要进行多次迭代和改进。给定的许可权可以在资源组级别进行控制，适用于一个组中的所有资源，或者细化到服务的特定实例，经过一段时间之后，您会发现哪些访问策略对于您的项目比较理想。

好的做法是从先从一组最少的许可权开始，然后再根据需要慎重扩展。对于 Kubernetes，您应该看一下它的[基于角色的访问控制 (RBAC)](https://kubernetes.io/docs/admin/authorization/rbac/) 以配置集群内授权。

对于开发环境，早先定义的用户责任可以转换为下列内容：

|           | IAM 访问策略|Cloud Foundry|
| --------- | ----------- | ------- |
|开发者| <ul><li>资源组：*查看者*</li><li>资源组中的平台访问角色：*查看者*</li><li>日志记录和监视服务角色：*写入者*</li></ul> | <ul><li>组织角色：*审计员*</li><li>空间角色：*审计员*</li></ul> |
|测试者| <ul><li>无需配置。测试者访问的是部署的应用程序，而不是开发环境</li></ul> | <ul><li>无需配置</li></ul> |
|操作员| <ul><li>资源组：*查看者*</li><li>资源组中的平台访问角色：*操作员*和*查看者*</li><li>日志记录和监视服务角色：*写入者*</li></ul> | <ul><li>组织角色：*审计员*</li><li>空间角色：*开发者*</li></ul> |
|管道功能用户| <ul><li>资源组：*查看者*</li><li>资源组中的平台访问角色：*编辑者*和*查看者*</li></ul> | <ul><li>组织角色：*审计员*</li><li>空间角色：*开发者*</li></ul> |

IAM 访问策略和 Cloud Foundry 角色是在 [Identify and Access Management 用户界面](https://{DomainName}/iam/#/users)中定义的：

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="针对开发者角色配置许可权" />
</p>

## 复制多个环境

在该处，您可以重复类似步骤以构建其他环境。

1. 每个环境创建一个资源组。
1. 每个环境创建一个集群和需要的服务实例。
1. 每个环境创建一个 Cloud Foundry 空间。
1. 在每个空间中创建需要的服务实例。

<p style="text-align: center;">
  <img title="使用单独的集群以隔离环境" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="显示单独集群以隔离环境的图" />
</p>

使用工具组合（如 [{{site.data.keyword.cloud_notm}} `ibmcloud` CLI](https://github.com/IBM-Cloud/ibm-cloud-developer-tools)、[HashiCorp 的 `terraform`](https://www.terraform.io/)、[用于 Terraform 的 {{site.data.keyword.cloud_notm}} 提供者](https://github.com/IBM-Cloud/terraform-provider-ibm)、Kubernetes CLI `kubectl`），您可以通过编制脚本来自动创建这些环境。

环境的单独 Kubernetes 集群随附一些有用的属性：
* 无论环境如何，所有集群的外观都差不多；
* 更容易控制谁有权访问特定集群；
* 部署和底层资源的更新周期更灵活；有新的 Kubernetes 版本时，您可以选择先更新开发集群，验证应用程序，然后再更新其他环境；
* 避免了将可能会相互影响的不同工作负载混合在一起，如将生产部署与其他部署隔离开。

还有一种方法是使用 [Kubernetes 名称空间](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)以及 [Kubernetes 资源配额](https://kubernetes.io/docs/concepts/policy/resource-quotas/)来隔离环境并控制资源消耗。

<p style="text-align: center;">
  <img title="使用单独的名称空间来隔离环境" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="显示单独名称空间以隔离环境的图" />
</p>

在 LogDNA UI 的`搜索`输入框中，使用`名称空间：`字段来根据名称空间过滤日志。
{: tip}

## 设置 Delivery Pipeline

在部署到其他环境中时，可以将您的持续集成/持续交付管道设置为驱动整个流程：
* 使用来自 `development` 分支的最新、最好的代码持续更新`开发`环境，并在专用集群上运行单元测试和集成测试。
* 将开发构建升级到`测试`环境，可以是自动完成（如果先前各阶段中的所有测试都没有问题），也可以是通过手动升级流程来完成。有些团队在此处也会使用不同的分支，将有效的开发状态合并到 `stable` 分支作为示例；
* 重复类似的过程以移动到`生产`环境。

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="从构建到部署的 CI/CD 管道" />
</p>

在配置 DevOps 管道时，确保使用功能用户的 API 密钥。只有功能用户应该需要具有将应用程序部署到集群所需的权限。

在构建阶段，请务必正确地对 Docker 映像进行版本控制。您可以使用 Git 落实修订版作为映像标记的一部分，也可以使用 DevOps 工具链提供的唯一标识；只要有助于轻松将映像映射到该映像中包含的实际构建和源代码，任何标识都可以。

在熟悉 Kubernetes 之后，Kubernetes 的软件包管理器 [Helm](https://helm.sh/) 就会成为对应用程序进行版本控制、组合和部署的便捷工具。[此样本 DevOps 工具链](https://github.com/open-toolchain/simple-helm-toolchain)很适合作为起点，并且预先配置了针对 Kubernetes 集群的持续交付。随着项目增长为多个微服务，[Helm 伞形 chart](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies) 将为编写应用程序提供不错的解决方案。

## 扩展教程

祝贺您，您的应用程序现在可以安全地从开发部署到生产。下面是改进应用程序交付的其他建议。

* 将 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 添加到管道以在部署期间执行质量控制。
* 使用 [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) 复查团队成员的编码贡献情况以及开发者之间的交互。
* 按照教程[计划、创建并更新部署环境](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments)以实现环境部署自动化。

## 相关信息

* [{{site.data.keyword.iamshort}} 入门](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [使用资源组来组织资源的最佳实践](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [使用 LogDNA 和 Sysdig 分析日志并监视运行状况](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [持续部署到 Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Hello Helm 工具链](https://github.com/open-toolchain/simple-helm-toolchain)
* [使用 Kubernetes 和 Helm 开发微服务应用程序](https://github.com/open-toolchain/microservices-helm-toolchain)
* [授予用户查看 LogDNA 中日志的许可权](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [授予用户查看 Sysdig 中度量值的许可权](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
