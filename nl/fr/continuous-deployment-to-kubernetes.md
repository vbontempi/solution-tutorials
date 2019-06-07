---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# Déploiement continu sur Kubernetes 
{: #continuous-deployment-to-kubernetes}

Ce tutoriel vous guide tout au long du processus de configuration de l'intégration continue et d'un pipeline de distribution pour les applications conteneurisées s'exécutant sur {{site.data.keyword.containershort_notm}}. Vous allez apprendre à configurer le contrôle de source, puis à générer, tester et déployer le code à différentes étapes de déploiement. Vous allez ensuite ajouter des intégrations à d'autres services tels que les scanners de sécurité, les notifications Slack et l'analyse. 

{:shortdesc}

## Objectifs
{: #objectives}

* Créer des clusters Kubernetes de développement et de production. 
* Créer une application de démarrage, l'exécuter localement et la transférer dans un référentiel Git. 
* Configurer le pipeline de distribution DevOps pour qu'il se connecte à votre référentiel Git, créer et déployer l'application de démarrage sur les clusters de développement/production. 
* Explorer et intégrer l'application pour utiliser des scanners de sécurité, des notifications Slack et l'analyse. 

## Services utilisés
{: #services}

Ce tutoriel utilise les services {{site.data.keyword.Bluemix_notm}} suivants : 

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**Attention :** ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

![](images/solution21/Architecture.png)

1. Transférez le code dans un référentiel Git privé. 
2. Le pipeline enregistre les modifications dans Git et génère l'image du conteneur. 
3. L'image du conteneur est chargée dans le registre déployé dans un cluster de développement Kubernetes. 
4. Validez les modifications et déployez-les sur le cluster de production. 
5. Configurez les notifications Slack pour les activités de déploiement. 


## Avant de commencer
{: #prereq}

* [Installez {{site.data.keyword.dev_cli_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - Script pour installer docker, kubectl, helm, ibmcloud cli et les plug-ins requis.
* [Configurez l'interface de ligne de commande {{site.data.keyword.registrylong_notm}} et votre espace de nom de registre.](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Intégrez les concepts de base de Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Création d'un cluster de développement Kubernetes 
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} propose des outils puissants en combinant les technologies Docker et Kubernetes, une expérience utilisateur intuitive et une sécurité et un isolement intégrés pour automatiser le déploiement, l'exploitation, la mise à l'échelle et la surveillance d'applications conteneurisées dans un cluster d'hôtes de calcul.

Pour effectuer ce tutoriel, vous devez sélectionner le cluster **Payant** de type **Standard**. Vous devrez configurer deux clusters, un pour le développement et un pour la production.
{: shortdesc}

1. Créez le premier cluster de développement Kubernetes à partir du catalogue [{{site.data.keyword.Bluemix}}](https://{DomainName}/containers-kubernetes/launch). Ultérieurement, vous devrez répéter ces étapes et créer un cluster de production. 

   Pour plus de facilité, vérifiez les détails de la configuration, tels que le nombre d'unités centrales, la mémoire et le nombre de noeuds worker que vous obtenez.
{:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. Sélectionnez le **Type de cluster** et cliquez sur **Créer un cluster** pour mettre à disposition un cluster Kubernetes. Le **Type de machine** le plus petit avec 2 **UC**, 4 **Go de RAM** et 1 **Noeud worker** est suffisant pour ce tutoriel. Toutes les autres options peuvent conserver leurs valeurs par défaut. 
3. Vérifiez l'état du **Cluster** et du **Noeud worker** et patientez jusqu'à ce qu'ils soient prêts (**ready**).

**Remarque :** ne commencez pas tant que le noeud worker n'est pas prêt.

## Création d'une application de démarrage
{: #create_application}

{{site.data.keyword.containershort_notm}} propose une sélection d'applications de démarrage qui peuvent être créées à l'aide de la commande `ibmcloud dev create` ou de la console Web. Dans ce tutoriel, vous allez utiliser la console Web. L'application de démarrage réduit considérablement le temps de développement en proposant des kits de démarrage contenant tout le code réutilisable de génération et de configuration nécessaire pour vous permettre de commencer à coder la logique métier plus rapidement. 

1. Dans la [console {{site.data.keyword.cloud_notm}}](https://{DomainName}), utilisez l'option de menu de gauche et sélectionnez [Applications Web](https://{DomainName}/developer/appservice/dashboard).
2. Dans la section **Démarrer à partir du Web**, cliquez sur le bouton **Initiation**.
3. Sélectionnez la vignette `Node.js Web App with Express.js`, puis `Créer une application` pour créer une application de démarrage Node.js.
4. Entrez le **nom** `mynodestarter`. Ensuite, cliquez sur **Créer**.

## Configuration du pipeline de distribution DevOps 
{: #create_devops}

1. Une fois que vous avez créé l'application de démarrage, sous **Déployez votre application**, cliquez sur le bouton **Déployer dans le cloud**.
2. En sélectionnant la méthode de déploiement du cluster Kubernetes, sélectionnez le cluster créé précédemment, puis cliquez sur **Créer**. Vous créez ainsi automatiquement une chaîne d’outils et un pipeline de distribution. ![](images/solution21/BindCluster.png)
3. Une fois le pipeline créé, cliquez sur **Afficher la chaîne d'outils**, puis sur **Pipeline de distribution** pour afficher le pipeline. ![](images/solution21/Delivery-pipeline.png)
4. Une fois les étapes de déploiement terminées, cliquez sur **Afficher les journaux et l'historique** pour consulter les journaux.
5. Consultez l'URL affichée pour accéder à l'application (`http://worker-public-ip:portnumber/`). ![](images/solution21/Logs.png)
Vous avez terminé cette étape. Vous avez utilisé l'interface utilisateur des Services d'appli pour créer les applications de démarrage et configuré le pipeline pour générer et déployer l'application sur votre cluster. 

## Clonage, génération et exécution de l'application en local 
{: #cloneandbuildapp}

Dans cette section, vous allez utiliser l'application de démarrage créée dans la section précédente, la cloner sur votre ordinateur local, modifier le code, puis la générer/l'exécuter localement.
{: shortdesc}

### Clonage de l'application
1. Dans la Présentation de la chaîne d'outils, sélectionnez la vignette **Git** sous **Code**. Vous êtes redirigé vers votre page de référentiel git où vous pouvez cloner le référentiel.![HelloWorld](images/solution21/DevOps_Toolchain.png)

2. Si vous n'avez pas encore configuré les clés SSH, une barre de notification contenant des instructions devrait apparaître dans la partie supérieure. Suivez les étapes en ouvrant le lien **ajouter une clé SSH** dans un nouvel onglet ou si vous souhaitez utiliser HTTPS au lieu de SSH, suivez les étapes en cliquant sur **créer un jeton d'accès personnel**. N'oubliez pas de sauvegarder la clé ou le jeton pour référence ultérieure. 
3. Sélectionnez SSH ou HTTPS et copiez l'URL de git. Clonez la source sur votre ordinateur local. Si vous êtes invité à entrer un nom d'utilisateur, indiquez votre nom d'utilisateur git. Pour le mot de passe, utilisez une **clé SSH** ou un **jeton d'accès personnel** ou celui créé que vous avez créé à l'étape précédente.

   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   {: codeblock}

4. Ouvrez le référentiel cloné dans l'environnement IDE de votre choix et accédez à `public/index.html`. Mettez à jour le code en essayant de remplacer "Congratulations!" par autre chose, puis enregistrez le fichier.

### Génération de l'application en local
Vous pouvez générer et exécuter l’application comme vous le feriez normalement avec `mvn` pour le développement local java ou `npm` pour le développement de noeud. Vous pouvez également générer une image Docker et exécuter l'application dans un conteneur pour garantir une exécution cohérente localement et sur le cloud. Utilisez les étapes suivantes pour générer l'image Docker.
{: shortdesc}

1. Vérifiez que votre moteur Docker local est démarré. Pour ce faire, exécutez la commande ci-dessous : 
   ```
   docker ps
   ```
   {: codeblock}
2. Accédez au répertoire de projet généré cloné. 
   ```
   cd <project name>
   ```
   {: codeblock}
3. Générez l'application en local
   ```
   ibmcloud dev build
   ```
   {: codeblock}

   L'exécution peut prendre quelques minutes, car toutes les dépendances de l'application sont téléchargées et une image Docker contenant votre application et tout l'environnement requis est générée. 

### Exécution de l'application en local

1. Exécutez le conteneur.
   ```
   ibmcloud dev run
   ```
   {: codeblock}

   Cette commande utilise votre moteur Docker local pour exécuter l'image Docker que vous avez créée à l'étape précédente. 
2. Une fois votre conteneur démarré, accédez à http://localhost:3000/
   ![](images/solution21/node_starter_localhost.png)

## Transfert de l'application dans votre référentiel Git 

Dans cette section, vous allez valider vos modifications dans le référentiel Git. Le pipeline prend en compte la validation et transmet automatiquement les modifications à votre cluster. 
1. Dans la fenêtre de votre terminal, assurez-vous que vous vous trouvez dans le référentiel que vous avez cloné. 
2. Transférez les modifications dans votre référentiel en trois étapes simples : ajouter (add), valider (commit) et transférer (push). 
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
   {: codeblock}

3. Accédez à la chaîne d'outils que vous avez créée précédemment et cliquez sur la vignette **Pipeline de distribution**.
4. Confirmez que vous voyez les étapes **BUILD** et **DEPLOY**.
   ![](images/solution21/Delivery-pipeline.png)
5. Patientez jusqu'à la fin de l'étape **DEPLOY**.
6. Cliquez sur l'**url** de l'application sous Résultat de la dernière exécution pour afficher vos modifications en temps réel.

Si l'application n'est pas mise à jour, consultez les journaux des étapes DEPLOY et BUILD du pipeline. 

## Sécurité à l'aide de Vulnerability Advisor 
{: #vulnerability_advisor}

Au cours de cette étape, vous allez explorer [Vulnerability Advisor](https://{DomainName}/docs/services/va?topic=va-va_index#va_index). Vulnerability Advisor permet de vérifier le statut de sécurité des images de conteneur avant le déploiement, ainsi que de vérifier le statut des conteneurs en cours d'exécution.

1. Accédez à la chaîne d'outils que vous avez créée précédemment et cliquez sur la vignette **Pipeline de distribution**.
1. Cliquez sur **Ajouter une étape** et remplacez MyStage par **Validate Stage** puis cliquez sur TRAVAUX  > **AJOUTER UN TRAVAIL**.

   1. Sélectionnez **Test** comme type de travail et remplacez **Test** par **Vulnerability Advisor** dans la zone.
   1. Sous Type de testeur, sélectionnez **Vulnerability Advisor**. Toutes les autres zones doivent être renseignées automatiquement. L'espace de nom de Container Registry doit être identique à celui mentionné dans l'**Etape de génération** de cette chaîne d'outils.
{:tip}
   1. Modifiez la section **Script de test** et remplacez `SAFE\ to\ deploy` à la dernière ligne par `NO\ ISSUES`
   1. Enregistrez l'étape. 
1. Faites glisser et déplacez **Validate Stage** au milieu, puis cliquez sur **Exécuter** ![](images/solution21/run.png) sur **Validate Stage**. Vous pouvez constater que l'étape **Validate stage** échoue.

   ![](images/solution21/toolchain.png)

1. Cliquez sur **Afficher les journaux et l'historique** pour afficher l'évaluation de la vulnérabilité. La fin du journal indique :

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   Vous pouvez voir les évaluations de vulnérabilité détaillées de tous les référentiels analysés [ici](https://{DomainName}/containers-kubernetes/registry/private)
   {:tip}

   L'étape peut échouer en indiquant que l'image *n'a pas été analysée* si l'analyse des vulnérabilités dure plus de 3 minutes. Ce délai peut être modifié en éditant le script du travail et en augmentant le nombre d'itérations pour attendre les résultats de l'analyse.
   {:tip}

1. Corrigez les vulnérabilités à l'aide de l'action corrective. Ouvrez le référentiel cloné dans un environnement IDE ou sélectionnez la vignette de l'IDE Web Eclipse Orion, ouvrez `Dockerfile` et ajoutez la commande ci-dessous après `EXPOSE 3000` 
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. Validez et transférez les modifications. Vous devriez ainsi déclencher la chaîne d’outils et corriger l’étape **Validate Stage**.

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## Création d'un cluster de production Kubernetes 

{: #deploytoproduction}

Dans cette section, vous allez exécuter le pipeline de déploiement en déployant l'application Kubernetes dans les environnements de développement et de production respectivement. Dans l'idéal, nous souhaitons configurer un déploiement automatique pour l'environnement de développement et un déploiement manuel pour l'environnement de production. Avant cela, explorons les deux manières de réaliser cette procédure. Il est possible d'utiliser un cluster pour les environnements de développement et de production. Cependant, il est recommandé de disposer de deux clusters distincts, un pour le développement et un pour la production. Explorons la mise en place d'un deuxième cluster pour la production.
{: shortdesc}

1. Suivez les instructions de la section [Création d'un cluster de développement Kubernetes](#create_kube_cluster) et créez un cluster. Nommez ce cluster `prod-cluster`.
2. Accédez à la chaîne d'outils que vous avez créée précédemment et cliquez sur la vignette **Pipeline de distribution**.
3. Renommez **Etape de déploiement** en `Deploy dev`. Pour ce faire, cliquez sur l’icône Paramètres >  **Configurer l'étape**.
   ![](images/solution21/deploy_stage.png)
4. Clonez l’étape **Deploy dev** (icône Paramètres > Cloner l'étape) et nommez l’étape clonée `Deploy prod`.
5. Définissez le **Déclencheur d'étape** sur `Exécuter les travaux lorsque cette étape est exécutée manuellement`. ![](images/solution21/prod-stage.png)
6. Sous l'onglet **Travail**, remplacez le nom du cluster par celui du nouveau cluster, puis **Sauvegardez** l'étape.
7. Vous devez maintenant disposer de la configuration de déploiement complète. Pour déployer de développement à production, vous devez exécuter manuellement l’étape `Deploy prod` pour effectuer le déploiement dans l'environnement de production. ![](images/solution21/full-deploy.png)

Cette étape est terminée. Vous avez créé un cluster de production et configuré le pipeline pour envoyer manuellement les mises à jour vers le cluster de production. Il s'agit d'une étape de processus de simplification par rapport à un scénario plus avancé dans lequel vous incluez des tests d'unité et des tests d'intégration dans le pipeline. 

## Configuration des notifications Slack 
{: #setup_slack}

1. Retournez à la liste des [chaînes d'outils](https://{DomainName}/devops/toolchains) , sélectionnez votre chaîne d'outils, puis cliquez sur **Ajouter un outil**.
2. Recherchez Slack dans la zone de recherche ou faites défiler l'écran vers le bas pour afficher **Slack**. Cliquez pour visualiser la page de configuration.
    ![](images/solution21/configure_slack.png)
3. Pour **Webhook Slack**, suivez les étapes décrites dans ce [lien](https://my.slack.com/services/new/incoming-webhook/). Vous devez vous connecter avec vos données d'identification Slack et fournir un nom de canal existant ou en créer un. 
4. Une fois que l'intégration de Webhook entrant est ajoutée, copiez l'**URL du webhook** et collez-la sous **Webhook Slack**.
5. Le canal Slack est le nom de canal que vous avez fourni lors de la création d’une intégration Webhook ci-dessus. 
6. Le **Nom d'équipe Slack** est le nom d'équipe (première partie) de team-name.slack.com. Par exemple, kube est le nom de l'équipe dans kube.slack.com 
7. Cliquez sur **Créer une intégration**. Une nouvelle vignette est ajoutée à votre chaîne d’outils.
    ![](images/solution21/toolchain_slack.png)
8. A partir de maintenant, chaque fois que votre chaîne d'outils s'exécute, vous devriez voir des notifications Slack dans le canal que vous avez configuré.
    ![](images/solution21/slack_channel.png)

## Suppression de ressources 
{: #removeresources}

Dans cette étape, vous nettoyez les ressources pour supprimer ce que vous avez créé aux étapes précédentes. 

- Supprimez le référentiel Git. 
- Supprimez la chaîne d'outils. 
- Supprimez les deux clusters. 
- Supprimez le canal Slack. 

## Extension du tutoriel 
{: #expandTutorial}

Vous souhaitez en savoir plus ? Voici quelques idées de ce que vous pouvez faire ensuite : 

- [Analyse des journaux et surveillance de la santé des applications avec LogDNA et Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Ajout d'un environnement de test et déploiement sur un 3e cluster. 
- Déploiement du cluster de production [sur plusieurs emplacements](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Amélioration de votre pipeline avec des contrôles de qualité et des analyses supplémentaires à l'aide de [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Contenu associé
{: #related}

* Guide complet de la solution Kubernetes, [déplacement d'applications basées sur des machines virtuelles vers Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes).
* [Sécurité](https://{DomainName}/docs/containers?topic=containers-security#cluster) d'IBM Cloud Container Service.
* [Intégrations](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations) à la chaîne d'outils.
* Analyse des journaux et surveillance de la santé des applications avec [LogDNA et Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).


