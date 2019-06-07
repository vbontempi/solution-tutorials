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

# Examen des services, des ressources et de l’utilisation d'{{site.data.keyword.cloud_notm}}
{: #cloud-usage}
L'adoption croissante du cloud impose aux directeurs informatiques et financiers de se familiariser avec cette approche en termes d'innovation et de contrôle des coûts. En effet, il est possible de répondre à des questions telles que : "Quels services les équipes utilisent-elles ?" "Combien coûte l'exploitation d'une solution ?" et "Comment puis-je contenir les dépenses ?" en examinant les données disponibles. Ce tutoriel présente des approches permettant d'explorer ces informations et de répondre aux questions liées à une utilisation courante.
{:shortdesc}

## Objectifs
{: #objectives}
* Détailler les artefacts {{site.data.keyword.cloud_notm}} : applications et services Cloud Foundry, ressources Identity and Access Management et unités Infrastructure
* Associer les artefacts {{site.data.keyword.cloud_notm}} à l'utilisation et à la facturation
* Définir les relations entre les artefacts {{site.data.keyword.cloud_notm}} et les équipes de développement
* Exploiter les données d'utilisation et de facturation pour créer des jeux de données à des fins comptables 

## Architecture
{: #architecture}

![Architecture](images/solution38/Architecture.png)

## Avant de commencer
{: #prereqs}

* Installez [l'interface CLI {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* Installez [cURL](https://curl.haxx.se/)
* Installez [Node.js](https://nodejs.org/)
* Installez [json2csv](https://www.npmjs.com/package/json2csv) à l'aide de la commande `npm install -g json2csv`
* Installez [jq](https://stedolan.github.io/jq/)

## Contexte 
{: #background}

Avant d'exécuter des commandes qui répertorient et détaillent l'utilisation d'{{site.data.keyword.cloud_notm}}, il est utile de connaître les grandes catégories d'utilisation et leur fonction. Les termes clés utilisés ultérieurement dans le tutoriel sont en gras. Une visualisation utile des artefacts ci-dessous est disponible dans la [documentation sur la gestion du compte](https://{DomainName}/docs/account?topic=account-overview#overview).

### Cloud Foundry
Cloud Foundry est une plateforme sous forme de service (PaaS) open source sur {{site.data.keyword.cloud_notm}} qui vous permet de déployer et de mettre à l'échelle des applications et des **Services** sans gérer de serveurs. Cloud Foundry organise les applications et les services en organisations ou espaces. Une **organisation** est un compte de développement qu'un ou plusieurs utilisateurs peuvent posséder et utiliser. Une organisation peut contenir plusieurs espaces. Chaque **espace** offre aux utilisateurs un accès à un emplacement partagé pour le développement, le déploiement et la maintenance d'applications.

### Identity and Access Management
IBM Cloud Identity and Access Management (IAM) vous permet d'authentifier en toute sécurité les utilisateurs des services de plateforme et de contrôler l'accès aux ressources de manière cohérente sur la plateforme {{site.data.keyword.cloud_notm}}. Des offres plus récentes et des services Cloud Foundry migrés existent en tant que **Ressources** gérées par {{site.data.keyword.cloud_notm}} Identity and Access Management. Les ressources sont organisées en **Groupes de ressources** et fournissent un contrôle d'accès par le biais de règles et de rôles.

### Infrastructure
L'infrastructure englobe diverses options de calcul : serveurs bare metal, instances de serveur virtuel et noeuds Kubernetes. Chacune est considérée comme une **Unité** dans la console. 

### Compte
Les artefacts susmentionnés sont associés à un **Compte** à des fins de facturation. Les **utilisateurs** sont invités au compte et se voient accorder l'accès à différentes ressources du compte.

## Octroi de droits
Pour afficher l'inventaire et l'utilisation du cloud, vous devez disposer des rôles appropriés, attribués par l'administrateur du compte. Si vous êtes l'administrateur du compte, passez à la section suivante. 

1. L'administrateur du compte doit se connecter à {{site.data.keyword.cloud_notm}} et accéder à la page [**Utilisateurs Identity & Access**](https://{DomainName}/iam/#/users).
2. L'administrateur du compte peut sélectionner votre nom dans la liste des utilisateurs du compte pour attribuer les droits appropriés. 
3. Dans l'onglet **Règles d'accès**, cliquez sur le bouton **Affecter un accès** et effectuez les modifications suivantes.
   1. Sélectionnez la vignette **Affecter l'accès au sein d'un groupe de ressources**. Sélectionnez le ou les **Groupes de ressources** auxquels l'accès doit être accordé et appliquez le rôle **Afficheur**. Terminez en cliquant sur le bouton **Affecter**. Ce rôle est requis pour les commandes `resource groups` et `billing resource-group-usage`.
   2. Sélectionnez la vignette **Affecter l'accès à l'aide de Cloud Foundry**. Sélectionnez le menu déroulant dynamique en regard de chaque **Organisation** à laquelle l'accès doit être octroyé. Sélectionnez **Editer le rôle d'organisation** dans le menu. Sélectionnez **Responsable de la facturation** dans la liste **Rôles de l'organisation**. Terminez en cliquant sur le bouton **Sauvegarder un rôle**. Ce rôle est requis pour la commande `billing org-usage`.
   3. Sélectionnez la vignette **Affecter l'accès aux ressources**. Sélectionnez **Tous les services avec l'offre Identity and Access activée** dans le menu déroulant **Services**. Vérifiez le rôle **Editeur** sous **Affecter des rôles d'accès à une plateforme**. Ce rôle est requis pour la commande `resource tag-attach`.

## Localisation de ressources avec la commande de recherche 
{: #search}

Avec l'utilisation croissante des services cloud par les équipes de développement, les responsables ont tout intérêt à savoir quels services sont déployés. Les informations de déploiement permettent de répondre aux questions relatives à l’innovation et à la gestion des services : 
- Quelles compétences liées au service peuvent être partagées entre les équipes pour améliorer d'autres projets ? 
- Quels points communs entre les équipes pourraient manquer à certaines ? 
- Quelles équipes utilisent un service qui nécessite un correctif critique ou sera bientôt obsolète ? 
- Comment les équipes peuvent-elles examiner leurs instances de service pour minimiser l'augmentation des coûts ? 

La recherche ne se limite pas aux services et aux ressources. Vous pouvez également interroger des artefacts Cloud tels que des organisations et des espaces Cloud Foundry, des groupes de ressources, des liaisons de ressources, des alias, etc. Pour plus d'exemples, consultez la documentation sur la [recherche de ressources ibmcloud](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search).
{:tip}

1. Connectez-vous à {{site.data.keyword.cloud_notm}} via la ligne de commande et accédez à votre compte Cloud Foundry. Voir [CLI Getting Started](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh
    ibmcloud login
    ```
    {: pre}

    ```sh
    ibmcloud target --cf
    ```
    {: pre}

2. Dressez l'inventaire de tous les services Cloud Foundry utilisés dans le compte.
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. Des filtres booléens peuvent être appliqués pour élargir ou restreindre les recherches. Par exemple, recherchez les services et les applications Cloud Foundry, ainsi que les ressources IAM, à l'aide de la requête ci-dessous.
    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. Pour notifier les équipes à l'aide d'un type de service particulier, effectuez une requête avec le nom du service. Remplacez `<name>` par du texte, par exemple `weather` ou `cloudant`. Ensuite, obtenez le nom de l'organisation en remplaçant `<Organization ID>` par l'**ID organisation**.
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

L'étiquetage et la recherche peuvent être utilisés ensemble pour fournir une identification personnalisée des ressources. Cela implique de lier l'étiquette à une ou plusieurs ressources et d'effectuer une recherche à l'aide d'un ou de plusieurs noms d'étiquette. Créez une étiquette nommée env:tutorial.

1. Reliez l'étiquette à une ressource. Vous pouvez obtenir un CRN de ressource à partir de l'interface utilisateur ou avec `ibmcloud resource service-instance <name|id>`. Les noms de ressources Cloud (Cloud Resource Names - CRN) identifient de manière unique les ressources IBM Cloud. 
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <resource CRN>
   ```
   {: pre}
1. Recherchez les artefacts Cloud correspondant à une étiquette donnée à l'aide de la requête ci-dessous. 
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

En combinant des requêtes de recherche avancées à un schéma d'étiquetage convenu par l'entreprise, les responsables et les chefs d'équipe peuvent plus facilement identifier les applications, les ressources et les services Cloud et prendre des mesures adéquates. 

## Exploration de l'utilisation avec le Tableau de bord de l'utilisation
{: #dashboard}

Une fois que la direction est informée des services utilisés par les équipes, la question suivante est souvent posée : "Combien coûte l'exploitation de ces services ?" Le moyen le plus simple de déterminer l'utilisation et le coût consiste à consulter le Tableau de bord de l'utilisation d'{{site.data.keyword.cloud_notm}}. 

1. Connectez-vous à {{site.data.keyword.cloud_notm}} et accédez au [Tableau de bord d'utilisation de compte](https://{DomainName}/account/usage).
2. Dans le menu déroulant **Groupes**, sélectionnez une organisation Cloud Foundry pour afficher l'utilisation du service.
3. Pour une **Offre de services** donnée, cliquez sur **Afficher les instances** pour afficher les instances de service créées.
4. Sur la page suivante, choisissez une instance et cliquez sur **Afficher l'instance**. La page affichée fournit des détails sur l'instance, tels que l'Organisation, l'Espace et l'Emplacement, ainsi que des éléments de poste individuels permettant de générer un coût total. 
5. A l'aide des éléments de navigation, consultez le [Tableau de bord de l'utilisation](https://{DomainName}/account/usage).
6. Dans le menu déroulant **Groupes**, définissez le sélecteur sur **Groupes de ressources** et sélectionnez un groupe tel que **default**.
7. Procédez à un examen similaire des instances disponibles. 

## Exploration de l'utilisation à partir de la ligne de commande

Dans cette section, vous allez explorer l'utilisation avec l'interface de ligne de commande. 

1. Répertoriez toutes les organisations Cloud Foundry disponibles et définissez une variable d'environnement pour en stocker une à des fins de test.
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<org name>
    ```
    {: pre}

2. Détaillez la facturation et l'utilisation pour une organisation donnée à l'aide de la commande `billing`. Spécifiez un mois particulier à l'aide de l'indicateur `-d` avec une date au format AAAA-MM.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Procédez à un examen similaire pour les ressources. Chaque compte a un groupe de ressources `default`. Remplacez la valeur `<group>` d'un groupe de ressources répertorié dans la première commande.
```sh
    ibmcloud resource groups
    ```
    {: pre}

    ```sh
    export RESOURCE_GROUP=<group>
    ```
    {: pre}

    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP
    ```
    {: pre}

4. Vous pouvez afficher les services Cloud Foundry et les ressources IAM à l'aide de la commande `resource-instances-usage`. Selon votre niveau d'autorisation, exécutez les commandes appropriées. 
    - Si vous êtes l'administrateur du compte ou si vous avez obtenu le rôle Administrateur pour Tous les services avec l'offre Identity and Access activée, exécutez simplement la commande `resource-instances-usage`.
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  Si vous n'êtes pas l'administrateur du compte, vous pouvez utiliser les commandes suivantes, car le rôle Afficheur a été défini au début du tutoriel.
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. Pour afficher les unités d'infrastructure, utilisez la commande `sl`. Utilisez ensuite la commande `vs` pour passer en revue {{site.data.keyword.virtualmachinesshort}}.
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## Exportation des données d'utilisation à partir de la ligne de commande
{: #usagecmd}

Certains responsables ont souvent besoin d'exporter les données vers une autre application. Un exemple courant est l’exportation de données d’utilisation dans une feuille de calcul. Dans cette section, vous allez exporter les données d'utilisation au format CSV (valeurs séparées par une virgule), compatible avec la plupart des applications de tableur. 

Cette section utilise deux outils tiers :`jq` et `json2csv`. Chacune des commandes ci-dessous est composée de trois étapes : obtention des données d’utilisation sous forme de fichier JSON, analyse du fichier JSON et formatage du résultat sous forme de fichier CSV. Le processus d'obtention et de conversion des données JSON n'est pas limité à `json2csv` et d'autres outils peuvent être exploités. 

Utilisez l'option `-p` pour imprimer des résultats formatés. Si les données sont mal imprimées, supprimez l'argument `-p` pour imprimer les données CSV brutes.
 {:tip}

1. Exportez l'utilisation d'un groupe de ressources avec les coûts anticipés pour chaque type de ressource.
```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. Détaillez les instances pour chaque type de ressource du groupe de ressources.
```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. Adoptez la même approche pour une organisation Cloud Foundry.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. Ajoutez des coûts anticipés aux données à l'aide d'une requête `jq` plus avancée. Vous créez ainsi plus de lignes car certains types de ressources ont plusieurs métriques de coûts.
```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. Utilisez la même requête `jq` pour répertorier également les ressources Cloud Foundry avec les coûts associés.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. Exportez les données dans un fichier en supprimant l’option `-p` et en injectant la sortie dans un fichier. Le fichier CSV obtenu peut être ouvert avec un tableur.
```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## Exportation des données d'utilisation à l'aide d'API
{: #usageapi}

Bien que les commandes de facturation (`billing`) soient utiles, il est fastidieux d’essayer d’assembler une vue globale à partir de l'interface de ligne de commande. De même, le Tableau de bord de l'utilisation présente un aperçu des organisations et des groupes de ressources, mais pas nécessairement l'utilisation d'une équipe ou d'un projet. Dans cette section, vous allez commencer à explorer une approche davantage basée sur les données pour obtenir une utilisation pour des exigences personnalisées. 

1. Dans le terminal, définissez la variable d'environnement `IBMCLOUD_TRACE=true` pour imprimer les demandes et les réponses de l'API.
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. Réexécutez la commande `billing org-usage` pour afficher les appels d'API. Notez que plusieurs hôtes et routes d’API sont utilisés pour cette commande unique.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Obtenez un jeton OAuth et exécutez l'un des appels d'API figurant dans la commande `billing org-usage`. Notez que certaines API utilisent le jeton UAA tandis que d'autres peuvent utiliser un jeton IAM comme autorisation du support.
    ```sh
    export IAM_TOKEN=`ibmcloud iam oauth-tokens | head -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    export UAA_TOKEN=`ibmcloud iam oauth-tokens | tail -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    curl -H "Authorization: Bearer $UAA_TOKEN" https://mccp.us-south.cf.cloud.ibm.com/v2/organizations?q=name:$ORG_NAME
    ```
    {: pre}

4. Pour exécuter les API d'infrastructure, obtenez et définissez les variables d'environnement pour vos **Nom d'utilisateur de l'API** et **Clé d'authentification** figurant dans votre [Profil utilisateur](https://control.softlayer.com/account/user/profile). Si vous ne voyez pas de Nom d'utilisateur de l'API ni de Clé d'authentification, vous pouvez en créer dans le menu **Actions** en regard de votre nom dans la [Liste d'utilisateurs](https://control.softlayer.com/account/users).
    ```sh
    export IAAS_USERNAME=<API Username>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<Authentication Key>
    ```
    {: pre}

5. Obtenez les totaux et les éléments de facturation d'infrastructure à l'aide des API suivantes. Des API similaires sont documentées [ici](https://softlayer.github.io/reference/services/SoftLayer_Account/).
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. Désactivez le traçage.
```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

Bien que l'approche basée sur les données offre la plus grande flexibilité pour explorer leur utilisation, une explication plus détaillée dépasserait le cadre de ce tutoriel. Le projet GitHub [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample) a été créé et combine les services {{site.data.keyword.cloud_notm}} avec des fonctions Cloud pour fournir un exemple de mise en oeuvre.

## Extension du tutoriel 
{: #expand}

Utilisez les suggestions suivantes pour étendre votre investigation aux données d'inventaire et d'utilisation. 

- Explorez les commandes `ibmcloud billing` avec l'option `--output json`. Vous découvrirez des propriétés de données supplémentaires disponibles et non traitées dans ce tutoriel. 
- Consultez l'article de blogue [Using the {{site.data.keyword.cloud_notm}} command line to find resources](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/) (Utilisation de la ligne de commande IBM Cloud pour rechercher des ressources) afin d'obtenir davantage d'exemples sur la recherche de ressources ibmcloud (`ibmcloud resource search`) et les propriétés pouvant être utilisées dans vos requêtes.
- Recherchez dans [Infrastructure Account APIs](https://softlayer.github.io/reference/services/SoftLayer_Account/) (API de compte d'infrastructure) des API supplémentaires pour étudier l'utilisation de l'infrastructure. 
- Consultez l'aide [jq Manual](https://stedolan.github.io/jq/manual/) pour connaître les requêtes avancées permettant d'agréger des données d'utilisation.
