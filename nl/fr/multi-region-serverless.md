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

# Déploiement d'applications sans serveur dans plusieurs régions
{: #multi-region-serverless}

Ce tutoriel explique comment configurer IBM Cloud Internet Services et {{site.data.keyword.openwhisk_short}} pour déployer des applications sans serveur dans plusieurs régions. 

Les plateformes informatiques sans serveur permettent aux développeurs de créer rapidement des API sans utiliser de serveurs. {{site.data.keyword.openwhisk}} prend en charge la génération automatique d'API REST pour les actions, la conversion d'actions en noeuds finaux HTTP et la possibilité d'activer l'authentification d'API sécurisée. Cette fonctionnalité est utile non seulement pour exposer les API à des consommateurs externes, mais également pour créer des applications de microservices. 

{{site.data.keyword.openwhisk_short}} est disponible dans plusieurs emplacements {{site.data.keyword.cloud_notm}}. Pour augmenter la résilience et réduire le temps de réponse du réseau, les applications peuvent déployer leur serveur de back-end dans plusieurs emplacements. Ensuite, avec IBM Cloud Internet Services (CIS), les développeurs peuvent exposer un point d’entrée unique, chargé de la répartition du trafic vers le serveur de back-end sain le plus proche. 

## Objectifs
{: #objectives}

* Déployer des actions {{site.data.keyword.openwhisk_short}}.
* Exposer des actions via {{site.data.keyword.APIM}} avec un domaine personnalisé.
* Répartir le trafic sur plusieurs emplacements avec Cloud Internet Services. 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

Ce tutoriel utilise une application Web publique dotée d'un back-end implémenté avec {{site.data.keyword.openwhisk_short}}. Pour réduire le temps d'attente du réseau et éviter les pannes, l'application est déployée dans plusieurs emplacements. Deux emplacements sont configurés dans le tutoriel. 

<p style="text-align: center;">

  ![Architecture](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. Les utilisateurs accèdent à l'application. La demande passe par Internet Services. 
2.  Internet Services redirige les utilisateurs vers le serveur de back-end d'API sain le plus proche. 
3. {{site.data.keyword.cloudcerts_short}} fournit à l'API son certificat SSL. Le trafic est chiffré de bout en bout. 
4. L'API est implémentée avec {{site.data.keyword.openwhisk_short}}. 

## Avant de commencer
{: #prereqs}

1. Cloud Internet Services nécessite de posséder un domaine personnalisé pour pouvoir configurer le DNS de ce domaine et le diriger vers les serveurs de noms de Cloud Internet Services.Si vous ne possédez pas de domaine, vous pouvez en acheter un auprès d'un registraire tel que [godaddy.com](http://godaddy.com).
1. Installez tous les outils de ligne de commande nécessaires [en procédant comme suit](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).

## Configuration d'un domaine personnalisé

La première étape consiste à créer une instance IBM Cloud Internet Services (CIS) et à faire pointer votre domaine personnalisé vers les serveurs de nom CIS. 

1. Accédez à [Internet Services](https://{DomainName}/catalog/services/internet-services) dans le catalogue {{site.data.keyword.Bluemix_notm}}.
1. Définissez le nom du service, puis cliquez sur **Créer** pour créer une instance du service. Vous pouvez utiliser n’importe quel forfait pour ce tutoriel. 
1. Lorsque l'instance de service est mise à disposition, définissez votre nom de domaine en cliquant sur **Commençons !** et sur **Ajouter un domaine**. 
1. Cliquez sur **Etape suivante**. Lorsque les serveurs de noms sont attribués, configurez votre registraire ou votre fournisseur de nom de domaine pour utiliser les serveurs de noms répertoriés. 
1. Une fois que vous avez configuré votre registraire ou le fournisseur DNS, les modifications peuvent nécessiter jusqu'à 24 heures pour prendre effet. 

   Lorsque le statut du domaine sur la page Présentation passe de *En attente* à *Actif*, vous pouvez utiliser la commande `dig <your_domain_name> ns` pour vérifier que les nouveaux serveurs de nom ont pris effet.
   {:tip}

### Obtention d'un certificat pour le domaine personnalisé 

L'exposition d'actions {{site.data.keyword.openwhisk_short}} via un domaine personnalisé nécessite une connexion HTTPS sécurisée. Vous devez obtenir un certificat SSL pour le domaine et le sous-domaine que vous prévoyez d'utiliser avec le back-end sans serveur. En supposant un domaine *mydomain.com*, les actions peuvent être hébergées sur *api.mydomain.com*. Le certificat doit être émis pour *api.mydomain.com*.

Vous pouvez obtenir des certificats SSL gratuits auprès de [Let's Encrypt](https://letsencrypt.org/). Au cours du processus, vous devrez peut-être configurer un enregistrement DNS de type TXT dans l'interface DNS de Cloud Internet Services pour prouver que vous êtes le propriétaire du domaine.
{:tip}

Une fois que vous avez obtenu le certificat SSL et la clé privée de votre domaine, veillez à les convertir au format [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Pour convertir un certificat au format PEM : 
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. Pour convertir une clé privée au format PEM : 
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Importation du certificat dans un référentiel central

1. Créez une instance [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) dans un emplacement pris en charge.
1. Dans le tableau de bord de service, utilisez **Importer le certificat** :
   * Définissez **Nom** sur le sous-domaine et le domaine personnalisés, tel que *api.mydomain.com*.
   * Recherchez le **Fichier de certificat** au format PEM.
   * Recherchez le **Fichier de clé privée** au format PEM.
   * **Importez**.

## Déploiement d'actions dans plusieurs emplacements 

Dans cette section, vous allez créer des actions, les exposer en tant qu'API et mapper le domaine personnalisé à l'API avec un certificat SSL stocké dans {{site.data.keyword.cloudcerts_short}}. 

<p style="text-align: center;">

  ![ Architecture d'API](images/solution44-multi-region-serverless/api-architecture.png)
</p>

L'action **doWork** implémente l'une de vos opérations d'API. L'action **healthz** sera utilisée ultérieurement lors de la vérification de la santé de votre API. Il peut s'agir d'une opération simple qui renvoie le message *OK* ou d'une vérification plus complexe, comme l'envoi d'une requête ping aux bases de données ou à d'autres services critiques requis par votre API. 

Les trois sections suivantes doivent être répétées pour chaque emplacement où vous souhaitez héberger le back-end de l'application. Pour ce tutoriel, vous pouvez choisir les cibles *Dallas (us-south)* et *London (eu-gb)*.

### Définition des actions

1. Accédez à [{{site.data.keyword.openwhisk_short}} / Actions](https://{DomainName}/openwhisk/actions).
1. Basculez vers l'emplacement cible et sélectionnez une organisation et un espace où déployer les actions. 
1. Créez une action
   1. Définissez **Nom** sur **doWork**
   1. Définissez **Package de regroupement** sur **default**.
   1. Définissez **Exécution** sur la version la plus récente de **Node.js**.
   1. **Créez**.
1. Remplacez le code d'action par : 
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. **Sauvegardez**
1. Créez une autre action à utiliser comme contrôle de santé de votre API : 
   1. Définissez **Nom** sur **healthz**
   1. Définissez **Package de regroupement** sur **default**.
   1. Définissez **Exécution** sur la version la plus récente de **Node.js**.
   1. **Créez**.
1. Remplacez le code d'action par : 
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. **Sauvegardez**

### Exposition des actions avec une API gérée 

L'étape suivante consiste à créer une API gérée pour exposer vos actions. 

1. Accédez à [{{site.data.keyword.openwhisk_short}} / API](https://{DomainName}/openwhisk/apimanagement).
1. Créez une API {{site.data.keyword.openwhisk_short}} gérée :
   1. Définissez **Nom de l'API** sur **App API**.
   1. Définissez **Chemin de base** sur **/api**. 
1. Créez une opération : 
   1. Définissez **Chemin** sur **/do**. 
   1. Définissez **Instruction** sur **GET**.
   1. Définissez **Package** sur **default**.
   1. Définissez **Action** sur **doWork**
   1. **Créez**.
1. Créez une autre opération :
   1. Définissez **Chemin** sur **/healthz**. 
   1. Définissez **Instruction** sur **GET**.
   1. Définissez **Package** sur **default**.
   1. Définissez **Action** sur **healthz**
   1. **Créez**.
1. **Sauvegardez** l'API.

### Configuration du domaine personnalisé pour l'API gérée 

La création d’une API gérée génère un noeud final par défaut tel que `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`. Dans cette section, vous allez configurer ce noeud final pour qu'il puisse gérer les demandes provenant de votre sous-domaine personnalisé, le domaine qui sera configuré ultérieurement dans IBM Cloud Internet Services. 

1. Accédez à [API / Domaines personnalisés](https://{DomainName}/apis/domains).
1. Dans le sélecteur **Région**, sélectionnez l'emplacement cible.
1. Localisez le domaine personnalisé lié à l'organisation et à l'espace où vous avez créé les actions et l'API gérée. Cliquez sur **Modifier les paramètres** dans le menu d'action.
1. Notez la valeur de **Domaine par défaut / alias**.
1. Sélectionnez **Appliquer le domaine personnalisé**
   1. Définissez **Nom du domaine** sur le domaine que vous utiliserez avec l'équilibreur de charge global CIS, tel que *api.mydomain.com*.
   1. Sélectionnez l'instance {{site.data.keyword.cloudcerts_short}} qui détient le certificat.
   1. Sélectionnez le certificat pour le domaine.
1. Accédez au tableau de bord de votre instance **Cloud Internet Services**, sous **Fiabilité / DNS** et créez un **enregistrement DNS TXT** :
   1. Définissez **Nom** sur votre sous-domaine personnalisé, tel que **api**.
   1. Définissez **Contenu** sur **Domaine par défaut / Alias**
   1. Sauvegardez l'enregistrement.
1. Sauvegardez les paramètres du domaine personnalisé. La boîte de dialogue vérifie l'existence de l'enregistrement DNS TXT. 

   Si aucun enregistrement TXT n’est trouvé, vous devrez probablement attendre qu’il se propage et réessayer d’enregistrer les paramètres. L’enregistrement DNS TXT peut être supprimé une fois les paramètres appliqués.
   {: tip}

Répétez les sections précédentes pour configurer d'autres emplacements. 

## Répartition du trafic entre les emplacements 

**A ce stade, vous avez configuré des actions dans plusieurs emplacements**, mais il n'existe pas de point d’entrée unique pour les atteindre. Dans cette section, vous allez configurer un équilibreur de charge global (Global Load Balancer - GLB) pour répartir le trafic entre les emplacements. 

<p style="text-align: center;">

  ![Architecture de l'équilibreur de charge global](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### Création d'un contrôle de santé 

Internet Services appelle régulièrement ce noeud final pour vérifier l'intégrité du back-end. 

1. Accédez au tableau de bord de votre instance IBM Cloud Internet Services. 
1. Sous **Fiabilité / Equilibreurs de charge globaux**, créez un contrôle de santé :
   1. Définissez **Type de moniteur** sur **HTTPS**.
   1. Définissez **Chemin** sur **/api/healthz**. 
   1. **Mettez la ressource à disposition**.

### Création de pools d'origines

En créant un pool par emplacement, vous pouvez configurer ultérieurement des itinéraires géographiques dans votre équilibreur de charge global afin de rediriger les utilisateurs vers l'emplacement le plus proche. Une autre option consiste à créer un pool unique avec tous les emplacements et effectuer le cycle de l’équilibreur de charge dans les origines du pool. 

Pour chaque emplacement :
1. Créez un pool d'origines.
1. Définissez **Nom** sur **app-&lt;location&gt;** tel que _app-Dallas_.
1. Sélectionnez le bilan de santé créé.
1. Définissez **Régions du contrôle de santé** sur une région proche de l'emplacement où {{site.data.keyword.openwhisk_short}} est déployé.
1. Définissez **Nom de l'origine** sur **app-&lt;location&gt;**.
1. Définissez **Adresse de l'origine** sur le domaine/alias par défaut de l'API gérée (telle que _5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_).
1. **Mettez la ressource à disposition**.

### Création d'un équilibreur de charge global (GLB) 

1. Créez un équilibreur de charge. 
1. Définissez **Nom d'hôte de l'équilibreur** sur **api.mydomain.com**.
1. Ajoutez les pools d'origines régionaux. 
1. **Mettez la ressource à disposition**.

Après quelques instants, accédez à `https://api.mydomain.com/api/do?name=John&place=Earth`. Cette adresse devrait renvoyer la fonction en cours d'exécution dans le premier pool sain. 

### Test de basculement

Pour que vous puissiez tester le basculement, un contrôle de santé du pool doit échouer afin que le GLB soit redirigé vers le prochain pool sain. Pour simuler une défaillance, vous pouvez modifier la fonction de contrôle de santé pour la faire échouer. 

1. Accédez à [{{site.data.keyword.openwhisk_short}} / Actions](https://{DomainName}/openwhisk/actions).
1. Sélectionnez le premier emplacement configuré dans le GLB.
1. Editez la fonction `healthz` et remplacez son implémentation par `throw new Error()`.
1. Enregistrez.
1. Patientez pendant que le contrôle de santé s'exécute sur ce pool d'origines. 
1. Obtenez à nouveau `https://api.mydomain.com/api/do?name=John&place=Earth`, il devrait maintenant être redirigé vers l'autre origine saine.
1. Annulez les modifications de code pour revenir à une origine saine. 

## Suppression de ressources 
{: #removeresources}

### Suppression des ressources CIS 

1. Supprimez le GLB.
1. Supprimez les pools d'origines.
1. Supprimez les contrôles de santé.

### Suppression des actions

1. Supprimez les [APIs](https://{DomainName}/openwhisk/apimanagement)
1. Supprimez les [actions](https://{DomainName}/openwhisk/actions)

## Contenu associé
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Clusters Kubernetes multirégions résilients et sécurisés avec Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
