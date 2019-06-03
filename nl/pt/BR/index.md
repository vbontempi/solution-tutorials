---
copyright:
  years: 2017, 2018
lastupdated: "2019-05-06"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Tutoriais de solução
{: #tutorials}

Aprenda como construir, implementar e escalar soluções reais no IBM Cloud. Esses guias fornecem instruções passo a passo sobre como usar o IBM Cloud para implementar padrões comuns com base nas melhores práticas e tecnologias comprovadas.
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

## Tutoriais em destaque
<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicar segurança de ponta a ponta a um aplicativo em nuvem
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie um aplicativo em nuvem seguro que apresente dados criptografados com suas próprias chaves, autenticação do usuário e auditoria de segurança.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="Diagrama de arquitetura para a solução Aplicar segurança de ponta a ponta a um aplicativo em nuvem" />
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Apps Cloud Foundry Enterprise isolados
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Forneça uma plataforma de inovação para sua organização, implementando uma plataforma Cloud Foundry isolada, de nível empresarial usando o Cloud Foundry Enterprise Environment.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="Diagrama de arquitetura para a solução Apps corporativos do Cloud Foundry isolados" />
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Websites e apps da web
{: #websites }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-scalable-webapp-kubernetes#scalable-webapp-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                App da web escalável no Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie o esqueleto de um aplicativo da web Java, execute-o localmente em um contêiner e, em seguida, implemente-o em um cluster Kubernetes. Além disso, ligue um domínio customizado, monitore o funcionamento do ambiente e escale.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution2/Architecture.png" alt="Diagrama de arquitetura para a solução App da web escalável no Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Mover um aplicativo baseado em VM para o Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Pegue um aplicativo baseado em VM, conteinerize-o, implemente-o em um cluster Kubernetes. Use as etapas como guias gerais para outros aplicativos.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution30/modern_architecture.png" alt="Diagrama de arquitetura para a solução Mover um aplicativo baseado em VM para o Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-strategies-for-resilient-applications#strategies-for-resilient-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Estratégias para aplicativos resilientes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Independentemente da opção de Cálculo: Kubernetes, Cloud Foundry, Cloud Functions ou Virtual Servers, as empresas buscam minimizar o tempo de inatividade e criar arquiteturas resilientes que alcançam a máxima disponibilidade.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution39/Architecture.png" alt="Diagrama de arquitetura para a solução Estratégias para aplicativos resilientes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Implementação contínua no Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configure uma integração contínua e um pipeline de integração para aplicativos conteinerizados em execução em um cluster Kubernetes. Inclua integrações em outros serviços, como scanners de segurança, notificações do Slack e análise de dados.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution21/Architecture.png" alt="Diagrama de arquitetura para a solução Implementação contínua no Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Acelere a entrega de arquivos estáticos usando o Object Storage e o CDN
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Hospede e entregue ativos de website (imagens, vídeos, documentos) e conteúdo gerado pelo usuário em um Cloud Object Storage e use uma Rede de Entrega de Conteúdo (CDN) para entrega rápida e segura para usuários ao redor do mundo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution3/Architecture.png" alt="Diagrama de arquitetura para a solução Acelerar a entrega de arquivos estáticos usando o Object Storage e o CDN"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-pub-sub-object-storage#pub-sub-object-storage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Processamento de dados assíncronos usando o armazenamento de objeto e o sistema de mensagens de publicação/assinatura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Use o Hub de mensagens baseado no Apache Kafka para orquestrar cargas de trabalho entre microsserviços em execução em um cluster Kubernetes e armazenar dados no Object Storage.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution25/Architecture.png" alt="Diagrama de arquitetura para a solução Processamento de dados assíncronos usando o armazenamento de objeto e o sistema de mensagens de publicação/assinatura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-lamp-stack#lamp-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicativo da web na pilha LAMP
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie um servidor virtual Ubuntu Linux com servidor da web Apache, MySQL e PHP. Em seguida, instale e configure o aplicativo de software livre WordPress na pilha LAMP.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution4/Architecture.png" alt="Diagrama de arquitetura para a solução Aplicativo da web na pilha LAMP"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Implementar uma pilha LAMP usando o Terraform
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Use o Terraform para provisionar um servidor virtual Linux com servidor da web Apache, MySQL, PHP e o serviço IBM Cloud Object Storage. Atualize a configuração para escalar os recursos e ajustar o ambiente.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution10/architecture-2.png" alt="Diagrama de arquitetura para a solução Implementar uma pilha LAMP usando o Terraform"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Planejar, criar e atualizar ambientes de implementação
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Automatize a criação e a manutenção de múltiplos ambientes de implementação com a CLI do IBM Cloud e o Terraform.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution26-plan-create-update-deployments/architecture.png" alt="Diagrama de arquitetura para a solução Planejar, criar e atualizar ambientes de implementação"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Usar Virtual Servers para construir um app da web altamente disponível e escalável
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie um balanceador de carga, dois servidores de aplicativos em execução no Ubuntu com NGINX e PHP instalados, um servidor de banco de dados MySQL e armazenamento de arquivo durável para armazenar arquivos de aplicativo e backups.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution14/Architecture.png" alt="Diagrama de arquitetura para a solução Usar Virtual Servers para construir um app da web altamente disponível e escalável"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-mean-stack#mean-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicativo da web moderno usando a pilha MEAN
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Construa um aplicativo da web usando a pilha MEAN popular - DB Mongo, Express, Angular, Node.js. Execute o app localmente, crie e use um banco de dados como um serviço, implemente o app e monitore o aplicativo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution7/Architecture.png" alt="Diagrama de arquitetura para a solução Aplicativo da web moderno usando a pilha MEAN"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-sql-database#sql-database">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Banco de dados SQL para dados de nuvem
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Provisione um serviço de banco de dados relacional SQL, crie uma tabela e carregue um conjunto de dados grande no banco de dados. Implemente um app da web para fazer uso desses dados e mostre como acessar o banco de dados de nuvem.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution5/Architecture.png" alt="Diagrama de arquitetura para a solução Banco de dados SQL para dados de nuvem"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicativo da web serverless e API
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie um aplicativo da web serverless hospedando o conteúdo do website estático em Páginas do GitHub e usando o Cloud Functions para implementar o back-end do aplicativo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution8/Architecture.png" alt="Diagrama de arquitetura para a solução Aplicativo da web serverless e API"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Implementar apps serverless em múltiplas regiões
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Use o Cloud Functions e o Internet Services para construir aplicativos serverless globalmente disponíveis e seguros.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution44-multi-region-serverless/Architecture.png" alt="Diagrama de arquitetura para a solução Implementar apps serverless em múltiplas regiões"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Analisar logs e monitorar o funcionamento do aplicativo com LogDNA e Sysdig
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Use o IBM Log Analysis com LogDNA para entender e diagnosticar atividades do aplicativo. Monitore aplicativos com o IBM Cloud Monitoring com Sysdig.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution12/Architecture.png" alt="Diagrama de arquitetura para a solução Analisar logs e monitorar o funcionamento do aplicativo com LogDNA e Sysdig"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Apps Cloud Foundry Enterprise isolados
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Forneça uma plataforma de inovação para sua organização, implementando uma plataforma Cloud Foundry isolada, de nível empresarial usando o Cloud Foundry Enterprise Environment.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="Diagrama de arquitetura para a solução Apps corporativos Cloud Foundry"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Robôs de bate-papo
{: #chatbots }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-slack-chatbot-database-watson#slack-chatbot-database-watson">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Construir um Slackbot acionado por banco de dados
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Construa um Slackbot acionado por banco de dados com o IBM Watson Assistant, o Cloudant e o IBM Cloud Functions.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution19/SlackbotArchitecture.png" alt="Diagrama de arquitetura para a solução Construir um Slackbot acionado por banco de dados"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-watson-chatbot#android-watson-chatbot">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Construir um robô de bate-papo Android ativado por voz
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Defina intentos, entidades e construa um fluxo de diálogo para que o robô de bate-papo responda às consultas do cliente. Ative os serviços de fala para texto e de texto para fala para interação fácil com o app Android.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution28-watson-chatbot-android/architecture.png" alt="Diagrama de arquitetura para a solução Construir um robô de bate-papo Android ativado por voz"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Segurança
{: #security }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Proteger o aplicativo da web em múltiplas regiões
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie, proteja, implemente e balanceie a carga de um aplicativo da web em múltiplas regiões usando um pipeline de entrega contínua.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution1/Architecture.png" alt="Diagrama de arquitetura para a solução Proteger o aplicativo da web em múltiplas regiões"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Clusters Kubernetes multiregion resilientes e seguros
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Integre o Cloud Internet Services a clusters Kubernetes para entregar uma solução resiliente e segura em múltiplas regiões.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution32-multi-region-k8s-cis/Architecture.png" alt="Diagrama de arquitetura para a solução Clusters Kubernetes multiregion resilientes e seguros"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Criar, proteger e gerenciar APIs de REST
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie uma nova API de REST usando a estrutura da API do Node.js LoopBack. Inclua gerenciamento, visibilidade, segurança e limitação de taxa para a API usando o serviço API Connect no IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution13/Architecture.png" alt="Diagrama de arquitetura para a solução Criar, proteger e gerenciar APIs de REST"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicar segurança de ponta a ponta a um aplicativo em nuvem
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie um aplicativo em nuvem seguro que apresente dados criptografados com suas próprias chaves, autenticação do usuário e auditoria de segurança.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="Diagrama de arquitetura para a solução Aplicar segurança de ponta a ponta a um aplicativo em nuvem"/>
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
                App móvel iOS com Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie um aplicativo iOS Swift com Push Notifications no IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution6/Architecture.png" alt="Diagrama de arquitetura para a solução App móvel iOS com Push Notifications"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                App móvel nativo Android com Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Grave um aplicativo nativo Android com Push Notifications no IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution9/Architecture.png" alt="Diagrama de arquitetura para a solução App móvel nativo Android com Push Notifications"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-hybrid-mobile-push-analytics#hybrid-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicativo móvel híbrido com Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Desenvolva um aplicativo Cordova híbrido com Push Notifications no IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution15/Architecture.png" alt="Diagrama de arquitetura para a solução Aplicativo móvel híbrido com Push Notifications"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-mobile-backend#serverless-mobile-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicativo móvel com um back-end serverless
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Use o Cloud Functions com serviços cognitivos e de dados para construir um back-end serverless para um aplicativo móvel.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution11/Architecture.png" alt="Diagrama de arquitetura para a solução Aplicativo móvel com um back-end serverless"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Aprendizado de máquina e análise de dados
{: #ml }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Logs de big data com análise de dados de fluxo e SQL
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Colete, armazene e analise registros de log para suportar requisitos regulamentares e auxiliar a descoberta de informações. Usando o sistema de mensagens de publicação/assinatura, escale a solução para milhões de registros e, em seguida, execute a análise nos logs persistidos com SQL familiar.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution31/Architecture.png" alt="Diagrama de arquitetura para a solução Logs de big data com análise de dados de fluxo e SQL"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-smart-data-lake#smart-data-lake">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Construir um data lake com o Object Storage
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Forneça ferramentas aos cientistas de dados para consultar dados usando o SQL Query e conduzir análise no Watson Studio. Compartilhe dados e insights por meio de gráficos interativos.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution29/architecture.png" alt="Diagrama de arquitetura para a solução Construir um data lake com o Object Storage"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-github-traffic-analytics#serverless-github-traffic-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Combinando o serverless e o Cloud Foundry para recuperação de dados e análise de dados
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Colete automaticamente as estatísticas de tráfego do GitHub para repositórios, armazene-as em um banco de dados SQL e inicie a análise de dados de tráfego.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution24-github-traffic-analytics/Architecture.png" alt="Diagrama de arquitetura para a solução Combinando serverless e Cloud Foundry para recuperação de dados e análise de dados"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#create-deploy-retrain-machine-learning-model">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Construir, implementar, testar e treinar novamente um modelo de aprendizado de máquina preditivo
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Construa um modelo de aprendizado de máquina preditivo, implemente-o como uma API, teste e treine novamente o modelo com dados de feedback.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution22-build-machine-learning-model/architecture_diagram.png" alt="Diagrama de arquitetura para a solução Construir, implementar, testar e treinar novamente um modelo de aprendizado de máquina preditivo"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-analytics-spark#big-data-analytics-spark">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Analisar e visualizar dados abertos com o Apache Spark
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Analise e visualize conjuntos de dados abertos usando um Jupyter Notebook. Usa o serviço Apache Spark com o IBM Watson Studio e Pixiedust para gerar gráficos.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution23/Architecture.png" alt="Diagrama de arquitetura para a solução Analisar e visualizar dados abertos com o Apache Spark"/>
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
                Reunir, visualizar e analisar dados do IoT
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configure um dispositivo IoT, reúna grandes quantias de dados no Watson IoT Platform, analise dados com aprendizado de máquina e crie visualizações.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution16/Architecture.png" alt="Diagrama de arquitetura para a solução Reunir, visualizar e analisar dados do IoT"/>
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
                Melhores práticas para organizar usuários, equipes, aplicativos
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Uma visão geral dos conceitos disponíveis no IBM Cloud para fazer o gerenciamento de identidade e de acesso e como eles podem ser implementados para suportar os múltiplos estágios de desenvolvimento de um aplicativo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution20-users-teams-applications/architecture.png" alt="Diagrama de arquitetura para a solução Melhores práticas para organizar usuários, equipes, aplicativos"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-usage#cloud-usage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Revisando serviços, recursos e uso do IBM Cloud
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Uma introdução a várias abordagens usadas para responder perguntas comuns relacionadas ao uso.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution38/Architecture.png" alt="Diagrama de arquitetura para a solução Revisando serviços, recursos e uso do IBM Cloud"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Rede
{: #Network }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Sub-redes privadas e públicas em uma Nuvem Particular Virtual
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie uma nuvem particular virtual com sub-redes e instâncias. Proteja seus recursos anexando grupos de segurança e permita somente acesso mínimo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution40-vpc-public-app-private-backend/Architecture.png" alt="Diagrama de arquitetura para a solução Sub-redes privadas e públicas em uma Nuvem Particular Virtual"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-multi-region#vpc-multi-region">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Implementar cargas de trabalho isoladas em múltiplos locais e zonas
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Implemente uma carga de trabalho em nuvens particulares virtuais entre múltiplas zonas e regiões. Distribua o tráfego entre zonas com balanceadores de carga local e global.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution41-vpc-multi-region/Architecture.png" alt="Diagrama de arquitetura para a solução Implementar cargas de trabalho isoladas em múltiplos locais e zonas"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-site2site-vpn#vpc-site2site-vpn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Usar um gateway VPC/VPN para acesso no local seguro e privado para recursos em nuvem
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Conecte uma Nuvem Particular Virtual a outro ambiente de computação por meio de uma Rede Privada Virtual segura e consuma serviços do IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution46-vpc-vpn/ArchitectureDiagram.png" alt="Diagrama de arquitetura para a solução Usar um gateway VPC/VPN para acesso no local seguro e privado para recursos em nuvem"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server#vpc-secure-management-bastion-server">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Acessar instâncias remotas de forma segura com um host bastion
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Implemente um host bastion para acessar de forma segura as instâncias remotas dentro de uma nuvem particular virtual.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png" alt="Diagrama de arquitetura para a solução Acessar instâncias remotas de forma segura com um host bastion"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Isolar cargas de trabalho com uma rede privada segura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configure um Virtual Router Appliance para criar um gabinete seguro. Associe VLANs, provisione servidores, configure o roteamento de IP e firewalls.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution33-secure-network-enclosure/Secure-priv-enc.png" alt="Diagrama de arquitetura para a solução Isolar cargas de trabalho com uma rede privada segura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-nat-config-private#nat-config-private">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Configurar o NAT para acesso à Internet por meio de uma rede privada
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Use o mascaramento de NAT para converter endereços IP privados na interface pública de saída.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution35-nat-config-private/vra-nat.png" alt="Diagrama de arquitetura para a solução Configurar o NAT para acesso à Internet por meio de uma rede privada"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-byoip#byoip">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Bring Your Own IP Address
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Uma visão geral de padrões de implementação BYOIP e um guia para identificar o padrão apropriado.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution37-byoip/byoipdecision.png" alt="Diagrama de arquitetura para a solução Bring Your Own IP Address"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                VPN em uma rede privada segura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie uma conexão privada entre um ambiente de rede remota e servidores na rede privada do IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png" alt="Diagrama de arquitetura para a solução VPN em uma rede privada segura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vlan-spanning#vlan-spanning">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Vinculando redes privadas seguras na rede IBM
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Implemente duas redes privadas que estão vinculadas de forma segura na rede privada do IBM Cloud usando o serviço VLAN Spanning.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution43-vlan-spanning/vlan-spanning.png" alt="Diagrama de arquitetura para a solução Vinculando redes privadas seguras na rede IBM"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-web-app-private-network#web-app-private-network">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Entrega de aplicativo da web por meio de uma rede privada segura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Crie um aplicativo da web voltado para a Internet escalável e seguro hospedado na rede privada protegida usando um virtual router appliance (VRA), VLANs, NAT e firewalls.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution42-web-app-private-network/web-app-private.png" alt="Diagrama de arquitetura para a solução Entrega de aplicativo da web por meio de uma rede privada segura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

