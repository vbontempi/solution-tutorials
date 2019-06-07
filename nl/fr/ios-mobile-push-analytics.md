---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-08"
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

# Développement d'une application mobile iOS avec Push Notifications
{: #ios-mobile-push-analytics}

Découvrez à quel point il est facile de créer rapidement une application iOS Swift avec un service mobile à forte valeur ajoutée, tel que {{site.data.keyword.mobilepushshort}} sur {{site.data.keyword.Bluemix_short}}.

Ce tutoriel vous guide dans la création d'une application de démarrage mobile, l'ajout d'un service mobile, la configuration d'un SDK client, l'importation du code dans Xcode, puis l'amélioration de l'application. {:shortdesc: .shortdesc}

## Objectifs
{:#objectives}

- Créer une application mobile avec les services {{site.data.keyword.mobilepushshort}} et {{site.data.keyword.mobileanalytics_short}} à partir du kit de démarrage Basic Swift. 
- Obtenir les données d'identification du service APN et configurer l'instance de service {{site.data.keyword.mobilepushshort}}.
- Télécharger le code et configurer le SDK client. 
- Envoyer et surveiller des notifications {{site.data.keyword.mobilepushshort}}.

  ![](images/solution6/Architecture.png)

## Produits
{:#products}

Ce tutoriel utilise les produits suivants : 
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Avant de commencer
{: #prereqs}

1. Compte [Apple Developers](https://developer.apple.com/) pour envoyer des notifications à distance à partir d'une instance de service {{site.data.keyword.mobilepushshort}} sur {{site.data.keyword.Bluemix_short}} (le fournisseur) vers des appareils et des applications iOS.
2. Xcode pour importer et améliorer votre code.

## Création d'une application mobile a à partir du kit de démarrage Basic Swift 
{: #get_code}

1. Accédez à [Mobile Dashboard](https://{DomainName}/developer/mobile/dashboard) pour créer votre `application` à partir de `kits de démarrage` prédéfinis.
2. Cliquez sur **Kits de démarrage** et faites défiler la liste vers le bas pour sélectionner le kit de démarrage **De base**.
    ![](images/solution6/mobile_dashboard.png)
3. Entrez un nom d'application qui sera également le nom du projet et de l'application Xcode .
4. Sélectionnez `iOS Swift` comme plateforme et cliquez sur **Créer**.
    ![](images/solution6/create_mobile_project.png)
5. Cliquez sur **Ajouter une ressource** > Mobile > **Push Notifications** et sélectionnez l'emplacement où vous souhaitez mettre à disposition le service, le groupe de ressources et le forfait **Lite**.
6. Cliquez sur **Créer** pour mettre le service {{site.data.keyword.mobilepushshort}} à disposition. Une application est créée sous l'onglet **Applications**.

      **Remarque :** le service {{site.data.keyword.mobilepushshort}} doit déjà être ajouté à l'application de démarrage vide. 

## Téléchargement du code et configuration des SDK client 
{: #download_code}

Si vous n'avez pas encore téléchargé le code, cliquez sur `Télécharger le code` sous Applications > `Votre application mobile`. Le code téléchargé est fourni avec le SDK client **{{site.data.keyword.mobilepushshort}}**. Le SDK client est disponible sur CocoaPods et Carthage. Pour cette solution, vous allez utiliser CocoaPods.

1. Pour installer CocoaPods sur votre poste, ouvrez le `Terminal` et exécutez la commande ci-dessous. 
   ```
   sudo gem install cocoapods
   ```
   {: pre:}
2. Décompressez le code téléchargé et à l'aide du terminal, accédez au dossier décompressé 

   ```
   cd <Name of Project>
   ```
   {: pre:}
3. Le dossier comprend déjà un `fichier pod` avec les dépendances requises. Exécutez la commande ci-dessous pour installer les dépendances (SDK client). 

  ```
  pod install
  ```
  {: pre:}

## Obtenir les données d'identification du service APN et configurer l'instance de service {{site.data.keyword.mobilepushshort}}.
{: #obtain_apns_credentials}

   Pour les appareils et applications iOS, le service APN (Apple Push Notification) permet aux développeurs d'application d'envoyer des notifications à distance depuis une instance de service {{site.data.keyword.mobilepushshort}} sur {{site.data.keyword.Bluemix_short}} (le fournisseur) vers des appareils et applications iOS. Les messages sont envoyés à une application cible sur l'appareil.

   Vous devez vous procurer et configurer vos données d'identification APN. Les certificats APN sont gérés de façon sécurisée par le service {{site.data.keyword.mobilepushshort}} et utilisés pour la connexion au serveur APN en tant que fournisseur.

### Enregistrement d'un ID d'appli

   L'ID d'application (l'identificateur de bundle) est un identificateur unique identifiant une application spécifique. Chaque application
requiert un ID d'application. Les services tels que le service {{site.data.keyword.mobilepushshort}} sont configurés avec l'ID
d'application.
   Vérifiez que vous disposez d'un compte [Apple Developer](https://developer.apple.com/). Ce prérequis est impératif.

   1. Accédez au portail [Apple Developer](https://developer.apple.com/), cliquez sur `Member Center`, puis sélectionnez `Certificates, IDs & Profiles`.
   2. Accédez à la section `Identifiers` > App IDs.
   3. Sur la page `Registering App IDs`, indiquez le nom de l'application dans la zone App ID Description Name. Par exemple : ACME {{site.data.keyword.mobilepushshort}}.Indiquez une chaîne pour le préfixe d'ID d'application.
   4. Pour le suffixe d'ID d'application, sélectionnez `Explicit App ID` et indiquez une valeur d'ID de bundle. Il est recommandé de fournir un nom de domaine inversé de type chaîne. Par exemple : com.ACME.push.
   5. Cochez la case `{{site.data.keyword.mobilepushshort}}` et cliquez sur `Continue`.
   6. Passez en revue vos paramètres, puis cliquez sur `Register` > `Done`.
     Votre ID d'application est à présent enregistré.

     ![](images/solution6/push_ios_register_appid.png)

### Création d'un certificat SSL APN pour le développement et la distribution
   Pour pouvoir obtenir un certificat APN, vous devez d'abord générer une demande de signature de certificat et la soumettre à Apple, l'autorité de certification. La demande de signature de certificat contient des informations qui identifient votre société, ainsi que la clé publique et la clé privée que vous utilisez pour vous connecter à Apple {{site.data.keyword.mobilepushshort}}. Ensuite, générez le certificat SSL dans le portail des développeurs iOS. Le certificat, avec sa clé publique et sa clé privée, est stocké dans Keychain Access.
   Vous pouvez utiliser APN dans deux modes :

- Mode bac à sable pour le développement et le test.
- Mode production lors de la distribution des applications via l'App Store (ou d'autres mécanismes de distribution d'entreprise).

   Vous devez vous procurer des certificats distincts pour vos environnements de développement et de distribution. Les certificats sont
associés à un ID d'application pour l'application qui est le destinataire des notifications distantes. Pour la production, vous pouvez créer jusqu'à deux certificats. {{site.data.keyword.Bluemix_short}} utilise les certificats afin d'établir une connexion SSL à APN.

   1. Accédez au site Apple Developer, cliquez sur **Member Center**, puis sélectionnez **Certificates, IDs & Profiles**.
   2. Dans la zone **Identifiers**, cliquez sur **App IDs**.
   3. Dans la liste des ID d'application, sélectionnez votre ID d'application, puis `Edit`.
   4. Sélectionnez la case **{{site.data.keyword.mobilepushshort}}**, puis dans le volet **Development SSL certificate**, cliquez sur **Create Certificate**.

     ![Certificats SSL de Push Notifications](images/solution6/certificate_createssl.png)

   5. Quand l'écran **About Creating a Certificate Signing Request (CSR)** s'affiche, suivez les instructions pour créer un fichier de demande de signature de certificat (Certificate Signing Request - CSR). Attribuez un nom significatif qui permet d'identifier s'il s'agit d'un certificat pour le développement (bac à sable) ou pour la distribution (production), par exemple, sandbox-apns-certificate ou production-apns-certificate. 
   6. Cliquez sur **Continue** et, dans l'écran Upload CSR file, cliquez sur **Choose File**, puis sélectionnez le fichier **CertificateSigningRequest.certSigningRequest** que vous venez de créer. Cliquez sur **Continue**.
   7. Sur le panneau Download, Install and Backup, cliquez sur Download. Le fichier **aps_development.cer** est téléchargé.
        ![Téléchargement du certificat](images/solution6/push_certificate_download.png)   

        ![Génération du certificat](images/solution6/generate_certificate.png)
   8. Sur votre poste mac, ouvrez **Trousseaux d'accès**, **Fichier**, **Importer** et sélectionnez le fichier .cer téléchargé pour l'installer.
   9. Cliquez avec le bouton droit sur le nouveau certificat et la clé privée, puis sélectionnez **Exporter** et définissez le **Format de fichier** sur le format pfx (Personal inFormation eXchange) (format `.p12`).
     ![Exportation du certificat et des clés](images/solution6/keychain_export_key.png)
   10. Dans la zone **Enregistrer sous**, donnez au certificat un nom significatif, par exemple `sandbox_apns.p12` ou **production_apns.p12**, puis cliquez sur Enregistrer.
     ![Exportation du certificat et des clés](images/solution6/certificate_p12v2.png)
   11. Dans la zone **Mot de passe**, entrez un mot de passe pour protéger les éléments exportés, puis cliquez sur OK. Vous pouvez utiliser ce mot de passe pour configurer vos paramètres APN sur la console du service {{site.data.keyword.mobilepushshort}}.
       ![Exportation du certificat et des clés](images/solution6/export_p12.png)
   12. **Key Access.app** vous invite à exporter votre clé depuis l'écran **Trousseaux**. Entrez le mot de passe administrateur pour votre Mac afin de permettre au système d'exporter ces éléments puis sélectionnez l'option `Toujours autoriser`. Un certificat `.p12` est généré sur votre bureau.

      Pour un certificat SSL de production, dans le volet **Production SSL certificate**, cliquez sur **Create Certificate** et répétez les 5 à 12 ci-dessus.
      {:tip}

### Création d'un profil de mise à disposition pour le développement
   Le profil de mise à disposition est utilisé conjointement avec l'ID d'application pour déterminer quels sont les appareils qui peuvent installer et exécuter votre application et quels sont les services auxquels votre application peut accéder. Pour chaque ID d'application, vous créez deux profils de mise à disposition : un pour le développement et un pour la distribution. Xcode utilise le profil de mise à disposition pour le développement afin de déterminer quels sont les développeurs qui sont autorisés à construire l'application et quels sont les appareils qui peuvent être testés avec l'application.

   Prenez soin d'enregistrer un ID d'application, de l'activer pour le service {{site.data.keyword.mobilepushshort}} et de le configurer pour utiliser un certificat SSL APN à des fins de développement et de production.

   Créez un profil de mise à disposition pour le développement, comme suit :

   1. Accédez au portail [Apple Developer](https://developer.apple.com/), cliquez sur `Member Center`, puis sélectionnez `Certificates, IDs & Profiles`.
   2. Accédez à [Mac Developer Library](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site), faites défiler jusqu'à la section `Creating Development Provisioning Profiles` et suivez les instructions de création d'un profil de développement.
     **Remarque** : lorsque vous configurez un profil de mise à disposition pour le développement, sélectionnez les options suivantes :

     - **iOS App Development**
     - **For iOS and watchOS apps **

### Création d'un profil de mise à disposition pour la distribution dans l'App Store
   Utilisez le profil de mise à disposition dans l'App Store afin de soumettre votre application pour la distribution dans l'App Store.

   1. Accédez au site [Apple Developer](https://developer.apple.com/), cliquez sur `Member Center`, puis sélectionnez `Certificates, IDs & Profiles`.
   2. Cliquez deux fois sur le profil de mise à disposition téléchargé afin de l'installer dans Xcode.
     Après l'obtention des données d'identification, l'étape suivante consiste à [Configurer une instance de service](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2).

### Configuration
de l'instance
de service

   Pour envoyer des notifications à l'aide du service {{site.data.keyword.mobilepushshort}} , téléchargez les certificats .p12 que vous avez créés à l'étape précédente. Ce certificat contient la clé privée et les certificats SSL requis pour construire et publier votre application.

   **Remarque** : une fois le fichier `.cer` dans votre accès à la chaîne de certificats, exportez-le sur votre ordinateur pour créer un certificat `.p12`.

1. Cliquez sur {{site.data.keyword.mobilepushshort}} sous la section Services ou cliquez sur les trois points verticaux en regard du service {{site.data.keyword.mobilepushshort}} et sélectionnez `Ouvrir le tableau de bord`.
2. Sur le tableau de bord de {{site.data.keyword.mobilepushshort}}, l'option `Configurer` devrait apparaître sous `Gérer`.

Pour configurer le service APN sur la console des services `Push Notifications`, procédez comme suit :

1. Sélectionnez l'`option Mobile` pour mettre à jour les informations dans le formulaire Données d'identification push APN.
2. Sélectionnez `Serveur APN de bac à sable/développement` ou `Serveur APN de production` selon vos besoins, puis chargez le certificat `.p12` que vous avez créé.
3. Dans la zone Mot de passe, entrez le mot de passe associé au fichier de certificat .p12, puis cliquez sur Sauvegarder.

![](images/solution6/Mobile_push_configure.png)

## Configuration, envoi et surveillance de notifications {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. Le code d’initialisation d'envoi (sous l'`application func`) et le code d’enregistrement de notification sont disponibles dans `AppDelegate.swift`. Indiquez un USER_ID unique (facultatif). 
2. Exécutez l'application sur un appareil physique car les notifications ne peuvent pas être envoyées à un simulateur d'iPhone. 
3. Ouvrez le service {{site.data.keyword.mobilepushshort}} sous `Services mobiles` > **Services existants** sur le tableau de bord {{site.data.keyword.Bluemix_short}} Mobile Dashboard et, pour envoyer des notifications {{site.data.keyword.mobilepushshort}} de base, procédez comme suit :
  * Sélectionnez `Messages` et rédigez un message en choisissant l'option Envoyer à. Les options prises en charge sont Appareil par étiquette, ID appareil, ID utilisateur, Appareils Android, Appareils IOS, Notifications Web, Tous les appareils et autres navigateurs.

       **Remarque **: quand vous sélectionnez l'option Tous les appareils, tous les appareils qui sont abonnés au service {{site.data.keyword.mobilepushshort}} reçoivent des notifications.
  * Dans la zone `Message`, composez votre message. Configurez les paramètres facultatifs, selon les besoins.
  * Cliquez sur `Envoyer` et vérifiez que les appareils physiques ont reçu la notification.
    ![](images/solution6/send_notifications.png)

4. Une notification devrait apparaître sur votre iPhone.

   ![](images/solution6/iphone_notification.png)

5. Vous pouvez surveiller les notifications que vous avez envoyées en accédant à `Surveillance` dans le service {{site.data.keyword.mobilepushshort}}.

Le service IBM {{site.data.keyword.mobilepushshort}} étend désormais les capacités de surveillance des performances push en générant des graphiques depuis vos données utilisateur. Cet utilitaire vous permet de répertorier toutes les notifications {{site.data.keyword.mobilepushshort}} envoyées ou tous les appareils enregistrés et d'analyser les informations sur une base quotidienne, hebdomadaire ou mensuelle.
![](images/solution6/monitoring_messages.png) 

## Contenu associé
{: #related_content}

- [Notifications basées sur les balises](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Sécurité de {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)
