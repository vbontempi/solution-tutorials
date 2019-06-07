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

# Terraform を使用した LAMP スタックのデプロイ
{: #infrastructure-as-code-terraform}

[Terraform](https://www.terraform.io/) を使用すると、インフラストラクチャーを安全かつ予想どおりに作成、変更、および改善できます。 これは、API を宣言構成ファイルに体系化するオープン・ソース・ツールであり、体系化されたファイルは、チーム・メンバー間で共有してコードとして処理し、編集、レビュー、およびバージョン管理できます。

このチュートリアルでは、サンプル構成を使用して、**L**inux 仮想サーバーを **A**pache Web サーバー、**M**ySQL、および **P**HP サーバーとともにプロビジョンします (**LAMP** スタックと呼ばれます)。次に、構成を更新して、{{site.data.keyword.cos_full_notm}} サービスを追加してリソースをスケーリングし、環境 (メモリー、CPU、およびディスク・サイズ) を調整します。構成によって作成されたすべてのリソースを削除して終了します。

## 達成目標
{: #objectives}

* Terraform および {{site.data.keyword.Bluemix_notm}} Provider for Terraform を構成します。
* Terraform を使用して、LAMP スタック構成を作成、更新、スケーリングし、最終的に破棄します。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムおよびサービスを使用します。
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

このチュートリアルでは、費用が発生する場合があります。 [料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー図](images/solution10/architecture-2.png)
</p>

1. LAMP スタック構成を記述するために、一連の Terraform ファイルが作成されます。
1. 構成ディレクトリーから、`terraform` が呼び出されます。
1. リソースをプロビジョンするために、`terraform` により {{site.data.keyword.cloud_notm}} API が呼び出されます。

## 始める前に
{: #prereqs}

インフラストラクチャー・マスター・ユーザーに連絡して、以下の許可を取得します。
- ネットワーク (**「パブリックおよびプライベートのネットワーク・アップリンク (Public and Private Network Uplink)」**を追加するため)
- API キー

## 前提条件

{: #prereq}

[インストーラー](https://www.terraform.io/intro/getting-started/install.html)を介して、または macOS で [Homebrew](https://brew.sh/) を使用してコマンド `brew install terraform` を実行することにより、**Terraform** をインストールします。

**Windows** では、以下のステップに従って terraform のセットアップを完了します。
   1. ダウンロードした zip から `C:\terraform` (フォルダー `terraform` を作成します) にファイルをコピーします。
   2. コマンド・プロンプトを管理者として開き、terraform バイナリーを使用する PATH を設定します。

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## {{site.data.keyword.Bluemix_notm}} Provider for Terraform の構成
{: #setup}

マルチクラウド方式をサポートするため、Terraform はプロバイダーと連携します。 プロバイダーは、API の対話を理解し、リソースを公開する必要があります。このセクションでは、CLI を構成して {{site.data.keyword.Bluemix_notm}} プロバイダーの場所を指定します。

1. 端末またはコマンド・プロンプト・ウィンドウで `terraform` を実行し、Terraform インストールを確認します。  `Common commands` のリストが表示されます。

  ```
  terraform
  ```

2. ご使用のシステムに適切な [{{site.data.keyword.Bluemix_notm}} プロバイダー](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)のプラグインをダウンロードして、アーカイブを抽出します。 `terraform-provider-ibm_VERSION` バイナリー・プラグイン・ファイルが表示されます。

3. Windows 以外のシステムの場合は、ユーザーのホーム・ディレクトリーに `.terraform.d/plugins` ディレクトリーを作成し、バイナリー・ファイルをその内部に配置します。参照には、次のコマンドを使用します。

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    **Windows** の場合、ファイルは、ユーザーの「アプリケーション データ」ディレクトリーの下にある `terraform.d/plugins` に配置する必要があります。

  - コマンド・プロンプトで、次のコマンドを実行します。 [プロバイダー構成](https://www.terraform.io/docs/configuration/providers.html)
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - **Windows Powershell** を起動し (「スタート + R」>「Powershell」)、次のコマンドを実行して `terraform.rc` ファイルを作成します。
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   最初のプロンプトで、次のコンテンツを入力します。
   ```
    # ~/.terraformrc
    providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        PATH_TO_YOUR_APPDATA_PLUGINS は、スラッシュ (/) を含む絶対パスにする必要があります。 例えば、`C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe` です。
        {: tip}

  - Enter をクリックしてプロンプトを終了します。

## terraform 構成の準備

{: #terraformconfig}

このセクションでは、{{site.data.keyword.Bluemix_notm}} で提供されているサンプルの Terraform 構成を使用して、terraform 構成の基本について説明します。

1. https://github.com/IBM-Cloud/LAMP-terraform-ibm にアクセスし、独自のコピーを自分のアカウントに **fork** します。
2. fork をローカルに複製します。
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. 構成ファイルを検査します。
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml): サーバー・インストール構成が含まれます。このファイルで、サーバー・インストールに関連するすべてのスクリプトをサーバーに対するインストール対象に追加できます。 このファイルに注入された `phpinfo();` を参照してください。
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf): プロバイダーのユーザー名および API キーが必要となるプロバイダーに関連する変数が含まれます。
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf): 指定された変数を含む VM をデプロイするためのサーバー構成が含まれます。
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars): **SoftLayer** のユーザー名と API キー、{{site.data.keyword.Bluemix_notm}} API キー、およびスペースと組織の名前が含まれます。サーバーをデプロイするたびにこれらの資格情報をコマンド・ラインから再入力することを避けるベスト・プラクティスとして、これらの資格情報をこのファイルに追加できます。注: このファイルを資格情報とともに公開しないでください。
4. 任意の IDE で [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) ファイルを開き、**公開 SSH** 鍵を追加してファイルを変更します。 これは、この構成によって作成される VM へのアクセスに使用されます。 SSH 鍵の作成について詳しくは、[このリンク](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html)の指示に従ってください。 公開鍵をクリップボードにコピーするには、端末で次のコマンドを実行できます。

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     このコマンドによって SSH がクリップボードにコピーされたら、この SSH を [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) の 69 行目前後にある `ssh_key` のデフォルト変数に渡すことができます。

    **Windows** で、[Git Bash](https://git-scm.com/download/win) をダウンロード、インストール、および起動し、次のコマンドを実行して、公開 SSH 鍵をクリップボードにコピーします。

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) ファイルを IDE で開き、リストされているすべての資格情報を追加してファイルを変更します。これらの資格情報をこのファイルに追加することで、terraform の適用を実行するたびにこれらの資格情報を再入力する必要がなくなります。 このチュートリアルの残りを完了するには、[terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) ファイルに、リストされている 5 つの資格情報をすべて追加する必要があります。

## terraform 構成から LAMP スタック・サーバーを作成する
{: #Createserver}
このセクションでは、terraform 構成サンプルから LAMP スタック・サーバーを作成する方法について説明します。 この構成を使用して仮想マシン・インスタンスをプロビジョンし、**A**pache、**M**ySQL (**M**ariaDB)、および **P**HP をそのインスタンスにインストールします。

1. 複製したリポジトリーのフォルダーにナビゲートします。
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. terraform 構成を初期化します。 これにより、`terraform-provider-ibm_VERSION` プラグインもインストールされます。
   ````bash
    terraform init
   ````
   {: pre}
3. terraform 構成を適用します。 これにより、構成で定義されているリソースが作成されます。
   ```
    terraform apply
   ```
   {: pre}
   次のような出力が表示されます。![ソース管理 URL](images/solution10/created.png)
4. 次に、[インフラストラクチャー・デバイス・リスト](https://{DomainName}/classic/devices)にアクセスして、サーバーが作成されたことを確認します。![ソース管理 URL](images/solution10/configuration.png)

## {{site.data.keyword.cos_full_notm}} サービスの追加とリソースのスケーリング

{: #modify}

このセクションでは、仮想サーバー・リソースをスケーリングして、[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) サービスをインフラストラクチャー環境に追加する方法について説明します。

1. `vm.tf` ファイルを編集して以下のものを増やし、ファイルを保存します。
 - CPU コアの数を 4 コアに増やします。
 - RAM (メモリー) を 4096 に増やします。
 - ディスク・サイズを 100GB に増やします。

2. 次に、新しい [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) サービスを追加します。これを行うには、新しいファイルを作成して、そのファイルに **ibm-cloud-object-storage.tf** という名前を付けます。 次のコード・スニペットを新しく作成したファイルに追加します。 次のコード・スニペットにより、組織名とスペース名用の変数名が作成された後、これらの 2 つの変数名を使用して、サービスの作成に必要なスペース GUID が取得されます。 これにより、{{site.data.keyword.cos_full_notm}} サービス名が `lamp_objectstorage` に設定された後、スペース GUID、サービスの完全修飾名、およびプラン・タイプが必要となります。 次のコードにより、何らかの従量課金 (PAYG) プランであることを前提として、プレミアム・プランが作成されます。 ライト・プランを使用することもできますが、ライト・プランはアカウントごとに 1 つのサービスのみに制限されることに注意してください。

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

   # a cloud object storage
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # you can only have one Lite plan per account so let's use the Premium - it is pay-as-you-go
     plan = "Premium"
   }
   ```
   {: pre}
   **注:** {{site.data.keyword.cos_full_notm}} が正常に作成されたことを確認するために、後で、「lamp_objectstorage」というラベルをログで探します。

3. 次を実行して、terraform 構成を再度初期化します。

   ```bash
    terraform init
   ```
   {: pre}

4. 次を実行して、terraform の変更を適用します。
   ```bash
    terraform apply
   ```
   {: pre}
   **注:** terraform apply コマンドが正常に実行されると、新しい `terraform.tfstate` が表示されます。 ファイルがディレクトリーに追加されます。 このファイルには、最後に適用した内容および構成に対する将来のすべての変更をトラッキングするための、完全デプロイメントの確認が含まれます。 このファイルが削除されたり破損したりした場合、terraform デプロイメント構成は失われます。

## VM および {{site.data.keyword.cos_short}} の検証
{: #verifyvm}

このセクションでは、VM および {{site.data.keyword.cos_short}} を検証して、正常に作成されていることを確認します。

**VM の検証**

1. 左側のメニューで、**「インフラストラクチャー」**をクリックし、仮想サーバー・デバイスのリストを表示します。
2. **「デバイス」**->**「デバイス・リスト (Device List)」**をクリックして、作成したサーバーを見つけます。 サーバー・デバイスがリストされて表示されます。
3. サーバーをクリックして、サーバー構成に関する詳細情報を表示します。 次のスクリーン・ショットを見ると、サーバーが正常に作成されたことがわかります。 ![ソース管理 URL](images/solution10/configuration.png)
4. 次に、Web ブラウザーでサーバーをテストしてみましょう。 Web ブラウザーでサーバーのパブリック IP アドレスを開きます。 次のような、サーバーのデフォルトのインストール・ページが表示されます。![ソース管理 URL](images/solution10/LAMP.png)


**{{site.data.keyword.cos_full_notm}} の検証**

1. **{{site.data.keyword.Bluemix_notm}} ダッシュボード**に、作成されて使用する準備ができた {{site.data.keyword.cos_full_notm}} サービスのインスタンスが表示されます。 ![object-storage](images/solution10/ibm-cloud-object-storage.png)

   {{site.data.keyword.cos_full_notm}} について詳しくは、[ここ](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)を参照してください。

## リソースの削除
{: #deleteresources}

次のコマンドを使用して、リソースを削除します。
   ```bash
   terraform destroy
   ```
   {: pre}

**注:** リソースを削除するには、インフラストラクチャー管理許可が必要となる場合があります。 管理スーパーユーザー・アカウントがない場合は、インフラストラクチャー・ダッシュボードを使用して、リソースのキャンセルを要求してください。「デバイス」の下のインフラストラクチャー・ダッシュボードから、デバイスのキャンセルを要求できます。![object-storage](images/solution10/rm.png)

## 関連コンテンツ

- [Terraform](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [CDN を使用した静的ファイルの配信の高速化 - {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

