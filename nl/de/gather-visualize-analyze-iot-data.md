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

# IoT-Daten sammeln, visualisieren und analysieren
{: #gather-visualize-analyze-iot-data}
In diesem Lernprogramm erfahren Sie, wie Sie ein IoT-Gerät einrichten, Daten im {{site.data.keyword.iot_short_notm}} erfassen, Daten untersuchen und Visualisierungen erstellen und dann mithilfe von erweiterten Services für maschinelles Lernen Daten analysieren und Unregelmäßigkeiten in den Protokolldaten erkennen können.
{:shortdesc}

## Lernziele
{: #objectives}

* IoT-Simulator einrichten
* Objektgruppendaten an {{site.data.keyword.iot_short_notm}} senden
* Visualisierungen erstellen
* Vom Gerät generierte Daten analysieren und Unregelmäßigkeiten erkennen

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Node.js-Anwendung](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience) mit Spark-Service und {{site.data.keyword.cos_full_notm}}
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Architektur
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* Geräte senden mithilfe des MQTT-Protokolls Sensordaten an {{site.data.keyword.iot_full}}.
* Protokolldaten werden in eine {{site.data.keyword.cloudant_short_notm}}-Datenbank exportiert.
* {{site.data.keyword.DSX_short}} extrahiert Daten mit einer Pull-Operation aus dieser Datenbank.
* Daten werden durch ein Jupyter-Notebook analysiert und visualisiert.

## Vorbereitende Schritte
{: #prereqs}

[{{site.data.keyword.Bluemix_notm}}-Entwicklertools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - Führen Sie das Script aus, um die 'ibmcloud'-CLI und die erforderlichen Plug-ins zu installieren.

## IoT-Plattform erstellen
{: #iot_starter}

Zuerst erstellen Sie den IoT-Plattformservice (IoT, Internet der Dinge), den Hub, über den Geräte verwaltet, sichere Verbindungen hergestellt und **Daten erfasst** werden können, und stellen Protokolldaten für Visualisierungen und Anwendungen zur Verfügung.

1. Rufen Sie den [**{{site.data.keyword.Bluemix_notm}}-Katalog**](https://{DomainName}/catalog/) auf und wählen Sie die [**Internet of Things-Plattform**](https://{DomainName}/catalog/services/internet-of-things-platform) unter dem Abschnitt **Internet of Things** aus.
2. Geben Sie `IoT demo hub` als Servicenamen ein, klicken Sie auf **Erstellen** und **starten** Sie das Dashboard.
3. Wählen Sie im Seitenmenü **Sicherheit > Verbindungssicherheit** und anschließend **TLS Optional** unter **Standardregel** > **Sicherheitsstufe** aus. Klicken Sie dann auf **Speichern**.
4. Wählen Sie im Seitenmenü **Geräte** > **Gerätetypen** und **+ Gerätetyp hinzufügen** aus.
5. Geben Sie `simulator` als **Name** ein und klicken Sie auf **Weiter** und **Fertig**.
6. Klicken Sie anschließend auf **Geräte registrieren**.
7. Wählen Sie `simulator` für **Vorhandenen Gerätetyp auswählen** aus und geben Sie dann `phone` als **Geräte-ID** ein.
8. Klicken Sie so lange auf **Weiter**, bis die Anzeige **Gerätesicherheit** (auf der Registerkarte 'Sicherheit') erscheint.
9. Geben Sie einen Wert für das **Authentifizierungstoken** ein, z. B. `myauthtoken`, und klicken Sie auf **Weiter**.
10. Wenn Sie auf **Fertig** geklickt haben, werden Ihre Verbindungsinformationen angezeigt. Lassen Sie diese Registerkarte geöffnet.

Die IoT-Plattform ist jetzt für den Empfang von Daten konfiguriert. Geräte müssen ihre Daten mit dem angegebenen Gerätetyp, der angegebenen ID und dem angegebenen Token an die IoT-Plattform senden.

## Gerätesimulator erstellen
{: #confignodered}
Als Nächstes stellen Sie eine Node.js-Webanwendung bereit und greifen auf diese über Ihr Telefon zu, das eine Verbindung zur IoT-Plattform herstellt und Beschleunigungssensordaten und Ausrichtungsdaten an die Plattform sendet.

1. Klonen Sie das Github-Repository:
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. Öffnen Sie den Code in einer IDE Ihrer Wahl und ändern Sie die Werte für `name` und `host` in der Datei **manifest.yml** in einen eindeutigen Wert.
3. Übertragen Sie die Anwendung mit einer Push-Operation an {{site.data.keyword.Bluemix_notm}}.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. In nur wenigen Minuten wird Ihre Anwendung implementiert und Sie sollten eine ähnliche URL wie `<UNIQUE_NAME>.mybluemix.net` sehen.
5. Rufen Sie diese URL über einen Browser auf Ihrem Telefon auf.
6. Geben Sie die Verbindungsinformationen von der Registerkarte 'IoT-Dashboard' unter **Geräteberechtigungsnachweise** ein und klicken Sie auf **Verbinden**.
7. Ihr Telefon beginnt mit der Übertragung von Daten. Suchen Sie auf der **Registerkarte für IBM {{site.data.keyword.iot_short_notm}}** im Abschnitt **Kürzliche Ereignisse** nach neuen Einträgen.
  ![](images/solution16/recent_events_with_phone.png)

## Livedaten in IBM {{site.data.keyword.iot_short_notm}} anzeigen
{: #createcards}
Jetzt erstellen Sie ein Board und Karten, um Gerätedaten im Dashboad anzuzeigen. Weitere Informationen zu Boards und Karten finden Sie im Abschnitt zum [Visualisieren von Echtzeitdaten mithilfe von Boards und Karten](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html).

### Board erstellen
{: #createboard}

1. Öffnen Sie das **IBM {{site.data.keyword.iot_short_notm}}-Dashboard**.
2. Wählen Sie **Boards** im linken Menü aus und klicken Sie dann auf **Neues Board erstellen**.
3. Geben Sie einen Namen für das Board ein, z. B. `Simulators`, und klicken Sie auf **Weiter** und anschließend auf **Übergeben**.  
4. Wählen Sie das Board aus, das Sie gerade erstellt haben, um es zu öffnen.

### Gerätedaten anzeigen
{: #cardtemp}
1. Klicken Sie auf **Neue Karte hinzufügen** und wählen Sie dann den Kartentyp **Kurvendiagramm** aus, der sich im Abschnitt 'Geräte' befindet.
2. Wählen Sie Ihr Gerät in der Liste aus und klicken Sie dann auf **Weiter**.
3. Klicken Sie auf **Neues Dataset verbinden**.
4. Wählen Sie auf der Seite 'Wertkarte erstellen' die folgenden Werte aus oder geben Sie sie ein und klicken Sie auf **Weiter**.
   - Ereignis: sensorData
   - Eigenschaft: ob
   - Name: OrientationBeta
   - Typ: Float
   - Min: -180
   - Max: 180
5. Wählen Sie auf der Seite 'Kartenvorschau' die Option **K** für die Kurvendiagrammgröße aus und klicken Sie auf **Weiter** > **Übergeben** .
6. Die Karte wird im Dashboard angezeigt und enthält ein Kurvendiagramm der Live-Temperaturdaten.
7. Verwenden Sie den Browser Ihres Mobiltelefons, um den Simulator erneut zu starten, und neigen Sie das Telefon langsam nach vorne und nach hinten.
8. Auf der **Registerkarte für IBM {{site.data.keyword.iot_short_notm}}** sollte zu sehen sein, dass das Diagramm aktualisiert wurde.
   ![](images/solution16/board.png)

## Protokolldaten in {{site.data.keyword.cloudant_short_notm}} speichern
1. Rufen Sie den [**{{site.data.keyword.Bluemix_notm}}-Katalog**](https://{DomainName}/catalog/) auf und erstellen Sie eine neue [{{site.data.keyword.cloudant_short_notm}}-Instanz](https://{DomainName}/catalog/services/cloudant) mit dem Namen `iot-db`.
2. Führen Sie unter **Verbindungen** die folgenden Schritte aus:
   1. **Erstellen Sie eine Verbindung**.
   1. Wählen Sie die Cloud Foundry-Position, die Organisation und den Bereich aus, in denen ein Aliasname für den {{site.data.keyword.cloudant_short_notm}}-Service erstellt werden soll.
   1. Erweitern Sie den Bereichsnamen in der Tabelle **Verbindungsposition** und verwenden Sie die Schaltfläche **Verbinden** neben **iot demo hub**, um einen Aliasnamen für den {{site.data.keyword.cloudant_short_notm}}-Service in diesem Bereich zu erstellen. 
   1. Verbinden Sie die App und führen Sie ein erneutes Staging durch.
3. Öffnen Sie das **IBM {{site.data.keyword.iot_short_notm}}-Dashboard**.
4. Wählen Sie im linken Menü **Erweiterungen** aus und klicken Sie dann unter **Protokolldatenspeicher** auf **Einrichten**.
5. Wählen Sie die {{site.data.keyword.cloudant_short_notm}}-Datenbank `iot-db` aus.
6. Geben Sie `devicedata` als **Datenbankname** ein und klicken Sie auf **Fertig**.
7. Es sollte ein neues Fenster geladen werden, in dem die Autorisierung angefordert wird. Wenn Sie dieses Fenster nicht sehen, inaktivieren Sie Ihren Popup-Blocker und aktualisieren Sie die Seite.

Ihre Gerätedaten werden jetzt in {{site.data.keyword.cloudant_short_notm}} gespeichert. Starten Sie nach ein paar Minuten das {{site.data.keyword.cloudant_short_notm}}-Dashboard, um Ihre Daten anzuzeigen.

![](images/solution16/cloudant.png)

## Anomalien mithilfe von maschinellem Lernen erkennen
{: #data_experience}

In diesem Abschnitt verwenden Sie das Jupyter-Notebook, das im IBM {{site.data.keyword.DSX_short}}-Service verfügbar ist, um Ihre mobilen Protokolldaten zu laden und Anomalien mithilfe des z-Scores zu ermitteln. Der *z-Score* ist ein Standardscore, der die Differenz eines Rohwertes vom Mittelwert eines Elements (Standardabweichung) angibt.

### Neues Projekt erstellen
1. Rufen Sie den [**{{site.data.keyword.Bluemix_notm}}-Katalog**](https://{DomainName}/catalog/) auf und wählen Sie unter **AI** die Option [**{{site.data.keyword.DSX_short}}**](https://{DomainName}/catalog/services/data-science-experience) aus.
2. **Erstellen** Sie den Service und starten Sie das Dashboard, indem Sie auf **Einstieg** klicken.
3. Erstellen Sie ein Projekt. > Wählen Sie **Data-Science** aus. > Klicken Sie auf **Projekt erstellen** und geben Sie `Detect Anomaly` (Anomalie erkennen) als **Name** des Projekts ein.
4. Belassen Sie das Kontrollkästchen zum **Einschränken darüber, wer ein Collaborator sein kann** inaktiviert, da es keine vertrauliche Daten gibt.
5. Klicken Sie unter **Speicher definieren** auf **Hinzufügen** und wählen Sie einen vorhandenen **Cloud Object Storage**-Service aus oder erstellen Sie einen neuen Service (unter **Lite**-Plan > Erstellen). Klicken Sie auf **Aktualisieren**, um den erstellten Service anzuzeigen.
6. Klicken Sie auf **Erstellen**. Ihr neues Projekt wird geöffnet und Sie können mit dem Hinzufügen von Ressourcen zu diesem Projekt beginnen.

### Verbindung zu {{site.data.keyword.cloudant_short_notm}} für Daten

1. Klicken Sie auf **Assets** > **+ Zu Projekt hinzufügen** > **Verbindung**.  
2. Wählen Sie die {{site.data.keyword.cloudant_short_notm}}-Datenbank **iot-db** aus, in der die Servicedaten gespeichert sind.
3. Überprüfen Sie die **Berechtigungsnachweise** und klicken Sie dann auf **Erstellen**.

### Apache Spark-Service erstellen

1. Klicken Sie in der oberen Navigationsleiste auf **Services**. > Wählen Sie dann die Option zum Berechnen von Services aus.
2. Klicken Sie auf **Service hinzufügen**.
   1. Klicken Sie auf **Hinzufügen** für **Apache Spark**.
   1. Wählen Sie den **Lite**-Plan aus.
   1. Klicken Sie auf **Erstellen**.
3. Wählen Sie eine Organisation und einen Bereich aus, ändern Sie bei Bedarf den Servicenamen und wählen Sie **Bestätigen** aus.
1. Navigieren Sie über **Projekte** zum Projekt `Detect Anomaly`.
1. Blättern Sie in den **Einstellungen** zu **Zugeordnete Services**.
1. Klicken Sie auf **Service hinzufügen** und wählen Sie **Spark** aus.
1. Wählen Sie die zuvor erstellte **Apache Spark**-Instanz aus.

### Jupyter-Notebook (ipynb) erstellen
1. Klicken Sie auf **+ Zu Projekt hinzufügen** und fügen Sie ein neues **Notebook** hinzu.
2. Geben Sie `Anomaly-detection-notebook` als **Name** ein.
3. Geben Sie `https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb` in das Feld für die **Notebook-URL** ein.
4. Wählen Sie den zuvor zugeordneten **Apache Spark**-Service als Laufzeit aus.
5. Erstellen Sie das **Notebook**. Legen Sie `Python 3.5 mit Spark 2.1` als Kernel fest. Stellen Sie sicher, dass das Notebook mit Metadaten und Code erstellt wurde.
   ![Jupyter-Notebook - Watson Studio](images/solution16/jupyter_notebook_watson_studio.png)
   Um eine Aktualisierung durchzuführen, klicken Sie auf **Kernel** > 'Kernel ändern'. Um das Notebook als **vertrauenswürdig** einzustufen, klicken Sie auf **Datei** > 'Notebook vertrauen'.
   {:tip}

### Notebook ausführen und Anomalien feststellen   
1. Wählen Sie die Zelle aus, die mit `!pip install --upgrade pixiedust` beginnt, und klicken Sie anschließend auf **Ausführen** oder verwenden Sie Tastenkombination **Strg + Eingabetaste**, um den Code auszuführen.
2. Wenn die Installation abgeschlossen ist, starten Sie den Spark-Kernel erneut, indem Sie auf das Symbol **Kernel neu starten** klicken.
3. Importieren Sie in der nächsten Codezelle Ihre {{site.data.keyword.cloudant_short_notm}}-Berechtigungsnachweise in diese Zelle, indem Sie die folgenden Schritte ausführen:
   * Klicken Sie auf ![](images/solution16/data_icon.png).
   * Wählen Sie die Registerkarte **Verbindungen** aus.
   * Klicken Sie auf **In Code einfügen**. Ein Wörterverzeichnis mit dem Namen _credentials_1_ wird mit Ihren {{site.data.keyword.cloudant_short_notm}}-Berechtigungsnachweisen erstellt. Wenn der Name nicht als _credentials_1_ angegeben ist, benennen Sie das Verzeichnis in `credentials_1` um. `credentials_1` wird in den verbleibenden Zellen verwendet.
4. Geben Sie in der Zelle mit dem Datenbanknamen (`dbName`) den Namen der {{site.data.keyword.cloudant_short_notm}}-Datenbank ein, die die Quelle der Daten ist, z. B. *iotp_yourWatsonIoTProgId_DBName_Year-month-day*. Ändern Sie die Werte von `deviceId` und `deviceType` entsprechend, um Daten verschiedener Geräte darzustellen.
   Suchen Sie die genaue Datenbank, indem Sie zu Ihrer zuvor erstellten {{site.data.keyword.cloudant_short_notm}}-Instanz **iot-db** navigieren. > Starten Sie von dort aus das Dashboard.
   {:tip}
5. Speichern Sie das Notebook und führen Sie die einzelnen Codezellen nacheinander oder alle Zellen aus (**Zelle** > Alle ausführen). Am Ende des Notebooks sollten Anomalien für die Bewegungsdaten des Geräts (oa,ob und og) angezeigt werden.
   Sie können das Zeitintervall, das für Sie von Interesse ist, in die gewünschte Tageszeit ändern. Suchen Sie nach den Werten für `start` und `end`.
   {:tip}
   ![Jupyter-Notebook - DSX](images/solution16/anomaly_detection_watson_studio.png)
6. Neben der Erkennung von Anomalien haben Sie in diesem Abschnitt die folgenden wesentlichen Aspekte kennengelernt:
    * Verwendung von Spark zur Vorbereitung der Daten für die Visualisierung.
    * Nutzung von Pandas für die Datenvisualisierung.
    * Balkendiagramme, Histogramme für Gerätedaten.
    * Korrelation zwischen zwei Sensoren über die Korrelationsmatrix.
    * Ein Boxplot für jeden Sensor, der mit der Plot-Funktion von Pandas erzeugt wird.
    * Dichtediagramme anhand der Kernel-Dichteschätzung (KDE).
    ![](images/solution16/density_plots_sensor_data.png)

## Ressourcen entfernen
{:removeresources}

1. Navigieren Sie zur [Ressourcenliste](https://{DomainName}/resources/). > Wählen Sie die Position, die Organisation und den Bereich aus, an denen Sie die App und die Services erstellt haben. Löschen Sie unter **Cloud Foundry-Apps** die Node.js-App, die Sie weiter oben im Abschnitt erstellt haben.
2. Löschen Sie unter **Services** die entsprechende Internet of Things-Plattform, Apache Spark, {{site.data.keyword.cloudant_short_notm}} und {{site.data.keyword.cos_full_notm}}-Services, die Sie für dieses Lernprogramm erstellt haben.

## Zugehöriger Inhalt
{:related}

* [Modell für maschinelles Lernen erstellen, implementieren, testen und erneut trainieren](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* Übersicht über [IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html)
* Anomalieerkennung mit [Jupyter-Notebook](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb)
* [Informationen zu z-Score](https://en.wikipedia.org/wiki/Standard_score)
* Kognitive IoT-Lösungen für die Anomalieerkennung durch die Verwendung von Deep Learning entwickeln - [5 Post Series](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/)
