---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: data-hd-operatingsystem="ios"}
{:android: data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Application mobile avec back-end sans serveur 
{: #serverless-mobile-backend}

Dans ce tutoriel, vous allez apprendre à utiliser {{site.data.keyword.openwhisk}} avec les services Cognitive et Data pour créer un back-end sans serveur pour une application mobile.
{:shortdesc}

Certains développeurs mobiles n'ont pas d'expérience en gestion de logique côté serveur ou même des serveurs. Ils préfèrent concentrer leurs efforts sur l'application qu'ils génèrent. Or, que se passerait-il s’ils pouvaient réutiliser leurs compétences de développement pour développer leur back-end mobile ? 

{{site.data.keyword.openwhisk_short}} est une plateforme sans serveur pilotée par événements. Comme [indiqué dans cet exemple](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp), les actions que vous déployez peuvent facilement être transformées en noeuds finaux HTTP en tant qu'*actions Web* pour créer une API de back-end d'application Web. Une application Web étant un client de l'API REST, il est facile d'étendre cet exemple et d'appliquer la même approche pour créer un back-end destiné à une application mobile. Et avec {{site.data.keyword.openwhisk_short}}, les développeurs mobiles peuvent écrire les actions dans le même langage que celui utilisé pour leur application mobile, Java pour Android et Swift pour iOS. 

Ce tutoriel est configurable en fonction de votre plateforme cible. Vous consultez actuellement la documentation de la version **iOS / Swift** de ce tutoriel. Utilisez l'onglet situé au début de cette documentation pour sélectionner la version **Android / Java** de ce tutoriel.
{: swift}

Ce tutoriel est configurable en fonction de votre plateforme cible. Vous consultez actuellement la documentation de la version **Android / Java** de ce tutoriel. Utilisez l'onglet situé au début de cette documentation pour sélectionner la version **iOS / Swift** de ce tutoriel.
{: java}

## Objectifs
{: #objectives}

* Déployer un back-end mobile sans serveur avec {{site.data.keyword.openwhisk_short}}.
* Ajouter une authentification d'utilisateur à une application mobile avec {{site.data.keyword.appid_short}}.
* Analyser les commentaires des utilisateurs avec {{site.data.keyword.toneanalyzershort}}.
* Envoyer des notifications avec {{site.data.keyword.mobilepushshort}}.

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

L'application présentée dans ce tutoriel est une application de commentaires en retour qui analyse intelligemment le ton du texte des commentaires en retour et reconnaît de manière appropriée le client par le biais de {{site.data.keyword.mobilepushshort}}.

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. L'utilisateur s'authentifie avec [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID). {{site.data.keyword.appid_short}} fournit des jetons d'accès et d'identification.
2. Les autres appels à l'API de back-end incluent le jeton d'accès. 
3. Le back-end est implémenté avec [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk). Les actions sans serveur, exposées en tant qu'Actions Web, attendent l'envoi du jeton dans les en-têtes de requête et vérifient sa validité (signature et date d'expiration) avant d'autoriser l'accès à l'API même. 
4. Lorsque l'utilisateur envoie des commentaires en retour, ils sont stockés dans [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
5. Le texte des commentaires en retour est traité avec [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer).
6. Sur la base du résultat de l'analyse, une notification est renvoyée à l'utilisateur avec [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush).
7. L'utilisateur reçoit la notification sur l'unité. 

## Avant de commencer
{: #prereqs}

Ce tutoriel utilise l'outil de ligne de commande {{site.data.keyword.Bluemix_notm}} pour mettre à disposition des ressources et déployer du code. Veillez à installer l'outil de ligne de commande `ibmcloud`.

* [{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Script d'installation de l'interface CLI ibmcloud et des plug-ins requis (Cloud Foundry et {{site.data.keyword.openwhisk_short}})

De plus, les logiciels et comptes suivants sont nécessaires : 

   1. Java 8
   2. Android Studio 2.3.3
   3. Compte de développeur Google pour configurer Firebase Cloud Messaging 
   4. Bash shell, cURL
   {: java}


   1. Xcode
   2. Compte de développeur Apple pour configurer le service de notification Apple Push 
   3. Bash shell, cURL
   {: swift}

Dans ce tutoriel, vous allez configurer le service Push Notifications pour l'application. Ce tutoriel suppose que vous avez effectué le tutoriel de base {{site.data.keyword.mobilepushshort}} pour [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) ou [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics) et que vous connaissez la configuration du service Firebase Cloud Messaging ou Apple Push Notification.
{:tip}

Pour que les utilisateurs de Windows 10 puissent utiliser les instructions de la ligne de commande, nous vous recommandons d’installer le sous-système Windows pour Linux et Ubuntu comme décrit dans [cet article](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10).
{: tip}
{: java}

## Obtention du code applicatif 

Le référentiel contient à la fois l'application mobile et le code d'actions {{site.data.keyword.openwhisk_short}}. 

1. Réservez le code dans le référentiel GitHub 

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-android
   ```
   {: pre}
   {: java}

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-ios
   ```
   {: pre}
   {: swift}

2. Examinez la structure du code

| Fichier                                     | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions) | Code des actions {{site.data.keyword.openwhisk_short}} du back-end mobile sans serveur |
| [**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android) | Code de l'application mobile          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh) | Script auxiliaire pour installer, désinstaller et mettre à jour le déclencheur, les actions et les règles {{site.data.keyword.openwhisk_short}} |
{: java}

| Fichier                                     | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions) | Code des actions {{site.data.keyword.openwhisk_short}} du back-end mobile sans serveur |
| [**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp) | Code de l'application mobile          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh) | Script auxiliaire pour installer, désinstaller et mettre à jour le déclencheur, les actions et les règles {{site.data.keyword.openwhisk_short}} |
{: swift}

## Mise à disposition de services pour gérer l'authentification des utilisateurs, la persistance des commentaires et l'analyse 
{: #provision_services}

Dans cette section, vous mettez à disposition les services utilisés par l’application. Vous pouvez choisir de mettre à disposition les services à partir du catalogue {{site.data.keyword.Bluemix_notm}} ou à l'aide de la ligne de commande `ibmcloud`. 

Il est recommandé de créer un espace pour mettre à disposition les services et déployer le back-end sans serveur. Cette action permet de conserver toutes les ressources ensemble. 

### Mise à disposition de services à partir du catalogue {{site.data.keyword.Bluemix_notm}}

1. Accédez au catalogue [{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/catalog/)
2. Créez un service [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) avec le forfait **Lite**. Définissez le nom sur **serverlessfollowup-db**.
3. Créez un service [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) avec le forfait **Standard**. Définissez le nom sur **serverlessfollowup-tone**.
4. Créez un service [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) avec le forfait **Tranche graduée**. Définissez le nom sur **serverlessfollowup-appid**.
5. Créez un service [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) avec le forfait **Lite**. Définissez le nom sur **serverlessfollowup-mobilepush**.

### Mise à disposition de services à partir de la ligne de commande 

Dans la ligne de commande, exécutez les commandes suivantes pour mettre à dispositions les services et extraire leurs données d'identification: 

   ```sh
   ibmcloud cf create-service cloudantNoSQLDB Lite serverlessfollowup-db
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service tone_analyzer standard serverlessfollowup-tone
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service AppID "Graduated tier" serverlessfollowup-appid
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service imfpush lite serverlessfollowup-mobilepush
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

## Configuration de {{site.data.keyword.mobilepushshort}}
{: #push_notifications}

Lorsqu'un utilisateur envoie un nouveau commentaire, l'application analyse ce commentaire et renvoie une notification à l'utilisateur.  Par conséquent, l'utilisation de Push Notifications est un bon moyen de communiquer avec l'utilisateur. Le service {{site.data.keyword.mobilepushshort}} permet d'envoyer des notifications aux utilisateurs iOS ou Android via une API unifiée. Dans cette section, vous allez configurer le service {{site.data.keyword.mobilepushshort}} pour votre plateforme cible. 

### Configuration de Firebase Cloud Messaging (FCM)
{: java}

   1. Dans la [console Firebase](https://console.firebase.google.com), créez un projet. Définissez le nom sur **serverlessfollowup**
   2. Accédez aux **Paramètres** du projet
   3. Sous l'onglet **Général**, ajoutez deux applications :
      1. une dont le nom de package est défini à : **com.ibm.mobilefirstplatform.clientsdk.android.push**
      2. et une dont le nom de package est défini à : **serverlessfollowup.app**
   4. Téléchargez le fichier `google-services.json` contenant les deux applications définies à partir de la console Firebase et placez ce fichier dans le dossier `android/app` du répertoire de réservation. 
   5. Recherchez l'ID de l'expéditeur et la clé du serveur (également appelée clé d'API par la suite) sous l'onglet **Cloud Messaging**.
   6. Dans le tableau de bord du service Push Notifications, définissez la valeur de l'ID de l'expéditeur et de la clé d'API.
   {: java}

### Configuration du service Apple Push Notifications (APN) 
{: swift}

1. Accédez au portail [Apple Developer](https://developer.apple.com/) et enregistrez un ID d'application.
2. Créez un certificat SSL APN pour le développement et la distribution
3. Créez un profil de mise à disposition pour le développement. 
4. Configurez l'instance
de service {{site.data.keyword.mobilepushshort}} sur {{site.data.keyword.Bluemix_notm}}. Pour connaître les étapes détaillées, reportez-vous à [Obtention des données d'identification du service APN et configuration du service {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-).
{: swift}

## Déploiement d'un back-end sans serveur
{: #serverless_backend}

Tous les services étant configurés, vous pouvez maintenant déployer le back-end sans serveur. Les artefacts {{site.data.keyword.openwhisk_short}} suivants sont créés dans cette section :

| Artefact                      | Type                           | Description                              |
| ----------------------------- | ------------------------------ | ---------------------------------------- |
| `serverlessfollowup`          | Package                        | Package permettant de regrouper les actions et de conserver toutes les données d'identification du service.|
| `serverlessfollowup-cloudant` | Liaison de package                | Lié au package {{site.data.keyword.cloudant_short_notm}} intégré   |
| `serverlessfollowup-push`     | Liaison de package                | Lié au package {{site.data.keyword.mobilepushshort}}  |
| `auth-validate`               | Action                         | Valide les jetons d'accès et d'identification |
| `users-add`                   | Action                         | Conserve les informations utilisateur (id, nom, e-mail, image, id d'unité) |
| `users-prepare-notify`        | Action                         | Formate un message à utiliser avec {{site.data.keyword.mobilepushshort}} |
| `feedback-put`                | Action                         | Stocke les commentaires en retour utilisateur dans la base de données   |
| `feedback-analyze`            | Action                         | Analyse des commentaires en retour avec {{site.data.keyword.toneanalyzershort}}   |
| `users-add-sequence`          | Séquence exposée en tant qu'action Web | `auth-validate` et `users-add`          |
| `feedback-put-sequence`       | Séquence exposée en tant qu'action Web | `auth-validate` et `feedback-put`       |
| `feedback-analyze-sequence`   |Séquence | `read-document` de {{site.data.keyword.cloudant_short_notm}}, `feedback-analyze`, `users-prepare-notify` et `sendMessage` avec {{site.data.keyword.mobilepushshort}} |
| `feedback-analyze-trigger`    | Déclencheur                        | Appelé par {{site.data.keyword.openwhisk_short}} lorsque des commentaires en retour sont stockés dans la base de données |
| `feedback-analyze-rule`       | Règle                           | Lie le déclencheur `feedback-analyze-trigger` à la séquence `feedback-analyze-sequence` |

### Compilation du code
{: java}
1. A la racine du répertoire de réservation, compilez le code des actions
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### Configuration et déploiement des actions
{: java}

2. Copiez template.local.env vers local.env

   ```sh
   cp template.local.env local.env
   ```
3. Obtenez les données d'identification pour les services {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}} et {{site.data.keyword.appid_short}} à partir du tableau de bord {{site.data.keyword.Bluemix_notm}} (ou la sortie des commandes ibmcloud exécutées précédemment) et remplacez les marques de réservation dans `local.env` par des valeurs correspondantes. Ces propriétés sont injectées dans un package afin que toutes les actions puissent accéder à la base de données. 
4. Déployez les actions sur {{site.data.keyword.openwhisk_short}}. `deploy.sh` charge les données d'identification à partir de `local.env` pour créer les bases de données {{site.data.keyword.cloudant_short_notm}} (utilisateurs, commentaires et humeurs) et déployez les artefacts {{site.data.keyword.openwhisk_short}} pour l’application.
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   Vous pouvez utiliser `./deploy.sh --uninstall` pour supprimer les artefacts {{site.data.keyword.openwhisk_short}} une fois le tutoriel terminé.
   {: tip}

### Configuration et déploiement des actions
{: swift}

1. Copiez template.local.env vers local.env
   ```sh
   cp template.local.env local.env
   ```
2. Obtenez les données d'identification pour les services {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}} et {{site.data.keyword.appid_short}} à partir du tableau de bord {{site.data.keyword.Bluemix_notm}} (ou la sortie des commandes ibmcloud exécutées précédemment) et remplacez les marques de réservation dans `local.env` par des valeurs correspondantes. Ces propriétés sont injectées dans un package afin que toutes les actions puissent accéder à la base de données. 
3. Déployez les actions sur {{site.data.keyword.openwhisk_short}}. `deploy.sh` charge les données d'identification à partir de `local.env` pour créer les bases de données {{site.data.keyword.cloudant_short_notm}} (utilisateurs, commentaires et humeurs) et déployez les artefacts {{site.data.keyword.openwhisk_short}} pour l’application.

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   Vous pouvez utiliser `./deploy.sh --uninstall` pour supprimer les artefacts {{site.data.keyword.openwhisk_short}} une fois le tutoriel terminé.
   {: tip}

## Configuration et exécution d'une application mobile native pour recueillir les commentaires des utilisateurs 
{: #mobile_app}

Les actions {{site.data.keyword.openwhisk_short}} sont prêtes pour l'application mobile. Avant d'exécuter l'application mobile, vous devez configurer ses paramètres pour cibler les services que vous avez créés. 

1. Avec Android Studio, ouvrez le projet situé dans le dossier `android` de votre répertoire de réservation.
2. Editez `android/app/src/main/res/values/credentials.xml` et remplissez les espaces avec les valeurs des données d'identification. Vous avez besoin des éléments {{site.data.keyword.appid_short}} `tenantId`, {{site.data.keyword.mobilepushshort}} `appGuid` et `clientSecret` ainsi que des noms d'organisation et d'espace sur lesquels {{site.data.keyword.openwhisk_short}} a été déployé. Pour un hôte d'API, lancez l'invite de commande ou le terminal et exécutez la commande
   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
3. Ouvrez `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` et `LoginAndRegistrationListener.java` et mettez à jour la région du service Push Notifications (BMSClient) et la région AppID en fonction de l'emplacement de création de vos instances de service.
4. Générez le projet.
5. Démarrez l'application sur une unité réelle ou avec un émulateur. Pour que l'émulateur reçoive des notifications push, veillez à choisir une image avec les API Google et à vous connecter avec un compte Google au sein de l'émulateur.
   {: tip}
6. Observez {{site.data.keyword.openwhisk_short}} en arrière-plan.
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. Dans l'application, sélectionnez **Connexion** pour vous authentifier avec un compte Facebook ou Google. Une fois connecté, entrez un message de commentaire en retour et appuyez sur le bouton **Envoyer un commentaire**. Quelques secondes après l'envoi du commentaire, vous devriez recevoir une notification push sur l'unité. Le texte de notification peut être personnalisé en modifiant les modèles de documents dans la base de données `moods` de l'instance de service {{site.data.keyword.cloudant_short_notm}}. Utilisez le bouton **View token** pour inspecter les jetons d'accès et d'identification générés par {{site.data.keyword.appid_short}} lors de la connexion.
{: java}


1. Le SDK client Push et d’autres SDK sont disponibles sur CocoaPods et Carthage. Pour cette solution, nous allons utiliser CocoaPods.
2. Ouvrez un terminal et exécutez la commande `cd ` dans le dossier `followupapp`. Exécutez la commande ci-dessous pour installer les dépendances requises. 
   ```sh
   pod install
   ```
   {: pre}
3. Ouvrez le fichier portant l'extension `.xcworkspace` situé dans le dossier `followupapp` de votre répertoire de réservation pour lancer votre code dans Xcode.
4. Editez le fichier `BMSCredentials.plist` et remplissez les espaces avec les valeurs des données d'identification. Vous avez besoin des éléments {{site.data.keyword.appid_short}} `tenantId`, {{site.data.keyword.mobilepushshort}} `appGuid` et `clientSecret` ainsi que des noms d'organisation et d'espace sur lesquels {{site.data.keyword.openwhisk_short}} a été déployé. Pour un hôte d'API, lancez le terminal et exécutez la commande

   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
5. Ouvrez `AppDelegate.swift` et mettez à jour la région du service Push Notifications (BMSClient) et la région AppID en fonction de l'emplacement de création de vos instances de service.
6. Générez le projet.
7. Démarrez l'application sur une unité réelle ou avec un simulateur. 
8. Observez {{site.data.keyword.openwhisk_short}} en arrière-plan en exécutant la commande ci-dessous sur un terminal.
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. Dans l'application, sélectionnez **Connexion** pour vous authentifier avec un compte Facebook ou Google. Une fois connecté, entrez un message de commentaire en retour et appuyez sur le bouton **Envoyer un commentaire**. Quelques secondes après l'envoi du commentaire, vous devriez recevoir une notification push sur l'unité. Le texte de notification peut être personnalisé en modifiant les modèles de documents dans la base de données `moods` de l'instance de service {{site.data.keyword.cloudant_short_notm}}. Utilisez le bouton **View token** pour inspecter les jetons d'accès et d'identification générés par {{site.data.keyword.appid_short}} lors de la connexion.
{: swift}

## Suppression de ressources 

1. Utilisez `deploy.sh` pour supprimer les artefacts {{site.data.keyword.openwhisk_short}} :

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. Supprimez les services {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.appid_short}}, {{site.data.keyword.mobilepushshort}} et {{site.data.keyword.toneanalyzershort}} dans la console {{site.data.keyword.Bluemix_notm}}.

## Contenu associé

* {{site.data.keyword.appid_short}} fournit une configuration par défaut pour vous aider dans la configuration initiale de vos fournisseurs d'identité. Avant de publier votre application, [mettez à jour la configuration avec vos propres données d'identification](https://{DomainName}/docs/services/appid?topic=appid-social#social). Vous pouvez également [personnaliser le widget de connexion](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget). 


* Lorsque vous créez une action OpenWhisk Swift avec un fichier source Swift (fichiers .swift sous le dossier `actions`), vous devez la compiler dans un fichier binaire avant d'exécuter l'action. Cela fait, les appels suivants à l'action sont beaucoup plus rapides jusqu'à ce que le conteneur qui contient votre action soit purgé. Ce délai est dénommé délai de démarrage à froid. Pour éviter ce délai, vous pouvez compiler votre fichier Swift en binaire, puis le télécharger dans OpenWhisk sous forme de fichier zip. Dans la mesure où vous avez besoin de l'échafaudage OpenWhisk, le moyen le plus simple de créer le fichier binaire est de le générer dans le même environnement que celui dans lequel il s'exécute. Pour connaître les étapes supplémentaires, reportez-vous à [Package an Action as a Swift executable](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions).
{: swift}
