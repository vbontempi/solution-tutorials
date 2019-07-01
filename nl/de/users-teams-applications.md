---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# Bewährte Verfahren für die Organisation von Benutzern, Teams und Anwendungen
{: #users-teams-applications}

In diesem Lernprogramm erhalten Sie eine Übersicht über die in {{site.data.keyword.cloud_notm}} verfügbaren Konzepte zur Verwaltung des Identitäts- und Zugriffsmanagements sowie zu ihrer Implementierung bei der Unterstützung der verschiedenen Entwicklungsphasen einer Anwendung.
{:shortdesc}

Bei der Erstellung einer Anwendung ist es gängige Praxis, mehrere Umgebungen zu definieren, die den Entwicklungszyklus eines Projekts vom Entwickler, der Code beisteuert, bis hin zum Anwendungscode, der für Endbenutzer verfügbar gemacht wird, abbilden. Solche Umgebungen haben typischerweise Namen wie *Sandbox*, *Test*, *Staging*, *Benutzerakzeptanztest* (UAT - User Acceptance Testing), *Vorproduktion*, *Produktion* u. ä.

Diese separaten Umgebungen dienen verschiedenen Zwecken, wie zum Beispiel dazu, die zugrunde liegenden Ressourcen zu isolieren, Governance- und Zugriffsrichtlinien zu implementieren, den Produktionsbetrieb zu schützen und Änderungen vor der Einführung in die Produktion zu validieren.

## Lernziele
{: #objectives}

* {{site.data.keyword.iamlong}} und Cloud Foundry-Zugriffsmodelle kennen lernen
* Projekt mit Separation zwischen Rollen und Umgebungen konfigurieren
* Kontinuierliche Integration einrichten

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet: 
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um einen Kostenvoranschlag auf der Grundlage der von Ihnen projektierten Nutzung zu generieren.

## Projekt definieren

Betrachten Sie ein Beispielprojekt mit den folgenden Komponenten:
* Mehrere in {{site.data.keyword.containershort_notm}} bereitgestellte Microservices
* Datenbanken
* Dateispeicherbuckets

In diesem Projekt werden drei Umgebungen definiert:
* *Development* - Diese Umgebung wird kontinuierlich mit jedem Codebeitrag (Commit), jedem durchgeführten Komponententest und jedem durchgeführten Smoketest aktualisiert. Sie ermöglicht den Zugriff auf die zuletzt erfolgte und größte Bereitstellung des Projekts.
* *Testing* - Diese Umgebung wird erstellt, wenn ein stabiler Zweig oder Tag des Codes erreicht wurde. In dieser Umgebung werden die Benutzerakzeptanztests durchgeführt. Die Konfiguration dieser Umgebung ist der Produktionsumgebung ähnlich. Sie wird mit realistischen Daten (z. B. anonymisierten Produktionsdaten) geladen.
* *Production* - Diese Umgebung wird mit der in der vorherigen Umgebung validierten Version aktualisiert.

**Eine Delivery Pipeline verwaltet den Fortschritt eines Builds durch die Umgebung.** Diese kann voll automatisiert werden oder manuelle Validierungspunkte einschließen, um Builds zwischen Umgebungen zu genehmigen. Dieses Konzept ist wirklich offen und sollte auf die bewährten Verfahren und Arbeitsabläufe des Unternehmens abgestimmt werden.

Um die Ausführung der Build-Pipeline zu unterstützen, wird ein **funktionaler Benutzer** eingeführt, d. h. ein regulärer {{site.data.keyword.cloud_notm}}-Benutzer, bei dem es sich jedoch um ein Teammitglied handelt, das keine reale Identität in der physischen Welt besitzt. Dieser funktionale Benutzer ist wird als Eigner der Delivery Pipelines sowie aller anderen Cloudressourcen eingesetzt, die strenge Eigentumsrechte erfordern. Dieser Ansatz ist für den Fall hilfreich, dass ein Teammitglied das Unternehmen verlässt oder in ein anderes Projekt versetzt wird. Der funktionale Benutzer wird für Ihr Projekt dediziert und über die Laufzeit des Projekts nicht geändert. Das nächste Element, das erstellt werden sollte, ist ein [API-Schlüssel](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey) für diesen funktionalen Benutzer. Sie wählen diesen API-Schlüssel aus, wenn Sie DevOps-Pipelines einrichten oder wenn Sie Automationsscripts ausführen wollen, um die Identität dieses funktionalen Benutzers anzunehmen.

Im Hinblick auf die Verteilung der Zuständigkeiten an Projektteammitglieder sollen die folgenden Rollen und zugehörigen Berechtigungen definiert werden:

|           | Entwicklung | Test | Produktion |
| --------- | ----------- | ------- | ---------- |
| Entwickler | <ul><li>Steuert Code bei.</li><li>Kann auf Protokolldateien zugreifen.</li><li>Kann die App- und Servicekonfiguration anzeigen.</li><li>Kann die bereitgestellten Anwendungen verwenden.</li></ul> | <ul><li>Kann auf Protokolldateien zugreifen.</li><li>Kann die App- und Servicekonfiguration anzeigen.</li><li>Kann die bereitgestellten Anwendungen verwenden.</li></ul> | <ul><li>Kein Zugriff.</li></ul> |
| Tester    | <ul><li>Kann die bereitgestellten Anwendungen verwenden.</li></ul> | <ul><li>Kann die bereitgestellten Anwendungen verwenden.</li></ul> | <ul><li>Kein Zugriff.</li></ul> |
| Operator  | <ul><li>Kann auf Protokolldateien zugreifen.</li><li>Kann die App- und Servicekonfiguration anzeigen/festlegen.</li></ul> | <ul><li>Kann auf Protokolldateien zugreifen.</li><li>Kann die App- und Servicekonfiguration anzeigen/festlegen.</li></ul> | <ul><li>Kann auf Protokolldateien zugreifen.</li><li>Kann die App- und Servicekonfiguration anzeigen/festlegen.</li></ul> |
| Funktionaler Benutzer für Pipeline  | <ul><li>Kann Anwendungen bereitstellen/entfernen.</li><li>Kann die App- und Servicekonfiguration anzeigen/festlegen.</li></ul> | <ul><li>Kann Anwendungen bereitstellen/entfernen.</li><li>Kann die App- und Servicekonfiguration anzeigen/festlegen.</li></ul> | <ul><li>Kann Anwendungen bereitstellen/entfernen.</li><li>Kann die App- und Servicekonfiguration anzeigen/festlegen.</li></ul> |

## Identity and Access Management (IAM)
{: #first_objective}

{{site.data.keyword.iamshort}} (IAM) ermöglicht Ihnen eine sichere Authentifizierung von Benutzern für Plattform- und Infrastrukturservices sowie eine konsistente Steuerung des Zugriffs auf **Ressourcen** auf der gesamten {{site.data.keyword.cloud_notm}}-Plattform. Eine Gruppe von {{site.data.keyword.cloud_notm}}-Services wird zur Verwendung von Cloud IAM für die Zugriffssteuerung eingerichtet und in **Ressourcengruppen** innerhalb Ihres **Kontos** organisiert, sodass es möglich ist, **Benutzern** rasch und einfach Zugriff auf mehr als eine Ressource gleichzeitig zu erteilen. Benutzern und Service-IDs wird der Zugriff auf die Ressourcen in Ihrem Konto durch **Cloud IAM-Zugriffsrichtlinien** zugeordnet.

Eine **Richtlinie** weist einem Benutzer oder einer Service-ID eine oder mehrere **Rollen** mit einer Kombination von Attributen zu, die den Bereich des Zugriffs definieren. Die Richtlinie kann Zugriff auf einen einzelnen Service bis hinunter zur Instanzebene bereitstellen oder sie kann für eine Gruppe von Ressourcen gelten, die in einer Ressourcengruppe zusammengefasst sind. Abhängig von den Benutzerrollen, die Sie zuweisen, werden dem Benutzer bzw. der Service-ID unterschiedliche Zugriffsebenen zur Ausführung von Plattformmanagement-Tasks, für den Zugriff auf einen Service über die Benutzerschnittstelle (UI) oder zur Ausführung bestimmter Typen von API-Aufrufen erteilt.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="Diagramm des IAM-Modells" />
</p>

Zum gegenwärtigen Zeitpunkt können nicht alle Services im {{site.data.keyword.cloud_notm}}-Katalog durch IAM verwaltet werden. Für diese Services können Sie weiterhin Cloud Foundry verwenden, indem Sie Benutzern Zugriff auf die Organisation und den Bereich, zu dem die Instanz gehört, durch eine Cloud Foundry-Rolle erteilen, die zugeordnet wurde, um die zulässige Zugriffsebene zu definieren.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Diagramm des Cloud Foundry-Modells" />
</p>

## Ressourcen für eine einzelne Umgebung erstellen

Obwohl die drei Umgebungen, die in diesem Beispielprojekt benötigt werden, unterschiedliche Zugriffsberechtigungen erfordern und möglicherweise mit unterschiedlichen Kapazitäten ausgestattet werden müssen, verfügen sie doch über ein gemeinsames Architekturmuster.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="Architekturdiagramm mit einer einzelnen Umgebung" />
</p>

Beginnen Sie mit der Erstellung der Entwicklungsumgebung (Development).

1. [Wählen Sie eine {{site.data.keyword.cloud_notm}}-Position aus,](https://{DomainName}) an der die Umgebung bereitgestellt werden soll.
1. Für Cloud Foundry-Services und -Apps:
   1. [Erstellen Sie eine Organisation für das Projekt](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg).
   1. [Erstellen Sie einen Cloud Foundry-Bereich für die Umgebung](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo).
   1. Erstellen Sie die Cloud Foundry-Services, die von dem Projekt unter diesem Bereich verwendet werden.
1. [Erstellen Sie eine Ressourcengruppe für die Umgebung](https://{DomainName}/account/resource-groups).
1. Erstellen Sie die Services, die mit der Ressourcengruppe kompatibel sind, wie zum Beispiel {{site.data.keyword.cos_full_notm}}, {{site.data.keyword.la_full_notm}}, {{site.data.keyword.mon_full_notm}} und {{site.data.keyword.cloudant_short_notm}}, in dieser Gruppe.
1. [Erstellen Sie einen neuen Kubernetes-Cluster](https://{DomainName}/containers-kubernetes/catalog/cluster) in {{site.data.keyword.containershort_notm}}, indem Sie sicherstellen, dass Sie die zuvor erstellte Ressourcengruppe auswählen.
1. Konfigurieren Sie {{site.data.keyword.la_full_notm}} und {{site.data.keyword.mon_full_notm}} zum Senden von Protokollen und zum Überwachen des Clusters.

Das folgende Diagramm zeigt, wo die Projektressourcen unter dem Konto erstellt werden:

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="Diagramm mit Darstellung der Projektressourcen" />
</p>

## Rollen innerhalb der Umgebung zuweisen

1. Laden Sie Benutzer in das Konto ein.
1. Weisen Sie den Benutzern Richtlinien zu, um zu steuern, wer auf die Ressourcengruppe, die Services in der Gruppe sowie auf die {{site.data.keyword.containershort_notm}}-Instanz und ihre Berechtigungen zugreifen kann. Informationen zur Auswahl der richtigen Richtlinien für einen Benutzer in der Umgebung finden Sie unter [Zugriffsrichtliniendefinition](https://{DomainName}/docs/containers?topic=containers-users#access_policies). Benutzer mit derselben Gruppe von Richtlinien können [derselben Zugriffsgruppe](https://{DomainName}/docs/iam?topic=iam-groups#groups) hinzugefügt werden. Dies vereinfacht das Benutzermanagement, da Richtlinien den Zugriffsgruppen zugewiesen und von allen Benutzern in der Gruppe übernommen werden.
1. Konfigurieren Sie die Rollen für die Cloud Foundry-Organisation und den Cloud Foundry-Bereich entsprechend den Anforderungen der Benutzer in der Umgebung. Informationen zur Zuweisung der richtigen Rollen für die Umgebung finden Sie unter [Rollendefinition](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess).

Informationen dazu, wie ein Service IAM- und Cloud Foundry-Rollen bestimmten Aktionen zuordnet, finden Sie in der Dokumentation von Services. Lesen Sie zum Beispiel die Informationen dazu, [wie der {{site.data.keyword.mon_full_notm}}-Service Aktionen IAM-Rollen zuordnet](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam).

Die Zuweisung der richtigen Rollen für Benutzer erfordert wahrscheinlich mehrere Durchgänge und Verfeinerungen. Da Berechtigungen auf der Ressourcengruppenebene für alle Ressourcen in einer Gruppe oder bis hin zu einer bestimmten Instanz eines Service differenziert gesteuert werden können, werden Sie mit der Zeit herausfinden, welche Zugriffsrichtlinien sich für Ihr Projekt am besten eignen.

Es ist gute Praxis, mit einem minimalen Satz von Berechtigungen zu beginnen und diesen dann je nach Bedarf vorsichtig zu erweitern. Für Kubernetes ist zu empfehlen, sich mit der [rollenbasierten Zugriffssteuerung (Role-Based Access Control - RBAC)](https://kubernetes.io/docs/admin/authorization/rbac/) im Hinblick auf die clusterinternen Berechtigungen vertraut zu machen.

Für die Entwicklungsumgebung (Development) lassen sich die zuvor definierten Benutzerzuständigkeiten wie folgt umsetzen:

|           | IAM-Zugriffsrichtlinien | Cloud Foundry |
| --------- | ----------- | ------- |
| Entwickler | <ul><li>Ressourcengruppe: *Anzeigeberechtigter*</li><li>Plattformzugriffsrollen in der Ressourcengruppe: *Anzeigeberechtigter*</li><li>Rolle für Prokoll- und Überwachungsservices: *Schreibberechtigter*</li></ul> | <ul><li>Organisationsrolle: *Auditor*</li><li>Bereichsrolle: *Auditor*</li></ul> |
| Tester    | <ul><li>Keine Konfiguration erforderlich. Ein Tester greift auf die bereitgestellte Anwendung und nicht auf Entwicklungsumgebungen zu.</li></ul> | <ul><li>Keine Konfiguration erforderlich.</li></ul> |
| Operator  | <ul><li>Ressourcengruppe: *Anzeigeberechtigter*</li><li>Plattformzugriffsrollen in der Ressourcengruppe: *Operator*, *Anzeigeberechtigter*</li><li>Rolle für Prokoll- und Überwachungsservices: *Schreibberechtigter*</li></ul> | <ul><li>Organisationsrolle: *Auditor*</li><li>Bereichsrolle: *Entwickler*</li></ul> |
| Funktionaler Benutzer für Pipeline | <ul><li>Ressourcengruppe: *Anzeigeberechtigter*</li><li>Plattformzugriffsrollen in der Ressourcengruppe: *Editor*, *Anzeigeberechtigter*</li></ul> | <ul><li>Organisationsrolle: *Auditor*</li><li>Bereichsrolle: *Entwickler*</li></ul> |

Die IAM-Zugriffsrichtlinien und Cloud Foundry-Rollen werden in der [Identify and Access Management-Benutzerschnittstelle](https://{DomainName}/iam/#/users) definiert:

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="Konfiguration von Berechtigungen für die Entwicklerrolle" />
</p>

## Konfiguration für mehrere Umgebungen replizieren

Von hier aus können Sie ähnliche Schritte wiederholen, um die anderen Umgebungen zu erstellen.

1. Erstellen Sie nur eine Ressourcengruppe pro Umgebung.
1. Erstellen Sie nur einen Cluster und die erforderlichen Serviceinstanzen pro Umgebung.
1. Erstellen Sie nur einen Cloud Foundry-Bereich pro Umgebung.
1. Erstellen Sie die erforderlichen Serviceinstanzen in jedem Bereich.

<p style="text-align: center;">
  <img title="Verwendung separater Cluster zur Isolation von Umgebungen" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="Diagramm, das separate Cluster zur Isolation von Umgebungen darstellt" />
</p>

Durch Kombinieren von Tools wie der [{{site.data.keyword.cloud_notm}}-CLI `ibmcloud`](https://github.com/IBM-Cloud/ibm-cloud-developer-tools), dem [HashiCorp-Tool `terraform`](https://www.terraform.io/), dem [{{site.data.keyword.cloud_notm}}-Provider für Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm) und der Kubernetes-CLI `kubectl` können Sie die Erstellung dieser Umgebungen mit Scripting unterstützen und automatisieren.

Separate Kubernetes-Cluster für die Umgebungen zeichnen sich durch vorteilhafte Eigenschaften aus:
* Unabhängig von der Umgebung zeigen alle Cluster ein ähnliches Erscheinungsbild.
* Es ist einfacher, den Zugriff auf einen bestimmten Cluster zu steuern.
* Es bietet Flexibilität in den Aktualisierungszyklen für Bereitstellungen und zugrunde liegende Ressourcen. Wenn eine neue Kubernetes-Version verfügbar wird, bietet dies die Option, den Entwicklungscluster zuerst zu aktualisieren, die Anwendung zu validieren und dann erst die andere Umgebung zu aktualisieren.
* Es vermeidet die Vermischung verschiedener Workloads, die sich gegenseitig beeinträchtigen könnten, wie zum Beispiel durch Isolieren der Bereitstellung in der Produktionsumgebung von den anderen Bereitstellungen.

Ein anderer Ansatz besteht darin, [Kubernetes-Namensbereiche](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) in Verbindung mit [Kubernetes-Ressourcengrößenbeschränkungen](https://kubernetes.io/docs/concepts/policy/resource-quotas/) zu verwenden, um Umgebungen zu isolieren und die Ressourcennutzung zu steuern.

<p style="text-align: center;">
  <img title="Verwendung separater Namensbereiche zur Isolation von Umgebungen" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="Diagramm, dass separate Namensbereiche zur Isolation von Umgebungen darstellt" />
</p>

Verwenden Sie im Sucheingabefeld (`Search`) der LogDNA-Benutzerschnittstelle das Feld `namespace: `, um Protokolle nach Namensbereich zu filtern.
{: tip}

## Delivery Pipeline einrichten

Bei der Bereitstellung in den verschiedenen Umgebungen kann Ihre Pipeline für kontinuierliche Integration und Continuous Delivery zur Durchführung des vollständigen Prozesses eingerichtet werden:
* Sie können die Umgebung `Development` kontinuierlich mit dem neuesten und umfassendsten Code aus dem Entwicklungszweig (`Development`) aktualisieren, indem Sie Komponententests und Integrationstests auf dem dedizierten Cluster durchführen.
* Sie können Entwicklungsbuilds an die Testumgebung `Testing` weitergeben, und dies entweder automatisch, wenn alle Tests aus den vorherigen Stages erfolgreich sind, oder durch einen manuellen Weitergabeprozess. Einige Teams verwenden auch hier verschiedene Zweige, indem sie beispielsweise den Entwicklungsarbeitsstatus zu einem `stabilen` Zweig zusammenführen.
* Wiederholen Sie einen ähnlichen Prozess, um die Verschiebung in die Produktionsumgebung (`Production`) durchzuführen.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="Eine CI/CD-Pipeline vom Build zur Bereitstellung" />
</p>

Stellen Sie bei der Konfiguration der DevOps-Pipeline sicher, dass Sie den API-Schlüssel eines funktionalen Benutzers verwenden. Nur der funktionale Benutzer sollte über die erforderlichen Berechtigungen zum Bereitstellen von Apps in Ihren Clustern verfügen müssen.

Während der Buildphase ist es wichtig, die Docker-Images geeignet zu versionieren. Sie können die Git-Commit-Revision als Teil des Image-Tags oder eine eindeutige ID, die von Ihrer DevOps-Toolchain bereitgestellt wird, verwenden. Sie können jede ID verwenden, die die Zuordnung des Image zum tatsächlichen Build und zu dem Quellcode, der in dem Image enthalten ist, leicht macht.

Wenn Ihnen der Umgang mit Kubernetes geläufiger wird, wird [Helm](https://helm.sh/), der Paketmanager für Kubernetes, zu einem nützlichen Tool zur Versionierung, Assemblierung und Bereitstellung Ihrer Anwendung. [Dieses DevOps-Toolchain-Beispiel](https://github.com/open-toolchain/simple-helm-toolchain) ist ein guter Ausgangspunkt und für Continuous Delivery an einen Kubernetes-Cluster vorkonfiguriert. Wenn Ihr Projekt zu einer Reihe von Microservices heranwächst, stellt das [Helm-Umbrella-Diagramm](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies) eine gute Lösung zur Zusammenstellung Ihrer Anwendung bereit.

## Lernprogramm erweitern

Herzlichen Glückwunsch! Ihre Anwendung kann jetzt sicher aus der Entwicklungsumgebung in die Produktionsumgebung verschoben und dort bereitgestellt werden. Nachfolgend finden Sie weitere Vorschläge zur Verbesserung der Anwendungsbereitstellung.

* Fügen Sie [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) Ihrer Pipeline hinzu, um bei Bereitstellungen Qualitätskontrollen durchzuführen.
* Prüfen Sie die Codeergänzungen von Teammitgliedern und die Interaktionen zwischen Entwicklern mit [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).
* Befolgen Sie das Lernprogramm [Bereitstellungsumgebungen planen, erstellen und aktualisieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments), um die Bereitstellung Ihrer Umgebungen zu automatisieren.

## Zugehörige Informationen

* [Einführung in {{site.data.keyword.iamshort}}](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [Bewährte Verfahren für das Zusammenfassen von Ressourcen in Ressourcengruppen](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [Protokolle analysieren und den Anwendungsstatus mit LogDNA und Sysdig überwachen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [Fortlaufende Bereitstellung in Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Toolchain 'Hello Helm'](https://github.com/open-toolchain/simple-helm-toolchain)
* [Microserviceanwendung mit Kubernetes und Helm entwickeln](https://github.com/open-toolchain/microservices-helm-toolchain)
* [Berechtigungen zum Anzeigen von Protokollen in LogDNA einem Benutzer erteilen](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [Berechtigungen zum Anzeigen von Metriken in Sysdig einem Benutzer erteilen](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
