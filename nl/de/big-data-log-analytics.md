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

# Big Data-Protokolle mit Streaming Analytics und SQL
{: #big-data-log-analytics}

In diesem Lernprogramm erstellen eine Pipeline für die Protokollanalyse zum Erfassen, Speichern und Analysieren von Protokolleinträgen zur Unterstützung gesetzlicher Bestimmungen oder Erfassung von Informationen. Diese Lösung nutzt mehrere Services, die in {{site.data.keyword.cloud_notm}} {{site.data.keyword.messagehub}}, {{site.data.keyword.cos_short}}, SQL Query und {{site.data.keyword.streaminganalyticsshort}} verfügbar sind. Ein Programm unterstützt Sie durch die Simulation einer Übertragung von Web-Server-Protokollnachrichten aus einer statischen Datei in {{site.data.keyword.messagehub}}.

Mit {{site.data.keyword.messagehub}} wird die Pipeline so skaliert, dass sie Millionen von Protokolleinträge von einer Vielzahl von Producern empfängt. Durch die Anwendung von {{site.data.keyword.streaminganalyticsshort}} können Protokolldaten in Echtzeit überprüft werden, um Geschäftsprozesse zu integrieren. Protokollnachrichten können auch ohne großen Aufwand mithilfe von {{site.data.keyword.cos_short}} in einen Langzeitspeicher umgeleitet werden und somit können Entwickler, Supportmitarbeiter und Prüfer direkt über SQL Query mit den Daten arbeiten.

Dieses Lernprogramm konzentriert sich zwar auf die Protokollanalyse, kann aber auch auf andere Szenarios angewendet werden: speicherbegrenzte IoT-Geräte können in ähnlicher Weise Nachrichten an {{site.data.keyword.cos_short}} streamen oder Marketing-Experten können Kundenereignisse segmentieren und über digitale Einrichtungen hinweg mit SQL Query analysieren.
{:shortdesc}

## Lernziele

{: #objectives}

* Publish/Subscribe-Messaging von Apache Kafka verstehen
* Protokolldaten für Prüf- und Konformitätsanforderungen speichern
* Protokolle überwachen, um Ausnahmebehandlungsvorgänge zu erstellen
* Forensische und statistische Analyse von Protokolldaten durchführen

## Verwendete Services

{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur

{: #architecture}

<p style="text-align: center;">

  ![Architektur](images/solution31/Architecture.png)
</p>

1. Die Anwendung generiert Protokollereignisse für {{site.data.keyword.messagehub}}.
2. Das Protokollereignis wird von {{site.data.keyword.streaminganalyticsshort}} abgefangen und analysiert.
3. Das Protokollereignis wird an eine CSV-Datei in {{site.data.keyword.cos_short}} angehängt.
4. Ein Prüfer oder Mitarbeiter des Support-Teams gibt eine SQL-Job aus.
5. SQL Query wird für die Protokolldatei in {{site.data.keyword.cos_short}} ausgeführt.
6. Die Ergebnismenge wird in {{site.data.keyword.cos_short}} gespeichert und an den Prüfer und den Mitarbeiter des Support-Teams übergeben.

## Vorbereitende Schritte

{: #prereqs}

* [Installieren Sie Git](https://git-scm.com/).
* [Installieren Sie die {{site.data.keyword.Bluemix_notm}}-CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
* [Installieren Sie Node.js](https://nodejs.org)
* [Laden Sie den Kafka 0.10.2.X-Client herunter](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz).

## Services erstellen

{: #setup}

In diesem Abschnitt erstellen Sie die Services, die für die Analyse der von Ihren Anwendungen generierten Protokollaneignisse erforderlich sind.

In diesem Abschnitt wird die Befehlszeile zum Erstellen von Serviceinstanzen verwendet. Alternativ können Sie diesen Schritt über die Serviceseite im Katalog über die bereitgestellten Links ausführen.
{: tip}

1. Melden Sie sich bei {{site.data.keyword.cloud_notm}} über die Befehlszeile an und geben Sie Ihr Cloud Foundry-Konto als Ziel an. Weitere Informationen finden Sie in der [Einführung zur CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Erstellen Sie eine Lite-Instanz von [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
    ```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. Erstellen Sie eine Lite-Instanz von [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. Erstellen Sie eine Standard-Instanz von [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams).
    ```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## Messaging-Topic und {{site.data.keyword.cos_short}}-Bucket erstellen

{: #topics}

Erstellen Sie zuerst ein {{site.data.keyword.messagehub}}-Topic und ein {{site.data.keyword.cos_short}}-Bucket. Topics definieren, wo Anwendungen Nachrichten in Publish/Subscribe-Messaging-Systemen bereitstellen. Nachdem Nachrichten empfangen und verarbeitet wurden, werden sie in einer Datei gespeichert, die sic in einem {{site.data.keyword.cos_short}}-Bucket befindet.

1. Greifen Sie in Ihrem Browser auf die Serviceinstanz `log-analysis-hub` über die [Ressourcen](https://{DomainName}/resources?search=log-analysis) zu.
2. Klicken Sie auf die Schaltfläche mit dem **+**, um ein Topic zu erstellen.
3. Geben Sie als **Topicnamen** den Namen `webserver` ein und klicken Sie auf die Schaltfläche **Topic erstellen**.
4. Klicken Sie auf **Serviceberechtigungsnachweise** und auf die Schaltfläche **Neuer Berechtigungsnachweis**.
5. Geben Sie im daraufhin angezeigten Dialogfeld `webserver-flow` als **Name** ein und klicken Sie auf die Schaltfläche **Hinzufügen**.
6. Klicken Sie auf **Berechtigungsnachweise anzeigen** und kopieren Sie die Informationen an einen sicheren Ort. Sie werden im nächsten Abschnitt verwendet.
7. Wählen Sie in der [Ressourcenliste](https://{DomainName}/resources?search=log-analysis) die Serviceinstanz `log-analysis-cos` aus.
8. Klicken Sie auf **Bucket erstellen**.
    * Geben Sie einen eindeutigen **Namen** für das Bucket ein.
    * Wählen Sie **Regionsübergreifend** für **Ausfallsicherheit** aus.
    * Wählen Sie **us-geo** als **Standort** aus.
    * Klicken Sie auf **Bucket erstellen**.

## Streams-Datenflussquelle erstellen

{: #streamsflow}

In diesem Abschnitt beginnen Sie mit der Konfiguration eines Streams-Datenflusses, der Protokollnachrichten empfängt. Der {{site.data.keyword.streaminganalyticsshort}}-Service basiert auf {{site.data.keyword.streamsshort}} und kann Millionen von Ereignissen pro Sekunde analysieren, wodurch Antwortzeiten im Sub-Millisekunden-Bereich und sofortige Entscheidungen möglich sind.

1. Greifen Sie in Ihrem Browser auf [Watson Data Platform](https://dataplatform.ibm.com) zu.
2. Wählen Sie die Schaltfläche oder Kachel **Neues Projekt** und anschließend die Kachel **Basis** aus und klicken Sie auf **OK**.
    * Geben Sie als **Namen** den Namen `webserver-logs` ein.
    * Die Option **Speicher** sollte auf `log-analysis-cos` gesetzt sein. Ist dies nicht der Fall, wählen Sie die Serviceinstanz aus.
    * Klicken Sie auf die Schaltfläche **Erstellen**.
3. Wählen Sie auf der angezeigten Seite die Registerkarte **Einstellungen** aus und wählen Sie **Streams Designer** in **Tools** aus. Um den Vorgang abzuschließen, klicken Sie auf die Schaltfläche **Speichern**.
4. Klicken Sie auf die Schaltfläche **Zu Projekt hinzufügen** und dann in der oberen Navigationsleiste auf **Streams-Datenfluss**.
    * Klicken Sie auf **IBM Streaming Analytics-Instanz einem containerbasierten Plan zuordnen**.
    * Erstellen Sie eine neue {{site.data.keyword.streaminganalyticsshort}}-Instanz, indem Sie auf das Optionsfeld **Lite** und anschließend auf **Erstellen** klicken. Wählen Sie nicht 'Lite VM' aus.
    * Geben Sie den **Servicenamen** als `log-analysis-sa` an und klicken Sie auf **Bestätigen**.
    * Geben Sie als **Name** für den Streams-Datenfluss `webserver-flow` ein.
    * Klicken Sie abschließend auf **Erstellen**.
5. Wählen Sie auf der angezeigten Seite die Kachel **{{site.data.keyword.messagehub}}** aus.
    * Kicken Sie auf **Verbindung hinzufügen** und wählen Sie Ihre {{site.data.keyword.messagehub}}-Instanz `log-analysis-hub` aus. Wenn die gewünschte Instanz nicht angezeigt wird, wählen Sie die Option **IBM {{site.data.keyword.messagehub}}** aus. Geben Sie manuell die **Verbindungsdetails** ein, die Sie aus den **Serviceberechtigungsnachweisen** im vorherigen Abschnitt erhalten haben. Geben Sie als **Name** für die Verbindung `webserver-flow` ein.
    * Klicken Sie auf **Erstellen**, um die Verbindung zu erstellen.
    * Wählen Sie `webserver` aus der Dropdown-Liste **Topic** aus.
    * Wählen Sie **Mit der ersten neuen Nachricht beginnen** in der Dropdown-Liste **Ursprüngliches Offset** aus.
    * Klicken Sie auf **Weiter**.
6. Lassen Sie die Seite **Vorschau der Daten** geöffnet, da sie im nächsten Abschnitt verwendet wird.

## Tools der Kafka-Konsole mit {{site.data.keyword.messagehub}} verwenden

{: #kafkatools}

Der Streams-Datenfluss `webserver-flow` ist momentan inaktiv und wartet auf Nachrichten. In diesem Abschnitt konfigurieren Sie Tools der Kafka-Konsole für die Verwendung mit {{site.data.keyword.messagehub}}. Mit diesen Tools können Sie beliebige Nachrichten über den Terminal erzeugen und diese an {{site.data.keyword.messagehub}} senden, wodurch der Streams-Datenfluss `webserver-flow` ausgelöst wird.

1. Laden Sie den [Kafka 0.10.2.X-Client](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz) herunter und dekomprimieren Sie die Datei.
2. Wechseln Sie in das Verzeichnis `bin` und erstellen Sie eine Textdatei mit dem Namen `message-hub.config` mit den folgenden Inhalten.
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. Ersetzen Sie `USER` und `PASSWORD` in der Datei `message-hub.config` durch die Werte für `user` und `password` aus den **Serviceberechtigungsnachweisen** aus dem vorherigen Abschnitt. Speichern Sie die Datei `message-hub.config`.
4. Führen Sie im Verzeichnis `bin` den folgenden Befehl aus. Ersetzen Sie `KAFKA_BROKERS_SASL` durch den Wert für `kafka_brokers_sasl`, der in den **Serviceberechtigungsnachweisen** angezeigt wird. Siehe dazu folgendes Beispiel.
    ```sh
    ./kafka-console-producer.sh --broker-list KAFKA_BROKERS_SASL \
    --producer.config message-hub.config --topic webserver
    ```
    {: pre}
    ```sh
    ./kafka-console-producer.sh --broker-list \
    kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093 \
    --producer.config message-hub.config --topic webserver
    ```
5. Das Kafka-Konsolentool wartet auf eine Eingabe. Kopieren Sie die nachfolgenden Protokollnachricht und fügen Sie sie in den Terminal ein. Drücken Sie die `Eingabetaste`, um die Protokollnachricht an {{site.data.keyword.messagehub}} zu senden. Beachten Sie, dass die gesendeten Nachrichten auch auf der Seite **Vorschau der Daten** von `webserver-flow` angezeigt werden.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![Vorschauseite](images/solution31/preview_data.png)

## Streams-Datenflussziel erstellen

{: #streamstarget}

In diesem Abschnitt schließen Sie die Konfiguration des Streams-Datenflusses durch das Definieren eines Ziels ab. Das Ziel wird verwendet, um eingehende Protokollnachrichten in dem zuvor erstellten {{site.data.keyword.cos_short}}-Bucket zu speichern. Das Speichern und Anhängen eingehender Protokollnachrichten an eine Datei wird dabei automatisch von {{site.data.keyword.streaminganalyticsshort}} vorgenommen.

1. Klicken Sie auf der Seite **Vorschau der Daten** von `webserver-flow` auf die Schaltfläche **Weiter**.
2. Wählen Sie die Kachel **{{site.data.keyword.cos_full_notm}}** als Ziel aus.
    * Klicken Sie auf **Verbindung hinzufügen** und wählen Sie `log-analysis-cos` aus.
    * Klicken Sie auf **Erstellen**.
    * Geben Sie den folgenden **Dateipfad** an: `/NAME_DES_BUCKETS/http-logs_%TIME.csv`. Ersetzen Sie dabei `NAME_DES_BUCKETS` durch den im ersten Abschnitt verwendeten Wert.
    * Wählen Sie **csv** aus der Dropdown-Liste **Format** aus.
    * Aktivieren Sie das Kontrollkästchen **Spaltenkopfzeile**.
    * Wählen Sie **Dateigröße** in der Dropdown-Liste **Dateierstellungsrichtlinie** aus.
    * Geben Sie `102400` in das Textfeld **Dateigröße (KB)** ein, um den Grenzwert auf 100 MB zu setzen.
    * Klicken Sie auf **Weiter**.
3. Klicken Sie auf **Speichern**.
4. Klicken Sie auf die Wiedergabeschaltfläche **>**, um den **Streams-Datenfluss zu starten**.
5. Nachdem der Nachrichtenfluss gestartet wurde, senden Sie erneut mehrere Protokollnachrichten aus dem Kafka-Konsolentool. Sie können beobachten, wie Nachrichten ankommen, indem Sie `webserver-flow` in Streams Designer anzeigen.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. Kehren Sie zu Ihrem Bucket in {{site.data.keyword.cos_short}} zurück. Sobald genug Nachrichten fließen oder der Datenfluss erneut gestartet wurde, wird eine Datei `log.csv` angezeigt.

![webserver-flow](images/solution31/flow.png)

## Bedingtes Verhalten zu Streams-Datenflüssen hinzufügen

{: #streamslogic}

Bis jetzt ist der Streams-Datenfluss eine einfache Pipe, über die Nachrichten von {{site.data.keyword.messagehub}} nach {{site.data.keyword.cos_short}} verschoben werden. Es ist wahrscheinlich, dass Teams Ereignisse, die von Interesse sind, in Echtzeit erkennen möchten. Beispielsweise können für einige Teams Alerts beim Auftreten von Ereignissen mit dem Statuscode HTTP 500 (Anwendungsfehler) nützlich sein. In diesem Abschnitt fügen Sie dem Datenfluss eine bedingte Logik hinzu, um Ereignisse mit dem Statuscode HTTP 200 (OK) und ohne diesen Statuscode zu identifizieren.

1. Verwenden Sie die Stiftsymbol, um den **Streams-Datenfluss zu bearbeiten**.
2. Erstellen Sie einen Filterknoten, der HTTP 200-Antworten verarbeitet.
    * Ziehen Sie in der Palette **Knoten** den Knoten **Filter** aus **VERARBEITUNG UND ANALYSE** in den Erstellungsbereich.
    * Geben Sie `OK` in das Textfeld 'Name' ein, das derzeit das Wort `Filter` enthält.
    * Geben Sie die folgende Anweisung in den Textbereich **Bedingungsausdruck** ein.
      ```sh
      responseCode == 200
      ```
      {: pre}
    * Zeichnen Sie mit der Maus eine Linie von der Ausgabe des **{{site.data.keyword.messagehub}}**-Knotens (rechte Seite) zur Eingabe des **OK**-Knotens (linke Seite).
    * Ziehen Sie in der Palette **Knoten** den Knoten **Debug** aus **ZIELE** in den Erstellungsbereich.
    * Verbinden Sie den Knoten **Debug** mit dem Knoten **OK**, indem Sie eine Linie zwischen den beiden Knoten ziehen.
3. Wiederholen Sie den Prozess, um einen Filter `Not OK` unter Verwendung derselben Knoten und der folgenden Bedingungsanweisung zu erstellen.
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. Klicken Sie auf die Wiedergabeschaltfläche, um den **Streams-Datenfluss zu speichern und auszuführen**.
5. Wenn Sie dazu aufgefordert werden, klicken Sie auf den Link zum **Ausführen der neuen Version**.

![Datenfluss-Designer](images/solution31/flow_design.png)

## Nachrichtenvolumen erhöhen

{: #streamsload}

Erhöhen Sie für eine bedingte Verarbeitung in Ihrem Streams-Datenfluss das an {{site.data.keyword.messagehub}} gesendete Nachrichtenvolumen. Das bereitgestellte Node.js-Programm simuliert einen realistischen Nachrichtenfluss am {{site.data.keyword.messagehub}} auf der Basis des Datenverkehrs zum Webserver. Um die Skalierbarkeit von {{site.data.keyword.messagehub}} und {{site.data.keyword.streaminganalyticsshort}} zu demonstrieren, erhöhen Sie den Durchsatz von Protokollnachrichten.

In diesem Abschnitt wird [node-rdkafka](https://www.npmjs.com/package/node-rdkafka) verwendet. Wenn die Installation des Simulators fehlschlägt, finden Sie Anweisungen zur Fehlerbehebung auf der npmjs-Seite. Wenn Probleme bestehen bleiben, können Sie mit dem nächsten Abschnitt fortfahren und die Daten manuell hochladen.

1. Laden Sie die [erforderliche Protokolldatei (1. bis 31. Juli,  ASCII-Format, 20.7 MB, gzip-komprimiert)](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz) von NASA herunter.
2. Klonen Sie den Protokollsimulator von [IBM-Cloud auf GitHub](https://github.com/IBM-Cloud/kafka-log-simulator) und installieren Sie ihn.
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. Wechseln Sie in das Simulatorverzeichnis und führen Sie die folgenden Befehle aus, um den Simulator zu konfigurieren und Protokollereignisnachrichten zu erstellen. Ersetzen Sie `LOGFILE` durch die Datei, die Sie heruntergeladen haben. Ersetzen Sie `BROKERLIST` und `APIKEY` durch die entsprechenden, zuvor verwendeten **Serviceberechtigungsnachweise**. Siehe dazu folgendes Beispiel.
    ```sh
    npm install
    ```
    ```sh
    npm run build
    ```
    ```sh
    node dist/index.js --file LOGFILE --parser httpd --broker-list BROKERLIST \
    --api-key APIKEY --topic webserver --rate 100
    ```
    {: pre}
    ```sh
    node dist/index.js --file /Users/ibmcloud/Downloads/NASA_access_log_Jul95 \
    --parser httpd --broker-list \
    "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093" \
    --api-key Np15YZKN3SCdABUsOpJYtpue6jgJ7CwYgsoCWaPbuyFbdM4R \
    --topic webserver --rate 100
    ```
4. Wenn der Simulator mit dem Erstellen von Nachrichten beginnt, kehren Sie in Ihrem Browser zu Ihrem Streams-Datenfluss `webserver-flow` zurück.
5. Wenn die gewünschte Anzahl von Nachrichten durch die bedingten Verzweigungen geflossen sind, stoppen Sie den Simulator mit der Tastenkombination `Strg+C`.
6. Testen Sie die Skalierungsmöglichkeiten in {{site.data.keyword.messagehub}}, indem Sie den Wert für `--rate` erhöhen oder verringern.

Der Simulator verzögert das Senden der nächsten Nachricht basierend auf der abgelaufenen Zeit im Web-Serverprotokoll. Wenn Sie `-- rate 1` festlegen, werden Ereignisse in Echtzeit gesendet. Die Einstellung `-- rate 100` bedeutet, dass für jede einzelne Sekunde abgelaufene Zeit im Web-Serverprotokoll eine Verzögerung von 10 ms zwischen den Nachrichten verwendet wird.
{: tip}

![Durchflusslast auf 10 gesetzt](images/solution31/flow_load_10.png)

## Protokolldaten mithilfe von SQL Query untersuchen

{: #sqlquery}

Abhängig von der Anzahl der Nachrichten, die vom Simulator gesendet werden, hat sich die Protokolldatei für {{site.data.keyword.cos_short}} auf jeden Fall vergrößert. Sie werden nun als Prüfer für die Beantwortung von Prüfungs -oder Compliance-Fragen agieren, indem Sie SQL Query mit Ihrer Protokolldatei kombinieren. Der Vorteil bei der Verwendung von SQL Query besteht darin, dass die Protokolldatei direkt zugänglich ist. Es sind keine zusätzlichen Transformationen oder Datenbankserver erforderlich.

Wenn Sie nicht darauf warten möchten, dass der Simulator alle Protokollnachrichten sendet, laden Sie die [vollständige CSV-Datei](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp) in {{site.data.keyword.cos_short}} hoch, um sofort mit Ihrer neuen Aufgabe zu beginnen.
{: tip}

1. Greifen Sie auf die `log-analysis-sql`-Serviceinstanz über die [Ressourcenliste](https://{DomainName}/resources?search=log-analysis) zu. Wählen Sie **Benutzerschnittstelle öffnen**, um SQL Query zu starten.
2. Geben Sie die folgende SQL-Anweisung in den Textbereich **SQL hier eingeben ...** ein.
    ```sql
    -- What are the top 10 web pages on NASA from July 1995?
    -- Which mission might be significant?
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. Rufen Sie die Objekt-SQL-URL aus der Protokolldatei ab.
    * Wählen Sie in der [Ressourcenliste](https://{DomainName}/resources?search=log-analysis) die Serviceinstanz `log-analysis-cos` aus.
    * Wählen Sie das zuvor erstellte Bucket aus.
    * Klicken Sie auf das Überlaufmenü in der Datei `http-logs_TIME.csv` und wählen Sie **Objekt-SQL-URL** aus.
    * **Kopieren** Sie die URL in die Zwischenablage.
4. Aktualisieren Sie die `FROM`-Klausel mit Ihrer SQL-Objekt-URL und klicken Sie auf **Ausführen**.
5. Das Ergebnis wird auf der Registerkarte **Ergebnis** angezeigt. Während einige Seiten - wie die Homepage des Kennedy Space Center - erwartet werden, ist eine Mission recht beliebt zu dieser Zeit.
6. Wählen Sie die Registerkarte **Abfragedetails** aus, um weitere Informationen anzuzeigen, z. B. die Position, an der das Ergebnis in {{site.data.keyword.cos_short}} gespeichert wurde.
7. Testen Sie die folgenden Frage/Antwort-Paare aus, indem Sie sie einzeln zum Textbereich **SQL hier eingeben ...** hinzufügen.
    ```sql
    -- Who are the top 5 viewers?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- Which viewer has suspicious activity based on application failures?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- Which requests showed a page not found error to the user?
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- What are the top 10 largest files?
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- What is the distribution of total traffic by hour?
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- Why did the previous result return an empty hour?
    -- Hint, find the malformed hostname.
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

FROM-Klauseln sind nicht auf eine einzige Datei beschränkt. Verwenden Sie `cos://us-geo/NAME_DES_BUCKETS/`, um SQL-Abfragen für alle Dateien in dem Bucket auszuführen.
{: tip}

## Lernprogramm erweitern

{: #expand}

Herzlichen Glückwunsch. Sie haben eine Protokollanalyse-Pipeline mit {{site.data.keyword.cloud_notm}} erstellt. Nachstehend finden Sie weitere Vorschläge, um Ihre Lösung zu verbessern.

* Verwenden Sie zusätzliche Ziele in Streams Designer zum Speichern von Daten in [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) oder führen Sie Code in [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) aus.
* Führen Sie das Lernprogramm zum [Erstellen eines Data Lakes mithilfe von Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage) aus, um ein Dashboard zum Protokollieren von Daten hinzuzufügen.
* Integrieren Sie mithilfe von [{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect) einige zusätzliche Systeme mit {{site.data.keyword.messagehub}}.

## Services entfernen

{: #removal}

Verwenden Sie in der [Ressourcenliste](https://{DomainName}/resources?search=log-analysis) das Menüelement **Löschen** oder **Service löschen** im Überlaufmenü, um die folgenden Serviceinstanzen zu entfernen.

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## Zugehöriger Inhalt

{:related}

* [Apache Kafka](https://kafka.apache.org/)
