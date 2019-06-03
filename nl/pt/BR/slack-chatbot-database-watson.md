---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Construir um Slackbot acionado por banco de dados
{: #slack-chatbot-database-watson}

Neste tutorial, você construirá um Slackbot para criar e procurar entradas de banco de dados Db2 para eventos e conferências. O Slackbot é suportado pelo serviço {{site.data.keyword.conversationfull}}. Você integrará o Slack e o {{site.data.keyword.conversationfull}} usando uma integração do Assistente.

A integração do Slack canaliza as mensagens entre o Slack e o {{site.data.keyword.conversationshort}}. Lá, algumas ações de diálogo do lado do servidor executam consultas SQL com relação a um banco de dados Db2. Todo o código (mas não muito) é escrito em Node.js.

## Objetivos
{: #objectives}

* Conectar o {{site.data.keyword.conversationfull}} ao Slack usando uma integração
* Criar, implementar e ligar ações do Node.js no {{site.data.keyword.openwhisk_short}}
* Acessar um banco de dados Db2 por meio do {{site.data.keyword.openwhisk_short}} usando Node.js

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}} ](https://{DomainName}/catalog/services/db2-warehouse) ou [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

  ![Arquitetura](images/solution19/SlackbotArchitecture.png)
</p>

## Antes de Começar
{: #prereqs}

Para concluir este tutorial, você precisa da versão mais recente da [CLI do {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) do [plug-in instalado](/docs/cli?topic=cloud-cli-plug-ins) do {{site.data.keyword.openwhisk_short}}.


## Configuração de serviço e ambiente
Nesta seção, você configurará os serviços necessários e preparará o ambiente. A maioria deles pode ser realizada na interface da linha de comandos (CLI) usando scripts. Eles estão disponíveis no GitHub.

1. Clone o [repositório GitHub](https://github.com/IBM-Cloud/slack-chatbot-database-watson) e navegue para o diretório clonado:
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. Se você não estiver com login efetuado, use `login ibmcloud` para efetuar login interativamente.
3. Destine a organização e o espaço nos quais criar o serviço de banco de dados com:
   ```
   ibmcloud target --cf
   ```
4. Crie uma instância do {{site.data.keyword.dashdbshort}} e nomeie-a como **eventDB**:
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
   Também é possível usar outro plano diferente de **Entrada**.
5. Para acessar o serviço de banco de dados no {{site.data.keyword.openwhisk_short}} posteriormente, você precisa da autorização. Assim, você cria credenciais de serviço e rotula-as como **slackbotkey**:   
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. Crie uma instância do serviço {{site.data.keyword.conversationshort}}. Use **eventConversation** como nome e o plano Lite grátis.
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. Em seguida, você registrará ações para o {{site.data.keyword.openwhisk_short}} e ligará as credenciais de serviço a essas ações. Algumas das ações são ativadas como ações da web e um segredo é configurado para evitar chamadas desautorizadas. Escolha um segredo e passe-o como um parâmetro, substitua **YOURSECRET** adequadamente.

   Uma das ações é chamada para criar uma tabela no {{site.data.keyword.dashdbshort}}. Ao usar uma ação do {{site.data.keyword.openwhisk_short}}, você não precisa de um driver Db2 local nem precisa usar a interface baseada no navegador para criar manualmente a tabela. Para executar o registro e a configuração, execute a linha abaixo e isso executará o arquivo **setup.sh** que contém todas as ações. Se o seu sistema não suportar comandos shell, copie cada linha do arquivo **setup.sh** e execute-a individualmente.

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **Nota:** por padrão, o script também insere poucas linhas de dados de amostra. É possível desativá-lo comentando a linha a seguir no script acima: `#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r`
8. Extraia as informações de namespace para as ações implementadas.

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   Anote a parte antes de **/slackdemo/eventInsert**. Ela é a organização e o espaço codificados. Você precisará dela na próxima seção.

## Carregar a qualificação/área de trabalho
Nesta parte do tutorial, você carregará uma área de trabalho ou qualificação predefinida no serviço {{site.data.keyword.conversationshort}}.
1. Na [{{site.data.keyword.Bluemix_short}} Lista de recursos](https://{DomainName}/resources), abra a visão geral de seus serviços. Localize a instância do serviço {{site.data.keyword.conversationshort}} criado na seção anterior. Clique em sua entrada e, em seguida, no alias do serviço para abrir os detalhes do serviço.
2. Clique em **Ativar ferramenta** para chegar ao {{site.data.keyword.conversationshort}} Tool.
3. Alterne para **Qualificações**, em seguida, clique em **Criar qualificação** e, em seguida, em **Importar qualificação**.
4. No diálogo, depois de clicar em **Escolher arquivo JSON**, selecione o arquivo **assistant-skill.json** no diretório local. Deixe a opção de importação em **Tudo (intentos, entidades e diálogo)** e, em seguida, clique em **Importar**. Isso cria uma nova qualificação denominada **TutorialSlackbot**.
5. Clique em **Diálogo** para ver os nós de diálogo. É possível expandi-los para ver uma estrutura como a mostrada abaixo.

   O diálogo tem nós para manipular perguntas para ajuda e um simples "Obrigado". O nó **newEvent** e seu filho reúnem a entrada necessária e, em seguida, chamam uma ação para inserir um novo registro de evento no Db2.

   O nó **eventos de consulta** esclarece se os eventos são procurados por seu identificador ou por data. A procura real e a coleta dos dados necessários são, então, executadas pelos nós-filhos **eventos de consulta por nome abreviado** e **evento de consulta por datas**.

   O **credential_node** configura o segredo para as ações de diálogo e as informações sobre a organização do Cloud Foundry. Esse último é necessário para chamar as ações.

  Detalhes serão explicados posteriormente abaixo assim que tudo estiver configurado.
   ![](images/solution19/SlackBot_Dialog.png)   
6. Clique no nó de diálogo **credential_node**, abra o editor JSON clicando no ícone de menu à direita de **Em seguida, responda com**.

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   Substitua **org_space** pelas informações de organização e espaço codificados que você recuperou anteriormente. Substitua qualquer "@" por "%40". Em seguida, mude **YOURSECRET** para seu segredo real de antes. Feche o editor JSON clicando no ícone novamente.

## Criar um assistente e integrar-se ao Slack

Agora, você criará um assistente associado à qualificação de antes e o integrará ao Slack. 
1. Clique em **Qualificações** na parte superior esquerda, em seguida, selecione em **Assistentes**. Em seguida, clique em **Criar assistente**.
2. No diálogo, preencha **TutorialAssistant** como nome, em seguida, clique em **Criar assistente**. Na próxima tela, escolha **Incluir qualificação de diálogo**. Depois disso, escolha **Incluir qualificação existente**, escolha **TutorialSlackbot** na lista e inclua-o.
3. Depois de incluir a qualificação, clique em **Incluir integração**, em seguida, na lista de **Integrações gerenciadas**, selecione **Slack**.
4. Siga as instruções para integrar o seu robô de bate-papo ao Slack.

## Testar o Slackbot e aprender
Abra sua área de trabalho do Slack para obter uma unidade de teste do robô de bate-papo. Inicie um bate-papo direto com o robô.

1. Digite **help** no formulário do sistema de mensagens. O robô deve responder com alguma orientação.
2. Agora, insira **novo evento** para começar a reunir dados para um novo registro de eventos. Você usará intervalos do {{site.data.keyword.conversationshort}} para coletar todas as entradas necessárias.
3. Primeiro, é o identificador ou o nome do evento. As aspas são necessárias. Elas permitem inserir nomes mais complexos. Insira **"Reunião informal: IBM Cloud"** como o nome do evento. O nome do evento é definido como uma entidade baseada em padrão **eventName**. Ele suporta diferentes tipos de aspas duplas no início e no final.
4. Em seguida, é o local do evento. A entrada é baseada na [entidade do sistema **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details). Como uma limitação, somente as cidades reconhecidas pelo {{site.data.keyword.conversationshort}} podem ser usadas. Tente **Friedrichshafen** como uma cidade.
5. As informações de contato, como um endereço de e-mail ou URI para um website, serão solicitadas na próxima etapa. Inicie com **https://www.ibm.com/events**. Você usará uma entidade baseada em padrão para esse campo.
6. As próximas perguntas estão reunindo data e hora para o início e o término. O **sys-date** e o **sys-time** são usados que permitem formatos de entrada diferentes. Use **próxima quinta-feira** como a data de início, **6 h** para o horário, use a data exata da próxima quinta-feira, por exemplo, **09-05-2019** e **22h** para a data e hora de encerramento.
7. Por último, com todos os dados coletados, um resumo é impresso e uma ação do servidor, implementada como ação do {{site.data.keyword.openwhisk_short}}, é chamada para inserir um novo registro no Db2. Depois disso, o diálogo alterna para um nó-filho para limpar o ambiente de processamento removendo as variáveis de contexto. O processo de entrada inteiro pode ser cancelado a qualquer momento inserindo **cancelar**, **saída** ou semelhante. Nesse caso, a opção do usuário é reconhecida e o ambiente limpo.
   ![](images/solution19/SlackSampleChat.png)   

Com alguns dados de amostra, é hora de procurar.
1. Digite **mostrar informações de evento**. Em seguida, é uma pergunta se a procura deve ser pelo identificador ou por data. Insira um **nome** e para a próxima pergunta **"Think 2019"**. Agora, o robô de bate-papo deve exibir informações sobre esse evento. O diálogo tem múltiplas respostas para escolha.
2. Com o {{site.data.keyword.conversationshort}} como um back-end, é possível inserir frases mais complexas e, assim, ignorar partes do diálogo. Use **mostrar evento pelo nome "Think 2019"** como entrada. O robô de bate-papo retorna diretamente o registro de evento.
3. Agora você procurará por data. Uma procura é definida por um par de datas, a data de início do evento precisa estar entre elas. Com a **conferência de procura por data em fevereiro de 2019** como entrada, o resultado deve ser o evento **Think 2019** novamente. A entidade **Fevereiro** é interpretada como duas datas, 1º de fevereiro e 28 de fevereiro, fornecendo, assim, entrada para o início e o término do intervalo de data. [Se 2019 não fosse especificado, um fevereiro no futuro seria identificado](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time). 

Após mais algumas procuras e novas entradas de eventos, é possível revisitar o histórico do bate-papo e melhorar o diálogo futuro. Siga as instruções na documentação do [{{site.data.keyword.conversationshort}} em **Melhorando o entendimento**](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro).


## Remover recursos
{:removeresources}

A execução do script de limpeza no diretório principal exclui a tabela de eventos do {{site.data.keyword.dashdbshort}} e remove as ações do {{site.data.keyword.openwhisk_short}}. Isso pode ser útil quando você começa a modificar e ampliar o código. O script de limpeza não muda a área de trabalho ou a qualificação do {{site.data.keyword.conversationshort}}.   
```bash
sh cleanup.sh
```
{:codeblock}

Na [{{site.data.keyword.Bluemix_short}} Lista de recursos](https://{DomainName}/resources), abra a visão geral de seus serviços. Localize a instância do serviço {{site.data.keyword.conversationshort}}, em seguida, exclua-a.

## Expandir o tutorial
Deseja incluir ou mudar este tutorial? Aqui estão algumas ideias:
1. Inclua recursos de procura em, por exemplo, procura de caracteres curinga ou procura de durações de eventos ("dê-me todos os eventos maiores que 8 horas").
2. Use {{site.data.keyword.databases-for-postgresql}} em vez de {{site.data.keyword.dashdbshort}}. O [tutorial repositório GitHub para este Slackbot](https://github.com/IBM-Cloud/slack-chatbot-database-watson) já tem código para suportar o {{site.data.keyword.databases-for-postgresql}}.
3. Inclua um serviço de clima e recupere dados de previsão para a data e o local do evento.
4. Exporte os dados do evento como o arquivo **.ics** do iCalendar.
5. Conecte o robô de bate-papo ao Facebook Messenger incluindo outra integração.
6. Inclua elementos interativos, como botões, na saída.      


## Conteúdo relacionado
{:related}

Aqui estão os links para informações adicionais sobre os tópicos abordados neste tutorial.

Postagens do blog relacionadas ao robô de bate-papo:
* [Robôs de bate-papo: alguns truques com intervalos no IBM Watson Conversation](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [Robôs de bate-papo dinâmicos: melhores práticas](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [Construindo robô de bate-papo: mais dicas e sugestões](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

Documentação e SDKs:
* Repositório GitHub com [dicas e truques para manipular variáveis no IBM Watson Conversation](https://github.com/IBM-Cloud/watson-conversation-variables)
* [{{site.data.keyword.openwhisk_short}} documentação](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Documentação: [IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Db2 Developer Community Edition grátis](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) para desenvolvedores
* Documentação: [Descrição da API do driver ibm_db Node.js](https://github.com/ibmdb/node-ibm_db)
* [{{site.data.keyword.cloudantfull}} documentação](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
