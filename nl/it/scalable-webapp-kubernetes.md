---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Applicazione web scalabile su Kubernetes
{: #scalable-webapp-kubernetes}

Questa esercitazione ti spiega come eseguire lo scaffolding di un'applicazione web, eseguirlo in locale in un contenitore e distribuirlo quindi a un cluster Kubernetes creato con [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster). Imparerai inoltre ad eseguire il bind di un dominio personalizzato, monitorare l'integrità dell'ambiente e ridimensionare l'applicazione.
{:shortdesc}

I contenitori sono un modo standard per assemblare applicazioni e tutte le sue dipendenze in modo da poter spostare facilmente le applicazioni tra gli ambienti. A differenza delle macchine virtuali, i contenitori non includono il sistema operativo. I contenitori includono solo il codice dell'applicazione, il runtime, gli strumenti di sistema, le librerie e le impostazioni. I contenitori sono più leggeri, portatili ed efficienti rispetto alle macchine virtuali.

Per gli sviluppatori che desiderano dare il via ai loro progetti, la CLI {{site.data.keyword.dev_cli_notm}} consente di sviluppare e distribuire le applicazioni in tempi rapidi generando delle applicazioni template che puoi eseguire immediatamente o personalizzare come punto di partenza per le tue soluzioni. Oltre a generare il codice dell'applicazione starter, l'immagine del contenitore Docker e le risorse CloudFoundry, i generatori di codice utilizzati dalla CLI di sviluppo e la console web generano i file di ausilio nella distribuzione negli ambienti [Kubernetes](https://kubernetes.io/). I template generano i grafici [Helm](https://github.com/kubernetes/helm) che descrivono la configurazione della distribuzione Kubernetes iniziale dell'applicazione e vengono facilmente estesi per creare distribuzioni multi-immagine o complesse, come necessario.

## Obiettivi
{: #objectives}

* Eseguire lo scaffolding di un'applicazione starter.
* Distribuire l'applicazione al cluster Kubernetes.
* Eseguire il bind di un dominio personalizzato.
* Monitorare i log e l'integrità del cluster.
* Ridimensionare i pod Kubernetes.

## Servizi utilizzati
{: #services}

Questa esercitazione utilizza i seguenti runtime e servizi:
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Questa esercitazione può comportare degli addebiti. Utilizza il [Calcolatore del prezzo](https://{DomainName}/pricing/) per generare una stima dei costi basata sul tuo utilizzo previsto.

## Architettura
{: #architecture}

<p style="text-align: center;">

  ![Architettura](images/solution2/Architecture.png)
</p>

1. Uno sviluppatore genera un'applicazione starter con {{site.data.keyword.dev_cli_notm}}.
1. La creazione dell'applicazione produce un'immagine contenitore Docker.
1. Viene eseguito il push dell'immagine in uno spazio dei nomi in {{site.data.keyword.containershort_notm}}.
1. L'applicazione viene distribuita a un cluster Kubernetes.
1. Gli utenti accedono all'applicazione.

## Prima di iniziare
{: #prereqs}

* [Configura la CLI {{site.data.keyword.registrylong_notm}} e il tuo spazio dei nomi del registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [Installa {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - Script per installare docker, kubectl, helm, CLI ibmcloud e i plugin richiesti
* [Comprendi i principi di base di Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## Crea un cluster Kubernetes
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} offre potenti strumenti combinando le tecnologie Docker e Kubernetes, un'esperienza utente intuitiva e la sicurezza e l'isolamento integrati per automatizzare la distribuzione, il funzionamento, il ridimensionamento e il monitoraggio di applicazioni inserite in un contenitore in un cluster di host di calcolo.

La parte principale di questa esercitazione può essere eseguita con un cluster gratuito (**Free**). Due sezioni facoltative correlate a Ingress Kubernetes e al dominio personalizzato richiedono un cluster a pagamento (**Paid**) di tipo **Standard**.

1. Crea un cluster Kubernetes dal [catalogo {{site.data.keyword.Bluemix}}](https://{DomainName}/containers-kubernetes/launch).

   Per facilità di utilizzo, controlla i dettagli di configurazione come il numero di CPU, la memoria e il numero di nodi di lavoro che ottieni con i piani Lite e Standard.
   {:tip}

   ![Creazione del cluster Kubernetes su IBM Cloud](images/solution2/KubernetesClusterCreation.png)
2. Seleziona il tipo di cluster (**Cluster type**) e fai clic su **Create Cluster** per eseguire il provisioning di un cluster Kubernetes.
3.  Controlla lo stato del tuo **cluster** e **nodo di lavoro** e attendi che diventi **ready**.

### Configura kubectl

In questo passo, configurerai kubectl per puntare al tuo cluster appena creato per il futuro. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) è uno strumento di riga di comando che utilizzi per interagire con un cluster Kubernetes.

1. Utilizza `ibmcloud login` per eseguire l'accesso in modo interattivo. Fornisci l'organizzazione (org), l'ubicazione e lo spazio in cui viene creato il cluster. Puoi riconfermare i dettagli eseguendo il comando `ibmcloud target`.
2. Quando il cluster è pronto, richiama la configurazione del cluster impostando la variabile di ambiente MYCLUSTER sul tuo nome del cluster:
   ```bash
   export MYCLUSTER=<CLUSTER_NAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
   ```
   {: pre}
3. Copia e incolla il comando **export** per impostare la variabile di ambiente KUBECONFIG come indicato. Per verificare se la variabile di ambiente è impostata correttamente o meno, esegui questo comando:
`echo $KUBECONFIG`
4. Controlla che il comando `kubectl` sia configurato correttamente
   ```bash
   kubectl cluster-info
   ```
   {: pre}
   ![](images/solution2/kubectl_cluster-info.png)


## Crea un'applicazione starter
{: #create_application}

La strumentazione `ibmcloud dev` riduce notevolmente i tempi di sviluppo generando starter dell'applicazione con tutto il codice necessario per contenitore tipo, creazione e configurazione in modo che tu possa iniziare a codificare la logica di business più velocemente.

1. Avvia la procedura guidata `ibmcloud dev`.
   ```
   ibmcloud dev create
   ```
   {: pre}

1. Seleziona `Backend Service / Web App` > `Java - MicroProfile / JavaEE` > `Java Web App with Eclipse MicroProfile and Java EE (Web App)` per creare uno starter Java. (Per creare invece uno starter Node.js, utilizza `Backend Service / Web App` > `Node`> `Node.js Web App with Express.js (Web App)` )
1. Immetti un nome (**name**) per la tua applicazione.
1. Seleziona il gruppo di risorse in cui distribuire questa applicazione.
1. Non aggiungere ulteriori servizi.
1. Non aggiungere una toolchain DevOps, seleziona **manual deployment**.

Questo genera un'applicazione starter completa con il codice e tutti i file di configurazione necessari per lo sviluppo locale e la distribuzione al cloud su Cloud Foundry o Kubernetes. 

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### Crea l'applicazione

Puoi creare ed eseguire l'applicazione come faresti normalmente con `mvn` per lo sviluppo locale java o con `npm` per lo sviluppo dei nodi.  Puoi anche creare un'immagine docker ed eseguire l'applicazione in un contenitore per garantire l'esecuzione coerente in locale e nel cloud. Per creare la tua immagine docker, utilizza la seguente procedura.

1. Assicurati che il motore Docker locale sia avviato.
   ```
   docker ps
   ```
   {: pre}
2. Passa alla directory del progetto generato.
   ```
   cd <project name>
   ```
   {: pre}
3. Crea l'applicazione.
   ```
   ibmcloud dev build
   ```
   {: pre}

   L'esecuzione di questo comando potrebbe richiedere alcuni minuti poiché vengono scaricate tutte le dipendenze dell'applicazione e viene creata un'immagine Docker, che contiene la tua applicazione e tutto l'ambiente richiesto.

### Esegui l'applicazione in locale

1. Esegui il contenitore.
   ```
   ibmcloud dev run
   ```
   {: pre}

   Questo utilizza il tuo motore Docker locale per eseguire l'immagine docker che hai creato nel passo precedente.
2. Dopo l'avvio del tuo contenitore, vai a `http://localhost:9080/`. Se hai creato un'applicazione Node.js , vai a `http://localhost:3000/`.
  ![](images/solution2/LibertyLocal.png)

## Distribuisci l'applicazione al cluster utilizzando il grafico helm
{: #deploy}

In questa sezione, esegui prima il push dell'immagine Docker al registro del contenitore privato IBM Cloud e crei quindi una distribuzione Kubernetes che punta a tale immagine.

1. Trova il tuo spazio dei nomi (**namespace**) elencando tutti gli spazi dei nomi nel registro.
   ```sh
   ibmcloud cr namespaces
   ```
   {: pre}
   Se hai uno spazio dei nomi, annotane il nome per utilizzarlo in seguito. Se non ne hai uno, crealo.
   ```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. Imposta le variabili di ambiente MYNAMESPACE e MYPROJECT rispettivamente sul tuo spazio dei nomi e sul tuo nome del progetto.

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. Identifica il tuo registro del contenitore (**Container Registry**) (ad es. us.icr.io) eseguendo `ibmcloud cr info`
4. Imposta la variabile di ambiente MYREGISTRY sul tuo registro.
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. Crea e contrassegna (`-t`) l'immagine docker
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. Esegui il push dell'immagine docker al tuo registro del contenitore su IBM Cloud
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. In un IDE, vai a **values.yaml** in `chart\YOUR PROJECT NAME` e aggiorna il valore **image repository** puntando alla tua immagine nel registro del contenitore IBM Cloud. Salva (**Save**) il file.

   Per i dettagli del repository delle immagini, esegui `echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}`

8. [Helm](https://helm.sh/) ti aiuta a gestire le applicazioni Kubernetes tramite i grafici Helm, cosa che aiuta a definire, installare ed eseguire l'upgrade anche dell'applicazione Kubernetes più complessa. Inizializza Helm andando a `chart\YOUR PROJECT NAME` ed eseguendo questo comando nel tuo cluster

   ```bash
   helm init
   ```
   {: pre}
   Per eseguire l'upgrade di helm, esegui questo comando `helm init --upgrade`
   {:tip}

9. Per installare un grafico Helm, passa alla directory `chart\YOUR PROJECT NAME` ed esegui questo comando
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. Utilizza `kubectl get service ${MYPROJECT}-service` per la tua applicazione Java e `kubectl get service ${MYPROJECT}-application-service`  per la tua applicazione Node.js per identificare la porta pubblica sulla quale è in ascolto il servizio. La porta è un numero di 5 cifre (ad es. 31569) in `PORT(S)`.
11. Per l'IP pubblico del nodo di lavoro, esegui questo comando
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. Accedi all'applicazione all'indirizzo `http://worker-ip-address:portnumber/`.

## Utilizza il dominio fornito da IBM per il tuo cluster
{: #ibm_domain}

Nel passo precedente, l'accesso all'applicazione è stato eseguito con una porta non standard. Il servizio è stato esposto mediante la funzione NodePort di Kubernetes.

I cluster a pagamento sono forniti con un dominio fornito da IBM  Questo ti dà una migliore opzione di esporre le applicazioni con un URL appropriato e su porte HTTP/S standard.

Utilizza Ingress per configurare la connessione in entrata del cluster al servizio.

![Ingress](images/solution2/Ingress.png)

1. Identifica il tuo **dominio Ingress** fornito da IBM
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   per trovare
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. Crea un file Ingress `ingress-ibmdomain.yml` che punta al tuo dominio con il supporto per HTTP e HTTPS. Utilizza il seguente file come un template, sostituendo tutti i valori racchiusi in <> con i valori appropriati dall'output sopra indicato. **service-name** è il nome in `==> v1/Service` nel passo precedente oppure esegui `kubectl get svc` per trovare il nome del servizio di tipo **NodePort**.
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-ibmdomain-http-and-https
   spec:
     tls:
     - hosts:
       -  <nameofproject>.<ingress-sub-domain>
       secretName: <ingress-secret>
     rules:
     - host: <nameofproject>.<ingress-sub-domain>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: codeblock}
3. Distribuisci l'Ingress
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. Accedi alla tua applicazione all'indirizzo `https://<nameofproject>.<ingress-sub-domain>/`

## Utilizza il tuo dominio personalizzato
{: #custom_domain}

Per utilizzare il tuo dominio personalizzato, devi aggiornare i tuoi record DNS con un record CNAME che punta al tuo dominio fornito da IBM oppure con un record A che punta all'indirizzo IP pubblico portatile dell'Ingress fornito da IBM. Dato che un cluster a pagamento viene fornito con indirizzi IP fissi, un record A è una buona opzione.

Per ulteriori informazioni, vedi il documento relativo all'[utilizzo del controller Ingress con un dominio personalizzato](https://{DomainName}/docs/containers?topic=containers-ingress#ingress).

### con HTTP

1. Crea un file Ingress `ingress-customdomain-http.yml` che punta al tuo dominio:
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-customdomain-http
   spec:
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
2. Distribuisci l'Ingress
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. Accedi all'applicazione all'indirizzo `http://<customdomain>/`

### con HTTPS

Se tu dovessi provare ora ad accedere alla tua applicazione con HTTPS `https://<customdomain>/`, riceverai probabilmente un'avvertenza di sicurezza dal tuo browser web che ti informa che la connessione non è privata. Otterrai anche un 404 poiché l'Ingress appena configurato non saprebbe come indirizzare il traffico HTTPS.

1. Ottieni un certificato SSL attendibile per il tuo dominio. Avrai bisogno del certificato e della chiave:
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3
   Puoi utilizzare [Let's Encrypt](https://letsencrypt.org/) per generare il certificato attendibile.
2. Salva il certificato e la chiave in file in formato ascii a base64.
3. Crea un segreto TLS per archiviare il certificato e la chiave:
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<custom-domain.cert> --key=<custom-domain.key>
   ```
   {: pre}
4. Crea un file Ingress `ingress-customdomain-https.yml` che punta al tuo dominio:
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-customdomain-https
   spec:
     tls:
     - hosts:
       - <my-custom-domain.com>
       secretName: <my-custom-domain-secret-name>
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
5. Distribuisci l'Ingress:
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. Accedi all'applicazione all'indirizzo `https://<customdomain>/`.

## Monitora l'integrità dell'applicazione
{: #monitor_application}

1. Per controllare l'integrità della tua applicazione, vai a [clusters](https://{DomainName}/containers-kubernetes/clusters) per visualizzare un elenco dei cluster e fai clic sul cluster che hai creato in precedenza.
2. Fai clic su **Kubernetes Dashboard** per avviare il dashboard in una nuova scheda.
   ![](images/solution2/launch_kubernetes_dashboard.png)
3. Seleziona **Nodes** nel riquadro di sinistra, fai clic sul nome (**Name**) dei nodi e guarda le risorse di assegnazione (**Allocation Resources**) per vedere l'integrità dei tuoi nodi.
   ![](images/solution2/KubernetesDashboard.png)
4. Per esaminare i log dell'applicazione dal contenitore, seleziona **Pods**, **pod-name** e **Logs**.
5. Per eseguire l'**ssh** nel contenitore, identifica il tuo nome pod dal passo precedente ed esegui
   ```sh
   kubectl exec -it <pod-name> -- bash
   ```
   {: pre}

## Ridimensiona i pod Kubernetes
{: #scale_cluster}

Man mano che il carico aumenta sulla tua applicazione, puoi aumentare manualmente il numero di repliche pod nella tua distribuzione. Le repliche sono gestite da un [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/). Per ridimensionare l'applicazione a due repliche, esegui questo comando:

   ```sh
 kubectl scale deployment <nameofproject>-deployment --replicas=2
   ```
   {: pre}

Dopo poco, vedrai due pod per la tua applicazione nel dashboard Kubernetes (o con `kubectl get pods`). Il controller Ingress nel cluster gestirà il bilanciamento del carico tra le due repliche. Anche il ridimensionamento orizzontale può essere reso automatico.

Fai riferimento alla documentazione di Kubernetes per il ridimensionamento manuale e automatico:

   * [Ridimensionamento di una distribuzione](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [Ridimensionamento orizzontale automatico del pod](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## Rimuovi le risorse

* Elimina il cluster oppure elimina solo le risorse Kubernetes create per l'applicazione, se intendi riutilizzare il cluster.

## Contenuto correlato

* [Servizio Kubernetes IBM Cloud](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [Distribuzione continua su Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
