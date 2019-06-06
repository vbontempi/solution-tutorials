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

# Aplicación de seguridad de extremo a extremo a una aplicación de nube
{: #cloud-e2e-security}

Ninguna arquitectura de aplicación está completa sin una visión clara de los riesgos de seguridad potenciales y de cómo protegerse frente a dicha amenaza. Los datos de la aplicación son un recurso crítico que no se puede perder, comprometer ni robar. Además, los datos deberían protegerse tanto en reposo como en tránsito mediante técnicas de cifrado. El cifrado de los datos en reposo impide que se distribuya la información, incluso si se los datos pierden o son robados. El cifrado de datos en tránsito (por ejemplo, a través de Internet) mediante métodos como HTTPS, SSL y TLS evita que sean espiados y los ataques de tipo hombre en el medio.

Autenticar y autorizar el acceso de los usuarios a recursos específicos es otro requisito común para muchas aplicaciones. Es posible que sea necesario dar soporte a diferentes esquemas de autenticación: clientes y proveedores que utilizan identidades sociales, socios de los directorios alojados en la nube y empleados de un proveedor de identidades de una organización.

En esta guía de aprendizaje se muestran los servicios de seguridad clave disponibles en el catálogo de {{site.data.keyword.cloud}} y cómo utilizarlos conjuntamente. Una aplicación que proporciona compartición de archivos pondrá en práctica los conceptos de seguridad.
{:shortdesc}

## Objetivos
{: #objectives}

* Cifrado del contenido de los grupos de almacenamiento con sus propias claves de cifrado
* Necesidad de que los usuarios se autentiquen antes de acceder a una aplicación
* Supervisión y auditoría de llamadas de API relacionadas con la seguridad y otras acciones a través de los servicios de nube

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.registryshort_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker)
* [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/key-protect)
* Opcional: [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)

Esta guía de aprendizaje requiere una [cuenta que no sea Lite](https://{DomainName}/docs/account?topic=account-accounts#accounts) y puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

En la guía de aprendizaje se utiliza una aplicación de ejemplo que permite a los grupos de usuarios cargar archivos a una agrupación de almacenamiento común y proporcionar acceso a dichos archivos mediante enlaces que se pueden compartir. La aplicación está escrita en Node.js y se despliega como contenedor de Docker en {{site.data.keyword.containershort_notm}}. Aprovecha varios servicios y funciones relacionados con la seguridad para mejorar la seguridad de la aplicación.

<p style="text-align: center;">

  ![Arquitectura](images/solution34-cloud-e2e-security/Architecture.png)
</p>

1. El usuario se conecta a la aplicación.
2. Si se utiliza un dominio personalizado y un certificado TLS, {{site.data.keyword.cloudcerts_short}} gestiona y despliega el certificado.
3. {{site.data.keyword.appid_short}} protege la aplicación y redirige al usuario a la página de autenticación. Los usuarios también pueden registrarse.
4. La aplicación se ejecuta en un clúster de Kubernetes desde una imagen almacenada en {{site.data.keyword.registryshort_notm}}. Esta imagen se explora automáticamente para detectar vulnerabilidades.
5. Los archivos cargados se almacenan en {{site.data.keyword.cos_short}} con los metadatos adjuntos almacenados en {{site.data.keyword.cloudant_short_notm}}.
6. Los grupos de almacenamiento de archivos aprovechan una clave proporcionada por el usuario para cifrar los datos.
7. {{site.data.keyword.cloudaccesstrailshort}} registra las actividades de gestión de la aplicación.

## Antes de empezar
{: #prereqs}

1. Instale todas las herramientas de línea de mandatos (CLI) necesarias [siguiendo estos pasos](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).
2. Asegúrese de tener la última versión de los plugins utilizados en esta guía de aprendizaje; utilice `ibmcloud plugin update --all` para actualizar.

## Creación de servicios
{: #setup}

### Decida dónde desea desplegar la aplicación

1. Identifique la **ubicación**, la **organización y el espacio de Cloud Foundry** y el **grupo de recursos** en el que desplegará la aplicación y sus recursos.

### Captura de actividades de usuario y de aplicación 
{: #activity-tracker }

El servicio {{site.data.keyword.cloudaccesstrailshort}} registra las actividades iniciadas por el usuario que cambian el estado de un servicio en {{site.data.keyword.Bluemix_notm}}. Al final de esta guía de aprendizaje, revisará los sucesos que se han generado completando los pasos de la guía de aprendizaje.

1. Acceda al catálogo de {{site.data.keyword.Bluemix_notm}} y cree una instancia de [{{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/catalog/services/activity-tracker). Tenga en cuenta que solo puede haber una instancia de {{site.data.keyword.cloudaccesstrailshort}} por espacio de Cloud Foundry.
2. Cuando se haya creado la instancia, cambie su nombre por **secure-file-storage-activity-tracker**.
3. Para ver todos los sucesos de seguimiento de actividad, asegúrese de que tiene permisos asignados:
   1. Vaya a [Identidad y acceso > Usuarios](https://{DomainName}/iam/#/users).
   2. Seleccione su nombre de usuario en la lista.
   3. En **Políticas de acceso**, si aún no dispone de una, cree una política para el servicio {{site.data.keyword.loganalysisshort_notm}} con el rol de **Visor** en la ubicación en la que se ha creado el servicio.
   4. En **Acceso a Cloud Foundry**, asegúrese de que tiene el rol **Desarrollador** en el espacio de Cloud Foundry donde se ha suministrado {{site.data.keyword.cloudaccesstrailshort}}. Si no es así, trabaje con el gestor de Cloud Foundry de su organización para asignar el rol.

Encontrará instrucciones detalladas sobre cómo configurar permisos en la [documentación de {{site.data.keyword.cloudaccesstrailshort}}](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-grant_permissions#grant_iam_policy).
{: tip}

### Creación de un clúster para la aplicación

{{site.data.keyword.containershort_notm}} proporciona un entorno para desplegar apps de alta disponibilidad en contenedores de Docker que se ejecutan en clústeres de Kubernetes.

Sáltese esta sección si tiene un clúster **Estándar** existente que desea reutilizar con esta guía de aprendizaje.
{: tip}

1. Acceda a la [página de creación del clúster](https://{DomainName}/containers-kubernetes/catalog/cluster/create).
   1. Establezca como **Ubicación** la que ha utilizado en el paso anterior.
   2. Establezca **Tipo de clúster** en **Estándar**.
   3. Establezca **Disponibilidad** en **Una sola zona**.
   4. Seleccione una **Zona maestra**.
2. Conserve la **Versión de Kubernetes** y el **Aislamiento de hardware** predeterminados.
3. Si tiene previsto desplegar solo esta guía de aprendizaje en este clúster, establezca **Nodos de trabajo** en **1**.
4. Establezca el **Nombre de clúster** en **secure-file-storage-cluster**.
5. Pulse el botón **Crear clúster**.

Mientras se está suministrando el clúster, va a crear los otros servicios que necesita la guía de aprendizaje.

### Utilización de sus propias claves de cifrado

{{site.data.keyword.keymanagementserviceshort}} le ayuda a suministrar claves cifradas para apps en servicios de {{site.data.keyword.Bluemix_notm}}. {{site.data.keyword.keymanagementserviceshort}} y {{site.data.keyword.cos_full_notm}} [funcionan conjuntamente para proteger sus datos en reposo](https://{DomainName}/docs/services/key-protect/integrations?topic=key-protect-integrate-cos#integrate-cos). En esta sección, creará una clave raíz para el grupo de almacenamiento.

1. Cree una instancia de [{{site.data.keyword.keymanagementserviceshort}}](https://{DomainName}/catalog/services/kms).
   * Defina como nombre **secure-file-storage-kp**.
   * Seleccione el grupo de recursos donde se va a crear la instancia de servicio.
2. En **Gestionar**, pulse el botón **Añadir clave** para crear una nueva clave raíz. Se utilizará para cifrar el contenido del grupo de almacenamiento.
   * Defina como nombre **secure-file-storage-root-enckey**.
   * Establezca el tipo de clave en **Clave raíz**.
   * A continuación, **Genere la clave**.

Para traer su propia clave (BYOK), [importe una clave raíz existente](https://{DomainName}/docs/services/key-protect?topic=key-protect-import-root-keys#import-root-keys).
{: tip}

### Configuración del almacenamiento para los archivos de usuario

La aplicación de compartición de archivos guarda los archivos en un grupo de {{site.data.keyword.cos_short}}. La relación entre los archivos y los usuarios se almacena como metadatos en una base de datos de {{site.data.keyword.cloudant_short_notm}}. En esta sección, creará y configurará estos servicios.

#### Un grupo para el contenido

1. Cree una instancia de [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage).
   * Establezca el **nombre** en **secure-file-storage-cos**.
   * Utilice el mismo **grupo de recursos** que ha usado para los servicios anteriores.
2. Seleccione **Credenciales de servicio**, cree una *Nueva credencial*.
   * Establezca el **nombre** en **secure-file-storage-cos-acckey**.
   * Establezca **Rol** en **Escritor**.
   * No especifique un **ID de servicio**.
   * Establezca los **Parámetros de configuración en línea** en **{"HMAC":true}**. Esto es necesario para generar los URL prefirmados.
   * Pulse
**Añadir**.
   * Anote las credenciales pulsando **Ver credenciales**. Las
necesitará en un paso posterior.
3. Pulse **Punto final** en el menú: establezca para **Resistencia** el valor **Regional** y para **Ubicación** la ubicación de destino. Copie el punto final de servicio **Público**. Se utilizará más adelante en la configuración de la aplicación.

Antes de crear el grupo, otorgará el acceso **secure-file-storage-cos** a la clave raíz almacenada en **secure-file-storage-kp**.

1. Vaya a [Identidad y acceso > Autorizaciones](https://{DomainName}/iam/#/authorizations) en la consola de {{site.data.keyword.cloud_notm}}.
2. Pulse el botón **Crear**.
3. En el menú **Servicio de origen**, seleccione **Almacenamiento de objetos de nube**.
4. En el menú **Instancia de servicio de origen**, seleccione el servicio **secure-file-storage-cos** que se ha creado previamente.
5. En el menú **Servicio de destino**, seleccione **Protección de claves**.
6. En el menú **Instancia de servicio de destino**, seleccione el servicio **secure-file-storage-kp** que se debe autorizar.
7. Habilite el rol **Lector**.
8. Pulse el botón **Autorizar**.

Por último, cree el grupo.

1. Acceda a la instancia de servicio **secure-file-storage-cos** desde la [Lista de recursos](https://{DomainName}/resources).
2. Pulse **Crear grupo**.
   1. Establezca el **nombre** en un valor exclusivo, como **&lt;your-initials&gt;-secure-file-upload**.
   2. Establezca **Resistencia** en **Regional**.
   3. Establezca **Ubicación** en la misma ubicación en la que ha creado el servicio **secure-file-storage-kp**.
   4. Establezca **Clase de almacenamiento** en **Estándar**
3. Marque el recuadro de selección para **Añadir claves de protección de claves**.
   1. Seleccione el servicio **secure-file-storage-kp**.
   2. Seleccione **secure-file-storage-root-enckey** como clave.
4. Pulse **Crear grupo**.

#### Relaciones de correlación de base de datos entre los usuarios y sus archivos

La base de datos de {{site.data.keyword.cloudant_short_notm}} contendrá metadatos para todos los archivos cargados desde la aplicación.

1. Cree una instancia de [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB).
   * Establezca el **nombre** en **secure-file-storage-cloudant**.
   * Establezca la ubicación.
   * Utilice el mismo **grupo de recursos** que ha usado para los servicios anteriores.
   * Establezca **Métodos de autenticación disponibles** en **Utilizar solo IAM**.
2. En **Credenciales de servicio**, cree una *Nueva credencial*.
   * Establezca el **nombre** en **secure-file-storage-cloudant-acckey**.
   * Establezca **Rol** en **Gestor**
   * Conserve los valores predeterminados para los campos *opcionales*
   * **Añadir**.
3. Anote las credenciales pulsando **Ver credenciales**. Las
necesitará en un paso posterior.
4. En **Gestionar**, inicie el panel de control de Cloudant.
5. Pulse **Crear base de datos** para crear una base de datos denominada **secure-file-storage-metadata**.

### Autenticación de usuarios

Con {{site.data.keyword.appid_short}}, puede proteger los recursos y añadir autenticación a las aplicaciones. {{site.data.keyword.appid_short}} [se integra](https://{DomainName}/docs/containers?topic=containers-ingress_annotation#appid-auth) con {{site.data.keyword.containershort_notm}} para autenticar los usuarios que acceden a las aplicaciones desplegadas en el clúster.

1. Cree una instancia de [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID).
   * Establezca el **Nombre de servicio** en **secure-file-storage-appid**.
   * Utilice la misma **ubicación** y el mismo **grupo de recursos** que para los servicios anteriores.
2. En **Proveedores de identidad / Gestionar**, en el separador **Valores de autenticación**, añada un **URL de redirección web** que señale al dominio que utilizará para la aplicación. Por ejemplo, si el subdominio Ingress del clúster es `<cluster-name>.us-south.containers.appdomain.cloud`, el URL de redirección será `https://secure-file-storage.<cluster-name>.us-south.containers.appdomain.cloud/appid_callback`. {{site.data.keyword.appid_short}} requiere que el URL de redirección web sea **https**. Puede ver el subdominio Ingress en el panel de control del clúster o con `ibmcloud ks clúster-get <cluster-name>`.

Debe personalizar los proveedores de identidades utilizados así como el inicio de sesión y la experiencia de gestión de los usuarios en el panel de control de {{site.data.keyword.appid_short}}. En esta guía de aprendizaje se utilizan los valores predeterminados para simplificar. En el caso de un entorno de producción, considere la posibilidad de utilizar la autenticación de varios factores (MFA) y reglas avanzadas de contraseña.
{: tip}

## Despliegue de la app

Se han configurado todos los servicios. En esta sección desplegará la aplicación de la guía de aprendizaje en el clúster.

### Obtención del código

1. Obtenga el código de la aplicación:
   ```sh
   git clone https://github.com/IBM-Cloud/secure-file-storage
   ```
   {: codeblock}
2. Vaya al directorio **secure-file-storage**:
   ```sh
   cd secure-file-storage
   ```
   {: codeblock}

### Creación de la imagen de Docker

1. [Cree la imagen de Docker](https://{DomainName}/docs/services/Registry?topic=registry-registry_images_#registry_images_creating) en {{site.data.keyword.registryshort_notm}}.
   - Localice el punto final de registro con `ibmcloud cr info`, como por ejemplo **us**.icr.io o **uk**.icr.io.
   - Cree un espacio de nombres para almacenar la imagen del contenedor.
      ```sh
      ibmcloud cr namespace-add secure-file-storage-namespace
      ```
   - Utilice **secure-file-storage** como nombre de la imagen.

   ```sh
   ibmcloud cr build -t <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest .
   ```
   {: codeblock}

### Especificación de credenciales y de valores de configuración

1. Copie `credentials.template.env` en `credentials.env`:
   ```sh
   cp credentials.template.env credentials.env
   ```
   {: codeblock}
2. Edite `credentials.env` y rellene los espacios en blanco con estos valores:
   * el punto final regional de servicio de {{site.data.keyword.cos_short}}, el nombre del grupo, las credenciales creadas para **secure-file-storage-cos**,
   * y las credenciales para **secure-file-storage-cloudant**.
3. Copie `secure-file-storage.template.yaml` en `secure-file-storage.yaml`:
   ```sh
   cp secure-file-storage.template.yaml secure-file-storage.yaml
   ```
   {: codeblock}
4. Edite `secure-file-storage.yaml` y sustituya los marcadores (`$IMAGE_PULL_SECRET`, `$REGISTRY_URL`, `$REGISTRY_NAMESPACE`, `$IMAGE_NAME`, `$TARGET_NAMESPACE`, `$INGRESS_SUBDOMAIN`, `$INGRESS_SECRET`) por los valores correctos. Por ejemplo, suponiendo que la aplicación se despliegue en el espacio de nombres _default_ Kubernetes:

| Variable | Valor | Descripción |
| -------- | ----- | ----------- |
| `$IMAGE_PULL_SECRET` | Mantenga las líneas comentadas en el archivo .yaml | Un secreto para acceder al registro.  |
| `$REGISTRY_URL` | *us.icr.io* | El registro en el que se ha creado la imagen en la sección anterior. |
| `$REGISTRY_NAMESPACE` | *secure-file-storage-namespace* | El espacio de nombres del registro en el que se ha creado la imagen en la sección anterior. |
| `$IMAGE_NAME` | *secure-file-storage* | El nombre de la imagen de Docker. |
| `$TARGET_NAMESPACE` | *default* | El espacio de nombres Kubernetes en el que se enviará la app. |
| `$INGRESS_SUBDOMAIN` | *secure-file-stora-123456.us-south.containers.appdomain.cloud* | Recupere este valor de la página de visión general del clúster o con `ibmcloud ks cluster-get secure-file-storage-cluster`. |
| `$INGRESS_SECRET` | *secure-file-stora-123456* | Recupere este valor de la página de visión general del clúster o con `ibmcloud ks cluster-get secure-file-storage-cluster`. |

`$IMAGE_PULL_SECRET` solo es necesario si desea utilizar un espacio de nombres de Kubernetes distinto del predeterminado. Esto requiere una configuración adicional de Kubernetes (es decir, la [creación de un secreto de registro de Docker en el nuevo espacio de nombres](https://{DomainName}/docs/containers?topic=containers-images#other)).
{: tip}

### Despliegue en el clúster

1. Recupere la configuración del clúster y establezca la variable de entorno KUBECONFIG.
   ```sh
   $(ibmcloud ks cluster-config --export secure-file-storage-cluster)
   ```
   {: codeblock}
2. Cree el secreto utilizado por la aplicación para obtener las credenciales de servicio:
   ```sh
   kubectl create secret generic secure-file-storage-credentials --from-env-file=credentials.env
   ```
   {: codeblock}
3. Enlace la instancia de servicio de {{site.data.keyword.appid_short_notm}} al clúster.
   ```sh
   ibmcloud ks cluster-service-bind --cluster secure-file-storage-cluster --namespace default --service secure-file-storage-appid
   ```
   {: codeblock}
   Si tiene varios servicios con el mismo nombre, el mandato fallará. Debe pasar el GUID de servicio en lugar de su nombre. Para localizar el GUID de un servicio, utilice `ibmcloud resource service-instance secure-file-storage-appid`.
   {: tip}
4. Despliegue la app.
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}

## Prueba de la aplicación

Se puede acceder a la aplicación en `https://secure-file-storage.<cluster-name>.<location>.containers.appdomain.cloud/`.

1. Vaya a la página de inicio de la aplicación. Se le redirigirá a la página de inicio de sesión predeterminada de {{site.data.keyword.appid_short_notm}}.
2. Regístrese para obtener una nueva cuenta con una dirección de correo electrónico válida.
3. Espere a recibir el correo electrónico en su bandeja de entrada para verificar la cuenta.
4. Inicie una sesión.
5. Elija el archivo que quiere cargar. Pulse **Cargar**.
6. Utilice la acción **Compartir** en un archivo para generar un URL prefirmado que se puede compartir con otros para acceder al archivo. El enlace se establece de modo que caduca transcurridos 5 minutos.

Los usuarios autenticados tienen sus propios espacios para almacenar archivos. Aunque no pueden ver los archivos de los demás, pueden generar URL prefirmados para otorgar acceso temporal a un archivo específico.

Encontrará más información sobre la aplicación en el [repositorio de código fuente](https://github.com/IBM-Cloud/secure-file-storage).

## Revisión de sucesos de seguridad
Ahora que la aplicación y sus servicios se han desplegado correctamente, puede revisar los sucesos de seguridad generados por ese proceso. Todos los sucesos están disponibles de forma centralizada en la instancia {{site.data.keyword.cloudaccesstrailshort}} y se puede acceder a los mismos mediante la [interfaz gráfica de usuario (Kibana), la CLI o la API](https://{DomainName}/docs/services/cloud-activity-tracker/how-to?topic=cloud-activity-tracker-viewing_event_status#viewing_event_status).

1. En la [Lista de recursos de {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), localice la instancia de {{site.data.keyword.cloudaccesstrailshort}} **secure-file-storage-activity-tracker** y abra su panel de control.
2. De forma predeterminada, el separador **Gestionar** muestra **Registros de espacio**. Vaya a **Registros de cuenta** pulsando en el selector situado junto a **Ver registros**. Debería mostrar varios sucesos.
3. Pulse **Ver en Kibana** para abrir el visor de sucesos completo.
4. Revise los detalles del suceso pulsando **Descubrir**.
5. En los **Campos disponibles**, añada **action_str** e **initiator.name_str**.
6. Expanda las entradas interesantes pulsando el icono de triángulo y, a continuación, eligiendo un formato de tabla o JSON.

Puede cambiar los valores para la renovación automática y el rango de tiempo visualizado y, por lo tanto, cambiar cómo y qué datos se analizan.
{: tip}

## Opcional: utilización de un dominio personalizado y cifrado del tráfico de red
De forma predeterminada, se puede acceder a la aplicación en un nombre de host genérico en un subdominio de `containers.appdomain.cloud`. Sin embargo, también se puede utilizar un dominio personalizado con la app desplegada. Para el soporte continuado de **https**, para el acceso con tráfico de red cifrado, se debe proporcionar un certificado para el nombre de host deseado o un certificado comodín. En la sección siguiente, cargará un certificado existente en {{site.data.keyword.cloudcerts_short}} y desplegará el certificado en el clúster. También actualizará la configuración de la app para que utilice el dominio personalizado.

Encontrará un ejemplo de cómo obtener un certificado de [Cifremos](https://letsencrypt.org/) en el siguiente [blog de {{site.data.keyword.cloud}}](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/).
{: tip}

1. Cree una instancia de [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/certificate-manager)
   * Defina como nombre **secure-file-storage-certmgr**.
   * Utilice la misma **ubicación** y el mismo **grupo de recursos** que para los otros servicios.
2. Pulse **Importar certificado** para importar el certificado.
   * Establezca el nombre en **SecFileStorage** y la descripción en **Certificate for e2e security tutorial**.
   * Cargue el archivo de certificado con el botón **Examinar**.
   * Pulse **Importar** para completar el proceso de importación.
3. Localice la entrada correspondiente al certificado importado y expándala
   * Verifique la entrada del certificado, es decir, que el nombre del dominio coincide con el dominio personalizado. Si ha cargado un certificado comodín, el nombre de dominio incluye un asterisco.
   * Pulse el símbolo **copiar** situado junto al **crn** del certificado.
4. Vaya a la línea de mandatos para desplegar la información del certificado como un secreto en el clúster. Ejecute el mandato siguiente después de copiar el crn del paso anterior.
   ```sh
   ibmcloud ks alb-cert-deploy --secret-name secure-file-storage-certificate --cluster secure-file-storage-cluster --cert-crn <the copied crn from previous step>
   ```
   {: codeblock}
   Verifique que el clúster conoce el certificado ejecutando el mandato siguiente.
   ```sh
   ibmcloud ks alb-certs --cluster secure-file-storage-cluster
   ```
   {: codeblock}
5. Edite el archivo `secure-file-storage.yaml`.
   * Localice la sección **Ingress**.
   * Elimine el comentario y edite las líneas que cubren dominios personalizados y rellene el dominio y el nombre de host.
   La entrada CNAME para el dominio personalizado tiene que apuntar al clúster. Consulte esta [guía sobre la correlación de dominios personalizados](https://{DomainName}/docs/containers?topic=containers-ingress#private_3) en la documentación para obtener detalles.
   {: tip}
6. Aplique los cambios de configuración al despliegue:
   ```sh
   kubectl apply -f secure-file-storage.yaml
   ```
   {: codeblock}
7. Vuelva al navegador. En la [lista de recursos de {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), localice el servicio {{site.data.keyword.appid_short}} que ha creado y configurado anteriormente e inicie su panel de control de gestión.
   * Vaya a **Gestionar** en **Proveedores de identidad** y luego a **Valores**.
   * En el formulario **Añadir URL de redirección web**, añada `https://secure-file-storage.<your custom domain>/appid_callback` como otro URL.
8. Ahora todo debería estar correctamente configurado. Para probar la app, acceda a la misa en el dominio personalizado configurado `https://secure-file-storage.<your custom domain>`.

## Ampliación de la guía de aprendizaje

La seguridad se debe gestionar constantemente. Pruebe las sugerencias siguientes para mejorar la seguridad de la aplicación.

* Utilice [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights) para realizar exploraciones de código estáticas y dinámicas
* Asegúrese de publicar solo código de calidad mediante políticas y reglas con [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights)

## Eliminación de recursos
{:removeresources}

Para eliminar el recurso, suprima el contenedor desplegado y luego los servicios suministrados.

1. Suprima el contenedor desplegado:
   ```sh
   kubectl delete -f secure-file-storage.yaml
   ```
   {: codeblock}
2. Suprima los secretos del despliegue:
   ```sh
   kubectl delete secret secure-file-storage-credentials
   ```
   {: codeblock}
3. Elimine la imagen de Docker del registro del contenedor:
   ```sh
   ibmcloud cr image-rm <location>.icr.io/secure-file-storage-namespace/secure-file-storage:latest
   ```
   {: codeblock}
4. En la [lista de recursos de {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/resources), localice los recursos que se han creado para esta guía de aprendizaje. Utilice el recuadro de búsqueda y **secure-file-storage** como patrón. Suprima cada uno de los servicios pulsando en el menú de contexto situado junto a cada servicio y eligiendo **Suprimir servicio**. Tenga en cuenta que el servicio {{site.data.keyword.keymanagementserviceshort}} solo se puede eliminar después de que se haya suprimido la clave. Pulse la instancia de servicio para ir al panel de control relacionado y para suprimir la clave.

Si comparte una cuenta con otros usuarios, asegúrese siempre de suprimir solo sus propios recursos.
{: tip}

## Contenido relacionado
{:related}

* [Documentación de {{site.data.keyword.security-advisor_short}}](https://{DomainName}/docs/services/security-advisor?topic=security-advisor-about#about)
* [Seguridad para proteger y supervisar las apps de la nube](https://www.ibm.com/cloud/garage/architectures/securityArchitecture)
* [Seguridad de la plataforma {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/overview?topic=overview-security#security)
* [Seguridad en IBM Cloud](https://www.ibm.com/cloud/security)
* [Guía de aprendizaje: Prácticas recomendadas para organizar usuarios, equipos y aplicaciones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)
* [Protección de apps en IBM Cloud con certificados comodín](https://www.ibm.com/blogs/bluemix/2018/07/secure-apps-on-ibm-cloud-with-wildcard-certificates/)
