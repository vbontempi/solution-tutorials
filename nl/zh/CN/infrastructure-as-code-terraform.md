---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-23"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# 使用 Terraform 部署 LAMP 堆栈
{: #infrastructure-as-code-terraform}

[Terraform](https://www.terraform.io/) 支持以安全且可预测的方式创建、更改和改进基础架构。Terraform 是一个开放式源代码工具，用于将 API 编码为声明式配置文件，这些文件可以在团队成员之间共享，视为代码，进行编辑、复查和版本化。

在本教程中，您将使用样本配置来供应 **L**inux 虚拟服务器（使用 **A**pache Web 服务器、**M**ySQL 和 **P**HP 服务器），这称为 **LAMP** 堆栈。然后，将更新配置以添加 {{site.data.keyword.cos_full_notm}} 服务并缩放资源以调整环境（内存、CPU 和磁盘大小）。最后将删除配置创建的所有资源。

## 目标
{: #objectives}

* 配置 Terraform 和 {{site.data.keyword.Bluemix_notm}} Provider for Terraform。
* 使用 Terraform 创建、更新、缩放并最终销毁 LAMP 堆栈配置。

## 使用的服务
{: #services}

本教程使用以下运行时和服务：
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

本教程可能会发生成本。请使用[定价计算器](https://{DomainName}/pricing/)来根据您预测的使用量生成成本估算。

## 体系结构
{: #architecture}

<p style="text-align: center;">

  ![体系结构图](images/solution10/architecture-2.png)
</p>

1. 创建一组 Terraform 文件来描述 LAMP 堆栈配置。
1. 从配置目录中调用 `terraform`。
1. `terraform` 调用 {{site.data.keyword.cloud_notm}} API 来供应资源。

## 开始之前
{: #prereqs}

请联系基础架构主用户以获取以下许可权：
- 网络许可权（用于添加**公共和专用网络上行链路**）
- API 密钥

## 先决条件

{: #prereq}

通过[安装程序](https://www.terraform.io/intro/getting-started/install.html)来安装 **Terraform**，或者在 MacOS 上使用 [Homebrew](https://brew.sh/) 运行命令 `brew install terraform` 来安装 Terraform。

在 **Windows** 上，执行以下步骤完成 Terraform 设置：
   1. 将下载的 zip 中的文件复制到 `C:\terraform`（创建 `terraform` 文件夹）。
   2. 以管理员身份打开命令提示符，然后将 PATH 设置为使用 Terraform 二进制文件。

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## 配置 {{site.data.keyword.Bluemix_notm}} Provider for Terraform
{: #setup}

为了支持多云方法，Terraform 使用多个提供者。提供者负责了解 API 交互并公开资源。在此部分中，您将配置 CLI 以指定 {{site.data.keyword.Bluemix_notm}} 提供者的位置。

1. 通过在终端或命令提示符窗口中运行 `terraform`，检查 Terraform 安装。您应该会看到`常用命令`的列表。

  ```
  terraform
  ```

2. 下载与系统相应的 [{{site.data.keyword.Bluemix_notm}} Provider](https://github.com/IBM-Cloud/terraform-provider-ibm/releases) 插件，并解压缩归档。您应该会看到 `terraform-provider-ibm_VERSION` 二进制插件文件。

3. 对于非 Windows 系统，请在用户的主目录中创建 `.terraform.d/plugins` 目录，并将二进制文件放入其中。使用以下命令可获取参考信息。

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    在 **Windows** 上，该文件需要放入用户的“Application Data”目录下的 `terraform.d/plugins` 中。

  - 在命令提示符处，运行以下命令来创建[提供者配置](https://www.terraform.io/docs/configuration/providers.html)
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - 启动 **Windows Powershell**（开始 + R > Powershell），然后运行以下命令来创建 `terraform.rc` 文件
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   在第一个提示符处，输入以下内容
   ```
    # ~/.terraformrc
    providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        PATH_TO_YOUR_APPDATA_PLUGINS 应该是包含正斜杠 (/) 的绝对路径。例如，`C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe`
        {: tip}

  - 单击 Enter 键以退出提示符。

## 准备 Terraform 配置

{: #terraformconfig}

在此部分中，您将使用 {{site.data.keyword.Bluemix_notm}} 提供的样本 Terraform 配置来了解 Terraform 配置的基础知识。

1. 访问 https://github.com/IBM-Cloud/LAMP-terraform-ibm，并将您自己的副本**派生**到您的帐户。
2. 在本地克隆派生：
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. 检查配置文件
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml) - 包含服务器安装配置，在此可以添加与服务器安装相关以及与要在服务器上安装的内容相关的所有脚本。请查看注入到此文件中的 `phpinfo();`。
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf) - 包含与提供者相关的变量，其中需要提供者的用户名和 API 密钥。
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) - 包含使用指定变量部署 VM 的服务器配置。
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) - 包含 **SoftLayer** 用户名和 API 密钥、{{site.data.keyword.Bluemix_notm}} API 密钥以及您的空间/组织名称。可以将这些凭证添加到此文件中以获得最佳实践，避免每次部署服务器时，都要从命令行重新输入这些凭证。注：不要使用您的凭证发布此文件。
4. 在所选的 IDE 中打开 [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) 文件，然后通过添加**公共 SSH** 密钥来修改该文件。此密钥将用于访问此配置创建的 VM。有关创建 SSH 密钥的信息，请按照[此链接](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html)中的指示信息进行操作。要将公用密钥复制到剪贴板，可以在终端中运行以下命令。

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     此命令将 SSH 复制到剪贴板，然后可以将其粘贴到 [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) 中大约第 69 行的 `ssh_key` 缺省变量下。

    在 **Windows** 上，下载、安装并启动 [Git Bash](https://git-scm.com/download/win)，然后运行以下命令，以将公共 SSH 密钥复制到剪贴板。

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. 使用 IDE 打开 [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) 文件，通过添加列出的所有凭证来修改该文件；在该文件中添加这些凭证意味着以后每次运行 terraform apply 时，无需重新输入这些凭证。为了完成本教程的其余部分，您必须添加 [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) 文件中列出的所有五个凭证。

## 通过 Terraform 配置创建 LAMP 堆栈服务器
{: #Createserver}
在此部分中，您将学习如何通过 Terraform 配置样本来创建 LAMP 堆栈服务器。该配置用于供应虚拟机实例，并将 **A**pache、**M**ySQL (**M**ariaDB) 和 **P**HP 安装在该实例上。

1. 导航至克隆的存储库所在的文件夹。
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. 初始化 Terraform 配置。这还将安装 `terraform-provider-ibm_VERSION` 插件。
   ````bash
    terraform init
   ````
   {: pre}
3. 应用 Terraform 配置。这将创建在配置中定义的资源。
   ```
    terraform apply
   ```
   {: pre}
   您应该会看到类似于下面的输出。![源代码控制 URL](images/solution10/created.png)
4. 接下来，转至[基础架构设备列表](https://{DomainName}/classic/devices)以验证服务器是否已创建。![源代码控制 URL](images/solution10/configuration.png)

## 添加 {{site.data.keyword.cos_full_notm}} 服务和缩放资源

{: #modify}

在此部分中，您将了解如何缩放虚拟服务器资源，并将 [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) 服务添加到基础架构环境。

1. 编辑 `vm.tf` 文件以增大以下各项的值，然后保存该文件。
 - 将 CPU 核心数增加到 4 个核心
 - 将 RAM（内存）增加到 4096
 - 将磁盘大小增加到 100 GB

2. 接下来，添加新的 [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) 服务，为此，请创建一个新文件，并将其命名为 **ibm-cloud-object-storage.tf**。将以下代码片段添加到新创建的文件中。这些代码片段为组织名称和空间名称创建变量名称，然后这两个变量名称用于检索创建服务所需的空间 GUID。代码将 {{site.data.keyword.cos_full_notm}} 服务名称设置为 `lamp_objectstorage`，然后您需要空间 GUID、服务标准名称和套餐类型。以下代码将创建高端套餐，这是现收现付套餐。您还可以使用轻量套餐，但请注意，轻量套餐限制为每个帐户仅一个服务。

   ```
   variable "org_name" {
     description = "Enter your IBM Cloud org name, you can get your org name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   variable "space_name" {
     description = "Enter your IBM Cloud space name, you can get your space name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   data "ibm_space" "space" {
     space = "${var.space_name}"
     org   = "${var.org_name}"
   }

   # Cloud Object Storage
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # 每个帐户只能有一个轻量套餐，因此我们将使用高端套餐 - 这是现收现付套餐
     plan = "Premium"
   }
   ```
   {: pre}
   **注：**我们稍后会在日志中查找标签“lamp_objectstorage”，以确保 {{site.data.keyword.cos_full_notm}} 已成功创建。

3. 通过运行以下命令，再次初始化 Terraform 配置：

   ```bash
    terraform init
   ```
   {: pre}

4. 通过运行以下命令，应用 Terraform 更改：
   ```bash
    terraform apply
   ```
   {: pre}
   **注：**成功运行 terraform apply 命令后，您应该会看到新的 `terraform.tfstate` 文件已添加到目录中。此文件包含完整的部署确认，可跟踪上次应用的内容以及未来对配置的任何修改。如果此文件被除去或丢失，那么您将丢失 Terraform 部署配置。

## 验证 VM 和 {{site.data.keyword.cos_short}}
{: #verifyvm}

在此部分中，您将验证 VM 和 {{site.data.keyword.cos_short}} 以确保已成功进行创建。

**验证 VM**

1. 在左侧菜单中，单击**基础架构**以查看虚拟服务器设备的列表。
2. 单击**设备** -> **设备列表**以查找创建的服务器。您应该会看到您的服务器设备列出。
3. 单击服务器以查看有关服务器配置的更多信息。请查看下面的屏幕快照，在其中可以看到服务器已成功创建。![源代码控制 URL](images/solution10/configuration.png)
4. 接下来，将在 Web 浏览器中测试服务器。在 Web 浏览器中打开服务器公共 IP 地址。您应该会看到类似于下图的服务器缺省安装页面。![源代码控制 URL](images/solution10/LAMP.png)


**验证 {{site.data.keyword.cos_full_notm}}**

1. 在 **{{site.data.keyword.Bluemix_notm}}“仪表板”**中，您应该会看到 {{site.data.keyword.cos_full_notm}} 服务实例已创建并可供使用。![object-storage](images/solution10/ibm-cloud-object-storage.png)

   有关 {{site.data.keyword.cos_full_notm}} 的更多信息可以在[此处](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)找到。

## 除去资源
{: #deleteresources}

使用以下命令删除资源：
   ```bash
   terraform destroy
   ```
   {: pre}

**注：**要删除资源，您需要基础架构管理员许可权。如果您没有管理员超级用户帐户，请使用基础架构仪表板来请求取消资源。可以在基础架构仪表板中的“设备”下请求取消设备。![object-storage](images/solution10/rm.png)

## 相关内容

- [Terraform](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [使用 CDN 加速交付静态文件 - {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

