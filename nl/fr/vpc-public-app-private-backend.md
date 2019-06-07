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

# Sous-réseaux privés et publics dans un cloud privé virtuel
{: #vpc-public-app-private-backend}

IBM va accepter un nombre limité de clients pour un programme d'accès anticipé à VPC qui commencera début avril 2019 ; l'accès élargi ne sera ouvert que dans les mois qui suivront. Si votre organisation souhaite avoir accès à IBM Virtual Private Cloud, veuillez remplir ce [formulaire de nomination](https://{DomainName}/vpc){: new_window} et un interlocuteur IBM entrera en contact avec vous concernant la procédure à suivre.
{: important}

Ce tutoriel vous guide dans la création de votre propre {{site.data.keyword.vpc_full}} (Virtual Private Cloud - VPC) avec un sous-réseau public et privé et une instance de serveur virtuel (Virtual Server Instance - VSI) dans chaque sous-réseau. Un VPC est votre propre cloud privé sur une infrastructure de cloud partagé avec un isolement logique des autres réseaux virtuels. 

Un [sous-réseau](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#subnet) est une plage d'adresses IP. Il est lié à une seule zone et ne peut pas couvrir plusieurs zones ou régions. Pour les besoins du VPC, la caractéristique importante d'un sous-réseau est le fait que les sous-réseaux peuvent être isolés les uns des autres ou interconnectés de la manière habituelle. Un sous-réseau peut être isolé à l'aide de [listes de contrôle d'accès](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#access-control-list) (Access Control Lists - ACL) au réseau qui font office de pare-feux pour contrôler le flux des paquets de données entre les sous-réseaux. De même, les [groupes de sécurité](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#security-group) (Security Groups -SG) agissent comme des pare-feux virtuels pour contrôler le flux de paquets de données à destination et en provenance des différentes instances VSI. 

Le sous-réseau public est utilisé pour les ressources devant être exposées au monde extérieur. Les ressources dont l'accès est restreint et qui ne doivent jamais être directement accessibles par le public sont placées dans l'enceinte du sous-réseau privé. Les instances sur ce type de sous-réseau peuvent être votre base de données back-end ou un stockage secret que vous ne souhaitez pas rendre accessible au public. Vous allez définir des SG pour autoriser ou refuser le trafic vers les instances VSI.
{:shortdesc}

En résumé, à l'aide d'un VPC, vous pouvez 

- créer un réseau défini par logiciel (Software-Defined Network - SDN), 
- isoler des charges de travail, 
- obtenir un contrôle approfondi du trafic entrant et sortant. 

## Objectifs

{: #objectives}

- Comprendre les objets d'infrastructure disponibles pour les clouds privés virtuels 
- Apprendre à créer un cloud privé virtuel, des sous-réseaux et des instances 
- Savoir comment appliquer des groupes de sécurité pour sécuriser l'accès aux serveurs 

## Services utilisés

{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

![Architecture](images/solution40-vpc-public-app-private-backend/Architecture.png)


1. L'administrateur (DevOps) configure l'infrastructure requise (VPC, sous-réseaux, groupes de sécurité avec règles, VSI) sur le cloud. 
2. L'internaute envoie une requête HTTP/HTTPS au serveur Web sur le front-end. 
3. Le front-end demande des ressources privées au serveur de back-end sécurisé et fournit des résultats à l'utilisateur. 

## Avant de commencer

{: #prereqs}

- Vérifiez les droits utilisateur. Assurez-vous que votre compte d'utilisateur dispose des droits suffisants pour créer et gérer des ressources VPC. Pour obtenir la liste des droits requis, voir [Granting permissions needed for VPC users](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).

- Une clé SSH est nécessaire pour la connexion aux serveurs virtuels. Si vous ne possédez pas de clé SSH, consultez les [instructions de création d'une clé](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites). 

## Création d'un cloud privé virtuel
{: #create-vpc}

Pour créer votre propre {{site.data.keyword.vpc_short}},

1. Accédez à la page [VPC - Présentation](https://{DomainName}/vpc/overview), et cliquez sur **Créer un VPC**.
2. Dans la section **Nouveau cloud privé virtuel** :
   * Entrez **vpc-pubpriv** comme nom de VPC.
   * Sélectionnez un **Groupe de ressources**.
   * Ajoutez éventuellement des **Balises** pour organiser vos ressources.
3. Sélectionnez **Créer une nouvelle valeur par défaut (Autoriser tout)** en tant que liste de contrôle d'accès (Access Control List - ACL) par défaut de votre VPC.
1. Désélectionnez SSH, exécutez la commande ping à partir du **Groupe de sécurité par défaut**. 
4. Sous **Nouveau sous-réseau pour VPC** :
   * Comme nom unique, entrez **vpc-pubpriv-backend-subnet**.
   * Sélectionnez un emplacement. 
   * Entrez la plage d'adresses IP du sous-réseau en notation CIDR, à savoir **10.xxx.0.0/24**. Laissez le **Préfixe d'adresse** tel quel et sélectionnez le **Nombre d'adresses** 256.
5. Sélectionnez **Utiliser la valeur par défaut de VPC** pour la liste de contrôle d'accès (ACL) du sous-réseau. Vous pouvez configurer les règles entrantes et sortantes ultérieurement. 
6. Cliquez sur **Créer une instance de serveur virtuel** pour mettre l'instance à disposition.

Si les instances VSI connectées au sous-réseau privé ont besoin d'un accès à Internet pour charger le logiciel, réglez la passerelle publique sur **Connecté**, car la connexion d'une passerelle publique permet à toutes les ressources connectées de communiquer avec Internet. Une fois tous les logiciels nécessaires copiés dans les instances VSI, ramenez la passerelle publique sur **Déconnecté** afin que le sous-réseau ne puisse plus accéder à Internet.
{: important}

Pour confirmer la création du sous-réseau, cliquez sur **Sous-réseaux** dans le volet de gauche et attendez que l'état passe à **Disponible**. Vous pouvez créer un autre sous-réseau sous **Sous-réseaux**.

## Création d'un groupe de sécurité de back-end et d'une instance VSI 
{: #backend-subnet-vsi}

Dans cette section, vous allez créer un groupe de sécurité et une instance de serveur virtuel pour le back-end. 

### Création d'un groupe de sécurité de back-end 

Par défaut, un groupe de sécurité est créé avec votre VPC, ce qui permet de diriger tout le trafic SSH (port TCP 22) et Ping (ICMP de type 8) vers les instances connectées. 

Pour créer un groupe de sécurité pour le back-end :   
1. Cliquez sur **Groupes de sécurité** sous **Réseau**, puis sur **Nouveau groupe de sécurité**.  
2. Entrez **vpc-pubpriv-backend-sg** en tant que nom et sélectionnez le VPC que vous avez créé précédemment.  
3. Cliquez sur **Créer un groupe de sécurité**.

Vous modifierez ensuite le groupe de sécurité pour ajouter les règles entrantes et sortantes. 

### Création d'une instance de serveur virtuel de back-end 

Pour créer une instance de serveur virtuel sur le sous-réseau que vous venez de créer :

1. Cliquez sur le sous-réseau de back-end sous **Sous-réseaux**.
2. Cliquez sur **Instances connectées**, puis sur **Nouvelle instance**.
3. Entrez un nom unique et choisissez **vpc-pubpriv-backend-vsi**. Sélectionnez ensuite le VPC que vous avez créé précédemment et l'**Emplacement** comme précédemment. 
4. Sélectionnez l'image **Ubuntu Linux**, cliquez sur **Tous les profils** et, sous **Calcul**, sélectionnez **c-2x4** avec 2 UC et 4 Go de RAM. 
5. Pour les **Clés SSH**, choisissez la clé SSH que vous avez créée plus tôt.
6. Sous **Interfaces réseau**, cliquez sur l'icône **Editer** en regard des groupes de sécurité
   * Sélectionnez **vpc-pubpriv-backend-subnet** en tant que sous-réseau.
   * Décochez le groupe de sécurité par défaut et cochez **vpc-pubpriv-backend-sg** comme étant actif.
   * Cliquez sur **Sauvegarder**.
7. Cliquez sur **Créer une instance de serveur virtuel**.

## Création d'un sous-réseau de front-end, d'un groupe de sécurité et d'une instance VSI 
{: #frontend-subnet-vsi}

De même que pour le back-end, vous créez un sous-réseau front-end avec une instance de serveur virtuel et un groupe de sécurité. 

### Création d'un sous-réseau pour le front-end 

Pour créer un sous-réseau pour le front-end,

1. Cliquez sur **Sous-réseaux** sous **Réseau** dans le volet gauche > **Nouveau sous-réseau**.
   * Entrez **vpc-pubpriv-frontend-subnet** en tant que nom, puis sélectionnez le VPC que vous avez créé.
   * Sélectionnez un emplacement. 
   * Entrez la plage d'adresses IP du sous-réseau en notation CIDR, à savoir **10.xxx.1.0/24**. Laissez le **Préfixe d'adresse** tel quel et sélectionnez le **Nombre d'adresses** 256.
1. Sélectionnez **Utiliser la valeur par défaut de VPC** pour la liste de contrôle d'accès (ACL) du sous-réseau. Vous pouvez configurer les règles entrantes et sortantes ultérieurement. 
1. Dans la mesure où toutes les instances de serveur virtuel du sous-réseau possèdent une adresse IP flottante, il n'est pas nécessaire d'activer une passerelle publique pour le sous-réseau. Les instances de serveur virtuel sont reliées à Internet via leur adresse IP flottante. 
1. Cliquez sur **Créer un sous-réseau** pour le mettre à disposition.

### Création d'un groupe de sécurité de front-end 

Pour créer un groupe de sécurité pour le front-end : 
1. Cliquez sur **Groupes de sécurité** sous Réseau, puis sur **Nouveau groupe de sécurité**.
2. Entrez **vpc-pubpriv-frontend-sg** en tant que nom et sélectionnez le VPC que vous avez créé précédemment.
3. Cliquez sur **Créer un groupe de sécurité**.

### Création d'une instance de serveur virtuel de front-end  

Pour créer une instance de serveur virtuel sur le sous-réseau que vous venez de créer :

1. Cliquez sur le sous-réseau de front-end sous **Sous-réseaux**.
2. Cliquez sur **Instances connectées**, puis sur **Nouvelle instance**.
3. Entrez un nom unique, **vpc-pubpriv-frontend-vsi**, sélectionnez le VPC créé précédemment, puis le même **Emplacement** que précédemment. 
4. Sélectionnez l'image **Ubuntu Linux**, cliquez sur **Tous les profils** et, sous **Calcul**, sélectionnez **c-2x4** avec 2 UC et 4 Go de RAM. 
5. Pour les **Clés SSH**, choisissez la clé SSH que vous avez créée plus tôt.
6. Sous **Interfaces réseau**, cliquez sur l'icône **Editer** en regard des groupes de sécurité
   * Sélectionnez **vpc-pubpriv-frontend-subnet** en tant que sous-réseau.
   * Décochez le groupe de sécurité par défaut et activez **vpc-pubpriv-frontend-sg**.
   * Cliquez sur **Sauvegarder**.
   * Cliquez sur **Créer une instance de serveur virtuel**.
7. Patientez jusqu'à ce que le statut de l'instance VSI passe à **Démarré**. Sélectionnez ensuite l'instance VSI de front-end **vpc-pubpriv-frontend-vsi**, faites défiler l'écran jusqu'à **Interfaces réseau** et cliquez sur **Réserver** sous **IP flottante** pour associer une adresse IP publique à votre instance VSI de front-end. Enregistrez l'adresse IP associée dans un presse-papiers pour référence ultérieure. 

## Configuration de la connectivité entre le front-end et le back-end 
{: #setup-connectivity-frontend-backend}

Une fois tous les serveurs en place, dans cette section, vous configurez la connectivité pour permettre des opérations régulières entre les serveurs de front-end et de back-end. 

### Configuration d'un groupe de sécurité de front-end 

1. Accédez à **Groupes de sécurité** dans la section **Réseau**, puis cliquez sur **vpc-pubpriv-frontend-sg**.
2. Tout d’abord, ajoutez les règles **entrantes** suivantes à l’aide de **Ajouter une règle**. Elles autorisent les requêtes HTTP entrantes et Ping (ICMP). 

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
	      <td>ICMP</td>
	      <td>Type : <strong>8</strong>, Code : <strong>Leave empty</strong></td>
      </tr>
   </tbody>
   </table>

3. Ensuite, ajoutez ces règles **sortantes**.

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
         <td>Type : <strong>Groupe de sécurité</strong> - Nom : <strong>vpc-pubpriv-backend-sg</strong></td>
         <td>TCP</td>
         <td>Port du serveur de back-end, voir le conseil. </td>
      </tr>
   </tbody>
   </table>

Les ports des services de back-end types sont les suivants : MySQL utilise le port 3306, PostgreSQL utilise le port 5432. Db2 est accessible sur le port 50000 ou 50001. Microsoft SQL Server utilise par défaut le port 1433. L'une des nombreuses [listes avec les ports courants se trouve sur Wikipedia](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers).
{:tip }

### Configuration d'un groupe de sécurité de back-end 
De même que pour le front-end, configurez le groupe de sécurité pour le back-end. 

1. Accédez à **Groupes de sécurité** dans la section **Réseau**, puis cliquez sur **vpc-pubpriv-backend-sg**.
2. Ajoutez la règle **entrante** suivante à l'aide de **Ajouter une règle**. Cette opération permet une connexion au service de back-end. 

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
         <td>Type : <strong>Groupe de sécurité</strong> - Nom : <strong>vpc-pubpriv-frontend-sg</strong></td>
         <td>TCP</td>
         <td>Port du serveur de back-end </td>
      </tr>
   </tbody>
   </table>


## Installation du logiciel et réalisation des tâches de maintenance 
{: #install-software-maintenance-tasks}

Appliquez les étapes décrites dans [Accès sécurisé aux instances distantes à l'aide d'un hôte bastion](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) pour effectuer une maintenance sécurisée des serveurs à l'aide d'un hôte bastion faisant office de serveur `jump` et d'un groupe de sécurité de maintenance. 

## Suppression de ressources 
{: #remove-resources}

1. Dans la console de gestion VPC, cliquez sur **IP flottantes**, puis sur l'adresse IP de vos instances VSI, puis dans le menu d'action, sélectionnez **Libérer**. Confirmez que vous souhaitez libérer l'adresse IP. 
2. Ensuite, accédez à **Instances de serveur virtuel** et **Supprimez** vos instances. Les instances sont supprimées et leur statut **Suppression** est conservé pendant un certain temps. Veillez à actualiser le navigateur de temps en temps. 
3. Une fois les instances VSI supprimées, passez aux **Sous-réseaux**. Si le sous-réseau est connecté à une passerelle publique, cliquez sur le nom du sous-réseau. Dans les détails du sous-réseau, déconnectez la passerelle publique. Les sous-réseaux sans passerelle publique peuvent être supprimés de la page de présentation. Supprimez les sous-réseaux. 
4. Une fois les sous-réseaux supprimés, cliquez sur l'onglet **VPC** et supprimez le VPC. 

Lorsque vous utilisez la console, vous devrez peut-être actualiser votre navigateur pour consulter les informations d'état mises à jour après la suppression d'une ressource.
{:tip}

## Extension du tutoriel 
{: #expand-tutorial}

Vous souhaitez enrichir ou développer ce tutoriel ? Voici quelques suggestions :

- Ajoutez un [équilibreur de charge](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-load-balancer) pour répartir le trafic entrant sur plusieurs instances.
- Créez un [réseau privé virtuel](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn) (Virtual Private Network - VPN) afin que votre VPC puisse se connecter en toute sécurité à un autre réseau privé, tel qu'un réseau local ou un autre VPC.


## Contenu associé
{: #related}

- [Glossaire de VPC ](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)
- [VPC utilisant l'interface CLI IBM Cloud](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli#creating-a-vpc-using-the-ibm-cloud-cli)
- [VPC utilisant les API REST](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis#creating-a-vpc-using-the-rest-apis)
