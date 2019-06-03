---
copyright:
  years: 2018, 2019
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

# Processamento de dados assíncronos usando o armazenamento de objeto e o sistema de mensagens de publicação/assinatura
{: #pub-sub-object-storage}
Neste tutorial, você aprenderá como usar um serviço de sistema de mensagens baseado no Apache Kafka para orquestrar as cargas de trabalho de longa execução para aplicativos em execução em um cluster Kubernetes. Esse padrão é usado para desacoplar seu aplicativo permitindo maior controle sobre o ajuste de escala e desempenho. O {{site.data.keyword.messagehub}} pode ser usado para enfileirar o trabalho a ser feito sem afetar os aplicativos de produtor, tornando-o um sistema ideal para tarefas de longa execução. 

{:shortdesc}

Você simulará esse padrão usando um exemplo de processamento de arquivo. Primeiro, crie um aplicativo de IU que será usado para fazer upload de arquivos para armazenamento de objeto e gerar mensagens indicando o trabalho a ser feito. Em seguida, você criará um aplicativo trabalhador separado que processará de forma assíncrona os arquivos transferidos por upload do usuário quando ele receber mensagens.

## Objetivos
{: #objectives}

* Implementar um padrão de produtor-consumidor com o {{site.data.keyword.messagehub}}
* Ligar serviços a um cluster Kubernetes

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

Neste tutorial, o aplicativo da IU é gravado no Node.js e o aplicativo trabalhador é gravado em Java destacando a flexibilidade desse padrão. Embora ambos os aplicativos estejam em execução no mesmo cluster Kubernetes neste tutorial, uma parte também poderia ter sido implementada como um aplicativo do Cloud Foundry ou uma função serverless.

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. O usuário faz upload do arquivo usando o aplicativo de IU
2. O arquivo é salvo no {{site.data.keyword.cos_full_notm}}
3. A mensagem é enviada para o tópico do {{site.data.keyword.messagehub}} indicando que o novo arquivo está aguardando processamento.
4. Quando prontos, os trabalhadores atendem as mensagens e iniciam o processamento do novo arquivo.

## Antes de Começar
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - Ferramenta para instalar a CLI do {{site.data.keyword.cloud_notm}}, o Kubernetes, o Helm e o Docker.

## Criar um cluster Kubernetes
{: #create_kube_cluster}

1. Crie um cluster Kubernetes por meio do [Catálogo](https://{DomainName}/containers-kubernetes/launch). Nomeie-o como `mycluster` para que seja fácil seguir este tutorial. Este tutorial pode ser realizado com um cluster **Grátis**.
   ![Criação de cluster Kubernetes no IBM Cloud](images/solution25/KubernetesClusterCreation.png)
2. Verifique o status do **Cluster** e dos **Nós do trabalhador** e aguarde que eles estejam **prontos**.

### Configurar kubectl

Nesta etapa, você configurará kubectl para apontar para o seu cluster recém-criado que avança. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) é uma ferramenta de linha de comandos que você usa para interagir com um cluster Kubernetes.

1. Use `ibmcloud login` para efetuar login interativamente. Forneça a organização (org), a localização e o espaço sob os quais o cluster é criado. É possível reconfirmar os detalhes ao executar o comando `ibmcloud target`.
2. Quando o cluster estiver pronto, recupere a configuração de cluster:
   ```bash
   ibmcloud cs cluster-config <cluster-name>
   ```
   {: pre}
3. Copie e cole o comando **export** para configurar a variável de ambiente KUBECONFIG como instruído. Para verificar se a variável de ambiente KUBECONFIG está configurada adequadamente ou não, execute o comando a seguir: `echo $KUBECONFIG`
4. Verificar se o comando `kubectl` está configurado corretamente
   ```bash
   kubectl cluster-info
   ```
  {: pre}
   ![](images/solution2/kubectl_cluster-info.png)

## Criar uma instância do {{site.data.keyword.messagehub}}
 {: #create_messagehub}

O {{site.data.keyword.messagehub}} é um serviço de sistema de mensagens rápido, escalável e totalmente gerenciado, com base no Apache Kafka, um sistema de mensagens de software livre, de alto rendimento, que fornece uma plataforma de baixa latência para manipulação de feeds de dados em tempo real.

 1. No Painel, clique em [**Criar recurso**](https://{DomainName}/catalog/) e selecione [**{{site.data.keyword.messagehub}}**](https://{DomainName}/catalog/services/event-streams) na seção Application Services.
 2. Nomeie o serviço `mymessagehub` e clique em **Criar**.
 3. Forneça as credenciais de serviço para seu cluster ligando a instância de serviço ao espaço de nomes do Kubernetes `padrão`.
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

O comando cluster-service-bind cria um segredo de cluster que contém as credenciais de sua instância de serviço no formato JSON. Use `kubectl get secrets` para ver o segredo gerado com o nome `binding-mymessagehub`. Consulte [Integrando serviços](https://{DomainName}/docs/containers?topic=containers-integrations#integrations) para obter mais informações

{:tip}

## Criar uma instância do Object Storage

{: #create_cos}

O {{site.data.keyword.cos_full_notm}} é criptografado e disperso em múltiplos locais geográficos e acessado por meio de HTTP usando uma API de REST. O {{site.data.keyword.cos_full_notm}} fornece armazenamento em nuvem flexível, com custo reduzido e escalável para dados não estruturados. Você usará isso para armazenar os arquivos transferidos por upload pela IU.

1. No Painel, clique em [**Criar recurso**](https://{DomainName}/catalog/) e selecione [**{{site.data.keyword.cos_short}}**](https://{DomainName}/catalog/services/cloud-object-storage) na seção Armazenamento.
2. Nomeie o serviço `myobjectstorage`, clique em **Criar**.
3. Clique em **Criar depósito**.
4. Configure o nome do depósito para um nome exclusivo, como `username-mybucket`.
5. Selecione Resiliência de **Região cruzada** e Local **us-geo** e clique em **Criar**
6. Forneça as credenciais de serviço para seu cluster ligando a instância de serviço ao espaço de nomes do Kubernetes `padrão`.
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## Implementar o aplicativo de IU no cluster

O aplicativo de IU é um aplicativo da web Node.js Express simples que permite que o usuário faça upload de arquivos. Ele armazena os arquivos na instância do Object Storage criada acima e, em seguida, envia uma mensagem para o tópico do {{site.data.keyword.messagehub}} "tópico de trabalho" que um novo arquivo está pronto para ser processado.

1. Clone o repositório de aplicativo de amostra localmente e mude o diretório para a pasta `pubsub-ui`.
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. Abra `config.js` e atualize o COSBucketName com seu nome do depósito.
3. Construa e implemente o aplicativo. O comando de implementação gera uma imagem do docker, envia-a por push para seu {{site.data.keyword.registryshort_notm}} e, em seguida, cria uma implementação do Kubernetes. Siga as instruções interativas ao implementar o app.
```sh
  ibmcloud dev build ibmcloud dev deploy -t container
```
4. Visite o aplicativo e faça upload dos arquivos por meio da pasta `sample-files`. Os arquivos transferidos por upload serão armazenados no Object Storage e o status será "aguardando" até que eles sejam processados pelo aplicativo trabalhador. Deixe essa janela do navegador aberta.

   ![](images/solution25/files_uploaded.png)

## Implementar o aplicativo trabalhador no cluster

O aplicativo trabalhador é um aplicativo Java que atende ao tópico do {{site.data.keyword.messagehub}} Kafka "tópico de trabalho" para mensagens. Em uma nova mensagem, o trabalhador recuperará o nome do arquivo por meio da mensagem e, em seguida, obterá o conteúdo do arquivo do Object Storage. Em seguida, ele simulará o processamento do arquivo e enviará outra mensagem para o tópico "trabalho de resultado" na conclusão. O aplicativo de IU atendará a esse tópico e atualizará o status.

1. Mude o diretório para o diretório `pubsub-worker`
```sh
  cd ../pubsub-worker
```
2. Abra `resources/cos.properties` e atualize a propriedade `bucket.name` com seu nome do depósito.
2. Construa e implemente o aplicativo trabalhador.
```
  ibmcloud dev build ibmcloud dev deploy -t container
```
3. Após a implementação ser concluída, verifique a janela do navegador com seu aplicativo da web novamente. Observe que o status ao lado de cada arquivo agora mudou para "processado".
![](images/solution25/files_processed.png)

Neste tutorial, você aprendeu como é possível usar o {{site.data.keyword.messagehub}} baseado em Kafka para implementar um padrão de produtor-consumidor. Isso permite que o aplicativo da web seja rápido e transfira o processamento pesado para outros aplicativos. Quando o trabalho precisa ser feito, o produtor (aplicativo da web) cria mensagens e o trabalho tem a carga balanceada entre um ou mais trabalhadores que assinam as mensagens. Você usou um aplicativo Java em execução no Kubernetes para manipular o processamento, mas esses aplicativos também podem ser [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing). Os aplicativos em execução no Kubernetes são ideais para cargas de trabalho de longa execução e intensivas, em que o {{site.data.keyword.openwhisk_short}} seria mais adequado para processos de curta duração.

## Remover recursos
{:removeresources}

Navegue para [Lista de recursos](https://{DomainName}/resources/) e
1. exclua o cluster Kubernetes `mycluster`
2. exclua o {{site.data.keyword.cos_full_notm}} `myobjectstorage`
3. exclua o {{site.data.keyword.messagehub}} `mymessagehub`
4. selecione **Kubernetes** no menu à esquerda, **Registro** e, em seguida, exclua os repositórios `pubsub-xxx`.

## Conteúdo relacionado
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [Gerenciar acesso ao Object Storage](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [Processamento de dados do {{site.data.keyword.messagehub}} com o {{site.data.keyword.openwhisk_short}}](https://github.com/IBM/openwhisk-data-processing-message-hub)
