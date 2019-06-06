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

# Proceso asíncrono de datos mediante Object Storage y mensajería pub/sub
{: #pub-sub-object-storage}
En esta guía de aprendizaje, aprenderá a utilizar un servicio de mensajería basado en Apache Kafka para coordinar las cargas de trabajo de larga ejecución de las aplicaciones que se ejecutan en un clúster de Kubernetes. Este patrón se utiliza para desacoplar la aplicación, lo que permite controlar mejor el escalado y el rendimiento. Se puede utilizar {{site.data.keyword.messagehub}} para poner en cola el trabajo que se debe realizar sin que ello afecte a las aplicaciones de productor, lo que lo convierte en un sistema ideal para las tareas de larga ejecución. 

{:shortdesc}

Simulará este patrón utilizando un ejemplo de proceso de archivos. En primer lugar, cree una aplicación de interfaz de usuario que se utilizará para cargar archivos en el almacenamiento de objetos y generar mensajes que indiquen el trabajo que se debe realizar. A continuación, creará una aplicación de nodo trabajador independiente que procesará de forma asíncrona los archivos cargados por el usuario cuando reciba mensajes.

## Objetivos
{: #objectives}

* Implementación de un patrón de productor-consumidor con {{site.data.keyword.messagehub}}
* Enlace de servicios a un clúster de Kubernetes

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

En esta guía de aprendizaje, la aplicación de interfaz de usuario está escriba en Node.js y la aplicación de nodo trabajo está escrita en Java,
lo que demuestra la flexibilidad de este patrón. Aunque ambas aplicaciones se ejecutan en el mismo clúster de Kubernetes en esta guía de aprendizaje, cualquiera de las dos piezas se podría haber implementado como una aplicación de Cloud Foundry o como una función sin servidor.

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. El usuario carga el archivo utilizando la aplicación de interfaz de usuario
2. El archivo se guarda en {{site.data.keyword.cos_full_notm}}
3. Se envía un mensaje al tema {{site.data.keyword.messagehub}} para indicar que el nuevo archivo está en espera de proceso.
4. Cuando estén listos, los nodos trabajadores escuchan los mensajes y empiezan a procesar el nuevo archivo.

## Antes de empezar
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli): herramienta para instalar la CLI de {{site.data.keyword.cloud_notm}}, Kubernetes, Helm y Docker.

## Creación de un clúster de Kubernetes
{: #create_kube_cluster}

1. Cree un clúster de Kubernetes desde el [Catálogo](https://{DomainName}/containers-kubernetes/launch). En esta guía de aprendizaje lo llamaremos `mycluster`. Esta guía de aprendizaje se puede seguir con un clúster **Gratuito**.
   ![Creación de clúster de Kubernetes en IBM Cloud](images/solution25/KubernetesClusterCreation.png)
2. Compruebe el estado del **Clúster** y de los **Nodos trabajadores** y espere a que estén **listos**.

### Configuración de kubectl

En este paso, configurará kubectl de modo que apunte el clúster que acaba de crear. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) es una herramienta de línea de mandatos que se utiliza para interactuar con un clúster de Kubernetes.

1. Utilice `ibmcloud login` para iniciar la sesión de forma interactiva. Especifique la organización (org), la ubicación y el espacio bajo los que se crea el clúster. Puede volver a confirmar los detalles con el mandato `ibmcloud target`.
2. Cuando el clúster esté preparado, recupere la configuración de clúster:
   ```bash
   ibmcloud cs cluster-config <cluster-name>
   ```
   {: pre}
3. Copie y pegue el mandato **export** para establecer la variable de entorno KUBECONFIG tal como se indica. Para verificar si la variable de entorno KUBECONFIG se ha establecido correctamente o no, ejecute el mandato siguiente: `echo $KUBECONFIG`
4. Compruebe que el mandato `kubectl` esté configurado correctamente
   ```bash
   kubectl cluster-info
   ```
  {: pre}
   ![](images/solution2/kubectl_cluster-info.png)

## Creación de una instancia de {{site.data.keyword.messagehub}}
 {: #create_messagehub}

{{site.data.keyword.messagehub}} es un servicio de mensajería rápido, escalable y completamente gestionado basado en Apache Kafka, un sistema de mensajería de alto rendimiento de código abierto que proporciona una plataforma de baja latencia para manejar canales de información de datos en tiempo real.

 1. En el panel de control, pulse [**Crear recurso**](https://{DomainName}/catalog/) y seleccione [**{{site.data.keyword.messagehub}}**](https://{DomainName}/catalog/services/event-streams) en la sección Servicios de aplicación.
 2. Llame al servicio `mymessagehub` y pulse **Crear**.
 3. Especifique las credenciales de servicio a su clúster enlazando la instancia de servicio con el espacio de nombres `default` de Kubernetes.
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

El mandato cluster-service-bind crea un secreto de clúster que contiene las credenciales de la instancia de servicio en formato JSON. Utilice `kubectl get secrets` para ver el secreto generado con el nombre `binding-mymessagehub`. Consulte [Integración de servicios](https://{DomainName}/docs/containers?topic=containers-integrations#integrations) para obtener más información

{:tip}

## Creación de una instancia de Object Storage

{: #create_cos}

{{site.data.keyword.cos_full_notm}} está cifrado y distribuido entre varias ubicaciones geográficas, y se accede a través de HTTP mediante una API REST. {{site.data.keyword.cos_full_notm}} ofrece un almacenamiento en la nube flexible, eficiente en coste y escalable para datos no estructurados. Lo utilizará esto para almacenar los archivos cargados por la interfaz de usuario.

1. En el panel de control, pulse [**Crear recurso**](https://{DomainName}/catalog/) y seleccione [**{{site.data.keyword.cos_short}}**](https://{DomainName}/catalog/services/cloud-object-storage) en la sección Almacenamiento.
2. Llame al servicio `myobjectstorage` y pulse **Crear**.
3. Pulse **Crear grupo**.
4. Establezca el nombre de grupo en un nombre exclusivo, como `username-mybucket`.
5. Seleccione la resistencia **Varias regiones** y la ubicación **us-geo** y pulse **Crear**
6. Especifique las credenciales de servicio a su clúster enlazando la instancia de servicio con el espacio de nombres `default` de Kubernetes.
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## Despliegue de la aplicación de interfaz de usuario en el clúster

La aplicación de interfaz de usuario es una sencilla aplicación web Node.js Express que permite al usuario cargar archivos. Almacena los archivos en la instancia de Object Storage creada anteriormente y luego envía un mensaje al tema de {{site.data.keyword.messagehub}} "work-topic" que indica que un nuevo archivo esté listo para ser procesado.

1. Clone el repositorio de aplicaciones de ejemplo localmente y vaya a la carpeta `pubsubui`.
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. Abra `config.js` y actualice COSBucketName con el nombre del grupo.
3. Cree y despliegue la aplicación. El mandato deploy genera imágenes de docker, las envía a {{site.data.keyword.registryshort_notm}} y luego crea un despliegue de Kubernetes. Siga las instrucciones interactivas cuando despliegue la app.
```sh
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
4. Vaya a la aplicación y cargue los archivos desde la carpeta `sample-files`. Los archivos subidos se almacenarán en Object Storage y el estado será "awaiting" hasta que los procese la aplicación de trabajo. Deje abierta esta ventana del navegador.

   ![](images/solution25/files_uploaded.png)

## Despliegue de la aplicación de trabajo en el clúster

La aplicación de trabajo es una aplicación Java que escucha en el tema de {{site.data.keyword.messagehub}} Kafka "work-topic" si hay mensajes. En un mensaje nuevo, el trabajador recuperará el nombre del archivo del mensaje y, a continuación, obtendrá el contenido del archivo de Object Storage. Luego simulará el proceso del archivo y enviará otro mensaje al tema "result-work" cuando termine. La aplicación de interfaz de usuario escuchará este tema y actualizará el estado.

1. Vaya al directorio `pubsub-worker`
```sh
  cd ../pubsub-worker
```
2. Abra `resources/cos.properties` y actualice la propiedad `bucket.name` con el nombre del grupo.
2. Cree y despliegue la aplicación del nodo trabajador.
```
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
3. Una vez completado el despliegue, compruebe de nuevo la ventana del navegador con la aplicación web. Observe que el estado situado junto a cada archivo ahora es "processed".
![](images/solution25/files_processed.png)

En esta guía de aprendizaje, ha aprendido a utilizar Kafka basado en {{site.data.keyword.messagehub}} para implementar un patrón de consumidor de productor. Esto permite que la aplicación web sea rápida y descargue el proceso pesado a otras aplicaciones. Cuando hay que llevar a cabo un trabajo, el productor (la aplicación web) crea mensajes y la carga del trabajo se equilibra entre uno o varios nodos trabajadores que se suscriben a los mensajes. Ha utilizado una aplicación Java que se ejecuta en Kubernetes para gestionar el proceso, pero estas aplicaciones también pueden estar en [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing). Las aplicaciones que se ejecutan en Kubernetes son ideales para cargas de trabajo largas e intensivas, mientras que {{site.data.keyword.openwhisk_short}} se ajusta mejor a procesos de corta duración.

## Eliminación de recursos
{:removeresources}

Vaya a la [Lista de recursos](https://{DomainName}/resources/) y
1. suprima el clúster de Kubernetes `mycluster`
2. suprima {{site.data.keyword.cos_full_notm}} `myobjectstorage`
3. suprima {{site.data.keyword.messagehub}} `mymessagehub`
4. seleccione **Kubernetes** en el menú de la izquierda, pulse **Registro** y suprima los repositorios `pubsub-xxx`.

## Contenido relacionado
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [Gestión del acceso a Object Storage](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [Proceso de datos de {{site.data.keyword.messagehub}} con {{site.data.keyword.openwhisk_short}}](https://github.com/IBM/openwhisk-data-processing-message-hub)
