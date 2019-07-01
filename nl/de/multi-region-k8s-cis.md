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

# Ausfallsichere und geschützte Kubernetes-Cluster für mehrere Regionen mit Cloud Internet Services
{: #multi-region-k8s-cis}

Für Benutzer kommt es zu weniger Ausfallzeiten, wenn eine Anwendung vor dem Hintergrund der Ausfallsicherheit konzipiert wird. Wenn Sie eine Lösung mit {{site.data.keyword.containershort_notm}} implementieren, profitieren Sie von integrierten Funktionen wie Lastausgleich und Isolation. Gleichzeitig erhöhen Sie die Ausfallsicherheit gegenüber potenziellen Fehlern bei Hosts, Netzen oder Apps. Wenn Sie mehrere Cluster erstellen und wenn ein Cluster ausfällt, können Benutzer trotzdem auf eine App zugreifen, die auch in einem anderen Cluster implementiert ist. Mit mehreren Clustern an verschiedenen Standorten können Benutzer auch auf den nächstgelegenen Cluster zugreifen und die Netzlatenzzeit verringern. Für zusätzliche Ausfallsicherheit haben Sie die Möglichkeit, auch Mehrfachzonen-Cluster auszuwählen. Dies bedeutet, dass Ihre Knoten in mehreren Zonen innerhalb eines Standorts implementiert werden.

Dieses Lernprogramm zeigt, wie Cloud Internet Services (CIS), eine einheitliche Plattform, mit der das Domain Name System (DNS), die globalen Lastausgleichsfunktion (GLB), Web Application Firewall (WAF) und der Distributed-Denial-of-Service-Schutz (DDoS) konfiguriert und verwaltet werden kann, für Internetanwendungen in Kubernetes-Cluster integriert werden kann, um dieses Szenario zu unterstützen und eine sichere und ausfallsichere Lösung an mehreren Standorten bereitzustellen.

## Lernziele
{: #objectives}

* Eine Anwendung auf mehreren Kubernetes-Clustern an verschiedenen Standorten implementieren
* Den Datenverkehr über mehrere Cluster hinweg mit einer globalen Lastausgleichsfunktion verteilen
* Benutzer zum am nahesten gelegenen Cluster weiterleiten
* Ihre Anwendung vor Sicherheitsbedrohungen schützen
* Anwendungsleistung mit Caching erhöhen

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur
{: #architecture}

<p style="text-align: center;">
  ![Architektur](images/solution32-multi-region-k8s-cis/Architecture.png)
</p>

1. Der Entwickler erstellt Docker-Images für die Anwendung.
2. Die Images werden mit einer Push-Operation an {{site.data.keyword.registryshort_notm}} in Dallas und London übertragen.
3. Die Anwendung wird an beiden Standorten auf Kubernetes-Clustern implementiert.
4. Endbenutzer greifen auf die Anwendung zu.
5. Cloud Internet Services ist so konfiguriert, dass Anforderungen an die Anwendung abgefangen werden und die Last über die Cluster verteilt wird. Außerdem wird die Anwendung mithilfe des DDoS-Schutzes und Web Application Firewall (WAF) vor allgemeinen Bedrohungen geschützt. Optionale Assets wie Images und CSS-Dateien werden in den Cache gestellt.

## Vorbereitende Schritte
{: #prereqs}

* Für Cloud Internet Services ist es erforderlich, dass Sie eine eigene Domäne besitzen, sodass Sie die DNS-Instanz für diese Domäne so konfigurieren können, dass sie auf die Namen von Cloud Internet Services-Namenservern verweist.
* [Installieren Sie Git](https://git-scm.com/).
* [Installieren Sie die {{site.data.keyword.Bluemix_notm}}-CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli).
* [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Script zur Installation von Docker, kubectl, Helm, 'ibmcloud'-CLI und der erforderlichen Plug-ins.
* [Richten Sie die {{site.data.keyword.registrylong_notm}}-CLI und den Namensbereich der Registry ein](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Machen Sie sich mit den Grundlagen von Kubernetes vertraut](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Anwendung an einem Standort implementieren

In diesem Lernprogramm wird eine Kubernetes-Anwendung auf Clustern an mehreren Standorten implementiert. Sie beginnen mit einem Standort (Dallas) und wiederholen Sie diese Schritte für London.

### Kubernetes-Cluster erstellen
{: #create_cluster}

Gehen Sie wie folgt vor, um einen Cluster zu erstellen:
1. Wählen Sie **{{site.data.keyword.containershort_notm}}** aus dem [{{site.data.keyword.cloud_notm}}-Katalog](https://{DomainName}/containers-kubernetes/catalog/cluster/create) aus.
1. Setzen Sie den **Standort** auf **Dallas**.
1. Wählen Sie den **Standard**-Cluster aus.
1. Wählen Sie eine oder mehrere Zonen als **Standort** aus. Durch die Erstellung eines Mehrfachzonen-Clusters erhöht sich die Ausfallsicherheit der Anwendung. Für Benutzer kommt es bei der Verteilung von Apps auf mehrere Zone zu deutlich weniger Ausfallzeiten. Weitere Informationen zu Mehrfachzonen-Clustern finden Sie [hier](https://{DomainName}/docs/containers?topic=containers-plan_clusters#ha_clusters).
1. Setzen Sie **Maschinentyp** auf die kleinsten verfügbaren Maschinentyp - **2 CPUs** und **4 GB RAM** ist ausreichend für dieses Lernprogramm.
1. Verwenden Sie **2** Workerknoten.
1. Setzen Sie **Clustername** auf **my-us-cluster**. Verwenden Sie aus Gründen der Konsistenz mit diesem Lernprogramm das Benennungsmuster *`my-<location>-cluster`*.

Während der Cluster für den Einsatz vorbereitet wird, bereiten Sie die Anwendung vor.

### Namensbereich in der {{site.data.keyword.registryshort_notm}} erstellen

1. Geben Sie für die {{site.data.keyword.Bluemix_notm}}-CLI 'Dallas' als Ziel an.
   ```bash
   ibmcloud target -r us-south
   ```
   {: pre}
2. Erstellen Sie einen Namensbereich für die Anwendung.
   ```bash
   ibmcloud cr namespace-add <ihr_namensbereich>
   ```
   {: pre}

Sie können auch einen vorhandenen Namensbereich wiederverwenden, wenn Sie über einen an diesem Standort verfügen. Mit dem Befehl `ibmcloud cr namespaces` können Sie die vorhandenen Namensbereiche auflisten.
{: tip}

### Anwendung erstellen

In diesem Schritt wird die Anwendung in einem Docker-Image erstellt. Bei der Konfiguration des zweiten Clusters können Sie diesen Schritt überspringen. Es ist eine einfache Hello World-App.

1. Klonen Sie den Quellcode für die [Hello World-App](https://github.com/IBM/container-service-getting-started-wt){:new_windows} in Ihr Benutzerausgangsverzeichnis. Das Repository enthält verschiedene Versionen einer ähnlichen App in Ordnern, die jeweils mit 'Lab' beginnen.
   ```bash
   git clone https://github.com/IBM/container-service-getting-started-wt.git
   ```
   {: pre}
1. Navigieren Sie zum Verzeichnis `Lab 1`.
   ```bash
   cd 'container-service-getting-started-wt/Lab 1'
   ```
   {: pre}
1. Erstellen Sie ein Docker-Image, das die App-Dateien des Verzeichnisses `Lab 1` enthält.
   ```bash
   docker build --tag multi-region-hello-world:1 .
   ```
   {: pre}

### Image vorbereiten, das an die standortbezogene Registry übertragen werden soll

Kennzeichnen Sie das Image mit der Zielregistry:

   ```bash
   docker tag multi-region-hello-world:1 us.icr.io/<ihr_namensbereich_us-south>/hello-world:1
   ```
   {: pre}

### Image an die standortbezogene Registry übertragen

1. Stellen Sie sicher, dass Ihre lokale Docker-Engine eine Push-Operation an die Registry in Dallas durchführen kann.
   ```bash
   ibmcloud cr login
   ```
   {: pre}
2. Übertragen Sie das Image mit einer Push-Operation.
   ```bash
   docker push us.icr.io/<ihr_namensbereich_us-south>/hello-world:1
   ```
   {: pre}

### Anwendung im Kubernetes-Cluster implementieren

An diesem Punkt sollte der Cluster bereit sein. Sie können seinen Status in der [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/clusters)-Konsole überprüfen.

1. Rufen Sie die Konfiguration des Clusters ab:
   ```bash
   ibmcloud cs cluster-config <name_des_us-clusters>
   ```
   {: pre}
1. Kopieren Sie die Ausgabe und fügen Sie sie ein, um die Umgebungsvariable KUBECONFIG zu definieren. Die Variable wird von `kubectl` verwendet.
1. Führen Sie die Anwendung im Cluster mit zwei Replikaten aus:
   ```bash
   kubectl run hello-world-deployment --image=us.icr.io/<ihr_namensbereich_us-south>/hello-world:1 --replicas=2
   ```
   {: pre}
   Beispielausgabe: `deployment "hello-world-deployment" created`.
1. Machen Sie die Anwendung im Cluster zugänglich:
   ```bash
   kubectl expose deployment/hello-world-deployment --type=ClusterIP --port=80 --name=hello-world-service --target-port=8080
   ```
   {: pre}
   Es wird eine Nachricht wie `service "hello-world-service" exposed` zurückgegeben.

### Domänenname und IP-Adresse abrufen, die dem Cluster zugeordnet sind
{: #CSALB_IP_subdomain}

Wenn ein Kubernetes-Cluster erstellt wird, wird ihm eine Ingress-Unterdomäne (z. B. *my-us-cluster.us-south.containers.appdomain.cloud*) und eine öffentliche IP-Adresse für die Lastausgleichsfunktion für Anwendungen zugeordnet.

1. Rufen Sie die Ingress-Unterdomäne des Clusters ab:
   ```bash
   ibmcloud cs cluster-get <name_des_us-clusters>
   ```
   {: pre}
   Suchen Sie den Wert für `Ingress-Unterdomäne`.
1. Notieren Sie diese Informationen für einen späteren Schritt.

In diesem Lernprogramm wird die Ingress-Unterdomäne verwendet, um die globalen Lastausgleichsfunktion zu konfigurieren. Sie können die Unterdomäne auch gegen die öffentliche IP-Adresse der Lastausgleichsfunktion für Anwendungen (`ibmcloud cs albs -cluster <us-cluster-name>`) austauschen. Beide Optionen werden unterstützt.
{: tip}

## Implementierung an einem weiteren Standort

Wiederholen Sie die vorherigen Schritte für London, wobei Sie folgende Ersetzungen vornehmen:
* den Standortnamen **Dallas** durch **London**;
* den Standortaliasname **us-south** durch **eu-gb**;
* die Registry *us.icr.io* durch **uk.icr.io**;
* und den Clusternamen *my-us-cluster* durch **my-uk-cluster**.

## Lastausgleich an mehreren Standorten konfigurieren

Ihre Anwendung wird jetzt in zwei Clustern ausgeführt. Es fehlt jedoch eine Komponente für die Benutzer, um transparent von einem einzigen Einstiegspunkt aus auf die Cluster zugreifen zu können.

In diesem Abschnitt konfigurieren Sie Cloud Internet Services (CIS), um die Last zwischen den beiden Clustern zu verteilen. CIS ist ein One-Stop-Shop-Service, der eine globale Lastausgleichsfunktion (GLB), Caching, Web Application Firewall (WAF) und Seitenregeln zum Sichern Ihrer Anwendungen bereitstellt und gleichzeitig die gewünschte Zuverlässigkeit und Leistung für Ihre Cloud-Anwendungen liefert.

Um eine globale Lastausgleichsfunktion zu konfigurieren, müssen Sie:
* mit einer angepassten Domäne auf CIS-Namenserver verweisen,
* die IP-Adressen oder Unterdomänennamen der Kubernetes-Custer abrufen,
* Statusprüfungen konfigurieren, um die Verfügbarkeit Ihrer Anwendungen zu überprüfen
* und Ursprungspools definieren, die auf die Cluster verweisen.

### Angepasste Domäne bei Cloud Internet Services registrieren
{: #create_cis_instance}

Der erste Schritt besteht darin, eine Instanz von CIS zu erstellen und Ihre angepasste Domäne auf CIS-Namensserver zu verweisen.

1. Wenn Sie nicht Eigentümer einer Domäne sind, können Sie eine Domäne von einem Registrator wie [godaddy.com](http://godaddy.com) erwerben.
2. Navigieren Sie zu den [Internet-Services](https://{DomainName}/catalog/services/internet-services) im {{site.data.keyword.Bluemix_notm}}-Katalog.
3. Legen Sie den Servicenamen fest und klicken Sie auf **Erstellen**, um eine Instanz des Service zu erstellen.
4. Wenn die Serviceinstanz bereitstellt wurde, geben Sie Ihren Domänennamen an und klicken Sie auf **Domäne hinzufügen**.
5. Wenn die Namensserver zuordnet wurden, konfigurieren Sie Ihren Registrator oder Domänennamenprovider für die Verwendung der aufgeführten Namensserver.
6. Nachdem Sie Ihren Registrator oder den DNS-Provider konfiguriert haben, kann es bis zu 24 Stunden dauern, bis die Änderungen wirksam werden.

   Wenn sich der Status der Domäne auf der Übersichtsseite von *Anstehend* in *Aktiv* ändert, können Sie mit dem Befehl `dig <your_domain_name> ns` überprüfen, ob die neuen Namensserver betriebsbereit sind.
   {:tip}

### Statusprüfung für globale Lastausgleichsfunktion konfigurieren

Anhand einer Statusprüfung erhalten Sie Einblicke in die Verfügbarkeit von Pools, um ggf. Datenverkehr an Pools in einwandfreiem Zustand leiten zu können. Diese Prüfungen senden regelmäßig HTTP/HTTPS-Anforderungen und überwachen die Antworten.

1. Navigieren Sie im Cloud Internet Services-Dashboard zu **Zuverlässigkeit** > **Globale Lastausgleichsfunktion** und klicken Sie dann unten auf der Seite auf **Statusprüfung erstellen**.
1. Setzen Sie **Pfad** auf **/**.
1. Setzen Sie **Überwachungstyp** auf **HTTP**.
1. Klicken Sie auf **1 Instanz bereitstellen**.

   Wenn Sie Ihre eigenen Anwendungen erstellen, können Sie einen dedizierten Statusendpunkt wie */heathz* definieren, an dem Sie den Anwendungsstatus melden.
   {:tip}

### Ursprungspools definieren

Ein Pool ist eine Gruppe von Ursprungsservern, an die der Datenverkehr intelligent weitergeleitet wird, wenn sie an eine globale Lastausgleichsfunktion angeschlossen ist. Bei Clustern in Großbritannien und den Vereinigten Staaten können Sie standortbasierte Pools definieren und CIS so konfigurieren, dass Benutzer basierend darauf, wo eine Benutzeranforderung ihren Ursprung hat, an den nächstgelegenen Cluster weitergeleitet werden.

#### Ein Pool für den Cluster in London
1. Klicken Sie auf **Pool erstellen**.
2. Setzen Sie **Name** auf **UK**.
3. Setzen Sie die Option **Statusprüfung** auf die im vorherigen Abschnitt erstellte Prüfung.
4. Setzen Sie **Region für Statusprüfung** auf **Westeuropa**.
5. Setzen Sie **Ursprungsname** auf **uk-cluster**.
6. Setzen Sie **Ursprungsadresse** auf die Ingress-Unterdomäne des Clusters in London, z. B. *my_uk_cluster.eu-gb.containers.appdomain.cloud*.
7. Klicken Sie auf **1 Instanz bereitstellen**.

#### Ein Pool für den Cluster in Dallas
1. Klicken Sie auf **Pool erstellen**.
2. Setzen Sie **Name** auf **US**.
3. Setzen Sie die Option **Statusprüfung** auf die im vorherigen Abschnitt erstellte Prüfung.
4. Setzen Sie **Region für Statusprüfung** auf **Westliches Nordamerika**.
5. Setzen Sie **Ursprungsname** auf **us-cluster**.
6. Setzen Sie **Ursprungsadresse** auf die Ingress-Unterdomäne des Clusters in Dallas, z. B. *my_us_cluster.us-south.containers.appdomain.cloud*.
7. Klicken Sie auf **1 Instanz bereitstellen**.

#### Ein Pool mit beiden Clustern
1. Klicken Sie auf **Pool erstellen**.
1. Setzen Sie **Name** auf **Alle**.
1. Setzen Sie die Option **Statusprüfung** auf die im vorherigen Abschnitt erstellte Prüfung.
1. Setzen Sie **Region für Statusprüfung** auf **Östliches Nordamerika**.
1. Fügen Sie zwei Ursprünge hinzu:
   1. einen, für den der **Ursprungsname** auf **us-cluster** und die **Ursprungsadresse** auf die Ingress-Unterdomäne des Clusters in Dallas gesetzt ist
   2. einen, für den der **Ursprungsname** auf **uk-cluster** und die **Ursprungsadresse** auf die Ingress-Unterdomäne des Clusters in London gesetzt ist
2. Klicken Sie auf **1 Instanz bereitstellen**.

### Globale Lastausgleichsfunktion erstellen

Wenn die ursprünglichen Pools definiert sind, können Sie die Konfiguration der Lastausgleichsfunktion abschließen.

1. Klicken Sie auf **Lastausgleichsfunktion erstellen**.
1. Geben Sie unter **Hostname des Lastausgleichs** einen Namen für die globalen Lastausgleichsfunktion ein. Dieser Name wird auch unabhängig vom Standort Teil Ihrer universellen Anwendungs-URL (`http://<glb_name>.<your_domain_name>`).
1. Klicken Sie unter **Standardursprungspools** auf **Pool hinzufügen** und fügen Sie den Pool mit dem Namen **Alle** hinzu.
1. Erweitern Sie den Abschnitt **Geo-Routen konfigurieren (optional)**:
   1. Klicken Sie auf **Route hinzufügen**, wählen Sie **Westeuropa** aus und klicken Sie auf **Hinzufügen**.
   1. Klicken Sie auf **Pool hinzufügen**, um den Pool **UK** auszuwählen.
   1. Konfigurieren Sie zusätzliche Routen, wie in der folgenden Tabelle gezeigt.
   1. Klicken Sie auf **1 Instanz bereitstellen**.

| Region               | Ursprungspool |
| :---------------:    | :---------: |
|Westeuropa            |     UK      |
|Osteuropa             |     UK      |
|Nordostasien          |     UK      |
|Südostasien           |     UK      |
|Westliches Nordamerika|     US      |
|Östliches Nordamerika |     US      |

Mit dieser Konfiguration werden die Benutzer in Europa und Asien an den Cluster in London, Benutzer in den USA an den Cluster in Dallas weitergeleitet. Wenn eine Anforderung keiner der definierten Routen entspricht, wird sie an die **Standardursprungspools** weitergeleitet.

### Ingress-Ressource für Kubernetes-Cluster pro Standort erstellen

Die globale Lastausgleichsfunktion ist nun für die Verarbeitung von Anforderungen bereit. Alle Statusprüfungen sollten grün sein. Es gibt jedoch einen letzten Konfigurationsschritt, der in den Kubernetes-Clustern erforderlich ist, um Anforderungen von der globalen Lastausgleichsfunktion (GLB) korrekt zu beantworten: Sie müssen eine Ingress-Ressource definieren, um Anforderungen aus der GLB-Domäne zu verarbeiten.

1. Erstellen Sie eine Ingress-Ressourcendatei mit dem Namen **glb-ingress.yaml**
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: glb-ingress
   spec:
    rules:
      - host: <glb_name>.<your_domain_name>
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-world-service
              servicePort: 80
   ```
    {: pre}
    Ersetzen Sie `<glb_name>.<your_domain_name>` durch die URL, die Sie im vorherigen Abschnitt definiert haben.
1. Implementieren Sie diese Ressource in den Clustern in London und Dallas, nachdem Sie die Variable KUBECONFIG für den Cluster am jeweiligen Standort festgelegt haben:
   ```bash
   kubectl create -f glb-ingress.yaml
   ```
   {: pre}
   Es wird die Nachricht `ingress.extension "glb-ingress" created` ausgegeben.

An diesem Punkt haben Sie erfolgreich eine globale Lastausgleichsfunktion mit Kubernetes-Clustern an mehreren Standorten konfiguriert. Sie können auf die URL `http://<glb_name>.<your_domain_name>` der globalen Lastausgleichsfunktion zugreifen, um Ihre Anwendung anzuzeigen. Basierend auf Ihrem Standort werden Sie an den am nächsten gelegenen Cluster weitergeleitet (oder an einen Cluster aus dem Standardpool, wenn CIS Ihre IP-Adresse keinem bestimmten Standort zuordnen konnte).

## Anwendung sichern
{: #secure_via_CIS}

### Web Application Firewall aktivieren

Web Application Firewall (WAF) schützt Ihre Webanwendung vor ISO Layer-7-Angriffen. In der Regel wird die Firewall mit gruppierten Regelsätzen kombiniert, die darauf abzielen, Schutz vor Schwachstellen in der Anwendung zu bieten, indem schädlicher Datenverkehr herausgefiltert wird.

1. Navigieren Sie im Cloud Internet Services-Dashboard zu **Sicherheit** und klicken Sie dann auf die Registerkarte **Verwalten**.
1. Stellen Sie im Abschnitt **Web Application Firewall** sicher, dass WAF aktiviert ist.
1. Klicken Sie auf **OWASP-Regelsatz anzeigen**. Auf dieser Seite können Sie den **OWASP-Kernregelsatz** überprüfen und die Regeln einzeln aktivieren oder inaktivieren. Wenn eine Regel aktiviert ist und eine eingehende Anforderung die Regel auslöst, wird die globale Bedrohungsquote erhöht. Die Einstellung für die **Sensitivität** legt fest, ob eine **Aktion** für die Anforderung ausgelöst wird.
   1. Übernehmen Sie die OWASP-Standardregelsätze.
   1. Legen Sie **Sensitivität** auf `Niedrig` fest.
   1. Setzen Sie **Aktion** auf `Simulieren`, um alle Ereignisse zu protokollieren.
1. Klicken Sie auf **Zurück zur Sicherheit**.
1. Klicken Sie auf **CIS-Regelsatz anzeigen**. Auf dieser Seite werden zusätzliche Regeln angezeigt, die im Rahmen allgemeiner Technologiestacks zum Hosten von Websites erstellt wurden.

### Höhere Leistung und Schutz vor Denial-of-Service-Attacken 
{: #proxy_setting}

Eine Distributed-Denial-of-Service-Attacke ([DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)) ist ein mutwilliger Versuch, den normalen Datenverkehr eines Servers, Service oder Netzes zu unterbrechen, indem das Ziel oder die umgebende Infrastruktur mit einer Flut von Internetdatenverkehr überlastet wird. Mit CIS können Sie Ihre Domäne vor DDoS schützen.

1. Wählen Sie im CIS-Dashboard **Zuverlässigkeit** > **Globale Lastausgleichsfunktion** aus.
1. Suchen Sie die von Ihnen erstellte globale Lastausgleichsfunktion in der Tabelle mit den **Lastausgleichsfunktionen**.
1. Aktivieren Sie in der Spalte **Proxy** die Sicherheits- und Leistungsfunktionen:

   ![Aktivierte Umschaltfunktion für CIS-Proxy](images/solution32-multi-region-k8s-cis/cis-proxy.png)

**Ihre globale Lastausgleichsfunktion ist jetzt geschützt**. Ein unmittelbarer Vorteil ist, dass die Ursprungs-IP-Adresse Ihrer Cluster für die Clients nicht sichtbar ist. Wenn CIS eine Bedrohung für eine bevorstehende Anforderung erkennt, wird der Benutzer möglicherweise eine Anzeige wie diese sehen, bevor er an Ihre Anwendung umgeleitet wird:

   ![Überprüfung - DDoS-Schutz](images/solution32-multi-region-k8s-cis/cis-DDoS.png)

Darüber hinaus können Sie jetzt steuern, welche Inhalte von CIS im Cache gespeichert werden und wie lange diese dort verbleiben. Rufen Sie **Leistung** > **Caching** auf, um die globale Cachingebene und das Ablaufdatum des Browsers zu definieren. Sie können die globalen Sicherheits- und Caching-Regeln mit **Seitenregeln** anpassen. Seitenregeln ermöglichen mithilfe bestimmter Domänenpfade eine differenzierte Konfiguration. Als Beispiel für Seitenregeln könnten Sie sich entscheiden, den gesamten Inhalt unter **/assets** für **3 Tage** im Cache zu speichern:

   ![Seitenregeln](images/solution32-multi-region-k8s-cis/cis-pagerules.png)

## Ressourcen entfernen
{:removeresources}

### Kubernetes-Clusterressourcen entfernen
1. Entfernen Sie die Ingress-Instanz.
1. Entfernen Sie den Service.
1. Entfernen Sie die Implementierung.
1. Löschen Sie die Cluster, wenn Sie sie speziell für dieses Lernprogramm erstellt haben.

### CIS-Ressourcen entfernen
1. Entfernen Sie die globale Lastausgleichsfunktion.
1. Entfernen Sie die Ursprungspools.
1. Entfernen Sie die Statusprüfungen.

## Zugehöriger Inhalt
{:related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [IBM CIS für optimale Sicherheit verwalten](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#best-practice-2-configure-your-security-level-selectively)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
* [{{site.data.keyword.registrylong_notm}} Basic](https://{DomainName}/docs/services/Registry?topic=registry-registry_overview#registry_planning)
* [Einzelne Instanz-Apps in Kubernetes-Clustern implementieren](https://{DomainName}/docs/containers?topic=containers-cs_apps_tutorial#cs_apps_tutorial_lesson1)
* [Bewährte Verfahren für die Sicherung von Datenverkehr und Internetanwendungen über CIS](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#manage-your-ibm-cis-for-optimal-security)
* [App-Verfügbarkeit mit Clustern in mehreren Zonen verbessern](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
