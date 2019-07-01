---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Umfassende Sicherheit auf eine Cloudanwendung anwenden
{: #cloud-e2e-security}

Ohne ein klares Verständnis potenzieller Sicherheitsrisiken und dem Schutz vor solchen Bedrohungen ist keine Anwendungsarchitektur vollständig. Anwendungsdaten sind eine kritische Ressource, die nicht verloren, beeinträchtigt oder gestohlen werden darf. Darüber hinaus sollten ruhende Daten und Daten bei der Übertragung durch Verschlüsselungsverfahren geschützt werden. Das Verschlüsseln von ruhenden Daten schützt Informationen auch dann vor einer Offenlegung, wenn die Daten verloren gehen oder gestohlen werden. Das Verschlüsseln von Daten bei der Übertragung (z. B. über das Internet) durch Methoden wie HTTPS, SSL und TLS verhindert das Ausspionieren und so genannte Man-in-the-Middle-Attacken.

Das Authentifizieren und Autorisieren des Zugriffs von Benutzern auf bestimmte Ressourcen ist eine weitere allgemeine Voraussetzung für viele Anwendungen. Dabei müssen möglicherweise unterschiedliche Authentifizierungsschemata unterstützt werden: Kunden und Lieferanten, die soziale Identitäten verwenden, Partner aus Cloud-gehosteten Verzeichnissen und Mitarbeiter des Identitätsproviders einer Organisation.

Dieses Lernprogramm führt Sie durch die wichtigsten Sicherheitsservices, die im {{site.data.keyword.cloud}}-Katalog verfügbar sind, und zeigt, wie Sie diese zusammen verwenden können. Eine Anwendung, die eine gemeinsame Dateinutzung ermöglicht, setzt Sicherheitskonzepte in die Praxis um.
{:shortdesc}

## Lernziele
{: #objectives}

* Inhalt in Speicherbuckets mit eigenen Verschlüsselungsschlüsseln verschlüsseln
* Benutzer auffordern, sich vor dem Zugriff auf eine Anwendung zu authentifizieren
* Sicherheitsbezogene API-Aufrufe und andere Aktionen in allen Cloud-Services überwachen und prüfen

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* Optional: [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

Für dieses Lernprogramm darf kein [Lite-Konto](https://{DomainName}/docs/account?topic=account-accounts#accounts) verwendet werden und es können Kosten entstehen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur
{: #architecture}

Das Lernprogramm bietet eine Beispielanwendung, mit der Gruppen von Benutzern Dateien in einen gemeinsamen Speicherpool hochladen und über gemeinsam nutzbare Links Zugriff auf diese Dateien erhalten können. Die Anwendung wird in Node.js geschrieben und als Docker-Container in {{site.data.keyword.containershort_notm}} implementiert. Dabei werden mehrere sicherheitsrelevante Services und Funktionen genutzt, um das Sicherheitsniveau der Anwendung zu verbessern.

<p style="text-align: center;">

  ![Architektur](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. Der Benutzer stellt eine Verbindung zur Anwendung her.
2. Wenn Sie eine angepasste Domäne und ein TLS-Zertifikat verwenden, wird das Zertifikat von den {{site.data.keyword.cloudcerts_short}} verwaltet und bereitgestellt.
3. {{site.data.keyword.appid_short}} sichert die Anwendung und leitet den Benutzer zu der Authentifizierungsseite um. Benutzer können sich auch anmelden.
4. Die Anwendung wird in einem Kubernetes-Cluster aus einem Image ausgeführt, das in der {{site.data.keyword.registryshort_notm}} gespeichert ist. Dieses Image wird automatisch auf Sicherheitslücken überprüft.
5. Hochgeladene Dateien werden in {{site.data.keyword.cos_short}} mit den zugehörigen Metadaten aus {{site.data.keyword.cloudant_short_notm}} gespeichert.
6. Dateispeicherbuckets nutzen einen vom Benutzer bereitgestellten Schlüssel zum Verschlüsseln von Daten.
7. Anwendungsmanagementaktivitäten werden von {{site.data.keyword.cloudaccesstrailshort}} protokolliert.

## Vorbereitende Schritte
{: #prereqs}

1. Installieren Sie alle erforderlichen Befehlszeilentools (CLI), indem Sie [die folgenden Schritte ausführen](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
2. Stellen Sie sicher, dass Sie über die neueste Version der Plug-ins verfügen, die in diesem Lernprogramm verwendet werden. Verwenden Sie ggf. `ibmcloud plugin update --all`, um ein Upgrade durchzuführen.

## Services erstellen
{: #setup}

### Entscheiden, wo die Anwendung implementiert werden soll

1. Geben Sie die **Position**, die **Cloud Foundry-Organisation und den Bereich** und die **Ressourcengruppe** an, in der Sie die Anwendung und ihre Ressourcen implementieren möchten.

### Benutzer- und Anwendungsaktivitäten erfassen 
{: #activity-tracker }

Der {{site.data.keyword.cloudaccesstrailshort}}-Service zeichnet vom Benutzer initiierte Aktivitäten auf, die den Status eines Service in {{site.data.keyword.Bluemix_notm}} ändern. Am Ende dieses Lernprogramms werden Sie die Ereignisse überprüfen, die durch die Ausführung der Schritte des Lernprogramms generiert wurden.

1. Greifen Sie auf den {{site.data.keyword.Bluemix_notm}}-Katalog zu und erstellen Sie eine Instanz von [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker). Beachten Sie, dass es nur eine Instanz von {{site.data.keyword.cloudaccesstrailshort}} pro Cloud Foundry-Bereich geben kann.
2. Nachdem die Instanz erstellt wurde, ändern Sie ihren Namen in **secure-file-storage-activity-tracker**.
3. Um alle Activity Tracker-Ereignisse anzuzeigen, müssen Sie sicherstellen, dass Sie über die entsprechenden Berechtigungen verfügen:
   1. Wechseln Sie zu [Identität & Zugriff > Benutzer](https://{DomainName}/iam/#/users).
   2. Wählen Sie Ihren Benutzernamen aus der Liste aus.
   3. Falls eine entsprechende Richtlinie fehlt, erstellen Sie unter **Zugriffsrichtlinien** eine Richtlinie für den {{site.data.keyword.loganalysisshort_notm}}-Service mit der Rolle **Anzeigeberechtigter** an der Position, an der der Service erstellt wurde.
   4. Stellen Sie unter **Cloud Foundry-Zugriff** sicher, dass Sie über die Rolle **Entwickler** in dem Cloud Foundry-Bereich verfügen, in dem {{site.data.keyword.cloudaccesstrailshort}} bereitgestellt wurde. Ist dies nicht der Fall, arbeiten Sie mit dem Cloud Foundry-Manager Ihrer Organisation zusammen, um die Rolle zuzuordnen.

Detaillierte Anweisungen zum Einrichten von Berechtigungen finden Sie in der [Dokumentation zu {{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy).
{: tip}

### Cluster für die Anwendung erstellen

{{site.data.keyword.containershort_notm}} stellt eine Umgebung bereit, in der hoch verfügbare Anwendungen in Docker-Containern implementiert werden, die in Kubernetes-Clustern ausgeführt werden.

Überspringen Sie diesen Abschnitt, wenn Sie über einen vorhandenen **Standard**-Cluster verfügen, den Sie in diesem Lernprogramm wiederverwenden möchten.
{: tip}

1. Greifen Sie auf die [Seite zum Erstellen von Clustern](https://{DomainName}/containers-kubernetes/catalog/cluster/create) zu.
   1. Setzen Sie die **Position** auf die in den vorherigen Schritten verwendete Position.
   2. Setzen Sie **Clustertyp** auf **Standard**.
   3. Setzen Sie **Verfügbarkeit** auf **Einzelne Zone**.
   4. Wählen Sie eine **Masterzone** aus.
2. Belassen Sie die Standardeinstellungen für **Kubernetes-Version** und **Hardware-Isolation**.
3. Wenn Sie vorhaben, nur dieses Lernprogramm für diesen Cluster zu implementieren, legen Sie den Wert für **Workerknoten** auf **1** fest.
4. Setzen Sie den **Clusternamen** auf **secure-file-storage-cluster**.
5. Klicken Sie auf die Schaltfläche **Cluster erstellen**.

Während der Cluster bereitgestellt wird, erstellen Sie die anderen Services, die für das Lernprogramm erforderlich sind.

### Eigene Verschlüsselungsschlüssel verwenden

Mit {{site.data.keyword.keymanagementserviceshort}} können Sie verschlüsselte Schlüssel für Apps in {{site.data.keyword.Bluemix_notm}}-Services bereitstellen. {{site.data.keyword.keymanagementserviceshort}} und {{site.data.keyword.cos_full_notm}} [arbeiten zusammen, um Ihre ruhenden Daten zu schützen](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos). In diesem Abschnitt erstellen Sie einen Rootschlüssel für das Speicherbucket.

1. Erstellen Sie eine Instanz von [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms).
   * Setzen Sie den Namen auf **secure-file-storage-kp**.
   * Wählen Sie die Ressourcengruppe aus, in der die Serviceinstanz erstellt werden soll.
2. Klicken Sie unter **Verwalten** auf die Schaltfläche **Schlüssel hinzufügen**, um einen neuen Rootschlüssel zu erstellen. Dieser wird verwendet, um den Inhalt des Speicherbuckets zu verschlüsseln.
   * Setzen Sie den Namen auf **secure-file-storage-root-enckey**.
   * Setzen Sie den Schlüsseltyp auf **Rootschlüssel**.
   * Anschließend **generieren Sie den Schlüssel**.

Generieren und verwalten Sie Ihren eigenen Schlüssel (BYOK, Bring your own key), indem Sie [einen vorhandenen Rootschlüssel importieren](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys).
{: tip}

### Speicher für Benutzerdateien einrichten

Die Anwendung für die gemeinsame Nutzung von Dateien speichert Dateien in einem {{site.data.keyword.cos_short}}-Bucket. Die Beziehung zwischen Dateien und Benutzern wird als Metadaten in einer {{site.data.keyword.cloudant_short_notm}}-Datenbank gespeichert. In diesem Abschnitt erstellen und konfigurieren Sie diese Services.

#### Ein Bucket für den Inhalt

1. Erstellen Sie eine Instanz von [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
   * Setzen Sie den **Namen** auf **secure-file-storage-cos**.
   * Verwenden Sie die gleiche **Ressourcengruppe** wie für die vorherigen Services.
2. Erstellen Sie unter **Serviceberechtigungsnachweise** einen *neuen Berechtigungsnachweis*.
   * Setzen Sie den **Namen** auf **secure-file-storage-cos-acckey**.
   * Setzen Sie **Rolle** auf **Autor**.
   * Geben Sie keine **Service-ID** an.
   * Setzen Sie **Inline-Konfigurationsparameter** auf **{"HMAC":true}**. Dies ist erforderlich, um bereits signierte URLs zu generieren.
   * Klicken Sie auf **Hinzufügen**.
   * Notieren Sie sich die Berechtigungsnachweise, indem Sie auf **Berechtigungsnachweise anzeigen** klicken. Sie benötigen diese in einem späteren Schritt.
3. Klicken Sie im Menü auf **Endpunkt**: Setzen Sie **Ausfallsicherheit** auf **Regional** und **Position** auf die Zielposition. Kopieren Sie den **öffentlichen** Serviceendpunkt. Dieser wird später bei der Konfiguration der Anwendung verwendet.

Vor dem Erstellen des Buckets erteilen Sie dem Rootschlüssel, der in **secure-file-storage-kp** gespeichert ist, Zugriff des Typs **secure-file-storage-cos**.

1. Rufen Sie [Identität & Zugriff > Autorisierungen](https://{DomainName}/iam/#/authorizations) in der {{site.data.keyword.cloud_notm}}-Konsole auf.
2. Klicken Sie auf die Schaltfläche **Erstellen**.
3. Wählen Sie im Menü **Quellenservice** die Option **Cloud Object Storage** aus.
4. Wählen Sie im Menü **Quellenserviceinstanz** den zuvor erstellten Service **secure-file-storage-cos** aus.
5. Wählen Sie im Menü **Zielservice** die Option **Key Protect** aus.
6. Wählen Sie im Menü **Zielserviceinstanz** den Service **secure-file-storage-kp** für die Autorisierung aus.
7. Aktivieren Sie die Rolle **Leser**.
8. Klicken Sie auf die Schaltfläche **Autorisieren**.

Erstellen Sie schließlich das Bucket.

1. Greifen Sie auf die **secure-file-storage-cos**-Serviceinstanz über die [Ressourcenliste](https://{DomainName}/resources) zu.
2. Klicken Sie auf **Bucket erstellen**.
   1. Setzen Sie den **Namen** auf einen eindeutigen Wert, z. B. **&lt;ihre-initialen&gt;-secure-file-upload**.
   2. Setzen Sie **Ausfallsicherheit** auf **Regional**.
   3. Setzen Sie den **Standort** auf den Standort, an dem Sie auch den **secure-file-storage-kp**-Service erstellt haben.
   4. Setzen Sie **Speicherklasse** auf **Standard** .
3. Aktivieren Sie das Kontrollkästchen zum **Hinzufügen von Key Protect-Schlüsseln**.
   1. Wählen Sie den **secure-file-storage-kp**-Service aus.
   2. Wählen Sie **secure-file-storage-root-enckey** als Schlüssel aus.
4. Klicken Sie auf **Bucket erstellen**.

#### Eine Datenbank ordnet Beziehungen zwischen Benutzern und ihren Dateien zu

Die {{site.data.keyword.cloudant_short_notm}}-Datenbank enthält Metadaten für alle Dateien, die von der Anwendung hochgeladen werden.

1. Erstellen Sie eine Instanz von [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB).
   * Setzen Sie den **Namen** auf **secure-file-storage-cloudant**.
   * Legen Sie den Standort fest.
   * Verwenden Sie die gleiche **Ressourcengruppe** wie für die vorherigen Services.
   * Setzen Sie **Verfügbare Authentifizierungsmethoden** auf **Nur IAM verwenden**.
2. Erstellen Sie unter **Serviceberechtigungsnachweise** einen *neuen Berechtigungsnachweis*.
   * Setzen Sie den **Namen** auf **secure-file-storage-cloudant-acckey**.
   * Setzen Sie **Rolle** auf **Manager**.
   * Behalten Sie die Standardwerte für die *optionalen* Felder bei.
   * Klicken Sie auf **Hinzufügen**.
3. Notieren Sie sich die Berechtigungsnachweise, indem Sie auf **Berechtigungsnachweise anzeigen** klicken. Sie benötigen diese in einem späteren Schritt.
4. Starten Sie unter **Verwalten** das Cloudant-Dashboard.
5. Klicken Sie auf **Datenbank erstellen**, um eine Datenbank mit dem Namen **secure-file-storage-metadata** zu erstellen.

### Benutzer authentifizieren

Mit {{site.data.keyword.appid_short}} können Sie Ressourcen sichern und die Authentifizierung zu Ihren Anwendungen hinzufügen. {{site.data.keyword.appid_short}} kann mit {{site.data.keyword.containershort_notm}} [integriert](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth) werden, um Benutzer zu authentifizieren, die auf im Cluster bereitgestellte Anwendungen zugreifen.

1. Erstellen Sie eine Instanz von [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID).
   * Setzen Sie den **Servicenamen** auf **secure-file-storage-appid**.
   * Verwenden Sie denselben **Standort** und dieselbe **Ressourcengruppe** wie für die vorherigen Services.
2. Fügen Sie unter **Identitätsprovider/Verwalten** auf der Registerkarte **Authentifizierungseinstellungen** eine **Web-Weiterleitungs-URL** hinzu, die auf die Domäne verweist, die Sie für die Anwendung verwenden werden. Wenn die Ingress-Unterdomäne Ihres Clusters beispielsweise
`<cluster-name>.us-south.containers.appdomain.cloud` ist, lautet die Weiterleitungs-URL `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback`. Für {{site.data.keyword.appid_short}} ist es erforderlich, dass die Web-Weiterleitungs-URL das Protokoll **HTTPS** verwendet. Sie können Ihre Ingress-Unterdomäne im Cluster-Dashboard oder mit `ibmcloud ks cluster-get <cluster-name>` anzeigen.

Sie sollten die verwendeten Identitätsprovider sowie die Optionen für die Anmeldung und Benutzerverwaltung im {{site.data.keyword.appid_short}}-Dashboard anpassen. In diesem Lernprogramm werden zur Vereinfachung die Standardwerte verwendet. In einer Produktionsumgebung sollten Sie die Verwendung der Mehrfaktorauthentifizierung (MFA) und von erweiterten Kennwortregeln in Betracht ziehen.
{: tip}

## App implementieren

Alle Services sind konfiguriert. In diesem Abschnitt implementieren Sie nun die Anwendung 'tutorial' im Cluster.

### Code abrufen

1. Rufen Sie den Code der Anwendung ab:
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. Wechseln Sie in das Verzeichnis **secure-file-storage**:
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### Docker-Image erstellen

1. [Erstellen Sie das Docker-Image](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating) in {{site.data.keyword.registryshort_notm}}.
   - Suchen Sie den Registry-Endpunkt mit `ibmcloud cr info`, wie z. B. **us**.icr.io oder **uk**.icr.io.
   - Erstellen Sie einen Namensbereich zum Speichern des Container-Image.
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - Verwenden Sie **secure-file-storage** als Imagenamen.

   ```sh
   ibmcloud cr build -t <position>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### Berechtigungsnachweise und Konfigurationseinstellungen angeben

1. Kopieren Sie `credentials.template.env` in `credentials.env`:
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. Bearbeiten Sie `credentials.env` und füllen Sie die Leerstellen mit den folgenden Werten aus:
   * dem regionalen Endpunkt des {{site.data.keyword.cos_short}}-Service, dem Bucketnamen, den für **secure-file-storage-cos** erstellten Berechtigungsnachweisen
   * und den Berechtigungsnachweisen für **secure-file-storage-cloudant**.
3. Kopieren Sie `secure-file-storage.template.yaml` in `secure-file-storage.yaml`:
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. Bearbeiten Sie `secure-file-storage.yaml` und ersetzen Sie die Platzhalter (`$IMAGE_PULL_SECRET`, `$REGISTRY_URL`, `$REGISTRY_NAMESPACE`, `$IMAGE_NAME`, `$TARGET_NAMESPACE`, `$INGRESS_SUBDOMAIN`, `$INGRESS_SECRET`) mit den korrekten Werten. Beispiel (unter der Annahme, dass die Anwendung im Kubernetes-Standardnamensbereich _default_ implementiert wird):

| Variable | Wert  | Beschreibung|
| -------- | ----- | ----------- |
| `$IMAGE_PULL_SECRET` | In der .yaml-Datei kommentierte Zeilen beibehalten | Ein geheimer Schlüssel für den Zugriff auf die Registry.  |
| `$REGISTRY_URL` | *us.icr.io* | Die Registry, in der das Image im vorherigen Abschnitt erstellt wurde. |
| `$REGISTRY_NAMESPACE` | *secure-file-storage-namespace* | Der Namensbereich der Registry, in dem das Image im vorherigen Abschnitt erstellt wurde. |
| `$IMAGE_NAME` | *secure-file-storage* | Der Name des Docker-Image. |
| `$TARGET_NAMESPACE` | *default* | Der Kubernetes-Namensbereich, in den die App übertragen werden soll. |
| `$INGRESS_SUBDOMAIN` | *secure-file-stora-123456.us-south.containers.appdomain.cloud* | Wird von der Übersichtsseite des Clusters oder mit `ibmcloud ks cluster-get secure-file-storage-cluster` abgerufen. |
| `$INGRESS_SECRET` | *secure-file-stora-123456* | Wird von der Übersichtsseite des Clusters oder mit `ibmcloud ks cluster-get secure-file-storage-cluster` abgerufen. |

`$IMAGE_PULL_SECRET` ist nur erforderlich, wenn ein anderer Kubernetes-Namensbereich als der Standardnamensbereich verwendet werden soll. Dies erfordert eine zusätzliche Kubernetes-Konfiguration (z. B. das [Erstellen eines geheimen Schlüssels für die Docker-Registry in dem neuen Namensbereich](https://{DomainName}/docs/containers?topic=containers-images#other)).
{: tip}

### Implementierung im Cluster

1. Rufen Sie die Clusterkonfiguration ab und legen Sie die Umgebungsvariable KUBECONFIG fest.
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. Erstellen Sie den geheimen Schlüssel, der von der Anwendung verwendet wird, um Serviceberechtigungsnachweise abzurufen:
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. Binden Sie die {{site.data.keyword.appid_short_notm}}-Serviceinstanz an den Cluster.
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
   Wenn Sie über mehrere Services mit demselben Namen verfügen, schlägt der Befehl fehl. Sie sollten die Service-GUID anstelle des Namens übergeben. Um die GUID eines Service zu suchen, verwenden Sie `ibmcloud resource service-instance secure-file-storage-appid`.
   {: tip}
4. Implementieren Sie die App.
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## Anwendung testen

Sie können unter `https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/` auf die Anwendung zugreifen.

1. Rufen Sie die Startseite der Anwendung auf. Sie werden zur Standardanmeldeseite von {{site.data.keyword.appid_short_notm}} weitergeleitet.
2. Registrieren Sie sich mit einer gültigen E-Mail-Adresse für ein neues Konto.
3. Warten Sie auf die E-Mail in Ihrem Posteingang, um das Konto zu bestätigen.
4. Melden Sie sich an.
5. Wählen Sie eine Datei zum Hochladen aus. Klicken Sie auf **Hochladen**.
6. Verwenden Sie die Aktion **Freigeben** für eine Datei, um eine bereits signierte URL zu generieren, die gemeinsam mit anderen Personen für den Zugriff auf die Datei genutzt werden kann. Der Link ist so festgelegt, dass er nach fünf Minuten abläuft.

Authentifizierte Benutzer verfügen über eigene Bereiche zum Speichern von Dateien. Obwohl sie keine anderen Dateien sehen können, können sie bereits signierte URLs generieren, um temporären Zugriff auf eine bestimmte Datei zu gewähren.

Weitere Details zu der Anwendung finden Sie im [Quellcode-Repository](https://github.com/IBM-Cloud/secure-file-storage).

## Sicherheitsereignisse überprüfen
Nachdem die Anwendung und ihre Services erfolgreich implementiert wurden, können Sie die Sicherheitsereignisse, die von diesem Prozess generiert wurden, überprüfen. Alle Ereignisse sind zentral in der {{site.data.keyword.cloudaccesstrailshort}}-Instanz verfügbar und können über die [grafische Benutzerschnittstelle (Kibana), CLI oder API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status) aufgerufen werden.

1. Suchen Sie in der [{{site.data.keyword.Bluemix_notm}}-Ressourcenliste](https://{DomainName}/resources) die {{site.data.keyword.cloudaccesstrailshort}}-Instanz **secure-file-storage-activity-tracker** und öffnen Sie das zugehörige Dashboard.
2. Auf der Registerkarte **Verwalten** werden standardmäßig **Bereichsprotokolle** angezeigt. Wechseln Sie zu **Kontoprotokolle**, indem Sie neben **Protokolle anzeigen** auf den Selektor klicken. Es sollten mehrere Ereignisse angezeigt werden.
3. Klicken Sie auf **In Kibana anzeigen**, um die vollständige Ereignisanzeige zu öffnen.
4. Überprüfen Sie die Ereignisdetails, indem Sie auf **Erkennen** klicken.
5. Fügen Sie aus **Verfügbare Felder** die Werte **action_str** und **initiator.name_str** hinzu.
6. Erweitern Sie interessante Einträge, indem Sie auf das Dreieckssymbol klicken und dann ein Tabellen- oder JSON-Format auswählen.

Sie können die Einstellungen für die automatische Aktualisierung und den angezeigten Zeitbereich und dadurch die Art und Weise ändern, wie und welche Daten analysiert werden.
{: tip}

## Optional: Angepasste Domäne verwenden und Netzverkehr verschlüsseln
Standardmäßig ist die Anwendung über einen generischen Hostnamen an einer Unterdomäne von `containers.appdomain.cloud` zugänglich. Es ist jedoch auch möglich, eine angepasste Domäne mit der implementierten App zu verwenden. Für die weitere Unterstützung von **https**, dem Zugriff mit verschlüsseltem Netzverkehr, muss entweder ein Zertifikat für den gewünschten Hostnamen oder ein Platzhalterzertifikat bereitgestellt werden. Im folgenden Abschnitt laden Sie ein vorhandenes Zertifikat in {{site.data.keyword.cloudcerts_short}} hoch und stellen es im Cluster bereit. Außerdem werden Sie die App-Konfiguration für die Verwendung der angepassten Domäne aktualisieren.

Ein Beispiel für das Abrufen eines Zertifikats von [Let's Encrypt](https://letsencrypt.org/) ist in dem folgenden [{{site.data.keyword.cloud}} Blog](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/) beschrieben.
{: tip}

1. Erstellen Sie eine Instanz von [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager).
   * Setzen Sie den Namen auf **secure-file-storage-certmgr**.
   * Verwenden Sie denselben **Standort** und dieselbe **Ressourcengruppe** wie für die anderen Services.
2. Klicken Sie auf **Zertifikat importieren**, um das Zertifikat zu importieren.
   * Setzen Sie den Namen auf **SecFileStorage** und die Beschreibung auf **Zertifikat für e2e-Sicherheitslernprogramm**.
   * Laden Sie die Zertifikatsdatei mithilfe der Schaltfläche **Durchsuchen** hoch.
   * Klicken Sie auf **Importieren**, um den Importprozess abzuschließen.
3. Suchen Sie den Eintrag für das importierte Zertifikat und erweitern Sie diesen.
   * Überprüfen Sie den Zertifikatseintrag beispielsweise darauf, ob der Domänenname mit Ihrer benutzerdefinierten Domäne übereinstimmt. Wenn Sie ein Platzhalterzertifikat hochgeladen haben, wird ein Stern in den Domänennamen eingeschlossen.
   * Klicken Sie neben dem **CRN** des Zertifikats auf das Symbol **Kopieren**.
4. Wechseln Sie zur Befehlszeile, um die Zertifikatsinformationen als geheimen Schlüssel für den Cluster bereitzustellen. Führen Sie nach dem Kopieren des CRN aus dem vorherigen Schritt den folgenden Befehl aus.
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <der kopierte crn aus dem vorherigen schritt>
   ```
   {: codeblock}
   Stellen Sie sicher, dass der Cluster Kenntnis über das Zertifikat Bescheid hat, indem Sie den folgenden Befehl ausführen.
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. Bearbeiten Sie die Datei `secure-file-storage.yaml`.
   * Suchen Sie den Abschnitt für **Ingress**.
   * Entfernen Sie die Kommentarzeichen, bearbeiten Sie die Zeilen zu angepassten Domänen und geben Sie Ihren Domänen- und Hostnamen an.
   Der CNAME-Eintrag für Ihre angepasste Domäne muss auf den Cluster verweisen. Weitere Details finden Sie in der Dokumentation in diesem [Leitfaden zum Zuordnen von angepassten Domänen](https://{DomainName}/docs/containers?topic=containers-ingress#private_3).
   {: tip}
6. Wenden Sie die Konfigurationsänderungen auf den bereitgestellten Container an:
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. Wechseln Sie zurück zum Browser. Suchen Sie in der [{{site.data.keyword.Bluemix_notm}}-Ressourcenliste](https://{DomainName}/resources) den zuvor erstellten und konfigurierten {{site.data.keyword.appid_short}}-Service und starten Sie dessen Management-Dashboard.
   * Rufen Sie unter **Identitätsprovider** die Option **Verwalten** und anschließend **Einstellungen** auf.
   * Fügen Sie im Formular **Web-Weiterleitungs-URLs hinzufügen** die URL `https://secure-file-storage.<your custom domain>/appid_callback` als weitere URL hinzu.
8. Alles sollte jetzt ordnungsgemäß eingerichtet sein. Testen Sie die App, indem Sie auf sie in der konfigurierten angepassten Domäne `https://secure-file-storage.<your custom domain>` zugreifen.

## Lernprogramm erweitern

Die Sicherheit kann immer optimiert werden. Folgen Sie den folgenden Empfehlungen, um die Sicherheit Ihrer Anwendung zu erhöhen.

* Verwenden Sie [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights), um statische und dynamische Codeüberprüfungen auszuführen.
* Stellen Sie sicher, dass nur der qualitativ hochwertiger Code freigegeben wird, indem Sie Richtlinien und Regeln in [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) verwenden.

## Ressourcen entfernen
{:removeresources}

Um die Ressource zu entfernen, löschen Sie den implementierten Container und anschließend die bereitgestellten Services.

1. Löschen Sie den implementierten Container:
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. Löschen Sie die geheimen Schlüssel für die Implementierung:
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. Entfernen Sie das Docker-Image aus der Container-Registry:
   ```sh
   ibmcloud cr image-rm <standort>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. Suchen Sie in der [{{site.data.keyword.Bluemix_notm}}-Ressourcenliste](https://{DomainName}/resources) nach den Ressourcen, die für dieses Lernprogramm erstellt wurden. Verwenden Sie das Suchfeld und **secure-file-storage** als Muster. Löschen Sie jeden der Services, indem Sie auf das Kontextmenü neben den einzelnen Services klicken und **Service löschen** auswählen. Beachten Sie, dass der {{site.data.keyword.keymanagementserviceshort}}-Service erst entfernt werden kann, nachdem der Schlüssel gelöscht wurde. Klicken Sie auf die Serviceinstanz, um zum zugehörigen Dashboard zu gelangen und den Schlüssel zu löschen.

Wenn Sie ein Konto mit anderen Benutzern gemeinsam nutzen, stellen Sie immer sicher, dass nur Ihre eigenen Ressourcen gelöscht werden.
{: tip}

## Zugehöriger Inhalt
{:related}

* [{{site.data.keyword.security-advisor_short}}-Dokumentation](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [Sicherheitskomponenten zum Absichern und Überwachen Ihrer Cloud-Apps](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [{{site.data.keyword.Bluemix_notm}}-Plattformsicherheit](https://{DomainName}/docs/overview?topic=overview-security#security)
* [Sicherheit in IBM Cloud](https://www.ibm.com/cloud/security)
* [Lernprogramm: Bewährte Verfahren für die Organisation von Benutzern, Teams, Anwendungen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [Apps in IBM Cloud mit Platzhalterzertifikaten schützen](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
