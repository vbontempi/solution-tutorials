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

# Utilisation de serveurs virtuels pour créer des applications Web évolutives à haute disponibilité 
{: #highly-available-and-scalable-web-application}

L'ajout de serveurs à une application est un modèle courant qui permet de gérer une charge supplémentaire. Un autre aspect essentiel pour augmenter la disponibilité et la résilience d'une application consiste à déployer l'application sur plusieurs zones ou emplacements avec réplication des données et équilibrage de la charge. 

Le présent tutoriel vous guide dans un scénario de création des éléments suivants : 

- Deux serveurs d'applications Web à Dallas. 
- Equilibreur de charge dans le cloud, pour équilibrer le trafic sur deux serveurs au même emplacement. 
- Un serveur de base de données MySQL. 
- Un emplacement de stockage de fichiers pérenne pour conserver les fichiers d'application et des sauvegardes. 
- Configurez le deuxième emplacement avec les premières configurations, puis ajoutez les services Cloud Internet Services (CIS) pour diriger le trafic vers l'emplacement sain si une copie échoue. 

## Objectifs
{: #objectives}

* Créer des {{site.data.keyword.virtualmachinesshort}} pour installer PHP et MySQL 
* Utiliser {{site.data.keyword.filestorage_short}} pour conserver les fichiers d'application et les sauvegardes de la base de données 
* Mettre en service {{site.data.keyword.loadbalancer_short}} pour répartir les demandes sur les serveurs d'applications 
* Etendre la solution en ajoutant un deuxième site pour une meilleure résilience et une disponibilité accrue 

## Services utilisés
{: #services}

Ce tutoriel utilise les environnements d’exécution et les services suivants : 
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

Ce tutoriel peut entraîner des coûts. Utilisez la [Calculatrice de prix](https://{DomainName}/pricing/) pour générer une estimation du coût en fonction de l'utilisation envisagée. 

## Architecture
{: #architecture}

L'application est une simple interface PHP - un blogue Wordpress - avec une base de données MySQL. Plusieurs serveurs de front-end traitent les demandes. 

<p style="text-align: center;">
  ![Diagramme d'architecture](images/solution14/Architecture.png)
</p>

1. L'utilisateur se connecte à l'application. 
2. {{site.data.keyword.loadbalancer_short}} sélectionne l'un des serveurs sains pour gérer la demande.
3. Le serveur sélectionné accède aux fichiers d'application stockés dans une mémoire de fichiers partagée. 
4. Le serveur extrait également des informations de la base de données et affiche la page à l'utilisateur. 
5. Le contenu de la base de données est sauvegardé à intervalles réguliers. Un serveur de base de données de secours est disponible en cas d'échec du serveur maître. 

## Avant de commencer
{: #prereqs}

### Configuration de l'accès au VPN 

Dans ce tutoriel, l'équilibreur de charge est la porte d'entrée des utilisateurs de l'application. Il n'est pas nécessaire que les {{site.data.keyword.virtualmachinesshort}} soient visibles sur le réseau Internet public. Ainsi, ils ne peuvent être mis à disposition qu'avec une adresse IP privée et vous utilisez votre connexion VPN pour travailler sur ces serveurs. 

1. [Assurez-vous que votre accès VPN est activé.](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

Vous devez être **Utilisateur principal** pour activer l'accès VPN ou contactez l'utilisateur principal pour l'accès.
{:tip}
2. Obtenez vos données d'identification d'accès VPN en sélectionnant votre utilisateur dans la [liste des utilisateurs](https://{DomainName}/iam#/users). 
3. Connectez-vous au VPN via [l'interface Web](https://www.softlayer.com/VPN-Access) ou utilisez un client VPN pour [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) ou [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

Vous pouvez choisir d'ignorer cette étape et de rendre tous vos serveurs visibles sur le réseau Internet public (bien que les garder privés offre un niveau de sécurité supplémentaire). Pour les rendre publics, sélectionnez **Liaisons montantes sur réseau public et privé** lors de la mise en service des {{site.data.keyword.virtualmachinesshort}}.
{: tip}

### Vérification des droits de compte 

Contactez votre utilisateur principal d'infrastructure pour obtenir les droits suivants :
- **Réseau** afin que vous puissiez créer des {{site.data.keyword.virtualmachinesshort}} avec des **Liaisons montantes sur réseau public et privé** (ce droit est requis uniquement si vous utilisez le VPN pour vous connecter aux serveurs)

## Mise à disposition d'un serveur pour la base de données 
{: #database_server}

Dans cette section, vous configurez un serveur en tant que base de données maître. 

1. Accédez au catalogue dans la console {{site.data.keyword.Bluemix}}, puis sélectionnez [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) dans la section Infrastructure.
2. Sélectionnez **Serveur virtuel public**, puis cliquez sur **Créer**.
3. Configurez le serveur avec les éléments suivants : 
   - Définissez **Nom** sur **db1**
   - Sélectionnez un emplacement où mettre le serveur à disposition. **Tous les autres serveurs et ressources créés dans le présent tutoriel devront être créés sur le même site.**
   - Sélectionnez l'image **Ubuntu Minimal**
   - Conservez la version de calcul par défaut. Ce tutoriel a été testé avec la version la plus faible, mais devrait fonctionner avec n’importe quelle version. 
   - Sous **Disques de stockage connectés**, sélectionnez le disque d'amorçage de 25 Go.
   - Sous **Interface réseau**, sélectionnez l'option **Liaison montante de 100 Mbits/s sur réseau privé**.

     Si vous n'avez pas configuré l'accès VPN, sélectionnez l'option **Liaisons montantes de 100 Mbits/s sur réseau public et privé**.
     {: tip}
   - Passez en revue les autres options de configuration et cliquez sur **Mise à disposition** pour mettre le serveur à disposition.

      ![Configuration du serveur virtuel](images/solution14/db-server.png)

   Remarque : le processus de mise à disposition peut prendre de 2 à 5 minutes pour que le serveur soit prêt à être utilisé. Une fois le serveur créé, vous trouverez les données d'identification du serveur dans la page de détail du serveur sous **Unités > Liste des unités**. Pour vous connecter au serveur via SSH, vous avez besoin de l'adresse IP privée ou publique du serveur, du nom d'utilisateur et du mot de passe (cliquez sur la flèche en regard du nom de l'unité).
   {: tip}

## Installation et configuration de MySQL
{: #mysql}

Le serveur n'est pas livré avec une base de données. Dans cette section, vous installez MySQL sur le serveur. 

### Installation de MySQL

1. Connectez-vous au serveur à l'aide de SSH : 
   ```sh
   ssh root@<Private-OR-Public-IP-Address>
   ```

   N'oubliez pas de vous connecter au client VPN avec [l'adresse de site](https://www.softlayer.com/VPN-Access) appropriée en fonction de l'**emplacement** de votre serveur virtuel.
   {:tip}
2. Installez MySQL :
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   Vous pouvez être invité à saisir un mot de passe. Lisez les instructions sur la console affichée.
{:tip}
3. Exécutez le script suivant pour sécuriser la base de données MySQL : 
   ```sh
   mysql_secure_installation
   ```

   Vous serez peut-être invité à entrer plusieurs options. Choisissez judicieusement en fonction de vos besoins.
{:tip}

### Création d'une base de données pour l'application 

1. Connectez-vous à MySQL et créez une base de données appelée `wordpress` :
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. Accordez l'accès à la base de données, remplacez database-username et database-password par les valeurs que vous avez définies précédemment. 

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. Vérifiez si la base de données a été créée à l'aide de la commande : 

   ```sh
   show databases;
   ```

4. Quittez la base de données à l'aide de la commande : 

   ```sh
   exit
   ```

5. Notez le nom de la base de données, l'utilisateur et le mot de passe. Vous en aurez besoin lors de la configuration des serveurs d'applications. 

### Visibilité du serveur MySQL pour les autres serveurs du réseau 

Par défaut, MySQL écoute uniquement sur l'interface locale. Les serveurs d'applications doivent se connecter à la base de données. Par conséquent, la configuration de MySQL doit être modifiée pour permettre l'écoute sur les interfaces du réseau privé. 

1. Editez le fichier my.cnf en utilisant `nano /etc/mysql/my.cnf` et ajoutez les lignes suivantes :
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. Quittez et enregistrez le fichier en utilisant Ctrl+X.

3. Redémarrez MySQL :

   ```sh
   systemctl restart mysql
   ```

4. Confirmez que MySQL écoute sur toutes les interfaces en exécutant la commande suivante : 
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## Création d'un stockage de fichiers pour les sauvegardes de base de données 
{: #database_backup}

Les sauvegardes peuvent être effectuées et stockées de nombreuses manières dans MySQL. Ce tutoriel utilise une entrée crontab pour vider le contenu de la base de données sur le disque. Les fichiers de sauvegarde sont archivés dans un stockage de fichiers. Bien entendu, il s’agit d’un mécanisme de sauvegarde simple. Si vous envisagez de gérer votre propre serveur de base de données MySQL dans un environnement de production, vous pouvez [mettre en oeuvre l'une des stratégies de sauvegarde décrites dans la documentation MySQL](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html). 

### Création du stockage de fichiers 
{: #create_for_backup}

1. Accédez au catalogue dans la console {{site.data.keyword.Bluemix}}, puis sélectionnez [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage).
2. Cliquez sur **Créer**
3. Configurez le service avec les éléments suivants : 
   - Définissez le **Type de stockage** sur **Endurance**
   - Sélectionnez le même **Emplacement** que celui où vous avez créé le serveur de base de données 
   - Sélectionnez une méthode de facturation 
   - Sous **Sélectionnez un package de stockage**, sélectionnez **0,25 IOPS/Go**
   - Sous **Taille de stockage**, sélectionnez **20 Go**
   - Conservez la valeur **Indiquer la taille de l'espace d'instantané** à **0 Go**
   - Cliquez sur Continuer pour créer le service. 

### Autorisation du serveur de base de données à utiliser le stockage de fichiers 

Pour qu'un serveur virtuel puisse monter un stockage de fichiers, il doit y être autorisé. 

1. Sélectionnez le nouveau stockage de fichiers dans la [liste des éléments existants](https://{DomainName}/classic/storage/file). 
2. Sous **Hôtes autorisés**, cliquez sur **Hôte autorisé** et sélectionnez le serveur (de base de données) virtuel (choisissez **Unités** > Serveur virtuel pour Type d'unité > Entrez le nom du serveur).

### Montage du stockage de fichiers pour les sauvegardes de base de données 

Le stockage de fichiers peut être monté en tant que lecteur NFS sur le serveur virtuel. 

1. Installez les bibliothèques client NFS : 
   ```sh
   apt-get -y install nfs-common
   ```

2. Créez un fichier appelé `/etc/systemd/system/mnt-database.mount` en exécutant la commande suivante : 
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. Editez le fichier mnt-database.mount à l'aide de la commande suivante :
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. Ajoutez le contenu ci-dessous au fichier mnt-database.mount et remplacez `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` de `What` par le **Point de montage** du stockage de fichiers (par exemple, *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). Vous pouvez obtenir l'URL du **Point de montage** sous le service de stockage de fichiers créé. 
   ```
   [Unit]
   Description = Mount for Container Storage

   [Mount]
   What=CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT
   Where=/mnt/database
   Type=nfs
   Options=vers=3,sec=sys,noauto

   [Install]
   WantedBy = multi-user.target
   ```
   Utilisez Ctrl + X pour enregistrer et quitter la fenêtre nano
   {: tip}

5. Créez le point de montage 
  ```sh
  mkdir /mnt/database
  ```

6. Montez le stockage 
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. Vérifiez que le montage a été correctement effectué 
   ```sh
   mount
   ```
Les dernières lignes doivent répertorier le montage du stockage de fichiers. Si tel n'est pas le cas, utilisez `journalctl -xe` pour déboguer l'opération de montage.
   {: tip}

### Configuration d'une sauvegarde à intervalles réguliers 

1. Créez le script shell `/root/dbbackup.sh` (utilisez `touch` et `nano`) à l'aide des commandes suivantes en remplaçant `CHANGE_ME` par le mot de passe de la base de données spécifié précédemment :
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip > /mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. Assurez-vous que le fichier est exécutable 
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. Editez le fichier crontab
   ```sh
   crontab -e
   ```
4. Pour que la sauvegarde soit effectuée tous les jours à 23 heures, définissez le contenu comme suit, enregistrez le fichier et fermez l'éditeur. 
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## Mise à disposition de deux serveurs pour l'application PHP 
{: #app_servers}

Dans cette section, vous créez deux serveurs d'applications Web. 

1. Accédez au catalogue dans la console {{site.data.keyword.Bluemix}}, puis sélectionnez le service [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) dans la section Infrastructure.
2. Sélectionnez **Serveur virtuel public**, puis cliquez sur **Créer**.
3. Configurez le serveur avec les éléments suivants : 
   - Définissez **Nom** sur **app1**
   - Sélectionnez l'emplacement où vous avez mis à disposition le serveur de base de données 
   - Sélectionnez l'image **Ubuntu Minimal**
   - Conservez la version de calcul par défaut. 
   - Sous **Disques de stockage connectés**, sélectionnez le disque d'amorçage de 25 Go.
   - Sous **Interface réseau**, sélectionnez l'option **Liaison montante de 100 Mbits/s sur réseau privé**.

     Si vous n'avez pas configuré l'accès VPN, sélectionnez l'option **Liaisons montantes de 100 Mbits/s sur réseau public et privé**.
     {: tip}
   - Passez en revue les autres options de configuration et cliquez sur **Mise à disposition** pour mettre le serveur à disposition.
 ![Configuration du serveur virtuel](images/solution14/db-server.png)

4. Répétez les étapes 1 à 3 pour mettre à disposition un autre serveur virtuel nommé **app2**

## Création d'un stockage de fichiers pour partager des fichiers entre les serveurs d'applications 
{: shared_storage}

Ce stockage de fichiers est utilisé pour partager les fichiers d'application entre les serveurs *app1* et *app2*. 

### Création du stockage de fichiers 
{: #create_for_sharing}

1. Accédez au catalogue dans la console {{site.data.keyword.Bluemix}}, puis sélectionnez [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage).
2. Cliquez sur **Créer**
3. Configurez le service avec les éléments suivants : 
   - Définissez le **Type de stockage** sur **Endurance**
   - Sélectionnez le même **Emplacement** que celui où vous avez créé les serveurs d'applications 
   - Sélectionnez une méthode de facturation 
   - Sous **Sélectionnez un package de stockage**, sélectionnez **2 IOPS/Go**
   - Sous **Taille de stockage**, sélectionnez **20 Go**
   - Sous **Indiquer la taille de l'espace d'instantané**, sélectionnez **20 Go**
   - Cliquez sur Continuer pour créer le service. 

### Configuration d'instantanés réguliers 

Les [Instantanés](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) vous offrent une option pratique pour protéger vos données sans impact sur les performances. De plus, vous pouvez répliquer des instantanés vers un autre centre de données. 

1. Sélectionnez le stockage de fichiers dans la [liste des éléments existants](https://{DomainName}/classic/storage/file). 
2. Sous **Plannings d'échantillonnage**, modifiez le planning d'échantillonnage. Le planning peut être défini comme suit : 
   1. Ajoutez un instantané toutes les heures, réglez les minutes sur 30 et conservez les 24 derniers instantanés. 
   2. Ajoutez un instantané quotidien, réglez l'heure sur 23 heures et conservez les 7 derniers instantanés. 
   3. Ajoutez un instantané hebdomadaire, réglez l'heure sur 1 heure du matin, conservez les 4 derniers instantanés et cliquez sur Sauvegarder.
     ![Instantanés de sauvegarde](images/solution14/snapshots.png)


### Autorisation des serveurs d'applications à utiliser le stockage de fichiers 

1. Sous **Hôtes autorisés**, cliquez sur **Hôte autorisé** pour autoriser les serveurs d'applications (app1 et app2) à utiliser ce stockage de fichiers.

### Montage du stockage de fichiers 

Répétez les étapes suivantes sur chaque serveur d'applications (app1 et app2) : 

1. Installez les bibliothèques client NFS 
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. Créez un fichier en utilisant `touch /etc/systemd/system/mnt-www.mount` et éditez-le à l'aide de `nano /etc/systemd/system/mnt-www.mount` avec le contenu suivant en remplaçant `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` de `What` par le **Point de montage** du stockage de fichiers (par exemple, *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). Les points de montage se trouvent dans la [liste des volumes de stockage de fichiers](https://{DomainName}/classic/storage/file) 
   ```
   [Unit]
   Description = Mount for Container Storage

   [Mount]
   What=CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT
   Where=/mnt/www
   Type=nfs
   Options=vers=3,sec=sys,noauto

   [Install]
   WantedBy = multi-user.target
   ```
3. Créez le point de montage 
   ```sh
   mkdir /mnt/www
   ```
4. Montez le stockage 
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. Vérifiez que le montage a été correctement effectué 
   ```sh
   mount
   ```
Les dernières lignes doivent répertorier le montage du stockage de fichiers. Si tel n'est pas le cas, utilisez `journalctl -xe` pour déboguer l'opération de montage.
   {: tip}

Enfin, toutes les étapes liées à la configuration des serveurs peuvent être automatisées à l'aide d'un [script de mise à disposition](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script) ou par la [capture d'une image](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template).
{: tip}

## Installation et configuration de l'application PHP sur les serveurs d'applications 
{: #php_application}

Dans ce tutoriel, vous allez mettre en place un blogue Wordpress. Tous les fichiers Wordpress sont installés sur le stockage de fichiers partagé afin que les deux serveurs d'applications puissent y accéder. Avant d'installer Wordpress, vous devez configurer un serveur Web et une exécution PHP. 

### Installation de nginx et PHP

Répétez les étapes suivantes sur chaque serveur d'applications : 

1. Installez nginx
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. Installez PHP et le client mysql 
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. Arrêtez le service PHP et nginx
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. Remplacez le contenu à l'aide de `nano /etc/nginx/sites-available/default` par les lignes suivantes :
   ```sh
   server {
          listen 80 default_server;
          listen [::]:80 default_server;

          root /mnt/www/html;

          index index.php;

          server_name _;

          location = /favicon.ico {
                  log_not_found off;
                  access_log off;
          }

          location = /robots.txt {
                  allow all;
                  log_not_found off;
                  access_log off;
          }

          location / {
                  # following https://codex.wordpress.org/Nginx
                  try_files $uri $uri/ /index.php?$args;
          }

          # pass the PHP scripts to the local FastCGI server
          location ~ \.php$ {
                  include snippets/fastcgi-php.conf;
                  fastcgi_pass unix:/run/php/php7.2-fpm.sock;
          }

          location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                  expires max;
                  log_not_found off;
          }

          # deny access to .htaccess files, if Apache's document root
          # concurs with nginx's one
          location ~ /\.ht {
                  deny all;
          }
   }
   ```
5. Créez un dossier `html` dans le répertoire `/mnt/www` sur l’un des deux serveurs d’application à l’aide des commandes suivantes : 
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### Installation et configuration de WordPress

Dans la mesure où Wordpress est installé sur le montage de stockage de fichiers, il vous suffit d'effectuer les étapes suivantes sur l’un des serveurs. Prenons le serveur **app1**.

1. Extrayez les fichiers d'installation de Wordpress 

   Si votre serveur d'applications dispose d'une liaison au réseau public, vous pouvez télécharger directement les fichiers Wordpress à partir du serveur virtuel : 

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   Si le serveur virtuel ne comporte qu'une liaison au réseau privé, vous devez récupérer les fichiers d'installation sur un autre poste disposant d'un accès Internet et les copier sur le serveur virtuel. En supposant que vous avez récupéré les fichiers d'installation de Wordpress à partir de https://wordpress.org/latest.tar.gz, vous pouvez les copier sur le serveur virtuel à l'aide de la commande `scp` :

   ```sh
   scp latest.tar.gz root@PRIVATE_IP_ADDRESS_OF_THE_SERVER:/tmp
   ```
   Remplacez `latest` par le nom de fichier que vous avez téléchargé à partir du site wordpress.
   {: tip}

   Puis connectez-vous au serveur virtuel via SSH et accédez au répertoire `tmp` 

   ```sh
   cd /tmp
   ```

2. Extrayez les fichiers d'installation

   ```
   tar xzvf latest.tar.gz
   ```

3. Préparez les fichiers Wordpress 
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. Copiez les fichiers sur le stockage de fichiers partagé 
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. Définissez les droits
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -exec chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. Appelez le service Web suivant et injectez le résultat dans `/mnt/www/html/wp-config.php` à l'aide de `nano`
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   Si votre serveur virtuel n'est pas relié au réseau public, vous pouvez simplement ouvrir https://api.wordpress.org/secret-key/1.1/salt/ à partir de votre navigateur Web.

7. Définissez les données d'identification de la base de données à l'aide de `nano /mnt/www/html/wp-config.php`, mettez à jour les données d'identification de la base de données :

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   Wordpress est configuré. Pour terminer l'installation, vous devez accéder à l'interface utilisateur de Wordpress. 

Sur les deux serveurs d'applications, démarrez le serveur Web et le moteur d'exécution PHP : 
7. Démarrez le service en lançant les commandes suivantes : 

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

Accédez à l'installation de Wordpress à l'adresse `http://YourAppServerIPAddress/` en utilisant l'adresse IP privée (si vous passez par la connexion VPN) ou l'adresse IP publique du serveur *app1* ou *app2*.
![Configuration du serveur virtuel](images/solution14/wordpress.png)

Si vous avez configuré les serveurs d'applications avec uniquement une liaison de réseau privé, vous ne pouvez pas installer de plugins, de thèmes ou de mises à niveau pour Wordpress directement à partir de la console d'administration Wordpress. Vous devez télécharger les fichiers via l'interface utilisateur Wordpress.
{: tip}

## Mise à disposition d'un serveur d'équilibrage de charge devant les serveurs d'applications 
{: #load_balancer}

A ce stade, nous disposons de deux serveurs d'applications avec des adresses IP distinctes. Ils risquent même de ne pas être visibles sur le réseau Internet public si vous choisissez de mettre à disposition uniquement la liaison montante de réseau privé. L'ajout d'un équilibreur de charge devant ces serveurs rend l'application publique. L'équilibreur de charge masque également l'infrastructure sous-jacente pour les utilisateurs. L'équilibreur de charge surveille la santé des serveurs d'applications et répartit les demandes entrantes vers des serveurs sains.

1. Accédez au catalogue pour créer un [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer)
2. A l'étape **Plan**, sélectionnez le même centre de données que pour les serveurs *app1* et *app2*
3. Dans **Paramètres réseau**,
   1. Sélectionnez le sous-réseau où les serveurs *app1* et *app2* ont été mis à disposition
   2. Utilisez le pool système IBM par défaut pour l'adresse IP publique de l'équilibreur de charge. 
4. Dans **De base**,
   1. Nommez l'équilibreur de charge, par exemple **app-lb-1**
   2. Conservez la configuration de protocole par défaut - par défaut, l’équilibreur de charge est configuré pour HTTP. Le protocole SSL est pris en charge avec vos propres certificats. Reportez-vous à [Import your SSL certificates in the load balancer](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates) (Importer vos certificats SSL dans l'équilibreur de charge)
      {: tip}
5. Dans **Instances de serveur**, ajoutez les serveurs *app1* et *app2* 
6. Passez en revue les valeurs et cliquez sur Créer pour terminer l’assistant. 

### Modification de la configuration de Wordpress pour utiliser l'URL de l'équilibreur de charge 

La configuration de Wordpress doit être modifiée pour utiliser l'adresse de l'équilibreur de charge. En effet, Wordpress conserve une référence à [l'URL du blogue et injecte cet emplacement dans les pages](https://codex.wordpress.org/Settings_General_Screen). Si vous ne modifiez pas ce paramètre, Wordpress redirige directement les utilisateurs vers les serveurs de back-end, évitant ainsi l'équilibreur de charge ou ne fonctionnant pas du tout si les serveurs disposent uniquement d'une adresse IP privée. 

1. Recherchez l'adresse de l'équilibreur de charge dans sa page de détails. L'équilibreur de charge que vous avez créé se trouve sous [Réseau / Equilibrage de charge / Local](https://{DomainName}/classic/network/loadbalancing/cloud).

   Vous pouvez également utiliser votre propre nom de domaine avec l'équilibreur de charge en ajoutant un enregistrement CNAME pointant vers l'adresse de l'équilibreur de charge dans votre configuration DNS.
   {: tip}
2. Connectez-vous en tant qu'administrateur dans le blogue Wordpress via l'URL *app1* ou *app2*.
3. Dans Paramètres / Général, définissez l'adresse Wordpress (URL) et l'adresse du site (URL) sur l'adresse de l'équilibreur de charge. 
4. Sauvegardez les paramètres. Wordpress devrait se rediriger vers l'adresse de l'équilibreur de charge.
Il peut s'écouler un certain temps avant que l'adresse de l'équilibreur de charge soit activée en raison de la propagation du DNS.
   {: tip}

### Test du comportement de l'équilibreur de charge 

L'équilibreur de charge est configuré pour vérifier l'intégrité des serveurs et rediriger les utilisateurs uniquement vers des serveurs sains. Pour comprendre le fonctionnement de l'équilibreur de charge, vous pouvez : 

1. Consultez les journaux nginx sur les deux serveurs *app1* et *app2* à l'aide de la commande :
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   Vous devriez voir les pings réguliers de l'équilibreur de charge pour vérifier la santé du serveur.
   {: tip}
2. Accédez à Wordpress via l'adresse de l'équilibreur de charge et veillez à forcer le rechargement de la page. Notez que dans les journaux nginx, *app1* et *app2* servent tous deux du contenu pour la page. L'équilibreur de charge redirige le trafic vers les deux serveurs comme prévu. 

3. Arrêtez nginx sur *app1*
   ```sh
   systemctl nginx stop
   ```

4. Après un court instant, rechargez la page Wordpress, notez que tous les hits vont à *app2*.

5. Arrêtez nginx sur *app2*.

6. Rechargez la page Wordpress. L'équilibreur de charge renvoie une erreur car il n'y a pas de serveur sain. 

7. Redémarrez nginx sur *app1*.
   ```sh
   systemctl nginx start
   ```

8. Une fois que l'équilibreur de charge détecte que *app1* est sain, il redirige le trafic vers ce serveur.

## Extension de la solution avec un deuxième emplacement (facultatif) 
{: #secondregion}

Pour augmenter la résilience et la disponibilité, vous pouvez étendre la configuration de l'infrastructure avec un deuxième emplacement et exécuter votre application sur deux emplacements. 

Avec un second déploiement d'emplacement, l'architecture se présente comme suit. 

<p style="text-align: center;">

  ![Diagramme d'architecture](images/solution14/Architecture2.png)
</p>

1. Les utilisateurs accèdent à l'application via IBM Cloud Internet Services (CIS). 
2. CIS achemine le trafic vers un endroit sain. 
3. Dans un emplacement, un équilibreur de charge redirige le trafic vers un serveur. 
4. L'application accède à la base de données. 
5. L'application stocke et récupère les ressources multimédias d'un stockage de fichiers. 

Pour implémenter cette architecture, vous devez effectuer les opérations suivantes à l'emplacement 2 : 

- Répétez toutes les étapes précédentes sur le nouvel emplacement. 
- Configurez une réplication de base de données entre les deux serveurs MySQL sur plusieurs emplacements. 
- Configurez IBM Cloud Internet Services pour répartir le trafic entre les emplacements sur des serveurs sains, comme décrit dans [cet autre tutoriel](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis). 

## Suppression de ressources 
{: #removeresources}

1. Supprimez l'équilibreur de charge 
2. Annulez *db1*, *app1* et *app2*
3. Supprimez les deux services de stockage de fichiers 
4. Si un deuxième emplacement est configuré, supprimez toutes les ressources et les services Cloud Internet Services. 

## Contenu associé
{: #related}

- Le contenu statique diffusé par votre application peut tirer parti d'un Réseau de diffusion de contenu (CDN) placé devant l'équilibreur de charge afin de réduire la charge sur vos serveurs de back-end. Reportez-vous à [Accélération de la distribution de fichiers statiques à l'aide d'Object Storage et CDN](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn) pour consulter un tutoriel mettant en oeuvre un Réseau de diffusion de contenu (CDN).
- Dans ce tutoriel, vous allez mettre à disposition deux serveurs. Plusieurs serveurs peuvent être ajoutés automatiquement pour gérer une charge supplémentaire. La [Mise à l'échelle auto](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale) vous permet d'automatiser le processus de redimensionnement manuel associé à l'ajout ou la suppression de serveurs virtuels pour prendre en charge vos applications métier. 
- Pour renforcer les options de disponibilité et de reprise après incident, le stockage de fichiers peut être configuré pour effectuer des [instantanés réguliers et automatiques](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) du contenu et la [réplication](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication) sur un autre centre de données. 
