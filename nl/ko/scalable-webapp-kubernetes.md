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

# Kubernetes에서 스케일링 가능 웹 애플리케이션
{: #scalable-webapp-kubernetes}

이 튜토리얼에서는 웹 애플리케이션을 스캐폴드하고 컨테이너에서 로컬로 실행한 후 [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)를 사용하여 작성된 Kubernetes 클러스터에 배치하는 방법에 대해 설명합니다. 또한 사용자 정의 도메인을 바인드하고 환경의 상태를 모니터하고 애플리케이션을 스케일링하는 방법에 대해 학습합니다.
{:shortdesc}

컨테이너는 환경 사이에서 앱을 원활하게 이동할 수 있도록 앱 및 앱의 모든 종속 항목을 패키지하는 표준 방법입니다. 가상 머신과는 달리 컨테이너는 운영 체제를 번들로 포함하지 않습니다. 앱 코드, 런타임, 시스템 도구, 라이브러리 및 설정만 컨테이너에 패키지됩니다. 컨테이너는 가상 머신보다 가볍고 휴대하기 쉬우며 효율적입니다. 

프로젝트를 시작하려고 하는 개발자의 경우 {{site.data.keyword.dev_cli_notm}} CLI를 사용하면 즉시 실행하거나 자체 솔루션의 스타터로 사용자 정의할 수 있는 템플리트 애플리케이션을 생성하여 신속하게 애플리케이션 개발 및 배치를 수행할 수 있습니다. 스타터 애플리케이션 코드, Docker 컨테이너 이미지 및 CloudFoundry 자산 생성 외에도 디바이스 CLI 및 웹 콘솔에서 사용하는 코드 생성기는 [Kubernetes](https://kubernetes.io/) 환경에 배치를 지원하는 파일을 생성합니다. 템플리트는 애플리케이션의 초기 Kubernetes 배치 구성을 설명하고 필요에 따라 다중 이미지 또는 복합 배치를 작성하기 위해 쉽게 확장되는 [Helm](https://github.com/kubernetes/helm) 차트를 생성합니다. 

## 목표
{: #objectives}

* 스타터 애플리케이션을 스캐폴드합니다. 
* 애플리케이션을 Kubernetes 클러스터에 배치합니다. 
* 사용자 정의 도메인을 바인드합니다. 
* 클러스터의 상태 및 로그를 모니터합니다. 
* Kubernetes 팟(Pod)을 스케일링합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

<p style="text-align: center;">

  ![아키텍처](images/solution2/Architecture.png)
</p>

1. 개발자가 {{site.data.keyword.dev_cli_notm}}를 사용하여 스타터 애플리케이션을 생성합니다. 
1. 애플리케이션을 빌드하면 Docker 컨테이너 이미지가 생성됩니다. 
1. 이 이미지가 {{site.data.keyword.containershort_notm}}의 네임스페이스에 푸시됩니다. 
1. 애플리케이션이 Kubernetes 클러스터에 배치됩니다. 
1. 사용자가 애플리케이션에 액세스합니다. 

## 시작하기 전에
{: #prereqs}

* [{{site.data.keyword.registrylong_notm}} CLI 및 레지스트리 네임스페이스를 설정하십시오. ](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [{{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)(docker, kubectl, helm, ibmcloud cli 및 필수 플러그인을 설치하는 스크립트)를 설치하십시오. 
* [Kubernetes의 기본사항에 대해 이해하십시오. ](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## Kubernetes 클러스터 작성
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}}는 Docker 및 Kubernetes 기술, 직관적인 사용자 경험, 기본 제공 보안 및 격리를 결합하여 컴퓨팅 호스트의 클러스터에서 컨테이너화된 앱의 배치, 오퍼레이션, 스케일링 및 모니터링을 자동화하는 강력한 도구를 제공합니다. 

이 튜토리얼의 주요 부분은 **무료** 클러스터를 사용하여 수행할 수 있습니다. Kubernetes Ingress 및 사용자 정의 도메인과 관련된 두 가지 선택적 절에는 **표준** 유형의 **유료** 클러스터가 필요합니다. 

1. [{{site.data.keyword.Bluemix}} 카탈로그](https://{DomainName}/containers-kubernetes/launch)에서 Kubernetes 클러스터를 작성하십시오. 

   사용하기 쉽도록 CPU 수, 메모리 및 Lite 및 표준 플랜 사용 시 얻는 작업자 노드 수 등의 구성 세부사항을 확인하십시오.
   {:tip}

   ![IBM Cloud에서 Kubernetes 클러스터 작성](images/solution2/KubernetesClusterCreation.png)
2. **클러스터 유형**을 선택한 후 **클러스터 작성**을 클릭하여 Kubernetes 클러스터를 프로비저닝하십시오. 
3.  **클러스터** 및 **작업자 노드**의 상태를 확인한 후 **준비** 상태가 될 때까지 기다리십시오. 

### kubectl 구성

이 단계에서는 앞으로 새로 작성되는 클러스터를 가리키도록 kubectl을 구성합니다. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/)은 Kubernetes 클러스터와 상호작용하기 위해 사용하는 명령행 도구입니다. 

1. `ibmcloud login`을 사용하여 대화식으로 로그인하십시오. 클러스터가 작성되는 조직(org), 위치 및 영역을 제공하십시오. `ibmcloud target` 명령을 실행하여 세부사항을 다시 확인할 수 있습니다. 
2. 클러스터가 준비되면 MYCLUSTER 환경 변수를 클러스터 이름으로 설정하여 클러스터 구성을 검색하십시오. 
   ```bash
   export MYCLUSTER=<CLUSTER_NAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
   ```
   {: pre}
3. **export** 명령을 복사한 후 붙여넣어 지시대로 KUBECONFIG 환경 변수를 설정하십시오. KUBECONFIG 환경 변수가 제대로 설정되었는지 확인하려면 다음 명령을 실행하십시오.
  `echo $KUBECONFIG`
4. `kubectl` 명령이 올바르게 구성되었는지 확인하십시오. 
   ```bash
   kubectl cluster-info
   ```
   {: pre}
   ![](images/solution2/kubectl_cluster-info.png)


## 스타터 애플리케이션 작성
{: #create_application}

`ibmcloud dev` 도구를 사용하면 더 빠르게 비즈니스 로직 코딩을 시작할 수 있도록 필요한 모든 표준 유형, 빌드 및 구성 코드를 가진 애플리케이션 스타터를 생성하여 개발 시간을 상당히 줄일 수 있습니다. 

1. `ibmcloud dev` 마법사를 시작하십시오. 
   ```
   ibmcloud dev create
   ```
   {: pre}

1. `Backend Service / Web App` > `Java - MicroProfile / JavaEE` > `Java Web App with Eclipse MicroProfile and Java EE (Web App)`를 선택하여 Java 스타터를 작성하십시오. (Node.js 스타터를 대신 작성하려면 `Backend Service / Web App` > `Node`> `Node.js Web App with Express.js (Web App)`를 사용하십시오.)
1. 애플리케이션의 **이름**을 입력하십시오. 
1. 이 애플리케이션을 배치할 리소스 그룹을 선택하십시오. 
1. 추가적인 서비스는 추가하지 마십시오. 
1. DevOps 도구 체인을 추가하지 말고 **수동 배치**를 선택하십시오. 

그러면 로컬 개발 및 Cloud Foundry 또는 Kubernetes의 클라우드에 배치를 위해 필요한 모든 구성 파일 및 코드로 완성되는 스타터 애플리케이션이 생성됩니다.  

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### 애플리케이션 빌드

`mvn`(java 로컬 개발의 경우) 또는 `npm`(노드 개발의 경우)을 사용하여 평소처럼 애플리케이션을 빌드하고 실행할 수 있습니다. Docker 이미지를 빌드하고 컨테이너에서 애플리케이션을 실행하여 로컬로 또는 클라우드에서 일관된 실행을 보장할 수도 있습니다. Docker 이미지를 빌드하려면 다음 단계를 사용하십시오.

1. 로컬 Docker 엔진이 시작되었는지 확인하십시오. 
   ```
   docker ps
   ```
   {: pre}
2. 생성된 프로젝트 디렉토리로 변경하십시오. 
   ```
   cd <project name>
   ```
   {: pre}
3. 애플리케이션을 빌드하십시오. 
   ```
   ibmcloud dev build
   ```
   {: pre}

   모든 애플리케이션 종속 항목이 다운로드되고 애플리케이션 및 필요한 모든 환경이 포함된 Docker 이미지가 빌드되기 때문에 실행하는 데 몇 분이 걸립니다. 

### 애플리케이션을 로컬에서 실행

1. 컨테이너를 실행하십시오. 
   ```
   ibmcloud dev run
   ```
   {: pre}

   여기서는 로컬 Docker 엔진을 사용하여 이전 단계에서 빌드된 Docker 이미지를 실행합니다. 
2. 컨테이너가 시작되면 `http://localhost:9080/`로 이동하십시오. Node.js 애플리케이션을 작성한 경우에는 `http://localhost:3000/`로 이동하십시오.
  ![](images/solution2/LibertyLocal.png)

## Helm 차트를 사용하여 클러스터에 애플리케이션 배치
{: #deploy}

이 절에서는 먼저 Docker 이미지를 IBM Cloud 개인용 컨테이너 레지스트리에 푸시한 후 해당 이미지를 가리키는 Kubernetes 배치를 작성합니다. 

1. 레지스트리에 있는 모든 네임스페이스를 나열하여 **네임스페이스**를 찾으십시오. 
   ```sh
   ibmcloud cr namespaces
   ```
   {: pre}
   네임스페이스가 있으면 나중에 사용할 수 있도록 이름을 기록해 두십시오. 네임스페이스가 없으면 작성하십시오.
   ```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. MYNAMESPACE 및 MYPROJECT 환경 변수를 각각 네임스페이스 및 프로젝트 이름으로 설정하십시오. 

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. `ibmcloud cr info`를 실행하여 **컨테이너 레지스트리**(예: us.icr.io)를 식별하십시오. 
4. MYREGISTRY env var을 레지스트리로 설정하십시오. 
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. Docker 이미지를 빌드하고 태그를 지정(`-t`)하십시오. 
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. Docker 이미지를 IBM Cloud의 컨테이너 레지스트리에 푸시하십시오. 
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. IDE에서는 `chart\YOUR PROJECT NAME` 아래의 **values.yaml**로 이동하여 IBM Cloud 컨테이너 레지스트리의 이미지를 가리키는 **이미지 저장소** 값을 업데이트하십시오. 파일을 **저장**하십시오. 

   이미지 저장소 세부사항을 보려면 `echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}`를 실행하십시오. 

8. [Helm](https://helm.sh/)을 사용하면 가장 복잡한 Kubernetes 애플리케이션의 정의, 설치 및 업그레이드도 지원하는 Helm Charts를 통해 Kubernetes 애플리케이션을 관리할 수 있습니다. `chart\YOUR PROJECT NAME`으로 이동한 후 클러스터에서 아래 명령을 실행하여 Helm을 초기화하십시오. 

   ```bash
   helm init
   ```
   {: pre}
   Helm을 업그레이드하려면 `helm init --upgrade` 명령을 실행하십시오.
   {:tip}

9. Helm 차트를 설치하려면 `chart\YOUR PROJECT NAME` 디렉토리로 변경한 후 아래 명령을 실행하십시오. 
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. `kubectl get service ${MYPROJECT}-service`(Java 애플리케이션의 경우) 및 `kubectl get service ${MYPROJECT}-application-service`(Node.js 애플리케이션의 경우)를 사용하여 서비스가 청취 중인 공용 포트를 식별하십시오. 이 포트는 `PORT(S)` 아래의 5자리 숫자(예: 31569)입니다. 
11. 작업자 노드의 공인 IP의 경우 아래 명령을 실행하십시오. 
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. `http://worker-ip-address:portnumber/`에서 애플리케이션에 액세스하십시오. 

## 클러스터에 대해 IBM 제공 도메인 사용
{: #ibm_domain}

이전 단계에서는 표준이 아닌 포트를 사용하여 애플리케이션에 액세스했습니다. 서비스는 Kubernetes NodePort 기능을 사용하여 노출되었습니다. 

유료 클러스터는 IBM 제공 도메인과 함께 제공됩니다. 이는 표준 HTTP/S 포트에서 적절한 URL을 사용하여 애플리케이션을 노출하는 더 나은 옵션을 제공합니다. 

Ingress를 사용하여 서비스에 대한 클러스터 인바운드 연결을 설정하십시오. 

![Ingress](images/solution2/Ingress.png)

1. IBM 제공 **Ingress 도메인**을 식별하여 
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   다음을 찾으십시오.
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. HTTP 및 HTTPS를 지원하는 도메인을 가리키는 Ingress 파일 `ingress-ibmdomain.yml`을 작성하십시오. 다음과 같은 파일을 템플리트로 사용하여 <>로 묶인 모든 값을 위 출력의 적절한 값으로 대체하십시오. **service-name**은 위 단계의 `==> v1/Service` 아래에 있는 이름입니다. 그렇지 않으면 `kubectl get svc`를 실행하여 **NodePort** 유형의 서비스 이름을 찾으십시오. 
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
3. Ingress를 배치하십시오. 
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. `https://<nameofproject>.<ingress-sub-domain>/`에서 애플리케이션에 액세스하십시오. 

## 자체 사용자 정의 도메인 사용
{: #custom_domain}

사용자 정의 도메인을 사용하려면 IBM 제공 도메인을 가리키는 CNAME 레코드 또는 IBM 제공 Ingress의 이식 가능 공인 IP 주소를 가리키는 A 레코드를 사용하여 DNS 레코드를 업데이트해야 합니다. 유료 클러스터는 고정 IP 주소와 함께 제공되므로 A 레코드가 적절한 옵션입니다. 

자세한 정보는 [사용자 정의 도메인과 함께 Ingress 제어기 사용](https://{DomainName}/docs/containers?topic=containers-ingress#ingress)을 참조하십시오. 

### HTTP 사용

1. 도메인을 가리키는 Ingress 파일 `ingress-customdomain-http.yml`을 작성하십시오. 
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
2. Ingress를 배치하십시오. 
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. `http://<customdomain>/`에서 애플리케이션에 액세스하십시오. 

### HTTPS 사용

이번에는 `https://<customdomain>/`에서 HTTPS를 사용하여 애플리케이션에 액세스해야 하는 경우에는 연결이 개인용이 아님을 알리는 보안 경고가 웹 브라우저로부터 수신됩니다. 방금 구성된 Ingress는 HTTPS 트래픽의 방향을 지시하는 방법을 모르기 때문에 404도 수신됩니다. 

1. 도메인에 대해 신뢰할 수 있는 SSL 인증서를 확보하십시오. 인증서 및 키가 필요합니다. 
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3
   [Let's Encrypt](https://letsencrypt.org/)를 사용하여 신뢰할 수 있는 인증서를 생성할 수 있습니다.
2. 인증서 및 키를 base64 ascii 형식 파일로 저장하십시오. 
3. 인증서 및 키를 저장할 TLS 시크릿을 작성하십시오. 
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<custom-domain.cert> --key=<custom-domain.key>
   ```
   {: pre}
4. 도메인을 가리키는 Ingress 파일 `ingress-customdomain-https.yml`을 작성하십시오. 
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
5. Ingress를 배치하십시오. 
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. `https://<customdomain>/`에서 애플리케이션에 액세스하십시오. 

## 애플리케이션 상태 모니터
{: #monitor_application}

1. 애플리케이션의 상태를 확인하려면 [클러스터](https://{DomainName}/containers-kubernetes/clusters)로 이동하여 클러스터 목록을 확인한 후 위에서 작성한 클러스터를 클릭하십시오. 
2. **Kubernetes 대시보드**를 클릭하여 새 탭에서 대시보드를 실행하십시오.
   ![](images/solution2/launch_kubernetes_dashboard.png)
3. 왼쪽 분할창에서 **노드**를 선택하고 노드의 **이름**을 클릭한 후 **할당 리소스**에서 노드의 상태를 확인하십시오.
   ![](images/solution2/KubernetesDashboard.png)
4. 컨테이너에서 애플리케이션 로그를 검토하려면 **팟(Pod)**, **팟(Pod) 이름** 및 **로그**를 선택하십시오. 
5. **ssh**를 사용하여 컨테이너에 연결하려면 이전 단계에서 팟(Pod) 이름을 식별한 후 다음을 실행하십시오. 
   ```sh
   kubectl exec -it <pod-name> -- bash
   ```
   {: pre}

## Kubernetes 팟(Pod) 스케일링
{: #scale_cluster}

애플리케이션에서 로드가 증가함에 따라 배치에서 팟(Pod) 복제본 수를 수동으로 늘릴 수 있습니다. 복제본은 [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)에 의해 관리됩니다. 애플리케이션을 두 개의 복제본으로 스케일링하려면 다음 명령을 실행하십시오. 

   ```sh
 kubectl scale deployment <nameofproject>-deployment --replicas=2
   ```
   {: pre}

잠시 후 Kubernetes 대시보드에(또는 `kubectl get pods`를 사용하면) 애플리케이션에 대한 두 개의 팟(Pod)이 표시됩니다. 클러스터의 Ingress 제어기는 두 복제본 간 로드 밸런싱을 처리합니다. 수평적 스케일링은 자동화될 수도 있습니다. 

수동 및 자동 스케일링에 대해서는 Kubernetes 문서를 참조하십시오. 

   * [배치 스케일링](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [수평적 팟(Pod) 자동 스케일링](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## 리소스 제거

* 클러스터를 삭제하거나 애플리케이션에 대해 작성된 Kubernetes 아티팩트만 삭제하십시오(클러스터를 재사용하려는 경우). 

## 관련 컨텐츠

* [IBM Cloud Kubernetes Service](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [Kubernetes에 지속적 배치](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
