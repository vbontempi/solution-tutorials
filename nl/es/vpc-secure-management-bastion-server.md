---
copyright:
  years: 2019
lastupdated: "2019-04-02"

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
{:important: .important}

# Acceso seguro a instancias remotas con un host bastión
{: #vpc-secure-management-bastion-server}

IBM aceptará la participación de un número limitado de clientes en un programa de acceso temprano a VPC a partir de principios de abril de 2019 y en los meses siguientes se ampliará su uso. Si su organización desea obtener acceso a IBM Virtual Private Cloud, complete este [formulario de nominación](https://{DomainName}/vpc){: new_window} y un representante de IBM se pondrá en contacto con usted para indicarle los siguientes pasos a seguir.
{: important}

En esta guía de aprendizaje se muestra el despliegue de un host bastión para acceder de forma segura a instancias remotas dentro de una nube privada virtual. Un host bastión es una instancia que se suministra en una subred pública y a la que se puede acceder a través de SSH. Una vez configurado, el host bastión actúa como un servidor de **salto** que permite la conexión segura con instancias suministradas en una subred privada.

Para reducir la exposición de los servidores dentro de la VPC, se creará y utilizará un host bastión. Las tareas administrativas en los servidores individuales se van a realizar utilizando SSH, al que se accede como proxy a través del bastión. El acceso a los servidores y el acceso normal a Internet desde los servidores, por ejemplo para la instalación de software, solo se permitirá con un grupo de seguridad de mantenimiento especial conectado a esos servidores.
{:shortdesc}

## Objetivos
{: #objectives}

* Aprender a configurar un host bastión y grupos de seguridad con reglas
* Gestionar servidores de forma segura con el host bastión

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:  

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)  
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

  ![Arquitectura](images/solution47-vpc-secure-management-bastion-server/ArchitectureDiagram.png)

1. Después de configurar la infraestructura necesaria (subredes, grupos de seguridad con reglas, VSI) en la nube, el administrador (DevOps) se conecta (SSH) al host bastión utilizando la clave SSH privada.
2. El administrador asigna un grupo de seguridad de mantenimiento con las reglas de salida adecuadas.
3. El administrador se conecta (SSH) de forma segura a la dirección IP privada de la instancia a través del host bastión para instalar o actualizar cualquier software necesario, por ejemplo un servidor web
4. El usuario de Internet realiza una solicitud HTTP/HTTPS al servidor web.

## Antes de empezar
{: #prereqs}

- Compruebe los permisos de usuario. Asegúrese de que la cuenta de usuario tiene permisos suficientes para crear y gestionar recursos de VPC. Para obtener una lista de los permisos necesarios, consulte [Cómo otorgar los permisos necesarios para usuarios de VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Necesita una clave SSH para conectarse a los servidores virtuales. Si no tiene una clave SSH, consulte las [instrucciones para crear una clave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- En la guía de aprendizaje se presupone que va a añadir el host bastión a una [nube privada virtual](https://{DomainName}/vpc/network/vpcs) existente. **Si no dispone de una nube privada virtual en la cuenta, cree una antes de continuar con los pasos siguientes.**

## Creación de un host bastión
{: #create-bastion-host}

En esta sección, creará y configurará un host bastión junto con un grupo de seguridad en una subred independiente.

### Creación de una subred
{: #create-bastion-subnet}

1. Pulse **Subredes** en **Red** en el panel izquierdo y luego pulse **Nueva subred**.  
   * Especifique **vpc-secure-bastion-subnet** como nombre y seleccione la VPC que ha creado.  
   * Seleccione una ubicación y una zona.  
   * Especifique el rango de IP para la subred en notación CIDR, es decir, **10.xxx.0.0/24**. Deje el **Prefijo de dirección** tal como está y seleccione como **Número de direcciones** el valor 256.
1. Seleccione **Valor predeterminado de VPC** para la lista de control de accesos (ACL) de subred. Puede configurar las reglas de entrada y de salida más adelante.
1. Cambie el valor de **Pasarela pública** por **Conectada**. 
1. Pulse **Crear subred** para suministrarla.

### Creación y configuración del grupo de seguridad de bastión

Vamos a crear un grupo de seguridad y vamos a configurar las reglas de entrada en la VSI de bastión.

1. Vaya a **Grupos de seguridad** y pulse **Nuevo grupo de seguridad**. Especifique **vpc-secure-bastion-sg** como nombre y seleccione la VPC. 
2. Ahora cree las reglas de entrada siguientes pulsando **Añadir regla** en la sección de entrada. Permiten el acceso SSH y Ping (ICMP).
 
	**Regla de entrada:**
	<table>
	   <thead>
	      <tr>
	         <td><strong>Origen</strong></td>
	         <td><strong>Protocolo</strong></td>
	         <td><strong>Valor</strong></td>
	      </tr>
	   <tbody>
	      <tr>
	         <td>Cualquiera - 0.0.0.0/0</td>
	         <td>TCP</td>
	         <td>Entre: <strong>22</strong> Y <strong>22</strong></td>
	      </tr>
         <tr>
            <td>Cualquiera - 0.0.0.0/0</td>
	         <td>ICMP</td>
	         <td>Tipo: <strong>8</strong>,Código: <strong>Dejar vacío</strong></td>
         </tr>
	   </tbody>
	</table>

   Para mejorar aún más la seguridad, se puede restringir el tráfico de entrada a la red de la empresa o a una red doméstica típica. Puede ejecutar `curl ipecho.net/plain ; echo` `para obtener la dirección IP externa de la red y utilizarla en lugar de la otra.
   {:tip }

### Creación de una instancia de bastión
Con la subred y el grupo de seguridad creados, cree la instancia de servidor virtual de bastión.

1. En **Subredes** en el panel izquierdo, seleccione **vpc-secure-bastion-subnet**.
2. Pulse **Instancias conectadas** y suministre una **Nueva instancia** llamada **vpc-secure-vsi** bajo su propia VPC. Seleccione Ubuntu Linux como su imagen y **c-2x4** (2 vCPU y 4 GB de RAM) como su perfil.
3. Seleccione una **Ubicación** y asegúrese de volver a utilizar posteriormente la misma ubicación.
4. Para crear una nueva **clave SSH**, pulse **Nueva clave**
   * Especifique **vpc-ssh-key** como nombre de clave.
   * Deje la **Región** tal como está.
   * Copie el contenido de la clave SSH local existente y péguela bajo **Clave pública**.  
   * Pulse **Añadir clave SSH**.
5. En **Interfaces de red**, pulse el icono **Editar** situado junto a Grupos de seguridad. 
   * Asegúrese de que **vpc-secure-subnet** esté seleccionado como subred.
   * Elimine la marca del grupo de seguridad predeterminado y marque **vpc-secure-bastion-sg**.
   * Pulse **Guardar**.
6. Pulse **Crear instancia de servidor virtual**.
7. Cuando se haya puesto en marcha la instancia, pulse **vpc-secure-bastion-vsi** y **reserve** una IP flotante.

### Prueba del bastión

Cuando la dirección IP flotante del bastión esté activa, intente conectarse a la misma mediante **ssh**:

   ```sh
   ssh -i ~/.ssh/<PRIVATE_KEY> root@<BASTION_FLOATING_IP_ADDRESS>
   ```
   {:pre}

## Configuración de un grupo de seguridad con reglas de acceso de mantenimiento
{: #maintenance-security-group}

Con el acceso al bastión en funcionamiento, continúe y cree el grupo de seguridad para tareas de mantenimiento, como instalar y actualizar el software.

1. Vaya a **Grupos de seguridad** y suministre un nuevo grupo de seguridad llamado **vpc-secure-maintenance-sg** con las siguientes reglas de salida

   <table>
   <thead>
      <tr>
         <td><strong>Destino</strong></td>
         <td><strong>Protocolo</strong></td>
         <td><strong>Valor</strong> </td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Cualquiera - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>Entre: <strong>80</strong> Y <strong>80</strong></td>
      </tr>
      <tr>
         <td>Cualquiera - 0.0.0.0/0</td>
         <td>TCP</td>
         <td>Entre: <strong>443</strong> Y <strong>443</strong></td>
      </tr>
       <tr>
         <td>Cualquiera - 0.0.0.0/0 </td>
         <td>TCP</td>
         <td>Entre: <strong>53</strong> Y <strong>53</strong></td>
      </tr>
      <tr>
         <td>Cualquiera - 0.0.0.0/0</td>
         <td>UDP</td>
         <td>Entre: <strong>53</strong> Y <strong>53</strong></td>
      </tr>
   </tbody>
   </table>

   Las solicitudes del servidor DNS se gestionan en el puerto 53. DNS utiliza TCP para la transferencia de zona y UDP para consultas de nombres de forma regular (primaria) o inversa. Las solicitudes HTTP están en los puertos 80 y 443.
   {:tip }

2. A continuación, añada esta regla de **entrada**, que permite el acceso SSH desde el host bastión.

   <table>
	   <thead>
	      <tr>
	         <td><strong>Origen</strong></td>
	         <td><strong>Protocolo</strong></td>
	         <td><strong>Valor</strong> </td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>Tipo: <strong>Grupo de seguridad</strong> - Nombre: <strong>vpc-secure-bastion-sg</strong></td>
	         <td>TCP</td>
	         <td>Entre: <strong>22</strong> Y <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>

3. Cree el grupo de seguridad.
4. Vaya a **Todos los grupos de seguridad para VPC** y seleccione **vpc-secure-sg**.
5. Por último, edite el grupo de seguridad y añada la siguiente regla de **salida**.

   <table>
	   <thead>
	      <tr>
	         <td><strong>Destino</strong></td>
	         <td><strong>Protocolo</strong></td>
	         <td><strong>Valor</strong> </td>
	      </tr>
	   </thead>
	   <tbody>
	     <tr>
	         <td>Tipo: <strong>Grupo de seguridad</strong> - Nombre: <strong>vpc-secure-maintenance-sg</strong></td>
	         <td>TCP</td>
	         <td>Entre: <strong>22</strong> Y <strong>22</strong></td>
	      </tr>
	   </tbody>
	</table>


## Utilización del host bastión para acceder a otras instancias de la VPC
{: #bastion-host-access-instances}

En esta sección, creará una subred privada con una instancia de servidor virtual y un grupo de seguridad. De forma predeterminada, cualquier subred creada en una VPC es privada.

Si ya tiene instancias de servidor virtual en la VPC a la que desea conectarse, puede saltarse las tres secciones siguientes y empezar a [añadir las instancias de servidor virtual al grupo de seguridad de mantenimiento](#add-vsi-to-maintenance).

### Creación de una subred
{: #create-private-subnet}

Para crear una nueva subred,

1. Pulse **Subredes** en **Red** en el panel izquierdo y luego pulse **Nueva subred**.  
   * Especifique **vpc-secure-private-subnet** como nombre y seleccione la VPC que ha creado.  
   * Seleccione una ubicación.  
   * Especifique el rango de IP para la subred en notación CIDR, es decir, **10.xxx.1.0/24**. Deje el **Prefijo de dirección** tal como está y seleccione como **Número de direcciones** el valor 256.
1. Seleccione **Valor predeterminado de VPC** para la lista de control de accesos (ACL) de subred. Puede configurar las reglas de entrada y de salida más adelante.
1. Cambie el valor de **Pasarela pública** por **Conectada**. 
1. Pulse **Crear subred** para suministrarla.

### Creación de un grupo de seguridad

Para crear un nuevo grupo de seguridad:  
1. Pulse **Grupos de seguridad** en Red y luego pulse **Nuevo grupo de seguridad**.  
2. Especifique **vpc-secure-private-sg** como nombre y seleccione la VPC que ha creado anteriormente.   
3. Pulse **Crear grupo de seguridad**.  

### Creación de una instancia de servidor virtual

Para crear una instancia de servidor virtual en la subred recién creada:

1. Pulse en la subred privada bajo **Subredes**.
2. Pulse **Instancias conectadas** y luego **Nueva instancia**.
3. Especifique un nombre exclusivo, **vpc-secure-private-vsi**, seleccione la VPC que ha creado anteriormente y, a continuación, la misma **Ubicación** que antes.
4. Seleccione la imagen de **Ubuntu Linux**, pulse **Todos los perfiles** y, bajo **Cálculo**, elija **c-2x4** con 2 vCPU y 4 GB de RAM
5. Para **Claves SSH**, seleccione la clave SSH que ha creado anteriormente para el bastión.
6. En **Interfaces de red**, pulse el icono **Editar** situado junto a Grupos de seguridad.   
   * Seleccione **vpc-secure-private-subnet** como la subred.  
   * Elimine la marca del grupo de seguridad predeterminado y active **vpc-secure-private-sg**.  
   * Pulse **Guardar**.  
7. Pulse **Crear instancia de servidor virtual**.  


### Adición de servidores virtuales al grupo de seguridad de mantenimiento
{: #add-vsi-to-maintenance}

Para el trabajo administrativo en los servidores, tiene que asociar los servidores virtuales específicos con el grupo de seguridad de mantenimiento. En las siguientes secciones habilitará el mantenimiento, iniciará una sesión en el servidor privado, actualizará la información del paquete de software y, a continuación, volverá a eliminar la asociación del grupo de seguridad.

Vamos a habilitar el grupo de seguridad de mantenimiento para el servidor.

1. Vaya a **Grupos de seguridad** y seleccione el grupo de seguridad **vpc-secure-maintenance-sg**.  
2. Pulse **Interfaces conectadas** y luego **Editar interfaces**.  
3. Expanda las instancias de servidor virtual y active la selección que hay junto a **primaria** en la columna **Interfaces**.
4. Pulse **Guardar** para que se apliquen los cambios.

### Conexión a la instancia

Para ejecutar SSH en una instancia mediante su **IP privada**, utilizará el host bastión como **host de salto**.

1. Obtenga la dirección IP privada de una instancia de servidor virtual en **Instancias de servidor virtual**.
2. Utilice el mandato ssh con `-J` para iniciar una sesión en el servidor con la dirección **IP flotante** de bastión que ha utilizado anteriormente y la dirección **IP privada** del servidor que se muestra en **Interfaces de red**.

   ```sh
   ssh -J root@<BASTION_FLOATING_IP_ADDRESS> root@<PRIVATE_IP_ADDRESS>
   ```
   {:pre}
   
   El distintivo `-J` recibe soporte en OpenSSH versión 7.3 y posteriores. En las versiones anteriores, `-J` no está disponible. En ese caso, el método más seguro y directo es utilizar la modalidad de reenvío stdio de ssh (`-W`) para hacer que la conexión "rebote" a través de un host bastión. Por ejemplo, `ssh -o ProxyCommand="ssh -W %h:%p root@<BASTION_FLOATING_IP_ADDRESS" root@<PRIVATE_IP_ADDRESS>`
   {:tip }

### Instalación de software y realización de tareas de mantenimiento

Una vez conectado, puede instalar software en el servidor virtual de la subred privada o realizar tareas de mantenimiento.

1. En primer lugar, actualice la información del paquete de software:
   ```sh
   apt-get update
   ```
   {:pre}
2. Instale el software que desee, como por ejemplo Nginx, MySQL o IBM Db2.

Cuando haya terminado, desconéctese del servidor con el mandato `exit`. 

Para permitir solicitudes HTTP/HTTPS procedentes del usuario de Internet, asigne una **IP flotante** a la VSI en la subred privada y abra los puertos necesarios (80 para HTTP y 443 para HTTPS) mediante las reglas de entrada del grupo de seguridad de VSI privada.
{:tip}

### Inhabilitación del grupo de seguridad de mantenimiento

Una vez que haya terminado de instalar software o de realizar el mantenimiento, debe eliminar los servidores virtuales del grupo de seguridad de mantenimiento para mantenerlos aislados.

1. Vaya a **Grupos de seguridad** y seleccione el grupo de seguridad **vpc-secure-maintenance-sg**.  
2. Pulse **Interfaces conectadas** y luego **Editar interfaces**.  
3. Expanda las instancias de servidor virtual y elimine la marca de selección que hay junto a **primaria** en la columna **Interfaces**.
4. Pulse **Guardar** para que se apliquen los cambios.

## Eliminación de recursos
{: #removeresources}

1. Vaya a **Instancias de servidor virtual** y **suprima** las instancias. Las instancias se suprimirán y su estado será **Suprimiendo** durante un tiempo. Asegúrese de renovar el navegador de vez en cuando.
2. Una vez eliminadas las VSI, vaya a **Subredes** y suprima las subredes.
4. Cuando las subredes se hayan suprimido, vaya al separador **Nubes virtuales privadas** y suprima la VPC.

Si utiliza la consola, es posible que tenga que renovar el navegador para ver información de estado actualizada después de suprimir un recurso.
{:tip}

## Contenido relacionado
{: #related}

* [Subredes privadas y públicas en una nube privada virtual](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend#vpc-public-app-private-backend)
