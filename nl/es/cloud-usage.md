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

# Revisión de los servicios, los recursos y el uso de {{site.data.keyword.cloud_notm}}
{: #cloud-usage}
A medida que aumenta la adopción de la nube, los gestores de TI y de finanzas necesitan comprender el uso de la nube en el contexto de innovación y control de costes. Mediante la investigación de los datos disponibles se puede responder a preguntas como "¿Qué servicios están utilizando los equipos?", "¿Cuánto cuesta poner una solución en funcionamiento?" y "¿Cómo puedo controlar la dispersión?". Esta guía de aprendizaje contiene una introducción a las formas de explorar dicha información y de responder a las preguntas relacionadas con el uso común.
{:shortdesc}

## Objetivos
{: #objectives}
* Especificación de artefactos de {{site.data.keyword.cloud_notm}}: apps y servicios de Cloud Foundry, recursos de Identity and Access Management y dispositivos de la infraestructura
* Asociación de los artefactos de {{site.data.keyword.cloud_notm}} con el uso y la facturación
* Definición de relaciones entre artefactos de {{site.data.keyword.cloud_notm}} y equipos de desarrollo
* Aprovechamiento de los datos sobre uso y facturación para crear conjuntos de datos que se puedan utilizar en contabilidad

## Arquitectura
{: #architecture}

![Arquitectura](images/solution38/Architecture.png)

## Antes de empezar
{: #prereqs}

* Instale la [CLI de {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli)
* Instale [cURL](https://curl.haxx.se/)
* Instale [Node.js](https://nodejs.org/)
* Instale [json2csv](https://www.npmjs.com/package/json2csv) mediante el mandato `npm install -g json2csv`
* Instale [jq](https://stedolan.github.io/jq/)

## Contexto
{: #background}

Antes de ejecutar mandatos para obtener datos detallados e inventario sobre el uso de {{site.data.keyword.cloud_notm}}, resulta útil conocer el contexto sobre la amplia gama de categorías de uso y su función. Los términos clave que se utilizan más adelante en esta guía de aprendizaje aparecen en negrita. Encontrará una explicación útil de los artefactos siguientes en la [documentación sobre gestión de la cuenta](https://{DomainName}/docs/account?topic=account-overview#overview).

### Cloud Foundry
Cloud Foundry es una plataforma como servicio (PaaS) de código abierto de {{site.data.keyword.cloud_notm}} que le permite desplegar y escalar aplicaciones y **Servicios** sin tener que gestionar servidores. Cloud Foundry organiza las aplicaciones y los servicios en organizaciones o espacios.  Una **Org** es una cuenta de desarrollo que uno o varios usuarios pueden poseer y utilizar. Una org puede contener varios espacios. Cada **espacio** proporciona a los usuarios acceso a una ubicación compartida para realizar funciones de desarrollo, despliegue y mantenimiento de aplicaciones.

### Identity and Access Management
IBM Cloud Identity and Access Management (IAM) le permite autenticar de forma segura usuarios tanto para servicios de la plataforma como para el control del acceso a recursos de la plataforma {{site.data.keyword.cloud_notm}}. Las ofertas más recientes y los servicios migrados de Cloud Foundry constituyen **recursos** gestionados por {{site.data.keyword.cloud_notm}} Identity and Access Management. Los recursos se organizan en **Grupos de recursos** y proporcionan control de acceso a través de políticas y roles.

### Infraestructura
La infraestructura abarca una variedad de opciones de cálculo: servidores nativos, instancias de servidor virtual y nodos de Kubernetes. Cada una de estas opciones constituye un **Dispositivo** en la consola.

### Cuenta
Los artefactos mencionados anteriormente están asociados a una **Cuenta** con fines de facturación. Se invita a los **usuarios** a la cuenta y se les otorga acceso a los distintos recursos de la cuenta.

## Asignación de permisos
Para ver el inventario y el uso de la nube, necesita que el administrador de la cuenta le asigne los roles adecuados. Si es el administrador de la cuenta, continúe en la sección siguiente.

1. El administrador de la cuenta debe iniciar una sesión en {{site.data.keyword.cloud_notm}} y acceder a la página [**Usuarios de Identity & Access**](https://{DomainName}/iam/#/users).
2. El administrador de cuentas puede seleccionar su nombre en la lista de usuarios de la cuenta para asignarle los permisos apropiados.
3. En el separador **Políticas de acceso**, pulse el botón **Asignar acceso** y realice los cambios siguientes.
   1. Seleccione el mosaico **Asignar acceso dentro de un grupo de recursos**. Seleccione el **grupo o grupos de recursos** a los que se va a asignar acceso y aplique el rol **Visor**. Para terminar pulse el botón **Asignar**. Este rol se necesita para los mandatos `resource groups` y `billing resource-group-usage`.
   2. Seleccione el mosaico **Asignar acceso mediante Cloud Foundry**. Seleccione el menú de desbordamiento situado junto a cada **Organización** a la que se va a otorgar acceso. Seleccione **Editar rol de organización** en el menú. Seleccione **Gestor de facturación** en la lista **Roles de organización**. Para terminar pulse el botón **Guardar rol**. Este rol es necesario para el mandato `billing org-usage`.
   3. Seleccione el mosaico **Asignar acceso a recursos**. Seleccione **Todos los servicios habilitados para identidad y acceso** en el menú desplegable **Servicios**. Marque el rol de **Editor** bajo **Asignar roles de acceso a la plataforma**. Este rol es necesario para el mandato `resource tag-attach`.

## Localización de recursos con el mandato de búsqueda
{: #search}

A medida que los equipos de desarrollo empiecen a utilizar los servicios de la nube, los gestores se beneficiarán de saber qué servicios se han desplegado. La información de despliegue ayuda a responder a preguntas relacionadas con la innovación y la gestión de servicios:
- ¿Qué conocimientos relacionados con el servicio se pueden compartir entre equipos para mejorar otros proyectos?
- ¿Qué tienen en común los equipos que puede faltar en alguno de ellos?
- ¿Qué equipos están utilizando un servicio que requiere un arreglo crítico o pronto quedará en desuso?
- ¿Cómo pueden los equipos revisar sus instancias de servicio para minimizar la dispersión?

La búsqueda no se limita a servicios y recursos. También puede consultar artefactos de la nube, como organizaciones y espacios de Cloud Foundry, grupos de recursos, enlaces de recursos, alias, etc. Para ver más ejemplos, consulte la documentación del mandato [ibmcloud resource search](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud_commands_resource#ibmcloud_resource_search).
{:tip}

1. Inicie una sesión en {{site.data.keyword.cloud_notm}} mediante la línea de mandatos y seleccione la cuenta de Cloud Foundry. Consulte [Iniciación a la CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#ibmcloud-cli).
    ```sh
    ibmcloud login
    ```
    {: pre}

    ```sh
    ibmcloud target --cf
    ```
    {: pre}

2. Realice un inventario de todos los servicios de Cloud Foundry que se utilizan en la cuenta.
    ```sh
    ibmcloud resource search 'type:cf-service-instance'
    ```
    {: pre}

3. Se pueden aplicar filtros booleanos para ampliar o limitar las búsquedas. Por ejemplo, puede buscar servicios y apps de Cloud Foundry, así como recursos de IAM, mediante la consulta siguiente.
    ```sh
    ibmcloud resource search 'type:cf-service-instance OR type:cf-application OR type:resource-instance'
    ```
    {: pre}

4. Para notificar a los equipos que utilizan un determinado tipo de servicio, realice la consulta utilizando el nombre de servicio. Sustituya `<name>` por texto, como por ejemplo `weather` o `cloudant`. Para obtener el nombre de la organización, sustituya `<Organization ID>` por el **ID de la organización**.
    ```sh
   ibmcloud resource search 'service_name: *<name>*'
    ```
    {: pre}

    ```sh
    ibmcloud cf curl /v2/organizations/<Organization ID> | tail -n +3 | jq .entity.name
    ```
    {: pre}

El etiquetado y la búsqueda se pueden utilizar conjuntamente para proporcionar una identificación personalizada de los recursos. Esto implica: adjuntar la etiqueta a los recursos y realizar la búsqueda utilizando los nombres de etiqueta. Cree una etiqueta llamada env:tutorial.

1. Adjunte la etiqueta a un recurso. Puede obtener un CRN de recursos de la interfaz de usuario o bien con el mandato `ibmcloud resource service-instance <name|id>`. Los nombres de recursos de nube (CRN) identifican de forma exclusiva los recursos de IBM Cloud.
   ```sh
   ibmcloud resource tag-attach --tag-names env:tutorial --resource-id <resource CRN>
   ```
   {: pre}
1. Busque los artefactos de la nube que coinciden con una determinada etiqueta mediante la consulta siguiente.
   ```
   ibmcloud resource search 'tags: "env:tutorial"'
   ```
   {: pre}

Mediante la combinación de consultas de búsqueda avanzada con un esquema de etiquetado adaptado a la empresa, los gestores y los jefes de equipo pueden identificar y actuar más fácilmente sobre apps, recursos y servicios de la nube.

## Exploración del uso con el panel de control de uso
{: #dashboard}

Una vez que el equipo de gestión conoce los servicios que utilizan los equipos, la siguiente pregunta más frecuente suele ser "¿Cuánto cuesta poner en funcionamiento estos servicios?" El medio más rápido de determinar el uso y el coste consiste en revisar el panel de control de uso de {{site.data.keyword.cloud_notm}}.

1. Inicie una sesión en {{site.data.keyword.cloud_notm}} y acceda al [Panel de control de uso de la cuenta](https://{DomainName}/account/usage).
2. En el menú desplegable **Grupos**, seleccione una organización de Cloud Foundry para ver el uso del servicio.
3. Para una determinada **Oferta de servicio**, pulse **Ver instancias** para ver las instancias de servicio que se han creado.
4. En la página siguiente, elija una instancia y pulse **Ver instancia**. La página resultante proporciona detalles sobre la instancia, como por ejemplo Organización, Espacio y Ubicación, así como los elementos de línea individuales que componen el coste total.
5. Utilizando el rastro de navegación, vuelva al [panel de control de uso](https://{DomainName}/account/usage).
6. En el menú desplegable **Grupos**, cambie el selector por **Grupos de recursos** y seleccione un grupo, como por ejemplo **default**.
7. Realice una revisión similar de las instancias disponibles.

## Exploración del uso con la línea de mandatos

En esta sección, explorará el uso con la interfaz de línea de mandatos.

1. Obtenga una lista de todas las organizaciones de Cloud Foundry disponibles y defina una variable de entorno para almacenar una para realizar pruebas.
    ```sh
    ibmcloud cf orgs
    ```
    {: pre}

    ```sh
    export ORG_NAME=<org name>
    ```
    {: pre}

2. Especifique la facturación y el uso correspondientes a una determinada organización con el mandato `facturación`. Especifique un mes determinado con el distintivo `-d` con una fecha en el formato AAAA-MM.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Realice la misma investigación para recursos. Cada cuenta tiene un grupo de recursos `default`. Sustituya el valor `<group>` por un grupo de recursos mostrado en el primer mandato.
    ```sh
    ibmcloud resource groups
    ```
    {: pre}

    ```sh
    export RESOURCE_GROUP=<group>
    ```
    {: pre}

    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP
    ```
    {: pre}

4. Puede ver tanto los servicios de Cloud Foundry como los recursos de IAM con el mandato `resource-instances-usage`. En función de su nivel de permiso, ejecute los mandatos apropiados.
    - Si es el administrador de la cuenta o se le ha otorgado el rol de administrador para todos los servicios habilitados para identidad y acceso, simplemente ejecute `resource-instances-usage`.
        ```sh
        ibmcloud billing resource-instances-usage
        ```
        {: pre}
    -  Si no es el administrador de la cuenta, se pueden utilizar los mandatos siguientes gracias al rol de visor que se ha establecido al principio de la guía de aprendizaje.
        ```sh
        ibmcloud billing resource-instances-usage -o $ORG_NAME
        ```
        {: pre}

        ```sh
        ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP
        ```
        {: pre}

6. Para ver dispositivos de la infraestructura, utilice el mandato `sl`. Luego utilice el mandato `vs` para revisar {{site.data.keyword.virtualmachinesshort}}.
    ```sh
    ibmcloud sl vs list
    ```
    {: pre}

    ```sh
    ibmcloud sl vs detail <id>
    ```
    {: pre}

## Exportación del uso con la línea de mandatos
{: #usagecmd}

A menudo, los directores de departamento necesitan que los datos se exporten a otra aplicación. Un ejemplo común es la exportación de datos de uso a una hoja de cálculo. En esta sección, exportará los datos de uso al formato de valores separados por coma (CSV), que es compatible con la mayoría de las aplicaciones de hojas de cálculo.

En esta sección se utilizan dos herramientas de terceros: `jq` y `json2csv`. Cada uno de los mandatos siguientes se compone de tres pasos: obtención de datos de uso, como JSON, análisis de JSON y formateo del resultado como CSV. El proceso de obtención y conversión de datos JSON no se limita a `json2csv`; se pueden aprovechar otras herramientas.

Utilice la opción `-p` para ver los resultados en un formato adecuado. Si los datos no se visualizan bien, elimine el argumento `-p` para que se visualicen datos CSV sin formato.
{:tip}

1. Exporte los datos de uso de un grupo de recursos con estimaciones de costes para cada tipo de recurso.
    ```sh
    ibmcloud billing resource-group-usage $RESOURCE_GROUP --output json | \
    jq '.[0].resources[] | {resource_name,billable_cost}' | \
    json2csv -f resource_name,billable_cost -p
    ```
    {: pre}

2. Especifique las instancias para cada tipo de recurso del grupo de recursos.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

3. Siga el mismo enfoque para una organización de Cloud Foundry.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name -p
    ```
    {: pre}

4. Añada una estimación de costes a los datos con una consulta `jq` más avanzada. Esto creará más filas, ya que algunos tipos de recursos tienen varias métricas de costes.
    ```sh
    ibmcloud billing resource-instances-usage -g $RESOURCE_GROUP --output json | \
    jq '.[] | {month,resource_name,resource_instance_name,organization_name,space_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

5. Utilice la misma consulta `jq` para obtener una lista de los recursos de Cloud Foundry con costes asociados.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost -p
    ```
    {: pre}

6. Exporte los datos a un archivo eliminando la opción `-p` y enviando la salida a un archivo. El archivo CSV resultante se puede abrir con una aplicación de hoja de cálculo.
    ```sh
    ibmcloud billing resource-instances-usage -o $ORG_NAME --output json | \
    jq '.[] | {month,resource_name,organization_name,space_name,resource_group_name,metric: .usage[].metric,cost : .usage[].cost}' | \
    json2csv -f month,resource_name,resource_instance_name,organization_name,space_name,metric,cost > $ORG_NAME.csv
    ```
    {: pre}

## Exportación de datos de uso con las API
{: #usageapi}

Aunque los mandatos `billing` resultan útiles, intentar ensamblan una vista "global" con la interfaz de línea de mandatos es una tarea tediosa. Paralelamente, el panel de control de uso ofrece una visión genera de las organizaciones y grupos de recursos, pero no necesariamente del uso de un equipo o de un proyecto. En esta sección empezará a explorar un enfoque más guiado por datos a fin de obtener datos de uso que se ajusten a requisitos personalizados.

1. En el terminal, establezca la variable de entorno `IBMCLOUD_TRACE=true` de modo que imprima las solicitudes y las respuestas de API.
    ```sh
    export IBMCLOUD_TRACE=true
    ```
    {: pre}

2. Vuelva a ejecutar el mandato `billing org-usage` para ver las llamadas de API. Observe que para este mandato se utilizan varios hosts y rutas de API.
    ```sh
    ibmcloud billing org-usage $ORG_NAME
    ```
    {: pre}

3. Obtenga una señal de OAuth y ejecute una de las llamadas de API mostradas en el mandato `billing org-usage`. Tenga en cuenta que algunas API utilizan la señal de UAA, mientras que otras pueden utilizar una señal IAM como autorización de portadora.
    ```sh
    export IAM_TOKEN=`ibmcloud iam oauth-tokens | head -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    export UAA_TOKEN=`ibmcloud iam oauth-tokens | tail -n 1 | awk ' {print $4} '`
    ```
    {: pre}

    ```sh
    curl -H "Authorization: Bearer $UAA_TOKEN" https://mccp.us-south.cf.cloud.ibm.com/v2/organizations?q=name:$ORG_NAME
    ```
    {: pre}

4. Para ejecutar las API de infraestructura, obtenga y establezca las variables de entorno para el **Nombre de usuario de API** y para la **Clave de autenticación** que se ven en el [Perfil de usuario](https://control.softlayer.com/account/user/profile). Si no ve un nombre de usuario y una clave de autenticación de API, puede crearlos en el menú **Acciones** situado junto a su nombre en la [Lista de usuarios](https://control.softlayer.com/account/users).
    ```sh
    export IAAS_USERNAME=<API Username>
    ```
    {: pre}

    ```sh
    export IAAS_KEY=<Authentication Key>
    ```
    {: pre}

5. Obtenga los totales de facturación de la infraestructura y los elementos de facturación mediante las siguientes API. [Aquí](https://softlayer.github.io/reference/services/SoftLayer_Account/) encontrará información sobre API similares.
    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getNextInvoiceTotalAmount
    ```
    {: pre}

    ```sh
    curl -u $IAAS_USERNAME:$IAAS_KEY https://api.softlayer.com/rest/v3/SoftLayer_Account/getAllBillingItems
    ```
    {: pre}

6. Desactive el rastreo.
    ```sh
    export IBMCLOUD_TRACE=false
    ```
    {: pre}

Aunque el enfoque basado en datos proporciona la mayor flexibilidad para explorar el uso, una explicación más completa queda fuera del ámbito de esta guía de aprendizaje. Se ha creado el proyecto GitHub [openwhisk-cloud-usage-sample](https://github.com/IBM-Cloud/openwhisk-cloud-usage-sample) que combina servicios de {{site.data.keyword.cloud_notm}} con Cloud Functions para ofrecer una implementación de ejemplo.

## Ampliación de la guía de aprendizaje
{: #expand}

Aquí encontrará algunas sugerencias para ampliar su investigación del inventario y de los datos relacionados con el uso.

- Explore los mandatos de `ibmcloud billing` con la opción `--output json`. Esto mostrará las propiedades de datos adicionales disponibles que no quedan cubiertas en la guía de aprendizaje.
- Lea la publicación de blog sobre [Utilización de la línea de mandatos de {{site.data.keyword.cloud_notm}} para buscar recursos](https://www.ibm.com/blogs/bluemix/2018/06/where-are-my-resources/) para ver más ejemplos de `búsqueda de recursos de ibmcloud` y las propiedades se pueden utilizar en las consultas.
- Revise las [API de la cuenta de infraestructura](https://softlayer.github.io/reference/services/SoftLayer_Account/) para ver más API a fin de investigar el uso de la infraestructura.
- Revise el [manual de jq](https://stedolan.github.io/jq/manual/) para ver consultas avanzadas para agregar datos de uso.
