---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-23"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# LAMP-Stack mit Terraform bereitstellen
{: #infrastructure-as-code-terraform}

Mit [Terraform](https://www.terraform.io/) können Sie Ihre Infrastruktur sicher und vorhersehbar erstellen, ändern und verbessern. Terraform ist ein Open Source-Tool, das APIs in deklarative Konfigurationsdateien umwandelt, die an Teammitglieder weitergegeben und von diesen als Code gehandhabt, bearbeitet, überprüft und versioniert werden können.

In diesem Lernprogramm verwenden Sie eine Beispielkonfiguration zur Bereitstellung eines virtuellen **L**-inux-Servers mit **A**-Pache-Web-Server, **M**ySQL und **P**HP-Server, der als **LAMP**-Stack bezeichnet wird. Sie aktualisieren dann die Konfiguration, um den {{site.data.keyword.cos_full_notm}}-Service hinzuzufügen und die Ressourcen zu skalieren, um die Umgebung zu optimieren (Speicher, CPU und Plattengröße). Löschen Sie abschließend alle Ressourcen, die im Rahmen der Konfiguration erstellt wurden.

## Lernziele
{: #objectives}

* Terraform und den {{site.data.keyword.Bluemix_notm}}-Provider für Terraform konfigurieren
* Mit Terraform eine LAMP-Stack-Konfiguration erstellen, aktualisieren, skalieren und letztendlich löschen

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur
{: #architecture}

<p style="text-align: center;">

  ![Diagramm der Architektur](images/solution10/architecture-2.png)
</p>

1. Es wird eine Gruppe von Terraform-Dateien erstellt, um die LAMP-Stack-Konfiguration zu beschreiben.
1. `Terraform` wird über das Konfigurationsverzeichnis aufgerufen.
1. `Terraform` ruft die {{site.data.keyword.cloud_notm}}-API zum Bereitstellen der Ressourcen auf.

## Vorbereitende Schritte
{: #prereqs}

Wenden Sie sich an Ihren Infrastruktur-Masterbenutzer, um die folgenden Berechtigungen zu erhalten:
- Netz (zum Hinzufügen eines **Uplinks zum öffentlichen und privaten Netz**)
- API-Schlüssel

## Voraussetzungen

{: #prereq}

Installieren Sie **Terraform** über das [Installationsprogramm](https://www.terraform.io/intro/getting-started/install.html) oder verwenden Sie [Homebrew](https://brew.sh/) unter Mac OS, indem Sie den Befehl `brew install terraform` ausführen.

Führen Sie unter **Windows** die folgenden Schritte aus, um die Terraform-Konfiguration durchzuführen:
   1. Kopieren Sie Dateien aus der heruntergeladenen ZIP-Datei in `C:\terraform`  (erstellen Sie dazu den Ordner `terraform`).
   2. Öffnen Sie die Eingabeaufforderung als Administrator und legen Sie die Umgebungsvariable PATH so fest, dass Terraform-Binärdateien verwendet werden.

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## {{site.data.keyword.Bluemix_notm}}-Provider für Terraform konfigurieren
{: #setup}

Um einen Multi-Cloud-Ansatz zu unterstützen, arbeitet Terraform mit Providern zusammen. Ein Provider ist dafür verantwortlich, API-Interaktionen zu verstehen und Ressourcen bereitzustellen. In diesem Abschnitt konfigurieren Sie die Befehlszeilenschnittstelle, um die Position des {{site.data.keyword.Bluemix_notm}}-Providers anzugeben.

1. Überprüfen Sie die Terraform-Installation durch Ausführen von `terraform` in Ihrem Terminal oder in einem Fenster mit Eingabeaufforderung. Es sollte eine Liste mit `allgemeinen Befehlen` angezeigt werden.

  ```
  terraform
  ```

2. Laden Sie das entsprechende Plug-in für den [{{site.data.keyword.Bluemix_notm}}-Provider](https://github.com/IBM-Cloud/terraform-provider-ibm/releases) für Ihr System herunter und extrahieren Sie das Archiv. Sie sollten die binäre Plug-in-Datei `terraform-provider-ibm_VERSION` sehen.

3. Erstellen Sie für Nicht-Windows-Systeme ein Verzeichnis `.terraform.d/plugins` im Ausgangsverzeichnis Ihres Benutzers und legen Sie die Binärdatei in diesem Verzeichnis ab. Verwenden Sie die folgenden Befehle als Referenz.

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    Unter **Windows** muss die Datei unter dem Verzeichnis mit den Anwendungsdaten Ihres Benutzers im Verzeichnis in `terraform.d/plugins` abgelegt werden.

  - Führen Sie die folgenden Befehle in einer Befehlszeile für eine [Provider-Konfiguration](https://www.terraform.io/docs/configuration/providers.html) aus.
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - Starten Sie **Windows Powershell** (Start + R > Powershell) und führen Sie den folgenden Befehl aus, um die Datei `terraform.rc` zu erstellen.
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   Geben Sie an der ersten Eingabeaufforderung den folgenden Inhalt ein:
   ```
    # ~/.terraformrc
    providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        PATH_TO_YOUR_APPDATA_PLUGINS muss ein absoluter Pfad mit Schrägstrich (/) sein. Beispiel: `C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe`.
        {: tip}

  - Drücken Sie die Eingabetaste, um die Eingabeaufforderung zu beenden.

## Terraform-Konfiguration vorbereiten

{: #terraformconfig}

In diesem Abschnitt lernen Sie die Grundlagen einer Terraform-Konfiguration kennen, indem Sie eine von {{site.data.keyword.Bluemix_notm}} bereitgestellte Beispielkonfiguration für Terraform verwenden.

1. Besuchen Sie 'https://github.com/IBM-Cloud/LAMP-terraform-ibm' und **zweigen** Sie Ihre eigene Kopie für Ihr Konto ab.
2. Klonen Sie diese Abzweigung lokal:
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. Überprüfen Sie die Konfigurationsdateien.
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml): Enthält Serverinstallationskonfigurationen. Hier können Sie alle Scripts hinzufügen, die sich auf die Serverinstallation beziehen, und die angeben, was auf dem Server installiert werden soll. Siehe die Einfügung `phpinfo();` in dieser Datei.
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf): Enthält die Variablen, die sich auf den Provider beziehen, für den der Benutzername und das Kennwort erforderlich sind.
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf): Enthält die Serverkonfigurationen zur Implementierung der VM mit angegebenen Variablen.
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars): Enthält den **SoftLayer**-Benutzernamen und -API-Schlüssel, den {{site.data.keyword.Bluemix_notm}}-API-Schlüssel und die Bereichs-/Organisationsnamen. Diese Berechtigungsnachweise können als bewährtes Verfahren zu dieser Datei hinzugefügt werden, um die Berechtigungsnachweise nicht bei jeder Bereitstellung des Servers erneut über die Befehlszeile eingeben zu müssen. Hinweis: Veröffentlichen Sie diese Datei NICHT mit Ihren Berechtigungsnachweisen.
4. Öffnen Sie die Datei [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) in einer IDE Ihrer Wahl und ändern Sie die Datei, indem Sie Ihren **öffentlichen SSH**-Schlüssel hinzufügen. Dieser wird verwendet, um auf die VM zuzugreifen, die von dieser Konfiguration erstellt wurde. Weitere Informationen zum Erstellen von SSH-Schlüsseln finden Sie unter [diesem Link](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html). Um den öffentlichen Schlüssel in die Zwischenablage zu kopieren, können Sie den unten stehenden Befehl in Ihrem Terminal ausführen.

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     Mit diesem Befehl wird der SSH-Schlüssel in die Zwischenablage kopiert. Anschließend können Sie diesen in die Datei [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) unter der Standardvariablen `ssh_key` bei ungefähr Zeile 69 einfügen.

    Laden Sie unter **Windows** [Git Bash](https://git-scm.com/download/win) herunter und installieren und starten Sie es und führen Sie dann den nachfolgenden Befehl aus, um den öffentlichen SSH-Schlüssel in die Zwischenablage zu kopieren.

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. Öffnen Sie die Datei [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) mit Ihrer IDE und ändern Sie die Datei durch das Hinzufügen aller aufgeführten Berechtigungsnachweise. Dadurch müssen Sie diese Berechtigungsnachweise nicht jedes Mal erneut eingeben, wenn Sie Terraform ausführen. Sie müssen alle fünf Berechtigungsnachweise hinzufügen, die in der Datei [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) aufgelistet sind, um den Rest dieses Lernprogramms durchführen zu können.

## LAMP-Stack-Server aus der Terraform-Konfiguration erstellen
{: #Createserver}
In diesem Abschnitt erfahren Sie, wie Sie einen LAMP-Stack-Server aus dem Terraform-Konfigurationsbeispiel erstellen. Die Konfiguration wird verwendet, um eine Instanz der virtuellen Maschine bereitzustellen und **A**pache, **M**ySQL (**M**ariaDB) und **P**HP auf diese Instanz zu installieren.

1. Navigieren Sie zu dem Ordner des von Ihnen geklonten Repositorys.
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. Initialisieren Sie die Terraform-Konfiguration. Dadurch wird auch das Plug-in `terraform-provider-ibm_VERSION` installiert.
   ````bash
    terraform init
   ````
   {: pre}
3. Wenden Sie die Terraform-Konfiguration an. Dadurch werden die Ressourcen erstellt, die in der Konfiguration definiert sind.
   ```
    terraform apply
   ```
   {: pre}
   Es sollte eine Ausgabe ähnlich der folgenden Ausgabe angezeigt werden.![URL der Quellcodeverwaltung](images/solution10/created.png)
4. Als Nächstes müssen Sie Ihre [Infrastrukturgeräteliste](https://{DomainName}/classic/devices) aufrufen, um sicherzustellen, dass der Server erstellt wurde.![URL der Quellcodeverwaltung](images/solution10/configuration.png)

## {{site.data.keyword.cos_full_notm}}-Service hinzufügen und die Ressourcen skalieren

{: #modify}

In diesem Abschnitt erfahren Sie, wie Sie die virtuelle Serverressource skalieren und einen [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)-Service zu Ihrer Infrastrukturumgebung hinzufügen können.

1. Bearbeiten Sie die Datei `vm.tf`, um die folgenden Speicherbereiche zu erhöhen, und speichern Sie die Datei.
 - Anzahl der CPU-Kerne auf 4 Kerne erhöhen
 - RAM (Speicher) auf 4096 erhöhen
 - Plattengröße auf 100 GB erhöhen

2. Fügen Sie nun einen neuen [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)-Service hinzu. Erstellen Sie dazu eine neue Datei mit dem Namen **ibm-cloud-object-storage.tf**. Fügen Sie die folgenden Codeausschnitte zu der neu erstellten Datei hinzu. Diese Codeausschnitte erstellen einen Variablennamen für den Organisations- und Bereichsnamen. Diese beiden Variablennamen werden dann zum Abrufen der GUID des Bereichs verwendet, in dem der Service erstellt werden soll. Der Name des {{site.data.keyword.cos_full_notm}}-Service wird auf `lamp_objectstorage` gesetzt und anschließend benötigen Sie eine Bereichs-GUID, den vollständig qualifizierten Namen des Service und einen Plantyp. Mit dem Code unten wird ein Premium-Plan erstellt, da es sich hierbei sowieso um einen nutzungsabhängigen Plan handelt. Sie können auch den Lite-Plan verwenden, müssen aber berücksichtigen, dass der Lite-Plan auf nur einen Service pro Konto begrenzt ist.

   ```
   variable "org_name" {
     description = "Enter your IBM Cloud org name, you can get your org name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   variable "space_name" {
     description = "Enter your IBM Cloud space name, you can get your space name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   data "ibm_space" "space" {
     space = "${var.space_name}"
     org   = "${var.org_name}"
   }

   # ein Cloudobjektspeicher
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # Sie können nur einen Lite-Plan pro Konto haben, also verwenden Sie den Premium-Plan - es ist ein nutzungsabhängiger
     Plan = "Premium"
   }
   ```
   {: pre}
   **Hinweis:** Zu einem späteren Zeitpunkt werden Sie nach der Bezeichnung 'lamp_objectstorage' in den Protokollen suchen, um sicherzustellen, dass {{site.data.keyword.cos_full_notm}} erfolgreich erstellt wurde.

3. Initialisieren Sie die Terraform-Konfiguration erneut, indem Sie Folgendes ausführen:

   ```bash
    terraform init
   ```
   {: pre}

4. Wenden Sie die Terraform-Änderungen an, indem Sie Folgendes ausführen:
   ```bash
    terraform apply
   ```
   {: pre}
   **Hinweis:** Nachdem Sie den Befehl 'terraform apply' erfolgreich ausgeführt haben, sollte eine neue Datei `terraform.tfstate` zu Ihrem Verzeichnis hinzugefügt worden sein. Diese Datei enthält die vollständige Implementierungsbestätigung, mit der die zuletzt angewendeten und alle künftigen Änderungen an der Konfiguration verfolgt werden können. Wenn diese Datei entfernt wird oder verloren geht, verlieren Sie Ihre Terraform-Implementierungskonfigurationen.

## VM und {{site.data.keyword.cos_short}} überprüfen
{: #verifyvm}

In diesem Abschnitt überprüfen Sie die VM und die {{site.data.keyword.cos_short}}-Instanz, um sicherzustellen, dass sie erfolgreich erstellt wurden.

**VM überprüfen**

1. Klicken Sie im Menü auf der linken Seite auf **Infrastruktur**, um die Liste der virtuellen Servereinheiten anzuzeigen.
2. Klicken Sie auf **Geräte** -> **Geräteliste**, um den erstellten Server zu suchen. Ihre Servereinheit sollte dort aufgeführt sein.
3. Klicken Sie auf den Server, um weitere Informationen zur Serverkonfiguration anzuzeigen. Im Screenshot unten können Sie sehen, dass der Server erfolgreich erstellt wurde. ![URL der Quellencodeverwaltung](images/solution10/configuration.png)
4. Als Nächstes testen Sie den Server im Web-Browser. Öffnen Sie die öffentliche IP-Adresse des Servers im Web-Browser. Sie sollten die Standardinstallationsseite für den Server wie nachfolgend gezeigt sehen.![URL der Quellencodeverwaltung](images/solution10/LAMP.png)


**{{site.data.keyword.cos_full_notm}} überprüfen**

1. Im **{{site.data.keyword.Bluemix_notm}}-Dashboard** sollte eine Instanz des {{site.data.keyword.cos_full_notm}}-Service angezeigt werden, der für Sie erstellt wurde und betriebsbereit ist. ![object-storage](images/solution10/ibm-cloud-object-storage.png)

   Weitere Informationen zu {{site.data.keyword.cos_full_notm}} finden Sie [hier](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage).

## Ressourcen entfernen
{: #deleteresources}

Löschen Sie Ressourcen mit dem folgenden Befehl:
   ```bash
   terraform destroy
   ```
   {: pre}

**Hinweis:** Wenn Sie Ressourcen löschen möchten, benötigen Sie die Administratorberechtigung für die Infrastruktur. Wenn Sie über kein Konto eines Superusers mit Administratorberechtigung verfügen, fordern Sie an, die Ressourcen über das Infrastruktur-Dashboard zu löschen. Eine solche Anforderung können Sie im Infrastruktur-Dashboard unter den entsprechenden Geräten stellen. ![object-storage](images/solution10/rm.png)

## Zugehöriger Inhalt

- [Terraform](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [{{site.data.keyword.Bluemix_notm}}-Provider für Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [Bereitstellung statischer Dateien mithilfe eines CDN beschleunigen - {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

