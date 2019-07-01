---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# Fortlaufende Bereitstellung in Kubernetes
{: #continuous-deployment-to-kubernetes}

In diesem Lernprogramm wird gezeigt, wie Sie eine CI/CD-Pipeline (CI, Continuous Intergration; CD, Continuous Delivery) für containerisierte Anwendungen einrichten, die in {{site.data.keyword.containershort_notm}} ausgeführt werden. Sie erfahren, wie Sie die Quellcodeverwaltung einrichten und anschließend den Code in verschiedenen Implementierungsphasen erstellen, testen und bereitstellen. Außerdem fügen Sie Integrationen zu anderen Services wie Sicherheitsscannern, Slack-Benachrichtigungen und Analysen hinzu.

{:shortdesc}

## Lernziele
{: #objectives}

* Einen Kubernetes-Cluster für die Entwicklung und Produktion erstellen
* Eine Starteranwendung erstellen, diese lokal ausführen und mit einer Push-Operation an ein Git-Repository übertragen
* DevOps Delivery Pipeline für die Verbindung zum Git-Repository konfigurieren und die Starter-App erstellen und in Clustern für die Entwicklung/Produktion implementieren
* Die App untersuchen und für die Verwendung von Sicherheitsscannern, Slack-Benachrichtigungen und Analysen integrieren

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden {{site.data.keyword.Bluemix_notm}}-Services verwendet:

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**Achtung:** Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur
{: #architecture}

![](images/solution21/Architecture.png)

1. Der Code wird mit einer Push-Operation an ein privates Git-Repository übertragen.
2. Die Pipeline nimmt die Änderungen in Git auf und erstellt ein Container-Image.
3. Das Container-Image wurde in die Registry hochgeladen, die in einem Kubernetes-Cluster für die Entwicklung implementiert ist.
4. Die Änderungen werden überprüft und im Produktionscluster implementiert.
5. Slack-Benachrichtigungen werden für Bereitstellungsaktivitäten eingerichtet.


## Vorbereitende Schritte
{: #prereq}

* [Installieren Sie {{site.data.keyword.dev_cli_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - Script zur Installation von Docker, kubectl, Helm, 'ibmcloud'-CLI und der erforderlichen Plug-ins.
* [Richten Sie die {{site.data.keyword.registrylong_notm}}-CLI und den Namensbereich der Registry ein](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Machen Sie sich mit den Grundlagen von Kubernetes vertraut](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Kubernetes-Cluster für die Entwicklung erstellen
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} bietet leistungsfähige Tools durch die Kombination von Docker- und Kubernetes-Technologien, eine intuitive Benutzererfahrung sowie integrierte Sicherheit und Isolation, um die Implementierung, den Betrieb, die Skalierung und die Überwachung von containerisierten Apps in einem Cluster von Rechenhosts zu automatisieren.

Um dieses Lernprogramm ausführen zu können, müssen Sie den **kostenpflichtigen** Cluster des Typs **Standard** auswählen. Sie müssen zwei Cluster einrichten, einen für die Entwicklung und einen für die Produktion.
{: shortdesc}

1. Erstellen Sie den ersten Kubernetes-Cluster für die Entwicklung über den [{{site.data.keyword.Bluemix}}-Katalog](https://{DomainName}/containers-kubernetes/launch). Später müssen diese Schritte für einen Produktionscluster wiederholen.

   Überprüfen Sie zur Benutzerfreundlichkeit die Konfigurationsdetails wie die Anzahl der CPUs, den Speicher und die Anzahl der Workerknoten, die Sie erhalten.
   {:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. Wählen Sie den **Clustertyp** aus und klicken Sie auf **Cluster erstellen**, um einen Kubernetes-Cluster bereitzustellen. Der kleinste **Maschinentyp** mit zwei **CPUs**, vier **GB RAM** und einem **Workerknoten** ist für dieses Lernprogramm ausreichend. Für alle anderen Optionen können die Standardwerte übernommen werden.
3. Überprüfen Sie den Status der **Cluster** und **Workerknoten** und warten Sie, bis sie **bereit** sind.

**Hinweis:** Fahren Sie erst fort, wenn die Workerknoten bereit sind.

## Starteranwendung erstellen
{: #create_application}

{{site.data.keyword.containershort_notm}} bietet eine Auswahl von Starteranwendungen, die mit dem Befehl `ibmcloud dev create` oder über die Webkonsole erstellt werden können. In diesem Lernprogramm wird die Webkonsole verwendet. Die Starteranwendung reduziert die Entwicklungszeit erheblich, indem sie Anwendungsstarter mit allen erforderlichen Boilerplate-, Build- und Konfigurationscode generiert, so dass Sie schneller mit dem Codieren der Geschäftslogik beginnen können.

1. Verwenden Sie in der [{{site.data.keyword.cloud_notm}}-Konsole](https://{DomainName}) die Menüoption auf der linken Seite und wählen Sie [Web-Apps](https://{DomainName}/developer/appservice/dashboard) aus.
2. Klicken Sie im Abschnitt **Im Web starten** auf die Schaltfläche **Einstieg**.
3. Wählen Sie die Kachel `Node.js-Web-App mit Express.js` und anschließend `App erstellen` aus, um eine Node.js-Starteranwendung zu erstellen.
4. Geben Sie als **Namen** den Namen `mynodestarter` ein. Klicken Sie dann auf **Erstellen**.

## DevOps Delivery Pipeline
{: #create_devops}

1. Nachdem Sie die Startanwendung erfolgreich erstellt haben, klicken Sie unter **App bereitstellen** auf die Schaltfläche **In Cloud bereitstellen**.
2. Wählen Sie die Bereitstellungsmethode 'Kubernetes-Cluster' und anschließend den zuvor erstellten Cluster aus und klicken Sie auf **Erstellen**. Dadurch werden eine Toolchain und eine Delivery Pipeline erstellt. ![](images/solution21/BindCluster.png)
3. Sobald die Pipeline erstellt wurde, klicke Sie auf **Toolchain anzeigen** und dann auf **Delivery Pipeline**, um die Pipeline anzuzeigen. ![](images/solution21/Delivery-pipeline.png)
4. Klicken Sie nach Abschluss der Bereitstellungsphasen auf **Protokolle und Verlauf anzeigen**, um die Protokolle anzuzeigen.
5. Rufen Sie die URL für den Zugriff auf die Anwendung auf (`http://worker-public-ip:portnumber/`). ![](images/solution21/Logs.png)
Sie haben es geschafft. Sie haben mithilfe der Benutzerschnittstelle des App-Service die Starteranwendungen erstellt und die Pipeline konfiguriert, um die Anwendung zu erstellen und im Cluster bereitzustellen.

## Anwendung klonen, erstellen und lokal ausführen
{: #cloneandbuildapp}

In diesem Abschnitt verwenden Sie die Starter-App, die Sie im vorherigen Abschnitt erstellt haben, klonen sie auf Ihre lokale Maschine, ändern den Code und erstellen/führen sie dann lokal aus.
{: shortdesc}

### Anwendung klonen
1. Wählen Sie in der Toolchain-Übersicht die Kachel **Git** unter **Code** aus. Sie werden auf Ihre Git-Repository-Seite umgeleitet, auf der Sie das Repository klonen können.![HelloWorld](images/solution21/DevOps_Toolchain.png)

2. Wenn Sie noch keine SSH-Schlüssel eingerichtet haben, sollten Sie oben eine Benachrichtigungsleiste mit entsprechenden Anweisungen sehen. Führen Sie entsprechenden Schritte aus, indem Sie über den Link **SSH-Schlüssel hinzufügen** eine neue Registerkarte öffnen. Wenn Sie HTTPS anstelle von SSH verwenden möchten, führen Sie die entsprechenden Schritte aus, indem Sie auf **Persönliches Zugriffstoken erstellen** klicken. Denken Sie daran, den Schlüssel oder das Token als zukünftige Referenz zu speichern.
3. Wählen Sie SSH oder HTTPS aus und kopieren Sie die Git-URL. Klonen Sie die Quelle auf Ihre lokale Maschine. Wenn Sie zur Eingabe eines Benutzernamens aufgefordert werden, geben Sie Ihren Git-Benutzernamen an. Verwenden Sie für das Kennwort einen vorhandenen **SSH-Schlüssel** oder ein **persönliches Zugriffstoken** oder das im vorherigen Schritt erstellte Token.

   ```bash
   git clone <ihre_repository-url>
   cd <name_der_app>
   ```
   {: codeblock}

4. Öffnen Sie das geklonte Repository in einer IDE Ihrer Wahl und navigieren Sie zu `public/index.html`. Aktualisieren Sie den Code, indem Sie versuchen, 'Congratulations!' in etwas anderes zu ändern, und speichern Sie dann die Datei.

### Anwendung lokal erstellen
Sie können die Anwendung so erstellen und ausführen, wie Sie es normalerweise tun würden, indem Sie `mvn` für die lokale Java-Entwicklung oder `npm` für die Knotenentwicklung verwenden. Sie können auch ein Docker-Image erstellen und die Anwendung in einem Container ausführen, um eine konsistente Ausführung auf lokaler Ebene und in der Cloud zu gewährleisten. Führen Sie die folgenden Schritte aus, um das Docker-Image zu erstellen.
{: shortdesc}

1. Stellen Sie sicher, dass Ihre lokale Docker-Engine gestartet wurde. Führen Sie dazu den folgenden Befehl aus:
   ```
   docker ps
   ```
   {: codeblock}
2. Navigieren Sie zu dem generierten Projektverzeichnis, das geklont wurde.
   ```
   cd <projektname>
   ```
   {: codeblock}
3. Erstellen Sie die Anwendung lokal.
   ```
   ibmcloud dev build
   ```
   {: codeblock}

   Die Ausführung kann einige Minuten in Anspruch nehmen, da alle Anwendungsabhängigkeiten heruntergeladen werden und ein Docker-Image, das Ihre Anwendungen und alle erforderlichen Umgebungen enthält, erstellt wird.

### Anwendung lokal ausführen

1. Führen Sie den Container aus.
   ```
   ibmcloud dev run
   ```
   {: codeblock}

   Dazu wird die lokale Docker-Engine verwendet, um das Docker-Image auszuführen, das Sie im vorherigen Schritt erstellt haben.
2. Nachdem der Container gestartet wurde, rufen Sie die Adresse 'http://localhost:3000/' auf.
   ![](images/solution21/node_starter_localhost.png)

## Anwendung mit einer Push-Operation in das Git-Repository übertragen

In diesem Abschnitt schreiben Sie die vorgenommene Änderung im Git-Repository fest. Die Pipeline nimmt die Festschreibung auf und überträgt die Änderungen automatisch mit einer Push-Operation an den Cluster.
1. Stellen Sie in Ihrem Terminalfenster sicher, dass Sie sich innerhalb des von Ihnen geklonten Repositorys befinden.
2. Übertragen Sie die Änderung mit drei einfachen Schritten in das Repository: Hinzufügen, festschreiben und Push-Operation durchführen.
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
   {: codeblock}

3. Wechseln Sie zur Toolchain, die Sie zuvor erstellt haben, und klicken Sie auf die Kachel **Delivery Pipeline**.
4. Bestätigen Sie, dass die Stages **BUILD** und **BEREITSTELLUNG** angezeigt werden.
   ![](images/solution21/Delivery-pipeline.png)
5. Warten Sie, bis die Stage **BEREITSTELLUNG** abgeschlossen ist.
6. Klicken Sie unter 'Letztes Ausführungsergebnis' auf die **URL** der Anwendung, um Ihre Änderungen live anzuzeigen.

Wenn Ihre Anwendung nicht aktualisiert wird, überprüfen Sie die Protokolle der BEREITSTELLUNGS- und BUILD-Stages Ihrer Pipeline.

## Sicherheit mit Vulnerability Advisor
{: #vulnerability_advisor}

In diesem Schritt untersuchen Sie den [Vulnerability Advisor](https://{DomainName}/docs/services/va?topic=va-va_index#va_index). Mit Vulnerability Advisor können Sie den Sicherheitsstatus von Container-Images vor der Bereitstellung und den Status von aktiven Containern überprüfen.

1. Wechseln Sie zur Toolchain, die Sie zuvor erstellt haben, und klicken Sie auf die Kachel **Delivery Pipeline**.
1. Klicken Sie auf **Stage hinzufügen** und ändern Sie 'MyStage' in **Stage 'Überprüfen'**. Klicken Sie anschließend auf 'JOBS' > **JOB HINZUFÜGEN**.

   1. Wählen Sie **Test** als Jobtyp aus und ändern Sie im Feld den Eintrag **Test** in **Vulnerability Advisor**.
   1. Wählen Sie unter 'Testtyp' die Option **Vulnerability Advisor** aus. Alle anderen Felder sollten automatisch gefüllt werden.
      Der Namensbereich der Container-Registry sollte mit dem in dieser Toolchain unter **Build-Stage** erwähnten Namensbereich übereinstimmen.
      {:tip}
   1. Bearbeiten Sie den Abschnitt **Testscript** und ersetzen Sie `SAFE\ to\ deploy` in der letzten Zeile durch `NO\ ISSUES`.
   1. Speichern Sie die Stage.
1. Ziehen und verschieben Sie die **Stage 'Überprüfen'** in die Mitte und klicken Sie dann auf **Ausführen** ![](images/solution21/run.png) für die **Stage 'Überprüfen'**. Es wird angezeigt, dass die **Stage 'Überprüfen'** fehlschlägt.

   ![](images/solution21/toolchain.png)

1. Klicken Sie auf **Protokolle und Verlauf anzeigen**, um die Anfälligkeitsbewertung anzuzeigen. Am Ende des Protokolls heißt es:

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   Die detaillierten Anfälligkeitsbewertungen aller gescannten Repositorys werden [hier](https://{DomainName}/containers-kubernetes/registry/private) aufgeführt.
   {:tip}

   Die Stage kann aus dem Grund fehlschlagen, dass das Image *nicht gescannt wurde*, wenn der Scan auf Anfälligkeiten länger als drei Minuten dauert. Dieses Zeitlimit kann geändert werden, indem das Job-Script bearbeitet und die Anzahl der Iterationen erhöht wird, um auf die Scanergebnisse zu warten.
   {:tip}

1. Beheben Sie die Schwachstellen, indem Sie die Korrekturmaßnahme befolgen. Öffnen Sie das geklonte Repository in einer IDE oder wählen Sie die Kachel für die Web-IDE von Eclipse Orion aus, öffnen Sie die `Dockerfile` und fügen Sie den folgenden Befehl nach `EXPOSE 3000` hinzu.
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. Schreiben Sie die Änderungen fest und übertragen Sie die Änderungen mit einer Push-Operation. Dadurch sollte die Toolchain ausgelöset und die **Stage 'Überprüfen'** korrigiert werden.

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## Kubernetes-Cluster für die Produktion erstellen

{: #deploytoproduction}

In diesem Abschnitt schließen Sie die Bereitstellungspipeline ab, indem Sie die Kubernetes-Anwendung entsprechend in Entwicklungs- und Produktionsumgebungen bereitstellen. Idealerweise sollten Sie eine automatische Bereitstellung für die Entwicklungsumgebung und eine manuelle Bereitstellung für die Produktionsumgebung einrichten. Untersuchen Sie jedoch zunächst die beiden Möglichkeiten, die Ihnen dazu zur Verfügung stehen. Es ist möglich, einen Cluster sowohl für die Entwicklungs- als auch für die Produktionsumgebung zu verwenden. Es wird jedoch empfohlen, zwei separate Cluster zu verwenden (einen für die Entwicklung und einen für die Produktion). Im folgenden Abschnitt erfahren Sie, wie Sie einen zweiten Cluster für die Produktion einrichten.
{: shortdesc}

1. Befolgen Sie die Anweisungen im Abschnitt zum [Erstellen eines Kubernetes-Clusters für die Entwicklung](#create_kube_cluster) und erstellen Sie einen neuen Cluster. Nennen Sie diesen Cluster `prod-cluster`.
2. Wechseln Sie zur Toolchain, die Sie zuvor erstellt haben, und klicken Sie auf die Kachel **Delivery Pipeline**.
3. Benennen Sie die **Bereitstellungsstage** in `Deploy dev` um. Klicken Sie dazu auf das Symbol 'Einstellungen' > **Stage konfigurieren**.
   ![](images/solution21/deploy_stage.png)
4. Klonen Sie die Stage **Deploy dev** (Symbol 'Einstellungen' > Stage klonen) und geben Sie der geklonten Stage den Namen `Deploy prod`.
5. Ändern Sie den **Stageauslöser** in `Nur Jobs ausführen, wenn diese Stage manuell ausgeführt wird`. ![](images/solution21/prod-stage.png)
6. Ändern Sie auf der Registerkarte **Job** den Clusternamen in den neu erstellten Cluster und **speichern** Sie dann die Stage.
7. Sie sollten jetzt über die vollständige Bereitstellungskonfiguration verfügen. Für eine Bereitstellung von der Entwicklungs- in die Proktionsumgebung müssen Sie die Stage `Deploy prod` manuell ausführen. ![](images/solution21/full-deploy.png)

Sie haben es geschafft. Sie haben nun einen Produktionscluster erstellt und die Pipeline so konfiguriert, dass Aktualisierungen mit einer manuellen Push-Operation an den Produktionscluster übertragen werden können. Hierbei handelt es sich um eine vereinfachte Prozessphase gegenüber einem erweiterten Szenario, das Komponenten- und Integrationstests als Teil der Pipeline umfassen würde.

## Slack-Benachrichtigungen einrichten
{: #setup_slack}

1. Wechseln Sie zurück zur Liste der [Toolchains](https://{DomainName}/devops/toolchains) und wählen Sie Ihre Toolchain aus. Klicken Sie dann auf **Tool hinzufügen**.
2. Suchen Sie im Suchfeld nach Slack oder blättern Sie nach unten, um **Slack** anzuzeigen. Klicken Sie, um die Konfigurationsseite anzuzeigen.
    ![](images/solution21/configure_slack.png)
3. Führen Sie für einen **Slack-Webhook** die Schritte unter diesem [Link](https://my.slack.com/services/new/incoming-webhook/) aus. Sie müssen sich mit Ihren Slack-Berechtigungsnachweisen anmelden und einen vorhandenen Kanalnamen angeben oder einen neuen Kanal erstellen.
4. Sobald die eingehende Webhook-Integration hinzugefügt wurde, kopieren Sie die **Webhook-URL** und fügen Sie sie unter **Slack-Webhook** ein.
5. Der Slack-Kanal ist der Kanalname, den Sie bei der Erstellung einer Webhook-Integration oben angegeben haben.
6. **Slack-Teamname** ist der Teamname (erster Teil) von 'team-name.slack.com.', d. h., 'kube' ist z. B. der Teamname in 'kube.slack.com'.
7. Klicken Sie auf **Integration erstellen**. Eine neue Kachel wird zu Ihrer Toolchain hinzugefügt.
    ![](images/solution21/toolchain_slack.png)
8. Ab jetzt sollten bei jedem Ausführen Ihrer Toolchain Slack-Benachrichtigungen in dem von Ihnen konfigurierten Kanal angezeigt werden.
    ![](images/solution21/slack_channel.png)

## Ressourcen entfernen
{: #removeresources}

In diesem Schritt bereinigen Sie die Ressourcen so, dass alles, was Sie weiter oben erstellt haben, entfernt wird.

- Löschen Sie das Git-Repository.
- Löschen Sie die Toolchain.
- Löschen Sie die beiden Cluster.
- Löschen Sie den Slack-Kanal.

## Lernprogramm erweitern
{: #expandTutorial}

Möchten Sie mehr erfahren? Im Folgenden finden Sie einige Ideen, was Sie als Nächstes tun können:

- [Analysieren Sie Protokolle und überwachen Sie den Anwendungsstatus mit LogDNA und Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Fügen Sie eine Testumgebung hinzu und implementieren Sie sie in einem dritten Cluster.
- Implementieren Sie den Produktionscluster [über mehrere Standorte](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Erweitern Sie Ihre Pipeline mithilfe von [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) mit zusätzlichen qualitätsverbessernden Steuerelementen und Analysen.

## Zugehöriger Inhalt
{: #related}

* Umfassendes Kubernetes-Lösungshandbuch: [Verschieben von VM-basierten Apps nach Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes)
* [Sicherheit](https://{DomainName}/docs/containers?topic=containers-security#cluster) für den IBM Cloud Container-Service
* Toolchain-[Integrationen](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations)
* Protokolle analysieren und den Anwendungsstatus mit [LogDNA und Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis) überwachen


