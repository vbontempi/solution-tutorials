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


# Aplicación web y API sin servidor
{: #serverless-api-webapp}

En esta guía de aprendizaje, creará una aplicación web sin servidor alojando contenido de sitio web estático en GitHub Pages e implementando el programa de fondo de la aplicación mediante {{site.data.keyword.openwhisk}}.

Como plataforma controlada por sucesos, {{site.data.keyword.openwhisk_short}} da soporte a una [variedad de casos de uso](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases). Una de ellas es la creación de aplicaciones web y API. Con las apps web, los sucesos son las interacciones entre los navegadores web (o los clientes REST) y la app web, las solicitudes HTTP. En lugar de suministrar una máquina virtual, un contenedor o un entorno de ejecución de Cloud Foundry para desplegar el programa de fondo, puede implementar la API de programa de fondo con una plataforma sin servidor. Esta puede ser una buena solución para evitar pagar por el tiempo de desocupación y dejar la plataforma se escale cuando sea necesario.

Cualquier acción (o función) en {{site.data.keyword.openwhisk_short}} se puede convertir en un punto final HTTP preparado para ser consumido por los clientes web. Cuando están habilitadas para la web, estas acciones se denominan *acciones web*. Si tiene acciones web, las puede ensamblar en una API completa con API Gateway. API Gateway es un componente de {{site.data.keyword.openwhisk_short}} para exponer las API. Se proporciona con seguridad, soporte de OAuth, limitación de tasas y soporte de dominios personalizados.

## Objetivos

* Despliegue de un programa de fondo sin servidor y de una base de datos
* Exposición de una API REST
* Alojamiento de un sitio web estático
* Opcional: utilización de un dominio personalizado para la API REST

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

La aplicación que se muestra en esta guía de aprendizaje es un sitio web sencillo de registro de visitas en el que los usuarios pueden publicar mensajes.

<p style="text-align: center;">

   ![Arquitectura](./images/solution8/Architecture.png)
</p>

1. El usuario accede a la aplicación alojada en GitHub Pages.
2. La aplicación web llama a una API de programa de fondo.
3. La API de programa de fondo se define en API Gateway.
4. API Gateway reenvía la solicitud a [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk).
5. Las acciones de {{site.data.keyword.openwhisk_short}} utilizan [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) para almacenar y recuperar entradas del registro de visitas.

## Antes de empezar
{: #prereqs}

En esta guía se utiliza GitHub Pages para alojar el sitio web estático. Asegúrate de tener una cuenta de GitHub público.

## Creación de la base de datos Guestbook

Empezaremos por crear un {{site.data.keyword.cloudant_short_notm}}. {{site.data.keyword.cloudant_short_notm}} es una capa de datos totalmente gestionada diseñada para aplicaciones web y móviles modernas que aprovecha un esquema JSON flexible. {{site.data.keyword.cloudant_short_notm}} se basa y es compatible con Apache CouchDB y es accesible a través de una API HTTPS segura, que se escala a medida que crece la aplicación.

1. En el catálogo, seleccione **Cloudant**.
2. Establezca como nombre de servicio **guestbook-db**, seleccione **Utilizar credenciales antiguas e IAM** como métodos de autenticación y pulse **Crear**.
3. De nuevo en la lista de recursos, pulse la entrada ***guestbook-db** bajo la columna Nombre. Nota: es posible que tenga que esperar hasta que se suministre el servicio. 
4. En la pantalla de detalles del servicio, pulse ***Iniciar panel de control de Cloudant***, que se abrirá en otro separador del navegador. Nota: es posible que tenga que iniciar una sesión en la instancia de Cloudant. 
5. Pulse ***Crear base de datos*** y cree una base de datos llamada ***guestbook***.

  ![](images/solution8/Create_Database.png)

6. De nuevo en el separador de detalles del servicio, bajo **Credenciales de servicio**
   1. Cree una **Nueva credencial**, acepte los valores predeterminados y pulse **Añadir**.
   2. Pulse **Ver credenciales** bajo Acciones. Necesitaremos estas credenciales más adelante para permitir que las acciones de Cloud Functions lean/escriban en el servicio Cloudant.

## Creación de acciones sin servidor

En esta sección, creará acciones sin servidor (normalmente denominadas funciones). {{site.data.keyword.openwhisk}} (que se basa en Apache OpenWhisk) es una plataforma de función como servicio (FaaS) que ejecuta funciones en respuesta a sucesos entrantes y que no cuesta nada cuando no se utiliza.

![](images/solution8/Functions.png)

### Secuencia de acciones para guardar la entrada del registro de visitas

Creará una **secuencia**, que es una cadena de acciones en la que la salida de una acción actúa como entrada para la acción siguiente, y así sucesiva. La primera secuencia que creará se utilizará para conservar un mensaje dirigido al visitante. Si se proporciona un nombre, un ID de correo electrónico y un comentario, la secuencia será la siguiente:
   * Cree un documento que desee conservar.
   * Almacene el documento en la base de datos de {{site.data.keyword.cloudant_short_notm}}.

Empiece por crear la primera acción:

1. Vaya a **Funciones** https://{DomainName}/openwhisk.
2. En el panel de la izquierda, pulse **Acciones** y **Crear**.
3. **Cree una acción** llamada `prepare-entry-for-save` y seleccione **Node.js** como tiempo de ejecución (Nota: seleccione la versión más reciente).
4. Sustituya el código existente por el siguiente fragmento de código:
   ```js
   /**
    * Preparar entrada del registro de visitas que se desea conservar
    */
   function main(params) {
     if (!params.name || !params.comment) {
       return Promise.reject({ error: 'no name or comment'});
     }

     return {
       doc: {
         createdAt: new Date(),
          name: params.name,
          email: params.email,
          comment: params.comment
       }
     };
   }
   ```
   {: codeblock}
1. **Guardar**

Luego añada la acción a una secuencia:

1. Pulse **Secuencias de delimitación** y luego **Añadir a secuencia**.
1. Como nombre de la secuencia especifique `save-guestbook-entry-sequence` y pulse **Crear y añadir**.

Por último, añada una segunda acción a la secuencia:

1. Pulse **save-guestbook-entry-sequence** y **Añadir**.
1. Seleccione **Utilizar público**, **Cloudant** y, a continuación, seleccione **create-document** en **Acciones**
1. Cree un **Nuevo enlace** 
1. Como nombre especifique `binding-for-guestbook`
1. Para la instancia de Cloudant, seleccione `Especificar sus propias credenciales` y cumplimente los siguientes campos con la información de credenciales capturada para el servicio cloudant: nombre de usuario, contraseña, host y base de datos = `guestbook` y pulse **Añadir** y luego **Guardar**.
   {: tip}
1. Para probarlo, pulse **Cambiar entrada** y escriba el siguiente JSON
    ```json
    {
      "name": "John Smith",
      "email": "john@smith.com",
      "comment": "this is my comment"
    }
    ```
    {: codeblock}
1. Pulse **Aplicar** y luego **Invocar**.
    ![](images/solution8/Save_Entry_Invoke.png)

### Secuencia de acciones para recuperar entradas

La segunda secuencia se utiliza para recuperar las entradas del registro de visitas existente. La secuencia será la siguiente:
   * Obtener una lista de todos los documentos de la base de datos.
   * Dar formato a los documentos y devolverlos.

1. En **Funciones**, pulse **Acciones** y luego **Cree** una nueva acción de Node.js y llámela `set-read-input`.
2. Sustituya el código existente por el siguiente fragmento de código. Esta acción pasa los parámetros adecuados a la siguiente acción.
   ```js
   function main(params) {
     return {
       params: {
         include_docs: true
       }
     };
   }
   ```
   {: codeblock}
1. **Guardar**

Añada la acción a una secuencia:

1. Pulse **Secuencias de delimitación**, **Añadir a secuencia** y **Crear nueva**
1. Especifique `read-guestbook-entries-sequence` como **Nombre de acción** y pulse **Crear y añadir**.

Complete la secuencia:

1. Pulse la secuencia **read-guestbook-entries-sequence** y luego pulse **Añadir** para crear y añadir la segunda acción para obtener documentos de Cloudant.
1. En **Utilizar pública**, seleccione **{{site.data.keyword.cloudant_short_notm}}** y, a continuación, **list-documents**
1. En **Mis enlaces**, seleccione **binding-for-guestbook** y **Añadir** para crear y añadir esta acción pública a la secuencia.
1. Vuelva a pulsar **Añadir** para crear y añadir la tercera acción que dará formato los documentos de {{site.data.keyword.cloudant_short_notm}}.
1. En **Crear nuevo**, especifique `format-entries` como nombre y pulse **Crear y añadir**.
1. Pulse **format-entries** y sustituya el código por el siguiente:
   ```js
   const md5 = require('spark-md5');

   function main(params) {
     return {
       entries: params.rows.map((row) => { return {
         name: row.doc.name,
         email: row.doc.email,
         comment: row.doc.comment,
         createdAt: row.doc.createdAt,
         icon: (row.doc.email ? `https://secure.gravatar.com/avatar/${md5.hash(row.doc.email.trim().toLowerCase())}?s=64` : null)
       }})
     };
   }
   ```
   {: codeblock}
1. **Guardar**
1. Elija la secuencia pulsando en **Acciones** y luego **read-guestbook-entries-sequence**.
1. Pulse **Guardar** y luego **Invocar**. La salida se debería parecer a la siguiente: ![](images/solution8/Read_Entries_Invoke.png)

## Creación de una API

![](images/solution8/Cloud_Functions_API.png)

1. Vaya a Acciones https://{DomainName}/openwhisk/actions.
2. Seleccione la secuencia **read-guestbook-entries-sequence**. Junto al nombre, pulse **Acción web**, marque **Habilitar acción web** y pulse **Guardar**.
3. Haga lo mismo para la secuencia **save-guestbook-entry-sequence**.
4. Vaya a API https://{DomainName}/openwhisk/apimanagement y **cree una API de {{site.data.keyword.openwhisk_short}}**
5. Defina como nombre `guestbook` y como vía de acceso base `/guestbook`
6. Pulse **Crear operación** y cree una operación para recuperar entradas del registro de visitas:
   1. Establezca la **vía de acceso** en `/entries`
   2. Establezca el **verbo** en `GET*`
   3. Seleccione la acción **read-guestbook-entries-sequence**
7. Pulse **Crear operación** y cree una operación para conservar una entrada del registro de visitas:
   1. Establezca la **vía de acceso** en `/entries`
   2. Establezca el **verbo** en `PUT`
   3. Seleccione la acción **save-guestbook-entry-sequence**
8. Guarde y exponga la API.

## Despliegue de la app web

1. Dirija el repositorio de la interfaz de usuario Guestbook https://github.com/IBM-Cloud/serverless-guestbook a su GitHub público.
2. Modifique **docs/guestbook.js** y sustituya el valor de **apiUrl** por la ruta suministrada por API Gateway.
3. Confirme el archivo modificado en el repositorio al que lo ha dirigido.
4. En la página de valores del repositorio, desplácese hasta **GitHub Pages**, cambie el origen por **master branch /docs folder** y pulse Guardar.
5. Acceda a la página pública del repositorio.
6. Debería ver la entrada "test" del registro de visitas que se ha creado antes.
7. Añada entradas nuevas.

![](images/solution8/Guestbook.png)

## Opcional: utilización de su propio dominio para la API

La creación de una API gestionada le proporciona un punto final predeterminado como `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app`. En esta sección, configurará este punto final para que pueda manejar las solicitudes procedentes de su subdominio personalizado.

### Obtención de un certificado para el dominio personalizado

Para exponer las acciones de {{site.data.keyword.openwhisk_short}} a través de un dominio personalizado se necesita una conexión HTTPS segura. Debe obtener un certificado SSL para el dominio y el subdominio que tiene previsto utilizar con el servidor de fondo sin servidor. Suponiendo que tiene un dominio como *mydomain.com*, las acciones se pueden alojar en *guestbook-api.mydomain.com*. Se tendrá que emitir el certificado para *guestbook-api.mydomain.com* (o **.mydomain.com*).

Puede conseguir certificados SSL gratuitos en [Cifremos](https://letsencrypt.org/). Durante el proceso, es posible que tenga que configurar un registro DNS de tipo TXT en la interfaz DNS para probar que es el propietario del dominio.
{:tip}

Cuando haya obtenido el certificado SSL y la clave privada para el dominio, asegúrese de convertirlos al formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Para convertir un certificado en formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
1. Para convertir una clave privada al formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```

