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

# Skalierbare Webanwendung unter Kubernetes
{: #scalable-webapp-kubernetes}

In diesem Lernprogramm erfahren Sie, wie Sie ein Gerüst für eine Webanwendung erstellen, die Anwendung lokal in einem Container ausführen und anschließend in einem Kubernetes-Cluster bereitstellen, der mit [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster) erstellt wurde. Darüber hinaus erfahren Sie, wie Sie eine angepasste Domäne binden, den Status der Umgebung überwachen und die Anwendung skalieren.
{:shortdesc}

Container sind eine Standardmethode zum Paketieren von Apps mit allen zugehörigen Abhängigkeiten, damit die Apps problemlos zwischen Umgebungen verschoben werden können. Im Gegensatz zu virtuellen Maschinen, wird bei Containern das Betriebssystem nicht in das Paket einbezogen. Nur App-Code, Laufzeit, Systemtools, Bibliotheken und Einstellungen werden in Container gepackt. Container sind schlanker, leichter portierbar und effizienter als virtuelle Maschinen.

Für die beschleunigte Umsetzung von Entwicklungsprojekten ermöglicht die {{site.data.keyword.dev_cli_notm}}-CLI die zeiteffiziente Anwendungsentwicklung und -bereitstellung durch das Generieren von Vorlagenanwendungen, die sofort ausgeführt werden können oder als Ausgangspunkt für eigene angepasste Lösungen dienen können. Neben dem Generieren des Starteranwendungscodes, des Docker-Container-Image und der Cloud Foundry-Assets erstellen die von der Entwicklungs-CLI und der Webkonsole verwendeten Codegeneratoren Dateien, die die Bereitstellung in [Kubernetes](https://kubernetes.io/)-Umgebungen unterstützen. Die Vorlagen erstellen [Helm](https://github.com/kubernetes/helm)-Diagramme zur Beschreibung der ursprünglichen Konfiguration für die Bereitstellung der Anwendung in Kubernetes und können bei Bedarf ohne großen Aufwand zu komplexen Bereitstellungen mit mehreren Images erweitert werden.

## Ziele
{: #objectives}

* Gerüst einer Starteranwendung erstellen
* Die Anwendung im Kubernetes-Cluster bereitstellen
* Eine angepasste Domäne binden
* Protokolle und Status des Clusters überwachen
* Kubernetes-Pods skalieren

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur
{: #architecture}

<p style="text-align: center;">

  ![Architektur](images/solution2/Architecture.png)
</p>

1. Ein Entwickler generiert eine Starteranwendung mit {{site.data.keyword.dev_cli_notm}}.
1. Durch den Buildvorgang für die Anwendung wird ein Docker-Container-Image erstellt.
1. Das Image wird durch eine Push-Operation in einen Namensbereich in {{site.data.keyword.containershort_notm}} übertragen.
1. Die Anwendung wird in einem Kubernetes-Cluster bereitgestellt.
1. Benutzer greifen auf die Anwendung zu.

## Vorbereitende Schritte
{: #prereqs}

* [{{site.data.keyword.registrylong_notm}}-CLI und Registry-Namensbereich einrichten](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [{{site.data.keyword.dev_cli_notm}} installieren](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - Script zum Installieren von Docker, Kubectl, Helm, IBM Cloud-ClI und erforderlicher Plug-ins
* [Grundlegende Informationen zu Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## Kubernetes-Cluster erstellen
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} bietet leistungsstarke Tools durch die Kombination von Docker-und Kubernetes-Technologien, ein intuitives Benutzererlebnis sowie integrierte Sicherheit und Isolation, um die Bereitstellung, den Betrieb, die Skalierung und die Überwachung containerisierter Apps in einem Cluster von Rechenhosts zu automatisieren.

Der größte Teil dieses Lernprogramms kann mit einem **kostenlosen** Cluster durchgeführt werden. Für die beiden optionalen Abschnitte über Kubernetes Ingress und eine angepasste Domäne ist ein **kostenpflichtiger** Cluster des Typs **Standard** erforderlich.

1. Erstellen Sie einen Kubernetes-Cluster aus dem [{{site.data.keyword.Bluemix}}-Katalog](https://{DomainName}/containers-kubernetes/launch).

   Überprüfen Sie zur Verbesserung der Benutzerfreundlichkeit Details wie CPU-Anzahl, Hauptspeicher und die Anzahl der Worker-Knoten, die im Lieferumfang des Lite- und des Standard-Plans enthalten sind.{:tip}

   ![Kubernetes-Cluster in IBM Cloud erstellen](images/solution2/KubernetesClusterCreation.png)
2. Wählen Sie den **Clustertyp** aus und klicken Sie auf **Cluster erstellen**, um einen Kubernetes-Cluster bereitzustellen.
3.  Überprüfen Sie den Status für Ihren **Cluster** und für Ihre **Workerknoten** und warten Sie, bis der Status **ready** (bereit) lautet.

### Tool 'kubectl' konfigurieren

In diesem Schritt konfigurieren Sie das Tool 'kubectl' so, dass Ihr neu erstellter Cluster referenziert wird. Das Befehlszeilentool [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) wird für die Interaktion mit einem Kubernetes-Cluster verwendet.

1. Melden Sie sich mit `ibmcloud login` interaktiv an. Geben Sie die Organisation (org), den Standort und den Bereich an, in dem der Cluster erstellt wird. Sie können die Details überprüfen, indem Sie den Befehl `ibmcloud target` ausführen.
2. Sobald der Cluster betriebsbereit ist, rufen Sie die Clusterkonfiguration ab, indem Sie in der Umgebungsvariablen MYCLUSTER den Namen Ihres Clusters angeben:
   ```bash
   export MYCLUSTER=<CLUSTERNAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
   ```
   {: pre}
3. Kopieren Sie den Befehl **export** und fügen Sie ihn ein, um die Umgebungsvariable KUBECONFIG wie angegeben festzulegen. Führen Sie den folgenden Befehl aus, um zu prüfen, ob die Umgebungsvariable KUBECONFIG ordnungsgemäß festgelegt ist:
  `echo $KUBECONFIG`
4. Überprüfen Sie, dass der Befehl `kubectl` ordnungsgemäß konfiguriert ist:
   ```bash
   kubectl cluster-info
   ```
   {: pre}
   ![](images/solution2/kubectl_cluster-info.png)


## Starteranwendung erstellen
{: #create_application}

Die Tools in `ibmcloud dev` verkürzen die Entwicklungszeit erheblich durch das Generieren von Starteranwendungen mit dem erforderlichen Boilerplate-, Build- und Konfigurationscode, damit Sie schneller mit dem Codieren Ihrer Geschäftslogik beginnen können.

1. Starten Sie den `ibmcloud dev`-Assistenten:
   ```
   ibmcloud dev create
   ```
   {: pre}

1. Wählen Sie `Back-End-Service / Web-App` > `Java - MicroProfile / JavaEE` > `Java-Web-App mit Eclipse MicroProfile und Java EE (Web-App)` aus, um ein Java-Starterprogramm zu erstellen. Wenn Sie stattdessen ein Node.js-Starterprogramm erstellen möchten, verwenden Sie die Option `Back-End-Service / Web-App` > `Node`> `Node.js-Web-App mit Express.js (Web-App)`.
1. Geben Sie einen **Namen** für Ihre Anwendung ein.
1. Wählen Sie die Ressourcengruppe aus, in der diese Anwendung bereitgestellt werden soll.
1. Fügen Sie keine zusätzlichen Services hinzu.
1. Fügen Sie keine DevOps-Toolchain hinzu und wählen Sie die **manuelle Bereitstellung** aus.

Dadurch wird eine vollständige Starteranwendung mit dem zugehörigen Code und allen erforderlichen Konfigurationsdateien für die lokale Bereitstellung und die Bereitstellung in der Cloud in Cloud Foundry oder Kubernetes generiert. 

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### Anwendung erstellen

Sie können die Anwendung auf die gewohnte Weise mit `mvn` (für die lokale Entwicklung mit Java) oder mit `npm` (für die Entwicklung mit Node.js) erstellen und ausführen. Darüber hinaus können Sie ein Docker-Image erstellen und die Anwendung in einem Container ausführen, um eine konsistente Ausführung auf dem lokalen System und in der Cloud zu gewährleisten. Führen Sie die folgenden Schritte aus, um Ihr Docker-Image zu erstellen.

1. Stellen Sie sicher, dass Ihre lokale Docker-Engine gestartet ist:
   ```
   docker ps
   ```
   {: pre}
2. Wechseln Sie in das generierte Projektverzeichnis:
   ```
   cd <project name>
   ```
   {: pre}
3. Erstellen Sie den Build der Anwendung:
   ```
   ibmcloud dev build
   ```
   {: pre}

   Dieser Vorgang kann mehrere Minuten dauern, bis alle Anwendungsabhängigkeiten heruntergeladen wurden und ein Docker-Image mit Ihrer Anwendung und der gesamten erforderlichen Umgebung erstellt ist.

### Anwendung lokal ausführen

1. Führen Sie den Container aus:
   ```
   ibmcloud dev run
   ```
   {: pre}

   Dabei wird Ihre lokale Docker-Engine verwendet, um das Docker-Image auszuführen, das Sie im vorherigen Schritt erstellt haben.
2. Nachdem Ihr Container gestartet wurde, rufen Sie `http://localhost:9080/` auf. Wenn Sie eine Node.js-Anwendung erstellt haben, rufen Sie `http://localhost:3000/` auf.
  ![](images/solution2/LibertyLocal.png)

## Anwendung mithilfe eines Helm-Diagramms im Cluster bereitstellen
{: #deploy}

In diesem Abschnitt übertragen Sie zunächst das Docker-Image mit einer Push-Operation in die private IBM Cloud-Containter-Registry und erstellen anschließend eine Kubernetes-Bereitstellung, die auf dieses Image verweist.

1. Finden Sie Ihren **Namensbereich**, indem Sie alle in der Registry enthaltenen Namensbereiche auflisten:
   ```sh
   ibmcloud cr namespaces
   ```
   {: pre}
   Wenn Sie über einen Namensbereich verfügen, notieren Sie den Namen für die spätere Verwendung. Andernfalls erstellen Sie einen Namensbereich:
   ```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. Geben Sie in den Umgebungsvariablen MYNAMESPACE und MYPROJECT Ihren Namensbereich und Ihren Projektnamen an:

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. Ermitteln Sie Ihre **Container-Registry** (z. B. us.icr.io), indem Sie den Befehl `ibmcloud cr info` ausführen.
4. Geben Sie in der Umgebungsvariablen MYREGISTRY Ihre Registry an:
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. Erstellen Sie das Docker-Image und versehen Sie es mit Tags (Option `-t`):
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. Übertragen Sie das Docker-Image mit einer Push-Operation in Ihre Container-Registry in IBM Cloud:
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. Navigieren Sie in einer IDE zu **values.yaml** im Verzeichnis `chart\IHR PROJEKTNAME` und geben Sie als Wert für das **Image-Repository** Ihr Image aus der IBM Cloud-Container-Registry an. Wählen Sie die Option **Speichern** aus, um die Datei zu speichern.

   Rufen Sie die Details für das Image-Repository auf, indem Sie den Befehl `echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}` ausführen.

8. [Helm](https://helm.sh/) unterstützt Sie bei der Verwaltung von Kubernetes-Anwendungen durch Helm-Diagramme, die das Definieren, Installieren und Aktualisieren hoch komplexer Kubernetes-Anwendungen erleichtern. Initialisieren Sie Helm, indem Sie zu `chart\IHR PROJEKTNAME` navigieren und den folgenden Befehl in Ihrem Cluster ausführen:

   ```bash
   helm init
   ```
   {: pre}
   Führen Sie diesen Befehl aus, um ein Upgrade für Helm durchzuführen: `helm init --upgrade`.
   {:tip}

9. Wechseln Sie zum Installieren eines Helm-Diagramms in das Verzeichnis `chart\IHR PROJEKTNAME` und führen Sie den folgenden Befehl aus:
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. Verwenden Sie `kubectl get service ${MYPROJECT}-service` für Ihre Java-Anwendung und `kubectl get service ${MYPROJECT}-application-service` für Ihre Node.js-Anwendung, um den öffentlichen Port zu ermitteln, der von dem Service überwacht wird. Die Portnummer ist als fünfstellige Zahl (z. B. 31569) unter `PORT(S)` angegeben.
11. Führen Sie den folgenden Befehl aus, um die öffentliche IP-Adresse des Workerknotens zu ermitteln:
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. Rufen Sie die Anwendung unter der Webadresse `http://worker-ip-adresse:portnummer/` auf.

## Von IBM bereitgestellte Domäne für Ihren Cluster verwenden
{: #ibm_domain}

Im vorherigen Schritt wurde die Anwendung über einen nicht standardmäßigen Port aufgerufen. Der Service wurde über die Kubernetes-Funktion NodePort zugänglich gemacht.

Für kostenpflichtige Cluster wird von IBM eine Domäne zur Verfügung gestellt. Dadurch können Sie Anwendungen wie gewohnt mit einer URL-Adresse und über standardmäßige HTTP- bzw. HTTPS-Ports verfügbar machen.

Verwenden Sie Ingress, um die eingehende Clusterverbindung für den Service einzurichten.

![Ingress](images/solution2/Ingress.png)

1. Ermitteln Sie die von IBM bereitgestellte **Ingress-Domäne**:
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   Suchen Sie die folgenden Angaben:
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. Erstellen Sie eine Ingress-Datei `ingress-ibmdomain.yml`, die auf Ihre Domäne verweist und Unterstützung für HTTP und HTTPS bietet. Verwenden Sie die folgende Datei als Vorlage und ersetzen Sie alle in <> eingeschlossenen Werte durch die entsprechenden Werte aus der obigen Ausgabe. Dabei ist **service-name** der Name aus `==> v1/Service` im vorherigen Schritt. Oder führen Sie `kubectl get svc` aus, um den Servicenamen des Typs **NodePort** zu finden.
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-ibmdomain-http-and-https
   spec:
     tls:
     - hosts:
       -  <projektname>.<ingress-unterdomäne>
       secretName: <geheimer_ingress-schlüssel>
     rules:
     - host: <nameofproject>.<ingress-unterdomäne>
       http:
         paths:
         - path: /
           backend:
             serviceName: <servicename>
             servicePort: 9080
   ```
   {: codeblock}
3. Stellen Sie die Ingress-Datei bereit:
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. Rufen Sie Ihre Anwendung unter der Webadresse `https://<nameofproject>.<ingress-sub-domain>/` auf.

## Eigene angepasste Domäne verwenden
{: #custom_domain}

Um Ihre angepasste Domäne zu verwenden, müssen Sie Ihre DNS-Datensätze entweder mit einem CNAME-Datensatz aktualisieren, der auf Ihre von IBM bereitgestellte Domäne verweist, oder mit einem A-Datensatz, der auf die portierbare, öffentliche IP-Adresse der von IBM bereitgestellten Ingress-Instanz verweist. Da ein kostenpflichtiger Cluster eine feste IP-Adresse beinhaltet, empfiehlt sich die Verwendung eines A-Datensatzes.

Weitere Informationen finden Sie im Abschnitt [Ingress-Controller mit angepasster Domäne verwenden](https://{DomainName}/docs/containers?topic=containers-ingress#ingress).

### Mit HTTP

1. Erstellen Sie eine Ingress-Datei `ingress-customdomain-http.yml` , die auf Ihre Domäne verweist:
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-customdomain-http
   spec:
     rules:
     - host: <meine-angepasste-domäne.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <servicename>
             servicePort: 9080
   ```
   {: pre}
2. Stellen Sie die Ingress-Datei bereit:
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. Rufen Sie Ihre Anwendung unter der Webadresse `http://<customdomain>/` auf.

### Mit HTTPS

Wenn Sie an dieser Stelle versuchen, Ihre Anwendung über HTTPS aufzurufen (`https://<customdomain>/`), weist in Ihrem Web-Browser wahrscheinlich eine Sicherheitswarnung darauf hin, dass dies keine private Verbindung ist. Außerdem wird ein Fehler des Typs 404 ausgegeben, da die aktuelle Ingress-Konfiguration keinen Datenverkehr über HTTPS weiterleiten kann.

1. Fordern Sie ein anerkanntes SSL-Zertifikat für Ihre Domäne an. Sie benötigen das folgende Zertifikat und den zugehörigen Schlüssel: 
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3.
   Sie können über [Let's Encrypt](https://letsencrypt.org/) ein anerkanntes Zertifikat generieren.
2. Speichern Sie das Zertifikat und den Schlüssel in ASCII-Dateien mit Base64-Codierung.
3. Erstellen Sie einen geheimen TLS-Schlüssel zum Speichern des Zertifikats und des Schlüssels:
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<angepasste-domäne.cert> --key=<angepasste-domäne.key>
   ```
   {: pre}
4. Erstellen Sie eine Ingress-Datei `ingress-customdomain-https.yml`, die auf Ihre Domäne verweist:
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-customdomain-https
   spec:
     tls:
     - hosts:
       - <meine_angepasste_domäne.com>
       secretName: <name-des-geheimen-schlüssels-meiner-angepassten-domäne>
     rules:
     - host: <meine-angepasste-domäne.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <servicename>
             servicePort: 9080
   ```
   {: pre}
5. Stellen Sie die Ingress-Datei bereit:
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. Rufen Sie Ihre Anwendung unter der Webadresse `https://<customdomain>/` auf.

## Status der Anwendung überwachen
{: #monitor_application}

1. Um den Status Ihrer Anwendung zu überprüfen, navigieren Sie zu [Cluster](https://{DomainName}/containers-kubernetes/clusters), um eine Liste der Cluster anzuzeigen, und klicken Sie auf den Cluster, den Sie zuvor erstellt haben.
2. Klicken Sie auf **Kubernetes-Dashboard**, um das Dashboard in einer neuen Registerkarte aufzurufen.
   ![](images/solution2/launch_kubernetes_dashboard.png)
3. Wählen Sie in der linken Anzeige **Knoten** aus, klicken Sie auf den **Namen** des Knotens und prüfen Sie den Status Ihrer Knoten unter **Zuordnungsressourcen**.
   ![](images/solution2/KubernetesDashboard.png)
4. Um die Anwendungsprotokolle im Container zu überprüfen, wählen Sie **Pods**, **Podname** und **Protokolle** aus.
5. Um den Container über **ssh** aufzurufen, übernehmen Sie den Podnamen aus dem vorherigen Schritt und führen Sie den folgenden Befehl aus:
   ```sh
   kubectl exec -it <podname> -- bash
   ```
   {: pre}

## Kubernetes-Pods skalieren
{: #scale_cluster}

Wenn die Workload für Ihre Anwendung zunimmt, können Sie die Anzahl der Podreplikate in Ihrer Bereitstellung manuell erhöhen. Replikate werden in einer Replikatgruppe ([ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)) verwaltet. Führen Sie den folgenden Befehl aus, um die Anwendung auf zwei Replikate zu skalieren:

   ```sh
 kubectl scale deployment <projektname>-deployment --replicas=2
   ```
   {: pre}

Nach kurzer Zeit werden im Kubernetes-Dashboard (oder durch den Befehl `kubectl get pods`) zwei Pods für Ihre Anwendung angezeigt. Der Ingress-Controller im Cluster steuert die Verteilung der Workload auf die beiden Replikate. Die horizontale Skalierung kann auch automatisiert werden.

Informationen zur manuellen und automatischen Skalierung finden Sie in der folgenden Kubernetes-Dokumentation:

   * [Bereitstellung skalieren](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [Automatische horizontale Skalierung für Pods](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## Ressourcen entfernen

* Löschen Sie den Cluster oder löschen Sie nur die für die Anwendung erstellten Kubernetes-Artefakte, falls Sie den Cluster wiederverwenden möchten.

## Zugehörige Inhalte

* [IBM Cloud-Kubernetes-Service](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [Continuous Delivery in Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
