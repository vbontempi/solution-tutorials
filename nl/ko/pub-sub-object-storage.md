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

# 오브젝트 스토리지 및 발행/구독 메시징을 사용하여 비동기 데이터 처리
{: #pub-sub-object-storage}
이 튜토리얼에서는 Apache Kafka 기반 메시징 서비스를 사용하여 장기 실행 워크로드를 Kubernetes 클러스터에서 실행 중인 애플리케이션에 오케스트레이션하는 방법에 대해 학습합니다. 이 패턴은 애플리케이션을 분리하여 스케일링 및 성능에 대한 제어를 강화하는 데 사용됩니다. {{site.data.keyword.messagehub}}를 사용하면 생성자 애플리케이션에 영향을 주지 않고 수행될 작업을 큐에 넣을 수 있어 장기 실행 태스크를 위해 이상적인 시스템입니다.  

{:shortdesc}

파일 처리 예제를 사용하여 이 패턴을 시뮬레이션합니다. 먼저 파일을 오브젝트 스토리지에 업로드하고 수행될 작업을 표시하는 메시지를 생성하는 UI 애플리케이션을 작성하십시오. 다음으로 메시지를 수신할 때 사용자가 업로드한 파일을 비동기로 처리하는 별도의 작업자 애플리케이션을 작성합니다. 

## 목표
{: #objectives}

* {{site.data.keyword.messagehub}}를 사용하여 생성자-이용자 패턴을 구현합니다. 
* 서비스를 Kubernetes 클러스터에 바인드합니다. 

## 사용되는 서비스
{: #services}

이 튜토리얼에서는 다음과 같은 런타임 및 서비스를 사용합니다. 
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

이 튜토리얼에서는 비용이 발생할 수 있습니다. [가격 계산기](https://{DomainName}/pricing/)를 사용하여 예상 사용량에 따른 비용 추정치를 생성하십시오. 

## 아키텍처
{: #architecture}

이 튜토리얼에서는 UI 애플리케이션이 Node.js로 작성되고 작업자 애플리케이션이 Java로 작성되어 이 패턴의 유연성을 강조합니다. 이 튜토리얼에서 두 애플리케이션은 동일한 Kubernetes 클러스터에서 실행되지만 각각 Cloud Foundry 애플리케이션 또는 서버리스 기능으로 구현되었을 수도 있습니다. 

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. 사용자가 UI 애플리케이션을 사용하여 파일을 업로드합니다. 
2. 파일이 {{site.data.keyword.cos_full_notm}}에 저장됩니다. 
3. 새 파일이 처리를 기다리고 있다는 메시지가 {{site.data.keyword.messagehub}} 주제에 전송됩니다. 
4. 준비가 되면 작업자가 메시지를 청취하고 새 파일의 처리를 시작합니다. 

## 시작하기 전에
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - {{site.data.keyword.cloud_notm}} CLI, Kubernetes, Helm 및 Docker를 설치하는 도구입니다. 

## Kubernetes 클러스터 작성
{: #create_kube_cluster}

1. [카탈로그](https://{DomainName}/containers-kubernetes/launch)에서 Kubernetes 클러스터를 작성하십시오. 이 튜토리얼을 쉽게 따라하기 위해 이름을 `mycluster`로 지정하십시오. 이 튜토리얼은 **무료** 클러스터를 사용하여 수행할 수 있습니다.
   ![IBM Cloud에서 Kubernetes 클러스터 작성](images/solution25/KubernetesClusterCreation.png)
2. **클러스터** 및 **작업자 노드**의 상태를 확인하여 **준비** 상태가 될 때까지 기다리십시오. 

### kubectl 구성

이 단계에서는 앞으로 새로 작성되는 클러스터를 가리키도록 kubectl을 구성합니다. [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/)은 Kubernetes 클러스터와 상호작용하기 위해 사용하는 명령행 도구입니다. 

1. `ibmcloud login`을 사용하여 대화식으로 로그인하십시오. 클러스터가 작성되는 조직(org), 위치 및 영역을 제공하십시오. `ibmcloud target` 명령을 실행하여 세부사항을 다시 확인할 수 있습니다. 
2. 클러스터가 준비되면 클러스터 구성을 검색하십시오. 
   ```bash
   ibmcloud cs cluster-config <cluster-name>
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

## {{site.data.keyword.messagehub}} 인스턴스 작성
 {: #create_messagehub}

{{site.data.keyword.messagehub}}는 실시간 데이터 피드를 처리하기 위해 대기 시간이 짧은 플랫폼을 제공하는 처리량이 많은 오픈 소스 메시징 시스템인 Apache Kafka를 기반으로 하는 빠르고 확장 가능한 완전히 관리되는 메시징 서비스입니다. 

 1. 대시보드에서 [**리소스 작성**](https://{DomainName}/catalog/)을 클릭한 후 애플리케이션 서비스 섹션에서 [**{{site.data.keyword.messagehub}}**](https://{DomainName}/catalog/services/event-streams)를 선택하십시오. 
 2. 서비스 이름을 `mymessagehub`로 지정한 후 **작성**을 클릭하십시오. 
 3. 서비스 인스턴스를 `default` Kubernetes 네임스페이스에 바인드하여 클러스터에 서비스 인증 정보를 제공하십시오. 
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

cluster-service-bind 명령은 JSON 형식으로 서비스 인스턴스의 인증 정보를 보유하는 클러스터 시크릿을 작성합니다. `kubectl get secrets`를 사용하여 이름이 `binding-mymessagehub`인 생성된 시크릿을 확인하십시오. 자세한 정보는 [서비스 통합](https://{DomainName}/docs/containers?topic=containers-integrations#integrations)을 참조하십시오. 

{:tip}

## Object Storage 인스턴스 작성

{: #create_cos}

{{site.data.keyword.cos_full_notm}}는 암호화되고 여러 지리적 위치에 분산되어 있고 REST API를 사용하여 HTTP를 통해 액세스됩니다. {{site.data.keyword.cos_full_notm}}는 구조화되지 않은 데이터에 대한 유연하고 비용 효율이 높고 확장 가능한 클라우드 스토리지를 제공합니다. 이를 사용하여 UI에 의해 업로드된 파일을 저장합니다. 

1. 대시보드에서 [**리소스 작성**](https://{DomainName}/catalog/)을 클릭한 후 스토리지 섹션에서 [**{{site.data.keyword.cos_short}}**](https://{DomainName}/catalog/services/cloud-object-storage)를 선택하십시오. 
2. 서비스 이름을 `myobjectstorage`로 지정한 후 **작성**을 클릭하십시오. 
3. **버킷 작성**을 클릭하십시오. 
4. 버킷 이름을 고유 이름(예: `username-mybucket`)으로 설정하십시오. 
5. **교차 지역** 복원성 및 **us-geo** 위치를 선택한 후 **작성**을 클릭하십시오. 
6. 서비스 인스턴스를 `default` Kubernetes 네임스페이스에 바인드하여 클러스터에 서비스 인증 정보를 제공하십시오. 
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## 클러스터에 UI 애플리케이션 배치

UI 애플리케이션은 사용자가 파일을 업로드할 수 있게 하는 단순 Node.js Express 웹 애플리케이션입니다. 이는 위에서 작성된 Object Storage 인스턴스에 파일을 저장하고 새 파일을 처리할 준비가 되었다는 메시지를 {{site.data.keyword.messagehub}} 주제 "work-topic"에 전송합니다. 

1. 샘플 애플리케이션 저장소를 로컬로 복제한 후 디렉토리를 `pubsub-ui` 폴더로 변경하십시오. 
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. `config.js`를 열고 버킷 이름으로 COSBucketName을 업데이트하십시오. 
3. 애플리케이션을 빌드하고 배치하십시오. 배치 명령은 Docker 이미지를 생성하여 {{site.data.keyword.registryshort_notm}}에 푸시한 후 Kubernetes 배치를 작성합니다. 앱을 배치하는 동안 대화식 지시사항을 따르십시오. 
```sh
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
4. 애플리케이션을 방문하여 `sample-files` 폴더에서 파일을 업로드하십시오. 업로드된 파일은 Object Storage에 저장되며 작업자 애플리케이션에 의해 처리될 때까지 상태는 "대기 중"입니다. 이 브라우저 창을 열어 두십시오. 

   ![](images/solution25/files_uploaded.png)

## 클러스터에 작업자 애플리케이션 배치

작업자 애플리케이션은 {{site.data.keyword.messagehub}} Kafka "work-topic" 주제에서 메시지를 청취하는 Java 애플리케이션입니다. 새 메시지의 경우 작업자는 메시지에서 파일의 이름을 검색한 후 Object Storage에서 파일 컨텐츠를 가져옵니다. 그런 다음 파일의 처리를 시뮬레이션한 후 완료 시 또 다른 메시지를 "result-work" 주제에 전송합니다. UI 애플리케이션은 이 주제를 청취하고 상태를 업데이트합니다. 

1. 디렉토리를 `pubsub-worker` 디렉토리로 변경
```sh
  cd ../pubsub-worker
```
2. `resources/cos.properties`를 열고 버킷 이름으로 `bucket.name` 특성을 업데이트하십시오. 
2. 작업자 애플리케이션을 빌드하고 배치하십시오. 
```
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
3. 배치가 완료되면 웹 애플리케이션이 있는 브라우저 창을 다시 확인하십시오. 각 파일 옆의 상태가 이제 "처리됨"으로 변경됩니다.
![](images/solution25/files_processed.png)

이 튜토리얼에서는 Kafka 기반 {{site.data.keyword.messagehub}}를 사용하여 생성자-이용자 패턴을 구현하는 방법에 대해 학습했습니다. 이를 통해 웹 애플리케이션의 속도를 높이고 무거운 처리를 다른 애플리케이션에 넘길 수 있습니다. 작업을 수행해야 하는 경우 생성자(웹 애플리케이션)는 메시지를 생성하며 메시지를 구독하는 한 명 이상의 작업자 사이에서 작업의 로드 밸런싱이 수행됩니다. Kubernetes에서 실행되는 Java 애플리케이션을 사용하여 처리를 수행했지만 이 애플리케이션은 [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing)일 수도 있습니다. Kubernetes에서 실행되는 애플리케이션은 장기 실행되는 집중적인 워크로드에 이상적인 반면 단기 프로세스에는 {{site.data.keyword.openwhisk_short}}가 더 적합합니다. 

## 리소스 제거
{:removeresources}

[리소스 목록](https://{DomainName}/resources/)으로 이동한 후 다음을 수행하십시오. 
1. Kubernetes 클러스터 `mycluster`를 삭제하십시오. 
2. {{site.data.keyword.cos_full_notm}} `myobjectstorage`를 삭제하십시오. 
3. {{site.data.keyword.messagehub}} `mymessagehub`를 삭제하십시오. 
4. 왼쪽 메뉴에서 **Kubernetes**를 선택하고 **레지스트리**를 선택한 후 `pubsub-xxx` 저장소를 삭제하십시오. 

## 관련 컨텐츠
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [Object Storage에 대한 액세스 관리](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [{{site.data.keyword.openwhisk_short}}를 사용하여 {{site.data.keyword.messagehub}} 데이터 처리](https://github.com/IBM/openwhisk-data-processing-message-hub)
