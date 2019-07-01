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


# Serverunabhängige Webanwendung und API
{: #serverless-api-webapp}

In diesem Lernprogramm erstellen Sie eine serverunabhängige Webanwendung, indem Sie statische Websiteinhalte auf GitHub-Seiten hosten und das Back-End der Anwendung mit {{site.data.keyword.openwhisk}} bereitstellen.

Die ereignisgesteuerte Plattform {{site.data.keyword.openwhisk_short}} bietet Unterstützung für eine [Vielzahl von Anwendungsfällen](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases). Die Erstellung von Webanwendungen und APIs ist einer der unterstützten Anwendungsfälle. Beim Arbeiten mit Webanwendungen sind Ereignisse (HTTP-Anforderungen) die Interaktionen zwischen den Web-Browsern (oder REST-Clients) und Ihrer Web-App. Anstatt eine virtuelle Maschine, einen Container oder eine Cloud Foundry-Laufzeit für die Implementierung Ihres Back-Ends bereitzustellen, können Sie Ihre Back-End-API mit einer serverunabhängigen Plattform implementieren. Diese hilfreiche Lösung vermeidet Kosten für Leerlaufzeit und ermöglicht die bedarfsgerechte Skalierung der Plattform.

Jede Aktion (oder Funktion) in {{site.data.keyword.openwhisk_short}} kann in einen HTTP-Endpunkt umgesetzt werden, der von Web-Clients verarbeitet werden kann. Aktionen, die für das Web aktiviert sind, werden als *Webaktionen* bezeichnet. Vorhandene Webaktionen können zu einer vollständigen API mit API-Gateway zusammengefasst werden. Das API-Gateway ist eine Komponente von {{site.data.keyword.openwhisk_short}} zum Bereitstellen von APIs. Es beinhaltet Sicherheit, OAuth-Unterstützung, Tarifbegrenzung und Unterstützung für angepasste Domänen.

## Ziele

* Serverunabhängiges Back-End und eine Datenbank bereitstellen
* Eine REST-API zugänglich machen
* Eine statische Website hosten
* Optional: Eine angepasste Domäne für die REST-API verwenden

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur
{: #architecture}

Die in diesem Lernprogramm gezeigte Anwendung ist eine Gästebuch-Website, in der Benutzer Nachrichten posten können.

<p style="text-align: center;">

   ![Architektur](./images/solution8/Architecture.png)
</p>

1. Der Benutzer greift auf die Anwendung zu, die in GitHub Pages gehostet wird.
2. Die Webanwendung ruft eine Back-End-API auf.
3. Die Back-End-API wird im API-Gateway definiert.
4. Das API-Gateway leitet die Anforderung an [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) weiter.
5. Die {{site.data.keyword.openwhisk_short}}-Aktionen verwenden [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) zum Speichern und Abrufen von Gästebucheinträgen.

## Vorbereitende Schritte
{: #prereqs}

In dieser Anleitung wird GitHub Pages zum Hosten der statischen Website verwendet. Stellen Sie sicher, dass Sie über ein öffentliches GitHub-Konto verfügen.

## Datenbank 'guestbook' für Gästebuch erstellen

Erstellen Sie zu Beginn eine {{site.data.keyword.cloudant_short_notm}}-Instanz. {{site.data.keyword.cloudant_short_notm}} ist eine vollständig verwaltete Datenebene, die für moderne Webanwendungen und mobile Anwendungen konzipiert ist und ein flexibles JSON-Schema nutzt. {{site.data.keyword.cloudant_short_notm}} basiert auf Apache CouchDB, ist mit Apache CouchDB kompatibel und kann über eine sichere HTTPS-API aufgerufen werden, die entsprechend dem Wachstum Ihrer Anwendung skaliert werden kann.

1. Wählen Sie im Katalog den Eintrag **Cloudant** aus.
2. Geben Sie **guestbook-db** als Servicenamen an, wählen Sie als Authentifizierungsmethode **Traditionelle Berechtigungsnachweise und IAM-Berechtigungsnachweise verwenden** aus und klicken Sie auf **Erstellen**.
3. Rufen Sie wieder die Ressourcenliste auf und klicken Sie auf den Listeneintrag ***guestbook-db** in der Spalte 'Name'. Hinweis: Sie müssen gegebenenfalls warten, bis der Service bereitgestellt ist. 
4. Klicken Sie in der Anzeige für Servicedetails auf ***Cloudant-Dashboard starten***, um das Dshboard in einer neuen Browserregisterkarte zu öffnen. Hinweis: Sie müssen sich gegebenenfalls bei Ihrer Cloudant-Instanz anmelden. 
5. Klicken Sie auf ***Datenbank erstellen*** und erstellen Sie eine Datenbank mit dem Namen ***guestbook*** (Gästebuch).

  ![](images/solution8/Create_Database.png)

6. Rufen Sie wieder die Registerkarte für Servicedetails unter **Serviceberechtigungsnachweise** auf.
   1. Erstellen Sie einen **neuen Berechtigungsnachweis**, übernehmen Sie die Standardeinstellungen und klicken Sie auf **Hinzufügen**.
   2. Klicken Sie unter 'Aktionen' auf **Berechtigungsnachweise anzeigen**. Diese Berechtigungsnachweise werden später benötigt, damit Cloud Functions-Aktionen in Ihrem Cloudant-Service lesen bzw. schreiben können.

## Serverunabhängige Aktionen erstellen

In diesem Abschnitt erstellen Sie serverunabhängige Aktionen (häufig als 'Funktionen' bezeichnet). {{site.data.keyword.openwhisk}} (basierend auf Apache OpenWhisk) ist eine FaaS-Plattform (FaaS = Function-as-a-Service) zum Ausführen von Funktionen als Antwort auf eingehende Ereignisse. Sie verursacht keine Kosten, solange sie nicht verwendet wird.

![](images/solution8/Functions.png)

### Folge von Aktionen zum Speichern des Gästebucheintrags

Sie erstellen eine **Folge**, d. h. eine Kette von Aktionen, bei der die Ausgabe einer Aktion jeweils als Eingabe für die nächste Aktion verwendet wird. Die erste Folge, die Sie erstellen, wird dazu verwendet, eine Gastnachricht zu speichern. Wenn für die Folge ein Name, eine E-Mail-ID und ein Kommentar angegeben wird, führt sie Folgendes aus:
   * Ein Dokument erstellen, das als persistent definiert werden soll.
   * Das Dokument in der {{site.data.keyword.cloudant_short_notm}}-Datenbank speichern.

Erstellen Sie zunächst die erste Aktion:

1. Rufen Sie **Funktionen** https://{Domänenname}/openwhisk auf.
2. Klicken Sie im linken Anzeigebereich auf **Aktionen** und anschließend auf **Erstellen**.
3. Wählen Sie **Aktion erstellen** aus, geben Sie für die Aktion den Namen `eintrag-zum-speichern-vorbereiten` an und wählen Sie **Node.js** als Laufzeit aus. (Hinweis: Wählen Sie die aktuelle Version aus.)
4. Ersetzen Sie den vorhandenen Code durch das folgende Code-Snippet:
   ```js
   /**
    * Zu speichernden Gästebucheintrag vorbereiten
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. Aktivieren Sie die Option **Speichern**.

Fügen Sie anschließend die Aktion zu einer Folge hinzu:

1. Klicken Sie auf **Einschließende Folgen** und danach auf **Zu Folge hinzufügen**.
1. Geben Sie als Namen für die Folge `folge-zum-speichern-eines gästebucheintrags` ein und klicken Sie auf **Erstellen und hinzufügen**.

Fügen Sie schließlich eine zweite Aktion zu der Folge hinzu:

1. Klicken Sie auf den Eintrag **folge-zum-speichern-eines-gästebucheintrags** und danach auf **Hinzufügen**.
1. Wählen Sie **Öffentlich verwenden**, **Cloudant** aus und danach den Eintrag **dokument-erstellen** unter **Aktionen**.
1. Erstellen Sie eine **neue Bindung**. 
1. Geben Sie den Namen `bindung-für-gästebuch` ein.
1. Wählen Sie für die Cloudant-Instanz die Option `Eigene Berechtigungsnachweise eingeben` aus und geben Sie in die folgenden Felder die Berechtigungsnachweisinformationen ein, die für Ihren Cloudant-Service erfasst wurden: Benutzername, Kennwort, Host und Datenbank = `guestbook`. Klicken Sie auf **Hinzufügen** und danach auf **Speichern**.
   {: tip}
1. Klicken Sie zum Testen auf **Eingabe ändern** und geben Sie den folgenden JSON-Code ein:
    ```json
    {
      "name": "John Smith",
      "email": "john@smith.com",
      "comment": "Dies ist mein Kommentar"
    }
    ```
    {: codeblock}
1. Aktivieren Sie die Option **Anwenden** und anschließend die Option **Aufrufen**.
    ![](images/solution8/Save_Entry_Invoke.png)

### Folge von Aktionen zum Abrufen von Einträgen

Die zweite Folge von Aktionen wird zum Abrufen der vorhandenen Gästebucheinträge verwendet. Durch diese Folge wird Folgendes ausgeführt:
   * Alle Dokumente aus der Datenbank auflisten
   * Die Dokumente formatieren und zurückgeben

1. Klicken Sie unter **Funktionen** auf **Aktionen** und danach auf **Erstellen**, um eine neue Node.js-Aktion mit dem Namen `leseeingabe-festlegen` zu erstellen.
2. Ersetzen Sie den vorhandenen Code durch das folgende Code-Snippet. Diese Aktion übergibt die entsprechenden Parameter an die nächste Aktion.
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. Aktivieren Sie die Option **Speichern**.

Fügen Sie die Aktion zu einer Folge hinzu:

1. Klicken Sie auf **Einschließende Folgen**, **Zu Folge hinzufügen** und **Neu erstellen**.
1. Geben Sie `folge_zum_lesen_von_gästebucheinträgen` für **Aktionsname** ein und klicken Sie auf **Erstellen und hinzufügen**.

Vervollständigen Sie die Folge:

1. Klicken Sie auf die Folge mit dem Namen **folge_zum_lesen_von_gästebucheinträgen** und klicken Sie anschließend auf **Hinzufügen**, um eine zweite Aktion zum Abrufen von Dokumenten aus Cloudant zu erstellen und hinzuzufügen.
1. Wählen Sie unter **Öffentlich verwenden** die Option **{{site.data.keyword.cloudant_short_notm}}** und danach **dokumente-auflisten** aus.
1. Wählen Sie unter **Meine Bindungen** den Eintrag **bindung-für_gästebuch** und die Option **Hinzufügen** aus, um diese öffentliche Aktion zu Ihrer Folge hinzuzufügen.
1. Klicken Sie erneut auf **Hinzufügen**, um eine dritte Aktion zu erstellen und hinzuzufügen, die die Dokumente aus {{site.data.keyword.cloudant_short_notm}} formatiert.
1. Geben Sie unter **Neu erstellen** den Namen `einträge-formatieren` ein und klicken Sie anschließend auf **Erstellen und hinzufügen**.
1. Klicken Sie auf **einträge-formatieren** und ersetzen Sie den Code durch den folgenden Code:
   ```js
   const md5 = require('spark-md5');

   function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. Aktivieren Sie die Option **Speichern**.
1. Wählen Sie die Folge aus, indem Sie auf **Aktionen** und danach auf **folge-zum-lesen-von-gästebucheinträgen** klicken.
1. Klicken Sie auf **Speichern** und danach auf **Aufrufen**. Die Ausgabe müsste so aussehen:
   ![](images/solution8/Read_Entries_Invoke.png)

## API erstellen

![](images/solution8/Cloud_Functions_API.png)

1. Rufen Sie die Aktionen unter der Webadresse https://{Domänenname}/openwhisk/actions auf.
2. Wählen Sie die Folge mit dem Namen **folge-zum-lesen-von-gästebucheinträgen** aus. Klicken Sie neben dem Namen auf **Webaktion**, wählen Sie die Option **Webaktion aktivieren** und danach **Speichern** aus.
3. Wiederholen Sie diesen Vorgang für die Folge **folge-zum-speichern-eines-gästebucheintrags**.
4. Rufen Sie die APIs unter der Webadresse https://{Domänenname}/openwhisk/apimanagement auf und **Erstellen Sie eine {{site.data.keyword.openwhisk_short}}-API**.
5. Geben Sie den Namen `Gästebuch` an und den Basispfad `/guestbook`.
6. Klicken Sie auf **Operation erstellen** und erstellen Sie eine Operation zum Abrufen von Gästebucheinträgen:
   1. Geben Sie für **Pfad** die Zeichenfolge `/entries` an.
   2. Geben Sie für **Verb** die Zeichenfolge `GET*` an.
   3. Wählen Sie die Aktion **folge-zum-lesen-von-gästebucheinträgen** aus.
7. Klicken Sie auf **Operation erstellen** und erstellen Sie eine Operation zum Speichern eines Gästebucheintrags:
   1. Geben Sie für **Pfad** die Zeichenfolge `/entries` an.
   2. Geben Sie für **Verb** die Zeichenfolge `PUT` an.
   3. Wählen Sie die Aktion **folge-zum-speichern-eines-gästebucheintrags** aus.
8. Speichern Sie die API und stellen Sie sie bereit.

## Web-App bereitstellen

1. Erstellen Sie eine Verzweigung von Ihrem Repository der Gästebuchbenutzerschnittstelle (https://github.com/IBM-Cloud/serverless-guestbook) zu Ihrem öffentlichen GitHub.
2. Bearbeiten Sie **docs/guestbook.js** und ersetzen Sie den Wert **apiUrl** durch die vom API-Gateway angegebene Route.
3. Schreiben Sie die geänderte Datei in Ihrer Repository-Verzweigung fest.
4. Blättern Sie auf der Seite 'Einstellungen' für Ihr Repository zu **GitHub Pages**, ändern Sie die Quelle in den **Ordner /docs im Masterzweig** und Speichern Sie die Änderung.
5. Rufen Sie die öffentlichen Seiten für Ihr Repository auf.
6. Der Gästebucheintrag 'test', den Sie zuvor erstellt haben, müsste angezeigt werden.
7. Fügen Sie neue Einträge hinzu.

![](images/solution8/Guestbook.png)

## Optional: Eigene Domäne für die API verwenden

Beim Erstellen einer verwalteten API wird ein Standardendpunkt wie `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app` bereitgestellt. Im vorliegenden Abschnitt konfigurieren Sie diesen Endpunkt, damit Anforderungen verarbeitet werden können, die von Ihrer angepassten Unterdomäne stammen.

### Zertifikat für angepasste Domäne anfordern

Für das Bereitstellen von {{site.data.keyword.openwhisk_short}}-Aktionen in einer angepassten Domäne ist eine sichere HTTPS-Verbindung erforderlich. Sie sollten ein SSL-Zertifikat für die Domäne und Unterdomäne abrufen, die Sie mit dem serverunabhängigen Back-End verwenden möchten. Wenn eine Domäne wie *mydomain.com* verwendet wird, können die Aktionen unter *guestbook-api.mydomain.com* gehostet werden. In diesem Fall muss das Zertifikat für *guestbook-api.mydomain.com* (oder **.mydomain.com*) ausgegeben werden.

Kostenlose SSL-Zertifikate können Sie bei [Let's Encrypt](https://letsencrypt.org/) anfordern. Während dieses Vorgangs müssen Sie gegebenenfalls einen DNS-Datensatz im TXT-Format in der DNS-Schnittstelle konfigurieren, um sich als Eigner der Domäne auszuweisen.
{:tip}

Nachdem Sie das SSL-Zertifikat und den privaten Schlüssel für Ihre Domäne erhalten haben, müssen Sie diese in das [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail)-Format umwandeln.

1. So wandeln Sie ein Zertifikat in das PEM-Format um:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. So wandeln Sie einen privaten Schlüssel in das PEM-Format um:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### Zertifikat in zentrales Repository importieren

1. Erstellen Sie eine [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)-Instanz an einem unterstützten Standort.
1. Verwenden Sie im Service-Dashboard die Funktion **Zertifikat importieren**:
   * Geben Sie für **Name** die angepasste Unterdomäne und Domäne an (z. B. `guestbook-api.mydomain.com`).
   * Suchen Sie nach der **Zertifikatsdatei** im PEM-Format.
   * Suchen Sie nach der **Datei mit dem privaten Schlüssel** im PEM-Format.
   * Klicken Sie auf **Importieren**.

### Angepasste Domäne für verwaltete API konfigurieren

1. Rufen Sie [APIs / Angepasste Domänen](https://{DomainName}/apis/domains) auf.
1. Wählen Sie im Selektor 'Region' den Standort aus, an dem Sie die Aktionen bereitgestellt haben.
1. Suchen Sie die angepasste Domäne, die mit der Organisation und dem Bereich verknüpft ist, in der bzw. dem Sie die Aktionen und die verwaltete API erstellt haben. Klicken Sie im Aktionsmenü auf **Einstellungen ändern**.
1. Notieren Sie den Wert für **Standarddomäne / Alias**.
1. Wählen Sie die Option **Angepasste Domäne anwenden** aus.
   1. Geben Sie für **Domänenname** die Domäne an, die Sie verwenden möchten (z. B. `guestbook-api.mydomain.com`).
   1. Wählen Sie die {{site.data.keyword.cloudcerts_short}}-Instanz aus, die das Zertifikat enthält.
   1. Wählen Sie das Zertifikat für die Domäne aus.
1. Wechseln Sie zu Ihrem DNS-Provider und erstellen Sie einen neuen **DNS-TXT-Datensatz**, der Ihre Domäne der API-Standarddomäne oder dem zugehörigen Alias zuordnet. Der DNS-TXT-Datensatz kann entfernt werden, nachdem die Einstellungen angewendet wurden.
1. Speichern Sie die Einstellungen für die angepasste Domäne. Das Dialogmodul überprüft, ob der DNS-TXT-Datensatz vorhanden ist.
1. Rufen Sie abschließend wieder die Einstellungen für Ihren DNS-Provider auf und erstellen Sie einen CNAME-Datensatz, der Ihre angepasste Domäne (z. B. guestbook-api.mydomain.com) auf die Standarddomäne bzw. den Alias der Standarddomäne verweist. Dadurch wird Datenverkehr über Ihre angepasste Domäne zu Ihrer Back-End-API geleitet.

Sobald die DNS-Änderungen weitergegeben wurden, können Sie Ihre Gästebuch-API über die Webadresse https://guestbook-api.mydomain.com/guestbook aufrufen.

1. Bearbeiten Sie die Datei **docs/guestbook.js** und ändern Sie den Wert für **apiUrl** in https://guestbook-api.mydomain.com/guestbook.
1. Schreiben Sie die geänderte Datei fest.
1. Ihre Anwendung greift jetzt über Ihre angepasste Domäne auf die API zu.

## Ressourcen entfernen

* Löschen Sie den {{site.data.keyword.cloudant_short_notm}}-Service.
* Löschen Sie die API in {{site.data.keyword.openwhisk_short}}.
* Löschen Sie Aktionen in {{site.data.keyword.openwhisk_short}}.

## Zugehörige Inhalte
* [Weitere Anleitungen und Beispiele für serverunabhängige Komponenten](https://developer.ibm.com/code/journey/category/serverless/)
* [Einführung in {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [{{site.data.keyword.openwhisk}} - häufige Anwendungsfälle](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [REST-APIs aus Aktionen erstellen](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
