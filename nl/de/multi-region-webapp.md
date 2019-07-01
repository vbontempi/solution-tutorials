---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Sichere Web-Anwendung in mehreren Regionen
{: #multi-region-webapp}

Dieses Lernprogramm führt Sie durch die Erstellung, Sicherung, Bereitstellung und Lastverteilung einer Cloud Foundry-Anwendung in mehreren Regionen mithilfe einer [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)-Pipeline.

In Apps oder in Teilen von Apps werden Ausfälle auftreten - dies lässt sich nie völlig vermeiden. Die Ursachen für solche Ausfälle können vielfältig sein: ein Fehler in Ihrem App-Code, eine geplante Wartungsmaßnahme für die von der App genutzten Ressourcen, der Ausfall einer Zone, eines Standorts oder eines Rechenzentrums, in dem Ihre App gehostet wird, aufgrund eines Hardwarefehlers. Auf solche Ausfälle müssen Sie vorbereitet sein. Mit {{site.data.keyword.Bluemix_notm}} können Sie Ihre Anwendung an [mehreren Standorten](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg) bereitstellen, um die Ausfallsicherheit der Anwendung zu optimieren. Darüber hinaus eröffnet die Ausführung Ihrer Anwendung an mehreren Standorten die Möglichkeit, Benutzerdatenverkehr zum nächstgelegenen Standort umzuleiten, um die Latenzzeit zu reduzieren.

## Ziele

* Stellen Sie eine Cloud Foundry-Anwendung mit {{site.data.keyword.contdelivery_short}} an mehreren Standorten bereit.
* Ordnen Sie der Anwendung eine angepasste Domäne zu.
* Konfigurieren Sie den globalen Lastausgleich für Ihre Anwendung mit mehreren Standorten.
* Binden Sie ein SSL-Zertifikat an Ihre Anwendung.
* Überwachen Sie die Leistung der Anwendung.

## Verwendete Services

Im vorliegenden Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs) Cloud Foundry-App
* [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) for DevOps
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur

Dieses Lernprogramm beinhaltet ein Aktiv/Aktiv-Szenario, in dem zwei Kopien der Anwendung an zwei verschiedenen Standorten implementiert werden und Kundenanforderungen im Umlaufverfahren abwickeln. Wenn eine der beiden Kopien ausfällt, verweist die DNS-Konfiguration automatisch auf den betriebsbereiten Standort.

<p style="text-align: center;">

   ![Architektur](./images/solution1/Architecture.png)
</p>

