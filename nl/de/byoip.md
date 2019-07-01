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

# Bring your own IP (BYOIP)
{: #byoip}

Bring Your own IP (BYOIP) ist eine häufige Voraussetzung, wenn es erforderlich ist, bestehende Clientnetze mit der Infrastruktur zu verbinden, die unter {{site.data.keyword.Bluemix_notm}} bereitgestellt wird. Die Absicht besteht in der Regel darin, die Änderungen an den Netzweiterleitungskonfigurationen und -operationen des Clients mit der Annahme eines einzigen IP-Adressraums auf der Basis des vorhandenen IP-Adressierungsschemas auf ein Minimum zu reduzieren.

Dieses Lernprogramm enthält eine kurze Übersicht über die BYOIP-Implementierungsmuster, die mit {{site.data.keyword.Bluemix_notm}} verwendet werden können, und eine Entscheidungsstruktur zum Identifizieren des geeigneten Musters bei der Realisierung des sicheren Gehäuses, wie im Lernprogramm zum [Isolieren von Workloads mit einem sicheren privaten Netz](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) beschrieben. Für die Konfiguration ist möglicherweise zusätzlicher Input von Ihrem Networking-Team vor Ort, dem technischen {{site.data.keyword.Bluemix_notm}}-Support oder IBM Services erforderlich.

{:shortdesc}

## Lernziele
{: #objectives}

* BYOIP-Implementierungsmuster verstehen
* Implementierungsmuster für {{site.data.keyword.Bluemix_notm}} auswählen

## IP-Adressierung in {{site.data.keyword.Bluemix_notm}}

{{site.data.keyword.Bluemix_notm}} verwendet eine Reihe von privaten Adressbereichen, insbesondere 10.0.0.0/8, die in einigen Fällen mit vorhandenen Clientnetzen in Konflikt stehen können. Falls ein solcher Konflikt existiert, gibt es eine Reihe von Mustern, die BYOIP unterstützen, um die Interoperabilität mit dem IBM Cloud-Netz zu gewährleisten.

-	Netzadressumsetzung (NAT, Network Address Translation)
-	Tunnelung mit GRE (Generic Routing Encapsulation)
-	GRE-Tunnelung mit IP-Alias
-	Virtuelles Overlay-Netz

Die Auswahl des Musters wird durch die Anwendungen bestimmt, die unter {{site.data.keyword.Bluemix_notm}} gehostet werden sollen. Es gibt zwei Schlüsselaspekte: die Anwendungssensitivität bei der Musterimplementierung und das Ausmaß der Überlappung von Adressbereichen zwischen dem Clientnetz und {{site.data.keyword.Bluemix_notm}}. Zusätzliche Überlegungen sollten auch dann angestellt werden, wenn eine dedizierte private Netzverbindung zu {{site.data.keyword.Bluemix_notm}} verwendet werden soll. Die Dokumentation für [{{site.data.keyword.BluDirectLink}}
](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link) ist eine Leseempfehlung für alle Benutzer, die BYOIP in Betracht ziehen. Für {{site.data.keyword.BluDirectLink}}-Benutzer sollte statt den hier dargestellten Informationen die zugehörige Anleitung befolgt werden.

## Übersicht über Implementierungsmuster
{: #patterns_overview}

1. **NAT**: NAT-Adressumsetzung am On-Premises-Client-Router. Führen Sie On-Premises-NAT durch, um das Client-Adressierungsschema in die von {{site.data.keyword.Bluemix_notm}} zugewiesenen IP-Adressen in bereitgestellte IaaS-Services umzusetzen.  
2. **GRE-Tunnelung**: Das Adressierungsschema wird vereinheitlicht, indem der IP-Datenverkehr über einen GRE-Tunnel zwischen {{site.data.keyword.Bluemix_notm}} und dem On-Premise-Netz (normalerweise über VPN) weitergeleitet wird. Dies ist das in [diesem Lernprogramm](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN) dargestellte Szenario. 

   Je nachdem, in welchem Maß sich Adressräume überschneiden können, gibt es zwei Untermuster.
     * Keine Adressüberschneidung, wenn es bei den Adressbereichen keine Überschneidung und somit kein Risiko eines Konflikts zwischen Netzen gibt.
     * Teilweise Überschneidung von Adressen, wenn der Client und IP-Adressräume in {{site.data.keyword.Bluemix_notm}} denselben Adressbereich verwenden und somit die Gefahr von Überschneidungen und Konflikten besteht. In diesem Fall werden Clientteilnetzadressen ausgewählt, die sich im privaten {{site.data.keyword.Bluemix_notm}}-Netz nicht überschneiden.

3. GRE-Tunnelung + IP-Alias
Das Adressierungsschema wird vereinheitlicht, indem der IP-Datenverkehr über einen GRE-Tunnel zwischen dem On-Premises-Netz und IP-Adressen geleitet wird, die Servern in {{site.data.keyword.Bluemix_notm}} zugewiesen sind. Dies ist ein Sonderfall des Szenarios, das in [diesem Lernprogramm](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN) veranschaulicht wird. Eine zusätzliche Schnittstelle und ein IP-Aliasname für ein kompatibles IP-Teilnetz werden auf den virtuellen und Bare-Metal-Servern erstellt, die in {{site.data.keyword.Bluemix_notm}} bereitgestellt und von einer entsprechenden Weiterleitungskonfiguration auf der Virtual Router Appliance (VRA) unterstützt werden.

4. Virtuelles Overlay-Netz
[{{site.data.keyword.Bluemix_notm}} Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) unterstützt BYOIP für vollständig virtuelle Umgebungen in {{site.data.keyword.Bluemix_notm}}. Es könnte als Alternative zum sicheren privaten Netzgehäuse betrachtet werden, das in [diesem Lernprogramm](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) beschrieben ist.

Alternativ können Sie eine Lösung wie VMware NSX verwenden, die ein virtuelles Overlay-Netz in einem Layer über dem [{{site.data.keyword.Bluemix_notm}}-Netz implementiert. Alle BYOIP-Adressen im virtuellen Overlay sind unabhängig von [{{site.data.keyword.Bluemix_notm}}-Netzadressbereichen. Weitere Informationen finden Sie in der [Einführung zu VMware und [{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud).

## Entscheidungsstruktur für Implementierungsmuster
{: #decision_tree}

Die nachfolgende Entscheidungsstruktur kann zum Festlegen des geeigneten Implementierungsmusters verwendet werden. 

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

Die folgenden Anmerkungen dienen als weitere Orientierungshilfe:

### Ist NAT problematisch für Ihre Anwendungen?
{: #nat_consideration}

Es gibt zwei bestimmte Fälle, bei denen NAT problematisch sein könnte. In diesen Fällen sollte NAT nicht verwendet werden. 

- Einige Anwendungen wie die Microsoft AD-Domänenkommunikation und P2P-Anwendungen könnten technische Probleme mit NAT haben.
- Wenn unbekannte Server mit [{{site.data.keyword.Bluemix_notm}} kommunizieren müssen oder Hunderte von bidirektionalen Verbindungen zwischen [{{site.data.keyword.Bluemix_notm}} und lokalen Servern erforderlich sind. In diesem Fall kann die gesamte Zuordnung nicht für den Client-Router/die NAT-Tabelle konfiguriert werden, da die Zuordnung im Vorfeld nicht ermittelt werden konnte.


### Keine Adressüberschneidung
{: #no-overlap}

Wird im lokalen Netz 10.0.0.0/8 verwendet? Wenn keine Adressüberschneidungen zwischen dem lokalen und dem privaten [{{site.data.keyword.Bluemix_notm}}-Netz vorhanden ist, kann die in [diesem Lernprogramm](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN) beschriebene GRE-Tunnelung zwischen lokalen und IBM Cloud-Instanzen verwendet werden, um die Notwendigkeit einer NAT-Übersetzung zu vermeiden. Dafür muss die Nutzung der Netzadressen mit einem Networking-Team vor Ort überprüft werden. 

### Partielle Adressüberschneidung
{: #partial_overlap}

Wenn ein Teil des 10.0.0.0/8-Bereichs im lokalen Netz verwendet wird, gibt es nicht überlappende Teilnetze im [{{site.data.keyword.Bluemix_notm}}-Netz? Überprüfen Sie die Nutzung bestehender Netzadressen mit dem Networking-Tteam vor Ort und wenden Sie sich an die technische Vertriebsabteilung von [{{site.data.keyword.Bluemix_notm}}, um verfügbare, sich nicht überlappende Netze zu ermitteln. 

### Ist IP-Aliasing problematisch?
{: #ip_alias}

Wenn keine überschneidungssicheren Adressen vorhanden sind, kann das IP-Aliasing auf virtuellen und Bare-Metal-Servern implementiert werden, die in dem sicheren privaten Netzgehäuse bereitgestellt werden. Das IP-Aliasing ordnet mehrere Teilnetzadressen in einer oder mehreren Netzschnittstellen auf jedem Server zu. 

Das IP-Aliasing ist ein gängiges Verfahren. Es wird jedoch empfohlen, die Server- und Anwendungskonfigurationen zu überprüfen, um festzustellen, ob sie gut in Konfigurationen mit mehreren Ausgangsservern und IP-Aliasnamenkonfigurationen funktionieren.  

Für die Erstellung dynamischer Routen (z. B. BGP) oder statische Routen für die BYOIP-Teilnetze ist eine zusätzliche Routing-Konfiguration auf der VRA erforderlich. 

## Zugehöriger Inhalt
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
