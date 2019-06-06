---
copyright:
  years: 2019
lastupdated: "2019-05-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}
{:important: .important}

# Utilización de una pasarela VPC/VPN para conseguir un acceso seguro y privado local a los recursos de la nube
{: #vpc-site2site-vpn}

IBM aceptará la participación de un número limitado de clientes en un programa de acceso temprano a VPC a partir de principios de abril de 2019 y en los meses siguientes se ampliará su uso. Si su organización desea obtener acceso a IBM Virtual Private Cloud, complete este [formulario de nominación](https://{DomainName}/vpc){: new_window} y un representante de IBM se pondrá en contacto con usted para indicarle los siguientes pasos a seguir.
{: important}

IBM ofrece una serie de maneras de ampliar de forma segura una red de sistemas locales con recursos en la nube de IBM. Esto le permite aprovechar la elasticidad de suministrar servidores cuando los necesita y eliminarlos cuando ya no son necesarios. Además, puede conectar de forma fácil y segura sus prestaciones locales a los servicios de {{site.data.keyword.cloud_notm}}.

Esta guía de aprendizaje le muestra cómo conectar una pasarela de red privada virtual (VPN) local a una VPN de nube creada dentro de un VPC (una pasarela VPC/VPN). En primer lugar, creará una nueva {{site.data.keyword.vpc_full}} (VPC) y los recursos asociados, como subredes, listas de control de accesos (ACL) de red, grupos de seguridad e instancia de servidor virtual (VSI). 
La pasarela VPC/VPN establecerá un enlace de sitio a sitio [IPsec](https://en.wikipedia.org/wiki/IPsec) con una pasarela VPN local. Los protocolos IPsec e [Internet Key Exchange](https://en.wikipedia.org/wiki/Internet_Key_Exchange), IKE, constituyen estándares abiertos probados para una comunicación segura.

Para demostrar aún más el acceso seguro y privado, desplegará un microservicio en una VSI para acceder a {{site.data.keyword.cos_short}} (COS), que representa una aplicación de línea empresarial.
El servicio COS tiene un punto final directo que se puede utilizar para la entrada/salida privada sin coste cuando todo el acceso se encuentra dentro de la misma región de {{site.data.keyword.cloud_notm}}. Un sistema local también accederá al microservicio COS. Todo el tráfico fluirá a través de la VPN y, por lo tanto, de forma privada a través de {{site.data.keyword.cloud_notm}}.

Hay muchas soluciones de VPN locales populares para las pasarelas de sitio a sitio. En esta guía de aprendizaje se utiliza la pasarela VPN [strongSwan](https://www.strongswan.org/) para conectar con la pasarela VPC/VPN. Para simular un centro de datos local, instalará la pasarela strongSwan en una VSI en {{site.data.keyword.cloud_notm}}.

{:shortdesc}
En resumen, con una VPC puede

- conectar los sistemas locales a los servicios y las cargas de trabajo que se ejecutan en {{site.data.keyword.cloud_notm}},
- garantizar la conectividad privada y de bajo coste a COS,
- conectar los sistemas basados en la nube a los servicios y a las cargas de trabajo que se ejecutan localmente.

## Objetivos
{: #objectives}

* Acceso a un entorno de nube privada virtual desde un centro de datos local o una nube privada (virtual).
* Acceso seguro a los servicios de la nube mediante puntos finales de servicio privados.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.vpn_full}}](https://{DomainName}/vpc/provision/vpngateway)
- [{{site.data.keyword.cos_full}}](https://{DomainName}/catalog/services/cloud-object-storage)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.
Aunque no hay cargos de red para acceder a COS desde el micro servicio en esta guía de aprendizaje, se incurrirá en los cargos de red estándar para el acceso a la VPC.

## Arquitectura
{: #architecture}

En el siguiente diagrama se muestra la nube privada virtual que contiene un servidor de apps. El servidor de apps aloja un microservicio que interactúa con el servicio {{site.data.keyword.cos_short}}. Una red local (simulada) y el entorno de nube virtual están conectados a través de pasarelas VPN.

![Arquitectura](images/solution46-vpc-vpn/ArchitectureDiagram.png)

1. La infraestructura (VPC, Subredes, Grupos de seguridad con reglas, ACL de red y VSI) se configura utilizando un script proporcionado.
2. El microservicio interactúa con {{site.data.keyword.cos_short}} a través de un punto final directo.
3. Se suministra una pasarela VPC/VPN para exponer el entorno de nube privada virtual a la red local.
4. Se utiliza el software de pasarela IPsec de código abierto Strongswan de forma local para establecer la conexión VPN con el entorno de nube.

## Antes de empezar
{: #prereqs}

- Instale todas las herramientas de línea de mandatos (CLI) necesarias [siguiendo estos pasos](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview). Necesita el plugin de infraestructura de CLI opcional.
- Inicie una sesión en {{site.data.keyword.cloud_notm}} a través de la línea de mandatos. Consulte [Iniciación a la CLI](https://{DomainName}/docs/cli/reference/ibmcloud?topic=cloud-cli-ibmcloud-cli) para obtener más información.
- Compruebe los permisos de usuario. Asegúrese de que la cuenta de usuario tiene permisos suficientes para crear y gestionar recursos de VPC. Para obtener una lista de los permisos necesarios, consulte [Cómo otorgar los permisos necesarios para usuarios de VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources).
- Necesita una clave SSH para conectarse a los servidores virtuales. Si no tiene una clave SSH, consulte las [instrucciones para crear una clave](/docs/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- Instale [**jq**](https://stedolan.github.io/jq/download/). Lo utilizan los scripts proporcionados para procesar la salida JSON.

## Despliegue de un servidor de apps virtual en una nube privada virtual
A continuación descargará los scripts para configurar un entorno VPC de línea base y un código para que un microservicio interactúe con {{site.data.keyword.cos_short}}. Luego suministrará el servicio {{site.data.keyword.cos_short}} y configurará la línea base.

### Obtención del código
{: #setup}
En la guía de aprendizaje se utilizan scripts para desplegar una línea base de recursos de infraestructura antes de crear las pasarelas VPN. Estos scripts y el código del microservicio se encuentran en un repositorio GitHub.

1. Obtenga el código de la aplicación:
   ```sh
   git clone https://github.com/IBM-Cloud/vpc-tutorials
   ```
   {: codeblock}
2. Vaya a los scripts correspondientes a guía de aprendizaje; para ello vaya a **vpc-tutorials** y luego a **vpc-site2site-vpn**:
   ```sh
   cd vpc-tutorials/vpc-site2site-vpn
   ```
   {: codeblock}

### Creación de servicios
En esta sección iniciará una sesión en {{site.data.keyword.cloud_notm}} en la CLI y creará una instancia de {{site.data.keyword.cos_short}}.

1. Verifique que ha seguido los pasos de requisito previo de inicio de sesión
    ```sh
    ibmcloud target
    ```
    {: codeblock}
2. Cree una instancia de [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage) mediante un plan **estándar** o **lite**.
   ```sh
   ibmcloud resource service-instance-create vpns2s-cos cloud-object-storage lite global
   ```
   {: codeblock}
   
   Tenga en cuenta que solo se puede crear una instancia lite por cuenta. Si ya tiene una instancia de {{site.data.keyword.cos_short}}, la puede reutilizar.
   {: tip}

3. Cree una clave de servicio con el rol de lector (**Reader**):
   ```sh
   ibmcloud resource service-key-create vpns2s-cos-key Reader --instance-name vpns2s-cos
   ```
   {: codeblock}
4. Obtenga los detalles de la clave de servicio en formato JSON y almacénelos en un nuevo archivo **credentials.json** en el subdirectorio **vpc-app-cos**. La app utilizará este archivo más adelante.
   ```sh
   ibmcloud resource service-key vpns2s-cos-key --output json > vpc-app-cos/credentials.json
   ```
   {: codeblock}
   

### Creación de los recursos de línea base de una nube privada virtual
{: #create-vpc}
La guía de aprendizaje proporciona un script para crear los recursos de línea base necesarios para esta guía de aprendizaje, es decir, el entorno de inicio. El script puede generar ese entorno en una VPC existente o crear una nueva VPC.

En los pasos siguientes creará estos recursos configurando y ejecutando un script de configuración. El script incorpora la configuración de un host bastión, tal como se describe en el apartado sobre [acceso seguro a instancias remotas con un host bastión](https://{DomainName}/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server).

1. Copie el archivo de configuración de ejemplo en un archivo para utilizarlo:

   ```sh
   cp config.sh.sample config.sh
   ```
   {: codeblock}

2. Edite el archivo **config.sh** y adapte los valores a su entorno. Debe cambiar el valor de **SSHKEYNAME** por el nombre o por la lista de nombres separados por comas de las claves SSH (consulte "Antes de empezar"). Modifique los distintos valores de **ZONE** para adaptarlos a su región de la nube. Las demás variables se pueden dejar tal como están.
3. Para crear los recursos en una nueva VPC, ejecute el script tal como se indica a continuación:

   ```sh
   ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

   Para reutilizar una VPC existente, pase su nombre al script de esta forma. Sustituya **YOUR_EXISTING_VPC** por el nombre de la VPC real.
   ```sh
   REUSE_VPC=YOUR_EXISTING_VPC ./vpc-site2site-vpn-baseline-create.sh
   ```
   {: codeblock}

4. Este mandato creará los siguientes recursos, incluidos los recursos relacionados con el bastión:
   - 1 VPC (opcional)
   - 1 pasarela pública
   - 3 subredes dentro de la VPC
   - 4 grupos de seguridad con reglas de entrada y salida
   - 3 VSI: vpns2s-onprem-vsi (la IP flotante es ONPREM_IP), vpns2s-cloud-vsi (la IP flotante es VSI_CLOUD_IP) y vpns2s-bastion (la IP flotante es BASTION_IP_ADDRESS)

   Anote para utilizarlos más adelante los valores devueltos para **BASTION_IP_ADDRESS**, **VSI_CLOUD_IP**, **ONPREM_IP**, **CLOUD_CIDR** y **ONPREM_CIDR**. La salida también se guarda en el archivo **network_config.sh**. El archivo se puede utilizar para la configuración automatizada.

### Creación de la pasarela y la conexión de la red privada virtual
En los pasos siguientes añadirá una pasarela VPN y una conexión asociada a la subred con la VSI de la aplicación.

1. Vaya a la página [Visión general de VPC](https://{DomainName}/vpc/overview), pulse **VPN** en el separador de navegación y **Nueva pasarela VPN** en el diálogo. En el formulario **Nueva pasarela VPN para VPC**, escriba **vpns2s-gateway** como nombre. Asegúrese de seleccionar los valores correctos de VPC, grupo de recursos y **vpns2s-cloud-subred**.
2. Deje la opción **Nueva conexión VPN para VPC** activada. Escriba **vpns2s-gateway-conn** como nombre.
3. Para la **Dirección de pasarela de igual**, utilice la dirección IP flotante **vpns2s-onprem-vsi** (ONPREM_IP). Escriba **20_PRESHARED_KEY_KEEP_SECRET_19** como **Clave precompartida**.
4. Para **Redes locales**, utilice la información suministrada para **CLOUD_CIDR**, y para **Subredes iguales**, la suministrada para **ONPREM_CIDR**.
5. Deje los valores de **Detección de iguales inactivos** tal como está. Pulse **Crear pasarela VPN** para crear la pasarela y una conexión asociada.
6. Espere a que la pasarela VPN esté disponible (es posible que tenga que renovar la pantalla).
7. Anote la dirección de **IP de pasarela** asignada como **GW_CLOUD_IP**. 

### Creación de la pasarela de red privada virtual local
A continuación, creará la pasarela VPN en el otro sitio, en el entorno local simulado. Utilizará el software IPsec basado en código abierto [strongSwan](https://strongswan.org/).

1. Conéctese a la VSI "local" **vpns2s-onprem-vsi** mediante ssh. Ejecute el siguiente mandato y sustituya **ONPREM_IP** por la dirección IP devuelta anteriormente.

   ```sh
   ssh root@ONPREM_IP
   ```
   {:pre}

   En función del entorno, es posible que tenga que utilizar `ssh -i <path to your private key file> root@ONPREMP_IP`.
   {:tip}

2. A continuación, en la máquina **vpns2s-onprem-vsi**, ejecute los mandatos siguientes para actualizar el gestor de paquetes y para instalar el software strongSwan.

   ```sh
   apt-get update
   ```
   {:pre}
   
   ```sh
   apt-get install strongswan
   ```
   {:pre}

3. Configure el archivo **/etc/sysctl.conf** añadiendo tres líneas al final. Copie lo siguiente y ejecútelo:

   ```sh
   cat >> /etc/sysctl.conf << EOF
   net.ipv4.ip_forward = 1
   net.ipv4.conf.all.accept_redirects = 0
   net.ipv4.conf.all.send_redirects = 0
   EOF
   ```
   {:codeblock}

4. A continuación, edite el archivo **/etc/ipsec.secrets**. Añada la línea siguiente para configurar las direcciones IP de origen y de destino y la clave precompartida configurada anteriormente. Sustituya **ONPREM_IP** por el valor conocido de la ip flotante de vpns2s-onprem-vsi.  Sustituya **GW_CLOUD_IP** por la dirección ip conocida de la pasarela VPN de VPC.

   ```
   ONPREM_IP GW_CLOUD_IP : PSK "20_PRESHARED_KEY_KEEP_SECRET_19"
   ```
   {:pre}

5. El último archivo que tiene que configurar es **/etc/ipsec.conf**. Añada el siguiente bloque de código al final de dicho archivo. Sustituya **ONPREM_IP**, **ONPREM_CIDR**, **GW_CLOUD_IP** y **CLOUD_CIDR** por sus respectivos valores conocidos.

   ```sh
   # configuración básica
   config setup
      charondebug="all"
      uniqueids=yes
      strictcrlpolicy=no
   
   # conexión a centro de datos vpc/vpn
   # izquierda=onprem / derecha=vpc
   conn tutorial-site2site-onprem-to-cloud
      authby=secret
      left=%defaultroute
      leftid=ONPREM_IP
      leftsubnet=ONPREM_CIDR
      right=GW_CLOUD_IP
      rightsubnet=CLOUD_CIDR
      ike=aes256-sha2_256-modp1024!
      esp=aes256-sha2_256!
      keyingtries=0
      ikelifetime=1h
      lifetime=8h
      dpddelay=30
      dpdtimeout=120
      dpdaction=restart
      auto=start
   ```
   {:pre}

6. Reinicie la pasarela VPN y luego compruebe su estado ejecutando: ipsec restart

   ```sh
   ipsec restart
   ```
   {:pre}
   ```sh
   ipsec status
   ```
   {:pre}

   Debería indicar que se ha establecido una conexión. Mantenga abierto el terminal y la conexión ssh con esta máquina.

## Prueba de la conectividad
Puede probar la conexión VPN de sitio a sitio utilizando SSH o desplegando el microservicio que interactúa con {{site.data.keyword.cos_short}}.

### Prueba mediante ssh
Para probar que la conexión VPN se ha establecido correctamente, utilice el entorno local simulado como proxy para iniciar la sesión en el servidor de aplicaciones basado en la nube. 

1. En un nuevo terminal, ejecute el mandato siguiente después de sustituir los valores. Utiliza el host strongSwan como host de salto para conectarse a través de la VPN a la dirección IP privada del servidor de aplicaciones.

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
  
2. Una vez que se haya conectado correctamente, cierre la conexión ssh.

3. En el terminal VSI "local", detenga la pasarela VPN:
   ```sh
   ipsec stop
   ```
   {:pre}
4. En la ventana de mandatos del paso 1), intente establecer de nuevo la conexión:

   ```sh
   ssh -J root@ONPREM_IP root@VSI_CLOUD_IP
   ```
   {:pre}
   El mandato no se debería ejecutar correctamente porque la conexión VPN no está activa, por lo que no hay ningún enlace directo entre el entorno local simulado y el entorno de nube.

   Tenga en cuenta que, en función de los detalles de despliegue, esta conexión realmente se establece correctamente. El motivo es que la conectividad intra-VPC recibe soporte entre zonas. Si desplegar la VSI local simulada en otra VPC o en [{{site.data.keyword.virtualmachinesfull}}](/docs/vsi?topic=virtual-servers-about-public-virtual-servers), se necesitaría la VPN para acceder correctamente.
   {:tip}
   
5. En el terminal VSI "local", inicie de nuevo la pasarela VPN:
   ```sh
   ipsec start
   ```
   {:pre}
 

### Prueba mediante un microservicio
Puede probar la conexión VPN funcional accediendo a un microservicio en la VSI de nube desde la VSI local.

1. Copie el código de la app del microservicio de la máquina local en la VSI de la nube. El mandato utiliza el bastión como host de salto a la VSI de la nube. Sustituya **BASTION_IP_ADDRESS** y **VSI_CLOUD_IP** en consecuencia.
   ```sh
   scp -r  -o "ProxyJump root@BASTION_IP_ADDRESS"  vpc-app-cos root@VSI_CLOUD_IP:vpc-app-cos
   ```
   {:pre}
2. Conéctese a la VSI de nube utilizando de nuevo el bastión como host de salto.
   ```sh
   ssh -J root@BASTION_IP_ADDRESS root@VSI_CLOUD_IP
   ```
   {:pre}
3. En la VSI de la nube, vaya al directorio del código:
   ```sh
   cd vpc-app-cos
   ```
   {:pre}
4. Instale Python y el PIP del gestor de paquetes de Python.
   ```sh
   apt-get update; apt-get install python python-pip
   ```
   {:pre}
5. Instale los paquetes de Python necesarios utilizando **pip**.
   ```sh
   pip install -r requirements.txt
   ```
   {:pre}
6. Inicie la app:
   ```sh
   python browseCOS.py
   ```
   {:pre}
7. En el terminal VSI "local", acceda al servicio. Sustituya VSI_CLOUD_IP en consecuencia.
   ```sh
   curl VSI_CLOUD_IP/api/bucketlist
   ```
   {:pre}
   El mandato debería devolver un objeto JSON.

## Eliminación de recursos
{: #remove-resources}

1. En la consola de gestión de VPC, pulse **VPN**. En el menú de acciones de la pasarela VPN, seleccione **Suprimir** para eliminar la pasarela.
2. A continuación, pulse **IP flotantes** en la navegación y luego en la dirección IP de las VSI. En el menú de navegación, seleccione **Liberar**. Confirme que desea liberar la dirección IP.
3. A continuación, cambie a **Instancias de servidor virtual** y **suprima** las instancias. Las instancias se suprimirán y su estado será **Suprimiendo** durante un tiempo. Asegúrese de renovar el navegador de vez en cuando.
4. Una vez eliminadas las VSI, vaya a **Subredes**. Si la subred tiene una pasarela pública conectada, pulse en el nombre de la subred. En los detalles de la subred, desconecte la pasarela pública. Las subredes sin una pasarela pública se pueden suprimir desde la página de visión general. Suprima las subredes.
5. Después de que se hayan suprimido las subredes, vaya a **VPC** y suprima la VPC.

Si utiliza la consola, es posible que tenga que renovar el navegador para ver información de estado actualizada después de suprimir un recurso.
{:tip}

## Ampliación de la guía de aprendizaje 
{: #expand-tutorial}

¿Desea añadir o ampliar esta guía de aprendizaje? Estas son algunas de las ideas:

- Añada un [equilibrador de carga](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc) para distribuir el tráfico de microservicios de entrada entre varias instancias.
- Despliegue la [aplicación en un servidor público, los datos y los servicios en un host privado](/docs/tutorials?topic=solution-tutorials-vpc-public-app-private-backend).


## Contenido relacionado
{: #related}

- [Glosario de VPC](/docs/vpc?topic=vpc-vpc-glossary)
- [Consulta del plugin de la CLI de IBM Cloud para VPC](/docs/infrastructure-service-cli-plugin?topic=infrastructure-service-cli-vpc-reference)
- [VPC mediante las API REST](/docs/infrastructure/vpc/example-code.html)
- Guía de aprendizaje de la solución: [Acceso seguro a instancias remotas con un host bastión](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server)
