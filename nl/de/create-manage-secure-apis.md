---
copyright:
  years: 2017, 2019
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

# REST-APIs erstellen, sichern und verwalten
{: #create-manage-secure-apis}

In diesem Lernprogramm wird gezeigt, wie REST-APIs mit dem Node.js-API-Framework von LoopBack erstellt werden. Mit LoopBack können Sie schnell REST-APIs erstellen, die Geräte und Browser mit Daten und Services verbinden. Außerdem können Ihrer Anwendung mit {{site.data.keyword.apiconnect_long}} Verwaltungs- und Sichtbarkeitsfunktionen und Ratenbegrenzungen hinzufügen.
{:shortdesc}

## Lernziele

* REST-API mit wenig oder ohne Codierung erstellen
* API in {{site.data.keyword.Bluemix_notm}} für Entwickler veröffentlichen
* Vorhandene APIs {{site.data.keyword.apiconnect_short}} importieren
* Zugriff für Systems of Record sicher bereitstellen und steuern

## Verwendete Services

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:

* [LoopBack](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* [SDK for Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs) - Cloud Foundry-App

## Architektur

![Architektur](images/solution13/Architecture.png)

1. Der Entwickler definiert eine RESTful-API.
2. Der Entwickler veröffentlicht die API in {{site.data.keyword.apiconnect_long}}.
3. Benutzer und Anwendungen verwenden die API.

## Vorbereitende Schritte

* Laden Sie [Node.js](https://nodejs.org/en/download/) Version 6.x herunter und installieren Sie das Programm (verwenden Sie [nvm](https://github.com/creationix/nvm) oder Ähnliches, wenn bereits eine aktuellere Version von Node.js installiert ist)

## REST-API in Node.js erstellen

{: #create_api}
In diesem Abschnitt erstellen Sie eine API in Node.js mit [LoopBack](https://loopback.io/doc/index.html). LoopBack ist ein erweiterbares, Open-Source-Node.js-Framework, das es Ihnen ermöglicht, dynamische End-to-End-REST-APIs mit wenig oder ohne Codierung zu erstellen.

### Anwendung erstellen

1. Installieren Sie das {{site.data.keyword.apiconnect_short}}-Befehlszeilentool. Wenn Sie Probleme bei der Installation von {{site.data.keyword.apiconnect_short}} haben, verwenden Sie `sudo` vor dem Befehl oder befolgen Sie die [hier](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit) aufgeführten Anweisungen.
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. Geben Sie den folgenden Befehl ein, um die Anwendung zu erstellen.
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. Drücken Sie die `Eingabetaste`, um **entries-api** als **Namen der Anwendung** zu verwenden.
4. Drücken Sie die `Eingabetaste`, um **entries-api** als das **Verzeichnis zu verwenden, das das Projekt enthalten soll**.
5. Wählen Sie **3.x (aktuell)** als **Version von LoopBack** aus.
6. Wählen Sie **empty-server** als **Art der Anwendung** aus.

![APIC LoopBack-Gerüst](images/solution13/apic_loopback.png)

### Datenquelle hinzufügen

Datenquellen stellen Back-End-Systeme dar, wie z. B. Datenbanken, externe REST-APIs, SOAP-Web-Services und Speicherservices. Datenquellen stellen normalerweise CRUD-Funktionen (Create, Retrieve, Update, Delete) für Erstellungs-, Abruf-, Aktualisierungs- und Löschoperationen bereit. Während LoopBack viele Typen von [Datenquellen](http://loopback.io/doc/en/lb3/Connectors-reference.html) unterstützt, verwenden Sie der Einfachheit halber einen speicherinternen Datenspeicher mit Ihrer API.

1. Wechseln Sie in das Verzeichnis für das neue Projekt und starten Sie den API Designer.
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. Klicken Sie in der Navigationsleiste auf **Datenquellen**. Klicken Sie dann auf die Schaltfläche **Hinzufügen**. Daraufhin wird das Dialogfeld **Neue LoopBack-Datenquelle** geöffnet.
3. Geben Sie `entriesDS` in das Textfeld **Name** ein und klicken Sie auf die Schaltfläche **Neu**.
4. Wählen Sie **Speicherinterne DB** im Kombinationsfeld **Connector** aus.
5. Klicken Sie oben links auf **Alle Datenquellen**. Die Datenquelle wird in der Liste der Datenquellen angezeigt.

   Der Editor aktualisiert die JSON-Datei für Server/Datenquellen automatisch mit den Einstellungen für die neue Datenquelle.
   {:tip}

![Datenquellen im API Designer](images/solution13/datastore.png)

### Modell hinzufügen

Ein Modell ist ein JavaScript-Objekt mit Node- und REST-APIs, das Daten in Back-End-Systemen darstellt. Modelle sind über Datenquellen mit diesen Systemen verbunden. In diesem Abschnitt definieren Sie die Struktur der Daten und verbinden diese mit den zuvor erstellten Datenquellen.

1. Klicken Sie in der Navigationsleiste auf **Modelle**. Klicken Sie dann auf die Schaltfläche **Hinzufügen**. Daraufhin wird das Dialogfeld **Neues LoopBack-Modell** geöffnet.
2. Geben Sie `entry` in das Textfeld **Name** ein und klicken Sie auf die Schaltfläche **Neu**.
3. Wählen Sie **entriesDS** aus dem Kombinationsfeld **Datenquelle** aus.
4. Fügen Sie über die Karte **Eigenschaften** die folgenden Eigenschaften mithilfe des Symbols **Eigenschaft hinzufügen** hinzu.
    1. Geben Sie `name` für den **Eigenschaftsnamen** ein und wählen Sie **Zeichenfolge** aus dem Kombinationsfeld **Typ** aus.
    2. Geben Sie `email` für den **Eigenschaftsnamen** ein und wählen Sie **Zeichenfolge** aus dem Kombinationsfeld **Typ** aus.
    3. Geben Sie `comment` für den **Eigenschaftsnamen** ein und wählen Sie **Zeichenfolge** aus dem Kombinationsfeld **Typ** aus.
5. Klicken Sie rechts oben auf das Symbol **Speichern**, um das Modell zu speichern.

![Modellgenerator](images/solution13/models.png)

## LoopBack-Anwendung testen

In diesem Abschnitt starten Sie eine lokale Instanz Ihrer LoopBack-Anwendung und testen die API, indem Sie Daten mithilfe von API Designer einfügen und abfragen. Sie müssen das Erkundungstool (Explore) von API Designer verwenden, um REST-Endpunkte in Ihrem Browser zu testen, da es die richtigen Sicherheitsheader und andere Anforderungsparameter enthält.

1. Starten Sie den lokalen Server, indem Sie auf das Symbol **Starten** in der linken unteren Ecke klicken und auf die Nachricht **Aktiv** warten.
2. Klicken Sie im Banner auf den Link **Erkunden**, um das Erkundungstool von API Designer anzuzeigen. In der Seitenleiste werden die REST-Operationen angezeigt, die für die LoopBack-Modelle in der API verfügbar sind.
3. Klicken Sie im linken Teilfenster auf die Operation **entry.create**, um den Endpunkt anzuzeigen. Im mittleren Teilfenster werden Übersichtsdaten zu dem Endpunkt angezeigt, einschließlich Parameter, Sicherheitsinformationen, Modellinstanzdaten und Antwortcodes. Das rechte Teilfenster enthält Mustercode zum Aufrufen des Endpunkts unter Verwendung des cURL-Befehls und Sprachen wie Ruby, Python, Java und Node.
4. Klicken Sie im rechten Teilfenster auf den Link zum **Ausprobieren**. Blättern Sie abwärts zu **Parameter** und geben Sie Folgendes in den Textbereich **Daten** ein:
    ```javascript
    {
      "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. Klicken Sie auf die Schaltfläche **Aufrufoperation**.
  ![API über API Designer testen](images/solution13/data_entry_1.png)
6. Stellen sie sicher, dass die POST-Operation erfolgreich war, indem Sie nach **Response Code: 200 OK** suchen. Wenn eine Fehlernachricht aufgrund eines nicht vertrauenswürdigen Zertifikats für 'localhost' angezeigt wird, klicken Sie auf den Link, der in der Fehlernachricht im Erkundungstool von API Designer angegeben ist, um das Zertifikat zu akzeptieren, und rufen Sie anschließend die Operationen in Ihrem Web-Browser auf. Die genaue Vorgehensweise hängt von dem verwendeten Web-Browser ab.
7. Fügen Sie einen weiteren Eintrag mit cURL hinzu. Stellen Sie sicher, dass der Port mit dem Port Ihrer Anwendung übereinstimmt.
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. Klicken Sie auf die Operation **entry.find**, dann auf den Link zum **Ausprobieren** und anschließend auf die Schaltfläche **Aufrufoperation**. Die Antwort zeigt alle Einträge an. Es sollte JSON für **Jane Doe** und **John Doe** angezeigt werden.
  ![entry_find](images/solution13/find_response.png)

Sie können die LoopBack-Anwendung auch manuell starten, indem Sie den Befehl `npm start` aus dem Verzeichnis `entries-api` absetzen. Ihre REST-APIs stehen unter [http://localhost:3000/api/entries](http://localhost:3000/api/entries) zur Verfügung; sie werden jedoch nicht von {{site.data.keyword.apiconnect_short}} verwaltet und geschützt.
{:tip}

## {{site.data.keyword.apiconnect_short}}-Service erstellen

Erstellen Sie zur Vorbereitung der nächsten Schritte einen **{{site.data.keyword.apiconnect_short}}**-Service unter {{site.data.keyword.Bluemix_notm}}. {{site.data.keyword.apiconnect_short}} fungiert als Gateway zu Ihrer API und bietet auch Verwaltungs- und Sicherheitsfunktionen sowie und Ratenbegrenzungen.

1. Starten Sie die {{site.data.keyword.Bluemix_notm}}-[Ressourcenliste](https://{DomainName}/resources).
2. Navigieren Sie zu **Katalog > Integration > {{site.data.keyword.apiconnect_short}}** und klicken Sie auf die Schaltfläche **Erstellen**.

## API in {{site.data.keyword.Bluemix_notm}} veröffentlichen

{: #publish}
Sie verwenden API Designer, um Ihre Anwendung in {{site.data.keyword.Bluemix_notm}} als Cloud Foundry-Anwendung zu implementieren und außerdem die API-Definition in **{{site.data.keyword.apiconnect_short}}** zu veröffentlichen. API Designer ist Ihr lokales Toolkit. Wenn Sie ihn geschlossen haben, starten Sie ihn mit `apic edit` aus dem Projektverzeichnis erneut.

Die Anwendung kann auch manuell mit dem Befehl `ibmcloud cf push` implementiert werden; sie wird jedoch nicht geschützt. Verwenden Sie zum [Importieren der Datei](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing) in {{site.data.keyword.apiconnect_short}} die OpenAPI-Definitionsdatei aus dem Ordner `definitions`. Bei der Implementierung mit API Designer wird die Anwendung geschützt und die Definition automatisch importiert.
{:tip}

1. Klicken Sie in API Designer auf den Link **Veröffentlichen** im Banner. Klicken Sie anschließend auf **Ziele hinzufügen und verwalten > IBM Bluemix-Ziel hinzufügen**.
2. Wählen Sie die **Region** und die **Organisation** für die Veröffentlichung aus.
3. Wählen Sie den **Sandbox**-Katalog aus und klicken Sie auf **Weiter**.
4. Geben Sie `entries-api-application` in das Textfeld **Neuen Anwendungsnamen eingeben** ein und klicken Sie auf das Plussymbol (**+**).
5. Klicken Sie in der Liste auf **entries-api-application** und dann auf **Speichern**.
6. Klicken Sie in der linken oberen Ecke des Banners auf das 'Hamburger'-Symbol **Menü**. Klicken Sie anschließend auf **Projekte** und auf das Listenelement **entries-api**.
7. Klicken Sie in der API Designer-Benutzerschnittstelle auf **APIs > entries-api > Assemblieren**.
8. Klicken Sie im Assembly Editor auf das Symbol **Richtlinien filtern**.
9. Wählen Sie **DataPower-Gatewayrichtlinien** aus und klicken Sie auf **Speichern**.
10. Klicken Sie in der oberen Leiste auf **Veröffentlichen** und wählen Sie das Ziel aus. Wählen Sie **Anwendung veröffentlichen** und 'Produkte bereitstellen oder veröffentlichen' > **Bestimmte Produkte** > **entries-api** aus.
11. Klicken Sie auf **Veröffentlichen** und warten Sie, bis die Anwendung erfolgreich veröffentlicht wurde.
    ![Dialogfeld für die Veröffentlichung](images/solution13/publish.png)

    Eine Anwendung enthält die Loopback-Modelle, -Datenquellen und -Codierung, die sich auf Ihre API beziehen. Ein Produkt ermöglicht Ihnen anzugeben, wie eine API Entwicklern zur Verfügung gestellt wird.
    {:tip}

Die API-Anwendung wird jetzt in {{site.data.keyword.Bluemix_notm}} als Cloud Foundry-Anwendung veröffentlicht. Sie können sie sehen, indem Sie sich die Cloud Foundry-Anwendungen in der {{site.data.keyword.Bluemix_notm}}-[Ressourcenliste](https://{DomainName}/resources) ansehen. Der direkte Zugriff über die URL ist jedoch nicht möglich, da die Anwendung geschützt ist. Im nächsten Abschnitt wird angezeigt, wie auf die verwalteten APIs zugegriffen werden kann.

## API-Gateway

Bis jetzt haben Sie Ihre API lokal entworfen und getestet. In diesem Abschnitt verwenden Sie {{site.data.keyword.apiconnect_short}}, um Ihre implementierte API unter {{site.data.keyword.Bluemix_notm}} zu testen.

1. Starten Sie die {{site.data.keyword.Bluemix_notm}}-[Ressourcenliste](https://{DomainName}/resources).
2. Suchen Sie Ihren **{{site.data.keyword.apiconnect_short}}**-Service unter **Cloud Foundry-Services** und wählen Sie ihn aus.
3. Klicken Sie auf das Menü **Durchsuchen** und klicken Sie dann auf den Link **Sandbox**.
4. Klicken Sie auf die Operation **entry.create**.
5. Klicken Sie im rechten Teilfenster auf den Link zum **Ausprobieren**. Blättern Sie abwärts zu **Parameter** und geben Sie Folgendes in den Textbereich **Daten** ein:
    ```javascript
    {
      "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. Klicken Sie auf die Schaltfläche **Operation aufrufen**. Bei erfolgreicher Durchführung sollte eine Antwort des Typs **Code: 200** angezeigt werden.

![Gateway](images/solution13/gateway.png)

Ihre verwaltete und sichere API-URL wird neben jeder Operation angezeigt und sollte wie folgt aussehen: `https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries`.
{: tip}

## Ratenbegrenzung

Durch das Festlegen von Ratenbegrenzungen können Sie den Netzverkehr für Ihre APIs und für bestimmte Operationen innerhalb der APIs verwalten. Eine Ratenbegrenzung ist die maximale Anzahl von Aufrufen, die in einem bestimmten Zeitintervall zulässig ist.

1. Klicken Sie in API Designer auf **Produkte > entries-api**.
2. Wählen Sie **Standard-Plan** auf der linken Seite aus.
3. Erweitern Sie **Standard-Plan** und blättern Sie abwärts zum Feld **Ratenbegrenzungen**.
4. Setzen Sie die Felder auf **10** Aufrufe / **1** **Minute**.
5. Wählen Sie **Festen Grenzwert durchsetzen** und klicken Sie auf das Symbol **Speichern**.
  ![Seite für die Ratenbegrenzung](images/solution13/rate_limit.png)
6. Befolgen Sie die Schritte im Abschnitt [API in {{site.data.keyword.Bluemix_notm}} veröffentlichen](#publish), um die API erneut zu veröffentlichen.

Ihre API ist jetzt auf 10 Anfragen pro Minute begrenzt. Verwenden Sie die Funktion zum **Ausprobieren**, um den Grenzwert zu erreichen. Rufen Sie weitere Informationen zum [Festlegen von Ratenbegrenzungen](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits) ab oder erkunden Sie API Designer, um sich mit allen verfügbaren Verwaltungsfunktionen vertraut zu machen.

## Lernprogramm erweitern

Herzlichen Glückwunsch. Sie haben eine API erstellt, die sowohl verwaltet als auch sicher ist. Im Folgenden finden Sie weitere Vorschläge zur Erweiterung Ihrer API.

* Datenpersistenz mit dem LoopBack-Connector von [{{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant) hinzufügen
* API Designer zum [Anzeigen zusätzlicher Einstellungen](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0) für die Verwaltung der API verwenden
* API für die **Analyse** und **Visualisierungen** überprüfen, die in {{site.data.keyword.apiconnect_short}} [verfügbar](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics) ist.

## Zugehöriger Inhalt

* [LoopBack-Dokumentation](https://loopback.io/doc/index.html)
* [Einführung in {{site.data.keyword.apiconnect_long}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
