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


# Aplicación web moderna utilizando la pila MEAN
{: #mean-stack}

Esta guía de aprendizaje le guía en el proceso de creación de una aplicación web utilizando la popular pila MEAN. Se compone de una infraestructura web **M**ongo, **E**xpress, **A**ngular y de un tiempo de ejecución Node.js. Aprenderá a ejecutar un iniciador MEAN localmente, a crear y utilizar una base de datos como servicio (DBasS) gestionada, a desplegar la app en {{site.data.keyword.cloud_notm}} y a supervisar la aplicación.  

## Objetivos

{: #objectives}

- Creación y ejecución de una app Node.js de inicio localmente.
- Creación de una base de datos como servicio (DBasS) gestionada.
- Despliegue la app Node.js en la nube.
- Escalado de los recursos de MongoDB.
- Supervisión del rendimiento de la aplicación.

## Servicios utilizados

{: #products}

En esta guía de aprendizaje se utilizan los siguientes servicios de {{site.data.keyword.Bluemix_notm}}:

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**Atención:** esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura

{:#architecture}

<p style="text-align: center;">

![Diagrama de la arquitectura](images/solution7/Architecture.png)</p>

1. El usuario accede a la aplicación mediante un navegador web.
2. La app Node.js accede a la base de datos {{site.data.keyword.composeForMongoDB}} para captar datos.

## Antes de empezar

{: #prereqs}

1. [Instale Git](https://git-scm.com/)
2. [Instale la CLI de {{site.data.keyword.Bluemix_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)


Para desarrollar y ejecutar la aplicación localmente:
1. [Instale Node.js y NPM](https://nodejs.org/)
2. [Instale y ejecute MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/)

## Ejecución de la app MEAN localmente

{: #runapplocally}

En esta sección, ejecutará una base de datos MongoDB local, clonará un código de ejemplo MEAN y ejecutará la aplicación de forma local para utilizar la base de datos MongoDB local.

{: shortdesc}

1. Siga [estas](https://docs.mongodb.com/manual/administration/install-community/) instrucciones para instalar y ejecutar la base de datos MongoDB de forma local. Una vez finalizada la instalación, utilice el mandato siguiente para confirmar que el servidor **mongod** se está ejecutando. Confirme que la base de datos se está ejecutando con el siguiente mandato.
  ```sh
  mongo
  ```
  {: codeblock}

2. Clone el código de inicio de MEAN.

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. Instale los paquetes necesarios.

  ```sh
  npm install
  ```
  {: codeblock}

4. Copie el archivo .env.example en .env. Edite la información necesaria; como mínimo añada su propio SESSION_SECRET.

5. Ejecute node server.js para iniciar la app
  ```sh
  node server.js
  ```
  {: codeblock}

6. Acceda a la aplicación, cree un nuevo usuario e inicie una sesión

## Creación de una instancia de base de datos MongoDB en la nube

{: #createdatabase}

En esta sección, creará una base de datos {{site.data.keyword.composeForMongoDB}} en la nube. {{site.data.keyword.composeForMongoDB}} es la base de datos como servicio que normalmente es más fácil de configurar y que proporciona funciones integradas de copia de seguridad y de escalado. Encontrará diversos tipos de bases de datos en el [catálogo de IBM Cloud](https://{DomainName}/catalog/?category=data).  Para crear {{site.data.keyword.composeForMongoDB}}, siga los pasos que se indican a continuación.

{: shortdesc}

1. Inicie una sesión en la cuenta de {{site.data.keyword.cloud_notm}} mediante la línea de mandatos y seleccione como destino su cuenta de {{site.data.keyword.cloud_notm}}. 

  ```sh
  ibmcloud login
   ibmcloud target --cf
  ```
  {: codeblock}

  Encontrará más mandatos de la CLI [aquí.](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)

2. Cree una instancia de {{site.data.keyword.composeForMongoDB}}. Esto también se puede hacer mediante la [interfaz de usuario de la consola](https://{DomainName}/catalog/services/compose-for-mongodb). El nombre de servicio debe ser **media-starter-mongodb** ya que la aplicación está configurada para buscar este servicio con este nombre.

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## Despliegue de la app en la nube

{: #deployapp}

En esta sección, desplegará la app node.js en {{site.data.keyword.cloud_notm}} que ha utilizado la base de datos MongoDB gestionada. El código fuente contiene un archivo [**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml) que se ha configurado para que utilice el servicio "mongodb" creado anteriormente. La aplicación utiliza la variable de entorno VCAP_SERVICES para acceder a las credenciales de la base de datos de Compose MongoDB. Esto se puede ver en el [archivo server.js](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js). 

{: shortdesc}

1. Envíe el código a la nube.

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. Una vez enviado el código, debería poder ver la app en su navegador. Se ha generado un nombre de host aleatorio parecido al siguiente: `https://mean-random-name.mybluemix.net`. Puede obtener el URL de la aplicación en el panel de control de la consola o desde la línea de mandatos.![App activa](images/solution7/live-app.png)


## Escalado de los recursos de la base de datos MongoDB
{: #scaledatabase}

Si el servicio necesita más almacenamiento, o si desea reducir la cantidad de almacenamiento asignado a su servicio, puede hacerlo mediante el escalado de recursos.

{: shortdesc}

1. Mediante el **panel de control** de la consola, vaya a la sección **conexiones** y pulse la base de datos de la **instancia de MongoDB**.
2. En el panel **Detalles del despliegue**, pulse la opción **Escalar recursos**.
  ![](images/solution7/mongodb-scale-show.png)
3. Ajuste el **graduador** para aumentar o reducir el almacenamiento asignado al servicio de la base de datos {{site.data.keyword.composeForMongoDB}}.
4. Pulse **Despliegue de escalado** para activar el escalado y volver a la visión general del panel de control. Aparecerá un mensaje 'Escalado iniciado' en la parte superior de la página que le indica que el proceso de escalado está en curso. ![](images/solution7/scaling-in-progress.png)


## Supervisión del rendimiento de la aplicación
{: #monitorapplication}

Para comprobar el estado de la aplicación, puede utilizar el servicio de supervisión de disponibilidad incorporado. El servicio de supervisión de disponibilidad se conecta automáticamente a las aplicaciones de la nube. El servicio de supervisión de disponibilidad ejecuta pruebas sintéticas desde ubicaciones de todo el mundo y de forma continuada para detectar y solucionar de forma proactiva problemas de rendimiento antes de que afecten a sus visitantes. Siga los pasos que se indican a continuación para acceder al panel de control de supervisión.

{: shortdesc}

1. Mediante el panel de control de la consola, en la aplicación, seleccione el separador **Supervisión**.
2. Pulse **Ver todas las pruebas** para ver las pruebas.
   ![](images/solution7/alert_frequency.png)


## Contenido relacionado

{: #related}

- Configuración del control de origen y [entrega continua](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops).
- Protección de aplicaciones web en [varias ubicaciones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp).
- Creación, protección y gestión de las [API REST](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis).
