---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Combinación de método sin servidor y Cloud Foundry para el análisis y la recuperación de datos
{: #serverless-github-traffic-analytics}
En esta guía de aprendizaje, creará una aplicación para recopilar automáticamente las estadísticas de tráfico de GitHub para los repositorios y proporcionará la base para el análisis del tráfico. GitHub solo proporciona acceso a los datos del tráfico de los últimos 14 días. Si desea analizar estadísticas correspondientes a un periodo de tiempo más largo, debe descargar y almacenar los datos usted mismo. En esta guía de aprendizaje, desplegará una acción sin servidor para recuperar los datos de tráfico y almacenarlos en una base de datos SQL. Además, se utiliza una app Cloud Foundry para gestionar los repositorios y proporcionar acceso a las estadísticas para el análisis de los datos. La app y la acción sin servidor que se describen en esta guía de aprendizaje implementan una solución con capacidad de varios arrendatarios con la característica inicial establecida para dar soporte a la modalidad de un solo arrendatario.

![](images/solution24-github-traffic-analytics/Architecture.png)

## Objetivos

* Despliegue de una app de base de datos Python con soporte de varios arrendatarios y acceso seguro
* Integración del ID de la app como proveedor de autenticación basado en OpenID Connect
* Configuración de la recopilación automatizada y sin servidor de estadísticas de tráfico de GitHub
* Integración de {{site.data.keyword.dynamdashbemb_short}} para el análisis gráfico del tráfico

## Productos
En esta guía de aprendizaje se utilizan los productos siguientes:
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Antes de empezar
{: #prereqs}

Para completar esta guía de aprendizaje necesita la última versión de la [CLI de IBM Cloud](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) y tener instalado el [plugin](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) de {{site.data.keyword.openwhisk_short}}.

## Configuración del servicio y del entorno (shell)
En esta sección, se configuran los servicios necesarios y se prepara el entorno. Todo esto se puede realizar desde el entorno de shell.

1. Clone el [repositorio GitHub](https://github.com/IBM-Cloud/github-traffic-stats) y vaya al directorio clonado y a su subdirectorio **backend**:
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. Utilice el mandato `ibmcloud login` para iniciar una sesión de forma interactiva en {{site.data.keyword.Bluemix_short}}. Puede volver a confirmar los detalles con el mandato `ibmcloud target`. Debe tener una organización y un espacio definidos.

3. Cree una instancia de {{site.data.keyword.dashdbshort}} con el plan de **Entrada** y llámela **ghstatsDB**:
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. Para acceder al servicio de base de datos desde {{site.data.keyword.openwhisk_short}} más adelante, necesita la autorización. Por lo tanto, cree las credenciales de servicio y etiquételas como **ghstatskey**:   
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. Cree una instancia del servicio {{site.data.keyword.appid_short}}. Utilice **ghstatsAppID** como nombre y el plan **Por niveles**.
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
   A partir de aquí, cree un alias de esa nueva instancia de servicio en el espacio de Cloud Foundry. Sustituya **YOURSPACE** por el espacio en el que está realizando el despliegue.
   ```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. Cree una instancia del servicio {{site.data.keyword.dynamdashbemb_short}} con el plan **lite**.
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
   De nuevo, cree un alias de dicha instancia de servicio nueva y sustituya **YOURSPACE**.
   ```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. En el directorio **backend**, envíe la aplicación a IBM Cloud. El mandato utiliza una ruta aleatoria para la aplicación.
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
   Espera a que finalice
el despliegue. Los archivos de la aplicación se cargan, se crea el entorno de tiempo de ejecución y los servicios se enlazan a la aplicación. La información del servicio se toman del archivo `manifest.yml`. Es necesario actualizar dicho archivo si ha utilizado otros nombres de servicio. Una vez que el proceso finaliza correctamente, se visualiza el URI de la aplicación.

   El mandato anterior utiliza una ruta aleatoria, pero exclusiva para la aplicación. Si desea elegir una usted mismo, añádala como parámetro adicional al mandato, por ejemplo, `ibmcloud cf push your-app-name`. También puede editar el archivo `manifest.yml`, cambiar el valor de **name** y cambiar **random-route** de **true** a **false**.
   {:tip}

## Configuración del ID de app y de GitHub (navegador)
Los pasos siguientes se llevan a cabo mediante el navegador de Internet. En primer lugar, configure {{site.data.keyword.appid_short}} de modo que utilice Cloud Directory y trabaje con la app Python. A continuación, cree una señal de acceso de GitHub. Se necesita para que la función desplegada recupere los datos de tráfico.

1. En la [Lista de recursos de {{site.data.keyword.Bluemix_short}}](https://{DomainName}/resources), abra la visión general de los servicios. Localice la instancia del servicio {{site.data.keyword.appid_short}} en la sección **Servicios**. Pulse la entrada para abrir los detalles.
2. En el panel de control del servicio, pulse **Gestionar** en **Proveedores de identidades** en el menú del lado izquierdo. Aparecerá una lista de los proveedores de identidades disponibles, como Facebook, Google, SAML 2.0 Federation y Cloud Directory. Cambie el valor de Cloud Directory por **On** y deje todos los demás proveedores en **Off**.
   
   Es posible que desee configurar la [Autenticación de varios factores (MFA)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) y reglas de contraseña avanzadas. Estos temas no quedan cubiertos en esta guía de aprendizaje.
   {:tip}

3. En la parte inferior de esta página se encuentra la lista de los URL de redirección. Especifique el **URL** de la aplicación + /redirect_uri. Por ejemplo, `https://github-traffic-stats-random-word.mybluemix.net/redirect_uri`.

   Para probar la app localmente, el URL de redirección es `http://0.0.0.0:5000/redirect_uri`. Puede configurar varios URL de redirección.
   {:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. En el menú de la izquierda, pulse **Usuarios**. Se abre la lista de usuarios en el directorio de Cloud. Pulse el botón **Añadir usuario** para añadirse como primer usuario. Ahora ha terminado de configurar el servicio {{site.data.keyword.appid_short}}.
5. En el navegador, visite [Github.com](https://github.com/settings/tokens) y vaya a **Valores -> Valores del desarrollador -> Señales de acceso personal**. Pulse el botón **Generar nueva señal**. Escriba **GHStats Tutorial** como **Descripción de la señal**. Luego habilite **public_repo** bajo la categoría **repo** y **read:org** bajo **admin:org**. Ahora, en la parte inferior de esta página, pulse **Generar señal**. La nueva señal de acceso se visualiza en la página siguiente. La necesita durante la siguiente configuración de la aplicación.
![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## Configuración y prueba de la app Python
Después de la preparación, debe configurar y probar la app. La app se ha escrito en Python mediante la popular infraestructura [Flask](http://flask.pocoo.org/). Se pueden añadir y eliminar repositorios de la colección de estadísticas. Se accede a los datos sobre el tráfico en una vista de tabla.

1. En un navegador, abra el URI de la app desplegada. Debería ver una página de bienvenida.    ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. En el navegador, añada `/admin/initialize-app` al URI y acceda a la página. Se utiliza para inicializar la aplicación y sus datos. Pulse el botón **Iniciar inicialización**. Esto le llevará a una página de configuración protegida con contraseña. La dirección de correo electrónico con la que se inicia la sesión se toma como identificación para el administrador del sistema. Utilice la dirección de correo electrónico y la contraseña que ha configurado anteriormente.

3. En la página de configuración, escriba un nombre (se utiliza para el saludo), su nombre de usuario de GitHub y la señal de acceso que ha generado antes. Pulse **Inicializar**. Esto crea las tablas de base de datos e inserta algunos valores de configuración. Por último, crea registros de base de datos para el administrador del sistema y para un arrendatario.    ![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. Una vez realizado, se le lleva a la lista de repositorios gestionados. Ahora puede añadir repositorios proporcionando el nombre de la cuenta de GitHub o la organización y el nombre del repositorio. Después de especificar los datos, pulse **Añadir repositorio**. El repositorio, junto con un identificador que se acaba de asignar, deberían aparecer en la tabla. Puede eliminar repositorios del sistema especificando su ID y pulsando **Suprimir repositorio**. ![](images/solution24-github-traffic-analytics/RepositoryList.png)

## Despliegue de Cloud Function y del desencadenante
Con la app de gestión configurada, despliegue una acción, un desencadenante y una regla para conectarlos para {{site.data.keyword.openwhisk_short}}. Estos objetos se utilizan para recopilar automáticamente los datos sobre el tráfico de GitHub según la planificación especificada. La acción conecta con la base de datos, itera sobre todos los arrendatarios y sus repositorios y obtiene la vista y los datos de clonación correspondientes a cada repositorio. Estas estadísticas se fusionan en la base de datos.

1. Vaya al directorio **functions**.
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. Cree una nueva acción **collectStats**. Utiliza un [entorno Python 3](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments) que ya incluye el controlador de base de datos necesario. El código fuente de la acción se suministra en el archivo `ghstats.zip`.
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   Si modifica el código fuente de la acción (`__main__.py`), puede volver a empaquetar el archivo zip con `zip -r ghstats.zip  __main__.py github.py`. Consulte el archivo `setup.sh` para ver detalles.
   {:tip}
3. Enlace la acción con el servicio de base de datos. Utilice la instancia y la clave de servicio que ha creado durante la configuración del entorno.
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. Cree un desencadenante basado en el [paquete de alarmas](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm). Da soporte a diferentes formas de especificar la alarma. Utilice un estilo de tipo [cron](https://en.wikipedia.org/wiki/Cron). Entre el 21 de abril y el 21 de diciembre, el desencadenante se activa cada día a las 6 AM UTC. Asegúrate de definir una fecha de inicio futura.
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  Puede cambiar el desencadenante de una programación diaria a una planificación semanal aplicando `"0 6 * * 0"`. Esto activaría el desencadenante cada domingo a las 6 AM.
  {:tip}
5. Por último, cree una regla **myStatsRule** que conecte el desencadenante **myDaily** con la acción **collectStats**. Ahora, el desencadenante hace que la acción se ejecute según la planificación especificada en el paso anterior.
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. Invoque la acción para realizar una ejecución de prueba inicial. El valor **repoCount** devuelto debe reflejar el número de repositorios que ha configurado anteriormente.
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
   La salida se parecerá a la siguiente:
   ```
   {
       "repoCount": 18
   }
   ```
7. En la ventana del navegador con la página de la app, ahora puede visitar el tráfico del repositorio. De forma predeterminada, se muestran 10 entradas. Puede cambiar este valor por otro. También se pueden clasificar las columnas de la tabla o se puede utilizar el recuadro de búsqueda para filtrar repositorios específicos. Podría especificar una fecha y un nombre de organización y luego clasificar por el valor viewcount para obtener una lista de las clasificaciones más altas de un día determinado.
   ![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## Conclusiones
En esta guía de aprendizaje, ha desplegado una acción sin servidor y un desencadenante y una regla relacionados. Esto permite recuperar automáticamente los datos sobre el tráfico de los repositorios GitHub. La información sobre esos repositorios, incluida la señal de acceso específica del arrendatario, se almacena en una base de datos de SQL ({{site.data.keyword.dashdbshort}}). La app de Cloud Foundry utiliza esta base de datos para gestionar usuarios y repositorios y para presentar estadísticas sobre tráfico en el portal de la app. Los usuarios pueden ver las estadísticas sobre el tráfico en tablas en las que se pueden realizar búsquedas o las pueden visualizar en un panel de control incorporado (servicio {{site.data.keyword.dynamdashbemb_short}}, ver imagen siguiente). También se puede descargar la lista de repositorios y los datos sobre el tráfico como archivos CSV.

La app de Cloud Foundry gestiona el acceso a través de un cliente de OpenID Connect que se conecta a {{site.data.keyword.appid_short}}. ![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## Limpieza
Para limpiar los recursos utilizados para esta guía de aprendizaje, puede suprimir los servicios relacionados y la app, así como la acción, el desencadenante y la regla, en el orden inverso al que se han creado:

1. Suprima la regla de {{site.data.keyword.openwhisk_short}}, el desencadenante y la acción.
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. Suprima la app Python y sus servicios.
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## Ampliación de la guía de aprendizaje
¿Desea ampliar o modificar esta guía de aprendizaje? Estas son algunas de las ideas:
* Amplíe la app para el soporte de varios arrendatarios.
* Integre un diagrama para los datos.
* Utilice proveedores de identidades sociales.
* Añada un selector de fechas a la página de estadísticas para filtrar los datos mostrados.
* Utilice una página de inicio de sesión personalizada para {{site.data.keyword.appid_short}}.
* Explore las relaciones de codificación social entre los desarrolladores que utilizan [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Contenido relacionado
Estos son algunos enlaces con información adicional sobre los temas tratados en esta guía de aprendizaje.

Documentación y SDK:
* [Documentación de {{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Documentación: [IBM Knowledge Center de {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Documentación de {{site.data.keyword.appid_short}}](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [Tiempo de ejecución Python en IBM Cloud](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
