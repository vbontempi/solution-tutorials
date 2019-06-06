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

# Registros de Big Data con análisis continuo y SQL
{: #big-data-log-analytics}

En esta guía de aprendizaje, creará un conducto de análisis de registros diseñado para recopilar, almacenar y analizar registros para dar soporte a los requisitos normativos y al descubrimiento de información de ayuda. Esta solución aprovecha varios servicios disponibles en {{site.data.keyword.cloud_notm}}: {{site.data.keyword.messagehub}}, {{site.data.keyword.cos_short}}, SQL Query y {{site.data.keyword.streaminganalyticsshort}}. Un programa le ayudará mediante la simulación de una transmisión de mensajes de registro del servidor web de un archivo estático a {{site.data.keyword.messagehub}}.

Con {{site.data.keyword.messagehub}}, el conducto se puede escalar para que reciba millones de registros procedentes de diversos productores. Mediante la aplicación de {{site.data.keyword.streaminganalyticsshort}}, los datos de registro se pueden inspeccionar en tiempo real para integrar los procesos de negocio. Los mensajes de registro también se pueden redirigir fácilmente a almacenamiento a largo plazo utilizando {{site.data.keyword.cos_short}}, donde los desarrolladores, el personal de soporte y los auditores pueden trabajar directamente con los datos mediante SQL Query.

Aunque esta guía de aprendizaje se centra en el análisis de registros, se puede aplicar a otros escenarios: los dispositivos IoT limitados por almacenamiento pueden transmitir mensajes de forma similar a {{site.data.keyword.cos_short}} o los profesionales de marketing pueden segmentar y analizar sucesos de clientes de las propiedades digitales con SQL Query.
{:shortdesc}

## Objetivos

{: #objectives}

* Comprensión de la mensajería de tipo publicación y suscripción de Apache Kafka
* Almacenamiento de los datos de registro para cumplir con los requisitos de auditoría y conformidad
* Supervisión de registros para crear procesos de manejo de excepciones
* Realización de análisis forenses y estadísticos sobre los datos de registro

## Servicios utilizados

{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:

* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/message-hub)
* [{{site.data.keyword.sqlquery_short}}](https://{DomainName}/catalog/services/sql-query)
* [{{site.data.keyword.streaminganalyticsshort}}](https://{DomainName}/catalog/services/streaming-analytics)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura

{: #architecture}

<p style="text-align: center;">

  ![Arquitectura](images/solution31/Architecture.png)
</p>

1. La aplicación genera sucesos de registro en {{site.data.keyword.messagehub}}
2. {{site.data.keyword.streaminganalyticsshort}} intercepta y analiza el suceso de registro
3. El suceso de registro se añade a un archivo CSV ubicado en {{site.data.keyword.cos_short}}
4. El auditor o el personal de soporte emite un trabajo SQL
5. Se ejecuta SQL Query sobre el archivo de registro en {{site.data.keyword.cos_short}}
6. El conjunto de resultados se guarda en {{site.data.keyword.cos_short}} y se distribuye al auditor y al personal de soporte

## Antes de empezar

{: #prereqs}

* [Instale Git](https://git-scm.com/)
* [Instale la CLI de {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* [Instale Node.js](https://nodejs.org)
* [Descargue el cliente Kafka 0.10.2.X](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz)

## Creación de servicios

{: #setup}

En esta sección, creará los servicios necesarios para realizar el análisis de los sucesos de registro generados por las aplicaciones.

En esta sección se utiliza la línea de mandatos para crear instancias de servicio. Si lo desea, puede hacer lo mismo desde la página del servicio en el catálogo utilizando los enlaces que se proporcionan.
{: tip}

1. Inicie una sesión en {{site.data.keyword.cloud_notm}} mediante la línea de mandatos y seleccione la cuenta de Cloud Foundry. Consulte [Iniciación a la CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh
    ibmcloud login
    ```
    {: pre}
    ```sh
    ibmcloud target --cf
    ```
    {: pre}
2. Cree una instancia Lite de [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
    ```sh
    ibmcloud resource service-instance-create log-analysis-cos cloud-object-storage \
    lite global
    ```
    {: pre}
3. Cree una instancia Lite de [SQL Query](https://{DomainName}/catalog/services/sql-query).
    ```sh
    ibmcloud resource service-instance-create log-analysis-sql sql-query lite \
    us-south
    ```
    {: pre}
4. Cree una instancia Estándar de [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams).
    ```sh
    ibmcloud service create messagehub standard log-analysis-hub
    ```
    {: pre}

## Creación de un tema de mensajería y de un grupo de {{site.data.keyword.cos_short}}

{: #topics}

Comience por crear un tema de {{site.data.keyword.messagehub}} y un grupo de {{site.data.keyword.cos_short}}. Los temas definen dónde las aplicaciones entregan mensajes en los sistemas de mensajería de publicación y suscripción. Después de que los mensajes se reciban y se procesen, se guardan en un archivo ubicado en un grupo de {{site.data.keyword.cos_short}}.

1. En el navegador, acceda a la instancia de servicio `log-analysis-hub` desde la [Recursos](https://{DomainName}/resources?search=log-analysis).
2. Pulse el botón **+** para crear un tema.
3. Especifique el **Nombre de tema** `servidor web` y pulse el botón **Crear tema**.
4. Pulse **Credenciales de servicio** y el botón **Nueva credencial**.
5. En el diálogo resultante, escriba `webserver-flow` como **Nombre** y pulse el botón **Añadir**.
6. Pulse **Ver credenciales** y copie la información en un lugar seguro. Se utilizará en la sección siguiente.
7. De nuevo en la [Lista de recursos](https://{DomainName}/resources?search=log-analysis), seleccione la instancia de servicio `log-analysis-cos`.
8. Pulse **Crear grupo**.
    * Especifique un **Nombre** exclusivo para el grupo.
    * Seleccione **Varias regiones** para **Resistencia**.
    * Seleccione **us-geo** como **Ubicación**.
    * Pulse **Crear grupo**.

## Creación de un origen de flujo Streams

{: #streamsflow}

En esta sección, empezará a configurar un flujo Streams que recibe mensajes de registro. El servicio {{site.data.keyword.streaminganalyticsshort}} recibe soporte de {{site.data.keyword.streamsshort}}, que puede analizar millones de sucesos por segundo y permite tiempos de respuesta por debajo del milisegundo y permite una toma instantánea de decisiones.

1. En el navegador, acceda a [Watson Data Platform](https://dataplatform.ibm.com).
2. Seleccione el botón o mosaico **Nuevo proyecto** y luego el mosaico **Básico** y pulse **Aceptar**.
    * Especifique el **Nombre** `webserver-logs`.
    * La opción **Almacenamiento** debería estar establecida en `log-analysis-cos`. Si no es así, seleccione la instancia de servicio.
    * Pulse el botón **Crear**.
3. En la página resultante, seleccione el separador **Valores** y marque **Streams Designer** en **Herramientas**. Para terminar pulse el botón **Guardar**.
4. Pulse el botón **Añadir al proyecto** y luego **Flujo Streams** en la barra de navegación superior.
    * Pulse **Asociar una instancia de IBM Streaming Analytics con un plan basado en contenedor**.
    * Cree una nueva instancia de {{site.data.keyword.streaminganalyticsshort}} seleccionando el botón **Lite** y pulsando **Crear**. No seleccione VM Lite.
    * Especifique el **Nombre de servicio** `log-analysis-sa` y pulse **Confirmar**.
    * Escriba el **Nombre** del flujo de Streams `webserver-flow`.
    * Para terminar, pulse **Crear**.
5. En la página resultante, seleccione el mosaico **{{site.data.keyword.messagehub}}**.
    * Pulse **Añadir conexión** y seleccione la instancia de {{site.data.keyword.messagehub}} `log-analysis-hub`. Si no ve la instancia en la lista, seleccione la opción **IBM {{site.data.keyword.messagehub}}**. Especifique manualmente los **Detalles de conexión** que ha obtenido de las **Credenciales de servicio** en la sección anterior. Asigne a la conexión el **Nombre** `webserver-flow`.
    * Pulse **Crear** para crear la conexión.
    * Seleccione `webserver` en la lista desplegable **Tema**.
    * Seleccione **Empezar con el primer mensaje nuevo** en el desplegable **Desplazamiento inicial**.
    * Pulse **Continuar**.
6. Deje la página **Vista previa de los datos** abierta; se utilizará en la siguiente sección.

## Utilización de las herramientas de la consola de Kafka con {{site.data.keyword.messagehub}}

{: #kafkatools}

En este momento, `webserver-flow` está desocupado y a la espera de mensajes. En esta sección, configurará las herramientas de la consola Kafka para trabajar con {{site.data.keyword.messagehub}}. Las herramientas de la consola Kafka le permiten generar mensajes arbitrarios desde el terminal y enviarlos a {{site.data.keyword.messagehub}}, lo que desencadenará `webserver-flow`.

1. Descargue y descomprima el [cliente Kafka 0.10.2.X](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.2.1/kafka_2.11-0.10.2.1.tgz).
2. Vaya al directorio `bin` y cree un archivo de texto denominado `message-hub.config` con el contenido siguiente.
    ```sh
    sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="USER" password="PASSWORD";
    security.protocol=SASL_SSL
    sasl.mechanism=PLAIN
    ssl.protocol=TLSv1.2
    ssl.enabled.protocols=TLSv1.2
    ssl.endpoint.identification.algorithm=HTTPS
    ```
    {: pre}
3. Sustituya `USER` y `PASSWORD` en el archivo `message-hub.config` por sus valores de `user` y `password` de las **Credenciales de servicio** de la sección anterior. Guarde `message-hub.config`.
4. En el directorio `bin`, ejecute el siguiente mandato. Sustituya `KAFKA_BROKERS_SASL` por el valor de `kafka_brokers_sasl` que aparece en las **Credenciales de servicio**. A continuación se muestra un ejemplo.
    ```sh
    ./kafka-console-producer.sh --broker-list KAFKA_BROKERS_SASL \
    --producer.config message-hub.config --topic webserver
    ```
    {: pre}
    ```sh
    ./kafka-console-producer.sh --broker-list \
    kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093 \
    --producer.config message-hub.config --topic webserver
    ```
5. La herramienta de la consola de Kafka está esperando información de entrada. Copie y pegue el mensaje de registro de abajo en el terminal. Pulse `Intro` para enviar el mensaje de registro a {{site.data.keyword.messagehub}}. Tenga en cuenta que los mensajes enviados también se muestran en la página de **Vista previa de los datos** de `webserver-flow`.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
![Página Vista previa](images/solution31/preview_data.png)

## Creación de un destino de flujo Streams

{: #streamstarget}

En esta sección, completará la configuración de flujo de Streams definiendo un destino. El destino se utilizará para almacenar los mensajes de registro de entrada en el grupo {{site.data.keyword.cos_short}} creado anteriormente. El proceso de almacenar y añadir los mensajes de registro de entrada a un archivo se efectuará automáticamente mediante {{site.data.keyword.streaminganalyticsshort}}.

1. En la **Página de vista previa** de `webserver-flow`, pulse el botón **Continuar**.
2. Seleccione el mosaico **{{site.data.keyword.cos_full_notm}}** como destino.
    * Pulse **Añadir conexión** y seleccione `log-analysis-cos`.
    * Pulse **Crear**.
    * Especifique la **Vía de acceso a archivo** `/YOUR_BUCKET_NAME/http-logs_%TIME.csv`. Sustituya `YOUR_BUCKET_NAME` por el nombre utilizado en la primera sección.
    * Seleccione **csv** en el menú desplegable **Formato**.
    * Marque el recuadro de selección **Fila de cabecera de columna**.
    * Seleccione **Tamaño de archivo** en el menú desplegable **Política de creación de archivos**.
    * Establezca el límite en 100 MB especificando `102400` en el recuadro de texto **Tamaño de archivo (KB)**.
    * Pulse **Continuar**.
3. Pulse **Guardar**.
4. Pulse el botón de reproducción **>** para **Iniciar el flujo de Streams**.
5. Una vez iniciado el flujo, vuelva a enviar varios mensajes de registro desde la herramienta de la consola de Kafka. Puede ver cómo llegan los mensajes visualizando `webserver-flow` en Streams Designer.
    ```javascript
    { "host": "199.72.81.55", "timestamp": "01/Jul/1995:00:00:01 -0400", "request": "GET /history/apollo/ HTTP/1.0", "responseCode": 200, "bytes": 6245 }
    ```
    {: pre}
6. Vuelva al grupo en {{site.data.keyword.cos_short}}. Aparecerá un nuevo archivo `log.csv` después de que entren en el flujo suficientes mensajes o de que se reinicie el flujo.

![webserver-flow](images/solution31/flow.png)

## Adición de comportamiento condicional a flujos de Streams

{: #streamslogic}

Hasta ahora, el flujo de Streams es un conducto sencillo: mueve mensajes de {{site.data.keyword.messagehub}} a {{site.data.keyword.cos_short}}. Es más que probable que los equipos deseen conocer los sucesos que les interesan en tiempo real. Por ejemplo, algunos equipos pueden aprovechar las alertas cuando se producen sucesos HTTP 500 (error de la aplicación). En esta sección, añadirá lógica condicional al flujo para identificar los códigos HTTP 200 (OK) y no HTTP 200.

1. Utilice el botón de lápiz para **Editar el flujo de Streams**.
2. Cree un nodo de filtro que gestione las respuestas HTTP 200.
    * En la paleta **Nodos**, arrastre el nodo **Filtro** de **PROCESO Y ANÁLISIS** al lienzo.
    * Escriba `OK` en el recuadro de texto de nombre, que actualmente contiene la palabra `Filtro`.
    * Escriba la siguiente sentencia en el área de texto **Expresión de la condición**.
      ```sh
      responseCode == 200
      ```
      {: pre}
    * Con el ratón, trace una línea desde la salida del nodo **{{site.data.keyword.messagehub}}** (lado derecho) hasta la entrada del nodo **OK** (lado izquierdo).
    * En la paleta **Nodos**, arrastre el nodo **Depurar** que se encuentra bajo **DESTINOS** al lienzo.
    * Conecte el nodo **Depurar** con el nodo **OK** dibujando una línea entre los dos.
3. Repita el proceso para crear un filtro `Not OK` utilizando los mismos nodos y la sentencia de condición siguiente.
    ```sh
    responseCode >= 300
    ```
    {: pre}
4. Pulse el botón de reproducción para **Guardar y ejecutar el flujo de Streams**.
5. Si se le solicita, pulse el enlace para **ejecutar la nueva versión**.

![Diseñador de flujos](images/solution31/flow_design.png)

## Aumento de la carga de mensajes

{: #streamsload}

Para ver el manejo condicional en el flujo de Streams, aumentará el volumen de mensajes que se envían a {{site.data.keyword.messagehub}}. El programa Node.js proporcionado simula un flujo de mensajes realista destinado a {{site.data.keyword.messagehub}} basado en el tráfico en del servidor web. Para demostrar la escalabilidad de {{site.data.keyword.messagehub}} y de {{site.data.keyword.streaminganalyticsshort}}, incrementará el rendimiento de los mensajes de registro.

En esta sección se utiliza [node-rdkafka](https://www.npmjs.com/package/node-rdkafka). Consulte la página npmjs para obtener instrucciones sobre resolución de problemas si falla la instalación del simulador. Si los problemas persisten, puede saltar a la siguiente sección y cargar manualmente los datos.

1. Descargue y descomprima el archivo de registro [Jul 01 to Jul 31, ASCII format, 20.7 MB gzip compressed](http://ita.ee.lbl.gov/traces/NASA_access_log_Aug95.gz) de la NASA.
2. Clone e instale el simulador de registro de [IBM-Cloud on GitHub](https://github.com/IBM-Cloud/kafka-log-simulator).
    ```sh
    git clone https://github.com/IBM-Cloud/kafka-log-simulator.git
    ```
    {: pre}
3. Cambie al directorio del simulador y ejecute los mandatos siguientes para configurar el simulador y generar mensajes de sucesos de registro. Sustituya `LOGFILE` por el archivo que ha descargado. Sustituya `BROKERLIST` y `APIKEY` por las **Credenciales de servicio** correspondientes utilizadas anteriormente. A continuación se muestra un ejemplo.
    ```sh
    npm install
    ```
    ```sh
    npm run build
    ```
    ```sh
    node dist/index.js --file LOGFILE --parser httpd --broker-list BROKERLIST \
    --api-key APIKEY --topic webserver --rate 100
    ```
    {: pre}
    ```sh
    node dist/index.js --file /Users/ibmcloud/Downloads/NASA_access_log_Jul95 \
    --parser httpd --broker-list \
    "kafka04-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka05-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka02-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka01-prod02.messagehub.services.us-south.bluemix.net:9093,\
    kafka03-prod02.messagehub.services.us-south.bluemix.net:9093" \
    --api-key Np15YZKN3SCdABUsOpJYtpue6jgJ7CwYgsoCWaPbuyFbdM4R \
    --topic webserver --rate 100
    ```
4. En el navegador, vuelva a `webserver-flow` después de que el simulador empiece a generar mensajes.
5. Detenga el simulador después de que hayan pasado por las ramas condicionales el número deseado de mensajes mediante `control+C`.
6. Experimente con el escalado de {{site.data.keyword.messagehub}} aumentando o reduciendo el valor de `--rage`.

El simulador retrasará el envío del siguiente mensaje en función del tiempo transcurrido en el registro del servidor web. Si se establece `--rate 1`, los sucesos se envían en tiempo real. Si se establece `--rate 100`, significa que por cada segundo de tiempo transcurrido en el registro del servidor web se utiliza un retardo de 10 ms entre mensajes.
{: tip}

![Carga de flujo establecida en 10](images/solution31/flow_load_10.png)

## Investigación de datos de registro mediante SQL Query

{: #sqlquery}

En función del número de mensajes que envíe el simulador, el archivo de registro de {{site.data.keyword.cos_short}} puede alcanzar un tamaño considerable. Ahora actuará como un investigador que responde a las preguntas de auditoría o de conformidad mediante la combinación de SQL Query con el archivo de registro. La ventaja de utilizar SQL Query es que se puede acceder directamente al archivo de registro; no es necesario realizar ninguna transformación adicional ni se necesita más servidores de bases de datos.

Si prefiere no esperar a que el simulador envíe todos los mensajes de registro, cargue el [archivo CSV completo](https://ibm.box.com/s/dycyvojotfpqvumutehdwvp1o0fptwsp) en {{site.data.keyword.cos_short}} para comenzar a trabajar de inmediato.
{: tip}

1. Acceda a la instancia de servicio `log-analysis-sql` desde la [Lista de recursos](https://{DomainName}/resources?search=log-analysis). Seleccione **Abrir IU** para iniciar SQL Query.
2. Escriba las siguientes sentencias SQL en el área de texto **Escribir SQL aquí...**.
    ```sql
    -- What are the top 10 web pages on NASA from July 1995?
    -- Which mission might be significant?
    SELECT REQUEST, COUNT(REQUEST)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE REQUEST LIKE '%.htm%'
    GROUP BY REQUEST
    ORDER BY 2 DESC
    LIMIT 10
    ```
    {: pre}
3. Recupere el URL de Object SQL del archivo de registros.
    * En la [Lista de recursos](https://{DomainName}/resources?search=log-analysis), seleccione la instancia de servicio `log-analysis-cos`.
    * Seleccione el grupo que ha creado previamente.
    * Pulse el menú de desbordamiento en el archivo `http-logs_TIME.csv` y seleccione **URL de Object SQL**.
    * **Copie** el URL en el portapapeles.
4. Actualice la cláusula `FROM` con el URL de Object SQL y pulse **Ejecutar**.
5. Podrá ver el resultado en el separador **Resultado**. Se esperan algunas páginas, como la página de inicio de Kennedy Space Center, ya que en este momento hay una misión que genera una gran expectativa.
6. Seleccione el separador **Detalles de la consulta** para ver información adicional, como la ubicación en la que se ha almacenado el resultado en {{site.data.keyword.cos_short}}.
7. Intente los siguientes pares de preguntas y respuestas añadiéndolos individualmente al área de texto **Escribir SQL aquí...**.
    ```sql
    -- Who are the top 5 viewers?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY HOST
    ORDER BY 2 DESC
    LIMIT 5
    ```
    {: pre}

    ```sql
    -- Which viewer has suspicious activity based on application failures?
    SELECT HOST, COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 500
    GROUP BY HOST
    ORDER BY 2 DESC;
    ```
    {: pre}

    ```sql
    -- Which requests showed a page not found error to the user?
    SELECT DISTINCT REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE `responseCode` == 404
    ```
    {: pre}

    ```sql
    -- What are the top 10 largest files?
    SELECT DISTINCT REQUEST, BYTES
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE BYTES > 0
    ORDER BY CAST(BYTES as Integer) DESC
    LIMIT 10
    ```
    {: pre}

    ```sql
    -- What is the distribution of total traffic by hour?
    SELECT SUBSTRING(TIMESTAMP, 13, 2), COUNT(*)
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    GROUP BY 1
    ORDER BY 1 ASC
    ```
    {: pre}

    ```sql
    -- Why did the previous result return an empty hour?
    -- Hint, find the malformed hostname.
    SELECT HOST, REQUEST
    FROM cos://us-geo/YOUR_BUCKET_NAME/http-logs_TIME.csv
    WHERE SUBSTRING(TIMESTAMP, 13, 2) == ''
    ```
    {: pre}

Las cláusulas FROM no están limitadas a un solo archivo. Utilice `cos://us-geo/YOUR_BUCKET_NAME/` para ejecutar consultas SQL sobre todos los archivos del grupo.
{: tip}

## Ampliación de la guía de aprendizaje

{: #expand}

Enhorabuena, ha creado un conducto de análisis de registros con {{site.data.keyword.cloud_notm}}. A continuación encontrará algunas sugerencias para mejorar la solución.

* Utilice destinos adicionales en Streams Designer para almacenar datos en [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) o para ejecutar código en [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* Siga la guía de aprendizaje [Creación de un lago de datos mediante Object Storage](https://{DomainName}/docs/tutorials?topic=solution-tutorials-smart-data-lake#build-a-data-lake-using-object-storage) para añadir un panel de control a los datos de registro
* Integre sistemas adicionales con {{site.data.keyword.messagehub}} mediante [{{site.data.keyword.appconserviceshort}}](https://{DomainName}/catalog/services/app-connect).

## Eliminación de servicios

{: #removal}

En la [Lista de recursos](https://{DomainName}/resources?search=log-analysis), utilice el elemento de menú **Suprimir** o **Suprimir servicio** del menú de desbordamiento para eliminar las instancias de servicio siguientes.

* log-analysis-sa
* log-analysis-hub
* log-analysis-sql
* log-analysis-cos

## Contenido relacionado

{:related}

* [Apache Kafka](https://kafka.apache.org/)
