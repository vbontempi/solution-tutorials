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

# Enlace de redes privadas seguras a través de la red de IBM
{: #vlan-spanning}

A medida que aumenta la necesidad de un alcance global y de las operaciones de tipo 24-7 de la aplicación web, aumenta la necesidad de alojar servicios en varios centros de datos de la nube. Los centros de datos en varias ubicaciones proporcionan resistencia en el caso de que se produzca una anomalía en la región geográfica, y además acercan las cargas de trabajo a los usuarios distribuidos globalmente, reduciendo así la latencia y aumentando el rendimiento. La [red de {{site.data.keyword.Bluemix_notm}}](https://www.ibm.com/cloud/data-centers/) permite a los usuarios enlazar cargas de trabajo alojadas en redes privadas seguras a través de centros de datos y ubicaciones.

En esta guía de aprendizaje se muestra la configuración de una conexión IP de direccionamiento privado a través de la red privada de {{site.data.keyword.Bluemix_notm}} entre dos redes privadas seguras alojadas en distintos centros de datos. Todos los recursos son propiedad de una cuenta de {{site.data.keyword.Bluemix_notm}}. Se utiliza la guía de aprendizaje sobre [Aislamiento de cargas de trabajo con una red privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) para desplegar dos redes privadas que están enlazadas de forma segura a través de la red privada de {{site.data.keyword.Bluemix_notm}} mediante el servicio [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning). 
{:shortdesc}

## Objetivos
{: #objectives}

- Enlace de redes seguras dentro de una cuenta de {{site.data.keyword.Bluemix_notm}} IaaS
- Configuración de reglas de cortafuegos para el acceso de sitio al sitio 
- Configuración del direccionamiento entre sitios

## Servicios utilizados
{: #products}

En esta guía de aprendizaje se utilizan los siguientes servicios de {{site.data.keyword.Bluemix_notm}}: 
* [Dispositivo de direccionador virtual](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#about)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning)

Esta guía de aprendizaje puede incurrir en costes. El VRA solo está disponible en los planes de precios mensuales.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

  ![Arquitectura](images/solution43-vlan-spanning/vlan-spanning.png)
</p>


1. Despliegue de redes privadas seguras
2. Configuración de VLAN Spanning
3. Configuración del direccionamiento IP entre redes privadas
4. Configuración de reglas de cortafuegos para el acceso a sitios remotos

## Antes de empezar
{: #prereqs}

Esta guía de aprendizaje se basa en la guía de aprendizaje sobre [Aislamiento de cargas de trabajo con una red privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). Antes de comenzar es necesario revisar esta guía de aprendizaje y sus requisitos previos. 

## Configuración de sitios de red privados seguros
{: #private_network}

La guía de aprendizaje [Aislamiento de cargas de trabajo con una red privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) se utiliza dos veces para implementar redes privadas en dos centros de datos diferentes. No hay ninguna restricción en cuanto a la utilización de dos centros de datos, aparte de tener en cuenta el impacto de la latencia sobre el tráfico o sobre las cargas de trabajo que se comunicarán entre los sitios. 

Se puede seguir la guía de aprendizaje [Aislamiento de cargas de trabajo con una red privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) sin cargo alguno para cada centro de datos seleccionado, registrando la siguiente información para pasos posteriores. 

| Elemento  | Centro de datos 1 | Centro de datos 2 |
|:------ |:--- | :--- |
| Centro de datos |  |  |
| Dirección IP pública de VRA | <Dirección IP pública de VRA del CD 1 > | <Dirección IP pública VRA del CD 2> |
| Dirección IP privada de VRA | <Dirección IP privada de VRA del CD 1 > | <Dirección IP privada de VRA del CD 2 > |
| Subred privada y CIDR de VRA |  |  |
| ID de VLAN privada | &lt;ID de VLAN privada del CD 1&gt;  | &lt;ID de VLAN privada del CD 2&gt; |
| Dirección IP privada de VSI | <Dirección IP privada de VSI del CD 1 > | <Dirección IP privada de VSI del CD 2 > |
| Subred y CIDR de zona APP | <Subred/CIDR de APP del CD 1> | <Subred/CIDR de APP del CD 2> |

1. Continúe en la página Detalles de pasarela para cada VRA a través de la página [Dispositivos de pasarela](https://{DomainName}/classic/network/gatewayappliances).  
2. Localice la sección VLAN de pasarela y pulse en la [VLAN]( https://{DomainName}/classic/network/vlans) de la pasarela de la red **Privada** para ver los detalles de la VLAN. El nombre debería contener el id `bcrxxx`, que indica 'backend customer router' (direccionador de cliente de fondo), y tener el formato `nnnxx.bcrxxx.xxxx`.
3. El VRA suministrado se verá en la sección **Dispositivos*. En la sección **Subredes**, anote la dirección IP y el CIDR de la subred privada de VRA (/26). La subred será de tipo primario con 64 IP. Estos detalles se necesitarán posteriormente para la configuración de direccionamiento. 
4. De nuevo en la página Detalles de pasarela, localice la sección **VLAN asociadas** y pulse [VLAN]( https://{DomainName}/classic/network/vlans) en la red **Privada** asociada para crear la red segura y la zona APP. 
5. La VSI suministrada se verá en la sección **Dispositivos*. En la sección **Subredes**, anote la dirección IP y el CIDR de subred de VSI (/26), ya que necesitará esta información para la configuración del direccionamiento. Esta VLAN y la subred se identifican como la zona APP en ambas configuraciones de cortafuegos de VRA y se registran como &lt;Subred/CIDR de zona APP&gt;.


## Configuración de VLAN Spanning 
{: #configure-vlan-spanning}

De forma predeterminada, los servidores (y los VRA) en distintas VLAN y centros de datos no pueden comunicarse entre sí a través de la red privada. En estas guías de aprendizaje, en un solo centro de datos se utilizan VRA para enlazar las VLAN y las subredes con direccionamiento IP clásico y cortafuegos para crear una red privada para la comunicación de servidor entre las VLAN. Aunque se pueden comunicar en el mismo centro de datos, en esta configuración los servidores pertenecientes a la misma cuenta de {{site.data.keyword.Bluemix_notm}} no se pueden comunicar entre centros de datos. 

El servicio [VLAN Spanning]( https://{DomainName}/docs/infrastructure/vlans?topic=vlans-vlan-spanning#vlan-spanning) elimina esta restricción de comunicación entre las VLAN y las subredes que **NO** están asociadas a VRA. Hay que tener en cuenta que incluso cuando la función VLAN Spanning está habilitada, las VLAN asociadas con VRA solo se pueden comunicar a través de su VRA asociado, según determina el cortafuegos VRA y la configuración de direccionamiento. Cuando la función VLAN Spanning está habilitada, los VRA propiedad de una cuenta de {{site.data.keyword.Bluemix_notm}} se conectan a través de la red privada y se pueden comunicar. 

Las VLAN que no están asociadas con las redes privadas seguras creadas por los VRA se 'expanden', lo que permite la interconexión de estas VLAN 'no asociadas' entre los centros de datos. Esto incluye las VLAN de VRA Gateway (tránsito) que pertenecen a la misma cuenta de IBM Cloud en distintos centros de datos. Por lo tanto, permita que los VRA se comuniquen a través de centros de datos cuando la función VLAN Spanning está habilitada. Con la conectividad de VRA a VRA, el cortafuegos VRA y la configuración de direccionamiento permiten conectarse a los servidores dentro de las redes seguras. 

Habilite la función VLAN Spanning:

1. Vaya a la página [VLAN]( https://{DomainName}/classic/network/vlans).
2. Seleccione el separador **Ampliar** en la parte superior de la página
3. Seleccione el botón de opción "On" de VLAN Spanning. El cambio en la red tardará varios minutos en completarse.
4. Confirme que los dos VRA ahora se pueden comunicar:

   Inicie sesión en el VRA del centro de datos 1 ejecute ping en el VRA del centro de datos 2

   ```
   SSH vyatta@<DC1 VRA Private IP Address>
   ping <DC2 VRA Private IP Address>
   ```
   {: codeblock}

   Inicie sesión en el VRA del centro de datos 2 ejecute ping en el VRA del centro de datos 1
   ```
   SSH vyatta@<DC2 VRA Private IP Address>
   ping <DC1 VRA Private IP Address>
   ```
   {: codeblock}

## Configuración del direccionamiento de IP de VRA 
{: #vra_routing}

Cree el direccionamiento de VRA en cada centro de datos para permitir que las VSI de las zonas APP de ambos centros de datos se comuniquen. 

1. Cree una ruta estática en el centro de datos 1 a la subred privada de la zona APP en el centro de datos 2, en modalidad de edición de VRA.
   ```
   ssh vyatta@<DC1 VRA Private IP Address>
   conf
   set protocols static route <DC2 APP zone subnet/CIDR>  next-hop <DC2 VRA Private IP>
   commit
   ```
   {: codeblock}
2. Cree una ruta estática en el centro de datos 2 a la subred privada de la zona APP en el centro de datos 1, en modalidad de edición de VRA.
   ```
   ssh vyatta@<DC2 VRA Private IP Address>
   conf
   set protocols static route <DC1 APP zone subnet/CIDR>  next-hop <DC1 VRA Private IP>
   commit
   ```
   {: codeblock}
2. Revise la tabla de direccionamiento de VRA desde la línea de mandatos de VRA. En este momento, las VSI no pueden comunicarse porque no existen reglas de cortafuegos de zona APP para permitir el tráfico entre las dos subredes de la zona APP. Se necesita reglas de cortafuegos para el tráfico iniciado en cualquiera de los lados.
   ```
   show ip route
   ```
   {: codeblock}

Ahora se verá la nueva ruta para permitir que la zona APP se comunique a través de la red privada de IBM. 

## Configuración del cortafuegos VRA
{: #vra_firewall}

Las reglas de cortafuegos de zona APP existentes solo se configuran para permitir el tráfico de entrada y salida de esta subred a los servicios {{site.data.keyword.Bluemix_notm}} en la red privada de {{site.data.keyword.Bluemix_notm}} y para el acceso a Internet pública a través de NAT. Otras subredes asociadas con las VSI en este VRA, o en otros centros de datos, se bloquean. El paso siguiente consiste en actualizar el grupo de recursos `ibmprivate` asociado a la regla de cortafuegos APP-TO-INSIDE para permitir el acceso explícito a la subred en el otro centro de datos. 

1. En modalidad de mandato de edición del VRA del centro de datos 1, añada <subred de zona APP del CD2>/CIDR al grupo de recursos `ibmprivate`
   ```
   set resources group address-group ibmprivate address <DC2 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
2. En modalidad de mandato de edición del VRA del centro de datos 2, añada <subred de zona APP del CD1>/CIDR al grupo de recursos `ibmprivate`
   ```
   set resources group address-group ibmprivate address <DC1 APP zone subnet/CIDR>
   commit
   save
   ```
   {: codeblock}
3. Verifique que ahora los VSI de ambos centros de datos se pueden comunicar
   ```bash
   ping <Remote Subnet Gateway IP>
   ssh root@<VSI Private IP>
   ping <Remote Subnet Gateway IP>
   ```
   {: codeblock}

   Si las VSI no pueden comunicarse, siga las instrucciones de la guía de aprendizaje sobre [Aislamiento de cargas de trabajo con una red privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) para supervisar el tráfico en las interfaces y revisar los registros del cortafuegos. 

## Eliminación de recursos
{: #removeresources}

Pasos que se deben seguir para eliminar los recursos que se crean en esta guía de aprendizaje. 

El VRA está en el plan de pago mensual. La cancelación no da derecho a un reembolso. Se recomienda cancelar solo si este VRA no se volverá a necesitar durante el mes siguiente. Si se necesita un clúster de alta disponibilidad VRA dual, este VRA se puede actualizar en la página [Detalles de pasarela](https://{DomainName}/classic/network/gatewayappliances).
{:tip}

1. Cancele todos los servidores virtuales o servidores locales
2. Cancele el VRA
3. Cancele las VLAN adicionales por incidencia de soporte. 


## Ampliación de la guía de aprendizaje

Esta guía de aprendizaje se puede utilizar junto con la guía de aprendizaje [VPN en una red privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#vpn-into-a-secure-private-network) para enlazar las dos redes seguras con una red remota de usuario a través de una VPN IPSec. Los enlaces VPN se pueden establecer en ambas redes seguras para aumentar la capacidad de recuperación del acceso a la plataforma {{site.data.keyword.Bluemix_notm}} IaaS. Tenga en cuenta que IBM no permite el direccionamiento del tráfico de usuario entre centros de datos de cliente a través de la red privada de IBM. La configuración de direccionamiento para evitar los bucles de red queda fuera del ámbito de esta guía de aprendizaje. 


## Material relacionado
{:related}

1. Virtual Routing and Forwarding (VRF) es una alternativa al uso de VLAN Spanning para conectar redes entre una cuenta de {{site.data.keyword.Bluemix_notm}}. VRF es obligatorio para todos los clientes que utilicen [{{site.data.keyword.BluDirectLink}}](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview). [Visión general de Virtual Routing and Forwarding (VRF) en IBM Cloud](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-overview-of-virtual-routing-and-forwarding-vrf-on-ibm-cloud#customer-vrf-overview)
2. [La red de IBM Cloud](https://www.ibm.com/cloud/data-centers/)
