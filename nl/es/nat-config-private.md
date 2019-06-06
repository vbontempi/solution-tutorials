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

# Configuración de NAT para el acceso a Internet desde una red privada
{: #nat-config-private}

En el mundo actual de aplicaciones y servicios de TI basados en la web, pocas aplicaciones existen de forma aislada. Los desarrolladores deben tener en cuenta el acceso a servicios en Internet, ya sea código de aplicación de código abierto y actualizaciones o servicios de 'terceros' que proporcionan funcionalidad de aplicación a través de API REST. El uso de máscaras de conversión de direcciones de red (NAT) es un método común para proteger el acceso a servicios alojados en internet desde redes privadas. Con máscaras de NAT, las direcciones IP privadas se convierten en la dirección IP de la interfaz pública de salida en una relación entre muchos y uno, protegiendo la dirección IP privada del acceso público.  

En esta guía de aprendizaje se explica la configuración de la máscara de Network Address Translation (NAT) en un Virtual Router Appliance (VRA) para conectar una subred protegida en la red privada de {{site.data.keyword.Bluemix_notm}}. Se basa en la guía de aprendizaje [Aislamiento de cargas de trabajo con una red privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) e incorpora la configuración Source NAT (SNAT), en la que las direcciones de origen se ocultan y se utilizan reglas de cortafuegos para proteger el tráfico de salida. Encontrará configuraciones de NAT más complejas en la [documentación complementaria de VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).
{:shortdesc}

## Objetivos
{: #objectives}

-	Configurar la conversión de direcciones de red de origen (SNAT) en un dispositivo de direccionador virtual (VRA)
-	Configurar reglas de cortafuegos para el acceso a Internet

## Servicios utilizados
{: #products}

En esta guía de aprendizaje se utilizan los siguientes servicios de {{site.data.keyword.Bluemix_notm}}: 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)

Esta guía de aprendizaje puede incurrir en costes. El VRA solo está disponible en los planes de precios mensuales.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

  ![Arquitectura](images/solution35-nat-config-private/vra-nat.png)
</p>

1.	Documentar los servicios de Internet necesarios.
2.	Configurar NAT.
3.	Crear reglas y una zona de cortafuegos de Internet.

## Antes de empezar
{: #prereqs}

En esta guía de aprendizaje se permite que los hosts del alojamiento de red privada segura creados en la guía de aprendizaje [Aislamiento de cargas de trabajo con una red privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure) accedan a los servicios públicos de internet. Esta guía de aprendizaje debe completarse en primer lugar. 

## Documentación de los servicios de Internet
{: #Document_outbound}

El primer paso consiste en identificar los servicios a los que se accederá en Internet pública y documentar los puertos que deben estar habilitados para el tráfico de entrada y de salida correspondiente de Internet. Esta lista de puertos se necesitará para las reglas de cortafuegos en un paso posterior. 

En este ejemplo solo se habilitan los puertos http y https, ya que estos cubren la mayoría de los requisitos. Los servicios DNS y NTP se proporcionan desde la red privada de {{site.data.keyword.Bluemix_notm}}. Si se requieren estos y otros servicios, como SMTP (Puerto 25) o MySQL (Puerto 3306), se necesitarán reglas de cortafuegos adicionales. Las dos reglas de puerto básicas son:

-	Puerto 80 (http)
-	Puerto 443 (https)

Verifique si el servicio de terceros da soporte a la creación de una lista blanca de direcciones de origen. Si es así, la dirección IP pública del VRA deberá configurar el servicio de terceros para limitar el acceso al servicio. 


## Máscaras de NAT en Internet 
{: #NAT_Masquerade}

Siga estas instrucciones para configurar el acceso a Internet externo para los hosts de la zona APP mediante el uso de máscaras de NAT. 

1.	Ejecute SSH en VRA y entre en la modalidad \[edit\] (config).
   ```bash
   SSH vyatta@<VRA Private IP Address>
   configure
   ```
   {: codeblock}
2.	Cree las reglas SNAT en el VRA, especificando la misma `<Subnet Gateway IP>/<CIDR>` que se ha determinado para la subred/VLAN de zona de la APP en la guía de aprendizaje de suministro de VRA anterior. 
   ```
   set service nat source rule 1000 description 'pass traffic to the internet'
   set service nat source rule 1000 outbound-interface 'dp0bond1'
   set service nat source rule 1000 source address <Subnet Gateway IP>/<CIDR>
   set service nat source rule 1000 translation address masquerade
   commit
   save
   ```
   {: codeblock}

## Creación de cortafuegos
{: #Create_firewalls}

1.	Cree reglas de cortafuegos 
   ```
   set security firewall name APP-TO-OUTSIDE default-action drop
   set security firewall name APP-TO-OUTSIDE description 'APP traffic to the Internet'
   set security firewall name APP-TO-OUTSIDE default-log

   set security firewall name APP-TO-OUTSIDE rule 90 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 90 action accept
   set security firewall name APP-TO-OUTSIDE rule 90 destination port 80

   set security firewall name APP-TO-OUTSIDE rule 100 protocol tcp
   set security firewall name APP-TO-OUTSIDE rule 100 action accept
   set security firewall name APP-TO-OUTSIDE rule 100 destination port 443

   set security firewall name APP-TO-OUTSIDE rule 200 protocol icmp
   set security firewall name APP-TO-OUTSIDE rule 200 icmp type 8
   set security firewall name APP-TO-OUTSIDE rule 200 action accept
   commit
   ```
   {: codeblock}

## Creación de zona y aplicación de reglas
{: #Create_zone}

1.	Cree la zona OUTSIDE para controlar el acceso a internet externo.
   ```
   set security zone-policy zone OUTSIDE default-action drop
   set security zone-policy zone OUTSIDE interface dp0bond1
   set security zone-policy zone OUTSIDE description 'External internet'
   ```
   {: codeblock}
2.	Asigne cortafuegos para controlar el tráfico de entrada y de salida de internet.
   ```
   set security zone-policy zone APP to OUTSIDE firewall APP-TO-OUTSIDE
   commit
   save
   ```
   {: codeblock}
3.	Compruebe que la VSI de la zona APP ahora puede acceder a los servicios de Internet. Inicie una sesión en la VSI local utilizando SSH, utilice ping y curl para validar el acceso icmp y tcp a los sitios de Internet.  
   ```bash
   ssh root@<VSI Private IP>
   ping 8.8.8.8
   curl www.google.com
   ```
   {: codeblock}

## Eliminación de recursos
{:removeresources}
Pasos que se deben seguir para eliminar los recursos que se crean en esta guía de aprendizaje. 

El VRA está en el plan de pago mensual. La cancelación no da derecho a un reembolso. Se recomienda cancelar solo si este VRA no se volverá a necesitar durante el mes siguiente. Si se necesita un clúster de alta disponibilidad VRA dual, este VRA se puede actualizar en la página [Detalles de pasarela](https://{DomainName}/classic/network/gatewayappliances).
{:tip}  

1. Cancele todos los servidores virtuales o servidores locales
2. Cancele el VRA
3. Cancele las VLAN adicionales por incidencia de soporte. 

## Material relacionado
{:related}

-	[Conversión de direcciones de red VRA]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#network-address-translation-nat-) 
-	[Máscaras de NAT]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-setting-up-nat-rules-on-vyatta-5400#one-to-many-nat-rule-masquerade-)
-	[Documentación de VRA complementaria]( https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-supplemental-vra-documentation#supplemental-vra-documentation).

