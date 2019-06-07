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


# Application Web sans serveur et API
{: #serverless-api-webapp}

Dans ce tutoriel, vous allez créer une application Web sans serveur en hébergeant du contenu Web statique sur GitHub Pages et en implémentant le back-end de l'application à l'aide d'{{site.data.keyword.openwhisk}}.

En tant que plateforme pilotée par événements, {{site.data.keyword.openwhisk_short}} prend en charge toute une [variété de cas d'utilisation](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases). La génération d'applications Web et d'API en fait partie. Avec les applications Web, les événements constituent les interactions entre les navigateurs Web (ou clients REST) et votre application Web, à savoir les requêtes HTTP. Plutôt que de mettre à disposition une machine virtuelle, un conteneur ou un environnement d'exécution Cloud Foundry pour déployer votre back-end, vous pouvez implémenter votre API de back-end avec une plateforme sans serveur. Cette solution permet d'éviter les coûts des temps d'inactivité et de dimensionner la plateforme selon les besoins. 

Toute action (ou fonction) dans {{site.data.keyword.openwhisk_short}} peut être transformée en un point final HTTP prêt à être utilisé par les clients Web. Lorsqu'elles sont activées pour le Web, ces actions sont appelées *actions Web*. Une fois que vous disposez d'actions Web, vous pouvez les assembler en une API complète à l'aide d'API Gateway. API Gateway est un composant de {{site.data.keyword.openwhisk_short}} permettant d'exposer des API. Il offre des fonctionnalités de sécurité, de support OAuth, de limitation de débit, de support de domaine personnalisé. 

## Objectifs

* Déployer un back-end sans serveur et une base de données 
* Exposer une API REST 
* Héberger un site Web statique
* Facultatif : utiliser un domaine personnalisé pour l'API REST 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

L’application présentée dans ce tutoriel est un simple site Web de livre d’or où les utilisateurs peuvent publier des messages. 

<p style="text-align: center;">

   ![Architecture](./images/solution8/Architecture.png)
</p>

1. L’utilisateur accède à l’application hébergée dans GitHub Pages. 
2. L'application Web appelle une API de back-end. 
3. L'API de back-end est définie dans API Gateway. 
4. API Gateway transmet la demande à [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk).
5. Les actions {{site.data.keyword.openwhisk_short}} utilisent [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) pour stocker et extraire des entrées du livre d'or.

## Avant de commencer
{: #prereqs}

Ce guide utilise GitHub Pages pour héberger le site Web statique. Veillez à disposer d'un compte github public.

## Création de la base de données Guestbook 

Commencez par créer {{site.data.keyword.cloudant_short_notm}}. {{site.data.keyword.cloudant_short_notm}} est une couche de données entièrement gérée, conçue pour les applications actuelles Web et mobiles, qui exploite un schéma JSON flexible. {{site.data.keyword.cloudant_short_notm}} repose sur et est compatible avec Apache CouchDB, et est accessible via une API HTTPS sécurisée, qui s'adapte à l'évolution de votre application.

1. Dans le catalogue, sélectionnez **Cloudant**.
2. Définissez le nom du service sur **guestbook-db**, sélectionnez **Use both legacy credentials and IAM** comme méthodes d'authentification, puis cliquez sur **Create**.
3. De retour dans la liste des ressources, cliquez sur l'entrée ***guestbook-db** dans la colonne Nom. Remarque : vous devrez peut-être patienter jusqu'à ce que le service soit mis à disposition. 
4. Dans l'écran des détails du service, cliquez sur ***Launch Cloudant Dashboard*** qui apparaît alors dans un autre onglet du navigateur. Remarque : la connexion peut être requise pour votre instance Cloudant.  
5. Cliquez sur ***Create Database*** et créez une base de données nommée ***guestbook***.

  ![](images/solution8/Create_Database.png)

6. Revenez à l'onglet Détails du service, sous **Données d'identification pour le service**
   1. Créez de **Nouvelles données d'identification**, acceptez les valeurs par défaut et cliquez sur **Ajouter**.
   2. Cliquez sur **Afficher les données d'identification** sous Actions. Vous aurez besoin de ces données d'identification ultérieurement pour permettre aux actions de Cloud Functions de lire/écrire dans votre service Cloudant. 

## Création d'actions sans serveur

Dans cette section, vous allez créer des actions sans serveur (communément appelées fonctions). {{site.data.keyword.openwhisk}} (basé sur Apache OpenWhisk) est une plateforme de fonction en tant que service (FaaS) qui exécute des fonctions en réponse à des événements entrants et ne génère aucun coût lorsqu'elle n'est pas utilisée.

![](images/solution8/Functions.png)

### Séquence d'actions pour sauvegarder une entrée du livre d'or 

Créez une **séquence**, à savoir une chaîne d'actions dans laquelle la sortie d'une action constitue une entrée pour l'action suivante et ainsi de suite. La première séquence que vous créez est utilisée pour conserver un message du livre d'or. Indiquez un nom, une adresse e-mail et un commentaire pour permettre à la séquence de : 
   * Créer un document à conserver.
   * Stocker le document dans la base de données {{site.data.keyword.cloudant_short_notm}}.

Commencez par créer la première action : 

1. Accédez à **Fonctions** https://{DomainName}/openwhisk.
2. Dans le volet de gauche, cliquez sur **Actions**, puis sur **Créer**.
3. **Créez une action** avec le nom `prepare-entry-for-save` et sélectionnez **Node.js** pour Exécution (remarque : sélectionnez la dernière version).
4. Remplacez le code existant par le fragment de code ci-dessous :  
   ```js
   /**
    * Prepare the guestbook entry to be persisted
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. **Sauvegardez**

Ajoutez ensuite l'action à une séquence : 

1. Cliquez sur **Séquences de regroupement**, puis sur **Ajouter à la séquence**.
1. Pour le nom de la séquence, entrez `save-guestbook-entry-sequence`, puis cliquez sur **Créer et ajouter**.

Enfin, ajoutez une deuxième action à la séquence : 

1. Cliquez sur **save-guestbook-entry-sequence**, puis cliquez sur **Ajouter**.
1. Sélectionnez **Action publique**, **Cloudant**, puis choisissez **create-document** sous **Actions**
1. Créez une **Nouvelle liaison** 
1. Pour Nom, entrez `binding-for-guestbook`
1. Pour Instance Cloudant, sélectionnez `Input your own credentials` et renseignez les zones suivantes avec les données d'identification capturées pour votre service Cloudant : Nom d'utilisateur, Mot de passe, Hôte et Base de données = `guestbook`, cliquez sur **Ajouter**, puis sur **Sauvegarder**.
   {: tip}
1. Pour effectuer un test, cliquez sur **Changer l'entrée**, puis entrez le code JSON ci-dessous
    ```json
    {
      "name": "John Smith",
      "email": "john@smith.com",
      "comment": "this is my comment"
    }
    ```
    {: codeblock}
1. **Appliquez**, puis **Appelez**.
    ![](images/solution8/Save_Entry_Invoke.png)

### Séquence d'actions pour extraire des entrées

La deuxième séquence est utilisée pour extraire les entrées existantes du livre d'or. Cette séquence permet de : 
   * Répertorier tous les documents de la base de données. 
   * Formater les documents et les renvoyer. 

1. Sous **Fonctions**, cliquez sur **Actions**, puis sur **Créer** une action Node.js et nommez-la `set-read-input`.
2. Remplacez le code existant par le fragment de code ci-dessous. Cette action transmet les paramètres appropriés à l'action suivante. 
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. **Sauvegardez**

Ajoutez l'action à une séquence :

1. Cliquez sur **Séquences de regroupement**, **Ajouter à la séquence** et **Création**
1. Entrez `read-guestbook-entries-sequence` pour **Nom de l'action** et cliquez sur **Créer et ajouter**.

Effectuez la séquence :

1. Cliquez sur la séquence **read-guestbook-entries-sequence**, puis cliquez sur **Ajouter** pour créer et ajouter la deuxième action afin d'obtenir des documents auprès de Cloudant.
1. Sous **Action publique**, choisissez **{{site.data.keyword.cloudant_short_notm}}**, puis **list-documents**
1. Sous **Mes liaisons**, choisissez **binding-for-guestbook** et cliquez sur **Ajouter** pour créer et ajouter cette action publique à votre séquence.
1. Cliquez à nouveau sur **Ajouter** pour créer et ajouter la troisième action qui formate les documents provenant de {{site.data.keyword.cloudant_short_notm}}.
1. Sous **Création**, entrez `format-entries` pour le nom, puis cliquez sur **Créer et ajouter**.
1. Cliquez sur **format-entries** et remplacez le code par le code ci-dessous : 
   ```js
   const md5 = require('spark-md5');

   function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. **Sauvegardez**
1. Choisissez la séquence en cliquant sur **Actions**, puis sur **read-guestbook-entries-sequence**.
1. Cliquez sur **Sauvegarder**, puis sur **Appeler**. La sortie doit se présenter comme suit :
   ![](images/solution8/Read_Entries_Invoke.png)

## Création d'une API

![](images/solution8/Cloud_Functions_API.png)

1. Accédez à Actions https://{DomainName}/openwhisk/actions.
2. Sélectionnez la séquence **read-guestbook-entries-sequence**. En regard du nom, cliquez sur **Action Web**, cochez **Activer une action Web** et **Sauvegarder**.
3. Procédez de même pour la séquence **save-guestbook-entry-sequence**.
4. Accédez aux API https://{DomainName}/openwhisk/apimanagement et **créez une API {{site.data.keyword.openwhisk_short}}**
5. Définissez le nom sur `guestbook` et le chemin d'accès sur `/guestbook`
6. Cliquez sur **Créer une opération** et créez une opération pour extraire les entrées du livre d'or :
   1. Définissez **path** sur `/entries`
   2. Définissez **verb** sur `GET*`
   3. Sélectionnez l'action **read-guestbook-entries-sequence**
7. Cliquez sur **Créer une opération** et créez une opération pour conserver une entrée du livre d'or :
   1. Définissez **path** sur `/entries`
   2. Définissez **verb** sur `PUT`
   3. Sélectionnez l'action **save-guestbook-entry-sequence**
8. Sauvegardez et exposez l'API. 

## Déploiement de l'application Web 

1. Faites dériver le référentiel d'interface utilisateur Guestbook, https://github.com/IBM-Cloud/serverless-guestbook, vers le GitHub public.
2. Modifiez le fichier **docs/guestbook.js** et remplacez la valeur d'**apiUrl** par la route indiquée par API Gateway.
3. Validez le fichier modifié dans votre référentiel dérivé. 
4. Dans la page Paramètres de votre référentiel, faites défiler l'écran jusqu'à **GitHub Pages**, remplacez le dossier source par le **dossier branche maître /docs** et sauvegardez.
5. Accédez à la page publique de votre référentiel. 
6. L’entrée du livre d'or "test" créée précédemment devrait apparaître. 
7. Ajoutez de nouvelles entrées.

![](images/solution8/Guestbook.png)

## Facultatif : utilisation de votre domaine pour l'API 

La création d’une API gérée génère un noeud final par défaut tel que `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`. Dans cette section, vous allez configurer ce noeud final pour qu'il puisse gérer les demandes provenant de votre sous-domaine personnalisé. 

### Obtention d'un certificat pour le domaine personnalisé 

L'exposition d'actions {{site.data.keyword.openwhisk_short}} via un domaine personnalisé nécessite une connexion HTTPS sécurisée. Vous devez obtenir un certificat SSL pour le domaine et le sous-domaine que vous prévoyez d'utiliser avec le back-end sans serveur. En supposant un domaine *mydomain.com*, les actions peuvent être hébergées sur *guestbook-api.mydomain.com*. Le certificat doit être émis pour *guestbook-api.mydomain.com* (ou **.mydomain.com*).

Vous pouvez obtenir des certificats SSL gratuits auprès de [Let's Encrypt](https://letsencrypt.org/). Au cours du processus, vous devrez peut-être configurer un enregistrement DNS de type TXT dans l'interface DNS pour prouver que vous êtes le propriétaire du domaine.
{:tip}

Une fois que vous avez obtenu le certificat SSL et la clé privée de votre domaine, veillez à les convertir au format [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Pour convertir un certificat au format PEM : 
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. Pour convertir une clé privée au format PEM : 
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### Importation du certificat dans un référentiel central

1. Créez une instance [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) dans un emplacement pris en charge.
1. Dans le tableau de bord de service, utilisez **Importer le certificat** :
   * Définissez **Nom** sur le sous-domaine et le domaine personnalisés, tel que `guestbook-api.mydomain.com`.
   * Recherchez le **Fichier de certificat** au format PEM.
   * Recherchez le **Fichier de clé privée** au format PEM.
   * **Importez**.

### Configuration du domaine personnalisé pour l'API gérée 

1. Accédez à [API / Domaines personnalisés](https://{DomainName}/apis/domains).
1. Dans le sélecteur Région, sélectionnez l'emplacement où vous avez déployé les actions.
1. Localisez le domaine personnalisé lié à l'organisation et à l'espace où vous avez créé les actions et l'API gérée. Cliquez sur **Modifier les paramètres** dans le menu d'action.
1. Notez la valeur de **Domaine par défaut / alias**.
1. Sélectionnez **Appliquer le domaine personnalisé**
   1. Définissez **Nom du domaine** sur le domaine que vous utiliserez, tel que `guestbook-api.mydomain.com`.
   1. Sélectionnez l'instance {{site.data.keyword.cloudcerts_short}} qui détient le certificat.
   1. Sélectionnez le certificat pour le domaine.
1. Accédez à votre fournisseur DNS et créez un **enregistrement DNS TXT** mappant votre domaine sur le domaine/l'alias par défaut de l'API. L’enregistrement DNS TXT peut être supprimé une fois les paramètres appliqués. 
1. Sauvegardez les paramètres du domaine personnalisé. La boîte de dialogue vérifie l'existence de l'enregistrement DNS TXT. 
1. Enfin, revenez aux paramètres du fournisseur DNS et créez un enregistrement CNAME faisant pointer votre domaine personnalisé (par exemple guestbook-api.mydomain.com) vers le domaine/l'alias par défaut. Le trafic passe alors via votre domaine personnalisé pour être acheminé vers votre API de back-end.

Une fois les modifications DNS propagées, vous pouvez accéder à l'API du livre d'or à l'adresse https://guestbook-api.mydomain.com/guestbook.

1. Editez le fichier **docs/guestbook.js** et mettez à jour la valeur d'**apiUrl** avec https://guestbook-api.mydomain.com/guestbook
1. Validez le fichier modifié.
1. Votre application accède maintenant à l'API via votre domaine personnalisé. 

## Suppression de ressources 

* Supprimez le service {{site.data.keyword.cloudant_short_notm}}
* Supprimez l'API de {{site.data.keyword.openwhisk_short}}
* Supprimez les actions de {{site.data.keyword.openwhisk_short}}

## Contenu associé
* [Autres guides et exemples sur application sans serveur](https://developer.ibm.com/code/journey/category/serverless/)
* [Initiation à {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [Cas d'utilisation courante d'{{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [Création d'API REST à partir d'actions](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
