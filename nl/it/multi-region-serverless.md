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

# Distribuzione di applicazioni senza server in più regioni
{: #multi-region-serverless}

Questa esercitazione mostra come configurare IBM Cloud Internet Services e {{site.data.keyword.openwhisk_short}} per distribuire le applicazioni senza server in più regioni.

Le piattaforme di calcolo senza server offrono agli sviluppatori un modo rapido per creare API senza server. {{site.data.keyword.openwhisk}} supporta la generazione automatica dell'API REST per le azioni, trasformando le azioni in endpoint HTTP, e la capacità di abilitare l'autenticazione API protetta. Questa funzionalità è utile non solo per esporre le API ai consumatori esterni ma anche per creare applicazioni di microservizi.

{{site.data.keyword.openwhisk_short}} è disponibile in più ubicazioni {{site.data.keyword.cloud_notm}}. Per aumentare la resilienza e ridurre la latenza di rete, le applicazioni possono distribuire il loro backend in più ubicazioni. Quindi, con IBM CIS (Cloud Internet Services), gli sviluppatori possono esporre un singolo punto di ingresso incaricato di distribuire il traffico al back-end integro più vicino. 

## Obiettivi
{: #objectives}

* Distribuire le azioni {{site.data.keyword.openwhisk_short}}.
* Esporre le azioni tramite {{site.data.keyword.APIM}} con un dominio personalizzato.
* Distribuire il traffico su più ubicazioni con Cloud Internet Services.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

Questa esercitazione prende in considerazione un'applicazione web pubblica con un back-end implementato con {{site.data.keyword.openwhisk_short}}. Per ridurre la latenza di rete ed evitare interruzioni, l'applicazione è distribuita in più ubicazioni. In questa esercitazione sono configurate due ubicazioni.

<p style="text-align: center;">

  ![Architettura](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. Gli utenti accedono all'applicazione. La richiesta passa attraverso Internet Services.
2. Internet Services reindirizza gli utenti al backend API integro più vicino.
3. {{site.data.keyword.cloudcerts_short}} fornisce all'API il suo certificato SSL. Il traffico viene crittografato end-to-end.
4. L'API viene implementata con {{site.data.keyword.openwhisk_short}}.

## Prima di iniziare
{: #prereqs}

1. CIS (Cloud Internet Services) richiede che tu sia il proprietario di un dominio personalizzato in modo che tu possa configurare il DNS affinché questo dominio punti al server dei nomi CIS (Cloud Internet Services). Se non sei il proprietario di un dominio, puoi acquistarne uno da un registrar come ad esempio [godaddy.com](http://godaddy.com).
1. Installa tutti gli strumenti CLI necessari [seguendo questi passi](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).

## Configura un dominio personalizzato

Il primo passo consiste nel creare un'istanza di IBM CIS (Cloud Internet Services) e puntare il tuo dominio personalizzato ai server dei nomi CIS.

1. Passa a [Internet Services](https://{DomainName}/catalog/services/internet-services) nel catalogo {{site.data.keyword.Bluemix_notm}}.
1. Imposta il nome servizio e fai clic su **Create** per creare un'istanza del servizio. Per questa esercitazione, puoi utilizzare i piani dei prezzi. 
1. Quando viene eseguito il provisioning dell'istanza del servizio, imposta il tuo nome dominio facendo clic su **Let's get started** e facendo clic su **Add domain**.
1. Fai clic su **Next step**. Quando vengono assegnati i server dei nomi, configura il tuo registrar o il provider del nome del dominio in modo che utilizzi i server dei nomi elencati.
1. Una volta configurato il tuo registrar o il provider DNS, potresti dover attendere fino a 24 ore affinché le modifiche diventino effettive.

   Quando lo stato del dominio nella pagina Overview cambia da *Pending* a *Active*, puoi utilizzare il comando `dig <your_domain_name> ns` per verificare che i nuovi server dei nomi siano stati applicati.
   {:tip}

### Ottieni un certificato per il dominio personalizzato

L'esposizione delle azioni {{site.data.keyword.openwhisk_short}} tramite un dominio personalizzato richiederà una connessione HTTPS sicura. Devi ottenere un certificato SSL per il dominio e il dominio secondario che intendi utilizzare con il backend senza server. Presupponendo un dominio come *mydomain.com*, le azioni potrebbero essere ospitate in *api.mydomain.com*. Il certificato dovrà essere emesso per *api.mydomain.com*.

Puoi ottenere certificati SSL gratuiti da [Let's Encrypt](https://letsencrypt.org/). Durante il processo, potresti dover configurare un record DNS di tipo TXT nell'interfaccia DNS di CIS (Cloud Internet Services) per dimostrare che sei il proprietario del dominio.
{:tip}

Una volta ottenuto il certificato SSL e la chiave privata per il tuo dominio, assicurati di convertirli nel formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Per convertire un certificato nel formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. Per convertire una chiave privata nel formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Importa il certificato in un repository centrale

1. Crea un'istanza [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) in un'ubicazione supportata.
1. Nel dashboard del servizio, utilizza **Import Certificate**:
   * Imposta **Name** sul sottodominio e sul dominio personalizzati, come *api.mydomain.com*.
   * Individua, sfogliando, il file di certificato (**Certificate file**) in formato PEM.
   * Individua, sfogliando, il file di chiave privata (**Private key file**) in formato PEM.
   * Seleziona **Import**.

## Distribuisci le azioni in più ubicazioni

In questa sezione, creerai le azioni, le esporrai come un'API e assocerai il dominio personalizzato all'API con un certificato SSL archiviato in {{site.data.keyword.cloudcerts_short}}.

<p style="text-align: center;">

  ![Architettura API](images/solution44-multi-region-serverless/api-architecture.png)
</p>

L'azione **doWork** implementa una delle tue operazioni API. L'azione **healthz** verrà utilizzata successivamente sul controllo se la tua API è integra. Potrebbe essere un no-op che semplicemente restituisce *OK* oppure potrebbe eseguire un controllo più complesso come l'esecuzione di un ping dei database o di altri servizi critici richiesti dalla tua API.

Le seguenti tre sezioni dovranno essere ripetute per ogni ubicazione dove desideri ospitare il back-end dell'applicazione. Per questa esercitazione, puoi scegliere *Dallas (us-south)* e *London (eu-gb)* come destinazioni.

### Definisci le azioni

1. Vai a [{{site.data.keyword.openwhisk_short}} / Actions](https://{DomainName}/openwhisk/actions).
1. Passa all'ubicazione di destinazione e seleziona un'organizzazione e uno spazio dove distribuire le azioni.
1. Crea un'azione
   1. Imposta **Name** su **doWork**.
   1. Imposta **Enclosing Package** su **default**.
   1. Imposta **Runtime** sulla versione più recente di **Node.js**.
   1. Seleziona **Create**.
1. Modifica il codice azione in:
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. **Salva**
1. Crea un'altra azione da utilizzare come controllo dell'integrità per la nostra API:
   1. Imposta **Name** su **healthz**.
   1. Imposta **Enclosing Package** su **default**.
   1. Imposta **Runtime** sul **Node.js** più recente.
   1. Seleziona **Create**.
1. Modifica il codice azione in:
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. **Salva**

### Esponi le azioni con un'API gestita

Il passo successivo implica la creazione di un'API gestita per esporre le tue azioni.

1. Vai a [{{site.data.keyword.openwhisk_short}} / API](https://{DomainName}/openwhisk/apimanagement).
1. Crea una nuova API {{site.data.keyword.openwhisk_short}} gestita:
   1. Imposta **API name** su **App API**.
   1. Imposta **Base path** su **/api**.
1. Crea un'operazione:
   1. Imposta **Path** su **/do**.
   1. Imposta **Verb** su **GET**.
   1. Imposta **Package** su **default**.
   1. Imposta **Action** su **doWork**.
   1. Seleziona **Create**
1. Crea un'altra operazione:
   1. Imposta **Path** su **/healthz**.
   1. Imposta **Verb** su **GET**.
   1. Imposta **Package** su **default**.
   1. Imposta **Action** su **healthz**.
   1. Seleziona **Create**
1. Salva (**Save**) l'API

### Configura il dominio personalizzato per l'API gestita

La creazione di un'API gestita ti fornisce un endpoint predefinito come `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`. In questa sezione, configurerai questo endpoint per poter gestire le richieste provenienti dal tuo sottodominio personalizzato, il dominio che verrà successivamente configurato in IBM Cloud Internet Services.

1. Vai a [APIs / Custom domains](https://{DomainName}/apis/domains).
1. Nel selettore **Region**, seleziona l'ubicazione di destinazione.
1. Individua il dominio personalizzato collegato all'organizzazione e allo spazio in cui hai creato le azioni e l'API gestita. Fai clic su **Change Settings** nel menu delle azioni.
1. Prendi nota del valore di **Default domain / alias**.
1. Seleziona **Apply custom domain**
   1. Imposta **Domain name** sul dominio che utilizzerai con il GLB (Global Load Balancer) CIS, come ad esempio *api.mydomain.com*.
   1. Seleziona l'istanza {{site.data.keyword.cloudcerts_short}} che contiene il certificato.
   1. Seleziona il certificato per il dominio. 
1. Vai al dashboard della tua istanza di **Cloud Internet Services**, in **Reliability / DNS** e crea un nuovo record TXT DNS (**DNS TXT record**):
   1. Imposta **Name** sul tuo sottodominio personalizzato, come ad esempio **api**.
   1. Imposta **Content** su **Default domain / alias**
   1. Salva il record
1. Salva le impostazioni del dominio personalizzato. Internet Services controllerà l'esistenza del record TXT DNS.

   Se il record TXT non viene trovato, potresti doverne attendere la propagazione e riprovare a salvare le impostazioni. Il record TXT DNS può essere rimosso una volta che sono state applicate le impostazioni.
   {: tip}

Ripeti le sezioni precedenti per configurare ulteriori ubicazioni.

## Distribuisci il traffico tra le ubicazioni

**A questo punto, hai configurato le azioni in più ubicazioni** ma non c'è nessun singolo punto di ingresso per raggiungerle. In questa sezione, configurerai un GLB (global load balancer) per distribuire il traffico tra le ubicazioni.

<p style="text-align: center;">

  ![Architettura del GLB (global load balancer)](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### Crea un controllo dell'integrità

Internet Services richiamerà regolarmente questo endpoint per controllare l'integrità del back-end.

1. Vai al dashboard della tua istanza di IBM Cloud Internet Services.
1. In **Reliability / Global Load Balancers**, crea un controllo dell'integrità:
   1. Imposta **Monitor type** su **HTTPS**.
   1. Imposta **Path** su **/api/healthz**.
   1. **Esegui il provisioning della risorsa**.

### Crea i pool di origine

Creando un singolo pool per ogni ubicazione, puoi successivamente configurare gli instradamenti geografici nel tuo GLB (global load balancer) per reindirizzare gli utenti all'ubicazione più vicina. Un'altra opzione sarebbe quella di creare un singolo pool con tutte le ubicazioni e fare in modo che il programma di bilanciamento del carico scorra le origini nel pool.

Per ogni ubicazione:
1. Crea un pool di origine.
1. Imposta **Name** su **app-&lt;location&gt;** come ad esempio _app-Dallas_.
1. Seleziona il controllo dell'integrità creato prima.
1. Imposta **Health Check Region** su una regione vicina all'ubicazione dove è stata eseguita la distribuzione di {{site.data.keyword.openwhisk_short}}.
1. Imposta **Origin Name** su **app-&lt;location&gt;**.
1. Imposta **Origin Address** sul dominio/alias predefinito per l'API gestita (come ad esempio _5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_).
1. **Esegui il provisioning della risorsa**.

### Crea un GLB (global load balancer)

1. Crea un programma di bilanciamento del carico.
1. Imposta **Balancer hostname** su **api.mydomain.com**.
1. Aggiungi i pool di origine regionali.
1. **Esegui il provisioning della risorsa**.

Poco dopo, vai a `https://api.mydomain.com/api/do?name=John&place=Earth`. Dovrebbe rispondere con la funzione in esecuzione nel primo pool integro.

### Verifica il failover

Per verificare il failover, un controllo dell'integrità del pool deve avere esito negativo in modo che il GLB esegua il reindirizzamento al successivo pool integro. Per simulare un malfunzionamento, puoi modificare la funzione di controllo dell'integrità per fare in modo che abbia esito negativo.

1. Vai a [{{site.data.keyword.openwhisk_short}} / Actions](https://{DomainName}/openwhisk/actions).
1. Seleziona la prima ubicazione configurata nel GLB.
1. Modifica la funzione `healthz` e modificane l'implementazione come `throw new Error()`.
1. Salva.
1. Attendi l'esecuzione del controllo dell'integrità da questo pool di origine.
1. Ottieni nuovamente `https://api.mydomain.com/api/do?name=John&place=Earth`; dovrebbe ora eseguire il reindirizzamento a un'altra origine integra.
1. Annulla le modifiche al codice per ritornare a un'origine integra.

## Rimuovi le risorse
{: #removeresources}

### Rimuovi le risorse CIS

1. Rimuovi il GLB.
1. Rimuovi i pool di origine.
1. Rimuovi i controlli dell'integrità.

### Rimuovi le azioni

1. Rimuovi le [API](https://{DomainName}/openwhisk/apimanagement)
1. Rimuovi le [azioni](https://{DomainName}/openwhisk/actions)

## Contenuto correlato
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Cluster Kubernetes multi-regione resilienti e protetti con Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
