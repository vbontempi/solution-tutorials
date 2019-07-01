---
copyright:
  years: 2019
lastupdated: "2019-04-02"
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
{:important: .important}

# Private und öffentliche Teilnetze in einer virtuellen privaten Cloud (VPC)
{: #vpc-public-app-private-backend}

IBM akzeptiert seit April 2019 eine begrenzte Anzahl von Kunden, die an einem Early Access-Programm für Virtual Private Cloud (VPC) teilnehmen, wobei eine erweiterte Nutzung in den folgenden Monaten eröffnet werden soll. Wenn Ihre Organisation Zugriff auf IBM Virtual Private Cloud erhalten möchte, füllen Sie bitte dieses [Nominierungsformular](https://{DomainName}/vpc){: new_window} aus. Anschließend wird ein IBM Ansprechpartner Kontakt mit Ihnen in Bezug auf die weiteren Schritte aufnehmen.
{: important}

Dieses Lernprogramm führt Sie durch die Erstellung einer eigenen {{site.data.keyword.vpc_full}} (VPC) mit einem öffentlichen und einem privaten Teilnetz sowie einer virtuellen Serverinstanz (VSI) in jedem Teilnetz. Eine VPC ist Ihre eigene, private Cloud in einer gemeinsam genutzten Cloudinfrastruktur mit logischer Isolation von anderen virtuellen Netzen.

Ein [Teilnetz](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#subnet) ist ein IP-Adressbereich. Es ist an eine einzelne Zone gebunden und kann sich nicht über mehrere Zonen oder Regionen erstrecken. Für die Zwecke einer VPC ist das wichtige Merkmal eines Teilnetzes die Tatsache, dass Teilnetze voneinander isoliert werden können und auch miteinander auf normale Weise verbunden werden können. Die Teilnetzisolation lässt sich durch Network [Access Control Lists](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#access-control-list) (ACLs - Netzzugriffssteuerlisten) realisieren, die als Firewalls zur Steuerung des Datenpaketflusses zwischen den Teilnetzen fungieren. In ähnlicher Weise fungieren [Sicherheitsgruppen](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#security-group) (SGs) als virtuelle Firewalls zur Steuerung des Datenpaketflusses an und von einzelnen virtuellen Serverinstanzen (VSIs).

Das öffentliche Teilnetz wird für Ressourcen verwendet, die für die Außenwelt zugänglich gemacht werden müssen. Ressourcen mit eingeschränktem Zugrif, auf die von der Außenwelt nie direkt zugegriffen werden sollte, werden in dem privaten Teilnetz positioniert. Instanzen in einem solchen Teilnetz könnten Ihre Back-End-Datenbank oder ein geheimer Speicher sein, der nicht öffentlich zugänglich sein soll. Sie werden Sicherheitsgruppen (SGs) definieren, um Datenverkehr an die VSIs zuzulassen oder zu verweigern.
{:shortdesc}

VPC bietet Ihnen zusammengefasst die folgenden Möglichkeiten:

- Softwaredefiniertes Netz (SDN) erstellen
- Workloads isolieren
- Differenzierte Steuerung des ein- und ausgehenden Datenverkehrs

## Lernziele

{: #objectives}

- Für virtuelle private Clouds verfügbare Infrastrukturobjekte kennen lernen
- Vorgehensweise zur Erstellung einer virtuellen privaten Cloud sowie von Teilnetzen und Serverinstanzen kennen lernen
- Anwendung von Sicherheitsgruppen zum Schutz des Zugriffs auf die Server kennen lernen

## Verwendete Services

{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet: 

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um einen Kostenvoranschlag auf der Grundlage der von Ihnen projektierten Nutzung zu generieren.

## Architektur
{: #architecture}

![Architektur](images/solution40-vpc-public-app-private-backend/Architecture.png)


1. Der Administrator (DevOps) richtet die erforderliche Infrastruktur (VPC, Teilnetze, Sicherheitsgruppen mit Regeln, VSIs) in der Cloud ein.
2. Der Internetbenutzer sendet eine HTTP/HTTPS-Anforderung an den Web-Server auf dem Front-End.
3. Das Front-End fordert private Ressourcen aus dem geschützten Back-End an und stellt dem Benutzer Ergebnisse zu.

## Vorbereitende Schritte

{: #prereqs}

- Überprüfen Sie die Benutzerberechtigungen. Stellen Sie sicher, dass Ihr Benutzerkonto über die entsprechenden Berechtigungen verfügt, um VPC-Ressourcen erstellen und verwalten zu können. Eine Liste der erforderlichen Berechtigungen finden Sie unter [Benötigte Berechtigungen für VPC-Benutzer erteilen](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).

- Sie benötigen einen SSH-Schlüssel für die Herstellung einer Verbindung zu den virtuellen Servern. Wenn Sie keinen SSH-Schlüssel haben, befolgen Sie die [Anweisungen zum Erstellen eines Schlüssels](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).

## Virtuelle private Cloud erstellen
{: #create-vpc}

Gehen Sie wie folgt vor, um eine eigene {{site.data.keyword.vpc_short}} zu erstellen:

1. Navigieren Sie zur Seite [VPC-Übersicht](https://{DomainName}/vpc/overview) und klicken Sie auf **VPC erstellen**.
2. Unter dem Abschnitt **Neue virtuelle private Cloud**:
   * Geben Sie **vpc-pubpriv** als Namen für Ihre VPC ein.
   * Wählen Sie eine **Ressourcengruppe** aus.
   * Fügen Sie optional **Tags** hinzu, um Ihre Ressourcen zu organisieren.
3. Wählen Sie **Neuen Standard erstellen (Alle zulassen)** als Ihre VPC-Standardzugriffssteuerungsliste (ACL) aus.
1. Wählen Sie SSH ab und überprüfen Sie die **Standardsicherheitsgruppe** mit Ping.
4. Unter **Neues Teilnetz für VPC**:
   * Geben Sie **vpc-pubpriv-backend-subnet** als eindeutigen Namen ein.
   * Wählen Sie eine Position (Standort) aus.
   * Geben Sie den IP-Adressbereich für das Teilnetz in CIDR-Notation ein, das heißt **10.xxx.0.0/24**. Belassen Sie die Option **Adresspräfix** so, wie sie ist, und wählen Sie für **Anzahl Adressen** den Wert 256 aus.
5. Wählen Sie die Option **VPC-Standdard verwenden** für Ihre Teilnetzzugriffssteuerliste (ACL) aus. Sie können die Regeln für ein- und ausgehenden Datenverkehr später konfigurieren.
6. Klicken Sie auf **Virtuelle private Cloud erstellen**, um die Instanz bereitzustellen.

Wenn die VSIs, die dem privaten Teilnetz zugeordnet sind, zum Laden von Software Zugriff auf das Internet benötigen, schalten Sie das öffentliche Gateway auf **Angehängt** (Attached), weil das Anhängen eines öffentlichen Gateways die Kommunikation zwischen allen angehängten Ressourcen und dem öffentlichen Internet ermöglicht. Sobald die VSIs sämtliche erforderliche Software haben, schalten Sie das öffentliche Gateway wieder auf **Getrennt** (Detached), sodass das Teilnetz das öffentliche Internet nicht erreichen kann.
{: important}

Zur Überprüfung der Erstellung des Teilnetzes klicken Sie auf **Teilnetze** im linken Teilfenster und warten Sie, bis sich der Status in **Verfügbar** ändert. Sie können ein neues Teilnetz unter **Teilnetze** erstellen.

## Back-End-Sicherheitsgruppe und VSI erstellen
{: #backend-subnet-vsi}

In diesem Abschnitt erstellen Sie eine Sicherheitsgruppe und eine virtuelle Serverinstanz für das Back-End.

### Back-End-Sicherheitsgruppe erstellen

Standardmäßig wird eine Sicherheitsgruppe zusammen mit Ihrer VPC erstellt, die jeden SSH-Zugriff (TCP-Port 22) und Ping-Zugriff (ICMP-Typ 8) auf die angehängten Instanzen zulässt.

Gehen Sie wie folgt vor, um eine neue Sicherheitsgruppe für das Back-End zu erstellen:  
1. Klicken Sie auf **Sicherheitsgruppen** unter **Netz** und anschließend auf **Neue Sicherheitsgruppe**.  
2. Geben Sie **vpc-pubpriv-backend-sg** als Namen ein und wählen Sie die VPC aus, die Sie zuvor erstellt haben.  
3. Klicken Sie auf **Sicherheitsgruppe erstellen**.

Sie werden die Sicherheitsgruppe später bearbeiten, um die Regeln für ein- und ausgehenden Datenverkehr hinzuzufügen.

### Virtuelle Serverinstanz für das Back-End erstellen

Gehen Sie wie folgt vor, um eine virtuelle Serverinstanz in dem neu erstellten Teilnetz zu erstellen:

1. Klicken Sie auf das Back-End-Teilnetz unter **Teilnetze**.
2. Klicken Sie auf **Angehängte Instanzen** und anschließend auf **Neue Instanz**.
3. Geben Sie einen eindeutigen Namen ein und wählen Sie **vpc-pubpriv-backend-vsi** aus. Wählen Sie anschließend die VPC, die Sie zuvor erstellt haben, und die **Position** (Standort) wie zuvor aus.
4. Wählen Sie das **Ubuntu-Linux-Image** aus, klicken Sie auf **Alle Profile** und wählen Sie unter **Datenverarbeitung** (Compute) die Option **c-2x4** mit 2vCPUs und 4 GB RAM aus.
5. Wählen Sie für **SSH-Schlüssel** den SSH-Schlüssel aus, den Sie zuvor erstellt haben.
6. Klicken Sie unter **Netzschnittstellen** auf das Symbol **Bearbeiten** neben den Sicherheitsgruppen.
   * Wählen Sie **vpc-pubpriv-backend-subnet** als Teilnetz aus.
   * Wählen Sie die Standardsicherheitsgruppe ab und wählen Sie **vpc-pubpriv-backend-sg** als aktiv aus.
   * Klicken Sie auf **Speichern**.
7. Klicken Sie auf **Virtuelle Serverinstanz erstellen**.

## Front-End-Teilnetz, Sicherheitsgruppe und VSI erstellen
{: #frontend-subnet-vsi}

Ähnlich wie für das Back-End erstellen Sie ein Front-End-Teilnetz mit einer virtuellen Serverinstanz und einer Sicherheitsgruppe.

### Teilnetz für das Front-End erstellen

Gehen Sie wie folgt vor, um ein neues Teilnetz für das Front-End zu erstellen:

1. Klicken Sie auf **Teilnetze** unter **Netz** im linken Teilfenster und anschließend auf **Neues Teilnetz**.
   * Geben Sie **vpc-pubpriv-frontend-subnet** als Namen ein und wählen Sie dann die VPC aus, die Sie erstellt haben.
   * Wählen Sie eine Position (Standort) aus.
   * Geben Sie den IP-Adressbereich für das Teilnetz in CIDR-Notation ein, das heißt **10.xxx.1.0/24**. Belassen Sie die Option **Adresspräfix** so, wie sie ist, und wählen Sie für **Anzahl Adressen** den Wert 256 aus.
1. Wählen Sie **VPC-Standard** für Ihre Teilnetzzugriffssteuerliste (ACL) aus. Sie können die Regeln für ein- und ausgehenden Datenverkehr später konfigurieren.
1. Da allen virtuellen Serverinstanzen im Teilnetz eine variable IP-Adresse zugeordnet werden soll, ist es nicht erforderlich, ein öffentliches Gateway für das Teilnetz zu aktivieren. Die virtuellen Serverinstanzen erhalten ihre Internetkonnektivität über ihre variable IP-Adresse.
1. Klicken Sie auf **Teilnetz erstellen**, um es bereitzustellen.

### Front-End-Sicherheitsgruppe erstellen

Gehen Sie wie folgt vor, um eine neue Sicherheitsgruppe für das Front-End zu erstellen:
1. Klicken Sie auf **Sicherheitsgruppen** unter **Netz** und anschließend auf **Neue Sicherheitsgruppe**.
2. Geben Sie **vpc-pubpriv-frontend-sg** als Namen ein und wählen Sie die VPC aus, die Sie zuvor erstellt haben.
3. Klicken Sie auf **Sicherheitsgruppe erstellen**.

### Virtuelle Serverinstanz für das Front-End erstellen

Gehen Sie wie folgt vor, um eine virtuelle Serverinstanz in dem neu erstellten Teilnetz zu erstellen:

1. Klicken Sie auf das Front-End-Teilnetz unter **Teilnetze**.
2. Klicken Sie auf **Angehängte Instanzen** und anschließend auf **Neue Instanz**.
3. Geben Sie einen eindeutigen Namen (**vpc-pubpriv-frontend-vsi**) ein und wählen Sie die VPC, die Sie zuvor erstellt haben, und die **Position** (Standort) wie zuvor aus.
4. Wählen Sie das **Ubuntu-Linux-Image** aus, klicken Sie auf **Alle Profile** und wählen Sie unter **Datenverarbeitung** (Compute) die Option **c-2x4** mit 2vCPUs und 4 GB RAM aus.
5. Wählen Sie für **SSH-Schlüssel** den SSH-Schlüssel aus, den Sie zuvor erstellt haben.
6. Klicken Sie unter **Netzschnittstellen** auf das Symbol **Bearbeiten** neben den Sicherheitsgruppen.
   * Wählen Sie **vpc-pubpriv-frontend-subnet** als Teilnetz aus.
   * Wählen Sie die Standardsicherheitsgruppe ab und aktivieren Sie **vpc-pubpriv-frontend-sg**.
   * Klicken Sie auf **Speichern**.
   * Klicken Sie auf **Virtuelle Serverinstanz erstellen**.
7. Warten Sie, bis sich der Status der VSI in **Eingeschaltet** ändert. Wählen Sie dann die Front-End-VSI **vpc-pubpriv-frontend-vsi** aus, blättern Sie zu **Netzschnittstellen** und klicken Sie auf **Reservieren** unter **Variable IP-Adresse**, um Ihrer Front-End-VSI eine öffentliche IP-Adresse zuzuordnen. Speichern Sie die zugeordnete IP-Adresse in einer Zwischenablage zur künftigen Verwendung.

## Konnektivität zwischen Front-End und Back-End einrichten
{: #setup-connectivity-frontend-backend}

Wenn alle Server eingerichtet sind, richten Sie in diesem Abschnitt die Konnektivität ein, um den regulären Betrieb zwischen den Front-End- und den Back-End-Servern zu ermöglichen.

### Front-End-Sicherheitsgruppe konfigurieren

1. Navigieren Sie zu **Sicherheitsgruppen** im Abschnitt **Teilnetz** und klicken Sie dann auf **vpc-pubpriv-frontend-sg**.
2. Fügen Sie zunächst die folgenden Regeln für **eingehenden** Datenverkehr über die Option **Regel hinzufügen** hinzu. Diese Regeln lassen eingehende HTTP-Anforderungen und Pingprüfungen (ICMP) zu.

	<table>
   <thead>
      <tr>
         <td><strong>Quelle</strong></td>
         <td><strong>Protokoll</strong></td>
         <td><strong>Wert</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Any - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>Von <strong>80</strong> bis <strong>80</strong></td>
      </tr>
      <tr>
         <td>Any - 0.0.0.0/0</td>
         <td>TCP</td>
         <td>Von <strong>443</strong> bis <strong>443</strong></td>
      </tr>
      <tr>
         <td>Any - 0.0.0.0/0</td>
	      <td>ICMP</td>
	      <td>Typ: <strong>8</strong>, Code: <strong>Leer lassen</strong></td>
      </tr>
   </tbody>
   </table>

3. Fügen Sie als Nächstes die folgenden Regeln für **ausgehenden** Datenverkehr hinzu.

   <table>
   <thead>
      <tr>
         <td><strong>Ziel</strong></td>
         <td><strong>Protokoll</strong></td>
         <td><strong>Wert</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Typ: <strong>Sicherheitsgruppe</strong> - Name: <strong>vpc-pubpriv-backend-sg</strong></td>
         <td>TCP</td>
         <td>Port des Back-End-Servers, siehe Tipp.</td>
      </tr>
   </tbody>
   </table>

Die folgenden Ports werden für typische Back-End-Services verwendet. MySQL verwendet Port 3306, PostgreSQL Port 5432. Db2 ist über Port 50000 oder 50001 zugänglich. Microsoft SQL Server verwendet standardmäßig Port 1433. Eine von vielen [Listen mit gängigen Ports finden Sie auf Wikipedia](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers).
{:tip }

### Back-End-Sicherheitsgruppe konfigurieren
Konfigurieren Sie ähnlich wie für das Front-End die Sicherheitsgruppe für das Back-End.

1. Navigieren Sie zu **Sicherheitsgruppen** im Abschnitt **Teilnetz** und klicken Sie dann auf **vpc-pubpriv-backend-sg**.
2. Fügen Sie die folgende Regel für **eingehenden** Datenverkehr über die Option **Regel hinzufügen** hinzu. Sie lässt eine Verbindung zum Back-End-Service zu.

   <table>
   <thead>
      <tr>
         <td><strong>Quelle</strong></td>
         <td><strong>Protokoll</strong></td>
         <td><strong>Wert</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Typ: <strong>Sicherheitsgruppe</strong> - Name: <strong>vpc-pubpriv-frontend-sg</strong></td>
         <td>TCP</td>
         <td>Port des Back-End-Servers</td>
      </tr>
   </tbody>
   </table>


## Software installieren und Verwaltungstasks durchführen
{: #install-software-maintenance-tasks}

Führen Sie die in [Sicherer Zugriff auf ferne Instanzen mit einem Bastion-Host](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) aufgeführten Schritte für eine geschützte Wartung der Server unter Verwendung eines Bastion-Hosts, der als `Jump-Server` fungiert, und einer Wartungssicherheitsgruppe aus.

## Ressourcen entfernen
{: #remove-resources}

1. Klicken Sie in der VPC-Managementkonsole auf **Variable IP-Adressen** und dann auf die IP-Adresse für Ihre VSIs. Wählen Sie anschließend im Aktionsmenü die Option **Freigeben** aus. Bestätigen Sie, dass die IP-Adresse freigegeben werden soll.
2. Wechseln Sie anschließend zu **Virtuelle Serverinstanzen** und **löschen** Sie Ihre Instanzen. Die Instanzen werden gelöscht und sie verbleiben eine Zeitlang im Status **Wird gelöscht**. Stellen Sie sicher, dass Sie den Browser ab und zu aktualisieren.
3. Sobald die VSIs entfernt wurden, wechseln Sie zu **Teilnetze**. Wenn das Teilnetz ein angehängtes öffentliches Gateway hat, klicken Sie auf den Teilnetznamen. Trennen Sie das öffentliche Gateway in den Teilnetzdetails. Teilnetze ohne öffentliches Gateway können über die Übersichtsseite gelöscht werden. Löschen Sie Ihre Teilnetze.
4. Wechseln Sie nach dem Löschen der Teilnetze zur Registerkarte **VPC** und löschen Sie Ihre VPC.

Wenn Sie die Konsole verwenden, müssen Sie Ihre Browseransicht möglicherweise aktualisieren, damit aktualisierte Statusinformationen nach dem Löschen einer Ressource angezeigt werden.
{:tip}

## Lernprogramm erweitern
{: #expand-tutorial}

Möchten Sie diesem Lernprogramm etwas hinzufügen oder es erweitern? Hier sind einige Ideen:

- Fügen Sie eine [Lastausgleichsfunktion](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-load-balancer) hinzu, um eingehenden Datenverkehr auf mehrere Instanzen zu verteilen.
- Erstellen Sie ein [virtuelles privates Netz](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn) (VPN), sodass Ihre VPC eine sichere Verbindung zu einem anderen privaten Netz, zum Beispiel zu einem lokalen Netz, oder einer anderen VPC herstellen kann.


## Zugehörige Inhalte
{: #related}

- [VPC-Glossar](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)
- [VPC mit Verwendung der IBM Cloud-CLI](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli#creating-a-vpc-using-the-ibm-cloud-cli)
- [VPC mit Verwendung der REST-APIs](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis#creating-a-vpc-using-the-rest-apis)
