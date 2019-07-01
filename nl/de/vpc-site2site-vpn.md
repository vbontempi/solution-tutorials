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

# VPC/VPN-Gateway für sicheren und privaten lokalen Zugriff auf Cloudressourcen verwenden
{: #vpc-site2site-vpn}

IBM akzeptiert seit April 2019 eine begrenzte Anzahl von Kunden, die an einem Early Access-Programm für Virtual Private Cloud (VPC) teilnehmen, wobei eine erweiterte Nutzung in den folgenden Monaten eröffnet werden soll. Wenn Ihre Organisation Zugriff auf IBM Virtual Private Cloud erhalten möchte, füllen Sie bitte dieses [Nominierungsformular](https://{DomainName}/vpc){: new_window} aus. Anschließend wird ein IBM Ansprechpartner Kontakt mit Ihnen in Bezug auf die weiteren Schritte aufnehmen.
{: important}

IBM bietet verschiedene Möglichkeiten an, um ein lokales Computernetz mit Ressourcen in der IBM Cloud sicher zu erweitern. Dadurch können Sie von der Elastizität der Bereitstellung von Servern profitieren, wenn Sie sie benötigen, und sie wieder entfernen, wenn sie nicht länger erforderlich sind. Darüber hinaus können Sie Ihre lokalen Funktionen problemlos und sicher mit den {{site.data.keyword.cloud_notm}}-Services verbinden.

Dieses Lernprogramm führt Sie durch die Herstellung einer Verbindung eines lokalen VPN-Gateways (VPN - Virtual Private Network, virtuelles privates Netz) mit einem Cloud-VPN, das in einer VPC erstellt wurde (VPC/VPN-Gateway). Zunächst werden Sie  eine neue {{site.data.keyword.vpc_full}} (VPC) und die zugeordneten Ressourcen, wie Teilnetze, Netzzugriffssteuerlisten (Network ACLs), Sicherungsgruppen und virtuelle Serverinstanzen (VSIs), erstellen.
Das VPC/VPN-Gateway richtet dann eine [IPsec](https://en.wikipedia.org/wiki/IPsec)-Site-zu-site-Verbindung zu einem lokalen VPN-Gateway ein. Das IPsec-Protokoll und die [Internet Key Exchange-Protokolle](https://en.wikipedia.org/wiki/Internet_Key_Exchange) (IKE-Protokolle) sind bewährte offene Standards für die sichere Kommunikation.

Zur weiteren Veranschaulichung des sicheren und privaten Zugriffs werden Sie einen Microservice in einer VSI bereitstellen, um auf {{site.data.keyword.cos_short}} (COS) zuzugreifen, wobei COS eine Geschäftsanwendungslinie darstellt.
Der COS-Service hat einen direkten Endpunkt, der für privaten kostenlosen Ingress/Egress verwendet werden kann, wenn alle Zugriffe innerhalb derselben Region der {{site.data.keyword.cloud_notm}} erfolgen. Ein lokaler Computer greift dann auf den COS-Microservice zu. Der gesamte Datenverkehr fließt durch das VPN und somit privat durch {{site.data.keyword.cloud_notm}}.

Es sind viele populäre Lösungen für lokale VPNs für Site-zu-Site-Gateways verfügbar. In diesem Lernprogramm wird das VPN-Gateway [strongSwan](https://www.strongswan.org/) für die Verbindung mit dem VPC/VPN-Gateway verwendet. Zur Simulation eines lokalen Rechenzentrums installieren Sie das Gateway 'strongSwan' auf einer VSI in {{site.data.keyword.cloud_notm}}.

{:shortdesc}
Eine VPC bietet Ihnen zusammengefasst die folgenden Möglichkeiten:

- Lokale Systeme mit Services und Workloads verbinden, die in {{site.data.keyword.cloud_notm}} ausgeführt werden
- Private und kostengünstige Konnektivität zu COS sicherstellen
- Cloudbasierte Systeme mit Services und Workloads verbinden, die lokal ausgeführt werden

## Lernziele
{: #objectives}

* Zugreifen auf eine virtuelle private Cloudumgebung über ein lokales Rechenzentrum oder eine (virtuelle) private Cloud
* Sicheres Erreichen von Cloud-Services durch private Serviceendpunkte

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet: 
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um einen Kostenvoranschlag auf der Grundlage der von Ihnen projektierten Nutzung zu generieren.
Obwohl keine Netzgebühren für den Zugriff auf COS über den Microservice in diesem Lernprogramm erhoben werden, fallen Standardnetzgebühren für den Zugriff auf die VPC an.

## Architektur
{: #architecture}

Das folgende Diagramm zeigt die virtuelle private Cloud, die einen App-Server enthält. Der App-Server betreibt einen Microservice, der mit dem {{site.data.keyword.cos_short}}-Service kommuniziert. Ein (simuliertes) lokales Netz und die virtuelle Cloudumgebung sind über VPN-Gateways verbunden.

![Architektur](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. Die Infrastruktur (VPC, Teilnetze, Sicherheitsgruppen mit Regeln, Netzzugriffssteuerlisten und VSIs) werden mithilfe eines bereitgestellten Scripts eingerichtet.
2. Der Microservice kommuniziert mit {{site.data.keyword.cos_short}} über einen direkten Endpunkt.
3. Ein VPC/VPN-Gateway wird bereitgestellt, um die virtuelle private Cloudumgebung für das lokale Netz zugänglich zu machen.
4. Die quelloffene Strongswan-Software für IPsec-Gateways wird lokal verwendet, um die VPN-Verbindung mit der Cloudumgebung herzustellen.

## Vorbereitende Schritte
{: #prereqs}

- Installieren Sie alle erforderlichen Befehlszeilentools (CLI-Tools), indem Sie [diese Schritte ausführen](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview). Sie benötigen das optionale CLI-Infrastruktur-Plug-in.
- Melden Sie sich bei {{site.data.keyword.cloud_notm}} über die Befehlszeile an. Weitere Informationen finden Sie unter [Einführung zur CLI](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli).
- Überprüfen Sie die Benutzerberechtigungen. Stellen Sie sicher, dass Ihr Benutzerkonto über die entsprechenden Berechtigungen verfügt, um VPC-Ressourcen erstellen und verwalten zu können. Eine Liste der erforderlichen Berechtigungen finden Sie unter [Benötigte Berechtigungen für VPC-Benutzer erteilen](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).
- Sie benötigen einen SSH-Schlüssel für die Herstellung einer Verbindung zu den virtuellen Servern. Wenn Sie keinen SSH-Schlüssel haben, befolgen Sie die [Anweisungen zum Erstellen eines Schlüssels](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- Installieren Sie [**jq**](https://stedolan.github.io/jq/download/). Dieses Tool wird von den bereitgestellten Scripts zur Verarbeitung von JSON-Ausgaben verwendet.

## Virtuellen App-Server in einer virtuellen privaten Cloud bereitstellen
Im Folgenden werden Sie die Scripts zur Einrichtung einer VPC-Basisumgebung und Code für einen Microservice, der mit dem {{site.data.keyword.cos_short}} kommuniziert, herunterladen. Anschließend stellen Sie den {{site.data.keyword.cos_short}}-Service bereit und richten die Basisversion ein.

### Code abrufen
{: #setup}
In dem Lernprogramm werden Scripts verwendet, um eine Basis der Infrastrukturressourcen bereitzustellen, bevor Sie die VPN-Gateways erstellen. Diese Scripts und der Code für den Microservice sind in einem GitHub-Repository verfügbar.

1. Rufen Sie den Anwendungscode ab:
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. Greifen Sie auf die Scripts für dieses Lernprogramm zu, indem Sie in das Verzeichnis **vpc-tutorials** und in das Verzeichnis **vpc-site2site-vpn** wechseln:
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### Services erstellen
In diesem Abschnitt melden Sie sich über die Befehlszeilenschnittstelle (CLI) bei {{site.data.keyword.cloud_notm}} an und erstellen eine Instanz von {{site.data.keyword.cos_short}}.

1. Stellen Sie sicher, dass Sie die vorausgesetzten Schritte für die Anmeldung ausgeführt haben.
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. Erstellen Sie eine Instanz von [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) unter Verwendung eines **Standardplans** oder eines **Lite-Plans**.
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   Beachten Sie, dass pro Konto nur eine Lite-Instanz erstellt werden kann. Wenn Sie bereits eine Instanz von {{site.data.keyword.cos_short}} haben, können Sie diese wiederverwenden.
   {: tip}

3. Erstellen Sie einen Serviceschlüssel mit der Rolle 'Leseberechtigter' (**Reader**):
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. Rufen Sie die Serviceschlüsseldetails im JSON-Format ab und speichern Sie diese in einer neuen Datei **credentials.json** im Unterverzeichnis **vpc-app-cos**. Die Datei wird später von der App verwendet.
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### Virtual Private Cloud-Basisressourcen erstellen
{: #create-vpc}
Das Lernprogramm stellt ein Script zum Erstellen der Basisressourcen bereit, die für dieses Lernprogramm erforderlich sind, das heißt, die Startumgebung. Das Script kann diese Umgebung entweder in einer vorhandenen VPC generieren oder eine neue VPC erstellen.

Erstellen Sie im Folgenden diese Ressourcen, indem Sie ein Setup-Script konfigurieren und anschließend ausführen. Das Script umfasst die Einrichtung eines Bastion-Hosts, wie dies in [Sicherer Zugriff auf ferne Instanzen mit einem Bastion-Host](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) beschrieben wird.

1. Kopieren Sie die Beispielkonfigurationsdatei in eine Datei hinüber, die verwendet werden soll:

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. Bearbeiten Sie die Datei **config.sh** und passen Sie die Einstellungen an Ihre Umgebung an. Sie müssen den Wert von **SSHKEYNAME** in den Namen oder die durch Kommas getrennte Liste von Namen von SSH-Schlüsseln ändern (siehe "Vorbereitende Schritte"). Ändern Sie die verschiedenen Einstellungen für **ZONE**, sodass sie Ihrer Cloudregion entsprechen. Alle anderen Variablen können unverändert beibehalten werden.
3. Führen Sie das Script wie folgt aus, um die Ressourcen in einer neuen VPC zu erstellen:

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   Zur Wiederverwendung einer vorhandenen VPC übergeben Sie ihren Namen wie folgt an das Script. Ersetzen Sie **IHRE_VORHANDENE_VPC** durch den tatsächlichen VPC-Namen.
   ```sh
   REUSE_VPC=IHRE_VORHANDENE_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. Dadurch werden die folgenden Ressourcen, einschließlich der zum Bastion-Host gehörigen Ressourcen, erstellt:
   - 1 VPC (optional)
   - 1 öffentliches Gateway
   - 3 Teilnetze innerhalb der VPC
   - 4 Sicherheitsgruppen mit Ingress- und Egress-Regeln
   - 3 VSIs: vpns2s-onprem-vsi (variable IP-Adress: ONPREM_IP), vpns2s-cloud-vsi (variable IP-Adresse: VSI_CLOUD_IP) und vpns2s-bastion (variable IP-Adresse: BASTION_IP_ADDRESS)

   Notieren Sie sich die zurückgegebenen Werte für **BASTION_IP_ADDRESS**, **VSI_CLOUD_IP**, **ONPREM_IP**, **CLOUD_CIDR** und **ONPREM_CIDR** für später. Die Ausgabe wird außerdem in der Datei **network_config.sh** gespeichert. Diese Datei kann für eine automatisierte Konfiguration verwendet werden.

### VPN-Gateway und Verbindung erstellen
Im folgenden Abschnitt werden Sie dem Teilnetz mit der Anwendungs-VSI ein VPN-Gateway und eine zugeordnete Verbindung hinzufügen.

1. Navigieren Sie zur Seite [VPC-Übersicht](https://{DomainName}/vpc/overview) und klicken Sie auf **VPNs** auf der Navigationsregisterkarte und im anschließenden Dialog auf **Neues VPN-Gateway**. Geben Sie im Formular **Neues VPN-Gateway für VPC** den Namen **vpns2s-gateway** ein. Stellen Sie sicher, dass die richtige VPC, die richtige Ressourcengruppe sowie als Teilnetz **vpns2s-cloud-subnet** ausgewählt werden.
2. Lassen Sie die Option **Neue VPN-Verbindung für VPC** aktiviert. Geben Sie **vpns2s-gateway-conn** als Namen ein.
3. Verwenden Sie als **Peer-Gateway-Adresse** die variable IP-Adresse von **vpns2s-onprem-vsi** (ONPREM_IP). Geben Sie im Feld **Vorab verteilter Schlüssel** den Wert **20_PRESHARED_KEY_KEEP_SECRET_19** ein.
4. Verwenden Sie für **Lokale Teilnetze** die Informationen, die für **CLOUD_CIDR** bereitgestellt wurden, und für **Peer-Teilnetze** die Information, die für **ONPREM_CIDR** bereitgestellt wurde.
5. Belassen Sie die Einstellungen in **Erkennung inaktiver Peers** unverändert. Klicken Sie auf **VPN-Gateway erstellen**, um das Gateway und die zugeordnete Verbindung zu erstellen.
6. Warten Sie, bis das VPN-Gateway verfügbar wird. (Sie müssen möglicherweise die Anzeige aktualisieren.)
7. Notieren Sie sich die zugewiesene **Gateway-IP-Adresse** als **GW_CLOUD_IP**. 

### Lokales VPN-Gateway erstellen
Als Nächstes werden Sie das VPN-Gateway an dem anderen Standort (Site) in der lokalen On-Premises-Umgebung erstellen. Dazu verwenden Sie die quelloffene IPsec-Software [strongSwan](https://strongswan.org/).

1. Stellen Sie eine Verbindung zu der lokalen On-Premises-VSI **vpns2s-onprem-vsi** mithilfe von 'ssh' her. Führen Sie den folgenden Befehl aus, indem Sie **ONPREM_IP** durch die IP-Adresse ersetzen, die zuvor zurückgegeben wurde.

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   Abhängig von Ihrer Umgebung ist es möglich, dass Sie den Befehl `ssh -i <pfad_zu_ihrer_datei_mit_dem_privaten_schlüssel> root@ONPREMP_IP` verwenden müssen.
   {:tip}

2. Führen Sie als Nächstes auf der Maschine **vpns2s-onprem-vsi** die folgenden Befehle aus, um den Paketmanager zu aktualisieren und die strongSwan-Software zu installieren.

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. Konfigurieren Sie die Datei **/etc/sysctl.conf**, indem Sie drei Zeilen am Ende hinzufügen. Kopieren Sie die folgenden Anweisungen hinüber und führen Sie sie aus:

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. Bearbeiten Sie als Nächstes die Datei **/etc/ipsec.secrets**. Fügen Sie die folgende Zeile hinzu, um die Quellen- und die Ziel-IP-Adresse sowie den zuvor konfigurierten vorab verteilten Schlüssel (PRESHARED_KEY) zu konfigurieren. Ersetzen Sie **ONPREM_IP** durch den bekannten Wert der variablen IP-Adresse der VSI 'vpns2s-onprem-vsi'. Ersetzen Sie **GW_CLOUD_IP** durch die bekannte IP-Adresse des VPC/VPN-Gateways.

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. Die letzte Datei, die Sie konfigurieren müssen, ist die Datei **/etc/ipsec.conf**. Fügen Sie am Ende dieser Datei den folgenden Codeblock hinzu. Ersetzen Sie **ONPREM_IP**, **ONPREM_CIDR**, **GW_CLOUD_IP** und **CLOUD_CIDR** durch die entsprechenden bekannten Werte.

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

6. Starten Sie das VPN-Gateway erneut und prüfen Sie seinen Status, indem Sie die folgenden Befehle ausführen:

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   Der Status sollte angeben, dass eine Verbindung eingerichtet wurde. Halten Sie die Terminal- und SSH-Verbindung zu dieser Maschine geöffnet.

## Konnektivität testen
Sie können die Site-zu-Site-VPN-Verbindung mithilfe von SSH oder durch Bereitstellen des Microservice, der mit {{site.data.keyword.cos_short}} kommuniziert, testen.

### Mit SSH testen
Zum Testen, ob die VPN-Verbindung erfolgreich eingerichtet wurde, verwenden Sie die simulierte On-Premises-Umgebung als Proxy, um sich am cloudbasierten Anwendungsserver anzumelden. 

1. Führen Sie in einem neuen Terminal den folgenden Befehl aus, indem Sie die Werte entsprechend ersetzen. Er verwendet den strongSwan-Host als Jump-Host, um eine Verbindung über VPN zu der privaten IP-Adresse des Anwendungsservers herzustellen.

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. Sobald die Verbindung erfolgreich hergestellt wurde, schließen Sie die SSH-Verbindung.

3. Stoppen Sie das VPN-Gateway im VSI-Terminal "onprem":
   ```sh
   ipsec stop
   ```
   {:pre}
4. Versuchen Sie im Befehlsfenster von Schritt 1, die Verbindung erneut herzustellen:

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
   Der Befehl sollte nicht erfolgreich ausgeführt werden, da die VPN-Verbindung nicht aktiv ist und daher keine direkte Verbindung zwischen der simulierten On-Premises-Umgebung und der Cloudumgebung besteht.

   Beachten Sie, dass diese Verbindung abhängig von den Bereitstellungsdetails trotzdem erfolgreich hergestellt wird. Dies hat den Grund, dass eine VPC-interne Konnektivität zwischen Zonen unterstützt wird. Wenn Sie die simulierte On-Premises-VSI in einer anderen VPC oder in [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers) bereitstellen würden, würde das VPN für einen erfolgreichen Zugriff benötigt.
   {:tip}
   
5. Starten Sie im VSI-Terminal "onprem" das VPN-Gateway erneut:
   ```sh
   ipsec start
   ```
   {:pre}
 

### Mit einem Microservice testen
Sie können die verwendete VPN-Verbindung testen, indem Sie auf einen Microservice in der Cloud-VSI über die VSI "onprem" zugreifen.

1. Kopieren Sie den Code für die Microservice-App von Ihrer lokalen Maschine in die Cloud-VSI. Der Befehl verwendet den Bastion-Host als Jump-Host für die Verbindung zu der Cloud-VSI. Ersetzen Sie **BASTION_IP_ADDRESS** und **VSI_CLOUD_IP** entsprechend.
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. Stellen Sie eine Verbindung zu der Cloud-VSI her, indem Sie wieder den Bastion-Host als Jump-Host verwenden.
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. Wechseln Sie in der Cloud-VSI in das Codeverzeichnis:
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. Installieren Sie Python und den Python-Paketmanager PIP.
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. Installieren Sie die erforderlichen Python-Pakete mit **pip**.
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. Starten Sie die App:
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. Greifen Sie im VSI-Terminal "onprem" auf den Service zu. Ersetzen Sie VSI_CLOUD_IP entsprechend.
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
   Der Befehl sollte ein JSON-Objekt zurückgeben.

## Ressourcen entfernen
{: #remove-resources}

1. Klicken Sie in der VPC-Managementkonsole auf **VPNs**. Wählen Sie im Aktionsmenü auf dem VPN-Gateway die Option **Löschen** aus, um das Gateway zu entfernen.
2. Klicken Sie als Nächstes auf **Variable IP-Adressen** in der Navigation und anschließend auf die IP-Adresse für Ihre VSIs. Wählen Sie im Aktionsmenü die Option **Freigeben** aus. Bestätigen Sie, dass die IP-Adresse freigegeben werden soll.
3. Wechseln Sie anschließend zu **Virtuelle Serverinstanzen** und **löschen** Sie Ihre Instanzen. Die Instanzen werden gelöscht und sie verbleiben eine Zeitlang im Status **Wird gelöscht**. Stellen Sie sicher, dass Sie den Browser ab und zu aktualisieren.
4. Sobald die VSIs entfernt wurden, wechseln Sie zu **Teilnetze**. Wenn das Teilnetz ein angehängtes öffentliches Gateway hat, klicken Sie auf den Teilnetznamen. Trennen Sie das öffentliche Gateway in den Teilnetzdetails. Teilnetze ohne öffentliches Gateway können über die Übersichtsseite gelöscht werden. Löschen Sie Ihre Teilnetze.
5. Wechseln Sie nach dem Löschen der Teilnetze zu **VPC** und löschen Sie Ihre VPC.

Wenn Sie die Konsole verwenden, müssen Sie Ihre Browseransicht möglicherweise aktualisieren, damit aktualisierte Statusinformationen nach dem Löschen einer Ressource angezeigt werden.
{:tip}

## Lernprogramm erweitern 
{: #expand-tutorial}

Möchten Sie diesem Lernprogramm etwas hinzufügen oder es erweitern? Hier sind einige Ideen:

- Fügen Sie eine [Lastausgleichsfunktion](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc) hinzu, um eingehenden Microservicedatenverkehr auf mehrere Instanzen zu verteilen.
- Stellen Sie die [Anwendung auf einem öffentlichen Server, Ihre Daten und Services auf einem privaten Host](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend) bereit.


## Zugehörige Inhalte
{: #related}

- [VPC-Glossar](/docs/vpc?topic=vpc-vpc-glossary)
- [IBM Cloud CLI-Plug-in für VPC-Referenz](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [VPC mit Verwendung der REST-APIs](/docs/infrastructure/vpc/example-code.html)
- Lernprogramm für Lösung: [Sicherer Zugriff auf ferne Instanzen mit einem Bastion-Host](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
