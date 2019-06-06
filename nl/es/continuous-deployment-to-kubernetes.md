---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-15"


---


{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:pre: .pre}
{:tip: .tip}


# Despliegue continuo en Kubernetes
{: #continuous-deployment-to-kubernetes}

En esta guía de aprendizaje se muestra el proceso de configuración de una integración y de un conducto de entrega continua para aplicaciones contenerizadas que se ejecutan en {{site.data.keyword.containershort_notm}}.  Aprenderá a configurar el control de origen y, luego a crear, probar y desplegar el código en distintas etapas de despliegue. A continuación, añadirá integraciones a otros servicios, como escáneres de seguridad, notificaciones de Slack y analítica.

{:shortdesc}

## Objetivos
{: #objectives}

* Creación de clústeres de Kubernetes de desarrollo y de producción.
* Creación de una aplicación de inicio, ejecución de la misma localmente y envío a un repositorio Git.
* Configuración del conducto de entrega DevOps para conectar con el repositorio Git, crear y desplegar la app de inicio en clústeres de desarrollo y producción.
* Exploración e integración de la app para que utilice escáneres de seguridad, notificaciones de Slack y analítica.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes servicios de {{site.data.keyword.Bluemix_notm}}:

- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
- [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery)
- Slack

**Atención:** esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

![](images/solution21/Architecture.png)

1. Se envía código a un repositorio Git privado.
2. El conducto toma los cambios en Git y crea la imagen del contenedor.
3. La imagen de contenedor se carga en el registro desplegado en un clúster de Kubernetes de desarrollo.
4. Los cambios se validan y se despliegan en el clúster de producción.
5. Se configuran notificaciones de Slack para las actividades de despliegue.


## Antes de empezar
{: #prereq}

* [Instale {{site.data.keyword.dev_cli_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli): script para instalar docker, kubectl, helm, la cli de ibmcloud y los plugins necesarios.
* [Configure la CLI de {{site.data.keyword.registrylong_notm}} y el espacio de nombres del registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Comprenda los conceptos básicos de Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Creación de clúster de Kubernetes de desarrollo
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} combina Docker y
Kubernetes para ofrecer herramientas potentes, una interfaz intuitiva para el usuario y funciones integradas de seguridad e identificación para automatizar el despliegue, operación, escalado y supervisión de apps contenerizadas sobre un clúster de hosts de cálculo.

Para completar esta guía de aprendizaje, tendrá que seleccionar el clúster de **Pago** de tipo **Estándar**. Tendrá que configurar dos clústeres, uno para desarrollo y otro para producción.
{: shortdesc}

1. Cree el primer clúster de Kubernetes de desarrollo desde el [catálogo de {{site.data.keyword.Bluemix}}](https://{DomainName}/containers-kubernetes/launch). Luego tendrá que repetir estos pasos y crear un clúster de producción.

   Para realizar un uso adecuado, compruebe los detalles de configuración, como el número de CPU, la memoria y el número de nodos trabajadores que se obtienen.
   {:tip}

   ![](images/solution21/KubernetesPaidClusterCreation.png)

2. Seleccione el **Tipo de clúster** y pulse **Crear clúster** para suministrar un clúster de Kubernetes. El **Tipo de máquina** más pequeño con 2 **CPU**, 4 **GB de RAM** y 1 **Nodo trabajador** es suficiente para esta guía de aprendizaje. Se pueden dejar los valores predeterminados de todas las demás opciones.
3. Compruebe el estado del **Clúster** y de los **Nodos trabajadores** y espere a que estén **listos**.

**Nota:** no continúe hasta que los nodos trabajadores estén listos.

## Creación de una aplicación inicial
{: #create_application}

{{site.data.keyword.containershort_notm}} ofrece una selección de aplicaciones de inicio; estas aplicaciones de inicio se pueden crear con el mandato `ibmcloud dev create` o en la consola web. En esta guía de aprendizaje, vamos a utilizar la consola web. La aplicación de inicio reduce considerablemente el tiempo de desarrollo mediante la generación de iniciadores de aplicaciones con todos los contenedores modelo necesarios, la creación y la configuración del código para que pueda empezar a codificar rápidamente la lógica necesaria para la empresa.

1. En la [consola de {{site.data.keyword.cloud_notm}}](https://{DomainName}), utilice la opción del menú de la izquierda y seleccione [Apps web](https://{DomainName}/developer/appservice/dashboard).
2. En la sección **Empezar desde la web**, pulse el botón **Iniciación**.
3. Seleccione el mosaico `App web Node.js con Express.js` y luego `Crear app` para crear una aplicación de inicio Node.js.
4. Especifique el **nombre** `mynodestarter`. A continuación, pulse en **Crear**.

## Configuración del conducto de entrega DevOps
{: #create_devops}

1. Ahora que ha creado correctamente la aplicación de inicio, en **Desplegar la app**, pulse el botón **Desplegar en la nube**.
2. Cuando seleccione el método de despliegue de clúster de Kubernetes, seleccione el clúster creado anteriormente y pulse **Crear**. Esto creará una cadena de herramientas y un conducto de entrega. ![](images/solution21/BindCluster.png)
3. Una vez creado el conducto, pulse **Ver cadena de herramientas** y luego **Conducto de entrega** para ver el conducto. ![](images/solution21/Delivery-pipeline.png)
4. Cuando finalicen las etapas de despliegue, pulse **Ver registros e historial** para ver los registros.
5. Visite el URL que se muestra para acceder a la aplicación (`http://worker-public-ip:portnumber/`). ![](images/solution21/Logs.png)
Hecho; ha utilizado la interfaz de usuario de servicio de app para crear las aplicaciones de inicio y ha configurado el conducto para crear y desplegar la aplicación en el clúster.

## Clonación, compilación y ejecución de la aplicación localmente
{: #cloneandbuildapp}

En esta sección, utilizará la app de inicio creada en la sección anterior, la clonará en la máquina local, modificará el código y, a continuación, la compilará y la ejecutará localmente.
{: shortdesc}

### Clonación de la aplicación
1. En la visión general de la cadena de herramientas, seleccione el mosaico **Git** bajo **Código**. Se le redirigirá a la página del repositorio de git, donde puede clonar el repositorio.![HelloWorld](images/solution21/DevOps_Toolchain.png)

2. Si aún no ha configurado claves SSH, debería ver una barra de notificación en la parte superior con instrucciones. Siga los pasos abriendo el enlace **añadir una clave SSH** en un nuevo separador o, si desea utilizar HTTPS en lugar de SSH, siga los pasos pulsando **Crear una señal de acceso personal**. Recuerde guardar la clave o la señal para consultarla en el futuro.
3. Seleccione SSH o HTTPS y copie el URL de git. Clone el origen en la máquina local. Si se le solicita un nombre de usuario, especifique el nombre de usuario de git. Para la contraseña, utilice una **clave SSH** o una **señal de acceso personal** existente o la que se ha creado en el paso anterior.

   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   {: codeblock}

4. Abra el repositorio clonado en el IDE que elija y vaya a `public/index.html`. Actualice el código intentando cambiar "Congratulations!" por otra cosa y guarde el archivo.

### Compilación de la aplicación localmente
Puede crear y ejecutar la aplicación igual que lo haría con `mvn` para el desarrollo local de Java o con `npm` para el desarrollo de nodos.  También puede crear una imagen de docker y ejecutar la aplicación en un contenedor para garantizar la ejecución coherente localmente y en la nube. Siga los siguientes pasos para crear la imagen de docker.
{: shortdesc}

1. Asegúrese de que el motor de Docker local se ha iniciado; para comprobarlo ejecute este mandato:
   ```
   docker ps
   ```
   {: codeblock}
2. Vaya al directorio de proyecto generado clonado.
   ```
   cd <project name>
   ```
   {: codeblock}
3. Compile de la aplicación localmente.
   ```
   ibmcloud dev build
   ```
   {: codeblock}

   Esto puede tardar unos minutos en ejecutarse ya que se descargan todas las dependencias de la aplicación y se crea una imagen de Docker que contiene la aplicación y el entorno necesario.

### Ejecución local de la aplicación

1. Ejecute el contenedor.
   ```
   ibmcloud dev run
   ```
   {: codeblock}

   Esto utiliza el motor de Docker local para ejecutar la imagen de docker que ha creado en el paso anterior.
2. Después de que se inicie el contenedor, vaya a http://localhost:3000/
   ![](images/solution21/node_starter_localhost.png)

## Envío de la aplicación a su repositorio Git

En esta sección, confirmará el cambio en el repositorio Git. El conducto tomará la confirmación y enviará los cambios al clúster automáticamente.
1. En la ventana de su terminal, asegúrese de que está dentro del repositorio que ha clonado.
2. Envíe el cambio al repositorio con tres pasos simples: añadir (add), confirmar (commit) y enviar (push).
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
   {: codeblock}

3. Vaya a la cadena de herramientas que ha creado anteriormente y pulse el mosaico **Conducto de entrega**.
4. Confirme que ve la etapa n **BUILD** y **DEPLOY**.   ![](images/solution21/Delivery-pipeline.png)
5. Espere a que finalice la etapa **DEPLOY**.
6. Pulse el **URL** de la aplicación en el resultado de la última ejecución para ver los cambios en directo.

Si no ve que la aplicación se actualiza, consulte los registros de las etapas DEPLOY y BUILD del conducto.

## Utilización segura de Vulnerability Advisor
{: #vulnerability_advisor}

En este paso, explorará el programa [Vulnerability Advisor](https://{DomainName}/docs/services/va?topic=va-va_index#va_index). El programa Vulnerability Advisor comprueba el estado de seguridad de las imágenes de contenedor antes del despliegue, y también comprueba el estado de los contenedores en ejecución.

1. Vaya a la cadena de herramientas que ha creado anteriormente y pulse el mosaico **Conducto de entrega**.
1. Pulse **Añadir etapa** y cambie MyStage por **Etapa de validación** y luego pulse TRABAJOS > **AÑADIR TRABAJO**.

   1. Seleccione **Probar** como Tipo de trabajo y cambie **Prueba** por **Vulnerability Advisor** en el recuadro.
   1. En Tipo de prueba, seleccione **Vulnerability Advisor**. Los demás cambios se deberían rellenar automáticamente.
      El espacio de nombres del registro de contenedor debería ser el mencionado en **Crear etapa** de esta cadena de herramientas.
      {:tip}
   1. Edite la sección **Script de prueba** y sustituya `SAFE\ to\ deploy` en la última línea por `NO\ ISSUES`
   1. Guarde la etapa
1. Arrastre y mueva la **Etapa de validación** al centro y, a continuación, pulse **Ejecutar** ![](images/solution21/run.png) en **Etapa de validación**. Verá que la **Etapa de validación** falla.

   ![](images/solution21/toolchain.png)

1. Pulse **Ver registros e historial** para ver la evaluación de vulnerabilidades. Al final del registro encontrará lo siguiente:

   ```
   The scan results show that 3 ISSUES were found for the image.

   Configuration Issues Found
   ==========================

   Configuration Issue ID                     Policy Status   Security Practice                                    How to Resolve
   application_configuration:mysql.ssl-ca     Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-ca is not specified in /etc/mysql/my.cnf.
                                                              Certificate Authority (CA) certificate.
   application_configuration:mysql.ssl-cert   Active          A setting in /etc/mysql/my.cnf that specifies the    ssl-cert is not specified in /etc/mysql/my.cnf
                                                              server public key certificate. This certificate      file.
                                                              can be sent to the client and authenticated
                                                              against its CA certificate.
   application_configuration:mysql.ssl-key    Active          A setting in /etc/mysql/my.cnf that identifies the   ssl-key is not specified in /etc/mysql/my.cnf.
                                                              server private key.
   ```

   Encontrará la evaluación de vulnerabilidades detallada de todos los repositorios explorados [aquí](https://{DomainName}/containers-kubernetes/registry/private)
   {:tip}

   La etapa puede fallar con la indicación de que la imagen *no se ha explorado* si la exploración de vulnerabilidades tarda más de 3 minutos. Este tiempo de espera se puede cambiar editando el script del trabajo y aumentando el número de iteraciones para esperar a los resultados de la exploración.
   {:tip}

1. Vamos a arreglar las vulnerabilidades siguiendo la acción correctiva. Abra el repositorio clonado en un IDE o seleccione el mosaico IDE web de Eclipse Orion, abra `Dockerfile` y añada el mandato siguiente después de `EXPOSE 3000`
   ```sh
   RUN apt-get remove -y mysql-common \
     && rm -rf /etc/mysql
   ```
   {: codeblock}

1. Confirme y envíe los cambios. Esto debería activar la cadena de herramientas y arreglar la **Etapa de validación**.

   ```
   git add Dockerfile
   git commit -m "Fix Vulnerabilities"
   git push origin master
   ```

   {: codeblock}

## Creación de clúster de Kubernetes de producción

{: #deploytoproduction}

En esta sección, completará el conducto de despliegue desplegando la aplicación de Kubernetes en entornos de desarrollo y de producción respectivamente. Lo ideal es configurar un despliegue automático para el entorno de desarrollo y un despliegue manual para el entorno de producción. Antes de hacerlo, vamos a examinar las dos maneras de conseguirlo. Se puede utilizar un clúster para ambos entornos, el de desarrollo y el de producción. Sin embargo, se recomienda tener dos grupos separados, uno para desarrollo y otro para producción. Vamos a examinar la configuración de un segundo clúster para producción.
{: shortdesc}

1. Siguiendo las instrucciones de la sección [Creación de un clúster de Kubernetes de desarrollo](#create_kube_cluster), cree un nuevo clúster. Llame a este clúster `prod-cluster`.
2. Vaya a la cadena de herramientas que ha creado anteriormente y pulse el mosaico **Conducto de entrega**.
3. Cambie el nombre de **Deploy Stage** por `Deploy dev`; para ello, pulse el icono de valores >  **Configurar etapa**.
   ![](images/solution21/deploy_stage.png)
4. Clone la etapa **Deploy dev** (icono de valores > Clonar etapa) y llame a la etapa clonada `Deploy prod`.
5. Cambie **stage trigger** por `Run jobs only when this stage is run manually`. ![](images/solution21/prod-stage.png)
6. En el separador **Trabajo**, cambie el nombre del clúster por el clúster que acaba de crear y **guarde** la etapa.
7. Ahora debería tener la configuración de despliegue completa; para desplegar desde desarrollo en producción, debe ejecutar manualmente la etapa `Deploy prod` para desplegar en producción.![](images/solution21/full-deploy.png)

Hecho; ahora ha creado un clúster de producción y ha configurado el conducto para enviar actualizaciones a su clúster de producción manualmente. Se trata de una etapa de proceso de simplificación sobre un escenario más avanzado en el que incluiría pruebas de unidad y pruebas de integración como parte del conducto.

## Configuración de notificaciones de Slack
{: #setup_slack}

1. Vuelva para ver la lista de [cadenas de herramientas](https://{DomainName}/devops/toolchains) y seleccione su cadena de herramientas; luego pulse **Añadir una herramienta**.
2. Busque slack en el recuadro de búsqueda o desplácese hacia abajo hasta ver **Slack**. Pulse para ver la página de configuración.
![](images/solution21/configure_slack.png)
3. Para **Webhook de Slack**, siga los pasos de este [enlace](https://my.slack.com/services/new/incoming-webhook/). Tiene que iniciar la sesión con sus credenciales de Slack y proporcionar un nombre de canal existente o bien crear uno nuevo.
4. Cuando se haya añadido la integración del webhook de entrada, copie el **URL de webhook** y péguelo bajo **Webhook de Slack**.
5. El canal de slack es el nombre de canal que ha proporcionado al crear antes una integración de webhook.
6. **Nombre de equipo de Slack** es el nombre de equipo (primera parte) de team-name.slack.com. Por ejemplo, kube es el nombre de equipo en kube.slack.com
7. Pulse **Crear integración**. Se añadirá un nuevo mosaico a la cadena de herramientas.     ![](images/solution21/toolchain_slack.png)
8. A partir de ahora, siempre que se ejecute la cadena de herramientas, debería ver notificaciones de slack en el canal que ha configurado.
![](images/solution21/slack_channel.png)

## Eliminación de recursos
{: #removeresources}

En este paso limpiará los recursos para eliminar lo que ha creado anteriormente.

- Suprima el repositorio Git.
- Suprima la cadena de herramientas.
- Suprima los dos clústeres.
- Suprima el canal de Slack.

## Ampliación de la guía de aprendizaje
{: #expandTutorial}

¿Desea prender más? A continuación encontrará algunas ideas sobre lo que puede hacer a continuación:

- [Análisis de registros y supervisión del estado de las aplicaciones con LogDNA y Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Añada un entorno de prueba y despliéguelo en un tercer clúster.
- Despliegue del clúster de producción [en varias ubicaciones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Mejore el conducto con controles y analíticas de calidad adicionales mediante [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Contenido relacionado
{: #related}

* Guía de la solución Kubernetes de extremo a extremo, [transferencia de apps basadas en VM a Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vm-to-containers-and-kubernetes#vm-to-containers-and-kubernetes).
* [Seguridad](https://{DomainName}/docs/containers?topic=containers-security#cluster) del servicio IBM Cloud Container.
* [Integraciones](https://{DomainName}/docs/services/ContinuousDelivery?topic=ContinuousDelivery-integrations#integrations) de cadena de herramientas.
* Análisis de registros y supervisión del estado de las aplicaciones con [LogDNA y Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).


