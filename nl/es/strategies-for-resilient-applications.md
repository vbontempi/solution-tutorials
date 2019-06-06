---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-11"
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

# Estrategias para aplicaciones resistentes
{: #strategies-for-resilient-applications}

Independientemente de la opción de cálculo (Kubernetes, Cloud Foundry, Cloud Functions o servidores virtuales), las empresas tratan de minimizar el tiempo de inactividad y de crear arquitecturas resistentes que alcancen la máxima disponibilidad. En esta guía de aprendizaje se describen las prestaciones que ofrece IBM Cloud para crear soluciones resistentes y, de este modo, responder a las preguntas siguientes.

- ¿Qué debo tener en cuenta a la hora de preparar una solución para que esté disponible a nivel mundial?
- ¿Cómo me ayudan las opciones de cálculo disponibles a ofrecer aplicaciones de varias regiones?
- ¿Cómo se importan los artefactos de aplicación o de servicio en regiones adicionales?
- ¿Cómo se pueden duplicar las bases de datos entre ubicaciones?
- ¿Qué servicios de respaldo se deben utilizar: almacenamiento en bloque, almacenamiento de archivos, Object Storage, bases de datos?
- ¿Hay alguna consideración específica del servicio?

## Objetivos
{: #objectives}

* Aprender los conceptos de la arquitectura relacionados con la creación de aplicaciones resistentes.
* Comprender cómo se correlacionan estos conceptos con las ofertas de servicios y de cálculo de IBM Cloud

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
* [{{site.data.keyword.cfee_full_notm}}](https://{DomainName}/cfadmin/create)
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
* [{{site.data.keyword.BluVirtServers}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudant)
* [{{site.data.keyword.Db2_on_Cloud_short}}](https://{DomainName}/catalog/services/db2)
* {{site.data.keyword.databases-for}}
* [{{site.data.keyword.cos_short}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura y conceptos

{: #architecture}

Para diseñar una arquitectura resistente, debe tener en cuenta los bloques individuales de su solución y sus capacidades específicas. 

A continuación se muestra una arquitectura de varias regiones que muestra los diferentes componentes que pueden existir en una configuración de varias regiones. ![Arquitectura](images/solution39/Architecture.png)

El diagrama de la arquitectura anterior puede ser diferente en función de la opción de cálculo. Verá diagramas de arquitecturas específicas bajo cada opción de cálculo en las secciones siguientes. 

### Recuperación tras desastre con dos regiones 

Para facilitar la recuperación tras desastre, se utilizan dos arquitecturas ampliamente aceptadas: **activa/activa** y **activa/pasiva**. Cada arquitectura tiene sus propios costes y ventajas relacionados con el tiempo y el esfuerzo durante la recuperación.

#### Configuración activa-activa

En una arquitectura activa/activa, ambas ubicaciones tienen instancias activas idénticas con un equilibrador de carga que distribuye el tráfico entre ellas. Con este enfoque, la duplicación de datos debe estar en vigor para sincronizar los datos entre ambas regiones en tiempo real.

![Activa/Activa](images/solution39/Active-active.png)

Esta configuración proporciona una mayor disponibilidad con menos remediación menos que una arquitectura activa/pasiva. Las solicitudes se responden desde ambos centros de datos. Debe configurar los servicios de extremo (equilibrador de carga) con el tiempo de espera y la lógica de reintentos adecuados para direccionar automáticamente la solicitud al segundo centro de datos si se produce una anomalía en el entorno del primer centro de datos.

Cuando examine el **objetivo de punto de recuperación** (RPO) en un entorno activo/activo, la sincronización de datos entre los dos centros de datos activos debe ser extremadamente puntual para permitir un flujo de solicitudes fluido.

#### Configuración activa-pasiva

Una arquitectura activa/pasiva se basa en una región activa y en una segunda región (pasiva) que se utiliza como reserva. En el caso de que se produzca un corte en la región activa, la región pasiva pasa a estar activa. Es posible que se necesite intervención manual para garantizar que las bases de datos o el almacenamiento de archivos estén actualizados con los requisitos de la aplicación y de los usuarios. 

![Activa/Activa](images/solution39/Active-passive.png)

Las solicitudes se responden desde el sitio activo. En el caso de que se produzca un corte o un error de la aplicación, se realiza un trabajo previo a la aplicación para que el centro de datos en espera esté listo para atender la solicitud. El paso del centro de datos activo al pasivo es una operación que lleva mucho tiempo. Tanto el **objetivo de tiempo de recuperación** (RTO) como el **objetivo de punto de recuperación** (RPO) son altos en comparación con la configuración activa/activa.

### Recuperación tras desastre con tres regiones

En la época actual de servicios "Siempre activos" con tolerancia cero al tiempo de inactividad, los clientes esperan que se pueda acceder a cada uno de los servicios del negocio desde cualquier parte del mundo. Una estrategia rentable para las empresas implica el diseño de la infraestructura de modo que ofrezca disponibilidad continua, en lugar de crear infraestructuras de recuperación tras desastre.

El uso de tres centros de datos ofrece una mayor resistencia y disponibilidad que el uso de dos. También puede ofrecer un mejor rendimiento, al distribuirse la carga de forma más uniforme entre los centros de datos. Si la empresa solo dispone de dos centros de datos, una variante consiste en desplegar dos aplicaciones en un centro de datos y desplegar la tercera aplicación en el segundo centro de datos. De forma alternativa, se puede desplegar la lógica empresarial y las capas de presentación en la topología de 3 activas y desplegar la capa de datos en la topología de 2 activas.

#### Configuración activa-activa-activa (3 activas)

![](images/solution39/Active-active-active.png)

Responde a las solicitudes la aplicación que se ejecuta en cualquiera de los tres centros de datos activos. Un estudio de caso del sitio web de IBM.com indica que la configuración 3 activas solo necesita el 50 % la capacidad de cálculo, de memoria y de red por clúster, pero que la configuración 2 activa necesita el 100 % por clúster. La capa de datos es donde se radica la diferencia de coste. Para ver más información, consulte el artículo [*Always On: Assess, Design, Implement, and Manage Continuous Availability*](http://www.redbooks.ibm.com/redpapers/pdfs/redp5109.pdf).

#### Configuración activa-activa-pasiva

![](images/solution39/Active-active-passive.png)

En este caso, cuando cualquiera de las dos aplicaciones activas de los centros de datos primario y secundario sufren una caída, se activa la aplicación en espera del tercer centro de datos. Se sigue el procedimiento de recuperación tras desastre descrito en el caso de ejemplo de dos centros de datos para restaurar la normalidad en el proceso de solicitudes de los clientes. La aplicación en espera del tercer centro de datos se puede configurar con una configuración de espera en caliente o en frío.

Consulte [esta guía](https://www.ibm.com/cloud/garage/content/manage/hadr-on-premises-app/) para obtener más información sobre la recuperación tras desastre.

### Arquitecturas de varias regiones

En una arquitectura de varias regiones, una aplicación se despliega en diferentes ubicaciones donde cada región ejecuta una copia idéntica de la aplicación. 

Una región es una ubicación geográfica específica en la que puede desplegar apps, servicios y otros recursos de {{site.data.keyword.cloud_notm}}. Las [regiones de {{site.data.keyword.cloud_notm}}](https://{DomainName}/docs/containers?topic=containers-regions-and-zones) consisten en una o varias zonas, que son centros de datos físicos que alojan los recursos de cálculo, de red y de almacenamiento y la refrigeración y la potencia que alojan servicios y aplicaciones. Las zonas están aisladas unas de otras, lo que garantiza que no haya un punto único de error compartido.

Además, en una arquitectura de varias regiones, se necesita un equilibrador de carga global como [Cloud Internet Services](https://{DomainName}/catalog/services/internet-services) para distribuir el tráfico entre las regiones.

El despliegue de una solución en varias regiones ofrece las ventajas siguientes:
- Mejora de la latencia para los usuarios finales: la velocidad es la clave, de modo que cuanto más cerca esté el origen del programa de fondo de los usuarios finales, mejor y más rápida será la experiencia de los usuarios.
- Recuperación tras desastre: cuando falla la región activa, tiene una región de reserva para realizar una rápida recuperación.
- Requisitos empresariales: en algunos casos es necesario almacenar datos en regiones distintas, separados por varios cientos de kilómetros. Por lo tanto, los que se encuentran en este caso tienen que almacenar datos en varias regiones. 

### Arquitecturas con varias zonas dentro de las regiones

Crear aplicaciones con regiones de varias zonas significa que la aplicación se despliega entre varias zonas dentro de una región, y también se pueden tener dos o tres regiones. 

Con la arquitectura de regiones con varias zonas se necesita un equilibrador de carga local que distribuya el tráfico localmente entre las zonas de una región y además, si se configura una segunda región, un equilibrador de carga global que distribuya el tráfico entre las regiones. 

Encontrará más información sobre regiones y zonas [aquí](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-zones).

## Opciones de cálculo 

En esta sección se describen las opciones de cálculo que están disponibles en {{site.data.keyword.cloud_notm}}. Para cada opción de cálculo, se proporciona un diagrama de arquitectura junto con una guía de aprendizaje sobre cómo desplegar dicha arquitectura.

Nota: las arquitecturas de opciones de cálculo no incluyen bases de datos ni otros servicios; solo se centran en desplegar una app en dos regiones para la opción de cálculo seleccionada. Cuando haya desplegado cualquiera de los ejemplos de opciones de cálculo de varias regiones, el siguiente paso lógico sería añadir bases de datos y otros servicios. En las secciones siguientes de esta guía de aprendizaje de soluciones se explican las opciones de [base de datos](#databaseservices) y de [servicios que no son de base de datos](#nondatabaseservices).

### Cloud Foundry 

Cloud Foundry permite desplegar una arquitectura de varias regiones; también mediante servicios de conducto de [entrega continua](https://{DomainName}/catalog/services/continuous-delivery) se puede desplegar la aplicación entre varias regiones. La arquitectura de varias regiones de Cloud Foundry es similar a la siguiente:

![Arquitectura de CF](images/solution39/CF2-Architecture.png)

La misma aplicación se despliega en varias regiones y un equilibrador de carga global direcciona el tráfico a la región en buen estado más cercana. En la guía de aprendizaje [**Aplicación web segura entre varias regiones**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp) se muestra el despliegue de una arquitectura similar.

### {{site.data.keyword.cfee_full_notm}}

**{{site.data.keyword.cfee_full_notm}} (CFEE)** ofrece las mismas funciones que Cloud Foundry pública, junto con características adicionales.

**{{site.data.keyword.cfee_full_notm}}** le permite crear instancias de varias plataformas de Cloud Foundry, aisladas y de grado empresarial según demanda. Las instancias de CFEE se ejecutan dentro de su propia cuenta en [{{site.data.keyword.cloud_notm}}
](http://ibm.com/cloud). El entorno se despliega en hardware aislado sobre [{{site.data.keyword.containershort_notm}}](https://www.ibm.com/cloud/container-service). El usuario tiene el control completo sobre el entorno, incluido el control de accesos, la gestión de la capacidad, la gestión de cambios, la supervisión y los servicios.

A continuación se muestra una arquitectura de varias regiones que utiliza {{site.data.keyword.cfee_full_notm}}.

![Arquitectura](images/solution39/CFEE-Architecture.png)

Para desplegar esta arquitectura se necesita lo siguiente: 

- Configurar dos instancias de CFEE: una en cada región.
- Crear y enlazar los servicios a la cuenta de CFEE. 
- Enviar las apps destinadas al punto final de la API de CFEE. 
- Configurar la réplica de la base de datos, tal como se haría en Cloud Foundry pública. 

Consulte también la guía paso a paso [Despliegue del asistente de logística en Cloud Foundry Enterprise Environment (CFEE)](https://github.com/IBM-Cloud/logistics-wizard/blob/master/Deploy_Microservices_CFEE.md). Le mostrará el despliegue de una aplicación basada en microservicios en CFEE. Una vez desplegada en una instancia de CFEE, puede crear una réplica del procedimiento en una segunda región y conectar los [Servicios de Internet](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) frente a las dos instancias de CFEE para equilibrar la carga del tráfico. 

Consulte la [documentación de {{site.data.keyword.cfee_full_notm}}](https://{DomainName}/docs/cloud-foundry?topic=cloud-foundry-about#about) para obtener más información.

### Kubernetes

Con Kubernetes, puede conseguir una arquitectura de varias zonas dentro de las regiones; podría ser un caso de uso de la arquitectura activa/activa. Si implementa una solución con {{site.data.keyword.containershort_notm}}, se beneficia de las prestaciones incorporadas, como el equilibrio de carga y el aislamiento y el aumento de la resistencia frente a posibles anomalías con hosts, redes o apps. Mediante la creación de varios clústeres, si se produce una interrupción en un clúster, los usuarios pueden acceder a una app que también está desplegada en otro clúster. Con varios clústeres en regiones diferentes, los usuarios también pueden acceder al clúster más cercano con una latencia de red reducida. Para conseguir una mayor resistencia, tiene la opción de seleccionar también los clústeres de varias zonas, lo que significa que los nodos se despliegan en varias zonas dentro de una región. 

La arquitectura de varias regiones de Kubernetes se parece a la siguiente.

![Kubernetes](images/solution39/Kub-Architecture.png)

1. El desarrollador crea imágenes de Docker para la aplicación.
2. Las imágenes se envían a {{site.data.keyword.registryshort_notm}} en dos ubicaciones diferentes.
3. La aplicación se despliega en clústeres de Kubernetes en ambas ubicaciones.
4. Los usuarios finales acceden a la aplicación.
5. Se configura Cloud Internet Services para que intercepte las solicitudes a la aplicación y para que distribuya la carga entre los clústeres. También se habilita la protección DDoS y el cortafuegos de aplicaciones web para proteger las aplicaciones frente a amenazas comunes. Los activos como imágenes y archivos CSS se pueden colocar en la memoria caché.

La guía de aprendizaje [**Clústeres de Kubernetes de varias regiones resistentes y seguros con Cloud Internet Services**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis) le muestra los pasos a seguir para desplegar esta arquitectura.

### {{site.data.keyword.openwhisk_short}}

{{site.data.keyword.openwhisk_short}} está disponible en varias ubicaciones de {{site.data.keyword.cloud_notm}}. Para aumentar la resistencia y reducir la latencia de red, las aplicaciones pueden desplegar su programa de fondo en varias ubicaciones. A continuación, con IBM Cloud Internet Services (CIS), los desarrolladores pueden exponer un único punto de entrada encargado de distribuir el tráfico al programa de fondo en buen estado más cercano. La arquitectura de varias regiones de {{site.data.keyword.openwhisk_short}} se parece a esta.

 ![Funciones-Arquitectura](images/solution39/Functions-Architecture.png)

1. Los usuarios acceden a la aplicación. La solicitud pasa por Internet Services.
2. Internet Services redirige a los usuarios al programa de fondo de API en buen estado más cercano.
3. El gestor de certificados proporciona la API con su certificado SSL. El tráfico se cifra de extremo a extremo.
4. La API se implementa con Cloud Functions.

Encontrará más información sobre esta arquitectura en la guía de aprendizaje [**Despliegue de apps sin servidor entre varias regiones**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#multi-region-serverless).

### {{site.data.keyword.baremetal_short}} y {{site.data.keyword.virtualmachinesshort}}

{{site.data.keyword.virtualmachinesshort}} y {{site.data.keyword.baremetal_short}} ofrecen la posibilidad de implementar una arquitectura de varias regiones. Puede suministrar servidores en varias ubicaciones disponibles en {{site.data.keyword.cloud_notm}}.

![ubicaciones de servidor](images/solution39/ServersLocation.png)

Cuando prepare este tipo de arquitectura mediante {{site.data.keyword.virtualmachinesshort}} y {{site.data.keyword.baremetal_short}}, tenga en cuenta lo siguiente: almacenamiento de archivos, copias de seguridad, recuperación y bases de datos, selección entre una base de datos como servicio o instalación de una base de datos en un servidor virtual. 

En la arquitectura siguiente se muestra el despliegue de una arquitectura de varias regiones mediante {{site.data.keyword.virtualmachinesshort}} en una arquitectura de tipo activa/pasiva donde una región está activa y la segunda región es pasiva. 

![VM-Arquitectura](images/solution39/vm-Architecture2.png)

Estos son los componentes necesarios para esta arquitectura: 

1. Los usuarios acceden a la aplicación a través de IBM Cloud Internet Services (CIS).
2. CIS direcciona el tráfico a la ubicación activa.
3. Dentro de una ubicación, un equilibrador de carga redirecciona el tráfico a un servidor.
4. Las bases de datos se despliegan en un servidor virtual, lo que significa que debe configurar la base de datos y configurar réplicas y copias de seguridad entre regiones. La alternativa sería utilizar una base de datos como servicio, un tema que se trata más adelante en la guía de aprendizaje.
5. El almacenamiento de archivos para almacenar las imágenes y los archivos de la aplicación, almacenamiento de archivos, ofrece la posibilidad de tomar una instantánea en un momento y una fecha determinados, y esta instantánea se puede reutilizar en otra región, algo que se haría manualmente. 

En la guía de aprendizaje [**Utilización de servidores virtuales para crear una app web escalable y con alta disponibilidad**](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#highly-available-and-scalable-web-application) se implementa esta arquitectura.

## Bases de datos y archivos de aplicación
{: #databaseservices}

{{site.data.keyword.cloud_notm}} ofrece diversas [bases de datos como servicio](https://{DomainName}/catalog/?category=databases) con bases de datos relacionales y no relacionales, en función de las necesidades de su empresa. Una [base de datos como servicio (DBaaS)](https://www.ibm.com/cloud/learn/what-is-cloud-database) ofrece varias ventajas. Mediante una DBaaS como {{site.data.keyword.cloudant}}, puede aprovechar el soporte de varias regiones que le permite realizar réplicas en directo entre los servicios de base de datos de distintas regiones, realizar copias de seguridad, realizar funciones de escalado y maximizar el tiempo de actividad. 

**Principales características:** 

- Un servicio de base de datos que se crea y al que se accede a través de una plataforma de nube
- Permite a los usuarios de la empresa alojar bases de datos sin tener que adquirir hardware dedicado
- Puede gestionarla el usuario o se puede ofrecer como servicio y que lo gestione un proveedor
- Da soporte a bases de datos SQL o NoSQL
- Se accede a través de una interfaz web o una API de proveedor

**Preparación para la arquitectura de varias regiones:**

- ¿Cuáles son las opciones de resistencia del servicio de base de datos?
- ¿Cómo se maneja la réplica entre varios servicios de base de datos entre regiones?
- ¿Cómo se hace copia de seguridad de los datos?
- ¿Cuáles son los enfoques de recuperación tras desastre para cada opción?

### {{site.data.keyword.cloudant}}

{{site.data.keyword.cloudant}} es una base de datos distribuida que está optimizada para manejar cargas de trabajo pesadas, típicas de apps móviles y web grandes y de rápido crecimiento. Disponible como un servicio de {{site.data.keyword.Bluemix_notm}} con soporte de SLA y completamente gestionado, {{site.data.keyword.cloudant}} puede escalar de forma elástica el rendimiento y el almacenamiento por separado. {{site.data.keyword.cloudant}} también está disponible como una instalación local descargable, y su API y su potente protocolo de réplica son compatibles con un ecosistema de código abierto que incluye CouchDB, PouchDB y bibliotecas para las pilas de desarrollo web y móvil más populares.

{{site.data.keyword.cloudant}} da soporte a la [réplica](https://{DomainName}/docs/services/Cloudant/api?topic=cloudant-replication-api#replication-operation) entre varias instancias en distintas ubicaciones. Cualquier cambio que se haya producido en la base de datos de origen se reproduce en la base de datos de destino. Puede crear réplicas entre tantas bases de datos como desee, ya sea de forma continua o como tarea única. En el diagrama siguiente se muestra una configuración típica que utiliza dos instancias de {{site.data.keyword.cloudant}}, una en cada región:

![activa-activa](images/solution39/Active-active.png)

Consulte [estas instrucciones](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-configuring-ibm-cloudant-for-cross-region-disaster-recovery#configuring-ibm-cloudant-for-cross-region-disaster-recovery) para configurar la réplica entre instancias de {{site.data.keyword.cloudant}}. El servicio también proporciona instrucciones y herramientas para [copia hacer copia de seguridad y restaurar datos](https://{DomainName}/docs/services/Cloudant/guides?topic=cloudant-ibm-cloudant-backup-and-recovery#ibm-cloudant-backup-and-recovery).

### {{site.data.keyword.Db2_on_Cloud_short}}, {{site.data.keyword.dashdbshort_notm}} y {{site.data.keyword.Db2Hosted_notm}}

{{site.data.keyword.cloud_notm}} ofrece varios [servicios de bases de datos Db2](https://{DomainName}/catalog/?search=db2h). Son los siguientes:

- [**{{site.data.keyword.Db2_on_Cloud_short}}**](https://{DomainName}/catalog/services/db2): una base de datos SQL en la nube completamente gestionada para cargas de trabajo operativas típicas, de tipo OLTP.
- [**{{site.data.keyword.dashdbshort_notm}}**](https://{DomainName}/catalog/services/db2-warehouse): un servicio de depósito de datos en la nube completamente gestionado para cargas de trabajo analíticas de alto rendimiento y con tamaños de petabytes. Ofrece planes de servicio tanto SMP como MPP y utiliza un almacén de datos optimizado en columnas y proceso en memoria.
- [**{{site.data.keyword.Db2Hosted_notm}}**](https://{DomainName}/catalog/services/db2-hosted): un sistema de base de datos alojado por IBM y gestionado por el usuario. Ofrece Db2 con acceso administrativo completo a la infraestructura de la nube, con lo que elimina el coste, la complejidad y el riesgo de gestionar su propia infraestructura.

A continuación no centraremos en {{site.data.keyword.Db2_on_Cloud_short}} como DBaaS para cargas de trabajo operativas. Estas cargas de trabajo son típicas para las aplicaciones que se describen en esta guía de aprendizaje.

#### Soporte de varias regiones para {{site.data.keyword.Db2_on_Cloud_short}}

{{site.data.keyword.Db2_on_Cloud_short}} ofrece varias [opciones de alta disponibilidad y recuperación tras desastre (HADR)](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-fs#overview). Puede elegir la opción de alta disponibilidad cuando cree un nuevo servicio. Más adelante, puede [añadir un nodo de recuperación tras desastre geo-replicado](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha) desde el panel de control de la instancia. La opción de nodo de recuperación tras desastre fuera del local le proporciona la posibilidad de sincronizar los datos en tiempo real con un nodo de base de datos en el centro de datos de {{site.data.keyword.cloud_notm}} fuera del sitio que elija.

Encontrará más información en la [documentación sobre alta disponibilidad](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-ha#ha).

#### Copia de seguridad y restauración

{{site.data.keyword.Db2_on_Cloud_short}} incluye copias de seguridad diarias para los planes de pago. Generalmente, las copias de seguridad se almacenan mediante {{site.data.keyword.cos_short}} y por lo tanto se utilizan tres centros de datos para aumentar la disponibilidad de los datos retenidos. Las copias de seguridad se conservan durante 14 días. Puede utilizarlas para realizar una recuperación en un punto en el tiempo. La [documentación sobre copia de seguridad y restauración](https://{DomainName}/docs/services/Db2onCloud?topic=Db2onCloud-bnr#br) contiene detalles sobre cómo puede restaurar los datos en el punto en que estaban en la fecha y hora deseadas.

### {{site.data.keyword.databases-for}}

{{site.data.keyword.databases-for}} ofrece varios sistemas de base de datos de código abierto como servicios completamente gestionados. Son los siguientes:  
* [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)
* [{{site.data.keyword.databases-for-redis}}](https://{DomainName}/catalog/services/databases-for-redis)
* [{{site.data.keyword.databases-for-elasticsearch}}](https://{DomainName}/catalog/services/databases-for-elasticsearch)
* [{{site.data.keyword.databases-for-etcd}}](https://{DomainName}/catalog/services/databases-for-etcd)
* [{{site.data.keyword.databases-for-mongodb}}](https://{DomainName}/catalog/services/databases-for-mongodb)

Todos estos servicios comparten las mismas características:   
* Para proporcionar alta disponibilidad, se despliegan en clústeres. Encontrará detalles en la documentación de cada servicio:
  - [PostgreSQL](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-high-availability#high-availability)
  - [Redis](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-high-availability#high-availability)
  - [Elasticsearch](https://{DomainName}/docs/services/databases-for-elasticsearch?topic=databases-for-elasticsearch-high-availability#high-availability)
  - [etcd](https://{DomainName}/docs/services/databases-for-etcd?topic=databases-for-etcd-high-availability#high-availability)
  - [MongoDB](https://{DomainName}/docs/services/databases-for-mongodb?topic=databases-for-mongodb-high-availability#high-availability)
* Cada clúster se distribuye entre varias zonas.
* Se realizan réplicas de los datos en las zonas.
* Los usuarios pueden ampliar los recursos de almacenamiento y de memoria para una instancia. Consulte la [documentación sobre escalado, es decir, {{site.data.keyword.databases-for-redis}}](https://{DomainName}/docs/services/databases-for-redis?topic=databases-for-redis-dashboard-settings#settings), para ver más detalles.
* Las copias de seguridad se realizan a diario o a petición. Los detalles están documentados para cada servicio. A continuación encontrará la [documentación sobre copia de seguridad, es decir, {{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/docs/services/databases-for-postgresql?topic=databases-for-postgresql-dashboard-backups#backups).
* Los datos en reposo, las copias de seguridad y el tráfico de red están cifrados.
* Cada [servicio se puede gestionar mediante el plugin de la CLI de {{site.data.keyword.databases-for}}](https://{DomainName}/docs/databases-cli-plugin?topic=cloud-databases-cli-cdb-reference#cloud-databases-cli-plug-in)


### {{site.data.keyword.cos_full_notm}}

{{site.data.keyword.cos_full_notm}} (COS) proporciona almacenamiento en la nube duradero, seguro y rentable. La información que se almacena con {{site.data.keyword.cos_full_notm}} está cifrada y distribuida entre varias ubicaciones geográficas. Cuando se crean grupos de almacenamiento dentro de una instancia de COS, el usuario decide en qué ubicación se debe crear el grupo y qué opción de resistencia se debe utilizar.

Existen tres tipos de resistencia de grupo:
   - La opción de resistencia **Varias regiones** distribuye los datos entre varias áreas metropolitanas. Se puede considerar una opción de varias regiones. Cuando se accede al contenido almacenado en un grupo con la opción Varias regiones, COS ofrece un punto final especial capaz de recuperar el contenido de una región en buen estado.
   - La resistencia **regional** distribuye los datos en una sola área metropolitana. Se puede considerar una opción de varias zonas dentro de una configuración de región.
   - La resistencia de **un solo centro de datos** distribuye los datos entre varios dispositivos dentro de un único centro de datos.

Consulte [esta documentación](https://{DomainName}/docs/services/cloud-object-storage/basics?topic=cloud-object-storage-endpoints#select-regions-and-endpoints) para obtener más información sobre las opciones de resistencia de {{site.data.keyword.cos_full_notm}}.

### {{site.data.keyword.filestorage_full_notm}}

{{site.data.keyword.filestorage_full_notm}} es un sistema de almacenamiento de archivos basado en NFS persistente, rápido, flexible y conectado a la red. En este entorno de almacenamiento adjunto de red (NAS), tiene un control total sobre la función y el rendimiento de las comparticiones de archivos. Los recursos compartidos de {{site.data.keyword.filestorage_short}} se pueden conectar a hasta 64 dispositivos autorizados sobre conexiones TCP/IP direccionadas para aumentar la capacidad de recuperación.

Algunas de las características del almacenamiento de archivos son _Instantáneas_, _Réplica_ y _Acceso simultáneo_. Consulte [la documentación](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-about#getting-started-with-file-storage) para ver una lista completa de características.

Una vez conectado a los servidores, se puede utilizar fácilmente un servicio {{site.data.keyword.filestorage_short}} para almacenar copias de seguridad de datos y archivos de aplicación como imágenes y vídeos; estas imágenes y archivos se pueden utilizar en servidores diferentes de la misma región.

Cuando se añade una segunda región, se puede utilizar la característica de instantáneas de {{site.data.keyword.filestorage_short}} para realizar una instantánea automáticamente o de forma manual y luego reutilizarla en la segunda región pasiva. 

La réplica se puede planificar para que copie automáticamente las instantáneas en un volumen de destino de un centro de datos remoto. Las copias se pueden recuperar en el sitio remoto si se produce un suceso catastrófico o los datos resultan dañados. Encontrará más información sobre las instantáneas del almacenamiento de archivos [aquí](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#snapshots).

## Servicios que no son de base de datos
{: #nondatabaseservices}

{{site.data.keyword.cloud_notm}} ofrece varios [servicios](https://{DomainName}/catalog) que no son de base de datos, tanto de IBM como de terceros. Cuando planifique una arquitectura de varias regiones, tendrá que entender cómo funcionan los servicios, como los servicios Watson, en una configuración de varias regiones.

### {{site.data.keyword.conversationfull}}

[{{site.data.keyword.conversationfull}}](https://{DomainName}/docs/services/assistant?topic=assistant-index#about) es una plataforma que permite a los desarrolladores y a los usuarios que no son técnicos colaborar en la creación de asistentes conversacionales con soporte de IA.

Un asistente es un bot cognitivo que puede personalizar para adaptarlo a sus necesidades empresariales y desplegar en varios canales para ofrecer ayuda a los clientes donde y cuando la necesiten. El asistente incluye uno o varios conocimientos. Un conocimiento de diálogo contiene los datos de entrenamiento y la lógica que permite a un asistente ayudar a sus clientes.

Es importante tener en cuenta que {{site.data.keyword.conversationshort}} V1 no tiene estado. {{site.data.keyword.conversationshort}} ofrece un tiempo de actividad del 99,5 %; sin embargo, es posible que para las aplicaciones altamente disponibles en varias regiones desee tener varias instancias de estos servicios en las regiones. 

Una vez que ha creado instancias en varias ubicaciones, utilice la herramienta {{site.data.keyword.conversationshort}} para exportar, desde una instancia, un espacio de trabajo existente, incluidas intenciones, entidades y diálogo. A continuación, importe este espacio de trabajo en otras ubicaciones.

## Resumen

| Oferta | Opciones de resistencia |
| -------- | ------------------ |
| Cloud Foundry | <ul><li>Desplegar aplicaciones en varias ubicaciones</li><li>Responder a solicitudes procedentes de varias ubicaciones con Cloud Internet Services</li><li>Utilizar las API de Cloud Foundry para configurar organizaciones y espacios y enviar apps a varias ubicaciones</li></ul> |
| {{site.data.keyword.cfee_full_notm}} | <ul><li>Desplegar aplicaciones en varias ubicaciones</li><li>Responder a solicitudes procedentes de varias ubicaciones con Cloud Internet Services</li><li>Utilizar las API de Cloud Foundry para configurar organizaciones y espacios y enviar apps a varias ubicaciones</li><li>Basado en el servicio Kubernetes</li></ul> |
| {{site.data.keyword.containerlong_notm}} | <ul><li>Resistencia por diseño con soporte para clústeres de varias zonas</li><li>Responder a solicitudes procedentes de clústeres distribuidos en varias ubicaciones con Cloud Internet Services</li></ul> |
| {{site.data.keyword.openwhisk_short}} | <ul><li>Disponible en múltiples ubicaciones</li><li>Responder a solicitudes procedentes de varias ubicaciones con Cloud Internet Services</li><li>Utilizar la API de Cloud Functions para desplegar acciones en varias ubicaciones</li></ul> |
| {{site.data.keyword.baremetal_short}} y {{site.data.keyword.virtualmachinesshort}} | <ul><li>Suministrar servidores en múltiples ubicaciones</li><li>Conectar servidores de la misma ubicación a un equilibrador de carga local</li><li>Responder a solicitudes procedentes de varias ubicaciones con Cloud Internet Services</li></ul> |
| {{site.data.keyword.cloudant}} | <ul><li>Réplica única y continua entre bases de datos</li><li>Redundancia de datos automática dentro de una región</li></ul> |
| {{site.data.keyword.Db2_on_Cloud_short}} | <ul><li>Suministrar un nodo de recuperación tras desastre geo-replicado para la sincronización de datos en tiempo real</li><li>Copia de seguridad diaria con planes de pago</li></ul> |
| {{site.data.keyword.databases-for-postgresql}}, {{site.data.keyword.databases-for-redis}} | <ul><li>Basado en clústeres de Kubernetes de varias zonas</li><li>Réplicas de lectura en varias regiones</li><li>Copias de seguridad diarias y a petición</li></ul> |
| {{site.data.keyword.cos_short}} | <ul><li>Un solo centro de datos, resistencia regional y en varias regiones</li><li>Utilizar API para sincronizar el contenido entre los grupos de almacenamiento</li></ul> |
| {{site.data.keyword.filestorage_short}} | <ul><li>Utilizar instantáneas para capturar automáticamente contenido en un destino de un centro de datos remoto</li></ul> |
| {{site.data.keyword.conversationshort}} | <ul><li>Utilizar la API de Watson para exportar e importar la especificación de espacio de trabajo entre varias instancias en distintas ubicaciones</li></ul> |

## Contenido relacionado

{:related}

- IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
- [Mejora de la disponibilidad de la app con clústeres de varias zonas](https://www.ibm.com/blogs/bluemix/2018/06/improving-app-availability-multizone-clusters/)
- [Cloud Foundry, aplicación web segura entre varias regiones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#secure-web-application-across-multiple-regions)
- [Cloud Functions, despliegue de apps sin servidor entre varias regiones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-serverless#deploy-serverless-apps-across-multiple-regions)
- [Kubernetes, clústeres de Kubernetes de varias regiones resistentes y seguros con Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#resilient-and-secure-multi-region-kubernetes-clusters-with-cloud-internet-services)
- [Servidores virtuales, creación de una app web escalable y de alta disponibilidad](https://{DomainName}/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
