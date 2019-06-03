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

# {{site.data.keyword.cloud_notm}} 서비스, 리소스 및 사용량 검토
{: #cloud-usage}
클라우드로의 전환이 증가함에 따라, IT 및 재무 관리자는 혁신 및 비용 제어의 문맥에서 클라우드 사용량을 이해해야 합니다. "팀에서 어느 서비스를 사용하고 있는가?", "솔루션을 운영하는 데 얼마의 비용이 드는가?", "스프롤 현상을 방지할 수 있는가?"와 같은 질문은 사용 가능한 데이터를 조사함으로써 답변할 수 있습니다. 이 튜토리얼에서는 이러한 정보를 탐색하고 일반적인 사용량 관련 질문에 응답하는 방법을 소개합니다.
{:shortdesc}

## 목표
{: #objectives}
* {{site.data.keyword.cloud_notm}} 아티팩트(Cloud Foundry 앱 및 서비스, Identity and Access Management 리소스 및 인프라 디바이스)의 명세 작성
* {{site.data.keyword.cloud_notm}} 아티팩트를 사용량 및 청구와 연관
* {{site.data.keyword.cloud_notm}} 아티팩트와 개발 팀 간의 관계 정의
* 사용량 및 청구 데이터를 이용하여 회계 목적의 데이터 세트 작성

## 아키텍처
{: #architecture}

![아키텍처](images/solution38/Architecture.png)

## 시작하기 전에
{: #prereqs}

* [{{site.data.keyword.cloud_notm}} CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)를 설치하십시오. 
* [cURL](https://curl.haxx.se/)을 설치하십시오. 
* [Node.js](https://nodejs.org/)를 설치하십시오. 
* `npm install -g json2csv` 명령을 사용하여 [json2csv](https://www.npmjs.com/package/json2csv)를 설치하십시오. 
* [jq](https://stedolan.github.io/jq/)를 설치하십시오. 

## 배경
{: #background}

{{site.data.keyword.cloud_notm}} 사용량을 명세화하여 자세히 표시하는 명령을 실행하기 전에, 사용량의 큰 카테고리와 이들의 기능에 대해 어느 정도 배경 지식을 갖추는 것이 좋습니다. 이 튜토리얼에서 나중에 사용되는 주요 용어는 굵은체로 표시되어 있습니다. 아래 아티팩트에 대한 유용한 시각화 자료는 [계정 관리 문서](https://{DomainName}/docs/account?topic=account-overview#overview)에 있습니다. 

### Cloud Foundry
Cloud Foundry는 서버를 관리하지 않으면서 애플리케이션 및 **서비스**를 배치하고 스케일링할 수 있게 해 주는 {{site.data.keyword.cloud_notm}}의 오픈 소스 PaaS(Platform as a Service)입니다. Cloud Foundry는 애플리케이션 및 서비스를 조직 및 영역으로 구성합니다. **조직**은 한 명 이상의 사용자가 소유하고 사용할 수 있는 개발 계정입니다. 조직은 여러 영역을 포함할 수 있습니다. 각 **영역**은 애플리케이션 개발, 배치 및 유지보수를 위한 공유 위치에 대한 액세스를 사용자에게 제공합니다. 

### Identity and Access Management
IBM Cloud Identity and Access Management(IAM)는 두 플랫폼 서비스에 사용자를 안전하게 인증하고 {{site.data.keyword.cloud_notm}} 플랫폼 전체에서 일관되게 리소스에 대한 액세스를 제어할 수 있게 해 줍니다. 최신 오퍼링과 마이그레이션된 Cloud Foundry 서비스는 {{site.data.keyword.cloud_notm}} Identity and Access Management에서 관리하는 **리소스**로 존재합니다. 리소스는 **리소스 그룹**으로 구성되며 정책 및 역할을 통한 액세스 제어를 제공합니다. 

### 인프라
인프라는 다양한 컴퓨팅 옵션(Bare Metal Server, Virtual Server 인스턴스 및 Kubernetes 노드)을 포함합니다. 각각은 콘솔에서 **디바이스**로 표시됩니다. 

### 계정
앞서 언급된 아티팩트는 청구를 위해 **계정**과 연관됩니다. **사용자**는 계정에 초대되며 계정 내 여러 리소스에 대한 액세스 권한이 부여됩니다. 

## 권한 지정
Cloud 인벤토리 및 사용량을 보려면 계정 관리자가 지정한 적절한 역할이 있어야 합니다. 자신이 계정 관리자인 경우에는 다음 절로 진행하십시오. 

1. 계정 관리자는 {{site.data.keyword.cloud_notm}}에 로그인하여 [**Identity & Access 사용자**](https://{DomainName}/iam/#/users) 페이지에 액세스해야 합니다. 
2. 계정 관리자는 사용자 목록에서 자신의 이름을 선택하여 적절한 권한을 지정할 수 있습니다. 
3. **액세스 정책** 탭에서 **액세스 권한 지정** 단추를 클릭하고 다음 변경을 수행하십시오. 
   1. **리소스 그룹 내 액세스 권한 지정** 타일을 선택하십시오. 액세스 권한을 지정할 **리소스 그룹**을 선택하고 **뷰어** 역할을 적용하십시오. **지정** 단추를 클릭하여 완료하십시오. 이 역할은 `resource groups` 및 `billing resource-group-usage` 명령에 필요합니다. 
   2. **Cloud Foundry를 사용하여 액세스 권한 지정** 타일을 선택하십시오. 액세스 권한을 부여할 각 **조직** 옆에 있는 오버플로우 메뉴를 선택하십시오. 메뉴에서 **조직 역할 편집**을 선택하십시오. **조직 역할** 목록에서 **청구 관리자**를 선택하십시오. **역할 저장** 단추를 클릭하여 완료하십시오. 이 역할은 `billing org-usage` 명령에 필요합니다. 
   3. **리소스에 대한 액세스 권한 지정** 타일을 선택하십시오. **서비스** 드롭 다운 메뉴에서 **모든 Identity and Access 사용 서비스**를 선택하십시오. **플랫폼 액세스 역할 지정**에서 **편집자** 역할을 선택하십시오. 이 역할은 `resource tag-attach` 명령에 필요합니다. 

## search 명령으로 리소스 검색
{: #search}

개발 팀이 Cloud 서비스 사용을 시작하면, 관리자에게는 어느 서비스가 배치되었는지 알 수 있다는 장점이 있습니다. 배치 정보는 혁신 및 서비스 관리와 관련된 질문에 답변하는 데 도움을 줍니다. 
- 프로젝트를 개선하기 위해 팀 간에 공유할 수 있는 서비스 관련 스킬은 무엇인가? 
- 전체 팀에서 공통적으로 부족한 것은 무엇인가? 
- 중요 수정사항이 있거나 곧 사용하지 않게 될 서비스를 사용하고 있는 팀은 어느 팀인가? 
- 스프롤 현상을 최소화하기 위해 팀은 어떻게 서비스 인스턴스를 검토해야 하는가? 

검색은 서비스 및 리소스로 제한되지 않습니다. Cloud Foundry 조직 및 영역, 리소스 그룹, 리소스 바인딩, 별명 등의 다른 IBM Cloud 아티팩트도 조회할 수 있습니다. 더 많은 예는 [IBM Cloud 리소스 검색](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search) 문서를 참조하십시오.
{:tip}

1. 명령행을 통해 {{site.data.keyword.cloud_notm}}에 로그인한 후 자신의 Cloud Foundry 계정을 대상으로 설정하십시오. [CLI 시작하기](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)를 참조하십시오.
    ```sh
    ibmcloud login
    ```
    {: pre}

    ```sh
    ibmcloud target --cf
    ```
    {: pre}

2. 계정 내에서 사용되고 있는 모든 Cloud Foundry 서비스를 명세화하십시오.
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. 검색 범위를 넓히거나 좁히기 위해 부울 필터를 적용할 수 있씁니다. 예를 들어, Cloud Foundry 서비스 및 앱과 IAM 리소스를 모두 찾으려면 아래 조회를 사용하십시오.
    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. 특정 서비스 유형을 사용하는 팀에게 알림을 전송하려면 해당 서비스 이름을 조회하십시오. `<name>`을 `weather` 또는 `cloudant`와 같은 항목으로 대체하십시오. 그 후 `<Organization ID>`를 **조직 ID**로 대체하여 조직의 이름을 가져오십시오.
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

사용자 정의된 리소스 식별을 제공하기 위해 태그 지정과 검색을 함께 사용할 수 있습니다. 이는 태그를 리소스에 연결하고 태그 이름을 사용하여 검색하는 작업을 포함합니다. 여기서는 env:tutorial이라는 태그를 작성합니다. 

1. 태그를 리소스에 연결하십시오. 리소스 CRN은 사용자 인터페이스 또는 `ibmcloud resource service-instance <name|id>`를 사용하여 얻을 수 있습니다. CRN(Cloud Resource Name)은 IBM Cloud 리소스를 고유하게 식별합니다. 
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <resource CRN>
   ```
   {: pre}
1. 아래 조회를 사용하여 해당 태그와 일치하는 IBM Cloud 아티팩트를 검색하십시오. 
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

관리자 및 팀 리드는 고급 검색 조회를 엔터프라이즈와 합의된 태그 지정 스키마와 결합하여 IBM Cloud 앱, 리소스 및 서비스를 더 쉽게 식별하고 이에 대해 조치를 수행할 수 있습니다. 

## 사용량 대시보드로 사용량 탐색
{: #dashboard}

관리 부서가 팀에서 사용 중인 서비스를 파악한 후 나오는 주된 질문은 "이러한 서비스를 운용하는 데 드는 비용은 얼마인가?"입니다. 사용량과 비용을 판별하는 가장 간단한 방법은 {{site.data.keyword.cloud_notm}} 사용량 대시보드를 검토하는 것입니다. 

1. {{site.data.keyword.cloud_notm}}에 로그인하여 [계정 사용량 대시보드](https://{DomainName}/account/usage)에 액세스하십시오. 
2. **그룹** 드롭 다운 메뉴에서 서비스 사용량을 볼 Cloud Foundry 조직을 선택하십시오. 
3. 특정 **서비스 오퍼링**에 대해 **인스턴스 보기**를 클릭하여 작성된 서비스 인스턴스를 보십시오. 
4. 그 다음 표시되는 페이지에서 인스턴스를 선택하고 **인스턴스 보기**를 클릭하십시오. 결과 페이지는 조직, 영역 및 위치와 같은 인스턴스 관련 세부사항과 총 비용을 구성하는 개별 항목을 제공합니다. 
5. 이동 경로를 사용하여 [사용량 대시보드](https://{DomainName}/account/usage)에 다시 방문하십시오. 
6. **그룹** 드롭 다운 메뉴에서 선택기를 **리소스 그룹**으로 변경하고 **default**와 같은 그룹을 선택하십시오. 
7. 사용 가능한 인스턴스에 대해 위와 같은 검토를 수행하십시오. 

## 명령행으로 사용량 탐색

이 절에서는 명령행 인터페이스로 사용량을 탐색합니다. 

1. 사용 가능한 모든 Cloud Foundry 조직을 나열하고 테스트를 위해 하나를 저장할 환경 변수를 설정하십시오.
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<org name>
    ```
    {: pre}

2. `billing` 명령으로 해당 조직의 청구 및 사용량에 대한 명세를 작성하십시오. `-d` 플래그를 YYYY-MM 형식의 날짜와 함께 사용하여 특정 월을 지정하십시오.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. 위와 동일한 리소스 조사를 수행하십시오. 모든 계정에는 `default` 리소스 그룹이 있습니다. 값 `<group>`을 첫 번째 명령에서 나열된 리소스 그룹으로 대체하십시오.
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

4. `resource-instances-usage` 명령을 사용하여 Cloud Foundry 서비스 및 IAM 리소스를 모두 볼 수 있습니다. 권한 레벨에 따라 적절한 명령을 실행하십시오. 
    - 자신이 계정 관리자이거나 모든 Identity and Access 사용 서비스에 대해 관리자 역할이 부여된 경우에는 `resource-instances-usage`만 실행하면 됩니다.
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  계정 관리자가 아닌 경우에는 튜토리얼의 시작 부분에서 설정된 뷰어 역할로 인해 다음 명령을 사용할 수 있습니다.
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. 인프라 디바이스를 보려면 `sl` 명령을 사용하십시오. 그 후 `vs` 명령을 사용하여 {{site.data.keyword.virtualmachinesshort}}를 검토하십시오.
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## 명령행으로 사용량 내보내기
{: #usagecmd}

관리 부서의 직원이 다른 애플리케이션으로 데이터를 내보내야 하는 경우가 있습니다. 일반적인 예로는 사용량 데이터를 스프레드시트로 내보내는 것이 있습니다. 이 절에서는 대부분의 스프레드시트 애플리케이션과 호환되는 쉼표로 구분된 값(CSV) 형식으로 사용량 데이터를 내보냅니다. 

이 절에서는 두 가지 서드파티 도구(`jq` 및 `json2csv`)를 사용합니다. 아래의 각 명령은 데이터를 JSON으로 얻고, JSON을 구문 분석하고, 결과를 CSV로 형식화하는 세 단계로 구성되어 있습니다. JSON 데이터 획득 및 변환 프로세스는 `json2csv`만으로 제한되지 않으며 다른 도구를 이용할 수도 있습니다. 

`-p` 옵션을 사용하여 결과를 보기 좋게 출력하십시오. 데이터 출력이 보기 좋지 않은 경우에는 `-p` 인수를 제거하여 원시 CSV 데이터를 출력하십시오.
{:tip}

1. 리소스 그룹의 사용량을 각 리소스 유형에 대한 예상 비용과 함께 내보내십시오.
    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. 리소스 그룹의 각 리소스 유형에 대해 인스턴스의 명세를 작성하십시오.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. Cloud Foundry 조직에 대해서도 동일한 접근법을 따르십시오.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. 고급 `jq` 조회를 사용하여 데이터에 예상 비용을 추가하십시오. 일부 리소스 유형에는 여러 비용 메트릭이 있으므로 이는 더 많은 행을 작성합니다.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. 동일한 `jq` 조회를 사용하여 Cloud Foundry 리소스 또한 연관된 비용과 함께 나열하십시오.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. `-p` 옵션을 제거하고 출력을 파일에 입력하여 데이터를 파일로 내보내십시오. 결과 CSV 파일은 스프레드시트 애플리케이션으로 열 수 있습니다.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## API로 사용량 내보내기
{: #usageapi}

`billing` 명령은 유용하지만, 명령행 인터페이스로 "큰 그림"을 짜맞추는 작업은 복잡할 수 있습니다. 마찬가지로, 사용량 대시보드는 조직 및 리소스 그룹에 대한 개요를 제공하지만 이것이 반드시 팀 또는 프로젝트의 사용량인 것은 아닙니다. 이 절에서는 사용자 정의 요구사항을 위한 사용량을 얻는 데 필요한, 더 데이터 중심적인 접근법에 대해 알아봅니다. 

1. 터미널에서 환경 변수 `IBMCLOUD_TRACE=true`를 설정하여 API 요청 및 응답을 출력하십시오.
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. `billing org-usage` 명령을 다시 실행하여 API 호출을 보십시오. 이 하나의 명령을 위해 여러 호스트 및 API 라우트가 사용되는 것을 볼 수 있습니다.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. OAuth 토큰을 얻고 `billing org-usage` 명령에서 본 API 호출 중 하나를 실행하십시오. Bearer 권한 부여에 UAA 토큰을 사용하는 API도 있고, IAM 토큰을 사용하는 API도 있다는 점을 참고하십시오.
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

4. 인프라 API를 실행하려면 [사용자 프로파일](https://control.softlayer.com/account/user/profile)에서 **API 사용자 이름** 및 **인증 키**를 얻고 이에 대한 환경 변수를 설정해야 합니다. API 사용자 이름 및 인증 키가 표시되지 않는 경우에는 [사용자 목록](https://control.softlayer.com/account/users)의 자기 이름 옆에 있는 **조치** 메뉴에서 이를 작성할 수 있습니다.
    ```sh
    export IAAS_USERNAME=<API Username>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<Authentication Key>
    ```
    {: pre}

5. 다음 API를 사용하여 인프라 청구 총액 및 청구 항목을 얻으십시오. 이와 유사한 API는 [여기](https://softlayer.github.io/reference/services/SoftLayer_Account/)에 기록되어 있습니다.
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. 추적을 사용 안함으로 설정하십시오.
    ```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

데이터 중심 접근법은 사용량 탐색에 있어서 가장 높은 유연성을 제공하지만, 더 자세한 설명은 이 튜토리얼의 범위를 벗어납니다. 샘플 구현을 제공하기 위해 {{site.data.keyword.cloud_notm}} 서비스를 Cloud Functions와 결합하는 GitHub 프로젝트 [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample)이 작성되었습니다. 

## 튜토리얼 확장
{: #expand}

인벤토리 및 사용량 관련 데이터에 대해 더 자세히 조사하려면 다음 제안사항을 사용하십시오. 

- `--output json` 옵션을 사용하여 `ibmcloud billing` 명령에 대해 자세히 알아보십시오. 이는 이 튜토리얼에서 다루지 않은, 사용 가능한 추가 데이터 특성을 보여줍니다. 
- `ibmcloud resource search`에 대한 추가 예와 조회에 사용할 수 있는 특성에 대해 알아보려면 블로그 게시물 [Using the {{site.data.keyword.cloud_notm}} command line to find resources](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/)를 읽으십시오. 
- 인프라 사용량을 조사할 수 있는 추가 API를 보려면 [인프라 계정 API](https://softlayer.github.io/reference/services/SoftLayer_Account/)를 검토하십시오. 
- 사용량 데이터 집계를 위한 고급 조회를 보려면 [jq 매뉴얼](https://stedolan.github.io/jq/manual/)을 검토하십시오. 
