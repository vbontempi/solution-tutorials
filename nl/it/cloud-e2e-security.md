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

# Applica la sicurezza end-to-end a un'applicazione cloud
{: #cloud-e2e-security}

Nessuna architettura di applicazione è completa senza una chiara comprensione dei potenziali rischi per la sicurezza e di come proteggersi da tali minacce. I dati dell'applicazione sono una risorsa critica che non può essere persa, compromessa o rubata. Inoltre, i dati devono essere protetti sia quando inattivi che in transito attraverso delle tecniche di crittografia. La crittografia dei dati inattivi protegge le informazioni dalla divulgazione anche in caso di smarrimento o furto. La crittografia dei dati in transito (ad esempio su Internet) tramite metodi quali HTTPS, SSL e TLS impedisce l'intercettazione e gli attacchi cosiddetti MITM (man-in-the-middle).

Autenticare e autorizzare l'accesso degli utenti a risorse specifiche è un altro requisito comune per molte applicazioni. Potrebbe essere necessario supportare diversi schemi di autenticazione: clienti e fornitori che utilizzano identità social, partner dalle directory ospitate su cloud e dipendenti dal provider di identità di un'organizzazione.

Questa esercitazione illustra i principali servizi di sicurezza disponibili nel catalogo {{site.data.keyword.cloud}} e come utilizzarli insieme. Un'applicazione che fornisce la condivisione dei file metterà in pratica i concetti di sicurezza.
{:shortdesc}

## Obiettivi
{: #objectives}

* Crittografare il contenuto nei bucket di archiviazione con le tue proprie chiavi di crittografia
* Richiedere agli utenti di autenticarsi prima di accedere a un'applicazione
* Monitorare e controllare le chiamate API relative alla sicurezza e altre azioni tra i servizi cloud

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* Facoltativo: [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

Questa esercitazione richiede un [account diverso da Lite](https://{DomainName}/docs/account?topic=account-accounts#accounts) e può comportare dei costi. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

L'esercitazione presenta un'applicazione di esempio che consente a gruppi di utenti di caricare file in un pool di archiviazione comune e di accedere a tali file tramite link condivisibili. L'applicazione è scritta in Node.js e viene distribuita come contenitore Docker in {{site.data.keyword.containershort_notm}}. Sfrutta diversi servizi e funzioni correlati alla sicurezza per migliorare il comportamento di sicurezza dell'applicazione.

<p style="text-align: center;">

  ![Architettura](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. L'utente si connette all'applicazione.
2. Se si utilizza un dominio personalizzato e un certificato TLS, il certificato viene gestito e distribuito da {{site.data.keyword.cloudcerts_short}}.
3. {{site.data.keyword.appid_short}} protegge l'applicazione e reindirizza l'utente alla pagina di autenticazione. Gli utenti possono anche registrarsi.
4. L'applicazione viene eseguita in un cluster Kubernetes da un'immagine archiviata in {{site.data.keyword.registryshort_notm}}. Questa immagine viene automaticamente scansionata per rilevare eventuali vulnerabilità.
5. I file caricati vengono archiviati in {{site.data.keyword.cos_short}} con i metadati associati archiviati in {{site.data.keyword.cloudant_short_notm}}.
6. I bucket di archiviazione file utilizzano una chiave fornita dall'utente per crittografare i dati.
7. Le attività di gestione delle applicazioni vengono registrate da {{site.data.keyword.cloudaccesstrailshort}}.

## Prima di iniziare
{: #prereqs}

1. Installa tutti gli strumenti CLI necessari [seguendo questi passi](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
2. Assicurati di disporre della versione più recente dei plugin utilizzati in questa esercitazione; usa `ibmcloud plugin update --all` per eseguire l'upgrade.

## Crea i servizi
{: #setup}

### Decidi dove distribuire l'applicazione

1. Identifica l'**ubicazione**, l'**organizzazione e lo spazio Cloud Foundry** e il **gruppo di risorse** in cui distribuirai l'applicazione e le sue risorse.

### Acquisisci le attività dell'utente e dell'applicazione 
{: #activity-tracker }

Il servizio {{site.data.keyword.cloudaccesstrailshort}} registra le attività avviate dall'utente che modificano lo stato di un servizio in {{site.data.keyword.Bluemix_notm}}. Alla fine di questa esercitazione, riesaminerai gli eventi che sono stati generati completando i passi dell'esercitazione.

1. Accedi al catalogo {{site.data.keyword.Bluemix_notm}} e crea un'istanza di [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker). Nota che ci può essere solo un'istanza di {{site.data.keyword.cloudaccesstrailshort}} per ogni spazio Cloud Foundry.
2. Una volta creata l'istanza, modifica il suo nome in **secure-file-storage-activity-tracker**.
3. Per visualizzare tutti gli eventi del programma di traccia dell'attività, assicurati di avere le autorizzazioni assegnate:
   1. Vai a [Identity & Access > Users](https://{DomainName}/iam/#/users).
   2. Seleziona il tuo nome utente dall'elenco.
   3. In **Access Policies**, se manca, crea una politica per il servizio {{site.data.keyword.loganalysisshort_notm}} con il ruolo **Viewer** nell'ubicazione in cui è stato creato il servizio.
   4. In **Cloud Foundry Access**, assicurati di disporre del ruolo **Developer** nello spazio Cloud Foundry in cui è stato eseguito il provisioning di {{site.data.keyword.cloudaccesstrailshort}}. In caso contrario, collabora con il gestore di Cloud Foundry della tua organizzazione per l'assegnazione del ruolo.

Trova istruzioni dettagliate sulla configurazione delle autorizzazioni nella [documentazione di {{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy).
{: tip}

### Crea un cluster per l'applicazione

{{site.data.keyword.containershort_notm}} fornisce un ambiente per distribuire applicazioni altamente disponibili nei contenitori Docker eseguiti nei cluster Kubernetes.

Salta questa sezione se hai un cluster **Standard** esistente che vuoi riutilizzare con questa esercitazione.
{: tip}

1. Accedi alla [pagina di creazione del cluster](https://{DomainName}/containers-kubernetes/catalog/cluster/create).
   1. Imposta **Location** sull'ubicazione utilizzata nei passi precedenti.
   2. Imposta **Cluster type** su **Standard**.
   3. Imposta **Availability** su **Single Zone**.
   4. Seleziona una **Master Zone**.
2. Mantieni i valori predefiniti per **Kubernetes version** e **Hardware isolation**.
3. Se prevedi di distribuire solo questa esercitazione su questo cluster, imposta i nodi di lavoro (**Worker nodes**) su **1**.
4. Imposta **Cluster name** su **secure-file-storage-cluster**.
5. Fai clic sul pulsante **Create Cluster**.

Mentre viene eseguito il provisioning del cluster, creerai gli altri servizi richiesti dall'esercitazione.

### Utilizza le tue chiavi di crittografia

{{site.data.keyword.keymanagementserviceshort}} ti aiuta ad eseguire il provisioning di chiavi crittografate per le applicazioni nei servizi {{site.data.keyword.Bluemix_notm}}. {{site.data.keyword.keymanagementserviceshort}} e {{site.data.keyword.cos_full_notm}} [lavorano insieme per proteggere i tuoi dati inattivi](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos). In questa sezione, creerai una chiave root per il bucket di archiviazione.

1. Crea un'istanza di [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms).
   * Imposta il nome su **secure-file-storage-kp**.
   * Seleziona il gruppo di risorse in cui creare l'istanza del servizio.
2. In **Manage**, fai clic sul pulsante **Add Key** per creare una nuova chiave root. Verrà utilizzata per crittografare il contenuto del bucket di archiviazione.
   * Imposta il nome su **secure-file-storage-root-enckey**.
   * Imposta il tipo di chiave su **Root key**.
   * Quindi, fai clic su **Generate key**.

Porta la tua chiave personale (BYOK) [importando una chiave root esistente](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys).
{: tip}

### Configura l'archiviazione per i file utente

L'applicazione di condivisione file salva i file in un bucket {{site.data.keyword.cos_short}}. La relazione tra file e utenti viene archiviata come metadati in un database {{site.data.keyword.cloudant_short_notm}}. In questa sezione, creerai e configurerai questi servizi.

#### Un bucket per il contenuto

1. Crea un'istanza di [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
   * Imposta il **nome** su **secure-file-storage-cos**.
   * Utilizza lo stesso **gruppo di risorse** che hai usato per i servizi precedenti.
2. In **Service credentials**, crea una *nuova credenziale*.
   * Imposta il **nome** su **secure-file-storage-cos-acckey**.
   * Imposta **Role** su **Writer**.
   * Non specificare un ID servizio (**Service ID**).
   * Imposta **Inline Configuration Parameters** su **{"HMAC":true}**. Questo è necessario per generare URL pre-firmati.
   * Fai clic su **Add**.
   * Prendi nota delle credenziali facendo clic su **View credentials**. Ti serviranno in un passo successivo.
3. Fai clic su **Endpoint** dal menu: imposta **Resiliency** su **Regional** e imposta **Location** sull'ubicazione di destinazione. Copia l'endpoint del servizio **pubblico**. Verrà utilizzato in seguito nella configurazione dell'applicazione.

Prima di creare il bucket, concederai l'accesso **secure-file-storage-cos** alla chiave root archiviata in **secure-file-storage-kp**.

1. Vai a [Identity & Access > Authorizations](https://{DomainName}/iam/#/authorizations) nella console {{site.data.keyword.cloud_notm}}.
2. Fai clic sul pulsante **Create**.
3. Nel menu **Source service**, seleziona **Cloud Object Storage**.
4. Nel menu **Source service instance**, seleziona il servizio **secure-file-storage-cos** creato in precedenza.
5. Nel menu **Target service**, seleziona **Key Protect**.
6. Nel menu **Target service instance**, seleziona il servizio **secure-file-storage-kp** da autorizzare.
7. Abilita il ruolo **Reader**.
8. Fai clic sul pulsante **Authorize**.

Infine, crea il bucket.

1. Accedi all'istanza del servizio **secure-file-storage-cos** dall'[elenco delle risorse](https://{DomainName}/resources).
2. Fai clic su **Create bucket**.
   1. Imposta il **nome** su un valore univoco, ad esempio **&lt;your-initials&gt;-secure-file-upload**.
   2. Imposta **Resiliency** su **Regional**.
   3. Imposta **Location** sulla stessa ubicazione in cui hai creato il servizio **secure-file-storage-kp**.
   4. Imposta **Storage class** su **Standard**
3. Seleziona la casella di spunta **Add Key Protect Keys**.
   1. Seleziona il servizio **secure-file-storage-kp**.
   2. Seleziona **secure-file-storage-root-enckey** come chiave.
4. Fai clic su **Create bucket**.

#### Un database associa le relazioni tra gli utenti e i loro file

Il database {{site.data.keyword.cloudant_short_notm}} conterrà i metadati relativi a tutti i file caricati dall'applicazione.

1. Crea un'istanza di [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB).
   * Imposta il **nome** su **secure-file-storage-cloudant**.
   * Imposta l'ubicazione.
   * Utilizza lo stesso **gruppo di risorse** che hai usato per i servizi precedenti.
   * Imposta **Available authentication methods** su **Use only IAM**.
2. In **Service credentials**, crea una *nuova credenziale*.
   * Imposta il **nome** su **secure-file-storage-cloudant-acckey**.
   * Imposta **Role** su **Manager**
   * Mantieni i valori predefiniti per i campi *facoltativi*
   * **Add**.
3. Prendi nota delle credenziali facendo clic su **View credentials**. Ti serviranno in un passo successivo.
4. In **Manage**, avvia il dashboard Cloudant.
5. Fai clic su **Create Database** per creare un database denominato **secure-file-storage-metadata**.

### Autentica gli utenti

Con {{site.data.keyword.appid_short}}, puoi proteggere le risorse e aggiungere l'autenticazione alle tue applicazioni. {{site.data.keyword.appid_short}} [si integra](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth) con {{site.data.keyword.containershort_notm}} per autenticare gli utenti che accedono alle applicazioni distribuite nel cluster.

1. Crea un'istanza di [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID).
   * Imposta **Service name** su **secure-file-storage-appid**.
   * Utilizza la stessa **ubicazione** e lo stesso **gruppo di risorse** che hai usato per i servizi precedenti.
2. In **Identity Providers / Manage**, nella scheda **Authentication Settings**, aggiungi un **URL di reindirizzamento web** che punta al dominio che utilizzerai per l'applicazione. Ad esempio, se il dominio secondario Ingress per il tuo cluster è
`<cluster-name>.us-south.containers.appdomain.cloud`, l'URL di reindirizzamento sarà `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback`. {{site.data.keyword.appid_short}} richiede che l'URL di reindirizzamento web sia **https**. Puoi visualizzare il tuo dominio secondario Ingress nel dashboard del cluster o con `ibmcloud ks cluster-get <cluster-name>`.

Si consiglia di personalizzare i provider di identità utilizzati e l'esperienza di gestione degli accessi e degli utenti nel dashboard {{site.data.keyword.appid_short}}. Questa esercitazione utilizza le impostazioni predefinite per semplicità. Per un ambiente di produzione, considera l'utilizzo dell'autenticazione a più fattori (MFA) e delle regole avanzate per le password.
{: tip}

## Distribuisci l'applicazione

Tutti i servizi sono stati configurati. In questa sezione distribuirai l'applicazione dell'esercitazione al cluster.

### Ottieni il codice

1. Ottieni il codice dell'applicazione:
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. Vai alla directory **secure-file-storage**:
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### Crea l'immagine Docker

1. [Crea l'immagine Docker](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating) in {{site.data.keyword.registryshort_notm}}.
   - Trova l'endpoint del registro con `ibmcloud cr info`, ad esempio **us**.icr.io o **uk**.icr.io.
   - Crea uno spazio dei nomi per archiviare l'immagine del contenitore.
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - Utilizza **secure-file-storage** come nome dell'immagine.

   ```sh
   ibmcloud cr build -t <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### Compila le credenziali e le impostazioni di configurazione

1. Copia `credentials.template.env` in `credentials.env`:
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. Modifica `credentials.env` e riempi gli spazi con questi valori:
   * l'endpoint regionale del servizio {{site.data.keyword.cos_short}}, il nome del bucket, le credenziali create per **secure-file-storage-cos**,
   * e le credenziali per **secure-file-storage-cloudant**.
3. Copia `secure-file-storage.template.yaml` in `secure-file-storage.yaml`:
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. Modifica `secure-file-storage.yaml` e sostituisci i segnaposto (`$IMAGE_PULL_SECRET`, `$REGISTRY_URL`, `$REGISTRY_NAMESPACE`, `$IMAGE_NAME`, `$TARGET_NAMESPACE`, `$INGRESS_SUBDOMAIN`, `$INGRESS_SECRET`) con i valori corretti. Ad esempio, supponiamo che l'applicazione sia distribuita nello spazio dei nomi Kubernetes _predefinito_:

| Variabile | Valore | Descrizione |
| -------- | ----- | ----------- |
| `$IMAGE_PULL_SECRET` | Mantieni le righe commentate in .yaml | Un segreto per accedere al registro.  |
| `$REGISTRY_URL` | *us.icr.io* | Il registro in cui è stata creata l'immagine nella sezione precedente. |
| `$REGISTRY_NAMESPACE` | *secure-file-storage-namespace* | Lo spazio dei nomi del registro in cui è stata creata l'immagine nella sezione precedente. |
| `$IMAGE_NAME` | *secure-file-storage* | Il nome dell'immagine Docker. |
| `$TARGET_NAMESPACE` | *default* | Lo spazio dei nomi Kubernetes in cui verrà eseguito il push dell'applicazione. |
| `$INGRESS_SUBDOMAIN` | *secure-file-stora-123456.us-south.containers.appdomain.cloud* | Richiama dalla pagina della panoramica del cluster o con `ibmcloud ks cluster-get secure-file-storage-cluster`. |
| `$INGRESS_SECRET` | *secure-file-stora-123456* | Richiama dalla pagina della panoramica del cluster o con `ibmcloud ks cluster-get secure-file-storage-cluster`. |

`$IMAGE_PULL_SECRET` è necessario solo se vuoi utilizzare un altro spazio dei nomi Kubernetes rispetto a quello predefinito. Ciò richiede un'ulteriore configurazione di Kubernetes (ad esempio la [creazione di un segreto di registro Docker nel nuovo spazio dei nomi](https://{DomainName}/docs/containers?topic=containers-images#other)).
{: tip}

### Distribuisci al cluster

1. Richiama la configurazione del cluster e imposta la variabile di ambiente KUBECONFIG.
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. Crea il segreto utilizzato dall'applicazione per ottenere le credenziali del servizio:
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. Esegui il bind dell'istanza del servizio {{site.data.keyword.appid_short_notm}} al cluster.
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
   Se hai diversi servizi con lo stesso nome, il comando avrà esito negativo. Devi passare il GUID del servizio anziché il suo nome. Per trovare il GUID di un servizio, utilizza `ibmcloud resource service-instance secure-file-storage-appid`.
   {: tip}
4. Distribuisci l'applicazione.
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## Verifica l'applicazione

L'applicazione è accessibile all'indirizzo `https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/`.

1. Vai alla home page dell'applicazione. Verrai reindirizzato alla pagina di accesso predefinita di {{site.data.keyword.appid_short_notm}}.
2. Registrati per un nuovo account con un indirizzo email valido.
3. Attendi l'email nella posta in arrivo per verificare l'account.
4. Esegui l'accesso.
5. Scegli un file da caricare. Fai clic su **Upload**.
6. Utilizza l'azione **Share** su un file per generare un URL pre-firmato che può essere condiviso con altri per accedere al file. Il link è impostato per scadere dopo 5 minuti.

Gli utenti autenticati dispongono di spazi propri per archiviare i file. Sebbene non possano vedere i reciproci file, possono generare degli URL pre-firmati per concedere l'accesso temporaneo a un file specifico.

Puoi trovare ulteriori dettagli sull'applicazione nel [repository del codice sorgente](https://github.com/IBM-Cloud/secure-file-storage).

## Esamina gli eventi di sicurezza
Ora che l'applicazione e i suoi servizi sono stati distribuiti correttamente, puoi riesaminare gli eventi di sicurezza generati da tale processo. Tutti gli eventi sono disponibili centralmente nell'istanza {{site.data.keyword.cloudaccesstrailshort}} e sono accessibili tramite [l'interfaccia utente grafica (Kibana), la CLI o l'API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status).

1. Dall'[elenco delle risorse {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), individua l'istanza {{site.data.keyword.cloudaccesstrailshort}} **secure-file-storage-activity-tracker** e apri il relativo dashboard.
2. Per impostazione predefinita, la scheda **Manage** mostra **Space logs**. Passa a **Account logs** facendo clic sul selettore accanto a **View logs**. Dovrebbero essere visualizzati diversi eventi.
3. Fai clic su **View in Kibana** per aprire il visualizzatore eventi completo.
4. Esamina i dettagli dell'evento facendo clic su **Discover**.
5. Da **Available Fields**, aggiungi **action_str** e **initiator.name_str**.
6. Espandi le voci interessate facendo clic sull'icona a triangolo e quindi scegliendo un formato tabella o JSON.

Puoi modificare le impostazioni per l'aggiornamento automatico e l'intervallo di tempo visualizzato, modificando in tal modo come e quali dati vengono analizzati.
{: tip}

## Facoltativo: utilizza un dominio personalizzato e crittografa il traffico di rete
Per impostazione predefinita, l'applicazione è accessibile su un nome host generico in un dominio secondario di `containers.appdomain.cloud`. Tuttavia, è anche possibile utilizzare un dominio personalizzato con l'applicazione distribuita. Per il supporto continuo di **https**, per l'accesso con traffico di rete crittografato, è necessario fornire un certificato per il nome host desiderato o un certificato jolly. Nella seguente sezione, caricherai un certificato esistente in {{site.data.keyword.cloudcerts_short}} e lo distribuirai al cluster. Aggiornerai anche la configurazione dell'applicazione per utilizzare il dominio personalizzato.

Un esempio di come ottenere un certificato da [Let's Encrypt](https://letsencrypt.org/) è descritto nel seguente [blog di {{site.data.keyword.cloud}}](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/).
{: tip}

1. Crea un'istanza di [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)
   * Imposta il nome su **secure-file-storage-certmgr**.
   * Utilizza la stessa **ubicazione** e lo stesso **gruppo di risorse** che hai usato per gli altri servizi.
2. Fai clic su **Import Certificate** per importare il tuo certificato.
   * Imposta il nome su **SecFileStorage** e la descrizione su **Certificate for e2e security tutorial**.
   * Carica il file del certificato utilizzando il pulsante **Browse**.
   * Fai clic su **Import** per completare il processo di importazione.
3. Individua la voce per il certificato importato ed espandila
   * Verifica la voce del certificato, ad esempio che il nome di dominio corrisponda al tuo dominio personalizzato. Se hai caricato un certificato jolly, nel nome del dominio è incluso un asterisco.
   * Fai clic sul simbolo di **copia** accanto al **crn** del certificato.
4. Passa alla riga di comando per distribuire al cluster le informazioni sul certificato sotto forma di segreto. Immetti il seguente comando dopo aver copiato il crn dal passo precedente.
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <the copied crn from previous step>
   ```
   {: codeblock}
   Verifica che il cluster riconosca il certificato immettendo il seguente comando.
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. Modifica il file `secure-file-storage.yaml`.
   * Trova la sezione per **Ingress**.
   * Elimina il commento e modifica le righe relative ai domini personalizzati e inserisci il tuo dominio e nome host.
   La voce CNAME per il tuo dominio personalizzato deve puntare al cluster. Per i dettagli, consulta questa [guida sull'associazione dei domini personalizzati](https://{DomainName}/docs/containers?topic=containers-ingress#private_3) nella documentazione.
   {: tip}
6. Applica le modifiche di configurazione alla distribuzione:
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. Ritorna al browser. Nell'[elenco delle risorse {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), individua il servizio {{site.data.keyword.appid_short}} precedentemente creato e configurato e avvia il suo dashboard di gestione.
   * Vai a **Manage** sotto **Identity Providers** e seleziona **Settings**.
   * Nel modulo **Add web redirect URLs**, aggiungi `https://secure-file-storage.<your custom domain>/appid_callback` come un altro URL.
8. Adesso dovrebbe essere tutto pronto. Verifica l'applicazione accedendo ad essa tramite il dominio personalizzato che hai configurato `https://secure-file-storage.<your custom domain>`.

## Espandi l'esercitazione

La sicurezza non è mai abbastanza. Prova i seguenti suggerimenti per migliorare la sicurezza della tua applicazione.

* Utilizza [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) per eseguire scansioni di codice statico e dinamico
* Assicurati che venga rilasciato solo il codice di qualità utilizzando politiche e regole con [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)

## Rimuovi le risorse
{:removeresources}

Per rimuovere la risorsa, elimina il contenitore distribuito e quindi i servizi di cui è stato eseguito il provisioning.

1. Elimina il contenitore distribuito:
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. Elimina i segreti per la distribuzione:
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. Rimuovi l'immagine Docker dal registro del contenitore:
   ```sh
   ibmcloud cr image-rm <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. Nell'[elenco delle risorse {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), individua le risorse create per questa esercitazione. Utilizza la casella di ricerca e **secure-file-storage** come pattern. Elimina ciascuno dei servizi facendo clic sul menu contestuale accanto a ogni servizio e scegliendo **Delete Service**. Nota che il servizio {{site.data.keyword.keymanagementserviceshort}} può essere rimosso solo dopo aver eliminato la chiave. Fai clic sull'istanza del servizio per accedere al dashboard correlato e per eliminare la chiave.

Se condividi un account con altri utenti, assicurati sempre di eliminare solo le tue risorse.
{: tip}

## Contenuto correlato
{:related}

* [Documentazione di {{site.data.keyword.security-advisor_short}}](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [Sicurezza per salvaguardare e monitorare le tue applicazioni cloud](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [Sicurezza della piattaforma {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/overview?topic=overview-security#security)
* [Sicurezza in IBM Cloud](https://www.ibm.com/cloud/security)
* [Esercitazione: Procedure consigliate per l'organizzazione di utenti, team, applicazioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [Applicazioni sicure su IBM Cloud con i certificati jolly](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
