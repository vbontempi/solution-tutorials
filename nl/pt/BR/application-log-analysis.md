---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Analisar logs e monitorar o funcionamento do aplicativo com LogDNA e Sysdig
{: #application-log-analysis}

Este tutorial mostra como o serviço [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/) pode ser usado para configurar e acessar logs de um aplicativo do Kubernetes que é implementado no {{site.data.keyword.Bluemix_notm}}. Você implementará um aplicativo Python em um cluster provisionado no {{site.data.keyword.containerlong_notm}}, configurará um agente LogDNA, gerará diferentes níveis de logs do aplicativo e acessará logs de trabalhador, logs de pod ou logs de rede. Em seguida, você procurará, filtrará e visualizará esses logs por meio da IU da web do {{site.data.keyword.la_short}}.

Além disso, você também configurará o serviço [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/) e configurará o agente Sysdig para monitorar o desempenho e o funcionamento de seu aplicativo e seu cluster do {{site.data.keyword.containerlong_notm}}.
{:shortdesc}

## Objetivos
{: #objectives}
* Implementar um aplicativo em um cluster Kubernetes para gerar entradas de log.
* Acessar e analisar diferentes tipos de logs para solucionar problemas e priorizar problemas.
* Obter visibilidade operacional para o desempenho e o funcionamento de seu app e o cluster que está executando seu app.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

  ![](images/solution12/Architecture.png)

1. O usuário se conecta ao aplicativo e gera entradas de log.
1. O aplicativo é executado em um cluster Kubernetes por meio de uma imagem armazenada no {{site.data.keyword.registryshort_notm}}.
1. O usuário configurará o agente de serviço {{site.data.keyword.la_full_notm}} para acessar logs de nível de cluster e de aplicativo.
1. O usuário configurará o agente de serviço {{site.data.keyword.mon_full_notm}} para monitorar o funcionamento e o desempenho do cluster do {{site.data.keyword.containerlong_notm}} e também do app implementado no cluster.

## Pré-requisitos
{: #prereq}

* [Instale o {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - Script para instalar o docker, kubectl, helm, ibmcloud cli e plug-ins necessários.
* [Configure a CLI {{site.data.keyword.registrylong_notm}} e o espaço de nomes de registro](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Conceda permissões a um usuário para visualizar logs no LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna)
* [Conceda permissões a um usuário para visualizar métricas no Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig)

## Criar um cluster Kubernetes
{: #create_cluster}

O {{site.data.keyword.containershort_notm}} fornece um ambiente para implementar apps altamente disponíveis nos contêineres do Docker que são executados em clusters Kubernetes.

Ignore esta seção se você tiver um cluster **Padrão** existente e desejar reutilizar com este tutorial.
{: tip}

1. Crie **um novo Cluster** por meio do [Catálogo do {{site.data.keyword.Bluemix}}](https://{DomainName}/kubernetes/catalog/cluster/create) e escolha o cluster **Padrão**.
   O encaminhamento de log *não* está ativado para o cluster **Grátis**.
   {:tip}
1. Selecione um grupo de recursos e uma geografia.
1. Por conveniência, use o nome `mycluster` para ser consistente com este tutorial.
1. Selecione uma **Zona do trabalhador** e selecione o menor **Tipo de máquina** com 2 **CPUs** e 4 **GB de RAM**, pois isso é suficiente para este tutorial.
1. Selecione 1 **Nó do trabalhador** e deixe todas as outras opções configuradas como padrões. Clique em  ** Criar Cluster **.
1. Verifique o status de seu **Cluster** e **Nó do trabalhador** e aguarde que eles estejam **prontos**.

## Provisionar uma instância do {{site.data.keyword.la_short}}
{: #provision_logna_instance}

Os aplicativos implementados em um cluster do {{site.data.keyword.containerlong_notm}} no {{site.data.keyword.Bluemix_notm}} provavelmente gerarão algum nível de saída de diagnóstico, ou seja, logs. Como um desenvolvedor ou um operador, você pode desejar acessar e analisar diferentes tipos de logs, como logs de trabalhador, logs de pod, logs de app ou logs de rede para solucionar problemas e priorizar problemas.

Usando o serviço {{site.data.keyword.la_short}}, é possível agregar logs de várias origens e retê-los o tempo que for necessário. Isso permite analisar o "cenário geral" quando necessário e solucionar problemas de situações mais complexas.

Para provisionar um serviço {{site.data.keyword.la_short}},

1. Navegue para a página [observabilidade](https://{DomainName}/observe/) e, em **Criação de log**, clique em **Criar instância**.
1. Forneça um **Nome de serviço** exclusivo.
1. Escolha uma região/local e selecione um grupo de recursos.
1. Selecione **Procura de log de 7 dias** como seu plano e clique em **Criar**.

O serviço fornece um sistema de gerenciamento de log centralizado no qual os dados do log são hospedados no IBM Cloud.

## Implementar e configurar um app do Kubernetes para encaminhar logs
{: #deploy_configure_kubernetes_app}

O [código pronto para execução para o app de criação de log está localizado neste repositório GitHub](https://github.com/IBM-Cloud/application-log-analysis). O aplicativo é gravado usando [Django](https://www.djangoproject.com/), uma estrutura da web do lado do servidor Python popular. Clone ou faça download do repositório e, em seguida, implemente o app no {{site.data.keyword.containershort_notm}} no {{site.data.keyword.Bluemix_notm}}.

### Implementar o aplicativo Python

Em um terminal:
1. Clone o repositório GitHub:
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. Mude para o diretório do aplicativo
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. Construa uma imagem do Docker com o [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile) no {{site.data.keyword.registryshort_notm}}.
   - Localize o **Registro de contêiner** com `ibmcloud cr info`, como us.icr.io ou uk.icr.io.
   - Crie um espaço de nomes para armazenar a imagem do contêiner.
      ```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - Substitua `<CONTAINER_REGISTRY>` por seu valor do registro de contêiner e use **app-log-analysis** como o nome da imagem.
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - Substitua o valor **image** no arquivo `app-log-analysis.yaml` com a tag de imagem `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest`
1. Execute o comando a seguir para recuperar a configuração de cluster e configure a variável de ambiente `KUBECONFIG`:
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. Implemente o app:
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. Para acessar o aplicativo, você precisa do `public IP` do nó do trabalhador e o `NodePort`
   - Para IP público, execute o comando a seguir:
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - Para o NodePort que será de 5 dígitos (por exemplo, 3xxxx), execute o comando a seguir:
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   Agora é possível acessar o aplicativo em `http://worker-ip-address:portnumber`

### Configure o cluster para enviar logs para sua instância do LogDNA

Para configurar o cluster Kubernetes para enviar logs para a sua instância do {{site.data.keyword.la_full_notm}}, deve-se instalar um pod *logdna-agent* em cada nó do cluster. O agente do LogDNA lê os arquivos de log do pod no qual ele está instalado e encaminha os dados do log para a sua instância do LogDNA.

1. Navegue para a página [Observabilidade](https://{DomainName}/observe/) e clique em **Criação de log**.
1. Clique em **Editar recursos de log** ao lado do serviço que você criou anteriormente e selecione **Kubernetes**.
1. Copie e execute o primeiro comando em um terminal no qual você configurou a variável de ambiente `KUBECONFIG` para criar um segredo do kubernetes com a chave de ingestão do LogDNA para sua instância de serviço.
1. Copie e execute o segundo comando para implementar um agente LogDNA em cada nó do trabalhador de seu cluster Kubernetes. O agente LogDNA coleta logs com a extensão **.log* e arquivos sem extensão que estão armazenados no diretório */var/log* de seu pod. Por padrão, os logs são coletados de todos os namespaces, incluindo kube-system, e encaminhados automaticamente para o serviço {{site.data.keyword.la_full_notm}}.
1. Depois de configurar uma origem de log, ative a IU do LogDNA clicando em **Visualizar LogDNA**. Pode levar alguns minutos antes de você começar a ver os logs.

## Gerar e acessar logs do aplicativo
{: generate_application_logs}

Nesta seção, você gerará logs do aplicativo e os revisará no LogDNA.

### Gerar logs do aplicativo

O aplicativo implementado nas etapas anteriores permite que você registre uma mensagem em um nível de log escolhido. Os níveis de log disponíveis são **crítico**, **erro**, **aviso**, **informações** e **depuração**. A infraestrutura de criação de log do aplicativo é configurada para permitir somente entradas de log em um nível ou acima de um nível configurado para passar. Inicialmente, o nível do criador de logs é configurado como **warn**. Portanto, uma mensagem registrada em **info** com uma configuração do servidor de **warn** não seria mostrada na saída de diagnóstico.

Dê uma olhada no código no arquivo [**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py). O código contém instruções de **impressão**, além de chamadas para funções de **criador de logs**. As mensagens impressas são gravadas no fluxo **stdout** (saída regular, console do aplicativo/terminal), as mensagens do criador de logs aparecem no fluxo **stderr** (log de erro).

1. Visite o app da web em `http://worker-ip-address:portnumber`.
1. Gere várias entradas de log enviando mensagens em níveis diferentes. A IU permite mudar a configuração de criador de logs para o nível de log do servidor também. Mude o nível de log do lado do servidor no meio para torná-lo mais interessante. Por exemplo, é possível registrar um "500 erro interno do servidor" como **erro** ou "Esta é a minha primeira entrada de log" como **informações**.

### Acessar logs do aplicativo

É possível acessar o log específico do aplicativo na IU do LogDNA usando os filtros.

1. Na barra superior, clique em **Todos os apps**.
1. Em contêineres, verifique **app-log-analysis**. Uma nova visualização não salva é mostrada com logs do aplicativo de todos os níveis.
1. Para ver logs de níveis de log específicos, clique em **Todos os níveis** e selecione múltiplos níveis como Erro, informações, aviso, etc.,

## Procurar e filtrar logs
{: #search_filter_logs}

A IU do {{site.data.keyword.la_short}}, por padrão, mostra todas as entradas de log disponíveis (tudo). As entradas mais recentes são mostradas na parte inferior por meio de uma atualização automática.
Nesta seção, você modificará o que e o quanto é exibido e salvará isso como uma **Visualização** para uso futuro.

### Procurar logs

1. Na caixa de entrada **Procurar** localizada na parte inferior da página na IU do LogDNA,
   - é possível procurar linhas que contenham um texto específico como **"Esta é a minha primeira entrada de log"** ou **500 erro interno do servidor**.
   - ou um nível de log específico inserindo `level:info` em que level é um campo que aceita o valor de sequência.

   Para obter mais campos de procura e ajuda, clique no ícone de ajuda de sintaxe ao lado da caixa de entrada de procura
   {:tip}
1. Para ir para um intervalo de tempo específico, insira **5 minutos atrás** na caixa de entrada **Ir para o intervalo de tempo**. Clique no ícone ao lado da caixa de entrada para localizar os outros formatos de tempo dentro de seu período de retenção.
1. Para destacar os termos, clique no ícone **Alternar ferramentas do visualizador**.
1. Insira **erro** como seu termo de destaque na primeira caixa de entrada, **contêiner** como seu termo de destaque na segunda caixa de entrada e verifique as linhas destacadas com os termos.
1. Clique no ícone **Alternar linha de tempo** para ver as linhas com logs em um horário específico de um dia.

### Filtrar logs

É possível filtrar logs por tags, origens, apps ou níveis.

1. Na barra superior, clique em **Todas as tags** e marque a caixa de seleção **k8s** para ver os logs relacionados ao Kubernetes.
1. Clique em **Todas as origens** e selecione o nome do host (nó do trabalhador) que você está interessado em verificar os logs. Funciona bem se você tiver múltiplos nós do trabalhador em seu cluster.
1. Para verificar os logs de contêiner ou de arquivo, clique em **Todos os apps** e selecione as caixas de seleção que você está interessado em ver os logs.

### Criar uma visualização

As visualizações são atalhos salvos para um conjunto específico de filtros e consultas de procura.

Assim que procurar ou filtrar logs, você deverá ver **Visualização não salva** na barra superior. Para salvar isso como uma visualização:
1. Clique em **Todos os apps** e marque a caixa de seleção ao lado de **app-log-analysis**
1. Clique em **Visualização não salva** > clique em **salvar como nova visualização/alerta** e nomeiem a visualização como **app-log-analysis-view**. Deixe a **Categoria** como vazia.
1. Clique em **Salvar visualização** e a nova visualização deverá aparecer na área de janela esquerda mostrando os logs para o app.

### Visualizar logs com gráficos e detalhamentos

Nesta seção, você criará uma placa e, em seguida, incluirá um gráfico com um detalhamento para visualizar os dados no nível do app. Uma placa é uma coleção de gráficos e detalhamentos.

1. Na área de janela esquerda, clique no ícone **placa** (acima do ícone de configurações) > clique em **NOVA PLACA**.
1. Clique em **Editar** na barra superior e nomeie isso como **app-log-analysis-board**. Clique em **Salvar**.
1. Clique em **Incluir gráfico**:
   - Insira **app** como seu campo na primeira caixa de entrada e pressione Enter.
   - Escolha **app-log-analysis** como seu valor de campo.
   - Clique em **Incluir gráfico**.
1. Selecione **Contagens** como sua métrica para ver o número de linhas em cada intervalo nas últimas 24 horas.
1. Para incluir um detalhamento, clique na seta abaixo do gráfico:
   - Escolha **Histograma** como seu tipo de detalhamento.
   - Escolha **nível** como seu tipo de campo.
   - Clique em **Incluir detalhamento** para ver um detalhamento com todos os níveis que você registrou para o app.

## Incluir {{site.data.keyword.mon_full_notm}} e monitorar seu cluster
{: #monitor_cluster_sysdig}

A seguir, você incluirá {{site.data.keyword.mon_full_notm}} no aplicativo. O serviço verifica regularmente a disponibilidade e o tempo de resposta do app.

1. Navegue para a página [observabilidade](https://{DomainName}/observe/) e, em **Monitoramento**, clique em **Criar instância**.
1. Forneça um **Nome de serviço** exclusivo.
1. Escolha uma região/local e selecione um grupo de recursos.
1. Selecione **Camada graduada** como seu plano e clique em **Criar**.
1. Clique em **Editar recursos de log** ao lado do serviço que você criou anteriormente e selecione **Kubernetes**.
1. Copie e execute o comando em **Instalar o agente Sysdig em seu cluster** em um terminal no qual você configurou a variável de ambiente `KUBECONFIG` para implementar o agente Sysdig em seu cluster. Aguarde até que a implementação seja concluída.

### Configurar o {{site.data.keyword.mon_short}}

Para configurar o Sysdig para monitorar o funcionamento e o desempenho de seu cluster:
1. Clique em **Visualizar Sysdig** e você deverá ver a IU do monitor do sysdig. Na página de boas-vindas, clique em **Avançar**.
1. Escolha **Kubernetes** como seu método de instalação em configuração do ambiente.
1. Clique em **Acessar a próxima etapa** ao lado da mensagem de êxito da configuração do agente e clique em **Vamos iniciar** na próxima página.
1. Clique em **Avançar** e, em seguida, em **Concluir integração** para ver a guia `Explore` da IU do Sysdig.

### Monitorar seu cluster

Para verificar o funcionamento e o desempenho de seu app e cluster:
1. De volta ao aplicativo em execução em `http://worker-ip-address:portnumber`, gere várias entradas de log.
1. Expanda **mycluster** na área de janela esquerda > expanda namespace **padrão** > clique em **app-log-analysis-deployment** para ver a Contagem de solicitações, o Tempo de resposta, etc. no assistente do monitor do Sysdig.
1. Para verificar os códigos de resposta de solicitação de HTTP, clique na seta ao lado de **Funcionamento de pod do Kubernetes** na barra superior e selecione **HTTP** em **Aplicativos**. Mude o intervalo para **10 M** na barra inferior da IU do Sysdig.
1. Para monitorar a latência do aplicativo,
   - Na guia Explorar, selecione **Implementações e pods**.
   - Clique na seta ao lado de `HTTP` e, em seguida, selecione Métricas > Rede.
   - Selecione **net.http.request.time**.
   - Selecione Horário: **Soma** e Grupo: **Média**.
   - Clique em **Mais opções** e, em seguida, clique no ícone **Topologia**.
   - Clique em **Pronto** e clique duas vezes na caixa para expandir a visualização.
1. Para monitorar o namespace do Kubernetes no qual o aplicativo está em execução,
   - Na guia Explorar, selecione **Implementações e pods**.
   - Clique na seta ao lado de `net.http.request.time`.
   - Selecione Painéis padrão > Kubernetes.
   - Selecione Estado do Kubernetes > Visão geral do estado do Kubernetes.

### Criar um painel customizado

Junto com os painéis predefinidos, é possível criar seu próprio painel customizado para exibir as visualizações mais úteis/relevantes e as métricas para os contêineres que executam seu app em um único local. Cada painel é composto por uma série de painéis configurados para exibir dados específicos em um número de formatos diferentes.

Para criar um painel:
1. Clique em **Painéis** na área de janela esquerda > clique em **Incluir painel**.
1. Clique em **Painel em branco** > nomeie o painel como **Visão geral de solicitação de contêiner** > clique em **Criar painel**.
1. Selecione **Lista superior** como seu novo painel e nomeie o painel como **Tempo de solicitação por contêiner**
   - Em **Métricas**, digite **net.http.request.time**.
   - Escopo: clique em **Substituir escopo do painel** > selecione **container.image** > selecione **é** > selecione _a imagem do aplicativo_
   - Segmente por **container.id** e você deverá ver o tempo de solicitação de rede de cada contêiner.
   - Clique em **Salvar**.
1. Para incluir um novo painel, clique no ícone **mais** e selecione **Número (#)** como o tipo de painel
   - Em **Métricas**, digite **net.http.request.count** > mude a agregação de tempo de **Média (méd)** para **Soma**.
   - Escopo: clique em **Substituir escopo do painel** > selecione **container.image** > selecione **é** > selecione _a imagem do aplicativo_
   - Compare com **1 hora** atrás e você deverá ver a contagem de solicitações de rede de cada contêiner.
   - Clique em **Salvar**.
1. Para editar o escopo desse painel,
   - Clique em **Editar escopo** no painel de título.
   - Selecione/Digite **Kubernetes.cluster.name** na lista suspensa
   - Deixe o nome de exibição vazio e selecione **é**.
   - Selecione **mycluster** como o valor e clique em **Salvar**.

## Remover recursos
{: #remove_resource}

- Remova as instâncias do LogDNA e do Sysdig da página [Observabilidade](https://{DomainName}/observe).
- Exclua o cluster, incluindo nó do trabalhador, app e contêineres. Essa ação não pode ser desfeita
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## Expandir o tutorial
{: #expand_tutorial}

- Use o [serviço {{site.data.keyword.at_full}}](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started) para rastrear como os aplicativos interagem com os serviços do IBM Cloud.
- [Inclua alertas](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts) em sua visualização.
- [Exporte logs](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export) para um arquivo local.

## Conteúdo relacionado
{:related}
- [Reconfigurando a chave de ingestão usada por um cluster Kubernetes](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [Arquivando logs para o IBM Cloud Object Storage](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [Configurando alertas no Sysdig](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [Trabalhando com canais de notificação na IU do Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
