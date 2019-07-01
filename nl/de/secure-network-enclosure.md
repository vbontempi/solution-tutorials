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

# Workloads durch ein sicheres privates Netz isolieren
{: #secure-network-enclosure}

Isolierte und geschützte private Netzumgebungen sind eine Grundvoraussetzung im IaaS-Modell für die Bereitstellung von Anwendungen in der öffentlichen Cloud. Firewalls, VLANs, Routing und VPNs sind erforderliche Komponenten für die Erstellung isolierter privater Umgebungen. Diese Isolation ermöglicht die sichere Bereitstellung von virtuellen Maschinen und Bare-Metal-Servern in komplexen, mehrschichtigen Anwendungstopologien und bietet Schutz vor Bedrohungen im öffentlichen Internet.  

In diesem Lernprogramm erfahren Sie, wie eine [Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-) (VRA) in {{site.data.keyword.Bluemix_notm}} konfiguriert werden kann, um einen geschützten privaten Netzbereich zu erstellen. Die VRA-Gateway-Appliance stellt in einem einzigen selbstverwalteten Paket eine Firewall, ein VPN-Gateway sowie Netzadressumsetzung (Network Address Translation, NAT) und auf Unternehmen abgestimmtes Routing zur Verfügung. In diesem Lernprogramm wird am Beispiel einer VRA gezeigt, wie eine geschlossene und isolierte Netzumgebung in {{site.data.keyword.Bluemix_notm}} erstellt werden kann. In diesem geschlossenen Netzbereich können Topologien mit den bekannten und vielfach genutzten Technologien wie IP-Routing, VLANs, IP-Teilnetze, Firewallregeln sowie virtuelle Server und Bare-Metal-Server erstellt werden.  

{:shortdesc}

Das vorliegende Lernprogramm ist ein Ausgangspunkt für den klassischen Netzbetrieb in {{site.data.keyword.Bluemix_notm}} und beschreibt keine eigenständige Produktionsfunktionalität. Die folgenden zusätzlichen Funktionalitäten sollten ebenfalls in Betracht gezogen werden:
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [Hardware-Firewall-Appliances](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* [IPSec-VPN](https://{DomainName}/catalog/infrastructure/ipsec-vpn) für die sichere Anbindung an Ihr Rechenzentrum
* Hochverfügbarkeit durch VRA-Cluster und duale Uplinks
* Sicherheitsereignisse protokollieren und überwachen

## Ziele 
{: #objectives}

* Virtual Router Appliance (VRA) bereitstellen
* VLANs und IP-Teilnetze zum Bereitstellen von virtuellen Maschinen und Bare-Metal-Servern definieren
* VRA und geschlossenen Netzbereich durch Firewallregeln schützen

## Verwendete Services
{: #products}

Im vorliegenden Lernprogramm werden die folgenden {{site.data.keyword.Bluemix_notm}}-Services verwendet:
* [Virtual Router Appliance](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)

Für dieses Lernprogramm können Kosten anfallen. Die Virtual Router Appliance (VRA) ist nur mit einem monatlichen Preistarif verfügbar.

## Architektur
{: #architecture}

<p style="text-align: center;">

  ![Architektur](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. VPN konfigurieren
2. VRA bereitstellen 
3. Virtuellen Server erstellen
4. Zugriff über VRA steuern
5. Firewall für Netzbereich konfigurieren
6. APP-Zone definieren
7. INSIDE-Zone definieren

## Vorbereitende Schritte
{: #prereqs}

### VPN-Zugriff konfigurieren

Der im vorliegenden Lernprogramm erstellte, geschlossene Netzbereich ist im öffentlichen Internet nicht sichtbar. Die VRA und alle zugehörigen Server sind nur im privaten Netz zugänglich und werden über Ihr VPN angebunden. 

1. [Stellen Sie sicher, dass Ihr VPN-Zugriff aktiviert ist](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Sie sollten ein **Masterbenutzer** sein, um den VPN-Zugriff zu aktivieren oder den Zugriff vom Masterbenutzer einrichten lassen.
     {:tip}
2. Rufen Sie Ihre Berechtigungsnachweise für den VPN-Zugriff ab, indem Sie Ihren Benutzernamen in der [Benutzerliste](https://{DomainName}/iam#/users) auswählen.
3. Melden Sie sich [über die Webschnittstelle](https://www.softlayer.com/VPN-Access) bei dem VPN an oder verwenden Sie einen VPN-Client für [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) oder [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Verwenden Sie für den VPN-Client den FQDN eines einzelnen VPN-Zugriffspunkts im Rechenzentrum von der [Seite für den VPN-Webzugriff](https://www.softlayer.com/VPN-Access) im Format *vpn.xxxnn.softlayer.com* als Gateway-Adresse.
   {:tip}

### Kontoberechtigungen überprüfen

Die folgenden Berechtigungen können Sie beim Masterbenutzer der Infrastruktur anfordern:
- **Schnellberechtigungen**: Basisbenutzer
- **Netz**: Zum Erstellen und Konfigurieren des geschlossenen Netzbereichs ist 'All Network Permissions' erforderlich 
- **Services**: SSH-Schlüssel verwalten

### SSH-Schlüssel hochladen

Über das Portal können Sie den [öffentlichen SSH-Schlüssel hochladen](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial), der für den Zugriff und die Verwaltung der VRA und des privaten Netzes verwendet wird.  

### Zielrechenzentrum

Wählen Sie ein {{site.data.keyword.Bluemix_notm}}-Rechenzentrum aus, um das sichere private Netz bereitzustellen. 

### VLANs bestellen

Um den privaten Netzbereich im Zielrechenzentrum zu erstellen, müssen zunächst die erforderlichen privaten VLANs für Server zugeordnet werden. Das erste private VLAN und das erste öffentliche VLAN ist kostenlos. Weitere VLANs in einer mehrschichtigen Anwendungstopologie sind kostenpflichtig. 

Um sicherzustellen, dass genügend VLANs auf demselben Router des Rechenzentrums verfügbar sind und der VRA zugeordnet werden können, empfiehlt es sich, die VLANs zu bestellen. Weitere Informationen finden Sie unter [VLANs bestellen](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans).

## Virtual Router Appliance bereitstellen
{: #VRA}

Im ersten Schritt wird eine VRA bereitgestellt, die das IP-Routing und die Firewall für den privaten Netzbereich zur Verfügung stellen soll. Das Internet ist aus dem privaten Netzbereich über ein von {{site.data.keyword.Bluemix_notm}} bereitgestelltes öffentliches Transit-VLAN erreichbar. Ein Gateway und eine optionale Hardare-Firewall ermöglichen die Anbindung des öffentlichen VLAN an die VLANs im geschützten privaten Netzbereich. In der Lösung für das vorliegende Lernprogramm werden die Gateway- und die Firewallkomponente durch eine Virtual Router Appliance (VRA) bereitgestellt. 

1. Wählen Sie im Katalog eine [Gateway-Appliance](https://{DomainName}/gen1/infrastructure/provision/gateway) aus.
3. Wählen Sie im Abschnitt **Gateway-Anbieter** die Option 'AT&T' aus. Als Uplink-Geschwindigkeit können Sie 'bis 20 Gb/s' oder 'bis 2 Gb/s' auswählen.
4. Geben Sie im Abschnitt **Hostname** einen Hostnamen und eine Domäne für Ihre neue VRA ein.
5. Wenn Sie das Kontrollkästchen **Hochverfügbarkeit** auswählen, werden zwei VRA-Einheiten in einer Konfiguration für aktive Sicherung mit VRRP ausgeführt.
6. Wählen Sie im Abschnitt **Standort** den gewünschten Standort und den **Pod** für Ihre VRA aus.
7. Wählen Sie 'Einzelprozessor' oder 'Dualprozessor' aus. Eine Liste der Server wird angezeigt. Klicken Sie auf das zugehörige Optionsfeld, um einen Server auszuwählen. 
8. Wählen Sie die Größe des Arbeitsspeichers (**RAM**) aus. Für eine Produktionsumgebung werden mindestens 64 GB empfohlen. Der Mindestwert für eine Testumgebung sind 8 GB.
9. Wählen Sie einen **SSH-Schlüssel aus** (optional). Der ausgewählte SSH-Schlüssel wird in der VRA installiert, damit der Benutzer 'vyatta' mit diesem Schlüssel auf die VRA zugreifen kann.
10. Festplattenlaufwerk. Behalten Sie die Standardeinstellung bei.
11. Wählen Sie im Abschnitt **Uplink-Port-Geschwindigkeiten** die Kombination aus Geschwindigkeit, Redundanz und privater und/oder öffentlicher Schnittstelle aus, die Ihren Anforderungen entspricht.
12. Behalten Sie im Abschnitt **Add-ons** die Standardeinstellung bei. Wenn Sie IPv6 in der öffentlichen Schnittstelle verwenden möchten, wählen Sie die IPv6-Adresse aus.

Auf der rechten Seite wird Ihre **Bestellübersicht** angezeigt. Wählen Sie das Kontrollkästchen _Die im Folgenden aufgeführten Servicevereinbarungen anderer Anbieter habe ich gelesen und stimme ihnen zu:_ aus und klicken Sie auf die Schaltfläche **Erstellen**. Ihr Gateway wird bereitgestellt.

In der [Einheitenliste](https://{DomainName}/classic/devices) wird die VRA nahezu sofort mit einem **Uhrsymbol** angezeigt, das auf aktive Transaktionen hinweist, die momentan auf dieser Einheit ausgeführt werden. Bis die VRA-Erstellung abgeschlossen ist, wird das **Uhrsymbol** weiterhin angezeigt. In diesem Zeitraum können außer dem Anzeigen der Details keine weiteren Konfigurationsoptionen für die Einheit ausgeführt werden.
{:tip}

### Bereitgestellte VRA überprüfen

1. Überprüfen Sie die neue VRA. Wählen Sie im [Infrastruktur-Dashboard](https://{DomainName}/classic) die Option **Netz** im linken Teilfenster aus und danach **Gateway-Appliances**, um die Seite [Gateway-Appliances](https://{DomainName}/classic/network/gatewayappliances) zu öffnen. Wählen Sie den Namen der neu bereitgestellten VRA in der Spalte **Gateway** aus, um die Seite 'Gateway-Details' aufzurufen. ![](images/solution33-secure-network-enclosure/Gateway-detail.png)

2. Notieren Sie die `private` und die `öffentliche` IP-Adresse der VRA für die zukünftige Verwendung.

## Erstkonfiguration der VRA
{: #initial_VRA_setup}

1. Melden Sie sich von Ihrer Workstation aus über das SSL-VPN bei der VRA mit dem Standardbenutzerkonto **vyatta** an und bestätigen Sie die SSH-Sicherheitsanweisungen. 
   ```bash
   SSH vyatta@<private IP-Adresse der VRA>
   ```
   {: codeblock}

   Wenn SSH zur Eingabe eines Kennworts auffordert, wurde der SSH-Schlüssel nicht in den Build einbezogen. Rufen Sie die VRA über den [Web-Browser](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui) auf und verwenden Sie dabei die `private IP-Adresse der VRA`. Das Kennwort stammt von der Seite [Softwarekennwörter](https://{DomainName}/classic/devices/passwords). Wählen Sie auf der Registerkarte **Konfiguration** die Verzweigung 'System/login/vyatta' aus und fügen Sie den gewünschten SSH-Schlüssel hinzu.{:tip}

   Zum Konfigurieren muss die VRA mit dem Befehl `configure` in den Bearbeitungsmodus (edit) versetzt werden. Im Bearbeitungsmodus (`edit`) wird die Eingabeaufforderung von `$` in `#` geändert. Nachdem die VRA-Konfiguration erfolgreich geändert wurde, können Sie die Änderungen mit dem Befehl `compare` anzeigen und mit dem Befehl `validate` überprüfen. Wenn Sie eine Änderung mit dem Befehl `commit` festschreiben, wird sie in die aktive Konfiguration übernommen und automatisch in der Startkonfiguration gespeichert.


   {:tip}
2. Erhöhen Sie die Sicherheit, indem Sie nur die SSH-Anmeldung zulassen. Nachdem die Anmeldung über das private Netz erfolgreich eingerichtet wurde, können Sie den Zugriff und die Authentifizierung mit Benutzer-ID und Kennwort inaktivieren. 
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
   Ab diesem Punkt des Lernprogramms wird davon ausgegangen, dass alle VRA-Befehle an der Eingabeaufforderung `edit` im Anschluss an den Befehl `configure` eingegeben werden.
3. Überprüfen Sie die Erstkonfiguration:
   ```
   show
   ```
   {: codeblock}

   Die VRA ist für die {{site.data.keyword.Bluemix_notm}}-IaaS-Umgebung vorkonfiguriert. Dazu gehören die folgenden Komponenten:
   - NTP-Server
   - Namensserver
   - SSH
   - HTTPS-Web-Server 
   - Standardzeitzone 'US/Chicago'
4. Legen Sie die örtliche Zeitzone nach Bedarf fest. Beim automatischen Vervollständigen mit der Tabulatortaste werden die potenziellen Zeitzonenwerte aufgelistet.
   ```
   set system time-zone <zeitzone>
   ```
   {: codeblock}
5. Legen Sie das Ping-Verhalten fest. Um das Routing und die Fehlerbehebung für die Firewall zu unterstützen, wird das Pingsignal nicht inaktiviert. 
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. Aktivieren Sie den statusabhängigen Firewall-Betrieb. Die VRA-Firewall ist standardmäßig statusunabhängig. 
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. Schreiben Sie Ihre Änderungen fest und speichern Sie sie automatisch in der Startkonfiguration: 
   ```
   commit
   ```
   {: codeblock}

## Ersten virtuellen Server bestellen
{: #order_virtualserver}

An dieser Stelle wird ein virtueller Server erstellt, um die Diagnose von VRA-Konfigurationsfehlern zu unterstützen. Der erfolgreiche Zugriff auf die VSI wird über das private {{site.data.keyword.Bluemix_notm}}-Netz überprüft, bevor in einem späteren Schritt der Zugriff auf die VSI über die VRA geleitet wird. 

1. Bestellen Sie einen [virtuellen Server](https://{DomainName}/catalog/infrastructure/virtual-server-group).  
2. Wählen Sie **Öffentlicher virtueller Server** aus und setzen Sie den Vorgang fort.
3. Gehen Sie auf der Bestellseite wie folgt vor:
   - Geben Sie für **Abrechnung** die Option **Stündlich** an.
   - Legen Sie *VSI-Hostname* und *Domänenname* fest. Dieser Domänenname wird zwar nicht für Routing und DNS verwendet, er sollte jedoch mit Ihren Namenskonventionen für das Netz übereinstimmen. 
   - Geben Sie für **Standort** denselben Standort an wie für die VRA.
   - Geben Sie für **Profil** den Wert **C1.1x1** an.
   - Fügen Sie den **SSH-Schlüssel** hinzu, den Sie vorher angegeben haben.
   - Geben Sie für **Betriebssystem** die Option **CentOS 7.x - Minimal** an.
   - Im Abschnitt **Uplink-Port-Geschwindigkeiten** muss für die Netzschnittstelle anstelle des Standardwerts *>Öffentliches und privates Netz* der Wert **Uplink zum privaten Netz** angegeben werden. Dadurch wird sichergestellt, dass der neue Server nicht direkt auf das Internet zugreifen kann und der Internetzugang durch die Routing- und Firewallregeln in der VRA gesteuert wird.
   - Geben Sie für **Privates VLAN** die VLAN-ID des zuvor bestellten privaten VLAN an.
4. Klicken Sie auf das entsprechende Feld, um die Servicevereinbarungen anderer Anbieter zu akzeptieren, und klicken Sie anschließend auf **Erstellen**.
5. Überwachen Sie die vollständige Durchführung auf der Seite [Einheiten](https://{DomainName}/classic/devices) oder per E-Mail. 
6. Notieren Sie die *private IP-Adresse* der VSI für einen späteren Arbeitsschritt und prüfen Sie im Abschnitt **Netz** auf der Seite **Einheitendetails**, dass die VSI dem richtigen VLAN zugeordnet ist. Wenn dies nicht zutrifft, löschen Sie die betreffende VSI und erstellen Sie eine neue VSI im richtigen VLAN. 
7. Überprüfen Sie den erfolgreichen Zugang zu der VSI über das private {{site.data.keyword.Bluemix_notm}}-Netz mit Ping und SSH von Ihrer lokalen Workstation aus über das VPN:
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## VLAN-Zugriff über die VRA leiten
{: #routing_vlan_via_vra}

Das bzw. die privaten VLAN(s) für den virtuellen Server wurde(n) vom {{site.data.keyword.Bluemix_notm}}-Managementsystem dieser VRA zugeordnet. In diesem Stadium ist die VSI weiterhin per IP-Routing im privaten {{site.data.keyword.Bluemix_notm}}-Netz erreichbar. Als nächstes leiten Sie das Teilnetz über die VRA, um das geschützte private Netz zu erstellen, und überprüfen, dass die VSI danach nicht zugänglich ist. 

1. Rufen Sie die Gateway-Details für die VRA über die Seite [Gateway-Appliances](https://{DomainName}/classic/network/gatewayappliances) auf und suchen Sie den Abschnitt **Zugeordnete VLANs** auf der unteren Hälfte der Seite. Das zugehörige VLAN wird in diesem Abschnitt aufgelistet. 
2. Wenn zu diesem Zeitpunkt weitere VLANs hinzugefügt werden sollen, navigieren Sie zum Abschnitt **VLAN zuordnen**. Das Dropdown-Feld *VLAN auswählen*, in dem weitere bereitgestellte VLANs ausgewählt werden können, müsste aktiviert sein. ![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png)

   Wenn kein auswählbares VLAN angezeigt wird, sind keine VLANS auf demselben Router wie die VRA verfügbar. In diesem Fall muss ein [Support-Ticket](https://{DomainName}/unifiedsupport/cases/add) geöffnet werden, um ein privates VLAN auf demselben Router wie die VRA anzufordern.
   {:tip}
5. Wählen Sie das VLAN aus, das Sie der VRA zuordnen möchten, und klicken Sie auf 'Speichern'. Die Erstzuordnung des VLAN kann mehrere Minuten dauern. Nachdem die Zuordnung abgeschlossen ist, müsste das VLAN unter der Überschrift **Zugeordnete VLANs** angezeigt werden. 

Zu diesem Zeitpunkt wird das VLAN und das zugehörige Teilnetz nicht durch die VRA geschützt oder über die VRA geleitet und die VSI ist über das private {{site.data.keyword.Bluemix_notm}}-Netz zugänglich. Für das VLAN wird der Status *Bypassed* (Umgangen) angezeigt.

4. Wählen Sie in der rechten Spalte die Option **Aktionen** und anschließend die Option **VLAN umleiten** aus, um das VLAN bzw. Teilnetz über die VRA zu leiten. Dieser Vorgang dauert mehrere Minuten. Nach dem Aktualisieren der Anzeige wird für das VLAN der Status *Routed* (Weitergeleitet) angezeigt. 
5. Wählen Sie den [VLAN-Namen](https://{DomainName}/classic/network/vlans/) aus, um die Details für das VLAN anzuzeigen. Die bereitgestellte VSI und das zugeordnete primäre IP-Teilnetz wird angezeigt. Notieren Sie die private VLAN-ID \<nnnn\> (im vorliegenden Beispiel 1199), da diese in einem späteren Schritt verwendet wird. 
6. Wählen Sie das [Teilnetz](https://{DomainName}/classic/network/subnets) aus, um die Details für das IP-Teilnetz anzuzeigen. Notieren Sie die Netzadresse des Teilnetzes, die Gateway-Adresse und die CIDR (/26), da diese für die weitere VRA-Konfiguration erforderlich sind. Da im privaten Netz 64 primäre IP-Adressen bereitgestellt werden, finden Sie die Gateway-Adresse möglicherweise erst auf Seite 2 oder 3. 
7. Überprüfen Sie, dass das Teilnetz bzw. VLAN über die VRA geleitet wird und die VSI **NICHT** mit dem Befehl `ping` von Ihrer Workstation aus über das Managementnetz erreichbar ist. 
   ```bash
   ping <private IP-Adresse der VSI>
   ```
   {: codeblock}

Damit ist die Konfiguration der VRA über die {{site.data.keyword.Bluemix_notm}}-Konsole abgeschlossen. Die weiteren Konfigurationsschritte für den Netzbereich und das IP-Routing werden direkt auf der VRA über SSH ausgeführt. 

## IP-Routing und geschützten Netzbereich konfigurieren
{: #vra_setup}

Beim Festschreiben der VRA-Konfiguration wird die aktive Konfiguration geändert und die Änderungen werden automatisch in der Startkonfiguration gespeichert.

Falls es erforderlich wird, auf eine zuvor verwendete Konfiguration zurückzugreifen, können Sie die letzten 20 Festschreibungspunkte anzeigen, vergleichen und wiederherstellen. Weitere Details zum Festschreiben und Speichern der Konfiguration finden Sie in der Veröffentlichung [Vyatta Network OS Basic System Configuration Guide](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
   ```bash
   show system commit
   rollback n
   compare
   ```
   {: codeblock}

### VRA-IP-Routing konfigurieren

Konfigureren Sie die virtuelle VRA-Netzschnittstelle so, dass das neue Teilnetz vom privaten {{site.data.keyword.Bluemix_notm}}-Netz aus weitergeleitet wird.  

1. Melden Sie sich über SSH bei der VRA an. 
   ```bash
   SSH vyatta@<private VRA-IP-Adresse>
   ```
   {: codeblock}
2. Erstellen Sie eine neue virtuelle Schnittstelle mit der privaten VLAN-ID, der IP-Adresse des Teilnetz-Gateways und der CIDR, die Sie in den vorherigen Schritten notiert haben. Die CIDR lautet in der Regel /26. 
   ```
   set interfaces bonding dp0bond0 vif <VLAN-ID> address <Teilnetz-Gateway-IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   Es ist wichtig, dass die **`Teilnetz-Gateway-IP`** verwendet wird. Dies ist in der Regel die Anfangsadresse des Teilnetzes plus eins. Wenn Sie eine ungültige Gateway-Adresse eingeben, wird der Fehler `Configuration path: interfaces bonding dp0bond0 vif xxxx address [x.x.x.x] is not valid` ausgegeben. Korrigieren Sie den Befehl und geben Sie ihn erneut ein. Sie finden den korrekten Wert unter 'Netz > IP-Management > Teilnetze'. Klicken Sie auf das Teilnetz, für das Sie die Gateway-Adresse ermitteln. Der zweite Listeneintrag mit der Beschreibung **Gateway** ist die IP-Adresse, die als <Teilnetz-Gateway-IP>/<CIDR> einzugeben ist.
   {: tip}

3. Listen Sie die neue virtuelle Schnittstelle (vif) auf: 
   ```
   show interfaces
   ```
   {: codeblock}

   Das vorliegende Beispiel für die Schnittstellenkonfiguration enthält die vif 1199 und die Teilnetz-Gateway-Adresse.
   ![](images/solution33-secure-network-enclosure/show_interfaces.png)
4. Überprüfen Sie, dass die VSI wieder über das Managementnetz von Ihrer Workstation aus zugänglich ist: 
   ```bash
   ping <private IP-Adresse der VSI>
   ```
   {: codeblock}
   
   Wenn die VSI nicht erreichbar ist, überprüfen Sie, dass die IP-Routingtabelle der VRA wie erwartet konfiguriert ist. Falls erforderlich, löschen Sie die Route und erstellen Sie sie erneut. Zum Ausführen eines Befehls 'show' im Konfigurationsmodus können Sie den Befehl 'run' verwenden:    
   ```bash
    run show ip route <Teilnetz-Gateway-IP>
   ```
   {: codeblock}

Damit ist die Konfiguration für das IP-Routing abgeschlossen.

### Geschützten Netzbereich konfigurieren

Der geschützte private Netzbereich wird durch Konfigurieren von Zonen und Firewallregeln erstellt. Lesen Sie die VRA-Dokumentation zur [Firewallkonfiguration](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls), bevor Sie fortfahren. 

Die beiden folgenden Zonen werden definiert:
   - INSIDE: Nicht öffentliche Netze und Managementnetze von IBM
   - APP: Das Benutzer-VLAN und das Teilnetz im privaten Netzbereich		

1. Definieren Sie Firewalls und Standardeinstellungen:
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   Wenn ein Befehl 'set' versehentlich zweimal ausgeführt wird, erhalten Sie eine Nachricht *'Configuration path xxxxxxxx is not valid. Node exists'*. Diese Nachricht kann ignoriert werden. Um einen falschen Parameter zu ändern, muss zuerst der Knoten mit dem Befehl 'delete security xxxxx xxxx xxxxx' gelöscht werden.
   {:tip}
2. Erstellen Sie die Ressourcengruppe für das private {{site.data.keyword.Bluemix_notm}}-Netz. Diese Adressgruppe definiert, welche privaten {{site.data.keyword.Bluemix_notm}}-Netze auf den geschützten Netzbereich zugreifen können, und welche Netze von dem Netzbereich aus erreichbar sind. Zwei Gruppen von IP-Adressen benötigen aus- und eingehenden Zugriff auf den geschützten Netzbereich: die SSL-VPN-Rechenzentren und das {{site.data.keyword.Bluemix_notm}}-Servicenetz (privates bzw. Back-End-Netz). In den [{{site.data.keyword.Bluemix_notm}}-IP-Bereichen](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges) finden Sie die vollständige Liste der IP-Bereiche, die zugelassen werden müssen. 
   - Definieren Sie die SSL-VPN-Adresse(n) des Rechenzentrums bzw. der Rechenzentren, das bzw. die Sie für den VPN-Zugriff verwenden. Wählen Sie im Abschnitt 'SSL-VPN' der {{site.data.keyword.Bluemix_notm}}-IP-Bereiche die VPN-Zugriffspunkte für Ihr Rechenzentrum oder Ihren Cluster der Rechenzentren aus. Das vorliegende Beispiel zeigt die VPN-Adressbereiche für die {{site.data.keyword.Bluemix_notm}}-Rechenzentren am Standort London.
     ```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - Definieren Sie die Adressbereiche für das {{site.data.keyword.Bluemix_notm}}-Servicenetz (im Back-End-Netz oder privaten Netz) für WDC04 (DAL01) und Ihr Zielrechenzentrum. In diesem Beispiel werden WDC04 (mit zwei Adressen) sowie DAL01 und LON06 verwendet.
     ```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. Erstellen Sie die APP-Zone für das Benutzer-VLAN und das Teilnetz sowie die INSIDE-Zone für das private {{site.data.keyword.Bluemix_notm}}-Netz. Ordnen Sie die zuvor erstellten Firewalls zu. In Zonendefinitionen werden die VRA-Netzschnittstellennamen verwendet, um die zugeordnete Zone für jedes VLAN anzugeben. In dem Befehl zum Erstellen der APP-Zone muss die VLAN-ID des VLAN angegeben werden, das in einem vorherigen Schritt der VRA zugeordnet wurde. Dies ist im folgenden Code als `VLAN-ID` angegeben.
   ```
   set security zone-policy zone INSIDE description "IBM Internal network"
   set security zone-policy zone INSIDE default-action drop
   set security zone-policy zone INSIDE interface dp0bond0
   set security zone-policy zone INSIDE to APP firewall INSIDE-TO-APP

   set security zone-policy zone APP description "Application network"
   set security zone-policy zone APP default-action drop

   set security zone-policy zone APP interface dp0bond0.<VLAN-ID>
   set security zone-policy zone APP to INSIDE firewall APP-TO-INSIDE
   ```
   {: codeblock}
4. Schreiben Sie die Konfiguration fest und überprüfen Sie von Ihrer Workstation aus mit Ping, dass die Firewall jetzt keinen Datenverkehr für die VSI über die VRA zulässt: 
   ```
   commit
   ```
   {: codeblock}

   ```bash
   ping <private IP-Adresse der VSI>
   ```
   {: codeblock}
5. Definieren Sie die Firewallzugriffsregelen für UDP, TCP und ICMP:
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
6. Überprüfen Sie den Firewallzugriff. 
   - Überprüfen Sie, dass die Firwall INSIDE-TO-APP jetzt ICMP- sowie UDP/TCP-Datenverkehr von Ihrer lokalen Maschine zulässt:
     ```bash
     ping <private IP-Adresse der VSI>
     SSH root@<private IP-Adresse der VSI>
     ```
     {: codeblock}
   - Überprüfen Sie, dass die Firewall APP-TO-INSIDE jetzt ICMP- sowie UDP/TCP-Datenverkehr zulässt. Melden Sie sich über SSH bei der VSI an und überprüfen Sie mit Ping einen der {{site.data.keyword.Bluemix_notm}}-Namensserver mit den IP-Adressen 10.0.80.11 und 10.0.80.12:
     ```bash
     SSH root@<private IP-Adresse der VSI>
     [root@vsi  ~]# ping 10.0.80.11
     ```
     {: codeblock}
7. Überprüfen Sie den weiterhin bestehenden Zugang zur VRA-Managementschnittstelle über SSH von Ihrer Workstation aus. Wenn der Zugang weiterhin besteht, überprüfen und speichern Sie die Konfiguration. Andernfalls kann durch einen Warmstart der VRA eine vorherige funktionierende Konfiguration wiederhergestellt werden. 
   ```bash
   SSH vyatta@<private IP-Adresse der VRA>
   ```
   {: codeblock}

   ```
   show security
   ```
   {: codeblock}

### Debugging für Firewallregeln

Die Firewallprotokolle können über die Eingabeaufforderung für VRA-Betriebsbefehle angezeigt werden. In dieser Konfiguration wird nur verworfener Datenverkehr für jede Zone protokolliert, um die Diagnose fehlerhafter Firewallkonfigurationen zu unterstützen.  

1. Überprüfen Sie die Firewallprotokolle für verweigerten Datenverkehr. Durch die regelmäßige Überprüfung der Protokolle wird erkennbar, ob Server in der APP-Zone regulär oder unerlaubt versuchen, Services im IBM Netz zu kontaktieren. 
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. Wenn Services oder Server nicht erreichbar sind und die Firewall-Protokolle keine Hinweise liefern, überprüfen Sie mit der zuvor verwendeten `VLAN-ID`, ob der erwartete IP-Datenverkehr für ping/ssh in der VRA-Netzschnittstelle vom privaten {{site.data.keyword.Bluemix_notm}}-Netz aus oder in der VRA-Schnittstelle zum VLAN vorhanden ist.
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN-ID> traffic
   ```

## VRA schützen
{: #securing_the_vra}

1. Wenden Sie die VRA-Sicherheitsrichtlinie an. Die standardmäßige, richtlinienbasierte Einrichtung von Firewallzonen schützt nicht den Zugriff auf die VRA selbst. Dieser Zugriffsschutz wird durch CPP (Control Plane Policing, Überwachung von Steuerebenen) konfiguriert. Die VRA stellt einen grundlegenden CPP-Regelsatz als Vorlage bereit. Fusionieren Sie diesen Regelsatz mit Ihrer Konfiguration:
   ```bash
   merge /opt/vyatta/etc/cpp.conf
   ```
   {: codeblock}

Dadurch wird ein neuer Firewallregelsatz mit dem Namen `CPP` erstellt. Überprüfen Sie die zusätzlichen Regeln im Bearbeitungsmodus und schreiben Sie sie fest: 
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. Schützen Sie den öffentlichen SSH-Zugriff. Aufgrund eines bekannten Problems mit der Vyatta-Firmware wird derzeit nicht empfohlen, mit `set service SSH listen-address x.x.x.x` den SSH-Verwaltungszugriff über das öffentliche Netz zu verwenden. Alternativ kann der externe Zugriff über die CPP-Firewall für den Bereich der öffentlichen IP-Adressen blockiert werden, die von der öffentlichen VRA-Schnittstelle verwendet werden. Das hier verwendete `<öffentliche VRA-IP-Teilnetz>` stimmt mit `<öffentliche IP-Adresse der VRA>` überein, wobei das letzte Oktett auf null gesetzt ist (x.x.x.0). 
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <öffentliches VRA-IP-Teilnetz>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit
   ```
   {: codeblock}
3. Überprüfen Sie den VRA-SSH-Verwaltungszugriff über das interne IBM Netz. Wenn der Zugriff auf die VRA über SSH nach dem Durchführen von Commitoperationen verloren geht, können Sie die VRA über die KVM-Konsole aufrufen, die auf der Seite 'Einheitendetails' der VRA über das Dropdown-Menü für Aktionen verfügbar ist.

Damit ist die Einrichtung des geschützten privaten Netzbereichs mit einer einzelnen Firewallzone, die ein VLAN und ein Teilnetz enthält, abgeschlossen. Mithilfe der dargestellten Anweisungen können weitere Firewallzonen, Regeln, virtuelle Server und Bare-Metal-Server, VLANs und Teilnetze hinzugefügt werden. 

## Ressourcen entfernen
{: #removeresources}

In diesem Schritt bereinigen Sie die Ressourcen, um die in diesem Lernprogramm erstellten Elemente zu entfernen.

Die VRA wird mit einem monatlichen Preistarif bereitgestellt. Die Stornierung führt nicht zu einer Erstattung. Eine Stornierung wird nur empfohlen, wenn die betreffende VRA im Folgemonat nicht mehr benötigt wird. Wenn ein Hochverfügbarkeitscluster mit zwei VRAs erforderlich ist, kann für die bestehende einzelne VRA ein Upgrade auf der Seite [Gateway-Details](https://{DomainName}/classic/network/gatewayappliances/) durchgeführt werden.
{:tip}  

- Stornieren Sie alle virtuellen Server oder Bare-Metal-Server.
- Stornieren Sie die VRA.
- Stornieren Sie alle zusätzlichen VLANs durch ein Support-Ticket. 

## Zugehörige Inhalte
{: #related}

- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Statische und portierbare IP-Teilnetze](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence Platform](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Vyatta-Dokumentation](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
