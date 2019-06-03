---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
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

# Criar, proteger e gerenciar APIs de REST
{: #create-manage-secure-apis}

Este tutorial demonstra como criar APIs de REST usando a Estrutura de API do Node.js LoopBack. Com o Loopback, é possível criar rapidamente APIs de REST que conectam dispositivos e navegadores a dados e serviços. Você também incluirá gerenciamento, visibilidade, segurança e limitação de taxa em suas APIs usando o {{site.data.keyword.apiconnect_long}}.
{:shortdesc}

## Objetivos

* Construir uma API de REST com pouca a nenhuma codificação
* Publique sua API no {{site.data.keyword.Bluemix_notm}} para atingir os desenvolvedores
* Trazer as APIs existentes para o {{site.data.keyword.apiconnect_short}}
* Expor e controlar de forma segura o acesso aos sistemas de registro

## Serviços usados

Este tutorial usa os tempos de execução e serviços a seguir:

* [ Loopback ](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* App do Cloud Foundry [SDK for Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs)

## Arquitetura

![Arquitetura](images/solution13/Architecture.png)

1. O desenvolvedor define a API de RESTful
2. O desenvolvedor publica a API no {{site.data.keyword.apiconnect_long}}
3. Os usuários e aplicativos consomem a API

## Antes de Começar

* Faça download e instale o [Node.js](https://nodejs.org/en/download/) versão 6.x (use [nvm](https://github.com/creationix/nvm) ou semelhante se você já tiver uma versão mais recente do Node.js instalada)

## Criar uma API de REST no Node.js

{: #create_api}
Nesta seção, você criará uma API no Node.js usando o [LoopBack](https://loopback.io/doc/index.html). LoopBack é uma estrutura Node.js de software livre altamente extensível que permite criar APIs de REST dinâmicas de ponta a ponta com pouca ou nenhuma codificação.

### Criar aplicativo

1. Instale a ferramenta de linha de comandos do {{site.data.keyword.apiconnect_short}}. Se você tiver problemas na instalação do {{site.data.keyword.apiconnect_short}}, use `sudo` antes do comando ou siga as instruções [aqui](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit).
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. Insira o comando a seguir para criar o aplicativo.
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. Pressione `Enter` para usar **entries-api** como o **nome de seu aplicativo**.
4. Pressione `Enter` para usar **entries-api** como o **diretório para conter o projeto**.
5. Escolha **3.x (atual)** como a **versão do LoopBack**.
6. Escolha **empty-server** como o **tipo de aplicativo**.

![Esqueleto do APIC Loopback](images/solution13/apic_loopback.png)

### Incluir uma origem de dados

As origens de dados representam sistemas back-end, como bancos de dados, APIs de REST externas, serviços da web SOAP e serviços de armazenamento. As origens de dados geralmente fornecem funções de criação, recuperação, atualização e exclusão (CRUD). Enquanto o Loopback suporte muitos tipos de [origens de dados](http://loopback.io/doc/en/lb3/Connectors-reference.html), por uma questão de simplicidade, você usará um armazenamento de dados na memória com sua API.

1. Mude o diretório para o novo projeto e ative o API Designer.
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. Na navegação, clique em **Origens de dados**. Em seguida, clique no botão **Incluir**. Um diálogo **Nova origem de dados de LoopBack** será aberto.
3. Digite `entriesDS` no campo de texto **Nome** e clique no botão **Novo**.
4. Selecione **Db na memória** na caixa de combinação **Conector**.
5. Clique em **Todas as origens de dados** na parte superior esquerda. A origem de dados aparecerá na lista de origens de dados.

   O editor atualiza automaticamente o arquivo server/datasources.json com configurações para a nova origem de dados.
   {:tip}

![Origens de dados do API Designer](images/solution13/datastore.png)

### Incluir um modelo

Um modelo é um objeto JavaScript com as APIs de Node e REST que representa dados em sistemas back-end. Os modelos são conectados a esses sistemas por meio de origens de dados. Nesta seção, você definirá a estrutura de seus dados e a conectará à origem de dados criada anteriormente.

1. Na navegação, clique em **Modelos**. Em seguida, clique no botão **Incluir**. Um diálogo **Novo modelo de LoopBack** será aberto.
2. Digite `entry` no campo de texto **Nome** e clique no botão **Novo**.
3. Selecione **entriesDS** na caixa de combinação **Origem de dados**.
4. No cartão **Propriedades**, inclua as propriedades a seguir usando o ícone **Incluir propriedade**.
    1. Digite `name` para o **Nome da propriedade** e selecione **sequência** na caixa de combinação **Tipo**.
    2. Digite `email` para o **Nome da propriedade** e selecione **sequência** na caixa de combinação **Tipo**.
    3. Digite `comment` para o **Nome da propriedade** e selecione **sequência** na caixa de combinação **Tipo**.
5. Clique no ícone **Salvar** no canto superior direito para salvar o modelo.

![Gerador de modelo](images/solution13/models.png)

## Testar seu aplicativo LoopBack

Nesta seção, você iniciará uma instância local de seu aplicativo Loopback e testará a API inserindo e consultando dados usando o API Designer. Deve-se usar a ferramenta Explore do API Designer para testar os terminais REST em seu navegador porque ela inclui os cabeçalhos de segurança adequados e outros parâmetros de solicitação.

1. Inicie o servidor local clicando no ícone **Iniciar** no canto inferior esquerdo e aguarde a mensagem **Em execução**.
2. No banner, clique no link **Explore** para ver a ferramenta Explore do API Designer. A barra lateral mostra as operações REST disponíveis para os modelos LoopBack na API.
3. Clique na operação **entry.create** na área de janela esquerda para exibir o terminal. A área de janela central exibe informações de resumo sobre o terminal, incluindo seus parâmetros, segurança, dados de instância do modelo e códigos de resposta. A área de janela direita fornece o código de amostra para chamar o terminal usando o comando cURL e as linguagens como Ruby, Python, Java e Node.
4. Na área de janela direita, clique no link **Testar**. Role para baixo até **Parâmetros** e insira o seguinte na área de texto **dados**:
    ```javascript {
      "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. Clique no botão **Chamar operação**.
   ![Testando uma API no API Designer](images/solution13/data_entry_1.png)
6. Confirme se o POST foi bem-sucedido verificando o **Código de resposta: 200 OK**. Se você vir uma mensagem de erro devido a um certificado não confiável para o host local, clique no link fornecido na mensagem de erro na ferramenta API Designer Explore para aceitar o certificado, em seguida, continue para chamar as operações em seu navegador da web. O procedimento exato depende do navegador da web que você está usando.
7. Inclua outra entrada usando cURL. Confirme se a porta corresponde à porta de seu aplicativo.
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. Clique na operação **entry.find**, em seguida, no link **Testar** seguido pelo botão **Chamar operação**. A resposta exibe todas as entradas. Você deverá ver JSON para **Jane Doe** e **John Doe**.
   ![entry_find](images/solution13/find_response.png)

Também é possível iniciar manualmente o aplicativo Loopback emitindo o comando `npm start` por meio do diretório `entries-api`. As suas APIs de REST estarão disponíveis em [http://localhost:3000/api/entries](http://localhost:3000/api/entries); no entanto, elas não serão gerenciadas e protegidas pelo {{site.data.keyword.apiconnect_short}}.
{:tip}

## Criar serviço {{site.data.keyword.apiconnect_short}}

Para preparar-se para as próximas etapas, você criará um serviço **{{site.data.keyword.apiconnect_short}}** no {{site.data.keyword.Bluemix_notm}}. O {{site.data.keyword.apiconnect_short}} age como o gateway para sua API e também fornece limites de gerenciamento, segurança e taxa.

1. Ative a [Lista de recursos](https://{DomainName}/resources) do {{site.data.keyword.Bluemix_notm}}.
2. Navegue para **Catálogo > Integração > {{site.data.keyword.apiconnect_short}}** e clique no botão **Criar**.

## Publicar uma API no {{site.data.keyword.Bluemix_notm}}

{: #publish}
Você usará o API Designer para implementar seu aplicativo no {{site.data.keyword.Bluemix_notm}} como um aplicativo do Cloud Foundry e também publicará sua definição de API para o **{{site.data.keyword.apiconnect_short}}**. O API Designer é seu kit de ferramentas local. Se você o fechou, reative-o com `apic edit` no diretório de projeto.

O aplicativo também pode ser implementado manualmente usando o comando `ibmcloud cf push`; no entanto, ele não será protegido. Para [importar a API](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing) para o {{site.data.keyword.apiconnect_short}}, use o arquivo de definição do OpenAPI disponível na pasta `definitions`. A implementação usando o API Designer protege o aplicativo e importa a definição automaticamente.
{:tip}

1. De volta ao API Designer, clique no link **Publicar** no banner. Em seguida, clique em **Incluir e gerenciar destinos > Incluir destino do IBM Bluemix**.
2. Selecione a **Região** e a **Organização** para as quais você deseja publicar.
3. Selecione o Catálogo **Ambiente de simulação** e clique em **Avançar**.
4. Digite `entries-api-application` no campo de texto **Digitar um novo nome do aplicativo** e clique no ícone **+**.
5. Clique em **entries-api-application** na lista e clique em **Salvar**.
6. Clique no ícone de hambúrguer **Menu** no canto superior esquerdo do banner. Em seguida, clique em **Projetos** e no item de lista **entries-api**.
7. Na IU do API Designer, clique nos links **APIs > entries-api > Montar**.
8. No editor Montar, clique no ícone **Políticas de filtro**.
9. Selecione **Políticas do DataPower Gateway** e clique em **Salvar**.
10. Clique em **Publicar** na barra superior e selecione seu destino. Selecione **Publicar aplicativo** e Estágio ou Publicar produtos > Selecione **Produtos específicos** > **entries-api**.
11. Clique em **Publicar** e aguarde até que o aplicativo conclua a publicação.
    ![Diálogo de publicação](images/solution13/publish.png)

    Um aplicativo contém os modelos de Loopback, as origens de dados e o código que está relacionado à sua API. Um produto permite que você declare como uma API é disponibilizada para os desenvolvedores.
    {:tip}

O aplicativo de API é agora publicado no {{site.data.keyword.Bluemix_notm}} como um aplicativo do Cloud Foundry. É possível vê-lo procurando nos aplicativos do Cloud Foundry na [Lista de recursos](https://{DomainName}/resources) do {{site.data.keyword.Bluemix_notm}}, mas o acesso direto usando a URL não é possível, pois o aplicativo está protegido. A próxima seção mostrará como as APIs gerenciadas podem ser acessadas.

## Gateway de API

Até agora, você estava projetando e testando sua API localmente. Nesta seção, você usará o {{site.data.keyword.apiconnect_short}} para testar sua API implementada no {{site.data.keyword.Bluemix_notm}}.

1. Ativa a [Lista de recursos](https://{DomainName}/resources) do {{site.data.keyword.Bluemix_notm}}.
2. Localize e selecione seu serviço **{{site.data.keyword.apiconnect_short}}** em **Serviços do Cloud Foundry**.
3. Clique no menu **Explore** e, em seguida, clique no link **Ambiente de simulação**.
4. Clique na operação **entry.create**.
5. Na área de janela direita, clique em **Experimente**. Role para baixo até **Parâmetros** e insira o seguinte na área de texto **dados**.
    ```javascript {
      "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. Clique no botão **Chamar operação**. Uma resposta **Code: 200** deve ser exibida indicando sucesso.

![gateway](images/solution13/gateway.png)

Sua URL de API gerenciada e segura é exibida ao lado de cada operação e deve ser semelhante a `https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries`.
{: tip}

## Limitação de taxa

Configurar os limites de taxa permite gerenciar o tráfego de rede para suas
APIs e para operações específicas dentro de suas APIs. Um limite de taxa é o número máximo de chamadas permitidas em um intervalo de tempo específico.

1. De volta ao API Designer, clique em **Produtos > entries-api**.
2. Selecione **Plano padrão** à esquerda.
3. Expanda **Plano padrão** e role para baixo até o campo **Limites de taxa**.
4. Configure os campos para **10** chamadas / **1****Minuto**.
5. Selecione **Cumprir limite máximo** e clique no ícone **Salvar**.
   ![Página de limite de taxa](images/solution13/rate_limit.png)
6. Siga as etapas na seção [Publicar API no {{site.data.keyword.Bluemix_notm}}](#publish) para publicar novamente sua API.

Sua API está agora limitada a 10 solicitações por minuto. Use o recurso **Testar** para atingir o limite. Consulte mais informações sobre [Configurando limites de taxa](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits) ou explore o API Designer para ver todos os recursos de gerenciamento disponíveis.

## Expandir o tutorial

Parabéns, você construiu uma API que é gerenciada e segura. Abaixo estão sugestões adicionais para aprimorar sua API.

* Incluir a persistência de dados usando o Conector [{{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant) LoopBack
* Usar o API Designer para [visualizar configurações adicionais](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0) para gerenciar sua API
* Revisar **Análise de dados** e **Visualizações** da API [disponíveis](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics) no {{site.data.keyword.apiconnect_short}}

## Conteúdo relacionado

* [Documentação de Loopback](https://loopback.io/doc/index.html)
* [Introdução ao {{site.data.keyword.apiconnect_long}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
