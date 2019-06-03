---
copyright:
  years: 2017, 2018
lastupdated: "2019-05-06"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Esercitazioni della soluzione
{: #tutorials}

Scopri come creare, distribuire e ridimensionare soluzioni reali su IBM Cloud. Queste guide forniscono istruzioni dettagliate su come utilizzare IBM Cloud per implementare pattern comuni basati su prassi ottimali e tecnologie collaudate.
<style>
<!--
    #tutorials { /* hide the page header */
        display: none !important
    }
    p.last-updated { /* hide the last updated */
        display: none !important;
    }
    .doesNotExist, #doc-content, #single-content { /* use full width */
        width: calc(100% - 8%) !important;
        max-width: calc(100% - 8%) !important;
    }
    aside.side-nav, #topic-toc-wrapper { /* no need for side-nav */
        display: none !important;
    }
    .detailContentArea { /* use full width */
        max-width: 100% !important;
    }
    .allCategories {
        display: flex !important;
        flex-direction: row !important;
        flex-wrap: wrap !important;
    }
    .categoryBox {
        flex-grow: 1 !important;
        width: calc(33% - 20px) !important;
        text-decoration: none !important;
        margin: 0 10px 20px 0 !important;
        padding: 20px !important;
        border: 1px #dfe6eb solid !important;
        box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.2) !important;
        text-align: center !important;
        text-overflow: ellipsis !important;
        overflow: hidden !important;
    }
    .solutionBoxContainer {}
    .solutionBoxContainer a {
        text-decoration: none !important;
        border: none !important;
    }
    .solutionBox {
        display: inline-block !important;
        width: 100% !important;
        margin: 0 10px 20px 0 !important;
        padding: 10px !important;
        border: 1px #dfe6eb solid !important;
        box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.2) !important;
    }
    @media screen and (min-width: 960px) {
        .solutionBox {
        width: calc(50% - 3%) !important;
        }
        .solutionBox.solutionBoxFeatured {
        width: calc(50% - 3%) !important;
        }
        .solutionBoxContent {
        height: 270px !important;
        }
    }
    @media screen and (min-width: 1298px) {
        .solutionBox {
        width: calc(33% - 2%) !important;
        }
        .solutionBoxContent {
        min-height: 270px !important;
        }
    }
    .solutionBox:hover {
        border-color: rgb(136, 151, 162) !important;
    }
    .solutionBoxContent {
        display: flex !important;
        flex-direction: column !important;
    }
    .solutionBoxTitle {
        margin: 0rem !important;
        margin-bottom: 5px !important;
        font-size: 14px !important;
        font-weight: 700 !important;
        line-height: 16px !important;
        height: 37px !important;
        text-overflow: ellipsis !important;
        overflow: hidden !important;
        display: -webkit-box !important;
        -webkit-line-clamp: 2 !important;
        -webkit-box-orient: vertical !important;
    }
    .solutionBoxDescription {
        flex-grow: 1 !important;
        display: flex !important;
        flex-direction: column !important;
    }
    .descriptionContainer {
    }
    .descriptionContainer p {
        margin: 0 !important;
        overflow: hidden !important;
        display: -webkit-box !important;
        -webkit-line-clamp: 4 !important;
        -webkit-box-orient: vertical !important;
        font-size: 12px !important;
        font-weight: 400 !important;
        line-height: 1.5 !important;
        letter-spacing: 0 !important;
        max-height: 70px !important;
    }
    .architectureDiagramContainer {
        flex-grow: 1 !important;
        min-width: 250px !important;
        padding: 0 10px !important;
        text-align: center !important;
        display: flex !important;
        flex-direction: column !important;
        justify-content: center !important;
    }
    .architectureDiagram {
        max-height: 175px !important;
        padding: 5px !important;
        margin: 0 auto !important;
    }
-->
</style>