## Node.js-Anwendung erstellen
{: #create}

Erstellen Sie zunächst eine Node.js-Starteranwendung, die in einer Cloud Foundry-Umgebung ausgeführt wird.

1. Klicken Sie auf **[Katalog](https://{DomainName}/catalog/)** in der {{site.data.keyword.Bluemix_notm}}-Konsole.
2. Klicken Sie auf **Cloud Foundry-Apps** in der Kategorie **Plattform** und wählen Sie **[{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)**
     ![](images/solution1/SDKforNodejs.png) aus.
3. Geben Sie einen **eindeutigen Namen** für Ihre Anwendung ein, der auch als Hostname verwendet wird (z. B. myusername-nodeapp). Klicken Sie anschließend auf **Erstellen**.
4.  Klicken Sie nach dem Starten der Anwendung auf den Link **URL besuchen** auf der Seite **Übersicht**, damit Ihre Anwendung live in einer neuen Registerkarte angezeigt wird.

![HelloWorld](images/solution1/HelloWorld.png)

Der Anfang ist gemacht. Ihre eigene Node.js-Starteranwendung wird jetzt in {{site.data.keyword.Bluemix_notm}} ausgeführt.

Übertragen Sie nun mit einer Push-Operation den Quellcode Ihrer Anwendung in ein Repository und stellen Sie Ihre Änderungen automatisch bereit.

## Quellcodeverwaltung und {{site.data.keyword.contdelivery_short}} einrichten
{: #devops}

In diesem Schritt richten Sie ein Git-Repository für die Quellcodeverwaltung ein, in dem Ihr Code gespeichert wird, und erstellen anschließend eine Pipeline, um Codeänderungen automatisch bereitzustellen.

1. Wählen Sie im linken Anzeigebereich der Anwendung, die Sie soeben erstellt haben, die Option **Übersicht** aus und blättern Sie zu **{{site.data.keyword.contdelivery_short}}**. Klicken Sie auf **Aktivieren**.

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. Behalten Sie die Standardoptionen bei und klicken Sie auf **Erstellen**. Sie verfügen nun über eine erstellte Standard-**Toolchain**.

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. Wählen Sie die Kachel **Git** unter **Code** aus. Sie werden zu der Seite für Ihr Git-Repository weitergeleitet.
4. Wenn Sie noch keine SSH-Schlüssel eingerichtet haben, finden Sie entsprechende Anweisungen in einer Benachrichtigungsleiste oben auf der Seite. Führen Sie die angegebenen Schritte aus, indem Sie den Link **SSH-Schlüssel hinzufügen** in einer neuen Registerkarte öffnen. Wenn Sie HTTPS anstelle von SSH verwenden möchten, führen Sie die angegebenen Schritte für die Option **Persönliches Zugriffstoken erstellen** aus. Denken Sie daran, den Schlüssel oder das Token für die weitere Verwendung zu speichern.
5. Wählen Sie 'SSH' oder 'HTTPS' aus und kopieren Sie die Git-URL. Klonen Sie die Quelle auf Ihre lokale Maschine.
   ```bash
   git clone <ihre_repository-url>
   cd <name_ihrer_app>
   ```
   **Hinweis:** Wenn Sie zur Eingabe eines Benutzernamens aufgefordert werden, geben Sie Ihren Git-Benutzernamen an. Verwenden Sie für das Kennwort einen vorhandenen **SSH-Schlüssel** bzw. ein **persönliches Zugriffstoken** oder das im vorherigen Schritt erstellte Kennwort.
6. Öffnen Sie das geklonte Repository in einer IDE Ihrer Wahl und navigieren Sie zu `public/index.html`. Sie können nun den Code aktualisieren. Geben Sie anstelle von "Hello World" eine andere Zeichenfolge an. 
7. Führen Sie die Anwendung lokal aus, indem Sie nacheinander die Befehle
  `npm install`, `npm build`,  `npm start` ausführen und rufen Sie in Ihrem Browser `localhost:<port_number>`
  **<portnummer>** auf, wie in der Konsole angezeigt.
8. Übertragen Sie die Änderung mit drei einfachen Schritten in Ihr Repository: Hinzufügen (add), Festschreiben (commit) und Push-Operation (push).
   ```bash
   git add public/index.html
   git commit -m "meine ersten Änderungen"
   git push origin master
   ```
9. Rufen Sie die Toolchain auf, die Sie zuvor erstellt haben, und klicken Sie auf die Kachel **Delivery Pipeline**.
10. Überprüfen Sie, dass die Stages **BUILD** und **DEPLOY** angezeigt werden.
  ![HelloWorld](images/solution1/DevOps_Pipeline.png)
11. Warten Sie, bis die Stage **DEPLOY** abgeschlossen ist.
12. Klicken Sie auf die **url** der Anwendung im Ergebnis der letzten Ausführung, damit Ihre Änderungen live angezeigt werden.

Nehmen Sie weitere Änderungen an Ihrer Anwendung vor und schreiben Sie die Änderungen regelmäßig in Ihrem Git-Repository fest. Wenn Ihre Anwendung nicht aktualisiert wird, überprüfen Sie die Protokolle für die Stages DEPLOY und BUILD in Ihrer Pipeline.

## An anderem Standort bereitstellen
{: #deploy_another_region}

Als nächstes wird dieselbe Anwendung an einem anderen {{site.data.keyword.Bluemix_notm}}-Standort bereitgestellt. Dabei kann dieselbe Toolchain verwendet werden; es wird jedoch eine andere Stage DEPLOY hinzugefügt, um die Bereitstellung der Anwendung an einem anderen Standort zu verarbeiten.

1. Navigieren Sie zur **Übersicht** der Anwendung und blättern Sie zu **Toolchain anzeigen**.
2. Wählen Sie **Delivery Pipeline** unter 'Bereitstellen' aus.
3. Klicken Sie auf das **Zahnradsymbol** in der Stage **DEPLOY** und
wählen Sie **Stage klonen** aus.
   ![HelloWorld](images/solution1/CloneStage.png)
4. Benennen Sie die Stage in "In Großbritannien bereitstellen" um und wählen Sie **JOBS** aus.
5. Ändern Sie die **IBM Cloud-Region** in **London - https://api.eu-gb.bluemix.net**. Erstellen Sie einen **Bereich**, falls noch kein Bereich vorhanden ist.
6. Ändern Sie **Bereitstellungsscript** in `cf push "${CF_APP}" -d eu-gb.mybluemix.net`.

   ![HelloWorld](images/solution1/DeployToUK.png)
7. Klicken Sie auf **Speichern** und führen Sie die neue Stage aus, indem Sie auf die **Schaltfläche 'Wiedergabe'** klicken.

## Registrieren Sie eine angepasste Domäne bei IBM Cloud Internet Services

{: #domain_cis}

IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) ist eine einheitliche Plattform zum Konfigurieren und Verwalten von Domain Name System (DNS), Global Load Balancing (GLB), Web Application Firewall (WAF) und des DDoS-Schutzes (Distributed Denial of Service) für Webanwendungen. Es bietet einen schnellen, sehr leistungsfähigen, zuverlässigen und sicheren Internet-Service für Kunden, deren Unternehmen unter IBM Cloud ausgeführt wird, mit drei zentralen Funktionen für die Verbesserung des Workflows: Sicherheit, Zuverlässigkeit und Leistung.  

Beim Bereitstellen einer realen Anwendung werden Sie vermutlich die Verwendung einer eigenen Domäne anstelle der von IBM bereitgestellten Domäne mybluemix.net bevorzugen. Im vorliegenden Schritt können Sie nach dem Erstellen einer angepassten Domäne die von IBM Cloud Internet Services bereitgestellten DNS-Server verwenden.

1. Erwerben Sie eine Domäne von einem Registrator (z. B. [http://godaddy.com](http://godaddy.com)).
2. Navigieren Sie zu [Internet Services](https://{DomainName}/catalog/services/internet-services) im {{site.data.keyword.Bluemix_notm}}-Katalog.
2. Geben Sie einen Servicenamen ein und klicken Sie auf **Erstellen**, um eine Instanz des Service zu erstellen.
3. Legen Sie nach dem Bereitstellen der Serviceinstanz Ihren Domänennamen fest und klicken Sie auf **Domäne hinzufügen**.
4. Konfigurieren Sie nach dem Zuordnen der Namensserver den gewünschten Registrator oder DNS-Provider für die Verwendung der aufgelisteten Namensserver.
5. Nach dem Konfigurieren des gewünschten Registrators oder DNS-Providers kann es bis zu 24 Stunden dauern, bis die Änderungen wirksam werden. Wenn der Status der Domäne auf der Seite 'Übersicht' von *Anstehend* in *Aktiv* geändert ist, können Sie mit dem Befehl `dig <your_domain_name> ns` überprüfen, ob die IBM Cloud-Namensserver verwendet werden.
  {:tip}

## Globalen Lastausgleich in der Anwendung hinzufügen

{: #add_glb}

In diesem Abschnitt verwenden Sie die globale Lastausgleichsfunktion (Global Load Balancer, GLB) in IBM Cloud Internet Services, um den Datenverkehr auf mehrere Standorte zu verteilen. GLB ermöglicht durch die Verwendung eines Ursprungspools das Verteilen des Datenverkehrs auf mehrere Ausgangspunkte.

### Erstellen Sie vor der Erstellung der GLB eine Statusprüfung für die GLB.

1. Rufen Sie in der Anwendung 'Cloud Internet Services' **Zuverlässigkeit** > **Globale Lastausgleichsfunktion** auf und klicken Sie unten auf der Seite auf **Statusprüfung erstellen**.
2. Geben Sie den Pfad ein, den Sie überwachen möchten (z. B. `/`) und wählen Sie einen Typ (HTTP oder HTTPS) aus. In der Regel können Sie einen dedizierten Statusendpunkt erstellen. Klicken Sie auf **1 Instanz bereitstellen**.
   ![Statusprüfung](images/solution1/health_check.png)

### Erstellen Sie anschließend einen Ursprungspool mit zwei Ursprüngen.

1. Klicken Sie auf **Pool erstellen**.
2. Geben Sie einen Namen für den Pool ein, wählen Sie die Statusprüfung aus, die Sie soeben erstellt haben, und eine Region, die sich in der Nähe des Standorts Ihrer Node.js-Anwendung befindet.
3. Geben Sie einen Namen für den ersten Ursprung und den Hostnamen für die Anwendung in Dallas (`<your_app>.mybluemix.net`) ein.
4. Fügen Sie auf dieselbe Weise einen weiteren Ursprung hinzu, dessen Adresse auf die Anwendung in London (`<your_app>.eu-gb.mybluemix.net`) verweist.
5. Klicken Sie auf **1 Instanz bereitstellen**.
   ![Ursprungspool](images/solution1/origin_pool.png)

### Erstellen Sie eine globale Lastausgleichsfunktion (Global Load Balancer, GLB). 

1. Klicken Sie auf **Lastausgleichsfunktion erstellen**.
2. Geben Sie einen Namen für die globale Lastausgleichsfunktion ein. Dieser Name wird auch ein Bestandteil Ihrer universellen Anwendungs-URL (`http://<glb_name>.<your_domain_name>`), unabhängig vom Standort.
3. Klicken Sie auf **Pool hinzufügen** und wählen Sie den Ursprungspool aus, den Sie soeben erstellt haben.
4. Klicken Sie auf **1 Instanz bereitstellen**.
   ![Globale Lastausgleichsfunktion](images/solution1/load_balancer.png)

In dieser Phase ist die globale Lastausgleichsfunktion (GLB) konfiguriert, aber die Cloud Foundry-Anwendungen können noch nicht auf Anforderungen des konfigurierten GLB-Domänennamens antworten. Zur Vervollständigung der Konfiguration aktualisieren Sie die Anwendungen mit Routen, die die angepasste Domäne verwenden.

## Angepasste Domäne und Routen für Ihre Anwendung konfigurieren

{: #add_domain}

In diesem Schritt ordnen Sie den Namen der angepassten Domäne dem sicheren Endpunkt für den {{site.data.keyword.Bluemix_notm}}-Standort zu, an dem Ihre Anwendung ausgeführt wird.

1. Klicken Sie in der Menüleiste auf **Verwalten** und anschließend auf **Konto**: [Konto](https://{DomainName}/account).
2. Navigieren Sie auf der Seite für das Konto zur Anwendung **Cloud Foundry-Organisationen** und wählen Sie **Domänen** in der Spalte 'Aktionen' aus.
3. Klicken Sie auf **Domäne hinzufügen** und geben Sie den Namen Ihrer angepassten Domäne ein, die Sie von einem Registrator erhalten haben.
4. Wählen Sie den entsprechenden Standort aus und klicken Sie auf **Speichern**.
5. Fügen Sie in ähnlicher Weise den Namen der angepassten Domäne zum Standort London hinzu.
6. Rufen Sie wieder die {{site.data.keyword.Bluemix_notm}} [Ressourcenliste](https://{DomainName}/resources) auf, navigieren Sie zu **Cloud Foundry-Apps**, klicken Sie auf die Anwendung in Dallas, klicken Sie auf **Route** > **Routen bearbeiten** und dann auf **Route hinzufügen**.
   ![Route hinzufügen](images/solution1/ApplicationRoutes.png)
7. Geben Sie den GLB-Hostnamen, den Sie zuvor konfiguriert haben, in das Feld **Host eingeben (optional)** ein und wählen Sie die angepasste Domäne aus, die Sie soeben hinzugefügt haben. Klicken Sie auf **Speichern**.
8. Konfigurieren Sie in ähnlicher Weise die Domäne und die Routen für die Anwendung am Standort London.

Sie können Ihre Anwendung nun über die URL `glb_name.ihr_domänenname` aufrufen und die globale Lastausgleichsfunktion verteilt automatisch den Datenverkehr für Ihre Anwendungen mit mehreren Standorten. Sie können dies überprüfen, indem Sie Ihre Anwendung am Standort Dallas stoppen, die Anwendung am Standort London aktiviert lassen und über die Lastausgleichsfunktion auf die Anwendung zugreifen.

Dies funktioniert zwar momentan, aber da in den vorherigen Schritten die Continuous Delivery konfiguriert wurde, kann die Konfiguration möglicherweise überschrieben werden, wenn ein weiterer Build ausgelöst wird. Um die Änderungen permanent festzuschreiben, kehren Sie zu den Toolchains zurück und ändern Sie die Datei *manifest.yml*:

1. Navigieren Sie in der {{site.data.keyword.Bluemix_notm}}-[Ressourcenliste](https://{DomainName}/resources) zu **Cloud Foundry-Apps** und klicken Sie auf die Anwendung am Standort Dallas, navigieren Sie dann zur **Übersicht** der Anwendung und blättern Sie zu **Toolchain anzeigen**.
2. Wählen Sie die Kachel 'Git' unter 'Code' aus.
3. Wählen Sie *manifest.yml* aus.
4. Klicken Sie auf **Bearbeiten** und fügen Sie angepasste Routen hinzu. Ersetzen Sie die ursprünglichen Domänen- und Hostkonfigurationen durch Routen (`routes`).

   ```
   applications:
   - path: .
	  name: <ihre_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <ihre_app>.mybluemix.net
	  - route: <glb-name>.<ihr_domänenname>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. Schreiben Sie die Änderungen fest und stellen Sie sicher, dass die Builds für beide Standorte erfolgreich ausgeführt werden.  

## Alternative: Angepasste Domäne der IBM Cloud-Systemdomäne zuordnen

Es kann vorkommen, dass Sie keine vorgeschaltete globale Lastausgleichsfunktion vor Ihren Anwendungen mit mehreren Standorten nutzen möchten, jedoch den Namen der angepassten Domäne dem sicheren Endpunkt für den {{site.data.keyword.Bluemix_notm}}-Standort zuordnen müssen, an dem Ihre Anwendung ausgeführt wird.

Führen Sie mit der Cloud Internet Services-Anwendung die folgenden Schritte aus, um `CNAME`-Datensätze für Ihre Anwendung einzurichten:

1. Rufen Sie in der Anwendung 'Cloud Internet Services' **Zuverlässigkeit** > **DNS** auf.
2. Wählen Sie **CNAME** in der Dropdown-Liste **Typ** aus, geben Sie einen Alias für Ihre Anwendung in das Feld 'Name' und die Anwendungs-URL in das Feld für den Domänennamen ein. Die Anwendung `<your_app>.mybluemix.net` am Standort Dallas kann einem CNAME `<your_app>` zugeordnet werden.
3. Klicken Sie auf **Datensatz hinzufügen**. Setzen Sie das Umschaltsteuerelement PROXY auf ON, um die Sicherheit Ihrer Anwendung zu verbessern.
4. Legen Sie auf ähnliche Weise den Datensatz `CNAME` für den Standortendpunkt in London fest.
   ![CNAME-Datensätze](images/solution1/cnames.png)

Wenn Sie eine andere Standarddomäne als `mybluemix.net` verwenden (z. B. `cf.appdomain.cloud` oder `cf.cloud.ibm.com`), stellen Sie sicher, dass die [entsprechende Systemdomäne](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain) verwendet wird.
{:tip}

Wenn Sie einen anderen DNS-Provider verwenden, variieren die Schritte zum Einrichten des CNAME-Datensatzes entsprechend dem verwendeten DNS-Provider. Wenn Sie beispielsweise den Provider GoDaddy verwenden, folgen Sie der Anleitung unter [Domains Help](https://www.godaddy.com/help/add-a-cname-record-19236) von GoDaddy.

Damit Ihre Cloud-Foundry-Anwendungen über die angepasste Domäne erreichbar sind, müssen Sie die angepasste Domäne zur [Domänenliste in der Cloud Foundry-Organisation, in der die Anwendungen bereitgestellt werden](https://{/docs/apps?topic=creating-apps-updatingapps#updatingapps) hinzufügen. Nach diesem Vorgang können Sie die Routen zu den Anwendungsmanifesten hinzufügen:

   ```
   applications:
   - path: .
	  name: <ihre_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <ihre_app>.mybluemix.net
	  - route: <ihre_app>.<ihr_domänenname>
	  disk_quota: 1024M
   ```
   {: pre}

## SSL-Zertifikat an Ihre Anwendung binden
{: #ssl}

1. Fordern Sie ein SSL-Zertifikat an. Sie können beispielsweise ein Zertifikat unter https://www.godaddy.com/web-security/ssl-certificate erwerben oder ein kostenloses Zertifikat unter https://letsencrypt.org/ generieren.
2. Navigieren Sie unter 'Anwendung' zu **Übersicht** > **Routen** > **Domänen verwalten**.
3. Klicken Sie auf die Schaltfläche zum Hochladen des SSL-Zertifikats und laden Sie das Zertifikat hoch.
5. Greifen Sie über HTTPS auf Ihre Anwendung zu anstatt über HTTP.

## Leistung der Anwendung überwachen
{: #monitor}

Prüfen Sie nun den allgemeinen Status Ihrer Anwendung mit mehreren Standorten.

1. Wählen Sie im Anwendungsdashboard die Option **Überwachung** aus.
2. Klicken Sie auf **Alle Tests anzeigen**
   ![](images/solution1/alert_frequency.png).

Bei der Verfügbarkeitsüberwachung werden rund um die Uhr synthetische Tests über Standorte in der ganzen Welt ausgeführt, um Leistungsprobleme proaktiv zu erkennen und zu beheben, bevor die Benutzer davon betroffen sind. Wenn Sie ein angepasste Route für Ihre Anwendung konfiguriert haben, ändern Sie die Testdefinition, um über die angepasste Domäne auf Ihre Anwendung zuzugreifen.

## Ressourcen entfernen

* Löschen Sie die Toolchain.
* Löschen Sie die beiden, an zwei Standorten bereitgestellten Cloud Foundry-Anwendungen.
* Löschen Sie die GLB, die Ursprungspools und die Statusprüfung.
* Löschen sie die DNS-Konfiguration.
* Löschen Sie die Internet Services-Instanz.

## Zugehörige Inhalte

[Cloudant-Datenbank hinzufügen](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[Cloud Foundry-Anwendungen automatisch skalieren](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
