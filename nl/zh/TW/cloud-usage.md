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

# 檢閱 {{site.data.keyword.cloud_notm}} 服務、資源和使用情況
{: #cloud-usage}
隨著雲端採用增加，IT 和財務經理將需要掌握如何使用雲端來進行創新並控制成本。藉由調查可用資料，可以回答下列問題：「團隊使用哪些服務？」、「解決方案需要多少營運成本？」以及「如何控制蔓生問題？」。本指導教學將介紹探索這些資訊的方法，並回答與使用情況相關的常見問題。
{:shortdesc}

## 目標
{: #objectives}
* 逐項列出 {{site.data.keyword.cloud_notm}} 構件：Cloud Foundry 應用程式和服務、Identity and Access Management 資源以及基礎架構裝置
* 將 {{site.data.keyword.cloud_notm}} 構件與用量和計費相關聯
* 定義 {{site.data.keyword.cloud_notm}} 構件與開發團隊之間的關係
* 使用用量和計費資料來建立資料集以用於核算

## 架構
{: #architecture}

![架構](images/solution38/Architecture.png)

## 開始之前
{: #prereqs}

* 安裝 [{{site.data.keyword.cloud_notm}} CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* 安裝 [cURL](https://curl.haxx.se/)
* 安裝 [Node.js](https://nodejs.org/)
* 使用指令 `npm install -g json2csv` 安裝 [json2csv](https://www.npmjs.com/package/json2csv)
* 安裝 [jq](https://stedolan.github.io/jq/)

## 背景
{: #background}

在執行可清查和詳述 {{site.data.keyword.cloud_notm}} 用量的指令之前，先瞭解各種種類的使用情況及其功能的背景知識會非常有用。本指導教學後面使用的關鍵術語均用粗體表示。在[管理帳戶文件](https://{DomainName}/docs/account?topic=account-overview#overview)中可以找到以下構件的有用視覺化。

### Cloud Foundry
Cloud Foundry 是 {{site.data.keyword.cloud_notm}} 上的一個開放程式碼平台即服務 (PaaS)，讓您能部署和調整應用程式與**服務**，而無需管理伺服器。Cloud Foundry 將應用程式和服務組織成組織或空間。**組織**是一位或多位使用者可以擁有並使用的開發帳戶。一個組織可以包含多個空間。每個**空間**都可讓使用者存取用於應用程式開發、部署和維護的共用位置。

### Identity and Access Management
IBM Cloud Identity and Access Management (IAM) 讓您能對這兩個平台服務的使用者進行安全鑑別，並在整個 {{site.data.keyword.cloud_notm}} 平台上以一致的方式來控制對資源的存取權。較新的供應項目和移轉的 Cloud Foundry 服務以**資源**形式存在，而資源由 {{site.data.keyword.cloud_notm}} Identity and Access Management 進行管理。資源會組織成**資源群組**，並透過原則和角色提供存取控制。

### 基礎架構
基礎架構包含各種運算選項：裸機伺服器、虛擬伺服器實例和 Kubernetes 節點。在主控台中，每個選項都被視為**裝置**。

### 帳戶
上述構件與**帳戶**相關聯，以便進行計費。可以邀請**使用者**加入帳戶並向其授與對帳戶中不同資源的存取權。

## 指派許可權
若要檢視雲端庫存和用量，您需要由帳戶管理者指派適當的角色。如果您是帳戶管理者，請繼續執行下節。

1. 帳戶管理者應該登入 {{site.data.keyword.cloud_notm}}，然後存取[**身分及存取使用者**](https://{DomainName}/iam/#/users)頁面。
2. 帳戶管理者可以從帳戶中的使用者清單中選取您的姓名，以指定適當的許可權。
3. 在**存取原則**標籤上，按一下**指派存取權**按鈕，然後執行下列變更。
   1. 選取**指派對資源群組的存取權**。選取要向其授權存取權的**資源群組**，然後套用**檢視者**角色。藉由按一下**指派**按鈕以完成作業。此角色對 `resource groups` 和 `billing resource-group-usage` 指令而言為必要。
   2. 選取**使用 Cloud Foundry 指派存取權**。選取要向其授權存取權的每個**組織**旁邊的溢位功能表。從功能表中，選取**編輯組織角色**。從**組織角色**清單中，選取**帳單管理員**。藉由按一下**儲存角色**按鈕以完成作業。此角色對 `billing org-usage` 指令而言為必要。
   3. 選取**指派對資源的存取權**磚。在**服務**下拉功能表中，選取**所有已啟用身分及存取的服務**。勾選**指派平台存取角色**下的**編輯者**角色。此角色對 `resource tag-attach` 指令而言為必要。

## 使用 search 指令尋找資源
{: #search}

隨著開發團隊開始使用雲端服務，管理員可以透過瞭解部署哪些服務而獲益。部署資訊有助於回答與創新和服務管理相關的問題：
- 可以跨團隊共用哪些與服務相關的技能，以便改進其他專案？
- 不同團隊有哪些共通性可能在某些團隊中缺乏？
- 哪些團隊所使用的服務需要關鍵修正程式或者很快會予以淘汰？
- 團隊如何檢閱自己的服務實例，讓蔓生問題減到最少？

搜尋的範圍不限於服務和資源。您還可以查詢雲端構件，例如 Cloud Foundry 組織和空間、資源群組、資源連結、別名等。如需其他範例，請參閱 [ibmcloud resource search](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search) 文件。
{:tip}

1. 透過指令行登入 {{site.data.keyword.cloud_notm}}，並設定 Cloud Foundry 帳戶的目標。請參閱 [CLI 開始使用](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)。
    ```sh
    ibmcloud login
    ```
    {: pre}

    ```sh
ibmcloud target --cf
  ```
    {: pre}

2. 列出帳戶中使用的所有 Cloud Foundry 服務庫存。
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. 可以套用布林過濾器來擴大或縮小搜尋範圍。例如，使用以下查詢來尋找 Cloud Foundry 服務和應用程式以及 IAM 資源。

    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. 若要通知使用特定服務類型的團隊，請使用服務名稱進行查詢。將 `<name>` 取代為文字，例如 `weather` 或 `cloudant`。然後，藉由將 `<Organization ID>` 取代為**組織 ID** 來取得組織名稱。
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

標籤和搜尋可以一起使用，以提供資源的自訂識別方式。這涉及：將標籤連接到資源，然後使用標籤名稱進行搜尋。請建立名為 env:tutorial 的標籤。

1. 將該標籤連接到資源。您可以從使用者介面或使用 `ibmcloud resource service-instance <name|id>` 來取得資源 CRN。雲端資源名稱 (CRN) 用於唯一地識別 IBM Cloud 資源。
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <resource CRN>
   ```
   {: pre}
1. 使用以下查詢來搜尋與給定標籤相符的雲端構件。
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

藉由將進階搜尋查詢與企業協議的標籤綱目結合使用，管理員和團隊領導人可以更輕鬆地識別雲端應用程式、資源和服務並對其執行動作。

## 使用用量儀表板探索用量
{: #dashboard}

管理在瞭解團隊使用的服務後，下一個常問的問題就是「這些服務需要多少操作成本？」確定用量和成本的最直接方法是檢閱 {{site.data.keyword.cloud_notm}} 用量儀表板。

1. 登入 {{site.data.keyword.cloud_notm}}，然後存取[帳戶用量儀表板](https://{DomainName}/account/usage)。
2. 從**群組**下拉功能表中，選取要檢視其服務使用的 Cloud Foundry 組織。
3. 對於給定的**服務供應項目**，按一下**檢視實例**可檢視已建立的服務實例。
4. 在下列頁面上，選擇一個實例，然後按一下**檢視實例**。產生的頁面會提供有關該實例的詳細資料，例如組織、空間和位置以及構成總成本的各個行項目。
5. 使用瀏覽途徑來重新造訪[用量儀表板](https://{DomainName}/account/usage)。
6. 從**群組**下拉功能表中，將該選取器切換到**資源群組**，然後選取一個群組，例如 **default**。
7. 對可用實例處理類似的檢閱。

## 使用指令行探索用量

在本節中，您將使用指令行介面來探索用量。

1. 列出可用的所有 Cloud Foundry 組織，然後設定一個環境變數以儲存要測試的 Cloud Foundry 組織。
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<org name>
    ```
    {: pre}

2. 使用 `billing` 指令來逐項列出給定組織的計費及用量。使用 `-d` 旗標與 YYYY-MM 格式的日期來指定特定月份。
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. 對資源處理相同的調查。每個帳戶都有一個 `default` 資源群組。將值 `<group>` 替換為第一個指令中列出的資源群組。
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

4. 可以使用 `resource-instances-usage` 指令來檢視 Cloud Foundry 服務和 IAM 資源。請根據您的許可權層次，執行適當的指令。
    - 如果您是帳戶管理者或已獲授權對所有已啟用身分及存取的服務的「管理者」角色，則只需執行 `resource-instances-usage`。
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  如果您不是帳戶管理者，則可以使用下列指令，因為在本指導教學開始部分已設定「檢視者」角色。
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. 若要檢視基礎架構裝置，請使用 `sl` 指令。然後，使用 `vs` 指令來檢閱 {{site.data.keyword.virtualmachinesshort}}。
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## 使用指令行匯出用量
{: #usagecmd}

管理中有些人經常需要將資料匯出到其他應用程式。常見的例子是將用量資料匯出到試算表中。在本節中，您要將用量資料匯出為逗點分隔值 (CSV) 格式，此格式與大多數試算表應用程式相容。

本節將使用兩個協力廠商工具：`jq` 和 `json2csv`。以下每個指令都由三個步驟組成：取得 JSON 格式的用量資料，並剖析 JSON，然後將結果的格式設定為 CSV。取得和轉換 JSON 資料的處理程序不侷限於使用 `json2csv` 來執行，還可以利用其他工具。

使用 `-p` 選項可細緻列印結果。如果資料顯示質量不佳，請移除 `-p` 引數以顯示原始 CSV 資料。
{:tip}

1. 匯出資源群組的使用情況，包括每種資源類型的預期成本。
    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. 逐項列出資源群組中每種資源類型的實例。
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. 對 Cloud Foundry 組織採用相同的方法。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. 使用更進階的 `jq` 查詢，向資料新增預期成本。這將建立更多行，因為某些資源類型具有多個成本度量值。
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. 使用相同的 `jq` 查詢還可列出 Cloud Foundry 資源及關聯的成本。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. 藉由移除 `-p` 選項，並藉由管線將輸出傳送到檔案，可將資料匯出到檔案。產生的 CSV 檔案可以使用試算表應用程式開啟。
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## 使用 API 匯出用量
{: #usageapi}

雖然 `billing` 指令很有用，但嘗試使用指令行介面來組合「全局」視圖會非常繁瑣。與此類似，用量儀表板提供組織和資源群組的概觀，但不一定是團隊或專案的使用情況。在本節中，您將開始探索資料驅動程度更高的方法，以根據自訂需求來取得使用情況。

1. 在終端機中，設定環境變數 `IBMCLOUD_TRACE=true` 來顯示 API 要求和回應。
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. 重新執行 `billing org-usage` 指令以查看 API 呼叫。請注意，此單一指令可使用多個主機和 API 路徑。
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. 取得 OAuth 記號並執行在 `billing org-usage` 指令中顯示的其中一個 API 呼叫。請注意，一些 API 使用 UAA 記號，另一些 API 可能使用 IAM 記號作為持有人授權。
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

4. 若要執行基礎架構 API，請為[使用者設定檔](https://control.softlayer.com/account/user/profile)中顯示的 **API 使用者名稱**和**鑑別金鑰**設定環境變數。如果看不到 API 使用者名稱和鑑別金鑰，則可以在[使用者清單](https://control.softlayer.com/account/users)中您姓名旁邊的**動作**功能表上建立 API 使用者名稱和鑑別金鑰。
    ```sh
    export IAAS_USERNAME=<API Username>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<Authentication Key>
    ```
    {: pre}

5. 使用下列 API 來取得基礎架構計費總計和計費項目。[這裡](https://softlayer.github.io/reference/services/SoftLayer_Account/)記錄類似的 API。
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. 停用追蹤。
    ```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

雖然資料驅動方法在探索用量方面提供最大的彈性，但更詳盡的說明已超出本指導教學的範圍。建立 GitHub 專案 [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample)，用於將 {{site.data.keyword.cloud_notm}} 服務與 Cloud Functions 結合使用，以提供範例實作。

## 擴展指導教學
{: #expand}

使用下列建議可將調查範圍擴展到庫存以及與使用情況相關的資料。

- 探索搭配 `--output json` 選項的 `ibmcloud billing` 指令。這將顯示本指導教學中未涵蓋的其他可用資料內容。
- 閱讀部落格文章 [Using the {{site.data.keyword.cloud_notm}} command line to find resources](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/)，以瞭解有關 `ibmcloud resource search` 的更多範例，以及可以在查詢中使用哪些內容。
- 檢閱 [Infrastructure Account APIs](https://softlayer.github.io/reference/services/SoftLayer_Account/)，以瞭解用於調查基礎架構使用情況的其他 API。
- 檢閱 [jq Manual](https://stedolan.github.io/jq/manual/)，以瞭解用於聚集用量資料的進階查詢。
