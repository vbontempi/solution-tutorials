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

# Planification, création et mise à jour d'environnements de déploiement 
{: #plan-create-update-deployments}

Les environnements de déploiement multiples sont courants lors de la création d'une solution. Ils reflètent le cycle de vie d'un projet, du développement à la production. Ce tutoriel présente des outils tels que l'interface CLI {{site.data.keyword.Bluemix_notm}} et [Terraform](https://www.terraform.io/) permettant d'automatiser la création et la maintenance de ces environnements de déploiement.
{:shortdesc}

Les développeurs n'aiment pas écrire deux fois la même chose. Le principe [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) en est un exemple. De même, ils n'aiment pas avoir à effectuer une multitude de clics dans une interface utilisateur pour configurer un environnement. En conséquence, les administrateurs système et les développeurs utilisent depuis longtemps des scripts shell pour automatiser des tâches répétitives, sujettes aux erreurs et sans intérêt. 

L'infrastructure en tant que service (IaaS), la plateforme en tant que service (PaaS), le conteneur en tant que service (CaaS), les fonctions en tant que service (FaaS) ont offert aux développeurs un niveau élevé d'abstraction et ont facilité l'acquisition de ressources telles que serveurs bare metal, bases de données gérées, machines virtuelles, clusters Kubernetes, etc. Mais une fois que vous avez mis ces ressources à disposition, vous devez les interconnecter, configurer l’accès des utilisateurs, mettre à jour la configuration régulièrement, etc. La possibilité d'automatiser toutes ces étapes et de reproduire l’installation et la configuration dans différents environnements est indispensable à l'heure actuelle. 

Dans un projet, il est assez courant que plusieurs environnements prennent en charge les différentes phases du cycle de développement, avec de légères différences entre les environnements tels que la capacité, la mise en réseau, les données d'identification et la prolixité du journal. Dans [cet autre tutoriel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications), nous avons présenté les bonnes pratiques pour organiser les utilisateurs, les équipes et les applications, ainsi qu'un exemple de scénario. L'exemple de scénario prend en compte trois environnements, *Développement*, *Test* et *Production*. Comment automatiser la création de ces environnements ? Quels outils pourraient être utilisés ? 

## Objectifs
{: #objectives}

* Définir un ensemble d'environnements à déployer 
* Ecrire des scripts à l'aide de la CLI d'{{site.data.keyword.Bluemix_notm}} et de [Terraform](https://www.terraform.io/) pour automatiser le déploiement de ces environnements 
* Déployer ces environnements dans votre compte 

## Services utilisés
{: #services}

Ce tutoriel utilise les produits suivants : 
* [Fournisseur d'{{site.data.keyword.Bluemix_notm}} pour Terraform](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Identity and Access Management](https://{DomainName}/iam/#/users)
* [Interface de ligne de commande {{site.data.keyword.Bluemix_notm}} - Interface CLI `ibmcloud`](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. Un ensemble de fichiers Terraform est créé pour décrire l'infrastructure cible en tant que code. 
1. Un opérateur utilise `terraform apply` pour mettre les environnements à disposition. 
1. Des scripts shell sont écrits pour compléter la configuration des environnements. 
1. L'opérateur exécute les scripts sur les environnements. 
1. Les environnements sont entièrement configurés, prêts à être utilisés. 

## Présentation des outils disponibles 
{: #tools}

Le premier outil permettant d’interagir avec {{site.data.keyword.Bluemix_notm}} et de créer des déploiements reproductibles est l’interface de ligne de commande [{{site.data.keyword.Bluemix_notm}} - CLI `ibmcloud`](/docs/cli?topic=cloud-cli-install-ibmcloud-cli). Avec `ibmcloud` et ses plugins, vous pouvez automatiser la création et la configuration de vos ressources cloud. A partir de la ligne de commande, vous pouvez mettre à disposition les {{site.data.keyword.virtualmachinesshort}}, les clusters Kubernetes, {{site.data.keyword.openwhisk_short}} et les applications et services Cloud Foundry. 

[Terraform](https://www.terraform.io/) de HashiCorp est un autre outil présenté dans [ce tutoriel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform). Selon HashiCorp, *Terraform vous permet de créer, de modifier et d’améliorer vos infrastructures de manière sûre et prévisible. Il s’agit d’un outil open source codifiant les API dans des fichiers de configuration déclaratifs pouvant être partagés par les membres de l’équipe, traités comme du code, modifiés, revus et corrigés.* Il s'agit d'une infrastructure en tant que code. Rédigez l'infrastructure que vous souhaitez obtenir et Terraform crée, met à jour et supprime les ressources du cloud selon vos indications. 

Pour prendre en charge une approche multicloud, Terraform travaille avec des fournisseurs. Un fournisseur est chargé de la compréhension des interactions des API et de l'exposition des ressources. {{site.data.keyword.Bluemix_notm}} dispose de [son fournisseur pour Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm), ce qui permet aux utilisateurs d’{{site.data.keyword.Bluemix_notm}} de gérer les ressources avec Terraform. Bien que Terraform soit classé comme infrastructure en tant que code, il ne se limite pas aux ressources d'infrastructure en tant que service (IaaS). Le fournisseur {{site.data.keyword.Bluemix_notm}} pour Terraform prend en charge les ressources IaaS (bare metal, machine virtuelle, services réseau, etc.), CaaS ({{site.data.keyword.containershort_notm}} et clusters Kubernetes), PaaS (Cloud Foundry et services) et FaaS ({{site.data.keyword.openwhisk_short}}).

## Ecriture de scripts pour automatiser le déploiement 
{: #scripts}

Lorsque vous commencez à décrire votre infrastructure sous forme de code, il est essentiel de traiter les fichiers que vous créez comme du code standard, afin de les stocker dans un système de gestion de contrôle de source. Avec le temps, ces pratiques présentent de nombreux avantages tels que l'utilisation du flux de travail de révision du contrôle de source pour valider les modifications avant de les appliquer, et l'ajout d'un pipeline d'intégration continue pour déployer automatiquement les modifications de l'infrastructure. 

[Ce référentiel Git](https://github.com/IBM-Cloud/multiple-environments-as-code) contient tous les fichiers de configuration nécessaires à la configuration des environnements définis précédemment. Vous pouvez cloner le référentiel pour suivre les sections suivantes détaillant le contenu des fichiers. 

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

Le référentiel est structuré comme suit : 

| Répertoire | Description |
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) | Accueil des fichiers Terraform |
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) | Fichiers Terraform permettant de mettre à disposition des ressources communes aux trois environnements |
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) | Fichiers Terraform spécifiques à un environnement donné |
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) | Fichiers Terraform permettant de configurer les règles utilisateur|

### Traitement de charges lourdes avec Terraform 

Les environnements de *Développement*, de *Test* et de *Production* sont similaires. 

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="Diagramme illustrant un environnement de déploiement" />
</p>

Ils partagent une organisation commune et des ressources spécifiques à l'environnement. Ils diffèrent par la capacité attribuée et les droits d'accès. Les fichiers Terraform sont structurés selon une configuration ***globale*** permettant de mettre à disposition l'organisation Cloud Foundry et une configuration ***spécifique à chaque environnement***, utilisant les espaces de travail Terraform, afin de mettre à disposition les ressources spécifiques à l'environnement : 

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### Configuration globale

Tous les environnements partagent une organisation Cloud Foundry commune et chaque environnement dispose de son propre espace. 

Dans le répertoire [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) se trouvent les scripts Terraform permettant de mettre cette organisation à disposition. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf) contient la définition de l'organisation :

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

Dans cette ressource, toutes les propriétés sont configurées via des variables. Dans les sections suivantes, vous apprenez à définir ces variables. 

Pour déployer pleinement les environnements, vous utilisez une combinaison de Terraform et de l'interface de ligne de commande (CLI) {{site.data.keyword.Bluemix_notm}}. Les scripts shell écrits via la CLI peuvent nécessiter de faire référence à cette organisation ou au compte par nom ou par ID. Le répertoire *global* inclut également [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf) qui génère un fichier contenant ces informations sous forme de clés/valeurs pouvant être réutilisées dans les scripts : 

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

### Environnements individuels 

Il existe différentes approches pour gérer plusieurs environnements avec Terraform. Vous pouvez dupliquer les fichiers Terraform dans des répertoires distincts, un répertoire par environnement. Avec les [modules Terraform](https://www.terraform.io/docs/modules/index.html), vous pouvez factoriser la configuration commune en tant que groupe et réutiliser des modules dans des environnements, ce qui réduit la duplication de code. Des répertoires séparés signifient que vous pouvez faire évoluer l'environnement de *développement* pour tester les modifications, puis les propager à d'autres environnements. Dans ce cas, il est courant que les *modules* Terraform se trouvent également dans leur propre référentiel de code source afin que vous puissiez référencer une version spécifique d'un module dans vos fichiers d'environnement. 

Etant donné que les environnements sont assez simples et similaires, vous allez utiliser un autre concept Terraform appelé [espaces de travail](https://www.terraform.io/docs/state/workspaces.html#best-practices). Les espaces de travail vous permettent d'utiliser les mêmes fichiers terraform (.tf) dans des environnements différents. Dans l'exemple, *Développement*, *Test* et *Production* sont des espaces de travail. Ils utilisent les mêmes définitions Terraform, mais avec des variables de configuration différentes (noms différents, capacités différentes). 

Chaque environnement nécessite : 
* un espace Cloud Foundry dédié 
* un groupe de ressources dédié 
* un cluster Kubernetes 
* une base de données 
* un stockage de fichiers 

L'espace Cloud Foundry est lié à l'organisation créée à l'étape précédente. Les fichiers d'environnement Terraform doivent faire référence à cette organisation. L'[état à distance de Terraform](https://www.terraform.io/docs/state/remote.html) trouve alors toute son utilité. Il permet de référencer un état Terraform existant en mode lecture seule. Il s'agit d'une construction très utile pour fractionner votre configuration Terraform en petits blocs, laissant la responsabilité de chaque bloc à différentes équipes. [backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf) contient la définition de l'état distant *global* utilisé pour rechercher l'organisation créée précédemment :

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

Une fois que vous pouvez faire référence à l'organisation, il est simple de créer un espace au sein de cette organisation. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf) contient la définition des ressources pour l'environnement.

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

Notez comment le nom de l'organisation est référencé à partir de l'état distant *global*. Les autres propriétés proviennent des variables de configuration. 

Vient ensuite le groupe de ressources. 

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

Le cluster Kubernetes est créé dans ce groupe de ressources. Le fournisseur {{site.data.keyword.Bluemix_notm}} dispose d'une ressource Terraform pour représenter un cluster : 

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

Là encore, la plupart des propriétés seront initialisées à partir de variables de configuration. Vous pouvez ajuster le centre de données, le nombre de noeuds worker, le type de noeuds worker. 

Des services compatibles IAM tels que {{site.data.keyword.cos_full_notm}} et {{site.data.keyword.cloudant_short_notm}} sont également créés en tant que ressources dans le groupe : 

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

Des liaisons Kubernetes (secrets) peuvent être ajoutées pour récupérer les données d'identification du service à partir de vos applications : 

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

## Déploiement de cet environnement dans votre compte 

### Installation de l'interface CLI {{site.data.keyword.Bluemix_notm}}

1. Suivez ces [instructions](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) pour installer la CLI 
1. Validez l'installation en exécutant : 
   ```sh
   ibmcloud
   ```
   {: codeblock}

### Installation de Terraform et du fournisseur {{site.data.keyword.Bluemix_notm}} pour Terraform

1. [Téléchargez et installez Terraform pour votre système.](https://www.terraform.io/intro/getting-started/install.html)
1. [Téléchargez le fichier binaire Terraform pour le fournisseur {{site.data.keyword.Bluemix_notm}}.](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)
   Pour configurer Terraform avec le fournisseur {{site.data.keyword.Bluemix_notm}}, cliquez sur ce [lien](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup)
   {:tip}
1. Créez un fichier `.terraformrc` dans votre répertoire personnel qui pointe vers le binaire Terraform. Dans l'exemple suivant, `/opt/provider/terraform-provider-ibm` constitue le chemin vers le répertoire.
   ```sh
   # ~/.terraformrc
   providers {
     ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### Obtention du code

Si vous ne l'avez pas encore fait, clonez le référentiel du tutoriel : 

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### Définition de la clé de l'API de la plateforme 

1. Si vous n'en avez pas déjà une, obtenez une [clé d'API de plateforme](https://{DomainName}/iam/#/apikeys) et enregistrez-la à des fins de référence ultérieure. 

   > Si, dans les étapes suivantes, vous envisagez de créer une organisation Cloud Foundry pour héberger les environnements de déploiement, assurez-vous que vous êtes le propriétaire du compte. 
1. Copiez [terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl) vers *terraform/credentials.tfvars* en exécutant la commande ci-dessous
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. Editez `terraform/credentials.tfvars` et définissez la valeur `ibmcloud_api_key` sur la clé d'API de plateforme que vous avez obtenue.

### Création ou réutilisation d'une organisation Cloud Foundry 

Vous pouvez choisir de créer une organisation ou de réutiliser (importer) une organisation existante. Pour créer l'organisation parent des trois environnements de déploiement, **vous devez être le propriétaire du compte**.

#### Création d'une organisation

1. Accédez au répertoire `terraform/global` 
1. Copiez [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) vers `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Editez `global.tfvars`
   1. Définissez **org_name** sur le nom de l'organisation à créer
   1. Définissez **org_managers** sur la liste des ID utilisateur auxquels que vous souhaitez attribuer le rôle *Responsable* dans l'organisation. L'utilisateur créant l'organisation est automatiquement responsable et ne doit pas être ajouté à la liste. 
   1. Définissez **org_users** sur une liste de tous les utilisateurs que vous souhaitez inviter dans l'organisation. Vous devez y ajouter les utilisateurs dont vous souhaitez configurer l'accès ultérieurement

   ```sh
   org_name = "a-new-organization"
   org_managers = [ "user1@domain.com", "another-user@anotherdomain.com" ]
   org_users = [ "user1@domain.com", "another-user@anotherdomain.com", "more-user@domain.com" ]
   ```
   {: codeblock}
1. Initialisez Terraform à partir du dossier `terraform/global`
   ```sh
   terraform init
   ```
   {: codeblock}
1. Observez le plan Terraform 
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. Appliquez les modifications
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Une fois le plan Terraform terminé, les éléments suivants sont créés : 
* une nouvelle organisation Cloud Foundry 
* un fichier `global.env` sous le répertoire `outputs` de votre réservation. Ce fichier contient des variables d'environnement que vous pouvez référencer dans d'autres scripts. 
* le fichier `terraform.tfstate`

> Ce tutoriel utilise le fournisseur de back-end `local` pour l’état Terraform. Pratique pour découvrir Terraform ou travailler seul sur un projet, mais lorsque vous travaillez en équipe ou sur une infrastructure plus grande, Terraform prend également en charge la sauvegarde de l'état sur un site distant. Etant donné que l'état de Terraform est essentiel pour les opérations Terraform, il est recommandé d'utiliser un stockage distant, résilient et hautement disponible pour l'état de Terraform. Reportez-vous à [Terraform Backend Types](https://www.terraform.io/docs/backends/types/index.html) pour consulter une liste d'options disponibles. Certains serveurs de back-end prennent même en charge la gestion des versions et le verrouillage des états Terraform. 

#### Réutilisation d'une organisation que vous gérez 

Si vous n'êtes pas le propriétaire du compte mais que vous gérez une organisation dans le compte, vous pouvez également importer une organisation existante dans Terraform. 

1. Extrayez l'ID global unique de l'organisation 
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. Accédez au répertoire `terraform/global`
1. Copiez [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) vers `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Initialisez Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. Après l’initialisation de Terraform, importez l’organisation dans l’état Terraform. 
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. Ajustez la variable `global.tfvars` pour qu'elle corresponde au nom et à la structure de l'organisation existante 
1. Appliquez les modifications
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### Création d'un espace, d'un cluster et de services pour chaque environnement 

Cette section porte sur l'environnement de `développement`. Les étapes sont identiques pour les autres environnements, seules les valeurs que vous choisissez pour les variables sont différentes. 

1. Accédez au dossier `terraform/per-environment` pour la réservation 
1. Copiez le modèle de fichier `tfvars`. Il y en a un par environnement : 
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. Editez `development.tfvars`
   1. Définissez **environment_name** sur le nom de l'espace Cloud Foundry que vous souhaitez créer
   1. Définissez **space_developers** sur la liste des développeurs pour cet espace. **Veillez à ajouter votre nom à la liste afin que Terraform puisse fournir des services en votre nom.**
   1. Définissez **cluster_datacenter** sur l'emplacement où vous souhaitez créer le cluster. Recherchez les emplacements disponibles à l'aide de la commande :
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. Définissez les VLAN privé (**cluster_private_vlan_id**) et public (**cluster_public_vlan_id**) du cluster. Recherchez les VLAN disponibles pour l'emplacement à l'aide de la commande :
      ```sh
      ibmcloud cs vlans <location>
      ```
      {: codeblock}
   1. Définissez **cluster_machine_type**. Recherchez les types de machine disponibles et les caractéristiques pour l'emplacement à l'aide de la commande :
      ```sh
      ibmcloud cs machine-types <location>
      ```
      {: codeblock}

   1. Définissez **resource_quota**. Recherchez les définitions de quota de ressources disponibles à l'aide de la commande :
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. Initialisez Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. Créez un espace de travail Terraform pour l'environnement de *développement* 
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   Pour basculer ultérieurement entre les environnements, utilisez
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. Observez le plan Terraform 
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Il devrait indiquer :
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. Appliquez les modifications
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Une fois le plan Terraform terminé, les éléments suivants sont créés : 
* un groupe de ressources
* un espace Cloud Foundry 
* un cluster Kubernetes avec un pool de noeuds worker et une zone qui lui est attachée 
* une base de données 
* un secret Kubernetes avec les données d'identification de la base de données 
* un stockage 
* un secret Kubernetes avec les données d'identification du stockage 
* une instance de journalisation (LogDNA) 
* une instance de surveillance (Sysdig) 
* un fichier `development.env` sous le répertoire `outputs` de votre réservation. Ce fichier contient des variables d'environnement que vous pouvez référencer dans d'autres scripts. 
* le fichier `terraform.tfstate` spécifique à l’environnement sous `terraform.tfstate.d/development`.

Vous pouvez répéter ces étapes pour les environnements de `test` et de `production`.

### Attribution de règles utilisateur 

Dans les étapes précédentes, les rôles dans l'organisation Cloud Foundry et les espaces ont été configurés avec le fournisseur Terraform. Pour les règles utilisateur sur d'autres ressources telles que les clusters Kubernetes, vous utilisez le dossier [roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) dans le référentiel cloné. 

Pour l'environnement de *développement* tel que défini dans ce [tutoriel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications), les règles à définir sont les suivantes :

|           | Règles d'accès IAM |
| --------- | ----------- |
| Développeur | <ul><li>Groupe de ressources : *Afficheur*</li><li>Rôles d'accès à la plateforme dans le groupe de ressources : *Afficheur*</li><li>Rôle de service de journalisation & surveillance : *Auteur*</li></ul> |
| Testeur    | <ul><li>Aucune configuration nécessaire. Le testeur accède à l'application déployée, pas aux environnements de développement </li></ul> |
| Opérateur  | <ul><li>Groupe de ressources : *Afficheur*</li><li>Rôles d'accès à la plateforme dans le groupe de ressources : *Opérateur*, *Afficheur*</li><li>Rôle de service de journalisation & surveillance : *Auteur*</li></ul> |
| Utilisateur fonctionnel du pipeline | <ul><li>Groupe de ressources : *Afficheur*</li><li>Rôles d'accès à la plateforme dans le groupe de ressources : *Editeur*, *Afficheur*</li></ul> |

Etant donné qu'une équipe peut être composée de plusieurs développeurs, testeurs, vous pouvez utiliser le [concept de groupe d'accès](https://{DomainName}/docs/iam?topic=iam-groups#groups) pour simplifier la configuration des règles utilisateur. Des groupes d'accès peuvent être créés par le propriétaire du compte afin qu'un même accès puisse être attribué à toutes les entités du groupe avec une seule règle. 

Pour le rôle *Développeur* dans l'environnement de *développement*, cela se traduit par : 

   ```sh
resource "ibm_iam_access_group" "developer_role" {
  name        = "${var.access_group_name_developer_role}"
  description = "${var.access_group_description}"
}
resource "ibm_iam_access_group_policy" "resourcepolicy_developer" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Viewer"]

  resources = [{
    resource_type = "resource-group"
    resource      = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_platform_accesspolicy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles        = ["Viewer"]

  resources = [{
    resource_group_id = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_logging_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "logdna"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.logdna_instance_id}"
  }]
}
resource "ibm_iam_access_group_policy" "developer_monitoring_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "sysdig-monitor"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.sysdig_instance_id}"
  }]
}

   ```

Le fichier [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) de la réservation contient des exemples de ces ressources pour les rôles *Développeur*, *Opérateur*, *Testeur* et *Utilisateur fonctionnel* définis. Pour configurer les règles définies dans une section précédente pour les utilisateurs dotés des rôles*Développeur, Opérateur, Testeur et Utilisateur fonctionnel* dans l'environnement de *développement*, 

1. Accédez au répertoire `terraform/roles/development`
2. Copiez le modèle de fichier `tfvars`. Il y en a un par environnement (vous pouvez trouver les modèles de `production` et de `test` dans leurs dossiers respectifs dans le répertoire `roles`) 

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. Editez `development.tfvars`

   - Définissez **iam_access_members_developers** sur la liste des développeurs auxquels vous souhaitez accorder l'accès.
   - Définissez **iam_access_members_operators** sur la liste des opérateurs, etc.
4. Initialisez Terraform
   ```sh
   terraform init
   ```
   {: codeblock}

5. Observez le plan Terraform 
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Il devrait indiquer :
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. Appliquez les modifications
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
Vous pouvez répéter ces étapes pour les environnements de `test` et de `production`.

## Suppression de ressources 

1. Accédez au dossier `development` sous `roles`
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. Détruisez les groupes d'accès et les règles d'accès 
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Activez l'espace de travail de `développement`
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. Détruisez le groupe de ressources, les espaces, les services, les clusters 
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Répétez ces étapes pour les espaces de travail de `test` et de `production`
1. Si vous l'avez créée, détruisez l'organisation 
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## Contenu associé

* [Tutoriel Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Fournisseur Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
* [Exemples d'utilisation du fournisseur {{site.data.keyword.Bluemix_notm}} pour Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
