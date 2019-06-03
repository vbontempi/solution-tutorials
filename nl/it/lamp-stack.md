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

# Applicazione web PHP su uno Stack LAMP
{: #lamp-stack}

Questa esercitazione ti guida nella creazione di un server virtuale Ubuntu **L**inux con il server web **A**pache, il database **M**ySQL e la creazione di script **P**HP. Questa combinazione di software, più comunemente nota come stack LAMP, è molto diffusa e viene spesso utilizzata per fornire siti web e applicazioni web. Utilizzando {{site.data.keyword.BluVirtServers}}, distribuirai rapidamente il tuo stack LAMP con il monitoraggio e la scansione delle vulnerabilità integrati. Per vedere il server LAMP in azione, installerai e configurerai il sistema di gestione del contenuto [WordPress](https://wordpress.org/) gratuito e open source.

## Obiettivi

* Eseguire il provisioning di un server LAMP nel giro di qualche minuto
* Applicare l'ultima versione Apache, MySQL e PHP
* Ospitare un sito web o un blog installando e configurando WordPress
* Utilizzare il monitoraggio per rilevare le interruzioni e le prestazioni lente
* Valutare le vulnerabilità e proteggere dal traffico non desiderato

## Servizi utilizzati

Questa esercitazione utilizza i seguenti runtime e servizi:

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura

![Diagramma dell'architettura](images/solution4/Architecture.png)

1. L'utente finale accede al server LAMP e alle applicazioni utilizzando un browser web

## Prima di iniziare

{: #prereqs}

1. Contatta il tuo amministratore dell'infrastruttura per ottenere le seguenti autorizzazioni.
  * Autorizzazione di rete necessaria per completare l'**uplink di rete pubblica e privata**

### Configura l'accesso VPN

1. [Assicurati che il tuo accesso VPN sia abilitato](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Devi essere un utente master (**Master User**) per abilitare l'accesso VPN o contattare l'utente master per l'accesso.
     {:tip}
2. Ottieni le tue credenziali di accesso VPN nella [tua pagina utente nell'elenco utenti](https://{DomainName}/iam#/users).
3. Esegui l'accesso alla VPN mediante [l'interfaccia web](https://www.softlayer.com/VPN-Access) oppure utilizza un client VPN per [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) o [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Per il client VPN, utilizza il nome dominio completo (FQDN) di un singolo punto di accesso VPN a un data center dalla [pagina di accesso web VPN](https://www.softlayer.com/VPN-Access), del modulo *vpn.xxxnn.softlayer.com* come indirizzo gateway.
   {:tip}

## Crea i servizi

In questa sezione, eseguirai il provisioning di un server virtuale pubblico con una configurazione fissa. {{site.data.keyword.BluVirtServers_short}} può essere distribuito nel giro di qualche minuto dalle immagini di server virtuale in specifiche ubicazioni geografiche. I server virtuali spesso si occupano dei picchi di richieste dopo i quali possono essere sospesi o spenti in modo che l'ambiente cloud corrisponda perfettamente alle tue esigenze dell'infrastruttura.

1. Nel tuo browser, accedi alla [pagina del catalogo {{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group).
2. Seleziona **Public Virtual Server** e fai clic su **Create**.
3. In **Image**, seleziona l'ultima versione di **LAMP** in **Ubuntu**. Anche se viene fornito preinstallato con Apache, MySQL e PHP, reinstallerai PHP e MySQL con la versione più recente.
4. In **Network Interface** seleziona l'opzione **Public and Private Network Uplink**.
5. Esamina le altre opzioni di configurazione e fai clic su **Provision** per creare il tuo server virtuale.
  ![Configura il server virtuale](images/solution4/ConfigureVirtualServer.png)

Dopo che il server è stato creato, vedrai le credenziali di accesso del server. Anche se puoi connetterti tramite SSH utilizzando l'indirizzo IP pubblico del server, ti consigliamo di accedere al server attraverso la rete privata e di disabilitare l'accesso SSH sulla rete pubblica.

1. Attieniti a [questa procedura](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network) per proteggere la macchina virtuale e per disabilitare l'accesso SSH sulla rete pubblica.
1. Utilizzando i tuoi nome utente, password e indirizzo IP privato, stabilisci una connessione al server con SSH.
   ```sh
   sudo ssh root@<Private-IP-Address>
   ```
   {: pre}

  Puoi trovare l'indirizzo IP privato e la password del server nel dashboard.
  {:tip}

  ![Server virtuale creato](images/solution4/VirtualServerCreated.png)

## Reinstalla Apache, MySQL e PHP

Ti consigliamo di aggiornare periodicamente lo stack LAMP con le patch di sicurezza e le correzioni dei bug più recenti. In questa sezione, eseguirai i comandi per aggiornare le origini di pacchetto Ubuntu e reinstallare Apache, MySQL e PHP con la versione più recente. Nota l'accento circonflesso (^) alla fine del comando.

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

Un'opzione alternativa consiste nell'eseguire un upgrade di tutti i pacchetti con `sudo apt-get update && sudo apt-get dist-upgrade`.
{:tip}

## Verifica l'installazione e la configurazione

In questa sezione, verificherai che Apache, MySQL e PHP siano aggiornati e in esecuzione sull'immagine Ubuntu. Implementerai anche le impostazioni di sicurezza consigliate per MySQL.

1. Verifica Ubuntu aprendo l'indirizzo IP pubblico nel browser. Dovresti vedere la pagina di benvenuto di Ubuntu.
   ![Verifica Ubuntu](images/solution4/VerifyUbuntu.png)
2. Verifica che la porta 80 sia disponibile per il traffico web eseguendo il seguente comando.
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![Verifica la porta](images/solution4/VerifyPort.png)
3. Esamina le versioni Apache, MySQL e PHP installate utilizzando i seguenti comandi.
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
4. Esegui questo script per proteggere il database MySQL.
  ```sh
  mysql_secure_installation
  ```
  {: pre}
5. Immetti la password root MySQL e configura le impostazioni di sicurezza per il tuo ambiente. Quando hai terminato, esci dal prompt mysql immettendo `\q`.
  ```sh
  mysql -u root -p
  ```
  {: pre}

  Il nome utente e la password predefiniti di MySQL sono root e root.
  {:tip}
6. Puoi inoltre creare rapidamente una pagina di informazioni PHP con il seguente comando.
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. Visualizza la pagina di informazioni PHP che hai creato: apri un browser e vai a `http://{YourPublicIPAddress}/info.php`. Sostituisci l'indirizzo IP pubblico del tuo server virtuale. Sarà simile alla seguente immagine.

![Informazioni PHP](images/solution4/PHPInfo.png)

### Installa e configura WordPress

Prova il tuo stack LAMP installando un'applicazione. La seguente procedura installa la piattaforma WordPress open source, che viene spesso utilizzata per creare siti web e blog. Per ulteriori informazioni e per le impostazioni per l'installazione di produzione, vedi la [documentazione di WordPress](https://codex.wordpress.org/Main_Page).

1. Esegui questo comando per installare WordPress.
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. Configura WordPress per utilizzare MySQL e PHP. Esegui questo comando per aprire un editor di testo e creare il file `/etc/wordpress/config-localhost.php`.
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. Copia le seguenti righe nel file sostituendo *yourPassword* con la tua password del database MySQL e lasciando immutati gli altri valori. Salva il file ed esci utilizzando `Ctrl+X`.
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
4. In una directory di lavoro, crea un file di testo `wordpress.sql` per configurare il database WordPress.
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. Aggiungi i seguenti comandi sostituendo la tua password del database a *yourPassword* e lasciando immutati gli altri valori. Salva quindi il file.
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. Esegui il seguente comando per creare il database.
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. Una volta completato il comando, elimina il file `wordpress.sql`. Sposta l'installazione di WordPress alla radice dei documenti del server web.
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. Completa la configurazione di WordPress ed esegui la pubblicazione sulla piattaforma. Apri un browser e vai a `http://{tuoIndirizzoIPPubblicoVM}/wordpress`. Sostituisci l'indirizzo IP pubblico della tua VM. Dovrebbe essere simile alla seguente immagine.
   ![Sito WordPress in esecuzione](images/solution4/WordPressSiteRunning.png)

## Configura il dominio

Per utilizzare un nome dominio esistente con il tuo server LAMP, aggiorna il record A per puntare all'indirizzo IP pubblico del server virtuale.
Puoi visualizzare l'indirizzo IP pubblico del server dal dashboard.

## Monitoraggio e utilizzo del server

Per garantire la disponibilità del server e la migliore esperienza dell'utente, il monitoraggio deve essere abilitato su ogni server di produzione. In questa sezione, esplorerai le opzioni disponibili per monitorare il tuo server virtuale e comprendere l'utilizzo del server in qualsiasi momento.

### Monitoraggio del server

Sono disponibili due tipi di monitoraggio di base: SERVICE PING e SLOW PING.

* **SERVICE PING** controlla che il tempo di risposta del server sia pari a 1 secondo o meno
* **SLOW PING** controlla che il tempo di risposta del server sia pari a 5 secondi o meno

Poiché il SERVICE PING viene aggiunto per impostazione predefinita, aggiungi il monitoraggio SLOW PING con la seguente procedura.

1. Dal dashboard, seleziona il tuo server dall'elenco di dispositivi e fai quindi clic sulla scheda **Monitoring**.
  ![Monitoraggio dei ping lenti](images/solution4/SlowPing.png)
2. Fai clic su **Manage Monitors**.
3. Aggiungi l'opzione di monitoraggio **SLOW PING** e fai clic su **Add Monitor**. Seleziona il tuo indirizzo IP pubblico dall'elenco di indirizzi IP.
  ![Aggiungi monitoraggio dei ping lenti](images/solution4/AddSlowPing.png)

  **Nota**: i monitoraggi duplicati con le stesse configurazioni non sono consentiti. Può essere creato solo un singolo monitoraggio per ogni configurazione.

Se non viene ricevuta una risposta entro il lasso di tempo assegnato, viene inviato un avviso all'indirizzo email sull'account {{site.data.keyword.Bluemix_notm}}.
  ![Due monitoraggi](images/solution4/TwoMonitoring.png)

### Utilizzo del server

Seleziona la scheda **Usage** per comprendere l'utilizzo della CPU e della memoria del server corrente.
  ![Utilizzo del server](images/solution4/ServerUsage.png)

## Sicurezza del server

I {{site.data.keyword.BluVirtServers}} forniscono diverse opzioni di sicurezza, come ad esempio la scansione delle vulnerabilità e dei firewall aggiuntivi.

### Programma di scansione delle vulnerabilità

Il programma di scansione delle vulnerabilità esegue una scansione del server per rilevare le eventuali vulnerabilità correlate al server. Per eseguire una scansione delle vulnerabilità sul server, attieniti alla seguente procedura.

1. Dal dashboard, seleziona il tuo server e fai quindi clic sulla scheda **Security**.
2. Fai clic su **Scan** per avviare la scansione.
3. Una volta completata la scansione, fai clic su **Scan Complete** per visualizzare il report della scansione.
  ![Due monitoraggi](images/solution4/Vulnerabilities.png)
4. Esamina le eventuali vulnerabilità notificate.
  ![Due monitoraggi](images/solution4/VulnerabilityResults.png)

### Firewall

Un altro modo per proteggere il server è aggiungendo un firewall. I firewall forniscono un livello di sicurezza essenziale, impedendo al traffico indesiderato di raggiungere i tuoi server, riducendo la probabilità di un attacco e facendo in modo che le tue risorse server siano dedicate al loro uso previsto. Le opzioni firewall vengono fornite su richiesta senza interruzioni del servizio.

I firewall sono disponibili come una funzione aggiuntiva per tutti i server nella rete pubblica dell'infrastruttura. Come parte del processo di ordine, puoi selezionare l'hardware specifico per il dispositivo o un firewall software per fornire protezione. In alternativa, puoi distribuire i dispositivi firewall dedicati all'ambiente e distribuire il server virtuale a una VLAN protetta. Per ulteriori informazioni, vedi [Firewall](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started).

## Rimuovi le risorse

Per rimuovere il tuo server virtuale, completa la seguente procedura.

1. Accedi al [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices). 
2. Dal menu **Devices**, seleziona **Device List**.
3. Fai clic su **Actions** per il server virtuale che vuoi rimuovere e seleziona **Cancel**.

## Contenuto correlato

* [Distribuisci uno stack LAMP utilizzando Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
