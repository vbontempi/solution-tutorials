---
copyright:
  years: 2019
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

# Isolierte Cloud Foundry Enterprise-Apps
{: #isolated-cloud-foundry-enterprise-apps}

Mit {{site.data.keyword.cfee_full_notm}} (CFEE) können Sie bei Bedarf mehrere isolierte, unternehmensspezifische Cloud Foundry-Plattformen erstellen. Damit erhalten Sie eine private Cloud Foundry-Instanz, die auf einem isolierten Kubernetes-Cluster implementiert ist. Anders als bei der öffentlichen Cloud haben Sie umfassende Kontrolle über die Umgebung, vor allem in Bereichen wie Zugriffssteuerung, Kapazität, Version, Ressourcennutzung und Überwachung. {{site.data.keyword.cfee_full_notm}} bietet die Geschwindigkeit und die Innovation einer Platform as a Service mit den Besitzanteilen an der Infrastruktur, wie sich sich auch in der unternehmensweiten IT-Infrastruktur finden.

Ein Anwendungsfall für {{site.data.keyword.cfee_full_notm}} ist eine unternehmenseigene Innovationsplattform. Sie als Entwickler in einem Unternehmen können entweder neue Mikroservices erstellen oder traditionelle Anwendungen in CFEE migrieren. Mikroservices können dann über den Cloud Foundry Marketplace für weitere Entwickler veröffentlicht werden. Dann können andere Entwickler in Ihrer CFEE-Instanz Services in der Anwendung so wie heute in der öffentlichen Cloud üblich verwenden.

Dieses Lernprogramm enthält die Schritte zum Erstellen und Konfigurieren einer {{site.data.keyword.cfee_full_notm}}-Instanz, Einrichten der Zugriffssteuerung und Implementieren von Apps und Services. Außerdem erfahren Sie mehr über die Beziehung zwischen CFEE und [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index), indem Sie einen angepassten Service-Broker implementieren, der angepasste Services mit CFEE integriert.

## Lernziele

{: #objectives}

- Vergleich und Gegenüberstellung von CFEE und einer öffentlichen Cloud Foundry-Instanz
- Apps und Services in CFEE implementieren
- Beziehung zwischen Cloud Foundry und {{site.data.keyword.containershort_notm}} verstehen
- Grundlegende Vernetzung von Cloud Foundry und {{site.data.keyword.containershort_notm}} untersuchen

## Verwendete Services

{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur

{: #architecture}

![Architektur](./images/solution45-CFEE-apps/Architecture.png)

1. Der Administrator erstellt eine CFEE-Instanz und fügt Benutzer mit Entwicklerzugriff hinzu.
2. Der Entwickler überträgt eine Node.js-Starter-App mit einer Push-Operation an CFEE.
3. Die Node.js-Starter-App verwendet [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) zum Speichern von Daten.
4. Der Entwickler fügt einen neuen Service für Willkommensnachrichten hinzu.
5. Die Node.js-Starter-App bindet den neuen Service von einem angepassten Service-Broker.
6. Die Node.js-Starter-App zeigt die Willkommensnachricht in verschiedenen Sprachen über den den Service an.

## Voraussetzungen

{: #prereq}

- [{{site.data.keyword.cloud_notm}}-CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [Cloud Foundry-CLI](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## {{site.data.keyword.cfee_full_notm}} bereitstellen

{:provision_cfee}

In diesem Abschnitt erstellen Sie eine Instanz von {{site.data.keyword.cfee_full_notm}}, die über {{site.data.keyword.containershort_notm}} in Kubernetes-Workerknoten implementiert wird.

1. [Bereiten Sie Ihr {{site.data.keyword.cloud_notm}}-Konto](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare) für die Erstellung der erforderlichen Infrastrukturressourcen vor.
2. Erstellen Sie über den {{site.data.keyword.cloud_notm}}-Katalog eine Serviceinstanz von [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create).
3. Konfigurieren Sie CFEE, indem Sie folgende Informationen angeben:
   - Wählen Sie den **Standard**-Plan aus.
   - Geben Sie einen **Namen** für die Serviceinstanz ein.
   - Wählen Sie eine **Ressourcengruppe** aus, in der die Umgebung erstellt wird. Sie benötigen die Berechtigung zum Zugriff auf mindestens eine Ressourcengruppe in dem Konto, um eine CFEE-Instanz erstellen zu können.
   - Wählen Sie eine **Region** und einen **Standort** aus, wo die Instanz bereitgestellt wird. Informationen hierzu finden Sie in der Liste der [verfügbaren Bereitstellungsstandorte und Rechenzentren](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets).
   - Wählen Sie eine öffentliche Cloud Foundry-**Organisation** und einen Cloud Foundry-**Bereich** für die Implementierung von **{{site.data.keyword.composeForPostgreSQL}}** aus.
   - Wählen Sie die **Anzahl der Zellen** für die Cloud Foundry-Umgebung aus. Eine Zelle führt Diego- und Cloud Foundry-Anwendungen aus. Für hoch verfügbare Anwendungen sind mindestens **2** Zellen erforderlich.
   - Wählen Sie den **Maschinentyp** aus, der die Größe der Cloud Foundry-Zellen (CPU und Speicher) bestimmt.
4. Überprüfen Sie den Abschnitt **Infrastruktur**, um die Eigenschaften des Kubernetes-Clusters anzuzeigen, der CFEE unterstützt. Die **Anzahl der Workerknoten** entspricht der Anzahl der Zellen plus 2. Zwei der bereitgestellten Kubernetes-Workerknoten fungieren als die CFEE-Steuerebene. Der Kubernetes-Cluster, auf dem die Umgebung implementiert ist, wird im {{site.data.keyword.cloud_notm}}-Dashboard für [Cluster](https://{DomainName}/containers-kubernetes/clusters) angezeigt.
5. Klicken Sie auf die Schaltfläche **Erstellen**, um die automatisierte Implementierung zu starten.

Die automatisierte Implementierung dauert etwa 90 bis 120 Minuten. Nach der erfolgreichen Erstellung erhalten Sie eine E-Mail, die die Bereitstellung von CFEE und unterstützenden Services bestätigt.

### Organisationen und Bereiche erstellen

Nachdem Sie {{site.data.keyword.cfee_full_notm}} erstellt haben, finden Sie im Abschnitt [Organisationen und Bereiche erstellen](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs) Informationen zur Strukturierung der Umgebung mithilfe von Organisationen und Bereichen. Apps in einer {{site.data.keyword.cfee_full_notm}}-Instanz beziehen sich auf bestimmte Bereiche. Ähnliches gilt für einen Bereich in einer Organisation. Mitglieder einer Organisation nutzen gemeinsam einen Kontingentplan, Apps, Serviceinstanzen und angepasste Domänen.

Hinweis: Das Menü **Verwalten > Konto > Cloud Foundry-Organisationen**, das sich in der oberen Kopfzeile von {{site.data.keyword.cloud_notm}} befindet, ist ausschließlich für öffentliche {{site.data.keyword.cloud_notm}}-Organisationen bestimmt. CFEE-Organisationen werden auf der Seite **Organisationen** einer CFEE-Instanz verwaltet.

Führen Sie die folgenden Schritte aus, um eine CFEE-Organisation und einen entsprechenden Bereich zu erstellen.

1. Wählen Sie im [Cloud Foundry-Dashboard](https://{DomainName}/dashboard/cloudfoundry/overview) unter **Unternehmen** die Option **Umgebungen** aus.
2. Wählen Sie Ihre CFEE-Instanz und dann **Organisationen** aus.
3. Klicken Sie auf die Schaltfläche **Organisationen erstellen**, geben Sie `tutorial` (Lernprogramm) als **Organisationsnamen** an und wählen Sie einen **Kontingentplan** aus. Klicken Sie abschließend auf **Hinzufügen**.
4. Klicken Sie auf die neu erstellte Organisation `tutorial`. Wählen Sie dann die Registerkarte **Bereiche** aus und klicken Sie auf die Schaltfläche **Bereich erstellen**.
5. Geben Sie `dev` als **Bereichsnamen** an und klicken Sie auf **Hinzufügen**.

### Benutzer zu Organisationen und Bereichen hinzufügen

In CFEE können Sie Rollen zuweisen, mit denen der Benutzerzugriff gesteuert werden kann. Dazu muss der entsprechende Benutzer jedoch auf der Seite **Identität & Zugriff** unter **Verwalten > Benutzer** in der Kopfzeile von {{site.data.keyword.cloud_notm}} zu Ihrem {{site.data.keyword.cloud_notm}}-Konto eingeladen werden.

Sobald der Benutzer eingeladen wurde, führen Sie die folgenden Schritte aus, um den Benutzer zur erstellten Organisation `tutorial` hinzuzufügen: 

1. Wählen Sie Ihre [CFEE-Instanz](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) und dann erneut **Organisationen** aus.
2. Wählen Sie die Organisation `tutorial` aus der Liste aus.
3. Klicken Sie auf die Registerkarte **Mitglieder**, um einen neuen Benutzer anzuzeigen und hinzuzufügen.
4. Klicken Sie auf die Schaltfläche **Mitglieder hinzufügen**, suchen Sie nach dem Benutzernamen, wählen Sie die entsprechenden **Organisationsrollen** aus und klicken Sie auf **Hinzufügen**.

Weitere Informationen zum Hinzufügen von Benutzern zu CFEE-Organisationen und -Bereichen finden Sie [hier](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users).

## CFEE-Anwendungen implementieren, konfigurieren und ausführen

{:deploycfeeapps}

In diesem Abschnitt implementieren Sie eine Node.js-Anwendung in CFEE. Nach der Implementierung binden Sie eine {{site.data.keyword.cloudant_short_notm}}-Instanz an diese Anwendung und aktivieren die Prüfungs- und Protokollierungspersistenz. Außerdem wird die Stratos Console zum Verwalten der Anwendung hinzugefügt.

### Anwendung in CFEE implementieren

1. Klonen Sie die Beispielanwendung [get-started-node](https://github.com/IBM-Cloud/get-started-node) über einen Terminal.
   ```sh
   git clone https://github.com/IBM-Cloud/get-started-node
   ```
   {: codeblock}
2. Führen Sie die App lokal aus, um sicherzustellen, dass sie erstellt und ordnungsgemäß gestartet wird. Bestätigen Sie den Vorgang über `http://localhost:3000/` in Ihrem Browser.
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. Melden Sie sich bei {{site.data.keyword.cloud_notm}} an und geben Sie Ihre CFEE-Instanz als Ziel an. Eine interaktive Eingabeaufforderung hilft Ihnen bei der Auswahl der neuen CFEE-Instanz. Da nur eine CFEE-Organisation und ein entsprechender Bereich vorhanden sind, werden diese als Standardziel verwendet. Sie können `ibmcloud target -o tutorial -s dev` ausführen, wenn Sie mehr als eine Organisation oder einen Bereich hinzugefügt haben.
   ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
   {: codeblock}
4. Übertragen Sie die App **get-started-node** mit einer Push-Operation an CFEE.
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. Der Endpunkt Ihrer App wird in der abschließenden Ausgabe neben der Eigenschaft `routes` angezeigt. Öffnen Sie die URL in Ihrem Browser, um zu bestätigen, dass die Anwendung ausgeführt wird.

### Cloudant-Datenbank erstellen und an die App binden

Wenn Sie {{site.data.keyword.cloud_notm}}-Services an die App **get-started-node** binden möchten, müssen Sie zunächst einen Service in Ihrem {{site.data.keyword.cloud_notm}}-Konto erstellen.

1. Erstellen Sie einen [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)-Service. Geben Sie als **Servicenamen** den Namen `cfee-cloudant` an und wählen Sie die Position aus, an der auch die CFEE-Instanz erstellt wurde.
2. Fügen Sie die neu erstellte {{site.data.keyword.cloudant_short_notm}}-Serviceinstanz zu CFEE hinzu.
   1. Navigieren Sie zurück zur **Organisation** `tutorial`. Klicken Sie auf die Registerkarte **Bereiche** und wählen Sie den Bereich `dev` aus.
   2. Wählen Sie die Registerkarte **Services** aus und klicken Sie auf die Schaltfläche **Service hinzufügen**.
   3. Geben Sie `cfee-cloudant` in das Suchfeld ein und wählen Sie das Ergebnis aus. Klicken Sie abschließend auf **Hinzufügen**. Der Service steht jetzt für CFEE-Anwendungen zur Verfügung; er befindet sich jedoch weiterhin in öffentlichen {{site.data.keyword.cloud_notm}}-Instanzen.
3. Wählen Sie im Überlaufmenü der angezeigten Serviceinstanz die Option **An Anwendung binden** aus.
4. Wählen Sie die zuvor übertragene App **get-started-node** aus und klicken Sie auf **Nach der Bindung erneutes Staging für die Anwendung durchführen**. Klicken Sie abschließend auf die Schaltfläche **Binden**. Warten Sie, bis das erneute Staging für die Anwendung durchgeführt wird. Sie können den Fortschritt mit dem Befehl `ibmcloud app show GetStartedNode` überprüfen.
5. Greifen Sie in Ihrem Browser auf die Anwendung zu, fügen Sie Ihren Namen hinzu und drücken Sie die `Eingabetaste`. Ihr Name wird zu einer {{site.data.keyword.cloudant_short_notm}}-Datenbank hinzugefügt.
6. Bestätigen Sie die Operation, indem Sie die `tutorial`-Instanz aus der Liste auf der Registerkarte **Services** auswählen. Dadurch wird die Detailseite der Serviceinstanz in der öffentlichen {{site.data.keyword.cloud_notm}} geöffnet.
7. Klicken Sie auf **Cloudant-Dashboard starten** und wählen Sie die Datenbank `mydb` aus. Es sollte ein JSON-Dokument mit Ihrem Namen vorhanden sein.

### Prüfungs- und Protokollierungspersistenz aktivieren

Mit Prüfungen können CFEE-Administratoren Cloud Foundry-Aktivitäten wie z. B. die Anmeldung, die Erstellung von Organisationen und Bereichen, die Benutzerzugehörigkeit und die Rollenzuweisungen, die Anwendungsimplementierungen, die Servicebindungen und die Domänenkonfiguration verfolgen. Die Prüfungsfunktionalität wird durch Integration mit dem {{site.data.keyword.cloudaccesstrailshort}}-Service unterstützt.

Cloud Foundry-Anwendungsprotokolle können durch Integration von {{site.data.keyword.loganalysisshort_notm}} gespeichert werden. Die von einem CFEE-Administrator ausgewählte {{site.data.keyword.loganalysisshort_notm}}-Serviceinstanz wird automatisch konfiguriert, um Cloud Foundry-Protokollierungsereignisse, die von der CFEE-Instanz generiert wurden, zu empfangen und persistent zu speichern.

Um die Prüfungs- und Protokollierungspersistenz in CFEE zu aktivieren, befolgen Sie die [hier aufgeführten Schritte](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging).

### Stratos-Konsole zum Verwalten der App installieren

Die Stratos-Konsole ist ein webbasiertes Open-Source-Tool für die Arbeit mit Cloud Foundry. Die Stratos-Konsole kann optional installiert und in einer bestimmten CFEE-Umgebung verwendet werden, um Organisationen, Bereiche und Anwendungen zu verwalten.

Führen Sie die folgenden Schritte durch, um die Stratos-Konsole zu installieren:

1. Öffnen Sie die CFEE-Instanz, in der Sie die Stratos-Konsole installieren möchten.
2. Klicken Sie auf der Seite **Übersicht** auf **Stratos-Konsole installieren**. Die Schaltfläche ist nur für Benutzer mit Administrator- oder Editorberechtigungen sichtbar.
3. Wählen Sie im Dialogfeld zum Installieren der Stratos-Konsole eine Installationsoption aus. Sie können die Stratos-Konsole entweder in der CFEE-Steuerebene oder in einer der Zellen installieren. Wählen Sie eine Version der Stratos-Konsole und die Anzahl der zu installierenden Instanzen der Anwendung aus. Wenn Sie die Stratos-Konsole in einer Zelle installieren, werden Sie aufgefordert, die Organisation und den Bereich anzugeben, in denen die Anwendung implementiert werden soll.
4. Klicken Sie auf **Installieren**.

Die Installation der Anwendung dauert ca. 5 Minuten. Sobald die Installation abgeschlossen ist, erscheint anstelle der Schaltfläche **Stratos-Konsole installieren** die Schaltfläche **Stratos-Konsole** auf der Übersichtsseite. Weitere Informationen zur Stratos-Konsole finden Sie [hier](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app).

## Beziehung zwischen CFEE und Kubernetes

Als Anwendungsplattform wird CFEE in einer Art dedizierter oder gemeinsam genutzter virtueller Infrastruktur ausgeführt. Viele Jahre lang machten sich Entwickler wenig Gedanken über die zugrunde liegende Cloud Foundry-Plattform, da IBM diese für sie verwaltete. Mit CFEE sind Sie nicht nur ein Entwickler, der Cloud Foundry-Anwendungen schreibt, sondern auch einen Betreiber der Cloud Foundry-Plattform. Der Grund dafür ist, dass CFEE in einem Kubernetes-Cluster implementiert wird, den Sie steuern.

Selbst wenn viele Cloud Foundry-Entwickler möglicherweise nicht mit Kubernetes vertraut sind, gibt es doch viele Konzepte, die beide Plattformen teilen. Wie Cloud Foundry isoliert Kubernetes Anwendungen in Containern, die in einem Kubernetes-Konstrukt, einem so genannten Pod, ausgeführt werden. Ähnlich wie bei Anwendungsinstanzen können Pods mehrere Kopien (auch Replikatgruppen genannt) mit dem Anwendungslastausgleich von Kubernetes haben. Die Cloud Foundy-App `GetStartedNode`, die Sie zuvor implementiert haben, wird im Pod `diego-cell-0` ausgeführt. Um eine hohe Verfügbarkeit zu unterstützen, würde dann ein weiterer Pod `diego-cell-1` auf einem separaten Kubernetes-Workerknoten ausgeführt werden. Da diese Cloud Foundry-Apps "innerhalb" von Kubernetes ausgeführt werden, können Sie auch über eine Kubernetes-basierte Vernetzung mit anderen Kubernetes-Mikroservices kommunizieren. In den folgenden Abschnitten werden die Beziehungen zwischen CFEE und Kubernetes näher erläutert.

## Kubernetes-Service-Broker implementieren

In diesem Abschnitt implementieren Sie einen Mikroservice in Kubernetes, der als Service-Broker für CFEE fungiert. [Service-Broker](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md) stellen Details zu verfügbaren Services sowie zum Binden und zur Bereitstellung von Unterstützung für Ihre Cloud Foundry-Anwendung bereit. Sie haben einen integrierten {{site.data.keyword.cloud_notm}}-Service-Broker verwendet, um den {{site.data.keyword.cloudant_short_notm}}-Service hinzuzufügen. Jetzt implementieren und verwenden Sie einen angepassten Service-Broker. Anschließend ändern Sie die App `GetStartedNode` so, dass der Service-Broker verwendet wird, der eine "Willkommensnachricht" in mehreren Sprachen zurückgibt.

1. Klonen Sie in Ihrem Terminal die Projekte, die Kubernetes-Implementierungsdateien und die Implementierung des Service-Brokers bereitstellen.

   ```sh
   git clone https://github.com/IBM-Cloud/cfee-service-broker-kubernetes.git
   ```
   {: codeblock}

   ```sh
   cd cfee-service-broker-kubernetes
   ```
   {: codeblock}

   ```sh
   git clone https://github.com/IBM/sample-resource-service-brokers.git
   ```
   {: codeblock}

2. Erstellen und speichern Sie das Docker-Image, das den Service-Broker enthält, in der {{site.data.keyword.registryshort_notm}}. Verwenden Sie den Befehl `ibmcloud cr info`, um die Registry-URL manuell oder automatisch mit dem nachfolgenden Befehl `export REGISTRY` abzurufen. Mit dem Befehl `cr namespace-add` wird ein Namensbereich erstellt, in dem das Docker-Image gespeichert wird.

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   Erstellen Sie einen Namensbereich mit dem Namen `cfee-tutorial`.

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   Bearbeiten Sie die Datei `./cfee-service-broker-kubernetes/deployment.yml`. Aktualisieren Sie das Attribut `image` so, dass es Ihre Container-Registry-URL widerspiegelt.
3. Implementieren Sie das Container-Image im Kubernetes-Cluster von CFEE. Dieser Cluster befindet sich in der Standardressourcengruppe (`default`), die (falls noch nicht geschehen) als Ziel angegeben sein sollte. Exportieren Sie mithilfe des Clusternamens die Variable KUBECONFIG mit dem Befehl `cluster-config`. Erstellen Sie anschließend die Implementierung.
   ```sh
   ibmcloud target -g default
   ```
   {: codeblock}

   ```sh
   ibmcloud ks clusters
   ```
   {: codeblock}

   ```sh
   $(ibmcloud ks cluster-config <your-cfee-cluster-name> --export)
   ```
   {: codeblock}

   ```sh
   kubectl apply -f deployment.yml
   ```
   {: codeblock}
4. Stellen Sie sicher, dass die Pods den Status `Aktiv` aufweisen. Es kann ein paar Momente dauern, bis Kubernetes das Image mit einer Pull-Operation extrahiert und die Container startet. Beachten Sie, dass Sie zwei Pods haben, da die Datei `deployment.yml` zweit `Replikate` angefordert hat.
   ```sh
   kubectl get pods
   ```
   {: codeblock}

## Sicherstellen, dass der Service-Broker implementiert wurde

Nachdem Sie den Service-Broker implementiert haben, stellen Sie sicher, dass er ordnungsgemäß funktioniert. Sie überprüfen dies auf verschiedene Arten: zunächst mit dem Kubernetes-Dashboard, dann durch Zugriff auf den Broker über eine Cloud Foundry-App und schließlich durch das Binden eines Service vom Broker.

### Pods über das Kubernetes-Dashboard anzeigen

In diesem Abschnitt wird sichergestellt, dass Kubernetes-Artefakte über das {{site.data.keyword.containershort_notm}}-Dashboard konfiguriert werden.

1. Greifen Sie über die Seite mit den [Kubernetes-Clustern](https://{DomainName}/containers-kubernetes/clusters) auf den CFEE-Cluster zu, indem Sie auf das Zeilenelement klicken, das mit dem Namen Ihres CFEE-Service beginnt und auf **-cluster** endet.
2. Öffnen Sie das **Kubernetes-Dashboard**, indem Sie auf die entsprechende Schaltfläche klicken.
3. Klicken Sie im linken Menü auf den Link **Services** und wählen Sie **tutorial-broker-service** aus. Dieser Service wurde implementiert, als Sie in früheren Schritten `kubectl apply` ausgeführt haben.
4. Beachten Sie im angezeigten Dashboard Folgendes:
   - Dem Service wurde eine Overlay-IP-Adresse (172.x.x.x) bereitgestellt, die nur innerhalb des Kubernetes-Clusters aufgelöst werden kann.
   - Der Service verfügt über zwei Endpunkte, die den beiden Pods entsprechen, auf denen die Service-Broker-Container ausgeführt werden.

Nachdem Sie bestätigt haben, dass der Service verfügbar ist und als Proxy für Service-Broker-Pods dient, können Sie überprüfen, ob der Broker mit Informationen zu verfügbaren Services antwortet.

Sie können Cloud Foundry-bezogene Artefakte über das Kubernetes-Dashboard anzeigen. Klicken Sie dazu auf **Namensbereich**, um alle Namensbereiche mit Artefakten, einschließlich des Cloud Foundry-Namensbereichs (`cf`), anzuzeigen.
{: tip}

### Zugriff auf den Broker über einen Cloud Foundry-Container

Um die Cloud Foundry-Kommunikation mit Kubernetes zu demonstrieren, stellen Sie direkt von einer Cloud-Foundry-Anwendung eine Verbindung zum Service-Broker her.

1. Stellen Sie in Ihrem Terminal mithilfe von `ibmcloud target` sicher, dass Sie immer noch mit Ihrer CFEE-Organisation `tutorial` und dem Bereich `dev` verbunden sind. Falls erforderlich, geben Sie CFEE erneut als Ziel an.
   ```sh
   ibmcloud target --cf
   ```
   {: codeblock}
2. Standardmäßig ist SSH in Bereichen inaktiviert. Dies ist anders als bei der öffentlichen Cloud. Aktivieren Sie daher SSH in Ihrem CFEE-Bereich `dev`.
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. Verwenden Sie den `kubectl`-Befehl, um dieselbe Cluster-IP anzuzeigen, die Sie auch im Kubenetes-Dashboard gesehen haben. Stellen Sie über SSH eine Verbindung zur Anwendung `GetStartedNode` her und rufen Sie Daten vom Service-Broker mithilfe der IP-Adresse ab. Beachten Sie, dass der letzte Befehl zu einem Fehler führen kann, der aber mit dem nächsten Schritt behoben werden kann.
   ```sh
   kubectl get service tutorial-broker-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf ssh GetStartedNode
   ```
   {: codeblock}

   ```sh
   export CLUSTER_IP=<CLUSTER-IP-ADRESSE>
   ```
   {: codeblock}

   ```sh
   wget --user TestServiceBrokerUser --password TestServiceBrokerPassword -O- http://$CLUSTER_IP/v2/catalog
   ```
   {: codeblock}
4. Sie haben sehr wahrscheinlich einen Fehler des Typs **Verbindung zurückgewiesen** erhalten. Dies ist auf die standardmäßigen [Anwendungssicherheitsgruppen](https://docs.cloudfoundry.org/concepts/asg.html) von CFEE zurückzuführen. Eine Anwendungssicherheitsgruppe (Application Security Group, ASG) definiert den zulässigen IP-Bereich für den abgehenden Datenverkehr von einem Cloud Foundry-Container. Der Fehler tritt auf, da sich die Anwendung `GetStartedNode` außerhalb des Standardbereichs befindet. Beenden Sie die SSH-Sitzung und laden Sie die Anwendungssicherheitsgruppe `public_networks` herunter.
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. Bearbeiten Sie die Datei `public_networks.json` und überprüfen Sie, ob die verwendete Cluster-IP-Adresse außerhalb der vorhandenen Regeln verwendet wird. Beispiel: Der Bereich `172.32.0.0-192.167.255.255` enthält wahrscheinlich nicht die Cluster-IP und muss aktualisiert werden. Wenn Ihre Cluster-IP-Adresse beispielsweise `172.21.107.63` lautet, müssten Sie die Datei bearbeiten, um so etwas wie `172.0.0.0-255.255.255.255` zu erhalten.
6. Passen Sie die Regel `destination` der Anwendungssicherheitsgruppe so an, dass sie die IP-Adresse des Kubernetes-Service enthält. Trimmen Sie die Datei so, dass sie nur die JSON-Daten enthält, die mit den eckigen Klammern beginnen und enden. Laden Sie dann die neue Anwendungssicherheitsgruppe hoch.
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. Wiederholen Sie Schritt 3, der nun mit Testkatalogdaten erfolgreich sein sollte. Schließen Sie den Vorgang ab, indem Sie die SSH-Sitzung beenden.
   ```sh
   exit
   ```
   {: codeblock}

### Service-Broker mit CFEE registrieren

Damit Entwickler Services vom Service-Broker bereitstellen und binden können, registrieren Sie diesen mit CFEE. Bisher haben Sie mit dem Broker über eine IP-Adresse gearbeitet. Dies ist jedoch problematisch. Wenn der Service-Broker erneut gestartet wird, erhält er eine neue IP-Adresse, was eine Aktualisierung von CFEE erforderlich macht. Um dieses Problem zu beheben, verwenden Sie eine weitere Kubernetes-Funktion namens KubeDNS, die einen vollständig qualifizierten Domänennamen (FQDN) für den Service-Broker bereitstellt.

1. Registrieren Sie den Service-Broker mit CFEE unter Verwendung des FQDN des Service `tutorial-service-broker`. Auch diese Route ist ein interner Bestandteil Ihres CFEE-Kubernetes-Clusters.
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. Fügen Sie dann die Services hinzu, die vom Broker angeboten werden. Da der Beispielbroker nur über einen einzigen Testservice verfügt, ist ein einziger Befehl erforderlich.
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. Greifen Sie in Ihrem Browser über die Seite mit den [**Umgebungen**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) auf Ihre CFE-Instanz zu. Navigieren Sie dann zu `Organisationen -> Bereiche` und wählen Sie den Bereich `dev` aus.
4. Wählen Sie die Registerkarte **Services** und die Schaltfläche **Service erstellen** aus.
5. Suchen Sie im Suchtexfeld nach **Test**. Der Testservice **Test Node Resource Service Broker Display Name** vom Broker wird angezeigt.
6. Wählen Sie den Service aus, klicken Sie auf die Schaltfläche **Erstellen** und geben Sie den Namen `welcome-service` an, um eine Serviceinstanz zu erstellen. Im nächsten Abschnitt wird deutlich, warum es 'welcome-service' heißt. Binden Sie dann den Service mit dem Element **An Anwendung binden** im Überlaufmenü an die App `GetStartedNode`.
7. Um den gebundenen Service anzuzeigen, führen Sie den Befehl `cf env` aus. Die Anwendung `GetStartedNode` kann nun die Daten im Objekt `credentials` in ähnlicher Form nutzen, wie sie aktuell Daten von `cloudantNoSQLDB` verwendet.
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## Service zur Anwendung hinzufügen

Bis zu diesem Punkt haben Sie zwar einen Service-Broker implementiert, aber keinen eigentlichen Service. Während der Service-Broker Bindungsdaten für `GetStartedNode` bereitstellt, wurde keine tatsächliche Funktionalität vom Service selbst hinzugefügt. Um dies zu korrigieren, wurde ein Testservice erstellt, der eine "Willkommensnachricht" an `GetStartedNode` in verschiedenen Sprachen bereitstellt. Sie implementieren nun diesen Service in CFEE und aktualisieren den Broker und die Anwendung für dessen Verwendung.

1. Übertragen Sie die Implementierung von `welcome-service` mit einer Push-Operation an CFEE, damit zusätzliche Entwicklungsteams den Service als API nutzen können.
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. Greifen Sie über Ihren Browser auf den Service zu. Die `Route` für den Service wird als Teil der Ausgabe des Befehls `push` angezeigt. Die resultierende URL ähnelt der folgenden URL: `https://welcome.<your-cfee-cluster-domain>`. Aktualisieren Sie die Seite, um die verschiedenen Sprachen zu sehen.
3. Kehren Sie zum Ordner `sample-resource-service-broker` zurück und bearbeiten Sie die Node.js-Beispielimplementierung `sample-resource-service-brokers/node-resource-service-broker/testresourceserviceservicebroker.js`, indem Sie durch Einfügen einer 'url'-Eigenschaft nach der Zeile 854 die Cluster-URL hinzufügen. 

   ```javascript
   // TODO - Tatsächliche Arbeit hier durchführen

   var generatedUserid   = uuid();
   var generatedPassword = uuid();
    
   result = {
     credentials: {
       userid   : generatedUserid,
       password : generatedPassword,
       url: 'https://welcome.<your-cfee-cluster-domain>'
     }
   };
   ```
   {: codeblock}
4. Erstellen und implementieren Sie den aktualisierten Service-Broker. Dadurch wird sichergestellt, dass die 'url'-Eigenschaft für Anwendungen bereitgestellt wird, die den Service binden.
   ```sh
   cd ..
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   ```sh
   kubectl patch deployment tutorial-servicebroker-deployment -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"version\":\"2\"}}}}}"
   ```
   {: codeblock}
5. Navigieren Sie im Terminal zurück zum Anwendungsordner `get-started-node`. 
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. Bearbeiten Sie die Datei `get-started-node/server.js` in `get-stared-node`, um die folgende Middleware einzuschließen. Fügen Sie sie unmittelbar vor dem Aufruf `app.listen()` ein.
   ```javascript
   // Use the welcome service
   const request = require('request');
   const testService = appEnv.services['testnoderesourceservicebrokername'];

   if (testService) {
     const { credentials: { url} } = testService[0];
     app.get("/api/welcome", (req, res) => request(url, (e, r, b) => res.send(b)));
   } else {
     app.get("/api/welcome", (req, res) => res.send('Welcome'));
   }
   ```
   {: codeblock}
7. Führen Sie ein Refactoring für die Seite `get-started-node/views/index.html` aus. Ändern Sie `data-i18n="welcome"` in `id="welcome"` im Tag `h1`. Fügen Sie dann am Ende der Funktion `$(document).ready()` einen Aufruf zum Service hinzu.
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. Aktualisieren Sie die App `GetStartedNode`. Schließen Sie die Paketabhängigkeit `request` mit ein, die zu `server.js` hinzugefügt wurde, binden Sie `welcome-service` erneut, um die neue `url`-Eigenschaft aufzunehmen, und übertragen Sie die abschließend den neuen Code der App mit einer Push-Operation.
   ```sh
   npm i request -S
   ```
   {: codeblock}

   ```sh
   ibmcloud cf unbind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf bind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}

Sie sind fertig! Besuchen Sie jetzt die Anwendung und aktualisieren Sie die Seite mehrmals, um die Willkommensnachricht in verschiedenen Sprachen anzuzeigen.

![Antwort des Service-Brokers](./images/solution45-CFEE-apps/service-broker.png)

Wenn Sie weiterhin nur **Willkommen** und keine anderen Sprachen sehen, führen Sie den Befehl `ibmcloud cf env GetStartedNode` aus und stellen Sie sicher, dass die Eigenschaft `url` für den Service vorhanden ist. Ist dies nicht der Fall, verfolgen Sie Ihre Schritte zurück, aktualisieren Sie den Broker und implementieren Sie ihn erneut. Wenn der Wert für `url` vorhanden ist, stellen Sie sicher, dass er eine Nachricht in Ihrem Browser zurückgibt. Führen Sie anschließend die Befehle `unbind-service` und `bind-service` erneut gefolgt von `ibmcloud cf restart GetStartedNode` aus.
{: tip}

Der Willkommensservice verwendet Cloud Foundry für die Implementierung, Sie könnten jedoch genauso gut Kubernetes verwenden. Der Hauptunterschied liegt darin, dass die URL zum Service wahrscheinlich `welcome-service.default.svc.cluster.local` sein würde. Die Verwendung der KubeDNS-URL hat den zusätzlichen Vorteil, dass der Datenverkehr zu Services intern im Kubernetes-Cluster stattfindet.

Mit {{site.data.keyword.cfee_full_notm}} sind diese alternativen Ansätzen jetzt möglich.

## Dieses Lernprogramm für Lösungen mithilfe einer Toolchain implementieren  

Optional können Sie das vollständige Lösungslernprogramm mithilfe einer Toolchain implementieren. Befolgen Sie die [Anweisungen zur Toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes), um alle oben genannten Komponenten mit einer Toolchain zu implementieren. 

Hinweis: Bei der Verwendung einer Toolchain müssen Sie unter anderem als Voraussetzung eine CFEE-Instanz sowie eine CFEE-Organisation und einen CFEE-Bereich erstellt haben. Detaillierte Anweisungen finden Sie in der Readme zu den [Anweisungen zur Toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes). 

## Zugehöriger Inhalt
{:related}

* [Apps in Kubernetes-Clustern implementieren](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Diego-Komponenten und -Architektur in Cloud Foundry](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [CFEE-Service-Broker in Kubernetes mit einer Toolchain](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

