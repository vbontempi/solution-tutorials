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

# Aceleración de la entrega de archivos estáticos mediante una CDN
{: #static-files-cdn}

En esta guía de aprendizaje se muestra cómo alojar y mostrar activos de un sitio web (imágenes, vídeos, documentos) y contenido generado por el usuario en un {{site.data.keyword.cos_full_notm}} y cómo utilizar un [{{site.data.keyword.cdn_full}} (CDN)](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai) para su distribución rápida y segura a usuarios de todo el mundo.

Las aplicaciones web tienen distintos tipos de contenido: contenido HTML, imágenes, vídeos, hojas de estilo en cascada, archivos JavaScript, contenido generado por el usuario. Algunos de estos contenidos cambian a menudo, otros no tanto, a algunos se accede con mucha frecuencia, a otros solo ocasionalmente. A medida que crece la audiencia de la aplicación, es posible que desee descargar la representación de estos contenidos a otro componente, liberando recursos para la aplicación principal. Es posible que también desee que estos contenidos se muestren desde una ubicación cercana a los usuarios de la aplicación, dondequiera que se encuentren.

Hay muchas razones por las que resulta recomendable utilizar una red de entrega de contenido en estas situaciones:
* la CDN almacenará en la memoria caché el contenido y extraer el contenido del origen (los servidores) solo si no está disponible en su memoria caché o si ha caducado;
* con varios centros de datos en todo el mundo, la CDN mostrará al contenido almacenado en memoria caché de la ubicación más cercana a sus usuarios;
* como se ejecuta en un dominio diferente del de su aplicación principal, el navegador podrá cargar más contenido en paralelo; la mayoría de los navegadores tienen un límite en el número de conexiones por nombre de host.

## Objetivos
{: #objectives}

* Carga de archivos a un grupo de {{site.data.keyword.cos_full_notm}}.
* Colocación del contenido a disponibilidad general con una red de entrega de contenido (CDN).
* Exposición de archivos mediante una aplicación web de Cloud Foundry.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los productos siguientes:
   * [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
   * [{{site.data.keyword.cdn_full}}](https://{DomainName}/catalog/infrastructure/cdn-powered-by-akamai)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura

<p style="text-align: center;">
![Arquitectura](images/solution3/Architecture.png)
</p>

1. El usuario accede a la aplicación
2. La aplicación incluye contenido distribuido a través de una red de entrega de contenido
3. Si el contenido no está disponible en la CDN o si ha caducado, la CDN extraer el contenido del origen.

## Antes de empezar
{: #prereqs}

Póngase en contacto con el usuario maestro de la cuenta de la infraestructura para obtener los permisos siguientes:
   * Gestionar cuenta de CDN
   * Gestionar almacenamiento
   * Clave de API

Estos permisos se necesitan para poder ver y utilizar los servicios de almacenamiento y de la CDN.

## Obtención del código de la aplicación web
{: #get_code}

Utilizaremos una aplicación web sencilla con diferentes tipos de contenido, como imágenes, vídeos y hojas de estilo en cascada. Almacenará el contenido en un grupo de almacenamiento y configurará la CDN para que utilice el grupo como su origen.

Para empezar, recupere el código de la aplicación:

   ```sh
   git clone https://github.com/IBM-Cloud/webapp-with-cos-and-cdn
   ```
  {: pre}

## Creación de un almacenamiento de objetos
{: #create_cos}

{{site.data.keyword.cos_full_notm}} ofrece un almacenamiento en la nube flexible, eficiente en coste y escalable para datos no estructurados.

1. Vaya al [catálogo](https://{DomainName}/catalog/) en la consola y seleccione [**Almacenamiento de objetos**](https://{DomainName}/catalog/services/cloud-object-storage) en la sección Almacenamiento.
2. Cree una instancia nueva de {{site.data.keyword.cos_full_notm}}
4. En el panel de control del servicio, pulse **Crear grupo**.
5. Establezca un nombre de grupo exclusivo, como por ejemplo `username-mywebsite`, y pulse **Crear**. Evite utilizar puntos (.) en el nombre del grupo.

## Carga de archivos en un grupo
{: #upload}

En esta sección, utilizará la herramienta de línea de mandatos **curl** para cargar archivos en el grupo.

1. Inicie una sesión en {{site.data.keyword.Bluemix_notm}} desde la CLI y obtenga una señal de IAM de IBM Cloud.
   ```sh
   ibmcloud login
   ibmcloud iam oauth-tokens
   ```
   {: pre}
2. Copie la señal de la salida del mandato del paso anterior.
   ```
   IAM token:  Bearer <token>
   ```
   {: screen}
3. Establezca el valor de la señal y el nombre de grupo en una variable de entorno para facilitar su acceso.
   ```sh
   export IAM_TOKEN=<REPLACE_WITH_TOKEN>
   export BUCKET_NAME=<REPLACE_WITH_BUCKET_NAME>
   ```
   {: pre}
4. Cargue los archivos denominados `a-css-file.css`, `a-picture.png` y `a-video.mp4` desde el directorio de contenido del código de la aplicación web que ha descargado anteriormente. Cargue los archivos en la raíz del grupo.
  ```sh
   cd content
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-picture.png" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: image/png" \
        -T a-picture.png
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-css-file.css" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: text/css" \
        -T a-css-file.css
  ```
  {: pre}
  ```sh
   curl -X "PUT" \
         "https://s3-api.us-geo.objectstorage.softlayer.net/$BUCKET_NAME/a-video.mp4" \
        -H "x-amz-acl: public-read" \
        -H "Authorization: Bearer $IAM_TOKEN" \
        -H "Content-Type: video/mp4" \
        -T a-video.mp4
  ```
  {: pre}
5. Visualice los archivos desde el panel de control.
   ![](images/solution3/Buckets.png)
6. Acceda a los archivos a través de su navegador utilizando un enlace similar al del ejemplo siguiente:

   `http://s3-api.us-geo.objectstorage.softlayer.net/YOUR_BUCKET_NAME/a-picture.png`

## Colocación de los archivos a disponibilidad general con una CDN

En esta sección, creará un servicio CDN. El servicio CDN distribuye el contenido en el lugar en el que se necesita. La primera vez que se solicita contenido, se extrae del servidor de host (su grupo en {{site.data.keyword.cos_full_notm}}) en la red y se queda allí para que otros usuarios acceden a dicho contenido rápidamente sin la latencia de red para volver a acceder al servidor host.

### Creación de una instancia de CDN

1. Vaya al catálogo en la consola y seleccione **Red de entrega de contenido** en la sección Red. Esta CDN recibe soporte de Akamai. Pulse **Crear**.
2. En el diálogo siguiente, establezca el **Nombre de host** para la CDN en su dominio personalizado. Aunque establezca un dominio personalizado, puede seguir accediendo al contenido de CDN a través del CNAME proporcionado por IBM. Por lo tanto, si no tiene previsto utilizar un dominio personalizado, puede definir un nombre arbitrario.
3. Establezca el prefijo **CNAME personalizado** en un valor exclusivo.
4. A continuación, en **Configurar el origen**, seleccione **Almacenamiento de objetos** para configurar la CDN para COS.
5. Establezca el **Punto final** en el punto final de la API del grupo, como por ejemplo *s3-api.us-geo.objectstorage.softlayer.net*.
6. Deje la **Cabecera de host** y la **Vía de acceso** en blanco. Establezca el **Nombre de grupo** en *your-bucket-name*.
7. Habilite los puertos HTTP (80) y HTTPS (443).
8. Para **Certificado SSL**, seleccione *Certificado SAN DV* si desea utilizar un dominio personalizado. De lo contrario, para acceder al almacenamiento con un CNAME, seleccione la opción **Certificado comodín*.
9. Pulse **Crear**.

### Acceso al contenido a través de CNAME de CDN

1. Seleccione la instancia de CDN [en la lista](https://{DomainName}/classic/network/cdn).
2. Si antes ha seleccionado *Certificado SAN DV*, se le solicitará la validación del dominio una vez que se haya completado la configuración inicial. Siga los pasos que se muestran cuando pulse **Ver validación de dominio**.
3. El panel **Detalles** muestra tanto el **Nombre de host** como el **CNAME** de la CDN.
4. Acceda al archivo con `https://your-cdn-cname.cdnedge.bluemix.net/a-picture.png` o, si utiliza un dominio personalizado, con `https://your-cdn-hostname/a-picture.png`. Si omite el nombre de archivo, debería ver en su lugar S3 ListBucketResult.

## Despliegue de la aplicación Cloud Foundry

La aplicación contiene una página web public/index.html que incluye referencias a los archivos alojados ahora en {{site.data.keyword.cos_full_notm}}. El archivo `app.js` del programa de fondo muestra esta página web y sustituye un marcador por la ubicación real de su CDN. De esta forma, la CDN muestra todos los activos que utiliza la página web.

1. Desde un terminal, vaya al directorio en el que ha extraído el código.
   ```
   cd webapp-with-cos-and-cdn
   ```
   {: pre}
2. Envíe la aplicación sin iniciarla.
   ```
   ibmcloud cf push --no-start
   ```
   {: pre}
3. Configure la variable de entorno CDN_NAME para que la app pueda hacer referencia al contenido de CDN.
   ```
   ibmcloud cf set-env webapp-with-cos-and-cdn CDN_CNAME your-cdn.cdnedge.bluemix.net
   ```
   {: pre}
4. Inicie la app.
   ```
   ibmcloud cf start webapp-with-cos-and-cdn
   ```
   {: pre}
5. Acceda a la app con su navegador web; desde la CDN se cargan la hoja de estilos, una imagen y un vídeo.

![](images/solution3/Application.png)

La utilización de una CDN con {{site.data.keyword.cos_full_notm}} constituye una potente combinación que le permite alojar archivos y mostrarlos a usuarios de todo el mundo. También puede utilizar {{site.data.keyword.cos_full_notm}} para almacenar los archivos que los usuarios carguen en la aplicación.

## Eliminación de recursos

* Suprima la aplicación Cloud Foundry
* Suprima el servicio de red de entrega de contenido
* Suprima el servicio {{site.data.keyword.cos_full_notm}} o el grupo

## Contenido relacionado

[{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)

[Gestión de acceso a {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage/iam?topic=cloud-object-storage-getting-started-with-iam#getting-started-with-iam)

[Iniciación a CDN](https://{DomainName}/docs/infrastructure/CDN?topic=CDN-getting-started#getting-started)
