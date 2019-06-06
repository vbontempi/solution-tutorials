---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Creación de un lago de datos mediante el almacenamiento de objetos
{: #smart-data-lake}

Las definiciones del término lago de datos varían, pero, en el contexto de esta guía de aprendizaje, un lago de datos en un método de almacenamiento de datos en su formato nativo para su uso por parte de la organización. Con este fin, creará un lago de datos para su organización mediante {{site.data.keyword.cos_short}}. Combinando {{site.data.keyword.cos_short}} y SQL Query, los analistas de datos pueden consultar los datos allí donde estén mediante SQL. También aprovechará el servicio SQL Query en Jupyter Notebook para realizar un análisis sencillo. Cuando termine, permita que usuarios que no sean técnicos descubran la información que les interese mediante {{site.data.keyword.dynamdashbemb_notm}}.

## Objetivos

- Utilizar {{site.data.keyword.cos_short}} para almacenar archivos de datos sin formato
- Consultar datos directamente desde {{site.data.keyword.cos_short}} mediante SQL Query
- Perfilar y analizar datos en {{site.data.keyword.DSX_full}}
- Compartir datos en la organización con {{site.data.keyword.dynamdashbemb_notm}}

## Servicios utilizados

- [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
- [SQL Query](https://{DomainName}/catalog/services/sql-query)
- [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio)
- [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Arquitectura

![Arquitectura](images/solution29/architecture.png)

1. Los datos sin formato se almacenan en {{site.data.keyword.cos_short}}
2. Los datos se reducen, se mejoran o se perfilan con SQL Query
3. El análisis de datos se realiza en {{site.data.keyword.DSX}}
4. La línea de negocio accede a una aplicación web
5. Los datos perfilados se extraen de {{site.data.keyword.cos_short}}
6. Los gráficos de línea de negocio se crean mediante {{site.data.keyword.dynamdashbemb_notm}}

## Antes de empezar

- [Instale Git](https://git-scm.com/)
- [Instale la CLI de {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview)
- [Instale Aspera Connect](http://downloads.asperasoft.com/connect2/)
- [Instale Node.js y NPM](https://nodejs.org)

## Creación de servicios

En esta sección, creará los servicios necesarios para crear el lago de datos.

En esta sección se utiliza la línea de mandatos para crear instancias de servicio. Si lo desea, puede hacer lo mismo desde la página del servicio en el catálogo utilizando los enlaces que se proporcionan.
{: tip}

1. Inicie una sesión en {{site.data.keyword.cloud_notm}} mediante la línea de mandatos y seleccione la cuenta de Cloud Foundry. Consulte [Iniciación a la CLI]https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Cree una instancia de [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) con un alias de Cloud Foundry. Si ya tiene una instancia de servicio, ejecute el mandato `service-alias-create` con el nombre de servicio existente.
    ```sh
    ibmcloud resource service-instance-create data-lake-cos cloud-object-storage lite global
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-cos --instance-name data-lake-cos
    ```
    {: pre}
3. Cree una instancia de [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create data-lake-sql sql-query lite us-south
    ```
    {: pre}
4. Cree una instancia de [{{site.data.keyword.DSX}}](https://{DomainName}/catalog/services/watson-studio).
    ```sh
    ibmcloud service create data-science-experience free-v1 data-lake-studio
    ```
    {: pre}
5. Cree una instancia de [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded) con un alias de Cloud Foundry.
    ```sh
    ibmcloud resource service-instance-create data-lake-dde dynamic-dashboard-embedded lite us-south
    ```
    {: pre}
    ```sh
    ibmcloud resource service-alias-create dashboard-nodejs-dde --instance-name data-lake-dde
    ```
    {: pre}
6. Vaya a un directorio de trabajo y ejecute el siguiente mandato para clonar el [repositorio GitHub](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard) de la aplicación del panel de control. Luego envíe la aplicación a la organización de Cloud Foundy. La aplicación enlazará automáticamente los servicios anteriores que necesite mediante su archivo [manifest.yml](https://github.com/IBM-Cloud/nodejs-data-lake-dashboard/blob/master/manifest.yml).
    ```sh
    git clone https://github.com/IBM-Cloud/nodejs-data-lake-dashboard.git
    ```
    {: pre}
    ```sh
    cd nodejs-data-lake-dashboard
    ```
    {: pre}
    ```sh
    npm install
    ```
    {: pre}
    ```sh
    npm run push
    ```
    {: pre}

    Después del despliegue, la aplicación será pública y escuchará en un nombre de host aleatorio. Puede ir a la página [Lista de recursos](https://{DomainName}/resources), seleccionar la app en Apps de Cloud Foundry y ver el URL y ejecutar el mandato `ibmcloud cf app dashboard-nodejs routes` para ver rutas.
    {: tip}

7. Confirme que la aplicación está activa accediendo a su URL público en el navegador.

![Página de inicio del panel de control](images/solution29/dashboard-start.png)

## Carga de datos

En esta sección, cargará datos en un grupo de {{site.data.keyword.cos_short}} utilizando {{site.data.keyword.CHSTSshort}} incorporado. {{site.data.keyword.CHSTSshort}} protege los datos a medida que se cargan en el grupo y [puede reducir enormemente el tiempo de transferencia](https://www.ibm.com/blogs/bluemix/2018/03/ibm-cloud-object-storage-simplifies-accelerates-data-to-the-cloud/).

1. Descargue el archivo CSV [City of Los Angeles / Traffic Collision Data from 2010](https://data.lacity.org/api/views/d5tf-ez2w/rows.csv?accessType=DOWNLOAD). El archivo tiene 81 MB y puede tardar unos minutos en descargarse.
2. En el navegador, acceda a la instancia de servicio **data-lake-cos** desde la [Lista de recursos](https://{DomainName}/resources).
3. Cree un nuevo grupo para almacenar los datos.
    - Pulse el botón **Crear un grupo**.
    - Seleccione **Regional** en el menú desplegable **Resistencia**.
    - Seleccione **us-south** en **Ubicación**. Actualmente {{site.data.keyword.CHSTSshort}} solo está disponible para los grupos creados en la ubicación `us-south`. Como alternativa, elija otra ubicación y utilice el tipo de transferencia **Estándar** en la sección siguiente.
    - Especifique un **Nombre** de grupo y pulse **Crear**. Si recibe el error *AccessDenied*, intente con un nombre de grupo más específico.
4. Cargue el archivo CSV en {{site.data.keyword.cos_short}}.
    - Desde el grupo, pulse el botón **Añadir objetos**.
    - Seleccione el botón de selección **Transferencia de alta velocidad de Aspera**.
    - Pulse el botón **Añadir archivos**. Se abrirá el plugin de Aspera, que estará en una ventana separada, posiblemente detrás de la ventana del navegador.
    - Localice el archivo CSV que ha descargado anteriormente y selecciónelo.

![Grupo con archivo CSV](images/solution29/cos-bucket.png)

## Utilización de datos

En esta sección convertirá el conjunto de datos original sin formato en un archivo de destino basado en los atributos de tiempo y antigüedad. Esto resulta de gran ayuda para los consumidores del lago de datos que tienen intereses específicos o que deben gestionar conjuntos de datos muy grandes.

Utilizará SQL Query para manipular los datos donde residan en {{site.data.keyword.cos_short}} mediante sentencias SQL con las que está familiarizado. SQL Query ofrece soporte integrado para CSV, JSON y Parquet; no se necesita ningún otro servicio de cálculo ni sistema de extracción, transformación y carga adicional.

1. Acceda a la instancia del servicio SQL Query **data-lake-sql** desde su [Lista de recursos](https://{DomainName}/resources).
2. Seleccione **Abrir IU**.
3. Cree un nuevo conjunto de datos ejecutando SQL directamente en el archivo CSV cargado anteriormente.
    - Escriba las siguientes sentencias SQL en el área de texto **Escribir SQL aquí...**.
        ```
        SELECT
        `Dr Number` AS id,
        `Date Occurred` AS date,
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
        FROM cos://us-south/<your-bucket-name>/Traffic_Collision_Data_from_2010_to_Present.csv
        WHERE
        `Time Occurred` >= 1700 AND
        `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND
        `Victim Age` <= 35
        ```
        {: codeblock}
    - Sustituya el URL de la cláusula `FROM` por el nombre del grupo.
4. El valor **Target** creará automáticamente un grupo de {{site.data.keyword.cos_short}} que contendrá el resultado. Cambie el valor de **Target** por `cos://us-south/<your-bucket-name>/results`.
5. Pulse el botón **Ejecutar**. Los resultados aparecerán a continuación.
6. En el separador **Detalles de la consulta**, pulse el icono **Iniciar** situado junto al URL de **Ubicación del resultado** para ver el conjunto de datos provisional, que ahora también se almacena en {{site.data.keyword.cos_short}}.

![Cuaderno](images/solution29/sql-query.png)

## Combinación de cuadernos Jupyter con SQL Query

En esta sección, utilizará el cliente de SQL Query en Jupyter Notebook. Con esto se reutilizan los datos almacenados {{site.data.keyword.cos_short}} en una herramienta de análisis de datos. La combinación también crea conjuntos de datos que se almacenan automáticamente en {{site.data.keyword.cos_short}} y que luego se pueden utilizar con {{site.data.keyword.dynamdashbemb_notm}}.

1. Cree un nuevo Jupyter Notebook en {{site.data.keyword.DSX}}.
    - En un navegador, abra [{{site.data.keyword.DSX}}](https://dataplatform.ibm.com/home?context=analytics&apps=data_science_experience&nocache=true).
    - Pulse el mosaico **Crear un proyecto** seguido de **Ciencia de datos**.
    - Pulse **Crear proyecto** y luego especifique un **Nombre de proyecto**.
    - Asegúrese de que **Almacenamiento** tiene el valor **data-lake-cos**.
    - Pulse **Crear**.
    - En el proyecto resultante, pulse **Añadir a proyecto** y **Cuaderno**.
    - En el separador **En blanco**, especifique un **Nombre de cuaderno**.
    - Deje los valores predeterminados de **Idioma** y **Tiempo de ejecución**; pulse **Crear cuaderno**.
2. Desde el cuaderno, instale e importe PixieDust e ibmcloudsql añadiendo los siguientes mandatos a la solicitud de entrada **En [ ]: ** y pulse **Ejecutar**.
    ```python
    !conda install pyarrow
    !conda install sqlparse
    !pip install --user ibmcloudsql
    import ibmcloudsql
    from pixiedust.display import *
    ```
    {: codeblock}
3. Añada una clave de API de {{site.data.keyword.cos_short}} al cuaderno de notas. Esto permitirá que los resultados de SQL Query se almacenen en {{site.data.keyword.cos_short}}.
    - Añada lo siguiente en el siguiente indicador **En [ ]: ** y pulse **Ejecutar**.
        ```python
        import getpass
        cloud_api_key = getpass.getpass('Enter your IBM Cloud API Key')
        ```
        {: codeblock}
    - En el terminal, cree una clave de API.
        ```sh
        ibmcloud iam api-key-create data-lake-cos-key
        ```
        {: pre}
    - Copie la **Clave de API** en el portapapeles.
    - Pegue la clave de API en el recuadro de texto en el cuaderno y pulse la tecla `Intro`.
    - También debería guardar la clave de API en un lugar seguro y permanente; el cuaderno no guarda la clave de API.
4. Añada el CRN (nombre de recurso de nube) de la instancia de SQL Query en el cuaderno.
    - En el siguiente indicador **En [ ]: **, asigne el CRN a una variable del cuaderno.
        ```python
        sql_crn = '<SQL_QUERY_CRN>'
        ```
        {: codeblock}
    - En el terminal, copie el CRN de la propiedad **ID** en el portapapeles.
        ```sh
        ibmcloud resource service-instance data-lake-sql
        ```
        {: pre}
    - Pegue la CRN entre comillas simples y pulse **Ejecutar**.
5. Añada otra variable al cuaderno para especificar el grupo de {{site.data.keyword.cos_short}} y pulse **Ejecutar**.
    ```python
    sql_cos_endpoint = 'cos://us-south/<your-bucket-name>'
    ```
    {: codeblock}
6. Ejecute los mandatos siguientes en otro indicador de **En [ ]: ** y en pulse **Ejecutar** para ver el conjunto de resultados. También se añadirá un nuevo archivo `accidents/jobid=<id>/<part>.csv*` al grupo que incluye el resultado de la sentencia `SELECT`.
    ```python
    sqlClient = ibmcloudsql.SQLQuery(cloud_api_key, sql_crn, sql_cos_endpoint + '/accidents')

    data_source = sql_cos_endpoint + "/Traffic_Collision_Data_from_2010_to_Present.csv"

    query = """
    SELECT
        `Time Occurred` AS time,
        `Area Name` AS area,
        `Victim Age` AS age,
        `Victim Sex` AS sex,
        `Location` AS location
    FROM  {}
    WHERE
        `Time Occurred` >= 1700 AND `Time Occurred` <= 2000 AND
        `Victim Age` >= 20 AND `Victim Age` <= 35
    """.format(data_source)

    traffic_collisions = sqlClient.run_sql(query)
    traffic_collisions.head()
    ```
    {: codeblock}

## Visualización de datos mediante PixieDust

En esta sección, visualizará el conjunto de resultados anterior mediante PixieDust y Mapbox para identificar mejor los patrones o puntos de interés correspondientes a incidentes de tráfico.

1. Cree una expresión de tabla común para convertir la columna `location` en columnas `latitude` y `longitude` separadas. **Ejecute** lo siguiente desde el indicador del cuaderno:
    ```python
    query = """
    WITH location AS (
        SELECT
            id,
            cast(split(coordinates, ',')[0] as float) as latitude,
            cast(split(coordinates, ',')[1] as float) as longitude
        FROM (SELECT
                `Dr Number` as id,
                regexp_replace(Location, '[()]', '') as coordinates
            FROM {0}
        )
    )
    SELECT
        d.`Dr Number` as id,
        d.`Date Occurred` as date,
        d.`Time Occurred` AS time,
        d.`Area Name` AS area,
        d.`Victim Age` AS age,
        d.`Victim Sex` AS sex,
        l.latitude,
        l.longitude
    FROM {0} AS d
        JOIN
        location AS l
        ON l.id = d.`Dr Number`
    WHERE
        d.`Time Occurred` >= 1700 AND
        d.`Time Occurred` <= 2000 AND
        d.`Victim Age` >= 20 AND
        d.`Victim Age` <= 35 AND
        l.latitude != 0.0000 AND
        l.latitude != 0.0000
    """.format(data_source)

    traffic_location = sqlClient.run_sql(query)
    traffic_location.head()
    ```
    {: codeblock}
2. En el siguiente indicador **En [ ]: **, **ejecute** el mandato `display` para ver el resultado mediante PixieDust.
    ```python
    display(traffic_location)
    ```
    {: codeblock}
3. Seleccione el botón desplegable del gráfico y luego seleccione **Correlacionar**.
4. Añada `latitude` y `longitude` a **Claves**. Añada `id` y `age` a **Valores**. Pulse **Aceptar** para ver la correlación.
5. Pulse el icono **Guardar** para guardar el cuaderno en {{site.data.keyword.cos_short}}.

![Cuaderno](images/solution29/notebook-mapbox.png)

## Compartición del conjunto de datos con la organización

No todos los usuarios del lago de datos son analistas de datos. Puede permitir que usuarios que no sean técnicos obtengan información del lago de datos mediante {{site.data.keyword.dynamdashbemb_notm}}. De forma parecida a como funciona SQL Query, {{site.data.keyword.dynamdashbemb_notm}} puede leer datos directamente de {{site.data.keyword.cos_short}} mediante paneles de control integrados. En esta sección se presenta una solución que permite que cualquier usuario acceda al lago de datos y cree un panel de control personalizado.

1. Acceda al URL público de la aplicación de panel de control que ha enviado a {{site.data.keyword.Bluemix_notm}} anteriormente.
2. Seleccione una plantilla que se ajuste al diseño que desea. (En los pasos siguientes se utiliza el segundo diseño de la primera fila.)
3. Utilice el botón `Añadir un origen` que aparezca en el lateral `Orígenes seleccionadas`, amplíe el acordeón `nombre de grupo` y pulse en una de las entradas de la tabla `accidents/jobid=...`. Cierre el diálogo con el icono X de la parte superior derecha.
4. En la izquierda, pulse el icono `Visualizaciones` y luego pulse **Resumen**.
5. Seleccione el origen `accidents/jobid=...`, amplíe la `Tabla` y cree un gráfico.
    - Arrastre y suelte `id` en la fila **Valor**.
    - Contraiga el gráfico con el icono de la esquina superior.
6. De nuevo desde `Visualizaciones`, cree un gráfico de **Correlación de árbol**:
    - Arrastre y suelte `área` en la fila **Jerarquía de áreas**.
    - Arrastre y suelte `id` en la fila **Tamaño**.
    - Contraiga el gráfico para ver el resultado.

![Diagrama del panel de control](images/solution29/dashboard-chart.png)

## Exploración del panel de control

En esta sección, llevará a cabo algunos pasos adicionales para explorar las características de la aplicación del panel de control y de {{site.data.keyword.dynamdashbemb_notm}}.

1. Pulse el botón **Modalidad** en la barra de herramientas de la aplicación de panel de control de ejemplo para cambiar la vista de modalidad `VISTA`.
2. Pulse cualquiera de los mosaicos de colores del gráfico inferior o los valores de `área` de la leyenda del gráfico. Esto aplica un filtro local al separador, lo que hace que el otro gráfico o gráficos muestren datos específicos del filtro.
3. Pulse el botón **Guardar** en la barra de herramientas.
    - Especifique el nombre del panel de control en el campo de entrada correspondiente.
    - Seleccione el separador **Espec** para ver la especificación del panel de control. Una especificación es el formato de archivo nativo de {{site.data.keyword.dynamdashbemb_notm}}. En el mismo encontrará información sobre los gráficos que ha creado, así como sobre el origen de datos de {{site.data.keyword.cos_short}} utilizado.
    - Guarde el panel de control en el almacenamiento local del navegador con el botón **Guardar** del diálogo.
4. Pulse el botón **Nuevo** de la barra de herramientas para crear un nuevo panel de control. Para abrir un panel de control guardado, pulse el botón **Abrir**. Para suprimir un panel de control, utilice el icono **Suprimir** en el diálogo Abrir panel de control.

En aplicaciones de producción, cifre la información como URL, nombres de usuario y contraseñas para evitar que la vean los usuarios finales. Consulte [Cifrado de información de origen de datos](https://{DomainName}/docs/services/cognos-dashboard-embedded?topic=cognos-dashboard-embedded-encryptingdatasourceinformation#encrypting-data-source-information).
{: tip}

## Ampliación de la guía de aprendizaje

Enhorabuena, ha creado un lago de datos mediante {{site.data.keyword.cos_short}}. A continuación encontrará algunas sugerencias adicionales para mejorar el lago de datos.

- Experimente con otros conjuntos de datos mediante SQL Query
- Envíe en modalidad continua datos procedentes de varios orígenes al lago de datos; encontrará cómo hacerlo en [Registros de big data con análisis en modalidad continua y SQL](https://{DomainName}/docs/tutorials?topic=solution-tutorials-big-data-log-analytics#big-data-log-analytics)
- Edite el código de la aplicación del panel de control para guardar especificaciones del panel de control en [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) o {{site.data.keyword.cos_short}}
- Cree una instancia del servicio [{{site.data.keyword.appid_full_notm}}](https://{DomainName}/catalog/services/app-id) para habilitar la seguridad en la aplicación del panel de control

## Eliminación de recursos

Ejecute los mandatos siguientes para eliminar los servicios, las aplicaciones y las claves que haya utilizado.

```sh
ibmcloud resource service-binding-delete dashboard-nodejs-dde dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-binding-delete dashboard-nodejs-cos dashboard-nodejs
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-dde
```
{: pre}
```sh
ibmcloud resource service-alias-delete dashboard-nodejs-cos
```
{: pre}
```sh
ibmcloud iam api-key-delete data-lake-cos-key
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-dde
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-cos
```
{: pre}
```sh
ibmcloud resource service-instance-delete data-lake-sql
```
{: pre}
```sh
ibmcloud service delete data-lake-studio
```
{: pre}
```sh
ibmcloud app delete dashboard-nodejs
```
{: pre}

## Contenido relacionado

- [ibmcloudsql](https://github.com/IBM-Cloud/sql-query-clients/tree/master/Python)
- [Cuadernos Jupyter](http://jupyter.org/)
- [Mapbox](https://{DomainName}/catalog/services/mapbox-maps)
- [PixieDust](https://www.ibm.com/cloud/pixiedust)
