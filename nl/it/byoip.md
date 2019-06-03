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

# BYOIP (Bring Your Own IP)
{: #byoip}

La funzione BYOIP, ossia la possibilità di utilizzare il proprio IP, è un requisito frequente in cui si desidera connettere le reti client esistenti all'infrastruttura di cui è stato eseguito il provisioning su {{site.data.keyword.Bluemix_notm}}. L'intento è in genere quello di ridurre al minimo le modifiche alla configurazione e alle operazioni di instradamento della rete dei client con l'adozione di un singolo spazio di indirizzi IP basato sullo schema di indirizzamento IP esistente dei client.

Questa esercitazione presenta una breve panoramica dei pattern di implementazione BYOIP che possono essere utilizzati con {{site.data.keyword.Bluemix_notm}} e una struttura ad albero delle decisioni per identificare il pattern appropriato quando si realizza l'enclosure sicura come descritto nell'esercitazione [Isola i carichi di lavoro con una rete privata sicura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). La configurazione potrebbe richiedere input aggiuntivi dal tuo team di rete in loco, dal supporto tecnico {{site.data.keyword.Bluemix_notm}} o dai servizi IBM.

{:shortdesc}

## Obiettivi
{: #objectives}

* Comprendere i pattern di implementazione BYOIP
* Selezionare il pattern di implementazione per {{site.data.keyword.Bluemix_notm}}.

## Indirizzamento IP di {{site.data.keyword.Bluemix_notm}}

{{site.data.keyword.Bluemix_notm}} utilizza una serie di intervalli di indirizzi privati, in particolare 10.0.0.0/8, e in alcuni casi questi intervalli possono entrare in conflitto con le reti client esistenti. Laddove esistono conflitti di indirizzi, esistono diversi pattern che supportano BYOIP per consentire l'interoperabilità con la rete IBM Cloud.

-	NAT (Network Address Translation)
-	Tunneling GRE (Generic Routing Encapsulation)
-	Tunneling GRE con alias IP
-	Rete di sovrapposizione virtuale

La scelta del pattern è determinata dalle applicazioni destinate ad essere ospitate su {{site.data.keyword.Bluemix_notm}}. Ci sono due aspetti chiave: la sensibilità dell'applicazione all'implementazione del pattern e l'estensione della sovrapposizione degli intervalli di indirizzi tra la rete client e {{site.data.keyword.Bluemix_notm}}. Si applicheranno anche ulteriori considerazioni se c'è l'intenzione di utilizzare una connessione di rete privata dedicata a {{site.data.keyword.Bluemix_notm}}. La documentazione di [{{site.data.keyword.BluDirectLink}}
](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link) è consigliata per tutti gli utenti interessati all'utilizzo di BYOIP. Per gli utenti di {{site.data.keyword.BluDirectLink}}, le indicazioni associate devono essere seguite in deferenza alle informazioni qui presentate.

## Panoramica dei pattern di implementazione
{: #patterns_overview}

1. **NAT**: traduzione di indirizzi NAT sul router client in loco. Esegui NAT in loco per convertire lo schema di indirizzamento client negli indirizzi IP assegnati da {{site.data.keyword.Bluemix_notm}} ai servizi IaaS forniti.  
2. **Tunneling GRE**: lo schema di indirizzamento viene unificato instradando il traffico IP su un tunnel GRE tra {{site.data.keyword.Bluemix_notm}} e la rete in loco, in genere tramite VPN. Questo è lo scenario illustrato in [questa esercitazione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN). 

   Esistono due pattern secondari a seconda del potenziale di sovrapposizione dello spazio di indirizzi.
     * Nessuna sovrapposizione di indirizzi: quando non c'è sovrapposizione di intervalli di indirizzi e rischio di conflitto tra le reti.
     * Sovrapposizione parziale di indirizzi: quando gli spazi di indirizzi IP del client e di {{site.data.keyword.Bluemix_notm}} utilizzano lo stesso intervallo di indirizzi ed esiste una possibilità di sovrapposizione e conflitto. In questo caso, vengono scelti gli indirizzi di sottorete del client che non si sovrappongono nella rete privata di {{site.data.keyword.Bluemix_notm}}.

3. Tunneling GRE + alias IP
Lo schema di indirizzamento viene unificato instradando il traffico IP su un tunnel GRE tra gli indirizzi IP della rete in loco e alias assegnati ai server su {{site.data.keyword.Bluemix_notm}} . Questo è un caso speciale dello scenario illustrato in [questa esercitazione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN). Un'interfaccia e un alias IP aggiuntivi per una sottorete IP compatibile vengono creati sui server virtuali e bare metal con provisioning su {{site.data.keyword.Bluemix_notm}}, supportati da una configurazione di instradamento appropriata sul VRA.

4. Rete di sovrapposizione virtuale
[{{site.data.keyword.Bluemix_notm}} Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) supporta BYOIP per ambienti completamente virtuali su {{site.data.keyword.Bluemix_notm}}. Potrebbe essere considerata un'alternativa all'enclosure di rete privata sicura descritta in [questa esercitazione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure).

In alternativa, prendi in considerazione una soluzione come VMware NSX che implementa una rete di sovrapposizione virtuale in un livello della rete [{{site.data.keyword.Bluemix_notm}}. Tutti gli indirizzi BYOIP nella sovrapposizione virtuale sono indipendenti dagli intervalli di indirizzi di rete di [{{site.data.keyword.Bluemix_notm}}. Vedi [Introduzione a VMware e [{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud).

## Struttura ad albero delle decisioni del pattern
{: #decision_tree}

La seguente struttura ad albero delle decisioni può essere utilizzata per determinare il pattern di implementazione appropriato. 

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

Le seguenti note forniscono ulteriori indicazioni:

### NAT è problematico per le tue applicazioni?
{: #nat_consideration}

Esistono due casi distinti in cui NAT potrebbe essere problematico. In questi casi, NAT non deve essere utilizzato. 

- Alcune applicazioni come la comunicazione del dominio Microsoft AD e le applicazioni P2P potrebbero avere problemi tecnici con NAT.
- Laddove dei server sconosciuti devono comunicare con [{{site.data.keyword.Bluemix_notm}} o sono richieste centinaia di connessioni bidirezionali tra [{{site.data.keyword.Bluemix_notm}} e i server in loco. In questo caso non è possibile configurare tutta l'associazione sulla tabella router/NAT del client a causa dell'impossibilità di identificare preventivamente l'associazione.


### Nessuna sovrapposizione di indirizzi
{: #no-overlap}

10.0.0.0/8 è utilizzato nella rete in loco? Quando non esiste alcuna sovrapposizione di indirizzi tra la rete in loco e la rete privata [{{site.data.keyword.Bluemix_notm}}, come descritto in [questa esercitazione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN), è possibile utilizzare il tunneling GRE tra la rete in loco e IBM Cloud per evitare la necessità della traduzione NAT. Ciò richiede una revisione dell'utilizzo degli indirizzi di rete con il team di rete in loco. 

### Sovrapposizione parziale di indirizzi
{: #partial_overlap}

Se uno qualsiasi degli intervalli 10.0.0.0/8 è in uso sulla rete in loco, ci sono delle sottoreti senza sovrapposizioni disponibili sulla rete [{{site.data.keyword.Bluemix_notm}}? Esamina l'utilizzo degli indirizzi di rete esistenti con il team di rete in loco e contatta il settore vendite tecniche di [{{site.data.keyword.Bluemix_notm}} per identificare le reti senza sovrapposizioni disponibili. 

### L'aliasing IP è problematico?
{: #ip_alias}

Se non esistono indirizzi sovrapposti, l'aliasing IP può essere implementato su server virtuali e bare metal distribuiti nell'enclosure di rete privata sicura. L'aliasing IP assegna più indirizzi di sottorete su una o più interfacce di rete su ciascun server. 

L'aliasing IP è una tecnica comunemente utilizzata, anche se si consiglia di riesaminare le configurazioni del server e delle applicazioni per determinare se funzionano correttamente in configurazioni di alias multi-home e IP.  

Sarà richiesta una configurazione di instradamento aggiuntiva sul VRA per creare instradamenti dinamici (ad esempio BGP) o instradamenti statici per le sottoreti BYOIP. 

## Contenuto correlato
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [Virtual Private Cloud (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
