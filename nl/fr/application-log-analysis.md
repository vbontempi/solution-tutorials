---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Analyse des journaux et surveillance de la santé des applications avec LogDNA et Sysdig 
{: #application-log-analysis}

Ce tutoriel montre comment le service [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/) peut être utilisé pour configurer les journaux d'une application Kubernetes déployée sur {{site.data.keyword.Bluemix_notm}} et y accéder. Vous allez déployer une application Python sur un cluster mis à disposition sur {{site.data.keyword.containerlong_notm}}, configurer un agent LogDNA, générer différents niveaux de journaux d'application et accéder aux journaux worker, de pod ou de réseau. Vous allez ensuite rechercher, filtrer et visualiser ces journaux via l'interface utilisateur Web {{site.data.keyword.la_short}}.

De plus, vous allez configurer le service [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/) et l'agent Sysdig pour surveiller les performances et la santé de votre application et de votre cluster {{site.data.keyword.containerlong_notm}}.
{:shortdesc}

## Objectifs
{: #objectives}
* Déployer une application sur un cluster Kubernetes pour générer des entrées de journal. 
* Accéder à différents types de journaux et les analyser afin de résoudre les incidents et d’anticiper les problèmes. 
* Bénéficier d'une visibilité opérationnelle sur les performances et la santé de votre application, ainsi que sur le cluster exécutant votre application. 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

  ![](images/solution12/Architecture.png)

1. L'utilisateur se connecte à l'application et génère des entrées de journal. 
1. L'application s'exécute dans un cluster Kubernetes à partir d'une image stockée dans {{site.data.keyword.registryshort_notm}}.
1. L'utilisateur configure l'agent de service {{site.data.keyword.la_full_notm}} pour accéder aux journaux des applications et des clusters.
1. L'utilisateur configure l'agent de service {{site.data.keyword.mon_full_notm}} pour surveiller l'intégrité et les performances du cluster {{site.data.keyword.containerlong_notm}}, ainsi que de l'application déployée sur le cluster.

## Conditions préalables
{: #prereq}

* [Installez {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - Script pour installer docker, kubectl, helm, ibmcloud cli et les plug-ins requis.
* [Configurez l'interface de ligne de commande {{site.data.keyword.registrylong_notm}} et votre espace de nom de registre.](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Octroyez des droits à un utilisateur pour afficher les journaux dans LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna)
* [Octroyez des droits à un utilisateur pour afficher les métriques dans Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig)

## Création d'un cluster Kubernetes 
{: #create_cluster}

{{site.data.keyword.containershort_notm}} fournit un environnement permettant de déployer des applications hautement disponibles dans des conteneurs Docker exécutés dans des clusters Kubernetes.

Ignorez cette section si vous disposez d'un cluster **Standard** existant et que vous souhaitez le réutiliser avec ce tutoriel.
{: tip}

1. Créez un **cluster** à partir du catalogue [{{site.data.keyword.Bluemix}}](https://{DomainName}/kubernetes/catalog/cluster/create) et choisissez le cluster **Standard**.
   Le transfert des journaux *n'est pas* activé pour le cluster **Gratuit**.
   {:tip}
1. Sélectionnez un groupe de ressources et une géographie. 
1. Pour plus de commodité, utilisez le nom `mycluster` pour être cohérent avec ce tutoriel.
1. Sélectionnez une **Zone worker** et le **Type de machine** le plus petit avec 2 **UC** et 4 **Go de RAM** car ces quantités sont suffisantes pour ce tutoriel.
1. Sélectionnez 1 **Noeud worker** et laissez toutes les autres options définies par défaut. Cliquez sur **Créer un cluster**.
1. Vérifiez l'état de votre **Cluster** et de votre **Noeud worker** et patientez jusqu'à ce qu'ils soient prêts (**ready**).

## Mise à disposition d'une instance {{site.data.keyword.la_short}}
{: #provision_logna_instance}

Les applications déployées sur un cluster {{site.data.keyword.containerlong_notm}} dans {{site.data.keyword.Bluemix_notm}} généreront probablement un certain niveau de sortie de diagnostic, c'est-à-dire des journaux. En tant que développeur ou opérateur, vous pouvez accéder à différents types de journaux, tels que les journaux worker, les journaux de pod, les journaux d'applications ou les journaux de réseau, et les analyser afin de résoudre les incidents et les anticiper. 

En utilisant le service {{site.data.keyword.la_short}}, il est possible d'agréger les journaux de différentes sources et de les conserver aussi longtemps que nécessaire. Vous pouvez ainsi analyser le "contexte global" lorsque cela est nécessaire et résoudre des situations plus complexes. 

Pour mettre à disposition un service {{site.data.keyword.la_short}},

1. Accédez à la page [Observabilité](https://{DomainName}/observe/) et,sous **Journalisation**, cliquez sur **Créer une instance**.
1. Fournissez un **Nom de service** unique.
1. Choisissez une région/un emplacement et sélectionnez un groupe de ressources. 
1. Sélectionnez **Recherche dans les journaux pendant 7 jours** comme plan et cliquez sur **Créer**.

Le service fournit un système de gestion des journaux centralisé dans lequel les données de journal sont hébergées sur IBM Cloud. 

## Déploiement et configuration d'une application Kubernetes pour transférer des journaux 
{: #deploy_configure_kubernetes_app}

[Le code prêt à l'emploi pour l'application de journalisation se trouve dans ce référentiel GitHub](https://github.com/IBM-Cloud/application-log-analysis). L'application est écrite à l'aide de [Django](https://www.djangoproject.com/), une infrastructure Web côté serveur Python bien connue. Clonez ou téléchargez le référentiel, puis déployez l'application sur {{site.data.keyword.containershort_notm}} sur {{site.data.keyword.Bluemix_notm}}.

### Déploiement de l'application Python

Sur un terminal :
1. Clonez le référentiel GitHub : 
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. Accédez au répertoire de l'application
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. Créez une image Docker avec le fichier [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile) dans {{site.data.keyword.registryshort_notm}}.
   - Recherchez le **Container Registry** avec `ibmcloud cr info`, tel que us.icr.io or uk.icr.io.
   - Créez un espace de nom pour stocker l'image du conteneur.
```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - Remplacez `<CONTAINER_REGISTRY>` par la valeur de registre de votre conteneur et utilisez **app-log-analysis** comme nom d'image.
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - Remplacez la valeur **image** dans le fichier `app-log-analysis.yaml` par la balise d'image `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest`
1. Exécutez la commande ci-dessous pour extraire la configuration du cluster et définir la variable d’environnement `KUBECONFIG` :
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. Déployez l'application :
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. Pour accéder à l'application, vous avez besoin de l'`adresse IP publique` du noeud worker et de `NodePort`
   - Pour une adresse IP publique, exécutez la commande suivante :
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - Pour le NodePort qui est composé de 5 chiffres (par exemple, 3xxxx), exécutez la commande ci-dessous :
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   Vous pouvez maintenant accéder à l'application à l'adresse `http://worker-ip-address:portnumber`

### Configuration du cluster pour envoyer des journaux à votre instance LogDNA 

Pour configurer votre cluster Kubernetes de manière à envoyer des journaux à votre instance {{site.data.keyword.la_full_notm}}, vous devez installer un pod *logdna-agent* sur chaque noeud de votre cluster. L'agent LogDNA lit les fichiers journaux depuis le pod où il est installé et envoie les données des journaux à votre instance LogDNA.

1. Accédez à la page [Observabilité](https://{DomainName}/observe/) et cliquez sur **Journalisation**.
1. Cliquez sur **Editer les sources de journal** en regard du service que vous avez créé et sélectionnez **Kubernetes**.
1. Copiez et exécutez la première commande sur un terminal sur lequel vous avez défini la variable d’environnement `KUBECONFIG` afin de créer un secret kubernetes avec la clé d’ingestion LogDNA pour votre instance de service. 
1. Copiez et exécutez la deuxième commande pour déployer un agent LogDNA sur chaque noeud worker de votre cluster Kubernetes. L'agent LogDNA collecte les journaux portant l'extension **.log* et les fichiers sans extension stockés dans le répertoire */var/log* de votre pod. Par défaut, les journaux sont collectés à partir de tous les espaces de nom, y compris kube-system, et automatiquement envoyés au service {{site.data.keyword.la_full_notm}}.
1. Après avoir configuré une source de journal, lancez l'interface utilisateur LogDNA en cliquant sur **Afficher LogDNA**. L'affichage des journaux peut prendre quelques minutes.

## Génération des journaux d'application et accès 
{: generate_application_logs}

Dans cette section, vous allez générer des journaux d'application et les examiner dans LogDNA. 

### Génération des journaux d'application 

L'application déployée aux étapes précédentes vous permet de consigner un message à un niveau de journalisation choisi. Les niveaux de journalisation disponibles sont **critique**, **erreur**, **avertissement**, **info** et **débogage**. L'infrastructure de journalisation de l'application est configurée pour autoriser uniquement le passage d'entrées de journal supérieures ou égales à un niveau défini. Initialement, le niveau de journalisation est configuré pour **avertissement**. Ainsi, un message consigné au niveau **info** avec le paramètre de serveur **warn** n'apparaît pas dans la sortie de diagnostic. 

Observez le code dans le fichier [**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py). Le code contient des instructions **print** ainsi que des appels à des fonctions **logger**. Les messages imprimés sont écrits dans le flux **stdout** (sortie standard, console/terminal d’application), les messages de journalisation apparaissent dans le flux **stderr** (journal des erreurs). 

1. Visitez l'application Web à l'adresse `http://worker-ip-address:portnumber`.
1. Générez plusieurs entrées de journal en soumettant des messages à différents niveaux. L'interface utilisateur permet également de modifier le paramètre de journalisation pour le niveau de journalisation du serveur. Modifiez le niveau de journalisation côté serveur intermédiaire pour augmenter son intérêt. Par exemple, vous pouvez consigner une "erreur de serveur interne 500" en tant qu'**erreur** ou le message "Ceci est la première entrée de journal" en tant qu'**info**.

### Accès aux journaux d'application 

Vous pouvez accéder au journal spécifique de l'application dans l'interface utilisateur LogDNA à l'aide des filtres. 

1. Sur la barre du haut, cliquez sur **Toutes les applications**.
1. Sous conteneurs, vérifiez **app-log-analysis**. Une nouvelle vue non enregistrée s'affiche avec les journaux d'application de tous les niveaux.
1. Pour voir les journaux de niveau(x) spécifique(s), cliquez sur **Tous les niveaux**, et sélectionnez plusieurs niveaux tels que Erreur, info, avertissement, etc.

## Recherche et filtrage de journaux 
{: #search_filter_logs}

L'interface utilisateur {{site.data.keyword.la_short}} par défaut affiche toutes les entrées de journal disponibles (Tout). Les entrées les plus récentes sont affichées à la fin par le biais d’une actualisation automatique. Dans cette section, vous allez modifier le type et le volume d'informations affichées et les sauvegarder en tant que **Vue** pour une utilisation ultérieure. 

### Recherche dans les journaux

1. Dans la zone de saisie **Recherche** située au bas de la page dans l’interface utilisateur de LogDNA, 
   - vous pouvez rechercher des lignes contenant un texte spécifique tel que **"Ceci est la première entrée de journal"** ou **erreur de serveur interne 500**.
   - ou un niveau de journalisation spécifique en entrant `level:info` où level est une zone qui accepte une valeur de chaîne. 

   Pour consulter davantage de zones de recherche et d’aide, cliquez sur l’icône d’aide sur la syntaxe en regard de la zone de saisie de la recherche.
{:tip}
1. Pour passer à une période spécifique, entrez **il y a 5 min** dans la zone de saisie **Jump to timeframe**. Cliquez sur l'icône située en regard de la zone de saisie pour rechercher les autres formats horaire compris dans la durée de conservation. 
1. Pour mettre les termes en surbrillance, cliquez sur l’icône **Toggle Viewer Tools**.
1. Entrez **erreur** comme terme en surbrillance dans la première zone de saisie, **conteneur** comme terme en surbrillance dans la deuxième zone de saisie et vérifiez les lignes surlignées avec les termes.
1. Cliquez sur l'icône **Toggle Timeline** pour afficher les lignes contenant des journaux à une heure précise de la journée.

### Filtrage des journaux

Vous pouvez filtrer les journaux par étiquettes, sources, applications ou niveaux. 

1. Sur la barre du haut, cliquez sur **All Tags** et cochez la case **k8s** pour afficher les journaux liés à Kubernetes.
1. Cliquez sur **All Sources** et sélectionnez le nom de l'hôte (noeud worker) concerné lors de la vérification des journaux. Cette méthode est efficace si le cluster comporte plusieurs noeuds worker. 
1. Pour vérifier les journaux de conteneur ou de fichier, cliquez sur **All Apps** et cochez la ou les cases dont vous souhaitez visualiser les journaux.

### Création d'une vue

Les vues sont des raccourcis enregistrés vers un ensemble spécifique de filtres et de requêtes de recherche. 

Dès que vous recherchez ou filtrez des journaux, l'option **Unsaved View** devrait apparaître dans la barre supérieure. Pour enregistrer les résultats en tant que vue: 
1. Cliquez sur **All Apps** et cochez la case en regard de **app-log-analysis**
1. Cliquez sur **Unsaved View** >, cliquez sur **save as new view / alert** et nommez la vue **app-log-analysis-view**. Laissez **Category** vide.
1. Cliquez sur **Save View**. Une nouvelle vue devrait apparaître dans le volet de gauche et afficher les journaux de l'application.

### Visualisation des journaux dans des graphiques et des ventilations 

Dans cette section, vous allez créer un tableau, puis ajouter un graphique avec une ventilation pour visualiser les données au niveau de l'application. Un tableau est un ensemble de graphiques et de ventilations. 

1. Dans le volet de gauche, cliquez sur l'icône du **tableau** (au-dessus de l'icône des paramètres) > cliquez sur **NEW BOARD**.
1. Cliquez sur **Edit** dans la barre supérieure et attribuez le nom **app-log-analysis-board**. Cliquez sur **Save**.
1. Cliquez sur **Add Graph**:
   - Entrez **app** comme zone de la première zone de saisie et appuyez sur Entrée.
   - Choisissez **app-log-analysis** comme valeur de zone.
   - Cliquez sur **Add Graph**.
1. Sélectionnez **Counts** en tant que métrique pour afficher le nombre de lignes de chaque intervalle au cours des dernières 24 heures.
1. Pour ajouter une ventilation, cliquez sur la flèche située sous le graphique : 
   - Choisissez **Histogram** comme type de ventilation.
   - Choisissez **level** comme type de zone.
   - Cliquez sur **Add Breakdown** pour afficher une ventilation de tous les niveaux que vous avez enregistrés pour l'application.

## Ajout de {{site.data.keyword.mon_full_notm}} et surveillance du cluster
{: #monitor_cluster_sysdig}

Les étapes suivantes consistent à ajouter {{site.data.keyword.mon_full_notm}} à l'application. Le service vérifie régulièrement la disponibilité et le temps de réponse de l'application. 

1. Accédez à la page [Observabilité](https://{DomainName}/observe/) et,sous **Surveillance**, cliquez sur **Créer une instance**.
1. Fournissez un **Nom de service** unique.
1. Choisissez une région/un emplacement et sélectionnez un groupe de ressources. 
1. Sélectionnez **Par niveau de catégorie** comme plan et cliquez sur **Créer**.
1. Cliquez sur **Editer les sources de journal** en regard du service que vous avez créé et sélectionnez **Kubernetes**.
1. Copiez et exécutez la commande sous **Installez l'agent Sysdig sur votre cluster** sur un terminal où vous avez défini la variable d'environnement `KUBECONFIG` pour déployer l'agent Sysdig dans votre cluster. Attendez la fin du déploiement. 

### Configuration de {{site.data.keyword.mon_short}}

Pour configurer Sysdig afin de surveiller la santé et les performances de votre cluster : 
1. Cliquez sur **Afficher Sysdig**, vous devriez voir l'interface utilisateur du moniteur sysdig. Sur la page d'accueil, cliquez sur **Suivant**.
1. Choisissez **Kubernetes** comme méthode d'installation dans l'environnement de configuration. 
1. Cliquez sur **Go to Next step** en regard du message de réussite de la configuration de l'agent, puis cliquez sur **Commençons !** sur la page suivante.
1. Cliquez sur **Next**, puis sur **Complete onboarding** pour afficher l'onglet `Explorer` de l'interface utilisateur Sysdig.

### Surveillance du cluster 

Pour vérifier la santé et les performances de votre cluster amd d'application : 
1. De retour dans l'application exécutée à l'adresse `http://worker-ip-address:portnumber`, générez plusieurs entrées de journal.
1. Développez **mycluster** dans le volet gauche > développez **default** namespace > cliquez sur **app-log-analysis-deployment** pour afficher le nombre de demandes, le temps de réponse, etc., dans l'assistant de surveillance Sysdig.
1. Pour vérifier les codes de requête-réponse HTTP, cliquez sur la flèche en regard de **Kubernetes Pod Health** dans la barre supérieure et sélectionnez **HTTP** sous **Applications**. Modifiez l'intervalle à **10 M** dans la barre inférieure de l'interface utilisateur Sysdig.
1. Pour surveiller le temps de réponse de l'application, 
   - Dans l'onglet Explorer, sélectionnez **Deployments and Pods**.
   - Cliquez sur la flèche en regard de `HTTP`, puis sélectionnez Métriques > Réseau.
   - Sélectionnez **net.http.request.time**.
   - Sélectionnez Heure : **Somme** et Groupe : **Moyenne**.
   - Cliquez sur **Plus d'options**, puis sur l'icône **Topologie**.
   - Cliquez sur **Terminé** et cliquez deux fois sur la case pour développer la vue. 
1. Pour surveiller l'espace de nom Kubernetes où l'application est en cours d'exécution, 
   - Dans l'onglet Explorer, sélectionnez **Deployments and Pods**.
   - Cliquez sur la flèche en regard de `net.http.request.time`.
   - Sélectionnez Default Dashboards > Kubernetes.
   - Sélectionnez Kubernetes State > Kubernetes State Overview.

### Création d'un tableau de bord personnalisé 

En plus des tableaux de bord prédéfinis, vous pouvez créer votre propre tableau de bord personnalisé pour afficher les vues et les métriques les plus utiles/pertinentes pour les conteneurs exécutant votre application à un emplacement unique. Chaque tableau de bord est composé d'une série de panneaux configurés pour afficher des données spécifiques dans un certain nombre de formats différents. 

Pour créer un tableau de bord, procédez comme suit :
1. Cliquez sur **Dashboards** dans le volet le plus à gauche > cliquez sur **Add Dashboard**.
1. Cliquez sur **Blank Dashboard** > nommez le tableau de bord **Container Request Overview** > cliquez sur **Create Dashboard**.
1. Sélectionnez **Top List** comme nouveau panneau et nommez-le **Request time per container**
   - Sous **Metrics**, entrez **net.http.request.time**.
   - Portée : cliquez sur **Override Dashboard Scope** > sélectionnez **container.image** > sélectionnez **is** > sélectionnez _the application image_
   - Segmentez par **container.id** et vous devriez voir le temps de requête net de chaque conteneur.
   - Cliquez sur **Save**.
1. Pour ajouter un nouveau panneau, cliquez sur l'icône **plus** et sélectionnez **Number(#)** comme type de panneau
   - Sous **Métriques**, entrez **net.http.request.count** > remplacez l'agrégation horaire **Moyenne** par **Somme**.
   - Portée : cliquez sur **Override Dashboard Scope** > sélectionnez **container.image** > sélectionnez **is** > sélectionnez _the application image_
   - Comparez à il y a **1 heure** et vous devriez voir le nombre de requêtes net de chaque conteneur.
   - Cliquez sur **Save**.
1. Pour éditer la portée de ce tableau de bord, 
   - Cliquez sur **Editer une portée** dans le panneau de titre.
   - Sélectionnez/entrez **Kubernetes.cluster.name** dans la liste déroulante.
   - Laissez le nom d'affichage vide et sélectionnez **is**.
   - Sélectionnez **mycluster** comme valeur et cliquez sur **Save**.

## Suppression de ressources 
{: #remove_resource}

- Supprimez les instances LogDNA et Sysdig de la page [Observabilité](https://{DomainName}/observe).
- Supprimez le cluster, y compris le noeud worker, l'application et les conteneurs. Cette action est irréversible.
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## Extension du tutoriel 
{: #expand_tutorial}

- Utilisez le [service {{site.data.keyword.at_full}}](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started) pour suivre la manière dont les applications interagissent avec les services IBM Cloud.
- [Ajoutez des alertes](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts) à votre vue.
- [Exportez les journaux ](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export) dans un fichier local.

## Contenu associé
{:related}
- [Réinitialisation de la clé d'ingestion utilisée par un cluster Kubernetes](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [Archivage de journaux dans le stockage d'objets IBM Cloud Object Storage](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [Configuration des alertes dans Sysdig](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [Utilisation de canaux de notification dans l'interface utilisateur Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
