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

# Génération d'un lac de données à l'aide d'Object Storage
{: #smart-data-lake}

Les définitions de l'expression lac de données varient, mais dans le cadre de ce tutoriel, un lac de données est une approche permettant de stocker des données dans leur format natif à des fins d'organisation. Pour ce faire, vous allez créer un lac de données pour votre organisation à l'aide d'{{site.data.keyword.cos_short}}. En combinant {{site.data.keyword.cos_short}} et SQL Query, les analystes de données peuvent interroger les données où elles se trouvent à l'aide de SQL. Vous pouvez également exploiter le service SQL Query dans un bloc-notes Jupyter pour effectuer une analyse simple. Lorsque vous avez terminé, permettez aux utilisateurs non techniques de découvrir leurs propres informations à l'aide d'{{site.data.keyword.dynamdashbemb_notm}}.

## Objectifs

- Utiliser {{site.data.keyword.cos_short}} pour stocker des fichiers de données brutes 
- Interroger des données directement à partir de {{site.data.keyword.cos_short}} à l'aide de SQL Query 
- Affinage et analyse de données dans {{site.data.keyword.DSX_full}}
- Partage de données au sein de votre organisation avec {{site.data.keyword.dynamdashbemb_notm}}

## Services utilisés

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [ SQL Query ](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Architecture

![Architecture](images/solution29/architecture.png)

1. Les données brutes sont stockées dans {{site.data.keyword.cos_short}}
2. Les données sont réduites, améliorées ou détaillées avec SQL Query
3. L'analyse des données s'exécute dans {{site.data.keyword.DSX}}
4. Le secteur d'activité accède à une application Web 
5. Les données détaillées sont extraites d'{{site.data.keyword.cos_short}}
6. Les graphiques de secteur d'activité sont créés à l'aide d'{{site.data.keyword.dynamdashbemb_notm}}

## Avant de commencer

- [Installez git](https://git-scm.com/)
- [Installez l'interface CLI {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
- [Installez Aspera Connect](http://downloads.asperasoft.com/connect2/)
- [Installez Node.js et NPM](https://nodejs.org)

## Création de services

Dans cette section, vous créez les services nécessaires à la génération de votre lac de données. 

Dans cette section, vous utilisez la ligne de commande pour créer des instances de service. Vous pouvez également procéder de la même manière à partir de la page de service du catalogue à l'aide des liens fournis.
{: tip}

1. Connectez-vous à {{site.data.keyword.cloud_notm}} via la ligne de commande et accédez à votre compte Cloud Foundry. Voir [CLI Getting Started]https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Créez une instance d'[{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) avec un alias Cloud Foundry. Si vous disposez déjà d'une instance de service, exécutez la commande `service-alias-create` avec le nom du service existant.
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. Créez une instance de [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. Créez une instance de [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio).
```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. Créez une instance d'[{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded) avec un alias Cloud Foundry.
```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. Accédez à un répertoire de travail et exécutez la commande suivante pour cloner le [référentiel GitHub](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard) de l'application de tableau de bord. Transférez ensuite l'application vers votre organisation Cloud Foundy. L'application lie automatiquement les services requis ci-dessus à l'aide de son fichier [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml).
    ```sh
    git clone https://github.com/IBM-Cloud/nodejs-data-lake-dashboard.git
    ```
    {: pre}
    ```sh
    cd nodejs-data-lake-dashboard
    ```
    {: pre}
    ```sh
    npm install
    ```
    {: pre}
    ```sh
    npm run push
    ```
    {: pre}

    Après le déploiement, l'application devient publique et écoute un nom d'hôte aléatoire. Vous pouvez accéder à la page [Liste des ressources](https://{DomainName}/resources), sélectionner l'application sous Cloud Foundry Apps et afficher l'URL ou exécuter la commande `ibmcloud cf app dashboard-nodejs routes` pour visualiser les routes.
    {: tip}

7. Confirmez que l'application est active en accédant à son URL publique dans le navigateur. 

![Page d'arrivée du tableau de bord](images/solution29/dashboard-start.png)

## Transfert de données

Dans cette section, vous allez télécharger des données dans un compartiment {{site.data.keyword.cos_short}} à l’aide du service {{site.data.keyword.CHSTSshort}} intégré. {{site.data.keyword.CHSTSshort}} protège les données lors de leur transfert dans le compartiment et [peut réduire considérablement le temps de transfert.](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/).

1. Téléchargez le fichier CSV [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD). La taille du fichier est 81 Mo et le téléchargement peut prendre quelques minutes. 
2. Dans votre navigateur, accédez à l'instance de service **data-lake-cos** à partir de la [Liste de ressources](https://{DomainName}/resources).
3. Créez un compartiment pour stocker les données. 
    - Cliquez sur le bouton **Créer un compartiment**.
    - Sélectionnez **Régional** dans la liste déroulante **Résilience**.
    - Sélectionnez **us-south** dans **Emplacement**. Actuellement, {{site.data.keyword.CHSTSshort}} est uniquement disponible pour les compartiments créés dans l'emplacement `us-south`. Vous pouvez également choisir un autre emplacement et utiliser le type de transfert **Standard** dans la section suivante. 
    - Indiquez un **nom** de compartiment et cliquez sur **Créer**. Si vous recevez une erreur *AccessDenied*, essayez avec un nom de compartiment plus spécifique.
4. Transférez le fichier CSV sur {{site.data.keyword.cos_short}}.
    - Dans votre compartiment, cliquez sur le bouton **Ajouter un objet**.
    - Sélectionnez le bouton d'option **Aspera high-speed transfer**.
    - Cliquez sur le bouton **Ajouter des fichiers**. Le plug-in Aspera apparaît dans une fenêtre distincte, éventuellement derrière la fenêtre de votre navigateur. 
    - Recherchez et sélectionnez le fichier CSV transféré. 

![Compartiment avec fichier CSV](images/solution29/cos-bucket.png)

## Gestion de données

Dans cette section, vous allez convertir le jeu de données brut d'origine en une cohorte ciblée basée sur les attributs de temps et d'âge. Cette opération est utile pour les utilisateurs du lac de données qui ont des intérêts spécifiques ou qui auraient des difficultés avec des jeux de données très volumineux. 

Vous utilisez SQL Query pour manipuler les données où elles se trouvent dans {{site.data.keyword.cos_short}} à l'aide d'instructions SQL courantes. SQL Query comporte un support intégré pour CSV, JSON et Parquet - aucun service de calcul supplémentaire ni fonctionnalité d'extraction, transformation, chargement n'est nécessaire.

1. Accédez à l'instance de service **data-lake-sql** de SQL Query à partir de la [Liste de ressources](https://{DomainName}/resources).
2. Sélectionnez **Open UI**.
3. Créez un nouveau jeu de données en exécutant SQL directement sur le fichier CSV transféré. 
    - Entrez le code SQL suivant dans la zone de texte **Type SQL here ...**.
    ```
        SELECT
        `Dr Number` AS id,
        `Date Occurred` AS date,
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
        FROM cos://us-south/<your-bucket-name>/Traffic_Collision_Data_from_2010_to_Present.csv
        WHERE
        `Time Occurred` >= 1700 AND
        `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND
        `Victim Age` <= 35
        ```
        {: codeblock}
    - Remplacez l'URL dans la clause `FROM` par le nom de votre compartiment. 
4. La **cible** crée automatiquement un compartiment {{site.data.keyword.cos_short}} pour contenir le résultat. Remplacez la **cible** par `cos://us-south/<your-bucket-name>/results`.
5. Cliquez sur le bouton **Run**. Les résultats apparaissent à la suite. 
6. Sous l'onglet **Query Details**, cliquez sur l'icône **Launch** en regard de l'URL **Result Location** pour afficher le jeu de données intermédiaire, qui est également stocké dans {{site.data.keyword.cos_short}}.

![Bloc-notes](images/solution29/sql-query.png)

## Combinaison des bloc-notes Jupyter avec SQL Query 

Dans cette section, vous allez utiliser le client SQL Query dans un bloc-notes Jupyter. Cette opération réutilise les données stockées sur {{site.data.keyword.cos_short}} dans un outil d'analyse de données. La combinaison crée également des jeux de données qui sont automatiquement stockés dans {{site.data.keyword.cos_short}} et qui peuvent ensuite être utilisés avec {{site.data.keyword.dynamdashbemb_notm}}.

1. Créez un bloc-notes Jupyter dans {{site.data.keyword.DSX}}.
    - Dans un navigateur, ouvrez [{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true).
    - Cliquez sur la vignette **Create a Project**, puis sur **Data Science**.
    - Cliquez sur **Create project**, puis indiquez un **nom de projet**.
    - Vérifiez que **Storage** est défini sur **data-lake-cos**.
    - Cliquez sur **Create**.
    - Dans le projet qui apparaît, cliquez sur **Add to project** et **Notebook**.
    - Dans l'onglet **Blank**, entrez un **nom de bloc-notes**.
    - Conservez les valeurs par défaut de **Language** et **Runtime** ; cliquez sur **Create notebook**.
2. A partir du bloc-notes, installez et importez PixieDust et ibmcloudsql en ajoutant les commandes suivantes à l'invite d'entrée **In[ ]:**, puis cliquez sur **Run**.
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. Ajoutez une clé d'API {{site.data.keyword.cos_short}} au bloc-notes. Cela permet de stocker les résultats de SQL Query dans {{site.data.keyword.cos_short}}.
    - Ajoutez les lignes suivantes dans la prochaine invite **In[ ]:**, puis cliquez sur **Run**.
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - A partir du terminal, créez une clé d'API.
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - Copiez la **clé d'API** dans le presse-papiers.
    - Copiez la clé d'API dans la zone de texte du bloc-notes et appuyez sur la touche `Entrée`.
    - Vous devez également stocker la clé d'API dans un emplacement sécurisé et permanent. Le bloc-notes ne stocke pas la clé d'API. 
4. Ajoutez le CRN (Nom de la ressource Cloud) de l'instance SQL Query au bloc-notes. 
    - Dans la prochaine invite **In[ ]:**, attribuez le CRN à une variable de votre bloc-notes.
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - A partir du terminal, copiez le CRN de la propriété **ID** dans votre presse-papiers.
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - Collez le CRN entre guillemets simples, puis cliquez sur **Run**.
5. Ajoutez une autre variable au bloc-notes pour spécifier le compartiment {{site.data.keyword.cos_short}}, puis cliquez sur **Run**.
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. Exécutez les commandes suivantes dans une autre invite **In[ ]:**, puis cliquez sur **Run** pour afficher l'ensemble de résultats. Le nouveau fichier `accidents/jobid=<id>/<part>.csv*` est ajouté à votre compartiment qui inclut le résultat de `SELECT`.
    ```python
    sqlClient = ibmcloudsql.SQLQuery(cloud_api_key, sql_crn, sql_cos_endpoint + '/accidents')

    data_source = sql_cos_endpoint + "/Traffic_Collision_Data_from_2010_to_Present.csv"

    query = """
    SELECT
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
    FROM  {}
    WHERE
        `Time Occurred` >= 1700 AND `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND `Victim Age` <= 35
    """.format(data_source)

    traffic_collisions = sqlClient.run_sql(query)
    traffic_collisions.head()
    ```
    {: codeblock}

## Visualisation de données à l'aide de PixieDust

Dans cette section, vous allez visualiser l'ensemble de résultats précédent à l'aide de PixieDust et de Mapbox afin de mieux identifier les modèles ou les zones sensibles (hot spots) d'incidents de trafic. 

1. Créez une expression de table commune pour convertir la colonne `location` en deux colonnes distinctes, `latitude` et `longitude`. **Exécutez** le code suivant à partir de l'invite du bloc-notes.
    ```python
    query = """
    WITH location AS (
        SELECT
            id,
            cast(split(coordinates, ',')[0] as float) as latitude,
            cast(split(coordinates, ',')[1] as float) as longitude
        FROM (SELECT
                `Dr Number` as id,
                regexp_replace(Location, '[()]', '') as coordinates
            FROM {0}
        )
    )
    SELECT
        d.`Dr Number` as id,
        d.`Date Occurred` as date,
        d.`Time Occurred` AS time,
        d.`Area Name` AS area,
        d.`Victim Age` AS age,
        d.`Victim Sex` AS sex,
        l.latitude,
        l.longitude
    FROM {0} AS d
        JOIN
        location AS l
        ON l.id = d.`Dr Number`
    WHERE
        d.`Time Occurred` >= 1700 AND
        d.`Time Occurred` <= 2000 AND
        d.`Victim Age` >= 20 AND
        d.`Victim Age` <= 35 AND
        l.latitude != 0.0000 AND
        l.latitude != 0.0000
    """.format(data_source)

    traffic_location = sqlClient.run_sql(query)
    traffic_location.head()
    ```
    {: codeblock}
2. Dans la prochaine invite **In[ ]:**, **exécutez** la commande `display` pour afficher le résultat à l'aide de PixieDust.
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. Sélectionnez le bouton de liste déroulante de graphiques, puis sélectionnez **Map**.
4. Ajoutez la `latitude` et la `longitude` aux **Keys**. Ajoutez `id` et `age` aux **Values**. Cliquez sur **OK** pour afficher la carte.
5. Cliquez sur l'icône **Save** pour enregistrer le bloc-notes dans {{site.data.keyword.cos_short}}.

![Bloc-notes](images/solution29/notebook-mapbox.png)

## Partage du jeu de données avec l'organisation 

Tous les utilisateurs du lac de données ne sont pas des analystes scientifiques des données. Vous pouvez permettre aux utilisateurs non techniques d’obtenir une analyse issue du lac de données à l’aide d’{{site.data.keyword.dynamdashbemb_notm}}. De même que SQL Query, {{site.data.keyword.dynamdashbemb_notm}} peut lire des données directement à partir d'{{site.data.keyword.cos_short}} à l'aide de tableaux de bord prédéfinis. Cette section présente une solution qui permet à n’importe quel utilisateur d’accéder au lac de données et de créer un tableau de bord personnalisé. 

1. Accédez à l'URL publique de l'application de tableau de bord que vous avez transférée vers {{site.data.keyword.Bluemix_notm}}.
2. Sélectionnez un modèle qui correspond à l'agencement souhaité. (Les étapes suivantes utilisent le deuxième agencement.) 
3. Utilisez le bouton `Add a source` qui apparaît dans l'étagère latérale `Selected sources`, développez le `nom de compartiment` et cliquez sur l'une des entrées de la table `accidents/jobid=...`. Fermez la boîte de dialogue à l’aide de l’icône X située dans l'angle supérieur droit. 
4. A gauche, cliquez sur l'icône `Visualizations`, puis sur **Summary**.
5. Sélectionnez la source `accidents/jobid=...`, développez `Table` et créez un graphique.
    - Faites glisser et déposez `id` sur la ligne **Value**.
    - Réduisez le graphique à l'aide de l'icône située dans l'angle supérieur. 
6. De nouveau, à partir de `Visualizations`, créez un graphique **Tree map** :
    - Faites glisser et déposez `area` sur la ligne **Area hierarchy**.
    - Faites glisser et déposez `id` sur la ligne **Size**.
    - Réduisez le graphique pour afficher le résultat. 

![Graphique de tableau de bord](images/solution29/dashboard-chart.png)

## Exploration du tableau de bord

Dans cette section, vous allez effectuer quelques étapes supplémentaires pour explorer les fonctionnalités de l'application de tableau de bord et d'{{site.data.keyword.dynamdashbemb_notm}}.

1. Cliquez sur le bouton **Mode** dans la barre d'outils de l'exemple d'application du tableau de bord pour modifier l'affichage du mode `VUE`.
2. Cliquez sur l'une des vignettes colorées du graphique inférieur ou des valeurs `de surface` dans la légende du graphique. Vous appliquez ainsi un filtre local à l'onglet, ce qui oblige les autres graphiques à afficher des données spécifiques au filtre. 
3. Cliquez sur le bouton **Sauvegarder** de la barre d'outils.
    - Entrez le nom de votre tableau de bord dans la zone de saisie correspondante. 
    - Sélectionnez l'onglet **Spec** pour afficher les spécifications du tableau de bord. Une spécification correspond au format de fichier natif d'{{site.data.keyword.dynamdashbemb_notm}}. Vous y trouverez des informations sur les graphiques que vous avez créés ainsi que sur la source de données {{site.data.keyword.cos_short}} utilisée. 
    - Enregistrez votre tableau de bord sur la mémoire de stockage locale du navigateur en utilisant le bouton **Sauvegarder** de la boîte de dialogue.
4. Cliquez sur le bouton **Nouveau** de la barre d'outils pour créer un tableau de bord. Pour ouvrir un tableau de bord sauvegardé, cliquez sur le bouton **Ouvrir**. Pour supprimer un tableau de bord, utilisez l'icône **Supprimer** de la boîte de dialogue Ouvrir le tableau de bord.

Dans les applications de production, chiffrez les informations telles que les URL, les noms d'utilisateur et les mots de passe pour éviter qu'elles ne soient visibles par les utilisateurs finaux. Voir [Encrypting data source information](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information).
{: tip}

## Extension du tutoriel 

Félicitations, vous avez généré un lac de données à l'aide d'{{site.data.keyword.cos_short}}. Vous trouverez ci-dessous des suggestions supplémentaires pour améliorer votre lac de données. 

- Testez d'autres jeux de données à l'aide de SQL Query 
- Diffusez des données provenant de sources multiples dans votre lac de données en effectuant le tutoriel [Analyse des journaux de big data avec Streaming Analytics et SQL Query](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics) 
- Modifiez le code de l'application de tableau de bord pour stocker les spécifications de tableau de bord dans [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) ou {{site.data.keyword.cos_short}}
- Créez une instance de service [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id) pour activer la sécurité dans l'application de tableau de bord

## Suppression de ressources 

Exécutez les commandes suivantes pour supprimer les services, les applications et les clés utilisés. 

```sh
ibmcloud resource service-binding-delete dashboard-nodejs-dde dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-binding-delete dashboard-nodejs-cos dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-dde
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-cos
```
{: pre}
```sh
ibmcloud iam api-key-delete data-lake-cos-key
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-dde
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-cos
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-sql
```
{: pre}
```sh
ibmcloud service delete data-lake-studio
```
{: pre}
```sh
ibmcloud app delete dashboard-nodejs
```
{: pre}

## Contenu associé

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Jupyter Notebooks](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
