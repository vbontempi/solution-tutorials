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

# Análisis y visualización de datos abiertos con Apache Spark
{: #big-data-analytics-spark}

En esta guía de aprendizaje, analizará y visualizará conjuntos de datos abiertos mediante {{site.data.keyword.DSX_full}}, Jupyter Notebook y Apache Spark. Empezará por combinar datos que describen el crecimiento de la población, la esperanza de vida y los códigos ISO de los países en un solo marco de datos. Para descubrir información importante, utilizará una biblioteca de Python llamada Pixiedust para consultar y visualizar los datos de varias maneras.

<p style="text-align: center;">

  ![](images/solution23/Architecture.png)
</p>

## Objetivos
{: #objectives}

* Despliegue de Apache Spark y de {{site.data.keyword.DSX_short}} en IBM Cloud
* Utilización de Jupyter Notebook y de un kernel Python
* Importación, transformación, análisis y visualización de conjuntos de datos

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
   * {{site.data.keyword.sparkl}}
   * {{site.data.keyword.DSX_full}}
   * {{site.data.keyword.cos_full_notm}}

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Configuración del servicio y del entorno
Empiece por suministrar los servicios que se utilizan en esta guía de aprendizaje y cree un proyecto en {{site.data.keyword.DSX_short}}.

Puede suministrar servicios para {{site.data.keyword.Bluemix_short}} desde la [Lista de recursos](https://{DomainName}/resources) y desde el [catálogo](https://{DomainName}/catalog/). Como alternativa, {{site.data.keyword.DSX_short}} le permite crear o añadir servicios de Data & Analytics existentes desde su panel de control y desde los valores del proyecto.
{:tip}

1. En el [catálogo de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/catalog), vaya a la sección **IA**. Cree el servicio **{{site.data.keyword.DSX_short}}**. Pulse el botón **Iniciación** para iniciar el panel de control de **{{site.data.keyword.DSX_short}}**.
2. En el panel de control, pulse el mosaico **Crear un proyecto** > y seleccione **Estándar** > Crear proyecto. En el campo **Nombre**, especifique `1stProject`. Puede dejar la descripción vacía.
3. En el lado derecho de la página, puede **Definir almacenamiento**. Si ya ha suministrado almacenamiento, seleccione una instancia de la lista. Si no es así, pulse **Añadir** y siga las instrucciones del nuevo separador del navegador. Cuando termine de crear el servicio, pulse **Renovar** para ver el nuevo servicio.
4. Pulse el botón **Crear** para crear el proyecto. Se le dirigirá a la página de visión general del proyecto.  
   ![](images/solution23/NewProject.png)
5. En la página de visión general, pulse **Valores**.
6. En la sección **Servicios asociados**, pulse **Añadir servicio** y seleccione **Spark** en el menú. En la pantalla que aparece, puede elegir una instancia de servicio Spark existente o puede crear una nueva.

## Creación y preparación de un cuaderno
[Jupyter Notebook](http://jupyter.org/) es una aplicación web de código abierto que le permite crear y compartir documentos que contienen código activo, ecuaciones, visualizaciones y texto explicativo. Los cuadernos y otros recursos se organizan en proyectos.
1. Pulse el separador **Activos**, desplácese hasta la sección **Cuadernos** y pulse **Nuevo cuaderno**.
2. Utilice el cuaderno en **Blanco**. Escriba `MyNotebook` como **Nombre**.
3. En el menú **Seleccionar tiempo de ejecución**, seleccione la instancia de Spark que ha añadido a los valores del proyecto. Conserve el **Lenguaje** predeterminado como **Python 3.5**.
4. Pulse **Crear cuaderno** para completar el proceso.
5. El campo en el que especifica texto y mandatos se denomina **celda**. Copie el código siguiente en la celda vacía para importar el [paquete **Pixiedust**](https://pixiedust.github.io/pixiedust/use.html). Para ejecutar la celda, pulse el icono **Ejecutar** de la barra de herramientas o pulse **Mayús + Intro** en el teclado.
   ```Python
   import pixiedust
   ```
   {:codeblock}
   ![](images/solution23/FirstCell_ImportPixiedust.png)

Si nunca ha trabajado con Jupyter Notebooks, pulse el icono **Docs** en el menú superior derecho. Vaya a **Analizar datos** y luego a la [sección **Cuadernos**](https://dataplatform.ibm.com/docs/content/analyze-data/notebooks-parent.html?context=analytics) para obtener más información sobre [los cuadernos y sus componentes](https://dataplatform.ibm.com/docs/content/analyze-data/parts-of-a-notebook.html?context=analytics&linkInPage=true).
{:tip}

## Carga de datos
A continuación, cargue tres conjuntos de datos abiertos y póngalos a disposición del cuaderno. La biblioteca **Pixiedust** le permite [cargar fácilmente archivos **CSV** utilizando un URL](https://pixiedust.github.io/pixiedust/loaddata.html).

1.  Copie la línea siguiente en la siguiente celda vacía del cuaderno, pero no la ejecute todavía.
   ```Python
   df_pop = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
2. En otro separador del navegador, vaya a la sección [Comunidad](https://dataplatform.ibm.com/community?context=analytics). En **Conjuntos de datos**, busque [**Total population by country**](https://dataplatform.ibm.com/exchange/public/entry/view/889ca053a19986a4445839358a91963e) y pulse el mosaico. En la parte superior derecha, pulse el icono **enlace** para obtener un URI de acceso. Copie el URI y sustituya el texto **YourAccessURI** de la celda del cuaderno por el enlace. Pulse el icono **Ejecutar** en la barra de herramientas o **Mayús + Intro**.
3. Repita el paso para otro conjunto de datos. Copie la línea siguiente en la siguiente celda vacía del cuaderno.
   ```Python
   df_life = pixiedust.sampleData('YourAccessURI')
   ```
   {:codeblock}
4. En el otro separador del navegador con los **Conjuntos de datos**, busque [**Life expectancy at birth by country in total years**](https://dataplatform.ibm.com/exchange/public/entry/view/f15be429051727172e0d0c226e2ce895). Vuelva a obtener el enlace y utilícelo para sustituir **YourAccessURI** en la celda del cuaderno y pulse **Ejecutar** para iniciar el proceso de carga.
5. Para el último de los tres conjuntos de datos, cargue una lista de nombres de países y sus códigos ISO desde una colección de conjuntos de datos abiertos en Github. Copie el código en la siguiente celda vacía del cuaderno y ejecútelo.
   ```Python
     df_countries = pixiedust.sampleData('https://raw.githubusercontent.com/datasets/country-list/master/data.csv')
   ```
   {:codeblock}

La lista de códigos de país se utilizará posteriormente para simplificar la selección de datos utilizando un código de país en lugar del nombre exacto del país.

## Transformación de datos
Cuando los datos estén disponibles, transfórmelos ligeramente y combine los tres conjuntos en un solo marco de datos.
1. El siguiente bloque de códigos definirá el marco de datos para los datos sobre población. Esto se lleva a cabo con una sentencia de SQL que cambia el nombre de las columnas. A continuación, se crea una vista y se imprime el esquema. Copie este código en la siguiente celda vacía y ejecútelo.
   ```Python
   sqlContext.registerDataFrameAsTable(df_pop, "PopTable")
   df_pop = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Population FROM PopTable")
   df_pop.createOrReplaceTempView('population')
   df_pop.printSchema()
   ```
   {:codeblock}
2. Repita el proceso para los datos sobre esperanza de vida (Life Expectancy). En lugar de imprimir el esquema, este código imprime las 10 primeras filas.  
   ```Python
   sqlContext.registerDataFrameAsTable(df_life, "lifeTable")
   df_life = sqlContext.sql("SELECT `Country or Area` as Country, Year, Value as Life FROM lifeTable")
   df_life = df_life.withColumn("Life", df_life["Life"].cast("double"))
   df_life.createOrReplaceTempView('life')
   df_life.show(10)
   ```
   {:codeblock}

3. Repita la transformación del esquema para los datos sobre el país.
   ```Python
   sqlContext.registerDataFrameAsTable(df_countries, "CountryTable")
   df_countries = sqlContext.sql("SELECT `Name` as Country, Code as CountryCode FROM CountryTable")
   df_countries.createOrReplaceTempView('countries')
   ```
   {:codeblock}

4. Ahora los nombres de columna son más sencillos e iguales en los conjuntos de datos, que se pueden combinar en un único marco de datos. Realice una unión **externa** entre los datos sobre expectativa de vida y población. A continuación, en la misma sentencia, realice una unión **interna** para añadir los códigos de país. Todo se ordenará por país y año. La salida define el marco de datos **df_all**. Al utilizar una unión interna, los datos resultantes contienen solo los países que se encuentran en la lista ISO. Este proceso elimina de los datos las entradas regionales y de otro tipo.
   ```Python
   df_all = df_life.join(df_pop, ['Country', 'Year'], 'outer').join(df_countries, ['Country'], 'inner').orderBy(['Country', 'Year'], ascending=True)
   df_all.show(10)
   ```
   {:codeblock}

5. Cambie el tipo de datos de **Year** por un entero.
   ```Python
   df_all = df_all.withColumn("Year", df_all["Year"].cast("integer"))
   df_all.printSchema()
   ```
   {:codeblock}

Los datos combinados están listos para ser analizados.

## Análisis de datos
En esta parte, utilice [Pixiedust para visualizar los datos en distintos gráficos](https://pixiedust.github.io/pixiedust/displayapi.html). Empiece por comparar la esperanza de vida de algunos países.

1. Copie el código en la siguiente celda vacía y ejecútelo.
   ```Python
   df_all.createOrReplaceTempView('l2')
   dfl2=spark.sql("SELECT Life, Country, Year FROM l2 where CountryCode in ('CN','DE','FR','IN','US')")
   display(dfl2)
   ```
   {:codeblock}
2. Se muestra una tabla desplazable. Pulse el icono del gráfico directamente bajo el bloque de código y seleccione **Gráfico de líneas**. Aparecerá un cuadro de diálogo emergente con **Pixiedust: Line Chart Options**. Especifique un **Título de gráfico** como "Comparison of Life Expectancy". Desde los **Campos** mostrados, arrastre **Year** al recuadro **Claves** y **Life** al área **Valores**. Escriba **1000** para **# of Rows to Display**. Pulse **Aceptar** para que se trace el gráfico de líneas. En el lado derecho, asegúrese de que **mapplotlib** esté seleccionado como **Visualizador**. Pulse selector **Agrupar por** y seleccione **Country**. Se mostrará un gráfico similar al siguiente.    ![](images/solution23/LifeExpectancy.png)

3. Cree un gráfico que se centre en el año 2010. Copie el código en la siguiente celda vacía y ejecútelo.
   ```Python
   df_all.createOrReplaceTempView('life2010')
   df_life_2010=spark.sql("SELECT Life, Country FROM life2010 WHERE Year=2010 AND Life is not NULL ")
   display(df_life_2010)
   ```
   {:codeblock}
4. En el selector de gráficos, seleccione **Mapa**. En el diálogo de configuración, arrastre **Country** al área **Claves**. Mueva **Life** al recuadro **Valores**. De forma parecida al primer gráfico, aumente el **Número de filas que mostrar** a **1000**. Pulse **Aceptar** para trazar el mapa. Seleccione **brunel** como **Visualizador**. Se muestra un mapa mundial en colores correspondiente a la esperanza de vida. Puede utilizar el ratón para aumentar el mapa.
![](images/solution23/LifeExpectancyMap2010.png)

## Ampliación de la guía de aprendizaje
A continuación encontrará algunas ideas y sugerencias para mejorar esta guía de aprendizaje.
* Cree y visualice una consulta que muestre la tasa de esperanza de vida en relación con el crecimiento de la población para el país que elija
* Calcule y visualice las tasas de crecimiento de la población por país en un mapa mundial
* Cargue e integre de datos adicionales desde el catálogo de conjuntos de datos
* Exporte los datos combinados a un archivo o a una base de datos

## Contenido relacionado
{:related}
A continuación se proporcionan enlaces relacionados con los temas tratados en esta guía de aprendizaje.
* [Watson Data Platform](https://dataplatform.ibm.com): utilice Watson Data Platform para colaborar y crear aplicaciones maestras. Visualice rápidamente y descubra detalles de sus datos y establezca una colaboración entre los equipos.
* [PixieDust](https://www.ibm.com/cloud/pixiedust): herramienta de productividad de código abierto para Jupyter Notebooks
* [Cognitive Class.ai](https://cognitiveclass.ai/): cursos sobre análisis de datos y cálculo cognitivo
* [IBM Watson Data Lab](https://ibm-watson-data-lab.github.io/): lo que hacemos con los datos, que usted también puede hacer
* [Servicio Analytics Engine](https://{DomainName}/catalog/services/analytics-engine): desarrolle y despliegue aplicaciones analíticas mediante Apache Spark y Apache Hadoop de código abierto
