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

# Analyse et visualisation de données ouvertes avec Apache Spark
{: #big-data-analytics-spark}

Dans ce tutoriel, vous allez analyser et visualiser des ensembles de données ouverts à l'aide d'{{site.data.keyword.DSX_full}}, d'un bloc-notes Jupyter et d'Apache Spark. Vous commencez par combiner des données décrivant la croissance démographique, l’espérance de vie et les codes ISO de pays en une seule trame de données. Pour découvrir des informations, vous utilisez ensuite une bibliothèque Python appelée Pixiedust pour interroger et visualiser les données de différentes manières.

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## Objectifs
{: #objectives}

* Déployer Apache Spark et {{site.data.keyword.DSX_short}} sur IBM Cloud
* Utiliser un bloc-notes Jupyter et un noyau Python
* Importer, transformer, analyser et visualiser des jeux de données 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Configuration du service et de l'environnement 
Commencez par mettre à disposition les services utilisés dans ce tutoriel et créez un projet dans {{site.data.keyword.DSX_short}}.

Vous pouvez mettre à disposition des services pour {{site.data.keyword.Bluemix_short}} dans la [Liste de ressources](https://{DomainName}/resources) et le [catalogue](https://{DomainName}/catalog/). Sinon, {{site.data.keyword.DSX_short}} vous permet de créer ou d’ajouter des services Data & Analytics existants à partir de son tableau de bord et de ses paramètres de projet.
{:tip}

1. Dans le [catalogue {{site.data.keyword.Bluemix_short}}](https://{DomainName}/catalog), accédez à la section **AI**. Créez le service **{{site.data.keyword.DSX_short}}**. Cliquez sur le bouton **Get Started** pour lancer le tableau de bord **{{site.data.keyword.DSX_short}}**.
2. Dans le tableau de bord, cliquez sur la tuile **Create a project** > sélectionnez **Standard** > Create project. Dans la zone **Name**, entrez `1stProject` en tant que nom. Vous pouvez laisser la description vide.
3. Dans la partie droite de la page, vous pouvez **définir le stockage**. Si vous avez déjà mis à disposition le stockage, sélectionnez une instance dans la liste. Si tel n'est pas le cas, cliquez sur **Add** et suivez les instructions du nouvel onglet du navigateur. Une fois la création du service terminée, cliquez sur **Refresh** pour visualiser le nouveau service.
4. Cliquez sur le bouton **Create** pour créer le projet. Vous êtes alors redirigé vers la page de présentation du projet.  
   ![](images/solution23/NewProject.png)
5. Dans la page de présentation, cliquez sur **Settings**.
6. Dans la section **Associated services**, cliquez sur **Add Service** et sélectionnez **Spark** dans le menu. Dans l'écran qui apparaît, vous pouvez choisir une instance de service Spark existante ou en créer une autre. 

## Création et préparation d'un bloc-notes
[Jupyter Notebook](http://jupyter.org/) est une application Web à source ouverte qui vous permet de créer et de partager des documents contenant du code opérationnel, des équations, des visualisations et du texte narratif. Les blocs-notes et autres ressources sont organisés en projets.
1. Cliquez sur l'onglet **Assets**, faites défiler l'écran vers le bas jusqu'à la section **Notebooks** et cliquez sur **New notebook**.
2. Utilisez un bloc-notes **vide**. Entrez `MyNotebook` pour **Name**.
3. Dans le menu **Select runtime**, choisissez l'instance Spark que vous avez ajoutée aux paramètres du projet. Conservez la valeur par défaut **Language** en tant que **Python 3.5**.
4. Cliquez sur **Create Notebook** pour terminer le processus.
5. La zone où vous entrez du texte et des commandes s'appelle une **Cellule**. Copiez le code suivant dans la cellule vide pour importer le package [**Pixiedust**](https://pixiedust.github.io/pixiedust/use.html). Exécutez la cellule en cliquant sur l'icône **Run** de la barre d'outils ou en appuyant sur les touches **Maj+Entrée** du clavier.
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

Si vous n'avez jamais utilisé Jupyter Notebook, cliquez sur l'icône **Docs** dans le menu en haut à droite. Accédez à **Analyze data**, puis à la [section **Notebooks**](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics) pour en savoir plus sur les [blocs-notes et leurs composants](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true).
{:tip}

## Chargement des données 
Chargez ensuite trois jeux de données ouvertes et mettez-les à disposition dans le bloc-notes. La bibliothèque **Pixiedust** facilite le [chargement des fichiers **CSV** à l'aide d'une adresse URL](https://pixiedust.github.io/pixiedust/loaddata.html).

1.  Copiez la ligne suivante dans la prochaine cellule vide de votre bloc-notes, mais ne l'exécutez pas encore. 
   ```Python
   df_pop = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
2. Dans un autre onglet du navigateur, allez à la section [Community](https://dataplatform.ibm.com/community?context=analytics). Sous **Data Sets**, recherchez [**Total population by country**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e) et cliquez sur la tuile. En haut à droite, cliquez sur l'icône de **lien** pour obtenir un URI d'accès. Copiez l'URI et remplacez le texte **YourAccessURI** dans la cellule du bloc-notes par le lien. Cliquez sur l'icône **Run** de la barre d'outils ou appuyez sur les touches ** Maj+Entrée**.
3. Répétez cette étape pour un autre jeu de données. Copiez la ligne suivante dans la prochaine cellule vide de votre bloc-notes. 
   ```Python
   df_life = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
4. Dans l’autre onglet du navigateur contenant les **ensembles de données**, recherchez [**Life expectancy at birth by country in total years**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895). Obtenez à nouveau le lien et utilisez-le pour remplacer **YourAccessURI** dans la cellule du bloc-notes, puis sélectionnez **Run** pour démarrer le processus de chargement.
5. Pour le dernier des trois jeux de données, chargez une liste de noms de pays et leurs codes ISO à partir d'une collection de jeux de données ouvertes sur Github. Copiez le code dans la prochaine cellule vide du bloc-notes et exécutez-le. 
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

La liste des codes de pays est utilisée ultérieurement pour simplifier la sélection des données en utilisant un code de pays au lieu du nom de pays exact. 

## Transformation des données
Une fois les données disponibles, transformez-les légèrement et combinez les trois jeux en une seule trame de données. 
1. Le bloc de code suivant redéfinit la trame de données pour les données de population. Cette opération est réalisée via une instruction SQL qui renomme les colonnes. Une vue est ensuite créée et le schéma imprimé. Copiez ce code dans la prochaine cellule vide et exécutez-le. 
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. Répétez ces étapes pour les données d'espérance de vie. Au lieu d'imprimer le schéma, ce code imprime les 10 premières lignes.   
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. Répétez la transformation du schéma pour les données de pays. 
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. Les noms de colonne sont désormais plus simples et identiques pour tous les jeux de données, et peuvent être combinés en une seule trame de données. Effectuez une jointure **externe** sur les données d'espérance de vie et de population. Ensuite, dans la même instruction, effectuez une jointure **interne** pour ajouter les codes de pays. Toutes les données sont triées par pays et par année. La sortie définit la trame de données **df_all**. Avec une jointure interne, les données obtenues ne contiennent que les pays qui figurent dans la liste ISO. Ce processus supprime les entrées régionales et autres dans les données. 
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. Remplacez le type de données pour **Year** par un entier.
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

Vos données combinées sont prêtes à être analysées. 

## Analyse des données 
Dans cette partie, utilisez [Pixiedust pour visualiser les données dans différents graphiques](https://pixiedust.github.io/pixiedust/displayapi.html). Commencez par comparer l'espérance de vie de certains pays.

1. Copiez le code dans la prochaine cellule vide et exécutez-le. 
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. Un tableau déroulant est affiché. Cliquez sur l'icône de graphique directement sous le bloc de code et sélectionnez **Line Chart**. Une boîte de dialogue contextuelle contenant **Pixiedust: Line Chart Options** apparaît. Entrez un **titre de graphique** comme "Comparison of Life Expectancy". Dans les **zones** proposées, faites glisser **Year** dans la zone **Keys**, **Life** dans la zone **Values**. Entrez **1000** pour **# of Rows to Display**. Appuyez sur **OK** pour tracer le graphique à courbes. Dans la partie droite, assurez-vous que **mapplotlib** est sélectionné en tant que **Renderer**. Cliquez sur le sélecteur **Cluster By** et choisissez le **pays**. Un graphique similaire à celui-ci est affiché.
   ![](images/solution23/LifeExpectancy.png)

3. Créez un graphique centré sur l'année 2010. Copiez le code dans la prochaine cellule vide et exécutez-le. 
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. Dans le sélecteur de graphique, sélectionnez **Map**. Dans la boîte de dialogue de configuration, faites glisser **Country** dans la zone **Keys**. Déplacez **Life** dans la zone **Values**. Comme pour le premier graphique, augmentez la valeur de **# of Rows to Display** à **1000**. Appuyez sur **OK** pour tracer la carte. Indiquez **brunel** pour **Renderer**. Une carte du monde en couleur illustrant l'espérance de vie est affichée. Vous pouvez utiliser la souris pour zoomer sur la carte.
   ![](images/solution23/LifeExpectancyMap2010.png)

## Extension du tutoriel 
Voici quelques idées et suggestions pour améliorer ce tutoriel.
* Créez et visualisez une requête montrant le taux d'espérance de vie par rapport à la croissance démographique du pays de votre choix 
* Calculez et visualisez les taux de croissance de la population par pays sur une carte du monde 
* Chargez et intégrez des données supplémentaires issues du catalogue de jeux de données 
* Exportez les données combinées dans un fichier ou une base de données 

## Contenu associé
{:related}
Vous trouverez ci-dessous des liens relatifs aux sujets abordés dans ce tutoriel. 
* [Watson Data Platform](https://dataplatform.ibm.com) : utilisez Watson Data Platform pour collaborer et générer des applications plus intelligentes. Visualisez et découvrez rapidement les analyses de vos données et collaborez entre les équipes. 
* [PixieDust](https://www.ibm.com/cloud/pixiedust) : outil de productivité open source pour blocs-notes Jupyter
* [Cognitive Class.ai](https://cognitiveclass.ai/) : cours de science des données et d'informatique cognitive
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/) : tout ce que nous avons créé à partir de données, et que vous pouvez aussi créer
* [Analytics Engine service](https://{DomainName}/catalog/services/analytics-engine) : développez et déployez des applications d'analyse à l'aide des logiciels open source Apache Spark et Apache Hadoop
