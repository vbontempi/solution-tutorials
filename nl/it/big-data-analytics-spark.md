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

# Analizza e visualizza i dati aperti con Apache Spark
{: #big-data-analytics-spark}

In questa esercitazione, analizzerai e visualizzerai delle serie di dati aperti utilizzando {{site.data.keyword.DSX_full}}, un notebook Jupyter e Apache Spark. Inizierai combinando i dati che descrivono la crescita della popolazione, l'aspettativa di vita e i codici ISO dei paesi in un singolo frame di dati. Per rilevare informazioni approfondite, utilizzerai una libreria Python chiamata Pixiedust per eseguire query e visualizzare i dati in vari modi.

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## Obiettivi
{: #objectives}

* Distribuire Apache Spark e {{site.data.keyword.DSX_short}} su IBM Cloud
* Lavorare con un notebook Jupyter e un kernel Python
* Importare, trasformare, analizzare e visualizzare serie di dati

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Configurazione del servizio e dell'ambiente
Inizia eseguendo il provisioning dei servizi utilizzati in questa esercitazione e crea un progetto all'interno di {{site.data.keyword.DSX_short}}.

Puoi eseguire il provisioning dei servizi per {{site.data.keyword.Bluemix_short}} dall'[elenco delle risorse](https://{DomainName}/resources) e dal [catalogo](https://{DomainName}/catalog/). In alternativa, {{site.data.keyword.DSX_short}} ti consente di creare o aggiungere i servizi Data & Analytics esistenti dal suo dashboard e dalle sue impostazioni del progetto.
{:tip}

1. Dal [catalogo {{site.data.keyword.Bluemix_short}}](https://{DomainName}/catalog), vai alla sezione **AI**. Crea il servizio **{{site.data.keyword.DSX_short}}**. Fai clic sul pulsante **Get Started** per avviare il dashboard **{{site.data.keyword.DSX_short}}**.
2. Nel dashboard, fai clic sul tile **Create a project** > seleziona **Standard** > Create project. Nel campo **Name**, immetti il nome `1stProject`. Puoi lasciare vuoto il campo della descrizione.
3. Sul lato destro della pagina, puoi definire l'archiviazione facendo clic su **Define storage**. Se hai già eseguito il provisioning dell'archiviazione, seleziona un'istanza dall'elenco. In caso contrario, fai clic su **Add** e segui le istruzioni nella nuova scheda del browser. Una volta terminato con la creazione del servizio, fai clic su **Refresh** per vedere il nuovo servizio.
4. Fai clic sul pulsante **Create** per creare il progetto. Verrai reindirizzato alla pagina di panoramica del progetto.  
   ![](images/solution23/NewProject.png)
5. Nella pagina di panoramica, fai clic su **Settings**.
6. Nella sezione **Associated services**, fai clic su **Add Service** e seleziona **Spark** dal menu. Nella schermata risultante, puoi scegliere un'istanza del servizio Spark esistente o crearne una nuova.

## Crea e prepara un notebook
Il [notebook Jupyter](http://jupyter.org/) è un'applicazione web open-source che ti consente di creare e condividere documenti che contengono codice attivo, equazioni, visualizzazioni e testo narrativo. I notebook e le altre risorse sono organizzati in progetti.
1. Fai clic sulla scheda **Assets**, scorri verso il basso fino alla sezione **Notebooks** e fai clic su **New notebook**.
2. Utilizza il notebook **Blank**. Immetti `MyNotebook` nel campo **Name**.
3. Dal menu **Select runtime**, scegli l'istanza Spark che hai aggiunto alle impostazioni del progetto. Per **Language** mantieni l'impostazione predefinita **Python 3.5**.
4. Fai clic su **Create Notebook** per completare il processo.
5. Il campo in cui immetti il testo e i comandi è chiamato **cella**. Copia il seguente codice nella cella vuota per importare il [pacchetto **Pixiedust**](https://pixiedust.github.io/pixiedust/use.html). Esegui la cella facendo clic sull'icona **Run** nella barra degli strumenti o premendo **Maiusc+Invio** sulla tastiera.
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

Se non hai mai lavorato con i notebook Jupyter, fai clic sull'icona **Docs** nel menu in alto a destra. Passa a **Analyze data** e quindi alla [sezione **Notebooks**](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics) per ulteriori informazioni sui [notebook e le loro parti](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true).
{:tip}

## Carica i dati
Successivamente carica tre serie di dati aperti e rendile disponibili all'interno del notebook. La libreria **Pixiedust** ti consente facilmente di [caricare i file **CSV** utilizzando un URL](https://pixiedust.github.io/pixiedust/loaddata.html).

1.  Copia la seguente riga nella successiva cella vuota del tuo notebook, ma non eseguirla ancora.
   ```Python
   df_pop = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
2. In un'altra scheda del browser, vai alla sezione [Community](https://dataplatform.ibm.com/community?context=analytics). In **Data Sets**, cerca [**Total population by country**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e) e fai clic sul tile. Nell'agolo superiore destro, fai clic sull'icona del **link** per ottenere un URI di accesso. Copia l'URI e sostituisci il testo **YourAccessURI** nella cella del notebook con il link. Fai clic sull'icona **Run** nella barra degli strumenti o premi **Maiusc+Invio**.
3. Ripeti il passo per un'altra serie di dati. Copia la seguente riga nella successiva cella vuota del tuo notebook.
   ```Python
   df_life = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
4. Nell'altra scheda del browser con **Data Sets**, cerca [**Life expectancy at birth by country in total years**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895). Ottieni di nuovo il link e utilizzalo per sostituire **YourAccessURI** nella cella del notebook e fai clic su **Run** per avviare il processo di caricamento.
5. Per l'ultima delle tre serie di dati, carica un elenco di nomi di paese e i relativi codici ISO da una raccolta di serie di dati aperti su Github. Copia il codice nella successiva cella vuota del notebook ed eseguila.
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

L'elenco dei codici paese verrà utilizzato in seguito per semplificare la selezione dei dati utilizzando un codice paese anziché il nome esatto del paese.

## Trasforma i dati
Dopo aver reso disponibili i dati, trasformali leggermente e combina le tre serie in un singolo frame di dati.
1. Il seguente blocco di codice ridefinirà il frame di dati per i dati della popolazione. Questa operazione viene effettuata con un'istruzione SQL che rinomina le colonne. Viene quindi creata una vista e stampato lo schema. Copia questo codice nella successiva cella vuota ed eseguila.
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. Ripeti lo stesso processo per i dati sull'aspettativa di vita. Invece di stampare lo schema, questo codice stampa le prime 10 righe.  
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. Ripeti la trasformazione dello schema per i dati del paese.
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. I nomi delle colonne sono ora più semplici e uguali per tutte le serie di dati, che possono essere combinati in un singolo frame di dati. Esegui un'unione **esterna** sui dati relativi all'aspettativa di vita e alla popolazione. Quindi, nella stessa istruzione, esegui un'unione **interna** per aggiungere i codici paese. Tutti i dati saranno ordinati per paese e anno. L'output definisce il frame di dati **df_all**. Utilizzando un'unione interna, i dati risultanti contengono solo i paesi che si trovano nell'elenco ISO. Questo processo rimuove le voci regionali e di altro tipo dai dati.
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. Modifica il tipo di dati per **Year** in un numero intero.
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

I tuoi dati combinati sono pronti per essere analizzati.

## Analizza i dati
In questa parte, utilizza [Pixiedust per visualizzare i dati in diversi grafici](https://pixiedust.github.io/pixiedust/displayapi.html). Inizia confrontando l'aspettativa di vita per alcuni paesi.

1. Copia il codice nella successiva cella vuota ed eseguila.
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. Viene visualizzata una tabella scorrevole. Fai clic sull'icona del grafico direttamente sotto il blocco di codice e seleziona **Line Chart**. Verrà visualizzata una finestra di dialogo a comparsa con **Pixiedust: Line Chart Options**. In **Chart Title** immetti un titolo per il grafico come "Comparison of Life Expectancy". Dai **campi** offerti, trascina **Year** nella casella **Keys** e **Life** nell'area **Values**. Immetti **1000** per **# of Rows to Display**. Premi **OK** per tracciare il grafico a linee. Sul lato destro, assicurati che **mapplotlib** sia selezionato come **Renderer**. Fai clic sul selettore **Cluster By** e scegli **Country**. Verrà mostrato un grafico simile al seguente.
   ![](images/solution23/LifeExpectancy.png)

3. Crea un grafico focalizzato sull'anno 2010. Copia il codice nella successiva cella vuota ed eseguila.
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. Nel selettore del grafico, scegli **Map**. Nella finestra di dialogo di configurazione, trascina **Country** nell'area **Keys**. Sposta **Life** nella casella **Values**. Come nel primo grafico, aumenta il valore di **# of Rows to Display** a **1000**. Premi **OK** per tracciare la mappa. Scegli **brunel** come **Renderer**. Viene mostrata una mappa del mondo relativa all'aspettativa di vita. Puoi utilizzare il mouse per ingrandire la mappa.
   ![](images/solution23/LifeExpectancyMap2010.png)

## Espandi l'esercitazione
Di seguito trovi alcune idee e suggerimenti per migliorare questa esercitazione.
* Crea e visualizza una query che mostra il tasso di aspettativa di vita relativo alla crescita della popolazione per un paese di tua scelta
* Calcola e visualizza i tassi di crescita della popolazione per paese su una mappa del mondo
* Carica e integra dati aggiuntivi dal catalogo di serie di dati
* Esporta i dati combinati in un file o in un database

## Contenuto correlato
{:related}
Di seguito vengono forniti dei link relativi agli argomenti trattati in questa esercitazione.
* [Watson Data Platform](https://dataplatform.ibm.com): utilizza Watson Data Platform per collaborare e creare applicazioni più intelligenti. Visualizza e rileva rapidamente le informazioni approfondite dai tuoi dati e collabora con i team.
* [PixieDust](https://www.ibm.com/cloud/pixiedust): strumento di produttività open source per i notebook Jupyter
* [Cognitive Class.ai](https://cognitiveclass.ai/): corsi di Data Science e Cognitive Computing
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/): esempi e discussioni su tutto ciò che è possibile fare con i dati
* [Servizio Analytics Engine](https://{DomainName}/catalog/services/analytics-engine): sviluppa e distribuisci applicazioni di analisi utilizzando Apache Spark e Apache Hadoop open source
