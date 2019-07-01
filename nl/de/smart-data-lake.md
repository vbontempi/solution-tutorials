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

# Data Lake mithilfe von Object Storage erstellen
{: #smart-data-lake}

Die Definitionen für den Begriff 'Data Lake' variieren. Im Kontext dieses Lernprogramms ist der Data Lake ein Ansatz zur Speicherung von Daten im nativen Format für die Verwendung in einer Organisation. Für diesen Zweck erstellen Sie einen Data Lake für Ihre Organisation mithilfe von {{site.data.keyword.cos_short}}. Durch die Kombination von {{site.data.keyword.cos_short}} und SQL Query können Datenanalysten mithilfe von SQL Daten an den jeweiligen Speicherpositionen abfragen. Außerdem verwenden Sie den SQL Query-Service in einem Jupyter-Notizbuch, um eine einfache Analyse durchzuführen. Anschließend geben Sie Benutzern ohne technisches Knowhow die Möglichkeit, mit dem {{site.data.keyword.dynamdashbemb_notm}} eigene Erkenntnisse zu gewinnen.

## Ziele

- Mit {{site.data.keyword.cos_short}} Rohdaten in Dateien speichern
- Daten mithilfe von SQL Query direkt in {{site.data.keyword.cos_short}} abfragen
- Daten in {{site.data.keyword.DSX_full}} eingrenzen und analysieren
- Daten in Ihrer Organisation über {{site.data.keyword.dynamdashbemb_notm}} gemeinsam nutzen

## Verwendete Services

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [SQL Query](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Architektur

![Architektur](images/solution29/architecture.png)

1. Rohdaten werden in {{site.data.keyword.cos_short}} gespeichert.
2. Daten werden mit SQL Query eingegrenzt, erweitert oder optimiert.
3. Die Datenanalyse wird in {{site.data.keyword.DSX}} durchgeführt.
4. Ein Geschäftsbereich greift auf eine Webanwendung zu.
5. Eingegrenzte Daten werden aus {{site.data.keyword.cos_short}} abgerufen.
6. Diagramme für Geschäftsbereiche werden mit {{site.data.keyword.dynamdashbemb_notm}} erstellt.

## Vorbereitende Schritte

- [Git installieren](https://git-scm.com/)
- [{{site.data.keyword.Bluemix_notm}}-CLI installieren](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
- [Aspera Connect installieren](http://downloads.asperasoft.com/connect2/)
- [Node.js und NPM installieren](https://nodejs.org)

## Services erstellen

In diesem Abschnitt erstellen Sie die erforderlichen Services zum Einrichten Ihres Data Lake.

Im vorliegenden Abschnitt wird die Befehlszeile verwendet, um Serviceinstanzen zu erstellen. Alternativ können Sie zum Erstellen von Instanzen die bereitgestellten Links auf der Seite für Services im Katalog verwenden.
{: tip}

1. Melden Sie sich bei {{site.data.keyword.cloud_notm}} über die Befehlszeile mit Ihrem Cloud Foundry-Konto an. Weitere Informationen finden Sie unter [Einführung in die CLI]https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview.
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Erstellen Sie eine [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)-Instanz mit einem Cloud Foundry-Alias. Wenn Sie bereits über eine Serviceinstanz verfügen, führen Sie den Befehl `service-alias-create` mit dem vorhandenen Servicenamen aus.
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. Erstellen Sie eine Instanz von [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. Erstellen Sie eine [{{site.data.keyword.DSX}}-Instanz](https://{DomainName}/catalog/services/watson-studio).
    ```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. Erstellen Sie eine [{{site.data.keyword.dynamdashbemb_notm}}-Instanz](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded) mit einem Cloud Foundry-Alias.
    ```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. Wechseln Sie in ein Arbeitsverzeichnis und führen Sie den folgenden Befehl aus, um das [GitHub-Repository](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard) der Dashboardanwendung zu klonen. Übertragen Sie anschließend die Anwendung mit einer Push-Operation in Ihre Cloud Foundry-Organisation. Die Anwendung bindet automatisch die erforderlichen Services mithilfe der zugehörigen Datei [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml).
    ```sh
    git clone https://github.com/IBM-Cloud/nodejs-data-lake-dashboard.git
    ```
    {: pre}
    ```sh
    cd nodejs-data-lake-dashboard
    ```
    {: pre}
    ```sh
    npm install
    ```
    {: pre}
    ```sh
    npm run push
    ```
    {: pre}

    Nach der Bereitstellung ist die Anwendung öffentlich zugänglich und überwacht einen beliebigen Hostnamen. Sie können die Seite [Ressourcenliste](https://{DomainName}/resources) aufrufen, die App unter 'Cloud Foundry-Apps' auswählen und die URL anzeigen oder den Befehl `ibmcloud cf app dashboard-nodejs routes` ausführen, um Routen anzuzeigen.
    {: tip}

7. Überprüfen Sie, dass die Anwendung aktiv ist, indem Sie die öffentliche URL der Anwendung im Browser aufrufen.

![Dashboard-Landing-Page](images/solution29/dashboard-start.png)

## Daten hochladen

In diesem Abschnitt laden Sie Daten in einen {{site.data.keyword.cos_short}}-Bucket unter Verwendung der integrierten Komponente {{site.data.keyword.CHSTSshort}}. {{site.data.keyword.CHSTSshort}} schützt die Daten beim Upload in den Bucket und [kann die Übertragungsdauer deutlich verkürzen](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/).

1. Laden Sie die CSV-Datei [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD) herunter. Die Datei hat eine Größe von 81 MB und der Download kann mehrere Minuten dauern.
2. Rufen Sie in Ihrem Browser die Serviceinstanz **data-lake-cos** aus der [Ressourcenliste](https://{DomainName}/resources) auf.
3. Erstellen Sie einen neuen Bucket zum Speichern von Daten.
    - Klicken Sie auf die Schaltfläche **Bucket erstellen**.
    - Wählen Sie den Eintrag **Regional** in der Dropdown-Liste **Ausfallsicherheit** aus.
    - Wählen Sie die Option **us-south** unter **Standort** aus. {{site.data.keyword.CHSTSshort}} ist derzeit nur für Buckets verfügbar, die am Standort `us-south` erstellt wurden. Alternativ können Sie einen anderen Standort auswählen und im nächsten Abschnitt den Übertragungstyp **Standard** auswählen.
    - Geben Sie einen **Namen** für den Bucket an und klicken Sie auf **Erstellen**. Wenn ein Fehler *AccessDenied* ausgegeben wird, geben Sie einen eindeutigeren Bucketnamen an.
4. Laden Sie die CSV-Datei in {{site.data.keyword.cos_short}} hoch.
    - Klicken Sie in Ihrem Bucket auf die Schaltfläche **Objekte hinzufügen**.
    - Wählen Sie das Optionsfeld **Aspera-Hochgeschwindigkeitsübertragung** aus.
    - Klicken Sie auf die Schaltfläche **Dateien hinzufügen**. Daraufhin wird das Plug-in 'Aspera' in einem separaten Fenster geöffnet (möglicherweise hinter Ihrem Browserfenster).
    - Navigieren Sie zu CSV-Datei, die zuvor heruntergeladen wurde.

![Bucket mit CSV-Datei](images/solution29/cos-bucket.png)

## Mit Daten arbeiten

In diesem Abschnitt wandeln Sie die ursprünglichen Rohdaten auf der Basis von Zeit- und Altersattributen in eine Zielkohorte um. Dies ist hilfreich für Data Lake-Konsumenten, die spezielle Interessen verfolgen oder mit sehr umfangreichen Datensätzen überfordert wären.

Sie verwenden SQL Query zum Bearbeiten von Daten an ihrer jeweiligen Speicherposition in {{site.data.keyword.cos_short}} unter Verwendung gängiger SQL-Anweisungen. SQL Query bietet integrierte Unterstützung für CSV, JSON und Parquet, d. h. es sind keine zusätzlichen Berechnungsservices oder Prozesse zum Extrahieren/Umwandeln/Laden erforderlich.

1. Rufen Sie die SQL Query-Serviceinstanz **data-lake-sql** über Ihre [Ressourcenliste](https://{DomainName}/resources) auf.
2. Wählen Sie **Benutzerschnittstelle öffnen** aus.
3. Erstellen Sie ein neues Dataset, indem Sie SQL direkt für die zuvor hochgeladene CSV-Datei ausführen.
    - Geben Sie den folgenden SQL-Code in den Textbereich **SQL hier eingeben...** ein:
        ```
        SELECT
        `Dr Number` AS id,
        `Date Occurred` AS date,
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
        FROM cos://us-south/<your-bucket-name>/Traffic_Collision_Data_from_2010_to_Present.csv
        WHERE
        `Time Occurred` >= 1700 AND
        `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND
        `Victim Age` <= 35
        ```
        {: codeblock}
    - Ersetzen Sie die URL in der Klausel `FROM` durch Ihren Bucketnamen.
4. Das **Ziel** erstellt automatisch einen {{site.data.keyword.cos_short}}-Bucket für das Ergebnis. Ändern Sie das **Ziel** in `cos://us-south/<your-bucket-name>/results`.
5. Klicken Sie auf die Schaltfläche **Ausführen**. Die Ergebnisse werden unten angezeigt.
6. Klicken Sie auf der Registerkarate **Abfragedetails** auf das Symbol **Starten** neben der URL **Ergebnisposition**, um das temporäre Dataset anzuzeigen, das jetzt auch in {{site.data.keyword.cos_short}} gespeichert wird.

![Notizbuch](images/solution29/sql-query.png)

## Jupyter-Notizbücher mit SQL Query kombinieren

In diesem Abschnitt verwenden Sie den SQL Query-Client in einem Jupyter-Notizbuch. Dadurch werden die in {{site.data.keyword.cos_short}} gespeicherten Daten in einem Datenanalysetool wiederverwendet. Darüber hinaus erstellt diese Kombination Datasets, die automatisch in {{site.data.keyword.cos_short}} gespeichert werden und anschließend mit {{site.data.keyword.dynamdashbemb_notm}} verwendet werden können.

1. Erstellen Sie in {{site.data.keyword.DSX}} ein neues Jupyter-Notizbuch.
    - Öffnen Sie [{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true) in einem Browser.
    - Klicken Sie auf die Kachel **Projekt erstellen** vor **Data Science**.
    - Klicken Sie auf **Projekt erstellen** und geben Sie einen **Projektnamen** an.
    - Stellen Sie sicher, dass für **Speicher** der Wert **data-lake-cos** angegeben ist.
    - Klicken Sie auf **Erstellen**.
    - Klicken Sie in dem resultierenden Projekt auf **Zu Projekt hinzufügen** und auf **Notizbuch**.
    - Geben Sie auf der Registerkarte **Leer** einen **Notizbuchnamen** ein.
    - Lassen Sie die Standardeinstellungen für **Sprache** und **Laufzeit** unverändert und klicken Sie auf **Notizbuch erstellen**.
2. Installieren und importieren Sie vom Notizbuch aus 'PixieDust' und 'ibmcloudsql', indem Sie die folgenden Befehle in der Eingabeanforderung **In [ ]: ** hinzufügen und wählen Sie anschließend **Ausführen** aus:
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. Fügen Sie einen {{site.data.keyword.cos_short}}-API-Schlüssel zum Notizbuch hinzu. Dadurch können SQL Query-Ergebnisse in {{site.data.keyword.cos_short}} gespeichert werden.
    - Fügen Sie Folgendes in der nächsten Eingabeaufforderung **In [ ]: ** ein und wählen Sie anschließend **Ausführen** aus:
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - Erstellen Sie über das Terminal einen API-Schlüssel:
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - Kopieren Sie den **API-Schlüssel** in die Zwischenablage.
    - Fügen Sie den API-Schlüssel aus  der Zwischenablage in das Textfeld im Notizbuch ein und drücken Sie anschließend die `Eingabetaste`.
    - Speichern Sie den API-Schlüssel an einer geschützten, permanenten Speicherposition, da er vom Notizbuch nicht gespeichert wird.
4. Fügen Sie den CRN (Cloud Resource Name) der SQL Query-Instanz zum Notizbuch hinzu.
    - Ordnen Sie in der nächsten Eingabeaufforderung **In [ ]: ** den CRN einer Variablen in Ihrem Notizbuch zu.
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - Kopieren Sie vom Terminal aus den CRN aus der Eigenschaft **ID** in die Zwischenablage.
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - Fügen Sie den CRN zwischen den einfachen Anführungszeichen ein und wählen Sie anschließend **Ausführen** aus.
5. Fügen Sie eine weitere Variable zum Notizbuch hinzu, um den {{site.data.keyword.cos_short}}-Bucket anzugeben, und wählen Sie anschließend **Ausführen** aus.
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. Führen Sie die folgenden Befehle in einer weiteren Eingabeaufforderung **In [ ]: ** aus und wählen Sie **Ausführen** aus, um die Ergebnismenge anzuzeigen. Außerdem wird eine neue  Datei `accidents/jobid=<id>/<part>.csv*` in Ihrem Bucket hinzugefügt, die das Ergebnis der Anweisung `SELECT` enthält.
    ```python
    sqlClient = ibmcloudsql.SQLQuery(cloud_api_key, sql_crn, sql_cos_endpoint + '/accidents')

    data_source = sql_cos_endpoint + "/Traffic_Collision_Data_from_2010_to_Present.csv"

    query = """
    SELECT
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
    FROM  {}
    WHERE
        `Time Occurred` >= 1700 AND `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND `Victim Age` <= 35
    """.format(data_source)

    traffic_collisions = sqlClient.run_sql(query)
    traffic_collisions.head()
    ```
    {: codeblock}

## Daten mit PixieDust darstellen

In diesem Abschnitt rufen Sie eine Darstellung der vorherigen Ergebnismenge mit PixieDust und Mapbox auf, um Muster oder Brennpunkte für Datenverkehrsstörungen leichter zu erkennen.

1. Erstellen Sie einen allgemeinen Tabellenausdruck zum Umwandeln der Spalte `Standort` in separate Spalten `Breitengrad` und `Längengrad`. Führen**** Sie den folgenden Code über die Eingabeaufforderung des Notizbuchs aus:
    ```python
    query = """
    WITH location AS (
        SELECT
            id,
            cast(split(coordinates, ',')[0] as float) as latitude,
            cast(split(coordinates, ',')[1] as float) as longitude
        FROM (SELECT
                `Dr Number` as id,
                regexp_replace(Location, '[()]', '') as coordinates
            FROM {0}
        )
    )
    SELECT
        d.`Dr Number` as id,
        d.`Date Occurred` as date,
        d.`Time Occurred` AS time,
        d.`Area Name` AS area,
        d.`Victim Age` AS age,
        d.`Victim Sex` AS sex,
        l.latitude,
        l.longitude
    FROM {0} AS d
        JOIN
        location AS l
        ON l.id = d.`Dr Number`
    WHERE
        d.`Time Occurred` >= 1700 AND
        d.`Time Occurred` <= 2000 AND
        d.`Victim Age` >= 20 AND
        d.`Victim Age` <= 35 AND
        l.latitude != 0.0000 AND
        l.latitude != 0.0000
    """.format(data_source)

    traffic_location = sqlClient.run_sql(query)
    traffic_location.head()
    ```
    {: codeblock}
2. Führen Sie in der nächsten Eingabeaufforderung **In [ ]: ** den**** Befehl `display` aus, um das Ergebnis unter Verwendung von PixieDust anzuzeigen.
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. Wählen Sie die Dropdown-Liste für Diagramme und anschließend **Zuordnen** aus.
4. Fügen Sie `Breitengrad` und `Längengrad` unter **Schlüssel** hinzu. Fügen Sie `ID` und `Alter` unter **Werte** hinzu. Klicken Sie auf **OK**, um die Zuordnung anzuzeigen.
5. Klicken Sie auf das Symbol **Speichern**, um Ihr Notizbuch in {{site.data.keyword.cos_short}} zu speichern.

![Notizbuch](images/solution29/notebook-mapbox.png)

## Dataset für die Organisation freigeben

Nicht jeder Data Lake-Benutzer ist ein Data-Scientist. Mit {{site.data.keyword.dynamdashbemb_notm}} können Sie auch Benutzern ohne technisches Knowhow Einblicke in den Data Lake ermöglichen. Ähnlich wie SQL Query kann{{site.data.keyword.dynamdashbemb_notm}} über vordefinierte Dashboards Daten direkt aus {{site.data.keyword.cos_short}} lesen. In diesem Abschnitt wird eine Lösung beschrieben, die es jedem Benutzer ermöglicht, auf den Data Lake zuzugreifen und ein angepasstes Dashboard zu erstellen.

1. Rufen Sie die öffentliche URL der Dashboardanwendunge auf, die Sie zuvor mit einer Push-Operation in {{site.data.keyword.Bluemix_notm}} übertragen haben.
2. Wählen Sie eine Vorlage aus, die dem gewünschten Layout entspricht. (In den folgenden Schritten wird das zweite Layout aus der ersten Zeile verwendet.)
3. Verwenden Sie die Schaltfläche `Quelle hinzufügen` in der Seitenleiste `Ausgewählte Quellen`, erweitern Sie das Element `Bucketname` und klicken Sie auf einen der Tabelleneinträge `accidents/jobid=...` Schließen Sie das Dialogmodul durch Klicken auf das Symbol 'X' in der rechten oberen Ecke.
4. Klicken Sie auf der linken Seite auf das Symbol `Darstellungen` und anschließend auf **Zusammenfassung**.
5. Wählen Sie die Quelle `accidents/jobid=...` aus, erweitern Sie das Element `Tabelle` und erstellen Sie ein Diagramm.
    - Ziehen Sie das Element `ID` in die Zeile **Wert** und übergeben Sie es dort.
    - Minimieren Sie das Diagramm durch Klicken auf das Symbol in der oberen Ecke.
6. Erstellen Sie über `Darstellungen` ein Diagramm **Treemap**:
    - Ziehen Sie das Element `Bereich` in die Zeile **Bereichshierarchie** und übergeben Sie es dort.
    - Ziehen Sie das Element `ID` in die Zeile **Größe** und übergeben Sie es dort.
    - Minimieren Sie das Diagramm, um das Ergebnis anzuzeigen.

![Dashboarddiagramm](images/solution29/dashboard-chart.png)

## Dashboard erkunden

In diesem Abschnitt führen Sie einige zusätzliche Schritte aus, um die Funktionen der Dashboardanwendung und {{site.data.keyword.dynamdashbemb_notm}} zu erkunden.

1. Klicken Sie auf die Schaltfläche **Modus** in der Symbolleiste der Beispieldashboardanwendung, um die Modusansicht `ANZEIGEN` aufzurufen.
2. Klicken Sie auf eine der farbigen Kacheln im unteren Diagramm oder auf Werte für `Bereich` in der Legende des Diagramms. Dadurch wird ein lokaler Filter auf die Registerkarte angewendet und in den übrigen Diagrammen werden spezifische Daten für den Filter angezeigt.
3. Klicken Sie auf die Schaltfläche **Speichern** in der Symbolleiste.
    - Geben Sie den Namen Ihres Dashboards in das entsprechende Eingabefeld ein.
    - Wählen Sie die Registerkarte **Spec** aus, um die Spezifikation für das Dashboard anzuzeigen. 'Spec' ist das native Dateiformat für {{site.data.keyword.dynamdashbemb_notm}}. Es enthält Informationen zu den Diagrammen, die Sie erstellt haben, und die verwendete {{site.data.keyword.cos_short}}-Datenquelle.
    - Speichern Sie Ihr Dashboard im lokalen Speicher des Browsers unter Verwendung der Schaltfläche **Speichern** im Dialogfenster.
4. Klicken Sie auf die Schaltfläche **Neu** in der Symbolleiste, um ein neues Dashboard zu erstellen. Wenn Sie ein gespeichertes Dashboard öffnen möchten, klicken Sie auf die Schaltfläche **Öffnen**. Wenn Sie ein Dashboard löschen möchten, verwenden Sie das Symbol **Löschen** im Dialogfenster 'Dashboard öffnen'.

In Produktionsanwendungen sollten Informationen wie URLs, Benutzernamen und Kennwörter verschlüsselt werden, damit sie für Endbenutzer nicht sichtbar sind. Weitere Informationen finden Sie im Abschnitt [Datenquelleninformationen verschlüsseln](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information).
{: tip}

## Lernprogramm erweitern

Glückwunsch! Sie haben mit {{site.data.keyword.cos_short}} einen Data Lake erstellt. Im Folgenden finden Sie Vorschläge zum Erweitern des Data Lake.

- Experimentieren Sie mit zusätzlichen Datasets unter Verwendung von SQL Query.
- Übertragen Sie Daten aus mehreren Quellen in Ihren Data Lake, indem Sie das Lernprogramm [Big Data-Protokolle mit Streaming Analytics und SQL](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics) durcharbeiten.
- Bearbeiten Sie den Code der Dashboardanwendung, um Dashboardspezifikationen in [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) oder {{site.data.keyword.cos_short}} zu speichern.
- Erstellen Sie eine [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id)-Serviceinstanz, um die Sicherheit in der Dashboardanwendung zu aktivieren.

## Ressourcen entfernen

Führen Sie die folgenden Befehle aus, um die verwendeten Services, Anwendungen und Schlüssel zu entfernen:

```sh
ibmcloud resource service-binding-delete dashboard-nodejs-dde dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-binding-delete dashboard-nodejs-cos dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-dde
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-cos
```
{: pre}
```sh
ibmcloud iam api-key-delete data-lake-cos-key
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-dde
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-cos
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-sql
```
{: pre}
```sh
ibmcloud service delete data-lake-studio
```
{: pre}
```sh
ibmcloud app delete dashboard-nodejs
```
{: pre}

## Zugehörige Inhalte

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Jupyter-Notizbücher](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
