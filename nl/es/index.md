---
copyright:
  years: 2017, 2018
lastupdated: "2019-05-06"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# Guías de aprendizaje de soluciones
{: #tutorials}

Descubra cómo crear, desplegar y escalar soluciones del mundo real en IBM Cloud. En estas guías se ofrecen instrucciones paso a paso sobre cómo utilizar IBM Cloud para implementar patrones comunes basados en prácticas recomendadas y tecnologías probadas.
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

## Guías de aprendizaje destacadas
<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicación de seguridad de extremo a extremo a una aplicación de nube
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una aplicación de nube segura que incluya datos cifrados con sus propias claves, autenticación de usuario y auditoría de seguridad.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="Diagrama de la arquitectura de la solución Aplicación de seguridad de extremo a extremo a una aplicación de nube" />
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox solutionBoxFeatured">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Apps de empresa aisladas de Cloud Foundry
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Proporcione una plataforma de innovación a su organización mediante el despliegue de una plataforma de Cloud Foundry aislada a nivel de empresa mediante Cloud Foundry Enterprise Environment.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="Diagrama de arquitectura para la solución Apps de empresa aisladas de Cloud Foundry" />
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Sitios web y apps web
{: #websites }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-scalable-webapp-kubernetes#scalable-webapp-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                App web escalable en Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Diseñe una aplicación web Java, ejecútela localmente en un contenedor y luego despliéguela en un clúster de Kubernetes. Enlace también un dominio personalizado, supervise el estado del entorno y escale.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution2/Architecture.png" alt="Diagrama de la arquitectura de la solución App web escalable en Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Traslado de una aplicación basada en VM a Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Tome una aplicación basada en VM, colóquela en un contenedor y despliéguela en un clúster Kubernetes. Siga estos pasos como guía general para otras aplicaciones.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution30/modern_architecture.png" alt="Diagrama de la arquitectura de la solución Traslado de una aplicación basada en VM a Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-strategies-for-resilient-applications#strategies-for-resilient-applications">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Estrategias para aplicaciones resistentes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Independientemente de la opción de cálculo (Kubernetes, Cloud Foundry, Cloud Functions o servidores virtuales), las empresas tratan de minimizar el tiempo de inactividad y de crear arquitecturas resistentes que alcancen la máxima disponibilidad.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution39/Architecture.png" alt="Diagrama de la arquitectura de la solución Estrategias para aplicaciones resistentes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Despliegue continuo en Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configure un conducto de integración y entrega continua para las aplicaciones contenerizadas que se ejecutan en un clúster Kubernetes. Añada integraciones a otros servicios como escáneres de seguridad, notificaciones de Slack y analítica.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution21/Architecture.png" alt="Diagrama de la arquitectura de la solución Despliegue continuo en Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aceleración de la entrega de archivos estáticos mediante Object Storage y CDN
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Aloje y suministre activos de sitios web (imágenes, vídeos, documentos) y contenido generado por el usuario en Cloud Object Storage y utilice Content Delivery Network (CDN) para una entrega rápida y segura a los usuarios de todo el mundo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution3/Architecture.png" alt="Diagrama de la arquitectura de la solución Aceleración de la entrega de archivos estáticos mediante Object Storage y CDN"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-pub-sub-object-storage#pub-sub-object-storage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Proceso asíncrono de datos mediante Object Storage y mensajería pub/sub
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilice el concentrador de mensajes basado en Apache Kafka para organizar las cargas de trabajo entre los microservicios que se ejecutan en un clúster de Kubernetes y almacene los datos en Object Storage.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution25/Architecture.png" alt="Diagrama de la arquitectura de la solución Proceso asíncrono de datos mediante Object Storage y mensajería pub/sub"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-lamp-stack#lamp-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicación web en la pila de LAMP
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree un servidor virtual Ubuntu Linux, con el servidor web Apache, MySQL y PHP. Luego instale y configure la aplicación de código abierto WordPress en la pila LAMP.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution4/Architecture.png" alt="Diagrama de la arquitectura de la solución Aplicación web en la pila de LAMP"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Despliegue de una pila LAMP mediante Terraform
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilice Terraform para suministrar un servidor virtual Linux, con el servidor web Apache, MySQL, PHP y el servicio IBM Cloud Object Storage. Actualice la configuración para escalar los recursos y ajustar el entorno.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution10/architecture-2.png" alt="Diagrama de la arquitectura de la solución Despliegue de una pila LAMP mediante Terraform"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Planificación, creación y actualización de entornos de despliegue
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Automatice la creación y el mantenimiento de varios entornos de despliegue con la CLI de IBM Cloud y Terraform.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution26-plan-create-update-deployments/architecture.png" alt="Diagrama de la arquitectura de la solución Planificación, creación y actualización de entornos de despliegue"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Uso de servidores virtuales para crear una app web altamente disponible y escalable
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree un equilibrador de carga, dos servidores de aplicaciones que se ejecuten en Ubuntu con NGINX y PHP instalados, un servidor de bases de datos MySQL y un almacenamiento de archivos duradero para almacenar archivos de aplicación y copias de seguridad.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution14/Architecture.png" alt="Diagrama de la arquitectura de la solución Uso de servidores virtuales para crear una app web altamente disponible y escalable"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-mean-stack#mean-stack">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicación web moderna utilizando la pila MEAN
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una aplicación web utilizando la pila MEAN ampliamente utilizada: Mongo DB, Express, Angular, Node.js. Ejecute la app localmente, cree y utilice un servicio de base de datos como servicio, despliegue la app y supervise la aplicación.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution7/Architecture.png" alt="Diagrama de la arquitectura de la solución Aplicación web moderna utilizando la pila MEAN"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-sql-database#sql-database">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                SQL Database for Cloud Data
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Suministre un servicio de base de datos relacional SQL, cree una tabla y cargue un conjunto de datos grande en la base de datos. Despliegue una app web para utilizar esos datos y mostrar cómo acceder a la base de datos de la nube.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution5/Architecture.png" alt="Diagrama de la arquitectura de la solución SQL Database for Cloud Data"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicación web y API sin servidor
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una aplicación web sin servidor alojando contenido del sitio web estático en páginas GitHub y utilizando las funciones de la nube para implementar el programa de fondo de la aplicación.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution8/Architecture.png" alt="Diagrama de la arquitectura de la solución Aplicación web y API sin servidor"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Despliegue de apps sin servidor entre varias regiones
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilice las funciones de la nube y los servicios de Internet para crear aplicaciones sin servidor seguras y disponibles a nivel global.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution44-multi-region-serverless/Architecture.png" alt="Diagrama de la arquitectura de la solución Despliegue de apps sin servidor entre varias regiones"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Análisis de registros y supervisión del estado de las aplicaciones con LogDNA y Sysdig
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilice IBM Log Analysis con LogDNA para comprender y diagnosticar actividades de aplicaciones. Supervise las aplicaciones con IBM Cloud Monitoring con Sysdig.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution12/Architecture.png" alt="Diagrama de la arquitectura de la solución Análisis de registros y supervisión del estado de las aplicaciones con LogDNA y Sysdig"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#isolated-cloud-foundry-enterprise-apps">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Apps de empresa aisladas de Cloud Foundry
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Proporcione una plataforma de innovación a su organización mediante el despliegue de una plataforma de Cloud Foundry aislada a nivel de empresa mediante Cloud Foundry Enterprise Environment.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution45-CFEE-apps/Architecture.png" alt="Diagrama de arquitectura para la solución Apps de empresa aisladas de Cloud Foundry"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Chatbots
{: #chatbots }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-slack-chatbot-database-watson#slack-chatbot-database-watson">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Creación de un Slackbot controlado por base de datos
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Creación de un Slackbot controlado por base de datos con IBM Watson Assistant, Cloudant e IBM Cloud Functions.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution19/SlackbotArchitecture.png" alt="Diagrama de arquitectura para la solución Creación de un Slackbot controlado por base de datos"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-watson-chatbot#android-watson-chatbot">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Creación de un chatbot Android con soporte de voz
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Defina intenciones y entidades y cree un flujo de diálogo para que el chatbot responda a las consultas de los clientes. Habilite los servicios de voz a texto y de texto a voz para facilitar la interacción con la app Android.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution28-watson-chatbot-android/architecture.png" alt="Diagrama de arquitectura para la solución Creación de un chatbot Android con soporte de voz"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Seguridad
{: #security }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicación web segura en varias regiones
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Creación, protección, despliegue y equilibrio de la carga de una aplicación web en varias regiones mediante un conducto de entrega continua.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution1/Architecture.png" alt="Diagrama de arquitectura para la solución Aplicación web segura en varias regiones"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Clústeres de varias regiones resistentes y seguros con clústeres de Kubernetes
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Integre los servicios de Internet de la nube con clústeres de Kubernetes para ofrecer una solución segura y resistente en varias regiones.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution32-multi-region-k8s-cis/Architecture.png" alt="Diagrama de arquitectura para la solución Clústeres de varias regiones resistentes y seguros con clústeres de Kubernetes"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Creación, protección y gestión de las API REST
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una nueva API REST mediante la infraestructura de API LoopBack Node.js. Añada funciones de gestión, visibilidad, seguridad y limitación de velocidad a la API mediante el servicio de API Connect en IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution13/Architecture.png" alt="Diagrama de arquitectura para la solución Creación, protección y gestión de las API REST"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-e2e-security#cloud-e2e-security">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicación de seguridad de extremo a extremo a una aplicación de nube
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una aplicación de nube segura que incluya datos cifrados con sus propias claves, autenticación de usuario y auditoría de seguridad.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution34-cloud-e2e-security/Architecture.png" alt="Diagrama de la arquitectura de la solución Aplicación de seguridad de extremo a extremo a una aplicación de nube"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Móvil
{: #mobile }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                App móvil iOS con Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una aplicación Swift de iOS con Push Notifications en IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution6/Architecture.png" alt="Diagrama de arquitectura para la solución App móvil iOS con Push Notifications"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                App móvil nativa de Android con Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Escriba una aplicación nativa de Android con Push Notifications en IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution9/Architecture.png" alt="Diagrama de arquitectura para la solución App móvil nativa de Android con Push Notifications"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-hybrid-mobile-push-analytics#hybrid-mobile-push-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicación móvil híbrida con Push Notifications
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Desarrolle una aplicación de Cordova híbrida con Push Notifications en IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution15/Architecture.png" alt="Diagrama de arquitectura para la solución Aplicación móvil híbrida con Push Notifications"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-mobile-backend#serverless-mobile-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aplicación móvil con un programa de fondo sin servidor
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilice Cloud Functions con servicios de datos y cognitivos para crear un programa de fondo sin servidor para una aplicación móvil.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution11/Architecture.png" alt="Diagrama de arquitectura para la solución Aplicación móvil con un programa de fondo sin servidor"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Aprendizaje automático y analítica
{: #ml }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Registros de Big Data con análisis continuo y SQL
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Recopile, almacene y analice registros para dar soporte a los requisitos normativos y al descubrimiento de información de ayuda. Mediante la mensajería de publicación/suscripción, escale la solución a millones de registros y luego realice un análisis en los registros persistentes con el sistema SQL con el que está familiarizado.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution31/Architecture.png" alt="Diagrama de arquitectura para la solución Registros de big data con análisis continuo y SQL"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-smart-data-lake#smart-data-lake">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Creación de un lago de datos con Object Storage
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Suministre herramientas a los analistas de datos para que consulten datos con SQL Query y realicen el análisis en Watson Studio. Comparta datos e información significativa mediante diagramas interactivos.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution29/architecture.png" alt="Diagrama de arquitectura para la solución Creación de un lago de datos con Object Storage"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-serverless-github-traffic-analytics#serverless-github-traffic-analytics">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Combinación de método sin servidor y Cloud Foundry para el análisis y la recuperación de datos
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Recopile automáticamente estadísticas de tráfico de GitHub para los repositorios, almacénelos en una base de datos SQL y empiece a utilizar la analítica de tráfico.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution24-github-traffic-analytics/Architecture.png" alt="Diagrama de arquitectura para la solución Combinación de método sin servidor y Cloud Foundry para el análisis y la recuperación de datos"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#create-deploy-retrain-machine-learning-model">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Creación, despliegue, prueba y reentrenamiento de un modelo predictivo de aprendizaje automático
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree un modelo predictivo de aprendizaje automático, despliéguelo como una API, pruebe y vuelva a entrenar el modelo con datos de los comentarios recibidos.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution22-build-machine-learning-model/architecture_diagram.png" alt="Diagrama de arquitectura para la solución Creación, despliegue, prueba y reentrenamiento de un modelo predictivo de aprendizaje automático"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-big-data-analytics-spark#big-data-analytics-spark">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Análisis y visualización de datos abiertos con Apache Spark
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Analice y visualice conjuntos de datos abiertos mediante Jupyter Notebook. Utiliza el servicio Apache Spark con IBM Watson Studio y Pixiedust para generar gráficos.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution23/Architecture.png" alt="Diagrama de la arquitectura de la solución Análisis y visualización de datos abiertos con Apache Spark"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Internet de las cosas
{: #iot }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#gather-visualize-analyze-iot-data">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Recopilación, visualización y análisis de datos IoT
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configure un dispositivo IoT, recopile grandes cantidades de datos en la plataforma Watson IoT, analice los datos con el aprendizaje automático y cree visualizaciones.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution16/Architecture.png" alt="Diagrama de la arquitectura de la solución Recopilación, visualización y análisis de datos IoT"/>
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
                Prácticas recomendadas para organizar usuarios, equipos, aplicaciones
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Una visión general de los conceptos disponibles en IBM Cloud para controlar la gestión de identidad y acceso y de cómo se pueden implementar para dar soporte a las diversas fases de desarrollo de una aplicación.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution20-users-teams-applications/architecture.png" alt="Diagrama de arquitectura para la solución Prácticas recomendadas para organizar usuarios, equipos, aplicaciones"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-cloud-usage#cloud-usage">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Revisión de servicios, recursos y uso de IBM Cloud
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Introducción a diversos enfoques utilizados para responder a preguntas comunes relacionadas con el uso.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution38/Architecture.png" alt="Diagrama de arquitectura para la solución Revisión de servicios, recursos y uso de IBM Cloud"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

## Kubernetes
{: #Network }

<div class = "solutionBoxContainer">
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Subredes privadas y públicas en una nube privada virtual
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una nube privada virtual con subredes e instancias. Proteja sus recursos mediante adjuntando grupos de seguridad y solo permita un acceso mínimo.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution40-vpc-public-app-private-backend/Architecture.png" alt="Diagrama de arquitectura para la solución Subredes privadas y públicas en una nube privada virtual"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-multi-region#vpc-multi-region">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Despliegue de cargas de trabajo aisladas en varias ubicaciones y zonas
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Despliegue una carga de trabajo en nubes privadas virtuales en varias zonas y regiones. Distribuya el tráfico entre las zonas con equilibradores de carga locales y globales.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution41-vpc-multi-region/Architecture.png" alt="Diagrama de la arquitectura de la solución Despliegue de cargas de trabajo aisladas en varias ubicaciones y zonas"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-site2site-vpn#vpc-site2site-vpn">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Utilización de una pasarela VPC/VPN para conseguir un acceso seguro y privado local a los recursos de la nube
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Conecte una red privada virtual a otro entorno de cálculo a través de una red privada virtual segura y consuma los servicios de IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution46-vpc-vpn/ArchitectureDiagram.png" alt="Diagrama de arquitectura para la solución Utilización de una pasarela VPC/VPN para conseguir un acceso seguro y privado local a los recursos de la nube"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server#vpc-secure-management-bastion-server">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Acceso seguro a instancias remotas con un host bastión
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Despliegue un host bastión para acceder de forma segura a instancias remotas dentro de una nube privada virtual.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png" alt="Diagrama de arquitectura para la solución Acceso seguro a instancias remotas con un host bastión"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Aislamiento de cargas de trabajo con una red privada segura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Configure un dispositivo de direccionador virtual para crear un alojamiento seguro. Asocie VLAN, suministre servidores y configure direccionamiento IP y cortafuegos.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution33-secure-network-enclosure/Secure-priv-enc.png" alt="Diagrama de arquitectura para la solución Aislamiento de cargas de trabajo con una red privada segura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-nat-config-private#nat-config-private">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Configuración de NAT para el acceso a Internet desde una red privada
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Utilice máscaras de NAT para convertir las direcciones IP privadas en una interfaz pública de salida.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution35-nat-config-private/vra-nat.png" alt="Diagrama de arquitectura para la solución Configuración de NAT para el acceso a Internet desde una red privada"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-byoip#byoip">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Traiga su propia dirección IP (BYOIP)
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Una visión general sobre los patrones de implementación de BYOIP y una guía para identificar el patrón adecuado.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution37-byoip/byoipdecision.png" alt="Diagrama de arquitectura para la solución Traiga su propia dirección IP (BYOIP)"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                VPN en una red privada segura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una conexión privada entre un entorno de red remoto y servidores en la red privada de IBM Cloud.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png" alt="Diagrama de arquitectura para la solución VPN en una red privada segura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-vlan-spanning#vlan-spanning">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Enlace de redes privadas seguras a través de la red de IBM
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Despliegue dos redes privadas que estén enlazadas de forma segura a través de la red privada de IBM Cloud mediante el servicio VLAN Spanning.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution43-vlan-spanning/vlan-spanning.png" alt="Diagrama de arquitectura para la solución Enlace de redes privadas seguras a través de la red de IBM"/>
                </div>
            </div>
        </div>
    </div>
    </a>
    <a href = "/docs/tutorials?topic=solution-tutorials-web-app-private-network#web-app-private-network">
    <div class = "solutionBox">
        <div class = "solutionBoxContent">
            <h3 class="solutionBoxTitle">
                Servicio de aplicaciones web desde una red privada segura
            </h3>
            <div class="solutionBoxDescription">
                <div class="descriptionContainer">
                    <p>Cree una aplicación web orientada a Internet escalable y segura alojada en una red privada protegida mediante un dispositivo de direccionador virtual (VRA), VLAN, NAT y cortafuegos.</p>
                </div>
                <div class="architectureDiagramContainer">
                    <img class="architectureDiagram" src = "images/solution42-web-app-private-network/web-app-private.png" alt="Diagrama de arquitectura para la solución Servicio de aplicaciones web desde una red privada segura"/>
                </div>
            </div>
        </div>
    </div>
    </a>
</div>

