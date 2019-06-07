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

# Application Web évolutive sur Kubernetes 
{: #scalable-webapp-kubernetes}

Ce tutoriel vous explique comment construire une application Web, l'exécuter localement dans un conteneur, puis la déployer dans un cluster Kubernetes créé avec [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster). Vous apprendrez également comment lier un domaine personnalisé, surveiller la santé de votre environnement et dimensionner l'application.
{:shortdesc}

Les conteneurs constituent un moyen standard de conditionner des applications et toutes leurs dépendances. Ainsi, vous pouvez déplacer en toute transparence des applications entre divers environnements. A la différence des machines virtuelles, les conteneurs n'incluent pas le système d'exploitation. Seuls le code d'application, l'environnement d'exécution, les outils système, les bibliothèques et les paramètres sont inclus dans les conteneurs. Les conteneurs sont plus légers, plus portables et plus efficaces que les machines virtuelles.

Pour les développeurs souhaitant lancer leurs projets, l'interface CLI d'{{site.data.keyword.dev_cli_notm}} permet un développement et un déploiement rapides des applications par la génération de modèles d'applications que vous pouvez exécuter immédiatement ou personnaliser en tant que démarreurs pour vos propres solutions. Outre la génération de code applicatif de démarrage, d'images de conteneur Docker et d'actifs CloudFoundry, les générateurs de code utilisés par la CLI de développement et la console Web génèrent des fichiers destinés à faciliter le déploiement dans les environnements [Kubernetes](https://kubernetes.io/). Les modèles génèrent des chartes [Helm](https://github.com/kubernetes/helm) qui décrivent la configuration de déploiement de Kubernetes initiale de l’application et peuvent aisément être étendues pour créer des déploiements multi-images ou complexes, selon les besoins. 

## Objectifs
{: #objectives}

* Construction d'une application de démarrage.
* Déploiement de l'application sur le cluster Kubernetes. 
* Liaison d'un domaine personnalisé. 
* Surveillance des journaux et de la santé du cluster. 
* Mise à l'échelle de pods Kubernetes.

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

<p style="text-align: center;">

  ![Architecture](images/solution2/Architecture.png)
</p>

1. Un développeur génère une application de démarrage avec {{site.data.keyword.dev_cli_notm}}.
1. La génération de l'application produit une image de conteneur Docker. 
1. L'image est transférée dans un espace de nom dans {{site.data.keyword.containershort_notm}}.
1. L'application est déployée sur un cluster Kubernetes.
1. Les utilisateurs accèdent à l'application. 

## Avant de commencer
{: #prereqs}

* [Configurez l'interface de ligne de commande {{site.data.keyword.registrylong_notm}} et votre espace de nom de registre](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [Installez {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - Script pour installer docker, kubectl, helm, ibmcloud cli et les plug-ins requis
* [Intégrez les concepts de base de Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## Création d'un cluster Kubernetes 
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} propose des outils puissants en combinant les technologies Docker et Kubernetes, une expérience utilisateur intuitive et une sécurité et un isolement intégrés pour automatiser le déploiement, l'exploitation, la mise à l'échelle et la surveillance d'applications conteneurisées dans un cluster d'hôtes de calcul.

La majeure partie de ce tutoriel peut être réalisée avec un cluster **gratuit**. Deux sections facultatives relatives à Kubernetes Ingress et au domaine personnalisé nécessitent un cluster **payant** de type **Standard**. 

1. Créez un cluster Kubernetes à partir du catalogue [{{site.data.keyword.Bluemix}}](https://{DomainName}/containers-kubernetes/launch). 

   Pour plus de facilité, vérifiez les détails de la configuration, tels que le nombre d'unités centrales, la mémoire et le nombre de noeuds worker dont vous disposez avec les forfaits Lite et Standard.
{:tip}

   ![Création de clusters Kubernetes sur IBM Cloud](images/solution2/KubernetesClusterCreation.png)
2. Sélectionnez le **Type de cluster** et cliquez sur **Créer un cluster** pour mettre à disposition un cluster Kubernetes. 
3.  Vérifiez l'état du **Cluster** et du **Noeud worker** et patientez jusqu'à ce qu'ils soient prêts (**ready**).

### Configuration de kubectl

Dans cette étape, vous configurez kubectl pour qu'il pointe vers le nouveau cluster. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) est un outil de ligne de commande qui vous permet d'interagir avec un cluster Kubernetes.

1. Utilisez `ibmcloud login` pour vous connecter de manière interactive. Indiquez l'organisation (org), l'emplacement et l'espace sous lesquels le cluster est créé. Vous pouvez confirmer les détails en exécutant la commande `ibmcloud target`. 
2. Lorsque le cluster est prêt, extrayez la configuration du cluster en définissant la variable d'environnement MYCLUSTER sur votre nom de cluster : 
   ```bash
   export MYCLUSTER=<CLUSTER_NAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
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


## Création d'une application de démarrage
{: #create_application}

Les outils `ibmcloud dev` réduisent considérablement le temps de développement en proposant des kits de démarrage contenant tout le code réutilisable de génération et de configuration nécessaire pour vous permettre de commencer à coder la logique métier plus rapidement. 

1. Démarrez l'assistant `ibmcloud dev`.
   ```
   ibmcloud dev create
   ```
   {: pre}

1. Sélectionnez `Service de back-end / Appli Web` > `Java - MicroProfile / JavaEE` > `Java Web App with Eclipse MicroProfile and Java EE (Web App)` pour créer une application de démarrage Java. (Pour créer un démarreur Node.js à la place, utilisez `Service de back-end / Appli Web` > `Noeud`> `Node.js Web App with Express.js (Web App)` )
1. Entrez un **nom** pour votre application.
1. Sélectionnez le groupe de ressources où déployer cette application. 
1. N'ajoutez pas d'autres services. 
1. N'ajouter pas de chaîne d’outils DevOps, sélectionnez **déploiement manuel**.

Cette action génère une application de démarrage avec le code et tous les fichiers de configuration nécessaires pour le développement local et le déploiement sur le cloud sur Cloud Foundry ou Kubernetes.  

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### Génération de l'application

Vous pouvez générer et exécuter l’application comme vous le feriez normalement avec `mvn` pour le développement local java ou `npm` pour le développement de noeud. Vous pouvez également générer une image Docker et exécuter l'application dans un conteneur pour garantir une exécution cohérente localement et sur le cloud. Utilisez les étapes suivantes pour générer l'image Docker.


1. Vérifiez que le moteur Docker local est démarré. 
   ```
   docker ps
   ```
   {: pre}
2. Accédez au répertoire de projet généré.
   ```
   cd <project name>
   ```
   {: pre}
3. Générez l'application.
   ```
   ibmcloud dev build
   ```
   {: pre}

   L'exécution peut prendre quelques minutes, car toutes les dépendances de l'application sont téléchargées et une image Docker contenant votre application et tout l'environnement requis est générée. 

### Exécution de l'application en local

1. Exécutez le conteneur.
   ```
   ibmcloud dev run
   ```
   {: pre}

   Cette commande utilise votre moteur Docker local pour exécuter l'image Docker que vous avez créée à l'étape précédente. 
2. Une fois le conteneur démarré, accédez à `http://localhost:9080/`. Si vous avez créé une application Node.js, accédez à `http://localhost:3000/`.
  ![](images/solution2/LibertyLocal.png)

## Déploiement de l'application sur le cluster à l'aide de la charte Helm 
{: #deploy}

Dans cette section, vous devez d'abord transférer l'image Docker dans le registre de conteneurs privés IBM Cloud, puis créer un déploiement Kubernetes pointant vers cette image. 

1. Recherchez **l'espace de nom** en répertoriant tous les espace de nom du registre.
   ```sh
   ibmcloud cr namespaces
   ```
   {: pre}
   Si vous disposez d'un espace de nom, notez son nom pour pouvoir l'utiliser ultérieurement. Si vous n'en avez pas, créez-le.
```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. Définissez les variables d'environnement MYNAMESPACE et MYPROJECT sur votre espace de nom et votre nom de projet, respectivement 

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. Identifiez **Container Registry** (par exemple, us.icr.io) en exécutant `ibmcloud cr info`
4. Définissez la variable d’environnement MYREGISTRY dans votre registre. 
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. Générez et étiquetez (`-t`) l'image Docker 
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. Transférez l'image Docker dans votre registre de conteneurs sur IBM Cloud 
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. Dans un environnement de développement intégré, accédez à **values.yaml** sous `chart\YOUR PROJECT NAME` et mettez à jour la valeur **image repository** pointant vers votre image dans le registre de conteneurs IBM Cloud. **Sauvegardez** le fichier.

   Pour plus de détails sur le référentiel d'images, exécutez la commande `echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}`

8. [Helm](https://helm.sh/) vous aide à gérer les applications Kubernetes via les Chartes Helm, qui aident à définir, installer et mettre à niveau y compris l'application Kubernetes la plus complexe. Initialisez Helm en accédant à `chart\YOUR PROJECT NAME` et en exécutant la commande ci-dessous dans votre cluster. 

   ```bash
   helm init
   ```
   {: pre}
   Pour mettre à jour helm, exécutez la commande `helm init --upgrade`
   {:tip}

9. Pour installer une charte Helm, accédez au répertoire `chart\YOUR PROJECT NAME` et exécutez la commande ci-dessous 
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. Utilisez `kubectl get service ${MYPROJECT}-service` pour votre application Java et `kubectl get service ${MYPROJECT}-application-service` pour votre application Node.js afin d'identifier le port public sur lequel le service est à l'écoute. Le port est un numéro à 5 chiffres (par exemple, 31569) sous `PORT(S)`.
11. Pour l'adresse IP publique du noeud worker, exécutez la commande ci-dessous 
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. Accédez à l'application à l'adresse `http://worker-ip-address:portnumber/`.

## Utilisation du domaine fourni par IBM pour votre cluster 
{: #ibm_domain}

Dans l'étape précédente, l'application était accessible via un port non standard. Le service a été exposé via la fonctionnalité Kubernetes NodePort.

Les clusters payants sont fournis avec un domaine fourni par IBM. Vous disposez donc d'une meilleure option pour exposer les applications avec une URL appropriée et sur des ports HTTP/S standard. 

Utilisez Ingress pour configurer la connexion entrante du cluster au service. 

![Ingress](images/solution2/Ingress.png)

1. Identifiez votre **domaine Ingress** fourni par IBM 
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   pour rechercher
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. Créez un fichier ingress `ingress-ibmdomain.yml` pointant vers votre domaine avec la prise en charge de HTTP et HTTPS. Utilisez le fichier suivant comme modèle en remplaçant toutes les valeurs encadrées par <> par les valeurs appropriées figurant ci-dessus.**service-name** est le nom situé sous `==> v1/Service` à l'étape ci-dessus ou exécutez `kubectl get svc` pour rechercher le nom de service de type **NodePort**.
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-ibmdomain-http-and-https
   spec:
     tls:
     - hosts:
       -  <nameofproject>.<ingress-sub-domain>
       secretName: <ingress-secret>
     rules:
     - host: <nameofproject>.<ingress-sub-domain>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: codeblock}
3. Déployez Ingress
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. Accédez à votre application à l'adresse `https://<nameofproject>.<ingress-sub-domain>/`

## Utilisation de votre domaine personnalisé 
{: #custom_domain}

Pour utiliser votre domaine personnalisé, vous devez mettre à jour vos enregistrements DNS avec un enregistrement CNAME pointant vers votre domaine fourni par IBM ou un enregistrement A pointant vers l'adresse IP publique portable de l'Ingress fourni par IBM. Dans le mesure où un cluster payant est livré avec des adresses IP fixes, un enregistrement A est une bonne option. 

Pour plus d'informations, consultez [Utilisation du contrôleur Ingress avec un domaine personnalisé](https://{DomainName}/docs/containers?topic=containers-ingress#ingress).

### avec HTTP 

1. Créez un fichier Ingres `ingress-customdomain-http.yml` pointant vers votre domaine :
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-customdomain-http
   spec:
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
2. Déployez Ingress
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. Accédez à votre application à l'adresse `http://<customdomain>/` 

### Avec HTTPS

Actuellement, si vous envisagez d'accéder à votre application avec HTTPS, `https://<customdomain>/`, vous recevrez probablement un avertissement de sécurité de votre navigateur Web vous indiquant que la connexion n'est pas privée. Vous obtiendrez également une erreur 404 car l'Ingress que vous venez de configurer ignore comment diriger le trafic HTTPS. 

1. Obtenez un certificat SSL approuvé pour votre domaine. Vous avez besoin du certificat et de la clé : 
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3
   Vous pouvez utiliser [Let's Encrypt](https://letsencrypt.org/) pour générer un certificat sécurisé.
2. Enregistrez le certificat et la clé dans des fichiers au format base64 ascii. 
3. Créez un secret TLS pour stocker le certificat et la clé :
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<custom-domain.cert> --key=<custom-domain.key>
   ```
   {: pre}
4. Créez un fichier Ingress `ingress-customdomain-https.yml` pointant vers votre domaine :
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-customdomain-https
   spec:
     tls:
     - hosts:
       - <my-custom-domain.com>
       secretName: <my-custom-domain-secret-name>
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
5. Déployez Ingress : 
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. Accédez à votre application à l'adresse `https://<customdomain>/`. 

## Surveillance de la santé de l'application 
{: #monitor_application}

1. Pour vérifier la santé de votre application, accédez aux [clusters](https://{DomainName}/containers-kubernetes/clusters) pour afficher une liste de clusters, puis cliquez sur le cluster que vous avez créé ci-dessus. 
2. Cliquez sur **Tableau de bord Kubernetes** pour lancer le tableau de bord dans un nouvel onglet.
   ![](images/solution2/launch_kubernetes_dashboard.png)
3. Sélectionnez des **Noeuds** dans le volet de gauche, cliquez sur le **Nom** des noeuds et consultez les **Allocation Resources** pour visualiser la santé des noeuds.
   ![](images/solution2/KubernetesDashboard.png)
4. Pour examiner les journaux d'application du conteneur, sélectionnez **Pods**, **pod-name** et **Journaux**.
5. Pour vous connecter via **ssh** dans le conteneur, identifiez le nom de votre pod de l'étape précédente et exécutez 
   ```sh
   kubectl exec -it <pod-name> -- bash
   ```
   {: pre}

## Mise à l'échelle de pods Kubernetes
{: #scale_cluster}

Lorsque la charge de votre application augmente, vous pouvez augmenter manuellement le nombre de répliques de pod dans votre déploiement. Les répliques sont gérées par un [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/). Pour redimensionner l'application à deux répliques, exécutez la commande suivante : 

   ```sh
 kubectl scale deployment <nameofproject>-deployment --replicas=2
   ```
   {: pre}

Après quelques instants, deux pods apparaissent pour votre application dans le tableau de bord Kubernetes (ou avec `kubectl get pods`). Le contrôleur Ingress du cluster gère l’équilibrage de la charge entre les deux répliques. La mise à l'échelle horizontale peut également être automatique. 

Pour plus d'informations sur la mise à l'échelle manuelle et automatique, reportez-vous à la documentation de Kubernetes : 

   * [Mise à l'échelle d'un déploiement](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [Mise à l'échelle automatique de pods horizontaux](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## Suppression de ressources 

* Supprimez le cluster ou ne supprimez que les artefacts Kubernetes créés pour l'application si vous envisagez de réutiliser le cluster. 

## Contenu associé

* [IBM Cloud Kubernetes Service](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [Déploiement continu sur Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
