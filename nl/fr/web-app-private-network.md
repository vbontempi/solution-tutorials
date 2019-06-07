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

# Service d'application Web à partir d'un réseau privé sécurisé
{: #web-app-private-network}

L'hébergement d'applications Web est un modèle de déploiement commun pour le cloud public, dans lequel les ressources peuvent être dimensionnées à la demande pour répondre aux demandes d'utilisation à court et à long terme. La sécurité des charges de travail des applications est une condition préalable fondamentale qui vient compléter les qualités de résilience et l'évolutivité offertes par le cloud public.  

Ce tutoriel vous guide dans la création d'une application Web évolutive et sécurisée, hébergée dans un réseau privé et sécurisé à l'aide d'un dispositif Virtual Router Appliance (VRA), de VLAN, de la conversion NAT et de pare-feux. L'application comprend un équilibreur de charge, deux serveurs d'applications Web et un serveur de base de données MySQL. Cette section associe trois tutoriels pour illustrer la manière dont les applications Web peuvent être déployées en toute sécurité sur la plateforme IaaS {{site.data.keyword.Bluemix_notm}} à l'aide de la mise en réseau classique.
{:shortdesc}

## Objectifs
{: #objectives}

- Créer des serveurs virtuels pour installer PHP et MySQL 
- Mettre en service un équilibreur de charge pour répartir les demandes sur les serveurs d'applications 
- Déployer un dispositif Virtual Router Appliance (VRA) pour créer un réseau sécurisé
- Définir les VLAN et les sous-réseaux IP  
- Sécuriser le réseau avec des règles de pare-feu 
- SNAT pour le déploiement d’applications 

## Services utilisés
{: #products}

Ce tutoriel utilise les services {{site.data.keyword.Bluemix_notm}} suivants :  

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [Load Balancer]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

Ce tutoriel peut entraîner des coûts. Le VRA est disponible uniquement moyennant un forfait mensuel. 

## Architecture
{: #architecture}

<p style="text-align: center;">

  ![Architecture](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	Configurez un réseau privé sécurisé 
2.	Configurez l'accès NAT pour le déploiement d'applications 
3.	Déployez une application Web évolutive et un équilibreur de charge 

## Avant de commencer
{: #prereqs}

Ce tutoriel utilise trois tutoriels existants, qui sont déployés dans l'ordre. Passez en revue ces trois tutoriels avant de commencer :

-	[Isolation de charges de travail à l'aide d'un réseau privé sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[Configuration d'une conversion NAT pour l'accès Internet à partir d'un réseau sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[Utilisation de serveurs virtuels pour créer des applications Web évolutives à haute disponibilité ]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## Configuration du réseau privé sécurisé
{: #private_network}

Les environnements de réseau privé isolés et sécurisés sont au coeur du modèle de sécurité d'applications IaaS sur un cloud public. Les pare-feux, les réseaux locaux virtuels (VLAN), le routage et les réseaux privés (VPN) sont tous des composants nécessaires à la création d'environnements privés isolés. La première étape consiste à créer l'enceinte du réseau privé sécurisé dans laquelle l'application Web est déployée.   

- [Isolation de charges de travail à l'aide d'un réseau privé sécurisé](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

Ce tutoriel peut être effectué sans modification. Dans une étape ultérieure, trois machines virtuelles sont déployées dans la zone APP en tant que serveurs Web Nginx et base de données MySQL.  

## Configuration de la conversion NAT pour le déploiement d'applications sécurisées 
{: #nat_config}

L'installation d'applications à code source ouvert nécessite un accès sécurisé à Internet pour accéder aux référentiels source. Pour empêcher les serveurs du réseau privé sécurisé d'être exposés sur Internet public, le NAT source est utilisé lorsque l'adresse source est masquée et les règles de pare-feu sont utilisées pour sécuriser les demandes de référentiel d'applications sortantes. Toutes les demandes entrantes sont refusées.  

- [Configuration d'une conversion NAT pour l'accès Internet à partir d'un réseau sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

Ce tutoriel peut être effectué sans modification. Dans la prochaine étape, la configuration NAT est utilisée pour accéder aux modules Nginx et MySQL requis.   


## Déploiement d'une application Web évolutive et d'un équilibreur de charge 
{: #scalable_app}

Une installation Wordpress sur Nginx et MySQL avec un équilibreur de charge est utilisée pour illustrer la manière dont une application Web évolutive et résiliente peut être déployée dans un réseau privé sécurisé.  

Ce tutoriel vous guide tout au long de ce scénario pour la création d'un équilibreur de charge {{site.data.keyword.Bluemix_notm}}, de deux serveurs d'applications Web et d'un serveur de base de données MySQL. Les serveurs sont déployés dans la zone APP du réseau privé sécurisé afin de séparer les autres charges de travail du réseau public à l'aide du pare-feu.  

- [Utilisation de serveurs virtuels pour créer des applications Web évolutives à haute disponibilité ]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

Ce tutoriel comporte trois modifications : 

1.	Les serveurs virtuels utilisés dans ce tutoriel sont déployés sur le VLAN et le sous-réseau protégés par la zone de pare-feu APP située derrière le dispositif VRA. 
2. Spécifiez l'&lt;ID VLAN privé&gt; lors de l'organisation des serveurs virtuels. Pour plus d'informations sur la spécification de l'&lt;ID VLAN privé&gt; lors de la commande d'un serveur virtuel, reportez-vous aux instructions [Commande du premier serveur virtuel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver) du tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). Pensez également à sélectionner votre clé SSH téléchargée plus tôt dans le tutoriel pour permettre l'accès aux serveurs virtuels.  
3. Il est vivement recommandé de **NE PAS** utiliser le service de stockage de fichiers pour ce tutoriel en raison des performances médiocres de resynchronisation lors de la copie des fichiers Wordpress sur un stockage partagé. Cela n'affecte pas le tutoriel général. Les étapes relatives à la création du stockage de fichiers et à la configuration des montages peuvent être ignorées pour les serveurs d'applications et la base de données. Dans le cas contraire, toutes les étapes d'[Installation et configuration de l'application PHP sur les serveurs d'applications](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application) doivent être effectuées sur les deux serveurs d'applications. Avant de suivre les étapes décrites dans [Installation et configuration de l'application PHP sur les serveurs d'applications](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application), créez d'abord le répertoire `/mnt/www/` sur les deux serveurs d'applications. Ce répertoire a été créé initialement dans la section de stockage de fichiers désormais supprimée.  

   ```sh
   mkdir /mnt/www
   ```

Au terme de cette étape, l’équilibreur de charge doit être sain et le site Wordpress accessible sur Internet. Les serveurs virtuels constituant l'application Web sont protégés des accès externes sur Internet par le pare-feu VRA et le seul accès possible est via l'équilibreur de charge. Pour un environnement de production, la protection contre les attaques DDoS et le pare-feu pour applications Web (WAF) doivent également être pris en compte, tels que fournis par [{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services). 


## Suppression de ressources 
{: #removeresources}

Etapes à suivre pour supprimer les ressources créées dans ce tutoriel.  

Le VRA est disponible moyennant un forfait mensuel. L'annulation ne donne pas lieu à un remboursement. Il est suggéré d’annuler uniquement si ce VRA n'est plus nécessaire le mois suivant.
{:tip}  

1. Annulez tous les serveurs virtuels ou les serveurs bare metal
2. Annulez le VRA 
3. Annulez tout VLAN supplémentaire à l'aide d'un ticket de demande de service. 
4. Supprimez l'équilibreur de charge 
5. Supprimez les services de stockage de fichiers 

## Extension du tutoriel  

1. Dans ce tutoriel, seuls deux serveurs virtuels sont initialement mis à disposition en tant que niveau d'application, d'autres serveurs peuvent être ajoutés automatiquement pour gérer une charge supplémentaire. La [Mise à l'échelle auto]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group) vous permet d'automatiser le processus de redimensionnement manuel associé à l'ajout ou la suppression de serveurs virtuels pour prendre en charge vos applications métier. 

2. Protégez séparément les données utilisateur en ajoutant un deuxième réseau local virtuel privé et un sous-réseau IP au VRA afin de créer une zone DATA pour héberger le serveur de base de données MySQL. Configurez les règles de pare-feu pour autoriser uniquement le trafic IP MySQL sur le port 3306 entrant à partir de la zone APP vers la zone DATA.  

