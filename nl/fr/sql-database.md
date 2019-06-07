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

# Base de données SQL pour Cloud Data 
{: #sql-database}

Ce tutoriel explique comment mettre à disposition un service de base de données SQL (relationnelle), créer une table et charger un jeu de données volumineux (informations urbaines) dans la base de données. Vous déployez ensuite une application Web "worldcities" pour utiliser ces données et montrer comment accéder à la base de données dans le cloud. L'application est écrite en langage Python à l'aide de l'[infrastructure Flask](http://flask.pocoo.org/).

![](images/solution5/Architecture.png)

## Objectifs

* Mettre à disposition une base de données SQL 
* Créer le schéma de base de données (table) 
* Charger des données 
* Connecter l'application et le service de base de données (données d'identification de partage) 
* Surveillance, sécurité, sauvegardes et reprise

## Produits

Ce tutoriel utilise les produits suivants : 
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## Avant de commencer
{: #prereqs}

Accédez à [GeoNames](http://www.geonames.org/) et téléchargez et extrayez le fichier [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip). Il contient des informations sur les villes de plus de 1000 habitants. Vous allez l'utiliser comme jeu de données.

## Mise à disposition de la base de données SQL
Commencez par créer une instance du service **[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)**.

![](images/solution5/Catalog.png)

1. Consultez le [tableau de bord {{site.data.keyword.Bluemix_short}}](https://{DomainName}). Cliquez sur **Catalogue** dans la barre de navigation supérieure.
2. Cliquez sur **Données & analyse** sous Plateforme dans le volet gauche et sélectionnez **{{site.data.keyword.dashdbshort_notm}}**.
3. Sélectionnez le forfait **Entrée** et remplacez le nom du service suggéré par "sqldatabase" (vous utiliserez ce nom ultérieurement). Sélectionnez un emplacement pour le déploiement de la base de données et assurez-vous que l'organisation et l'espace appropriés sont sélectionnés. 
4. Cliquez sur **Créer**. Après un court instant, vous devriez recevoir une notification de réussite. 
5. Dans la **Liste de ressources**, cliquez sur l'entrée correspondant au nouveau service {{site.data.keyword.dashdbshort_notm}}.
6. Cliquez sur **Ouvrir** pour lancer la console de base de données. Si vous utilisez la console pour la première fois, une visite guidée vous est proposée. 

## Création d'une table
Une table est nécessaire pour stocker les données exemple. Créez-la à l'aide de la console.

1. Dans la console pour {{site.data.keyword.dashdbshort_notm}}, cliquez sur **Explorer** dans la barre de navigation. Une liste de schémas existants dans la base de données apparaît. 
2. Recherchez et sélectionnez le schéma commençant par "DASH".
3. Cliquez sur **"+ Nouvelle table"** pour afficher un formulaire pour le nom de la table et ses colonnes.
4. Indiquez "cities" comme nom de table. Copiez les définitions de colonne du fichier [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) dans la zone réservée aux colonnes et aux types de données. 
5. Cliquez sur **Créer** pour définir la nouvelle table.   
   ![](images/solution5/TableCitiesCreated.png)

## Chargement des données 
Une fois la table "cities" créée, vous pouvez y charger des données. Cette opération peut être effectuée de différentes manières, par exemple à partir de votre ordinateur local ou du stockage COS (Cloud Object Storage) avec l'interface Swift ou Amazon S3, en utilisant le service de migration [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift). Pour ce tutoriel, vous allez télécharger les données à partir de votre ordinateur. Au cours de ce processus, vous adaptez la structure de la table et le format des données pour qu'ils correspondent parfaitement au contenu du fichier. 

1. Dans la barre de navigation supérieure, cliquez sur **Load**. Ensuite, sous **File selection**, cliquez sur **browse files** pour localiser et sélectionner le fichier "cities1000.txt" que vous avez téléchargé dans la première section de ce guide.
2. Cliquez sur **Next** pour accéder à la vue d'ensemble du schéma. Choisissez à nouveau le schéma commençant par "DASH", puis la table "CITIES". Cliquez à nouveau sur **Next**.   

   Dans la mesure où la table est vide, il n’est pas important d’ajouter des données aux données existantes ou de les remplacer.
{:tip }
3. A présent, personnalisez la manière dont les données du fichier "cities1000.txt" sont interprétées pendant le processus de chargement. Tout d'abord, désactivez l'"en-tête dans la première ligne" car le fichier ne contient que des données. Ensuite, entrez "0x09" comme séparateur. Cela signifie que les valeurs du fichier sont délimitées par des tabulations. Enfin, choisissez "AAAA-MM-JJ" comme format de date. Votre écran devrait donc ressembler à celui-ci :     
  ![](images/solution5/LoadTabSeparator.png)
4. Cliquez sur **Next** pour examiner les paramètres de chargement. Acceptez et cliquez sur **Begin Load** pour commencer à charger les données dans la table "CITIES". La progression est affichée. Une fois les données téléchargées, quelques secondes suffisent pour effectuer le chargement et présenter les premières statistiques.    
   ![](images/solution5/LoadProgressSteps.png)

## Vérification des données chargées à l'aide de SQL 
Les données ont été chargées dans la base de données relationnelle. Il n'y a pas eu d'erreur, mais vous devez tout de même exécuter quelques tests rapides. Utilisez l'éditeur SQL intégré pour saisir et exécuter des instructions SQL. 

1. Dans la barre de navigation supérieure, cliquez sur **Exécuter SQL**. Au lieu de l'éditeur SQL intégré, vous pouvez utiliser des outils SQL traditionnels et basés sur le cloud sur votre ordinateur de bureau ou serveur avec {{site.data.keyword.dashdbshort_notm}}. Les informations de connexion se trouvent dans le menu des paramètres. Certains outils sont même proposés au téléchargement dans la section "Téléchargements" du menu situé derrière l’icône de "livre" (représentant la documentation et l'aide).
{:tip }
2. Dans l'éditeur SQL, entrez ou copiez la requête suivante :    
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   puis appuyez sur le bouton **Tout exécuter**. Dans la section des résultats, le même nombre de lignes que celui indiqué par le processus de chargement doit être affiché.    
3. Dans l'éditeur SQL, entrez l'instruction suivante sur une nouvelle ligne : 
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. Dans l'éditeur, sélectionnez le texte de l'instruction ci-dessus. Cliquez sur le bouton **Exécutez la sélection**. Seule cette instruction doit être exécutée maintenant et renvoyer des statistiques par pays dans la section des résultats. 

## Déploiement du code applicatif
[Le code prêt à l'emploi pour l'application de base de données se trouve dans ce référentiel Github](https://github.com/IBM-Cloud/cloud-sql-database). Clonez ou téléchargez le référentiel, puis transférez-le vers IBM Cloud.

1. Clonez le référentiel Github : 
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. Transférez l'application vers IBM Cloud. Vous devez être connecté à l'emplacement, à l'organisation et à l'espace pour lesquels la base de données a été mise à disposition. Copiez et collez ces commandes une ligne à la fois.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. Une fois le processus de transfert (push) terminé, vous devriez pouvoir accéder à l'application. Aucune configuration supplémentaire n'est nécessaire. Le fichier `manifest.yml` indique à IBM Cloud de lier l’application et le service de base de données nommé "sqldatabase". 

## Sécurité, sauvegarde & reprise, surveillance
{{site.data.keyword.dashdbshort_notm}} est un service géré. IBM veille à la sécurité de l'environnement, aux sauvegardes quotidiennes et à la surveillance du système. Dans le forfait d’entrée, l’environnement de la base de données est une configuration à service partagé avec une administration réduite et des options configurées pour les utilisateurs. Toutefois, si vous utilisez l'un des forfaits d'entreprise, [plusieurs options vous permettent de gérer les utilisateurs, de configurer une sécurité supplémentaire pour la base de données](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html) et de [surveiller la base de données](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html).    

En plus des options d'administration traditionnelles, le [service {{site.data.keyword.dashdbshort_notm}} propose également une API REST pour la surveillance, la gestion des utilisateurs, les utilitaires, la charge, l'accès au stockage, etc.](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html) L’interface Swagger exécutable de cette API est accessible dans le menu situé derrière l’icône "livre" sous "API REST". Certains outils pouvant être utilisés pour la surveillance, entre autres, comme IBM Data Server Manager, peuvent même être téléchargés dans la section "Téléchargements" de ce même menu. 

## Test de l'application
L'application permettant d'afficher des informations sur la ville en fonction du jeu de données chargé est réduite au minimum. Elle propose un formulaire de recherche permettant de spécifier un nom de ville et quelques villes préconfigurées. Les noms sont convertis en `/search?name=cityname` (formulaire de recherche) ou `/city/cityname` (villes spécifiées directement). Les deux demandes sont traitées à partir des mêmes lignes de code en arrière-plan. Le nom de ville est transmis en tant que valeur à une instruction SQL préparée à l'aide d'un marqueur de paramètre pour des raisons de sécurité. Les lignes sont extraites de la base de données et transmises à un modèle HTML pour être affichées. 

## Nettoyage
Pour nettoyer les ressources utilisées dans ce tutoriel, procédez comme suit : 
1. Consultez la [Liste de ressources {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources). Localisez votre application.
2. Cliquez sur l'icône de menu de l'application et choisissez **Supprimer l'application**. Dans la boîte de dialogue, cochez la case pour supprimer le service {{site.data.keyword.dashdbshort_notm}} associé.
3. Cliquez sur le bouton **Supprimer**. L'application et le service de base de données sont supprimés et vous revenez à la liste des ressources. 

## Extension du tutoriel 
Vous souhaitez développer cette application ? Voici quelques suggestions :
1. Ajoutez une recherche générique sur les noms alternatifs. 
2. Recherchez des villes d'un pays spécifique et au sein de valeurs de population données uniquement. 
3. Modifiez la mise en page en remplaçant les styles CSS et en développant les modèles. 
4. Autorisez la création d'informations sur la ville à partir de formulaires ou autorisez la mise à jour de données existantes, par ex. la population.

## Contenu associé
* Documentation : [IBM Knowledge Center for {{site.data.keyword.dashdbshort_notm}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Foire aux questions sur {{site.data.keyword.Db2_on_Cloud_long_notm}} et {{site.data.keyword.dashdblong_notm}}](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html) en réponse à des questions relatives aux services gérés, à la sauvegarde de données, au chiffrement et à la sécurité des données, etc.
* [Free Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) pour les développeurs
* Documentation : [Description de l'API du pilote ibm_db Python](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
