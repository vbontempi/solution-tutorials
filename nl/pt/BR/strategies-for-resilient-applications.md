---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-11"
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

# Estratégias para aplicativos resilientes
{: #strategies-for-resilient-applications}

Independentemente da opção de cálculo, Kubernetes, Cloud Foundry, Cloud Functions ou Virtual Servers, as empresas buscam minimizar o tempo de inatividade e criar arquiteturas resilientes que atingem a máxima disponibilidade. Este tutorial destaca os recursos do IBM Cloud para construir soluções resilientes e, ao fazer isso, responde às perguntas a seguir.

- O que devo considerar ao preparar uma solução para estar globalmente disponível?
- Como as opções de cálculo disponíveis ajudam a entregar aplicativos de multiregion?
- Como importar artefatos de aplicativo ou de serviço para regiões adicionais?
- Como os bancos de dados podem ser replicados nos locais?
- Quais serviços auxiliares devem ser usados: Block Storage, File Storage, Object Storage, Databases?
- Há alguma consideração específica do serviço?

## Objetivos
{: #objectives}

* Aprenda os conceitos arquiteturais envolvidos ao construir aplicativos resilientes.
* Entenda como esses conceitos são mapeados para as ofertas de cálculo e serviço do IBM Cloud

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura e conceitos

{: #architecture}

Para projetar uma arquitetura resiliente, é necessário considerar os blocos individuais de sua solução e seus recursos específicos. 

Abaixo, há uma arquitetura multiregion que mostra os diferentes componentes que podem existir em uma configuração de multiregion. ![Arquitetura](images/solution39/Architecture.png)

O diagrama de arquitetura acima pode ser diferente dependendo da opção de cálculo. Você verá diagramas de arquitetura específicos em cada opção de cálculo em seções posteriores. 

### Recuperação de desastre com duas regiões 

Para facilitar a recuperação de desastre, duas arquiteturas amplamente aceitas são usadas: **ativa/ativa** e **ativa/passiva**. Cada arquitetura tem seus próprios custos e benefícios relacionados ao tempo e ao esforço durante a recuperação.

#### Configuração ativa/ativa

Em uma arquitetura ativa/ativa, ambos os locais têm instâncias ativas idênticas com um balanceador de carga que distribui o tráfego entre eles. Usando essa abordagem, a replicação de dados deve estar em vigor para sincron ambas as regiões em tempo real.

![Ativa/Ativa](images/solution39/Active-active.png)

Essa configuração fornece maior disponibilidade com menos correção manual do que uma arquitetura ativa/passiva. As solicitações são entregues por meio de ambos os data centers. Será necessário configurar os serviços de borda (balanceador de carga) com o tempo limite apropriado e a lógica de nova tentativa para rotear automaticamente a solicitação para o segundo data center se ocorrer uma falha no primeiro ambiente do data center.

Ao considerar o **objetivo do ponto de recuperação** (RPO) no cenário ativo/ativo, a sincronização de dados entre os dois data centers ativos deve ser extremamente oportuna para permitir o fluxo contínuo de solicitação.

#### Configuração ativa/passiva

Uma arquitetura ativa/passiva conta com uma região ativa e uma segunda região (passiva) usada como um backup. No caso de uma indisponibilidade na região ativa, a região passiva torna-se ativa. A intervenção manual pode ser necessária para assegurar que os bancos de dados ou o armazenamento de arquivo sejam atuais com as necessidades do aplicativo e do usuário. 

![Ativa/Ativa](images/solution39/Active-passive.png)

As solicitações são entregues por meio do site ativo. No caso de uma indisponibilidade ou falha do aplicativo, o trabalho de pré-aplicativo é executado para tornar o data center de espera pronto para entregar a solicitação. Alternar do data center ativo para o passivo é uma operação demorada. O **objetivo do tempo de recuperação** (RTO) e o **objetivo do ponto de recuperação** (RPO) são superiores em comparação com a configuração ativa/ativa.

### Recuperação de desastre com três regiões

Na era de hoje de serviços "Sempre ativados" com tolerância zero para tempo de inatividade, os clientes esperam que cada serviço de negócios permaneça acessível ininterruptamente em qualquer lugar do mundo. Uma estratégia com custo reduzido para empresas envolve arquitetar sua infraestrutura para disponibilidade contínua em vez de construir infraestruturas de recuperação de desastre.

O uso de três data centers fornece maior resiliência e disponibilidade do que dois. Também pode oferecer melhor desempenho ao difundir a carga mais uniformemente entre os data centers. Se a empresa tiver somente dois data centers, uma variante disso será implementar dois aplicativos em um data center e implementar o terceiro aplicativo no segundo data center. Como alternativa, é possível implementar a lógica de negócios e as camadas de apresentação na topologia de 3 ativos e implementar a camada de dados na topologia 2 de ativos.

#### Configuração ativa/ativa/ativa (3 ativos)

![](images/solution39/Active-active-active.png)

As solicitações são entregues pelo aplicativo em execução em qualquer um dos três data centers ativos. Um estudo de caso no website IBM.com indica que 3 ativos requerem somente 50% da capacidade de cálculo, memória e rede por cluster, mas 2 ativos requerem 100% por cluster. A camada de dados é onde a diferença de custo se destaca. Para obter detalhes adicionais, leia [*Sempre ativado: avaliar, projetar, implementar e gerenciar a disponibilidade contínua*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf).

#### Configuração ativa/ativa/passiva

![](images/solution39/Active-active-passive.png)

Nesse cenário, quando um dos dois aplicativos ativos nos data centers primário e secundário sofre uma indisponibilidade, o aplicativo de espera no terceiro centro de dados é ativado. O procedimento de recuperação de desastre descrito no cenário de dois data centers é seguido para restaurar a normalidade para processar solicitações do cliente. O aplicativo de espera no terceiro data center pode ser definido em uma configuração de espera a quente ou a frio.

Consulte [este guia](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/) para saber mais sobre recuperação de desastre.

### Arquiteturas multiregion

Em uma arquitetura multiregion, um aplicativo é implementado em locais diferentes nos quais cada região executa uma cópia idêntica do aplicativo. 

Uma região é uma localização geográfica específica na qual é possível implementar apps, serviços e outros recursos do {{site.data.keyword.cloud_notm}}. As [regiões do {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/containers?topic=containers-regions-and-zones) consistem em uma ou mais zonas, que são data centers físicos que hospedam os recursos de cálculo, rede e armazenamento e o resfriamento e a energia relacionados que hospedam os serviços e aplicativos. As zonas são isoladas umas das outras, o que assegura nenhum ponto único de falha compartilhado.

Além disso, em uma arquitetura multiregion, um balanceador de carga global como o [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services) é necessário para distribuir o tráfego entre as regiões.

A implementação de uma solução em múltiplas regiões vem com os benefícios a seguir:
- Melhorar a latência para usuários finais - a velocidade é a chave, quanto mais perto sua origem de back-end estiver dos usuários finais, melhor e mais rápida a experiência para os usuários.
- Recuperação de desastre - quando a região ativa falha, você tem uma região de backup para recuperar rapidamente.
- Requisitos de negócios - em alguns casos, é necessário armazenar dados em regiões distintas, separadas por várias centenas de quilômetros. Portanto, aqueles que nesse caso você precisa armazenar dados em múltiplas regiões. 

### Multizonas dentro de arquiteturas de regiões

Construir aplicativos de regiões multizona significa ter seu aplicativo implementado em zonas dentro de uma região e, em seguida, você também pode ter duas ou três regiões. 

Com a arquitetura de região multizona, é necessário ter um balanceador de carga local para distribuir o tráfego localmente entre as zonas em uma região e, em seguida, se uma segunda região for configurada, um balanceador de carga global distribuirá tráfego entre as regiões. 

É possível aprender mais sobre regiões e zonas [aqui](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones).

## Opções de cálculo 

Esta seção revisa as opções de cálculo disponíveis no {{site.data.keyword.cloud_notm}}. Para cada opção de cálculo, um diagrama de arquitetura é fornecido junto com um tutorial sobre como implementar tal arquitetura.

Nota: todas as arquiteturas de opções de cálculo não têm bancos de dados ou outros serviços incluídos, elas se concentram somente na implementação de um app em duas regiões para a opção de cálculo selecionada. Depois de implementar qualquer um dos exemplos de opções de cálculo de multiregion, a próxima etapa lógica seria incluir bancos de dados e outros serviços. As seções posteriores deste tutorial de solução abrangerão [bancos de dados](#databaseservices) e [serviços não de banco de dados](#nondatabaseservices).

### Cloud Foundry 

O Cloud Foundry oferece a capacidade de alcançar a implementação de uma arquitetura multiregion, além disso, usar um serviço de pipeline de [entrega contínua](https://{DomainName}/catalog/services/continuous-delivery) permite implementar seu aplicativo em múltiplas regiões. A arquitetura para multiregion do Cloud Foundry é semelhante a esta:

![Arquitetura do CF](images/solution39/CF2-Architecture.png)

O mesmo aplicativo é implementado em múltiplas regiões e um balanceador de carga global roteia o tráfego para a região mais próxima e funcional. O tutorial [**Aplicativo da web seguro em múltiplas regiões**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp) conduz você na implementação de uma arquitetura semelhante.

### {{site.data.keyword.cfee_full_notm}}

O **{{site.data.keyword.cfee_full_notm}} (CFEE)** oferece todas as mesmas funcionalidades que o Cloud Foundry público juntamente com recursos adicionais.

O **{{site.data.keyword.cfee_full_notm}}** permite instanciar múltiplas plataformas Cloud Foundry isoladas e de classificação on demand. As instâncias do CFEE são executadas dentro de sua própria conta em [{{site.data.keyword.cloud_notm}}
](http://ibm.com/cloud). O ambiente é implementado no hardware isolado em cima [do {{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service). Você tem controle total no ambiente, incluindo controle de acesso, gerenciamento da capacidade, gerenciamento de mudanças, monitoramento e serviços.

Uma arquitetura multiregion usando o {{site.data.keyword.cfee_full_notm}} está abaixo.

![Arquitetura](images/solution39/CFEE-Architecture.png)

A implementação dessa arquitetura requer o seguinte: 

- Configurar duas instâncias do CFEE - uma em cada região.
- Criar e ligar os serviços à conta do CFEE. 
- Enviar os apps por push com destino ao terminal da API do CFEE. 
- Configurar a replicação do banco de dados, assim como você faria com o Cloud Foundry público. 

Além disso, verifique o guia passo a passo [Assistente de implementação de logística para o Cloud Foundry Enterprise Environment (CFEE)](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md). Ele o conduzirá na implementação de um aplicativo baseado em microsserviço para o CFEE. Depois de implementado em uma instância do CFEE, é possível replicar o procedimento para uma segunda região e anexar os [Serviços da Internet](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) na frente das duas instâncias do CFEE para balancear a carga do tráfego. 

Consulte a [documentação do {{site.data.keyword.cfee_full_notm}}](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about) para obter detalhes adicionais.

### Kubernetes

Com o Kubernetes, é possível alcançar uma arquitetura de multizona dentro de regiões, esse pode ser um caso de uso ativo/ativo. Ao implementar uma solução com o {{site.data.keyword.containershort_notm}}, você se beneficia de recursos integrados, como balanceamento de carga e isolamento, maior resiliência com relação a falhas potenciais com hosts, redes ou apps. Ao criar vários clusters e se uma indisponibilidade ocorrer com um cluster, os usuários ainda poderão acessar um app que também esteja implementado em outro cluster. Com múltiplos clusters em regiões diferentes, os usuários também podem acessar o cluster mais próximo com a latência de rede reduzida. Para resiliência adicional, você tem a opção de selecionar também os clusters de multizona, o que significa que seus nós são implementados em múltiplas zonas dentro de uma região. 

A arquitetura multiregion do Kubernetes é semelhante a esta.

![Kubernetes](images/solution39/Kub-Architecture.png)

1. O desenvolvedor constrói as imagens do Docker para o aplicativo.
2. As imagens são enviadas por push para o {{site.data.keyword.registryshort_notm}} em dois locais diferentes.
3. O aplicativo é implementado em clusters Kubernetes em ambas as localizações.
4. Os usuários finais acessam o aplicativo.
5. O Cloud Internet Services é configurado para interceptar as solicitações ao aplicativo e distribuir a carga entre os clusters. Além disso, o DDoS Protection e o Web Application Firewall são ativados para proteger o aplicativo de ameaças comuns. Opcionalmente, ativos como imagens, arquivos CSS são armazenados em cache.

O tutorial [**Clusters Kubernetes de multiregion resilientes e seguros com o Cloud Internet Services**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis) conduz você nas etapas para implementar tal arquitetura.

### {{site.data.keyword.openwhisk_short}}

O {{site.data.keyword.openwhisk_short}} está disponível em várias localizações do {{site.data.keyword.cloud_notm}}. Para aumentar a resiliência e reduzir a latência de rede, os aplicativos podem implementar seu back-end em várias localizações. Em seguida, com o IBM Cloud Internet Services (CIS), os desenvolvedores podem expor um ponto de entrada único encarregado de distribuir o tráfego para o back-end funcional mais próximo. A arquitetura para multiregion do {{site.data.keyword.openwhisk_short}} é semelhante a esta.

 ![Arquitetura de funções](images/solution39/Functions-Architecture.png)

1. Os usuários acessam o aplicativo. A solicitação passa pelo Internet Services.
2. O Internet Services redireciona os usuários para o back-end de API funcional mais próximo.
3. O Certificate Manager fornece a API com seu certificado SSL. O tráfego é criptografado de ponta a ponta.
4. A API é implementada com o Cloud Functions.

Descubra como implementar essa arquitetura seguindo o tutorial [**Implementar apps serverless em múltiplas regiões**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless).

### {{site.data.keyword.baremetal_short}} e {{site.data.keyword.virtualmachinesshort}}

O {{site.data.keyword.virtualmachinesshort}} e o {{site.data.keyword.baremetal_short}} oferecem a capacidade para alcançar uma arquitetura multiregion. É possível provisionar servidores em muitos locais disponíveis no {{site.data.keyword.cloud_notm}}.

![locais do servidor](images/solution39/ServersLocation.png)

Ao preparar-se para tal arquitetura usando o {{site.data.keyword.virtualmachinesshort}} e o {{site.data.keyword.baremetal_short}}, considere o seguinte: armazenamento de arquivo, backups, recuperação e bancos de dados, selecionando entre um banco de dados como serviço ou instalando um banco de dados em um servidor virtual. 

A arquitetura abaixo demonstra a implementação de uma arquitetura multiregion usando o {{site.data.keyword.virtualmachinesshort}} em uma arquitetura ativa/passiva em que uma região é ativa e a segunda região é passiva. 

![Arquitetura de VM](images/solution39/vm-Architecture2.png)

Os componentes necessários para tal arquitetura: 

1. Os usuários acessam o aplicativo por meio do IBM Cloud Internet Services (CIS).
2. O CIS roteia o tráfego para o local ativo.
3. Dentro de uma localização, um balanceador de carga redireciona o tráfego para um servidor.
4. Bancos de dados implementados em um servidor virtual, o que significa que você configuraria o banco de dados e configuraria replicações e backups entre regiões. A alternativa seria usar um banco de dados como serviço, um tópico discutido posteriormente no tutorial.
5. Armazenamento de arquivo para armazenar as imagens e os arquivos do aplicativo. O armazenamento de arquivo oferece a capacidade de tomar uma captura instantânea em um determinado horário e data, essa captura instantânea pode, então, ser reutilizada em outra região, algo que você faria manualmente. 

O tutorial [**Usar Virtual Servers para construir um app da web altamente disponível e escalável**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application) implementa essa arquitetura.

## Bancos de dados e arquivos de aplicativo
{: #databaseservices}

O {{site.data.keyword.cloud_notm}} oferece uma seleção [de bancos de dados como um serviço](https://{DomainName}/catalog/?category=databases) com bancos de dados relacionais e não relacionais, dependendo de suas necessidades de negócios. O [banco de dados como um serviço (DBaaS)](https://www.ibm.com/cloud/learn/what-is-cloud-database) vem com muitas vantagens. Usando um DBaaS como {{site.data.keyword.cloudant}}, é possível obter vantagens do suporte de multiregion, permitindo que você faça replicação em tempo real entre dois serviços de banco de dados em regiões diferentes, executar backups e ter ajuste de escala e tempo de atividade máximo. 

**Recursos-chave:** 

- Um serviço de banco de dados construído e acessado por meio de uma plataforma de nuvem
- Permite que usuários corporativos hospedem bancos de dados sem comprar hardware dedicado
- Pode ser gerenciado pelo usuário ou oferecido como um serviço e gerenciado por um provedor
- Pode suportar bancos de dados SQL ou NoSQL
- Acessado por meio de uma interface da web ou de uma API fornecida pelo fornecedor

**Preparando-se para a arquitetura multiregion:**

- Quais são as opções de resiliência do serviço de banco de dados?
- Como a replicação é manipulada entre múltiplos serviços de banco de dados entre regiões?
- Como os dados são submetidos a backup?
- Quais são as abordagens de recuperação de desastre para cada um?

### {{site.data.keyword.cloudant}}

O {{site.data.keyword.cloudant}} é um banco de dados distribuído que é otimizado para manipulação de cargas de trabalho pesadas que são típicas de apps móveis e da web grandes e de rápido crescimento. Disponível como um serviço do {{site.data.keyword.Bluemix_notm}} totalmente gerenciado, suportado por ANS, o {{site.data.keyword.cloudant}} escala elasticamente o rendimento e o armazenamento independentemente. O {{site.data.keyword.cloudant}} também está disponível como uma instalação no local transferível por download e sua API e seu protocolo de replicação poderoso são compatíveis com um ecossistema de software livre que inclui o CouchDB, o PouchDB e as bibliotecas para as pilhas de desenvolvimento móvel e da web mais populares.

O {{site.data.keyword.cloudant}} suporta a [replicação](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation) entre múltiplas instâncias nos locais. Qualquer mudança que ocorreu no banco de dados de origem é reproduzida no banco de dados de destino. É possível criar replicações entre qualquer número de bancos de dados, seja continuamente ou como uma tarefa 'única'. O diagrama a seguir mostra uma configuração típica que usa duas instâncias do {{site.data.keyword.cloudant}}, uma em cada região:

![ativa-ativa](images/solution39/Active-active.png)

Consulte [estas instruções](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery) para configurar a replicação entre as instâncias do {{site.data.keyword.cloudant}}. O serviço também fornece instruções e conjunto de ferramentas para [fazer backup e restaurar dados](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery).

### {{site.data.keyword.Db2_on_Cloud_short}}, {{site.data.keyword.dashdbshort_notm}} e {{site.data.keyword.Db2Hosted_notm}}

O {{site.data.keyword.cloud_notm}} oferece vários [serviços de banco de dados Db2](https://{DomainName}/catalog/?search=db2h). Eles são:

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2): um banco de dados SQL de nuvem totalmente gerenciado para típicas cargas de trabalho operacionais semelhantes ao OLTP.
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse): um serviço de data warehouse em nuvem totalmente gerenciado para cargas de trabalho analíticas de alto desempenho e de escala de petabyte. Ele oferece planos de serviços SMP e MPP e utiliza um armazenamento de dados colunar otimizado e processamento em memória.
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted): um hospedado pela IBM e gerenciado pelo sistema de banco de dados do usuário. Ele fornece o Db2 com acesso administrativo total na infraestrutura em nuvem, eliminando, assim, o custo, a complexidade e o risco de gerenciar sua própria infraestrutura.

A seguir, nos concentraremos no {{site.data.keyword.Db2_on_Cloud_short}} como DBaaS para cargas de trabalho operacionais. Essas cargas de trabalho são típicas para os aplicativos discutidos neste tutorial.

#### Suporte de multiregion para o {{site.data.keyword.Db2_on_Cloud_short}}

O {{site.data.keyword.Db2_on_Cloud_short}} oferece várias [opções para alcançar o High Availability and Disaster Recovery (HADR)](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview). É possível escolher a opção Alta disponibilidade ao criar um novo serviço. Mais tarde, é possível [incluir um Nó de recuperação de desastre replicado geograficamente](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha) por meio do painel da instância. A opção de nó DR externo fornece a capacidade de sincronizar seus dados em tempo real para um nó de banco de dados em um data center do {{site.data.keyword.cloud_notm}} externo de sua escolha.

Mais informações estão disponíveis na [Documentação de Alta Disponibilidade](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha).

#### Backup e restauração

O {{site.data.keyword.Db2_on_Cloud_short}} inclui backups diários para planos pagos. Geralmente, os backups são armazenados usando o {{site.data.keyword.cos_short}} e, portanto, utilizando três data centers para maior disponibilidade de dados retidos. Os backups são mantidos por 14 dias. É possível usá-los para executar uma recuperação point-in-time. A [documentação de backup e restauração](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br) fornece detalhes sobre como é possível restaurar dados para a data e hora desejadas.

### {{site.data.keyword.databases-for}}

O {{site.data.keyword.databases-for}} oferece vários sistemas de banco de dados de software livre como serviços totalmente gerenciados. Eles são:  
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

Todos esses serviços compartilham as mesmas características:   
* Para alta disponibilidade, eles são implementados em clusters. Os detalhes podem ser localizados na documentação de cada serviço:
  - [PostgreSQL](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* Cada cluster é difundido em múltiplas zonas.
* Os dados são replicados entre as zonas.
* Os usuários podem aumentar a capacidade dos recursos de armazenamento e de memória para uma instância. Consulte a [documentação sobre ajuste de escala para, por exemplo, {{site.data.keyword.databases-for-redis}}](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings) para obter detalhes.
* Os backups são tomados diariamente ou on demand. Os detalhes são documentados para cada serviço. Aqui está a [documentação de backup para, por exemplo, {{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups).
* Os dados em repouso, os backups e o tráfego de rede são criptografados.
* Cada [serviço pode ser gerenciado usando o plug-in da CLI do {{site.data.keyword.databases-for}}](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in)


### {{site.data.keyword.cos_full_notm}}

O {{site.data.keyword.cos_full_notm}} (COS) fornece armazenamento em nuvem durável, seguro e com custo reduzido. As informações armazenadas com o {{site.data.keyword.cos_full_notm}} são criptografadas e dispersas em múltiplos locais geográficos. Ao criar depósitos de armazenamento dentro de uma instância do COS, você decide em qual local o depósito deve ser criado e qual opção de resiliência usar.

Há três tipos de resiliência de depósito:
   - A resiliência de **Região cruzada** distribuirá seus dados em várias áreas metropolitanas. Isso pode ser visto como uma opção de multiregion. Ao acessar o conteúdo armazenado em um depósito de Região cruzada, o COS oferece um terminal especial capaz de recuperar o conteúdo de uma região funcional.
   - A resiliência **Regional** difundirá dados em uma única área metropolitana. Isso pode ser visto como uma multizona dentro de uma configuração de região.
   - A resiliência de **Data center único** difunde dados em múltiplos dispositivos dentro de um único data center.

Consulte [esta documentação](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints) para obter uma explicação detalhada de opções de resiliência do {{site.data.keyword.cos_full_notm}}.

### {{site.data.keyword.filestorage_full_notm}}

O {{site.data.keyword.filestorage_full_notm}} é um armazenamento de arquivo baseado em NFS persistente, rápido e conectado à rede flexível. Nesse ambiente de armazenamento conectado à rede (NAS), você tem controle total sobre sua função de compartilhamentos de arquivo e sobre o desempenho. Os compartilhamentos do {{site.data.keyword.filestorage_short}} podem ser conectados a até 64 dispositivos autorizados sobre conexões TCP/IP roteadas para resiliência.

Alguns dos recursos de armazenamento de arquivo são _Capturas instantâneas_, _Replicação_, _Acesso simultâneo_. Consulte [a documentação](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage) para obter uma lista integral de recursos.

Uma vez conectado aos seus servidores, um serviço {{site.data.keyword.filestorage_short}} pode ser usado facilmente para armazenar backups de dados, arquivos de aplicativo como imagens e vídeos, essas imagens e arquivos podem, então, ser usados dentro de diferentes servidores na mesma região.

Ao incluir uma segunda região, é possível usar o recurso de capturas instantâneas do {{site.data.keyword.filestorage_short}} para tomar uma captura instantânea automaticamente ou manualmente e, em seguida, reutilizá-la na segunda região passiva. 

A replicação pode ser planejada para copiar capturas instantâneas automaticamente para um volume de destino em um data center remoto. As cópias poderão ser recuperadas no site remoto se ocorrer um evento catastrófico ou se os dados forem corrompidos. Mais informações sobre capturas instantâneas do File Storage podem ser localizadas [aqui](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots).

## Serviços não de banco de dados
{: #nondatabaseservices}

O {{site.data.keyword.cloud_notm}} oferece uma seleção de [serviços](https://{DomainName}/catalog) não de banco de dados, esses são serviços da IBM e serviços de terceiros. Ao planejar a arquitetura multiregion, é necessário entender como os serviços, como os serviços do Watson, podem funcionar em uma configuração de multiregion.

### {{site.data.keyword.conversationfull}}

O [{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about) é uma plataforma que permite que desenvolvedores e usuários não técnicos colaborem na construção de assistentes de conversação desenvolvidos com IA.

Um assistente é um robô cognitivo que pode ser customizado para suas necessidades de negócios e implementado em múltiplos canais para trazer ajuda para seus clientes onde e quando eles precisarem. O assistente inclui uma ou muitas qualificações. Uma qualificação de diálogo contém os dados de treinamento e a lógica que permite que um assistente ajude seus clientes.

É importante observar que o {{site.data.keyword.conversationshort}} V1 é stateless. O {{site.data.keyword.conversationshort}} entrega 99,5% de tempo de atividade, mas ainda, para aplicativos altamente disponíveis em múltiplas regiões, você pode até mesmo desejar ter múltiplas instâncias desses serviços entre regiões. 

Depois de ter criado instâncias em múltiplos locais, use o {{site.data.keyword.conversationshort}} do conjunto de ferramentas para exportar, de uma instância, uma área de trabalho existente, incluindo intentos, entidades e diálogo. Em seguida, importe essa área de trabalho em outros locais.

## Resumo

| Oferta | Opções de resiliência |
| -------- | ------------------ |
| Cloud Foundry | <ul><li>Implantar aplicativos em diversas localizações</li><li>Entregue solicitações de várias localizações com o Cloud Internet Services</li><li>Use as APIs do Cloud Foundry para configurar orgs, espaços e apps de push para várias localizações</li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>Implantar aplicativos em diversas localizações</li><li>Entregue solicitações de várias localizações com o Cloud Internet Services</li><li>Use as APIs do Cloud Foundry para configurar orgs, espaços e apps de push para várias localizações</li><li>Construído no serviço Kubernetes</li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>Resiliência por design com suporte para clusters de multizona</li><li>Entregar solicitações de clusters difundidos em múltiplos locais com o Cloud Internet Services</li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>Disponível em múltiplos locais</li><li>Entregue solicitações de várias localizações com o Cloud Internet Services</li><li>Use a API do Cloud Functions para implementar ações em múltiplos locais</li></ul> |
| {{site.data.keyword.baremetal_short}} e {{site.data.keyword.virtualmachinesshort}} | <ul><li>Provisionar servidores em múltiplos locais</li><li>Conectar servidores no mesmo local a um balanceador de carga local</li><li>Entregue solicitações de várias localizações com o Cloud Internet Services</li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>Uma captura e replicação contínua entre bancos de dados</li><li>Redundância automática de dados em uma região</li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>Provisionar um nó de recuperação de desastre replicado geograficamente para sincronização de dados em tempo real</li><li>Backup diário com planos pagos</li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>Construído em clusters Kubernetes de multizona</li><li>Réplicas de leitura de região cruzada</li><li>Backups diários e on demand</li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>Resiliência de data center único, regional e regional cruzada</li><li>Use a API para sincronizar o conteúdo entre os depósitos de armazenamento</li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>Use capturas instantâneas para capturar conteúdo automaticamente para um destino em um data center remoto</li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>Use a API do Watson para exportar e importar a especificação da área de trabalho entre múltiplas instâncias nos locais</li></ul> |

## Conteúdo relacionado

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [Melhorando a disponibilidade do app com clusters multizonas](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry, aplicativo da web seguro em múltiplas regiões](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [Cloud Functions, implementar apps serverless em múltiplas regiões](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes, clusters Kubernetes de multiregion resilientes e seguros com o Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [Virtual Servers, construir app da web escalável e altamente disponível](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
