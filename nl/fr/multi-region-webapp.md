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

# Sécurisation d'une application Web dans plusieurs régions 
{: #multi-region-webapp}

Ce tutoriel vous guide dans la création, la sécurisation, le déploiement et l'équilibrage de la charge d'une application Cloud Foundry dans plusieurs régions à l'aide d'un pipeline [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery). 

Des pannes vont se produire sur vos applications ou certaines parties de vos applications - c'est un fait Il peut s'agir d'un problème dans votre code, d'une maintenance planifiée ayant un impact sur les ressources utilisées par votre application, d'une panne matérielle entraînant la panne d’une zone, d'un emplacement, d'un centre de données où votre application est hébergée. L'une de ces situations se produira et vous devez être préparé. Avec {{site.data.keyword.Bluemix_notm}}, vous pouvez déployer votre application sur [plusieurs emplacements](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg) pour augmenter la résilience de vos applications. Et avec votre application s'exécutant désormais dans plusieurs emplacements, vous pouvez également rediriger le trafic des utilisateurs vers l'emplacement le plus proche pour réduire le temps d'attente. 

## Objectifs

* Déployer une application Cloud Foundry sur plusieurs emplacements avec {{site.data.keyword.contdelivery_short}}.
* Mapper un domaine personnalisé sur l'application. 
* Configurer l'équilibrage de charge global pour votre application multiemplacement. 
* Lier un certificat SSL à votre application. 
* Surveiller les performances de l'application. 

## Services utilisés

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry App
* [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) for DevOps
* [Internet services](https://{DomainName}/catalog/services/internet-services)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture

Ce tutoriel implique un scénario actif/actif dans lequel deux copies de l'application sont déployées dans deux emplacements différents et les deux copies traitent les demandes des clients de manière alternée. La configuration DNS pointe automatiquement vers l'emplacement sain si une copie échoue. 

<p style="text-align: center;">

   ![Architecture](./images/solution1/Architecture.png)
</p>

## Création d'une application Node.js 
{: #create}

Commencez par créer une application de démarrage Node.js qui s'exécute dans un environnement Cloud Foundry. 

1. Cliquez sur **[Catalogue](https://{DomainName}/catalog/)** dans la console {{site.data.keyword.Bluemix_notm}}.
2. Cliquez sur **Applications Cloud Foundry** dans la catégorie **Plateforme** et sélectionnez **[{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)**.
     ![](images/solution1/SDKforNodejs.png)
3. Entrez un **nom unique** pour votre application, qui sera également votre nom d'hôte, par exemple : myusername-nodeapp. Ensuite, cliquez sur **Créer**.
4.  Une fois l'application démarrée, cliquez sur le lien **Visiter l'URL** de la page **Présentation** pour afficher votre application en direct sur un nouvel onglet.

![HelloWorld](images/solution1/HelloWorld.png)

C'est un bon début ! Vous disposez de votre propre application de démarrage Node.js s'exécutant dans {{site.data.keyword.Bluemix_notm}}. 

Vous allez ensuite déplacer le code source de votre application dans un référentiel et déployer vos modifications automatiquement. 

## Configuration du contrôle de source et de {{site.data.keyword.contdelivery_short}}
{: #devops}

Dans cette étape, vous configurez un référentiel de contrôle de source git pour stocker votre code, puis vous créez un pipeline afin de déployer automatiquement les modifications du code. 

1. Dans le volet gauche de l'application que vous venez de créer, sélectionnez **Présentation** et faites défiler l'écran pour localiser **{{site.data.keyword.contdelivery_short}}**. Cliquez sur **Activer**.

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. Conservez les options par défaut et cliquez sur **Créer**. Une **chaîne d'outils** doit avoir été créée par défaut. 

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. Sélectionnez la vignette **Git** sous **Code**. Vous êtes alors dirigé vers votre page de référentiel git. 
4. Si vous n'avez pas encore configuré les clés SSH, une barre de notification contenant des instructions devrait apparaître dans la partie supérieure. Effectuez les étapes en ouvrant le lien d'**ajout d'une clé SSH** dans un nouvel onglet ou si vous souhaitez utiliser HTTPS au lieu de SSH, suivez les étapes en cliquant sur **créer un jeton d'accès personnel**. N'oubliez pas de sauvegarder la clé ou le jeton pour référence ultérieure. 
5. Sélectionnez SSH ou HTTPS et copiez l'URL de git. Clonez la source sur votre ordinateur local. 
   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   **Remarque :** si vous êtes invité à entrer un nom d'utilisateur, indiquez votre nom d'utilisateur git. Pour le mot de passe, utilisez une **clé SSH** ou un **jeton d'accès personnel** ou celui créé que vous avez créé à l'étape précédente.
6. Ouvrez le référentiel cloné dans l'environnement IDE de votre choix et accédez à `public/index.html`. A présent, mettez le code à jour. Remplacez "Hello World" par un autre texte. 
7. Exécutez l’application en local en lançant les commandes l’une après l’autre `npm install`, `npm build`, `npm start` et consultez `localhost:<port_number>` dans votre navigateur.
  **<port_number>** tel qu'affiché sur la console.
8. Transférez les modifications dans votre référentiel en trois étapes simples : ajouter (add), valider (commit) et transférer (push). 
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
9. Accédez à la chaîne d'outils que vous avez créée précédemment et cliquez sur la vignette **Pipeline de distribution**.
10. Confirmez que vous voyez les étapes **BUILD** et **DEPLOY**.
  ![HelloWorld](images/solution1/DevOps_Pipeline.png)

11. Patientez jusqu'à la fin de l'étape **DEPLOY**.
12. Cliquez sur l'**url** de l'application sous Résultat de la dernière exécution pour afficher vos modifications en temps réel.

Continuez à apporter d'autres modifications à votre application et validez-les périodiquement dans votre référentiel git. Si l'application n'est pas mise à jour, consultez les journaux des étapes DEPLOY et BUILD du pipeline. 

## Déploiement dans un autre emplacement
{: #deploy_another_region}

Vous allez ensuite déployer la même application dans un autre emplacement {{site.data.keyword.Bluemix_notm}}. Vous pouvez utiliser la même chaîne d’outils, mais vous allez ajouter une autre étape DEPLOY pour gérer le déploiement de l’application dans un autre emplacement. 

1. Accédez à la **Présentation** de l'application et faites défiler l'écran jusqu'à l'option **Afficher la chaîne d'outils**.
2. Sélectionnez **Delivery Pipeline** dans Distribuer.
3. Cliquez sur l'**icône représentant un engrenage** à l'étape **DEPLOY** et sélectionnez **Cloner l'étape**.
   ![HelloWorld](images/solution1/CloneStage.png)
4. Renommez l'étape en "Deploy to UK" et sélectionnez **TRAVAUX**.
5. Remplacez **Région IBM Cloud** par **London - https://api.eu-gb.bluemix.net**. Créez un **espace** si vous n'en avez pas.
6. Remplacez la valeur de **Script de déploiement** par `cf push "${CF_APP}" -d eu-gb.mybluemix.net`

   ![HelloWorld](images/solution1/DeployToUK.png)
7. Cliquez sur **Sauvegarder** et exécutez la nouvelle étape en cliquant sur le **bouton Lire**.

## Enregistrement d'un domaine personnalisé avec IBM Cloud Internet Services  

{: #domain_cis}

IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) est une plateforme uniforme permettant de configurer et de gérer le système de noms de domaine (Domain Name System -DNS), l'équilibrage de la charge globale (Global Load Balancing - GLB), le pare-feu pour applications Web (Web Application Firewall - WAF) et la protection contre le déni de service distribué (Distributed Denial of Service - DDoS) pour les applications Web. Elle fournit un service Internet rapide, hautement performant, fiable et sécurisé aux clients qui exploitent leur entreprise sur IBM Cloud avec trois fonctionnalités principales pour améliorer votre flux de travail : sécurité, fiabilité et performances.   

Lors du déploiement d'une application réelle, vous souhaiterez probablement utiliser votre propre domaine au lieu du domaine mybluemix.net fourni par IBM. Dans cette étape, après avoir créé un domaine personnalisé, vous pouvez utiliser les serveurs DNS fournis par IBM Cloud Internet Services. 

1. Achetez un domaine auprès d'un registraire tel que [http://godaddy.com](http://godaddy.com).
2. Accédez à [Internet Services](https://{DomainName}/catalog/services/internet-services) dans le catalogue {{site.data.keyword.Bluemix_notm}}.
2. Entrez un nom de service, puis cliquez sur **Créer** pour créer une instance du service. 
3. Lorsque l'instance de service est mise à disposition, définissez votre nom de domaine et cliquez sur **Ajouter un domaine**. 
4. Lorsque les serveurs de noms sont attribués, configurez votre registraire ou votre fournisseur de nom de domaine pour utiliser les serveurs de noms répertoriés. 
5. Une fois que vous avez configuré votre registraire ou le fournisseur DNS, les modifications peuvent nécessiter jusqu'à 24 heures pour prendre effet. Lorsque le statut du domaine sur la page Présentation passe de *En attente* à *Actif*, vous pouvez utiliser la commande `dig <your_domain_name> ns` pour vérifier que les serveurs de noms IBM Cloud ont pris effet.
   {:tip}

## Ajout d'un équilibrage de charge global à l'application 

{: #add_glb}

Dans cette section, vous utilisez le GLB (Global Load Balancer) dans IBM Cloud Internet Services pour gérer le trafic sur plusieurs sites. Le GLB utilise un pool d’origines qui permet de répartir le trafic sur plusieurs origines. 

### Avant de créer un GLB, créez un contrôle de santé pour le GLB. 

1. Dans l'application Cloud Internet Services, accédez à **Fiabilité** > **Equilibreur de charge global** et, au bas de la page, cliquez sur **Créer un contrôle de santé**.
2. Entrez le chemin que vous souhaitez surveiller, par exemple `/`, et sélectionnez un type (HTTP ou HTTPS). En règle générale, vous pouvez créer un noeud final dédié à la santé. Cliquez sur **Mettre à disposition une instance**.
   ![Contrôle de santé](images/solution1/health_check.png)

### Création d'un groupe d’origines contenant deux origines 

1. Cliquez sur **Créer un pool**.
2. Entrez un nom pour le pool, sélectionnez le contrôle de santé que vous venez de créer et une région proche de l'emplacement de votre application node.js. 
3. Entrez un nom pour la première origine et le nom d'hôte de l'application située à Dallas `<your_app>.mybluemix.net`.
4. De même, ajoutez une autre origine avec l'adresse d'origine pointant vers l'application située à Londres `<your_app>.eu-gb.mybluemix.net`.
5. Cliquez sur **Mettre à disposition une instance**.
   ![Pool d'origines](images/solution1/origin_pool.png)

### Création d'un équilibreur de charge global (GLB) 

1. Cliquez sur **Créer un équilibreur de charge**.
2. Entrez un nom pour l’équilibreur de charge global. Ce nom fait également partie de l'URL de votre application universelle (`http://<glb_name>.<your_domain_name>`), quel que soit l'emplacement.
3. Cliquez sur **Ajouter un pool** et sélectionnez le pool d'origines que vous venez de créer.
4. Cliquez sur **Mettre à disposition une instance**.
   ![Equilibreur de charge global](images/solution1/load_balancer.png)

À ce stade, le GLB est configuré, mais les applications Cloud Foundry ne sont pas encore prêtes à répondre aux demandes provenant du nom de domaine du GLB configuré. Pour terminer la configuration, vous devez mettre à jour les applications avec les routes à l'aide du domaine personnalisé. 

## Configuration du domaine personnalisé et des routes vers votre application 

{: #add_domain}

Au cours de cette étape, vous mappez le nom de domaine personnalisé sur le noeud final sécurisé de l'emplacement {{site.data.keyword.Bluemix_notm}} où votre application est en cours d'exécution. 

1. Dans la barre de menus, cliquez sur **Gérer**, puis sur **Compte** : [Compte](https://{DomainName}/account).
2. Sur la page du compte, accédez à l'application **Organisations Cloud Foundry**, et sélectionnez **Domaines** dans la colonne Actions.
3. Cliquez sur **Ajouter un domaine** et entrez votre nom de domaine personnalisé obtenu auprès du registraire.
4. Sélectionnez l'emplacement correct et cliquez sur **Sauvegarder**.
5. De même, ajoutez le nom de domaine personnalisé à Londres. 
6. Revenez à la [Liste de ressources](https://{DomainName}/resources) {{site.data.keyword.Bluemix_notm}}, accédez à **Applications Cloud Foundry**, cliquez sur l'application située à Dallas, cliquez sur **Route** > **Editer les routes**, et cliquez sur **Ajouter une route**.
   ![Ajouter une route](images/solution1/ApplicationRoutes.png)
7. Entrez le nom d'hôte du GLB que vous avez configuré précédemment dans la zone **Entrez un hôte (facultatif)**, puis sélectionnez le domaine personnalisé que vous venez d'ajouter. Cliquez sur **Sauvegarder**.
8. De même, configurez le domaine et les routes pour l'application à Londres. 

A ce stade, vous pouvez consulter votre application sur l'URL `<glb_name>.<your_domain_name>` et l'équilibreur de charge global répartit automatiquement le trafic pour vos applications multiemplacements. Vous pouvez le vérifier en arrêtant votre application à Dallas, en la maintenant active à Londres et en accédant à l'application via l'équilibreur de charge global. 

Dans la mesure où vous avez configuré la distribution continue aux étapes précédentes, même si l'opération fonctionne à ce moment-là, la configuration peut être remplacée lorsqu'une autre génération est déclenchée. Pour rendre ces modifications permanentes, revenez dans les chaînes d'outils et modifiez le fichier *manifest.yml* :

1. Dans la [Liste de ressources](https://{DomainName}/resources) {{site.data.keyword.Bluemix_notm}}, accédez à **Applications Cloud Foundry**, cliquez sur l'application située à Dallas, accédez à la **Présentation** de l'application et faites défiler l'écran jusqu'à **Afficher la chaîne d'outils**.
2. Sélectionnez la vignette Git sous Code.
3. Sélectionnez *manifest.yml*.
4. Cliquez sur **Editer** et ajoutez des routes personnalisées. Remplacez les configurations de domaine et d'hôte d'origine par `Routes` uniquement.

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <glb_name>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. Validez les modifications et assurez-vous que les générations pour les deux emplacements ont réussi.   

## Alternative : mappage du domaine personnalisé sur le domaine du système IBM Cloud. 

Il est possible que vous ne souhaitiez pas utiliser un équilibreur de charge global face à vos applications multiemplacements, mais que vous deviez mapper le nom de domaine personnalisé sur le noeud final sécurisé de l'emplacement {{site.data.keyword.Bluemix_notm}} où est exécutée votre application. 

Avec l'application Cloud Intenet Services, procédez comme suit pour configurer les enregistrements `CNAME` pour votre application : 

1. Dans l'application Cloud Internet Services, accédez à **Fiabilité** > **DNS**.
2. Sélectionnez **CNAME** dans la liste déroulante **Type**, entrez un alias pour votre application dans la zone Nom et l'URL de l'application dans la zone du nom de domaine. L’application `<your_app>.mybluemix.net` à Dallas peut être mappée à un CNAME `<your_app>`.
3. Cliquez sur **Ajouter Enregistrement**. Basculez le commutateur PROXY sur ON pour améliorer la sécurité de votre application. 
4. De même, définissez l'enregistrement `CNAME` pour le noeud final Londres.
   ![Enregistrements CNAME](images/solution1/cnames.png)

Si vous utilisez un autre domaine par défaut que `mybluemix.net`, tel que `cf.appdomain.cloud` ou `cf.cloud.ibm.com`, veillez à utiliser le [domaine système correspondant](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain).
{:tip}

Si vous utilisez un autre fournisseur DNS, la procédure de configuration de l'enregistrement CNAME varie en fonction de votre fournisseur DNS. Par exemple, si vous utilisez GoDaddy, vous suivez la procédure de la page
[Domaines Aide](https://www.godaddy.com/help/add-a-cname-record-19236) de GoDaddy.

Pour que vos applications Cloud Foundry soient accessibles via le domaine personnalisé, vous devez ajouter ce domaine à la [liste des domaines de l'organisation Cloud Foundry où les applications sont déployées](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#updatingapps). Après quoi, vous pouvez ajouter les routes aux manifestes de l'application : 

   ```
   applications:
   - path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <your_app>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}

## Liaison du certificat SSL à votre application 
{: #ssl}

1. Obtenez un certificat SSL. Par exemple, vous pouvez en acheter un sur le site https://www.godaddy.com/web-security/ssl-certificate ou en générer un gratuitement sur le site https://letsencrypt.org/.
2. Accédez à **Présentation** > **Routes** > **Gestion des domaines** pour l'application.
3. Cliquez sur le bouton de téléchargement du certificat SSL et envoyez le certificat. 
5. Accédez à votre application via https au lieu de http. 

## Surveillance des performances de l'application
{: #monitor}

Vérifiez la santé de votre application multiemplacement. 

1. Dans le tableau de bord de l'application, sélectionnez **Surveillance**.
2. Cliquez sur **Afficher tous les tests**
   ![](images/solution1/alert_frequency.png)

Availability Monitoring exécute des tests synthétiques à partir de sites répartis dans le monde entier, 24 heures sur 24, pour détecter et résoudre de manière proactive les problèmes de performances avant que les utilisateurs ne soient affectés. Si vous avez configuré une route personnalisée pour votre application, modifiez la définition de test pour accéder à votre application via son domaine personnalisé. 

## Suppression de ressources 

* Supprimez la chaîne d'outils 
* Supprimez les deux applications Cloud Foundry déployées dans les deux emplacements 
* Supprimez le GLB, les pools d'origines et le contrôle de santé 
* Supprimez la configuration DNS 
* Supprimez l'instance Internet Services

## Contenu associé

[Ajout d'une base de données Cloudant](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[Applications Cloud Foundry Auto-Scaling](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
