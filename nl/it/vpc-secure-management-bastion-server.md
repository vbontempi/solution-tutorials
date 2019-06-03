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

# Accedi in modo sicuro alle istanze remote con un host bastion
{: #vpc-secure-management-bastion-server}

IBM accetterà un numero limitato di clienti che parteciperanno a un programma Early Access al VPC a partire dai primi di aprile 2019, con l'utilizzo completo che verrà aperto nei mesi seguenti. Se la tua organizzazione desidera ottenere l'accesso all'IBM Virtual Private Cloud, completa questo [modulo per nomina](https://{DomainName}/vpc){: new_window} e un rappresentante di IBM ti contatterà per spiegarti i passi successivi.
{: important}

Questa esercitazione ti guida nella distribuzione di un host bastion per accedere in modo sicuro alle istanze remote all'interno di un VPC (virtual private cloud). L'host bastion è un'istanza di cui è stato eseguito il provisioning in una sottorete pubblica ed è possibile accedervi tramite SSH. Una volta configurato, l'host bastion agisce come un server **jump** consentendo una connessione sicura alle istanze di cui è stato eseguito il provisioning in una sottorete privata.

Per ridurre l'esposizione dei server all'interno del VPC, creerai e utilizzerai un host bastion. Le attività amministrative sui singoli server vengono eseguite utilizzando SSH, tramite proxy attraverso bastion. L'accesso ai server e l'accesso a internet regolare dai server, ad esempio per l'installazione software, sarà consentito solo con un gruppo di sicurezza di manutenzione speciale collegato a tali server.
{:shortdesc}

## Obiettivi
{: #objectives}

* Apprendere come configurare un host bastion e i gruppi di sicurezza con regole
* Gestire in modo sicuro i server tramite l'host bastion

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:  

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)  
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto. 

## Architettura
{: #architecture}

  ![Architettura](images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png)

1. Una volta configurata l'infrastruttura richiesta (sottoreti, gruppi di sicurezza con regole, VSI) sul cloud, l'amministratore (DevOps) si connette (SSH) all'host bastion utilizzando la chiave SSH privata.
2. L'amministratore assegna un gruppo di sicurezza di manutenzione con le regole in uscita appropriate. 
3. L'amministratore si connette in modo sicuro (SSH) all'indirizzo IP privato dell'istanza tramite l'host bastion per installare o aggiornare il software necessario, ad esempio un server web
4. L'utente di internet effettua una richiesta HTTP/HTTPS al server web.

## Prima di iniziare
{: #prereqs}

- Controlla le autorizzazioni utente. Assicurati che il tuo account utente disponga di autorizzazioni sufficienti per creare e gestire le risorse VPC. Per un elenco di autorizzazioni necessarie, vedi [Concessione delle autorizzazioni necessarie agli utenti VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Hai bisogno di una chiave SSH per collegarti ai server virtuali. Se non hai una chiave SSH, vedi le [istruzioni per creare una chiave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- L'esercitazione presuppone che tu stia aggiungendo l'host bastion a un [VPC (virtual private cloud)](https://{DomainName}/vpc/network/vpcs) esistente. **Se non hai un VPC (virtual private cloud) nel tuo account, creane uno prima di procedere con i prossimi passi.**

## Crea un host bastion
{: #create-bastion-host}

In questa sezione, creerai e configurerai un host bastion insieme a un gruppo di sicurezza in una sottorete separata. 

### Crea una sottorete
{: #create-bastion-subnet}

1. Fai clic su **Subnets** in **Network** nel riquadro di sinistra, quindi su **New subnet**.  
   * Immetti **vpc-secure-bastion-subnet** come nome, quindi seleziona il VPC che hai creato.   
   * Seleziona un'ubicazione e una zona.   
   * Immetti l'intervallo di IP per la sottorete nella notazione CIDR, ad esempio **10.xxx.0.0/24**. Lascia invariato **Address prefix** e seleziona 256 come valore per **Number of addresses**.
1. Seleziona **VPC default** per l'ACL (access control list) della tua sottorete. Puoi configurare le regole in entrata e in uscita in un secondo momento. 
1. Commuta **Public gateway** in **Attached**. 
1. Fai clic su **Create subnet** per eseguirne il provisioning.

### Crea e configura il gruppo di sicurezza bastion

Creiamo un gruppo di sicurezza e configuriamo le regole in entrata nella tua VSI bastion.

1. Passa a **Security groups** e fai clic su **New security group**. Immetti **vpc-secure-bastion-sg** come nome e seleziona il tuo VPC. 
2. Ora, crea le seguenti regole in entrata facendo clic su **Add rule** nella sezione in entrata. Consentono l'accesso SSH e il Ping (ICMP).
 
	**Regola in entrata:**
	<table>
	   <thead>
	      <tr>
	         <td><strong>Origine</strong></td>
	         <td><strong>Protocollo</strong></td>
	         <td><strong>Valore</strong></td>
	      </tr>
	   <tbody>
	      <tr>
	         <td>Qualsiasi - 0.0.0.0/0 </td>
	         <td>TCP</td>
	         <td>Da: <strong>22</strong> a <strong>22</strong></td>
	      </tr>
         <tr>
            <td>Qualsiasi - 0.0.0.0/0 </td>
	         <td>ICMP</td>
	         <td>Tipo: <strong>8</strong>,Codice: <strong>Lasciare vuoto</strong></td>
         </tr>
	   </tbody>
	</table>

   Per migliorare ulteriormente la sicurezza, il traffico in entrata potrebbe essere limitato alla rete aziendale o a una tipica rete domestica. Puoi eseguire `curl ipecho.net/plain ; echo` per ottenere l'indirizzo IP esterno della tua rete e utilizzare invece tale indirizzo.
   {:tip }

### Crea un'istanza bastion
Con la sottorete e il gruppo di sicurezza già implementati, quindi, crea la VSI (virtual server instance) bastion.

1. In **Subnets** nel riquadro di sinistra, seleziona **vpc-secure-bastion-subnet**.
2. Fai clic su **Attached instances** ed esegui il provisioning di una **nuova istanza** denominata **vpc-secure-vsi** nel tuo VPC. Seleziona Ubuntu Linux come tua immagine e **c-2x4** (2 vCPU e 4 GB di RAM) come tuo profilo.
3. Seleziona un'**ubicazione** e assicurati, in seguito, di utilizzare di nuovo la stessa ubicazione. 
4. Per creare una nuova **chiave SSH**, fai clic su **New key**
   * Immetti **vpc-ssh-key** come nome chiave.
   * Lascia invariato **Region**.
   * Copia il contenuto della chiave SSH locale esistente e incollalo in **Public key**.  
   * Fai clic su **Add SSH key**.
5. In **Network interfaces**, fai clic sull'icona **Edit** accanto a Security Groups 
   * Assicurati che **vpc-secure-subnet** sia selezionato come sottorete. 
   * Deseleziona il gruppo di sicurezza predefinito e seleziona **vpc-secure-bastion-sg**.
   * Fai clic su **Save**.
6. Fai clic su **Create virtual server instance**.
7. Una volta accesa l'istanza, fai clic su **vpc-secure-bastion-vsi** e **riserva** un IP mobile.

### Verifica il tuo bastion

Una volta che l'indirizzo IP mobile del tuo bastion è attivo, prova a connetterlo utilizzando **ssh**:

   ```sh
   ssh -i ~/.ssh/<PRIVATE_KEY> root@<BASTION_FLOATING_IP_ADDRESS>
   ```
   {:pre}

## Configura un gruppo di sicurezza con regole di accesso di manutenzione
{: #maintenance-security-group}

Con l'accesso al bastion funzionante, prosegui e crea il gruppo di sicurezza per le attività di manutenzione come l'installazione e l'aggiornamento del software. 

1. Passa a **Security groups** ed esegui il provisioning di un nuovo gruppo di sicurezza denominato **vpc-secure-maintenance-sg** con le regole in uscita riportate di seguito

   <table>
   <thead>
      <tr>
         <td><strong>Destinazione</strong></td>
         <td><strong>Protocollo</strong></td>
         <td><strong>Valore</strong> </td>
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
         <td>TCP</td>
         <td>Da: <strong>53</strong> a <strong>53</strong></td>
      </tr>
      <tr>
         <td>Qualsiasi - 0.0.0.0/0 </td>
         <td>UDP</td>
         <td>Da: <strong>53</strong> a <strong>53</strong></td>
      </tr>
   </tbody>
   </table>

   Le richieste del server DNS vengono indirizzate alla porta 53. DNS utilizza TCP per il trasferimento di zona e UDP per le query dei nomi regolari (primarie) o inverse. Le richieste HTTP vengono effettuate sulla porta 80 e 443.
   {:tip }

2. Aggiungi quindi questa regola **in entrata** che consente l'accesso SSH dall'host bastion.

   <table>
	   <thead>
	      <tr>
	         <td><strong>Origine</strong></td>
	         <td><strong>Protocollo</strong></td>
	         <td><strong>Valore</strong> </td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>Tipo: <strong>Gruppo di sicurezza</strong> - Nome: <strong>vpc-secure-bastion-sg</strong></td>
	         <td>TCP</td>
	         <td>Da: <strong>22</strong> a <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>

3. Crea il gruppo di sicurezza.
4. Passa a **All Security Groups for VPC**, quindi seleziona **vpc-secure-sg**.
5. Infine, modifica il gruppo di sicurezza e aggiungi la seguente regola **in uscita**.

   <table>
	   <thead>
	      <tr>
	         <td><strong>Destinazione</strong></td>
	         <td><strong>Protocollo</strong></td>
	         <td><strong>Valore</strong> </td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>Tipo: <strong>Gruppo di sicurezza</strong> - Nome: <strong>vpc-secure-maintenance-sg</strong></td>
	         <td>TCP</td>
	         <td>Da: <strong>22</strong> a <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>


## Utilizza l'host bastion per accedere alle altre istanze nel VPC
{: #bastion-host-access-instances}

In questa sezione, creerai una sottorete privata con la VSI (virtual server instance) e un gruppo di sicurezza. Per impostazione predefinita, qualsiasi sottorete creata in un VPC è privata.

Se disponi già di VSI (virtual server instance) nel tuo VPC a cui vuoi connetterti, puoi ignorare le prossime tre sezioni e iniziare ad [aggiungere le tue VSI (virtual server instance) al gruppo di sicurezza di manutenzione](#add-vsi-to-maintenance).

### Crea una sottorete
{: #create-private-subnet}

Per creare una nuova sottorete,

1. Fai clic su **Subnets** in **Network** nel riquadro di sinistra, quindi su **New subnet**.  
   * Immetti **vpc-secure-private-subnet** come nome, quindi seleziona il VPC che hai creato.   
   * Seleziona un'ubicazione.   
   * Immetti l'intervallo di IP per la sottorete nella notazione CIDR, ad esempio **10.xxx.1.0/24**. Lascia invariato **Address prefix** e seleziona 256 come valore per **Number of addresses**.
1. Seleziona **VPC default** per l'ACL (access control list) della tua sottorete. Puoi configurare le regole in entrata e in uscita in un secondo momento. 
1. Commuta **Public gateway** in **Attached**. 
1. Fai clic su **Create subnet** per eseguirne il provisioning.

### Crea un gruppo di sicurezza

Per creare un nuovo gruppo di sicurezza:  
1. Fai clic su **Security groups** in Network, poi su **New security group**.  
2. Immetti **vpc-secure-private-sg** come nome e seleziona il VPC che hai creato in precedenza.    
3. Fai clic su **Create security group**.  

### Crea una VSI (virtual server instance)

Per creare una VSI (virtual server instance) nella sottorete appena creata:

1. Fai clic sulla sottorete privata in **Subnets**.
2. Fai clic su **Attached instances**, poi su **New instance**.
3. Immetti un nome univoco, **vpc-secure-private-vsi**, seleziona il VPC che hai creato prima, quindi la stessa **ubicazione** come hai fatto in precedenza. 
4. Seleziona l'immagine **Ubuntu Linux**, fai clic su **All profiles** e, in **Compute**, scegli **c-2x4** con 2vCPU e 4 GB di RAM
5. Per **SSH keys**, seleziona la chiave SSH che hai creato in precedenza per il bastion.
6. In **Network interfaces**, fai clic sull'icona **Edit** accanto a Security Groups   
   * Seleziona **vpc-secure-private-subnet** come sottorete.   
   * Deseleziona il gruppo di sicurezza predefinito e attiva **vpc-secure-private-sg**.  
   * Fai clic su **Save**.  
7. Fai clic su **Create virtual server instance**.  


### Aggiungi i server virtuali al gruppo di sicurezza di manutenzione
{: #add-vsi-to-maintenance}

Per attività amministrative sui server, devi associare specifici server virtuali al gruppo di sicurezza di manutenzione. Nella seguente sezione, abiliterai la manutenzione, accederai al server privato, aggiornerai le informazioni sul pacchetto software, quindi annullerai di nuovo l'associazione del gruppo di sicurezza. 

Abilitiamo il gruppo di sicurezza di manutenzione per il server. 

1. Passa a **Security groups** e seleziona il gruppo di sicurezza **vpc-secure-maintenance-sg**.  
2. Fai clic su **Attached interfaces**, poi su **Edit interfaces**.   
3. Espandi le VSI (virtual server instance) e attiva la sezione accanto a **primary** nella colonna **Interfaces**.
4. Fai clic su **Save** per applicare le modifiche. 

### Connettiti all'istanza

Per eseguire l'SSH in un'istanza utilizzando il suo **IP privato**, utilizzerai l'host bastion come tuo **host jump**.

1. Ottieni l'indirizzo IP privato di una VSI (virtual server instance) in **Virtual server instances**.
2. Utilizza il comando ssh con `-J` per accedere al server con l'indirizzo **IP mobile** del bastion che hai utilizzato in precedenza e l'indirizzo **IP privato** del server mostrato in **Network interfaces**.

   ```sh
   ssh -J root@<BASTION_FLOATING_IP_ADDRESS> root@<PRIVATE_IP_ADDRESS>
   ```
   {:pre}
   
   L'indicatore `-J` è supportato in OpenSSH versione 7.3+. Nelle versioni precedenti, `-J` non è disponibile. In questo caso, il modo più sicuro e più semplice è quello di utilizzare la modalità di inoltro stdio di ssh (`-W`) per "far rimbalzare" la connessione tramite un host bastion. Ad esempio, `ssh -o ProxyCommand="ssh -W %h:%p root@<BASTION_FLOATING_IP_ADDRESS" root@<PRIVATE_IP_ADDRESS>`
   {:tip }

### Installa il software ed esegui le attività di manutenzione

Una volta connesso, puoi installare il software sul server virtuale nella sottorete privata oppure eseguire le attività di manutenzione. 

1. Innanzitutto, aggiorna le informazioni sul pacchetto software:
   ```sh
   apt-get update
   ```
   {:pre}
2. Installa il software desiderato, ad esempio Nginx o MySQL oppure IBM Db2.

Una volta terminato, disconnettiti dal server con il comando `exit`. 

Per consentire le richieste HTTP/HTTPS dall'utente di internet, assegna un **IP mobile** alla VSI nella sottorete privata e apri le porte richieste (80 - HTTP e 443 - HTTPS) tramite le regole in entrata nel gruppo di sicurezza della VSI privata.
{:tip}

### Disabilita il gruppo di sicurezza di manutenzione

Una volta terminata l'installazione del software o l'esecuzione della manutenzione, devi rimuovere i server virtuali dal gruppo di sicurezza di manutenzione per mantenerli isolati. 

1. Passa a **Security groups** e seleziona il gruppo di sicurezza **vpc-secure-maintenance-sg**.  
2. Fai clic su **Attached interfaces**, poi su **Edit interfaces**.   
3. Espandi le VSI (virtual server instance) e deseleziona la selezione accanto a **primary** nella colonna **Interfaces**.
4. Fai clic su **Save** per applicare le modifiche. 

## Rimuovi le risorse
{: #removeresources}

1. Passa a **Virtual server instances** ed **elimina** le tue istanze. Le istanze verranno eliminate e il loro stato rimarrà per un po' in **Deleting**. Assicurati di aggiornare il tuo browser di tanto in tanto. 
2. Una volta terminato con le VSI, passa a **Subnets** ed elimina le tue sottoreti.
4. Una volta eliminate le sottoreti, passa alla scheda **Virtual private clouds** ed elimina il tuo VPC.

Quando utilizzi la console, potresti dover aggiornare il tuo browser per vedere le informazioni sullo stato aggiornate dopo l'eliminazione di una risorsa.
{:tip}

## Contenuto correlato
{: #related}

* [Sottoreti private e pubbliche in un VPC (Virtual Private Cloud)](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend)
