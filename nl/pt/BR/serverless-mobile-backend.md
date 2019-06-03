---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: data-hd-operatingsystem="ios"}
{:android: data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Aplicativo móvel com um back-end serverless
{: #serverless-mobile-backend}

Neste tutorial, você aprenderá como usar o {{site.data.keyword.openwhisk}} juntamente com os serviços Cognitivos e de Dados para construir um back-end serverless para um aplicativo móvel.
{:shortdesc}

Nem todos os desenvolvedores móveis têm experiência de gerenciamento de lógica do lado do servidor ou de um servidor para iniciar. Eles prefeririam focar seus esforços no app que estão construindo. Agora, e se eles pudessem reutilizar suas qualificações de desenvolvimento existentes para gravar seu back-end móvel?

O {{site.data.keyword.openwhisk_short}} é uma plataforma acionada por evento serverless. Como [destacado neste exemplo](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp), as ações que você implementa podem ser facilmente convertidas em terminais HTTP como *ações da web* para construir uma API de back-end de aplicativo da web. Um aplicativo da web sendo um cliente para a API de REST, é fácil levar esse exemplo um passo adiante e aplicar a mesma abordagem para construir um back-end para um app móvel. E com o {{site.data.keyword.openwhisk_short}}, os desenvolvedores móveis podem gravar as ações na mesma linguagem usada para seu app móvel, Java para Android e Swift para iOS.

Este tutorial é configurável com base em sua plataforma de destino. Você está visualizando atualmente a documentação para a versão **iOS/Swift** deste tutorial. Use a guia na parte superior desta documentação para selecionar a versão **Android/Java** deste tutorial.
{: swift}

Este tutorial é configurável com base em sua plataforma de destino. Você está visualizando atualmente a documentação para a versão **Android/Java** deste tutorial. Use a guia na parte superior desta documentação para selecionar a versão **iOS/Swift** deste tutorial.
{: java}

## Objetivos
{: #objectives}

* Implementar um back-end móvel serverless com o {{site.data.keyword.openwhisk_short}}.
* Incluir a autenticação do usuário em um app móvel com o {{site.data.keyword.appid_short}}.
* Analisar o feedback do usuário com o {{site.data.keyword.toneanalyzershort}}.
* Enviar notificações com o {{site.data.keyword.mobilepushshort}}.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

O aplicativo mostrado neste tutorial é um app de feedback que analisa de forma inteligente o tom do texto de feedback e reconhece apropriadamente o cliente por meio de um {{site.data.keyword.mobilepushshort}}.

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. O usuário é autenticado com relação ao [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID). O {{site.data.keyword.appid_short}} fornece tokens de acesso e de identificação.
2. As chamadas adicionais para a API de back-end incluem o token de acesso.
3. O backend é implementado com o  [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk). As ações serverless, expostas como Ações da web, esperam que o token seja enviado nos cabeçalhos da solicitação e verifiquem sua validade (assinatura e data de expiração) antes de permitir acesso à API real.
4. Quando o usuário envia um feedback, o feedback é armazenado no [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
5. O texto de feedback é processado com o  [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer).
6. Com base no resultado da análise, uma notificação é enviada de volta para o usuário com o [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush).
7. O usuário recebe a notificação no dispositivo.

## Antes de Começar
{: #prereqs}

Este tutorial usa a ferramenta de linha de comandos do {{site.data.keyword.Bluemix_notm}} para provisionar recursos e implementar o código. Certifique-se de instalar a ferramenta de linha de comandos `ibmcloud`.

* [{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Script para instalar a CLI ibmcloud e os plug-ins necessários (Cloud Foundry e {{site.data.keyword.openwhisk_short}})

Além disso, você precisará do software e das contas a seguir:

   1. Java 8
   2. Android Studio 2.3.3
   3. Conta do Google Developer para configurar o Firebase Cloud Messaging
   4. Shell bash, cURL
   {: java}


   1. Xcode
   2. Conta do Apple Developer para configurar o Apple Push Notification Service
   3. Shell bash, cURL
   {: swift}

Neste tutorial, você configurará notificações push para o aplicativo. O tutorial supõe que o tutorial básico do {{site.data.keyword.mobilepushshort}} para [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) ou [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics) foi concluído e você está familiarizado com a configuração do Firebase Cloud Messaging ou do Apple Push Notification Service.
{:tip}

Para que os usuários do Windows 10 trabalhem com as instruções da linha de comandos, recomendamos a instalação do Windows Subsystem for Linux e Ubuntu, conforme descrito [neste artigo](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10).
{: tip}
{: java}

## Obter o código do aplicativo

O repositório contém tanto o aplicativo móvel quanto o código de ações do {{site.data.keyword.openwhisk_short}}.

1. Efetue check-out do código no repositório GitHub

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-android
   ```
   {: pre}
   {: java}

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-ios
   ```
   {: pre}
   {: swift}

2. Revise a estrutura do código

| Arquivo                                     | Descrição                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions) | Código para as ações do {{site.data.keyword.openwhisk_short}} do back end móvel sem servidor |
| [**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android) | Código para o aplicativo móvel          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh) | Script auxiliar para instalar, desinstalar, atualizar o acionador {{site.data.keyword.openwhisk_short}}, ações, regras |
{: java}

| Arquivo                                     | Descrição                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions) | Código para as ações do {{site.data.keyword.openwhisk_short}} do back end móvel sem servidor |
| [**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp) | Código para o aplicativo móvel          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh) | Script auxiliar para instalar, desinstalar, atualizar o acionador {{site.data.keyword.openwhisk_short}}, ações, regras |
{: swift}

## Provisionar serviços para manipular a autenticação do usuário, a persistência de feedback e a análise
{: #provision_services}

Nesta seção, você provisionará os serviços usados pelo aplicativo. É possível escolher provisionar os serviços do catálogo do {{site.data.keyword.Bluemix_notm}} ou usando a linha de comandos `ibmcloud`.

É recomendado criar um novo espaço para provisionar os serviços e implementar o back-end serverless. Isso ajuda a manter todos os recursos juntos.

### Provisionar serviços do catálogo do {{site.data.keyword.Bluemix_notm}}

1. Acesse o [catálogo do {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/catalog/)
2. Crie um serviço [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) com o plano **Lite**. Configure o nome como **serverlessfollowup-db**.
3. Crie um serviço [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) com o plano **Padrão**. Configure o nome como **serverlessfollowup-tone**.
4. Crie um serviço [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) com o plano **Camada graduada**. Configure o nome como **serverlessfollowup-appid**.
5. Crie um serviço [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) com o plano **Lite**. Configure o nome como **serverlessfollowup-mobilepush**.

### Provisionar serviços na linha de comandos

Com a linha de comandos, execute os comandos a seguir para provisionar os serviços e recuperar suas credenciais:

   ```sh
   ibmcloud cf create-service cloudantNoSQLDB Lite serverlessfollowup-db
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service tone_analyzer standard serverlessfollowup-tone
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service AppID "Graduated tier" serverlessfollowup-appid
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service imfpush lite serverlessfollowup-mobilepush
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

## Configurar o {{site.data.keyword.mobilepushshort}}
{: #push_notifications}

Quando um usuário enviar um novo feedback, o aplicativo analisará esse feedback e enviará de volta uma notificação para o usuário. O usuário pode ter movido para outra tarefa ou pode não ter o app móvel iniciado, portanto, usar notificações push é uma boa maneira de se comunicar com o usuário. O serviço {{site.data.keyword.mobilepushshort}} torna possível enviar notificações para usuários do iOS ou Android por meio de uma API unificada. Nesta seção, você configurará o serviço {{site.data.keyword.mobilepushshort}} para sua plataforma de destino.

### Configurar o Firebase Cloud Messaging (FCM)
{: java}

   1. No [Console do Firebase](https://console.firebase.google.com), crie um novo projeto. Configure o nome como **serverlessfollowup**
   2. Navegue para as **Configurações** do projeto
   3. Na guia **Geral**, inclua dois aplicativos:
      1. um com o nome de pacote configurado para: **com.ibm.mobilefirstplatform.clientsdk.android.push**
      2. e um com o nome do pacote configurado como: **serverlessfollowup.app**
   4. Faça download do `google-services.json` contendo os dois aplicativos definidos no console do Firebase e coloque esse arquivo na pasta `android/app` do diretório de check-out.
   5. Localize o ID do emissor e a Chave do servidor (também chamada de Chave de API posteriormente) na guia **Sistema de mensagens em nuvem**.
   6. No painel do serviço Push Notifications, configure o valor do ID do emissor e da Chave de API.
   {: java}

### Configurar o Apple Push Notifications Service (APNs)
{: swift}

1. Acesse o portal do [Apple Developer](https://developer.apple.com/) e registre um ID do app.
2. Crie um certificado SSL de APNs de desenvolvimento e
distribuição.
3. Crie um perfil de
fornecimento de desenvolvimento.
4. Configure a instância de serviço do {{site.data.keyword.mobilepushshort}} no {{site.data.keyword.Bluemix_notm}}. Consulte [Obter credenciais do APNs e configurar o serviço {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-) para obter as etapas detalhadas.
{: swift}

## Implementar um back-end serverless
{: #serverless_backend}

Com todos os serviços configurados, agora é possível implementar o back-end serverless. Os artefatos do {{site.data.keyword.openwhisk_short}} a seguir serão criados nesta seção:

| Artefato                      | Tipo                           | Descrição                              |
| ----------------------------- | ------------------------------ | ---------------------------------------- |
| `serverlessfollowup`          | Pacote                         | Um pacote para agrupar as ações e manter todas as credenciais de serviço |
| `serverlessfollowup-cloudant` | Ligação de pacote                | Ligado ao pacote integrado do {{site.data.keyword.cloudant_short_notm}}   |
| `serverlessfollowup-push`     | Ligação de pacote                | Ligado ao pacote do {{site.data.keyword.mobilepushshort}}  |
| `auth-validate`               | Ação                         | Valida os tokens de acesso e de identificação |
| `users-add`                   | Ação                         | Persiste as informações do usuário (ID, nome, e-mail, figura, ID do dispositivo) |
| `users-prepare-notify`        | Ação                         | Formata uma mensagem a ser usada com o {{site.data.keyword.mobilepushshort}} |
| `feedback-put`                | Ação                         | Armazena um feedback do usuário no banco de dados   |
| `feedback-analyze`            | Ação                         | Analisa um feedback com o {{site.data.keyword.toneanalyzershort}}   |
| `users-add-sequence`          | Sequência exposta como ação da web | `auth-validate` e `users-add`          |
| `feedback-put-sequence`       | Sequência exposta como ação da web | `auth-validate` e `feedback-put`       |
| `feedback-analyze-sequence`   | Arquivo                       | `read-document` do {{site.data.keyword.cloudant_short_notm}}, `feedback-analyze`, `users-prepare-notify` e `sendMessage` com o {{site.data.keyword.mobilepushshort}} |
| `feedback-analyze-trigger`    | Disparo                        | Chamado por {{site.data.keyword.openwhisk_short}} quando um feedback é armazenado no banco de dados |
| `feedback-analyze-rule`       | Regra                           | Vincula o acionador `feedback-analyze-trigger` com a sequência `feedback-analyze-sequence` |

### Compilar o código
{: java}
1. Na raiz do diretório de check-out, compile o código de ações
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### Configurar e implementar as ações
{: java}

2. Copiar template.local.env para local.env

   ```sh
   cp template.local.env local.env
   ```
3. Obtenha as credenciais para os serviços {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}} e {{site.data.keyword.appid_short}} no painel do {{site.data.keyword.Bluemix_notm}} (ou a saída dos comandos ibmcloud que executamos antes) e substitua os itens temporários em `local.env` pelos valores correspondentes. Essas propriedades serão injetadas em um pacote para que todas as ações possam obter acesso ao banco de dados.
4. Implemente as ações para {{site.data.keyword.openwhisk_short}}. `deploy.sh` carrega as credenciais de `local.env` para criar os bancos de dados {{site.data.keyword.cloudant_short_notm}} (usuários, feedback e humores) e implementar os artefatos {{site.data.keyword.openwhisk_short}} para o aplicativo.
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   É possível usar `./deploy.sh -- uninstall` para remover os artefatos do {{site.data.keyword.openwhisk_short}} depois de ter concluído o tutorial.
   {: tip}

### Configurar e implementar as ações
{: swift}

1. Copiar template.local.env para local.env
   ```sh
   cp template.local.env local.env
   ```
2. Obtenha as credenciais para os serviços {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}} e {{site.data.keyword.appid_short}} no painel do {{site.data.keyword.Bluemix_notm}} (ou a saída dos comandos ibmcloud que executamos antes) e substitua os itens temporários em `local.env` pelos valores correspondentes. Essas propriedades serão injetadas em um pacote para que todas as ações possam obter acesso ao banco de dados.
3. Implemente as ações para {{site.data.keyword.openwhisk_short}}. `deploy.sh` carrega as credenciais de `local.env` para criar os bancos de dados {{site.data.keyword.cloudant_short_notm}} (usuários, feedback e humores) e implementar os artefatos {{site.data.keyword.openwhisk_short}} para o aplicativo.

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   É possível usar `./deploy.sh -- uninstall` para remover os artefatos do {{site.data.keyword.openwhisk_short}} depois de ter concluído o tutorial.
   {: tip}

## Configurar e executar um aplicativo móvel nativo para coletar feedback do usuário
{: #mobile_app}

Nossas ações do {{site.data.keyword.openwhisk_short}} estão prontas para o nosso app móvel. Antes de executar o app móvel, é necessário definir suas configurações para destinar os serviços criados.

1. Com o Android Studio, abra o projeto localizado na pasta `android` de seu diretório de check-out.
2. Edite `android/app/src/main/res/values/credentials.xml` e preencha os espaços em branco com valores de credenciais. Você precisará do `tenantId` do {{site.data.keyword.appid_short}}, do `appGuid` e `clientSecret` do {{site.data.keyword.mobilepushshort}} e dos nomes de organização e espaço nos quais o {{site.data.keyword.openwhisk_short}} foi implementado. Para o host da API, ative o prompt de comandos ou o terminal e execute o comando
   ```sh
   ibmcloud fn property get -- apihost
   ```
   {: pre}
3. Abra `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` e `LoginAndRegistrationListener.java` e atualize a região do serviço de notificações push (BMSClient) e a região do AppID, dependendo do local no qual suas instâncias de serviço são criadas.
4. Construa o projeto.
5. Inicie o aplicativo em um dispositivo real ou com um emulador.
   Para que o emulador receba notificações push, certifique-se de selecionar uma imagem com as APIs do Google e de efetuar login com uma conta do Google dentro do emulador.
   {: tip}
6. Assista ao {{site.data.keyword.openwhisk_short}} no plano de fundo
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. No aplicativo, selecione **Efetuar login** para autenticar com uma conta do Facebook ou do Google. Depois de efetuar login, digite uma mensagem de feedback e pressione o botão **Enviar feedback**. Alguns segundos depois que o feedback tiver sido enviado, você deverá receber uma notificação push no dispositivo. O texto de notificação pode ser customizado modificando os documentos de modelo no banco de dados `moods` na instância de serviço do {{site.data.keyword.cloudant_short_notm}}. Use o botão **Visualizar token** para inspecionar os tokens de acesso e de identificação gerados pelo {{site.data.keyword.appid_short}} no login.
{: java}


1. Envie por push o Client SDK e outros SDKs estão disponíveis no CocoaPods e Carthage. Para essa solução, vamos usar CocoaPods.
2. Abra o Terminal e `cd` na pasta `followupapp`. Execute o comando a seguir para instalar as dependências necessárias.
   ```sh
   pod install
   ```
   {: pre}
3. Abra o arquivo com a extensão `.xcworkspace` localizada sob a pasta `followupapp` de seu diretório de check-out para ativar seu código no Xcode.
4. Edite `BMSCredentials.plist file` e preencha os espaços em branco com valores de credenciais. Você precisará do `tenantId` do {{site.data.keyword.appid_short}}, do `appGuid` e `clientSecret` do {{site.data.keyword.mobilepushshort}} e dos nomes de organização e espaço nos quais o {{site.data.keyword.openwhisk_short}} foi implementado. Para o host da API, ative o terminal e execute o comando

   ```sh
   ibmcloud fn property get -- apihost
   ```
   {: pre}
5. Abra `AppDelegate.swift` e atualize a região do serviço de notificações push (BMSClient) e a região do AppID, dependendo do local no qual suas instâncias de serviço são criadas.
6. Construa o projeto.
7. Inicie o aplicativo em um dispositivo real ou com um simulador.
8. Assista ao {{site.data.keyword.openwhisk_short}} no segundo plano, executando o comando a seguir em um Terminal.
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. No aplicativo, selecione **Efetuar login** para autenticar com uma conta do Facebook ou do Google. Depois de efetuar login, digite uma mensagem de feedback e pressione o botão **Enviar feedback**. Alguns segundos depois que o feedback tiver sido enviado, você deverá receber uma notificação push no dispositivo. O texto de notificação pode ser customizado modificando os documentos de modelo no banco de dados `moods` na instância de serviço do {{site.data.keyword.cloudant_short_notm}}. Use o botão **Visualizar token** para inspecionar os tokens de acesso e de identificação gerados pelo {{site.data.keyword.appid_short}} no login.
{: swift}

## Remover recursos

1. Use `deploy.sh` para remover os artefatos do {{site.data.keyword.openwhisk_short}}:

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. Exclua os serviços {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.appid_short}}, {{site.data.keyword.mobilepushshort}} e {{site.data.keyword.toneanalyzershort}} por meio do console do {{site.data.keyword.Bluemix_notm}}.

## Conteúdo relacionado

* O {{site.data.keyword.appid_short}} fornece uma configuração padrão para ajudar
com a configuração inicial dos provedores de identidade. Antes de publicar seu app, [atualize a configuração para as suas próprias credenciais](https://{DomainName}/docs/services/appid?topic=appid-social#social). Você também será capaz de [customizar o widget de login](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget).


* Ao criar uma ação do OpenWhisk Swift com um arquivo de origem Swift (arquivos .swift na pasta `actions` ), ela deve ser compilada em um binário antes da execução da ação. Uma vez feito, as chamadas subsequentes para a ação são muito mais rápidas até que o contêiner que contém sua ação seja limpo. Esse atraso é conhecido como o atraso de cold start.
  Para evitar o atraso de cold start, é possível compilar seu arquivo Swift em um
binário e, em seguida, fazer upload dele para o OpenWhisk em um arquivo zip. Como você precisa do esqueleto do OpenWhisk, a maneira mais fácil de criar o binário é construí-lo no mesmo ambiente em que ele é executado. Consulte [Empacotar uma Ação como um executável do Swift](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions) para obter etapas adicionais.
{: swift}
