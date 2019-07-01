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

# Bereitstellung statischer Dateien mit einem CDN beschleunigen
{: #static-files-cdn}

In diesem Lernprogramm erfahren Sie, wie Sie Website-Assets (Bilder, Videos, Dokumente) und vom Benutzer generierte Inhalte in einem {{site.data.keyword.cos_full_notm}} hosten und bereitstellen können und wie ein [{{site.data.keyword.cdn_full}} (CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai) für die schnelle und sichere Bereitstellung für Benutzer auf der ganzen Welt verwendet werden kann.

Webanwendungen enthalten verschiedene Arten von Inhalten: HTML-Inhalte, Bilder, Cascading Style Sheets, JavaScript-Dateien und von Benutzern generierte Inhalte. Manche Inhalte werden häufig geändert, andere weniger oft; manche werden sehr häufig aufgerufen, andere eher selten. Wenn der Benutzerkreis der Anwendung größer wird, kann es sinnvoll werden, die Bereitstellung dieser Inhalte an eine andere Komponente zu delegieren, um Ressourcen für Ihre Hauptanwendung freizugeben. Außerdem kann es hilfreich sein, diese Inhalte über einen Standort in der Nähe der jeweiligen Anwendungsbenutzer in verschiedenen Teilen der Welt bereitzustellen.

In solchen Situationen sprechen viele Gründe für die Verwendung eines CDN (Content Delivery Network):
* Im CDN werden die Inhalte zwischengespeichert, d. h. sie werden aus der Quelle (Ihrem Server) nur abgerufen, wenn sie nicht im Cache des Servers verfügbar oder wenn sie abgelaufen sind.
* Die Vielzahl von Rechenzentren auf der ganzen Welt ermöglichen dem CDN, die zwischengespeicherten Inhalte am nächstgelegenen Standort für die jeweiligen Benutzer bereitzustellen.
* Wenn der Browser in einer anderen Domäne ausgeführt wird als Ihre Hauptanwendung, können mehr Inhalte parallel geladen werden, da die meisten Browser nur eine begrenzte Anzahl von Verbindungen pro Hostname unterstützen.

## Ziele
{: #objectives}

* Dateien in einen {{site.data.keyword.cos_full_notm}}-Bucket hochladen
* Inhalte über ein Content Delivery Network (CDN) global verfügbar machen
* Dateien über eine Cloud Foundry-Webanwendung bereitstellen

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Produkte verwendet:
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur

<p style="text-align: center;">
![Architektur](images/solution3/Architecture.png)
</p>

1. Der Benutzer ruft die Anwendung auf.
2. Die Anwendung verfügt über Inhalte, die über ein Content Delivery Network verteilt werden.
3. Wenn die Inhalte nicht im CDN verfügbar oder abgelaufen sind, ruft das CDN die Inhalte aus der ursprünglichen Quelle ab.

## Vorbereitende Schritte
{: #prereqs}

Wenden Sie sich an den Masterbenutzer Ihres Infrastrukturkontos, um die folgenden Berechtigungen zu erhalten:
   * CDN-Konto verwalten
   * Speicher verwalten
   * API-Schlüssel

Diese Berechtigungen sind erforderlich, um die Speicher- und CDN-Services anzuzeigen und zu verwenden.

## Code der Webanwendung abrufen
{: #get_code}

Angenommen, es geht um eine einfache Webanwendung mit verschiedenartigen Inhalten wie Bilder, Videos und Cascading Style Sheets. Sie speichern die Inhalte in einem Speicherbucket und konfigurieren das CDN so, dass der Bucket als ursprüngliche Quelle verwendet wird.

Rufen Sie zunächst den Anwendungscode ab:

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## Objektspeicher erstellen
{: #create_cos}

{{site.data.keyword.cos_full_notm}} stellt flexiblen, kosteneffizienten und skalierbaren Cloudspeicher für unstrukturierte Daten bereit.

1. Rufen Sie den [Katalog](https://{DomainName}/catalog/) in der Konsole auf und wählen Sie im Abschnitt 'Speicher' die Option [**Objektspeicher**](https://{DomainName}/catalog/services/cloud-object-storage) aus.
2. Erstellen Sie eine neue {{site.data.keyword.cos_full_notm}}-Instanz.
4. Klicken Sie im Service-Dashboard auf **Bucket erstellen**.
5. Geben Sie einen eindeutigen Bucketnamen (z. B. `benutzername-meinewebsite`) an und klicken Sie auf **Erstellen**. Der Bucketname darf keine Punkte (.) enthalten.

## Dateien in einen Bucket hochladen
{: #upload}

In diesem Abschnitt laden Sie mit dem Befehlszeilentool **curl** Dateien in den Bucket hoch.

1. Melden Sie sich über die CLI bei {{site.data.keyword.Bluemix_notm}} an und fordern Sie bei IBM Cloud IAM ein Token an.
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. Kopieren Sie das Token aus der Ausgabe des Befehls im vorherigen Schritt.
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. Fügen Sie den Wert für das Token und den Bucketnamen in eine Umgebungsvariable ein, um den Zugriff zu erleichtern.
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. Laden Sie die Dateien mit den Namen `a-css-file.css`, `a-picture.png` und `a-video.mp4` aus dem Verzeichnis für Inhalte des Webanwendungscodes hoch, den Sie zuvor heruntergeladen haben. Laden Sie die Dateien in das Stammverzeichnis des Buckets hoch.
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. Zeigen Sie Ihre Dateien in Ihrem Dashboard an.
   ![](images/solution3/Buckets.png)
6. Rufen Sie die Dateien in Ihrem Browser über einen Link ähnlich dem folgenden Beispiel auf:

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## Dateien über ein CDN global verfügbar machen

In diesem Abschnitt erstellen Sie einen CDN-Service. Der CDN-Service verteilt Inhalte an die gewünschten Standorte. Beim ersten Abrufen eines Inhalts wird der Inhalt aus dem Host-Server (Ihr Bucket in {{site.data.keyword.cos_full_notm}}) in das Netz abgerufen und darin gespeichert, damit er für andere Benutzer schnell zugänglich ist (ohne die Netzlatenzzeit für das erneute Aufrufen des Host-Servers).

### CDN-Instanz erstellen

1. Rufen Sie den Katalog in der Konsole auf und wählen Sie **Content Delivery Network** (Netz zur Bereitstellung von Inhalten) im Abschnitt 'Netz' aus. Dieses CDN stammt von Akamai. Klicken Sie auf **Erstellen**.
2. Geben Sie im nächsten Dialogfenster Ihre angepasste Domäne als **Hostnamen** für das CDN an. Obwohl Sie eine angepasste Domäne angegeben haben, können Sie die Inhalte in dem CDN weiterhin über das von IBM bereitgestellte Element CNAME aufrufen. Dies bedeutet, wenn Sie keine angepasste Domäne verwenden möchten, können Sie einen beliebigen Namen festlegen.
3. Legen Sie einen eindeutigen Wert für das Präfix **Custom CNAME** (angepasster CNAME) fest.
4. Wählen Sie danach unter **Ursprung konfigurieren**, die Option **Object Storage** aus, um das CDN für COS zu konfigurieren.
5. Geben Sie als **Endpunkt** Ihren Bucket-API-Endpunkt (z. B. *s3-api.us-geo.objectstorage.softlayer.net*) an.
6. Lassen Sie die Felder für **Host-Header** und **Pfad** leer. Geben Sie *Ihren Bucketnamen* für **Bucketname** an.
7. Aktivieren Sie sowohl den HTTP-Port (80) als auch den HTTPS-Port (443).
8. Wählen Sie für **SSL-Zertifikat** die Option *DV-SAN-Zertifikat* aus, wenn Sie eine angepasste Domäne verwenden möchten. Wählen Sie andernfalls für den Zugriff auf den Speicher über CNAME die Option '**Platzhalterzertifikat*' aus.
9. Klicken Sie auf **Erstellen**.

### Eigene Inhalte über CDN-CNAME aufrufen

1. Wählen Sie die CDN-Instanz [in der Liste](https://{DomainName}/classic/network/cdn) aus.
2. Wenn Sie zuvor *DV-SAN-Zertifikat* ausgewählt haben, werden Sie zum Überprüfen der Domäne aufgefordert, sobald die Erstkonfiguration abgeschlossen ist. Folgen Sie den Schritten, die nach dem Klicken auf **Domänenüberprüfung anzeigen** angegeben werden.
3. In der Anzeige **Details** wird der **Hostname** und der **CNAME** für Ihr CDN angezeigt.
4. Rufen Sie Ihre Datei über `https://ihr-cdn-cname.cdnedge.bluemix.net/a-picture.png` auf oder (wenn Sie eine angepasste Domäne verwenden) über `https://ihr-cdn-hostname/a-picture.png`. Wenn Sie den Dateinamen nicht angeben, müsste stattdessen 'S3 ListBucketResult' angezeigt werden.

## Cloud Foundry-Anwendung bereitstellen

Die Anwendung enthält eine Website 'public/index.html' mit Verweisen auf die Dateien, die jetzt in {{site.data.keyword.cos_full_notm}} gehostet werden. Das Back-End `app.js` stellt diese Webseite bereit und ersetzt einen Platzhalter durch die tatsächliche Position Ihres CDN. Dadurch werden alle von der Webseite verwendeten Assets durch das CDN bereitgestellt.

1. Wechseln Sie von einem Terminal aus in das Verzeichnis, in das Sie den Code ausgecheckt haben:
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. Übertragen Sie die Anwendung mit einer Push-Operation, ohne sie zu starten:
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. Konfigurieren Sie die Umgebungsvariable CDN_NAME so, dass die App auf die CDN-Inhalte verweisen kann:
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. Starten Sie die App:
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. Rufen Sie die App in Ihrem Web-Browser auf. Die Formatvorlage der Seite, ein Bild und ein Video werden aus dem CDN geladen.

![](images/solution3/Application.png)

Die Verwendung eines CDN zusammen mit {{site.data.keyword.cos_full_notm}} ist eine leistungsfähige Kombination, mit der Sie Dateien hosten und für Benutzer auf der ganzen Welt bereitstellen können. Außerdem können Sie {{site.data.keyword.cos_full_notm}} verwenden, um Dateien zu speichern, die von den Benutzern in Ihre Anwendung hochgeladen werden.

## Ressourcen entfernen

* Löschen Sie die Cloud Foundry-Anwendung.
* Löschen Sie den Content Delivery Network-Service.
* Löschen Sie den {{site.data.keyword.cos_full_notm}}-Service oder -Bucket.

## Zugehörige Inhalte

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[Zugriff auf {{site.data.keyword.cos_full_notm}} verwalten](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[Einführung in CDN](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
