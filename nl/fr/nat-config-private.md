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

# Configuration d'une conversion NAT pour l'accès Internet à partir d'un réseau privé
{: #nat-config-private}

Dans le monde actuel des applications et des services informatiques basés sur le Web, peu d’applications existent isolément. Les développeurs exigent désormais l'accès aux services sur Internet, qu’il s’agisse de code applicatif open source et de mises à jour, ou de services tiers fournissant des fonctionnalités d’application via des API REST. Le masquage NAT est une approche couramment utilisée pour sécuriser l'accès au service hébergé sur Internet à partir de réseaux privés. Dans le masquage NAT, les adresses IP privées sont converties en adresse IP de l'interface publique sortante dans une relation plusieurs-à-un, protégeant ainsi l'adresse IP privée de l'accès public.   

Ce tutoriel présente la configuration du masquage NAT sur un dispositif VRA (Virtual Router Appliance) afin de connecter un sous-réseau sécurisé sur le réseau privé {{site.data.keyword.Bluemix_notm}}. Il s'appuie sur le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure), et ajoute une configuration SNAT (Source NAT), où l'adresse source est masquée et où les règles de pare-feu sont utilisées pour sécuriser le trafic sortant. Vous trouverez des configurations NAT plus complexes dans la [documentation VRA supplémentaire]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Objectifs
{: #objectives}

-	Configurer la conversion SNAT sur un dispositif Virtual Router Appliance (VRA) 
-	Configurer les règles de pare-feu pour l'accès Internet 

## Services utilisés
{: #products}

Ce tutoriel utilise les services {{site.data.keyword.Bluemix_notm}} suivants :  

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Ce tutoriel peut entraîner des coûts. Le VRA est disponible uniquement moyennant un forfait mensuel. 

## Architecture
{: #architecture}

<p style="text-align: center;">

  ![Architecture](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	Documenter les services Internet requis. 
2.	Configurer la conversion NAT. 
3.	Créer une zone et des règles de pare-feu Internet. 

## Avant de commencer
{: #prereqs}

Ce tutoriel active les hôtes de l'enceinte du réseau privé sécurisé créés pendant le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) pour accéder aux services Internet publics. Ce tutoriel doit être effectué en premier lieu.  

## Documentation des services Internet
{: #Document_outbound}

La première étape consiste à identifier les services accessibles sur l'Internet public et à documenter les ports qui doivent être activés pour le trafic sortant et le trafic entrant correspondants provenant d'Internet. Cette liste de ports est requise pour les règles de pare-feu dans une étape ultérieure.  

Dans cet exemple, seuls les ports http et https sont activés, car ils couvrent la majorité des exigences. Les services DNS et NTP sont fournis à partir du réseau privé {{site.data.keyword.Bluemix_notm}}. Si ces services et d'autres tels que SMTP (port 25) ou MySQL (port 3306) sont requis, des règles de pare-feu supplémentaires sont nécessaires. Les deux règles de port de base sont les suivantes : 

-	Port 80 (http)
-	Port 443 (https)

Vérifiez si le service tiers prend en charge la liste blanche des adresses source. Dans l'affirmative, l'adresse IP publique du dispositif VRA est nécessaire pour configurer le service tiers afin de limiter l'accès au service.  


## Masquage NAT vers Internet  
{: #NAT_Masquerade}

Effectuez les instructions ci-dessous pour configurer l'accès Internet externe pour les hôtes de la zone APP à l'aide du masquage NAT.  

1.	Connectez-vous à l'aide de SSH au dispositif VRA et passez en mode \[edit\] (config).
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2.	Créez les règles de conversion SNAT sur le VRA, en spécifiant le même service `<Subnet Gateway IP>/<CIDR>` que celui déterminé pour le sous-réseau/VLAN de la zone APP dans le tutoriel précédent de mise à disposition du VRA. 
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Subnet Gateway IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## Création de pare-feux
{: #Create_firewalls}

1.	Créez des règles de pare-feu 
   ```
   set security firewall name APP-TO-OUTSIDE default-action drop
   set security firewall name APP-TO-OUTSIDE description 'APP traffic to the Internet'
   set security firewall name APP-TO-OUTSIDE default-log

   set security firewall name APP-TO-OUTSIDE rule 90 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 90 action accept
   set security firewall name APP-TO-OUTSIDE rule 90 destination port 80

   set security firewall name APP-TO-OUTSIDE rule 100 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 100 action accept
   set security firewall name APP-TO-OUTSIDE rule 100 destination port 443

   set security firewall name APP-TO-OUTSIDE rule 200 protocol icmp
   set security firewall name APP-TO-OUTSIDE rule 200 icmp type 8
   set security firewall name APP-TO-OUTSIDE rule 200 action accept
   commit
   ```
   {: codeblock}

## Création d'une zone et application de règles
{: #Create_zone}

1.	Créez une zone OUTSIDE pour contrôler l’accès à Internet externe. 
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	Attribuez des pare-feux pour contrôler le trafic depuis et vers Internet. 
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE 
   commit
   save
   ```
   {: codeblock}
3.	Vérifiez que l'instance VSI dans la zone APP peut maintenant accéder aux services sur Internet. Connectez-vous à l'instance VSI locale à l’aide de SSH, utilisez les commandes ping et curl pour valider l’accès icmp et tcp aux sites Internet.   
   ```bash
   ssh root@<VSI Private IP>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## Suppression de ressources 
{:removeresources}
Etapes à suivre pour supprimer les ressources créées dans ce tutoriel.  

Le VRA est disponible moyennant un forfait mensuel. L'annulation ne donne pas lieu à un remboursement. Il est suggéré d’annuler uniquement si ce VRA n'est plus nécessaire le mois suivant. Si un cluster à haute disponibilité VRA double est requis, ce VRA unique peut être mis à niveau sur la page [Détails de la passerelle](https://{DomainName}/classic/network/gatewayappliances).
{:tip}  

1. Annulez tous les serveurs virtuels ou les serveurs bare metal
2. Annulez le VRA 
3. Annulez tout VLAN supplémentaire à l'aide d'un ticket de demande de service.  

## Documentation associée
{:related}

-	[Conversion NAT du VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[Masquage NAT]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[Documentation supplémentaire sur le VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).

