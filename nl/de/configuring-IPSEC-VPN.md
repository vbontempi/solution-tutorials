---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
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

# VPN in ein sicheres privates Netzwerk
{: #configuring-IPSEC-VPN}

Die Erstellung einer privaten Verbindung zwischen einer fernen Netzumgebung und Servern in einem privaten Netz in {{site.data.keyword.Bluemix_notm}} ist eine gängige Anforderung. In der Regel unterstützt diese Verbindung hybride Workloads, Datenübertragungen, private Workloads oder die Verwaltung von Systemen unter {{site.data.keyword.Bluemix_notm}}. Dabei ist ein Site-to-Site-VPN-Tunnel (VPN, Virtual Private Network) der übliche Ansatz, die Verbindungen zwischen Netzen zu sichern. 

{{site.data.keyword.Bluemix_notm}} bietet eine Reihe von Optionen für die Site-to-Site-Konnektivität des Rechenzentrums, entweder über ein VPN über das öffentliche Internet oder über eine private dedizierte Netzverbindung. 

Der Abschnitt [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
enthält weitere Informationen zu dedizierten sicheren Netzverbindungen zu {{site.data.keyword.Bluemix_notm}}. Ein VPN über das öffentliche Internet ist eine kostengünstige Option, allerdings ohne Bandbreitengarantien. 

Es gibt zwei geeignete VPN-Optionen für die Konnektivität über das öffentliche Internet zu Servern, die unter {{site.data.keyword.Bluemix_notm}} bereitgestellt werden:

-	[IPsec-VPN]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[Virtual Router Appliance-VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Dieses Lernprogramm umfasst die Konfiguration eines Site-to-Site-IPsec-VPN mithilfe einer Virtual Router Appliance (VRA), um ein Teilnetz in einem Client-Rechenzentrum mit einem sicheren Teilnetz im privaten {{site.data.keyword.Bluemix_notm}}-Netz zu verbinden. 

Dieses Beispiel baut auf dem Lernprogramm [Workloads mit einem sicheren privaten Netz isolieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) auf. Es verwendet eine Site-to-Site-IPsec-VPN,
einen GRE-Tunnel und statisches Routing. Komplexere VPN-Konfigurationen, die dynamisches Routing (BGP usw.) und VTI-Tunnel verwenden, finden Sie in der [ergänzenden VRA-Dokumentation](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Lernziele
{: #objectives}

- Konfigurationsparameter für das IPSec-VPN dokumentieren
- IPSec-VPN für eine Virtual Router Appliance (VRA) konfigurieren
- Datenverkehr über einen GRE-Tunnel weiterleiten

## Verwendete Services
{: #products}

In diesem Lernprogramm werden die folgenden {{site.data.keyword.Bluemix_notm}}-Services verwendet: 

* [Virtual Router Appliance-VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Für dieses Lernprogramm können Kosten anfallen. Die VRA ist nur im Rahmen eines monatlichen Preistarifs verfügbar.

## Architektur
{: #architecture}

<p style="text-align: center;">

  ![Architektur](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	Die VPN-Konfiguration wird dokumentiert.
2.	Das IPSec-VPN wird auf einer VRA erstellt.
3.	Das Rechenzentrum-VPN und der Tunnel werden konfiguriert.
4.	Der GRE-Tunnel wird erstellt.
5.	Die statische IP-Route wird erstellt.
6.	Die Firewall wird konfiguriert. 

## Vorbereitende Schritte
{: #prereqs}

Dieses Lernprogramm verbindet das sichere private Gehäuse im Lernprogramm [Workloads mit einem sicheren privaten Netz isolieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) mit dem Rechenzentrum. Dieses Lernprogramm muss zuerst durchgeführt werden.

## VPN-Konfiguration dokumentieren
{: #Document_VPN}

Die Konfiguration einer IPsec-VPN-Site-zu-Site-Verbindung zwischen Ihrem Rechenzentrum und {{site.data.keyword.Bluemix_notm}} erfordert eine Koordination mit Ihrem Neworking-Team vor Ort, um viele der Konfigurationsparameter, den Tunneltyp und IP-Routing-Informationen zu ermitteln. Die Parameter müssen für eine funktionsfähige VPN-Verbindung eine exakte Übereinstimmung aufweisen. In der Regel legt Ihr Networking-Team vor Ort die Konfiguration fest, um vereinbarten Unternehmensstandards zu entsprechen. Außerdem kann das Team Ihnen die nötige IP-Adresse des VPN-Gateways des Rechenzentrums und Adressbereiche von zugänglichen Teilnetzen bereitstellen.

Vor Beginn der Konfiguration des VPN müssen die IP-Adressen der VPN-Gateways und die Teilnetzbereiche des IP-Netzes ermittelt werden und für die VPN-Konfiguration des Rechenzentrums und das sichere private Netzgehäuse in {{site.data.keyword.Bluemix_notm}} verfügbar sein. Diese sind in der folgenden Abbildung dargestellt, wobei die APP-Zone im sicheren Gehäuse über den IPSec-Tunnel mit den Systemen im IP-Teilnetz des Client-Rechenzentrums verbunden wird.   

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

Die folgenden Parameter müssen zwischen dem {{site.data.keyword.Bluemix_notm}}-Benutzer, der das VPN konfiguriert, und dem Networking-Team für das Rechenzentrum des Clients vereinbart und dokumentiert werden. In diesem Beispiel werden die fernen und lokalen Tunnel-IP-Adressen auf 192.168.10.1 und 192.168.10.2 gesetzt. Mit der Zustimmung des Networking-Teams kann ein beliebiges Teilnetz verwendet werden. 

| Element| Beschreibung|
|:------ |:--- | 
| &lt;ike-gruppenname&gt; | Zugewiesener Name der IKE-Gruppe für die Verbindung. |
| &lt;ike-verschlüsselung&gt; | Vereinbarter IKE-Verschlüsselungsstandard für die Verwendung zwischen {{site.data.keyword.Bluemix_notm}} und dem Rechenzentrum des Clients (normalerweise *aes256*). |
| &lt;ike-hashwert&gt; | Vereinbarter IKE-Hashwert zwischen {{site.data.keyword.Bluemix_notm}} und dem Rechenzentrum des Clients, in der Regel *sha1*. |
| &lt;ike-lebensdauer&gt; | IKE-Lebensdauer aus dem Rechenzentrum des Clients, in der Regel *3600*. |
| &lt;esp-gruppenname&gt; | Zugewiesener Name der ESP-Gruppe für die Verbindung. |
| &lt;esp-verschlüsselung&gt; | Vereinbarter ESP-Verschlüsselungsstandard zwischen {{site.data.keyword.Bluemix_notm}} und dem Rechenzentrum des Clients, in der Regel *aes256*. |
| &lt;esp-hashwert&gt; | Vereinbarter ESP-Hashwert zwischen {{site.data.keyword.Bluemix_notm}} und dem Rechenzentrum des Clients, in der Regel *sha1*. |
| &lt;esp-lebensdauer&gt; | ESP-Lebensdauer aus dem Rechenzentrum des Clients, in der Regel *1800*. |
| &lt;öffentliche vpn-ip des rechenzentrums&gt;  | Auf das Internet ausgerichtete IP-Adresse des VPN-Gateways im Rechenzentrum des Clients. | 
| &lt;öffentliche ip der VRA&gt; | Öffentliche IP-Adresse der VRA. |
| &lt;ferne tunnel-ip\/24&gt; | IP-Adresse, die dem fernen Ende des IPsec-Tunnels zugeordnet ist. Paar der IP-Adresse in einem Bereich, der nicht mit der IP-Cloud oder dem Rechenzentrum des Clients in Konflikt steht.   |
| &lt;lokale tunnel-ip\/24&gt; | IP-Adresse, die dem lokalen Ende des IPsec-Tunnels zugeordnet ist.   |
| &lt;teilnetz/cidr des rechenzentrums&gt; | IP-Adresse des Teilnetzes, auf das im Rechenzentrum des Clients oder im Rahmen von CIDR zugegriffen werden soll. |
| &lt;teilnetz/cidr der app-zone&gt; | IP-Netzadresse und CIDR eines App-Zonen-Teilnetzes aus dem Lernprogramm zum Erstellen von VRAs. | 
| &lt;gemeinsam genutzter geheimer schlüssel&gt; | Gemeinsam genutzter Verschlüsselungsschlüssel für die Verwendung zwischen {{site.data.keyword.Bluemix_notm}} und dem Rechenzentrum des Clients. |

## IPsec-VPN auf einer VRA konfigurieren
{: #Configure_VRA_VPN}

Wenn Sie das VPN in {{site.data.keyword.Bluemix_notm}} erstellen möchten, sind die Befehle und alle Variablen, die geändert werden müssen, unten mit &lt; &gt; hervorgehoben. Die Änderungen sind Zeile für Zeile für jede Zeile angegeben, die geändert werden muss. Die Werte stammen aus der
Tabelle. 

1. Stellen Sie eine Verbindung über SSH zur VRA her und wechseln Sie in den Bearbeitungsmodus (`[edit]`).
   ```bash
   SSH vyatta@<private ip-adresse der VRA>
   configure
   ```
   {: codeblock}
2. Erstellen Sie eine IKE-Gruppe (Internet Key Exchange).
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike-verschlüsselung>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike-hashwert>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lebensdauer>
   ```
   {: codeblock}
3. Erstellen Sie eine ESP-Gruppe (Encapsulating Security Payload).
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp-verschlüsselung>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp-hashwert>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lebensdauer>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. Definieren Sie die Site-to-Site-Verbindung.
   ```
   set security vpn ipsec site-to-site peer <öffentliche vpn-ip des rechenzentrums>  authentication mode pre-shared-secret
   set security vpn ipsec site-to-site peer <öffentliche vpn-ip des rechenzentrums>  authentication pre-shared-secret <gemeinsam genutzter geheimer schlüssel>
   set security vpn ipsec site-to-site peer <öffentliche vpn-ip des rechenzentrums>  connection-type initiate
   set security vpn ipsec site-to-site peer <öffentliche vpn-ip des rechenzentrums>  ike-group <ike-gruppenname>
   set security vpn ipsec site-to-site peer <öffentliche vpn-ip des rechenzentrums>  local-address <öffentliche ip der VRA>
   set security vpn ipsec site-to-site peer <öffentliche vpn-ip des rechenzentrums>  default-esp-group <esp-gruppenname>
   set security vpn ipsec site-to-site peer <öffentliche vpn-ip des rechenzentrums>  tunnel 1
   set security vpn ipsec site-to-site peer <öffentliche vpn-ip des rechenzentrums>  tunnel 1 protocol gre
   commit
   save
   ```
   {: codeblock}

## VPN und Tunnel des Rechenzentrums konfigurieren
{: #Configure_DC_VPN}

1. Das Networking-Team im Rechenzentrum des Clients konfiguriert eine identische IPsec-VPN-Verbindung, wobei die Parameter &lt;öffentliche vpn-ip des rechenzentrums&gt; und &lt;öffentliche ip der vra&gt;, die lokale und ferne Tunneladresse sowie auch die Parameter &lt;teilnetz/cidr des rechenzentrums&gt; und &lt;teilnetz/cidr der app-zone&gt; vertauscht sind. Die speziellen Konfigurationsbefehle im Rechenzentrum des Clients hängen vom Anbieter des VPN ab.
1. Überprüfen Sie, ob die öffentliche IP-Adresse des VPN-Gateways des Rechenzentrums über das Internet zugänglich ist, bevor Sie fortfahren:
   ```
   ping <öffentliche vpn-ip des rechenzentrums>
   ```
   {: codeblock}
2. Wenn die VPN-Konfiguration des Rechenzentrums abgeschlossen ist, sollte die IPsec-Verbindung automatisch angezeigt werden. Stellen Sie sicher, dass die Verbindung hergestellt wurde und der Status zeigt, dass mindestens ein aktiver IPsec-Tunnel vorhanden ist. Überprüfen Sie im Rechenzentrum, dass beide Enden des VPNs aktive IPsec-Tunnel aufweisen.
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. Wenn die Verbindung nicht erstellt wurde, überprüfen Sie mit dem Befehl 'debug', ob die lokalen und fernen Adressen korrekt angegeben wurden und sich die anderen Parameter wie erwartet verhalten:
   ``` bash
   show vpn debug
   ```
   {: codeblock}

Die Zeile `peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......` sollte in der Ausgabe vorhanden sein. Ist sie nicht vorhanden oder es wird der Status 'CONNECTING' angezeigt, weist die VPN-Konfiguration einen Fehler auf.  

## GRE-Tunnel definieren 
{: #Define_Tunnel}

1. Erstellen Sie den GRE-Tunnel im VRA-Bearbeitungsmodus.
   ```
   set interfaces tunnel tun0 address <lokale tunnel-ip/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <öffentliche ip der VRA>
   set interfaces tunnel tun0 remote-ip <öffentliche vpn-ip des Rechenzentrums>
   commit
   ```
   {: codeblock}
2. Nachdem die beiden Enden des Tunnels konfiguriert wurden, sollte dieser automatisch angezeigt werden. Überprüfen Sie den Betriebsstatus des Tunnels über die VRA-Befehlszeile.
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   Der erste Befehl sollte für den Tunnel den Status und die Verbindung als `u/u` (UP/UP) angeben. Der zweite Befehl zeigt mehr Details zum Tunnel und zum übertragenen und empfangenen Datenverkehr an.
3. Stellen Sie sicher, dass Datenverkehr über den Tunnel geleitet wird
   ```bash
   ping <ferne tunnel-ip>
   ```
   {: codeblock}
   Die TX- und RX-Zähler in einem Befehl `show interfaces tunnel tun0` sollten sich erhöhen, während der Datenverkehr mit `ping` überprüft wird.
4. Wenn der Datenverkehr nicht fließt, kann mithilfe der Befehle `monitor interface` überwacht werden, welcher Datenverkehr an den einzelnen Schnittstellen auftritt. Die Schnittstelle `tun0` zeigt den internen Datenverkehr über den Tunnel an. Die Schnittstelle `dp0bond1` zeigt den eingeschlossenen Datenfluss zum und vom fernen VPN-Gateway.
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

Wenn kein Rücklauf beobachtet wird, muss das Networking-Team des Rechenzentrums die Datenübertragung an den VPN- und Tunnelschnittstellen am fernen Standort überwachen, um das Problem zu lokalisieren. 

## Statische IP-Route erstellen
{: #Define_Routing}

Erstellen Sie ein VRA-Routing, um den Datenverkehr über den Tunnel in das ferne Teilnetz zu leiten.

1. Erstellen Sie eine statische Route im VRA-Bearbeitungsmodus.
   ```
   set protocols static route <teilnetz/cidr des rechenzentrums>  next-hop <ferne tunnel-ip>
   ```
   {: codeblock}   
2. Überprüfen Sie die VRA-Routing-Tabelle über die VRA-Befehlszeile. Zu diesem Zeitpunkt durchquert kein Datenverkehr die Route, da keine Firewallregeln vorhanden sind, um Datenverkehr über den Tunnel zuzulassen. Damit Datenverkehr auf einer der beiden Seiten initiiert werden kann, sind entsprechende Firewallregeln erforderlich.
   ```bash
   show ip route
   ```
   {: codeblock}

## Firewall konfigurieren
{: #Configure_firewall}

1. Erstellen Sie Ressourcengruppen für zulässigen ICMP-Datenverkehr und TCP-Ports. 
   ```
   set res group icmp-group icmpgrp type 8
   set res group icmp-group icmpgrp type 11
   set res group icmp-group icmpgrp type 3

   set res group port tcpports port 22
   set res group port tcpports port 80
   set res group port tcpports port 443
   commit
   ```
   {: codeblock}
2. Erstellen Sie Firewallregeln für den Datenverkehr zum fernen Teilnetz im VRA-Bearbeitungsmodus.
   ```
   set security firewall name APP-TO-TUNNEL default-action drop
   set security firewall name APP-TO-TUNNEL default-log

   set security firewall name APP-TO-TUNNEL rule 100 action accept
   set security firewall name APP-TO-TUNNEL rule 100 protocol tcp
   set security firewall name APP-TO-TUNNEL rule 100 destination port tcpports

   set security firewall name APP-TO-TUNNEL rule 200 protocol icmp
   set security firewall name APP-TO-TUNNEL rule 200 icmp group icmpgrp
   set security firewall name APP-TO-TUNNEL rule 200 action accept
   commit
   ```
   {: codeblock}

   ```
   set security firewall name TUNNEL-TO-APP default-action drop
   set security firewall name TUNNEL-TO-APP default-log

   set security firewall name TUNNEL-TO-APP rule 100 action accept
   set security firewall name TUNNEL-TO-APP rule 100 protocol tcp
   set security firewall name TUNNEL-TO-APP rule 100 destination port tcpports

   set security firewall name TUNNEL-TO-APP rule 200 protocol icmp
   set security firewall name TUNNEL-TO-APP rule 200 icmp group icmpgrp
   set security firewall name TUNNEL-TO-APP rule 200 action accept 
   commit
   ```
   {: codeblock}
2. Erstellen Sie eine Zone für den Tunnel und ordnen Sie Firewalls für den Datenverkehr zu, der in einer der Zonen initiiert wird.
   ```
   set security zone-policy zone TUNNEL description "GRE Tunnel"
   set security zone-policy zone TUNNEL default-action drop
   set security zone-policy zone TUNNEL interface tun0

   set security zone-policy zone TUNNEL to APP firewall TUNNEL-TO-APP
   set security zone-policy zone APP to TUNNEL firewall APP-TO-TUNNEL
   commit
   save
   ```
   {: codeblock}
3. Um sicherzustellen, dass die Firewalls und das Routing an beiden Enden ordnungsgemäß konfiguriert sind und keinen ICMP- und TCP-Datenverkehr zulassen, setzen Sie ein Pingsignal an die Gateway-Adresse des fernen Teilnetzes ab. Rufen Sie dazu zunächst die VRA-Befehlszeile auf und melden Sie sich dann nach erfolgreichem Pingsignal bei der VSI an.
   ```bash
   ping <gateway-ip des fernen teilnetzes>
   ssh root@<private ip der VSI>
   ping <gateway-ip des fernen teilnetzes>
   ```
   {: codeblock}
4. Wenn das Pingsignal von der VRA-Befehlszeile fehlschlägt, überprüfen Sie, ob eine Ping-Antwort als Antwort auf eine Ping-Anforderung in der Tunnelschnittstelle angezeigt wird.
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
   Keine Antwort weist auf ein Problem mit den Firewallregeln oder dem Routing im Rechenzentrum hin. Wenn eine Antwort in der Monitorausgabe vorhanden ist, jedoch eine Zeitlimitüberschreitung für das Pingsignal auftritt, überprüfen Sie die Konfiguration der lokalen VRA-Firewallregeln.
5. Wenn das Pingsignal von der VSI fehlschlägt, weist dies auf ein Problem mit den VRA-Firewallregel, dem Routing in der VRA oder in der VSI-Konfiguration hin. Führen Sie den vorherigen Schritt aus, um sicherzustellen, dass eine Anforderung an das Rechenzentrum gesendet und eine Antwort vom Rechenzentrum zurückgegeben wird. Die Überwachung des Datenverkehrs auf dem lokalen VLAN und die Überprüfung der Firewallprotokolle helfen dabei, das Problem auf das Routing oder die Firewall einzugrenzen.
   ```
   monitor interfaces bonding dp0bond0.<vlan-id>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

Damit wird die Konfiguration des VPN aus dem sicheren privaten Netzgehäuse abgeschlossen. Weitere Lernprogramme in dieser Reihe zeigen, wie das Gehäuse auf Services im öffentlichen Internet zugreifen kann.

## Ressourcen entfernen
{:removeresources}

Nachfolgend finden Sie die Schritte zum Entfernen der Ressourcen, die in diesem Lernprogramm erstellt wurden.

Die VRA ist im Rahmen eines monatlichen Zahlungsplans verfügbar. Bei eine Stornierung wird keine Erstattung geleistet. Eine Stornierung wird nur empfohlen, wenn diese VRA im nächsten Monat nicht mehr benötigt wird. Wenn ein Hochverfügbarkeitscluster mit zwei VRAs erforderlich ist, kann diese einzelne VRA auf der Seite mit den [Gateway-Details](https://{DomainName}/classic/network/gatewayappliances) aktualisiert werden.
{:tip}  

1. Stornieren Sie alle virtuellen Server oder Bare-Metal-Server.
2. Stornieren Sie die VRA.
3. Stornieren Sie alle zusätzlichen VLANs über ein Support-Ticket. 

## Zugehöriger Inhalt
{:related}
- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Statische und portierbare IP-Teilnetze](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Vyatta-Dokumentation](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS, Konfigurationshandbuch für IPsec Site-to-Site-VPN, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS, Konfigurationshandbuch für Tunnel, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
