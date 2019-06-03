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

# Hosting di applicazioni web da una rete privata sicura
{: #web-app-private-network}

L'hosting delle applicazioni web è un pattern di distribuzione comune per il cloud pubblico in cui le risorse possono essere ridimensionate su richiesta per soddisfare le richieste di utilizzo a breve e a lungo termine. La sicurezza per i carichi di lavoro dell'applicazione è un prerequisito fondamentale per integrare la resilienza e la scalabilità fornite dal cloud pubblico. 

Questa esercitazione crea un'applicazione web Internet scalabile e sicura ospitata in una rete privata utilizzando un VRA (virtual router appliance), VLAN, NAT e firewall. L'applicazione comprende un programma di bilanciamento del carico, due server delle applicazioni web e un server database MySQL. Combina tre esercitazioni che mostrano in che modo le applicazioni web possono essere distribuite in modo sicuro sulla piattaforma {{site.data.keyword.Bluemix_notm}} IaaS utilizzando la rete classica.
{:shortdesc}

## Obiettivi
{: #objectives}

- Creare server virtuali per installare PHP e MySQL
- Eseguire il provisioning di un programma di bilanciamento del carico per distribuire le richieste ai server delle applicazioni
- Distribuire un VRA (Virtual Router Appliance) per creare una rete sicura
- Definire le VLAN e le sottoreti IP 
- Proteggere la rete con le regole firewall
- SNAT (Source Network Address Translation) per la distribuzione dell'applicazione

## Servizi utilizzati
{: #products}

Questa esercitazione utilizza i seguenti servizi {{site.data.keyword.Bluemix_notm}}: 

* [VPN VRA (Virtual Router Appliance)](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [Programma di bilanciamento del carico]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

Questa esercitazione può comportare degli addebiti. Il VRA è disponibile solo su un piano a pagamento mensile. 

## Architettura
{: #architecture}

<p style="text-align: center;">

  ![Architettura](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	Configura una rete privata sicura
2.	Configura l'accesso NAT per la distribuzione dell'applicazione
3.	Distribuisci l'applicazione web scalabile e il programma di bilanciamento del carico

## Prima di iniziare
{: #prereqs}

Questa esercitazione utilizza tre esercitazioni esistenti che vengono distribuite in sequenza. Prima di iniziare, devi esaminarle tutte e tre: 

-	[Isola i carichi di lavoro con una rete privata sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[Configura NAT per l'accesso a Internet da una rete sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[Utilizza i server virtuali per creare un'applicazione web altamente disponibile e scalabile]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## Configura una rete privata sicura
{: #private_network}

Gli ambienti di rete privati sicuri e isolati sono centrali nel modello di sicurezza dell'applicazione IaaS nel cloud pubblico. Firewall, VLAN, instradamento e VPN sono tutti componenti necessari nella creazione di ambienti privati isolati.
Il primo passo consiste nel creare un'enclosure di rete privata sicura all'interno dell'applicazione web che verrà distribuita.   

- [Isola i carichi di lavoro con una rete privata sicura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

Questa esercitazione può essere seguita senza modifiche. In un passo successivo, verranno distribuite tre VM nella zona APP come server delle applicazioni Nginx e un database MySQL. 

## Configura NAT per la distribuzione sicura dell'applicazione
{: #nat_config}

L'installazione delle applicazioni open-source richiede l'accesso sicuro a Internet per accedere ai repository di origine. Per impedire che i server nella rete privata vengano esposti sull'Internet pubblico, viene utilizzato Source NAT dove l'indirizzo di origine viene offuscato e vengono utilizzate le regole firewall per proteggere le richieste in uscita del repository dell'applicazione. Tutte le richieste in entrata vengono negate.  

- [Configura NAT per l'accesso a Internet da una rete sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

Questa esercitazione può essere seguita senza modifiche. Nel passo successivo, la configurazione NAT verrà utilizzata per accedere ai moduli Nginx e MySQL richiesti.  


## Distribuisci l'applicazione web scalabile e il programma di bilanciamento del carico
{: #scalable_app}

Viene utilizzata un'installazione Wordpress su Nginx e MySQL, con un programma di bilanciamento del carico, per illustrare in che modo un'applicazione web scalabile e resiliente può essere distribuita nella rete privata sicura 

Questa esercitazione ti guida in questo scenario con la creazione di un programma di bilanciamento del carico {{site.data.keyword.Bluemix_notm}}, di due server delle applicazioni web e di un server database MySQL. Questi server vengono distribuiti nella zona APP nella rete privata sicura per fornire una separazione firewall tra gli altri carichi di lavoro e la rete pubblica.  

- [Utilizza i server virtuali per creare un'applicazione web altamente disponibile e scalabile]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

Ci sono tre modifiche da questa esercitazione: 

1.	I server virtuali utilizzati in questa esercitazione vengono distribuiti nella VLAN e nella sottorete protette dalla zona del firewall APP dietro il VRA.
2. Specifica l'&lt;ID VLAN privata&gt; quando ordini i server virtuali. Vedi le istruzioni presenti in [Ordina il primo server virtuale](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver) nell'esercitazione [Isola i carichi di lavoro con una rete privata sicura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) per i dettagli su come specificare l'&lt;ID VLAN privata&gt; quando ordini un server virtuale. Inoltre, ricordati di selezionare la tua chiave SSH caricata in precedenza nell'esercitazione per consentire l'accesso ai server virtuali.  
3. Ti consigliamo vivamente di **NON** utilizzare il servizio di archiviazione file per questa esercitazione a causa di prestazioni insoddisfacenti di rsync nel copiare i file Wordpress nell'archiviazione condivisa. Ciò non influisce sull'esercitazione complessiva. I passi correlati alla creazione dell'archiviazione file e alla configurazione dei montaggi possono essere ignorati per i server delle applicazioni e per db. In alternativa, tutti i passi di [Installa e configura l'applicazione PHP sui server delle applicazioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application) devono essere eseguiti su entrambi i server delle applicazioni.
   Prima di completare i passi presenti in [Installa e configura l'applicazione PHP sui server delle applicazioni](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application), crea innanzitutto la directory `/mnt/www/` su entrambi i server delle applicazioni. Questa directory è stata creata originariamente nella sezione dell'archiviazione file ora rimossa.  

   ```sh
   mkdir /mnt/www
   ```

Alla fine di questo passo, il programma di bilanciamento del carico deve avere uno stato integro e il sito Wordpress deve essere accessibile su Internet. I server virtuali che compongono l'applicazione web vengono protetti dall'accesso esterno tramite internet dal firewall VRA e l'unico accesso è tramite il programma di bilanciamento del carico. Per un ambiente di produzione, devi considerare anche la protezione DDoS e WAF (Web Application Firewall) come previsti da [{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services).


## Rimuovi le risorse
{: #removeresources}

Passi da eseguire per rimuovere le risorse create in questa esercitazione. 

Il VRA è su un piano a pagamento mensile. L'annullamento non prevede un rimborso. Ti consigliamo di annullare solo se questo VRA non sarà richiesto di nuovo nel prossimo mese. {:tip}  

1. Annulla qualsiasi server virtuale o server bare metal 
2. Annulla il VRA
3. Annulla eventuali VLAN aggiuntive mediante il ticket di supporto.
4. Elimina il programma di bilanciamento del carico
5. Elimina i servizi File Storage

## Espandi l'esercitazione 

1. In questa esercitazione, inizialmente viene eseguito il provisioning solo di due server virtuali come livello dell'applicazione, possono essere aggiunti automaticamente altri server per gestire ulteriore carico. Il [ridimensionamento automatico]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group) ti fornisce la possibilità di automatizzare il processo di ridimensionamento manuale associato all'aggiunta o alla rimozione dei server virtuali per supportare le tue applicazioni aziendali. 

2. Proteggi separatamente i dati utente aggiungendo una seconda VLAN privata e una sottorete IP al VRA per creare una zona DATA per ospitare il server database MySQL. Configura le regole firewall per consentire solo il traffico IP MySQL sulla porta 3306 in entrata dalla zona APP alla zona DATA.  

