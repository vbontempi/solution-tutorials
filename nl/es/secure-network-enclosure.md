---
copyright:
  years: 2018, 2019
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

# Aislamiento de cargas de trabajo con una red privada segura
{: #secure-network-enclosure}

La necesidad de disponer de entornos de red privados aislados y seguros es fundamental para el modelo de despliegue de aplicaciones IaaS en la nube pública. Los cortafuegos, las VLAN, el direccionamiento y las VPN constituyen componentes necesarios para crear entornos privados aislados. Este aislamiento permite desplegar máquinas virtuales y servidores nativos de forma segura en topologías de aplicaciones complejas de varios niveles al tiempo que ofrece protección frente a los riesgos de Internet pública.  

En esta guía de aprendizaje se describe cómo se puede configurar un [Dispositivo de direccionador virtual](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-faqs-for-ibm-virtual-router-appliance#what-is-vra-) (VRA) en {{site.data.keyword.Bluemix_notm}} para crear una red privada segura (alojamiento). El dispositivo de pasarela de VRA proporciona un solo paquete autogestionado, un cortafuegos, una pasarela VPN, una conversor de direcciones de red (NAT) y un direccionamiento de nivel empresarial. En esta guía de aprendizaje, se utiliza un VRA para mostrar cómo se puede crear un entorno de red aislado en {{site.data.keyword.Bluemix_notm}}. Dentro de esta aplicación, se pueden crear topologías de aplicación, utilizando tecnologías con las que esté familiarizado y bien conocidas de direccionamiento IP, VLAN, subredes IP, reglas de cortafuegos y servidores virtuales y nativos.  

{:shortdesc}

Esta guía de aprendizaje constituye un punto de partida para la red clásica en {{site.data.keyword.Bluemix_notm}} y no debe considerarse una funcionalidad de producción tal como está. Las prestaciones adicionales que se pueden tener en cuenta son:
* [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-get-started-with-ibm-cloud-direct-link#get-started-with-ibm-cloud-direct-link)
* [Dispositivos de cortafuegos de hardware](https://{DomainName}/docs/infrastructure/fortigate-10g?topic=fortigate-10g-exploring-firewalls#exploring-firewalls)
* [VPN IPSec](https://{DomainName}/catalog/infrastructure/ipsec-vpn) para establecer una conectividad segura con el centro de datos.
* Alta disponibilidad con VRA en clúster y enlaces ascendentes duales.
* Registro y auditoría de sucesos de seguridad.

## Objetivos 
{: #objectives}

* Despliegue un dispositivo de direccionador virtual (VRA)
* Definición de las VLAN y las subredes IP para desplegar máquinas virtuales y servidores nativos
* Protección del VRA y del alojamiento con reglas de cortafuegos

## Servicios utilizados
{: #products}

En esta guía de aprendizaje se utilizan los siguientes servicios de {{site.data.keyword.Bluemix_notm}}:
* [Dispositivo de direccionador virtual](https://{DomainName}/catalog/infrastructure/virtual-router-appliance)

Esta guía de aprendizaje puede incurrir en costes. El VRA solo está disponible en los planes de precios mensuales.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

  ![Arquitectura](images/solution33-secure-network-enclosure/Secure-priv-enc.png)
</p>

1. Configurar VPN
2. Desplegar VRA 
3. Crear un servidor virtual
4. Dirigir el acceso mediante el VRA
5. Configurar el cortafuegos del alojamiento
6. Definir una zona APP
7. Definir una zona INSIDE

## Antes de empezar
{: #prereqs}

### Configuración del acceso VPN

En esta guía de aprendizaje, el alojamiento de red que se crea no resulta visible en Internet pública. Solo se podrá acceder al VRA y a los servidores cualquier servidor sólo serán a través de la red privada, y utilizará su VPN para la conectividad. 

1. [Asegúrese de que el acceso VPN esté habilitado](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Debe ser un **Usuario maestro** para poder habilitar el acceso VPN; si no es así, póngase en contacto con el usuario maestro para obtener acceso.
     {:tip}
2. Obtenga las credenciales de acceso de VPN seleccionando su usuario en la [Lista de usuarios](https://{DomainName}/iam#/users).
3. Inicie una sesión en la VPN mediante [la interfaz web](https://www.softlayer.com/VPN-Access) o utilice un cliente VPN para [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) o [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Para el cliente VPN, utilice el FQDN de punto de acceso VPN de un solo centro de datos de la [página de acceso web de VPN](https://www.softlayer.com/VPN-Access), con el formato *vpn.xxxnn.softlayer.com* como dirección de pasarela.
   {:tip}

### Comprobación de los permisos de la cuenta

Póngase en contacto con el usuario maestro de la infraestructura para obtener los permisos siguientes:
- **Permisos rápidos**: usuario básico
- **Red** para que pueda crear y configurar el alojamiento; se necesitan todos los permisos de red. 
- **Servicios** para gestionar las claves SSH

### Carga de las claves SSH

En el portal, [cargue la clave pública SSH](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-getting-started-tutorial#getting-started-tutorial) que se utilizará para acceder y administrar el VRA y la red privada.  

### Centro de datos de destino

Elija un centro de datos {{site.data.keyword.Bluemix_notm}} para desplegar la red privada segura. 

### Solicitud de las VLAN

Para crear el alojamiento privado en el centro de datos de destino, primero se deben asignar las VLAN privadas necesarias para los servidores. No se aplica ningún cargo a la primera VLAN privada y a la primera pública. Las VLAN adicionales para dar soporte a una topología de aplicaciones multinivel sí están sujetas a cargos. 

Para asegurarse de que hay suficientes VLAN disponibles en el mismo direccionador del centro de datos y que se pueden asociar mediante el VRA, se recomienda que se soliciten. Consulte [Solicitud de VLAN](https://{DomainName}/docs/infrastructure/vlans?topic=vlans-ordering-premium-vlans#order-vlans).

## Suministro del dispositivo de direccionador virtual
{: #VRA}

El primer paso consiste en desplegar un VRA que proporcione direccionamiento de IP y el cortafuegos para el alojamiento de red privada. Se puede acceder a Internet desde el alojamiento mediante una VLAN de tránsito con interfaz pública proporcionada por {{site.data.keyword.Bluemix_notm}}, una pasarela y, opcionalmente, un cortafuegos de hardware que establezca la conectividad entre la VLAN pública y las VLAN del alojamiento privado. En esta guía de aprendizaje de la solución, un dispositivo de direccionador virtual (VRA) proporciona esta pasarela y el perímetro del cortafuegos. 

1. En el catálogo, seleccione un [Dispositivo de pasarela](https://{DomainName}/gen1/infrastructure/provision/gateway)
3. En la sección **Proveedor de pasarela**, seleccione AT&T. Puede elegir una velocidad de enlace de "hasta 20 Gbps" o de "hasta 2 Gbps".
4. En la sección **Nombre de host**, especifique un nombre de host y un dominio para el nuevo VRA.
5. Si marca el recuadro de selección **Alta disponibilidad**, obtendrá dos dispositivos VRA que trabajarán en una configuración de tipo activo/reserva utilizando VRRP.
6. En la sección **Ubicación**, seleccione la ubicación y el **Pod** donde residirá el VRA.
7. Seleccione Procesador único o Procesador dual. Verá una lista de servidores. Para elegir un servidor, pulse su botón de selección. 
8. Seleccione la cantidad de **RAM**. En el caso de un entorno de producción, se recomienda utilizar un mínimo de 64 GB de RAM. El mínimo de 8 GB es para entornos de prueba.
9. Seleccione una **Clave SSH** (opcional). Esta clave ssh se instalará en el VRA, por lo que se puede utilizar vyatta se puede utilizar para acceder al VRA con esta clave.
10. Unidad de disco duro. Conserve el valor predeterminado.
11. En la sección **Velocidad de puerto de enlace ascendente**, seleccione la combinación de velocidad, redundancia e interfaces privadas y/o públicas que se adapten a sus necesidades.
12. En la sección **Complementos**, mantenga el valor predeterminado. Si desea utilizar IPv6 en la interfaz pública, seleccione la dirección IPv6.

En el lado derecho, verá el **Resumen del pedido**. Marque el recuadro de selección _He leído y acepto los acuerdos de servicio de terceros mostrados a continuación:_ y pulse el botón **Crear**. Se desplegará la pasarela.

La [Lista de dispositivos](https://{DomainName}/classic/devices) mostrará el VRA casi de inmediato con un símbolo de **Reloj**, que indica que hay transacciones en curso en este dispositivo. Hasta que finalice la creación del VRA, el símbolo de **reloj** permanecerá y, aparte de ver detalles, no podrá realizar ninguna acción de configuración en el dispositivo. 
{:tip}

### Revisión del VRA desplegado

1. Examine el nuevo VRA. En el [Panel de control de la infraestructura](https://{DomainName}/classic), seleccione **Red** en la parte izquierda y **Dispositivos de pasarela** para ir a la página [Dispositivos de pasarela](https://{DomainName}/classic/network/gatewayappliances). Seleccione el nombre del VRA que acaba de crear en la columna **Pasarela** para ir a la página de detalles de la pasarela. ![](images/solution33-secure-network-enclosure/Gateway-detail.png)

2. Anote las direcciones IP `Privada` y `Pública` del VRA para utilizarlas en el futuro.

## Configuración inicial del VRA
{: #initial_VRA_setup}

1. En la estación de trabajo, mediante la VPN SSL, inicie sesión en el VRA utilizando la cuenta de **vyatta** predeterminada y acepte las solicitudes de seguridad SSH. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   Si SSH solicita una contraseña, significa que la clave SSH no se ha incluido en la compilación. Acceda al VRA mediante el [navegador web](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#accessing-the-device-using-the-web-gui) utilizando la `Dirección IP privada de VRA`. La contraseña viene de la página [Contraseñas de software](https://{DomainName}/classic/devices/passwords). En el separador **Configuración**, seleccione la ramificación Sistema/login/vyatta y añada la clave SSH deseada. 
   {:tip}

   Para configurar el VRA, este se debe colocar en modalidad \[edit\] con el mandato `configure`. Cuando se está en modalidad `edit`, el indicador pasa de `$` a `#`. Después de un cambio de configuración correcto del VRA, puede ver los cambios con el mandato `compare` y puede comprobar los cambios con el mandato `validate`. Cuando se confirma un cambio con el mandato `commit`, se aplica a la configuración en ejecución y se guarda automáticamente en la configuración de arranque.


   {:tip}
2. Mejore la seguridad permitiendo solo inicios de sesión SSH. Ahora que el inicio de sesión SSH se ha realizado correctamente mediante la red privada, inhabilite el acceso mediante autenticación con id de usuario y contraseña. 
   ```
   configure
   set service ssh disable-password-authentication
   commit
   exit
   ```
   {: codeblock}
   A partir de este punto, en esta guía de aprendizaje se presupone que todos los mandatos VRA se especifican en el indicador de mandatos `edit`, después de entrar en `configure`.
3. Revise la configuración inicial
   ```
   show
   ```
   {: codeblock}

   El VRA está preconfigurado para el entorno IaaS de {{site.data.keyword.Bluemix_notm}}. Esto incluye lo siguiente:
   - Servidor NTP
   - Servidores de nombres
   - SSH
   - Servidor web HTTPS 
   - Huso horario predeterminado US/Chicago
4. Establezca el huso horario local que necesite. Si opta por completar automáticamente con la tecla de tabulación, se listarán los valores potenciales de huso horario
   ```
   set system time-zone <timezone>
   ```
   {: codeblock}
5. Defina el comportamiento de ping. Ping no está inhabilitado para ayudar en la resolución de problemas de direccionamiento y de cortafuegos. 
   ```
   set security firewall all-ping enable
   set security firewall broadcast-ping disable
   ```
   {: codeblock}
6. Habilite el funcionamiento del cortafuegos con estado. De forma predeterminada, el cortafuegos VRA es sin estado. 
   ```
   set security firewall global-state-policy icmp
   set security firewall global-state-policy udp
   set security firewall global-state-policy tcp
   ```
   {: codeblock}
7. Confirme y guarde automáticamente los cambios en la configuración de arranque. 
   ```
   commit
   ```
   {: codeblock}

## Solicitud del primer servidor virtual
{: #order_virtualserver}

En este punto se crea un servidor virtual para ayudar en el diagnóstico de los errores de configuración de VRA. El acceso correcto a la VSI se valida a través de la red privada de {{site.data.keyword.Bluemix_notm}} antes de que el acceso a la misma se direccione a través del VRA en un paso posterior. 

1. Solicite un [servidor virtual](https://{DomainName}/catalog/infrastructure/virtual-server-group)  
2. Seleccione **Servidor virtual público** y continúe.
3. En la página del pedido:
   - Establezca **Facturación** en **Por hora**.
   - Establezca el *Nombre de host de VSI* y el *Nombre de dominio*. Este nombre de dominio no se utiliza para el direccionamiento ni para DNS, pero debe ajustarse a los estándares de denominación de la red. 
   - Establezca **Ubicación** en la misma que el VRA.
   - Establezca **Perfil** en **C1.1x1**
   - Añada la **Clave SSH** que ha especificado anteriormente.
   - Establezca **Sistema operativo** en **CentOS 7.x - Mínimo**
   - En **Velocidades de puerto de enlace ascendente**, la interfaz de red se debe cambiar del valor predeterminado, *pública y privada*, a solo un **Enlace ascendente de red privada**. Esto garantiza que el nuevo servidor no tenga acceso directo a Internet, y que el acceso se controle mediante las reglas de direccionamiento y de cortafuegos del VRA.
   - Establezca **VLAN privada** en el ID de VLAN de la VLAN privada que se ha solicitado anteriormente.
4. Pulse el recuadro de selección para aceptar los acuerdos de servicio de 'terceros' y pulse **Crear**.
5. Supervise la finalización en la página [Dispositivos](https://{DomainName}/classic/devices) o por correo electrónico. 
6. Anote la *Dirección IP privada* del VSI para un paso posterior y que bajo la sección **Red** de la página **Detalles de dispositivo** se asigne a la VSI la VLAN correcta. Si no es así, suprima esta VSI y cree una VSI nueva en la VLAN correcta. 
7. Compruebe que se puede acceder correctamente a la VSI a través de la red privada de {{site.data.keyword.Bluemix_notm}} mediante ping y SSH desde la estación de trabajo local a través de la VPN.
   ```bash
   ping <VSI Private IP Address>
   SSH root@<VSI Private IP Address>
   ```
   {: codeblock}

## Direccionamiento del acceso de VLAN a través del VRA
{: #routing_vlan_via_vra}

Las VLAN privadas correspondientes al servidor virtual se habrán asociado mediante el sistema de gestión de {{site.data.keyword.Bluemix_notm}} a este VRA. En esta fase, todavía se puede acceder al VSI mediante direccionamiento IP en la red privada de {{site.data.keyword.Bluemix_notm}}. Ahora direccionará la subred mediante el VRA para crear la red privada segura y validará mediante la confirmación de que ahora la VSI no resulta accesible. 

1. Vaya a los detalles de la pasarela para el VRA en la página [Dispositivos de pasarela](https://{DomainName}/classic/network/gatewayappliances) y localice la sección **VLAN asociadas** en la mitad inferior de la página. Allí se mostrará la VLAN asociada. 
2. Si desea añadir ahora más VLAN, vaya a la sección **Asociar una VLAN**. Se debería habilitar el recuadro desplegable *Seleccionar VLAN* para poder seleccionar otras VLAN suministradas. ![](images/solution33-secure-network-enclosure/Gateway-Associate-VLAN.png)

   Si no aparece ninguna VLAN que se pueda elegir, significa que no hay VLAN disponibles en el mismo direccionador que el VRA. Tendrá que crear una [incidencia de soporte](https://{DomainName}/unifiedsupport/cases/add) para solicitar una VLAN privada en el mismo direccionador que el VRA.
   {:tip}
5. Seleccione la VLAN que desea asociar con el VRA y pulse Guardar. La asociación de VLAN inicial puede tardar un par de minutos en completarse. Una vez completada, la VLAN debería aparecer bajo la cabecera **VLAN asociadas**. 

En esta etapa, la VLAN y la subred asociada no están protegidas o direccionadas a través del VRA y se puede acceder a la VSI a través de la red privada de {{site.data.keyword.Bluemix_notm}}. El estado de la VLAN es *Ignorada*.

4. Seleccione **Acciones** en la columna de la derecha y luego **Direccionar VLAN** para direccionar la VLAN/Subred a través del VRA. Este proceso puede durar unos minutos. Una renovación de pantalla mostrará que se ha *Direccionado*. 
5. Seleccione el [Nombre de VLAN](https://{DomainName}/classic/network/vlans/) para ver los detalles de la VLAN. Se puede ver la VSI suministrada, así como la Subred IP primaria asignada. Anote el ID de la VLAN privada \<nnnn\> (1199 en este ejemplo), ya que lo utilizará en un paso posterior. 
6. Seleccione la [subred](https://{DomainName}/classic/network/subnets) para ver los detalles de la subred IP. Anote la red, las direcciones de pasarela y el CIDR (/26) de la subred, ya que los necesitará para seguir configurando el VRA. Se suministran 64 direcciones IP primarias en la red privada; para localizar la dirección de la pasarela quizás deba seleccionar la página 2 o 3. 
7. Valide que la subred/VLAN se direcciona al VRA y que **NO** se puede acceder a la VSI a través de la red de gestión desde la estación de trabajo mediante `ping`. 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}

Con esto finaliza la configuración del VRA mediante la de consola {{site.data.keyword.Bluemix_notm}}. El trabajo adicional para configurar el alojamiento y el direccionamiento de IP se realiza ahora directamente en el VRA a través de SSH. 

## Configuración del direccionamiento de IP y del alojamiento seguro
{: #vra_setup}

Cuando se confirma la configuración de VRA, la configuración en ejecución se modifica y los cambios se guardan automáticamente en la configuración de arranque.

Si se desea volver a una configuración correcta anterior, de forma predeterminada se pueden ver, comparar y restaurar los últimos 20 puntos de confirmación.  Consulte la [Guía de configuración básica del sistema de Vyatta Network OS](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation) para ver más información sobre cómo confirmar y guardar la configuración.
   ```bash
   show system commit
   rollback n
   compare
   ```
   {: codeblock}

### Configuración del direccionamiento de IP de VRA

Configure la interfaz de red virtual de VRA de modo que direccione a la nueva subred desde la red privada de {{site.data.keyword.Bluemix_notm}}.  

1. Inicie una sesión en el VRA mediante SSH. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}
2. Cree una nueva interfaz virtual con el ID de VLAN privado, la dirección IP de la pasarela de subred y el CIDR grabado en los pasos anteriores. Normalmente, el CIDR será /26. 
   ```
   set interfaces bonding dp0bond0 vif <VLAN ID> address <Subnet Gateway IP>/<CIDR>
   commit
   ```
   {: codeblock}
    
   Es muy importante que se utilice la dirección **`<Subnet Gateway IP>`**. Suele ser una más que la dirección de inicio de la dirección de subred. Si se especifica una dirección de pasarela no válida, se producirá el error `Configuration path: interfaces bonding dp0bond0 vif xxxx address [x.x.x.x] is not valid`. Corrija el mandato y vuelva a especificarla. Puede buscarla en Red > Gestión de IP > Subredes. Pulse la subred que necesita para conocer la dirección de pasarela. La segunda entrada de la lista con la descripción **Gateway** es la dirección IP que debe especificar como <IP de pasarela de subred>/<CIDR>.
   {: tip}

3. Obtenga una lista de la nueva interfaz virtual (vif): 
   ```
   show interfaces
   ```
   {: codeblock}

   Esta es una configuración de interfaz de ejemplo que muestra la vif 1199 y la dirección de pasarela de subred.    ![](images/solution33-secure-network-enclosure/show_interfaces.png)
4. Compruebe que se puede acceder a la VSI a través de la red de gestión desde la estación de trabajo. 
   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
   
   Si no se puede acceder a la VSI, compruebe que la tabla de direccionamiento de IP de VRA está configurada como se esperaba. Suprima y vuelva a crear la ruta si es necesario. Para ejecutar un programa show en modalidad de configuración, puede utilizar el mandato run    
   ```bash
    run show ip route <Subnet Gateway IP>
   ```
   {: codeblock}

Con esto finaliza la configuración del direccionamiento de IP.

### Configuración del alojamiento seguro

El alojamiento de red privada seguro se crea a través de la configuración de zonas y reglas de cortafuegos. Consulte la documentación de VRA sobre [configuración de cortafuegos](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-manage-your-ibm-firewalls#manage-firewalls) antes de continuar. 

Se definen dos zonas:
   - INSIDE: la red privada y la de gestión de IBM
   - APP: la VLAN de usuario y la subred dentro del alojamiento de la red privada		

1. Defina cortafuegos y valores predeterminados.
   ```
   configure
   set security firewall name APP-TO-INSIDE default-action drop
   set security firewall name APP-TO-INSIDE default-log

   set security firewall name INSIDE-TO-APP default-action drop
   set security firewall name INSIDE-TO-APP default-log
   commit
   ```
   {: codeblock}
    
   Si un mandato set se ejecuta dos veces por equivocación, recibirá un mensaje *'Configuration path xxxxxxxx is not valid. Node exists'*. Puede pasarlo por alto. Para cambiar un parámetro incorrecto, primero es necesario suprimir el nodo con 'delete security xxxxx xxxx xxxxx'.
   {:tip}
2. Cree el grupo de recursos de la red privada de {{site.data.keyword.Bluemix_notm}}. Este grupo de direcciones define las redes privadas de {{site.data.keyword.Bluemix_notm}} que pueden acceder al alojamiento y las redes a las que se puede acceder desde el alojamiento. Hay dos conjuntos de direcciones IP que necesitan acceder a y desde el alojamiento: los centros de datos de VPN de SSL y la red de servicio de {{site.data.keyword.Bluemix_notm}} (red de fondo/privada). En [rangos de IP de {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-ibm-cloud-ip-ranges#ibm-cloud-ip-ranges) encontrará la lista completa de rangos de IP que se deben permitir. 
   - Defina la dirección de VPN de SSL de los centros de datos que está utilizando para el acceso VPN. En la sección VPN de SSL de Rangos de IP de {{site.data.keyword.Bluemix_notm}}, seleccione los puntos de acceso de VPN para el centro de datos o el clúster del CD. En este ejemplo se muestran los rangos de direcciones de VPN para los centros de datos de {{site.data.keyword.Bluemix_notm}} de Londres.
     ```
     set resources group address-group ibmprivate address 10.2.220.0/24
     set resources group address-group ibmprivate address 10.200.196.0/24
     set resources group address-group ibmprivate address 10.3.200.0/24
     ```
     {: codeblock}
   - Defina los rangos de direcciones para la red de servicio (en red de fondo/privada) de {{site.data.keyword.Bluemix_notm}} correspondientes a WDC04, DAL01 y al centro de datos de destino. En este ejemplo son WDC04 (dos direcciones), DAL01 y LON06.
     ```
     set resources group address-group ibmprivate address 10.3.160.0/20
     set resources group address-group ibmprivate address 10.201.0.0/20
     set resources group address-group ibmprivate address 10.0.64.0/19
     set resources group address-group ibmprivate address 10.201.64.0/20
     commit
     ```
     {: codeblock}
3. Cree la zona APP para la VLAN de usuario y la subred y la zona INSIDE para la red privada de {{site.data.keyword.Bluemix_notm}}. Asigne los cortafuegos que ha creado anteriormente. La definición de zona utiliza los nombres de interfaz de red VRA para identificar la zona asociada con cada VLAN. El mandato para crear la zona APP requiere que se especifique el ID de VLAN de la VLAN asociada con el VRA anterior. Esto se resalta a continuación como `<VLAN ID>`.
   ```
   set security zone-policy zone INSIDE description "IBM Internal network"
   set security zone-policy zone INSIDE default-action drop
   set security zone-policy zone INSIDE interface dp0bond0
   set security zone-policy zone INSIDE to APP firewall INSIDE-TO-APP 

   set security zone-policy zone APP description "Application network"
   set security zone-policy zone APP default-action drop

   set security zone-policy zone APP interface dp0bond0.<VLAN ID>
   set security zone-policy zone APP to INSIDE firewall APP-TO-INSIDE 
   ```
   {: codeblock}
4. Confirme la configuración y, desde la estación de trabajo, verifique mediante ping que ahora el cortafuegos está denegando el tráfico a través de VRA a VSI: 
   ```
   commit
   ```
   {: codeblock}

   ```bash
   ping <VSI Private IP Address>
   ```
   {: codeblock}
5. Defina las reglas de acceso de cortafuegos para udp, tcp e icmp.
   ```
   set security firewall name INSIDE-TO-APP rule 200 protocol icmp
   set security firewall name INSIDE-TO-APP rule 200 icmp type 8
   set security firewall name INSIDE-TO-APP rule 200 action accept
   set security firewall name INSIDE-TO-APP rule 200 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 100 action accept
   set security firewall name INSIDE-TO-APP rule 100 protocol tcp
   set security firewall name INSIDE-TO-APP rule 100 source address ibmprivate

   set security firewall name INSIDE-TO-APP rule 110 action accept
   set security firewall name INSIDE-TO-APP rule 110 protocol udp
   set security firewall name INSIDE-TO-APP rule 110 source address ibmprivate
   commit

   set security firewall name APP-TO-INSIDE rule 200 protocol icmp
   set security firewall name APP-TO-INSIDE rule 200 icmp type 8
   set security firewall name APP-TO-INSIDE rule 200 action accept
   set security firewall name APP-TO-INSIDE rule 200 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 100 action accept
   set security firewall name APP-TO-INSIDE rule 100 protocol tcp
   set security firewall name APP-TO-INSIDE rule 100 destination address ibmprivate

   set security firewall name APP-TO-INSIDE rule 110 action accept
   set security firewall name APP-TO-INSIDE rule 110 protocol udp
   set security firewall name APP-TO-INSIDE rule 110 destination address ibmprivate
   commit
   ```
   {: codeblock}
6. Valide el acceso de cortafuegos. 
   - Confirme que el cortafuegos INSIDE-TO-APP ahora permite el tráfico ICMP y udp/tcp desde la máquina local 
     ```bash
     ping <VSI Private IP Address>
     SSH root@<VSI Private IP Address>
     ```
     {: codeblock}
   - Confirme que el cortafuegos APP-TO-INSIDE permite el tráfico ICMP y udp/tcp. Inicie una sesión en la VSI mediante SSH y ejecute ping sobre uno de los servidores de nombres de {{site.data.keyword.Bluemix_notm}} en 10.0.80.11 y 10.0.80.12.
     ```bash
     SSH root@<VSI Private IP Address>
     [root@vsi  ~]# ping 10.0.80.11 
     ```
     {: codeblock}
7. Valide el acceso continuado a la interfaz de gestión de VRA mediante SSH desde la estación de trabajo. Si el acceso se mantiene, revise y guarde la configuración. De lo contrario, al rearrancar VRA se volverá a una configuración en funcionamiento. 
   ```bash
   SSH vyatta@<VRA Private IP Address>
   ```
   {: codeblock}

   ```
   show security  
   ```
   {: codeblock}

### Depuración de reglas de cortafuegos

Los registros de cortafuegos se pueden ver desde el indicador de mandatos operativos de VRA. En esta configuración, solo el tráfico descartado para cada zona se registra para ayudar en el diagnóstico de una configuración errónea del cortafuegos.  

1. Revise los registros del cortafuegos para el tráfico denegado. Una revisión periódica de los registros identificará si los servidores de la zona APP están intentando de forma válida o errónea ponerse en contacto con los servicios de la red de IBM. 
   ```
   show log firewall name INSIDE-TO-APP
   show log firewall name APP-TO-INSIDE
   ```
   {: codeblock}
2. Si no se puede establecer contacto con los servicios o con los servidores y no se ve nada en los registros del cortafuegos, compruebe si el tráfico IP ping/ssh esperado está presente en la interfaz de red de VRA desde la red privada de {{site.data.keyword.Bluemix_notm}} o en la interfaz de VRA a la VLAN mediante el `<VLAN ID>` de antes.
   ```bash
   monitor interface bonding dp0bond0 traffic
   monitor interface bonding dp0bond0.<VLAN ID> traffic
   ```

## Protección del VRA
{: #securing_the_vra}

1. Aplique una política de seguridad de VRA. De forma predeterminada, la distribución en zonas de cortafuegos basada en políticas no protege el acceso al propio VRA. Esto se configura a través de Control Plane Policing (CPP). VRA proporciona un conjunto de reglas de CPP básico como plantilla. Debe incorporarlo a su configuración:
   ```bash
   merge /opt/vyatta/etc/cpp.conf 
   ```
   {: codeblock}

Esto crea un nuevo conjunto de reglas de cortafuegos llamado `CPP`; revise las reglas adicionales y confirme en la modalidad additional \[edit\]. 
   ```
   show security firewall name CPP
   commit
   ```
   {: codeblock}
2. Protección del acceso SSH público. Debido a un problema pendiente en este momento con el firmware de Vyatta, no se recomienda utilizar `set service SSH listen-address x.x.x.x` para limitar el acceso de administración de SSH a través de la red pública. Como alternativa, el acceso externo se puede bloquear mediante el cortafuegos de CPP para el rango de direcciones IP públicas utilizadas por la interfaz pública de VRA. La `<VRA Public IP Subnet>` que se utiliza aquí es la misma `<VRA Public IP Address>`, con el último octeto convertido en cero (x.x.x.0). 
   ```
   set security firewall name CPP rule 900 action drop
   set security firewall name CPP rule 900 destination address <VRA Public IP Subnet>/24
   set security firewall name CPP rule 900 protocol tcp
   set security firewall name CPP rule 900 destination port 22
   commit 
   ```
   {: codeblock}
3. Valide el acceso administrativo SSH de VRA a través de la red interna de IBM. Si se pierde el acceso al VRA a través de SSH después de realizar confirmaciones, puede acceder al VRA a través de la consola de KVM disponible en la página Detalles del dispositivo del VRA a través del menú desplegable de acción.

Con esto finaliza la configuración del alojamiento de red privada segura que protege una sola zona del cortafuegos que contiene una VLAN y una subred. Se pueden añadir zonas de cortafuegos adicionales, reglas, servidores virtuales y nativos, VLAN y subredes siguiendo las mismas instrucciones. 

## Eliminación de recursos
{: #removeresources}

En este paso limpiará los recursos para eliminar lo que ha creado anteriormente.

El VRA está en el plan de pago mensual. La cancelación no da derecho a un reembolso. Se recomienda cancelar solo si este VRA no se volverá a necesitar durante el mes siguiente. Si se necesita un clúster de alta disponibilidad VRA dual, este VRA se puede actualizar en la página [Detalles de pasarela](https://{DomainName}/classic/network/gatewayappliances/).
{:tip}  

- Cancele todos los servidores virtuales o servidores locales
- Cancele el VRA
- Cancele las VLAN adicionales por incidencia de soporte. 

## Contenido relacionado
{: #related}

- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Subredes IP estáticas y portables](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-and-ips#about-subnets-and-ips)
- [IBM QRadar Security Intelligence Platform](http://www-01.ibm.com/support/knowledgecenter/SS42VS)
- [Documentación de Vyatta](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
