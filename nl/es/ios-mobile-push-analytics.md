---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-08"
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

# Aplicación móvil iOS con Push Notifications
{: #ios-mobile-push-analytics}

Aprenda lo fácil que es crear rápidamente una aplicación iOS Swift con el servicio móvil de gran valor {{site.data.keyword.mobilepushshort}} en {{site.data.keyword.Bluemix_short}}.

Esta guía de aprendizaje le muestra el proceso de creación de una aplicación de inicio móvil, de adición de un servicio móvil, de configuración del SDK del cliente, de importación del código en Xcode y de mejora de la aplicación.
{:shortdesc: .shortdesc}

## Objetivos
{:#objectives}

- Creación de una app móvil con los servicios {{site.data.keyword.mobilepushshort}} y {{site.data.keyword.mobileanalytics_short}} a partir de un kit de inicio de Swift básico.
- Obtención de las credenciales de APNs y configuración de la instancia de servicio de {{site.data.keyword.mobilepushshort}}.
- Descarga del código y configuración del SDK del cliente.
- Envío y supervisión de {{site.data.keyword.mobilepushshort}}.

  ![](images/solution6/Architecture.png)

## Productos
{:#products}

En esta guía de aprendizaje se utilizan los productos siguientes:
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## Antes de empezar
{: #prereqs}

1. Cuenta de [Apple Developers](https://developer.apple.com/) para enviar notificaciones remotas desde la instancia de servicio de {{site.data.keyword.mobilepushshort}} en {{site.data.keyword.Bluemix_short}} (el proveedor) a las aplicaciones y dispositivos de iOS.
2. Xcode para importar y mejorar el código.

## Creación de una app móvil a partir de un kit de inicio de Swift básico
{: #get_code}

1. Vaya al [Panel de control móvil](https://{DomainName}/developer/mobile/dashboard) para crear su `App` a partir de `Kits de inicio` predefinidos.
2. Pulse **Kits de inicio** y desplácese para seleccionar el kit de inicio **Básico**.
    ![](images/solution6/mobile_dashboard.png)
3. Especifique un nombre de app que también será el nombre del proyecto Xcode y el nombre de la app.
4. Seleccione `iOS Swift` como plataforma y pulse **Crear**.     ![](images/solution6/create_mobile_project.png)
5. Pulse **Añadir recurso** > Móvil > **Notificaciones Push** y seleccione la ubicación en la que desea suministrar el servicio, el grupo de recursos y el plan de precios **Lite**.
6. Pulse **Crear** para suministrar el servicio {{site.data.keyword.mobilepushshort}}. Se creará una nueva app en el separador **Apps**.

​      **Nota:** el servicio {{site.data.keyword.mobilepushshort}} debería estar ya añadido con el kit de inicio vacío.

## Descarga del código y configuración de los SDK de cliente
{: #download_code}

Si todavía no ha descargado el código, pulse `Descargar código` en Apps > `Su app móvil` El código descargado se incluye con el SDK de cliente de **{{site.data.keyword.mobilepushshort}}**. El SDK de cliente está disponible en CocoaPods y Carthage. Para esta solución, utilizará CocoaPods.

1. Para instalar CocoaPods en la máquina, abra el `Terminal` y ejecute el mandato siguiente.
   ```
   sudo gem install cocoapods
   ```
   {: pre:}
2. Descomprima el código descargado y, mediante el terminal, vaya a la carpeta descomprimida

   ```
   cd <Name of Project>
   ```
   {: pre:}
3. La carpeta ya incluye un `podfile` con las dependencias necesarias. Ejecute el mandato siguiente para instalar las dependencias (SDK de cliente) y se instalarán las dependencias necesarias.

  ```
  pod install
  ```
  {: pre:}

## Obtención de las credenciales de APNs y configuración de la instancia de servicio de {{site.data.keyword.mobilepushshort}}.
{: #obtain_apns_credentials}

   Para dispositivos y aplicaciones iOS, el servicio de notificaciones push de Apple (APNs) permite a los desarrolladores de aplicaciones enviar notificaciones remotas desde la instancia del servicio {{site.data.keyword.mobilepushshort}} de {{site.data.keyword.Bluemix_short}} (el proveedor) a dispositivos y aplicaciones de iOS. Los mensajes se envían a una aplicación de destino del dispositivo.

   Debe obtener y configurar las credenciales de APNs. Los certificados de APNs se gestionan de forma segura mediante el servicio {{site.data.keyword.mobilepushshort}} y se utilizan para conectarse al servidor de APNs como proveedor.

### Registro de un ID de App

   El ID de app (el identificador de paquete) es un identificador exclusivo que identifica una aplicación específica. Cada aplicación requiere un ID de app. Los servicios como {{site.data.keyword.mobilepushshort}} se configuran en el ID de la app.
   Asegúrese de que dispone de una cuenta de [Apple Developers](https://developer.apple.com/). Es un requisito previo obligatorio.

   1. Vaya al portal de [Apple Developer](https://developer.apple.com/), pulse `Member Center` y seleccione `Certificados, ID y perfiles`.
   2. Vaya a la sección `Identificadores` > ID de app.
   3. En la página `Registro de ID de app`, proporcione el nombre de la app en el campo Nombre de descripción de ID de app. Por ejemplo: ACME {{site.data.keyword.mobilepushshort}}. Proporcione una cadena de caracteres para el prefijo de ID de app.
   4. Para el sufijo de ID de app, seleccione `ID de app explícito` y proporcione un valor de ID de paquete. Se recomienda que proporcione una cadena de estilo de dominio inverso. Por ejemplo: com.ACME.push.
   5. Marque el recuadro de selección `{{site.data.keyword.mobilepushshort}}` y pulse `Continuar`.
   6. Repase los valores y pulse `Registrar` > `Hecho`.
     Se ha registrado el ID de la app.

     ![](images/solution6/push_ios_register_appid.png)

### Creación de un certificado SSL de APNs de desarrollo y distribución
   Para poder obtener un certificado de APNs, debe generar en primer lugar una solicitud de firma de certificado (CSR) y enviarla a Apple, la entidad emisora de certificados (CA). La CSR contiene información que identifica a la empresa y a la clave pública y privada que utilice para firmar su Apple {{site.data.keyword.mobilepushshort}}. A continuación, genere el certificado SSL en el Portal de desarrollador de iOS. El certificado, junto con su clave pública y privada, se almacena en el Acceso de cadena de claves.
   Puede utilizar APNs de dos maneras:

- La modalidad de recinto de seguridad (sandbox) durante el desarrollo y pruebas.
- La modalidad de producción al distribuir aplicaciones mediante App Store (u otros mecanismos de distribución de empresa).

   Debe obtener certificados independientes para los entornos de desarrollo y de distribución. Los certificados están asociados con un ID de App para la app que es el destinatario de las notificaciones remotas. Para la producción, puede crear un máximo de dos certificados. {{site.data.keyword.Bluemix_short}} utiliza los certificados para establecer una conexión SSL con APNs.

   1. Vaya al sitio web de Apple Developer, pulse **Member Center** y seleccione **Certificados, ID y perfiles**.
   2. En el área **Identificadores**, pulse **ID de app**.
   3. En la lista de ID de app, seleccione el ID de app y, a continuación, seleccione `Editar`.
   4. Marque el recuadro de selección **{{site.data.keyword.mobilepushshort}}** y luego, en el panel **Certificado SSL de desarrollo**, pulse **Crear certificado**.

     ![Certificados SSL de notificación Push](images/solution6/certificate_createssl.png)

   5. Cuando aparezca la **pantalla Acerca de la creación de una solicitud de firma de certificado (CSR)**, siga las instrucciones que se muestran para crear un archivo de solicitud de firma de certificado (CSR). Proporcione un nombre significativo que le ayude a identificar si se trata de un certificado de desarrollo (recinto de seguridad) o de distribución (producción); por ejemplo, sandbox-apns-certificate o production-apns-certificate.
   6. Pulse **Continuar** y, en la pantalla Cargar archivo CSR, pulse **Elegir archivo** y seleccione el archivo **CertificateSigningRequest.certSigningRequest** que acaba de crear. Pulse **Continuar**.
   7. En el panel de descarga, instalación y copia de seguridad, pulse Descargar. Se descarga el archivo **aps_development.cer**.
        ![Descargar certificado](images/solution6/push_certificate_download.png)

        ![Generar certificado](images/solution6/generate_certificate.png)
   8. En su Mac, abra **Acceso de cadena de claves**, **Archivo**, **Importar** y seleccione el archivo .cer descargado para instalarlo.
   9. Pulse con el botón derecho en el nuevo certificado y en la clave privada, seleccione **Exportar** y cambie el **Formato de archivo** por el formato de intercambio de información personal (formato `.p12`).
     ![Exportar certificado y claves](images/solution6/keychain_export_key.png)
   10. En el campo **Guardar como**, escriba un nombre significativo para el certificado. Por ejemplo, `sandbox_apns.p12` o **production_apns.p12**, y luego pulse Guardar.
     ![Exportar certificado y claves](images/solution6/certificate_p12v2.png)
   11. En el campo **Especificar una contraseña**, escriba una contraseña para proteger los elementos exportados y, a continuación, pulse Aceptar. Puede utilizar esta contraseña para configurar los valores de APNs en la consola de servicio de {{site.data.keyword.mobilepushshort}}.
![Exportar certificado y claves](images/solution6/export_p12.png)
   12. **Key Access.app** le solicita que exporte su clave desde la pantalla **Cadena de claves**. Especifique la contraseña de administración para Mac para permitir al sistema exportar estos elementos y, a continuación, seleccione la opción `Permitir siempre`. Se generará un certificado `.p12` en el escritorio.

      En el caso de SSL de producción, en el panel **Certificado SSL de producción**, pulse **Crear certificado** y repita los pasos del 5 al 12.
      {:tip}

### Creación de un perfil de suministro de desarrollo
   El perfil de suministro funciona con el ID de App para determinar qué dispositivos pueden instalar y ejecutar la app y a qué servicios puede acceder la app. Para cada ID de App, cree dos perfiles de suministro: uno para desarrollo y otro para distribución. Xcode utiliza el perfil de suministro de desarrollo para determinar qué desarrolladores están permitidos para crear la aplicación y qué dispositivos están permitidos para probarse en la aplicación.

   Asegúrese de que ha registrado un ID de App, de que lo ha habilitado para el servicio de {{site.data.keyword.mobilepushshort}} y de que lo ha configurado para utilizar un certificado SSL de APNs de desarrollo y producción.

   Cree un perfil de suministro de desarrollo, como se indica a continuación:

   1. Vaya al portal de [Apple Developer](https://developer.apple.com/), pulse `Member Center` y seleccione `Certificados, ID y perfiles`.
   2. Vaya a [Mac Developer Library](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site), desplácese a la sección `Creación de perfiles de suministro de desarrollo` y siga las instrucciones para crear un perfil de desarrollo.
     **Nota:** cuando configure un perfil de suministro de desarrollo, seleccione las opciones siguientes:

     - **Desarrollo de apps de iOS**
     - **Para apps iOS y watchOS**

### Creación de un perfil de suministro de distribución del almacén
   Utilice el perfil de suministro del almacén para enviar la app para su distribución a la App Store.

   1. Vaya al portal de [Apple Developer](https://developer.apple.com/), pulse `Member Center` y seleccione `Certificados, ID y perfiles`.
   2. Efectúe una doble pulsación en el perfil de suministro descargado para instalarlo en Xcode.
     Después de obtener las credenciales, el siguiente paso es [Configurar una instancia de servicio](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2).

### Configuración de la instancia de servicio

   Para utilizar el servicio {{site.data.keyword.mobilepushshort}} para enviar notificaciones, cargue los certificados .p12 que ha creado en el paso anterior. Este certificado contiene la clave privada y los certificados SSL necesarios para crear y publicar la aplicación.

   **Nota:** después de que el archivo `.cer` se encuentre en el acceso de cadena de claves, expórtelo al sistema para crear un certificado `.p12`.

1. Pulse {{site.data.keyword.mobilepushshort}} bajo la sección Servicios o pulse los tres puntos verticales que hay junto al servicio {{site.data.keyword.mobilepushshort}} y seleccione `Abrir panel de control`.
2. En el panel de control de {{site.data.keyword.mobilepushshort}}, debería ver la opción `Configurar` en `Gestionar`.

Para configurar APNs en la consola de `servicios de notificaciones push`, siga los pasos siguientes:

1. Elija la opción `Móvil` para actualizar la información del formulario de Credenciales push de APNs.
2. Seleccione `Servidor de APNs de recinto de pruebas/desarrollo` o `Servidor de APNs de producción` según corresponda y, a continuación, cargue el certificado `.p12` que ha creado.
3. En el campo Contraseña, especifique la contraseña asociada con el archivo de certificado .p12 y, a continuación, pulse Guardar.

![](images/solution6/Mobile_push_configure.png)

## Configuración, envío y supervisión de {{site.data.keyword.mobilepushshort}}
{: #configure_push}

1. Encontrará el código de iniciación de envíos (bajo `aplicación func`) y el código de registro de notificaciones en `AppDelegate.swift`. Especifique un USER_ID exclusivo (opcional).
2. Ejecute la app en un dispositivo físico, ya que las notificaciones no se pueden enviar a un simulador de iPhone.
3. Abra el servicio {{site.data.keyword.mobilepushshort}} bajo `Servicios móviles` > **Servicios existentes** del panel de control móvil de {{site.data.keyword.Bluemix_short}} y, para enviar {{site.data.keyword.mobilepushshort}} básico, siga estos pasos:
  * Seleccione `Mensajes` y componga un mensaje eligiendo una opción Enviar a. Las opciones admitidas son Dispositivo por etiqueta, ID de dispositivo, ID de usuario, dispositivos Android, dispositivos iOS, Notificaciones web, Todos los dispositivos y otros navegadores.

       **Nota:** cuando seleccione la opción Todos los dispositivos, todos los dispositivos suscritos a {{site.data.keyword.mobilepushshort}} recibirán notificaciones.
  * En el campo `Mensaje`, redacte el mensaje. Elija configurar los valores opcionales según sea necesario.
  * Pulse `Enviar` y verifique que los dispositivos físicos han recibido la notificación.     ![](images/solution6/send_notifications.png)

4. Debería ver una notificación en su iPhone.

   ![](images/solution6/iphone_notification.png)

5. Para supervisar las notificaciones enviadas, vaya a `Supervisión` en el servicio {{site.data.keyword.mobilepushshort}}.

El servicio {{site.data.keyword.mobilepushshort}} de IBM ahora amplía sus funciones para supervisar el rendimiento push generando gráficos desde los datos de usuario. Puede utilizar el programa de utilidad para obtener una lista de todos los {{site.data.keyword.mobilepushshort}} enviados o para obtener una lista de todos los dispositivos registrados y para analizar información de forma diaria, semanal o mensual. ![](images/solution6/monitoring_messages.png)

## Contenido relacionado
{: #related_content}

- [Notificaciones basadas en etiquetas](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [API de {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [Seguridad en {{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)
