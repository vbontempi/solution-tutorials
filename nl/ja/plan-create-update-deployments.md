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

#                 デプロイメント環境の計画、作成、および更新
            
{: #plan-create-update-deployments}

ソリューションを構築するときには、一般に、複数のデプロイメント環境を用意します。それらの環境には、開発から実動までのプロジェクトのライフサイクルが反映されます。このチュートリアルでは、{{site.data.keyword.Bluemix_notm}} CLI や [Terraform](https://www.terraform.io/) などのツールを使用して、そのようなデプロイメント環境の作成と保守を自動化します。
{:shortdesc}

開発者は、同じものを繰り返し作成することを好みません。[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 原則はその一例です。同様に、ユーザー・インターフェースを何回もクリックして環境をセットアップすることも好みません。そのため、システム管理者と開発者は、繰り返し行う退屈で間違いやすいタスクを自動化するために、シェル・スクリプトを長年使用してきました。

Infrastructure as a Service (IaaS)、Platform as a Service (PaaS)、Container as a Service (CaaS)、Functions as a Service (FaaS) によって開発者は高水準の抽象化を行えるようになり、ベア・メタル・サーバー、マネージド・データベース、仮想マシン、Kubernetes クラスターなどのリソースも簡単に獲得できるようになりました。しかし、こうしたリソースをプロビジョンした後には、リソースを相互接続したり、ユーザー・アクセスを構成したり、時間が経ったら構成を更新したりするなど、さまざまな作業が必要になります。こうしたすべての手順を自動化し、さまざまな環境でインストールと構成を繰り返せる機能が最近では必要不可欠になっています。

開発サイクルのさまざまなフェーズをサポートするために、容量、ネットワーキング、資格情報、ログの詳細度などが若干異なる複数の環境を 1 つのプロジェクトで使用することがよくあります。[こちらのチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)で、ユーザー、チーム、およびアプリケーションを編成するためのベスト・プラクティスとサンプル・シナリオを紹介しました。このサンプル・シナリオでは、*開発*、*テスト*、*実動* という 3 つの環境について考えました。これらの環境の作成を自動化するにはどうすればよいでしょうか? どのようなツールを使用できるでしょうか?

## 達成目標
{: #objectives}

* デプロイする一連の環境を定義する
* {{site.data.keyword.Bluemix_notm}} CLI と [Terraform](https://www.terraform.io/) を使用して、それらの環境のデプロイメントを自動化するためのスクリプトを作成する
* アカウントでそれらの環境をデプロイする

## 使用するサービス
{: #services}

このチュートリアルでは、以下の製品を使用します。
* [{{site.data.keyword.Bluemix_notm}} provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Identity and Access Management](https://{DomainName}/iam/#/users)
* [{{site.data.keyword.Bluemix_notm}} コマンド・ライン・インターフェース - `ibmcloud` CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. ターゲットのインフラストラクチャーをコードとして記述するために一連の Terraform ファイルを作成します。
1. オペレーターが `terraform apply` を使用して環境をプロビジョンします。
1. 環境の構成を実行するシェル・スクリプトを作成します。
1. オペレーターが環境に対してスクリプトを実行します。
1. 環境が完全に構成され、使用できる状態になります。

## 使用可能なツールの概要
{: #tools}

繰り返し可能なデプロイメントを作成するために {{site.data.keyword.Bluemix_notm}} と対話する最初のツールは、[{{site.data.keyword.Bluemix_notm}} コマンド・ライン・インターフェース、つまり `ibmcloud` CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) です。`ibmcloud` とそのプラグインで、クラウド・リソースの作成と構成を自動化できます。このコマンド・ラインから、{{site.data.keyword.virtualmachinesshort}}、Kubernetes クラスター、{{site.data.keyword.openwhisk_short}}、Cloud Foundry のアプリとサービスのすべてをプロビジョンできます。

[このチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)で紹介しているもう 1 つのツールは、HashiCorp の [Terraform](https://www.terraform.io/) です。HashiCorp によると、*Terraform はインフラストラクチャーを安全かつ予測可能な方法で作成、変更、改善することを可能にします。これは、宣言型の構成ファイルとして API を体系化するオープン・ソース・ツールです。構成ファイルをチーム・メンバー間で共有し、コードとして扱い、編集、レビュー、バージョン管理を行うことができます。* これは Infrastructure as Code です。どのようなインフラストラクチャーにするべきか記述すれば、Terraform が必要に応じてクラウド・リソースを作成、更新、削除します。

マルチクラウド方式をサポートするため、Terraform はプロバイダーと連携します。プロバイダーは、API の対話を理解し、リソースを公開する必要があります。{{site.data.keyword.Bluemix_notm}} には、[Terraform 用のプロバイダー](https://github.com/IBM-Cloud/terraform-provider-ibm)が用意されているので、{{site.data.keyword.Bluemix_notm}} のユーザーは Terraform を使用してリソースを管理できます。Terraform は、Infrastructure as Code として分類されますが、Infrastructure as a Service リソースに制限されません。{{site.data.keyword.Bluemix_notm}} Provider for Terraform は、IaaS (ベア・メタル、仮想マシン、ネットワーク・サービスなど)、CaaS ({{site.data.keyword.containershort_notm}} および Kubernetes クラスター)、PaaS (Cloud Foundry およびサービス)、FaaS ({{site.data.keyword.openwhisk_short}}) のリソースをサポートしています。

## デプロイメントを自動化するスクリプトを作成する
{: #scripts}

Infrastructure as Code の記述を開始する際には、作成するファイルを通常のコードとして扱い、コードをソース管理システムに保管することが重要です。これは後でさまざまな点で役に立ちます。例えば、ソース管理のレビュー・ワークフローを使用して、変更内容を検証してから適用したり、継続的統合パイプラインを追加して、インフラストラクチャーの変更内容を自動的にデプロイしたりできます。

[この Git リポジトリー](https://github.com/IBM-Cloud/multiple-environments-as-code)に、前述の環境をセットアップするために必要なすべての構成ファイルが含まれています。これ以降のセクションではそれらのファイルの内容について詳しく説明するので、このリポジトリーを複製してから進めてください。

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

リポジトリーの構造は、以下のとおりです。

| ディレクトリー | 説明 |
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) | Terraform ファイルのホーム |
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) | 3 つの環境に共通するリソースをプロビジョンするための Terraform ファイル |
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) | 特定の環境に固有の Terraform ファイル |
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) | ユーザー・ポリシーを構成するための Terraform ファイル |

### Terraform に実行させる困難な作業

*開発*、*テスト*、および*実動* の各環境は似ています。

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="1 つのデプロイメント環境を示す図" />
</p>

これらの環境は、共通の組織と環境固有のリソースを共有します。環境の違いは、割り当てる容量とアクセス権限です。この点が terraform ファイルに表されています。Cloud Foundry 組織をプロビジョンする ***global*** 構成と、Terraform ワークスペースを使用して環境固有のリソースをプロビジョンする ***per-environment*** 構成に分かれています。

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### グローバル構成

すべての環境で共通の Cloud Foundry 組織を共有し、環境ごとに専用のスペースを使用します。

[terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) ディレクトリーの下には、組織をプロビジョンするための Terraform スクリプトがあります。[main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf) には、組織の定義が入っています。

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

このリソースでは、すべてのプロパティーを変数で構成します。次のセクションで、これらの変数の設定方法について説明します。

環境を完全にデプロイするには、Terraform と {{site.data.keyword.Bluemix_notm}} CLI を併用します。CLI で作成するシェル・スクリプトで、この組織またはアカウントを名前または ID を使用して参照する必要がある場合があります。スクリプトで再使用するのに適したキー/値の形式でこの情報を表したファイルを出力する [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf) も、*global* ディレクトリーに入っています。

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

### 個々の環境

Terraform で複数の環境を管理する方法には、さまざまな方法があります。別々のディレクトリー (環境ごとに 1 つのディレクトリー) の下に Terraform ファイルを複製するという方法があります。[Terraform モジュール](https://www.terraform.io/docs/modules/index.html)を使用して共通の構成をグループ化し、別の環境にモジュールを再使用してコードの重複を減らすという方法もあります。ディレクトリーを別にする場合は、*開発* 環境を改良して変更内容をテストしてから、変更内容を他の環境に伝搬させることができます。このようなケースでは、特定のバージョンのモジュールを環境ファイルで参照できるように Terraform の*モジュール* も専用のソース・コード・リポジトリーに置くことがよくあります。

ここでは、各環境が比較的単純で似ているので、[ワークスペース](https://www.terraform.io/docs/state/workspaces.html#best-practices)という Terraform の別の概念を使用することにします。ワークスペースを使用すると、同じ terraform ファイル (.tf) をさまざまな環境で使用できます。この例では、*開発*、*テスト*、*実動* のワークスペースを使用します。これらには同じ Terraform 定義を使用しますが、異なる構成変数 (異なる名前、異なる容量) を使用します。

環境ごとに、以下が必要です。
* 専用の Cloud Foundry スペース
* 専用のリソース・グループ
* Kubernetes クラスター
* データベース
* ファイル・ストレージ

Cloud Foundry スペースは、前の手順で作成した組織にリンクします。Terraform の環境ファイルで、この組織を参照する必要があります。ここで、[Terraform のリモート状態](https://www.terraform.io/docs/state/remote.html)が役に立ちます。これにより、既存の Terraform 状態を読み取り専用モードで参照できます。この仕組みは、Terraform 構成を細分化して各部分を別々のチームに任せる場合に便利です。[backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf) に、先ほど作成した組織を検出するために使用する *global* のリモート状態の定義が入っています。

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

組織を参照できたら、その組織内にスペースを作成するのは簡単です。[main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf) に、環境に必要なリソースの定義が入っています。

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

組織名を *global* リモート状態から参照していることに注目してください。その他のプロパティーは、構成変数から取得されます。

次は、リソース・グループです。

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

このリソース・グループに Kubernetes クラスターを作成します。{{site.data.keyword.Bluemix_notm}} プロバイダーには、クラスターを表す Terraform リソースがあります。

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

ここでも、大多数のプロパティーは構成変数から初期設定されます。データ・センター、ワーカーの数、ワーカーのタイプを調整できます。

IAM 対応サービス ({{site.data.keyword.cos_full_notm}} や {{site.data.keyword.cloudant_short_notm}} など) もグループ内にリソースとして作成します。

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

アプリケーションからサービス資格情報を取得するために、Kubernetes バインディング (シークレット) を追加できます。

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

## アカウントでこの環境をデプロイする

### {{site.data.keyword.Bluemix_notm}} CLI をインストールする

1. [この手順](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)に従って、CLI をインストールします。
1. 以下を実行して、インストールを検証します。
   ```sh
   ibmcloud
   ```
   {: codeblock}

### Terraform と {{site.data.keyword.Bluemix_notm}} Provider for Terraform をインストールする

1. [システムに合った Terraform をダウンロードし、インストールします。](https://www.terraform.io/intro/getting-started/install.html)
1. [{{site.data.keyword.Bluemix_notm}} プロバイダー用の Terraform バイナリーをダウンロードします。](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)
   {{site.data.keyword.Bluemix_notm}} プロバイダーで Terraform をセットアップするには、この[リンク](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup)を参照してください。
   {:tip}
1. Terraform バイナリーを指す `.terraformrc` ファイルをホーム・ディレクトリーに作成します。以下の例では、`/opt/provider/terraform-provider-ibm` がディレクトリーへの経路です。
   ```sh
   # ~/.terraformrc
   providers {
     ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### コードを取得する

まだ終わっていない場合は、チュートリアルのリポジトリーを複製します。

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### プラットフォーム API キーを設定する

1. まだ終わっていない場合は、[プラットフォーム API キー](https://{DomainName}/iam/#/apikeys)を取得し、後で参照できるようにその API キーを保存します。

   > 後の手順で新規 Cloud Foundry 組織を作成してデプロイメント環境をホストする場合は、自分がアカウントの所有者であることを確認してください。
1. 以下のコマンドを実行して、[terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl) を *terraform/credentials.tfvars* にコピーします。
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. `terraform/credentials.tfvars` を編集し、`ibmcloud_api_key` の値を、取得したプラットフォーム API キーに設定します。

### Cloud Foundry 組織を作成または再使用する

新規組織を作成するか、または既存の組織を再使用 (インポート) することができます。3 つのデプロイメント環境の親組織を作成するには、**アカウント所有者で行う必要があります**。

#### 新規組織の作成方法

1. `terraform/global` ディレクトリーに移動します。
1. [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) を `global.tfvars` にコピーします。
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. `global.tfvars` を編集します。
   1. **org_name** に、作成する組織名を設定します。
   1. **org_managers** に、組織の*管理者* の役割を付与するユーザー ID のリストを設定します。組織を作成したユーザーは自動的に管理者になるので、このリストに追加しないでください。
   1. **org_users** に、組織に招待するすべてのユーザーのリストを設定します。後の手順でユーザーのアクセス権限を構成したい場合は、そのユーザーをこのリストに追加する必要があります。

   ```sh
   org_name = "a-new-organization"
   org_managers = [ "user1@domain.com", "another-user@anotherdomain.com" ]
   org_users = [ "user1@domain.com", "another-user@anotherdomain.com", "more-user@domain.com" ]
   ```
   {: codeblock}
1. `terraform/global` フォルダーから Terraform を初期設定します。
   ```sh
   terraform init
   ```
   {: codeblock}
1. Terraform プランを参照します。
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. 変更を適用します。
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Terraform が完了すると、以下が作成されています。
* 新規 Cloud Foundry 組織
* チェックアウトの `outputs` ディレクトリー下に `global.env` ファイル。このファイルには、他のスクリプトで参照できる環境変数が入っています。
* `terraform.tfstate` ファイル

> このチュートリアルでは、Terraform 状態のために `local` バックエンド・プロバイダーを使用しています。Terraform を試したり、1 人だけのプロジェクトで作業したりする場合には便利です。しかし、チームで作業したり大規模なインフラストラクチャーを扱ったりする場合には、リモート・ロケーションに状態を保存することもできます。Terraform 状態が Terraform の操作にとって重要な場合は、リモートにある可用性と耐障害性が高いストレージに Terraform 状態を保存することをお勧めします。選択可能なオプションのリストについては、[Terraform Backend Types](https://www.terraform.io/docs/backends/types/index.html) を参照してください。バックエンドによっては、Terraform 状態のバージョン管理とロックもサポートされています。

#### 管理している組織を再使用する方法

アカウント所有者でなくてもアカウントの組織を管理している場合は、既存の組織を Terraform にインポートすることもできます。

1. 組織 GUID を取得します。
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. `terraform/global` ディレクトリーに移動します。
1. [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) を `global.tfvars` にコピーします。
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Terraform を初期設定します。
   ```sh
   terraform init
   ```
   {: codeblock}
1. Terraform を初期設定した後、組織を Terraform 状態にインポートします。
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. 既存の組織名および構成と一致するように、`global.tfvars` を調整します。
1. 変更を適用します。
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### 環境固有のスペース、クラスター、およびサービスを作成する

このセクションでは、`開発`環境に焦点を当てます。他の環境でも手順は同じです。異なるのは、変数として選択する値のみです。

1. チェックアウトの `terraform/per-environment` フォルダーに移動します。
1. テンプレートの `tfvars` ファイルをコピーします。環境ごとに 1 つあります。
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. `development.tfvars` を編集します。
   1. **environment_name** に、作成する Cloud Foundry スペースの名前を設定します。
   1. **space_developers** に、このスペースの開発者のリストを設定します。**自分の代わりに Terraform がサービスをプロビジョンできるように、必ず、自分の名前をリストに加えてください。**
   1. **cluster_datacenter** に、クラスターを作成するロケーションを設定します。以下を使用して、使用可能なロケーションを確認してください。
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. クラスターのプライベート (**cluster_private_vlan_id**) およびパブリック (**cluster_public_vlan_id**) の VLAN を設定します。以下を使用して、対象のロケーションで使用可能な VLAN を確認してください。
      ```sh
      ibmcloud cs vlans <location>
      ```
      {: codeblock}
   1. **cluster_machine_type** を設定します。以下を使用して、対象のロケーションで使用可能なマシン・タイプおよび特性を確認してください。
      ```sh
      ibmcloud cs machine-types <location>
      ```
      {: codeblock}

   1. **resource_quota** を設定します。以下を使用して、使用可能なリソース割り当て量定義を確認してください。
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. Terraform を初期設定します。
   ```sh
   terraform init
   ```
   {: codeblock}
1. *開発* 環境用の新規 Terraform ワークスペースを作成します。
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   後で環境を切り替える際には、次を使用します。
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. Terraform プランを参照します。
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   次のように報告されるはずです。
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. 変更を適用します。
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Terraform が完了すると、以下が作成されています。
* リソース・グループ
* Cloud Foundry スペース
* Kubernetes クラスター (ワーカー・プールとそれに接続されたゾーンを含む)
* データベース
* Kubernetes シークレット (データベース資格情報を含む)
* ストレージ
* Kubernetes シークレット (ストレージ資格情報を含む)
* ロギング (LogDNA) インスタンス
* モニタリング (Sysdig) インスタンス
* チェックアウトの `outputs` ディレクトリー下に `development.env` ファイル。このファイルには、他のスクリプトで参照できる環境変数が入っています。
* `terraform.tfstate.d/development` の下に、環境固有の `terraform.tfstate`。

`testing` および `production` について、この手順を繰り返します。

### ユーザー・ポリシーを割り当てる

上記の手順では、Terraform プロバイダーで Cloud Foundry の組織とスペースの役割を構成できました。Kubernetes クラスターなどの他のリソースに対するユーザー・ポリシーについては、複製したリポジトリー内の [roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) フォルダーを使用します。

[このチュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)で定義した*開発* 環境では、以下のポリシーを定義します。

|           | IAM アクセス・ポリシー |
| --------- | ----------- |
| 開発者 | <ul><li>リソース・グループ: *ビューアー*</li><li>リソース・グループのプラットフォーム・アクセス役割: *ビューアー*</li><li>ロギング & モニタリング・サービスの役割: *ライター*</li></ul> |
| テスター    | <ul><li>構成不要。テスターは、開発環境ではなくデプロイ済みのアプリケーションにアクセスする</li></ul> |
| オペレーター  | <ul><li>リソース・グループ: *ビューアー*</li><li>リソース・グループのプラットフォーム・アクセス役割: *オペレーター*、*ビューアー*</li><li>ロギング & モニタリング・サービスの役割: *ライター*</li></ul> |
| パイプラインの機能ユーザー | <ul><li>リソース・グループ: *ビューアー*</li><li>リソース・グループのプラットフォーム・アクセス役割: *エディター*、*ビューアー*</li></ul> |

チームが複数の開発者とテスターで構成されている場合は、[アクセス・グループの概念](https://{DomainName}/docs/iam?topic=iam-groups#groups)を利用して、ユーザー・ポリシーの構成を簡略化できます。アカウント所有者がアクセス・グループを作成すれば、単一のポリシーを使用してグループ内のすべてのエンティティーに同じアクセス権限を割り当てられるようになります。

*開発* 環境の*開発者* 役割については、次のように表せます。

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

チェックアウトの [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) ファイルに、定義した*開発者*、*オペレーター*、*テスター*、および*機能ユーザー*の各役割に対するこれらのリソースの例があります。*開発* 環境の*開発者、オペレーター、テスター、および機能ユーザー*の役割を持つユーザーに対して、前のセクションで定義したポリシーを設定するには、以下のようにします。

1. `terraform/roles/development` ディレクトリーに移動します。
2. テンプレートの `tfvars` ファイルをコピーします。環境ごとに 1 つあります (`production` と `testing` のテンプレートは、`roles` ディレクトリー内にあるそれぞれのフォルダーの下にあります)。

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. `development.tfvars` を編集します。

   - **iam_access_members_developers** に、アクセス権限を付与する開発者のリストを設定します。
   - **iam_access_members_operators** に、オペレーターなどのリストを設定します。
4. Terraform を初期設定します。
   ```sh
   terraform init
   ```
   {: codeblock}

5. Terraform プランを参照します。
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   次のように報告されるはずです。
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. 変更を適用します。
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
`testing` および `production` について、この手順を繰り返します。

## リソースの削除

1. `roles` の下の `development` フォルダーに移動します。
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. アクセス・グループとアクセス・ポリシーを破棄します。
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. `development` ワークスペースをアクティブにします。
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. リソース・グループ、スペース、サービス、クラスターを破棄します。
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. `testing` および `production` のワークスペースについて、この手順を繰り返します。
1. 組織を作成した場合は、それを破棄します。
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## 関連コンテンツ

* [Terraform チュートリアル](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Terraform プロバイダー](https://ibm-cloud.github.io/tf-ibm-docs/)
* [{{site.data.keyword.Bluemix_notm}} Provider for Terraform の使用例](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
