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

# Clústeres de Kubernetes de varias regiones resistentes y seguros con Cloud Internet Services
{: #multi-region-k8s-cis}

Los usuarios tienen menos probabilidades de sufrir un tiempo de inactividad cuando una aplicación está diseñada pensando en su resistencia. Si implementa una solución con {{site.data.keyword.containershort_notm}}, se beneficia de las prestaciones incorporadas, como el equilibrio de carga y el aislamiento y el aumento de la resistencia frente a posibles anomalías con hosts, redes o apps. Mediante la creación de varios clústeres, si se produce una interrupción en un clúster, los usuarios pueden acceder a una app que también está desplegada en otro clúster. Con varios clústeres en ubicaciones diferentes, los usuarios también pueden acceder al clúster más cercano y reducir la latencia de red. Para conseguir una mayor resistencia, tiene la opción de seleccionar también los clústeres de varias zonas, lo que significa que los nodos se despliegan en varias zonas dentro de una ubicación.

En esta guía de aprendizaje se muestra cómo Cloud Internet Services (CIS), una plataforma uniforme para configurar y gestionar el sistema de nombres de dominio (DNS), el equilibrador de carga global (GLB), el cortafuegos de aplicación web (WAF) y para proteger frente a una denegación de servicio distribuido (DDoS) para aplicaciones de internet, se puede integrar con clústeres de Kubernetes para dar soporte a este escenario y para ofrecer una solución segura y resistente en varias ubicaciones.

## Objetivos
{: #objectives}

* Despliegue de una aplicación en varios clústeres de Kubernetes en una ubicación distinta.
* Distribución del tráfico entre varios clústeres con un equilibrador de carga global.
* Direccionamiento de los usuarios al clúster más cercano.
* Protección de la aplicación frente a amenazas de seguridad.
* Aumento del rendimiento de la aplicación con memoria caché.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

<p style="text-align: center;">
  ![Arquitectura](images/solution32-multi-region-k8s-cis/Architecture.png)
</p>

1. El desarrollador crea imágenes de Docker para la aplicación.
2. Las imágenes se envían a {{site.data.keyword.registryshort_notm}} en Dallas y Londres.
3. La aplicación se despliega en clústeres de Kubernetes en ambas ubicaciones.
4. Los usuarios finales acceden a la aplicación.
5. Se configura Cloud Internet Services para que intercepte las solicitudes a la aplicación y para que distribuya la carga entre los clústeres. También se habilita la protección DDoS y el cortafuegos de aplicaciones web para proteger las aplicaciones frente a amenazas comunes. Los activos como imágenes y archivos CSS se pueden colocar en la memoria caché.

## Antes de empezar
{: #prereqs}

* Cloud Internet Services necesita que tenga un dominio personalizado para poder configurar el DNS para que este dominio apunte a los servidores de nombres de Cloud Internet Services.
* [Instale Git](https://git-scm.com/).
* [Instale la CLI de {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli).
* [IBM Cloud Developer Tools](https://github.com/IBM-Cloud/ibm-cloud-developer-tools): scripts para instalar docker, kubectl, helm, cli de ibmcloud y los plugins necesarios.
* [Configure la CLI de {{site.data.keyword.registrylong_notm}} y el espacio de nombres del registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace).
* [Comprenda los conceptos básicos de Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/).

## Despliegue de una aplicación en una ubicación

En esta guía de aprendizaje se despliega una aplicación de Kubernetes en clústeres en varias ubicaciones. Comenzará con una ubicación, Dallas, y luego repetirá estos pasos para Londres.

### Creación de un clúster de Kubernetes
{: #create_cluster}

Para crear un clúster:
1. Seleccione **{{site.data.keyword.containershort_notm}}** en el [catálogo de {{site.data.keyword.cloud_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster/create).
1. Establezca **Ubicación** en **Dallas**.
1. Seleccione el clúster **Estándar**.
1. Seleccione una o varias zonas como **Ubicación**. La creación de un clúster multizona aumenta la resistencia de la aplicación. Los usuarios tienen muchas menos probabilidades de experimentar tiempos de inactividad si la app está distribuida entre varias zonas. Encontrará más información sobre los clústeres multizona [aquí](https://{DomainName}/docs/containers?topic=containers-plan_clusters#ha_clusters).
1. Establezca **Tipo de máquina** en el más pequeño disponible: **2 CPU** y **4 GB de RAM** es suficiente para esta guía de aprendizaje.
1. Utilice **2** nodos trabajadores.
1. Establezca el **Nombre de clúster** en **my-us-cluster**. Utilice el patrón de denominación *`my-<location>-cluster`* para seguir esta guía de aprendizaje.

Mientras el clúster se está preparando, preparará la aplicación.

### Creación de un espacio de nombres en {{site.data.keyword.registryshort_notm}}

1. Defina Dallas como destino de la CLI de {{site.data.keyword.Bluemix_notm}}.
   ```bash
   ibmcloud target -r us-south
   ```
   {: pre}
2. Cree un espacio de nombres para la aplicación.
   ```bash
   ibmcloud cr namespace-add <your_namespace>
   ```
   {: pre}

También puede reutilizar un espacio de nombres existente si tiene uno en la ubicación. Puede ver una lista de los espacios de nombres existentes con el mandato `ibmcloud cr namespaces`.
{: tip}

### Creación de la aplicación

En este paso se crea la aplicación en una imagen de Docker. Puede saltarse este paso si está configurando el segundo clúster. Se trata de una sencilla app HelloWorld.

1. Clone el código fuente de la [app Hello world](https://github.com/IBM/container-service-getting-started-wt){:new_windows} en el directorio inicial de su usuario. El repositorio contiene distintas versiones de una app similar en carpetas que empiezan por Lab.
   ```bash
   git clone https://github.com/IBM/container-service-getting-started-wt.git
   ```
   {: pre}
1. Vaya al directorio `Lab 1`.
   ```bash
   cd 'container-service-getting-started-wt/Lab 1'
   ```
   {: pre}
1. Cree una imagen de Docker que incluya los archivos de la app del directorio `Lab 1`.
   ```bash
   docker build --tag multi-region-hello-world:1 .
   ```
   {: pre}

### Preparación de la imagen que se va a enviar al registro específico de la ubicación

Etiquete la imagen con el registro de destino:

   ```bash
   docker tag multi-region-hello-world:1 us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### Envío de la imagen al registro específico de la ubicación

1. Asegúrese de que el motor de Docker local puede enviar al registro en Dallas.
   ```bash
   ibmcloud cr login
   ```
   {: pre}
2. Envíe la imagen por push.
   ```bash
   docker push us.icr.io/<your_us-south_namespace>/hello-world:1
   ```
   {: pre}

### Despliegue de la aplicación en el clúster de Kubernetes

En esta etapa, el clúster debería estar listo. Puede comprobar su estado en la consola de [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/clusters).

1. Recupere la configuración del clúster:
   ```bash
   ibmcloud cs cluster-config <us-cluster-name>
   ```
   {: pre}
1. Copie y pegue la salida para establecer la variable de entorno KUBECONFIG. `kubectl` utiliza la variable.
1. Ejecute la aplicación en el clúster con dos réplicas:
   ```bash
   kubectl run hello-world-deployment --image=us.icr.io/<your_US-South_namespace>/hello-world:1 --replicas=2
   ```
   {: pre}
   Salida de ejemplo: `deployment "hello-world-deployment" created`.
1. Consiga que se pueda acceder a la aplicación dentro del clúster
   ```bash
   kubectl expose deployment/hello-world-deployment --type=ClusterIP --port=80 --name=hello-world-service --target-port=8080
   ```
   {: pre}
   Devuelve un mensaje parecido al siguiente: `service "hello-world-service" exposed`.

### Obtención del nombre de dominio y de la dirección IP asignados al clúster
{: #CSALB_IP_subdomain}

Cuando se crea un clúster de Kubernetes, se le asigna un subdominio Ingress (por ejemplo, *my-us-cluster.us-south.containers.appdomain.cloud*) y una dirección IP de equilibrador de carga de aplicación pública.

1. Recupere el subdominio Ingress del clúster:
   ```bash
   ibmcloud cs cluster-get <us-cluster-name>
   ```
   {: pre}
   Busque el valor de `Subdominio de Ingress`.
1. Anote esta información para un paso posterior.

En esta guía de aprendizaje se utiliza el subdominio Ingress para configurar el equilibrador de carga global. También puede cambiar el subdominio por la dirección IP del equilibrador de carga de aplicación pública (`ibmcloud cs albs -cluster <us-cluster-name>`). Se da soporte a ambas opciones.
{: tip}

## Y luego a otra ubicación

Repita los pasos anteriores en Londres sustituyendo:
* el nombre de ubicación **Dallas** por **Londres**;
* el alias de ubicación **us-sur** por **eu-gb**;
* el registro *us.icr.io* por **uk.icr.io**;
* y el nombre de clúster *my-us-cluster* por **my-uk-cluster**.

## Configuración del equilibrio de carga en varias ubicaciones

Ahora la aplicación se está ejecutando en dos clústeres, pero falta un componente para que los usuarios acceden a los clústeres de forma transparente desde un único punto de entrada.

En esta sección, configurará Cloud Internet Services (CIS) para distribuir la carga entre los dos clústeres. CIS es un servicio que se adquiere una vez y que ofrece un equilibrador de carga global (GLB), memoria caché, cortafuegos de aplicación web (WAF) y regla de página para proteger las aplicaciones mientras se garantiza la fiabilidad y el rendimiento de las aplicaciones en la nube.

Para configurar un equilibrador de carga global, necesitará:
* hacer que un dominio personalizado apunte a servidores de nombres de CIS,
* recuperar las direcciones IP o los nombres de subdominio de los clústeres de Kubernetes,
* configurar comprobaciones de estado para validar la disponibilidad de la aplicación,
* y definir agrupaciones de origen que apunten a los clústeres.

### Registro de un dominio personalizado con Cloud Internet Services
{: #create_cis_instance}

El primer paso consiste en crear una instancia de CIS y hacer que el dominio personalizado apunte a los servidores de nombres de CIS.

1. Si no tiene un dominio, puede adquirir uno de un registrador como [godaddy.com](http://godaddy.com).
2. Vaya a [Servicios de Internet](https://{DomainName}/catalog/services/internet-services) en el catálogo de {{site.data.keyword.Bluemix_notm}}.
3. Establezca el nombre de servicio y pulse **Crear** para crear una instancia del servicio.
4. Cuando se haya suministrado la instancia de servicio, defina el nombre de dominio y pulse **Añadir dominio**.
5. Cuando se hayan asignado los servidores de nombres, configure el registrador o el proveedor de nombres de dominio para utilizar los servidores de nombres que se muestran.
6. Después de configurar el registrador o el proveedor de DNS, los cambios pueden tardar hasta 24 en entrar en vigor.

   Cuando el estado del dominio en la página Visión general pase de *Pendiente* a *Activo*, puede utilizar el mandato `dig <your_domain_name> ns` para verificar que los servidores de nombres han entrado en vigor.
   {:tip}

### Configuración de comprobaciones de estado para el equilibrador de carga global

Una comprobación de estado ayuda a obtener información sobre la disponibilidad de las agrupaciones para que el tráfico se pueda direccionar a las que están en buen estado. Estas comprobaciones envían solicitudes HTTP/HTTPS periódicamente y supervisan las respuestas.

1. En el panel de control de Cloud Internet Services, vaya a **Fiabilidad** > **Equilibrador de carga global** y, en la parte inferior de la página, pulse **Crear comprobación de estado**.
1. Establezca **Vía de acceso** en **/**
1. Establezca **Tipo de supervisor** en **HTTP**.
1. Pulse **Suministrar 1 instancia**.

   Cuando cree sus propias aplicaciones, puede definir un punto final de estado dedicado, como por ejemplo */heathz*, en el que puede informar del estado de la aplicación.
   {:tip}

### Definición de agrupaciones de origen

Una agrupación es un grupo de servidores de origen al que se direcciona el tráfico de forma inteligente al adjuntarse a un GLB. Con clústeres en el Reino Unido y en Estados Unidos, puede definir agrupaciones basadas en ubicación y configurar CIS de modo que redirija a los usuarios a los clústeres más cercanos en función de la ubicación geográfica de las solicitudes de los clientes.

#### Una agrupación para el clúster en Londres
1. Pulse **Crear agrupación**.
2. Como **Nombre**, defina **RU**
3. Establezca la **Comprobación de estado** en la que se ha creado en la sección anterior
4. Establezca la **Región de comprobación de estado** en **Europa occidental**
5. Establezca **Nombre de origen** en **uk-cluster**
6. Establezca la **Dirección de origen** en el subdominio Ingress del clúster en Londres, por ejemplo *my_uk_cluster.eu-gb.containers.appdomain.cloud*
7. Pulse **Suministrar 1 instancia**.

#### Una agrupación para el clúster en Dallas
1. Pulse **Crear agrupación**.
2. Como **Nombre**, defina **EE.UU.**
3. Establezca la **Comprobación de estado** en la que se ha creado en la sección anterior
4. Establezca la **Región de comprobación de estado** en **América del Norte occidental**
5. Establezca **Nombre de origen** en **us-cluster**
6. Establezca la **Dirección de origen** en el subdominio Ingress del clúster en Dallas, por ejemplo *my_us_cluster.us-south.containers.appdomain.cloud*
7. Pulse **Suministrar 1 instancia**.

#### Y una agrupación con ambos clústeres
1. Pulse **Crear agrupación**.
1. Como **Nombre**, defina **Todo**
1. Establezca la **Comprobación de estado** en la que se ha creado en la sección anterior
1. Establezca la **Región de comprobación de estado** en **América del Norte oriental**
1. Añada dos orígenes:
   1. uno con el **Nombre de origen** establecido en **us-cluster** y la **Dirección de origen** establecida en el subdominio Ingress del clúster en Dallas
   2. uno con el **Nombre de origen** establecido en **uk-cluster** y la **Dirección de origen** establecida en el subdominio Ingress del clúster en Londres
2. Pulse **Suministrar 1 instancia**.

### Creación del equilibrador de carga global

Con las agrupaciones de origen definidas, puede completar la configuración del equilibrador de carga.

1. Pulse **Crear equilibrador de carga**.
1. Especifique un nombre bajo **Nombre de host del equilibrador** para el equilibrador de carga global. Este nombre también formará parte de su URL de aplicación universal (`http://<glb_name>.<your_domain_name>`), independientemente de la ubicación.
1. En **Agrupaciones de origen predeterminadas**, pulse **Añadir agrupación** y añada la agrupación denominada **Todo**.
1. Amplíe la sección **Configurar geo rutas (opcional)**:
   1. Pulse **Añadir ruta**, seleccione **Europa occidental** y pulse **Añadir**.
   1. Pulse **Añadir agrupación** para seleccionar la agrupación **RU**.
   1. Configure rutas adicionales como se muestra en la tabla siguiente.
   1. Pulse **Suministrar 1 instancia**.

| Región               | Agrupación de origen |
| :---------------:    | :---------: |
|Europa occidental        |     RU      |
|Europa oriental        |     RU      |
|Asia noreste        |     RU      |
|Asia sudeste        |     RU      |
|América del Norte occidental |     EE.UU.      |
|América del Norte oriental |     EE.UU.      |

Con esta configuración, a los usuarios de Europa y Asia se les redirigirá al clúster de Londres, y a los usuarios de Estados Unidos, al clúster de Dallas. Cuando una solicitud no coincida con ninguna de las rutas definidas, se le redirigirá a las **Agrupaciones de origen predeterminadas**.

### Creación de un recurso Ingress para clústeres de Kubernetes por ubicación

El equilibrador de carga global está ahora preparado para atender las solicitudes. Todos los controles de estado deben estar en verde. Sin embargo, hay un último paso de configuración necesario en los clústeres de Kubernetes para que respondan correctamente a las solicitudes procedentes del equilibrador de carga global: debe definir un recurso Ingress para que gestione las solicitudes procedentes del dominio del equilibrador de carga global.

1. Cree un archivo de recursos Ingress denominado **glb-ingress.yaml**
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: glb-ingress
   spec:
    rules:
      - host: <glb_name>.<your_domain_name>
        http:
          paths:
          - path: /
            backend:
              serviceName: hello-world-service
              servicePort: 80
   ```
    {: pre}
    Sustituya `<glb_name>.<your_domain_name>` por el URL que ha definido en la sección anterior.
1. Despliegue este recurso en clústeres de Londres y Dallas, después de establecer la variable KUBECONFIG para los clústeres de las respectivas ubicaciones:
   ```bash
   kubectl create -f glb-ingress.yaml
   ```
   {: pre}
   Muestra el mensaje `ingress.extension "glb-ingress" created`.

En esta etapa, ha configurado correctamente un equilibrador de carga global con clústeres de Kubernetes en varias ubicaciones. Puede acceder al URL del equilibrador de carga global `http://<glb_name>.<your_domain_name>` para ver su aplicación. En función de su ubicación, se le redirigiría al clúster más cercano o a un clúster de la agrupación predeterminada si CIS no es capaz de correlacionar la dirección IP con una ubicación específica.

## Protección de la aplicación
{: #secure_via_CIS}

### Activación del cortafuegos de aplicación web

El cortafuegos de aplicación web (WAF) protege la aplicación web ante ataques ISO de capa 7. Generalmente, se combina con conjuntos de reglas agrupadas, que tienen como objetivo proteger contra las vulnerabilidades en la aplicación filtrando el tráfico malicioso.

1. En el panel de control de Cloud Internet Services, vaya a **Seguridad** y al separador **Gestionar**.
1. En la sección **Cortafuegos de aplicación web**, asegúrese de que WAF está habilitado.
1. Pulse **Ver conjunto de reglas OWASP**. En esta página puede revisar el **Conjunto de reglas de núcleo de OWASP** y habilitar o inhabilitar las reglas individualmente. Cuando una regla está habilitada, si una solicitud de entrada activa la regla, la puntuación de amenaza global aumenta. El valor **Sensibilidad** decide si se activa una **Acción** para la solicitud.
   1. Deje los conjuntos de reglas OWASP predeterminados tal como están.
   1. Establezca **Sensibilidad** en `Baja`.
   1. Establezca **Acción** en `Simular` para registrar todos los sucesos.
1. Pulse **Volver a seguridad**.
1. Pulse **Ver conjunto de reglas CIS**. Esta página muestra reglas adicionales creadas sobre las pilas de tecnología comunes para alojar sitios web.

### Aumento del rendimiento y protección frente a ataques de denegación de servicio 
{: #proxy_setting}

Un ataque de denegación de servicio distribuido ([DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack)) es un intento malintencionado de interrumpir el tráfico normal de un servidor, servicio o red, desbordando el destino o su infraestructura circundante con una gran cantidad de tráfico de Internet. CIS está equipado para proteger su dominio de DDoS.

1. En el panel de control de CIS, seleccione **Fiabilidad** > **Equilibrador de carga global**.
1. Localice el GLB que ha creado en la tabla **Equilibradores de carga**.
1. Habilite las características de seguridad y rendimiento en la columna **Proxy**:

   ![Activación del proxy CIS](images/solution32-multi-region-k8s-cis/cis-proxy.png)

**Ahora su GLB está protegido**. Una ventaja inmediata es que la dirección IP de origen de los clústeres quedará oculta para los clientes. Si CIS detecta una amenaza en una futura solicitud, el usuario puede ver una pantalla como esta antes de ser redirigido a la aplicación:

   ![verificación - protección de DDoS](images/solution32-multi-region-k8s-cis/cis-DDoS.png)

Además, ahora puede controlar el contenido que se almacena en la memoria caché de CIS y el tiempo que permanece almacenado en memoria caché. Vaya a **Rendimiento** > **Almacenamiento en memoria caché** para definir el nivel de almacenamiento en memoria caché global y la caducidad del navegador. Puede personalizar las reglas de seguridad global y de almacenamiento en memoria caché con **Reglas de página**. Las reglas de página permiten una configuración más precisa mediante vías de acceso de dominio específicas. Por ejemplo, con reglas de página puede decidir almacenar en memoria caché todo el contenido bajo **/assets** durante **3 días**:

   ![reglas de página](images/solution32-multi-region-k8s-cis/cis-pagerules.png)

## Eliminación de recursos
{:removeresources}

### Eliminación de los recursos de clúster de Kubernetes
1. Elimine Ingress.
1. Elimine el servicio.
1. Elimine el despliegue.
1. Suprima los clústeres si los ha creado específicamente para esta guía de aprendizaje.

### Eliminación de los recursos de CIS
1. Elimine el GLB.
1. Elimine las agrupaciones de origen.
1. Elimine las comprobaciones de estado.

## Contenido relacionado
{:related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Gestión de IBM CIS para una seguridad óptima](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#best-practice-2-configure-your-security-level-selectively)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
* [{{site.data.keyword.registrylong_notm}} Básico](https://{DomainName}/docs/services/Registry?topic=registry-registry_overview#registry_planning)
* [Despliegue de apps de una sola instancia en clústeres de Kubernetes](https://{DomainName}/docs/containers?topic=containers-cs_apps_tutorial#cs_apps_tutorial_lesson1)
* [Práctica recomendada para proteger el tráfico y la aplicación en internet mediante CIS](https://{DomainName}/docs/infrastructure/cis?topic=cis-manage-your-ibm-cis-for-optimal-security#manage-your-ibm-cis-for-optimal-security)
* [Mejora de la disponibilidad de la app con clústeres de varias zonas](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
