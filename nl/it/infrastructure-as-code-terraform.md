---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-23"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Distribuisci uno stack LAMP utilizzando Terraform
{: #infrastructure-as-code-terraform}

[Terraform](https://www.terraform.io/) ti consente di creare, modificare e migliorare l'infrastruttura in modo sicuro e prevedibile. È uno strumento open source che codifica le API in file di configurazione dichiarativi che possono essere condivisi tra membri del team, trattati come codice, modificati, revisionati e controllati per quanto riguarda la versione 

In questa esercitazione, utilizzerai una configurazione di esempio per eseguire il provisioning di un server virtuale **L**inux con il server web **A**pache, **M**ySQL e il server **P**HP, combinazione nota come stack **LAMP**. Aggiornerai quindi la configurazione, aggiungerai il servizio {{site.data.keyword.cos_full_notm}} e ridimensionerai le risorse per regolare l'ambiente (memoria, CPU e dimensione del disco). Termina eliminando tutte le risorse create dalla configurazione.

## Obiettivi
{: #objectives}

* Configurare Terraform e il {{site.data.keyword.Bluemix_notm}} Provider for Terraform.
* Utilizzare Terraform per creare, aggiornare, ridimensionare e, infine, eliminare una configurazione di stack LAMP.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

<p style="text-align: center;">

  ![Diagramma dell'architettura](images/solution10/architecture-2.png)
</p>

1. Viene creata una serie di file Terraform per descrivere la configurazione di stack LAMP.
1. `terraform` viene richiamato dalla directory di configurazione.
1. `terraform` richiama l'API {{site.data.keyword.cloud_notm}} per eseguire il provisioning delle risorse.

## Prima di iniziare
{: #prereqs}

Contatta il tuo utente master dell'infrastruttura per ottenere le seguenti autorizzazioni:
- Rete (per aggiungere **Public and Private Network Uplink**)
- Chiave API

## Prerequisiti

{: #prereq}

Installa **Terraform** tramite il [programma di installazione](https://www.terraform.io/intro/getting-started/install.html) oppure utilizza [Homebrew](https://brew.sh/) su macOS eseguendo il comando: `brew install terraform`

Su **Windows**, attieniti alla seguente procedura per completare la configurazione di terraform:
   1. Copia i file dallo zip scaricato in `C:\terraform` (crea una cartella `terraform`).
   2. Apri il prompt dei comandi come un amministratore e imposta il PATH per utilizzare i file binari di terraform.

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## Configura il {{site.data.keyword.Bluemix_notm}} Provider for Terraform
{: #setup}

Per supportare un approccio multi-cloud, Terraform lavora con i provider. Un provider è responsabile della comprensione delle interazioni API e dell'esposizione delle risorse. In questa sezione, configurerai la CLI per specificare l'ubicazione del provider {{site.data.keyword.Bluemix_notm}}.

1. Controlla l'installazione di Terraform eseguendo `terraform` nella tua finestra del terminale o del prompt di comandi. Dovresti vedere un elenco di comandi comuni (`Common commands`).

  ```
  terraform
  ```

2. Scarica il plugin [{{site.data.keyword.Bluemix_notm}} Provider](https://github.com/IBM-Cloud/terraform-provider-ibm/releases) appropriato per il tuo sistema ed estrai l'archivio. Dovresti vedere un file di plugin binario `terraform-provider-ibm_VERSION`.

3. Per i sistemi non Windows, crea una directory `.terraform.d/plugins` nella tua directory home utente e inserisci al suo interno il file binario. Utilizza i seguenti comandi come riferimento.

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    Su **Windows**, il file deve essere inserito in `terraform.d/plugins` nella tua directory "AppData".

  - Esegui i comandi qui di seguito indicati in un prompt dei comandi [Provider Configuration](https://www.terraform.io/docs/configuration/providers.html)
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - Avvia **Windows Powershell** (Start + R > Powershell) ed esegui questo comando per creare il file `terraform.rc`
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   Al primo prompt, immetti il seguente contenuto
   ```
    # ~/.terraformrc
    providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        PATH_TO_YOUR_APPDATA_PLUGINS deve essere un percorso assoluto con le barre (/). Ad esempio, `C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe`
        {: tip}

  - Fai clic su Invio per uscire dal prompt.

## Prepara la configurazione di terraform

{: #terraformconfig}

In questa sezione, imparerai le basi di una configurazione terraform utilizzando una configurazione Terraform di esempio fornita da {{site.data.keyword.Bluemix_notm}}.

1. Visita https://github.com/IBM-Cloud/LAMP-terraform-ibm e biforca (**Fork**) una tua copia nel tuo account.
2. Clona la tua biforcazione localmente:
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. Ispeziona i file di configurazione
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml) - contiene le configurazioni di installazione del server, qui è dove puoi aggiungere tutti gli script correlati a quanto installi sul server. Vedi `phpinfo();` inserito in questo file.
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf) - contiene le variabili correlate al provider dove sono necessari la chiave api e il nome utente del provider.
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) - contiene le configurazioni server per distribuire la VM con le variabili specificate.
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) - contiene la chiave api e il nome utente **SoftLayer**, la chiave API {{site.data.keyword.Bluemix_notm}} e i tuoi nomi spazio/organizzazione. Come prassi ottimale, queste credenziali possono essere aggiunte a questo file per evitare di reimmetterle dalla riga di comando a ogni distribuzione del server. Nota: NON pubblicare questo file con le tue credenziali.
4. Apri il file [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) in un IDE di tua scelta e modificalo aggiungendo la tua chiave **SSH pubblica**. Verrà utilizzata per accedere alla VM creata da questa configurazione. Per informazioni sulla creazione di chiavi SSH, attieniti alle istruzioni in [questo collegamento](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html) Per copiare la chiave pubblica nei tuoi appunti, puoi eseguire questo comando nel tuo terminale.

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     Questo comando copierà l'SSH nei tuoi appunti e potrai quindi incollarlo in [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) nella variabile predefinita `ssh_key`, circa alla riga 69.

    Su **Windows**, scarica, installa e avvia [Git Bash](https://git-scm.com/download/win) ed esegui questo comando per copiare la chiave SSH pubblica nei tuoi appunti.

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. Apri il file [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) con il tuo IDE, modifica il file aggiungendo le credenziali elencate, operazione grazie alla quale non hai bisogno di reimmettere tali credenziali ogni volta che esegui terraform apply. Devi aggiungere tutte e cinque le credenziali elencate nel file [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) per completare il resto di questa esercitazione.

## Crea un server stack LAMP dalla configurazione terraform
{: #Createserver}
In questa sezione, imparerai come creare un server stack LAMP dall'esempio di configurazione terraform. La configurazione viene utilizzata per eseguire il provisioning di un'istanza della macchina virtuale e installare **A**pache, **M**ySQL (**M**ariaDB) e **P**HP in tale istanza.

1. Vai alla cartella del repository che hai clonato.
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. Inizializza la configurazione terraform. Questo installerà anche il plugin `terraform-provider-ibm_VERSION`.
   ````bash
    terraform init
   ````
   {: pre}
3. Applica la configurazione terraform. Questo creerà le risorse definite nella configurazione.
   ```
    terraform apply
   ```
   {: pre}
   Dovresti vedere un output simile al seguente ![URL di controllo origine](images/solution10/created.png)
4. Vai quindi all'[elenco dispositivi dell'infrastruttura](https://{DomainName}/classic/devices) per verificare che il server sia stato creato. ![URL di controllo origine](images/solution10/configuration.png)

## Aggiungi il servizio {{site.data.keyword.cos_full_notm}} e ridimensiona le risorse

{: #modify}

In questa sezione, vedrai come ridimensionare le risorse del server virtuale e aggiungere un servizio [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) al tuo ambiente dell'infrastruttura.

1. Modifica il file `vm.tf` per aumentare quanto segue e salva quindi il file.
 - Aumenta il numero di core CPU a 4 core
 - Aumenta la RAM (memoria) a 4096
 - Aumenta la dimensione del disco a 100GB

2. Aggiungi quindi un nuovo servizio [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage); per eseguire tale operazione, crea un nuovo file e denominalo **ibm-cloud-object-storage.tf**. Aggiungi i frammenti di codice di seguito indicati al file appena creato. I frammenti di codice di seguito indicati creano un nome di variabile per il nome organizzazione e il nome spazio; questi due nomi di variabile vengono utilizzati quindi per richiamare il guid spazio in cui è necessario creare il servizio. Imposta il nome servizio {{site.data.keyword.cos_full_notm}} su `lamp_objectstorage`; ti serve quindi un guid spazio, un nome servizio completo e un tipo di piano. Il codice di seguito creerà un piano premium, che è un piano a pagamento. Puoi anche utilizzare il piano Lite, ma nota che il piano Lite è limitato a solo un singolo servizio per ogni account.

   ```
   variable "org_name" {
     description = "Enter your IBM Cloud org name, you can get your org name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   variable "space_name" {
     description = "Enter your IBM Cloud space name, you can get your space name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   data "ibm_space" "space" {
     space = "${var.space_name}"
     org   = "${var.org_name}"
   }

   # a cloud object storage
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # you can only have one Lite plan per account so let's use the Premium - it is pay-as-you-go
     plan = "Premium"
   }
   ```
   {: pre}
   **Nota:** in seguito cercheremo quindi l'etichetta "lamp_objectstorage", nei log per assicurarci che {{site.data.keyword.cos_full_notm}} sia stato creato correttamente.

3. Inizializza nuovamente la configurazione terraform eseguendo:

   ```bash
    terraform init
   ```
   {: pre}

4. Applica le modifiche terraform eseguendo:
   ```bash
    terraform apply
   ```
   {: pre}
   **Nota:** dopo la corretta esecuzione del comando terraform apply, dovresti vedere un nuovo file `terraform.tfstate`. aggiunto alla tua directory. Questo file contiene la conferma di distribuzione completa per tenere traccia di quanto hai applicato per ultimo e delle eventuali future modifiche alla tua configurazione. Se questo file viene rimosso o se va perduto, perderai le tue configurazioni di distribuzione di terraform.

## Verifica VM e {{site.data.keyword.cos_short}}
{: #verifyvm}

In questa sezione, verificherai la VM e {{site.data.keyword.cos_short}} per assicurati che ne sia stata eseguita la creazione correttamente.

**Verifica la VM**

1. Nel menu sul lato sinistro, fai clic su **Infrastructure** per visualizzare l'elenco di dispositivi server virtuali.
2. Fai clic su **Devices** -> **Device List** per trovare il server creato. Dovresti vedere il tuo dispositivo server elencato.
3. Fai clic sul server per visualizzare ulteriori informazioni sulla sua configurazione. Guardando l'acquisizione di schermo qui di seguito, possiamo vedere che il server è stato creato con esito positivo. ![URL di controllo origine](images/solution10/configuration.png)
4. Verifichiamo ora il server nel browser web. Apri l'indirizzo IP pubblico del server nel browser web. Dovresti vedere la pagina di installazione predefinita del server come qui di seguito. ![URL di controllo origine](images/solution10/LAMP.png)


**Verifica {{site.data.keyword.cos_full_notm}}**

1. Dal **dashboard {{site.data.keyword.Bluemix_notm}}**, dovresti vedere che un'istanza del servizio {{site.data.keyword.cos_full_notm}} è stata creata ed è pronta per l'uso. ![object-storage](images/solution10/ibm-cloud-object-storage.png)

   Ulteriori informazioni su {{site.data.keyword.cos_full_notm}} sono disponibili [qui](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage).

## Rimuovi le risorse
{: #deleteresources}

Elimina le risorse utilizzando il seguente comando:
   ```bash
   terraform destroy
   ```
   {: pre}

**Nota:** per eliminare le risorse, avrai bisogno delle autorizzazioni di amministratore dell'infrastruttura. Se non ha un account di super utente amministratore, richiedi di annullare le risorse utilizzando il dashboard dell'infrastruttura. Puoi richiedere di annullare un dispositivo dal dashboard dell'infrastruttura nei dispositivi. ![object-storage](images/solution10/rm.png)

## Contenuto correlato

- [Terraform](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [{{site.data.keyword.Bluemix_notm}} Provider for Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [Accelera la fornitura di file statici utilizzando una CDN - {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

