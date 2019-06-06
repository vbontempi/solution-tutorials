---
copyright:
  years: 2019
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

# Apps de empresa aisladas de Cloud Foundry
{: #isolated-cloud-foundry-enterprise-apps}

Con {{site.data.keyword.cfee_full_notm}} (CFEE) puede crear varias plataformas de Cloud Foundry aisladas a nivel de empresa bajo demanda. Esto le permite obtener una instancia de Cloud Foundry desplegada en un clúster de Kubernetes aislado. A diferencia de la nube pública, el usuario tiene un control completo sobre el entorno: control de acceso, capacidad, versión, uso de recursos y supervisión. {{site.data.keyword.cfee_full_notm}} proporciona la velocidad y la innovación de una plataforma como un servicio con la propiedad de la infraestructura de la TI de la empresa.

Un caso de uso de {{site.data.keyword.cfee_full_notm}} es una plataforma de innovación propiedad de la empresa. Como desarrollador dentro de una empresa, puede crear nuevos microservicios o migrar aplicaciones antiguas a CFEE. Luego puede publicar los microservicios para que los utilicen otros desarrolladores mediante el mercado de Cloud Foundry. Una vez allí, otros desarrolladores de su instancia de CFEE pueden consumir servicios dentro de la aplicación del mismo modo que se hace actualmente en la nube pública.

La guía de aprendizaje le guiará por el proceso de creación y configuración de {{site.data.keyword.cfee_full_notm}}, de configuración del control de accesos y de despliegue de apps y servicios. También revisará la relación entre CFEE y [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) desplegando un intermediario de servicios personalizado que integre los servicios personalizados con CFEE.

## Objetivos

{: #objectives}

- Comparar y contrastar CFEE con Cloud Foundry público
- Desplegar apps y servicios dentro de CFEE
- Comprender la relación entre Cloud Foundry y {{site.data.keyword.containershort_notm}}
- Investigar la red básica de Cloud Foundry y {{site.data.keyword.containershort_notm}}

## Servicios utilizados

{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:

- [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
- [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura

{: #architecture}

![Arquitectura](./images/solution45-CFEE-apps/Architecture.png)

1. El administrador crea una instancia de CFEE y añade usuarios con acceso de desarrollador.
2. El desarrollador envía una app de inicio Node.js a CFEE.
3. La app de inicio Node.js utiliza [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant) para almacenar los datos.
4. El desarrollador añade un nuevo servicio de "mensaje de bienvenida".
5. La app de inicio Node.js enlaza el nuevo servicio desde un intermediario de servicio personalizado.
6. La app de inicio Node.js muestra el mensaje de "bienvenida" en diferentes idiomas desde el servicio.

## Requisitos previos

{: #prereq}

- [CLI de {{site.data.keyword.cloud_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
- [CLI de Cloud Foundry](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org/en/)

## Suministro de {{site.data.keyword.cfee_full_notm}}

{:provision_cfee}

En esta sección, creará una instancia de {{site.data.keyword.cfee_full_notm}} desplegada en nodos trabajadores de Kubernetes desde {{site.data.keyword.containershort_notm}}.

1. [Prepare su cuenta de {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-prepare#prepare) para garantizar la creación de los recursos de la infraestructura necesarios.
2. En el catálogo de {{site.data.keyword.cloud_notm}}, cree una instancia de servicio de [{{site.data.keyword.cfee_full_notm}} ](https://{DomainName}/cfadmin/create).
3. Configure CFEE suministrando la siguiente información:
   - Seleccione el plan **Estándar**.
   - Escriba un **Nombre** para la instancia de servicio.
   - Seleccione un **Grupo de recursos** en el que se va a crear el entorno. Necesitará permiso para acceder al menos a un grupo de recursos en la cuenta para poder crear una instancia de CFEE.
   - Seleccione la **Geografía** y la **Ubicación** en las que se va a desplegar la instancia. Consulte la lista de [ubicaciones y centros de datos de suministro disponibles](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#provisioning-targets).
   - Seleccione una **Organización** y un **Espacio** de Cloud Foundry públicos donde se desplegará **{{site.data.keyword.composeForPostgreSQL}}**.
   - Seleccione el **Número de células** para el entorno de Cloud Foundry. Una célula ejecuta aplicaciones de Diego y Cloud Foundry. Se necesitan como mínimo **2** células para las aplicaciones de alta disponibilidad.
   - Seleccione el **Tipo de máquina**, que determina el tamaño de las células de Cloud Foundry (CPU y memoria).
4. Revise la sección **Infraestructura** para ver las propiedades del clúster de Kubernetes que da soporte a CFEE. El **Número de nodos trabajadores** es igual al número de células más 2. Dos de los nodos trabajadores de Kubernetes suministrados actúan como plano de control CFEE. El clúster de Kubernetes en el que se despliega el entorno aparecerá en el panel de control [Clústeres](https://{DomainName}/containers-kubernetes/clusters) de {{site.data.keyword.cloud_notm}}.
5. Pulse el botón **Crear** para comenzar el despliegue automatizado.

El despliegue automatizado tarda entre 90 y 120 minutos aproximadamente. Una vez que se ha creado correctamente, recibirá un correo electrónico con la confirmación del suministro de CFEE y de los servicios de soporte.

### Creación de organizaciones y de espacios

Después de crear {{site.data.keyword.cfee_full_notm}}, consulte [creación de organizaciones y espacios](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-create_orgs#create_orgs) para obtener información sobre cómo estructurar el entorno mediante organizaciones y espacios. Las apps de {{site.data.keyword.cfee_full_notm}} abarcan espacios específicos. Paralelamente, un espacio reside dentro de una organización. Los miembros de una organización comparten un plan de cuotas, apps, instancias de servicio y dominios personalizados.

Nota: el menú **Gestionar > Cuenta > Orgs de Cloud Foundry** ubicado en la cabecera {{site.data.keyword.cloud_notm}} principal está destinado exclusivamente a las organizaciones de {{site.data.keyword.cloud_notm}} público. Las organizaciones de CFEE se gestionan en la página **organizaciones** de una instancia de CFEE.

Siga los pasos que se indican a continuación para crear una organización y un espacio de CFEE.

1. En el [panel de control de Cloud Foundry](https://{DomainName}/dashboard/cloudfoundry/overview), seleccione **Entornos** bajo **Empresa**.
2. Seleccione su instancia de CFEE y luego seleccione **Organizaciones**.
3. Pulse el botón **Crear organizaciones**, especifique una `guía de aprendizaje` como **Nombre de organización** y seleccione un **Plan de presupuesto**. Para terminar, pulse **Añadir**.
4. Pulse la `guía de aprendizaje` de la organización recién creada, seleccione el separador **Espacios** y pulse el botón **Crear espacio**.
5. Especifique `dev` como **Nombre de espacio** y pulse **Añadir**.

### Adición de usuarios a orgs y espacios

En CFEE, puede asignar roles que controlan el acceso de usuario, pero, para ello, el usuario debe ser invitado a su cuenta de {{site.data.keyword.cloud_notm}} en la página **Identidad y acceso** bajo **Gestionar > Usuarios** en la cabecera de {{site.data.keyword.cloud_notm}}.

Una vez que se ha invitado al usuario, siga los pasos que se indican a continuación para añadir el usuario a la organización de la `guía de aprendizaje` creada, 

1. Seleccione su [instancia de CFEE](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) y luego seleccione de nuevo **Organizaciones**.
2. Seleccione la org de la `guía de aprendizaje` creada en la lista.
3. Pulse el separador **Miembros** para ver y añadir un nuevo usuario.
4. Pulse el botón **Añadir miembros**, busque el nombre de usuario, seleccione los **Roles de organización** adecuados y pulse **Añadir**.

Puede encontrar más información sobre cómo añadir usuarios a orgs y espacios de CFEE [aquí](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-adding_users#adding_users).

## Despliegue, configuración y ejecución de apps de CFEE

{:deploycfeeapps}

En esta sección, desplegará una aplicación Node.js en CFEE. Una vez desplegada, enlazará un {{site.data.keyword.cloudant_short_notm}} a la misma y habilitará la auditoría y la persistencia de registro. También se añadirá la consola Stratos para gestionar la aplicación.

### Despliegue de la aplicación en CFEE

1. En el terminal, clone la aplicación de ejemplo [get-started-node](https://github.com/IBM-Cloud/get-started-node).
   ```sh
   git clone https://github.com/IBM-Cloud/get-started-node
   ```
   {: codeblock}
2. Ejecute la app localmente para asegurarse de que se compila y se inicia correctamente. Confirme accediendo a `http://localhost:3000/` en el navegador.
   ```sh
   cd get-started-node
   npm install
   npm start
   ```
   {: codeblock}
3. Inicie una sesión {{site.data.keyword.cloud_notm}} y seleccione su instancia de CFEE como destino. Una solicitud interactiva le ayudará a seleccionar la nueva instancia de CFEE. Puesto que solo existe una organización y un espacio CFEE, estos serán el destino predeterminado. Puede ejecutar `ibmcloud target -o tutorial -s dev` si ha añadido más de una organización o espacio.
   ```sh
   ibmcloud login
   ibmcloud target --cf
   ```
   {: codeblock}
4. Envíe la app **get-started-node** a CFEE.
   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
5. El punto final de la app se mostrará en la salida final junto a la propiedad `routes`. Abra el URL en el navegador para confirmar que la aplicación se está ejecutando.

### Creación y enlace de una base de datos Cloudant a la app

Para enlazar servicios de {{site.data.keyword.cloud_notm}} a la aplicación **get-started-node**, primero tendrá que crear un servicio en la cuenta de {{site.data.keyword.cloud_notm}}.

1. Cree un servicio de [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant). Especifique el **nombre de servicio** `cfee-cloudant` y elija la misma ubicación en la que se ha creado la instancia de CFEE.
2. Añada la instancia de servicio de {{site.data.keyword.cloudant_short_notm}} que acaba de crear a CFEE.
   1. Vuelva a la `guía de aprendizaje` **Organización**. Pulse el separador **Espacios** y seleccione el espacio `dev`.
   2. Seleccione el separador **Servicios** y pulse el botón **Añadir servicio**.
   3. Escriba `cfee-cloudant` en el recuadro de texto de búsqueda y seleccione el resultado. Para terminar, pulse **Añadir**. Ahora el servicio está disponible para las aplicaciones CFEE; sin embargo, sigue residiendo en {{site.data.keyword.cloud_notm}} público.
3. En el menú de desbordamiento de la instancia de servicio que se muestra, seleccione **Enlazar a aplicación**.
4. Seleccione la aplicación **GetStartedNode** que ha enviado anteriormente y pulse **Volver a transferir aplicación después del enlace**. Por último, pulse el botón **Enlazar**. Espere a que la aplicación se vuelva a transferir. Puede comprobar el progreso con el mandato `ibmcloud app show GetStartedNode`.
5. En el navegador, acceda a la aplicación, añada su nombre y pulse `intro`. Su nombre se agregará a una base de datos de {{site.data.keyword.cloudant_short_notm}}.
6. Para confirmar, seleccione la instancia de `guía de aprendizaje` de la lista del separador **Servicios**. Esto abrirá la página de detalles de la instancia del servicio en {{site.data.keyword.cloud_notm}} público.
7. Pulse **Iniciar panel de control de Cloudant** y seleccione la base de datos `mydb`. Debería existir un documento JSON con su nombre.

### Habilitación de persistencia de auditoría y de registro

La auditoría permite a los administradores de CFEE realizar un seguimiento de las actividades de Cloud Foundry, como el inicio de sesión, la creación de organizaciones y espacios, la pertenencia de usuarios y asignaciones de roles, los despliegues de aplicaciones, los enlaces de servicio y la configuración de dominios. Se da soporte a la auditoría mediante la integración con el servicio {{site.data.keyword.cloudaccesstrailshort}}.

Los registros de aplicaciones de Cloud Foundry se pueden almacenar mediante la integración de {{site.data.keyword.loganalysisshort_notm}}. La instancia de servicio de {{site.data.keyword.loganalysisshort_notm}} seleccionada por un administrador de CFEE se configura automáticamente de modo que reciba y conserve los sucesos de registro de Cloud Foundry generados a partir de la instancia CFEE.

Para habilitar la persistencia de auditoría y de registro de CFEE, siga [estos pasos](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-auditing-logging#auditing-logging).

### Instalación de Stratos Console para gestionar la app

Stratos Console es una herramienta basada en web de código abierto para trabajar con Cloud Foundry. La aplicación Stratos Console se puede instalar de forma opcional y utilizarse en un entorno de CFEE específico para gestionar sus organizaciones, espacios y aplicaciones.

Para instalar la aplicación Stratos Console:

1. Abra la instancia de CFEE en la que desea instalar Stratos Console.
2. En la página **Visión general**, pulse **Instalar Stratos Console**. El botón está visible únicamente para los usuarios con permisos de administrador o editor.
3. En el diálogo Instalar Stratos Console, seleccione una opción de instalación. Puede instalar la aplicación Stratos Console en el plano de control de CFEE o en una de las células. Seleccione una versión de Stratos Console y el número de instancias de la aplicación a instalar. Si instala la app Stratos Console en una célula, se le solicitará que proporcione una organización y espacio en los que desplegar la aplicación.
4. Pulse **Instalar**.

La instalación de la aplicación tarda 5 minutos aproximadamente. Una vez completada la instalación, aparecerá el botón de **Stratos Console** en lugar del botón **Instalar Stratos Console** en la página de visión general. Encontrará más información sobre Stratos Console [aquí](https://{DomainName}/docs/tutorials?topic=solution-tutorials-isolated-cloud-foundry-enterprise-apps#install-the-stratos-console-to-manage-the-app).

## La relación entre CFEE y Kubernetes

Como plataforma de aplicaciones, CFEE se ejecuta en una infraestructura virtual compartida o dedicada. Durante muchos años, los desarrolladores no han tenido muy en cuenta la plataforma Cloud Foundry subyacente porque IBM lo gestionaba para ellos. Con CFEE, usted no es solo es un desarrollador que escribe aplicaciones Cloud Foundry, sino que también es un operador de la plataforma Cloud Foundry. Esto se debe a que CFEE se despliega en un clúster de Kubernetes que controla usted.

Aunque algunos desarrolladores de Cloud Foundry no han utilizado Kubernetes, existen varios conceptos que ambos comparten. Al igual que Cloud Foundry, Kubernetes aísla las aplicaciones en contenedores, que se ejecutan dentro de una construcción de Kubernetes que se denomina pod. De forma similar a lo que sucede con las instancias de aplicaciones, los pods pueden tener varias copias (denominadas conjuntos de réplicas) con equilibrio de carga de aplicaciones proporcionado por Kubernetes.  La aplicación `GetStartedNode` de Cloud Foundry que ha desplegado antes se ejecuta dentro del pod `diego-cell-0`. Para dar soporte a la alta disponibilidad, se ejecutaría otro pod `diego-cell-1` en otro nodo trabajador de Kubernetes. Debido a que estas apps Cloud Foundry se ejecutan "dentro" de Kubernetes, también puede comunicarse con otros microservicios de Kubernetes mediante redes basadas en Kubernetes. En las secciones siguientes se ilustran las relaciones entre CFEE y Kubernetes con más detalle.

## Despliegue de un intermediario de servicio de Kubernetes

En esta sección, desplegará un microservicio en Kubernetes que actúe como un intermediario de servicio para CFEE. Los [intermediarios de servicio](https://github.com/openservicebrokerapi/servicebroker/blob/v2.13/spec.md) proporcionan detalles sobre los servicios disponibles, así como enlaces y soporte de suministro para la aplicación Cloud Foundry. Ha utilizado un intermediario de servicios de {{site.data.keyword.cloud_notm}} incorporado para añadir el servicio {{site.data.keyword.cloudant_short_notm}}. Ahora desplegará y utilizará uno personalizado. Luego modificará la app `GetStartedNode` para que utilice el intermediario de servicio, que devolverá un mensaje de "Bienvenida" en varios idiomas.

1. De vuelta en el terminal, clone los proyectos que proporcionan los archivos de despliegue de Kubernetes y la implementación del intermediario de servicios.

   ```sh
   git clone https://github.com/IBM-Cloud/cfee-service-broker-kubernetes.git
   ```
   {: codeblock}

   ```sh
   cd cfee-service-broker-kubernetes
   ```
   {: codeblock}

   ```sh
   git clone https://github.com/IBM/sample-resource-service-brokers.git
   ```
   {: codeblock}

2. Cree y almacene la imagen de Docker que contiene el intermediario de servicio en {{site.data.keyword.registryshort_notm}}. Utilice el mandato `ibmcloud cr info` para recuperar manualmente el URL de registro o hágalo automáticamente con el mandato `export REGISTRY` siguiente. El mandato `cr namespace-add` creará un espacio de nombres para almacenar la imagen de docker.

   ```sh
   export REGISTRY=$(ibmcloud cr info | head -2 | awk '{ print $3 }')
   ```
   {: codeblock}

   Cree un espacio de nombres denominado `cfee-tutorial`.

   ```sh
   ibmcloud cr namespace-add cfee-tutorial
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   Edite el archivo `./cfee-service-broker-kubernetes/deployment.yml`. Actualice el atributo `image` para que refleje el URL del registro del contenedor.
3. Despliegue la imagen de contenedor en el clúster de Kubernetes de CFEE. El clúster de CFEE se encuentra en el grupo de recursos `default`, que se debe especificar como destino si aún no se ha hecho. Utilizando el nombre de su clúster, exporte la variable KUBECONFIG con el mandato `cluster-config`. Luego cree el despliegue.
   ```sh
   ibmcloud target -g default
   ```
   {: codeblock}

   ```sh
   ibmcloud ks clusters
   ```
   {: codeblock}

   ```sh
   $(ibmcloud ks cluster-config <your-cfee-cluster-name> --export)
   ```
   {: codeblock}

   ```sh
   kubectl apply -f deployment.yml
   ```
   {: codeblock}
4. Verifique que el estado de los pods sea `Running`. Kubernetes puede tardar un rato en extraer la imagen e iniciar los contenedores. Tenga en cuenta que tiene dos pods porque `deployment.yml` ha solicitado 2 `réplicas`.
   ```sh
   kubectl get pods
   ```
   {: codeblock}

## Verificación de que se ha desplegado el intermediario de servicio

Ahora que ha desplegado el intermediario de servicio, confirme que funciona correctamente. Lo hará de varias maneras: primero utilizando el panel de control de Kubernetes, luego accediendo al intermediario desde una app de Cloud Foundry y finalmente enlazando realmente un servicio del intermediario.

### Visualización de pods desde el panel de control de Kubernetes

En esta sección se confirmará que los artefactos de Kubernetes están configurados mediante el panel de control de {{site.data.keyword.containershort_notm}}.

1. En la página [Clústeres de Kubernetes](https://{DomainName}/containers-kubernetes/clusters), acceda al clúster de CFEE pulsando el elemento de fila que empieza por el nombre del servicio CFEE y termina por **-cluster**.
2. Abra al **Panel de control de Kubernetes** pulsando el botón correspondiente.
3. Pulse el enlace **Servicios** en el menú de la izquierda y seleccione **tutorial-broker-service**. Este servicio se ha desplegado cuando se ha ejecutado `kubectl apply` en pasos anteriores.
4. En el panel de control resultante, observe lo siguiente:
   - Se ha proporcionado al servicio una dirección IP (172.x.x.x) que solo se puede resolver dentro del clúster de Kubernetes.
   - El servicio tiene dos puntos finales, que corresponden a las dos pods que tienen en ejecución los contenedores del intermediario de servicio.

Una vez confirmado que el servicio está disponible y que actúa como proxy de los pods del intermediario de servicio, puede verificar que el intermediario responde con información sobre los servicios disponibles.

Puede ver los artefactos relacionados con Cloud Foundry desde el panel de control de Kubernetes. Para verlos, pulse **Espacios de nombres** para ver todos los espacios de nombres con artefactos, incluido el espacio de nombres de Cloud Foundry `cf`.
{: tip}

### Acceso al intermediario desde un contenedor de Cloud Foundry

Para demostrar la comunicación entre Cloud Foundry y Kubernetes, se conectará con el intermediario de servicio directamente desde una aplicación de Cloud Foundry.

1. De nuevo en el terminal, confirme que sigue conectado a la organización `guía de aprendizaje` de CFEE y al espacio `dev` con el mandato `ibmcloud target`. Si es necesario, vuelva a especificar CFEE como destino.
   ```sh
   ibmcloud target --cf
   ```
   {: codeblock}
2. De forma predeterminada, SSH está inhabilitado en los espacios. Esto difiere de lo que sucede en la nube pública, por lo que debe habilitar SSH en el espacio `dev` de CFEE.
   ```sh
   ibmcloud cf allow-space-ssh dev
   ```
   {: codeblock}
3. Utilice el mandato `kubectl` para mostrar el mismo ClusterIP que ha visto en el panel de control de Kubernetes. Luego ejecute SSH sobre la aplicación `GetStartedNode` y recupere datos del intermediario de servicio utilizando la dirección IP. Tenga en cuenta que el último mandato puede generar un error, que el siguiente paso resolverá.
   ```sh
   kubectl get service tutorial-broker-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf ssh GetStartedNode
   ```
   {: codeblock}

   ```sh
   export CLUSTER_IP=<CLUSTER-IP address>
   ```
   {: codeblock}

   ```sh
   wget --user TestServiceBrokerUser --password TestServiceBrokerPassword -O- http://$CLUSTER_IP/v2/catalog
   ```
   {: codeblock}
4. Es probable que haya recibido un error de **Conexión rechazada**. Esto se debe a los [grupos de seguridad de la aplicación](https://docs.cloudfoundry.org/concepts/asg.html) predeterminados de CFEE. Un grupo de seguridad de aplicación (ASG) define el rango de IP permitido para el tráfico de salida procedente de un contenedor de Cloud Foundry. Puesto que `GetStartedNode` está fuera del rango predeterminado, se produce el error. Salga de la sesión SSH y descargue el ASG `public_networks`.
   ```sh
   exit
   ```
   {: codeblock}

   ```sh
   ibmcloud cf security-group public_networks > public_networks.json
   ```
   {: codeblock}
5. Edite el archivo `public_networks.json` y verifique que la dirección de ClusterIP que se utiliza está fuera de las reglas existentes. Por ejemplo, el rango `172.32.0.0-192.167.255.255` probablemente no incluya la ClusterIP y se deba actualizar. Si, por ejemplo, la dirección de ClusterIP se parece a `172.21.107.63`, debería editar el archivo para que contenga algo parecido a `172.0.0.0-255.255.255.255`.
6. Ajuste la regla de `destino` de ASG para que incluya la dirección IP del servicio Kubernetes. Recorte el archivo para que solo incluya los datos JSON, que empiezan y terminan por corchetes. Luego cargue el nuevo ASG.
   ```sh
   ibmcloud cf update-security-group public_networks public_networks.json
   ```
   {: codeblock}

   ```sh
   ibmcloud cf restart GetStartedNode
   ```
   {: codeblock}
7. Repita el paso 3, que ahora debe ejecutarse correctamente con los datos ficticios del catálogo. Para terminar, salga de la sesión SSH.
   ```sh
   exit
   ```
   {: codeblock}

### Registro del intermediario de servicio con CFEE

Para permitir que los desarrolladores puedan suministrar y enlazar servicios desde el intermediario de servicio, lo registrará con CFEE. Antes ha trabajado con el intermediario utilizando una dirección IP. Sin embargo, esto resulta problemático. Si se reinicia el intermediario de servicio, recibe una nueva dirección IP, que requiere que se actualice CFEE. Para solucionar este problema, utilizará otra característica de Kubernetes llamada KubeDNS que proporciona un nombre de dominio completo (FQDN) al intermediario de servicio.

1. Registre el intermediario de servicio con CFEE utilizando el FQDN del servicio `tutorial-service-broker`. De nuevo, esta ruta es interna del clúster de Kubernetes de CFEE.
   ```sh
   ibmcloud cf create-service-broker my-company-broker TestServiceBrokerUser TestServiceBrokerPassword http://tutorial-broker-service.default.svc.cluster.local
   ```
   {: codeblock}
2. Luego añada los servicios que ofrece el intermediario. Puesto que el intermediario de ejemplo sólo tiene un servicio ficticio, solo se necesita un mandato.
   ```sh
   ibmcloud cf enable-service-access testnoderesourceservicebrokername
   ```
   {: codeblock}
3. En el navegador, acceda a la instancia de CFEE desde la página [**Entornos**](https://{DomainName}/dashboard/cloudfoundry?filter=cf_environments) y vaya a `organizaciones -> espacios` y seleccione su espacio `dev`.
4. Seleccione el separador **Servicios** y el botón **Crear servicio**.
5. En el recuadro de texto de búsqueda, busque **Test**. Se mostrará el servicio ficticio **Test Node Resource Service Broker Display Name** del intermediario.
6. Seleccione el servicio, pulse el botón **Crear** y especifique el nombre `welcome-service` para crear una instancia de servicio. En el texto quedará claro por qué se llama welcome-service. A continuación, enlace el servicio a la app `GetStartedNode` utilizando el elemento **Enlazar con aplicación** del menú de desbordamiento.
7. Para ver el servicio enlazado, ejecute el mandato `cf env`. Ahora la aplicación `GetStartedNode` puede aprovechar los datos en el objeto `credentials` de forma similar a cómo utiliza actualmente los datos de `cloudantNoSQLDB`.
   ```sh
   ibmcloud cf env GetStartedNode
   ```
   {: codeblock}

## Adición del servicio a la aplicación

Hasta ahora ha desplegado un intermediario de servicio, pero no un servicio real. Aunque el intermediario de servicio proporciona datos de enlace a `GetStartedNode`, no se ha añadido ninguna funcionalidad real del propio servicio. Para corregirlo, se ha creado un servicio ficticio que proporciona un mensaje de "bienvenida" a `GetStartedNode` en varios idiomas. Desplegará este servicio en CFEE y actualizará el intermediario y la aplicación para que lo utilicen.

1. Envíe la implementación `welcome-service` a CFEE para permitir que los equipos de desarrollo lo aprovechen como una API.
   ```sh
   cd sample-resource-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}
2. Acceda al servicio mediante el navegador. La `ruta` al servicio se mostrará como parte de la salida del mandato `push`. El URL resultante será similar a `https://welcome.<your-cfee-cluster-domain>`. Renueve la página para ver otros idiomas.
3. Vuelva a la carpeta `sample-resource-service-brokers` y edite la implementación de ejemplo de Node.js `sample-resource-service-brokers/node-resource-service-broker/testresourceservicebroker.js` para añadir el URL del clúster tras la línea 854. 

   ```javascript
   // TODO - Do your actual work here
    
   var generatedUserid   = uuid();
   var generatedPassword = uuid();
    
   result = {
     credentials: {
       userid   : generatedUserid,
       password : generatedPassword,
       url: 'https://welcome.<your-cfee-cluster-domain>'
     }
   };
   ```
   {: codeblock}
4. Cree y despliegue el intermediario de servicio actualizado. Esto garantizará que la propiedad URL se proporcionará a las apps que enlacen con el servicio.
   ```sh
   cd ..
   ```
   {: codeblock}

   ```sh
   ibmcloud cr build . -t $REGISTRY/cfee-tutorial/service-broker-impl
   ```
   {: codeblock}

   ```sh
   kubectl patch deployment tutorial-servicebroker-deployment -p "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"version\":\"2\"}}}}}"
   ```
   {: codeblock}
5. En el terminal, vuelva a la carpeta de aplicación `get-started-node`. 
   ```sh
   cd ..
   ```
   {: codeblock}
   ```sh
   cd get-started-node
   ```
   {: codeblock}
6. Edite el archivo `get-started-node/server.js` en `get-started-node` para incluir el siguiente middleware. Insértelo antes de la llamada `app.listen()`.
   ```javascript
   // Utilizar el servicio welcome
   const request = require('request');
   const testService = appEnv.services['testnoderesourceservicebrokername'];

   if (testService) {
     const { credentials: { url} } = testService[0];
     app.get("/api/welcome", (req, res) => request(url, (e, r, b) => res.send(b)));
   } else {
     app.get("/api/welcome", (req, res) => res.send('Welcome'));
   }
   ```
   {: codeblock}
7. Refactorice la página `get-started-node/views/index.html`. Cambie `data-i18n="welcome"` por `id="welcome"` en la etiqueta `h1`. Luego añada una llamada al servicio al final de la función `$(document).ready()`.
   ```html
   <h1 id="welcome"></h1> <!-- Welcome -->
   ```
   {: codeblock}

   ```javascript
   $.get('./api/welcome').done(data => document.getElementById('welcome').innerHTML= data);
   ```
   {: codeblock}
8. Actualice la app `GetStartedNode`. Incluya la dependencia de paquete `request` que se ha añadido a `server.js`, vuelva a enlazar `welcome-service` para que tome la nueva propiedad `url` y finalmente envíe el nuevo código de la app.
   ```sh
   npm i request -S
   ```
   {: codeblock}

   ```sh
   ibmcloud cf unbind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf bind-service GetStartedNode welcome-service
   ```
   {: codeblock}

   ```sh
   ibmcloud cf push
   ```
   {: codeblock}

Ha terminado. Ahora visite la aplicación y renueve la página varias veces para ver el mensaje de bienvenida en diferentes idiomas.

![Respuesta del intermediario de servicio](./images/solution45-CFEE-apps/service-broker.png)

Si continúa viendo solo **Welcome** y no en otros idiomas, ejecute el mandato `ibmcloud cf env GetStartedNode` y confirme que aparece el `URL` al servicio. Si no es así, vuelva a rastrear los pasos, actualice y vuelva a desplegar el intermediario. Si el valor de `url` está presente, confirme que devuelve un mensaje en el navegador. Luego vuelva a ejecutar los mandatos `unbind-service` y `bind-service` seguidos de `ibmcloud cf restart GetStartedNode`.
{: tip}

Aunque el servicio de bienvenida utiliza Cloud Foundry como implementación, también podría usar fácilmente Kubernetes. La principal diferencia es que el URL al servicio probablemente sea `welcome-service.default.svc.cluster.local`. Utilizar el URL de KubeDNS tiene la ventaja añadido de mantener internamente el tráfico de red a los servicios del clúster de Kubernetes.

Con {{site.data.keyword.cfee_full_notm}}, estos enfoques alternativos no son posible.

## Despliegue de esta guía de aprendizaje de la solución mediante una cadena de herramientas  

Si lo desea, puede desplegar la guía de aprendizaje completa de la solución utilizando una cadena de herramientas. Siga las [instrucciones de la cadena de herramientas](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes) para desplegar todo lo anterior utilizando una cadena de herramientas. 

Nota: se aplican algunos requisitos previos cuando se utiliza una cadena de herramientas. Debe haber creado una instancia de CFEE y una org y un espacio de CFEE. Encontrará instrucciones detalladas en el archivo readme de la [cadena de herramientas](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes). 

## Contenido relacionado
{:related}

* [Despliegue de apps en clústeres de Kubernetes](https://{DomainName}/docs/containers?topic=containers-app#app)
* [Componentes y arquitectura de Cloud Foundry Diego](https://docs.cloudfoundry.org/concepts/diego/diego-architecture.html)
* [Intermediario de servicio de CFEE en Kubernetes con una cadena de herramientas](https://github.com/IBM-Cloud/cfee-service-broker-kubernetes)

