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

# Crea, distribuisci, verifica e riaddestra un modello di machine learning predittivo
{: #create-deploy-retrain-machine-learning-model}
Questa esercitazione ti guida nel processo di creazione di un modello di machine learning predittivo, distribuendolo come API da utilizzare nelle applicazioni, verificando il modello e riaddestrandolo con i dati di feedback. Tutto ciò avviene in un'esperienza self-service integrata e unificata su IBM Cloud.

In questa esercitazione, viene utilizzata la **serie di dati di fiori Iris** per creare un modello di machine learning per classificare le specie di fiori.

Nella terminologia del machine learning, la classificazione è considerata un'istanza di apprendimento con supervisione, vale a dire l'apprendimento in cui è disponibile un set di addestramento di osservazioni correttamente identificate.
{:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## Obiettivi
{: #objectives}

* Importare i dati in un progetto.
* Creare un modello di machine learning.
* Distribuire il modello e provare l'API.
* Verificare un modello di machine learning.
* Creare una connessione dati di feedback per l'apprendimento continuo e la valutazione del modello.
* Riaddestra il tuo modello.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## Prima di iniziare
{: #prereqs}
* IBM Watson Studio e Watson Knowledge Catalog sono applicazioni che fanno parte di IBM Watson. Per creare un account IBM Watson, inizia registrandoti per una o entrambe queste applicazioni.

   Vai a [Try IBM Watson](https://dataplatform.ibm.com/registration/stepone) e registrati per le applicazioni IBM Watson.

## Importa i dati in un progetto

{:#import_data_project}

Un progetto è il modo in cui organizzi le tue risorse per raggiungere un obiettivo particolare. Le risorse del tuo progetto possono includere dati, collaboratori e strumenti di analisi come i notebook Jupyter e i modelli di machine learning.

Puoi creare un progetto per aggiungere dati e aprire una risorsa dati nel programma di affinamento per pulire e modellare i dati.

**Crea un progetto:**

1. Vai al [catalogo {{site.data.keyword.Bluemix_short}}](https://{DomainName}/catalog) e seleziona [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services) nella sezione **AI**. **Crea** il servizio. Fai clic sul pulsante **Get Started** per avviare il dashboard **{{site.data.keyword.DSX_short}}**.
2. Crea un **progetto** > Fai clic su **Create Project** nel tile **Standard**. Aggiungi un nome come `iris_project` e una descrizione facoltativa per il progetto.
3. Lascia la casella di spunta **Restrict who can be a collaborator** deselezionata poiché non ci sono dati riservati.
4. In **Define Storage**, fai clic su **Add** e scegli un servizio Cloud Object Storage esistente oppure creane uno nuovo (seleziona **Lite** plan > Create). Premi **Refresh** per visualizzare il servizio creato.
5. Fai clic su **Create**. Il tuo nuovo progetto viene aperto e puoi iniziare ad aggiungere risorse ad esso.

**Importa i dati:**

Come accennato in precedenza, utilizzerai la **serie di dati Iris**. La serie di dati Iris è stata introdotta nel 1936 da R.A. Fisher in un articolo intitolato [The Use of Multiple Measurements in Taxonomic Problems](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf) ed è disponibile anche in [UCI Machine Learning Repository](http://archive.ics.uci.edu/ml/). Questa piccola serie di dati viene spesso utilizzata per testare algoritmi e visualizzazioni di machine learning. L'obiettivo è quello di classificare i fiori Iris tra tre specie (Setosa, Versicolor o Virginica) in base alle misurazioni di lunghezza e larghezza di sepali e petali. La serie di dati Iris contiene 3 classi di 50 istanze ciascuna, in cui ogni classe fa riferimento a un tipo di pianta Iris.
![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


**Scarica** [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv) che consiste di 40 istanze di ogni classe. Utilizzerai le restanti 10 istanze di ogni classe per riaddestrare il tuo modello.

1. Nella sezione **Assets** del tuo progetto, fai clic sull'icona **Find and Add Data** ![Mostra l'icona Find data.](images/solution22-build-machine-learning-model/data_icon.png).
2. In **Load**, fai clic su **browse** e carica il file `iris_initial.csv` scaricato.
      ![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. Una volta aggiunto, dovresti visualizzare `iris_initial.csv` nella sezione **Data assets** del progetto. Fai clic sul nome per visualizzare i contenuti della serie di dati.

## Associa i servizi
{:#associate_services}
1. In **Settings**, scorri fino a **Associated services** > fai clic su **Add service** > scegli **Spark**.
   ![](images/solution22-build-machine-learning-model/associate_services.png)
2. Seleziona il piano **Lite** e fai clic su **Create**. Utilizza i valori predefiniti e fai clic su **Confirm**.
3. Fai di nuovo clic su **Add Service** e scegli **Watson**. Fai clic su **Add** sul tile **Machine Learning** > scegli il piano **Lite** > e fai clic su **Create**.
4. Lascia i valori predefiniti e fai clic su **Confirm** per eseguire il provisioning di un servizio di Machine Learning.

## Crea un modello di machine learning

{:#build_model}

1. Fai clic su **Add to project** e seleziona **Watson Machine Learning model**. Nella finestra di dialogo, aggiungi **iris_model** come nome e una descrizione facoltativa.
2. Nella sezione **Machine Learning Service**, dovresti vedere il servizio di Machine Learning che hai associato nel passo precedente.
   ![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. Seleziona **Model builder** come tipo di modello e, nella sezione **Spark Service or Environment**, scegli il servizio Spark che hai creato in precedenza
4. Seleziona **Manual** per creare manualmente un modello. Fai clic su **Create**.

   Per il metodo automatico, ti affidi completamente alla preparazione automatica dei dati (ADP). Per il metodo manuale, oltre ad alcune funzioni gestite dal trasformatore ADP, puoi aggiungere e configurare i tuoi propri stimatori, che sono gli algoritmi utilizzati nell'analisi.
   {:tip}

5. Nella pagina successiva, seleziona `iris_initial.csv` come tua serie di dati e fai clic su **Next**.
6. Nella pagina **Select a technique**, le colonne Label e Feature sono pre-popolate in base alla serie di dati aggiunta. Seleziona **species (String)** come colonna dell'etichetta (**Label Col**) e **petal_length (Decimal)** e **petal_width (Decimal)** come colonne delle funzioni (**Feature columns**).
7. Scegli **Multiclass Classification** come tua tecnica suggerita.
   ![](images/solution22-build-machine-learning-model/model_technique.png)
8. Per **Validation Split**, configura la seguente impostazione:

   **Train:** 50%,
   **Test** 25%,
   **Holdout:** 25%

9. Fai clic su **Add Estimators**, seleziona **Decision Tree Classifier** e fai clic su **Add**.

   Puoi valutare più stimatori in una sola volta. Ad esempio, puoi aggiungere **Decision Tree Classifier** e **Random Forest Classifier** come stimatori per addestrare il tuo modello e scegliere la soluzione migliore in base all'output di valutazione.
   {:tip}

10. Fai clic su **Next** per addestrare il modello. Quando visualizzi lo stato come **Trained & Evaluated**, fai clic su **Save**.
   ![](images/solution22-build-machine-learning-model/trained_model.png)

11. Fai clic su **Overview** per controllare i dettagli del modello.

## Distribuisci il modello e prova l'API.

{:#deploy_model}

1. Nel modello creato, fai clic su **Deployments** > **Add Deployment**.
2. Scegli **Web Service**. Aggiungi un nome come `iris_deployment` e una descrizione facoltativa.
3. Fai clic su **Save**. Nella pagina di panoramica, fai clic sul nome del nuovo servizio web. Una volta che lo stato diventa **DEPLOY_SUCCESS**, puoi controllare l'endpoint di calcolo del punteggio, i frammenti di codice in vari linguaggi di programmazione e la specifica API sotto **Implementation**.
4. Fai clic su **View API Specification** per visualizzare e verificare gli endpoint API {{site.data.keyword.pm_short}}.
   ![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   Per iniziare a lavorare con l'API, devi generare un **token di accesso** utilizzando il **nome utente** e la **password** disponibili nella scheda **Service Credentials** dell'istanza del servizio {{site.data.keyword.pm_short}} nell'[elenco delle risorse {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/) . Segui le istruzioni riportate nella pagina della specifica API per generare un **token di accesso**.
   {:tip}
5. Per effettuare una previsione online, utilizza la chiamata API `POST /online`.
   * `instance_id` può essere trovato nella scheda **Service Credentials** del servizio {{site.data.keyword.pm_short}} nell'[elenco delle risorse {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/).
   * `deployment_id` e `published_model_id` sono disponibili nella sezione della panoramica (**Overview**) della tua distribuzione.
   *  Per `online_prediction_input`, utilizza il seguente JSON

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * Fai clic su **Try it out** per visualizzare l'output JSON.

6. Utilizzando gli endpoint API, puoi ora chiamare questo modello da qualsiasi applicazione.

## Verifica il tuo modello

{:#test_model}

1. Sotto **Test**, dovresti vedere i dati di input (Feature data) che vengono popolati automaticamente.
2. Fai clic su **Predict** e dovresti visualizzare il valore **Predicted value for species** in un grafico.
   ![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. Per l'input e l'output JSON, fai clic sulle icone accanto all'input e all'output attivi.
4. Puoi modificare i dati di input e continuare a verificare il tuo modello.

## Crea una connessione dati di feedback

{:#create_feedback_connection}

1. Per l'apprendimento continuo e la valutazione del modello, devi memorizzare i nuovi dati da qualche parte. Crea un [servizio {{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse) > piano **Entry** che funge da connessione dati di feedback.
2. Nella pagina **Manage** di {{site.data.keyword.dashdbshort}}, fai clic su **Open**. Nella navigazione in alto, seleziona **Load**.
3. Fai clic su **browse files** sotto **My computer** e carica `iris_initial.csv`. Fai clic su **Next**.
4. Seleziona **DASHXXXX**, ad esempio, DASH1234 come tuo **Schema** e fai clic su **New Table**. Denominalo `IRIS_FEEDBACK` e fai clic su **Next**.
5. I tipi di dati vengono rilevati automaticamente. Fai clic su **Next** e quindi su **Begin Load**.
   ![](images/solution22-build-machine-learning-model/define_table.png)
6. Viene creata una nuova destinazione **DASHXXXX.IRIS_FEEDBACK**.

   La utilizzerai nel passo successivo in cui dovrai riaddestrare il modello per ottenere prestazioni e precisione migliori.

## Riaddestra il tuo modello

{:#retrain_model}

1. Torna al tuo [elenco delle risorse {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources) e, nel servizio {{site.data.keyword.DSX_short}} che hai utilizzato, fai clic su **Projects** > iris_project >  **iris-model** (sotto assets) > Evaluation.
2. In **Performance Monitoring**, fai clic su **Configure Performance Monitoring**.
3. Nella pagina Configure Performance Monitoring,
   * Seleziona il servizio Spark. Il tipo di previsione dovrebbe venire popolato automaticamente.
   * Scegli **weightedPrecision** come metrica e imposta `0.98` come soglia facoltativa.
   * Fai clic su **Create new connection** per puntare al servizio IBM Db2 Warehouse on Cloud che hai creato nella sezione precedente.
   * Seleziona la connessione a Db2 Warehouse e, una volta che i dettagli di connessione vengono popolati, fai clic su **Create**.
     ![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * Fai clic su **Select feedback data reference** e punta alla tabella IRIS_FEEDBACK, quindi fai clic su **Select**.
     ![](images/solution22-build-machine-learning-model/select_source.png)
   * Nella casella **Record count required for re-evaluation**, immetti il numero minimo di nuovi record per attivare il riaddestramento. Utilizza **10** o lascia vuoto per utilizzare il valore predefinito di 1000.
   * Nella casella **Auto retrain**, seleziona una delle seguenti opzioni:
     - Per avviare il riaddestramento automatico ogni volta che le prestazioni del modello scendono al di sotto della soglia che hai impostato, seleziona **when model performance is below threshold**. Per questa esercitazione, sceglierai questa opzione poiché la nostra precisione è al di sotto della soglia (0,98).
     - Per proibire il riaddestramento automatico, seleziona **never**.
     - Per avviare il riaddestramento automatico indipendentemente dalle prestazioni, seleziona **always**.
   * Nella casella **Auto deploy**, seleziona una delle seguenti opzioni:
     - Per avviare la distribuzione automatica ogni volta che le prestazioni del modello sono migliori rispetto alla versione precedente, seleziona **when model performance is better than previous version**. Per questa esercitazione, sceglierai questa opzione perché il nostro obiettivo è migliorare continuamente le prestazioni del modello.
     - Per proibire la distribuzione automatica, seleziona **never**.
     - Per avviare la distribuzione automatica indipendentemente dalle prestazioni, seleziona **always**.
   * Fai clic su **Save**.
     ![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. Scarica il file [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv). Successivamente, fai clic su **Add feedback data**, seleziona il file csv scaricato e fai clic su **Open**.
5. Fai clic su **New evaluation** per iniziare.
     ![](images/solution22-build-machine-learning-model/retraining_model.png)
6. Una volta completata la valutazione, puoi consultare la sezione **Last Evalution Result** per controllare il valore migliorato **WeightedPrecision**.

## Rimuovi le risorse
{:removeresources}

1. Vai all'[elenco delle risorse {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/) > e scegli l'ubicazione, l'organizzazione e lo spazio in cui hai creato i servizi.
2. Elimina i rispettivi servizi {{site.data.keyword.DSX_short}}, {{site.data.keyword.sparks}}, {{site.data.keyword.pm_short}}, {{site.data.keyword.dashdbshort}} e {{site.data.keyword.cos_short}} che hai creato per questa esercitazione.

## Contenuto correlato
{:related}

- [Panoramica di Watson Studio](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [Rileva le anomalie utilizzando Machine Learning](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [Creazione automatica dei modelli](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [Machine learning e AI](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
