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

# Despliegue de cargas de trabajo aisladas en varias ubicaciones y zonas
{: #vpc-multi-region}

IBM aceptará la participación de un número limitado de clientes en un programa de acceso temprano a VPC a partir de principios de abril de 2019 y en los meses siguientes se ampliará su uso. Si su organización desea obtener acceso a IBM Virtual Private Cloud, complete este [formulario de nominación](https://{DomainName}/vpc){: new_window} y un representante de IBM se pondrá en contacto con usted para indicarle los siguientes pasos a seguir.
{: important}

En esta guía de aprendizaje se muestran los pasos a seguir para configurar cargas de trabajo aisladas mediante el suministro de VPC en distintas regiones de IBM Cloud. Regiones con subredes e instancias de servidor virtual (VSI). Estos VSI se crean en varias zonas dentro de una región para aumentar la resistencia dentro de una región y a nivel global mediante la configuración de equilibradores de carga con agrupaciones de fondo, escuchas frontales y comprobaciones de estado adecuadas.

Para el equilibrador de carga global, suministrará un servicio IBM Cloud Internet Services (CIS) desde el catálogo y, para gestionar el certificado SSL para todas las solicitudes HTTPS entrantes, se creará el servicio de catálogo de {{site.data.keyword.cloudcerts_long_notm}} y se importará el certificado junto con la clave privada.

{:shortdesc}

## Objetivos
{: #objectives}

* Comprensión del aislamiento de cargas de trabajo mediante los objetos de infraestructura disponibles para nubes privadas virtuales.
* Utilización de un equilibrador de carga entre zonas dentro de una región para distribuir el tráfico entre servidores virtuales.
* Utilización de un equilibrador de carga global entre regiones para aumentar la resistencia y reducir la latencia.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:

- [{{site.data.keyword.vpc_full}}](https://{DomainName}/vpc/provision/vpc)
- [{{site.data.keyword.vsi_is_full}}](https://{DomainName}/vpc/provision/vs)
- [{{site.data.keyword.loadbalancer_full}}](https://{DomainName}/vpc/provision/loadBalancer)
- IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-services)
- [{{site.data.keyword.cloudcerts_long_notm}}](https://{DomainName}/catalog/services/cloudcerts)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/estimator/review) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

  ![Arquitectura](images/solution41-vpc-multi-region/Architecture.png)

1. El administrador (DevOps) suministra VSI en las subredes bajo dos zonas diferentes en una VPC en la región 1 y se repite lo mismo en una VPC creada en la región 2.
2. El administrador crea un equilibrador de carga con una agrupación de servidores de fondo de subredes en distintas zonas de la región 1 y un escucha frontal. Se repite lo mismo en la región 2.
3. El administrador proporciona servicios de Internet en la nube con un dominio personalizado asociado y crea un equilibrador de carga global que apunta a los equilibradores de carga creados en dos VPC diferentes.
4. El administrador habilita el cifrado de HTTPS mediante la adición del certificado SSL de dominio al servicio del gestor de certificados.
5. El usuario de Internet realiza una solicitud HTTP/HTTPS y el equilibrador de carga global la gestiona.
6. La solicitud se direcciona a los equilibradores de carga tanto globales como locales. A continuación, la instancia de servidor disponible responde a la solicitud.

## Antes de empezar
{: #prereqs}

- Compruebe los permisos de usuario. Asegúrese de que la cuenta de usuario tiene permisos suficientes para crear y gestionar recursos de VPC. Para obtener una lista de los permisos necesarios, consulte [Cómo otorgar los permisos necesarios para usuarios de VPC](/docs/infrastructure/vpc?topic=vpc-managing-user-permissions-for-vpc-resources#managing-user-permissions-for-vpc-resources).
- Necesita una clave SSH para conectarse a los servidores virtuales. Si no tiene una clave SSH, consulte las [instrucciones para crear una clave](/docs/infrastructure/vpc?topic=vpc-getting-started-with-ibm-cloud-virtual-private-cloud-infrastructure#prerequisites).
- Cloud Internet Services necesita que tenga un dominio personalizado para poder configurar el DNS para que este dominio apunte a los servidores de nombres de Cloud Internet Services. Si no tiene un dominio, puede adquirir uno de un registrador como [godaddy.com](http://godaddy.com/).

## Creación de VPC, subredes y VSI
{: #create-infrastructure}

En esta sección, creará su propia VPC en la región 1 con subredes creadas en dos zonas diferentes de la región 1 y después suministrará VSI.

Para crear su propia {{site.data.keyword.vpc_short}} en la región 1,

1. Vaya a la página [Visión general de VPC](https://{DomainName}/vpc/overview) y pulse **Crear una VPC**.
2. En la sección **Nueva nube privada virtual**:
   * Especifique **vpc-region1** como nombre para la VPC.
   * Seleccione un **Grupo de recursos**.
   * Si lo desea, añada **Etiquetas** para organizar los recursos.
3. Seleccione **Crear nuevo valor predeterminado (Permitir todo)** como la lista de control de accesos (ACL) predeterminada de la VPC.
4. Elimine la marca de SSH y ejecute ping desde el **Grupo de seguridad predeterminado** y deje **acceso clásico** sin seleccionar.
5. En **Nueva subred para VPC**:
   * Como nombre exclusivo especifique **vpc-region1-zone1-subnet**.
   * Seleccione una ubicación (por ejemplo, Dallas), a la que llamaremos **región 1**, y a una zona en la región 1 (por ejemplo, Dallas 1), a la que llamaremos **zona 1**.
   * Especifique el rango de IP para la subred en notación CIDR, es decir, **10.xxx.0.0/24**. Deje el **Prefijo de dirección** tal como está y seleccione como **Número de direcciones** el valor 256.
6. Seleccione **Utilizar valor predeterminado de VPC** para la lista de control de accesos (ACL) de subred. Puede configurar las reglas de entrada y de salida más adelante.
7. Dado que todas las instancias de servidor virtual de la subred tendrán una IP flotante conectada, no es necesaria habilitar una pasarela pública para la subred. Las instancias del servidor virtual tendrán conectividad a Internet a través de su IP flotante.
8. Pulse **Crear nube privada virtual** para suministrar la instancia.

Para confirmar la creación de la subred, pulse **Subredes** en el panel izquierdo y espere a que el estado sea **Disponible**. Puede crear una nueva subred en **Subredes**.

### Creación de subred en la zona 2

1. Pulse **Nueva subred**, especifique **vpc-region1-zone2-subnet** como nombre exclusivo para la subred y seleccione **vpc-region1** como la VPC.
2. Seleccione la ubicación a la que hemos llamado región 1 arriba (por ejemplo, Dallas) y seleccione otra zona en la región 1 (por ejemplo, Dallas 2); llamemos a la zona seleccionada **zona 2**.
3. Especifique el rango de IP para la subred en notación CIDR, es decir, **10.xxx.64.0/24**. Deje el **Prefijo de dirección** tal como está y seleccione como **Número de direcciones** el valor 256.
4. Seleccione **Utilizar valor predeterminado de VPC** para la lista de control de accesos (ACL) de subred.

### Suministro de VSI
Cuando el estado de las subredes sea **Disponible**,

1. Pulse **vpc-region1-zone1-subnet** y pulse **Instancias conectadas** y luego **Nueva instancia**.
2. Especifique un nombre exclusivo y elija **vpc-region1-zone1-vsi**. A continuación, seleccione la VPC que ha creado anteriormente y la **Ubicación** junto con la **zona** como antes.
3. Elija una imagen de **Ubuntu Linux**, pulse **Todos los perfiles** y, en **Cálculo**, seleccione **c-2x4** con 2 vCPU y 4 GB de RAM.
4. Para **Claves SSH**, seleccione la clave SSH que ha creado inicialmente.
5. En **Interfaces de red**, pulse el icono **Editar** situado junto a Grupos de seguridad.
   * Compruebe si **vpc-region1-zone1-subnet** está seleccionada como subred. Si no es así, selecciónela.
   * Pulse **Guardar**.
   * Pulse **Crear instancia de servidor virtual**.
6.  Espere hasta que el estado de la VSI sea **Activada**. A continuación, seleccione la VSI **vpc-region1-zone1-vsi**, desplácese hasta **Interfaces de red** y pulse **Reservar** en **IP flotante** para asociar una dirección IP pública a la VSI. Guarde la dirección IP asociada en un portapapeles para futuras referencias.
7. **REPITA** los pasos del 1 a 6 para suministrar una VSI en la **zona 2** de la **región 1**.

Vaya a **VPC** y a **Subredes** en **Red** en el panel izquierdo y **REPITA** los pasos anteriores para suministrar una nueva VPC con subredes y VSI en **región 2** siguiendo los mismos convenios de denominación que antes.

## Instalación y configuración del servidor web en las VSI
{: #install-configure-web-server-vsis}

Siga los pasos mostrados en [acceso seguro a instancias remotas con un host bastión](/docs/tutorials?topic=solution-tutorials-vpc-secure-management-bastion-server) para realizar un mantenimiento seguro de los servidores utilizando un host bastión que actúe como servidor `puente` y un grupo de seguridad de mantenimiento.
{:tip}

Cuando haya ejecutado SSH correctamente en el servidor suministrado en la subred de la zona 1 de la región 1,

1. En el indicador, ejecute los mandatos siguientes para instalar Nginx como servidor web
   ```
   sudo apt-get update
   sudo apt-get install nginx
   ```
   {:codeblock}
2. Compruebe el estado del servicio Nginx con el mandato siguiente:
   ```
   sudo systemctl status nginx
   ```
   {:pre}
   La salida debería mostrar que el servicio Nginx está **activo** y en ejecución.
3. Tendrá que abrir los puertos **HTTP (80)** y **HTTPS (443)** para que reciban tráfico (solicitudes). Para ello, ajuste el cortafuegos mediante [UFW](https://help.ubuntu.com/community/UFW) - `sudo ufw enable` y mediante la habilitación del perfil ‘Nginx Full’, que incluye reglas para ambos puertos:
   ```
   sudo ufw allow 'Nginx Full'
   ```
   {:pre}
4. Para verificar que Nginx funciona como lo esperado, abra `http://FLOATING_IP` en el navegador que desee; debería ver la página de bienvenida predeterminada de Nginx.
5. Para actualizar la página html con los detalles de la región y de la zona, ejecute el mandato siguiente
   ```
   nano /var/www/html/index.nginx-debian.html
   ```
   {:pre}
   Agregue la región y la zona (_server running in **zone 1 of region 1**_) a la etiqueta `h1` añadiendo `Welcome to nginx!` y guarde los cambios.
6. Reinicie el servidor nginx para reflejar los cambios
   ```
   sudo systemctl restart nginx
   ```
   {:pre}

**REPITA** los pasos 1-6 para instalar y configurar el servidor web en las VSI de las subredes de todas las zonas y no olvide actualizar el html con información de zona respectiva.


## Distribución del tráfico entre zonas con equilibradores de carga
{: #distribute-traffic-with-load-balancers}

En esta sección, creará dos equilibradores de carga. Una en cada región para distribuir el tráfico entre varias instancias de servidor bajo las subredes respectivas dentro de diferentes zonas.

### Configuración de equilibradores de carga

1. Vaya a **Equilibradores de carga** y pulse **Nuevo equilibrador de carga**.
2. Especifique **vpc-lb-region1** como nombre exclusivo, seleccione **vpc-region1** como nube privada virtual seguida del grupo de recursos que ha creado la VPC: tipo **Public** y **region1** como región.
3. Seleccione las IP privadas de la **zona 1** y de la **zona 2** de la **región 1**.
4. Cree una nueva agrupación de fondo de VSI que actúen como iguales para compartir el tráfico direccionado a la agrupación. Establezca los parámetros con los valores siguientes y pulse **crear**.
	- **Nombre**:  region1-pool
	- **Protocolo**: HTTP
	- **Método**: Round robin
	- **Retención de sesiones**: None
	- **Vía de acceso de comprobación de estado**: /
	- **Protocolo de estado**: HTTP
	- **Intervalo (seg)**: 15
	- **Tiempo de espera excedido (seg)**: 2
	- **Número máximo de reintentos**: 2
5. Pulse **Adjuntar** para añadir instancias de servidor a la agrupación de region1-pool
   - Seleccione la IP privada de **vpc-region1-zone1-subnet**, seleccione la instancia que ha creado y establezca 80 como el puerto.
   - Pulse **Añadir** y esta vez seleccione la IP privada de **vpc-region1-zone2-subnet**, seleccione la instancia y establezca 80 como el puerto.
   - Pulse **Adjuntar** para finalizar la creación de una agrupación de programas de fondo.
6. Pulse **Nuevo escucha** para crear un nuevo escucha de componente frontal; un escucha es un proceso que comprueba si hay solicitudes de conexión.
   - **Protocolo**: HTTP
   - **Puerto**: 80
   - **Agrupación de fondo**: region1-pool
   - **Número máximo de conexiones**: deje este campo en blanco y pulse **crear**.
7. Pulse **Crear equilibrador de carga** para suministrar un equilibrador de carga.

### Prueba de los equilibradores de carga

1. Espere hasta que el estado del equilibrador de carga sea **Activado**.
2. Abra la **Dirección** en un navegador web.
3. Renueve la página varias veces y observe que el equilibrador de carga accede a distintos servidores en cada renovación.
4. **Guarde** la dirección para futuras referencias.

Si se fija, las solicitudes no están cifradas y solo dan soporte a HTTP. Configurará un certificado SSL y habilitará HTTPS en la sección siguiente.

**REPITA** los pasos 1 a 7 anteriores en la **región 2**.

## Protección del tráfico dentro de la VPC con HTTPS
{: #secure_https}

Antes de añadir un escucha HTTPS, debe generar un certificado SSL, verificar la autenticidad de su dominio personalizado, buscar un lugar en el que alojar el certificado y correlacionarlo con el servicio de infraestructura.

### Suministro de un servicio de CIS y configuración del dominio personalizado.

En esta sección, creará el servicio IBM Cloud Internet Services (CIS), configurará un dominio personalizado apuntando a los servidores de nombres de CIS y luego configurará un equilibrador de carga global.

1. Vaya a [Servicios de Internet](https://{DomainName}/catalog/services/internet-services) en el catálogo de {{site.data.keyword.Bluemix_notm}}.
2. Establezca el nombre de servicio y pulse **Crear** para crear una instancia del servicio. Puede utilizar cualquier plan de precios para esta guía de aprendizaje.
3. Cuando se haya suministrado la instancia de servicio, establezca el nombre de dominio pulsando **Empecemos** y pulse **Añadir dominio**.
4. Pulse **Paso siguiente**. Cuando se hayan asignado los servidores de nombres, configure el registrador o el proveedor de nombres de dominio para utilizar los servidores de nombres que se muestran.
5. Después de configurar el registrador o el proveedor de DNS, los cambios pueden tardar hasta 24 en entrar en vigor.

   Cuando el estado del dominio en la página Visión general pase de *Pendiente* a *Activo*, puede utilizar el mandato `dig <YOUR_DOMAIN_NAME> ns` para verificar que los servidores de nombres han entrado en vigor.
   {:tip}

Debería obtener un certificado SSL para el dominio y el subdominio que tiene previsto utilizar con el equilibrador de carga global. Suponiendo que el dominio es mydomain.com, el equilibrador de carga global se puede alojar en `lb.mydomain.com`. Será necesario emitir el certificado para lb.mydomain.com.

Puede conseguir certificados SSL gratuitos en [Cifremos](https://letsencrypt.org/). Durante el proceso, es posible que tenga que configurar un registro DNS de tipo TXT en la interfaz DNS de Cloud Internet Services para probar que es el propietario del dominio.
{:tip}

Cuando haya obtenido el certificado SSL y la clave privada para el dominio, asegúrese de convertirlos al formato [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail).

1. Para convertir un certificado en formato PEM:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
2. Para convertir una clave privada al formato PEM:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Importación del certificado y autorización del servicio de equilibrador de carga

Puede gestionar los certificados SSL a través de IBM Certificate Manager.

1. Cree una instancia de [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts) en una ubicación soportada.
2. En el panel de control del servicio, utilice **Importar certificado**:
   * Establezca como **Nombre** el subdominio y el dominio personalizados; por ejemplo, *lb.mydomain.com*.
   * Examine el **Archivo de certificado** en formato PEM.
   * Examine el **Archivo de clave privada** en formato PEM.
   * **Importe**.
3. Cree una autorización que proporcione a la instancia de servicio del equilibrador de carga acceso a la instancia del gestor de certificados que contiene el certificado SSL. Puede gestionar autorizaciones de este tipo a través de [Autorizaciones de identidad y acceso](https://{DomainName}/iam#/authorizations).
  - Pulse **Crear** y seleccione **Servicio de infraestructura** como servicio de origen
  - **Equilibrador de carga para VPC** como tipo de recurso
  - **Gestor de certificados** como servicio de destino
  - Asigne el rol de acceso al servicio de **Escritor**.
  - Para crear un equilibrador de carga debe otorgar la autorización Todas las instancias de recurso a la instancia de recurso de origen. La instancia de servicio de destino puede ser **Todas las instancias** o puede ser una instancia de recurso de gestor de certificados específica.

### Creación de un escucha HTTPS

Vaya a los [equilibradores de carga](https://{DomainName}/vpc/network/loadBalancers)

1. Seleccione **vpc-lb-region1**
2. En **Escuchas frontales**, pulse **Nuevo escucha**

   -  **Protocolo**: HTTPS
   -  **Puerto**: 443
   -  **Agrupación de fondo**: POOL en la misma región
   -  Seleccione el certificado SSL para **lb.YOUR-DOMAIN-NAME**

3. Pulse **Crear** para configurar un escucha HTTPS

**REPITA** estos pasos en el equilibrador de carga de la **región 2**.

## Configuración de un equilibrador de carga global
{: #global-load-balancer}

En esta sección, configurará un equilibrador de carga global (GLB) que distribuya el tráfico de entrada a los equilibradores de carga locales configurados en distintas regiones de {{site.data.keyword.Bluemix_notm}}.

### Distribución del tráfico entre regiones con un equilibrador de carga global
Abra el servicio CIS que ha creado navegando a la [Lista de recursos](https://{DomainName}/resources) bajo los servicios.

1. Vaya a **Equilibradores de carga globales** en **Fiabilidad** y pulse **crear equilibrador de carga**.
2. Especifique **lb.YOUR-DOMAIN-NAME** como nombre de host y TTL como 60 segundos.
3. Pulse **Añadir agrupación** para definir una agrupación de origen predeterminada
   - **Nombre**: lb-region1
   - **Comprobación de estado**: CREE UNA NUEVA COMPROBACIÓN DE ESTADO
     - **Tipo de supervisor**: HTTP
     - **Vía de acceso**: /
     - **Puerto**: 80
   - **Región de comprobación de estado**: Eastern North America
   - **orígenes**
     - **nombre**: region1
     - **dirección**: DIRECCIÓN DEL EQUILIBRADOR DE CARGA LOCAL DE LA **REGIÓN 1**
     - **peso**: 1
     - Pulse **Añadir**

4. **AÑADA** otra **agrupación de origen** que apunte al equilibrador de carga de la **región 2** en la región **Western Europe** y pulse **Suministrar 1 recurso** para suministrar el equilibrador de carga global.

Espere hasta que el estado de comprobación de **estado** indique **Correcto**. Abra el enlace **lb.YOUR-DOMAIN-NAME** en el navegador que elija para ver el equilibrador de carga global en acción.

### Prueba de migración tras error
Ya debería haber visto que la mayoría de veces accede a los servidores de la **región 1**, ya que tienen asignado un peso mayor que los servidores de la **región 2**. Vamos a introducir un error de comprobación de estado en la agrupación de origen de la **región 1**.

1. Vaya a [instancias de servidor virtual](https://{DomainName}/vpc/compute/vs).
2. Pulse los **puntos suspensivos (...)** que hay junto a los servidores que se ejecutan en la **zona 1** de la **región 1** y pulse **Detener**.
3. **REPITA** este proceso para los servidores que se ejecutan en la **zona 2** de la **región 1**.
4. Vuelva al GLB bajo el servicio CIS y espere hasta que el estado sea **Crítico**.
5. Ahora, cuando renueve el URL del dominio, siempre debería acceder a los servidores de la **región 2**.

No olvide **iniciar** los servidores de la zona 1 y de la zona 2 de la región 1
{:tip}

## Eliminación de recursos
{: #removeresources}

- Elimine el equilibrador de carga global, las agrupaciones de origen y las comprobaciones de estado bajo el servicio CIS
- Elimine los certificados en el servicio gestor de certificados.
- Elimine los equilibradores de carga, las VSI, las subredes y las VPC.
- En la [Lista de recursos](https://{DomainName}/resources), suprima los servicios utilizados en esta guía de aprendizaje.


## Contenido relacionado
{: #related}

* [Utilización de equilibradores de carga en VPC de IBM Cloud](/docs/infrastructure/vpc-network?topic=vpc-network---beta-using-load-balancers-in-ibm-cloud-vpc#--beta-using-load-balancers-in-ibm-cloud-vpc)
