---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-23"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Déploiement d'une pile LAMP à l'aide de Terraform
{: #infrastructure-as-code-terraform}

[Terraform](https://www.terraform.io/) vous permet de créer, de modifier et d’améliorer vos infrastructures de manière sûre et prévisible. Il s’agit d’un outil open source qui codifie les API dans des fichiers de configuration déclaratifs pouvant être partagés par les membres de l’équipe, traités comme du code, modifiés, revus et corrigés. 

Dans ce tutoriel, vous allez utiliser un exemple de configuration pour mettre à disposition la combinaison d'un serveur virtuel **L**inux, d'un serveur Web **A**pache, de **M**ySQL et d'un serveur **P**HP, appelée pile **LAMP**. Vous allez ensuite mettre à jour la configuration pour ajouter le service {{site.data.keyword.cos_full_notm}} et mettre à l'échelle les ressources pour ajuster l'environnement (mémoire, unité centrale et taille du disque). Enfin, vous allez supprimer toutes les ressources créées par la configuration. 

## Objectifs
{: #objectives}

* Configurer Terraform et le fournisseur {{site.data.keyword.Bluemix_notm}} pour Terraform.
* Utiliser Terraform pour créer, mettre à jour, mettre à l'échelle et enfin détruire la configuration de pile LAMP. 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

<p style="text-align: center;">

  ![Diagramme d'architecture](images/solution10/architecture-2.png)
</p>

1. Un ensemble de fichiers Terraform est créé pour décrire la configuration de la pile LAMP. 
1. `terraform` est appelé à partir du répertoire de configuration.
1. `terraform` appelle l'API {{site.data.keyword.cloud_notm}} pour mettre à disposition les ressources.

## Avant de commencer
{: #prereqs}

Contactez votre utilisateur principal d'infrastructure pour obtenir les droits suivants :
- Réseau (pour ajouter une **Liaison montante sur réseau public et privé**)
- Clé d'API 

## Conditions préalables

{: #prereq}

Installez **Terraform** via [installer](https://www.terraform.io/intro/getting-started/install.html) ou utilisez [Homebrew](https://brew.sh/) sur macOS en exécutant la commande suivante : `brew install terraform`

Sous **Windows**, procédez comme suit pour effectuer la configuration de terraform :
   1. Copiez les fichiers du zip téléchargé dans `C:\terraform` (créez un dossier `terraform`).
   2. Ouvrez l'invite de commande en tant qu'administrateur et configurez PATH pour que cette variable utilise les fichiers binaires terraform. 

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## Configuration du fournisseur {{site.data.keyword.Bluemix_notm}} pour Terraform
{: #setup}

Pour prendre en charge une approche multicloud, Terraform collabore avec des fournisseurs. Un fournisseur est chargé de la compréhension des interactions des API et de l'exposition des ressources. Dans cette section, vous allez configurer la CLI pour spécifier l'emplacement du fournisseur {{site.data.keyword.Bluemix_notm}}.

1. Vérifiez l'installation de Terraform en exécutant la commande `terraform` dans la fenêtre du terminal ou de l'invite de commande. La liste des `Commandes courantes` devrait apparaître.

  ```
  terraform
  ```

2. Téléchargez le plug-in [{{site.data.keyword.Bluemix_notm}} Provider](https://github.com/IBM-Cloud/terraform-provider-ibm/releases) correspondant à votre système et extrayez l'archive. Vous devriez voir le fichier de plug-in binaire `terraform-provider-ibm_VERSION`.

3. Pour les systèmes autres que Windows, créez un répertoire `.terraform.d/plugins` dans le répertoire de base de votre utilisateur et placez-y le fichier binaire. Utilisez les commandes suivantes pour référence. 

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    Sous **Windows**, le fichier doit être placé dans `terraform.d/plugins` sous le répertoire "Application Data" de votre utilisateur.

  - Exécutez les commandes ci-dessous à l'invite de commande [Configuration du fournisseur](https://www.terraform.io/docs/configuration/providers.html)
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - Lancez **Windows Powershell** (Start + R > Powershell) et exécutez la commande ci-dessous pour créer le fichier `terraform.rc`
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   A la première invite, entrez le contenu ci-dessous
   ```
    # ~/.terraformrc
    providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        PATH_TO_YOUR_APPDATA_PLUGINS doit être un chemin absolu avec une barre oblique (/). Par exemple, `C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe`
        {: tip}

  - Cliquez sur Entrée pour quitter l'invite. 

## Préparation de la configuration de terraform 

{: #terraformconfig}

Dans cette section, vous allez apprendre les bases d'une configuration terraform à l'aide d'un exemple de configuration Terraform fourni par {{site.data.keyword.Bluemix_notm}}.

1. Consultez https://github.com/IBM-Cloud/LAMP-terraform-ibm et **déviez** votre propre copie sur votre compte.
2. Clonez la déviation localement : 
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. Examinez les fichiers de configuration
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml) - contient les configurations d'installation du serveur. Vous pouvez y ajouter tous les scripts liés à l'installation de votre serveur aux éléments à installer sur le serveur. Consultez la fonction `phpinfo();` injectée dans ce fichier.
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf) - contient les variables liées au fournisseur où se trouvent le nom d'utilisateur et la clé d'API du fournisseur nécessaires.
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) - contient les configurations de serveur pour déployer la machine virtuelle avec les variables spécifiées.
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) - contient le nom d'utilisateur et la clé d'API **SoftLayer**, la clé d'API {{site.data.keyword.Bluemix_notm}} et vos noms d'espace/organisation. selon les bonnes pratiques, ces données d'identification peuvent être ajoutées à ce fichier afin d'éviter de les saisir à nouveau à partir de la ligne de commande lors de chaque déploiement du serveur. Remarque : NE publiez PAS ce fichier avec vos données d'identification.
4. Ouvrez le fichier [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) dans l'environnement IDE de votre choix et modifiez-le en ajoutant la clé **SSH publique**. Il vous permet d'accéder à la machine virtuelle créée par cette configuration. Pour plus d'informations sur la création de clés SSH, appliquez les instructions de [ce lien](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html). Pour copier la clé publique dans votre presse-papiers, vous pouvez exécuter la commande ci-dessous dans votre terminal. 

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     Cette commande copie le SSH dans votre presse-papiers. Vous pouvez ensuite le coller dans le fichier [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) sous la variable par défaut `ssh_key` autour de la ligne 69.

    Sous **Windows**, téléchargez, installez, lancez [Git Bash](https://git-scm.com/download/win) et exécutez la commande ci-dessous pour copier la clé SSH publique dans votre presse-papiers.

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. Ouvrez le fichier [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) avec l'environnement IDE, modifiez-le en ajoutant toutes les données d'identification répertoriées. L'ajout des données d'identification dans ces fichiers vous permet de ne plus avoir à saisir ces données chaque fois que vous exécutez terraform. Vous devez ajouter les cinq données d'identification répertoriées dans le fichier [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) afin de terminer ce tutoriel. 

## Création d'un serveur de pile LAMP à partir de la configuration terraform 
{: #Createserver}
Dans cette section, vous allez apprendre à créer un serveur de pile LAMP à partir de l'exemple de configuration de terraform. Cette configuration permet de mettre à disposition une instance de machine virtuelle et installer **A**pache, **M**ySQL (**M**ariaDB) et **P**HP sur cette instance.

1. Accédez au dossier du référentiel que vous avez cloné. 
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. Initialisez la configuration de terraform. Cette opération installe également le plug-in `terraform-provider-ibm_VERSION`.
   ````bash
    terraform init
   ````
   {: pre}
3. Appliquez la configuration de terraform. Vous créez ainsi les ressources définies dans la configuration. 
   ```
    terraform apply
   ```
   {: pre}
   Une sortie semblable à celle ci-dessous devrait apparaître.![URL de contrôle de source](images/solution10/created.png)
4. Accédez ensuite à la [liste des unités de votre infrastructure](https://{DomainName}/classic/devices) pour vérifier que le serveur a été créé. ![URL de contrôle de source](images/solution10/configuration.png)

## Ajout du service {{site.data.keyword.cos_full_notm}} et mise à l'échelle des ressources

{: #modify}

Dans cette section, vous allez apprendre à mettre à l'échelle la ressource de serveur virtuel et ajouter un service [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) à votre environnement d'infrastructure.

1. Editez le fichier `vm.tf` pour augmenter les éléments suivants et enregistrez le fichier.
 - Augmentez le nombre de coeurs de processeur à 4 coeurs 
 - Augmentez la mémoire RAM à 4096 
 - Augmentez la taille du disque à 100 Go 

2. Ajoutez ensuite un nouveau service [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage). Pour ce faire, créez un fichier et nommez-le **ibm-cloud-object-storage.tf**. Ajoutez le fragment de code ci-dessous au fichier que vous venez de créer. Le fragment de code ci-dessous crée un nom de variable pour le nom de l'organisation et le nom de l'espace, puis ces deux noms de variable sont utilisés pour extraire l'identificateur global unique de l'espace où le service doit être créé. Il définit le nom du service {{site.data.keyword.cos_full_notm}} sur `lamp_objectstorage`. Vous devez ensuite disposer d'un identificateur global unique d'espace, d'un nom complet de service et d'un type de forfait. Le code ci-dessous crée un forfait payant dans la mesure où il s’agit d’un forfait de paiement à la carte. Vous pouvez également utiliser le forfait Lite, mais notez que ce forfait est limité à un seul service par compte. 

   ```
   variable "org_name" {
     description = "Enter your IBM Cloud org name, you can get your org name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   variable "space_name" {
     description = "Enter your IBM Cloud space name, you can get your space name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   data "ibm_space" "space" {
     space = "${var.space_name}"
     org   = "${var.org_name}"
   }

   # a cloud object storage
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # you can only have one Lite plan per account so let's use the Premium - it is pay-as-you-go
     plan = "Premium"
   }
   ```
   {: pre}
   **Remarque :** vous rechercherez ultérieurement l'étiquette "lamp_objectstorage" dans les journaux pour vous assurer que {{site.data.keyword.cos_full_notm}} a été créé.

3. Initialisez à nouveau la configuration de terraform en exécutant : 

   ```bash
    terraform init
   ```
   {: pre}

4. Appliquez les modifications de terraform en exécutant : 
   ```bash
    terraform apply
   ```
   {: pre}
   **Remarque :** une fois que vous avez exécuté la commande terraform apply, un nouveau fichier `terraform.tfstate` devrait être ajouté à votre répertoire. Ce fichier contient la confirmation complète du déploiement pour vous permettre de garder une trace de vos dernières modifications et de toute modification future de votre configuration. Si ce fichier est supprimé ou perdu, vous perdez vos configurations de déploiement terraform. 

## Vérification de la MV et d'{{site.data.keyword.cos_short}}
{: #verifyvm}

Dans cette section, vous allez vérifier la machine virtuelle et {{site.data.keyword.cos_short}} pour vous assurer que la création est correcte. 

**Vérifiez la machine virtuelle**

1. Dans le menu de gauche, cliquez sur **Infrastructure** pour afficher la liste des unités de serveur virtuel. 
2. Cliquez sur **Unités** -> **Liste des unités** pour localiser le serveur créé. Votre unité serveur devrait être répertoriée. 
3. Cliquez sur le serveur pour afficher plus d'informations sur sa configuration. Dans la capture d'écran ci-dessous, vous pouvez constater que le serveur a été créé. ![URL de contrôle de source](images/solution10/configuration.png)
4. Testez ensuite le serveur dans le navigateur Web. Ouvrez l'adresse IP publique du serveur dans le navigateur Web. La page d'installation par défaut du serveur ci-dessous devrait apparaître. ![URL de contrôle de source](images/solution10/LAMP.png)


**Vérifiez {{site.data.keyword.cos_full_notm}}**

1. Dans le **tableau de bord d'{{site.data.keyword.Bluemix_notm}}**, une nouvelle instance du service {{site.data.keyword.cos_full_notm}} que vous pouvez utiliser devrait apparaître. ![object-storage](images/solution10/ibm-cloud-object-storage.png)

   Vous trouverez plus d'informations sur {{site.data.keyword.cos_full_notm}} [ici](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage).

## Suppression de ressources 
{: #deleteresources}

Supprimez les ressources à l'aide de la commande suivante :
   ```bash
   terraform destroy
   ```
   {: pre}

**Remarque :** pour supprimer des ressources, vous devez disposer de droits d'administrateur de l'infrastructure. Si vous ne possédez pas de compte administrateur superutilisateur, veuillez demander l'annulation des ressources à l'aide du tableau de bord d'infrastructure. Vous pouvez demander l'annulation d'une unité dans le tableau de bord d'infrastructure sous Unités. ![object-storage](images/solution10/rm.png)

## Contenu associé

- [Terraform](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [Fournisseur d'{{site.data.keyword.Bluemix_notm}} pour Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [Accélération de la distribution de fichiers statiques à l'aide d'un CDN - {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

