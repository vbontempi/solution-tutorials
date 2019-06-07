---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-15"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Déplacement vers Kubernetes d'une application basée sur une MV  
{: #vm-to-containers-and-kubernetes}

Ce tutoriel vous guide tout au long du processus de déplacement d'une application basée sur une machine virtuelle (MV) vers un cluster Kubernetes à l'aide d'{{site.data.keyword.containershort_notm}}. [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) propose des outils puissants en combinant les technologies Docker et Kubernetes, une expérience utilisateur intuitive ainsi que des fonctionnalités intégrées de sécurité et d'isolement pour automatiser le déploiement, l'exploitation, la mise à l'échelle et la surveillance d'applications conteneurisées dans un cluster d'hôtes de calcul.
{: shortdesc}

Les leçons de ce tutoriel incluent des concepts sur la manière d'utiliser une application existante, de la conteneuriser et de la déployer dans un cluster Kubernetes. Pour conteneuriser votre application basée sur une MV, vous pouvez choisir parmi les options suivantes. 

1. Identifiez les composants d'une grande application monolithique pouvant être séparés en microservices. Vous pouvez conteneuriser ces microservices et les déployer dans un cluster Kubernetes. 
2. Conteneurisez l'application complète et déployez-la dans un cluster Kubernetes. 

La procédure de migration varie selon le type d'application. Vous pouvez utiliser ce tutoriel pour en savoir plus sur les étapes générales à suivre et les éléments à prendre en compte avant de migrer votre application. 

## Objectifs
{: #objectives}

- Comprendre comment identifier des microservices dans une application basée sur une MV et apprendre à mapper des composants entre des MV et Kubernetes. 
- Apprendre à conteneuriser une application basée sur une MV. 
- Découvrir comment déployer le conteneur dans un cluster Kubernetes dans {{site.data.keyword.containershort_notm}}.
- Mettre en pratique toutes vos nouvelles connaissances et lancer l'application **JPetStore** dans votre cluster. 

## Services utilisés
{: #products}

Ce tutoriel utilise les services {{site.data.keyword.Bluemix_notm}} suivants : 

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**Attention :** ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{:#architecture}

### Architecture d'application traditionnelle avec MV 

Le diagramme suivant montre un exemple d'architecture d'application traditionnelle basée sur des machines virtuelles. 

<p style="text-align: center;">
![Diagramme d'architecture](images/solution30/traditional_architecture.png)
</p>

1. L'utilisateur envoie une demande au noeud final public de l'application Java. Le noeud final public est représenté par un service d'équilibrage de la charge qui équilibre le trafic réseau entrant entre les instances de serveur d'applications disponibles. 
2. L'équilibreur de charge sélectionne l'une des instances de serveur d'applications saines qui s'exécutent sur une MV et transmet la demande. 
3. Le serveur d'applications stocke les données d'applications dans une base de données MySQL qui s'exécute sur une MV. Les instances de serveur d'applications hébergent l'application Java et s'exécutent sur une MV. Les fichiers d'application, tels que le code applicatif, les fichiers de configuration et les dépendances sont stockés sur la MV. 

### Architecture conteneurisée 

Le diagramme suivant montre un exemple d'architecture de conteneur moderne qui s'exécute dans un cluster Kubernetes. 

<p style="text-align: center;">
![Diagramme d'architecture](images/solution30/modern_architecture.png)
</p>

1. L'utilisateur envoie une demande au noeud final public de l'application Java. Le noeud final public est représenté par un équilibreur de charge d'application (Application Load Balancer - ALB) Ingress qui équilibre la charge du trafic réseau entrant entre les pods d'application dans le cluster. L'ALB est un ensemble de règles permettant le trafic réseau entrant vers une application exposée publiquement. 
2. L'ALB transmet la demande à l'un des pods d'application disponibles dans le cluster. Les pods d'application s'exécutent sur des noeuds worker pouvant être une machine virtuelle ou physique. 
3. Les pods d'application stockent les données dans des volumes persistants. Les volumes persistants peuvent être utilisés pour partager des données entre des instances d'application ou des noeuds worker. 
4. Les pods d'application stockent les données dans un service de base de données {{site.data.keyword.Bluemix_notm}}. Vous pouvez exécuter votre propre base de données dans le cluster Kubernetes, mais une base de données gérée en tant que service (DBasS) est généralement plus facile à configurer et offre des sauvegardes et une mise à l'échelle intégrées. Vous pouvez trouver de nombreux types de bases de données différents dans le [catalogue d'IBM Cloud](https://{DomainName}/catalog/?category=data).

### MV, conteneurs et Kubernetes 

{{site.data.keyword.containershort_notm}} permet d'exécuter des applications conteneurisées dans des clusters Kubernetes et fournit les outils et les fonctions suivants :

- Expérience utilisateur intuitive et outils puissants 
- Sécurité et isolement intégrés pour permettre la livraison rapide d'applications sécurisées 
- Services cloud intégrant les fonctionnalités cognitives issues d'IBM® Watson™ 
- Capacité à gérer des ressources de cluster dédiées pour les applications sans état et les charges de travail avec état 

#### Machines virtuelles et conteneurs 

Les applications traditionnelles (**MV**) fonctionnent sur un équipement natif. Une application n'utilise généralement pas toutes les ressources d'un hôte de calcul. La plupart des entreprises essaient d'exécuter plusieurs applications sur un seul hôte de calcul afin d'éviter de gaspiller des ressources. Vous pouvez exécuter plusieurs copies d'une même application, mais pour obtenir un isolement, vous pouvez utiliser des MV pour exécuter plusieurs instances d'application (MV) sur le même équipement. Ces MV possèdent des piles de système d'exploitation complètes qui les rendent relativement volumineuses et inefficaces en raison de la duplication tant au moment de l'exécution que sur le disque. 

Les **conteneurs** constituent un moyen standard de conditionner des applications et toutes leurs dépendances. Ainsi, vous pouvez déplacer en toute transparence des applications entre divers environnements. A la différence des machines virtuelles, les conteneurs n'incluent pas le système d'exploitation. Seuls le code d'application, l'environnement d'exécution, les outils système, les bibliothèques et les paramètres sont inclus dans les conteneurs. Les conteneurs sont plus légers, plus portables et plus efficaces que les machines virtuelles.

De plus, les conteneurs vous permettent de partager le système d'exploitation de l'hôte. Vous réduisez ainsi les doublons tout en conservant l'isolement. Les conteneurs vous permettent également de supprimer des fichiers inutiles, tels que les bibliothèques système et les fichiers binaires, afin d'économiser de l'espace et de réduire la surface d'attaque. Pour en savoir plus sur les machines virtuelles et les conteneurs, cliquez [ici](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html).

#### Orchestration Kubernetes 

[Kubernetes](http://kubernetes.io/) est un orchestrateur de conteneur permettant de gérer le cycle de vie des applications conteneurisées dans un cluster de noeuds worker. Vos applications peuvent nécessiter de nombreuses autres ressources pour s'exécuter, telles que des volumes, des réseaux et des secrets qui vous aideront à vous connecter à d'autres services cloud, ainsi que des clés sécurisées. Kubernetes vous aide à ajouter ces ressources à votre application. Le paradigme essentiel de Kubernetes est son modèle déclaratif. L'utilisateur indique l'état souhaité et Kubernetes tente de s'y conformer, puis conserve l'état décrit. 

Cet [atelier à votre rythme](https://github.com/IBM/kube101/blob/master/workshop/README.md) peut vous aider à acquérir votre première expérience pratique de Kubernetes. En outre, consultez la page de documentation sur les [concepts](https://kubernetes.io/docs/concepts/) Kubernetes pour en savoir plus sur les concepts de Kubernetes. 

### Avantages IBM

En utilisant les clusters Kubernetes avec {{site.data.keyword.containerlong_notm}}, vous bénéficiez des avantages suivants : 

- Plusieurs centres de données où vous pouvez déployer vos clusters. 
- Prise en charge des options réseau d'intrusion et d'équilibrage de charge. 
- Prise en charge des volumes persistants dynamiques. 
- Serveurs maîtres Kubernetes gérés par IBM, à haute disponibilité. 

## Dimensionnement des clusters
{: #sizing_clusters}

Lorsque vous concevez votre architecture de cluster, vous souhaitez équilibrer les coûts avec la disponibilité, la fiabilité, la complexité et les capacités de reprise. Les clusters Kubernetes dans {{site.data.keyword.containerlong_notm}} fournissent des options d'architecture basées sur les besoins de vos applications. Avec un minimum de planification, vous pouvez tirer le meilleur parti de vos ressources cloud sans architecture ni coûts excessifs. Même si vous surestimez ou sous-estimez vos besoins, vous pouvez aisément agrandir ou réduire le cluster soit avec des noeuds worker supplémentaires, soit avec des noeuds worker plus grands. 

Pour exécuter une application de production dans le cloud à l'aide de Kubernetes, posez-vous les questions suivantes : 

1. Prévoyez-vous du trafic à partir d'un emplacement géographique spécifique ? Dans l'affirmative, sélectionnez l'emplacement le plus proche de vous pour obtenir des performances optimales. 
2. Combien de répliques de cluster souhaitez-vous pour obtenir une disponibilité accrue ? Vous pourriez commencer avec trois clusters : un pour le développement, un pour les tests et un pour la production. Consultez le guide de solution [Bonnes pratiques en matière d'organisation des utilisateurs, des équipes et des applications](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments) pour la création d'environnements multiples. 
3. Quel est [l'équipement](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes) nécessaire pour les noeuds worker ? Machines virtuelles ou bare metal ?
4. Combien de noeuds worker sont-ils nécessaires ? Cela dépend fortement de l'échelle des applications, plus vous disposez de noeuds, plus votre application sera résistante. 
5. Combien de répliques sont-elles nécessaires pour une disponibilité plus élevée ? Déployez des clusters de répliques dans plusieurs emplacements afin de rendre votre application plus disponible et de la protéger des pannes dues à une défaillance sur un emplacement donné. 
6. Quel est l'ensemble minimal de ressources nécessaire au démarrage de l'application ? Vous pouvez tester votre application pour déterminer la quantité de mémoire et d'unité centrale requise pour son exécution. Le noeud worker devrait alors disposer de suffisamment de ressources pour déployer et démarrer l'application. Assurez-vous ensuite de définir des quotas de ressources dans les spécifications du pod. Ce paramètre est utilisé par Kubernetes pour sélectionner (ou planifier) un noeud worker doté d'une capacité suffisante pour prendre en charge la demande. Estimez le nombre de pods qui s'exécuteront sur le noeud worker et les ressources nécessaires pour ces pods. Au minimum, le noeud worker doit être suffisant pour prendre en charge un pod pour l'application. 
7. A quel moment augmenter le nombre de noeuds worker ? Vous pouvez surveiller l'utilisation du cluster et ajouter des noeuds si nécessaire. Consultez le présent tutoriel pour comprendre comment [analyser les journaux et surveiller la santé des applications Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana). 
8. Un stockage redondant et fiable est-il nécessaire ? Dans l'affirmative, créez une réservation de volume persistant pour le stockage NFS ou liez un service de base de données IBM Cloud au pod. 

Pour illustrer concrètement les informations ci-dessus, supposons que vous souhaitiez exécuter une application Web de production dans le cloud et que vous prévoyiez une charge de trafic moyenne à élevée. Examinons les ressources dont vous auriez besoin : 

1. Configurez trois clusters : un pour le développement, un pour les tests et un pour la production. 
2. Les clusters de développement et de test peuvent commencer avec une option de RAM et d'unité centrale minimale (par exemple, 2 UC , 4 Go de RAM et un noeud worker pour chaque cluster). 
3. Pour le cluster de production, vous pouvez disposer de davantage de ressources à des fins de performances, de haute disponibilité et de résilience. Vous pouvez alors choisir une option dédiée ou même une option bare metal et disposer d’au moins 4 UC, 16 Go de RAM et deux noeuds worker. 

## Choix de l'option de base de données à utiliser 
{: #database_options}

Avec Kubernetes, vous disposez de deux options pour gérer les bases de données : 

1. Vous pouvez exécuter votre base de données dans le cluster Kubernetes. Pour ce faire, vous devez créer un microservice pour exécuter la base de données. Si vous utilisez un exemple de base de données MySQL, procédez comme suit : 
   - Créez un fichier Docker MySQL ; consultez un exemple de fichier [MySQL Dockerfile ici](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile).
   - Vous devez utiliser des secrets pour stocker les données d'identification de la base de données. Consultez un exemple [ici](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret).
   - Un fichier deployment.yaml contenant la configuration de votre base de données est nécessaire pour effectuer le déploiement sur Kubernetes. Consultez un exemple [ici](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml).
2. La deuxième option consiste à utiliser l'option de base de données gérée en tant que service (DBasS). Cette option est généralement plus facile à configurer et fournit des sauvegardes et une mise à l'échelle intégrées. Vous pouvez trouver de nombreux types de bases de données différents dans le [catalogue d'IBM Cloud](https://{DomainName}/catalog/?category=data).Pour utiliser cette option, procédez comme suit : 
   - Créez une base de données gérée en tant que service (DBasS) à partir du [catalogue IBM Cloud](https://{DomainName}/catalog/?category=data). 
   - Stockez les données d'identification de la base de données dans un secret. Vous trouverez plus d'informations sur les secrets dans la section "Stockage des données d'identification dans les secrets Kubernetes". 
   - Utilisez la base de données en tant que service (DBasS) dans votre application. 

## Choix de l'emplacement de stockage des fichiers d'application 
{: #decide_where_to_store_data}

{{site.data.keyword.containershort_notm}} fournit plusieurs options pour stocker et partager des données sur les pods. Toutes les options de stockage n'offrent pas le même niveau de persistance et de disponibilité en cas d'incident.
{: shortdesc}

### Stockage de données non persistant 

De par leur conception, les conteneurs ont une durée de vie brève et ne sont pas à l'abri de défaillances inattendues. Vous pouvez stocker des données dans le système de fichiers local d'un conteneur. Les données hébergées au sein d'un conteneur ne peuvent pas être partagées avec d'autres conteneurs ou pods et sont perdues en cas de panne ou de suppression du conteneur.

### Création d'un stockage de données persistant pour votre application 

Vous pouvez conserver des données d'application et des données de conteneur sur le [stockage de fichiers NFS](https://www.ibm.com/cloud/file-storage/details) ou [le stockage par blocs](https://www.ibm.com/cloud/block-storage) en utilisant des volumes persistants Kubernetes natifs.
{: shortdesc}

Pour mettre à disposition le stockage de fichiers NFS ou le stockage par blocs, vous devez demander le stockage de votre pod en créant une réservation de volume persistant (Persistent Volume Claim - PVC). Dans la PVC, vous pouvez choisir parmi des classes de stockage prédéfinies qui définissent le type de stockage, la taille du stockage en gigaoctets, les opérations d'entrée-sortie par seconde (IOPS), la stratégie de conservation des données ainsi que les droits de lecture et d'écriture sur le stockage. Une PVC met à disposition de manière dynamique un volume persistant (VP) qui représente une unité de stockage réel dans {{site.data.keyword.Bluemix_notm}}. Vous pouvez monter la PVC sur votre pod pour lire et écrire sur le VP. Les données stockées dans les VP sont disponibles, même si le conteneur se bloque ou si le pod est replanifié. Le stockage de fichiers NFS et le stockage par blocs sur lesquels s'appuie le volume persistant sont mis en cluster par IBM pour assurer la haute disponibilité de vos données.

Pour apprendre à créer une PVC, effectuez les étapes décrites dans la [documentation sur le stockage d'{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage).

### Déplacement de données existantes vers un stockage persistant 

Pour copier des données de votre machine locale vers le stockage persistant, vous devez monter la PVC sur un pod. Ensuite, vous pouvez copier les données de votre machine locals vers le volume persistant de votre pod.
{: shortdesc}

1. Pour copier des données, vous devez d’abord créer une configuration similaire à celle-ci : 

   ```bash
   kind: Pod
   apiVersion: v1
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
          claimName: mypvc
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/mnt/data"
             name: task-pv-storage
   ```
   {: codeblock}

2. Ensuite, pour copier les données de votre machine locale vers le pod, utilisez une commande similaire à celle-ci : 
   ```
    kubectl cp <local_filepath>/<filename> <namespace>/<pod>:<pod_filepath>
   ```
   {: pre}

3. Copiez les données d’un pod de votre cluster vers votre machine locale : 
   ```
   kubectl cp <namespace>/<pod>:<pod_filepath>/<filename> <local_filepath>/<filename>
   ```
   {: pre}


### Configuration de sauvegardes du stockage persistant 

Les partages de fichiers et le stockage par blocs sont mis à disposition au même emplacement que votre cluster. Le stockage même est hébergé sur des serveurs en cluster par IBM pour assurer une haute disponibilité. Cependant, les partages de fichiers et le stockage par blocs ne sont pas sauvegardés automatiquement et risquent d'être inaccessibles en cas de défaillance de l'emplacement global. Pour éviter que vos données ne soient perdues ou endommagées, vous pouvez configurer des sauvegardes périodiques que vous pouvez utiliser pour restaurer vos données en cas de besoin. 

Pour plus d'informations, consultez les options de [sauvegarde et restauration](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) relatives au stockage de fichiers NFS et au stockage par blocs.

## Préparation du code
{: #prepare_code}

### Application des principes des 12 facteurs 

L'[application à douze facteurs](https://12factor.net/) est une méthodologie permettant de générer des applications natives dans le cloud. Lorsque vous souhaitez conteneuriser une application, la déplacer vers le cloud et l'orchestrer avec Kubernetes, il est important de comprendre et d'appliquer certains de ces principes. Certains de ces principes sont nécessaires dans {{site.data.keyword.Bluemix_notm}}.
{: shortdesc}

Voici quelques-uns des principes clés requis : 

- **Base de code** - Tous les fichiers de code source et de configuration sont suivis dans un système de contrôle de version (par exemple, un référentiel GIT), ce qui est nécessaire si vous utilisez le pipeline DevOps pour le déploiement. 
- **Générer, publier, exécuter** - L'application à 12 facteurs utilise une séparation stricte entre les étapes de génération, de publication et d'exécution. Cette opération peut être automatisée avec un pipeline de distribution DevOps intégré pour générer et tester l'application avant de la déployer sur le cluster. Consultez le [tutoriel Déploiement continu sur Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) pour apprendre à configurer une intégration continue et un pipeline de distribution. Ce document traite de la configuration des étapes de contrôle de la source, de génération, de test et de déploiement et vous explique comment ajouter des intégrations telles que des scanners de sécurité, des notifications et des analyses. 
- **Config** - Toutes les informations de configuration sont stockées dans des variables d'environnement. Aucune donnée d'identification de service n'est codée dans le code de l'application. Pour stocker les données d'identification, vous pouvez utiliser les secrets Kubernetes. Vous trouverez ci-après davantage d'informations sur les données d'identification. 

### Stockage des données d'identification dans les secrets Kubernetes
{: secrets}

Ce n'est jamais une bonne pratique de stocker des données d'identification dans le code de l'application. Pour éviter cela, Kubernetes fournit des **["secrets"](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)** qui contiennent des informations sensibles, telles que les mots de passe, les jetons OAuth ou les clés ssh. Les secrets Kubernetes sont chiffrés par défaut ; il s'agit donc d'une option plus sûre et plus flexible pour stocker des données sensibles que le stockage de ces données en l'état dans une définition de `pod` ou dans une image Docker. 

Pour utiliser les secrets dans Kubernetes, vous pouvez par exemple procéder comme suit : 

1. Créez un fichier et stockez les données d'identification du service qu'il contient. 
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api_key>
   }
   ```
   {: codeblock}

2. Créez ensuite un secret Kubernetes en exécutant une commande ci-dessous et vérifiez que le secret est créé à l'aide de `kubectl get secrets` après avoir exécuté la commande ci-dessous : 

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## Conteneurisation de votre application 
{: #build_docker_images}

Pour conteneuriser votre application, vous devez créer une image Docker.
{: shortdesc}

Une image est créée à partir d'un [fichier Docker](https://docs.docker.com/engine/reference/builder/#usage), à savoir un fichier contenant des instructions et des commandes qui permettent de générer l'image. Un fichier Docker peut référencer dans ses
instructions des artefacts de génération stockés séparément, comme une application, sa configuration, et ses dépendances.

Pour créer votre propre fichier Docker pour votre application existante, vous pouvez utiliser les commandes suivantes : 

- FROM - Choisissez une image parent pour définir l'exécution du conteneur.
- ADD/COPY - Copiez le contenu d'un répertoire dans le conteneur.
- WORKDIR - Définissez le répertoire de travail à l'intérieur du conteneur.
- RUN - Installez les packages logiciels dont les applications ont besoin pendant l'exécution.
- EXPOSE - Créez un port disponible à l'extérieur du conteneur.
- ENV NAME - Définissez les variables d’environnement.
- CMD - Définissez les commandes à exécuter lors du lancement du conteneur.

Les images sont généralement stockées dans un registre qui peut être accessible au public (registre public) ou configuré de sorte à limiter l'accès à un petit groupe d'utilisateurs (registre privé). Des registres publics, tel que Docker Hub, peuvent être utilisés pour vous familiariser avec Docker et Kubernetes pour créer votre première application conteneurisée dans un cluster. Toutefois, en ce qui concerne les applications d'entreprise, utilisez un registre privé, comme celui fourni dans {{site.data.keyword.registrylong_notm}}, pour protéger vos images contre l'utilisation et la modification par des utilisateurs non autorisés. 

Pour conteneuriser une application et la stocker dans {{site.data.keyword.registrylong_notm}} :

1. Vous devez créer un fichier Docker. Vous trouverez ci-dessous un exemple de fichier Docker. 
   ```
   # Build JPetStore war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # Use WebSphere Liberty base image from the Docker Store
   FROM websphere-liberty:latest

   # Copy war from build stage and server.xml into image
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. Une fois le fichier Docker créé, vous devez générer l’image de conteneur et la transmettre à {{site.data.keyword.registrylong_notm}}. Vous pouvez générer un conteneur à l'aide d'une commande semblable à : 
   ```
   ibmcloud cr build -t <image_name> <directory_of_Dockerfile>
   ```
   {: pre}

## Déploiement de votre application sur un cluster Kubernetes 
{: #deploy_to_kubernetes}

Une fois qu'une image de conteneur est générée et transférée dans le cloud, vous devez effectuer un déploiement sur votre cluster Kubernetes. Pour ce faire, vous devez créer un fichier deployment.yaml.
{: shortdesc}

### Création d'un fichier Kubernetes deployment.yaml 

Pour créer des fichiers Kubernetes deployment.yaml, vous devez procéder de la manière suivante : 

1. Créez un fichier deployment.yaml. Voici un exemple de fichier [YAML de déploiement](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml).

2. Dans votre fichier deployment.yaml, vous pouvez définir des [quotas de ressources](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) pour vos conteneurs afin de spécifier la quantité d'unité centrale et de mémoire nécessaire à chaque conteneur pour démarrer correctement. Si des quotas de ressources sont spécifiés dans les conteneurs, le planificateur Kubernetes peut prendre de meilleures décisions concernant le noeud worker sur lequel vos pods doivent être placés. 

3. Ensuite, vous pouvez utiliser les commandes ci-dessous pour créer et afficher le déploiement et les services créés : 

   ```
   kubectl create -f <filepath/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## Récapitulatif
{: #summary}

Dans ce tutoriel, vous avez appris : 

- Les différences entre les MV, les conteneurs et Kubernetes. 
- Comment définir des clusters pour différents types d’environnement (développement, test et production). 
- Comment gérer le stockage de données et l'importance du stockage de données persistant. 
- Comment appliquer les principes des 12 facteurs à votre application et utiliser des secrets pour les données d'identification dans Kubernetes. 
- Comment générer des images Docker et les transmettre à {{site.data.keyword.registrylong_notm}}.
- Comment créer des fichiers de déploiement Kubernetes et déployer l'image Docker sur Kubernetes. 

## Mise en pratique de toutes vos nouvelles connaissances et exécution de l'application JPetStore dans votre cluster. 
{: #runthejpetstore}

Pour mettre en pratique tout ce que vous avez appris, suivez la [démonstration](https://github.com/ibm-cloud/ModernizeDemo/) pour exécuter l'application **JPetStore** sur votre cluster et appliquez les concepts appris.  L'application JPetStore possède des fonctionnalités enrichies vous permettant d'étendre une application dans Kubernetes avec les services IBM Watson s'exécutant en tant que microservice distinct. 

## Contenu associé
{: #related}

- [Mise en route](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/) avec Kubernetes et {{site.data.keyword.containershort_notm}}.
- Laboratoires {{site.data.keyword.containershort_notm}} sur [GitHub](https://github.com/IBM/container-service-getting-started-wt).
- [Documents](http://kubernetes.io/) Kubernetes principaux.
- [Stockage persistant](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) dans {{site.data.keyword.containershort_notm}}.
- [Guide de solutions des bonnes pratiques](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications) pour l'organisation des utilisateurs, des équipes et des applications.
- [Analyse des journaux et surveillance de la santé des applications avec LogDNA et Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Configurez l'[intégration continue et un pipeline de distribution](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) pour les applications conteneurisées exécutées dans Kubernetes.
- Déploiement du cluster de production [sur plusieurs emplacements](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Utilisation de [plusieurs clusters sur plusieurs emplacements](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations) pour une haute disponibilité. 
