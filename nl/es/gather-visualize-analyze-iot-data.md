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

# Recopilación, visualización y análisis de datos IoT
{: #gather-visualize-analyze-iot-data}
En esta guía de aprendizaje se explica cómo configurar un dispositivo IoT, recopilar datos en {{site.data.keyword.iot_short_notm}}, explorar datos y crear visualizaciones y utilizar los servicios avanzados de aprendizaje automático para analizar datos y detectar anomalías en los datos históricos.
{:shortdesc}

## Objetivos
{: #objectives}

* Configuración de un simulador de IoT.
* Envío de datos recopilados a {{site.data.keyword.iot_short_notm}}.
* Creación de visualizaciones.
* Análisis de los datos generados por el dispositivo y detección de anomalías.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.iot_full}}](https://{DomainName}/catalog/services/internet-of-things-platform)
* [Aplicación Node.js](https://{DomainName}/catalog/starters/sdk-for-nodejs)
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience) con el servicio Spark y {{site.data.keyword.cos_full_notm}}
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)


Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

   ![](images/solution16/Architecture.png)
</p>

* Los dispositivos envían datos del sensor a {{site.data.keyword.iot_full}} mediante el protocolo MQTT
* Los datos históricos se exportan a una base de datos de {{site.data.keyword.cloudant_short_notm}}
* {{site.data.keyword.DSX_short}} extrae datos de esta base de datos
* Los datos se analizan y se visualizan mediante un cuaderno Jupyter

