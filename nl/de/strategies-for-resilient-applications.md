---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-11"
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

# Strategien für ausfallsichere Anwendungen
{: #strategies-for-resilient-applications}

Unabhängig von der Berechnungsoption, Kubernetes, Cloud Foundry, Cloud Functions oder Virtual Servers versuchen Unternehmen, die Ausfallzeiten zu minimieren und ausfallsichere Architekturen einzurichten, die maximale Verfügbarkeit ermöglichen. Im vorliegenden Lernprogramm werden die Funktionen von IBM Cloud zum Erstellen ausfallsicherer Lösungen behandelt und dabei die folgenden Fragen beantwortet:

- Was ist beim Vorbereiten einer global verfügbaren Lösung zu beachten?
- Wie können die verfügbaren Berechnungsoptionen die Bereitstellung von Anwendungen in mehreren Regionen unterstützen?
- Wie kann ich Anwendungs- oder Serviceartefakte in zusätzliche Regionen importieren?
- Wie können Datenbanken standortübergreifend repliziert werden?
- Welche Unterstützungsservicees sollten verwendet werden: Blockspeicher, Dateispeicher, Objektspeicher oder Datenbanken?
- Sind servicespezifische Aspekte zu beachten?

## Ziele
{: #objectives}

* Architekturkonzepte für die Erstellung ausfallsicherer Anwendungen kennenlernen
* Erfahren, wie solche Konzepte in IBM Cloud-Computing- und -Serviceangebote umgesetzt werden können

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur und Konzepte

{: #architecture}

Beim Entwickeln einer ausfallsicheren Architektur müssen Sie die einzelnen Blöcke Ihrer Lösung und die zugehörigen Funktionen berücksichtigen. 

Die nachfolgende Architektur mit mehreren Regionen veranschaulicht, welche verschiedenen Komponenten in einer Konfiguration für mehrere Regionen vorkommen können. ![Architektur](images/solution39/Architecture.png)

Das obige Architekturdiagramm kann je nach Berechnungsoption variieren. In den weiteren Abschnitten finden Sie unter jeder Berechnungsoption spezielle Architekturdiagramme. 

### Disaster-Recovery mit zwei Regionen 

Zur Erleichterung der Disaster-Recovery (Notfallwiederherstellung) werden zwei häufig genutzte Architekturen verwendet: **Aktiv/Aktiv** und **Aktiv/Passiv**. Jede Architektur weist spezielle Vor- und Nachteile in Bezug auf Zeit und Aufwand bei der Wiederherstellung auf.

#### Aktiv/Aktiv-Konfiguration

In einer Aktiv/Aktiv-Architektur verfügen beide Standorte über identische aktive Instanzen und eine Lastausgleichsfunktion verteilt den Datenverkehr zwischen ihnen. Bei diesem Ansatz muss die Datenreplikation eingerichtet sein, um die Daten zwischen den beiden Regionen in Echtzeit zu synchronisieren.

![Aktiv/Aktiv](images/solution39/Active-active.png)

Diese Konfiguration bietet höhere Verfügbarkeit und einen geringeren manuellen Korrekturaufwand als eine Aktiv/Passiv-Architektur. Anforderungen werden von beiden Rechenzentren bedient. Sie sollten die Edge-Services (Lastausgleichsfunktion) mit der entsprechenden Logik für Zeitlimits und Neuversuche ausstatten, damit die Anforderung automatisch an das zweite Rechenzentrum weitergeleitet wird, wenn in der Umgebung des ersten Rechenzentrums ein Fehler auftritt.

Im Aktiv/Aktiv-Szenario muss die Zielsetzung in Bezug auf den maximal tolerierbaren Datenverlust bei einem Ausfall (**Recovery Point Objective**, RPO) eine extrem zeitnahe Datensynchronisation zwischen den beiden aktiven Rechenzentren beinhalten, um einen reibungslosen Anforderungsablauf zu ermöglichen.

#### Aktiv/Passiv-Konfiguration

Eine Aktiv/Passiv-Architektur besteht aus einer aktiven Region und einer zweiten, passiven Region, die als Backup dient. Bei einer Betriebsunterbrechung in der aktiven Region, übernimmt die passive Region die Rolle der aktiven Region. Manuelle Eingriffe können erforderlich sein, um sicherzustellen, dass die Datenbanken oder der Dateispeicher den aktuellen Anwendungs- und Benutzeranforderungen entsprechen. 

![Aktiv/Aktiv](images/solution39/Active-passive.png)

Anforderungen werden vom aktiven Standort bedient. Bei einer Betriebsunterbrechung oder einem Anwendungsfehler werden vorbereitende Operationen ausgeführt, damit das Standby-Rechenzentrum bereit ist, die Anforderung zu bedienen. Das Umschalten vom aktiven zum passiven Rechenzentrum ist ein zeitaufwendiger Vorgang. Sowohl die **Recovery Time Objective** (RTO) als auch die **Recovery Point Objective** (RPO) ist höher als bei der Aktiv/Aktiv-Konfiguration.

### Disaster-Recovery mit drei Regionen

In der heutigen Zeit der ständig verfügbaren Services mit Nulltoleranz für Ausfallzeit erwarten die Kunden, dass jeder Geschäftsservice von jedem Punkt der Erde aus rund um die Uhr erreichbar ist. Eine kosteneffiziente Strategie für Unternehmen sieht vor, dass die Infrastruktur auf kontinuierliche Verfügbarkeit ausgerichtet wird, anstatt Infrastrukturen für Disaster-Recovery zu erstellen.

Drei Rechenzentren bieten eine größere Ausfallsicherheit und höhere Verfügbarkeit als zwei Rechenzentren. Dieser Ansatz kann auch die Leistung steigern, indem die Workload gleichmäßiger auf die Rechenzentren verteilt wird. Wenn das Unternehmen nur über zwei Rechenzentren verfügt, besteht eine Variante darin, zwei Anwendungen im ersten Rechenzentrum bereitzustellen und die dritte Anwendung im zweiten Rechenzentrum. Alternativ können Sie Geschäftslogik und Darstellungsebenen in der dreifach aktiven Topologie (Aktiv/Aktiv/Aktiv) bereitstellen und die Datenebene in der zweifach aktiven Topologie (Aktiv/Aktiv).

#### Aktiv/Aktiv/Aktiv-Konfiguration (dreifach aktive Konfiguration)

![](images/solution39/Active-active-active.png)

Anforderungen werden von der Anwendung bedient, die in einem der drei aktiven Rechenzentren ausgeführt wird. Eine Fallstudie auf der IBM.com-Website zeigt, dass eine dreifach aktive Konfiguration nur 50 % der Rechen-, Speicher- und Netzkapazität pro Cluster beansprucht, während die zweifach aktive Konfiguration 100 % der Kapazität pro Cluster beansprucht. Auf der Datenebene ist der Kostenunterschied am deutlichsten zu sehen. Weitere Details hierzu finden Sie in der Veröffentlichung [*Always On: Assess, Design, Implement, and Manage Continuous Availability*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf).

#### Aktiv/Aktiv/Passiv-Konfiguration

![](images/solution39/Active-active-passive.png)

Wenn in diesem Szenario eine der beiden aktiven Anwendungen im ersten und im zweiten Rechenzentrum ausfällt, wird die Standby-Anwendung im dritten Rechenzentrum aktiviert. Die im Szenario mit zwei Rechenzentren beschriebene Disaster-Recovery-Prozedur wird eingesetzt, um die normale Verarbeitung von Kundenanforderungen wiederherzustellen. Die Standby-Anwendung im dritten Rechenzentrum kann als Hot- oder Cold-Standby-Konfiguration eingerichtet werden.

Weitere Informationen zur Disaster-Recovery enthält [dieser Leitfaden](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/).

### Architekturen für mehrere Regionen

In einer Architektur mit mehreren Regionen wird eine Anwendung an verschiedenen Standorten bereitgestellt. Dabei wird in jeder Region eine identische Kopie der Anwendung ausgeführt. 

Eine Region ist eine bestimmter Standort, an dem Apps, Services und andere {{site.data.keyword.cloud_notm}}-Ressourcen bereitgestellt werden können. [{{site.data.keyword.cloud_notm}}-Regionen](https://{DomainName}/docs/containers?topic=containers-regions-and-zones) umfassen mindestens eine Zone, die aus einem physischen Rechenzentrum mit den zugehörigen Computing-, Netz- und Speicherressourcen sowie Kühlung und Stromversorgung für das Hosting von Services und Anwendungen besteht. Die Zonen sind voneinander isoliert, d. h. sie weisen keinen gemeinsamen Single Point of Failure auf.

Darüber hinaus ist in einer Architektur mit mehreren Regionen ein globale Lastausgleichsfunktion wie [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services) erforderlich, um den Datenverkehr auf die Regionen zu verteilen.

Die Bereitstellung einer Lösung in mehreren Regionen bietet folgende Vorteile:
- Verbesserte Latenz für Endbenutzer: Auf die Geschwindigkeit kommt es an. Je näher der Ausgangspunkt Ihres Back-Ends an den Endbenutzern liegt, umso besser (und schneller) die Benutzererfahrung.
- Disaster-Recovery: Wenn die aktive Region ausfällt, ermöglicht eine Backup-Region die schnelle Wiederherstellung.
- Geschäftsanforderungen: In manchen Fällen müssen Daten in entfernten Regionen gespeichert werden, die mehrere hundert Kilometer auseinander liegen. Daher müssen die Daten in einem solchen Fall in mehreren Regionen gespeichert werden. 

### Architekturen mit mehreren Zonen innerhalb der Regionen

Bei der Erstellung von Anwendungen mit mehreren regionalen Zonen muss Ihre Anwendung innerhalb einer Region zonenübergreifend bereitgestellt werden und zusätzlich können zwei oder drei Regionen vorhanden sein. 

In Architekturen mit mehreren Zonen in einer Region ist eine lokale Lastausgleichsfunktion erforderlich, die den Datenverkehr lokal auf die Zonen in einer Region verteilt. Falls eine zweite Region eingerichtet ist, muss zusätzlich eine globale Lastausgleichsfunktion den Datenverkehr auf die Regionen verteilen. 

Weitere Informationen über Regionen und Zonen [finden Sie hier](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones).

## Berechnungsoptionen 

In diesem Abschnitt werden die in {{site.data.keyword.cloud_notm}} verfügbaren Berechnungsoptionen erläutert. Für jede Berechnungsoption wird ein Architekturdiagramm bereitgestellt. Außerdem erfahren Sie in einem Lernprogramm, wie eine solche Architektur bereitgestellt wird.

Hinweis: Die dargestellten Architekturen für Berechnungsoptionen enthalten weder Datenbanken noch andere Services, sie veranschaulichen nur die Bereitstellung einer App in zwei Regionen für die ausgewählte Berechnungsoption. Nachdem Sie eines der Beispiele für regionsübergreifende Berechnungsoptionen implementiert haben, besteht der nächste Schritt darin, Datenbanken und andere Services hinzuzufügen. In späteren Abschnitten dieses Lernprogramms werden [Datenbanken](#databaseservices) und [Services ohne Datenbanken](#nondatabaseservices) behandelt.

### Cloud Foundry 

Cloud Foundry bietet die Funktionalität zum Bereitstellen einer Architektur mit mehreren Regionen. Darüber hinaus bietet die Verwendung von Services in einer [Continuous-Delivery-Pipeline](https://{DomainName}/catalog/services/continuous-delivery) Ihnen die Möglichkeit, Ihre Anwendung in mehreren Regionen bereitzustellen. Die Architektur für Cloud Foundry mit mehreren Regionen sieht so aus:

![CF-Architektur](images/solution39/CF2-Architecture.png)

Dieselbe Anwendung wird in mehreren Regionen bereitgestellt und eine globale Lastausgleichsfunktion leitet den Datenverkehr zur nächstgelegenen betriebsbereiten Region weiter. Das Lernprogramm [**Sichere Webanwendung in mehreren Regionen**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp) führt Sie durch die Bereitstellung einer ähnlichen Architektur.

### {{site.data.keyword.cfee_full_notm}}

**{{site.data.keyword.cfee_full_notm}} (CFEE)** bietet den gleichen Funktionsumfang wie eine öffentliche Cloud Foundry-Instanz und darüber hinaus zusätzliche Features.

**{{site.data.keyword.cfee_full_notm}}** ermöglicht Ihnen die Instanziierung mehrerer isolierter und auf Unternehmen abgestimmter Cloud Foundry-Plattformen je nach Bedarf. CFEE-Instanzen werden in Ihrem eigenen Konto in [{{site.data.keyword.cloud_notm}}
](http://ibm.com/cloud) ausgeführt. Die Umgebung wird auf isolierter Hardware bereitgestellt, die auf [{{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service) aufgesetzt wird. Sie haben die vollständige Kontrolle über die Umgebung, einschließlich Zugriffssteuerung, Kapazitätsmanagement, Änderungsmanagement, Überwachung und Services.

Nachfolgend ist eine Architektur mit mehreren Regionen unter Verwendung von {{site.data.keyword.cfee_full_notm}} dargestellt.

![Architektur](images/solution39/CFEE-Architecture.png)

Für die Bereitstellung dieser Architektur ist Folgendes erforderlich: 

- Zwei CFEE-Instanzen einrichten (in jeder Region eine)
- Services erstellen und an das CFEE-Konto binden 
- Apps per Push-Operation in den CFEE-API-Endpunkt übertragen 
- Datenbankreplikation einrichten (wie in einer öffentlichen Cloud Foundry-Instanz) 

Weitere Details enthält die schrittweise Anleitung [Deploy Logistics Wizard to Cloud Foundry Enterprise Environment (CFEE)](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md). Diese Anleitung führt Sie durch die Bereitstellung einer microservicebasierten Anwendung in CFEE. Nach der Bereitstellung in einer CFEE-Instanz können Sie die Prozedur auf eine zweite Region anwenden und den beiden CFEE-Instanzen [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) vorschalten, um einen Lastausgleich für den Datenverkehr zu ermöglichen. 

Weitere Details hierzu finden Sie in der [{{site.data.keyword.cfee_full_notm}}-Dokumentation](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about).

### Kubernetes

Mit Kubernetes können Sie eine Architektur mit mehreren Zonen innerhalb von Regionen realisieren. Dies kann ein Anwendungsfall für eine Aktiv/Aktiv-Konfiguration sein. Bei der Bereitstellung einer Lösung mit {{site.data.keyword.containershort_notm}} können Sie von integrierten Funktionen wie Lastausgleich und Isolierung sowie erhöhter Ausfallsicherheit bei potenziellen Fehlern in Hosts, Netzen oder Apps profitieren. Wenn mehrere Cluster erstellt wurden und ein Cluster ausfällt, können die Benutzer weiterhin auf die betroffene App zugreifen, da sie noch in einem weiteren Cluster bereitgestellt ist. Falls mehrere Cluster in verschiedenen Regionen vorhanden sind, können Benutzer außerdem den nächstgelegenen Cluster aufrufen und dadurch die Netzlatenz reduzieren. Zur Erhöhung der Ausfallsicherheit können Sie darüber hinaus die Mehrzonencluster auswählen, d. h. Ihre Knoten werden in mehreren Zonen innerhalb einer Region bereitgestellt. 

Die Kubernetes-Architektur mit mehreren Regionen sieht so aus:

![Kubernetes](images/solution39/Kub-Architecture.png)

1. Der Entwickler erstellt Docker-Images für die Anwendung.
2. Die Images werden mit einer Push-Operation an zwei verschiedene Standorte in {{site.data.keyword.registryshort_notm}} übertragen.
3. Die Anwendung wird in Kubernetes-Clustern an beiden Standorten bereitgestellt.
4. Endbenutzer rufen die Anwendung auf.
5. Cloud Internet Services wird so konfiguriert, dass Anforderungen für die Anwendung abgefangen und die Workload auf die Cluster verteilt wird. Außerdem werden DDoS-Schutz und Web Application Firewall aktiviert, um die Anwendung vor Bedrohungen zu schützen. Assets wie Bilder und CSS-Dateien können optional zwischengespeichert werden.

Das Lernprogramm [**Ausfallsichere und geschützte Kubernetes-Cluster in mehreren Regionen mit Cloud Internet Services**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis) führt Sie schrittweise durch die Bereitstellung einer solchen Architektur.

### {{site.data.keyword.openwhisk_short}}

{{site.data.keyword.openwhisk_short}} ist an mehreren {{site.data.keyword.cloud_notm}}-Standorten verfügbar. Um die Ausfallsicherheit zu erhöhen und die Netzlatenz zu verringern, können Anwendungen das zugehörige Back-End an mehreren Standorten bereitstellen. Anschließend können Entwickler mit IBM Cloud Internet Services (CIS) einen einzigen Eingangspunkt bereitstellen, der für die Verteilung des Datenverkehrs auf das nächste einwandfreie Back-End zuständig ist.Die Architektur für {{site.data.keyword.openwhisk_short}} in mehreren Regionen sieht so aus:

 ![Functions-Architektur](images/solution39/Functions-Architecture.png)

1. Benutzer greifen auf die Anwendung zu. Die Anforderung wird über Internet Services geleitet.
2. Internet Services leitet die Benutzer zum nächstgelegenen betriebsbereiten API-Back-End um.
3. Certificate Manager stellt die API mit dem zugehörigen SSL-Zertifikat bereit. Der Datenverkehr wird durchgängig verschlüsselt.
4. Die API wird mit Cloud Functions bereitgestellt.

Eine Anleitung zum Implementieren dieser Architektur bietet das Lernprogramm [**Serverunabhängige Apps in mehreren Regionen bereitstellen**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless).

### {{site.data.keyword.baremetal_short}} und {{site.data.keyword.virtualmachinesshort}}

{{site.data.keyword.virtualmachinesshort}} und {{site.data.keyword.baremetal_short}} stellen die Funktionalität zum Einrichten einer Architektur mit mehreren Regionen bereit. Sie können in {{site.data.keyword.cloud_notm}} Server an vielen verfügbaren Standorten bereitstellen.

![Serverstandorte](images/solution39/ServersLocation.png)

Berücksichtigen Sie bei der Vorbereitung einer solchen Architektur mit {{site.data.keyword.virtualmachinesshort}}n und {{site.data.keyword.baremetal_short}}n folgende Aspekte: Dateispeicher, Backups, Wiederherstellung und Datenbanken (wählen Sie zwischen Database as a Service (DBaaS) und der Installation einer Datenbank auf einem virtuellen Server). 

Die folgende Architektur veranschaulicht die Bereitstellung einer Architektur mit mehreren Regionen unter Verwendung von {{site.data.keyword.virtualmachinesshort}}n in einer Aktiv/Passiv-Architektur, in der die erste Region aktiv ist und die zweite Region passiv. 

![VM-Architektur](images/solution39/vm-Architecture2.png)

Folgende Komponenten sind für eine solche Architektur erforderlich: 

1. Benutzer greifen über IBM Cloud Internet Services (CIS) auf die Anwendung zu.
2. CIS leitet den Datenverkehr an den aktiven Standort weiter.
3. Innerhalb eines Standorts leitet eine Lastausgleichsfunktion den Datenverkehr zu einem Server um.
4. Datenbanken werden auf einem virtuellen Server bereitgestellt, d. h. Sie konfigurieren die Datenbank und richten Replikationen zwischen Regionen ein. Alternativ können Sie Database as a Service verwenden. Diese Möglichkeit wird weiter unten im vorliegenden Lernprogramm erläutert.
5. Dateispeicher für die Anwendungsimages und -dateien. Der Dateispeicher ermöglicht das Erstellen eines Snapshots an einem bestimmten Zeitpunkt, der danach in einer anderen Region wiederverwendet werden kann. Dieser Vorgang müsste sonst manuell ausgeführt werden. 

Im Lernprogramm [**Mit Virtual Servers eine hochverfügbare und skalierbare Web-App erstellen**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application) wird die Bereitstellung dieser Architektur erläutert.

## Datenbanken und Anwendungsdateien
{: #databaseservices}

{{site.data.keyword.cloud_notm}} bietet eine Auswahl von [Database as a Service](https://{DomainName}/catalog/?category=databases)-Instanzen mit relationalen oder nicht relationalen Datenbanken, gemäß Ihren Geschäftsanforderungen. [Database as a Service (DBaaS)](https://www.ibm.com/cloud/learn/what-is-cloud-database) bietet zahlreiche Vorteile. Mit einer DBaaS-Instanz wie {{site.data.keyword.cloudant}} können Sie die Vorteile der regionsübergreifenden Unterstützung nutzen, die es Ihnen ermöglicht, Live-Replikationen zwischen zwei Datenbankservices in verschiedenen Regionen durchzuführen, Backups zu erstellen sowie Skalierungen vorzunehmen und die Betriebszeit zu maximieren. 

**Hauptkomponenten** 

- Ein Datenbankservice, der über eine Cloud-Plattform erstellt und aufgerufen wird.
- Ermöglicht Benutzern in Unternehmen das Hosten von Datenbanken, ohne den Erwerb dedizierter Hardware.
- Kann vom Benutzer verwaltet oder als Service angeboten und von einem Provider verwaltet werden.
- Kann SQL- oder NoSQL-Datenbanken unterstützen.
- Wird über eine Webschnittstelle oder über die API eines Anbieters aufgerufen.

**Architektur mit mehreren Regionen vorbereiten:**

- Welche Optionen für Ausfallsicherheit bietet der Datenbankservice?
- Wie wird die Replikation zwischen mehreren Datenbankservices regionsübergreifend abgewickelt?
- Wie werden die Daten gesichert?
- Welche Ansätze für die Disaster-Recovery sind möglich?

### {{site.data.keyword.cloudant}}

{{site.data.keyword.cloudant}} ist eine verteilte Datenbank, die auf die Verarbeitung großer Workloads abgestimmt ist, wie sie für umfangreiche und schnell wachsende Web-Apps und mobile Apps typisch sind. Als SLA-gestützter und vollständig verwalteter {{site.data.keyword.Bluemix_notm}}-Service ermöglicht {{site.data.keyword.cloudant}} die flexible und unabhängige Skalierung von Durchsatz und Speicher. Außerdem ist {{site.data.keyword.cloudant}} als lokale Installation für den Download verfügbar und die zugehörige API sowie das leistungsfähige Replikationsprotokoll sind mit einem Open Source-Geschäftsumfeld kompatibel, das CouchDB, PouchDB sowie Bibliotheken für die gängigsten Entwicklungsstacks für Web-Apps und mobile Apps enthält.

{{site.data.keyword.cloudant}} unterstützt die standortübergreifende [Replikation](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation) zwischen mehreren Instanzen. Jede in der Quellendatenbank vorgenommene Änderung wird in die Zieldatenbank übernommen. Sie können Replikationen (entweder als fortlaufende oder als einmalige Task) zwischen einer beliebigen Anzahl von Datenbanken erstellen. Das folgende Diagramm zeigt eine typische Konfiguration mit zwei {{site.data.keyword.cloudant}}-Instanzen (eine in jeder Region):

![Aktiv/Aktiv](images/solution39/Active-active.png)

Weitere Informationen zum Konfigurieren der Replikation zwischen {{site.data.keyword.cloudant}}-Instanzen finden Sie [in diesen Anweisungen](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery). Der Service stellt auch Anleitungen und Tools zur [Sicherung und Wiederherstellung von Daten](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery) bereit.

### {{site.data.keyword.Db2_on_Cloud_short}}, {{site.data.keyword.dashdbshort_notm}} und {{site.data.keyword.Db2Hosted_notm}}

{{site.data.keyword.cloud_notm}} stellt verschiedene [Db2-Datenbankservices](https://{DomainName}/catalog/?search=db2h) zur Verfügung. Diese Services sind:

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2): Eine vollständig verwaltete Cloud-SQL-Datenbank für typische, OLTP-ähnliche operative Workloads.
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse): Ein vollständig verwalteter, cloudbasierter Data Warehouse-Service für Analyseworkloads im Petabyte-Bereich mit hohem Leistungsbedarf. Er bietet sowohl SMP- als auch MPP-Servicepläne und nutzt einen optimierten Speicher für Spaltendaten sowie die speicherinterne Verwarbeitung.
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted): Ein Datenbanksystem, das von IBM gehostet und vom Benutzer verwaltet wird. Das Datenbanksystem ermöglicht Db2 uneingeschränkten Verwaltungszugriff auf die Cloudinfrastruktur und erspart damit Kosten, Komplexität und Risiko, die bei der Verwaltung einer eigenen Infrastruktur entstehen würden.

Im Folgenden wird die Verwendung von {{site.data.keyword.Db2_on_Cloud_short}} als DBaaS für operative Workloads behandelt. Solche Workloads sind typisch für die Anwendungen, um die es in diesem Lernprogramm geht.

#### Regionsübergreifende Unterstützung für {{site.data.keyword.Db2_on_Cloud_short}}

{{site.data.keyword.Db2_on_Cloud_short}} bietet mehrere [Optionen zum Realisieren von Hochverfügbarkeit und Disaster-Recovery](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview). Beim Erstellen eines neuen Service können Sie die Option für Hochverfügbarkeit auswählen. Später können Sie über das Dashboard der Instanz [Disaster-Recovery-Knoten mit geografischer Replikation hinzufügen](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha). Die Option für ausgelagerte Disaster-Recovery-Knoten ermöglicht die Synchronisation Ihrer Daten in Echtzeit mit einem Datenbankknoten in einem {{site.data.keyword.cloud_notm}}-Rechenzentrum an einem beliebigen anderen Standort Ihrer Wahl.

Weitere Informationen enthält die [Dokumentation für Hochverfügbarkeit](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha).

#### Sicherung und Wiederherstellung

{{site.data.keyword.Db2_on_Cloud_short}} beinhaltet tägliche Sicherungen für kostenpflichtige Pläne. Die Sicherungen werden normalerweise über {{site.data.keyword.cos_short}} gespeichert und nutzen drei Rechenzentren, um eine bessere Verfügbarkeit der gespeicherten Daten zu gewährleisten. Die Sicherungen werden 14 Tage lang aufbewahrt. Sie können für die Durchführung einer punktuellen Wiederherstellung verwendet werden. Die [Dokumentation über Sicherung und Wiederherstellung](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br) enthält Details dazu, wie Sie Daten für einen bestimmten Zeitpunkt (Datum und Uhrzeit) wiederherstellen können.

### {{site.data.keyword.databases-for}}

{{site.data.keyword.databases-for}} stellt mehrere Open-Sorce-Datenbanksysteme als vollständig verwaltete Services bereit. Diese Datenbanksysteme sind:  
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

Alle diese Services weisen die gleichen Merkmale auf:   
* Für hohe Verfügbarkeit werden sie in Clustern bereitgestellt. Weitere Informationen finden Sie in der Dokumentation der einzelnen Services:
  - [PostgreSQL](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* Jeder Cluster wird auf mehrere Zonen verteilt.
* Daten werden zonenübergreifend repliziert.
* Benutzer können die Speicherkapazität und die Speicherressourcen für eine Instanz skalieren. Details hierzu finden Sie in der [Dokumentation über die Skalierung z. B. für {{site.data.keyword.databases-for-redis}}](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings).
* Sicherungen werden täglich oder auf Anforderung erstellt. Details sind für die einzelnen Services dokumentiert. Beispiel: die [Dokumentation über die Sicherung für {{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups).
* Ruhende Daten, Sicherungen und Netzdatenverkehr sind verschlüsselt.
* Jeder [Service kann mit dem {{site.data.keyword.databases-for}}-CLI-Plug-in](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in) verwaltet werden.


### {{site.data.keyword.cos_full_notm}}

{{site.data.keyword.cos_full_notm}} (COS) stellt dauerhaften, sicheren und kosteneffizienten Cloudspeicher zur Verfügung. Mit {{site.data.keyword.cos_full_notm}} gespeicherte Informationen werden verschlüsselt und auf mehrere Standorte verteilt. Beim Erstellen von Speicherbuckets in einer COS-Instanz können Sie festlegen, an welchem Standort der Bucket erstellt wird und welche Option für Ausfallsicherheit verwendet werden soll.

Die folgenden drei Typen der Ausfallsicherheit für Buckets sind verfügbar:
   - Bei der **regionsübergreifenden** Ausfallsicherheit werden Ihre Daten auf mehrere städtische Ballungsräume verteilt. Dies kann auch als 'Option für mehrere Regionen' bezeichnet werden. Für den Zugriff auf Inhalte, die in einem regionsübergreifenden Bucket gespeichert sind, stellt COS einen speziellen Endpunkt bereit, der Inhalte aus einer betriebsbereiten Region ermöglicht.
   - Bei der **regionalen** Ausfallsicherheit werden Daten auf einen einzigen städtischen Ballungsraum verteilt. Dies kann auch als 'mehrere Zonen innerhalb einer Region' bezeichnet werden.
   - Bei der Verwendung eines **einzelnen Rechenzentrums** für die Ausfallsicherheit werden Daten auf mehrere Appliances in einem einzigen Rechenzentrum verteilt.

In [dieser Dokumentation](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints) finden Sie eine ausführliche Erläuterung der {{site.data.keyword.cos_full_notm}}-Optionen für Ausfallsicherheit.

### {{site.data.keyword.filestorage_full_notm}}

{{site.data.keyword.filestorage_full_notm}} ist ein persistenter, schneller und flexibler, NFS-basierter NAS-Dateispeicher. In dieser NAS-Umgebung (NAS = Network-Attached Storage) haben Sie die vollständige Kontrolle über die Funktion und Leistung Ihrer Dateifreigaben. {{site.data.keyword.filestorage_short}}-Freigaben können über TCP/IP-Routingverbindungen für bis zu 64 berechtigte Geräte verfügbar gemacht werden, um die Ausfallsicherheit sicherzustellen.

Einige der Dateispeicherfunktionen sind _Snapshot_, _Replikation_ und _Gleichzeitiger Zugriff_. Eine vollständige Liste der Funktionen finden Sie [in der Dokumentation](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage).

Ein an Ihre Server angehängter {{site.data.keyword.filestorage_short}}-Service kann ohne großen Aufwand zum Speichern von Datensicherungen und Anwendungsdateien (z. B. Bilder und Videos) verwendet werden. Diese Bilder und Dateien können anschließend für verschiedene Server in derselben Region genutzt werden.

Beim Hinzufügen einer zweiten Region können Sie mit der Snapshotfunktion von {{site.data.keyword.filestorage_short}} einen Snapshot automatisch oder manuell erstellen und anschließend in der zweiten (passiven) Region wiederverwenden. 

Die Replikation kann so geplant werden, dass Snapshots automatisch auf einen Zieldatenträger in einem fernen Rechenzentrum kopiert werden. Diese Kopien können an einem fernen Standort wiederhergestellt werden, wenn ein katastrophaler Ausfall eintritt oder Ihre Daten beschädigt werden. Weitere Informationen zu File Storage-Snapshots finden Sie [hier](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots).

## Services ohne Datenbanken
{: #nondatabaseservices}

{{site.data.keyword.cloud_notm}} stellt eine Auswahl von [Services](https://{DomainName}/catalog) ohne Datenbanken bereit. Dazu gehören sowohl IBM Services als auch Services anderer Anbieter. Bei der Planung einer Architektur mit mehreren Regionen müssen Sie wissen, wie Services (z. B. Watson-Services) in einer Konfiguration mit mehreren Regionen funktionieren.

### {{site.data.keyword.conversationfull}}

Die Plattform [{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about) ermöglicht Entwicklern und Benutzern ohne technisches Knowhow, gemeinsam an der Erstellung von interaktiven, KI-basierten Assistenten zu arbeiten.

Ein Assistent ist ein kognitiver Bot, den Sie an Ihre Geschäftsanforderungen anpassen und über mehrere Kanäle bereitstellen können, um Ihre Kunden zeit- und ortsnah durch Hilfeinformationen zu unterstützen. Der Assistent enthält mindestens einen Skill. Ein Dialogskill enthält die Trainingsdaten und die Logik, die es einem Assistenten ermöglichen, Ihren Kunden zu helfen.

Beachten Sie, dass {{site.data.keyword.conversationshort}} V1 statusunabhängig ist. {{site.data.keyword.conversationshort}} stellt eine Betriebszeit von 99,5 % bereit. Für Anwendungen mit besonders hoher Verfügbarkeit in mehreren Regionen kann es dennoch sinnvoll sein, mehrere Instanzen dieser Services regionsübergreifend bereitzustellen. 

Nachdem Sie Instanzen an mehreren Standorten erstellt haben, können Sie mit den {{site.data.keyword.conversationshort}}-Tools einen vorhandenen Arbeitsbereich (einschließlich Absichten, Entitäten und Dialogmodul) aus einer Instanz exportieren und anschließend in andere Standorte importieren.

## Übersicht

| Angebot | Optionen für Ausfallsicherheit |
| -------- | ------------------ |
| Cloud Foundry | <ul><li>Anwendungen an mehreren Standorten bereitstellen</li><li>Anforderungen von mehreren Standorten mit Cloud Internet Services bedienen</li><li>Mit Cloud Foundry-APIs Organisationen und Bereiche konfigurieren und Apps per Push-Operation an mehrere Standorte übertragen</li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>Anwendungen an mehreren Standorten bereitstellen</li><li>Anforderungen von mehreren Standorten mit Cloud Internet Services bedienen</li><li>Mit Cloud Foundry-APIs Organisationen und Bereiche konfigurieren und Apps per Push-Operation an mehrere Standorte übertragen</li><li>Basiert auf Kubernetes-Service</li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>Für Ausfallsicherheit konzipiert; mit Unterstützung für Cluster mit mehreren Zonen</li><li>Anforderungen von Clustern, die über mehrere Standorte verteilt sind, mit Cloud Internet Services bedienen</li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>An mehreren Standorten verfügbar</li><li>Anforderungen von mehreren Standorten mit Cloud Internet Services bedienen</li><li>Mit der Cloud Functions-API Aktionen an mehreren Standorten bereitstellen</li></ul> |
| {{site.data.keyword.baremetal_short}} und {{site.data.keyword.virtualmachinesshort}} | <ul><li>Server an mehreren Standorten bereitstellen</li><li>Server am gleichen Standort an eine lokale Lastausgleichsfunktion anbinden</li><li>Anforderungen von mehreren Standorten mit Cloud Internet Services bedienen</li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>Einmalreplikation und fortlaufende Replikation zwischen Datenbanken</li><li>Automatische Datenredundanz innerhalb einer Region</li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>Disaster-Recovery-Knoten mit geografischer Replikation für die Synchronisation von Echtzeitdaten bereitstellen</li><li>Tägliche Sicherungen in kostenpflichtigen Plänen</li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>Basiert auf Kubernetes-Clustern mit mehreren Zonen</li><li>Regionsübergreifende Lesereplikate</li><li>Tägliche und bedarfsgesteuerte Sicherungen</li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>Einzelnes Rechenzentrum für Ausfallsicherheit, regionale und regionsübergreifende Ausfallsicherheit</li><li>Mithilfe einer API Inhalte zwischen mehreren Speicherbuckets synchronisieren</li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>Mit Snapshots Inhalte automatisch an einem Bestimmungsort in einem fernen Rechenzentrum erfassen</li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>Mit Watson-API die Arbeitsbereichsspezifikation zwischen mehreren Instanzen standortübergreifend exportieren und importieren</li></ul> |

## Zugehörige Inhalte

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [App-Verfügbarkeit durch Cluster mit mehreren Zonen verbessern](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry - sichere Webanwendung in mehreren Regionen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [Cloud Functions - serverunabhängige Apps in mehreren Regionen bereitstellen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes - ausfallsichere und geschützte Kubernetes-Cluster in mehreren Regionen durch Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [Virtual Servers - hochverfügbare und skalierbare Web-App erstellen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
