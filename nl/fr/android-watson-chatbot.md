---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# Génération d'un robot conversationnel Android à commande vocale
{: #android-watson-chatbot}

Découvrez à quel point il est facile de créer rapidement un robot conversationnel natif pour Android doté de la voix avec les services {{site.data.keyword.conversationshort}}, {{site.data.keyword.texttospeechshort}} et {{site.data.keyword.speechtotextshort}} sur {{site.data.keyword.Bluemix_short}}.

Ce tutoriel vous guide dans le processus de définition des intentions et des entités et dans la création d'un flux de dialogue permettant à votre robot conversationnel de répondre aux requêtes des clients. Vous allez apprendre comment activer les services {{site.data.keyword.speechtotextshort}} et {{site.data.keyword.texttospeechshort}} pour faciliter l'interaction avec l'application Android.
{:shortdesc}

## Objectifs
{: #objectives}

- Utiliser {{site.data.keyword.conversationshort}} pour personnaliser et déployer un robot conversationnel.
- Permettre aux utilisateurs finaux d'interagir avec le robot conversationnel en utilisant la voix et l'audio. 
- Configurer et exécuter l'application Android. 

## Services utilisés
{: #services}

Ce tutoriel utilise les produits suivants : 

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## Architecture
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* Les utilisateurs interagissent avec une application mobile à l'aide de la voix.
* Le son est transcrit en texte avec {{site.data.keyword.speechtotextfull}}.
* Le texte est transmis à {{site.data.keyword.conversationfull}}.
* La réponse d'{{site.data.keyword.conversationfull}} est convertie en audio par {{site.data.keyword.texttospeechfull}} et le résultat est renvoyé à l'application mobile.

## Avant de commencer
{: #prereqs}

- Téléchargez et installez [Android Studio](https://developer.android.com/studio/index.html).

## Création de services
{: #setup}

Dans cette section, vous créez les services requis par le tutoriel, en commençant par {{site.data.keyword.conversationshort}}, pour créer des assistants virtuels cognitifs qui aident vos clients.

1. Accédez au [**catalogue {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) et sélectionnez le service [{{site.data.keyword.conversationshort}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation) > forfait **Lite** :
   1. Définissez **Nom** sur **android-chatbot-assistant**
   1. **Créez**.
2. Cliquez sur **Données d'identification du service** dans le volet gauche, puis sur **Nouvelles données d'identification**.
   1. Définissez **Nom** sur **for-android-app**
   1. **Ajoutez**.
3. Cliquez sur **Afficher les données d'identification** pour afficher les données d'identification. Notez la **Clé d'API** et l'**URL**, vous en aurez besoin pour l'application mobile.

Le service {{site.data.keyword.speechtotextshort}} convertit la voix humaine en texte écrit pouvant être envoyé en tant qu'entrée au service {{site.data.keyword.conversationshort}} sur {{site.data.keyword.Bluemix_short}}.

1. Accédez au [**catalogue {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) et sélectionnez le service [{{site.data.keyword.speechtotextshort}}](https://{DomainName}/catalog/services/speech-to-text) > forfait **Lite**.
   1. Définissez **Nom** sur **android-chatbot-stt**
   1. **Créez**.
2. Cliquez sur **Données d'identification du service** dans le volet gauche, puis sur **Nouvelles données d'identification** pour ajouter des données d'identification.
   1. Définissez **Nom** sur **for-android-app**
   1. **Ajoutez**.
3. Cliquez sur **Afficher les données d'identification** pour afficher les données d'identification. Notez la **Clé d'API** et l'**URL**, vous en aurez besoin pour l'application mobile.

Le service {{site.data.keyword.texttospeechshort}} traite le texte et le langage naturel pour générer une sortie audio synthétisée avec une cadence et une intonation appropriées. Le service fournit plusieurs voix et peut être configuré dans l'application Android. 

1. Accédez au [**catalogue {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) et sélectionnez le service [{{site.data.keyword.texttospeechshort}}](https://{DomainName}/catalog/services/text-to-speech) > forfait **Lite**.
   1. Définissez **Nom** sur **android-chatbot-tts**
   1. **Créez**.
2. Cliquez sur **Données d'identification du service** dans le volet gauche, puis sur **Nouvelles données d'identification** pour ajouter des données d'identification.
   1. Définissez **Nom** sur **for-android-app**
   1. **Ajoutez**.
3. Cliquez sur **Afficher les données d'identification** pour afficher les données d'identification. Notez la **Clé d'API** et l'**URL**, vous en aurez besoin pour l'application mobile.

## Création d'une compétence
{: #create_workspace}

Une compétence est un conteneur d'artefacts qui définissent le flux de conversation. 

Pour ce tutoriel, vous allez enregistrer et utiliser le fichier [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) avec des intentions, des entités et un flux de dialogue prédéfinis sur votre ordinateur. 

1. Dans la page des détails du service {{site.data.keyword.conversationshort}}, accédez à **Manage** dans le volet de gauche, cliquez sur **Launch tool** pour afficher le tableau de bord {{site.data.keyword.conversationshort}}.
1. Cliquez sur l'onglet **Skills**.
1. **Créez une nouvelle compétence**, puis **importez-la** et choisissez le fichier JSON téléchargé ci-dessus.
1. Sélectionnez l'option **Everything** et cliquez sur **Import**. Une compétence est créée avec des intentions, des entités et un flux de dialogue prédéfinis. 
1. Revenez à la liste des compétences. Sélectionnez le menu d'actions de la compétence `Ana` pour **afficher les détails de l'API**.

### Définition d'une intention 
{:#define_intent}

Une intention représente l'objectif d'une entrée de l'utilisateur, tel que la réponse à une question ou le traitement d'une facture. Vous définissez une intention pour chaque type de demande d'utilisateur qui doit être prise en charge par votre application. En reconnaissant l'intention exprimée dans l'entrée d'un utilisateur, le service {{site.data.keyword.conversationshort}} peut choisir le flux de dialogue approprié pour y répondre. Dans l'outil, le nom de l'intention est toujours précédé du préfixe `#`.

En termes simples, les intentions sont les intentions de l'utilisateur final. Voici des exemples de noms d'intention. 
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. Cliquez sur la nouvelle compétence - **Ana**.

   Ana est un robot d’assurance permettant aux utilisateurs d’interroger leurs prestations de santé et de déposer des demandes de remboursement.
   {:tip}
2. Cliquez sur le premier onglet pour visualiser toutes les **intentions**.
3. Cliquez sur **Add intent** pour créer une intention. Entrez `Cancel_Policy` comme nom d'intention après le symbole `#` et indiquez une description facultative.
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. Cliquez sur **Create intent**.
5. Ajoutez des exemples d'entrées utilisateur où il est demandé de résilier un contrat d'assurance 
   - `Je souhaite résilier mon contrat`
   - `Résilier mon contrat maintenant`
   - `Je souhaite cesser les versements sur mon contrat.`
6. Ajoutez des exemples d'entrée utilisateur les uns après les autres et cliquez sur **add example**. Répétez ces étapes pour tous les autres exemples d'entrée utilisateur.

   N'oubliez pas d'ajouter au moins 5 exemples d'entrée utilisateur pour mieux former votre robot conversationnel.
   {:tip}

7. Cliquez sur le bouton **de fermeture** ![](images/solution28-watson-chatbot-android/close_icon.png) en regard du nom de l'intention pour enregistrer l'intention.
8. Cliquez sur **Content Catalog** et sélectionnez **General**. Cliquez sur **Add to skill**.

   Le catalogue de contenu vous aide à démarrer plus rapidement en ajoutant les intentions existantes (banque, service client, assurance, telco, commerce électronique, etc.). Ces intentions sont formées aux questions courantes que les utilisateurs peuvent poser.
   {:tip}

### Définition d'une entité
{:#define_entity}

Une entité représente un terme ou un objet qui correspond à vos intentions et qui fournit un contexte spécifique pour une intention. Répertoriez les valeurs possibles pour chaque entité et les synonymes que les utilisateurs peuvent entrer. En reconnaissant les entités qui sont mentionnées dans l'entrée utilisateur, le service {{site.data.keyword.conversationshort}} peut choisir les actions spécifiques à exécuter pour satisfaire une intention. Dans l'outil, le nom d'une entité est toujours précédé du préfixe `@`.

Voici des exemples de noms d'entité 
 - `@location`
 - `@menu_item`
 - `@product`

1. Cliquez sur l'onglet **Entities** pour visualiser les entités existantes.
2. Cliquez sur **Add entity** et entrez le nom de l'entité, `location`, après `@`. Cliquez sur **Create entity**.
3. Entrez `address` comme nom de valeur et sélectionnez **Synonyms**.
4. Ajoutez `place` en tant que synonyme et cliquez sur l'icône ![](images/solution28-watson-chatbot-android/plus_icon.png). Répétez l'opération avec synonymes `office`, `centre`, `branch`, etc., puis cliquez sur **Add Value**.
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. Cliquez sur **close** ![](images/solution28-watson-chatbot-android/close_icon.png) pour enregistrer les modifications.
6. Cliquez sur l'onglet **System entities** pour vérifier les entités communes créées par IBM pouvant être utilisées dans tous les cas d'utilisation.

   Les entités de système peuvent être utilisées pour reconnaître un large éventail de valeurs pour les types d'objet qu'elles représentent. Par exemple, l'entité de système `@sys-number` correspond à n'importe quelle valeur numérique, y compris des nombres entiers, des fractions décimales ou même des nombres écrits sous forme de mots.
   {:tip}
7. Activez ou désactivez **Status** (`on/off`) pour les entités système @sys-person et @sys-location.

### Génération du flux de dialogue 
{:#build_dialog}

Un dialogue est un flux conversationnel à ramifications qui définit la réponse de votre application lorsqu'elle reconnaît les intentions et les entités définies. Vous utilisez le générateur de dialogue inclus dans l'outil pour créer des conversations avec les utilisateurs, en fournissant des réponses basées sur les intentions et les entités reconnues dans leurs entrées.

1. Cliquez sur l'onglet **Dialog** pour visualiser le processus de dialogue existant avec les intentions et les entités.
2. Cliquez sur **Add node** pour ajouter un nouveau noeud au dialogue.
3. Sous **if assistant recognizes**, entrez `#Cancel_Policy`.
4. Sous **Then respond with**, entrez la réponse `Ce service n'est pas disponible en ligne. Veuillez vous rendre à notre agence la plus proche pour résilier votre contrat.`
5. Cliquez sur ![](images/solution28-watson-chatbot-android/save_node.png) pour fermer et enregistrer le noeud.
6. Faites défiler l'écran pour visualiser le noeud `#greeting`. Cliquez sur le noeud pour afficher les détails.
   ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. Cliquez sur l'icône ![](images/solution28-watson-chatbot-android/add_condition.png) pour **ajouter une nouvelle condition**. Sélectionnez `or` dans la liste déroulante et entrez `#General_Greetings` en tant qu'intention. **Then respond with section** affiche la réponse de l'assistant lorsqu'il est salué par l'utilisateur.
   ![](images/solution28-watson-chatbot-android/apply_condition.png)

   Une variable contextuelle est une variable que vous définissez dans un noeud et à laquelle vous attribuez éventuellement une valeur. D'autres noeuds ou logiques d'application peuvent ensuite définir ou modifier la valeur de la variable contextuelle. L'application peut transmettre des informations au dialogue, et le dialogue peut mettre à jour ces informations et les transmettre en retour à l'application ou à un noeud suivant. Pour cela, le dialogue utilise des variables contextuelles.
{:tip}

8. Testez le flux de dialogue à l'aide du bouton **Try it**.

## Liaison de la compétence à un assistant 

Un **assistant** est un bot cognitif que vous pouvez personnaliser en fonction des besoins de votre entreprise et que vous pouvez déployer sur plusieurs canaux pour apporter de l'aide à vos clients où et quand ils en ont besoin. Vous personnalisez l'assistant en y ajoutant les **compétences** nécessaires pour répondre aux objectifs de vos clients.

1. Dans l'outil {{site.data.keyword.conversationshort}}, accédez à **Assistants** et utilisez **Create new**.
   1. Définissez **Nom** sur **android-chatbot-assistant**
   1. **Créez**.
1. Utilisez **Add Dialog skill** pour sélectionner la compétence créée dans les sections précédentes.
   1. **Ajoutez une compétence existante**
   1. Sélectionnez **Ana**
1. Sélectionnez le menu d'actions dans Assistant > **Settings** > **API Details**, notez l'**Assistant ID**. Vous devez y faire référence à partir de l'application mobile (dans le fichier `config.xml` de l'application Android).

## Configuration et exécution de l'application Android 
{:#configure_run_android_app}

Le référentiel contient le code d'application Android avec les dépendances gradle requises. 

1. Exécutez la commande ci-dessous pour cloner le [référentiel GitHub](https://github.com/IBM-Cloud/chatbot-watson-android) :
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. Lancez Android Studio > **Ouvrez un projet Android Studio existant** et pointez sur le code téléchargé. La génération du **Gradle** se déclenche automatiquement et toutes les dépendances sont téléchargées.
3. Ouvrez `app/src/main/res/values/config.xml` pour afficher les marques de réservation (`ASSISTANT_ID_HERE`) des données d'identification du service. Entrez les données d'identification du service (que vous avez enregistrées précédemment) dans leurs marques de réservation respectives et enregistrez le fichier. 
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <!--Watson Assistant service credentials-->
       <!-- REPLACE `ASSISTANT_ID_HERE` with ID of the Assistant to use -->
       <string name="assistant_id">ASSISTANT_ID_HERE</string>

       <!-- REPLACE `ASSISTANT_API_KEY_HERE` with Watson Assistant service API Key-->
       <string name="assistant_apikey">ASSISTANT_API_KEY_HERE</string>

       <!-- REPLACE `ASSISTANT_URL_HERE` with Watson Assistant service URL-->
       <string name="assistant_url">ASSISTANT_URL_HERE</string>

       <!--Watson Speech To Text(STT) service credentials-->
       <!-- REPLACE `STT_API_KEY_HERE` with Watson Speech to Text service API Key-->
       <string name="STT_apikey">STT_API_KEY_HERE</string>

       <!-- REPLACE `STT_URL_HERE` with Watson Speech to Text service URL-->
       <string name="STT_url">STT_URL_HERE</string>

       <!--Watson Text To Speech(TTS) service credentials-->
       <!-- REPLACE `TTS_API_KEY_HERE` with Watson Text to Speech service API Key-->
       <string name="TTS_apikey">TTS_API_KEY_HERE</string>

       <!-- REPLACE `TTS_URL_HERE` with Watson Text to Speech service URL-->
       <string name="TTS_url">TTS_URL_HERE</string>
   </resources>
   ```
4. Générez le projet et démarrez l'application sur une unité réelle ou avec un simulateur.
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. **Entrez votre requête** dans l'espace prévu ci-dessous et cliquez sur l'icône en forme de flèche pour l'envoyer au service {{site.data.keyword.conversationshort}}.
6. La réponse est transmise au service {{site.data.keyword.texttospeechshort}} et vous devriez entendre une voix la lire. 
7. Cliquez sur l'icône du **micro** située dans l'angle inférieur gauche de l'application pour saisir un discours qui est converti en texte et peut ensuite être envoyé au service {{site.data.keyword.conversationshort}} en cliquant sur l'icône en forme de flèche.


## Suppression de ressources 
{:removeresources}

1. Accédez à la [Liste des ressources,](https://{DomainName}/resources/)
1. Supprimez les services que vous avez créés : 
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## Contenu associé
{:related}

- [Création d'entités, de synonymes, d'entités système](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [Variables de contexte](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [Génération d'un dialogue complexe](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [Collecte d'informations à l'aide d'attributs](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [Options de déploiement](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [Amélioration de vos compétences ](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
