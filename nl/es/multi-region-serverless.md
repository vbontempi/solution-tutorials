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

# Despliegue de apps sin servidor entre varias regiones
{: #multi-region-serverless}

En esta guía de aprendizaje se muestra cómo configurar IBM Cloud Internet Services y {{site.data.keyword.openwhisk_short}} para desplegar apps sin servidor en varias regiones.

Las plataformas de cálculo sin servidor ofrecen a los programadores un método rápido de crear API sin servidores. {{site.data.keyword.openwhisk}} da soporte a la generación automática de API REST para acciones, convirtiendo acciones en puntos finales HTTP, y a la posibilidad de habilitar la autenticación de API segura. Esta característica resulta útil no solo para exponer las API a consumidores externos, sino también para crear aplicaciones de microservicios.

{{site.data.keyword.openwhisk_short}} está disponible en varias ubicaciones de {{site.data.keyword.cloud_notm}}. Para aumentar la resistencia y reducir la latencia de red, las aplicaciones pueden desplegar su programa de fondo en varias ubicaciones. A continuación, con IBM Cloud Internet Services (CIS), los desarrolladores pueden exponer un único punto de entrada encargado de distribuir el tráfico al programa de fondo en buen estado más cercano.

## Objetivos
{: #objectives}

* Despliegue de acciones de {{site.data.keyword.openwhisk_short}}.
* Exposición de acciones a través de {{site.data.keyword.APIM}} con un dominio personalizado.
* Distribución del tráfico entre varias ubicaciones con Cloud Internet Services.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

En la guía de aprendizaje se utiliza una aplicación web pública con un programa de fondo implementado con {{site.data.keyword.openwhisk_short}}. Para reducir la latencia de la red y evitar interrupciones, la aplicación se despliega en varias ubicaciones. Se configuran dos ubicaciones en la guía de aprendizaje.

<p style="text-align: center;">

  ![Arquitectura](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. Los usuarios acceden a la aplicación. La solicitud pasa por Internet Services.
2. Internet Services redirige a los usuarios al programa de fondo de API en buen estado más cercano.
3. {{site.data.keyword.cloudcerts_short}} proporciona la API con su certificado SSL. El tráfico se cifra de extremo a extremo.
4. La API se implementa con {{site.data.keyword.openwhisk_short}}.

## Antes de empezar
{: #prereqs}

1. Cloud Internet Services necesita que tenga un dominio personalizado para poder configurar el DNS para que este dominio apunte a los servidores de nombres de Cloud Internet Services. Si no tiene un dominio, puede adquirir uno de un registrador como [godaddy.com](http://godaddy.com).
1. Instale todas las herramientas de línea de mandatos (CLI) necesarias [siguiendo estos pasos](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).

## Configuración de un dominio personalizado

El primer paso consiste en crear una instancia de IBM Cloud Internet Services (CIS) y hacer que el dominio personalizado apunte a los servidores de nombres de CIS.

1. Vaya a [Servicios de Internet](https://{DomainName}/catalog/services/internet-services) en el catálogo de {{site.data.keyword.Bluemix_notm}}.
1. Establezca el nombre de servicio y pulse **Crear** para crear una instancia del servicio. Puede utilizar cualquier plan de precios para esta guía de aprendizaje.
1. Cuando se haya suministrado la instancia de servicio, establezca el nombre de dominio pulsando **Empecemos** y pulse **Añadir dominio**.
1. Pulse **Paso siguiente**. Cuando se hayan asignado los servidores de nombres, configure el registrador o el proveedor de nombres de dominio para utilizar los servidores de nombres que se muestran.
1. Después de configurar el registrador o el proveedor de DNS, los cambios pueden tardar hasta 24 en entrar en vigor.

   Cuando el estado del dominio en la página Visión general pase de *Pendiente* a *Activo*, puede utilizar el mandato `dig <your_domain_name> ns` para verificar que los servidores de nombres han entrado en vigor.
   {:tip}

### Obtención de un certificado para el dominio personalizado

Para exponer las acciones de {{site.data.keyword.openwhisk_short}} a través de un dominio personalizado se necesita una conexión HTTPS segura. Debe obtener un certificado SSL para el dominio y el subdominio que tiene previsto utilizar con el servidor de fondo sin servidor. Suponiendo que tiene un dominio como *mydomain.com*, las acciones se pueden alojar en *api.mydomain.com*. Se tendrá que emitir el certificado para *api.mydomain.com*.

Puede conseguir certificados SSL gratuitos en [Cifremos](https://letsencrypt.org/). Durante el proceso, es posible que tenga que configurar un registro DNS de tipo TXT en la interfaz DNS de Cloud Internet Services para probar que es el propietario del dominio.
{:tip}

Cuando haya obtenido el certificado SSL y la clave privada para el dominio, asegúrese de convertirlos al formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Para convertir un certificado en formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. Para convertir una clave privada al formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Importación del certificado en un repositorio central

1. Cree una instancia de [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) en una ubicación soportada.
1. En el panel de control del servicio, utilice **Importar certificado**:
   * Establezca como **Nombre** el subdominio y el dominio personalizados; por ejemplo, *api.mydomain.com*.
   * Examine el **Archivo de certificado** en formato PEM.
   * Examine el **Archivo de clave privada** en formato PEM.
   * **Importe**.

## Despliegue de acciones en varias ubicaciones

En esta sección, creará acciones, las expondrá como una API y correlacionará el dominio personalizado con la API con un certificado SSL almacenado en {{site.data.keyword.cloudcerts_short}}.

<p style="text-align: center;">

  ![Arquitectura de API](images/solution44-multi-region-serverless/api-architecture.png)
</p>

La acción **doWork** implementa una de las operaciones de API. La acción **healthz** se va a utilizar más adelante para comprobar si la API está en buen estado. Puede ser una no-op que simplemente devuelva *OK* o puede hacer una comprobación más compleja, como ping de las bases de datos o de otros servicios críticos que requiere su API.

Las tres secciones siguientes se deben repetir para cada ubicación en la que desee alojar el programa de fondo de la aplicación. Para esta guía de aprendizaje, puede seleccionar *Dallas (us-south)* y *London (eu-gb)* como destinos.

### Definición de acciones

1. Vaya a [{{site.data.keyword.openwhisk_short}} / Acciones](https://{DomainName}/openwhisk/actions).
1. Vaya a la ubicación de destino y seleccione una organización y un espacio donde desplegar las acciones.
1. Cree una acción
   1. Establezca **Nombre** en **doWork**.
   1. Establezca **Paquete contenedor** en **default**.
   1. Establezca **Tiempo de ejecución** en la versión más reciente de **Node.js**.
   1. **Crear**.
1. Cambie el código de acción por:
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. **Guardar**
1. Cree otra acción que se utilizará como comprobación del estado de nuestra API:
   1. Establezca **Nombre** en **healthz**.
   1. Establezca **Paquete contenedor** en **default**.
   1. Establezca **Tiempo de ejecución** en la versión más reciente de **Node.js**.
   1. **Crear**.
1. Cambie el código de acción por:
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. **Guardar**

### Exposición de las acciones con una API gestionada

El paso siguiente implica la creación de una API gestionada para exponer sus acciones.

1. Vaya a [{{site.data.keyword.openwhisk_short}} / API](https://{DomainName}/openwhisk/apimanagement).
1. Cree una nueva API de {{site.data.keyword.openwhisk_short}} gestionada:
   1. Establezca **Nombre de API** en **App API**.
   1. Establezca **Vía de acceso base** en **/api**.
1. Cree una operación:
   1. Establezca **Vía de acceso** en **/do**.
   1. Establezca **Verbo** en **GET**.
   1. Establezca **Paquete** en **default**.
   1. Establezca **Acción** en **doWork**.
   1. **Crear**
1. Cree otra operación:
   1. Establezca **Vía de acceso** en **/healthz**.
   1. Establezca **Verbo** en **GET**.
   1. Establezca **Paquete** en **default**.
   1. Establezca **Acción** en **healthz**.
   1. **Crear**
1. **Guarde** la API

### Configuración del dominio personalizado para la API gestionada

La creación de una API gestionada le proporciona un punto final predeterminado como `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`. En esta sección, configurará este punto final para poder gestionar las solicitudes procedentes del subdominio personalizado, el dominio que se configurará más adelante en IBM Cloud Internet Services.

1. Vaya a [API / Dominios personalizados](https://{DomainName}/apis/domains).
1. En el selector **Región**, seleccione la ubicación de destino.
1. Localice el dominio personalizado enlazado con la organización y el espacio donde ha creado las acciones y la API gestionada. Pulse **Cambiar valores** en el menú de acciones.
1. Anote el valor de **Dominio / alias predeterminado**.
1. Marque **Aplicar dominio personalizado**
   1. Establezca el **Nombre de dominio** en el dominio que utilizará con el equilibrador de carga global de CIS, como por ejemplo *api.mydomain.com*.
   1. Seleccione la instancia de {{site.data.keyword.cloudcerts_short}} que contiene el certificado.
   1. Seleccione el certificado para el dominio.
1. Vaya al panel de control de la instancia de **Cloud Internet Services** y, bajo **Fiabilidad / DNS**, cree un nuevo **Registro de TXT DNS**:
   1. Establezca **Nombre** en el subdominio personalizado, como por ejemplo **api**.
   1. Establezca **Contenido** en el **Dominio / alias predeterminado**
   1. Guarde el registro.
1. Guarde los valores del dominio personalizado. El cuadro de diálogo comprobará la existencia del registro DNS TXT.

   Si no se encuentra el registro de TXT, es posible que tenga que esperar a que se propague y tenga que volver a intentar guardar los valores. El registro DNS TXT se puede eliminar cuando se hayan aplicado los valores.
   {: tip}

Repita las secciones anteriores para configurar más ubicaciones.

## Distribución del tráfico entre ubicaciones

**En esta etapa, tiene acciones de configuración en varias ubicaciones**, pero no hay ningún punto de entrada único para acceder a las mismas. En esta sección, configurará un equilibrador de carga global (GLB) para distribuir el tráfico entre las ubicaciones.

<p style="text-align: center;">

  ![Arquitectura del equilibrador de carga global](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### Creación de una comprobación de estado

Los servicios de Internet llamarán regularmente a este punto final para comprobar el estado del programa de fondo.

1. Vaya al panel de control de la instancia de IBM Cloud Internet Services.
1. En **Fiabilidad / Equilibrador de carga global**, cree una comprobación de estado:
   1. Establezca **Tipo de supervisor** en **HTTPS**.
   1. Establezca **Vía de acceso** en **/api/healthz**.
   1. **Suministre el recurso**.

### Creación de agrupaciones de origen

Si crea una agrupación por ubicación, luego puede configurar geo rutas en el equilibrador de carga para redirigir a los usuarios a la ubicación más cercana. Otra opción sería crear una única agrupación con todas las ubicaciones y hacer pasar el ciclo del equilibrador de carga a través de los orígenes de la agrupación.

Para cada ubicación:
1. Cree una agrupación de origen.
1. Establezca **Nombre** en **app-&lt;location&gt;**, como por ejemplo _app-Dallas_.
1. Seleccione la comprobación de estado que ha creado antes.
1. Establezca la **Región de comprobación de estado** en una región cercana a la ubicación en la que se despliegan {{site.data.keyword.openwhisk_short}}.
1. Establezca **Nombre de origen** en **app-&lt;location&gt;**.
1. Establezca la **Dirección de origen** en el dominio / alias predeterminado para la API gestionada (por ejemplo, _5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_).
1. **Suministre el recurso**.

### Creación de un equilibrador de carga global

1. Cree un equilibrador de carga.
1. Establezca **Nombre de host del equilibrador** en **api.mydomain.com**.
1. Añada las agrupaciones de orígenes regionales.
1. **Suministre el recurso**.

Después de un breve período de tiempo, vaya a `https://api.mydomain.com/api/do?name=John&place=Earth`. La función debería ejecutarse en la primera agrupación en buen estado.

### Prueba de la migración tras error

Para probar la migración tras error, una comprobación de estado de la agrupación debe fallar de modo que el GLB redirija a la siguiente agrupación en buen estado. Para simular una anomalía, puede modificar la función de comprobación de estado para hacer que falle.

1. Vaya a [{{site.data.keyword.openwhisk_short}} / Acciones](https://{DomainName}/openwhisk/actions).
1. Seleccione la primera ubicación configurada en el GLB.
1. Edite la función `healthz` y cambie su implementación por `throw new Error()`.
1. Guarde los datos.
1. Espere a que se ejecute la comprobación de estado para esta agrupación de origen.
1. Vuelva a obtener `https://api.mydomain.com/api/do?name=John&place=Earth`; ahora debería redirigir al otro origen en buen estado.
1. Invierta los cambios en el código para recuperar el origen en buen estado.

## Eliminación de recursos
{: #removeresources}

### Eliminación de los recursos de CIS

1. Elimine el GLB.
1. Elimine las agrupaciones de origen.
1. Elimine las comprobaciones de estado.

### Eliminación de acciones

1. Elimine las [API](https://{DomainName}/openwhisk/apimanagement)
1. Elimine las [acciones](https://{DomainName}/openwhisk/actions)

## Contenido relacionado
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Clústeres de Kubernetes de varias regiones resistentes y seguros con Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
