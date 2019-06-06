---
copyright:
  years: 2017, 2019
lastupdated: "2019-04-15"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Traslado de una app basada en VM a Kubernetes
{: #vm-to-containers-and-kubernetes}

En esta guía de aprendizaje se muestra el proceso de trasladar una app basada en VM a un clúster de Kubernetes mediante {{site.data.keyword.containershort_notm}}. [{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) combina Docker y
Kubernetes para ofrecer herramientas potentes, una interfaz intuitiva para el usuario y funciones integradas de seguridad e identificación para automatizar el despliegue, operación, escalado y supervisión de apps contenerizadas sobre un clúster de hosts de cálculo.
{: shortdesc}

Las lecciones de esta guía de aprendizaje incluyen conceptos sobre cómo tomar una app existente, contener la app y desplegarla en un clúster de Kubernetes. Para contenerizar la app basada en VM, puede elegir entre las opciones siguientes.

1. Identifique los componentes de una app monolítica grande que se pueda separar en su propio microservicio. Puede contenerizar estos microservicios y desplegarlos en un clúster de Kubernetes.
2. Contenerice toda la app y despliéguela en un clúster de Kubernetes.

En función del tipo de app que tenga, los pasos para migrar la app pueden variar. Puede utilizar esta guía de aprendizaje para conocer ver los pasos generales que debe seguir y las cosas que debe tener en cuenta antes de migrar la app.

## Objetivos
{: #objectives}

- Comprender cómo se identifican microservicios en una app basada en VM y aprender a correlacionar componentes entre VM y Kubernetes.
- Aprender a contenerizar una app basada en VM.
- Aprender a desplegar el contenedor en un clúster de Kubernetes en {{site.data.keyword.containershort_notm}}.
- Poner en práctica todo lo aprendido y ejecutar la app **JPetStore** en el clúster.

## Servicios utilizados
{: #products}

En esta guía de aprendizaje se utilizan los siguientes servicios de {{site.data.keyword.Bluemix_notm}}:

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**Atención:** esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{:#architecture}

### Arquitectura de app tradicional con máquinas virtuales (VM)

En el diagrama siguiente se muestra un ejemplo de una arquitectura de app tradicional basada en máquinas virtuales.

<p style="text-align: center;">
![Diagrama de la arquitectura](images/solution30/traditional_architecture.png)

</p>

1. El usuario envía una solicitud al punto final público de la app Java. El punto final público se representa mediante un servicio de equilibrador de carga que equilibra la carga del tráfico de red entrante entre las instancias disponibles del servidor de app.
2. El equilibrador de carga selecciona una de las instancias de servidor de app que esté en buen estado que se ejecutan en una máquina virtual y reenvía la solicitud.
3. El servidor de app almacena datos de la app en una base de datos MySQL que se ejecuta en una VM. Las instancias de servidor de app alojan la app Java y se ejecutan en una VM. Los archivos de la app, como por ejemplo el código de la app, los archivos de configuración y las dependencias, se almacenan en la máquina virtual.

### Arquitectura contenerizada

En el diagrama siguiente se muestra un ejemplo de una arquitectura de contenedor moderna que se ejecuta en un clúster de Kubernetes.

<p style="text-align: center;">
![Diagrama de la arquitectura](images/solution30/modern_architecture.png)
</p>

1. El usuario envía una solicitud al punto final público de la app Java. El punto final público está representado por un equilibrador de carga de aplicación (ALB) Ingress que equilibra la carga del tráfico de red de entrada entre las pods de app del clúster. El ALB es una colección de reglas que permiten el tráfico de red de entrada en una app expuesta públicamente.
2. El ALB reenvía la solicitud a uno de las pods de la app disponibles en el clúster. Las pods de la app se ejecutan en nodos trabajadores que pueden ser una máquina virtual o física.
3. Los pods de la app almacenan datos en volúmenes persistentes. Los volúmenes persistentes se pueden utilizar para compartir datos entre instancias de la app o nodos trabajadores.
4. Los pods de la app guardan los datos en un servicio de base de datos de {{site.data.keyword.Bluemix_notm}}. Puede ejecutar su propia base de datos dentro del clúster de Kubernetes, pero una base de datos como servicio (DBasS) gestionada suele ser más fácil de configurar y proporciona copias de seguridad integradas y funciones de escalado. Encontrará diversos tipos de bases de datos en el [catálogo de IBM Cloud](https://{DomainName}/catalog/?category=data).

### VM, contenedores y Kubernetes

{{site.data.keyword.containershort_notm}} proporciona la posibilidad de ejecutar apps en contenedores en clústeres de Kubernetes y proporciona las herramientas y funciones siguientes:

- Una experiencia de usuario intuitiva y herramientas potentes
- Aislamiento y seguridad integrados para facilitar la entrega rápida de aplicaciones seguras
- Servicios de nube que incluyen prestaciones cognitivas de IBM® Watson™
- Posibilidad de gestionar recursos de clúster dedicado tanto para aplicaciones sin estado como para cargas de trabajo con estado

#### Máquinas virtuales frente a contenedores

Las apps tradicionales de **VM** se ejecutan en hardware nativo. Generalmente una sola app no utiliza todos los recursos de un solo host de cálculo. Muchas organizaciones intentan ejecutar varias apps en un solo host de cálculo para no malgastar recursos. Podría ejecutar varias copias de la misma app, pero, para proporcionar aislamiento, puede utilizar máquinas virtuales para ejecutar varias instancias de las apps (VM) en el mismo hardware. Estas VM tienen pilas de sistema operativo completas que las hacen relativamente grandes e ineficientes debido a la duplicación tanto en tiempo de ejecución como en disco.

Los **contenedores** son una forma estándar de empaquetar apps y todas sus dependencias para poder moverlas entre entornos sin complicaciones. A diferencia de las máquinas virtuales, los contenedores no incorporan el sistema operativo. Solo el código de app, el tiempo de ejecución, las herramientas del sistema, las bibliotecas y los valores se empaquetan dentro de los contenedores. Los contenedores son más ligeros, portátiles y eficientes que una máquina virtual.

Además, los contenedores le permiten compartir el sistema operativo del host. Esto reduce la duplicación al tiempo que proporciona aislamiento. Los contenedores también le permiten descartar los archivos innecesarios, como bibliotecas del sistema y binarios, para ahorrar espacio y reducir la superficie de ataque. Encontrará más información sobre máquinas virtuales y contenedores [aquí](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html).

#### Orquestación de Kubernetes

[Kubernetes](http://kubernetes.io/) es un orquestador de contenedores que ayuda a gestionar el ciclo de vida de las apps contenerizadas en un clúster de nodos trabajadores. Es posible que sus apps necesiten muchos otros recursos para ejecutarse, como por ejemplo volúmenes, redes y secretos, que le ayudarán a conectarse a otros servicios de nube y a claves seguras. Kubernetes le ayuda a añadir estos recursos a su app. El paradigma clave de Kubernetes es su modelo declarativo. El usuario proporciona el estado deseado y Kubernetes intenta ajustarse al estado descrito y luego lo mantiene.

Este [taller que puede seguir a su ritmo](https://github.com/IBM/kube101/blob/master/workshop/README.md) le ayudará en su primera experiencia práctica con Kubernetes. Consulte también la página de la documentación sobre [conceptos](https://kubernetes.io/docs/concepts/) de Kubernetes para obtener más información.

### Lo que IBM está haciendo por usted

Si utiliza clústeres de Kubernetes con {{site.data.keyword.containerlong_notm}}, obtiene las siguientes ventajas:

- Varios centros de datos donde puede desplegar sus clústeres.
- Soporte para las opciones de red de entrada y de equilibrador de carga.
- Soporte dinámico de volúmenes persistentes.
- Nodos maestros de Kubernetes gestionados por IBM de alta disponibilidad.

## Dimensionamiento de los clústeres
{: #sizing_clusters}

A medida que diseña la arquitectura del clúster, desea equilibrar los costes frente a disponibilidad, fiabilidad, complejidad y capacidad de recuperación. Los clústeres de Kubernetes de {{site.data.keyword.containerlong_notm}} proporcionan opciones arquitectónicas basadas en las necesidades de sus apps. Con una pequeña planificación, puede sacar el máximo provecho de los recursos de la nube sin planificar arquitecturas excesivas ni incurrir en un coste excesivo. Incluso si su planificación peca de sobrestimación o de subestimación, puede escalar el clúster con nodos trabajadores adicionales o con nodos trabajadores de mayor tamaño.

Para ejecutar una app de producción en la nube mediante Kubernetes, tenga en cuenta los elementos siguientes:

1. ¿Espera tráfico procedente de una ubicación geográfica específica? Si es así, seleccione la ubicación que tenga más cerca físicamente para obtener el mejor rendimiento.
2. ¿Cuántas réplicas de su clúster desea obtener una mayor disponibilidad? Un buen punto de partida podría ser tres clústeres: uno para desarrollo, uno para pruebas y otro para producción. Consulte la guía de soluciones sobre [prácticas recomendadas para organizar usuarios, equipos y aplicaciones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments) para crear varios entornos.
3. ¿Qué [hardware](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes) necesita para los nodos trabajadores? ¿Máquinas virtuales o servidores nativos?
4. ¿Cuántos nodos trabajadores necesita? Esto depende en gran medida de la escala de las apps; cuantos más nodos tenga, más resistente será su app.
5. ¿Cuántas réplicas debería tener para conseguir una alta disponibilidad? Despliegue clústeres de réplicas en varias ubicaciones para aumentar la disponibilidad de la app y para proteger la app frente a un error en la ubicación.
6. ¿Cuál es el conjunto mínimo de recursos que necesita la app para iniciarse? Se recomienda probar la app para ver la cantidad de memoria y de CPU que necesita para ejecutarse. El nodo trabajador debe tener suficientes recursos para desplegar e iniciar la app. Luego asegúrese de establecer las cuotas de recursos como parte de las especificaciones de pod. Este valor es el que utiliza Kubernetes para seleccionar (o planificar) un nodo trabajador de modo que tenga suficiente capacidad para dar soporte a la solicitud. Realice una estimación de la cantidad de pods que se ejecutarán en el nodo trabajador y de los requisitos de recursos para dichos pods. Como mínimo, el nodo trabajador debe ser lo suficientemente grande como para dar soporte a un pod para la app.
7. ¿Cuándo se debe aumentar el número de nodos trabajadores? Puede supervisar el uso del clúster y aumentar el número de nodos cuando sea necesario. Consulte esta guía de aprendizaje para comprender cómo se [analizar los registros y se supervisa el estado de las aplicaciones de Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana).
8. ¿Necesita almacenamiento redundante y fiable? En caso afirmativo, cree una reclamación de volumen persistente para el almacenamiento NFS o enlace un servicio de base de datos de IBM Cloud a su pod.

En relación con este punto, supongamos que desea ejecutar una aplicación web de producción en la nube y espera una carga de tráfico entre media y alta. Vamos a explorar qué recursos necesitará:

1. Configure tres clústeres: uno para desarrollo, uno para pruebas y otro para producción.
2. Los clústeres de desarrollo y de pruebas se pueden iniciar con la opción mínima de RAM y de CPU (es decir, 2 CPU, 4 GB de RAM y un nodo trabajador para cada clúster).
3. En el caso del clúster de producción, es posible que desee disponer de más recursos para obtener un mejor rendimiento, alta disponibilidad y resistencia. Podríamos elegir una opción de servidor dedicado o incluso nativo y tener al menos 4 CPU, 16 GB de RAM y dos nodos trabajadores.

## Decisión sobre qué opción de base de datos utilizar
{: #database_options}

Con Kubernetes, tiene dos opciones para manejar bases de datos:

1. Puede ejecutar la base de datos dentro del clúster de Kubernetes; para ello tendría que crear un microservicio para ejecutar la base de datos. Si se utiliza el ejemplo de base de datos MySQL, debe hacer lo siguiente:
   - Cree un Dockerfile MySQL; consulte un ejemplo de [Dockerfile MySQL](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile) aquí.
   - Necesitaría utilizar secretos para almacenar la credencial de la base de datos. Consulte [aquí](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret) un ejemplo.
   - Necesitaría un archivo deployment.yaml con la configuración de la base de datos para desplegarse en Kubernetes. Consulte [aquí](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml) un ejemplo.
2. La segunda opción sería utilizar la opción de base de datos como servicio (DBasS) gestionada. Esta opción suele ser más fácil de configurar y proporciona copias de seguridad y escalado incorporadas. Encontrará diversos tipos de bases de datos en el [catálogo de IBM Cloud](https://{DomainName}/catalog/?category=data). Para utilizar esta opción, tendría que hacer lo siguiente:
   - Cree una base de datos como servicio (DBasS) gestionada desde el [catálogo de IBM Cloud](https://{DomainName}/catalog/?category=data).
   - Almacene las credenciales de la base de datos dentro de un secreto. Encontrará más información sobre los secretos en la sección sobre "Almacenamiento de credenciales en secretos de Kubernetes".
   - Utilice la base de datos como servicio (DBasS) en la aplicación.

## Decisión sobre dónde almacenar los archivos de la aplicación
{: #decide_where_to_store_data}

{{site.data.keyword.containershort_notm}} proporciona varias opciones para almacenar y compartir datos entre los pods. No todas las opciones de almacenamiento ofrecen el mismo nivel de persistencia y de disponibilidad en situaciones de desastre.
{: shortdesc}

### Almacenamiento de datos no persistente

Los contenedores y pods son, por diseño, efímeros y pueden fallar inesperadamente. Puede almacenar datos en el sistema de archivos local de un contenedor. Los datos que hay dentro de un contenedor no se pueden compartir con otros contenedores o pods y se pierden cuando el contenedor se bloquea o se elimina.

### Más información sobre cómo crear almacenamiento de datos persistente para la app

Puede guardar de forma permanente datos de la app y datos del contenedor en [almacenamiento de archivos NFS](https://www.ibm.com/cloud/file-storage/details) o en [almacenamiento en bloque](https://www.ibm.com/cloud/block-storage) utilizando los volúmenes persistentes de Kubernetes nativos.
{: shortdesc}

Para suministrar almacenamiento en bloque o almacenamiento de archivos NFS, debe solicitar almacenamiento para el pod mediante la creación de una reclamación de volumen persistente (PVC). En la PVC, puede elegir entre las clases de almacenamiento predefinido que definen el tipo de almacenamiento, el tamaño de almacenamiento en gigabytes, IOPS, la política de retención de datos y los permisos de lectura y escritura correspondientes al almacenamiento. Una PVC suministra de forma dinámica un volumen persistente (PV) que representa un dispositivo de almacenamiento real en {{site.data.keyword.Bluemix_notm}}. Puede montar la PVC en su pod para leer y escribir en el PV. Los datos que se almacenan en los PV están disponibles, incluso si el contenedor se bloquea o se replanifica el pod. IBM coloca el almacenamiento de archivos NFS y el almacenamiento en bloque que dan soporte al PV para ofrecer una alta disponibilidad de los datos.

Para obtener información sobre cómo crear una PVC, siga los pasos de la [documentación del almacenamiento de {{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage).

### Más información sobre cómo mover los datos existentes al almacenamiento persistente

Para copiar los datos de la máquina local en el almacenamiento persistente, debe montar la PVC en un pod. A continuación, puede copiar los datos de la máquina local en el volumen persistente en el pod.
{: shortdesc}

1. Para copiar datos, primero tiene que crear una configuración que se parezca a la siguiente:

   ```bash
   kind: Pod
   apiVersion: v1
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
          claimName: mypvc
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/mnt/data"
             name: task-pv-storage
   ```
   {: codeblock}

2. A continuación, para copiar los datos de la máquina local en el pod, utilice un mandato como este:
   ```
    kubectl cp <local_filepath>/<filename> <namespace>/<pod>:<pod_filepath>
   ```
   {: pre}

3. Copie los datos de un pod en el clúster en la máquina local:
   ```
   kubectl cp <namespace>/<pod>:<pod_filepath>/<filename> <local_filepath>/<filename>
   ```
   {: pre}


### Configuración de copias de seguridad para el almacenamiento persistente

Las comparticiones de archivos y el almacenamiento en bloque se suministran en la misma ubicación que su clúster. IBM aloja el almacenamiento propiamente dicho en servidores en clúster para proporcionar alta disponibilidad. Sin embargo, no se realizan automáticamente copias de seguridad de las comparticiones de archivo ni del almacenamiento en bloque, y pueden no resultar accesibles si falla toda la ubicación. Para evitar que los datos se pierdan o se dañen, puede configurar copias de seguridad periódicas que puede utilizar para restaurar los datos cuando sea necesario.

Para obtener más información, consulte las opciones de [copia de seguridad y restauración](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) de almacenamiento en bloque y de almacenamiento de archivos NFS.

## Preparación del código
{: #prepare_code}

### Aplicación de los principios de 12 factores

La [app de doce factores (twelve-factor app)](https://12factor.net/) es una metodología para crear apps nativas en la nube. Cuando quiera contenerizar una app, mover dicha app a la nube y orqueste la app con Kubernetes; es importante entender y aplicar algunos de estos principios. Algunos de estos principios son necesarios en {{site.data.keyword.Bluemix_notm}}.
{: shortdesc}

Estos son algunos de los principios clave necesarios:

- **Código base**: se realiza un seguimiento de todo el código fuente y de todos los archivos de configuración dentro de un sistema de control de versiones (por ejemplo, un repositorio GIT), lo cual es necesario si se utiliza un conducto DevOps para el despliegue.
- **Creación, release, ejecución**: la app de 12 factores utiliza una separación estricta entre las etapas de creación, release y ejecución. Esto se puede automatizar con un conducto de entrega DevOps integrado para compilar y probar la app antes de desplegarla en el clúster. Consulte la [guía de aprendizaje sobre despliegue continuo en Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) para aprender a configurar una integración continua y un conducto de entrega. Describe las etapas de configuración de control de origen, compilación, prueba y despliegue y muestra cómo añadir integraciones, como escáneres de seguridad, notificaciones y análisis.
- **Configuración**: toda la información sobre configuración se almacena en variables de entorno. No se codifica ninguna credencial de servicio dentro del código de la app. Para almacenar credenciales, puede utilizar secretos de Kubernetes. A continuación encontrará más información sobre las credenciales.

### Almacenamiento de credenciales en secretos de Kubernetes
{: secrets}

Nunca resulta recomendable almacenar credenciales dentro del código de la app. En su lugar, Kubernetes proporciona los denominados **["secretos"](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)**, que contienen información confidencial, como contraseñas, señales OAuth o claves ssh. Los secretos de Kubernetes están cifrados de forma predeterminada, lo que hace que los secretos constituyan una opción más segura y flexible para almacenar datos confidenciales que almacenar estos datos en una definición de `pod` o en una imagen de docker.

Una forma de utilizar secretos en Kubernetes es la siguiente:

1. Cree un archivo y almacene en el mismo las credenciales de servicio.
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api_key>
   }
   ```
   {: codeblock}

2. A continuación, cree un secreto de Kubernetes ejecutando el mandato siguiente y verifique que el secreto mediante el mandato `kubectl get secrets` después de ejecutar el mandato siguiente:

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## Contenerización de la app
{: #build_docker_images}

Para contenerizar la app, debe crear una imagen de Docker.
{: shortdesc}

Una imagen se crea a partir de un [Dockerfile](https://docs.docker.com/engine/reference/builder/#usage), que es un archivo que contiene instrucciones y mandatos para crear la imagen. Un Dockerfile podría hacer referencia a los artefactos de compilación en sus instrucciones que se almacenan por separado, como por ejemplo una app, la configuración de la app y sus dependencias.

Para crear su propio Dockerfile para la app existente, puede utilizar los mandatos siguientes:

- FROM - elegir una imagen padre para definir el tiempo de ejecución del contenedor.
- ADD/COPY - copiar el contenido de un directorio en el contenedor.
- WORKDIR - definir el directorio de trabajo dentro del contenedor.
- RUN - instalar los paquetes de software que necesitan las apps durante el tiempo de ejecución.
- EXPOSE - hacer que un puerto esté disponible fuera del contenedor.
- ENV NAME - definir variables de entorno.
- CMD - definir mandatos que se ejecutan cuando se inicia el contenedor.

Las imágenes se almacenan normalmente en un registro que puede ser accesible para el público (registro público) o configurado con acceso limitado para un pequeño grupo de usuarios (registro privado). Los registros públicos, como por ejemplo Docker Hub, se pueden utilizar para empezar a trabajar con Docker y Kubernetes para crear la primera app contenerizada de un clúster. Pero, cuando se trata de apps de empresa, utilice un registro privado, como el que se proporciona en {{site.data.keyword.registrylong_notm}}, para proteger las imágenes de ser utilizadas y cambiadas por usuarios no autorizados.

Para contenerizar una app y almacenarla en {{site.data.keyword.registrylong_notm}}:

1. Tendría que crear un Dockerfile; a continuación se muestra un ejemplo de Dockerfile.
   ```
   # Crear JPetStore war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # Utilizar la imagen base WebSphere Liberty del almacén de Docker
   FROM websphere-liberty:latest

   # Copiar war de la etapa de compilación y server.xml en la imagen
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. Una vez que se ha creado un Dockerfile, deberá crear la imagen del contenedor y enviarla a {{site.data.keyword.registrylong_notm}}. Puede crear un contenedor con un mandato como el siguiente:
   ```
   ibmcloud cr build -t <image_name> <directory_of_Dockerfile>
   ```
   {: pre}

## Despliegue de la app en un clúster de Kubernetes
{: #deploy_to_kubernetes}

Una vez que se ha creado una imagen de contenedor y se ha enviado a la nube, debe desplegarla en el clúster de Kubernetes. Para ello debe crear un archivo deployment.yaml.
{: shortdesc}

### Más información sobre cómo crear un archivo yaml de despliegue de Kubernetes

Para crear archivos deployment.yaml de Kubernetes, tiene que hacer algo parecido a esto:

1. Cree un archivo deployment.yaml; a continuación encontrará un ejemplo de archivo [YAML de despliegue](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml).

2. En el archivo deployment.yaml, puede definir [cuotas de recursos](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/) para los contenedores a fin de especificar la cantidad de CPU y de memoria que necesita cada contenedor para iniciarse correctamente. Si los contenedores tienen cuotas de recursos especificadas, el planificador de Kubernetes puede tomar mejores decisiones acerca del nodo trabajador en el que colocar sus pods.

3. A continuación, puede utilizar los mandatos siguientes para crear y ver el despliegue y los servicios creados:

   ```
   kubectl create -f <filepath/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## Resumen
{: #summary}

En esta guía de aprendizaje, ha aprendido lo siguiente:

- Las diferencias entre las VM, los contenedores y Kubernetes.
- Cómo definir clústeres para distintos tipos de entorno (desarrollo, prueba y producción).
- Cómo manejar el almacenamiento de datos y la importancia del almacenamiento de datos persistente.
- Cómo aplicar los principios de 12 factores a su app y cómo utilizar secretos para credenciales en Kubernetes.
- Cómo crear imágenes de docker y enviarlas a {{site.data.keyword.registrylong_notm}}.
- Cómo crear archivos de despliegue de Kubernetes y desplegar la imagen de Docker en Kubernetes.

## Puesta en práctica de todo lo aprendido, ejecución de la app JPetStore en el clúster
{: #runthejpetstore}

Para poner en práctica todo lo que ha aprendido, siga la [demostración](https://github.com/ibm-cloud/ModernizeDemo/) para ejecutar la app **JPetStore** en el clúster y aplicar los conceptos aprendidos. La app JPetStore tiene algunas funciones ampliadas para permitirle ampliar una app en Kubernetes mediante servicios de IBM Watson que se ejecutan como un microservicio aparte.

## Contenido relacionado
{: #related}

- [Cómo empezar](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/) con Kubernetes y {{site.data.keyword.containershort_notm}}.
- Labs de {{site.data.keyword.containershort_notm}} sobre [GitHub](https://github.com/IBM/container-service-getting-started-wt).
- [Documentación](http://kubernetes.io/) principal de Kubernetes.
- [Almacenamiento persistente](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning) en {{site.data.keyword.containershort_notm}}.
- [Guía de soluciones de prácticas recomendadas](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications) para organizar usuarios, equipos y apps.
- [Análisis de registros y supervisión del estado de las aplicaciones con LogDNA y Sysdig](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis).
- Configuración de [integración continua y conducto de entrega](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) para apps contenerizadas que se ejecutan en Kubernetes.
- Despliegue del clúster de producción [en varias ubicaciones](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp).
- Utilización de [varios clústeres en diversas ubicaciones](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations) para conseguir una alta disponibilidad.
