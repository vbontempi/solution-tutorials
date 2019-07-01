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

# SQL-Datenbank für Clouddaten
{: #sql-database}

In diesem Lernprogramm erfahren Sie, wie ein (relationaler) SQL-Datenbankservice bereitgestellt, eine Tabelle erstellt  und ein umfangreiches Dataset (mit Informationen zu Städten) in die Datenbank geladen wird. Anschließend implementieren Sie die Web-App 'worldcities' (Weltstädte), die auf diese Daten zurückgreift und den Zugriff auf die Clouddatenbank veranschaulicht. Die App wurde in Python unter Verwendung des [Flask-Frameworks](http://flask.pocoo.org/) geschrieben.

![](images/solution5/Architecture.png)

## Ziele

* SQL-Datenbank bereitstellen
* Datenbankschema (Tabelle) erstellen
* Daten laden
* App und Datenbankservice verbinden (Berechtigungsnachweise gemeinsam nutzen)
* Überwachung, Sicherheit, Backup & Wiederherstellung

## Produkte

Im vorliegenden Lernprogramm werden die folgenden Produkte verwendet:
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## Vorbereitende Schritte
{: #prereqs}

Rufen Sie [GeoNames](http://www.geonames.org/) auf, laden Sie die Datei [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip) herunter und dekomprimieren Sie die Datei. Sie enthält Informationen zu Städten, die mehr als 1000 Einwohner haben. Diese Datei wird im Folgenden als Dataset verwendet.

## SQL-Datenbank bereitstellen
Erstellen Sie zu Beginn eine Instanz des **[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)**-Service.

![](images/solution5/Catalog.png)

1. Rufen Sie das [{{site.data.keyword.Bluemix_short}}-Dashboard](https://{DomainName}) auf. Klicken Sie in der oberen Navigationsleiste auf **Katalog**.
2. Klicken Sie im linken Teilfenster unter 'Plattform' auf **Daten & Analyse** und wählen Sie **{{site.data.keyword.dashdbshort_notm}}** aus.
3. Wählen Sie den **Entry**-Plan aus und ändern Sie den vorgeschlagenen Servicenamen in 'sqldatabase' (dieser Name wird später verwendet). Wählen Sie eine Position für die Bereitstellung der Datenbank aus und stellen Sie sicher, dass die richtige Organisation und der richtige Bereich ausgewählt sind.
4. Klicken Sie auf **Erstellen**. Nach kurzer Wartezeit müsste eine Nachricht auf die erfolgreiche Ausführung hinweisen.
5. Klicken Sie in der **Ressourcenliste** auf den Eintrag für Ihren neu erstellten {{site.data.keyword.dashdbshort_notm}}-Service.
6. Klicken Sie auf **Öffnen**, um die Datenbankkonsole zu starten. Wenn Sie die Konsole zum ersten Mal verwenden, wird eine geführte Tour für die Konsole angeboten.

## Tabelle erstellen
Sie benötigen eine Tabelle für die Beispieldaten. Erstellen Sie diese Tabelle mithilfe der Konsole.

1. Klicken Sie in der Konsole für {{site.data.keyword.dashdbshort_notm}} in der Navigationsleiste auf **Erkunden**. Daraufhin wird eine Liste mit den in der Datenbank vorhandenen Schemas angezeigt.
2. Suchen Sie das Schema, das mit der Zeichenfolge 'DASH' beginnt, und klicken Sie darauf.
3. Klicken Sie auf **"+ Neue Tabelle"**, um ein Formular für den Namen und die Spalten der Tabelle zu öffnen.
4. Geben Sie 'städte' als Tabellennamen ein. Kopieren Sie die Spaltendefinitionen aus der Datei [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) und fügen Sie sie in das Feld für die Spalten und Datentypen ein.
5. Klicken Sie auf **Erstellen**, um die neue Tabelle zu definieren.   
   ![](images/solution5/TableCitiesCreated.png)

## Daten laden
Als Nächstes laden Sie Daten in die neu erstellte Tabelle 'städte'. Dies kann auf verschiedene Arten erfolgen, z. B. von Ihrer lokalen Maschine aus oder aus Cloud Object Storage (COS) mit Swift oder mit der Amazon S3-Schnittstelle unter Verwendung des [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift)-Migrationsservice. Im vorliegenden Lernprogramm laden Sie Daten aus Ihrer Maschine hoch. Während dieses Vorgangs passen Sie die Tabellenstruktur und das Datenformat an den Inhalt der Datei an.

1. Klicken Sie in der oberen Navigationsleiste auf **Laden**. Klicken Sie anschließend unter **Dateiauswahl** auf **Dateien durchsuchen**, um die Datei 'cities1000.txt' zu finden und auszuwählen, die Sie im ersten Abschnitt dieser Anleitung heruntergeladen haben.
2. Klicken Sie auf **Weiter**, um die Schemaübersicht aufzurufen. Wählen Sie erneut das Schema aus, dessen Name mit 'DASH' beginnt, und danach die Tabelle 'CITIES'. Klicken Sie erneut auf **Weiter**.   

   Da die Tabelle leer ist, spielt es keine Rolle, ob neue Daten angehängt oder vorhandene Daten überschrieben werden.
   {:tip }
3. Passen Sie nun an, wie die Daten aus der Datei 'cities1000.txt' beim Ladeprozess interpretiert werden. Inaktivieren Sie zuerst die Option 'Header in erster Zeile', da die Datei ausschließlich Daten enthält. Geben Sie danach '0x09' als Trennzeichen ein. Dies bedeutet, dass Werte in der Datei durch Tabulatorzeichen begrenzt werden. Wählen Sie schließlich 'JJJJ-MM-TT' als Datumsformat aus. Nun müsste die Darstellung dem folgenden Screenshot entsprechen.    
  ![](images/solution5/LoadTabSeparator.png)
4. Klicken Sie auf **Weiter**. Daraufhin wird angeboten, die Ladeeinstellungen zu überprüfen. Bestätigen Sie die Aufforderung und klicken Sie auf **Ladevorgang starten**, um mit dem Laden der Daten in die Tabelle 'STÄDTE' zu beginnen. Der Fortschritt des Ladevorgangs wird angezeigt. Nach dem Hochladen der Daten müssten schon nach wenigen Sekunden verschiedene Statistiken angezeigt werden.   
   ![](images/solution5/LoadProgressSteps.png)

## Geladene Daten mit SQL überprüfen
Die Daten wurden in die relationale Datenbank geladen. Obwohl keine Fehler gemeldet wurden, sollten Sie einige Schnelltests durchführen. Geben Sie im integrierten SQL-Editor verschiedene SQL-Anweisungen ein und führen Sie diese aus.

1. Klicken Sie in der oberen Navigationsleiste auf **SQL ausführen**.
   Anstelle des integrierten SQL-Editors können Sie cloudbasierte und konventionelle SQL-Tools auf Ihrem Desktop oder Ihrer Servermaschine mit {{site.data.keyword.dashdbshort_notm}} verwenden. Die Verbindungsinformationen finden Sie im Menü für Einstellungen. Einige Tools werden im Abschnitt 'Downloads' zum Herunterladen angeboten, das hinter dem Buchsymbol für Dokumentation und Hilfe verfügbar ist.
    {:tip }
2. Geben Sie die folgende Abfrage in den SQL-Editor ein oder kopieren Sie sie in den Editor:   
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   Aktivieren Sie anschließend die Schaltfläche **Alle ausführen**. Im Abschnitt für Ergebnisse müsste die gleiche Zeilenanzahl angezeigt werden wie vom Ladeprozess gemeldet.   
3. Geben Sie im SQL-Editor die folgende Anweisung in eine neue Zeile ein:
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. Wählen Sie im Editor den Text der obigen Anweisung aus. Klicken Sie auf die Schaltfläche **Ausgewählte ausführen**. Daraufhin müsste diese Anweisung ausgeführt werden, die nach Land geordnete Statistikdaten im Abschnitt für Ergebnisse anzeigt.

## Anwendungscode bereitstellen
Der lauffähige [Code für die Datenbank-App ist in diesem GitHub-Repository enthalten](https://github.com/IBM-Cloud/cloud-sql-database). Klonen Sie das Repository oder laden Sie es herunter und übertragen Sie es anschließend mit einer Push-Operation in IBM Cloud.

1. Klonen Sie das GitHub-Repository:
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. Übertragen Sie die Anwendung mit einer Push-Operation in IBM Cloud. Sie müssen bei dem Standort, der Organisation und dem Bereich angemeldet sein, für die die Datenbank bereitgestellt wurde. Kopieren und übergeben Sie diese Befehle Zeile für Zeile.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. Nachdem die Push-Operation abgeschlossen ist, sollten Sie auf die App zugreifen können. Es sind keine weiteren Konfigurationsschritte erforderlich. Die Datei `manifest.yml` weist IBM Cloud an, die App und den Datenbankservice mit dem Namen 'sqldatabase' miteinander zu verknüpfen.

## Sicherheit, Sicherung & Wiederherstellung, Überwachung
{{site.data.keyword.dashdbshort_notm}} ist ein verwalteter Service. IBM ist für den Schutz der Umgebung sowie für tägliche Backups und die Systemüberwachung zuständig. Im Entry-Plan wird für die Datenbankumgebung eine Multi-Tenant-Konfiguration mit reduziertem Verwaltungsaufwand und konfigurierten Benutzeroptionen verwendet. Wenn Sie jedoch einen der Enterprise-Pläne verwenden, stehen [mehrere Optionen zum Verwalten von Benutzern, zum Konfigurieren zusätzlicher Datenbanksicherheit](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html) und zum [Überwachen der Datenbank](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html) zur Verfügung.   

Neben den traditionellen Verwaltungsoptionen bietet der [{{site.data.keyword.dashdbshort_notm}}-Service auch eine REST-API für Überwachung, Benutzerverwaltung, Dienstprogramme, Ladeprozess, Speicherzugriff usw](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html). Die ausführbare Swagger-Schnittstelle dieser API kann über das Menü hinter dem Buchsymbol unter 'REST-APIs' aufgerufen werden. Einige Tools (z. B. IBM Data Server Manager) für die Überwachung und für weitere Funktion können im Abschnitt 'Downloads' dieses Menüs heruntergeladen werden.

## App testen
Die App zum Anzeigen von Städteinformationen aus dem geladenen Dataset ist auf ein Minimum reduziert. Sie enthält ein Suchformular zum Angeben eines Stadtnamens und wenige vorkonfigurierte Städte. Diese Werte werden entweder in `/search?name=cityname` (Suchformular) oder in `/city/cityname` (direkt angegebene Städte) umgesetzt. Beide Anforderungen werden aus denselben Codezeilen im Hintergrund bedient. Der Name der Stadt wird als Wert an eine vorbereitete SQL-Anweisung übergeben (dabei wird aus Sicherheitsgründen eine Parametermarke verwendet). Die Zeilen werden aus der Datenbank abgerufen und zum Darstellen an eine HTML-Vorlage übergeben.

## Bereinigung
Führen Sie die folgenden Schritte aus, um die im Lernprogramm verwendeten Ressourcen zu bereinigen:
1. Rufen Sie die [{{site.data.keyword.Bluemix_short}}-Ressourcenliste](https://{DomainName}/resources) auf. Suchen Sie Ihre App.
2. Klicken Sie auf das Menüsymbol für die App und wählen Sie **App löschen** aus. Markieren Sie im Dialogfenster die zu löschende Ressource mit einem Häkchen, um den zugehörigen {{site.data.keyword.dashdbshort_notm}}-Service zu löschen.
3. Klicken Sie auf die Schaltfläche **Löschen**. Die App und der Datenbankservice werden entfernt und die Ressourcenliste wird wieder angezeigt.

## Lernprogramm erweitern
Sie möchten diese App erweitern? Hier einige Vorschläge:
1. Bieten Sie eine Platzhaltersuche nach alternativen Namen an.
2. Suchen Sie nach Städten in einem bestimmten Land und mit bestimmten Einwohnerzahlen.
3. Ändern Sie das Seitenlayout, indem Sie die CSS-Stile ersetzen und die Vorlagen erweitern.
4. Lassen Sie die formularbasierte Erstellung neuer Städteinformationen oder Aktualisierungen der vorhandenen Daten (z. B. Bevölkerung) zu.

## Zugehörige Inhalte
* Dokumentation: [IBM Knowledge Center für {{site.data.keyword.dashdbshort_notm}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* In [Häufig gestellte Fragen zu {{site.data.keyword.Db2_on_Cloud_long_notm}} und {{site.data.keyword.dashdblong_notm}}](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html) werden Fragen zum verwalteten Service, zur Datensicherung, zur Datenverschlüsselung, zur Sicherheit und zu vielen anderen Themen beantwortet.
* [Kostenlose Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) für Entwickler
* Dokumentation: [API-Beschreibung für den Python-Treiber für ibm_db](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
