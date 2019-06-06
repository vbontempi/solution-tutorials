---
copyright:
  years: 2017, 2019
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

# Creación, protección y gestión de las API REST
{: #create-manage-secure-apis}

En esta guía de aprendizaje se muestra cómo crear API REST mediante la infraestructura de API LoopBack Node.js. Con Loopback se pueden crear rápidamente API REST que conecten dispositivos y navegadores a datos y servicios. También añadirá funciones de gestión, visibilidad, seguridad y limitación de tasas a las API mediante {{site.data.keyword.apiconnect_long}}.
{:shortdesc}

## Objetivos

* Creación de una API REST con poca o ninguna codificación
* Publicación de la API en {{site.data.keyword.Bluemix_notm}} para que accedan los desarrolladores
* Incorporación de API existentes a {{site.data.keyword.apiconnect_short}}
* Exposición segura y control de acceso a los sistemas de registro

## Servicios utilizados

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:

* [Bucle de retorno](https://loopback.io/)
* [{{site.data.keyword.apiconnect_short}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
* App de Cloud Foundry [SDK para Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs)

## Arquitectura

![Arquitectura](images/solution13/Architecture.png)

1. El desarrollador define la API RESTful
2. El desarrollador publica la API en {{site.data.keyword.apiconnect_long}}
3. Los usuarios y las aplicaciones consumen la API

## Antes de empezar

* Descargue e instale [Node.js](https://nodejs.org/en/download/) versión 6.x (utilice [nvm](https://github.com/creationix/nvm) o similar si ya tiene instalada una versión más reciente de Node.js)

## Creación de una API REST en Node.js

{: #create_api}
En esta sección, creará una API en Node.js mediante [LoopBack](https://loopback.io/doc/index.html). LoopBack es una infraestructura ampliable de Node.js de código abierto que le permite crear API REST completas y dinámicas con poca o sin ninguna codificación.

### Creación de la aplicación

1. Instalar la herramienta de línea de mandatos de {{site.data.keyword.apiconnect_short}}. Si tiene algún problema para instalar {{site.data.keyword.apiconnect_short}}, utilice `sudo` antes del mandato o siga las instrucciones que encontrará [aquí](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_prereq_install_toolkit#installing-the-api-connect-toolkit).
    ```sh
    npm install -g apiconnect
    ```
    {: pre}
2. Especifique el mandato siguiente para crear la aplicación.
    ```sh
    apic loopback --name entries-api
    ```
    {: pre}
3. Pulse `Intro` para utilizar **entries-api** como **nombre de la aplicación**.
4. Pulse `Intro` para utilizar **entries-api** como **directorio que contendrá el proyecto**.
5. Seleccione **3.x (actual)** como **versión de LoopBack**.
6. Seleccione **empty-server** como **tipo de aplicación**.

![Diseño de APIC Loopback](images/solution13/apic_loopback.png)

### Adición de un origen de datos

Los orígenes de datos representan sistemas de fondo como bases de datos, API REST externas, servicios web SOAP y servicios de almacenamiento. Los orígenes de datos suelen proporcionar funciones de creación, recuperación, actualización y supresión (CRUD). Aunque Loopback da soporte a muchos tipos de [orígenes de datos](http://loopback.io/doc/en/lb3/Connectors-reference.html), para simplificar el proceso utilizará un almacén de datos en memoria con la API.

1. Vaya al directorio del nuevo proyecto e inicie el programa API Designer.
    ```sh
    cd entries-api
    ```
    {: pre}

    ```sh
    apic edit
    ```
    {: pre}
2. En el menú de navegación, pulse **Orígenes de datos**. A continuación, pulse el botón **Añadir**. Se abrirá el diálogo **Nuevo origen de datos LoopBack**.
3. Escriba `entriesDS` en el campo de texto **Nombre** y pulse el botón **Nuevo**.
4. Seleccione **In-memory db** en el recuadro combinado **Conector**.
5. Pulse **Todos los orígenes de datos** en la parte superior izquierda. El origen de datos aparecerá en la lista de orígenes de datos.

   El editor actualiza automáticamente el archivo server/datasources.json
con los valores del nuevo origen de datos.
   {:tip}

![recursos de datos de API Designer](images/solution13/datastore.png)

### Adición de un modelo

Un modelo es un objeto JavaScript con las API Node y REST que representan los datos en sistemas de fondo. Los modelos se conectan a estos sistemas mediante orígenes de datos. En esta sección, definirá la estructura de sus datos y los conectará con el origen de datos creado anteriormente.

1. En el menú de navegación, pulse **Modelos**. A continuación, pulse el botón **Añadir**. Se abrirá el diálogo **Nuevo modelo de LoopBack**.
2. Escriba `entry` en el campo de texto **Nombre** y pulse el botón **Nuevo**.
3. Seleccione **entriesDS** en el recuadro combinado **Origen de datos**.
4. En la tarjeta **Propiedades**, añada las propiedades siguientes utilizando el icono **Añadir propiedad**.
    1. Escriba `name` para el **Nombre de propiedad** y seleccione **string** en el recuadro combinado **Tipo**.
    2. Escriba `email` para el **Nombre de propiedad** y seleccione **string** en el recuadro combinado **Tipo**.
    3. Escriba `comment` para el **Nombre de propiedad** y seleccione **string** en el recuadro combinado **Tipo**.
5. Pulse el icono **Guardar** en la parte superior derecha para guardar el modelo.

![Generador de modelos](images/solution13/models.png)

## Prueba de la aplicación LoopBack

En esta sección, iniciará una instancia local de la aplicación Loopback y probará la API insertando y consultando datos mediante API Designer. Debe utilizar la herramienta Explore del programa API Designer para probar los puntos finales REST en el navegador porque incluye las cabeceras de seguridad adecuadas y otros parámetros de solicitud.

1. Inicie el servidor local pulsando el icono **Iniciar** en la esquina inferior izquierda y espere a que se aparezca el mensaje **En ejecución**.
2. En el mensaje de cabecera, pulse el enlace **Explore** para ver la herramienta Explore de API Designer. La barra lateral muestra las operaciones REST disponibles para los modelos de LoopBack en la API.
3. Pulse la operación **entry.create** en el panel izquierdo para ver el punto final. El panel central muestra
información de resumen sobre el punto final, incluidos sus parámetros, seguridad, datos de instancia del modelo y códigos de respuesta. El panel derecho proporciona código de ejemplo para llamar al punto final mediante el mandato cURL y lenguajes como Ruby, Python, Java y Node.
4. En el panel de la derecha, pulse el enlace **Pruébelo**. Desplácese hacia abajo hasta **Parámetros** y especifique lo siguiente en el área de texto **data**:
    ```javascript
    {
      "name": "Jane Doe",
      "email": "janedoe@mycompany.com",
      "comment": "Jane likes Blue"
    }
    ```
    {: pre}
5. Pulse el botón **Llamar operación**.   ![Prueba de una API desde API Designer](images/solution13/data_entry_1.png)
6. Confirme que POST se ha ejecutado correctamente comprobando si aparece el **Código de respuesta: 200 OK**. Si ve un mensaje de error debido a un certificado no de confianza para localhost, pulse el enlace proporcionado en el mensaje de error de la herramienta Explorar de API Designer para aceptar el certificado y, a continuación, siga llamando las operaciones en el navegador web. El procedimiento exacto depende del navegador web que está utilizando.
7. Añada otra entrada mediante cURL. Confirme que el puerto coincide con el puerto de la aplicación.
    ```sh
    curl --request POST \
    --url https://localhost:4002/api/entries \
    --header 'accept: application/json' \
    --header 'content-type: application/json' \
    --header 'x-ibm-client-id: default' \
    --header 'x-ibm-client-secret: SECRET' \
    --data '{"name":"John Doe","email":"johndoe@mycomany.com","comment":"John likes Orange"}' \
    --insecure
    ```
    {: pre}
8. Pulse la operación **entry.find** y luego el enlace **Pruébelo** seguido del botón **Llamar operación**. La respuesta muestra todas las entradas. Debería ver JSON correspondiente a **Jane Doe** y a **John Doe**.
![entry_find](images/solution13/find_response.png)

También puede iniciar manualmente la aplicación Loopback emitiendo el mandato `npm start` desde el directorio `entries-api`. Las API REST estarán disponibles en [http://localhost:3000/api/entries](http://localhost:3000/api/entries); sin embargo, {{site.data.keyword.apiconnect_short}} no las gestionará ni las protegerá.
{:tip}

## Creación del servicio {{site.data.keyword.apiconnect_short}}

Para preparar los pasos siguientes, creará un servicio **{{site.data.keyword.apiconnect_short}}** en {{site.data.keyword.Bluemix_notm}}. {{site.data.keyword.apiconnect_short}} actúa como pasarela a la API y también proporciona funciones de gestión y de seguridad y límites de tasas.

1. Inicie la [Lista de recursos](https://{DomainName}/resources) de {{site.data.keyword.Bluemix_notm}}.
2. Vaya a **Catálogo > Integración > {{site.data.keyword.apiconnect_short}}** y pulse el botón **Crear**.

## Publicación de una API en {{site.data.keyword.Bluemix_notm}}

{: #publish}
Utilizará API Designer para desplegar la aplicación en {{site.data.keyword.Bluemix_notm}} como una aplicación de Cloud Foundry y también para publicar la definición de API en **{{site.data.keyword.apiconnect_short}}**. API Designer es su kit de herramientas local. Si lo cierra, vuélvalo a iniciar con `apic edit` desde el directorio del proyecto.

La aplicación también se puede desplegar manualmente con el mandato `ibmcloud cf push`; sin embargo, no estará protegida. Para [importar la API](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rest_landing#tut_rest_landing) en {{site.data.keyword.apiconnect_short}}, utilice el archivo de definición de OpenAPI disponible en la carpeta `definitions`. El despliegue mediante API Designer protege la aplicación e importa la definición automáticamente.
{:tip}

1. De nuevo en API Designer, pulse el enlace **Publicar** en el mensaje de cabecera. Luego pulse **Añadir y gestionar destinos > Añadir destino de IBM Bluemix**.
2. Seleccione la **Región** y la **Organización** en las que desea publicar.
3. Seleccione el catálogo de **recinto de pruebas** y pulse **Siguiente**.
4. Escriba `entries-api-application` en el campo de texto **Escribir un nuevo nombre de aplicación** y pulse el icono **+**.
5. Pulse **entries-api-application** en la lista y pulse **Guardar**.
6. Pulse el icono **Menú** de la parte superior izquierda de la cabecera. Luego pulse **Proyectos** y el elemento de la lista **entries-api**.
7. En la interfaz de usuario de API Designer, pulse los enlaces **API > entries-api > Ensamblar**.
8. En el editor de ensamblajes, pulse el icono **Filtrar políticas**.
9. Seleccione **Políticas de pasarela de DataPower** y pulse **Guardar**.
10. Pulse **Publicar** en la barra superior y seleccione el destino. Seleccione **Publicar aplicación** y Transferir o Publicar productos > Seleccione **Productos específicos** > **entries-api**.
11. Pulse **Publicar** y espere a que la aplicación termine de publicarse.
![Diálogo de publicación](images/solution13/publish.png)

    Una aplicación contiene los modelos de Loopback, los orígenes de datos y el código relacionados con la API. Un producto le permite declarar cómo se pone una API a disposición de los desarrolladores.
    {:tip}

Ahora la API está publicada en {{site.data.keyword.Bluemix_notm}} como aplicación de Cloud Foundry. La puede ver examinando las aplicaciones de Cloud Foundry de la [Lista de recursos](https://{DomainName}/resources) de {{site.data.keyword.Bluemix_notm}}, pero no se puede acceder directamente mediante el URL ya que la aplicación está protegida. En la sección siguiente se muestra cómo se puede acceder a las API gestionadas.

## Pasarela de API

Hasta ahora, ha estado diseñando y probando su API localmente. En esta sección, utilizará {{site.data.keyword.apiconnect_short}} para probar la API desplegada en {{site.data.keyword.Bluemix_notm}}.

1. Inicie la [Lista de recursos](https://{DomainName}/resources) de {{site.data.keyword.Bluemix_notm}}.
2. Localice y seleccione su servicio **{{site.data.keyword.apiconnect_short}}** en **Servicios de Cloud Foundry**.
3. Pulse el menú **Explorar** y pulse en enlace **Recinto de pruebas**.
4. Pulse la operación **entry.create**.
5. En el panel de la derecha, pulse **Probarlo**. Desplácese hacia abajo hasta **Parámetros** y especifique lo siguiente en el área de texto **data**.
    ```javascript
    {
      "name": "Cloud User",
      "email": "cloud@mycompany.com",
      "comment": "Entry on the cloud!"
    }
    ```
6. Pulse el botón **Llamar operación**. Se debería mostrar la respuesta **Código: 200** que indica que se ha realizado correctamente.

![pasarela](images/solution13/gateway.png)

El URL de su API gestionada y protegida se muestra junto a cada operación y debería parecerse al siguiente: `https://us.apiconnect.ibmcloud.com/orgs/ORG-SPACE/catalogs/sb/api/entries`.
{: tip}

## Limitación de velocidad

El establecimiento de límites de velocidad le permite gestionar el tráfico de red para las
API y las operaciones específicas dentro de las API. Un límite de tasa es el número máximo de llamadas permitidas en un intervalo de tiempo específico.

1. De nuevo en API Designer, pulse **Productos > entries-api**.
2. Seleccione **Plan predeterminado** a la izquierda.
3. Amplíe el **Plan predeterminado** y desplácese hacia abajo hasta el campo **Límites de tasas**.
4. Establezca los campos en **10** llamadas / **1** **minuto**.
5. Seleccione **Imponer límite** y pulse el icono **Guardar**.
  ![Página Límite de tasas](images/solution13/rate_limit.png)
6. Siga los pasos de la sección [Publicación de la API en {{site.data.keyword.Bluemix_notm}}](#publish) para volver a publicar la API.

Ahora su API está limitada a 10 solicitudes por minuto. Utilice la característica **Pruébelo** para alcanzar el límite. Consulte más información sobre [Configuración de límites de tasas](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_rate_limit#setting-up-rate-limits) o explore el programa API Designer para ver las características de gestión disponibles.

## Ampliación de la guía de aprendizaje

Enhorabuena: ha creado una API gestionada y segura. A continuación encontrará algunas sugerencias para mejorar la API.

* Añada persistencia mediante el [conector LoopBack de {{site.data.keyword.cloudant}}](https://{DomainName}/catalog/services/cloudant)
* Utilice API Designer para [ver valores adicionales](http://127.0.0.1:9000/#/design/apis/editor/entries-api:1.0.0) para gestionar la API
* Examine **Análisis** y **Visualizaciones** de la API, [disponibles](https://{DomainName}/docs/services/apiconnect/tutorials?topic=apiconnect-tut_insights_analytics#gaining-insights-from-basic-analytics) en {{site.data.keyword.apiconnect_short}}

## Contenido relacionado

* [Documentación de Loopback](https://loopback.io/doc/index.html)
* [Iniciación a {{site.data.keyword.apiconnect_long}}](https://{DomainName}/docs/services/apiconnect?topic=apiconnect-index#index)
