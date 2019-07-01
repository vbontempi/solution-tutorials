---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# Bereitstellungsumgebungen planen, erstellen und aktualisieren
{: #plan-create-update-deployments}

Beim Erstellen einer Lösung werden häufig mehrere Bereitstellungsumgebungen verwendet. Sie spiegeln den Lebenszyklus eines Projekts von der Entwicklung bis zur Produktion wider. Im vorliegenden Lernprogramm werden Tools wie die {{site.data.keyword.Bluemix_notm}}-Befehlszeilenschnittstelle (Command Line Interface, CLI) und [Terraform](https://www.terraform.io/) zum Automatisieren der Erstellung und Wartung dieser Bereitstellungsumgebungen eingeführt.
{:shortdesc}

Entwickler möchten möglichst nicht zweimal dieselbe Codesequenz schreiben. Das Prinzip der [Vermeidung von Wiederholungen](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) ist ein Beispiel hierfür. Ebenso möchten Sie nicht zahllose Klickaktionen in einer Benutzerschnittstelle ausführen müssen, um eine Umgebung einzurichten. Aus diesem Grund arbeiten Systemadministratoren und Entwickler schon seit Langem mit Shell-Scripts, um häufig wiederkehrende, fehlerträchtige und routinemäßige Tasks zu automatisieren.

Infrastructure as a Service (IaaS), Platform as a Service (PaaS), Container as a Service (CaaS) und Functions as a Service (FaaS) ermöglichen den Entwicklern einen hohen Abstraktionsgrad und erleichtern das Anfordern von Ressourcen wie Bare-Metal-Server, verwaltete Datenbanken, virtuelle Maschinen, Kubernetes-Cluster usw. Diese Ressourcen müssen nach der Bereitstellung jedoch miteinander verbunden werden, um den Benutzerzugang zu konfigurieren, die Konfiguration fortlaufend zu aktualisieren usw. Die Möglichkeit zum Automatisieren dieser Schritte und zum Wiederholen der Installation bzw. Konfiguration in unterschiedlichen Umgebungen ist heute unverzichtbar.

Mehrere Umgebungen kommen in Projekten häufig zum Einsatz, um in den verschiedenen Phasen des Entwicklungszyklus kleine Abweichungen in den Bereichen Kapazität, Netzbetrieb, Berechtigungsnachweise oder Protokolldetails zu unterstützen. In [diesem anderen Lernprogramm](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications) werden bewährte Verfahren zum Organisieren von Benutzern, Teams und Anwendungen sowie ein Beispielszenario erläutert. Das Beispielszenario berücksichtigt drei Umgebungen: *Entwicklung*, *Test* und *Produktion*. Wie kann die Erstellung dieser Umgebungen automatisiert werden? Welche Tools können verwendet werden?

## Ziele
{: #objectives}

* Eine Reihe von Umgebungen definieren, die bereitgestellt werden können
* Mit der {{site.data.keyword.Bluemix_notm}}-CLI und mit [Terraform](https://www.terraform.io/) Scripts zum Automatisieren der Bereitstellung dieser Umgebungen schreiben
* Die Umgebungen in Ihrem Konto bereitstellen

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Produkte verwendet:
* [{{site.data.keyword.Bluemix_notm}}-Provider für Terraform](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Identity and Access Management](https://{DomainName}/iam/#/users)
* [{{site.data.keyword.Bluemix_notm}}-Befehlszeilenschnittstelle - die `ibmcloud`-CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. Es wird eine Gruppe von Terraform-Dateien erstellt, um die Zielinfrastruktur als Code zu beschreiben.
1. Ein Operator verwendet `terraform apply`, um die Umgebungen bereitzustellen.
1. Es werden Shell-Scripts geschrieben, um die Konfiguration der Umgebungen abzuschließen.
1. Der Operator führt die Scripts in den Umgebungen aus.
1. Die Umgebungen sind vollständig konfiguriert und betriebsbereit.

## Übersicht über die verfügbaren Tools
{: #tools}

Das erste Tool für die Interaktion mit {{site.data.keyword.Bluemix_notm}} und zum Erstellen reproduzierbarer Bereitstellungen ist die [{{site.data.keyword.Bluemix_notm}}-Befehlszeilenschnittstelle (`ibmcloud`-CLI](/docs/cli?topic=cloud-cli-install-ibmcloud-cli). Mit `ibmcloud` und den zugehörigen Plug-ins können Sie die Erstellung und Konfiguration Ihrer Cloudressourcen automatisieren. {{site.data.keyword.virtualmachinesshort}}, Kubernetes-Cluster, {{site.data.keyword.openwhisk_short}}, Cloud Foundry-Apps und -Services können alle mithilfe der Befehlszeile bereitgestellt werden.

Ein weiteres Tool, das in [diesem Lernprogramm](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform) eingeführt wird, ist [Terraform](https://www.terraform.io/) von HashiCorp. Laut der Beschreibung von HashiCorp ermöglicht *Terraform das sichere und vorhersehbare Erstellen, Ändern und Verbessern der Infrastruktur. Dieses Open-Source-Tool codiert APIs in deklarativen Konfigurationsdateien, die von Teammitgliedern gemeinsam genutzt, als Code behandelt, bearbeitet, überprüft und versioniert werden können.* Das Tool beschreibt Infrastruktur in Form von Code. Sie beschreiben, wie Ihre Infrastruktur aussehen soll, und Terraform erstellt, aktualisiert bzw. entfernt Cloudressourcen je nach Bedarf.

Zur Unterstützung eines Multi-Cloud-Konzepts arbeitet Terraform mit Providern. Ein Provider ist dafür verantwortlich, API-Interaktionen zu durchschauen und Ressourcen bereitzustellen. {{site.data.keyword.Bluemix_notm}} verfügt über [einen Provider für Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm) und bietet dadurch den Benutzern von {{site.data.keyword.Bluemix_notm}} die Möglichkeit, Ressourcen mit Terraform zu verwalten. Obwohl Terraform der Kategorie 'Intrastruktur als Code' angehört, kann das Tool nicht nur für IaaS-Ressourcen (Infrastructure-As-A-Service) verwendet werden. Der {{site.data.keyword.Bluemix_notm}}-Provider für Terraform unterstützt Ressourcen der folgenden Typen: IaaS (Bare-Metal-Systeme, virtuelle Maschinen, Netzservices usw.), CaaS ({{site.data.keyword.containershort_notm}}- und Kubernetes-Cluster), PaaS (Cloud Foundry und Services) und FaaS ({{site.data.keyword.openwhisk_short}}).

## Scripts zum Automatisieren der Bereitstellung schreiben
{: #scripts}

Beim Beschreiben der Infrastruktur als Code, müssen Dateien, die Sie erstellen, wie regulärer Code behandelt und in einem Quellcodeverwaltungssystem gespeichert werden. Dies bietet im Laufe der Zeit sehr vorteilhafte Möglichkeiten wie die Verwendung des Prüfworkflows der Quellcodeverwaltung zum Überprüfen von Änderungen, bevor sie angewendet werden, und damit eine kontinuierliche Integrationspipeline für die automatische Bereitstellung von Infrastrukturänderungen.

[Dieses Git-Repository](https://github.com/IBM-Cloud/multiple-environments-as-code) enthält alle erforderlichen Konfigurationsdateien zum Einrichten der zuvor definierten Umgebungen. Sie können das Repository klonen, um die nächsten Abschnitte durchzuarbeiten, in denen der Inhalt dieser Dateien erläutert wird.

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

Das Repository ist wie folgt strukturiert:

| Verzeichnis | Beschreibung |
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) | Ausgangsposition für die Terraform-Dateien |
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) | Terraform-Dateien zum Bereitstellen gemeinsamer Ressourcen für die drei Umgebungen |
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) | Spezielle Terraform-Dateien für eine bestimmte Umgebung |
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) | Terraform-Dateien zum Konfigurieren von Benutzerrichtlinien|

### Verarbeitung großer Arbeitslasten mit Terraform

Die Umgebungen für *Entwicklung*, *Test* und *Produktion* sind sehr ähnlich aufgebaut.

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="Diagramm einer Bereitstellungsumgebung" />
</p>

Sie nutzen eine gemeinsame Organisation und umgebungsspezifische Ressourcen. Sie unterscheiden sich jedoch in der zugeordneten Kapazität und in den Zugriffsberechtigungen. In den Terraform-Dateien ist dies an einer ***globalen*** Konfiguration zum Bereitstellen der Cloud Foundry-Organisation erkennbar und an einer Konfiguration ***für jede Umgebung***, die in Terraform-Arbeitsbereichen die umgebungsspezifischen Ressourcen bereitstellt.

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### Globale Konfiguration

Alle Umgebungen greifen auf eine gemeinsame Cloud Foundry-Organisation zurück und jede Umgebung verfügt über einen eigenen Bereich.

Im Verzeichnis [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) finden Sie die Terraform-Scripts zum Bereitstellen dieser Organisation. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf) enthält die Definition für die Organisation:

   ```sh
   # Neue Organisation für das Projekt erstellen
   resource "ibm_org" "organization" {
     name             = "${var.org_name}"
     managers         = "${var.org_managers}"
     users            = "${var.org_users}"
     auditors         = "${var.org_auditors}"
     billing_managers = "${var.org_billing_managers}"
   }
   ```

In dieser Ressource werden alle Eigenschaften durch Variablen konfiguriert. In den nächsten Abschnitten erfahren Sie, wie diese Variablen festgelegt werden.

Für die vollständige Bereitstellung der Umgebungen verwenden Sie eine Mischung aus Terraform und der {{site.data.keyword.Bluemix_notm}}-CLI. Mit der CLI geschriebene Shell-Scripts müssen gegebenenfalls mit Name oder ID auf diese Organisation oder auf das Konto verweisen. Das Verzeichnis *global* enthält außerdem [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf), das eine Datei erzeugt, in der diese Informationen als Schlüssel bzw. Werte enthalten sind, die zum Erstellen von Scripten wiederverwendet werden können.

   ```sh
   # Geeignete Eigenschaftendatei für Shell-Scripts mit hilfreichen Variablen für die Umgebung generieren
   resource "local_file" "output" {
     content = <<EOF
ACCOUNT_GUID=${data.ibm_account.account.id}
ORG_GUID=${ibm_org.organization.id}
ORG_NAME=${var.org_name}
EOF

     filename = "../outputs/global.env"
   }
   ```

### Einzelne Umgebungen

Es sind verschiedene Ansätze verfügbar, um mehrere Umgebungen zu verwalten. Sie können die Terraform-Dateien in separaten Verzeichnissen duplizieren (ein Verzeichnis pro Umgebung). Mithilfe von [Terraform-Modulen](https://www.terraform.io/docs/modules/index.html) können Sie die gemeinsame Konfiguration als Gruppe anlegen und Module für mehrere Umgebungen wiederverwenden, um doppelte Codeabschnitte nach Möglichkeit zu vermeiden. Separate Verzeichnisse bieten die Möglichkeit, die *Entwicklungsumgebung* so zu gestalten, dass Änderungen zunächst getestet und erst danach in andere Umgebungen übernommen werden. In diesem Fall werden die Terraform-*Module* häufig in einem eigenen Quellcode-Repository gespeichert, damit Sie in Ihren Umgebungsdateien auf eine bestimmte Modulversion verweisen können.

Wenn die Umgebungen eher einfach und sehr ähnlich konzipiert sind, können Sie auf ein anderes Terraform-Konzept zurückgreifen, das als [Arbeitsbereiche](https://www.terraform.io/docs/state/workspaces.html#best-practices) bezeichnet wird. Arbeitsbereiche ermöglichen Ihnen die Verwendung derselben Terraform-Dateien (.tf) für unterschiedliche Umgebungen. In dem angegebenen Beispiel handelt es sich bei den Umgebungen *development* (Entwicklung), *testing* (Test) und *production* (Produktion) um Arbeitsbereiche. Sie verwenden dieselben Terraform-Definitionen, jedoch mit unterschiedlichen Konfigurationsvariablen (z. B. verschiedene Namen und Kapazitäten).

Für jede Umgebung ist Folgendes erforderlich:
* Ein dedizierter Cloud Foundry-Bereich
* Eine dedizierte Ressourcengruppe
* Ein Kubernetes-Cluster
* Eine Datenbank
* Ein Dateispeicher

Der Cloud Foundry-Bereich ist mit der Organisation verknüpft, die im vorherigen Schritt erstellt wurde. In den Terraform-Dateien für die Umgebung muss auf diese Organisation verwiesen werden. An dieser Stelle kommt das [Terraform-Konzept 'Ferner Status'](https://www.terraform.io/docs/state/remote.html) ins Spiel. Es ermöglicht Verweise auf einen vorhandenen Terraform-Status im Lesezugriffsmodus. Dieses hilfreiche Konstrukt teilt Ihre Terraform-Konfiguration in mehrere Abschnitte auf, für die jeweils andere Teams verantwortlich sein können. Die Datei [backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf) enthält die Definition des fernen Status *global*, der zum Suchen der zuvor erstellten Organisation verwendet wird:

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

Sobald auf die Organisation verwiesen werden kann, ist es ohne großen Aufwand möglich, einen Bereich innerhalb der Organisation zu erstellen. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf) enthält die Definition der Ressourcen für die Umgebung.

   ```sh
   # Ein Cloud Foundry-Bereich pro Umgebung
   resource "ibm_space" "space" {
     name       = "${var.environment_name}"
     org        = "${data.terraform_remote_state.global.org_name}"
     managers   = "${var.space_managers}"
     auditors   = "${var.space_auditors}"
     developers = "${var.space_developers}"
   }
   ```

Beachten Sie, wie im fernen Status *global* auf den Namen der Organisation verwiesen wird. Die übrigen Eigenschaften werden aus Konfigurationsvariablen übernommen.

Nun ist die Ressourcengruppe an der Reihe.

   ```sh
   # Eine Ressourcengruppe
   resource "ibm_resource_group" "group" {
    name     = "${var.environment_name}"
    quota_id = "${data.ibm_resource_quota.quota.id}"
}

   data "ibm_resource_quota" "quota" {
	name = "${var.resource_quota}"
}
   ```

Der Kubernetes-Cluster wird in dieser Ressourcengruppe erstellt. Der {{site.data.keyword.Bluemix_notm}}-Provider verfügt über eine Terraform-Ressource, die einen Cluster darstellt:

   ```sh
  # Ein Cluster
  resource "ibm_container_cluster" "cluster" {
  	name              = "${var.environment_name}-cluster"
  	datacenter        = "${var.cluster_datacenter}"
  	org_guid          = "${data.terraform_remote_state.global.org_guid}"
  	space_guid        = "${ibm_space.space.id}"
  	account_guid      = "${data.terraform_remote_state.global.account_guid}"
  	hardware          = "${var.cluster_hardware}"
  	machine_type      = "${var.cluster_machine_type}"
  	public_vlan_id    = "${var.cluster_public_vlan_id}"
  	private_vlan_id   = "${var.cluster_private_vlan_id}"
  	resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool" "cluster_workerpool" {
  worker_pool_name  = "${var.environment_name}-pool"
  machine_type      = "${var.cluster_machine_type}"
  cluster           = "${ibm_container_cluster.cluster.id}"
  size_per_zone     = "${var.worker_num}"
  hardware          = "${var.cluster_hardware}"
  resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool_zone_attachment" "cluster_zone" {
  cluster           = "${ibm_container_cluster.cluster.id}"
  worker_pool       =  "${element(split("/",ibm_container_worker_pool.cluster_workerpool.id),1)}"
  zone              = "${var.cluster_datacenter}"
  public_vlan_id    = "${var.cluster_public_vlan_id}"
  private_vlan_id   = "${var.cluster_private_vlan_id}"
  resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

Auch hier werden die meisten Eigenschaften durch Konfigurationsvariablen initialisiert. Sie können das Rechenzentrum sowie die Anzahl und den Typ der Worker anpassen.

Für IAM aktivierte Services wie {{site.data.keyword.cos_full_notm}} und {{site.data.keyword.cloudant_short_notm}} werden ebenfalls als Ressourcen in der Gruppe erstellt.

   ```sh
# Eine Datenbank
resource "ibm_resource_instance" "database" {
    name              = "database"
    service           = "cloudantnosqldb"
    plan              = "${var.cloudantnosqldb_plan}"
    location          = "${var.cloudantnosqldb_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
# Ein Cloud-Objektspeicher
resource "ibm_resource_instance" "objectstorage" {
    name              = "objectstorage"
    service           = "cloud-object-storage"
    plan              = "${var.cloudobjectstorage_plan}"
    location          = "${var.cloudobjectstorage_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

Kubernetes-Bindungen (geheime Schlüssel) können hinzugefügt werden, um die Serviceberechtigungsnachweise aus Ihren Anwendungen abzurufen:

   ```sh
   # Cloudant-Service an den Cluster binden
   resource "ibm_container_bind_service" "bind_database" {
      cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	  service_instance_name       = "${ibm_resource_instance.database.name}"
      namespace_id                = "default"
      account_guid                = "${data.terraform_remote_state.global.account_guid}"
      org_guid                    = "${data.terraform_remote_state.global.org_guid}"
      space_guid                  = "${ibm_space.space.id}"
      resource_group_id           = "${ibm_resource_group.group.id}"
}

   # Cloud-Objektspeicherservice an den Cluster binden
   resource "ibm_container_bind_service" "bind_objectstorage" {
  	cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	space_guid                  = "${ibm_space.space.id}"
  	service_instance_id         = "${ibm_resource_instance.objectstorage.name}"
  	namespace_id                = "default"
  	account_guid                = "${data.terraform_remote_state.global.account_guid}"
  	org_guid                    = "${data.terraform_remote_state.global.org_guid}"
  	space_guid                  = "${ibm_space.space.id}"
  	resource_group_id           = "${ibm_resource_group.group.id}"
}
   ```

## Umgebung in Ihrem Konto bereitstellen

### {{site.data.keyword.Bluemix_notm}}-CLI installieren

1. Führen Sie [diese Anweisungen](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) aus, um die CLI zu installieren.
1. Überprüfen Sie die Installation, indem Sie Folgendes ausführen:
   ```sh
   ibmcloud
   ```
   {: codeblock}

### Terraform und den {{site.data.keyword.Bluemix_notm}}-Provider für Terraform installieren

1. [Laden Sie Terraform herunter und installieren Sie es für Ihr System.](https://www.terraform.io/intro/getting-started/install.html)
1. [Laden Sie die Terraform-Binärdatei für den {{site.data.keyword.Bluemix_notm}}-Provider herunter.](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)
   Informationen zum Einrichten von Terraform mit {{site.data.keyword.Bluemix_notm}}-Provider finden Sie [über diesen Link](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup).
   {:tip}
1. Erstellen Sie eine Datei `.terraformrc` in Ihrem Ausgangsverzeichnis, die auf die Terraform-Binärdatei verweist. Im folgenden Beispiel ist `/opt/provider/terraform-provider-ibm` die Route zu dem Verzeichnis.
   ```sh
   # ~/.terraformrc
   providers {
     ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### Code abrufen

Klonen Sie das Repository des Lernprogramms, falls noch nicht geschehen:

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### Legen Sie den Plattform-API-Schlüssel fest

1. Falls Sie noch nicht über einen solchen Schlüssel verfügen, rufen Sie einen [Plattform-API-Schlüssel](https://{DomainName}/iam/#/apikeys) ab und speichern Sie den API-Schlüssel zur späteren Verwendung.

   > Wenn Sie in späteren Schritten eine neue Cloud Foundry-Organisation als Host für die Bereitstellungsumgebungen erstellen möchten, stellen Sie sicher, dass Sie der Eigner des Kontos sind.
1. Kopieren Sie [terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl) in *terraform/credentials.tfvars*, indem Sie den folgenden Befehl ausführen:
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. Bearbeiten Sie `terraform/credentials.tfvars` und legen Sie als Wert für `ibmcloud_api_key` den von Ihnen abgerufenen Plattform-API-Schlüssel fest.

### Cloud Foundry-Organisation erstellen oder wiederverwenden

Sie können entweder eine neue Organisation erstellen oder eine vorhandene Organisation wiederverwenden (importieren). Zum Erstellen der übergeordneten Organisation für die drei Bereitstellungsumgebungen **müssen Sie der Eigner des Kontos sein**.

#### Neue Organisation erstellen

1. Wechseln Sie in das Verzeichnis `terraform/global`.
1. Kopieren Sie die Datei [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) in die Datei `global.tfvars`:
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Bearbeiten Sie `global.tfvars`:
   1. Geben Sie in **org_name** den Namen der zu erstellenden Organisation an.
   1. Geben Sie in **org_managers** eine Liste der Benutzer-IDs an, denen Sie die Rolle *Manager* in der Organisation erteilen möchten. Der Benutzer, der die Organisation erstellt, hat automatisch die Rolle 'Manager' und sollte nicht zur Liste hinzugefügt werden.
   1. Geben Sie in **org_users** eine Liste aller Benutzer an, die Sie in die Organisation einladen möchten. Benutzer müssen in diese Liste eingefügt werden, wenn Sie den Zugriff für diese Benutzer in späteren Schritten konfigurieren möchten.

   ```sh
   org_name = "eine-neue-organisation"
   org_managers = [ "benutzer1@domäne.com", "anderer-benutzer@andere-domäne.com" ]
   org_users = [ "benutzer1@domäne.com", "anderer-benutzer@andere-domäne.com", "weiterer-benutzer@domäne.com" ]
   ```
   {: codeblock}
1. Initialisieren Sie Terraform im Ordner `terraform/global`:
   ```sh
   terraform init
   ```
   {: codeblock}
1. Zeigen Sie den Terraform-Plan an:
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. Wenden Sie die Änderungen an:
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Sobald Terraform abgeschlossen ist, wurde Folgendes erstellt:
* Eine neue Cloud Foundry-Organisation.
* Eine Datei `global.env` im Verzeichnis `outputs` in Ihrem Checkout. Diese Datei enthält Umgebungsvariablen, auf die Sie in anderen Scripts verweisen können.
* Die Datei `terraform.tfstate`.

> Im vorliegenden Lernprogramm wird der `lokale` Back-End-Provider für den Terraform-Status verwendet. Dies ist hilfreich, um Terraform zu erkunden oder allein an einem Projekt zu arbeiten. Für die Arbeit im Team oder an einer umfangreichen Infrastruktur unterstützt Terraform außerdem das Speichern des Status an einer fernen Position. Da der Terraform-Status für alle Terraform-Operationen von zentraler Bedeutung ist, wird die Verwendung eines fernen, hochverfügbaren und ausfallsicheren Speichers für den Terraform-Status empfohlen. Eine Liste der verfügbaren Optionen finden Sie unter [Terraform Backend Types](https://www.terraform.io/docs/backends/types/index.html). Manche Back-Ends bieten darüber hinaus Unterstützung für die Versionierung und Sperrung des Terraform-Status.

#### So können Sie eine von Ihnen verwaltete Organisation wiederverwenden

Wenn Sie nicht der Kontoeigner sind, jedoch eine Organisation in dem Konto verwalten, können Sie eine vorhandene Organisation in Terraform importieren.

1. Rufen Sie die GUID der Organisation ab:
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. Wechseln Sie in das Verzeichnis `terraform/global`.
1. Kopieren Sie die Datei [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) in die Datei `global.tfvars`:
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Initialisieren Sie Terraform:
   ```sh
   terraform init
   ```
   {: codeblock}
1. Importieren Sie nach dem Initialisieren von Terraform die Organisation in den Terraform-Status:
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. Passen Sie `global.tfvars` an den Namen und die Struktur der vorhandenen Organisation an.
1. Wenden Sie die Änderungen an:
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### Bereich, Cluster und Services für jede Umgebung erstellen

In diesem Abschnitt wird der Fokus auf die Umgebung `development` (Entwicklung) gelegt. Die auszuführenden Schritte sind die gleichen wie für die anderen Umgebungen, nur die Werte variieren.

1. Wechseln Sie in den Ordner `terraform/per-environment` für das Checkout.
1. Kopieren Sie die Vorlagendatei `tfvars`. Es gibt eine Vorlagendatei für jede Umgebung:
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. Bearbeiten Sie die Datei `development.tfvars`.
   1. Geben Sie für **environment_name** (Umgebungsname) den Namen des Cloud Foundry-Bereichs an, den Sie erstellen möchten.
   1. Geben Sie in **space_developers** (Bereichsentwickler) die Liste der Entwickler für diesen Bereich an. **Tragen Sie Ihren Namen in die Liste ein, damit Terraform Services in Ihrem Auftrag bereitstellen kann.**
   1. Geben Sie für **cluster_datacenter** (Clusterrechenzentrum) den Standort an, an dem der Cluster erstellt werden soll. Suchen Sie die verfügbaren Standorte mit dem folgenden Befehl:
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. Geben Sie das private VLAN (**cluster_private_vlan_id**) und das öffentliche VLAN (**cluster_public_vlan_id**) für den Cluster an. Suchen Sie die verfügbaren VLANS für den Standort mit dem folgenden Befehl:
      ```sh
      ibmcloud cs vlans <standort>
      ```
      {: codeblock}
   1. Geben Sie den Clustermaschinentyp (**cluster_machine_type**) an. Suchen Sie die verfügbaren Maschinentypen und Merkmale für den Standort mit dem folgenden Befehl:
      ```sh
      ibmcloud cs machine-types <standort>
      ```
      {: codeblock}

   1. Legen Sie die Ressourcenquote (**resource_quota**) fest. Suchen Sie die verfügbaren Definitionen für Ressourcenquoten mit dem folgenden Befehl:
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. Initialisieren Sie Terraform:
   ```sh
   terraform init
   ```
   {: codeblock}
1. Erstellen Sie einen neuen Terraform-Arbeitsbereich für die Entwicklungsumgebung (*development*):
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   Verwenden Sie später zum Wechseln zwischen Umgebungen den folgenden Befehl:
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. Zeigen Sie den Terraform-Plan an:
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Folgendes müsste zurückgegeben werden:
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. Wenden Sie die Änderungen an:
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Sobald Terraform abgeschlossen ist, wurde Folgendes erstellt:
* Eine Ressourcengruppe
* Ein Cloud Foundry-Bereich
* Ein Kubernetes-Cluster mit einem Worker-Pool und einer zugehörigen Zone
* Eine Datenbank
* Ein geheimer Schlüssel für Kubernetes mit den Datenbankberechtigungsnachweisen
* Ein Speicher
* Ein geheimer Schlüssel für Kubernetes mit den Speicherberechtigungsnachweisen
* Eine Protokollierungsinstanz 'logging(LogDNA)'
* Eine Überwachungsinstanz 'monitoring(Sysdig)'
* Eine Datei `development.env` im Verzeichnis `outputs` in Ihrem Checkout (diese Datei enthält Umgebungsvariablen, auf die Sie in anderen Scripts verweisen können)
* Die umgebungsspezifische Datei `terraform.tfstate` im Verzeichnis `terraform.tfstate.d/development`

Sie können diese Schritte für die Testumgebung (`testing`) und für die Produktionsumgebung (`production`) wiederholen.

### Benutzerrichtlinien zuordnen

In den vorherigen Schritten konnten Rollen in der Cloud Foundry-Organisation und Bereiche mit dem Terraform-Provider konfiguriert werden. Für Benutzerrichtlinien in anderen Ressourcen (z. B. in den Kubernetes-Clustern) verwenden Sie den Ordner [roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) in dem geklonten Repository.

Für die Umgebung *development* (wie in [diesem Lernprogramm](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications) definiert) müssen die folgenden Richtlinien definiert werden:

|           | IAM-Zugriffsrichtlinien |
| --------- | ----------- |
| Entwickler | <ul><li>Ressourcengruppe: *Anzeigeberechtigter*</li><li>Plattformzugriffsrollen in der Ressourcengruppe: *Anzeigeberechtigter*</li><li>Rolle für Protokollierung & Überwachung: *Schreibberechtigter*</li></ul> |
| Tester    | <ul><li>Keine Konfiguration erforderlich. Der Tester greift auf die bereitgestellte Anwendung zu, nicht auf die Entwicklungsumgebungen.</li></ul> |
| Operator  | <ul><li>Ressourcengruppe: *Anzeigeberechtigter*</li><li>Plattformzugriffsrollen in der Ressourcengruppe: *Operator*, *Anzeigeberechtigter*</li><li>Rolle für Protokollierung & Überwachung: *Schreibberechtigter*</li></ul> |
| Funktionaler Pipeline-Benutzer | <ul><li>Ressourcengruppe: *Anzeigeberechtigter*</li><li>Plattformzugriffsrollen in der Ressourcengruppe: *Editor* (Bearbeiter), *Anzeigeberechtigter*</li></ul> |

Da ein Team aus mehreren Entwicklern und Testern bestehen kann, können Sie das [Konzept der Zugriffsgruppen](https://{DomainName}/docs/iam?topic=iam-groups#groups) nutzen, um die Konfiguration von Benutzerrichtlinien zu vereinfachen. Da Zugriffsgruppen vom Kontoeigner erstellt werden können, kann derselbe Zugriff für alle Entitäten innerhalb der Gruppe durch eine einzige Richtlinie zugeordnet werden.

Für die Rolle *Entwickler* in der Entwicklungsumgebung (*development*) wird dies durch den folgenden Code umgesetzt:

   ```sh
resource "ibm_iam_access_group" "developer_role" {
  name        = "${var.access_group_name_developer_role}"
  description = "${var.access_group_description}"
}
resource "ibm_iam_access_group_policy" "resourcepolicy_developer" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Viewer"]

  resources = [{
    resource_type = "resource-group"
    resource      = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_platform_accesspolicy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles        = ["Viewer"]

  resources = [{
    resource_group_id = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_logging_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "logdna"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.logdna_instance_id}"
  }]
}
resource "ibm_iam_access_group_policy" "developer_monitoring_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "sysdig-monitor"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.sysdig_instance_id}"
  }]
}

   ```

Die Datei [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) im Checkout enthält Beispiele dieser Ressourcen für die definierten Rollen 'Entwickler' (*Developer*), *Operator*, *Tester* und 'Funktionaler Benutzer' (*Functional User*). So legen Sie die Richtlinien (wie in einem vorherigen Abschnitt definiert) für die Benutzer mit den Rollen *'Entwickler', 'Operator', 'Tester' und 'Funktionaler Benutzer'* in der Entwicklungsumgebung (*development*) fest:

1. Wechseln Sie in das Verzeichnis `terraform/roles/development`.
2. Kopieren Sie die Vorlagendatei `tfvars`. Es gibt eine solche Datei für jede Umgebung (die Vorlagendateien für die Umgebungen `production` und `testing` finden Sie in den entsprechenden Ordnern im Verzeichnis `roles`).

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. Bearbeiten Sie die Datei `development.tfvars`.

   - Geben Sie in **iam_access_members_developers** die Liste der Entwickler an, denen Sie Zugriff erteilen möchten.
   - Geben Sie in **iam_access_members_operators** die Liste der Operatoren (Bediener) an usw.
4. Initialisieren Sie Terraform:
   ```sh
   terraform init
   ```
   {: codeblock}

5. Zeigen Sie den Terraform-Plan an:
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Folgendes müsste zurückgegeben werden:
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. Wenden Sie die Änderungen an:
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
Sie können diese Schritte für die Testumgebung (`testing`) und für die Produktionsumgebung (`production`) wiederholen.

## Ressourcen entfernen

1. Navigieren Sie zum Ordner `development` im Verzeichnis `roles`:
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. Löschen Sie die Zugriffsgruppen und Zugriffsrichtlinien:
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Aktivieren Sie den Arbeitsbereich `development`:
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. Löschen Sie die Ressourcengruppe, Bereiche, Services und Cluster:
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Wiederholen Sie die Schritte für die Arbeitsbereiche `testing` und `production`.
1. Falls Sie die Organisation erstellt haben, löschen Sie sie:
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## Zugehörige Inhalte

* [Lernprogramm für Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Terraform-Provider](https://ibm-cloud.github.io/tf-ibm-docs/)
* [Beispiele unter Verwendung von {{site.data.keyword.Bluemix_notm}}-Provider für Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
