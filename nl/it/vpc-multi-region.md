---
copyright:
  years: 2019
lastupdated: "2019-04-02"
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
{:important: .important}

# Distribuisci i carichi di lavoro isolati tra più ubicazioni e zone
{: #vpc-multi-region}

IBM accetterà un numero limitato di clienti che parteciperanno a un programma Early Access al VPC a partire dai primi di aprile 2019, con l'utilizzo completo che verrà aperto nei mesi seguenti. Se la tua organizzazione desidera ottenere l'accesso all'IBM Virtual Private Cloud, completa questo [modulo per nomina](https://{DomainName}/vpc){: new_window} e un rappresentante di IBM ti contatterà per spiegarti i passi successivi.
{: important}

Questa esercitazione ti guida nella configurazione dei carichi di lavoro isolati eseguendo il provisioning dei VPC in diverse regioni IBM Cloud. Le regioni con sottoreti e VSI (virtual server instance). Queste VSI vengono create in più zone all'interno di una regione per aumentare la resilienza all'interno di una regione e globalmente configurando i programmi di bilanciamento del carico con i pool di backend, i listener di frontend e i controlli dell'integrità appropriati.

Per il programma di bilanciamento globale, eseguirai il provisioning di un servizio IBM Cloud Internet Services (CIS) dal catalogo e, per gestire il certificato SSL per tutte le richieste HTTPS in entrata, creerai il servizio del catalogo {{site.data.keyword.cloudcerts_long_notm}} e importerai il certificato insieme alla chiave privata.

{:shortdesc}

## Obiettivi
{: #objectives}

* Comprendere l'isolamento dei carichi di lavoro tramite gli oggetti dell'infrastruttura disponibili per i VPC (virtual private cloud).
* Utilizzare un programma di bilanciamento del carico tra le zone all'interno di una regione per distribuire il traffico tra i server virtuali. 
* Utilizzare un programma di bilanciamento del carico globale tra le regioni per aumentare la resilienza e ridurre la latenza.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.loadbalancer_full}}](https://{DomainName}/vpc/provision/loadBalancer)
- IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
- [{{site.data.keyword.cloudcerts_long_notm}}](https://{DomainName}/catalog/services/cloudcerts)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/estimator/review) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura
{: #architecture}

  ![Architettura](images/solution41-vpc-multi-region/Architecture.png)

1. L'amministratore (DevOps) esegue il provisioning delle VSI nelle sottoreti in due diverse zone in un VPC nella regione 1 e ripete lo stesso processo in un VPC creato nella regione 2.
2. L'amministratore crea un programma di bilanciamento del carico con un pool di backend dei server delle sottoreti in zone diverse della regione 1 e un listener di frontend. Ripete lo stesso processo nella regione 2. 
3. L'amministratore esegue il provisioning del servizio Cloud Internet Services con un dominio personalizzato associato e crea un programma di bilanciamento del carico globale che punta ai programmi di bilanciamento del carico creati in due diversi VPC. 
4. L'amministratore abilita la crittografia HTTPS aggiungendo il certificato SSL del dominio al servizio Certificate Manager. 
5. L'utente di internet effettua una richiesta HTTP/HTTPS e il programma di bilanciamento del carico globale gestisce la richiesta. 
6. La richiesta viene instradata ai programmi di bilanciamento del carico sia globali che locali. La richiesta viene poi soddisfatta dall'istanza del server disponibile. 

## Prima di iniziare
{: #prereqs}

- Controlla le autorizzazioni utente. Assicurati che il tuo account utente disponga di autorizzazioni sufficienti per creare e gestire le risorse VPC. Per un elenco di autorizzazioni necessarie, vedi [Concessione delle autorizzazioni necessarie agli utenti VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Hai bisogno di una chiave SSH per collegarti ai server virtuali. Se non hai una chiave SSH, vedi le [istruzioni per creare una chiave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- CIS (Cloud Internet Services) richiede che tu sia il proprietario di un dominio personalizzato in modo che tu possa configurare il DNS affinché questo dominio punti al server dei nomi CIS (Cloud Internet Services). Nel caso in cui tu non lo sia, puoi acquistarne uno da un registrar come ad esempio [godaddy.com](http://godaddy.com/).

## Crea VPC, sottoreti e VSI
{: #create-infrastructure}

In questa sezione, creerai il tuo VPC nella regione 1 con sottoreti create in due diverse zone della regione 1 seguito dal provisioning delle VSI.

Per creare il tuo {{site.data.keyword.vpc_short}} nella regione 1,

1. Passa alla pagina della [panoramica di VPC](https://{DomainName}/vpc/overview) e fai clic su **Create a VPC**.
2. Nella sezione **New virtual private cloud**: 
   * Immetti **vpc-region1** come nome per il tuo VPC.
   * Seleziona un **gruppo di risorse**.
   * Facoltativamente, aggiungi delle **tag** per organizzare le tue risorse.
3. Seleziona **Create new default (Allow all)** come ACL (access control list) predefinito del tuo VPC.
4. Deseleziona SSH ed esegui il ping da **Default security group** e lascia deselezionato **classic access**.
5. In **New subnet for VPC**:
   * Come nome univoco, immetti **vpc-region1-zone1-subnet**.
   * Seleziona un'ubicazione (ad esempio, Dallas) che chiamerai **regione 1** e una zona nella regione 1 (ad esempio, Dallas 1) che chiamerai **zona 1**.
   * Immetti l'intervallo di IP per la sottorete nella notazione CIDR, ad esempio **10.xxx.0.0/24**. Lascia invariato **Address prefix** e seleziona 256 come valore per **Number of addresses**.
6. Seleziona **Use VPC default** per l'ACL (access control list) della tua sottorete. Puoi configurare le regole in entrata e in uscita in un secondo momento. 
7. Poiché tutte le VSI nella sottorete avranno un IP mobile collegato, non è necessario abilitare un gateway pubblico per la sottorete. Le VSI avranno la connettività Internet tramite i loro IP mobili. 
8. Fai clic su **Create virtual private cloud** per eseguire il provisioning dell'istanza.

Per confermare la creazione della sottorete, fai clic su **Subnets** nel riquadro di sinistra e aspetta che lo stato cambi in **Available**. Puoi creare una nuova sottorete in **Subnets**.

### Crea una sottorete nella zona 2

1. Fai clic su **New Subnet**, immetti **vpc-region1-zone2-subnet** come nome univoco per la tua sottorete e seleziona **vpc-region1** come VPC.
2. Seleziona un'ubicazione che chiameremo come la regione 1 di cui sopra (ad esempio, Dallas) e seleziona una zona diversa nella regione 1 (ad esempio, Dallas 2), chiamerai la zona selezionata **zona 2**.
3. Immetti l'intervallo di IP per la sottorete nella notazione CIDR, ad esempio **10.xxx.64.0/24**. Lascia invariato **Address prefix** e seleziona 256 come valore per **Number of addresses**.
4. Seleziona **Use VPC default** per l'ACL (access control list) della tua sottorete. 

### Esegui il provisioning delle VSI
Una volta che lo stato delle sottorete cambia in **Available**,

1. Fai clic su **vpc-region1-zone1-subnet** e fai clic su **Attached instances**, quindi su **New instance**.
2. Immetti un nome univoco e seleziona **vpc-region1-zone1-vsi**. Quindi, seleziona il VPC che hai creato prima e l'**ubicazione** insieme alla **zona** come hai fatto in precedenza. 
3. Scegli una qualsiasi immagine **Ubuntu Linux**, fai clic su **All profiles** e in **Compute**, scegli **c-2x4** con 2vCPU e 4 GB di RAM.
4. Per **SSH keys**, seleziona la chiave SSH che hai creato inizialmente. 
5. In **Network interfaces**, fai clic sull'icona **Edit** accanto a Security Groups
   * Controlla se **vpc-region1-zone1-subnet** è selezionato come sottorete. Nel caso in cui non lo sia, selezionalo. 
   * Fai clic su **Save**.
   * Fai clic su **Create virtual server instance**.
6.  Attendi fino a quando lo stato della VSI non diventa **Powered On**. Quindi, seleziona la VSI **vpc-region1-zone1-vsi**, scorri fino a **Network Interfaces** e fai clic su **Reserve** in **Floating IP** per associare un indirizzo IP pubblico alla tua VSI. Salva l'indirizzo IP associato negli appunti per riferimento futuro. 
7. **RIPETI** i passi da 1 a 6 per eseguire il provisioning di una VSI nella **zona 2** della **regione 1**.

Passa a **VPC** e **Subnets** in **Network** nel riquadro di sinistra e **RIPETI** i passi sopra riportati per eseguire il provisioning di un nuovo VPC con sottoreti e VSI nella **regione 2** seguendo le stesse convenzioni di denominazione indicate in precedenza. 

## Installa e configura un server web nelle VSI
{: #install-configure-web-server-vsis}

Segui i passi menzionati in [Accedi in modo sicuro alle istanze remote con un host bastion](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) per la manutenzione sicura dei server utilizzando un host bastion che funge da server `jump` e un gruppo di sicurezza di manutenzione. {:tip}

Una volta eseguito correttamente l'SSH nel server di cui è stato eseguito il provisioning nella sottorete della zona 1 della regione 1,

1. Quando richiesto, esegui i comandi riportati di seguito per installare Nginx come tuo server web
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   {:codeblock}
2. Controlla lo stato del servizio Nginx con i seguenti comandi:
   ```
   sudo systemctl status nginx
   ```
   {:pre}
   L'output dovrebbe mostrarti il servizio Nginx come **attivo** e in esecuzione.
3. Dovrai aprire le porte **HTTP (80)** e **HTTPS (443)** per ricevere il traffico (le richieste). Puoi eseguire tale operazione modificando il firewall tramite [UFW](https://help.ubuntu.com/community/UFW) - `sudo ufw enable` e abilitando il profilo ‘Nginx Full’ che include le regole per entrambe le porte:
   ```
   sudo ufw allow 'Nginx Full'
   ```
   {:pre}
4. Per verificare che Nginx funzioni come previsto, apri `http://FLOATING_IP` nel tuo browser e dovresti vedere la pagina di benvenuto predefinita di Nginx.
5. Per aggiornare la pagina html con i dettagli della regione e della zona, esegui il comando riportato di seguito
   ```
   nano /var/www/html/index.nginx-debian.html
   ```
   {:pre}
   Accoda la regione e la zona che specificano che il _server è in esecuzione nella **zona 1 della regione 1**_ alla tag `h1` inserendo tra virgolette `Welcome to nginx!` e salva le modifiche.
6. Riavvia il server nginx per riflettere le modifiche
   ```
   sudo systemctl restart nginx
   ```
   {:pre}

**RIPETI** i passi da 1 a 6 per installare e configurare il server web nelle VSI nelle sottoreti di tutte le zone e non dimenticare di aggiornare l'html con le rispettive informazioni sulla zona. 


## Distribuisci il traffico tra le zone con i programmi di bilanciamento del carico
{: #distribute-traffic-with-load-balancers}

In questa sezione, creerai due programmi di bilanciamento del carico. Uno in ciascuna regione per distribuire il traffico tra più istanze del server nelle rispettive sottoreti all'interno di zone differenti. 

### Configura i programmi di bilanciamento del carico

1. Passa a **Load balancers** e fai clic su **New load balancer**.
2. Assegna **vpc-lb-region1** come nome univoco, seleziona **vpc-region1** come tuo VPC (Virtual Private Cloud) seguito dal gruppo di risorse in cui è stato creato il VPC, Tipo: **Public** e **region1** come regione.
3. Seleziona gli IP privati sia della **zona 1** che della **zona 2** della **regione 1**.
4. Crea un nuovo pool di backend delle VSI che agiscono come peer uguali per condividere il traffico instradato al pool. Imposta i parametri con i valori riportati di seguito e fai clic su **create**.
	- **Name**:  region1-pool
	- **Protocol**: HTTP
	- **Method**: Round robin
	- **Session stickiness**: Nessuno
	- **Health check path**: /
	- **Health protocol**: HTTP
	- **Interval(sec)**: 15
	- **Timeout(sec)**: 2
	- **Max retries**: 2
5. Fai clic su **Attach** per aggiungere le istanze del server a region1-pool
   - Seleziona l'IP privato di **vpc-region1-zone1-subnet**, seleziona l'istanza che hai creato e imposta 80 come porta.
   - Fai clic su **Add** e, questa volta, seleziona l'IP privato di **vpc-region1-zone2-subnet**, seleziona l'istanza e imposta 80 come porta.
   - Fai clic su **Attach** per completare la creazione del pool di backend.
6. Fai clic su **New listener** per creare un nuovo listener di frontend; un listener è un processo che controlla le richieste di connessione. 
   - **Protocol**: HTTP
   - **Port**: 80
   - **Back-end pool**: region1-pool
   - **Maxconnections**: lascialo vuoto e fai clic su **create**.
7. Fai clic su **Create load balancer** per eseguire il provisioning di un programma di bilanciamento del carico. 

### Verifica i programmi di bilanciamento del carico

1. Attendi fino a quando lo stato del programma di bilanciamento del carico non diventa **Active**.
2. Apri l'**indirizzo** in un browser web.
3. Aggiorna la pagina diverse volte e nota che il programma di bilanciamento del carico sta provando a raggiungere server diversi a ogni aggiornamento. 
4. **Salva** l'indirizzo per riferimento futuro. 

Se osservi, le richieste non vengono crittografate e viene supportato solo HTTP. Configurerai un certificato SSL e abiliterai HTTPS nella prossima sezione. 

**RIPETI** i passi da 1 a 7 sopra riportati nella **regione 2**.

## Proteggi il traffico all'interno del VPC con HTTPS
{: #secure_https}

Prima di aggiungere un listener HTTPS, devi generare un certificato SSL, verificare l'autenticità del tuo dominio personalizzato, avere un'ubicazione in cui conservare il certificato e associarlo al servizio dell'infrastruttura.

### Esegui il provisioning di un servizio CIS e configura il dominio personalizzato. 

In questa sezione, creerai il servizio IBM Cloud Internet Services (CIS),  configurerai un dominio personalizzato facendo in modo che punti ai server dei nomi CIS e successivamente configurerai un programma di bilanciamento del carico globale.

1. Passa a [Internet Services](https://{DomainName}/catalog/services/internet-services) nel catalogo {{site.data.keyword.Bluemix_notm}}.
2. Imposta il nome servizio e fai clic su **Create** per creare un'istanza del servizio. Per questa esercitazione, puoi utilizzare i piani dei prezzi. 
3. Quando viene eseguito il provisioning dell'istanza del servizio, imposta il tuo nome dominio facendo clic su **Let's get started** e facendo clic su **Add domain**.
4. Fai clic su **Next step**. Quando vengono assegnati i server dei nomi, configura il tuo registrar o il provider del nome del dominio in modo che utilizzi i server dei nomi elencati. 
5. Una volta configurato il tuo registrar o il provider DNS, potresti dover attendere fino a 24 ore affinché le modifiche diventino effettive. 

   Quando lo stato del dominio nella pagina Overview cambia da *Pending* a *Active*, puoi utilizzare il comando `dig <YOUR_DOMAIN_NAME> ns` per verificare che i nuovi server dei nomi siano diventati effettivi.
   {:tip}

Devi ottenere un certificato SSL per il dominio e il dominio secondario che intendi utilizzare con il programma di bilanciamento del carico globale. Presupponendo un dominio come mydomain.com, il programma di bilanciamento del carico globale potrebbe essere ospitato in `lb.mydomain.com`. Il certificato dovrà essere emesso per lb.mydomain.com.

Puoi ottenere certificati SSL gratuiti da [Let's Encrypt](https://letsencrypt.org/). Durante il processo, potresti dover configurare un record DNS di tipo TXT nell'interfaccia DNS di CIS (Cloud Internet Services) per dimostrare che sei il proprietario del dominio.
{:tip}

Una volta ottenuto il certificato SSL e la chiave privata per il tuo dominio, assicurati di convertirli nel formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Per convertire un certificato nel formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
2. Per convertire una chiave privata nel formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Importa il certificato e autorizza il servizio del programma di bilanciamento del carico

Puoi gestire i certificati SSL tramite IBM Certificate Manager.

1. Crea un'istanza [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) in un'ubicazione supportata. 
2. Nel dashboard del servizio, utilizza **Import Certificate**:
   * Imposta **Name** sul dominio secondario e sul dominio personalizzato, ad esempio *lb.mydomain.com*.
   * Individua, sfogliando, il file di certificato (**Certificate file**) in formato PEM.
   * Individua, sfogliando, il file di chiave privata (**Private key file**) in formato PEM.
   * Seleziona **Import**.
3. Crea un'autorizzazione che fornisca all'istanza del servizio del programma di bilanciamento del carico l'accesso all'istanza del gestore certificati che contiene il certificato SSL. Puoi gestire una simile autorizzazione tramite le [Autorizzazioni di identità e accesso](https://{DomainName}/iam#/authorizations). 
  - Fai clic su **Create** e scegli **Infrastructure Service** come servizio di origine
  - **Load Balancer for VPC** come tipo di risorsa
  - **Certificate Manager** come servizio di destinazione
  - Assegna il ruolo di accesso al servizio Scrittore (**Writer**).
  - Per creare un programma di bilanciamento del carico, devi concedere l'autorizzazione All resource instances per l'istanza della risorsa di origine. L'istanza del servizio di destinazione può essere **All instances** oppure può essere la tua specifica istanza della risorsa del gestore certificati. 

### Crea un listener HTTPS

Ora, passa ai [programmi di bilanciamento del carico](https://{DomainName}/vpc/network/loadBalancers)

1. Seleziona **vpc-lb-region1**
2. In **Front-end listeners**, fai clic su **New listener**

   -  **Protocol**: HTTPS
   -  **Port**: 443
   -  **Back-end pool**: POOL nella stessa regione
   -  Scegli il certificato SSL per **lb.YOUR-DOMAIN-NAME**

3. Fai clic su **Create** per configurare un listener HTTPS

**RIPETI** la stessa procedura nel programma di bilanciamento del carico della **regione 2**.

## Configura un programma di bilanciamento del carico globale
{: #global-load-balancer}

In questa sezione, configurerai un programma di bilanciamento globale (global load balancer - GLB) distribuendo il traffico in entrata ai programmi di bilanciamento del carico locali configurati in diverse regioni {{site.data.keyword.Bluemix_notm}}.

### Distribuisci il traffico tra le regioni con un programma di bilanciamento del carico globale
Apri il servizio CIS che hai creato andando all'[elenco delle risorse](https://{DomainName}/resources) nei servizi. 

1. Passa a **Global Load Balancers** in **Reliability** e fai clic su **Create load balancer**.
2. Immetti **lb.YOUR-DOMAIN-NAME** come il tuo nome host e un valore TTL di 60 secondi.
3. Fai clic su **Add pool** per definire un pool di origine predefinito
   - **Name**: lb-region1
   - **Health check**: CREA UN NUOVO CONTROLLO DELL'INTEGRITÀ
     - **Monitor Type**: HTTP
     - **Path**: /
     - **Port**: 80
   - **Health check region**: Eastern North America
   - **origins**
     - **name**: region1
     - **address**: INDIRIZZO DEL PROGRAMMA DI BILANCIAMENTO DEL CARICO LOCALE **REGION1**
     - **weight**: 1
     - Fai clic su **Add**

4. **AGGIUNGI** un ulteriore **pool di origine** che punti al programma di bilanciamento del carico **region2** nella regione **Western Europe** e fai clic su **Provision 1 Resource** per eseguire il provisioning del tuo programma di bilanciamento del carico globale.

Attendi fino a quando lo stato del controllo **dell'integrità** non diventa **Healthy**. Apri il link **lb.YOUR-DOMAIN-NAME** in un browser di tua scelta per vedere il programma di bilanciamento del carico in funzione. 

### Test di failover
Al momento, dovresti aver notato che per la maggior parte del tempo stai provando a raggiungere i server nella **regione 1** poiché le è stato assegnato un peso maggiore rispetto ai server della **regione 2**. Introduciamo un errore del controllo dell'integrità nel pool di origine della **regione 1**,

1. Passa alle [VSI (virtual server instance)](https://{DomainName}/vpc/compute/vs).
2. Fai clic sui **tre punti(...)** accanto ai server in esecuzione nella **zona 1** della **regione 1** e fai clic su **Stop**.
3. **RIPETI** la stessa procedura per i server in esecuzione nella **zona 2** della **regione 1**.
4. Ritorna al GLB nel servizio CIS e attendi fino a quando lo stato dell'integrità non diventa **Critical**.
5. Ora, quando aggiorni l'url del tuo dominio, dovresti sempre provare a raggiungere i server nella **regione 2**.

Non dimenticare di **avviare** i server nella zona 1 e nella zona 2 della regione 1
{:tip}

## Rimuovi le risorse
{: #removeresources}

- Rimuovi il programma di bilanciamento del carico globale, i pool di origine e i controlli dell'integrità nel servizio CIS
- Rimuovi i certificati nel servizio Certificate Manager. 
- Rimuovi i programmi di bilanciamento del carico, le VSI, le sottoreti e i VPC.
- Nell'[elenco delle risorse](https://{DomainName}/resources), elimina i servizi utilizzati in questa esercitazione. 


## Contenuto correlato
{: #related}

* [Utilizzo di Load Balancers in IBM Cloud VPC](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc#--beta-using-load-balancers-in-ibm-cloud-vpc)
