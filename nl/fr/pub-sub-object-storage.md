---
copyright:
  years: 2018, 2019
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

# Traitement asynchrone des données à l'aide d'Object Storage et de la messagerie de publication/d'abonnement
{: #pub-sub-object-storage}
Dans ce tutoriel, vous allez apprendre à utiliser un service de messagerie basé sur Apache Kafka afin d'orchestrer des charges de travail longues pour des applications exécutées dans un cluster Kubernetes. Ce modèle est utilisé pour découpler votre application, permettant ainsi un meilleur contrôle de la mise à l'échelle et des performances. {{site.data.keyword.messagehub}} peut être utilisé pour mettre en file d'attente le travail à effectuer sans affecter les applications du producteur, ce qui en fait un système idéal pour les tâches de longue durée. 

{:shortdesc}

Vous allez simuler ce modèle à l'aide d'un exemple de traitement de fichier. Commencez par créer une application d'interface utilisateur qui vous permet de transférer des fichiers sur Object Storage et de générer des messages indiquant le travail à effectuer. Créez ensuite une application worker distincte qui traite de manière asynchrone les fichiers transférés par l'utilisateur lorsqu'elle reçoit les messages. 

## Objectifs
{: #objectives}

* Implémenter un modèle producteur-consommateur avec {{site.data.keyword.messagehub}}
* Lier des services à un cluster Kubernetes 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

Dans ce tutoriel, l'application d'interface utilisateur est écrite dans Node.js et l'application worker est écrite en Java, ce qui souligne la flexibilité de ce modèle. Même si les deux applications s'exécutent dans le même cluster Kubernetes dans ce tutoriel, l'une ou l'autre peut également avoir été implémentée en tant qu'application Cloud Foundry ou en tant que fonction sans serveur. 

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. L'utilisateur transfère un fichier à l'aide de l'application d'interface utilisateur. 
2. Le fichier est sauvegardé dans {{site.data.keyword.cos_full_notm}}
3. Le message est envoyé à la rubrique {{site.data.keyword.messagehub}} indiquant que le nouveau fichier est en attente de traitement.
4. Une fois prêtes, les applications worker écoutent les messages et commencent à traiter le nouveau fichier. 

## Avant de commencer
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - Outil pour installer {{site.data.keyword.cloud_notm}} CLI, Kubernetes, Helm et Docker.

## Création d'un cluster Kubernetes 
{: #create_kube_cluster}

1. Créez un cluster Kubernetes à partir du [catalogue](https://{DomainName}/containers-kubernetes/launch). Nommez-le `mycluster` pour pouvoir suivre facilement ce tutoriel. Ce tutoriel peut être réalisé avec un cluster gratuit (**Free**).
   ![Création d'un cluster Kubernetes sur IBM Cloud](images/solution25/KubernetesClusterCreation.png)
2. Vérifiez l'état du **Cluster** et du **Noeud worker** et patientez jusqu'à ce qu'ils soient prêts (**ready**).

### Configuration de kubectl

Dans cette étape, vous configurez kubectl pour qu'il pointe vers le nouveau cluster. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) est un outil de ligne de commande qui vous permet d'interagir avec un cluster Kubernetes.

1. Utilisez `ibmcloud login` pour vous connecter de manière interactive. Indiquez l'organisation (org), l'emplacement et l'espace sous lesquels le cluster est créé. Vous pouvez confirmer les détails en exécutant la commande `ibmcloud target`. 
2. Lorsque le cluster est prêt, extrayez la configuration du cluster :
   ```bash
   ibmcloud cs cluster-config <cluster-name>
   ```
   {: pre}
3. Copiez et collez la commande **export** afin de définir la variable d'environnement KUBECONFIG comme indiqué. Pour vérifier si la variable d'environnement KUBECONFIG est définie correctement, exécutez la commande suivante :
  `echo $KUBECONFIG`
4. Vérifiez que la commande `kubectl` est correctement configurée 
   ```bash
   kubectl cluster-info
   ```
  {: pre}
   ![](images/solution2/kubectl_cluster-info.png)

## Création d'une instance {{site.data.keyword.messagehub}}
 {: #create_messagehub}

{{site.data.keyword.messagehub}} est un service de messagerie rapide, évolutif et entièrement géré, basé sur Apache Kafka, un système de messagerie open source à haut débit qui propose une plateforme à faible temps de réponse pour la gestion des flux de données en temps réel.

 1. Dans le tableau de bord, cliquez sur [**Créer une ressource**](https://{DomainName}/catalog/) et sélectionnez [**{{site.data.keyword.messagehub}}**](https://{DomainName}/catalog/services/event-streams) dans la section Services d'application.
 2. Nommez le service `mymessagehub` et cliquez sur **Créer**.
 3. Fournissez les données d'identification du service à votre cluster en liant l'instance de service à l'espace de nom Kubernetes `par défaut`.
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

La commande cluster-service-bind crée un secret de cluster contenant les données d'identification de votre instance de service au format JSON. Utilisez `kubectl get secrets ` pour visualiser le secret généré avec le nom `binding-mymessagehub`. Pour plus d'informations, voir [Intégration de services](https://{DomainName}/docs/containers?topic=containers-integrations#integrations)

{:tip}

## Création d'une instance Object Storage

{: #create_cos}

{{site.data.keyword.cos_full_notm}} est chiffré et dispersé sur plusieurs emplacements géographiques et accessible via HTTP à l'aide d'une API REST. {{site.data.keyword.cos_full_notm}} fournit un stockage dans le cloud flexible, économique et évolutif pour les données non structurées. Vous allez l'utiliser pour stocker les fichiers téléchargés par l'interface utilisateur. 

1. Dans le tableau de bord, cliquez sur [**Créer une ressource**](https://{DomainName}/catalog/) et sélectionnez [**{{site.data.keyword.cos_short}}**](https://{DomainName}/catalog/services/cloud-object-storage) dans la section Stockage.
2. Nommez le service `myobjectstorage` et cliquez sur **Créer**.
3. Cliquez sur **Créer un compartiment**.
4. Définissez le compartiment sur un nom unique, tel que`username-mybucket`.
5. Sélectionnez la résilience **Inter-région** et l'emplacement **us-geo**, et cliquez sur **Créer**
6. Fournissez les données d'identification du service à votre cluster en liant l'instance de service à l'espace de nom Kubernetes `par défaut`.
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## Déploiement de l'application d'interface utilisateur sur le cluster

L'application d'interface utilisateur est une simple application Web Node.js Express qui permet à l'utilisateur de transférer des fichiers. Elle stocke les fichiers dans l'instance Object Storage créée ci-dessus, puis envoie un message à la rubrique "work-topic" de {{site.data.keyword.messagehub}} indiquant qu'un nouveau fichier est prêt à être traité. 

1. Clonez localement le référentiel d'applications exemple et accédez au dossier `pubsub-ui`.
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. Ouvrez `config.js` et mettez à jour COSBucketName avec le nom du compartiment.
3. Générez et déployez l'application. La commande deploy génère des images docker, les envoie au {{site.data.keyword.registryshort_notm}}, puis crée un déploiement Kubernetes. Suivez les instructions interactives lors du déploiement de l'application. 
```sh
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
4. Consultez l'application et téléchargez les fichiers à partir du dossier `sample-files`. Les fichiers téléchargés sont stockés dans Object Storage et le statut est "en attente" jusqu'à ce qu'ils soient traités par l'application worker. Conservez cette fenêtre de navigateur ouverte. 

   ![](images/solution25/files_uploaded.png)

## Déploiement de l'application worker sur le cluster

L'application worker est une application Java qui écoute les messages dans la rubrique "work-topic" Kafka {{site.data.keyword.messagehub}}. Pour un nouveau message, le worker extrait le nom du fichier dans le message, puis extrait le contenu du fichier à partir d'Object Storage. Il simule ensuite le traitement du fichier et envoie un autre message à la rubrique "result-work" au terme de l'exécution. L'application d'interface utilisateur écoute cette rubrique et met à jour le statut. 

1. Accédez au répertoire `pubsub-worker`
```sh
  cd ../pubsub-worker
```
2. Ouvrez `resources/cos.properties` et mettez à jour la propriété `bucket.name` en indiquant le nom de votre compartiment.
2. Générez et déployez l'application worker. 
```
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
3. Une fois le déploiement terminé, vérifiez à nouveau la fenêtre du navigateur avec votre application Web. Notez que le statut en regard de chaque fichier est maintenant remplacé par "traité".
![](images/solution25/files_processed.png)

Dans ce tutoriel, vous avez appris à utiliser {{site.data.keyword.messagehub}} basé sur Kafka pour implémenter un modèle producteur-consommateur. Cela permet à l'application Web d'être rapide et de décharger les traitements lourds vers d'autres applications. Lorsque des tâches doivent être effectuées, le producteur (application Web) crée des messages et la charge de travail est répartie entre un ou plusieurs workers abonnés aux messages. Vous avez utilisé une application Java exécutée sur Kubernetes pour gérer le traitement, mais ces applications peuvent également être [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing). Les applications exécutées sur Kubernetes sont idéales pour les charges de travail longues et intensives, alors que {{site.data.keyword.openwhisk_short}} est plus adapté aux processus de courte durée. 

## Suppression de ressources 
{:removeresources}

Accédez à la [Liste des ressources](https://{DomainName}/resources/) et 
1. supprimez le cluster `mycluster` de Kubernetes 
2. supprimez {{site.data.keyword.cos_full_notm}} `myobjectstorage`
3. supprimez {{site.data.keyword.messagehub}} `mymessagehub`
4. sélectionnez **Kubernetes** dans le menu de gauche, **Registry**, puis supprimez les référentiels `pubsub-xxx`.

## Contenu associé
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [Gestion de l'accès à Object Storage](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [Traitement des données {{site.data.keyword.messagehub}} avec {{site.data.keyword.openwhisk_short}}](https://github.com/IBM/openwhisk-data-processing-message-hub)
