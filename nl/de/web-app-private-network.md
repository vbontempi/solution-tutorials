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

# Bereitstellung von Webanwendungen über ein sicheres privates Netz
{: #web-app-private-network}

Das Hosting von Webanwendungen ist ein gängiges Bereitstellungsmuster für eine öffentliche Cloud, bei dem Ressourcen bei Bedarf skaliert werden können, um kurz- und langfristige Nutzungsanforderungen zu erfüllen. Sicherheit für die Anwendungsworkloads ist eine fundamentale Voraussetzung, um die durch die öffentliche Cloud verfügbare Ausfallsicherheit und Skalierbarkeit zu vervollständigen. 

Dieses Lernprogramm führt Sie durch die Erstellung einer skalierbaren und sicheren, über das Internet verbundenen Webanwendung, die in einem privaten Netz gehostet wird und durch eine Virtual Router Appliance (VRA), durch VLANs, durch NAT sowie durch Firewalls geschützt wird. Die Anwendung umfasst eine Lastausgleichsfunktion, zwei Webanwendungsserver und einen MySQL-Datenbankserver. Sie kombiniert drei Lernprogramme, die veranschaulichen, wie Webanwendungen sicher auf der {{site.data.keyword.Bluemix_notm}} IaaS-Plattform mit klassischem Netzbetrieb bereitgestellt werden können.
{:shortdesc}

## Lernziele
{: #objectives}

- Virtuelle Server für die Installation von PHP und MySQL erstellen
- Lastausgleichsfunktion (Load Balancer) bereitstellen, die Anforderungen an die Anwendungsserver verteilt
- Eine Virtual Router Appliance (VRA) zur Erstellung eines sicheren Netzes bereitstellen
- VLANs und IP-Teilnetze definieren 
- Netz mithilfe von Firewallregeln schützen
- Network Address Translation (SNAT) für die Anwendungsbereitstellung nutzen

## Verwendete Services
{: #products}

In diesem Lernprogramm werden die folgenden {{site.data.keyword.Bluemix_notm}}-Services verwendet: 

* [Virtual Router Appliance-VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [Lastausgleichsfunktion]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

Für dieses Lernprogramm können Kosten anfallen. Die VRA ist nur im Rahmen eines monatlichen Preistarifs verfügbar.

## Architektur
{: #architecture}

<p style="text-align: center;">

  ![Architektur](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	Sicheres privates Netz konfigurieren
2.	NAT-Zugriff für die Anwendungsbereitstellung konfigurieren
3.	Skalierbare Web-App und Lastausgleichsfunktion bereitstellen

## Vorbereitende Schritte
{: #prereqs}

Dieses Lernprogramm greift auf drei vorhandene Lernprogramme zurück, die in bestimmter Reihenfolge bereitgestellt werden. Alle drei sollten vor Beginn durchgesehen werden:

-	[Workloads mit einem sicheren privaten Netz isolieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[NAT für den Internetzugriff über ein privates Netz konfigurieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[Virtuelle Server zum Erstellen einer hoch verfügbaren und skalierbaren Web-App verwenden]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## Sicheres privates Netz konfigurieren
{: #private_network}

Isolierte und sichere private Netzumgebungen spielen im IaaS-Anwendungssicherheitsmodell in der öffentlichen Cloud eine zentrale Rolle. Für die Erstellung isolierter privater Umgebungen sind Firewalls, VLANs, Routing-Funktionalität und VPNs erforderlich.
Der erste Schritt besteht darin, ein sicheres privates Netzgehäuse zu erstellen, in dem die Web-App bereitgestellt wird.  

- [Workloads mit einem sicheren privaten Netz isolieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

Dieses Lernprogramm kann ohne Änderung befolgt werden. In einem späteren Schritt werden drei virtuelle Maschinen in der APP-Zone für Nginx-Web-Server und für eine MySQL-Datenbank bereitgestellt. 

## NAT für sichere Anwendungsbereitstellung konfigurieren
{: #nat_config}

Die Installation von Open-Source-Anwendungen erfordert einen sicheren Zugriff auf das Internet, um auf die Quellenrepositorys zuzugreifen. Zum Schutz der Server in dem sicheren privaten Netz vor dem öffentlichen Internet wird Source Network Address Translation (Source NAT) verwendet, wodurch die Quellenadresse verschleiert wird. Firewallregeln werden zum Schutz der ausgehenden Anwendungsrepository-Anforderungen verwendet. Alle eingehenden Anforderungen werden abgelehnt. 

- [NAT für den Internetzugriff über ein privates Netz konfigurieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

Dieses Lernprogramm kann ohne Änderung befolgt werden. Im nächsten Schritt wird die NAT-Konfiguration verwendet, um auf die erforderlichen Nginx- und MySQL-Module zuzugreifen.  


## Skalierbare Web-App und Lastausgleichsfunktion bereitstellen
{: #scalable_app}

Eine Wordpress-Installation auf Nginx und MySQL und eine Lastausgleichsfunktion werden verwendet, um zu veranschaulichen, wie eine skalierbare und ausfallsichere Webanwendung im sicheren privaten Netz bereitgestellt werden kann. 

Dieses Lernprogramm führt Sie durch dieses Szenario mit Erstellung einer {{site.data.keyword.Bluemix_notm}}-Lastausgleichsfunktion, zwei Webanwendungsservern und einem MySQL-Datenbankserver. Die Server werden in der APP-Zone im sicheren privaten Netz bereitgestellt, um eine Firewalltrennung von anderen Workloads und dem öffentlichen Netz zu ermöglichen. 

- [Virtuelle Server zum Erstellen einer hoch verfügbaren und skalierbaren Web-App verwenden]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

Es gibt drei Änderungen gegenüber diesem Lernprogramm:

1.	Die in diesem Lernprogramm verwendeten virtuellen Server werden in dem VLAN und dem Teilnetz bereitgestellt, das durch die APP-Firewallzone hinter der VRA geschützt wird.
2. Geben Sie die &lt;Private VLAN-ID&gt; an, wenn Sie die virtuellen Server bestellen. Details zur Angabe der &lt;privaten VLAN-ID&gt; bei der Bestellung eines virtuellen Servers finden Sie in den Anweisungen unter [Ersten virtuellen Server bestellen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver) im Lernprogramm [Workloads mit einem sicheren privaten Netz isolieren]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). Wählen Sie außerdem unbedingt Ihren SSH-Schlüssel aus, der zuvor in dem Lernprogramm hochgeladen wurde, um den Zugriff auf die virtuellen Server zuzulassen. 
3. Es wird dringend empfohlen, den Dateispeicherservice für dieses Lernprogramm wegen der schwachen Leistung des Befehls 'rsync' beim Kopieren der Wordpress-Dateien in den gemeinsam genutzten Speicher **nicht** zu verwenden. Dies betrifft nicht das Lernprogramm insgesamt. Die Schritte zur Erstellung des Dateispeichers und zur Einrichtung von Mounts können für die App-Server und die Datenbank ignoriert werden. Alternativ müssen alle Schritte, die in [PHP-Anwendung auf den Anwendungsservern installieren und konfigurieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application) beschrieben werden, auf beiden App-Servern ausgeführt werden.
   Vor der Ausführung der Schritte in [PHP-Anwendung auf den Anwendungsservern installieren und konfigurieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application) erstellen Sie zunächst das Verzeichnis `/mnt/www/` auf beiden App-Servern. Dieses Verzeichnis wurde ursprünglich in dem jetzt entfernten Dateispeicherabschnitt erstellt. 

   ```sh
   mkdir /mnt/www
   ```

Am Ende dieses Schritts sollte sich die Lastausgleichsfunktion in einwandfreiem Zustand befinden und die Wordpress-Site sollte über das Internet zugänglich sein. Die virtuellen Server, die die Webanwendung bilden, werden gegen externen Zugriff über das Internet durch die VRA-Firewall geschützt und der Zugriff wird ausschließlich durch die Lastausgleichsfunktion zugelassen. Im Hinblick auf eine Produktionsumgebung sollten auch ein Schutz gegen DDoS-Angriffe und eine Web Application Firewall (WAF), wie sie durch [{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services) bereitgestellt wird, in Betracht gezogen werden.


## Ressourcen entfernen
{: #removeresources}

Nachfolgend werden die Schritte zum Entfernen der in diesem Lernprogramm erstellten Ressourcen beschrieben. 

Die VRA ist im Rahmen eines monatlichen Preistarifs verfügbar. Die Stornierung hat keine Rückerstattung zur Folge. Eine Stornierung dieser VRA wird nur empfohlen, wenn sie im nächsten Monat nicht erneut erforderlich sein wird.
{:tip}  

1. Stornieren Sie alle virtuellen Server oder Bare-Metal-Server.
2. Stornieren Sie die VRA.
3. Stornieren Sie alle zusätzlichen VLANs über ein Support-Ticket.
4. Löschen Sie die Lastausgleichsfunktion.
5. Löschen Sie die Dateispeicherservices.

## Lernprogramm erweitern 

1. In diesem Lernprogramm werden nur zwei virtuelle Server zu Anfang als App-Ebene bereitgestellt. Zur Verarbeitung zusätzlicher Auslastung könnten weitere Server automatisch hinzugefügt werden. Die [automatische Skalierung]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group) gibt Ihnen die Möglichkeit, den manuellen Skalierungsprozess zu automatisieren, der mit dem Hinzufügen und Entfernen virtueller Server zur Unterstützung Ihrer Geschäftsanwendungen verbunden ist.

2. Schützen Sie Ihre Benutzerdaten separat, indem Sie der VRA ein zweites privates VLAN und IP-Teilnetz hinzufügen, um eine Datenzone (DATA) für das Hosting des MySQL-Datenbankservers zu erstellen. Konfigurieren Sie Firewallregeln, die nur eingehenden IP-Datenverkehr für MySQL über Port 3306 aus der APP-Zone in die DATA-Zone zulassen. 

