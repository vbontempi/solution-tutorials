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

# Creación, despliegue, prueba y reentrenamiento de un modelo predictivo de aprendizaje automático
{: #create-deploy-retrain-machine-learning-model}
Esta guía de aprendizaje le guiará por el proceso de creación de un modelo predictivo de aprendizaje automático, de desplegarlo como una API que se utilizará en las aplicaciones, de probar el modelo y de volverlo a entrenar con datos de comentarios. Todo esto se produce en IBM Cloud como una experiencia de autoservicio integrada y unificada.

En esta guía de aprendizaje, se utiliza un **conjunto de datos sobre la flor iris** para crear un modelo de aprendizaje automático para clasificar las especies de esta flor.

En la terminología del aprendizaje automático, la clasificación se considera una instancia de aprendizaje supervisado, es decir, aprendizaje en el que se encuentra disponible un conjunto de datos de entrenamiento con observaciones correctamente identificadas.
{:tip}

{:shortdesc}

<p style="text-align: center;">
  ![](images/solution22-build-machine-learning-model/architecture_diagram.png)
</p>

## Objetivos
{: #objectives}

* Importación de datos en un proyecto.
* Creación de un modelo de aprendizaje automático.
* Despliegue del modelo y prueba de la API.
* Prueba de un modelo de aprendizaje automático.
* Creación de una conexión de datos de comentarios para el aprendizaje continuo y la evaluación del modelo.
* Reentrenamiento del modelo.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience)
* [{{site.data.keyword.sparkl}}](https://{DomainName}/catalog/services/apache-spark)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage)
* [{{site.data.keyword.pm_full}}](https://{DomainName}/catalog/services/machine-learning)
* [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)

## Antes de empezar
{: #prereqs}
* Las aplicaciones IBM Watson Studio y Watson Knowledge Catalog forman parte de IBM Watson. Para crear una cuenta de IBM Watson, empiece por registrarse para una o ambas de estas aplicaciones.

   Vaya a [Probar IBM Watson](https://dataplatform.ibm.com/registration/stepone) y regístrese para las apps de IBM Watson.

## Importación de datos en un proyecto

{:#import_data_project}

Un proyecto es la forma en que se organizan los recursos para alcanzar un determinado objetivo. Los recursos del proyecto pueden incluir datos, colaboradores y herramientas de análisis, como por ejemplo cuadernos de Jupyter y modelos de aprendizaje automático.

Puede crear un proyecto para añadir datos y abrir un activo de datos en el pulidor de datos para limpiar y adaptar los datos.

**Cree un proyecto:**

1. Vaya al [catálogo de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/catalog) y seleccione [{{site.data.keyword.DSX_short}}](https://{DomainName}/catalog/services/data-science-experience?taxonomyNavigation=app-services) en la sección **IA**. **Cree** el servicio. Pulse el botón **Iniciación** para iniciar el panel de control de **{{site.data.keyword.DSX_short}}**.
2. Cree un **proyecto** > Pulse **Crear proyecto** en el mosaico **Estándar**. Añada un nombre, como por ejemplo `iris_project`, una descripción opcional para el proyecto.
3. Deje el recuadro de sección **Restringir quién puede ser un colaborador** sin marcar, ya que no hay datos confidenciales.
4. En **Definir almacenamiento**, pulse **Añadir** y elija un servicio Cloud Object Storage existente o cree un nuevo (seleccione Plan **Lite** > Crear). Pulse **Renovar** para ver el servicio creado.
5. Pulse **Crear**. El nuevo proyecto se abre y puede empezar a añadirle recursos.

**Importe datos:**

Tal como se ha mencionado anteriormente, utilizará el **conjunto de datos Iris**. El conjunto de datos Iris es el utilizado en un artículo clásico de 1936 de R.A. Fisher, [The Use of Multiple Measurements in Taxonomic Problems](http://rcs.chemometrics.ru/Tutorials/classification/Fisher.pdf), y también lo encontrará en el [repositorio de UCI Machine Learning](http://archive.ics.uci.edu/ml/). Este pequeño conjunto de datos se suele utilizar para probar los algoritmos de aprendizaje automático y las visualizaciones. El objetivo es clasificar las flores de iris entre las tres especies (Setosa, Versicolor o Virginica) a partir de mediciones de longitud y anchura de los sépalos y pétalos. El conjunto de datos Iris contiene 3 clases de 50 instancias cada una, en las que cada clase corresponde a un tipo de planta iris. ![](images/solution22-build-machine-learning-model/iris_machinelearning.png)


**Descargue** [iris_initial.csv](https://ibm.box.com/shared/static/nnxx7ozfvpdkjv17x4katwu385cm6k5d.csv), que consta de 40 instancias de cada clase. Utilizará las otras 10 instancias de cada clase para volver a entrenar el modelo.

1. En el proyecto, bajo **Activos** pulse el icono **Buscar y añadir datos** ![Muestra el icono Buscar datos.](images/solution22-build-machine-learning-model/data_icon.png).
2. En **Cargar**, pulse **examinar** y cargue el archivo `iris_initial.csv` descargado.
      ![](images/solution22-build-machine-learning-model/find_and_add_data.png)
3. Una vez añadido, debe ver `iris_initial.csv` en la sección **Activos de datos** del proyecto. Pulse el nombre para ver el contenido del conjunto de datos.

## Servicios asociados
{:#associate_services}
1. En **Valores**, desplácese hasta **Servicios asociados** > pulse **Añadir servicio** > seleccione **Spark**.    ![](images/solution22-build-machine-learning-model/associate_services.png)
2. Seleccione el plan **Lite** y pulse **Crear**. Utilice los valores predeterminados y pulse **Confirmar**.
3. Vuelva a pulsar **Añadir servicio** y elija **Watson**. Pulse **Añadir** en el mosaico **Aprendizaje automático** > seleccione el plan **Lite** > pulse **Crear**.
4. Deje los valores predeterminados y pulse **Confirmar** para suministrar un servicio de aprendizaje automático.

## Creación de un modelo de aprendizaje automático

{:#build_model}

1. Pulse **Añadir a proyecto** y seleccione **Modelo de aprendizaje automático de Watson**. En el diálogo, añada **iris_model** como nombre y una descripción opcional.
2. En la sección **Servicio de aprendizaje automático**, debería ver el servicio de aprendizaje automático que ha asociado en el paso anterior.    ![](images/solution22-build-machine-learning-model/machine_learning_model_creation.png)
3. Seleccione **Creador de modelos** como tipo de modelo y en la sección **Servicio de Spark o entorno**, seleccione el servicio spark que ha creado anteriormente
4. Seleccione **Manual** para crear un modelo de forma manual. Pulse **Crear**.

   Para el método automático, se basará por completo en la preparación automática de datos (ADP). Para el método manual, además de algunas funciones gestionadas por el transformador de ADP, puede añadir y configurar sus propios estimadores, que son los algoritmos que se utilizan en el análisis.
   {:tip}

5. En la página siguiente, seleccione `iris_initial.csv` como conjunto de datos y pulse **Siguiente**.
6. En la página **Seleccionar una técnica**, en función del conjunto de datos que se haya añadido, las columnas de etiqueta y de característica ya están cumplimentadas. Seleccione **species (Serie)** como **Col de etiqueta** y **petal_length (Decimal)** y **petal_width (Decimal)** como **Columnas de característica**.
7. Seleccione **Clasificación multiclase** como técnica recomendada.
   ![](images/solution22-build-machine-learning-model/model_technique.png)
8. Para **División de validación**, configure el valor siguiente:

   **Entrenamiento:** 50%,
   **Prueba:** 25%,
   **Reserva:** 25%

9. Pulse **Añadir estimadores**, seleccione **Clasificador de árboles de decisión** y a continuación seleccione **Añadir**.

   Puede evaluar varios estimadores de una sola vez. Por ejemplo, puede añadir **Clasificador de árboles de decisión** y **Clasificador de bosque aleatorio** como estimadores para entrenar el modelo y elegir el mejor ajuste en función de la información resultante de evaluación.
   {:tip}

10. Pulse **Siguiente** para entrenar el modelo. Cuando vea el estado **Entrenado y evaluado**, pulse **Guardar**.
   ![](images/solution22-build-machine-learning-model/trained_model.png)

11. Pulse **Visión general** para comprobar los detalles del modelo.

## Despliegue del modelo y prueba de la API

{:#deploy_model}

1. En el modelo creado, pulse **Despliegues** > **Añadir despliegue**.
2. Elija **Servicio web**. Añada un nombre, como por ejemplo `iris_deployment`, y una descripción opcional.
3. Pulse **Guardar**. En la página de visión general, pulse el nombre del nuevo servicio web. Cuando el estado sea **DEPLOY_SUCCESS**,
puede comprobar el punto final de puntuación, los fragmentos de código en varios lenguajes de programación y la especificación de API bajo **Implementación**.
4. Pulse **Ver especificación de API** para ver y probar los puntos finales de la API de {{site.data.keyword.pm_short}}.
![](images/solution22-build-machine-learning-model/machine_learning_api.png)

   Para empezar a trabajar con la API, tiene que generar una **señal de acceso** mediante los valores **username** y **password** disponibles en el separador **Credenciales de servicio** de la instancia de servicio de {{site.data.keyword.pm_short}} en la [Lista de recursos de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/). Siga las instrucciones de la página de especificación de la API para generar una **señal de acceso**.
   {:tip}
5. Para realizar una predicción en línea, utilice la llamada a la API `POST /online`.
   * Encontrará el valor de `instance_id` en el separador **Credenciales de servicio** del servicio {{site.data.keyword.pm_short}} bajo [Lista de recursos de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/).
   * `deployment_id` y `published_model_id` están en la **Visión general** del despliegue.
   *  Para `online_prediction_input`, utilice el siguiente JSON

     ```json
     {
     	"fields": ["sepal_length", "sepal_width", "petal_length", "petal_width"],
     	"values": [
     		[5.1, 3.5, 1.4, 0.2]
     	]
     }
     ```
   * Pulse **Pruébelo** para ver la salida JSON.

6. Utilizando los puntos finales de la API, ahora puede llamar a este modelo desde cualquier aplicación.

## Prueba del modelo

{:#test_model}

1. En **Probar**, debería ver los datos de entrada (datos de característica) cumplimentados automáticamente.
2. Pulse **Predicción** y debería ver el **Valor previsto para especies** en un gráfico.
   ![](images/solution22-build-machine-learning-model/model_predict_test.png)
3. Para la entrada y salida de JSON, pulse en los iconos que hay junto a la entrada y la salida activas.
4. Puede cambiar los datos de entrada y seguir probando el modelo.

## Creación de una conexión de datos de comentarios

{:#create_feedback_connection}

1. Para el aprendizaje continuo y la evaluación del modelo, tiene que almacenar nuevos datos en algún lugar. Cree un plan [{{site.data.keyword.dashdbshort}}](https://{DomainName}/catalog/services/db2-warehouse) servicio > **Entrada** que actúe como conexión de datos de comentarios.
2. En la página {{site.data.keyword.dashdbshort}} **Gestionar**, pulse **Abrir**. En la parte de navegación superior, seleccione **Cargar**.
3. Pulse **examinar archivos** en **Mi sistema** y cargue `iris_initial.csv`. Pulse **Siguiente**.
4. Seleccione **DASHXXXX**, por ejemplo DASH1234, como **Esquema** y pulse **Nueva tabla**. Llámela `IRIS_FEEDBACK` y pulse **Siguiente**.
5. Los tipos de datos se detectan automáticamente. Pulse **Siguiente** y **Comenzar carga**.
   ![](images/solution22-build-machine-learning-model/define_table.png)
6. Se crea un nuevo destino **DASHXXXX.IRIS_FEEDBACK**.

   Lo utilizará en el siguiente paso, en el que va a volver a entrenar el modelo para mejorar su rendimiento y su precisión.

## Reentreno del modelo

{:#retrain_model}

1. Vuelva a la [Lista de recursos de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources) y, bajo el servicio {{site.data.keyword.DSX_short}} que ha estado utilizando, pulse **Proyectos** > iris_project > **iris-model** (bajo activos) > Evaluación.
2. En **Supervisión del rendimiento**, pulse **Configurar supervisión del rendimiento**.
3. En la página de configuración de la supervisión del rendimiento,
   * Seleccione el servicio Spark. El tipo de predicción debería estar cumplimentado automáticamente.
   * Seleccione **weightedPrecision** como métrica y establezca `0.98` como umbral opcional.
   * Pulse **Crear nueva conexión** para apuntar al almacén de IBM Db2 en la nube que ha creado en la sección anterior.
   * Seleccione la conexión de almacén de Db2 y, una vez que cumplimentados los detalles de la conexión, pulse **Crear**.      ![](images/solution22-build-machine-learning-model/new_db2_connection.png)
   * Pulse **Seleccionar referencia de datos de comentarios**, apunte a la tabla IRIS_FEEDBACK y pulse **Seleccionar**.      ![](images/solution22-build-machine-learning-model/select_source.png)
   * En el recuadro **Recuento de registros necesario para la reevaluación**, escriba el número mínimo de registros nuevos para que se active el reentrenamiento. Utilice **10** o deje el campo en blanco para utilizar el valor predeterminado (1000).
   * En el recuadro **Reentreno automático**, seleccione una de las siguientes opciones:
     - Para iniciar el reentrenamiento automático siempre que el rendimiento del modelo esté por debajo del umbral que ha establecido, seleccione **cuando el rendimiento de modelo esté por debajo del umbral**. En esta guía de aprendizaje, elegirá esta opción ya que nuestro nivel de precisión está por debajo del umbral (0,98).
     - Para prohibir el reentrenamiento automático, seleccione **nunca**.
     - Para iniciar el reentrenamiento automático independientemente del rendimiento, seleccione **siempre**.
   * En el recuadro **Despliegue automático**, seleccione una de las opciones siguientes:
     - Para iniciar el despliegue automático siempre que el rendimiento del modelo sea mejor que el de la versión anterior, seleccione **cuando el rendimiento del modelo sea mejor que el de la versión anterior**. En esta guía de aprendizaje, elegirá esta opción ya que nuestro objetivo es mejorar continuamente el rendimiento del modelo.
     - Para prohibir el despliegue automático, seleccione **nunca**.
     - Para iniciar el despliegue automático independientemente del rendimiento, seleccione **siempre**.
   * Pulse **Guardar**.
     ![](images/solution22-build-machine-learning-model/configure_performance_monitoring.png)
4. Descargue el archivo [iris_retrain.csv](https://ibm.box.com/shared/static/96kvmwhb54700pjcwrd9hd3j6exiqms8.csv). A continuación, pulse **Añadir datos de comentarios**, seleccione el archivo csv descargado y pulse **Abrir**.
5. Pulse **Nueva evaluación** para empezar.      ![](images/solution22-build-machine-learning-model/retraining_model.png)
6. Cuando finalice la evaluación, puede comprobar en la sección **Resultado de última evaluación** si el valor **WeightedPrecision** ha mejorado.

## Eliminación de recursos
{:removeresources}

1. Vaya a la [Lista de recursos de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources/) > seleccione la ubicación, la organización y el espacio donde ha creado los servicios.
2. Suprima los servicios {{site.data.keyword.DSX_short}}, {{site.data.keyword.sparks}}, {{site.data.keyword.pm_short}}, {{site.data.keyword.dashdbshort}} y {{site.data.keyword.cos_short}} respectivos que ha creado para esta guía de aprendizaje.

## Contenido relacionado
{:related}

- [Visión general de Watson Studio](https://dataplatform.ibm.com/docs/content/getting-started/overview-ws.html?audience=wdp&context=wdp)
- [Detección de anomalías mediante aprendizaje automático](https://{DomainName}/docs/tutorials?topic=solution-tutorials-gather-visualize-analyze-iot-data#data_experiencee)
<!-- - [Watson Data Platform Tutorials](https://www.ibm.com/analytics/us/en/watson-data-platform/tutorial/) -->
- [Creación automática de un modelo](https://datascience.ibm.com/docs/content/analyze-data/ml-model-builder.html?linkInPage=true)
- [Aprendizaje automático e IA](https://dataplatform.ibm.com/docs/content/analyze-data/wml-ai.html?audience=wdp&context=wdp)
<!-- - [Watson machine learning client library](https://dataplatform.ibm.com/docs/content/analyze-data/pm_service_client_library.html#client-libraries) -->
