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

# Implémentation du mode BYOIP (Bring Your Own IP)
{: #byoip}

L’approche BYOIP (Bring Your own IP) est une exigence fréquente lorsque vous souhaitez connecter les réseaux client existants à l’infrastructure mise à disposition sur {{site.data.keyword.Bluemix_notm}}. L'intention est généralement de minimiser les modifications apportées à la configuration et aux opérations de routage réseau du client avec l'adoption d'un seul espace d'adressage IP, basé sur le schéma d'adressage IP existant du client. 

Ce tutoriel présente un bref aperçu des modèles d'implémentation BYOIP pouvant être utilisés avec {{site.data.keyword.Bluemix_notm}} et un arbre de décisions permettant d'identifier le modèle approprié lors de la réalisation de l'enceinte sécurisée, comme décrit dans le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). La configuration peut nécessiter des informations supplémentaires de la part de votre équipe réseau sur site, du support technique {{site.data.keyword.Bluemix_notm}} ou des services IBM. 

{:shortdesc}

## Objectifs
{: #objectives}

* Comprendre les modèles d'implémentation de BYOIP 
* Sélectionner le modèle d'implémentation pour {{site.data.keyword.Bluemix_notm}}

## Adressage IP {{site.data.keyword.Bluemix_notm}}

{{site.data.keyword.Bluemix_notm}} utilise un certain nombre de plages d'adresses privées, plus particulièrement 10.0.0.0/8, et dans certains cas, ces plages peuvent entrer en conflit avec les réseaux client existants. Si des conflits d'adresses se produisent, un certain nombre de modèles permettent de prendre en charge BYOIP pour garantir l'interopérabilité avec le réseau IBM Cloud. 

-	Conversion NAT
-	Tunnellisation GRE (Generic Routing Encapsulation) 
-	Tunnellisation GRE avec alias d'adresse IP
-	Réseau superposé virtuel

Le choix du modèle est déterminé par les applications destinées à être hébergées sur {{site.data.keyword.Bluemix_notm}}. Il existe deux aspects essentiels, la sensibilité des applications à la mise en oeuvre du modèle et l'étendue du chevauchement des plages d'adresses entre le réseau client et {{site.data.keyword.Bluemix_notm}}. Des considérations supplémentaires s'appliquent également si l'intention d'utiliser une connexion à {{site.data.keyword.Bluemix_notm}} via un réseau privée dédiée est envisagée. Il est recommandé à tous les utilisateurs qui envisagent de mettre en oeuvre BYOIP de lire la documentation de [{{site.data.keyword.BluDirectLink}}
](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link). Pour les utilisateurs d'{{site.data.keyword.BluDirectLink}}, les instructions associées doivent être appliquées en tenant compte des informations présentées ici.

## Présentation des modèles de mise en oeuvre 
{: #patterns_overview}

1. **NAT** : conversion d'adresse NAT sur le routeur client local. Effectuez une conversion NAT sur site pour convertir le schéma d'adressage client en adresses IP attribuées par {{site.data.keyword.Bluemix_notm}} aux services IaaS mis à disposition.  
2. **Tunnellisation GRE** : le schéma d'adressage est unifié par le routage du trafic IP via un tunnel GRE entre {{site.data.keyword.Bluemix_notm}} et le réseau local, généralement via un réseau privé virtuel (VPN). Il s'agit du scénario illustré dans ce [tutoriel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN).  

   Il existe deux sous-modèles en fonction de l'éventualité de chevauchement des espaces adresse. 
     * Absence de chevauchement d'adresses : il n'y a pas de chevauchement d'adresses dans les plages d'adresses et de risques de conflit entre les réseaux. 
     * Chevauchement d'adresses partiel : les espaces adresse IP du client et d'{{site.data.keyword.Bluemix_notm}} utilisent la même plage d'adresses et il existe un risque de chevauchement et de conflit. Dans ce cas, les adresses de sous-réseau client qui ne se chevauchent pas dans le réseau privé d'{{site.data.keyword.Bluemix_notm}} sont choisies. 

3. Tunnellisation GRE + alias d'IP
Le schéma d'adressage est unifié par le routage du trafic IP via un tunnel GRE entre le réseau local et les alias d'adresses IP attribués aux serveurs sur {{site.data.keyword.Bluemix_notm}}. Il s'agit d'un cas particulier du scénario illustré dans [ce tutoriel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN). Une interface et un alias IP supplémentaires pour un sous-réseau IP compatible sont créés sur les serveurs virtuels et bare metal mis à disposition sur {{site.data.keyword.Bluemix_notm}}, pris en charge par une configuration de routage appropriée sur le dispositif VRA. 

4. Réseau superposé virtuel
[Le VPC (Virtual Private Cloud) d'{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) prend en charge le modèle BYOIP pour les environnements intégralement virtuels sur {{site.data.keyword.Bluemix_notm}}. Il peut être considéré comme une alternative à l'enceinte de réseau privé sécurisé décrite dans [ce tutoriel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure). 

Vous pouvez également envisager une solution telle que VMware NSX qui implémente un réseau superposé virtuel dans une couche sur le [réseau {{site.data.keyword.Bluemix_notm}}. Toutes les adresses BYOIP du réseau superposé virtuel sont indépendantes des [plages d’adresses réseau {{site.data.keyword.Bluemix_notm}}. Voir [Getting started with VMware and [{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud).

## Arbre de décisions de modèle d'implémentation
{: #decision_tree}

L'arbre de décisions représenté ici peut être utilisé pour déterminer le modèle d'implémentation approprié.  

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

Les remarques suivantes fournissent des indications supplémentaires : 

### La conversion NAT pose-t-elle un problème pour vos applications ? 
{: #nat_consideration}

Vous trouverez ci-dessous deux cas distincts dans lesquels la conversion NAT peut poser un problème. Dans ces cas, la conversion NAT ne doit pas être utilisée.  

- Certaines applications telles que la communication de domaine Microsoft AD et les applications P2P peuvent rencontrer des problèmes techniques avec la conversion NAT. 
- Lorsque des serveurs inconnus doivent communiquer avec [{{site.data.keyword.Bluemix_notm}} ou des centaines de connexions bidirectionnelles entre [{{site.data.keyword.Bluemix_notm}} et des serveurs sur site sont nécessaires. Dans ce cas, une partie du mappage ne peut pas être configurée sur la table NAT/routeur du client, car il est impossible d'identifier le mappage au préalable. 


### Aucun chevauchement d'adresses
{: #no-overlap}

La plage 10.0.0.0/8 est-elle utilisée dans le réseau sur site ? En l'absence de chevauchement d'adresses entre le réseau local et le [réseau privé {{site.data.keyword.Bluemix_notm}}, vous pouvez utiliser la tunnellisation GRE décrite dans [ce tutoriel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN) entre le réseau sur site et IBM Cloud pour éviter la conversion NAT. Pour cela, vous devez passer en revue l'utilisation des adresses réseau avec l'équipe réseau sur site.  

### Chevauchement d'adresses partiel
{: #partial_overlap}

Si l'un des composants de la plage 10.0.0.0/8 est utilisé sur le réseau local, existe-t-il des sous-réseaux ne se chevauchant pas disponibles sur le [réseau {{site.data.keyword.Bluemix_notm}} ? Examinez l'utilisation des adresses réseau existantes avec l'équipe réseau sur site et contactez le [service commercial {{site.data.keyword.Bluemix_notm}} pour identifier les réseaux disponibles qui ne se chevauchent pas.  

### L'attribution d'alias d'IP est-elle problématique ? 
{: #ip_alias}

S'il n'existe aucune adresse protégée contre le chevauchement, l'attribution d'alias d'IP peut être implémentée sur des serveurs virtuels et bare metal déployés dans l'enceinte de réseau privé sécurisé. L'attribution d'alias d'IP affecte plusieurs adresses de sous-réseau sur une ou plusieurs interfaces réseau de chaque serveur.  

L'attribution d'alias d'IP est une technique couramment utilisée, bien qu'il soit recommandé de passer en revue les configurations du serveur et des applications pour déterminer si elles fonctionnent correctement avec des configurations à emplacements multiples et d'alias IP.   

Une configuration de routage supplémentaire sur le dispositif VRA est nécessaire pour créer des routes dynamiques (par exemple, BGP) ou des routes statiques pour les sous-réseaux BYOIP.  

## Contenu associé
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
