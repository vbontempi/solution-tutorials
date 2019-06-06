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

# Aplicación móvil nativa de Android con Push Notifications
{: #android-mobile-push-analytics}

Aprenda lo fácil que es crear rápidamente una aplicación Android nativa con el servicio móvil de gran valor {{site.data.keyword.mobilepushshort}} en {{site.data.keyword.Bluemix_notm}}.

Esta guía de aprendizaje le muestra el proceso de creación de una aplicación de inicio móvil, de adición de un servicio móvil, de configuración del SDK del cliente, de importación del código en Android Studio y de mejora de la aplicación.

## Objetivos
{: #objectives}

* Creación de una app móvil con el servicio {{site.data.keyword.mobilepushshort}}.
* Obtención de credenciales de FCM.
* Descarga del código y realización de la configuración necesaria.
* Configuración, envío y supervisión de {{site.data.keyword.mobilepushshort}}.

![](images/solution9/Architecture.png)

## Productos
{: #products}

En esta guía de aprendizaje se utilizan los productos siguientes:
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Antes de empezar
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html) para importar y mejorar el código.
- Cuenta de Google para iniciar una sesión en la consola de Firebase para el ID de remitente y la clave de API de servidor.

## Creación de una app móvil de Android a partir del kit de inicio
{: #get_code}
El panel de control móvil de {{site.data.keyword.Bluemix_notm}} le permite realizar un seguimiento del desarrollo de una app móvil mediante la creación de la app a partir de un kit de inicio.
1. Vaya al [Panel de control móvil](https://{DomainName}/developer/mobile/dashboard).
2. Pulse **Kits de inicio** y pulse **Crear app**.
    ![](images/solution9/mobile_dashboard.png)
3. Escriba un nombre de app, que también puede ser el nombre del proyecto android.
4. Seleccione **Android** como plataforma y pulse **Crear**.

    ![](images/solution9/create_mobile_project.png)
5. Pulse **Añadir recurso** > Móvil > **Notificaciones Push** y seleccione la ubicación en la que desea suministrar el servicio, el grupo de recursos y el plan de precios **Lite**.
6. Pulse **Crear** para suministrar el servicio {{site.data.keyword.mobilepushshort}}. Se creará una nueva app en el separador **Apps**.

    **Nota:** el servicio {{site.data.keyword.mobilepushshort}} debería estar añadido con el kit de inicio vacío.
    En el paso siguiente, obtendrá las credenciales de Firebase Cloud Messaging (FCM).

En el siguiente paso, descargará el código de diseño y configurará el SDK de Push Android.

## Descarga del código y realización de la configuración necesaria
{: #download_code}

Si todavía no ha descargado el código, utilice el panel de control móvil de {{site.data.keyword.Bluemix_notm}} para obtener el código pulsando el botón **Descargar código** bajo Apps > **Su app móvil**.
El código descargado se suministra con el SDK de cliente de **{{site.data.keyword.mobilepushshort}}** incluido. El SDK de cliente está disponible en Gradle y Maven. En esta guía de aprendizaje, utilizará **Gradle**.

1. Inicie Android Studio > **Abra un proyecto de Android Studio existente** y apunte al código descargado.
2. Se activará automáticamente una compilación de **Gradle** y se descargarán todas las dependencias.
3. Añada la dependencia de **Servicios de Google Play** al archivo de nivel de módulo `build.gradle (Module: app)` al final, después de `dependencies{.....}`
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. Copie el archivo `google-services.json` que ha creado y descárguelo en el directorio raíz del módulo de aplicación de Android. Tenga en cuenta que el archivo `google-service.json` incluye los nombres de paquete añadidos.
5. Todos los permisos necesarios están dentro del archivo `AndroidManifest.xml` y sus dependencias. Los servicios Push y Analytics están incluidos en **build.gradle (Module: app)**.
6. El servicio de intenciones **Firebase Cloud Messaging (FCM)** y los filtros de intenciones correspondientes a las notificaciones de sucesos `RECEIVE` y `REGISTRATION` están incluidos en `AndroidManifest.xml`

## Obtención de credenciales de FCM
{: #obtain_fcm_credentials}

Firebase Cloud Messaging (FCM) es la pasarela utilizada para suministrar el servicio {{site.data.keyword.mobilepushshort}} a dispositivos Android, al navegador Google Chrome y a las apps y extensiones de Chrome. Para configurar el servicio {{site.data.keyword.mobilepushshort}} en la consola, debe obtener credenciales de FCM (ID de remitente y clave de API).

La clave de la API se almacena de forma segura y la utiliza el servicio {{site.data.keyword.mobilepushshort}} para conectarse al servidor de FCM y el ID de remitente (número de proyecto) lo utilizará el SDK de Android y el SDK de JS para Google Chrome y Mozilla Firefox en el lado de cliente. Para configurar FCM y obtener sus credenciales, lleve a cabo los pasos siguientes:

1. Vaya a la [consola de Firebase](https://console.firebase.google.com/?pli=1); se necesita una cuenta de usuario de Google.
2. Seleccione **Añadir proyecto**.
3. En la ventana **Crear un proyecto**, especifique un nombre de proyecto, elija un país/región y pulse **Crear proyecto**.
4. En el panel de navegación de la izquierda, seleccione **Configuración** (pulse el icono de configuración situado junto a **Visión general**)> **Configuración del proyecto**.
5. Seleccione el separador Cloud Messaging para obtener las credenciales del proyecto: una Clave de API de servidor y un ID de remitente.
    **Nota:**  la clave de servidor que se muestra en FCM es la misma que la clave de API del servidor.
    ![](images/solution9/fcm_console.png)

También necesitaría generar el archivo `google-services.json`. Realice los pasos siguientes:

1. En la consola de Firebase, pulse el icono **Configuración del proyecto** > separador **General** bajo el proyecto que ha creado anteriormente y seleccione **Añadir Firebase a la app Android**

    ![](images/solution9/firebase_project_settings.png)
2. En la ventana modal de la app **Añadir Firebase a Android**, añada **com.ibm.mobilefirstplatform.clientsdk.android.push** como nombre de paquete para registrar el sdk de android de {{site.data.keyword.mobilepushshort}}. Los campos Apodo de app y SHA-1 son opcionales. Pulse **REGISTRAR APP** > **Continuar** > **Finalizar**.

    ![](images/solution9/add_firebase_to_your_app.png)

3. Pulse **AÑADIR APP** > **Añadir Firebase a la app**.  Incluya el nombre de paquete de la aplicación escribiendo el nombre de paquete **com.ibm.mysampleapp** y añada Firebase a la ventana de la app de Android. Los campos Apodo de app y SHA-1 son opcionales. Pulse **REGISTRAR APP** > Continuar > Finalizar.
     **Nota:** encontrará el nombre del paquete de la aplicación en el archivo `AndroidManifest.xml` una vez que descargue el código.
4. Descargue el archivo de configuración más reciente `google-services.json` en **Sus apps**.

    ![](images/solution9/google_services.png)

    **Nota**: FCM es la nueva versión de Google Cloud Messaging (GCM). Asegúrese de que utiliza las credenciales de FCM para nuevas apps. Las apps existentes seguirán funcionando con las configuraciones de GCM.

*Los pasos y la interfaz de usuario de la consola de Firebase están sujetos a cambios; consulte la documentación de Google para la parte de Firebase si es necesario*

## Configuración, envío y supervisión de {{site.data.keyword.mobilepushshort}}

{: #configure_push}

1. El SDK de {{site.data.keyword.mobilepushshort}} ya se ha importado en la app y el código de inicialización de Push se encuentra en el archivo `MainActivity.java`.

    **Nota:** las credenciales de servicio forman parte del archivo `/res/values/credentials.xml`.
2. El registro de las notificaciones se produce en `MainActivity.java`.  (Opcional) Especifique un USER_ID exclusivo.
3. Ejecute la app en un dispositivo físico o en un emulador para recibir notificaciones.
4. Abra el servicio {{site.data.keyword.mobilepushshort}} bajo **Servicios móviles** > **Servicios existentes** del panel de control móvil de {{site.data.keyword.Bluemix_notm}} y, para enviar {{site.data.keyword.mobilepushshort}} básico, siga estos pasos:
   - Pulse **Gestionar** > **Configurar**.
   - Seleccione **Móvil** y, a continuación, actualice el separador Credenciales de push de GCM/FCM con el ID de remitente/número de proyecto y la clave de API (clave del servidor) que ha creado inicialmente en la consola de Firebase.

     ![](images/solution9/configure_push_notifications.png)
   - Pulse **Guardar**. El servicio {{site.data.keyword.mobilepushshort}} ya está configurado.
   - Seleccione **Enviar notificaciones** y redacte un mensaje seleccionando una opción de envío. Las opciones admitidas son dispositivo por etiqueta, id de dispositivo, id de usuario, dispositivos android, dispositivos IOS, notificaciones web y todos los dispositivos.
     **Nota:** cuando seleccione la opción **Todos los dispositivos**, todos los dispositivos suscritos a {{site.data.keyword.mobilepushshort}} recibirán notificaciones.
   - En el campo **Mensaje**, redacte el mensaje. Elija configurar los valores opcionales según sea necesario.
   - Pulse **Enviar** y verifique que el dispositivo físico ha recibido la notificación.

     ![](images/solution9/android_send_notifications.png)
5. Debería ver una notificación en su dispositivo Android.

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. Para supervisar las notificaciones enviadas, vaya a **Supervisión** en el servicio {{site.data.keyword.mobilepushshort}}.
     El servicio {{site.data.keyword.mobilepushshort}} de IBM ahora amplía sus funciones para supervisar el rendimiento push generando gráficos desde los datos de usuario. Puede utilizar el programa de utilidad para obtener una lista de todos los {{site.data.keyword.mobilepushshort}} enviados o para obtener una lista de todos los dispositivos registrados y para analizar información de forma diaria, semanal o mensual. ![](images/solution6/monitoring_messages.png)

## Contenido relacionado
{: #related_content}
- [Personalización de los valores de {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [Notificaciones basadas en etiquetas](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API de {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Seguridad en {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)