## Antes de empezar
{: #prereqs}

[Herramientas del desarrollador de {{site.data.keyword.Bluemix_notm}}](https://github.com/IBM-Cloud/ibm-cloud-developer-tools): ejecute el script para instalar la cli de ibmcloud y los plugins necesarios

## Creación de una plataforma IoT
{: #iot_starter}

Para empezar, creará el servicio Internet de las cosas: el concentrador que puede gestionar dispositivos, conectarse de forma segura y **recopilar datos**, y hacer que los datos históricos estén disponibles para visualizaciones y aplicaciones.

1. Vaya al [catálogo de **{{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) y seleccione [**Plataforma Internet de las cosas**](https://{DomainName}/catalog/services/internet-of-things-platform) en la sección **Internet de las cosas**.
2. Especifique `IoT demo Hub` como nombre de servicio, pulse **Crear** e **inicie** en el panel de control.
3. En el menú lateral, seleccione **Seguridad > Seguridad de conexión** y elija **TLS Opcional** en **Regla predeterminada** > **Nivel de seguridad** y pulse **Guardar**.
4. En el menú lateral, seleccione **Dispositivos** > **Tipos de dispositivo** y **+ Añadir tipo de dispositivo**.
5. Especifique `simulator` como **Nombre** y pulse **Siguiente** y **Terminado**.
6. A continuación, pulse **Registrar dispositivos**
7. Elija `simulator` para **Seleccionar tipo de dispositivo existente** y, a continuación, especifique `phone` para **ID de dispositivo**.
8. Pulse **Siguiente** hasta que aparezca la pantalla **Seguridad de dispositivo** (en el separador Seguridad).
9. Especifique un valor para la **Señal de autenticación**, como por ejemplo `myauthtoken`, y pulse **Siguiente**.
10. Después de pulsar **Terminado**, se visualizará la información de conexión. Mantenga este separador abierto.

Ahora la plataforma IoT está configurada para empezar a recibir datos. Los dispositivos tendrán que enviar sus datos a la plataforma IoT especificando el tipo de dispositivo, el ID y la señal.

## Creación de un simulador de dispositivo
{: #confignodered}
A continuación, desplegará una aplicación web Node.js y la visitará en su teléfono, con lo que se conectará a la plataforma IoT y le enviará datos de orientación y del acelerómetro del dispositivo.

1. Clone el repositorio Github:
   ```bash
   git clone https://github.com/IBM-Cloud/iot-device-phone-simulator
   cd iot-device-phone-simulator
   ```
2. Abra el código en el IDE que elija y cambie los valores `name` y `host` del archivo **manifest.yml** por un valor exclusivo.
3. Envíe la aplicación a {{site.data.keyword.Bluemix_notm}}.
   ```bash
   ibmcloud login
   ibmcloud target --cf
   ibmcloud cf push
   ```
4. En pocos minutos, la aplicación se desplegará y debería ver un URL parecido a `<UNIQUE_NAME>.mybluemix.net`
5. Visite este URL en su teléfono utilizando un navegador.
6. Especifique la información de conexión del separador Panel de control de IoT en **Credenciales de dispositivo** y pulse **Conectar**.
7. Su teléfono empezará a transmitir datos. De nuevo en el **separador IBM {{site.data.keyword.iot_short_notm}}**, compruebe si hay entradas nuevas en la sección **Sucesos recientes**.
  ![](images/solution16/recent_events_with_phone.png)

## Visualización de datos en directo en IBM {{site.data.keyword.iot_short_notm}}
{: #createcards}
A continuación creará un panel y tarjetas para mostrar datos del dispositivo en el panel de control. Para obtener más información sobre paneles y tarjetas, consulte [Visualización de datos en tiempo real utilizando paneles y tarjetas](https://console.ng.bluemix.net/docs/services/IoT/data_visualization.html).

### Creación de un panel
{: #createboard}

1. Abra el **panel de control de IBM {{site.data.keyword.iot_short_notm}}**.
2. Seleccione **Paneles** en el menú de la izquierda y luego pulse **Crear nuevo panel**.
3. Especifique un nombre para el panel, como por ejemplo `Simuladores`, pulse **Siguiente** y luego **Enviar**.  
4. Seleccione el panel que acaba de crear para abrirlo.

### Visualización de datos del dispositivo
{: #cardtemp}
1. Pulse **Añadir nueva tarjeta** y seleccione el tipo de tarjeta **Gráfico de líneas**, que se encuentra en la sección Dispositivos.
2. Seleccione el dispositivo en la lista y pulse **Siguiente**.
3. Pulse **Conectar nuevo conjunto de datos**.
4. En la página Crear tarjeta de valores, seleccione o escriba los siguientes valores y pulse **Siguiente**.
   - Suceso: sensorData
   - Propiedad: ob
   - Nombre: OrientationBeta
   - Tipo: Float
   - Mínimo: -180
   - Máximo: 180
5. En la página Vista previa de la tarjeta, seleccione **L** para el tamaño del gráfico de líneas y pulse **Siguiente** > **Enviar**
6. La tarjeta aparece en el panel de control e incluye un gráfico de líneas de los datos de temperatura en tiempo real.
7. Utilice el navegador de su teléfono móvil para volver a iniciar el navegador e incline lentamente el teléfono hacia adelante y hacia atrás.
8. De nuevo en el **separador IBM {{site.data.keyword.iot_short_notm}}**, debería ver cómo se actualiza al gráfico.
![](images/solution16/board.png)

## Almacenamiento de datos históricos en {{site.data.keyword.cloudant_short_notm}}
1. Vaya al [catálogo de **{{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) y cree un nuevo [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) llamado `iot-db`.
2. En **Conexiones**:
   1. **Crear conexión**
   1. Seleccione la ubicación, la organización y el espacio de Cloud Foundry donde se debe crear un alias para el servicio {{site.data.keyword.cloudant_short_notm}}.
   1. Expanda el nombre de espacio en la tabla **Ubicación de conexión** y utilice el botón **Conectar** situado junto al **concentrador de demostración de iot** para crear un alias para el servicio {{site.data.keyword.cloudant_short_notm}} en ese espacio. 
   1. Conectar y volver a transferir la app.
3. Abra el **panel de control de IBM {{site.data.keyword.iot_short_notm}}**.
4. Seleccione **Extensiones** en el menú de la izquierda y luego pulse **Configuración** en **Almacenamiento de datos históricos**.
5. Seleccione la base de datos de {{site.data.keyword.cloudant_short_notm}} `iot-db`.
6. Escriba `devicedata` como **Nombre de base de datos** y pulse **Terminado**.
7. Se debería cargar una nueva ventana que solicita autorización. Si no ve esta ventana, inhabilite el bloqueador de ventanas emergentes y renueve la página.

Ahora los datos del dispositivo están guardados en {{site.data.keyword.cloudant_short_notm}}. Después de unos minutos, inicie el panel de control de {{site.data.keyword.cloudant_short_notm}} para ver los datos.

![](images/solution16/cloudant.png)

## Detección de anomalías mediante aprendizaje automático
{: #data_experience}

En esta sección, utilizará Jupyter Notebook que está disponible en el servicio IBM {{site.data.keyword.DSX_short}} para cargar los datos históricos móviles y para detectar anomalías mediante el sistema z-score. El sistema *z-score* es una puntuación estándar que indica cuán desviado está un elemento de la media

### Creación de un nuevo proyecto
1. Vaya al [catálogo de **{{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) y, en **IA**, seleccione [**{{site.data.keyword.DSX_short}}**](https://{DomainName}/catalog/services/data-science-experience).
2. **Cree** el servicio e inicie el panel de control pulsando **Empezar**
3. Cree un proyecto > Seleccione **Ciencia de datos** > Pulse **Crear proyecto** y especifique `Detectar anomalía` como **Nombre** del proyecto.
4. Deje el recuadro de sección **Restringir quién puede ser un colaborador** sin marcar, ya que no hay datos confidenciales.
5. En **Definir almacenamiento**, pulse **Añadir** y elija un servicio **Cloud Object Storage** existente o cree un nuevo (seleccione Plan **Lite** > Crear). Pulse **Renovar** para ver el servicio creado.
6. Pulse **Crear**. El nuevo proyecto se abre y puede empezar a añadirle recursos.

### Conexión a {{site.data.keyword.cloudant_short_notm}} para obtener datos

1. Pulse **Activos** > **+ Añadir a proyecto** > **Conexión**  
2. Seleccione la {{site.data.keyword.cloudant_short_notm}} **iot-db** en la que se han almacenado los datos del dispositivo.
3. Marque las **Credenciales** y pulse **Crear**.

### Creación de un servicio Apache Spark

1. Pulse **Servicios** en la barra de navegación superior > Servicios de cálculo.
2. Pulse **Añadir servicio**.
   1. Pulse **Añadir** en **Apache Spark**.
   1. Seleccione el plan **Lite**.
   1. Pulse **Crear**.
3. Seleccione una organización y un espacio, cambie el nombre del servicio si lo desea y **Confirme**.
1. Vaya al proyecto `Detectar anomalía` a través de **Proyectos**.
1. En **Valores**, desplácese hasta **Servicios asociados**.
1. Pulse **Añadir servicio** y seleccione **Spark**.
1. Seleccione la instancia de **Apache Spark** creada anteriormente.

### Creación de un cuaderno Jupyter (ipynb)
1. Pulse **+ Añadir a proyecto** y añada un nuevo **cuaderno**.
2. Especifique `Anomaly-detection-notebook` como **Nombre**.
3. Especifique `https://github.com/IBM-Cloud/iot-device-phone-simulator/raw/master/anomaly-detection/Anomaly-detection-watson-studio-python3.ipynb` en el campo **URL de cuaderno**.
4. Seleccione el servicio **Apache Spark** asociado anteriormente como tiempo de ejecución.
5. Cree el **cuaderno**. Establezca `Python 3.5 con Spark 2.1` como Kernel. Compruebe que el cuaderno se ha creado con metadatos y código.
   ![Jupyter Notebook Watson Studio](images/solution16/jupyter_notebook_watson_studio.png)
   Para actualizar, utilice **Kernel** > Cambiar kernel. Para que el cuaderno sea **fiable**, pulse **Archivo** > Cuaderno fiable.
   {:tip}

### Ejecución del cuaderno y detección de anomalías   
1. Seleccione la célula que empieza por `!pip install --upgrade pixiedust,` y pulse **Ejecutar** o **Control + Intro** para ejecutar el código.
2. Cuando finalice la instalación, reinicie el kernel Spark pulsando el icono **Reiniciar Kernel**.
3. En la siguiente célula de código, importe las credenciales de {{site.data.keyword.cloudant_short_notm}} en dicha célula siguiendo los pasos siguientes:
   * Pulse ![](images/solution16/data_icon.png)
   * Seleccione el separador **Conexiones**.
   * Pulse **Insertar para codificar**. Se crea un diccionario llamado _credentials_1_ con sus credenciales de {{site.data.keyword.cloudant_short_notm}}. Si no se especifica el nombre como _credentials_1_, cambie el nombre del diccionario por `credentials_1`. `credentials_1` se utiliza en las otras células.
4. En la célula con el nombre de la base de datos (`dbName`), escriba el nombre de la base de datos de {{site.data.keyword.cloudant_short_notm}} que es el origen de los datos, por ejemplo *iotp_yourWatsonIoTProgId_DBName_Year-month-day*. Para visualizar datos de distintos dispositivos, cambie los valores de `deviceId` y `deviceType` en consecuencia.
   Para saber la base de datos exacta, vaya a la instancia de {{site.data.keyword.cloudant_short_notm}} **iot-db** que ha creado antes e inicie el panel de control.
   {:tip}
5. Guarde el cuaderno y ejecute cada célula de código una tras otra o bien ejecute todo (**Célula** > Ejecutar todo) y al final del cuaderno debería ver las anomalías correspondientes a los datos de movimiento de dispositivos (oa, ob y og).
   Puede cambiar el intervalo de tiempo por la hora deseada del día. Busque los valores `start` y `end`.
   {:tip}
   ![DSX de Jupyter Notebook](images/solution16/anomaly_detection_watson_studio.png)
6. Junto con la detección de anomalías, en esta sección se ha mostrado lo siguiente
    * Uso de Spark para preparar los datos para su visualización.
    * Uso de Pandas para la visualización de datos
    * Gráficos de barras, histogramas para datos de dispositivo.
    * Correlación entre dos sensores mediante una matriz de correlaciones.
    * Un diagrama de caja para cada sensor de dispositivo, generado con la función de trazo de Pandas.
    * Diagramas de densidad mediante la estimación de densidad del kernel (KDE).     ![](images/solution16/density_plots_sensor_data.png)

## Eliminación de recursos
{:removeresources}

1. Vaya a [Lista de recurso](https://{DomainName}/resources/) > elija la ubicación, la organización y el espacio donde ha creado la app y los servicios. En **Apps de Cloud Foundry**, suprima la app Node.JS que ha creado anteriormente.
2. En **Servicios**, suprima la plataforma Internet de las cosas respectiva, Apache Spark y los servicios {{site.data.keyword.cloudant_short_notm}} y {{site.data.keyword.cos_full_notm}} que ha creado para esta guía de aprendizaje.

## Contenido relacionado
{:related}

* [Creación, despliegue, prueba y reentrenamiento de un modelo predictivo de aprendizaje automático](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-deploy-retrain-machine-learning-model#build-deploy-test-and-retrain-a-predictive-machine-learning-model)
* Visión general de [IBM {{site.data.keyword.DSX_short}}](https://dataplatform.cloud.ibm.com/docs/content/wsj/getting-started/overview-ws.html)
* Detección de anomalías, [Jupyter Notebook](https://github.com/IBM-Cloud/iot-device-phone-simulator/blob/master/anomaly-detection/Anomaly-detection-DSX.ipynb)
* [Visión general de z-score](https://en.wikipedia.org/wiki/Standard_score)
* Desarrollo de soluciones IoT cognitivas para la detección de anomalías mediante Deep Learning, [serie de 5 posts](https://developer.ibm.com/series/iot-anomaly-detection-deep-learning/)
