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

# Développement d'une application mobile native Android avec Push Notifications
{: #android-mobile-push-analytics}

Découvrez à quel point il est facile de créer rapidement une application Android native avec un service mobile à forte valeur ajoutée, tel que {{site.data.keyword.mobilepushshort}} sur {{site.data.keyword.Bluemix_notm}}.

Ce tutoriel vous guide dans la création d'une application de démarrage mobile, l'ajout d'un service mobile, la configuration d'un SDK client, l'importation du code dans Android Studio, puis l'amélioration de l'application. 

## Objectifs
{: #objectives}

* Créer une application mobile avec le service {{site.data.keyword.mobilepushshort}}.
* Obtenir les données d'identification FCM.
* Télécharger le code et effectuer la configuration requise. 
* Configurer, envoyer et surveiller {{site.data.keyword.mobilepushshort}}.

![](images/solution9/Architecture.png)

## Produits
{: #products}

Ce tutoriel utilise les produits suivants : 
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Avant de commencer
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html) pour importer et améliorer votre code.
- Compte Google pour se connecter à la console Firebase afin d'obtenir l'ID de l'expéditeur et la clé d'API du serveur. 

## Création d'une application mobile Android à partir du kit de démarrage 
{: #get_code}
Le tableau de bord {{site.data.keyword.Bluemix_notm}} Mobile Dashboard vous permet d'accélérer le développement de votre application mobile en créant votre application à partir d'un kit de démarrage.
1. Accédez à [Mobile Dashboard](https://{DomainName}/developer/mobile/dashboard)
2. Cliquez sur **Kits de démarrage**, puis sur **Créer une appli**.
    ![](images/solution9/mobile_dashboard.png)
3. Entrez un nom d'applications, il peut également s'agir du nom de votre projet Android. 
4. Sélectionnez **Android** comme plateforme et cliquez sur **Créer**.

    ![](images/solution9/create_mobile_project.png)
5. Cliquez sur **Ajouter une ressource** > Mobile > **Push Notifications** et sélectionnez l'emplacement où vous souhaitez mettre à disposition le service, le groupe de ressources et le forfait **Lite**.
6. Cliquez sur **Créer** pour mettre le service {{site.data.keyword.mobilepushshort}} à disposition. Une application est créée sous l'onglet **Applications**.

    **Remarque :** le service {{site.data.keyword.mobilepushshort}} doit être ajouté à l'application de démarrage vide. Dans l'étape suivante, vous obtenez des données d'identification Firebase Cloud Messaging (FCM). 

A l'étape suivante, vous téléchargez le code échafaudé et configurez le SDK Push Android. 

## Téléchargement du code et réalisation de la configuration requise 
{: #download_code}

Si vous n'avez pas encore téléchargé le code, utilisez le tableau de bord {{site.data.keyword.Bluemix_notm}} Mobile Dashboard pour obtenir le code en cliquant sur le bouton **Télécharger le code** sous Applications > **Votre application mobile**.
Le code téléchargé est fourni avec le SDK client **{{site.data.keyword.mobilepushshort}}** inclus. Le SDK client est disponible sur Gradle et Maven. Pour ce tutoriel, vous allez utiliser **Gradle**.

1. Lancez Android Studio > **Ouvrez un projet Android Studio existant** et pointez vers le code téléchargé.
2. La génération de **Gradle** est automatiquement déclenchée et toutes les dépendances sont téléchargées.
3. Ajoutez la dépendance des **services Google Play** à votre fichier `build.gradle (Module: app)` au niveau du module, à la fin, après les `dépendances{.....}`
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. Copiez le fichier `google-services.json` que vous avez créé et téléchargé dans le répertoire racine de votre module d'application Android. Vous pouvez constater que le fichier `google-service.json` inclut les noms de packages que vous avez ajouté.
5. Les droits requis se trouvent tous dans le fichier `AndroidManifest.xml` et ses dépendances. Push et Analytics sont inclus dans **build.gradle (Module: app)**.
6. Le service d'intention **Firebase Cloud Messaging (FCM)** et les filtres d'intention pour les notifications d'événements `RECEIVE` et `REGISTRATION` sont inclus dans `AndroidManifest.xml`

## Obtention des données d'identification FCM
{: #obtain_fcm_credentials}

Firebase Cloud Messaging (FCM) est la passerelle utilisée pour envoyer des {{site.data.keyword.mobilepushshort}} aux appareils Android, au navigateur Google Chrome et aux applications et extensions Chrome. Pour configurer le service {{site.data.keyword.mobilepushshort}} sur la console, vous devez vous procurer vos données d'identification FCM (ID d'émetteur et clé d'API).

La clé d'API est stockée de façon sécurisée et utilisée par le service {{site.data.keyword.mobilepushshort}} pour la connexion au serveur FCM. L'ID d'émetteur (numéro de projet) est utilisé par le logiciel SDK Android et le logiciel SDK JS pour Google Chrome et Mozilla Firefox côté client. Pour installer FCM et obtenir vos données d'identification, procédez comme suit :

1. Consultez la [Console Firebase](https://console.firebase.google.com/?pli=1)  -  Un compte utilisateur Google est requis.
2. Sélectionnez **Ajouter un projet**.
3. Dans la fenêtre **Créer un projet**, indiquez un nom de projet, choisissez un pays/une région et cliquez sur **Créer un projet**.
4. Dans le volet de navigation de gauche, sélectionnez **Paramètres** (Cliquez sur l’icône Paramètres en regard de **Présentation**)> **Paramètres du projet**.
5. Sélectionnez l'onglet Cloud Messaging pour obtenir vos données d'identification de projet : Clé d'API du serveur et ID d'émetteur.
    **Remarque :** la clé de serveur répertoriée dans FCM est identique à la clé d'API du serveur.
    ![](images/solution9/fcm_console.png)

Vous devez également générer le fichier `google-services.json`. Exécutez les étapes suivantes :

1. Dans la console Firebase, cliquez sur l’icône **Paramètres du projet** > **Général** sous le projet que vous avez créé ci-dessus et sélectionnez **Add Firebase to your Android App**

    ![](images/solution9/firebase_project_settings.png)
2. Dans la fenêtre **Add Firebase to your Android**, ajoutez **com.ibm.mobilefirstplatform.clientsdk.android.push** en tant que nom de package pour enregistrer le SDK Android {{site.data.keyword.mobilepushshort}}. Les zones App nickname et SHA-1 sont facultatives. Cliquez sur **REGISTER APP** > **Continue** > **Finish**.

    ![](images/solution9/add_firebase_to_your_app.png)

3. Cliquez sur **ADD APP** > **Add Firebase to your app**. Incluez le nom de package de votre application en l'entrant dans **com.ibm.mysampleapp** et poursuivez dans la fenêtre Add Firebase to your Android App. Les zones App nickname et SHA-1 sont facultatives. Cliquez sur **REGISTER APP** > Continue > Finish.
     **Remarque :** vous pouvez trouver le nom du package de votre application dans le fichier `AndroidManifest.xml` une fois le code téléchargé.
4. Téléchargez le dernier fichier de configuration `google-services.json` sous **Your apps**.

    ![](images/solution9/google_services.png)

    **Remarque **: FCM est la nouvelle version de Google Cloud Messaging (GCM). Prenez soin d'utiliser des données d'identification FCM pour les nouvelles applications. Les appli existantes continueraient à fonctionner avec les configurations GCM.

*Les étapes et l'interface utilisateur de la console Firebase sont susceptibles d'être modifiées. Reportez-vous à la documentation Google pour la partie Firebase si nécessaire.*

## Configuration, envoi et surveillance de {{site.data.keyword.mobilepushshort}}

{: #configure_push}

1. Le SDK {{site.data.keyword.mobilepushshort}} est déjà importé dans l'application et le code d'initialisation Push se trouve dans le fichier `MainActivity.java`.

    **Remarque :** les données d'identification du service font partie du fichier `/res/values/credentials.xml`.
2. L'enregistrement pour les notifications s'effectue dans `MainActivity.java`. (Facultatif) Indiquez un USER_ID unique.
3. Exécutez l'application sur un appareil physique ou un émulateur pour recevoir des notifications. 
4. Ouvrez le service {{site.data.keyword.mobilepushshort}} sous **Services mobiles** > **Services existants** sur le tableau de bord {{site.data.keyword.Bluemix_notm}} Mobile Dashboard et, pour envoyer des notifications {{site.data.keyword.mobilepushshort}} de base, procédez comme suit :
   - Cliquez sur **Gérer ** > **Configurer**.
   - Sélectionnez **Mobile**, puis mettez à jour l'onglet Données d'identification push GCM/FCM avec l'ID d'émetteur/Numéro de projet et la clé d'API que vous avez créés au début dans la console Firebase.

     ![](images/solution9/configure_push_notifications.png)
   - Cliquez sur **Sauvegarder**. Le service {{site.data.keyword.mobilepushshort}} est à présent configuré.
   - Sélectionnez **Envoyer des notifications** et rédigez un message en choisissant une option d'envoi. Les options prises en charge sont Appareil par étiquette, ID appareil, ID utilisateur, Appareils android, Appareils IOS, Notifications web et Tous les appareils.
**Remarque **: quand vous sélectionnez l'option **Tous les appareils**, tous les appareils qui sont abonnés au service {{site.data.keyword.mobilepushshort}} reçoivent des notifications.

   - Dans la zone **Message**, composez votre message. Configurez les paramètres facultatifs, selon les besoins.
   - Cliquez sur **Envoyer** et vérifiez que votre appareil physique a reçu la notification.

     ![](images/solution9/android_send_notifications.png)
5. Une notification devrait apparaître sur votre appareil Android.

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. Vous pouvez surveiller les notifications que vous avez envoyées en accédant à **Surveillance** dans le service {{site.data.keyword.mobilepushshort}}. Le service IBM {{site.data.keyword.mobilepushshort}} étend désormais les capacités de surveillance des performances push en générant des graphiques depuis vos données utilisateur. Cet utilitaire vous permet de répertorier toutes les notifications {{site.data.keyword.mobilepushshort}} envoyées ou tous les appareils enregistrés et d'analyser les informations sur une base quotidienne, hebdomadaire ou mensuelle.
![](images/solution6/monitoring_messages.png) 

## Contenu associé
{: #related_content}
- [Personnalisation des paramètres {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [Notifications basées sur les balises](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sécurité de {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

