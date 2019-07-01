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

# Isolierte Workloads in mehreren Standorten und Zonen bereitstellen
{: #vpc-multi-region}

IBM akzeptiert seit April 2019 eine begrenzte Anzahl von Kunden, die an einem Early Access-Programm für Virtual Private Cloud (VPC) teilnehmen, wobei eine erweiterte Nutzung in den folgenden Monaten eröffnet werden soll. Wenn Ihre Organisation Zugriff auf IBM Virtual Private Cloud erhalten möchte, füllen Sie bitte dieses [Nominierungsformular](https://{DomainName}/vpc){: new_window} aus. Anschließend wird ein IBM Ansprechpartner Kontakt mit Ihnen in Bezug auf die weiteren Schritte aufnehmen.
{: important}

Dieses Lernprogramm führt Sie durch die Schritte zur Einrichtung isolierter Workloads, indem Sie virtuelle private Clouds (VPCs) in verschiedenen IBM Cloud-Regionen bereitstellen. Dies sind Regionen mit Teilnetzen und virtuellen Serverinstanzen (VSIs). Diese VSIs werden in mehreren Zonen in einer Region erstellt, um die Ausfallsicherheit in einer Region zu erhöhen, und global, indem Lastausgleichsfunktionen mit Back-End-Pools, Front-End-Listenern und geeigneten Statusprüfungen konfiguriert werden.

Für die globale Lastausgleichsfunktion werden Sie einen IBM CIS-Service (Cloud Internet Services) aus dem Katalog bereitstellen. Zur Verwaltung des SSL-Zertifikats für alle eingehenden HTTPS-Anforderungen wird ein {{site.data.keyword.cloudcerts_long_notm}}-Katalogservice erstellt und das Zertifikat wird zusammen mit dem privaten Schlüssel importiert.

{:shortdesc}

## Lernziele
{: #objectives}

* Kennenlernen der Isolation von Workloads durch Infrastrukturobjekte, die für virtuelle private Clouds verfügbar sind
* Verwenden der Lastausgleichsfunktion zwischen Zonen in einer Region, um Datenverkehr auf virtuelle Server zu verteilen
* Verwenden einer globalen Lastausgleichsfunktion zwischen Regionen, um die Ausfallsicherheit zu erhöhen und die Latenz zu verringern

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet: 

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.loadbalancer_full}}](https://{DomainName}/vpc/provision/loadBalancer)
- IBM Cloud [Internet-Services](https://{DomainName}/catalog/services/internet-services)
- [{{site.data.keyword.cloudcerts_long_notm}}](https://{DomainName}/catalog/services/cloudcerts)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/estimator/review), um einen Kostenvoranschlag auf der Grundlage der von Ihnen projektierten Nutzung zu generieren.

## Architektur
{: #architecture}

  ![Architektur](images/solution41-vpc-multi-region/Architecture.png)

1. Der Administrator (DevOps) stellt VSIs in Teilnetzen unter zwei verschiedenen Zonen in einer VPC in Region 1 bereit und wiederholt die gleiche Prozedur in einer VPC, die in Region 2 erstellt wurde.
2. Der Administrator erstellt eine Lastausgleichsfunktion mit einem Back-End-Pool von Servern von Teilnetzen in verschiedenen Zonen der Region 1 sowie einen Front-End-Listener. Er wiederholt die gleiche Prozedur in Region 2.
3. Der Administrator stellt einen Service für Internet-Services mit einer zugeordneten angepassten Domäne bereit und erstellt eine globale Lastausgleichsfunktion, die auf die Lastausgleichsfunktionen verweist, die in zwei verschiedenen VPCs erstellt wurden.
4. Der Administrator aktiviert die HTTPS-Verschlüsselung, indem er dem Zertifikatmanager-Service das SSL-Zertifikat der Domäne hinzufügt.
5. Der Internetbenutzer sendet eine HTTP/HTTPS-Anforderung und die globale Lastausgleichsfunktion verarbeitet die Anforderung.
6. Die Anforderung wird an die Lastausgleichsfunktionen (die globale und die lokalen) weitergeleitet. Die Anforderung wird anschließend durch die verfügbare Serverinstanz erfüllt.

## Vorbereitende Schritte
{: #prereqs}

- Überprüfen Sie die Benutzerberechtigungen. Stellen Sie sicher, dass Ihr Benutzerkonto über die entsprechenden Berechtigungen verfügt, um VPC-Ressourcen erstellen und verwalten zu können. Eine Liste der erforderlichen Berechtigungen finden Sie unter [Benötigte Berechtigungen für VPC-Benutzer erteilen](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Sie benötigen einen SSH-Schlüssel für die Herstellung einer Verbindung zu den virtuellen Servern. Wenn Sie keinen SSH-Schlüssel haben, befolgen Sie die [Anweisungen zum Erstellen eines Schlüssels](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- Für Cloud Internet Services müssen Sie über eine angepasste Domäne verfügen, sodass Sie das DNS für diese Domäne so konfigurieren können, dass es auf die Cloud Internet Services-Namensserver verweist. Wenn Sie keine Domäne haben, können Sie eine von einem Registrator wie [godaddy.com](http://godaddy.com/) kaufen.

## VPCs, Teilnetze und VSIs erstellen
{: #create-infrastructure}

In diesem Abschnitt werden Sie eine eigene VPC in Region 1 mit Teilnetzen erstellen, die in zwei verschiedenen Zonen von Region 1 erstellt werden, und anschließend VSIs bereitstellen.

Gehen Sie wie folgt vor, um eine eigene {{site.data.keyword.vpc_short}} in Region 1 zu erstellen:

1. Navigieren Sie zur Seite [VPC-Übersicht](https://{DomainName}/vpc/overview) und klicken Sie auf **VPC erstellen**.
2. Unter dem Abschnitt **Neue virtuelle private Cloud**:
   * Geben Sie **vpc-region1** als Namen für Ihre VPC ein.
   * Wählen Sie eine **Ressourcengruppe** aus.
   * Fügen Sie optional **Tags** hinzu, um Ihre Ressourcen zu organisieren.
3. Wählen Sie **Neuen Standard erstellen (Alle zulassen)** als Ihre VPC-Standardzugriffssteuerungsliste (ACL) aus.
4. Wählen Sie SSH ab und überprüfen Sie die **Standardsicherheitsgruppe** mit Ping und belassen Sie die Option **Klassischer Zugriff** abgewählt.
5. Unter **Neues Teilnetz für VPC**:
   * Geben Sie **vpc-region1-zone1-subnet** als eindeutigen Namen ein.
   * Wählen Sie eine Position (z. B. Dallas) aus. Benennen Sie diese Position zum Beispiel mit **region 1** und eine Zone in Region 1 (z. B. Dallas 1) mit **zone 1**.
   * Geben Sie den IP-Adressbereich für das Teilnetz in CIDR-Notation ein, das heißt **10.xxx.0.0/24**. Belassen Sie die Option **Adresspräfix** so, wie sie ist, und wählen Sie für **Anzahl Adressen** den Wert 256 aus.
6. Wählen Sie die Option **VPC-Standdard verwenden** für Ihre Teilnetzzugriffssteuerliste (ACL) aus. Sie können die Regeln für ein- und ausgehenden Datenverkehr später konfigurieren.
7. Da allen virtuellen Serverinstanzen im Teilnetz eine variable IP-Adresse zugeordnet werden soll, ist es nicht erforderlich, ein öffentliches Gateway für das Teilnetz zu aktivieren. Die virtuellen Serverinstanzen erhalten ihre Internetkonnektivität über ihre variable IP-Adresse.
8. Klicken Sie auf **Virtuelle private Cloud erstellen**, um die Instanz bereitzustellen.

Zur Überprüfung der Erstellung des Teilnetzes klicken Sie auf **Teilnetze** im linken Teilfenster und warten Sie, bis sich der Status in **Verfügbar** ändert. Sie können ein neues Teilnetz unter **Teilnetze** erstellen.

### Teilnetz in Zone 2 erstellen

1. Klicken Sie auf **Neues Teilnetz**, geben Sie **vpc-region1-zone2-subnet** als eindeutigen Namen für Ihr Teilnetz ein und wählen Sie **vpc-region1** als VPC aus.
2. Wählen Sie die Position aus, die oben mit 'region 1' (z. B. Dallas) bezeichnet wurde, und wählen Sie eine andere Zone in Region 1 (z. B. Dallas 2) aus. Nennen Sie die ausgewählte Zone z. B. **zone 2**.
3. Geben Sie den IP-Adressbereich für das Teilnetz in CIDR-Notation ein, das heißt **10.xxx.64.0/24**. Belassen Sie die Option **Adresspräfix** so, wie sie ist, und wählen Sie für **Anzahl Adressen** den Wert 256 aus.
4. Wählen Sie die Option **VPC-Standdard verwenden** für Ihre Teilnetzzugriffssteuerliste (ACL) aus. 

### VSIs bereitstellen
Sobald sich der Status der Teilnetze in **Verfügbar** ändert, gehen Sie wie folgt vor:

1. Klicken Sie auf **vpc-region1-zone1-subnet** und klicken Sie auf **Angehängte Instanzen** und anschließend auf **Neue Instanz**.
2. Geben Sie einen eindeutigen Namen ein und wählen Sie **vpc-region1-zone1-vsi** aus. Wählen Sie dann die VPC, die Sie zuvor erstellt haben, und die **Position** zusammen mit der **Zone** wie zuvor aus.
3. Wählen Sie ein **Ubuntu-Linux-Image** aus, klicken Sie auf **Alle Profile** und wählen Sie unter **Datenverarbeitung** (Compute) die Option **c-2x4** mit 2vCPUs und 4 GB RAM aus.
4. Wählen Sie für **SSH-Schlüssel** den SSH-Schlüssel aus, den Sie zu Anfang erstellt haben.
5. Klicken Sie unter **Netzschnittstellen** auf das Symbol **Bearbeiten** neben den Sicherheitsgruppen.
   * Prüfen Sie, ob **vpc-region1-zone1-subnet** als Teilnetz ausgewählt ist. Ist dies nicht der Fall, wählen Sie dieses Teilnetz aus.
   * Klicken Sie auf **Speichern**.
   * Klicken Sie auf **Virtuelle Serverinstanz erstellen**.
6.  Warten Sie, bis sich der Status der VSI in **Eingeschaltet** ändert. Wählen Sie dann die VSI **vpc-region1-zone1-vsi** aus, blättern Sie zu **Netzschnittstellen** und klicken Sie auf **Reservieren** unter **Variable IP-Adresse**, um Ihrer VSI eine öffentliche IP-Adresse zuzuordnen. Speichern Sie die zugeordnete IP-Adresse in einer Zwischenablage zur künftigen Verwendung.
7. **Wiederholen** Sie die Schritte 1-6, um eine VSI in **zone 2** von **region 1** bereitzustellen.

Navigieren Sie zu **VPC** und **Teilnetze** unter **Netz** im linken Teilfenster und **wiederholen** Sie die obigen Schritte zur Bereitstellung einer neuen VPC mit Teilnetzen und VSIs in **region2**, indem Sie dieselben Konventionen wie oben befolgen.

## Web-Server auf den VSIs installieren und konfigurieren
{: #install-configure-web-server-vsis}

Führen Sie die in [Sicherer Zugriff auf ferne Instanzen mit einem Bastion-Host](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) aufgeführten Schritte für eine geschützte Wartung der Server unter Verwendung eines Bastion-Hosts, der als `Jump-Server` fungiert, und einer Wartungssicherheitsgruppe aus.
{:tip}

Gehen Sie wie folgt vor, wenn Sie erfolgreich eine SSH-Verbindung in den Server hergestellt haben, der im Teilnetz von Zone 1 von Region 1 bereitgestellt wurde:

1. Führen Sie über die Eingabeaufforderung die nachfolgenden Befehle aus, um Nginx als Ihren Web-Server zu installieren:
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   {:codeblock}
2. Prüfen Sie den Status des Nginx-Service mit dem folgenden Befehl:
   ```
   sudo systemctl status nginx
   ```
   {:pre}
   Die Ausgabe sollte zeigen, dass der Nginx-Service aktiv (**active**) ist und ausgeführt wird.
3. Sie müssen die Ports **HTTP (80)** und **HTTPS (443)** öffnen, um Datenverkehr (Anforderungen) zu empfangen. Sie können dies tun, indem Sie die Firewall über [UFW](https://help.ubuntu.com/community/UFW) - `sudo ufw enable` anpassen und das Profil ‘Nginx Full’ aktivieren, das Rollen für beide Ports enthält:
   ```
   sudo ufw allow 'Nginx Full'
   ```
   {:pre}
4. Zur Überprüfung, ob Nginx wie erwartet funktioniert, öffnen Sie `http://VARIABLE_IP-ADRESSE` in Ihrem bevorzugten Browser. Es sollte die Standardeinführungsseite von Nginx angezeigt werden.
5. Zum Aktualisieren der HTML-Seite mit den Regions- und Zonendetails führen Sie den nachfolgenden Befehl aus:
   ```
   nano /var/www/html/index.nginx-debian.html
   ```
   {:pre}
   Hängen Sie die Region und die Zone wie zum Beispiel durch die Angabe _server running in **zone 1 of region 1**_ an den Tag `h1` mit der Zeichenfolge `Welcome to nginx!` an und speichern Sie die Änderungen.
6. Starten Sie den Nginx-Server neu, damit die Änderungen wirksam werden.
   ```
   sudo systemctl restart nginx
   ```
   {:pre}

**Wiederholen** Sie die Schritte 1-6, um den Web-Server auf den VSIs in Teilnetzen aller Zonen zu installieren und zu konfigurieren, und denken Sie daran, das entsprechende HTML mit den jeweiligen Zoneninformationen zu aktualisieren.


## Datenverkehr mit Lastausgleichsfunktionen auf Zonen verteilen
{: #distribute-traffic-with-load-balancers}

In diesem Abschnitt erstellen Sie zwei Lastausgleichsfunktionen. Eine in der Region, um Datenverkehr auf mehrere Serverinstanzen unter den entsprechenden Teilnetzen in verschiedenen Zonen zu verteilen.

### Lastausgleichsfunktionen konfigurieren

1. Navigieren Sie zu **Lastausgleichsfunktionen** (Load Balancer) und klicken Sie auf **Neue Lastausgleichsfunktion**.
2. Geben Sie **vpc-lb-region1** als eindeutigen Namen an, wählen Sie **vpc-region1** als ihre virtuelle private Cloud aus. Wählen Sie anschließend die Ressourcengruppe, in der die VPC erstellt wurde, mit dem Typ **Public** und **region1** als Region aus.
3. Wählen Sie die privaten IP-Adressen von **zone 1** und **zone 2** von **region 1** aus.
4. Erstellen Sie einen neuen Back-End-Pool von VSIs, die als gleiche Peers fungieren, um den an den Pool geleiteten Datenverkehr gemeinsam zu verarbeiten. Legen Sie die Parameter mit den folgenden Werten fest und klicken Sie auf **Erstellen**.
	- **Name:**  region1-pool
	- **Protokoll (Protocol):** HTTP
	- **Methode:** Round robin
	- **Sitzungstreue (Session stickiness):** None
	- **Statusprüfungspfad (Health check path):** /
	- **Statusprotokoll (Health protocol):** HTTP
	- **Intervall(Sek):** 15
	- **Zeitlimit (Sek):** 2
	- **Maximale Anzahl Wiederholungen (Max retries):** 2
5. Klicken Sie auf **Anhängen** (Attach), um dem Pool von Region 1 Serverinstanzen hinzuzufügen.
   - Wählen Sie die private IP-Adresse von **vpc-region1-zone1-subnet** und die Instanz, die Sie erstellt haben aus und legen Sie den Port 80 fest.
   - Klicken Sie auf **Hinzufügen** und wählen Sie nun die private IP-Adresse von **vpc-region1-zone2-subnet** und die Instanz aus und legen Sie den Port 80 fest.
   - Klicken Sie auf **Anhängen** (Attach), um die Erstellung des Back-End-Pools abzuschließen.
6. Klicken Sie auf **Neuer Listener**, um einen neuen Front-End-Listener zu erstellen. Ein Listener ist ein Prozess, der auf Verbindungsanforderungen prüft.
   - **Protokoll (Protocol):** HTTP
   - **Port:** 80
   - **Back-End-Pool:** region1-pool
   - **Maximale Anzahl Verbindungen (Maxconnections):** Lassen Sie dieses Feld leer und klicken Sie auf **Erstellen**.
7. Klicken Sie auf **Lastausgleichsfunktion erstellen**, um eine Lastausgleichsfunktion bereitzustellen.

### Lastausgleichsfunktionen testen

1. Warten Sie, bis sich der Status der Lastausgleichsfunktionen in **Aktiv** ändert.
2. Öffnen Sie die **Adresse** in einem Web-Browser.
3. Aktualisieren Sie die Seite mehrmals und beachten Sie, dass die Lastausgleichsfunktion bei jeder Aktualisierung andere Server ansteuert.
4. **Speichern** Sie die Adresse als zukünftige Referenz.

Bei genauerer Beobachtung erkennen Sie, dass die Anforderungen nicht verschlüsselt werden und nur HTTP unterstützen. Sie werden im nächsten Abschnitt ein SSL-Zertifikat konfigurieren und HTTPS aktivieren.

**Wiederholen** Sie die obigen Schritte 1-7 in **region 2**.

## Sicherer Datenverkehr in der VPC mit HTTPS
{: #secure_https}

Vor dem Hinzufügen eines HTTPS-Listeners müssen Sie ein SSL-Zertifikat generieren, die Authentizität Ihrer angepassten Domäne überprüfen, eine Position zum Speichern des Zertifikats festlegen und das Zertifikat dem Infrastrukturservice zuordnen.

### CIS-Service bereitstellen und angepasste Domäne konfigurieren

In diesem Abschnitt werden Sie den CIS-Service (CIS - IBM Cloud Internet Services) erstellen, eine angepasste Domäne konfigurieren, indem Sie einen Verweis auf CIS-Namensserver angeben, und später eine globale Lastausgleichsfunktion konfigurieren.

1. Navigieren Sie zu [Internet Services](https://{DomainName}/catalog/services/internet-services) im {{site.data.keyword.Bluemix_notm}}-Katalog.
2. Legen Sie den Servicenamen fest und klicken Sie auf **Erstellen**, um eine Instanz des Service zu erstellen. Sie können beliebige Preistarife für dieses Lernprogramm verwenden.
3. Wenn die Serviceinstanz bereitgestellt ist, legen Sie Ihren Domänennamen fest, indem Sie auf **Los geht's** und dann auf **Domäne hinzufügen** klicken.
4. Klicken Sie auf **Nächster Schritt**. Wenn die Namensserver zugewiesen sind, konfigurieren Sie Ihren Registrator oder den Domänennamenprovider so, dass die aufgeführten Namensserver verwendet werden.
5. Nach der Konfiguration des Registrators bzw. des DNS-Providers kann es bis zu 24 Stunden dauern, bis die Änderungen wirksam werden.

   Wenn der Status der Domäne auf der Übersichtsseite von *Anstehend* in *Aktiv* wechselt, können Sie den Befehl `dig <IHR_DOMÄNENNAME> ns` verwenden, um zu prüfen, ob die neuen Namensserver in Betrieb gesetzt wurden.
   {:tip}

Sie sollten ein SSL-Zertifikat für die Domäne und die Unterdomäne abrufen, die Sie mit der globalen Lastausgleichsfunktion verwenden wollen. Wenn zum Beispiel eine Domäne wie 'mydomain.com' verwendet wird, könnte die globale Lastausgleichsfunktion unter `lb.mydomain.com` gehostet werden. Das Zertifikat muss in diesem Fall für 'lb.mydomain.com' ausgestellt werden.

Sie können kostenlose SSL-Zertifikate von [Let's Encrypt](https://letsencrypt.org/) abrufen. Während des Prozesses müssen Sie möglicherweise einen DNS-Datensatz vom Typ TXT in der DNS-Schnittstelle von Cloud Internet Services konfigurieren, um nachzuweisen, dass Sie der Eigner der Domäne sind.
{:tip}

Sobald Sie das SSL-Zertifikat und den privaten Schlüssel für Ihre Domäne abgerufen haben, stellen Sie sicher, dass Sie diese in das Format [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) konvertieren.

1. Gehen Sie wie folgt vor, um ein Zertifikat in das PEM-Format zu konvertieren:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
2. Gehen Sie wie folgt vor, um einen privaten Schlüssel in das PEM-Format zu konvertieren:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Zertifikat importieren und Lastausgleichsservice autorisieren

Sie können die SSL-Zertifikate mithilfe von IBM Certificate Manager verwalten.

1. Erstellen Sie eine [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)-Instanz an einer unterstützten Position (Standort).
2. Verwenden Sie im Service-Dashboard die Option **Zertifikat importieren**:
   * Geben Sie in **Name** die angepasste Unterdomäne und Domäne an. Beispiel: *lb.mydomain.com*.
   * Suchen Sie nach der **Zertifikatsdatei** im PEM-Format.
   * Suchen Sie nach der **Datei mit privatem Schlüssel** im PEM-Format.
   * **Importieren** Sie die Dateien.
3. Erstellen Sie eine Berechtigung, die der Lastausgleichsserviceinstanz Zugriff auf die Zertifikatmanagerinstanz erteilt, die das SSL-Zertifikat enthält. Sie können eine solche Berechtigung über [Identity and Access Authorizations](https://{DomainName}/iam#/authorizations) verwalten.
  - Klicken Sie auf **Erstellen** und wählen Sie **Infrastrukturservice** als Quellenservice aus.
  - Wählen Sie **Lastausgleichsfunktion für VPC** als Ressourcentyp aus.
  - Wählen Sie **Zertifikatmanager** als Zielservice aus.
  - Weisen Sie die Servicezugriffsrolle **Schreibberechtigter** (Writer) zu.
  - Zum Erstellen einer Lastausgleichsfunktion müssen Sie die Berechtigung 'Alle Ressourceninstanzen' für die Quellenressourceninstanz erteilen. Die Zielserviceinstanz kann **Alle Instanzen** sein oder sie kann Ihre spezielle Ressourceninstanz des Zertifikatmanagers sein.

### HTTPS-Listener erstellen

Navigieren Sie nun zu den [Lastausgleichsfunktionen](https://{DomainName}/vpc/network/loadBalancers).

1. Wählen Sie **vpc-lb-region1** aus.
2. Klicken Sie unter **Front-End-Listener** auf **Neuer Listener**.

   -  **Protokoll:** HTTPS
   -  **Port:** 443
   -  **Back-End-Pool:** POOL in derselben Region
   -  Wählen Sie das SSL-Zertifikat für **lb.IHR_DOMÄNENNAME** aus.

3. Klicken Sie auf **Erstellen**, um einen HTTPS-Listener zu konfigurieren.

**Wiederholen** Sie die gleichen Schritte für die Lastausgleichsfunktion von **region 2**.

## Globale Lastausgleichsfunktion konfigurieren
{: #global-load-balancer}

In diesem Abschnitt konfigurieren Sie eine globale Lastausgleichsfunktion (GLB - Global Load Balancer), die den eingehenden Datenverkehr auf die lokalen Lastausgleichsfunktionen verteilt, die in verschiedenen {{site.data.keyword.Bluemix_notm}}-Regionen konfiguriert sind.

### Datenverkehr mit einer globalen Lastausgleichsfunktion auf Regionen verteilen
Öffnen Sie den CIS-Service, den Sie erstellt haben, indem Sie zur [Ressourcenliste](https://{DomainName}/resources) unter 'Services' navigieren.

1. Navigieren Sie zu **Globale Lastausgleichsfunktionen** unter **Zuverlässigkeit** und klicken Sie auf **Lastausgleichsfunktion erstellen**.
2. Geben Sie **lb.IHR_DOMÄNENNAME** als Ihren Hostnamen und für TTL den Wert 60 Sekunden ein.
3. Klicken Sie auf **Pool hinzufügen**, um einen Standardursprungspool zu definieren.
   - **Name:** lb-region1
   - **Statusprüfungspfad (Health check):** NEUE STATUSPRÜFUNG ERSTELLEN
     - **Überwachungstyp (Monitor Type):** HTTP
     - **Pfad (Path):** /
     - **Port:** 80
   - **Region der Statusprüfung (Health check region):** Östliches Nordamerika
   - **Ursprünge (origins)**
     - **Name (name):** region1
     - **Adresse (address):** ADRESSE DER LOKALEN LASTAUSGLEICHSFUNKTION VON **REGION1**
     - **Gewichtung (weight):** 1
     - Klicken Sie auf **Hinzufügen**.

4. **Fügen** Sie einen weiteren **Ursprungspool** hinzu, der auf die Lastausgleichsfunktion von **region2** in der Region **Westeuropa** verweist, und klicken Sie auf **1 Ressource bereitstellen**, um Ihre globale Lastausgleichsfunktion bereitzustellen.

Warten Sie ab, bis sich der Status der **Statusprüfung** in **Einwandfrei** ändert. Öffnen Sie die Verbindung **lb.IHR_DOMÄNENNAME** in einem Browser Ihrer Wahl, um die globale Lastausgleichsfunktion in Aktion zu sehen.

### Failovertest
Sie sollten inzwischen gesehen haben, dass Sie meistens auf den Servern in **region 1** landen, da diesen im Vergleich zu den Servern in **region 2** eine höhere Gewichtung zugeordnet ist. Nun führen Sie eine fehlschlagende Statusprüfung im Ursprungspool in **region 1** ein:

1. Navigieren Sie zu [Virtuelle Serverinstanzen](https://{DomainName}/vpc/compute/vs).
2. Klicken Sie auf die **drei Punkte (...)** neben dem Server oder neben den Servern, die in **zone 1** von **region 1** ausgeführt werden, und klicken Sie auf **Stoppen**.
3. **Wiederholen** Sie den gleichen Vorgang für Server, die in **zone 2** von **region 1** ausgeführt werden.
4. Kehren Sie zu 'Globale Lastausgleichsfunktion' (GLB) unter 'CIS-Service' zurück und warten Sie, bis sich der Status der Statusprüfung in **Kritisch** ändert.
5. Wenn Sie jetzt Ihre Domänen-URL aktualisieren, sollten Sie immer auf den Servern in **region 2** landen.

Vergessen Sie nicht, die Server in Zone 1 und Zone 2 von Region 1 zu **starten**.
{:tip}

## Ressourcen entfernen
{: #removeresources}

- Entfernen Sie die globale Lastausgleichsfunktion, die Ursprungspools und die Statusprüfungen unter dem CIS-Service.
- Entfernen Sie die Zertifikate im Zertifikatmanager-Service.
- Entfernen Sie die Lastausgleichsfunktionen, VSIs, Teilnetze und VPCs.
- Löschen Sie in der [Ressourcenliste](https://{DomainName}/resources) die Services, die in diesem Lernprogramm verwendet wurden.


## Zugehörige Inhalte
{: #related}

* [Lastausgleichsfunktionen in einer IBM Cloud-VPC verwenden](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc#--beta-using-load-balancers-in-ibm-cloud-vpc)
