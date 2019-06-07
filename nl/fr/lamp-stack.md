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

# Application Web PHP sur une pile LAMP 
{: #lamp-stack}

Ce tutoriel vous guide dans la création d'un serveur virtuel Ubuntu **L**inux avec un serveur Web **A**pache, une base de données **M**ySQL et des scripts **P**HP. Cette combinaison de logiciels - plus communément appelée pile LAMP - est très populaire et souvent utilisée pour créer des sites Web et des applications Web. A l'aide de {{site.data.keyword.BluVirtServers}}, vous déployez rapidement votre pile LAMP avec une surveillance et une analyse intégrées des vulnérabilités. Pour voir le serveur LAMP en action, vous devez installer et configurer le système de gestion de contenu [WordPress](https://wordpress.org/) gratuit et à code source ouvert. 

## Objectifs

* Mettre à disposition un serveur LAMP en quelques minutes 
* Appliquer la dernière version d'Apache, MySQL et PHP 
* Héberger un site Web ou un blogue en installant et configurant WordPress 
* Utiliser la surveillance pour détecter les pannes et les performances lentes 
* Evaluer les vulnérabilités et se protéger du trafic indésirable 

## Services utilisés

Ce tutoriel utilise les environnements d’exécution et les services suivants : 

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture

![Diagramme d'architecture](images/solution4/Architecture.png)

1. L'utilisateur final accède au serveur LAMP et aux applications à l'aide d'un navigateur Web 

## Avant de commencer

{: #prereqs}

1. Contactez votre administrateur d'infrastructure pour obtenir les droits suivants. 
  * Droit réseau requis pour établir la **Liaison montante sur réseau public et privé**

### Configuration de l'accès au VPN 

1. [Assurez-vous que votre accès VPN est activé.](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

Vous devez être **Utilisateur principal** pour activer l'accès VPN ou contactez l'utilisateur principal pour l'accès.
{:tip}
2. Obtenez vos données d'identification d'accès VPN dans [votre page utilisateur sous la liste des utilisateurs](https://{DomainName}/iam#/users).
3. Connectez-vous au VPN via [l'interface Web](https://www.softlayer.com/VPN-Access) ou utilisez un client VPN pour [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) ou [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Pour le client VPN, utilisez le nom de domaine complet d'un point d'accès unique de VPN de centre de données à partir de la [page d'accès Web VPN](https://www.softlayer.com/VPN-Access), au format *vpn.xxxnn.softlayer.com* en tant qu'adresse de passerelle.
{:tip}

## Création de services

Dans cette section, vous mettez à disposition un serveur virtuel public avec une configuration fixe. Le service {{site.data.keyword.BluVirtServers_short}} peut être déployé en quelques minutes à partir d'images de serveurs virtuels situés dans des emplacements géographiques spécifiques. Les serveurs virtuels répondent souvent aux pics de demande, après quoi ils peuvent être suspendus ou mis hors tension afin que l'environnement cloud s'adapter parfaitement à vos besoins d'infrastructure. 

1. Dans votre navigateur, accédez à la page du catalogue [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group).
2. Sélectionnez **Serveur virtuel public**, puis cliquez sur **Créer**.
3. Sous **Image**, sélectionnez la dernière version de**LAMP** sous **Ubuntu**. Même si la pile est préinstallée avec Apache, MySQL et PHP, vous réinstallez PHP et MySQL avec la dernière version. 
4. Sous **Interface réseau**, sélectionnez l'option **Liaison montante sur réseau public et privé**.
5. Passez en revue les autres options de configuration et cliquez sur **Mise à disposition** pour créer le serveur virtuel.
 ![Configuration du serveur virtuel](images/solution4/ConfigureVirtualServer.png)


Une fois le serveur créé, les données d'identification de connexion du serveur apparaissent. Bien que vous puissiez vous connecter via SSH à l'aide de l'adresse IP publique du serveur, il est recommandé d'accéder au serveur via le réseau privé et de désactiver l'accès SSH sur le réseau public. 

1. Effectuez [ces étapes](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network) pour sécuriser la machine virtuelle et désactiver l'accès SSH sur le réseau public.
1. A l'aide de vos nom d'utilisateur, mot de passe et adresse IP privée, connectez-vous au serveur avec SSH. 
   ```sh
   sudo ssh root@<Private-IP-Address>
   ```
   {: pre}

  L'adresse IP privée du serveur et son mot de passe se trouvent dans le tableau de bord.
{:tip}

  ![Serveur virtuel créé](images/solution4/VirtualServerCreated.png)

## Réinstallation d'Apache, MySQL et PHP 

Il est conseillé de mettre à jour régulièrement la pile LAMP avec les derniers correctifs de sécurité et correctifs de bogues. Dans cette section, vous exécutez des commandes pour mettre à jour les sources de package Ubuntu et réinstallez Apache, MySQL et PHP avec la dernière version. Notez l'accent circonflexe (^) à la fin de la commande.

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

Une autre option consiste à mettre à jour tous les packages à l'aide de la commande `sudo apt-get update && sudo apt-get dist-upgrade`.
{:tip}

## Vérification de l'installation et de la configuration 

Dans cette section, vous vérifiez qu'Apache, MySQL et PHP sont à jour et s'exécutent sur l'image Ubuntu. Vous implémentez également les paramètres de sécurité recommandés pour MySQL. 

1. Vérifiez Ubuntu en ouvrant l'adresse IP publique dans le navigateur. La page d'accueil Ubuntu devrait apparaître.
   ![Vérification d'Ubuntu](images/solution4/VerifyUbuntu.png)
2. Vérifiez que le port 80 est disponible pour le trafic Web en exécutant la commande suivante. 
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![Vérification du port](images/solution4/VerifyPort.png)
3. Passez en revue les versions d'Apache, MySQL et PHP installées à l’aide des commandes suivantes. 
  ```sh
  apache2 -v
  ```
  {: pre}
  ```sh
  mysql -V
  ```
  {: pre}
  ```sh
   php -v
  ```
   {: pre}
4. Exécutez le script suivant pour sécuriser la base de données MySQL. 
  ```sh
  mysql_secure_installation
  ```
  {: pre}
5. Entrez le mot de passe root MySQL et configurez les paramètres de sécurité pour votre environnement. Lorsque vous avez terminé, quittez l’invite mysql en entrant `\q`.
  ```sh
  mysql -u root -p
  ```
  {: pre}

  Le nom d'utilisateur et le mot de passe par défaut de MySQL sont root et root.
{:tip}
6. De plus, vous pouvez créer rapidement une page d’informations PHP avec la commande suivante. 
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. Affichez la page d’informations PHP que vous avez créée : ouvrez un navigateur et accédez à `http://{YourPublicIPAddress}/info.php`. Remplacez l'adresse IP publique de votre serveur virtuel. Elle devrait se présenter comme suit : 

![Informations PHP](images/solution4/PHPInfo.png)

### Installation et configuration de WordPress

Découvrez votre pile LAMP en installant une application. Les étapes suivantes installent la plateforme open source WordPress, qui est souvent utilisée pour créer des sites Web et des blogues. Pour plus d'informations et de paramètres pour l'installation de production, consultez la [documentation WordPress](https://codex.wordpress.org/Main_Page).

1. Exécutez la commande suivante pour installer WordPress. 
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. Configurez WordPress pour utiliser MySQL et PHP. Exécutez la commande suivante pour ouvrir un éditeur de texte et créer le fichier `/etc/wordpress/config-localhost.php`.
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. Copiez les lignes suivantes dans le fichier en remplaçant *yourPassword* par le mot de passe de votre base de données MySQL et en conservant les autres valeurs. Enregistrez et quittez le fichier en utilisant `Ctrl+X`.
   ```php
   <?php
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'wordpress');
   define('DB_PASSWORD', 'yourPassword');
   define('DB_HOST', 'localhost');
   define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
   ?>
   ```
   {: pre}
4. Dans un répertoire de travail, créez un fichier texte `wordpress.sql` pour configurer la base de données WordPress.
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. Ajoutez les commandes suivantes en remplaçant votre mot de passe par *yourPassword* et en conservant les autres valeurs. Puis, sauvegardez le fichier.
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. Exécutez la commande suivante pour créer la base de données :
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. Une fois la commande terminée, supprimez le fichier `wordpress.sql`. Déplacez l'installation WordPress vers la racine du document du serveur Web. 
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. Effectuez la configuration de WordPress et publiez sur la plateforme. Ouvrez un navigateur et accédez à `http://{yourVMPublicIPAddress}/wordpress`. Remplacez l'adresse IP publique de votre MV. Elle devrait se présenter comme suit.
   ![Site WordPress en cours d'exécution](images/solution4/WordPressSiteRunning.png)

## Configuration de domaine

Pour utiliser un nom de domaine existant avec votre serveur LAMP, mettez à jour l'enregistrement A pour qu'il pointe vers l'adresse IP publique du serveur virtuel. Vous pouvez visualiser l'adresse IP publique du serveur dans le tableau de bord. 

## Surveillance et utilisation du serveur 

Pour garantir la disponibilité du serveur et une expérience utilisateur optimale, la surveillance doit être activée sur chaque serveur de production. Dans cette section, vous explorez les options disponibles pour surveiller votre serveur virtuel et comprendre son utilisation à tout moment. 

### Surveillance du serveur

Deux types de surveillance de base sont disponibles : SERVICE PING et SLOW PING.

* **SERVICE PING** vérifie que le temps de réponse du serveur est égal à 1 seconde ou moins 
* **SLOW PING** vérifie que le temps de réponse du serveur est égal à 5 secondes ou moins

Dans la mesure où SERVICE PING est ajouté par défaut, ajoutez une surveillance SLOW PING en procédant comme suit. 

1. Dans le tableau de bord, sélectionnez votre serveur dans la liste des unités, puis cliquez sur l'onglet **Surveillance**.
  ![Surveillance Slow Ping](images/solution4/SlowPing.png)
2. Cliquez sur **Gérer les moniteurs**.
3. Ajoutez l'option de surveillance **SLOW PING** et cliquez sur **Ajouter moniteur**. Sélectionnez votre adresse IP publique pour l'adresse IP.
  ![Ajout de la surveillance Slow Ping](images/solution4/AddSlowPing.png)

  **Remarque** : les moniteurs en double avec les mêmes configurations ne sont pas autorisés. Un seul moniteur par configuration peut être créé. 

Si aucune réponse n'est reçue dans le délai imparti, une alerte est envoyée à l'adresse électronique du compte {{site.data.keyword.Bluemix_notm}}.
  ![Deux cycles de surveillance](images/solution4/TwoMonitoring.png)

### Utilisation du serveur

Sélectionnez l'onglet **Utilisation** pour afficher l'utilisation de la mémoire et du processeur du serveur actuel.
  ![Utilisation du serveur](images/solution4/ServerUsage.png)

## Sécurité de serveur

Le service {{site.data.keyword.BluVirtServers}} fournit plusieurs options de sécurité, telles que l'analyse de vulnérabilité et des pare-feux supplémentaires.

### Scanner de vulnérabilité

Le scanner de vulnérabilité recherche sur le serveur les vulnérabilités liées au serveur. Pour exécuter une analyse de vulnérabilité sur le serveur, effectuez les étapes ci-dessous. 

1. Dans le tableau de bord, sélectionnez votre serveur, puis cliquez sur l'onglet **Sécurité**.
2. Cliquez sur **Analyser** pour lancer l'analyse.
3. Une fois l'analyse terminée, cliquez sur **Fin de l'Analyse** pour afficher le rapport d'analyse.
  ![Deux cycles de surveillance](images/solution4/Vulnerabilities.png)
4. Passez en revue les vulnérabilités signalées.
  ![Deux cycles de surveillance](images/solution4/VulnerabilityResults.png)

### Pare-feux

Un autre moyen de sécuriser le serveur consiste à ajouter un pare-feu. Les pare-feux constituent une couche de sécurité essentielle : empêcher le trafic indésirable de contacter vos serveurs, réduire le risque d'attaque et consacrer les ressources de votre serveur à l'usage auquel elles sont destinées. Les options de pare-feu sont mises à disposition à la demande, sans interruption de service. 

Des pare-feux sont disponibles en tant que fonction complémentaire pour tous les serveurs sur le réseau public Infrastructure. Dans le cadre du processus de commande, vous pouvez sélectionner un pare-feu matériel ou logiciel spécifique au terminal afin d'assurer votre protection. Vous pouvez également déployer des dispositifs de pare-feu dédiés et déployer le serveur virtuel sur un réseau VLAN protégé. Pour plus d'informations, voir [Pare-feux](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started).

## Suppression de ressources 

Pour supprimer votre serveur virtuel, procédez comme suit. 

1. Connectez-vous à [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices).
2. Dans le menu **Unités**, sélectionnez **Liste d'unités**.
3. Cliquez sur **Actions** pour le serveur virtuel que vous souhaitez supprimer et sélectionnez **Annuler**.

## Contenu associé

* [Déploiement d'une pile LAMP à l'aide de Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
