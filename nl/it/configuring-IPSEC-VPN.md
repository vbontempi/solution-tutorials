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

# VPN in una rete privata sicura
{: #configuring-IPSEC-VPN}

La necessità di creare una connessione privata tra un ambiente di rete remoto e i server sulla rete privata di {{site.data.keyword.Bluemix_notm}} è un requisito comune. Più tipicamente, questa connettività supporta carichi di lavoro ibridi, trasferimenti di dati, carichi di lavoro privati o amministrazione di sistemi su {{site.data.keyword.Bluemix_notm}}. Un tunnel VPN (Virtual Private Network) site-to-site è l'approccio adottato generalmente per garantire la connettività tra le reti. 

{{site.data.keyword.Bluemix_notm}} fornisce una serie di opzioni per la connettività dei data center site-to-site, utilizzando una VPN su Internet pubblico o tramite una connessione di rete dedicata privata. 

Vedi {{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
per ulteriori dettagli sui link di rete sicura dedicati a {{site.data.keyword.Bluemix_notm}}. Una VPN su Internet pubblico offre un'opzione di costo inferiore, ma senza garanzie di larghezza di banda. 

Esistono due opzioni VPN idonee per la connettività su Internet pubblico ai server di cui è stato eseguito il provisioning su {{site.data.keyword.Bluemix_notm}}:

-	[VPN IPSEC]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[VPN VRA (Virtual Router Appliance)](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Questa esercitazione presenta la configurazione di una VPN IPSec site-to-site che utilizza un VRA (Virtual Router Appliance) per connettere una sottorete in un data center del client a una sottorete protetta sulla rete privata {{site.data.keyword.Bluemix_notm}}. 

Questo esempio si basa sull'esercitazione [Isola i carichi di lavoro con una rete privata sicura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure). Utilizza una VPN IPSec
site-to-site, un tunnel GRE e l'instradamento statico. Per configurazioni VPN più complesse che utilizzano l'istradamento dinamico (BGP ecc.) e i tunnel VTI, è possibile consultare la [documentazione supplementare di VRA](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Obiettivi
{: #objectives}

- Documentare i parametri di configurazione per la VPN IPSec
- Configurare la VPN IPSec su un VRA (Virtual Router Appliance)
- Instradare il traffico attraverso un tunnel GRE

## Servizi utilizzati
{: #products}

Questa esercitazione utilizza i seguenti servizi {{site.data.keyword.Bluemix_notm}}: 

* [VPN VRA (Virtual Router Appliance)](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Questa esercitazione può comportare degli addebiti. Il VRA è disponibile solo su un piano a pagamento mensile. 

## Architettura
{: #architecture}

<p style="text-align: center;">

  ![Architettura](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	Documenta la configurazione VPN
2.	Crea la VPN IPSec su un VRA
3.	Configurazione della VPN del data center e del tunnel
4.	Crea il tunnel GRE
5.	Crea l'instradamento IP statico
6.	Configura il firewall 

## Prima di iniziare
{: #prereqs}

Questa esercitazione connette l'enclosure privata sicura illustrata nell'esercitazione [Isola i carichi di lavoro con una rete privata sicura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) al tuo data center. È necessario prima completare tale esercitazione.

## Documenta la configurazione VPN
{: #Document_VPN}

La configurazione di un link site-to-site della VPN IPSec tra il tuo data center e {{site.data.keyword.Bluemix_notm}} richiede il coordinamento con il tuo team di rete in loco per determinare molti dei parametri di configurazione, il tipo di tunnel e le informazioni di instradamento IP. I parametri devono essere perfettamente conformi per ottenere una connessione VPN operativa. In genere, il tuo team di rete in loco specificherà la configurazione per soddisfare gli standard aziendali concordati e ti fornirà l'indirizzo IP necessario per il gateway VPN del data center e gli intervalli di indirizzi di sottorete a cui è possibile accedere.

Prima di iniziare la configurazione della VPN, gli indirizzi IP dei gateway VPN e gli intervalli di sottorete della rete IP devono essere determinati e disponibili per la configurazione VPN del data center e per l'enclosure di rete privata sicura in {{site.data.keyword.Bluemix_notm}}. Questi sono illustrati nella seguente figura, in cui la zona APP nell'enclosure sicura verrà connessa tramite il tunnel IPSec ai sistemi nella ‘Sottorete IP DC’ nel data center del client.   

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

I seguenti parametri devono essere concordati e documentati tra l'utente {{site.data.keyword.Bluemix_notm}} che configura la VPN e il team di rete per il data center del client. In questo esempio gli indirizzi IP del tunnel remoto e locale sono impostati su 192.168.10.1 e 192.168.10.2. Qualsiasi sottorete arbitraria può essere utilizzata con il consenso del team di rete in loco. 

| Elemento  | Descrizione |
|:------ |:--- | 
| &lt;ike group name&gt; | Nome fornito al gruppo IKE per la connessione. |
| &lt;ike encryption&gt; | Standard di crittografia IKE concordato da utilizzare tra {{site.data.keyword.Bluemix_notm}} e il data center del client, in genere *aes256*. |
| &lt;ike hash&gt; | Hash IKE concordato tra {{site.data.keyword.Bluemix_notm}} e il data center del client, in genere *sha1*. |
| &lt;ike-lifetime&gt; | Durata IKE dal data center del client, in genere *3600*. |
| &lt;esp group name&gt; | Nome fornito al gruppo ESP per la connessione. |
| &lt;esp encryption&gt; | Standard di crittografia ESP concordato tra {{site.data.keyword.Bluemix_notm}} e il data center del client, in genere *aes256*. |
| &lt;esp hash&gt; | Hash ESP concordato tra {{site.data.keyword.Bluemix_notm}} e il data center del client, in genere *sha1*. |
| &lt;esp-lifetime&gt; | Durata ESP dal data center del client, in genere *1800*. |
| &lt;DC VPN Public IP&gt;  | Indirizzo IP pubblico esposto a Internet del gateway VPN nel data center del client. | 
| &lt;VRA Public IP&gt; | Indirizzo IP pubblico del VRA. |
| &lt;Remote tunnel IP\/24&gt; | Indirizzo IP assegnato all'estremità remota del tunnel IPSec. Coppia di indirizzi IP nell'intervallo che non è in conflitto con IP Cloud o con il data center del client.   |
| &lt;Local tunnel IP\/24&gt; | Indirizzo IP assegnato all'estremità locale del tunnel IPSec.   |
| &lt;DC Subnet/CIDR&gt; | Indirizzo IP della sottorete a cui accedere nel data center del client e CIDR. |
| &lt;App Zone subnet/CIDR&gt; | Indirizzo IP di rete e CIDR della sottorete della Zona APP dall'esercitazione di creazione del VRA. | 
| &lt;Shared-Secret&gt; | Chiave di crittografia condivisa da utilizzare tra {{site.data.keyword.Bluemix_notm}} e il data center del client. |

## Configura la VPN IPSec su un VRA
{: #Configure_VRA_VPN}

Per creare la VPN su {{site.data.keyword.Bluemix_notm}}, i comandi e tutte le variabili che devono essere modificate, sono evidenziati di seguito con &lt; &gt;. Le modifiche sono identificate riga per riga, per ogni riga che deve essere modificata. I valori provengono dalla
tabella. 

1. Esegui l'SSH nel VRA ed entra nella modalità `[edit]`.
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2. Crea il gruppo IKE (Internet Key Exchange).
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike encryption>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike hash>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lifetime>
   ```
   {: codeblock}
3. Crea il gruppo ESP (Encapsulating Security Payload)
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp encryption>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp hash>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lifetime>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. Definisci la connessione site-to-site
   ```
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication mode pre-shared-secret
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication pre-shared-secret <Shared-Secret>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  connection-type initiate
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  ike-group <ike group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  local-address <VRA Public IP>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  default-esp-group <esp group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1 protocol gre
   commit
   save
   ```
   {: codeblock}

## Configura la VPN del data center e il tunnel
{: #Configure_DC_VPN}

1. Il team di rete nel data center del client configurerà una connessione VPN IPSec identica con i parametri &lt;DC VPN Public IP&gt; e &lt;VRA Public IP&gt; scambiati, l'indirizzo del tunnel locale e remoto e anche i parametri &lt;DC Subnet/CIDR&gt; e &lt;App Zone subnet/CIDR&gt; scambiati. I comandi di configurazione specifici nel data center del client dipenderanno dal fornitore della VPN.
1. Verifica che l'indirizzo IP pubblico del gateway VPN DC sia accessibile su Internet prima di procedere:
   ```
   ping <DC VPN Public IP>
   ```
   {: codeblock}
2. Dopo che la configurazione VPN del data center è stata completata, il link IPSec dovrebbe attivarsi automaticamente. Verifica che il link sia stato stabilito e che lo stato mostri che ci sono uno o più tunnel IPsec attivi. Verifica con il data center che entrambe le estremità della VPN mostrino tunnel IPsec attivi.
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. Se il link non è stato creato, verifica che gli indirizzi locali e remoti siano stati specificati correttamente e che gli altri parametri siano quelli previsti utilizzando il comando di debug:
   ``` bash
   show vpn debug
   ```
   {: codeblock}

La riga `peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......` dovrebbe essere presente nell'output. Se non esiste o mostra 'CONNECTING', c'è un errore nella configurazione VPN.  

## Definisci il tunnel GRE 
{: #Define_Tunnel}

1. Crea il tunnel GRE in modalità di modifica VRA.
   ```
   set interfaces tunnel tun0 address <Local tunnel IP/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <VRA Public IP>
   set interfaces tunnel tun0 remote-ip <DC VPN Public IP>
   commit
   ```
   {: codeblock}
2. Dopo che entrambe le estremità sono state configurate, il tunnel dovrebbe attivarsi automaticamente. Controlla lo stato operativo del tunnel dalla riga di comando VRA.
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   Il primo comando dovrebbe mostrare il tunnel con lo stato e il link come `u/u` (UP/UP). Il secondo comando mostra ulteriori dettagli sul tunnel e che il traffico viene trasmesso e ricevuto.
3. Verifica che il traffico scorra attraverso il tunnel
   ```bash
   ping <Remote tunnel IP>
   ```
   {: codeblock}
   I conteggi di TX e RX su `show interfaces tunnel tun0` dovrebbero aumentare mentre c'è il traffico `ping`.
4. Se il traffico non scorre, è possibile utilizzare i comandi `monitor interface` per osservare quale traffico viene rilevato su ciascuna interfaccia. L'interfaccia `tun0` mostra il traffico interno sul tunnel. L'interfaccia `dp0bond1` mostrerà il flusso di traffico incapsulato da e verso il gateway VPN remoto.
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

Se non viene rilevato alcun traffico di ritorno, il team di rete del data center dovrà monitorare i flussi di traffico nelle interfacce VPN e tunnel sul sito remoto per localizzare il problema. 

## Crea l'instradamento IP statico
{: #Define_Routing}

Crea l'instradamento VRA per indirizzare il traffico verso la sottorete remota tramite il tunnel.

1. Crea l'instradamento statico in modalità di modifica VRA.
   ```
   set protocols static route <DC Subnet/CIDR>  next-hop <Remote tunnel IP>
   ```
   {: codeblock}   
2. Esamina la tabella di instradamento VRA dalla riga di comando VRA. In questo momento, nessun traffico attraverserà l'instradamento in quanto non esistono regole del firewall per consentire il traffico attraverso il tunnel. Le regole del firewall sono necessarie per il traffico avviato da entrambe le parti.
   ```bash
   show ip route
   ```
   {: codeblock}

## Configura il firewall
{: #Configure_firewall}

1. Crea i gruppi di risorse per il traffico ICMP consentito e le porte TCP. 
   ```
   set res group icmp-group icmpgrp type 8
   set res group icmp-group icmpgrp type 11
   set res group icmp-group icmpgrp type 3

   set res group port tcpports port 22
   set res group port tcpports port 80
   set res group port tcpports port 443
   commit
   ```
   {: codeblock}
2. Crea le regole del firewall per il traffico verso la sottorete remota in modalità di modifica VRA.
   ```
   set security firewall name APP-TO-TUNNEL default-action drop
   set security firewall name APP-TO-TUNNEL default-log

   set security firewall name APP-TO-TUNNEL rule 100 action accept
   set security firewall name APP-TO-TUNNEL rule 100 protocol tcp
   set security firewall name APP-TO-TUNNEL rule 100 destination port tcpports

   set security firewall name APP-TO-TUNNEL rule 200 protocol icmp
   set security firewall name APP-TO-TUNNEL rule 200 icmp group icmpgrp
   set security firewall name APP-TO-TUNNEL rule 200 action accept
   commit
   ```
   {: codeblock}

   ```
   set security firewall name TUNNEL-TO-APP default-action drop
   set security firewall name TUNNEL-TO-APP default-log

   set security firewall name TUNNEL-TO-APP rule 100 action accept
   set security firewall name TUNNEL-TO-APP rule 100 protocol tcp
   set security firewall name TUNNEL-TO-APP rule 100 destination port tcpports

   set security firewall name TUNNEL-TO-APP rule 200 protocol icmp
   set security firewall name TUNNEL-TO-APP rule 200 icmp group icmpgrp
   set security firewall name TUNNEL-TO-APP rule 200 action accept 
   commit
   ```
   {: codeblock}
2. Crea la zona per il tunnel e associa i firewall per il traffico avviato in entrambe le zone.
   ```
   set security zone-policy zone TUNNEL description "GRE Tunnel"
   set security zone-policy zone TUNNEL default-action drop
   set security zone-policy zone TUNNEL interface tun0

   set security zone-policy zone TUNNEL to APP firewall TUNNEL-TO-APP
   set security zone-policy zone APP to TUNNEL firewall APP-TO-TUNNEL
   commit
   save
   ```
   {: codeblock}
3. Per verificare che i firewall e l'instradamento alle due estremità siano configurati correttamente e che ora consentano il traffico ICMP e TCP, esegui il ping dell'indirizzo gateway della sottorete remota, prima dalla riga di comando VRA e, in caso di esito positivo, accedendo alla VSI.
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}
4. Se il ping dalla riga di comando VRA non riesce, verifica che una risposta di ping sia visualizzata in risposta a una richiesta di ping sull'interfaccia del tunnel.
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
   Nessuna risposta indica un problema con le regole del firewall o con l'instradamento nel data center. Se viene visualizzata una risposta nell'output di monitoraggio, ma il comando ping scade, controlla la configurazione delle regole del firewall VRA locale.
5. Se il ping dalla VSI non riesce, ciò indica un problema con le regole del firewall VRA, con l'instradamento nel VRA o con la configurazione VSI. Completa il passo precedente per assicurarti che venga inviata una richiesta e che la risposta venga vista dal data center. Il monitoraggio del traffico sulla VLAN locale e l'ispezione dei log del firewall aiuteranno a isolare il problema all'instradamento o al firewall.
   ```
   monitor interfaces bonding dp0bond0.<VLAN ID>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

Questo passo completa la configurazione della VPN dall'enclosure di rete privata sicura. Le ulteriori esercitazioni in questa serie illustrano come l'enclosure può accedere ai servizi su Internet pubblico.

## Rimuovi le risorse
{:removeresources}

Passi da eseguire per rimuovere le risorse create in questa esercitazione.

Il VRA è su un piano a pagamento mensile. L'annullamento non prevede un rimborso. Ti consigliamo di annullare solo se questo VRA non sarà richiesto di nuovo nel prossimo mese. Se è richiesto un cluster ad alta disponibilità VRA duale, è possibile eseguire l'upgrade di questo singolo VRA nella pagina dei [dettagli del gateway](https://{DomainName}/classic/network/gatewayappliances).
{:tip}  

1. Annulla qualsiasi server virtuale o server bare metal
2. Annulla il VRA
3. Annulla eventuali VLAN aggiuntive mediante il ticket di supporto. 

## Contenuto correlato
{:related}
- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Sottoreti IP statici e portatili](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Documentazione di Vyatta](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS IPsec Site-to-Site VPN Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS Tunnels Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
