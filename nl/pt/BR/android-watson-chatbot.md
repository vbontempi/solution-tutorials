---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# Construir um robô de bate-papo Android ativado por voz
{: #android-watson-chatbot}

Aprenda como é fácil criar rapidamente um robô de bate-papo nativo Android ativado por voz com os serviços {{site.data.keyword.conversationshort}}, {{site.data.keyword.texttospeechshort}} e {{site.data.keyword.speechtotextshort}} no {{site.data.keyword.Bluemix_short}}.

Este tutorial conduz você no processo de definição de intentos e entidades e construção de um fluxo de diálogo para que seu robô de bate-papo responda às consultas do cliente. Você aprenderá como ativar os serviços {{site.data.keyword.speechtotextshort}} e {{site.data.keyword.texttospeechshort}} para interação fácil com o app Android.
{:shortdesc}

## Objetivos
{: #objectives}

- Use o {{site.data.keyword.conversationshort}} para customizar e implementar um robô de bate-papo.
- Permita que os usuários finais interajam com o robô de bate-papo usando voz e áudio.
- Configure e execute o app Android.

## Serviços usados
{: #services}

Este tutorial usa os produtos a seguir:

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## Arquitetura
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* Os usuários interagem com um aplicativo móvel usando sua voz.
* O áudio é transcrito para texto com o {{site.data.keyword.speechtotextfull}}.
* O texto é passado para o {{site.data.keyword.conversationfull}}.
* A resposta do {{site.data.keyword.conversationfull}} é convertida em áudio pelo {{site.data.keyword.texttospeechfull}} e o resultado enviado de volta para o aplicativo móvel.

## Antes de Começar
{: #prereqs}

- Faça download e instale o [Android Studio](https://developer.android.com/studio/index.html).

## Criar serviços
{: #setup}

Nesta seção, você criará os serviços necessários para o tutorial iniciando com o {{site.data.keyword.conversationshort}} para construir assistentes virtuais cognitivos que ajudam seus clientes.

1. Acesse o [**Catálogo do {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e selecione Serviço [{{site.data.keyword.conversationshort}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation) > Plano **Lite**:
   1. Configure **Nome** como **android-chatbot-assistant**
   1. **Criar**.
2. Clique em **Credenciais de serviço** na área de janela esquerda e clique em **Nova credencial**.
   1. Configure **Nome** como **for-android-app**.
   1. **Incluir**.
3. Clique em **Visualizar credenciais** para ver as credenciais. Anote a **Chave de API** e a **URL**, elas serão necessárias para o aplicativo móvel.

O serviço {{site.data.keyword.speechtotextshort}} converte a voz humana na palavra escrita que pode ser enviada como uma entrada para o serviço {{site.data.keyword.conversationshort}} no {{site.data.keyword.Bluemix_short}}.

1. Acesse o [**Catálogo do {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e selecione Serviço [{{site.data.keyword.speechtotextshort}}](https://{DomainName}/catalog/services/speech-to-text) > Plano **Lite**.
   1. Configure **Nome** como **android-chatbot-stt**.
   1. **Criar**.
2. Clique em **Credenciais de serviço** na área de janela esquerda e clique em **Nova credencial** para incluir uma nova credencial.
   1. Configure **Nome** como **for-android-app**.
   1. **Incluir**.
3. Clique em **Visualizar credenciais** para ver as credenciais. Anote a **Chave de API** e a **URL**, elas serão necessárias para o aplicativo móvel.

O serviço {{site.data.keyword.texttospeechshort}} processa o texto e a língua natural para gerar a saída de áudio sintetizada completa com cadência e entonação apropriadas. O serviço fornece várias vozes e pode ser configurado no app Android.

1. Acesse o [**Catálogo do {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) e selecione Serviço [{{site.data.keyword.texttospeechshort}}](https://{DomainName}/catalog/services/text-to-speech) > Plano **Lite**.
   1. Configure **Nome** como **android-chatbot-tts**.
   1. **Criar**.
2. Clique em **Credenciais de serviço** na área de janela esquerda e clique em **Nova credencial** para incluir uma nova credencial.
   1. Configure **Nome** como **for-android-app**.
   1. **Incluir**.
3. Clique em **Visualizar credenciais** para ver as credenciais. Anote a **Chave de API** e a **URL**, elas serão necessárias para o aplicativo móvel.

## Criar uma qualificação
{: #create_workspace}

Uma habilidade é um contêiner para os artefatos que definem o fluxo de conversa.

Para este tutorial, você salvará e usará o arquivo [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) com os intentos, as entidades e o fluxo de diálogo predefinidos para sua máquina.

1. Na página de detalhes do serviço {{site.data.keyword.conversationshort}}, navegue para **Gerenciar** na área de janela esquerda, clique em **Ativar ferramenta** para ver o painel do {{site.data.keyword.conversationshort}}.
1. Clique na guia **Qualificações**.
1. **Criar novo**, em seguida, **Importar qualificação** e escolha o arquivo JSON transferido por download acima.
1. Selecione a opção **Tudo** e clique em **Importar**. Uma nova qualificação é criada com intentos, entidades e fluxo de diálogo predefinidos.
1. Volte para a lista de qualificações. Selecione o menu Ação na qualificação `Ana` para **Visualizar detalhes da API**.

### Definir um intento
{:#define_intent}

Um intento representa o propósito da entrada de um usuário, como responder a uma pergunta ou processar um pagamento de conta. Você define uma intenção para cada tipo de solicitação do usuário que você quer que seu aplicativo suporte. Reconhecendo o intento expresso na entrada de um usuário, o serviço {{site.data.keyword.conversationshort}} pode escolher o fluxo de diálogo correto para responder a ele. Na ferramenta, o nome de uma intenção é sempre prefixado com o caractere `#`.

Simplificando, os intentos são as intenções do usuário final. A seguir estão exemplos de nomes de intento.
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. Clique na qualificação recém-criada - **Ana**.

   Ana é um robô de seguros para que os usuários consultem seus benefícios de saúde e solicitações de arquivo.
   {:tip}
2. Clique na primeira guia para ver todos os **Intentos**.
3. Clique em **Incluir intento** para criar um novo intento. Insira `Cancel_Policy` como seu nome de intento após `#` e forneça uma descrição opcional.
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. Clique em **Criar intento**.
5. Inclua exemplos de usuário quando solicitado para cancelar uma política
   - `I want to cancel my policy`
   - `Drop my policy now`
   - `I wish to stop making payments on my policy.`
6. Inclua exemplos de usuário um após outro e clique em **incluir exemplo**. Repita isso para todos os outros exemplos de usuário.

   Lembre-se de incluir pelo menos 5 exemplos de usuário para treinar seu robô melhor.
   {:tip}

7. Clique no botão **fechar**![](images/solution28-watson-chatbot-android/close_icon.png) ao lado do nome do intento para salvá-lo.
8. Clique em **Catálogo de conteúdo** e selecione **Geral**. Clique em **Incluir na qualificação**.

   O catálogo de conteúdo o ajuda na introdução mais rápida, incluindo intentos existentes (financeiro, atendimento ao cliente, seguro, telco, e-commerce e muito mais). Esses intentos são treinados em perguntas comuns que os usuários podem fazer.
   {:tip}

### Definir uma entidade
{:#define_entity}

Uma entidade representa um termo ou objeto que é relevante para seus intentos e que fornece um contexto específico para um intento. Você lista os valores possíveis para cada entidade e os sinônimos que os usuários podem inserir. Ao reconhecer as entidades mencionadas na entrada do usuário, o serviço {{site.data.keyword.conversationshort}} pode escolher as ações específicas a serem tomadas para cumprir uma intenção. Na ferramenta, o nome de uma entidade é sempre prefixado com o caractere `@`.

A seguir estão exemplos de nomes de entidades
 - `@location`
 - `@menu_item`
 - `@product`

1. Clique na guia **Entidades** para ver as entidades existentes.
2. Clique em **Incluir entidade** e insira o nome da entidade como `location` após `@`. Clique em  ** Criar entidade **.
3. Insira `address` como o nome do valor e selecione **Synonyms**.
4. Inclua `place` como um sinônimo e clique no ícone ![](images/solution28-watson-chatbot-android/plus_icon.png). Repita com os sinônimos `office`, `centre`, `branch`, etc. e clique em **Incluir valor**.
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. Clique em **fechar**![](images/solution28-watson-chatbot-android/close_icon.png) para salvar as mudanças.
6. Clique na guia **Entidades do sistema** para verificar as entidades comuns criadas pela IBM que podem ser usadas em qualquer caso de uso.

   Entidades do Sistema podem ser usadas para reconhecer uma ampla variedade de valores para os tipos de objetos que eles representam. Por exemplo, a entidade do sistema `@sys-number` corresponde a qualquer valor numérico, incluindo números inteiros, frações decimais ou mesmo números gravados como palavras.
   {:tip}
7. Alterne o **Status** de off para `on` para entidades do sistema @sys-person e @sys-location.

### Construir o fluxo de diálogo
{:#build_dialog}

Um diálogo é um fluxo de conversa de ramificação que define como o seu aplicativo responde quando ele reconhece os intentos e entidades definidos. Você usa o construtor de diálogo na ferramenta para criar conversas com usuários, fornecendo respostas com base nas intenções e entidades que você reconhece na entrada deles.

1. Clique na guia **Diálogo** para ver o fluxo de diálogo existente com intentos e entidades.
2. Clique em **Incluir nó** para incluir um novo nó no diálogo.
3. Em **se o assistente reconhecer**, insira `#Cancel_Policy`.
4. Em **Em seguida, responda com**, insira a resposta `This facility is not available online. Please visit our nearest branch to cancel your policy.`
5. Clique em ![](images/solution28-watson-chatbot-android/save_node.png) para fechar e salvar o nó.
6. Role para ver o nó `#greeting`. Clique no nó para ver os detalhes.
   ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. Clique no ícone ![](images/solution28-watson-chatbot-android/add_condition.png) para **incluir uma nova condição**. Selecione `or` na lista suspensa e insira `#General_Greetings` como o intento. A seção **Em seguida, responda com** mostra a resposta do assistente quando cumprimentado pelo usuário.
   ![](images/solution28-watson-chatbot-android/apply_condition.png)

   Uma variável de contexto é uma variável que você define em um nó e, opcionalmente, para a qual especifica um valor padrão. Outros nós ou lógicas de aplicativo podem posteriormente configurar ou mudar o valor da variável de contexto. O aplicativo pode passar informações para o diálogo e o diálogo pode atualizar essas informações e passá-las de volta para o aplicativo ou para um nó subsequente. O diálogo faz isso usando variáveis de contexto.
   {:tip}

8. Teste o fluxo de diálogo usando o botão **Tentar**.

## Vincular a qualificação a um assistente

Um **assistente** é um robô cognitivo que pode ser customizado para suas necessidades de negócios e implementado em múltiplos canais para trazer ajuda para seus clientes onde e quando eles precisarem. Você customiza o assistente incluindo nele as **qualificações** que ele precisa para satisfazer os objetivos de seus clientes.

1. Na ferramenta {{site.data.keyword.conversationshort}}, alterne para **Assistentes** e use **Criar novo**.
   1. Configure **Nome** como **android-chatbot-assistant**
   1. ** Criar **
1. Use **Incluir qualificação de diálogo** para selecionar a qualificação criada nas seções anteriores.
   1. **Incluir qualificação existente**
   1. Selecione **Ana**
1. Selecione o menu Ação no Assistente > **Configurações** > **Detalhes da API**, anote o **ID do assistente**, você precisará referenciá-lo no aplicativo móvel (no arquivo `config.xml` do app Android).

## Configurar e executar o app Android
{:#configure_run_android_app}

O repositório contém o código do aplicativo Android com as dependências de gradle necessárias.

1. Execute o comando a seguir para clonar o [repositório GitHub](https://github.com/IBM-Cloud/chatbot-watson-android):
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. Ative o Android Studio > **Abra um projeto existente do Android Studio** e aponte para o código transferido por download. A construção do **Gradle** será acionada automaticamente e todas as dependências serão transferidas por download.
3. Abra `app/src/main/res/values/config.xml` para ver os itens temporários (`ASSISTANT_ID_HERE`) para as credenciais de serviço. Insira as credenciais de serviço (salvas anteriormente) em seus respectivos itens temporários e salve o arquivo.
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <!--Watson Assistant service credentials-->
       <!-- REPLACE `ASSISTANT_ID_HERE` with ID of the Assistant to use -->
       <string name="assistant_id">ASSISTANT_ID_HERE</string>

       <!-- REPLACE `ASSISTANT_API_KEY_HERE` with Watson Assistant service API Key-->
       <string name="assistant_apikey">ASSISTANT_API_KEY_HERE</string>

       <!-- REPLACE `ASSISTANT_URL_HERE` with Watson Assistant service URL-->
       <string name="assistant_url">ASSISTANT_URL_HERE</string>

       <!--Watson Speech To Text(STT) service credentials-->
       <!-- REPLACE `STT_API_KEY_HERE` with Watson Speech to Text service API Key-->
       <string name="STT_apikey">STT_API_KEY_HERE</string>

       <!-- REPLACE `STT_URL_HERE` with Watson Speech to Text service URL-->
       <string name="STT_url">STT_URL_HERE</string>

       <!--Watson Text To Speech(TTS) service credentials-->
       <!-- REPLACE `TTS_API_KEY_HERE` with Watson Text to Speech service API Key-->
       <string name="TTS_apikey">TTS_API_KEY_HERE</string>

       <!-- REPLACE `TTS_URL_HERE` with Watson Text to Speech service URL-->
       <string name="TTS_url">TTS_URL_HERE</string>
   </resources>
   ```
4. Construa o projeto e inicie o aplicativo em um dispositivo real ou com um simulador.
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. **Insira sua consulta** no espaço fornecido abaixo e clique no ícone de seta para enviar a consulta para o serviço {{site.data.keyword.conversationshort}}.
6. A resposta será passada para o serviço {{site.data.keyword.texttospeechshort}} e você deverá ouvir uma voz lendo a resposta.
7. Clique no ícone **mic** no canto inferior esquerdo do app para o discurso de entrada que é convertido em texto e, em seguida, pode ser enviado para o serviço {{site.data.keyword.conversationshort}} clicando no ícone de seta.


## Remover recursos
{:removeresources}

1. Navegue para [Lista de recursos,](https://{DomainName}/resources/)
1. Exclua os serviços que você criou:
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## Conteúdo relacionado
{:related}

- [Criando entidades, sinônimos, entidades do sistema](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [Variáveis de contexto](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [Construindo um diálogo complexo](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [Reunindo informações com intervalos](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [Opções de implementação                           ](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [Melhorar a sua qualificação](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
