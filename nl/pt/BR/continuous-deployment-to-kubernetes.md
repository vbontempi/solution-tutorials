---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# Implementação contínua no Kubernetes
{: #continuous-deployment-to-kubernetes}

Este tutorial conduz você pelo processo de configuração de uma integração contínua e um pipeline de entrega para aplicativos conteinerizados em execução no {{site.data.keyword.containershort_notm}}. Você aprenderá como configurar o controle de fonte, em seguida, construir, testar e implementar o código para diferentes estágios de implementação. Em seguida, você incluirá integrações em outros serviços, como scanners de segurança, notificações de Slack e análise de dados.

{:shortdesc}

## Objetivos
{: #objectives}

* Criar clusters Kubernetes de desenvolvimento e produção.
* Criar um aplicativo iniciador, executá-lo localmente e enviá-lo por push para um repositório Git.
* Configurar o pipeline de entrega do DevOps para se conectar ao seu repositório Git, construir e implementar o app iniciador para os clusters de desenvolvimento/produção.
* Explore e integre o app para usar scanners de segurança, notificações de Slack e análise de dados.

## Serviços usados
{: #services}

Este tutorial usa os serviços do {{site.data.keyword.Bluemix_notm}} a seguir:

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**Atenção:** este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

![](images/solution21/Architecture.png)

1. Enviar por push o código para um repositório Git privado.
2. O pipeline seleciona mudanças no Git e constrói a imagem do contêiner.
3. Imagem de contêiner transferida por upload para o registro implementado em um cluster Kubernetes de desenvolvimento.
4. Valide as mudanças e implemente no cluster de produção.
5. Configuração de notificações do Slack para atividades de implementação.


## Antes de Começar
{: #prereq}

* [Instale o {{site.data.keyword.dev_cli_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - Script para instalar o docker, kubectl, helm, ibmcloud cli e plug-ins necessários.
* [Configure a CLI {{site.data.keyword.registrylong_notm}} e o espaço de nomes de registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Entenda os conceitos básicos de Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Criar o cluster Kubernetes de desenvolvimento
{: #create_kube_cluster}

O {{site.data.keyword.containershort_notm}} fornece ferramentas poderosas, combinando s tecnologias Docker e Kubernetes, uma experiência intuitiva do usuário e a segurança e o isolamento integrados para automatizar a implementação, a operação, o ajuste de escala e o monitoramento de apps conteinerizados em um cluster de hosts de cálculo.

Para concluir este tutorial, seria necessário selecionar o cluster **Pago** do tipo **Padrão**. Seria necessário configurar dois clusters, um para desenvolvimento e um para produção.
{: shortdesc}

1. Crie o primeiro cluster Kubernetes de desenvolvimento por meio do [catálogo do {{site.data.keyword.Bluemix}}](https://{DomainName}/containers-kubernetes/launch). Posteriormente, será necessário repetir estas etapas e criar um cluster de produção.

   Para facilitar o uso, verifique os detalhes de configuração, como o número de CPUs, a memória e o número de nós do trabalhador obtidos.
   {:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. Selecione o **Tipo de cluster** e clique em **Criar cluster** para provisionar um cluster Kubernetes. O menor **Tipo de máquina** com 2 **CPUs**, 4 **GB de RAM** e 1 **Nó do trabalhador** é suficiente para este tutorial. Todas as outras opções podem ser deixadas para seus padrões.
3. Verifique o status do **Cluster** e dos **Nós do trabalhador** e aguarde que eles estejam **prontos**.

**Nota:** não continue até que seus trabalhadores estejam prontos.

## Criar um cluster Starter
{: #create_application}

O {{site.data.keyword.containershort_notm}} oferece uma seleção de aplicativos iniciadores, os quais podem ser criados usando o comando `ibmcloud dev create` ou o console da web. Neste tutorial, usaremos o console da web. O aplicativo iniciador reduz significativamente o tempo de desenvolvimento, gerando iniciadores de aplicativo com todo o código necessário de modelo, construção e configuração para que seja possível iniciar a codificação da lógica de negócios mais rapidamente.

1. No [console do {{site.data.keyword.cloud_notm}}](https://{DomainName}), use a opção de menu do lado esquerdo e selecione [Apps da web](https://{DomainName}/developer/appservice/dashboard).
2. Na seção **Iniciar por meio da web**, clique no botão **Introdução**.
3. Selecione o ladrilho `Node.js Web App with Express.js` e, em seguida, `Create app` para criar um aplicativo iniciador Node.js.
4. Insira o **nome** `mynodestarter`. Em seguida, clique em **Criar**.

## Configurar o pipeline de entrega do DevOps
{: #create_devops}

1. Agora que você criou com êxito o aplicativo iniciador, sob o botão **Implementar seu app**, clique no botão **Implementar no Cloud**.
2. Selecionando o método de implementação do Cluster Kubernetes, selecione o cluster criado anteriormente e, em seguida, clique em **Criar**. Isso criará uma cadeia de ferramentas e um pipeline de entrega para você. ![](images/solution21/BindCluster.png)
3. Depois que o pipeline for criado, clique em **Visualizar cadeia de ferramentas** e, em seguida, em **Pipeline de entrega** para visualizar o pipeline. ![](images/solution21/Delivery-pipeline.png)
4. Depois que os estágios de implementação forem concluídos, clique em **Visualizar logs e histórico** para ver os logs.
5. Visite a URL exibida para acessar o aplicativo (`http://worker-public-ip:portnumber/`). ![](images/solution21/Logs.png)
Pronto, você usou a IU do serviço de app para criar os aplicativos iniciadores e configurou o pipeline para construir e implementar o aplicativo em seu cluster.

## Clonar, construir e executar o aplicativo localmente
{: #cloneandbuildapp}

Nesta seção, você usará o app iniciador criado na seção anterior, o clonará em sua máquina local, modificará o código e, em seguida, o construirá/executará localmente.
{: shortdesc}

### Clonar o aplicativo
1. Na visão geral de Cadeia de ferramentas, selecione o ladrilho **Git** em **Código**. Você será redirecionado para sua página de repositório git na qual é possível clonar o repositório.![HelloWorld](images/solution21/DevOps_Toolchain.png)

2. Se você ainda não tiver configurado as chaves SSH, deverá ver uma barra de notificação na parte superior com instruções. Siga as etapas abrindo o link **incluir uma chave SSH** em uma nova guia ou, se você desejar usar HTTPS em vez de SSH, siga as etapas clicando em **criar um token de acesso pessoal**. Lembre-se de salvar a chave ou o token para referência futura.
3. Selecione SSH ou HTTPS e copie a URL do git. Clone a origem para sua máquina local. Se for solicitado que você forneça um nome do usuário, forneça seu nome do usuário do git. Para a senha, use uma **chave SSH** existente ou um **token de acesso pessoal** ou aquele criado na etapa anterior.

   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   {: codeblock}

4. Abra o repositório clonado em um IDE de sua escolha e navegue para `public/index.html`. Atualize o código tentando mudar "Congratulations!" para outra coisa e salve o arquivo.

### Construir o aplicativo localmente
É possível construir e executar o aplicativo, como você normalmente usaria `mvn` para desenvolvimento local java ou `npm` para desenvolvimento de nó.  Também é possível construir uma imagem do docker e executar o aplicativo em um contêiner para assegurar execução consistente localmente e na nuvem. Use as etapas a seguir para construir sua imagem do docker.
{: shortdesc}

1. Assegure-se de que o mecanismo do Docker local tenha sido iniciado, para verificar, execute o comando abaixo:
   ```
   docker ps
   ```
   {: codeblock}
2. Navegue para o diretório de projeto gerado clonado.
   ```
   cd <project name>
   ```
   {: codeblock}
3. Construa o aplicativo localmente.
   ```
   ibmcloud dev build
   ```
   {: codeblock}

   Isso pode levar alguns minutos para ser executado, pois todas as dependências do aplicativo são transferidas por download e uma imagem do Docker, que contém seu aplicativo e todo o ambiente necessário, é construída.

### Executar o aplicativo localmente

1. Execute o contêiner.
   ```
   ibmcloud dev run
   ```
   {: codeblock}

   Isso usa o mecanismo de Docker local para executar a imagem do docker que você construiu na etapa anterior.
2. Depois que seu contêiner for iniciado, acesse http://localhost:3000/
   ![](images/solution21/node_starter_localhost.png)

## Enviar por push o aplicativo para seu repositório Git

Nesta seção, você confirmará sua mudança em seu repositório Git. O pipeline selecionará a confirmação e enviará por push as mudanças para seu cluster automaticamente.
1. Em sua janela do terminal, certifique-se de que você esteja dentro do repositório clonado.
2. Envie por push a mudança para seu repositório com três etapas simples: inclua, confirme e envie por push.
   ```bash
   git add public/index.html git commit -m "my first changes" git push origin master
   ```
   {: codeblock}

3. Acesse a cadeia de ferramentas que você criou anteriormente e clique no bloco **Delivery Pipeline**.
4. Confirme se você vê o estágio **BUILD** e **DEPLOY**.
   ![](images/solution21/Delivery-pipeline.png)
5. Aguarde até que o estágio **DEPLOY** seja concluído.
6. Clique na **url** do aplicativo sob o resultado da Última execução para visualizar suas mudanças em tempo real.

Se você não vir a atualização do aplicativo, verifique os logs dos estágios DEPLOY e BUILD de seu pipeline.

## Segurança usando o Vulnerability Advisor
{: #vulnerability_advisor}

Nesta etapa, você explorará o [Vulnerability Advisor](https://{DomainName}/docs/services/va?topic=va-va_index#va_index). O Vulnerability Advisor é usado para verificar o status de segurança de imagens de contêiner antes da implementação e ele também verifica o status de contêineres em execução.

1. Acesse a cadeia de ferramentas que você criou anteriormente e clique no bloco **Delivery Pipeline**.
1. Clique em **Incluir estágio** e mude o MyStage para **Estágio Validate** e, em seguida, clique em JOBS > **ADD JOB**.

   1. Selecione **Teste** como o Tipo de tarefa e mude **Teste** para **Vulnerability Advisor** na caixa.
   1. Em Tipo de testador, selecione **Vulnerability Advisor**. Todos os outros campos devem ser preenchidos automaticamente.
      O namespace do Container Registry deve ser igual ao mencionado em **Estágio Build** dessa cadeia de ferramentas.
      {:tip}
   1. Edite a seção **Script de teste** e substitua `SAFE\ to\ deploy` na última linha por `NO\ ISSUES`
   1. Salvar o estágio
1. Arraste e mova o **Estágio Validate** para o meio, em seguida, clique em **Executar** ![](images/solution21/run.png) no **Estágio Validate**. Você verá que o **Estágio Validate** falha.

   ![](images/solution21/toolchain.png)

1. Clique em **Visualizar logs e histórico** para ver a avaliação de vulnerabilidade. O final do log diz:

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   É possível ver as avaliações detalhadas de vulnerabilidade de todos os repositórios varridos [aqui](https://{DomainName}/containers-kubernetes/registry/private)
   {:tip}

   O estágio pode falhar dizendo que a imagem *não foi varrida* se a varredura de vulnerabilidades levar mais de 3 minutos. Esse tempo limite pode ser mudado editando o script da tarefa e aumentando o número de iterações para aguardar os resultados da varredura.
   {:tip}

1. Vamos corrigir as vulnerabilidades, seguindo a ação corretiva. Abra o repositório clonado em um IDE ou selecione o ladrilho IDE da web Eclipse Orion, abra `Dockerfile` e inclua o comando abaixo após `EXPOSE 3000`
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. Confirme e envie por push as mudanças. Isso deve acionar a cadeia de ferramentas e corrigir o **Estágio Validate**.

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## Criar cluster Kubernetes de produção

{: #deploytoproduction}

Nesta seção, você concluirá o pipeline de implementação implementando o aplicativo Kubernetes em ambientes de desenvolvimento e produção, respectivamente. Idealmente, desejamos configurar uma implementação automática para o ambiente de desenvolvimento e uma implementação manual para o ambiente de produção. Antes de fazer isso, vamos explorar as duas maneiras nas quais é possível entregar isso. É possível usar um cluster para o ambiente de desenvolvimento e produção. No entanto, é recomendável ter dois clusters separados, um para desenvolvimento e um para produção. Vamos explorar a configuração de um segundo cluster para produção.
{: shortdesc}

1. Siga as instruções na seção [Criar cluster Kubernetes de desenvolvimento](#create_kube_cluster) e crie um novo cluster. Nomeie esse cluster como `prod-cluster`.
2. Acesse a cadeia de ferramentas que você criou anteriormente e clique no bloco **Delivery Pipeline**.
3. Renomeie o **Estágio Deploy** para `Deploy dev`, é possível fazer isso clicando em Ícone de configurações > **Configurar estágio**.    ![](images/solution21/deploy_stage.png)
4. Clone o estágio **Deploy dev** (ícone de configurações > Clonar estágio) e nomeie o estágio clonado como `Deploy prod`.
5. Mude o **acionador de estágio** para `Run jobs only when this stage is run manually`. ![](images/solution21/prod-stage.png)
6. Na guia **Tarefa**, mude o nome do cluster para o cluster recém-criado e, em seguida, **Salve** o estágio.
7. Agora você deve ter a configuração de implementação completa, para implementar de desenvolvimento para produção, deve-se executar manualmente o estágio `Deploy prod` para implementar na produção. ![](images/solution21/full-deploy.png)

Pronto, você agora criou um cluster de produção e configurou o pipeline para enviar por push as atualizações para seu cluster de produção manualmente. Este é um estágio de processo de simplificação sobre um cenário mais avançado no qual você incluiria testes de unidade e testes de integração como parte do pipeline.

## Configurar notificações do Slack
{: #setup_slack}

1. Volte para visualizar a lista de [cadeias de ferramentas](https://{DomainName}/devops/toolchains) e selecione sua cadeia de ferramentas, em seguida, clique em **Incluir uma ferramenta**.
2. Procure o Slack na caixa de procura ou role para baixo para ver **Slack**. Clique para ver a página de configuração.
    ![](images/solution21/configure_slack.png)
3. Para **Webhook do Slack**, siga as etapas neste [link](https://my.slack.com/services/new/incoming-webhook/). É necessário efetuar login com suas credenciais do Slack e fornecer um nome de canal existente ou criar um novo.
4. Depois que a integração de webhook Recebida for incluída, copie a **URL do webhook** e cole-a sob **Webhook do Slack**.
5. O canal do Slack é o nome do canal fornecido durante a criação de uma integração de webhook acima.
6. **Nome da equipe do Slack** é o team-name (primeira parte) de team-name.slack.com. Por exemplo, kube é o nome da equipe em kube.slack.com
7. Clique em
**Criar integração**. Um novo ladrilho será incluído em sua cadeia de ferramentas.
   ![](images/solution21/toolchain_slack.png)
8. De agora em diante, sempre que a sua cadeia de ferramentas for executada, você deverá ver as notificações do Slack no canal configurado.
   ![](images/solution21/slack_channel.png)

## Remover recursos
{: #removeresources}

Nesta etapa, você limpará os recursos para remover o que criou acima.

- Exclua o repositório Git.
- Exclua a cadeia de ferramentas.
- Exclua os dois clusters.
- Exclua o canal Slack.

## Expandir o tutorial
{: #expandTutorial}

Deseja aprender mais? Aqui estão algumas ideias do que pode ser feito em seguida:

- [Analise logs e monitore o funcionamento do aplicativo com LogDNA e Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Inclua um ambiente de teste e implemente-o em um 3o. cluster.
- Implemente o cluster de produção [em diversas localizações](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Aprimore seu pipeline com controles de qualidade adicionais e análise de dados usando o [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Conteúdo relacionado
{: #related}

* Guia da solução Kubernetes de ponta a ponta, [Movendo os apps baseados na VM para o Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes).
* [Segurança](https://{DomainName}/docs/containers?topic=containers-security#cluster) para o IBM Cloud Container Service.
* [Integrações](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations) de cadeia de ferramentas.
* Analise logs e monitore o funcionamento do aplicativo com [LogDNA e Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).


