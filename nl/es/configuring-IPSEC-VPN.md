---
copyright:
  years: 2018, 2019
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

# VPN en una red privada segura
{: #configuring-IPSEC-VPN}

La necesidad de crear una conexión privada entre un entorno de red remoto y los servidores de la red privada de {{site.data.keyword.Bluemix_notm}} es un requisito común. Por lo general, esta conectividad da soporte a cargas de trabajo híbridas, transferencias de datos, cargas de trabajo privadas o administración de sistemas en {{site.data.keyword.Bluemix_notm}}. Un túnel de red privada virtual (VPN) de sitio a sitio es el enfoque habitual para asegurar la conectividad entre las redes. 

{{site.data.keyword.Bluemix_notm}} proporciona una serie de opciones para la conectividad del centro de datos de sitio a sitio, ya sea utilizando una VPN a través de Internet pública o mediante una conexión de red privada dedicada. 

Consulte [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
para obtener más información sobre enlaces de red segura dedicada con {{site.data.keyword.Bluemix_notm}}. Una VPN a través de Internet pública proporciona una opción de bajo coste, aunque sin garantías de ancho de banda. 

Hay dos opciones de VPN adecuadas para la conectividad a través de Internet pública a los servidores suministrados en {{site.data.keyword.Bluemix_notm}}:

-	[VPN IPSEC]( https://{DomainName}/catalog/infrastructure/ipsec-vpn)
-	[Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Esta guía de aprendizaje muestra la configuración de una VPN IPSec de sitio a sitio que utiliza un dispositivo de direccionador virtual (VRA) para conectar una subred de un centro de datos del cliente con una subred protegida en la red privada de {{site.data.keyword.Bluemix_notm}}. 

Este ejemplo se basa en la guía de aprendizaje [Aislamiento de cargas de trabajo con una red privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure). Utiliza una VPN IPSec de sitio a sitio, un túnel GRE y direccionamiento estático. Encontrará configuraciones de VPN más complejas que utilizan direccionamiento dinámico (BGP, etc.) y túneles VTI en la [documentación complementaria de VRA](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Objetivos
{: #objectives}

- Documentación de los parámetros de configuración de VPN IPSec
- Configuración de VPN IPSec en un dispositivo de direccionador virtual
- Direccionamiento del tráfico a través de un túnel GRE

## Servicios utilizados
{: #products}

En esta guía de aprendizaje se utilizan los siguientes servicios de {{site.data.keyword.Bluemix_notm}}: 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Esta guía de aprendizaje puede incurrir en costes. El VRA solo está disponible en los planes de precios mensuales.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

  ![Arquitectura](images/solution36-configuring-IPSEC-VPN/sec-priv-vpn.png)
</p>

1.	Documentación de la configuración de VPN
2.	Creación de IPSec VPN en un VRA
3.	Configuración de VPN del centro de datos y del túnel
4.	Creación de un túnel GRE
5.	Creación de una ruta IP estática
6.	Configuración del cortafuegos 

## Antes de empezar
{: #prereqs}

En esta guía de aprendizaje se conecta el alojamiento privado seguro de la guía de aprendizaje sobre [Aislamiento de cargas de trabajo con una red privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) con el centro de datos. Esta guía de aprendizaje debe completarse en primer lugar.

## Documentación de la configuración de VPN
{: #Document_VPN}

La configuración de un enlace de sitio a sitio de VPN IPSec entre el centro de datos y {{site.data.keyword.Bluemix_notm}} requiere coordinación con el equipo de gestión de red en el sitio para determinar muchos de los parámetros de configuración, el tipo de túnel y la información de direccionamiento de IP. Los parámetros tienen que estar perfectamente coordinados para conseguir una conexión VPN operativa. Generalmente, el equipo de gestión de red del sitio especifica una configuración que se ajusta a los estándares corporativos y le proporciona la dirección IP necesaria de la pasarela VPN del centro de datos, así como los rangos de direcciones de subred a los que se puede acceder.

Antes de empezar a configurar la VPN, las direcciones IP de las pasarelas VPN y los rangos de subred de red IP deben determinarse y deben estar disponibles para la configuración de VPN del centro de datos y para el alojamiento de red privada segura en {{site.data.keyword.Bluemix_notm}}. Esto se muestra en la figura siguiente, en la que la zona APP del alojamiento seguro se conectará a través del túnel IPSec a los sistemas de la 'Subred IP de CD' del centro de datos del cliente.   

<p style="text-align: center;">

  ![](images/solution36-configuring-IPSEC-VPN/vpn-addresses.png)
</p>

Los parámetros siguientes se deben acordar y documentar entre el usuario de {{site.data.keyword.Bluemix_notm}} que configura la VPN y el equipo de gestión de red del centro de datos del cliente. En este ejemplo, las direcciones IP de túnel remoto y local se establecen en 192.168.10.1 y 192.168.10.2. Se puede utilizar cualquier subred arbitraria que se acuerde con el equipo de gestión de red del sitio. 

| Elemento  | Descripción |
|:------ |:--- | 
| &lt;ike group name&gt; | Nombre asignado al grupo IKE para la conexión. |
| &lt;ike encryption&gt; | Estándar de cifrado IKE acordado que se utilizará entre {{site.data.keyword.Bluemix_notm}} y el centro de datos del cliente, generalmente *aes256*. |
| &lt;ike hash&gt; | Hash de IKE acordado entre {{site.data.keyword.Bluemix_notm}} y el centro de datos del cliente, generalmente *sha1*. |
| &lt;ike-lifetime&gt; | Tiempo de vida de IKE del centro de datos del cliente, generalmente *3600*. |
| &lt;esp group name&gt; | Nombre asignado al grupo ESP para la conexión. |
| &lt;esp encryption&gt; | Estándar de cifrado de ESP acordado entre {{site.data.keyword.Bluemix_notm}} y el centro de datos del cliente, generalmente *aes256*. |
| &lt;esp hash&gt; | Hash de ESP acordado entre {{site.data.keyword.Bluemix_notm}} y el centro de datos del cliente, generalmente *sha1*. |
| &lt;esp-lifetime&gt; | Tiempo de vida de ESP del centro de datos del cliente, generalmente *1800*. |
| &lt;DC VPN Public IP&gt;  | Dirección IP pública de cara a Internet de la pasarela VPN en el centro de datos del cliente. | 
| &lt;VRA Public IP&gt; | Dirección IP pública del VRA. |
| &lt;Remote tunnel IP\/24&gt; | Dirección IP asignada al extremo remoto del túnel IPSec. Par de direcciones IP del rango que no entra en conflicto con las IP del centro de datos del cliente o de la nube.   |
| &lt;Local tunnel IP\/24&gt; | Dirección IP asignada al extremo local del túnel IPSec.   |
| &lt;DC Subnet/CIDR&gt; | Dirección IP de la subred a la que se va a acceder en el centro de datos del cliente y en CIDR. |
| &lt;App Zone subnet/CIDR&gt; | Dirección IP de red y CIDR de la subred de la zona APP de la guía de aprendizaje de creación de VRA. | 
| &lt;Shared-Secret&gt; | Clave de cifrado compartida que se va a utilizar entre {{site.data.keyword.Bluemix_notm}} y el centro de datos del cliente. |

## Configuración de VPN IPSec en un VRA
{: #Configure_VRA_VPN}

Para crear la VPN en {{site.data.keyword.Bluemix_notm}}, los mandatos y todas las variables que se tienen que cambiar se resaltan a continuación con &lt; &gt;. Los cambios se identifican línea a línea, para cada línea que se debe cambiar. Los valores proceden de la tabla. 

1. Ejecute SSH en VRA y entre en la modalidad `[edit]`.
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2. Cree un grupo de intercambio de claves de Internet (IKE).
   ```
   set security vpn ipsec ike-group <ike group name> proposal 1
   set security vpn ipsec ike-group <ike group name> proposal 1 encryption <ike encryption>
   set security vpn ipsec ike-group <ike group name> proposal 1 hash <ike hash>
   set security vpn ipsec ike-group <ike group name> proposal 1 dh-group 2
   set security vpn ipsec ike-group <ike group name> lifetime <ike-lifetime>
   ```
   {: codeblock}
3. Cree un grupo ESP (Encapsulating Security Payload)
   ```
   set security vpn ipsec esp-group <esp group name> proposal 1 encryption <esp encryption>
   set security vpn ipsec esp-group <esp group name> proposal 1 hash <esp hash>
   set security vpn ipsec esp-group <esp group name> lifetime <esp-lifetime>
   set security vpn ipsec esp-group <esp group name> mode tunnel
   set security vpn ipsec esp-group <esp group name> pfs enable
   ```
   {: codeblock}
4. Defina la conexión entre sitio y sitio
   ```
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication mode pre-shared-secret
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  authentication pre-shared-secret <Shared-Secret>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  connection-type initiate
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  ike-group <ike group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  local-address <VRA Public IP>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  default-esp-group <esp group name>
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1
   set security vpn ipsec site-to-site peer <DC VPN Public IP>  tunnel 1 protocol gre
   commit
   save
   ```
   {: codeblock}

## Configuración de la VPN y del túnel del centro de datos
{: #Configure_DC_VPN}

1. El equipo de gestión de red del centro de datos del cliente configurará una conexión VPN IPSec idéntica con los valores &lt;DC VPN Public IP&gt; y &lt;VRA Public IP&gt; intercambiados, la dirección de túnel local y remota y los parámetros &lt;DC Subnet/CIDR&gt; y &lt;App Zone subnet/CIDR&gt; también intercambiados. Los mandatos de configuración específicos del centro de datos del cliente dependerán del proveedor de la VPN.
1. Confirme que se puede acceder a la dirección IP pública de la pasarela de VPN del centro de datos a través de Internet antes de continuar:
   ```
   ping <DC VPN Public IP>
   ```
   {: codeblock}
2. Cuando la configuración de VPN del centro de datos finalice, el enlace IPSec debería aparecer automáticamente. Compruebe que se ha establecido el enlace y que el estado muestra que hay uno o varios túneles IPsec activos. Verifique con el centro de datos que ambos extremos de la VPN muestran los túneles IPsec activos.
   ```bash
   show vpn ipsec sa
   show vpn ipsec status
   ```
   {: codeblock}
3. Si el enlace no se ha creado, compruebe que las direcciones locales y remotas se han especificado correctamente y que otros parámetros son los esperados utilizando el mandato debug:
   ``` bash
   show vpn debug
   ```
   {: codeblock}

La línea `peer-<DC VPN Public IP>-tunnel-1: ESTABLISHED 5 seconds ago, <VRA Public IP>[500].......` debería aparecer en la salida. Si no es así o si muestra 'CONNECTING', significa que se ha producido un error en la configuración de la VPN.  

## Definición de túnel GRE 
{: #Define_Tunnel}

1. Cree el túnel GRE en modalidad de edición de VRA.
   ```
   set interfaces tunnel tun0 address <Local tunnel IP/24>
   set interfaces tunnel tun0 encapsulation gre
   set interfaces tunnel tun0 mtu 1300
   set interfaces tunnel tun0 local-ip <VRA Public IP>
   set interfaces tunnel tun0 remote-ip <DC VPN Public IP>
   commit
   ```
   {: codeblock}
2. Cuando ambos extremos del túnel se hayan configurado, debería aparecer automáticamente. Compruebe el estado operativo del túnel desde la línea de mandatos de VRA.
   ```
   show interfaces tunnel
   show interfaces tun0
   ```
   {: codeblock}
   El primer mandato debería mostrar el túnel con el estado y el enlace `u/u` (UP/UP). El segundo mandato muestra más detalles sobre el túnel y el tráfico que se transmite y se recibe.
3. Compruebe que el tráfico fluye por el túnel
   ```bash
   ping <Remote tunnel IP>
   ```
   {: codeblock}
   Los recuentos de TX y de RX del mandato `show interfaces tunnel tun0` deberían ir aumentando mientras haya tráfico `ping`. 
4. Si el tráfico no fluye, se pueden utilizar mandatos `monitor interface` para observar el tráfico que se ve en cada interfaz. La interfaz `tun0` muestra el tráfico interno sobre el túnel. La interfaz `dp0bond1` mostrará el flujo de tráfico encapsulado de salida y de entrada de la pasarela VPN remota.
   ```
   monitor interface tunnel tun0 traffic
   monitor interface bonding dp0bond1 traffic 
   ```
   {: codeblock}

Si no se ve ningún tráfico de retorno, el equipo de gestión de red del centro de datos tendrá que supervisar los flujos de tráfico en la VPN y en las interfaces de túnel en el sitio remoto para localizar el problema. 

## Creación de una ruta IP estática
{: #Define_Routing}

Cree el direccionamiento de VRA para dirigir el tráfico a la subred remota a través del túnel.

1. Cree una ruta estática en modalidad de edición de VRA.
   ```
   set protocols static route <DC Subnet/CIDR>  next-hop <Remote tunnel IP>
   ```
   {: codeblock}   
2. Revise la tabla de direccionamiento de VRA desde la línea de mandatos de VRA. En este momento, no habrá tráfico por la ruta porque no existen reglas de cortafuegos para permitan el tráfico a través del túnel. Se necesita reglas de cortafuegos para el tráfico iniciado en cualquiera de los lados.
   ```bash
   show ip route
   ```
   {: codeblock}

## Configuración del cortafuegos
{: #Configure_firewall}

1. Cree grupos de recursos para el tráfico icmp permitido y los puertos tcp. 
   ```
   set res group icmp-group icmpgrp type 8
   set res group icmp-group icmpgrp type 11
   set res group icmp-group icmpgrp type 3

   set res group port tcpports port 22
   set res group port tcpports port 80
   set res group port tcpports port 443
   commit
   ```
   {: codeblock}
2. Cree reglas de cortafuegos para el tráfico destinado a la subred remota en modalidad de edición de VRA.
   ```
   set security firewall name APP-TO-TUNNEL default-action drop
   set security firewall name APP-TO-TUNNEL default-log

   set security firewall name APP-TO-TUNNEL rule 100 action accept
   set security firewall name APP-TO-TUNNEL rule 100 protocol tcp
   set security firewall name APP-TO-TUNNEL rule 100 destination port tcpports

   set security firewall name APP-TO-TUNNEL rule 200 protocol icmp
   set security firewall name APP-TO-TUNNEL rule 200 icmp group icmpgrp
   set security firewall name APP-TO-TUNNEL rule 200 action accept
   commit
   ```
   {: codeblock}

   ```
   set security firewall name TUNNEL-TO-APP default-action drop
   set security firewall name TUNNEL-TO-APP default-log

   set security firewall name TUNNEL-TO-APP rule 100 action accept
   set security firewall name TUNNEL-TO-APP rule 100 protocol tcp
   set security firewall name TUNNEL-TO-APP rule 100 destination port tcpports

   set security firewall name TUNNEL-TO-APP rule 200 protocol icmp
   set security firewall name TUNNEL-TO-APP rule 200 icmp group icmpgrp
   set security firewall name TUNNEL-TO-APP rule 200 action accept
   commit
   ```
   {: codeblock}
2. Cree una zona para el túnel y asocie cortafuegos para el tráfico iniciado en cualquiera de las zonas.
   ```
   set security zone-policy zone TUNNEL description "GRE Tunnel"
   set security zone-policy zone TUNNEL default-action drop
   set security zone-policy zone TUNNEL interface tun0

   set security zone-policy zone TUNNEL to APP firewall TUNNEL-TO-APP
   set security zone-policy zone APP to TUNNEL firewall APP-TO-TUNNEL
   commit
   save
   ```
   {: codeblock}
3. Para comprobar que los cortafuegos y el direccionamiento en ambos extremos se han configurado correctamente y ahora permiten el tráfico ICMP y TCP, ejecute ping sobre la dirección de pasarela de la subred remota, primero desde la línea de mandatos VRA y, si es correcto, iniciando una sesión en la VSI.
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}
4. Si el mandato ping desde la línea de mandatos de VRA falla, compruebe que se ha visto una respuesta de ping como respuesta a una solicitud de ping en la interfaz de túnel.
   ```
   monitor interface tunnel tun0 traffic
   ```
   {: codeblock}
   Si no hay respuesta significa que existe un problema con las reglas de cortafuegos o con el direccionamiento en el centro de datos. Si se ve una respuesta en la salida del supervisor, pero se excede el tiempo de espera del mandato ping, compruebe la configuración de las reglas de los cortafuegos VRA locales.
5. Si el ping desde la VSI falla, significa que hay un problema con las reglas de cortafuegos de VRA, con el direccionamiento en el VRA o con la configuración de la VSI. Complete el paso anterior para asegurarse de que se envía una solicitud al centro de datos y se ve la respuesta procedente del mismo. La supervisión del tráfico en la VLAN local y la inspección de los registros del cortafuegos le ayudarán a aislar el problema al direccionamiento o al cortafuegos.
   ```
   monitor interfaces bonding dp0bond0.<VLAN ID>
   show log firewall name APP-TO-TUNNEL
   show log firewall name TUNNEL-TO-APP
   ```
   {: codeblock}

Esto completa la configuración de la VPN desde el alojamiento de red privada segura. Otras guías de aprendizaje de esta serie muestran cómo el alojamiento puede acceder a los servicios en Internet pública.

## Eliminación de recursos
{:removeresources}

Pasos que se deben seguir para eliminar los recursos que se crean en esta guía de aprendizaje.

El VRA está en el plan de pago mensual. La cancelación no da derecho a un reembolso. Se recomienda cancelar solo si este VRA no se volverá a necesitar durante el mes siguiente. Si se necesita un clúster de alta disponibilidad VRA dual, este VRA se puede actualizar en la página [Detalles de pasarela](https://{DomainName}/classic/network/gatewayappliances).
{:tip}  

1. Cancele todos los servidores virtuales o servidores locales
2. Cancele el VRA
3. Cancele las VLAN adicionales por incidencia de soporte. 

## Contenido relacionado
{:related}
- [IBM Virtual Router Appliance](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-accessing-and-configuring-the-ibm-virtual-router-appliance#vra-basics)
- [Subredes IP estáticas y portables](https://{DomainName}/docs/infrastructure/subnets?topic=subnets-about-subnets-ips#about-subnets-ips)
- [Documentación de Vyatta](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation)
- [Brocade Vyatta Network OS IPsec Site-to-Site VPN Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-ipsec-vpn.pdf)
- [Brocade Vyatta Network OS Tunnels Configuration Guide, 5.2R1](https://public.dhe.ibm.com/cloud/bluemix/network/vra/vyatta-network-os-5.2r1-tunnels.pdf)
