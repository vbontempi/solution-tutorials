---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Uso de servidores virtuales para crear una app web altamente disponible y escalable
{: #highly-available-and-scalable-web-application}

La adición de más servidores a una aplicación es un patrón común para manejar la carga adicional. Otro aspecto clave para aumentar la disponibilidad y la resistencia de una aplicación es desplegar la aplicación en varias zonas o ubicaciones mediante la duplicación de datos y el equilibrio de carga.

Esta guía de aprendizaje le muestra un escenario con la creación de:

- Dos servidores de aplicaciones web en Dallas.
- Cloud Load Balancer, para equilibrar el tráfico entre dos servidores de una ubicación.
- Un servidor de bases de datos MySQL.
- Un almacén de archivos duradero para almacenar archivos de aplicación y copias de seguridad.
- La configuración de la segunda ubicación con la misma configuración que la primera y la adición de Cloud Internet Services para que dirijan el tráfico a la ubicación en buen estado si falla una copia.

## Objetivos
{: #objectives}

* Creación de {{site.data.keyword.virtualmachinesshort}} para instalar PHP y MySQL
* Utilización de {{site.data.keyword.filestorage_short}} para conservar archivos de la aplicación y copias de seguridad de la base de datos
* Suministro de {{site.data.keyword.loadbalancer_short}} para distribuir solicitudes a los servidores de aplicaciones
* Ampliación de la solución mediante la adición de una segunda ubicación para mejorar la resistencia y aumentar la disponibilidad

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/load-balancer-group)
* [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)
* [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

Se trata de una aplicación frontal PHP simple (un blog de Wordpress) con una base de datos MySQL. Hay carios servidores frontales que manejan las solicitudes.

<p style="text-align: center;">
  ![Diagrama de la arquitectura](images/solution14/Architecture.png)
</p>

1. El usuario se conecta a la aplicación.
2. {{site.data.keyword.loadbalancer_short}} selecciona uno de los servidores en buen estado para que gestione la solicitud.
3. El servidor elegido accede a los archivos de la aplicación almacenados en un almacenamiento de archivos compartidos.
4. El servidor también extrae información de la base de datos y finalmente muestra la página al usuario.
5. A intervalos regulares se realiza una copia de seguridad del contenido de la base de datos. Hay un servidor de bases de datos de reserva disponible por si falla el maestro.

## Antes de empezar
{: #prereqs}

### Configuración del acceso VPN

En esta guía de aprendizaje, el equilibrador de carga es la puerta principal para los usuarios de la aplicación. No es necesario que {{site.data.keyword.virtualmachinesshort}} esté visible en Internet pública. Se puede suministrar solo con una dirección IP privada, ya que utilizará su conexión VPN para trabajar en los servidores.

1. [Asegúrese de que el acceso VPN esté habilitado](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Debe ser un **Usuario maestro** para poder habilitar el acceso VPN; si no es así, póngase en contacto con el usuario maestro para obtener acceso.
     {:tip}
2. Obtenga las credenciales de acceso de VPN seleccionando su usuario en la [Lista de usuarios](https://{DomainName}/iam#/users).
3. Inicie una sesión en la VPN mediante [la interfaz web](https://www.softlayer.com/VPN-Access) o utilice un cliente VPN para [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) o [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

Puede saltarse este paso y hacer que todos los servidores estén visibles en Internet pública (aunque mantenerlos privados proporciona un nivel adicional de seguridad). Para hacerlos públicos, seleccione **Enlace ascendente de red pública y privada** cuando suministre {{site.data.keyword.virtualmachinesshort}}.
{: tip}

### Comprobación de los permisos de la cuenta

Póngase en contacto con el usuario maestro de la infraestructura para obtener los permisos siguientes:
- **Red** para que pueda crear {{site.data.keyword.virtualmachinesshort}} con **Enlace ascendente de red pública y privada** (este permiso no es necesario si utiliza la VPN para conectarse a los servidores)

## Suministro de un servidor para la base de datos
{: #database_server}

En esta sección, configurará un servidor para que actúe como base de datos maestra.

1. Vaya al catálogo en la consola de {{site.data.keyword.Bluemix}} y seleccione [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) en la sección Infraestructura.
2. Seleccione **Servidor virtual público** y pulse **Crear**.
3. Configure el servidor con lo siguiente:
   - Como **Nombre**, defina **db1**
   - Seleccione la ubicación en la que suministrar el servidor. **Todos los demás servidores y recursos creados en esta guía de aprendizaje se tendrán que crear en la misma ubicación.**
   - Seleccione la imagen **Ubuntu Minimal**
   - Conserve el tipo de cálculo predeterminado. La guía de aprendizaje se ha probado con el tipo más pequeño, pero debería funcionar con cualquier tipo.
   - En **Discos de almacenamiento conectados**, seleccione el disco de arranque de 25 GB.
   - En **Interfaz de red**, seleccione la opción **100 Mbps de enlace ascendente de red privada**.

     Si no ha configurado el acceso VPN, seleccione la opción **100 Mbps de enlace ascendente de red pública y privada**.
     {: tip}
   - Revise las otras opciones de configuración y pulse **Suministrar** para suministrar el servidor.

      ![Configurar servidor virtual](images/solution14/db-server.png)

   Nota: el proceso de suministro puede tardar entre 2 y 5 minutos hasta que el servidor esté listo para su uso. Una vez que se haya creado el servidor, encontrará las credenciales del servidor en la página de detalles del servidor bajo **Dispositivos > Lista de dispositivos**. Para ejecutar SSH en el servidor, necesita la dirección IP privada o pública del servidor, el nombre de usuario y la contraseña (pulse la flecha que hay junto al nombre del dispositivo).
   {: tip}

## Instalación y configuración de MySQL
{: #mysql}

El servidor no viene con una base de datos. En esta sección, instalará MySQL en el servidor.

### Instalación de MySQL

1. Conéctese al servidor mediante SSH:
   ```sh
   ssh root@<Private-OR-Public-IP-Address>
   ```

   Recuerde conectarse al cliente de VPN con la [dirección de sitio](https://www.softlayer.com/VPN-Access) correcta en función de la **Ubicación** del servidor virtual.
   {:tip}
2. Instale MySQL:
   ```sh
   apt-get update
   apt-get -y install mysql-server
   ```

   Puede que se le solicite una contraseña. Lea las instrucciones que se muestran en la consola.
   {:tip}
3. Ejecute el script siguiente como ayuda para proteger la base de datos MySQL:
   ```sh
   mysql_secure_installation
   ```

   Puede que se le soliciten un par de opciones. Elija cuidadosamente en función de sus necesidades.
   {:tip}

### Creación de una base de datos para la aplicación

1. Inicie una sesión en MySQL y cree una base de datos llamada `wordpress`:
   ```sh
   mysql -u root -p
   CREATE DATABASE wordpress;
   ```

2. Otorgue el acceso a la base de datos, sustituya database-username y database-password por los valores que ha definido anteriormente.

   ```
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.* TO database-username@'%' IDENTIFIED BY 'database-password';
   ```
   ```
   FLUSH PRIVILEGES;
   ```

3. Para comprobar si la base de datos se ha creado, utilice:

   ```sh
   show databases;
   ```

4. Para salir de la base de datos, escriba:

   ```sh
   exit
   ```

5. Anote el nombre de la base de datos, el usuario y la contraseña. Necesitará esta información cuando configure los servidores de la aplicación.

### Cómo hacer que el servidor MySQL esté visible para otros servidores de la red

De forma predeterminada, MySQL solo escucha en la interfaz local. Los servidores de la aplicación se tendrán que conectar a la base de datos, de modo que la configuración de MySQL se tiene que modificar para que escuche en las interfaces de la red privada.

1. Edite el archivo my.cnf con `nano /etc/mysql/my.cnf` y añada estas líneas:
   ```
   [mysqld]
   bind-address    = 0.0.0.0
   ```

2. Salga y guarde el archivo con Ctrl+X.

3. Reinicie MySQL:

   ```sh
   systemctl restart mysql
   ```

4. Confirme que MySQL está a la escucha en todas las interfaces ejecutando el mandato siguiente:
   ```sh
   netstat --listen --numeric-ports | grep 3306
   ```

## Creación de un almacenamiento de archivos para copias de seguridad de base de datos
{: #database_backup}

Hay muchas maneras de hacer copias de seguridad y de almacenar cuando se trata de MySQL. En esta guía de aprendizaje se utiliza una entrada crontab para hacer un vuelvo del contenido de la base de datos en disco. Los archivos de copia de seguridad se almacenarán en un almacenamiento de archivos. Obviamente, este es un mecanismo de copia de seguridad simple. Si tiene intención de gestionar su propio servidor de bases de datos MySQL en un entorno de producción, deseará [implementar una de las estrategias de copia de seguridad que se describen en la documentación de MySQL](https://dev.mysql.com/doc/refman/5.7/en/backup-and-recovery.html).

### Creación del almacenamiento de archivos
{: #create_for_backup}

1. Vaya al catálogo en la consola de {{site.data.keyword.Bluemix}} y seleccione [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
2. Pulse **Crear**
3. Configure el servicio con lo siguiente:
   - Establezca el **Tipo de almacenamiento** en **Resistencia**
   - Seleccione la **Ubicación** en la que ha creado el servidor de bases de datos
   - Seleccionar un método de facturación
   - En **Paquetes de almacenamiento**, seleccione **0,25 IOPS/GB**
   - En **Tamaño de almacenamiento**, seleccione **20 GB**
   - Mantenga el **Tamaño de espacio de instantánea** en **0 GB**
   - Pulse continuar para crear el servicio.

### Autorización al servidor de bases de datos para que utilice el almacenamiento de archivos

Para que un servidor virtual pueda montar un almacenamiento de archivos, debe tener autorización para ello.

1. Seleccione el almacenamiento de archivos que acaba de crear en la [lista de elementos existentes](https://{DomainName}/classic/storage/file).
2. En **Hosts autorizados**, pulse **Autorizar host** y seleccione el servidor virtual (base de datos) (elija **Dispositivos** > Servidor virtual como Tipo de dispositivo > Escriba el nombre del servidor).

### Montaje del almacenamiento de archivos para copias de seguridad de base de datos

El almacenamiento de archivos se puede montar como una unidad NFS en el servidor virtual.

1. Instale las bibliotecas del cliente NFS:
   ```sh
   apt-get -y install nfs-common
   ```

2. Cree un archivo llamado `/etc/systemd/system/mnt-database.mount` con el mandato siguiente
   ```bash
   touch /etc/systemd/system/mnt-database.mount
   ```

3. Edite el archivo mnt-database.mount de la siguiente manera:
   ```
   nano /etc/systemd/system/mnt-database.mount
   ```
4. Añada el contenido a continuación al archivo mnt-database.mount y sustituya `CHANGE_ME_TO_FILE_STORAGE_STORAGE_MOUNT_POINT` de `What` por el **Punto de montaje** del almacenamiento de archivos (es decir, *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). Encontrará el URL de **Punto de montaje** bajo el servicio de almacenamiento de archivos creado.
   ```
   [Unit]
   Description = Mount for Container Storage

   [Mount]
   What=CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT
   Where=/mnt/database
   Type=nfs
   Options=vers=3,sec=sys,noauto

   [Install]
   WantedBy = multi-user.target
   ```
   Utilice Ctrl+X para guardar y salir de la ventana nano
   {: tip}

5. Cree el punto de montaje
  ```sh
  mkdir /mnt/database
  ```

6. Monte el almacenamiento
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-database.mount
   ```

7. Compruebe si el montaje se ha realizado correctamente
   ```sh
   mount
   ```
   Las últimas líneas deben mostrar el montaje del almacenamiento de archivos. Si no es así, utilice `journalctl -xe` para depurar la operación de montaje.
   {: tip}

### Configuración de una copia de seguridad a intervalos regulares

1. Cree un script de shell `/root/dbbackup.sh` (utilice `touch` y `nano`) con los mandatos siguientes sustituyendo `CHANGE_ME` por la contraseña de la base de datos que ha especificado anteriormente:
   ```sh
   #!/bin/bash
   mysqldump -u root -p CHANGE_ME --all-databases --routines | gzip > /mnt/datamysql/backup-`date '+%m-%d-%Y-%H-%M-%S'`.sql.gz
   ```
2. Asegúrese de que el archivo sea ejecutable
   ```sh
   chmod 700 /root/dbbackup.sh
   ```
3. Edite crontab
   ```sh
   crontab -e
   ```
4. Para que la copia de seguridad se realice cada día a las 23:00, defina el siguiente contenido, guarde el archivo y cierre el editor
   ```
   0 23 * * * /root/dbbackup.sh
   ```

## Suministro de dos servidores para la aplicación PHP
{: #app_servers}

En esta sección, creará dos servidores de aplicaciones web.

1. Vaya al catálogo en la consola de {{site.data.keyword.Bluemix}} y seleccione el servicio [{{site.data.keyword.virtualmachinesshort}}](https://{DomainName}/catalog/infrastructure/virtual-server-group) en la sección Infraestructura.
2. Seleccione **Servidor virtual público** y pulse **Crear**.
3. Configure el servidor con lo siguiente:
   - Como **Nombre**, defina **app1**
   - Seleccione la ubicación en la que ha suministrado el servidor de bases de datos
   - Seleccione la imagen **Ubuntu Minimal**
   - Conserve el tipo de cálculo predeterminado.
   - En **Discos de almacenamiento conectados**, seleccione 25 GB como disco de arranque.
   - En **Interfaz de red**, seleccione la opción **100 Mbps de enlace ascendente de red privada**.

     Si no ha configurado el acceso VPN, seleccione la opción **100 Mbps de enlace ascendente de red pública y privada**.
     {: tip}
   - Revise las otras opciones de configuración y pulse **Suministrar** para suministrar el servidor.
![Configurar servidor virtual](images/solution14/db-server.png)
4. Repita los pasos del 1 a 3 para suministrar otro servidor virtual denominado **app2**

## Creación de un almacenamiento de archivos para compartir archivos entre los servidores de aplicaciones
{: shared_storage}

Este almacenamiento de archivos se utiliza para compartir los archivos de aplicaciones entre los servidores *app1* y *app2*.

### Creación del almacenamiento de archivos
{: #create_for_sharing}

1. Vaya al catálogo en la consola de {{site.data.keyword.Bluemix}} y seleccione [{{site.data.keyword.filestorage_short}}](https://{DomainName}/catalog/infrastructure/file-storage)
2. Pulse **Crear**
3. Configure el servicio con lo siguiente:
   - Establezca el **Tipo de almacenamiento** en **Resistencia**
   - Seleccione la **Ubicación** en la que ha creado los servidores de aplicaciones
   - Seleccionar un método de facturación
   - En **Paquetes de almacenamiento**, seleccione **2 IOPS/GB**
   - En **Tamaño de almacenamiento**, seleccione **20 GB**
   - En **Tamaño de espacio de instantáneas**, seleccione **20 GB**
   - Pulse continuar para crear el servicio.

### Configuración de instantáneas regulares

Las [instantáneas](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) constituyen una opción cómoda para proteger sus datos sin que ello afecte al rendimiento. Además, puede duplicar las instantáneas en otro centro de datos.

1. Seleccione el almacenamiento de archivos en la [lista de elementos existentes](https://{DomainName}/classic/storage/file).
2. En **Planificaciones de instantáneas**, edite la planificación de instantáneas. La planificación se podría definir de la siguiente manera:
   1. Para añadir una instantánea cada hora, establezca el minuto en 30 y mantenga las últimas 24 instantáneas
   2. Para añadir una instantánea cada día, establezca la hora en las 23:00 y mantenga las últimas 7 instantáneas
   3. Para añadir una instantánea semanal, establezca la hora en la 1:00, mantenga las 4 últimas instantáneas y pulse Guardar.
![Instantáneas de copia de seguridad](images/solution14/snapshots.png)

### Autorización a los servidores de aplicaciones para que utilicen el almacenamiento de archivos

1. En **Hosts autorizados**, pulse **Autorizar host** para autorizar a los servidores de aplicaciones (app1 y app2) a utilizar este almacenamiento de archivos.

### Montaje del almacenamiento de archivos

Repita los pasos siguientes en cada servidor de aplicaciones (app1 y app2):

1. Instale las bibliotecas del cliente NFS
   ```sh
   apt-get update
   apt-get -y install nfs-common
   ```
2. Cree un archivo mediante `touch /etc/systemd/system/mnt-www.mount` y edítelo con `nano /etc/systemd/system/mnt-www.mount` con el siguiente contenido, sustituyendo `CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT` de `What` por el **Punto de montaje** correspondiente al almacenamiento de archivos (es decir, *fsf-lon0601a-fz.adn.networklayer.com:/IBM01SEV12345_100/data01*). Encontrará los puntos de montaje en la [lista de volúmenes de almacenamiento de archivos](https://{DomainName}/classic/storage/file)
   ```
   [Unit]
   Description = Mount for Container Storage

   [Mount]
   What=CHANGE_ME_TO_FILE_STORAGE_MOUNT_POINT
   Where=/mnt/www
   Type=nfs
   Options=vers=3,sec=sys,noauto

   [Install]
   WantedBy = multi-user.target
   ```
3. Cree el punto de montaje
   ```sh
   mkdir /mnt/www
   ```
4. Monte el almacenamiento
   ```sh
   systemctl enable --now /etc/systemd/system/mnt-www.mount
   ```
5. Compruebe si el montaje se ha realizado correctamente
   ```sh
   mount
   ```
   Las últimas líneas deben mostrar el montaje del almacenamiento de archivos. Si no es así, utilice `journalctl -xe` para depurar la operación de montaje.
   {: tip}

En principio todos los pasos relacionados con la configuración de los servidores se pueden automatizar mediante un [script de suministro](https://{DomainName}/docs/vsi?topic=virtual-servers-managing-a-provisioning-script#managing-a-provisioning-script) o mediante la [captura de una imagen](https://{DomainName}/docs/infrastructure/image-templates?topic=image-templates-getting-started#creating-an-image-template).
{: tip}

## Instalación y configuración de la aplicación PHP en los servidores de aplicaciones
{: #php_application}

En esta guía de aprendizaje se configura un blog de Wordpress. Todos los archivos de Wordpress se instalarán en el almacenamiento de archivos compartidos para que ambos servidores de aplicaciones puedan acceder a los mismos. Antes de instalar Wordpress, es necesario configurar un servidor web y un entorno de ejecución PHP.

### Instalación de nginx y PHP

Repita los pasos siguientes en cada servidor de aplicaciones:

1. Instale nginx
   ```sh
   apt-get update
   apt-get -y install nginx
   ```
2. Instale PHP y el cliente mysql
   ```sh
   apt-get -y install php-fpm php-mysql
   ```
3. Detenga el servicio PHP y nginx
   ```sh
   systemctl stop php7.2-fpm
   systemctl stop nginx
   ```
4. Sustituya el contenido utilizando `nano /etc/nginx/sites-available/default` por el siguiente:
   ```sh
   server {
          listen 80 default_server;
          listen [::]:80 default_server;

          root /mnt/www/html;

          index index.php;

          server_name _;

          location = /favicon.ico {
                  log_not_found off;
                  access_log off;
          }

          location = /robots.txt {
                  allow all;
                  log_not_found off;
                  access_log off;
          }

          location / {
                  # following https://codex.wordpress.org/Nginx
                  try_files $uri $uri/ /index.php?$args;
          }

          # pass the PHP scripts to the local FastCGI server
          location ~ \.php$ {
                  include snippets/fastcgi-php.conf;
                  fastcgi_pass unix:/run/php/php7.2-fpm.sock;
          }

          location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                  expires max;
                  log_not_found off;
          }

          # deny access to .htaccess files, if Apache's document root
          # concurs with nginx's one
          location ~ /\.ht {
                  deny all;
          }
   }
   ```
5. Cree una carpeta `html` dentro del directorio `/mnt/www` en uno de los dos servidores de apps con los siguientes mandatos
   ```sh
   cd  /mnt
   mkdir /www/html
   cd www
   mkdir html
   ```

### Instalación y configuración de WordPress

Puesto que Wordpress se instalará en el montaje del almacenamiento de archivos, sólo tiene que seguir los pasos siguientes en uno de los servidores. Vamos a seleccionar **app1**.

1. Recupere los archivos de instalación de Wordpress

   Si el servidor de aplicaciones tiene un enlace de red pública, puede descargar directamente los archivos de Wordpress desde dentro del servidor virtual:

   ```sh
   apt-get install curl
   cd /tmp
   curl -O https://wordpress.org/latest.tar.gz
   ```

   Si el servidor virtual solo tiene un enlace de red privado, tendrá que recuperar los archivos de instalación de otra máquina con acceso a Internet y copiarlos en el servidor virtual. Suponiendo que ha recuperado los archivos de instalación de Wordpress de https://wordpress.org/latest.tar.gz, puede copiarlos en el servidor virtual con `scp`:

   ```sh
   scp latest.tar.gz root@PRIVATE_IP_ADDRESS_OF_THE_SERVER:/tmp
   ```
   Sustituya `latest` por el nombre del archivo que ha descargado del sitio web de wordpress.
   {: tip}

   luego ejecute ssh sobre el servidor virtual y vaya al directorio `tmp`

   ```sh
   cd /tmp
   ```

2. Extraiga los archivos de instalación

   ```
   tar xzvf latest.tar.gz
   ```

3. Prepare los archivos de Wordpress
   ```sh
   cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
   mkdir /tmp/wordpress/wp-content/upgrade
   ```

4. Copie los archivos en el almacenamiento de archivos compartidos
   ```sh
   rsync -av -P /tmp/wordpress/. /mnt/www/html
   ```

5. Defina los permisos
   ```sh
   chown -R www-data:www-data /mnt/www/html
   find /mnt/www/html -type d -exec chmod g+s {} \;
   chmod g+w /mnt/www/html/wp-content
   chmod -R g+w /mnt/www/html/wp-content/themes
   chmod -R g+w /mnt/www/html/wp-content/plugins
   ```

6. Llame al siguiente servicio web e inyecte el resultado en `/mnt/www/html/wp-config.php` utilizando `nano`
   ```sh
   curl -s https://api.wordpress.org/secret-key/1.1/salt/
   ```

   Si el servidor virtual no tiene ningún enlace de red pública, simplemente puede abrir https://api.wordpress.org/secret-key/1.1/salt/ desde el navegador web.

7. Establezca las credenciales de la base de datos utilizando `nano /mnt/www/html/wp-config.php` y actualice las credenciales de la base de datos:

   ```
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'database-username');
   define('DB_PASSWORD', 'database-password');
   define('DB_HOST', 'database-server-ip-address');

   ```
   Wordpress está configurado. Para finalizar la instalación, tiene que acceder a la interfaz de usuario de Wordpress.

En ambos servidores de aplicaciones, inicie el servidor web y el tiempo de ejecución de PHP:
7. Para iniciar el servicio, ejecute los mandatos siguientes

   ```sh
   systemctl start php7.2-fpm
   systemctl start nginx
   ```

Acceda a la instalación de Wordpress en `http://YourAppServerIPAddress/` utilizando la dirección IP privada (si utiliza la conexión VPN) o la dirección IP pública de *app1* o *app2*.
![Configurar servidor virtual](images/solution14/wordpress.png)

Si ha configurado los servidores de aplicaciones solo con un enlace de red privada, no podrá instalar los plugins de Wordpress, los temas o las actualizaciones directamente desde la consola de administración de Wordpress. Tendrá que cargar los archivos a través de la interfaz de usuario de Wordpress.
{: tip}

## Suministro de un servidor equilibrador de carga frente a los servidores de aplicaciones
{: #load_balancer}

En este punto, tenemos dos servidores de aplicaciones con direcciones IP separadas. Incluso puede que no sean visibles en Internet pública si ha optado por suministrar solo el enlace de red privada. Al añadir un equilibrador de carga frente a estos servidores, convertirá la aplicación en pública. El equilibrador de carga también ocultará la infraestructura subyacente a los usuarios. El equilibrador de carga supervisará el estado de los servidores de aplicaciones y enviará las solicitudes entrantes a los servidores en buen estado.

1. Vaya al catálogo para crear un [{{site.data.keyword.loadbalancer_short}}](https://{DomainName}/catalog/infrastructure/ibm-cloud-load-balancer)
2. En el paso **Plan**, seleccione el mismo centro de datos que *app1* y *app2*
3. En **Valores de red**,
   1. Seleccione la subred en la que se han suministrado *app1* y *app2*
   2. Utilice la agrupación de sistemas IBM predeterminada para la IP pública del equilibrador de carga.
4. En **Basic**,
   1. Asigne un nombre al equilibrador de carga, como por ejemplo **app-lb-1**
   2. Mantenga la configuración de protocolo predeterminada: de forma predeterminada, el equilibrador de carga está configurado para HTTP.
      El protocolo SSL recibe soporte con sus propios certificados. Consulte el apartado sobre [Importación de sus certificados SSL en el equilibrador de carga](https://{DomainName}/docs/infrastructure/ssl-certificates?topic=ssl-certificates-accessing-ssl-certificates#accessing-ssl-certificates)
      {: tip}
5. En **Instancias del servidor**, añada los servidores *app1* y *app2*
6. Pulse Revisar y Crear para finalizar el asistente.

### Cambio de la configuración de Wordpress para que utilice el URL del equilibrador de carga

La configuración de Wordpress se debe cambiar para que utilice la dirección del equilibrador de carga. De hecho, Wordpress mantiene una referencia al [URL del blog e inyecta esta ubicación en las páginas](https://codex.wordpress.org/Settings_General_Screen). Si no cambia este valor, Wordpress redirigirá directamente a los usuarios a los servidores de fondo, pasando por alto el equilibrador de carga o bien no funcionará en absoluto si los servidores solo tienen una dirección IP privada.

1. Localice la dirección del equilibrador de carga en su página de detalles. Encontrará el equilibrador de carga que ha creado en [Red / Equilibrio de carga / Local](https://{DomainName}/classic/network/loadbalancing/cloud).

   También puede utilizar su propio nombre de dominio con el equilibrador de carga añadiendo un registro CNAME que apunte a la dirección del equilibrador de carga en la configuración de DNS.
   {: tip}
2. Inicie una sesión como administrador en el blog de Wordpress a través del URL de *app1* o *app2*
3. En Configuración / General, establezca la dirección de Wordpress (URL) y la dirección del sitio (URL) en la dirección del equilibrador de carga
4. Guarde los valores. Wordpress debería redirigirse a la dirección del equilibrador de carga
La dirección del equilibrador de carga puede tardar un rato en activarse debido a la propagación de DNS.
   {: tip}

### Prueba del comportamiento del equilibrador de carga

El equilibrador de carga se ha configurado para que compruebe el estado de los servidores y redirija a los usuarios solo a los servidores en buen estado. Para entender cómo funciona el equilibrador de carga, puede hacer lo siguiente

1. Revise los registros de nginx en *app1* y *app2* con:
   ```sh
   tail -f /var/log/nginx/*.log
   ```

   Ya debería ver los pings regulares procedentes del equilibrador de carga para comprobar el estado del servidor.
   {: tip}
2. Acceda a Wordpress a través de la dirección del equilibrador de carga y asegúrese de forzar una recarga de la página. Observe en los registros de nginx que tanto *app1* como *app2* ofrecen contenido para la página. El equilibrador de carga está redirigiendo el tráfico a ambos servidores tal como se esperaba.

3. Detenga nginx en *app1*
   ```sh
   systemctl nginx stop
   ```

4. Después de un breve periodo de tiempo de recarga de la página de Wordpress, observe que todas las coincidencias van a *app2*.

5. Detenga nginx en *app2*.

6. Vuelva a cargar la página de Wordpress. El equilibrador de carga devolverá un error ya que no hay ningún servidor en buen estado.

7. Reinicie nginx en *app1*
   ```sh
   systemctl nginx start
   ```

8. Cuando el equilibrador de carga detecte que *app1* está en buen estado, redirigirá el tráfico a este servidor.

## Ampliación de la solución con una segunda ubicación (opcional)
{: #secondregion}

Para aumentar la resistencia y la disponibilidad, puede ampliar la configuración de la infraestructura con una segunda ubicación y hacer que la aplicación se ejecute en dos ubicaciones.

Con un segundo despliegue de ubicación, la arquitectura tendrá el siguiente aspecto.

<p style="text-align: center;">

  ![Diagrama de la arquitectura](images/solution14/Architecture2.png)
</p>

1. Los usuarios acceden a la aplicación a través de IBM Cloud Internet Services (CIS).
2. CIS dirige el tráfico a una ubicación en buen estado.
3. Dentro de una ubicación, un equilibrador de carga redirecciona el tráfico a un servidor.
4. La aplicación accede a la base de datos.
5. La aplicación almacena y recupera activos de medios de un almacenamiento de archivos.

Para implementar esta arquitectura, tendrá que hacer lo siguiente en la ubicación dos:

- Repita todos los pasos anteriores en la nueva ubicación.
- Configure una réplica de base de datos entre los dos servidores MySQL entre ubicaciones.
- Configure IBM Cloud Internet Services de modo que distribuya el tráfico entre las ubicaciones a servidores en buen estado, tal como se describe en [esta otra guía de aprendizaje](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis).

## Eliminación de recursos
{: #removeresources}

1. Suprima el equilibrador de carga
2. Cancele *db1*, *app1* y *app2*
3. Suprima los dos servicios de almacenamiento de archivos
4. Si se ha configurado una segunda ubicación, suprima todos los recursos y Cloud Internet Services.

## Contenido relacionado
{: #related}

- El contenido estático suministrado por la aplicación puede beneficiarse de una red de entrega de contenido frente al equilibrador de carga para reducir la carga en los servidores de fondo. Consulte el tema sobre [Aceleración de la entrega de archivos estáticos mediante CDN - Almacenamiento de objetos](https://{DomainName}/docs/tutorials?topic=solution-tutorials-static-files-cdn#static-files-cdn) para ver una guía de aprendizaje en la que se implementa una red de entrega de contenido.
- En esta guía de aprendizaje se suministran dos servidores y se pueden añadir más servidores automáticamente para gestionar la carga adicional. El [escalado automático](https://{DomainName}/docs/infrastructure/SLautoscale?topic=slautoscale-about-auto-scale#about-auto-scale) le ofrece la posibilidad de automatizar el proceso de escalado manual asociado a la adición o a la eliminación de servidores virtuales para dar soporte a sus aplicaciones empresariales.
- Para aumentar la disponibilidad y las opciones de recuperación tras desastre, se pueden configurar el almacenamiento de archivos de modo que realice [instantáneas regulares automáticas](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-snapshots#working-with-snapshots) del contenido y [réplicas](https://{DomainName}/docs/infrastructure/FileStorage?topic=FileStorage-replication#working-with-replication) en otro centro de datos.
