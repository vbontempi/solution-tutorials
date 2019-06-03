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

# Construir um data lake usando o armazenamento de objetos
{: #smart-data-lake}

As definições do termo data lake variam, mas no contexto deste tutorial, um data lake é uma abordagem para armazenar dados em seu formato nativo para uso organizacional. Para esse fim, você criará um data lake para sua organização usando o {{site.data.keyword.cos_short}}. Combinando o {{site.data.keyword.cos_short}} e o SQL Query, os analistas de dados podem consultar dados nos quais ele está usando SQL. Você também alavancará o serviço SQL Query em um Jupyter Notebook para conduzir uma análise simples. Quando tiver concluído, permita que os usuários não técnicos descubram seus próprios insights usando o {{site.data.keyword.dynamdashbemb_notm}}.

## Objetivos

- Usar o {{site.data.keyword.cos_short}} para armazenar arquivos de dados brutos
- Consultar dados diretamente do {{site.data.keyword.cos_short}} usando o SQL Query
- Refinar e analisar dados no {{site.data.keyword.DSX_full}}
- Compartilhe dados em sua organização com o {{site.data.keyword.dynamdashbemb_notm}}

## Serviços usados

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [ SQL Query ](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Arquitetura

![Arquitetura](images/solution29/architecture.png)

1. Os dados brutos são armazenados no {{site.data.keyword.cos_short}}
2. Os dados são reduzidos, aprimorados ou refinados com o SQL Query
3. A análise de dados ocorre no {{site.data.keyword.DSX}}
4. A linha de negócios acessa um aplicativo da web
5. Os dados refinados são puxados do {{site.data.keyword.cos_short}}
6. Os gráficos de linha de negócios são construídos usando o {{site.data.keyword.dynamdashbemb_notm}}

## Antes de Começar

- [Instale o Git](https://git-scm.com/)
- [Instale a CLI do {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
- [Instale o Aspera Connect](http://downloads.asperasoft.com/connect2/)
- [Instale o Node.js e o NPM](https://nodejs.org)

## Criar serviços

Nesta seção, você criará os serviços necessários para construir seu data lake.

Esta seção usa a linha de comandos para criar instâncias de serviço. Como alternativa, é possível fazer o mesmo por meio da página de serviço no catálogo usando os links fornecidos.
{: tip}

1. Efetue login no {{site.data.keyword.cloud_notm}} por meio da linha de comandos e tenha como destino sua conta do Cloud Foundry. Consulte [Introdução à CLI]https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
    ```sh ibmcloud login
    ```
    {: pre}
    ```sh ibmcloud target --cf
    ```
    {: pre}
2. Crie uma instância do [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) com um alias do Cloud Foundry. Se você já tiver uma instância de serviço, execute o comando `service-alias-create` com o nome do serviço existente.
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. Crie uma instância do [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. Crie uma instância do [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio).
    ```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. Crie uma instância do [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded) com um alias do Cloud Foundry.
    ```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. Mude para um diretório ativo e execute o comando a seguir para clonar o [Repositório GitHub](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard) do aplicativo de painel. Em seguida, envie por push o aplicativo para sua organização do Cloud Foundry. O aplicativo ligará automaticamente os serviços necessários acima usando seu arquivo [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml).
    ```sh
    git clone https://github.com/IBM-Cloud/nodejs-data-lake-dashboard.git
    ```
    {: pre}
    ```sh
    cd nodejs-data-lake-dashboard
    ```
    {: pre}
    ```sh npm install
    ```
    {: pre}
    ```sh
    npm run push
    ```
    {: pre}

    Após a implementação, o aplicativo será público e atendendo em um nome do host aleatório. É possível chegar à página [Lista de recursos](https://{DomainName}/resources), selecionar o app em Apps do Cloud Foundry e visualizar a URL ou executar o comando `ibmcloud cf app dashboard-nodejs routes` para ver as rotas.
    {: tip}

7. Confirme se o aplicativo está ativo acessando sua URL pública no navegador.

![Página inicial do painel](images/solution29/dashboard-start.png)

## Fazendo upload de dados

Nesta seção, você fará upload de dados para um depósito do {{site.data.keyword.cos_short}} usando o {{site.data.keyword.CHSTSshort}} integrado. O {{site.data.keyword.CHSTSshort}} protege os dados conforme eles são transferidos por upload para o depósito e [pode reduzir significativamente o tempo de transferência](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/).

1. Faça download do arquivo CSV [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD). O arquivo tem 81 MB e pode levar alguns minutos para fazer download.
2. Em seu navegador, acesse a instância de serviço **data-lake-cos** por meio da [Lista de recursos](https://{DomainName}/resources).
3. Crie um novo depósito para armazenar dados.
    - Clique no botão **Criar um depósito**.
    - Selecione **Regional** na lista suspensa **Resiliência**.
    - Selecione **us-south** no **Local**. O {{site.data.keyword.CHSTSshort}} está disponível somente para depósitos criados no local `us-south` neste momento. Como alternativa, escolha outro local e use o tipo de transferência **Padrão** na próxima seção.
    - Forneça um depósito **Nome** e clique em **Criar**. Se você receber um erro *AccessDenied*, tente com um nome de depósito mais exclusivo.
4. Faça upload do arquivo CSV para o {{site.data.keyword.cos_short}}.
    - No depósito, clique no botão **Incluir objetos**.
    - Selecione o botão de opções **Transferência de alta velocidade do Aspera**.
    - Clique no botão **Incluir arquivos**. Isso abrirá o plug-in Aspera, que estará em uma janela separada, possivelmente atrás da janela do navegador.
    - Procure e selecione o arquivo CSV transferido por download anteriormente.

![Depósito com arquivo CSV](images/solution29/cos-bucket.png)

## Trabalhando com dados

Nesta seção, você converterá o conjunto original de dados brutos em um coorte destinado com base em atributos de tempo e idade. Isso é útil para os consumidores de data lake que têm interesses específicos ou que têm problemas com conjuntos de dados muito grandes.

Você usará o SQL Query para manipular os dados nos quais ele reside em {{site.data.keyword.cos_short}} usando instruções SQL familiares. O SQL Query tem suporte integrado para CSV, JSON e Parquet. Nenhum serviço de cálculo adicional ou extração-transformação-carregamento é necessário.

1. Acesse a instância do serviço SQL Query **data-lake-sql** por meio de sua [Lista de recursos](https://{DomainName}/resources).
2. Selecione **Abrir IU**.
3. Crie um novo conjunto de dados, executando SQL diretamente no arquivo CSV transferido por upload anteriormente.
    - Insira o SQL a seguir na área de texto **Digitar SQL aqui...**
        ```
        SELECIONAR
        `Dr Number` AS id,
        `Date Occurred` AS date,
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
        FROM cos://us-south/<your-bucket-name>/Traffic_Collision_Data_from_2010_to_Present.csv
        WHERE
        `Time Occurred` >= 1700 AND
        `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND
        `Victim Age` <= 35
        ```
        {: codeblock}
    - Substitua a URL na cláusula `FROM` pelo nome do seu depósito.
4. O **Destino** criará automaticamente um depósito do {{site.data.keyword.cos_short}} conter o resultado. Mude o **Destino** para `cos://us-south/<your-bucket-name>/results`.
5. Clique no botão **Executar**. Os resultados aparecerão abaixo.
6. Na guia **Detalhes da consulta**, clique no ícone **Ativar** logo depois da URL **Local de resultado** para visualizar o conjunto de dados intermediário, que agora também está armazenado no {{site.data.keyword.cos_short}}.

![Notebook](images/solution29/sql-query.png)

## Combinar os Jupyter Notebooks com o SQL Query

Nesta seção, você usará o cliente SQL Query dentro de um Jupyter Notebook. Isso reutiliza os dados armazenados em {{site.data.keyword.cos_short}} dentro de uma ferramenta de análise de dados. A combinação também cria conjuntos de dados que são armazenados automaticamente no {{site.data.keyword.cos_short}}, que podem, então, ser usados com o {{site.data.keyword.dynamdashbemb_notm}}.

1. Crie um novo Jupyter Notebook no {{site.data.keyword.DSX}}.
    - Em um navegador, abra [{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true).
    - Clique no ladrilho **Criar um projeto** seguido por **Ciência de dados**.
    - Clique em **Criar projeto** e, em seguida, forneça um **Nome do projeto**.
    - Assegure-se de que **Armazenamento** esteja configurado como **data-lake-cos**.
    - Clique em **Criar**.
    - No projeto resultante, clique em **Incluir no projeto** e **Bloco de notas**.
    - Na guia **Em branco**, insira um **Nome do bloco de notas**.
    - Deixe os padrões de **Idioma** e **Tempo de execução** ; clique em **Criar bloco de notas**.
2. No Bloco de notas, instale e importe PixieDust e ibmcloudsql, incluindo os comandos a seguir no prompt de entrada **In [ ]:** e, em seguida, **Executar**.
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. Inclua uma chave de API do {{site.data.keyword.cos_short}} no Bloco de notas. Isso permitirá que os resultados do SQL Query sejam armazenados no {{site.data.keyword.cos_short}}.
    - Inclua o seguinte no próximo prompt **In [ ]:** e, em seguida, **Executar**.
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - No terminal, crie uma chave de API.
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - Copie a **Chave de API** para a área de transferência.
    - Cole a Chave de API na caixa de texto no Bloco de notas e pressione a tecla `enter`.
    - Você também deve armazenar a Chave de API em um local seguro e permanente; o Bloco de notas não armazena a chave de API.
4. Inclua o CRN (Cloud Resource Name) da instância do SQL Query no Bloco de notas.
    - No próximo prompt **In [ ]:** designe o CRN a uma variável em seu Bloco de notas.
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - No terminal, copie o CRN da propriedade **ID** para a sua área de transferência.
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - Cole o CRN entre as aspas simples e, em seguida, **Executar**.
5. Inclua outra variável no Bloco de notas para especificar o depósito do {{site.data.keyword.cos_short}} e **Executar**.
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. Execute os comandos a seguir em outro prompt **In [ ]:** e **Executar** para visualizar o conjunto de resultados. Você também terá um novo arquivo `accidents/jobid=<id>/<part>.csv*` incluído em seu depósito que inclui o resultado do `SELECT`.
    ```python
    sqlClient = ibmcloudsql.SQLQuery(cloud_api_key, sql_crn, sql_cos_endpoint + '/accidents')

    data_source = sql_cos_endpoint + "/Traffic_Collision_Data_from_2010_to_Present.csv"

    query = """
    SELECT
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
    FROM  {}
    WHERE
        `Time Occurred` >= 1700 AND `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND `Victim Age` <= 35
    """.format(data_source)

    traffic_collisions = sqlClient.run_sql(query)
    traffic_collisions.head()
    ```
    {: codeblock}

## Visualize dados usando PixieDust

Nesta seção, você visualizará o conjunto de resultados anterior usando PixieDust e o Mapbox para identificar melhor os padrões ou pontos de acesso para incidentes de tráfego.

1. Crie uma expressão de tabela comum para converter a coluna `location` em colunas `latitude` e `longitude` separadas. **Execute** o seguinte no prompt do Bloco de notas.
    ```python
    query = """
    WITH location AS (
        SELECT
            id,
            cast(split(coordinates, ',')[0] as float) as latitude,
            cast(split(coordinates, ',')[1] as float) as longitude
        FROM (SELECT
                `Dr Number` as id,
                regexp_replace(Location, '[()]', '') as coordinates
            FROM {0}
        )
    )
    SELECT
        d.`Dr Number` as id,
        d.`Date Occurred` as date,
        d.`Time Occurred` AS time,
        d.`Area Name` AS area,
        d.`Victim Age` AS age,
        d.`Victim Sex` AS sex,
        l.latitude,
        l.longitude
    FROM {0} AS d
        JOIN
        location AS l
        ON l.id = d.`Dr Number`
    WHERE
        d.`Time Occurred` >= 1700 AND
        d.`Time Occurred` <= 2000 AND
        d.`Victim Age` >= 20 AND
        d.`Victim Age` <= 35 AND
        l.latitude != 0.0000 AND
        l.latitude != 0.0000
    """.format(data_source)

    traffic_location = sqlClient.run_sql(query)
    traffic_location.head()
    ```
    {: codeblock}
2. No próximo prompt **In [ ]:**, **Execute** o comando `display` para visualizar o resultado usando PixieDust.
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. Selecione o botão de menu suspenso; em seguida, selecione **Mapa**.
4. Inclua `latitude` e `longitude` em **Chaves**. Inclua `id` e `age` em **Valores**. Clique em **OK** para visualizar o mapa.
5. Clique no ícone **Salvar** para salvar seu Bloco de notas no {{site.data.keyword.cos_short}}.

![Bloco de notas](images/solution29/notebook-mapbox.png)

## Compartilhar seu conjunto de dados com a organização

Nem todos os usuários do data lake são um cientista de dados. É possível permitir que usuários não técnicos obtenham insight do data lake usando o {{site.data.keyword.dynamdashbemb_notm}}. De forma semelhante ao SQL Query, o {{site.data.keyword.dynamdashbemb_notm}} pode ler dados diretamente do {{site.data.keyword.cos_short}} usando painéis pré-construídos. Esta seção apresenta uma solução que permite que qualquer usuário acesse o data lake e construa um painel customizado.

1. Acesse a URL pública do aplicativo de painel que você enviou por push para o {{site.data.keyword.Bluemix_notm}} anteriormente.
2. Selecione um modelo que corresponda ao layout desejado. (As etapas a seguir usam o segundo layout na primeira linha.)
3. Use o botão `Add a source` que aparece na prateleira lateral `Selected sources`, expanda a sanfona `bucket name` e clique em um das entradas de tabela `accidents/jobid=...`. Feche o diálogo usando o ícone X na parte superior direita.
4. À esquerda, clique no ícone `Visualizations` e, em seguida, clique em **Resumo**.
5. Selecione a origem `accidents/jobid=...`, expanda `Table` e crie um gráfico.
    - Arraste e solte `id` na linha **Valor**.
    - Reduza o gráfico usando o ícone no canto superior.
6. Novamente em `Visualizações`, crie um gráfico **Mapa de árvore**:
    - Arraste e solte `area` na linha **Hierarquia de área**.
    - Arraste e solte `id` na linha **Tamanho**.
    - Reduza o gráfico para visualizar o resultado.

![Gráfico de painel](images/solution29/dashboard-chart.png)

## Explorar seu painel

Nesta seção, você executará algumas etapas adicionais para explorar os recursos do aplicativo de painel e do {{site.data.keyword.dynamdashbemb_notm}}.

1. Clique no botão **Modo** na barra de ferramentas do aplicativo de painel de amostra para mudar a visualização de modo `VIEW`.
2. Clique em qualquer um dos ladrilhos coloridos no gráfico inferior ou em valores de `area` na legenda do gráfico. Isso aplica um filtro local à guia, que faz com que os outros gráficos mostrem dados específicos para o filtro.
3. Clique no botão **Salvar** na barra de ferramentas.
    - Insira o nome de seu painel no campo de entrada correspondente.
    - Selecione a guia **Especificação** para visualizar a especificação desse painel. Uma especificação é o formato de arquivo nativo para {{site.data.keyword.dynamdashbemb_notm}}. Nela, você localizará informações sobre os gráficos criados, bem como a origem de dados do {{site.data.keyword.cos_short}} usada.
    - Salve seu painel no armazenamento local do navegador usando o botão **Salvar** do diálogo.
4. Clique no botão **Novo** da barra de ferramentas para criar um novo painel. Para abrir um painel salvo, clique no botão **Abrir**. Para excluir um painel, use o ícone **Excluir** no diálogo Abrir painel.

Em aplicativos de produção, criptografe as informações, como URLs, nomes de usuários e senhas, para evitar que elas sejam vistas pelos usuários finais. Consulte [Criptografando informações de origem de dados](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information).
{: tip}

## Expandir o tutorial

Parabéns, você construiu um data lake usando o {{site.data.keyword.cos_short}}. Abaixo estão sugestões adicionais para aprimorar seu data lake.

- Experimentar com conjuntos de dados adicionais usando o SQL Query
- Transmitir dados de múltiplas origens para seu data lake, concluindo [Logs de big data com análise de dados de fluxo e SQL](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics)
- Editar o código do aplicativo de painel para armazenar as especificações do painel no [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) ou {{site.data.keyword.cos_short}}
- Criar uma instância de serviço do [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id) para ativar a segurança no aplicativo de painel

## Remover recursos

Execute os comandos a seguir para remover serviços, aplicativos e chaves usados.

```sh
ibmcloud resource service-binding-delete dashboard-nodejs-dde dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-binding-delete dashboard-nodejs-cos dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-dde
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-cos
```
{: pre}
```sh
ibmcloud iam api-key-delete data-lake-cos-key
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-dde
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-cos
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-sql
```
{: pre}
```sh
ibmcloud service delete data-lake-studio
```
{: pre}
```sh
ibmcloud app delete dashboard-nodejs
```
{: pre}

## Conteúdo relacionado

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Jupyter Notebooks](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
