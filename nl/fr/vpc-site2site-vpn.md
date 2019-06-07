---
copyright:
  years: 2019
lastupdated: "2019-05-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}
{:important: .important}

# Utilisation d'une passerelle VPC/VPN pour un accès sur site sécurisé et privé aux ressources de cloud
{: #vpc-site2site-vpn}

IBM va accepter un nombre limité de clients pour un programme d'accès anticipé à VPC qui commencera début avril 2019 ; l'accès élargi ne sera ouvert que dans les mois qui suivront. Si votre organisation souhaite avoir accès à IBM Virtual Private Cloud, veuillez remplir ce [formulaire de nomination](https://{DomainName}/vpc){: new_window} et un interlocuteur IBM entrera en contact avec vous concernant la procédure à suivre.
{: important}

IBM propose plusieurs méthodes pour étendre de manière sécurisée un réseau informatique local avec des ressources dans IBM Cloud. Vous bénéficiez ainsi de l'élasticité des serveurs de mise à disposition lorsque vous en avez besoin et vous pouvez les supprimer lorsqu'ils ne sont plus nécessaires. De plus, vous pouvez connecter facilement et en toute sécurité vos fonctionnalités sur site aux services {{site.data.keyword.cloud_notm}}.

Ce tutoriel vous explique comment connecter une passerelle de réseau privé virtuel (Virtual Private Network - VPN) sur site à un réseau VPN dans le cloud créé au sein d'un réseau VPC (passerelle VPC/VPN). Tout d'abord, vous allez créer un réseau {{site.data.keyword.vpc_full}} (VPC) et les ressources associées, telles que des sous-réseaux, des listes de contrôle d'accès (Access Control List - ACL) au réseau, des groupes de sécurité (Security Group - SG) et des instances de serveur virtuel (Virtual Server Instance - VSI). La passerelle VPC/VPN établit une liaison [IPsec](https://en.wikipedia.org/wiki/IPsec) site à site vers une passerelle VPN locale. Les protocoles IPsec et [Internet Key Exchange](https://en.wikipedia.org/wiki/Internet_Key_Exchange) (IKE) sont des normes ouvertes éprouvées permettant une communication sécurisée. 

Pour mieux illustrer l'accès sécurisé et privé, vous déployez un microservice sur une VSI pour accéder à {{site.data.keyword.cos_short}} (COS), représentant une application métier. Le service COS dispose d'un noeud final direct qui peut être utilisé pour une entrée/sortie privée sans frais lorsque tous les accès se trouvent dans la même région d'{{site.data.keyword.cloud_notm}}. Un ordinateur sur site accède au microservice COS. Tout le trafic passe par le VPN et donc par {{site.data.keyword.cloud_notm}}.

De nombreuses solutions VPN sur site populaires sont disponibles pour les passerelles de site à site. Ce tutoriel utilise la passerelle VPN [strongSwan](https://www.strongswan.org/) pour se connecter à la passerelle VPC/VPN. Pour simuler un centre de données sur site, vous devez installer la passerelle strongSwan sur une instance VSI dans {{site.data.keyword.cloud_notm}}.

{:shortdesc}
En résumé, à l'aide d'un VPC, vous pouvez 

- connecter vos systèmes sur site aux services et charges de travail exécutés dans {{site.data.keyword.cloud_notm}},
- assurer une connectivité privée et économique au COS, 
- connecter vos systèmes de cloud à des services et des workloads s'exécutant sur site. 

## Objectifs
{: #objectives}

* Accéder à un environnement de cloud privé virtuel à partir d'un centre de données sur site ou d'un cloud privé (virtuel). 
* Accéder en toute sécurité aux services cloud à l'aide de noeuds finaux de service privé. 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. Bien qu'il n'y ait pas de frais de mise en réseau pour accéder au COS à partir du microservice dans ce tutoriel, des frais de mise en réseau standard pour accéder au VPC sont facturés. 

## Architecture
{: #architecture}

Le diagramme suivant montre le cloud privé virtuel contenant un serveur d'applications. Le serveur d'applications héberge un microservice s'interfaçant avec le service {{site.data.keyword.cos_short}}. Un réseau sur site (simulé) et l'environnement de cloud virtuel sont connectés via des passerelles VPN. 

![Architecture](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. L'infrastructure (VPC, sous-réseaux, groupes de sécurité avec règles, ACL réseau et VSI) est configurée à l'aide d'un script fourni. 
2. Le microservice s'interface avec {{site.data.keyword.cos_short}} via un noeud final direct.
3. Une passerelle VPC/VPN est configurée pour exposer l'environnement de cloud privé virtuel au réseau local. 
4. Le logiciel de passerelle IPsec open source Strongswan est utilisé sur site pour établir la connexion du VPN avec l'environnement dans le cloud. 

## Avant de commencer
{: #prereqs}

- Installez tous les outils de ligne de commande nécessaires [en procédant comme suit](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview). Vous devez disposer du plug-in facultatif d'infrastructure d'interface CLI.
- Connectez-vous à {{site.data.keyword.cloud_notm}} via la ligne de commande. Pour plus de détails, voir [CLI Getting Started](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli).
- Vérifiez les droits utilisateur. Assurez-vous que votre compte d'utilisateur dispose des droits suffisants pour créer et gérer des ressources VPC. Pour obtenir la liste des droits requis, voir [Granting permissions needed for VPC users](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).
- Une clé SSH est nécessaire pour la connexion aux serveurs virtuels. Si vous ne possédez pas de clé SSH, consultez les [instructions de création d'une clé](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites). 
- Installez [**jq**](https://stedolan.github.io/jq/download/). Il est utilisé par les scripts fournis pour traiter la sortie JSON. 

## Déploiement d'un serveur d'applications virtuel dans un cloud privé virtuel 
Dans la section suivante, vous téléchargez les scripts pour configurer un environnement VPC de base de référence et un code pour un microservice devant servir d'interface avec {{site.data.keyword.cos_short}}. Vous mettez ensuite à disposition le service {{site.data.keyword.cos_short}} et configurez la base de référence. 

### Obtention du code
{: #setup}
Le tutoriel utilise des scripts pour déployer une base de référence de ressources d'infrastructure avant de créer les passerelles VPN. Ces scripts et le code du microservice sont disponibles dans un référentiel GitHub. 

1. Obtenez le code de l'application : 
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. Accédez aux scripts pour ce tutoriel en accédant à **vpc-tutorials**, puis à **vpc-site2site-vpn** :
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### Création de services
Dans cette section, vous vous connectez à {{site.data.keyword.cloud_notm}} dans l'interface CLI et vous créez une instance {{site.data.keyword.cos_short}}.

1. Veillez à effectuer les étapes préalables à la connexion.
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. Créez une instance [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) à l'aide d'un forfait **standard** ou **lite**.
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   Notez qu'une seule instance Lite peut être créée par compte. Si vous disposez déjà d'une instance {{site.data.keyword.cos_short}}, vous pouvez la réutiliser.
   {: tip}

3. Créez une clé de service avec le rôle **Lecteur** :
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. Obtenez les détails de la clé de service au format JSON et stockez-les dans un nouveau fichier **credentials.json** du sous-répertoire **vpc-app-cos**. Ce fichier est utilisé ultérieurement par l'application. 
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### Création de ressources de base de référence pour un réseau VPC 
{: #create-vpc}
Ce tutoriel fournit un script permettant de créer les ressources requises de base de référence, c’est-à-dire l’environnement de démarrage. Le script peut générer cet environnement dans un VPC existant ou créer un autre VPC. 

Dans la section suivante, créez ces ressources en configurant puis en exécutant un script de configuration. Le script intègre la configuration d'un hôte bastion, comme indiqué dans la section [Accès sécurisé aux instances distantes à l'aide d'un hôte bastion](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server).

1. Copiez l'exemple de fichier de configuration dans un fichier à utiliser : 

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. Editez le fichier **config.sh** et adaptez les paramètres pour votre environnement. Vous devez remplacer la valeur de **SSHKEYNAME** par le nom ou la liste de noms de clés SSH séparés par des virgules (voir la section "Avant de commencer"). Modifiez les différents paramètres **ZONE** en fonction de votre région cloud. Toutes les autres variables peuvent être conservées en l'état.
3. Pour créer les ressources dans un nouveau réseau VPC, exécutez le script comme suit : 

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   Pour réutiliser un VPC existant, transmettez son nom au script comme suit. Remplacez **YOUR_EXISTING_VPC** par le nom du VPC réel.
   ```sh
   REUSE_VPC=YOUR_EXISTING_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. Vous créez ainsi les ressources suivantes, y compris les ressources liées au bastion : 
   - 1 VPC (facultatif)
   - 1 passerelle publique
   - 3 sous-réseaux dans le VPC
   - 4 groupes de sécurité avec règles d'entrée et de sortie
   - 3 VSI : vpns2s-onprem-vsi (floating-ip est ONPREM_IP), vpns2s-cloud-vsi (floating-ip est VSI_CLOUD_IP) et vpns2s-bastion (floating-ip est BASTION_IP_ADDRESS)

   Notez pour une utilisation ultérieure les valeurs renvoyées pour **BASTION_IP_ADDRESS**, **VSI_CLOUD_IP**, **ONPREM_IP**, **CLOUD_CIDR** et **ONPREM_CIDR**. La sortie est également stockée dans le fichier **network_config.sh**. Ce fichier peut être utilisé pour une configuration automatisée. 

### Création de la passerelle de réseau privé virtuel et connexion
Dans la section suivante, vous ajoutez une passerelle de VPN et une connexion associée au sous-réseau avec l'instance VSI d'application. 

1. Accédez à la page [VPC - Présentation](https://{DomainName}/vpc/overview), puis cliquez sur **Réseaux VPN** dans l'onglet de navigation et sur **Nouvelle passerelle VPN** dans la boîte de dialogue. Dans le formulaire **Nouvelle passerelle VPN pour VPC**, entrez **vpns2s-gateway** en tant que nom. Assurez-vous que le VPC correct, le groupe de ressources et **vpns2s-cloud-subnet** en tant que sous-réseau sont sélectionnés.
2. Laissez l'option **Nouvelle connexion VPN pour VPC** activée. Entrez **vpns2s-gateway-conn** en tant que nom.
3. Pour l'**Adresse de passerelle homologue**, utilisez l'adresse IP flottante de **vpns2s-onprem-vsi** (ONPREM_IP). Entrez **20_PRESHARED_KEY_KEEP_SECRET_19** pour **Clé pré-partagée**.
4. Pour **Sous-réseaux locaux**, utilisez les informations fournies pour **CLOUD_CIDR**, pour **Sous-réseaux homologues**, utilisez les informations fournies pour **ONPREM_CIDR**.
5. Conservez les paramètres de **Détection d'homologue mort (DPD)** en l'état. Cliquez sur **Créer une passerelle VPN** pour créer la passerelle et une connexion associée.
6. Attendez que la passerelle VPN soit disponible (vous devrez peut-être actualiser l'écran). 
7. Notez l'adresse **IP de la passerelle** attribuée, à savoir **GW_CLOUD_IP**. 

### Création de la passerelle VPN sur site 
Vous créez à présent la passerelle VPN sur l’autre site, dans l’environnement simulé sur site. Vous utilisez le logiciel IPsec open source [strongSwan](https://strongswan.org/).

1. Connectez-vous à l'instance VSI "sur site" **vpns2s-onprem-vsi** à l'aide de ssh. Exécutez la procédure suivante et remplacez **ONPREM_IP** par l'adresse IP renvoyée précédemment.

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   Selon votre environnement, vous devrez peut-être utiliser `ssh -i <path to your private key file> root@ONPREMP_IP`.
   {:tip}

2. Ensuite, sur la machine **vpns2s-onprem-vsi**, exécutez les commandes suivantes pour mettre à jour le gestionnaire de packages et installer le logiciel strongSwan.

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. Configurez le fichier **/etc/sysctl.conf** en ajoutant trois lignes à la fin. Copiez les lignes suivantes et exécutez le fichier : 

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. Ensuite, éditez le fichier **/etc/ipsec.secrets**. Ajoutez la ligne suivante pour configurer les adresses IP source et de destination et la clé pré-partagée configurée précédemment. Remplacez **ONPREM_IP** par la valeur connue de l'adresse IP flottante de l'instance vpns2s-onprem-vsi. Remplacez **GW_CLOUD_IP** par l'adresse IP connue de la passerelle VPC VPN.

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. Le dernier fichier que vous devez configurer est **/etc/ipsec.conf**. Ajoutez le bloc de code suivant à la fin de ce fichier. Remplacez **ONPREM_IP**, **ONPREM_CIDR**, **GW_CLOUD_IP** et **CLOUD_CIDR** par les valeurs connues respectives.

   ```sh
   # basic configuration
   config setup
      charondebug="all"
      uniqueids=yes
      strictcrlpolicy=no
   
   # connection to vpc/vpn datacenter 
   # left=onprem / right=vpc
   conn tutorial-site2site-onprem-to-cloud
      authby=secret
      left=%defaultroute
      leftid=ONPREM_IP
      leftsubnet=ONPREM_CIDR
      right=GW_CLOUD_IP
      rightsubnet=CLOUD_CIDR
      ike=aes256-sha2_256-modp1024!
      esp=aes256-sha2_256!
      keyingtries=0
      ikelifetime=1h
      lifetime=8h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start
   ```
   {:pre}

6. Redémarrez la passerelle VPN, puis vérifiez son état en exécutant : ipsec restart

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   Un message devrait signaler qu'une connexion a été établie. Laissez ouverts le terminal et la connexion SSH à cette machine. 

## Test de la connectivité
Vous pouvez tester la connexion VPN de site à site à l’aide de SSH ou en déployant le microservice qui interface {{site.data.keyword.cos_short}}. 

### Test à l'aide de ssh
Pour vérifier que la connexion VPN a bien été établie, utilisez l'environnement sur site simulé en tant que proxy pour vous connecter au serveur d'applications dans le cloud.  

1. Dans un nouveau terminal, exécutez la commande suivante après avoir remplacé les valeurs. Elle utilise l'hôte strongSwan en tant qu'hôte de saut pour se connecter via un réseau privé virtuel à l'adresse IP privée du serveur d'applications. 

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. Une fois la connexion établie, fermez la connexion SSH. 

3. Dans le terminal de l'instance VSI sur site, arrêtez la passerelle VPN : 
   ```sh
   ipsec stop
   ```
   {:pre}
4. Dans la fenêtre de commande de l'étape 1), tentez de rétablir la connexion : 

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
La commande ne doit pas aboutir car la connexion VPN n'est pas active et, par conséquent, il n'existe pas de liaison directe entre les environnements sur site et cloud simulés. 

   Notez que, selon les détails du déploiement, cette connexion est tout de même établie. La raison en est que la connectivité intra-VPC est prise en charge entre les zones. Si vous souhaitez déployer la VSI sur site simulée sur un autre VPC ou sur [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers), le VPN est nécessaire pour permettre l'accès.
   {:tip}
   
5. Dans le terminal de l'instance VSI sur site, redémarrez la passerelle VPN : 
   ```sh
   ipsec start
   ```
   {:pre}
 

### Test à l'aide d'un microservice
Vous pouvez tester la connexion VPN active en accédant à un microservice sur l'instance VSI dans le cloud à partir de l'instance VSI sur site. 

1. Copiez le code de l'application de microservice de votre machine locale vers l'instance VSI dans le cloud. La commande utilise le bastion en tant qu'hôte de saut vers l'instance VSI dans le cloud. Remplacez **BASTION_IP_ADDRESS** et **VSI_CLOUD_IP** en conséquence.
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. Connectez-vous à l'instance VSI dans le cloud en utilisant à nouveau le bastion comme hôte de saut. 
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. Sur l'instance VSI dans le cloud, accédez au répertoire de code : 
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. Installez Python et le PIP du gestionnaire de packages Python. 
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. Installez les packages Python nécessaires à l’aide de **pip**.
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. Démarrez l'application :
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. Dans le terminal de l'instance VSI sur site, accédez au service. Remplacez VSI_CLOUD_IP en conséquence.
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
   La commande doit renvoyer un objet JSON.

## Suppression de ressources 
{: #remove-resources}

1. Dans la console de gestion du VPC, cliquez sur **Réseaux VPN**. Dans le menu d'actions de la passerelle VPN, sélectionnez **Supprimer** pour supprimer la passerelle.
2. Ensuite, cliquez sur **IP flottantes** dans le volet de navigation, puis sur l’adresse IP de vos instances VSI. Dans le menu d'actions, sélectionnez **Libérer**. Confirmez que vous souhaitez libérer l'adresse IP. 
3. Ensuite, accédez à **Instances de serveur virtuel** et **supprimez** vos instances. Les instances sont supprimées et leur statut **Suppression** est conservé pendant un certain temps. Veillez à actualiser le navigateur de temps en temps.
4. Une fois les instances VSI supprimées, passez aux **Sous-réseaux**. Si le sous-réseau est connecté à une passerelle publique, cliquez sur le nom du sous-réseau. Dans les détails du sous-réseau, déconnectez la passerelle publique. Les sous-réseaux sans passerelle publique peuvent être supprimés de la page de présentation. Supprimez les sous-réseaux. 
5. Une fois les sous-réseaux supprimés, basculez vers **VPC** et supprimez votre VPC. 

Lorsque vous utilisez la console, vous devrez peut-être actualiser votre navigateur pour consulter les informations d'état mises à jour après la suppression d'une ressource.
{:tip}

## Extension du tutoriel  
{: #expand-tutorial}

Vous souhaitez enrichir ou développer ce tutoriel ? Voici quelques suggestions :

- Ajoutez un [équilibreur de charge](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc) pour répartir le trafic de microservice entrant sur plusieurs instances.
- Déployez l'[application sur un serveur public, vos données et services sur un hôte privé](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend).


## Contenu associé
{: #related}

- [Glossaire de VPC ](/docs/vpc?topic=vpc-vpc-glossary)
- [Référence IBM Cloud CLI plugin for VPC](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [VPC utilisant les API REST](/docs/infrastructure/vpc/example-code.html)
- Tutoriel de solution : [Accès sécurisé aux instances distantes à l'aide d'un hôte bastion](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
