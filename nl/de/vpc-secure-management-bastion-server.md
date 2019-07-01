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

# Sicherer Zugriff auf ferne Instanzen mit einem Bastion-Host
{: #vpc-secure-management-bastion-server}

IBM akzeptiert seit April 2019 eine begrenzte Anzahl von Kunden, die an einem Early Access-Programm für Virtual Private Cloud (VPC) teilnehmen, wobei eine erweiterte Nutzung in den folgenden Monaten eröffnet werden soll. Wenn Ihre Organisation Zugriff auf IBM Virtual Private Cloud erhalten möchte, füllen Sie bitte dieses [Nominierungsformular](https://{DomainName}/vpc){: new_window} aus. Anschließend wird ein IBM Ansprechpartner Kontakt mit Ihnen in Bezug auf die weiteren Schritte aufnehmen.
{: important}

Dieses Lernprogramm führt Sie durch die Bereitstellung eines Bastion-Hosts, um einen sicheren Zugriff auf ferne Instanzen innerhalb einer virtuellen privaten Cloud (VPC) zu ermöglichen. Der Bastion-Host ist eine Instanz, die in einem öffentlichen Teilnetz bereitgestellt wurde und auf die über SSH zugegriffen werden kann. Nach der Einrichtung fungiert der Bastion-Host als **Jump-Server**, der eine sichere Verbindung zu Instanzen ermöglicht, die in einem privaten Teilnetz bereitgestellt wurden.

Zur Verringerung des Sicherheitsrisikos für Server innerhalb der VPC erstellen und verwenden Sie einen Bastion-Host. Verwaltungstasks auf den einzelnen Servern werden mithilfe von SSH und durch den Bastion-Host als Proxy-Funktion durchgeführt. Der Zugriff auf die Server und der reguläre Internetzugriff von den Servern aus, zum Beispiel für die Softwareinstallation, werden nur mit einer besonderen Wartungssicherheitsgruppe zugelassen, die diesen Servern zugeordnet ist.
{:shortdesc}

## Lernziele
{: #objectives}

* Vorgehensweise zur Einrichtung eines Bastion-Hosts und von Sicherheitsgruppen mit Regeln kennen lernen
* Sichere Verwaltung von Servern über den Bastion-Host

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:   

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)  
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um einen Kostenvoranschlag auf der Grundlage der von Ihnen projektierten Nutzung zu generieren.

## Architektur
{: #architecture}

  ![Architektur](images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png)

1. Nach der Einrichtung der erforderlichen Infrastruktur (Teilnetze, Sicherheitsgruppen mit Regeln, VSIs) in der Cloud stellt der Administrator (DevOps) eine Verbindung (SSH) zu dem Bastion-Host unter Verwendung eines privaten SSH-Schlüssels her.
2. Der Administrator weist einer Wartungssicherheitsgruppe die geeigneten Regeln für ausgehenden Datenverkehr zu.
3. Der Administrator stellt eine sichere Verbindung (SSH) zur privaten IP-Adresse der Instanz über den Bastion-Host her, um erforderliche Software, zum Beispiel einen Web-Server, zu installieren oder zu aktualisieren.
4. Der Internetbenutzer sendet eine HTTP/HTTPS-Anforderung an den Web-Server.

## Vorbereitende Schritte
{: #prereqs}

- Überprüfen Sie die Benutzerberechtigungen. Stellen Sie sicher, dass Ihr Benutzerkonto über die entsprechenden Berechtigungen verfügt, um VPC-Ressourcen erstellen und verwalten zu können. Eine Liste der erforderlichen Berechtigungen finden Sie unter [Benötigte Berechtigungen für VPC-Benutzer erteilen](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Sie benötigen einen SSH-Schlüssel für die Herstellung einer Verbindung zu den virtuellen Servern. Wenn Sie keinen SSH-Schlüssel haben, befolgen Sie die [Anweisungen zum Erstellen eines Schlüssels](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- Das Lernprogramm geht davon, dass Sie den Bastion-Host in einer vorhandenen [virtuellen privaten Cloud](https://{DomainName}/vpc/network/vpcs) hinzufügen. **Wenn Sie keine virtuelle private Cloud in Ihrem Konto haben, erstellen Sie eine, bevor Sie mit den nächsten Schritten fortfahren.**

## Bastion-Host erstellen
{: #create-bastion-host}

In diesem Abschnitt erstellen und konfigurieren Sie einen Bastion-Host zusammen mit einer Sicherheitsgruppe in einem separaten Teilnetz.

### Teilnetz erstellen
{: #create-bastion-subnet}

1. Klicken Sie auf **Teilnetze** unter **Netz** im linken Teilfenster und anschließend auf **Neues Teilnetz**.  
   * Geben Sie **vpc-secure-bastion-subnet** als Namen ein und wählen Sie dann die VPC aus, die Sie erstellt haben.  
   * Wählen Sie eine Position (Standort) und eine Zone aus.  
   * Geben Sie den IP-Adressbereich für das Teilnetz in CIDR-Notation ein, das heißt **10.xxx.0.0/24**. Belassen Sie die Option **Adresspräfix** so, wie sie ist, und wählen Sie für **Anzahl Adressen** den Wert 256 aus.
1. Wählen Sie **VPC-Standard** für Ihre Teilnetzzugriffssteuerliste (ACL) aus. Sie können die Regeln für ein- und ausgehenden Datenverkehr später konfigurieren.
1. Schalten Sie das **öffentliche Gateway** in **Angehängt** (Attached) um. 
1. Klicken Sie auf **Teilnetz erstellen**, um es bereitzustellen.

### Bastion-Sicherheitsgruppe erstellen und konfigurieren

Erstellen Sie nun eine Sicherheitsgruppe und konfigurieren Sie Regeln für eingehenden Datenverkehr für Ihre Bastion-VSI.

1. Navigieren Sie zu **Sicherheitsgruppen** und klicken Sie auf **Neue Sicherheitsgruppe**. Geben Sie **vpc-secure-bastion-sg** als Namen ein und wählen Sie Ihre VPC aus. 
2. Erstellen Sie jetzt die folgenden Eingangsregeln, indem Sie im Abschnitt für eingehenden Datenverkehr auf **Regel hinzufügen** klicken. Diese Regeln ermöglichen den SSH-Zugriff und Pingprüfungen (ICMP).
 
	**Eingangsregel:**
	<table>
	   <thead>
	      <tr>
	         <td><strong>Quelle</strong></td>
	         <td><strong>Protokoll</strong></td>
	         <td><strong>Wert</strong></td>
	      </tr>
	   <tbody>
	      <tr>
	         <td>Any - 0.0.0.0/0</td>
	         <td>TCP</td>
	         <td>Von <strong>22</strong> bis <strong>22</strong></td>
	      </tr>
         <tr>
            <td>Any - 0.0.0.0/0</td>
	         <td>ICMP</td>
	         <td>Typ: <strong>8</strong>, Code: <strong>Leer lassen</strong></td>
         </tr>
	   </tbody>
	</table>

   Zur weiteren Erhöhung der Sicherheit kann der eingehende Datenverkehr auf das Unternehmensnetz oder ein typisches Basisnetz eingeschränkt werden. Sie können alternativ den Befehl `curl ipecho.net/plain ; echo` ausführen, um die externe IP-Adresse Ihres Netzes abzurufen und diese zu verwenden.
   {:tip }

### Bastion-Instanz erstellen
Nach der Einrichtung des Teilnetzes und der Sicherheitsgruppe können Sie als Nächstes die virtuelle Bastion-Serverinstanz erstellen.

1. Wählen Sie unter **Teilnetze** im linken Teilfenster den Eintrag **vpc-secure-bastion-subnet** aus.
2. Klicken Sie auf **Angehängte Instanzen** und stellen Sie eine **neue Instanz** mit dem Namen **vpc-secure-vsi** unter Ihrer eigenen VPC bereit. Wählen Sie Ubuntu-Linux als Ihr Image und **c-2x4** (2 vCPUs und 4 GB RAM) als Profil aus.
3. Wählen Sie eine **Position** (Standort) aus und stellen Sie sicher, dass Sie später dieselbe Position erneut verwenden.
4. Klicken Sie auf **Neuer Schlüssel**, um einen neuen **SSH-Schlüssel** zu erstellen.
   * Geben Sie **vpc-ssh-key** als Schlüsselnamen ein.
   * Belassen Sie die **Region** unverändert.
   * Kopieren Sie den Inhalt Ihres vorhandenen lokalen SSH-Schlüssels und fügen Sie ihn unter **Öffentlicher Schlüssel** ein.  
   * Klicken Sie auf **SSH-Schlüssel hinzufügen**.
5. Klicken Sie unter **Netzschnittstellen** auf das Symbol **Bearbeiten** neben den Sicherheitsgruppen. 
   * Stellen Sie sicher, dass das Teilnetz **vpc-secure-subnet** ausgewählt wird.
   * Wählen Sie die Standardsicherheitsgruppe ab und markieren Sie **vpc-secure-bastion-sg**. 
   * Klicken Sie auf **Speichern**.
6. Klicken Sie auf **Virtuelle Serverinstanz erstellen**.
7. Sobald die Instanz eingeschaltet ist, klicken Sie auf **vpc-secure-bastion-vsi** und **reservieren** Sie eine variable IP-Adresse.

### Bastion testen

Wenn die variable IP-Adresse Ihres Bastion-Hosts aktiv ist, versuchen Sie, mit **ssh** eine Verbindung zu dieser Adresse herzustellen:

   ```sh
   ssh -i ~/.ssh/<PRIVATER_SCHLÜSSEL> root@<VARIABLE_IP-ADRESSE_FÜR_BASTION>
   ```
   {:pre}

## Sicherheitsgruppe mit Wartungszugriffsregeln konfigurieren
{: #maintenance-security-group}

Wenn der Zugriff auf den Bastion-Host funktioniert, fahren Sie fort und erstellen Sie die Sicherheitsgruppe für Verwaltungstasks, wie zum Beispiel das Installieren und Aktualisieren der Software.

1. Navigieren Sie zu **Sicherheitsgruppen** und stellen Sie eine neue Sicherheitsgruppe mit dem Namen **vpc-secure-maintenance-sg** mit den nachfolgenden Regeln für ausgehenden Datenverkehr bereit:

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
         <td>Any - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>Von <strong>53</strong> bis <strong>53</strong></td>
      </tr>
      <tr>
         <td>Any - 0.0.0.0/0</td>
         <td>UDP</td>
         <td>Von <strong>53</strong> bis <strong>53</strong></td>
      </tr>
   </tbody>
   </table>

   DNS-Serveranforderungen werden an Port 53 adressiert. DNS verwendet TCP für den Zonentransfer und UDP für Namensabfragen, die entweder regulär (primär) oder umgekehrt (reverse) erfolgen. HTTP-Anforderungen werden an Port 80 und 443 gerichtet.
   {:tip }

2. Fügen Sie als Nächstes die folgende Regel für **eingehenden** Datenverkehr hinzu, die den SSH-Zugriff über den Bastion-Host ermöglicht.

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
	         <td>Typ: <strong>Sicherheitsgruppe</strong> - Name: <strong>vpc-secure-bastion-sg</strong></td>
	         <td>TCP</td>
	         <td>Von <strong>22</strong> bis <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>

3. Erstellen Sie die Sicherheitsgruppe.
4. Navigieren Sie zu **Alle Sicherheitsgruppen für VPC** und wählen Sie **vpc-secure-sg** aus.
5. Bearbeiten Sie schließlich die Sicherheitsgruppe und fügen Sie die folgende Regel für **ausgehenden** Datenverkehr hinzu.

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
	         <td>Typ: <strong>Sicherheitsgruppe</strong> - Name: <strong>vpc-secure-maintenance-sg</strong></td>
	         <td>TCP</td>
	         <td>Von <strong>22</strong> bis <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>


## Bastion-Host für den Zugriff auf andere Instanzen in der VPC verwenden
{: #bastion-host-access-instances}

In diesem Abschnitt werden Sie ein privates Teilnetz mit einer virtuellen Serverinstanz und einer Sicherheitsgruppe erstellen. Jedes Teilnetz, das in einer VPC erstellt wird, ist standardmäßig privat.

Wenn Sie bereits virtuelle Serverinstanzen in Ihrer VPC haben, zu denen Sie eine Verbindung herstellen wollen, können Sie die nächsten drei Abschnitte überspringen und damit beginnen, Ihre [virtuellen Serverinstanzen der Wartungssicherheitsgruppe hinzuzufügen](#add-vsi-to-maintenance).

### Teilnetz erstellen
{: #create-private-subnet}

Gehen Sie wie folgt vor, um ein neues Teilnetz zu erstellen:

1. Klicken Sie auf **Teilnetze** unter **Netz** im linken Teilfenster und anschließend auf **Neues Teilnetz**.  
   * Geben Sie **vpc-secure-private-subnet** als Namen ein und wählen Sie dann die VPC aus, die Sie erstellt haben.  
   * Wählen Sie eine Position (Standort) aus.  
   * Geben Sie den IP-Adressbereich für das Teilnetz in CIDR-Notation ein, das heißt **10.xxx.1.0/24**. Belassen Sie die Option **Adresspräfix** so, wie sie ist, und wählen Sie für **Anzahl Adressen** den Wert 256 aus.
1. Wählen Sie **VPC-Standard** für Ihre Teilnetzzugriffssteuerliste (ACL) aus. Sie können die Regeln für ein- und ausgehenden Datenverkehr später konfigurieren.
1. Schalten Sie das **öffentliche Gateway** in **Angehängt** (Attached) um. 
1. Klicken Sie auf **Teilnetz erstellen**, um es bereitzustellen.

### Sicherheitsgruppe erstellen

Gehen Sie wie folgt vor, um eine neue Sicherheitsgruppe zu erstellen:  
1. Klicken Sie auf **Sicherheitsgruppen** unter **Netz** und anschließend auf **Neue Sicherheitsgruppe**.  
2. Geben Sie **vpc-secure-private-sg** als Namen ein und wählen Sie die VPC aus, die Sie zuvor erstellt haben.   
3. Klicken Sie auf **Sicherheitsgruppe erstellen**.  

### Virtuelle Serverinstanz erstellen

Gehen Sie wie folgt vor, um eine virtuelle Serverinstanz in dem neu erstellten Teilnetz zu erstellen:

1. Klicken Sie auf das private Teilnetz unter **Teilnetze**.
2. Klicken Sie auf **Angehängte Instanzen** und anschließend auf **Neue Instanz**.
3. Geben Sie einen eindeutigen Namen (**vpc-secure-private-vsi**) ein und wählen Sie die VPC, die Sie zuvor erstellt haben, und dieselbe **Position** (Standort) wie zuvor aus.
4. Wählen Sie das **Ubuntu-Linux-Image** aus, klicken Sie auf **Alle Profile** und wählen Sie unter **Datenverarbeitung** (Compute) die Option **c-2x4** mit 2vCPUs und 4 GB RAM aus.
5. Wählen Sie in **SSH-Schlüssel** den SSH-Schlüssel aus, den Sie zuvor für den Bastion-Host erstellt haben.
6. Klicken Sie unter **Netzschnittstellen** auf das Symbol **Bearbeiten** neben den Sicherheitsgruppen.   
   * Wählen Sie **vpc-secure-private-subnet** als Teilnetz aus.  
   * Wählen Sie die Standardsicherheitsgruppe ab und aktivieren Sie **vpc-secure-private-sg**.  
   * Klicken Sie auf **Speichern**.  
7. Klicken Sie auf **Virtuelle Serverinstanz erstellen**.  


### Virtuelle Server der Wartungssicherheitsgruppe hinzufügen
{: #add-vsi-to-maintenance}

Für Verwaltungstätigkeiten auf den Servern müssen Sie die jeweiligen virtuellen Server der Wartungssicherheitsgruppe zuordnen. Im Folgenden richten Sie die Wartung ein, melden sich beim privaten Server an, aktualisieren die Softwarepaketinformationen und heben die Zuordnung der Sicherheitsgruppe wieder auf.

Aktivieren Sie die Wartungssicherheitsgruppe für den Server.

1. Navigieren Sie zu **Sicherheitsgruppen** und wählen Sie die Sicherheitsgruppe **vpc-secure-maintenance-sg** aus.  
2. Klicken Sie auf **Angehängte Schnittstellen** und dann auf **Schnittstellen bearbeiten**.  
3. Erweitern Sie die virtuellen Serverinstanzen und aktivieren Sie die Auswahl neben **Primär** in der Spalte **Schnittstellen**.
4. Klicken Sie auf **Speichern**, um die Änderungen anzuwenden.

### Verbindung zur Instanz herstellen

Zum Herstellen einer SSH-Verbindung zu einer Instanz über die **private IP-Adresse** verwenden Sie den Bastion-Host als **Jump-Host**.

1. Ermitteln Sie die private IP-Adresse einer virtuellen Serverinstanz unter **Virtuelle Serverinstanzen**.
2. Verwenden Sie den SSH-Befehl mit `-J`, um sich beim Server mit der **variablen IP-Adresse** des Bastion-Hosts anzumelden, die Sie zuvor verwendet haben, und mit der **privaten IP-Adresse** des Servers, die unter **Netzschnittstellen** angezeigt wird.

   ```sh
   ssh -J root@<VARIABLE_IP-ADRESSE_DES_BASTION-HOSTS> root@<PRIVATE_IP-ADRESSE>
   ```
   {:pre}
   
   Das Flag `-J` wird ab OpenSSH Version 7.3 unterstützt. In älteren Versionen ist `-J` nicht verfügbar. In diesem Fall besteht die sicherste und einfachste Methode darin, den STDIO-Weiterleitungsmodus (`-W`) des SSH-Befehls zu verwenden, um die Verbindung per Zurückweisung ("bounce") durch einen Bastion-Host weiterzuleiten. Beispiel: `ssh -o ProxyCommand="ssh -W %h:%p root@<VARIABLE_IP-ADRESSE_DES_BASTION-HOSTS" root@<PRIVATE_IP-ADRESSE>`
   {:tip }

### Software installieren und Verwaltungstasks durchführen

Wenn die Verbindung hergestellt ist, können Sie Software auf dem virtuellen Server im privaten Teilnetz installieren oder Verwaltungstasks durchführen.

1. Aktualisieren Sie zuerst die Softwarepaketinformationen:
   ```sh
   apt-get update
   ```
   {:pre}
2. Installieren Sie die gewünschte Software, zum Beispiel Nginx, MySQL oder IBM Db2.

Wenn diese Aufgaben erledigt sind, trennen Sie die Verbindung zu dem Server mit dem Befehl `exit`. 

Um HTTP/HTTPS-Anforderungen vom Internetbenutzer zuzulassen, weisen Sie der VSI eine **variable IP-Adresse** im privaten Teilnetz zu und öffnen die erforderlichen Ports (80 für HTTP und 443 für HTTPS) über die Regeln für eingehenden Datenverkehr in der Sicherheitsgruppe der privaten VSI.
{:tip}

### Wartungssicherheitsgruppe inaktivieren

Wenn Sie die Installation von Software oder Ihre Wartungsarbeiten abgeschlossen haben, sollten Sie die virtuellen Server aus der Wartungssicherheitsgruppe entfernen, um sie isoliert zu halten.

1. Navigieren Sie zu **Sicherheitsgruppen** und wählen Sie die Sicherheitsgruppe **vpc-secure-maintenance-sg** aus.  
2. Klicken Sie auf **Angehängte Schnittstellen** und dann auf **Schnittstellen bearbeiten**.  
3. Erweitern Sie die virtuellen Serverinstanzen und wählen Sie die Auswahl neben **Primär** in der Spalte **Schnittstellen** ab.
4. Klicken Sie auf **Speichern**, um die Änderungen anzuwenden.

## Ressourcen entfernen
{: #removeresources}

1. Wechseln Sie zu **Virtuelle Serverinstanzen** und **löschen** Sie Ihre Instanz. Die Instanzen werden gelöscht und sie verbleiben eine Zeitlang im Status **Wird gelöscht**. Stellen Sie sicher, dass Sie den Browser ab und zu aktualisieren.
2. Sobald die VSIs entfernt wurden, wechseln Sie zu **Teilnetze** und löschen Ihre Teilnetze.
4. Wechseln Sie nach dem Löschen der Teilnetze zur Registerkarte **Virtuelle private Clouds** und löschen Sie Ihre VPC.

Wenn Sie die Konsole verwenden, müssen Sie Ihre Browseransicht möglicherweise aktualisieren, damit aktualisierte Statusinformationen nach dem Löschen einer Ressource angezeigt werden.
{:tip}

## Zugehörige Inhalte
{: #related}

* [Private und öffentliche Teilnetze in einer virtuellen privaten Cloud (VPC)](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend)
