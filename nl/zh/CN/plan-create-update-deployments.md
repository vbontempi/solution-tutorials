---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# 规划、创建和更新部署环境
{: #plan-create-update-deployments}

构建解决方案时，多个部署环境十分常见。这些环境反映了从开发到生产的项目生命周期。本教程将介绍 {{site.data.keyword.Bluemix_notm}} CLI 和 [Terraform](https://www.terraform.io/) 等工具，以自动创建和维护这些部署环境。
{:shortdesc}

开发者很讨厌重复编写相同的东西。[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 原则就是其中一个例子。与此类似，开发者也不喜欢在用户界面中必须通过大量点击来设置环境。因此，系统管理员和开发者长期以来一直是使用 shell 脚本自动执行重复性、易出错和无趣的任务。

基础架构即服务 (IaaS)、平台即服务 (PaaS)、容器即服务 (CaaS) 和函数即服务 (FaaS) 为开发者提供了高级别的抽象，支持更轻松地获取各种资源，如裸机服务器、受管数据库、虚拟机、Kubernetes 集群等。但是，供应了这些资源后，需要将其连接在一起，配置用户访问权，随时间变化更新配置等。能够自动执行所有这些步骤并在不同环境下重复执行安装和配置是如今必不可少的任务。

在一个项目中有多个环境十分常见，这些环境可支持开发周期的不同阶段，各个环境之间在容量、联网、凭证、日志详细程度等方面略有差异。在[其他教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)中，我们已经介绍过组织用户、团队和应用程序的最佳实践以及一个样本场景。该样本场景考虑了三个环境：*开发*、*测试*和*生产*。如何自动创建这些环境？可以使用哪些工具？

## 目标
{: #objectives}

* 定义要部署的一组环境
* 使用 {{site.data.keyword.Bluemix_notm}} CLI 和 [Terraform](https://www.terraform.io/) 编写脚本以自动部署这些环境
* 在帐户中部署这些环境

## 使用的服务
{: #services}

本教程使用以下产品：
* [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Identity and Access Management](https://{DomainName}/iam/#/users)
* [{{site.data.keyword.Bluemix_notm}} 命令行界面 - `ibmcloud` CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. 创建一组 Terraform 文件，用于描述目标基础架构即代码。
1. 操作员使用 `terraform apply` 来供应各个环境。
1. 编写 shell 脚本以完成环境配置。
1. 操作员针对环境运行脚本。
1. 环境已完全配置，可供使用。

## 可用工具概述
{: #tools}

第一个与 {{site.data.keyword.Bluemix_notm}} 进行交互并创建可重复部署的工具是 [{{site.data.keyword.Bluemix_notm}} 命令行界面 - `ibmcloud` CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)。借助 `ibmcloud` 及其插件，可以自动创建和配置云资源。通过命令行，可以供应 {{site.data.keyword.virtualmachinesshort}}、Kubernetes 集群、{{site.data.keyword.openwhisk_short}}、Cloud Foundry 应用程序和服务等所有这些资源。

[本教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)中介绍的另一个工具是 HashiCorp 的产品 [Terraform](https://www.terraform.io/)。据 HashiCorp 称，*通过 Terraform，能够以可预测的方式安全地创建、更改和改进基础架构。Terraform 是一个开放式源代码工具，用于将 API 编码为声明式配置文件，这些文件可以在团队成员之间共享，视为代码，进行编辑、复查和版本化。*这属于基础架构即代码。您写下基础架构应该具备的样子，随后 Terraform 将根据需要创建、更新和除去云资源。

为了支持多云方法，Terraform 使用多个提供者。提供者负责了解 API 交互并公开资源。{{site.data.keyword.Bluemix_notm}} 具有自己的[用于 Terraform 的提供者](https://github.com/IBM-Cloud/terraform-provider-ibm)，支持 {{site.data.keyword.Bluemix_notm}} 用户使用 Terraform 来管理资源。虽然 Terraform 被分类为基础架构即代码，但适用范围并不局限于基础架构即服务资源。{{site.data.keyword.Bluemix_notm}} Provider for Terraform 支持 IaaS（裸机、虚拟机、网络服务等）、CaaS（{{site.data.keyword.containershort_notm}} 和 Kubernetes 集群）、PaaS（Cloud Foundry 和服务）和 FaaS ({{site.data.keyword.openwhisk_short}}) 资源。

## 编写自动部署脚本
{: #scripts}

开始描述基础架构即代码时，请务必将创建的文件视为常规代码，从而将其存储在源代码控制管理系统中。这样在一段时间后就能体现出各种优点，例如使用源代码控制复查工作流程验证更改之后再应用更改，添加持续集成管道以自动部署基础架构更改。

[此 Git 存储库](https://github.com/IBM-Cloud/multiple-environments-as-code)具有设置早先定义的环境所需的全部配置文件。可以克隆此存储库，以按照详细描述文件内容的以下各部分进行操作。

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

存储库的结构如下所示：

|目录|描述|
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) |Terraform 文件的主目录|
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) |用于供应三种环境的通用资源的 Terraform 文件|
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) |特定于给定环境的 Terraform 文件|
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) |用于配置用户策略的 Terraform文件|

### 通过 Terraform 处理棘手任务

*开发*、*测试*和*生产*环境看上去在很多方面都一样。

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="显示一个部署环境的图" />
</p>

各个环境共享一个共同的组织和特定于环境的资源。环境在分配的容量和访问权方面有所不同。这些特点反映在 Terraform 文件中，***global*** 配置用于供应 Cloud Foundry 组织，***per-environment*** 配置使用 Terraform 工作空间来供应特定于环境的资源：

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### 全局配置

所有环境都共享一个共同的 Cloud Foundry 组织，每个环境都有其自己的空间。

在 [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) 目录下，可以找到 Terraform 脚本来供应此组织。[main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf) 包含组织的定义：

   ```sh
   # create a new organization for the project
   resource "ibm_org" "organization" {
     name             = "${var.org_name}"
     managers         = "${var.org_managers}"
     users            = "${var.org_users}"
     auditors         = "${var.org_auditors}"
     billing_managers = "${var.org_billing_managers}"
   }
   ```

在此资源中，所有属性均通过变量进行配置。在以下各部分中，您将了解如何设置这些变量。

要完全部署环境，将组合使用 Terraform 和 {{site.data.keyword.Bluemix_notm}} CLI。使用 CLI 编写的 shell 脚本可能需要引用按名称或标识来引用此组织或帐户。*global* 目录还包含 [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf)，这用于生成一个文件，其中将此信息作为适合在脚本编制中复用的键/值包含在内：

   ```sh
   # generate a property file suitable for shell scripts with useful variables relating to the environment
   resource "local_file" "output" {
     content = <<EOF
ACCOUNT_GUID=${data.ibm_account.account.id}
ORG_GUID=${ibm_org.organization.id}
ORG_NAME=${var.org_name}
EOF

     filename = "../outputs/global.env"
   }
   ```

### 不同的环境

使用 Terraform 来管理多个环境有多种不同的方法。可以在单独的目录下复制 Terraform 文件，每个环境一个目录。通过 [Terraform 模块](https://www.terraform.io/docs/modules/index.html)，可以将公共配置视为一个组，并在不同环境中复用模块，从而减少代码重复。单独的目录意味着可以逐渐改进*开发*环境来测试更改，然后将更改传播到其他环境。在这种情况下，Terraform *模块*也常常位于其自己的源代码存储库中，以便可以在环境文件中引用模块的特定版本。

考虑到各个环境相当简单且类似，您将使用另一个名为[工作空间](https://www.terraform.io/docs/state/workspaces.html#best-practices)的 Terraform 概念。通过工作空间，可以在不同环境中使用相同的 Terraform 文件 (.tf)。在示例中，*development*、*testing* 和 *production* 是工作空间。它们将使用相同的 Terraform 定义，但具有不同的配置变量（不同的名称和不同的容量）。

每个环境需要：
* 一个专用 Cloud Foundry 空间
* 一个专用资源组
* 一个 Kubernetes 集群
* 一个数据库
* 一个文件存储器

Cloud Foundry 空间已链接到上一步中创建的组织。环境 Terraform 文件需要引用此组织。这正是 [Terraform 远程状态](https://www.terraform.io/docs/state/remote.html)的用武之地。它允许以只读方式引用现有 Terraform 状态。这是一个非常有用的构造，支持将 Terraform 配置拆分成更小的部分，从而将各个部分的责任分配给不同的团队。[backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf) 包含 *global* 远程状态的定义，用于查找早先创建的组织：

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

可以引用组织后，就可以直接在此组织中创建空间。[main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf) 包含针对环境的资源定义。

   ```sh
   # a Cloud Foundry space per environment
   resource "ibm_space" "space" {
     name       = "${var.environment_name}"
     org        = "${data.terraform_remote_state.global.org_name}"
     managers   = "${var.space_managers}"
     auditors   = "${var.space_auditors}"
     developers = "${var.space_developers}"
   }
   ```

请注意 *global* 远程状态中是如何引用组织名称的。其他属性通过配置变量获取。

接下来是资源组。

   ```sh
   # a resource group
   resource "ibm_resource_group" "group" {
    name     = "${var.environment_name}"
    quota_id = "${data.ibm_resource_quota.quota.id}"
}

   data "ibm_resource_quota" "quota" {
	name = "${var.resource_quota}"
}
   ```

Kubernetes 集群在此资源组中进行创建。{{site.data.keyword.Bluemix_notm}} 提供者具有 Terraform 资源来表示集群：

   ```sh
  # a cluster
  resource "ibm_container_cluster" "cluster" {
  	name              = "${var.environment_name}-cluster"
  	datacenter        = "${var.cluster_datacenter}"
  	org_guid          = "${data.terraform_remote_state.global.org_guid}"
  	space_guid        = "${ibm_space.space.id}"
  	account_guid      = "${data.terraform_remote_state.global.account_guid}"
  	hardware          = "${var.cluster_hardware}"
  	machine_type      = "${var.cluster_machine_type}"
  	public_vlan_id    = "${var.cluster_public_vlan_id}"
  	private_vlan_id   = "${var.cluster_private_vlan_id}"
  	resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool" "cluster_workerpool" {
  worker_pool_name  = "${var.environment_name}-pool"
  machine_type      = "${var.cluster_machine_type}"
  cluster           = "${ibm_container_cluster.cluster.id}"
  size_per_zone     = "${var.worker_num}"
  hardware          = "${var.cluster_hardware}"
  resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool_zone_attachment" "cluster_zone" {
  cluster           = "${ibm_container_cluster.cluster.id}"
  worker_pool       =  "${element(split("/",ibm_container_worker_pool.cluster_workerpool.id),1)}"
  zone              = "${var.cluster_datacenter}"
  public_vlan_id    = "${var.cluster_public_vlan_id}"
  private_vlan_id   = "${var.cluster_private_vlan_id}"
  resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

同样，大多数属性将通过配置变量进行初始化。可以调整数据中心、工作程序数和工作程序类型。

{{site.data.keyword.cos_full_notm}} 和 {{site.data.keyword.cloudant_short_notm}} 等启用 IAM 的服务也作为该组内的资源进行创建：

   ```sh
# a database
resource "ibm_resource_instance" "database" {
    name              = "database"
    service           = "cloudantnosqldb"
    plan              = "${var.cloudantnosqldb_plan}"
    location          = "${var.cloudantnosqldb_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
# a cloud object storage
resource "ibm_resource_instance" "objectstorage" {
    name              = "objectstorage"
    service           = "cloud-object-storage"
    plan              = "${var.cloudobjectstorage_plan}"
    location          = "${var.cloudobjectstorage_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

可以添加 Kubernetes 绑定（私钥），以从应用程序中检索服务凭证：

   ```sh
   # bind the cloudant service to the cluster
   resource "ibm_container_bind_service" "bind_database" {
      cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	  service_instance_name       = "${ibm_resource_instance.database.name}"
      namespace_id                = "default"
      account_guid                = "${data.terraform_remote_state.global.account_guid}"
      org_guid                    = "${data.terraform_remote_state.global.org_guid}"
      space_guid                  = "${ibm_space.space.id}"
      resource_group_id           = "${ibm_resource_group.group.id}"
}

   # bind the cloud object storage service to the cluster
   resource "ibm_container_bind_service" "bind_objectstorage" {
  	cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	space_guid                  = "${ibm_space.space.id}"
  	service_instance_id         = "${ibm_resource_instance.objectstorage.name}"
  	namespace_id                = "default"
  	account_guid                = "${data.terraform_remote_state.global.account_guid}"
  	org_guid                    = "${data.terraform_remote_state.global.org_guid}"
  	space_guid                  = "${ibm_space.space.id}"
  	resource_group_id           = "${ibm_resource_group.group.id}"
}
   ```

## 在帐户中部署此环境

### 安装 {{site.data.keyword.Bluemix_notm}} CLI

1. 按照[这些指示信息](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)来安装 CLI。
1. 通过运行以下命令验证安装：
   ```sh
   ibmcloud
   ```
   {: codeblock}

### 安装 Terraform 和 {{site.data.keyword.Bluemix_notm}} Provider for Terraform

1. [下载并安装适用于您系统的 Terraform](https://www.terraform.io/intro/getting-started/install.html)。
1. [下载适用于 {{site.data.keyword.Bluemix_notm}} 提供者的 Terraform 二进制文件](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)。
   要设置适用于 {{site.data.keyword.Bluemix_notm}} 提供者的 Terraform，请参阅此[链接](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup)
   {:tip}
1. 在指向 Terraform 二进制文件的主目录中创建 `.terraformrc` 文件。在以下示例中，`/opt/provider/terraform-provider-ibm` 是目录的路径。
   ```sh
   # ~/.terraformrc
   providers {
     ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### 获取代码

如果尚未克隆教程存储库，请进行克隆：

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### 设置平台 API 密钥

1. 获取[平台 API 密钥](https://{DomainName}/iam/#/apikeys)并保存 API 密钥以供未来参考（如果尚无此密钥）。

   > 如果在后续步骤中计划创建新的 Cloud Foundry 组织来托管部署环境，请确保您是帐户的所有者。
1. 通过运行以下命令，将 [terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl) 复制到 *terraform/credentials.tfvars*：
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. 编辑 `terraform/credentials.tfvars`，并将 `ibmcloud_api_key` 的值设置为获取的平台 API 密钥。

### 创建或复用 Cloud Foundry 组织

可以选择创建新组织或复用（导入）现有组织。要创建三个部署环境的父组织，**您需要是帐户所有者**。

#### 创建新组织

1. 切换到 `terraform/global` 目录
1. 将 [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) 复制到 `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. 编辑 `global.tfvars`
   1. 将 **org_name** 设置为要创建的组织名称
   1. 将 **org_managers** 设置为要向其授予组织中*管理者*角色的用户标识的列表 - 创建组织的用户会自动成为管理者，因此不应添加到此列表中
   1. 将 **org_users** 设置为要邀请加入组织的所有用户的列表 - 对于要在后续步骤中配置其访问权的用户，需要在此处添加这些用户

   ```sh
   org_name = "a-new-organization"
   org_managers = [ "user1@domain.com", "another-user@anotherdomain.com" ]
   org_users = [ "user1@domain.com", "another-user@anotherdomain.com", "more-user@domain.com" ]
   ```
   {: codeblock}
1. 从 `terraform/global` 文件夹初始化 Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. 查看 Terraform 套餐
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. 应用更改
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Terraform 完成后，以下各项就已经创建：
* 新的 Cloud Foundry 组织
* 检出中 `outputs` 目录下的一个 `global.env` 文件。此文件包含可以在其他脚本中引用的环境变量
* `terraform.tfstate` 文件

> 对于 Terraform 状态，本教程使用的是 `local` 后端提供者。这在发现 Terraform 或单独在项目上工作时很方便，但以团队形式或在更大型基础架构上工作时，Terraform 还支持将状态保存到远程位置。由于 Terraform 状态对于 Terraform 操作至关重要，因此建议使用远程高可用性弹性存储器来存储 Terraform 状态。有关可用选项的列表，请参阅 [Terraform 后端类型](https://www.terraform.io/docs/backends/types/index.html)。一些后端甚至支持对 Terraform 状态进行版本控制和锁定。

#### 复用所管理的组织

如果您不是帐户所有者，但负责管理帐户中的组织，那么还可以将现有组织导入到 Terraform 中。

1. 检索组织 GUID
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. 切换到 `terraform/global` 目录
1. 将 [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) 复制到 `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. 初始化 Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. 初始化 Terraform 后，将组织导入到 Terraform 状态中
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. 调整 `global.tfvars` 以匹配现有组织名称和结构
1. 应用更改
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### 创建 per-environment 空间、集群和服务

此部分将重点介绍 `development` 环境。对于其他环境，步骤是相同的，只是为变量选取的值有所不同。

1. 切换到检出的 `terraform/per-environment` 文件夹
1. 复制模板 `tfvars` 文件。每个环境都有一个 tfvars 文件：
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. 编辑 `development.tfvars`
   1. 将 **environment_name** 设置为要创建的 Cloud Foundry 空间的名称
   1. 将 **space_developers** 设置为此空间的开发者列表。**确保将您的姓名添加到此列表中，以便 Terraform 可以代表您供应服务。**
   1. 将 **cluster_datacenter** 设置为要在其中创建集群的位置。使用以下命令查找可用的位置：
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. 为集群设置专用 (**cluster_private_vlan_id**) 和公用 (**cluster_public_vlan_id**) VLAN。使用以下命令查找位置可用的 VLAN：
      ```sh
      ibmcloud cs vlans <location>
      ```
      {: codeblock}
   1. 设置 **cluster_machine_type**。使用以下命令查找位置的可用机器类型和特征：
      ```sh
      ibmcloud cs machine-types <location>
      ```
      {: codeblock}

   1. 设置 **resource_quota**。使用以下命令查找可用的资源配额定义：
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. 初始化 Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. 为 *development* 环境创建新的 Terraform 工作空间
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   稍后可切换使用各个环境
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. 查看 Terraform 套餐
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   应该会报告为：
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. 应用更改
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Terraform 完成后，以下各项就已经创建：
* 一个资源组
* 一个 Cloud Foundry 空间
* 一个 Kubernetes 集群，连接有一个工作程序池和一个专区
* 一个数据库
* 包含数据库凭证的一个 Kubernetes 私钥
* 一个存储器
* 包含存储器凭证的一个 Kubernetes 私钥
* 一个日志记录 (LogDNA) 实例
* 一个监视 (Sysdig) 实例
* 检出中 `outputs` 目录下的一个 `development.env` 文件。此文件包含可以在其他脚本中引用的环境变量
* `terraform.tfstate.d/development` 下特定于环境的 `terraform.tfstate`。

您可以对 `testing` 和 `production` 重复这些步骤。

### 分配用户策略

在先前的步骤中，Cloud Foundry 组织和空间中的角色可以使用 Terraform 提供者进行配置。对于其他资源（如 Kubernetes 集群）上的用户策略，将使用克隆的存储库中的 [roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) 文件夹。

对于[本教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)中定义的*开发*环境，要定义的策略如下：

|           |IAM 访问策略|
| --------- | ----------- |
|开发者| <ul><li>资源组：*查看者*</li><li>资源组中的平台访问角色：*查看者*</li><li>日志记录和监视服务角色：*写入者*</li></ul> |
|测试者| <ul><li>无需配置。测试者访问的是部署的应用程序，而不是开发环境</li></ul> |
|操作员| <ul><li>资源组：*查看者*</li><li>资源组中的平台访问角色：*操作员*和*查看者*</li><li>日志记录和监视服务角色：*写入者*</li></ul> |
|管道功能用户| <ul><li>资源组：*查看者*</li><li>资源组中的平台访问角色：*编辑者*和*查看者*</li></ul> |

考虑到一个团队可能有多个开发者和测试者，因此可以利用[访问组概念](https://{DomainName}/docs/iam?topic=iam-groups#groups)来简化用户策略的配置。访问组可以由帐户所有者进行创建，这样就可以使用单个策略将相同的访问权分配给组内的所有实体。

对于*开发*环境中的*开发者*角色，这将转换为：

   ```sh
resource "ibm_iam_access_group" "developer_role" {
  name        = "${var.access_group_name_developer_role}"
  description = "${var.access_group_description}"
}
resource "ibm_iam_access_group_policy" "resourcepolicy_developer" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Viewer"]

  resources = [{
    resource_type = "resource-group"
    resource      = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_platform_accesspolicy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles        = ["Viewer"]

  resources = [{
    resource_group_id = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_logging_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "logdna"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.logdna_instance_id}"
  }]
}
resource "ibm_iam_access_group_policy" "developer_monitoring_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "sysdig-monitor"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.sysdig_instance_id}"
  }]
}

   ```

检出中的 [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) 文件包含针对已定义*开发者*、*操作员*、*测试者*和*功能用户*角色的这些资源的示例。针对*开发*环境中具有*开发者、操作员、测试者和功能用户*角色的用户，设置上一部分中定义的策略：

1. 切换到 `terraform/roles/development` 目录
2. 复制模板 `tfvars` 文件。每个环境都有一个 tfvars 文件（可以在 `roles` 目录中 `production` 和 `testing` 模板的各自文件夹下找到这两个模板）

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. 编辑 `development.tfvars`

   - 将 **iam_access_members_developers** 设置为要向其授予访问权的开发者的列表。
   - 将 **iam_access_members_operators** 设置为操作员的列表，依此类推。
4. 初始化 Terraform
   ```sh
   terraform init
   ```
   {: codeblock}

5. 查看 Terraform 套餐
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   应该会报告为：
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. 应用更改
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
您可以对 `testing` 和 `production` 重复这些步骤。

## 除去资源

1. 导航至 `roles` 下的 `development` 文件夹
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. 销毁访问组和访问策略
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. 激活 `development` 工作空间
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. 销毁资源组、空间、服务和集群
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. 对 `testing` 和 `production` 工作空间重复这些步骤。
1. 如果创建了组织，请将其销毁
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## 相关内容

* [Terraform 教程](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Terraform 提供者](https://ibm-cloud.github.io/tf-ibm-docs/)
* [Examples using {{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
