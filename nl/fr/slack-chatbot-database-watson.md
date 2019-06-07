---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Génération d'un Slackbot basé sur une base de données
{: #slack-chatbot-database-watson}

Dans ce tutoriel, vous allez générer un Slackbot pour créer et rechercher des entrées dans la base de données Db2 pour des événements et des conférences. Slackbot est pris en charge par le service {{site.data.keyword.conversationfull}}. Vous allez intégrer Slack et {{site.data.keyword.conversationfull}} à l'aide d'une intégration de l'Assistant.

L'intégration de Slack canalise les messages entre Slack et {{site.data.keyword.conversationshort}}. Certaines actions de dialogue côté serveur exécutent des requêtes SQL sur une base de données Db2. Tout le code (mais peu) est écrit en Node.js. 

## Objectifs
{: #objectives}

* Connecter {{site.data.keyword.conversationfull}} à Slack à l'aide d'une intégration
* Créer, déployer et lier des actions Node.js dans {{site.data.keyword.openwhisk_short}}
* Accéder à une base de données Db2 à partir de {{site.data.keyword.openwhisk_short}} à l'aide de Node.js 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}} ](https://{DomainName}/catalog/services/db2-warehouse) ou [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

<p style="text-align: center;">

  ![Architecture](images/solution19/SlackbotArchitecture.png)
</p>

## Avant de commencer
{: #prereqs}

Pour effectuer ce tutoriel, vous devez disposer de la dernière version de [l'interface CLI {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) et le [plug-in {{site.data.keyword.openwhisk_short}} doit être installé](/docs/cli?topic=cloud-cli-plug-ins).


## Configuration du service et de l'environnement 
Dans cette section, vous configurez les services nécessaires et préparez l'environnement. La plupart de ces opérations peuvent être effectuées à partir de l'interface de ligne de commande (CLI) à l'aide de scripts disponibles sur GitHub.

1. Clonez le [Référentiel GitHub](https://github.com/IBM-Cloud/slack-chatbot-database-watson) et accédez au répertoire cloné :
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. Si vous n'êtes pas connecté, utilisez `ibmcloud login` pour vous connecter de manière interactive.
3. Ciblez l'organisation et l'espace dans lequel créer le service de base de données : 
   ```
   ibmcloud target --cf
   ```
4. Créez une instance de {{site.data.keyword.dashdbshort}} et nommez-la **eventDB** :
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
   Vous pouvez également utiliser un autre forfait que le forfait **Entrée**.
5. Pour accéder ultérieurement au service de base de données à partir de {{site.data.keyword.openwhisk_short}}, vous devez disposer d'une autorisation. Vous créez alors des données d'identification du service et les étiquetez **slackbotkey** :   
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. Créez une instance du service {{site.data.keyword.conversationshort}}. Utilisez **eventConversation** comme nom et le forfait Lite gratuit.
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. Vous enregistrez ensuite des actions pour {{site.data.keyword.openwhisk_short}} et liez des données d'identification du service à ces actions. Certaines des actions sont activées en tant qu'actions Web et un secret est défini pour empêcher les appels non autorisés. Choisissez un secret et transmettez-le comme paramètre - remplacez **YOURSECRET** en conséquence.

   L'une des actions est appelée pour créer une table dans {{site.data.keyword.dashdbshort}}. En utilisant une action de {{site.data.keyword.openwhisk_short}}, vous n'avez besoin ni d'un pilote Db2 local ni de l'interface utilisateur basée sur un navigateur pour créer la table manuellement. Pour effectuer l'enregistrement et la configuration, exécutez la ligne ci-dessous et le fichier **setup.sh** contenant toutes les actions est exécuté. Si votre système ne prend pas en charge les commandes shell, copiez chaque ligne du fichier **setup.sh** et exécutez-la séparément. 

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **Remarque :** par défaut, le script insère également quelques lignes de données exemple. Vous pouvez désactiver cette insertion en mettant en commentaire la ligne suivante dans le script ci-dessus : `#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r`
8. Extrayez les informations de l'espace de nom pour les actions déployées. 

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   Notez la partie située avant **/slackdemo/eventInsert**. Il s'agit de l'organisation et de l'espace codés. Vous en aurez besoin à l'étape suivante.

## Chargement de la compétence / l'espace de travail 
Dans cette partie du tutoriel, vous chargez un espace de travail ou une compétence prédéfini dans le service {{site.data.keyword.conversationshort}}.
1. Dans la [Liste de ressources {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), ouvrez la présentation de vos services. Recherchez l'instance du service {{site.data.keyword.conversationshort}} créée dans la section précédente. Cliquez sur son entrée, puis sur l'alias du service pour ouvrir les détails du service. 
2. Cliquez sur **Lancer l'outil** pour accéder à l'outil {{site.data.keyword.conversationshort}}.
3. Basculez vers **Skills**, puis cliquez sur **Create skill** et sur **Import skill**.
4. Dans la boîte de dialogue, après avoir cliqué sur **Choose JSON file**, sélectionnez le fichier **assistant-skill.json** dans le répertoire local. Laissez l'option d'importation sur **Everything (Intents, Entities, and Dialog)**, puis cliquez sur **Import**. Vous créez ainsi une compétence appelée **TutorialSlackbot**.
5. Cliquez sur **Dialog** pour visualiser les noeuds de dialogue. Vous pouvez les développer pour afficher une structure comme celle illustrée ci-dessous. 

   Le dialogue contient des noeuds pour traiter les questions d’aide et un simple "Merci". Le noeud **newEvent** et son enfant rassemblent les entrées nécessaires, puis appellent une action pour insérer un nouvel enregistrement d'événement dans Db2.

   Le noeud **query events** précise si les événements sont recherchés par identificateur ou par date. La recherche et la collecte proprement dites des données nécessaires sont ensuite effectuées par les noeuds enfants, **query events by shortname** et **query event by dates**. 

   **credential_node** définit le secret des actions de dialogue et les informations relatives à l'organisation Cloud Foundry. Ce noeud est nécessaire pour appeler les actions. 

  Les détails sont expliqués ultérieurement ci-dessous une fois la configuration terminée.
  ![](images/solution19/SlackBot_Dialog.png)   
6. Cliquez sur le noeud de dialogue **credential_node**, ouvrez l'éditeur JSON en cliquant sur l'icône de menu à droite de **Then respond with**.

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   Remplacez **org_space** par les informations d'organisation et d'espace codées que vous avez extraites précédemment. Remplacez tous les signes "@" par "%40". Ensuite, remplacez **YOURSECRET** par le secret réel. Fermez l'éditeur JSON en cliquant à nouveau sur l'icône.

## Création d'un assistant et intégration de Slack 

Vous créez à présent un assistant associé à la compétence précédente et l'intégrer à Slack.  
1. Cliquez sur **Skills** en haut à gauche, puis sélectionnez **Assistants**. Ensuite, cliquez sur **Create assistant**.
2. Dans la boîte de dialogue, indiquez le nom dans **TutorialAssistant**, puis cliquez sur **Create assistant**. Sur l'écran suivant, choisissez **Add dialog skill**. Ensuite, choisissez **Add existing skill**, sélectionnez **TutorialSlackbot** dans la liste et ajoutez-le.
3. Après avoir ajouté la compétence, cliquez sur **Add integration**, puis dans la liste **Managed integrations**, sélectionnez **Slack**.
4. Suivez les instructions pour intégrer votre robot conversationnel à Slack. 

## Test du Slackbot et apprentissage
Ouvrez votre espace de travail Slack pour effectuer un essai du robot conversationnel. Commencez une discussion directe avec le robot. 

1. Entrez **help** dans le formulaire de messagerie. Le robot devrait renvoyer quelques conseils. 
2. Entrez à présent **new event** pour commencer à collecter des données pour un nouvel enregistrement d’événement. Vous utilisez les attributs de {{site.data.keyword.conversationshort}} pour collecter toutes les entrées nécessaires.
3. Indiquez d'abord l'identificateur ou le nom de l'événement. Les guillemets sont obligatoires. Ils permettent d'entrer des noms plus complexes. Entrez **"Meetup: IBM Cloud"** comme nom de l'événement. Le nom de l'événement est défini comme une entité basée sur un modèle **eventName**. Il prend en charge différents types de guillemets doubles au début et à la fin. 
4. Indiquez ensuite l'emplacement de l'événement. L'entrée est basée sur l'[entité système **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details). De manière limitative, seules les villes reconnues par {{site.data.keyword.conversationshort}} peuvent être utilisées. Essayez la ville **Friedrichshafen**.
5. Les informations de contact telles qu'une adresse électronique ou l'URI d'un site Web sont demandées à l'étape suivante. Commencez par **https://www.ibm.com/events**. Vous utilisez une entité basée sur un modèle pour cette zone. 
6. Les prochaines informations demandées sont la date et l'heure de début et de fin de la collecte. **sys-date** et **sys-time** sont utilisés pour permettre différents formats d'entrée. Utilisez **next Thursday** comme date de début, **6 pm** pour l'heure, indiquez la date exacte du jeudi suivant, par exemple **2019-05-09** et **22:00** pour la date et l'heure de fin.
7. Enfin, avec toutes les données collectées, un récapitulatif est imprimé et une action du serveur, implémentée en tant qu'action {{site.data.keyword.openwhisk_short}}, est appelée pour insérer un nouvel enregistrement dans Db2. Le dialogue bascule ensuite sur un noeud enfant pour nettoyer l'environnement de traitement en supprimant les variables de contexte. L'ensemble du processus de saisie peut être annulé à tout moment en saisissant **cancel**, **exit** ou une commande similaire. Dans ce cas, le choix de l'utilisateur est reconnu et l'environnement nettoyé.
  ![](images/solution19/SlackSampleChat.png)
   

Les exemples de données vous permettent d'effectuer des recherches.
1. Entrez **show event information**. Vous devez ensuite indiquer si la recherche doit être effectuée par identificateur ou par date. Entrez un **nom** et répondez à la question suivante **"Think 2019"**. Le robot conversationnel devrait désormais afficher des informations sur cet événement. Le dialogue propose plusieurs réponses. 
2. Avec {{site.data.keyword.conversationshort}} en tant que back-end, il est possible de saisir des phrases plus complexes et de sauter ainsi des parties du dialogue. Utilisez **show event by the name "Think 2019"** comme entrée. Le robot conversationnel renvoie directement l'enregistrement d'événement.
3. Vous allez à présent effectuer une recherche par date. Une recherche est définie par une paire de dates, la date de début de l'événement doit se trouver dans cet intervalle. Avec l'entrée **search conference by date in February 2019**, le résultat devrait être à nouveau l'événement **Think 2019**. L'entité **February** est interprétée comme deux dates, le 1er et le 28 février, fournissant ainsi une entrée de début et de fin de la plage de dates. [Si aucune année 2019 n'est spécifiée, le mois de février à venir est identifié.](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time). 

Après quelques recherches supplémentaires et de nouvelles entrées d’événement, vous pouvez examiner l’historique des discussions et améliorer le dialogue. Suivez les instructions de la documentation [{{site.data.keyword.conversationshort}} pour **améliorer la compréhension**](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro). 


## Suppression de ressources 
{:removeresources}

L'exécution du script de nettoyage dans le répertoire principal supprime la table d'événements de {{site.data.keyword.dashdbshort}} et supprime les actions de {{site.data.keyword.openwhisk_short}}. Cette action peut être utile lorsque vous commencez à modifier et à développer le code. Le script de nettoyage ne modifie pas l'espace de travail ou la compétence {{site.data.keyword.conversationshort}}.   
```bash
sh cleanup.sh
```
{:codeblock}

Dans la [Liste de ressources {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), ouvrez la présentation de vos services. Recherchez l'instance du service {{site.data.keyword.conversationshort}} et supprimez-la. 

## Extension du tutoriel 
Vous souhaitez développer ou modifier ce tutoriel ? Voici quelques suggestions :
1. Ajoutez des fonctionnalités de recherche, par exemple, une recherche générique ou de durées d’événement ("donnez-moi tous les événements de plus de 8 heures").
2. Utilisez {{site.data.keyword.databases-for-postgresql}} au lieu de {{site.data.keyword.dashdbshort}}. Le [référentiel GitHub de ce tutoriel Slackbot](https://github.com/IBM-Cloud/slack-chatbot-database-watson) contient déjà du code permettant de prendre en charge {{site.data.keyword.databases-for-postgresql}}.
3. Ajoutez un service météo et extrayez les données de prévision pour la date et le lieu de l'événement. 
4. Exportez les données d'événement sous forme de fichier iCalendar **.ics**.
5. Connectez le robot conversationnel à Facebook Messenger en ajoutant une autre intégration. 
6. Ajoutez des éléments interactifs à la sortie, tels que des boutons.       


## Contenu associé
{:related}

Voici des liens vers des informations supplémentaires sur les sujets abordés dans ce tutoriel. 

Articles de blogue relatifs aux robots conversationnels : 
* [Robots conversationnels : conseils sur les attributs d'IBM Watson Conversation](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [Robots conversationnels animés : bonnes pratiques](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [Génération de robots conversationnels : encore plus de trucs et astuces](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

Documentation et SDK :
* Référentiel GitHub avec [trucs et astuces pour gérer des variables dans IBM Watson Conversation](https://github.com/IBM-Cloud/watson-conversation-variables)
* [Documentation {{site.data.keyword.openwhisk_short}} ](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Documentation : [IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Free Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) pour les développeurs
* Documentation : [Description de l'API du pilote ibm_db Node.js](https://github.com/ibmdb/node-ibm_db)
* [Documentation {{site.data.keyword.cloudantfull}}](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
