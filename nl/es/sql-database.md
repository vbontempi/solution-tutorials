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

# SQL Database for Cloud Data
{: #sql-database}

En esta guía de aprendizaje se muestra cómo suministrar un servicio de base de datos SQL (relacional), cómo crear una tabla y cómo cargar un conjunto de datos grande (información sobre una ciudad) en la base de datos. A continuación, desplegará una app web "worldcities" para hacer uso de dichos datos y aprender a acceder a la base de datos de la nube. La app se ha escrito en Python mediante la [infraestructura Flask](http://flask.pocoo.org/).

![](images/solution5/Architecture.png)

## Objetivos

* Suministro de una base de datos SQL
* Creación del esquema de la base de datos (tabla)
* Carga de datos
* Conexión de la app y el servicio de base de datos (compartir credenciales)
* Supervisión, seguridad, copia de seguridad y recuperación

## Productos

En esta guía de aprendizaje se utilizan los productos siguientes:
   * [{{site.data.keyword.dashdblong_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)

## Antes de empezar
{: #prereqs}

Vaya a [GeoNames](http://www.geonames.org/) y descargue y extraiga el archivo [cities1000.zip](http://download.geonames.org/export/dump/cities1000.zip). Contiene información sobre ciudades con una población superior a 1000 habitantes. Lo va a utilizar como conjunto de datos.

## Suministro de la base de datos SQL
Empiece por crear una instancia del servicio **[{{site.data.keyword.dashdbshort_notm}}](https://{DomainName}/catalog/services/db2-warehouse-on-cloud)**.

![](images/solution5/Catalog.png)

1. Vaya al [panel de control de {{site.data.keyword.Bluemix_short}}](https://{DomainName}). Pulse **Catálogo** en la barra de navegación superior.
2. Pulse **Datos y análisis** bajo Plataforma en el panel izquierdo y seleccione **{{site.data.keyword.dashdbshort_notm}}**.
3. Elija el plan de **Entrada** y cambie el nombre de servicio sugerido por "sqldatabase" (más adelante utilizará este nombre). Elija una ubicación para el despliegue de la base de datos y asegúrese de que se ha seleccionado la organización y el espacio correctos.
4. Pulse **Crear**. Después de un corto periodo de tiempo, se le debería notificación que la operación se ha ejecutado correctamente.
5. En la **Lista de recursos**, pulse la entrada correspondiente el servicio {{site.data.keyword.dashdbshort_notm}} que acaba de crear.
6. Pulse **Abrir** para iniciar la consola de la base de datos. Si es la primera vez que utiliza la consola, se le ofrece la posibilidad de realizar una visita guiada.

## Creación de una tabla
Necesita una tabla para alojar los datos de ejemplo. Créela mediante la consola.

1. En la consola de {{site.data.keyword.dashdbshort_notm}}, pulse **Explorar** en la barra de navegación. Esto le llevará a una lista de esquemas existentes en la base de datos.
2. Localice y pulse en el esquema que empieza por "DASH".
3. Pulse **"+ Nueva tabla"** para ver un formulario con el nombre de la tabla y sus columnas.
4. Especifique "cities" como nombre de la tabla. Copie las definiciones de columna del archivo [cityschema.txt](https://github.com/IBM-Cloud/cloud-sql-database/blob/master/cityschema.txt) y péguelas en el recuadro correspondiente a columnas y tipos de datos.
5. Pulse **Crear** para definir la nueva tabla.   
   ![](images/solution5/TableCitiesCreated.png)

## Carga de datos
Ahora que se ha creado la tabla "cities", se van a cargar los datos en la misma. Esto se puede hacer de varias formas; por ejemplo, desde la máquina local o desde el almacenamiento de objetos en la nube (COS) con la interfaz Swift o Amazon S3, utilizando el servicio de migración de [{{site.data.keyword.dwl_full}}](https://{DomainName}/catalog/services/lift). En esta guía de aprendizaje, va a cargar datos de su máquina. Durante el proceso, adaptará la estructura de la tabla y el formato de los datos para que se ajusten al contenido de los archivos.

1. En el menú de navegación superior, pulse **Cargar**. A continuación, en **Selección de archivos**, pulse **Examinar archivos** para localizar y seleccionar el archivo "cities1000.txt" que ha descargado en la primera sección de esta guía.
2. Pulse **Siguiente** para ir a la visión general del esquema. Elija de nuevo el esquema que empieza por "DASH" y luego la tabla "CITIES". Vuelva a pulsar **Siguiente**.   

   Puesto que la tabla está vacía, no hay diferencia entre añadir o sobrescribir los datos existentes.
   {:tip }
3. Ahora personalice la forma en que se interpretan los datos del archivo "cities1000.txt" durante el proceso de carga. En primer lugar, inhabilite "Cabecera en primera fila" porque el archivo solo contiene datos. Luego escriba "0x09" como separador. Esto significa que los valores que hay dentro del archivo están delimitados por un tabulador. Por último, seleccione "AAAA-MM-DD" como formato de fecha. El resultado debería parecerse al de la siguiente captura de pantalla.    
  ![](images/solution5/LoadTabSeparator.png)
4. Pulse **Siguiente** y se le ofrecerá que revise los valores de carga. Acepte dichos valores y pulse **Iniciar carga** para empezar a cargar los datos en la tabla "CITIES". Se visualiza el progreso. Una vez que los datos se carguen, en unos segundos la carga debería finalizar y se deberían mostrar algunas estadísticas.   
   ![](images/solution5/LoadProgressSteps.png)

## Verificación de los datos cargados mediante SQL
Los datos se han cargado en la base de datos relacional. No se han producido errores, pero de todos modos debe realizar algunas pruebas rápidas. Utilice el editor de SQL incorporado para escribir y ejecutar algunas sentencias de SQL.

1. En el menú de navegación superior, pulse **Ejecutar SQL**.
   En lugar del editor de SQL incorporado, puede utilizar herramientas de SQL basadas en la nube y tradicionales en el escritorio o en la máquina servidor con {{site.data.keyword.dashdbshort_notm}}. Encontrará la información de conexión en el menú de configuración. Incluso puede descargar algunas herramientas en la sección "Descargas" del menú que se ofrece detrás del icono de "libro" (correspondiente a documentación y ayuda).
    {:tip }
2. En el "Editor de SQL", escriba o copie la consulta siguiente:   
   ```bash
   select count(*) from cities
   ```
   {:codeblock}
   Luego pulse el botón **Ejecutar todo**. En la sección de resultados, debe mostrarse el mismo número de filas indicado por el proceso de carga.   
3. En el "Editor de SQL", escriba la sentencia siguiente en una línea nueva:
   ```
   select countrycode, count(name) from cities
   group by countrycode
   order by 2 desc
   ```
   {:codeblock}   
4. En el editor, seleccione el texto de la sentencia anterior. Pulse el botón **Ejecutar seleccionados**. Ahora solo se debería ejecutar esta, que deberá devolver algunas estadísticas del país en la sección de resultados.

## Despliegue del código de aplicación
El [código de la app de base de datos, listo para ser ejecutado, se encuentra en el repositorio Github](https://github.com/IBM-Cloud/cloud-sql-database). Clone o descargue el repositorio y luego envíelo a IBM Cloud.

1. Clone el repositorio Github:
   ```bash
   git clone https://github.com/IBM-Cloud/cloud-sql-database
   cd cloud-sql-database
   ```
2. Envíe la aplicación a IBM Cloud. Debe haber iniciado una sesión en la ubicación, la organización y el espacio donde se ha suministrado la base de datos. Copie y pegue estos mandatos, uno en cada línea.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push your-app-name
   ```
3. Una vez que finalice el proceso de envío, debería poder acceder a la app. No es necesario efectuar ninguna configuración adicional. El archivo `manifest.yml` indica a IBM Cloud que enlace la app con el servicio de base de datos denominado "sqldatabase".

## Seguridad, copia de seguridad y recuperación, supervisión
{{site.data.keyword.dashdbshort_notm}} es un servicio gestionado. IBM se ocupa de proteger el entorno, de realizar copias de seguridad diarias y de supervisar el sistema. En el plan de entrada, el entorno de base de datos es una configuración de varios arrendatarios con administración reducida y opciones configuradas para los usuarios. Sin embargo, si utiliza uno de los planes de empresa, existen [varias opciones para gestionar usuarios, para configurar seguridad adicional para la base de datos](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.security.doc/doc/security.html) y para [supervisar la base de datos](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.admin.mon.doc/doc/c0001138.html).   

Además de las opciones de administración tradicionales, el [servicio {{site.data.keyword.dashdbshort_notm}} también ofrece una API REST para supervisión, gestión de usuarios, programas de utilidad, carga, acceso a almacenamiento y más](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/connecting/connect_api.html). Se puede acceder a la interfaz de Swagger ejecutable de dicha API desde el menú que hay detrás del icono "libro" bajo "API Rest". Algunas herramientas que se pueden utilizar para la supervisión y más, como por ejemplo IBM Data Server Manager, también se pueden descargar bajo la sección "Descargas" en ese mismo menú.

## Prueba de la app
La app que muestra información sobre una ciudad en función del conjunto de datos que se ha cargado se reduce a un mínimo. Ofrece un formulario de búsqueda para especificar el nombre de una ciudad y pocas ciudades preconfiguradas. Las solicitudes se especifican como `/search?name=cityname` (formulario de búsqueda) o `/city/cityname` (ciudades especificadas directamente). Ambas solicitudes se especifican desde las mismas líneas de código en segundo plano. El nombre de la ciudad se pasa como valor a una sentencia de SQL preparada mediante un marcador de parámetro por razones de seguridad. Las filas se captan de la base de datos y se pasan a una plantilla HTML para que su representación.

## Limpieza
Para limpiar los recursos utilizados por la guía de aprendizaje, siga estos pasos:
1. Vaya a la [Lista de recursos de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources). Localice la app.
2. Pulse en el icono de menú correspondiente a la app y seleccione **Suprimir app**. En la ventana de diálogo, marque la opción correspondiente al servicio {{site.data.keyword.dashdbshort_notm}} relacionado que desea suprimir.
3. Pulse el botón **Suprimir**. La app y el servicio de base de datos se eliminan y el usuario vuelve a la lista de recursos.

## Ampliación de la guía de aprendizaje
¿Desea ampliar esta app? Estas son algunas de las ideas:
1. Ofrecer una búsqueda con comodín en los nombres alternativos.
2. Buscar ciudades de un país específico y con determinados valores de población.
3. Cambiar el diseño de la página mediante la sustitución de estilos de CSS y la ampliación de plantillas.
4. Permitir la creación de información sobre nueva ciudad basada en formularios o permitir actualizaciones de los datos existentes, como por ejemplo la población.

## Contenido relacionado
* Documentación: [IBM Knowledge Center de {{site.data.keyword.dashdbshort_notm}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Preguntas frecuentes sobre {{site.data.keyword.Db2_on_Cloud_long_notm}} y {{site.data.keyword.dashdblong_notm}}](https://www.ibm.com/support/knowledgecenter/SS6NHC/com.ibm.swg.im.dashdb.doc/managed_service.html), donde se responde a preguntas relacionadas con el servicio gestionado, la copia de seguridad de datos, el cifrado y la seguridad de los datos y mucho más.
* [Db2 Developer Community Edition gratuito](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) para desarrolladores
* Documentación: [Descripción de API del controlador Python ibm_db](https://github.com/ibmdb/python-ibmdb/wiki/APIs)
* [IBM Data Server Manager](https://www.ibm.com/us-en/marketplace/data-server-manager)
