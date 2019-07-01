---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Protokolle analysieren und den Anwendungsstatus mit LogDNA und Sysdig überwachen
{: #application-log-analysis}

Dieses Lernprogramm zeigt, wie der [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/)-Service zum Konfigurieren und Zugreifen auf Protokolle einer Kubernetes-Anwendung verwendet werden kann, die in {{site.data.keyword.Bluemix_notm}} implementiert ist. Sie implementieren eine Python-Anwendung in einem Cluster, der in {{site.data.keyword.containerlong_notm}} bereitgestellt wurde, konfigurieren einen LogDNA-Agenten, generieren verschiedene Ebenen von Anwendungsprotokollen und greifen auf Worker-, Pod- oder Netzprotokolle zu. Anschließend durchsuchen, filtern und visualisieren Sie diese Protokolle über die {{site.data.keyword.la_short}}-Webbenutzerschnittstelle.

Darüber hinaus konfigurieren Sie auch den [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/)-Service und den Sysdig-Agenten, um die Leistung und den Status Ihrer Anwendung und Ihres {{site.data.keyword.containerlong_notm}}-Clusters zu überwachen.
{:shortdesc}

## Lernziele
{: #objectives}
* Eine Anwendung für einen Kubernetes-Cluster zum Generieren von Protokolleinträgen implementieren
* Auf unterschiedliche Typen von Protokollen zugreifen und diese analysieren, um Probleme zu beheben und möglichen Problemen zuvorzukommen
* Operative Einsichten in die Leistung und den Status Ihrer App und des Clusters erhalten, auf dem die App ausgeführt wird

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur
{: #architecture}

  ![](images/solution12/Architecture.png)

1. Der Benutzer stellt eine Verbindung zu der Anwendung her und generiert Protokolleinträge.
1. Die Anwendung wird in einem Kubernetes-Cluster aus einem Image ausgeführt, das in der {{site.data.keyword.registryshort_notm}} gespeichert ist. 
1. Der Benutzer konfiguriert den {{site.data.keyword.la_full_notm}}-Serviceagent für den Zugriff auf Protokolle auf Anwendungs- und Clusterebene.
1. Der Benutzer konfiguriert den {{site.data.keyword.mon_full_notm}}-Serviceagent zum Überwachen des Status und der Leistung des {{site.data.keyword.containerlong_notm}}-Clusters sowie der Anwendung, die für den Cluster implementiert wurde.

## Voraussetzungen
{: #prereq}

* [Installieren Sie {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli) - Script zur Installation von Docker, kubectl, Helm, 'ibmcloud'-CLI und der erforderlichen Plug-ins.
* [Richten Sie die {{site.data.keyword.registrylong_notm}}-CLI und den Namensbereich der Registry ein](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Erteilen Sie Berechtigungen für einen Benutzer zum Anzeigen von Protokollen in LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna).
* [Erteilen Sie Berechtigungen für einen Benutzer zum Anzeigen von Metriken in Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig).

## Kubernetes-Cluster erstellen
{: #create_cluster}

{{site.data.keyword.containershort_notm}} stellt eine Umgebung bereit, um hoch verfügbare Apps in Docker-Containern zu implementieren, die in Kubernetes-Clustern ausgeführt werden.

Überspringen Sie diesen Abschnitt, wenn Sie über einen vorhandenen **Standard**-Cluster verfügen, den Sie in diesem Lernprogramm wiederverwenden möchten.
{: tip}

1. Erstellen Sie **einen neuen Cluster** über den [{{site.data.keyword.Bluemix}}-Katalog](https://{DomainName}/kubernetes/catalog/cluster/create) und wählen Sie den **Standard**-Cluster aus.
   Für den **kostenlosen** Cluster ist die Protokollweiterleitung *nicht* aktiviert.
   {:tip}
1. Wählen Sie eine Ressourcengruppe und eine Region aus.
1. Verwenden Sie aus Gründen der Konsistenz für dieses Lernprogramm den Namen `mycluster`.
1. Wählen Sie eine **Workerzone** aus und wählen Sie den kleinsten **Maschinentyp** mit zwei **CPUs** und vier **GB RAM** aus, da dies für dieses Lernprogramm ausreichend ist.
1. Wählen Sie einen **Workerknoten** aus und belassen Sie für alle anderen Optionen die Standardeinstellung. Klicken Sie auf **Cluster erstellen**.
1. Überprüfen Sie den Status des **Clusters** und des **Workerknotens** und warten Sie, bis sie **bereit** sind.

## {{site.data.keyword.la_short}}-Instanz bereitstellen
{: #provision_logna_instance}

Anwendungen, die in einem {{site.data.keyword.containerlong_notm}}-Cluster in {{site.data.keyword.Bluemix_notm}} implementiert sind, generieren in der Regel eine bestimmte Art von Diagnosenachricht, d. h. Protokolle. Als Entwickler oder Bediener möchten Sie möglicherweise auf verschiedene Typen von Protokollen, wie Worker-, Pod-, App- oder Netzprotokolle, zugreifen und diese analysieren, um Probleme zu beheben und möglichen Problemen zuvorzukommen.

Wenn Sie den {{site.data.keyword.la_short}}-Service verwenden, ist es möglich, Protokolle aus verschiedenen Quellen zusammenzufassen und sie so lange wie nötig zu behalten. Dies erlaubt es, das "große Ganze" bei Bedarf zu analysieren und ggf. Fehler in komplexeren Situationen zu beheben.

Gehen Sie folgendermaßen vor, um einen {{site.data.keyword.la_short}}-Service bereitzustellen:

1. Navigieren Sie zur Seite für die [Beobachtbarkeit](https://{DomainName}/observe/) und klicken Sie unter **Protokollierung** auf **Instanz erstellen**.
1. Geben Sie einen eindeutigen **Servicenamen** an.
1. Wählen Sie eine Region/einen Standort und eine Ressourcengruppe aus.
1. Wählen Sie die **7-tägige Protokollsuche** als Plan aus und klicken Sie auf **Erstellen**.

Der Service stellt ein zentrales Protokollverwaltungssystem bereit, in dem Protokolldaten in IBM Cloud gehostet werden.

## Kubernetes-App bereitstellen und zum Weiterleiten von Protokollen konfigurieren
{: #deploy_configure_kubernetes_app}

Der ausführungsbereite [Code für die Protokollierungsapp befindet sich in diesem GitHub-Rpository](https://github.com/IBM-Cloud/application-log-analysis). Die Anwendung wurde in [Django](https://www.djangoproject.com/) geschrieben, einem beliebten serverseitgen Python-Web-Framework. Klonen Sie das Repository oder laden Sie es herunter und stellen Sie dann die App in {{site.data.keyword.containershort_notm}} unter {{site.data.keyword.Bluemix_notm}} bereit.

### Python-Anwendung implementieren

Auf einem Terminal:
1. Klonen Sie das GitHub-Repository:
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. Wechseln Sie in das Anwendungsverzeichnis:
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. Erstellen Sie ein Docker-Image mit der [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile) in {{site.data.keyword.registryshort_notm}}.
   - Suchen Sie die **Container-Registry** mit `ibmcloud cr info`, wie z. B. 'us.icr.io' oder 'uk.icr.io'.
   - Erstellen Sie einen Namensbereich zum Speichern des Container-Image.
      ```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - Ersetzen Sie `<CONTAINER_REGISTRY>` durch Ihren Container-Registry-Wert und verwenden Sie **app-log-analysis** als Imagenamen.
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - Ersetzen Sie den Wert für **image** in der Datei `app-log-analysis.yaml` durch das Image-Tag `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest`.
1. Führen Sie den folgenden Befehl aus, um die Clusterkonfiguration abzurufen, und legen Sie die Umgebungsvariable `KUBECONFIG` fest:
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. Implementieren Sie die App:
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. Für den Zugriff auf die Anwendung benötigen Sie die `öffentliche IP` des Workerknotens und den Knotenport (`NodePort`).
   - Führen Sie für die öffentliche IP den folgenden Befehl aus:
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - Führen Sie für den 5-stelligen Knotenport (z. B. 3xxxx) den folgenden Befehl aus:
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   Sie können jetzt auf die Anwendung unter `http://worker-ip-adresse:portnummer` zugreifen.

### Cluster zum Senden von Protokollen an die LogDNA-Instanz konfigurieren

Wenn Sie Ihren Kubernetes-Cluster so konfigurieren möchten, dass er Protokolle an Ihre {{site.data.keyword.la_full_notm}}-Instanz sendet, müssen Sie auf jedem Knoten des Clusters einen *logdna-agent*-Pod installieren. Der LogDNA-Agent liest Protokolldateien aus dem Pod, in dem er installiert ist, und leitet die Protokolldaten an Ihre LogDNA-Instanz weiter.

1. Navigieren Sie zur Seite für die [Beobachtbarkeit](https://{DomainName}/observe/) und klicken Sie auf **Protokollierung**.
1. Klicken Sie neben dem Service, den Sie zuvor erstellt haben, auf **Protokollressourcen bearbeiten** und wählen Sie **Kubernetes** aus.
1. Kopieren Sie den ersten Befehl und führen Sie ihn auf einem Terminal aus, auf dem Sie die Umgebungsvariable `KUBECONFIG` festgelegt haben, um einen geheimen Kubernetes-Schlüssel mit dem LogDNA-Aufnahmeschlüssel für Ihre Serviceinstanz zu erstellen.
1. Kopieren Sie den zweiten Befehl und führen Sie ihn aus, um einen LogDNA-Agenten auf jedem Workerknoten Ihres Kubernetes-Clusters zu implementieren. Der LogDNA-Agent erfasst Protokolle mit der Erweiterung **.log* und erweiterungslose Dateien, die im Verzeichnis *'/var/log*' des Pods gespeichert sind. Standardmäßig werden Protokolle aus allen Namensbereichen, einschließlich 'kube-system', erfasst und automatisch an den {{site.data.keyword.la_full_notm}}-Service weitergeleitet.
1. Nachdem Sie eine Protokollquelle konfiguriert haben, starten Sie die LogDNA-Benutzerschnittstelle, indem Sie auf **LogDNA anzeigen** klicken. Es kann einige Minuten dauern, bis die Protokolle angezeigt werden.

## Anwendungsprotokolle generieren und auf diese zugreifen
{: generate_application_logs}

In diesem Abschnitt generieren Sie Anwendungsprotokolle und überprüfen sie in LogDNA.

### Anwendungsprotokolle generieren

Die in den vorherigen Schritten implementierte Anwendung ermöglicht es Ihnen, eine Nachricht auf einer ausgewählten Protokollebene zu protokollieren. Die verfügbaren Protokollebenen sind **critical**, **error**, **warn**, **info** und **debug**. Die Protokollierungsinfrastruktur der Anwendung ist so konfiguriert, dass nur Protokolleinträge auf oder über einer festgelegten Ebene übergeben werden können. Anfänglich ist die Protokollfunktionsebene auf **warn** gesetzt. Eine Nachricht, die als **info** mit der Servereinstellung **warn** protokolliert wird, würde daher nicht in der Diagnosenachricht enthalten sein.

Werfen Sie einen Blick auf den Code in der Datei [**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py). Der Code enthält sowohl **print**-Anweisungen als auch Aufrufe an **Protokoll**funktionen. Gedruckte Nachrichten werden in den **stdout**-Stream geschrieben (reguläre Ausgabe, Anwendungskonsole/Terminal), Protokollfunktionsnachrichten werden im **stderr**-Stream (Fehlerprotokoll) angezeigt.

1. Rufen Sie die Web-App unter `http://worker-ip-address:portnumber` auf.
1. Generieren Sie mehrere Protokolleinträge, indem Sie Nachrichten auf verschiedenen Ebenen übergeben. Über die Benutzerschnittstelle können Sie die Einstellung der Protokollfunktion auch für die Serverprotokollebene ändern. Ändern Sie die serverseitige Protokollebene zwischendurch, um die Sache noch interessanter zu gestalten. Beispielsweise können Sie eine Nachricht des Typs "500 Interner Serverfehler" als **error** oder "Dies ist mein erster Protokolleintrag" als **info** protokollieren.

### Auf Anwendungsprotokolle zugreifen

Sie können mit den Filtern auf das anwendungsspezifische Protokoll in der LogDNA-Benutzerschnittstelle zugreifen.

1. Klicken Sie in der oberen Leiste auf **Alle Apps**.
1. Wählen Sie unter den Container die Option **app-log-analysis** aus. Eine neue nicht gespeicherte Ansicht wird mit Anwendungsprotokollen aller Ebenen angezeigt.
1. Um Protokolle für (eine) bestimmte Protokollebene(n) anzuzeigen, klicken Sie auf **Alle Ebenen** und wählen Sie mehrere Ebenen wie 'error', 'info', 'warn' usw. aus.

## Protokolle suchen und filtern
{: #search_filter_logs}

In der {{site.data.keyword.la_short}}-Benutzerschnittstelle werden standardmäßig alle verfügbaren Protokolleinträge angezeigt. Die letzten Einträge werden im unteren Bereich durch eine automatische Aktualisierung angezeigt.
In diesem Abschnitt ändern Sie, was und wie viel angezeigt wird, und speichern dies als **Ansicht** für eine zukünftige Verwendung.

### Protokolle durchsuchen

1. Im Eingabefeld **Suchen**, das sich unten auf der Seite in der LogDNA-Benutzerschnittstelle befindet, können Sie:
   - nach Zeilen suchen, die einen bestimmten Text wie **"Dies ist mein erster Protokolleintrag"** oder **500 Interner Serverfehler** enthalten
   - nach einer bestimmten Protokollebene suchen, indem Sie `level:info` eingeben, wobei 'level' ein Feld ist, das einen Zeichenfolgewert akzeptiert.

   Wenn Sie weitere Suchfelder und Hilfe anzeigen möchten, klicken Sie auf das Symbol für die Syntaxhilfe neben dem Eingabefeld für die Suche.
   {:tip}
1. Um zu einem bestimmten Zeitrahmen zu wechseln, geben Sie **Vor 5 Minuten** in das Eingabefeld für den **Sprung zum Zeitrahmen** ein. Klicken Sie auf das Symbol neben dem Eingabefeld, um die anderen Zeitformate in Ihrem Aufbewahrungszeitraum zu finden.
1. Um die Begriffe hervorzuheben, klicken Sie auf das Symbol **Viewer-Tools ein-/ausschalten**.
1. Geben Sie **error** als Hervorhebungsbegriff in das erste Eingabefeld ein, **container** als Hervorhebungsbegriff in das zweite Eingabefeld und markieren Sie die hervorgehobenen Zeilen mit den Begriffen.
1. Klicken Sie auf das Symbol **Zeitachse ein-/ausschalten**, um Zeilen mit Protokollen zu einer bestimmten Uhrzeit eines Tages anzuzeigen.

### Protokolle filtern

Sie können Protokolle nach Tags, Quellen, Apps oder Ebenen filtern.

1. Klicken Sie in der oberen Leiste auf **Alle Tags** und wählen Sie das Kontrollkästchen **k8s** aus, um die mit Kubernetes verbundenen Protokolle anzuzeigen.
1. Klicken Sie auf **Alle Quellen** und wählen Sie den Namen des Hosts (Workerknoten) aus, für den Sie die Protokolle überprüfen möchten. Dies funktioniert gut, wenn Sie mehrere Workerknoten in Ihrem Cluster haben.
1. Um Container- oder Dateiprotokolle zu überprüfen, klicken Sie auf **Alle Apps** und wählen Sie das/die Kontrollkästchen aus, für die Protokolle angezeigt werden sollen.

### Ansicht erstellen

Ansichten sind gespeicherte Direktaufrufe für eine bestimmte Gruppe von Filtern und Suchabfragen.

Sobald Sie Protokolle durchsuchen oder filtern, sollte in der oberen Leiste die Option **Nicht gespeicherte Ansicht** angezeigt werden. Gehen Sie wie folgt vor, um diese als Ansicht zu speichern:
1. Klicken Sie auf **Alle Apps** und wählen Sie das Kontrollkästchen neben **app-log-analysis** aus.
1. Klicken Sie auf **Nicht gespeicherte Ansicht**. > Klicken Sie auf **Als neue Ansicht/neuen Alert speichern** und geben Sie der Ansicht den Namen **app-log-analysis-view**. Lassen Sie die **Kategorie** leer.
1. Klicken Sie auf **Ansicht speichern**. Die neue Ansicht sollte im linken Fensterbereich mit den Protokollen für die App angezeigt werden.

### Protokolle mit Diagrammen und Aufgliederungen visualisieren

In diesem Abschnitt erstellen Sie ein Board und fügen dann ein Diagramm mit einer Aufgliederung hinzu, um die Daten der App-Ebene darzustellen. Ein Board ist eine Sammlung von Diagrammen und Aufgliederungen.

1. Klicken Sie im linken Teilfenster auf das Symbol **Board** (über dem Symbol 'Einstellungen'). > Klicken Sie auf **Neues Board**.
1. Klicken Sie in der oberen Leiste auf **Bearbeiten** und geben Sie **app-log-analysis-board** als Namen an. Klicken Sie auf **Speichern**.
1. Klicken Sie auf **Diagramm hinzufügen**:
   - Geben Sie **app** als Feld in das erste Eingabefeld ein und drücken Sie die Eingabetaste.
   - Wählen Sie **app-log-analysis** als Feldwert aus.
   - Klicken Sie auf **Diagramm hinzufügen**.
1. Wählen Sie **Anzahl** als Metrik aus, um die Anzahl der Zeilen in jedem Intervall während der letzten 24 Stunden anzuzeigen.
1. Klicken Sie auf den Pfeil unter dem Diagramm, um eine Aufgliederung hinzuzufügen:
   - Wählen Sie **Histogramm** als Aufgliederungstyp aus.
   - Wählen Sie **Ebene** als Feldtyp aus.
   - Klicken Sie auf **Aufgliederung hinzufügen**, um eine Aufgliederung mit allen Ebenen anzuzeigen, die Sie für die App protokolliert haben.

## {{site.data.keyword.mon_full_notm}} hinzufügen und den Cluster überwachen
{: #monitor_cluster_sysdig}

Im Folgenden wird {{site.data.keyword.mon_full_notm}} zur Anwendung hinzugefügt. Der Service überprüft regelmäßig die Verfügbarkeit und Antwortzeit der App.

1. Navigieren Sie zur Seite für die [Beobachtbarkeit](https://{DomainName}/observe/) und klicken Sie unter **Überwachung** auf **Instanz erstellen**.
1. Geben Sie einen eindeutigen **Servicenamen** an.
1. Wählen Sie eine Region/einen Standort und eine Ressourcengruppe aus.
1. Wählen Sie **Gestaffelte Preisstufe** als Plan aus und klicken Sie auf **Erstellen**.
1. Klicken Sie neben dem Service, den Sie zuvor erstellt haben, auf **Protokollressourcen bearbeiten** und wählen Sie **Kubernetes** aus.
1. Kopieren Sie den Befehl unter **Sysdig-Agent auf dem Cluster installieren** und führen Sie ihn auf einem Terminal aus, auf dem Sie die Umgebungsvariable `KUBECONFIG` für die Implementierung des Sysdig-Agenten im Cluster angegeben haben. Warten Sie, bis die Implementierung abgeschlossen ist.

### {{site.data.keyword.mon_short}} konfigurieren

Gehen Sie wie folgt vor, um Sysdig für die Überwachung des Status und der Leistung des Clusters zu konfigurieren:
1. Klicken Sie auf **Sysdig anzeigen**. Die Benutzerschnittstelle für die Sysdig-Überwachung sollte daraufhin angezeigt werden. Klicken Sie auf der Begrüßungsseite auf **Weiter**.
1. Wählen Sie **Kubernetes** als Installationsmethode im Abschnitt zur Konfiguration der Umgebung aus.
1. Klicken Sie auf **Weiter mit dem nächsten Schritt** neben der Nachricht über die erfolgreiche Agentenkonfiguration und klicken Sie auf der nächsten Seite auf **Los geht's**.
1. Klicken Sie auf **Weiter** und anschließend auf **Vollständiges Onboarding**, um die Registerkarte `Durchsuchen` der Sysdig-Benutzerschnittstelle anzuzeigen.

### Cluster überwachen

Gehen Sie wie folgt vor, um den Status und die Leistung Ihrer App und Ihres Clusters zu überprüfen:
1. Erstellen Sie in der Anwendung, die unter `http://worker-ip-address:portnumber` ausgeführt wird, mehrere Protokolleinträge.
1. Erweitern Sie **mycluster** im linken Teilfenster. > Erweitern Sie den **Standard**-Namensbereich. > Klicken Sie auf **app-log-analysis-deployment**, um die Anzahl der Anforderungen, die Antwortzeit usw. im Sysdig-Überwachungsassistenten anzuzeigen.
1. Um die HTTP-Anforderung/Antwort-Codes zu überprüfen, klicken Sie auf den Pfeil neben **Kubernetes-Pod-Status** in der oberen Leiste und wählen Sie unter **Anwendungen** die Option **HTTP** aus. Ändern Sie das Intervall in der unteren Leiste der Sysdig-Benutzerschnittstelle in **10 M**.
1. Gehen Sie wie folgt vor, um die Latenzzeit der Anwendung zu überwachen:
   - Wählen Sie auf der Registerkarte 'Erkunden' die Option **Implementierungen und Pods** aus.
   - Klicken Sie auf den Pfeil neben `HTTP` und wählen Sie dann 'Metriken > Netz' aus.
   - Wählen Sie **net.http.request.time** aus.
   - Wählen Sie für die Zeit die Option **Summe** und für die Gruppe die Option **Durchschnitt** aus.
   - Klicken Sie auf **Weitere Optionen** und dann auf das Symbol **Topologie**.
   - Klicken Sie auf **Fertig** und doppelklicken Sie auf das Feld, um die Ansicht zu erweitern.
1. Gehen Sie wie folgt vor, um den Kubernetes-Namensbereich zu überwachen, in dem die Anwendung ausgeführt wird:
   - Wählen Sie auf der Registerkarte 'Erkunden' die Option **Implementierungen und Pods** aus.
   - Klicken Sie auf den Pfeil neben `net.http.request.time`.
   - Wählen Sie 'Standarddashboards > Kubernetes' aus.
   - Wählen Sie 'Kubernetes-Status > Übersicht zu Kubernetes-Status' aus.

### Angepasstes Dashboard erstellen

Zusammen mit den vordefinierten Dashboards können Sie Ihr eigenes angepasstes Dashboard erstellen, um die nützlichsten/relevantesten Ansichten und Metriken für die Container anzuzeigen, auf dem Ihre App an einer einzigen Position ausgeführt wird. Jedes Dashboard besteht aus einer Reihe von Anzeigen, die so konfiguriert sind, dass bestimmte Daten in einer Reihe unterschiedlicher Formate angezeigt werden.

Gehen Sie wie folgt vor, um ein Dashboard zu erstellen:
1. Klicken Sie im linken Teilfenster auf **Dashboards**. > Klicken Sie dann auf **Dashboard hinzufügen**.
1. Klicken Sie auf **Leeres Dashboard**. > Geben Sie dem Dashboard den Namen **Übersicht zu Containeranforderungen**. > Klicken Sie auf **Dashboard erstellen**.
1. Wählen Sie **Oberste Liste** als neue Anzeige aus und geben Sie der Anzeige den Namen **Anforderungszeit pro Container**.
   - Geben Sie unter **Metriken** den Wert **net.http.request.time** ein.
   - Bereich: Klicken Sie auf **Dashboardbereich überschreiben**. > Wählen Sie **container.image** aus. > Wählen Sie **is** aus. > Wählen Sie das _Anwendungsimage_ aus.
   - Nehmen Sie eine Segmentierung nach **Container-ID** vor. Daraufhin sollten die Netzanforderungszeiten für die einzelnen Container angezeigt werden.
   - Klicken Sie auf **Speichern**.
1. Um eine neue Anzeige hinzuzufügen, klicken Sie auf das **Plus**-Symbol und wählen Sie **Zahl(#)** als Anzeigetyp aus.
   - Geben Sie unter **Metriken** den Wert **net.http.request.count** ein. > Ändern Sie die Zeitaggregation von **Durchschnitt** in **Summe**.
   - Bereich: Klicken Sie auf **Dashboardbereich überschreiben**. > Wählen Sie **container.image** aus. > Wählen Sie **ist** aus. > Wählen Sie das _Anwendungsimage_ aus.
   - Stellen Sie einen Vergleich mit dem Wert vor **1 Stunde** an. Es sollten die Netzanforderungszeiten für die einzelnen Container angezeigt werden.
   - Klicken Sie auf **Speichern**.
1. Gehen Sie wie folgt vor, um den Bereich dieses Dashboards zu bearbeiten:
   - Klicken Sie in der Titelanzeige auf **Bereich bearbeiten**.
   - Wählen Sie **Kubernetes.cluster.name** in der Dropdown-Liste aus (oder geben Sie den Wert ein).
   - Lassen Sie den Anzeigenamen leer und wählen Sie **ist** aus.
   - Wählen Sie **mycluster** als Wert aus und klicken Sie auf **Speichern**.

## Ressourcen entfernen
{: #remove_resource}

- Entfernen Sie die LogDNA-und Sysdig-Instanzen von der Seite für die [Beobachtbarkeit](https://{DomainName}/observe).
- Löschen Sie den Cluster, einschließlich Workerknoten, App und Container. Diese Aktion kann nicht rückgängig gemacht werden.
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## Lernprogramm erweitern
{: #expand_tutorial}

- Verwenden Sie den [{{site.data.keyword.at_full}}-Service](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started), um zu verfolgen, wie Anwendungen mit IBM Cloud-Services interagieren.
- [Fügen Sie Alerts](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts) Ihrer Ansicht hinzu.
- [Exportieren Sie Protokolle](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export) in eine lokale Datei.

## Zugehöriger Inhalt
{:related}
- [Den von einem Kubernetes-Cluster verwendeten Einpflegeschlüssel zurücksetzen](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [Protokolle in IBM Cloud Object Storage archivieren](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [Alerts in Sysdig konfigurieren](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [Mit Benachrichtigungskanälen in der Sysdig-Benutzerschnittstelle arbeiten](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
