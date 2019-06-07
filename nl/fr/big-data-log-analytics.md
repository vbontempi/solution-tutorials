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

# Analyse des journaux de big data avec Streaming Analytics et SQL Query 
{: #big-data-log-analytics}

Dans ce tutoriel, vous allez générer un pipeline d'analyse de journaux conçu pour collecter, stocker et analyser des enregistrements de journaux afin de répondre aux exigences réglementaires ou de faciliter la découverte d'informations. Cette solution exploite plusieurs services disponibles dans {{site.data.keyword.cloud_notm}} : {{site.data.keyword.messagehub}}, {{site.data.keyword.cos_short}}, SQL Query et {{site.data.keyword.streaminganalyticsshort}}. Un programme vous aide en simulant la transmission des messages de journal du serveur Web à partir d'un fichier statique vers {{site.data.keyword.messagehub}}.

Avec {{site.data.keyword.messagehub}}, le pipeline évolue pour recevoir des millions d'enregistrements de journaux provenant de différents producteurs. A l'aide de {{site.data.keyword.streaminganalyticsshort}}, vous pouvez inspecter les données de journal en temps réel pour intégrer les processus métier. Les messages de journal peuvent également être facilement redirigés vers un stockage à long terme à l'aide d'{{site.data.keyword.cos_short}}, où les développeurs, le personnel de support et les auditeurs peuvent utiliser les données directement à l'aide de SQL Query.

Bien que ce tutoriel se concentre sur l'analyse des journaux, il est applicable à d'autres scénarios : les unités IoT limitées en stockage peuvent également transmettre des messages à {{site.data.keyword.cos_short}} ou les professionnels du marketing peuvent segmenter et analyser des événements client dans des propriétés numériques à l'aide de SQL Query.
{:shortdesc}

## Objectifs

{: #objectives}

* Comprendre la messagerie de publication/abonnement Apache Kafka 
* Stocker les données de journal pour les exigences d'audit et de conformité 
* Surveiller les journaux pour créer des processus de gestion des exceptions 
* Effectuer des analyses contextuelles et statistiques sur les données du journal 

## Services utilisés

{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture

{: #architecture}

<p style="text-align: center;">

  ![Architecture](images/solution31/Architecture.png)
</p>

1. L'application génère des événements de journal pour {{site.data.keyword.messagehub}}
2. L'événement de journal est intercepté et analysé par {{site.data.keyword.streaminganalyticsshort}}
3. L'événement de journal est ajouté à la fin d'un fichier CSV situé dans {{site.data.keyword.cos_short}}
4. L'auditeur ou le personnel de support émet le travail SQL 
5. SQL Query s'exécute sur le fichier journal dans {{site.data.keyword.cos_short}}
6. L'ensemble de résultats est stocké dans {{site.data.keyword.cos_short}} et remis à l'auditeur et au personnel de support

## Avant de commencer

{: #prereqs}

* [Installez Git](https://git-scm.com/)
* [Installez l'interface CLI {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* [Installez Node.js](https://nodejs.org)
* [Téléchargez le client Kafka 0.10.2.X](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)

## Création de services

{: #setup}

Dans cette section, vous allez créer les services nécessaires à l'analyse des événements de journal générés par vos applications. 

Dans cette section, vous utilisez la ligne de commande pour créer des instances de service. Vous pouvez également procéder de la même manière à partir de la page de service du catalogue à l'aide des liens fournis.
{: tip}

1. Connectez-vous à {{site.data.keyword.cloud_notm}} via la ligne de commande et accédez à votre compte Cloud Foundry. Voir [CLI Getting Started](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Créez une instance Lite de [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. Créez une instance Lite de [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. Créez une instance Standard de [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams).
```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## Création d'une rubrique de messagerie et d'un compartiment {{site.data.keyword.cos_short}} 

{: #topics}

Commencez par créer une rubrique {{site.data.keyword.messagehub}} et un compartiment {{site.data.keyword.cos_short}}. Les rubriques définissent à quel endroit les applications remettent des messages dans les systèmes de messagerie de publication-abonnement. Une fois les messages reçus et traités, ils sont stockés dans un fichier situé dans un compartiment {{site.data.keyword.cos_short}}.

1. Dans votre navigateur, accédez à l'instance de service `log-analysis-hub` à partir des [Ressources](https://{DomainName}/resources?search=log-analysis).
2. Cliquez sur le bouton **+** pour créer une rubrique.
3. Entrez le **Nom de rubrique** `webserver` et cliquez sur le bouton **Créer une rubrique**.
4. Cliquez sur **Données d'identification pour le service** et sur le bouton **Nouvelles données d'identification**.
5. Dans la boîte de dialogue, entrez `webserver-flow` en tant que **Nom** et cliquez sur le bouton **Ajouter**.
6. Cliquez sur **Afficher les données d'identification** et copiez les informations dans un endroit sûr. Elles seront utilisées à la section suivante.
7. De retour dans la [Liste de ressources](https://{DomainName}/resources?search=log-analysis), sélectionnez l'instance de service `log-analysis-cos`.
8. Cliquez sur **Créer un compartiment**.
    * Entrez un **Nom** unique pour le compartiment.
    * Sélectionnez **Inter-région** pour **Résilience**.
    * Sélectionnez **us-geo** comme **Emplacement**.
    * Cliquez sur **Créer un compartiment**.

## Création d'une source de flux Streams 

{: #streamsflow}

Dans cette section, vous allez commencer à configurer un flux Streams recevant des messages de journal. Le service {{site.data.keyword.streaminganalyticsshort}} est optimisé par {{site.data.keyword.streamsshort}}, un programme qui peut analyser des millions d'événements par seconde, permettant ainsi des temps de réponse inférieurs à la milliseconde et une prise de décision instantanée.

1. Dans votre navigateur, accédez à [Watson Data Platform](https://dataplatform.ibm.com).
2. Sélectionnez le bouton ou la vignette **Nouveau projet**, puis la vignette **De base** et cliquez sur **OK**.
    * Entrez le **Nom** `webserver-logs`. 
    * L'option de **Stockage** doit être définie sur `log-analysis-cos`. Si tel n'est pas le cas, sélectionnez l'instance de service.
    * Cliquez sur le bouton **Créer**.
3. Sur la page qui apparaît, sélectionnez l'onglet **Paramètres** et cochez **Streams Designer** dans **Outils**. Terminez en cliquant sur le bouton **Sauvegarder**. 
4. Cliquez sur le bouton **Add to project**, puis sur **Streams flow** dans la barre de navigation supérieure.
    * Cliquez sur **Associate an IBM Streaming Analytics instance with a container-based plan**.
    * Créez une instance {{site.data.keyword.streaminganalyticsshort}} en sélectionnant le bouton d'option **Lite** et en cliquant sur **Create**. Ne sélectionnez pas Lite VM. 
    * Indiquez le **Nom de service**, `log-analysis-sa`, et cliquez sur **Confirmer**.
    * Entrez le **Nom** du flux Streams, `webserver-flow`.
    * Terminez en cliquant sur **Créer**.
5. Sur la page qui apparaît, sélectionnez la vignette **{{site.data.keyword.messagehub}}**.
    * Cliquez sur **Ajouter une connexion** et sélectionnez votre instance `log-analysis-hub` {{site.data.keyword.messagehub}}. Si l'instance ne figure pas dans la liste, sélectionnez l'option **IBM {{site.data.keyword.messagehub}}**. Saisissez manuellement les **Détails de connexion** obtenus à partir des **Données d'identification pour le service** dans la section précédente. **Nommez** la connexion `webserver-flow`.
    * Cliquez sur **Créer** pour créer la connexion.
    * Sélectionnez `webserver` dans la liste déroulante **Rubrique**.
    * Sélectionnez **Start with the first new message** dans la liste déroulante **Décalage initial**.
    * Cliquez sur **Continuer**.
6. Laissez la page **Preview Data** ouverte ; elle sera utilisée dans la section suivante.

## Utilisation des outils de la console Kafka avec {{site.data.keyword.messagehub}}

{: #kafkatools}

La connexion `webserver-flow` est actuellement inactive et en attente de messages. Dans cette section, vous allez configurer les outils de la console Kafka pour qu’ils fonctionnent avec {{site.data.keyword.messagehub}}. Les outils de la console Kafka vous permettent de générer des messages arbitraires à partir du terminal et de les envoyer à {{site.data.keyword.messagehub}}, ce qui déclenche `webserver-flow`.

1. Téléchargez et décompressez le [client Kafka 0.10.2.X](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz).
2. Accédez au répertoire `bin` et créez un fichier texte nommé `message-hub.config` avec le contenu suivant.
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. Remplacez `USER` et `PASSWORD` dans votre fichier `message-hub.config` par les valeurs `user` et `password` indiquées dans les **Données d'identification pour le service** à la section précédente. Sauvegardez `message-hub.config`.
4. Dans le répertoire `bin`, exécutez la commande suivante. Remplacez `KAFKA_BROKERS_SASL` par la valeur `kafka_brokers_sasl` indiquée dans les **Données d'identification pour le service**. Un exemple est fourni.
```sh
    ./kafka-console-producer.sh --broker-list KAFKA_BROKERS_SASL \
    --producer.config message-hub.config --topic webserver
    ```
    {: pre}
    ```sh
    ./kafka-console-producer.sh --broker-list \
    kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093 \
    --producer.config message-hub.config --topic webserver
    ```
5. L'outil de la console Kafka est en attente de saisie. Copiez et collez le message de journal ci-dessous dans le terminal. Appuyez sur `Entrée` pour envoyer le message de journal à {{site.data.keyword.messagehub}}. Notez que les messages envoyés s'affichent également dans la page`webserver-flow`, **Preview Data**.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![Page d'aperçu](images/solution31/preview_data.png)

## Création d'une cible de flux Streams 

{: #streamstarget}

Dans cette section, vous allez effectuer la configuration du flux Streams en définissant une cible. Cette cible est utilisée pour stocker les messages de journal entrants dans le compartiment {{site.data.keyword.cos_short}} créé précédemment. Le processus de stockage et d’ajout des messages de journal entrants dans un fichier est effectué automatiquement par {{site.data.keyword.streaminganalyticsshort}}.

1. Sur la page **Preview Page** de `webserver-flow`, cliquez sur le bouton **Continue**.
2. Sélectionnez la vignette **{{site.data.keyword.cos_full_notm}}** comme cible.
    * Cliquez sur **Ajouter une connexion** et sélectionnez l'instance `log-analysis-cos`. 
    * Cliquez sur **Create**.
    * Entrez la valeur de **File path**, `/YOUR_BUCKET_NAME/http-logs_%TIME.csv`. Remplacez `YOUR_BUCKET_NAME` par le nom du compartiment utilisé dans la première section.
    * Sélectionnez **csv** dans la liste déroulante **Format**.
    * Cochez la case **Column header row** .
    * Sélectionnez **File Size** dans la liste déroulante **File Creation Policy**.
    * Définissez la limite sur 100 Mo en entrant `102400` dans la zone de texte **File Size (KB)**.
    * Cliquez sur **Continue**.
3. Cliquez sur **Save**.
4. Cliquez sur le bouton de **>** lecture pour **démarrer le flux Streams**.
5. Une fois le flux démarré, envoyez à nouveau plusieurs messages de journal à partir de l’outil de la console Kafka. Vous pouvez observer l’arrivée des messages en consultant le flux `webserver-flow` dans Streams Designer.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. Revenez au compartiment dans {{site.data.keyword.cos_short}}. Un fichier `log.csv` est créé dès que suffisamment de messages ont été entrés dans le flux ou que le flux est redémarré. 

![webserver-flow](images/solution31/flow.png)

## Ajout d'un comportement conditionnel aux flux Streams

{: #streamslogic}

Jusqu'à présent, le flux Streams était un simple canal permettant de transférer des messages de {{site.data.keyword.messagehub}} vers {{site.data.keyword.cos_short}}. Il est très probable que les équipes souhaiteront connaître les événements pertinents en temps réel. Par exemple, des équipes spécifiques peuvent tirer parti d'alertes lorsque des événements HTTP 500 (erreur d'application) se produisent. Dans cette section, vous allez ajouter une logique conditionnelle au flux pour identifier les codes HTTP 200 (OK) et autres que HTTP 200. 

1. Utilisez le bouton en forme de crayon pour **éditer le flux Streams**.
2. Créez un noeud de filtre qui gère les réponses HTTP 200. 
    * Dans la palette de **Noeuds**, faites glisser le noeud **Filtrer** de **PROCESSING AND ANALYTICS** vers le canevas.
    * Entrez `OK` dans la zone de texte du nom, qui contient actuellement le mot `Filtrer`.
    * Entrez l'instruction suivante dans la zone de texte **Condition Expression**.
      ```sh
      responseCode == 200
      ```
      {: pre}
    * Avec la souris, tracez une ligne à partir de la sortie du noeud **{{site.data.keyword.messagehub}}** (côté droit) vers l'entrée du noeud **OK** (côté gauche).
    * Dans la palette de **Noeuds**, faites glisser le noeud **Déboguer** se trouvant sous **TARGETS** vers le canevas.
    * Connectez le noeud **Déboguer** au noeud **OK** en traçant une ligne entre les deux.
3. Répétez le processus pour créer un filtre `Not OK` à l'aide des mêmes noeuds et de l'instruction de condition suivante.
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. Cliquez sur le bouton de lecture pour **enregistrer et exécuter le flux Streams**.
5. Si vous y êtes invité, cliquez sur le lien pour **exécuter la nouvelle version**.

![Flow Designer](images/solution31/flow_design.png)

## Augmentation du volume de messages

{: #streamsload}

Pour afficher la gestion conditionnelle dans votre flux Streams, vous augmentez le volume de messages envoyés à {{site.data.keyword.messagehub}}. Le programme Node.js fourni simule un flux réaliste de messages vers {{site.data.keyword.messagehub}} en fonction du trafic sur le serveur Web. Pour démontrer l'évolutivité de {{site.data.keyword.messagehub}} et {{site.data.keyword.streaminganalyticsshort}}, augmentez le débit des messages de journal.

Cette section utilise [node-rdkafka](https://www.npmjs.com/package/node-rdkafka). Consultez la page npmjs pour obtenir des instructions de traitement des incidents en cas d'échec de l'installation du simulateur. Si les problèmes persistent, vous pouvez passer à la section suivante et télécharger manuellement les données. 

1. Téléchargez et décompressez le fichier journal [compressé gzip de 20,7 Mo, au format ASCII, du 01 juillet au 31 juillet](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz) émis par la NASA.
2. Clonez et installez le simulateur de journal à partir d'[IBM-Cloud sur GitHub](https://github.com/IBM-Cloud/kafka-log-simulator).
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. Accédez au répertoire du simulateur et exécutez les commandes suivantes pour configurer le simulateur et générer des messages d'événement de journal. Remplacez `LOGFILE` par le fichier que vous avez téléchargé. Remplacez `BROKERLIST` et `APIKEY` par les **Données d'identification pour le service** correspondantes utilisées précédemment. Un exemple est fourni.
```sh
    npm install
    ```
    ```sh
    npm run build
    ```
    ```sh
    node dist/index.js --file LOGFILE --parser httpd --broker-list BROKERLIST \
    --api-key APIKEY --topic webserver --rate 100
    ```
    {: pre}
    ```sh
    node dist/index.js --file /Users/ibmcloud/Downloads/NASA_access_log_Jul95 \
    --parser httpd --broker-list \
    "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093" \
    --api-key Np15YZKN3SCdABUsOpJYtpue6jgJ7CwYgsoCWaPbuyFbdM4R \
    --topic webserver --rate 100
    ```
4. Dans votre navigateur, revenez à `webserver-flow` une fois que le simulateur a commencé à générer des messages.
5. A l'aide de `control+C`, arrêtez le simulateur après qu'un nombre souhaité de messages a traversé les branches conditionnelles. 
6. Testez la mise à l'échelle de {{site.data.keyword.messagehub}} en augmentant ou en diminuant la valeur `--rate`.

Le simulateur retarde l'envoi du message suivant en fonction du temps écoulé dans le journal du serveur Web. La définition de l'option `--rate 1` envoie des événements en temps réel. Le paramètre `--rate 100` signifie que, pour chaque seconde écoulée dans le journal du serveur Web, un délai de 10 ms entre les messages est utilisé.
{: tip}

![Débit réglé à 10](images/solution31/flow_load_10.png)

## Analyse des données de journal à l'aide de SQL Query 

{: #sqlquery}

En fonction du nombre de messages envoyés par le simulateur, la taille du fichier journal sur {{site.data.keyword.cos_short}} a certainement augmenté. Vous intervenez ensuite en tant qu'investigateur répondant aux questions d'audit ou de conformité en combinant SQL Query avec votre fichier journal. L'utilisation de SQL Query présente l'avantage de rendre le fichier journal directement accessible. Aucune transformation supplémentaire ni serveur de base de données n'est nécessaire. 

Si vous préférez ne pas attendre que le simulateur envoie tous les messages du journal, téléchargez le [fichier CSV complet](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp) dans {{site.data.keyword.cos_short}} pour commencer immédiatement.
{: tip}

1. Accédez à l'instance de service `log-analysis-sql` à partir de la [Liste de ressources](https://{DomainName}/resources?search=log-analysis). Sélectionnez **Open UI** pour lancer SQL Query.
2. Entrez le code SQL suivant dans la zone de texte **Type SQL here ...**.
    ```sql
    -- What are the top 10 web pages on NASA from July 1995?
    -- Which mission might be significant?
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. Extrayez l'URL du code SQL de l'objet du fichier journal. 
    * Dans la [Liste de ressources](https://{DomainName}/resources?search=log-analysis), sélectionnez l'instance de service `log-analysis-cos`.
    * Sélectionnez le compartiment que vous avez créé précédemment. 
    * Dans le menu déroulant dynamique, cliquez sur le fichier `http-logs_TIME.csv` et sélectionnez **Object SQL URL**.
    * **Copiez** l'URL dans le presse-papiers.
4. Mettez à jour la clause `FROM` avec l'URL du code SQL et cliquez sur **Run**.
5. Le résultat apparaît dans l'onglet **Result**. Bien que certaines pages - comme la page d’accueil du Kennedy Space Center - soient attendues, une mission est très populaire. 
6. Sélectionnez l'onglet **Query Details** pour afficher des informations supplémentaires telles que l'emplacement où le résultat a été stocké sur {{site.data.keyword.cos_short}}.
7. Essayez les paires question-réponse suivantes en les ajoutant individuellement à la zone de texte **Type SQL here ...**.
    ```sql
    -- Who are the top 5 viewers?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- Which viewer has suspicious activity based on application failures?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- Which requests showed a page not found error to the user?
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- What are the top 10 largest files?
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- What is the distribution of total traffic by hour?
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- Why did the previous result return an empty hour?
    -- Hint, find the malformed hostname.
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

Les clauses FROM ne sont pas limitées à un seul fichier. Utilisez `cos://us-geo/YOUR_BUCKET_NAME/` pour exécuter des requêtes SQL sur tous les fichiers du compartiment.
{: tip}

## Extension du tutoriel 

{: #expand}

Félicitations, vous avez créé un pipeline d’analyse de journaux avec {{site.data.keyword.cloud_notm}}. Vous trouverez ci-dessous des suggestions supplémentaires pour améliorer votre solution. 

* Utilisez des cibles supplémentaires dans Streams Designer pour stocker des données dans [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) ou exécuter du code dans [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* Suivez le tutoriel [Génération d'un lac de données à l'aide d'Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage) pour ajouter un tableau de bord permettant de journaliser des données
* Intégrez des systèmes supplémentaires à {{site.data.keyword.messagehub}} à l'aide d'[{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect).

## Suppression de services

{: #removal}

Dans la [Liste des ressources](https://{DomainName}/resources?search=log-analysis), utilisez l'option de menu **Supprimer** ou **Supprimer un service** dans le menu déroulant dynamique afin de supprimer les instances de service suivantes.

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## Contenu associé

{:related}

* [Apache Kafka](https://kafka.apache.org/)
