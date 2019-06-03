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

# Collegamento delle reti private sicure sulla rete IBM
{: #vlan-spanning}

Man mano che la necessità per la portata globale e le operazioni 24 ore su 24/7 giorni su 7 aumenta, aumenta la necessità di ospitare i servizi in più data center cloud. I data center in più ubicazioni forniscono la resilienza nel caso di un problema geografico e portano anche i carichi di lavoro più vicini agli utenti distribuiti riducendo la latenza e aumentando le prestazioni percepite. La [rete {{site.data.keyword.Bluemix_notm}}](https://www.ibm.com/cloud/data-centers/) consente agli utenti di collegare carichi di lavoro ospitati in reti private sicure tra i data center e le ubicazioni. 

Questa esercitazione presenta la configurazione di una connessione IP instradata privatamente sulla rete privata {{site.data.keyword.Bluemix_notm}} tra due reti private sicure ospitate in data center diversi. Tutte le risorse sono di proprietà di un account {{site.data.keyword.Bluemix_notm}}. Utilizza l'esercitazione [Isola i carichi di lavoro con una rete privata sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) per distribuire due reti private collegate in modo sicuro sulla rete privata {{site.data.keyword.Bluemix_notm}} utilizzando il servizio [Spanning delle VLAN]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning).
{:shortdesc}

## Obiettivi
{: #objectives}

- Collegare reti sicure all'interno di un account {{site.data.keyword.Bluemix_notm}} IaaS
- Configurare le regole firewall per l'accesso site-to-site 
- Configurare l'instradamento tra i siti

## Servizi utilizzati
{: #products}

Questa esercitazione utilizza i seguenti servizi {{site.data.keyword.Bluemix_notm}}: 
* [VRA (Virtual Router Appliance)](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [Spanning delle VLAN]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

Questa esercitazione potrebbe comportare degli addebiti. Il VRA è disponibile solo su un piano a pagamento mensile. 

## Architettura
{: #architecture}

<p style="text-align: center;">

  ![Architettura](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. Distribuisci le reti private sicure
2. Configura lo Spanning delle VLAN
3. Configura l'instradamento IP tra le reti private
4. Configura le regole firewall per l'accesso al sito remoto

## Prima di iniziare
{: #prereqs}

Questa esercitazione si basa sull'esercitazione [Isola i carichi di lavoro con una rete privata sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). Devi esaminare tale esercitazione e i suoi prerequisiti prima di iniziare.  

## Configura i siti della rete privata sicura
{: #private_network}

L'esercitazione [Isola i carichi di lavoro con una rete privata sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) viene utilizzata due volte per implementare le reti private in due diversi data center. Non ci sono limitazioni su quali due data center puoi utilizzare, a parte tenere presente l'impatto della latenza sul traffico o sui carichi di lavoro che comunicheranno tra i siti.  

Puoi seguire l'esercitazione [Isola i carichi di lavoro con una rete privata sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) senza modifiche per ogni data center selezionato, registrando le seguenti informazioni per i passi successivi.  

| Elemento  | Datacenter1 | Datacenter2 |
|:------ |:--- | :--- |
| Data center |  |  |
| Indirizzo IP pubblico VRA | <DC1 VRA Public IP Address> | <DC2 VRA Public IP Address> |
| Indirizzo IP privato VRA | <DC1 VRA Private IP Address> | <DC2 VRA Private IP Address> |
| Sottorete privata VRA & CIDR |  |  |
| ID VLAN privata | &lt;ID VLAN privata DC1&gt;  | &lt;ID VLAN privata DC2&gt; |
| Indirizzo IP privato VSI | <DC1 VSI Private IP Address> | <DC2 VSI Private IP Address> |
| Sottorete zona APP & CIDR | <DC1 APP zone subnet/CIDR> | <DC2 APP zone subnet/CIDR> |

1. Vai alla pagina Gateway Details per ogni VRA tramite la pagina [Gateway Appliances](https://{DomainName}/classic/network/gatewayappliances).  
2. Individua la sezione Gateway VLANs e fai clic sulla [VLAN]( https://{DomainName}/classic/network/vlans) del gateway sulla rete **privata** per visualizzare i dettagli della VLAN. Il nome deve contenere l'id, `bcrxxx`, rappresenta il BCR ('backend customer router') e deve avere il formato `nnnxx.bcrxxx.xxxx`.
3. Il VRA di cui è stato eseguito il provisioning comparirà nella sezione **Devices*. Dalla sezione **Subnets**, prendi nota dell'indirizzo IP della sottorete privata e del CIDR VRA (/26). La sottorete sarà di tipo primario con 64 IP. Questi dettagli saranno necessari in seguito per la configurazione dell'instradamento.  
4. Di nuovo nella pagina Gateway Details, individua la sezione **Associated VLANs** e fai clic sulla [VLAN]( https://{DomainName}/classic/network/vlans) sulla rete **privata** che è stata associata per creare la rete sicura e la zona APP. 
5. La VSI di cui è stato eseguito il provisioning comparirà nella sezione **Devices*. Dalla sezione **Subnets**, prendi nota dell'indirizzo IP della sottorete VSI e del CIDR (/26) poiché saranno necessari per la configurazione dell'instradamento. Questa VLAN e sottorete viene identificata come la zona APP in entrambe le configurazioni del firewall VRA e viene registrata come la &lt;sottorete zona APP/CIDR&gt;.


## Configura lo Spanning delle VLAN 
{: #configure-vlan-spanning}

Per impostazione predefinita, i server (e i VRA) su VLAN e data center diversi, non possono comunicare tra loro sulla rete privata. In queste esercitazioni, all'interno di un singolo data center, i VRA vengono utilizzati per collegare le VLAN e le sottoreti con i classici firewall e instradamento IP per creare una rete privata per la comunicazione tra le VLAN. Sebbene possano comunicare nello stesso data center, in questa configurazione, i server che appartengono allo stesso account {{site.data.keyword.Bluemix_notm}} non sono in grado di comunicare tra i data center. 

Il servizio [Spanning delle VLAN]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning) alleggerisce questa limitazione di comunicazione tra le VLAN e le sottoreti che **NON** sono associate ai VRA. Devi notare che anche quando lo spanning delle VLAN è abilitato, le VLAN associate ai VRA possono comunicare solo tramite il loro VRA associato, come stabilito dalla configurazione dell'instradamento e del firewall VRA. Quando lo spanning delle VLAN è abilitato per i VRA di proprietà di un account {{site.data.keyword.Bluemix_notm}}, vengono connessi sulla rete privata e possono comunicare.  

Le VLAN non associate alle reti private sicure create dai VRA, vengono ‘sottoposte a spanning’ consentendo l'interconnessione di queste VLAN ‘non associate’ tra i data center. Ciò include le VLAN del VRA Gateway (transito) che appartengono allo stesso account IBM Cloud in data center diversi. Quindi, consente ai VRA di comunicare tra i data center quando lo spanning delle VLAN è abilitato. Con la connettività da VRA a VRA, la configurazione dell'instradamento e del firewall VRA consente ai server all'interno delle reti sicure di connettersi.  

Abilita lo spanning delle VLAN:

1. Vai alla pagina [VLANs]( https://{DomainName}/classic/network/vlans).
2. Seleziona la scheda **Span** nella parte superiore della pagina. 
3. Seleziona il pulsante di scelta ‘On’ per lo spanning delle VLAN. Il completamento della modifica della rete richiederà alcuni minuti. 
4. Conferma che ora i due VRA possono comunicare:

   Accedi al VRA del data center 1 ed esegui il ping del VRA del data center 2

   ```
   SSH vyatta@<DC1 VRA Private IP Address>
   ping <DC2 VRA Private IP Address>
   ```
   {: codeblock}

   Accedi al VRA del data center 2 ed esegui il ping del VRA del data center 1
   ```
   SSH vyatta@<DC2 VRA Private IP Address>
   ping <DC1 VRA Private IP Address>
   ```
   {: codeblock}

## Configura l'instradamento IP VRA 
{: #vra_routing}

Crea l'instradamento VRA in ciascun data center per consentire alle VSI nelle zone APP in entrambi i data center di comunicare.  

1. Crea un instradamento statico nel data center 1 alla sottorete privata della zona APP nel data center 2, in modalità di modifica VRA.
   ```
   ssh vyatta@<DC1 VRA Private IP Address>
   conf
   set protocols static route <DC2 APP zone subnet/CIDR>  next-hop <DC2 VRA Private IP>
   commit
   ```
   {: codeblock}
2. Crea un instradamento statico nel data center 2 alla sottorete privata della zona APP nel data center 1, in modalità di modifica VRA.
   ```
   ssh vyatta@<DC2 VRA Private IP Address>
   conf
   set protocols static route <DC1 APP zone subnet/CIDR>  next-hop <DC1 VRA Private IP>
   commit
   ```
   {: codeblock}
2. Esamina la tabella di instradamento VRA dalla riga di comando VRA. In questo momento, le VSI non possono comunicare poiché non esistono regole firewall della zona APP per consentire il traffico tra le due sottoreti della zona APP. Le regole del firewall sono necessarie per il traffico avviato da entrambe le parti. 
   ```
   show ip route
   ```
   {: codeblock}

Ora vedrai il nuovo instradamento per consentire alla zona APP di comunicare tramite la rete privata IBM. 

## Configurazione del firewall VRA
{: #vra_firewall}

Le regole firewall della zona APP esistenti sono configurate solo per consentire il traffico da e verso questa sottorete per i servizi {{site.data.keyword.Bluemix_notm}} sulla rete privata {{site.data.keyword.Bluemix_notm}} e per l'accesso Internet pubblico tramite NAT. Le altre reti associate alle VSI su questo VRA o in altri data center sono bloccate. Il prossimo passo consiste nell'aggiornare il gruppo di risorse `ibmprivate` associate alla regola firewall APP-TO-INSIDE per consentire l'accesso esplicito alla sottorete nell'altro data center. 

1. Nella modalità del comando di modifica VRA del data center 1, aggiungi <DC2 APP zone subnet>/CIDR al gruppo di risorse `ibmprivate`
   ```
   set resources group address-group ibmprivate address <DC2 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
2. Nella modalità del comando di modifica VRA del data center 2, aggiungi <DC1 APP zone subnet>/CIDR al gruppo di risorse `ibmprivate`
   ```
   set resources group address-group ibmprivate address <DC1 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
3. Verifica che ora le VSI in entrambi i data center possano comunicare
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}

   Se le VSI non possono comunicare, segui le istruzioni presenti nell'esercitazione [Isola i carichi di lavoro con una rete privata sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) per monitorare il traffico nelle interfacce e per esaminare i log del firewall. 

## Rimuovi le risorse
{: #removeresources}

Passi da eseguire per rimuovere le risorse create in questa esercitazione. 

Il VRA è su un piano a pagamento mensile. L'annullamento non prevede un rimborso. Ti consigliamo di annullare solo se questo VRA non sarà richiesto di nuovo nel prossimo mese. Se è richiesto un cluster ad alta disponibilità VRA duale, è possibile eseguire l'upgrade di questo singolo VRA nella pagina dei [dettagli del gateway](https://{DomainName}/classic/network/gatewayappliances).
{:tip}

1. Annulla qualsiasi server virtuale o server bare metal 
2. Annulla il VRA
3. Annulla eventuali VLAN aggiuntive mediante il ticket di supporto. 


## Estendi l'esercitazione

Questa esercitazione può essere utilizzata insieme all'esercitazione
[VPN in una rete privata sicura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network) per collegare entrambe le reti sicure a una rete remota utente su una VPN IPSec. I link VPN possono essere stabiliti a entrambe le reti sicure per una maggiore resilienza dell'accesso alla piattaforma {{site.data.keyword.Bluemix_notm}} IaaS. Tieni presente che IBM non consente l'instradamento del traffico utente tra i data center client sulla rete privata IBM. La configurazione dell'instradamento per evitare i loop di rete va oltre l'ambito di questa esercitazione.  


## Materiale correlato
{:related}

1. VRF (Virtual Routing and Forwarding) costituisce un'alternativa all'uso dello spanning delle VLAN per connettere le reti in un account {{site.data.keyword.Bluemix_notm}}. VRF è obbligatorio per tutti i client che utilizzano  [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview). [Panoramica di VRF (Virtual Routing and Forwarding) su IBM Cloud](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [La rete IBM Cloud](https://www.ibm.com/cloud/data-centers/)
