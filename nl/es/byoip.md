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

# Traiga su propia dirección IP (BYOIP)
{: #byoip}

Traiga su propia IP (BYOIP) es un requisito frecuente con el que se desea conectar las redes de clientes existentes con la infraestructura suministrada en {{site.data.keyword.Bluemix_notm}}. El objetivo suele ser minimizar el cambio en la configuración de direccionamiento de red de los clientes y en las operaciones con la adopción de un solo espacio de direcciones IP basado en el esquema de direccionamiento de IP existente de los clientes.

En esta guía de aprendizaje se ofrece una breve visión general de los patrones de implementación de BYOIP que se pueden utilizar con {{site.data.keyword.Bluemix_notm}} y un árbol de decisiones para identificar el patrón adecuado cuando se realiza el alojamiento seguro tal como se describe en la guía de aprendizaje [Aislamiento de cargas de trabajo con una red privada segura](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#isolate-workloads-with-a-secure-private-network). Es posible que la configuración requiera entrada de información adicional del equipo de red del sitio, del equipo de soporte técnico de {{site.data.keyword.Bluemix_notm}} o de los servicios de IBM.

{:shortdesc}

## Objetivos
{: #objectives}

* Comprender los patrones de implementación de BYOIP
* Seleccionar el patrón de implementación para {{site.data.keyword.Bluemix_notm}}.

## Direccionamiento de IP de {{site.data.keyword.Bluemix_notm}}

{{site.data.keyword.Bluemix_notm}} utiliza varios rangos de direcciones privadas, más específicamente 10.0.0.0/8, y en algunos casos estos rangos pueden entrar en conflicto con las redes existentes de los clientes. Si existen conflictos de direcciones, hay varios patrones que permiten BYOIP para facilitar la interoperatividad con la red de IBM Cloud.

-	Conversión de direcciones de red
-	Túnel GRE (Generic Routing Encapsulation)
-	Túnel GRE con alias de IP
-	Virtual Overlay Network

La elección del patrón viene determinada por las aplicaciones que se alojan en {{site.data.keyword.Bluemix_notm}}. Hay dos aspectos clave a tener en cuenta: la sensibilidad de la aplicación a la implementación de patrón y el grado de solapamiento de rangos de direcciones entre la red del cliente y {{site.data.keyword.Bluemix_notm}}. También se aplicarán consideraciones adicionales si existe la intención de utilizar una conexión de red privada dedicada con {{site.data.keyword.Bluemix_notm}}. Se recomienda a todos los usuarios que estén pensando en utilizar BYOIP que lean la documentación de [{{site.data.keyword.BluDirectLink}}
](https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link). En el caso de usuarios de {{site.data.keyword.BluDirectLink}}, se deben seguir las directrices asociadas junto con la información aquí presentada.

## Visión general de los patrones de implementación
{: #patterns_overview}

1. **NAT**: conversión de direcciones NAT en el direccionador del cliente local. Utilice NAT local para convertir el esquema de direccionamiento del cliente en las direcciones IP asignadas por {{site.data.keyword.Bluemix_notm}} a los servicios IaaS suministrados.  
2. **Túnel GRE**: el esquema de direccionamiento se unifica direccionando el tráfico IP a través de un túnel GRE entre {{site.data.keyword.Bluemix_notm}} y la red local, normalmente a través de VPN. Este es el caso que se ilustra en [esta guía de aprendizaje](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN). 

   Hay dos subpatrones que dependen del potencial de solapamiento de espacios de direcciones.
     * Sin solapamiento de direcciones: cuando no hay solapamiento de rangos de direcciones ni riesgo de conflicto entre las redes.
     * Solapamiento parcial de direcciones: cuando los espacios de direcciones IP del cliente y de {{site.data.keyword.Bluemix_notm}} utilizan el mismo rango de direcciones y existe potencial de solapamiento y de conflicto. En este caso, se seleccionan direcciones de subred de cliente que no se solapen en la red privada de {{site.data.keyword.Bluemix_notm}}.

3. Túnel GRE + alias de IP
El esquema de direccionamiento se unifica direccionando el tráfico de IP sobre un túnel GRE entre la red local y las direcciones IP de alias asignadas a los servidores en {{site.data.keyword.Bluemix_notm}}. Se trata de un caso especial que se ilustra en [esta guía de aprendizaje](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN). Se crea una interfaz adicional y un alias de IP para una subred IP compatible en los servidores virtuales y nativos que se suministran en {{site.data.keyword.Bluemix_notm}} y que reciben soporte de la configuración de direccionamiento adecuada en el VRA.

4. Virtual Overlay Network
[La red privada virtual (VPC) de {{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure) da soporte a BYOIP para entornos completamente virtuales en {{site.data.keyword.Bluemix_notm}}. Se puede considerar como una alternativa al alojamiento de red privada seguro que se describe en [esta guía de aprendizaje](https://{DomainName}/docs/tutorials?topic=solution-tutorials-secure-network-enclosure#secure-network-enclosure).

De forma alternativa, tenga en cuenta una solución como VMware NSX, que implemente una red de solapamiento virtual en una capa sobre la red de {{site.data.keyword.Bluemix_notm}}. Todas las direcciones BYOIP del solapamiento virtual son independientes de los rangos de direcciones de red de {{site.data.keyword.Bluemix_notm}}. Consulte [Iniciación a VMware y a [{{site.data.keyword.Bluemix_notm}}](https://{DomainName}/docs/infrastructure/vmware?topic=VMware-getting-started-tutorial#getting-started-with-vmware-and-ibm-cloud).

## Árbol de decisiones de patrón
{: #decision_tree}

Puede utilizar este árbol de decisiones para determinar el patrón de implementación adecuado. 

<p style="text-align: center;">

  ![](images/solution37-byoip/byoipdecision.png)
</p>

Las notas siguientes proporcionan orientación adicional:

### ¿Resulta NAT problemático para sus aplicaciones?
{: #nat_consideration}

Hay dos casos distintos en los que NAT podría resultar problemático. En estos casos, no se debe utilizar NAT. 

- Algunas aplicaciones, como la comunicación de dominio de Microsoft AD y las aplicaciones P2P, podrían tener problemas técnicos con NAT.
- En el caso de que servidores desconocidos tenga que comunicarse con {{site.data.keyword.Bluemix_notm}} o de que se necesiten cientos de conexiones bidireccionales entre {{site.data.keyword.Bluemix_notm}} y los servidores locales. En este caso, no se puede configurar toda la correlación en la tabla Direccionador/NAT del cliente debido a una incapacidad para identificar la correlación de antemano.


### Sin solapamiento de direcciones
{: #no-overlap}

¿Se utiliza 10.0.0.0/8 en la red local? Si no hay solapamiento de direcciones entre la red local y la red privada de {{site.data.keyword.Bluemix_notm}}, se puede utilizar el túnel GRE que se describe en [esta guía de aprendizaje](https://{DomainName}/docs/tutorials?topic=solution-tutorials-configuring-IPSEC-VPN#configuring-IPSEC-VPN) entre la instalación local e IBM Cloud para evitar la necesidad de realizar la conversión de NAT. Esto requiere una revisión de la utilización de direcciones de red con el equipo de red en el sitio. 

### Solapamiento parcial de direcciones
{: #partial_overlap}

Si se utiliza alguna de las direcciones del rango 10.0.0.0/8 en la red local, ¿hay subredes disponibles que no se solapen en la red de {{site.data.keyword.Bluemix_notm}}? Revise el uso de las direcciones de red existentes con el equipo de red en el sitio y póngase en contacto con el equipo de técnicos de ventas de {{site.data.keyword.Bluemix_notm}} para identificar las redes disponibles que no se solapan. 

### ¿Resulta problemática la creación de alias de IP?
{: #ip_alias}

Si no existen direcciones libres de solapamiento, se puede implementar la creación de alias de IP en los servidores virtual y nativo desplegados en el alojamiento de la red privada segura. La creación de alias de IP asigna varias direcciones de subred a una o a varias interfaces de red en cada servidor. 

La creación de alias de IP es una técnica de uso común, aunque se recomienda revisar las configuraciones de servidor y de las aplicaciones para determinar si funciona bien en configuraciones de varios inicios y alias de IP.  

Se necesitará una configuración de direccionamiento adicional en el VRA para crear rutas dinámicas (por ejemplo, BGP) o rutas estáticas para las subredes BYOIP. 

## Contenido relacionado
{: #related}

- [{{site.data.keyword.BluDirectLink}}
]( https://{DomainName}/docs/infrastructure/direct-link?topic=direct-link-configure-ibm-cloud-direct-link#configure-ibm-cloud-direct-link)
- [Red privada virtual (VPC)](https://{DomainName}/docs/infrastructure/vpc?topic=vpc-about-ibm-cloud-virtual-private-cloud-vpc-infrastructure#about-ibm-cloud-virtual-private-cloud-vpc-infrastructure)
