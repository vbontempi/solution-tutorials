---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Creación de un Slackbot controlado por base de datos
{: #slack-chatbot-database-watson}

En esta guía de aprendizaje, va a crear un Slackbot para crear y realizar búsquedas de sucesos y conferencias en entradas de una base de datos Db2. Slackbot tiene el soporte del servicio {{site.data.keyword.conversationfull}}. Integrará Slack y {{site.data.keyword.conversationfull}} mediante una integración con asistente.

La integración de Slack canaliza los mensajes entre Slack y {{site.data.keyword.conversationshort}}. Allí, algunas acciones de diálogo del lado del servidor realizan consultas SQL sobre una base de datos Db2. Todo el código (no es mucho) está escrito en Node.js.

## Objetivos
{: #objectives}

* Conexión de {{site.data.keyword.conversationfull}} con Slack mediante una integración
* Creación, despliegue y enlace de acciones de Node.js en {{site.data.keyword.openwhisk_short}}
* Acceso a una base de datos Db2 desde {{site.data.keyword.openwhisk_short}} mediante Node.js

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}} ](https://{DomainName}/catalog/services/db2-warehouse) o [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

  ![Arquitectura](images/solution19/SlackbotArchitecture.png)
</p>

## Antes de empezar
{: #prereqs}

Para completar esta guía de aprendizaje, necesita la versión más reciente de la [CLI de {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) y debe tener el {{site.data.keyword.openwhisk_short}} [plugin instalado](/docs/cli?topic=cloud-cli-plug-ins).


## Configuración del servicio y del entorno
En esta sección, va a configurar los servicios necesarios y a preparar el entorno. La mayor parte de estas tareas se pueden realizar desde la interfaz de línea de mandatos (CLI) mediante scripts. Están disponibles en GitHub.

1. Clone el [repositorio GitHub](https://github.com/IBM-Cloud/slack-chatbot-database-watson) y vaya al directorio clonado:
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. Si no ha iniciado una sesión, utilice `ibmcloud login` para iniciar la sesión de forma interactiva.
3. Elija como destino la organización y el espacio en los que va a crear el servicio de base de datos con este mandato:
   ```
   ibmcloud target --cf
   ```
4. Cree una instancia de {{site.data.keyword.dashdbshort}} con el nombre **eventDB**:
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
   También puede utilizar un plan que no sea el de **Entrada**.
5. Para acceder al servicio de base de datos desde {{site.data.keyword.openwhisk_short}} más adelante, necesita la autorización. Por lo tanto, cree las credenciales de servicio y etiquételas como **slackbotkey**:   
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. Cree una instancia del servicio {{site.data.keyword.conversationshort}}. Utilice **eventConversation** como nombre y el plan Lite gratuito.
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. A continuación, va a registrar acciones para {{site.data.keyword.openwhisk_short}} y a enlazar credenciales de servicio a dichas acciones. Algunas de las acciones se habilitan como acciones web y se establece un secreto para evitar invocaciones no autorizadas. Elija un secreto y páselo como parámetro; sustituya **YOURSECRET** en consecuencia.

   Se invoca una de las acciones para crear una tabla en {{site.data.keyword.dashdbshort}}. Mediante el uso de una acción de {{site.data.keyword.openwhisk_short}}, no necesita un controlador Db2 local ni tiene que utilizar la interfaz basada en navegador para crear manualmente la tabla. Para realizar el registro y la configuración, ejecute la línea siguiente, y esto ejecutará el archivo **setup.sh**, que contiene todas las acciones. Si el sistema no da soporte a mandatos de shell, copie cada línea fuera del archivo **setup.sh** y ejecútela individualmente.

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **Nota:** de forma predeterminada, el script también inserta unas pocas filas de datos de ejemplo. Para inhabilitarlo, comente la siguiente línea en el script anterior: `#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r`
8. Extraiga la información del espacio de nombres para las acciones desplegadas.

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   Anote la parte que hay antes de **/slackdemo/eventInsert**. Se trata de la organización y el espacio codificados. Lo necesitará en la siguiente sección.

## Cargar del conocimiento / espacio de trabajo
En esta parte de la guía de aprendizaje, va a cargar un espacio de trabajo o un conocimiento predefinido en el servicio {{site.data.keyword.conversationshort}}.
1. En la [Lista de recursos de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), abra la visión general de los servicios. Localice la instancia del servicio {{site.data.keyword.conversationshort}} que ha creado en la sección anterior. Pulse su entrada y luego el alias de servicio para abrir los detalles del servicio.
2. Pulse **Iniciar herramienta** para acceder a la herramienta {{site.data.keyword.conversationshort}}.
3. Vaya a **Conocimientos**, pulse **Crear conocimiento** y luego **Importar conocimiento**.
4. En el diálogo, después de pulsar **Elegir archivo JSON**, seleccione el archivo **assistant-skill.json** del directorio local. Deje la opción de importación en **todo (intenciones, entidades y diálogo)** y pulse **Importar**. Esto crea un conocimiento llamado **TutorialSlackbot**.
5. Pulse **Diálogo** para ver los nodos del diálogo. Puede ampliarlos para ver una estructura como la que se muestra a continuación.

   El diálogo tiene nodos para manejar las preguntas de ayuda y de simple agradecimiento. El nodo **newEvent** y su hijo recopilan la entrada necesaria y luego llaman a una acción para insertar un nuevo registro de sucesos en Db2.

   El nodo **query events** clarifica si los sucesos se buscan por su identificador o por fecha. Realizan la búsqueda real y la recopilación de los datos necesarios los nodos hijo **query events by shortname** y **query event by dates**.

   El nodo **credential_node** configura el secreto para las acciones de diálogo y la información acerca de la organización de Cloud Foundry. Este último se necesita para invocar las acciones.

  Los detalles se explicarán más adelante cuando todo esté configurado.
  ![](images/solution19/SlackBot_Dialog.png)   
6. Pulse el nodo de diálogo **credential_node**, abra el editor JSON pulsando el icono de menú situado a la derecha de **Responder con**.

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   Sustituya **org_space** por la información de organización y espacio codificada que ha recuperado anteriormente. Sustituya cualquier "@" por "%40". Luego cambie **YOURSECRET** por el secreto real de antes. Cierre el editor de JSON pulsando de nuevo el icono.

## Creación de un asistente e integración con Slack

Ahora creará un asistente asociado con el conocimiento de antes y lo integrará con Slack. 
1. Pulse **Conocimientos** en la parte superior izquierda y seleccione **Asistentes**. A continuación, pulse **Crear asistente**.
2. En el diálogo, rellene **TutorialAssistant** como nombre y pulse **Crear asistente**. En la pantalla siguiente, seleccione **Añadir conocimiento de diálogo**. A continuación, seleccione **Añadir conocimiento existente**, elija **TutorialSlackbot** en la lista y añádalo.
3. Después de añadir el conocimiento, pulse **Añadir integración** y, a continuación, en la lista de **Integraciones gestionadas**, seleccione **Slack**.
4. Siga las instrucciones para integrar su chatbot con Slack.

## Prueba de Slackbot y aprendizaje
Abra su espacio de trabajo Slack para realizar una prueba del chatbot. Inicie una conversación directa con el bot.

1. Escriba **help** en el formulario de mensajes. El bot debería responder con alguna pista.
2. Ahora escriba **new event** para empezar a obtener datos para un nuevo registro de sucesos. Utilizará ranuras de {{site.data.keyword.conversationshort}} para recopilar toda la información de entrada necesaria.
3. En primer lugar se encuentra el identificador o nombre del suceso. Se necesitan comillas. Permiten especificar nombres más complejos. Escriba **"Meetup: IBM Cloud"** como nombre del suceso. El nombre del suceso se define como una entidad basada en patrón **eventName**. Admite distintos tipos de comillas dobles al principio y al final.
4. A continuación se encuentra la ubicación del suceso. La entrada se basa en la [entidad del sistema **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details). Existe una limitación, que es que solo se pueden utilizar ciudades que {{site.data.keyword.conversationshort}} reconozca. Pruebe con **Friedrichshafen** como ciudad.
5. En el siguiente paso se solicita información de contacto, como dirección de correo electrónico o URI del sitio web. Empiece con **https://www.ibm.com/events**. Utilizará una entidad basada en patrón para este campo.
6. Las siguientes preguntas recopilan información de fecha y hora para el comienzo y el final. Se utilizan **sys-date** y **sys-time**, que admiten distintos formatos de entrada. Utilice **next Thursday** como fecha de inicio, **6 pm** como hora, utilice la fecha exacta del próximo jueves, por ejemplo **2019-05-09**, y **22:00** para la fecha y hora finales.
7. Por último, cuando se han recopilado todos los datos, se muestra un resumen y se invoca una acción del servidor, que se implementa como acción de {{site.data.keyword.openwhisk_short}}, para insertar un nuevo registro en Db2. A continuación, el diálogo conmuta a un nodo hijo para limpiar el entorno de proceso eliminando las variables de contexto. Todo el proceso de entrada se puede cancelar en cualquier momento escribiendo **cancel**, **exit** o similar. En ese caso, se reconoce la opción de usuario y se limpia el entorno.   ![](images/solution19/SlackSampleChat.png)   

Con algunos datos de ejemplo ya especificados, es hora de realizar una búsqueda.
1. Escriba **show event information**. Se le preguntará si desea buscar por identificador o por fecha. Especifique un **nombre** y, en la siguiente pregunta, **"Think 2019"**. Ahora el chatbot debería mostrar información acerca de ese suceso. El diálogo ofrece varias respuestas entre las que elegir.
2. Con {{site.data.keyword.conversationshort}} como programa de fondo, se pueden especificar frases más complejas omitiendo así partes del diálogo. Utilice **show event by the name "Think 2019"** como entrada. El chatbot devuelve directamente el registro de sucesos.
3. Ahora va a realizar una búsqueda por fecha. Una búsqueda se define mediante un par de fechas; la fecha de inicio del suceso tiene que estar en medio. Si especifica **search conference by date in February 2019** como entrada, el resultado debería de nuevo ser el suceso **Think 2019**. La entidad **February** se interpreta como dos fechas, 1 de febrero y 28 de febrero, con lo que se proporciona entrada para el inicio y el final del rango de fechas. [Si no se especificara el año 2019, se identificaría un febrero futuro](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time). 

Después de algunas búsquedas más y de nuevas entradas de sucesos, puede volver a visitar el historial de conversaciones y mejorar el futuro diálogo. Siga las instrucciones de la [documentación de {{site.data.keyword.conversationshort}} sobre **Cómo mejorar la comprensión**](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro).


## Eliminación de recursos
{:removeresources}

Si se ejecuta el script de limpieza en el directorio principal, se suprime la tabla de sucesos de {{site.data.keyword.dashdbshort}} y se eliminan las acciones de {{site.data.keyword.openwhisk_short}}. Esto puede resultar útil cuando empieza a modificar o a ampliar el código. El script de limpieza no modifica el espacio de trabajo de {{site.data.keyword.conversationshort}} ni el conocimiento.   
```bash
sh cleanup.sh
```
{:codeblock}

En la [Lista de recursos de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), abra la visión general de los servicios. Localice la instancia del servicio {{site.data.keyword.conversationshort}} y suprímala.

## Ampliación de la guía de aprendizaje
¿Desea ampliar o modificar esta guía de aprendizaje? Estas son algunas de las ideas:
1. Añada prestaciones de búsqueda, como por ejemplo búsqueda con comodines o búsqueda de duraciones de sucesos ("todos los sucesos de más de 8 horas").
2. Utilice {{site.data.keyword.databases-for-postgresql}} en lugar de {{site.data.keyword.dashdbshort}}. El [repositorio GitHub para esta guía de aprendizaje de Slackbot](https://github.com/IBM-Cloud/slack-chatbot-database-watson) ya contiene código que da soporte a {{site.data.keyword.databases-for-postgresql}}.
3. Añada un servicio meteorológico y recupere la previsión del tiempo correspondiente a la fecha y a la ubicación del suceso.
4. Exporte datos de sucesos como un archivo **.ics** de iCalendar.
5. Conecte el chatbot a Facebook Messenger mediante la adición de otra integración.
6. Añada elementos interactivos, como botones, a la salida.      


## Contenido relacionado
{:related}

Estos son algunos enlaces con información adicional sobre los temas tratados en esta guía de aprendizaje.

Publicaciones del blog relacionadas con chatbot:
* [Chatbots: algunos trucos con ranuras en IBM Watson Conversation](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [Chatbots en directo: prácticas recomendadas](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [Creación de chatbots: más trucos y sugerencias](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

Documentación y SDK:
* Repositorio GitHub con [sugerencias y trucos para gestionar variables en IBM Watson Conversation](https://github.com/IBM-Cloud/watson-conversation-variables)
* [Documentación de {{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Documentación: [IBM Knowledge Center de {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Db2 Developer Community Edition gratuito](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) para desarrolladores
* Documentación: [Descripción de API del controlador ibm_db Node.js](https://github.com/ibmdb/node-ibm_db)
* [Documentación de {{site.data.keyword.cloudantfull}}](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
