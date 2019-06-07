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

# Application de la sécurité de bout en bout à une application cloud 
{: #cloud-e2e-security}

Aucune architecture d’application n’est complète sans une compréhension claire des risques potentiels pour la sécurité et de la protection contre de telles menaces. Les données d'application sont des ressources essentielles qui ne peuvent être perdues, compromises ou volées. De plus, les données doivent être protégées au repos et en transit au moyen de techniques de chiffrement. Le chiffrement des données inactives empêche la divulgation des informations, même si elles sont perdues ou volées. Le chiffrement des données en transit (sur Internet, par exemple) à l'aide de méthodes telles que HTTPS, SSL et TLS empêche les écoutes illicites et les attaques de type "man-in-the-middle". 

L'authentification et l'autorisation de l'accès des utilisateurs à des ressources spécifiques constituent une autre exigence courante pour de nombreuses applications. Il peut être nécessaire de prendre en charge différents schémas d’authentification : clients et fournisseurs utilisant des identités sociales, partenaires d’annuaires hébergés dans le cloud et employés du fournisseur d’identité d’une entreprise.

Ce tutoriel décrit les principaux services de sécurité disponibles dans le catalogue {{site.data.keyword.cloud}} et explique comment les utiliser ensemble. Une application permettant le partage de fichiers met en pratique les concepts de sécurité.
{:shortdesc}

## Objectifs
{: #objectives}

* Chiffrer le contenu dans des compartiments de stockage avec vos propres clés de chiffrement 
* Demander aux utilisateurs de s'authentifier avant d'accéder à une application 
* Surveiller et auditer les appels d'API liés à la sécurité et d'autres actions sur les services cloud 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* Facultatif : [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

Ce tutoriel nécessite un compte [autre que Lite](https://{DomainName}/docs/account?topic=account-accounts#accounts) et peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

Ce tutoriel présente un exemple d'application permettant aux groupes d'utilisateurs de télécharger des fichiers vers un pool de stockage commun et d'accéder à ces fichiers via des liens partageables. L'application est écrite dans Node.js et déployée en tant que conteneur Docker sur {{site.data.keyword.containershort_notm}}. Elle exploite plusieurs services et fonctionnalités liés à la sécurité pour améliorer l'état de sécurité de l'application. 

<p style="text-align: center;">

  ![Architecture](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. L'utilisateur se connecte à l'application. 
2. Si vous utilisez un domaine personnalisé et un certificat TLS, le certificat est géré par et déployé à partir de {{site.data.keyword.cloudcerts_short}}.
3. {{site.data.keyword.appid_short}} sécurise l'application et redirige l'utilisateur vers la page d'authentification. Les utilisateurs peuvent également s'inscrire. 
4. L'application s'exécute dans un cluster Kubernetes à partir d'une image stockée dans {{site.data.keyword.registryshort_notm}}. Cette image est automatiquement analysée à la recherche de vulnérabilités. 
5. Les fichiers téléchargés sont stockés dans {{site.data.keyword.cos_short}} avec les métadonnées correspondantes stockées dans {{site.data.keyword.cloudant_short_notm}}.
6. Les compartiments de stockage de fichiers utilisent une clé fournie par l'utilisateur pour chiffrer les données. 
7. Les activités de gestion des applications sont consignées par {{site.data.keyword.cloudaccesstrailshort}}.

## Avant de commencer
{: #prereqs}

1. Installez tous les outils de ligne de commande nécessaires [en procédant comme suit](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
2. Assurez-vous de disposer de la dernière version des plugins utilisée dans ce tutoriel. Utilisez `ibmcloud plugin update --all` pour la mise à niveau.

## Création de services
{: #setup}

### Choix de l'emplacement du déploiement de l'application 

1. Identifiez l'**emplacement**, **l'organisation et l'espace Cloud Foundry** ainsi que le **groupe de ressources** où vous allez déployer l'application et ses ressources.

### Capture des activités des utilisateurs et des applications  
{: #activity-tracker }

Le service {{site.data.keyword.cloudaccesstrailshort}} enregistre les activités initiées par l'utilisateur qui changent l'état d'un service dans {{site.data.keyword.Bluemix_notm}}. A la fin de ce tutoriel, vous passerez en revue les événements générés par la réalisation de ses étapes. 

1. Accédez au catalogue {{site.data.keyword.Bluemix_notm}} et créez une instance de [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker). Notez qu'il ne peut y avoir qu'une seule instance d'{{site.data.keyword.cloudaccesstrailshort}} par espace Cloud Foundry. 
2. Une fois l’instance créée, remplacez son nom par **secure-file-storage-activity-tracker**.
3. Pour afficher tous les événements du suivi d'activité, assurez-vous que vous disposez des droits adéquats : 
   1. Accédez à [Identity & Access > Utilisateurs](https://{DomainName}/iam/#/users).
   2. Sélectionnez votre nom d'utilisateur dans la liste. 
   3. Sous **Règles d'accès**, si elle n'existe pas, créez une règle pour le service {{site.data.keyword.loganalysisshort_notm}} avec le rôle **Afficheur** à l'emplacement où le service a été créé. 
   4. Sous **Cloud Foundry Access**, vérifiez que vous disposez du rôle **Développeur** dans l'espace Cloud Foundry où {{site.data.keyword.cloudaccesstrailshort}} a été mis à disposition. Si tel n'est pas la cas, collaborez avec le responsable Cloud Foundry de votre entreprise pour obtenir ce rôle. 

Vous trouverez des instructions détaillées sur la configuration des droits dans la documentation [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy).
{: tip}

### Création d'un cluster pour l'application 

{{site.data.keyword.containershort_notm}} fournit un environnement permettant de déployer des applications hautement disponibles dans des conteneurs Docker exécutés dans des clusters Kubernetes.

Ignorez cette section si vous disposez d'un cluster **Standard** existant que vous souhaitez réutiliser avec ce tutoriel.
{: tip}

1. Accédez à la [page de création de cluster](https://{DomainName}/containers-kubernetes/catalog/cluster/create).
   1. Définissez l'**Emplacement** sur celui utilisé lors des étapes précédentes. 
   2. Définissez le **Type de cluster** sur **Standard**.
   3. Définissez **Disponibilité** sur **Zone unique**.
   4. Sélectionnez une **zone maître**.
2. Conservez la **Version de Kubernetes** et l'**Isolement du matériel** par défaut. 
3. Si vous envisagez de ne déployer que ce tutoriel sur ce cluster, définissez les **Noeuds worker** sur **1**. 
4. Définissez le **Nom de cluster** sur **secure-file-storage-cluster**. 
5. Cliquez sur le bouton **Créer un cluster**.

Pendant la mise à disposition du cluster, vous allez créer les autres services requis par le tutoriel. 

### Utilisation de vos propres clés de chiffrement

{{site.data.keyword.keymanagementserviceshort}} permet de mettre à disposition des clés chiffrées pour des applications au sein des services {{site.data.keyword.Bluemix_notm}}. {{site.data.keyword.keymanagementserviceshort}} et {{site.data.keyword.cos_full_notm}} [fonctionnent ensemble pour protéger vos données au repos](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos). Dans cette section, vous allez créer une clé racine pour le compartiment de stockage. 

1. Créez une instance de [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms).
   * Définissez le nom sur **secure-file-storage-kp**.
   * Sélectionnez le groupe de ressources où créer l'instance de service. 
2. Sous **Gérer**, cliquez sur le bouton **Ajouter une clé**pour créer une clé racine. Elle est utilisée pour chiffrer le contenu du compartiment de stockage. 
   * Définissez le nom sur **secure-file-storage-root-enckey**.
   * Définissez le type de clé sur **Root key**.
   * Puis, **Générez la clé**.

Apportez votre propre clé (Bring your own key - BYOK) en [important une clé racine existante](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys).
{: tip}

### Configuration du stockage pour les fichiers utilisateur 

L'application de partage de fichiers enregistre les fichiers dans un compartiment {{site.data.keyword.cos_short}}. La relation entre les fichiers et les utilisateurs est stockée sous forme de métadonnées dans une base de données {{site.data.keyword.cloudant_short_notm}}. Dans cette section, vous allez créer et configurer ces services. 

#### Compartiment destiné au contenu

1. Créez une instance de [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
   * Définissez le **nom** sur **secure-file-storage-cos**. 
   * Utilisez le même **groupe de ressources** que pour les services précédents. 
2. Sous **Données d'identification pour le service**, créez de *Nouvelles données d'identification*. 
   * Définissez le **nom** sur **secure-file-storage-cos-acckey**. 
   * Définissez **Rôle** sur **Auteur**.
   * Ne spécifiez pas d'**ID de service**.
   * Définissez les **Paramètres de configuration en ligne** à **{"HMAC":true}**. Ils sont nécessaires pour générer des URL pré-signées. 
   * Cliquez sur **Ajouter**. 
   * Prenez note des données d'identification en cliquant sur **Afficher les données d'identification**. Vous en aurez besoin dans une étape ultérieure. 
3. Cliquez sur **Noeud final** dans le menu : définissez **Résilience** sur **Régionale** et l'**Emplacement** sur l'emplacement cible. Copiez le noeud final de service **Public**. Il sera utilisé ultérieurement dans la configuration de l'application. 

Avant de créer le compartiment, vous devez accorder à **secure-file-storage-cos** l’accès à la clé racine stockée dans **secure-file-storage-kp**.

1. Accédez à [Identity & Access > Autorisations](https://{DomainName}/iam/#/authorizations) dans la console {{site.data.keyword.cloud_notm}}.
2. Cliquez sur le bouton **Créer**.
3. Dans le menu **Service source**, sélectionnez **Cloud Object Storage**.
4. Dans le menu **Instance de service source**, sélectionnez le service **secure-file-storage-cos** créé précédemment.
5. Dans le menu **Service cible**, sélectionnez **Key Protect**.
6. Dans le menu **Instance de service cible**, sélectionnez le service **secure-file-storage-kp** à autoriser.
7. Activez le rôle **Lecteur**.
8. Cliquez sur le bouton **Autoriser**.

Enfin, créez le compartiment. 

1. Accédez à l'instance de service **secure-file-storage-cos** à partir de la [Liste de ressources](https://{DomainName}/resources).
2. Cliquez sur **Créer un compartiment**.
   1. Définissez le **nom** sur une valeur unique, telle que **&lt;your-initials&gt;-secure-file-upload**.
   2. Définissez **Résilience** sur **Régionale**.
   3. Définissez **Emplacement** sur le même emplacement que celui où vous avez créé le service **secure-file-storage-kp**.
   4. Définissez la **Classe de stockage** sur **Standard**
3. Cochez la case **Add Key Protect Keys**.
   1. Sélectionnez le service **secure-file-storage-kp**.
   2. Sélectionnez **secure-file-storage-root-enckey** en tant que clé.
4. Cliquez sur **Créer un compartiment**.

#### Relation de carte de base de données entre les utilisateurs et leurs fichiers 

La base de données {{site.data.keyword.cloudant_short_notm}} contient des métadonnées pour tous les fichiers téléchargés à partir de l'application. 

1. Créez une instance de [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB).
   * Définissez le **nom** sur **secure-file-storage-cloudant**. 
   * Définissez l'emplacement.
   * Utilisez le même **groupe de ressources** que pour les services précédents. 
   * Définissez les **méthodes d'authentification disponibles ** sur **Utiliser uniquement IAM**.
2. Sous **Données d'identification pour le service**, créez de *Nouvelles données d'identification*. 
   * Définissez le **nom** sur **secure-file-storage-cloudant-acckey**. 
   * Définissez **Rôle** sur **Responsable**
   * Conservez les valeurs par défaut pour les zones *facultatives* 
   * **Ajoutez**.
3. Prenez note des données d'identification en cliquant sur **Afficher les données d'identification**. Vous en aurez besoin dans une étape ultérieure. 
4. Sous **Gérer**, lancez le tableau de bord Cloudant.
5. Cliquez sur **Create Database** pour créer une base de données nommée **secure-file-storage-metadata**.

### Authentification des utilisateurs

Avec {{site.data.keyword.appid_short}}, vous pouvez sécuriser des ressources et ajouter une authentification à vos applications. {{site.data.keyword.appid_short}} [s'intègre à](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth) avec {{site.data.keyword.containershort_notm}} pour authentifier les utilisateurs accédant aux applications déployées dans le cluster.

1. Créez une instance de [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID).
   * Définissez le **Nom de service** sur **secure-file-storage-appid**.
   * Utilisez le même **emplacement** et **groupe de ressources** que pour les services précédents. 
2. Sous **Fournisseurs d'identité/Gérer**, dans l'onglet **Paramètres d'authentification**, ajoutez une **URL de redirection Web** pointant vers le domaine que vous utilisez pour l'application. Par exemple, si votre sous-domaine Ingress de cluster est
`<cluster-name>.us-south.containers.appdomain.cloud`, l'URL de redirection est `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback`. {{site.data.keyword.appid_short}} nécessite une URL de redirection Web **https**. Vous pouvez afficher votre sous-domaine Ingress dans le tableau de bord du cluster ou avec `ibmcloud ks cluster-get <cluster-name>`.

Vous devez personnaliser les fournisseurs d'identité utilisés, ainsi que l'expérience de gestion des connexions et des utilisateurs dans le tableau de bord {{site.data.keyword.appid_short}}. Ce tutoriel utilise les valeurs par défaut pour plus de simplicité. Pour un environnement de production, envisagez d'utiliser l'Authentification multifacteur (MFA) et des règles de mot de passe avancées.
{: tip}

## Déploiement de l'application 

Tous les services ont été configurés. Dans cette section, vous allez déployer l'application du tutoriel sur le cluster. 

### Obtention du code

1. Obtenez le code de l'application : 
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. Accédez au répertoire **secure-file-storage** :
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### Génération de l'image Docker

1. [Générez l'image Docker](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating) dans {{site.data.keyword.registryshort_notm}}.
   - Recherchez le noeud final du registre avec `ibmcloud cr info`, tel que **us**.icr.io ou **uk**.icr.io.
   - Créez un espace de nom pour stocker l'image du conteneur.
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - Utilisez **secure-file-storage** en tant que nom d'image.

   ```sh
   ibmcloud cr build -t <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### Spécification des données d'identification et des paramètres de configuration 

1. Copiez `credentials.template.env` vers `credentials.env`:
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. Editez `credentials.env` et remplissez les espaces avec les valeurs suivantes :
   * le noeud final régional du service {{site.data.keyword.cos_short}}, le nom du compartiment, les données d'identification créées pour **secure-file-storage-cos**,
   * et les données d'identification pour **secure-file-storage-cloudant**.
3. Copiez `secure-file-storage.template.yaml` vers `secure-file-storage.yaml`:
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. Editez `secure-file-storage.yaml` et remplacez les marques de réservation (`$IMAGE_PULL_SECRET`, `$REGISTRY_URL`, `$REGISTRY_NAMESPACE`, `$IMAGE_NAME`, `$TARGET_NAMESPACE`, `$INGRESS_SUBDOMAIN`, `$INGRESS_SECRET`) par les valeurs correctes. Par exemple, supposons que l'application soit déployée dans l'espace de nom Kubernetes _default_ :

| Variable | Valeur | Description |
| -------- | ----- | ----------- |
| `$IMAGE_PULL_SECRET` | Conserver les lignes en commentaire dans le fichier .yaml | Secret pour accéder au registre.  |
| `$REGISTRY_URL` | *us.icr.io* | Registre où l'image a été construite dans la section précédente. |
| `$REGISTRY_NAMESPACE` | *secure-file-storage-namespace* | Espace de nom du registre où l'image a été construite dans la section précédente. |
| `$IMAGE_NAME` | *secure-file-storage* | Nom de l'image Docker. |
| `$TARGET_NAMESPACE` | *default* | Espace de nom Kubernetes où est envoyée l'application. |
| `$INGRESS_SUBDOMAIN` | *secure-file-stora-123456.us-south.containers.appdomain.cloud* | Extraire à partir de la page de présentation du cluster ou avec `ibmcloud ks cluster-get secure-file-storage-cluster`. |
| `$INGRESS_SECRET` | *secure-file-stora-123456* | Extraire à partir de la page de présentation du cluster ou avec `ibmcloud ks cluster-get secure-file-storage-cluster`. |

`$IMAGE_PULL_SECRET` n'est nécessaire que si vous souhaitez utiliser un autre espace de nom Kubernetes que celui par défaut. Cela nécessite une configuration Kubernetes supplémentaire (par exemple, la [création d’un secret de registre Docker dans le nouvel espace de nom](https://{DomainName}/docs/containers?topic=containers-images#other)).
{: tip}

### Déploiement sur le cluster

1. Extrayez la configuration du cluster et définissez la variable d'environnement KUBECONFIG. 
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. Créez le secret utilisé par l'application pour obtenir les données d'identification du service : 
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. Liez l'instance de service {{site.data.keyword.appid_short_notm}} au cluster.
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
Si vous avez plusieurs services du même nom, la commande échoue. Vous devez transmettre l'identificateur global unique (GUID) du service plutôt que son nom. Pour rechercher le GUID d'un service, utilisez `ibmcloud resource service-instance secure-file-storage-appid`.
   {: tip}
4. Déployez l'application.
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## Test de l'application

L'application est accessible à l'adresse `https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/`.

1. Accédez à la page d'accueil de l'application. Vous serez redirigé vers la page de connexion par défaut {{site.data.keyword.appid_short_notm}}. 
2. Inscrivez-vous pour un nouveau compte avec une adresse électronique valide. 
3. Attendez la réception du courrier électronique dans votre boîte de réception pour vérifier le compte. 
4. Connectez-vous.
5. Choisissez un fichier à envoyer par téléchargement. Cliquez sur **Transférer**.
6. Utilisez l'action **Partager** sur un fichier pour générer une URL pré-signée pouvant être partagée avec d'autres personnes pour accéder au fichier. Le lien est configuré pour expirer après 5 minutes. 

Les utilisateurs authentifiés ont leurs propres espaces pour stocker des fichiers. Bien qu'ils ne puissent pas voir les autres fichiers, ils peuvent générer des URL pré-signées pour accorder un accès temporaire à un fichier spécifique. 

Vous pouvez trouver plus de détails sur l'application dans le [référentiel de code source](https://github.com/IBM-Cloud/secure-file-storage).

## Examen des événements de sécurité 
Maintenant que l'application et ses services ont été déployés, vous pouvez consulter les événements de sécurité générés par ce processus. Tous les événements sont disponibles de manière centralisée dans l'instance {{site.data.keyword.cloudaccesstrailshort}} et sont accessibles via une [une interface graphique (Kibana), une interface de ligne de commande ou une API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status).

1. Dans la [Liste de ressources {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), localisez l'instance {{site.data.keyword.cloudaccesstrailshort}} **secure-file-storage-activity-tracker** et ouvrez son tableau de bord.
2. Par défaut, l'onglet **Gérer** affiche les **Journaux d'espace**. Basculez vers les **Journaux de compte** en cliquant sur le sélecteur en regard de **Afficher les journaux**. Plusieurs événements devraient s'afficher. 
3. Cliquez sur **Afficher dans Kibana** pour ouvrir l'afficheur d'événements complet.
4. Passez en revue les détails de l'événement en cliquant sur **Découvrir**.
5. Dans les **Zones disponibles**, ajoutez **action_str** et **initiator.name_str**.
6. Développez les entrées intéressantes en cliquant sur l'icône en forme de triangle, puis en choisissant un tableau ou un format JSON. 

Vous pouvez modifier les paramètres d'actualisation automatique et de la plage de temps affichée afin de modifier comment et quelles données sont analysées.
{: tip}

## Facultatif : utilisation d'un domaine personnalisé et chiffrement du trafic réseau 
Par défaut, l’application est accessible sur un nom d’hôte générique d’un sous-domaine de `containers.appdomain.cloud`. Cependant, il est également possible d'utiliser un domaine personnalisé avec l'application déployée. Pour que l’accès **https** au trafic réseau chiffré soit pris en charge de manière continue, vous devez fournir un certificat pour le nom d’hôte souhaité ou un certificat générique. Dans la section suivante, vous téléchargez un certificat existant sur {{site.data.keyword.cloudcerts_short}} et le déployer sur le cluster. Vous allez également mettre à jour la configuration de l'application pour utiliser le domaine personnalisé. 

Un exemple sur la manière d'obtenir un certificat auprès de [Let's Encrypt](https://letsencrypt.org/) est décrit dans le [blogue {{site.data.keyword.cloud}}](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/) suivant.
{: tip}

1. Créez une instance de [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)
   * Définissez le nom sur **secure-file-storage-certmgr**.
   * Utilisez les mêmes **emplacement** et **groupe de ressources** que pour les autres services. 
2. Cliquez sur **Importer le certificat** pour importer votre certificat.
   * Définissez le nom sur **SecFileStorage** et la description sur **Certificate for e2e security tutorial**.
   * Transférez le fichier de certificat à l'aide du bouton **Parcourir**.
   * Cliquez sur **Importer** pour terminer le processus d'importation.
3. Localisez l'entrée du certificat importé et développez-la. 
   * Vérifiez l'entrée de certificat, par exemple, que le nom de domaine correspond à votre domaine personnalisé. Si vous avez téléchargé un certificat générique, un astérisque est inclus dans le nom de domaine. 
   * Cliquez sur le symbole de **copie** en regard du **crn** du certificat.
4. Passez en ligne de commande pour déployer les informations de certificat dans le cluster en tant que secret. Exécutez la commande suivante après avoir copié le crn du certificat de l'étape précédente. 
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <the copied crn from previous step>
   ```
   {: codeblock}
Vérifiez que le cluster connaît le certificat en exécutant la commande suivante.
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. Editez le fichier `secure-file-storage.yaml`.
   * Recherchez la section relative à **Ingress**.
   * Supprimez la mise en commentaire et éditez les lignes couvrant les domaines personnalisés et complétez votre nom de domaine et votre nom d'hôte. L'entrée CNAME de votre domaine personnalisé doit pointer vers le cluster. Pour plus de détails, consultez ce [guide sur le mappage de domaines personnalisés](https://{DomainName}/docs/containers?topic=containers-ingress#private_3) dans la documentation.
   {: tip}
6. Appliquez les modifications de configuration au fichier déployé : 
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. Revenez au navigateur. Dans la [liste des ressources {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), localisez le service {{site.data.keyword.appid_short}} créé et configuré précédemment et lancez son tableau de bord de gestion. 
   * Accédez à **Gérer** sous **Fournisseurs d'identité**, puis à **Paramètres**.
   * Dans le formulaire **Ajouter des URL de redirection Web**, ajoutez `https://secure-file-storage.<your custom domain>/appid_callback` en tant qu'autre URL.
8. Tout devrait à présent être installé. Testez l'application en y accédant sur votre domaine personnalisé configuré `https://secure-file-storage.<your custom domain>`.

## Extension du tutoriel 

Optimisez la sécurité. Essayez les suggestions ci-dessous pour améliorer la sécurité de votre application. 

* Utilisez [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) pour effectuer des analyses de code statiques et dynamiques 
* Assurez-vous que seul le code qualité est publié en utilisant des stratégies et des règles avec [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)

## Suppression de ressources 
{:removeresources}

Pour supprimer la ressource, supprimez le conteneur déployé, puis les services fournis. 

1. Supprimez le conteneur déployé : 
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. Supprimez les secrets pour le déploiement : 
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. Supprimez l'image Docker du registre de conteneurs : 
   ```sh
   ibmcloud cr image-rm <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. Dans la [Liste de ressources {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), recherchez les ressources créées pour ce tutoriel. Utilisez la zone de recherche et **secure-file-storage** comme modèle. Supprimez chacun des services en cliquant sur le menu contextuel en regard de chaque service et en choisissant **Supprimer le service**. Notez que le service {{site.data.keyword.keymanagementserviceshort}} ne peut être supprimé que lorsque la clé a été supprimée. Cliquez sur l'instance de service pour accéder au tableau de bord associé et supprimer la clé. 

Si vous partagez un compte avec d'autres utilisateurs, veillez toujours à ne supprimer que vos propres ressources.
{: tip}

## Contenu associé
{:related}

* [Documentation {{site.data.keyword.security-advisor_short}}](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [Sécurité pour protéger et surveiller vos applications cloud](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [Sécurité de la plateforme {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/overview?topic=overview-security#security)
* [Sécurité d'IBM Cloud](https://www.ibm.com/cloud/security)
* [Tutoriel : Bonnes pratiques en matière d'organisation des utilisateurs, des équipes et des applications](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [Applications sécurisées dans IBM Cloud avec certificats génériques](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
