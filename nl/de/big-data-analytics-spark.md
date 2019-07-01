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

# Offene Daten mit Apache Spark analysieren und visualisieren
{: #big-data-analytics-spark}

In diesem Lernprogramm analysieren und visualisieren Sie offene Datasets mit {{site.data.keyword.DSX_full}}, einem Jupyter-Notebook und Apache Spark. Zunächst kombinieren Sie Daten, die das Bevölkerungswachstum, die Lebenserwartung und die ISO-Ländercodes beschreiben, in einem einzigen Datenrahmen. Um Einblicke zu erhalten, verwenden Sie dann eine Python-Bibliothek namens Pixiedust, um Daten auf verschiedene Arten abzufragen und zu visualisieren.

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## Lernziele
{: #objectives}

* Apache Spark und {{site.data.keyword.DSX_short}} in IBM Cloud implementieren
* Mit einem Jupyter-Notebook und einem Python-Kernel arbeiten
* Datasets importieren, transformieren, analysieren und visualisieren

## Verwendete Services
{: #services}

In diesem Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

Für dieses Lernprogramm können Kosten anfallen. Verwenden Sie den [Preisrechner](https://{DomainName}/pricing/), um basierend auf Ihrer geplanten Verwendung einen Kostenvoranschlag zu generieren.

## Service- und Umgebungskonfiguration
Stellen Sie zunächst die in diesem Lernprogramm verwendeten Services bereit und erstellen Sie ein Projekt in {{site.data.keyword.DSX_short}}.

Sie können Services für {{site.data.keyword.Bluemix_short}} aus der [Ressourcenliste](https://{DomainName}/resources) und dem [Katalog](https://{DomainName}/catalog/) bereitstellen. Alternativ können Sie mit {{site.data.keyword.DSX_short}} die vorhandenen Daten- & Analyseservices über die Dashboard- und Projekteinstellungen erstellen oder hinzufügen.
{:tip}

1. Navigieren Sie vom [{{site.data.keyword.Bluemix_short}}-Katalog](https://{DomainName}/catalog) aus zum Abschnitt **AI**. Erstellen Sie den **{{site.data.keyword.DSX_short}}**-Service. Klicken Sie auf die Startfläche **Einstieg**, um das **{{site.data.keyword.DSX_short}}**-Dashboard zu starten.
2. Klicken Sie im Dashboard auf die Kachel **Projekt erstellen**. > Wählen Sie **Standard** aus. > Erstellen Sie dann das Projekt. Geben Sie in das Feld **Name** `1stProject` als Namen ein. Sie können die Beschreibung leer lassen.
3. Auf der rechten Seite der Seite können Sie den **Speicher definieren**. Wenn Sie bereits Speicher bereitgestellt haben, wählen Sie eine Instanz in der Liste aus. Ist dies nicht der Fall, klicken Sie auf **Hinzufügen** und folgen Sie den Anweisungen auf der neuen Browserregisterkarte. Wenn Sie die Erstellung des Service abgeschlossen haben, klicken Sie auf **Aktualisieren**, um den neuen Service anzuzeigen.
4. Klicken Sie auf die Schaltfläche **Erstellen**, um das Projekt zu erstellen. Sie werden an die Übersichtsseite des Projekts weitergeleitet.  
   ![](images/solution23/NewProject.png)
5. Klicken Sie auf der Übersichtsseite auf **Einstellungen**.
6. Klicken Sie im Abschnitt **Zugeordnete Services** auf **Service hinzufügen** und wählen Sie **Spark** aus dem Menü aus. In der daraufhin geöffneten Anzeige können Sie entweder eine vorhandene Spark-Serviceinstanz auswählen oder eine neue Instanz erstellen.

## Notebook erstellen und vorbereiten
Das [Jupyter-Notebook](http://jupyter.org/) ist eine Open-Source-Webanwendung, mit der Sie Dokumente, die Live-Code, Gleichungen, Visualisierungen und beschreibenden Text enthalten, erstellen und zur gemeinsamen Nutzung freigeben können. Notebooks und andere Ressourcen sind in Projekten organisiert.
1. Klicken Sie auf die Registerkarte **Assets**, blättern Sie abwärts zum Abschnitt **Notebooks** und klicken Sie auf **Neues Notebook**.
2. Verwenden Sie ein **leeres** Notebook. Geben Sie `MyNotebook` als **Name** ein.
3. Wählen Sie im Menü **Laufzeit auswählen** die Spark-Instanz aus, die Sie zu den Projekteinstellungen hinzugefügt haben. Belassen Sie die Standardeinstellung für **Sprache** als **Python 3.5**.
4. Klicken Sie auf **Notebook erstellen**, um den Prozess abzuschließen.
5. Das Feld, in dem Sie Text und Befehle eingeben, wird als **Zelle** bezeichnet. Kopieren Sie den folgenden Code in die leere Zelle, um das [**Pixiedust**-Paket](https://pixiedust.github.io/pixiedust/use.html) zu importieren. Führen Sie die Zelle entweder durch Klicken auf das Symbol **Ausführen** in der Symbolleiste oder durch Drücken der Tastenkombination **Umschalttaste + Eingabetaste** auf der Tastatur aus.
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

Wenn Sie noch nie mit Jupyter-Notebooks gearbeitet haben, klicken Sie im Menü oben rechts auf das Symbol **Docs**. Navigieren Sie zu **Daten analysieren** und dann zum [Abschnitt **Notebooks**](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics), um mehr zu [Notebooks und den entsprechenden Bestandteilen](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true) zu erfahren.
{:tip}

## Daten laden
Laden Sie anschließend drei offene Datasets und stellen Sie sie im Notebook zur Verfügung. Die Bibliothek **Pixiedust** ermöglicht Ihnen das einfache [Laden von **CSV**-Dateien unter Verwendung einer URL](https://pixiedust.github.io/pixiedust/loaddata.html).

1.  Kopieren Sie die folgende Zeile in die nächste leere Zelle in Ihrem Notebook, führen Sie sie aber noch nicht aus.
   ```Python
   df_pop = pixiedust.sampleData('ihr_zugriffs-uri')
   ```
   {:codeblock}
2. Klicken Sie in einer anderen Browserregisterkarte auf den Abschnitt [Community](https://dataplatform.ibm.com/community?context=analytics). Suchen Sie unter **Datasets** nach [**Gesamtbevölkerung nach Land**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e) und klicken Sie auf die Kachel. Klicken Sie oben mit der rechten Maustaste auf das **Link**-Symbol, um einen Zugriffs-URI zu erhalten. Kopieren Sie den URI und ersetzen Sie den Text **ihr_zugriffs-uri** in der Notebook-Zelle durch den Link. Klicken Sie entweder auf das Symbol **Ausführen** in der Symbolleiste oder drücken Sie die Tastenkombination **Umschalttaste + Eingabetaste**.
3. Wiederholen Sie den Schritt für ein weiteres Dataset. Kopieren Sie die folgende Zeile in die nächste leere Zelle in Ihrem Notebook.
   ```Python
   df_life = pixiedust.sampleData(ihr_zugriffs-uri)
   ```
   {:codeblock}
4. Suchen Sie in der anderen Browserregisterkarte mit den **Dateigruppen** nach [**Lebenserwartung bei Geburt nach Land in Jahren insgesamt**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895). Rufen Sie den Link erneut ab und verwenden Sie ihn, um **ihr_zugriffs-uri** in der Notebookzelle zu ersetzen. Klicken Sie dann auf **Ausführen**, um den Ladeprozess zu starten.
5. Laden Sie für die letzten drei Datasets eine Liste mit Ländernamen und deren ISO-Codes aus einer Sammlung offener Datasets auf Github. Kopieren Sie den Code in die nächste leere Notebookzelle und führen Sie ihn aus.
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

Die Liste der Ländercodes wird später verwendet, um die Datenauswahl zu vereinfachen, indem anstelle des exakten Ländernamens ein Ländercode verwendet wird.

## Daten transformieren
Nachdem die Daten zur Verfügung gestellt wurden, können Sie sie problemlos transformieren und die drei Datensets in einem einzelnen Datenrahmen zusammenfassen.
1. Mit dem folgenden Codeblock wird der Datenrahmen für die Bevölkerungsdaten neu definiert. Dies geschieht mit einer SQL-Anweisung, die die Spalten umbenennt. Anschließend wird eine Sicht erstellt und das Schema wird gedruckt. Kopieren Sie diesen Code in die nächste leere Zelle und führen Sie sie aus.
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. Wiederholen Sie die Schritte für die Daten zur Lebenserwartung. Statt das Schema zu drucken, gibt dieser Code die ersten 10 Zeilen aus.  
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. Wiederholen Sie die Transformation des Schemas für die Länderdaten.
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. Die Spaltennamen sind jetzt einfacher und über Datasets hinweg gleich, die zu einem einzigen Datenrahmen kombiniert werden können. Führen Sie einen **Outer** Join für die Lebenserwartungs- und Bevölkerungsdaten aus. Fügen Sie dann in derselben Anweisung durch einen **Inner** Join die Ländercodes hinzu. Alles wird nach Land und Jahr geordnet. Die Ausgabe definiert den Datenrahmen **df_all**. Durch die Verwendung eines Inner Joins enthalten die resultierenden Daten nur Länder aus der ISO-Liste. Dieser Prozess entfernt regionale und andere Einträge aus den Daten.
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. Ändern Sie den Datentyp für **Year** in eine ganze Zahl.
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

Ihre kombinierten Daten können jetzt analysiert werden.

## Daten analysieren
Verwenden Sie in diesem Teil [Pixiedust, um die Daten in verschiedenen Diagrammen darzustellen.](https://pixiedust.github.io/pixiedust/displayapi.html) Vergleichen Sie zunächst die Lebenserwartung für einige Länder.

1. Kopieren Sie den Code in die nächste leere Zelle und führen Sie ihn aus.
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. Es wird eine verschiebbare Tabelle angezeigt. Klicken Sie auf das Diagrammsymbol direkt unter dem Codeblock und wählen Sie **Kurvendiagramm** aus. Es wird ein Popup-Dialogfeld mit den **Pixiedust: Kurvendiagrammoptionen** angezeigt. Geben Sie einen **Diagrammtitel** wie "Vergleich der Lebenserwartung" ein. Ziehen Sie aus den angebotenen **Feldern** den Eintrag **Year** in das Feld **Schlüssel** und **Life** in den Bereich **Werte**. Geben Sie **1000** für **Anzahl der anzuzeigenden Zeilen** ein. Klicken Sie auf **OK**, um das Kurvendiagramm darzustellen. Stellen Sie auf der rechten Seite sicher, dass **mapplotlib** als **Wiedergabefunktion** ausgewählt ist. Klicken Sie auf den Selektor **Cluster bilden nach** und wählen Sie **Country** aus. Es wird ein Diagramm ähnlich dem folgenden angezeigt.
   ![](images/solution23/LifeExpectancy.png)

3. Erstellen Sie ein Diagramm, das sich auf das Jahr 2010 konzentriert. Kopieren Sie den Code in die nächste leere Zelle und führen Sie ihn aus.
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. Wählen Sie im Diagrammselektor **Karte** aus. Ziehen Sie im Konfigurationsdialog **Country** in den Bereich **Schlüssel**. Verschieben Sie **Life** in das Feld **Werte**. Erhöhen Sie ähnlich wie beim ersten Diagramm die **Anzahl der anzuzeigenden Zeilen** auf **1000**. Klicken Sie auf **OK**, um die Karte abzubilden. Wählen Sie **brunel** als **Wiedergabefunktion** aus. Es wird eine Weltkarte dargestellt, in der die Lebenserwartung in den verschiedenen Ländern farblich dargestellt ist. Sie können mithilfe der Maus in die Karte zoomen.
   ![](images/solution23/LifeExpectancyMap2010.png)

## Lernprogramm erweitern
Im Folgenden finden Sie einige Ideen und Vorschläge, um dieses Lernprogramm zu erweitern.
* Erstellen und visualisieren Sie eine Abfrage mit der Lebenserwartung in Bezug auf das Bevölkerungswachstum für ein Land Ihrer Wahl.
* Berechnen und visualisieren Sie die Bevölkerungszuwachsraten pro Land auf einer Weltkarte
* Laden und integrieren Sie zusätzliche Daten aus dem Katalog der Datasets.
* Exportieren Sie die kombinierten Daten in eine Datei oder Datenbank.

## Zugehöriger Inhalt
{:related}
Im Folgenden finden Sie Links zu den Themen, die in diesem Lernprogramm behandelt werden.
* [Watson Data Platform](https://dataplatform.ibm.com): Verwenden Sie Watson Data Platform, um zusammen zu arbeiten und intelligentere Anwendungen zu erstellen. Visualisieren Sie Daten und erhalten Sie schnell Einblicke in Ihre Daten und arbeiten Sie mit verschiedenen Teams zusammen.
* [PixieDust](https://www.ibm.com/cloud/pixiedust): Ein Open-Source-Produktivitätstool für Jupyter Notebooks.
* [Cognitive Class.ai](https://cognitiveclass.ai/): Data Science- und Cognitive Computing-Kurse.
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/): Was wir können, können Sie auch.
* [Analytics Engine-Service](https://{DomainName}/catalog/services/analytics-engine): Entwickeln Sie Analyseanwendungen und stellen Sie diese mithilfe von Open-Source-Lösungen wie Apache Spark und Apache Hadoop bereit.
