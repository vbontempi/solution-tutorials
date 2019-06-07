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

# Liaison de réseaux privés sécurisés sur le réseau IBM
{: #vlan-spanning}

Le besoin croissant d'atteindre une portée mondiale et de garantir un fonctionnement 24/7 des applications Web s'accompagne de la nécessité de pouvoir héberger des services dans plusieurs centres de données du cloud. Les centres de données répartis sur plusieurs sites offrent des capacités de résilience en cas de défaillance géographique et rapprochent les charges de travail des utilisateurs dispersés dans le monde, ce qui réduit les temps d'attente et augmente les performances perçues. Le [réseau {{site.data.keyword.Bluemix_notm}}](https://www.ibm.com/cloud/data-centers/) permet aux utilisateurs de lier des charges de travail hébergées dans des réseaux privés sécurisés entre centres de données et sites.

Ce tutoriel présente la configuration d'une connexion IP routée de manière privée sur le réseau privé {{site.data.keyword.Bluemix_notm}} entre deux réseaux privés sécurisés hébergés dans deux centres de données différents. Toutes les ressources appartiennent à un compte {{site.data.keyword.Bluemix_notm}} unique. Ce tutoriel utilise le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) pour déployer deux réseaux privés liés de manière sécurisée sur le réseau privé {{site.data.keyword.Bluemix_notm}} à l'aide du service [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning). {:shortdesc}

## Objectifs
{: #objectives}

- Lier des réseaux sécurisés dans un compte {{site.data.keyword.Bluemix_notm}} IaaS
- Configurer des règles de pare-feu pour l'accès de site à site  
- Configurer le routage entre les sites 

## Services utilisés
{: #products}

Ce tutoriel utilise les services {{site.data.keyword.Bluemix_notm}} suivants :  
* [Dispositif Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

Ce tutoriel peut entraîner des coûts. Le VRA est disponible uniquement moyennant un forfait mensuel. 

## Architecture
{: #architecture}

<p style="text-align: center;">

  ![Architecture](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. Déployez des réseaux privés sécurisés 
2. Configurez VLAN Spanning
3. Configurez le routage IP entre réseaux privés 
4. Configurez les règles de pare-feu pour l'accès au site distant 

## Avant de commencer
{: #prereqs}

Ce tutoriel est basé sur le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). Passez en revue ce tutoriel et ses prérequis avant de commencer.  

## Configuration de sites du réseau privé sécurisé
{: #private_network}

Le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) est utilisé deux fois pour implémenter des réseaux privés dans deux centres de données différents. Il n’existe aucune restriction quant à l’utilisation de deux centres de données, excepté le fait d'observer l’impact du temps d'attente sur le trafic ou les charges de travail qui communiquent entre les sites.  

Vous pouvez effectuer le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) sans modification pour chaque centre de données sélectionné, en enregistrant les informations ci-dessous pour les étapes ultérieures. 

| Elément  | Centre de données1 | Centre de données2 |
|:------ |:--- | :--- |
| Centre de données |  |  |
| Adresse IP publique du VRA | <Adresse IP publique du VRA du CD1> | <Adresse IP publique du VRA du CD2> |
| Adresse IP privée du VRA | <Adresse IP privée du VRA du CD1> | <Adresse IP privée du VRA du CD2> |
| Sous-réseau privé & CIDR du VRA |  |  |
| ID VLAN privé | &lt;ID VLAN privé du CD1&gt;  | &lt;ID VLAN privé du CD2&gt; |
| Adresse IP privée de la VSI | <Adresse IP privée de la VSI du CD1> | <Adresse IP privée de la VSI du CD2> |
| Sous-réseau & CIDR de la zone APP | <Sous-réseau/CIDR de la zone APP du CD1> | <Sous-réseau/CIDR de la zone APP du CD2> |

1. Accédez à la page Détails de la passerelle pour chaque VRA via la page [Tous les dispositifs de passerelle](https://{DomainName}/classic/network/gatewayappliances).  
2. Recherchez la section VLANs de la Passerelle et cliquez sur le [VLAN]( https://{DomainName}/classic/network/vlans) de la passerelle sur le réseau **privé** pour afficher les détails du VLAN. Le nom doit contenir l'id, `bcrxxx`, signifiant "backend customer router" et être au format `nnnxx.bcrxxx.xxxx`.
3. Le dispositif VRA mis à disposition apparaît dans la section **Unités*. Dans la section **Sous-réseaux**, notez l'adresse IP du sous-réseau privé VRA et le CIDR (/26). Le sous-réseau est de type principal avec 64 adresses IP. Ces détails sont nécessaires ultérieurement pour la configuration du routage.  
4. Toujours sur la page Détails de la passerelle, localisez la section **VLAN associés** et cliquez sur le [VLAN]( https://{DomainName}/classic/network/vlans) du réseau **Privé** que vous avez associé pour créer le réseau sécurisé et la zone APP.  
5. L'instance VSI mise à disposition apparaît dans la section **Unités*. Dans la section **Sous-réseaux**, notez l'adresse IP du sous-réseau privé VSI et le CIDR (/26), car ces informations sont nécessaires à la configuration du routage. Ce VLAN et ce sous-réseau sont identifiés en tant que zone APP dans les deux configurations de pare-feu du VRA et sont enregistrés en tant que &lt;Sous-réseau/CIDR de la zone APP&gt;. 


## Configuration de VLAN Spanning 
{: #configure-vlan-spanning}

Par défaut, les serveurs (et les dispositifs VRA) situés sur différents VLAN et centres de données ne peuvent pas communiquer entre eux via le réseau privé. Dans ces tutoriels, au sein d'un centre de données unique, les dispositifs VRA permettent de lier des VLAN et des sous-réseaux avec un routage IP classique et des pare-feux afin de créer un réseau privé pour garantir la communication entre serveurs sur des VLAN. Bien qu'ils puissent communiquer dans le même centre de données, dans cette configuration, les serveurs appartenant au même compte {{site.data.keyword.Bluemix_notm}} ne peuvent pas communiquer d'un centre de données à un autre.  

Le service [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning) lève cette restriction de communication entre les VLAN et les sous-réseaux qui **NE sont PAS ** associés aux VRA. Il convient de noter que, même lorsque le service VLAN Spanning est activé, les VLAN associés aux VRA peuvent uniquement communiquer via leur VRA associé, tel que défini par la configuration des pare-feux et du routage du VRA. Lorsque le service VLAN Spanning est activé, les VRA appartenant à un compte {{site.data.keyword.Bluemix_notm}} sont connectés via le réseau privé et peuvent communiquer.  

Les VLAN non associés aux réseaux privés sécurisés créés par les VRA sont étendus, ce qui permet l’interconnexion de ces VLAN non associés entre les centres de données. Sont inclus les VLAN de passerelle VRA (de transit) appartenant au même compte IBM Cloud dans des centres de données différents. Les dispositifs VRA peuvent ainsi communiquer entre les centres de données lorsque le service VLAN Spanning est activé. Avec la connectivité VRA à VRA, la configuration des pare-feux et du routage du VRA permet aux serveurs des réseaux sécurisés de se connecter.  

Activez le service VLAN Spanning :

1. Accédez à la page [VLAN]( https://{DomainName}/classic/network/vlans).
2. Sélectionnez l'onglet **Span** en haut de la page
3. Sélectionnez le bouton d'option "On" du service VLAN Spanning. Plusieurs minutes sont nécessaires pour appliquer le changement de réseau. 
4. Confirmez que les deux VRA peuvent maintenant communiquer : 

   Connectez-vous au VRA du centre de données 1 et envoyez une commande ping au VRA du centre de données 2 

   ```
   SSH vyatta@<Adresse IP privée du VRA du CD1>
   ping <Adresse IP privée du VRA du CD2>
   ```
   {: codeblock}

   Connectez-vous au VRA du centre de données 2 et envoyez une commande ping au VRA du centre de données 1 
   ```
   SSH vyatta@<Adresse IP privée du VRA du CD2s>
   ping <Adresse IP privée du VRA du CD1>
   ```
   {: codeblock}

## Configuration du routage IP du VRA  
{: #vra_routing}

Créez le routage VRA dans chaque centre de données pour permettre aux instances VSI des zones APP des deux centres de données de communiquer.  

1. Créez une route statique dans le centre de données 1 vers le sous-réseau privé de la zone APP dans le centre de données 2, en mode édition VRA. 
   ```
   ssh vyatta@<Adresse IP privée du VRA du CD1>
   conf
   set protocols static route <Sous-réseau/CIDR de la zone APP du CD2>  next-hop <Adresse IP privée du VRA du CD2>
   commit
   ```
   {: codeblock}
2. Créez une route statique dans le centre de données 2 vers le sous-réseau privé de la zone APP dans le centre de données 1, en mode édition VRA. 
   ```
   ssh vyatta@<Adresse IP privée du VRA du CD2>
   conf
   set protocols static route <Sous-réseau/CIDR de la zone APP du CD1>  next-hop <Adresse IP privée du VRA du CD1>
   commit
   ```
   {: codeblock}
2. Passez en revue la table de routage du VRA à partir de la ligne de commande du VRA. A ce stade, les instances VSI ne peuvent pas communiquer car il n'existe aucune règle de pare-feu de zone APP pour autoriser le trafic entre les deux sous-réseaux de zone APP. Des règles de pare-feu sont requises pour le trafic déclenché à chaque extrémité. 
   ```
   show ip route
   ```
   {: codeblock}

La nouvelle route permettant à la zone APP de communiquer via le réseau privé IBM est désormais visible.  

## Configuration du pare-feu du VRA 
{: #vra_firewall}

Les règles de pare-feu de zone APP existantes sont uniquement configurées pour autoriser le trafic depuis et vers ce sous-réseau vers les services {{site.data.keyword.Bluemix_notm}} sur le réseau privé {{site.data.keyword.Bluemix_notm}} et pour un accès au réseau Internet public via la conversion NAT. Les autres sous-réseaux associés aux instances VSI sur ce VRA ou dans d'autres centres de données sont bloqués. L'étape suivante consiste à mettre à jour le groupe de ressources `ibmprivate` associé à la règle de pare-feu APP-TO-INSIDE afin de permettre un accès explicite au sous-réseau de l'autre centre de données.  

1. En mode de commande d'édition VRA du centre de données 1, ajoutez <Sous-réseau de la zone APP du CD2>/CIDR au groupe de ressources `ibmprivate`
   ```
   set resources group address-group ibmprivate address <Sous-réseau/CIDR de la zone APP du CD2>
   commit
   save
   ```
   {: codeblock}
2. En mode de commande d'édition VRA du centre de données 2, ajoutez <Sous-réseau>/CIDR de la zone APP du CD1 au groupe de ressources `ibmprivate`
   ```
   set resources group address-group ibmprivate address <Sous-réseau/CIDR de la zone APP du CD1>
   commit
   save
   ```
   {: codeblock}
3. Vérifiez que les VSI des deux centres de données peuvent maintenant communiquer 
   ```bash
   ping <IP de passerelle de sous-réseau distant>
   ssh root@<IP privée de VSI>
   ping <IP de passerelle de sous-réseau distant>
   ```
   {: codeblock}

   Si les VSI ne peuvent pas communiquer, effectuez les instructions du tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) pour surveiller le trafic sur les interfaces et consulter les journaux du pare-feu.  

## Suppression de ressources 
{: #removeresources}

Etapes à suivre pour supprimer les ressources créées dans ce tutoriel.  

Le VRA est disponible moyennant un forfait mensuel. L'annulation ne donne pas lieu à un remboursement. Il est suggéré d’annuler uniquement si ce VRA n'est plus nécessaire le mois suivant. Si un cluster à haute disponibilité VRA double est requis, ce VRA unique peut être mis à niveau sur la page [Détails de la passerelle](https://{DomainName}/classic/network/gatewayappliances).
{:tip}

1. Annulez tous les serveurs virtuels ou les serveurs bare metal
2. Annulez le VRA 
3. Annulez tout VLAN supplémentaire à l'aide d'un ticket de demande de service.  


## Extension du tutoriel 

Ce tutoriel peut être utilisé avec le tutoriel [Configuration d'un réseau privé virtuel (VPN) dans un réseau privé sécurisé](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network) pour lier les deux réseaux sécurisés à un réseau distant d'utilisateurs via un VPN IPSec. Des liens de VPN peuvent être établis vers les deux réseaux sécurisés pour une meilleure résilience de l'accès à la plateforme IaaS {{site.data.keyword.Bluemix_notm}}. Notez qu'IBM n'autorise pas le routage du trafic utilisateur entre des centres de données clients via le réseau privé IBM. La configuration du routage pour éviter les boucles réseau dépasse le cadre de ce tutoriel.  


## Documentation associée
{:related}

1. Le service VRF (Virtual Routing and Forwarding) constitue une alternative à l'utilisation du service VLAN Spanning pour connecter des réseaux via un compte {{site.data.keyword.Bluemix_notm}}. VRF  est obligatoire pour tous les clients qui utilisent [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview). [Présentation du service VRF (Virtual Routing and Forwarding) sur IBM Cloud](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [Le réseau IBM Cloud](https://www.ibm.com/cloud/data-centers/)
