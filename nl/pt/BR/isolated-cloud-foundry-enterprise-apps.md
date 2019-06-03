---
copyright:
  years: 2019
lastupdated: "2019-03-07"
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

# Apps Cloud Foundry Enterprise isolados
{: #isolated-cloud-foundry-enterprise-apps}

Com o {{site.data.keyword.cfee_full_notm}} (CFEE), é possível criar múltiplas plataformas Cloud Foundry isoladas e de classificação corporativa on demand. Com isso, você obtém uma instância privada do Cloud Foundry implementada em um cluster Kubernetes isolado. Diferentemente da nuvem pública, você terá controle total no ambiente: controle de acesso, capacidade, versão, uso de recurso e monitoramento. O {{site.data.keyword.cfee_full_notm}} fornece a velocidade e a inovação de uma plataforma como serviço com a propriedade de infraestrutura localizada na TI corporativa.

Um caso de uso para o {{site.data.keyword.cfee_full_notm}} é uma plataforma de inovação de propriedade corporativa. Você, como um desenvolvedor dentro de uma empresa, pode criar novos microsserviços ou migrar aplicativos anteriores para o CFEE. Os microsserviços podem, então, ser publicados para desenvolvedores adicionais usando o mercado do Cloud Foundry. Uma vez lá, outros desenvolvedores em sua instância do CFEE podem consumir serviços dentro do aplicativo, tal como é feito hoje na nuvem pública.

O tutorial o conduzirá pelo processo de criação e configuração de um {{site.data.keyword.cfee_full_notm}}, configuração de controle de acesso e implementação de apps e serviços. Você também revisará o relacionamento entre o CFEE e o [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) implementando um broker de serviço customizado que integra serviços customizados ao CFEE.

## Objetivos

{: #objectives}

- Comparar e contrastar o CFEE com o Cloud Foundry público
- Implementar apps e serviços no CFEE
- Entender o relacionamento entre o Cloud Foundry e o {{site.data.keyword.containershort_notm}}
- Investigar a rede básica do Cloud Foundry e do {{site.data.keyword.containershort_notm}}

## Serviços usados

{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura

{: #architecture}

![Arquitetura](./images/solution45-CFEE-apps/Architecture.png)

1. O administrador cria uma instância do CFEE e inclui usuários com acesso de desenvolvedor.
2. O desenvolvedor envia por push um app iniciador Node.js para o CFEE.
3. O app iniciador Node.js usa o [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) para armazenar dados.
4. O desenvolvedor inclui um novo serviço de "mensagem de boas-vindas".
5. O app iniciador Node.js liga o novo serviço por meio de um broker de serviço customizado.
6. O app iniciador Node.js exibe "bem-vindo" em diferentes idiomas do serviço.

## Pré-requisitos

{: #prereq}

- [{{site.data.keyword.cloud_notm}} CLI (interface da linha de comandos)](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [Cloud Foundry CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## Provisionar o {{site.data.keyword.cfee_full_notm}}

{:provision_cfee}

Nesta seção, você criará uma instância do {{site.data.keyword.cfee_full_notm}} implementada em nós do trabalhador do Kubernetes por meio do {{site.data.keyword.containershort_notm}}.

1. [Prepare sua conta do {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare) para assegurar a criação de recursos de infraestrutura necessários.
2. No catálogo do {{site.data.keyword.cloud_notm}}, crie uma instância de serviço do [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create).
3. Configure o CFEE fornecendo o seguinte:
   - Selecione o plano **Padrão**.
   - Insira um **Nome** para a instância de serviço.
   - Selecione um **Grupo de recursos** no qual o ambiente é criado. Você precisará de permissão para acessar pelo menos um grupo de recursos na conta para ser capaz de criar uma instância do CFEE.
   - Selecione uma **Geografia** e um **Local** nos quais a instância é implementada. Consulte a lista de [locais de fornecimento e data centers disponíveis](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets).
   - Selecione uma **Organização** e um **Espaço** do Cloud Foundry público nos quais o **{{site.data.keyword.composeForPostgreSQL}}** será implementado.
   - Selecione o **Número de células** para o ambiente do Cloud Foundry. Uma célula executa os aplicativos Diego e Cloud Foundry. Pelo menos **2** células são necessárias para aplicativos altamente disponíveis.
   - Selecione o **Tipo de máquina**, que determina o tamanho das células do Cloud Foundry (CPU e memória).
4. Revise a seção **Infraestrutura** para visualizar as propriedades do cluster Kubernetes que suporta o CFEE. O **Número de nós do trabalhador** é igual ao número de células mais 2. Dois dos nós do trabalhador do Kubernetes provisionados agem como o plano de controle do CFEE. O cluster Kubernetes no qual o ambiente é implementado aparecerá no painel [Clusters](https://{DomainName}/containers-kubernetes/clusters) do {{site.data.keyword.cloud_notm}}.
5. Clique no botão **Criar** para iniciar a implementação automatizada.

A implementação automatizada leva aproximadamente 90 a 120 minutos. Uma vez criada com êxito, você receberá um e-mail confirmando o fornecimento do CFEE e dos serviços de apoio.

### Criar organizações e espaços

Depois de ter criado o {{site.data.keyword.cfee_full_notm}}, consulte [criando organizações e espaços](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs) para obter informações sobre como estruturar o ambiente por meio de organizações e espaços. Os apps em um {{site.data.keyword.cfee_full_notm}} têm o escopo definido em espaços específicos. De forma semelhante, existe um espaço dentro de uma organização. Os membros de uma organização compartilham um plano de cota, apps, instâncias de serviços e domínios customizados.

Nota: o menu **Gerenciar > Conta > Organizações do Cloud Foundry** localizado no cabeçalho principal do {{site.data.keyword.cloud_notm}} é destinado exclusivamente para organizações públicas do {{site.data.keyword.cloud_notm}}. As organizações do CFEE são gerenciadas dentro da página **organizações** de uma instância do CFEE.

Siga as etapas abaixo para criar uma organização e um espaço do CFEE.

1. No [Painel do Cloud Foundry](https://{DomainName}/dashboard/cloudfoundry/overview), selecione **Ambientes** em **Empresa**.
2. Selecione sua instância do CFEE e, em seguida, selecione **Organizações**.
3. Clique no botão **Criar organizações**, forneça um `tutorial` como o **Nome da organização** e selecione um **Plano de cota**. Conclua clicando em **Incluir**.
4. Clique no `tutorial` da organização recém-criada e, em seguida, selecione a guia **Espaços** e clique no botão **Criar espaço**.
5. Forneça `dev` como um **Nome de espaço** e clique em **Incluir**.

### Incluir usuários em organizações e espaços

No CFEE, é possível designar funções que controlam o acesso de usuário, mas, para fazer isso, o usuário deve ser convidado para sua conta do {{site.data.keyword.cloud_notm}} na página **Identidade e acesso** sob **Gerenciar > Usuários** no cabeçalho {{site.data.keyword.cloud_notm}}.

Depois que o usuário tiver sido convidado, siga as etapas abaixo para incluir o usuário na organização `tutorial` criada, 

1. Selecione sua [instância do CFEE](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) e, em seguida, selecione **Organizações** novamente.
2. Selecione a organização `tutorial` criada na lista.
3. Clique na guia **Membros** para visualizar e incluir um novo usuário.
4. Clique no botão **Incluir membros**, procure o nome do usuário, selecione as **Funções de organização** apropriadas e clique em **Incluir**.

Mais sobre a inclusão de usuários em organizações e espaços do CFEE pode ser localizado [aqui](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users).

## Implementar, configurar e executar apps do CFEE

{:deploycfeeapps}

Nesta seção, você implementará um aplicativo Node.js no CFEE. Depois de implementado, você então ligará um {{site.data.keyword.cloudant_short_notm}} a ele e ativará a persistência de auditoria e de criação de log. O console Stratos também será incluído para gerenciar o aplicativo.

### Implementar o aplicativo no CFEE

1. Em seu terminal, clone o aplicativo de amostra [get-started-node](https://github.com/IBM-Cloud/get-started-node).
   ```sh
   git clone https://github.com/IBM-Cloud/get-iniciado-node
   ```
   {: codeblock}
2. Execute o app localmente para assegurar que ele seja construído e iniciado corretamente. Confirme acessando `http://localhost:3000/` em seu navegador.
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. Efetue login no {{site.data.keyword.cloud_notm}} e destine sua instância do CFEE. Um prompt interativo ajudará você a selecionar sua nova instância do CFEE. Como existem somente uma organização e um espaço do CFEE, eles serão os destinos padronizados. Será possível executar `ibmcloud target -o tutorial -s dev` se você tiver incluído mais de uma organização ou espaço.
   ```sh
   ibmcloud login ibmcloud target --cf
   ```
   {: codeblock}
4. Envie por push o app **get-started-node** para o CFEE.
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. O terminal de seu app será exibido na saída final ao lado da propriedade `routes`. Abra a URL em seu navegador para confirmar se o aplicativo está em execução.

### Criar e ligar o banco de dados Cloudant ao app

Para ligar os serviços do {{site.data.keyword.cloud_notm}} ao aplicativo **get-started-node**, você primeiro precisará criar um serviço em sua conta do {{site.data.keyword.cloud_notm}}.

1. Crie um serviço do [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant). Forneça o **nome do serviço** `cfee-cloudant` e escolha o mesmo local no qual a instância do CFEE foi criada.
2. Inclua a instância de serviço do {{site.data.keyword.cloudant_short_notm}} recém-criada no CFEE.
   1. Navegue de volta para a **Organização** `tutorial`. Clique na guia **Espaços** e selecione o espaço `dev`.
   2. Selecione a guia **Serviços** e clique no botão **Incluir serviço**.
   3. Digite `cfee-cloudant` na caixa de texto de procura e selecione o resultado. Conclua clicando em **Incluir**. O serviço está agora disponível para aplicativos CFEE; no entanto, ele ainda reside no {{site.data.keyword.cloud_notm}} público.
3. No menu overflow da instância de serviço mostrada, selecione **Ligar ao aplicativo**.
4. Selecione o aplicativo **GetStartedNode** que você enviou por push anteriormente e clique em **Remontar o aplicativo após a ligação**. Finalmente, clique no botão **Ligar**. Aguarde o aplicativo ser remontado. É possível verificar o progresso com o comando `ibmcloud app show GetStartedNode`.
5. Em seu navegador, acesse o aplicativo, inclua seu nome e pressione `enter`. Seu nome será incluído em um banco de dados {{site.data.keyword.cloudant_short_notm}}.
6. Confirme selecionando a instância do `tutorial` na lista na guia **Serviços**. Isso abrirá a página de detalhes da instância de serviço no {{site.data.keyword.cloud_notm}} público.
7. Clique em **Ativar o painel do Cloudant** e selecione o banco de dados `mydb`. Um documento JSON com seu nome deverá existir.

### Ativar a persistência de auditoria e criação de log

A auditoria permite que os administradores do CFEE rastreiem as atividades do Cloud Foundry, como login, criação de organizações e espaços, participação do usuário e designações de função, implementações de aplicativo, ligações de serviços e configuração de domínio. A auditoria é suportada por meio da integração com o serviço {{site.data.keyword.cloudaccesstrailshort}}.

Os logs do aplicativo Cloud Foundry podem ser armazenados pela integração do {{site.data.keyword.loganalysisshort_notm}}. A instância de serviço do {{site.data.keyword.loganalysisshort_notm}} selecionada por um administrador do CFEE é configurada automaticamente para receber e persistir eventos de criação de log do Cloud Foundry gerados por meio da instância do CFEE.

Para ativar a persistência de auditoria e de criação de log do CFEE, siga as [etapas aqui](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging).

### Instalar o Stratos Console para gerenciar o app

O Stratos Console é uma ferramenta baseada na web de software livre para trabalhar com o Cloud Foundry. O aplicativo Stratos Console pode ser instalado opcionalmente e usado em um ambiente do CFEE específico para gerenciar organizações, espaços e aplicativos.

Para instalar o aplicativo Stratos Console:

1. Abra a instância do CFEE na qual você deseja instalar o Stratos Console.
2. Na página **Visão geral**, clique em **Instalar o Stratos Console**. O botão fica visível somente para usuários com permissões de administrador ou de editor.
3. No diálogo Instalar o Stratos Console, selecione uma opção de instalação. É possível instalar o aplicativo Stratos Console no plano de controle do CFEE ou em uma das células. Selecione uma versão do Stratos Console e o número de instâncias do aplicativo a ser instalado. Se você instalar o aplicativo Stratos Console em uma célula, será solicitado que forneça a organização e o espaço no qual implementar o aplicativo.
4. Clique em **Instalar**.

O aplicativo leva cerca de 5 minutos para ser instalado. Depois que a instalação for concluída, um botão **Stratos Console** aparecerá no lugar do botão **Instalar o Stratos Console** na página de visão geral. Mais sobre o Stratos Console pode ser localizado [aqui](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app).

## O relacionamento entre o CFEE e o Kubernetes

Como uma plataforma de aplicativo, o CFEE é executado em alguma forma de infraestrutura virtual dedicada ou compartilhada. Por muitos anos, os desenvolvedores pensaram pouco sobre a plataforma Cloud Foundry subjacente porque a IBM a gerenciava para eles. Com o CFEE, você não é somente um desenvolvedor que está gravando aplicativos do Cloud Foundry, mas também um operador da plataforma Cloud Foundry. Isso é porque o CFEE é implementado em um cluster Kubernetes que você controla.

Embora os desenvolvedores do Cloud Foundry podem ser novos no Kubernetes, há muitos conceitos que eles compartilham. Como o Cloud Foundry, o Kubernetes isola aplicativos em contêineres, que são executados dentro de uma construção do Kubernetes chamada pod. Semelhante às instâncias do aplicativo, os pods podem ter múltiplas cópias (chamadas de conjuntos de réplicas) com balanceamento de carga do aplicativo fornecido pelo Kubernetes. O aplicativo `GetStartedNode` do Cloud Foundry que você implementou anteriormente é executado dentro do pod `diego-cell-0`. Para suportar a alta disponibilidade, outro pod `diego-cell-1` seria executado em um nó do trabalhador do Kubernetes separado. Como esses aplicativos do Cloud Foundry são executados "dentro do" Kubernetes, também é possível comunicar-se com outros microsserviços do Kubernetes usando a rede baseada no Kubernetes. As seções a seguir ajudarão a ilustrar os relacionamentos entre o CFEE e o Kubernetes em mais detalhes.

## Implementar um broker de serviço do Kubernetes

Nesta seção, você implementará um microsserviço no Kubernetes que age como um broker de serviço para o CFEE. Os [brokers de serviço](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md) fornecem detalhes sobre os serviços disponíveis, bem como suporte de ligação e fornecimento para o aplicativo do Cloud Foundry. Você usou um broker de serviço integrado do {{site.data.keyword.cloud_notm}} para incluir o serviço {{site.data.keyword.cloudant_short_notm}}. Agora você implementará e usará um customizado. Em seguida, você modificará o app `GetStartedNode` para usar o broker de serviço, que retornará uma mensagem de "Bem-vindo" em vários idiomas.

1. De volta ao seu terminal, clone os projetos que fornecem arquivos de implementação do Kubernetes e a implementação do broker de serviço.

   ```sh
   git clone https://github.com/IBM-Cloud/cfee-service-broker-kubernetes.git
   ```
   {: codeblock}

   ```sh
   cd cfee-service-broker-kubernetes
   ```
   {: codeblock}

   ```sh
   git clone https://github.com/IBM/sample-resource-service-brokers.git
   ```
   {: codeblock}

2. Construa e armazene a imagem do Docker que contém o broker de serviço no {{site.data.keyword.registryshort_notm}}. Use o comando `ibmcloud cr info` para recuperar manualmente a URL de registro ou automaticamente com o comando `export REGISTRY` abaixo. O comando `cr namespace-add` criará um namespace para armazenar a imagem do docker.

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   Crie um namespace chamado `cfee-tutorial`.

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build. -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   Edite o arquivo `./cfee-service-broker-kubernetes/deployment.yml`. Atualize o atributo `image` para refletir a URL de registro do contêiner.
3. Implemente a imagem de contêiner no cluster Kubernetes do CFEE. O cluster de seu CFEE existe no grupo de recursos `default`, que deve ser destinado, se ainda não estiver. Usando o nome de seu cluster, exporte a variável KUBECONFIG usando o comando `cluster-config`. Em seguida, crie a implementação.
   ```sh
   ibmcloud target -g default
   ```
   {: codeblock}

   ```sh
   ibmcloud ks clusters
   ```
   {: codeblock}

   ```sh
   $(ibmcloud ks cluster-config <your-cfee-cluster-name> --export)
   ```
   {: codeblock}

   ```sh
   kubectl apply -f deployment.yml
   ```
   {: codeblock}
4. Verifique se os pods têm o STATUS como `Running`. Pode levar alguns instantes para o Kubernetes puxar a imagem e iniciar os contêineres. Observe que você tem dois pods porque o `deployment.yml` solicitou 2 `replicas`.
   ```sh
   kubectl get pods
   ```
   {: codeblock}

## Verificar se o broker de serviço está implementado

Agora que você implementou o broker de serviço, confirme se ele funciona adequadamente. Você fará isso de várias maneiras: primeiro, usando o painel do Kubernetes, em seguida, acessando o broker por meio de um app do Cloud Foundry e, finalmente, ligando de fato um serviço por meio do broker.

### Visualizar seus pods no painel do Kubernetes

Esta seção confirmará se os artefatos do Kubernetes estão configurados usando o painel do {{site.data.keyword.containershort_notm}}

1. Na página [Clusters Kubernetes](https://{DomainName}/containers-kubernetes/clusters), acesse seu cluster do CFEE clicando no item de linha que inicia com o nome de seu serviço CFEE e que termina com **-cluster**.
2. Abra o **Painel do Kubernetes** clicando no botão correspondente.
3. Clique no link **Serviços** no menu esquerdo e selecione **tutorial-broker-service**. Esse serviço foi implementado quando você executou `kubectl apply` em etapas anteriores.
4. No painel resultante, observe o seguinte:
   - O serviço recebeu um endereço IP de sobreposição (172.x.x.x) que pode ser resolvido somente dentro do cluster Kubernetes.
   - O serviço tem dois terminais, que correspondem aos dois pods que têm os contêineres do broker de serviço em execução.

Tendo confirmado que o serviço está disponível e é um proxy para os pods do broker de serviço, é possível verificar se o broker responde com as informações sobre os serviços disponíveis.

É possível visualizar os artefatos relacionados do Cloud Foundry por meio do painel do Kubernetes. Para vê-los, clique em **Namespaces** para visualizar todos os namespaces com artefatos, incluindo o namespace do Cloud Foundry `cf`.
{: tip}

### Acessar o broker por meio de um contêiner do Cloud Foundry

Para demonstrar a comunicação do Cloud Foundry com o Kubernetes, você se conectará ao broker de serviço diretamente de um aplicativo do Cloud Foundry.

1. De volta ao seu terminal, confirme se você ainda está conectado à organização `tutorial` do CFEE e ao espaço `dev` usando `ibmcloud target`. Se necessário, destine novamente o CFEE.
   ```sh
   ibmcloud target --cf
   ```
   {: codeblock}
2. Por padrão, o SSH é desativado em espaços. Isso é diferente da nuvem pública, portanto, ative o SSH em seu espaço `dev` do CFEE.
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. Use o comando `kubectl` para mostrar o mesmo ClusterIP que você viu no painel do Kubenetes. Em seguida, use SSH para o aplicativo `GetStartedNode` e recupere dados do broker de serviço usando o endereço IP. Esteja ciente de que o último comando pode resultar em um erro, que a próxima etapa resolverá.
   ```sh
   kubectl get service tutorial-broker-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf ssh GetStartedNode
   ```
   {: codeblock}

   ```sh
   export CLUSTER_IP=<CLUSTER-IP address>
   ```
   {: codeblock}

   ```sh
   wget --user TestServiceBrokerUser --password TestServiceBrokerPassword -O- http://$CLUSTER_IP/v2/catalog
   ```
   {: codeblock}
4. É provável que você tenha recebido um erro **Conexão recusada**. Isso é devido aos [application security groups](https://docs.cloudfoundry.org/concepts/asg.html) padrão do CFEE. Um application security group (ASG) define o intervalo de IP permitido para o tráfego de egresso por meio de um contêiner do Cloud Foundry. Como o `GetStartedNode` existe fora do intervalo padrão, o erro ocorre. Saia da sessão SSH e faça do download do ASG `public_networks`.
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. Edite o arquivo `public_networks.json` e verifique se o endereço ClusterIP que está sendo usado está fora das regras existentes. Por exemplo, o intervalo `172.32.0.0-192.167.255.255` provavelmente não inclui o ClusterIP e precisa ser atualizado. Se, por exemplo, seu endereço ClusterIP for algo como este `172.21.107.63`, será necessário editar para que o arquivo tenha algo como este `172.0.0.0-255.255.255.255`.
6. Ajuste a regra `destination` do ASG para incluir o endereço IP do serviço do Kubernetes. Apare o arquivo para incluir somente os dados JSON, que iniciam e terminam com os colchetes. Em seguida, faça upload do novo ASG.
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. Repita a etapa 3, que agora deve ser bem-sucedida com dados do catálogo de mock. Conclua saindo da sessão SSH.
   ```sh
   exit
   ```
   {: codeblock}

### Registrar o broker de serviço com o CFEE

Para permitir que os desenvolvedores provisionem e liguem serviços por meio do broker de serviço, registre-o com o CFEE. Anteriormente, você trabalhou com o broker usando um endereço IP. No entanto, isso é problemático. Se o broker de serviço for reiniciado, ele receberá um novo endereço IP, que requerá a atualização do CFEE. Para abordar esse problema, você usará outro recurso do Kubernetes chamado KubeDNS que fornece um Nome Completo do Domínio (FQDN) para o broker de serviço.

1. Registre o broker de serviço com o CFEE usando o FQDN do serviço `tutorial-service-broker`. Novamente, essa rota é interna para o cluster Kubernetes do CFEE.
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. Em seguida, inclua os serviços oferecidos pelo broker. Como o broker de amostra tem somente um serviço mock, um único comando é necessário.
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. Em seu navegador, acesse sua instância do CFEE na página [**Ambientes**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) e navegue para `organizations -> spaces` e selecione seu espaço `dev`.
4. Selecione a guia **Serviços** e o botão **Criar serviço**.
5. Na caixa de texto de procura, procure **Teste**. O serviço mock **Testar o nome de exibição do broker de serviço do recurso do nó** do broker será exibido.
6. Selecione o serviço, clique no botão **Criar** e forneça o nome `welcome-service` para criar uma instância de serviço. Ficará claro na próxima seção por que ele é chamado welcome-service. Em seguida, ligue o serviço ao app `GetStartedNode` usando o item **Ligar ao aplicativo** no menu overflow.
7. Para visualizar o serviço ligado, execute o comando `cf env`. O aplicativo `GetStartedNode` pode agora alavancar os dados no objeto `credentials` semelhante a como ele usa dados do `cloudantNoSQLDB` atualmente.
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## Incluir o serviço em seu aplicativo

Até este ponto, você implementou um broker de serviço, mas não um serviço real. Embora o broker de serviço forneça dados de ligação para `GetStartedNode`, nenhuma funcionalidade real do serviço em si foi incluída. Para corrigir isso, um serviço mock foi criado, o qual fornece uma mensagem de "Bem-vindo" para `GetStartedNode` em vários idiomas. Você implementará esse serviço no CFEE e atualizará o broker e o aplicativo para usá-lo.

1. Envie por push a implementação de `welcome-service` para o CFEE para permitir que equipes de desenvolvimento adicionais a usem como uma API.
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. Acesse o serviço usando seu navegador. O `route` para o serviço será mostrado como parte da saída do comando `push`. A URL resultante será semelhante a `https://welcome.<your-cfee-cluster-domain>`. Atualize a página para ver idiomas alternativos.
3. Retorne para a pasta `sample-resource-service-brokers` e edite a implementação de amostra do Node.js `sample-resource-service-brokers/node-resource-service-broker/testresourceservicebroker.js` com a inclusão de url após a linha 854 com a inclusão da URL do cluster. 

   ```javascript
   // TODO - Do your actual work here
    
   var generatedUserid   = uuid();
   var generatedPassword = uuid();
    
   result = {
     credentials: {
       userid   : generatedUserid,
       password : generatedPassword,
       url: 'https://welcome.<your-cfee-cluster-domain>'
     }
   };
   ```
   {: codeblock}
4. Construa e implemente o broker de serviço atualizado. Isso assegurará que a propriedade de URL será fornecida para apps que ligam o serviço.
   ```sh
   cd ..
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build. -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   ```sh
   kubectl patch deployment tutorial-servicebroker-deployment -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"version\":\"2\"}}}}}"
   ```
   {: codeblock}
5. No terminal, navegue de volta para a pasta do aplicativo `get-started-node`. 
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. Edite o arquivo `get-started-node/server.js` em `get-stared-node` para incluir o middleware a seguir. Insira-o apenas antes da chamada `app.listen()`.
   ```javascript
   // Use the welcome service
   const request = require('request');
   const testService = appEnv.services['testnoderesourceservicebrokername'];

   if (testService) {
     const { credentials: { url} } = testService[0];
     app.get("/api/welcome", (req, res) => request(url, (e, r, b) => res.send(b)));
   } else {
     app.get("/api/welcome", (req, res) => res.send('Welcome'));
   }
   ```
   {: codeblock}
7. Refatore a página `get-started-node/views/index.html`. Mude `data-i18n="welcome"` para `id="welcome"` na tag `h1`. E, em seguida, inclua uma chamada para o serviço no final da função `$(document).ready()`.
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. Atualize o app `GetStartedNode`. Inclua a dependência de pacote `request` que foi incluída no `server.js`, religue o `welcome-service` para selecionar a nova propriedade `url` e, finalmente, envie por push o novo código do app.
   ```sh
   npm i request -S
   ```
   {: codeblock}

   ```sh
   ibmcloud cf unbind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf bind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}

Pronto! Agora visite o aplicativo e atualize a página várias vezes para ver a mensagem de boas-vindas em diferentes idiomas.

![Resposta do broker de serviço](./images/solution45-CFEE-apps/service-broker.png)

Se você continuar a ver somente **Welcome** e não outros idiomas, execute o comando `ibmcloud cf env GetStartedNode` e confirme se a `url` para o serviço está presente. Se não estiver, rastreie novamente suas etapas, atualize e reimplemente o broker. Se o valor `url` estiver presente, confirme se ele retorna uma mensagem em seu navegador. Em seguida, execute os comandos `unbind-service` e `bind-service` novamente seguidos por `ibmcloud cf restart GetStartedNode`.
{: tip}

Enquanto o serviço de boas-vindas usa o Cloud Foundry como sua implementação, você pode usar facilmente o Kubernetes. A principal diferença é que a URL para o serviço seria provavelmente `welcome-service.default.svc.cluster.local`. O uso da URL do KubeDNS incluiu o benefício adicional de manter o tráfego de rede para os serviços internos no cluster Kubernetes.

Com o {{site.data.keyword.cfee_full_notm}}, essas abordagens alternativas são agora possíveis.

## Implementar este tutorial de solução usando uma cadeia de ferramentas  

Opcionalmente, é possível implementar o tutorial completo de solução usando uma cadeia de ferramentas. Siga a [instrução da cadeia de ferramentas](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes) para implementar todos os acima usando uma cadeia de ferramentas. 

Nota: alguns pré-requisitos ao usar uma cadeia de ferramentas, deve-se ter criado uma instância do CFEE e criado uma organização do CFEE e um espaço do CFEE. Instruções detalhadas descritas no leia-me da [instrução da cadeia de ferramentas](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes). 

## Conteúdo relacionado
{:related}

* [Implementando apps em clusters Kubernetes](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Componentes e arquitetura do Cloud Foundry Diego](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [Broker de serviço do CFEE no Kubernetes com uma cadeia de ferramentas](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

