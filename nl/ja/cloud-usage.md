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

# {{site.data.keyword.cloud_notm}} サービス、リソース、および使用量の確認
{: #cloud-usage}
Cloud の導入が増えると、IT マネージャーや財務マネージャーは、イノベーションとコスト制御のコンテキストにおける Cloud の使用量を把握する必要があります。使用可能なデータを調べることによって、「チームではどのサービスを使用していますか。」、「ソリューションの運用コストはどのくらいですか。」、「スプロールを抑制するにはどうすればよいですか。」などの質問に答えることができます。このチュートリアルでは、そのような情報を調べて使用量に関連する一般的な質問に答える方法について説明します。
{:shortdesc}

## 達成目標
{: #objectives}
* {{site.data.keyword.cloud_notm}} 成果物の明細表示: Cloud Foundry アプリとサービス、ID およびアクセス管理リソース、インフラストラクチャー・デバイス
* 使用量および請求への {{site.data.keyword.cloud_notm}} 成果物の関連付け
* {{site.data.keyword.cloud_notm}} 成果物と開発チームの間の関係の定義
* 使用量および請求データを利用した会計用データ・セットの作成

## アーキテクチャー
{: #architecture}

![アーキテクチャー](images/solution38/Architecture.png)

## 始める前に
{: #prereqs}

* [{{site.data.keyword.cloud_notm}} CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) のインストール
* [cURL](https://curl.haxx.se/) のインストール
* [Node.js](https://nodejs.org/) のインストール
* コマンド `npm install -g json2csv` を使用した [json2csv](https://www.npmjs.com/package/json2csv) のインストール
* [jq](https://stedolan.github.io/jq/) のインストール

## 背景
{: #background}

{{site.data.keyword.cloud_notm}} の使用量をリストしたり詳細を表示したりするコマンドを実行する前に、使用量の幅広い分類とその機能に関する背景を把握しておくと役立ちます。後からチュートリアルで使用される重要な用語は、太字で示されています。以下の成果物を視覚化したものが[アカウントの管理の資料](https://{DomainName}/docs/account?topic=account-overview#overview)にあります。

### Cloud Foundry
Cloud Foundry は {{site.data.keyword.cloud_notm}} 上のオープン・ソースの Platform as a Service (PaaS) で、サーバーを管理することなくアプリケーションや**サービス**をデプロイおよびスケーリングできます。Cloud Foundry は、アプリケーションとサービスを組織またはスペースに編成します。**組織**は、1 人以上のユーザーが所有して使用できる開発アカウントです。組織には複数のスペースを含めることができます。各**スペース**では、ユーザーは、アプリケーション開発、デプロイメント、および保守を行うための共有の場所にアクセスできます。

### Identity and Access Management
IBM Cloud Identity and Access Management (IAM) を使用すると、両方のプラットフォーム・サービスのユーザーをセキュアに認証でき、また {{site.data.keyword.cloud_notm}} プラットフォームのリソースへのアクセスを一貫して制御できます。最近のオファリングおよびマイグレーション済みの Cloud Foundry サービスは、{{site.data.keyword.cloud_notm}} Identity and Access Management によって管理される**リソース**として存在します。リソースは**リソース・グループ**に編成され、ポリシーおよび役割によるアクセス制御を提供します。

### インフラストラクチャー
インフラストラクチャーは、ベアメタル・サーバー、仮想サーバー・インスタンス、Kubernetes ノードなどのさまざまなコンピュート・オプションを網羅しています。コンソールでは、それぞれ**デバイス**として表示されます。

### アカウント
前述の成果物は、請求の目的で**アカウント**に関連付けられています。**ユーザー**はアカウントに招待され、アカウント内のさまざまなリソースへのアクセス権限を付与されます。

## 許可の割り当て
Cloud のインベントリーおよび使用量を表示するには、アカウント管理者によって割り当てられる適切な役割が必要です。アカウント管理者の場合は、次のセクションに進みます。

1. アカウント管理者は、{{site.data.keyword.cloud_notm}} にログインし、[**「ID およびアクセス・ユーザー (Identity & Access Users)」**](https://{DomainName}/iam/#/users)ページにアクセスします。
2. アカウント管理者は、アカウントのユーザーのリストから自分の名前を選択して、適切な許可を割り当てることができます。
3. **「アクセス・ポリシー」**タブで、**「アクセス権限の割り当て」**ボタンをクリックして、以下の変更を実行します。
   1. **「リソース・グループ内のアクセス権限の割り当て」**タイルを選択します。アクセス権限を付与する**リソース・グループ**を選択し、**「ビューアー」**役割を適用します。**「割り当て」**ボタンをクリックして終了します。この役割は、`resource groups` コマンドと `billing resource-group-usage` コマンドに必要です。
   2. **「Cloud Foundry を使用したアクセス権限の割り当て」**タイルを選択します。アクセス権限を付与する各**組織**の横にあるオーバーフロー・メニューを選択します。メニューから**「組織の役割の編集」**を選択します。**「組織の役割」**リストから**「請求管理者」**を選択します。**「役割の保存」**ボタンをクリックして終了します。この役割は、`billing org-usage` コマンドに必要です。
   3. **「リソースへのアクセス権限の割り当て」**タイルを選択します。**「サービス」**ドロップダウン・メニューで**「すべての ID およびアクセス対応サービス」**を選択します。**「プラットフォームのアクセス役割の割り当て」**で**「エディター」**役割を選択します。この役割は、`resource tag-attach` コマンドに必要です。

## 検索コマンドを使用したリソースの検索
{: #search}

開発チームが Cloud サービスの使用を開始すると、マネージャーはデプロイされているサービスを把握することで恩恵を受けます。デプロイメント情報は、イノベーションやサービス管理に関連する質問に回答するのに役立ちます。
- チームで他のプロジェクトを強化するために共有できるサービス関連のスキルは何ですか。
- チームをまたいだ共通点のうち、一部で欠けている可能性があるものは何ですか。
- 重要な修正が必要なサービスまたはまもなく非推奨になるサービスを使用しているチームはどれですか。
- チームはスプロールを最小化するためにどのようにサービス・インスタンスを確認できますか。

検索は、サービスとリソースに限定されません。Cloud Foundry の組織とスペース、リソース・グループ、リソース・バインディング、別名などの Cloud 成果物を照会することもできます。例については、[ibmcloud resource search](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search) の資料を参照してください。
{:tip}

1. コマンド・ラインを使用して {{site.data.keyword.cloud_notm}} にログインし、ターゲットの Cloud Foundry アカウントを設定します。[CLI の概説](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)を参照してください。
    ```sh
    ibmcloud login
    ```
    {: pre}

    ```sh
    ibmcloud target --cf
    ```
    {: pre}

2. アカウント内で使用されているすべての Cloud Foundry サービスをリストします。
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. ブール・フィルターを適用して、検索を広げたり絞り込んだりできます。例えば、以下の照会を使用して、Cloud Foundry サービスとアプリおよび IAM リソースを検索します。
    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. 特定のサービス・タイプを使用しているチームに通知するには、サービス名を使用して照会します。`<name>` を `weather` や `cloudant` などのテキストに置き換えます。次に、`<Organization ID>` を**組織 ID** に置き換えて、組織の名前を取得します。
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

タグ付けと検索を一緒に使用して、リソースのカスタマイズされた ID を提供できます。そのためには、タグをリソースに添付し、タグ名を使用して検索します。env:tutorial というタグを作成します。

1. タグをリソースに添付します。ユーザー・インターフェースから、または `ibmcloud resource service-instance <name|id>` を使用して、リソース CRN を取得できます。クラウド・リソース名 (CRN) は、IBM Cloud リソースを一意的に識別します。
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <resource CRN>
   ```
   {: pre}
1. 以下の照会を使用して、指定したタグに一致する Cloud 成果物を検索します。
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

マネージャーとチーム・リーダーは、拡張検索照会を企業が同意したタグ付けスキーマと組み合わせて、Cloud のアプリ、リソース、およびサービスを簡単に識別したり、アクションを実行したりできます。

## 使用状況ダッシュボードを使用した使用量の調査
{: #dashboard}

チームが使用しているサービスを管理が認識すると、次によく寄せられる質問として、「サービスの運用コストはどのくらいですか。」というものがあります。使用量とコストを判断する最も簡単な方法は、{{site.data.keyword.cloud_notm}} 使用状況ダッシュボードを確認することです。

1. {{site.data.keyword.cloud_notm}} にログインし、[「アカウント使用状況ダッシュボード」](https://{DomainName}/account/usage)にアクセスします。
2. **「グループ」**ドロップダウン・メニューから、サービス使用量を表示する Cloud Foundry 組織を選択します。
3. 特定の**「サービス・オファリング」**で**「インスタンスの表示」**をクリックして、作成されているサービス・インスタンスを表示します。
4. 表示されたページで、インスタンスを選択し、**「インスタンスの表示」**をクリックします。結果のページに、組織、スペース、場所などのインスタンスに関する詳細および合計コストを構成する個々の明細が表示されます。
5. 階層リンクを使用して、[「使用状況ダッシュボード」](https://{DomainName}/account/usage)に再度アクセスします。
6. **「グループ」**ドロップダウン・メニューで、セレクターを**「リソース・グループ」**に変更し、**「デフォルト」**などのグループを選択します。
7. 使用可能なインスタンスについて同様の確認を実行します。

## コマンド・ラインを使用した使用量の調査

このセクションでは、コマンド・ライン・インターフェースを使用して使用量を調査します。

1. 使用可能なすべての Cloud Foundry 組織をリストし、テスト用の組織を格納するための環境変数を設定します。
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<org name>
    ```
    {: pre}

2. `billing` コマンドを使用して、特定の組織の請求および使用量の明細を表示します。YYYY-MM 形式の日付の `-d` フラグを使用して、特定の月を指定します。
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. リソースについて同様の調査を実行します。すべてのアカウントに `default` リソース・グループがあります。値 `<group>` を最初のコマンドでリストされたリソース・グループに置き換えます。
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

4. `resource-instances-usage` コマンドを使用して、Cloud Foundry サービスと IAM リソースの両方を表示できます。許可のレベルに応じて、適切なコマンドを実行します。
    - アカウント管理者であるか、またはすべての ID およびアクセス対応サービスの管理者役割が付与されている場合は、単に、`resource-instances-usage` を実行します。
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  アカウント管理者でない場合は、チュートリアルの最初にビューアー役割が設定されたため、以下のコマンドを使用できます。
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. インフラストラクチャー・デバイスを表示するには、`sl` コマンドを使用します。次に、`vs` コマンドを使用して、{{site.data.keyword.virtualmachinesshort}} を確認します。
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## コマンド・ラインを使用した使用量のエクスポート
{: #usagecmd}

多くの場合、管理では、データを別のアプリケーションにエクスポートすることが必要になります。一般的な例として、スプレッドシートへの使用量データのエクスポートがあります。このセクションでは、使用量データをコンマ区切り値 (CSV) 形式にエクスポートします。この形式は、ほとんどのスプレッドシート・アプリケーションと互換性があります。

このセクションでは、`jq` および `json2csv` の、2 つのサード・パーティー・ツールを使用します。以下の各コマンドは、3 つのステップで構成されています。つまり、使用量データを JSON として取得し、JSON を構文解析し、結果を CSV としてフォーマットします。JSON データの取得と変換のプロセスは、`json2csv` に限定されず、他のツールを活用できます。

`-p` オプションを使用して出力結果を整えます。データの出力結果が適切でない場合は、`-p` 引数を削除して未加工の CSV データを出力します。
{:tip}

1. リソース・グループの使用量を各リソース・タイプの予想コストとともにエクスポートします。
    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. リソース・グループ内の各リソース・タイプのインスタンスの明細を表示します。
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. Cloud Foundry 組織と同じアプローチに従います。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. より高度な `jq` 照会を使用して、予想コストをデータに追加します。一部のリソース・タイプには複数のコスト・メトリックがあるため、多くの行が作成されます。
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. 同じ `jq` 照会を使用して、Cloud Foundry リソースを関連コストとともにリストします。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. `-p` オプションを削除し、出力をファイルにパイピングして、データをファイルにエクスポートします。結果の CSV ファイルをスプレッドシート・アプリケーションで開くことができます。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## API を使用した使用量のエクスポート
{: #usageapi}

`billing` コマンドは役立つものの、コマンド・ライン・インターフェースを使用して「全体像」を組み立てることは面倒です。同様に、使用状況ダッシュボードは、組織とリソース・グループの概要を示しますが、必ずしもチームまたはプロジェクトの使用量が示されるとは限りません。このセクションでは、カスタム要件に応じて使用量を取得するための、データ駆動型のアプローチの探索を開始します。

1. 端末で、API 要求と応答を出力するように、環境変数 `IBMCLOUD_TRACE=true` を設定します。
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. `billing org-usage` コマンドを再実行して、API 呼び出しを表示します。この単一のコマンドに対して複数のホストと API 経路が使用されていることに注意してください。
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. OAuth トークンを取得し、`billing org-usage` コマンドで表示されたいずれかの API 呼び出しを実行します。ベアラー許可として UAA トークンを使用する API もあれば、IAM トークンを使用する API もあることに注意してください。
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

4. インフラストラクチャー API を実行するには、[ユーザー・プロファイル](https://control.softlayer.com/account/user/profile)内の **API ユーザー名**と**認証鍵**について環境変数を取得し、設定します。API ユーザー名と認証鍵がわからない場合は、[「ユーザー・リスト」](https://control.softlayer.com/account/users)の自分の名前の横にある**「アクション」**メニューで作成できます。
    ```sh
    export IAAS_USERNAME=<API Username>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<Authentication Key>
    ```
    {: pre}

5. 以下の API を使用して、インフラストラクチャーの請求合計と請求項目を取得します。同様の API の文書は、[こちら](https://softlayer.github.io/reference/services/SoftLayer_Account/)を参照してください。
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. トレースを無効にします。
    ```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

データ駆動型のアプローチは使用量の調査の柔軟性が最も高くなりますが、このチュートリアルの範囲を超えているため、詳細については説明しません。GitHub プロジェクト [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample) が作成され、{{site.data.keyword.cloud_notm}} サービスと Cloud Functions を組み合わせてサンプル実装が提供されています。

## チュートリアルを発展させる
{: #expand}

以下の提案事項を使用して、調査をインベントリーと使用量に関連するデータに拡張してください。

- `--output json` オプションを指定して `ibmcloud billing` コマンドを調査します。これにより、チュートリアルでカバーされていない使用可能な他のデータ・プロパティーが表示されます。
- `ibmcloud resource search` の例および照会で使用できるプロパティーについては、ブログ投稿 [Using the {{site.data.keyword.cloud_notm}} command line to find resources](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/) を参照してください。
- インフラストラクチャーの使用量を調べるための追加 API については、[Infrastructure Account APIs](https://softlayer.github.io/reference/services/SoftLayer_Account/) を参照してください。
- 使用量データを集約するための高度な照会については、[jq Manual](https://stedolan.github.io/jq/manual/) を参照してください。
