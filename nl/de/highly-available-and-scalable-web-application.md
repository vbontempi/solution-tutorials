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

# Virtuelle Server zum Erstellen einer hoch verfügbaren und skalierbaren Web-App verwenden
{: #highly-available-and-scalable-web-application}

Das Hinzufügen weiterer Server zu einer Anwendung ist ein gängiges Muster, um zusätzliche Lasten verarbeiten zu können. Ein weiterer wichtiger Aspekt, um eine Anwendungsverfügbarkeit und Ausfallsicherheit zu erhöhen, ist die Implementierung der Anwendung in mehreren Zonen oder Standorten mit Datenreplikation und Lastausgleich.

Dieses Lernprogramm führt Sie durch ein Szenario, für das Folgendes benötigt wird:

- Zwei Webanwendungsserver in Dallas.
- Eine Lastausgleichsfunktion für die Cloud, um die Datenverkehrslast auf zwei Server innerhalb eines Standortes zu verteilen.
- Ein MySQL-Datenbankserver.
- Ein permanenter Dateispeicher zum Speichern von Anwendungsdateien und Sicherungen.
- Die Konfiguration des zweiten Standortes mit den gleichen Konfigurationen wie für den ersten Standort und das anschließende Hinzufügen von Cloud Internet Services, um im Falle eines Ausfalls einer Kopie Datenverkehr an den funktionsfähigen Standort umleiten zu können.

## Lernziele
{: #objectives}

* {{site.data.keyword.virtualmachinesshort}} für die Installation von PHP und MySQL erstellen
* {{site.data.keyword.filestorage_short}} verwenden, um Anwendungsdateien und Datenbanksicherungen auf Platte zu speichern
* {{site.data.keyword.loadbalancer_short}} bereitstellen, um Anforderungen an die Anwendungsserver zu verteilen
* Lösung durch das Hinzufügen eines zweiten Standortes für eine bessere Anwendungsverfügbarkeit und Ausfallsicherheit erweitern

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet-Services](https://{DomainName}/catalog/services/internet-services)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur
{: #architecture}

Die Anwendung ist ein einfaches PHP-Front-End - ein Wordpress-Blog - mit einer MySQL-Datenbank. Mehrere Front-End-Server verarbeiten die Anforderungen.

<p style="text-align: center;">
  ![Diagramm der Architektur](images/solution14/Architecture.png)
</p>

1. Der Benutzer stellt eine Verbindung zu der Anwendung her.
2. Der {{site.data.keyword.loadbalancer_short}} (die Lastausgleichsfunktion) wählt einen der Server in einwandfreiem Zustand aus, um die Anforderung zu verarbeiten.
3. Der ausgewählte Server greift auf die in einem gemeinsam genutzten Dateispeicher gespeicherten Anwendungsdateien zu.
4. Der Server extrahiert außerdem Informationen aus der Datenbank und gibt die Seite schließlich an den Benutzer aus.
5. Der Datenbankinhalt wird in regelmäßigen Abständen gesichert. Falls der Master fehlschlägt, ist ein Standby-Datenbankserver verfügbar.

## Vorbereitende Schritte
{: #prereqs}

### VPN-Zugriff konfigurieren

In diesem Lernprogramm ist die Lastausgleichsfunktion der Einstiegspunkt für die Anwendungsbenutzer. Die {{site.data.keyword.virtualmachinesshort}}-Instanzen müssen im öffentlichen Internet nicht sichtbar sein. Daher können sie nur mit einer privaten IP-Adresse bereitgestellt werden und Sie verwenden Ihre VPN-Verbindung, um auf den Servern zu arbeiten.

1. [Stellen Sie sicher, dass der VPN-Zugriff aktiviert ist](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Sie sollten ein **Masterbenutzer** sein, um den VPN-Zugriff zu aktivieren. Wenden Sie sich ansonsten für den Zugriff an den Masterbenutzer.
     {:tip}
2. Rufen Sie Ihre VPN-Zugriffsberechtigungsnachweise ab, indem Sie Ihren Benutzer in der [Benutzerliste](https://{DomainName}/iam#/users) auswählen.
3. Melden Sie sich über [die Webschnittstelle](https://www.softlayer.com/VPN-Access) am VPN an oder verwenden Sie einen VPN-Client für [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [Mac OS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) oder [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

Sie können diesen Schritt überspringen und alle Ihre Server im öffentlichen Internet sichtbar machen (wenn Sie sie privat halten, sorgt dies jedoch für ein zusätzliches Maß an Sicherheit). Wenn Sie die Server öffentlich machen möchten, wählen Sie **Uplink zum öffentlichen und privaten Netz** bei der Bereitstellung von {{site.data.keyword.virtualmachinesshort}} aus.
{: tip}

### Kontoberechtigungen prüfen

Wenden Sie sich an Ihren Infrastruktur-Masterbenutzer, um die folgenden Berechtigungen zu erhalten:
- **Netz**, damit Sie {{site.data.keyword.virtualmachinesshort}} mit der Option **Uplink zum öffentlichen und privaten Netz** erstellen können (diese Berechtigung ist nicht erforderlich, wenn Sie das VPN verwenden, um eine Verbindung zu den Servern herzustellen).

## Einen Server für die Datenbank bereitstellen
{: #database_server}

In diesem Abschnitt konfigurieren Sie einen Server als Masterdatenbank.

1. Rufen Sie den Katalog in der {{site.data.keyword.Bluemix}}-Konsole auf und wählen Sie im Abschnitt 'Infrastruktur' die [{{site.data.keyword.virtualmachinesshort}}-Instanz](https://{DomainName}/catalog/infrastructure/virtual-server-group) aus.
2. Wählen Sie **Öffentlicher virtueller Server** aus und klicken Sie dann auf **Erstellen**.
3. Konfigurieren Sie den Server mit den folgenden Optionen:
   - Setzen Sie **Name** auf **db1**.
   - Wählen Sie einen Standort aus, an der der Server bereitgestellt werden soll. **Alle anderen Server und Ressourcen, die in diesem Lernprogramm erstellt wurden, müssen an demselben Standort erstellt werden.**
   - Wählen Sie das Image **Ubuntu Minimal** aus.
   - Behalten Sie die Standardberechnungsversion bei. Das Lernprogramm wurde mit der niedrigsten Version getestet, sollte jedoch mit jeder Version funktionieren.
   - Wählen Sie unter **Angeschlossene Speicherplatten** die 25-GB-Bootplatte aus.
   - Wählen Sie unter **Netzschnittstelle** die Option **100 MB/s Uplink zum öffentlichen und privaten Netz** aus.

     Wenn Sie keinen VPN-Zugriff konfiguriert haben, wählen Sie die Option **100 MB/s Uplink zum öffentlichen und privaten Netz** aus.
     {: tip}
   - Überprüfen Sie die anderen Konfigurationsoptionen und klicken Sie auf **Bereitstellen**, um den Server bereitzustellen.

      ![Virtuellen Server konfigurieren](images/solution14/db-server.png)

   Hinweis: Der Bereitstellungsprozess kann zwei bis fünf Minuten dauern, bis der Server betriebsbereit ist. Nachdem der Server erstellt wurde, finden Sie die Serverberechtigungsnachweise auf der Serverdetailseite unter **Geräte > Geräteliste**. Um eine SSH-Verbindung zum Server herzustellen, benötigen Sie die private oder öffentliche IP-Adresse des Servers, den Benutzernamen und das Kennwort (klicken Sie dazu auf den Pfeil neben dem Gerätenamen).
   {: tip}

## MySQL installieren und konfigurieren
{: #mysql}

Der Server ist nicht mit einer Datenbank ausgestattet. In diesem Abschnitt installieren Sie MySQL auf dem Server.

### MySQL installieren

1. Stellen Sie mithilfe von SSH eine Verbindung zum Server her:
   ```sh
   ssh root@<private_oder_öffentliche_ip-adresse>
   ```

   Denken Sie daran, eine Verbindung zum VPN-Client mit der richtigen [Site-Adresse](https://www.softlayer.com/VPN-Access) basierend auf dem **Standort** Ihres virtuellen Servers herzustellen.
   {:tip}
2. Installieren Sie MySQL:
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   Möglicherweise werden Sie zur Eingabe eines Kennworts aufgefordert. Lesen Sie die Anweisungen in der angezeigten Konsole.
   {:tip}
3. Führen Sie das folgende Script aus, um die sichere Installation der MySQL-Datenbank zu gewährleisten:
   ```sh
   mysql_secure_installation
   ```

   Sie werden möglicherweise zur Auswahl einiger Optionen aufgefordert. Treffen Sie diese Auswahl sorgfältig basierend auf Ihren Anforderungen.
   {:tip}

### Datenbank für die Anwendung erstellen

1. Melden Sie sich bei MySQL an und erstellen Sie eine Datenbank mit dem Namen `wordpress`:
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. Erteilen Sie Zugriff auf die Datenbank und ersetzen Sie den Datenbankbenutzernamen (database-username) und das Datenbankkennwort (database-password) durch das, was Sie zuvor festgelegt haben.

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. Überprüfen Sie, ob die Datenbank erstellt wurde:

   ```sh
   show databases;
   ```

4. Beenden Sie Datenbank mit folgendem Befehl:

   ```sh
   exit
   ```

5. Notieren Sie sich den Datenbanknamen, den Benutzer und das Kennwort. Sie benötigen diese Informationen, wenn Sie die Anwendungsserver konfigurieren.

### MySQL-Server für andere Server im Netz sichtbar machen

Standardmäßig ist MySQL nur an der lokalen Schnittstelle empfangsbereit. Die Anwendungsserver müssen eine Verbindung zur Datenbank herstellen können. Die MySQL-Konfiguration muss daher so geändert werden, dass MySQL an allen Netzschnittstellen empfangsbereit ist.

1. Bearbeiten Sie die Datei 'my.cnf' mit `nano /etc/mysql/my.cnf` und fügen Sie die folgenden Zeilen hinzu:
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. Beenden Sie die Datei und speichern Sie sie mit Strg + X.

3. Starten Sie MySQL erneut:

   ```sh
   systemctl restart mysql
   ```

4. Bestätigen Sie, dass MySQL an allen Schnittstellen empfangsbereit ist, indem Sie den folgenden Befehl ausführen:
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## Dateispeicher für Datenbanksicherungen erstellen
{: #database_backup}

Es gibt viele Möglichkeiten, wie Sicherungen ausgeführt und gespeichert werden können, wenn es um MySQL geht. In diesem Lernprogramm wird ein Crontab-Eintrag zum Erstellen eines Speicherauszugs des Datenbankinhalts auf Platte verwendet. Die Sicherungsdateien werden in einem Dateispeicher gespeichert. Dies ist natürlich ein einfacher Sicherungsmechanismus. Wenn Sie Ihren eigenen MySQL-Datenbankserver in einer Produktionsumgebung verwalten möchten, sollten Sie [eine der in der MySQL-Dokumentation beschriebenen Sicherungsstrategien implementieren](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html).

### Dateispeicher erstellen
{: #create_for_backup}

1. Rufen Sie den Katalog in der {{site.data.keyword.Bluemix}}-Konsole auf und wählen Sie [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage) aus.
2. Klicken Sie auf **Erstellen**.
3. Konfigurieren Sie den Service mit den folgenden Optionen:
   - Setzen Sie **Speichertyp** auf **Endurance**.
   - Wählen Sie den **Standort** aus, an dem Sie auch den Datenbankserver erstellt haben.
   - Wählen Sie eine Abrechnungsmethode aus.
   - Wählen Sie unter **Speicherpakete** die Option **0,25 IOPS/GB** aus.
   - Wählen Sie unter **Speichergröße** die Option **20 GB** aus.
   - Belassen Sie die **Größe des Snapshotbereichs** auf **0 GB**.
   - Klicken Sie auf 'Weiter', um den Service zu erstellen.

### Datenbankserver für die Verwendung des Dateispeichers autorisieren

Bevor ein virtueller Server einen Dateispeicher anhängen kann, muss er entsprechend autorisiert werden.

1. Wählen Sie den neu erstellten Dateispeicher aus der [Liste der vorhandenen Einträge](https://{DomainName}/classic/storage/file) aus.
2. Klicken Sie unter **Autorisierte Hosts** auf **Host autorisieren** und wählen Sie den virtuellen (Datenbank-)Server aus (wählen Sie **Geräte** > Virtueller Server als 'Gerätetyp'; > geben Sie den Namen des Servers ein).

### Dateispeicher für Datenbanksicherungen anhängen

Der Dateispeicher kann als NFS-Laufwerk in den virtuellen Server geladen werden.

1. Installieren Sie die NFS-Clientbibliotheken:
   ```sh
   apt-get -y install nfs-common
   ```

2. Erstellen Sie eine Datei mit dem Namen `/etc/systemd/system/mnt-database.mount`, indem Sie den folgenden Befehl ausführen:
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. Bearbeiten Sie die Datei 'mnt-database.mount' wie folgt:
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. Fügen Sie den nachfolgenden Inhalt zur Datei 'mnt-database.mount' hinzu und ersetzen Sie `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` von `What` durch den **Mountpunkt** des Dateispeichers (z. B. *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). Sie können die **Mountpunkt**-URL unter dem erstellten Dateispeicherservice abrufen.
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
   Verwenden Sie Strg + X, um das Nanofenster zu speichern und zu beenden.
   {: tip}

5. Erstellen Sie den Mountpunkt.
  ```sh
  mkdir /mnt/database
  ```

6. Hängen Sie den Speicher an.
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. Prüfen Sie, ob der Mount erfolgreich durchgeführt wurde.
   ```sh
   mount
   ```
   In den letzten Zeilen sollte der Dateispeichermount aufgelistet sein. Ist dies nicht der Fall, verwenden Sie `journalctl -xe`, um die Mountoperation zu debuggen.
   {: tip}

### Sicherung in regelmäßigen Intervallen einrichten

1. Erstellen Sie das Shell-Script `/root/dbbackup.sh` (verwenden Sie `touch` und `nano`) mit den folgenden Befehlen, indem Sie `CHANGE_ME` durch das zuvor angegebene Datenbankkennwort ersetzen:
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip > /mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. Stellen Sie sicher, dass die Datei ausführbar ist.
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. Bearbeiten Sie die Crontab-Datei.
   ```sh
   crontab -e
   ```
4. Damit die Sicherung jeden Tag um 23.00 Uhr ausgeführt wird, legen Sie den Inhalt folgendermaßen fest, speichern Sie die Datei und schließen Sie den Editor.
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## Zwei Server für die PHP-Anwendung bereitstellen
{: #app_servers}

In diesem Abschnitt erstellen Sie zwei Webanwendungsserver.

1. Rufen Sie den Katalog in der {{site.data.keyword.Bluemix}}-Konsole auf und wählen Sie den [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)-Service im Abschnitt 'Infrastruktur' aus.
2. Wählen Sie **Öffentlicher virtueller Server** aus und klicken Sie dann auf **Erstellen**.
3. Konfigurieren Sie den Server mit den folgenden Optionen:
   - Setzen Sie **Name** auf **app1**.
   - Wählen Sie dieselbe Position aus, an der Sie den Datenbankserver bereitgestellt haben.
   - Wählen Sie das Image **Ubuntu Minimal** aus.
   - Behalten Sie die Standardversion für die Datenverarbeitung bei.
   - Wählen Sie unter **Angeschlossene Speicherplatten** die 25-GB-Bootplatte aus.
   - Wählen Sie unter **Netzschnittstelle** die Option **100 MB/s Uplink zum öffentlichen und privaten Netz** aus.

     Wenn Sie keinen VPN-Zugriff konfiguriert haben, wählen Sie die Option **100 MB/s Uplink zum öffentlichen und privaten Netz** aus.
     {: tip}
   - Überprüfen Sie die anderen Konfigurationsoptionen und klicken Sie auf **Bereitstellen**, um den Server bereitzustellen.
     ![Virtuellen Server konfigurieren](images/solution14/db-server.png)
4. Wiederholen Sie die Schritte 1-3, um einen weiteren virtuellen Server mit dem Namen **app2** bereitzustellen.

## Dateispeicher für die gemeinsame Nutzung von Dateien zwischen den Anwendungsservern erstellen
{: shared_storage}

Mit diesem Dateispeicher können Sie die Anwendungsdateien zwischen dem Server *app1* und dem Server *app2* gemeinsam nutzen.

### Dateispeicher erstellen
{: #create_for_sharing}

1. Rufen Sie den Katalog in der {{site.data.keyword.Bluemix}}-Konsole auf und wählen Sie [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage) aus.
2. Klicken Sie auf **Erstellen**.
3. Konfigurieren Sie den Service mit den folgenden Optionen:
   - Setzen Sie **Speichertyp** auf **Endurance**.
   - Wählen Sie den **Standort** aus, an dem Sie auch die Anwendungsserver erstellt haben.
   - Wählen Sie eine Abrechnungsmethode aus.
   - Wählen Sie unter **Speicherpakete** die Option **2 IOPS/GB** aus.
   - Wählen Sie unter **Speichergröße** die Option **20 GB** aus.
   - Wählen Sie unter **Größe des Snapshotbereichs** die Option **20 GB** aus.
   - Klicken Sie auf 'Weiter', um den Service zu erstellen.

### Regelmäßige Snapshots konfigurieren

Mit [Snapshots](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) können Sie Ihre Daten bequem schützen, ohne dass sich dies auf die Leistung auswirkt. Darüber hinaus können Sie Snapshots in einem anderen Rechenzentrum replizieren.< 

1. Wählen Sie den Dateispeicher aus der [Liste der vorhandenen Einträge](https://{DomainName}/classic/storage/file) aus.
2. Bearbeiten Sie unter **Snapshotzeitpläne** den Snapshotzeitplan. Der Zeitplan könnte folgendermaßen definiert werden:
   1. Fügen Sie einen stündlichen Snapshot hinzu, legen Sie die Minutenangabe auf '30' fest und bewahren Sie die letzten 24 Snapshots auf.
   2. Fügen Sie einen täglichen Snapshot hinzu, setzen Sie die Zeit auf 23.00 Uhr und bewahren Sie die letzten 7 Snapshots auf.
   3. Fügen Sie einen wöchentlichen Snapshot hinzu, setzen Sie die Zeit auf 1.00 Uhr und bewahren Sie die letzten 4 Snapshots auf. Klicken Sie dann auf 'Speichern'.
      ![Snapshots sichern](images/solution14/snapshots.png)

### Anwendungsserver für die Verwendung des Dateispeichers autorisieren

1. Klicken Sie unter **Autorisierte Hosts** auf **Host autorisieren**, um die Anwendungsserver (app1 und app2) für die Verwendung dieses Dateispeichers zu autorisieren.

### Dateispeicher anhängen

Wiederholen Sie die folgenden Schritte auf den einzelnen Anwendungsservern (app1 und app2):

1. Installieren Sie die NFS-Clientbibliotheken.
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. Erstellen Sie eine Datei mit `touch /etc/systemd/system/mnt-www.mount` und bearbeiten Sie sie mit `nano /etc/systemd/system/mnt-www.mount` mit dem folgenden Inhalt, indem Sie `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` von `What` durch den **Mountpunkt** für den Dateispeicher ersetzen (z. B. *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). Sie finden die Mountpunkte unter [Liste der Dateispeicherdatenträger](https://{DomainName}/classic/storage/file).
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
3. Erstellen Sie den Mountpunkt.
   ```sh
   mkdir /mnt/www
   ```
4. Hängen Sie den Speicher an.
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. Prüfen Sie, ob der Mount erfolgreich durchgeführt wurde.
   ```sh
   mount
   ```
   In den letzten Zeilen sollte der Dateispeichermount aufgelistet sein. Ist dies nicht der Fall, verwenden Sie `journalctl -xe`, um die Mountoperation zu debuggen.
   {: tip}

Alle Schritte zur Konfiguration der Server könnten auch mithilfe eines [Bereitstellungsscripts](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script) oder durch die [Erfassung eines Image](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template) automatisiert werden.
{: tip}

## PHP-Anwendung auf den Anwendungsservern installieren und konfigurieren
{: #php_application}

In diesem Lernprogramm wird ein Wordpress-Blog eingerichtet. Alle Wordpress-Dateien werden im gemeinsam genutzten Dateispeicher installiert, so dass beide Anwendungsserver auf sie zugreifen können. Bevor Sie Wordpress installieren, müssen ein Web-Server und eine PHP-Laufzeitumgebung konfiguriert werden.

### nginx und PHP installieren

Wiederholen Sie die folgenden Schritte auf den einzelnen Anwendungsservern:

1. Installieren Sie nginx.
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. Installieren Sie PHP und den MySQL-Client.
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. Stoppen Sie den PHP-Service und nginx.
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. Ersetzen Sie den Inhalt mithilfe von `nano /etc/nginx/sites-available/default` durch die folgenden Informationen:
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
5. Erstellen Sie mithilfe des folgenden Befehls den Ordner `html` im Verzeichnis `/mnt/www` eines der beiden Anwendungsserver.
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### WordPress installieren und konfigurieren

Da Wordpress auf dem Dateispeichermount installiert wird, müssen Sie die folgenden Schritte nur auf einem der Server ausführen. Nehmen Sie für dieses Beispiel **app1**.

1. Rufen Sie die Wordpress-Installationsdateien ab.

   Wenn Ihr Anwendungsserver über eine öffentliche Netzverbindung verfügt, können Sie die Wordpress-Dateien direkt von innerhalb des virtuellen Servers herunterladen:

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   Wenn der virtuelle Server nur über eine private Netzverbindung verfügt, müssen Sie die Installationsdateien von einer anderen Maschine mit Internetzugang abrufen und sie auf den virtuellen Server kopieren. Unter der Annahme, dass Sie die Wordpress-Installationsdateien von 'https://wordpress.org/latest.tar.gz' abgerufen haben, können Sie sie mit `scp` auf den virtuellen Server kopieren:

   ```sh
   scp latest.tar.gz root@PRIVATE_IP-ADRESSE_DES_SERVERS:/tmp
   ```
   Ersetzen Sie `latest` durch den Namen der Datei, die Sie von der Wordpress-Website heruntergeladen haben.
   {: tip}

   Stellen Sie dann eine SSH-Verbindung zum virtuellen Server her und wechseln Sie in das Verzeichnis `tmp`.

   ```sh
   cd /tmp
   ```

2. Extrahieren Sie die Installationsdateien.

   ```
   tar xzvf latest.tar.gz
   ```

3. Bereiten Sie die Wordpress-Dateien vor.
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. Kopieren Sie die Dateien in den gemeinsam genutzten Dateispeicher.
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. Legen Sie Berechtigungen fest.
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -exec chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. Rufen Sie die folgenden Web-Services auf und fügen Sie das Ergebnis mithilfe von `nano` in `/mnt/www/html/wp-config.php` ein.
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   Wenn Ihr virtueller Server über keine öffentliche Netzverbindung verfügt, können Sie 'https://api.wordpress.org/secret-key/1.1/salt/' ganz einfach über Ihren Web-Browser öffnen.

7. Legen Sie die Datenbankberechtigungsnachweise unter Verwendung von `nano /mnt/www/html/wp-config.php` fest und aktualisieren Sie die Datenbankberechtigungsnachweise:

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   Wordpress ist konfiguriert. Um die Installation abzuschließen, müssen Sie auf die Wordpress-Benutzerschnittstelle zugreifen.

Starten Sie auf beiden Anwendungsservern den Web-Server und die PHP-Laufzeitumgebung:
7. Starten Sie den Service, indem Sie die folgenden Befehle ausführen:

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

Greifen Sie unter `http://YourAppServerIPAddress/` auf die Wordpress-Installation zu. Verwenden Sie dazu entweder die private IP-Adresse (wenn Sie die VPN-Verbindung nutzen) oder die öffentliche IP-Adresse von *app1* oder *app2*.
![Virtuellen Server konfigurieren](images/solution14/wordpress.png)

Wenn Sie die Anwendungsserver mit nur einer privaten Netzverbindung konfiguriert haben, können Sie keine Wordpress-Plug-ins, -Schemas oder -Upgrades direkt von der Wordpress-Administrationskonsole installieren. Sie müssen die Dateien über die Wordpress-Benutzerschnittstelle hochladen.
{: tip}

## Einen Lastausgleichsserver vor den Anwendungsservern bereitstellen
{: #load_balancer}

An diesem Punkt verfügen Sie über zwei Anwendungsserver mit separaten IP-Adressen. Sie sind möglicherweise sogar im öffentlichen Internet nicht sichtbar, wenn Sie nur einen Uplink zum privaten Netz bereitstellen. Wenn Sie eine Lastausgleichsfunktion vor diesen Servern hinzufügen, wird die Anwendung öffentlich. Die Lastausgleichsfunktion verbirgt außerdem die zugrunde liegende Infrastruktur für die Benutzer. Sie überwacht den Status der Anwendungsserver und sendet eingehende Anforderungen ggf. an Server in einwandfreiem Zustand.

1. Rufen Sie den Katalog auf, um eine [{{site.data.keyword.loadbalancer_short}}-Instanz](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer) zu erstellen.
2. Wählen Sie im Schritt **Plan** das gleiche Rechenzentrum für *app1* und *app2* aus.
3. Nehmen Sie in den **Netzeinstellungen** die folgenden Einstellungen vor:
   1. Wählen Sie das gleiche Teilnetz aus, in dem auch *app1* und *app2* bereitgestellt wurden.
   2. Verwenden Sie den IBM Standardsystempool für die öffentliche IP der Lastausgleichsfunktion.
4. Nehmen Sie unter **Basis** die folgenden Einstellungen vor:
   1. Geben Sie der Lastausgleichsfunktion einen Namen, z. B. **app-lb-1**.
   2. Behalten Sie die Standardprotokollkonfiguration bei - standardmäßig ist die Lastausgleichsfunktion für HTTP konfiguriert.
      Das SSL-Protokoll wird mit Ihren eigenen Zertifikaten unterstützt. Weitere Informationen finden Sie im Abschnitt zum [Importieren der SSL-Zertifikate in die Lastausgleichsfunktion](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates).
      {: tip}
5. Fügen Sie in **Serverinstanzen** die Server *app1* und *app2* hinzu.
6. Überprüfen und erstellen Sie die Lastausgleichsfunktion, um den Assistenten abzuschließen.

### Wordpress-Konfiguration so ändern, dass die URL der Lastausgleichsfunktion verwendet wird

Die Wordpress-Konfiguration muss so geändert werden, dass sie die Adresse der Lastausgleichsfunktion verwendet. Tatsächlich behält Wordpress einen Verweis auf [die Blog-URL und fügt diese Position in den Seiten ein](https://codex.wordpress.org/Settings_General_Screen). Wenn Sie diese Einstellung nicht ändern, leitet Wordpress die Benutzer direkt an die Back-End-Server weiter und umgeht so die Lastausgleichsfunkion oder funktioniert überhaupt nicht, wenn die Server nur über eine private IP-Adresse verfügen.

1. Suchen Sie die Adresse der Lastausgleichsfunktion auf deren Detailseite. Sie finden die von Ihnen erstellte Lastausgleichsfunktion unter [Netz / Lastausgleich / Lokal](https://{DomainName}/classic/network/loadbalancing/cloud).

   Sie können auch Ihren eigenen Domänennamen mit der Lastausgleichsfunktion verwenden, indem Sie einen CNAME-Datensatz hinzufügen, der auf die Adresse der Lastausgleichsfunktion in Ihrer DNS-Konfiguration verweist.
   {: tip}
2. Melden Sie sich als Administrator im Wordpress-Blog über die URL von *app1* oder *app2* an.
3. Setzen Sie unter 'Einstellungen/Allgemein' sowohl die Wordpress-Adresse (URL) als auch die Site-Adresse (URL) auf die Adresse der Lastausgleichsfunktion.
4. Speichern Sie die Einstellungen. Wordpress sollte an die Adresse der Lastausgleichsfunktion umgeleitet werden.
   Aufgrund der DNS-Weitergabe kann es einige Zeit dauern, bis die Adresse der Lastausgleichsfunktion aktiv wird.
   {: tip}

### Verhalten der Lastausgleichsfunktion testen

Die Lastausgleichsfunktion ist so konfiguriert, dass sie den Status der Server überprüft und Benutzer nur an Server in einwandfreiem Zustand weiterleitet. Um die Funktionsweise der Lastausgleichsfunktion zu verstehen, können Sie wie folgt vorgehen:

1. Rufen Sie die nginx-Protokolle für *app1* und *app2* mit dem folgenden Befehl auf:
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   Sie sollten bereits die regulären Pingsignale der Lastausgleichsfunktion sehen, mit denen sie den Serverstatus überprüft.
   {: tip}
2. Greifen Sie über die Adresse der Lastausgleichsfunktion auf Wordpress zu und erzwingen Sie ein erneutes Laden der Seite. Beachten Sie, dass in den nginx-Protokollen sowohl *app1* als auch *app2* Inhalte für die Seite enthalten. Die Lastausgleichsfunktion leitet den Datenverkehr wie erwartet auf beide Server um.

3. Stoppen Sie nginx auf *app1*.
   ```sh
   systemctl nginx stop
   ```

4. Laden Sie nach einem kurzen Moment die Wordpress-Seite erneut. Beachten Sie, dass sich alle Treffer auf *app2* beziehen.

5. Stoppen Sie nginx auf *app2*.

6. Laden Sie die Wordpress-Seite neu. Die Lastausgleichsfunktion gibt einen Fehler zurück, da kein Server in einwandfreiem Zustand vorhanden ist.

7. Starten Sie nginx erneut auf *app1*.
   ```sh
   systemctl nginx start
   ```

8. Sobald die Lastausgleichsfunktion *app1* als einwandfrei funktionierenden Server erkennt, wird der Datenverkehr an diesen Server umgeleitet.

## Lösung mit einem zweiten Standort erweitern (optional)
{: #secondregion}

Um die Ausfallsicherheit und Verfügbarkeit zu erhöhen, können Sie die Infrastrukturkonfiguration durch einen zweiten Standort erweitern und Ihre Anwendung an zwei Standorten ausführen.

Bei einer Implementierung des zweiten Standortes sieht die Architektur folgendermaßen aus.

<p style="text-align: center;">

  ![Diagramm der Architektur](images/solution14/Architecture2.png)
</p>

1. Benutzer greifen über IBM Cloud Internet Services (CIS) auf die Anwendung zu.
2. CIS leitet den Datenverkehr an einen ordnungsgemäß funktionierenden Standort weiter.
3. Innerhalb eines Standortes leitet eine Lastausgleichsfunktion den Datenverkehr an einen Server weiter.
4. Die Anwendung greift auf die Datenbank zu.
5. Die Anwendung speichert und ruft Medienressourcen aus einem Dateispeicher ab.

Um diese Architektur zu implementieren, müssen Sie am zweiten Standort die folgenden Schritte ausführen:

- Wiederholen Sie alle obigen Schritte für den neuen Standort.
- Richten Sie eine Datenbankreplikation zwischen den beiden MySQL-Servern über Standorte hinweg ein.
- Konfigurieren Sie IBM Cloud Internet Services, um den Datenverkehr zwischen den Standorten an ordnungsgemäß funktionierende Server zu verteilen, wie in [diesem anderen Lernprogramm](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis) beschrieben.

## Ressourcen entfernen
{: #removeresources}

1. Löschen Sie die Lastausgleichsfunktion.
2. Brechen Sie *db1*, *app1* und *app2* ab.
3. Löschen Sie die beiden Dateispeicherservices.
4. Wenn ein zweiter Standort konfiguriert ist, löschen Sie alle Ressourcen und Cloud Internet Services.

## Zugehöriger Inhalt
{: #related}

- Statischer Inhalt, der von Ihrer Anwendung bereitgestellt wird, kann von einem Content Delivery Network vor der Lastausgleichsfunktion profitieren, um die Last auf den Back-End-Servern zu reduzieren. Im Abschnitt zum [Beschleunigen der Bereitstellung statischer Dateien mithilfe eines CDN](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn) finden Sie ein Lernprogramm für die Implementierung eines Content Delivery Network.
- In diesem Lernprogramm stellen Sie zwei Server bereit. Es könnten automatisch weitere Server hinzugefügt werden, um zusätzliche Lasten zu verarbeiten. Mit der [automatischen Skalierung](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale) können Sie den manuellen Skalierungsprozess im Zusammenhang mit dem Hinzufügen oder Entfernen von virtuellen Servern automatisieren, um Ihre Geschäftsanforderungen zu unterstützen.
- Um die Verfügbarkeit und Disaster-Recovery-Optionen zu optimieren, kann Dateispeicher konfiguriert werden, um [automatische regelmäßige Snapshots](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) des Inhalts zu erstellen und eine [Replikation](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication) zu einem anderen Rechenzentrum durchzuführen.
