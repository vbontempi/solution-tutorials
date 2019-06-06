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

# Aplicación móvil híbrida con Push Notifications
{: #hybrid-mobile-push-analytics}

Aprenda lo fácil que es crear rápidamente una aplicación Cordova híbrida con un servicio móvil de gran valor, como {{site.data.keyword.mobilepushshort}} en {{site.data.keyword.Bluemix_notm}}.

Apache Cordova es una infraestructura de desarrollo móvil de código abierto. Le permite utilizar las tecnologías web estándar, como HTML5, CSS3 y JavaScript, para el desarrollo entre plataformas. Las aplicaciones se ejecutan dentro de los derivadores destinados a cada plataforma, y se basan en enlaces de API compatibles con estándares para acceder a las prestaciones de cada dispositivo, como sensores, datos, estado de red, etc.

Esta guía de aprendizaje le guía a través de la creación de una aplicación de inicio móvil de Cordova, la adición de un servicio móvil, la configuración el SDK del cliente, la descarga el código de diseño y la mejora de la aplicación.

## Objetivos

* Creación de un proyecto móvil con el servicio {{site.data.keyword.mobilepushshort}}.
* Obtención de credenciales de APNs y de FCM.
* Descarga del código y realización de la configuración necesaria.
* Configuración, envío y supervisión de {{site.data.keyword.mobilepushshort}}.

 ![](images/solution15/Architecture.png)

## Productos

En esta guía de aprendizaje se utilizan los productos siguientes:
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Antes de empezar
{: #prereqs}

- [CLI](https://cordova.apache.org/docs/en/latest/guide/cli/) de Cordova para ejecutar mandatos de Cordova.
- [Requisitos previos](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html) de Cordova-iOS y [requisitos previos](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html) de Cordova-Android
- Cuenta de Google para iniciar una sesión en la consola de Firebase para el ID de remitente y la clave de API de servidor.
- Cuenta de [Apple Developers](https://developer.apple.com/) para enviar notificaciones remotas desde la instancia de servicio de {{site.data.keyword.mobilepushshort}} en {{site.data.keyword.Bluemix_notm}} (el proveedor) a las aplicaciones y dispositivos de iOS.
- Xcode y Android Studio para importar y mejorar el código.

## Creación de un proyecto móvil de Cordova a partir del kit de inicio
{: #get_code}
El panel de control móvil de {{site.data.keyword.Bluemix_notm}} le permite realizar un seguimiento del desarrollo de una app móvil mediante la creación del proyecto a partir de un kit de inicio.
1. Vaya al [Panel de control móvil](https://{DomainName}/developer/mobile/dashboard).
2. Pulse **Kits de inicio** y pulse **Crear app**.
    ![](images/solution15/mobile_dashboard.png)
3. Escriba un nombre de proyecto, que también puede ser el nombre de la app.
4. Seleccione **Cordova** como plataforma y pulse **Crear**.

    ![](images/solution15/create_cordova_project.png)
5. Pulse **Añadir recurso** > Móvil > **Notificaciones Push** y seleccione la ubicación en la que desea suministrar el servicio, el grupo de recursos y el plan de precios **Lite**.
6. Pulse **Crear** para suministrar el servicio {{site.data.keyword.mobilepushshort}}. Se creará una nueva app en el separador **Apps**.

    **Nota:** el servicio {{site.data.keyword.mobilepushshort}} se debería añadir al kit de inicio vacío.

En el siguiente paso, descargará el código de diseño y completará la configuración necesaria.

## Descarga del código y realización de la configuración necesaria
{: #download_code}

Si todavía no ha descargado el código, utilice el panel de control móvil de {{site.data.keyword.Bluemix_notm}} para obtener el código pulsando el botón **Descargar código** bajo Proyectos > **Su proyecto móvil**.

1. En el IDE que elija, vaya a `/platforms/android/project.properties` y sustituya las dos últimas líneas (library.1 y library.2) por las líneas siguientes

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
  Los cambios anteriores son específicos de Android
  {: tip}
2. Para iniciar la app en un emulador de Android, ejecute el mandato siguiente en un terminal o en un indicador de mandatos
```
 $ cordova build android
 $ cordova run android
```
 Si ve un error, inicie un emulador e intente ejecutar el mandato anterior.
 {: tip}
3. Para obtener una vista previa de la app en el simulador de iOS, ejecute los mandatos siguientes en un terminal,

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    Encontrará el nombre del proyecto en el archivo `config.xml` ejecutando el mandato `cordova info`.
    {: tip}

## Obtención de credenciales de FCM y de APNs
{: #obtain_fcm_apns_credentials}

 ### Configuración de Firebase Cloud Messaging (FCM)

  1. En la [consola de Firebase](https://console.firebase.google.com), cree un nuevo proyecto. Defina como nombre **hybridmobileapp**
  2. Vaya a los **Valores** del proyecto
  3. En el separador **General**, añada dos aplicaciones:
       1. una con el nombre de paquete establecido en: **com.ibm.mobilefirstplatform.clientsdk.android.push**
       2. y una con el nombre de paquete establecido en: **io.cordova.hellocordovastarter**
  4. Localice el ID del remitente y la clave del servidor (también llamada clave de API de ahora en adelante) bajo el separador **Cloud Messaging**.
  5. En el panel de control del servicio {{site.data.keyword.mobilepushshort}}, establezca el valor del ID de remitente y clave de API.


Consulte [Obtención de credenciales de FCM](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials) para ver los pasos con mayor detalles.
{: tip}

### Configuración del servicio Apple {{site.data.keyword.mobilepushshort}} (APNs)

  1. Vaya al portal de [Apple Developer](https://developer.apple.com/) y registre un ID de app.
  2. Cree un certificado SSL de APNs de desarrollo y distribución.
  3. Cree un perfil de suministro de desarrollo.
  4. Configure la instancia de servicio de {{site.data.keyword.mobilepushshort}} en {{site.data.keyword.Bluemix_notm}}.

Consulte [Obtención de credenciales de APNs y configuración del servicio {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials) para ver los pasos detallados.
{: tip}

## Configuración, envío y supervisión de {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. En index.js, bajo la función `onDeviceReady`, sustituya los valores `{pushAppGuid}`  y

   `{pushClientSecret} `por las **credenciales** del servicio push: *appGuid* y *clientSecret*.

2. Vaya a Panel de control móvil > Proyectos > Proyecto Cordova, pulse en el servicio {{site.data.keyword.mobilepushshort}} y siga los pasos siguientes.

### APNs - Configuración de la instancia de servicio

Para utilizar el servicio {{site.data.keyword.mobilepushshort}} para enviar notificaciones, cargue los certificados .p12 que ha creado en el paso anterior. Este certificado contiene la clave privada y los certificados SSL necesarios para crear y publicar la aplicación.

**Nota:** después de que el archivo `.cer` se encuentre en el acceso de cadena de claves, expórtelo al sistema para crear un certificado `.p12`.

1. Pulse `{{site.data.keyword.mobilepushshort}}` bajo la sección Servicios o pulse los tres puntos verticales que hay junto al servicio {{site.data.keyword.mobilepushshort}} y seleccione `Abrir panel de control`.
2. En el panel de control de {{site.data.keyword.mobilepushshort}}, debería ver la opción `Configurar` en `Gestionar > Enviar notificaciones`.

Para configurar APNs en la consola de `servicios de notificaciones push`, siga los pasos siguientes:

1. Seleccione `Configurar` en el panel de control de servicios de Push Notification.
2. Elija la opción `Móvil` para actualizar la información del formulario de Credenciales push de APNs.
3. Seleccione `Recinto de pruebas (desarrollo)` o `Producción (distribución)` según corresponda y, a continuación, cargue el certificado `p.12` que ha creado.
4. En el campo Contraseña, especifique la contraseña asociada con el archivo de certificado .p12 y, a continuación, pulse Guardar.

### FCM - Configuración de la instancia de servicio

1. Seleccione **Móvil** y, a continuación, actualice el separador Credenciales de push de GCM/FCM con el ID de remitente/número de proyecto y la clave de API (clave del servidor) que ha creado inicialmente en la consola de Firebase.
2. Pulse **Guardar**. El servicio {{site.data.keyword.mobilepushshort}} ya está configurado.

### Envío de {{site.data.keyword.mobilepushshort}}

1. Seleccione **Enviar notificaciones** y redacte un mensaje seleccionando una opción de envío. Las opciones admitidas son dispositivo por etiqueta, id de dispositivo, id de usuario, dispositivos Android, dispositivos IOS, notificaciones web y todos los dispositivos.
   **Nota:** cuando seleccione la opción **Todos los dispositivos**, todos los dispositivos suscritos a {{site.data.keyword.mobilepushshort}} recibirán notificaciones.

2. En el campo **Mensaje**, redacte el mensaje. Elija configurar los valores opcionales según sea necesario.
3. Pulse **Enviar** y verifique que el dispositivo físico ha recibido la notificación.

### Supervisión de las notificaciones enviadas

Para supervisar las notificaciones enviadas, vaya a la sección **Supervisión**.
El servicio {{site.data.keyword.mobilepushshort}} de IBM ahora amplía sus funciones para supervisar el rendimiento push generando gráficos desde los datos de usuario. Puede utilizar el programa de utilidad para obtener una lista de todos los {{site.data.keyword.mobilepushshort}} enviados o para obtener una lista de todos los dispositivos registrados y para analizar información de forma diaria, semanal o mensual. ![](images/solution6/monitoring_messages.png)

## Contenido relacionado
{: #related_content}

- [Notificaciones basadas en etiquetas](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API de {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Seguridad en {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

