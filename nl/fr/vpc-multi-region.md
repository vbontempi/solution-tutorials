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

# Déploiement de charges de travail isolées sur plusieurs emplacements et zones 
{: #vpc-multi-region}

IBM va accepter un nombre limité de clients pour un programme d'accès anticipé à VPC qui commencera début avril 2019 ; l'accès élargi ne sera ouvert que dans les mois qui suivront. Si votre organisation souhaite avoir accès à IBM Virtual Private Cloud, veuillez remplir ce [formulaire de nomination](https://{DomainName}/vpc){: new_window} et un interlocuteur IBM entrera en contact avec vous concernant la procédure à suivre.
{: important}

Ce tutoriel vous guide dans les étapes de configuration de charges de travail isolées en mettant à disposition des clouds privés virtuels (Virtual Private Cloud - VPC) dans différentes régions d'IBM Cloud. Ces régions comportent des sous-réseaux et des instances de serveur virtuel (Virtual Server Instance - VSI). Ces instances VSI sont créées dans plusieurs zones d'une même région pour accroître la résilience de cette région. Au niveau global, sont configurés des équilibreurs de charge avec des pools de back-end, des écouteurs de front-end et de contrôles de santé adéquats. 

Pour configurer un équilibreur de charge global, vous devez mettre à disposition un service IBM Cloud Internet Services (CIS) à partir du catalogue. Pour gérer le certificat SSL de toutes les demandes HTTPS entrantes, le service de catalogue {{site.data.keyword.cloudcerts_long_notm}} est créé et le certificat ainsi que la clé privée sont importés. 

{:shortdesc}

## Objectifs
{: #objectives}

* Comprendre le concept d'isolement des charges de travail à l'aide des objets d'infrastructure disponibles pour les clouds privés virtuels. 
* Utiliser un équilibreur de charge entre les zones d'une région pour répartir le trafic entre des serveurs virtuels. 
* Utiliser un équilibreur de charge global entre les régions pour augmenter la résilience et réduire les temps de réponse. 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.loadbalancer_full}}](https://{DomainName}/vpc/provision/loadBalancer)
- IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
- [{{site.data.keyword.cloudcerts_long_notm}}](https://{DomainName}/catalog/services/cloudcerts)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/estimator/review) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

  ![Architecture](images/solution41-vpc-multi-region/Architecture.png)

1. L'administrateur (DevOps) met à disposition des instances VSI dans des sous-réseaux situés dans deux zones différentes d'un VPC de la région 1 et reproduit cette action dans un VPC créé dans la région 2. 
2. L'administrateur crée un équilibreur de charge avec un pool de back-end de serveurs de sous-réseaux situés dans différentes zones de la région 1 et un écouteur de front-end. Il reproduit cette action dans la région 2.
3. L'administrateur met à disposition un service Cloud Internet Services avec un domaine personnalisé associé et crée un équilibreur de charge global pointant sur les équilibreurs de charge créés dans deux VPC différents. 
4. L'administrateur active le chiffrement HTTPS en ajoutant le certificat SSL du domaine au service du gestionnaire de certificats. 
5. L'utilisateur Internet fait une demande HTTP/HTTPS et l'équilibreur de charge global gère cette demande. 
6. La demande est acheminée vers les équilibreurs de charge à la fois globaux et locaux. La demande est ensuite satisfaite par l'instance de serveur disponible. 

## Avant de commencer
{: #prereqs}

- Vérifiez les droits utilisateur. Assurez-vous que votre compte d'utilisateur dispose des droits suffisants pour créer et gérer des ressources VPC. Pour obtenir la liste des droits requis, voir [Granting permissions needed for VPC users](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Une clé SSH est nécessaire pour la connexion aux serveurs virtuels. Si vous ne possédez pas de clé SSH, consultez les [instructions de création d'une clé](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites). 
- Cloud Internet Services nécessite de posséder un domaine personnalisé pour pouvoir configurer le DNS de ce domaine et le diriger vers les serveurs de noms de Cloud Internet Services.Si vous ne possédez pas de domaine, vous pouvez en acheter un auprès d'un registraire tel que [godaddy.com](http://godaddy.com/).

## Création de VPC, de sous-réseaux et d'instances VSI 
{: #create-infrastructure}

Dans cette section, vous allez créer votre propre VPC dans la région 1 avec des sous-réseaux créés dans deux zones différentes de la région 1, et mettre à disposition des instances VSI. 

Pour créer votre propre {{site.data.keyword.vpc_short}} dans la région 1,

1. Accédez à la page [VPC - Présentation](https://{DomainName}/vpc/overview), et cliquez sur **Créer un VPC**.
2. Dans la section **Nouveau cloud privé virtuel** :
   * Entrez **vpc-region1** comme nom de VPC.
   * Sélectionnez un **Groupe de ressources**.
   * Ajoutez éventuellement des **Balises** pour organiser vos ressources.
3. Sélectionnez **Créer une nouvelle valeur par défaut (Autoriser tout)** en tant que liste de contrôle d'accès (Access Control List - ACL) par défaut de votre VPC.
4. Désélectionnez SSH, exécutez la commande ping à partir du **Groupe de sécurité par défaut** et conservez **accès classique** non sélectionné.
5. Sous **Nouveau sous-réseau pour VPC** :
   * Comme nom unique, entrez **vpc-region1-zone1-subnet**.
   * Sélectionnez un emplacement (par exemple, Dallas), appelons-le **région 1** et une zone de la région 1 (par exemple, Dallas 1), appelons-la **zone 1**.
   * Entrez la plage d'adresses IP du sous-réseau en notation CIDR, à savoir **10.xxx.0.0/24**. Laissez le **Préfixe d'adresse** tel quel et sélectionnez le **Nombre d'adresses** 256.
6. Sélectionnez **Utiliser la valeur par défaut de VPC** pour la liste de contrôle d'accès (ACL) du sous-réseau. Vous pouvez configurer les règles entrantes et sortantes ultérieurement. 
7. Dans la mesure où toutes les instances de serveur virtuel du sous-réseau possèdent une adresse IP flottante, il n'est pas nécessaire d'activer une passerelle publique pour le sous-réseau. Les instances de serveur virtuel sont reliées à Internet via leur adresse IP flottante. 
8. Cliquez sur **Créer une instance de serveur virtuel** pour mettre l'instance à disposition.

Pour confirmer la création du sous-réseau, cliquez sur **Sous-réseaux** dans le volet de gauche et attendez que l'état passe à **Disponible**. Vous pouvez créer un autre sous-réseau sous **Sous-réseaux**.

### Création d'un sous-réseau dans la zone 2 

1. Cliquez sur **Nouveau sous-réseau**, entrez **vpc-region1-zone2-subnet** comme nom unique pour votre sous-réseau et sélectionnez **vpc-region1** en tant que VPC.
2. Sélectionnez l'emplacement que vous avez appelé région 1 ci-dessus (par exemple, Dallas) et sélectionnez une autre zone dans la région 1 (par exemple, Dallas 2). Nommez la zone sélectionnée **zone 2**.
3. Entrez la plage d'adresses IP du sous-réseau en notation CIDR, à savoir **10.xxx.64.0/24**. Laissez le **Préfixe d'adresse** tel quel et sélectionnez le **Nombre d'adresses** 256.
4. Sélectionnez **Utiliser la valeur par défaut de VPC** pour la liste de contrôle d'accès (ACL) du sous-réseau. 

### Mise à disposition d'intances VSI
Une fois que le statut des sous-réseaux est passé à **Disponible**, 

1. Cliquez sur **vpc-region1-zone1-subnet**, puis sur **Instances connectées** et sur **Nouvelle instance**.
2. Entrez un nom unique et choisissez **vpc-region1-zone1-vsi**. Sélectionnez ensuite le VPC que vous avez créé précédemment et l'**Emplacement** ainsi que la **zone** comme précédemment. 
3. Sélectionnez l'image **Ubuntu Linux** de votre choix, cliquez sur **Tous les profils** et, sous **Calcul**, sélectionnez **c-2x4** avec 2 UC et 4 Go de RAM. 
4. Pour les **Clés SSH**, choisissez la clé SSH que vous avez créée initialement.
5. Sous **Interfaces réseau**, cliquez sur l'icône **Editer** en regard des groupes de sécurité
   * Vérifiez si **vpc-region1-zone1-subnet** est sélectionné en tant sous-réseau. Si tel n'est pas le cas, sélectionnez-le.
   * Cliquez sur **Sauvegarder**.
   * Cliquez sur **Créer une instance de serveur virtuel**.
6.  Patientez jusqu'à ce que le statut de l'instance VSI passe à **Démarré**. Sélectionnez ensuite l'instance VSI **vpc-region1-zone1-vsi**, faites défiler l'écran jusqu'à **Interfaces réseau** et cliquez sur **Réserver** sous **IP flottante** pour associer une adresse IP publique à votre instance VSI. Enregistrez l'adresse IP associée dans un presse-papiers pour référence ultérieure. 
7. **REPETEZ** les étapes 1 à 6 pour mettre à disposition une instance VSI dans la **zone 2** de la **région 1**.

Accédez à **VPC** et **Sous-réseaux** sous **Réseau** sur le volet gauche et **REPETEZ** les étapes ci-dessus pour mettre à disposition un nouveau VPC avec des sous-réseaux et des VSI dans la **région2** en respectant les conventions de dénomination précédentes.

## Installation et configuration du serveur Web sur les instances VSI 
{: #install-configure-web-server-vsis}

Appliquez les étapes décrites dans [Accès sécurisé aux instances distantes à l'aide d'un hôte bastion](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) pour effectuer une maintenance sécurisée des serveurs à l'aide d'un hôte bastion faisant office de serveur `jump` et d'un groupe de sécurité de maintenance. {:tip}

Une fois que vous avez réussi à vous connecter via SSH au serveur mis à disposition dans le sous-réseau de la zone 1 de la région 1, 

1. A l'invite, exécutez les commandes ci-dessous pour installer Nginx en tant que serveur Web 
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   {:codeblock}
2. Vérifiez l'état du service Nginx à l'aide de la commande suivante : 
   ```
   sudo systemctl status nginx
   ```
   {:pre}
   La sortie devrait montrer que le service Nginx est **actif** et en cours d'exécution.
3. Vous devez ouvrir les ports **HTTP (80)** et **HTTPS (443)** pour recevoir le trafic (demandes). Pour ce faire, vous pouvez régler le pare-feu via [UFW](https://help.ubuntu.com/community/UFW) - `sudo ufw enable` et en activant le profil ‘Nginx Full’, qui inclut des règles pour les deux ports : 
   ```
   sudo ufw allow 'Nginx Full'
   ```
   {:pre}
4. Pour vérifier que Nginx fonctionne comme prévu, ouvrez `http://FLOATING_IP` dans le navigateur de votre choix et la page d'accueil Nginx par défaut devrait s'afficher.
5. Pour mettre à jour la page html avec les détails de la région et de la zone, exécutez la commande ci-dessous. 
   ```
   nano /var/www/html/index.nginx-debian.html
   ```
   {:pre}
   Ajoutez la région et la zone, par exemple _serveur en cours d’exécution dans la **zone 1 de la région 1**_ avec la balise `h1` en indiquant `Bienvenue dans nginx !` et enregistrez les modifications.
6. Redémarrez le serveur nginx pour refléter les modifications 
   ```
   sudo systemctl restart nginx
   ```
   {:pre}

**REPETEZ** les étapes 1 à 6 pour installer et configurer le serveur Web sur les instances VSI dans les sous-réseaux de toutes les zones et n'oubliez pas de mettre à jour le code HTML avec les informations de zone respectives.


## Répartition du trafic entre les zones à l'aide des équilibreurs de charge 
{: #distribute-traffic-with-load-balancers}

Dans cette section, vous allez créer deux équilibreurs de charge. Un dans chaque région pour répartir le trafic entre plusieurs instances de serveur sous des sous-réseaux respectifs au sein de différentes zones. 

### Configuration des équilibreurs de charge 

1. Accédez à **Equilibreurs de charge** et cliquez sur **Nouvel équilibreur de charge**.
2. Attribuez le nom unique **vpc-lb-region1**, sélectionnez **vpc-region1** en tant que cloud privé virtuel, puis le groupe de ressources dans lequel le VPC a été créé. Entrez **Public** et **region1** en tant que région.
3. Sélectionnez les adresses IP privées de la **zone 1** et de la **zone 2** de la **region 1**.
4. Créez un pool de back-end d'instances VSI qui agissent en tant qu'homologues égaux pour partager le trafic acheminé vers le pool. Définissez les paramètres avec les valeurs ci-dessous et cliquez sur **Créer**.
	- **Nom** : region1-pool
	- **Protocole** : HTTP
	- **Méthode** : Permutation circulaire
	- **Conservation de session** : Néant
	- **Chemin du contrôle de santé** : /
	- **Protocole de santé** : HTTP
	- **Intervalle (s)** : 15
	- **Délai d'attente (s)** : 2
	- **Nb max. de nouvelles tentatives** : 2
5. Cliquez sur **Connecter** pour ajouter des instances de serveur à region1-pool
   - Sélectionnez l'adresse IP privée de **vpc-region1-zone1-subnet**, sélectionnez l'instance que vous avez créée et définissez 80 comme port.
   - Cliquez sur **Ajouter** et sélectionnez cette fois l'adresse IP privée **vpc-region1-zone2-subnet**, sélectionnez l'instance et définissez 80 comme port. 
   - Cliquez sur **Connecter** pour terminer la création d'un pool de back-end.
6. Cliquez sur **Nouvel écouteur** pour créer un nouvel écouteur de front-end, qui est un processus qui vérifie les demandes de connexion.
   - **Protocole** : HTTP
   - **Port** : 80
   - **Pool de back-end** : region1-pool
   - **Nb max. de connexions** : laissez cette zone vide et cliquez sur **Créer**.
7. Cliquez sur **Créer un équilibreur de charge** pour mettre à disposition un équilibreur de charge.

### Test des équilibreurs de charge 

1. Patientez jusqu'à ce que le statut de l'équilibreur de charge passe à **Actif**. 
2. Ouvrez l'**Adresse** dans un navigateur Web.
3. Actualisez la page plusieurs fois et notez que l'équilibreur de charge touche différents serveurs à chaque actualisation. 
4. **Sauvegardez** l'adresse pour référence ultérieure.

Notez que les demandes ne sont pas chiffrées et ne prennent en charge que HTTP. Dans la section suivante, vous allez configurer un certificat SSL et activer HTTPS.

**REPETEZ** les étapes 1 à 7 ci-dessus dans la **région 2**.

## Sécurisation du trafic dans le VPC avec HTTPS 
{: #secure_https}

Avant d'ajouter un écouteur HTTPS, vous devez générer un certificat SSL, vérifier l'authenticité de votre domaine personnalisé, rechercher un emplacement pour stocker le certificat et mapper ce dernier sur le service d'infrastructure. 

### Mise à disposition d'un service CIS et configuration d'un domaine personnalisé 

Dans cette section, vous allez créer un service IBM Cloud Internet Services (CIS), configurer un domaine personnalisé en le faisant pointer sur les serveurs de noms CIS, puis configurer un équilibreur de charge global. 

1. Accédez à [Internet Services](https://{DomainName}/catalog/services/internet-services) dans le catalogue {{site.data.keyword.Bluemix_notm}}.
2. Définissez le nom du service, puis cliquez sur **Créer** pour créer une instance du service. Vous pouvez utiliser n’importe quel forfait pour ce tutoriel. 
3. Lorsque l'instance de service est mise à disposition, définissez votre nom de domaine en cliquant sur **Commençons !** et sur **Ajouter un domaine**. 
4. Cliquez sur **Etape suivante**. Lorsque les serveurs de noms sont attribués, configurez votre registraire ou votre fournisseur de nom de domaine pour utiliser les serveurs de noms répertoriés. 
5. Une fois que vous avez configuré votre registraire ou le fournisseur DNS, les modifications peuvent nécessiter jusqu'à 24 heures pour prendre effet. 

   Lorsque le statut du domaine sur la page Présentation passe de *En attente* à *Actif*, vous pouvez utiliser la commande `dig <YOUR_DOMAIN_NAME> ns` pour vérifier que les nouveaux serveurs de nom ont pris effet.
   {:tip}

Vous devez obtenir un certificat SSL pour le domaine et le sous-domaine que vous prévoyez d'utiliser avec l'équilibreur de charge global. En supposant un domaine mydomain.com, l'équilibreur de charge global peut être hébergé sur `lb.mydomain.com`. Le certificat doit être émis pour lb.mydomain.com.

Vous pouvez obtenir des certificats SSL gratuits auprès de [Let's Encrypt](https://letsencrypt.org/). Au cours du processus, vous devrez peut-être configurer un enregistrement DNS de type TXT dans l'interface DNS de Cloud Internet Services pour prouver que vous êtes le propriétaire du domaine.
{:tip}

Une fois que vous avez obtenu le certificat SSL et la clé privée de votre domaine, veillez à les convertir au format [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Pour convertir un certificat au format PEM : 
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
2. Pour convertir une clé privée au format PEM : 
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Importation du certificat et autorisation du service d'équilibreur de charge 

Vous pouvez gérer les certificats SSL via IBM Certificate Manager. 

1. Créez une instance [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) dans un emplacement pris en charge.
2. Dans le tableau de bord de service, utilisez **Importer le certificat** :
   * Définissez **Nom** sur le sous-domaine et le domaine personnalisés, tel que *lb.mydomain.com*.
   * Recherchez le **Fichier de certificat** au format PEM.
   * Recherchez le **Fichier de clé privée** au format PEM.
   * **Importez**.
3. Créez une autorisation qui donne à l'instance du service d'équilibreur de charge l'accès à l'instance du gestionnaire de certificats contenant le certificat SSL. Vous pouvez gérer ce type d'autorisation via [Identity and Access Authorizations](https://{DomainName}/iam#/authorizations).
  - Cliquez sur **Créer** et choisissez **Service d'infrastructure** en tant que service source
  - **Equilibreur de charge pour VPC** en tant que type de ressource
  - **Certificate Manager** en tant que service cible
  - Attribuez le rôle d'accès au service **Auteur**.
  - Pour créer un équilibreur de charge, vous devez accorder l'autorisation Toutes les instances de ressource sur l'instance de ressource source. L'instance de service cible peut être **Toutes les instances** ou votre instance de ressource de gestionnaire de certificats spécifique. 

### Création d'un écouteur HTTPS

Accédez à présent aux [Equilibreurs de charge](https://{DomainName}/vpc/network/loadBalancers)

1. Sélectionnez **vpc-lb-region1**
2. Sous **Ecouteurs de front-end**, cliquez sur **Nouvel écouteur**

   -  **Protocole** : HTTPS
   -  **Port** : 443
   -  **Pool de back-end** : POOL dans la même région
   -  Choisissez le certificat SSL pour **lb.YOUR-DOMAIN-NAME**

3. Cliquez sur **Créer** pour configurer un écouteur HTTPS

**REPETEZ** ces étapes dans l'équilibreur de charge de la **région 2**.

## Configuration d'un équilibreur de charge global 
{: #global-load-balancer}

Dans cette section, vous configurez un équilibreur de charge global (Global Load Balancer - GLB) en répartissant le trafic entrant sur les équilibreurs de charge locaux configurés dans les différentes régions {{site.data.keyword.Bluemix_notm}}.

### Répartition du trafic entre les régions à l'aide d'un équilibreur de charge global 
Ouvrez le service CIS que vous avez créé en accédant à la [Liste de ressources](https://{DomainName}/resources) sous Services. 

1. Accédez à **Equilibreurs de charge globaux** sous **Fiabilité** et cliquez sur **Créer un équilibreur de charge**.
2. Entrez **lb.YOUR-DOMAIN-NAME** en tant que nom d’hôte et 60 secondes pour TTL.
3. Cliquez sur **Ajouter un pool** pour définir un pool d'origines par défaut
   - **Nom** : lb-region1
   - **Contrôle de santé** : CREER UN CONTROLE DE SANTE 
     - **Type de moniteur** : HTTP
     - **Chemin ** : /
     - **Port** : 80
   - **Région du contrôle de santé** : Est de l'Amérique du Nord
   - **origins**
     - **name** : region1
     - **address** : ADRESSE DE L'EQUILIBREUR DE CHARGE LOCAL **REGION1**
     - **weight** : 1
     - Cliquez sur **Ajouter**

4. **AJOUTEZ** un autre **pool d’origines** pointant sur l’équilibreur de charge **region2** dans la région **Europe de l'Ouest**, puis cliquez sur **Provisionner 1 ressource** pour mettre à disposition votre équilibreur de charge global.

Patientez jusqu'à ce que l'**Etat du contrôle de santé** passe à **Sain**. Ouvrez le lien **lb.YOUR-DOMAIN-NAME** dans le navigateur de votre choix pour voir l’équilibreur de charge global en action.

### Test de reprise en ligne
A présent, vous devriez constater que vous contactez la plupart du temps les serveurs de la **région 1**, car le poids qui leur a été attribué est supérieur à celui des serveurs de la **région 2**. Voici un échec du contrôle de santé dans le pool d'origines de la **région 1**,

1. Accédez aux [Instances de serveur virtuel](https://{DomainName}/vpc/compute/vs).
2. Cliquez sur les **points de suspension (...)** en regard du ou des serveurs exécutés dans la **zone 1** de la **région 1** et cliquez sur **Arrêter**.
3. **REPETEZ** les mêmes étapes pour le ou les serveurs s'exécutant dans la **zone 2** de la **région 1**.
4. Revenez à l'équilibreur de charge global (GLB) sous le service CIS et patientez jusqu'à ce que l'état de santé passe à **Critique**.
5. Désormais, lorsque vous actualisez l'URL de votre domaine, vous devez toujours utiliser les serveurs de la **region 2**.

N'oubliez pas de **démarrer** les serveurs des zones 1 et 2 de la région 1
{:tip}

## Suppression de ressources 
{: #removeresources}

- Supprimez l'équilibreur de charge global, les pools d'origines et les contrôles de santé dans le service CIS 
- Supprimez les certificats dans le service du gestionnaire de certificats. 
- Supprimez les équilibreurs de charge, les instances VSI, les sous-réseaux et les VPC. 
- Sous [Liste de ressources](https://{DomainName}/resources), supprimez les services utilisés dans ce tutoriel.


## Contenu associé
{: #related}

* [Utilisation des équilibreurs de charge dans le VPC IBM Cloud](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc#--beta-using-load-balancers-in-ibm-cloud-vpc)
