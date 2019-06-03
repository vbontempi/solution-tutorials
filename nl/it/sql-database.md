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

# Database SQL per dati Cloud
{: #sql-database}

Questa esercitazione mostra come eseguire il provisioning di un servizio di database SQL (relazionale), creare una tabella e caricare una serie di dati di grandi dimensioni (informazioni sulla città) nel database. Distribuirai quindi un'applicazione web sulle "città mondiali" per utilizzare tali dati e per mostrare in che modo accedere al database cloud. L'applicazione viene scritta in Python utilizzando il [framework Flask](http://flask.pocoo.org/).

![](images/solution5/Architecture.png)

## Obiettivi

* Eseguire il provisioning di un database SQL
* Creare lo schema del database (tabella)
* Caricare i dati
* Connettere l'applicazione e il servizio database (condividere le credenziali)
* Monitoraggio, sicurezza, backup e ripristino

## Prodotti

Questa esercitazione utilizza i seguenti prodotti:
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## Prima di iniziare
{: #prereqs}

Vai a [GeoNames](http://www.geonames.org/) e scarica ed estrai il file [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip). Contiene le informazioni sulle città che hanno una popolazione superiore ai 1000 abitanti. Lo utilizzerai come serie di dati. 

## Esegui il provisioning del database SQL
Inizia creando un'istanza del servizio **[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)**.

![](images/solution5/Catalog.png)

1. Visita il [dashboard di {{site.data.keyword.Bluemix_short}}](https://{DomainName}). Fai clic su **Catalog** nella barra di navigazione in alto. 
2. Fai clic su **Data & Analytics** in Platform nel riquadro di sinistra e seleziona **{{site.data.keyword.dashdbshort_notm}}**.
3. Seleziona il piano **Entry** e modifica il nome servizio suggerito in "sqldatabase" (utilizzerai tale nome in seguito). Seleziona un'ubicazione per la distribuzione del database e assicurati che siano selezionati l'organizzazione e lo spazio corretti. 
4. Fai clic su **Create**. Dopo poco, dovresti ricevere una notifica di esito positivo. 
5. In **Resource List**, fai clic sulla voce relativa al tuo servizio {{site.data.keyword.dashdbshort_notm}} appena creato.
6. Fai clic su **Open** per avviare la console del database. Se è la prima volta che utilizzi la console, ti viene offerta la possibilità di effettuare un tour. 

## Crea una tabella
Hai bisogno di una tabella per conservare i dati di esempio. Creala utilizzando la console.

1. Nella console per {{site.data.keyword.dashdbshort_notm}}, fai clic su **Explore** nella barra di navigazione. Questa operazione ti porterà a un elenco degli schemi esistenti nel database. 
2. Individua e fai clic sullo schema che inizia con "DASH".
3. Fai clic su **"+ New Table"** per visualizzare un formato per il nome tabella e le sue colonne. 
4. Utilizza "cities" come nome tabella. Copia le definizioni di colonna dal file [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) e incollale nella casella per le colonne e i tipi di dati. 
5. Fai clic su **Create** per definire la nuova tabella.    
   ![](images/solution5/TableCitiesCreated.png)

## Carica i dati
Ora che la tabella "cities" è stata creata, caricherai i dati al suo interno. Tale operazione può essere eseguita in diversi modi, ad esempio dalla tua macchina locale o dal COS (cloud object storage) con l'interfaccia Swift o Amazon S3, utilizzando il servizio di migrazione [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift). Per questa esercitazione, caricherai i dati dalla tua macchina. Durante questo processo, adatterai la struttura della tabella e il formato dei dati in modo che soddisfino perfettamente il contenuto del file. 

1. Nella navigazione in alto, fai clic su **Load**. Poi, in **File selection**, fai clic su **browse files** per individuare e selezionare il file "cities1000.txt" che hai scaricato nella prima sezione di questa guida. 
2. Fai clic su **Next** per accedere alla panoramica dello schema. Scegli di nuovo lo schema che inizia con "DASH", poi la tabella "CITIES". Fai di nuovo clic su **Next**.   

   Poiché la tabella è vuota, non fa differenza se esegui un'aggiunta ai dati esistenti oppure se li sovrascrivi.
   {:tip }
3. Ora personalizza in che modo i dati provenienti del file "cities1000.txt" vengono interpretati durante il processo di caricamento. Innanzitutto, disabilita "Header in first row" perché il file contiene solo dati. Poi, immetti "0x09" come separatore. Ciò significa che i valori all'interno del file vengono delimitati da una tabulazione. Infine, seleziona "YYYY-MM-DD" come formato data. Ora, dovresti aver ottenuto un risultato simile a quello di questa schermata.     
  ![](images/solution5/LoadTabSeparator.png)
4. Fai clic su **Next** e ti verrà offerto di esaminare le impostazioni di caricamento. Accetta e fai clic su **Begin Load** per iniziare a caricare i dati nella tabella "CITIES". Viene visualizzato l'avanzamento. Una volta caricati i dati, dovrebbero trascorrere solo pochi secondi fino al completamento del caricamento e alla presentazione di alcune statistiche.    
   ![](images/solution5/LoadProgressSteps.png)

## Verifica i dati caricati utilizzando SQL
I dati sono stati caricati nel database relazionale. Non si sono verificati errori, ma devi eseguire comunque alcuni rapidi test. Utilizza l'editor SQL integrato per immettere ed eseguire alcune istruzioni SQL.

1. Nella navigazione in alto, fai clic su **Run SQL**.
   Al posto dell'editor SQL integrato, puoi utilizzare gli strumenti basati su cloud e quelli SQL tradizionali presenti sul tuo desktop o sulla tua macchina server con {{site.data.keyword.dashdbshort_notm}}. Puoi trovare le informazioni di connessione nel menu delle impostazioni. Alcuni strumenti vengono anche offerti per il download nella sezione "Downloads" nel menu presente dietro l'icona "book" (rappresenta la documentazione e il supporto).
    {:tip }
2. Nell'editor SQL, immetti o copia la seguente query:   
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   quindi premi il pulsante **Run All**. Nella sezione dei risultati, dovrebbe essere visualizzato lo stesso numero di righe riportato dal processo di caricamento.    
3. Nell'editor SQL, immetti la seguente istruzione su una nuova riga: 
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. Nell'editor, seleziona il testo dell'istruzione di cui sopra. Fai clic sul pulsante **Run Selected**. Ora, dovrebbe essere eseguita solo questa istruzione, restituendo alcune statistiche del paese nella sezione dei risultati. 

## Distribuisci il codice dell'applicazione
Il [codice pronto per l'esecuzione per l'applicazione di registrazione si trova in questo repository GitHub](https://github.com/IBM-Cloud/cloud-sql-database). Clona o scarica il repository, quindi eseguine il push in IBM Cloud.

1. Clona il repository Github:
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. Esegui il push dell'applicazione in IBM Cloud. Devi essere collegato all'ubicazione, all'organizzazione e allo spazio in cui è stato eseguito il provisioning del database. Copia e incolla questi comandi una riga alla volta. 
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. Una volta terminato il processo di push, dovresti essere in grado di accedere all'applicazione. Non è necessaria alcuna ulteriore configurazione. Il file `manifest.yml` indica a IBM Cloud di eseguire il bind dell'applicazione e del servizio database denominato "sqldatabase".

## Sicurezza, backup e ripristino, monitoraggio
{{site.data.keyword.dashdbshort_notm}} è un servizio gestito. IBM si occupa della sicurezza dell'ambiente dei backup giornalieri e del monitoraggio del sistema. Nel piano Entry, l'ambiente database è una configurazione a più tenant con un'amministrazione ridotta e opzioni configurate per gli utenti. Tuttavia, se stai utilizzando uno dei piani Enterprise, sono disponibili [diverse opzioni per gestire gli utenti, per configurare sicurezza aggiuntiva del database](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html) e per [monitorare il database](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html).   

Oltre alle opzioni di amministrazione tradizionali, il [servizio {{site.data.keyword.dashdbshort_notm}} offre un'API REST per il monitoraggio, la gestione utente, i programmi di utilità, il caricamento, l'accesso all'archiviazione e altro](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html). Puoi accedere all'interfaccia Swagger eseguibile di tale API nel menu dietro l'icona "book" in "Rest APIs". Alcuni strumenti che possono essere utilizzati per il monitoraggio e altre funzioni, ad esempio IBM Data Server Manager, possono essere scaricati anche nella sezione "Downloads" presente in quello stesso menu. 

## Verifica l'applicazione
L'applicazione per visualizzare le informazioni sulla città basate sulla serie di dati caricata è ridotta al minimo. Offre un formato di ricerca per specificare un nome città e alcune città preconfigurate. Vengono tradotti in `/search?name=cityname` (formato di ricerca) o in `/city/cityname` (vengono specificate direttamente le città). Entrambe le richieste vengono servite dalle stesse righe di codice in background. cityname viene passato come valore a un'istruzione SQL preparata utilizzando un contrassegno del parametro per motivi di sicurezza. Le righe vengono recuperate dal database e passate a un template HTML per il rendering.

## Ripulitura
Per ripulire le risorse utilizzate dall'esercitazione, segui questi passi:
1. Visita l'[elenco risorse {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources). Individua la tua applicazione. 
2. Fai clic sull'icona menu per l'applicazione e scegli **Delete App**. Nella finestra di dialogo, utilizza un segno di spunta per indicare che vuoi eliminare il servizio {{site.data.keyword.dashdbshort_notm}} correlato.
3. Fai clic sul pulsante **Delete**. L'applicazione e il servizio database vengono rimossi e vieni riportato all'elenco delle risorse. 

## Espandi l'esercitazione
Vuoi estendere questa applicazione? Ecco alcune idee:
1. Offri una ricerca con caratteri jolly su nomi alternativi. 
2. Ricerca le città di uno specifico paese e solo all'interno di determinati valori di popolazione. 
3. Modifica il layout della pagina sostituendo gli stili CSS ed estendendo i template.
4. Consenti la creazione basata sul formato delle nuove informazioni sulla città oppure consenti gli aggiornamenti dei dati esistenti, ad esempio la popolazione. 

## Contenuto correlato
* Documentazione: [IBM Knowledge Center for {{site.data.keyword.dashdbshort_notm}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* Pagina [Frequently asked questions about {{site.data.keyword.Db2_on_Cloud_long_notm}} and {{site.data.keyword.dashdblong_notm}}](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html) che risponde alle domande relative al servizio gestito, al backup dei dati, alla crittografia e alla sicurezza dei dati e molto altro. 
* [Free Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) per sviluppatori
* Documentazione: [API Description of ibm_db Python driver](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
