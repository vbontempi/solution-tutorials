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


# Aplicativo da web moderno usando a pilha MEAN
{: #mean-stack}

Este tutorial conduz você na criação de um aplicativo da web usando a pilha MEAN popular. Ela é composta de um DB **M**ongo, estrutura da web **E**xpress, estrutura de front-end **A**ngular e um tempo de execução do Node.js. Você aprenderá como executar um iniciador de MEAN localmente, criar e usar um banco de dados como um serviço (DBasS) gerenciado, implementar o app no {{site.data.keyword.cloud_notm}} e monitorar o aplicativo.  

## Objetivos

{: #objectives}

- Criar e executar um app Node.js iniciador localmente.
- Criar um banco de dados como um serviço (DBasS) gerenciado.
- Implementar o app Node.js na nuvem.
- Escalar recursos do MongoDB.
- Aprenda como monitorar o desempenho do aplicativo.

## Serviços usados

{: #products}

Este tutorial usa os serviços do {{site.data.keyword.Bluemix_notm}} a seguir:

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**Atenção:** este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura

{:#architecture}

<p style="text-align: center;">

![Diagrama de arquitetura](images/solution7/Architecture.png)</p>

1. O usuário acessa o aplicativo usando um navegador da web.
2. O app Node.js acessa o banco de dados {{site.data.keyword.composeForMongoDB}} para buscar dados.

## Antes de Começar

{: #prereqs}

1. [Instalar Git](https://git-scm.com/)
2. [Instale a CLI do {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)


E para desenvolver e executar o aplicativo localmente:
1. [Instale o Node.js e o NPM](https://nodejs.org/)
2. [Instale e execute o MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/)

## Executar o app MEAN localmente

{: #runapplocally}

Nesta seção, você executará um banco de dados MongoDB local, clonará um código de amostra do MEAN e executará o aplicativo localmente para usar o banco de dados MongoDB local.

{: shortdesc}

1. Siga as instruções [aqui](https://docs.mongodb.com/manual/administration/install-community/) para instalar e executar o banco de dados MongoDB localmente. Após a instalação ser concluída, use o comando abaixo para confirmar se o servidor **mongod** está em execução. Confirme se seu banco de dados está em execução com o comando a seguir.
  ```sh
  mongo
  ```
  {: codeblock}

2. Clone o código de início do MEAN.

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. Instale os pacotes necessários.

  ```sh
  npm install
  ```
  {: codeblock}

4. Copie o arquivo .env.example para .env. Edite as informações necessárias, no mínimo inclua seu próprio SESSION_SECRET.

5. Execute o nó server.js para iniciar seu app
  ```sh
  node server.js
  ```
  {: codeblock}

6. Acesse seu aplicativo, crie um novo usuário e efetue login

## Criar instância do banco de dados MongoDB na nuvem

{: #createdatabase}

Nesta seção, você criará um banco de dados {{site.data.keyword.composeForMongoDB}} na nuvem. O {{site.data.keyword.composeForMongoDB}} é o banco de dados como um serviço que é normalmente mais fácil de configurar e fornece backups e ajuste de escala integrados. É possível localizar muitos tipos diferentes de bancos de dados no [Catálogo de nuvem da IBM](https://{DomainName}/catalog/?category=data).  Para criar o {{site.data.keyword.composeForMongoDB}}, siga as etapas abaixo.

{: shortdesc}

1. Efetue login em sua conta do {{site.data.keyword.cloud_notm}} por meio da linha de comandos e destine sua conta do {{site.data.keyword.cloud_notm}}. 

  ```sh
  ibmcloud login ibmcloud target --cf
  ```
  {: codeblock}

  É possível localizar mais comandos da CLI [aqui.](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

2. Crie uma instância do {{site.data.keyword.composeForMongoDB}}. Isso também pode ser feito usando a [IU do console](https://{DomainName}/catalog/services/compose-for-mongodb). O nome do serviço deve ser denominado **mean-starter-mongodb**, pois o aplicativo está configurado para procurar esse serviço por esse nome.

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## Implementar o app na nuvem

{: #deployapp}

Nesta seção, você implementará o app node.js no {{site.data.keyword.cloud_notm}} que usou o banco de dados MongoDB gerenciado. O código-fonte contém um arquivo [**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml) que foi configurado para usar o serviço "mongodb" criado anteriormente. O aplicativo usa a variável de ambiente VCAP_SERVICES para acessar as credenciais do banco de dados Compose MongoDB. Isso pode ser visualizado no [arquivo server.js](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js). 

{: shortdesc}

1. Envie por push o código para a nuvem.

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. Depois que o código tiver sido enviado por push, você deverá ser capaz de visualizar o app em seu navegador. Um nome de host aleatório foi gerado que pode ser semelhante a: `https://mean-random-name.mybluemix.net`. É possível obter a URL do aplicativo por meio do painel do console ou da linha de comandos.![App em tempo real](images/solution7/live-app.png)


## Escalando recursos do banco de dados MongoDB
{: #scaledatabase}

Se o seu serviço precisa de armazenamento adicional, ou você deseja reduzir a quantia de armazenamento alocado para seu serviço, é possível fazer isso escalando recursos.

{: shortdesc}

1. Usando o **painel** do console, acesse a seção **conexões** e clique no banco de dados **Instância do MongoDB**.
2. No painel **detalhes da implementação**, clique na opção **escalar recursos**.
   ![](images/solution7/mongodb-scale-show.png)
3. Ajuste a **régua de controle** para aumentar ou diminuir o armazenamento alocado para o seu serviço de banco de dados {{site.data.keyword.composeForMongoDB}}.
4. Clique em **Escalar implementação** para acionar o novo ajuste de escala e retorne para a visão geral do painel. Uma mensagem 'Ajuste de escala iniciado' aparecerá na parte superior da página para permitir que você saiba que a nova escalação está em andamento.
   ![](images/solution7/scaling-in-progress.png)


## Monitorar desempenho do aplicativo
{: #monitorapplication}

Para verificar o funcionamento de seu aplicativo, é possível usar o serviço Availability Monitoring integrado. O serviço Availability Monitoring é conectado automaticamente a seus aplicativos na nuvem. O serviço Availability Monitoring executa testes sintéticos de locais em todo o mundo, ininterruptamente, para detectar e corrigir proativamente problemas de desempenho antes de afetar seus visitantes. Siga as etapas abaixo para chegar ao painel de monitoramento.

{: shortdesc}

1. Usando o painel do console, em seu aplicativo, selecione a guia **Monitoramento**.
2. Clique em **Visualizar todos os testes** para visualizar os testes.
   ![](images/solution7/alert_frequency.png)


## Conteúdo relacionado

{: #related}

- Configure o controle de fonte e a [entrega contínua](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops).
- Proteja o aplicativo da web em [múltiplos locais](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp).
- Crie, assegure e gerencie [APIs de REST](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis).
