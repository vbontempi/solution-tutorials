---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}


# Application Web nouvelle génération utilisant la pile MEAN
{: #mean-stack}

Ce tutoriel vous guide dans la création d’une application Web à l’aide de la pile MEAN bien connue. Cette pile est constituée d'une base de données **M**ongo, d’une infrastructure **E**xpress web, d’une infrastructure front-end **A**ngular et d’une application d’exécution Node.js. Vous allez apprendre à exécuter un démarreur MEAN localement, à créer et à utiliser une base de données gérée en tant que service (DBasS), à déployer l'application sur {{site.data.keyword.cloud_notm}} et à surveiller l'application.   

## Objectifs

{: #objectives}

- Créer et exécuter une application de démarrage Node.js localement. 
- Créer une base de données gérée en tant que service (DBasS). 
- Déployer l'application Node.js sur le cloud. 
- Dimensionner les ressources MongoDB. 
- Apprendre à surveiller les performances des applications. 

## Services utilisés

{: #products}

Ce tutoriel utilise les services {{site.data.keyword.Bluemix_notm}} suivants : 

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**Attention :** ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture

{:#architecture}

<p style="text-align: center;">

![Diagramme d'architecture](images/solution7/Architecture.png)</p>

1. L'utilisateur accède à l'application à l'aide d'un navigateur web.
2. L'application Node.js accède à la base de données {{site.data.keyword.composeForMongoDB}} pour extraire des données. 

## Avant de commencer

{: #prereqs}

1. [Installez git](https://git-scm.com/)
2. [Installez l'interface CLI {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)


Et pour développer et exécuter l'application localement : 
1. [Installez Node.js et NPM](https://nodejs.org/)
2. [Installez et exécutez MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/)

## Exécution locale de l'application MEAN

{: #runapplocally}

Dans cette section, vous exécutez une base de données MongoDB locale, clonez un exemple de code MEAN et exécutez l'application localement pour utiliser la base de données MongoDB locale. 

{: shortdesc}

1. Suivez les instructions [ici](https://docs.mongodb.com/manual/administration/install-community/) pour installer et exécuter la base de données MongoDB localement. Une fois l’installation terminée, utilisez la commande ci-dessous pour vérifier que le serveur **mongod** est en cours d’exécution. Confirmez que votre base de données est en cours d'exécution en lançant la commande suivante. 
  ```sh
  mongo
  ```
  {: codeblock}

2. Clonez le code de démarrage MEAN.

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. Installez les packages requis. 

  ```sh
  npm install
  ```
  {: codeblock}

4. Copiez le fichier .env.example vers .env. Modifiez les informations nécessaires, ajoutez au minimum votre propre SESSION_SECRET. 

5. Exécutez le noeud server.js pour démarrer votre application. 
  ```sh
  node server.js
  ```
  {: codeblock}

6. Accédez à votre application, créez un utilisateur et connectez-vous. 

## Création d'une instance de la base de données MongoDB dans le cloud 

{: #createdatabase}

Dans cette section, vous allez créer une base de données {{site.data.keyword.composeForMongoDB}} dans le cloud. {{site.data.keyword.composeForMongoDB}} est une base de données en tant que service qui est généralement plus facile à configurer et permet des sauvegardes et une mise à l'échelle intégrées. Vous pouvez trouver de nombreux types de bases de données différents dans le [catalogue d'IBM Cloud](https://{DomainName}/catalog/?category=data). Pour créer {{site.data.keyword.composeForMongoDB}}, effectuez les étapes ci-dessous. 

{: shortdesc}

1. Connectez-vous à votre compte {{site.data.keyword.cloud_notm}} via la ligne de commande et ciblez votre compte {{site.data.keyword.cloud_notm}}. 

  ```sh
  ibmcloud login
  ibmcloud target --cf
  ```
  {: codeblock}

  Vous pouvez trouver plus de commandes CLI [ici.](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

2. Créez une instance de {{site.data.keyword.composeForMongoDB}}. Cette opération peut également être effectuée à l'aide de l'[interface utilisateur de la console](https://{DomainName}/catalog/services/compose-for-mongodb). Le nom du service doit être nommé **mean-starter-mongodb** car l'application est configurée pour rechercher ce service sous ce nom. 

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## Déploiement de l'application sur le cloud 

{: #deployapp}

Dans cette section, vous déployez l'application node.js sur l'instance {{site.data.keyword.cloud_notm}} qui utilisait la base de données MongoDB gérée. Le code source contient un fichier [**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml) qui a été configuré pour utiliser le service "mongodb" créé précédemment. L'application utilise la variable d'environnement VCAP_SERVICES pour accéder aux données d'identification de la base de données Compose MongoDB. Ces opérations sont visibles dans le fichier [server.js](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js). 

{: shortdesc}

1. Transférez le code dans le cloud.

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. Une fois que le code a été envoyé, l'application devrait apparaître dans votre navigateur.  Un nom d'hôte aléatoire a été généré et peut se présenter comme suit : `https://mean-random-name.mybluemix.net`. Vous pouvez obtenir l'URL de votre application à partir du tableau de bord de la console ou de la ligne de commande. ![Live App](images/solution7/live-app.png)


## Mise à l'échelle des ressources de la base de données MongoDB 
{: #scaledatabase}

Si votre service nécessite davantage d'espace de stockage, ou si vous souhaitez limiter la quantité de stockage allouée au service, vous pouvez effectuer une mise à l'échelle des ressources.

{: shortdesc}

1. A l'aide du **tableau de bord** de la console, accédez à la section des **connexions** et cliquez sur la base de données de **l'instance MongoDB**.
2. Dans le panneau des **détails du déploiement**, cliquez sur l'option de **mise à l'échelle des ressources**.
  ![](images/solution7/mongodb-scale-show.png)
3. Ajustez le **curseur** pour augmenter ou réduire le stockage alloué à votre service de base de données {{site.data.keyword.composeForMongoDB}}.
4. Cliquez sur **Scale Deployment** pour lancer le processus de mise à l'échelle et revenir dans la vue d'ensemble du tableau de bord. Un message de démarrage de la mise à l'échelle apparaît en haut de la page pour vous informer que la mise à l'échelle est en cours.
  ![](images/solution7/scaling-in-progress.png)


## Surveillance des performances de l'application
{: #monitorapplication}

Pour vérifier la santé de votre application, vous pouvez utiliser le service intégré Availability Monitoring. Ce service est automatiquement associé à vos applications dans le cloud. Le service Availability Monitoring exécute des tests synthétiques à partir de sites répartis dans le monde entier, 24 heures sur 24, pour détecter et résoudre de manière proactive les problèmes de performances avant qu'ils n'affectent vos visiteurs. Effectuez les étapes ci-dessous pour accéder au tableau de bord de surveillance. 

{: shortdesc}

1. A l'aide du tableau de bord de la console, sous votre application, sélectionnez l'onglet **Surveillance**.
2. Cliquez sur **Afficher tous les tests** pour consulter les tests.
   ![](images/solution7/alert_frequency.png)


## Contenu associé

{: #related}

- Configuration du contrôle de source et de [Continuous Delivery](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops). 
- Sécurisation d'une application Web dans [plusieurs emplacements](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp). 
- Création, sécurisation et gestion des [API REST](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis).
