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

# NAT für Internetzugriff aus einem privatem Netz konfigurieren
{: #nat-config-private}

In der heutigen Welt der webbasierten IT-Anwendungen und -Services werden nur wenige Anwendungen völlig eigenständig (isoliert) ausgeführt. Die Entwickler rechnen damit, dass auf Services im Internet zugegriffen wird (z. B. Open-Source-Anwendungscode und Aktualisierungen oder Services anderer Anbieter, die Anwendungsfunktionen über REST-APIs bereitstellen). Die Maskierung durch NAT (Network Address Translation) wird häufig verwendet, um den Zugriff aus privaten Netzen auf im Internet gehostete Services zu schützen. Bei der NAT-Maskierung werden private IP-Adressen in die IP-Adresse der öffentlichen Schnittstelle für abgehenden Datenverkehr in einer Viele-zu-eins-Beziehung umgesetzt, um die private IP-Adresse vor allgemeinem Zugriff zu schützen.  

In diesem Lernprogramm wird das Einrichten der NAT-Maskierung (NAT = Network Address Translation, Netzadressumsetzung) in einer Virtual Router Appliance (VRA) behandelt, um ein geschütztes Teilnetz im privaten {{site.data.keyword.Bluemix_notm}}-Netz zu verbinden. Es basiert auf dem Lernprogramm [Isolierte Workloads mit einem geschützten privaten Netz](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) und fügt eine SNAT-Konfiguration  (Source Network Address Translation) hinzu, um die Quellenadresse zu verschleiern und abgehenden Datenverkehr mithilfe von Firewallregeln zu schützen. Komplexere NAT-Konfigurationen enthält die [Ergänzende Dokumentation zu VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Ziele
{: #objectives}

-	Source Network Address Translation (SNAT) in einer Virtual Router Appliance (VRA) einrichten
-	Firewallregeln für den Internetzugang konfigurieren

## Verwendete Services
{: #products}

Im vorliegenden Lernprogramm werden die folgenden {{site.data.keyword.Bluemix_notm}}-Services verwendet: 

* [Virtual Router Appliance-VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Für dieses Lernprogramm können Kosten anfallen. Die Virtual Router Appliance (VRA) ist nur mit einem monatlichen Preistarif verfügbar.

## Architektur
{: #architecture}

<p style="text-align: center;">

  ![Architektur](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	Erforderliche Internet-Services dokumentieren
2.	NAT einrichten
3.	Internet-Firewallzone und Regeln erstellen

## Vorbereitende Schritte
{: #prereqs}

Dieses Lernprogramm ermöglicht Hosts in dem geschützten privaten Netzbereich, der im Lernprogramm [Workloads durch geschütztes privates Netz isolieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) erstellt wurde, den Zugang zu öffentlichen Internet-Services. Das angegebene Lernprogramm muss zuerst ausgeführt werden. 

## Internet-Services dokumentieren
{: #Document_outbound}

Der erste Schritt besteht darin, die Services zu ermitteln, auf die im öffentlichen Internet zugegriffen werden soll, und zu dokumentieren, welche Ports für den ausgehenden und eingehenden Datenverkehr aus dem Internet aktiviert werden müssen. Diese Liste der Ports wird in einem späteren Schritt für die Firewallregeln benötigt. 

Im vorliegenden Beispiel werden nur HTTP- und HTTPS-Ports aktiviert, da diese für die meisten Anforderungen ausreichen. DNS- und NTP-Services werden über das private {{site.data.keyword.Bluemix_notm}}-Netz bereitgestellt. Falls diese und andere Services (z. B. SMTP (Port 25) oder MySQL (Port 3306)) erforderlich sind, müssen zusätzliche Firewallregeln erstellt werden. Die beiden grundlegenden Portregeln lauten wie folgt:

-	Port 80 (http)
-	Port 443 (https)

Stellen Sie sicher, dass der Service eines anderen Anbieters das Whitelisting von Quellenadressen unterstützt. Wenn dies zutrifft, ist die öffentliche IP-Adresse der VRA erforderlich, um den Service eines anderen Anbieters so zu konfigurieren, dass der Zugang zu dem Service eingeschränkt wird. 


## NAT-Maskierung für Internetzugang 
{: #NAT_Masquerade}

Folgen Sie den hier angegebenen Anweisungen, um den externen Internetzugang für Hosts in der APP-Zone unter Verwendung der NAT-Maskierung zu konfigurieren. 

1.	Stellen Sie eine SSH-Verbindung zu VRA her und aktivieren Sie den Konfigurationsmodus \[edit\] (config).
   ```bash
   SSH vyatta@<Private IP-Adresse der VRA>
   configure
   ```
   {: codeblock}
2.	Erstellen Sie die SNAT-Regeln in der VRA und geben Sie dabei die `Teilnetz-Gateway-IP<CIDR>` an, die für das Teilnetz/VLAN der APP-Zone im vorherigen Lernprogramm für VRA-Bereitstellung festgelegt wurde. 
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Teilnetz-Gateway-IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## Firewalls erstellen
{: #Create_firewalls}

1.	Firewallregeln erstellen 
   ```
   set security firewall name APP-TO-OUTSIDE default-action drop
   set security firewall name APP-TO-OUTSIDE description 'APP traffic to the Internet'
   set security firewall name APP-TO-OUTSIDE default-log

   set security firewall name APP-TO-OUTSIDE rule 90 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 90 action accept
   set security firewall name APP-TO-OUTSIDE rule 90 destination port 80

   set security firewall name APP-TO-OUTSIDE rule 100 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 100 action accept
   set security firewall name APP-TO-OUTSIDE rule 100 destination port 443

   set security firewall name APP-TO-OUTSIDE rule 200 protocol icmp
   set security firewall name APP-TO-OUTSIDE rule 200 icmp type 8
   set security firewall name APP-TO-OUTSIDE rule 200 action accept
   commit
   ```
   {: codeblock}

## Zone erstellen und Regeln anwenden
{: #Create_zone}

1.	Erstellen Sie die Zone OUTSIDE, um den Zugang zum externen Internet zu steuern.
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	Ordnen Sie Firewalls zu, um den Datenverkehr zum und vom Internet zu steuern.
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE
   commit
   save
   ```
   {: codeblock}
3.	Überprüfen Sie, dass die virtuelle Serverinstanz (VSI) in der APP-Zone jetzt auf Services im Internet zugreifen kann. Melden Sie sich unter Verwendung von SSH bei der lokalen VSI an, und überprüfen Sie mit Ping und curl den ICMP- und TCP-Zugang zu Websites im Internet.  
   ```bash
   ssh root@<private IP der VSI>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## Ressourcen entfernen
{:removeresources}
Mit den folgenden Schritten können Sie die in diesem Lernprogramm erstellten Ressourcen entfernen. 

Die VRA wird mit einem monatlichen Preistarif bereitgestellt. Die Stornierung führt nicht zu einer Erstattung. Eine Stornierung wird nur empfohlen, wenn die betreffende VRA im Folgemonat nicht mehr benötigt wird. Wenn ein Hochverfügbarkeitscluster mit zwei VRAs erforderlich ist, kann für die bestehende einzelne VRA ein Upgrade auf der Seite [Gateway-Details](https://{DomainName}/classic/network/gatewayappliances) durchgeführt werden.
{:tip}  

1. Stornieren Sie alle virtuellen Server oder Bare-Metal-Server.
2. Stornieren Sie die VRA.
3. Stornieren Sie alle zusätzlichen VLANs mithilfe eines Support-Tickets. 

## Zugehörige Informationen
{:related}

-	[Netzadressumsetzung für VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[NAT-Maskierung]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[Ergänzende Dokumentation zu VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)

