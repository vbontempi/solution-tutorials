---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: data-hd-operatingsystem="ios"}
{:android: data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Aplicación móvil con un programa de fondo sin servidor
{: #serverless-mobile-backend}

En esta guía de aprendizaje, aprenderá a utilizar {{site.data.keyword.openwhisk}} junto con servicios cognitivos y de datos para crear un programa de fondo sin servidor para una aplicación móvil.
{:shortdesc}

No todos los desarrolladores de apps móviles tienen experiencia en la gestión de la lógica del servidor o de un servidor con el que comenzar. Preferirían concentrar sus esfuerzos en la app que están creando. Ahora bien, ¿qué pasaría si pudieran reutilizar sus conocimientos de desarrollo existentes para escribir su programa de fondo móvil?

{{site.data.keyword.openwhisk_short}} es una plataforma orientada a sucesos sin servidor. Tal como se [describe en este ejemplo](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp), las acciones que se despliegan se pueden convertir fácilmente en puntos finales HTTP como *acciones web* para crear una API de programa de fondo de aplicación web. Pensemos en una aplicación web que es un cliente para la API REST; resulta sencillo llevar este ejemplo un paso más allá y aplicar el mismo enfoque para crear un programa de fondo para una app móvil. Y, con {{site.data.keyword.openwhisk_short}}, los desarrolladores de apps móviles pueden escribir las acciones en el mismo lenguaje que utilizan para su app móvil, Java para Android y Swift para iOS.

Esta guía de aprendizaje se puede configurar en función de la plataforma de destino. Actualmente está visualizando la documentación correspondiente a la versión **iOS / Swift** de esta guía de aprendizaje. Utilice el separador de la parte superior de esta documentación para seleccionar la versión de **Android / Java** de esta guía de aprendizaje.
{: swift}

Esta guía de aprendizaje se puede configurar en función de la plataforma de destino. Actualmente está visualizando la documentación correspondiente a la versión **Android / Java** de esta guía de aprendizaje. Utilice el separador de la parte superior de esta documentación para seleccionar la versión de **iOS / Swift** de esta guía de aprendizaje.
{: java}

## Objetivos
{: #objectives}

* Despliegue de un programa de fondo móvil sin servidor con {{site.data.keyword.openwhisk_short}}.
* Adición de autenticación de usuario a una app móvil con {{site.data.keyword.appid_short}}.
* Análisis de los comentarios de los usuarios con {{site.data.keyword.toneanalyzershort}}.
* Envío de notificaciones con {{site.data.keyword.mobilepushshort}}.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

La aplicación que se muestra en esta guía de aprendizaje es una app de comentarios que analiza de forma inteligente el tono del texto de los comentarios y responde al cliente en consecuencia a través de un {{site.data.keyword.mobilepushshort}}.

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. El usuario se autentica en [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID). {{site.data.keyword.appid_short}} proporciona señales de acceso y de identificación.
2. Las siguientes llamadas a la API del programa de fondo incluyen la señal de acceso.
3. El programa de fondo se implementa con [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk). Las acciones sin servidor, que se exponen como acciones Web, esperan que la señal se envíe en la cabecera de la solicitud y verifican su validez (firma y fecha de caducidad) antes de permitir el acceso a la API real.
4. Cuando el usuario envía un comentario, este se guarda en [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
5. El texto de comentarios se procesa con [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer).
6. En función del resultado del análisis, se envía una notificación al cliente mediante [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush).
7. El usuario recibe la notificación en el dispositivo.

## Antes de empezar
{: #prereqs}

En esta guía de aprendizaje se utiliza la herramienta de línea de mandatos de {{site.data.keyword.Bluemix_notm}} para suministrar recursos y desplegar código. Asegúrese de instalar la herramienta de línea de mandatos `ibmcloud`.

* [Herramientas del desarrollador de {{site.data.keyword.Bluemix_notm}}](https://github.com/IBM-Cloud/ibm-cloud-developer-tools): script para instalar la CLI de ibmcloud y los plugins necesarios (Cloud Foundry y {{site.data.keyword.openwhisk_short}})

Además, necesitará el software y las cuentas siguientes:

   1. Java 8
   2. Android Studio 2.3.3
   3. Cuenta de Google Developer para configurar Firebase Cloud Messaging
   4. Bash shell, cURL
   {: java}


   1. Xcode
   2. Cuenta de Apple Developer para configurar Apple Push Notification Service
   3. Bash shell, cURL
   {: swift}

En esta guía de aprendizaje, configurará notificaciones push para la aplicación. En la guía de aprendizaje se da por supuesto que ha seguido la guía de aprendizaje básica de {{site.data.keyword.mobilepushshort}} para [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) o para [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics) y que está familiarizado con la configuración de Firebase Cloud Messaging o de Apple Push Notification Service.
{:tip}

Para que los usuarios de Windows 10 trabajen con las instrucciones de la línea de mandatos, recomendamos instalar el subsistema Windows para Linux y Ubuntu, tal como se describe en [este artículo](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10).
{: tip}
{: java}

## Obtención del código de la aplicación

El repositorio contiene tanto la aplicación móvil como el código de acciones de {{site.data.keyword.openwhisk_short}}.

1. Extraiga el código del repositorio GitHub

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-android
   ```
   {: pre}
   {: java}

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-ios
   ```
   {: pre}
   {: swift}

2. Revise la estructura del código

| Archivo                                     | Descripción                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions) | Código para las acciones de {{site.data.keyword.openwhisk_short}} del programa de fondo móvil sin servidor |
| [**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android) | Código para la aplicación móvil          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh) | Script de ayuda para instalar, desinstalar, actualizar el activador, las acciones y las reglas de {{site.data.keyword.openwhisk_short}} |
{: java}

| Archivo                                     | Descripción                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions) | Código para las acciones de {{site.data.keyword.openwhisk_short}} del programa de fondo móvil sin servidor |
| [**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp) | Código para la aplicación móvil          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh) | Script de ayuda para instalar, desinstalar, actualizar el activador, las acciones y las reglas de {{site.data.keyword.openwhisk_short}} |
{: swift}

## Suministro de servicios para gestionar la autenticación de los usuarios, la persistencia y el análisis de los comentarios
{: #provision_services}

En esta sección, se suministrarán los servicios que utiliza la aplicación. Puede optar por suministrar los servicios desde el catálogo de {{site.data.keyword.Bluemix_notm}} o mediante la línea de mandatos `ibmcloud`.

Se recomienda crear un nuevo espacio para suministrar los servicios y desplegar el programa de fondo sin servidor. Esto ayuda a mantener juntos todos los recursos.

### Suministro de servicios desde el catálogo de {{site.data.keyword.Bluemix_notm}}

1. Vaya al [catálogo de {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/catalog/)
2. Cree un servicio [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) con el plan **Lite**. Defina como nombre **serverlessfollowup-db**.
3. Cree un servicio [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) con el plan **Estándar**. Defina como nombre **serverlessfollowup-tone**.
4. Cree un servicio [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) con el plan **Por niveles** plan. Defina como nombre **serverlessfollowup-appid**.
5. Cree un servicio [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) con el plan **Lite**. Defina como nombre **serverlessfollowup-mobilepush**.

### Suministro de servicios desde la línea de mandatos

En la línea de mandatos, ejecute los mandatos siguientes para suministrar los servicios y recuperar sus credenciales:

   ```sh
   ibmcloud cf create-service cloudantNoSQLDB Lite serverlessfollowup-db
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service tone_analyzer standard serverlessfollowup-tone
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service AppID "Graduated tier" serverlessfollowup-appid
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service imfpush lite serverlessfollowup-mobilepush
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

## Configuración de {{site.data.keyword.mobilepushshort}}
{: #push_notifications}

Cuando un usuario envía un nuevo comentario, la aplicación analiza dicho comentario y envía una notificación al usuario. Es posible que el usuario haya pasado a otra tarea o que no se haya iniciado la app móvil, por lo que la utilización de notificaciones push es una buena forma de comunicarse con el usuario. El servicio {{site.data.keyword.mobilepushshort}} permite enviar notificaciones a los usuarios de iOS o Android a través de una API unificada. En esta sección, configurará el servicio {{site.data.keyword.mobilepushshort}} para la plataforma de destino.

### Configuración de Firebase Cloud Messaging (FCM)
{: java}

   1. En la [consola de Firebase](https://console.firebase.google.com), cree un nuevo proyecto. Defina como nombre **serverlessfollowup**
   2. Vaya a los **Valores** del proyecto
   3. En el separador **General**, añada dos aplicaciones:
      1. una con el nombre de paquete establecido en: **com.ibm.mobilefirstplatform.clientsdk.android.push**
      2. y una con el nombre de paquete establecido en: **serverlessfollowup.app**
   4. Descargue el archivo `google-services.json` que contiene dos aplicaciones definidas de la consola de Firebase y coloque este archivo en la carpeta `android/app` del directorio de extracción.
   5. Localice el ID del remitente y la clave del servidor (también llamada clave de API de ahora en adelante) bajo el separador **Cloud Messaging**.
   6. En el panel de control del servicio Push Notifications, establezca el valor del ID de remitente y clave de API.
   {: java}

### Configuración del servicio Apple Push Notifications (APNs)
{: swift}

1. Vaya al portal de [Apple Developer](https://developer.apple.com/) y registre un ID de app.
2. Cree un certificado SSL de APNs de desarrollo y distribución.
3. Cree un perfil de suministro de desarrollo.
4. Configure la instancia de servicio de {{site.data.keyword.mobilepushshort}} en {{site.data.keyword.Bluemix_notm}}. Consulte [Obtención de credenciales de APNs y configuración del servicio {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-) para ver los pasos detallados.
{: swift}

## Despliegue de un programa de fondo sin servidor
{: #serverless_backend}

Con todos los servicios configurados, ahora puede desplegar el programa de fondo sin servidor. En esta sección se crearán los siguientes artefactos de {{site.data.keyword.openwhisk_short}}:

| Artefacto                      | Tipo                           | Descripción                              |
| ----------------------------- | ------------------------------ | ---------------------------------------- |
| `serverlessfollowup`          | Paquete                        | Un paquete para agrupar las acciones y para conservar todas las credenciales de servicio |
| `serverlessfollowup-cloudant` | Enlace de paquete                | Enlazado al paquete {{site.data.keyword.cloudant_short_notm}} integrado   |
| `serverlessfollowup-push`     | Enlace de paquete                | Enlazado al paquete {{site.data.keyword.mobilepushshort}}  |
| `auth-validate`               | Acción                         | Valida las señales de acceso y de identificación |
| `users-add`                   | Acción                         | Conserva la información del usuario (id, nombre, dirección de correo electrónico, foto, id de dispositivo) |
| `users-prepare-notify`        | Acción                         | Da formato a un mensaje para utilizarlo con {{site.data.keyword.mobilepushshort}} |
| `feedback-put`                | Acción                         | Almacena un comentario de un usuario en la base de datos   |
| `feedback-analyze`            | Acción                         | Analiza un comentario con {{site.data.keyword.toneanalyzershort}}   |
| `users-add-sequence`          | Secuencia expuesta como acción web | `auth-validate` y `users-add`          |
| `feedback-put-sequence`       | Secuencia expuesta como acción web | `auth-validate` y `feedback-put`       |
| `feedback-analyze-sequence`   | Secuencia                       | `read-document` desde {{site.data.keyword.cloudant_short_notm}}, `feedback-analyze`, `users-prepare-notify` y `sendMessage` con {{site.data.keyword.mobilepushshort}} |
| `feedback-analyze-trigger`    | Activador                        | Lo llama {{site.data.keyword.openwhisk_short}} cuando se guarda un comentario en la base de datos |
| `feedback-analyze-rule`       | Regla                           | Enlace el `feedback-analyze-trigger` desencadenante con la secuencia `feedback-analyze-sequence` |

### Compilación del código
{: java}
1. Desde la raíz del directorio de extracción, compile el código de acciones
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### Configuración y despliegue de las acciones
{: java}

2. Copie template.local.env en local.env

   ```sh
   cp template.local.env local.env
   ```
3. Obtenga las credenciales correspondientes a los servicios {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}}
 y {{site.data.keyword.appid_short}} del panel de control de {{site.data.keyword.Bluemix_notm}} (o de la salida de los mandatos ibmcloud que se han ejecutado anteriormente) y sustituya los marcadores de `local.env` por los valores correspondientes. Estas propiedades se inyectarán en un paquete de forma que todas las acciones puedan acceder a la base de datos.
4. Despliegue las acciones en {{site.data.keyword.openwhisk_short}}. `deploy.sh` carga las credenciales de `local.env` para crear las bases de datos de {{site.data.keyword.cloudant_short_notm}} (usuarios, comentarios y tonos) y despliega los artefactos de {{site.data.keyword.openwhisk_short}} para la aplicación.
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   Puede utilizar `./deploy.sh --uninstall` para eliminar los artefactos de {{site.data.keyword.openwhisk_short}} cuando acabe la guía de aprendizaje.
   {: tip}

### Configuración y despliegue de las acciones
{: swift}

1. Copie template.local.env en local.env
   ```sh
   cp template.local.env local.env
   ```
2. Obtenga las credenciales correspondientes a los servicios {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.toneanalyzershort}}, {{site.data.keyword.mobilepushshort}}
 y {{site.data.keyword.appid_short}} del panel de control de {{site.data.keyword.Bluemix_notm}} (o de la salida de los mandatos ibmcloud que se han ejecutado anteriormente) y sustituya los marcadores de `local.env` por los valores correspondientes. Estas propiedades se inyectarán en un paquete de forma que todas las acciones puedan acceder a la base de datos.
3. Despliegue las acciones en {{site.data.keyword.openwhisk_short}}. `deploy.sh` carga las credenciales de `local.env` para crear las bases de datos de {{site.data.keyword.cloudant_short_notm}} (usuarios, comentarios y tonos) y despliega los artefactos de {{site.data.keyword.openwhisk_short}} para la aplicación.

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   Puede utilizar `./deploy.sh --uninstall` para eliminar los artefactos de {{site.data.keyword.openwhisk_short}} cuando acabe la guía de aprendizaje.
   {: tip}

## Configuración y ejecución de una aplicación móvil nativa para que recopile comentarios de usuario
{: #mobile_app}

Nuestras acciones de {{site.data.keyword.openwhisk_short}} están preparadas para nuestra app móvil. Antes de ejecutar la app móvil, tiene que configurar sus valores para que se ejecuten en los servicios que ha creado.

1. Con Android Studio, abra el proyecto situado en la carpeta `android` del directorio de extracción.
2. Edite `android/app/src/main/res/values/credentials.xml` y rellene los espacios en blanco con los valores de sus credenciales. Necesitará los valores de {{site.data.keyword.appid_short}} `tenantId`, {{site.data.keyword.mobilepushshort}} `appGuid` y `clientSecret` y los nombres de la organización y del espacio donde se ha desplegado {{site.data.keyword.openwhisk_short}}, Para el host de API, inicie un indicador de mandatos o un terminal y ejecute este mandato
   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
3. Abra `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` y `LoginAndRegistrationListener.java` y actualice la región del servicio de notificaciones push (BMSClient) y la región de AppID en función de la ubicación en la que se creen las instancias del servicio.
4. Cree el proyecto.
5. Inicie la aplicación en un dispositivo real o con un emulador.
   Para que el emulador reciba notificaciones push, asegúrese de elegir una imagen con las API de Google y de iniciar sesión con una cuenta de Google dentro del emulador.
   {: tip}
6. Observe {{site.data.keyword.openwhisk_short}} en segundo plano
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. En la aplicación, seleccione **Iniciar sesión** para autenticarse con una cuenta de Facebook o Google. Una vez iniciada la sesión, escriba un mensaje de comentario y pulse el botón **Enviar comentarios**. Pocos segundos después de que se haya enviado el comentario, debería recibir notificaciones push en el dispositivo. El texto de la notificación se puede personalizar modificando los documentos de plantilla en la base de datos `moods` de la instancia de servicio de {{site.data.keyword.cloudant_short_notm}}. Utilice el botón **Ver señal** para inspeccionar el acceso y las señales de identificación que genera {{site.data.keyword.appid_short}} tras el inicio de sesión.
{: java}


1. El SDK de cliente de push y otros SDK están disponibles en CocoaPods y Carthage. En esta solución vamos a utilizar CocoaPods.
2. Abra un terminal y ejecute `cd ` a la carpeta `followupapp`. Ejecute el mandato siguiente para instalar las dependencias necesarias.
   ```sh
   pod install
   ```
   {: pre}
3. Abra el archivo con la extensión `.xcworkspace` situada bajo la carpeta `followupapp` del directorio de extracción para iniciar el código en Xcode.
4. Edite el `archivo BMSCredentials.plist` y rellene los espacios en blanco con los valores de las credenciales. Necesitará los valores de `tenantId` de {{site.data.keyword.appid_short}}, `appGuid` y `clientSecret` de {{site.data.keyword.mobilepushshort}} y los nombres de la organización y del espacio donde ha desplegado {{site.data.keyword.openwhisk_short}}. Para el host de API, inicie el terminal y ejecute el mandato

   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
5. Abra `AppDelegate.swift` y actualice la región de servicio de notificaciones push (BMSClient) y la región de AppID en función de la ubicación en la que se crean las instancias de servicio.
6. Cree el proyecto.
7. Inicie la aplicación en un dispositivo real o con un simulador.
8. Observe {{site.data.keyword.openwhisk_short}} en segundo plano ejecutando el mandato siguiente en un terminal.
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. En la aplicación, seleccione **Iniciar sesión** para autenticarse con una cuenta de Facebook o Google. Una vez iniciada la sesión, escriba un mensaje de comentario y pulse el botón **Enviar comentarios**. Pocos segundos después de que se haya enviado el comentario, debería recibir notificaciones push en el dispositivo. El texto de la notificación se puede personalizar modificando los documentos de plantilla en la base de datos `moods` de la instancia de servicio de {{site.data.keyword.cloudant_short_notm}}. Utilice el botón **Ver señal** para inspeccionar el acceso y las señales de identificación que genera {{site.data.keyword.appid_short}} tras el inicio de sesión.
{: swift}

## Eliminación de recursos

1. Utilice `deploy.sh` para eliminar los artefactos de {{site.data.keyword.openwhisk_short}}:

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. Suprima los servicios {{site.data.keyword.cloudant_short_notm}}, {{site.data.keyword.appid_short}}, {{site.data.keyword.mobilepushshort}} y {{site.data.keyword.toneanalyzershort}} de la consola de {{site.data.keyword.Bluemix_notm}}.

## Contenido relacionado

* {{site.data.keyword.appid_short}} proporciona una configuración predeterminada para facilitar la configuración inicial de sus proveedores de identidad. Antes de publicar la app, [actualice la configuración con sus propias credenciales](https://{DomainName}/docs/services/appid?topic=appid-social#social). También estará en condiciones de [personalizar el widget de inicio de sesión](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget).


* Si crea una acción OpenWhisk Swift con un archivo de origen Swift (archivos .Swift de la carpeta `actions`), se tiene que compilar en un binario antes de que se ejecute la acción. Una vez hecho esto, las siguientes llamadas a la acción serán mucho más rápidas hasta que se depure el contenedor que alberga la acción. Este retraso se conoce como retraso de inicio en frío.
  Para evitar el retraso de inicio en frío, puede compilar el archivo Swift en un binario y luego cargarlo en OpenWhisk en un archivo zip. Como necesita la protección de OpenWhisk, la forma más sencilla de crear el binario consiste en crearlo en el mismo entorno en el que se va a ejecutar. Consulte [Empaquetado de una acción como un ejecutable Swift](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions) para ver los pasos a seguir.
{: swift}