## Esercitazioni presentate
<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applica la sicurezza end-to-end a un'applicazione cloud
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un'applicazione cloud sicura che presenta dati crittografati con le tue chiavi, l'autenticazione utente e il controllo della sicurezza.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applica la sicurezza end-to-end a un'applicazione cloud" />
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazioni Cloud Foundry Enterprise isolate
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Fornisci una piattaforma di innovazione alla tua organizzazione distribuendo una piattaforma Cloud Foundry di livello enterprise isolata utilizzando Cloud Foundry Enterprise Environment.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazioni Cloud Foundry Enterprise isolate" />
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Siti web e applicazioni web
{: #websites }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-scalable-webapp-kubernetes#scalable-webapp-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione web scalabile in Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Esegui lo scaffolding di un'applicazione web Java, la esegui localmente in un contenitore e poi la distribuisci in un cluster Kubernetes. Inoltre, esegui il bind di un dominio personalizzato, monitori l'integrità dell'ambiente ed esegui il ridimensionamento.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution2/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazione web scalabile in Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Sposta un'applicazione basata sulla VM in Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Prendi un'applicazione basata sulla VM, inseriscila in un contenitore e distribuiscila in un cluster Kubernetes. Utilizza questi passi come guide generali per le altre applicazioni.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution30/modern_architecture.png" alt="Diagramma dell'architettura per la soluzione Sposta un'applicazione basata sulla VM in Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-strategies-for-resilient-applications#strategies-for-resilient-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Strategie per le applicazioni resilienti
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Indipendentemente dall'opzione di calcolo: Kubernetes, Cloud Foundry, Cloud Functions o Server virtuali, le aziende cercano di ridurre il tempo di inattività e di creare architetture resilienti che raggiungono la massima disponibilità.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution39/Architecture.png" alt="Diagramma dell'architettura per la soluzione Strategie per le applicazioni resilienti"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Distribuzione continua su Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configura una pipeline di fornitura e integrazione continue per le applicazioni inserite nei contenitori in esecuzione su un cluster Kubernetes. Aggiungi le integrazioni agli altri servizi come scanner della sicurezza, notifiche Slack e analisi.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution21/Architecture.png" alt="Diagramma dell'architettura per la soluzione Distribuzione continua su Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Accelera la distribuzione dei file statici utilizzando l'Object Storage e la CDN
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Ospita e servi le risorse di sito web (immagini, video, documenti) e il contenuto generato dall'utente in un Cloud Object Storage e utilizza una CDN (Content Delivery Network) per la distribuzione veloce e sicura agli utenti in tutto il mondo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution3/Architecture.png" alt="Diagramma dell'architettura per la soluzione Accelera la distribuzione dei file statici utilizzando l'Object Storage e la CDN"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-pub-sub-object-storage#pub-sub-object-storage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Elaborazione dei dati asincrona utilizzando l'archiviazione oggetti e la messaggistica di pubblicazione/sottoscrizione
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilizza Message Hub basato su Apache Kafka per orchestrare i carichi di lavoro tra i microservizi in esecuzione in un cluster Kubernetes e archiviare i dati in Object Storage.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution25/Architecture.png" alt="Diagramma dell'architettura per la soluzione Elaborazione dei dati asincrona utilizzando l'archiviazione oggetti e la messaggistica di pubblicazione/sottoscrizione"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-lamp-stack#lamp-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione web nello stack LAMP
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un server virtuale Ubuntu Linux, con il server web Apache, MySQL e PHP. Quindi installa e configura l'applicazione open source WordPress sullo stack LAMP.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution4/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazione web nello stack LAMP"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Distribuisci uno stack LAMP utilizzando Terraform
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilizza Terraform per eseguire il provisioning di un server virtuale Linux, con il server web Apache, MySQL, PHP e il servizio IBM Cloud Object Storage. Aggiorna la configurazione per ridimensionare le risorse e ottimizzare l'ambiente.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution10/architecture-2.png" alt="Diagramma dell'architettura per la soluzione Distribuisci uno stack LAMP utilizzando Terraform"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Pianifica, crea e aggiorna ambienti di distribuzione
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Automatizza la creazione e la manutenzione di più ambienti di distribuzione con la CLI IBM Cloud e Terraform.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution26-plan-create-update-deployments/architecture.png" alt="Diagramma dell'architettura per la soluzione Pianifica, crea e aggiorna ambienti di distribuzione"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Utilizza i server virtuali per creare un'applicazione web altamente disponibile e scalabile
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un programma di bilanciamento del carico, due server delle applicazioni in esecuzione su Ubuntu con NGINX e PHP installati, un server database MySQL e un'archiviazione file durevole per archiviare i file di applicazione e i backup.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution14/Architecture.png" alt="Diagramma dell'architettura per la soluzione Utilizza i server virtuali per creare un'applicazione web altamente disponibile e scalabile"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-mean-stack#mean-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione web moderna utilizzando lo stack MEAN
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un'applicazione web utilizzando il diffuso stack MEAN - Mongo DB, Express, Angular, Node.js. Esegui l'applicazione localmente, crea e utilizza un DBaaS (database-as-a-service), distribuisci l'applicazione e monitora l'applicazione.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution7/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazione web moderna utilizzando lo stack MEAN"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-sql-database#sql-database">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Database SQL per dati Cloud
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Esegui il provisioning di un servizio di database SQL relazionale, crea una tabella e carica una serie di dati di grandi dimensioni nel database. Distribuisci un'applicazione per utilizzare tali dati e per mostrare in che modo accedere al database cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution5/Architecture.png" alt="Diagramma dell'architettura per la soluzione Database SQL per dati Cloud"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione web senza server e API
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un'applicazione web senza server ospitando il contenuto del sito web statico nelle pagine GitHub e utilizzando Cloud Functions per implementare il backend dell'applicazione.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution8/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazione web senza server e API"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Distribuzione di applicazioni senza server in più regioni
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilizza Cloud Functions e Internet Services per creare applicazioni senza server sicure e globalmente disponibili.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution44-multi-region-serverless/Architecture.png" alt="Diagramma dell'architettura per la soluzione Distribuzione di applicazioni senza server in più regioni"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Analizza i log e monitora l'integrità dell'applicazione con LogDNA e Sysdig
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilizza IBM Log Analysis con LogDNA per comprendere e diagnosticare le attività dell'applicazione. Monitora le applicazioni con IBM Cloud Monitoring con Sysdig.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution12/Architecture.png" alt="Diagramma dell'architettura per la soluzione Analizza i log e monitora l'integrità dell'applicazione con LogDNA e Sysdig"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazioni Cloud Foundry Enterprise isolate
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Fornisci una piattaforma di innovazione alla tua organizzazione distribuendo una piattaforma Cloud Foundry di livello enterprise isolata utilizzando Cloud Foundry Enterprise Environment.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazioni Cloud Foundry Enterprise isolate"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Chatbot
{: #chatbots }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-slack-chatbot-database-watson#slack-chatbot-database-watson">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Crea uno Slackbot controllato dal database
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea uno Slackbot controllato dal database con IBM Watson Assistant, Cloudant e IBM Cloud Functions.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution19/SlackbotArchitecture.png" alt="Diagramma dell'architettura per la soluzione Crea uno Slackbot controllato dal database"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-watson-chatbot#android-watson-chatbot">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Crea un chatbot Android abilitato alla funzione vocale
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Definisci gli intenti, le entità e crea un flusso di dialogo per il tuo chatbot per rispondere alle domande del cliente. Abilita i servizi Speech to text e Text to speech per una facile interazione con l'applicazione Android.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution28-watson-chatbot-android/architecture.png" alt="Diagramma dell'architettura per la soluzione Crea un chatbot Android abilitato alla funzione vocale"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Sicurezza
{: #security }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione web protetta in più regioni
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea, proteggi, distribuisci e bilancia il carico di un'applicazione web tra più regioni utilizzando una pipeline di fornitura continua.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution1/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazione web protetta in più regioni"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Cluster Kubernetes multi-regione resilienti e protetti
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Integra Cloud Internet Services con i cluster Kubernetes per distribuire una soluzione resiliente e sicura in più regioni.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution32-multi-region-k8s-cis/Architecture.png" alt="Diagramma dell'architettura per la soluzione Cluster Kubernetes multi-regione resilienti e protetti"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Crea, proteggi e gestisci le API REST
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea una nuova API REST utilizzando il framework API Node.js LoopBack. Aggiungi la gestione, la visibilità, la sicurezza e la limitazione della frequenza all'API utilizzando il servizio API Connect in IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution13/Architecture.png" alt="Diagramma dell'architettura per la soluzione Crea, proteggi e gestisci le API REST"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applica la sicurezza end-to-end a un'applicazione cloud
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un'applicazione cloud sicura che presenta dati crittografati con le tue chiavi, l'autenticazione utente e il controllo della sicurezza.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applica la sicurezza end-to-end a un'applicazione cloud"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Mobile
{: #mobile }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione mobile iOS con Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un'applicazione iOS Swift con Push Notifications su IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution6/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazione mobile iOS con Push Notifications"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione mobile nativa Android con Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Scrivi un'applicazione nativa Android con Push Notifications su IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution9/Architecture.png" alt="Diagramma dell'architettura per la soluzione Scrivi un'applicazione nativa Android con Push Notifications su IBM Cloud"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-hybrid-mobile-push-analytics#hybrid-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione mobile ibrida con Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Sviluppa un'applicazione Cordova ibrida con Push Notifications su IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution15/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazione mobile ibrida con Push Notifications"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-mobile-backend#serverless-mobile-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Applicazione mobile con un backend senza server
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilizza Cloud Functions con i servizi Cognitive e Data per creare un backend senza server per un'applicazione mobile.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution11/Architecture.png" alt="Diagramma dell'architettura per la soluzione Applicazione mobile con un backend senza server"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Machine Learning e analisi
{: #ml }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Log di big data con Streaming Analytics e SQL
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Raccogli, archivia e analizza i record di log per supportare i requisiti normativi e per facilitare il rilevamento delle informazioni. Utilizzando la messaggistica di pubblicazione-sottoscrizione, ridimensiona la soluzione a milioni di record e poi esegui l'analisi sui log indicati come persistenti con l'SQL noto.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution31/Architecture.png" alt="Diagramma dell'architettura per la soluzione Log di big data con Streaming Analytics e SQL"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-smart-data-lake#smart-data-lake">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Crea un data lake utilizzando Object Storage
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Fornisci gli strumenti ai data scientist per eseguire la query dei dati utilizzando SQL Query e per condurre l'analisi in Watson Studio. Condividi i dati e le informazioni approfondite tramite grafici interattivi.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution29/architecture.png" alt="Diagramma dell'architettura per la soluzione Crea un data lake utilizzando Object Storage"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-github-traffic-analytics#serverless-github-traffic-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Combinazione dell'azione senza server e di Cloud Foundry per il richiamo e l'analisi dei dati
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Raccogli automaticamente le statistiche sul traffico GitHub per i repository, archiviale in un database SQL e inizia a lavorare con l'analisi del traffico.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution24-github-traffic-analytics/Architecture.png" alt="Diagramma dell'architettura per la soluzione Combinazione dell'azione senza server e di Cloud Foundry per il richiamo e l'analisi dei dati"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#create-deploy-retrain-machine-learning-model">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Crea, distribuisci, verifica e riaddestra un modello di machine learning predittivo
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un modello di machine learning predittivo, distribuiscilo come un'API, verifica e riaddestra il modello con i dati di feedback.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution22-build-machine-learning-model/architecture_diagram.png" alt="Diagramma dell'architettura per la soluzione Crea, distribuisci, verifica e riaddestra un modello di machine learning predittivo"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-analytics-spark#big-data-analytics-spark">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Analizza e visualizza i dati aperti con Apache Spark
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Analizza e visualizza le serie di dati aperti utilizzando un notebook Jupyter. Utilizza il servizio Apache Spark con IBM Watson Studio e Pixiedust per generare i grafici.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution23/Architecture.png" alt="Diagramma dell'architettura per la soluzione Analizza e visualizza i dati aperti con Apache Spark"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Internet of Things
{: #iot }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#gather-visualize-analyze-iot-data">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Raccogli, visualizza e analizza i dati IoT
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configura un dispositivo IoT, raccogli grandi quantità di dati in Watson IoT Platform, analizza i dati con il machine learning e crea le visualizzazioni.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution16/Architecture.png" alt="Diagramma dell'architettura per la soluzione Raccogli, visualizza e analizza i dati IoT"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Identity and Access Management
{: #iam }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Procedure ottimali per organizzare gli utenti, i team e le applicazioni
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Una panoramica dei concetti disponibili in IBM Cloud per gestire IAM (Identity and Access Management) e il modo in cui possono essere implementati per supportare le diverse fasi di sviluppo di un'applicazione.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution20-users-teams-applications/architecture.png" alt="Diagramma dell'architettura per la soluzione Procedure ottimali per organizzare gli utenti, i team e le applicazioni"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-usage#cloud-usage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Revisione dei servizi, delle risorse e dell'utilizzo di IBM Cloud
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Un'introduzione ai diversi approcci utilizzati per rispondere alle domande più comuni relative all'utilizzo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution38/Architecture.png" alt="Diagramma dell'architettura per la soluzione Revisione dei servizi, delle risorse e dell'utilizzo di IBM Cloud"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Rete
{: #Network }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Sottoreti private e pubbliche in un VPC (Virtual Private Cloud)
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un VPC (virtual private cloud) con sottoreti e istanze. Proteggi le tue risorse collegando i gruppi di sicurezza e consenti solo l'accesso minimo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution40-vpc-public-app-private-backend/Architecture.png" alt="Diagramma dell'architettura per la soluzione Sottoreti private e pubbliche in un VPC (Virtual Private Cloud)"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-multi-region#vpc-multi-region">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Distribuisci i carichi di lavoro isolati tra più ubicazioni e zone
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Distribuisci un carico di lavoro in VPC (virtual private cloud) tra più ubicazioni e zone. Distribuisci il traffico nelle zone con programmi di bilanciamento del carico locali e globali.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution41-vpc-multi-region/Architecture.png" alt="Diagramma dell'architettura per la soluzione Distribuisci i carichi di lavoro isolati tra più ubicazioni e zone"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-site2site-vpn#vpc-site2site-vpn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Utilizza un gateway VPC/VPN per l'accesso in loco privato e sicuro alle risorse cloud
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Connetti un VPC (Virtual Private Cloud) a un altro ambiente di calcolo su una VPN (Virtual Private Network) sicura e utilizza i servizi IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution46-vpc-vpn/ArchitectureDiagram.png" alt="Diagramma dell'architettura per la soluzione Utilizza un gateway VPC/VPN per l'accesso in loco privato e sicuro alle risorse cloud"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server#vpc-secure-management-bastion-server">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Accedi in modo sicuro alle istanze remote con un host bastion
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Distribuisci un host bastion per accedere in modo sicuro alle istanze remote all'interno di un VPC (virtual private cloud).</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png" alt="Diagramma dell'architettura per la soluzione Accedi in modo sicuro alle istanze remote con un host bastion"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Isola i carichi di lavoro con una rete privata sicura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configura un VRA (Virtual Router Appliance) per creare un'enclosure sicura. Associa le VLAN, esegui il provisioning dei server, configura l'instradamento IP e i firewall.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution33-secure-network-enclosure/Secure-priv-enc.png" alt="Diagramma dell'architettura per la soluzione Isola i carichi di lavoro con una rete privata sicura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-nat-config-private#nat-config-private">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Configura NAT per l'accesso a Internet da una rete privata
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilizza il mascheramento NAT per convertire gli indirizzi IP privati in un'interfaccia pubblica in uscita.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution35-nat-config-private/vra-nat.png" alt="Diagramma dell'architettura per la soluzione Configura NAT per l'accesso a Internet da una rete privata"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-byoip#byoip">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                BYOIP (Bring Your Own IP)
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Una panoramica dei pattern di implementazione BYOIP e una guida per identificare il pattern appropriato.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution37-byoip/byoipdecision.png" alt="Diagramma dell'architettura per la soluzione BYOIP (Bring Your Own IP)"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                VPN in una rete privata sicura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea una connessione privata tra un ambiente di rete remoto e i server nella rete privata di IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png" alt="Diagramma dell'architettura per la soluzione VPN in una rete privata sicura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vlan-spanning#vlan-spanning">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Collegamento delle reti private sicure sulla rete IBM
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Distribuisci due reti private che sono collegate in modo sicuro sulla rete privata IBM Cloud utilizzando il servizio Spanning delle VLAN.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution43-vlan-spanning/vlan-spanning.png" alt="Diagramma dell'architettura per la soluzione Collegamento delle reti private sicure sulla rete IBM"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-web-app-private-network#web-app-private-network">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Hosting di applicazioni web da una rete privata sicura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crea un'applicazione web Internet scalabile e sicura ospitata in una rete privata utilizzando un VRA (virtual router appliance), VLAN, NAT e firewall.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution42-web-app-private-network/web-app-private.png" alt="Diagramma dell'architettura per la soluzione Hosting di applicazioni web da una rete privata sicura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

