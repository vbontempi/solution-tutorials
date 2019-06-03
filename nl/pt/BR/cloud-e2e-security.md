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

# Aplicar segurança de ponta a ponta a um aplicativo em nuvem
{: #cloud-e2e-security}

Nenhuma arquitetura do aplicativo está completa sem um entendimento claro de possíveis riscos de segurança e como proteger contra tais ameaças. Os dados do aplicativo são um recurso crítico que não pode ser perdido, comprometido ou roubado. Além disso, os dados devem ser protegidos em repouso e em trânsito por meio de técnicas de criptografia. A criptografia de dados em repouso protege as informações da divulgação mesmo quando elas são perdidas ou roubadas. Criptografar dados em trânsito (por exemplo, na Internet) por meio de métodos como HTTPS, SSL e TLS evita a espionagem do tráfego de rede e os ataques man-in-the-middle.

Autenticar e autorizar o acesso dos usuários a recursos específicos é outro requisito comum para muitos aplicativos. Diferentes esquemas de autenticação podem precisar ser suportados: clientes e fornecedores usando identidades sociais, parceiros de diretórios hospedados na nuvem e funcionários do provedor de identidade de uma organização.

Este tutorial conduz você pelos serviços de segurança de chave disponíveis no catálogo do {{site.data.keyword.cloud}} e como usá-los juntos. Um aplicativo que fornece compartilhamento de arquivo colocará os conceitos de segurança em prática.
{:shortdesc}

## Objetivos
{: #objectives}

* Criptografar conteúdo em depósitos de armazenamento com suas próprias chaves de criptografia
* Requerer que os usuários se autentiquem antes de acessar um aplicativo
* Monitorar e auditar as chamadas da API relacionadas à segurança e outras ações entre os serviços de nuvem

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* Opcional: [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

Este tutorial requer uma [conta não Lite](https://{DomainName}/docs/account?topic=account-accounts#accounts) e pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

O tutorial apresenta um aplicativo de amostra que permite que grupos de usuários façam upload de arquivos para um conjunto de armazenamentos comum e forneçam acesso a esses arquivos por meio de links compartilháveis. O aplicativo é escrito em Node.js e implementado como um contêiner do Docker no {{site.data.keyword.containershort_notm}}. Ele alavanca vários serviços e recursos relacionados à segurança para melhorar a postura de segurança do aplicativo.

<p style="text-align: center;">

  ![Arquitetura](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. O usuário se conecta ao aplicativo.
2. Se estiver usando um domínio customizado e um certificado TLS, o certificado será gerenciado e implementado por meio do {{site.data.keyword.cloudcerts_short}}.
3. O {{site.data.keyword.appid_short}} protege o aplicativo e redireciona o usuário para a página de autenticação. Os usuários também podem se inscrever.
4. O aplicativo é executado em um cluster Kubernetes por meio de uma imagem armazenada no {{site.data.keyword.registryshort_notm}}. Essa imagem é varrida automaticamente para vulnerabilidades.
5. Os arquivos transferidos por upload são armazenados no {{site.data.keyword.cos_short}} com metadados associados armazenados no {{site.data.keyword.cloudant_short_notm}}.
6. Os depósitos de armazenamento de arquivo alavancam uma chave fornecida pelo usuário para criptografar dados.
7. As atividades de gerenciamento de aplicativo são registradas pelo {{site.data.keyword.cloudaccesstrailshort}}.

## Antes de Começar
{: #prereqs}

1. Instale todas as ferramentas necessárias de linha de comandos (CLI) [seguindo estas etapas](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
2. Assegure-se de que você tenha a versão mais recente de plug-ins usados neste tutorial; use `ibmcloud plugin update --all` para fazer upgrade.

## Criar serviços
{: #setup}

### Decidir onde implementar o aplicativo

1. Identifique o **local**, a **organização e o espaço do Cloud Foundry** e o **grupo de recursos** em que você implementará o aplicativo e seus recursos.

### Capturar atividades do usuário e do aplicativo 
{: #activity-tracker }

O serviço {{site.data.keyword.cloudaccesstrailshort}} registra atividades iniciadas pelo usuário que mudam o estado de um serviço no {{site.data.keyword.Bluemix_notm}}. No final deste tutorial, você revisará os eventos que foram gerados concluindo as etapas do tutorial.

1. Acesse o catálogo do {{site.data.keyword.Bluemix_notm}} e crie uma instância do [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker). Observe que pode haver somente uma instância do {{site.data.keyword.cloudaccesstrailshort}} por espaço do Cloud Foundry.
2. Após a instância ser criada, mude seu nome para **secure-file-storage-activity-tracker**.
3. Para visualizar todos os eventos do rastreador de atividades, assegure-se de ter permissões designadas:
   1. Acesse [Identidade & Acesso > Usuários](https://{DomainName}/iam/#/users).
   2. Selecione seu nome do usuário na lista.
   3. Em **Políticas de acesso**, se estiver ausente, crie uma política para o serviço {{site.data.keyword.loganalysisshort_notm}} com a função de **Visualizador** no local em que o serviço foi criado.
   4. Em **Acesso ao Cloud Foundry**, assegure-se de que você tenha a função de **Desenvolvedor** no espaço do Cloud Foundry no qual o {{site.data.keyword.cloudaccesstrailshort}} foi provisionado. Se não, trabalhe com o gerenciador do Cloud Foundry da sua organização para designar a função.

Localize instruções detalhadas sobre a configuração de permissões na [documentação do {{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy).
{: tip}

### Criar um cluster para o aplicativo

O {{site.data.keyword.containershort_notm}} fornece um ambiente para implementar apps altamente disponíveis nos contêineres do Docker que são executados em clusters Kubernetes.

Ignore esta seção se houver um cluster **Padrão** existente que você deseja reutilizar com este tutorial.
{: tip}

1. Acesse a [página de criação de cluster](https://{DomainName}/containers-kubernetes/catalog/cluster/create).
   1. Configure o **Local** para aquele usado em etapas anteriores.
   2. Configure **Tipo de cluster** como **Padrão**.
   3. Configure **Disponibilidade** como **Zona única**.
   4. Selecione uma **Zona principal**.
2. Mantenha a **Versão do Kubernetes** e o **Isolamento de hardware** padrão.
3. Se você planejar implementar somente este tutorial neste cluster, configure **Nós do trabalhador** como **1**.
4. Configure o **Nome do cluster** como **secure-file-storage-cluster**.
5. Clique no botão **Criar cluster**.

Enquanto o cluster estiver sendo provisionado, você criará os outros serviços necessários para o tutorial.

### Usar suas próprias chaves de criptografia

O {{site.data.keyword.keymanagementserviceshort}} ajuda a provisionar chaves criptografadas para apps em serviços {{site.data.keyword.Bluemix_notm}}. O {{site.data.keyword.keymanagementserviceshort}} e o {{site.data.keyword.cos_full_notm}} [trabalham juntos para proteger seus dados em repouso](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos). Nesta seção, você criará uma chave raiz para o depósito de armazenamento.

1. Crie uma instância do [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms).
   * Configure o nome como **secure-file-storage-kp**.
   * Selecione o grupo de recursos no qual criar a instância de serviço.
2. Em **Gerenciar**, clique no botão **Incluir chave** para criar uma nova chave raiz. Ela será usada para criptografar o conteúdo do depósito de armazenamento.
   * Configure o nome como **secure-file-storage-root-enckey**.
   * Configure o tipo de chave como **Chave raiz**.
   * Em seguida, **Gerar chave**.

Use Bring your own key (BYOK) [importando uma chave raiz existente](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys).
{: tip}

### Configurar o armazenamento para arquivos de usuário

O aplicativo de compartilhamento de arquivo salva arquivos em um depósito do {{site.data.keyword.cos_short}}. O relacionamento entre os arquivos e os usuários é armazenado como metadados em um banco de dados {{site.data.keyword.cloudant_short_notm}}. Nesta seção, você criará e configurará esses serviços.

#### Um depósito para o conteúdo

1. Crie uma instância do [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
   * Configure o **name** como **secure-file-storage-cos**.
   * Use o mesmo **grupo de recursos** dos serviços anteriores.
2. Em **Credenciais de serviço**, crie uma *Nova credencial*.
   * Configure o **name** como **secure-file-storage-cos-acckey**.
   * Configure **Função** como **Gravador**.
   * Não especifique um **ID de serviço**.
   * Configure **Parâmetros de configuração sequenciais** como **{"HMAC":true}**. Isso é necessário para gerar URLs pré-assinadas.
   * Clique em **Incluir**.
   * Anote as credenciais clicando em **Visualizar credenciais**. Você precisará deles em uma etapa posterior.
3. Clique em **Terminal** no menu: configure **Resiliência** como **Regional** e configure o **Local** para o local de destino. Copie o terminal em serviço **Público**. Ele será usado posteriormente na configuração do aplicativo.

Antes de criar o depósito, você concederá acesso **secure-file-storage-cos** para a chave raiz armazenada em **secure-file-storage-kp**.

1. Acesse [Identidade e acesso > Autorizações](https://{DomainName}/iam/#/authorizations) no console do {{site.data.keyword.cloud_notm}}.
2. Clique no botão **Criar**.
3. No menu **Serviço de origem**, selecione **Cloud Object Storage**.
4. No menu **Instância de serviço de origem**, selecione o serviço **secure-file-storage-cos** criado anteriormente.
5. No menu **Serviço de destino**, selecione **Key Protect**.
6. No menu **Instância de serviço de destino**, selecione o serviço **secure-file-storage-kp** para autorizar.
7. Ative a função **Leitor**.
8. Clique no botão **Autorizar**.

Finalmente, crie o depósito.

1. Acesse a instância de serviço **secure-file-storage-cos** na [Lista de recursos](https://{DomainName}/resources).
2. Clique em **Criar depósito**.
   1. Configure o **nome** para um valor exclusivo, como **&lt;your-initials&gt;-secure-file-upload**.
   2. Configure **Resiliência** como **Regional**.
   3. Configure **Local** para o mesmo local em que você criou o serviço **secure-file-storage-kp**.
   4. Configure **Classe de armazenamento** como **Padrão**
3. Marque a caixa de seleção para **Incluir teclas do Key Protect**.
   1. Selecione o serviço **secure-file-storage-kp**.
   2. Selecione **secure-file-storage-root-enckey** como a chave.
4. Clique em **Criar depósito**.

#### Relacionamentos do mapa de banco de dados entre usuários e seus arquivos

O banco de dados {{site.data.keyword.cloudant_short_notm}} conterá metadados para todos os arquivos transferidos por upload do aplicativo.

1. Crie uma instância do [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB).
   * Configure o **nome** como **secure-file-storage-cloudant**.
   * Configure o local.
   * Use o mesmo **grupo de recursos** dos serviços anteriores.
   * Configure **Métodos de autenticação disponíveis** como **Usar somente o IAM**.
2. Em **Credenciais de serviço**, crie *Nova credencial*.
   * Configure o **nome** como **secure-file-storage-cloudant-acckey**.
   * Configure **Função** como **Gerenciador**
   * Mantenha os valores padrão para os campos *Opcionais*
   * **Incluir**.
3. Anote as credenciais clicando em **Visualizar credenciais**. Você precisará deles em uma etapa posterior.
4. Em **Gerenciar**, ative o painel do Cloudant.
5. Clique em **Criar banco de dados** para criar um banco de dados denominado **secure-file-storage-metadata**.

### Autenticar usuários

Com o {{site.data.keyword.appid_short}}, é possível proteger recursos e incluir a autenticação em seus aplicativos. O {{site.data.keyword.appid_short}} [integra-se](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth) ao {{site.data.keyword.containershort_notm}} para autenticar usuários que acessam os aplicativos implementados no cluster.

1. Crie uma instância do [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID).
   * Configure o **Nome do serviço** como **secure-file-storage-appid**.
   * Use o mesmo **local** e **grupo de recursos** como para os serviços anteriores.
2. Em **Provedores de identidade/Gerenciar**, na guia **Configurações de autenticação**, inclua uma **URL de redirecionamento da web** apontando para o domínio que será usado para o aplicativo. Por exemplo, se o subdomínio do Ingresso do cluster for
`<cluster-name>.us-south.containers.appdomain.cloud`, a URL de redirecionamento será `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback`. O {{site.data.keyword.appid_short}} requer a URL de redirecionamento da web seja **https**. É possível visualizar seu subdomínio do Ingresso no painel do cluster ou com `ibmcloud ks cluster-get <cluster-name>`.

É necessário customizar os provedores de identidade usados, assim como o login e a experiência de gerenciamento de usuários no painel do {{site.data.keyword.appid_short}}. Este tutorial usa os padrões para simplicidade. Para um ambiente de produção, considere usar as regras de Autenticação de Diversos Fatores (MFA) e de senha avançada.
{: tip}

## Implementar o app

Todos os serviços foram configurados. Nesta seção, você implementará o aplicativo tutorial para o cluster.

### Obtenha o código

1. Obtenha o código do aplicativo:
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. Acesse o diretório **secure-file-storage**:
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### Construir a imagem do Docker

1. [Construa a imagem do Docker](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating) no {{site.data.keyword.registryshort_notm}}.
   - Localize o terminal de registro com `ibmcloud cr info`, tal como **us**.icr.io ou **uk**.icr.io.
   - Crie um espaço de nomes para armazenar a imagem do contêiner.
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - Use **secure-file-storage** como o nome da imagem.

   ```sh
   ibmcloud cr build -t <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### Preencha as credenciais e definições de configuração

1. Copie `credentials.template.env` para `credentials.env`:
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. Edite `credentials.env` e preencha os espaços em branco com estes valores:
   * o terminal regional do serviço {{site.data.keyword.cos_short}}, o nome do depósito, as credenciais criadas para **secure-file-storage-cos**
   * e as credenciais para **secure-file-storage-cloudant**.
3. Copie `secure-file-storage.template.yaml` para `secure-file-storage.yaml`:
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. Edite `secure-file-storage.yaml` e substitua os itens temporários (`$IMAGE_PULL_SECRET`, `$REGISTRY_URL`, `$REGISTRY_NAMESPACE`, `$IMAGE_NAME`, `$TARGET_NAMESPACE`, `$INGRESS_SUBDOMAIN`, `$INGRESS_SECRET`) pelos valores corretos. Como exemplo, supondo que o aplicativo seja implementado no namespace do Kubernetes _padrão_:

| Variável | Valor | Descrição |
| -------- | ----- | ----------- |
| `$IMAGE_PULL_SECRET` | Manter as linhas comentadas no .yaml | Um segredo para acessar o registro.  |
| `$REGISTRY_URL` | *us.icr.io* | O registro no qual a imagem foi construída na seção anterior. |
| `$REGISTRY_NAMESPACE` | *secure-file-storage-namespace* | O namespace do registro no qual a imagem foi construída na seção anterior. |
| `$IMAGE_NAME` | *secure-file-storage* | O nome da imagem do Docker. |
| `$TARGET_NAMESPACE` | *default* | o namespace do Kubernetes no qual o app será enviado por push. |
| `$INGRESS_SUBDOMAIN` | *secure-file-stora-123456.us-south.containers.appdomain.cloud* | Recupere na página de visão geral do cluster ou com `ibmcloud ks cluster-get secure-file-storage-cluster`. |
| `$INGRESS_SECRET` | *secure-file-stora-123456* | Recupere na página de visão geral do cluster ou com `ibmcloud ks cluster-get secure-file-storage-cluster`. |

`$IMAGE_PULL_SECRET` será necessário somente se você desejar usar outro namespace do Kubernetes do que o padrão. Isso requer configuração adicional do Kubernetes (por exemplo, [criando um segredo de registro do Docker no novo namespace](https://{DomainName}/docs/containers?topic=containers-images#other)).
{: tip}

### Implementar no cluster

1. Recupere a configuração de cluster e configure a variável de ambiente KUBECONFIG.
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. Crie o segredo usado pelo aplicativo para obter credenciais de serviço:
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. Ligue a instância de serviço do {{site.data.keyword.appid_short_notm}} ao cluster.
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
   Se você tiver vários serviços com o mesmo nome, o comando falhará. É necessário passar o GUID do serviço em vez de seu nome. Para localizar o GUID de um serviço, use o `ibmcloud resource service-instance secure-file-storage-appid`.
   {: tip}
4. Implemente o app.
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## Testar o aplicativo

O aplicativo pode ser acessado em `https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/`.

1. Acesse a página inicial do aplicativo. Você será redirecionado para a página de login padrão do {{site.data.keyword.appid_short_notm}}.
2. Inscreva-se para uma nova conta com um endereço de e-mail válido.
3. Aguarde o e-mail em sua caixa de entrada para verificar a conta.
4. Efetue login.
5. Escolha um arquivo para fazer upload. Clique em  ** Upload **.
6. Use a ação **Compartilhar** em um arquivo para gerar uma URL pré-assinada que pode ser compartilhada com outras pessoas para acessar o arquivo. O link está configurado para expirar após 5 minutos.

Os usuários autenticados têm seus próprios espaços para armazenar arquivos. Embora eles não possam ver cada um dos outros arquivos, eles podem gerar URLs pré-assinadas para conceder acesso provisório a um arquivo específico.

É possível localizar mais detalhes sobre o aplicativo no [repositório de código-fonte](https://github.com/IBM-Cloud/secure-file-storage).

## Revisar eventos de segurança
Agora que o aplicativo e seus serviços foram implementados com êxito, é possível revisar os eventos de segurança gerados por esse processo. Todos os eventos estão centralmente disponíveis na instância do {{site.data.keyword.cloudaccesstrailshort}} e podem ser acessados por meio de [IU gráfica (Kibana), CLI ou API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status).

1. Na [Lista de recursos do {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), localize a instância do {{site.data.keyword.cloudaccesstrailshort}} **secure-file-storage-activity-tracker** e abra seu painel.
2. Por padrão, a guia **Gerenciar** mostra **Logs de espaço**. Alterne para **Logs de conta**, clicando no seletor ao lado de **Visualizar logs**. Ele deve exibir vários eventos.
3. Clique em **Visualizar no Kibana** para abrir o visualizador de eventos integral.
4. Revise os detalhes do evento clicando em **Descobrir**.
5. Em **Campos disponíveis**, inclua **action_str** e **initiator.name_str**.
6. Expanda as entradas interessantes clicando no ícone de triângulo, em seguida, escolhendo uma tabela ou formato JSON.

É possível mudar as configurações para a atualização automática e o intervalo de tempo exibido e, dessa forma, mudar como e quais dados são analisados.
{: tip}

## Opcional: usar um domínio customizado e criptografar o tráfego de rede
Por padrão, o aplicativo é acessível em um nome de host genérico em um subdomínio de `containers.appdomain.cloud`. No entanto, também é possível usar um domínio customizado com o app implementado. Para obter suporte continuado de **https**, o acesso com tráfego de rede criptografado, um certificado para o nome do host desejado ou um certificado curinga precisa ser fornecido. Na seção a seguir, você fará upload de um certificado existente para o {{site.data.keyword.cloudcerts_short}} e o implementará no cluster. Você também atualizará a configuração do app para usar o domínio customizado.

Um exemplo de como obter um certificado de [Let's Encrypt](https://letsencrypt.org/) é descrito no [blog do {{site.data.keyword.cloud}}](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/) a seguir.
{: tip}

1. Criar uma instância do  [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)
   * Configure o nome como **secure-file-storage-certmgr**.
   * Use o mesmo **local** e **grupo de recursos** como para os outros serviços.
2. Clique em **Importar certificado** para importar seu certificado.
   * Configure o nome como **SecFileStorage** e a descrição como **Tutorial de certificado para segurança e2e**.
   * Faça upload do arquivo de certificado usando o botão **Procurar**.
   * Clique em **Importar** para concluir o processo de importação.
3. Localize a entrada para o certificado importado e expanda-a
   * Verifique a entrada do certificado, por exemplo, se o nome de domínio corresponde ao seu domínio customizado. Se você transferiu por upload um certificado curinga, um asterisco será incluído no nome de domínio.
   * Clique no símbolo de **cópia** ao lado do **crn** do certificado.
4. Alterne para a linha de comandos para implementar as informações de certificado como um segredo para o cluster. Execute o comando a seguir depois de copiar no crn da etapa anterior.
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <the copied crn from previous step>
   ```
   {: codeblock}
   Verifique se o cluster sabe sobre o certificado, executando o comando a seguir.
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. Edite o arquivo `secure-file-storage.yaml`.
   * Localize a seção para **Ingresso**.
   * Remova o comentário e edite as linhas que cobrem domínios customizados e preencha seu domínio e nome do host.
   A entrada CNAME para seu domínio customizado precisa apontar para o cluster. Verifique este [guia sobre mapeamento de domínios customizados](https://{DomainName}/docs/containers?topic=containers-ingress#private_3) na documentação para obter detalhes.
   {: tip}
6. Aplique as mudanças na configuração ao implementado:
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. Alterne de volta para o navegador. Na [Lista de recursos do {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), localize o serviço {{site.data.keyword.appid_short}} criado e configurado anteriormente e ative seu painel de gerenciamento.
   * Acesse **Gerenciar** sob os **Provedores de identidade**e, em seguida, **Configurações**.
   * No formulário **Incluir URLs de redirecionamento da web**, inclua `https://secure-file-storage.<your custom domain>/appid_callback` como outra URL.
8. Tudo deve estar em seu lugar agora. Teste o app acessando-o em seu domínio customizado configurado `https://secure-file-storage.<your custom domain>`.

## Expandir o tutorial

A segurança nunca está concluída. Tente as sugestões a seguir para aprimorar a segurança de seu aplicativo.

* Use [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) para executar varreduras de código estático e dinâmico
* Assegure-se de que somente código de qualidade seja liberado usando políticas e regras com o [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)

## Remover recursos
{:removeresources}

Para remover o recurso, exclua o contêiner implementado e, em seguida, os serviços provisionados.

1. Exclua o contêiner implementado:
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. Exclua os segredos para a implementação:
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. Remova a imagem do Docker do registro do contêiner:
   ```sh
   ibmcloud cr image-rm <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. Na [Lista de recursos do {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), localize os recursos que foram criados para este tutorial. Use a caixa de procura e **secure-file-storage** como padrão. Exclua cada um dos serviços clicando no menu de contexto ao lado de cada serviço e escolhendo **Excluir serviço**. Observe que o serviço {{site.data.keyword.keymanagementserviceshort}} poderá ser removido somente após a chave ter sido excluída. Clique na instância de serviço para chegar ao painel relacionado e para excluir a chave.

Se você compartilhar uma conta com outros usuários, certifique-se sempre de excluir somente seus próprios recursos.
{: tip}

## Conteúdo relacionado
{:related}

* [{{site.data.keyword.security-advisor_short}} documentação](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [Segurança para proteger e monitorar seus apps em nuvem](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [Segurança da plataforma {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/overview?topic=overview-security#security)
* [Segurança no IBM Cloud](https://www.ibm.com/cloud/security)
* [Tutorial: melhores práticas para organizar usuários, equipes, aplicativos](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [Apps seguros no IBM Cloud com certificados curingas](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
