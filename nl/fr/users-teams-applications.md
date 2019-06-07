---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# Bonnes pratiques en matière d'organisation des utilisateurs, des équipes et des applications 
{: #users-teams-applications}

Ce tutoriel présente les concepts disponibles dans {{site.data.keyword.cloud_notm}} pour la gestion des identités et des accès, ainsi que la manière dont ces concepts peuvent être mis en oeuvre pour prendre en charge les multiples étapes de développement d'une application.
{:shortdesc}

Lors de la création d'une application, il est très courant de définir plusieurs environnements reflétant le cycle de vie de développement d'un projet, d'un développeur validant un code jusqu'à la mise à disposition du code de l'application pour les utilisateurs finaux. *Bac à sable*, *test*, *transfert*, *UAT* (test d'acceptation utilisateur), *préproduction*, *production* sont des noms types pour ces environnements.

Isoler les ressources sous-jacentes, mettre en oeuvre des règles de gouvernance et d'accès, protéger une charge de travail de production, valider les modifications avant de les appliquer à la production constituent certaines des raisons pour lesquelles il est préférable de créer ces environnements distincts. 

## Objectifs
{: #objectives}

* En savoir plus sur les modèles d'accès {{site.data.keyword.iamlong}} et Cloud Foundry
* Configurer un projet en séparant les rôles et les environnements 
* Configurer l'intégration continue 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Définition d'un projet

Soit l'exemple de projet avec les éléments suivants :
* plusieurs microservices déployés dans {{site.data.keyword.containershort_notm}},
* bases de données,
* compartiments de stockage de fichiers.

Dans ce projet, vous allez définir trois environnements :
* *Développement* - cet environnement est mis à jour en permanence à chaque validation, les tests d'unité, les tests de fumée sont exécutés. Il donne accès au dernier et au plus grand déploiement du projet. 
* *Test* - cet environnement est généré après une branche ou une balise stable du code. Les tests d'acceptation utilisateur sont effectués dans cet environnement. Sa configuration est similaire à l'environnement de production. Il contient des données réalistes (données de production anonymisées, par exemple). 
* *Production* - cet environnement est mis à jour avec la version validée dans l'environnement précédent.

**Un pipeline de distribution gère la progression d'une génération à travers l'environnement.** Il peut être entièrement automatisé ou inclure des guichets de validation manuels pour promouvoir les générations approuvées entre les environnements. Il s’agit d’une solution véritablement ouverte qui doit être configurée pour correspondre aux bonnes pratiques et flux de travail de l'entreprise. 

Pour prendre en charge l'exécution du pipeline de génération, vous introduisez un **utilisateur fonctionnel**, à savoir un utilisateur {{site.data.keyword.cloud_notm}} standard qui est membre de l'équipe sans véritable identité dans le monde physique. Cet utilisateur fonctionnel est propriétaire des pipelines de distribution et de toute autre ressource de cloud nécessitant une forte propriété. Cette approche est utile dans le cas où un membre de l'équipe quitte l'entreprise ou passe à un autre projet. L'utilisateur fonctionnel est dédié à votre projet et ne change pas pendant sa durée de vie. Vous devez ensuite créer une [clé d'API](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey) pour cet utilisateur fonctionnel. Vous sélectionnerez cette clé d'API lors de la configuration des pipelines DevOps ou de l'exécution de scripts d'automatisation pour emprunter l'identité de l'utilisateur fonctionnel. 

Pour l'affectation des responsabilités aux membres de l'équipe du projet, définissez les rôles et les autorisations associées suivants : 

|           | Développement | Test | Production |
| --------- | ----------- | ------- | ---------- |
| Développeur | <ul><li>contribue au code</li><li>peut accéder aux fichiers journaux</li><li>peut visualiser la configuration de l'application et du service </li><li>utilise les applications déployées </li></ul> | <ul><li>peut accéder aux fichiers journaux</li><li>peut visualiser la configuration de l'application et du service </li><li>utilise les applications déployées </li></ul> | <ul><li>aucun accès</li></ul> |
| Testeur    | <ul><li>utilise les applications déployées </li></ul> | <ul><li>utilise les applications déployées </li></ul> | <ul><li>aucun accès</li></ul> |
| Opérateur  | <ul><li>peut accéder aux fichiers journaux</li><li>peut visualiser/définir la configuration de l'application et du service </li></ul> | <ul><li>peut accéder aux fichiers journaux</li><li>peut visualiser/définir la configuration de l'application et du service </li></ul> | <ul><li>peut accéder aux fichiers journaux</li><li>peut visualiser/définir la configuration de l'application et du service </li></ul> |
| Utilisateur fonctionnel du pipeline | <ul><li>peut déployer/annuler le déploiement d'applications</li><li>peut visualiser/définir la configuration de l'application et du service </li></ul> | <ul><li>peut déployer/annuler le déploiement d'applications</li><li>peut visualiser/définir la configuration de l'application et du service </li></ul> | <ul><li>peut déployer/annuler le déploiement d'applications</li><li>peut visualiser/définir la configuration de l'application et du service </li></ul> |

