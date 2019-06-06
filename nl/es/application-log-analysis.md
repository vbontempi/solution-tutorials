---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-29"


---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Análisis de registros y supervisión del estado de las aplicaciones con LogDNA y Sysdig
{: #application-log-analysis}

En esta guía de aprendizaje se muestra cómo se puede utilizar el servicio [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/) para configurar y acceder a registros de una aplicación de Kubernetes desplegada en {{site.data.keyword.Bluemix_notm}}. Desplegará una aplicación Python en un clúster suministrados en {{site.data.keyword.containerlong_notm}}, configurará un agente LogDNA, generará distintos niveles de registros de la aplicación y accederá a registros de nodos trabajadores, a registros de pod o a registros de red. A continuación, buscará, filtrará y visualizará los registros mediante la interfaz de usuario web de {{site.data.keyword.la_short}}.

También configurará el servicio [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/) y configurará un agente de Sysdig para supervisar el rendimiento y el estado de la aplicación y del clúster de {{site.data.keyword.containerlong_notm}}.
{:shortdesc}

## Objetivos
{: #objectives}
* Despliegue de una aplicación en un clúster de Kubernetes para generar entradas de registro.
* Acceso y análisis de distintos tipos de registro para resolver problemas y anticiparse a los mismos.
* Obtención de visibilidad operativa sobre el rendimiento y el estado de la app y del clúster que ejecuta la app.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.containerlong_notm}}](https://{DomainName}/kubernetes/landing)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.la_full_notm}}](https://{DomainName}/observe/logging)
* [{{site.data.keyword.mon_full_notm}}](https://{DomainName}/observe/monitoring)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

  ![](images/solution12/Architecture.png)

1. El usuario se conecta a la aplicación y genera entradas de registro.
1. La aplicación se ejecuta en un clúster de Kubernetes desde una imagen almacenada en {{site.data.keyword.registryshort_notm}}.
1. El usuario configurará el agente de servicio de {{site.data.keyword.la_full_notm}} para acceder a los registros de aplicación y de clúster.
1. El usuario configurará el agente de servicio de {{site.data.keyword.mon_full_notm}} para supervisar el estado y el rendimiento del clúster de {{site.data.keyword.containerlong_notm}} y también la app desplegada en el clúster.

## Requisitos previos
{: #prereq}

* [Instale {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli): script para instalar docker, kubectl, helm, la cli de ibmcloud y los plugins necesarios.
* [Configure la CLI de {{site.data.keyword.registrylong_notm}} y el espacio de nombres del registro](/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Otorgue permisos a un usuario para ver registros en LogDNA](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-work_iam#user_logdna)
* [Otorgue permisos a un usuario para ver métricas en Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-iam_work#user_sysdig)

## Creación de un clúster de Kubernetes
{: #create_cluster}

{{site.data.keyword.containershort_notm}} proporciona un entorno para desplegar apps de alta disponibilidad en contenedores de Docker que se ejecutan en clústeres de Kubernetes.

Sáltese esta sección si tiene un clúster **Estándar** existente que desea reutilizar con esta guía de aprendizaje.
{: tip}

1. Cree **un clúster nuevo** desde del [catálogo de {{site.data.keyword.Bluemix}}](https://{DomainName}/kubernetes/catalog/cluster/create) y elija el clúster **Estándar**.
   El reenvío de registros *no* está habilitado para el clúster **Gratuito**.
   {:tip}
1. Seleccione un grupo de recursos y una geografía.
1. Para su comodidad, utilice el nombre `mycluster` para poder seguir fácilmente esta guía de aprendizaje.
1. Seleccione una **Zona de trabajo** y seleccione el **Tipo de máquina** más pequeño con 2 **CPU** y 4 **GB de RAM**, ya que es suficiente para esta guía de aprendizaje.
1. Seleccione 1 **Nodo trabajador** y deje los valores predeterminados de todas las demás opciones. Pulse **Crear clúster**.
1. Compruebe el estado del **Clúster** y del **Nodo trabajador** y espere a que estén **listos**.

## Suministro de una instancia de {{site.data.keyword.la_short}}
{: #provision_logna_instance}

Las aplicaciones desplegadas en un clúster de {{site.data.keyword.containerlong_notm}} en {{site.data.keyword.Bluemix_notm}} probablemente generarán algún nivel de salida de diagnóstico, es decir, registros. Como desarrollador o como operador, es posible que desee acceder y analizar distintos tipos de registros, como registros de nodo trabajador, registros de pod, registros de apps o registros de red para resolver problemas y anticiparse a los mismos.

Mediante el servicio {{site.data.keyword.la_short}}, se pueden agregar registros procedentes de varios orígenes y retenerlos el tiempo que sea necesario. Esto permite analizar la "imagen general" cuando sea necesario y resolver situaciones más complejas.

Para suministrar un servicio {{site.data.keyword.la_short}},

1. Vaya a la página de [observabilidad](https://{DomainName}/observe/) y, en **Registro**, pulse **Crear instancia**.
1. Especifique un **Nombre de servicio** exclusivo.
1. Elija una región/ubicación y seleccione un grupo de recursos.
1. Seleccione **Búsqueda de registros de 7 días** como plan y pulse **Crear**.

El servicio proporciona un sistema de gestión de registros centralizado, donde los datos de registro se alojan en IBM Cloud.

## Despliegue y configuración de una app de Kubernetes para reenviar registros
{: #deploy_configure_kubernetes_app}

El [código para el registro de la app listo para ser ejecutado, se encuentra en el repositorio GitHub](https://github.com/IBM-Cloud/application-log-analysis). La aplicación está escriba mediante [Django](https://www.djangoproject.com/), una popular infraestructura de web de servidor Python. Clone o descargue el repositorio y luego despliegue la app en {{site.data.keyword.containershort_notm}} en {{site.data.keyword.Bluemix_notm}}.

### Despliegue de la aplicación Python

En un terminal:
1. Clone el repositorio GitHub:
   ```sh
   git clone https://github.com/IBM-Cloud/application-log-analysis
   ```
   {: pre}
1. Vaya al directorio de la aplicación
   ```sh
   cd application-log-analysis
   ```
   {: pre}
1. Cree una imagen de Docker con el [Dockerfile](https://github.com/IBM-Cloud/application-log-analysis/blob/master/Dockerfile) en {{site.data.keyword.registryshort_notm}}.
   - Localice el **Registro de contenedor** con `ibmcloud cr info`, como por ejemplo us.icr.io o uk.icr.io.
   - Cree un espacio de nombres para almacenar la imagen del contenedor.
      ```sh
      ibmcloud cr namespace-add app-log-analysis-namespace
      ```
      {: pre}
   - Sustituya `<CONTAINER_REGISTRY>` por el valor del registro de contenedor y utilice **app-log-analysis** como nombre de la imagen.
      ```sh
      ibmcloud cr build -t <CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest .
      ```
      {: pre}
   - Sustituya el valor de **image** en el archivo `app-log-analysis.yaml` por el código de imagen `<CONTAINER_REGISTRY>/app-log-analysis-namespace/app-log-analysis:latest`
1. Ejecute el mandato siguiente para recuperar la configuración de clúster y establezca la variable de entorno `KUBECONFIG`:
   ```sh
   $(ibmcloud ks cluster-config --export mycluster)
   ```
   {: pre}
1. Despliegue la app:
   ```sh
   kubectl apply -f app-log-analysis.yaml
   ```
   {: pre}
1. Para acceder a la aplicación, necesita la `IP pública` del nodo trabajador y un `NodePort`
   - Para la IP pública, ejecute el mandato siguiente:
      ```sh
      ibmcloud ks workers mycluster
      ```
      {: pre}
   - Para el NodePort que será de 5 dígitos (por ejemplo, 3xxxx), ejecute el mandato siguiente:
      ```sh
      kubectl describe service app-log-analysis-svc
      ```
      {: pre}
   Ahora puede acceder a la aplicación en `http://worker-ip-address:portnumber`

### Configuración del clúster para que envíe registros a la instancia de LogDNA

Para configurar el clúster de Kubernetes para enviar registros a la instancia de {{site.data.keyword.la_full_notm}}, debe instalar un pod *logdna-agent* en cada nodo del clúster. El agente LogDNA lee archivos de registro del pod donde está instalado y reenvía los datos de registro a la instancia de LogDNA.

1. Vaya a la página [Observabilidad](https://{DomainName}/observe/) y pulse **Registro**.
1. Pulse **Editar recursos de registro** junto al servicio que ha creado anteriormente y seleccione **Kubernetes**.
1. Copie y ejecute el primer mandato en un terminal en el que ha establecido la variable de entorno `KUBECONFIG` para crear un secreto de kubernetes con la clave de ingesta de LogDNA para la instancia de servicio.
1. Copie y ejecute el segundo mandato para desplegar un agente LogDNA en cada nodo de trabajador de su clúster de Kubernetes. El agente LogDNA recopila registros con la extensión **.log* y los archivos sin extensión que se almacenan en el directorio */var/log* de su pod. De forma predeterminada, se recopilan los registros de todos los espacios de nombres, incluyendo kube-system, y se reenvían automáticamente al servicio {{site.data.keyword.la_full_notm}}.
1. Después de configurar un origen de registro, inicie la interfaz de usuario de LogDNA pulsando **Ver LogDNA**. Puede tardar unos minutos en empezar a ver registros.

## Generación y acceso a registros de aplicación
{: generate_application_logs}

En esta sección, generará registros de aplicación y los revisará en LogDNA.

### Generación de registros de aplicación

La aplicación desplegada en los pasos anteriores le permite registrar un mensaje al nivel de registro elegido. Los niveles de registro disponibles son **critical** (crítico), **error**, **warn** (aviso), **info** y **debug** (depuración). La infraestructura de registro de la aplicación se configura de modo que solo permita pasar las entradas de registro de un nivel establecido o por encima del mismo. Inicialmente, el nivel de registro se establece en **warn**. Por lo tanto, un mensaje registrado a nivel de **info** con el valor de servidor **warn** no se mostraría en la salida del diagnóstico.

Examine el código del archivo [**views.py**](https://github.com/IBM-Cloud/application-log-analysis/blob/master/app/views.py). El código contiene sentencias **print**, así como llamadas a funciones de **logger**. Los mensajes impresos se escriben en la secuencia **stdout** (salida normal, consola de aplicación / terminal), los mensajes de registro aparecen en la secuencia **stderr** (registro de errores).

1. Visite la app web que encontrará en `http://worker-ip-address:portnumber`.
1. Genere varias entradas de registro enviando mensajes a distintos niveles. La interfaz de usuario también permite cambiar el valor del registrador para el nivel de registro del servidor. Cambie a mitad el nivel de registro del servidor para que el ejemplo sea más interesante. Por ejemplo, puede registrar un "500 internal server error" como **error** o "This is my first log entry" como **info**.

### Acceso a registros de aplicación

Puede acceder al registro específico de la aplicación en la interfaz de usuario de LogDNA mediante los filtros.

1. En la barra superior, pulse **Todas las apps**.
1. En contenedores, marque **app-log-analysis**. Se muestra una nueva vista no guardada con registros de aplicación de todos los niveles.
1. Para ver los registros de niveles de registro específicos, pulse **Todos los niveles** y seleccione varios niveles, como error, información, aviso, etc.,

## Búsqueda y filtro de registros
{: #search_filter_logs}

La interfaz de usuario de {{site.data.keyword.la_short}}, de forma predeterminada, muestra todas las entradas de registro disponibles (Todo). Las entradas más recientes se muestran en la parte inferior con una renovación automática.
En esta sección, modificará qué y cuánto se visualiza y esto se guardará como una **vista** para su uso futuro.

### Búsqueda de registros

1. En el recuadro de entrada **Buscar** que se encuentra en la parte inferior de la página de la IU de LogDNA,
   - puede buscar líneas que contengan un texto específico, como **"This is my first log entry"** o **500 internal server error**.
   - o un nivel de registro específico especificando `level:info`, donde leves es un campo que acepta el valor de serie.

   Para ver más campos de búsqueda y para obtener ayuda, pulse el icono de ayuda de sintaxis situado junto al recuadro de entrada de la búsqueda.
   {:tip}
1. Para saltar a un intervalo de tiempo específico, especifique **Hace 5 minutos** en el recuadro de entrada **Saltar a intervalo de tiempo**. Pulse el icono que hay junto al recuadro de entrada para buscar otros formatos de tiempo en el periodo de retención.
1. Para resaltar los términos, pulse el icono **Conmutar herramientas del visor**.
1. Especifique **error** como término de resaltado en el primer recuadro de entrada, **container** como término de resaltado en el segundo recuadro de entrada y compruebe las líneas resaltadas con los términos.
1. Pulse el icono **Conmutar línea de tiempo** para ver las líneas con registros a una hora del día específica.

### Filtrado de registros

Puede filtrar registros por etiquetas, orígenes, apps o niveles.

1. En la barra superior, pulse **Todas las etiquetas** y marque el recuadro de selección **k8s** para ver los registros relacionados con Kubernetes.
1. Pulse **Todos los orígenes** y seleccione el nombre del host (nodo trabajador) cuyos registros le interesan. Funciona bien si tiene varios nodos trabajadores en el clúster.
1. Para comprobar los registros de contenedores o de archivos, pulse **Todas las apps** y seleccione los recuadros de selección cuyos registros desea ver.

### Creación de una vista

Las vistas son atajos a un conjunto específico de filtros y consultas de búsqueda.

Tan pronto como busque o filtre los registros, debería ver **Vista no guardada** en la barra superior. Para guardar esto como una vista:
1. Pulse **Todas las apps** y marque el recuadro de selección situado junto a **app-log-analysis**
1. Pulse **Vista no guardada** > pulse **Guardar como nueva vista/alerta** y llame a la vista **app-log-analysis-view**. Deje la **Categoría** en blanco.
1. Pulse **Guardar vista** y la vista nueva debería aparecer en el panel de la izquierda, con los registros de la app.

### Visualización de registros con gráficos y desgloses

En esta sección, creará un panel y luego añadirá un gráfico con un desglose para visualizar los datos de nivel de app. Un panel es una colección de gráficos y desgloses.

1. En el panel de la izquierda, pulse el icono **panel** (sobre el icono de valores) > pulse **NUEVO PANEL**.
1. Pulse **Editar** en la barra superior y llámelo **app-log-analysis-board**. Pulse **Guardar**.
1. Pulse **Añadir gráfico**:
   - Especifique **app** como campo en el primer recuadro de entrada y pulse Intro.
   - Seleccione **app-log-analysis** como valor de campo.
   - Pulse **Añadir gráfico**.
1. Seleccione **Recuentos** como métrica para ver el número de líneas de cada intervalo durante las últimas 24 horas.
1. Para agregar un desglose, pulse la flecha que aparece debajo del gráfico:
   - Elija **Histograma** como tipo de desglose.
   - Elija **nivel** como tipo de campo.
   - Pulse **Añadir desglose** para ver un desglose con todos los niveles que ha registrado para la app.

## Adición de {{site.data.keyword.mon_full_notm}} y supervisión del clúster
{: #monitor_cluster_sysdig}

A continuación, añadirá {{site.data.keyword.mon_full_notm}} a la aplicación. El servicio comprueba de forma regular la disponibilidad y el tiempo de respuesta de la app.

1. Vaya a la página de [observabilidad](https://{DomainName}/observe/) y, en **Supervisión**, pulse **Crear instancia**.
1. Especifique un **Nombre de servicio** exclusivo.
1. Elija una región/ubicación y seleccione un grupo de recursos.
1. Seleccione **Por niveles** como plan y pulse **Crear**.
1. Pulse **Editar recursos de registro** junto al servicio que ha creado anteriormente y seleccione **Kubernetes**.
1. Copie y ejecute el mandato en **Instalar el agente de Sysdig en el clúster** en un terminal en el que haya establecido la variable de entorno `KUBECONFIG` para desplegar el agente de Sysdig en el clúster. Espere a que
finalice el despliegue.

### Configuración de {{site.data.keyword.mon_short}}

Para configurar Sysdig para supervisar el estado y el rendimiento de su clúster:
1. Pulse **Ver Sysdig**; debería ver la interfaz de usuario del supervisor sysdig. En la página de bienvenida, pulse **Siguiente**.
1. Elija **Kubernetes** como método de instalación en el entorno de configuración.
1. Pulse **Ir al paso siguiente** junto al mensaje que indica que el agente se ha configurado correctamente y pulse **Empecemos** en la página siguiente.
1. Pulse **Siguiente** y, a continuación, **Completar la incorporación** para ver el separador `Explorar` de la interfaz de usuario de Sysdig.

### Supervisión del clúster

Para comprobar el estado y el rendimiento del clúster y de la app:
1. De nuevo en la aplicación que se ejecuta en `http://worker-ip-address:portnumber`, genere varias entradas de registro.
1. Amplíe **mycluster** en el panel de la izquierda > amplíe el espacio de nombres **default** > pulse **app-log-analysis-deployment** para ver el recuento de solicitudes, el tiempo de respuesta etc. en el asistente del supervisor de Sysdig.
1. Para comprobar los códigos de respuesta de solicitud de HTTP, pulse la flecha situada junto a **Estado del pod de Kubernetes** en la barra superior y seleccione **HTTP** en **Aplicaciones**. Cambie el intervalo por **10 M** en la barra inferior de la interfaz de usuario de Sysdig.
1. Para supervisar la latencia de la aplicación,
   - En el separador Explorar, seleccione **Despliegues y pods**.
   - Pulse la flecha situada junto a `HTTP` y seleccione Métricas > Red.
   - Seleccione **net.http.request.time**.
   - Tiempo de selección: **Suma** y Grupo: **Promedio**.
   - Pulse **Más opciones** y luego el icono **Topología**.
   - Pulse **Terminado** y realice una doble pulsación en el recuadro para ampliar la vista.
1. Para supervisar el espacio de nombres de Kubernetes en el que se ejecuta la aplicación,
   - En el separador Explorar, seleccione **Despliegues y pods**.
   - Pulse la flecha situada junto a `net.http.request.time`.
   - Seleccione Paneles de control predeterminados > Kubernetes.
   - Seleccione Estado de Kubernetes > Visión general del estado de Kubernetes.

### Creación de un panel de control personalizado

Junto con los paneles de control predefinidos, puede crear su propio panel de control personalizado para visualizar las vistas y las métricas más útiles y relevantes para los contenedores que ejecutan la app en una única ubicación. Cada panel de control incluye una serie de paneles que están configurados para mostrar datos específicos en diferentes formatos.

Para crear un panel de control:
1. Pulse **Paneles de control** en el panel de la izquierda > pulse **Añadir panel de control**.
1. Pulse **Panel de control en blanco** > llame al panel de control **Visión general de solicitud de contenedor** > pulse **Crear panel de control**.
1. Seleccione **Lista superior** como nuevo panel y llámelo **Tiempo de solicitud por contenedor**
   - En **Métricas**, escriba **net.http.request.time**.
   - Ámbito: pulse **Modificar ámbito del panel de control** > seleccione **container.image** > seleccione **es** > seleccione _la imagen de la aplicación_
   - Segmente por **container.id** y debería ver el tiempo de solicitud de red de cada contenedor.
   - Pulse **Guardar**.
1. Para añadir un panel nuevo, pulse en el icono **más** y seleccione **Número** como tipo de panel
   - En **Métricas**, escriba **net.http.request.count** > Cambie la agregación de tiempo de **Promedio** a **Suma**.
   - Ámbito: pulse **Modificar ámbito del panel de control** > seleccione **container.image** > seleccione **es** > seleccione _la imagen de la aplicación_
   - Compare con hace **1 hora** y debería ver el recuento de solicitudes de red de cada contenedor.
   - Pulse **Guardar**.
1. Para editar el ámbito de este panel de control,
   - Pulse **Editar ámbito** en el panel del título.
   - Seleccione o escriba **Kubernetes.cluster.name** en el menú desplegable
   - Deje el nombre de visualización vacío y seleccione **es**.
   - Seleccione **mycluster** como valor y pulse **Guardar**.

## Eliminación de recursos
{: #remove_resource}

- Elimine las instancias de LogDNA y de Sysdig de la página [Observabilidad](https://{DomainName}/observe).
- Suprima el clúster, incluido el nodo trabajador, la app y los contenedores. Esta acción no se puede deshacer.
   ```sh
   ibmcloud ks cluster-rm mycluster -f
   ```
   {:pre}

## Ampliación de la guía de aprendizaje
{: #expand_tutorial}

- Utilice el [servicio {{site.data.keyword.at_full}}](/docs/services/Activity-Tracker-with-LogDNA?topic=logdnaat-getting-started#getting-started) para realizar un seguimiento del modo en que las aplicaciones interactúan con los servicios de IBM Cloud.
- [Añada alertas](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-alerts#alerts) a la vista.
- [Exporte registros](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-export#export) a un archivo local.

## Contenido relacionado
{:related}
- [Restablecimiento de la clave de ingestión que utiliza un clúster de Kubernetes](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-kube_reset#kube_reset)
- [Archivado de registros en IBM Cloud Object Storage](/docs/services/Log-Analysis-with-LogDNA?topic=LogDNA-archiving#archiving)
- [Configuración de alertas en Sysdig](https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/205324292/Alerts)
- [Cómo trabajar con canales de notificación en la interfaz de usuario de Sysdig](/docs/services/Monitoring-with-Sysdig?topic=Sysdig-notifications#notifications)
