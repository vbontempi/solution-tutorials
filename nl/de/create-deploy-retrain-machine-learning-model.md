---
copyright:
  years: 2018, 2019
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

# Modell für maschinelles Lernen erstellen, implementieren, testen und erneut trainieren
{: #create-deploy-retrain-machine-learning-model}
In diesem Lernprogramm erhalten Sie eine Anleitung zum Erstellen eines prädiktiven Modells für maschinelles Lernen, zum Implementieren dieses Modells als in eine in Anwendungen zu verwendenden API, zum Testen des Modells und zum erneuten Training des Modells mit Feedbackdaten. All dies geschieht in einem integrierten und einheitlichen Self-Service-Erlebnis in IBM Cloud.

In diesem Lernprogramm wird das **Dataset 'Iris'** verwendet, um ein Modell für maschinelles Lernen zum Klassifizieren von Blumenarten zu erstellen.

In der Terminologie des maschinellen Lernens wird die Klassifizierung als eine Instanz des kontrollierten Lernens betrachtet, d. h. zu lernen, wo ein Trainingsset mit korrekt identifizierten Beobachtungen zur Verfügung steht.
{:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## Lernziele
{: #objectives}

* Daten in ein Projekt importieren
* Modell für maschinelles Lernen erstellen
* Modell implementieren und API testen
* Modell für maschinelles Lernen testen
* Verbindung für Feedbackdaten zur fortlaufenden Bewertung von Lernprozessen und Modellen erstellen
* Erneutes Training für das Modell

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## Vorbereitende Schritte
{: #prereqs}
* IBM Watson Studio und Watson Knowledge Catalog sind Anwendungen, die Teil von IBM Watson sind. Um ein IBM Watson-Konto zu erstellen, registrieren Sie sich zunächst für eine oder beide Anwendungen.

   Wechseln Sie zur Seite für das [Testen von IBM Watson](https://dataplatform.ibm.com/registration/stepone) und registrieren Sie sich für IBM Watson-Apps.

## Daten in ein Projekt importieren

{:#import_data_project}

Ein Projekt ist die Art und Weise, wie Sie Ihre Ressourcen organisieren, um ein bestimmtes Ziel zu erreichen. Ihre Projektressourcen können Daten, Collaboratoren und Analysewerkzeuge wie Jupyter-Notebooks und Modelle für maschinelles Lernen enthalten.

Sie können ein Projekt erstellen, um Daten hinzuzufügen und ein Datenasset im Datenrefinery-Tool zu öffnen, um Ihre Daten zu bereinigen und zu gestalten.

**Erstellen Sie ein Projekt:**

1. Rufen Sie den [{{site.data.keyword.Bluemix_short}}-Katalog](https://{DomainName}/catalog) auf und wählen Sie [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services) unter dem Abschnitt **AI** auf. **Erstellen** Sie den Service. Klicken Sie auf die Startfläche **Einstieg**, um das **{{site.data.keyword.DSX_short}}**-Dashboard zu starten.
2. Erstellen Sie ein **Projekt**. > Klicken Sie dazu auf der Kachel **Standard** auf **Projekt erstellen**. Fügen Sie den Namen `iris_project` und eine optionale Beschreibung für das Projekt hinzu.
3. Belassen Sie das Kontrollkästchen zum **Einschränken darüber, wer ein Collaborator sein kann** inaktiviert, da es keine vertrauliche Daten gibt.
4. Klicken Sie unter **Speicher definieren** auf **Hinzufügen** und wählen Sie einen vorhandenen Cloud Object Storage-Service aus oder erstellen Sie einen neuen Service (unter **Lite**-Plan > Erstellen). Klicken Sie auf **Aktualisieren**, um den erstellten Service anzuzeigen.
5. Klicken Sie auf **Erstellen**. Ihr neues Projekt wird geöffnet und Sie können mit dem Hinzufügen von Ressourcen zu diesem Projekt beginnen.

**Importieren Sie Daten:**

Wie bereits erwähnt, verwenden Sie das **Dataset 'Iris'**. Dieses Dataset wurde in der bekannten Abhandlung von R.A. Fisher über die [Verwendung mehrerer Messungen in taxonomischen Problemen](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf) aus dem Jahr 1936 verwendet und ist außerdem in der [UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/) zu finden. Dieses kleine Dataset wird häufig zum Testen von Algorithmen und Visualisierungen für maschinelles Lernen verwendet. Ziel ist es, die Iris-Blumen anhand von Messungen zu Länge und Bereite von Kelchblättern und Blüten in drei Arten (Setosa, Versicolor oder Virginica) zu klassifizieren. Das Dataset 'Iris' enthält drei Klassen mit jeweils 50 Instanzen, wobei sich jede Klasse auf eine Art von Iris-Pflanze bezieht.
![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


**Laden Sie** die Datei [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv) herunter, die aus 40 Instanzen von jeder Klasse besteht. Mit den restlichen 10 Klassen jeder Klasse führen Sie dann ein erneutes Training des Modells durch.

1. Klicken Sie unter **Assets** in Ihrem Projekt auf das Symbol **Daten suchen und hinzufügen** ![Zeigt das Symbol zum Suchen von Daten](images/solution22-build-machine-learning-model/data_icon.png).
2. Klicken Sie unter **Laden** auf **Durchsuchen** und laden Sie die heruntergeladene Datei `iris_initial.csv` hoch.
      ![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. Nach dem Hinzufügen sollte der Abschnitt `iris_initial.csv` unter dem Abschnitt **Datenressourcen** des Projekts angezeigt werden. Klicken Sie auf den Namen, um den Inhalt des Datasets anzuzeigen.

## Services zuordnen
{:#associate_services}
1. Blättern Sie unter **Einstellungen** zu **Zugeordnete Services**. > Klicken Sie auf **Service hinzufügen**. > Wählen Sie **Spark** aus.
   ![](images/solution22-build-machine-learning-model/associate_services.png)
2. Wählen Sie den **Lite**-Plan aus und klicken Sie auf **Erstellen**. Verwenden Sie die Standardwerte und klicken Sie auf **Bestätigen**.
3. Klicken Sie erneut auf **Service hinzufügen** und wählen Sie **Watson** aus. Klicken Sie auf der Kachel für **Maschinelles Lernen** auf **Hinzufügen**. > Wählen Sie den **Lite**-Plan aus. > Klicken Sie auf **Erstellen**.
4. Behalten Sie die Standardwerte bei und klicken Sie auf **Bestätigen**, um einen Service für maschinelles Lernen bereitzustellen.

## Modell für maschinelles Lernen erstellen

{:#build_model}

1. Klicken Sie auf **Zu Projekt hinzufügen** und wählen Sie das **Watson-Modell für maschinelles Lernen** aus. Fügen Sie im Dialogfeld **iris_model** als Namen und eine optionale Beschreibung hinzu.
2. Unter dem Abschnitt mit dem **Service für maschinelles Lernen** sollte der entsprechende Service angezeigt werden, den Sie im obigen Schritt zugeordnet haben.
   ![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. Wählen Sie **Modellbuilder** als Modelltyp und unter **Spark-Service oder -Umgebung** den Spark-Service aus, den Sie zuvor erstellt haben.
4. Wählen Sie **Manuell** aus, um ein Modell manuell zu erstellen. Klicken Sie auf **Erstellen**.

   Für die automatische Methode verlassen Sie sich vollständig auf die automatische Datenaufbereitung (ADP, Automatic Data Preparation). Bei der manuellen Methode können Sie zusätzlich zu einigen Funktionen, die vom ADP-Transformator verarbeitet werden, eigene Schätzfunktionen hinzufügen und konfigurieren, die die in der Analyse verwendeten Algorithmen sind.
   {:tip}

5. Wählen Sie auf der nächsten Seite `iris_initial.csv` als Dataset aus und klicken Sie auf **Weiter**.
6. Auf der Seite zum **Auswählen eines Verfahrens** sind Bezeichnungsspalten und Merkmalspalten basierend auf dem hinzufügten Dataset bereits ausgefüllt. Wählen Sie **species (String)** (Arten) als **Bezeichnungsspalten** und **petal_length (Decimal)** (Blütenlänge) und **petal_width (Decimal)** (Blütenbreite) als **Merkmalspalten** aus.
7. Wählen **Klassifikation mit mehreren Klassen** als empfohlenes Verfahren aus.
   ![](images/solution22-build-machine-learning-model/model_technique.png)
8. Konfigurieren Sie für **Validierungsaufteilung** die folgende Einstellung:

   **Training:** 50%,
   **Test** 25%,
   **Holdout:** 25%

9. Klicken Sie auf **Schätzer hinzufügen** und wählen Sie **Entscheidungsstruktur-Klassifikationsmerkmal** und dann **Hinzufügen** aus.

   Sie können mehrere Schätzer auf einmal bewerten. Sie können beispielsweise das **Klassifikationsmerkmal Entscheidungsstruktur** und das **Klassifikationsmerkmal Random Forest** als Schätzer zum Trainieren des Modells hinzufügen und basierend auf der Bewertungsausgabe das Passende auszuwählen.
   {:tip}

10. Klicken Sie auf **Weiter**, um das Training für das Modell durchzuführen. Sobald der Status als **Trainiert & Bewertet** angezeigt wird, klicken Sie auf **Speichern**.
   ![](images/solution22-build-machine-learning-model/trained_model.png)

11. Klicken Sie auf **Übersicht**, um die Details des Modells zu überprüfen.

## Implementieren Sie das Modell und testen Sie die API.

{:#deploy_model}

1. Klicken Sie unter dem erstellten Modell auf **Implementierungen** > **Implementierung hinzufügen**.
2. Wählen Sie **Web-Service** aus. Fügen Sie als Beispiel den Namen `iris_deployment` und eine optionale Beschreibung hinzu.
3. Klicken Sie auf **Speichern**. Klicken Sie auf der Übersichtsseite auf den Namen des neuen Web-Service. Sobald der Status angibt, dass die **Bereitstellung erfolgreich** war, können Sie den Scoring-Endpunkt, Codefragmente in verschiedenen Programmiersprachen und die API-Spezifikation unter **Implementierung** überprüfen.
4. Klicken Sie auf **API-Spezifikation anzeigen**, um {{site.data.keyword.pm_short}}-API-Endpunkte anzuzeigen und zu testen.
   ![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   Um mit der API arbeiten zu können, müssen Sie ein **Zugriffstoken** mit dem **Benutzernamen** und dem **Kennwort** generieren, die auf der Registerkarte **Berechtigungsnachweise** der {{site.data.keyword.pm_short}}-Serviceinstanz unter der [{{site.data.keyword.Bluemix_short}}-Ressourcenliste](https://{DomainName}/resources/) verfügbar sind. Befolgen Sie die Anweisungen auf der Seite der API-Spezifikation, um ein **Zugriffstoken** zu generieren.
   {:tip}
5. Um eine Online-Vorhersage zu erstellen, verwenden Sie den API-Aufruf `POST /online`.
   * Die Instanz-ID (`instance_id`) befindet sich auf der Registerkarte **Serviceberechtigungen** des {{site.data.keyword.pm_short}}-Service unter der [{{site.data.keyword.Bluemix_short}}-Ressourcenliste](https://{DomainName}/resources/).
   * Die Implementierungs-ID (`deployment_id`) und die ID des veröffentlichten Modells (`published_model_id`) befinden sich in der **Übersicht** der Implementierung.
   *  Verwenden Sie für `online_prediction_input` die unten stehende JSON.

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * Klicken Sie auf **Testen**, um die JSON-Ausgabe anzuzeigen.

6. Unter Verwendung der API-Endpunkte können Sie dieses Modell jetzt über jede beliebige Anwendung aufrufen.

## Modell testen

{:#test_model}

1. Unter **Testen** sollten Sie Eingabedaten (Merkmaldaten) sehen, die automatisch gefüllt werden.
2. Klicken Sie auf **Vorhersagen**. In einem Diagramm sollte der **vorhergesagte Wert für Arten** angezeigt werden.
   ![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. Klicken Sie für eine JSON-Eingabe und -Ausgabe auf die Symbole neben der aktiven Eingabe und Ausgabe.
4. Sie können die Eingabedaten ändern und das Modell weiter testen.

## Verbindung für Feedbackdaten erstellen

{:#create_feedback_connection}

1. Für die kontinuierliche Lern- und Modellbewertung müssen Sie irgendwo neue Daten speichern können. Erstellen Sie einen [{{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse)-Service > **Entry**-Plan, der als Verbindung für Feedbackdaten fungiert.
2. Klicken Sie auf der {{site.data.keyword.dashdbshort}}-Seite **Verwalten** auf **Öffnen**. Wählen Sie in der oberen Navigationsleiste die Option **Laden** aus.
3. Klicken Sie unter **Arbeitsplatz** auf **Dateien durchsuchen** und laden Sie die Datei `iris_initial.csv` hoch. Klicken Sie auf **Weiter**.
4. Wählen Sie **DASHXXXX**, z. B. DASH1234, als **Schema** aus und klicken Sie dann auf **Neue Tabelle**. Nennen Sie die Tabelle `IRIS_FEEDBACK` und klicken Sie auf **Weiter**.
5. Datentypen werden automatisch erkannt. Klicken Sie auf **Weiter** und dann auf die Option zum **Starten des Ladevorgangs**.
   ![](images/solution22-build-machine-learning-model/define_table.png)
6. Es wird ein neues Ziel **DASHXXXX.IRIS_FEEDBACK** erstellt.

   Sie werden dies im nächsten Schritt verwenden, in dem Sie das Modell erneut trainieren werden, um eine bessere Leistung und Genauigkeit zu erhalten.

## Modell erneut trainieren

{:#retrain_model}

1. Kehren Sie zu Ihrer [{{site.data.keyword.Bluemix_short}}-Ressourcenliste](https://{DomainName}/resources) zurück und klicken Sie unter dem {{site.data.keyword.DSX_short}}-Service, den Sie verwendet haben, auf **Projekte** > 'iris_project' >  **iris-model** (unter 'Assets') > 'Bewertung'.
2. Klicken Sie unter **Leistungsüberwachung** auf **Leistungsüberwachung konfigurieren**.
3. Führen Sie auf der Seite zur Leistungsüberwachung die folgenden Schritte aus:
   * Wählen Sie den Spark-Service aus. Der Vorhersagetyp sollte automatisch gefüllt werden.
   * Wählen Sie **weightedPrecision** als Metrik aus und legen Sie `0,98` als optionalen Schwellenwert fest.
   * Klicken Sie auf **Neue Verbindung erstellen**, um auf den IBM Db2 Warehouse on Cloud-Service zu verweisen, den Sie im obigen Abschnitt erstellt haben.
   * Wählen Sie die Db2 Warehouse-Verbindung aus und klicken Sie auf **Erstellen**, sobald die Verbindungsdetails mit Daten gefüllt wurden.
     ![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * Klicken Sie auf die Option zum **Auswählen der Feedbackdatenreferenz**, verweisen Sie auf die Tabelle IRIS_FEEDBACK und klicken Sie auf **Auswählen**.
     ![](images/solution22-build-machine-learning-model/select_source.png)
   * Geben Sie im Feld mit der **für die Neubewertung erforderlichen Anzahl an Datensätzen** die Mindestanzahl neuer Datensätze an, bei der ein erneutes Training erforderlich wird. Verwenden Sie **10** oder lassen Sie das Feld leer, um den Standardwert '1000' zu verwenden.
   * Wählen Sie im Feld für das **automatische erneute Training** eine der folgenden Optionen aus:
     - Wenn das automatische erneute Training immer dann gestartet werden soll, wenn die Modellleistung unter dem von Ihnen angegebenen Schwellenwert liegt, wählen Sie die Option **Wenn Modellleistung unter dem Schwellenwert liegt** aus. Für dieses Lernprogramm wählen Sie diese Option aus, da die angegebene Genauigkeit unter dem Schwellenwert (0,98) liegt.
     - Um das automatische erneute Training zu verhindern, wählen Sie die Option **Niemals** aus.
     - Wenn das automatische erneute Training unabhängig von der Leistung gestartet werden soll, wählen Sie die Option **Immer** aus.
   * Wählen Sie im Feld für die **automatische Bereitstellung** eine der folgenden Optionen aus:
     - Um die automatische Bereitstellung zu starten, wenn die Modellleistung besser als die vorherige Version ist, wählen Sie **Wenn die Modellleistung besser als die vorherige Version ist** aus. Für dieses Lernprogramm wählen Sie diese Option aus, da es Ziel ist, die Leistung des Modell stetig zu verbessern.
     - Um die automatische Bereitstellung zu verhindern, wählen Sie die Option **Niemals** aus.
     - Wenn die automatische Bereitstellung unabhängig von der Leistung gestartet werden soll, wählen Sie die Option **Immer** aus.
   * Klicken Sie auf **Speichern**.
     ![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. Laden Sie die Datei [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv) herunter. Klicken Sie anschließend auf **Feedbackdaten hinzufügen**, wählen Sie die heruntergeladene CSV-Datei aus und klicken Sie auf **Öffnen**.
5. Klicken Sie zum Starten auf **Neue Bewertung**.
     ![](images/solution22-build-machine-learning-model/retraining_model.png)
6. Sobald die Bewertung abgeschlossen ist, können Sie den Abschnitt **Letztes Bewertungsergebnis** auf den verbesserten Wert für **WeightedPrecision** überprüfen.

## Ressourcen entfernen
{:removeresources}

1. Navigieren Sie zur [{{site.data.keyword.Bluemix_short}}-Ressourcenliste](https://{DomainName}/resources/). > Wählen Sie die Position, die Organisation und den Bereich aus, an denen Sie die Services erstellt haben. 
2. Löschen Sie die entsprechenden {{site.data.keyword.DSX_short}}-, {{site.data.keyword.sparks}}-, {{site.data.keyword.pm_short}}-, {{site.data.keyword.dashdbshort}}- und {{site.data.keyword.cos_short}}-Services, die Sie für dieses Lernprogramm erstellt haben.

## Zugehöriger Inhalt
{:related}

- [Übersicht zu Watson Studio](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [Anomalien mithilfe von maschinellem Lernen erkennen](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [Automatische Modellerstellung](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [Maschinelles Lernen & AI](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
