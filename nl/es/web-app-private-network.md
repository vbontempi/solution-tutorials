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

# Servicio de aplicaciones web desde una red privada segura
{: #web-app-private-network}

El alojamiento de aplicaciones web es un patrón de despliegue común para la nube pública, donde los recursos se pueden escalar a petición para satisfacer las demandas de uso a corto y a largo plazo. La seguridad de las cargas de trabajo de aplicaciones es un requisito previo fundamental, para complementar la resistencia y la escalabilidad que ofrece la nube pública. 

Esta guía de aprendizaje le muestra cómo crear una aplicación web orientada a Internet escalable y segura alojada en una red privada protegida mediante un dispositivo de direccionador virtual (VRA), VLAN, NAT y cortafuegos. La aplicación comprende un equilibrador de carga, dos servidores de aplicaciones web y un servidor de bases de datos MySQL. Combina tres guías de aprendizaje para ilustrar cómo se pueden desplegar aplicaciones web de forma segura en la plataforma {{site.data.keyword.Bluemix_notm}} IaaS utilizando redes clásicas. 
{:shortdesc}

## Objetivos
{: #objectives}

- Creación de servidores virtuales para instalar PHP y MySQL
- Suministro de un equilibrador de carga para distribuir las solicitudes a los servidores de aplicaciones
- Despliegue de un dispositivo de direccionador virtual (VRA) para crear una red segura
- Definición de VLAN y subredes IP 
- Protección de la red con reglas de cortafuegos
- Source Network Address Translation (SNAT) para el despliegue de aplicaciones

## Servicios utilizados
{: #products}

En esta guía de aprendizaje se utilizan los siguientes servicios de {{site.data.keyword.Bluemix_notm}}: 

* [Virtual Router Appliance VPN](https://{DomainName}/docs/infrastructure/virtual-router-appliance?topic=virtual-router-appliance-about-the-vra#virtual-private-network-vpn-gateway)
* [Equilibrador de carga]( https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}]( https://{DomainName}/catalog/infrastructure/virtual-server-group)

Esta guía de aprendizaje puede incurrir en costes. El VRA solo está disponible en los planes de precios mensuales.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

  ![Arquitectura](images/solution42-web-app-private-network/web-app-private.png)
</p>

1.	Configure una red privada segura
2.	Configure el acceso NAT para el despliegue de aplicaciones
3.	Despliegue la app web escalable y el equilibrador de carga

## Antes de empezar
{: #prereqs}

En esta guía de aprendizaje se utilizan tres guías de aprendizaje existentes, que se despliegan en secuencia. Se deberán examinar las tres antes de comenzar:

-	[Aislamiento de cargas de trabajo con una red privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) 
-	[Configuración de NAT para el acceso a Internet desde una red segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)
-	[Uso de servidores virtuales para crear una app web altamente disponible y escalable]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)



## Configuración de una red privada segura
{: #private_network}

Los entornos de red privados aislados y seguros resultan fundamentales para el modelo de despliegue de aplicaciones IaaS en la nube pública. Los cortafuegos, las VLAN, el direccionamiento y las VPN constituyen componentes necesarios para crear entornos privados aislados. 
El primer paso consiste en crear el alojamiento de la red privada segura dentro del que se desplegará la app web.  

- [Aislamiento de cargas de trabajo con una red privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network)

Esta guía de aprendizaje se puede seguir sin cambios. En un paso posterior, se desplegarán tres máquinas virtuales en la zona APP como servidores web Nginx y una base de datos MySQL. 

## Configuración de NAT para el despliegue de aplicaciones seguras
{: #nat_config}

La instalación de aplicaciones de código abierto requiere un acceso seguro a Internet para acceder a los repositorios de origen. Para evitar que los servidores de la red privada segura de estén expuestos en Internet pública, se utiliza la NAT de origen, donde la dirección de origen está oculta y se utilizan reglas de cortafuegos para proteger las solicitudes del repositorio de aplicaciones de salida. Todas las solicitudes de entrada se deniegan. 

- [Configuración de NAT para el acceso a Internet desde una red segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-nat-config-private#configure-firewall-rules-for-internet-access-from-a-private-network)

Esta guía de aprendizaje se puede seguir sin cambios. En el siguiente paso, se utilizará la configuración NAT para acceder a los módulos Nginx y MySQL necesarios.  


## Despliegue la app web escalable y el equilibrador de carga
{: #scalable_app}

Se utiliza una instalación de Wordpress en Nginx y MySQL con un equilibrador de carga para ilustrar el despliegue de una aplicación web escalable y resistente en una red privada segura 

Esta guía de aprendizaje le muestra este escenario con la creación de un equilibrador de carga de {{site.data.keyword.Bluemix_notm}}, dos servidores de aplicaciones web y un servidor de bases de datos MySQL. Los servidores se despliegan en la zona APP en la red privada segura para proporcionar la separación de cortafuegos de otras cargas de trabajo y de la red pública. 

- [Uso de servidores virtuales para crear una app web altamente disponible y escalable]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)

Hay tres cambios en esta guía de aprendizaje:

1.	Los servidores virtuales que se utilizan en esta guía de aprendizaje se despliegan en la VLAN y en la subred protegida por la zona de cortafuegos de APP detrás de VRA.
2. Especifique el &lt;ID de VLAN privada&gt; cuando solicite los servidores virtuales. Consulte las instrucciones sobre cómo [Solicitar el primer servidor virtual](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#order_virtualserver) de la guía de aprendizaje sobre [Aislamiento de cargas de trabajo con una red privada segura]( https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network) para ver detalles sobre cómo especificar el &lt;ID de VLAN privada&gt; cuando solicite un servidor virtual. Asimismo, recuerde seleccionar la clave SSH que se ha cargado anteriormente en la guía de aprendizaje para permitir el acceso a los servidores virtuales. 
3. Se recomienda encarecidamente **NO** utilizar el servicio de almacenamiento de archivos para esta guía de aprendizaje por su bajo rendimiento de resincronización cuando se copian archivos de Wordpress en el almacenamiento compartido. Esto no afecta a la guía de aprendizaje general. Los pasos relacionados con la creación del almacenamiento de archivos y la configuración de montajes se pueden pasar por alto para los servidores de apps y la base de datos. Como alternativa, se deben seguir todos los pasos de [Instalación y configuración de la aplicación PHP en los servidores de aplicaciones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application) en ambos servidores de apps.
   Antes de seguir los pasos de [Instalación y configuración de la aplicación PHP en los servidores de aplicaciones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#php_application), cree el directorio `/mnt/www/` en ambos servidores de apps. Este directorio se ha creado originalmente en la sección de almacenamiento de archivos, ahora eliminada. 

   ```sh
   mkdir /mnt/www
   ```

Al final de este paso, el equilibrador de carga debería estar en buen estado y el sitio de Wordpress debería resultar accesible en Internet. Los servidores virtuales que forman la aplicación web están protegidos frente al acceso externo a través de Internet por el cortafuegos VRA y el único acceso es a través del equilibrador de carga. En el caso de un entorno de producción, se deben tener en cuenta la protección DDoS y el cortafuegos de aplicaciones web (WAF) proporcionados por [{{site.data.keyword.Bluemix_notm}} Internet Services](https://{DomainName}/catalog/services/internet-services).


## Eliminación de recursos
{: #removeresources}

Pasos que se deben seguir para eliminar los recursos que se crean en esta guía de aprendizaje. 

El VRA está en el plan de pago mensual. La cancelación no da derecho a un reembolso. Se recomienda cancelar solo si este VRA no se volverá a necesitar durante el mes siguiente. 
{:tip}  

1. Cancele todos los servidores virtuales o servidores locales
2. Cancele el VRA
3. Cancele las VLAN adicionales por incidencia de soporte.
4. Suprima el equilibrador de carga
5. Suprima los servicios de almacenamiento de archivos

## Ampliación de la guía de aprendizaje 

1. En esta guía de aprendizaje inicialmente solo se suministran dos servidores virtuales como capa de la app, y se pueden añadir más servidores automáticamente para gestionar la carga adicional. El [escalado automático]( https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-getting-started-with-auto-scaling#create-an-autoscale-group) le ofrece la posibilidad de automatizar el proceso de escalado manual asociado a la adición o a la eliminación de servidores virtuales para dar soporte a sus aplicaciones empresariales.

2. Proteja de forma separada los datos de usuario añadiendo una segunda VLAN privada y una subred IP al VRA para crear una zona DATA para alojar el servidor de bases de datos MySQL. Configure reglas de cortafuegos de modo que solo permitan el tráfico IP de MySQL en el puerto 3306 de entrada de la zona APP a la zona DATA. 

