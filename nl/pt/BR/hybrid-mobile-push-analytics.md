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

# Aplicativo móvel híbrido com Push Notifications
{: #hybrid-mobile-push-analytics}

Aprenda como é fácil criar rapidamente um aplicativo Hybrid Cordova com um serviço móvel de alto valor como {{site.data.keyword.mobilepushshort}} no {{site.data.keyword.Bluemix_notm}}.

Apache Cordova é uma estrutura de desenvolvimento móvel de software livre. Ele permite que você use as tecnologias padrão da web - HTML5, CSS3 e JavaScript para desenvolvimento de plataforma cruzada. Os aplicativos são executados em wrappers direcionados a cada plataforma e dependem das ligações de API compatíveis com padrões para acessar os recursos de cada dispositivo, como sensores, dados, status de rede, etc.

Este tutorial conduz você na criação de um aplicativo iniciador móvel Cordova, incluindo um serviço móvel, configurando o SDK do cliente, fazendo download do código em esqueleto e, em seguida, aprimorando ainda mais o aplicativo.

## Objetivos

* Criar um projeto móvel com o serviço {{site.data.keyword.mobilepushshort}}.
* Aprender a obter as credenciais de APNs e FCM.
* Fazer download do código e conclua a configuração necessária.
* Configurar, enviar e monitorar o {{site.data.keyword.mobilepushshort}}.

 ![](images/solution15/Architecture.png)

## Produtos

Este tutorial usa os produtos a seguir:
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Antes de Começar
{: #prereqs}

- [CLI](https://cordova.apache.org/docs/en/latest/guide/cli/) do Cordova para executar comandos do Cordova.
- [Pré-requisitos](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html) do Cordova-iOS e [Pré-requisitos](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html) do Cordova-Android
- Conta do Google para efetuar login no console do Firebase para o ID do emissor e a Chave de API do servidor.
- Conta do [Apple Developers](https://developer.apple.com/) para enviar notificações remotas da instância de serviço do {{site.data.keyword.mobilepushshort}} no {{site.data.keyword.Bluemix_notm}} (o provedor) para dispositivos iOS e aplicativos.
- Xcode e Android Studio para importar e aprimorar ainda mais seu código.

## Criar projeto móvel Cordova por meio do kit do iniciador
{: #get_code}
O Painel {{site.data.keyword.Bluemix_notm}} Mobile permite que você rastreie rapidamente o desenvolvimento de app móvel criando seu projeto por meio de um Kit do iniciador.
1. Navegue para [Painel Mobile](https://{DomainName}/developer/mobile/dashboard).
2. Clique em **Kits do iniciador** e clique em **Criar app**.
   ![](images/solution15/mobile_dashboard.png)
3. Insira um nome de projeto, esse pode ser o seu nome do app também.
4. Selecione **Cordova** como sua plataforma e clique em **Criar**.

    ![](images/solution15/create_cordova_project.png)
5. Clique em **Incluir recurso** > Dispositivo móvel > **Notificações push** e selecione a localização desejada para provisionar o serviço, o grupo de recursos e o plano de precificação **Lite**.
6. Clique em **Criar** para provisionar o serviço {{site.data.keyword.mobilepushshort}}. Um novo App será criado na guia **Apps**.

    **Nota:**{{site.data.keyword.mobilepushshort}} o serviço deve ser incluído no Iniciador vazio.

Na próxima etapa, você fará download do código em esqueleto e concluirá a configuração necessária.

## Fazer download do código e concluir a configuração necessária
{: #download_code}

Se você ainda não tiver transferido por download o código, use o painel {{site.data.keyword.Bluemix_notm}} Mobile para obter o código clicando no botão **Fazer download do código** em Projetos > **Seu projeto móvel**.

1. Em um IDE de sua escolha, navegue para `/platforms/android/project.properties` e substitua as duas últimas linhas (library.1 e library.2) pelas linhas abaixo

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
  As mudanças acima são específicas para o Android
  {: tip}
2. Para ativar o app em um emulador de Android, execute o comando a seguir em um terminal ou prompt de comandos
```
 $ cordova build android
 $ cordova run android
```
 Se você vir um erro, ative um emulador e tente executar o comando acima.
 {: tip}
3. Para visualizar o app no simulador iOS, execute os comandos abaixo em um terminal,

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    É possível localizar seu nome do projeto no arquivo `config.xml` executando o comando `cordova info`.
    {: tip}

## Obter credenciais de FCM e APNs
{: #obtain_fcm_apns_credentials}

 ### Configurar o Firebase Cloud Messaging (FCM)

  1. No [Console do Firebase](https://console.firebase.google.com), crie um novo projeto. Configure o nome como **hybridmobileapp**
  2. Navegue para as **Configurações** do projeto
  3. Na guia **Geral**, inclua dois aplicativos:
       1. um com o nome de pacote configurado para: **com.ibm.mobilefirstplatform.clientsdk.android.push**
       2. e um com o nome do pacote configurado como: **io.cordova.hellocordovastarter**
  4. Localize o ID do emissor e a Chave do servidor (também chamada de Chave de API posteriormente) na guia **Sistema de mensagens em nuvem**.
  5. No painel de serviço do {{site.data.keyword.mobilepushshort}}, configure o valor do ID do emissor e da Chave de API.


Consulte [Obter credenciais de FCM](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials) para obter as etapas detalhadas.
{: tip}

### Configurar Apple {{site.data.keyword.mobilepushshort}} Service (APNs)

  1. Acesse o portal do [Apple Developer](https://developer.apple.com/) e registre um ID do app.
  2. Crie um certificado SSL de APNs de desenvolvimento e
distribuição.
  3. Crie um perfil de
fornecimento de desenvolvimento.
  4. Configure a instância de serviço do {{site.data.keyword.mobilepushshort}} no {{site.data.keyword.Bluemix_notm}}.

Consulte [Obter credenciais de APNs e configurar o serviço {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials) para obter etapas detalhadas.
{: tip}

## Configurar, enviar e monitorar o {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. No index.js, sob a função `onDeviceReady`, substitua os valores `{pushAppGuid}` e

   `{pushClientSecret} `pelas **credenciais** do serviço de push - *appGuid* e *clientSecret*.

2. Acesse o painel Mobile > Projetos > Projeto Cordova, clique no serviço {{site.data.keyword.mobilepushshort}} e siga as etapas abaixo.

### APNs - Configurar a instância de serviço

Para usar o serviço {{site.data.keyword.mobilepushshort}} para enviar notificações, faça upload dos certificados .p12 que você criou na Etapa acima. Esse certificado contém a chave privada e os certificados SSL que são necessários para construir e publicar seu aplicativo.

**Nota:** depois que o arquivo `.cer` estiver em seu acesso de keychain, exporte-o para seu computador para criar um certificado `.p12`.

1. Clique em `{{site.data.keyword.mobilepushshort}}` na seção Serviços ou clique nos três pontos verticais ao lado do serviço {{site.data.keyword.mobilepushshort}} e selecione `Open dashboard`.
2. No Painel do {{site.data.keyword.mobilepushshort}}, você deverá ver a opção `Configure` em `Manage > Send Notifications`.

Para configurar os APNs no console de `Serviços de notificação push`, conclua as etapas:

1. Selecione `Configure` no painel de serviços Push Notification.
2. Escolha a `opção Móvel` para atualizar as informações no formulário APNs Push Credentials.
3. Selecione `Sandbox (development)` ou `Production (distribution)` conforme apropriado e, em seguida, faça upload do certificado `p.12` criado.
4. No campo Senha, insira a senha que está associada ao arquivo de certificado .p12 e, em seguida, clique em Salvar.

### FCM - Configurar a instância de serviço

1. Selecione **Mobile** e, em seguida, atualize a guia Credenciais de Push do GCM/FCM com o ID do emissor/Número do projeto e a Chave de API (Chave do servidor) que você criou inicialmente no console do Firebase.
2. Clique em **Salvar**. O serviço {{site.data.keyword.mobilepushshort}} está agora configurado.

### Enviar o {{site.data.keyword.mobilepushshort}}

1. Selecione **Enviar notificações** e edite uma mensagem escolhendo uma opção de envio. As opções suportadas são o dispositivo por tag, ID do dispositivo, ID do usuário, dispositivos Android, dispositivos IOS, notificações da web e todos os dispositivos.
   **Nota:** quando você selecionar a opção **Todos os dispositivos**, todos os dispositivos inscritos no {{site.data.keyword.mobilepushshort}} receberão notificações.

2. No campo **Mensagem**, componha sua mensagem. Escolha a
configurar das definições opcionais conforme necessário.
3. Clique em **Enviar** e verifique se seu dispositivo físico recebeu a notificação.

### Monitorar notificações enviadas

É possível monitorar suas notificações enviadas navegando para a seção **Monitoramento**.
Agora, o serviço IBM {{site.data.keyword.mobilepushshort}} estende os recursos para monitorar o
desempenho de push gerando gráficos de seus dados do usuário. É possível usar o utilitário para listar todos os {{site.data.keyword.mobilepushshort}} enviados ou listar todos os dispositivos registrados e analisar informações em uma base diária, semanal ou mensal. ![](images/solution6/monitoring_messages.png)

## Conteúdo relacionado
{: #related_content}

- [notificações baseadas em tag](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [APIs {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Segurança em {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

