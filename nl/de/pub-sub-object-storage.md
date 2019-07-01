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

# Asynchrone Datenverarbeitung mit Objektspeicher und Publish/Subscribe-Messaging
{: #pub-sub-object-storage}
In diesem Lernprogramm erfahren Sie, wie Sie mit einem auf Apache Kafka basierenden Messaging-Sservice lange laufende Workloads für Anwendungen koordinieren, die in einem Kubernetes-Cluster ausgeführt werden. Dieses Muster dient zum Entkoppeln Ihrer Anwendung und bietet bessere Steuerungsmöglichkeiten für Skalierung und Leistung. {{site.data.keyword.messagehub}} kann verwendet werden, um anstehende Arbeit in die Warteschlange zu stellen, ohne die Produktionsanwendungen zu beeinträchtigen. Dadurch eignet sich dieses System besonders gut für Tasks mit langer Laufzeit. 

{:shortdesc}

Dieses Muster wird unter Verwendung eines Beispiels für Dateiverarbeitung simuliert. Erstellen Sie zunächst eine Benutzerschnittstellenanwendung zum Hochladen von Dateien in den Objektspeicher und zum Generieren von Nachrichten für anstehende Arbeitsvorgänge. Erstellen Sie anschließend eine separate Workeranwendung, die die vom Benutzer hochgeladenen Dateien asynchron verarbeitet, sobald entsprechende Nachrichten empfangen werden.

## Ziele
{: #objectives}

* Erzeuger/Verbraucher-Muster mithilfe von {{site.data.keyword.messagehub}} implementieren
* Services an einen Kubernetes-Cluster binden

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur
{: #architecture}

In diesem Lernprogramm wird die Benutzerschnittstellenanwendung in Node.js geschrieben und die Workeranwendung in Java, um die Flexibilität dieses Musters zu veranschaulichen. Obwohl beide Anwendungen in diesem Lernprogramm im selben Kubernetes-Cluster ausgeführt werden, könnte jede der Anwendungen auch als Cloud Foundry-Anwendung oder als serverunabhängige Funktion implementiert werden.

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. Die Datei wird vom Benutzer mithilfe der Benutzerschnittstellenanwendung hochgeladen.
2. Die Datei wird in {{site.data.keyword.cos_full_notm}} gespeichert.
3. Eine Nachricht, die darauf hinweist, dass die neue Datei zur Verarbeitung ansteht, wird an das Thema {{site.data.keyword.messagehub}} gesendet.
4. Sobald die Betriebsbereitschaft hergestellt ist, sind Worker für Nachrichten empfangsbereit und beginnen mit der Verarbeitung der neuen Datei.

## Vorbereitende Schritte
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - Installationstool für {{site.data.keyword.cloud_notm}} CLI, Kubernetes, Helm und Docker.

## Kubernetes-Cluster erstellen
{: #create_kube_cluster}

1. Erstellen Sie einen Kubernetes-Cluster aus dem [Katalog](https://{DomainName}/containers-kubernetes/launch). Ordnen Sie dem Cluster den Namen `mycluster` zu, um die Durchführung dieses Lernprogramms zu vereinfachen. Dieses Lernprogramm kann mit einem **kostenlosen** Cluster durchgeführt werden.
   ![Kubernetes Cluster Creation on IBM Cloud](images/solution25/KubernetesClusterCreation.png)
2. Überprüfen Sie den Status für Ihren **Cluster** und für Ihre **Workerknoten** und warten Sie, bis der Status **ready** (bereit) lautet.

### Tool 'kubectl' konfigurieren

In diesem Schritt konfigurieren Sie das Tool 'kubectl' so, dass Ihr neu erstellter Cluster referenziert wird. Das Befehlszeilentool [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) wird für die Interaktion mit einem Kubernetes-Cluster verwendet.

1. Melden Sie sich mit `ibmcloud login` interaktiv an. Geben Sie die Organisation (org), den Standort und den Bereich an, in dem der Cluster erstellt wird. Sie können die Details überprüfen, indem Sie den Befehl `ibmcloud target` ausführen.
2. Sobald der Cluster betriebsbereit ist, rufen Sie die Clusterkonfiguration ab:
   ```bash
   ibmcloud cs cluster-config <clustername>
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

## {{site.data.keyword.messagehub}}-Instanz erstellen
 {: #create_messagehub}

{{site.data.keyword.messagehub}} ist ein schneller, skalierbarer, vollständig verwalteter Messaging-Service auf der Basis von Apache Kafka, einem Open-Source-Messaging-System mit hoher Durchsatzrate, das eine Plattform mit niedriger Latenzzeit für die Verarbeitung von Echtzeitdatenfeeds bereitstellt.

 1. Klicken Sie im Dashboard auf [**Ressource erstellen**](https://{DomainName}/catalog/) und wählen Sie [**{{site.data.keyword.messagehub}}**](https://{DomainName}/catalog/services/event-streams) im Abschnitt 'Application Services' aus.
 2. Ordnen Sie dem Service den Namen `mymessagehub` zu und klicken Sie auf **Erstellen**.
 3. Geben Sie die Serviceberechtigungsnachweise für Ihren Cluster an, indem Sie die Serviceinstanz an den Kubernetes-Namensbereich `default` binden:
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

Der Befehl 'cluster-service-bind' erstellt einen geheimen Schlüssel für den Cluster, der die Berechtigungsnachweise Ihrer Serviceinstanz im JSON-Format enthält. Zeigen Sie mit dem Befehl `kubectl get secrets ` den generierten geheimen Schlüssel mit dem Namen `binding-mymessagehub` an. Weitere Informationen finden Sie im Abschnitt [Services integrieren](https://{DomainName}/docs/containers?topic=containers-integrations#integrations).

{:tip}

## Object Storage-Instanz erstellen

{: #create_cos}

{{site.data.keyword.cos_full_notm}} wird verschlüsselt, auf mehrere geografische Standorte verteilt und mit HTTP über eine REST-API aufgerufen. {{site.data.keyword.cos_full_notm}} stellt flexiblen, kosteneffizienten und skalierbaren Cloudspeicher für unstrukturierte Daten bereit. In diesem Speicher werden die mit der Benutzerschnittstelle hochgeladenen Dateien gespeichert.

1. Klicken Sie im Dashboard, auf [**Ressource erstellen**](https://{DomainName}/catalog/) und wählen Sie [**{{site.data.keyword.cos_short}}**](https://{DomainName}/catalog/services/cloud-object-storage) im Abschnitt 'Speicher' aus.
2. Ordnen Sie dem Service den Namen `myobjectstorage` zu und klicken Sie auf **Erstellen**.
3. Klicken Sie auf **Bucket erstellen**.
4. Legen Sie für den Bucket einen eindeutigen Namen fest (z. B. `username-mybucket`).
5. Wählen Sie **regionsübergreifende** Ausfallsicherheit und den Standort **us-geo** aus und klicken Sie auf **Erstellen**.
6. Geben Sie die Serviceberechtigungsnachweise für Ihren Cluster an, indem Sie die Serviceinstanz an den Kubernetes-Namensbereich `default` binden:
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## Befehlszeilenanwendung im Cluster bereitstellen

Die Benutzerschnittstellenanwendung ist eine einfache Node.js-Express-Webanwendung, die es dem Benutzer ermöglicht, Dateien hochzuladen. Diese Anwendung speichert Dateien in der zuvor erstellten Object Storage-Instanz und sendet anschließend eine Nachricht an das {{site.data.keyword.messagehub}}-Thema 'work-topic', die angibt, dass eine Datei zur Verarbeitung bereit ist.

1. Klonen Sie das Beispielanwendungsrepository lokal und wechseln Sie in den Ordner `pubsub-ui`:
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. Öffnen Sie die Datei `config.js` und aktualisieren Sie 'COSBucketName' mit Ihrem Bucketnamen.
3. Erstellen Sie die Anwendung und stellen Sie sie bereit. Der Bereitstellungsbefehl generiert ein Docker-Image überträgt es mit einer Push-Operation in Ihre {{site.data.keyword.registryshort_notm}} und erstellt anschließend eine Kubernetes-Bereitstellung. Folgen Sie beim Bereitstellen der App den interaktiven Anweisungen.
```sh
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
4. Rufen Sie die Anwendung auf und laden Sie die Daten aus dem Ordner `sample-files` hoch. Die hochgeladenen Dateien werden in Object Storage gespeichert und in den Status 'awaiting' (anstehend) versetzt, bis sie von der Workeranwendung verarbeitet werden. Lassen Sie dieses Browserfenster geöffnet.

   ![](images/solution25/files_uploaded.png)

## Stellen Sie die Workeranwendung für den Cluster bereit

Die Workeranwendung ist eine Java-Anwendung, die Nachrichten für das {{site.data.keyword.messagehub}}-Kafka-Thema 'work-topic' überwacht. Wenn eine neue Nachricht erkannt wird, ruft der Worker den Namen der Datei aus der Nachricht ab und anschließend den Dateiinhalt aus Object Storage. Danach wird die Verarbeitung der Datei simuliert und anschließend eine weitere Nachricht an das Thema 'result-work' gesendet. Die Benutzerschnittstellenanwendung überwacht dieses Thema und aktualisiert den Status.

1. Wechseln Sie in das Verzeichnis `pubsub-worker`:
```sh
  cd ../pubsub-worker
```
2. Öffnen Sie die Datei `resources/cos.properties` und geben Sie in der Eigenschaft `bucket.name` Ihren Bucketnamen an.
2. Erstellen Sie die Workeranwendung und stellen Sie sie bereit:
```
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
3. Überprüfen Sie nach Abschluss der Bereitstellung das Browserfenster mit ihrer Webanwendung erneut. Beachten Sie, dass der Status für jede Datei jetzt in 'processed' (verarbeitet) geändert wurde.
![](images/solution25/files_processed.png)

In diesem Lernprogramm haben Sie erfahren, wie Sie mit dem auf Kafka basierenden {{site.data.keyword.messagehub}} ein Erzeuger/Verbraucher-Muster implementieren können. Dies beschleunigt die Ausführung der Webanwendung, indem umfangreiche Verarbeitungsvorgänge in andere Anwendungen ausgelagert werden. Wenn Arbeitsvorgänge anstehen, erstellt der Erzeuger (die Webanwendung) Nachrichten und die Workload wird per Lastausgleich auf eine oder mehrere Workeranwendungen verteilt, die Abonnenten der Nachrichten sind. Für die Verarbeitungsvorgänge haben Sie eine Java-Anwendung unter Kubernetes verwendet. Es können aber auch [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing)-Anwendungen verwendet werden. Unter Kubernetes ausgeführte Anwendungen sind ideal für intensive Workloads mit langer Laufzeit, während {{site.data.keyword.openwhisk_short}} für Prozesse mit kurzer Laufzeit besser geeignet ist.

## Ressourcen entfernen
{:removeresources}

Navigieren Sie zur [Ressourcenliste](https://{DomainName}/resources/) und führen Sie Folgendes aus:
1. Löschen Sie den Kubernetes-Cluster `mycluster`.
2. Löschen Sie {{site.data.keyword.cos_full_notm}} `myobjectstorage`.
3. Löschen Sie {{site.data.keyword.messagehub}} `mymessagehub`.
4. Wählen Sie in **Kubernetes** im linken Menü **Registry** aus und löschen Sie anschließend die Repositorys `pubsub-xxx`.

## Zugehörige Inhalte
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [Zugriff auf Object Storage verwalten](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [{{site.data.keyword.messagehub}}-Datenverarbeitung mit {{site.data.keyword.openwhisk_short}}](https://github.com/IBM/openwhisk-data-processing-message-hub)
