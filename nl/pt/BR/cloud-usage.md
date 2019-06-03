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

# Revisando serviços, recursos e uso do {{site.data.keyword.cloud_notm}}
{: #cloud-usage}
À medida que a adoção do Cloud aumentar, os gerentes de TI e de finanças precisarão entender o uso do Cloud no contexto de inovação e controle de custo. Perguntas como: "Quais serviços as equipes estão usando?", "Quanto custa para operar uma solução?" e "Como posso conter a expansão?" podem ser respondidas investigando os dados disponíveis. Este tutorial apresenta maneiras de explorar essas informações e responder perguntas comuns relacionadas ao uso.
{:shortdesc}

## Objetivos
{: #objectives}
* Detalhar em itens os artefatos do {{site.data.keyword.cloud_notm}}: apps e serviços do Cloud Foundry, recursos do Identity and Access Management e dispositivos de Infraestrutura
* Associar artefatos do {{site.data.keyword.cloud_notm}} com uso e faturamento
* Definir os relacionamentos entre artefatos do {{site.data.keyword.cloud_notm}} e equipes de desenvolvimento
* Alavancar dados de uso e faturamento para criar conjuntos de dados para propósitos de contabilidade

## Arquitetura
{: #architecture}

![Arquitetura](images/solution38/Architecture.png)

## Antes de Começar
{: #prereqs}

* Instale a [CLI do {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* Instale o [cURL](https://curl.haxx.se/)
* Instale o [Node.js](https://nodejs.org/)
* Instale o [json2csv](https://www.npmjs.com/package/json2csv) usando o comando `npm install -g json2csv`
* Instale o [jq](https://stedolan.github.io/jq/)

## Segundo plano
{: #background}

Antes de executar comandos que inventariam e detalham o uso do {{site.data.keyword.cloud_notm}}, é útil ter algum segundo plano sobre as categorias amplas de uso e suas funções. Os termos chave usados posteriormente no tutorial estão em negrito. Uma visualização útil dos artefatos abaixo pode ser localizada na [documentação Gerenciando sua conta](https://{DomainName}/docs/account?topic=account-overview#overview).

### Cloud Foundry
O Cloud Foundry é uma plataforma como serviço (PaaS) de software livre no {{site.data.keyword.cloud_notm}} que permite implementar e escalar aplicativos e **Serviços** sem gerenciar servidores. O Cloud Foundry organiza os aplicativos e serviços em organizações ou espaços. Uma **Organização** é uma conta de desenvolvimento que um ou muitos usuários podem possuir e usar. Uma organização pode conter múltiplos espaços. Cada **Espaço** fornece aos usuários acesso a um local compartilhado para desenvolvimento de aplicativo, implementação e manutenção.

### Identity and Access Management
O IBM Cloud Identity and Access Management (IAM) permite autenticar usuários de forma segura para ambos os serviços de plataforma e controlar o acesso aos recursos de forma consistente na plataforma {{site.data.keyword.cloud_notm}}. Ofertas mais recentes e serviços do Cloud Foundry migrados existem como **Recursos** gerenciados pelo {{site.data.keyword.cloud_notm}} Identity and Access Management. Os recursos são organizados em **Grupos de recursos** e fornecem controle de acesso por meio de Políticas e Funções.

### Infraestrutura
A infraestrutura abrange uma variedade de opções de cálculo: servidores bare metal, instâncias de servidor virtual e nós do Kubernetes. Cada um é visto como um **Dispositivo** no console.

### Conta
Os artefatos supramencionados são associados a uma **Conta** para propósitos de faturamento. Os **Usuários** são convidados para a conta e eles têm acesso concedido aos diferentes recursos dentro da conta.

## Designar permissões
Para visualizar o inventário e o uso do Cloud, você precisará das funções apropriadas designadas pelo administrador de conta. Se você for o administrador da conta, continue com a próxima seção.

1. O administrador de conta deve efetuar login no {{site.data.keyword.cloud_notm}} e acessar a página [**Usuários de identidade e acesso**](https://{DomainName}/iam/#/users).
2. O administrador de conta pode selecionar seu nome na lista de usuários na conta para designar as permissões apropriadas.
3. Na guia **Políticas de acesso**, clique no botão **Designar acesso** e execute as mudanças a seguir.
   1. Selecione o ladrilho **Designar acesso dentro de um grupo de recursos**. Selecione os **Grupos de recursos** aos quais conceder acesso e aplique a função de **Visualizador**. Conclua clicando no botão **Designar**. Essa função é necessária para os comandos `resource groups` e `billing resource-group-usage`.
   2. Selecione o ladrilho **Designar acesso usando o Cloud Foundry**. Selecione o menu overflow ao lado de cada **Organização** para a qual conceder acesso. Selecione **Editar função de organização** no menu. Selecione **Gerente de faturamento** na lista **Funções de organização**. Conclua clicando no botão **Salvar função**. Essa função é necessária para o comando `billing org-usage`.
   3. Selecione o ladrilho **Designar acesso aos recursos**. Selecione **Todos os serviços ativados por Identidade e Acesso** no menu suspenso **Serviços**. Verifique a função de **Editor** em **Designar funções de acesso da plataforma**. Essa função é necessária para o comando `resource tag-attach`.

## Localizar recursos com o comando de procura
{: #search}

À medida que as equipes de desenvolvimento começarem a usar os serviços do Cloud, os gerenciadores se beneficiarão de saber quais serviços foram implementados. As informações de implementação ajudam a responder perguntas relacionadas à inovação e ao gerenciamento de serviços:
- Quais qualificações relacionadas ao serviço podem ser compartilhadas entre as equipes para aprimorar outros projetos?
- Quais são os compartilhamentos entre as equipes que podem estar ausentes em algumas?
- Quais equipes estão usando um serviço que requer uma correção crítica ou que em breve será descontinuado?
- Como as equipes podem revisar suas instâncias de serviço para minimizar a expansão?

A procura não é limitada a serviços e recursos. Também é possível consultar artefatos do Cloud, como organizações e espaços do Cloud Foundry, grupos de recursos, ligações de recursos, aliases, etc. Para obter mais exemplos, consulte a documentação [ibmcloud resource search](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search).
{:tip}

1. Efetue login no {{site.data.keyword.cloud_notm}} por meio da linha de comandos e tenha como destino sua conta do Cloud Foundry. Consulte [Introdução à CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh ibmcloud login
    ```
    {: pre}

    ```sh
    ibmcloud target --cf
    ```
    {: pre}

2. Inventariar todos os serviços do Cloud Foundry usados dentro da conta.
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. Os filtros booleanos podem ser aplicados a procuras amplas ou estreitas. Por exemplo, localize serviços e apps do Cloud Foundry, bem como recursos do IAM usando a consulta abaixo.
    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. Para notificar as equipes usando um tipo de serviço específico, consulte usando o nome do serviço. Substitua `<name>` por texto, por exemplo, `weather` ou `cloudant`. Em seguida, obtenha o nome da organização, substituindo `<Organization ID>` pelo **ID da organização**.
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

A identificação e a procura podem ser usadas juntas para fornecer identificação customizada de recursos. Isso envolve: anexar a tag aos recursos e procurar usando os nomes de tag. Crie uma tag denominada env:tutorial.

1. Anexe a tag a um recurso. É possível obter um CRN de recurso por meio da interface com o usuário ou com `ibmcloud resource service-instance <name|id>`. Os Cloud Resource Names (CRNs) identificam exclusivamente os recursos do IBM Cloud.
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <resource CRN>
   ```
   {: pre}
1. Procure os artefatos do Cloud que correspondem a uma determinada tag usando a consulta abaixo.
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

Combinando consultas de procura avançada com um esquema de identificação de acordo corporativo, os gerenciadores e as lideranças de equipe podem identificar mais facilmente e tomar uma ação em apps, recursos e serviços do Cloud.

## Explorar o uso com o Painel de uso
{: #dashboard}

Depois que o gerenciamento estiver ciente dos serviços que as equipes estão usando, a próxima pergunta feita com frequência é: "Qual é o custo para operar esses serviços?" O meio mais direto de determinar o uso e o custo é revisando o Painel de uso do {{site.data.keyword.cloud_notm}}.

1. Efetue login no {{site.data.keyword.cloud_notm}} e acesse o [Painel de uso da conta](https://{DomainName}/account/usage).
2. No menu suspenso **Grupos**, selecione uma Organização do Cloud Foundry para visualizar o uso do serviço.
3. Para uma determinada **Oferta de serviços**, clique em **Visualizar instâncias** para visualizar as instâncias de serviço que foram criadas.
4. Na página a seguir, escolha uma instância e clique em **Visualizar instância**. A página resultante fornece detalhes sobre a instância, como Organização, Espaço e Local, bem como itens de linha individuais que constroem o custo total.
5. Usando a trilha de navegação, visite novamente o [Painel de uso](https://{DomainName}/account/usage).
6. No menu suspenso **Grupos**, mude o seletor para **Grupos de recursos** e selecione um grupo como **padrão**.
7. Conduza uma revisão semelhante de instâncias disponíveis.

## Explorar o uso com a linha de comandos

Nesta seção, você explorará o uso com a interface da linha de comandos.

1. Liste todas as organizações do Cloud Foundry disponíveis para você e configure uma variável de ambiente para armazenar uma para teste.
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<org name>
    ```
    {: pre}

2. Detalhe em itens o faturamento e o uso para uma determinada organização com o comando `billing`. Especifique um mês específico usando a sinalização `-d` com uma data no formato AAAA-MM.
    ```sh ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Conduza a mesma investigação para recursos. Cada conta tem um grupo de recursos `default`. Substitua o valor `<group>` para um grupo de recursos listado no primeiro comando.
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

4. É possível visualizar os serviços do Cloud Foundry e os recursos do IAM usando o comando `resource-instances-usage`. Dependendo de seu nível de permissão, execute os comandos apropriados.
    - Se você for o administrador de conta ou tiver recebido a função de Administrador para todos os serviços ativados por Identidade e Acesso, execute apenas o `resource-instances-usage`.
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  Se você não for o administrador de conta, os comandos a seguir poderão ser usados por causa da função de Visualizador que foi configurada no início do tutorial.
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. Para visualizar dispositivos de infraestrutura, use o comando `sl`. Em seguida, use o comando `vs` para revisar {{site.data.keyword.virtualmachinesshort}}.
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## Exportar o uso com a linha de comandos
{: #usagecmd}

Um gerenciamentos precisa, muitas vezes, que os dados sejam exportados para outro aplicativo. Um exemplo comum é exportar dados de uso para uma planilha. Nesta seção, você exportará dados de uso para o formato de valor separado por vírgula (CSV), que é compatível com a maioria dos aplicativos de planilha.

Esta seção usa duas ferramentas de terceiro: `jq` e `json2csv`. Cada um dos comandos abaixo é composto de três etapas: obtendo dados de uso como JSON, analisando o JSON e formatando o resultado como CSV. O processo de obtenção e conversão de dados JSON não é limitado a `json2csv` e outras ferramentas podem ser alavancadas.

Use a opção `-p` para impressão elegante dos resultados. Se a impressão dos dados for ruim, remova o argumento `-p` para imprimir os dados brutos CSV.
{:tip}

1. Exporte o uso de um grupo de recursos com custos antecipados para cada tipo de recurso.
    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. Detalhe em itens as instâncias para cada tipo de recurso no grupo de recursos.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. Siga a mesma abordagem para uma Organização do Cloud Foundry.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. Inclua custos antipados nos dados usando uma consulta `jq` mais avançada. Isso criará mais linhas, já que alguns tipos de recursos têm múltiplas métricas de custo.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. Use a mesma consulta `jq` para listar também os recursos do Cloud Foundry com custos associados.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. Exporte os dados para um arquivo removendo a opção `-p` e canalizando a saída para um arquivo. O arquivo CSV resultante pode ser aberto com um aplicativo de planilha.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## Exportar o uso com APIs
{: #usageapi}

Embora os comandos `billing` sejam úteis, a tentativa de montar uma visualização de "cenário geral" usando a interface da linha de comandos é tediosa. Da mesma forma, o Painel de uso apresenta uma visão geral de Organizações e Grupos de recursos, mas não necessariamente o uso de uma equipe ou de projeto. Nesta seção, você começará a explorar uma abordagem mais acionada por dados para obter o uso para requisitos customizados.

1. No terminal, configure a variável de ambiente `IBMCLOUD_TRACE=true` para imprimir solicitações e respostas da API.
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. Execute novamente o comando `billing org-usage` para ver as chamadas da API. Observe que múltiplos hosts e rotas de API são usados para esse único comando.
    ```sh ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Obtenha um token OAuth e execute uma das chamadas da API vistas no comando `billing org-usage`. Observe que algumas APIs usam o Token do UAA enquanto outras podem usar um Token do IAM como autorização de portador.
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

4. Para executar APIs de infraestrutura, obtenha e configure variáveis de ambiente para seu **Nome do usuário da API** e **Chave de autenticação** vistos em seu [Perfil do usuário](https://control.softlayer.com/account/user/profile). Se você não vir um Nome do usuário da API e uma Chave de autenticação, será possível criar um no menu **Ações** ao lado do seu nome na [Lista de usuários](https://control.softlayer.com/account/users).
    ```sh
    export IAAS_USERNAME=<API Username>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<Authentication Key>
    ```
    {: pre}

5. Obtenha os totais de faturamento de infraestrutura e os itens de faturamento usando as APIs a seguir. APIs semelhantes estão documentadas [aqui](https://softlayer.github.io/reference/services/SoftLayer_Account/).
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. Desative o rastreio.
    ```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

Embora a abordagem acionada por dados forneça a maior flexibilidade na exploração de uso, uma explicação mais completa está além do escopo deste tutorial. O projeto GitHub [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample) foi criado e combina os serviços do {{site.data.keyword.cloud_notm}} com o Cloud Functions para fornecer uma implementação de amostra.

## Expandir o tutorial
{: #expand}

Use as sugestões a seguir para expandir sua investigação sobre o inventário e os dados relacionados ao uso.

- Explore os comandos `ibmcloud billing` com a opção `--output json`. Isso mostrará propriedades de dados adicionais disponíveis e não cobertas no tutorial.
- Leia a postagem do blog [Usando a linha de comandos do {{site.data.keyword.cloud_notm}} para localizar recursos](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/) para obter mais exemplos sobre `ibmcloud resource search` e quais propriedades podem ser usadas em suas consultas.
- Revise as [APIs de conta de infraestrutura](https://softlayer.github.io/reference/services/SoftLayer_Account/) para obter APIs adicionais para investigar o uso da infraestrutura.
- Revise o [Manual jq](https://stedolan.github.io/jq/manual/) para consultas avançadas para agregar dados de uso.
