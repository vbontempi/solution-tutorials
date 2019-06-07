---
copyright:
  years: 2019
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

# Applications Cloud Foundry Enterprise isolées
{: #isolated-cloud-foundry-enterprise-apps}

{{site.data.keyword.cfee_full_notm}} (CFEE) vous permet de créer plusieurs plateformes Cloud Foundry de niveau entreprise isolées à la demande. Vous obtenez ainsi une instance Cloud Foundry privée, déployée sur un cluster Kubernetes isolé. Contrairement au cloud public, vous disposez du contrôle total sur l'environnement : contrôle d'accès, capacité, version, utilisation des ressources et surveillance. {{site.data.keyword.cfee_full_notm}} vous permet de bénéficier de la rapidité et de l’innovation d’une plateforme en tant que service tout en conservant la propriété de l'infrastructure exploitée par l'informatique d'entreprise.

Un cas d'utilisation d'{{site.data.keyword.cfee_full_notm}} est une plateforme d'innovation appartenant à l'entreprise. En tant que développeur au sein d'une entreprise, vous pouvez créer des microservices ou faire migrer des applications existantes vers CFEE. Les microservices peuvent ensuite être publiés à l'intention d'autres développeurs, à l'aide du marché Cloud Foundry. Sur ce marché, d'autres développeurs de votre instance CFEE peuvent consommer des services au sein de l'application, comme cela est le cas aujourd'hui dans le cloud public. 

Ce tutoriel vous guide tout au long du processus de création et de configuration d'un environnement {{site.data.keyword.cfee_full_notm}}, de la configuration du contrôle d'accès et du déploiement des applications et des services. Vous examinerez également la relation entre CFEE et [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) en déployant un courtier de services personnalisé qui intègre des services personnalisés à CFEE. 

## Objectifs

{: #objectives}

- Comparer CFEE à Public Cloud Foundry 
- Déployer des applications et des services dans CFEE 
- Comprendre la relation entre Cloud Foundry et {{site.data.keyword.containershort_notm}} 
- Etudier la connectivité de base entre Cloud Foundry et {{site.data.keyword.containershort_notm}}  

## Services utilisés

{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture

{: #architecture}

![Architecture](./images/solution45-CFEE-apps/Architecture.png)

1. L'administrateur crée une instance CFEE et ajoute des utilisateurs avec un accès développeur. 
2. Le développeur envoie une application de démarrage Node.js à CFEE. 
3. L'application de démarrage Node.js utilise [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) pour stocker des données. 
4. Le développeur ajoute un nouveau service de "message de bienvenue". 
5. L'application de démarrage Node.js lie le nouveau service à partir d'un courtier de services personnalisé. 
6. L'application de démarrage Node.js affiche le message de bienvenue dans différentes langues à partir du service. 

## Conditions préalables

{: #prereq}

- [Interface CLI {{site.data.keyword.cloud_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [Interface CLI Cloud Foundry](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## Mise à disposition d'{{site.data.keyword.cfee_full_notm}}

{:provision_cfee}

Dans cette section, vous créez une instance d'{{site.data.keyword.cfee_full_notm}} déployée sur les noeuds worker Kubernetes à partir d'{{site.data.keyword.containershort_notm}}.

1. [Préparez votre compte {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare) pour assurer la création des ressources d'infrastructure requises. 
2. Dans le catalogue {{site.data.keyword.cloud_notm}}, créez une instance de service d'[{{site.data.keyword.cfee_full_notm}} ](https://{DomainName}/cfadmin/create).
3. Configurez CFEE en indiquant les éléments suivants : 
   - Sélectionnez le forfait **Standard**.
   - Entrez un **Nom** pour l'instance de service.
   - Sélectionnez un **Groupe de ressources** dans lequel l'environnement est créé. Vous devez disposer du droit d'accéder à au moins un groupe de ressources du compte pour pouvoir créer une instance CFEE. 
   - Sélectionnez une **Géographie** et un **Emplacement** où l'instance doit être déployée. Consultez la liste des [emplacements et centres de données de mise à disposition disponibles](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets). 
   - Sélectionnez une **Organisation** et un **Espace** Cloud Foundry publics dans lesquels **{{site.data.keyword.composeForPostgreSQL}}** doit être déployé. 
   - Sélectionnez le **Nombre de cellules** pour l'environnement Cloud Foundry. Une cellule exécute les applications Diego et Cloud Foundry. Au moins **2** cellules sont requises pour les applications à haute disponibilité. 
   - Sélectionnez le **Type de machine**, ce qui détermine la taille des cellules de Cloud Foundry (unité centrale et mémoire). 
4. Consultez la section **Infrastructure** pour afficher les propriétés du cluster Kubernetes prenant en charge CFEE. Le **nombre de noeuds worker** est égal au nombre de cellules plus 2. Deux des noeuds worker Kubernetes mis à disposition agissent en tant que plan de contrôle CFEE. Le cluster Kubernetes sur lequel l'environnement est déployé apparaît dans le tableau de bord {{site.data.keyword.cloud_notm}} [Clusters](https://{DomainName}/containers-kubernetes/clusters). 
5. Cliquez sur le bouton **Créer** pour démarrer le déploiement automatisé. 

Le déploiement automatisé prend environ 90 à 120 minutes. Une fois la création terminée, vous recevez un courrier électronique confirmant la mise à disposition de CFEE et des services de support. 

### Création d'organisations et d'espaces 

Une fois que vous avez créé {{site.data.keyword.cfee_full_notm}}, consultez la section [Création d'organisations et d'espaces](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs) pour plus d'informations sur la structuration de l'environnement entre organisations et espaces. Dans un environnement {{site.data.keyword.cfee_full_notm}}, les applications sont limitées à des espaces spécifiques. De même, un espace existe au sein d'une organisation. Les membres d'une organisation se partagent un plan d'établissement des quotas, des applications, des instances de service et des domaines personnalisés.

Remarque : le menu **Gérer > Compte > Organisations Cloud Foundry**, situé dans l'en-tête {{site.data.keyword.cloud_notm}} supérieur, est exclusivement destiné aux organisations {{site.data.keyword.cloud_notm}} publiques. Les organisations CFEE sont gérées dans la page **organisations** d'une instance CFEE. 

Appliquez les étapes ci-dessous pour créer une organisation et un espace CFEE. 

1. Dans le [tableau de bord Cloud Foundry](https://{DomainName}/dashboard/cloudfoundry/overview), sélectionnez **Environnements** sous **Enterprise**.
2. Sélectionnez votre instance CFEE, puis sélectionnez **Organisations**.
3. Cliquez sur le bouton **Créer une organisation**, indiquez `tutoriel` en tant que **Nom de l'organisation** et sélectionnez un **Plan d'établissement des quotas**. Terminez en cliquant sur **Ajouter**. 
4. Cliquez sur le nouveau `tutoriel` d'organisation, sélectionnez l'onglet **Espaces**, puis cliquez sur le bouton **Créer un espace**. 
5. Indiquez `dev` en tant que **Nom d'espace** et cliquez sur **Ajouter**.

### Ajout d'utilisateurs aux organisations et aux espaces 

Dans CFEE, vous pouvez attribuer des rôles contrôlant l'accès des utilisateurs, mais pour ce faire, vous devez inviter l'utilisateur sur votre compte {{site.data.keyword.cloud_notm}}, dans la page **Identité & Accès** sous **Gérer > Utilisateurs** dans l'en-tête {{site.data.keyword.cloud_notm}}.

Une fois que l'utilisateur a été invité, suivez les étapes ci-dessous pour l'ajouter à l'organisation `tutoriel` créée.  

1. Sélectionnez votre [instance CFEE](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments), puis sélectionnez à nouveau **Organisations**.
2. Dans la liste, sélectionnez l'organisation `tutoriel` créée.
3. Cliquez sur l'onglet **Membres** pour afficher et ajouter un nouvel utilisateur.
4. Cliquez sur le bouton **Ajouter des membres**, recherchez le nom d'utilisateur, sélectionnez les **Rôles de l'organisation** appropriés et cliquez sur **Ajouter**.

Vous trouverez plus d'informations sur l'ajout d'utilisateurs aux organisations et aux espaces CFEE [ici](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users). 

## Déploiement, configuration et exécution d'applications CFEE 

{:deploycfeeapps}

Dans cette section, vous allez déployer une application Node.js sur CFEE. Une fois l'application déployée, vous lui associez une base de données {{site.data.keyword.cloudant_short_notm}} et activez la persistance des audits et de la journalisation. La console Stratos est également ajoutée pour gérer l’application. 

### Déploiement de l'application dans CFEE 

1. A partir de votre terminal, clonez l’exemple d’application [get-started-node](https://github.com/IBM-Cloud/get-started-node).
   ```sh
   git clone https://github.com/IBM-Cloud/get-started-node
   ```
   {: codeblock}
2. Exécutez l'application localement pour vous assurer qu'elle se génère et démarre correctement. Confirmez en accédant à `http://localhost:3000/` dans votre navigateur.
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. Connectez-vous à {{site.data.keyword.cloud_notm}} et ciblez votre instance CFEE. Une invite interactive vous aide à sélectionner votre nouvelle instance CFEE. Dans la mesure où il n'existe qu'une seule organisation et qu'un seul espace CFEE, ils constituent la cible par défaut. Vous pouvez exécuter `ibmcloud target -o tutorial -s dev` si vous avez ajouté plusieurs organisations ou espaces.
   ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
   {: codeblock}
4. Transférez l'application **get-started-node** vers CFEE.
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. Le noeud final de votre application apparaît dans la sortie finale en regard de la propriété `routes`. Ouvrez l'URL dans votre navigateur pour confirmer que l'application est en cours d'exécution. 

### Création et association de la base de données Cloudant à l'application 

Pour associer des services {{site.data.keyword.cloud_notm}} à l'application **get-started-node**, vous devez d'abord créer un service dans votre compte {{site.data.keyword.cloud_notm}}. 

1. Créez un service [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant). Indiquez le **nom du service** `cfee-cloudant` et choisissez l'emplacement où l'instance CFEE a été créée. 
2. Ajoutez la nouvelle instance de service {{site.data.keyword.cloudant_short_notm}} à CFEE.
   1. Revenez à l'**Organisation** `tutoriel`. Cliquez sur l'onglet **Espaces** et sélectionnez l'espace `dev`.
   2. Sélectionnez l'onglet **Services** et cliquez sur le bouton **Ajout d'un service**.
   3. Entrez `cfee-cloudant` dans la zone de recherche et sélectionnez le résultat. Terminez en cliquant sur **Ajouter**. Le service est maintenant disponible pour les applications CFEE ; cependant, il se trouve toujours dans {{site.data.keyword.cloud_notm}} public. 
3. Dans le menu déroulant dynamique de l'instance de service affichée, sélectionnez **Lier à une application**.
4. Sélectionnez l'application **GetStartedNode** que vous avez transférée précédemment et cliquez sur **Reconstituer une application après la liaison**. Enfin, cliquez sur le bouton **Lier**. Patientez pendant la reconstitution de l'application. Vous pouvez vérifier la progression à l'aide de la commande `ibmcloud app show GetStartedNode`. 
5. Dans votre navigateur, accédez à l'application, ajoutez votre nom et appuyez sur `Entrée`. Le nom est ajouté à une base de données {{site.data.keyword.cloudant_short_notm}}.
6. Confirmez en sélectionnant l'instance `tutoriel` dans la liste de l'onglet **Services**. Cette opération ouvre la page des détails de l'instance de service dans {{site.data.keyword.cloud_notm}} public.
7. Cliquez sur **Lancer le tableau de bord Cloudant** et sélectionnez la base de données `mydb`. Un document JSON avec le nom devrait s'afficher. 

### Activation de l'audit et de la persistance de la journalisation

L'audit permet aux administrateurs CFEE de suivre les activités de Cloud Foundry telles que la connexion, la création d'organisations et d'espaces, l'appartenance d'utilisateurs et les attributions de rôles, les déploiements d'applications, les liaisons de services et la configuration de domaines. L'audit est pris en charge via l'intégration au service {{site.data.keyword.cloudaccesstrailshort}}.

Les journaux d'application Cloud Foundry peuvent être stockés en intégrant {{site.data.keyword.loganalysisshort_notm}}. L'instance de service {{site.data.keyword.loganalysisshort_notm}} sélectionnée par un administrateur CFEE est automatiquement configurée pour recevoir et conserver les événements de journalisation Cloud Foundry générés à partir de l'instance CFEE. 

Pour activer l'audit et la persistance de la journalisation CFEE, appliquez les [étapes décrites ici](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging). 

### Installation de la console Stratos pour gérer l'application 

La console Stratos est un outil Web open source qui fonctionne avec Cloud Foundry. L'application de console Stratos peut éventuellement être installée et utilisée dans un environnement CFEE spécifique afin de gérer ses organisations, ses espaces et ses applications. 

Pour installer l'application de console Stratos :

1. Ouvrez l'instance CFEE dans laquelle vous voulez installer la console Stratos.
2. Dans la page **Vue d'ensemble**, cliquez sur **Installer la console Stratos**. Le bouton ne s'affiche que pour les utilisateurs disposant de droits d'administrateur ou d'éditeur. 
3. Dans la boîte de dialogue Installer la console Stratos, sélectionnez une option d'installation. Vous pouvez installer l'application de console Stratos sur le plan de contrôle CFEE ou dans l'une des cellules. Sélectionnez une version de la console Stratos ainsi que le nombre d'instances de l'application à installer. Si vous installez l'application de console Stratos dans une cellule, vous êtes invité à indiquer l'organisation et l'espace où déployer l'application.
4. Cliquez sur **Installer**.

L'installation de l'application prend environ 5 minutes. Une fois l'installation terminée, un bouton **Console Stratos** s'affiche dans la page de présentation à la place du bouton **Installer la console Stratos**. Vous trouverez plus d'informations sur la console Stratos [ici](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app).

## Relation entre CFEE et Kubernetes

En tant que plateforme d'application, CFEE fonctionne sur une forme d'infrastructure virtuelle dédiée ou partagée. Pendant de nombreuses années, les développeurs ont peu réfléchi à la plateforme Cloud Foundry sous-jacente, car IBM la gérait pour eux. Avec CFEE, vous êtes non seulement un développeur qui écrit des applications Cloud Foundry, mais également un opérateur de la plateforme Cloud Foundry. En effet, CFEE est déployé sur un cluster Kubernetes que vous contrôlez. 

Bien que Kubernetes soit un domaine nouveau pour les développeurs de Cloud Foundry, de nombreux concepts sont communs. A l'instar de Cloud Foundry, Kubernetes isole les applications dans des conteneurs, qui s'exécutent dans une structure Kubernetes appelée pod. Semblables aux instances d’application, les pods peuvent avoir plusieurs copies (appelées ensembles de répliques) dotées de l’équilibrage de charge d’application fourni par Kubernetes. L'application Cloud Foundry `GetStartedNode` que vous avez déployée précédemment s'exécute dans le pod `diego-cell-0`. Pour prendre en charge la haute disponibilité, un autre pod `diego-cell-1` est exécuté sur un noeud worker Kubernetes distinct. Dans la mesure où ces applications Cloud Foundry s'exécutent "à l'intérieur" de Kubernetes, vous pouvez également communiquer avec d'autres microservices Kubernetes à l'aide de la mise en réseau basée sur Kubernetes. Les sections suivantes aident à illustrer plus en détail les relations entre CFEE et Kubernetes. 

## Déploiement d'un courtier de services Kubernetes 

Dans cette section, vous allez déployer un microservice sur Kubernetes qui agit en tant que courtier de services pour CFEE. Les [courtiers de services](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md) fournissent des détails sur les services disponibles, ainsi que sur la liaison et la mise à disposition de votre application Cloud Foundry. Vous avez utilisé un courtier de services {{site.data.keyword.cloud_notm}} intégré pour ajouter le service {{site.data.keyword.cloudant_short_notm}}. Vous allez maintenant déployer et utiliser un courtier de services personnalisé. Vous allez ensuite modifier l'application `GetStartedNode` pour utiliser le courtier de services, qui renverra un message de "bienvenue" en plusieurs langues. 

1. Sur votre terminal, clonez les projets qui fournissent les fichiers de déploiement Kubernetes et l'implémentation de courtier de services. 

   ```sh
   git clone https://github.com/IBM-Cloud/cfee-service-broker-kubernetes.git
   ```
   {: codeblock}

   ```sh
   cd cfee-service-broker-kubernetes
   ```
   {: codeblock}

   ```sh
   git clone https://github.com/IBM/sample-resource-service-brokers.git
   ```
   {: codeblock}

2. Générez et stockez l'image Docker contenant le courtier de services sur {{site.data.keyword.registryshort_notm}}. Utilisez la commande `ibmcloud cr info` pour extraire manuellement l’URL du registre ou récupérez l'URL automatiquement à l’aide de la commande `export REGISTRY` ci-dessous. La commande `cr namespace-add` crée un espace de nom pour stocker l'image Docker.

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   Créez un espace de nom appelé `cfee-tutorial`.

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   Editez le fichier `./cfee-service-broker-kubernetes/deployment.yml`. Mettez à jour l'attribut `image` pour refléter l'URL du registre des conteneurs.
3. Déployez l'image des conteneurs sur le cluster Kubernetes de CFEE. Le cluster de votre environnement CFEE existe dans le groupe de ressources `default`, qui devrait être ciblé si cela n'est pas déjà le cas. En utilisant le nom de votre cluster, exportez la variable KUBECONFIG à l'aide de la commande `cluster-config`. Créez ensuite le déploiement. 
   ```sh
   ibmcloud target -g default
   ```
   {: codeblock}

   ```sh
   ibmcloud ks clusters
   ```
   {: codeblock}

   ```sh
   $(ibmcloud ks cluster-config <your-cfee-cluster-name> --export)
   ```
   {: codeblock}

   ```sh
   kubectl apply -f deployment.yml
   ```
   {: codeblock}
4. Vérifiez que le STATUT des pods est `En cours d'exécution`. Il faudra peut-être quelques instants à Kubernetes pour extraire l’image et démarrer les conteneurs. Notez que vous disposez de deux pods car `deployment.yml` a demandé 2 `répliques`. 
   ```sh
   kubectl get pods
   ```
   {: codeblock}

## Vérification du déploiement du courtier de services 

Une fois que vous avez déployé le courtier de services, vérifiez qu'il fonctionne correctement. Effectuez cette opération en plusieurs étapes : utilisez d'abord le tableau de bord Kubernetes, accédez ensuite au courtier à partir d'une application Cloud Foundry et enfin liez un service à partir du courtier. 

### Affichage des pods à partir du tableau de bord Kubernetes 

Cette section vous permet de confirmer que les artefacts Kubernetes sont configurés à l'aide du tableau de bord {{site.data.keyword.containershort_notm}}. 

1. Dans la page [Clusters Kubernetes](https://{DomainName}/containers-kubernetes/clusters), accédez à votre cluster CFEE en cliquant sur la ligne commençant par le nom de votre service CFEE et se terminant par **-cluster**.
2. Ouvrez le **tableau de bord Kubernetes** en cliquant sur le bouton correspondant.
3. Cliquez sur le lien **Services** dans le menu de gauche et sélectionnez **tutorial-broker-service**. Ce service a été déployé lorsque vous avez exécuté `kubectl apply` aux étapes précédentes. 
4. Dans le tableau de bord obtenu, notez les éléments suivants : 
   - Le service a reçu une adresse IP de superposition (172.x.x.x) qui ne peut être résolue qu’au sein du cluster Kubernetes. 
   - Le service comporte deux noeuds finaux, qui correspondent aux deux pods sur lesquels les conteneurs du courtier de services sont en cours d'exécution. 

Lorsque vous avez confirmé que le service est disponible et qu'il représente les pods du courtier de services, vous pouvez vérifier que le courtier répond en fournissant des informations sur les services disponibles. 

Vous pouvez afficher les artefacts liés à Cloud Foundry à partir du tableau de bord Kubernetes. Pour les visualiser, cliquez sur **Espaces de nom** pour afficher tous les espaces de nom comportant des artefacts, y compris l'espace de nom `cf` de Cloud Foundry.
{: tip}

### Accès au courtier à partir d'un conteneur Cloud Foundry 

Pour démontrer la communication de Cloud Foundry à Kubernetes, connectez-vous au courtier de services directement à partir d'une application Cloud Foundry. 

1. Sur votre terminal, vérifiez que vous êtes toujours connecté à l'organisation `tutoriel` CFEE et à votre espace `dev` à l'aide de la commande `ibmcloud target`. Si nécessaire, recentrez CFEE. 
   ```sh
   ibmcloud target --cf
   ```
   {: codeblock}
2. Par défaut, SSH est désactivé dans les espaces. A la différence du cloud public, vous devez activer SSH dans votre espace `dev` CFEE. 
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. Utilisez la commande `kubectl` pour afficher le ClusterIP que vous avez visualisé dans le tableau de bord Kubenetes. Exécutez ensuite SSH dans l'application `GetStartedNode` et récupérez les données du courtier de services à l'aide de l'adresse IP. Notez que la dernière commande peut entraîner une erreur qui sera résolue à l’étape suivante. 
   ```sh
   kubectl get service tutorial-broker-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf ssh GetStartedNode
   ```
   {: codeblock}

   ```sh
   export CLUSTER_IP=<CLUSTER-IP address>
   ```
   {: codeblock}

   ```sh
   wget --user TestServiceBrokerUser --password TestServiceBrokerPassword -O- http://$CLUSTER_IP/v2/catalog
   ```
   {: codeblock}
4. Il est probable que vous ayez reçu une erreur de **connexion refusée**. Elle est due aux [groupes de sécurité des applications](https://docs.cloudfoundry.org/concepts/asg.html) par défaut de CFEE. Un groupe de sécurité d'application (Application Security Group - ASG) définit la plage d'adresses IP autorisées pour le trafic de sortie à partir d'un conteneur Cloud Foundry. Dans la mesure où l'application `GetStartedNode` existe en dehors de la plage par défaut, l'erreur se produit. Quittez la session SSH et téléchargez le groupe ASG `public_networks`.
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. Editez le fichier `public_networks.json` et observez que l'adresse ClusterIP utilisée n'est pas conforme aux règles existantes. Par exemple, la plage `172.32.0.0-192.167.255.255` n'inclut probablement pas le ClusterIP et doit être mise à jour. Si, par exemple, votre adresse ClusterIP est similaire à `172.21.107.63`, vous devez éditer le fichier pour obtenir une adresse similaire à `172.0.0.0-255.255.255.255`.
6. Ajustez la règle de `destination` ASG pour inclure l'adresse IP du service Kubernetes. Tronquez le fichier pour n'inclure que les données JSON, qui commencent et finissent par des crochets. Téléchargez ensuite le nouveau groupe ASG. 
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. Répétez l'étape 3 qui devrait maintenant générer des données de catalogue factices. Terminez en quittant la session SSH. 
   ```sh
   exit
   ```
   {: codeblock}

### Enregistrement du courtier de services auprès de CFEE 

Pour permettre aux développeurs de mettre à disposition et de lier des services à partir du courtier de services, vous devez enregistrer ce dernier auprès de CFEE. Vous avez utilisé une adresse IP avec le courtier. Or, cela pose un problème. Si le courtier de services redémarre, il reçoit une nouvelle adresse IP, ce qui nécessite la mise à jour de CFEE. Pour résoudre ce problème, vous allez utiliser une autre fonctionnalité de Kubernetes appelée KubeDNS, qui fournit un nom de domaine pleinement qualifié (FQDN) au courtier de services. 

1. Enregistrez le courtier de services auprès de CFEE en utilisant le nom de domaine complet du service `tutorial-service-broker`. Là encore, ce chemin est interne à votre cluster Kubernetes CFEE. 
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. Ajoutez ensuite les services offerts par le courtier. Etant donné que l'exemple de courtier ne comporte qu'un seul service fictif, une seule commande est nécessaire. 
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. Dans votre navigateur, accédez à votre instance CFEE à partir de la page [**Environnements**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments), accédez à `organisations -> espaces` et sélectionnez votre espace `dev`.
4. Sélectionnez l'onglet **Services** et le bouton **Créer un service**.
5. Dans la zone de recherche, recherchez **Test**. Le service fictif **Test Node Resource Service Broker Display Name** du courtier s'affiche. 
6. Sélectionnez ce service, cliquez sur le bouton **Créer** et indiquez le nom `welcome-service` pour créer une instance de service. Nous verrons clairement dans la section suivante pourquoi ce service est nommé welcome-service. Liez ensuite le service à l'application `GetStartedNode` à l'aide de l'option **Lier à une application** du menu déroulant dynamique. 
7. Pour afficher le service lié, exécutez la commande `cf env`. L'application `GetStartedNode` peut désormais exploiter les données de l'objet `credentials` de la même manière qu'elle utilise les données `cloudantNoSQLDB`. 
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## Ajout du service à votre application 

Jusqu'à présent, vous avez déployé un courtier de services et non un véritable service. Si le courtier de services fournit des données de liaison à l'application `GetStartedNode`, aucune fonctionnalité réelle du service même n'a été ajoutée. Pour corriger cela, un service fictif a été créé. Il fournit un message de "bienvenue" à `GetStartedNode` en différentes langues. Vous allez déployer ce service sur CFEE et mettre à jour le courtier et l'application pour l'utiliser. 

1. Transférez l'implémentation `welcome-service` à CFEE pour permettre à d'autres équipes de développement de l'exploiter en tant qu'API. 
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. Accédez au service en utilisant votre navigateur. La `route` vers le service est indiquée dans la sortie de la commande `push`. L'URL obtenue est similaire à `https://welcome.<your-cfee-cluster-domain>`. Actualisez la page pour afficher les autres langues. 
3. Revenez au dossier `sample-resource-service-brokers` et éditez l'exemple d'implémentation Node.js, `sample-resource-service-brokers/node-resource-service-broker/testresourceservicebroker.js` en ajoutant l'URL du cluster après la ligne 854. 

   ```javascript
   // TODO - Do your actual work here
    
   var generatedUserid   = uuid();
   var generatedPassword = uuid();
    
   result = {
     credentials: {
       userid   : generatedUserid,
       password : generatedPassword,
       url: 'https://welcome.<your-cfee-cluster-domain>'
     }
   };
   ```
   {: codeblock}
4. Générez et déployez le courtier de services mis à jour. Cette opération garantit que la propriété URL est fournie aux applications qui lient le service. 
   ```sh
   cd ..
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   ```sh
   kubectl patch deployment tutorial-servicebroker-deployment -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"version\":\"2\"}}}}}"
   ```
   {: codeblock}
5. Dans le terminal, revenez au dossier de l'application `get-started-node`. 
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. Editez le fichier `get-started-node/server.js` dans `get-stared-node` pour inclure le middleware suivant. Insérez-le juste avant l'appel `app.listen()`.
   ```javascript
   // Use the welcome service
   const request = require('request');
   const testService = appEnv.services['testnoderesourceservicebrokername'];

   if (testService) {
     const { credentials: { url} } = testService[0];
     app.get("/api/welcome", (req, res) => request(url, (e, r, b) => res.send(b)));
   } else {
     app.get("/api/welcome", (req, res) => res.send('Welcome'));
   }
   ```
   {: codeblock}
7. Restructurez la page `get-started-node/views/index.html`. Remplacez `data-i18n="welcome"` par `id="welcome"` dans la balise `h1`. Ajoutez ensuite un appel au service à la fin de la fonction `$(document).ready()`.
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. Mettez à jour l'application `GetStartedNode`. Incluez la dépendance inter-package `request` qui a été ajoutée à `server.js`, reliez `welcome-service` pour récupérer la nouvelle propriété `url` et, enfin, envoyez le nouveau code de l'application.
   ```sh
   npm i request -S
   ```
   {: codeblock}

   ```sh
   ibmcloud cf unbind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf bind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}

Vous avez terminé. A présent, explorez l'application et actualisez la page plusieurs fois pour visualiser le message de bienvenue dans différentes langues. 

![Réponse du courtier de services](./images/solution45-CFEE-apps/service-broker.png)

Si seul le mot **Welcome** apparaît et aucune autre langue ne s'affiche, exécutez la commande `ibmcloud cf env GetStartedNode` et confirmez que l'`url` vers le service est présente. Si tel n'est pas le cas, reconstituez les étapes, mettez à jour et redéployez le courtier. Si la valeur `url` est présente, confirmez qu'elle renvoie un message dans votre navigateur. Ensuite, exécutez à nouveau les commandes `unbind-service` et `bind-service`, suivies de `ibmcloud cf restart GetStartedNode`.
{: tip}

Bien que le service de bienvenue utilise Cloud Foundry pour sa mise en oeuvre, vous pouvez tout aussi bien utiliser Kubernetes. La principale différence est que l'URL du service serait probablement `welcome-service.default.svc.cluster.local`. L'utilisation de l'URL KubeDNS présente l'avantage supplémentaire de conserver à l'intérieur du cluster Kubernetes le trafic réseau vers les services. 

Avec {{site.data.keyword.cfee_full_notm}}, ces autres approches sont désormais possibles.

## Déploiement du présent tutoriel de solution à l'aide d'une chaîne d'outils   

Vous pouvez éventuellement déployer le tutoriel complet de la solution à l'aide d'une chaîne d'outils. Suivez les [instructions de la chaîne d'outils](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes) pour déployer tout ce qui précède à l'aide d'une chaîne d'outils.  

Remarque : il existe certains prérequis lors de l'utilisation d'une chaîne d'outils ; vous devez avoir créé une instance CFEE, une organisation CFEE et un espace CFEE. Vous trouverez des instructions détaillées dans le fichier readme des [instructions de la chaîne d'outils](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes).  

## Contenu associé
{:related}

* [Déploiement d'applications dans des clusters Kubernetes](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Composants et architecture Cloud Foundry Diego](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [Courtier de services CFEE sur Kubernetes avec une chaîne d’outils](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