## Identity and Access Management (IAM)
{: #first_objective}

{{site.data.keyword.iamshort}} (IAM) vous permet d'authentifier en toute sécurité les utilisateurs des services de plateforme et d'infrastructure, et de contrôler l'accès aux **ressources** de manière cohérente sur la plateforme {{site.data.keyword.cloud_notm}}. Un ensemble de services {{site.data.keyword.cloud_notm}} sont activés pour utiliser Cloud IAM afin de contrôler les accès. Ils sont organisés en **groupes de ressources** dans votre **compte** pour permettre d'accorder à des **utilisateurs** un accès rapide et facile à plusieurs ressources à la fois. Des **règles** d'accès Cloud IAM permettent d'affecter à des ID utilisateur et de service un accès aux ressources de votre compte.

Une **règle** affecte à un utilisateur ou à un ID de service un ou plusieurs **rôles** en utilisant une combinaison d'attributs pour définir la portée de l'accès. La règle peut fournir un accès à un seul service, jusqu'au niveau instance, ou s'appliquer à un ensemble de ressources organisées dans un groupe de ressources. En fonction des rôles utilisateur que vous affectez, différents niveaux d'accès sont accordés à l'ID utilisateur ou de service, lui permettant d'effectuer des tâches de gestion de plateforme ou d'accéder à un service via l'interface utilisateur, ou d'émettre des appels API de types spécifiques.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="Diagramme du modèle IAM" />
</p>

Actuellement, certains services du catalogue {{site.data.keyword.cloud_notm}} ne peuvent pas être gérés à l'aide d'IAM. Pour ces services, vous pouvez continuer à utiliser Cloud Foundry en fournissant aux utilisateurs un accès à l'organisation et à l'espace auxquels appartient l'instance avec un rôle Cloud Foundry attribué pour définir le niveau d'accès autorisé. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Diagramme du modèle Cloud Foundry" />
</p>

## Création des ressources pour un environnement 

Bien que les trois environnements requis par cet exemple de projet requièrent des droits d'accès différents et qu'il soit peut-être nécessaire de leur attribuer des capacités différentes, ils partagent un modèle d'architecture commun. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="Diagramme d'architecture illustrant un environnement" />
</p>

Commencez par créer l'environnement de développement. 

1. [Sélectionnez un emplacement {{site.data.keyword.cloud_notm}}](https://{DomainName}) où déployer l'environnement.
1. Pour les services et applications Cloud Foundry : 
   1. [Créez une organisation pour le projet](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg).
   1. [Créez un espace Cloud Foundry pour l'environnement](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo).
   1. Créez les services Cloud Foundry utilisés par le projet sous cet espace. 
1. [Créez un groupe de ressources pour l'environnement](https://{DomainName}/account/resource-groups).
1. Créez les services compatibles avec ce groupe de ressources, tels qu'{{site.data.keyword.cos_full_notm}}, {{site.data.keyword.la_full_notm}}, {{site.data.keyword.mon_full_notm}} et {{site.data.keyword.cloudant_short_notm}} dans ce groupe.
1. [Créez un cluster Kubernetes](https://{DomainName}/containers-kubernetes/catalog/cluster) dans {{site.data.keyword.containershort_notm}}, veillez à sélectionner le groupe de ressources créé ci-dessus.
1. Configurez {{site.data.keyword.la_full_notm}} et {{site.data.keyword.mon_full_notm}} pour envoyer des journaux et surveiller le cluster.

Le diagramme suivant montre où sont créées les ressources du projet sous le compte : 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="Diagramme illustrant les ressources du projet" />
</p>

## Affectation de rôles au sein de l'environnement

1. Invitez des utilisateurs au compte
1. Attribuez des règles aux utilisateurs pour définir les utilisateurs autorisés à accéder au groupe de ressources, aux services du groupe, à l'instance {{site.data.keyword.containershort_notm}} et à leurs droits. Reportez-vous à la [définition de règles d'accès](https://{DomainName}/docs/containers?topic=containers-users#access_policies) pour sélectionner les règles appropriées pour un utilisateur de l'environnement. Les utilisateurs dotés du même jeu de règles peuvent être placés dans le [même groupe d'accès](https://{DomainName}/docs/iam?topic=iam-groups#groups). Cela simplifie la gestion des utilisateurs car les règles sont attribuées au groupe d'accès et héritées par tous les utilisateurs du groupe. 
1. Configurez leurs rôles d’organisation et d’espace Cloud Foundry en fonction de leurs besoins dans l’environnement. Reportez-vous à la [définition de rôle](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess) pour attribuer les rôles appropriés en fonction de l'environnement. 

Reportez-vous à la documentation des services pour comprendre comment un service mappe les rôles IAM et Cloud Foundry sur des actions spécifiques. Consultez, par exemple, [comment le service {{site.data.keyword.mon_full_notm}} mappe les rôles IAM sur des actions](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam).

L'attribution des rôles appropriés aux utilisateurs nécessite plusieurs itérations et améliorations. Les droits octroyés peuvent être contrôlés au niveau du groupe de ressources, pour toutes les ressources d'un groupe ou bien détaillés jusqu'à une instance spécifique d'un service. Vous découvrirez au fil du temps les règles d'accès idéales pour votre projet. 

Une bonne pratique consiste à commencer avec le jeu de règles minimal, puis à le développer avec précaution au besoin. Pour Kubernetes, vous devez examiner le [contrôle d'accès basé sur un rôle (RBAC)](https://kubernetes.io/docs/admin/authorization/rbac/) pour configurer les autorisations dans le cluster. 

Pour l'environnement de développement, les responsabilités de l'utilisateur définies précédemment peuvent se traduire de la façon suivante : 

|           | Règles d'accès IAM | Cloud Foundry |
| --------- | ----------- | ------- |
| Développeur | <ul><li>Groupe de ressources : *Afficheur*</li><li>Rôles d'accès à la plateforme dans le groupe de ressources : *Afficheur*</li><li>Rôle de service de journalisation & surveillance : *Auteur*</li></ul> | <ul><li>Rôle d'organisation : *Auditeur*</li><li>Rôle d'espace : *Auditeur*</li></ul> |
| Testeur    | <ul><li>Aucune configuration nécessaire. Le testeur accède à l'application déployée, pas aux environnements de développement </li></ul> | <ul><li>Aucune configuration nécessaire </li></ul> |
| Opérateur  | <ul><li>Groupe de ressources : *Afficheur*</li><li>Rôles d'accès à la plateforme dans le groupe de ressources : *Opérateur*, *Afficheur*</li><li>Rôle de service de journalisation & surveillance : *Auteur*</li></ul> | <ul><li>Rôle d'organisation : *Auditeur*</li><li>Rôle d'espace : *Développeur*</li></ul> |
| Utilisateur fonctionnel du pipeline | <ul><li>Groupe de ressources : *Afficheur*</li><li>Rôles d'accès à la plateforme dans le groupe de ressources : *Editeur*, *Afficheur*</li></ul> | <ul><li>Rôle d'organisation : *Auditeur*</li><li>Rôle d'espace : *Développeur*</li></ul> |

Les règles d'accès IAM et les rôles Cloud Foundry sont définis dans l' [interface utilisateur Identify and Access Management](https://{DomainName}/iam/#/users) : 

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="Configuration des droits pour le rôle de développeur" />
</p>

## Réplication pour plusieurs environnements 

A ce stade, vous pouvez répliquer des étapes similaires pour générer les autres environnements. 

1. Créez un groupe de ressources par environnement. 
1. Créez un cluster et les instances de service requises par environnement. 
1. Créez un espace Cloud Foundry par environnement. 
1. Créez les instances de service requises dans chaque espace. 

<p style="text-align: center;">
  <img title="Utilisation de clusters séparés pour isoler les environnements" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="Diagramme illustrant les clusters séparés pour isoler les environnements" />
</p>

En combinant des outils tels que [l'interface de ligne de commande CLI {{site.data.keyword.cloud_notm}} `ibmcloud`](https://github.com/IBM-Cloud/ibm-cloud-developer-tools), [`terraform` de HashiCorp](https://www.terraform.io/), le fournisseur [{{site.data.keyword.cloud_notm}} pour Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm), l'interface de ligne de commande CLI Kubernetes `kubectl`, vous pouvez scripter et automatiser la création de ces environnements.

Les clusters Kubernetes distincts pour les environnements offrent de bonnes propriétés : 
* quel que soit l'environnement, tous les clusters ont tendance à se ressembler ;
* ils facilitent le contrôle des accès à un cluster spécifique ; 
* ils permettent d'assouplir les cycles de mise à jour pour les déploiements et les ressources sous-jacentes. Lorsqu'une nouvelle version de Kubernetes est diffusée, ils vous donnent la possibilité de mettre à jour le cluster de développement en premier lieu, de valider votre application, puis de mettre à jour l'autre environnement ;
* ils permettent de séparer les charges de travail différentes qui peuvent avoir un impact mutuel, par exemple, isoler l'environnement de production par rapport aux autres environnements. 

Une autre approche consiste à utiliser les [espaces de nom Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) avec des [quotas de ressources Kubernetes](https://kubernetes.io/docs/concepts/policy/resource-quotas/) pour isoler les environnements et contrôler la consommation de ressources. 

<p style="text-align: center;">
  <img title="Utilisation d'espaces de nom séparés pour isoler les environnement" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="Diagramme illustrant des espaces de nom séparés pour isoler les environnements" />
</p>

Dans la zone de saisie `Rechercher` de l'interface utilisateur LogDNA, utilisez la zone `espace de nom :` pour filtrer les journaux en fonction de l'espace de nom.
{: tip}

## Configuration du pipeline de distribution  

Dans le cas de déploiements dans les différents environnements, le pipeline d’intégration continue/de distribution continue peut être configuré pour piloter l’ensemble du processus : 
* mettez à jour en permanence l'environnement `Développement` avec le code le plus récent et le plus important de la branche `développement`, en exécutant des tests d'unité et des tests d'intégration sur le cluster dédié ; 
* favorisez les générations de développement dans l'environnement `Test`, soit automatiquement si tous les tests des étapes précédentes sont corrects, soit via un processus de promotion manuel. Certaines équipes utilisent également des branches différentes, en fusionnant l’état de développement en cours pour former une branche `stable`, par exemple ; 
* répétez un processus similaire pour passer à l'environnement `Production`. 

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="Pipeline CI/CD de la génération au déploiement" />
</p>

Lors de la configuration du pipeline DevOps, veillez à utiliser la clé d'API d'un utilisateur fonctionnel. Seul l'utilisateur fonctionnel doit avoir les droits requis pour déployer des applications sur vos clusters. 

Pendant la phase de génération, il est important de gérer correctement les versions des images Docker. Vous pouvez utiliser la révision de validation Git dans le cadre de la balise image ou un identificateur unique fourni par votre chaîne d'outils DevOps, à savoir tout identificateur facilitant la correspondance de l'image avec la génération et le code source réels contenus dans l'image. 

Au fur et à mesure que vous vous familiarisez avec Kubernetes, [Helm](https://helm.sh/), le gestionnaire de packages de Kubernetes, devient un outil pratique pour la gestion de versions, l’assemblage et le déploiement de votre application. [Cet exemple de chaîne d'outils DevOps](https://github.com/open-toolchain/simple-helm-toolchain) constitue un bon point de départ et est préconfiguré pour une distribution continue dans un cluster Kubernetes. Au fur et à mesure que votre projet se développe en plusieurs microservices, la [charte cadre Helm](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies) constitue une bonne solution pour composer votre application. 

## Extension du tutoriel 

Félicitations, vous pouvez à présent déployer votre application en toute sécurité, du développement à la production. Vous trouverez ci-dessous des suggestions supplémentaires pour améliorer la distribution des applications. 

* Ajoutez [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) au pipeline pour effectuer un contrôle qualité lors des déploiements. 
* Examinez les contributions de codage des membres de l'équipe et les interactions entre développeurs avec [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).
* Effectuez le tutoriel [Planification, création et mise à jour d'environnements de déploiement](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments) pour automatiser le déploiement de vos environnements.

## Informations connexes 

* [Initiation à {{site.data.keyword.iamshort}}](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [Meilleures pratiques pour l'organisation des ressources dans un groupe de ressources](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [Analyse des journaux et surveillance de la santé avec LogDNA et Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [Déploiement continu sur Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Chaîne d'outils Hello Helm](https://github.com/open-toolchain/simple-helm-toolchain)
* [Développement d'une application de microservices avec Kubernetes et Helm](https://github.com/open-toolchain/microservices-helm-toolchain)
* [Octroi de droits à un utilisateur pour afficher les journaux dans LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [Octroi de droits à un utilisateur pour afficher les métriques dans Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
