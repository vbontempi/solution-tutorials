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

# 將以 VM 為基礎的應用程式移動到 Kubernetes
{: #vm-to-containers-and-kubernetes}

本指導教學將全程指導您使用 {{site.data.keyword.containershort_notm}}，將以 VM 為基礎的應用程式移至 Kubernetes 叢集。[{{site.data.keyword.containershort_notm}}](https://{DomainName}/docs/containers?topic=containers-container_index#container_index) 藉由結合 Docker 和 Kubernetes 技術、直覺式使用者體驗以及內建安全和隔離來提供功能強大的工具，以在運算主機的叢集中自動部署、操作、擴充及監視容器化應用程式。
{: shortdesc}

本指導教學中的課程包括關於如何選取現有應用程式、將應用程式容器化以及將應用程式部署到 Kubernetes 叢集的概念。若要將以 VM 為基礎的應用程式容器化，可以在下列兩個選項中進行選擇。

1. 識別大的整合型應用程式中可分成其自己之微服務的元件。可以對這些微服務進行容器化，然後將其部署到 Kubernetes 叢集。
2. 將整個應用程式容器化，然後將應用程式部署到 Kubernetes 叢集。

根據您擁有的應用程式的類型不同，移轉應用程式的步驟也可能不同。您可以使用此指導教學來瞭解在移轉應用程式之前必須執行的一般步驟和必須考慮的事項。

## 目標
{: #objectives}

- 瞭解如何識別以 VM 為基礎之應用程式中的微服務，並瞭解如何在 VM 和 Kubernetes 之間對映元件。
- 瞭解如何將以 VM 為基礎的應用程式容器化。
- 瞭解如何在 {{site.data.keyword.containershort_notm}} 中將容器部署到 Kubernetes 叢集。
- 將學到的所有作法付諸實踐，在叢集裡執行 **JPetStore** 應用程式。

## 使用的服務
{: #products}

本指導教學使用下列 {{site.data.keyword.Bluemix_notm}} 服務：

- [{{site.data.keyword.containershort}}](https://{DomainName}/containers-kubernetes/catalog/cluster)
- [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/registry/private)
- [{{site.data.keyword.composeForMySQL_full}}](https://{DomainName}/catalog/services/compose-for-mysql)

**注意：**本指導教學可能會產生成本。請使用[定價計算機](https://{DomainName}/pricing/)，根據您的預估用量產生成本預估。

## 架構
{:#architecture}

### 使用 VM 的傳統應用程式架構

下圖顯示了以虛擬機器為基礎的傳統應用程式架構範例。

<p style="text-align: center;">
![架構圖](images/solution30/traditional_architecture.png)
</p>

1. 使用者傳送要求至 Java 應用程式的公用端點。公用端點由負載平衡器服務代表，該負載平衡器可對各個可用應用程式伺服器實例之間的送入網路資料流量進行負載平衡。
2. 負載平衡器選取一個在 VM 上執行的執行狀況良好的應用程式伺服器實例，並轉遞要求。
3. 應用程式伺服器將應用程式資料儲存於在 VM 上執行的 MySQL 資料庫中。應用程式伺服器實例管理 Java 應用程式並在 VM 上執行。應用程式檔案（如應用程式碼、配置檔和相依關係）儲存在 VM 上。

### 容器化的架構

下圖顯示了在 Kubernetes 叢集中執行的現代容器架構的範例。

<p style="text-align: center;">
![架構圖](images/solution30/modern_architecture.png)
</p>

1. 使用者傳送要求至 Java 應用程式的公用端點。公用端點由 Ingress 應用程式負載平衡器 (ALB) 代表，該負載平衡器對叢集中各個應用程式 pod 之間的送入網路資料流量進行負載平衡。ALB 是一組規則，容許入埠網路資料流量進入公用的公開應用程式。
2. ALB 將要求轉遞到叢集中某個可用的應用程式 pod。應用程式 pod 在工作者節點上執行，這些節點可以是虛擬機器，也可以是實體機器。
3. 應用程式 pod 將資料儲存在持續性磁區中。持續性磁區可用於在應用程式實例或工作者節點之間共用資料。
4. 應用程式 pod 將資料儲存在 {{site.data.keyword.Bluemix_notm}} 資料庫服務中。您可以在 Kubernetes 叢集中執行自己的資料庫，但使用受管理的資料庫即服務 (DBasS) 通常更容易進行配置，並且提供了內建的備份和調整。您可以在 [IBM Cloud 型錄](https://{DomainName}/catalog/?category=data)找到許多不同類型的資料庫。

### VM、容器和 Kubernetes

{{site.data.keyword.containershort_notm}} 提供了在 Kubernetes 叢集裡執行容器化應用程式的功能，並交付了下列工具和功能：

- 直觀的使用者體驗和強大的工具
- 內建安全和隔離，以便能快速交付安全應用程式
- 包含來自 IBM® Watson™ 的認知能力的雲端服務
- 能夠管理無狀態應用程式和有狀態工作負載的專用叢集資源

#### 虛擬機器與容器

**VM**，傳統應用程式在原生硬體上執行。單一應用程式通常不使用單一運算主機的完整資源。大多數組織都嘗試在單一運算主機上執行多個應用程式以避免浪費資源。您可以執行同一應用程式的多個副本，但要提供隔離，可以使用 VM 在相同的硬體上執行多個應用程式實例。這些 VM 有完整的作業系統堆疊，由於在運行環境和在磁碟上都有重複，所以它們相對較大並且效率不佳。

**容器**是包裝應用程式及其所有相依關係的標準方法，以便您可以無縫地在環境之間移動應用程式。容器與虛擬機器不同，容器不會搭載作業系統。只有應用程式碼、運行環境、系統工具、程式庫和設定會包裝在容器內。容器比虛擬機器更輕量、可攜性更高且更有效率。


此外，使用容器可以共用主機作業系統。這樣就能在減少重複的情況下仍提供隔離。使用容器還可以捨棄不需要的檔案（如系統檔案庫和二進位檔）以節省空間並減少攻擊面。在[這裡](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldvd/ldvd_r_plan_container_vm.html)閱讀有關虛擬機器和容器的更多資訊。

#### Kubernetes 編排

[Kubernetes](http://kubernetes.io/) 是一種容器編排器，可以在工作者節點的叢集中管理容器化應用程式的生命週期。您的應用程式可能需要其他許多資源才能執行，如磁區、網路、可協助您連接其他雲端服務的密碼，以及安全金鑰。Kubernetes 可協助您將這些資源新增至應用程式。Kubernetes 的金鑰範例是其宣告的模型。使用者提供所需的狀態，Kubernetes 試圖符合所說明的狀態，然後保持該狀態。

此[自學式研習會](https://github.com/IBM/kube101/blob/master/workshop/README.md)可協助您獲得使用 Kubernetes 的第一次實際經驗。此外，請查看 Kubernetes [概念](https://kubernetes.io/docs/concepts/)文件頁面以進一步瞭解 Kubernetes 概念。

### IBM 能為您做什麼

透過將 Kubernetes 叢集與 {{site.data.keyword.containerlong_notm}} 一起使用，您可獲得下列優點：

- 可在其中部署叢集的多個資料中心。
- 對 Ingress 和負載平衡器網路連線功能選項的支援。
- 動態持續性磁區支援。
- IBM 管理的高可用性 Kubernetes 主節點。

## 調整叢集大小
{: #sizing_clusters}

在設計叢集架構時，您希望平衡有關可用性、可靠性、複雜性和回復功能的成本。{{site.data.keyword.containerlong_notm}} 中的 Kubernetes 叢集根據應用程式的需求來提供架構選項。只需進行少量規劃，即可最充分地利用雲端資源，而不會過度架構或過度花費。即使您高估或低估了實際情況，也可以透過更多工作者節點或更大的工作者節點來輕鬆擴展或縮減叢集。

若要使用 Kubernetes 在雲端中執行正式作業應用程式，請考慮下列各項目：

1. 您預期會有來自特定地理位置的資料流量嗎？如果有，請選取實際離您最近的位置，以獲得最佳效能。
2. 為了實現更高的可用性，需要有多少叢集抄本？剛開始比較適合使用三個叢集，一個用於開發，一個用於測試，一個用於正式作業。查看有關建立多個環境的[組織使用者、團隊、應用程式的最佳作法](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#replicate-for-multiple-environments)解決方案手冊。
3. 工作者節點需要什麼[硬體](https://{DomainName}/docs/containers?topic=containers-plan_clusters#planning_worker_nodes)？虛擬機器還是 Bare Metal Server？
4. 需要多少個工作者節點？這主要取決於應用程式規模，您擁有的節點數量越多，應用程式所具有的備援能力越高。
5. 為了實現更高可用性，應該具備多少個抄本？在多個位置部署抄本叢集以提高應用程式可用性，並防止應用程式因某個位置失敗而運作中斷。
6. 應用程式至少需要哪些資源才能啟動？您可能要測試應用程式執行所需的記憶體數量和 CPU。然後，您的工作者節點應該有足夠的資源來部署並啟動應用程式。接著，作為 pod 規格的一部分，確保設定資源配額。Kubernetes 正是使用此設定來選取（或排定）有足夠容量來支援要求的工作者節點。預估有多少個 pod 會在工作者節點上執行，以及這些 pod 的資源需求。工作者節點至少要大到足以支援應用程式的一個 pod。
7. 何時增加工作者節點數目？您可以監視叢集使用情況，並根據需要增加節點數。請參閱本指導教學以瞭解如何[分析日誌並監視 Kubernetes 應用程式的性能](https://{DomainName}/docs/tutorials?topic=solution-tutorials-kubernetes-log-analysis-kibana#kubernetes-log-analysis-kibana)。
8. 是否需要備援的可靠儲存空間？如果是，請建立 NFS 儲存空間的持續性磁區要求，或者將 IBM Cloud 資料庫服務連結到 pod。

若要使上述內容更具體，我們可以假設，您要在雲端中執行正式作業 Web 應用程式，並預期有中高量的資料流量負載。讓我們看一下您將需要哪些資源：

1. 設定三個叢集，一個用於開發，一個用於測試，一個用於正式作業。
2. 開發和測試叢集開始時可以使用最低 RAM 和 CPU 選項（例如，每個叢集 2 個 CPU、4GB RAM 和一個工作者節點）。
3. 對於正式作業叢集，可能需要有更多資源才能具有高效能、高可用性和備援能力。我們可能選擇專用選項，甚至選擇 Bare Metal Server 選項，並有至少 4 個 CPU、16GB RAM 和兩個工作者節點。

## 確定要使用的資料庫選項
{: #database_options}

利用 Kubernetes，您可以使用兩個選項來處理資料庫：

1. 您可以在 Kubernetes 叢集中執行資料庫，以執行建立微服務來執行資料庫所需的操作。如果使用 MySQL 資料庫範例，請執行下列動作：
   - 建立 MySQL Dockerfile，請在這裡查看範例 [MySQL Dockerfile](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile)。
   - 您需要使用密碼來儲存資料庫認證。請在[這裡](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/db/Dockerfile.secret)查看相應的範例。
   - 您需要有 deployment.yaml 檔案，其中包含要部署到 Kubernetes 的資料庫的配置。請在[這裡](https://github.com/IBM-Cloud/jpetstore-kubernetes/blob/master/jpetstore/jpetstore.yaml)查看相應的範例。
2. 第二個選項是使用受管理的資料庫即服務 (DBasS) 選項。此選項通常更容易配置，並提供內建備份和調整功能。您可以在 [IBM Cloud 型錄](https://{DomainName}/catalog/?category=data)找到許多不同類型的資料庫。若要使用此選項，請執行下列動作：
   - 從 [IBM Cloud  型錄](https://{DomainName}/catalog/?category=data)建立受管理資料庫即服務 (DBasS)。
   - 將資料庫認證儲存在密碼中。您可參考「在 Kubernetes 密碼中儲存認證」區段中的資訊，進一步瞭解密碼。
   - 在應用程式中使用資料庫即服務 (DBasS)。

## 確定儲存應用程式檔案的位置
{: #decide_where_to_store_data}

{{site.data.keyword.containershort_notm}} 提供了用於在不同 pod 中儲存和共用資料的幾個選項。在災難情況下，並非所有儲存空間選項提供的持續性和可用性層次都是相同的。
{: shortdesc}

### 非持續資料儲存空間

根據設計，容器和 pod 的生存時間短，並且可能會非預期發生故障。您可以在容器的本端檔案系統中儲存資料。容器內的資料不能與其他容器或 pod 共用，並且在容器當機或被移除時會遺失。

### 瞭解如何為應用程式建立持續資料儲存空間

您可以使用原生 Kubernetes 持續性磁區，在 [NFS 檔案儲存空間](https://www.ibm.com/cloud/file-storage/details)或[區塊儲存空間](https://www.ibm.com/cloud/block-storage)中，持續保存應用程式資料和容器資料。
{: shortdesc}

若要佈建 NFS 檔案儲存空間或區塊儲存空間，必須透過建立持續性磁區要求 (PVC) 來為 pod 要求儲存空間。在 PVC 中，可以從預先定義的儲存類別中進行選擇，其定義儲存空間類型、儲存空間大小（以 GB 為單位）、IOPS、資料保留原則以及對儲存空間的讀取和寫入權。PVC 動態佈建持續性磁區 (PV)，用於代表 {{site.data.keyword.Bluemix_notm}} 中的實際儲存裝置。您可以將 PVC 裝載到 pod 中以在 PV 中進行讀寫。儲存在 PV 中的資料即使在容器當機或者 pod 重新排定的情況下也是可用的。支援 PV 的 NFS 檔案儲存空間和區塊儲存空間由 IBM 建立叢集，以便為資料提供高可用性。

若要瞭解如何建立 PVC，請遵循 [{{site.data.keyword.containershort_notm}} 儲存空間文件](https://{DomainName}/docs/containers?topic=containers-file_storage#file_storage)中的步驟。

### 瞭解如何將現有資料移動到持續性儲存空間

若要將資料從本端機器複製到持續性儲存空間中，必須將 PVC 裝載到 pod。然後可以將資料從本端機器複製到 pod 中的持續性磁區。
{: shortdesc}

1. 若要複製資料，需要先建立類似於以下內容的配置：

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

2. 然後，若要將資料從本端機器複製到 pod，需要使用如下指令：
   ```
    kubectl cp <local_filepath>/<filename> <namespace>/<pod>:<pod_filepath>
   ```
   {: pre}

3. 將資料從叢集裡的 pod 複製到本端機器：
   ```
   kubectl cp <namespace>/<pod>:<pod_filepath>/<filename> <local_filepath>/<filename>
   ```
   {: pre}


### 為持續性儲存空間設定備份

檔案共用和區塊儲存空間會佈建到叢集所在的相同位置。儲存空間本身由 IBM 在叢集伺服器上管理以提供高可用性。但是，檔案共用和區塊儲存空間不會自動進行備份，因此它們在整個位置發生故障時可能無法進行存取。為了防止資料遺失或損壞，可以設定定期備份，以便在需要時可用於還原資料。


如需相關資訊，請參閱有關 NFS 檔案儲存空間和區塊儲存空間的[備份和還原](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning)選項。

## 準備程式碼
{: #prepare_code}

### 套用 12 因子原則

[十二因子 APP](https://12factor.net/)是用於建置雲端原生應用程式的方法。當您要將應用程式容器化、將此應用程式移動到雲端，以及利用 Kubernetes 編排應用程式時，請務必瞭解並套用其中一些原則。{{site.data.keyword.Bluemix_notm}} 中要求使用其中一些原則。
{: shortdesc}

下面是需要的一些關鍵原則：

- **程式碼庫** - 在版本控制系統（如 Git 儲存庫）中追蹤所有原始碼和配置檔，這在使用 DevOps 管道進行部署時是必要的。
- **建置、發佈、執行** - 12 因子應用程式在建置、發佈和執行階段之間使用嚴格分離。這可以使用整合的 DevOps 交付管線自動化，以在將應用程式部署到叢集之前先建置和測試應用程式。請查看[連續部署到 Kubernetes](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes) 指導教學，以瞭解如何設定持續整合和交付管線。其中涵蓋來源控制、建置、測試和部署階段的設定，並展示了如何新增安全掃描器、通知和分析等整合。
- **配置** - 所有配置資訊都儲存在環境變數中。沒有任何的服務認證會寫在應用程式碼內。若要儲存認證，可以使用 Kubernetes 密碼。下面提供有關認證的更多資訊。

### 將認證儲存在 Kubernetes 密碼中
{: secrets}

將認證儲存在應用程式碼中絕不是一個好的做法。Kubernetes 反而提供了所謂的**[「密碼」](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)**，用來存放機密性資訊，如密碼、OAuth 記號或 SSH 金鑰。Kubernetes 密碼依預設加密，相較於將敏感資料逐字儲存在 `pod` 定義或 docker 映像檔中，使用 Kubernetes 密碼來儲存此資料更安全，也更靈活。

在 Kubernetes 中使用密碼的一種方式如下：

1. 建立檔案並將服務認證儲存在其中。
   ```
   {
       "url": "https://gateway-a.watsonplatform.net/visual-recognition/api",
       "api_key": <api_key>
   }
   ```
   {: codeblock}

2. 然後，執行以下指令以建立 Kubernetes 密碼，並在執行以下指令之後使用 `kubectl get secrets` 驗證是否已建立該密碼：

   ```
   kubectl create secret generic watson-visual-secret --from-file=watson-secrets.txt=./watson-secrets.txt
   ```
   {: pre}

## 將應用程式容器化
{: #build_docker_images}

若要將應用程式容器化，必須建立 docker 映像。
{: shortdesc}

映像檔是從 [Dockerfile](https://docs.docker.com/engine/reference/builder/#usage) 建立的，該檔案中包含用於建置映像檔的指示和指令。Dockerfile 可能會在其指示中參照個別儲存的建置構件，例如應用程式、應用程式的配置，以及其相依關係。

若要為現有應用程式建置自己的 Dockerfile，請使用下列指令：

- FROM - 選擇用於定義容器運行環境的主映像檔。
- ADD/COPY - 將目錄內容複製到容器中。
- WORKDIR - 在容器中設定工作目錄。
- RUN - 安裝應用程式在執行時需要的套裝軟體。
- EXPOSE - 使一個埠在容器外部可用。
- ENV NAME - 定義環境變數。
- CMD - 定義在容器啟動時執行的指令。

映像檔一般儲存在登錄中，而登錄可供公開存取（公用登錄）或設定一小群使用者的有限存取（專用登錄）。開始使用 docker 和 Kubernetes 在叢集中建立第一個容器化應用程式時，可以使用公用登錄（如 docker Hub）。但是對於企業 APP，請使用專用登錄（如在 {{site.data.keyword.registrylong_notm}} 中提供的登錄），以保護映像檔不被未經授權的使用者使用和變更。

若要將應用程式容器化並將其儲存在 {{site.data.keyword.registrylong_notm}} 中，請執行下列動作：

1. 您需要建立 Dockerfile，下面是 Dockerfile 的範例。
   ```
   # Build JPetStore war
   FROM openjdk:8 as builder
   COPY . /src
   WORKDIR /src
   RUN ./build.sh all

   # Use WebSphere Liberty base image from the docker Store
   FROM websphere-liberty:latest

   # Copy war from build stage and server.xml into image
   COPY --from=builder /src/dist/jpetstore.war /opt/ibm/wlp/usr/servers/defaultServer/apps/
   COPY --from=builder /src/server.xml /opt/ibm/wlp/usr/servers/defaultServer/
   RUN mkdir -p /config/lib/global
   COPY lib/mysql-connector-java-3.0.17-ga-bin.jar /config/lib/global
   ```
   {: codeblock}

2. 建立 Dockerfile 之後，接著需要建置容器映像檔並將其推送到 {{site.data.keyword.registrylong_notm}}。您可以使用以下指令建置容器：
   ```
   ibmcloud cr build -t <image_name> <directory_of_Dockerfile>
   ```
   {: pre}

## 將應用程式部署到 Kubernetes 叢集
{: #deploy_to_kubernetes}

在建置容器映像檔並將其推送到雲端之後，接下來需要將其部署到 Kubernetes 叢集。若要執行該動作，需要建立 deployment.yaml 檔案。
{: shortdesc}

### 瞭解如何建立 Kubernetes deployment.yaml 檔案

若要建立 Kubernetes deployment.yaml 檔案，請執行下列動作：

1. 建立 deployment.yaml 檔案，下面是 [deployment YAML](https://github.com/ibm-cloud/ModernizeDemo/blob/master/jpetstore/jpetstore.yaml) 檔案的範例。

2. 在 deployment.yaml 檔案中，可以定義容器的[資源配額](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)以指定每個容器正常啟動需要多少 CPU 和記憶體。如果容器已指定資源配額，則 Kubernetes 排程器在決定將 pod 放置在哪個工作者節點上時，可以做出更好的選擇。

3. 接下來，您可以使用下面的指令來建立並檢視已建立的部署和服務：

   ```
   kubectl create -f <filepath/deployment.yaml>
   kubectl get deployments
   kubectl get services
   kubectl get pods
   ```
   {: pre}

## 摘要
{: #summary}

在本指導教學中，您學到了下列內容：

- VM、容器和 Kubernetes 之間的差別。
- 如何針對不同的環境類型（開發、測試、正式作業）定義叢集。
- 如何處理資料儲存空間以及持續資料儲存空間的重要性。
- 對應用程式套用 12 因子原則並套用密碼作為 Kubernetes 中的認證。
- 建置 docker 映像檔並將其推送到 {{site.data.keyword.registrylong_notm}}。
- 建立 Kubernetes 部署檔案並將 docker 映像檔部署到 Kubernetes。

## 將學到的所有做法付諸實踐，在叢集中執行 JPetStore 應用程式。
{: #runthejpetstore}

將學到的所有做法付諸實踐，按照[示範](https://github.com/ibm-cloud/ModernizeDemo/)，在叢集上執行 **JPetStore** 應用程式並套用所學的概念。JPetStore 應用程式有一些延伸的功能，您可以利用這些功能，讓 IBM Watson 服務作為個別的微服務執行，以在 Kubernetes 中延伸應用程式。

## 相關內容
{: #related}

- Kubernetes 和 {{site.data.keyword.containershort_notm}} [開始使用](https://developer.ibm.com/courses/all/get-started-kubernetes-ibm-cloud-container-service/)
- [GitHub](https://github.com/IBM/container-service-getting-started-wt) 上的 {{site.data.keyword.containershort_notm}} 實驗室。
- Kubernetes 主要[文件](http://kubernetes.io/)。
- {{site.data.keyword.containershort_notm}} 中的[持續性儲存空間](https://{DomainName}/docs/containers?topic=containers-storage_planning#storage_planning)。
- 用於組織使用者、團隊和應用程式的[最佳作法解決方案手冊](https://{DomainName}/docs/tutorials?topic=solution-tutorials-users-teams-applications#users-teams-applications)。
- [使用 LogDNA 及 Sysdig分析日誌並監視應用程式性能](https://{DomainName}/docs/tutorials?topic=solution-tutorials-application-log-analysis#application-log-analysis)。
- 為 Kubernetes 中執行的容器化應用程式設定[持續整合和交付管線](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)。
- [在多個位置](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#multi-region-webapp)部署正式作業叢集。
- 使用[跨多個位置的多個叢集](https://{DomainName}/docs/containers?topic=containers-regions-and-zones#regions-and-locations)以實現高可用性。
