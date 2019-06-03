---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# Planejar, criar e atualizar ambientes de implementação
{: #plan-create-update-deployments}

Múltiplos ambientes de implementação são comuns ao construir uma solução. Eles refletem o ciclo de vida de um projeto do desenvolvimento para a produção. Este tutorial apresenta ferramentas como a CLI do {{site.data.keyword.Bluemix_notm}} e o [Terraform](https://www.terraform.io/) para automatizar a criação e a manutenção desses ambientes de implementação.
{:shortdesc}

Os desenvolvedores não gostam de escrever a mesma coisa duas vezes. O princípio [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) é um exemplo disso. Da mesma forma, eles não gostam de ter que passar por um monte de cliques em uma interface com o usuário para configurar um ambiente. Consequentemente, os shell scripts têm sido usados há muito tempo pelos administradores e desenvolvedores de sistema para automatizar tarefas repetitivas, propensas a erros e desinteressantes.

A Infraestrutura como Serviço (IaaS), Plataforma como Serviço (PaaS), Container as a Service (CaaS), Functions as a Service (FaaS) têm fornecido aos desenvolvedores um alto nível de abstração e tornou mais fácil adquirir recursos como servidores bare metal, bancos de dados gerenciados, máquinas virtuais, clusters Kubernetes, etc. Mas uma vez provisionados esses recursos, é necessário conectá-los juntos, para configurar o acesso de usuário, para atualizar a configuração ao longo do tempo, etc. Estando apto a automatizar todas essas etapas e repetir a instalação, a configuração em diferentes ambientes é essencial nos dias de hoje.

Múltiplos ambientes são bastante comuns em um projeto para suportar as diferentes fases do ciclo de desenvolvimento com pequenas diferenças entre os ambientes, como capacidade, rede, credenciais, detalhamento do log. [Neste outro tutorial](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications), introduzimos as melhores práticas para organizar usuários, equipes e aplicativos e um cenário de amostra. O cenário de amostra considera três ambientes, *Desenvolvimento*, *Teste* e *Produção*. Como automatizar a criação desses ambientes? Quais ferramentas podem ser usadas?

## Objetivos
{: #objectives}

* Definir um conjunto de ambientes para implementar
* Gravar scripts usando a CLI do {{site.data.keyword.Bluemix_notm}} e o [Terraform](https://www.terraform.io/) para automatizar a implementação desses ambientes
* Implementar esses ambientes em sua conta

## Serviços usados
{: #services}

Este tutorial usa os produtos a seguir:
* [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Identidade e gerenciamento de acesso](https://{DomainName}/iam/#/users)
* [Interface da linha de comandos do {{site.data.keyword.Bluemix_notm}} - a CLI `ibmcloud`](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

Este tutorial pode incorrer em custos. Use o [Calculador de precificação](https://{DomainName}/pricing/) para gerar uma estimativa de custo com base no uso projetado.

## Arquitetura
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. Um conjunto de arquivos do Terraform é criado para descrever a infraestrutura de destino como código.
1. Um operador usa `terraform apply` para provisionar os ambientes.
1. Os shell scripts são gravados para concluir a configuração dos ambientes.
1. O operador executa os scripts com relação aos ambientes
1. Os ambientes estão totalmente configurados, prontos para serem usados.

## Visão geral das ferramentas disponíveis
{: #tools}

A primeira ferramenta para interagir com o {{site.data.keyword.Bluemix_notm}} e para criar implementações repetidas é a [interface da linha de comandos do {{site.data.keyword.Bluemix_notm}}, a CLI `ibmcloud`](/docs/cli?topic=cloud-cli-install-ibmcloud-cli). Com `ibmcloud` e seus plug-ins, é possível automatizar a criação e a configuração de seus recursos em nuvem. {{site.data.keyword.virtualmachinesshort}}, clusters Kubernetes, {{site.data.keyword.openwhisk_short}}, apps e serviços do Cloud Foundry, é possível provisionar todos eles por meio da linha de comandos.

Outra ferramenta introduzida [neste tutorial](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform) é [Terraform](https://www.terraform.io/) da HashiCorp. Citando a HashiCorp, o *Terraform permite criar, mudar e melhorar a infraestrutura de forma segura e previsível. Ele é uma ferramenta de software livre que codifica as APIs em arquivos de configuração declarativos que podem ser compartilhados entre membros da equipe, tratados como código, editados, revisados e com versão.* É uma infraestrutura como código. Você anota como sua infraestrutura deve se parecer e o Terraform criará, atualizará, removerá os recursos em nuvem conforme necessário.

Para suportar uma abordagem multinuvem, o Terraform funciona com os provedores. Um provedor é responsável por entender as interações da API e expor os recursos. O {{site.data.keyword.Bluemix_notm}} tem [seu provedor para o Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm) permitindo que os usuários do {{site.data.keyword.Bluemix_notm}} gerenciem recursos com o Terraform. Embora o Terraform seja categorizado como infraestrutura como código, ele não é limitado a recursos de Infraestrutura como Serviço. O {{site.data.keyword.Bluemix_notm}} Provider for Terraform suporta os recursos IaaS (bare metal, máquina virtual, serviços de rede, etc.), CaaS ({{site.data.keyword.containershort_notm}} e cluster Kubernetes), PaaS (Cloud Foundry e serviços) e FaaS ({{site.data.keyword.openwhisk_short}}).

## Gravar scripts para automatizar a implementação
{: #scripts}

Conforme você inicia a descrição de sua infraestrutura como código, é crítico tratar os arquivos criados como código regular, armazenando-os em um sistema de gerenciamento de controle de origem. Ao longo do tempo, isso trará boas propriedades, como usar o fluxo de trabalho de revisão de controle de fonte para validar as mudanças antes de aplicá-las, incluir um pipeline de integração contínua para implementar automaticamente as mudanças de infraestrutura.

[Este repositório Git](https://github.com/IBM-Cloud/multiple-environments-as-code) tem todos os arquivos de configuração necessários para configurar os ambientes definidos anteriormente. É possível clonar o repositório para seguir as próximas seções que detalham o conteúdo dos arquivos.

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

O repositório é estruturado conforme a seguir:

| Diretório | Descrição |
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) | Início para os arquivos do Terraform |
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) | Arquivos do Terraform para provisionar recursos comuns para os três ambientes |
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) | Arquivos do Terraform específicos para um determinado ambiente |
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) | Arquivos do Terraform para configurar políticas do usuário |

### Trabalho pesado com o Terraform

Os ambientes de *Desenvolvimento*, *Teste* e *Produção* praticamente são iguais.

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="Diagrama mostrando um ambiente de implementação" />
</p>

Eles compartilham uma organização comum e recursos específicos do ambiente. Eles serão diferentes pela capacidade alocada e pelos direitos de acesso. Os arquivos do terraform refletem isso com uma configuração ***global*** para provisionar a organização do Cloud Foundry e uma configuração ***por ambiente***, usando áreas de trabalho do Terraform, para provisionar recursos específicos do ambiente:

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### Configuração global

Todos os ambientes compartilham uma organização comum do Cloud Foundry e cada ambiente tem seu próprio espaço.

Sob o diretório [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global), você localiza os scripts do Terraform para provisionar essa organização. O [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf) contém a definição para a organização:

   ```sh
   # create a new organization for the project
   resource "ibm_org" "organization" {
     name             = "${var.org_name}"
     managers         = "${var.org_managers}"
     users            = "${var.org_users}"
     auditors         = "${var.org_auditors}"
     billing_managers = "${var.org_billing_managers}"
   }
   ```

Nesse recurso, todas as propriedades são configuradas por meio de variáveis. Nas próximas seções, você aprenderá como configurar essas variáveis.

Para implementar totalmente os ambientes, você usará uma combinação do Terraform e da CLI do {{site.data.keyword.Bluemix_notm}}. Os shell scripts gravados com a CLI podem precisar referenciar essa organização ou a conta por nome ou ID. O diretório *global* também inclui o [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf), que produzirá um arquivo contendo essas informações como chaves/valores adequados para serem reutilizados no script:

   ```sh
   # generate a property file suitable for shell scripts with useful variables relating to the environment
   resource "local_file" "output" {
     content = <<EOF
ACCOUNT_GUID=${data.ibm_account.account.id}
ORG_GUID=${ibm_org.organization.id}
ORG_NAME=${var.org_name}
EOF

     filename = "../outputs/global.env"
   }
   ```

### Ambientes individuais

Há diferentes abordagens para gerenciar múltiplos ambientes com o Terraform. É possível duplicar os arquivos do Terraform em diretórios separados, um diretório por ambiente. Com os [Módulos do Terraform](https://www.terraform.io/docs/modules/index.html), é possível fatorar a configuração comum como um grupo e reutilizar os módulos entre os ambientes, reduzindo a duplicação de código. Diretórios separados significam que é possível desenvolver o ambiente de *desenvolvimento* para testar as mudanças e, em seguida, propagar as mudanças para outros ambientes. É comum, neste caso, também ter os *módulos* do Terraform em seu próprio repositório de código-fonte para que seja possível referenciar uma versão específica de um módulo em seus arquivos de ambiente.

Dado que os ambientes são bastante simples e semelhantes, você usará outro conceito do Terraform chamado [áreas de trabalho](https://www.terraform.io/docs/state/workspaces.html#best-practices). As áreas de trabalho permitem que você use os mesmos arquivos do terraform (.tf) com ambientes diferentes. No exemplo, *desenvolvimento*, *teste* e *produção* são áreas de trabalho. Eles usarão as mesmas definições do Terraform, mas com variáveis de configuração diferentes (nomes diferentes, capacidades diferentes).

Cada ambiente requer:
* um espaço dedicado do Cloud Foundry
* um grupo de recursos dedicado
* um cluster Kubernetes
* um banco de dados
* um armazenamento de arquivo

O espaço do Cloud Foundry está vinculado à organização criada na etapa anterior. Os arquivos do Terraform do ambiente precisam referenciar essa organização. Esse é o local onde o [estado remoto do Terraform](https://www.terraform.io/docs/state/remote.html) ajudará. Ele permite a referência de um estado existente do Terraform no modo somente leitura. Essa é uma construção muito útil para dividir sua configuração do Terraform em partes menores deixando a responsabilidade de partes individuais para equipes diferentes. O [backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf) contém a definição do estado remoto *global* usado para localizar a organização criada anteriormente:

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

Uma vez que seja possível referenciar a organização, é fácil criar um espaço dentro dessa organização. O [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf) contém a definição dos recursos para o ambiente.

   ```sh
   # a Cloud Foundry space per environment
   resource "ibm_space" "space" {
     name       = "${var.environment_name}"
     org        = "${data.terraform_remote_state.global.org_name}"
     managers   = "${var.space_managers}"
     auditors   = "${var.space_auditors}"
     developers = "${var.space_developers}"
   }
   ```

Observe como o nome da organização é referenciado no estado remoto *global*. As outras propriedades são tomadas das variáveis de configuração.

Em seguida, vem o grupo de recursos.

   ```sh
   # a resource group
   resource "ibm_resource_group" "group" {
    name     = "${var.environment_name}"
    quota_id = "${data.ibm_resource_quota.quota.id}"
}

   data "ibm_resource_quota" "quota" {
	name = "${var.resource_quota}"
}
   ```

O cluster Kubernetes é criado nesse grupo de recursos. O provedor {{site.data.keyword.Bluemix_notm}} tem um recurso do Terraform para representar um cluster:

   ```sh
  # a cluster
  resource "ibm_container_cluster" "cluster" {
  	name              = "${var.environment_name}-cluster"
  	datacenter        = "${var.cluster_datacenter}"
  	org_guid          = "${data.terraform_remote_state.global.org_guid}"
  	space_guid        = "${ibm_space.space.id}"
  	account_guid      = "${data.terraform_remote_state.global.account_guid}"
  	hardware          = "${var.cluster_hardware}"
  	machine_type      = "${var.cluster_machine_type}"
  	public_vlan_id    = "${var.cluster_public_vlan_id}"
  	private_vlan_id   = "${var.cluster_private_vlan_id}"
  	resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool" "cluster_workerpool" {
  worker_pool_name  = "${var.environment_name}-pool"
  machine_type      = "${var.cluster_machine_type}"
  cluster           = "${ibm_container_cluster.cluster.id}"
  size_per_zone     = "${var.worker_num}"
  hardware          = "${var.cluster_hardware}"
  resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool_zone_attachment" "cluster_zone" {
  cluster           = "${ibm_container_cluster.cluster.id}"
  worker_pool       =  "${element(split("/",ibm_container_worker_pool.cluster_workerpool.id),1)}"
  zone              = "${var.cluster_datacenter}"
  public_vlan_id    = "${var.cluster_public_vlan_id}"
  private_vlan_id   = "${var.cluster_private_vlan_id}"
  resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

Mais uma vez, a maioria das propriedades será inicializada por meio de variáveis de configuração. É possível ajustar o data center, o número de trabalhadores, o tipo de trabalhadores.

Os serviços ativados pelo IAM, como {{site.data.keyword.cos_full_notm}} e {{site.data.keyword.cloudant_short_notm}}, são criados como recursos dentro do grupo também:

   ```sh
# a database
resource "ibm_resource_instance" "database" {
    name              = "database"
    service           = "cloudantnosqldb"
    plan              = "${var.cloudantnosqldb_plan}"
    location          = "${var.cloudantnosqldb_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
# a cloud object storage
resource "ibm_resource_instance" "objectstorage" {
    name              = "objectstorage"
    service           = "cloud-object-storage"
    plan              = "${var.cloudobjectstorage_plan}"
    location          = "${var.cloudobjectstorage_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

As ligações do Kubernetes (segredos) podem ser incluídas para recuperar as credenciais de serviço de seus aplicativos:

   ```sh
   # bind the cloudant service to the cluster
   resource "ibm_container_bind_service" "bind_database" {
      cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	  service_instance_name       = "${ibm_resource_instance.database.name}"
      namespace_id                = "default"
      account_guid                = "${data.terraform_remote_state.global.account_guid}"
      org_guid                    = "${data.terraform_remote_state.global.org_guid}"
      space_guid                  = "${ibm_space.space.id}"
      resource_group_id           = "${ibm_resource_group.group.id}"
}

   # bind the cloud object storage service to the cluster
   resource "ibm_container_bind_service" "bind_objectstorage" {
  	cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	space_guid                  = "${ibm_space.space.id}"
  	service_instance_id         = "${ibm_resource_instance.objectstorage.name}"
  	namespace_id                = "default"
  	account_guid                = "${data.terraform_remote_state.global.account_guid}"
  	org_guid                    = "${data.terraform_remote_state.global.org_guid}"
  	space_guid                  = "${ibm_space.space.id}"
  	resource_group_id           = "${ibm_resource_group.group.id}"
}
   ```

## Implementar este ambiente em sua conta

### Instalar a CLI do {{site.data.keyword.Bluemix_notm}}

1. Siga [estas instruções](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) para instalar a CLI
1. Valide a instalação executando:
   ```sh
   ibmcloud
   ```
   {: codeblock}

### Instalar o Terraform e o {{site.data.keyword.Bluemix_notm}} Provider for Terraform

1. [Faça download e instale o Terraform para seu sistema.](https://www.terraform.io/intro/getting-started/install.html)
1. [Faça download do binário do Terraform para o provedor do {{site.data.keyword.Bluemix_notm}}.](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)
   Para configurar o Terraform com o provedor do {{site.data.keyword.Bluemix_notm}}, consulte este [link](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup)
   {:tip}
1. Crie um arquivo `.terraformrc` em seu diretório inicial que aponte para o binário do Terraform. No exemplo a seguir, `/opt/provider/terraform-provider-ibm` é a rota para o diretório.
   ```sh
   # ~/.terraformrc providers {
     ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### Obtenha o código

Se você ainda não tiver feito isso, clone o repositório do tutorial:

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### Configurar a chave de API da plataforma

1. Se você ainda não tiver um, obtenha uma [chave de API da plataforma](https://{DomainName}/iam/#/apikeys) e salve a chave de API para referência futura.

   > Em etapas posteriores, caso planeje criar uma nova organização do Cloud Foundry para hospedar os ambientes de implementação, certifique-se de que você seja o proprietário da conta.
1. Copie [terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl) para *terraform/credentials.tfvars* executando o comando abaixo
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. Edite `terraform/credentials.tfvars` e configure o valor para `ibmcloud_api_key` para a chave de API da Plataforma que você obteve.

### Criar ou reutilizar uma organização do Cloud Foundry

É possível escolher criar uma nova organização ou reutilizar (importar) uma existente. Para criar a organização mãe dos três ambientes de implementação, **é necessário ser o proprietário da conta**.

#### Para criar uma nova organização

1. Mude para o diretório `terraform/global`
1. Copie [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) para `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Edite `global.tfvars`
   1. Configure **org_name** para o nome da organização a ser criado
   1. Configure **org_managers** para uma lista de IDs do usuário aos quais você deseja conceder a função de *Gerenciador* na organização, o usuário que está criando a organização é automaticamente um gerenciador e não deve ser incluído na lista
   1. Configure **org_users** para uma lista de todos os usuários que você deseja convidar para a organização, os usuários precisarão ser incluídos lá se você desejar configurar o acesso deles em etapas adicionais

   ```sh
   org_name = "a-new-organization"
   org_managers = [ "user1@domain.com", "another-user@anotherdomain.com" ]
   org_users = [ "user1@domain.com", "another-user@anotherdomain.com", "more-user@domain.com" ]
   ```
   {: codeblock}
1. Inicialize o Terraform por meio da pasta `terraform/global`
   ```sh
   terraform init
   ```
   {: codeblock}
1. Veja o plano Terraform.
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. Aplicar as mudanças
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Depois que o Terraform for concluído, ele será criado:
* uma nova organização do Cloud Foundry
* um arquivo `global.env` sob o diretório `outputs` em seu check-out. Este arquivo possui variáveis de ambiente que você pode referenciar em outros scripts
* o arquivo `terraform.tfstate`

> Este tutorial usa o provedor de back-end `local` para o estado do Terraform. Útil ao descobrir o Terraform ou trabalhar sozinho em um projeto, mas ao trabalhar em uma equipe ou em uma infraestrutura maior, o Terraform também suporta salvar o estado em um local remoto. Dado que o estado do Terraform é crítico para as operações do Terraform, é recomendável usar um armazenamento remoto, altamente disponível e resiliente para o estado do Terraform. Consulte [Tipos de back-end do Terraform](https://www.terraform.io/docs/backends/types/index.html) para obter uma lista de opções disponíveis. Alguns back-ends até suportam a versão e o bloqueio de estados do Terraform.

#### Para reutilizar uma organização que você está gerenciando

Se você não é o proprietário da conta, mas gerencia uma organização na conta, também é possível importar uma organização existente para o Terraform

1. Recupere o GUID da organização
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. Mude para o diretório `terraform/global`
1. Copie [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) para `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Inicializar o Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. Depois de inicializar o Terraform, importe a organização para o estado do Terraform
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. Ajuste o `global.tfvars` para corresponder ao nome da organização e estrutura existentes
1. Aplicar as mudanças
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### Criar espaço por ambiente, cluster e serviços

Esta seção se concentrará no ambiente `development`. As etapas serão as mesmas para os outros ambientes, somente os valores que você escolher para as variáveis serão diferentes.

1. Mude para a pasta `terraform/per-environment` do check-out
1. Copie o arquivo de modelo `tfvars`. Há um por ambiente:
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. Editar `development.tfvars`
   1. Configure **environment_name** para o nome do espaço do Cloud Foundry que você deseja criar
   1. Configure **space_developers** para a lista de desenvolvedores para esse espaço. **Certifique-se de incluir seu nome na lista para que o Terraform possa provisionar serviços em seu nome.**
   1. Configure **cluster_datacenter** para o local no qual você deseja criar o cluster. Encontre os locais disponíveis com:
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. Configure as VLANs privadas (**cluster_private_vlan_id**) e públicas (**cluster_public_vlan_id**) para o cluster. Localize as VLANs disponíveis para o local com:
      ```sh
      ibmcloud cs vlans <location>
      ```
      {: codeblock}
   1. Configure o **cluster_machine_type**. Localize os tipos de máquina disponíveis e as características para o local com:
      ```sh
      ibmcloud cs machine-types <location>
      ```
      {: codeblock}

   1. Configure o **resource_quota**. Localize as definições de cota de recurso disponíveis com:
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. Inicializar o Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. Crie uma nova área de trabalho do Terraform para o ambiente de *desenvolvimento*
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   Mais tarde, para alternar entre os ambientes, use
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. Veja o plano Terraform.
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Ele deve relatar:
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. Aplicar as mudanças
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Depois que o Terraform for concluído, ele será criado:
* um grupo de recursos
* um espaço do Cloud Foundry
* um cluster Kubernetes com um conjunto de trabalhadores e uma zona conectada a ele
* um banco de dados
* um segredo do Kubernetes com as credenciais do banco de dados
* um armazenamento
* um segredo do Kubernetes com as credenciais de armazenamento
* uma instância de criação de log (LogDNA)
* uma instância de monitoramento (Sysdig)
* um arquivo `development.env` sob o diretório `outputs` em seu check-out. Este arquivo possui variáveis de ambiente que você pode referenciar em outros scripts
* o `terraform.tfstate` específico do ambiente em `terraform.tfstate.d/development`.

É possível repetir as etapas para `teste` e `produção`.

### Designar políticas do usuário

Nas etapas anteriores, as funções na organização e nos espaços do Cloud Foundry podiam ser configuradas com o provedor do Terraform. Para políticas de usuário em outros recursos como os clusters Kubernetes, você usará a pasta [funções](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) no repositório clonado.

Para o ambiente de *Desenvolvimento* conforme definido [neste tutorial](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications), as políticas a serem definidas são:

|           | Políticas de acesso do IAM |
| --------- | ----------- |
| Desenvolvedor | <ul><li>Grupo de recursos: *Visualizador*</li><li>Funções de acesso da plataforma no grupo de recursos: *Visualizador*</li><li>Criação de log e Função de serviço de monitoramento: *Escritor*</li></ul> |
| Testador    | <ul><li>Nenhuma configuração necessária. O testador acessa o aplicativo implementado, não os ambientes de desenvolvimento</li></ul> |
| Operador  | <ul><li>Grupo de recursos: *Visualizador*</li><li>Funções de acesso da plataforma no grupo de recursos: *Operador*, *Visualizador*</li><li>Criação de log e Função de serviço de monitoramento: *Escritor*</li></ul> |
| Usuário funcional de pipeline | <ul><li>Grupo de recursos: *Visualizador*</li><li>Funções de acesso da plataforma no grupo de recursos: *Editor*, *Visualizador*</li></ul> |

Considerando que uma equipe pode ser composta de vários desenvolvedores, testadores, é possível alavancar o [conceito de grupo de acesso](https://{DomainName}/docs/iam?topic=iam-groups#groups) para simplificar a configuração de políticas do usuário. Os grupos de acesso podem ser criados pelo proprietário da conta para que o mesmo acesso possa ser designado a todas as entidades dentro do grupo com uma única política.

Para a função de *Desenvolvedor* no ambiente de *Desenvolvimento*, isso é convertido em:

   ```sh
resource "ibm_iam_access_group" "developer_role" {
  name        = "${var.access_group_name_developer_role}"
  description = "${var.access_group_description}"
}
resource "ibm_iam_access_group_policy" "resourcepolicy_developer" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}" roles = ["Visualizador"]

  resources = [{
    resource_type = "resource-group"
    resource      = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_platform_accesspolicy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}" roles = ["Visualizador"]

  resources = [{
    resource_group_id = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_logging_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}" roles = ["Escritor"]

  resources = [{
    service           = "logdna"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.logdna_instance_id}"
  }]
}
resource "ibm_iam_access_group_policy" "developer_monitoring_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}" roles = ["Escritor"]

  resources = [{
    service           = "sysdig-monitor"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.sysdig_instance_id}"
  }]
}

   ```

O arquivo [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) do check-out tem exemplos desses recursos para as funções definidas de *Desenvolvedor*, *Operador*, *testador* e *Usuário funcional*. Para configurar as políticas conforme definido em uma seção anterior para os usuários com as funções de *Desenvolvedor, Operador, Testador e Usuário funcional* no ambiente de *desenvolvimento*,

1. Mude para o diretório `terraform/roles/development`
2. Copie o arquivo de modelo `tfvars`. Há um por ambiente (é possível localizar os modelos `production` e `testing` sob suas respectivas pastas no diretório `roles`)

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. Editar `development.tfvars`

   - Configure **iam_access_members_developers** para a lista de desenvolvedores para os quais você gostaria de conceder o acesso.
   - Configure **iam_access_members_operators** para a lista de operadores e assim por diante.
4. Inicializar o Terraform
   ```sh
   terraform init
   ```
   {: codeblock}

5. Veja o plano Terraform.
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Ele deve relatar:
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. Aplicar as mudanças
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
É possível repetir as etapas para `teste` e `produção`.

## Remover recursos

1. Navegue para a pasta `development` em `roles`
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. Destrua os grupos de acesso e as políticas de acesso
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Ative a área de trabalho `development`
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. Destrua o grupo de recursos, espaços, serviços e clusters
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Repita as etapas para as áreas de trabalho `testing` e `production`
1. Se você a criou, destrua a organização
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## Conteúdo relacionado

* [Tutorial do Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Provedor do Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
* [Exemplos usando o {{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
