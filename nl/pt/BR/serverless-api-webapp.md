---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}


# Aplicativo da web serverless e API
{: #serverless-api-webapp}

Neste tutorial, você criará um aplicativo da web serverless hospedando conteúdo de website estático nas Páginas GitHub e implementando o back-end do aplicativo usando o {{site.data.keyword.openwhisk}}.

Como uma plataforma acionada por evento, o {{site.data.keyword.openwhisk_short}} suporta uma [variedade de casos de uso](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases). A construção de aplicativos da web e APIs é um deles. Com apps da web, os eventos são as interações entre os navegadores da web (ou clientes REST) e seu app da web, as solicitações de HTTP. Em vez de provisionar uma máquina virtual, um contêiner ou um tempo de execução do Cloud Foundry para implementar seu back-end, é possível implementar sua API de back-end com uma plataforma serverless. Essa pode ser uma boa solução para evitar pagar pelo tempo inativo e permitir a escala da plataforma, conforme necessário.

Qualquer ação (ou função) no {{site.data.keyword.openwhisk_short}} pode ser transformada em um terminal HTTP pronto para ser consumido pelos web clients. Quando ativado para a web, essas ações são chamadas de *ações da web*. Depois de ter ações da web, é possível montá-las em uma API com recursos completos com o API Gateway. O API Gateway é um componente do {{site.data.keyword.openwhisk_short}} para expor APIs. Ele vem com segurança, suporte ao OAuth, limitação de taxa, suporte ao domínio customizado.

## Objetivos

* Implementar um back-end serverless e um banco de dados
* Expor uma API de REST
* Hospedar um website estático
* Opcional: usar um domínio customizado para a API de REST

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

O aplicativo mostrado neste tutorial é um website de guestbook simples no qual os usuários podem postar mensagens.

<p style="text-align: center;">

   ![Arquitetura](./images/solution8/Architecture.png)
</p>

1. O usuário acessa o aplicativo hospedado em Páginas do GitHub.
2. O aplicativo da web chama uma API de back-end.
3. A API de back-end é definida no API Gateway.
4. O API Gateway encaminha a solicitação para o [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk).
5. As ações do {{site.data.keyword.openwhisk_short}} usam o [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) para armazenar e recuperar entradas do guestbook.

## Antes de Começar
{: #prereqs}

Este guia usa Páginas do GitHub para hospedar o website estático. Certifique-se de que você tenha uma conta pública do GitHub.

## Criar o banco de dados do Guestbook

Vamos começar criando um {{site.data.keyword.cloudant_short_notm}}. O {{site.data.keyword.cloudant_short_notm}} é uma camada de dados totalmente gerenciada projetada para aplicativos modernos da web e móveis que alavancam um esquema JSON flexível. O {{site.data.keyword.cloudant_short_notm}} é construído e compatível com o Apache CouchDB e acessível por meio de uma API HTTPS segura, que é escalada à medida que seu aplicativo cresce.

1. No Catálogo, selecione **Cloudant**.
2. Configure o nome do serviço como **guestbook-db**, selecione **Usar as credenciais anteriores e o IAM** como métodos de autenticação e clique em **Criar**.
3. De volta à lista de Recursos, clique na entrada ***guestbook-db** sob a coluna Nome. Nota: pode ser necessário esperar até que o serviço seja provisionado. 
4. Na tela de detalhes do serviço, clique em ***Ativar painel do Cloudant*** que será aberto em outra guia do navegador. Nota: talvez seja necessário efetuar login em sua instância do Cloudant. 
5. Clique em ***Criar banco de dados*** e crie um banco de dados denominado ***guestbook***.

  ![](images/solution8/Create_Database.png)

6. Volte à guia Detalhes do serviço, em **Credenciais de serviço**
   1. Crie **Nova credencial**, aceite os padrões e clique em **Incluir**.
   2. Clique em **Visualizar credenciais** em Ações. Precisaremos dessas credenciais posteriormente para permitir que as ações do Cloud Functions leiam/gravem em seu serviço Cloudant.

## Criar ações serverless

Nesta seção, você criará ações serverless (normalmente chamadas de Funções). O {{site.data.keyword.openwhisk}} (com base no Apache OpenWhisk) é uma plataforma Function-as-a-Service (FaaS) que executa funções em resposta a eventos recebidos e não custa nada quando não está em uso.

![](images/solution8/Functions.png)

### Sequência de ações para salvar a entrada do guestbook

Você criará uma **sequência** que é uma cadeia de ações em que a saída de uma ação age como uma entrada para a ação seguinte e assim por diante. A primeira sequência que você criará será usada para persistir uma mensagem guest. Fornecido um nome, um emailID e um comentário, a sequência será:
   * Crie um documento a ser persistido.
   * Armazene o documento no banco de dados {{site.data.keyword.cloudant_short_notm}}.

Inicie criando a primeira ação:

1. Alterne para **Functions** https://{DomainName}/openwhisk.
2. Na área de janela esquerda, clique em **Ações** e, em seguida, em **Criar**.
3. **Crie ação** com o nome `prepare-entry-for-save` e selecione **Node.js** como o Tempo de execução (nota: selecione a versão mais recente).
4. Substitua o código existente pelo fragmento de código abaixo:
   ```js
   /**
    * Prepare the guestbook entry to be persisted
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. **Salvar**

Em seguida, inclua a ação em uma sequência:

1. Clique em **Sequências de inclusão** e, em seguida, **Incluir na sequência**.
1. Para o nome de sequência, insira `save-guestbook-entry-sequence` e, em seguida, clique em **Criar e incluir**.

Finalmente, inclua uma segunda ação na sequência:

1. Clique em **save-guestbook-entry-sequence** e, em seguida, clique em **Incluir**.
1. Selecione **Usar público**, **Cloudant** e, em seguida, escolha **create-document** em **Ações**
1. Crie **Nova ligação** 
1. Para Nome, insira `binding-for-guestbook`
1. Para a instância do Cloudant, selecione `Input your own credentials` e preencha os campos a seguir com as informações de credenciais capturadas para seu serviço cloudant: Nome de usuário, Senha, Host e Banco de dados = `guestbook` e clique em **Incluir** e, em seguida, em **Salvar**.
   {: tip}
1. Para testar isso, clique em **Mudar entrada** e insira o JSON abaixo
    ```json
     {
      "name": "John Smith",
      "email": "john@smith.com",
      "comment": "this is my comment"
    }
    ```
    {: codeblock}
1. **Aplique** e, em seguida, **Chame**.
    ![](images/solution8/Save_Entry_Invoke.png)

### Sequência de ações para recuperar entradas

A segunda sequência é usada para recuperar as entradas existentes do guestbook. Essa sequência:
   * Listará todos os documentos do banco de dados.
   * Formatará os documentos e os retornará.

1. Em **Functions**, clique em **Ações** e, em seguida, em **Criar** uma nova ação do Node.js e nomeie-a como `set-read-input`.
2. Substitua o código existente pelo fragmento de código abaixo. Essa ação passa os parâmetros apropriados para a próxima ação.
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. **Salvar**

Inclua a ação em uma sequência:

1. Clique em **Sequências de inclusão**, **Incluir na sequência** e **Criar novo**
1. Insira `read-guestbook-entries-sequence` para o **Nome da ação** e clique em **Criar e incluir**.

Conclua a sequência:

1. Clique na sequência **read-guestbook-entries-sequence** e, em seguida, clique em **Incluir** para criar e incluir a segunda ação para obter documentos do Cloudant.
1. Em **Usar público**, escolha **{{site.data.keyword.cloudant_short_notm}}** e, em seguida, **list-documents**
1. Em **Minhas ligações**, escolha **binding-for-guestbook** e **Incluir** para criar e incluir essa ação pública em sua sequência.
1. Clique em **Incluir** novamente para criar e incluir a terceira ação que formatará os documentos do {{site.data.keyword.cloudant_short_notm}}.
1. Em **Criar novo**, insira `format-entries` para o nome e, em seguida, clique em **Criar e incluir**.
1. Clique em **format-entries** e substitua o código pelo abaixo:
   ```js
   const md5 = require('spark-md5');

   function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. **Salvar**
1. Escolha a sequência clicando em **Ações** e, em seguida, **read-guestbook-entries-sequence**.
1. Clique em **Salvar** e, em seguida, em **Chamar**. A saída deve ser semelhante à seguinte:
   ![](images/solution8/Read_Entries_Invoke.png)

## Criar uma API

![](images/solution8/Cloud_Functions_API.png)

1. Acesse Ações https://{DomainName}/openwhisk/actions.
2. Selecione a sequência **read-guestbook-entries-sequence**. Próximo ao nome, clique em **Ação da web**, marque **Ativar ação da web** e **Salvar**.
3. Faça o mesmo para a sequência **save-guestbook-entry-sequence**.
4. Acesse APIs https://{DomainName}/openwhisk/apimanagement e **Crie uma API do {{site.data.keyword.openwhisk_short}}**
5. Configure o nome como `guestbook` e o caminho base como `/guestbook`
6. Clique em **Criar operação** e crie uma operação para recuperar entradas do guestbook:
   1. Configure **caminho** para `/entries`
   2. Configure **verb** como `GET*`
   3. Selecione a ação **read-guestbook-entries-sequence**
7. Clique em **Criar operação** e crie uma operação para persistir uma entrada do guestbook:
   1. Configure **caminho** para `/entries`
   2. Configure **verb** como `PUT`
   3. Selecione a ação **save-guestbook-entry-sequence**
8. Salve e exponha a API.

## Implementar o app da web

1. Bifurque o repositório da interface com o usuário do Guestbook https://github.com/IBM-Cloud/serverless-guestbook para o seu GitHub público.
2. Modifique **docs/guestbook.js** e substitua o valor de **apiUrl** pela rota fornecida pelo API Gateway.
3. Confirme o arquivo modificado em seu repositório bifurcado.
4. Na página Configurações de seu repositório, role para **Páginas do GitHub**, mude a origem para a **pasta de ramificação/docs principal** e Salve.
5. Acesse a página pública para seu repositório.
6. Você deverá ver a entrada do guestbook "teste" criada anteriormente.
7. Inclua novas entradas.

![](images/solution8/Guestbook.png)

## Opcional: usar seu próprio domínio para a API

A criação de uma API gerenciada fornece um terminal padrão como `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`. Nesta seção, você configurará esse terminal para ser capaz de manipular solicitações vindas de seu subdomínio customizado.

### Obter um certificado para o domínio customizado

A exposição das ações do {{site.data.keyword.openwhisk_short}} por meio de um domínio customizado exigirá uma conexão HTTPS segura. Você deve obter um certificado SSL para o domínio e subdomínio que planeja usar com o back-end sem servidor. Supondo um domínio como *mydomain.com*, as ações podem ser hospedadas em *guestbook-api.mydomain.com*. O certificado precisará ser emitido para *guestbook-api.mydomain.com* (ou **.mydomain.com*).

É possível obter certificados SSL grátis em [Vamos criptografar](https://letsencrypt.org/). Durante o processo, pode ser necessário configurar um registro de DNS do tipo TXT em sua interface do DNS para provar que você é o proprietário do domínio.
{:tip}

Depois de ter obtido o certificado SSL e a chave privada para seu domínio, certifique-se de convertê-los para o formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Para converter um Certificado no formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. Para converter uma Chave privada no formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### Importar o certificado para um repositório central

1. Crie uma instância do [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) em uma localização suportada.
1. No painel de serviço, use **Importar certificado**:
   * Configure **Nome** para o subdomínio e domínio customizados, como `guestbook-api.mydomain.com`.
   * Procure o **Arquivo de certificado** no formato PEM.
   * Procure o **Arquivo de chave privado** no formato PEM.
   * **Importar**.

### Configure o domínio customizado para a API gerenciada

1. Acesse [APIs / Domínios customizados](https://{DomainName}/apis/domains).
1. No seletor de Região, selecione o local no qual você implementou as ações.
1. Localize o domínio customizado vinculado à organização e ao espaço no qual você criou as ações e a API gerenciada. Clique em **Mudar configurações** no menu Ação.
1. Anote o valor **Default domain / alias**.
1. Marque **Aplicar domínio customizado**
   1. Configure **Nome de domínio** para o domínio que será usado, como `guestbook-api.mydomain.com`.
   1. Selecione a instância {{site.data.keyword.cloudcerts_short}} que contém o certificado.
   1. Selecione o certificado para o domínio.
1. Acesse seu provedor DNS e crie um novo **Registro TXT do DNS** mapeando seu domínio para o domínio/alias padrão da API. O registro TXT do DNS poderá ser removido quando as configurações tiverem sido aplicadas.
1. Salve as configurações de domínio customizado. O diálogo verificará a existência do registro TXT do DNS.
1. Finalmente, retorne para as configurações do provedor DNS e crie um registro CNAME apontando seu domínio customizado (por exemplo, guestbook-api.mydomain.com) para o Domínio/Alias padrão. Isso fará com que o tráfego que passa por seu domínio customizado seja encaminhado para sua API de back-end.

Depois que as mudanças do DNS tiverem sido propagadas, você será capaz de acessar sua api do guestbook em https://guestbook-api.mydomain.com/guestbook.

1. Edite **docs/guestbook.js** e atualize o valor de **apiUrl** com https://guestbook-api.mydomain.com/guestbook
1. Confirme o arquivo modificado.
1. Seu aplicativo acessa agora a API por meio de seu domínio customizado.

## Remover recursos

* Exclua o serviço {{site.data.keyword.cloudant_short_notm}}
* Exclua a API do {{site.data.keyword.openwhisk_short}}
* Exclua as ações do {{site.data.keyword.openwhisk_short}}

## Conteúdo relacionado
* [Mais guias e amostras no serverless](https://developer.ibm.com/code/journey/category/serverless/)
* [Introdução ao {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [Casos de uso comum do {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [Criar APIs de REST por meio de ações](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
