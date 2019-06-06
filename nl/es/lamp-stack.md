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

# Aplicación web de PHP en una pila LAMP
{: #lamp-stack}

En esta guía de aprendizaje se muestra cómo crear un servidor virtual Ubuntu **L**inux con un servidor web **A**pache, una base de datos **M**ySQL y scripts **P**HP. Esta combinación de software (más comúnmente llamada una pila LAMP) es muy popular y a menudo se utiliza para suministrar sitios web y aplicaciones web. Con {{site.data.keyword.BluVirtServers}}, desplegará rápidamente
la pila LAMP con supervisión incorporada y exploración de vulnerabilidades. Para ver el servidor LAMP en acción, instalará y configurará el sistema de gestión de contenido [WordPress](https://wordpress.org/) gratuito y de código abierto.

## Objetivos

* Suministro de un servidor LAMP en cuestión de minutos
* Aplicación de la versión más reciente de Apache, MySQL y PHP
* Alojamiento de un sitio web o de un blog mediante la instalación y configuración de WordPress
* Utilización de la supervisión para detectar paradas y bajo rendimiento
* Evaluación de vulnerabilidades y protección contra el tráfico no deseado

## Servicios utilizados

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:

* [{{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura

![Diagrama de la arquitectura](images/solution4/Architecture.png)

1. El usuario final accede al servidor LAMP y a las aplicaciones mediante un navegador web

## Antes de empezar

{: #prereqs}

1. Póngase en contacto con el administrador de la infraestructura para obtener los permisos siguientes.
  * Permiso de red necesario para completar el **Enlace ascendente de red pública y privada**

### Configuración del acceso VPN

1. [Asegúrese de que el acceso VPN esté habilitado](/docs/infrastructure/iaas-vpn?topic=VPN-gettingstarted-with-virtual-private-networking#gettingstarted-with-virtual-private-networking).

     Debe ser un **Usuario maestro** para poder habilitar el acceso VPN; si no es así, póngase en contacto con el usuario maestro para obtener acceso.
     {:tip}
2. Obtenga las credenciales de acceso de VPN en [la página de usuario bajo la lista Usuarios](https://{DomainName}/iam#/users).
3. Inicie una sesión en la VPN mediante [la interfaz web](https://www.softlayer.com/VPN-Access) o utilice un cliente VPN para [Linux](/docs/infrastructure/iaas-vpn?topic=VPN-setup-ssl-vpn-connections), [macOS](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-mac-osx) o [Windows](/docs/infrastructure/iaas-vpn?topic=VPN-connect-ssl-vpn-windows7).

   Para el cliente VPN, utilice el FQDN de punto de acceso VPN de un solo centro de datos de la [página de acceso web de VPN](https://www.softlayer.com/VPN-Access), con el formato *vpn.xxxnn.softlayer.com* como dirección de pasarela.
   {:tip}

## Creación de servicios

En esta sección, suministrará un servidor virtual público con una configuración fija. {{site.data.keyword.BluVirtServers_short}} se puede desplegar en cuestión de minutos a partir de imágenes de servidor virtual en ubicaciones geográficas específicas. Los servidores virtuales suelen experimentar picos de demanda, después de los cuales se pueden suspender o apagar para que el entorno de nube se ajuste perfectamente a las necesidades de la infraestructura.

1. En el navegador, acceda a la [página del catálogo de {{site.data.keyword.BluVirtServers_short}}](https://{DomainName}/catalog/infrastructure/virtual-server-group).
2. Seleccione **Servidor virtual público** y pulse **Crear**.
3. En **Imagen**, seleccione la versión de **LAMP** más reciente bajo **Ubuntu**. A pesar de que vienen preinstalados con Apache, MySQL y PHP, reinstalará PHP y MySQL con la versión más reciente.
4. En **Interfaz de red**, seleccione la opción **Enlace ascendente de red pública y privada**.
5. Revise las otras opciones de configuración y pulse **Suministrar** para crear el servidor virtual.
![Configurar servidor virtual](images/solution4/ConfigureVirtualServer.png)

Una vez que se haya creado el servidor, verá las credenciales de inicio de sesión del servidor. Aunque se puede conectar a través de SSH utilizando la dirección IP pública del servidor, se recomienda acceder al servidor a través de la red privada e inhabilitar el acceso SSH en la red pública.

1. Siga [estos pasos](https://{DomainName}/docs/infrastructure/ssh-keys?topic=ssh-keys-restricting-ssh-access-on-a-public-network#restricting-ssh-access-on-a-public-network) para proteger la máquina virtual y para inhabilitar el acceso SSH en la red pública.
1. Con su nombre de usuario, contraseña y dirección IP privada, conéctese al servidor con SSH.
   ```sh
   sudo ssh root@<Private-IP-Address>
   ```
   {: pre}

  Encontrará la dirección IP privada y la contraseña del servidor en el panel de control.
  {:tip}

  ![Servidor virtual creado](images/solution4/VirtualServerCreated.png)

## Reinstalación de Apache, MySQL y PHP

Se aconseja actualizar periódicamente la pila LAMP con los últimos parches de seguridad y correcciones de errores. En esta sección, ejecutará mandatos para actualizar los orígenes de los paquetes Ubuntu y reinstalar Apache, MySQL y PHP con la versión más reciente. Observe el símbolo (^) al final del mandato.

```sh
sudo apt update && sudo apt install lamp-server^
```
{: pre}

Una opción alternativa consiste en actualizar todos los paquetes con `sudo apt-get update && sudo apt-get dist-upgrade`.
{:tip}

## Verificación de la instalación y de la configuración

En esta sección, verificará que Apache, MySQL y PHP están actualizados y en ejecución en la imagen de Ubuntu. También implementará los valores de seguridad recomendados para MySQL.

1. Verifique Ubuntu abriendo la dirección IP pública en el navegador. Debería ver la página de bienvenida de Ubuntu.    ![Verificar Ubuntu](images/solution4/VerifyUbuntu.png)
2. Verifique que el puerto 80 esté disponible para el tráfico web ejecutando el mandato siguiente.
   ```sh
   sudo netstat -ntlp | grep LISTEN
   ```
   {: pre}
   ![Verificar puerto](images/solution4/VerifyPort.png)
3. Revise las versiones de Apache, MySQL y PHP instaladas con los mandatos siguientes.
  ```sh
  apache2 -v
  ```
  {: pre}
  ```sh
  mysql -V
  ```
  {: pre}
  ```sh
   php -v
  ```
   {: pre}
4. Ejecute el script siguiente para proteger la base de datos MySQL.
  ```sh
  mysql_secure_installation
  ```
  {: pre}
5. Escriba la contraseña raíz de MySQL y configure los valores de seguridad para el entorno. Cuando haya terminado, salga del a solicitud indicador mysql escribiendo `\q`.
  ```sh
  mysql -u root -p
  ```
  {: pre}

  El nombre de usuario y la contraseña predeterminados de MySQL son root y root.
  {:tip}
6. Si lo desea, puede crear rápidamente una página de información de PHP con el mandato siguiente.
   ```sh
   sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
   ```
   {: pre}
7. Visualice la página de información de PHP que ha creado: abra un navegador y vaya a `http://{YourPublicIPAddress}/info.php`. Sustituya la dirección IP pública del servidor virtual. Se parecerá a la siguiente imagen.

![Info de PHP](images/solution4/PHPInfo.png)

### Instalación y configuración de WordPress

Empiece a probar su pila LAMP instalando una aplicación. En los siguientes pasos se instala la plataforma WordPress de código abierto, que se suele utilizar para crear sitios web y blogs. Para obtener más información y para ver los valores para una instalación de producción, consulte la [documentación de WordPress](https://codex.wordpress.org/Main_Page).

1. Ejecute el mandato siguiente para instalar WordPress.
   ```sh
   sudo apt install wordpress
   ```
   {: pre}
2. Configure WordPress para que utilice MySQL y PHP. Ejecute el mandato siguiente para abrir un editor de texto y crear el archivo `/etc/wordpress/config-localhost.php`.
   ```sh
   sudo sensible-editor /etc/wordpress/config-localhost.php
   ```
   {: pre}
3. Copie las líneas siguientes en el archivo, sustituyendo *yourPassword* por la contraseña de la base de datos MySQL y dejando los otros valores sin cambios. Guarde y salga del archivo con `Control+X`.
   ```php
   <?php
   define('DB_NAME', 'wordpress');
   define('DB_USER', 'wordpress');
   define('DB_PASSWORD', 'yourPassword');
   define('DB_HOST', 'localhost');
   define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
   ?>
   ```
   {: pre}
4. En un directorio de trabajo, cree un archivo de texto `wordpress.sql` para configurar la base de datos de WordPress.
   ```sh
   sudo sensible-editor wordpress.sql
   ```
   {: pre}
5. Añada los mandatos siguientes, sustituyendo la contraseña de la base de datos *yourPassword* y dejando los otros valores sin cambios. A continuación, guarde el archivo.
   ```sql
   CREATE DATABASE wordpress;
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER ON wordpress.*
   TO wordpress@localhost
   IDENTIFIED BY 'yourPassword';
   FLUSH PRIVILEGES;
   ```
   {: pre}
6. Ejecute el mandato siguiente para crear la base de datos.
   ```sh
   cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
   ```
   {: pre}
7. Cuando se complete el mandato, suprima el archivo `wordpress.sql`. Mueva la instalación de WordPress a la raíz del documento del servidor web.
   ```sh
   sudo ln -s /usr/share/wordpress /var/www/html/wordpress
   sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
   ```
   {: pre}
8. Complete la configuración de WordPress y publique en la plataforma. Abra un navegador y vaya a `http://{yourVMPublicIPAddress}/wordpress`. Sustituya la dirección IP pública de la máquina virtual. Debe parecerse a la siguiente imagen. ![Sitio WordPress en ejecución](images/solution4/WordPressSiteRunning.png)

## Configuración del dominio

Para utilizar un nombre de dominio existente con el servidor LAMP, actualice el registro A para que apunte a la dirección IP pública del servidor virtual.
Puede ver la dirección IP pública del servidor en el panel de control.

## Supervisión y uso del servidor

Para garantizar la disponibilidad del servidor y la mejor experiencia de usuario, la supervisión debe estar habilitada en cada servidor de producción. En esta sección, explorará las opciones que están disponibles para supervisar el servidor virtual y conocer el uso del servidor en cualquier momento.

### Supervisión del servidor

Hay dos tipos de supervisión básicos disponibles: SERVICE PING y SLOW PING.

* **SERVICE PING** comprueba que el tiempo de respuesta del servidor es igual o menor que 1 segundo
* **SLOW PING** comprueba que el tiempo de respuesta del servidor es igual o menor que 5 segundos

Puesto que SERVICE PING se añade de forma predeterminada, añada la supervisión SLOW PING siguiendo los pasos siguientes.

1. En el panel de control, seleccione su servidor en la lista de dispositivos y pulse el separador **Supervisión**.
  ![Supervisión Slow Ping](images/solution4/SlowPing.png)
2. Pulse **Gestionar supervisores**.
3. Añada la opción de supervisión **SLOW PING** y pulse **Añadir supervisor**. Seleccione la dirección IP pública correspondiente a la dirección IP.   ![Añadir supervisión Slow Ping](images/solution4/AddSlowPing.png)

  **Nota**: no se permiten supervisores duplicados con las mismas configuraciones. Solo se puede crear un supervisor por configuración.

Si no se recibe una respuesta en el intervalo de tiempo asignado, se envía una alerta a la dirección de correo electrónico de la cuenta de {{site.data.keyword.Bluemix_notm}}.
![Supervisión dos](images/solution4/TwoMonitoring.png)

### Uso del servidor

Seleccione el separador **Uso** para ver el uso de memoria y de CPU del servidor actual.   ![Uso del servidor](images/solution4/ServerUsage.png)

## Seguridad del servidor

{{site.data.keyword.BluVirtServers}} proporcionan varias opciones de seguridad como, por ejemplo, exploración de vulnerabilidades y cortafuegos de complementos.

### Explorador de vulnerabilidades

El explorador de vulnerabilidades explora el servidor para detectar cualquier vulnerabilidad relacionada con el servidor. Para ejecutar una exploración de vulnerabilidad en el servidor, siga los pasos que se indican a continuación.

1. En el panel de control, seleccione el servidor y pulse el separador **Seguridad**.
2. Pulse **Explorar** para iniciar la exploración.
3. Una vez completada la exploración, pulse **Exploración completa** para ver el informe de la exploración. ![Supervisión dos](images/solution4/Vulnerabilities.png)
4. Revise las vulnerabilidades detectadas.
  ![Supervisión dos](images/solution4/VulnerabilityResults.png)

### Cortafuegos

Otra forma de proteger el servidor consiste en añadir un cortafuegos. Los cortafuegos proporcionan una capa de seguridad esencial: evitan la entrada de tráfico no deseado en los servidores, lo que reduce la probabilidad de un ataque y permite dedicar los recursos del servidor a las tareas para las que se han diseñado. Las opciones de cortafuegos se suministran bajo demanda, sin interrupciones en el servicio.

Los cortafuegos están disponibles como una característica adicional para todos los servidores de la red pública de la infraestructura. Como parte del proceso del pedido, puede seleccionar un hardware específico del dispositivo o un software de cortafuegos para proporcionar protección. Como alternativa, puede desplegar dispositivos cortafuegos dedicados en el entorno y desplegar el servidor virtual en una VLAN protegida. Para obtener más información, consulte [Cortafuegos](https://{DomainName}/docs/infrastructure/hardware-firewall-dedicated?topic=hardware-firewall-dedicated-getting-started-with-hardware-firewall-dedicated#getting-started).

## Eliminación de recursos

Para eliminar el servidor virtual, siga los pasos siguientes.

1. Inicie sesión en [{{site.data.keyword.slportal}}](https://{DomainName}/classic/devices).
2. Desde el menú **Dispositivos**, seleccione **Lista de dispositivos**.
3. Pulse **Acciones** para el servidor virtual que desea eliminar y seleccione **Cancelar**.

## Contenido relacionado

* [Despliegue de una pila LAMP mediante Terraform](https://{DomainName}/docs/tutorials?topic=solution-tutorials-infrastructure-as-code-terraform#infrastructure-as-code-terraform)
