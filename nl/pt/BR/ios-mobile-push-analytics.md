---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-08"
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

# Aplicativo móvel iOS com Push Notifications
{: #ios-mobile-push-analytics}

Aprenda como é fácil criar rapidamente um aplicativo iOS Swift com o serviço móvel de alto valor {{site.data.keyword.mobilepushshort}} no {{site.data.keyword.Bluemix_short}}.

Este tutorial conduz você na criação de um aplicativo iniciador móvel, incluindo um serviço móvel, configurando o SDK do cliente, importando o código para Xcode e, em seguida, aprimorando ainda mais o aplicativo.
{:shortdesc: .shortdesc}

## Objetivos
{:#objectives}

- Criar um app móvel com o {{site.data.keyword.mobilepushshort}} e os serviços do {{site.data.keyword.mobileanalytics_short}} por meio do kit do iniciador Swift básico.
- Obtenha credenciais de APNs e configure a instância de serviço do {{site.data.keyword.mobilepushshort}}.
- Fazer download do código e do SDK do cliente de configuração.
- Enviar e monitorar o {{site.data.keyword.mobilepushshort}}.

  ![](images/solution6/Architecture.png)

## Produtos
{:#products}

Este tutorial usa os produtos a seguir:
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Antes de Começar
{: #prereqs}

1. Conta do [Apple Developers](https://developer.apple.com/) para enviar notificações remotas da instância de serviço do {{site.data.keyword.mobilepushshort}} no {{site.data.keyword.Bluemix_short}} (o provedor) para dispositivos iOS e aplicativos.
2. Xcode para importar e aprimorar seu código.

## Criar um app móvel por meio do kit do iniciador Swift básico
{: #get_code}

1. Navegue para [Painel Mobile](https://{DomainName}/developer/mobile/dashboard) para criar seu `App` por meio de `Starter Kits` predefinidos.
2. Clique em **Kits do iniciador** e role para baixo para selecionar o Kit do iniciador **Básico**.
    ![](images/solution6/mobile_dashboard.png)
3. Insira um nome de app que também será o projeto Xcode e o nome do app.
4. Selecione `iOS Swift` como sua plataforma e clique em **Criar**.
    ![](images/solution6/create_mobile_project.png)
5. Clique em **Incluir recurso** > Dispositivo móvel > **Notificações push** e selecione a localização desejada para provisionar o serviço, o grupo de recursos e o plano de precificação **Lite**.
6. Clique em **Criar** para provisionar o serviço {{site.data.keyword.mobilepushshort}}. Um novo App será criado na guia **Apps**.

​      **Nota:** o serviço {{site.data.keyword.mobilepushshort}} já deve estar incluído com o Iniciador vazio.

## Fazer download dos SDKs do cliente de configuração e código
{: #download_code}

Se você ainda não tiver transferido por download o código, clique em `Download Code` em Apps > `Your Mobile App`
O código transferido por download é fornecido com o **{{site.data.keyword.mobilepushshort}}** Client SDK incluído. O Client SDK está disponível em CocoaPods e Carthage. Para essa solução, você usará CocoaPods.

1. Para instalar o CocoaPods em sua máquina, abra o `Terminal` e execute o comando a seguir.
   ```
   sudo gem install cocoapods
   ```
   {: pre:}
2. Descompacte o arquivo ZIP transferido por download e, usando o terminal, navegue até a pasta descompactada

   ```
   cd <Name of Project>
   ```
   {: pre:}
3. A pasta já inclui um `podfile` com dependências necessárias. Execute o comando a seguir para instalar as dependências (Client SDKs) e as dependências necessárias serão instaladas.

  ```
  pod install
  ```
  {: pre:}

## Obtenha credenciais de APNs e configure a instância de serviço do {{site.data.keyword.mobilepushshort}}.
{: #obtain_apns_credentials}

   Para dispositivos iOS e aplicativos, o Apple Push Notification Service (APNs) permite que os desenvolvedores de aplicativos enviem notificações remotas da instância de serviço do {{site.data.keyword.mobilepushshort}} no {{site.data.keyword.Bluemix_short}} (o provedor) para dispositivos iOS e aplicativos. Mensagens são enviadas para um aplicativo de
destino no dispositivo.

   É necessário obter e configurar suas credenciais do APNs. Os certificados do APNs são gerenciados com segurança pelo serviço {{site.data.keyword.mobilepushshort}} e usados para se conectar ao servidor APNs como um provedor.

### Registrando um ID de app

   O ID de app (o identificador de pacote configurável) é um
identificador exclusivo que identifica um aplicativo específico. Cada aplicativo requer um ID de app. Serviços, como o serviço {{site.data.keyword.mobilepushshort}}, são configurados para o ID do app.
   Assegure-se de que você tenha uma conta do [Apple Developers](https://developer.apple.com/). Esse é um pré-requisito obrigatório.

   1. Acesse o portal do [Apple Developer](https://developer.apple.com/), clique em `Member Center` e selecione `Certificates, IDs &  Profiles`.
   2. Acesse `Identifiers` > seção IDs de app.
   3. Na página `Registrando IDs do app`, forneça o nome do app no campo Nome da descrição do ID do app. Por exemplo: ACME {{site.data.keyword.mobilepushshort}}. Forneça uma sequência para o Prefixo ID do app.
   4. Para o Sufixo de ID de app, escolha `ID de app explícito` e forneça um valor de ID de pacote. Recomenda-se fornecer uma sequência de estilo de nome de domínio reverso. Por exemplo: com.ACME.push.
   5. Marque a caixa de seleção `{{site.data.keyword.mobilepushshort}}` e clique em `Continuar`.
   6. Percorra as configurações e clique em `Registrar` > `Pronto`.
     Seu app ID agora está registrado.

     ![](images/solution6/push_ios_register_appid.png)

### Crie um certificado SSL de APNs de desenvolvimento e distribuição
   Antes de obter um certificado APNs, deve-se primeiro gerar uma solicitação de assinatura de certificado (CSR) e enviá-la para Apple, a autoridade de certificação (CA). O CSR contém informações que identificam sua empresa e a chave pública e privada que você usa para assinar para seu Apple {{site.data.keyword.mobilepushshort}}. Depois, gere o certificado SSL no
            portal de Desenvolvedor de iOS. O certificado, junto com
seu público e chave privada, é armazenado no Keychain Access.
   É possível usar APNs de dois modos:

- Modo de ambiente de simulação para desenvolvimento e teste.
- Modo de produção ao distribuir aplicativos por meio da App Store (ou outros mecanismos de distribuição da empresa).

   É necessário obter certificados separados para
seus ambientes de desenvolvimento e distribuição. Os certificados são
associados a um ID de app para o app que é o destinatário das
notificações remotas. Para produção, é possível criar até dois
certificados. O {{site.data.keyword.Bluemix_short}} usa os certificados para estabelecer uma conexão SSL com APNs.

   1. Acesse o website do Apple Developer, clique em **Centro de membros** e selecione **Certificados, IDs e perfis**.
   2. Na área **Identificadores**, clique em
**IDs de app**.
   3. Em lista de IDs de app, selecione seu ID do app e, em seguida, selecione `Edit`.
   4. Marque a caixa de seleção **{{site.data.keyword.mobilepushshort}}** e, em seguida, na área de janela **Certificado SSL de desenvolvimento**, clique em **Criar certificado**.

     ![Certificados SSL do Push Notification](images/solution6/certificate_createssl.png)

   5. Quando a **tela Sobre a criação de uma Solicitação de Assinatura de Certificado (CSR)** for exibida, siga as instruções mostradas para criar um arquivo de Solicitação de Assinatura de Certificado (CSR). Forneça um nome significativo que ajude a identificar se ele é um certificado para desenvolvimento (ambiente de simulação) ou distribuição (produção); por exemplo, sandbox-apns-certificate ou production-apns-certificate.
   6. Clique em **Continuar** e na tela Fazer upload do arquivo CSR, clique em **Escolher arquivo** e selecione o arquivo **CertificateSigningRequest.certSigningRequest** que você acabou de criar. Clique em **Continuar**.
   7. Na área de janela Download, instalação e backup, clique em Download. O arquivo **aps_development.cer** é transferido por download.
    ![Fazer download do certificado](images/solution6/push_certificate_download.png)

        ![Gerar certificado](images/solution6/generate_certificate.png)
   8. Em seu mac, abra **Acesso ao keychain**, **Arquivo**, **Importar** e selecione o arquivo .cer transferido por download para instalá-lo.
   9. Clique com o botão direito no novo certificado e na chave privada e, em seguida, selecione **Exportar** e mude o **Formato de arquivo** para o formato de troca de informações pessoais (formato `.p12`).
     ![Exportar certificado e chaves](images/solution6/keychain_export_key.png)
   10. No campo **Salvar como**, dê um nome significativo para o certificado. Por exemplo, `sandbox_apns.p12` ou **production_apns.p12**, em seguida, clique em Salvar.
    ![Exportar certificado e chaves](images/solution6/certificate_p12v2.png)
   11. No campo **Inserir uma senha**, insira uma senha para proteger os itens exportados, em seguida, clique em OK. É possível usar essa senha para definir as configurações de APNs no console de serviço do {{site.data.keyword.mobilepushshort}}.
    ![Exportar certificado e chaves](images/solution6/export_p12.png)
   12. O **Key Access.app** solicita que
você exporte sua chave da tela **Keychain**. Insira sua senha
administrativa para seu Mac para permitir que o sistema exporte esses itens; em seguida,
selecione a opção `Sempre permitir`. Um certificado
`.p12` é gerado em sua área de trabalho.

      Para SSL de produção, na área de janela **Certificado SSL de produção**, clique em **Criar certificado** e repita as Etapas 5 a 12 acima.
      {:tip}

### Criando um perfil de fornecimento de desenvolvimento
   O perfil de fornecimento funciona com o ID de app para
determinar quais dispositivos podem instalar e executar seu app e
quais serviços seu app pode acessar. Para cada ID de app, você cria
dois perfis de fornecimento: um para desenvolvimento e outro para
distribuição. Xcode usa o perfil de fornecimento de desenvolvimento
para determinar quais desenvolvedores podem criar o aplicativo e
quais dispositivos podem ser testados no aplicativo.

   Certifique-se de registrar um ID de app, de ativá-lo para o serviço {{site.data.keyword.mobilepushshort}} e de configurá-lo para
usar um certificado SSL APNs de desenvolvimento e produção.

   Crie um perfil de fornecimento de desenvolvimento, da seguinte forma:

   1. Acesse o portal do [Apple Developer](https://developer.apple.com/), clique em `Member Center` e selecione `Certificates, IDs &  Profiles`.
   2. Acesse a [Biblioteca do Mac Developer](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site), role para a seção `Creating Development Provisioning Profiles` e siga as instruções para criar um perfil de desenvolvimento.
     **Nota:** ao configurar um perfil de provisão de desenvolvimento, selecione as opções a seguir:

     - **iOS App Development**
     - **Para apps iOS e watchOS **

### Criando um perfil de fornecimento de distribuição de
armazenamento
   Use o perfil de fornecimento de armazenamento para
enviar seu app para distribuição para a App Store.

   1. Acesse o [Apple Developer](https://developer.apple.com/), clique em `Member Center` e selecione `Certificates, IDs & Profiles`.
   2. Dê um clique duplo no perfil de fornecimento
transferido por download para instalá-lo em Xcode.
     Depois de obter as credenciais, a próxima etapa será [Configurar uma instância de serviço](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2).

### Configurar
a instância de
serviço

   Para usar o serviço {{site.data.keyword.mobilepushshort}} para enviar notificações, faça upload dos certificados .p12 que você criou na Etapa acima. Esse certificado contém a chave privada e os certificados SSL que são necessários para construir e publicar seu aplicativo.

   **Nota:** depois que o arquivo `.cer` estiver em seu acesso ao keychain, exporte-o para seu computador para criar um certificado `.p12`.

1. Clique em {{site.data.keyword.mobilepushshort}} sob a seção Serviços ou clique em três pontos verticais ao lado do serviço {{site.data.keyword.mobilepushshort}} e selecione `Open dashboard`.
2. No Painel do {{site.data.keyword.mobilepushshort}}, você deverá ver a opção `Configure` em `Manage`.

Para configurar os APNs no console de `Serviços de notificação push`, conclua as etapas:

1. Escolha a `opção Móvel` para atualizar as informações no formulário APNs Push Credentials.
2. Selecione `Sandbox/Development APNs Server` ou `Production APNs Server` conforme apropriado e, em seguida, faça upload do certificado `.p12` que você criou.
3. No campo Senha, insira a senha que está associada ao arquivo de certificado .p12 e, em seguida, clique em Salvar.

![](images/solution6/Mobile_push_configure.png)

## Configurar, enviar e monitorar o {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. O código de inicialização de push (em `func application`) e o código de registro de notificação podem ser localizados em `AppDelegate.swift`. Forneça um USER_ID exclusivo (opcional).
2. Execute o app em um dispositivo físico, pois não é possível enviar as notificações para um iPhone Simulator.
3. Abra o serviço {{site.data.keyword.mobilepushshort}} em `Mobile Services` > **Serviços existentes** no painel {{site.data.keyword.Bluemix_short}} Mobile e, para enviar o {{site.data.keyword.mobilepushshort}} básico, conclua as etapas a seguir:
  * Selecione `Messages` e edite uma mensagem escolhendo uma opção Enviar para. As opções suportadas são: Dispositivo por tag, ID do dispositivo, ID do usuário, Dispositivos Android, Dispositivos iOS, Notificações da web, Todos os dispositivos e outros navegadores.

       **Nota:** quando você selecionar a opção Todos os dispositivos, todos os dispositivos inscritos no {{site.data.keyword.mobilepushshort}} receberão notificações.
  * No campo `Mensagem`, componha sua mensagem. Escolha a
configurar das definições opcionais conforme necessário.
  * Clique em `Send` e verifique se seus dispositivos físicos receberam a notificação.
    ![](images/solution6/send_notifications.png)

4. Você deverá ver uma notificação em seu iPhone.

   ![](images/solution6/iphone_notification.png)

5. É possível monitorar suas notificações enviadas navegando para `Monitoring` no Serviço {{site.data.keyword.mobilepushshort}}.

Agora, o serviço IBM {{site.data.keyword.mobilepushshort}} estende os recursos para monitorar o
desempenho de push gerando gráficos de seus dados do usuário. É possível usar o utilitário para listar todos os {{site.data.keyword.mobilepushshort}} enviados ou listar todos os dispositivos registrados e analisar informações em uma base diária, semanal ou mensal. ![](images/solution6/monitoring_messages.png)

## Conteúdo relacionado
{: #related_content}

- [notificações baseadas em tag](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [APIs {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Segurança em {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)
