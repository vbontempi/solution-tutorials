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


# Applicazione web senza server e API
{: #serverless-api-webapp}

In questa esercitazione, creerai un'applicazione web senza server ospitando il contenuto del sito web statico nelle pagine GitHub e implementando il backend dell'applicazione utilizzando {{site.data.keyword.openwhisk}}.

Come piattaforma guidata dagli eventi, {{site.data.keyword.openwhisk_short}} supporta [diversi casi di utilizzo](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases). La creazione di applicazioni web e di API è uno di essi. Con le applicazioni web, gli eventi sono le interazioni tra i browser web (o i client REST) e la tua applicazione web, le richieste HTTP. Invece di eseguire il provisioning di una VM, di un contenitore o di un runtime Cloud Foundry per distribuire il tuo backend, puoi implementare la tua API di backend con una piattaforma senza server. Questa può essere una buona soluzione per evitare di pagare per il tempo di inattività e per poter ridimensionare la piattaforma in base alle esigenze. 

Qualsiasi azione (o funzione) in {{site.data.keyword.openwhisk_short}} può essere trasformata in un endpoint HTTP pronto per essere utilizzato dai client web. Quando sono abilitate per il web, queste azioni sono denominate *azioni web*. Una volta che disponi di azioni web, puoi assemblarle in un'API completa con il gateway API. Il gateway API è un componente di {{site.data.keyword.openwhisk_short}} per esporre le API. Viene fornito con la sicurezza, il supporto OAuth, la limitazione della frequenza e il supporto del dominio personalizzato. 

## Obiettivi

* Distribuire un backend senza server e un database
* Esporre un'API REST
* Ospitare un sito web statico
* Facoltativo: utilizzare un dominio personalizzato per l'API REST

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura
{: #architecture}

L'applicazione mostrata in questa esercitazione è un semplice sito web per guestbook in cui gli utenti possono pubblicare i messaggi. 

<p style="text-align: center;">

   ![Architettura](./images/solution8/Architecture.png)
</p>

1. L'utente accede all'applicazione ospitata nelle pagine GitHub.
2. L'applicazione web richiama un'API di backend.
3. L'API di backend è definita nel gateway API. 
4. Il gateway API inoltra la richiesta a [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk).
5. Le azioni {{site.data.keyword.openwhisk_short}} utilizzano [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) per archiviare e richiamare le voci del guestbook. 

## Prima di iniziare
{: #prereqs}

Questa guida utilizza le pagine GitHub per ospitare il sito web statico. Assicurati di disporre di un account GitHub pubblico.

## Crea il database Guestbook

Iniziamo creando un {{site.data.keyword.cloudant_short_notm}}. {{site.data.keyword.cloudant_short_notm}} è un livello di dati completamente gestito progettato per le applicazioni web e mobili moderne che si avvalgono di uno schema JSON flessibile. {{site.data.keyword.cloudant_short_notm}} si basa ed è compatibile con Apache CouchDB ed è accessibile tramite un'API HTTPS sicura, che viene ridimensionata man mano che la tua applicazione cresce. 

1. Nel catalogo, seleziona **Cloudant**.
2. Imposta il nome servizio su **guestbook-db**, seleziona **Use both legacy credentials and IAM** come metodi di autenticazione e fai clic su **Create**.
3. Torna nell'elenco delle risorse, fai clic sulla voce ***guestbook-db** nella colonna Name. Nota: potresti dover attendere fino a quando non viene eseguito il provisioning del servizio.  
4. Nella schermata dei dettagli del servizio, fai clic su ***Launch Cloudant Dashboard*** che verrà aperto in un'altra scheda del browser. Nota: potresti dover accedere alla tua istanza Cloudant.  
5. Fai clic su ***Create Database*** e crea un database denominato ***guestbook***.

  ![](images/solution8/Create_Database.png)

6. Torna alla scheda dei dettagli del servizio, in **Service Credentials**
   1. Crea una nuova credenziale (**New credential**), accetta i valori predefiniti e fai clic su **Add**.
   2. Fai clic su **View credentials** in Actions. Avremo bisogno di queste credenziali in un secondo momento per consentire alle azioni Cloud Functions di leggere/scrivere nel tuo servizio Cloudant.

## Crea le azioni senza server

In questa sezione, creerai le azioni senza server (comunemente definite come Funzioni). {{site.data.keyword.openwhisk}} (basato su Apache OpenWhisk) è una piattaforma FaaS (Function-as-a-Service) che esegue funzioni in risposta agli eventi in entrata e non comporta costi quando non è in uso. 

![](images/solution8/Functions.png)

### Sequenza di azioni per salvare la voce del guestbook

Creerai una **sequenza** che è una catena di azioni in cui l'output di un'azione funge da input per l'azione successiva e così via. La prima sequenza che creerai viene utilizzata per conservare un messaggio guest. Forniti un nome, un ID email e un commento, la sequenza: 
   * Crea un documento da conservare. 
   * Archivia il documento nel database {{site.data.keyword.cloudant_short_notm}}.

Inizia creando la prima azione:

1. Passa a **Functions** https://{DomainName}/openwhisk.
2. Nel riquadro di sinistra, fai clic su **Actions** e poi su **Create**.
3. **Crea un'azione** con il nome `prepare-entry-for-save` e seleziona **Node.js** come Runtime (Nota: seleziona la versione più recente).
4. Sostituisci il codice esistente con il frammento di codice riportato di seguito: 
   ```js
   /**
    * Prepare the guestbook entry to be persisted
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. **Salva**

Quindi, aggiungi l'azione a una sequenza:

1. Fai clic su **Enclosing Sequences** e poi su **Add To Sequence**.
1. Per il nome della sequenza, immetti `save-guestbook-entry-sequence` e poi fai clic su **Create and Add**.

Infine, aggiungi una seconda azione alla sequenza:

1. Fai clic su **save-guestbook-entry-sequence** e poi fai clic su **Add**.
1. Seleziona **Use Public**, **Cloudant** e quindi scegli **create-document** in **Actions**
1. Crea un nuovo bind (**New Binding**) 
1. Per il nome, immetti `binding-for-guestbook`
1. Per l'istanza Cloudant, seleziona `Input your own credentials` e riempi i seguenti campi con le informazioni sulle credenziali acquisite per il tuo servizio cloudant: Username, Password, Host e Database = `guestbook` e fai clic su **Add** e quindi su **Save**.
   {: tip}
1. Per verificarlo, fai clic su **Change Input** e immetti il JSON riportato di seguito
    ```json
    {
      "name": "John Smith",
      "email": "john@smith.com",
      "comment": "this is my comment"
    }
    ```
    {: codeblock}
1. **Applica** e poi **richiama**.
    ![](images/solution8/Save_Entry_Invoke.png)

### Sequenza di azioni per richiamare le voci

La seconda sequenza viene utilizzata per richiamare le voci esistenti del guestbook. Questa sequenza:
   * Elenca tutti i documenti provenienti dal database.
   * Formatta i documenti e li restituisce. 

1. In **Functions**, fai clic su **Actions** e quindi su **Create** per creare una nuova azione Node.js e denominala `set-read-input`.
2. Sostituisci il codice esistente con il frammento di codice riportato di seguito. Questa azione passa i parametri appropriati all'azione successiva. 
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. **Salva**

Aggiungi l'azione a una sequenza:

1. Fai clic su **Enclosing Sequences**, **Add to Sequence** e **Create New**
1. Immetti `read-guestbook-entries-sequence` per **Action Name** e fai clic su **Create and Add**.

Completa la sequenza:

1. Fai clic sulla sequenza **read-guestbook-entries-sequence** e poi fai clic su **Add** per creare e aggiungere la seconda azione per ottenere i documenti da Cloudant.
1. In **Use Public**, scegli **{{site.data.keyword.cloudant_short_notm}}** e poi **list-documents**
1. In **My Bindings**, scegli **binding-for-guestbook** e **Add** per creare e aggiungere questa azione pubblica alla tua sequenza.
1. Fai di nuovo clic su **Add** per creare e aggiungere la terza azione che formatterà i documenti provenienti da {{site.data.keyword.cloudant_short_notm}}.
1. In **Create New**, immetti `format-entries` per il nome e poi fai clic su **Create and Add**.
1. Fai clic su **format-entries** e sostituisci il codice con quello riportato di seguito: 
   ```js
   const md5 = require('spark-md5');

   function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. **Salva**
1. Scegli la sequenza facendo clic su **Actions** e poi su **read-guestbook-entries-sequence**.
1. Fai clic su **Save** e poi su **Invoke**. L'output dovrebbe apparire simile al seguente:
   ![](images/solution8/Read_Entries_Invoke.png)

## Crea un'API

![](images/solution8/Cloud_Functions_API.png)

1. Vai a Actions, all'indirizzo https://{DomainName}/openwhisk/actions.
2. Seleziona la sequenza **read-guestbook-entries-sequence**. Accanto al nome, fai clic su **Web Action**, seleziona **Enable Web Action** e **Save**.
3. Fai lo stesso per la sequenza **save-guestbook-entry-sequence**.
4. Vai a APIs, all'indirizzo https://{DomainName}/openwhisk/apimanagement, e **crea un'API {{site.data.keyword.openwhisk_short}}**
5. Imposta il nome su `guestbook` e il percorso di base su `/guestbook`
6. Fai clic su **Create operation** e crea un'operazione per richiamare le voci del guestbook:
   1. Imposta **path** su `/entries`
   2. Imposta **verb** su `GET*`
   3. Seleziona l'azione **read-guestbook-entries-sequence**
7. Fai clic su **Create operation** e crea un'operazione per archiviare in modo persistente una voce del guestbook:
   1. Imposta **path** su `/entries`
   2. Imposta **verb** su `PUT`
   3. Seleziona l'azione **save-guestbook-entry-sequence**
8. Salva ed esponi l'API.

## Distribuisci l'applicazione web

1. Biforca il repository dell'interfaccia utente Guestbook all'indirizzo https://github.com/IBM-Cloud/serverless-guestbook nel tuo GitHub pubblico.
2. Modifica il file **docs/guestbook.js** e sostituisci il valore di **apiUrl** con l'instradamento fornito dal gateway API.
3. Esegui il commit del file modificato nel tuo repository biforcato.
4. Nella pagina delle impostazioni del tuo repository, scorri fino a **GitHub Pages**, modifica l'origine nella **cartella /docs del ramo master** e salva. 
5. Accedi alla pagina pubblica per il tuo repository.
6. Dovresti vedere la voce "test" del guestbook che hai creato in precedenza.
7. Aggiungi le nuove voci.

![](images/solution8/Guestbook.png)

## Facoltativo: utilizza il tuo dominio per l'API

La creazione di un'API gestita ti fornisce un endpoint predefinito come `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`. In questa sezione, configurerai questo endpoint per poter gestire le richieste provenienti dal tuo dominio secondario personalizzato. 

### Ottieni un certificato per il dominio personalizzato

L'esposizione delle azioni {{site.data.keyword.openwhisk_short}} tramite un dominio personalizzato richiederà una connessione HTTPS sicura. Devi ottenere un certificato SSL per il dominio e il dominio secondario che intendi utilizzare con il backend senza server. Presupponendo un dominio come *mydomain.com*, le azioni potrebbero essere ospitate in *guestbook-api.mydomain.com*. Il certificato dovrà essere emesso per *guestbook-api.mydomain.com* (or **.mydomain.com*).

Puoi ottenere certificati SSL gratuiti da [Let's Encrypt](https://letsencrypt.org/). Durante il processo, potresti dover configurare un record DNS di tipo TXT nella tua interfaccia DNS per dimostrare che sei il proprietario del dominio.
{:tip}

Una volta ottenuto il certificato SSL e la chiave privata per il tuo dominio, assicurati di convertirli nel formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Per convertire un certificato nel formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. Per convertire una chiave privata nel formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### Importa il certificato in un repository centrale

1. Crea un'istanza [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) in un'ubicazione supportata. 
1. Nel dashboard del servizio, utilizza **Import Certificate**:
   * Imposta **Name** sul dominio secondario e sul dominio personalizzato, ad esempio `guestbook-api.mydomain.com`.
   * Individua, sfogliando, il file di certificato (**Certificate file**) in formato PEM.
   * Individua, sfogliando, il file di chiave privata (**Private key file**) in formato PEM.
   * Seleziona **Import**.

### Configura il dominio personalizzato per l'API gestita

1. Vai a [APIs / Custom domains](https://{DomainName}/apis/domains).
1. Nel selettore della regione, seleziona l'ubicazione in cui hai distribuito le azioni. 
1. Individua il dominio personalizzato collegato all'organizzazione e allo spazio in cui hai creato le azioni e l'API gestita. Fai clic su **Change Settings** nel menu delle azioni. 
1. Prendi nota del valore di **Default domain / alias**.
1. Seleziona **Apply custom domain**
   1. Imposta **Domain name** sul dominio che utilizzerai, ad esempio `guestbook-api.mydomain.com`.
   1. Seleziona l'istanza {{site.data.keyword.cloudcerts_short}} che contiene il certificato.
   1. Seleziona il certificato per il dominio. 
1. Vai al tuo provider DNS e crea un nuovo **record TXT DNS** che associa il tuo dominio al dominio predefinito/alias (Default domain / alias) dell'API. Il record TXT DNS può essere rimosso una volta che sono state applicate le impostazioni. 
1. Salva le impostazioni del dominio personalizzato. Internet Services controllerà l'esistenza del record TXT DNS. 
1. Infine, ritorna alle impostazioni del tuo provider DNS e crea un record CNAME che punti al tuo dominio personalizzato (ad esempio, guestbook-api.mydomain.com) nel dominio predefinito/alias (Default domain / alias). Ciò farà sì che il traffico sul tuo dominio personalizzato venga instradato alla tua API di backend.

Una volta propagate le modifiche DNS, potrai accedere alla tua api guestbook all'indirizzo https://guestbook-api.mydomain.com/guestbook.

1. Modifica il file **docs/guestbook.js** e aggiorna il valore di **apiUrl** con https://guestbook-api.mydomain.com/guestbook
1. Esegui il commit del file modificato. 
1. Ora la tua applicazione accede all'API tramite il tuo dominio personalizzato. 

## Rimuovi le risorse

* Elimina il servizio {{site.data.keyword.cloudant_short_notm}}
* Elimina l'API da {{site.data.keyword.openwhisk_short}}
* Elimina le azioni da {{site.data.keyword.openwhisk_short}}

## Contenuto correlato
* [Altre guide ed esempi su funzioni senza server](https://developer.ibm.com/code/journey/category/serverless/)
* [Introduzione a {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [Casi di utilizzo comuni di {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [Creazione di API REST dalle azioni](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
