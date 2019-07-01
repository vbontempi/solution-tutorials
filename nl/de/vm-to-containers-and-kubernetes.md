---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-15"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# VM-basierte App in Kubernetes verschieben
{: #vm-to-containers-and-kubernetes}

Dieses Lernprogramm führt Sie durch den Prozess, durch den eine VM-basierte App mithilfe von {{site.data.keyword.containershort_notm}} auf einen Kubernetes-Cluster verschoben wird. [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) stellt leistungsfähige Tools durch eine Kombination von Docker- und Kubernetes-Technologien, intuitive Funktionalität für Benutzer sowie integrierte Sicherheits- und Isolationsfunktionalität zur Verfügung, um die Bereitstellung, den Betrieb, die Skalierung und die Überwachung containerisierter Apps in einem Cluster von Rechenhosts zu automatisieren.
{: shortdesc}

Die Lerneinheiten in diesem Lernprogramm enthalten Konzepte dazu, wie eine vorhandene App genommen, containerisiert und auf einem  Kubernetes-Cluster bereitgestellt wird. Zur Containerisierung Ihrer VM-basierten App können Sie unter den folgenden Optionen auswählen.

1. Geben Sie Komponenten einer umfangreichen, monolithischen App an, die in eigene Microservices aufgegliedert werden können. Sie können diese Microservices containerisieren und in einem Kubernetes-Cluster bereitstellen.
2. Containerisieren Sie die gesamte App und stellen Sie die App in einem Kubernetes-Cluster bereit.

Abhängig vom Typ der App, die Sie haben, können sich die Schritte für die Migration Ihrer App unterscheiden. In diesem Lernprogramm können Sie sich mit den allgemeinen Schritten vertraut machen, die Sie ausführen müssen, und Gesichtspunkte kennen lernen, die Sie vor der Migration Ihrer App berücksichtigen müssen.

## Lernziele
{: #objectives}

- Vorgehensweise bei der Angabe von Microservices in einer VM-basierten App und bei der Zuordnung von Komponenten zwischen VMs und Kubernetes kennen lernen
- Vorgehensweise zur Containerisierung einer VM-basierten App kennen lernen
- Vorgehensweise zur Bereitstellung des Containers auf einem Kubernetes-Cluster in {{site.data.keyword.containershort_notm}} kennen lernen
- Das Gelernte in die Praxis umsetzen und die App **JPetStore** im Cluster ausführen

## Verwendete Services
{: #products}

In diesem Lernprogramm werden die folgenden {{site.data.keyword.Bluemix_notm}}-Services verwendet:

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**Achtung:** Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um einen Kostenvoranschlag auf der Grundlage der von Ihnen projektierten Nutzung zu generieren.

## Architektur
{:#architecture}

### Traditionelle App-Architektur mit VMs

Das folgende Diagramm zeigt ein Beispiel für eine traditionelle App-Architektur, die auf virtuellen Maschinen (VMs) basiert.

<p style="text-align: center;">
![Architekturdiagramm](images/solution30/traditional_architecture.png)

</p>

1. Der Benutzer sendet eine Anforderung an den öffentlichen Endpunkt der Java-App. Der öffentliche Endpunkt wird durch einen Lastausgleichsservice dargestellt, der den eingehenden Netzverkehr zwischen den verfügbaren App-Serverinstanzen verteilt.
2. Die Lastausgleichsfunktion wählt eine der einwandfreien App-Serverinstanzen aus, die auf einer VM ausgeführt werden, und leitet die Anforderung weiter.
3. Der App-Server speichert App-Daten in einer MySQL-Datenbank, die auf einer VM ausgeführt wird. App-Serverinstanzen stellen die Java-App bereit und werden auf einer VM ausgeführt. App-Dateien, wie zum Beispiel der Code, Konfigurationsdateien und Abhängigkeiten der App, werden auf der VM gespeichert.

### Containerisierte Architektur

Das folgende Diagramm zeigt ein Beispiel für eine moderne Containerarchitektur, die in einem Kubernetes-Cluster ausgeführt wird.

<p style="text-align: center;">
![Architekturdiagramm](images/solution30/modern_architecture.png)
</p>

1. Der Benutzer sendet eine Anforderung an den öffentlichen Endpunkt der Java-App. Der öffentliche Endpunkt wird durch eine Ingress-Lastausgleichsfunktion für Anwendungen (ALB) dargestellt, die eingehenden Netzverkehr auf die App-Pods im Cluster verteilt. Die ALB enthält eine Sammlung von Regeln, die eingehenden Netzverkehr an eine öffentlich zugänglich gemachte App zulassen.
2. Die ALB leitet die Anforderung an einen der verfügbaren App-Pods im Cluster weiter. App-Pods werden auf Workerknoten ausgeführt, bei denen es sich um eine virtuelle oder eine physische Maschine handeln kann.
3. App-Pods speichern Daten auf persistenten Datenträgern. Persistente Datenträger können verwendet werden, um Daten zwischen App-Instanzen oder Workerknoten gemeinsam zu nutzen.
4. App-Pods speichern Daten in einem {{site.data.keyword.Bluemix_notm}}-Datenbankservice. Sie können eine eigene Datenbank im Kubernetes-Cluster ausführen, jedoch ist die Verwendung einer verwalteten Database as a Service-Datenbank (DBasS) in der Regel einfacher zu konfigurieren und stellt integrierte Backup- und Skalierungsfunktionen bereit. Eine Reihe verschiedener Typen von Datenbanken finden Sie im [IBM Cloud-Katalog](https://{DomainName}/catalog/?category=data).

### VMs, Container und Kubernetes

{{site.data.keyword.containershort_notm}} bietet die Funktionalität zur Ausführung von containerisierten Apps in Kubernetes-Clustern und stellt die folgenden Tools und Funktionen zur Verfügung:

- Intuitive Benutzerfunktionalität und leistungsfähige Tools
- Integrierte Sicherheits- und Isolationsfunktionen, die eine rasche Bereitstellung sicherer Anwendungen ermöglichen
- Cloud-Services, die kognitive Funktionen aus IBM® Watson™ einbeziehen
- Möglichkeit zur Verwaltung dedizierter Clusterressourcen für statusunabhängige Anwendungen und statusabhängige Workloads

#### Virtuelle Maschinen und Container im Vergleich

Virtuelle Maschinen (**VMs**) werden als traditionelle Apps auf nativer Hardware ausgeführt. Eine einzelne App verwendet in der Regel nicht den vollen Umfang der Ressourcen eines einzelnen Rechenhosts. Die meisten Organisationen versuchen, mehrere Apps auf einem Rechenhost auszuführen, um keine Ressourcen zu vergeuden. Sie könnten mehrere Kopien derselben App ausführen, jedoch können Sie zur Bereitstellung von Isolation VMs verwenden, um mehrere App-Instanzen (VMs) auf derselben Hardware auszuführen. Diese VMs verfügen über vollständige Betriebssystemstacks, was sie relativ groß und ineffizient macht, da die Ressourcen sowohl während der Ausführung als auch auf der Platte doppelt anfallen.

**Container** sind eine Standardmethode zur Paketierung von Apps und aller zugehörigen Abhängigkeiten, sodass sich die Apps nahtlos zwischen Umgebungen verschieben lassen. Im Gegensatz zu virtuellen Maschinen schließen Container das Betriebssystem nicht ein. Nur der App-Code, die Laufzeit, Systemtools, Bibliotheken und Einstellungen werden in Container verpackt. Container sind schlanker, portierbarer und effizienter als virtuelle Maschinen.

Darüber hinaus ermöglichen Container eine gemeinsame Nutzung des Hostbetriebssystems. Dadurch verringert sich die Duplizierung von Ressourcen, während die Isolationsfunktion erhalten bleibt. Container ermöglichen außerdem den Ausschluss nicht benötigter Dateien, wie Systembibliotheken und Binärdateien, um Speicherplatz einzusparen und Ihre potenzielle Angriffsfläche zu verkleinern. Weitere Informationen zu virtuellen Maschinen und Containern finden Sie [hier](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html).

#### Kubernetes-Orchestrierung

[Kubernetes](http://kubernetes.io/) ist ein Container-Orchestrator für die Verwaltung des Lebenszyklus von containerisierten Apps in einem Cluster mit Workerknoten. Ihre Apps benötigen für die Ausführung möglicherweise viele weitere Ressourcen, wie zum Beispiel Datenträger, Netze und geheime Schlüssel, die die Verbindung zu anderen Cloud-Services unterstützen, sowie sichere Schlüssel. Kubernetes hilft Ihnen, diese Ressourcen für Ihre App hinzuzufügen. Das Schlüsselparadigma von Kubernetes ist das deklarative Modell. Der Benutzer gibt den gewünschten Status an und Kubernetes versucht, den beschriebenen Status zu erfüllen und anschießend zu erhalten.

Dieser [autodidaktische Workshop](https://github.com/IBM/kube101/blob/master/workshop/README.md) kann Ihnen bei Ihren ersten praktischen Erfahrungen mit Kubernetes helfen. Darüber hinaus können Sie sich auf der Seite über [Konzepte](https://kubernetes.io/docs/concepts/) in der Kubernetes-Dokumentation eingehender mit den Konzepten von Kubernetes vertraut machen.

### Was IBM für Sie tut

Die Verwendung von Kubernetes-Clustern mit {{site.data.keyword.containerlong_notm}} bietet Ihnen die folgenden Vorteile:

- Mehrere Rechenzentren, in denen Sie Ihre Cluster bereitstellen können
- Unterstützung von Optionen für Netze mit Ingress- und Lastausgleichsfunktionen
- Unterstützung für dynamische persistente Datenträger
- Hoch verfügbare, von IBM verwaltete Kubernetes-Master

## Cluster dimensionieren
{: #sizing_clusters}

Beim Entwurf Ihrer Clusterarchitektur wollen Sie die Aspekte der Kosten einerseits und die Aspekte der Verfügbarkeit, Zuverlässigkeit, Komplexität und Wiederherstellung andererseits gegeneinander abwägen. Kubernetes-Cluster in {{site.data.keyword.containerlong_notm}} stellen architekturbezogene Optionen entsprechend den Anforderungen Ihrer Apps bereit. Mit ein wenig Planung können Sie Ihre Cloudressource optimal nutzen, ohne die Architektur überzustrapazieren oder zu hohe Kosten aufzuwenden. Selbst wenn Sie die Situation über- oder unterschätzen, können Sie problemlos eine Skalierung Ihres Clusters nach oben oder unten durchführen, indem Sie entweder zusätzliche Workerknoten oder größere Workerknoten verwenden.

Berücksichtigen Sie für die Ausführung einer App auf Produktionsebene in der Cloud unter Verwendung von Kubernetes die folgenden Aspekte:

1. Erwarten Sie Datenverkehr von einem bestimmten geografischen Standort? Wenn dies der Fall ist, wählen Sie den Standort (Position) aus, der sich Ihnen physisch am nächsten befindet, um die beste Leistung zu erzielen.
2. Wie viele Replikate Ihres Clusters sollen für hohe Verfügbarkeit bereitgestellt werden? Ein guter Ausgangspunkt könnten drei Cluster sein: einer für die Entwicklung, einen für Tests und einen für die Produktion. Lesen Sie die Informationen im Lösungshandbuch [Bewährte Verfahren für die Organisation von Benutzern, Teams und Anwendungen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments) zur Erstellung mehrerer Umgebungen.
3. Welche [Hardware](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes) benötigen Sie für die Workerknoten? Virtuelle Maschinen oder Bare-Metal-Einheiten?
4. Wie viele Workerknoten benötigen Sie? Dies hängt im Wesentlichen von der Skalierung (Scale) Ihrer App ab: je mehr Knoten Sie haben, desto ausfallsicherer ist Ihre App.
5. Wie viele Replikate sollten Sie für hohe Verfügbarkeit haben? Stellen Sie Replikatcluster an mehreren Standorten (Positionen) bereit, um Ihre App verfügbarer zu machen und sie gegen Ausfallzeiten aufgrund von Standortfehlern zu schützen.
6. Welchen minimalen Satz an Ressourcen benötigt Ihre App für den Start? Sie können Ihre App testen, um ihren Speicherbedarf und den CPU-Bedarf für die Ausführung zu ermitteln. Ihre Workerknoten sollten über ausreichend Ressourcen verfügen, um die App bereitstellen und starten zu können. Stellen Sie sicher, dass Sie anschließend Ressourcenquoten (Größenbeschränkungen) im Rahmen der Podspezifikationen festlegen. Diese Einstellung wird von Kubernetes zur Auswahl (oder Planung) eines Workerknotens verwendet, der über ausreichend Kapazität zur Unterstützung der Anforderung verfügt. Schätzen Sie die Anzahl der Pods, die auf dem Workerknoten ausgeführt werden, und den Ressourcenbedarf für diese Pods. Ihr Workerknoten muss mindestens so groß sein, dass er einen Pod für die App unterstützen kann.
7. Wann sollte die Anzahl der Workerknoten erhöht werden? Sie können die Clusternutzung überwachen und die Anzahl der Knoten bei Bedarf erhöhen. In diesem Lernprogramm wird erläutert, wie [Protokolle analysiert und der Zustand von Kubernetes-Anwendungen überwacht werden](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana).
8. Benötigen Sie redundanten, zuverlässigen Speicher? Wenn dies der Fall ist, erstellen Sie einen Persistent Volume Claim (Anforderung für persistenten Datenträger) für NFS-Speicher und binden Sie einen IBM Cloud-Datenbankservice an Ihren Pod.

Um das oben Ausgeführte etwas konkreter zu fassen, nehmen Sie einmal an, dass Sie eine Webanwendung der Produktionsumgebung in der Cloud ausführen wollen und ein mittleres bis hohes Datenverkehrsaufkommen erwarten. Ermitteln Sie die Ressourcen, die Sie zu diesem Zweck benötigen würden:

1. Richten Sie drei Cluster ein: einen für die Entwicklung, einen für das Testen und einen für die Produktion.
2. Die Cluster für die Entwicklung und das Testen können zu Anfang mit der minimalen RAM- und CPU-Option (z. B. 2 CPUs, 4 GB RAM und einen Workerknoten für jeden Cluster) konfiguriert werden.
3. Für den Produktionscluster können Sie aus Gründen der Leistung, hohen Verfügbarkeit und Ausfallsicherheit mehr Ressourcen nutzen. Sie könnten eine dedizierte oder sogar eine Bare-Metal-Option wählen, um mindestens 4 CPUs, 16 GB RAM und zwei Workerknoten zur Verfügung zu haben.

## Zu verwendende Datenbankoption festlegen
{: #database_options}

Mit Kubernetes haben Sie zwei Optionen zur Handhabung von Datenbanken:

1. Sie können Ihre Datenbank in einem Kubernetes-Cluster ausführen. Dazu müssten Sie einen Microservice zur Ausführung der Datenbank erstellen. Bei Verwendung des Beispiels für die MySQL-Datenbank müssten Sie wie folgt vorgehen:
   - Erstellen Sie eine MySQL-Dockerfile. Ein Beispiel finden Sie hier unter [MySQL-Dockerfile](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile).
   - Sie müssten geheime Schlüssel zum Speichern der Datenbankberechtigungsnachweise verwenden. Ein Beispiel dazu finden Sie [hier](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret).
   - Sie müssten eine Datei 'deployment.yaml' mit der Konfiguration Ihrer Datenbank in Kubernetes bereitstellen. Ein Beispiel dazu finden Sie [hier](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml).
2. Die zweite Option wäre die Verwendung der verwalteten Database as a Service-Option (DBasS). Diese Option ist zumeist einfacher zu konfigurieren und stellt integrierte Backup- und Skalierungsfunktionen bereit. Eine Reihe verschiedener Typen von Datenbanken finden Sie im [IBM Cloud-Katalog](https://{DomainName}/catalog/?category=data). Zur Verwendung dieser Option müssten Sie wie folgt vorgehen:
   - Erstellen Sie einen verwalteten DBasS-Service aus dem [IBM Cloud-Katalog](https://{DomainName}/catalog/?category=data).
   - Speichern Sie die Datenbankberechtigungsnachweise in einem geheimen Schlüssel. Weitere Einzelheiten zu geheimen Schlüsseln erfahren Sie im Abschnitt "Berechtigungsnachweise in geheimen Kubernetes-Schlüsseln speichern".
   - Verwenden Sie den DBasS-Service in Ihrer Anwendung.

## Position zum Speichern von Anwendungsdateien festlegen
{: #decide_where_to_store_data}

{{site.data.keyword.containershort_notm}} stellt verschiedene Optionen zur podübergreifenden Speicherung und gemeinsamen Nutzung von Daten bereit. Nicht alle Speicheroptionen bieten die gleiche Ebene von Persistenz und Verfügbarkeit bei Standortausfällen.
{: shortdesc}

### Nicht persistenter Datenspeicher

Container und Pods sind designbedingt von kurzer Lebensdauer und können unerwartet ausfallen. Sie können Daten im lokalen Dateisystem eines Containers speichern. Daten in einem Container können nicht mit anderen Containern oder Pods gemeinsam genutzt werden und gehen verloren, wenn der Container ausfällt oder entfernt wird.

### Erstellung des persistenten Datenspeichers für Apps kennen lernen

Sie können App-Daten und Containerdaten in [NFS-Dateispeicher](https://www.ibm.com/cloud/file-storage/details) oder in [Blockspeicher](https://www.ibm.com/cloud/block-storage) persistent speichern, indem Sie native persistente Kubernetes-Datenträger (Persistent Volumes) verwenden.
{: shortdesc}

Zur Bereitstellung von NFS-Dateispeicher oder Blockspeicher müssen Sie Speicher für Ihren Pod anfordern, indem Sie einen Persistent Volume Claim (PVC, Anforderung für persistenten Datenträger) erstellen. In Ihrem PVC können Sie unter vordefinierten Speicherklassen auswählen, die den Speichertyp, die Speichergröße in Gigabyte, die E/A-Operationen pro Sekunde (IOPS), die Datenaufbewahrungsrichtlinie sowie die Lese- und Schreibberechtigungen für Ihren Speicher definieren. Ein PVC stellt einen persistenten Datenträger (PV - Persistent Volume) dynamisch bereit, der eine reale Speichereinheit in {{site.data.keyword.Bluemix_notm}} darstellt. Sie können den PVC an Ihren Pod anhängen, um Lese- und Schreiboperationen auf dem PV auszuführen. Daten, die in PVs gespeichert werden, sind verfügbar, auch wenn der Container ausfällt oder der Pod neu geplant wird. Der NFS-Dateispeicher und der Blockspeicher, der den PV unterstützt, wird von IBM in einem Cluster angelegt, um Hochverfügbarkeit für Ihre Daten sicherzustellen.

Informationen zur Erstellung eines PVC finden Sie in den Schritten, die in der [Dokumentation zu {{site.data.keyword.containershort_notm}}-Speicher](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage) beschrieben werden.

### Verschiebung vorhandener Daten in persistenten Speicher kennen lernen

Zum Kopieren von Daten von Ihrer lokalen Maschine auf Ihren persistenten Speicher müssen Sie den PVC an einen Pod anhängen. Anschließend können Sie Daten von Ihrer lokalen Maschine auf den persistenten Datenträger in Ihrem Pod kopieren.
{: shortdesc}

1. Zum Kopieren von Daten müssen Sie zunächst eine Konfiguration erstellen, die ungefähr wie folgt aussieht:

   ```bash
   kind: Pod
   apiVersion: v1
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
          claimName: mypvc
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/mnt/data"
             name: task-pv-storage
   ```
   {: codeblock}

2. Anschließend würden Sie einen Befehl wie den folgenden verwenden, um Daten von Ihrer lokalen Maschine in den Pod zu kopieren:
   ```
    kubectl cp <lokaler_dateipfad>/<dateiname> <namensbereich>/<pod>:<pod-dateipfad>
   ```
   {: pre}

3. Kopieren Sie Daten aus einem Pod in Ihrem Cluster auf Ihre lokale Maschine:
   ```
   kubectl cp <namensbereich>/<pod>:<pod-dateipfad>/<dateiname> <lokaler_dateipfad>/<dateiname>
   ```
   {: pre}


### Backups für persistenten Speicher einrichten

Gemeinsam genutzte Dateiressourcen und Blockspeicher werden an derselben Position wie Ihr Cluster bereitgestellt. Der Speicher selbst wird von IBM auf Cluster-Servern gehostet, um hohe Verfügbarkeit bereitzustellen. Allerdings werden gemeinsam genutzte Dateiressourcen und Blockspeicher nicht automatisch durch Backups gesichert und können nicht mehr erreichbar sein, wenn ein gesamter Standort ausfällt. Wenn Sie Ihre Daten gegen Verlust oder Beschädigung schützen wollen, können Sie regelmäßige Backups einrichten, durch die Sie Ihre Daten bei Bedarf wiederherstellen können.

Weitere Informationen finden Sie unter den Optionen für die [Sicherung und Wiederherstellung](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) für NFS-Dateispeicher und Blockspeicher.

## Code vorbereiten
{: #prepare_code}

### 12-Faktor-Prinzipien anwenden

Die [12-Faktor-App](https://12factor.net/) ist eine Methodik zum Erstellen von cloudnativen Apps. Wenn Sie eine App containerisieren, diese App in die Cloud verschieben und sie mit Kubernetes orchestrieren wollen, ist es wichtig, einige dieser Prinzipien zu verstehen und anzuwenden. Einige dieser Prinzipien sind in {{site.data.keyword.Bluemix_notm}} erforderlich.
{: shortdesc}

Zu den erforderlichen wichtigen Prinzipien gehören zum Beispiel die folgenden:

- **Codebasis** - Sämtliche Quellcode- und Konfigurationsdateien werden in einem Versionssteuerungssystem verfolgt (z. B. in einem Git-Repository). Dies ist erforderlich, wenn eine DevOps-Pipeline für die Entwicklung genutzt wird.
- **Build, Freigabe, Ausführung** - Die 12-Faktor-App verwendet eine strikte Trennung der Build-, Freigabe- und Ausführungsphase. Dies kann durch eine integrierte DevOps-Delivery Pipeline automatisiert werden, sodass die App erstellt (Build) und getestet wird, bevor sie im Cluster bereitgestellt wird. Im Lernprogramm zur [Kontinuierlichen Bereitstellung in Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) können Sie lernen, wie eine kontinuierliche Integration und eine Delivery Pipeline eingerichtet werden. Es erläutert die Einrichtung der Quellcodeverwaltung sowie der Build-, Test- und Bereitstellungsphasen und zeigt, wie Integrationen wie zum Beispiel Sicherheitsscanner, Benachrichtigungen und Analysen hinzugefügt werden.
- **Konfiguration** - Sämtliche Konfigurationsinformationen werden in Umgebungsvariablen gespeichert. Es werden keine Serviceberechtigungsnachweise im App-Code fest codiert. Zum Speichern von Berechtigungsnachweisen können Sie geheime Kubernetes-Schlüssel verwenden. Berechtigungsnachweise werden nachfolgend eingehender behandelt.

### Berechtigungsnachweise in geheimen Kubernetes-Schlüsseln speichern
{: secrets}

Es ist in keinem Fall eine gute Idee, Berechtigungsnachweise im App-Code zu speichern. Vielmehr stellt Kubernetes so genannte **["geheime Schlüssel" (Secrets)](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)** zum Speichern sensibler Informationen wie Kennwörter, OAuth-Tokens oder SSH-Schlüssel bereit. Geheime Kubernetes-Schlüssel werden standardmäßig verschlüsselt, was geheime Schlüssel zu einer sichereren und flexibleren Option zum Speichern sensibler Daten macht, als solche Daten buchstäblich in einer `Poddefinition` oder einem Docker-Image zu speichern.

Eine Möglichkeit zur Verwendung von geheimen Schlüsseln in Kubernetes ist zum Beispiel die folgende:

1. Erstellen Sie eine Datei und speichern Sie die Serviceberechtigungsnachweise in dieser Datei.
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api-schlüssel>
   }
   ```
   {: codeblock}

2. Erstellen Sie einen geheimen Kubernetes-Schlüssel, indem Sie den folgenden Befehl ausführen, und prüfen Sie anschließend, ob der geheime Schlüssel erstellt wurde, indem Sie den Befehl `kubectl get secrets` verwenden:

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## App containerisieren
{: #build_docker_images}

Zum Containerisieren Ihrer App müssen Sie ein Docker-Image erstellen.
{: shortdesc}

Ein Image wird aus einer [Dockerfile](https://docs.docker.com/engine/reference/builder/#usage) erstellt, wobei es sich um eine Datei handelt, die Anweisungen und Befehle zum Erstellen des Image enthält. Eine Dockerfile kann auf Buildartefakte in den Anweisungen verweisen, die separat gespeichert sind, wie zum Beispiel eine App, die Konfiguration der App und die Abhängigkeiten der App. 

Zum Erstellen einer eigenen Dockerfile für Ihre vorhandene App können Sie die folgenden Befehle verwenden:

- FROM - Wählt ein übergeordnetes Image aus, um die Containerlaufzeit zu definieren.
- ADD/COPY - Kopiert den Inhalt eines Verzeichnisses in den Container.
- WORKDIR - Legt das Arbeitsverzeichnis im Container fest.
- RUN - Installiert Softwarepakete, die von der App während der Ausführung benötigt werden.
- EXPOSE - Macht einen Port außerhalb des Containers verfügbar.
- ENV NAME - Definiert Umgebungsvariablen.
- CMD - Definiert Befehle, die beim Start des Containers ausgeführt werden.

Images werden in der Regel in einer Registry gespeichert, auf die entweder öffentlich zugegriffen werden kann (öffentliche Registry) oder die mit eingeschränktem Zugriff für eine kleine Gruppe von Benutzern eingerichtet wird (private Registry). Öffentliche Registrys, wie Docker Hub, können zum Einstieg in die Arbeit mit Docker und Kubernetes verwendet werden, um eine erste containerisierte App in einem Cluster zu erstellen. Bei Apps für ein Unternehmen sollten Sie jedoch eine private Registry, zum Beispiel die in {{site.data.keyword.registrylong_notm}} bereitgestellte Registry, verwenden, um Ihre Images gegen Nutzung und Änderung durch nicht berechtigte Benutzer zu schützen.

Gehen Sie wie folgt vor, um eine App zu containerisieren und in {{site.data.keyword.registrylong_notm}} zu speichern:

1. Sie müssten eine Dockerfile erstellen. Das folgende Beispiel zeigt eine Dockerfile.
   ```
   # Build: Datei JPetStore.war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # WebSphere Liberty-Basisimage aus Docker-Store verwenden
   FROM websphere-liberty:latest

   # WAR-Datei aus Build-Stage und server.xml in Image kopieren
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. Nach der Erstellung einer Dockerfile müssen Sie das Container-Image erstellen und durch eine Push-Operation in {{site.data.keyword.registrylong_notm}} übertragen. Sie können einen Container mit einem Befehl wie dem folgenden erstellen:
   ```
   ibmcloud cr build -t <imagename> <verzeichnis_der_dockerfile>
   ```
   {: pre}

## App auf einem Kubernetes-Cluster bereitstellen
{: #deploy_to_kubernetes}

Wenn ein Container-Image erstellt und in die Cloud übertragen wurde, müssen Sie es als Nächstes in Ihrem Kubernetes-Cluster bereitstellen. Zu diesem Zweck müssen Sie eine Datei 'deployment.yaml' erstellen.
{: shortdesc}

### Erstellung einer Kubernetes-Bereitstellungsdatei 'deployment.yaml'

Zum Erstellen von Kubernetes-Dateien 'deployment.yaml' müssen Sie wie folgt vorgehen:

1. Erstellen Sie eine Datei 'deployment.yaml'. Ein Beispiel für eine solche Datei finden Sie unter [Datei 'deployment.yaml'](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml).

2. In Ihrer Datei 'deployment.yaml' können Sie [Ressourcenquoten](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) (Ressourcengrößenbeschränkungen) für Container definieren, um anzugeben, wie viel CPU- und Speicherkapazität jeder Container für einen ordnungsgemäßen Start benötigt. Wenn für Container Ressourcenquoten angegeben wurden, kann der Kubernetes-Scheduler bessere Entscheidungen darüber treffen, auf welchen Workerknoten Ihre Pods platziert werden sollen.

3. Als Nächstes können Sie die folgenden Befehle verwenden, um die Bereitstellung zu erstellen und die erstellten Services anzuzeigen:

   ```
   kubectl create -f <dateipfad/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## Zusammenfassung
{: #summary}

In diesem Lernprogramm haben Sie Folgendes kennen gelernt:

- Unterschiede zwischen VMs, Container und Kubernetes
- Vorgehensweise zur Definition von Clustern für verschiedene Umgebungstypen (Entwicklung, Testen und Produktion)
- Vorgehensweise zur Handhabung des Datenspeichers und Wichtigkeit von persistentem Datenspeicher
- Anwendung der 12-Faktor-Prinzipien auf Ihre App und Verwendung geheimer Schlüssel für Berechtigungsnachweise in Kubernetes
- Erstellung von Docker-Images und ihre Push-Übertragung in {{site.data.keyword.registrylong_notm}}
- Erstellung von Kubernetes-Bereitstellungsdateien und Bereitstellung des Docker-Image in Kubernetes

## Alles Gelernte in die Praxis umsetzen und App 'JPetStore' im Cluster ausführen
{: #runthejpetstore}

Um alles, was Sie gelernt haben, in die Praxis umzusetzen, befolgen Sie die [Demo](https://github.com/ibm-cloud/ModernizeDemo/), um die App **JPetStore** in Ihrem Cluster auszuführen und die erlernten Konzepte anzuwenden. Die App 'JPetStore' besitzt eine etwas erweiterte Funktionalität, sodass Sie eine App in Kubernetes durch IBM Watson-Services, die als separater Microservice ausgeführt werden, erweitern können.

## Zugehörige Inhalte
{: #related}

- [Einführung](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/) in Kubernetes und {{site.data.keyword.containershort_notm}}.
- {{site.data.keyword.containershort_notm}}-Laboratorien auf [GitHub](https://github.com/IBM/container-service-getting-started-wt).
- Kubernetes - [Hauptdokumentation](http://kubernetes.io/).
- [Persistenter Speicher](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) in {{site.data.keyword.containershort_notm}}.
- [Lösungshandbuch mit bewährten Verfahren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications) zur Organisation von Benutzern, Teams und Apps.
- [Protokolle analysieren und den Anwendungsstatus mit LogDNA und Sysdig überwachen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- [Kontinuierliche Integration und Delivery Pipeline](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) für containerisierte, in Kubernetes ausgeführte Apps einrichten.
- Produktionscluster [an mehreren Standorten](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp) bereitstellen.
- [Mehrere Cluster an mehreren Standorten](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations) für hohe Verfügbarkeit verwenden.
