---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# Pianifica, crea e aggiorna ambienti di distribuzione
{: #plan-create-update-deployments}

Quando crei una soluzione, sono molteplici gli ambienti di distribuzione comuni. Riflettono il ciclo di vita di un progetto, dallo sviluppo alla produzione. Questa esercitazione presenta strumenti quali la CLI {{site.data.keyword.Bluemix_notm}} e [Terraform](https://www.terraform.io/) per automatizzare la creazione e la gestione di questi ambienti di distribuzione.
{:shortdesc}

Gli sviluppatori non amano scrivere la stessa cosa due volte. Il principio [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) ne è un esempio. Allo stesso modo, non amano dover operare un gran numero di selezioni in un'interfaccia utente per configurare un ambiente. Di conseguenza, gli script shell sono da lungo tempo utilizzati dagli amministratori di sistema e dagli sviluppatori per automatizzare attività ripetitive, inclini ad errori e poco interessanti.

IaaS (Infrastructure as a Service), PaaS (Platform as a Service), CaaS (Container as a Service) e FaaS (Functions as a Service) hanno dato agli sviluppatori un elevato livello di astrazione ed è diventato più facile acquisire risorse quali server bare metal, database gestiti, macchine virtuali, cluster Kubernetes ecc. Dopo aver eseguito il provisioning di queste risorse, però, è necessario connetterle tra di loro, configurare l'accesso degli utenti, aggiornare la configurazione nel corso del tempo e così via. Potere automatizzare tutte queste operazioni e ripetere installazione e configurazione in ambienti differenti è oggi una cosa indispensabile.

Molteplici ambienti sono abbastanza comuni in un progetto per supportare le diverse fasi del ciclo di sviluppo con delle leggere differenze tra gli ambienti come la capacità, la rete, le credenziali e il livello di dettaglio dei log. In [quest'altra esercitazione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications), abbiamo presentato le prassi ottimali per organizzare utenti, team e applicazioni, nonché uno scenario di esempio. Lo scenario di esempio tiene conto di tre ambienti, sviluppo (*Development*), esecuzione di test (*Testing*) e produzione (*Production*). Come automatizzare la creazione di questi ambienti? Quali strumenti possono essere utilizzati?

## Obiettivi
{: #objectives}

* Definire una serie di ambienti da distribuire
* Scrivere gli script utilizzando la CLI {{site.data.keyword.Bluemix_notm}} e [Terraform](https://www.terraform.io/) per automatizzare la distribuzione di questi ambienti
* Distribuire questi ambienti nel tuo account

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti prodotti:
* [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Identity and Access Management](https://{DomainName}/iam/#/users)
* [CLI (command line interface) {{site.data.keyword.Bluemix_notm}} - la CLI `ibmcloud`](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. Viene creata una serie di file Terraform per descrivere l'IaC (infrastructure-as-code) di destinazione.
1. Un operatore utilizza `terraform apply` per eseguire il provisioning degli ambienti.
1. Gli script shell sono scritti per completare la configurazione degli ambienti.
1. L'operatore esegue gli script sugli ambienti
1. Gli ambienti sono completamente configurati e sono pronti per l'uso.

## Panoramica degli strumenti disponibili
{: #tools}

Il primo strumento a interagire con {{site.data.keyword.Bluemix_notm}} e creare delle distribuzioni ripetibili è la CLI (command line interface) [{{site.data.keyword.Bluemix_notm}}, la CLI `ibmcloud`](/docs/cli?topic=cloud-cli-install-ibmcloud-cli). Con `ibmcloud` e i relativi plugin, puoi automatizzare la creazione e la configurazione delle tue risorse cloud. Dalla riga di comando, puoi eseguire il provisioning di {{site.data.keyword.virtualmachinesshort}}, cluster Kubernetes, {{site.data.keyword.openwhisk_short}} e applicazioni e servizi Cloud Foundry.

Un altro strumento presentato in [questa esercitazione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform) è [Terraform](https://www.terraform.io/) di HashiCorp. È la stessa HashiCorp che ci dice che *Terraform ti consente di creare, modificare e migliorare l'infrastruttura in modo sicuro e prevedibile. È uno strumento open source che codifica le API in file di configurazione dichiarativi che possono essere condivisi tra membri del team, trattati come codice, modificati, revisionati e controllati per quanto riguarda la versione* Si tratta di IaC (infrastructure-as-code). Tu scrivi come dovrebbe presentarsi l'infrastruttura e Terraform creerà, aggiornerà e rimuoverà le risorse cloud come necessario.

Per supportare un approccio multi-cloud, Terraform lavora con i provider. Un provider è responsabile della comprensione delle interazioni API e dell'esposizione delle risorse. {{site.data.keyword.Bluemix_notm}} ha [un suo provider per Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm), consentendo agli utenti di {{site.data.keyword.Bluemix_notm}} di gestire le risorse con Terraform. Anche se Terraform è categorizzato come IaC (infrastructure-as-code), non è limitato alle risorse IaaS (Infrastructure-As-A-Service). Il {{site.data.keyword.Bluemix_notm}} Provider for Terraform supporta risorse IaaS (bare metal, macchina virtuale, servizi di rete ecc.), CaaS ({{site.data.keyword.containershort_notm}} e cluster Kubernetes), PaaS (Cloud Foundry e servizi) e FaaS ({{site.data.keyword.openwhisk_short}}).

## Scrivi script per automatizzare la distribuzione
{: #scripts}

Quando inizi a descrivere la tua IaC (infrastructure-as-code), è fondamentale trattare i file che crei come codice regolare, archiviandoli quindi in un sistema di gestione del controllo di origine. Nel corso del tempo, ciò si tradurrà in caratteristiche positive, quali l'utilizzo del flusso di lavoro di riesame del controllo di origine per convalidare le modifiche prima di applicarle, aggiungendo una pipeline di integrazione continua per distribuire automaticamente le modifiche all'infrastruttura.

[Questo repository Git](https://github.com/IBM-Cloud/multiple-environments-as-code) ha tutti i file di configurazione necessari per configurare gli ambienti definiti in precedenza. Puoi clonare il repository per seguire le prossime sezioni che descrivono in dettaglio il contenuto dei file.

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

Il repository è strutturato nel seguente modo:

| Directory | Descrizione |
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) | Ubicazione dei file Terraform |
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) | File Terraform per eseguire il provisioning delle risorse comuni ai tre ambienti |
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) | File Terraform specifici per un dato ambiente |
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) | File Terraform per configurare le politiche utente |

### Attività complesse con Terraform

Gli ambienti di sviluppo (*Development)*, esecuzione di test (*Testing*) e produzione (*Production*) sembrano più o meno uguali.

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="Diagramma che mostra un ambiente di distribuzione" />
</p>

Condividono un'organizzazione comune e risorse specifiche per l'ambiente. Presenteranno delle differenze per quanto riguarda la capacità allocata e i diritti di accesso. I file terraform riflettono tale condizione con una configurazione globale (***global***= per eseguire il provisioning dell'organizzazione Cloud Foundry e una configurazione per ogni ambiente (***per-environment***), utilizzando gli spazi di lavoro Terraform, per eseguire il provisioning di risorse specifiche per l'ambiente:

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### Configurazione globale

Tutti gli ambienti condividono un'organizzazione Cloud Foundry comune e ogni ambiente ha il suo spazio.

Nella directory [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global), trovi gli script Terraform per eseguire il provisioning di questa organizzazione. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf) contiene la definizione per l'organizzazione:

   ```sh
   # create a new organization for the project
   resource "ibm_org" "organization" {
     name             = "${var.org_name}"
     managers         = "${var.org_managers}"
     users            = "${var.org_users}"
     auditors         = "${var.org_auditors}"
     billing_managers = "${var.org_billing_managers}"
   }
   ```

In questa risorsa, tutte le proprietà sono configurate tramite variabili. Nelle sezioni successive, imparerai come impostare queste variabili.

Per distribuire completamente gli ambienti, utilizzerai una combinazione di Terraform e CLI {{site.data.keyword.Bluemix_notm}}. Gli script shell scritti con la CLI potrebbero aver bisogno di fare riferimento a questa organizzazione o all'account in base a nome o ID. La directory *global* include anche [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf) che produrrà un file che contiene queste informazioni come chiavi/valori adatti per essere riutilizzati nella creazione di script:

   ```sh
   # generate a property file suitable for shell scripts with useful variables relating to the environment
   resource "local_file" "output" {
     content = <<EOF
ACCOUNT_GUID=${data.ibm_account.account.id}
ORG_GUID=${ibm_org.organization.id}
ORG_NAME=${var.org_name}
EOF

     filename = "../outputs/global.env"
   }
   ```

### Singoli ambienti

Ci sono diversi approcci per gestire più ambienti con Terraform. Puoi duplicare i file Terraform in directory separate, una per ogni ambiente. Con i [moduli Terraform](https://www.terraform.io/docs/modules/index.html) puoi mettere insieme la configurazione comune in un gruppo e riutilizzare i moduli negli ambienti, riducendo la duplicazione di codice. Directory separate significano che puoi modificare l'ambiente di sviluppo (*development*) per eseguire dei test delle modifiche e propagare quindi le modifiche ad altri ambienti. È comune, in questo caso, avere anche i *moduli* Terraform in un loro repository di codice sorgente in modo che tu possa fare riferimento a una specifica versione di un modulo nei tuoi file di ambiente.

Dato che gli ambienti sono piuttosto semplici e simili, utilizzerai un altro concetto di Terraform denominato [spazio di lavoro](https://www.terraform.io/docs/state/workspaces.html#best-practices). Gli spazi di lavoro ti consentono di utilizzare gli stessi file terraform (.tf) con ambienti diversi. Nell'esempio, *development*, *testing* e *production* sono spazi di lavoro. Utilizzeranno le stesse definizioni Terraform ma con variabili di configurazione differenti (nomi differenti, capacità differenti).

Ogni ambiente richiede:
* uno spazio Cloud Foundry dedicato
* un gruppo di risorse dedicato
* un cluster Kubernetes
* un database
* un'archiviazione file

Lo spazio Cloud Foundry è collegato all'organizzazione creata nel passo precedente. I file Terraform dell'ambiente devono fare riferimento a questa organizzazione. In questo contesto, sarà di aiuto lo stato remoto ([remote state](https://www.terraform.io/docs/state/remote.html)) di Terraform. Consente di fare riferimento a uno stato Terraform esistente in modalità di sola lettura. Si tratta di un costrutto molto utile per suddividere la tua configurazione di Terraform in parti più piccole, lasciando la responsabilità delle singole parti a team differenti. [backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf) contiene la definizione dello stato remoto globale (*global*) utilizzato per trovare l'organizzazione creata in precedenza:

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

Una volta che puoi fare riferimento all'organizzazione, è semplice creare uno spazio all'interno di questa organizzazione. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf) contiene la definizione delle risorse per l'ambiente. 

   ```sh
   # a Cloud Foundry space per environment
   resource "ibm_space" "space" {
     name       = "${var.environment_name}"
     org        = "${data.terraform_remote_state.global.org_name}"
     managers   = "${var.space_managers}"
     auditors   = "${var.space_auditors}"
     developers = "${var.space_developers}"
   }
   ```

Nota come si fa riferimento al nome dell'organizzazione dal remote state (stato remoto) *global* (globale). Le altre proprietà vengono prese dalle variabili di configurazione.

Viene quindi il gruppo di risorse.

   ```sh
   # a resource group
   resource "ibm_resource_group" "group" {
    name     = "${var.environment_name}"
    quota_id = "${data.ibm_resource_quota.quota.id}"
}

   data "ibm_resource_quota" "quota" {
	name = "${var.resource_quota}"
}
   ```

Il cluster Kubernetes viene creato in questo gruppo di risorse. Il provider {{site.data.keyword.Bluemix_notm}} ha una risorsa Terraform per rappresentare un cluster:

   ```sh
  # a cluster
  resource "ibm_container_cluster" "cluster" {
  	name              = "${var.environment_name}-cluster"
  	datacenter        = "${var.cluster_datacenter}"
  	org_guid          = "${data.terraform_remote_state.global.org_guid}"
  	space_guid        = "${ibm_space.space.id}"
  	account_guid      = "${data.terraform_remote_state.global.account_guid}"
  	hardware          = "${var.cluster_hardware}"
  	machine_type      = "${var.cluster_machine_type}"
  	public_vlan_id    = "${var.cluster_public_vlan_id}"
  	private_vlan_id   = "${var.cluster_private_vlan_id}"
  	resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool" "cluster_workerpool" {
  worker_pool_name  = "${var.environment_name}-pool"
  machine_type      = "${var.cluster_machine_type}"
  cluster           = "${ibm_container_cluster.cluster.id}"
  size_per_zone     = "${var.worker_num}"
  hardware          = "${var.cluster_hardware}"
  resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool_zone_attachment" "cluster_zone" {
  cluster           = "${ibm_container_cluster.cluster.id}"
  worker_pool       =  "${element(split("/",ibm_container_worker_pool.cluster_workerpool.id),1)}"
  zone              = "${var.cluster_datacenter}"
  public_vlan_id    = "${var.cluster_public_vlan_id}"
  private_vlan_id   = "${var.cluster_private_vlan_id}"
  resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

Ancora una volta, la maggior parte delle proprietà verrà inizializzata dalle variabili di configurazione. Puoi regolare il datacenter, il numero di nodi di lavoro e il tipo di nodi di lavoro.

I servizi abilitati a IAM come {{site.data.keyword.cos_full_notm}} e {{site.data.keyword.cloudant_short_notm}} sono creati come risorse anche all'interno del gruppo:

   ```sh
# a database
resource "ibm_resource_instance" "database" {
    name              = "database"
    service           = "cloudantnosqldb"
    plan              = "${var.cloudantnosqldb_plan}"
    location          = "${var.cloudantnosqldb_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
# a cloud object storage
resource "ibm_resource_instance" "objectstorage" {
    name              = "objectstorage"
    service           = "cloud-object-storage"
    plan              = "${var.cloudobjectstorage_plan}"
    location          = "${var.cloudobjectstorage_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

I bind (segreti) Kubernetes possono essere aggiunti per richiamare le credenziali del servizio dalle tue applicazioni:

   ```sh
   # bind the cloudant service to the cluster
   resource "ibm_container_bind_service" "bind_database" {
      cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	  service_instance_name       = "${ibm_resource_instance.database.name}"
      namespace_id                = "default"
      account_guid                = "${data.terraform_remote_state.global.account_guid}"
      org_guid                    = "${data.terraform_remote_state.global.org_guid}"
      space_guid                  = "${ibm_space.space.id}"
      resource_group_id           = "${ibm_resource_group.group.id}"
}

   # bind the cloud object storage service to the cluster
   resource "ibm_container_bind_service" "bind_objectstorage" {
  	cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	space_guid                  = "${ibm_space.space.id}"
  	service_instance_id         = "${ibm_resource_instance.objectstorage.name}"
  	namespace_id                = "default"
  	account_guid                = "${data.terraform_remote_state.global.account_guid}"
  	org_guid                    = "${data.terraform_remote_state.global.org_guid}"
  	space_guid                  = "${ibm_space.space.id}"
  	resource_group_id           = "${ibm_resource_group.group.id}"
}
   ```

## Distribuisci questo ambiente nel tuo account

### Installa la CLI {{site.data.keyword.Bluemix_notm}}

1. Attieniti a [queste istruzioni](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) per installare la CLI
1. Convalida l'installazione eseguendo:
   ```sh
   ibmcloud
   ```
   {: codeblock}

### Installa Terraform e il {{site.data.keyword.Bluemix_notm}} Provider for Terraform

1. [Scarica e installa Terraform per il tuo sistema.](https://www.terraform.io/intro/getting-started/install.html)
1. [Scarica il file binario Terraform per il provider {{site.data.keyword.Bluemix_notm}}.](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)    Per configurare Terraform con il provider {{site.data.keyword.Bluemix_notm}}, fai riferimento a questo [collegamento](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup)
   {:tip}
1. Crea un file `.terraformrc` nella tua home directory che punta al file binario Terraform. Nel seguente esempio, `/opt/provider/terraform-provider-ibm` è l'instradamento alla directory.
   ```sh
   # ~/.terraformrc
   providers {
     ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### Ottieni il codice

Se non lo hai già fatto, clona il repository dell'esercitazione.

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### Imposta la chiave API della piattaforma

1. Se non ne hai già una, ottieni una [chiave API della piattaforma](https://{DomainName}/iam/#/apikeys) e salvala per ogni esigenza futura.

   > Se in passi successivi intendi creare una nuova organizzazione Cloud Foundry per ospitare gli ambienti di distribuzione, assicurati di essere il proprietario dell'account.
1. Copia [terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl) in *terraform/credentials.tfvars* eseguendo questo comando
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. Modifica `terraform/credentials.tfvars` e imposta il valore per `ibmcloud_api_key` sulla chiave API della piattaforma che hai ottenuto.

### Crea o riutilizza un'organizzazione Cloud Foundry

Puoi scegliere di creare una nuova organizzazione o di riutilizzare (importare) una esistente. Per creare l'organizzazione parent dei tre ambienti di distribuzione, **devi essere il proprietario dell'account**.

#### Per creare una nuova organizzazione

1. Passa alla directory `terraform/global`
1. Copia [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) in `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Modifica `global.tfvars`
   1. Imposta **org_name** sul nome dell'organizzazione da creare
   1. Imposta **org_managers** su un elenco di ID utente a cui desideri concedere il ruolo di gestore (*Manager*= nell'organizzazione; l'utente che crea l'organizzazione è automaticamente un gestore e non deve essere aggiunto all'elenco
   1. Imposta **org_users** su un elenco di tutti gli utenti che vuoi invitare nell'organizzazione; gli utenti devono essere aggiunti qui se vuoi configurare il loro accesso in ulteriori passi

   ```sh
   org_name = "a-new-organization"
   org_managers = [ "user1@domain.com", "another-user@anotherdomain.com" ]
   org_users = [ "user1@domain.com", "another-user@anotherdomain.com", "more-user@domain.com" ]
   ```
   {: codeblock}
1. Inizializza Terraform dalla cartella `terraform/global`
   ```sh
   terraform init
   ```
   {: codeblock}
1. Guarda il piano Terraform
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. Applica le modifiche
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Al completamento dell'esecuzione di Terraform, avrai creato:
* una nuova organizzazione Cloud Foundry
* un file `global.env` nella directory `outputs` nel tuo checkout. Questo file ha delle variabili di ambiente a cui puoi fare riferimento in altri script
* il file `terraform.tfstate`

> Questa esercitazione utilizza il provider di backend `locale` per lo stato Terraform. Questo è comodo quando stai acquisendo dimestichezza con Terraform o quando lavori da solo su un progetto; quando però lavori in un team, o su un'infrastruttura più grande, Terraform supporta anche il salvataggio dello stato in un'ubicazione remota. Dato che lo stato Terraform è critico per le operazioni di Terraform, ti consigliamo di utilizzare un'archiviazione resiliente, altamente disponibile e remota per lo stato Terraform; per un elenco delle opzioni disponibili, consulta il documento [Backend Types](https://www.terraform.io/docs/backends/types/index.html) di Terraform. Alcuni backend supportano anche il controllo della versione e il blocco degli stati Terraform.

#### Per riutilizzare un'organizzazione che stai gestendo

Se non sei il proprietario dell'account ma gestisci un'organizzazione nell'account, puoi anche importare un'organizzazione esistente in Terraform

1. Richiama il GUID dell'organizzazione
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. Passa alla directory `terraform/global`
1. Copia [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) in `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Inizializza Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. Dopo l'inizializzazione di Terraform, importa l'organizzazione nello stato Terraform
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. Ottimizza `global.tfvars` per una corrispondenza con il nome e la struttura dell'organizzazione esistente
1. Applica le modifiche
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### Crea spazio, cluster e servizi per ogni ambiente

Questa sezione si concentrerà sull'ambiente di sviluppo (`development`). I passi saranno gli stessi per gli altri ambienti; solo i valori che scegli per le variabili saranno diversi.

1. Passa alla cartella `terraform/per-environment` del checkout
1. Copia il file `tfvars` di template. Ce n'è uno per ogni ambiente:
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. Modifica `development.tfvars`
   1. Imposta **nome_ambiente** sul nome dello spazio Cloud Foundry che desideri creare
   1. Imposta **space_sviluppatori** sull'elenco di sviluppatori per questo spazio. **Assicurati di aggiungere il tuo nome all'elenco in modo che Terraform possa eseguire il provisioning di servizi per tuo conto.**
   1. Imposta **cluster_datacenter** sull'ubicazione dove desideri creare il cluster. Trovare le ubicazioni disponibili con:
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. Imposta la VLAN privata (**cluster_private_vlan_id**) e quella pubblica (**cluster_public_vlan_id**) per il cluster. Trova le VLAN disponibili per l'ubicazione con:
      ```sh
      ibmcloud cs vlans <location>
      ```
      {: codeblock}
   1. Imposta il tipo di macchina del cluster (**cluster_machine_type**). Trova le caratteristiche e i tipi di macchina disponibili per l'ubicazione con:
      ```sh
      ibmcloud cs machine-types <location>
      ```
      {: codeblock}

   1. Imposta la quota di risorse (**resource_quota**. Trova le definizioni di quota di risorse disponibili con:
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. Inizializza Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. Crea un nuovo spazio di lavoro Terraform per l'ambiente di sviluppo (*development*)
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   Successivamente, per passare da un ambiente all'altro, usa
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. Guarda il piano Terraform
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Dovrebbe notificare:
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. Applica le modifiche
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Al completamento dell'esecuzione di Terraform, avrai creato:
* un gruppo di risorse
* uno spazio Cloud Foundry
* un cluster Kubernetes con un pool di nodi di lavoro e una zona a esso collegati
* un database
* un segreto di Kubernetes con le credenziali del database
* un'archiviazione
* un segreto Kubernetes con le credenziali di archiviazione
* un'istanza logging(LogDNA)
* un'istanza monitoring(Sysdig)
* un file `development.env` nella directory `outputs` nel tuo checkout. Questo file ha delle variabili di ambiente a cui puoi fare riferimento in altri script
* il `terraform.tfstate` specifico per l'ambiente in `terraform.tfstate.d/development`.

Puoi ripetere i passi per `testing` e `production`.

### Assegna le politiche utente

Nei passi precedenti, i ruoli nell'organizzazione e negli spazi Cloud Foundry potevano essere configurati con il provider Terraform. Per le politiche utente su altre risorse come i cluster Kubernetes, utilizzerai la cartella [roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) nel repository clonato.

Per l'ambiente di sviluppo (*Development*) come definito in [questa esercitazione](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications), le politiche da definire sono:

|           | Politiche di accesso IAM |
| --------- | ----------- |
| Developer (Sviluppatore) | <ul><li>Gruppo di risorse: *Viewer* (Visualizzatore)</li><li>Ruoli di accesso alla piattaforma nel gruppo di risorse: *Viewer* (Visualizzatore)</li><li>Ruolo del servizio Logging & Monitoring (Registrazione e monitoraggio): *Writer* (Scrittore)</li></ul> |
| Tester    | <ul><li>Non è necessaria alcuna configurazione. Il tester accede all'applicazione distribuita, non agli ambienti di sviluppo</li></ul> |
| Operator (Operatore) | <ul><li>Gruppo di risorse: *Viewer* (Visualizzatore)</li><li>Ruoli di accesso alla piattaforma nel gruppo di risorse: *Operator* (Operatore), *Viewer* (Visualizzatore)</li><li>Ruolo del servizio Logging & Monitoring (Registrazione e monitoraggio): *Writer* (Scrittore)</li></ul> |
| Pipeline Functional User (Utente funzionale della pipeline) | <ul><li>Gruppo di risorse: *Viewer* (Visualizzatore)</li><li>Ruoli di accesso alla piattaforma nel gruppo di risorse: *Editor*, *Viewer* (Visualizzatore)</li></ul> |

Dato che un team può essere composto da diversi sviluppatori e tester, puoi avvalerti del [concetto di gruppo di accesso](https://{DomainName}/docs/iam?topic=iam-groups#groups) per semplificare la configurazione delle politiche utente. I gruppi di accesso possono essere creati dal proprietario dell'account in modo che lo stesso accesso possa essere assegnato a tutte le entità all'interno del gruppo con una singola politica.

Per il ruolo di sviluppatore (*Developer*) nell'ambiente di sviluppo (*Development*), questo si traduce in:

   ```sh
resource "ibm_iam_access_group" "developer_role" {
  name        = "${var.access_group_name_developer_role}"
  description = "${var.access_group_description}"
}
resource "ibm_iam_access_group_policy" "resourcepolicy_developer" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Viewer"]

  resources = [{
    resource_type = "resource-group"
    resource      = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_platform_accesspolicy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles        = ["Viewer"]

  resources = [{
    resource_group_id = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_logging_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "logdna"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.logdna_instance_id}"
  }]
}
resource "ibm_iam_access_group_policy" "developer_monitoring_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "sysdig-monitor"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.sysdig_instance_id}"
  }]
}

   ```

Il file [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) del checkout ha degli esempi di queste risorse per i ruoli di sviluppatore (*Developer*), operatore (*Operator*), tester (*Tester*) e utente funzionale (*Functional User*) definiti. Per impostare le politiche come sono definite in una sezione precedente per gli utenti con i ruoli di *sviluppatore (Developer), operatore (Operator), tester (Tester) e utente funzionale (Functional User)* nell'ambiente di sviluppo (*development*).

1. Passa alla directory `terraform/roles/development`
2. Copia il file `tfvars` di template. Ce n'è uno per ogni ambiente (puoi trovare i template `production` e `testing` nelle loro rispettive cartella nella directory `roles`)

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. Modifica `development.tfvars`

   - Imposta **iam_access_members_developers** sull'elenco di sviluppatori a cui vuoi concedere l'accesso.
   - Imposta **iam_access_members_operators** sull'elenco di operatori e così via.
4. Inizializza Terraform
   ```sh
   terraform init
   ```
   {: codeblock}

5. Guarda il piano Terraform
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Dovrebbe notificare:
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. Applica le modifiche
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
Puoi ripetere i passi per `testing` e `production`.

## Rimuovi le risorse

1. Vai alla cartella `development` in `roles`
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. Elimina i gruppi di accesso e le politiche di accesso
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Attiva lo spazio di lavoro `development`
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. Elimina il gruppo di risorse, gli spazi, i servizi e i cluster
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Ripeti i passi per gli spazi di lavoro `testing` e `production`
1. Se l'hai creata, elimina l'organizzazione
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## Contenuto correlato

* [Esercitazione di Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Provider Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
* [Esempi che utilizzano {{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
