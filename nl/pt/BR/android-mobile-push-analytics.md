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

# Aplicativo móvel nativo Android com Push Notifications
{: #android-mobile-push-analytics}

Aprenda como é fácil criar rapidamente um aplicativo Android nativo com um serviço móvel de alto valor como o {{site.data.keyword.mobilepushshort}} no {{site.data.keyword.Bluemix_notm}}.

Este tutorial conduz você na criação de um aplicativo iniciador móvel, incluindo um serviço móvel, configurando o SDK do cliente, importando o código para o Android Studio e, em seguida, aprimorando ainda mais o aplicativo.

## Objetivos
{: #objectives}

* Criar um app móvel com o serviço {{site.data.keyword.mobilepushshort}}.
* Obter as credenciais do FCM.
* Fazer download do código e concluir a configuração necessária.
* Configurar, enviar e monitorar o {{site.data.keyword.mobilepushshort}}.

![](images/solution9/Architecture.png)

## Produtos
{: #products}

Este tutorial usa os produtos a seguir:
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Antes de Começar
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html) para importar e aprimorar seu código.
- Conta do Google para efetuar login no console do Firebase para o ID do emissor e a Chave de API do servidor.

## Criar app móvel Android por meio do kit do iniciador
{: #get_code}
O Painel {{site.data.keyword.Bluemix_notm}} Mobile permite um caminho rápido para o desenvolvimento de app móvel, criando seu app por meio de um Kit do Iniciador.
1. Navegue para [Painel Mobile](https://{DomainName}/developer/mobile/dashboard)
2. Clique em **Kits do iniciador** e clique em **Criar app**.
   ![](images/solution9/mobile_dashboard.png)
3. Insira um nome de app, esse pode ser seu nome de projeto Android também.
4. Selecione **Android** como sua plataforma e clique em **Criar**.

    ![](images/solution9/create_mobile_project.png)
5. Clique em **Incluir recurso** > Dispositivo móvel > **Notificações push** e selecione a localização desejada para provisionar o serviço, o grupo de recursos e o plano de precificação **Lite**.
6. Clique em **Criar** para provisionar o serviço {{site.data.keyword.mobilepushshort}}. Um novo App será criado na guia **Apps**.

    **Nota:** o serviço {{site.data.keyword.mobilepushshort}} deve ser incluído com o Iniciador vazio.
    Na próxima etapa, você obterá as credenciais do Firebase Cloud Messaging (FCM).

Na próxima etapa, você fará download do código em esqueleto e configurará o Push Android SDK.

## Fazer download do código e concluir a configuração necessária
{: #download_code}

Se você ainda não tiver transferido por download o código, use o painel {{site.data.keyword.Bluemix_notm}} Mobile para obter o código clicando no botão **Fazer download do código** em Apps > **Seu app móvel**.
O código transferido por download é fornecido com o SDK do cliente **{{site.data.keyword.mobilepushshort}}** incluído. O SDK do cliente está disponível no Gradle e no Maven. Para este tutorial, você usará **Gradle**.

1. Ative o Android Studio > **Abra um projeto existente do Android Studio** e aponte para o código transferido por download.
2. A construção do **Gradle** será acionada automaticamente e todas as dependências serão transferidas por download.
3. Inclua a dependência **Serviços do Google Play** em seu arquivo de nível de Módulo `build.gradle (Module: app)` no final, após `dependencies{.....}`
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. Copie o arquivo `google-services.json` criado e transferido por download para o diretório raiz do módulo aplicativo Android. Observe que o arquivo `google-service.json` inclui os nomes de pacote adicionados.
5. As permissões necessárias estão todas dentro do arquivo `AndroidManifest.xml` e dependências. O Push e o Analytics são incluídos em **build.gradle (Module: app)**.
6. O serviço de intento **Firebase Cloud Messaging (FCM)** e os filtros de intento para as notificações de eventos `RECEIVE` e `REGISTRATION` são incluídos em `AndroidManifest.xml`

## Obter credenciais do FCM
{: #obtain_fcm_credentials}

O Firebase Cloud Messaging (FCM) é o gateway usado para entregar o {{site.data.keyword.mobilepushshort}} para dispositivos Android, navegador Google Chrome e apps e extensões do Chrome. Para configurar o serviço {{site.data.keyword.mobilepushshort}} no console, é necessário obter suas credenciais do FCM (ID do emissor e chave API).

A chave de API é armazenada de forma segura e usada pelo serviço {{site.data.keyword.mobilepushshort}} para se conectar ao servidor FCM e o ID do emissor (número do projeto) é usado pelo Android SDK e pelo JS SDK for Google Chrome e Mozilla Firefox no lado do cliente. Para configurar o FCM e obter suas credenciais, conclua as etapas:

1. Visite o [Console do Firebase](https://console.firebase.google.com/?pli=1) - uma conta do usuário do Google é necessária.
2. Selecione **Incluir projeto**.
3. Na janela **Criar um projeto**, forneça um nome de projeto, escolha um país/região e clique em **Criar projeto**.
4. Na área de janela de navegação à esquerda, selecione **Configurações** (clique no Ícone de Configurações próximo a **Visão geral**) > **Configurações do projeto**.
5. Escolha a guia Cloud Messaging para obter suas credenciais do projeto - Chave API do servidor e um ID do emissor.
    **Nota:** a chave do servidor listada no FCM é a mesma Chave de API do Servidor.
    ![](images/solution9/fcm_console.png)

Também seria necessário gerar o arquivo `google-services.json`. Conclua as etapas a seguir:

1. No console do Firebase, clique no ícone **Configurações do projeto** > guia **Geral** no projeto criado acima e selecione **Incluir Firebase em seu app Android**

    ![](images/solution9/firebase_project_settings.png)
2. Na janela modal do app **Incluir Firebase em seu Android**, inclua **com.ibm.mobilefirstplatform.clientsdk.android.push** como o Nome do pacote para registrar o Android SDK do {{site.data.keyword.mobilepushshort}}. Os campos Apelido do app e SHA-1 são opcionais. Clique em **REGISTRAR APP** > **Continuar** > **Concluir**.

    ![](images/solution9/add_firebase_to_your_app.png)

3. Clique em **INCLUIR APP** > **Incluir Firebase em seu app**. Inclua o nome do pacote de seu aplicativo inserindo o nome do pacote **com.ibm.mysampleapp**, em seguida, continue para incluir o Firebase em sua janela do app Android. Os campos Apelido do app e SHA-1 são opcionais. Clique em **REGISTRAR APP** > Continuar > Concluir.
     **Nota:** é possível localizar o nome do pacote de seu aplicativo no arquivo `AndroidManifest.xml` depois de fazer download do código.
4. Faça download do arquivo de configuração mais recente `google-services.json` em **Seus apps**.

    ![](images/solution9/google_services.png)

    **Nota**: FCM é a nova versão do Google Cloud Messaging (GCM). Assegure-se de usar as credenciais do FCM para novos apps. Apps existentes continuarão a funcionar com configurações do GCM.

*As etapas e a IU do console do Firebase estão sujeitas a mudanças, consulte a documentação do Google para a parte do Firebase, se necessário*

## Configurar, enviar e monitorar o {{site.data.keyword.mobilepushshort}}

{: #configure_push}

1. o {{site.data.keyword.mobilepushshort}} SDK já foi importado para o app e o código de inicialização de Push pode ser localizado no arquivo `MainActivity.java`.

    **Nota:** as credenciais de serviço fazem parte do arquivo `/res/values/credentials.xml`.
2. O registro para notificações acontece em `MainActivity.java`. (Opcional) Forneça um USER_ID exclusivo.
3. Execute o app em um dispositivo físico ou Emulador para receber notificações.
4. Abra o serviço {{site.data.keyword.mobilepushshort}} em **Serviços móveis** > **Serviços existentes** no painel {{site.data.keyword.Bluemix_notm}} Mobile e, para enviar o {{site.data.keyword.mobilepushshort}} básico, conclua as etapas a seguir:
   - Clique em **Gerenciar** > **Configurar**.
   - Selecione **Mobile** e, em seguida, atualize a guia Credenciais de Push do GCM/FCM com o ID do emissor/Número do projeto e a Chave de API (Chave do servidor) que você criou inicialmente no console do Firebase.

     ![](images/solution9/configure_push_notifications.png)
   - Clique em **Salvar**. O serviço {{site.data.keyword.mobilepushshort}} está agora configurado.
   - Selecione **Enviar notificações** e edite uma mensagem escolhendo uma opção de envio. As opções suportadas são: dispositivo por tag, ID do dispositivo, ID do usuário, dispositivos Android, dispositivos IOS, notificações da web e todos os dispositivos.
     **Nota:** quando você selecionar a opção **Todos os dispositivos**, todos os dispositivos inscritos no {{site.data.keyword.mobilepushshort}} receberão notificações.
   - No campo **Mensagem**, componha sua mensagem. Escolha a
configurar das definições opcionais conforme necessário.
   - Clique em **Enviar** e verifique se seu dispositivo físico recebeu a notificação.

     ![](images/solution9/android_send_notifications.png)
5. Você deverá ver uma notificação em seu dispositivo Android.

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. É possível monitorar suas notificações enviadas navegando para **Monitoramento** no Serviço {{site.data.keyword.mobilepushshort}}.
     Agora, o serviço IBM {{site.data.keyword.mobilepushshort}} estende os recursos para monitorar o
desempenho de push gerando gráficos de seus dados do usuário. É possível usar o utilitário para listar todos os {{site.data.keyword.mobilepushshort}} enviados ou listar todos os dispositivos registrados e analisar informações em uma base diária, semanal ou mensal. ![](images/solution6/monitoring_messages.png)

## Conteúdo relacionado
{: #related_content}
- [Customizar as configurações do {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [notificações baseadas em tag](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [APIs {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Segurança em {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

