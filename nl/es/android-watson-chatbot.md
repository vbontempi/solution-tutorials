---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# Creación de un chatbot Android con soporte de voz
{: #android-watson-chatbot}

Aprenda lo fácil que es crear rápidamente un chatbot nativo de Android con soporte de voz con los servicios {{site.data.keyword.conversationshort}}, {{site.data.keyword.texttospeechshort}} y {{site.data.keyword.speechtotextshort}} en {{site.data.keyword.Bluemix_short}}.

Esta guía de aprendizaje se muestra cómo definir intenciones y entidades y cómo crear un flujo de diálogo para que el chatbot responda a las consultas de los clientes. Aprenderá a habilitar los servicios {{site.data.keyword.speechtotextshort}} y {{site.data.keyword.texttospeechshort}} para facilitar la interactuación con la app de Android.
{:shortdesc}

## Objetivos
{: #objectives}

- Utilizar {{site.data.keyword.conversationshort}} para personalizar y desplegar un chatbot.
- Permitir que los usuarios finales interactúen con chatbot mediante voz y audio.
- Configurar y ejecutar la app de Android.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los productos siguientes:

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## Arquitectura
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* Los usuarios interactúan con una aplicación móvil mediante voz.
* El audio se transcribe a texto con {{site.data.keyword.speechtotextfull}}.
* El texto se pasa a {{site.data.keyword.conversationfull}}.
* La respuesta de {{site.data.keyword.conversationfull}} se convierte en audio mediante {{site.data.keyword.texttospeechfull}} y el resultado se devuelve a la aplicación móvil.

## Antes de empezar
{: #prereqs}

- Descargue e instale [Android Studio](https://developer.android.com/studio/index.html).

## Creación de servicios
{: #setup}

En esta sección, creará los servicios que necesita la guía de aprendizaje, empezando por {{site.data.keyword.conversationshort}} para crear asistentes virtuales cognitivos que ayuden a los clientes.

1. Vaya al [**catálogo de {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) y seleccione [Servicio {{site.data.keyword.conversationshort}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation) > Plan **Lite**:
   1. Establezca **Nombre** en **android-chatbot-assistant**
   1. **Crear**.
2. Pulse **Credenciales de servicio** en el panel izquierdo y pulse **Nueva credencial**.
   1. Establezca **Nombre** en **for-android-app**.
   1. **Añadir**.
3. Pulse **Ver credenciales** para ver las credenciales. Anote la **Clave de API** y el **URL**; necesitará esta información para la aplicación móvil.

El servicio {{site.data.keyword.speechtotextshort}} convierte la voz humana en texto escrito que se puede enviar como entrada al servicio {{site.data.keyword.conversationshort}} en {{site.data.keyword.Bluemix_short}}.

1. Vaya al [Catálogo de **{{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) y seleccione [Servicio {{site.data.keyword.speechtotextshort}}](https://{DomainName}/catalog/services/speech-to-text) > Plan **Lite**.
   1. Establezca **Nombre** en **android-chatbot-stt**.
   1. **Crear**.
2. Pulse **Credenciales de servicio** en el panel izquierdo y pulse **Nueva credencial** para añadir una nueva credencial.
   1. Establezca **Nombre** en **for-android-app**.
   1. **Añadir**.
3. Pulse **Ver credenciales** para ver las credenciales. Anote la **Clave de API** y el **URL**; necesitará esta información para la aplicación móvil.

El servicio {{site.data.keyword.texttospeechshort}} procesa texto y lenguaje natural para generar una salida de audio sintetizada completa con la cadencia y entonación adecuadas. El servicio proporciona varias voces y se puede configurar en la app Android.

1. Vaya al [**Catálogo de {{site.data.keyword.Bluemix_notm}}**](https://{DomainName}/catalog/) y seleccione [Servicio {{site.data.keyword.texttospeechshort}}](https://{DomainName}/catalog/services/text-to-speech) > Plan **Lite**.
   1. Establezca **Nombre** en **android-chatbot-tts**.
   1. **Crear**.
2. Pulse **Credenciales de servicio** en el panel izquierdo y pulse **Nueva credencial** para añadir una nueva credencial.
   1. Establezca **Nombre** en **for-android-app**.
   1. **Añadir**.
3. Pulse **Ver credenciales** para ver las credenciales. Anote la **Clave de API** y el **URL**; necesitará esta información para la aplicación móvil.

## Creación de un conocimiento
{: #create_workspace}

Un conocimiento es un contenedor para los artefactos que definen el flujo de la conversación.

En esta guía de aprendizaje, guardará y utilizará el archivo [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) con intenciones, entidades y flujo de diálogo predefinidos en su máquina.

1. En la página de detalles del servicio {{site.data.keyword.conversationshort}}, vaya a **Gestionar** en el panel izquierdo y pulse **Iniciar herramienta** para ver el panel de control de {{site.data.keyword.conversationshort}}.
1. Pulse el separador **Conocimientos**.
1. Pulse **Crear nuevo** e **Importar conocimiento** y elija el archivo JSON descargado antes.
1. Seleccione la opción **Todo** y pulse **Importar**. Se crea un nuevo conocimiento con intenciones, entidades y flujo de diálogo predefinidos.
1. Vuelva a la lista de conocimientos. Seleccione el menú de acciones del conocimiento `Ana` para **Ver detalles de la API**.

### Definición de una intención
{:#define_intent}

Una intención representa la finalidad de la entrada de un usuario, como por ejemplo responder a una pregunta o procesar el pago de una factura. El usuario define una intención para cada tipo de solicitud de usuario a la que desea que la aplicación dé soporte. Al reconocer la intención expresada en la entrada de un usuario, el servicio {{site.data.keyword.conversationshort}} puede elegir el flujo de diálogo correcto para responder a la misma. En la herramienta, el nombre de una intención siempre tiene como prefijo el carácter `#`.

Las intenciones son lo que quiere hacer el usuario final. A continuación se muestran algunos ejemplos de nombres de intención.
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. Pulse el conocimiento que acaba de crear: **Ana**.

   Ana es un bot de seguros para que los usuarios puedan consultar sus ventajas y archivar reclamaciones.
   {:tip}
2. Pulse el primer separador para ver todas las **Intenciones**.
3. Pulse **Añadir intención** para crear una nueva intención. Escriba `Cancel_Policy` como nombre de intención tras `#` y especifique una descripción opcional.
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. Pulse **Crear intención**.
5. Añada ejemplos de usuario cuando se solicite para cancelar una póliza.
   - `Quiero cancelar mi póliza`
   - `Eliminar mi póliza ahora`
   - `Quiero dejar de hacer pagos en mi póliza.`
6. Añada ejemplos de usuario uno tras otro y pulse **Añadir ejemplo**. Repita esto para todos los demás ejemplos de usuario.

   Recuerde añadir al menos 5 ejemplos de usuario para entrenar mejor el bot.
   {:tip}

7. Pulse el botón **cerrar** ![](images/solution28-watson-chatbot-android/close_icon.png) que hay junto al nombre de la intención para guardar la intención.
8. Pulse **Catálogo de contenido** y seleccione **General**. Pulse **Añadir a conocimiento**.

   El catálogo de contenido le ayuda a comenzar a trabajar más rápidamente añadiendo intenciones existentes (banca, atención al cliente, seguros, telecomunicaciones, comercio electrónico y muchos más). Estas intenciones se han entrenado con preguntas comunes que pueden formular los usuarios.
   {:tip}

### Definición de una entidad
{:#define_entity}

Una entidad representa un objeto que es relevante para las intenciones y que ofrece un contexto específico para una intención. Debe obtener una lista de los posibles valores para cada entidad y sinónimos que puedan especificar los usuarios. Cuando reconoce las entidades mencionadas en la información de entrada del usuario, el servicio {{site.data.keyword.conversationshort}} puede elegir las acciones específicas que debe llevar a cabo para cumplir una intención. En la herramienta, el nombre de una entidad siempre tiene como prefijo el carácter `@`.

A continuación se muestran ejemplos de nombres de entidades
 - `@location`
 - `@menu_item`
 - `@product`

1. Pulse el separador **Entidades** para ver las entidades existentes.
2. Pulse **Añadir entidad** y especifique el nombre de la entidad como `location` después de `@`. Pulse **Crear entidad**.
3. Especifique `address` como nombre de valor y seleccione **Sinónimos**.
4. Añada `place` como sinónimo y pulse el icono ![](images/solution28-watson-chatbot-android/plus_icon.png). Repita con sinónimos `office`, `centre`, `branch` etc., y pulse **Añadir valor**.
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. Pulse **cerrar** ![](images/solution28-watson-chatbot-android/close_icon.png) para guardar los cambios.
6. Pulse el separador **Entidades del sistema** para comprobar las entidades comunes creadas por IBM que se pueden utilizar en cualquier caso de uso.

   Las entidades del sistema se pueden utilizar para reconocer una amplia gama de valores correspondientes a los tipos de objetos que representan. Por ejemplo, la entidad del sistema `@sys-number` coincide con cualquier valor numérico, incluidos números enteros, fracciones decimales o incluso números escritos como palabras.
   {:tip}
7. Coloque **Estado** en `activado` para las entidades del sistema @sys-person y @sys-location.

### Creación del flujo de diálogo
{:#build_dialog}

Un diálogo es un flujo de conversación que define la forma en que la aplicación responde cuando reconoce intenciones y entidades definidas. Puede utilizar el creador de diálogos en la herramienta para crear conversaciones con los usuarios, proporcionando respuestas basadas en intenciones y entidades que se reconoce en su entrada.

1. Pulse el separador **Diálogo** para ver el flujo de diálogo existente con intenciones y entidades.
2. Pulse **Añadir nodo** para añadir un nodo nuevo al diálogo.
3. En **si el asistente reconoce**, especifique `#Cancel_Policy`.
4. En **Responder con**, especifique la respuesta `This facility is not available online. Please visit our nearest branch to cancel your policy.`
5. Pulse ![](images/solution28-watson-chatbot-android/save_node.png) para cerrar y guardar el nodo.
6. Desplácese para ver el nodo `#greeting`. Pulse el nodo para ver los detalles.    ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. Pulse el icono ![](images/solution28-watson-chatbot-android/add_condition.png) para **añadir una nueva condición**. Seleccione `or` en el menú desplegable y especifique `#General_Greetings` como intención. La sección **Responder con** muestra la respuesta del asistente cuando el usuario le saluda.
![](images/solution28-watson-chatbot-android/apply_condition.png)

   Una variable de contexto es una variable que el usuario define en un nodo y para la que puede especificar un valor predeterminado. Otros nodos o lógica de aplicación pueden establecer o cambiar posteriormente el valor de la variable de contexto. La aplicación puede pasar información al diálogo, y el diálogo puede actualizar esta información y pasarla a la aplicación o un nodo posterior. El diálogo lo hace mediante variables de contexto.
   {:tip}

8. Pruebe el flujo de diálogo con el botón **Pruébelo**.

## Enlace de un conocimiento con un asistente

Un **asistente** es un bot cognitivo que puede personalizar para adaptarlo a sus necesidades empresariales y desplegar en varios canales para ofrecer ayuda a los clientes donde y cuando la necesiten. Puede personalizar el asistente añadiéndole los **conocimientos** que necesita para satisfacer los objetivos de sus clientes.

1. En la herramienta {{site.data.keyword.conversationshort}}, cambie a **Asistentes** y utilice **Crear nuevo**.
   1. Establezca **Nombre** en **android-chatbot-assistant**
   1. **Crear**
1. Utilice **Añadir conocimiento de diálogo** para seleccionar el conocimiento creado en las secciones anteriores.
   1. **Añadir conocimiento existente**
   1. Seleccione **Ana**
1. Seleccione el menú de acción en Asistente > **Valores** > **Detalles de API**, anote el **ID de asistente**, ya que tendrá que hacer referencia al mismo desde la aplicación móvil (en el archivo `config.xml` de la app de Android).

## Configuración y ejecución de la app de Android
{:#configure_run_android_app}

El repositorio contiene el código de aplicación de Android con las dependencias de Gradle necesarias.

1. Ejecute el mandato siguiente para clonar el [repositorio GitHub](https://github.com/IBM-Cloud/chatbot-watson-android):
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. Inicie Android Studio > **Abra un proyecto de Android Studio existente** y apunte al código descargado. Se activará automáticamente una compilación de **Gradle** y se descargarán todas las dependencias.
3. Abra `app/src/main/res/values/config.xml` para ver los marcadores (`ASSISTANT_ID_HERE`) correspondientes a las credenciales de servicio. Especifique las credenciales de servicio (que ha guardado anteriormente) en sus respectivos marcadores de posición y guarde el archivo.
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <!--Credenciales de servicio de Watson Assistant-->
       <!-- SUSTITUYA `ASSISTANT_ID_HERE` por el ID de Assistant que va a utilizar -->
       <string name="assistant_id">ASSISTANT_ID_HERE</string>

       <!-- SUSTITUYA `ASSISTANT_API_KEY_HERE` con la clave de API de servicio de Watson Assistant-->
       <string name="assistant_apikey">ASSISTANT_API_KEY_HERE</string>

       <!-- SUSTITUYA `ASSISTANT_URL_HERE` por el URL de servicio de Watson Assistant-->
       <string name="assistant_url">ASSISTANT_URL_HERE</string>

       <!--Credenciales de servicio de Watson Speech To Text (STT)-->
       <!-- SUSTITUYA `STT_API_KEY_HERE` por la clave de API de servicio de Watson Speech to Text-->
       <string name="STT_apikey">STT_API_KEY_HERE</string>

       <!-- SUSTITUYA `STT_URL_HERE` por el URL de servicio de Watson Speech to Text-->
       <string name="STT_url">STT_URL_HERE</string>

       <!--Credenciales de servicio de Watson Text To Speech (TTS)-->
       <!-- SUSTITUYA `TTS_API_KEY_HERE` por la clave de API de servicio de Watson Text to Speech-->
       <string name="TTS_apikey">TTS_API_KEY_HERE</string>

       <!-- SUSTITUYA `TTS_URL_HERE` por el URL de servicio de Watson Text to Speech-->
       <string name="TTS_url">TTS_URL_HERE</string>
   </resources>
   ```
4. Cree el proyecto e inicie la aplicación en un dispositivo real o con un simulador.
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. **Especifique su consulta** en el espacio que se proporciona a continuación y pulse el icono de flecha para enviar la consulta al servicio {{site.data.keyword.conversationshort}}.
6. La respuesta se pasará al servicio {{site.data.keyword.texttospeechshort}} y debería oír una voz que lee la respuesta.
7. Pulse el icono **mic** en la esquina inferior izquierda de la app para entrar voz que se convertirá en texto y que luego se puede enviar al servicio {{site.data.keyword.conversationshort}} pulsando el icono de flecha.


## Eliminación de recursos
{:removeresources}

1. Vaya a la [Lista de recursos](https://{DomainName}/resources/).
1. Suprima los servicios que ha creado:
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## Contenido relacionado
{:related}

- [Creación de entidades, sinónimos y entidades del sistema](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [Variables de contexto](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [Creación de un diálogo complejo](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [Obtención de información con ranuras](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [Opciones de despliegue](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [Mejora de su conocimiento](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
