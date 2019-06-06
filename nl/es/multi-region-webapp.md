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

# Aplicación web segura en varias regiones
{: #multi-region-webapp}

En esta guía de aprendizaje se muestra el proceso de creación, protección, despliegue y equilibrio de la carga de una aplicación de Cloud Foundry entre varias regiones mediante un conducto de [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery).

Apps o partes de apps sufrirán caídas; esto es un hecho. Puede deberse a un problema del código, a un mantenimiento planificado que afecte a los recursos utilizados por la app o a una anomalía de hardware que desactive una zona, una ubicación o un centro de datos en el que se aloja la app. Se producirá alguno de estos problemas tiene que estar preparado. Con {{site.data.keyword.Bluemix_notm}}, puede desplegar su aplicación en [varias ubicaciones](https://{DomainName}/docs/overview?topic=overview-whatis-platform#ov_intro_reg) para aumentar la resistencia de la aplicación. Si su aplicación se ejecuta en varias ubicaciones, también puede redirigir el tráfico de los usuarios a la ubicación más cercana para reducir la latencia.

## Objetivos

* Despliegue una aplicación Cloud Foundry en varias ubicaciones con {{site.data.keyword.contdelivery_short}}.
* Correlación de un dominio personalizado con la aplicación.
* Configuración del equilibrio de carga global en la aplicación de varias ubicaciones.
* Enlace de un certificado SSL a la aplicación.
* Supervisión del rendimiento de las aplicaciones.

## Servicios utilizados

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [App {{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs) de Cloud Foundry
* [{{site.data.keyword.contdelivery_short}}](https://{DomainName}/catalog/services/continuous-delivery) for DevOps
* [Internet Services](https://{DomainName}/catalog/services/internet-services)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura

Esta guía de aprendizaje se utiliza un escenario activo/activo en el que se despliegan dos copias de la aplicación en dos ubicaciones diferentes y las dos copias responden a las solicitudes de los clientes mediante el sistema round robin. La configuración de DNS apunta automáticamente a la ubicación en buen estado si una copia falla.

<p style="text-align: center;">

   ![Arquitectura](./images/solution1/Architecture.png)
</p>

## Creación de una aplicación Node.js
{: #create}

Comience creando una aplicación de inicio Node.js que se ejecute en un entorno Cloud Foundry.

1. Pulse **[Catálogo](https://{DomainName}/catalog/)** en la consola de {{site.data.keyword.Bluemix_notm}}.
2. Pulse **Apps de Cloud Foundry** en la categoría **Plataforma** y seleccione **[{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)**.
     ![](images/solution1/SDKforNodejs.png)
3. Especifique un **nombre exclusivo** para la aplicación, que también será el nombre de host, por ejemplo: myusername-nodeapp. A continuación, pulse **Crear**.
4.  Después de que se inicie la aplicación, pulse el enlace **Visitar URL** en la página **Visión general** para ver la aplicación EN DIRECTO en un nuevo separador.

![HelloWorld](images/solution1/HelloWorld.png)

¡Buen comienzo! Tiene su propia aplicación de inicio Node.js que se ejecuta en {{site.data.keyword.Bluemix_notm}}.

A continuación, vamos a enviar el código fuente de la aplicación a un repositorio y a desplegar los cambios automáticamente.

## Configuración del control de origen y de {{site.data.keyword.contdelivery_short}}
{: #devops}

En este paso, configurará un repositorio de control de origen de git para almacenar el código y luego creará un conducto, que desplegará automáticamente los cambios en el código.

1. En el panel izquierdo de la aplicación que acaba de crear, seleccione **Visión general** y desplácese hasta localizar **{{site.data.keyword.contdelivery_short}}**. Pulse **Habilitar**.

   ![HelloWorld](images/solution1/Enable_Continuous_Delivery.png)
2. Mantenga las opciones predeterminadas y pulse **Crear**. Ahora se debería haber creado una **cadena de herramientas** predeterminada.

   ![HelloWorld](images/solution1/DevOps_Toolchain.png)
3. Seleccione el mosaico **Git** bajo **Código**. Se le dirigirá a la página del repositorio git.
4. Si aún no ha configurado claves SSH, debería ver una barra de notificación en la parte superior con instrucciones. Siga los pasos abriendo el enlace **añadir una clave SSH** en un nuevo separador o, si desea utilizar HTTPS en lugar de SSH, siga los pasos pulsando **Crear una señal de acceso personal**. Recuerde guardar la clave o la señal para consultarla en el futuro.
5. Seleccione SSH o HTTPS y copie el URL de git. Clone el origen en la máquina local.
   ```bash
   git clone <your_repo_url>
   cd <name_of_your_app>
   ```
   **Nota:** si se le solicita un nombre de usuario, especifique su nombre de usuario de git. Para la contraseña, utilice una **clave SSH** o una **señal de acceso personal** existente o la que se ha creado en el paso anterior.
6. Abra el repositorio clonado en el IDE que elija y vaya a `public/index.html`. Ahora vamos a actualizar el código. Intente cambiar "Hello World" por otra cosa.
7. Ejecute la aplicación localmente ejecutando los siguientes mandatos uno después de otro: `npm install`, `npm build`, `npm start `. Luego vaya a `localhost:<port_number>` en el navegador.
  **<port_number>** es el que se muestra en la consola.
8. Envíe el cambio al repositorio con tres pasos simples: añadir (add), confirmar (commit) y enviar (push).
   ```bash
   git add public/index.html
   git commit -m "my first changes"
   git push origin master
   ```
9. Vaya a la cadena de herramientas que ha creado anteriormente y pulse el mosaico **Conducto de entrega**.
10. Confirme que ve la etapa n **BUILD** y **DEPLOY**.   ![HelloWorld](images/solution1/DevOps_Pipeline.png)
11. Espere a que finalice la etapa **DEPLOY**.
12. Pulse el **URL** de la aplicación en el resultado de la última ejecución para ver los cambios en directo.

Siga realizando cambios en la aplicación y confirme periódicamente los cambios en el repositorio git. Si no ve que la aplicación se actualiza, consulte los registros de las etapas DEPLOY y BUILD del conducto.

## Despliegue en otra ubicación
{: #deploy_another_region}

A continuación, desplegaremos la misma aplicación en otra ubicación de {{site.data.keyword.Bluemix_notm}}. Podemos utilizar la misma cadena de herramientas, pero añadimos otra etapa DEPLOY para manejar el despliegue de la aplicación a otra ubicación.

1. Vaya a la **Visión general** de la aplicación y desplácese hasta encontrar **Ver cadena de herramientas**.
2. Seleccione **Conducto de entrega** en Entrega.
3. Pulse el icono de **engranaje** en la etapa **DEPLOY** y seleccione **Clonar etapa**.    ![HelloWorld](images/solution1/CloneStage.png)
4. Cambie el nombre de la etapa por "Deploy to UK" y seleccione **JOBS**.
5. Cambie la **Región de IBM Cloud** por **Londres - https://api.eu-gb.bluemix.net**. Cree un **espacio** si no tiene uno.
6. Cambie el **Script de despliegue** por `cf push "${CF_APP}" -d eu-gb.mybluemix.net`

   ![HelloWorld](images/solution1/DeployToUK.png)
7. Pulse **Guardar** y ejecute la nueva etapa pulsando el **botón Reproducir**.

## Registro de un dominio personalizado con IBM Cloud Internet Services

{: #domain_cis}

IBM [Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-) es una plataforma uniforme para configurar y gestionar el sistema de nombres de dominio (DNS), el equilibrio de carga global (GLB), el cortafuegos de aplicaciones web (WAF) y la protección frente a ataques de tipo denegación de servicio distribuido (DDoS) para las aplicaciones web. Proporciona un servicio de Internet rápido, muy eficiente, fiable y seguro para clientes que gestionan su negocio en IBM Cloud con tres características principales para mejorar su flujo de trabajo: seguridad, fiabilidad y rendimiento.  

Cuando despliegue una aplicación real, es probable que desee utilizar su propio dominio en lugar del dominio mybluemix.net proporcionado por IBM. En este paso, después cuando ya tenga un dominio personalizado, puede utilizar los servidores DNS que proporciona la plataforma IBM Cloud Internet Services.

1. Adquiera un dominio de un registrador, como por ejemplo [http://godaddy.com](http://godaddy.com).
2. Vaya a [Servicios de Internet](https://{DomainName}/catalog/services/internet-services) en el catálogo de {{site.data.keyword.Bluemix_notm}}.
2. Especifique un nombre de servicio y pulse **Crear** para crear una instancia del servicio.
3. Cuando se haya suministrado la instancia de servicio, defina el nombre de dominio y pulse **Añadir dominio**.
4. Cuando se hayan asignado los servidores de nombres, configure el registrador o el proveedor de nombres de dominio para utilizar los servidores de nombres que se muestran.
5. Después de configurar el registrador o el proveedor de DNS, los cambios pueden tardar hasta 24 en entrar en vigor.
  Cuando el estado del dominio en la página Visión general pase de *Pendiente* a *Activo*, puede utilizar el mandato `dig <your_domain_name> ns` para verificar que los servidores de nombres de IBM Cloud han entrado en vigor.
  {:tip}

## Adición de equilibrio de carga global a la aplicación

{: #add_glb}

En esta sección, utilizará el equilibrador de carga global (GLB) en IBM Cloud Internet Services para gestionar el tráfico entre varias ubicaciones. La GLB utiliza una agrupación de origen que permite distribuir el tráfico entre varios orígenes.

### Antes de crear un GLB, cree una comprobación de estado para el GLB.

1. En la aplicación Cloud Internet Services, vaya a **Fiabilidad** > **Equilibrador de carga global** y, en la parte inferior de la página, pulse **Crear comprobación de estado**.
2. Especifique la vía de acceso que desea supervisar, por ejemplo `/`, y seleccione un tipo (HTTP o HTTPS). Por lo general, puede crear un punto final de estado dedicado. Pulse **Suministrar 1 instancia**.    ![Comprobación de estado](images/solution1/health_check.png)

### Después de eso, cree una agrupación de origen con dos orígenes.

1. Pulse **Crear agrupación**.
2. Especifique un nombre para la agrupación, seleccione la comprobación de estado que acaba de crear y una región que esté cerca de la ubicación de la aplicación node.js.
3. Escriba un nombre para el primer origen y el nombre de host de la aplicación en Dallas `<your_app>.mybluemix.net`.
4. De forma similar, añada otro origen con la dirección de origen que apunte a la aplicación en Londres `<your_app>.eu-gb.mybluemix.net`.
5. Pulse **Suministrar 1 instancia**.
![Comprobación de estado](images/solution1/origin_pool.png)

### Creación de un equilibrador de carga global (GLB). 

1. Pulse **Crear equilibrador de carga**.
2. Especifique un nombre para el equilibrador de carga global. Este nombre también formará parte de su URL de aplicación universal (`http://<glb_name>.<your_domain_name>`), independientemente de la ubicación.
3. Pulse **Añadir agrupación** y seleccione la agrupación de origen que acaba de crear.
4. Pulse **Suministrar 1 instancia**.
![Equilibrador de carga global](images/solution1/load_balancer.png)

En esta etapa, el GLB está configurado, pero las aplicaciones Cloud Foundry aún no están preparadas para responder a las solicitudes procedentes del nombre de dominio GLB configurado. Para completar la configuración, actualizará las aplicaciones con rutas que utilicen el dominio personalizado.

## Configuración del dominio personalizado y de las rutas a la aplicación

{: #add_domain}

En este paso, correlacionará el nombre de dominio personalizado con el punto final seguro para la ubicación de {{site.data.keyword.Bluemix_notm}} en la que se ejecuta su aplicación.

1. En la barra de menús, pulse **Gestionar** y, a continuación, **Cuenta**: [Cuenta](https://{DomainName}/account).
2. En la página de la cuenta, vaya a la aplicación **Orgs de Cloud Foundry** y seleccione **Dominios** en la columna Acciones.
3. Pulse **Añadir un dominio** y especifique el nombre de dominio personalizado adquirido del registrador.
4. Seleccione la ubicación correcta y pulse **Guardar**.
5. De forma similar, añada el nombre de dominio personalizado a Londres.
6. Vuelva a la [Lista de recursos](https://{DomainName}/resources) de {{site.data.keyword.Bluemix_notm}}, vaya a **Apps de Cloud Foundry** y pulse la aplicación de Dallas; pulse **Ruta** > **Editar rutas** y pulse **Añadir ruta**.
   ![Añadir una ruta](images/solution1/ApplicationRoutes.png)
7. Especifique el nombre de host de GLB que ha configurado anteriormente en el campo **Especificar host (opcional)** y seleccione el dominio personalizado que acaba de añadir. Pulse **Guardar**.
8. Del mismo modo, configure el dominio y las rutas para la aplicación de Londres.

En este punto, puede visitar la aplicación con el URL `<glb_name>.<your_domain_name>` y el equilibrador de carga global distribuye automáticamente el tráfico para las aplicaciones de varias ubicaciones. Puede verificarlo deteniendo la aplicación en Dallas, manteniendo activa la aplicación en Londres y accediendo a la aplicación a través del equilibrador de carga global.

A pesar de que esto funciona en este momento, como hemos configurado la entrega continua en los pasos anteriores, la configuración se puede sobrescribir cuando se desencadene otra compilación. Para hacer que estos cambios sean persistentes, vuelva a las cadenas de herramientas y modifique el archivo *manifest.yml*:

1. En la [Lista de recursos](https://{DomainName}/resources) de {{site.data.keyword.Bluemix_notm}}, vaya a **Apps de Cloud Foundry** y pulse la aplicación en Dallas, vaya a la **Visión general** de la aplicación y desplácese hasta encontrar **Ver cadena de herramientas**.
2. Seleccione el mosaico Git bajo Código.
3. Seleccione *manifest.yml*.
4. Pulse **Editar** y añada rutas personalizadas. Sustituya solo las configuraciones de dominio y host originales por `Rutas`.

   ```
   applications:
- path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <glb_name>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}
  
5. Confirme los cambios y asegúrese de que las compilaciones para ambas ubicaciones se han ejecutado correctamente.  

## Alternativa: correlación del dominio personalizado con el dominio del sistema de IBM Cloud

Es posible que no desee utilizar un equilibrador de carga global para las aplicaciones de varias ubicaciones, pero tiene que correlacionar el nombre de dominio personalizado con el punto final seguro para la región de {{site.data.keyword.Bluemix_notm}} en la que se ejecuta su aplicación.

Con la aplicación Cloud Intenet Services, siga estos pasos para configurar registros `CNAME` para la aplicación:

1. En la aplicación Cloud Internet Services, vaya a **Fiabilidad** > **DNS**.
2. Seleccione **CNAME** en la lista desplegable **Tipo**, escriba un alias para la aplicación en el campo Nombre y el URL de la aplicación en el campo de nombre de dominio. La aplicación `<your_app>.mybluemix.net` de Dallas se puede correlacionar con un CNAME `<your_app>`.
3. Pulse **Añadir registro**. Coloque el conmutador PROXY en ON para mejorar la seguridad de la aplicación.
4. De forma similar, establezca el registro `CNAME` para el punto final de Londres. ![Registros CNAME](images/solution1/cnames.png)

Cuando utilice un dominio predeterminado que no sea `mybluemix.net`, como por ejemplo `cf.appdomain.cloud` o `cf.cloud.ibm.com`, asegúrese de utilizar el [dominio del sistema respectivo](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#mapcustomdomain).
{:tip}

Si utiliza otro proveedor de DNS, los pasos para configurar el registro CNAME varían en función del proveedor de DNS. Por ejemplo, si utiliza GoDaddy, seguirá las directrices de la [Ayuda sobre dominios](https://www.godaddy.com/help/add-a-cname-record-19236) de GoDaddy.

Para que se pueda acceder a sus aplicaciones Cloud Foundry a través del dominio personalizado, tendrá que añadir el dominio personalizado a la [lista de dominios de la organización de Cloud Foundry en la que se han desplegado las aplicaciones](https://{DomainName}/docs/apps?topic=creating-apps-updatingapps#updatingapps). Cuando termine, puede añadir las rutas a los manifiestos de la aplicación:

   ```
   applications:
- path: .
	  name: <your_app>
	  memory: 256M
	  instances: 1
	  routes:
	  - route: <your_app>.mybluemix.net
	  - route: <your_app>.<your_domain_name>
	  disk_quota: 1024M
   ```
   {: pre}

## Enlace de un certificado SSL a la aplicación
{: #ssl}

1. Obtenga un certificado SSL. Por ejemplo, puede adquirirlo en https://www.godaddy.com/web-security/ssl-certificate o puede generar uno gratuito en https://letsencrypt.org/.
2. Vaya a **Visión general** de la aplicación > **Rutas** > **Gestionar dominios**.
3. Pulse el botón para cargar el certificado SSL y cargue el certificado.
5. Acceda a la aplicación con https en lugar de con http.

## Supervisión del rendimiento de la aplicación
{: #monitor}

Vamos a comprobar el estado de la aplicación de varias ubicaciones.

1. En el panel de control de la aplicación, seleccione **Supervisión**.
2. Pulse **Ver todas las pruebas**
![](images/solution1/alert_frequency.png)

La supervisión de disponibilidad ejecuta pruebas sintéticas desde ubicaciones de todo el mundo, todo el tiempo para detectar y solucionar de forma proactiva problemas de rendimiento antes de que los usuarios se vean afectados. Si ha configurado una ruta personalizada para la aplicación, cambie la definición de prueba para acceder a la aplicación a través de su dominio personalizado.

## Eliminación de recursos

* Suprima la cadena de herramientas
* Suprima las dos aplicaciones de Cloud Foundry desplegadas en las dos ubicaciones
* Suprima el GLB, las agrupaciones de origen y la comprobación de estado
* Suprima la configuración de DNS
* Suprima la instancia de Internet Services

## Contenido relacionado

[Adición de una base de datos Cloudant](https://{DomainName}/docs/services/Cloudant/tutorials?topic=cloudant-creating-an-ibm-cloudant-instance-on-ibm-cloud#creating-an-ibm-cloudant-instance-on-ibm-cloud)

[Escalado automático de aplicaciones de Cloud Foundry](https://{DomainName}/docs/services/Auto-Scaling?topic=services/Auto-Scaling-get-started#get-started)

[Cloud Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
