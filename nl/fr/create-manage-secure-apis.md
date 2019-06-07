---
copyright:
  years: 2017, 2019
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

# Création, sécurisation et gestion des API REST
{: #create-manage-secure-apis}

Ce tutoriel montre comment créer des API REST à l’aide de l’infrastructure d’API Node.js LoopBack. LoopBack vous permet de créer rapidement des API REST qui connectent des appareils et des
navigateurs à des données et à des services. Vous allez également ajouter la gestion, la visibilité, la sécurité et la limitation de débit à vos API à l'aide d'{{site.data.keyword.apiconnect_long}}.
{:shortdesc}

## Objectifs

* Générer une API REST avec peu ou pas de codage 
* Publier votre API sur {{site.data.keyword.Bluemix_notm}} pour atteindre les développeurs
* Intégrer des API existantes à {{site.data.keyword.apiconnect_short}}
* Exposer et contrôler en toute sécurité l'accès aux systèmes d'enregistrement 

## Services utilisés

Ce tutoriel utilise les environnements d’exécution et les services suivants : 

* [LoopBack](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* [SDK pour l'application Cloud Foundry Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs) 

## Architecture

![Architecture](images/solution13/Architecture.png)

1. Le développeur définit l'API RESTful 
2. Le développeur publie une API dans {{site.data.keyword.apiconnect_long}}
3. Les utilisateurs et les applications utilisent l'API 

## Avant de commencer

* Téléchargez et installez la version 6.x de [Node.js](https://nodejs.org/en/download/) (utilisez [nvm](https://github.com/creationix/nvm) ou similaire si une version plus récente de Node.js est déjà installée) 

## Création d'une API REST dans Node.js 

{: #create_api}
Dans cette section, vous allez créer une API dans Node.js à l'aide de [LoopBack](https://loopback.io/doc/index.html). LoopBack est une infrastructure Node.js hautement extensible et open source qui vous permet de créer des API REST dynamiques de bout en bout avec peu ou pas de codage.

### Création d'une application

1. Installez l'outil de ligne de commande {{site.data.keyword.apiconnect_short}}. Si vous rencontrez des problèmes pour installer {{site.data.keyword.apiconnect_short}}, utilisez `sudo` avant la commande ou suivez les instructions fournies [ici](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit).
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. Entrez la commande suivante pour créer l'application.
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. Appuyez sur `Entrée` pour utiliser **entries-api** comme **nom de votre application**.
4. Appuyez sur `Entrée` pour utiliser **entries-api** en tant que **répertoire devant contenir le projet**.
5. Choisissez la version **3.x (actuelle)** de **LoopBack**.
6. Choisissez **empty-server** comme **type d'application**.

![Construction d'APIC Loopback](images/solution13/apic_loopback.png)

### Ajout d'une source de données

Les sources de données représentent des systèmes back-end tels que des bases de données, des API REST externes, des services Web SOAP et des services de stockage. Les sources de données fournissent généralement des fonctions de création, extraction, mise à jour et suppression (CRUD). Bien que Loopback prenne en charge de nombreux types de [sources de données](http://loopback.io/doc/en/lb3/Connectors-reference.html), pour des raisons de simplicité, vous utilisez un magasin de données en mémoire avec votre API. 

1. Accédez au nouveau projet dans le répertoire et lancez API Designer.
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. Dans le volet de navigation, cliquez sur **Sources de données**. Cliquez ensuite sur le bouton
**Ajouter**. Une boîte de dialogue **Nouvelle source de données de bouclage** apparaît.
3. Entrez `entriesDS` dans la zone de texte **Nom** et cliquez sur le bouton **Nouveau**.
4. Sélectionnez **In-memory db** dans la liste déroulante **Connecteur**.
5. Cliquez sur **Toutes les sources de données** en haut à gauche. La source de données apparaît dans la liste des sources de données. 

   L'éditeur met automatiquement à jour le fichier server/datasources.json avec les paramètres de la nouvelle source de données.
   {:tip}

![Sources de données d'API Designer](images/solution13/datastore.png)

### Ajout d'un modèle

Un modèle est un objet JavaScript avec Node.js et les API REST qui représentent les données dans les systèmes back-end. Les modèles sont connectés à ces systèmes via des sources de données. Dans cette section, vous allez définir la structure de vos données et la connecter à la source de données créée précédemment. 

1. Dans le volet de navigation, cliquez sur **Modèles**. Cliquez ensuite sur le bouton **Ajouter**. Une boîte de dialogue **Nouveau modèle de bouclage** apparaît.
2. Entrez `entry` dans la zone de texte **Nom** et cliquez sur le bouton **Nouveau**. 
3. Sélectionnez **entriesDS** dans la liste déroulante **Source de données**.
4. A partir de la fiche **Propriétés**, ajoutez les propriétés suivantes à l’aide de l’icône **Ajouter une propriété**.
    1. Entrez `name` pour le **Nom de propriété** et sélectionnez **string** dans la liste déroulante **Type**.
    2. Entrez `email` pour le **Nom de propriété** et sélectionnez **string** dans la liste déroulante **Type**.
    3. Entrez `comment` pour le **Nom de propriété** et sélectionnez **string** dans la liste déroulante **Type**.
5. Cliquez sur l'icône **Sauvegarder** en haut à droite pour enregistrer le modèle.

![Générateur de modèle](images/solution13/models.png)

## Test de votre application LoopBack

Dans cette section, vous allez démarrer une instance locale de votre application Loopback et tester l'API en insérant et en interrogeant des données à l'aide d'API Designer. Vous devez utiliser l'outil Explorer d'API Designer pour tester les noeuds finaux REST dans votre navigateur, car il inclut les en-têtes de sécurité appropriés et d'autres paramètres de demande. 

1. Démarrez le serveur local en cliquant sur l'icône **Démarrer** dans l'angle inférieur gauche et attendez le message **En cours d'exécution**.
2. Dans la bannière, cliquez sur le lien **Explorer** pour afficher l'outil Explorer de l'API Designer. La barre latérale présente les opérations REST disponibles pour les modèles LoopBack dans l'API. 
3. Cliquez sur l'opération **entry.create** dans le volet de gauche pour afficher le noeud final. Le volet central affiche des informations récapitulatives sur le noeud final, notamment ses paramètres, la sécurité, les données d'instance de modèle et les codes de réponse. Le volet de droite fournit un exemple de code pour appeler le noeud final à l'aide de la commande cURL, ainsi que des langages tels que Ruby, Python, Java et Node. 
4. Dans le volet de droite, cliquez sur le lien **Essayez**. Faites défiler la liste vers le bas jusqu'à **Paramètres** et entrez les informations suivantes dans la zone de texte **data** :
    ```javascript
    {
      "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. Cliquez sur le bouton **Appeler une opération**.
  ![Test d'une API à partir d'API Designer](images/solution13/data_entry_1.png)
6. Confirmez que le test POST a réussi en recherchant **Response Code: 200 OK**. Si un message d'erreur relatif à un certificat non sécurisé pour un hôte local s'affiche, cliquez sur le lien fourni dans le message d'erreur dans l'outil Explorer du concepteur d'API pour accepter le certificat, puis procédez à l'appel des opérations dans votre navigateur Web. La procédure exacte dépend du navigateur Web que vous utilisez.
7. Ajoutez une autre entrée en utilisant cURL. Confirmez que le port correspond au port de votre application.
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. Cliquez sur l'opération **entry.find**, puis sur le lien **Essayer**, suivi du bouton **Appeler une opération**. La réponse affiche toutes les entrées. JSON devrait apparaître pour **Jane Doe** et **John Doe**.
  ![entry_find](images/solution13/find_response.png)

Vous pouvez également démarrer manuellement l'application Loopback en lançant la commande `npm start` à partir du répertoire `entries-api`. Vos API REST sont disponibles à l'adresse [http://localhost:3000/api/entries](http://localhost:3000/api/entries). Toutefois, elles ne sont pas gérées et protégées par {{site.data.keyword.apiconnect_short}}.
{:tip}

## Création d'un service {{site.data.keyword.apiconnect_short}} 

Pour préparer les étapes suivantes, vous allez créer un service **{{site.data.keyword.apiconnect_short}}** sur {{site.data.keyword.Bluemix_notm}}. {{site.data.keyword.apiconnect_short}} agit en tant que passerelle vers votre API et offre également des fonctionnalités de gestion, de sécurité et de limites de débit.

1. Lancez la [Liste de ressources](https://{DomainName}/resources) {{site.data.keyword.Bluemix_notm}}.
2. Accédez à **Catalogue > Intégration > {{site.data.keyword.apiconnect_short}}**, puis cliquez sur le bouton **Créer**.

## Publication d'une API sur {{site.data.keyword.Bluemix_notm}}

{: #publish}
Vous utilisez API Designer pour déployer votre application sur {{site.data.keyword.Bluemix_notm}} en tant qu’application Cloud Foundry et pour publier votre définition d’API dans **{{site.data.keyword.apiconnect_short}}**. API Designer est votre kit d'outils local. Si vous l'avez fermé, relancez-le avec `apic edit` à partir du répertoire du projet. 

L'application peut également être déployée manuellement à l'aide de la commande `ibmcloud cf push`. Cependant, elle n'est pas sécurisée. Pour [importer l'API](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing) dans {{site.data.keyword.apiconnect_short}}, utilisez le fichier de définition OpenAPI disponible dans le dossier `definitions`. Le déploiement à l'aide d'API Designer permet de sécuriser l'application et d'importer la définition automatiquement.
{:tip}

1. Dans API Designer, cliquez sur le lien **Publier** dans la bannière. Cliquez ensuite sur **Ajouter et gérer des cibles > Ajouter une cible IBM Bluemix**.
2. Sélectionnez la **Région** et l'**Organisation** dans lesquelles vous souhaitez publier.
3. Sélectionnez le catalogue **Bac à sable** et cliquez sur **Suivant**.
4. Entrez `entries-api-application` dans la zone de texte **Entrez un nouveau nom d'application**, puis cliquez sur l'icône **+**.
5. Cliquez sur **entries-api-application** dans la liste, puis cliquez sur **Sauvegarder**.
6. Cliquez sur l'icône du **Menu** hamburgers dans l'angle supérieur gauche de la bannière. Cliquez ensuite sur **Projets** et sur l'élément de liste **entries-api**.
7. Dans l'interface utilisateur d'API Designer, cliquez sur **API > entries-api > Assembler**.
8. Dans l'éditeur d'assemblage, cliquez sur l'icône **Règles de filtrage**.
9. Sélectionnez **Règles DataPower Gateway** et cliquez sur **Sauvegarder**.
10. Cliquez sur **Publier** dans la barre supérieure et sélectionnez votre cible. Sélectionnez **Publier une application** et Mettre en préproduction ou publier des produits > Sélectionner des **produits spécifiques** > **entries-api**. 
11. Cliquez sur **Publier** et attendez que l'application termine sa publication.
![Dialogue de publication](images/solution13/publish.png)

    Une application contient les modèles Loopback, les sources de données et le code liés à votre API. Ce produit vous permet de déclarer comment une API est mise à la disposition des développeurs.
    {:tip}

L'application d'API est désormais publiée sur {{site.data.keyword.Bluemix_notm}} en tant qu'application Cloud Foundry. Elle apparaît dans les applications Cloud Foundry sous [Liste de ressources](https://{DomainName}/resources) {{site.data.keyword.Bluemix_notm}}, mais l'accès direct à l'aide de l'URL n'est pas possible car l'application est protégée. La section suivante montre comment accéder aux API gérées.

## API Gateway

Jusqu'à présent, vous avez conçu et testé votre API localement. Dans cette section, vous allez utiliser {{site.data.keyword.apiconnect_short}} pour tester votre API déployée sur {{site.data.keyword.Bluemix_notm}}.

1. Lancez la [Liste de ressources](https://{DomainName}/resources) {{site.data.keyword.Bluemix_notm}}.
2. Recherchez et sélectionnez votre service **{{site.data.keyword.apiconnect_short}}** sous **Services Cloud Foundry**.
3. Cliquez sur le menu **Explorer**, puis sur le lien **Bac à sable**.
4. Cliquez sur l'opération **entry.create**.
5. Dans le panneau de droite, cliquez sur **Essayer**. Faites défiler la liste vers le bas jusqu'à **Paramètres** et entrez les informations suivantes dans la zone de texte **data**.
    ```javascript
    {
      "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. Cliquez sur le bouton **Appeler une opération**. Une réponse **Code: 200** devrait apparaître pour indiquer que l'opération a abouti.

![passerelle](images/solution13/gateway.png)

L’URL de votre API gérée et sécurisée est affichée en regard de chaque opération et doit ressembler à `https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries`.
{: tip}

## Limitation de débit

Définir des limites de débit vous permet de gérer le trafic réseau de vos API et d'opérations spécifiques au sein de vos API. Une limite de débit est le nombre maximal d'appels autorisés dans un intervalle de temps donné. 

1. Dans API Designer, cliquez sur **Produits > entries-api**.
2. Sélectionnez **Plan par défaut** à gauche.
3. Développez **Plan par défaut** et faites défiler la liste vers le bas jusqu'à la zone **Limites de débit**.
4. Définissez les zones sur **10** appels / **1** **Minute**.
5. Sélectionnez **Appliquer la limite absolue** et cliquez sur l'icône **Sauvegarder**.
  ![Page de limites de débit](images/solution13/rate_limit.png)
6. Effectuez les étapes décrites dans la section [Publication de l'API dans {{site.data.keyword.Bluemix_notm}}](#publish) pour republier votre API.

Votre API est maintenant limitée à 10 demandes par minute. Utilisez la fonctionnalité **Essayer** pour atteindre la limite. Consultez [ici](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits) davantage d'informations sur la définition des limites de débit ou explorez API Designer pour connaître toutes les fonctionnalités de gestion disponibles. 

## Extension du tutoriel 

Félicitations, vous avez généré une API à la fois gérée et sécurisée. Vous trouverez ci-dessous des suggestions supplémentaires pour améliorer votre API.

* Ajoutez la persistance des données à l'aide du connecteur LoopBack [{{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant) 
* Utilisez API Designer pour [afficher des paramètres supplémentaires](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0) permettant de gérer votre API
* Consultez les **analyses** et les **visualisations** de l'API [disponibles](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics) dans {{site.data.keyword.apiconnect_short}}

## Contenu associé

* [Documentation sur Loopback](https://loopback.io/doc/index.html)
* [Initiation à {{site.data.keyword.apiconnect_long}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
