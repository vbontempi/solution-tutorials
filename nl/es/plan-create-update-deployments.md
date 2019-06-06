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

# Planificación, creación y actualización de entornos de despliegue
{: #plan-create-update-deployments}

Resulta habitual disponer de varios entornos de despliegue cuando se crea una solución. Reflejan el ciclo de vida de un proyecto desde el desarrollo hasta la producción. En esta guía de aprendizaje se presentan herramientas como la CLI de {{site.data.keyword.Bluemix_notm}} y [Terraform](https://www.terraform.io/) para automatizar la creación y el mantenimiento de estos entornos de despliegue.
{:shortdesc}

A los desarrolladores no les gusta escribir lo mismo dos veces. El principio [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) es un claro ejemplo. Del mismo modo, no les gusta tener que efectuar miles de pulsaciones en una interfaz de usuario para configurar un entorno. Por consiguiente, los administradores del sistema y los desarrolladores han utilizado durante mucho tiempo scripts de shell para automatizar las tareas repetitivas, propensas a que se cometan errores y poco interesantes.

Infraestructura como servicio (IaaS), Plataforma como servicio (PaaS), Contenedor como servicio (CaaS), Funciones como servicio (FaaS) han proporcionado a los desarrolladores un alto nivel de abstracción y les ha facilitado la adquisición de recursos como servidores nativos, bases de datos gestionadas, máquinas virtuales, clústeres de Kubernetes, etc. Pero, una vez que se han suministrado estos recursos, es necesario conectarlos, configurar el acceso de los usuarios, actualizar la configuración a lo largo del tiempo, etc. Ser capaz de automatizar todos estos pasos y repetir la instalación y la configuración bajo diferentes entornos se ha convertido en una necesidad.

Es habitual utilizar varios entornos en un proyecto para dar soporte a las distintas fases del ciclo de desarrollo con ligeras diferencias entre los entornos, como la capacidad, la red, las credenciales y el nivel de detalle del registro. En [esta otra guía de aprendizaje](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications), hemos incorporado las prácticas recomendadas para organizar usuarios, equipos y aplicaciones y un escenario de ejemplo. El caso de ejemplo de ejemplo tiene en cuenta tres entornos: *Desarrollo*, *Prueba* y *Producción*. ¿Cómo se automatiza la creación de estos entornos? ¿Qué herramientas se pueden utilizar?

## Objetivos
{: #objectives}

* Definición del conjunto de entornos que se van a desplegar
* Escritura de scripts mediante la CLI de {{site.data.keyword.Bluemix_notm}} y [Terraform](https://www.terraform.io/) para automatizar el despliegue de estos entornos
* Despliegue de estos entornos en su cuenta

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los productos siguientes:
* [Proveedor de {{site.data.keyword.Bluemix_notm}} para Terraform](https://ibm-cloud.github.io/tf-ibm-docs/index.html)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [Gestión de identidad y acceso](https://{DomainName}/iam/#/users)
* [Interfaz de línea de mandatos de {{site.data.keyword.Bluemix_notm}}: la CLI de `ibmcloud`](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)
* [HashiCorp Terraform](https://www.terraform.io/)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

![](./images/solution26-plan-create-update-deployments/architecture.png)
</p>

1. Se crea un conjunto de archivos Terraform para describir la infraestructura de destino como código.
1. Un operador utiliza `terraform apply` para suministrar los entornos.
1. Se escriben scripts de shell para completar la configuración de los entornos.
1. El operador ejecuta los scripts en los entornos
1. Los entornos quedan totalmente configurados, listos para ser utilizados.

## Visión general de las herramientas disponibles
{: #tools}

La primera herramienta para interactuar con {{site.data.keyword.Bluemix_notm}} y para crear despliegues que se puedan reproducir es la interfaz de línea de mandatos de [{{site.data.keyword.Bluemix_notm}}: la CLI `ibmcloud`](/docs/cli?topic=cloud-cli-install-ibmcloud-cli). Con `ibmcloud` y sus plugins, puede automatizar la creación y la configuración de los recursos de la nube. {{site.data.keyword.virtualmachinesshort}}, clústeres de Kubernetes, {{site.data.keyword.openwhisk_short}}, apps y servicios de Cloud Foundry: puede suministrar todos ellos desde la línea de mandatos.

Otra herramienta que se incorpora en [esta guía de aprendizaje](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform) es [Terraform](https://www.terraform.io/) de HashiCorp. Citando a HashiCorp, *Terraform le permite crear, cambiar y mejorar la infraestructura de forma segura y predecible. Es una herramienta de código abierto que codifica las API en archivos de configuración declarativos que se pueden compartir entre miembros del equipo, tratar como código, editar, revisar y se pueden crear versiones de los mismos.* Es infraestructura como código. El usuario escribe la base de la infraestructura y Terraform crea, actualiza y elimina los recursos de la nube según sea necesario.

Para dar soporte a un enfoque de varias nubes, Terraform trabaja con proveedores. Un proveedor es el responsable de comprender las interacciones de API y de exponer los recursos. {{site.data.keyword.Bluemix_notm}} tiene [su proveedor para Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm), que permite a los usuarios de {{site.data.keyword.Bluemix_notm}} gestionar recursos con Terraform. Aunque Terraform es un producto clasificado como infraestructura como código, no se limita a los recursos de una infraestructura como servicio. El proveedor de {{site.data.keyword.Bluemix_notm}} para Terraform da soporte a recursos IaaS (servidor nativo, máquina virtual, servicios de red, etc.), CaaS ({{site.data.keyword.containershort_notm}} y clústeres de Kubernetes), PaaS (Cloud Foundry y servicios) y FaaS ({{site.data.keyword.openwhisk_short}}).

## Escritura de scripts para automatizar el despliegue
{: #scripts}

A medida que empieza a describir su infraestructura como código, resulta de vital importancia tratar los archivos que crea como código normal, guardándolos en un sistema de gestión de control de origen. Con el tiempo, esto le aportará importantes ventajas, como por ejemplo utilizar el flujo de trabajo de revisión de control de origen para validar los cambios antes de aplicarlos, añadiendo un conducto de integración continua para desplegar automáticamente los cambios de la infraestructura.

[Este repositorio Git](https://github.com/IBM-Cloud/multiple-environments-as-code) tiene todos los archivos de configuración necesarios para configurar los entornos definidos anteriormente. Puede clonar el repositorio para seguir las siguientes secciones, donde se detalla el contenido de los archivos.

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```

El repositorio está estructurado del siguiente modo:

| Directorio | Descripción |
| ----------------- | ----------- |
| [terraform](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform) | Directorio inicial de los archivos de Terraform |
| [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global) | Archivos de Terraform para suministrar los recursos comunes a los tres entornos |
| [terraform/per-environment](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/per-environment) | Archivos de Terraform específicos de un entorno determinado |
| [terraform/roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) | Archivos de terraform para configurar políticas de usuario |

### Agilización del proceso con Terraform

Los entornos de *Desarrollo*, *Prueba* y *Producción* son muy parecidos.

<p style="text-align: center;">
  <img title="" src="./images/solution26-plan-create-update-deployments/one-environment.png" style="height: 400px;" alt="Diagrama que muestra un entorno de despliegue" />
</p>

Comparten una organización común y recursos específicos del entorno. Difieren en cuanto a capacidad asignada y a derechos de acceso. Los archivos de terraform reflejan este hecho con una configuración ***global*** para suministrar la organización Cloud Foundry y una configuración ***por entorno***, mediante el uso de espacios de trabajo Terraform, para suministrar los recursos específicos del entorno:

![](./images/solution26-plan-create-update-deployments/terraform-workspaces.png)

### Configuración global

Todos los entornos comparten una organización común de Cloud Foundry y cada entorno tiene su propio espacio.

En el directorio [terraform/global](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/global), encontrará los scripts de Terraform para suministrar esta organización. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/main.tf) contiene la definición correspondiente a la organización:

   ```sh
   # crear una nueva organización para el proyecto
   resource "ibm_org" "organization" {
     name             = "${var.org_name}"
     managers         = "${var.org_managers}"
     users            = "${var.org_users}"
     auditors         = "${var.org_auditors}"
     billing_managers = "${var.org_billing_managers}"
   }
   ```

En este recurso, todas las propiedades se configuran mediante variables. En las secciones siguientes, aprenderá a definir estas variables.

Para desplegar por completo los entornos, utilizará una combinación de Terraform y CLI de {{site.data.keyword.Bluemix_notm}}. Los scripts de shell escritos con la CLI pueden tener que hacer referencia a esta organización o a la cuenta por nombre o por ID. El directorio *global* también incluye [outputs.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/outputs.tf), que generará un archivo que contendrá esta información como claves/valores que se podrán reutilizar en scripts:

   ```sh
   # generar un archivo de propiedades adecuado para los scripts de shell con variables útiles relacionadas con el entorno
   resource "local_file" "output" {
     content = <<EOF
ACCOUNT_GUID=${data.ibm_account.account.id}
ORG_GUID=${ibm_org.organization.id}
ORG_NAME=${var.org_name}
EOF

     filename = "../outputs/global.env"
   }
   ```

### Entornos individuales

Existen diferentes enfoques para gestionar varios entornos con Terraform. Puede duplicar los archivos de Terraform en distintos directorios, uno por entorno. Con [módulos de Terraform](https://www.terraform.io/docs/modules/index.html), puede factorizar la configuración común como un grupo y reutilizar módulos entre entornos, reduciendo así la duplicación de código. Distintos directorios significa que puede desarrollar el entorno de *desarrollo* para probar los cambios y luego propagar los cambios a otros entornos. En este caso, es habituar disponer de *módulos modules* de Terraform en su propio repositorio de código abierto para poder hacer referencia a una versión específica de un módulo en sus archivos de entorno.

Dado que los entornos son bastante sencillos y parecidos, utilizará otro concepto de Terraform denominado [espacios de trabajo](https://www.terraform.io/docs/state/workspaces.html#best-practices). Los espacios de trabajo le permiten utilizar los mismos archivos de terraform (.tf) con distintos entornos. En el ejemplo, *development* (desarrollo), *testing* (prueba) y *production* (producción) son espacios de trabajo. Utilizarán las mismas definiciones de Terraform, pero con diferentes variables de configuración (distintos nombres y distintas capacidades).

Cada entorno necesita:
* un espacio de Cloud Foundry dedicado
* un grupo de recursos dedicado
* un clúster de Kubernetes
* una base de datos
* un almacén de archivos

El espacio de Cloud Foundry está enlazado a la organización creada en el paso anterior. Los archivos de Terraform del entorno tienen que hacer referencia a esta organización. Aquí es donde el [estado remoto de Terraform](https://www.terraform.io/docs/state/remote.html) supone una ayuda. Permite hacer referencia a un estado existente de Terraform en modalidad de solo lectura. Esta es una construcción muy útil para dividir la configuración de Terraform en piezas más pequeñas, asignando la responsabilidad sobre las piezas individuales a diferentes equipos. [backend.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/backend.tf) contiene la definición del estado remoto *global* que se utiliza para buscar la organización creada anteriormente:

   ```sh
   data "terraform_remote_state" "global" {
      backend = "local"

      config {
        path = "${path.module}/../global/terraform.tfstate"
      }
   }
   ```

Una vez que se puede hacer referencia a la organización, resulta sencillo crear un espacio dentro de esta organización. [main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/per-environment/main.tf) contiene la definición correspondiente a los recursos del entorno.

   ```sh
   # un espacio de Cloud Foundry por entorno
   resource "ibm_space" "space" {
     name       = "${var.environment_name}"
     org        = "${data.terraform_remote_state.global.org_name}"
     managers   = "${var.space_managers}"
     auditors   = "${var.space_auditors}"
     developers = "${var.space_developers}"
   }
   ```

Observe cómo se hace referencia al nombre de la organización desde el estado remoto *global*. Las otras propiedades se toman de las variables de configuración.

A continuación viene el grupo de recursos.

   ```sh
   # un grupo de recursos
   resource "ibm_resource_group" "group" {
    name     = "${var.environment_name}"
    quota_id = "${data.ibm_resource_quota.quota.id}"
}

   data "ibm_resource_quota" "quota" {
	name = "${var.resource_quota}"
}
   ```

El clúster de Kubernetes se crea en este grupo de recursos. El proveedor de {{site.data.keyword.Bluemix_notm}} tiene un recurso Terraform para representar un clúster:

   ```sh
  # un clúster
  resource "ibm_container_cluster" "cluster" {
  	name              = "${var.environment_name}-cluster"
  	datacenter        = "${var.cluster_datacenter}"
  	org_guid          = "${data.terraform_remote_state.global.org_guid}"
  	space_guid        = "${ibm_space.space.id}"
  	account_guid      = "${data.terraform_remote_state.global.account_guid}"
  	hardware          = "${var.cluster_hardware}"
  	machine_type      = "${var.cluster_machine_type}"
  	public_vlan_id    = "${var.cluster_public_vlan_id}"
  	private_vlan_id   = "${var.cluster_private_vlan_id}"
  	resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool" "cluster_workerpool" {
  worker_pool_name  = "${var.environment_name}-pool"
  machine_type      = "${var.cluster_machine_type}"
  cluster           = "${ibm_container_cluster.cluster.id}"
  size_per_zone     = "${var.worker_num}"
  hardware          = "${var.cluster_hardware}"
  resource_group_id = "${ibm_resource_group.group.id}"
}

resource "ibm_container_worker_pool_zone_attachment" "cluster_zone" {
  cluster           = "${ibm_container_cluster.cluster.id}"
  worker_pool       =  "${element(split("/",ibm_container_worker_pool.cluster_workerpool.id),1)}"
  zone              = "${var.cluster_datacenter}"
  public_vlan_id    = "${var.cluster_public_vlan_id}"
  private_vlan_id   = "${var.cluster_private_vlan_id}"
  resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

De nuevo, la mayoría de las propiedades se inicializarán a partir de variables de configuración. Usted puede ajustar el centro de datos, el número de nodos trabajadores y el tipo de nodos trabajadores.

Los servicios habilitados para IAM, como {{site.data.keyword.cos_full_notm}} y {{site.data.keyword.cloudant_short_notm}}, también se crean como recursos dentro del grupo:

   ```sh
# una base de datos
resource "ibm_resource_instance" "database" {
    name              = "database"
    service           = "cloudantnosqldb"
    plan              = "${var.cloudantnosqldb_plan}"
    location          = "${var.cloudantnosqldb_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
# un almacenamiento de objetos en la nube
resource "ibm_resource_instance" "objectstorage" {
    name              = "objectstorage"
    service           = "cloud-object-storage"
    plan              = "${var.cloudobjectstorage_plan}"
    location          = "${var.cloudobjectstorage_location}"
    resource_group_id = "${ibm_resource_group.group.id}"
}
   ```

Se pueden añadir enlaces de Kubernetes (secretos) para recuperar las credenciales de servicio de las aplicaciones:

   ```sh
   # enlazar el servicio cloudant al clúster
   resource "ibm_container_bind_service" "bind_database" {
      cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	  service_instance_name       = "${ibm_resource_instance.database.name}"
      namespace_id                = "default"
      account_guid                = "${data.terraform_remote_state.global.account_guid}"
      org_guid                    = "${data.terraform_remote_state.global.org_guid}"
      space_guid                  = "${ibm_space.space.id}"
      resource_group_id           = "${ibm_resource_group.group.id}"
}

   # enlazar el servicio cloud object storage al clúster
   resource "ibm_container_bind_service" "bind_objectstorage" {
  	cluster_name_id             = "${ibm_container_cluster.cluster.id}"
  	space_guid                  = "${ibm_space.space.id}"
  	service_instance_id         = "${ibm_resource_instance.objectstorage.name}"
  	namespace_id                = "default"
  	account_guid                = "${data.terraform_remote_state.global.account_guid}"
  	org_guid                    = "${data.terraform_remote_state.global.org_guid}"
  	space_guid                  = "${ibm_space.space.id}"
  	resource_group_id           = "${ibm_resource_group.group.id}"
}
   ```

## Despliegue de este entorno en la cuenta

### Instalación de la CLI de {{site.data.keyword.Bluemix_notm}}

1. Siga [estas instrucciones](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) para instalar la CLI
1. Valide la instalación con este mandato:
   ```sh
   ibmcloud
   ```
   {: codeblock}

### Instalación de Terraform y del proveedor de {{site.data.keyword.Bluemix_notm}} para Terraform

1. [Descargue e instale Terraform para el sistema.](https://www.terraform.io/intro/getting-started/install.html)
1. [Descargue el binario de Terraform correspondiente al proveedor de {{site.data.keyword.Bluemix_notm}}.](https://github.com/IBM-Cloud/terraform-provider-ibm/releases).
   Para configurar Terraform con el proveedor de {{site.data.keyword.Bluemix_notm}}, consulte este [enlace](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#setup)
   {:tip}
1. Cree un archivo `.terraformrc` en el directorio de inicio que apunte al binario de Terraform. En el siguiente ejemplo, `/opt/provider/terraform-provider-ibm` es la ruta al directorio.
   ```sh
   # ~/.terraformrc
    providers {
     ibm = "/opt/provider/terraform-provider-ibm_VERSION"
   }
   ```
   {: codeblock}

### Obtención del código

Si todavía no lo ha hecho, clone el repositorio de la guía de aprendizaje:

   ```sh
   git clone https://github.com/IBM-Cloud/multiple-environments-as-code
   ```
   {: codeblock}

### Definición de la clave de API de la plataforma

1. Si todavía no lo ha hecho, obtenga una [clave de API de la plataforma](https://{DomainName}/iam/#/apikeys) y guarde la clave de API para futuras referencias.

   > Si en los pasos siguientes va a crear una nueva organización de Cloud Foundry para alojar los entornos de despliegue, asegúrese de ser el propietario de la cuenta.
1. Copie [terraform/credentials.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/credentials.tfvars.tmpl) en *terraform/credentials.tfvars* con el mandato siguiente
   ```sh
   cp terraform/credentials.tfvars.tmpl terraform/credentials.tfvars
   ```
   {: codeblock}
1. Edite `terraform/credentials.tfvars` y establezca el valor de `ibmcloud_api_key` en la clave de API de la plataforma que ha obtenido.

### Creación o reutilización de una organización de Cloud Foundry

Puede elegir entre crear una nueva organización o reutilizar (importar) una existente. Para crear la organización padre de los tres entornos de despliegue, **tiene que ser el propietario de la cuenta**.

#### Para crear una nueva organización

1. Vaya al directorio `terraform/global`
1. Copie [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) en `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Edite `global.tfvars`
   1. Establezca **org_name** en el nombre de la organización que va a crear
   1. Establezca **org_managers** en una lista de los ID de usuario a los que desea otorgar el rol de *Gestor* en la org; el usuario que crea la org es automáticamente un gestor y no se debe añadir a la lista
   1. Establezca **org_users** en una lista de todos los usuarios que desea invitar a la org; los usuarios se tienen que añadir si desea configurar su acceso en pasos siguientes

   ```sh
   org_name = "a-new-organization"
   org_managers = [ "user1@domain.com", "another-user@anotherdomain.com" ]
   org_users = [ "user1@domain.com", "another-user@anotherdomain.com", "more-user@domain.com" ]
   ```
   {: codeblock}
1. Inicialice Terraform desde la carpeta `terraform/global`
   ```sh
   terraform init
   ```
   {: codeblock}
1. Revise el plan de Terraform
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}
1. Aplique los cambios.
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

Cuando Terraform termine, habrá creado:
* una nueva organización de Cloud Foundry
* un archivo `global.env` en el directorio `outputs` de extracción. Este archivo tiene variables de entorno a las que podría hacer referencia en otros scripts
* el archivo `terraform.tfstate`

> En esta guía de aprendizaje se utiliza el proveedor de programa de fondo `local` para el estado de Terraform. Esto resulta práctico cuando se descubre Terraform o cuando se trabaja solo en un proyecto, pero cuando se trabaja en equipo, o en una infraestructura grande, Terraform también da soporte a la operación de guardar el estado en una ubicación remota. Dado que el estado de Terraform resulta crítico para las operaciones de Terraform, se recomienda utilizar un almacenamiento remoto, altamente disponible y resistente para el estado de Terraform. Consulte [Tipos de programa de fondo de Terraform](https://www.terraform.io/docs/backends/types/index.html) para ver una lista de las opciones disponibles. Algunos programas de fondo permiten crear versiones y bloquear los estados de Terraform.

#### Reutilización de una organización que está gestionando

Si no es el propietario de la cuenta, pero gestiona una organización en la cuenta, también puede importar una organización existente en Terraform

1. Recupere el GUID de la organización
   ```sh
   ibmcloud iam org <org_name> --guid
   ```
   {: codeblock}
1. Vaya al directorio `terraform/global`
1. Copie [global.tfvars.tmpl](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/global/global.tfvars.tmpl) en `global.tfvars`
   ```sh
   cp global.tfvars.tmpl global.tfvars
   ```
   {: codeblock}
1. Inicialice Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. Después de inicializar Terraform, importe la organización en el estado de Terraform.
   ```sh
   terraform import -var-file=../credentials.tfvars -var-file=global.tfvars ibm_org.organization <guid>
   ```
   {: codeblock}
1. Ajuste `global.tfvars` para que coincida con el nombre y la estructura de la organización existente
1. Aplique los cambios.
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

### Creación de un espacio, un clúster y servicios por entorno

Esta sección se centrará en el entorno de `desarrollo`. Los pasos serán los mismos para los otros entornos, solo los valores que elija para las variables serán distintos.

1. Vaya a la carpeta `terraform/per-environment` del directorio extracción
1. Copie el archivo `tfvars` de plantilla. Hay uno por entorno:
   ```sh
   cp development.tfvars.tmpl development.tfvars
   cp testing.tfvars.tmpl testing.tfvars
   cp production.tfvars.tmpl production.tfvars
   ```
   {: codeblock}
1. Edite `development.tfvars`
   1. Establezca **environment_name** en el nombre del espacio de Cloud Foundry que desea crear
   1. Establezca **space_developers** en la lista de desarrolladores para este espacio. **Asegúrese de añadir su nombre a la lista para que Terraform pueda suministrar servicios en su nombre.**
   1. Establezca **cluster_datacenter** en la ubicación en la que desea crear el clúster. Localice las ubicaciones disponibles con:
      ```sh
      ibmcloud cs locations
      ```
      {: codeblock}
   1. Establezca las VLAN privada (**cluster_private_vlan_id**) y pública (**cluster_public_vlan_id**) para el clúster. Para localizar las VLAN disponibles para la ubicación, ejecute:
      ```sh
      ibmcloud cs vlans <location>
      ```
      {: codeblock}
   1. Establezca **cluster_machine_type**. Para localizar los tipos de máquina y las características disponibles para la ubicación, ejecute:
      ```sh
      ibmcloud cs machine-types <location>
      ```
      {: codeblock}

   1. Establezca **resource_quota**. Localice las definiciones de cuota de recurso disponibles con:
      ```sh
      ibmcloud resource quotas
      ```
      {: codeblock}
1. Inicialice Terraform
   ```sh
   terraform init
   ```
   {: codeblock}
1. Cree un espacio de trabajo de Terraform nuevo para el entorno de *desarrollo*
   ```sh
   terraform workspace new development
   ```
   {: codeblock}
   Luego, para cambiar de entorno, utilice
   ```sh
   terraform workspace select development
   ```
   {: codeblock}
1. Revise el plan de Terraform
   ```sh
   terraform plan -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Debería mostrar lo siguiente:
   ```
   Plan: 7 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
1. Aplique los cambios.
   ```sh
   terraform apply -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}

Cuando Terraform termine, habrá creado:
* un grupo de recursos
* un espacio de Cloud Foundry
* un clúster de Kubernetes con una agrupación de nodos trabajadores y una zona conectada
* una base de datos
* un secreto de Kubernetes con las credenciales de la base de datos
* un almacenamiento
* un secreto de Kubernetes con las credenciales de almacenamiento
* una instancia de registro (LogDNA)
* una instancia de supervisión (Sysdig)
* un archivo `development.env` en el directorio `outputs` de extracción. Este archivo tiene variables de entorno a las que podría hacer referencia en otros scripts
* el archivo `terraform.tfstate` específico del entorno bajo `terraform.tfstate.d/development`.

Puede repetir los pasos para los entornos de `prueba` y de `producción`.

### Asignación de políticas de usuario

En los pasos anteriores, los roles de la organización de Cloud Foundry y los espacios se pueden configurar con el proveedor de Terraform. Para políticas de usuario en otros recursos, como clústeres de Kubernetes, utilizará la carpeta [roles](https://github.com/IBM-Cloud/multiple-environments-as-code/tree/master/terraform/roles) del repositorio clonado.

Para el entorno de *desarrollo* definido en [esta guía de aprendizaje](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications), las políticas que debe definir son las siguientes:

|           | Políticas de acceso de IAM |
| --------- | ----------- |
| Desarrollador | <ul><li>Grupo de recursos: *Visor*</li><li>Roles de acceso a la plataforma en el grupo de recursos: *Visor*</li><li>Rol del servicio de registro y supervisión: *Escritor*</li></ul> |
| Comprobador    | <ul><li>No se necesita ninguna configuración. El comprobador accede a la aplicación desplegada, no a los entornos de desarrollo</li></ul> |
| Operador  | <ul><li>Grupo de recursos: *Visor*</li><li>Roles de acceso a la plataforma en el grupo de recursos: *Operador*, *Visor*</li><li>Rol del servicio de registro y supervisión: *Escritor*</li></ul> |
| Usuario funcional del conducto | <ul><li>Grupo de recursos: *Visor*</li><li>Roles de acceso a la plataforma en el grupo de recursos: *Editor*, *Visor*</li></ul> |

Dado que un equipo puede estar compuesto por varios desarrolladores y comprobadores, puede aprovechar el [concepto de grupo de acceso](https://{DomainName}/docs/iam?topic=iam-groups#groups) para simplificar la configuración de políticas de usuario. El propietario de la cuenta puede crear grupos de acceso para que se pueda asignar el mismo acceso a todas las entidades dentro del grupo con una única política.

Para el rol de *Desarrollador* en el entorno de *Desarrollo*, esto se traduce en:

   ```sh
resource "ibm_iam_access_group" "developer_role" {
  name        = "${var.access_group_name_developer_role}"
  description = "${var.access_group_description}"
}
resource "ibm_iam_access_group_policy" "resourcepolicy_developer" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Viewer"]

  resources = [{
    resource_type = "resource-group"
    resource      = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_platform_accesspolicy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Viewer"]

  resources = [{
    resource_group_id = "${data.terraform_remote_state.per_environment_dev.resource_group_id}"
  }]
}

resource "ibm_iam_access_group_policy" "developer_logging_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "logdna"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.logdna_instance_id}"
  }]
}
resource "ibm_iam_access_group_policy" "developer_monitoring_policy" {
  access_group_id = "${ibm_iam_access_group.developer_role.id}"
  roles           = ["Writer"]

  resources = [{
    service           = "sysdig-monitor"
    resource_instance_id = "${data.terraform_remote_state.per_environment_dev.sysdig_instance_id}"
  }]
}

   ```

El archivo [roles/development/main.tf](https://github.com/IBM-Cloud/multiple-environments-as-code/blob/master/terraform/roles/development/main.tf) del directorio de extracción contiene ejemplos de estos recursos para los roles de *Desarrollador*, *Operador*, *probador* y *Usuario funcional* definidos. Para establecer las políticas tal como se ha definido en una sección anterior para los usuarios con los roles de *Desarrollador, Operador, Probador y Usuario funcional* en el entorno de *desarrollo*,

1. Vaya al directorio `terraform/roles/development`
2. Copie el archivo `tfvars` de plantilla. Hay uno por entorno (encontrará las plantillas `production` y `testing` bajo sus respectivas carpetas en el directorio `roles`)

   ```sh
   cp development.tfvars.tmpl development.tfvars
   ```
3. Edite `development.tfvars`

   - Establezca **iam_access_members_developers** en la lista de desarrolladores a los que desea otorgar el acceso.
   - Establezca **iam_access_members_operators** en la lista de operadores, y así sucesivamente.
4. Inicialice Terraform
   ```sh
   terraform init
   ```
   {: codeblock}

5. Revise el plan de Terraform
   ```sh
   terraform plan -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
   Debería mostrar lo siguiente:
   ```
   Plan: 14 to add, 0 to change, 0 to destroy.
   ```
   {: codeblock}
6. Aplique los cambios.
   ```sh
   terraform apply -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
Puede repetir los pasos para los entornos de `prueba` y de `producción`.

## Eliminación de recursos

1. Vaya a la carpeta `development` en `roles`
   ```sh
   cd terraform/roles/development
   ```
   {: codeblock}
1. Elimine los grupos de acceso y las políticas de acceso
   ```sh
   terraform destroy -var-file=../../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Active el espacio de trabajo `development`
   ```sh
   cd terraform/per-environment
   terraform workspace select development
   ```
   {: codeblock}
1. Elimine el grupo de recursos, los espacios, los servicios y los clústeres
   ```sh
   terraform destroy -var-file=../credentials.tfvars -var-file=development.tfvars
   ```
   {: codeblock}
1. Repita los pasos para los entornos de `prueba` y de `producción`.
1. Si la ha creado, elimine la organización
   ```sh
   cd terraform/global
   terraform destroy -var-file=../credentials.tfvars -var-file=global.tfvars
   ```
   {: codeblock}

## Contenido relacionado

* [Guía de aprendizaje de Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
* [Proveedor de Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
* [Ejemplos de utilización del proveedor de {{site.data.keyword.Bluemix_notm}} para Terraform](https://github.com/IBM-Cloud/terraform-provider-ibm/tree/master/examples)
