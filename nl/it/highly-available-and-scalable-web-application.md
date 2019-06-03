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

# Utilizza i server virtuali per creare un'applicazione web altamente disponibile e scalabile
{: #highly-available-and-scalable-web-application}

L'aggiunta di ulteriori server a un'applicazione è un pattern comune per gestire del carico aggiuntivo. Un altro aspetto chiave per aumentare la disponibilità e la resilienza di un'applicazione consiste nel distribuire l'applicazione su più zone o ubicazioni con la replica dei dati e il bilanciamento del carico.

Questa esercitazione ti guida in uno scenario con la creazione di:

- Due server delle applicazioni web a Dallas.
- Un programma di bilanciamento del carico cloud, per bilanciare il carico del traffico tra due server in un'ubicazione.
- Un server di database MySQL.
- Un'archiviazione file durevole per archiviare i file di applicazione e i backup.
- Configura la seconda ubicazione con le stesse configurazioni della prima e aggiungi quindi Cloud Internet Services per puntare il traffico all'ubicazione integra se si verifica un malfunzionamento di una copia.

## Obiettivi
{: #objectives}

* Creare {{site.data.keyword.virtualmachinesshort}} per installare PHP e MySQL
* Utilizzare {{site.data.keyword.filestorage_short}} per archiviare in modo persistente i file delle applicazioni e i backup del database.
* Eseguire il provisioning di un {{site.data.keyword.loadbalancer_short}} per distribuire le richieste ai server delle applicazioni
* Estendere la soluzione aggiungendo una seconda ubicazione per una migliore resilienza e una disponibilità più elevata

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

L'applicazione è un semplice frontend PHP - un blog Wordpress - con un database MySQL. Diversi server di frontend gestiscono le richieste.

<p style="text-align: center;">
  ![Diagramma dell'architettura](images/solution14/Architecture.png)
</p>

1. L'utente si connette all'applicazione.
2. Il {{site.data.keyword.loadbalancer_short}} seleziona uno dei server integri per gestire la richiesta.
3. Il server scelto accede ai file delle applicazioni archiviati su un'archiviazione file condivisa.
4. Il server estrae anche le informazioni dal database e, infine, riproduce la pagina per l'utente.
5. Viene eseguito il backup del contenuto del database a intervalli regolari. Un server di database di stand-by è disponibile nel caso in cui si verifichi un malfunzionamento del master.

## Prima di iniziare
{: #prereqs}

### Configura l'accesso VPN

In questa esercitazione, il programma di bilanciamento del carico è il punto di ingresso per gli utenti dell'applicazione. Non è necessario che il {{site.data.keyword.virtualmachinesshort}} sia visibile su Internet pubblico. Ne può pertanto essere eseguito il provisioning solo con un indirizzo IP privato e utilizzerai la tua connessione VPN per lavorare sui server.

1. [Assicurati che il tuo accesso VPN sia abilitato](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Devi essere un utente master (**Master User**) per abilitare l'accesso VPN o contattare l'utente master per l'accesso.
     {:tip}
2. Ottieni le tue credenziali di accesso VPN selezionando il tuo utente nell'elenco utenti ([Users list](https://{DomainName}/iam#/users)).
3. Esegui l'accesso alla VPN mediante [l'interfaccia web](https://www.softlayer.com/VPN-Access) oppure utilizza un client VPN per [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) o [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

Puoi scegliere di tralasciare questo passo e rendere tutti i tuoi server visibili su Internet pubblico (anche se mantenerli privati fornisce un livello di sicurezza aggiuntivo). Per renderli pubblici, seleziona **Public and Private Network Uplink** quando esegui il provisioning di {{site.data.keyword.virtualmachinesshort}}.
{: tip}

### Controlla le autorizzazioni dell'account

Contatta il tuo utente master dell'infrastruttura per ottenere le seguenti autorizzazioni:
- **Rete** in modo che tu possa creare {{site.data.keyword.virtualmachinesshort}} con **Public and Private Network Uplink** (questa autorizzazione non è necessaria se utilizzi la VPN per stabilire una connessione ai server)

## Esegui il provisioning di un server per il database
{: #database_server}

In questa sezione, configuri un server perché funga da database master.

1. Vai al catalogo nella console {{site.data.keyword.Bluemix}} e seleziona [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) dalla sezione Infrastructure.
2. Seleziona **Public Virtual Server** e fai quindi clic su **Create**.
3. Configura il server con quanto segue:
   - Imposta **Name** su **db1**
   - Seleziona un'ubicazione dove eseguire il provisioning del server. **Tutti gli altri server e tutte le altre risorse creati in questa esercitazione dovranno essere creati nella stessa ubicazione.**
   - Seleziona l'immagine **Ubuntu Minimal**
   - Mantieni il profilo di calcolo predefinito. L'esercitazione è stata verificata con il profilo più piccolo ma dovrebbe funzionare con qualsiasi profilo.
   - In **Attached Storage Disks**, seleziona il disco di avvio da 25GB.
   - In **Network Interface**, seleziona l'opzione **100Mbps Private Network Uplink**.

     Se non hai configurato l'accesso VPN, seleziona l'opzione **100Mbps Public and Private Network Uplink**.
     {: tip}
   - Esamina le altre opzioni di configurazione e fai clic su **Provision** per eseguire il provisioning del server.

      ![Configura il server virtuale](images/solution14/db-server.png)

   Nota: il processo di provisioning può richiedere dai 2 ai 5 minuti perché il server sia pronto per l'uso. Dopo che il server è stato creato, ne troverai le credenziali nella relativa pagina dei dettagli in **Devices > Device List**. Per eseguire l'SSH nel server, ti serve l'indirizzo IP privato e pubblico, il nome utente e la password del server (fai clic sulla freccia accanto al nome del dispositivo).
   {: tip}

## Installa e configura MySQL
{: #mysql}

Il server non viene fornito con un database. In questa sezione, installi MySQL sul server.

### Installa MySQL

1. Stabilisci una connessione al server utilizzando SSH:
   ```sh
   ssh root@<Private-OR-Public-IP-Address>
   ```

   Ricordati di stabilire una connessione al client VPN con l'[indirizzo del sito](https://www.softlayer.com/VPN-Access) corretto basato sull'ubicazione (**Location**) del tuo server virtuale.
   {:tip}
2. Installa MySQL:
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   Ti potrebbe essere richiesta una password. Leggi con attenzione le istruzioni sulla console visualizzata.
   {:tip}
3. Esegui questo script per contribuire a proteggere il database MySQL:
   ```sh
   mysql_secure_installation
   ```

   Ti potrebbero essere richieste un paio di opzioni. Scegli con attenzione in base ai tuoi requisiti.
   {:tip}

### Crea un database per l'applicazione

1. Esegui l'accesso a MySQL e crea un database denominato `wordpress`:
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. Concedi l'accesso al database, sostituisci database-username e database-password con quanto hai impostato in precedenza.

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. Controlla se il database è stato creato utilizzando:

   ```sh
   show databases;
   ```

4. Esci dal database utilizzando:

   ```sh
   exit
   ```

5. Prendi nota del nome, dell'utente e della password del database. Ti serviranno quando configurerai i server delle applicazioni.

### Rendi il server MySQL visibile agli altri server sulla rete

Per impostazione predefinita, MySQL è in ascolto solo sull'interfaccia locale. I server delle applicazioni avranno bisogno di connettersi al database; pertanto, la configurazione MySQL deve essere modificata per ascoltare sulle interfacce di rete private.

1. Modifica il file my.cnf utilizzando `nano /etc/mysql/my.cnf` e aggiungi queste righe:
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. Esci e salva il file utilizzando Ctrl+X.

3. Riavvia MySQL:

   ```sh
   systemctl restart mysql
   ```

4. Conferma che MySQL sia in ascolto su tutte le interfacce eseguendo questo comando:
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## Crea un archivio file per i backup del database
{: #database_backup}

Quando si parla di MySQL, ci sono molti modi in cui è possibile eseguire e archiviare dei backup. Questa esercitazione utilizza una voce crontab per eseguire il dump del contenuto del database su disco. I file di backup verranno archiviati in un'archiviazione file. Ovviamente, questo è un semplice meccanismo di backup. Se intendi gestire il tuo server di database MySQL in un ambiente di produzione, vorrai [implementare una delle strategie di backup descritte nella documentazione di MySQL](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html).

### Crea l'archiviazione file
{: #create_for_backup}

1. Vai al catalogo nella console {{site.data.keyword.Bluemix}} e seleziona [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
2. Fai clic su **Create**
3. Configura il servizio con quanto segue:
   - Imposta **Storage Type** su **Endurance**
   - Seleziona un'ubicazione (**Location**) uguale a quella dove hai creato il server del database
   - Seleziona un metodo di fatturazione
   - In **Storage Packages**, seleziona **0.25 IOPS/GB**
   - In **Storage Size**, seleziona **20GB**
   - Mantieni **Snapshot Space Size** su **0GB**
   - Fai clic su Continue per creare il servizio.

### Autorizza il server di database per utilizzare l'archiviazione file

Prima che un server virtuale possa montare una File Storage (archiviazione file), è necessario che venga autorizzata.

1. Seleziona la File Storage (archiviazione file) appena creata dall'[elenco di elementi esistenti](https://{DomainName}/classic/storage/file).
2. In **Authorized Hosts**, fai clic su **Authorize Host** e seleziona il server (di database) virtuale (scegli **Devices** > Virtual Server as Device Type > Immetti il nome del server).

### Monta l'archiviazione file per i backup del database

La File Storage (archiviazione file) può essere montata come un'unità NFS nel server virtuale.

1. Installa le librerie del client NFS:
   ```sh
   apt-get -y install nfs-common
   ```

2. Crea un file denominato `/etc/systemd/system/mnt-database.mount` eseguendo questo comando
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. Modifica mnt-database.mount utilizzando:
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. Aggiungi il contenuto riportato di seguito al file mnt-database.mount e sostituisci `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` di `What` con il punto di montaggio (**Mount Point**) dell'archiviazione file (ad es. *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). Puoi ottenere l'URL del punto di montaggio (**Mount Point**) nel servizio di archiviazione file creato.
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
   Utilizza Ctrl+X per salvare e uscire dalla finestra di nano
   {: tip}

5. Crea il punto di montaggio
  ```sh
  mkdir /mnt/database
  ```

6. Monta l'archiviazione
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. Controlla se il montaggio è stato eseguito correttamente
   ```sh
   mount
   ```
   Le ultime righe dovrebbero elencare il montaggio della File Storage (archiviazione file). Se non è questo il caso, usa `journalctl -xe` per eseguire il debug dell'operazione di montaggio.
   {: tip}

### Configura un backup a intervalli regolari

1. Crea lo script shell `/root/dbbackup.sh` (utilizza `touch` e `nano`) con questi comandi sostituendo `CHANGE_ME` con la password del database da te specificata in precedenza.
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip > /mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. Assicurati che il file sia eseguibile
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. Modifica il crontab
   ```sh
   crontab -e
   ```
4. Per fare in modo che il backup venga eseguito ogni giorno alle 23, imposta il contenuto su quanto di seguito indicato, salva il file e chiudi l'editor.
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## Esegui il provisioning di due server per l'applicazione PHP
{: #app_servers}

In questa sezione, creerai due server delle applicazioni web.

1. Vai al catalogo nella console {{site.data.keyword.Bluemix}} e seleziona il servizio [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) dalla sezione Infrastructure.
2. Seleziona **Public Virtual Server** e fai quindi clic su **Create**.
3. Configura il server con quanto segue:
   - Imposta **Name** su **app1**
   - Seleziona la stessa ubicazione in cui hai eseguito il provisioning del server di database
   - Seleziona l'immagine **Ubuntu Minimal**
   - Mantieni il profilo di calcolo predefinito.
   - In **Attached Storage Disks**, seleziona 25GB come tuo disco di avvio.
   - In **Network Interface**, seleziona l'opzione **100Mbps Private Network Uplink**.

     Se non hai configurato l'accesso VPN, seleziona l'opzione **100Mbps Public and Private Network Uplink**.
     {: tip}
   - Esamina le altre opzioni di configurazione e fai clic su **Provision** per eseguire il provisioning del server.
     ![Configura il server virtuale](images/solution14/db-server.png)
4. Ripeti i passi da 1 a 3 per eseguire il provisioning di un altro server virtuale denominato **app2**

## Crea un'archiviazione file per condividere i file tra i server delle applicazioni
{: shared_storage}

Questa archiviazione file viene utilizzata per condividere i file delle applicazioni tra i server *app1* e *app2*.

### Crea l'archiviazione file
{: #create_for_sharing}

1. Vai al catalogo nella console {{site.data.keyword.Bluemix}} e seleziona [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
2. Fai clic su **Create**
3. Configura il servizio con quanto segue:
   - Imposta **Storage Type** su **Endurance**
   - Imposta un'ubicazione (**Location**) uguale a quella dove hai creato i server delle applicazioni
   - Seleziona un metodo di fatturazione
   - In **Storage Packages**, seleziona **2 IOPS/GB**
   - In **Storage Size**, seleziona **20GB**
   - In **Snapshot Space Size**, seleziona **20GB**
   - Fai clic su Continue per creare il servizio.

### Configura istantanee regolari

Le [istantanee](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) ti offrono una pratica opzione per proteggere i tuoi dati senza alcuna ripercussione sulle prestazioni. Puoi inoltre replicare le istantanee in un altro data center.

1. Seleziona l'archiviazione file (File Storage (archiviazione file)) dall'[elenco di elementi esistenti](https://{DomainName}/classic/storage/file)
2. In **Snapshot Schedules**, modifica la pianificazione dell'istantanea. La pianificazione può essere definita nel seguente modo:
   1. Aggiungi un'istantanea oraria, imposta il minuto su 30 e conserva le ultime 24 istantanee
   2. Aggiungi un'istantanea giornaliera, imposta l'ora sulle 23 e conserva le ultime 7 istantanee
   3. Aggiungi un'istantanea settimanale, imposta l'ora sulle 13 e conserva le ultime 4 istantanee e fai clic su Save.
      ![Istantanee di backup](images/solution14/snapshots.png)

### Autorizza i server delle applicazioni a utilizza l'archiviazione file

1. In **Authorized Hosts**, fai clic su **Authorize Host** per autorizzare i server delle applicazioni (app1 e app2) a utilizzare questa archiviazione file.

### Monta l'archiviazione file

Ripeti i seguenti passi su ciascun server delle applicazioni (app1 e app2):

1. Installa le librerie del client NFS
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. Crea un file utilizzando `touch /etc/systemd/system/mnt-www.mount` e procedi alla modifica utilizzando `nano /etc/systemd/system/mnt-www.mount` con il seguente contenuto sostituendo `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` di `What` con il punto di montaggio (**Mount Point**) per l'archiviazione file (ad es. *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). Puoi trovare i punti di montaggio nell'[elenco di volumi di archiviazione file](https://{DomainName}/classic/storage/file)
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
3. Crea il punto di montaggio
   ```sh
   mkdir /mnt/www
   ```
4. Monta l'archiviazione
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. Controlla se il montaggio è stato eseguito correttamente
   ```sh
   mount
   ```
   Le ultime righe dovrebbero elencare il montaggio della File Storage (archiviazione file). Se non è questo il caso, usa `journalctl -xe` per eseguire il debug dell'operazione di montaggio.
   {: tip}

Alla fine, tutti i passi correlati alla configurazione dei server potrebbero essere automatizzati utilizzando uno [script di provisioning](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script) oppure [acquisendo un'immagine](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template).
{: tip}

## Installa e configura l'applicazione PHP sui server delle applicazioni
{: #php_application}

Questa esercitazione imposta un blog Wordpress. Tutti i file Wordpress verranno installati nell'archiviazione file condivisa in modo che entrambi i server delle applicazioni possano accedervi. Prima di installare Wordpress, devono essere configurati un server web e un runtime PHP.

### Installa nginx e PHP

Ripeti la seguente procedura su ciascun server delle applicazioni:

1. Installa nginx
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. Installa il client PHP e mysql
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. Arresta il servizio PHP e nginx
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. Sostituisci il contenuto utilizzando `nano /etc/nginx/sites-available/default` con quanto segue:
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
5. Crea una cartella `html` all'interno della directory `/mnt/www` su uno dei due server delle applicazioni utilizzando
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### Installa e configura WordPress

Poiché Wordpress verrà installato sul montaggio della File Storage (archiviazione file), dovrai eseguire questi passi solo su uno dei server. Scegliamo **app1**.

1. Richiama i file di installazione di Wordpress

   Se il tuo server delle applicazioni ha un collegamento alla rete pubblica, puoi scaricare direttamente i file Wordpress dall'interno del server virtuale.

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   Se il server virtuale ha solo un collegamento alla rete privata, dovrai richiamare i file di installazione da un'altra macchina con accesso a Internet e copiarli quindi sul server virtuale. Presumendo che tu abbia richiamato i file di installazione Wordpress da https://wordpress.org/latest.tar.gz, puoi eseguirne la copia sul server virtuale con `scp`:

   ```sh
   scp latest.tar.gz root@PRIVATE_IP_ADDRESS_OF_THE_SERVER:/tmp
   ```
   Sostituisci `latest` con il nome file che hai scaricato dal sito web di wordpress.
   {: tip}

   ed esegui quindi l'ssh al server virtuale e passa alla directory `tmp`

   ```sh
   cd /tmp
   ```

2. Estrai i file di installazione

   ```
   tar xzvf latest.tar.gz
   ```

3. Prepara i file Wordpress
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. Copia i file nell'archiviazione file condivisa
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. Imposta le autorizzazioni
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -exec chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. Richiama il seguente servizio web e inserisci il risultato in `/mnt/www/html/wp-config.php` utilizzando `nano`
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   Se il tuo server virtuale non ha un collegamento alla rete pubblica, puoi semplicemente aprire https://api.wordpress.org/secret-key/1.1/salt/ dal tuo browser web.

7. Imposta le credenziali del database utilizzando `nano /mnt/www/html/wp-config.php`; aggiorna le credenziali del database.

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   Wordpress è configurato. Per completare l'installazione, devi accedere all'interfaccia utente Wordpress.

Su entrambi i server delle applicazioni, avvia il server web e il runtime PHP.
7. Avvia il servizio eseguendo i seguenti comandi

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

Accedi all'installazione Wordpress in `http://YourAppServerIPAddress/` utilizzando l'indirizzo IP privato (se ti servirai della connessione VPN) oppure l'indirizzo IP pubblico di *app1* or *app2*.
![Configura il server virtuale](images/solution14/wordpress.png)

Se hai configurato i server delle applicazioni solo con un collegamento alla rete privata, non sarai in grado di installare i plugin, i temi o gli upgrade di Wordpress direttamente dalla console di gestione di Wordpress. Dovrai caricare i file tramite l'interfaccia utente Wordpress.
{: tip}

## Esegui il provisioning di un server del programma di bilanciamento del carico davanti ai server delle applicazioni
{: #load_balancer}

A questo punto, abbiamo due server delle applicazioni con indirizzi IP separati. Potrebbero anche non essere visibili su Internet pubblico se scegli di eseguire il provisioning solo di un uplink di rete privata. L'aggiunta di un programma di bilanciamento del carico davanti a questi server renderà l'applicazione pubblica. Il programma di bilanciamento del carico nasconderà anche l'infrastruttura sottostante agli utenti. Il programma di bilanciamento del carico monitorerà l'integrità dei server delle applicazioni e invierà le richieste in entrata ai server integri.

1. Vai al catalogo per creare un [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer)
2. Nel passo **Plan**, seleziona lo stesso data center di *app1* e *app2*
3. In **Network Settings**,
   1. Seleziona la stessa sottorete dove era stato eseguito il provisioning di *app1* e *app2*
   2. Utilizza il pool di sistema IBM predefinito per l'IP pubblico del programma di bilanciamento del carico.
4. In **Basic**,
   1. Denomina il programma di bilanciamento del carico, ad esempio **app-lb-1**
   2. Mantieni la configurazione del protocollo predefinita; per impostazione predefinita, il programma di bilanciamento del carico è configurato per HTTP.
      Il protocollo SSL è supportato con i tuoi certificati. Consulta [Import your SSL certificates in the load balancer](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates)
      {: tip}
5. In **Server Instances**, aggiungi i server *app1* e *app2*
6. Esamina e crea per completare la procedura guidata.

### Modifica la configurazione Wordpress per utilizzare l'URL del programma di bilanciamento del carico

La configurazione di Wordpress deve essere modificata per utilizzare l'indirizzo del programma di bilanciamento del carico. In effetti, Wordpress mantiene un riferimento all'[URL del blog e inserisce questa ubicazione nelle pagine](https://codex.wordpress.org/Settings_General_Screen). Se non modifichi questa impostazione, Wordpress reindirizzerà gli utenti direttamente ai server di backend, ignorando quindi il programma di bilanciamento del carico o non funzionando affatto se i server hanno solo un indirizzo IP privato.

1. Trova l'indirizzo del programma di bilanciamento del carico nella relativa pagina dei dettagli. Puoi trovare il programma di bilanciamento del carico che hai creato in [Network / Load Balancing / Local](https://{DomainName}/classic/network/loadbalancing/cloud).

   Puoi anche utilizzare il tuo nome dominio con il programma di bilanciamento del carico aggiungendo un record CNAME che punta all'indirizzo del programma di bilanciamento del carico nella tua configurazione DNS.
   {: tip}
2. Accedi come amministratore nel blog Wordpress tramite l'URL *app1* o *app2*
3. In Settings / General, imposta sia Wordpress Address (URL) che Site Address (URL) sull'indirizzo del programma di bilanciamento del carico
4. Salva le impostazioni. Wordpress dovrebbe eseguire il reindirizzamento al programma di bilanciamento del carico
   Potrebbe volerci un po' di tempo prima che l'indirizzo del programma di bilanciamento del carico diventi attivo a causa della propagazione DNS.
   {: tip}

### Verifica la modalità di funzionamento del programma di bilanciamento del carico

Il programma di bilanciamento del carico è configurato per controllare l'integrità dei server e per reindirizzare gli utenti solo ai server integri. Per comprendere la modalità di funzionamento del programma di bilanciamento del carico, puoi

1. Consultare i log nginx sia su *app1* che su *app2* con:
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   Dovresti già vedere dei ping regolari dal programma di bilanciamento del carico per controllare l'integrità dei server.
   {: tip}
2. Accedi a Wordpress tramite l'indirizzo del programma di bilanciamento del carico e assicurarti di imporre un ricaricamento forzato della pagina. Nei log nginx, nota che sia *app1* e *app2* stanno servendo contenuto per la pagina. Il programma di bilanciamento del carico sta reindirizzando il traffico a entrambi i server come previsto.

3. Arresta nginx su *app1*
   ```sh
   systemctl nginx stop
   ```

4. Dopo poco, ricarica la pagina Wordpress, nota che tutti i riscontri stanno andando a *app2*.

5. Arresta nginx su *app2*.

6. Ricarica la pagina Wordpress. Il programma di bilanciamento del carico restituirà un errore poiché non c'è alcun server integro.

7. Riavvia nginx su *app1*
   ```sh
   systemctl nginx start
   ```

8. Dopo che avrà rilevato *app1* come integro, il programma di bilanciamento del carico reindirizzerà il traffico a questo server.

## Estendi la soluzione con una seconda ubicazione (facoltativo)
{: #secondregion}

Per aumentare resilienza e disponibilità, puoi estendere la configurazione dell'infrastruttura con una seconda ubicazione e fare in modo che la tua applicazione sia eseguita in due ubicazioni.

Con la distribuzione di una seconda ubicazione, l'architettura sarà simile alla seguente:

<p style="text-align: center;">

  ![Diagramma dell'architettura](images/solution14/Architecture2.png)
</p>

1. Gli utenti accedono all'applicazione tramite IBM CIS (Cloud Internet Services).
2. CIS instrada il traffico a un'ubicazione integra.
3. All'interno di un'ubicazione, un programma di bilanciamento del carico reindirizza il traffico a un server.
4. L'applicazione accede al database.
5. L'applicazione archivia e richiama le risorse di supporto da un'archiviazione file.

Per implementare questa architettura, devi eseguire le seguenti operazioni nell'ubicazione numero due:

- Ripeti tutti i passi precedenti sopra indicati nella nuova ubicazione.
- Configura una replica del database tra i due server MySQL nelle ubicazioni.
- Configura IBM Cloud Internet Services per distribuire il traffico tra le ubicazioni ai server integri, come descritto in[quest'altra esercitazione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis).

## Rimuovi le risorse
{: #removeresources}

1. Elimina il programma di bilanciamento del carico
2. Annulla *db1*, *app1* e *app2*
3. Elimina i due servizi di File Storage (archiviazione file)
4. Se è configurata una seconda ubicazione, elimina tutte le risorse e Cloud Internet Services.

## Contenuto correlato
{: #related}

- Il contenuto statico servito dalla tua applicazione potrebbe trarre vantaggio da una CDN (Content Delivery Network) davanti al programma di bilanciamento del carico per ridurre il carico sui tuoi server di backend. Consulta [Accelera la fornitura di file statici utilizzando una CDN - Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn) per un'esercitazione che implementa una CDN Content Delivery Network).
- In questa esercitazione, eseguiamo il provisioning di due server; è possibile aggiungere ulteriori server in modo automatico per gestire del carico aggiuntivo. Il [ridimensionamento automatico](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale) ti consente di automatizzare il processo di ridimensionamento manuale associato all'aggiunta o alla rimozione di server virtuali per supportare le tue applicazioni aziendali.
- Per aumentare le opzioni di disponibilità e ripristino di emergenza, è possibile configurare la File Storage (archiviazione file) per eseguire [istantanee regolari automatiche](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) del contenuto e la [replica](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication) su un altro data center.
