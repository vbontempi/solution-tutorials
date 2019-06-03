---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# Melhores práticas para organizar usuários, equipes, aplicativos
{: #users-teams-applications}

Este tutorial fornece uma visão geral dos conceitos disponíveis no {{site.data.keyword.cloud_notm}} para fazer o gerenciamento de identidade e de acesso e como eles podem ser implementados para suportar os múltiplos estágios de desenvolvimento de um aplicativo.
{:shortdesc}

Ao construir um aplicativo, é muito comum definir múltiplos ambientes que refletem o ciclo de vida de desenvolvimento de um projeto de um desenvolvedor que está confirmando código para o código do aplicativo que está sendo disponibilizado para os usuários finais. *Ambiente de simulação*, *teste*, *preparação*, *UAT* (user acceptance testing), *pré-produção*, *pré-produção* são nomes típicos para esses ambientes.

Isolar os recursos subjacentes, implementar políticas de governança e acesso, proteger uma carga de trabalho de produção, validar mudanças antes de enviá-las por push para produção são algumas das razões pelas quais você desejaria criar esses ambientes separados.

## Objetivos
{: #objectives}

* Aprender sobre os modelos de acesso do {{site.data.keyword.iamlong}} e do Cloud Foundry
* Configurar um projeto com separação entre funções e ambientes
* Configurar a integração contínua

## Serviços usados
{: #services}

Este tutorial usa os tempos de execução e serviços a seguir:
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Definir um projeto

Vamos considerar um projeto de amostra com os componentes a seguir:
* vários microsserviços implementados no {{site.data.keyword.containershort_notm}},
* bancos de dados,
* depósitos de armazenamento de arquivo.

Nesse projeto, definimos três ambientes:
* *Desenvolvimento* - esse ambiente é atualizado continuamente com cada confirmação, testes de unidade e testes de fumaça que são executados. Ele fornece acesso à maior e mais recente implementação do projeto.
* *Teste* - esse ambiente é construído após uma ramificação ou tag estável do código. Aqui é onde o teste de aceitação de usuário é feito. Sua configuração é semelhante ao ambiente de produção. Ele é carregado com dados realistas (dados de produção anonimizados, por exemplo).
* *Produção* - esse ambiente é atualizado com a versão validada no ambiente anterior.

**Um pipeline de entrega gerencia a progressão de uma construção por meio do ambiente.** Ele pode ser totalmente automatizado ou incluir portas de validação manual para promover construções aprovadas entre ambientes - isso é realmente aberto e deve ser configurado para corresponder às melhores práticas e fluxos de trabalho da empresa.

Para suportar a execução do pipeline de construção, introduzimos **um usuário funcional** - um usuário regular do {{site.data.keyword.cloud_notm}}, mas um membro da equipe sem nenhuma identidade real no mundo físico. Esse usuário funcional será proprietário dos pipelines de entrega e de quaisquer outros recursos em nuvem que requeiram uma propriedade forte. Essa abordagem ajuda no caso em que um membro da equipe deixa a empresa ou está se movendo para outro projeto. O usuário funcional será dedicado ao seu projeto e não mudará no tempo de vida do projeto. A próxima coisa que você desejará criar é [uma chave de API](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey) para esse usuário funcional. Você selecionará essa chave de API ao configurar os pipelines do DevOps ou quando desejar executar scripts de automação, para personificar o usuário funcional.

Quando se trata de designar responsabilidades para os membros da equipe do projeto, vamos definir as funções a seguir e as permissões relacionadas:

|           | Desenvolvimento | Teste | Produção |
| --------- | ----------- | ------- | ---------- |
| Desenvolvedor | <ul><li>contribui com código</li><li>pode acessar arquivos de log</li><li>pode visualizar a configuração do app e do serviço</li><li>usar os aplicativos implementados</li></ul> | <ul><li>pode acessar arquivos de log</li><li>pode visualizar a configuração do app e do serviço</li><li>usar os aplicativos implementados</li></ul> | <ul><li>sem acesso</li></ul> |
| Testador    | <ul><li>usar os aplicativos implementados</li></ul> | <ul><li>usar os aplicativos implementados</li></ul> | <ul><li>sem acesso</li></ul> |
| Operador  | <ul><li>pode acessar arquivos de log</li><li>pode visualizar/definir a configuração do app e do serviço</li></ul> | <ul><li>pode acessar arquivos de log</li><li>pode visualizar/definir a configuração do app e do serviço</li></ul> | <ul><li>pode acessar arquivos de log</li><li>pode visualizar/definir a configuração do app e do serviço</li></ul> |
| Usuário funcional de pipeline  | <ul><li>pode implementar/remover implementação de aplicativos</li><li>pode visualizar/definir a configuração do app e do serviço</li></ul> | <ul><li>pode implementar/remover implementação de aplicativos</li><li>pode visualizar/definir a configuração do app e do serviço</li></ul> | <ul><li>pode implementar/remover implementação de aplicativos</li><li>pode visualizar/definir a configuração do app e do serviço</li></ul> |

## Identity and Access Management (IAM)
{: #first_objective}

O {{site.data.keyword.iamshort}} (IAM) permite autenticar com segurança os usuários para os serviços de plataforma e de infraestrutura e controlar o acesso a **recursos** de forma consistente em toda a plataforma {{site.data.keyword.cloud_notm}}. Um conjunto de serviços do {{site.data.keyword.cloud_notm}} é ativado para usar o Cloud IAM para controle de acesso e é organizado em **grupos de recursos** em sua **conta** para permitir que os **usuários** tenham acesso rápido e fácil a mais de um recurso de cada vez. As **políticas** de acesso do Cloud IAM são usadas para designar aos usuários e aos IDs de serviço acesso para os recursos dentro de sua conta.

Uma **política** designa a um ID de usuário ou serviço uma ou mais **funções** com uma combinação de atributos que definem o escopo de acesso. A política pode fornecer acesso a um único serviço até o nível da instância ou a política pode se aplicar a um conjunto de recursos organizados juntos em um grupo de recursos. Dependendo das funções de usuário que você designa, o ID de usuário ou serviço é permitido em níveis variados de acesso para concluir tarefas de gerenciamento de plataforma ou acessar um serviço usando a IU ou executando tipos específicos de chamadas da API.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="Diagrama do modelo IAM" />
</p>

Neste momento, nem todos os serviços no catálogo do {{site.data.keyword.cloud_notm}} podem ser gerenciados usando o IAM. Para esses serviços, é possível continuar usando o Cloud Foundry, fornecendo aos usuários acesso à organização e ao espaço ao qual a instância pertence com uma função do Cloud Foundry designada para definir o nível de acesso permitido.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Diagrama do modelo Cloud Foundry" />
</p>

## Criar os recursos para um ambiente

Embora os três ambientes necessários para esse projeto de amostra requeiram diferentes direitos de acesso e possam precisar ser alocados a diferentes capacidades, eles compartilham um padrão de arquitetura comum.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="Diagrama de arquitetura mostrando um ambiente" />
</p>

Vamos iniciar construindo o ambiente de Desenvolvimento.

1. [Selecione um local do {{site.data.keyword.cloud_notm}}](https://{DomainName}) no qual implementar o ambiente.
1. Para serviços e apps do Cloud Foundry:
   1. [Crie uma organização para o projeto](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg).
   1. [Crie um espaço do Cloud Foundry para o ambiente](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo).
   1. Crie os serviços do Cloud Foundry usados pelo projeto sob esse espaço
1. [Crie um grupo de recursos para o ambiente](https://{DomainName}/account/resource-groups).
1. Crie os serviços compatíveis com o grupo de recursos como {{site.data.keyword.cos_full_notm}}, {{site.data.keyword.la_full_notm}}, {{site.data.keyword.mon_full_notm}} e {{site.data.keyword.cloudant_short_notm}} nesse grupo.
1. [Crie um novo cluster Kubernetes](https://{DomainName}/containers-kubernetes/catalog/cluster) no {{site.data.keyword.containershort_notm}}, certifique-se de selecionar o grupo de recursos criado acima.
1. Configure o {{site.data.keyword.la_full_notm}} e o {{site.data.keyword.mon_full_notm}} para enviar logs e monitorar o cluster.

O diagrama a seguir mostra onde os recursos do projeto são criados sob a conta:

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="Diagrama mostrando os recursos do projeto" />
</p>

## Designar funções dentro do ambiente

1. Convide usuários para a conta
1. Designe Políticas aos usuários para controlar quem pode acessar o grupo de recursos, os serviços dentro do grupo e a instância do {{site.data.keyword.containershort_notm}} e suas permissões. Consulte a [definição de política de acesso](https://{DomainName}/docs/containers?topic=containers-users#access_policies) para selecionar as políticas de direito para um usuário no ambiente. Os usuários com o mesmo conjunto de políticas podem ser colocados no [mesmo grupo de acesso](https://{DomainName}/docs/iam?topic=iam-groups#groups). Isso simplifica o gerenciamento de usuários, pois as políticas serão designadas ao grupo de acesso e herdadas por todos os usuários no grupo.
1. Configure suas funções de organização e espaço do Cloud Foundry com base em suas necessidades dentro do ambiente. Consulte a [definição de função](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess) para designar as funções de direito com base no ambiente.

Consulte a documentação de serviços para entender como um serviço está mapeando as funções do IAM e do Cloud Foundry para ações específicas. Consulte, por exemplo, [como o serviço {{site.data.keyword.mon_full_notm}} mapeia funções do IAM para ações](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam).

A designação das funções de direito aos usuários requererá várias iterações e refinamento. As permissões dadas podem ser controladas no nível do grupo de recursos, para todos os recursos em um grupo ou em baixa granularidade para uma instância específica de um serviço, você descobrirá ao longo do tempo quais são as políticas de acesso ideais para seu projeto.

Uma boa prática é iniciar com o conjunto mínimo de permissões e, em seguida, expandir cuidadosamente conforme necessário. Para o Kubernetes, você desejará consultar seu [Controle de Acesso Baseado na Função (RBAC)](https://kubernetes.io/docs/admin/authorization/rbac/) para configurar autorizações em cluster.

Para o ambiente de Desenvolvimento, as responsabilidades do usuário definidas anteriormente podem ser convertidas para o seguinte:

|           | Políticas de acesso do IAM | Cloud Foundry |
| --------- | ----------- | ------- |
| Desenvolvedor | <ul><li>Grupo de recursos: *Visualizador*</li><li>Funções de acesso da plataforma no grupo de recursos: *Visualizador*</li><li>Criação de log e Função de serviço de monitoramento: *Escritor*</li></ul> | <ul><li>Função de organização: *Auditor*</li><li>Função de espaço: *Auditor*</li></ul> |
| Testador    | <ul><li>Nenhuma configuração necessária. O testador acessa o aplicativo implementado, não os ambientes de desenvolvimento</li></ul> | <ul><li>Nenhuma configuração necessária</li></ul> |
| Operador  | <ul><li>Grupo de recursos: *Visualizador*</li><li>Funções de acesso da plataforma no grupo de recursos: *Operador*, *Visualizador*</li><li>Criação de log e Função de serviço de monitoramento: *Escritor*</li></ul> | <ul><li>Função de organização: *Auditor*</li><li>Função de espaço: *Desenvolvedor*</li></ul> |
| Usuário funcional de pipeline | <ul><li>Grupo de recursos: *Visualizador*</li><li>Funções de acesso da plataforma no grupo de recursos: *Editor*, *Visualizador*</li></ul> | <ul><li>Função de organização: *Auditor*</li><li>Função de espaço: *Desenvolvedor*</li></ul> |

As políticas de acesso do IAM e as funções do Cloud Foundry são definidas na [Interface com o usuário do Identify and Access Management](https://{DomainName}/iam/#/users):

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="Configuração de permissões para a função de desenvolvedor" />
</p>

## Replicar para múltiplos ambientes

De lá, é possível replicar etapas semelhantes para construir os outros ambientes.

1. Crie um grupo de recursos por ambiente.
1. Crie um cluster e instâncias de serviço necessárias por ambiente.
1. Crie um espaço do Cloud Foundry por ambiente.
1. Crie as instâncias de serviço necessárias em cada espaço.

<p style="text-align: center;">
  <img title="Usando clusters separados para isolar ambientes" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="Diagrama mostrando clusters separados para isolar ambientes" />
</p>

Usando uma combinação de ferramentas como a [CLI `ibmcloud` do {{site.data.keyword.cloud_notm}}](https://github.com/IBM-Cloud/ibm-cloud-developer-tools), o [`terraform` do HashiCorp](https://www.terraform.io/), o [{{site.data.keyword.cloud_notm}} Provider for Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm), a CLI do Kubernetes `kubectl`, é possível criar script e automatizar a criação desses ambientes.

Os clusters Kubernetes separados para os ambientes vêm com boas propriedades:
* Não importa o ambiente, todos os clusters tenderão a ser os mesmos;
* é mais fácil controlar quem tem acesso a um cluster específico;
* fornece flexibilidade nos ciclos de atualização para implementações e recursos subjacentes; quando há uma nova versão do Kubernetes, ela fornece a opção para atualizar primeiro o cluster de Desenvolvimento, validar seu aplicativo e, em seguida, atualizar o outro ambiente;
* evita a combinação de cargas de trabalho diferentes que podem afetar umas às outras, como isolar a implementação de produção das outras.

Outra abordagem é usar [namespaces do Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) em conjunto com [Cotas de recurso do Kubernetes](https://kubernetes.io/docs/concepts/policy/resource-quotas/) para isolar ambientes e controlar o consumo de recursos.

<p style="text-align: center;">
  <img title="Usando namespaces separados para isolar ambientes" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="Diagrama mostrando namespaces separados para isolar ambientes" />
</p>

Na caixa de entrada `Search` da IU do LogDNA, use o campo `namespace:` para filtrar os logs com base no namespace.
{: tip}

## Configurar o pipeline de entrega

Quando se trata de implementar nos diferentes ambientes, seu pipeline de integração contínua/entrega contínua pode ser configurado para conduzir o processo integral:
* atualize continuamente o ambiente `Development` com o maior e mais recente código da ramificação de `development`, executando testes de unidade e testes de integração no cluster dedicado;
* promova construções de desenvolvimento para o ambiente `Testing`, automaticamente se todos os testes dos estágios anteriores estiverem OK ou por meio de um processo de promoção manual. Algumas equipes usarão ramificações diferentes também aqui, mesclando o estado de desenvolvimento de trabalho para uma ramificação `stable` como exemplo;
* repita um processo semelhante para mover-se para o ambiente `Production`.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="Um pipeline de CI/CD da construção à implementação" />
</p>

Ao configurar o pipeline do DevOps, certifique-se de usar a chave de API de um usuário funcional. Somente o usuário funcional deve precisar ter os direitos necessários para implementar apps em seus clusters.

Durante a fase de construção, é importante criar a versão adequada das imagens do Docker. É possível usar a revisão de confirmação do Git como parte da tag de imagem ou um identificador exclusivo fornecido pela sua cadeia de ferramentas do DevOps; qualquer identificador que tornará mais fácil para você mapear a imagem para a construção real e o código-fonte contidos na imagem.

À medida que você se familiarizar com o Kubernetes, o [Helm](https://helm.sh/), o gerenciador de pacotes para Kubernetes, se tornará uma ferramenta útil para criar versão, montar e implementar seu aplicativo. [Essa cadeia de ferramentas do DevOps de amostra](https://github.com/open-toolchain/simple-helm-toolchain) é um bom ponto de início e é pré-configurada para entrega contínua para um cluster Kubernetes. À medida que seu projeto crescer em múltiplos microsserviços, o [gráfico de guarda-chuva do Helm](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies) fornecerá uma boa solução para editar seu aplicativo.

## Expandir o tutorial

Parabéns, seu aplicativo pode agora ser implementado de forma segura de desenvolvimento para produção. Abaixo estão sugestões adicionais para melhorar a entrega do aplicativo.

* Inclua [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) em seu pipeline para executar o controle de qualidade durante as implementações.
* Revise as contribuições de codificação do membro da equipe e as interações entre os desenvolvedores com o [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).
* Siga o tutorial [Planejar, criar e atualizar os ambientes de implementação](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments) para automatizar a implementação de seus ambientes.

## Informações relacionadas

* [Introdução ao {{site.data.keyword.iamshort}}](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [Melhores práticas para organizar recursos em um grupo de recursos](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [Analisar logs e monitorar o funcionamento com LogDNA e Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [Implementação contínua para o Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Cadeia de ferramentas do Hello Helm](https://github.com/open-toolchain/simple-helm-toolchain)
* [Desenvolver um aplicativo de microsserviços com Kubernetes e Helm](https://github.com/open-toolchain/microservices-helm-toolchain)
* [Conceder permissões a um usuário para visualizar logs no LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [Conceder permissões a um usuário para visualizar métricas no Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
