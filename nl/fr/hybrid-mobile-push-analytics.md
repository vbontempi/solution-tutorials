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

# Développement d'une application mobile hybride avec Push Notifications
{: #hybrid-mobile-push-analytics}

Découvrez à quel point il est facile de créer rapidement une application Hybrid Cordova avec un service mobile à forte valeur ajoutée, tel que {{site.data.keyword.mobilepushshort}} sur {{site.data.keyword.Bluemix_notm}}.

Apache Cordova est une infrastructure open source de développement mobile. Elle vous permet d'utiliser les technologies Web standard - HTML5, CSS3 et JavaScript pour le développement multiplateforme. Les applications s'exécutent dans des encapsuleurs ciblant chaque plateforme et s'appuient sur des liaisons d'API conformes aux normes pour accéder aux fonctionnalités de chaque unité, telles que capteurs, données, état du réseau, etc. 

Ce tutoriel vous guide dans la création d'une application de démarrage mobile Cordova, l'ajout d'un service mobile, la configuration d'un SDK client, le téléchargement du code échafaudé, puis l'amélioration de l'application. 

## Objectifs

* Créer un projet mobile avec le service {{site.data.keyword.mobilepushshort}}.
* Apprendre à obtenir des données d'identification APN et FCM. 
* Télécharger le code et effectuer la configuration requise. 
* Configurer, envoyer et surveiller {{site.data.keyword.mobilepushshort}}.

 ![](images/solution15/Architecture.png)

## Produits

Ce tutoriel utilise les produits suivants : 
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Avant de commencer
{: #prereqs}

- [Interface CLI](https://cordova.apache.org/docs/en/latest/guide/cli/) Cordova pour l'exécution des commandes Cordova.
- [Conditions préalables](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html) Cordova-iOS et [Conditions préalables](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html) Cordova-Android. 
- Compte Google pour se connecter à la console Firebase afin d'obtenir l'ID de l'expéditeur et la clé d'API du serveur. 
- Compte [Apple Developers](https://developer.apple.com/) pour envoyer des notifications à distance à partir d'une instance de service {{site.data.keyword.mobilepushshort}} sur {{site.data.keyword.Bluemix_notm}} (le fournisseur) vers des appareils et des applications iOS.
- Xcode et Android Studio pour importer et améliorer votre code.

## Création d'un projet mobile Cordova à partir du kit de démarrage 
{: #get_code}
Le tableau de bord {{site.data.keyword.Bluemix_notm}} Mobile Dashboard vous permet d'accélérer le développement de votre application mobile en créant votre projet à partir d'un kit de démarrage.
1. Accédez à [Mobile Dashboard](https://{DomainName}/developer/mobile/dashboard).
2. Cliquez sur **Kits de démarrage**, puis sur **Créer une appli**.
    ![](images/solution15/mobile_dashboard.png)
3. Entrez un nom de projet, il peut également s'agir du nom de votre application. 
4. Sélectionnez **Cordova** comme plateforme et cliquez sur **Créer**.

    ![](images/solution15/create_cordova_project.png)
5. Cliquez sur **Ajouter une ressource** > Mobile > **Push Notifications** et sélectionnez l'emplacement où vous souhaitez mettre à disposition le service, le groupe de ressources et le forfait **Lite**.
6. Cliquez sur **Créer** pour mettre le service {{site.data.keyword.mobilepushshort}} à disposition. Une application est créée sous l'onglet **Applications**.

    **Remarque :** le service {{site.data.keyword.mobilepushshort}} doit être ajouté à l'application de démarrage vide.

A l'étape suivante, vous téléchargez le code échafaudé et effectuez la configuration requise. 

## Téléchargement du code et réalisation de la configuration requise 
{: #download_code}

Si vous n'avez pas encore téléchargé le code, utilisez le tableau de bord {{site.data.keyword.Bluemix_notm}} Mobile Dashboard pour obtenir le code en cliquant sur le bouton **Télécharger le code** sous Projets > **Votre projet mobile**.


1. Dans l'environnement IDE de votre choix, accédez à `/platforms/android/project.properties` et remplacez les deux dernières lignes (library.1 and library.2) par les lignes ci-dessous

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
Les modifications ci-dessus sont spécifiques à Android
  {: tip}
2. Pour lancer l'application sur un émulateur Android, exécutez la commande ci-dessous dans un terminal ou une invite de commande
```
 $ cordova build android
 $ cordova run android
```
 Si une erreur apparaît, lancez un émulateur et tentez d’exécuter la commande ci-dessus.
 {: tip}
3. Pour prévisualiser l’application dans le simulateur iOS, exécutez les commandes ci-dessous dans un terminal,

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    Pour consulter le nom de votre projet dans le fichier `config.xml`, exécutez la commande `cordova info`.
    {: tip}

## Obtention des données d'identification FCM et APN 
{: #obtain_fcm_apns_credentials}

 ### Configuration de Firebase Cloud Messaging (FCM)

  1. Dans la [console Firebase](https://console.firebase.google.com), créez un projet. Définissez le nom sur **hybridmobileapp**
  2. Accédez aux **Paramètres** du projet
  3. Sous l'onglet **Général**, ajoutez deux applications :
       1. une dont le nom de package est défini à : **com.ibm.mobilefirstplatform.clientsdk.android.push**
       2. et une dont le nom de package est défini à : **io.cordova.hellocordovastarter**
  4. Recherchez l'ID de l'expéditeur et la clé du serveur (également appelée clé d'API par la suite) sous l'onglet **Cloud Messaging**.
  5. Dans le tableau de bord du service {{site.data.keyword.mobilepushshort}}, définissez la valeur de l'ID de l'expéditeur et de la clé d'API.
   


Pour connaître les étapes détaillées, reportez-vous à [Obtention des données d'identification FCM](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials).
{: tip}

### Configuration du service Apple {{site.data.keyword.mobilepushshort}} (APN) 

  1. Accédez au portail [Apple Developer](https://developer.apple.com/) et enregistrez un ID d'application.
  2. Créez un certificat SSL APN pour le développement et la distribution
  3. Créez un profil de mise à disposition pour le développement. 
  4. Configurez l'instance de service {{site.data.keyword.mobilepushshort}} sur {{site.data.keyword.Bluemix_notm}}. 

Pour connaître les étapes détaillées, reportez-vous à [Obtention des données d'identification du service APN et configuration du service {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials).
{: tip}

## Configuration, envoi et surveillance de {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. Dans index.js, sous la fonction `onDeviceReady`, remplacez les valeurs `{pushAppGuid}` et 

   `{pushClientSecret} `par les **données d'identification** du service Push Notifications, *appGuid* et *clientSecret*.

2. Accédez au tableau de bord Mobile Dashboard > Projets > Projet Cordova, cliquez sur le service {{site.data.keyword.mobilepushshort}} et effectuez les étapes ci-dessous. 

### Configuration de l'instance de service APN

Pour envoyer des notifications à l'aide du service {{site.data.keyword.mobilepushshort}} , téléchargez les certificats .p12 que vous avez créés à l'étape précédente. Ce certificat contient la clé privée et les certificats SSL requis pour construire et publier votre application.

**Remarque** : une fois que le fichier `.cer` se trouve dans votre accès à la chaîne de certificats, exportez-le sur votre ordinateur pour créer un certificat `.p12`.

1. Cliquez sur `{{site.data.keyword.mobilepushshort}}` sous la section Services ou cliquez sur les trois points verticaux en regard du service {{site.data.keyword.mobilepushshort}} et sélectionnez `Ouvrir le tableau de bord`.
2. Sur le tableau de bord de {{site.data.keyword.mobilepushshort}}, l'option `Configurer` devrait apparaître sous `Gérer > Envoyer des notifications`.

Pour configurer le service APN sur la console des services `Push Notifications`, procédez comme suit :

1. Sélectionnez `Configurer` sur le tableau de bord du service Push Notification.
2. Sélectionnez l'`option Mobile` pour mettre à jour les informations dans le formulaire Données d'identification push APN.
3. Sélectionnez `Bac à sable (développement)` ou `Production (distribution)` selon le cas et envoyez le certificat `p.12` que vous avez créé.
4. Dans la zone Mot de passe, entrez le mot de passe associé au fichier de certificat .p12, puis cliquez sur Sauvegarder.

### Configuration de l'instance de service FCM

1. Sélectionnez **Mobile**, puis mettez à jour l'onglet Données d'identification push GCM/FCM avec l'ID d'émetteur/Numéro de projet et la clé d'API que vous avez créés au début dans la console Firebase.
2. Cliquez sur **Sauvegarder**. Le service {{site.data.keyword.mobilepushshort}} est à présent configuré.

### Envoi de notifications {{site.data.keyword.mobilepushshort}}

1. Sélectionnez **Envoyer des notifications** et rédigez un message en choisissant une option d'envoi. Les options prises en charge sont Appareil par étiquette, ID appareil, ID utilisateur, Appareils Android, Appareils IOS, Notifications web, Tous les appareils.
**Remarque **: quand vous sélectionnez l'option **Tous les appareils**, tous les appareils qui sont abonnés au service {{site.data.keyword.mobilepushshort}} reçoivent des notifications.


2. Dans la zone **Message**, composez votre message. Configurez les paramètres facultatifs, selon les besoins.
3. Cliquez sur **Envoyer** et vérifiez que votre appareil physique a reçu la notification.

### Surveillance des notifications envoyées

Vous pouvez surveiller les notifications que vous avez envoyées en accédant à la section **Surveillance**.
Le service IBM {{site.data.keyword.mobilepushshort}} étend désormais les capacités de surveillance des performances push en générant des graphiques depuis vos données utilisateur. Cet utilitaire vous permet de répertorier toutes les notifications {{site.data.keyword.mobilepushshort}} envoyées ou tous les appareils enregistrés et d'analyser les informations sur une base quotidienne, hebdomadaire ou mensuelle.
![](images/solution6/monitoring_messages.png) 

## Contenu associé
{: #related_content}

- [Notifications basées sur les balises](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sécurité de {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

