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

# Configuration d'un réseau privé virtuel (VPN) dans un réseau privé sécurisé
{: #configuring-IPSEC-VPN}

La nécessité de créer une connexion privée entre un environnement réseau distant et des serveurs sur le réseau privé d'{{site.data.keyword.Bluemix_notm}} est une exigence courante. Généralement, cette connectivité prend en charge les charges de travail hybrides, les transferts de données, les charges de travail privées ou l'administration de systèmes sur {{site.data.keyword.Bluemix_notm}}. Un tunnel de réseau privé virtuel (Virtual Private Network - VPN) de site à site est l’approche habituelle pour sécuriser la connectivité entre les réseaux.  

{{site.data.keyword.Bluemix_notm}} fournit un certain nombre d'options pour la connectivité de centre de données de site à site, en utilisant un VPN sur le réseau Internet public ou via une connexion réseau dédiée privée. 

Consultez [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link) 
pour plus de détails sur les liaisons de réseau sécurisé dédiées avec {{site.data.keyword.Bluemix_notm}}. Un VPN sur le réseau Internet public offre une option à moindre coût, mais sans garantie de bande passante.  

Il existe deux options de VPN appropriées pour la connectivité sur le réseau Internet public avec les serveurs mis à disposition sur {{site.data.keyword.Bluemix_notm}} :

-	[VPN IPSEC]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Ce tutoriel présente la configuration d'un VPN IPSec de site à site à l'aide d'un dispositif VRA (Virtual Router Appliance) permettant de connecter un sous-réseau d'un centre de données client à un sous-réseau sécurisé sur le réseau privé {{site.data.keyword.Bluemix_notm}}. 

Cet exemple s'appuie sur le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure). Il utilise un VPN IPSec de site à site, un tunnel GRE et un routage statique. Des configurations VPN plus complexes utilisant le routage dynamique (BGP, etc.) et les tunnels VTI sont consultables dans la [documentation supplémentaire sur les dispositifs VRA ](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Objectifs
{: #objectives}

- Documenter les paramètres de configuration pour le VPN IPSec 
- Configurer un VPN IPSec sur un dispositif Virtual Router Appliance
- Acheminer le trafic via un tunnel GRE 

## Services utilisés
{: #products}

Ce tutoriel utilise les services {{site.data.keyword.Bluemix_notm}} suivants :  

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Ce tutoriel peut entraîner des coûts. Le VRA est disponible uniquement moyennant un forfait mensuel. 

## Architecture
{: #architecture}

<p style="text-align: center;">

  ![Architecture](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	Documentez la configuration d'un VPN 
2.	Créez un VPN IPSec sur un dispositif VRA 
3.	Configurez un VPN de centre de données et un tunnel 
4.	Créez un tunnel GRE 
5.	Créez une route IP statique 
6.	Configurez le pare-feu  

## Avant de commencer
{: #prereqs}

Ce tutoriel connecte l'enceinte de réseau privé sécurisé créée dans le tutoriel [Isolation de charges de travail à l'aide d'un réseau privé sécurisé](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) à votre centre de données. Ce tutoriel doit être effectué en premier lieu. 

## Documentation de la configuration d'un VPN 
{: #Document_VPN}

La configuration d'une liaison de site à site de VPN IPSec entre votre centre de données et {{site.data.keyword.Bluemix_notm}} nécessite une coordination avec votre équipe réseau sur site pour déterminer de nombreux paramètres de configuration, le type de tunnel et les informations de routage IP. Les paramètres doivent correspondre exactement pour que vous puissiez établir une connexion de VPN opérationnelle. En règle générale, votre équipe de réseau sur site définit la configuration en fonction des normes définies par l'entreprise et vous fournit l'adresse IP nécessaire de la passerelle de VPN du centre de données, ainsi que les plages d'adresses de sous-réseau accessibles. 

Avant de commencer la configuration du VPN, les adresses IP des passerelles VPN et des plages de sous-réseaux du réseau IP doivent être déterminées et disponibles pour la configuration du VPN du centre de données et pour l'enceinte de réseau privé sécurisé d'{{site.data.keyword.Bluemix_notm}}. Elles sont illustrées à la figure suivante, où la zone APP de l’enceinte sécurisée est connectée via le tunnel IPSec aux systèmes du "sous-réseau IP DC" du centre de données client.    

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

Les paramètres suivants doivent être convenus et documentés entre l'utilisateur {{site.data.keyword.Bluemix_notm}} qui configure le VPN et l'équipe réseau du centre de données client. Dans cet exemple, les adresses IP des tunnels distant et local sont définies sur 192.168.10.1 et 192.168.10.2. Tout sous-réseau arbitraire peut être utilisé avec l'accord de l'équipe de mise en réseau sur site. 

| Elément  | Description |
|:------ |:--- | 
| &lt;ike group name&gt; | Nom donné au groupe IKE pour la connexion. |
| &lt;ike encryption&gt; | Norme de chiffrement IKE convenue à utiliser entre {{site.data.keyword.Bluemix_notm}} et le centre de données client, généralement *aes256*. |
| &lt;ike hash&gt; | Hachage IKE convenu entre {{site.data.keyword.Bluemix_notm}} et le centre de données client, généralement *sha1*. |
| &lt;ike-lifetime&gt; | Durée de vie IKE du centre de données client, généralement *3600*. |
| &lt;esp group name&gt; | Nom attribué au groupe ESP pour la connexion. |
| &lt;esp encryption&gt; | Norme de chiffrement ESP convenue entre {{site.data.keyword.Bluemix_notm}} et le centre de données client, généralement *aes256*. |
| &lt;esp hash&gt; | Hachage ESP convenu entre {{site.data.keyword.Bluemix_notm}} et le centre de données client, généralement *sha1*. |
| &lt;esp-lifetime&gt; | Durée de vie ESP du centre de données client, généralement *1800*. |
| &lt;DC VPN Public IP&gt;  | Adresse IP publique sur Internet de la passerelle de VPN dans le centre de données client. | 
| &lt;VRA Public IP&gt; | Adresse IP publique du dispositif VRA. |
| &lt;Remote tunnel IP\/24&gt; | Adresse IP attribuée à l'extrémité distante du tunnel IPSec. Paire d'adresses IP dans la plage qui n'entre pas en conflit avec IP Cloud ou le centre de données client. |
| &lt;Local tunnel IP\/24&gt; | Adresse IP attribuée à l'extrémité locale du tunnel IPSec.   |
| &lt;DC Subnet/CIDR&gt; | Adresse IP du sous-réseau accessible dans le centre de données client et CIDR. |
| &lt;App Zone subnet/CIDR&gt; | Adresse IP du réseau et CIDR du sous-réseau de la zone APP créés dans le tutoriel de création d'un dispositif VRA. | 
| &lt;Shared-Secret&gt; | Clé de chiffrement partagée à utiliser entre {{site.data.keyword.Bluemix_notm}} et le centre de données client. |

## Configuration du VPN IPSec sur un dispositif VRA 
{: #Configure_VRA_VPN}

Pour créer le réseau VPN sur {{site.data.keyword.Bluemix_notm}}, les commandes et toutes les variables devant être modifiées sont mises en évidence ci-dessous avec les symboles &lt; &gt;. Les modifications sont identifiées ligne par ligne, pour chaque ligne à modifier. Les valeurs sont extraites du tableau.  

1. Connectez-vous à l'aide de SSH au dispositif VRA et passez en mode `[edit]`.
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2. Créez un groupe IKE (Internet Key Exchange). 
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike encryption>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike hash>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lifetime>
   ```
   {: codeblock}
3. Créez un groupe ESP (Encapsulating Security Payload) 
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp encryption>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp hash>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lifetime>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. Définissez la connexion de site à site 
   ```
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication mode pre-shared-secret
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication pre-shared-secret <Shared-Secret>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  connection-type initiate
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  ike-group <ike group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  local-address <VRA Public IP>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  default-esp-group <esp group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1 protocol gre
   commit
   save
   ```
   {: codeblock}

## Configuration d'un VPN de centre de données et d'un tunnel 
{: #Configure_DC_VPN}

1. L'équipe réseau du centre de données client configure une connexion de VPN IPSec identique avec une permutation des adresses &lt;DC VPN Public IP&gt; et &lt;VRA Public IP&gt;, avec les adresses de tunnels local et distant et avec une permutation des paramètres &lt;DC Subnet/CIDR&gt; et &lt;App Zone subnet/CIDR&gt;. Les commandes de configuration spécifiques dans le centre de données client dépendent du fournisseur du VPN. 
1. Vérifiez que l'adresse IP publique de la passerelle VPN DC est accessible sur Internet avant de poursuivre :
   ```
   ping <DC VPN Public IP>
   ```
   {: codeblock}
2. Lorsque la configuration du VPN du centre de données est terminée, la liaison IPSec devrait apparaître automatiquement. Vérifiez que la liaison a été établie et que l’état indique qu’il existe un ou plusieurs tunnels IPsec actifs. Vérifiez auprès du centre de données que les deux extrémités du VPN affichent les tunnels IPsec actifs. 
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. Si la liaison n'a pas été créée, vérifiez que les adresses locale et distante ont été correctement spécifiées et que les autres paramètres sont conformes aux attentes à l'aide de la commande debug : 
   ``` bash
   show vpn debug
   ```
   {: codeblock}

La ligne `peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......` devrait figurer dans la sortie. Si elle n'apparaît pas ou si "CONNECTING" est indiqué, une erreur s'est produite dans la configuration du VPN.  

## Définition du tunnel GRE  
{: #Define_Tunnel}

1. Créez le tunnel GRE en mode édition dans le VRA. 
   ```
   set interfaces tunnel tun0 address <Local tunnel IP/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <VRA Public IP>
   set interfaces tunnel tun0 remote-ip <DC VPN Public IP>
   commit
   ```
   {: codeblock}
2. Une fois que les deux extrémités du tunnel ont été configurées, il devrait apparaître automatiquement. Vérifiez l'état opérationnel du tunnel à partir de la ligne de commande du VRA. 
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   La première commande doit afficher le tunnel avec State et Link sous la forme `u/u` (UP/UP). La deuxième commande affiche davantage de détails sur le tunnel et indique que le trafic est transmis et reçus.
3. Vérifiez que le trafic traverse le tunnel
   ```bash
   ping <Remote tunnel IP>
   ```
   {: codeblock}
   Les décomptes TX et RX sur un `tunnel tun0 d'interfaces d'exposition` doivent s'incrémenter en cas de trafic `ping`.
4. Si le trafic ne circule pas, les commandes de l'`interface de contrôle` peuvent être utilisées pour étudier le trafic qui est observé sur chaque interface. L'interface `tun0` affiche le trafic interne via le tunnel. L'interface `dp0bond1` affiche le flux de trafic encapsulé vers et depuis la passerelle VPN distante.
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

Si aucun trafic de retour n'est visible, pour localiser le problème, l'équipe réseau du centre de données doit surveiller les flux de trafic au niveau des interfaces du VPN et du tunnel sur le site distant.  

## Création d'une route IP statique 
{: #Define_Routing}

Créez le routage VRA pour diriger le trafic vers le sous-réseau distant via le tunnel. 

1. Créez une route statique en mode édition dans le VRA. 
   ```
   set protocols static route <DC Subnet/CIDR>  next-hop <Remote tunnel IP>
   ```
   {: codeblock}   
2. Passez en revue la table de routage du VRA à partir de la ligne de commande du VRA. A ce stade, aucun trafic ne traverse la route car aucune règle de pare-feu n'autorise le trafic via le tunnel. Des règles de pare-feu sont requises pour le trafic déclenché à chaque extrémité. 
   ```bash
   show ip route
   ```
   {: codeblock}

## Configuration du pare-feu 
{: #Configure_firewall}

1. Créez des groupes de ressources pour le trafic icmp autorisé et les ports TCP.  
   ```
   set res group icmp-group icmpgrp type 8
   set res group icmp-group icmpgrp type 11
   set res group icmp-group icmpgrp type 3

   set res group port tcpports port 22
   set res group port tcpports port 80
   set res group port tcpports port 443
   commit
   ```
   {: codeblock}
2. Créez des règles de pare-feu pour le trafic vers le sous-réseau distant en mode d'édition du VRA. 
   ```
   set security firewall name APP-TO-TUNNEL default-action drop
   set security firewall name APP-TO-TUNNEL default-log

   set security firewall name APP-TO-TUNNEL rule 100 action accept
   set security firewall name APP-TO-TUNNEL rule 100 protocol tcp
   set security firewall name APP-TO-TUNNEL rule 100 destination port tcpports

   set security firewall name APP-TO-TUNNEL rule 200 protocol icmp
   set security firewall name APP-TO-TUNNEL rule 200 icmp group icmpgrp
   set security firewall name APP-TO-TUNNEL rule 200 action accept
   commit
   ```
   {: codeblock}

   ```
   set security firewall name TUNNEL-TO-APP default-action drop
   set security firewall name TUNNEL-TO-APP default-log

   set security firewall name TUNNEL-TO-APP rule 100 action accept
   set security firewall name TUNNEL-TO-APP rule 100 protocol tcp
   set security firewall name TUNNEL-TO-APP rule 100 destination port tcpports

   set security firewall name TUNNEL-TO-APP rule 200 protocol icmp
   set security firewall name TUNNEL-TO-APP rule 200 icmp group icmpgrp
   set security firewall name TUNNEL-TO-APP rule 200 action accept 
   commit
   ```
   {: codeblock}
2. Créez une zone pour le tunnel et associez des pare-feux au trafic déclenché dans l'une ou l'autre des zones. 
   ```
   set security zone-policy zone TUNNEL description "GRE Tunnel"
   set security zone-policy zone TUNNEL default-action drop
   set security zone-policy zone TUNNEL interface tun0

   set security zone-policy zone TUNNEL to APP firewall TUNNEL-TO-APP
   set security zone-policy zone APP to TUNNEL firewall APP-TO-TUNNEL
   commit
   save
   ```
   {: codeblock}
3. Pour vérifier que les pare-feux et le routage aux deux extrémités sont correctement configurés et autorisent désormais le trafic ICMP et TCP, exécutez une commande ping vers l'adresse de passerelle du sous-réseau distant, d'abord à partir de la ligne de commande du VRA, puis en vous connectant à l'instance VSI. 
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}
4. En cas d'échec de la commande ping sur la ligne de commande du VRA, vérifiez qu'une réponse au ping apparaît en réponse à une demande de ping sur l'interface de tunnel. 
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
L'absence de réponse indique un problème lié aux règles de pare-feu ou au routage dans le centre de données. Si une réponse apparaît dans la sortie du moniteur, mais que la commande ping arrive à expiration, vérifiez la configuration des règles de pare-feu VRA locales.
5. Un échec de la commande ping émise par l'instance VSI indique un problème lié aux règles de pare-feu VRA ou au routage dans la configuration du VRA ou de la VSI. Effectuez l'étape précédente pour vous assurer qu'une demande est envoyée et que la réponse est visible à partir du centre de données. La surveillance du trafic sur le VLAN local et l'examen des journaux du pare-feu aident à isoler le problème lié au routage ou au pare-feu. 
   ```
   monitor interfaces bonding dp0bond0.<VLAN ID>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

Cette opération termine la configuration du VPN à partir de l'enceinte de réseau privé sécurisé. Les tutoriels supplémentaires de cette série illustrent comment l'enceinte de réseau privé sécurisé peut accéder aux services sur le réseau Internet public. 

## Suppression de ressources 
{:removeresources}

Etapes à suivre pour supprimer les ressources créées dans ce tutoriel. 

Le VRA est disponible moyennant un forfait mensuel. L'annulation ne donne pas lieu à un remboursement. Il est suggéré d’annuler uniquement si ce VRA n'est plus nécessaire le mois suivant. Si un cluster à haute disponibilité VRA double est requis, ce VRA unique peut être mis à niveau sur la page [Détails de la passerelle](https://{DomainName}/classic/network/gatewayappliances).
{:tip}  

1. Annulez tous les serveurs virtuels ou les serveurs bare metal
2. Annulez le VRA 
3. Annulez tout VLAN supplémentaire à l'aide d'un ticket de demande de service.  

## Contenu associé
{:related}
- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Sous-réseaux IP statiques et portables](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Documentation Vyatta](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS IPsec Site-to-Site VPN Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS Tunnels Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
