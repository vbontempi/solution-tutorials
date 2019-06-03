---
copyright:
  years: 2019
lastupdated: "2019-05-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}
{:important: .important}

# Utilizza un gateway VPC/VPN per l'accesso in loco privato e sicuro alle risorse cloud
{: #vpc-site2site-vpn}

IBM accetterà un numero limitato di clienti che parteciperanno a un programma Early Access al VPC a partire dai primi di aprile 2019, con l'utilizzo completo che verrà aperto nei mesi seguenti. Se la tua organizzazione desidera ottenere l'accesso all'IBM Virtual Private Cloud, completa questo [modulo per nomina](https://{DomainName}/vpc){: new_window} e un rappresentante di IBM ti contatterà per spiegarti i passi successivi.
{: important}

IBM offre diversi modi per estendere in modo sicuro una rete di computer in loco con le risorse nel cloud IBM. Ciò ti consente di usufruire dell'elasticità del provisioning dei server quando hai bisogno di essi e di rimuoverli quando non sono più necessari. Inoltre, puoi connettere in modo facile e sicuro le tue capacità in loco ai servizi {{site.data.keyword.cloud_notm}}.

Questa esercitazione ti guida nella connessione di un gateway VPN (Virtual Private Network) in loco a una VPN cloud creata all'interno di un VPC (un gateway VPC/VPN). Innanzitutto, creerai un nuovo {{site.data.keyword.vpc_full}} (VPC) e le risorse associate come sottoreti, ACL (Access Control List) di rete, gruppi di sicurezza e VSI (Virtual Server Instance).
Il gateway VPC/VPN stabilirà un link site-to-site [IPsec](https://en.wikipedia.org/wiki/IPsec) a un gateway VPN in loco. I protocolli IPsec e [Internet Key Exchange](https://en.wikipedia.org/wiki/Internet_Key_Exchange), IKE, sono standard aperti comprovati per la comunicazione sicura.

Per dimostrare ulteriormente l'accesso sicuro e privato, distribuirai un microservizio su una VSI per accedere a {{site.data.keyword.cos_short}} (COS), che rappresenta un'applicazione LOB (line of business).
Il servizio COS ha un endpoint diretto che può essere utilizzato per entrata/uscita privata senza costi quando tutto l'accesso è all'interno della stessa regione di {{site.data.keyword.cloud_notm}}. Un computer in loco accederà al microservizio COS. Tutto il traffico fluirà tramite la VPN e quindi privatamente attraverso {{site.data.keyword.cloud_notm}}.

Sono disponibili molte soluzioni VPN in loco popolari per i gateway site-to-site. Questa esercitazione utilizza il gateway VPN [strongSwan](https://www.strongswan.org/) per connettersi con il gateway VPC/VPN. Per simulare un data center in loco, installerai il gateway strongSwan su una VSI in {{site.data.keyword.cloud_notm}}.

{:shortdesc}
In breve, utilizzando un VPC puoi

- connettere i tuoi sistemi in loco ai servizi e ai carichi di lavoro in esecuzione in {{site.data.keyword.cloud_notm}},
- assicurarti una connettività privata e a basso costo a COS,
- connettere i tuoi sistemi basati su cloud ai servizi e ai carichi di lavoro in esecuzione in loco. 

## Obiettivi
{: #objectives}

* Accedere a un ambiente VPC (virtual private cloud) da un data center o da un cloud privato (virtuale) in loco. 
* Raggiungere in modo sicuro i servizi cloud utilizzando endpoint di servizio privati.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. Sebbene in questa esercitazione non ci siano addebiti di rete per l'accesso a COS dal microservizio, incorrerai in addebiti di rete standard per l'accesso al VPC.

## Architettura
{: #architecture}

Il seguente diagramma mostra il VPC che contiene un server delle applicazioni. Il server delle applicazioni ospita un microservizio che si interfaccia con il servizio {{site.data.keyword.cos_short}}. Una rete in loco (simulata) e l'ambiente cloud virtuale sono connessi tramite i gateway VPN.

![Architettura](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. L'infrastruttura (VPC, sottoreti, gruppi di sicurezza con regole, ACL di rete e VSI) viene configurata utilizzando uno script fornito.
2. Il microservizio si interfaccia con {{site.data.keyword.cos_short}} attraverso un endpoint diretto.
3. Viene eseguito il provisioning di un gateway VPC/VPN per esporre l'ambiente VPC nella rete in loco. 
4. Il software del gateway IPsec open source Strongswan viene utilizzato in loco per stabilire la connessione VPN con l'ambiente cloud.

## Prima di iniziare
{: #prereqs}

- Installa tutti gli strumenti CLI necessari [seguendo questi passi](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview). Hai bisogno del plugin dell'infrastruttura CLI facoltativo. 
- Accedi a {{site.data.keyword.cloud_notm}} tramite la riga di comando. Per i dettagli, vedi [Introduzione alla CLI](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli).
- Controlla le autorizzazioni utente. Assicurati che il tuo account utente disponga di autorizzazioni sufficienti per creare e gestire le risorse VPC. Per un elenco di autorizzazioni necessarie, vedi [Concessione delle autorizzazioni necessarie agli utenti VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).
- Hai bisogno di una chiave SSH per collegarti ai server virtuali. Se non hai una chiave SSH, vedi le [istruzioni per creare una chiave](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- Installa [**jq**](https://stedolan.github.io/jq/download/). Viene utilizzato dagli script forniti per elaborare l'output JSON. 

## Distribuisci un server delle applicazioni virtuale in un VPC (virtual private cloud)
Nella seguente sezione, scaricherai gli script per configurare un ambiente VPC baseline e il codice per un microservizio per interfacciarti con {{site.data.keyword.cos_short}}. Dopodiché, eseguirai il provisioning del servizio {{site.data.keyword.cos_short}} e configurerai la baseline.

### Ottieni il codice
{: #setup}
L'esercitazione utilizza gli script per distribuire una baseline delle risorse dell'infrastruttura prima di creare i gateway VPN. Questi script e il codice per il microservizio sono disponibili in un repository GitHub.

1. Ottieni il codice dell'applicazione: 
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. Vai agli script per questa esercitazione andando in **vpc-tutorials**, poi **vpc-site2site-vpn**:
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### Crea i servizi
In questa sezione, accederai a {{site.data.keyword.cloud_notm}} sulla CLI e creerai un'istanza di {{site.data.keyword.cos_short}}.

1. Verifica di aver seguito i passi prerequisiti di accesso
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. Crea un'istanza di [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) utilizzando un piano **standard** o **lite**.
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   Tieni presente che puoi creare solo un'istanza lite per account. Se hai già un'istanza di {{site.data.keyword.cos_short}}, puoi riutilizzarla.
   {: tip}

3. Crea una chiave di servizio con il ruolo **Reader** (Lettore):
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. Ottieni i dettagli della chiave di servizio nel formato JSON e archiviali in un nuovo file **credentials.json** nella sottodirectory **vpc-app-cos**. Il file verrà utilizzato in seguito dall'applicazione. 
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### Crea le risorse baseline di VPC (Virtual Private Cloud)
{: #create-vpc}
L'esercitazione fornisce uno script per creare le risorse baseline necessarie per questa esercitazione, ad esempio l'ambiente di partenza. Lo script può generare tale ambiente in un VPC esistente o creare un nuovo VPC.

Nella seguente sezione, creerai queste risorse configurando e poi eseguendo uno script di configurazione. Lo script incorpora la configurazione di un host bastion trattata in [Accedi in modo sicuro alle istanze remote con un host bastion](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server).

1. Copia il file di configurazione di esempio in un file da utilizzare:

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. Modifica il file **config.sh** e adatta le impostazioni al tuo ambiente. Devi modificare il valore **SSHKEYNAME** con il nome o un elenco separato da virgole di nomi delle chiavi SSH (vedi "Prima di iniziare"). Modifica le diverse impostazioni **ZONE** in modo che corrispondano alla tua regione cloud. Tutte le altre variabili possono rimanere invariate. 
3. Per creare le risorse in un nuovo VPC, esegui lo script nel seguente modo: 

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   Per riutilizzare un VPC esistente, passa il suo nome allo script in questo modo. Sostituisci **YOUR_EXISTING_VPC** con il nome VPC reale.
   ```sh
   REUSE_VPC=YOUR_EXISTING_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. Ciò creerà le seguenti risorse, incluse le risorse correlate a bastion: 
   - 1 VPC (facoltativo)
   - 1 gateway pubblico
   - 3 sottoreti all'interno del VPC
   - 4 gruppi di sicurezza con regole in entrata e in uscita
   - 3 VSI: vpns2s-onprem-vsi (l'ip mobile è ONPREM_IP), vpns2s-cloud-vsi (l'ip mobile è VSI_CLOUD_IP) e vpns2s-bastion (l'ip mobile è BASTION_IP_ADDRESS)

   Prendi nota dei valori restituiti per **BASTION_IP_ADDRESS**, **VSI_CLOUD_IP**, **ONPREM_IP**, **CLOUD_CIDR** e **ONPREM_CIDR** per utilizzarli in un secondo momento. L'output viene anche archiviato nel file **network_config.sh**. Il file può essere utilizzato per la configurazione automatizzata. 

### Crea il gateway VPN (Virtual Private Network) e la connessione
Nella seguente sezione, aggiungerai un gateway VPN e una connessione associata alla sottorete con la VSI dell'applicazione. 

1. Passa alla pagina della [panoramica VPC](https://{DomainName}/vpc/overview), poi fai clic su **VPNs** nella scheda di navigazione e su **New VPN gateway** nella finestra di dialogo. Nel modulo **New VPN gateway for VPC**, immetti **vpns2s-gateway** come nome. Assicurati che vengano selezionati il VPC, il gruppo di risorse e **vpns2s-cloud-subnet** corretti come sottorete. 
2. Lascia attivato **New VPN connection for VPC**. Immetti **vpns2s-gateway-conn** come nome. 
3. Per **Peer gateway address**, utilizza l'indirizzo IP mobile di **vpns2s-onprem-vsi** (ONPREM_IP). Immetti **20_PRESHARED_KEY_KEEP_SECRET_19** come **Preshared key**.
4. Per **Local subnets**, utilizza le informazioni fornite per **CLOUD_CIDR**, per **Peer subnets** quelle fornite per **ONPREM_CIDR**.
5. Lascia invariate le impostazioni in **Dead peer detection**. Fai clic su **Create VPN gateway** per creare il gateway e una connessione associata.
6. Attendi che il gateway VPN diventi disponibile (potresti dover aggiornare la schermata).
7. Annota l'indirizzo **IP gateway** assegnato come **GW_CLOUD_IP**. 

### Crea il gateway VPN (Virtual Private Network) in loco
Successivamente, creerai il gateway VPN sull'altro sito, nell'ambiente in loco simulato. Utilizzerai il software IPsec basato su open source [strongSwan](https://strongswan.org/).

1. Connettiti alla VSI in loco **vpns2s-onprem-vsi** utilizzando ssh. Esegui quanto riportato di seguito e sostituisci **ONPREM_IP** con l'indirizzo IP restituito in precedenza. 

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   A seconda del tuo ambiente, potresti dover utilizzare `ssh -i <path to your private key file> root@ONPREMP_IP`.
   {:tip}

2. Poi, sulla macchina **vpns2s-onprem-vsi**, esegui i seguenti comandi per aggiornare il gestore pacchetti e per installare il software strongSwan.

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. Configura il file **/etc/sysctl.conf** aggiungendo tre righe alla sua fine. Copia quanto segue ed eseguilo: 

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. Modifica quindi il file **/etc/ipsec.secrets**. Aggiungi la seguente riga per configurare gli indirizzi IP di origine e di destinazione e la chiave precondivisa configurata in precedenza. Sostituisci **ONPREM_IP** con il valore conosciuto dell'ip mobile di vpns2s-onprem-vsi. Sostituisci **GW_CLOUD_IP** con l'indirizzo ip conosciuto del gateway VPN VPC. 

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. L'ultimo file che devi configurare è **/etc/ipsec.conf**. Aggiungi il seguente blocco di codice alla fine di tale file. Sostituisci **ONPREM_IP**, **ONPREM_CIDR**, **GW_CLOUD_IP** e **CLOUD_CIDR** con i rispettivi valori conosciuti. 

   ```sh
   # basic configuration
   config setup
      charondebug="all"
      uniqueids=yes
      strictcrlpolicy=no
   
   # connection to vpc/vpn datacenter 
   # left=onprem / right=vpc
   conn tutorial-site2site-onprem-to-cloud
      authby=secret
      left=%defaultroute
      leftid=ONPREM_IP
      leftsubnet=ONPREM_CIDR
      right=GW_CLOUD_IP
      rightsubnet=CLOUD_CIDR
      ike=aes256-sha2_256-modp1024!
      esp=aes256-sha2_256!
      keyingtries=0
      ikelifetime=1h
      lifetime=8h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start
   ```
   {:pre}

6. Riavvia il gateway VPN, quindi controllane lo stato eseguendo: ipsec restart

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   Deve riportare che è stata stabilita la connessione. Tieni aperta la connessione terminale e ssh a questa macchina. 

## Verifica la connettività
Puoi verificare la connessione VPN site-to-site utilizzando SSH o distribuendo il microservizio che si interfaccia con {{site.data.keyword.cos_short}}.

### Verifica utilizzando ssh
Per verificare che la connessione VPN sia stata stabilita correttamente, utilizza l'ambiente in loco simulato come proxy per accedere al server delle applicazioni basato sul cloud.  

1. In un nuovo terminale, esegui il seguente comando dopo aver sostituito i valori. Utilizza l'host strongSwan come host jump per la connessione tramite VPN all'indirizzo IP privato del server delle applicazioni. 

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. Una volta connesso correttamente, chiudi la connessione ssh.

3. Nel terminale della VSI in loco, arresta il gateway VPN:
   ```sh
   ipsec stop
   ```
   {:pre}
4. Nella finestra comandi del passo 1, prova a stabilire di nuovo la connessione: 

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
   Il comando deve avere esito negativo perché la connessione VPN non è attiva e quindi non ci sono link diretti tra l'ambiente cloud e quello simulato in loco. 

   Tieni presente che, a seconda dei dettagli di distribuzione, questa connessione ha effettivamente ancora esito positivo. Il motivo è che la connettività intra-VPC è supportata tra le zone. Se distribuisci la VSI in loco simulata in un altro VPC o in [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers), sarà necessaria la VPN per riuscire ad eseguire correttamente l'accesso.
   {:tip}
   
5. Nel terminale della VSI in loco, avvia di nuovo il gateway VPN: 
   ```sh
   ipsec start
   ```
   {:pre}
 

### Verifica utilizzando un microservizio
Puoi verificare la connessione VPN funzionante accedendo a un microservizio sulla VSI cloud dalla VSI in loco.

1. Copia il codice per l'applicazione del microservizio dalla tua macchina locale alla VSI cloud. Il comando utilizza il bastion come host jump alla VSI cloud. Sostituisci **BASTION_IP_ADDRESS** e **VSI_CLOUD_IP** di conseguenza. 
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. Connettiti alla VSI cloud utilizzando di nuovo il bastion come host jump.
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. Nella VSI cloud, passa alla directory del codice:
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. Installa Python e il PIP del gestore pacchetti Python. 
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. Installa i pacchetti Python necessari utilizzando **pip**.
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. Avvia l'applicazione:
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. Nel terminale della VSI in loco, accedi al servizio. Sostituisci VSI_CLOUD_IP di conseguenza.
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
   Il comando deve restituire un oggetto JSON.

## Rimuovi le risorse
{: #remove-resources}

1. Nella console di gestione VPC, fai clic su **VPNs**. Nel menu delle azioni sul gateway VPN, seleziona **Delete** per rimuovere il gateway.
2. Poi, fai clic su **Floating IPs** nella navigazione, poi sull'indirizzo IP delle tue VSI. Nel menu delle azioni, seleziona **Release**. Conferma che vuoi rilasciare l'indirizzo IP. 
3. Passa quindi a **Virtual server instances** ed **elimina** le tue istanze. Le istanze verranno eliminate e il loro stato rimarrà per un po' in **Deleting**. Assicurati di aggiornare il tuo browser di tanto in tanto. 
4. Una volta terminato con le VSI, passa a **Subnets**. Se la sottorete ha un gateway pubblico collegato, fai clic sul nome della sottorete. Nei dettagli della sottorete, scollega il gateway pubblico. Le sottoreti senza gateway pubblico possono essere eliminate dalla pagina della panoramica. Elimina le tue sottoreti. 
5. Una volta eliminate le sottoreti, passa a **VPC** ed elimina il tuo VPC.

Quando utilizzi la console, potresti dover aggiornare il tuo browser per vedere le informazioni sullo stato aggiornate dopo l'eliminazione di una risorsa.
{:tip}

## Espandi l'esercitazione 
{: #expand-tutorial}

Vuoi aggiungere qualcosa o estendere questa esercitazione? Ecco alcune idee:

- Aggiungi un [programma di bilanciamento del carico](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc) per distribuire il traffico del microservizio in entrata tra più istanze. 
- Distribuisci l'[applicazione in un server pubblico, i tuoi dati e servizi in un host privato](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend).


## Contenuto correlato
{: #related}

- [Glossario VPC](/docs/vpc?topic=vpc-vpc-glossary)
- [IBM Cloud CLI plugin for VPC Reference](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [VPC utilizzando le API REST](/docs/infrastructure/vpc/example-code.html)
- Esercitazione della soluzione: [Accedi in modo sicuro alle istanze remote con un host bastion](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
