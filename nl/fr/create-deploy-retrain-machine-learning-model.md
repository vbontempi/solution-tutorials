---
copyright:
  years: 2018, 2019
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

# Génération, déploiement, test et réentraînement d'un modèle d'apprentissage automatique prédictif
{: #create-deploy-retrain-machine-learning-model}
Ce tutoriel vous guide dans le processus de création d'un modèle d'apprentissage automatique prédictif, de son déploiement en tant qu'API à utiliser dans les applications, du test du modèle et de son réentraînement avec les données de commentaires en retour. Tous ces processus ont lieu au cours d'une expérience de libre-service intégrée et unifiée sur IBM Cloud. 

Dans ce tutoriel, le **jeu de données sur les espèces d'Iris** est utilisé pour créer un modèle d'apprentissage automatique permettant de classer les espèces de fleurs.

Dans la terminologie de l’apprentissage automatique, la classification est considérée comme un exemple d’apprentissage supervisé, c’est-à-dire un apprentissage où un ensemble de formation d’observations correctement identifiées est disponible. {:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## Objectifs
{: #objectives}

* Importer des données dans un projet. 
* Générer un modèle d'apprentissage machine. 
* Déployer le modèle et tester l'API. 
* Tester un modèle d'apprentissage automatique.
* Créer une connexion de données de commentaires en retour pour un apprentissage continu et une évaluation du modèle. 
* Réentraîner le modèle. 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## Avant de commencer
{: #prereqs}
* IBM Watson Studio et Watson Knowledge Catalog sont des applications qui font partie d'IBM Watson. Pour créer un compte IBM Watson, commencez par vous inscrire à l'une de ces applications ou aux deux. 

   Cliquez sur [Try IBM Watson](https://dataplatform.ibm.com/registration/stepone) et inscrivez-vous pour les applications IBM Watson.

## Importation de données dans un projet

{:#import_data_project}

Un projet est la façon dont vous organisez vos ressources pour atteindre un objectif particulier. Les ressources de votre projet peuvent inclure des données, des collaborateurs et des outils d’analyse tels que des blocs-notes Jupyter et des modèles d’apprentissage automatique. 

Vous pouvez créer un projet pour ajouter des données et ouvrir un actif de données dans le raffineur de données pour le nettoyage et la mise en forme de vos données.

**Créez un projet :**

1. Accédez au [catalogue {{site.data.keyword.Bluemix_short}}](https://{DomainName}/catalog) et sélectionnez [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services) dans la section **IA**. **Créez** le service. Cliquez sur le bouton **Get Started** pour lancer le tableau de bord **{{site.data.keyword.DSX_short}}**.
2. Créez un **projet** > cliquez sur la vignette **Create Project** sur **Standard**. Ajoutez un nom, par exemple, `iris_project` et une description facultative du projet.
3. Ne cochez pas la case **Restrict who can be a collaborator**, car il n'y a pas de données confidentielles.
4. Sous **Define Storage**, cliquez sur **Add** et choisissez un service Cloud Object Storage existant ou créez-en un (sélectionnez un forfait **Lite** > Create). Sélectionnez **Refresh** pour voir le service créé.
5. Cliquez sur **Créer**. Votre nouveau projet s'ouvre et vous pouvez commencer à y ajouter des ressources. 

**Importez des données :**

Comme mentionné précédemment, vous utilisez le **jeu de données Iris**. Le jeu de données Iris a été utilisé dans le document de R.A. Fisher datant de 1936, intitulé [The Use of Multiple Measurements in Taxonomic Problems](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf), et peut également être consulté dans [UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/). Ce petit jeu de données est souvent utilisé pour tester des algorithmes d’apprentissage automatique et des visualisations. L'objectif est de classer les fleurs d'iris en trois espèces (Setosa, Versicolor ou Virginica) à partir de mesures de longueur et de largeur de sépales et de pétales. Le jeu de données sur l'iris contient 3 classes de 50 instances chacune, chaque classe faisant référence à un type d'iris.
![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


**Téléchargez** [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv) qui comporte 40 instances de chaque classe. Vous utilisez les 10 autres instances de chaque classe pour réentraîner votre modèle. 

1. Sous **Assets** dans votre projet, cliquez sur l'icône **Find and Add Data**. ![Affiche l'icône de recherche de données.](images/solution22-build-machine-learning-model/data_icon.png).
2. Sous **Load**, cliquez sur **browse** et transférez le fichier `iris_initial.csv` téléchargé.
      ![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. Une fois le fichier ajouté, `iris_initial.csv` devrait apparaître dans la section **Data assets** du projet. Cliquez sur le nom pour afficher le contenu du jeu de données.

## Services associés 
{:#associate_services}
1. Sous **Settings**, faites défiler l'écran jusqu'à **Associated services** > cliquez sur **Add service** > choisissez **Spark**.
   ![](images/solution22-build-machine-learning-model/associate_services.png)
2. Sélectionnez le forfait **Lite** et cliquez sur **Create**. Utilisez les valeurs par défaut et cliquez sur **Confirm**.
3. Cliquez à nouveau sur **Add Service** et choisissez **Watson**. Cliquez sur **Add** sur la vignette **Machine Learning** > choisissez le forfait **Lite** > cliquez sur **Create**.
4. Conservez les valeurs par défaut et cliquez sur **Confirm** pour mettre à disposition un service d'apprentissage automatique.

## Génération d'un modèle d'apprentissage automatique

{:#build_model}

1. Cliquez sur **Add to project** et sélectionnez le **modèle d'apprentissage automatique Watson**. Dans la boîte de dialogue, ajoutez **iris_model** comme nom et une description facultative..
2. Sous la section **Machine Learning Service**, vous devriez voir le service d'apprentissage automatique que vous avez associé à l'étape ci-dessus.
   ![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. Sélectionnez **Model builder** comme type de modèle et, dans la section **Spark Service or Environment**, choisissez le service Spark que vous avez créé précédemment.
4. Sélectionnez **Manual** pour créer manuellement un modèle. Cliquez sur **Créer**.

   Pour la méthode automatique, vous dépendez entièrement du traitement automatique des données (ADP). Pour la méthode manuelle, en plus de certaines fonctions gérées par le transformateur ADP, vous pouvez ajouter et configurer vos propres estimateurs, qui sont les algorithmes utilisés dans l'analyse.
   {:tip}

5. Dans la page suivante, sélectionnez `iris_initial.csv` comme jeu de données et cliquez sur **Next**.
6. Dans la page **Select a technique**, en fonction de du jeu de données ajouté, les colonnes Label et les colonnes Feature sont préremplies. Sélectionnez **species (String)** comme **Label Col** et **petal_length (Decimal)** et **petal_width (Decimal)** comme **Feature**.
7. Choisissez **Multiclass Classification** comme technique suggérée.
   ![](images/solution22-build-machine-learning-model/model_technique.png)
8. Pour **Validation Split**, configurez le paramètre suivant :

   **Train:** 50%,
   **Test** 25%,
   **Holdout:** 25%

9. Cliquez sur **Add Estimators** et sélectionnez **Decision Tree Classifier**, puis **Add**.

   Vous pouvez évaluer plusieurs estimateurs en une fois. Par exemple, vous pouvez ajouter **Decision Tree Classifier** et **Random Forest Classifier** en tant qu'estimateurs pour former votre modèle et choisir le meilleur ajustement en fonction du résultat de l'évaluation.
   {:tip}

10. Cliquez sur **Next** pour former le modèle. Lorsque le statut **Trained & Evaluated** apparaît, cliquez sur **Save**.
   ![](images/solution22-build-machine-learning-model/trained_model.png)

11. Cliquez sur **Overview** pour vérifier les détails du modèle.

## Déploiement du modèle et test de l'API

{:#deploy_model}

1. Sous le modèle créé, cliquez sur **Deployments** > **Add Deployment**.
2. Choisissez **Web Service**. Ajoutez un nom, par exemple, `iris_deployment`, et une description facultative. 
3. Cliquez sur **Sauvegarder**. Sur la page de présentation, cliquez sur le nom du nouveau service Web. Une fois que le statut est **DEPLOY_SUCCESS**, vous pouvez vérifier le noeud final d'évaluation, les fragments de code dans différents langages de programmation et la spécification de l'API sous **Implementation**.
4. Cliquez sur **View API Specification** pour visualiser et tester les noeuds finaux de l'API {{site.data.keyword.pm_short}}.
   ![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   Pour commencer à utiliser l'API, vous devez générer un **jeton d'accès** à l'aide du **nom d'utilisateur** et du **mot de passe** disponibles dans l'onglet **Données d'identification pour le service** de l'instance de service {{site.data.keyword.pm_short}} dans la [Liste de ressources {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/). Suivez les instructions mentionnées sur la page de spécification de l'API pour générer un **jeton d'accès**.
{:tip}
5. Pour effectuer une prévision en ligne, utilisez l'appel d'API `POST /online`.
   * `instance_id` se trouve dans l'onglet **Données d'identification pour le service** du service {{site.data.keyword.pm_short}} dans la [Liste de ressources {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/).
   * `deployment_id` et `published_model_id` se trouvent dans la **présentation** de votre déploiement.
   *  Pour `online_prediction_input`, utilisez le code JSON ci-dessous 

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * Cliquez sur **Essayez** pour afficher la sortie JSON.

6. Grâce aux noeuds finaux de l'API, vous pouvez maintenant appeler ce modèle à partir de n'importe quelle application. 

## Test de votre modèle

{:#test_model}

1. Sous **Test**, les données d'entrée (données d'entité) doivent être automatiquement renseignées. 
2. Cliquez sur **Predict** pour afficher la valeur **Predicted value for species** dans un graphique.
   ![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. Pour l’entrée et la sortie JSON, cliquez sur les icônes en regard de l’entrée et de la sortie actives. 
4. Vous pouvez modifier les données d'entrée et continuer à tester votre modèle. 

## Création d'une connexion de données de commentaires en retour 

{:#create_feedback_connection}

1. Pour un apprentissage continu et une évaluation de modèle, vous devez stocker les nouvelles données. Créez un [service {{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse) > forfait **Entry** qui sert de connexion de données de commentaires en retour.
2. Dans la page **Manage** de {{site.data.keyword.dashdbshort}}, cliquez sur **Open**. Dans la barre de navigation supérieure, sélectionnez **Load**.
3. Cliquez sur **browse files** sous **My computer** et transférez le fichier `iris_initial.csv`. Cliquez sur **Next**.
4. Sélectionnez **DASHXXXX**, par exemple, DASH1234 pour **Schema**, puis cliquez sur **New Table**. Nommez la table `IRIS_FEEDBACK` et cliquez sur **Next**.
5. Les types de données sont automatiquement détectés. Cliquez sur **Next**, puis sur **Begin Load**.
   ![](images/solution22-build-machine-learning-model/define_table.png)
6. La cible **DASHXXXX.IRIS_FEEDBACK** est créée.

   Vous l'utilisez dans l'étape suivante, où vous réentraînez le modèle pour améliorer ses performances et la précision. 

## Réentraînement du modèle

{:#retrain_model}

1. Revenez à la [Liste de ressources {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources) et, sous le service {{site.data.keyword.DSX_short}} que vous utilisez, cliquez sur **Projects** > iris_project >  **iris-model** (sous Assets) > Evaluation.
2. Sous **Performance Monitoring**, cliquez sur **Configure Performance Monitoring**.
3. Dans la page Configure Performance Monitoring,
   * Sélectionnez le service Spark. Le type de prévision doit être renseigné automatiquement. 
   * Choisissez **weightedPrecision** comme métrique et définissez `0.98` comme seuil facultatif.
   * Cliquez sur **Create new connection** pour pointer sur IBM Db2 Warehouse dans le cloud que vous avez créé dans la section ci-dessus.
   * Sélectionnez la connexion Db2 Warehouse et une fois les détails de la connexion renseignés, cliquez sur **Create**.
     ![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * Cliquez sur **Select feedback data reference**, pointez sur la table IRIS_FEEDBACK et cliquez sur **Select**.
     ![](images/solution22-build-machine-learning-model/select_source.png)
   * Dans la zone **Record count required for re-evaluation**, entrez le nombre minimal de nouveaux enregistrements pour déclencher le réentraînement. Utilisez **10** ou laissez la zone vide pour utiliser la valeur par défaut 1000.
   * Dans la zone **Auto retrain**, sélectionnez l'une des options suivantes :
     - Pour démarrer le réentraînement automatique lorsque les performances du modèle sont inférieures au seuil que vous avez défini, sélectionnez **when model performance is below threshold**. Pour ce tutoriel, sélectionnez cette option car la précision indiquée est inférieure au seuil (.98).
     - Pour interdire le réentraînement automatique, sélectionnez **never**.
     - Pour démarrer le réentraînement automatique quelle que soit la performance, sélectionnez **always**.
   * Dans la zone **Auto deploy**, sélectionnez l'une des options suivantes :
     - Pour démarrer le déploiement automatique lorsque les performances du modèle sont meilleures que celles de la version précédente, sélectionnez **when model performance is better than previous version**. Pour ce tutoriel, sélectionnez cette option car votre objectif est d’améliorer continuellement les performances du modèle.
     - Pour interdire le déploiement automatique, sélectionnez **never**.
     - Pour démarrer le déploiement automatique quelle que soit la performance, sélectionnez **always**.
   * Cliquez sur **Save**.
     ![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. Téléchargez le fichier [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv). Ensuite, cliquez sur **Add feedback data**, sélectionnez le fichier csv téléchargé, puis cliquez sur **Open**.
5. Cliquez sur **New evaluation** pour commencer.
     ![](images/solution22-build-machine-learning-model/retraining_model.png)
6. Une fois l'évaluation terminée. Vous pouvez consulter la section **Last Evalution Result** pour connaître la valeur **WeightedPrecision** améliorée.

## Suppression de ressources 
{:removeresources}

1. Accédez à la [Liste de ressources {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/) > choisissez l'emplacement, l'organisation et l'espace dans lesquels vous avez créé les services. 
2. Supprimez les services {{site.data.keyword.DSX_short}}, {{site.data.keyword.sparks}}, {{site.data.keyword.pm_short}}, {{site.data.keyword.dashdbshort}} et {{site.data.keyword.cos_short}} respectifs que vous avez créés pour ce tutoriel.

## Contenu associé
{:related}

- [Présentation de Watson Studio](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [Détection d'anomalies à l'aide de l'apprentissage automatique](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [Création automatique de modèles](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [Apprentissage automatique & IA](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
