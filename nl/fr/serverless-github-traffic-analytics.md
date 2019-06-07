---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Combinaison d'une application sans serveur et de Cloud Foundry pour l'extraction et l'analyse de données 
{: #serverless-github-traffic-analytics}
Dans ce tutoriel, vous allez créer une application pour collecter automatiquement les statistiques de trafic GitHub pour les référentiels et fournir la base de l'analyse du trafic. GitHub fournit uniquement un accès aux données de trafic des 14 derniers jours. Si vous souhaitez analyser des statistiques sur une période plus longue, vous devez télécharger et stocker ces données vous-même. Dans ce tutoriel, vous allez déployer une action sans serveur pour extraire les données de trafic et les stocker dans une base de données SQL. De plus, une application Cloud Foundry est utilisée pour gérer les référentiels et fournir un accès aux statistiques pour l'analyse des données. L'application et l'action sans serveur décrites dans ce tutoriel implémentent une solution prenant en charge le mode à service partagé et l'ensemble de fonctionnalités initial prenant en charge le mode à service exclusif. 

![](images/solution24-github-traffic-analytics/Architecture.png)

## Objectifs

* Déployer une application de base de données Python avec prise en charge du service partagé et accès sécurisé 
* Intégrer l'ID d'application en tant que fournisseur d'authentification basé sur OpenID Connect 
* Configurer une collecte automatisée, sans serveur, de statistiques de trafic GitHub 
* Intégrer {{site.data.keyword.dynamdashbemb_short}} pour l'analyse de trafic graphique

## Produits
Ce tutoriel utilise les produits suivants : 
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Avant de commencer
{: #prereqs}

Pour effectuer ce tutoriel, vous devez disposer de la dernière version de [l'interface CLI d'IBM Cloud](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) et le [plug-in {{site.data.keyword.openwhisk_short}} doit être installé](/docs/cli?topic=cloud-cli-install-ibmcloud-cli).

## Configuration du service et de l'environnement (shell) 
Dans cette section, vous configurez les services nécessaires et préparez l'environnement. Vous pouvez effectuer ces opérations à partir de l'environnement shell. 

1. Clonez le [Référentiel GitHub](https://github.com/IBM-Cloud/github-traffic-stats) et accédez au répertoire cloné et à son sous-répertoire de **back-end** :
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. Utilisez `ibmcloud login` pour vous connecter de manière interactive à {{site.data.keyword.Bluemix_short}}. Vous pouvez confirmer les détails en exécutant la commande `ibmcloud target`. Vous devez avoir défini une organisation et un espace. 

3. Créez une instance {{site.data.keyword.dashdbshort}} avec le forfait **Entrée** et nommez-la **ghstatsDB** :
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. Pour accéder ultérieurement au service de base de données à partir de {{site.data.keyword.openwhisk_short}}, vous devez disposer d'une autorisation. Vous créez alors des données d'identification du service et les étiquetez **ghstatskey** :   
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. Créez une instance du service {{site.data.keyword.appid_short}}. Utilisez **ghstatsAppID** comme nom et le forfait **Tranche graduée**.
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
Créez ensuite un alias de cette nouvelle instance de service dans l'espace Cloud Foundry. Remplacez **YOURSPACE** par l’espace que vous déployez.
   ```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. Créez une instance du service {{site.data.keyword.dynamdashbemb_short}} à l'aide du forfait **lite**. 
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
   Là encore, créez un alias de cette nouvelle instance de service et remplacez **YOURSPACE**.
   ```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. Dans le répertoire **back-end**, transférez l'application vers IBM Cloud. La commande utilise une route aléatoire pour votre application. 
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
Attendez la fin du déploiement.Les fichiers d'application sont téléchargés, l'environnement d'exécution créé et les services liés à l'application. Les informations de service proviennent du fichier `manifest.yml`. Vous devez mettre à jour ce fichier si vous avez utilisé d'autres noms de service. Une fois le processus terminé, l'URI de l'application s'affiche. 

   La commande ci-dessus utilise une route aléatoire, mais unique pour l'application. Si vous souhaitez en choisir une vous-même, ajoutez-la en tant que paramètre supplémentaire à la commande, par exemple, `ibmcloud cf push your-app-name`. Vous pouvez également éditer le fichier `manifest.yml`, modifier le **nom** et remplacer la valeur de **random-route**, **true**, par **false**.
   {:tip}

## Configuration d'App ID et de GitHub (navigateur)
Les étapes suivantes sont toutes effectuées à l'aide de votre navigateur Internet. Configurez, tout d'abord, {{site.data.keyword.appid_short}} pour utiliser Cloud Directory et l'application Python. Créez ensuite un jeton d’accès GitHub. Il est nécessaire pour que la fonction déployée puisse extraire les données de trafic. 

1. Dans la [Liste de ressources {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), ouvrez la présentation de vos services. Recherchez l'instance du service {{site.data.keyword.appid_short}} créée dans la section **Services**. Cliquez sur son entrée pour ouvrir les détails. 
2. Dans le tableau de bord des services, cliquez sur **Gérer** sous **Fournisseurs d'identité** dans le menu de gauche. Vous affichez ainsi la liste des fournisseurs d'identité disponibles, tels que Facebook, Google, SAML 2.0 Federation et Cloud Directory. Basculez Cloud Directory sur **Activé** et tous les autres fournisseurs sur **Désactivé**.
   
   Vous pouvez configurer [l'Authentification multifacteur (Multi-Factor Authentication - MFA)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) et des règles de mot de passe avancées. Ce sujet n'est pas abordé dans le cadre de ce tutoriel.
{:tip}

3. Au bas de cette page se trouve la liste des URL de redirection. Entrez l'**url** de votre application + /redirect_uri. Par exemple, `https://github-traffic-stats-random-word.mybluemix.net/redirect_uri`.

   Pour tester l'application localement, l'URL de redirection est `http://0.0.0.0:5000/redirect_uri`. Vous pouvez configurer plusieurs URL de
redirection.
{:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. Dans le menu de gauche, cliquez sur **Utilisateurs**. La liste des utilisateurs de Cloud Directory apparaît. Cliquez sur le bouton **Ajout d'un utilisateur** pour vous ajouter en tant que premier utilisateur. Vous avez terminé la configuration du service {{site.data.keyword.appid_short}} .
5. Dans le navigateur, consultez [Github.com](https://github.com/settings/tokens) et accédez à **Settings -> Developer settings -> Personal access tokens**. Cliquez sur le bouton **Generate new token**. Entrez **GHStats Tutorial** pour **Token description**. Activez ensuite **public_repo** sous la catégorie **repo** et **read:org** sous **admin:org**. A présent, au bas de cette page, cliquez sur **Generate token**. Le nouveau jeton d'accès s'affiche sur la page suivante. Vous en aurez besoin lors de la configuration de l'application à venir.
   ![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## Configuration et test de l'application Python
Après la préparation, vous configurez et testez l'application. L'application est écrite en langage Python à l'aide de l'infrastructure [Flask](http://flask.pocoo.org/) bien connue. Les référentiels peuvent être ajoutés et supprimés de la collecte de statistiques. Les données de trafic sont accessibles sous forme de tableau. 

1. Dans un navigateur, ouvrez l'URI de l'application déployée. La page de bienvenue devrait apparaître.
   ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. Dans le navigateur, ajoutez `/admin/initialize-app` à l'URI et accédez à la page. Cette opération permet d'initialiser l'application et ses données. Cliquez sur le bouton **Start initialization**. Vous accédez alors à une page de configuration protégée par mot de passe. L'adresse e-mail avec laquelle vous vous connectez est considérée comme l'identification de l'administrateur système. Utilisez l'adresse e-mail et le mot de passe que vous avez configurés précédemment. 

3. Dans la page de configuration, entrez un nom (il est utilisé pour les messages d'accueil), votre nom d'utilisateur GitHub et le jeton d'accès que vous avez généré auparavant. Cliquez sur **Initialize**. Vous créez ainsi les tables de la base de données et insérez des valeurs de configuration. Enfin, vous créez des enregistrements de base de données pour l'administrateur système et un titulaire.
   ![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. Après quoi, la liste des référentiels gérés apparaît. Vous pouvez à présent ajouter des référentiels en fournissant le nom du compte ou de l'organisation GitHub et le nom du référentiel. Après avoir saisi les données, cliquez sur **Ajouter un référentiel**. Le référentiel, ainsi qu'un identificateur nouvellement attribué, doivent apparaître dans la table. Vous pouvez supprimer les référentiels du système en entrant leur ID et en cliquant sur **Supprimer un référentiel**.
![](images/solution24-github-traffic-analytics/RepositoryList.png) 

## Déploiement de Cloud Function et d'un déclencheur
Une fois l'application de gestion en place, déployez une action, un déclencheur et une règle pour connecter les deux pour {{site.data.keyword.openwhisk_short}}. Ces objets sont utilisés pour collecter automatiquement les données de trafic GitHub selon la planification spécifiée. Cette action effectue la connexion à la base de données, une itération sur tous les titulaires et leurs référentiels et obtient la vue et les données de clonage pour chaque référentiel. Ces statistiques sont fusionnées dans la base de données. 

1. Accédez au répertoire **functions**.
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. Créez une action **collectStats**. Elle utilise un [environnement Python 3](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments) qui inclut déjà le pilote de base de données requis. Le code source de l'action est fourni dans le fichier `ghstats.zip`.
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   Si vous modifiez le code source de l'action (`__main__.py`), vous pouvez alors reconditionner l'archive zip avec `zip -r ghstats.zip  __main__.py github.py`. Pour plus de détails, consultez le fichier `setup.sh`.
   {:tip}
3. Liez l'action au service de base de données. Utilisez l'instance et la clé de service que vous avez créées lors de la configuration de l'environnement. 
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. Créez un déclencheur basé sur le [package alarms](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm). Il prend en charge différentes formes de spécification de l'alarme. Utilisez le style [cron](https://en.wikipedia.org/wiki/Cron)-like. A compter du 21 avril et jusqu'au 21 décembre, le déclencheur s'exécute tous les jours à 6 heures UTC. Veillez à disposer d'une date de début ultérieure. 
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  Vous pouvez transformer le déclencheur quotidien en déclencheur hebdomadaire en appliquant `"0 6 * * 0"`. Il devrait s'exécuter tous les dimanches à 6 heures. {:tip}
5. Enfin, créez une règle **myStatsRule** qui connecte le déclencheur **myDaily** à l'action **collectStats**. Désormais, le déclencheur entraîne l'exécution de l'action selon la planification spécifiée à l'étape précédente. 
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. Appelez l'action pour un premier test. Le nombre **repoCount** renvoyé doit refléter le nombre de référentiels que vous avez configurés plus haut. 
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
   La sortie ressemble à ceci :
   ```
   {
       "repoCount": 18
   }
   ```
7. Dans la fenêtre de votre navigateur, à partir de la page de l'application, vous pouvez désormais consulter le trafic du référentiel. Par défaut, 10 entrées sont affichées. Vous pouvez les remplacer par différentes valeurs. Il est également possible de trier les colonnes de la table ou d'utiliser la zone de recherche pour filtrer des référentiels spécifiques. Vous pouvez entrer une date et un nom d'organisation, puis trier par nombre de vues pour répertorier les meilleurs marqueurs d'une journée donnée.
   ![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## Conclusions
Dans ce tutoriel, vous avez déployé une action sans serveur avec un déclencheur et une règle associés. Ils permettent d'extraire automatiquement les données de trafic pour les référentiels GitHub. Les informations sur ces référentiels, y compris le jeton d'accès spécifique au titulaire, sont stockées dans une base de données SQL ({{site.data.keyword.dashdbshort}}). L'application Cloud Foundry utilise cette base de données pour gérer les utilisateurs, les référentiels et pour présenter les statistiques de trafic dans le portail des applications. Les utilisateurs peuvent consulter les statistiques de trafic dans des tables interrogeables ou visualisées dans un tableau de bord intégré (service {{site.data.keyword.dynamdashbemb_short}}, voir image ci-dessous). Il est également possible de télécharger la liste des référentiels et les données de trafic sous forme de fichiers CSV. 

L'application Cloud Foundry gère l'accès via un client OpenID Connect se connectant à {{site.data.keyword.appid_short}}.
![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## Nettoyage
Pour nettoyer les ressources utilisées dans ce tutoriel, vous pouvez supprimer les services et l'application associés, ainsi que l'action, le déclencheur et la règle dans l'ordre inverse de leur création : 

1. Supprimez la règle, le déclencheur et l'action relatifs à {{site.data.keyword.openwhisk_short}}.
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. Supprimez l'application Python et ses services. 
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## Extension du tutoriel 
Vous souhaitez développer ou modifier ce tutoriel ? Voici quelques suggestions :
* Développez l'application pour la prise en charge du service partagé. 
* Intégrez un graphique pour les données. 
* Utilisez des fournisseurs d'identité sociale. 
* Ajoutez un sélecteur de date à la page de statistiques pour filtrer les données affichées. 
* Utilisez une page de connexion personnalisée pour {{site.data.keyword.appid_short}}.
* Explorez les relations de codage social entre développeurs à l'aide de [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Contenu associé
Voici des liens vers des informations supplémentaires sur les sujets abordés dans ce tutoriel. 

Documentation et SDK :
* [Documentation {{site.data.keyword.openwhisk_short}} ](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Documentation : [IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Documentation {{site.data.keyword.appid_short}}](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [Exécution de Python sur IBM Cloud](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
