---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-08"
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

# Isola i carichi di lavoro con una rete privata sicura
{: #secure-network-enclosure}

La necessità di ambienti di rete privati isolati e protetti è fondamentale per il modello di sviluppo di applicazioni IaaS sul cloud pubblico. Firewall, VLAN, instradamento e VPN sono tutti componenti necessari nella creazione di ambienti privati isolati. Questo isolamento consente alle macchine virtuali e ai server bare-metal di essere distribuiti in modo sicuro in topologie di applicazioni multilivello complesse fornendo al tempo stesso la protezione dai rischi su internet pubblico.  

Questa esercitazione evidenzia come può essere configurato un VRA ([Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-)) su {{site.data.keyword.Bluemix_notm}} per creare una rete privata sicura (enclosure). Il VRA Gateway Appliance fornisce un singolo pacchetto autogestito, firewall, gateway VPN, NAT (Network Address Translation) e instradamento di livello enterprise. In questa esercitazione, viene utilizzato un VRA per mostrare in che modo è possibile creare un ambiente di rete isolato e circoscritto su {{site.data.keyword.Bluemix_notm}}. In questa enclosure, è possibile creare le topologie di applicazione utilizzando le note e diffuse tecnologie di instradamento IP, VLAN, sottoreti IP, regole del firewall e server bare-metal.  

{:shortdesc}

Questa esercitazione è un punto di partenza per la rete classica su {{site.data.keyword.Bluemix_notm}} e non deve essere considerata una funzionalità di produzione così com'è. Ulteriori funzionalità che possono essere prese in considerazione sono:
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [Dispositivi firewall hardware](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* [VPN IPSec](https://{DomainName}/catalog/infrastructure/ipsec-vpn) per una connettività protetta al tuo data center.
* Alta disponibilità con VRA organizzati in cluster e uplink doppi.
* Registrazione e controllo degli eventi di sicurezza.

## Obiettivi 
{: #objectives}

* Distribuire un VRA (Virtual Router Appliance)
* Definire VLAN e sottoreti IP per distribuire macchine virtuali e server bare-metal
* Proteggere VRA ed enclosure con le regole del firewall

## Servizi utilizzati
{: #products}

Questa esercitazione utilizza i seguenti servizi {{site.data.keyword.Bluemix_notm}}:
* [Virtual Router Appliance](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)

Questa esercitazione può comportare degli addebiti. Il VRA è disponibile solo su un piano a pagamento mensile. 

## Architettura
{: #architecture}

<p style="text-align: center;">

  ![Architettura](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. Configura VPN
2. Distribuisci VRA 
3. Crea il Virtual Server
4. Instrada l'accesso tramite VRA
5. Configura il firewall dell'enclosure
6. Definisci la zona APP
7. Definisci la zona INSIDE

## Prima di iniziare
{: #prereqs}

### Configura l'accesso VPN

In questa esercitazione, l'enclosure di rete creata non è visibile su Internet pubblico. Il VRA e gli eventuali server saranno accessibili solo tramite la rete privata e utilizzerai la tua VPN per la connettività. 

1. [Assicurati che il tuo accesso VPN sia abilitato](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Devi essere un utente master (**Master User**) per abilitare l'accesso VPN o contattare l'utente master per l'accesso.
     {:tip}
2. Ottieni le tue credenziali di accesso VPN selezionando il tuo utente nell'elenco utenti ([Users list](https://{DomainName}/iam#/users)).
3. Esegui l'accesso alla VPN mediante [l'interfaccia web](https://www.softlayer.com/VPN-Access) oppure utilizza un client VPN per [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) o [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Per il client VPN, utilizza il nome dominio completo (FQDN) di un singolo punto di accesso VPN a un data center dalla [pagina di accesso web VPN](https://www.softlayer.com/VPN-Access), del modulo *vpn.xxxnn.softlayer.com* come indirizzo gateway.
   {:tip}

### Controlla le autorizzazioni dell'account

Contatta il tuo utente master dell'infrastruttura per ottenere le seguenti autorizzazioni:
- **Autorizzazioni rapide** - utente di base
- **Rete** in modo da poter creare e configurare l'enclosure. Sono obbligatorie tutte le autorizzazioni di rete. 
- **Servizi** per gestire le chiavi SSH

### Carica le chiavi SSH

Tramite il portale, [carica la chiave pubblica SSH](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial) che verrà utilizzata per accedere e amministrare il VRA e la rete privata.  

### Data center di destinazione

Scegli un data center {{site.data.keyword.Bluemix_notm}} per la distribuzione della rete privata sicura. 

### Ordina le VLAN

Per creare l'enclosure privata nel data center di destinazione, devono essere prima assegnate le VLAN private necessarie. La prima VLAN privata e la prima VLAN pubblica non comportano alcun addebito. Ulteriori VLAN per supportare una topologia dell'applicazione multilivello sono a pagamento. 

Per garantire che sia disponibile un numero di VLAN sufficiente sullo stesso router di data center e che sia possibile eseguirne l'associazione al VRA, ti consigliamo di ordinarle. Vedi il documento relativo all'[ordinazione di VLAN](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans).

## Esegui il provisioning del VRA (Virtual Router Appliance)
{: #VRA}

Il primo passo consiste nel distribuire un VRA che fornirà l'instradamento IP e il firewall per l'enclosure di rete privata. Internet è accessibile dall'enclosure mediante una VLAN di transito pubblica fornita da {{site.data.keyword.Bluemix_notm}}, un gateway e, facoltativamente, un firewall hardware per creare la connettività dalla VLAN pubblica alle VLAN di enclosure private sicure. In questa esercitazione della soluzione, un VRA (Virtual Router Appliance) fornisce questo gateway e il firewall per il perimetro. 

1. Dal catalogo, seleziona un [Gateway Appliance](https://{DomainName}/gen1/infrastructure/provision/gateway)
3. Nella sezione **Gateway Vendor**, seleziona AT&T. Puoi scegliere tra una velocità di uplink fino a 20 Gbps ("up to 20 Gbps") o fino a 2 Gbps ("up to 2 Gbps").
4. Nella sezione **Hostname** inserisci un nome host (Hostname) e un dominio (Domain) per il tuo nuovo VRA.
5. Se selezioni la casella di spunta **High Availability**, otterrai due dispositivi VRA che operano in una configurazione attiva/di backup utilizzando VRRP.
6. Nella sezione **Location**, seleziona l'ubicazione (Location) e il **Pod** in cui hai bisogno del tuo VRA.
7. Seleziona Single Processor o Dual Processor. Otterrai un elenco dei server. Scegli un server facendo clic sul relativo pulsante di opzione. 
8. Seleziona la quantità di **RAM**. Per un ambiente di produzione, ti consigliamo di utilizzare un minimo di 64GB di RAM. Per un ambiente di test, ti consigliamo un minimo di 8GB.
9. Seleziona una chiave SSH (**SSH Key**) (facoltativo). Questa chiave SSH verrà installata sul VRA in modo che l'utente vyatta possa essere utilizzato per accedere al VRA con questa chiave.
10. Per il disco rigido (Hard Drive), mantieni il valore predefinito.
11. Nella sezione **Uplink Port Speeds**, seleziona la combinazione di velocità, ridondanza e interfacce private e/o pubbliche che soddisfa le tue esigenze.
12. Nella sezione **Add-ons**, mantieni il valore predefinito. Se desideri utilizzare IPv6 sull'interfaccia pubblica, seleziona l'indirizzo IPv6.

Sul lato destro, puoi vedere il tuo riepilogo dell'ordine (**Order Summary**). Seleziona la casella di spunta _I have read and agree to the Third-Party Service Agreements listed below:_ e fai clic sul pulsante **Create**. Il tuo gateway verrà distribuito.

L'elenco dispositivi ([Device list](https://{DomainName}/classic/devices)) visualizzerà il VRA quasi immediatamente, contrassegnato da un simbolo **Clock** che indica che su questo dispositivo sono in corso delle transazioni. Il simbolo **Clock** persiste finché la creazione del VRA non sarà stata completata e, tranne visualizzare i dettagli, non è possibile eseguire alcuna azione di configurazione sul dispositivo.
{:tip}

### Esamina il VRA distribuito

1. Ispeziona il nuovo VRA. Nel [dashboard dell'infrastruttura](https://{DomainName}/classic), seleziona **Network** nel riquadro di sinistra, seguito da **Gateway Appliances**, per andare alla pagina [Gateway Appliances](https://{DomainName}/classic/network/gatewayappliances). Seleziona il nome del VRA appena creato nella colonna **Gateway** per procedere alla pagina Gateway Details. ![](images/solution33-secure-network-enclosure/Gateway-detail.png)

2. Prendi nota degli indirizzi IP privato (`Private`) e pubblico (`Public`) del VRA per utilizzi futuri.

## Configurazione di VRA iniziale
{: #initial_VRA_setup}

1. Dalla tua workstation, tramite la VPN SSL, esegui l'accesso al VRA utilizzando l'account **vyatta** predefinito e accettando i prompt di sicurezza di SSH. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   Se SSH richiede una password, la chiave SSH non era stata inclusa nella build. Accedi al VRA tramite il [browser web](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui) utilizzando l'indirizzo IP privato VRA (`VRA Private IP Address`). La password proviene dalla pagina [Software Passwords](https://{DomainName}/classic/devices/passwords). Nella scheda **Configuration**, seleziona il ramo System/login/vyatta e aggiungi la chiave SSH desiderata.
   {:tip}

   La configurazione del VRA richiede che quest'ultimo venga messo in modalità \[edit\] utilizzando il comando `configure`. Quando è in modalità `edit`, il prompt cambia da `$` a `#`. Dopo una corretta modifica della configurazione del VRA, puoi visualizzare le tue modifiche con il comando `compare` e controllare le tue modifiche con il comando `validate`. Eseguendone il commit con il comando `commit`, una modifica verrà applicata alla configurazione in esecuzione e salvata automaticamente nella configurazione di avvio.


   {:tip}
2. Migliora la sicurezza consentendo solo l'accesso SSH. Ora che l'accesso SSH ha esito positivo tramite la rete privata, disabilita l'accesso mediante autenticazione con id utente e password. 
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
   Da questo punto in questa esercitazione, si presume che tutti i comandi VRA siano immessi nel prompt `edit`, dopo l'immissione di `configure`.
3. Riesamina la configurazione iniziale
   ```
   show
   ```
   {: codeblock}

   Il VRA è preconfigurato per l'ambiente IaaS {{site.data.keyword.Bluemix_notm}} Ciò include quanto segue:
   - Server NTP
   - Server dei nomi
   - SSH
   - Server web HTTPS 
   - Fuso orario predefinito Stati Uniti/Chicago
4. Imposta il fuso orario locale come richiesto. Il completamento automatico con il tasto di tabulazione elencherà i potenziali valori di fuso orario.
   ```
   set system time-zone <timezone>
   ```
   {: codeblock}
5. Imposta la modalità di funzionamento del ping. Il ping non è disabilitato per assistenza nella risoluzione dei problemi di firewall e instradamento. 
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. Abilita l'operazione del firewall con stato. Per impostazione predefinita il firewall VRA è senza stato. 
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. Esegui il commit e salva automaticamente le tue modifiche alla configurazione di avvio. 
   ```
   commit
   ```
   {: codeblock}

## Ordina il primo server virtuale
{: #order_virtualserver}

A questo punto viene creato un server virtuale per assistere nella diagnostica degli errori di configurazione VRA. Un corretto accesso alla VSI viene convalidato sulla rete privata {{site.data.keyword.Bluemix_notm}} prima che l'accesso a essa venga instradato tramite il VRA in un passo successivo. 

1. Ordina un [server virtuale](https://{DomainName}/catalog/infrastructure/virtual-server-group)  
2. Seleziona **Public Virtual Server** e continua.
3. Nella pagina dell'ordine:
   - Imposta **Billing** su **Hourly**.
   - Imposta *VSI Hostname* e *Domain name*. Questo nome dominio non viene utilizzato per l'instradamento e DNS ma deve allinearsi con i tuoi standard di denominazione di rete. 
   - Imposta **Location** sul nome del VRA.
   - Imposta **Profile** su **C1.1x1**
   - Aggiungi la chiave SSH (**SSH Key**) che hai specificato in precedenza.
   - Imposta **Operating System** su **CentOS 7.x - Minimal**
   - In **Uplink Port Speeds**, l'interfaccia di rete deve essere modificata dal valore predefinito di *public and private* per specificare solo un **Private Network Uplink**. Questo assicura che il nuovo server non abbia alcun accesso diretto a Internet e che l'accesso sia controllato dalle regole di instradamento e firewall sul VRA.
   - Imposta **Private VLAN** sull'ID VLAN della VLAN privata ordinata in precedenza.
4. Fai clic su casella di spunta per accettare gli accordi di servizio di terze parti ('Third-Party') e seleziona quindi **Create**.
5. Monitora il completamento nella pagina [Devices](https://{DomainName}/classic/devices) oppure via email. 
6. Prendi nota dell'indirizzo IP privato (*Private IP address*) della VSI per un passo successivo e verifica nella sezione **Network** nella pagina **Device Details** che la VSI sia assegnata alla VLAN corretta. In caso contrario, elimina questa VSI e creane una nuova sulla VLAN corretta. 
7. Verificare il corretto accesso alla VSI tramite la rete privata {{site.data.keyword.Bluemix_notm}} utilizzando ping e SSH dalla tua workstation locale sulla VPN.
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## Instrada l'accesso VLAN tramite il VRA
{: #routing_vlan_via_vra}

La VLAN o le VLAN private per il server virtuale saranno state associate dal sistema di gestione {{site.data.keyword.Bluemix_notm}} a questo VRA. A questo punto, la VSI è ancora accessibile tramite l'instradamento IP sulla rete privata {{site.data.keyword.Bluemix_notm}}. Eseguirai ora l'instradamento alla sottorete tramite il VRA per creare la rete privata sicura ed eseguirai la convalida confermando che la VSI è ora non accessibile. 

1. Procedi ai dettagli del gateway (Gateway Details) per il VRA tramite la pagina [Gateway Appliances](https://{DomainName}/classic/network/gatewayappliances) e individua la sezione **Associated VLANs** nella metà inferiore della pagina. La VLAN associata sarà elencata qui. 
2. Se a questo punto vuoi aggiungere ulteriori VLAN, vai alla sezione **Associate a VLAN**. La casella a discesa *Select VLAN* dovrebbe essere abilitata e sarà possibile selezionare altre VLAN di cui è stato eseguito il provisioning. ![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png)

   Se non viene visualizzata alcuna VLAN idonea, non è disponibile alcuna VLAN sullo stesso router del VRA. Ciò richiederà l'apertura di un [ticket di supporto](https://{DomainName}/unifiedsupport/cases/add) per richiedere una VLAN privata sullo stesso router del VRA.
   {:tip}
5. Seleziona la VLAN che desideri associare con il VRA e fai clic su Save. Il completamento dell'associazione iniziale della LAN potrebbe richiedere un paio di minuti. Una volta completata, la VLAN dovrebbe essere visualizzata sotto l'intestazione **Associated VLANs**. 

A questo punto, la VLAN e la sottorete associata non sono protette o instradate tramite il VRA e la VSI è accessibile tramite la rete privata di {{site.data.keyword.Bluemix_notm}}. Lo stato della VLAN verrà visualizzato come *Bypassed*.

4. Seleziona **Actions** nella colonna di destra e quindi **Route VLAN** per instradare la VLAN/sottorete tramite il VRA. Questa operazione richiederà alcuni minuti. Un aggiornamento dello schermo mostrerà che è instradata (*Routed*). 
5. Seleziona il [nome della VLAN](https://{DomainName}/classic/network/vlans/)) per visualizzare i dettagli della VLAN. La VSI di cui è stato eseguito il provisioning può essere vista anche come la sottorete IP primaria assegnata. Prendi nota dell'ID della VLAN privata <nnnn\> (1199 in questo esempio) poiché verrà utilizzato in un passo successivo. 
6. Seleziona la [sottorete](https://{DomainName}/classic/network/subnets) per visualizzare i dettagli della sottorete IP. Prendi nota degli indirizzi gateway, del CIDR (/26) e della rete della sottorete poiché saranno necessari per l'ulteriore configurazione del VRA. Viene eseguito il provisioning di 64 indirizzi IP primari sulla rete privata e, per trovare l'indirizzo gateway, potrebbe essere necessario selezionare la pagina 2 o 3. 
7. Convalida che la sottorete/VLAN è instradata al VRA e che la VSI **NON** è accessibile tramite la rete di gestione dalla tua workstation utilizzando `ping`. 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}

Ciò completa l'instradamento del VRA tramite la console {{site.data.keyword.Bluemix_notm}}. Il lavoro aggiuntivo per configurare l'enclosure e l'instradamento IP viene ora eseguito direttamente sul VRA tramite SSH. 

## Configura l'instradamento IP e l'enclosure protetta
{: #vra_setup}

Quando viene eseguito il commit della configurazione VRA, la configurazione in esecuzione viene modificata e le modifiche vengono salvate automaticamente nella configurazione di avvio.

Se vuoi ritornare a una configurazione funzionante precedente, per impostazione predefinita è possibile visualizzare, confrontare o ripristinare gli ultimi 20 punti di commit. Vedi il manuale [Vyatta Network OS Basic System Configuration Guide](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation) per ulteriori dettagli sul commit e sul salvataggio della configurazione.
   ```bash
   show system commit
   rollback n
   compare
   ```
   {: codeblock}

### Configura l'instradamento IP VRA

Configura l'interfaccia di rete virtuale VRA per eseguire l'instradamento alla nuova sottorete dalla rete privata {{site.data.keyword.Bluemix_notm}}.  

1. Accedi al VRA mediante SSH. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}
2. Crea una nuova interfaccia virtuale con l'ID VLAN privato, l'indirizzo IP gateway di sottorete e il CIDR registrati nei passi precedenti. Il CIDR di norma sarà /26. 
   ```
   set interfaces bonding dp0bond0 vif <VLAN ID> address <Subnet Gateway IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   È fondamentale che venga utilizzato l'indirizzo **`<Subnet Gateway IP>`**. Di norma è uno dei primi indirizzi nell'intervallo di sottoreti. L'immissione di un indirizzo gateway non valido causerà l'errore `Configuration path: interfaces bonding dp0bond0 vif xxxx address [x.x.x.x] is not valid`. Correggi il comando e riesegui l'immissione. Puoi cercarlo in Network > IP Management > Subnets. Fai clic sulla sottorete di cui ti serve conoscere l'indirizzo gateway. La seconda voce nell'elenco con la descrizione (Description) **Gateway** è l'indirizzo IP da immettere come <Subnet Gateway IP>/<CIDR>.
   {: tip}

3. Elenca la nuova interfaccia virtuale (vif): 
   ```
   show interfaces
   ```
   {: codeblock}

   Questa è una configurazione dell'interfaccia di esempio che mostra vif 1199 e l'indirizzo gateway di sottorete.
   ![](images/solution33-secure-network-enclosure/show_interfaces.png)
4. Convalida che la VSI è nuovamente accessibile tramite la rete di gestione dalla tua workstation. 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
   
   Se la VSI non è accessibile, controlla che la tabella di instradamento IP VRA sia configurata come previsto. Elimina e crea nuovamente l'instradamento, se necessario. Per eseguire un comando show in modalità di configurazione, puoi utilizzare il comando run    
   ```bash
    run show ip route <Subnet Gateway IP>
   ```
   {: codeblock}

Ciò completa la configurazione dell'instradamento IP.

### Configura l'enclosure protetta

L'enclosure di rete privata sicura viene creata mediante la configurazione di zone e regole del firewall. Esamina la documentazione VRA sulla [configurazione del firewall](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls) prima di procedere. 

Sono definite due zone:
   - INSIDE:  la rete di gestione e quella privata IBM
   - APP:  la sottorete e la VLAN utente all'interno dell'enclosure di rete privata		

1. Definisci i firewall e i valori predefiniti.
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   Se un comando set viene accidentalmente eseguito due volte, riceverai un messaggio *'Configuration path xxxxxxxx is not valid. Node exists'*. Può essere ignorato. Per modificare un parametro non corretto, è necessario eliminare prima il nodo con 'delete security xxxxx xxxx xxxxx'.
   {:tip}
2. Crea il gruppo di risorse di rete privata {{site.data.keyword.Bluemix_notm}}. Questo gruppo di indirizzi definisce le reti private {{site.data.keyword.Bluemix_notm}} che possono accedere all'enclosure e le reti che possono essere raggiunte dall'enclosure. Due set di indirizzi IP devono accedere alla e dalla enclosure protetta, ossia i data center VPN SSL e la rete del servizio {{site.data.keyword.Bluemix_notm}} (rete di backend/privata). [{{site.data.keyword.Bluemix_notm}} IP Ranges](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges) fornisce l'elenco completo di intervalli IP che devono essere consentiti. 
   - Definisci l'indirizzo VPN SSL del(i) data center che stai utilizzando per l'accesso VPN. Dalla sezione SSL VPN di {{site.data.keyword.Bluemix_notm}} IP Ranges, seleziona i punti di accesso VPN per il tuo data center o cluster DC. L'esempio qui riportato mostra gli intervalli di indirizzi VPN per i data center {{site.data.keyword.Bluemix_notm}} di Londra.
     ```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - Definisci gli intervalli di indirizzi per la ‘Rete del servizio (nella rete di backend/privata)’ {{site.data.keyword.Bluemix_notm}} per WDC04, DAL01 e il tuo data center di destinazione. L'esempio qui riportato è WDC04 (due indirizzi), DAL01 e LON06.
     ```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. Crea la zona APP per la sottorete e la VLAN utente e la zona INSIDE per la rete privata {{site.data.keyword.Bluemix_notm}}. Assegna i firewall creati in precedenza. La definizione della zona utilizza i nomi di interfaccia di rete VRA per identificare la zona associata a ciascuna VLAN. Il comando per creare la zona APP richiede l'ID VLAN della VLAN associata al VRA creato in precedenza. Ciò viene evidenziato di seguito come `<VLAN ID>`.
   ```
   set security zone-policy zone INSIDE description "IBM Internal network"
   set security zone-policy zone INSIDE default-action drop
   set security zone-policy zone INSIDE interface dp0bond0
   set security zone-policy zone INSIDE to APP firewall INSIDE-TO-APP

   set security zone-policy zone APP description "Application network"
   set security zone-policy zone APP default-action drop

   set security zone-policy zone APP interface dp0bond0.<VLAN ID>
   set security zone-policy zone APP to INSIDE firewall APP-TO-INSIDE
   ```
   {: codeblock}
4. Esegui il commit della configurazione e, dalla tua workstation, utilizza ping per verificare che il firewall sta ora rifiutando il traffico tramite il VRA alla VSI: 
   ```
   commit
   ```
   {: codeblock}

   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
5. Definisci le regole di accesso al firewall per udp, tcp e icmp.
   ```
   set security firewall name INSIDE-TO-APP rule 200 protocol icmp
   set security firewall name INSIDE-TO-APP rule 200 icmp type 8
   set security firewall name INSIDE-TO-APP rule 200 action accept
   set security firewall name INSIDE-TO-APP rule 200 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 100 action accept
   set security firewall name INSIDE-TO-APP rule 100 protocol tcp
   set security firewall name INSIDE-TO-APP rule 100 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 110 action accept
   set security firewall name INSIDE-TO-APP rule 110 protocol udp
   set security firewall name INSIDE-TO-APP rule 110 source address ibmprivate
   commit

   set security firewall name APP-TO-INSIDE rule 200 protocol icmp
   set security firewall name APP-TO-INSIDE rule 200 icmp type 8
   set security firewall name APP-TO-INSIDE rule 200 action accept
   set security firewall name APP-TO-INSIDE rule 200 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 100 action accept
   set security firewall name APP-TO-INSIDE rule 100 protocol tcp
   set security firewall name APP-TO-INSIDE rule 100 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 110 action accept
   set security firewall name APP-TO-INSIDE rule 110 protocol udp
   set security firewall name APP-TO-INSIDE rule 110 destination address ibmprivate
   commit
   ```
   {: codeblock}
6. Convalida l'accesso al firewall. 
   - Conferma che il firewall INSIDE-TO-APP sta ora consentendo il traffico ICMP e udp/tcp dalla tua macchina locale
     ```bash
     ping <VSI Private IP Address>
     SSH root@<VSI Private IP Address>
     ```
     {: codeblock}
   - Conferma che il firewall APP-TO-INSIDE sta consentendo il traffico ICMP e udp/tcp. Esegui l'accesso alla VSI utilizzando SSH ed esegui il ping di uno dei server dei nomi {{site.data.keyword.Bluemix_notm}} a 10.0.80.11 e 10.0.80.12.
     ```bash
     SSH root@<VSI Private IP Address>
     [root@vsi  ~]# ping 10.0.80.11
     ```
     {: codeblock}
7. Convalida l'accesso continuo all'interfaccia di gestione VRA tramite SSH dalla tua workstation. Se l'accesso viene mantenuto, esamina e salva la configurazione. In caso contrario, un riavvio del VRA eseguirà il ripristino a una configurazione funzionante. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   ```
   show security
   ```
   {: codeblock}

### Debug delle regole del firewall

I log del firewall possono essere visualizzati dal prompt dei comandi operativi del VRA. In questa configurazione, solo il traffico rifiutato per ciascuna zona viene registrato per assistere nella diagnosi dell'errata configurazione del firewall.  

1. Esamina i log del firewall per il traffico rifiutato. Un esame periodico dei log identificherà se i server nella zona APP stanno provando, in modo valido o erroneo, a contattare i servizi sulla rete IBM. 
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. Se i servizi o i server non sono contattabili nei log del firewall non sono presenti dati, verifica se il traffico IP ping/ssh previsto è presente sull'interfaccia di rete VRA dalla rete privata {{site.data.keyword.Bluemix_notm}} o sull'interfaccia VRA alla VLAN utilizzando il `<VLAN ID>` precedentemente indicato.
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN ID> traffic
   ```

## Proteggi il VRA
{: #securing_the_vra}

1. Applica la politica di sicurezza VRA. Per impostazione predefinita, la suddivisione in zone dei firewall basata sulle politiche non protegge l'accesso al VRA stesso. Ciò viene configurato tramite CPP (Control Plane Policing). VRA fornisce un set di regole CPP di base come template. Uniscilo nella tua configurazione:
   ```bash
   merge /opt/vyatta/etc/cpp.conf
   ```
   {: codeblock}

Ciò crea un nuovo set di regole del firewall denominato `CPP`; visualizza le regole aggiuntive ed esegui il commit in modalità di modifica (\[edit\]). 
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. Protezione dell'accesso SSH pubblico. A causa di un problema al momento irrisolto con il firmware Vyatta, ti sconsigliamo di utilizzare `set service SSH listen-address x.x.x.x` per limitare l'accesso amministrativo SSH sulla rete pubblica. In alternativa, è possibile bloccare l'accesso esterno tramite il firewall CPP per l'intervallo di indirizzi IP pubblici utilizzati dall'interfaccia pubblica VRA. Il `<VRA Public IP Subnet>` qui utilizzato è uguale al `<VRA Public IP Address>` con l'ultimo ottetto pari a zero (x.x.x.0). 
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <VRA Public IP Subnet>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit
   ```
   {: codeblock}
3. Convalida l'accesso amministrativo SSH VRA sulla rete interna IBM. Se perdi l'accesso al VRA tramite SSH dopo l'esecuzione dei commit, puoi accedere al VRA tramite la console VKM disponibile nella pagina Device Details del VRA tramite il menu a discesa Action.

Ciò completa la configurazione dell'enclosure di rete privata sicura che protegge una singola zona del firewall contenente una VLAN e una sottorete. Ulteriori zone del firewall, regole, server virtuali e bare-metal, VLAN e sottoreti possono essere aggiunti attenendosi alle stesse istruzioni. 

## Rimuovi le risorse
{: #removeresources}

In questo passo, ripulirai le risorse per rimuovere ciò che hai creato in precedenza.

Il VRA è su un piano a pagamento mensile. L'annullamento non prevede un rimborso. Ti consigliamo di annullare solo se questo VRA non sarà richiesto di nuovo nel prossimo mese. Se è richiesto un cluster ad alta disponibilità VRA duale, è possibile eseguire l'upgrade di questo singolo VRA nella pagina dei [dettagli del gateway](https://{DomainName}/classic/network/gatewayappliances/).
{:tip}  

- Annulla qualsiasi server virtuale o server bare metal
- Annulla il VRA
- Annulla eventuali VLAN aggiuntive mediante il ticket di supporto. 

## Contenuto correlato
{: #related}

- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Sottoreti IP statici e portatili](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence Platform](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Documentazione di Vyatta](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
