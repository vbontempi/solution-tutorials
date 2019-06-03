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

# Clusters Kubernetes multiregion resilientes e seguros com o Cloud Internet Services
{: #multi-region-k8s-cis}

Os usuários são menos propensos a experienciar o tempo de inatividade quando um aplicativo é projetado com a resiliência em mente. Ao implementar uma solução com o {{site.data.keyword.containershort_notm}}, você se beneficia de recursos integrados, como balanceamento de carga e isolamento, aumenta a resiliência com relação a potenciais falhas com hosts, redes ou apps. Ao criar vários clusters e se uma indisponibilidade ocorrer com um cluster, os usuários ainda poderão acessar um app que também esteja implementado em outro cluster. Com múltiplos clusters em locais diferentes, os usuários também podem acessar o cluster mais próximo e reduzir a latência de rede. Para resiliência adicional, você tem a opção de selecionar também os clusters de multizona, o que significa que seus nós são implementados em múltiplas zonas dentro de um local.

Este tutorial destaca como o Cloud Internet Services (CIS), uma plataforma uniforme para configurar e gerenciar o Sistema de Nomes de Domínio (DNS), o Global Load Balancing (GLB), o Web Application Firewall (WAF) e a proteção contra o Distributed Denial of Service (DDoS) para aplicativos da Internet podem ser integrados a clusters Kubernetes para suportar esse cenário e entregar uma solução segura e resiliente em múltiplos locais.

## Objetivos
{: #objectives}

* Implementar um aplicativo em múltiplos clusters Kubernetes em um local diferente.
* Distribuir o tráfego entre múltiplos clusters com um Global Load Balancer.
* Rotear usuários para o cluster mais próximo.
* Proteger seu aplicativo contra ameaças de segurança.
* Aumentar o desempenho do aplicativo com o armazenamento em cache.

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

<p style="text-align: center;">
  ![Arquitetura](images/solution32-multi-region-k8s-cis/Architecture.png)
</p>

1. O desenvolvedor constrói as imagens do Docker para o aplicativo.
2. As imagens são enviadas por push para o {{site.data.keyword.registryshort_notm}} em Dallas e em Londres.
3. O aplicativo é implementado em clusters Kubernetes em ambas as localizações.
4. Os usuários finais acessam o aplicativo.
5. O Cloud Internet Services é configurado para interceptar as solicitações ao aplicativo e distribuir a carga entre os clusters. Além disso, o DDoS Protection e o Web Application Firewall são ativados para proteger o aplicativo de ameaças comuns. Opcionalmente, ativos como imagens, arquivos CSS são armazenados em cache.

## Antes de Começar
{: #prereqs}

* O Cloud Internet Services requer que você tenha um domínio customizado para que possa configurar o DNS para que esse domínio aponte para os servidores de nomes do Cloud Internet Services.
* [Instale o Git](https://git-scm.com/).
* [Instale a CLI do {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli).
* [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Script para instalar docker, kubectl, helm, ibmcloud cli e plug-ins necessários.
* [Configure a CLI {{site.data.keyword.registrylong_notm}} e o espaço de nomes de registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Entenda os conceitos básicos de Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Implementar um aplicativo em um local

Este tutorial implementa um aplicativo do Kubernetes em clusters em múltiplos locais. Você iniciará com um local, Dallas, e, em seguida, repetirá estas etapas para Londres.

### Criar um cluster Kubernetes
{: #create_cluster}

Para criar um cluster:
1. Selecione **{{site.data.keyword.containershort_notm}}** no [Catálogo do {{site.data.keyword.cloud_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster/create).
1. Configure **Local** para **Dallas**.
1. Selecione o cluster **Padrão**.
1. Selecione uma ou mais zonas como **Local**. A criação de um cluster de multizona aumenta a resiliência do aplicativo. Os usuários são muito menos propensos a experienciar o tempo de inatividade quando o app é distribuído entre múltiplas zonas. Mais sobre clusters de multizona pode ser localizado [aqui](https://{DomainName}/docs/containers?topic=containers-plan_clusters#ha_clusters).
1. Configure **Tipo de máquina** para o menor disponível - **2 CPUs** e **4 GB de RAM** são suficientes para este tutorial.
1. Use **2** nós do trabalhador.
1. Configure **Nome do cluster** para **my-us-cluster**. Use o padrão de nomenclatura *`my-<location>-cluster`* para ser consistente com este tutorial.

Enquanto o cluster está ficando pronto, você vai preparar o aplicativo.

### Criar um namespace no {{site.data.keyword.registryshort_notm}}

1. Destine a CLI do {{site.data.keyword.Bluemix_notm}} para Dallas.
   ```bash
   ibmcloud target -r us-south
   ```
   {: pre}
2. Crie um namespace para o aplicativo.
   ```bash
   ibmcloud cr namespace-add <your_namespace>
   ```
   {: pre}

Também será possível reutilizar um namespace existente se você tiver um no local. É possível listar namespaces existentes com `ibmcloud cr namespaces`.
{: tip}

### Construir o aplicativo

Esta etapa constrói o aplicativo em uma imagem do Docker. Será possível ignorar esta etapa se você estiver configurando o segundo cluster. É um app HelloWorld simples.

1. Clone o código-fonte para o [app Hello world](https://github.com/IBM/container-service-getting-started-wt){:new_windows} em seu diretório inicial do usuário. O repositório contém versões diferentes de um app semelhante em pastas que iniciam com Lab.
   ```bash
   git clone https://github.com/IBM/container-service-getting-started-wt.git
   ```
   {: pre}
1. Navegue para o diretório `Lab 1`.
   ```bash
   cd 'container-service-getting-started-wt/Lab 1'
   ```
   {: pre}
1. Construa uma imagem do Docker que inclua os arquivos de app do diretório `Lab 1`.
   ```bash
   docker build --tag multi-region-hello-world:1 .
   ```
   {: pre}

### Preparar a imagem para ser enviada por push para o registro específico do local

Identifique a imagem com o registro de destino:

   ```bash
   docker tag multi-region-hello-world:1 us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### Enviar por push a imagem para o registro específico do local

1. Assegure-se de que o mecanismo de Docker local possa enviar por push para o registro em Dallas.
   ```bash
   ibmcloud cr login
   ```
   {: pre}
2. Envie por push a imagem.
   ```bash
   docker push us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### Implementar o aplicativo no cluster Kubernetes

Nesse estágio, o cluster deve estar pronto. É possível verificar seu status no console do [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/clusters).

1. Recupere a configuração do cluster:
   ```bash
   ibmcloud cs cluster-config <us-cluster-name>
   ```
   {: pre}
1. Copie e cole a saída para configurar a variável de ambiente KUBECONFIG. A variável é usada pelo `kubectl`.
1. Execute o aplicativo no cluster com duas réplicas:
   ```bash
   kubectl run hello-world-deployment --image=us.icr.io/<your_US-South_namespace>/hello-world:1 --replicas=2
   ```
   {: pre}
   Saída de exemplo: `deployment "hello-world-deployment" created`.
1. Torne o aplicativo acessível dentro do cluster
   ```bash
   kubectl expose deployment/hello-world-deployment --type=ClusterIP --port=80 --name=hello-world-service --target-port=8080
   ```
   {: pre}
   Isso retorna uma mensagem como `service "hello-world-service" exposed`.

### Obter o nome de domínio e o endereço IP designados ao cluster
{: #CSALB_IP_subdomain}

Quando um cluster Kubernetes é criado, ele é designado a um subdomínio do Ingresso (por exemplo, *my-us-cluster.us-south.containers.appdomain.cloud*) e um endereço IP público do Balanceador de Carga do Aplicativo.

1. Recupere o subdomínio do Ingresso do cluster:
   ```bash
   ibmcloud cs cluster-get <us-cluster-name>
   ```
   {: pre}
   Procure o valor `Ingress Subdomain`.
1. Anote essas informações para uma etapa posterior.

Este tutorial usa o subdomínio do Ingresso para configurar o Global Load Balancer. Também é possível trocar o subdomínio para o endereço IP público do Balanceador de Carga de Aplicativo (`ibmcloud cs albs -cluster <us-cluster-name>`). Ambas as opções são suportadas.
{: tip}

## E, em seguida, para outro local

Repita as etapas anteriores em Londres, substituindo:
* o nome do local **Dallas** por **London**;
* o alias de local **us-south** por **eu-gb**;
* o registro *us.icr.io* por **uk.icr.io**;
* e o nome do cluster *my-us-cluster* por **my-uk-cluster**.

## Configurar o balanceamento de carga multilocal

Seu aplicativo está agora em execução em dois clusters, mas está faltando um componente para que os usuários acessem os clusters de forma transparente por meio de um único ponto de entrada.

Nesta seção, você configurará o Cloud Internet Services (CIS) para distribuir a carga entre os dois clusters. O CIS é um serviço de balcão que fornece o Global Load Balancer (GLB), o Armazenamento em Cache, o Web Application Firewall (WAF) e a regra de Página para proteger seus aplicativos, enquanto assegura a confiabilidade e o desempenho para seus aplicativos do Cloud.

Para configurar um balanceador de carga global, você precisará:
* apontar um domínio customizado para servidores de nomes do CIS,
* recuperar os endereços IP ou os nomes de subdomínio dos clusters Kubernetes,
* configurar verificações de funcionamento para validar a disponibilidade de seu aplicativo,
* e definir conjuntos de origem apontando para os clusters.

### Registrar um domínio customizado com o Cloud Internet Services
{: #create_cis_instance}

A primeira etapa é criar uma instância do CIS e apontar seu domínio customizado para servidores de nomes do CIS.

1. Se você não tiver um domínio, será possível comprar um de um registrador, como [godaddy.com](http://godaddy.com).
2. Navegue para o [Internet Services](https://{DomainName}/catalog/services/internet-services) no catálogo do {{site.data.keyword.Bluemix_notm}}.
3. Configure o nome do serviço e clique em **Criar** para criar uma instância do serviço.
4. Quando a instância de serviço for provisionada, configure seu nome de domínio e clique em **Incluir domínio**.
5. Quando os servidores de nomes forem designados, configure seu registrador ou provedor de nome de domínio para usar os servidores de nomes listados.
6. Depois de ter configurado seu registrador ou o provedor de DNS, pode ser necessário até 24 horas para que as mudanças entrem em vigor.

   Quando o status do domínio na página Visão geral muda de *Pendente* para *Ativo*, é possível usar o comando `dig <your_domain_name> ns` para verificar se os novos servidores de nomes entraram em vigor.
   {:tip}

### Configurar a verificação de funcionamento para o Global Load Balancer

Uma verificação de funcionamento ajuda a ganhar insight sobre a disponibilidade de conjuntos para que o tráfego possa ser roteado para os operantes. Essas verificações enviam periodicamente solicitações HTTP/HTTPS e monitoram as respostas.

1. No painel do Cloud Internet Services, navegue para **Confiabilidade** > **Global Load Balancer** e, na parte inferior da página, clique em **Criar verificação de funcionamento**.
1. Configure **Caminho** como **/**
1. Configure **Tipo de monitor** como **HTTP**.
1. Clique em **Provisionar 1 instância**.

   Ao construir seus próprios aplicativos, é possível definir um terminal de funcionamento dedicado, como */heathz*, no qual você relataria o estado do aplicativo.
   {:tip}

### Definir conjuntos de origem

Um conjunto é um grupo de servidores de origem para os quais o tráfego é roteado de forma inteligente quando conectado a um GLB. Com os clusters no Reino Unido e nos Estados Unidos, é possível definir conjuntos baseados em local e configurar o CIS para redirecionar usuários para os clusters mais próximos com base na localização geográfica das solicitações do usuário.

#### Um conjunto para o cluster em Londres
1. Clique em **Criar conjunto**.
2. Configure **Nome** como **UK**
3. Configure **Verificação de funcionamento** para a que foi criada na seção anterior
4. Configure **Região de verificação de funcionamento** como **Europa Ocidental**
5. Configure **Nome de origem** como **uk-cluster**
6. Configure **Endereço de origem** para o subdomínio do Ingresso do cluster em Londres, por exemplo, *my_uk_cluster.eu-gb.containers.appdomain.cloud*
7. Clique em **Provisionar 1 instância**.

#### Um conjunto para o cluster em Dallas
1. Clique em **Criar conjunto**.
2. Configure **Nome** como **EUA**
3. Configure **Verificação de funcionamento** para a que foi criada na seção anterior
4. Configure **Região de verificação de funcionamento** como **Oeste da América do Norte**
5. Configure **Nome de origem** como **us-cluster**
6. Configure **Endereço de origem** para o subdomínio do Ingresso do cluster em Dallas, por exemplo, *my_us_cluster.us-south.containers.appdomain.cloud*
7. Clique em **Provisionar 1 instância**.

#### E um conjunto com ambos os clusters
1. Clique em **Criar conjunto**.
1. Configure **Nome** como **Todos**
1. Configure **Verificação de funcionamento** para a que foi criada na seção anterior
1. Configure **Região de verificação de funcionamento** como **Leste da América do Norte**
1. Inclua duas origens:
   1. uma com **Nome de origem** configurado como **us-cluster** e o **Endereço de origem** configurado para o subdomínio do Ingresso do cluster em Dallas
   2. uma com **Nome de origem** configurado como **uk-cluster** e o **Endereço de origem** configurado para o subdomínio do Ingresso do cluster em Londres
2. Clique em **Provisionar 1 instância**.

### Criar o Global Load Balancer

Com os conjuntos de origem definidos, é possível concluir a configuração do balanceador de carga.

1. Clique em **Criar balanceador de carga**.
1. Insira um nome em **Nome do host do balanceador** para o Global Load Balancer. Esse nome também fará parte de sua URL do aplicativo universal (`http://<glb_name>.<your_domain_name>`), independentemente da localização.
1. Em **Conjuntos de origem padrão**, clique em **Incluir conjunto** e inclua o conjunto denominado **Todos**.
1. Expanda a seção de **Configurar rotas geográficas (opcional)**:
   1. Clique em **Incluir rota**, selecione **Europa Ocidental** e clique em **Incluir**.
   1. Clique em **Incluir conjunto** para selecionar o conjunto **UK**.
   1. Configure rotas adicionais, conforme mostrado na tabela a seguir.
   1. Clique em **Provisionar 1 instância**.

| Região               | Conjunto de Origem |
| :---------------:    | :---------: |
|Europa Ocidental      |     UK      |
|Leste Europeu         |     UK      |
|Nordeste Asiático     |     UK      |
|Sudeste Asiático      |     UK      |
|Oeste da América do Norte |     EUA      |
|Leste da América do Norte |     EUA      |

Com essa configuração, os usuários na Europa e na Ásia serão redirecionados para o cluster em Londres, usuários nos EUA para o cluster de Dallas. Quando uma solicitação não corresponder a nenhuma das rotas definidas, ela será redirecionada para os **Conjuntos de origem padrão**.

### Criar recurso de Ingresso para clusters Kubernetes por local

O Global Load Balancer está agora pronto para entregar solicitações. Todas as verificações de funcionamento devem ser verdes. Mas há uma última etapa de configuração necessária nos clusters Kubernetes para responder corretamente às solicitações vindas do Global Load Balancer: é necessário definir um recurso de Ingresso para manipular solicitações do domínio do GLB.

1. Crie um arquivo de recursos de Ingresso denominado **glb-ingresss.yaml**
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: glb-ingress
   spec:
    rules:
      - host: <glb_name>.<your_domain_name>
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-world-service
              servicePort: 80
   ```
    {: pre}
    Substitua `<glb_name>.<your_domain_name>` pela URL definida na seção anterior.
1. Implemente esse recurso nos clusters de Londres e Dallas, depois de configurar a variável KUBECONFIG para os respectivos clusters de local:
   ```bash
   kubectl create -f glb-ingress.yaml
   ```
   {: pre}
   Ele gera a mensagem `ingress.extension "glb-ingress" created`.

Neste estágio, você configurou com êxito um Global Load Balancer com clusters Kubernetes em múltiplos locais. É possível acessar a URL do GLB `http://<glb_name>.<your_domain_name>` para visualizar seu aplicativo. Com base em seu local, você é redirecionado para o cluster mais próximo ou um cluster do conjunto padrão se o CIS não pôde mapear seu endereço IP para um local específico.

## Proteger o aplicativo
{: #secure_via_CIS}

### Ativar o Web Application Firewall

O Web Application Firewall (WAF) protege seu aplicativo da web contra ataques da Camada 7 do ISO. Geralmente, ele é combinado com conjuntos de regras agrupadas, esses conjuntos de regras visam proteger contra vulnerabilidades no aplicativo filtrando o tráfego malicioso.

1. No painel do Cloud Internet Services, navegue para **Segurança**, em seguida, na guia **Gerenciar**.
1. Na seção **Web Application Firewall**, assegure-se de que o WAF esteja ativado.
1. Clique em **Visualizar conjunto de Regras do OWASP**. Nessa página, é possível revisar o **Conjunto de regras principal do OWASP** e ativar ou desativar as regras individualmente. Quando uma regra estiver ativada, se uma solicitação recebida acionar a regra, a pontuação de ameaça global será aumentada. A configuração **Sensibilidade** decidirá se uma **Ação** é acionada para a solicitação.
   1. Deixe os conjuntos de regras do OWASP padrão no estado em que se encontram.
   1. Configure **Sensibilidade** como `Low`.
   1. Configure **Ação** como `Simulate` para registrar todos os eventos.
1. Clique em **Voltar para a segurança**.
1. Clique em **Visualizar conjunto de regras do CIS**. Essa página mostra regras adicionais construídas em torno de pilhas de tecnologia comum para hospedar websites.

### Aumentar o desempenho e proteger contra ataques de Negação de Serviço 
{: #proxy_setting}

Um ataque distributed denial of service ([DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)) é uma tentativa maliciosa de interromper o tráfego normal de um servidor, serviço ou rede, sobrecarregando o destino ou sua infraestrutura circundante com uma inundação de tráfego de Internet. O CIS está equipado para proteger seu domínio contra DDoS.

1. No painel do CIS, selecione **Confiabilidade** > **Global Load Balancer**.
1. Localize o GLB que você criou na tabela **Load Balancers**.
1. Ative os recursos de Segurança e Desempenho na coluna **Proxy**:

   ![Alternar o Proxy do CIS para ON](images/solution32-multi-region-k8s-cis/cis-proxy.png)

**Seu GLB está agora protegido**. Um benefício imediato é que o endereço IP de origem de seus clusters será oculto dos clientes. Se o CIS detectar uma ameaça para uma solicitação futura, o usuário poderá ver uma tela como esta antes de ela ser redirecionada para seu aplicativo:

   ![verificando - proteção de DDoS](images/solution32-multi-region-k8s-cis/cis-DDoS.png)

Além disso, agora é possível controlar qual conteúdo é armazenado em cache pelo CIS e por quanto tempo ele permanece armazenado em cache. Acesse **Desempenho** > **Armazenamento em cache** para definir o nível de armazenamento em cache global e a expiração do navegador. É possível customizar as regras de segurança global e de armazenamento em cache com **Regras de página**. As Regras de página permitem a configuração de baixa granularidade usando caminhos de domínio específicos. Como exemplo com Regras de página, você poderia decidir armazenar em cache todo o conteúdo em **/assets** por **3 dias**:

   ![regras de página](images/solution32-multi-region-k8s-cis/cis-pagerules.png)

## Remover recursos
{:removeresources}

### Remover recursos de cluster do Kubernetes
1. Remova o Ingresso.
1. Remova o serviço.
1. Remova a implementação.
1. Exclua os clusters se eles tiverem sido criados especificamente para este tutorial.

### Remover recursos CIS
1. Remova o GLB.
1. Remova os conjuntos de origem.
1. Remova as verificações de funcionamento.

## Conteúdo relacionado
{:related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Gerencie seu IBM CIS para segurança ideal](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#best-practice-2-configure-your-security-level-selectively)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
* [{{site.data.keyword.registrylong_notm}}Básicas ](https://{DomainName}/docs/services/Registry?topic=registry-registry_overview#registry_planning)
* [Implementando apps de instância única em clusters do Kubernetes](https://{DomainName}/docs/containers?topic=containers-cs_apps_tutorial#cs_apps_tutorial_lesson1)
* [Melhor prática para proteger o tráfego e o aplicativo de Internet por meio do CIS](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#manage-your-ibm-cis-for-optimal-security)
* [Melhorando a disponibilidade do app com clusters multizonas](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
