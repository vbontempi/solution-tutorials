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

# Aplicación web escalable en Kubernetes
{: #scalable-webapp-kubernetes}

Esta guía de aprendizaje le muestra cómo diseñar una aplicación web, ejecutarla localmente en un contenedor y luego desplegarla en un clúster de Kubernetes creado con [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster). Además, aprenderá a enlazar un dominio personalizado, supervisar el estado del entorno y escalar la aplicación.
{:shortdesc}

Los contenedores son una forma estándar de empaquetar apps y todas sus dependencias para poder moverlas entre entornos sin complicaciones. A diferencia de las máquinas virtuales, los contenedores no incorporan el sistema operativo. El contenedor solo contiene código de la app, tiempo de ejecución, herramientas del sistema, bibliotecas y valores. Los contenedores son más ligeros, portátiles y eficientes que una máquina virtual.

Para los desarrolladores que desee iniciar rápidamente sus proyectos, la CLI de {{site.data.keyword.dev_cli_notm}} habilita el desarrollo y el despliegue rápido de aplicaciones mediante la generación de aplicaciones de plantilla que puede ejecutar inmediatamente o personalizar como punto de partida para sus propias soluciones. Además de generar el código de aplicación inicial, la imagen del contenedor Docker y los activos CloudFoundry, los generadores de código que utilizan la CLI de desarrollo y la consola web generan archivos para ayudar al despliegue en entornos [Kubernetes](https://kubernetes.io/). Las plantillas generan diagramas de [Helm](https://github.com/kubernetes/helm) que describen la configuración de despliegue inicial de Kubernetes de la aplicación y que se pueden ampliar fácilmente para crear despliegues de varias imágenes o más complejos, según sea necesario.

## Objetivos
{: #objectives}

* Diseño de una aplicación inicial.
* Despliegue de la aplicación en el clúster de Kubernetes.
* Enlace de un dominio personalizado.
* Supervisión de los registros y del estado del clúster.
* Escalado de pods de Kubernetes.

## Servicios utilizados
{: #services}

En esta guía de aprendizaje se utilizan los siguientes entornos de ejecución y servicios:
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

Esta guía de aprendizaje puede incurrir en costes. Utilice la [calculadora de precios](https://{DomainName}/pricing/) para generar una estimación de coste basada en el uso estimado.

## Arquitectura
{: #architecture}

<p style="text-align: center;">

  ![Arquitectura](images/solution2/Architecture.png)
</p>

1. Un desarrollador genera una aplicación inicial con {{site.data.keyword.dev_cli_notm}}.
1. Al crear la aplicación se genera una imagen de contenedor Docker.
1. La imagen se envía a un espacio de nombres en {{site.data.keyword.containershort_notm}}.
1. La aplicación se despliega en un clúster de Kubernetes.
1. Los usuarios acceden a la aplicación.

## Antes de empezar
{: #prereqs}

* [Configure la CLI de {{site.data.keyword.registrylong_notm}} y el espacio de nombres del registro](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [Instale {{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli): script para instalar docker, kubectl, helm, la cli de ibmcloud y los plugins necesarios.
* [Debe comprender los conceptos básicos de Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## Creación de un clúster de Kubernetes
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} combina Docker y
Kubernetes para ofrecer herramientas potentes, una interfaz intuitiva para el usuario y funciones integradas de seguridad e identificación para automatizar el despliegue, operación, escalado y supervisión de apps contenerizadas sobre un clúster de hosts de cálculo.

La parte principal de esta guía de aprendizaje se puede seguir con un clúster **gratuito**. Dos secciones opcionales relacionadas con Kubernetes Ingress y con el dominio personalizado necesitan un clúster de **pago** de tipo **Estándar**.

1. Cree un clúster de Kubernetes desde el [catálogo de {{site.data.keyword.Bluemix}}](https://{DomainName}/containers-kubernetes/launch).

   Para realizar un uso adecuado, compruebe los detalles de configuración, como el número de CPU, la memoria y el número de nodos trabajadores que se obtienen con los planes Lite y Estándar.
   {:tip}

   ![Creación de un clúster de Kubernetes en IBM Cloud](images/solution2/KubernetesClusterCreation.png)
2. Seleccione el **Tipo de clúster** y pulse **Crear clúster** para suministrar un clúster de Kubernetes.
3.  Compruebe el estado del **Clúster** y de los **Nodos trabajadores** y espere a que estén **listos**.

### Configuración de kubectl

En este paso, configurará kubectl de modo que apunte el clúster que acaba de crear. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) es una herramienta de línea de mandatos que se utiliza para interactuar con un clúster de Kubernetes.

1. Utilice `ibmcloud login` para iniciar la sesión de forma interactiva. Especifique la organización (org), la ubicación y el espacio bajo los que se crea el clúster. Puede volver a confirmar los detalles con el mandato `ibmcloud target`.
2. Cuando el clúster esté preparado, recupere la configuración de clúster estableciendo la variable de entorno MYCLUSTER en el nombre de clúster:
   ```bash
   export MYCLUSTER=<CLUSTER_NAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
   ```
   {: pre}
3. Copie y pegue el mandato **export** para establecer la variable de entorno KUBECONFIG tal como se indica. Para verificar si la variable de entorno KUBECONFIG se ha establecido correctamente o no, ejecute el mandato siguiente: `echo $KUBECONFIG`
4. Compruebe que el mandato `kubectl` esté configurado correctamente
   ```bash
   kubectl cluster-info
   ```
   {: pre}
   ![](images/solution2/kubectl_cluster-info.png)


## Creación de una aplicación inicial
{: #create_application}

La herramienta `ibmcloud dev` reduce considerablemente el tiempo de desarrollo mediante la generación de iniciadores de aplicaciones con todos los contenedores modelo necesarios, la creación y la configuración del código para que pueda empezar a codificar rápidamente la lógica necesaria para la empresa.

1. Inicie el asistente `ibmcloud dev`.
   ```
   ibmcloud dev create
   ```
   {: pre}

1. Seleccione `Servicio de fondo / App web` > `Java - MicroProfile / JavaEE` > `App web Java con Eclipse MicroProfile y Java EE (App web)` para crear un iniciador Java. (Para crear un iniciador Node.js, utilice `Servicio de fondo / App web` > `Node`> `App web Node.js con Express.js (App web)`)
1. Especifique un **nombre** para la aplicación.
1. Seleccione el grupo de recursos en el que va a desplegar esta aplicación.
1. No añada servicios adicionales.
1. No añada una cadena de herramientas DevOps; seleccione **despliegue manual**.

Esto genera una aplicación inicial completa con el código y todos los archivos de configuración necesarios para el desarrollo local y el despliegue en la nube en Cloud Foundry o Kubernetes. 

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### Creación de la aplicación

Puede crear y ejecutar la aplicación igual que lo haría con `mvn` para el desarrollo local de Java o con `npm` para el desarrollo de nodos.  También puede crear una imagen de docker y ejecutar la aplicación en un contenedor para garantizar la ejecución coherente localmente y en la nube. Siga los siguientes pasos para crear la imagen de docker.

1. Asegúrese de que el motor de Docker local se ha iniciado.
   ```
   docker ps
   ```
   {: pre}
2. Vaya al directorio del proyecto generado.
   ```
   cd <project name>
   ```
   {: pre}
3. Cree la aplicación.
   ```
   ibmcloud dev build
   ```
   {: pre}

   Esto puede tardar unos minutos en ejecutarse ya que se descargan todas las dependencias de la aplicación y se crea una imagen de Docker que contiene la aplicación y el entorno necesario.

### Ejecución local de la aplicación

1. Ejecute el contenedor.
   ```
   ibmcloud dev run
   ```
   {: pre}

   Esto utiliza el motor de Docker local para ejecutar la imagen de docker que ha creado en el paso anterior.
2. Después de que se inicie el contenedor, vaya a `http://localhost:9080/`. Si ha creado una aplicación Node.js, vaya a `http://localhost:3000/`.
![](images/solution2/LibertyLocal.png)

## Despliegue de la aplicación en el clúster mediante un diagrama de helm
{: #deploy}

En esta sección, primero debe enviar la imagen de Docker al registro de contenedor privado de IBM Cloud y, a continuación, crear un despliegue de Kubernetes que apunte a dicha imagen.

1. Localice su **espacio de nombres** obteniendo una lista de todos los espacios de nombres del registro.
   ```sh
   ibmcloud cr namespaces
   ```
   {: pre}
   Si tiene un espacio de nombres, anote el nombre para utilizarlo posteriormente. Si no tiene uno, créelo.
   ```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. Establezca las variables de entorno MYNAMESPACE y MYPROJECT en su espacio de nombres y nombre de proyecto respectivamente.

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. Identifique el **Registro de contenedores** (por ejemplo, us.icr.io) mediante el mandato `ibmcloud cr info`
4. Establezca la variable de entorno MYREGISTRY en el registro.
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. Cree y etiquete (`-t`) la imagen del docker
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. Envíe la imagen de docker en el registro de contenedores en IBM Cloud
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. En un IDE, vaya a **values.yaml** bajo `chart\YOUR PROJECT NAME` y actualice el valor de **repositorio de imágenes** para que apunte a la imagen del registro de contenedores de IBM Cloud. **Guarde** el archivo.

   Para ver detalles del repositorio de imágenes, ejecute `echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}`

8. [Helm](https://helm.sh/) le ayuda a gestionar aplicaciones Kubernetes mediante diagramas de Helm, que ayudan a definir, instalar y actualizar incluso la aplicación Kubernetes más compleja. Para inicializar Helm, vaya a `chart\YOUR PROJECT NAME` y ejecute el mandato siguiente en el clúster

   ```bash
   helm init
   ```
   {: pre}
   Para actualizar helm, ejecute este mandato `helm init --upgrade`
   {:tip}

9. Para instalar un diagrama de Helm, vaya al directorio `chart\YOUR PROJECT NAME` y ejecute el mandato siguiente
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. Utilice `kubectl get service ${MYPROJECT}-service` para su aplicación Java y `kubectl get service ${MYPROJECT}-application-service` para su aplicación Node.js para identificar el puerto público en el que escucha el servicio. El puerto es un número de 5 dígitos (por ejemplo, 31569) que hay bajo `PORT(S)`.
11. Para la IP pública del nodo trabajador, ejecute el mandato siguiente
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. Acceda a la aplicación en `http://worker-ip-address:portnumber/`.

## Utilización del dominio proporcionado por IBM para el clúster
{: #ibm_domain}

En el paso anterior, se ha accedido a la aplicación con un puerto no estándar. El servicio se ha expuesto mediante la característica NodePort de Kubernetes.

Los clústeres de pago se proporcionan con un dominio proporcionado por IBM. Esto le proporciona una mejor opción para exponer aplicaciones con un URL adecuado y en puertos HTTP/S estándares.

Utilice Ingress para configurar la conexión de entrada del clúster con el servicio.

![Ingress](images/solution2/Ingress.png)

1. Identifique el **dominio Ingress** proporcionado por IBM
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   para localizar
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. Cree un archivo Ingress `ingress-ibmdomain.yml` que apunte al dominio con soporte de HTTP y HTTPS. Utilice el siguiente archivo como plantilla, sustituyendo todos los valores especificados entre <> por los valores adecuados de la salida anterior.**service-name** es el nombre que hay bajo `==> v1/Service` en el paso anterior o ejecute `kubectl get svc` para encontrar el nombre de servicio de tipo **NodePort**.
   ```yaml
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-ibmdomain-http-and-https
   spec:
     tls:
     - hosts:
       -  <nameofproject>.<ingress-sub-domain>
       secretName: <ingress-secret>
     rules:
     - host: <nameofproject>.<ingress-sub-domain>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: codeblock}
3. Despliegue Ingress
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. Acceda a la aplicación en `https://<nameofproject>.<ingress-sub-domain>/`

## Utilización de su propio dominio personalizado
{: #custom_domain}

Para utilizar su dominio personalizado, debe actualizar los registros de DNS con un registro CNAME que apunte al dominio proporcionado por IBM o con un registro A que apunte a la dirección IP pública portable del servicio Ingress proporcionado por IBM. Dado que un clúster de pago viene con direcciones IP fijas, un registro A es una buena opción.

Consulte el apartado sobre [Utilización del controlador Ingress con un dominio personalizado](https://{DomainName}/docs/containers?topic=containers-ingress#ingress) para obtener más información.

### con HTTP

1. Cree un archivo Ingress `ingress-customdomain-http.yml` que apunte a su dominio:
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-for-customdomain-http
   spec:
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
2. Despliegue Ingress
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. Acceda a la aplicación en `http://<customdomain>/`

### con HTTPS

Si intenta acceder a su aplicación con HTTPS en este momento, `https://<customdomain>/`, es probable que reciba un aviso de seguridad del navegador web que indique que su conexión no es privada. También podría recibir un código 404 ya que el Ingress recién configurado no sabe cómo dirigir el tráfico HTTPS.

1. Obtenga un certificado SSL de confianza para el dominio. Necesitará el certificado y la clave:
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3
   Puede utilizar [Cifremos](https://letsencrypt.org/) para generar un certificado de confianza.
2. Guarde el certificado y la clave en archivos con formato ascii base64.
3. Cree un secreto TLS para almacenar el certificado y la clave:
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<custom-domain.cert> --key=<custom-domain.key>
   ```
   {: pre}
4. Cree un archivo Ingress `ingress-customdomain-https.yml` que apunte a su dominio:
   ```
   apiVersion: extensions/v1beta1
   kind: Ingress
   metadata:
     name: ingress-customdomain-https
   spec:
     tls:
     - hosts:
       - <my-custom-domain.com>
       secretName: <my-custom-domain-secret-name>
     rules:
     - host: <my-custom-domain.com>
       http:
         paths:
         - path: /
           backend:
             serviceName: <service-name>
             servicePort: 9080
   ```
   {: pre}
5. Despliegue el Ingress:
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. Acceda a la aplicación en `https://<customdomain>/`.

## Supervisión del estado de la aplicación
{: #monitor_application}

1. Para comprobar el estado de la aplicación, vaya a [clústeres](https://{DomainName}/containers-kubernetes/clusters) para ver una lista de clústeres y pulse el clúster que ha creado antes.
2. Pulse **Panel de control de Kubernetes** para iniciar el panel de control en un separador nuevo. ![](images/solution2/launch_kubernetes_dashboard.png)
3. Seleccione **Nodos** en el panel de la izquierda, pulse el **Nombre** de los nodos y consulte los **Recursos de asignación** para ver el estado de los nodos.    ![](images/solution2/KubernetesDashboard.png)
4. Para revisar los registros de la aplicación desde el contenedor, seleccione **Pods**, **pod-name** y **Registros**.
5. Para ejecutar **ssh** en el contenedor, identifique el nombre del pod del paso anterior y ejecute
   ```sh
   kubectl exec -it <pod-name> -- bash
   ```
   {: pre}

## Escalado de pods de Kubernetes
{: #scale_cluster}

A medida que aumenta la carga en la aplicación, puede aumentar manualmente el número de réplicas de pod del despliegue. Las réplicas se gestionan mediante un [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/). Para escalar la aplicación a dos réplicas, ejecute el mandato siguiente:

   ```sh
 kubectl scale deployment <nameofproject>-deployment --replicas=2
   ```
   {: pre}

Tras un corto periodo de tiempo, verá dos pods para su aplicación en el panel de control de Kubernetes (o con el mandato `kubectl get pods`). El controlador de Ingress del clúster manejará el equilibrio de la carga entre las dos réplicas. El escalado horizontal también se puede hacer automáticamente.

Consulte la documentación de Kubernetes para obtener información sobre el escalado manual y automático:

   * [Escalado de un despliegue](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [Escalado automático de pod horizontal](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## Eliminación de recursos

* Suprima el clúster o suprima solo los artefactos de Kubernetes creados para la aplicación si tiene previsto reutilizar el clúster.

## Contenido relacionado

* [Servicio Kubernetes de IBM Cloud](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [Despliegue continuo en Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
