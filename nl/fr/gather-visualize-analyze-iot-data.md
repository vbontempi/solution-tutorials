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

# Collecte, visualisation et analyse de données IoT
{: #gather-visualize-analyze-iot-data}
Ce tutoriel vous guide dans la configuration d'un terminal IoT, la collecte de données dans {{site.data.keyword.iot_short_notm}}, l'exploration de données et la création de visualisations, puis l'utilisation de services avancés d'apprentissage automatique pour analyser des données et détecter des anomalies dans les données d'historique.
{:shortdesc}

## Objectifs
{: #objectives}

* Configurer IoT Simulator. 
* Envoyer les données de collecte à {{site.data.keyword.iot_short_notm}}.
* Créer des visualisations. 
* Analyser les données générées par le terminal et détecter les anomalies. 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Application Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience) avec service Spark et {{site.data.keyword.cos_full_notm}}
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* Les terminaux envoient des données de capteur à {{site.data.keyword.iot_full}} à l'aide du protocole MQTT
* Les données d'historique sont exportées dans une base de données {{site.data.keyword.cloudant_short_notm}} 
* {{site.data.keyword.DSX_short}} extrait les données de cette base de données
* Les données sont analysées et visualisées via un bloc-notes Jupyter 

## Avant de commencer
{: #prereqs}

[{{site.data.keyword.Bluemix_notm}} Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Exécutez le script pour installer ibmcloud cli et les plug-ins requis

## Création du service Internet of Things Platform 
{: #iot_starter}

Pour commencer, créez le service Internet of Things Platform, le concentrateur capable de gérer les terminaux, de connecter et de **collecter des données** en toute sécurité, et de mettre à disposition les données d'historique pour les visualisations et les applications. 

1. Accédez au [**catalogue {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) et sélectionnez [**Internet of Things Platform**](https://{DomainName}/catalog/services/internet-of-things-platform) dans la section **Internet of Things**.
2. Entrez `IoT demo hub` en tant que nom de service, cliquez sur **Créer** et **lancez** le tableau de bord.
3. Dans le menu latéral, sélectionnez **Sécurité > Sécurité de la connexion**, choisissez **TLS optionnel** sous **Règle par défaut** > **Niveau de sécurité**, puis cliquez sur **Sauvegarder**.
4. Dans le menu latéral, sélectionnez **Terminaux** > **Types de terminal** et **+ Ajouter un type de terminal**.
5. Entrez `simulator` comme **nom** et cliquez sur **Suivant** et **Terminé**.
6. Ensuite, cliquez sur **Enregistrer des terminaux**
7. Choisissez `simulator` pour **Sélectionner un type de terminal existant**, puis entrez `phone` pour **ID de terminal**.
8. Cliquez sur **Suivant** jusqu'à ce que l'écran **Sécurité du terminal** (sous l'onglet Sécurité) s'affiche.
9. Entrez une valeur pour **Jeton d'authentification**, par exemple : `myauthtoken` et cliquez sur **Suivant**.
10. Lorsque vous cliquez sur **Terminé**, les informations de connexion s’affichent. Conservez cet onglet ouvert.

La plateforme IoT est maintenant configurée pour commencer à recevoir des données. Les terminaux doivent envoyer leurs données au service IoT Platform avec le type, l'ID et le jeton de terminal spécifiés. 

## Création d'un simulateur de terminal 
{: #confignodered}
Vous allez ensuite déployer une application Web Node.js et la consulter sur votre téléphone. Vous vous connectez ainsi au service IoT Platform et y envoyez les données d'accéléromètre de terminal et d'orientation. 

1. Clonez le référentiel Github : 
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. Ouvrez le code dans l'environnement IDE de votre choix et remplacez les valeurs `name` et `host` du fichier **manifest.yml** par une valeur unique.
3. Transférez l'application vers {{site.data.keyword.Bluemix_notm}}.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. Après quelques minutes, votre application est déployée et vous pouvez voir une URL similaire à `<UNIQUE_NAME>.mybluemix.net`
5. Explorez cette URL sur votre téléphone à l'aide d'un navigateur. 
6. Entrez les données de connexion de l'onglet Tableau de bord IoT sous **Données d'identification du terminal** et cliquez sur **Connecter**.
7. Votre téléphone va commencer à transmettre les données. De retour dans l'onglet **IBM {{site.data.keyword.iot_short_notm}}**, recherchez les nouvelles entrées dans la section **Evénements récents**.
  ![](images/solution16/recent_events_with_phone.png)

## Affichage des données en direct dans IBM {{site.data.keyword.iot_short_notm}}
{: #createcards}
Vous allez ensuite créer un tableau et des cartes pour afficher les données de terminal dans le tableau de bord. Pour plus d'informations sur les tableaux et les cartes,
consultez [Visualisation des données en temps réel à l'aide de tableaux et de cartes](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html).

### Création d'un tableau
{: #createboard}

1. Ouvrez le **tableau de bord IBM {{site.data.keyword.iot_short_notm}}**.
2. Sélectionnez **Tableaux** dans le menu de gauche, puis cliquez sur **Créer un nouveau tableau**.
3. Entrez un nom pour le tableau, `Simulators` comme exemple, puis cliquez sur **Suivant** et sur **Soumettre**.  
4. Sélectionnez le tableau que vous venez de créer pour l'ouvrir. 

### Affichage des données de terminal 
{: #cardtemp}
1. Cliquez sur **Ajouter une nouvelle carte**, puis sélectionnez le type de carte **Graphique à courbes**, situé dans la section Terminaux.
2. Sélectionnez votre terminal dans la liste, puis cliquez sur **Suivant**.
3. Cliquez sur **Connecter un nouveau jeu de données**.
4. Dans la page Créer une carte Valeur, sélectionnez ou entrez les valeurs suivantes,
puis cliquez sur **Suivant**.
   - Event: sensorData
   - Property: ob
   - Name: OrientationBeta
   - Type: Float
   - Min : -180
   - Max : 180
5. Dans la page Aperçu de la carte, sélectionnez **L** pour la taille du graphique à courbes, puis cliquez sur **Suivant** > **Soumettre**
6. La carte apparaît sur le tableau de bord et comprend un graphique à courbes des données de température opérationnelles. 
7. Utilisez le navigateur de votre téléphone portable pour lancer à nouveau le simulateur et inclinez lentement le téléphone vers l’avant et l’arrière. 
8. Dans **l'onglet IBM {{site.data.keyword.iot_short_notm}}**, vous pouvez observer que le graphique est mis à jour.
   ![](images/solution16/board.png)

## Stockage des données d'historique dans {{site.data.keyword.cloudant_short_notm}}
1. Accédez au [**catalogue {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) et créez une base de données [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) nommée `iot-db`.
2. Sous **Connexions** :
   1. **Créez une connexion**. 
   1. Sélectionnez l'emplacement, l'organisation et l'espace Cloud Foundry où un alias du service {{site.data.keyword.cloudant_short_notm}} doit être créé.
   1. Développez le nom de l’espace dans la table **Emplacement de connexion** et utilisez le bouton **Connecter** en regard de **iot demo hub** pour créer un alias pour le service {{site.data.keyword.cloudant_short_notm}} dans cet espace. 
   1. Connectez-vous et reconstituez l'application. 
3. Ouvrez le **tableau de bord IBM {{site.data.keyword.iot_short_notm}}**.
4. Sélectionnez **Extensions** dans le menu de gauche, puis cliquez sur **Configurer** sous **Stockage des données d'historique**.
5. Sélectionnez la base de données `iot-db` {{site.data.keyword.cloudant_short_notm}}.
6. Entrez `devicedata` pour **Nom de la base de données** et cliquez sur **Terminé**.
7. Une nouvelle fenêtre devrait apparaître pour demander une autorisation. Si tel n'est pas le cas, désactivez le bloqueur de fenêtres publicitaires et actualisez la page. 

Les données de votre terminal sont maintenant enregistrées dans {{site.data.keyword.cloudant_short_notm}}. Après quelques minutes, lancez le tableau de bord {{site.data.keyword.cloudant_short_notm}} pour afficher vos données.

![](images/solution16/cloudant.png)

## Détection des anomalies à l'aide de l'apprentissage automatique
{: #data_experience}

Dans cette section, vous utilisez le Bloc-notes Jupyter disponible dans le service IBM {{site.data.keyword.DSX_short}} pour charger vos données d'historique mobiles et détecter les anomalies à l'aide de z-score. *z-score* est un score standard qui indique de combien d'écarts types un élément se trouve de la moyenne. 

### Création d'un projet
1. Accédez au [**catalogue {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) et sous **IA**, sélectionnez [**{{site.data.keyword.DSX_short}}**](https://{DomainName}/catalog/services/data-science-experience).
2. **Créez** le service et lancez son tableau de bord en cliquant sur **Get Started**
3. Créez un Projet > Sélectionnez **Data Science** > Cliquez sur **Create project** et entrez `Detect Anomaly` comme **nom** du projet.
4. Ne cochez pas la case **Restrict who can be a collaborator**, car il n'y a pas de données confidentielles.
5. Sous **Define Storage**, cliquez sur **Add** et choisissez un service **Cloud Object Storage** existant ou créez-en un (sélectionnez un forfait **Lite** > Create). Sélectionnez **Refresh** pour voir le service créé.
6. Cliquez sur **Create**. Votre nouveau projet s'ouvre et vous pouvez commencer à y ajouter des ressources. 

### Connexion à {{site.data.keyword.cloudant_short_notm}} pour les données 

1. Cliquez sur **Assets** > **+ Add to Project** > **Connection**  
2. Sélectionnez la base **iot-db** {{site.data.keyword.cloudant_short_notm}} où sont stockées les données du terminal.
3. Vérifiez les données **Credentials**, puis cliquez sur **Create**.

### Création d'un service Apache Spark

1. Cliquez sur **Services** dans la barre de navigation supérieure > Compute Services.
2. Cliquez sur **Ajouter un service**.
   1. Cliquez sur **Add** sur **Apache Spark**.
   1. Sélectionnez le forfait **Lite**.
   1. Cliquez sur **Create**.
3. Sélectionnez une organisation et un espace, modifiez le nom du service si vous le souhaitez et **Confirmez**.
1. Accédez au projet `Detect Anomaly` via **Projects**.
1. Dans la zone **Settings**, faites défiler jusqu'à **Associated services**.
1. Cliquez sur **Add service**, sélectionnez **Spark**.
1. Sélectionnez l'instance **Apache Spark** précédemment créée.

### Création d'un bloc-notes Jupyter (ipynb) 
1. Cliquez sur **+ Add to Project** et ajoutez un nouveau **bloc-notes**.
2. Entrez `Anomaly-detection-notebook` pour le **nom**.
3. Entrez `https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb` dans **Notebook URL**.
4. Sélectionnez le service **Apache Spark** associé précédemment en tant qu'environnement d'exécution.
5. Créez **Notebook**. Définissez `Python 3.5 with Spark 2.1` en tant que noyau. Vérifiez que le bloc-notes est créé avec des métadonnées et du code.
   ![Jupyter Notebook Watson Studio](images/solution16/jupyter_notebook_watson_studio.png)
   Pour mettre à jour, sélectionnez **Kernel** > Change kernel. Pour **accréditer** le bloc-notes, sélectionnez **File** > Trust Notebook.
   {:tip}

### Exécution du bloc-notes et détection des anomalies    
1. Sélectionnez la cellule qui commence par `!pip install --upgrade pixiedust,`, puis cliquez sur **Exécuter** ou **Ctrl + Entrée** pour exécuter le code.
2. Une fois l'installation terminée, redémarrez le noyau Spark en cliquant sur l'icône **Restart Kernel**.
3. Dans la cellule de code suivante, importez vos données d'identification {{site.data.keyword.cloudant_short_notm}} dans cette cellule en procédant comme suit :
   * Cliquez sur ![](images/solution16/data_icon.png).
   * Sélectionnez l'onglet **Connexions**.
   * Cliquez sur **Insert to code**. Un dictionnaire appelé _credentials_1_ est créé avec vos données d'identification {{site.data.keyword.cloudant_short_notm}}. Si le nom n'est pas spécifié en tant que _credentials_1_, renommez le dictionnaire en `credentials_1`. `credentials_1` est utilisé dans les cellules restantes.
4. Dans la cellule portant le nom de la base de données (`dbName`), entrez le nom de la base de données {{site.data.keyword.cloudant_short_notm}}, à savoir la source des données, par exemple, *iotp_yourWatsonIoTProgId_DBName_Year-month-day*. Pour visualiser les données de différents terminaux, modifiez les valeurs de `deviceId` et `deviceType` en conséquence.
   Vous pouvez localiser la base de données exacte en accédant à l'instance **iot-db** {{site.data.keyword.cloudant_short_notm}} que vous avez créée > Lancer le tableau de bord.
   {:tip}
5. Enregistrez le bloc-notes et exécutez les cellules de code les unes après les autres ou exécutez toutes les cellules (**Cell** > Run All). Les anomalies des données de mouvement de terminal (oa, ob et og) devraient s'afficher à la fin du bloc-notes.
   Vous pouvez remplacer l'intervalle de temps d'intérêt par l'heure souhaitée de la journée. Recherchez les valeurs `start` et `end`.
   {:tip}
   ![Jupyter Notebook DSX](images/solution16/anomaly_detection_watson_studio.png)
6. Outre la détection des anomalies, les principales conclusions de cette section sont les suivantes : 
    * Utilisation de Spark pour préparer les données à la visualisation. 
    * Utilisation de Pandas pour la visualisation de données. 
    * Diagrammes à barres, histogrammes pour les données de terminal. 
    * Corrélation entre deux capteurs à travers la matrice de corrélation. 
    * Une boîte à moustaches pour chaque capteur de terminal, produite avec la fonction de traçage Pandas. 
    * Tracés de densité par estimation de densité de noyau (Kernel Density Estimation - KDE).
    ![](images/solution16/density_plots_sensor_data.png)

## Suppression de ressources 
{:removeresources}

1. Accédez à la [Liste de ressources](https://{DomainName}/resources/) > choisissez l'emplacement, l'organisation et l'espace dans lesquels vous avez créé l'application et les services. Sous **Applications Cloud Foundry**, supprimez l'application Node.JS créée ci-dessus.
2. Sous **Services**, supprimez les services respectifs Internet of Things Platform, Apache Spark, {{site.data.keyword.cloudant_short_notm}} et {{site.data.keyword.cos_full_notm}} que vous avez créés pour ce tutoriel.

## Contenu associé
{:related}

* [Génération, déploiement, test et réentraînement d'un modèle d'apprentissage automatique prédictif](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* Présentation d'[IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html)
* Détection d'anomalies de [Jupyter Notebook](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb)
* [Compréhension du score z](https://en.wikipedia.org/wiki/Standard_score)
* Développement de solutions IoT cognitives pour la détection d'anomalies à l'aide de l'apprentissage en profondeur - [Série de 5 articles](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/)
