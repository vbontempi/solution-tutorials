---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-19"
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

# Prácticas recomendadas para organizar usuarios, equipos, aplicaciones
{: #users-teams-applications}

En esta guía de aprendizaje se ofrece una visión general de los conceptos disponibles en {{site.data.keyword.cloud_notm}} para controlar la gestión de identidad y acceso y de cómo se pueden implementar para dar soporte a las diversas fases de desarrollo de una aplicación.
{:shortdesc}

Al crear una aplicación, es muy común definir varios entornos que reflejan el ciclo de vida de desarrollo de un proyecto, desde un desarrollador que confirma el código de la aplicación hasta la puesta del código de la aplicación a disposición de los usuarios finales. *Recinto de pruebas*, *prueba*, *transferencia*, *UAT* (prueba de aceptación del usuario), *preproducción*, *producción* son nombres típicos para estos entornos.

Aislar los recursos subyacentes, implementar políticas de control y acceso, proteger una carga de trabajo de producción y validar los cambios antes de enviarlos a la producción son algunos de los motivos por los que resulta recomendable crear estos entornos separados.

## Objetivos
{: #objectives}

* Obtención de información sobre {{site.data.keyword.iamlong}} y los modelos de acceso de Cloud Foundry
* Configuración de un proyecto con separación entre roles y entornos
* Configuración de la integración continua

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.iamlong}}](https://{DomainName}/docs/iam?topic=iam-iamoverview#iamoverview)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [Cloud Foundry](https://{DomainName}/catalog/?category=cf-apps&search=foundry)
* [{{site.data.keyword.cloudantfull}}](https://{DomainName}/catalog/services/cloudant)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Definición de un proyecto

Trabajaremos con un proyecto de ejemplo con los componentes siguientes:
* varios microservicios desplegados en {{site.data.keyword.containershort_notm}},
* bases de datos,
* grupos de almacenamiento de archivos.

En este proyecto, definiremos tres entornos:
* *Desarrollo*: este entorno se actualiza continuamente con cada confirmación, pruebas de unidades, pruebas de humo que se ejecutan. Ofrece acceso al despliegue más reciente y mejor del proyecto.
* *Prueba*: este entorno se crea después de una ramificación o etiquetado estable del código. Aquí es donde se realizan las pruebas de aceptación de usuario. Su configuración es similar a la del entorno de producción. Se carga con datos realistas (datos de producción anónimos, por ejemplo).
* *Producción*: este entorno se actualiza con la versión validada en el entorno anterior.

**Un conducto de entrega gestiona el progreso de una compilación en el entorno.** Puede ser totalmente automatizado o puede incluir puertas de validación manual para promocionar las compilaciones aprobadas entre entornos; es un entorno realmente abierto y se debe configurar para que se ajuste a las prácticas recomendadas y a los flujos de trabajo de la empresa.

Para dar soporte a la ejecución del conducto de compilación, incorporamos **un usuario funcional**: un usuario de {{site.data.keyword.cloud_notm}} normal, pero que es miembro de equipo sin identidad real en el mundo físico. Este usuario funcional será el propietario de los conductos de entrega y de cualquier otro recurso de nube que requiera una propiedad fuerte. Este enfoque ayuda en el caso de que un miembro del equipo abandone la empresa o se pase a otro proyecto. El usuario funcionar estará dedicado a su proyecto y no cambiará durante el ciclo de vida del proyecto. Lo siguiente que deseará crear es [una clave de API](https://{DomainName}/docs/iam?topic=iam-manapikey#manapikey) para este usuario funcional. Seleccionará esta clave de API cuando configure los conductos de DevOps o cuando desee ejecutar scripts de automatización, para suplantar al usuario funcional.

Cuando tengamos que asignar responsabilidades a los miembros del equipo del proyecto, definiremos los roles siguientes y los permisos relacionados:

|           | Desarrollo | Prueba | Producción |
| --------- | ----------- | ------- | ---------- |
| Desarrollador | <ul><li>contribuye al código</li><li>puede acceder a los archivos de registro</li><li>puede ver la configuración de la app y del servicio</li><li>utiliza las aplicaciones desplegadas</li></ul> | <ul><li>puede acceder a los archivos de registro</li><li>puede ver la configuración de la app y del servicio</li><li>utiliza las aplicaciones desplegadas</li></ul> | <ul><li>sin acceso</li></ul> |
| Comprobador    | <ul><li>utiliza las aplicaciones desplegadas</li></ul> | <ul><li>utiliza las aplicaciones desplegadas</li></ul> | <ul><li>sin acceso</li></ul> |
| Operador  | <ul><li>puede acceder a los archivos de registro</li><li>puede ver y definir la configuración de la app y del servicio</li></ul> | <ul><li>puede acceder a los archivos de registro</li><li>puede ver y definir la configuración de la app y del servicio</li></ul> | <ul><li>puede acceder a los archivos de registro</li><li>puede ver y definir la configuración de la app y del servicio</li></ul> |
| Usuario funcional del conducto  | <ul><li>puede desplegar/retirar el despliegue de aplicaciones</li><li>puede ver y definir la configuración de la app y del servicio</li></ul> | <ul><li>puede desplegar/retirar el despliegue de aplicaciones</li><li>puede ver y definir la configuración de la app y del servicio</li></ul> | <ul><li>puede desplegar/retirar el despliegue de aplicaciones</li><li>puede ver y definir la configuración de la app y del servicio</li></ul> |

## Identity and Access Management (IAM)
{: #first_objective}

{{site.data.keyword.iamshort}} (IAM) le permite autenticar usuarios de forma segura en los servicios de plataforma e infraestructura y controlar el acceso a los **recursos** de forma coherente en la plataforma de {{site.data.keyword.cloud_notm}}. Hay un conjunto de servicios de {{site.data.keyword.cloud_notm}} habilitados para utilizar Cloud IAM para controlar el acceso y están organizados en **grupos de recursos** dentro de su **cuenta** para poder proporcionar a los **usuarios** un acceso rápido y fácil a más de un recurso a la vez. Las **políticas** de acceso de Cloud IAM se utilizan para asignar accesos de usuario e ID de servicio a los recursos de su cuenta.

Una **política** asigna a un usuario o a un ID de servicio uno o varios **roles** con una combinación de atributos que define el ámbito de acceso. La política puede proporcionar acceso a un único servicio a nivel de instancia o puede aplicarse a un conjunto de recursos organizados juntos en un grupo de recursos. En función de los roles de usuario que asigne, al usuario o al ID de servicio se le permiten diversos niveles de acceso para completar tareas de gestión de la plataforma y acceder a un servicio utilizando la interfaz de usuario o realizar tipos de llamadas API específicos.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/iam-model.png" style="width: 70%;" alt="Diagrama del modelo de IAM" />
</p>

En este momento, no todos los servicios del catálogo de {{site.data.keyword.cloud_notm}} se pueden gestionar mediante IAM. Para estos servicios, puede seguir utilizando Cloud Foundry proporcionando a los usuarios acceso a la organización y al espacio a los que pertenece la instancia con un rol de Cloud Foundry asignado para definir el nivel de acceso permitido.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cloudfoundry-model.png" style="width: 70%;" alt="Diagrama del modelo de Cloud Foundry" />
</p>

## Creación de los recursos para un entorno

Aunque los tres entornos que necesita este proyecto de ejemplo requieren diferentes derechos de acceso y es posible que se les tengan que asignar diferentes capacidades, comparten un patrón de arquitectura común.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/one-environment.png" style="width: 80%;" alt="Diagrama de arquitectura que muestra un entorno" />
</p>

Comenzaremos por crear el entorno de desarrollo.

1. [Seleccione una ubicación de {{site.data.keyword.cloud_notm}}](https://{DomainName}) en la que desplegar el entorno.
1. Para servicios y apps de Cloud Foundry:
   1. [Cree una organización para el proyecto](https://{DomainName}/docs/account?topic=account-orgsspacesusers#createorg).
   1. [Cree un espacio de Cloud Foundry para el entorno](https://{DomainName}/docs/account?topic=account-orgsspacesusers#spaceinfo).
   1. Cree los servicios de Cloud Foundry que utiliza el proyecto bajo este espacio
1. [Cree un grupo de recursos para el entorno](https://{DomainName}/account/resource-groups).
1. Cree los servicios compatibles con el grupo de recursos, como {{site.data.keyword.cos_full_notm}}, {{site.data.keyword.la_full_notm}}, {{site.data.keyword.mon_full_notm}} y {{site.data.keyword.cloudant_short_notm}}, en este grupo.
1. [Cree un clúster de Kubernetes](https://{DomainName}/containers-kubernetes/catalog/cluster) en {{site.data.keyword.containershort_notm}}; asegúrese de seleccionar el grupo de recursos que ha creado antes.
1. Configure {{site.data.keyword.la_full_notm}} y {{site.data.keyword.mon_full_notm}} para enviar registros y para supervisar el clúster.

En el diagrama siguiente se muestra dónde se crean los recursos del proyecto en la cuenta:

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/resources.png" style="height: 400px;" alt="Diagrama que muestra los recursos del proyecto" />
</p>

## Asignación de roles dentro del entorno

1. Invite usuarios a la cuenta
1. Asigne políticas a los usuarios para controlar quién puede acceder al grupo de recursos, los servicios dentro del grupo y la instancia de {{site.data.keyword.containershort_notm}} y sus permisos. Consulte la [definición de políticas de acceso](https://{DomainName}/docs/containers?topic=containers-users#access_policies) para seleccionar las políticas adecuadas para un usuario en el entorno. Los usuarios con el mismo conjunto de políticas se pueden colocar en el [mismo grupo de acceso](https://{DomainName}/docs/iam?topic=iam-groups#groups). Esto simplifica la gestión de usuarios, ya que las políticas se asignarán al grupo de acceso y las heredarán todos los usuarios del grupo.
1. Configure los roles de la organización y del espacio de Cloud Foundry en función de sus necesidades dentro del entorno. Consulte la [definición de roles](https://{DomainName}/docs/iam?topic=iam-cfaccess#cfaccess) para asignar los roles adecuados en función del entorno.

Consulte la documentación de los servicios para entender cómo un servicio correlaciona los roles de IAM y de Cloud Foundry con acciones específicas. Consulte, por ejemplo, [cómo el servicio {{site.data.keyword.mon_full_notm}} correlaciona roles de IAM con acciones](https://{DomainName}/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam#iam).

Para asignar los roles adecuados a los usuarios hay que realizar varias iteraciones para ir mejorando la asignación. Los permisos otorgados se pueden controlar a nivel de grupo de recursos, para todos los recursos de un grupo o se pueden detallar para una instancia específica de un servicio; con el tiempo irá descubriendo cuáles son las políticas de acceso ideales para el proyecto.

Una buena práctica consiste en empezar con el conjunto mínimo de permisos y luego ampliarlo cuidadosamente según sea necesario. Para Kubernetes, se recomienda examinar su [Control de acceso basado en roles (RBAC)](https://kubernetes.io/docs/admin/authorization/rbac/) para configurar las autorizaciones en el clúster.

En el caso del entorno de desarrollo, las responsabilidades de usuario definidas anteriormente podrían traducirse en lo siguiente:

|           | Políticas de acceso de IAM | Cloud Foundry |
| --------- | ----------- | ------- |
| Desarrollador | <ul><li>Grupo de recursos: *Visor*</li><li>Roles de acceso a la plataforma en el grupo de recursos: *Visor*</li><li>Rol del servicio de registro y supervisión: *Escritor*</li></ul> | <ul><li>Rol de la organización: *Auditor*</li><li>Rol del espacio: *Auditor*</li></ul> |
| Comprobador    | <ul><li>No se necesita ninguna configuración. El comprobador accede a la aplicación desplegada, no a los entornos de desarrollo</li></ul> | <ul><li>No se necesita ninguna configuración</li></ul> |
| Operador  | <ul><li>Grupo de recursos: *Visor*</li><li>Roles de acceso a la plataforma en el grupo de recursos: *Operador*, *Visor*</li><li>Rol del servicio de registro y supervisión: *Escritor*</li></ul> | <ul><li>Rol de la organización: *Auditor*</li><li>Rol del espacio: *Desarrollador*</li></ul> |
| Usuario funcional del conducto | <ul><li>Grupo de recursos: *Visor*</li><li>Roles de acceso a la plataforma en el grupo de recursos: *Editor*, *Visor*</li></ul> | <ul><li>Rol de la organización: *Auditor*</li><li>Rol del espacio: *Desarrollador*</li></ul> |

Las políticas de acceso de IAM y los roles de Cloud Foundry se definen en la [interfaz de usuario de gestión de identidades y acceso](https://{DomainName}/iam/#/users):

<p style="text-align: center;">
  <img title="" src="./images/solution20-users-teams-applications/edit-policy.png" alt="Configuración de permisos para el rol de desarrollador" />
</p>

## Duplicación para varios entornos

A partir de ahí, puede duplicar pasos similares para crear los otros entornos.

1. Cree un grupo de recursos por entorno.
1. Cree un clúster y las instancias de servicio necesarias por entorno.
1. Cree un espacio de Cloud Foundry por entorno.
1. Cree las instancias de servicio necesarias en cada espacio.

<p style="text-align: center;">
  <img title="Utilización de distintos clústeres para aislar entornos" src="./images/solution20-users-teams-applications/multiple-environments.png" style="width: 80%;" alt="Diagrama que muestra distintos clústeres para aislar entornos" />
</p>

Mediante el uso de una combinación de herramientas como la [CLI de {{site.data.keyword.cloud_notm}}`ibmcloud`](https://github.com/IBM-Cloud/ibm-cloud-developer-tools), [`terraform` de HashiCorp](https://www.terraform.io/), el [proveedor de {{site.data.keyword.cloud_notm}} para Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm), la CLI de Kubernetes `kubectl`, puede crear scripts y automatizar la creación de estos entornos.

El hecho de disponer de varios clústeres de Kubernetes para los entornos ofrece varias ventajas:
* independientemente del entorno, todos los clústeres se parecerán;
* resulta más fácil controlar quién tiene acceso a un determinado clúster;
* proporciona flexibilidad en los ciclos de actualización para despliegues y recursos subyacentes; cuando hay una nueva versión de Kubernetes, le ofrece la opción de actualizar primero el clúster de desarrollo, validar la aplicación y luego actualizar el otro entorno;
* evita la combinación de diferentes cargas de trabajo que pueden tener un impacto entre sí, como aislar el despliegue de producción de los otros.

Otro enfoque consiste en utilizar [espacios de nombres de Kubernetes](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) junto con las [cuotas de recursos de Kubernetes](https://kubernetes.io/docs/concepts/policy/resource-quotas/) para aislar entornos y controlar el consumo de recursos.

<p style="text-align: center;">
  <img title="Utilización de espacios de nombres separados para aislar entornos" src="./images/solution20-users-teams-applications/multiple-environments-with-namespaces.png" style="width: 80%;" alt="Diagrama que muestra diversos espacios de nombres para aislar entornos" />
</p>

En el recuadro de entrada `Buscar` de la IU de LogDNA, utilice el campo `espacio de nombres: ` para filtrar los registros en función del espacio de nombres.
{: tip}

## Configuración del conducto de entrega

Cuando se trata de desplegar en los distintos entornos, se puede configurar la integración continua y la entrega continua para impulsar todo el proceso:
* actualice continuamente el entorno de `Desarrollo` con el código más reciente y mejor de la ramificación de `desarrollo`, mediante la ejecución de pruebas de unidad y de pruebas de integración en el clúster dedicado;
* promocione compilaciones de desarrollo en el entorno de `Prueba`, ya sea automáticamente si todas las pruebas de las etapas anteriores resultan correctas o mediante un proceso de promoción manual. Algunos equipos también utilizarán diferentes ramificaciones, fusionando el estado de desarrollo funcional a una rama `estable` como ejemplo;
* Repita un proceso similar para pasar al entorno de `Producción`.

<p style="text-align: center;">
  <img src="./images/solution20-users-teams-applications/cicd.png" alt="Un conducto CI/CD entre compilación y despliegue" />
</p>

Cuando configure el conducto de DevOps, asegúrese de utilizar la clave de API de un usuario funcional. Solo el usuario funcional debería tener los derechos necesarios para desplegar apps en los clústeres.

Durante la fase de compilación, es importante que las imágenes de Docker tengan la versión correcta. Puede utilizar la revisión de confirmación de Git como parte de la etiqueta de la imagen, o un identificador exclusivo proporcionado por la cadena de herramientas de DevOps; cualquier identificador que facilite la correlación de la imagen con la compilación real y el código fuente contenidos en la imagen.

A medida que se familiarice con Kubernetes, [Helm](https://helm.sh/), el gestor de paquetes de Kubernetes, se convertirá en una herramienta útil para crear versiones, ensamblar y desplegar la aplicación. [Esta cadena de herramientas de DevOps de ejemplo](https://github.com/open-toolchain/simple-helm-toolchain) constituye un buen punto de partida y está preconfigurada para entrega continua a un clúster de Kubernetes. A medida que el proyecto crezca con varios microservicios, el [diagrama de paraguas de Helm](https://github.com/kubernetes/helm/blob/master/docs/charts_tips_and_tricks.md#complex-charts-with-many-dependencies) le proporcionará una buena solución para componer la aplicación.

## Ampliación de la guía de aprendizaje

Enhorabuena, su aplicación ya se puede desplegar de forma segura de desarrollo a producción. A continuación encontrará algunas sugerencias para mejorar la entrega de aplicaciones.

* Añada [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) al conducto para realizar un control de calidad durante los despliegues.
* Revise las contribuciones al código de los miembros del equipo y las interacciones entre desarrolladores y [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).
* Siga la guía de aprendizaje sobre [Planificación, creación y actualización de entornos de despliegue](https://{DomainName}/docs/tutorials?topic=solution-tutorials-plan-create-update-deployments#plan-create-update-deployments) para automatizar el despliegue de sus entornos.

## Información relacionada

* [Iniciación a {{site.data.keyword.iamshort}}](https://{DomainName}/docs/iam?topic=iam-getstarted#getstarted)
* [Mejores prácticas para organizar los recursos en un grupo de recursos](https://{DomainName}/docs/resources?topic=resources-bp_resourcegroups#bp_resourcegroups)
* [Análisis de registros y supervisión del estado con LogDNA y Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)
* [Despliegue continuo en Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
* [Cadena de herramientas Hello Helm](https://github.com/open-toolchain/simple-helm-toolchain)
* [Desarrollo de una aplicación de microservicios con Kubernetes y Helm](https://github.com/open-toolchain/microservices-helm-toolchain)
* [Concesión de permisos a un usuario para ver registros en LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam)
* [Concesión de permisos a un usuario para ver métricas en Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work)
