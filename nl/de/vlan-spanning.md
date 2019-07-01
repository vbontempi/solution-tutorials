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

# Sichere private Netze über das IBM Netz verknüpfen
{: #vlan-spanning}

Mit steigendem Bedarf an globaler Reichweite und 24/7-Betrieb von Webanwendungen steigt auch die Notwendigkeit, Services in mehreren Cloudrechenzentren bereitzustellen. Rechenzentren, die über mehrere Standorte verteilt sind, bieten Ausfallsicherheit für den Fall eines Ausfalls in einer geografischen Region und bringen Workloads näher an global verteilte Benutzer heran, sodass sich Latenzzeiten verringern und die empfangbare Leistung erhöht. Das [{{site.data.keyword.Bluemix_notm}}-Netz](https://www.ibm.com/cloud/data-centers/) versetzt Benutzer in die Lage, Workloads, die in sicheren privaten und auf mehrere Rechenzentren und Standorte verteilten Netzen bereitgestellt werden, miteinander zu verknüpfen.

Dieses Lernprogramm veranschaulicht die Einrichtung einer privat weitergeleiteten IP-Verbindung über das private {{site.data.keyword.Bluemix_notm}}-Netz zwischen zwei sicheren privaten Netzen, die in verschiedenen Rechenzentren gehostet werden. Alle Ressourcen gehören zu ein und demselben {{site.data.keyword.Bluemix_notm}}-Konto. Es greift auf das Lernprogramm [Workloads mit einem sicheren privaten Netz isolieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) zurück, um zwei private Netze bereitzustellen, die über das private {{site.data.keyword.Bluemix_notm}}-Netz unter Verwendung des [VLAN-Spanning-Service]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning) sicher verknüpft werden.
{:shortdesc}

## Lernziele
{: #objectives}

- Sichere Netze innerhalb eines {{site.data.keyword.Bluemix_notm}} IaaS-Kontos verknüpfen
- Firewallregeln für den Zugriff zwischen Sites einrichten 
- Routing zwischen Sites konfigurieren

## Verwendete Services
{: #products}

In diesem Lernprogramm werden die folgenden {{site.data.keyword.Bluemix_notm}}-Services verwendet: 
* [Virtual Router Appliance (VRA)](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [VLAN-Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

Für dieses Lernprogramm können Kosten anfallen. Die VRA ist nur im Rahmen eines monatlichen Preistarifs verfügbar.

## Architektur
{: #architecture}

<p style="text-align: center;">

  ![Architektur](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. Sichere private Netze bereitstellen
2. VLAN-Spanning konfigurieren
3. IP-Routing zwischen privaten Netzen konfigurieren
4. Firewallregeln für Zugriff auf ferne Sites konfigurieren

## Vorbereitende Schritte
{: #prereqs}

Dieses Lernprogramm basiert auf dem Lernprogramm [Workloads mit einem sicheren privaten Netz isolieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). Dieses andere Lernprogramm und die entsprechenden Voraussetzungen sollten vor Beginn geprüft werden. 

## Sichere private Netzsites konfigurieren
{: #private_network}

Das Lernprogramm [Workloads mit einem sicheren privaten Netz isolieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) wird zweimal verwendet, um private Netze in zwei verschiedenen Rechenzentren zu implementieren. Es gibt keine Einschränkung in Bezug darauf, welche beiden Rechenzentren verwendet werden können, abgesehen davon, dass die latenzbedingte Beeinträchtigung des Datenverkehrs und der Workloads, die zwischen den Sites übertragen werden, berücksichtigt werden sollte. 

Das Lernprogramm [Workloads mit einem sicheren privaten Netz isolieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) kann ohne Änderung für jedes ausgewählte Rechenzentrum befolgt werden. Notieren Sie dabei jedoch die folgenden Informationen für spätere Schritte. 

| Element | Rechenzentrum1 (RZ1) | Rechenzentrum2 (RZ2) |
|:------ |:--- | :--- |
| Rechenzentrum |  |  |
| Öffentliche IP-Adresse der VRA | <öffentliche_ip-adresse_für_vra_in_rz1> | <öffentliche_ip-adresse_für_vra_in_rz1> |
| Private IP-Adresse der VRA | <private_ip-adresse_für_vra_in_rz1> | <private_ip-adresse_für_vra_in_rz2> |
| Privates Teilnetz und CIDR der VRA |  |  |
| Private VLAN-ID | &lt;private_vlan-id_in_rz1&gt;  | &lt;private_vlan-id_in_rz2&gt; |
| Private IP-Adresse der VSI | <private_ip-adresse_der_vsi_in_rz1> | <private_ip-adresse_der_vsi_in_rz2> |
| Teilnetz und CIDR der APP-Zone | <teilnetz/cidr_der_app-zone_in_rz1> | <teilnetz/cidr_der_app-zone_in_rz2> |

1. Fahren Sie mit der Seite 'Gateway-Details' für jede VRA über die Seite [Gateway-Appliances](https://{DomainName}/classic/network/gatewayappliances) fort.  
2. Suchen Sie den Abschnitt für Gateway-VLANs auf und klicken Sie auf 'Gateway [VLAN]( https://{DomainName}/classic/network/vlans)' im **privaten** Netz, um die VLAN-Details anzuzeigen. Der Name sollte die ID `bcrxxx` (für 'Back-End-Kundenrouter') enthalten und die Form `nnnxx.bcrxxx.xxxx` haben.
3. Die bereitgestellte VRA wird im Abschnitt **Geräte** angezeigt. Notieren Sie unter dem Abschnitt *Teilnetze** die IP-Adresse und das CIDR (/26) für das private Teilnetz der VRA. Das Teilnetz hat den primären Typ mit 64 IP-Adressen. Diese Details werden später für die Routing-Konfiguration benötigt. 
4. Suchen Sie ebenfalls auf der Seite für Gateway-Details den Abschnitt **Zugeordnete VLANs** auf und klicken Sie auf das [VLAN]( https://{DomainName}/classic/network/vlans) im **privaten** Netz, das zugeordnet wurde, um das sichere Netz und die APP-Zone zu erstellen. 
5. Die bereitgestellte VSI wird unter dem Abschnitt **Geräte** angezeigt. Notieren Sie sich unter dem Abschnitt *Teilnetze** die IP-Adresse und das CIDR (/26) für das VSI-Teilnetz, da diese Details für die Routing-Konfiguration benötigt werden. Dieses VLAN und Teilnetz wird als APP-Zone in beiden Firewallkonfigurationen für die VRA angegeben und als &lt;Teilnetz/CIDR der APP-Zone&gt; aufgezeichnet.


## VLAN-Spanning konfigurieren 
{: #configure-vlan-spanning}

Standardmäßig können Server (und VRAs) in verschiedenen VLANs und Rechenzentren über das private Netz nicht miteinander kommunizieren. In diesen Lernprogrammen werden VRAs in einem einzelnen Rechenzentrum dazu verwendet, VLANs und Teilnetze mit klassischem IP-Routing und Firewalls zu verknüpfen, um ein privates Netz für die Serverkommunikation über VLANs hinweg zu erstellen. Obwohl sie innerhalb desselben Rechenzentrums kommunizieren können, sind Server, die zu demselben {{site.data.keyword.Bluemix_notm}}-Konto gehören, in dieser Konfiguration nicht in der Lage, über Rechenzentren hinweg zu kommunizieren. 

Der [VLAN-Spanning-Service]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning) hebt diese Einschränkung für die Kommunikation zwischen den VLANs und Teilnetzen auf, die **keinen** VRAs zugeordnet sind. Es ist zu beachten, dass auch bei aktiviertem VLAN-Spanning VLANs, die VRAs zugeordnet sind, nur mit ihrer zugeordneten VRA kommunizieren können, wie dies durch die VRA-Firewall- und Routing-Konfiguration festgelegt wird. Wenn das VLAN-Spanning aktiviert ist, werden die VRAs, die zu einem {{site.data.keyword.Bluemix_notm}}-Konto gehören, über das private Netz verbunden, sodass sie kommunizieren können. 

VLANs, die den sicheren privaten Netzen nicht zugeordnet sind, die durch die VRAs erstellt wurden, werden ‘übergreifend’ verbunden, sodass eine gegenseitige Verbindung dieser ‘nicht zugeordneten’ VLANs über Rechenzentren hinweg möglich ist. Dies schließt die VRA-Gateway-VLANs (Übertragung) ein, die zu demselben IBM Cloud-Konto in verschiedenen Rechenzentren gehören. Dementsprechend wird zugelassen, dass VRAs über Rechenzentren hinweg kommunizieren können, wenn das VLAN-Spanning aktiviert ist. Mit der VRA-zu-VRA-Konnektivität wird die Verbindung von Servern durch die VRA-Firewall- und VRA-Routing-Konfiguration innerhalb des sicheren Netzes eingerichtet. 

Aktivieren Sie das VLAN-Spanning wie folgt:

1. Wechseln Sie zur Seite [VLANs]( https://{DomainName}/classic/network/vlans).
2. Wählen Sie die Registerkarte **Spanning** (Span) im oberen Bereich der Seite aus.
3. Wählen Sie das Optionsfeld ‘Ein’ (On) für das VLAN-Spanning aus. Die Ausführung dieser Netzänderung wird einige Minuten dauern.
4. Vergewissern Sie sich, dass die beiden VRAs jetzt kommunizieren können.

   Melden Sie sich bei der VRA von Rechenzentrum 1 an und prüfen Sie die Verbindung zur VRA von Rechenzentrum 2 mit Ping.

   ```
   SSH vyatta@<private_ip-adresse_für_vra_in_rz1>
   ping <private_ip-adresse_für_vra_in_rz2>
   ```
   {: codeblock}

   Melden Sie sich bei der VRA von Rechenzentrum 2 an und prüfen Sie die Verbindung zur VRA von Rechenzentrum 1 mit Ping.
   ```
   SSH vyatta@<private_ip-adresse_für_vra_in_rz2>
   ping <private_ip-adresse_für_vra_in_rz1>
   ```
   {: codeblock}

## IP-Routing für VRAs konfigurieren 
{: #vra_routing}

Erstellen Sie das VRA-Routing in jedem Rechenzentrum, um die VSIs in den APP-Zonen in beiden Rechenzentren für die Kommunikation einzurichten. 

1. Erstellen Sie eine statische Route in Rechenzentrum 1 zum privatem Teilnetz der APP-Zone in Rechenzentrum 2 im VRA-Bearbeitungsmodus.
   ```
   ssh vyatta@<private_ip-adresse_für_vra_in_rz1>
   conf
   set protocols static route <teilnetz/cidr_der_app-zone_in_rz2>  next-hop <private_ip-adresse_für_vra_in_rz2>
   commit
   ```
   {: codeblock}
2. Erstellen Sie eine statische Route in Rechenzentrum 2 zum privatem Teilnetz der APP-Zone in Rechenzentrum 1 im VRA-Bearbeitungsmodus.
   ```
   ssh vyatta@<private_ip-adresse_für_vra_in_rz2>
   conf
   set protocols static route <teilnetz/cidr_der_app-zone_in_rz1>  next-hop <private_ip-adresse_für_vra_in_rz1>
   commit
   ```
   {: codeblock}
2. Prüfen Sie die VRA-Routing-Tabelle über die VRA-Befehlszeile. Zu diesem Zeitpunkt können die VSIs nicht miteinander kommunizieren, da keine Firewallregeln für die APP-Zone vorhanden sind, um den Datenverkehr zwischen den beiden APP-Zonenteilnetzen zuzulassen. Firewallregeln sind für den Datenverkehr, der auf beiden Seiten initiiert wird, erforderlich.
   ```
   show ip route
   ```
   {: codeblock}

Die neue Route, die die Kommunikation der APP-Zone über das private IBM Netz ermöglicht, wird jetzt angezeigt. 

## VRA-Firewallkonfiguration
{: #vra_firewall}

Die vorhandenen Firewallregeln für die APP-Zone werden nur konfiguriert, um Datenverkehr in dieses Teilnetz und aus diesem Teilnetz an {{site.data.keyword.Bluemix_notm}}-Services im privaten {{site.data.keyword.Bluemix_notm}}-Netz sowie für den öffentlichen Internetzugriff über NAT zuzulassen. Andere Teilnetze, die VSIs über diese VRA zugeordnet werden oder die sich in anderen Rechenzentren befinden, werden blockiert. Der nächste Schritt besteht darin, die Ressourcengruppe `ibmprivate`, die der Firewallregel APP-TO-INSIDE zugeordnet ist, zu aktualisieren, um den Zugriff auf das Teilnetz in dem anderen Rechenzentrum explizit zuzulassen. 

1. Fügen Sie in Rechenzentrum 1 im VRA-Bearbeitungsmodus der Ressourcengruppe `ibmprivate` die Angabe <teilnetz/cidr_der_app-zone_in_rz2> hinzu.
   ```
   set resources group address-group ibmprivate address <teilnetz/cidr_der_app-zone_in_rz2>
   commit
   save
   ```
   {: codeblock}
2. Fügen Sie in Rechenzentrum 2 im VRA-Bearbeitungsmodus der Ressourcengruppe `ibmprivate` die Angabe <teilnetz/cidr_der_app-zone_in_rz1> hinzu.
   ```
   set resources group address-group ibmprivate address <teilnetz/cidr_der_app-zone_in_rz1>
   commit
   save
   ```
   {: codeblock}
3. Vergewissern Sie sich, dass die VSIs in beiden Rechenzentren jetzt kommunizieren können:
   ```bash
   ping <gateway-ip des fernen teilnetzes>
   ssh root@<private_ip-adresse_der_vsi>
   ping <gateway-ip des fernen teilnetzes>
   ```
   {: codeblock}

   Wenn die VSIs nicht kommunizieren müssen, führen Sie die Anweisungen im Lernprogramm [Workloads mit einem sicheren privaten Netz isolieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) aus, um den Datenverkehr über die Schnittstellen zu überwachen und die Firewallprotokolle zu prüfen. 

## Ressourcen entfernen
{: #removeresources}

Nachfolgend werden die Schritte zum Entfernen der in diesem Lernprogramm erstellten Ressourcen beschrieben. 

Die VRA ist im Rahmen eines monatlichen Preistarifs verfügbar. Die Stornierung hat keine Rückerstattung zur Folge. Eine Stornierung dieser VRA wird nur empfohlen, wenn sie im nächsten Monat nicht erneut erforderlich sein wird. Wenn ein dualer VRA-Hochverfügbarkeitscluster erforderlich ist, kann ein Upgrade für diese einzelne VRA über die Seite [Gateway-Details](https://{DomainName}/classic/network/gatewayappliances) durchgeführt werden.
{:tip}

1. Stornieren Sie alle virtuellen Server oder Bare-Metal-Server.
2. Stornieren Sie die VRA.
3. Stornieren Sie alle zusätzlichen VLANs über ein Support-Ticket. 


## Lernprogramm erweitern

Dieses Lernprogramm kann in Verbindung mit dem Lernprogramm [VPN in ein sicheres privates Netz](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network) verwendet werden, um beide sicheren Netze mit dem fernen Netz eines Benutzers über ein IPsec-VPN zu verbinden. VPN-Verbindungen können zu beiden sicheren Netzen eingerichtet werden, um die Ausfallsicherheit des Zugriffs auf die {{site.data.keyword.Bluemix_notm}} IaaS-Plattform zu erhöhen. Beachten Sie, dass IBM keine Weiterleitung von Benutzerdatenverkehr zwischen Clientrechenzentren über das private IBM Netz zulässt. Die Routing-Konfiguration zur Vermeidung von Netzschleifen liegt außerhalb des Rahmens dieses Lernprogramms. 


## Referenzliteratur
{:related}

1. Virtual Routing and Forwarding (VRF) ist eine Alternative zur Verwendung des VLAN-Spannings für das Verbinden von Netzen in einem {{site.data.keyword.Bluemix_notm}}-Konto. VRF ist für alle Clients obligatorisch, die [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview) verwenden. [Übersicht über VRF (Virtual Routing and Forwarding) in IBM Cloud](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [IBM Cloud-Netz](https://www.ibm.com/cloud/data-centers/)
