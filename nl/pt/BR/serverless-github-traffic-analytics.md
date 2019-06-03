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

# Combinando o serverless e o Cloud Foundry para recuperação de dados e análise de dados
{: #serverless-github-traffic-analytics}
Neste tutorial, você cria um aplicativo para coletar estatísticas de tráfego do GitHub automaticamente para repositórios e fornece a base para a análise de dados de tráfego. O GitHub fornece somente acesso aos dados de tráfego para os últimos 14 dias. Se você desejar analisa as estatísticas por um período de tempo mais longo, é necessário fazer download e armazenar os dados sozinho. Neste tutorial, você implementa uma ação serverless para recuperar os dados de tráfego e armazená-los em um banco de dados SQL. Além disso, um app do Cloud Foundry é usado para gerenciar repositórios e fornecer acesso às estatísticas para análise de dados. O app e a ação serverless discutidos neste tutorial implementam uma solução pronta para diversos locatários com o conjunto de recursos inicial que suporta o modo de locatário único.

![](images/solution24-github-traffic-analytics/Architecture.png)

## Objetivos

* Implementar um app de banco de dados Python com suporte de diversos locatários e acesso seguro
* Integrar o ID do app como provedor de autenticação baseado no OpenID Connect
* Configurar a coleta serverless automatizada de estatísticas de tráfego do GitHub
* Integrar o {{site.data.keyword.dynamdashbemb_short}} para análise de dados de tráfego gráfico

## Produtos
Este tutorial usa os produtos a seguir:
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Antes de Começar
{: #prereqs}

Para concluir este tutorial, você precisa da versão mais recente da [CLI do IBM Cloud](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) e do [plug-in instalado](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) do {{site.data.keyword.openwhisk_short}}.

## Configuração de serviço e ambiente (shell)
Nesta seção, você configura os serviços necessários e prepara o ambiente. Tudo isso pode ser realizado no ambiente de shell.

1. Clone o [repositório GitHub](https://github.com/IBM-Cloud/github-traffic-stats) e navegue para o diretório clonado e seu subdiretório **backend**:
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. Use `ibmcloud login` para efetuar login interativamente no {{site.data.keyword.Bluemix_short}}. É possível reconfirmar os detalhes ao executar o comando `ibmcloud target`. É necessário ter uma organização e um espaço configurados.

3. Crie uma instância do {{site.data.keyword.dashdbshort}} com o plano **Entry** e nomeie-a como **ghstatsDB**:
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. Para acessar o serviço de banco de dados no {{site.data.keyword.openwhisk_short}} posteriormente, você precisa da autorização. Assim, você cria credenciais de serviço e rotula-as como **ghstatskey**:   
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. Crie uma instância do serviço {{site.data.keyword.appid_short}}. Use **ghstatsAppID** como o nome e o plano **Camada graduada**.
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
   Depois disso, crie um alias dessa nova instância de serviço no espaço do Cloud Foundry. Substitua **YOURSPACE** pelo espaço no qual você está implementando.
   ```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. Crie uma instância do serviço {{site.data.keyword.dynamdashbemb_short}} usando o plano **lite**.
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
   Novamente, crie um alias dessa nova instância de serviço e substitua **YOURSPACE**.
   ```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. No diretório **backend**, envie por push o aplicativo para o IBM Cloud. O comando usa uma rota aleatória para seu aplicativo.
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
   Aguarde até que a implementação seja concluída. Os arquivos de aplicativo são transferidos por upload, o ambiente de tempo de execução criado e os serviços ligados ao aplicativo. As informações de serviço são obtidas do arquivo `manifest.yml`. É necessário atualizar esse arquivo, se você usou outros nomes de serviço. Depois que o processo for concluído com êxito, o URI do aplicativo será exibido.

   O comando acima usa uma rota aleatória, mas exclusiva, para o aplicativo. Se você mesmo desejar escolher uma, inclua-a como parâmetro adicional no comando, por exemplo, `ibmcloud cf push your-app-name`. Também é possível editar o arquivo `manifest.yml`, mudar o **nome** e mudar **random-route** de **true** para **false**.
   {:tip}

## Configuração do ID do app e do GitHub (navegador)
As etapas a seguir são todas executadas usando seu navegador da Internet. Primeiro, você configura o {{site.data.keyword.appid_short}} para usar o Cloud Directory e para trabalhar com o app Python. Depois disso, você cria um token de acesso GitHub. Ele é necessário para que a função implementada recupere os dados de tráfego.

1. Na [{{site.data.keyword.Bluemix_short}} Lista de recursos](https://{DomainName}/resources), abra a visão geral de seus serviços. Localize a instância do serviço {{site.data.keyword.appid_short}} na seção **Serviços**. Clique em sua entrada para abrir os detalhes.
2. No painel de serviço, clique em **Gerenciar** em **Provedores de identidade** no menu no lado esquerdo. Ele traz uma lista de provedores de identidade disponíveis, como Facebook, Google, Federação SAML 2.0 e o Cloud Directory. Alterne o Cloud Directory para **Ativado**, todos os outros provedores para **Desativado**.
   
   Você pode desejar configurar as regras de senha [Multi-Factor Authentication (MFA)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) e avançada. Elas não são discutidas como parte deste tutorial.
   {:tip}

3. Na parte inferior dessa página está a lista de URLs de redirecionamento. Insira a **url** de seu aplicativo + /redirect_uri. Por exemplo, `https://github-traffic-stats-random-word.mybluemix.net/redirect_uri`.

   Para testar o app localmente, a URL de redirecionamento é `http://0.0.0.0:5000/redirect_uri`. É possível configurar múltiplas URLs de redirecionamento.
   {:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. No menu à esquerda, clique em **Usuários**. Ele abre a lista de usuários no Cloud Directory. Clique no botão **Incluir usuário** para incluir a si mesmo como o primeiro usuário. Agora você está pronto para configurar o serviço {{site.data.keyword.appid_short}}.
5. No navegador, visite [Github.com](https://github.com/settings/tokens) e acesse **Configurações -> Configurações do desenvolvedor-> Tokens de acesso pessoal**. Clique no botão **Gerar novo token**. Insira **Tutorial do GHStats** para a **Descrição do token**. Depois disso, ative **public_repo** na categoria **repo** e **read:org** em **admin:org**. Agora, na parte inferior dessa página, clique em **Gerar token**. O novo token de acesso é exibido na próxima página. Você precisará dele durante a configuração do aplicativo a seguir.
   ![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## Configurar e testar o app Python
Após a preparação, você configura e testa o app. O app é escrito em Python usando a microestrutura [Flask](http://flask.pocoo.org/) popular. Os repositórios podem ser incluídos e removidos da coleta de estatísticas. Os dados de tráfego podem ser acessados em uma visualização tabular.

1. Em um navegador, abra o URI do app implementado. Você deverá ver uma página de boas-vindas.
   ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. No navegador, inclua `/admin/initialize-app` para o URI e acesse a página. Ele é usado para inicializar o aplicativo e seus dados. Clique no botão **Iniciar a inicialização**. Isso levará você a uma página de configuração protegida por senha. O endereço de e-mail com o qual você efetua login é obtido como identificação para o administrador do sistema. Use o endereço de e-mail e a senha que você configurou anteriormente.

3. Na página de configuração, insira um nome (ele é usado para saudações), seu nome do usuário do GitHub e o token de acesso que você gerou antes. Clique em **Inicializar**. Isso cria as tabelas de banco de dados e insere alguns valores de configuração. Finalmente, ele cria registros de banco de dados para o administrador do sistema e um locatário.
   ![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. Uma vez feito, você é levado para a lista de repositórios gerenciados. Agora é possível incluir repositórios fornecendo o nome da conta ou da organização do GitHub e o nome do repositório. Depois de inserir os dados, clique em **Incluir repositório**. O repositório, juntamente com um identificador recém-designado, deve aparecer na tabela. É possível remover repositórios do sistema inserindo seu ID e clicando em **Excluir repositório**.
![](images/solution24-github-traffic-analytics/RepositoryList.png)

## Implementar o Cloud Function e o acionador
Com o app de gerenciamento no lugar, implemente uma ação, um acionador e uma regra para conectar os dois para o {{site.data.keyword.openwhisk_short}}. Esses objetos são usados para coletar automaticamente os dados de tráfego do GitHub no planejamento especificado. A ação se conecta ao banco de dados, itera sobre todos os locatários e seus repositórios e obtém os dados de visualização e de clonagem para cada repositório. Essas estatísticas são mescladas no banco de dados.

1. Mude para o diretório **functions**.
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. Crie uma nova ação **collectStats**. Ela usa um [Ambiente do Python 3](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments) que já inclui o driver de banco de dados necessário. O código-fonte para a ação é fornecido no arquivo `ghstats.zip`.
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   Se você modificar o código-fonte para a ação (`__main__.py`), será possível reempacotar o archive zip com `zip -r ghstats.zip  __main__.py github.py` novamente. Consulte o arquivo `setup.sh` para obter detalhes.
   {:tip}
3. Ligue a ação para o serviço de banco de dados. Use a instância e a chave de serviço que você criou durante a configuração do ambiente.
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. Crie um acionador com base no [pacote de alarmes](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm). Ele suporta diferentes formas de especificar o alarme. Use o estilo semelhante ao [cron](https://en.wikipedia.org/wiki/Cron). Iniciando em 21 de abril e terminando em 21 de dezembro, o acionador é disparado diariamente às 6h UTC. Certifique-se de ter uma data de início futura.
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  É possível mudar o acionador de um planejamento diário para semanal aplicando `"0 6 * * 0"`. Isso dispararia todos os domingos às 6h.
  {:tip}
5. Finalmente, você cria uma regra **myStatsRule** que conecta o acionador **myDaily** à ação **collectStats**. Agora, o acionador faz com que a ação seja executada no planejamento especificado na etapa anterior.
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. Chame a ação para uma execução de teste inicial. O **repoCount** retornado deve refletir o número de repositórios que você configurou anteriormente.
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
   A saída será semelhante a esta:
   ```
   {
       "repoCount": 18
   }
   ```
7. Em sua janela do navegador com a página do app, agora é possível visitar o tráfego do repositório. Por padrão, 10 entradas são exibidas. É possível mudá-lo para valores diferentes. Também é possível classificar as colunas da tabela ou usar a caixa de procura para filtrar repositórios específicos. É possível inserir uma data e um nome da organização e, em seguida, classificar por viewcount para listar as pontuações superiores para um dia específico.
   ![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## Conclusões
Neste tutorial, você implementou uma ação serverless e um acionador e regra relacionados. Eles permitem recuperar automaticamente os dados de tráfego para repositórios GitHub. As informações sobre esses repositórios, incluindo o token de acesso específico do locatário, são armazenadas em um banco de dados SQL ({{site.data.keyword.dashdbshort}}). Esse banco de dados é usado pelo app Cloud Foundry para gerenciar usuários, repositórios e para apresentar as estatísticas de tráfego no portal do app. Os usuários podem ver as estatísticas de tráfego em tabelas pesquisáveis ou visualizadas em um painel integrado (serviço {{site.data.keyword.dynamdashbemb_short}}, consulte a imagem abaixo). Também é possível fazer download da lista de repositórios e dos dados de tráfego como arquivos CSV.

O app Cloud Foundry gerencia o acesso por meio de um cliente OpenID Connect conectando-se ao {{site.data.keyword.appid_short}}.
![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## Limpeza
Para limpar os recursos usados para este tutorial, é possível excluir os serviços relacionados e o app, bem como a ação, o acionador e a regra na ordem inversa, conforme criado:

1. Exclua a regra, o acionador e a ação {{site.data.keyword.openwhisk_short}}.
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. Exclua o app Python e seus serviços.
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## Expandir o tutorial
Deseja incluir ou mudar este tutorial? Aqui estão algumas ideias:
* Expanda o app para suporte de diversos locatários.
* Integre um gráfico para os dados.
* Use provedores de identidade social.
* Inclua um selecionador de data na página de estatísticas para filtrar dados exibidos.
* Use uma página de login customizado para o {{site.data.keyword.appid_short}}.
* Explore os relacionamentos de codificação social entre os desenvolvedores usando o [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Conteúdo relacionado
Aqui estão os links para informações adicionais sobre os tópicos abordados neste tutorial.

Documentação e SDKs:
* [{{site.data.keyword.openwhisk_short}} documentação](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Documentação: [IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.appid_short}} documentação](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [Tempo de execução do Python no IBM Cloud](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
