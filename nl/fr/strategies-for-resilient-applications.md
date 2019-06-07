---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-11"
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

# Stratégies pour les applications résilientes 
{: #strategies-for-resilient-applications}

Kubernetes, Cloud Foundry, Cloud Functions ou Virtual Servers, quelle que soit l'option de calcul choisie, les entreprises cherchent à minimiser les temps d'indisponibilité et à créer des architectures résilientes offrant une disponibilité maximale. Ce tutoriel met en lumière les capacités d'IBM Cloud dans le développement de solutions résilientes et répond ainsi aux questions suivantes. 

- Que faut-il prendre en compte lors de la préparation d'une solution disponible au niveau global ? 
- Comment les options de calcul disponibles aident-elles à déployer des applications dans plusieurs régions ? 
- Comment importer des artefacts d'application ou de service dans des régions supplémentaires ? 
- Comment les bases de données peuvent-elles être répliquées sur plusieurs sites ? 
- Quels services de sauvegarde doivent être utilisés : stockage en bloc, stockage de fichiers, stockage d'objets, bases de données ? 
- Existe-t-il des considérations spécifiques au service ? 

## Objectifs
{: #objectives}

* Découvrir les concepts architecturaux impliqués lors de la création d'applications résilientes. 
* Comprendre comment ces concepts sont mis en correspondance avec les offres de services et de calcul d'IBM Cloud. 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture et concepts

{: #architecture}

Pour concevoir une architecture résiliente, vous devez prendre en compte les blocs individuels de votre solution et leurs fonctionnalités spécifiques.  

Vous trouverez ci-dessous une architecture multirégion illustrant les différents composants pouvant exister dans une configuration multirégion. ![Architecture](images/solution39/Architecture.png)

Le diagramme d'architecture ci-dessus peut être différent selon l'option de calcul. Des diagrammes d'architecture spécifiques se trouvent sous chaque option de calcul dans les sections suivantes.  

### Reprise après incident avec deux régions  

Pour faciliter la reprise après incident, deux architectures largement acceptées sont utilisées : **actif/actif** et **actif/passif**. Chaque architecture a ses propres coûts et avantages liés au temps et aux efforts nécessaires à la reprise. 

#### Configuration actif/actif 

Dans une architecture actif/actif, les deux emplacements ont des instances actives identiques avec un équilibreur de charge qui répartit le trafic entre elles. En utilisant cette approche, la réplication de données doit être mise en oeuvre pour synchroniser les données en temps réel entre les deux régions. 

![Actif/Actif](images/solution39/Active-active.png)

Cette configuration offre une disponibilité plus élevée avec une correction manuelle moindre qu'une architecture actif/passif. Les demandes sont traitées à partir des deux centres de données. Vous devez configurer les services de périphérie (équilibreur de charge) avec un délai d'expiration et une logique de nouvelle tentative appropriés pour acheminer automatiquement la demande au deuxième centre de données en cas de défaillance de l'environnement du premier centre de données. 

Lors de la prise en compte d'un **objectif de point de reprise** (RPO) dans le scénario actif/actif, la synchronisation des données entre les deux centres de données actifs doit être extrêmement rapide pour permettre un flux de demandes fluide. 

#### Configuration actif/passif 

Une architecture actif/passif repose sur une région active et une deuxième région (passive) utilisée en tant que sauvegarde. En cas de panne dans la région active, la région passive devient active. Une intervention manuelle peut être nécessaire pour garantir que les bases de données ou le stockage de fichiers correspondent aux besoins de l'application et des utilisateurs.  

![Actif/Actif](images/solution39/Active-passive.png)

Les demandes sont diffusées à partir du site actif. En cas de panne ou de défaillance d'une application, un travail préalable à l'application est effectué pour préparer le centre de données de secours à répondre à la demande. Le passage du centre de données actif au centre de données passif est une opération fastidieuse. L'**objectif de temps de reprise** (RTO) et l'**objectif de point de reprise** (RPO) sont tous deux plus élevés que la configuration actif/actif. 

### Reprise après incident avec trois régions 

A l'ère des services connectés en permanence avec tolérance zéro en matière d'indisponibilité, les clients exigent que chaque service métier soit accessible 24 heures sur 24, partout dans le monde. Une stratégie rentable pour les entreprises implique de concevoir leur infrastructure en vue d'une disponibilité continue plutôt que de mettre en place des infrastructures de reprise après incident. 

L'utilisation de trois centres de données offre une résilience et une disponibilité supérieures à deux centres. Elle peut également offrir de meilleures performances en répartissant la charge de manière plus uniforme dans les centres de données. Si l'entreprise ne dispose que de deux centres de données, une variante consiste à déployer deux applications dans un centre de données et à déployer la troisième application dans le second centre de données. Vous pouvez également déployer les couches logique métier et présentation dans la topologie à 3 centres actifs et déployer la couche données dans la topologie à 2 centres actifs. 

#### Configuration actif/actif/actif (3 centres actifs) 

![](images/solution39/Active-active-active.png)

Les demandes sont traitées par l'application qui s'exécute dans l'un des trois centres de données actifs. Une étude de cas sur le site Web IBM.com indique qu'une configuration à 3 centres actifs ne nécessite que 50 % de la capacité de calcul, de la mémoire et du réseau par cluster, alors qu'une configuration à 2 centres actifs nécessite 100 % de la capacité par cluster. La couche données est l'endroit où la différence de coût se démarque. Pour plus de détails, voir [*Always On: Assess, Design, Implement, and Manage Continuous Availability*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf). 

#### Configuration actif-actif-passif

![](images/solution39/Active-active-passive.png)

Dans ce scénario, lorsqu'une des deux applications actives des centres de données principal et secondaire subit une panne, l'application de secours du troisième centre de données est activée. La procédure de reprise après incident décrite dans le scénario des deux centres de données est suivie pour rétablir le traitement normal des demandes des clients. L'application de secours du troisième centre de données peut être configurée en mode de secours à chaud ou à froid (hot ou cold standby). 

Reportez-vous à [ce guide](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/) pour plus d'informations sur la reprise après incident.

### Architectures multirégions 

Dans une architecture multirégion, une application est déployée à différents emplacements où chaque région exécute une copie identique de l'application.  

Une région est un emplacement géographique spécifique dans lequel vous pouvez déployer des applications, des services et d'autres ressources {{site.data.keyword.cloud_notm}}. [Les régions {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/containers?topic=containers-regions-and-zones) comprennent une ou plusieurs zones, qui sont des centres de données physiques hébergeant les ressources de calcul, de réseau et de stockage, ainsi que les dispositifs de refroidissement et d'alimentation associés, qui hébergent des services et des applications. Les zones sont isolées les unes des autres pour éviter le partage d'un point de défaillance unique.

De plus, dans une architecture multirégion, un équilibreur de charge global, tel que [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services), est requis pour répartir le trafic entre les régions. 

Le déploiement d'une solution sur plusieurs régions présente les avantages suivants : 
- Accélération des temps de réponse pour les utilisateurs finaux : la rapidité est essentielle, plus l'origine de votre serveur est proche des utilisateurs finaux, meilleure et plus rapide est l'expérience des utilisateurs. 
- Reprise après incident - lorsque la région active est en échec, vous disposez d'une région de secours que vous pouvez restaurer rapidement. 
- Besoins métier : dans certains cas, vous devez stocker des données dans des régions distinctes, séparées par plusieurs centaines de kilomètres. Par conséquent, dans ce cas, vous devez stocker des données dans plusieurs régions.  

### Zones multiples au sein d'architectures multirégions 

La génération d'applications régionales multizones signifie que votre application doit être déployée sur les zones d'une région. Vous pouvez également avoir deux ou trois régions.  

Avec une architecture multizone, vous avez besoin d'un équilibreur de charge local pour répartir le trafic localement entre les zones d'une région, puis, si une deuxième région est configurée, un équilibreur de charge global répartit alors le trafic entre les régions.  

Vous pouvez en apprendre plus sur les régions et les zones [ici](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones).

## Options de calcul  

Cette section présente les options de calcul disponibles dans {{site.data.keyword.cloud_notm}}. Pour chaque option de calcul, un diagramme d'architecture est fourni avec un tutoriel sur la manière de déployer cette architecture. 

Remarque : certaines architectures d'options de calcul ne comportent pas de base de données ou de service. Elles se concentrent uniquement sur le déploiement d'une application dans deux régions pour l'option de calcul sélectionnée. Une fois que vous avez déployé l'un des exemples d'option de calcul multirégion, l'étape logique suivante consiste à ajouter des bases de données et d'autres services. Les sections suivantes de ce tutoriel de solution traitent des [bases de données](#databaseservices) et des [services autres que de base de données](#nondatabaseservices). 

### Cloud Foundry 

Cloud Foundry offre la possibilité de déployer une architecture multirégion. De plus, l'utilisation de services de pipeline de [distribution continue](https://{DomainName}/catalog/services/continuous-delivery) vous permet de déployer votre application sur plusieurs régions. L'architecture multirégion de Cloud Foundry se présente comme suit : 

![Architecture CF](images/solution39/CF2-Architecture.png)

La même application est déployée dans plusieurs régions et un équilibreur de charge global achemine le trafic vers la région la plus proche et la plus saine. Le tutoriel [**Déploiement d'une application Web sécurisée sur plusieurs régions**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp) vous guide dans le déploiement d'une architecture similaire. 

### {{site.data.keyword.cfee_full_notm}}

**{{site.data.keyword.cfee_full_notm}} (CFEE)** offre les mêmes fonctionnalités que Cloud Foundry public ainsi que des fonctionnalités supplémentaires.

**{{site.data.keyword.cfee_full_notm}}** vous permet d'instancier plusieurs plateformes Cloud Foundry de niveau entreprise isolées à la demande. Les instances de CFEE s'exécutent dans votre propre compte dans [{{site.data.keyword.cloud_notm}}
](http://ibm.com/cloud). L'environnement est déployé sur un matériel isolé au-dessus d'[{{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service). Vous exercez un contrôle total sur l'environnement, y compris le contrôle d'accès, la gestion de la capacité, la gestion des changements, la surveillance et les services.

Une architecture multirégion utilisant {{site.data.keyword.cfee_full_notm}} est présentée ci-dessous.

![Architecture](images/solution39/CFEE-Architecture.png)

Le déploiement de cette architecture nécessite les éléments suivants :  

- Configuration de deux instances CFEE - une dans chaque région. 
- Création et liaison des services sur le compte CFEE.  
- Transfert des applications ciblant le noeud final de l'API CFEE.  
- Configuration de la réplication de base de données, exactement comme vous le feriez sur Cloud Foundry public.  

En outre, consultez le guide pas à pas [Deploy Logistics Wizard to Cloud Foundry Enterprise Environment (CFEE)](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md). Il traite du déploiement d'une application basée sur un microservice dans CFEE. Une fois l'application déployée sur une instance CFEE, vous pouvez répliquer la procédure dans une deuxième région et connecter [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) devant les deux instances CFEE pour équilibrer le trafic.  

Reportez-vous à la [documentation d'{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about) pour plus de détails.

### Kubernetes

Avec Kubernetes, vous pouvez réaliser une architecture multizone au sein de régions, ce qui peut être un cas d'utilisation actif/actif. Lorsque vous implémentez une solution avec {{site.data.keyword.containershort_notm}}, vous bénéficiez de fonctionnalités intégrées, telles que l'équilibrage de la charge et l'isolement, d'une résilience accrue après de potentielles défaillances des hôtes, des réseaux ou des applications. Si vous créez plusieurs clusters, en cas de panne d'un cluster, les utilisateurs peuvent toujours accéder à une application également déployée dans un autre cluster. Avec plusieurs clusters dans différentes régions, les utilisateurs peuvent également accéder au cluster le plus proche avec un temps de réponse réseau réduit. Pour une résilience accrue, vous avez également la possibilité de sélectionner les clusters multizones, ce qui signifie que les noeuds sont déployés sur plusieurs zones d'une région.  

L'architecture multirégion de Kubernetes se présente comme suit. 

![Kubernetes](images/solution39/Kub-Architecture.png)

1. Le développeur génère des images Docker pour l'application. 
2. Les images sont transférées vers {{site.data.keyword.registryshort_notm}} à deux emplacements différents.
3. L'application est déployée sur les clusters Kubernetes aux deux emplacements. 
4. Les utilisateurs finaux accèdent à l'application. 
5. Cloud Internet Services est configuré pour intercepter les demandes adressées à l'application et pour répartir la charge sur les clusters. En outre, DDoS Protection et Web Application Firewall sont activés pour protéger l’application des menaces courantes. Les actifs, tels que les images et les fichiers CSS, sont éventuellement mis en cache. 

Le tutoriel [**Clusters Kubernetes multirégions résilients et sécurisés avec Cloud Internet Services**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis) vous explique la procédure à suivre pour déployer ce type d'architecture. 

### {{site.data.keyword.openwhisk_short}}

{{site.data.keyword.openwhisk_short}} est disponible dans plusieurs emplacements {{site.data.keyword.cloud_notm}}. Pour augmenter la résilience et réduire le temps de réponse du réseau, les applications peuvent déployer leur serveur de back-end dans plusieurs emplacements. Ensuite, avec IBM Cloud Internet Services (CIS), les développeurs peuvent exposer un point d’entrée unique, chargé de la répartition du trafic vers le serveur de back-end sain le plus proche. L'architecture multirégion de {{site.data.keyword.openwhisk_short}} se présente comme suit.

 ![Architecture des fonctions](images/solution39/Functions-Architecture.png)

1. Les utilisateurs accèdent à l'application. La demande passe par Internet Services. 
2.  Internet Services redirige les utilisateurs vers le serveur de back-end d'API sain le plus proche. 
3. Certificate Manager fournit à l'API son certificat SSL. Le trafic est chiffré de bout en bout. 
4. L'API est implémentée avec Cloud Functions. 

Découvrez comment déployer cette architecture en suivant le tutoriel [**Déploiement d'applications sans serveur dans plusieurs régions**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless). 

### {{site.data.keyword.baremetal_short}} et {{site.data.keyword.virtualmachinesshort}}

{{site.data.keyword.virtualmachinesshort}} et {{site.data.keyword.baremetal_short}} offrent la possibilité de réaliser une architecture multirégion. Vous pouvez mettre à disposition des serveurs sur de nombreux emplacements disponibles sur {{site.data.keyword.cloud_notm}}. 

![emplacements de serveurs](images/solution39/ServersLocation.png)

Lors de la préparation de ce type d'architecture à l'aide de {{site.data.keyword.virtualmachinesshort}} et {{site.data.keyword.baremetal_short}}, prenez en compte les éléments suivants : stockage de fichiers, sauvegardes, restauration et bases de données, sélection d'une base de données en tant que service ou installation d'une base de données sur un serveur virtuel.  

L'architecture ci-dessous illustre le déploiement d'une architecture multirégion à l'aide de {{site.data.keyword.virtualmachinesshort}} dans une architecture actif/passif où une région est active et la deuxième région est passive.  

![Architecture VM](images/solution39/vm-Architecture2.png)

Les composants requis pour ce type d'architecture sont les suivants :  

1. Les utilisateurs accèdent à l'application via IBM Cloud Internet Services (CIS). 
2. CIS achemine le trafic vers l'emplacement actif. 
3. Dans un emplacement, un équilibreur de charge redirige le trafic vers un serveur. 
4. Des bases de données sont déployées sur un serveur virtuel, ce qui signifie que vous devez configurer la base de données et configurer les réplications et les sauvegardes entre les régions. L’alternative consiste à utiliser une base de données en tant que service, un sujet traité plus loin dans le présent tutoriel. 
5. Le stockage de fichiers permet d'archiver les images et les fichiers de l'application. Le stockage de fichiers offre la possibilité de prendre un instantané à une heure et une date données. Cet instantané peut ensuite être réutilisé dans une autre région, ce que vous feriez manuellement.  

Le tutoriel [**Utilisation de serveurs virtuels pour créer des applications Web évolutives à haute disponibilité**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application) permet d'implémenter cette architecture.

## Bases de données et fichiers d'application 
{: #databaseservices}

{{site.data.keyword.cloud_notm}} propose une sélection [de bases de données en tant que service](https://{DomainName}/catalog/?category=databases) comprenant des bases de données relationnelles et non relationnelles, en fonction des besoins de votre entreprise. La [base de données en tant que service (DBaaS)](https://www.ibm.com/cloud/learn/what-is-cloud-database) présente de nombreux avantages. A l'aide d'un DBaaS tel que {{site.data.keyword.cloudant}}, vous pouvez tirer parti de la prise en charge multirégion afin d'effectuer une réplication en direct entre deux services de base de données dans différentes régions, d'effectuer des sauvegardes, de bénéficier d'une mise à l'échelle et d'une disponibilité maximale.  

**Fonctions clés :** 

- Service de base de données construit et accessible via une plateforme cloud 
- Possibilité pour les utilisateurs de l'entreprise d'héberger des bases de données sans avoir à acheter de matériel dédié 
- Possibilité de gestion par l'utilisateur ou proposé en tant que service et géré par un fournisseur 
- Possibilité de prise en charge des bases de données SQL ou NoSQL 
- Accès via une interface Web ou une API fournie par le fournisseur 

**Préparation à l'architecture multirégion : **

- Quelles sont les options de résilience du service de base de données ? 
- Comment la réplication est-elle gérée entre plusieurs services de base de données dans différentes régions ? 
- Comment les données sont-elles sauvegardées ? 
- Quelles sont les approches de reprise après incident pour chaque type de données ? 

### {{site.data.keyword.cloudant}}

{{site.data.keyword.cloudant}} est une base de données répartie, optimisée pour gérer des charges de travail lourdes, typiques des applications Web et mobiles, volumineuses et en forte croissance. Disponible en tant que service {{site.data.keyword.Bluemix_notm}} entièrement géré, basé sur un contrat de niveau de service, {{site.data.keyword.cloudant}} adapte le débit et le stockage avec une grande flexibilité et en toute autonomie. {{site.data.keyword.cloudant}} est également disponible en tant qu’installation téléchargeable sur site. Son API et son puissant protocole de réplication sont compatibles avec un écosystème open source comprenant CouchDB, PouchDB et des bibliothèques pour les piles de développement Web et mobiles les plus utilisées.

{{site.data.keyword.cloudant}} prend en charge la [réplication](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation) entre plusieurs instances sur plusieurs emplacements. Toute modification survenue dans la base de données source est reproduite dans la base de données cible. Vous pouvez créer des réplications entre un nombre quelconque de bases de données, de manière continue ou ponctuelle. Le diagramme suivant illustre une configuration type utilisant deux instances {{site.data.keyword.cloudant}}, une dans chaque région : 

![actif-actif](images/solution39/Active-active.png)

Reportez-vous à [ces instructions](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery) pour configurer la réplication entre les instances {{site.data.keyword.cloudant}}. Le service fournit également des instructions et des outils pour la [sauvegarde et la restauration des données](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery). 

### {{site.data.keyword.Db2_on_Cloud_short}}, {{site.data.keyword.dashdbshort_notm}} et {{site.data.keyword.Db2Hosted_notm}}

{{site.data.keyword.cloud_notm}} propose plusieurs [services de base de données Db2](https://{DomainName}/catalog/?search=db2h). Il s'agit de :

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2) : base de données SQL dans le cloud, entièrement gérée pour des charges de travail opérationnelles courantes, de type OLTP.
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse) : service d'entreposage de données dans le cloud, entièrement géré pour des charges de travail d'analyses haute performance, à l'échelle du pétaoctet. Il offre des forfaits de service SMP et MPP et utilise un magasin de données en colonnes optimisé et un traitement en mémoire. 
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted) : service hébergé par IBM et géré par le système de base de données utilisateur. Il fournit à Db2 un accès administratif complet sur une infrastructure cloud, éliminant ainsi les coûts, la complexité et les risques liés à la gestion de sa propre infrastructure. 

Les sections suivantes traitent plus particulièrement de {{site.data.keyword.Db2_on_Cloud_short}} en tant que DBaaS pour les charges de travail opérationnelles. Ces charges de travail sont courantes pour les applications décrites dans le présent tutoriel. 

#### Prise en charge multirégion pour {{site.data.keyword.Db2_on_Cloud_short}}

{{site.data.keyword.Db2_on_Cloud_short}} propose plusieurs [options pour atteindre la Haute disponibilité et reprise après incident (HADR)](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview). Vous pouvez choisir l'option Haute disponibilité lorsque vous créez un service. Par la suite, vous pouvez [ajouter un noeud de reprise après incident (Disaster Recovery - DR) avec réplication géographique](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha) via le tableau de bord de l'instance. L'option de noeud DR externe vous permet de synchroniser vos données en temps réel avec un noeud de base de données dans un centre de données {{site.data.keyword.cloud_notm}} hors site de votre choix. 

Vous trouverez plus d'informations dans la [documentation sur la haute disponibilité](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha). 

#### Sauvegarde et restauration

{{site.data.keyword.Db2_on_Cloud_short}} inclut des sauvegardes quotidiennes dans ses forfaits payants. En règle générale, les sauvegardes sont stockées à l'aide d'{{site.data.keyword.cos_short}}, utilisant ainsi trois centres de données pour une disponibilité accrue des données conservées. Les sauvegardes sont conservées 14 jours. Vous pouvez les utiliser pour effectuer une récupération à un point de cohérence. La [documentation sur la sauvegarde et la restauration](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br) explique comment restaurer des données à la date et à l’heure souhaitées. 

### {{site.data.keyword.databases-for}}

{{site.data.keyword.databases-for}} offre plusieurs systèmes de base de données open source sous forme de services entièrement gérés. En voici la liste :  
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

Tous ces services partagent les mêmes caractéristiques :    
* Pour la haute disponibilité, ils sont déployés en clusters. Les détails peuvent être trouvés dans la documentation de chaque service : 
  - [PostgreSQL](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* Chaque cluster est réparti sur plusieurs zones. 
* Les données sont répliquées dans les zones 
* Les utilisateurs peuvent augmenter les ressources de stockage et de mémoire d'une instance. Pour plus de détails, reportez-vous à la [documentation sur la mise à l’échelle pour {{site.data.keyword.databases-for-redis}}, par exemple](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings).
* Les sauvegardes sont effectuées quotidiennement ou à la demande. Les détails sont documentés pour chaque service. Vous trouverez [ici la documentation sur la sauvegarde de {{site.data.keyword.databases-for-postgresql}}, par exemple](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups).
* Les données inactives, les sauvegardes et le trafic réseau sont chiffrés.  
* Chaque [service peut être géré à l’aide du plugin de la CLI d'{{site.data.keyword.databases-for}}](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in)


### {{site.data.keyword.cos_full_notm}}

{{site.data.keyword.cos_full_notm}} (COS) fournit un stockage dans le cloud pérenne, sécurisé et économique. Les informations stockées avec {{site.data.keyword.cos_full_notm}} sont chiffrées et dispersées sur plusieurs emplacements géographiques. Lors de la création de compartiments de stockage au sein d'une instance COS, vous décidez à quel emplacement le compartiment doit être créé et quelle option de résilience doit être utilisée. 

Il existe trois types de résilience de compartiment : 
   - La résilience **interrégionale** répartit vos données dans plusieurs zones métropolitaines. Elle peut être considérée comme une option multirégion. Lors de l'accès au contenu stocké dans un compartiment interrégional, COS propose un noeud final spécial capable de récupérer le contenu d'une région saine. 
   - La résilience **régionale** répartit vos données dans une seule zone métropolitaine.Elle peut être considérée comme une configuration multizone dans une région. 
   - La résilience de **centre de données unique** répartit les données sur plusieurs dispositifs au sein d'un même centre de données. 

Reportez-vous à [cette documentation](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints) pour obtenir une explication détaillée des options de résilience d'{{site.data.keyword.cos_full_notm}}.

### {{site.data.keyword.filestorage_full_notm}}

{{site.data.keyword.filestorage_full_notm}} est un stockage de fichiers basé sur NFS, connecté au réseau, pérenne, rapide et flexible. Cet environnement NAS vous permet d'avoir un contrôle total des fonctions et des performances de vos partages de fichiers. Les partages {{site.data.keyword.filestorage_short}} peuvent être connectés à un maximum de 64 unités autorisées via des connexions TCP/IP routées pour la résilience.

Certaines des fonctionnalités de stockage de fichiers sont les _instantanés_, la _réplication_, l'_accès simultané_. Reportez-vous à [la documentation](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage) pour obtenir une liste complète des fonctionnalités.

Une fois connecté à vos serveurs, un service {{site.data.keyword.filestorage_short}} peut être aisément utilisé pour stocker des sauvegardes de données, des fichiers d'application tels que des images et des vidéos. Ces images et fichiers peuvent ensuite être utilisés sur différents serveurs de la même région. 

Lors de l'ajout d'une deuxième région, vous pouvez utiliser la fonctionnalité d'instantanés de {{site.data.keyword.filestorage_short}} pour prendre un instantané automatiquement ou manuellement, puis le réutiliser dans la deuxième région passive.  

La réplication peut être programmée pour copier automatiquement des instantanés sur un volume de destination dans un centre de données distant. Les copies peuvent être récupérées sur le site distant en cas de données endommagées ou de catastrophe. Vous trouverez plus d'informations sur les instantanés de stockage de fichiers [ici](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots).

## Services autres que de base de données
{: #nondatabaseservices}

{{site.data.keyword.cloud_notm}} propose une sélection de [services](https://{DomainName}/catalog) autres que de base de données, qu'il s'agisse de services IBM ou de services tiers. Lors de la planification d'une architecture multirégion, vous devez comprendre comment des services tels que Watson peuvent fonctionner dans une configuration multirégion. 

### {{site.data.keyword.conversationfull}}

[{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about) est une plateforme qui permet aux développeurs et aux utilisateurs non techniques de collaborer à la création d'assistants conversationnels basés sur l'IA.

Un assistant est un bot cognitif que vous pouvez personnaliser en fonction des besoins de votre entreprise et que vous pouvez déployer sur plusieurs canaux pour apporter de l'aide à vos clients où et quand ils en ont besoin. L'assistant comprend une ou plusieurs compétences. Une compétence de dialogue contient les données d'apprentissage et la logique permettant à un assistant d'aider vos clients.

Il est important de noter que {{site.data.keyword.conversationshort}} V1 est sans état. {{site.data.keyword.conversationshort}} offre une disponibilité à 99,5 %, mais vous souhaiterez peut-être disposer de plusieurs instances de ce service pour des applications hautement disponibles dans plusieurs régions. 

Une fois que vous avez créé des instances dans plusieurs emplacements, utilisez les outils {{site.data.keyword.conversationshort}} pour exporter, à partir d'une instance, un espace de travail existant, y compris les intentions, les entités et le dialogue. Importez ensuite cet espace de travail dans d'autres emplacements. 

## Récapitulatif

| Offre | Options de résilience |
| -------- | ------------------ |
| Cloud Foundry | <ul><li>Déployer des applications sur plusieurs emplacements </li><li>Répondre aux demandes émanant de plusieurs emplacements avec Cloud Internet Services </li><li>Utiliser les API Cloud Foundry pour configurer des organisations, des espaces et des applications de diffusion (push) vers plusieurs emplacements. </li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>Déployer des applications sur plusieurs emplacements </li><li>Répondre aux demandes émanant de plusieurs emplacements avec Cloud Internet Services </li><li>Utiliser les API Cloud Foundry pour configurer des organisations, des espaces et des applications de diffusion (push) vers plusieurs emplacements. </li><li>Construit sur le service Kubernetes </li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>Résilience par conception avec prise en charge des clusters multizones </li><li>Répondre aux demandes émanant de clusters dispersés dans plusieurs emplacements avec Cloud Internet Services </li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>Disponible dans plusieurs emplacements</li><li>Répondre aux demandes émanant de plusieurs emplacements avec Cloud Internet Services </li><li>Utiliser l'API Cloud Functions pour déployer des actions dans plusieurs emplacements </li></ul> |
| {{site.data.keyword.baremetal_short}} et {{site.data.keyword.virtualmachinesshort}} | <ul><li>Mise à disposition de serveurs dans plusieurs emplacements </li><li>Connecter des serveurs se trouvant au même emplacement à un équilibreur de charge local </li><li>Répondre aux demandes émanant de plusieurs emplacements avec Cloud Internet Services </li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>Réplication unique et continue entre les bases de données </li><li>Redondance automatique des données dans une région </li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>Mise à disposition d'un noeud de reprise après incident géo-répliqué pour la synchronisation des données en temps réel </li><li>Sauvegarde quotidienne avec forfaits payants </li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>Construit sur des clusters Kubernetes multizones </li><li>Répliques de lecture interrégionales. </li><li>Sauvegardes quotidiennes et à la demande </li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>Résilience de centre de données unique, régionale et interrégionale </li><li>Utiliser l'API pour synchroniser le contenu dans des compartiments de stockage </li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>Utiliser des instantanés pour capturer automatiquement le contenu et le diffuser vers une destination dans un centre de données distant </li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>Utiliser l'API Watson pour exporter et importer une spécification d'espace de travail entre plusieurs instances sur plusieurs emplacements </li></ul> |

## Contenu associé

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [Amélioration de la disponibilité des applications avec des clusters multizones](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry - Sécurisation d'une application Web dans plusieurs régions](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [Cloud Functions - Déploiement d'applications sans serveur dans plusieurs régions](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes - Clusters Kubernetes multirégions résilients et sécurisés avec Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [Virtual Servers - Utilisation de serveurs virtuels pour créer des applications Web évolutives à haute disponibilité](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
