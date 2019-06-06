---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-23"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Despliegue de una pila LAMP mediante Terraform
{: #infrastructure-as-code-terraform}

[Terraform](https://www.terraform.io/) le permite crear, cambiar y mejorar la infraestructura de forma segura y predecible. Es una herramienta de código abierto que codifica las API en archivos de configuración declarativos que se pueden compartir entre miembros del equipo, tratar como código, editar, revisar y se pueden crear versiones de los mismos.

En esta guía de aprendizaje, utilizará una configuración de ejemplo para suministrar un servidor virtual **L**inux, con un servidor web **A**pache, **M**ySQL y un servidor **P**HP denominado pila **LAMP**. Luego actualizará la configuración para añadir el servicio {{site.data.keyword.cos_full_notm}} y escalar los recursos para ajustar el entorno (memoria, CPU y tamaño de disco). Para terminar, suprimirá todos los recursos creados por la configuración.

## Objetivos
{: #objectives}

* Configuración de Terraform y del proveedor de {{site.data.keyword.Bluemix_notm}} para Terraform.
* Utilización de Terraform para crear, actualizar, escalar y finalmente destruir una configuración de pila LAMP.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.virtualmachinesshort}}
](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

  ![Diagrama de la arquitectura](images/solution10/architecture-2.png)
</p>

1. Se crea un conjunto de archivos Terraform para describir la configuración de la pila LAMP.
1. Se invoca `terraform` desde el directorio de configuración.
1. `terraform` llama a la API de {{site.data.keyword.cloud_notm}} para suministrar los recursos.

## Antes de empezar
{: #prereqs}

Póngase en contacto con el usuario maestro de la infraestructura para obtener los permisos siguientes:
- Red (para añadir un **enlace ascendente de red pública y privada**)
- Clave de API

## Requisitos previos

{: #prereq}

Instale **Terraform** mediante el [programa de instalación](https://www.terraform.io/intro/getting-started/install.html) o utilice [Homebrew](https://brew.sh/) en macOS ejecutando este mandato: `brew install terraform`

En **Windows**, siga estos pasos para configurar terraform:
   1. Copie los archivos del zip que ha descargado en `C:\terraform` (cree la carpeta `terraform`).
   2. Abra el indicador de mandatos como administrador y establezca la variable PATH de modo que utilice los binarios de terraform.

      ```
      set PATH=%PATH%;C:\terraform
      ```
      {:pre}

## Configuración del proveedor de {{site.data.keyword.Bluemix_notm}} para Terraform
{: #setup}

Para dar soporte a un enfoque de varias nubes, Terraform trabaja con proveedores. Un proveedor es el responsable de comprender las interacciones de API y de exponer los recursos. En esta sección, configurará la CLI para especificar la ubicación del proveedor de {{site.data.keyword.Bluemix_notm}}.

1. Compruebe la instalación de Terraform ejecutando `terraform` en el terminal o en la ventana del indicador de mandatos.  Debería ver una lista de `mandatos comunes`.

  ```
  terraform
  ```

2. Descargue el plugin de [proveedor de {{site.data.keyword.Bluemix_notm}}](https://github.com/IBM-Cloud/terraform-provider-ibm/releases) adecuado para su sistema y descomprima el archivo. Debería ver el archivo del plugin binario `terraform-provider-ibm_VERSION`.

3. Para sistemas que no sean Windows, cree un directorio `.terraform.d/plugins` en el directorio inicial del usuario y coloque en el mismo el archivo binario. Utilice los mandatos siguientes como referencia.

  ```
  mkdir -p $HOME/.terraform.d/plugins
  mv $HOME/Downloads/terraform-provider-ibm_VERSION $HOME/.terraform.d/plugins/
  ```

    En **Windows**, el archivo se debe colocar en `terraform.d/plugins` bajo el directorio "Application Data" del usuario.

  - Ejecute los mandatos siguientes en un indicador de mandatos de [Configuración de proveedor](https://www.terraform.io/docs/configuration/providers.html)
   ```
  MD %USERPROFILE%\AppData\terraform.d\plugins
   ```
  ```
   MOVE PATH_TO_UNZIPPED_PROVIDER_FILE\terraform-provider-ibm_VERSION.exe  %USERPROFILE%\AppData\terraform.d\plugins
  ```
   - Inicie **Windows Powershell** (Inicio + R > Powershell) y ejecute el siguiente mandato para crear el archivo `terraform.rc`.
   ```
    echo > $env:APPDATA\terraform.rc
   ```
   En la primera solicitud, especifique el contenido siguiente
   ```
    # ~/.terraformrc
    providers {
        ibm = "PATH_TO_YOUR_APPDATA_PLUGINS/terraform-provider-ibm_VERSION.exe"
    }
   ```
        El valor de PATH_TO_YOUR_APPDATA_PLUGINS debería ser una vía de acceso absoluta con barra inclinada (/). Por ejemplo, `C:/Users/VMac/AppData/terraform.d/plugins/terraform-provider-ibm.exe`
        {: tip}

  - Pulse Intro para salir del indicador.

## Preparación de la configuración de terraform

{: #terraformconfig}

En esta sección, aprenderá los conceptos básicos de una configuración de terraform utilizando una configuración de Terraform de ejemplo proporcionada por {{site.data.keyword.Bluemix_notm}}.

1. Vaya a https://github.com/IBM-Cloud/LAMP-terraform-ibm y **desvíe** su propia copia a su cuenta.
2. Clone la copia localmente:
   ```bash
   git clone https://github.com/YOUR_USER_NAME/LAMP-terraform-ibm
   ```
3. Revise los archivos de configuración:
   - [install.yml](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/install.yml): contiene configuraciones de instalación de servidor; aquí es donde puede añadir todos los scripts relacionados con la instalación del servidor. Consulte `phpinfo();` inyectado en este archivo.
   - [provider.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/provider.tf): contiene las variables relacionadas con el proveedor en el que se necesitan el nombre de usuario del proveedor y la clave de api.
   - [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf): contiene configuraciones de servidor para desplegar la VM con variables especificadas.
   - [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars): contiene nombre de usuario y clave de api de **SoftLayer**, clave de API de {{site.data.keyword.Bluemix_notm}} y sus nombres de espacio y de organización. Se recomienda añadir estas credenciales a este archivo para evitar tener que volver a especificar dichas credenciales desde la línea de mandatos cada vez que se despliega el servidor. Nota: NO publique este archivo con sus credenciales.
4. Abra el archivo [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) en el IDE que elija y modifique el archivo añadiendo la clave **SSH pública**. Se utilizará para acceder a la máquina virtual que crea esta configuración. Para obtener información sobre cómo crear claves SSH, siga las instrucciones de [este enlace](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html) Para copiar la clave pública en su portapapeles, puede ejecutar el siguiente mandato en su terminal.

     ```bash
     pbcopy < ~/.ssh/id_rsa.pub
     ```
     Este mandato copiará la SSH en el portapapeles; luego puede pegar este [vm.tf](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/vm.tf) bajo la variable predeterminada `ssh_key` sobre la línea 69.

    En **Windows**, descargue, instale e inicie [Git Bash](https://git-scm.com/download/win) y ejecute el mandato siguiente para copiar la clave SSH pública en el portapapeles.

    ```
     clip < ~/.ssh/id_rsa.pub
    ```

5. Abra el archivo [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) con el IDE, modifique el archivo añadiendo todas las credenciales listadas; la adición de estas credenciales a estos archivos significa que no es necesario que vuelva a especificar estas credenciales cada vez que se ejecute terraform. Debe añadir las cinco credenciales listadas en el archivo [terraform.tfvars](https://github.com/IBM-Cloud/LAMP-terraform-ibm/blob/master/terraform.tfvars) para completar el resto de esta guía de aprendizaje.

## Creación de un servidor de pila LAMP a partir de la configuración de terraform
{: #Createserver}
En esta sección, aprenderá a crear un servidor de pila LAMP a partir del ejemplo de configuración de terraform. La configuración se utiliza para suministrar una instancia de máquina virtual e instalar **A**pache, **M**ySQL (**M**ariaDB), y **P**HP en dicha instancia.

1. Vaya a la carpeta del repositorio que ha clonado.
   ```bash
    cd LAMP-terraform-ibm
   ```
   {: pre}
2. Inicialice la configuración de terraform. Esto también instalará el plugin `terraform-provider-ibm_VERSION`.
   ````bash
    terraform init
   ````
   {: pre}
3. Aplique la configuración de terraform. Esto creará los recursos definidos en la configuración.
   ```
    terraform apply
   ```
   {: pre}
   Debería ver una salida parecida a la siguiente. ![URL de control de origen](images/solution10/created.png)
4. A continuación, examine la [lista de dispositivos de la infraestructura](https://{DomainName}/classic/devices) para verificar que se ha creado el servidor. ![URL de control de origen](images/solution10/configuration.png)

## Adición del servicio {{site.data.keyword.cos_full_notm}} y escalado de los recursos

{: #modify}

En esta sección aprenderá a escalar los recursos del servidor virtual y a añadir un servicio [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage) al entorno de la infraestructura.

1. Edite el archivo `vm.tf` para aumentar los siguientes valores y guarde el archivo.
 - Aumente el número de núcleos de CPU a 4 núcleos
 - Aumente la RAM (memoria) a 4096
 - Aumente el tamaño de disco a 100 GB

2. A continuación, añada un nuevo servicio [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/infrastructure/cloud-object-storage); para ello, cree un nuevo archivo y llámelo **ibm-cloud-object-storage.tf**. Añada los siguientes fragmentos de código al archivo que acaba de crear. Los fragmentos de código siguientes crean un nombre de variable para el nombre de organización y el nombre de espacio; luego estos dos nombres de variable se utilizan para recuperar el GUID de espacio en el que hay que crear el servicio. Establece el nombre del servicio {{site.data.keyword.cos_full_notm}} en `lamp_objectstorage`; luego necesita un guid de espacio, un nombre completo de servicio y un tipo de plan. El código que aparece a continuación creará un plan premium dado que es un plan de pago según uso. También puede utilizar el plan Lite, pero tenga en cuenta que el plan Lite se limita a un solo servicio por cuenta.

   ```
   variable "org_name" {
     description = "Enter your IBM Cloud org name, you can get your org name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   variable "space_name" {
     description = "Enter your IBM Cloud space name, you can get your space name under your IBM Cloud dashboard account: https://{DomainName}/dashboard"
   }

   data "ibm_space" "space" {
     space = "${var.space_name}"
     org   = "${var.org_name}"
   }

   # un almacenamiento de objetos en la nube
   resource "ibm_service_instance" "objectstorage" {
     name       = "lamp_objectstorage"
     space_guid = "${data.ibm_space.space.id}"
     service    = "cloud-object-storage"

     # solo puede tener un plan Lite por cuenta, de modo que utilizaremos el plan Premium; es de pago según uso
     plan = "Premium"
   }
   ```
   {: pre}
   **Nota:** observe la etiqueta "lamp_objectstorage"; luego la buscaremos en los registros para asegurarnos de que {{site.data.keyword.cos_full_notm}} se ha creado correctamente.

3. Vuelva a inicializar la configuración de terraform con el siguiente mandato:

   ```bash
    terraform init
   ```
   {: pre}

4. Aplique los cambios de terraform con el siguiente mandato:
   ```bash
    terraform apply
   ```
   {: pre}
   **Nota:** después de ejecutar correctamente el mandato terraform apply, debería ver un nuevo archivo `terraform.tfstate` en su directorio. Este archivo contiene la confirmación de despliegue completo para realizar un seguimiento de lo que ha aplicado por última vez y cualquier modificación futura en la configuración. Si este archivo se elimina o se pierde, perderá las configuraciones de despliegue de terraform.

## Verificación de VM y de {{site.data.keyword.cos_short}}
{: #verifyvm}

En esta sección, va a verificar la máquina virtual y {{site.data.keyword.cos_short}} para asegurarse de que se han creado correctamente.

**Verificación de máquina virtual**

1. En el menú de la izquierda, pulse **Infraestructura** para ver la lista de dispositivos de servidor virtual.
2. Pulse **Dispositivos** -> **Lista de dispositivos** para buscar el servidor creado. Debería ver el dispositivo de servidor en la lista.
3. Pulse en el servidor para ver más información sobre la configuración del servidor. En la captura de pantalla siguiente se puede ver que el servidor se ha creado correctamente. ![URL de control de origen](images/solution10/configuration.png)
4. Ahora vamos a probar el servidor en el navegador web. Abra la dirección IP pública del servidor en el navegador web. Debería ver la página de instalación predeterminada del servidor, como la siguiente.![URL de control de origen](images/solution10/LAMP.png)


**Verificación de {{site.data.keyword.cos_full_notm}}**

1. En el **panel de control de {{site.data.keyword.Bluemix_notm}}**, debería ver que se ha creado una instancia del servicio {{site.data.keyword.cos_full_notm}}, lista para ser utilizada.
![object-storage](images/solution10/ibm-cloud-object-storage.png)

   Encontrará más información sobre {{site.data.keyword.cos_full_notm}} [aquí](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage).

## Eliminación de recursos
{: #deleteresources}

Suprima los recursos con el siguiente mandato:
   ```bash
   terraform destroy
   ```
   {: pre}

**Nota:** para suprimir recursos, necesitará permisos de administrador de la infraestructura. Si no tiene una cuenta de superusuario administrador, solicite cancelar los recursos utilizando el panel de control de la infraestructura. Puede solicitar la cancelación de un dispositivo desde el panel de control de la infraestructura, bajo dispositivos. ![object-storage](images/solution10/rm.png)

## Contenido relacionado

- [Terraform](https://www.terraform.io/)
- [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
- [Proveedor de {{site.data.keyword.Bluemix_notm}} para Terraform](https://ibm-cloud.github.io/tf-ibm-docs/)
- [Aceleración de la entrega de archivos estáticos mediante una CDN: {{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn)

