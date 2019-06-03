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

# Revisione dei servizi, delle risorse e dell'utilizzo di {{site.data.keyword.cloud_notm}}
{: #cloud-usage}
Con l'aumentare dell'adozione del cloud, i responsabili IT e finanziari dovranno comprendere l'utilizzo del cloud nel contesto dell'innovazione e del controllo dei costi. È possibile rispondere a domande come: "Quali servizi utilizzano i team?", "Quanto costa gestire una soluzione?" e "Come posso contenere la dispersione?" analizzando i dati disponibili. Questa esercitazione introduce dei modi per esplorare tali informazioni e rispondere alle domande più comuni relative all'utilizzo.
{:shortdesc}

## Obiettivi
{: #objectives}
* Dettagliare le risorse {{site.data.keyword.cloud_notm}}: applicazioni e servizi Cloud Foundry, risorse di Identity and Access Management e dispositivi dell'infrastruttura
* Associare le risorse {{site.data.keyword.cloud_notm}} con l'utilizzo e la fatturazione
* Definire le relazioni tra le risorse {{site.data.keyword.cloud_notm}} e i team di sviluppo
* Sfruttare i dati di utilizzo e di fatturazione per creare serie di dati a fini contabili

## Architettura
{: #architecture}

![Architettura](images/solution38/Architecture.png)

## Prima di iniziare
{: #prereqs}

* Installa la [CLI {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* Installa [cURL](https://curl.haxx.se/)
* Installa [Node.js](https://nodejs.org/)
* Installa [json2csv](https://www.npmjs.com/package/json2csv) utilizzando il comando `npm install -g json2csv`
* Installa [jq](https://stedolan.github.io/jq/)

## Background
{: #background}

Prima di eseguire i comandi relativi all'inventario e al dettaglio dell'utilizzo di {{site.data.keyword.cloud_notm}}, è utile avere un qualche background sulle ampie categorie di utilizzo e sulla loro funzione. I termini chiave utilizzati più avanti nell'esercitazione sono in grassetto. Una visualizzazione utile delle seguenti risorse è disponibile nella [documentazione Gestione del tuo account](https://{DomainName}/docs/account?topic=account-overview#overview).

### Cloud Foundry
Cloud Foundry è un PaaS (platform-as-a-service) open-source su {{site.data.keyword.cloud_notm}} che ti consente di distribuire e ridimensionare le applicazioni e i **servizi** senza la gestione dei server. Cloud Foundry organizza le applicazioni e i servizi in organizzazioni o spazi.  Un'**organizzazione** è un account di sviluppo che uno o più utenti possono possedere e utilizzare. Un'organizzazione può contenere più spazi. Ogni **spazio** fornisce agli utenti l'accesso a un'ubicazione condivisa per lo sviluppo, la distribuzione e la manutenzione delle applicazioni.

### Identity and Access Management
IBM Cloud IAM (Identity and Access Management) ti consente di autenticare in modo sicuro gli utenti per i servizi di piattaforma e controllare l'accesso alle risorse in modo coerente attraverso la piattaforma {{site.data.keyword.cloud_notm}}. Le offerte più recenti e i servizi di Cloud Foundry migrati sono disponibili sotto forma di **risorse** gestite da {{site.data.keyword.cloud_notm}} Identity and Access Management. Le risorse sono organizzate in **gruppi di risorse** e forniscono il controllo degli accessi tramite politiche e ruoli.

### Infrastruttura
L'infrastruttura comprende diverse opzioni di calcolo: server bare metal, istanze di server virtuali e nodi Kubernetes. Ciascuno di questi è visto come un **dispositivo** nella console.

### Account
Le suddette risorse sono associate a un **account** per scopi di fatturazione. Gli **utenti** sono invitati all'account e viene loro concesso l'accesso alle diverse risorse all'interno dell'account.

## Assegna le autorizzazioni
Per visualizzare l'inventario e l'utilizzo del cloud, dovrai disporre dei ruoli appropriati assegnati dall'amministratore dell'account. Se sei l'amministratore dell'account, passa alla sezione successiva.

1. L'amministratore dell'account deve accedere a {{site.data.keyword.cloud_notm}} e andare alla pagina [**Identity & Access Users**](https://{DomainName}/iam/#/users).
2. L'amministratore dell'account può selezionare il tuo nome dall'elenco di utenti nell'account per assegnare le autorizzazioni appropriate.
3. Nella scheda **Access policies**, fai clic sul pulsante **Assign Access** ed effettua le seguenti modifiche.
   1. Seleziona il tile **Assign access within a resource group**. Seleziona i **gruppi di risorse** a cui concedere l'accesso e applica il ruolo **Viewer**. Termina facendo clic sul pulsante **Assign**. Questo ruolo è richiesto per i comandi `resource groups` e `billing resource-group-usage`.
   2. Seleziona il tile **Assign access by using Cloud Foundry**. Seleziona il menu di overflow accanto a ciascuna **organizzazione** a cui concedere l'accesso. Seleziona **Edit organization role** dal menu. Seleziona **Billing Manager** dall'elenco **Organization roles**. Termina facendo clic sul pulsante **Save role**. Questo ruolo è richiesto per il comando `billing org-usage`.
   3. Seleziona il tile **Assign access to resources**. Seleziona **All Identity and Access enabled services** nel menu a discesa **Services**. Seleziona il ruolo **Editor** in **Assign platform access roles**. Questo ruolo è richiesto per il comando `resource tag-attach`.

## Individua le risorse con il comando di ricerca
{: #search}

Man mano che i team di sviluppo iniziano a utilizzare i servizi cloud, i manager trarranno vantaggio dalla conoscenza di quali servizi sono stati distribuiti. Le informazioni sulla distribuzione aiutano a rispondere a domande relative all'innovazione e alla gestione dei servizi:
- Quali competenze relative ai servizi possono essere condivise tra i team per migliorare altri progetti?
- Quali sono gli aspetti carenti in comune tra tutti i team?
- Quali team stanno utilizzando un servizio che richiede una correzione critica o che presto sarà obsoleto?
- In che modo i team possono riesaminare le loro istanze del servizio per ridurre al minimo la dispersione?

La ricerca non è limitata ai servizi e alle risorse. Puoi anche eseguire query sulle risorse cloud come organizzazioni e spazi Cloud Foundry, gruppi di risorse, bind di risorse, alias, ecc. Per ulteriori esempi, vedi la documentazione [ibmcloud resource search](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search).
{:tip}

1. Accedi a {{site.data.keyword.cloud_notm}} tramite la riga di comando e specifica il tuo account Cloud Foundry. Vedi [Introduzione alla CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh
    ibmcloud login
    ```
    {: pre}

    ```sh
    ibmcloud target --cf
    ```
    {: pre}

2. Effettua l'inventario di tutti i servizi Cloud Foundry utilizzati nell'account.
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. I filtri booleani possono essere applicati per ampliare o restringere le ricerche. Ad esempio, trova i servizi e le applicazioni Cloud Foundry così come le risorse IAM utilizzando la seguente query.
    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. Per notificare i team che utilizzano un particolare tipo di servizio, esegui una query utilizzando il nome del servizio. Sostituisci `<name>` con del testo, ad esempio `weather` o `cloudant`. Quindi, ottieni il nome dell'organizzazione sostituendo `<Organization ID>` con l'**ID organizzazione**.
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

L'assegnazione di tag e la ricerca possono essere utilizzate insieme per fornire un'identificazione personalizzata delle risorse. Ciò implica collegare la tag alle risorse ed eseguire ricerche utilizzando i nomi delle tag. Crea una tag denominata env:tutorial.

1. Collega la tag a una risorsa. Puoi ottenere il CRN di una risorsa dall'interfaccia utente o tramite `ibmcloud resource service-instance <name|id>`. I CRN (Cloud Resource Name) identificano in modo univoco le risorse IBM Cloud.
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <resource CRN>
   ```
   {: pre}
1. Cerca le risorse cloud che corrispondono a una determinata tag utilizzando la seguente query.
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

Combinando le query di ricerca avanzata con uno schema di assegnazione di tag concordato a livello aziendale, i gestori e i responsabili del team possono identificare e intervenire più facilmente su applicazioni, risorse e servizi cloud.

## Esplora l'utilizzo con il Dashboard di utilizzo
{: #dashboard}

Una volta che la direzione è a conoscenza dei servizi che i team stanno utilizzando, la prossima domanda frequente è: "Quanto costa gestire questi servizi?" Il metodo più semplice per determinare l'utilizzo e il costo consiste nel riesaminare il Dashboard di utilizzo di {{site.data.keyword.cloud_notm}}.

1. Accedi a {{site.data.keyword.cloud_notm}} e vai alla pagina [Account Usage Dashboard](https://{DomainName}/account/usage).
2. Dal menu a discesa **Groups**, seleziona un'organizzazione Cloud Foundry per visualizzare l'utilizzo del servizio.
3. Per una determinata offerta di servizi (**Service Offering**), fai clic su **View Instances** per visualizzare le istanze del servizio che sono state create.
4. Nella pagina seguente, scegli un'istanza e fai clic su **View Instance**. La pagina risultante fornisce i dettagli sull'istanza come organizzazione, spazio e ubicazione, nonché i singoli elementi di riga che generano il costo totale.
5. Utilizzando il breadcrumb, rivisita il [Dashboard di utilizzo](https://{DomainName}/account/usage).
6. Dal menu a discesa **Groups**, cambia il selettore in **Resource Groups** e seleziona un gruppo, ad esempio **default**.
7. Conduci una revisione simile delle istanze disponibili.

## Esplora l'utilizzo con la riga di comando

In questa sezione, esplorerai l'utilizzo con l'interfaccia della riga di comando.

1. Elenca tutte le organizzazioni Cloud Foundry a tua disposizione e imposta una variabile di ambiente per memorizzarne una per la verifica.
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<org name>
    ```
    {: pre}

2. Dettaglia la fatturazione e l'utilizzo per una determinata organizzazione con il comando `billing`. Specifica un mese particolare utilizzando l'indicatore `-d` con una data in formato AAAA-MM.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Conduci la stessa analisi per le risorse. Ogni account ha un gruppo di risorse di `default`. Sostituisci il valore `<group>` per un gruppo di risorse elencato nel primo comando.
    ```sh
    ibmcloud resource groups
    ```
    {: pre}

    ```sh
    export RESOURCE_GROUP=<group>
    ```
    {: pre}

    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP
    ```
    {: pre}

4. Puoi visualizzare i servizi Cloud Foundry e le risorse IAM utilizzando il comando `resource-instances-usage`. A seconda del tuo livello di autorizzazione, esegui i comandi appropriati.
    - Se sei l'amministratore dell'account o ti è stato concesso il ruolo di Amministratore per tutti i servizi abilitati per l'accesso e l'identità, esegui semplicemente il comando `resource-instances-usage`.
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  Se non sei l'amministratore dell'account, è possibile utilizzare i seguenti comandi poiché è stato impostato il ruolo di Visualizzatore all'inizio dell'esercitazione.
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. Per visualizzare i dispositivi dell'infrastruttura, utilizza il comando `sl`. Quindi, utilizza il comando `vs` per esaminare i {{site.data.keyword.virtualmachinesshort}}.
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## Esporta l'utilizzo con la riga di comando
{: #usagecmd}

Alcuni dirigenti spesso hanno bisogno di esportare i dati in un'altra applicazione. Un esempio comune è l'esportazione dei dati di utilizzo in un foglio di calcolo. In questa sezione, esporterai i dati di utilizzo nel formato CSV (comma separated value), che è compatibile con la maggior parte delle applicazioni per fogli di calcolo.

Questa sezione utilizza due strumenti di terze parti: `jq` e`json2csv`. Ognuno dei seguenti comandi è composto da tre passi: acquisizione dei dati di utilizzo come JSON, analisi del JSON e formattazione del risultato come CSV. Il processo di acquisizione e conversione dei dati JSON non è limitato a `json2csv` ed è possibile utilizzare altri strumenti.

Utilizza l'opzione `-p` per stampare i risultati. Se i dati vengono stampati in modo inadeguato, rimuovi l'argomento `-p` per stampare i dati CSV non elaborati.
{:tip}

1. Esporta l'utilizzo di un gruppo di risorse con i costi previsti per ciascun tipo di risorsa.
    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. Dettaglia le istanze per ciascun tipo di risorsa nel gruppo di risorse.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. Segui lo stesso approccio per un'organizzazione Cloud Foundry.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. Aggiungi i costi previsti ai dati utilizzando una query `jq` più avanzata. Ciò creerà più righe poiché alcuni tipi di risorse hanno più metriche di costo.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. Utilizza la stessa query `jq` per elencare anche le risorse Cloud Foundry con i costi associati.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. Esporta i dati in un file rimuovendo l'opzione `-p` e inserendo l'output in un file. Il file CSV risultante può essere aperto con un'applicazione per fogli di calcolo.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## Esporta l'utilizzo con le API
{: #usageapi}

Sebbene i comandi `billing` siano utili, provare ad assemblare una vista del "quadro generale" utilizzando l'interfaccia della riga di comando è piuttosto tedioso. Allo stesso modo, il Dashboard di utilizzo presenta una panoramica delle organizzazioni e dei gruppi di risorse, ma non necessariamente l'utilizzo di un team o di un progetto. In questa sezione inizierai a esplorare un approccio più basato sui dati per ottenere l'utilizzo per i requisiti personalizzati.

1. Nel terminale, imposta la variabile di ambiente `IBMCLOUD_TRACE=true` per stampare le richieste e le risposte API.
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. Esegui di nuovo il comando `billing org-usage` per visualizzare le chiamate API. Nota che per questo singolo comando vengono utilizzati più host e instradamenti API.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Ottieni un token OAuth ed esegui una delle chiamate API visualizzate nel comando `billing org-usage`. Nota che alcune API utilizzano il token UAA mentre altre possono utilizzare un token IAM come autorizzazione bearer.
    ```sh
    export IAM_TOKEN=`ibmcloud iam oauth-tokens | head -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    export UAA_TOKEN=`ibmcloud iam oauth-tokens | tail -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    curl -H "Authorization: Bearer $UAA_TOKEN" https://mccp.us-south.cf.cloud.ibm.com/v2/organizations?q=name:$ORG_NAME
    ```
    {: pre}

4. Per eseguire le API di infrastruttura, ottieni e imposta le variabili di ambiente per il **nome utente API** e la **chiave di autenticazione** visualizzati nel tuo [profilo utente](https://control.softlayer.com/account/user/profile). Se non vedi un nome utente API e una chiave di autenticazione, puoi crearli nel menu **Actions** accanto al tuo nome nell'[elenco utenti](https://control.softlayer.com/account/users).
    ```sh
    export IAAS_USERNAME=<API Username>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<Authentication Key>
    ```
    {: pre}

5. Ottieni i totali di fatturazione dell'infrastruttura e le voci di fatturazione utilizzando le seguenti API. API simili sono documentate [qui](https://softlayer.github.io/reference/services/SoftLayer_Account/).
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. Disabilita la traccia.
    ```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

Sebbene l'approccio basato sui dati offra la massima flessibilità nell'esplorazione dell'utilizzo, una spiegazione più approfondita va oltre lo scopo di questa esercitazione. È stato creato il progetto GitHub [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample) che combina i servizi {{site.data.keyword.cloud_notm}} con Cloud Functions per fornire un'implementazione di esempio.

## Espandi l'esercitazione
{: #expand}

Utilizza i seguenti suggerimenti per espandere la tua analisi sui dati relativi all'inventario e all'utilizzo.

- Esplora i comandi `ibmcloud billing` con l'opzione `--output json`. Questo mostrerà ulteriori proprietà dei dati disponibili e non trattate nell'esercitazione.
- Leggi il post del blog [Using the {{site.data.keyword.cloud_notm}} command line to find resources](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/) per ulteriori esempi su `ibmcloud resource search` e su quali proprietà possono essere utilizzate nelle query.
- Esamina le [API dell'account dell'infrastruttura](https://softlayer.github.io/reference/services/SoftLayer_Account/) per informazioni sulle API aggiuntive per analizzare l'utilizzo dell'infrastruttura.
- Esamina il [manuale jq](https://stedolan.github.io/jq/manual/) per le query avanzate per aggregare i dati di utilizzo.
