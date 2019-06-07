---
copyright:
  years: 2019
lastupdated: "2019-04-02"

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
{:important: .important}

# Accès sécurisé aux instances distantes à l'aide d'un hôte bastion
{: #vpc-secure-management-bastion-server}

IBM va accepter un nombre limité de clients pour un programme d'accès anticipé à VPC qui commencera début avril 2019 ; l'accès élargi ne sera ouvert que dans les mois qui suivront. Si votre organisation souhaite avoir accès à IBM Virtual Private Cloud, veuillez remplir ce [formulaire de nomination](https://{DomainName}/vpc){: new_window} et un interlocuteur IBM entrera en contact avec vous concernant la procédure à suivre.
{: important}

Ce tutoriel vous guide dans le déploiement d'un hôte bastion pour accéder en toute sécurité à des instances distantes dans un cloud privé virtuel (Virtual Private Cloud - VPC). L'hôte Bastion est une instance mise à disposition dans un sous-réseau public et accessible via SSH. Une fois configuré, l'hôte bastion agit comme un serveur de **saut** permettant une connexion sécurisée aux instances mises à disposition dans un sous-réseau privé. 

Pour réduire l'exposition des serveurs au sein du VPC, vous allez créer et utiliser un hôte bastion. Les tâches administratives sur les serveurs individuels sont effectuées à l'aide de SSH, via un proxy du bastion. L’accès aux serveurs et l’accès Internet normal à partir des serveurs, par exemple pour l’installation de logiciels, ne sont autorisés que si un groupe de sécurité de maintenance spécial est connecté à ces serveurs.
{:shortdesc}

## Objectifs
{: #objectives}

* Apprendre à configurer un hôte bastion et des groupes de sécurité dotés de règles 
* Gestion sécurisée des serveurs via l'hôte bastion 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants :   

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)  
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

  ![Architecture](images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png)

1. Après avoir configuré l'infrastructure requise (sous-réseaux, groupes de sécurité avec règles, VSI) sur le cloud, l'administrateur (DevOps) se connecte (SSH) à l'hôte bastion à l'aide de la clé SSH privée. 
2. L'administrateur affecte un groupe de sécurité de maintenance doté des règles sortantes appropriées. 
3. L'administrateur se connecte (SSH) en toute sécurité à l'adresse IP privée de l'instance via l'hôte bastion pour installer ou mettre à jour tout logiciel requis, par exemple un serveur Web. 
4. L'internaute adresse une requête HTTP/HTTPS au serveur Web. 

## Avant de commencer
{: #prereqs}

- Vérifiez les droits utilisateur. Assurez-vous que votre compte d'utilisateur dispose des droits suffisants pour créer et gérer des ressources VPC. Pour obtenir la liste des droits requis, voir [Granting permissions needed for VPC users](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Une clé SSH est nécessaire pour la connexion aux serveurs virtuels. Si vous ne possédez pas de clé SSH, consultez les [instructions de création d'une clé](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites). 
- Ce tutoriel suppose que vous ajoutez l'hôte bastion dans un [cloud privé virtuel](https://{DomainName}/vpc/network/vpcs) existant. **Si vous ne disposez pas de cloud privé virtuel dans votre compte, créez-en un avant de passer aux étapes suivantes.**

## Création d'un hôte bastion
{: #create-bastion-host}

Dans cette section, vous allez créer et configurer un hôte bastion avec un groupe de sécurité dans un sous-réseau séparé. 

### Création d'un sous-réseau
{: #create-bastion-subnet}

1. Cliquez sur **Sous-réseaux** sous **Réseau** dans le volet gauche, puis sur **Nouveau sous-réseau**.  
   * Entrez **vpc-secure-bastion-subnet** en tant que nom, puis sélectionnez le VPC que vous avez créé.  
   * Sélectionnez un emplacement et une zone.  
   * Entrez la plage d'adresses IP du sous-réseau en notation CIDR, à savoir **10.xxx.0.0/24**. Laissez le **Préfixe d'adresse** tel quel et sélectionnez le **Nombre d'adresses** 256.
1. Sélectionnez **Utiliser la valeur par défaut de VPC** pour la liste de contrôle d'accès (ACL) du sous-réseau. Vous pouvez configurer les règles entrantes et sortantes ultérieurement. 
1. Modifiez la **Passerelle publique** sur **Connecté**. 
1. Cliquez sur **Créer un sous-réseau** pour le mettre à disposition.

### Création et configuration d'un groupe de sécurité de bastion 

Créez un groupe de sécurité et configurez les règles entrantes dans la VSI du bastion. 

1. Accédez aux **Groupes de sécurité** et cliquez sur **Nouveau groupe de sécurité**. Entrez **vpc-secure-bastion-sg** comme nom et sélectionnez le VPC.  
2. Créez maintenant les règles entrantes suivantes en cliquant sur **Ajouter une règle** dans la section entrante. Elles permettent l'accès SSH et Ping (ICMP). 
 
	**Règle entrante :**
	<table>
	   <thead>
	      <tr>
	         <td><strong>Source</strong></td>
	         <td><strong>Protocole</strong></td>
	         <td><strong>Valeur</strong></td>
	      </tr>
	   <tbody>
	      <tr>
	         <td>Indifférent - 0.0.0.0/0 </td>
	         <td>TCP</td>
	         <td>De : <strong>22</strong> à <strong>22</strong></td>
	      </tr>
         <tr>
            <td>Indifférent - 0.0.0.0/0 </td>
	         <td>ICMP</td>
	         <td>Type : <strong>8</strong>, Code : <strong>Leave empty</strong></td>
         </tr>
	   </tbody>
	</table>

   Pour renforcer la sécurité, le trafic entrant peut être limité au réseau de l'entreprise ou à un réseau domestique typique. Vous pouvez exécuter `curl ipecho.net/plain ; echo` pour obtenir l'adresse IP externe de votre réseau et l'utiliser à la place.
   {:tip }

### Création d'une instance bastion
Une fois le sous-réseau et le groupe de sécurité en place, créez l'instance de serveur virtuel bastion. 

1. Sous **Sous-réseaux** dans le volet gauche, sélectionnez **vpc-secure-bastion-subnet**.
2. Cliquez sur **Instances connectées** et mettez à disposition une **Nouvelle instance** appelée **vpc-secure-vsi** sous votre propre VPC. Sélectionnez Ubuntu Linux comme image et **c-2x4** (2 vCPU et 4 Go de RAM) comme profil.
3. Sélectionnez un **Emplacement** et veillez à utiliser le même emplacement ultérieurement.
4. Pour créer une **Clé SSH**, cliquez sur **Nouvelle clé**
   * Entrez **vpc-ssh-key** comme nom de clé.
   * Laissez la **Région** telle quelle.
   * Copiez le contenu de votre clé SSH locale existante sous **Clé publique**.  
   * Cliquez sur **Ajouter une clé SSH**.
5. Sous **Interfaces réseau**, cliquez sur l'icône **Editer** en regard des groupes de sécurité 
   * Assurez-vous que **vpc-secure-subnet** est sélectionné comme sous-réseau.
   * Désélectionnez le groupe de sécurité par défaut et cochez **vpc-secure-bastion-sg**.
   * Cliquez sur **Sauvegarder**.
6. Cliquez sur **Créer une instance de serveur virtuel**.
7. Une fois l'instance sous tension, cliquez sur **vpc-secure-bastion-vsi** et **réservez** une adresse IP flottante.

### Test de l'hôte bastion

Une fois activée l'adresse IP flottante du bastion, tentez de vous y connecter à l'aide de **ssh** :

   ```sh
   ssh -i ~/.ssh/<PRIVATE_KEY> root@<BASTION_FLOATING_IP_ADDRESS>
   ```
   {:pre}

## Configuration d'un groupe de sécurité avec des règles d'accès de maintenance 
{: #maintenance-security-group}

Une fois l'accès au bastion établi, poursuivez et créez le groupe de sécurité pour les tâches de maintenance, telles que l'installation et la mise à jour du logiciel. 

1. Accédez à **Groupes de sécurité** et mettez à disposition un nouveau groupe de sécurité appelé **vpc-secure-maintenance-sg** avec les règles sortantes ci-dessous

   <table>
   <thead>
      <tr>
         <td><strong>Destination</strong></td>
         <td><strong>Protocole</strong></td>
         <td><strong>Valeur</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Indifférent - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>De : <strong>80</strong> à <strong>80</strong></td>
      </tr>
      <tr>
         <td>Indifférent - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>De : <strong>443</strong> à <strong>443</strong></td>
      </tr>
       <tr>
         <td>Indifférent - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>De : <strong>53</strong> à <strong>53</strong></td>
      </tr>
      <tr>
         <td>Indifférent - 0.0.0.0/0 </td>
         <td>UDP</td>
         <td>De : <strong>53</strong> à <strong>53</strong></td>
      </tr>
   </tbody>
   </table>

   Les demandes de serveur DNS sont traitées sur le port 53. DNS utilise TCP pour le transfert de zone et UDP pour les requêtes de nom standard (principal) ou inversé. Les requêtes HTTP se trouvent sur les ports 80 et 443.
{:tip }

2. Ajoutez ensuite cette règle **entrante** qui autorise l’accès SSH à partir de l’hôte bastion. 

   <table>
	   <thead>
	      <tr>
	         <td><strong>Source</strong></td>
	         <td><strong>Protocole</strong></td>
	         <td><strong>Valeur</strong></td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>Type : <strong>Groupe de sécurité</strong> - Nom : <strong>vpc-secure-bastion-sg</strong></td>
	         <td>TCP</td>
	         <td>De : <strong>22</strong> à <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>

3. Créez le groupe de sécurité.
4. Accédez à **Tous les groupes de sécurité du VPC**, puis sélectionnez **vpc-secure-sg**.
5. Enfin, éditez le groupe de sécurité et ajoutez la règle **sortante** suivante. 

   <table>
	   <thead>
	      <tr>
	         <td><strong>Destination</strong></td>
	         <td><strong>Protocole</strong></td>
	         <td><strong>Valeur</strong></td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>Type : <strong>Groupe de sécurité</strong> - Nom : <strong>vpc-secure-maintenance-sg</strong></td>
	         <td>TCP</td>
	         <td>De : <strong>22</strong> à <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>


## Utilisation de l'hôte bastion pour accéder à d'autres instances du VPC 
{: #bastion-host-access-instances}

Dans cette section, vous créez un sous-réseau privé avec une instance de serveur virtuel et un groupe de sécurité. Par défaut, tout sous-réseau créé dans un VPC est privé. 

Si vous disposez déjà d'instances de serveur virtuel dans votre VPC auxquelles vous souhaitez vous connecter, vous pouvez ignorer les trois sections suivantes et commencer à [ajouter vos instances de serveur virtuel au groupe de sécurité de maintenance.](#add-vsi-to-maintenance).

### Création d'un sous-réseau
{: #create-private-subnet}

Pour créer un sous-réseau, 

1. Cliquez sur **Sous-réseaux** sous **Réseau** dans le volet gauche, puis sur **Nouveau sous-réseau**.  
   * Entrez **vpc-secure-private-subnet** en tant que nom, puis sélectionnez le VPC que vous avez créé.  
   * Sélectionnez un emplacement.   
   * Entrez la plage d'adresses IP du sous-réseau en notation CIDR, à savoir **10.xxx.1.0/24**. Laissez le **Préfixe d'adresse** tel quel et sélectionnez le **Nombre d'adresses** 256.
1. Sélectionnez **Utiliser la valeur par défaut de VPC** pour la liste de contrôle d'accès (ACL) du sous-réseau. Vous pouvez configurer les règles entrantes et sortantes ultérieurement. 
1. Modifiez la **Passerelle publique** sur **Connecté**. 
1. Cliquez sur **Créer un sous-réseau** pour le mettre à disposition.

### Création d'un groupe de sécurité

Pour créer un groupe de sécurité :   
1. Cliquez sur **Groupes de sécurité** sous Réseau, puis sur **Nouveau groupe de sécurité**.  
2. Entrez **vpc-secure-private-sg** en tant que nom et sélectionnez le VPC que vous avez créé précédemment.   
3. Cliquez sur **Créer un groupe de sécurité**.  

### Création d'une instance de serveur virtuel

Pour créer une instance de serveur virtuel sur le sous-réseau que vous venez de créer :

1. Cliquez sur le sous-réseau privé sous **Sous-réseaux**.
2. Cliquez sur **Instances connectées**, puis sur **Nouvelle instance**.
3. Entrez un nom unique, **vpc-secure-private-vsi**, sélectionnez le VPC créé précédemment, puis le même **Emplacement** que précédemment. 
4. Sélectionnez l'image **Ubuntu Linux**, cliquez sur **Tous les profils** et, sous **Calcul**, sélectionnez **c-2x4** avec 2 UC et 4 Go de RAM. 
5. Pour les **Clés SSH**, choisissez la clé SSH que vous avez créée plus tôt pour le bastion.
6. Sous **Interfaces réseau**, cliquez sur l'icône **Editer** en regard des groupes de sécurité   
   * Sélectionnez **vpc-secure-private-subnet** en tant que sous-réseau.  
   * Décochez le groupe de sécurité par défaut et activez **vpc-secure-private-sg**.  
   * Cliquez sur **Sauvegarder**.  
7. Cliquez sur **Créer une instance de serveur virtuel**.  


### Ajout de serveurs virtuels au groupe de sécurité de maintenance 
{: #add-vsi-to-maintenance}

Pour les tâches administratives sur les serveurs, vous devez associer les serveurs virtuels spécifiques au groupe de sécurité de maintenance. Dans la section suivante, vous activez la maintenance, vous vous connectez au serveur privé, mettez à jour les informations sur le package logiciel, puis dissociez à nouveau le groupe de sécurité. 

Activez le groupe de sécurité de maintenance pour le serveur. 

1. Accédez aux **Groupes de sécurité** et sélectionnez le groupe de sécurité **vpc-secure-maintenance-sg**.   
2. Cliquez sur **Interfaces connectées**, puis sur **Editer les interfaces**.  
3. Développez les instances de serveur virtuel et activez la sélection en regard de **principal** dans la colonne **Interfaces**.
4. Cliquez sur **Sauvegarder** pour que les modifications soient appliquées.

### Connexion à l'instance

Pour SSH dans une instance utilisant son **IP privée**, vous utilisez l'hôte bastion comme **hôte de saut**. 

1. Obtenez l'adresse IP privée d'une instance de serveur virtuel sous **Instances de serveur virtuel**.
2. Utilisez la commande ssh avec `-J` pour vous connecter au serveur avec l'adresse **IP flottante** du bastion que vous avez utilisée précédemment et l'adresse **IP privée** du serveur indiquée sous **Interfaces réseau**.

   ```sh
   ssh -J root@<BASTION_FLOATING_IP_ADDRESS> root@<PRIVATE_IP_ADDRESS>
   ```
   {:pre}
   
   L'option `-J` est prise en charge dans OpenSSH version 7.3+. Dans les versions antérieures, `-J` n'est pas disponible. Dans ce cas, le moyen le plus sûr et le plus simple consiste à utiliser le mode de transfert stdio (`-W`) de ssh pour "faire rebondir" la connexion via un hôte bastion, par exemple, `ssh -o ProxyCommand="ssh -W %h:%p root@<BASTION_FLOATING_IP_ADDRESS" root@<PRIVATE_IP_ADDRESS>`
   {:tip }

### Installation du logiciel et réalisation des tâches de maintenance 

Une fois connecté, vous pouvez installer un logiciel sur le serveur virtuel du sous-réseau privé ou effectuer des tâches de maintenance. 

1. Tout d'abord, mettez à jour les informations sur le package logiciel : 
   ```sh
   apt-get update
   ```
   {:pre}
2. Installez le logiciel souhaité, par exemple Nginx ou MySQL ou IBM Db2. 

Une fois terminé, déconnectez-vous du serveur à l'aide de la commande `exit`.  

Pour autoriser les requêtes HTTP/HTTPS par l'internaute, attribuez une adresse **IP flottante** à l'instance VSI du sous-réseau privé et ouvrez les ports requis (80 - HTTP et 443 - HTTPS) via les règles entrantes du groupe de sécurité de l'instance VSI privée.
{:tip}

### Désactivation du groupe de sécurité de maintenance 

Une fois que vous avez terminé l’installation du logiciel ou effectué la maintenance, vous devez supprimer les serveurs virtuels du groupe de sécurité de maintenance pour les maintenir isolés. 

1. Accédez aux **Groupes de sécurité** et sélectionnez le groupe de sécurité **vpc-secure-maintenance-sg**.   
2. Cliquez sur **Interfaces connectées**, puis sur **Editer les interfaces**.  
3. Développez les instances de serveur virtuel et décochez la sélection en regard de **principal** dans la colonne **Interfaces**.
4. Cliquez sur **Sauvegarder** pour que les modifications soient appliquées.

## Suppression de ressources 
{: #removeresources}

1. Accédez à **Instances de serveur virtuel** et **supprimez** vos instances. Les instances sont alors supprimées et leur statut **Suppression** est conservé pendant un certain temps. Veillez à actualiser le navigateur de temps en temps.
2. Une fois les instances VSI supprimées, passez aux **Sous-réseaux** et supprimez-les. 
4. Une fois les sous-réseaux supprimés, accédez à l'onglet **Clouds privés virtuels** et supprimez votre VPC. 

Lorsque vous utilisez la console, vous devrez peut-être actualiser votre navigateur pour consulter les informations d'état mises à jour après la suppression d'une ressource.
{:tip}

## Contenu associé
{: #related}

* [Sous-réseaux privés et publics dans un cloud privé virtuel](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend)
