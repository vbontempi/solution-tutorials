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

# Sottoreti private e pubbliche in un VPC (Virtual Private Cloud)
{: #vpc-public-app-private-backend}

IBM accetterà un numero limitato di clienti che parteciperanno a un programma Early Access al VPC a partire dai primi di aprile 2019, con l'utilizzo completo che verrà aperto nei mesi seguenti. Se la tua organizzazione desidera ottenere l'accesso all'IBM Virtual Private Cloud, completa questo [modulo per nomina](https://{DomainName}/vpc){: new_window} e un rappresentante di IBM ti contatterà per spiegarti i passi successivi.
{: important}

Questa esercitazione ti guida nella creazione del tuo {{site.data.keyword.vpc_full}} (VPC) con una sottorete pubblica e una privata e una VSI (virtual server instance) in ciascuna sottorete. Un VPC è il tuo cloud privato su un'infrastruttura cloud condivisa con isolamento logico da altre reti virtuali. 

Una [sottorete](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#subnet) è un intervallo di indirizzi IP. È collegato ad una singola zona e non può estendersi a più zone o regioni. Ai fini del VPC, la caratteristica importante di una sottorete è data dal fatto che le sottoreti possono essere isolate le une dalle altre, come pure essere interconnesse nel modo consueto. L'isolamento della sottorete può essere eseguito dagli ACL ([Access Control Lists](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#access-control-list)) di rete che agiscono come firewall per controllare il flusso dei pacchetti di dati tra le sottoreti. Allo stesso modo, i [Gruppi di sicurezza](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#security-group) (SG) agiscono come firewall virtuali per controllare il flusso dei pacchetti di dati da e verso singole VSI.

La sottorete pubblica viene utilizzata per le risorse che devono essere esposte al mondo esterno. Le risorse con accesso limitato a cui non si deve mai accedere direttamente dal mondo esterno vengono collocate all'interno della sottorete privata. Le istanze in una sottorete di questo tipo potrebbero essere il tuo database di backend oppure alcuni segreti archiviati che non vuoi siano accessibili pubblicamente. Definirai i gruppi di sicurezza per consentire o bloccare il traffico alle VSI.
{:shortdesc}

In breve, utilizzando VPC puoi

- creare una SDN (software-defined network),
- isolare i carichi di lavoro,
- avere un buon controllo del traffico in entrata e in uscita. 

## Obiettivi

{: #objectives}

- Comprendere gli oggetti dell'infrastruttura disponibili per i VPC (virtual private cloud)
- Apprendere informazioni per creare un VPC (virtual private cloud), sottoreti e istanze del server
- Sapere come applicare i gruppi di sicurezza per l'accesso sicuro ai server

## Servizi utilizzati

{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura
{: #architecture}

![Architettura](images/solution40-vpc-public-app-private-backend/Architecture.png)


1. L'amministratore (DevOps) configura l'infrastruttura richiesta (VPC, sottoreti, gruppi di sicurezza con regole, VSI) sul cloud.
2. L'utente di internet effettua una richiesta HTTP/HTTPS al server web nel frontend.
3. Il frontend richiede risorse private dal backend protetto e fornisce i risultati all'utente. 

## Prima di iniziare

{: #prereqs}

- Controlla le autorizzazioni utente. Assicurati che il tuo account utente disponga di autorizzazioni sufficienti per creare e gestire le risorse VPC. Per un elenco di autorizzazioni necessarie, vedi [Concessione delle autorizzazioni necessarie agli utenti VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).

- Hai bisogno di una chiave SSH per collegarti ai server virtuali. Se non hai una chiave SSH, vedi le [istruzioni per creare una chiave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).

## Crea un VPC (Virtual Private Cloud)
{: #create-vpc}

Per creare il tuo {{site.data.keyword.vpc_short}},

1. Passa alla pagina della [panoramica di VPC](https://{DomainName}/vpc/overview) e fai clic su **Create a VPC**.
2. Nella sezione **New virtual private cloud**: 
   * Immetti **vpc-pubpriv** come nome per il tuo VPC.
   * Seleziona un **gruppo di risorse**.
   * Facoltativamente, aggiungi delle **tag** per organizzare le tue risorse.
3. Seleziona **Create new default (Allow all)** come ACL (access control list) predefinito del tuo VPC.
1. Deseleziona SSH ed esegui il ping da **Default security group**.
4. In **New subnet for VPC**:
   * Come nome univoco, immetti **vpc-pubpriv-backend-subnet**.
   * Seleziona un'ubicazione. 
   * Immetti l'intervallo di IP per la sottorete nella notazione CIDR, ad esempio **10.xxx.0.0/24**. Lascia invariato **Address prefix** e seleziona 256 come valore per **Number of addresses**.
5. Seleziona **Use VPC default** per l'ACL (access control list) della tua sottorete. Puoi configurare le regole in entrata e in uscita in un secondo momento. 
6. Fai clic su **Create virtual private cloud** per eseguire il provisioning dell'istanza.

Se le VSI collegate alla sottorete privata devono accedere a Internet per caricare il software, passa il gateway pubblico a **Attached** perché collegando un gateway pubblico consentirai a tutte le risorse collegate di comunicare con l'internet pubblico. Una volta che le VSI hanno tutto il software necessario, riporta il gateway pubblico a **Detached** cosicché la sottorete non possa raggiungere l'internet pubblico.
{: important}

Per confermare la creazione della sottorete, fai clic su **Subnets** nel riquadro di sinistra e aspetta che lo stato cambi in **Available**. Puoi creare una nuova sottorete in **Subnets**.

## Crea un gruppo di sicurezza e una VSI per il backend
{: #backend-subnet-vsi}

In questa sezione, creerai un gruppo di sicurezza e una VSI per il backend.

### Crea un gruppo di sicurezza per il backend

Per impostazione predefinita, viene creato un gruppo di sicurezza insieme al tuo VPC che consente tutto il traffico SSH (porta TCP 22) e il ping (tipo 8 ICMP) alle istanze collegate. 

Per creare un nuovo gruppo di sicurezza per il backend:  
1. Fai clic su **Security groups** in **Network**, poi su **New security group**.  
2. Immetti **vpc-pubpriv-backend-sg** come nome e seleziona il VPC che hai creato in precedenza.   
3. Fai clic su **Create security group**.

Successivamente, modificherai il gruppo di sicurezza per aggiungere le regole in entrata e in uscita. 

### Crea una VSI (virtual server instance) per il backend

Per creare una VSI (virtual server instance) nella sottorete appena creata:

1. Fai clic sulla sottorete di backend in **Subnets**.
2. Fai clic su **Attached instances**, poi su **New instance**.
3. Immetti un nome univoco e seleziona **vpc-pubpriv-backend-vsi**. Quindi, seleziona il VPC che hai creato prima e l'**ubicazione** come hai fatto in precedenza. 
4. Scegli l'immagine **Ubuntu Linux**, fai clic su **All profiles** e in **Compute**, scegli **c-2x4** con 2vCPU e 4 GB di RAM.
5. Per **SSH keys**, seleziona la chiave SSH che hai creato in precedenza. 
6. In **Network interfaces**, fai clic sull'icona **Edit** accanto a Security Groups
   * Seleziona **vpc-pubpriv-backend-subnet** come sottorete. 
   * Deseleziona il gruppo di sicurezza predefinito e seleziona **vpc-pubpriv-backend-sg** come attivo. 
   * Fai clic su **Save**.
7. Fai clic su **Create virtual server instance**.

## Crea una sottorete, un gruppo di sicurezza e una VSI per il frontend
{: #frontend-subnet-vsi}

Analogamente al backend, creerai una sottorete di frontend con VSI (virtual server instance) e un gruppo di sicurezza. 

### Crea una sottorete per il frontend

Per creare una nuova sottorete per il frontend,

1. Fai clic su **Subnets** in **Network** nel riquadro di sinistra > **New subnet**.
   * Immetti **vpc-pubpriv-frontend-subnet** come nome, quindi seleziona il VPC che hai creato. 
   * Seleziona un'ubicazione. 
   * Immetti l'intervallo di IP per la sottorete nella notazione CIDR, ad esempio **10.xxx.1.0/24**. Lascia invariato **Address prefix** e seleziona 256 come valore per **Number of addresses**.
1. Seleziona **VPC default** per l'ACL (access control list) della tua sottorete. Puoi configurare le regole in entrata e in uscita in un secondo momento. 
1. Poiché tutte le VSI nella sottorete avranno un IP mobile collegato, non è necessario abilitare un gateway pubblico per la sottorete. Le VSI avranno la connettività Internet tramite i loro IP mobili. 
1. Fai clic su **Create subnet** per eseguirne il provisioning.

### Crea un gruppo di sicurezza per il frontend

Per creare un nuovo gruppo di sicurezza per il frontend:
1. Fai clic su **Security groups** in Network, poi su **New security group**.
2. Immetti **vpc-pubpriv-frontend-sg** come nome e seleziona il VPC che hai creato in precedenza. 
3. Fai clic su **Create security group**.

### Crea una VSI (virtual server instance) per il frontend

Per creare una VSI (virtual server instance) nella sottorete appena creata:

1. Fai clic sulla sottorete di frontend in **Subnets**.
2. Fai clic su **Attached instances**, poi su **New instance**.
3. Immetti un nome univoco, **vpc-pubpriv-frontend-vsi**, seleziona il VPC che hai creato prima, quindi la stessa **ubicazione** come hai fatto in precedenza. 
4. Seleziona l'immagine **Ubuntu Linux**, fai clic su **All profiles** e, in **Compute**, scegli **c-2x4** con 2vCPU e 4 GB di RAM
5. Per **SSH keys**, seleziona la chiave SSH che hai creato in precedenza. 
6. In **Network interfaces**, fai clic sull'icona **Edit** accanto a Security Groups
   * Seleziona **vpc-pubpriv-frontend-subnet** come sottorete. 
   * Deseleziona il gruppo di sicurezza predefinito e attiva **vpc-pubpriv-frontend-sg**.
   * Fai clic su **Save**.
   * Fai clic su **Create virtual server instance**.
7. Attendi fino a quando lo stato della VSI non diventa **Powered On**. Quindi, seleziona la VSI di frontend **vpc-pubpriv-frontend-vsi**, scorri fino a **Network Interfaces** e fai clic su **Reserve** in **Floating IP** per associare un indirizzo IP pubblico alla tua VSI di frontend. Salva l'indirizzo IP associato negli appunti per riferimento futuro. 

## Configura la connettività tra il frontend e il backend
{: #setup-connectivity-frontend-backend}

Con tutti i server implementati, in questa sezione configurerai la connettività per consentire le normali operazioni tra i server di frontend e di backend.

### Configura il gruppo di sicurezza per il frontend

1. Passa a **Security groups** nella sezione **Network**, poi fai clic su **vpc-pubpriv-frontend-sg**.
2. Innanzitutto, aggiungi le seguenti regole **in entrata** utilizzando **Add rule**. Consentono le richieste HTTP in entrata e il Ping (ICMP).

	<table>
   <thead>
      <tr>
         <td><strong>Origine</strong></td>
         <td><strong>Protocollo</strong></td>
         <td><strong>Valore</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Qualsiasi - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>Da: <strong>80</strong> a <strong>80</strong></td>
      </tr>
      <tr>
         <td>Qualsiasi - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>Da: <strong>443</strong> a <strong>443</strong></td>
      </tr>
      <tr>
         <td>Qualsiasi - 0.0.0.0/0 </td>
	      <td>ICMP</td>
	      <td>Tipo: <strong>8</strong>,Codice: <strong>Lasciare vuoto</strong></td>
      </tr>
   </tbody>
   </table>

3. Aggiungi quindi queste regole **in uscita**.

   <table>
   <thead>
      <tr>
         <td><strong>Destinazione</strong></td>
         <td><strong>Protocollo</strong></td>
         <td><strong>Valore</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Tipo: <strong>Gruppo di sicurezza</strong> - Nome: <strong>vpc-pubpriv-backend-sg</strong></td>
         <td>TCP</td>
         <td>Porta del server di backend, vedi suggerimento</td>
      </tr>
   </tbody>
   </table>

Queste sono le porte per i tipici servizi di backend. MySQL sta utilizzando la porta 3306, PostgreSQL la porta 5432. Si accede a Db2 sulla porta 50000 o 50001. Microsoft SQL Server utilizza la porta 1433 per impostazione predefinita. Uno dei molti [elenchi con le porte comuni si trova su Wikipedia](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers).
{:tip }

### Configura il gruppo di sicurezza per il backend
Analogamente al frontend, configura il gruppo di sicurezza per il backend.

1. Passa a **Security groups** nella sezione **Network**, poi fai clic su **vpc-pubpriv-backend-sg**.
2. Aggiungi la seguente regola **in entrata** utilizzando **Add rule**. Consente una connessione al servizio di backend.

   <table>
   <thead>
      <tr>
         <td><strong>Origine</strong></td>
         <td><strong>Protocollo</strong></td>
         <td><strong>Valore</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Tipo: <strong>Gruppo di sicurezza</strong> - Nome: <strong>vpc-pubpriv-frontend-sg</strong></td>
         <td>TCP</td>
         <td>Porta del server di backend</td>
      </tr>
   </tbody>
   </table>


## Installa il software ed esegui le attività di manutenzione
{: #install-software-maintenance-tasks}

Segui i passi menzionati in [Accedi in modo sicuro alle istanze remote con un host bastion](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) per la manutenzione sicura dei server utilizzando un host bastion che funge da server `jump` e un gruppo di sicurezza di manutenzione. 

## Rimuovi le risorse
{: #remove-resources}

1. Nella console di gestione VPC, fai clic su **Floating IPs**, poi sull'indirizzo IP per le tue VSI, quindi, nel menu delle azioni, seleziona **Release**. Conferma che vuoi rilasciare l'indirizzo IP. 
2. Passa quindi a **Virtual server instances** ed **elimina** le tue istanze. Le istanze verranno eliminate e il loro stato rimarrà per un po' in **Deleting**. Assicurati di aggiornare il tuo browser di tanto in tanto. 
3. Una volta terminato con le VSI, passa a **Subnets**. Se la sottorete ha un gateway pubblico collegato, fai clic sul nome della sottorete. Nei dettagli della sottorete, scollega il gateway pubblico. Le sottoreti senza gateway pubblico possono essere eliminate dalla pagina della panoramica. Elimina le tue sottoreti. 
4. Una volta eliminate le sottoreti, passa alla scheda **VPC** ed elimina il tuo VPC.

Quando utilizzi la console, potresti dover aggiornare il tuo browser per vedere le informazioni sullo stato aggiornate dopo l'eliminazione di una risorsa.
{:tip}

## Espandi l'esercitazione
{: #expand-tutorial}

Vuoi aggiungere qualcosa o estendere questa esercitazione? Ecco alcune idee:

- Aggiungi un [programma di bilanciamento del carico](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-load-balancer) per distribuire il traffico in entrata tra più istanze. 
- Crea una VPN ([virtual private network](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn)) in modo che il tuo VPC possa connettersi in modo sicuro a un'altra rete privata, ad esempio una rete in loco o un altro VPC.


## Contenuto correlato
{: #related}

- [Glossario VPC](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)
- [VPC utilizzando la CLI IBM Cloud](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli#creating-a-vpc-using-the-ibm-cloud-cli)
- [VPC utilizzando le API REST](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis#creating-a-vpc-using-the-rest-apis)
