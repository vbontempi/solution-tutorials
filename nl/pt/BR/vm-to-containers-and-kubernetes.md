---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-15"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Movendo um app baseado em VM para o Kubernetes
{: #vm-to-containers-and-kubernetes}

Este tutorial conduz você no processo de movimentação de um app baseado em VM para um cluster Kubernetes usando o {{site.data.keyword.containershort_notm}}. O [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) fornece ferramentas poderosas, combinando s tecnologias Docker e Kubernetes, uma experiência intuitiva do usuário e a segurança e o isolamento integrados para automatizar a implementação, a operação, o ajuste de escala e o monitoramento de apps conteinerizados em um cluster de hosts de cálculo.
{: shortdesc}

As lições neste tutorial incluem conceitos de como pega um app existente, conteinerizar o app e implementar o app em um cluster Kubernetes. Para conteinerizar seu app baseado em VM, é possível escolher entre as opções a seguir.

1. Identifique os componentes de um app monolítico grande que pode ser separado em seu próprio microsserviço. É possível conteinerizar esses microsserviços e implementá-los em um cluster Kubernetes.
2. Conteinerize o app inteiro e implemente o app em um cluster Kubernetes.

Dependendo do tipo de app que você tem, as etapas para migrar seu app podem variar. É possível usar esse tutorial para aprender sobre as etapas gerais que você tem que tomar e as coisas que devem ser consideradas antes de migrar seu app.

## Objetivos
{: #objectives}

- Entender como identificar os microsserviços em um app baseado em VM e aprender como mapear componentes entre VMs e Kubernetes.
- Aprender como conteinerizar um app baseado em VM.
- Aprender como implementar o contêiner em um cluster Kubernetes no {{site.data.keyword.containershort_notm}}.
- Colocar tudo o que foi aprendido em prática, executar o app **JPetStore** em seu cluster.

## Serviços usados
{: #products}

Este tutorial usa os serviços do {{site.data.keyword.Bluemix_notm}} a seguir:

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**Atenção:** este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{:#architecture}

### Arquitetura de app tradicional com VMs

O diagrama a seguir mostra um exemplo de uma arquitetura de app tradicional que é baseada em máquinas virtuais.

<p style="text-align: center;">
![Diagrama de arquitetura](images/solution30/traditional_architecture.png)

</p>

1. O usuário envia uma solicitação para o terminal público do app Java. O terminal público é representado por um serviço de balanceador de carga que balanceia o tráfego de rede recebido entre as instâncias do servidor de aplicativos disponíveis.
2. O balanceador de carga seleciona uma das instâncias do servidor de aplicativos funcional que são executadas em uma VM e encaminha a solicitação.
3. O servidor de aplicativos armazena dados do app em um banco de dados MySQL que é executado em uma VM. As instâncias do servidor de aplicativos hospedam o app Java e são executadas em uma VM. Os arquivos de app, como o código do app, os arquivos de configuração e as dependências são armazenados na VM.

### Arquitetura conteinerizada

O diagrama a seguir mostra um exemplo de uma arquitetura de contêiner moderno que é executada em um cluster Kubernetes.

<p style="text-align: center;">
![Diagrama de arquitetura](images/solution30/modern_architecture.png)
</p>

1. O usuário envia uma solicitação para o terminal público do app Java. O terminal público é representado por um balanceador de carga do aplicativo (ALB) do Ingresso que balanceia o tráfego de rede recebido entre os pods de app no cluster. O ALB é uma coleção de regras que permitem o tráfego de rede de entrada para um app exposto publicamente.
2. O ALB encaminha a solicitação para um dos pods de app disponíveis no cluster. Os pods de app são executados nos nós do trabalhador que podem ser uma máquina virtual ou física.
3. Os pods de app armazenam dados em volumes persistentes. Os volumes persistentes podem ser usados para compartilhar dados entre instâncias do app ou nós do trabalhador.
4. Os pods de app armazenam dados em um serviço de banco de dados do {{site.data.keyword.Bluemix_notm}}. É possível executar seu próprio banco de dados dentro do cluster Kubernetes, mas usar um banco de dados como um serviço (DBasS) gerenciado é normalmente mais fácil de configurar e fornece backups e ajuste de escala integrados. É possível localizar muitos tipos diferentes de bancos de dados no [Catálogo de nuvem da IBM](https://{DomainName}/catalog/?category=data).

### VMs, contêineres e Kubernetes

O {{site.data.keyword.containershort_notm}} fornece a capacidade de executar apps conteinerizados em clusters Kubernetes e entrega as ferramentas e funções a seguir:

- Experiência do usuário intuitiva e ferramentas poderosas
- Segurança integrada e isolamento para permitir a entrega rápida de aplicativos seguros
- Serviços de nuvem que incluem recursos cognitivos do IBM® Watson™
- Capacidade de gerenciar recursos de cluster dedicados para aplicativos stateless e cargas de trabalho stateful

#### Máquinas virtuais vs contêineres

**VMs**, os apps tradicionais são executados no hardware nativo. Um único app geralmente não usa os recursos integrais de um único host de cálculo. A maioria das organizações tenta executar múltiplos apps em um único host de cálculo para evitar desperdício de recursos. É possível executar múltiplas cópias do mesmo app, mas para fornecer isolamento, é possível usar VMs para executar múltiplas instâncias de app (VMs) no mesmo hardware. Essas VMs têm pilhas de sistema operacional integral que as tornam relativamente grandes e ineficientes devido à duplicação tanto no tempo de execução quanto no disco.

Os **contêineres** são uma maneira padrão de empacotar apps e todas as suas dependências para que seja possível mover facilmente os apps entre ambientes. Ao contrário de máquinas virtuais, os contêineres não empacotam o sistema operacional. Apenas o código do app, o tempo de execução, as ferramentas do sistema, as bibliotecas e as configurações são compactados dentro dos contêineres. Os contêineres são mais leves, móveis e eficientes do que máquinas virtuais.

Além disso, os contêineres permitem que você compartilhe o S.O. do host. Isso reduz a duplicação enquanto ainda fornece o isolamento. Os contêineres também permitem a você eliminar arquivos desnecessários, como bibliotecas do sistema e binários, para economizar espaço e reduzir sua superfície de ataque. Leia mais sobre máquinas virtuais e contêineres [aqui](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html).

#### Orquestração do Kubernetes

[Kubernetes](http://kubernetes.io/) é um orquestrador de contêiner para gerenciar o ciclo de vida de apps conteinerizados em um cluster de nós do trabalhador. Seus apps podem precisar de muitos outros recursos para execução, como volumes, redes e segredos, que ajudarão a se conectar a outros serviços de nuvem e chaves seguras. O Kubernetes ajuda você a incluir esses recursos em seu app. O paradigma chave do Kubernetes é seu modelo declarativo. O usuário fornece o estado desejado e as tentativas do Kubernetes de se adequar e, em seguida, mantém o estado descrito.

Este [workshop de autocontrole](https://github.com/IBM/kube101/blob/master/workshop/README.md) pode ajudá-lo a obter sua primeira experiência prática com o Kubernetes. Além disso, verifique a página de documentação de [conceitos](https://kubernetes.io/docs/concepts/) do Kubernetes para aprender mais sobre os conceitos de Kubernetes.

### O que a IBM está fazendo por você

Usando clusters Kubernetes com o {{site.data.keyword.containerlong_notm}}, você obtém os benefícios a seguir:

- Múltiplos data centers nos quais é possível implementar seus clusters.
- Suporte para opções de rede de ingresso e de balanceador de carga.
- Suporte de volume persistente dinâmico.
- Os principais do Kubernetes altamente disponíveis e gerenciados pela IBM.

## Dimensionando clusters
{: #sizing_clusters}

À medida que projeta sua arquitetura de cluster, você deseja balancear os custos com relação à disponibilidade, confiabilidade, complexidade e recuperação. Os clusters Kubernetes no {{site.data.keyword.containerlong_notm}} fornecem opções arquiteturais com base nas necessidades de seus apps. Com um pouco de planejamento, é possível obter o máximo de seus recursos em nuvem sem arquitetura excessiva ou gasto excessivo. Mesmo que você superestime ou subestime, é possível escalar facilmente o seu cluster para cima ou para baixo, com nós do trabalhador adicionais ou com nós do trabalhador maiores.

Para executar um app de produção na nuvem usando o Kubernetes, considere os itens a seguir:

1. Você espera tráfego de um local geográfico específico? Se sim, selecione o local que está fisicamente mais próximo de você para obter melhor desempenho.
2. Quantas réplicas de seu cluster você deseja para maior disponibilidade? Um bom ponto de início pode ser três clusters, um para o desenvolvimento, um para teste e um para produção. Verifique o guia de solução [Melhores práticas para organizar usuários, equipes, aplicativos](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments) para criar múltiplos ambientes.
3. Qual [hardware](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes) você precisa para os nós do trabalhador? Máquinas virtuais ou bare metal?
4. Quantos nós do trabalhador você precisa? Isso depende altamente da escala dos apps, quanto mais nós você tiver mais resiliente seu aplicativo será.
5. Quantas réplicas é necessário ter para maior disponibilidade? Implemente os clusters de réplicas em múltiplos locais para tornar seu app mais disponível e proteger o app de ficar inativo devido a uma falha de local.
6. Qual é o conjunto mínimo de recursos que seu app precisa para a inicialização? Você pode desejar testar seu app para a quantia de memória e CPU que ele requer para execução. Em seguida, seu nó do trabalhador deve ter recursos suficientes para implementar e iniciar o app. Certifique-se de configurar as cotas de recurso como parte das especificações de pod. Essa configuração é o que o Kubernetes usa para selecionar (ou planejar) um nó do trabalhador que tem capacidade suficiente para suportar a solicitação. Estime quantos pods serão executados no nó do trabalhador e os requisitos de recurso para esses pods. No mínimo, seu nó do trabalhador deve ser grande o suficiente para suportar um pod para o app.
7. Quando aumentar o número de nós do worder? É possível monitorar o uso do cluster e aumentar os nós quando necessário. Consulte este tutorial para entender como [analisar logs e monitorar o funcionamento de aplicativos do Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana).
8. Você precisa de armazenamento redundante e confiável? Se sim, crie uma solicitação de volume persistente para armazenamento NFS ou ligue um serviço de banco de dados do IBM Cloud ao seu pod.

Para tornar o acima mais específico, vamos supor que você deseja executar um aplicativo da web de produção na nuvem e esperar uma carga de média para alta de tráfego. Vamos explorar quais recursos seriam necessários:

1. Configure três clusters, um para desenvolvimento, um para teste e outro para produção.
2. Os clusters de desenvolvimento e teste podem ser iniciados com a opção mínima de RAM e CPU (por exemplo, 2 CPUs, 4 GB de RAM e um nó do trabalhador para cada cluster).
3. Para o cluster de produção, você pode desejar ter mais recursos para desempenho, alta disponibilidade e resiliência. Podemos escolher uma opção dedicada ou mesmo bare metal e ter pelo menos 4 CPUs, 16 GB de RAM, e dois nós do trabalhador.

## Decidir qual opção do Banco de Dados usar
{: #database_options}

Com o Kubernetes, você tem duas opções para manipular bancos de dados:

1. É possível executar seu banco de dados dentro do cluster Kubernetes, para fazer isso, você precisaria criar um microsserviço para executar o banco de dados. Se estiver usando o exemplo do banco de dados MySQL, será necessário fazer o seguinte:
   - Crie um MySQL Dockerfile, consulte um exemplo [MySQL Dockerfile](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile) aqui.
   - Você precisaria usar segredos para armazenar a credencial do banco de dados. Consulte o exemplo disso [aqui](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret).
   - Você precisaria de um arquivo deployment.yaml com a configuração de seu banco de dados para implementado em Kubernetes. Consulte o exemplo disso [aqui](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml).
2. A segunda opção seria usar a opção de banco de dados como um serviço (DBasS) gerenciado. Essa opção é geralmente mais fácil de configurar e fornece backups e ajuste de escala integrados. É possível localizar muitos tipos diferentes de bancos de dados no [Catálogo de nuvem da IBM](https://{DomainName}/catalog/?category=data). Para usar essa opção, seria necessário fazer o seguinte:
   - Crie um banco de dados como um serviço (DBasS) gerenciado por meio do [catálogo do IBM Cloud](https://{DomainName}/catalog/?category=data).
   - Armazene credenciais do banco de dados dentro de um segredo. Você aprenderá mais sobre segredos na seção "Armazenar credenciais em segredos do Kubernetes".
   - Use o banco de dados como um serviço (DBasS) em seu aplicativo.

## Decidir onde armazenar os arquivos de aplicativo
{: #decide_where_to_store_data}

O {{site.data.keyword.containershort_notm}} fornece várias opções para armazenar e compartilhar dados entre os pods. Nem todas as opções de armazenamento oferecem o mesmo nível de persistência e disponibilidade em situações de desastre.
{: shortdesc}

### Armazenamento de dados não persistentes

Os contêineres e os pods são, pelo design, de curta duração e podem falhar inesperadamente. É possível armazenar dados no sistema de arquivos local de um contêiner. Os dados dentro de um contêiner não podem ser compartilhados com outros contêineres ou pods e são perdidos quando o contêiner trava ou é removido.

### Aprender como criar armazenamento de dados persistentes para seu app

É possível persistir dados do app e dados do contêiner em [armazenamento de arquivo NFS](https://www.ibm.com/cloud/file-storage/details) ou [armazenamento de bloco](https://www.ibm.com/cloud/block-storage) usando volumes persistentes do Kubernetes nativo.
{: shortdesc}

Para provisionar armazenamento de arquivo NFS ou armazenamento de bloco, deve-se solicitar armazenamento para seu pod criando uma solicitação de volume persistente (PVC). Em seu PVC, é possível escolher entre as classes de armazenamento predefinidas que definem o tipo de armazenamento, o tamanho do armazenamento em gigabytes, o IOPS, a política de retenção de dados e as permissões de leitura e gravação para seu armazenamento. Um PVC provisiona dinamicamente um volume persistente (PV) que representa um dispositivo de armazenamento real no {{site.data.keyword.Bluemix_notm}}. É possível montar o PVC em seu pod para ler e gravar no PV. Os dados que são armazenados em PVs estão disponíveis, mesmo se o contêiner trava ou o pod é reagendado. O armazenamento de arquivo e o armazenamento de bloco do NFS que suportam o PV são armazenados em cluster pela IBM para fornecer alta disponibilidade para seus dados.

Para aprender como criar um PVC, siga as etapas abrangidas na [documentação de armazenamento do {{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage).

### Aprender como mover os dados existentes para o armazenamento persistente

Para copiar dados de sua máquina local para seu armazenamento persistente, deve-se montar o PVC em um pod. Em seguida, é possível copiar dados de sua máquina local para o volume persistente em seu pod.
{: shortdesc}

1. Para copiar a data, primeiro, seria necessário criar uma configuração que se pareça com algo como esta:

   ```bash
   kind: Pod
   apiVersion: v1
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
          claimName: mypvc
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/mnt/data"
             name: task-pv-storage
   ```
   {: codeblock}

2. Em seguida, para copiar dados de sua máquina local para o pod, você usaria um comando como este:
   ```
    kubectl cp <local_filepath>/<filename> <namespace>/<pod>:<pod_filepath>
   ```
   {: pre}

3. Copie dados de um pod em seu cluster para sua máquina local:
   ```
   kubectl cp <namespace>/<pod>:<pod_filepath>/<filename> <local_filepath>/<filename>
   ```
   {: pre}


### Configurar backups para armazenamento persistente

Os compartilhamentos de arquivo e armazenamento de bloco são provisionados no mesmo local de seu cluster. O armazenamento em si é hospedado em servidores em cluster pela IBM para fornecer alta disponibilidade. No entanto, os compartilhamentos de arquivo e o armazenamento de bloco não são submetidos a backup automaticamente e poderão ficar inacessíveis se o local inteiro falhar. Para proteger seus dados contra dano ou perda, é possível configurar backups periódicos, que podem ser usados para restaurar seus dados quando necessário.

Para obter mais informações, consulte as opções [backup e restauração](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) para armazenamento de arquivo NFS e armazenamento de bloco.

## Preparar seu código
{: #prepare_code}

### Aplicar os princípios de 12 fatores

O [app de doze fatores](https://12factor.net/) é uma metodologia para construir apps nativos em nuvem. Quando você desejar conteinerizar um app, mova esse app para a nuvem e orquestre o app com o Kubernetes, é importante entender e aplicar alguns desses princípios. Alguns desses princípios são necessários no {{site.data.keyword.Bluemix_notm}}.
{: shortdesc}

Aqui estão alguns dos princípios chave necessários:

- **Código base** - todos os arquivos de código-fonte e de configuração são rastreados dentro de um sistema de controle de versão (por exemplo, um repositório GIT), isso é necessário se você está usando o pipeline do DevOps para implementação.
- **Construir, liberar, executar** - o app de 12 fatores usa separação estrita entre os estágios de construção, liberação e execução. Isso pode ser automatizado com um pipeline de entrega do DevOps integrado para construir e testar o app antes de implementá-lo no cluster. Verifique o [tutorial de Implementação contínua no Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) para saber como configurar uma integração contínua e um pipeline de entrega. Ele abrange a configuração de estágios de controle de fonte, construção, teste e implementação e mostra como incluir integrações, como scanners de segurança, notificações e análise de dados.
- **Configuração** - todas as informações de configuração são armazenadas em variáveis de ambiente. Nenhuma credencial de serviço é codificada permanentemente dentro do código do app. Para armazenar credenciais, é possível usar segredos do Kubernetes. Mais informações sobre credenciais são abrangidas abaixo.

### Armazenar credenciais em segredos do Kubernetes
{: secrets}

Nunca é uma boa prática armazenar credenciais dentro do código do app. Em vez disso, o Kubernetes fornece os chamados **["segredos"](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)** que contêm informações confidenciais, como senhas, tokens OAuth ou chaves ssh. Os segredos do Kubernetes são criptografados por padrão, o que torna os segredos uma opção mais segura e mais flexível para armazenar dados sensíveis do que armazenar esses dados literalmente em uma definição `pod` ou em uma imagem do docker.

Uma maneira de usar segredos no Kubernetes é fazendo algo como isto:

1. Crie um arquivo e armazene as credenciais de serviço dentro dele.
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api_key>
   }
   ```
   {: codeblock}

2. Em seguida, crie um segredo do Kubernetes executando um comando abaixo e verifique se o segredo foi criado usando `kubectl get secrets` depois de executar o comando abaixo:

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## Conteinerizar seu app
{: #build_docker_images}

Para conteinerizar seu app, deve-se criar uma imagem do Docker.
{: shortdesc}

Uma imagem é criada por meio de um [Dockerfile](https://docs.docker.com/engine/reference/builder/#usage), que é um arquivo que contém instruções e comandos para construir a imagem. Um Dockerfile pode referenciar os artefatos de construção em suas instruções que são armazenadas separadamente, como um app, a configuração do app e suas dependências.

Para construir seu próprio Dockerfile para o app existente, é possível usar os comandos a seguir:

- FROM - escolha uma imagem pai para definir o tempo de execução do contêiner.
- ADD/COPY - copie o conteúdo de um diretório para o contêiner.
- WORKDIR - configure o diretório ativo dentro do contêiner.
- RUN - instalar pacotes de software que os apps precisam durante o tempo de execução.
- EXPOSE - disponibilize uma porta fora do contêiner.
- ENV NAME - defina variáveis de ambiente.
- CMD - defina comandos que são executados quando o contêiner é ativado.

As imagens geralmente são armazenadas em um registro que pode ser acessado pelo público (registro público) ou configurado com acesso
limitado para um pequeno grupo de usuários (registro privado). Os registros
públicos, como Docker Hub, podem ser usados na introdução ao Docker e Kubernetes para criar seu
primeiro app conteinerizado em um cluster. Mas quando se trata de apps corporativos, use um registro privado, como aquele fornecido no {{site.data.keyword.registrylong_notm}} para proteger suas imagens de serem usadas e mudadas por usuários não autorizados.

Para conteinerizar um app e armazená-lo no {{site.data.keyword.registrylong_notm}}:

1. Você precisaria criar um Dockerfile, abaixo há um exemplo de um Dockerfile.
   ```
   # Build JPetStore war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # Use WebSphere Liberty base image from the Docker Store
   FROM websphere-liberty:latest

   # Copy war from build stage and server.xml into image
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. Após a criação de um Dockerfile, em seguida, seria necessário construir a imagem de contêiner e enviá-la por push para o {{site.data.keyword.registrylong_notm}}. É possível construir um contêiner usando um comando como:
   ```
   ibmcloud cr build -t <image_name> <directory_of_Dockerfile>
   ```
   {: pre}

## Implementar seu app em um cluster Kubernetes
{: #deploy_to_kubernetes}

Depois que uma imagem de contêiner é construída e enviada por push para a nuvem, em seguida, é necessário implementar em seu cluster Kubernetes. Para fazer isso, seria necessário criar um arquivo deployment.yaml.
{: shortdesc}

### Aprender como criar um arquivo yaml de implementação do Kubernetes

Para criar arquivos deployment.yaml do Kubernetes, seria necessário fazer algo como isto:

1. Crie um arquivo deployment.yaml, aqui está um exemplo de um arquivo [YAML de implementação](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml).

2. Em seu arquivo deployment.yaml, é possível definir [cotas de recurso](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) para seus contêineres para especificar a quantia de CPU e memória que cada contêiner precisa para iniciar adequadamente. Se os contêineres tiverem cotas de recurso especificadas, o planejador do Kubernetes poderá tomar melhores decisões sobre o nó do trabalhador no qual colocar seus pods.

3. Em seguida, é possível usar os comandos abaixo para criar e visualizar a implementação e os serviços criados:

   ```
   kubectl create -f <filepath/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## Resumo
{: #summary}

Neste tutorial, você aprendeu o seguinte:

- As diferenças entre VMs, contêineres e Kubernetes.
- Como definir clusters para diferentes tipos de ambiente (desenvolvimento, teste e produção).
- Como manipular o armazenamento de dados e a importância do armazenamento de dados persistentes.
- Aplique os princípios de 12 fatores a seu app e use segredos para credenciais no Kubernetes.
- Construa imagens do docker e envie-as por push para o {{site.data.keyword.registrylong_notm}}.
- Crie arquivos de implementação do Kubernetes e implemente a imagem do Docker no Kubernetes.

## Colocar tudo o que foi aprendido em prática, executar o app JPetStore em seu cluster.
{: #runthejpetstore}

Para colocar tudo o que você aprendeu em prática, siga a [demo](https://github.com/ibm-cloud/ModernizeDemo/) para executar o app **JPetStore** em seu cluster e aplicar os conceitos aprendidos. O app JPetStore tem alguma funcionalidade ampliada para permitir que você amplie um app no Kubernetes por serviços IBM Watson em execução como um microsserviço separado.

## Conteúdo relacionado
{: #related}

- [Introdução](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/) ao Kubernetes e {{site.data.keyword.containershort_notm}}.
- Laboratórios do {{site.data.keyword.containershort_notm}} no [GitHub](https://github.com/IBM/container-service-getting-started-wt).
- [Docs](http://kubernetes.io/) principais do Kubernetes.
- [Armazenamento persistente](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) no {{site.data.keyword.containershort_notm}}.
- [Guia de solução de melhores práticas](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications) para organizar usuários, equipes e apps.
- [Analise logs e monitore o funcionamento do aplicativo com LogDNA e Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Configurar a [Delivery Pipeline e o pipeline de integração](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) para apps conteinerizados que são executados no Kubernetes.
- Implemente o cluster de produção [em diversas localizações](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Usar [múltiplos clusters em múltiplos locais](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations) para alta disponibilidade.
