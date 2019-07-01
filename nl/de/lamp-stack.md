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

# PHP-Webanwendung auf einem LAMP-Stack
{: #lamp-stack}

Dieses Lernprogramm enthält die Schritte zum Erstellen eines virtuellen Ubuntu **L**inux-Servers mit dem **A**pache-Web-Server, **M**ySQL Database und **P**HP-Scripting. Diese Kombination von Software - häufiger als LAMP-Stack bezeichnet - ist sehr beliebt und wird häufig zur Bereitstellung von Websites und Webanwendungen verwendet. Mit {{site.data.keyword.BluVirtServers}} können Sie schnell Ihren LAMP-Stack mit integrierter Überwachung und integriertem Scan auf Sicherheitslücken implementieren. Um den LAMP-Server in Aktion zu sehen, installieren und konfigurieren Sie das kostenlose Open-Source-Content-Management-System [WordPress](https://wordpress.org/).

## Lernziele

* Einen LAMP-Server in wenigen Minuten bereitstellen
* Die neueste Version von Apache, MySQL und PHP anwenden
* Eine Website oder einen Blog durch die Installation und Konfiguration von WordPress hosten
* Überwachung zur Erkennung von Ausfällen und langsamer Leistung verwenden
* Sicherheitslücken bewerten und Schutz vor unerwünschtem Datenverkehr durchsetzen

## Verwendete Services

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur

![Diagramm der Architektur](images/solution4/Architecture.png)

1. Der Endbenutzer greift über einen Web-Browser auf den LAMP-Server und die Anwendungen zu.

## Vorbereitende Schritte

{: #prereqs}

1. Wenden Sie sich an Ihren Infrastrukturadministrator, um die folgenden Berechtigungen zu erhalten.
  * Erforderliche Netzberechtigung für die Ausführung des **Uplinks zum öffentlichen und privaten Netz**

### VPN-Zugriff konfigurieren

1. [Stellen Sie sicher, dass der VPN-Zugriff aktiviert ist](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Sie sollten ein **Masterbenutzer** sein, um den VPN-Zugriff zu aktivieren. Wenden Sie sich ansonsten für den Zugriff an den Masterbenutzer.
     {:tip}
2. Rufen Sie Ihre VPN-Zugriffsberechtigungsnachweise auf [Ihrer Benutzerseite unter der Benutzerliste](https://{DomainName}/iam#/users) ab.
3. Melden Sie sich über [die Webschnittstelle](https://www.softlayer.com/VPN-Access) am VPN an oder verwenden Sie einen VPN-Client für [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [Mac OS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) oder [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Verwenden Sie für den VPN-Client den FQDN eines einzelnen VPN-Zugriffspunkts eines Rechenzentrums auf der [Seite für den VPN-Webzugriff](https://www.softlayer.com/VPN-Access) oder das Formular *vpn.xxxnn.softlayer.com* als Gateway-Adresse.
   {:tip}

## Services erstellen

In diesem Abschnitt stellen Sie einen öffentlichen virtuellen Server mit einer festen Konfiguration zur Verfügung. {{site.data.keyword.BluVirtServers_short}} kann in wenigen Minuten über Images eines virtuellen Servers an bestimmten Standorten implementiert werden. Virtuelle Server werden oft für Lastspitzen eingesetzt, nach denen sie ausgesetzt oder heruntergefahren werden können, sodass die Cloud-Umgebung perfekt zu Ihren Infrastrukturanforderungen passt.

1. Greifen Sie in Ihrem Browser auf die Seite mit dem [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)-Katalog zu.
2. Wählen Sie **Öffentlicher virtueller Server** aus und klicken Sie auf **Erstelle**.
3. Wählen Sie unter **Image** die aktuelle Version von **LAMP** unter **Ubuntu** aus. Auch wenn dies bereits mit Apache, MySQL und PHP installiert ist, installieren Sie PHP und MySQL mit der neuesten Version erneut.
4. Wählen Sie unter **Netzschnittstelle** die Option **Uplinks zum öffentlichen und privaten Netz** aus.
5. Überprüfen Sie die anderen Konfigurationsoptionen und klicken Sie auf **Bereitstellen**, um den virtuellen Server zu erstellen.
  ![Virtuellen Server konfigurieren](images/solution4/ConfigureVirtualServer.png)

Nachdem der Server erstellt wurde, werden die Anmeldeberechtigungsnachweise für den Server angezeigt. Obwohl Sie über SSH eine Verbindung über die öffentliche IP-Adresse des Servers herstellen können, wird empfohlen, über das private Netz auf den Server zuzugreifen und den SSH-Zugriff im öffentlichen Netz zu inaktivieren.

1. Befolgen Sie [diese Schritte](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network), um die virtuelle Maschine zu sichern und den SSH-Zugriff im öffentlichen Netz zu inaktivieren.
1. Stellen Sie mithilfe Ihres Benutzernamens, des Kennworts und der privaten IP-Adresse eine Verbindung zum Server mit SSH her.
   ```sh
   sudo ssh root@<private_ip-adresse>
   ```
   {: pre}

  Die private IP-Adresse und das Kennwort des Servers finden Sie im Dashboard.
  {:tip}

  ![Erstellter virtueller Server](images/solution4/VirtualServerCreated.png)

## Apache, MySQL und PHP erneut installieren

Es wird empfohlen, den LAMP-Stack regelmäßig mit den neuesten Sicherheitspatches und Fehlerkorrekturen zu aktualisieren. In diesem Abschnitt führen Sie Befehle aus, um Ubuntu-Paketquellen zu aktualisieren und Apache, MySQL und PHP mit der neuesten Version zu installieren. Beachten Sie das Winkelzeichen (^) am Ende des Befehls.

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

Alternativ können Sie ein Upgrade aller Pakete mit `sudo apt-get update && sudo apt-get dist-upgrade` durchführen.
{:tip}

## Installation und Konfiguration überprüfen

In diesem Abschnitt überprüfen Sie, ob Apache, MySQL und PHP auf dem neuesten Stand sind und auf dem Ubuntu-Image ausgeführt werden. Außerdem implementieren Sie die empfohlenen Sicherheitseinstellungen für MySQL.

1. Überprüfen Sie Ubuntu, indem Sie die öffentliche IP-Adresse im Browser öffnen. Sie sollten die Ubuntu-Begrüßungsseite sehen.
   ![Ubuntu überprüfen](images/solution4/VerifyUbuntu.png)
2. Überprüfen Sie, ob Port 80 für den Webdatenverkehr verfügbar ist, indem Sie den folgenden Befehl ausführen.
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![Port überprüfen](images/solution4/VerifyPort.png)
3. Überprüfen Sie die installierten Versionen von Apache, MySQL und PHP, indem Sie die folgenden Befehle verwenden.
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
4. Führen Sie das folgende Script aus, um die MySQL-Datenbank zu sichern.
  ```sh
  mysql_secure_installation
  ```
  {: pre}
5. Geben Sie das MySQL-Rootkennwort ein und konfigurieren Sie die Sicherheitseinstellungen für Ihre Umgebung. Wenn Sie fertig sind, beenden Sie die 'mysql'-Eingabeaufforderung, indem Sie `\q` eingeben.
  ```sh
  mysql -u root -p
  ```
  {: pre}

  Der MySQL-Standardbenutzername und das zugehörige Kennwort sind 'root' und 'root'.
  {:tip}
6. Außerdem können Sie mit dem folgenden Befehl schnell eine PHP-Infoseite erstellen.
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. Zeigen Sie die von Ihnen erstellte PHP-Infoseite an: Öffnen Sie einen Browser und wechseln Sie zu `http://{ihre_öffentliche_ip-adresse}/info.php`. Ersetzen Sie die öffentliche IP-Adresse Ihres virtuellen Servers. Das Ganze sieht dann in etwa wie folgt aus.

![PHP-Info](images/solution4/PHPInfo.png)

### WordPress installieren und konfigurieren

Machen Sie sich mit Ihrem LAMP-Stack vertraut, indem Sie eine Anwendung installieren. In den folgenden Schritten wird die Open-Source-WordPress-Plattform installiert, die häufig zum Erstellen von Websites und Blogs verwendet wird. Weitere Informationen und Einstellungen zur Installation in einer Produktionsumgebung finden Sie in der [WordPress-Dokumentation](https://codex.wordpress.org/Main_Page).

1. Führen Sie den folgenden Befehl aus, um WordPress zu installieren.
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. Konfigurieren Sie WordPress für die Verwendung von MySQL und PHP. Führen Sie den folgenden Befehl aus, um einen Texteditor zu öffnen und die Datei `/etc/wordpress/config-localhost.php` zu erstellen.
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. Kopieren Sie die folgenden Zeilen in die Datei. Dabei wird *yourPassword* durch das MySQL-Datenbankkennwort ersetzt, während die anderen Werte unverändert bleiben. Speichern und beenden Sie die Datei mit `Strg + X`.
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
4. Erstellen Sie in einem Arbeitsverzeichnis eine Textdatei `wordpress.sql`, um die WordPress-Datenbank zu konfigurieren.
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. Fügen Sie folgenden Befehle hinzu. Dabei wird Ihr Datenbankkennwort für *yourPassword* ersetzt und die anderen Werte unverändert gelassen. Speichern Sie dann die Datei.
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. Führen Sie den folgenden Befehl aus, um die Datenbank zu erstellen.
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. Wenn der Befehl erfolgreich ausgeführt wurde, löschen Sie die Datei `wordpress.sql`. Verschieben Sie die WordPress-Installation in das Dokumentstammverzeichnis des Web-Servers.
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. Schließen Sie die WordPress-Konfiguration ab und veröffentlichen Sie das System auf der Plattform. Öffnen Sie einen Browser und wechseln Sie zu `http://{öffentliche_ip-adresse_ihrer_vm}/wordpress`. Ersetzen Sie die öffentliche IP-Adresse Ihrer VM. Das Ganze sollte dann in etwa wie folgt aussehen.
   ![WordPress site running](images/solution4/WordPressSiteRunning.png)

## Domäne konfigurieren

Wenn Sie einen vorhandenen Domänennamen mit Ihrem LAMP-Server verwenden möchten, aktualisieren Sie den A-Datensatz so, dass er auf die öffentliche IP-Adresse des virtuellen Servers verweist.
Sie können die öffentliche IP-Adresse des Servers über das Dashboard anzeigen.

## Serverüberwachung und -verwendung

Um die Serververfügbarkeit und eine optimale Benutzererfahrung zu gewährleisten, sollte die Überwachung auf jedem Produktionsserver aktiviert werden. In diesem Abschnitt untersuchen Sie die Optionen, die für die Überwachung des virtuellen Servers und die Verwendung des Servers zu einem bestimmten Zeitpunkt verfügbar sind.

### Serverüberwachung

Es stehen zwei grundlegende Überwachungstypen zur Verfügung: SERVICE PING und SLOW PING.

* **SERVICE PING** prüft, ob die Serverantwortzeit gleich oder weniger als eine Sekunde ist.
* **SLOW PING** prüft, ob die Serverantwortzeit gleich oder weniger als fünf Sekunden ist.

Da SERVICE PING standardmäßig hinzugefügt wird, fügen Sie die SLOW PING-Überwachung mit den folgenden Schritten hinzu.

1. Wählen Sie im Dashboard den Server aus der Liste der Geräte aus und klicken Sie dann auf die Registerkarte **Überwachung**.
  ![SLOW PING-Überwachung](images/solution4/SlowPing.png)
2. Klicken Sie auf **Überwachungen verwalten**.
3. Fügen Sie die Überwachungsoption **SLOW PING** hinzu und klicken Sie auf **Überwachung hinzufügen**. Wählen Sie Ihre öffentliche IP-Adresse für die IP-Adresse aus.
  ![SLOW PING-Überwachung hinzufügen](images/solution4/AddSlowPing.png)

  **Hinweis**: Doppelte Überwachungen mit den gleichen Konfigurationen sind nicht zulässig. Es kann nur eine Überwachung pro Konfiguration erstellt werden.

Wenn eine Antwort nicht in dem zugeordneten Zeitrahmen empfangen wird, wird ein Alert an die E-Mail-Adresse im {{site.data.keyword.Bluemix_notm}}-Konto gesendet.
  ![Zwei Überwachungen](images/solution4/TwoMonitoring.png)

### Serververwendung

Wählen Sie die Registerkarte **Verwendung** aus, um die aktuelle Speicherbelegung und CPU-Auslastung des Servers zu verstehen.
  ![Serververwendung](images/solution4/ServerUsage.png)

## Serversicherheit

{{site.data.keyword.BluVirtServers}} bieten mehrere Sicherheitsoptionen, wie die Ermittlung von Sicherheitslücken und Add-on-Firewalls.

### Schwachstellenscanner

Der Schwachstellenscanner durchsucht den Server nach allen Sicherheitslücken, die sich auf den Server beziehen. Führen Sie die folgenden Schritte aus, um einen Scan auf Sicherheitslücken auf dem Server auszuführen.

1. Wählen Sie im Dashboard Ihren Server aus und klicken Sie dann auf die Registerkarte **Sicherheit**.
2. Klicken Sie auf **Scannen**, um den Scan zu starten.
3. Wenn der Scan abgeschlossen ist, klicken Sie auf **Scan abgeschlossen**, um den Scanbericht anzuzeigen.
  ![Zwei Überwachungen](images/solution4/Vulnerabilities.png)
4. Überprüfen Sie alle gemeldeten Sicherheitslücken.
  ![Zwei Überwachungen](images/solution4/VulnerabilityResults.png)

### Firewalls

Eine weitere Möglichkeit zum Sichern des Servers besteht darin, eine Firewall hinzuzufügen. Firewalls stellen eine wesentliche Sicherheitsschicht dar: Sie verhindern unerwünschten Datenverkehr von Ihrem Server, verringern die Wahrscheinlichkeit eines Angriffs und lassen zu, dass Ihre Serverressourcen für den vorgesehenen Gebrauch gezielt zugewiesen werden können. Firewalloptionen werden bei Bedarf ohne Serviceunterbrechungen bereitgestellt.

Firewalls sind als Add-on-Funktion für alle Server im öffentlichen Infrastrukturnetz verfügbar. Als Teil des Bestellprozesses können Sie gerätespezifische Hardware oder eine Software-Firewall auswählen, um den Schutz zu gewährleisten. Alternativ dazu können Sie dedizierte Firewall-Geräte in der Umgebung implementieren und den virtuellen Server in einem geschützten VLAN bereitstellen. Weitere Informationen finden Sie unter [Firewalls](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started).

## Ressourcen entfernen

Führen Sie die folgenden Schritte aus, um Ihren virtuellen Server zu entfernen.

1. Melden Sie sich beim [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices) an.
2. Wählen Sie im Menü **Geräte** die Option **Geräteliste** aus.
3. Klicken Sie für den virtuellen Server, den Sie entfernen möchten, auf **Aktionen** und wählen Sie **Abbrechen** aus.

## Zugehöriger Inhalt

* [LAMP-Stack mit Terraform bereitstellen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