### Importación del certificado en un repositorio central

1. Cree una instancia de [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) en una ubicación soportada.
1. En el panel de control del servicio, utilice **Importar certificado**:
   * Establezca como **Nombre** el subdominio y el dominio personalizados; por ejemplo, `guestbook-api.mydomain.com`.
   * Examine el **Archivo de certificado** en formato PEM.
   * Examine el **Archivo de clave privada** en formato PEM.
   * **Importe**.

### Configuración del dominio personalizado para la API gestionada

1. Vaya a [API / Dominios personalizados](https://{DomainName}/apis/domains).
1. En el selector de regiones, seleccione la ubicación en la que ha desplegado las acciones.
1. Localice el dominio personalizado enlazado con la organización y el espacio donde ha creado las acciones y la API gestionada. Pulse **Cambiar valores** en el menú de acciones.
1. Anote el valor de **Dominio / alias predeterminado**.
1. Marque **Aplicar dominio personalizado**
   1. Establezca como **Nombre de dominio** el dominio que utilizará, como por ejemplo `guestbook-api.mydomain.com`.
   1. Seleccione la instancia de {{site.data.keyword.cloudcerts_short}} que contiene el certificado.
   1. Seleccione el certificado para el dominio.
1. Vaya al proveedor de DNS y cree un nuevo **registro DNS TXT** correlacionando el dominio con el dominio / alias predeterminado de la API. El registro DNS TXT se puede eliminar cuando se hayan aplicado los valores.
1. Guarde los valores del dominio personalizado. El cuadro de diálogo comprobará la existencia del registro DNS TXT.
1. Por último, vuelva a los valores del proveedor de DNS y cree un registro CNAME para que su dominio personalizado (por ejemplo, guestbook-api.mydomain.com) apunte al dominio/alias predeterminado. Esto hará que el tráfico a través del dominio personalizado se redirija a la API de fondo.

Una vez que los cambios de DNS se hayan propagado, podrá acceder a la API del registro de visitas en https://guestbook-api.mydomain.com/guestbook.

1. Edite **docs/guestbook.js** y actualice el valor de **apiUrl** con https://guestbook-api.mydomain.com/guestbook
1. Confirme el archivo modificado.
1. Ahora su aplicación accede a la API a través del dominio personalizado.

## Eliminación de recursos

* Suprima el servicio {{site.data.keyword.cloudant_short_notm}}
* Suprima la API de {{site.data.keyword.openwhisk_short}}
* Suprima las acciones de {{site.data.keyword.openwhisk_short}}

## Contenido relacionado
* [Más guías y ejemplos sin servidor](https://developer.ibm.com/code/journey/category/serverless/)
* [Iniciación a {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-index#getting-started-with-openwhisk)
* [Casos de uso comunes de {{site.data.keyword.openwhisk}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#openwhisk_common_use_cases)
* [Creación de API REST desde acciones](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_apigateway#openwhisk_apigateway)
