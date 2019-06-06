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

# Subredes privadas y públicas en una nube privada virtual
{: #vpc-public-app-private-backend}

IBM aceptará la participación de un número limitado de clientes en un programa de acceso temprano a VPC a partir de principios de abril de 2019 y en los meses siguientes se ampliará su uso. Si su organización desea obtener acceso a IBM Virtual Private Cloud, complete este [formulario de nominación](https://{DomainName}/vpc){: new_window} y un representante de IBM se pondrá en contacto con usted para indicarle los siguientes pasos a seguir.
{: important}

En esta guía de aprendizaje se muestra cómo crear de su propia {{site.data.keyword.vpc_full}} (VPC) con una subred pública y una subred privada y una instancia de servidor virtual (VSI) en cada subred. Una VPC es su propia nube privada en la infraestructura de nube compartida con aislamiento lógico de otras redes virtuales.

Una [subred](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#subnet) es un rango de direcciones IP. Está enlazado a una sola zona y no puede abarcar varias zonas o regiones. A efectos de VPC, la característica importante de una subred es el hecho de que las subredes se pueden aislar unas de otras, así como interconectar de la forma habitual. El aislamiento de subredes se consigue mediante [listas de control de acceso](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#access-control-list) (ACL) de red que actúan como cortafuegos para controlar el flujo de paquetes de datos entre subredes. Del mismo modo, los [Grupos de seguridad](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#security-group) (SG) actúan como cortafuegos virtuales para controlar el flujo de paquetes de datos procedentes y destinados a VSI individuales.

La subred pública se utiliza para los recursos que deben estar expuestos al mundo exterior. Los recursos con acceso restringido a los que nunca se debe acceder directamente desde el mundo exterior se colocan dentro de la subred privada. Las instancias de este tipo de subred pueden ser la base de datos de fondo o algún almacén secreto que no desea que sea accesible públicamente. Se definen SG para permitir o denegar el tráfico a las VSI.
{:shortdesc}

En resumen, mediante VPC puede

- crear una red definida por software (SDN),
- aislar cargas de trabajo,
- ejercer un control exhaustivo del tráfico de entrada y de salida.

## Objetivos

{: #objectives}

- Comprensión de los objetos de infraestructura disponibles para nubes privadas virtuales
- Creación de una nube privada virtual, subredes e instancias de servidor
- Aplicación de grupos de seguridad para proteger el acceso a los servidores

## Servicios utilizados

{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

![Arquitectura](images/solution40-vpc-public-app-private-backend/Architecture.png)


1. El administrador (DevOps) configura la infraestructura necesaria (VPC, subredes, grupos de seguridad con reglas, VSI) en la nube.
2. El usuario de Internet realiza una solicitud HTTP/HTTPS al servidor web en la interfaz frontal.
3. La interfaz frontal solicita recursos privados del programa de fondo protegido y muestra los resultados al usuario.

## Antes de empezar

{: #prereqs}

- Compruebe los permisos de usuario. Asegúrese de que la cuenta de usuario tiene permisos suficientes para crear y gestionar recursos de VPC. Para obtener una lista de los permisos necesarios, consulte [Cómo otorgar los permisos necesarios para usuarios de VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).

- Necesita una clave SSH para conectarse a los servidores virtuales. Si no tiene una clave SSH, consulte las [instrucciones para crear una clave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).

## Creación de una nube privada virtual
{: #create-vpc}

Para crear su propia {{site.data.keyword.vpc_short}},

1. Vaya a la página [Visión general de VPC](https://{DomainName}/vpc/overview) y pulse **Crear una VPC**.
2. En la sección **Nueva nube privada virtual**:
   * Especifique **vpc-pubpriv** como nombre para la VPC.
   * Seleccione un **Grupo de recursos**.
   * Si lo desea, añada **Etiquetas** para organizar los recursos.
3. Seleccione **Crear nuevo valor predeterminado (Permitir todo)** como la lista de control de accesos (ACL) predeterminada de la VPC.
1. Elimine la marca de SSH y ejecute ping desde el **Grupo de seguridad predeterminado**.
4. En **Nueva subred para VPC**:
   * Como nombre exclusivo especifique **vpc-pubpriv-backend-subnet**.
   * Seleccione una ubicación.
   * Especifique el rango de IP para la subred en notación CIDR, es decir, **10.xxx.0.0/24**. Deje el **Prefijo de dirección** tal como está y seleccione como **Número de direcciones** el valor 256.
5. Seleccione **Utilizar valor predeterminado de VPC** para la lista de control de accesos (ACL) de subred. Puede configurar las reglas de entrada y de salida más adelante.
6. Pulse **Crear nube privada virtual** para suministrar la instancia.

Si las VSI conectadas a la subred privada tienen que acceder a Internet para cargar software, establezca para la pasarela pública el valor **Conectada** ya que conectar una pasarela pública permitirá que todos los recursos conectados se comuniquen con Internet pública. Cuando las VSI tengan todo el software necesario, vuelva a establecer para la pasarela pública el valor **Desconectada** para que la subred no pueda acceder a Internet pública.
{: important}

Para confirmar la creación de la subred, pulse **Subredes** en el panel izquierdo y espere a que el estado sea **Disponible**. Puede crear una nueva subred en **Subredes**.

## Creación de un grupo de seguridad de fondo y una VSI
{: #backend-subnet-vsi}

En esta sección, creará un grupo de seguridad y una instancia de servidor virtual para el programa de fondo.

### Creación de un grupo de seguridad de programa de fondo

De forma predeterminada, se crea un grupo de seguridad junto con la VPC que permite todo el tráfico SSH (TCP puerto 22) y Ping (ICMP de tipo 8) a las instancias conectadas.

Para crear un nuevo grupo de seguridad para el programa de fondo:  
1. Pulse **Grupos de seguridad** en **Red** y luego pulse **Nuevo grupo de seguridad**.  
2. Especifique **vpc-pubpriv-backend-sg** como nombre y seleccione la VPC que ha creado anteriormente.  
3. Pulse **Crear grupo de seguridad**.

Más adelante editará el grupo de seguridad para añadir las reglas de entrada y de salida.

### Creación de una instancia de servidor virtual de programa de fondo

Para crear una instancia de servidor virtual en la subred recién creada:

1. Pulse en la subred de programa de fondo bajo **Subredes**.
2. Pulse **Instancias conectadas** y luego **Nueva instancia**.
3. Especifique un nombre exclusivo y elija **vpc-pubpriv-backend-vsi**. A continuación, seleccione la VPC que ha creado anteriormente y la misma **Ubicación** que antes.
4. Elija la imagen de **Ubuntu Linux**, pulse **Todos los perfiles** y, en **Cálculo**, seleccione **c-2x4** con 2 vCPUs y 4 GB de RAM.
5. Para **Claves SSH**, seleccione la clave SSH que ha creado anteriormente.
6. En **Interfaces de red**, pulse el icono **Editar** situado junto a Grupos de seguridad.
   * Seleccione **vpc-pubpriv-backend-subnet** como la subred.
   * Elimine la marca de selección del grupo de seguridad predeterminado y marque **vpc-pubpriv-backend-sg** como activo.
   * Pulse **Guardar**.
7. Pulse **Crear instancia de servidor virtual**.

## Creación de una subred de programa de fondo, un grupo de seguridad y una VSI
{: #frontend-subnet-vsi}

De forma parecida a como lo ha hecho con el programa de fondo, creará una subred frontal con una instancia de servidor virtual y un grupo de seguridad.

### Creación de una subred para la interfaz frontal

Para crear una nueva subred para la interfaz frontal,

1. Pulse **Subredes** en **Red** en el panel izquierdo > **Nueva subred**.
   * Especifique **vpc-pubpriv-frontend-subnet** como nombre y seleccione la VPC que ha creado.
   * Seleccione una ubicación.
   * Especifique el rango de IP para la subred en notación CIDR, es decir, **10.xxx.1.0/24**. Deje el **Prefijo de dirección** tal como está y seleccione como **Número de direcciones** el valor 256.
1. Seleccione **Valor predeterminado de VPC** para la lista de control de accesos (ACL) de subred. Puede configurar las reglas de entrada y de salida más adelante.
1. Dado que todas las instancias de servidor virtual de la subred tendrán una IP flotante conectada, no es necesaria habilitar una pasarela pública para la subred. Las instancias del servidor virtual tendrán conectividad a Internet a través de su IP flotante.
1. Pulse **Crear subred** para suministrarla.

### Creación de un grupo de seguridad frontal

Para crear un nuevo grupo de seguridad para la interfaz frontal:
1. Pulse **Grupos de seguridad** en Red y luego pulse **Nuevo grupo de seguridad**.
2. Especifique **vpc-pubpriv-frontend-sg** como nombre y seleccione la VPC que ha creado anteriormente.
3. Pulse **Crear grupo de seguridad**.

### Creación de una instancia de servidor virtual frontal

Para crear una instancia de servidor virtual en la subred recién creada:

1. Pulse en la subred frontal bajo **Subredes**.
2. Pulse **Instancias conectadas** y luego **Nueva instancia**.
3. Especifique un nombre exclusivo, **vpc-pubpriv-frontend-vsi**, seleccione la VPC que ha creado anteriormente y, a continuación, la misma **Ubicación** que antes.
4. Seleccione la imagen de **Ubuntu Linux**, pulse **Todos los perfiles** y, bajo **Cálculo**, elija **c-2x4** con 2 vCPU y 4 GB de RAM
5. Para **Claves SSH**, seleccione la clave SSH que ha creado anteriormente.
6. En **Interfaces de red**, pulse el icono **Editar** situado junto a Grupos de seguridad.
   * Seleccione **vpc-pubpriv-frontend-subnet** como la subred.
   * Elimine la marca del grupo de seguridad predeterminado y active **vpc-pubpriv-frontend-sg**.
   * Pulse **Guardar**.
   * Pulse **Crear instancia de servidor virtual**.
7. Espere hasta que el estado de la VSI sea **Activada**. A continuación, seleccione la VSI frontal **vpc-pubpriv-frontend-vsi**, desplácese hasta **Interfaces de red** y pulse **Reservar** en **IP flotante** para asociar una dirección IP pública a la VSI frontal. Guarde la dirección IP asociada en un portapapeles para futuras referencias.

## Configuración de la conectividad entre el servidor frontal y el de fondo
{: #setup-connectivity-frontend-backend}

Con todos los servidores configurados, en esta sección configurará la conectividad para permitir operaciones usuales entre el servidor frontal y el de fondo.

### Configuración del grupo de seguridad frontal

1. Vaya a **Grupos de seguridad** en la sección **Red** y pulse **vpc-pubpriv-frontend-sg**.
2. En primer lugar, añada las siguientes reglas de **entrada** mediante la opción **Añadir regla**. Permiten las solicitudes HTTP y Ping (ICMP) de entrada.

	<table>
   <thead>
      <tr>
         <td><strong>Origen</strong></td>
         <td><strong>Protocolo</strong></td>
         <td><strong>Valor</strong></td>
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
         <td>Cualquiera - 0.0.0.0/0</td>
	      <td>ICMP</td>
	      <td>Tipo: <strong>8</strong>,Código: <strong>Dejar vacío</strong></td>
      </tr>
   </tbody>
   </table>

3. A continuación, añada estas reglas de **salida**.

   <table>
   <thead>
      <tr>
         <td><strong>Destino</strong></td>
         <td><strong>Protocolo</strong></td>
         <td><strong>Valor</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Tipo: <strong>Grupo de seguridad</strong> - Nombre: <strong>vpc-pubpriv-backend-sg</strong></td>
         <td>TCP</td>
         <td>Puerto del servidor de fondo, consulte la sugerencia</td>
      </tr>
   </tbody>
   </table>

Estos son los puertos de los servicios de fondo más comunes. MySQL utiliza el puerto 3306, PostgreSQL el puerto 5432. A Db2 se accede a través del puerto 50000 o 50001. Microsoft SQL Server utiliza de forma predeterminada el puerto 1433. Encontrará una de las muchas [listas con puertos comunes en Wikipedia](https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers).
{:tip }

### Configuración del grupo de seguridad de fondo
Al igual que ha hecho con el servidor frontal, configure el grupo de seguridad para el programa de fondo.

1. Vaya a **Grupos de seguridad** en la sección **Red** y pulse **vpc-pubpriv-backend-sg**.
2. Añada la siguiente regla de **entrada** mediante la opción **Añadir regla**. Permite una conexión con el servicio de fondo.

   <table>
   <thead>
      <tr>
         <td><strong>Origen</strong></td>
         <td><strong>Protocolo</strong></td>
         <td><strong>Valor</strong></td>
      </tr>
   </thead>
   <tbody>
      <tr>
         <td>Tipo: <strong>Grupo de seguridad</strong> - Nombre: <strong>vpc-pubpriv-frontend-sg</strong></td>
         <td>TCP</td>
         <td>Puerto del servidor de fondo</td>
      </tr>
   </tbody>
   </table>


## Instalación de software y realización de tareas de mantenimiento
{: #install-software-maintenance-tasks}

Siga los pasos mostrados en [acceso seguro a instancias remotas con un host bastión](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) para realizar un mantenimiento seguro de los servidores utilizando un host bastión que actúe como servidor `puente` y un grupo de seguridad de mantenimiento.

## Eliminación de recursos
{: #remove-resources}

1. En la consola de gestión de VPC, pulse **IP flotantes**, luego pulse la dirección IP de las VSI y, a continuación, en el menú de acciones, seleccione **Liberar**. Confirme que desea liberar la dirección IP.
2. A continuación, cambie a **Instancias de servidor virtual** y **suprima** las instancias. Las instancias se suprimirán y su estado será **Suprimiendo** durante un tiempo. Asegúrese de renovar el navegador de vez en cuando.
3. Una vez eliminadas las VSI, vaya a **Subredes**. Si la subred tiene una pasarela pública conectada, pulse en el nombre de la subred. En los detalles de la subred, desconecte la pasarela pública. Las subredes sin una pasarela pública se pueden suprimir desde la página de visión general. Suprima las subredes.
4. Después de que se hayan suprimido las subredes, vaya al separador **VPC** y suprima la VPC.

Si utiliza la consola, es posible que tenga que renovar el navegador para ver información de estado actualizada después de suprimir un recurso.
{:tip}

## Ampliación de la guía de aprendizaje
{: #expand-tutorial}

¿Desea añadir o ampliar esta guía de aprendizaje? Estas son algunas de las ideas:

- Añada un [equilibrador de carga](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-load-balancer) para distribuir el tráfico de entrada entre varias instancias.
- Cree una [red privada virtual](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-console#creating-a-vpn) (VPN) para que la VPC se pueda conectar de forma segura con otra red privada, como por ejemplo una red local u otra VPC.


## Contenido relacionado
{: #related}

- [Glosario de VPC](/docs/infrastructure/vpc?topic=vpc-vpc-glossary#vpc-glossary)
- [VPC mediante la CLI de IBM](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-ibm-cloud-cli#creating-a-vpc-using-the-ibm-cloud-cli)
- [VPC mediante las API REST](/docs/infrastructure/vpc?topic=vpc-creating-a-vpc-using-the-rest-apis#creating-a-vpc-using-the-rest-apis)
