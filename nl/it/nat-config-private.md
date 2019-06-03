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

# Configura NAT per l'accesso a Internet da una rete privata
{: #nat-config-private}

Nel mondo di oggi di applicazioni e servizi IT basati sul web, sono poche le applicazioni che funzionano in modo isolato. Gli sviluppatori oramai si aspettano un accesso ai servizi su internet, sia che si tratti di codice applicativo open-source che di aggiornamenti o servizi di 'terze parti' che forniscono la funzionalità applicativa tramite le API REST. Il mascheramento NAT (Network Address Translation) è un approccio comunemente utilizzato per proteggere l'accesso al servizio ospitato su internet dalle reti private. Nel mascheramento NAT, gli indirizzi IP privati sono convertiti nell'indirizzo IP dell'interfaccia pubblica in uscita in una relazione molti-a-uno, proteggendo l'indirizzo IP privato dall'accesso pubblico.  

Questa esercitazione presenta la configurazione del mascheramento NAT (Network Address Translation) su un VRA (Virtual Router Appliance) per connettersi a una rete protetta sulla rete privata {{site.data.keyword.Bluemix_notm}}. Si sviluppa sull'esercitazione [Isola i carichi di lavoro con una rete privata sicura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure), aggiungendo una configurazione SNAT (Source NAT), dove l'indirizzo di origine è offuscato e dove le regole del firewall vengono utilizzate per proteggere il traffico in uscita. Delle configurazioni NAT più complesse sono disponibili nella [documentazione supplementare di VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Obiettivi
{: #objectives}

-	Configurare SNAT (Source Network Address Translation) su un VRA (Virtual Router Appliance)
-	Configurare le regole del firewall per l'accesso a Internet

## Servizi utilizzati
{: #products}

Questa esercitazione utilizza i seguenti servizi {{site.data.keyword.Bluemix_notm}}: 

* [VPN VRA (Virtual Router Appliance)](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Questa esercitazione può comportare degli addebiti. Il VRA è disponibile solo su un piano a pagamento mensile. 

## Architettura
{: #architecture}

<p style="text-align: center;">

  ![Architettura](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	Documenta i servizi Internet richiesti.
2.	Configura NAT.
3.	Crea regole e zone firewall Internet.

## Prima di iniziare
{: #prereqs}

Questa esercitazione abilita gli host nell'enclosure di rete privata sicura creata dall'esercitazione [Isola i carichi di lavoro con una rete privata sicura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) ad accedere ai servizi internet pubblici. È necessario prima completare tale esercitazione. 

## Documenta i servizi Internet
{: #Document_outbound}

Il primo passo consiste nell'identificare i servizi a cui si accederà su internet pubblico e documentare le porte che devono essere abilitate per il traffico in uscita e quello in entrata corrispondente da internet. Questo elenco di porte sarà necessario per le regole del firewall in un passo successivo. 

In questo esempio, sono abilitate solo le porte http e https poiché coprono la maggior parte dei requisiti. I servizi DNS e NTP sono forniti dalla rete privata {{site.data.keyword.Bluemix_notm}}. Se sono necessari questi e altri servizi, come ad esempio SMTP (Porta 25) o MySQL (Porta 3306), saranno necessarie delle regole del firewall aggiuntive. Le due regole di porta di base sono:

-	Porta 80 (http)
-	Porta 443 (https)

Verifica se il servizio di terze parti supporta l'inserimento in whitelist degli indirizzi di origine. In caso affermativo, l'indirizzo IP pubblico del VRA sarà necessario per configurare il servizio di terze parti per limitare l'accesso al servizio. 


## Mascheramento NAT a Internet 
{: #NAT_Masquerade}

Attieniti alle istruzioni qui riportate per configurare l'accesso a internet esterno per gli host nella zona APP utilizzando il mascheramento NAT. 

1.	Esegui l'SSH nel VRA e entra nella modalità \[edit\] (config):
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2.	Crea le regole SNAT sul VRA, specificando lo stesso `<Subnet Gateway IP>/<CIDR>` come determinato per la sottorete/VLAN della zona APP nella precedente esercitazione di provisioning di VRA. 
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Subnet Gateway IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## Crea i firewall
{: #Create_firewalls}

1.	Crea le regole del firewall 
   ```
   set security firewall name APP-TO-OUTSIDE default-action drop
   set security firewall name APP-TO-OUTSIDE description 'APP traffic to the Internet'
   set security firewall name APP-TO-OUTSIDE default-log

   set security firewall name APP-TO-OUTSIDE rule 90 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 90 action accept
   set security firewall name APP-TO-OUTSIDE rule 90 destination port 80

   set security firewall name APP-TO-OUTSIDE rule 100 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 100 action accept
   set security firewall name APP-TO-OUTSIDE rule 100 destination port 443

   set security firewall name APP-TO-OUTSIDE rule 200 protocol icmp
   set security firewall name APP-TO-OUTSIDE rule 200 icmp type 8
   set security firewall name APP-TO-OUTSIDE rule 200 action accept
   commit
   ```
   {: codeblock}

## Crea la zona e applica le regole
{: #Create_zone}

1.	Crea la zona OUTSIDE per controllare l'accesso a internet esterno.
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	Assegna i firewall per controllare il traffico verso e da internet.
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE
   commit
   save
   ```
   {: codeblock}
3.	Convalida che la VSI nella zona APP può ora accedere ai servizi su Internet. Esegui l'accesso alla VIS locale utilizzando SSH, utilizza ping e curl per convalidare l'accesso icmp e tcp ai siti su Internet.  
   ```bash
   ssh root@<VSI Private IP>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## Rimuovi le risorse
{:removeresources}
Passi da eseguire per rimuovere le risorse create in questa esercitazione. 

Il VRA è su un piano a pagamento mensile. L'annullamento non prevede un rimborso. Ti consigliamo di annullare solo se questo VRA non sarà richiesto di nuovo nel prossimo mese. Se è richiesto un cluster ad alta disponibilità VRA duale, è possibile eseguire l'upgrade di questo singolo VRA nella pagina dei [dettagli del gateway](https://{DomainName}/classic/network/gatewayappliances).
{:tip}  

1. Annulla qualsiasi server virtuale o server bare metal
2. Annulla il VRA
3. Annulla eventuali VLAN aggiuntive mediante il ticket di supporto. 

## Materiale correlato
{:related}

-	[NAT (Network Address Translation) VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[Mascheramento NAT]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[Documentazione supplementare di VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).

