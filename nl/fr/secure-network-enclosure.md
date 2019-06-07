---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-08"
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

# Isolation de charges de travail à l'aide d'un réseau privé sécurisé 
{: #secure-network-enclosure}

La nécessité d'environnements de réseau privé isolés et sécurisés est au coeur du modèle de déploiement d'applications IaaS sur un cloud public. Les pare-feux, les réseaux locaux virtuels (VLAN), le routage et les réseaux privés (VPN) sont tous des composants nécessaires à la création d'environnements privés isolés. Cet isolement permet aux machines virtuelles et aux serveurs bare metal d'être déployés en toute sécurité dans des topologies d'applications multiniveau complexes, tout en protégeant efficacement des risques sur le réseau Internet public.   

Ce tutoriel explique comment un [Dispositif Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-) (VRA) peut être configuré sur {{site.data.keyword.Bluemix_notm}} pour créer un réseau privé sécurisé (enceinte). Le dispositif de passerelle VRA fournit dans un seul package autogéré, un pare-feu, une passerelle VPN, une conversion d'adresses réseau (NAT) et un routage de niveau entreprise. Dans ce tutoriel, un VRA est utilisé pour illustrer la création d'un environnement de réseau fermé et isolé sur {{site.data.keyword.Bluemix_notm}}. Dans cette enceinte, des topologies d’application peuvent être créées à l’aide des technologies bien connues de routage d'adresses IP, de VLAN, de sous-réseaux d'adresses IP, de règles de pare-feu, de serveurs virtuels et bare metal.   

{:shortdesc}

Ce tutoriel constitue un point de départ pour une mise en réseau classique sur {{site.data.keyword.Bluemix_notm}} et ne doit pas être considéré comme une capacité de production en l'état. Les fonctionnalités supplémentaires pouvant être envisagées sont les suivantes : 
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [Dispositifs de pare-feux matériels](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* [VPN IPSec](https://{DomainName}/catalog/infrastructure/ipsec-vpn) pour une connectivité sécurisée à votre centre de données.
* Haute disponibilité grâce aux VRA en cluster et aux doubles liaisons montantes. 
* Journalisation et audit des événements de sécurité. 

## Objectifs 
{: #objectives}

* Déployer un dispositif Virtual Router Appliance (VRA) 
* Définir des VLAN et des sous-réseaux IP pour déployer des machines virtuelles et des serveurs bare metal 
* Sécuriser le VRA et l'enceinte à l'aide des règles de pare-feu 

## Services utilisés
{: #products}

Ce tutoriel utilise les services {{site.data.keyword.Bluemix_notm}} suivants : 
* [Dispositif Virtual Router Appliance (VRA)](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)
Ce tutoriel peut entraîner des coûts. Le VRA est disponible uniquement moyennant un forfait mensuel. 

## Architecture
{: #architecture}

<p style="text-align: center;">

  ![Architecture](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. Configurer le VPN
2. Déployer le VRA 
3. Créer un serveur virtuel
4. Acheminer l'accès via le VRA 
5. Configurer le pare-feu de l'enceinte 
6. Définir la zone APP 
7. Définir la zone INSIDE 

## Avant de commencer
{: #prereqs}

### Configuration de l'accès au VPN 

Dans ce tutoriel, l'enceinte réseau créée n'est pas visible sur le réseau Internet public. Le VRA et tous les serveurs sont accessibles uniquement via le réseau privé et vous utilisez votre VPN pour la connectivité.  

1. [Assurez-vous que votre accès VPN est activé.](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

Vous devez être **Utilisateur principal** pour activer l'accès VPN ou contactez l'utilisateur principal pour l'accès.
{:tip}
2. Obtenez vos données d'identification d'accès VPN en sélectionnant votre utilisateur dans la [liste des utilisateurs](https://{DomainName}/iam#/users). 
3. Connectez-vous au VPN via [l'interface Web](https://www.softlayer.com/VPN-Access) ou utilisez un client VPN pour [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) ou [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Pour le client VPN, utilisez le nom de domaine complet d'un point d'accès unique de VPN de centre de données à partir de la [page d'accès Web VPN](https://www.softlayer.com/VPN-Access), au format *vpn.xxxnn.softlayer.com* en tant qu'adresse de passerelle.
{:tip}

### Vérification des droits de compte 

Contactez votre utilisateur principal d'infrastructure pour obtenir les droits suivants :
- **Droits rapides** - Utilisateur de base
- **Réseau** afin que vous puissiez créer et configurer l'enceinte. Tous les droits Réseau sont obligatoires. 
- Les droits **Services** pour gérer les clés SSH

### Téléchargement de clés SSH 

Via le portail, [téléchargez la clé publique SSH](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial) qui sera utilisée pour accéder et administrer le VRA et le réseau privé.   

### Centre de données cible 

Choisissez un centre de données {{site.data.keyword.Bluemix_notm}} pour déployer le réseau privé sécurisé.  

### Commande de réseaux VLAN

Pour créer l'enceinte privée dans le centre de données cible, vous devez d'abord attribuer les VLAN privés requis pour les serveurs. Le premier VLAN privé et le premier VLAN public sont gratuits. Les VLAN supplémentaires prenant en charge une topologie d’applications multiniveau sont facturables.  

Pour vous assurer que suffisamment de VLAN sont disponibles sur le même routeur de centre de données et peuvent être associés au VRA, nous vous conseillons de les commander. Voir [Commande de réseaux VLAN](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans).

## Mise à disposition du dispositif Virtual Router Appliance (VRA) 
{: #VRA}

La première étape consiste à déployer un VRA qui fournit le routage IP et le pare-feu pour l'enceinte de réseau privé. Le réseau Internet public est accessible à partir de l’enceinte par un VLAN de transit d'interface avec le public, fourni par {{site.data.keyword.Bluemix_notm}} ; une passerelle et, éventuellement, un pare-feu matériel établissent la connectivité du VLAN public vers les VLAN de l’enceinte privée sécurisée. Dans ce tutoriel de solution, un dispositif Virtual Router Appliance (VRA) fournit ce périmètre de passerelle et de pare-feu.  

1. Dans le catalogue, sélectionnez un [Dispositif de passerelle](https://{DomainName}/gen1/infrastructure/provision/gateway).
3. Dans la section **Fournisseur de passerelle**, sélectionnez AT&T. Vous pouvez choisir entre une vitesse de liaison montante "jusqu'à 20 Gbits/s" ou "jusqu'à 2 Gbits/s". 
4. Dans la section **Nom d'hôte**, entrez un nom d’hôte et un domaine pour votre nouveau VRA.
5. Si vous cochez la case **Haute disponibilité**, vous obtenez deux dispositifs VRA fonctionnant dans une configuration active/de sauvegarde à l'aide de VRRP. 
6. Dans la section **Emplacement**, sélectionnez l'emplacement et le **Module** dans lesquels doit se trouver le VRA. 
7. Sélectionnez Processeur unique ou Processeur double. Vous obtenez une liste de serveurs. Choisissez un serveur en cliquant sur son bouton radio.  
8. Sélectionnez la quantité de mémoire **RAM**. Pour un environnement de production, il est recommandé d'utiliser un minimum de 64 Go de RAM. 8 Go minimum pour l'environnement de test. 
9. Sélectionnez une **Clé SSH** (facultatif). Cette clé ssh étant installée sur le VRA, l'utilisateur vyatta peut être utilisé pour accéder au VRA à l'aide de cette clé. 
10. Unité de disque dur. Conservez la valeur par défaut. 
11. Dans la section **Vitesses de port pour la liaison montante**, sélectionnez la combinaison de vitesse, de redondance et d'interfaces privées et/ou publiques répondant à vos besoins. 
12. Dans la section **Modules complémentaires**, conservez les valeurs par défaut. Si vous souhaitez utiliser IPv6 sur l'interface publique, sélectionnez l'adresse IPv6.

Dans la partie droite, vous pouvez voir votre **Récapitulatif de la commande**. Cochez la case _J'ai lu et j'accepte les accords de services tiers ci-dessous :_ et cliquez sur le bouton **Créer**. Votre passerelle est déployée.

La [ Liste des unités](https://{DomainName}/classic/devices) affiche le VRA presque immédiatement avec le symbole d’une **horloge**, ce qui indique que des transactions sont en cours sur ce périphérique. Tant que la création du VRA n'est pas terminée, le symbole de l'**horloge** est affiché et, à l'exception de l'affichage des détails, il est impossible d'effectuer des actions de configuration sur le périphérique. {:tip}

### Examen du VRA déployé 

1. Inspectez le nouveau VRA. Dans le [tableau de bord Infrastructure](https://{DomainName}/classic), sélectionnez **Réseau** dans le volet de gauche, puis **Tous les dispositifs de passerelle** pour accéder à la page [Tous les dispositifs de passerelle](https://{DomainName}/classic/network/gatewayappliances). Sélectionnez le nom du nouveau VRA dans la colonne **Passerelle** pour accéder à la page Détails de la passerelle.![](images/solution33-secure-network-enclosure/Gateway-detail.png) 

2. Notez les adresses IP `privée` et `publique` du VRA en vue d'une utilisation ultérieure. 

## Configuration initiale du VRA 
{: #initial_VRA_setup}

1. A partir de votre poste de travail, via le VPN SSL, connectez-vous au VRA à l'aide du compte **vyatta** par défaut, en acceptant les invites de sécurité SSH.  
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   Si SSH vous invite à entrer un mot de passe, cela signifie que la clé SSH n'est pas incluse dans la version. Accédez au VRA via le [navigateur Web](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui) à l'aide de l'`Adresse IP privée du VRA`. Le mot de passe provient de la page [Mots de passe logiciel](https://{DomainName}/classic/devices/passwords). Dans l'onglet **Configuration**, sélectionnez la branche System/login/vyatta et ajoutez la clé SSH souhaitée.
   {:tip}

   La configuration du VRA nécessite que le VRA soit placé en mode \[edit\] à l'aide de la commande `configure`. En mode `édition`, l'invite passe de `$` à `#`. Une fois la modification de la configuration VRA réussie, vous pouvez afficher vos modifications à l’aide de la commande `compare` et les vérifier à l’aide de la commande `validate`. Lorsque vous validez une modification à l'aide de la commande `commit`, celle-ci est appliquée à la configuration en cours et automatiquement enregistrée dans la configuration de démarrage. 


   {:tip}
2. Améliorez la sécurité en autorisant uniquement la connexion SSH. Une fois la connexion SSH établie, sur le réseau privé, désactivez l'accès via l'authentification userid/password.  
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
A ce stade, dans ce tutoriel, il est supposé que toutes les commandes VRA sont entrées à l’invite `edit`, après la saisie de `configure`.
3. Examen de la configuration initiale 
   ```
   show
   ```
   {: codeblock}

   Le VRA est préconfiguré pour l'environnement IaaS {{site.data.keyword.Bluemix_notm}}. Les éléments suivants sont inclus : 
   - Serveur NTP
   - Serveurs de noms
   - SSH
   - Serveur Web HTTPS  
   - Fuseau horaire par défaut US/Chicago
4. Définissez le fuseau horaire local selon vos besoins. La saisie automatique avec la touche de tabulation répertorie les valeurs de fuseau horaire potentielles
   ```
   set system time-zone <timezone>
   ```
   {: codeblock}
5. Définissez le comportement de la commande PING. Le ping n'est pas désactivé pour faciliter le traitement des incidents liés au routage et au pare-feu.  
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. Activez le fonctionnement du pare-feu avec état. Par défaut, le pare-feu VRA est sans état.  
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. Validez et sauvegardez automatiquement vos modifications dans la configuration de démarrage.  
   ```
   commit
   ```
   {: codeblock}

## Commande du premier serveur virtuel 
{: #order_virtualserver}

Un serveur virtuel est créé à ce stade pour faciliter le diagnostic des erreurs de configuration du VRA. L'accès réussi à l'Instance de serveur virtuel (VSI) est validé sur le réseau privé {{site.data.keyword.Bluemix_notm}} avant que l'accès à celui-ci ne soit routé via le VRA lors d'une étape ultérieure.  

1. Commandez un [serveur virtuel](https://{DomainName}/catalog/infrastructure/virtual-server-group)  
2. Sélectionnez **Serveur virtuel public** et poursuivez.
3. Sur la page de commande : 
   - Définissez **Facturation** sur **Horaire**.
   - Définissez le *Nom d'hôte VSI* et le *Nom de domaine*. Ce nom de domaine n'est pas utilisé pour le routage et le DNS, mais doit respecter les normes de dénomination de votre réseau.  
   - Définissez l'**Emplacement** sur celui du VRA.
   - Définissez le **Profil** sur **C1.1x1**
   - Ajoutez la **Clé SSH** que vous avez spécifiée précédemment.
   - Définissez le **Système d'exploitation** sur **CentOS 7.x - Minimal**
   - Dans la section **Vitesses de port pour la liaison montante**, l'interface réseau par défaut, *publique et privée*, doit être modifiée pour spécifier uniquement **Liaison montante de réseau privé**. Cela garantit que le nouveau serveur n'a pas d'accès direct à Internet et que l'accès est contrôlé par les règles de routage et de pare-feu du VRA. 
   - Définissez le **VLAN privé** sur l'ID VLAN du VLAN privé commandé précédemment. 
4. Cliquez sur la case à cocher pour accepter les accords de service tiers, puis sur **Créer**. 
5. Surveillez l'exécution sur la page [Unités](https://{DomainName}/classic/devices) ou par courrier électronique.  
6. Notez l’*Adresse IP privée* de l'instance VSI pour une étape ultérieure, et vérifiez que, dans la section **Réseau** de la page **Détails de l'unité**, l'instance VSI est attribuée au VLAN correct. Si tel n'est pas le cas, supprimez cette VSI et créez une nouvelle VSI sur le VLAN correct.  
7. Vérifiez que l'accès à l'instance VSI a abouti via le réseau privé {{site.data.keyword.Bluemix_notm}} à l'aide de la commande ping et de SSH, à partir de votre poste de travail local via le réseau VPN. 
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## Routage de l'accès VLAN via le VRA 
{: #routing_vlan_via_vra}

Le ou les VLAN privés du serveur virtuel ont été associés à ce dispositif VRA par le système de gestion {{site.data.keyword.Bluemix_notm}}. A ce stade, la VSI est toujours accessible via le routage IP sur le réseau privé {{site.data.keyword.Bluemix_notm}}. Vous allez maintenant acheminer le sous-réseau via le VRA pour créer le réseau privé sécurisé et valider en confirmant que la VSI n'est plus accessible.  

1. Accédez à la page Détails de la passerelle du VRA via la page [Tous les dispositifs de passerelle](https://{DomainName}/classic/network/gatewayappliances) et localisez la section **VLAN associés** dans la moitié inférieure de la page. Le VLAN associé est répertorié ici.  
2. Si vous souhaitez ajouter des VLAN à ce stade, accédez à la section **Associer un VLAN**. La liste déroulante, *Sélectionner Type de VLAN*, doit être activée et vous pouvez sélectionner d’autres VLAN mis à disposition.![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png) 

   Si aucun VLAN éligible n'est affiché, aucun VLAN n'est disponible sur le même routeur que le VRA. Vous devez générer un [ticket de demande de service](https://{DomainName}/unifiedsupport/cases/add) pour demander un VLAN privé sur le même routeur que le VRA.
{:tip}
5. Sélectionnez le VLAN que vous souhaitez associer au VRA et cliquez sur Sauvegarder. L'association initiale du VLAN peut prendre quelques minutes. Une fois terminée, le VLAN doit apparaître sous l’en-tête **VLAN associés**.  

A ce stade, le VLAN et le sous-réseau associé ne sont pas protégés ni routés via le VRA et l'instance VSI est accessible via le réseau privé {{site.data.keyword.Bluemix_notm}}. L'état du VLAN est indiqué comme *Ignoré*.

4. Sélectionnez **Actions** dans la colonne de droite, puis **Router le VLAN** pour router le VLAN/sous-réseau via le VRA. Cette opération prend quelques minutes. L'actualisation de l'écran indique que le VLAN est *Routé*.  
5. Sélectionnez le [Nom du VLAN](https://{DomainName}/classic/network/vlans/) pour afficher les détails du VLAN. La VSI mise à disposition apparaît ainsi que le sous-réseau IP principal attribué. Notez l'ID du VLAN privé \<nnnn\> (1199 dans cet exemple), car il sera utilisé dans une étape ultérieure.  
6. Sélectionnez le [sous-réseau](https://{DomainName}/classic/network/subnets) pour afficher les détails du sous-réseau IP. Notez le réseau, les adresses de passerelle et le CIDR (/26) du sous-réseau, car ils sont nécessaires à la configuration ultérieure du VRA. 64 adresses IP principales sont configurées sur le réseau privé et pour rechercher l'adresse de la passerelle, il peut être nécessaire de sélectionner la page 2 ou 3.  
7. Vérifiez que le sous-réseau/VLAN est routé vers le VRA et que la VSI **N'EST PAS** accessible via le réseau de gestion à partir de votre poste de travail à l'aide de la commande `ping`.  
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}

Cette action termine la configuration du VRA via la console {{site.data.keyword.Bluemix_notm}}. Le travail supplémentaire de configuration de l'enceinte et du routage IP est maintenant effectué directement sur le VRA via SSH.  

## Configuration du routage IP et sécurisation de l'enceinte
{: #vra_setup}

Lorsque la configuration du VRA est validée, la configuration en cours est modifiée et les modifications sont automatiquement enregistrées dans la configuration de démarrage. 

Si vous souhaitez revenir à une configuration de travail précédente, par défaut, les 20 derniers points de validation peuvent être affichés, comparés ou restaurés. Reportez-vous au guide [Vyatta Network OS Basic System Configuration Guide](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation) pour plus de détails sur la validation et l'enregistrement de la configuration. 
   ```bash
   show system commit 
   rollback n
   compare
   ```
   {: codeblock}

### Configuration du routage IP du VRA 

Configurez l'interface réseau virtuel du VRA pour la router vers le nouveau sous-réseau à partir du réseau privé {{site.data.keyword.Bluemix_notm}}.   

1. Connectez-vous au VRA par SSH.  
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}
2. Créez une interface virtuelle avec l'ID du VLAN privé, l'adresse IP de la passerelle de sous-réseau et le CIDR enregistré au cours des étapes précédentes. Le CIDR est généralement /26.  
   ```
   set interfaces bonding dp0bond0 vif <VLAN ID> address <Subnet Gateway IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   Il est essentiel que l'adresse **`<Subnet Gateway IP>`** soit utilisée. Il s’agit généralement d’une adresse de plus que celle de départ du sous-réseau. L'entrée d'une adresse de passerelle non valide entraîne l'erreur `Configuration path: interfaces bonding dp0bond0 vif xxxx address [x.x.x.x] is not valid`. Corrigez la commande et entrez-la à nouveau.  Vous pouvez rechercher cette adresse dans Réseau > Gestion IP > Sous-réseaux. Cliquez sur le sous-réseau dont vous avez besoin pour connaître l'adresse de la passerelle. La deuxième entrée de la liste avec la description **Passerelle** est l'adresse IP à saisir pour <IP de passerelle du sous-réseau>/<CIDR>. {: tip}

3. Affichez la nouvelle interface virtuelle (vif) : 
   ```
   show interfaces
   ```
   {: codeblock}

   Voici un exemple de configuration d'interface illustrant vif 1199 et l'adresse de la passerelle de sous-réseau. ![](images/solution33-secure-network-enclosure/show_interfaces.png) 
4. Vérifiez que l'instance VSI est à nouveau accessible via le réseau de gestion à partir de votre poste de travail.  
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
   
   Si la VSI n'est pas accessible, vérifiez que la table de routage IP du VRA est configurée comme prévu. Supprimez et recréez la route si nécessaire. Pour exécuter une commande show en mode configuration, vous pouvez utiliser la commande run     
   ```bash
    run show ip route <Subnet Gateway IP>
   ```
   {: codeblock}

Cette action termine la configuration du routage IP.

### Configuration d'une enceinte sécurisée 

L'enceinte du réseau privé sécurisé est créée via la configuration de zones et de règles de pare-feu. Consultez la documentation du VRA sur la [configuration des pare-feux](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls) avant de poursuivre.  

Deux zones sont définies : 
   - INSIDE : réseaux privés et de gestion IBM
   - APP : VLAN et sous-réseau utilisateur dans l'enceinte du réseau privé		

1. Définissez les pare-feux et les paramètres par défaut. 
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   Si une commande set est exécutée deux fois par erreur, vous recevez le message *'Configuration path xxxxxxxx is not valid. Node exists'*. Ce message peut être ignoré. Pour modifier un paramètre incorrect, il est nécessaire de supprimer préalablement le noeud à l'aide de la commande 'delete security xxxxx xxxx xxxxx'.
   {:tip}
2. Créez le groupe de ressources de réseau privé {{site.data.keyword.Bluemix_notm}}. Ce groupe d'adresses définit les réseaux privés {{site.data.keyword.Bluemix_notm}} pouvant accéder à l'enceinte et les réseaux pouvant être atteints à partir de l'enceinte. Deux ensembles d’adresses IP nécessitent un accès vers et depuis l’enceinte sécurisée : il s’agit des centres de données du VPN SSL et du réseau {{site.data.keyword.Bluemix_notm}} Service Network (réseau de back-end/privé). [{{site.data.keyword.Bluemix_notm}} IP Ranges](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges) fournit la liste complète des plages d'adresses IP à autoriser.  
   - Définissez l'adresse VPN SSL du ou des centres de données que vous utilisez pour l'accès VPN. Dans la section VPN SSL des plages d'adresses IP d'{{site.data.keyword.Bluemix_notm}}, sélectionnez les points d'accès VPN de votre centre de données ou de votre cluster de centres de données. L'exemple ci-après représente les plages d'adresses VPN des centres de données {{site.data.keyword.Bluemix_notm}} de Londres.
     ```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - Définissez les plages d’adresses du réseau {{site.data.keyword.Bluemix_notm}} 'Service Network (sur réseau de back-end/privé)' pour WDC04, DAL01 et votre centre de données cible. L'exemple ci-après représente WDC04 (deux adresses), DAL01 et LON06.
```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. Créez la zone APP pour le VLAN et le sous-réseau utilisateur et la zone INSIDE pour le réseau privé {{site.data.keyword.Bluemix_notm}}. Attribuez les pare-feux créés précédemment. La définition de zone utilise les noms d'interface réseau du VRA pour identifier la zone associée à chaque VLAN. La commande permettant de créer la zone APP nécessite la spécification de l'ID du VLAN associé au VRA. Celle-ci est mise en évidence ci-après `<VLAN ID>`.
   ```
   set security zone-policy zone INSIDE description "IBM Internal network"
   set security zone-policy zone INSIDE default-action drop
   set security zone-policy zone INSIDE interface dp0bond0
   set security zone-policy zone INSIDE to APP firewall INSIDE-TO-APP 

   set security zone-policy zone APP description "Application network"
   set security zone-policy zone APP default-action drop

   set security zone-policy zone APP interface dp0bond0.<VLAN ID> 
   set security zone-policy zone APP to INSIDE firewall APP-TO-INSIDE 
   ```
   {: codeblock}
4. Validez la configuration et, à partir de votre poste de travail, vérifiez à l'aide de la commande ping que le pare-feu refuse maintenant le trafic via le VRA vers l'instance VSI :  
   ```
   commit
   ```
   {: codeblock}

   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
5. Définissez les règles d'accès au pare-feu pour udp, tcp et icmp.
   ```
   set security firewall name INSIDE-TO-APP rule 200 protocol icmp
   set security firewall name INSIDE-TO-APP rule 200 icmp type 8
   set security firewall name INSIDE-TO-APP rule 200 action accept 
   set security firewall name INSIDE-TO-APP rule 200 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 100 action accept 
   set security firewall name INSIDE-TO-APP rule 100 protocol tcp
   set security firewall name INSIDE-TO-APP rule 100 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 110 action accept 
   set security firewall name INSIDE-TO-APP rule 110 protocol udp
   set security firewall name INSIDE-TO-APP rule 110 source address ibmprivate
   commit

   set security firewall name APP-TO-INSIDE rule 200 protocol icmp
   set security firewall name APP-TO-INSIDE rule 200 icmp type 8
   set security firewall name APP-TO-INSIDE rule 200 action accept 
   set security firewall name APP-TO-INSIDE rule 200 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 100 action accept 
   set security firewall name APP-TO-INSIDE rule 100 protocol tcp
   set security firewall name APP-TO-INSIDE rule 100 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 110 action accept 
   set security firewall name APP-TO-INSIDE rule 110 protocol udp
   set security firewall name APP-TO-INSIDE rule 110 destination address ibmprivate
   commit
   ```
   {: codeblock}
6. Validez l'accès au pare-feu.  
   - Vérifiez que le pare-feu INSIDE-TO-APP autorise désormais le trafic ICMP et udp/tcp, à partir de votre ordinateur local.
     ```bash
     ping <VSI Private IP Address>
     SSH root@<VSI Private IP Address>
     ```
     {: codeblock}
   - Confirmez que le pare-feu APP-TO-INSIDE autorise le trafic ICMP et udp/tcp. Connectez-vous à l'instance VSI à l'aide de SSH et envoyez une requête ping à l'un des serveurs de noms{{site.data.keyword.Bluemix_notm}} à 10.0.80.11 et 10.0.80.12.
     ```bash
     SSH root@<VSI Private IP Address>
     [root@vsi  ~]# ping 10.0.80.11 
     ```
     {: codeblock}
7. Validez l'accès continu à l'interface de gestion du VRA via SSH à partir de votre poste de travail. Si l'accès est maintenu, vérifiez et enregistrez la configuration. Si tel n'est pas le cas, un redémarrage du VRA renvoie à une configuration opérationnelle.  
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   ```
   show security  
   ```
   {: codeblock}

### Débogage des règles de pare-feu 

Les journaux du pare-feu peuvent être consultés à partir de l'invite de commande opérationnelle du VRA. Dans cette configuration, seul le trafic perdu pour chaque zone est enregistré pour faciliter le diagnostic d'une configuration incorrecte du pare-feu.   

1. Consultez les journaux du pare-feu pour identifier le trafic refusé. L'examen périodique des journaux permet d'identifier si des serveurs de la zone APP tentent de contacter de manière correcte ou erronée des services sur le réseau IBM.  
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. Si les services ou les serveurs ne sont pas joignables et que rien ne s'affiche dans les journaux du pare-feu, vérifiez si le trafic IP ping/ssh prévu est présent sur l'interface réseau du VRA à partir du réseau privé {{site.data.keyword.Bluemix_notm}} ou sur l'interface du VRA vers le VLAN à l'aide du `<VLAN ID>` précédent.
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN ID> traffic
   ```

## Sécurisation du VRA 
{: #securing_the_vra}

1. Appliquez la politique de sécurité du VRA. Par défaut, la segmentation par pare-feu basée sur une politique ne sécurise pas l'accès au VRA même. Elle est configurée via le CPP (Control Plane Policing). Le VRA fournit un modèle d'ensemble de règles CPP de base que vous pouvez fusionner dans votre configuration : 
   ```bash
   merge /opt/vyatta/etc/cpp.conf 
   ```
   {: codeblock}

Cette opération crée un ensemble de règles de pare-feu nommé `CPP`, affiche les règles supplémentaires et valide en mode \[edit\].  
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. Sécurisation de l'accès SSH public. En raison d'un problème non résolu actuellement, lié au microprogramme Vyatta, il est déconseillé d'utiliser la commande `set service SSH listen-address x.x.x.x` pour limiter l'accès administratif SSH sur le réseau public. Alternativement, l'accès externe peut être bloqué via le pare-feu CPP pour la plage d'adresses IP publiques utilisées par l'interface publique du VRA. Le `<VRA Public IP Subnet>` utilisé ici est le même que le `<VRA Public IP Address>` avec le dernier octet égal à zéro (x.x.x.0). 
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <VRA Public IP Subnet>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit 
   ```
   {: codeblock}
3. Validez l'accès administratif SSH du VRA sur le réseau interne IBM. Si l'accès au VRA via SSH est perdu après l'exécution des validations, vous pouvez accéder au VRA via la console KVM disponible sur la page Détails de l'unité du VRA via le menu déroulant Action. 

Cette opération termine la configuration de l'enceinte de réseau privé sécurisé protégeant une zone unique de pare-feu contenant un VLAN et un sous-réseau. Vous pouvez ajouter des zones de pare-feu supplémentaires, des règles, des serveurs virtuels et bare metal, des VLAN et des sous-réseaux en appliquant les mêmes instructions.  

## Suppression de ressources 
{: #removeresources}

Dans cette étape, vous nettoyez les ressources pour supprimer ce que vous avez créé aux étapes précédentes. 

Le VRA est disponible moyennant un forfait mensuel. L'annulation ne donne pas lieu à un remboursement. Il est suggéré d’annuler uniquement si ce VRA n'est plus nécessaire le mois suivant. Si un cluster à haute disponibilité VRA double est requis, ce VRA unique peut être mis à niveau sur la page [Détails de la passerelle](https://{DomainName}/classic/network/gatewayappliances/).
{:tip}  

- Annulez tous les serveurs virtuels ou les serveurs bare metal
- Annulez le VRA 
- Annulez tout VLAN supplémentaire à l'aide d'un ticket de demande de service.  

## Contenu associé
{: #related}

- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Sous-réseaux IP statiques et portables](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence Platform](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Documentation Vyatta](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
