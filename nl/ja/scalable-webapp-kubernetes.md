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

# Kubernetes のスケーラブルな Web アプリケーション
{: #scalable-webapp-kubernetes}

このチュートリアルでは、Web アプリケーションを構築し、ローカル環境のコンテナー内で実行してから、[{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster) で作成した Kubernetes クラスターにデプロイする手順を説明します。また、カスタム・ドメインをバインドし、環境の正常性を監視し、アプリケーションを拡張する方法についても説明します。
{:shortdesc}

コンテナーは、環境間でアプリをシームレスに移動できるようにアプリとそのすべての依存関係をパッケージ化するための標準的な手段です。 仮想マシンとは異なり、コンテナーはオペレーティング・システムをバンドルしません。 アプリのコード、ランタイム、システム・ツール、ライブラリー、設定値のみがコンテナー内にパッケージされます。 コンテナーは、仮想マシンより軽量で移植しやすく、効率的です。

プロジェクトの促進を求めている開発者のために、{{site.data.keyword.dev_cli_notm}} CLI は、テンプレート・アプリケーションを生成して迅速にアプリケーションを開発、デプロイできるようになっています。テンプレート・アプリケーションは、すぐに実行することも、独自のソリューションのスターターとしてカスタマイズすることもできます。スターターのアプリケーション・コード、Docker コンテナー・イメージ、および CloudFoundry 資産を生成できるだけでなく、dev CLIおよび Web コンソールで使用されるコード生成プログラムは、[Kubernetes](https://kubernetes.io/) 環境へのデプロイメントを支援するためのファイルも生成できます。テンプレートは [Helm](https://github.com/kubernetes/helm) チャートを生成します。これはアプリケーションの初期 Kubernetes デプロイメント構成を記述したもので、必要に応じて、容易に拡張して、マルチイメージのデプロイメントや複雑なデプロイメントを作成することができます。

## 達成目標
{: #objectives}

* スターター・アプリケーションを構築する。
* Kubernetes クラスターにアプリケーションをデプロイする。
* カスタム・ドメインをバインドする。
* クラスターのログと正常性をモニターする。
* Kubernetes ポッドをスケーリングする。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
* [{{site.data.keyword.registrylong_notm}}](https://{DomainName}/containers-kubernetes/launchRegistryView)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/containers-kubernetes/catalog/cluster)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー](images/solution2/Architecture.png)
</p>

1. 開発者が {{site.data.keyword.dev_cli_notm}} を使用して、スターター・アプリケーションを生成します。
1. アプリケーションをビルドすることで、Docker コンテナー・イメージが生成されます。
1. イメージが、{{site.data.keyword.containershort_notm}} の名前空間にプッシュされます。
1. アプリケーションが Kubernetes クラスターにデプロイされます。
1. ユーザーがアプリケーションにアクセスします。

## 始める前に
{: #prereqs}

* [{{site.data.keyword.registrylong_notm}} CLI およびレジストリー名前空間をセットアップします](https://{DomainName}/docs/services/Registry?topic=registry-registry_setup_cli_namespace#registry_setup_cli_namespace)
* [{{site.data.keyword.dev_cli_notm}}](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) をインストールします (docker、kubectl、helm、ibmcloud cli、および必要なプラグインをインストールするスクリプト)
* [Kubernetes の基礎を理解します](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

## Kubernetes クラスターを作成する
{: #create_kube_cluster}

{{site.data.keyword.containershort_notm}} は、Docker と Kubernetes テクノロジーを結合させた強力なツール、直観的なユーザー・エクスペリエンス、標準装備のセキュリティーと分離機能を提供します。これらの機能を使用することで、コンピュート・ホストから成るクラスター内でコンテナー化アプリのデプロイメント、操作、スケーリング、モニタリングを自動化することができます。

このチュートリアルの大部分は、**無料**クラスターを使用して実行できます。Kubernetes Ingress とカスタム・ドメインに関連した 2 つのオプションのセクションについては、**標準**タイプの**有料**クラスターが必要です。

1. [{{site.data.keyword.Bluemix}} カタログ](https://{DomainName}/containers-kubernetes/launch)から Kubernetes クラスターを作成します。

   円滑に作業するため、ライト・プランと標準プランで使用できる構成の詳細 (CPU の数、メモリー、ワーカー・ノードの数など) を確認してください。
   {:tip}

   ![IBM Cloud での Kubernetes クラスターの作成](images/solution2/KubernetesClusterCreation.png)
2. **「クラスター・タイプ」**を選択し、**「クラスターの作成」**をクリックして、Kubernetes クラスターをプロビジョンします。
3.  **クラスター**と**ワーカー・ノード**の状況を確認し、**「準備完了 (Ready)」**になるまで待ちます。

### kubectl を構成する

この手順では、作成した新規クラスターを指すように、kubectl を構成します。[kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) は、Kubernetes クラスターの操作に使用するコマンド・ライン・ツールです。

1. `ibmcloud login` を使用して、対話式でログインします。クラスターを作成した組織、ロケーション、スペースを指定してください。詳細を再確認するには、`ibmcloud target` コマンドを実行します。
2. クラスターの準備ができたら、MYCLUSTER 環境変数にクラスター名を設定して、クラスター構成を取得します。
   ```bash
   export MYCLUSTER=<CLUSTER_NAME>
   ibmcloud cs cluster-config ${MYCLUSTER}
   ```
   {: pre}
3. 指示に従って **export** コマンドをコピーして貼り付け、KUBECONFIG 環境変数を設定します。KUBECONFIG 環境変数が適切に設定されたかどうかを確認するために、コマンド `echo $KUBECONFIG` を実行します。
  
4. `kubectl` コマンドが正しく構成されたことを確認します。
   ```bash
   kubectl cluster-info
   ```
   {: pre}
   ![](images/solution2/kubectl_cluster-info.png)


## スターター・アプリケーションを作成する
{: #create_application}

`ibmcloud dev` ツールを使用すると、アプリケーション・スターターと必要なすべてのボイラープレート、ビルド、構成コードを一緒に生成してビジネス・ロジックのコーディングを迅速に開始できるので、開発時間が大幅に短縮されます。

1. `ibmcloud dev` ウィザードを開始します。
   ```
   ibmcloud dev create
   ```
   {: pre}

1. `「バックエンド・サービス / Web アプリ (Backend Service / Web App)」`>`「Java - MicroProfile / JavaEE」`>`「Java Web アプリと Eclipse MicroProfile および Java EE (Web アプリ) (Java Web App with Eclipse MicroProfile and Java EE (Web App))」`の順に選択して、Java スターターを作成します。(代わりに Node.js スターターを作成するには、`「バックエンド・サービス / Web アプリ (Backend Service / Web App)」`>`「ノード (Node)」`>`「Node.js Web アプリと Express.js (Web アプリ) (Node.js Web App with Express.js (Web App))」`を使用します)
1. アプリケーションの**名前**を入力します。
1. このアプリケーションをデプロイするリソース・グループを選択します。
1. 他のサービスは追加しないでください。
1. DevOps ツールチェーンを追加しないでください。**「手動デプロイメント (manual deployment)」**を選択します。

この結果、ローカル開発と Cloud Foundry または Kubernetes 上のクラウドへのデプロイメントのためのコードおよび必要なすべての構成ファイルを完全に備えた、スターター・アプリケーションが生成されます。 

<!-- For an overview of the files generated, see [Project Contents Documentation](https://{DomainName}/docs/cloudnative/projects/java_project_contents.html#java-project-files). -->

![](images/solution2/Contents.png)

### アプリケーションをビルドする

Java ローカル開発用の `mvn` またはノード開発用の `npm` を使用して、通常どおりにアプリケーションをビルドして実行できます。ローカルとクラウドで実行の一貫性を確保するために、Docker イメージをビルドし、コンテナー内のアプリケーションを実行することもできます。Docker イメージをビルドするには、以下の手順に従います。

1. ローカルの Docker エンジンが開始されていることを確認します。
   ```
   docker ps
   ```
   {: pre}
2. 生成したプロジェクトのディレクトリーに移動します。
   ```
   cd <project name>
   ```
   {: pre}
3. アプリケーションをビルドします。
   ```
   ibmcloud dev build
   ```
   {: pre}

   すべてのアプリケーション依存関係がダウンロードされ、Docker イメージ (アプリケーションと必要なすべての環境を含む) がビルドされるので、この実行には数分かかることがあります。

### ローカルでアプリケーションを実行する

1. コンテナーを実行します。
   ```
   ibmcloud dev run
   ```
   {: pre}

   これには、前のステップでビルドした Docker イメージを実行するためのローカルの Docker エンジンが使用されます。
2. コンテナーが開始されたら、`http://localhost:9080/` にアクセスします。Node.js アプリケーションを作成した場合は、`http://localhost:3000/` にアクセスします。
  ![](images/solution2/LibertyLocal.png)

## Helm チャートを使用してアプリケーションをクラスターにデプロイする
{: #deploy}

このセクションでは、まず Docker イメージを IBM Cloud プライベート・コンテナー・レジストリーにプッシュしてから、そのイメージを指す Kubernetes デプロイメントを作成します。

1. レジストリーの名前空間をすべてリストして、自分の**名前空間**を見つけます。
   ```sh
   ibmcloud cr namespaces
   ```
   {: pre}
   名前空間がある場合は、後で使用できるように名前をメモします。ない場合は、作成します。
   ```sh
   ibmcloud cr namespace-add <Name>
   ```
   {: pre}
2. MYNAMESPACE 環境変数と MYPROJECT 環境変数に名前空間とプロジェクト名をそれぞれ設定します。

    ```sh
    export MYNAMESPACE=<NAMESPACE>
    ```
    {: pre}
    ```sh
    export MYPROJECT=<PROJECT_NAME>
    ```
    {: pre}
3. `ibmcloud cr info` を実行して、**コンテナー・レジストリー** (例: us.icr.io) を確認します。
4. MYREGISTRY 環境変数にレジストリーを設定します。
   ```sh
   export MYREGISTRY=<REGISTRY>
   ```
   {: pre}

5. Docker イメージをビルドしてタグを付けます (`-t`)。
   ```sh
   docker build . -t ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}

6. Docker イメージを、IBM Cloud 上のコンテナー・レジストリーにプッシュします。
   ```sh
   docker push ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}:v1.0.0
   ```
   {: pre}
7. IDE で、`chart\YOUR PROJECT NAME` の下にある **values.yaml** にナビゲートし、IBM Cloud コンテナー・レジストリー上のイメージを指す**イメージ・リポジトリー**の値を更新します。ファイルを**保存**します。

   イメージ・リポジトリーの詳細については、`echo ${MYREGISTRY}/${MYNAMESPACE}/${MYPROJECT}` を実行してください。

8. [Helm](https://helm.sh/) を使用すれば、Helm チャートで Kubernetes アプリケーションを管理できます。非常に複雑な Kubernetes アプリケーションでも Helm チャートで定義、インストール、アップグレードできます。`chart\YOUR PROJECT NAME` にナビゲートし、クラスターで以下のコマンドを実行して Helm を初期化します。

   ```bash
   helm init
   ```
   {: pre}
   Helm をアップグレードするには、コマンド `helm init --upgrade` を実行します。
   {:tip}

9. Helm チャートをインストールするには、`chart\YOUR PROJECT NAME` ディレクトリーに移動し、以下のコマンドを実行します。
  ```sh
  helm install . --name ${MYPROJECT}
  ```
  {: pre}
10. Java アプリケーションの場合は `kubectl get service ${MYPROJECT}-service`、Node.js アプリケーションの場合は `kubectl get service ${MYPROJECT}-application-service` を使用して、サービスが listen しているパブリック・ポートを調べます。ポートは、`PORT(S)` 下の 5 桁の数値 (例: 31569) です。
11. ワーカー・ノードのパブリック IP を調べるために、以下のコマンドを実行します。
   ```sh
   ibmcloud cs workers ${MYCLUSTER}
   ```
   {: pre}
12. `http://worker-ip-address:portnumber/` でアプリケーションにアクセスします。

## クラスターで IBM 提供ドメインを使用する
{: #ibm_domain}

前の手順では、非標準のポートを使用してアプリケーションにアクセスしました。Kubernetes の NodePort 機能によってサービスが公開されました。

有料クラスターには、IBM 提供のドメインが用意されています。そのため、適切な URL を使用して標準的な HTTP/S ポートでアプリケーションを公開することができます。

Ingress を使用して、サービスへのクラスター・インバウンド接続をセットアップします。

![Ingress](images/solution2/Ingress.png)

1. IBM 提供の **Ingress ドメイン**を調べます。
   ```
   ibmcloud cs cluster-get ${MYCLUSTER}
   ```
   {: pre}
   次を見つけます。
   ```
   Ingress subdomain:	mycluster.us-south.containers.appdomain.cloud
   Ingress secret:		mycluster
   ```
   {: screen}
2. HTTP と HTTPS をサポートしているドメインを指す Ingress ファイル `ingress-ibmdomain.yml` を作成します。以下のファイルをテンプレートとして使用してください。<> で囲まれたすべての値を、上記の出力中の該当する値に置き換えます。**service-name** は、上記の手順で `==> v1/Service` の下に表示された名前です。あるいは、`kubectl get svc` を使用して、タイプ **NodePort** のサービス名を確認します。
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
3. Ingress をデプロイします。
   ```sh
   kubectl apply -f ingress-ibmdomain.yml
   ```
   {: pre}
4. `https://<nameofproject>.<ingress-sub-domain>/` でアプリケーションにアクセスします。

## 独自のカスタム・ドメインを使用する
{: #custom_domain}

カスタム・ドメインを使用するには、IBM 提供のドメインを指す CNAME レコードを使用するか、または IBM 提供の Ingress のポータブル・パブリック IP アドレスを指す A レコードを使用して、DNS レコードを更新する必要があります。固定 IP アドレスが用意されている有料クラスターの場合は、A レコードを選択することをお勧めします。

詳しくは、[カスタム・ドメインで Ingress コントローラーを使用する](https://{DomainName}/docs/containers?topic=containers-ingress#ingress)を参照してください。

### HTTP の使用

1. ドメインを指す Ingress ファイル `ingress-customdomain-http.yml` を作成します。
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
2. Ingress をデプロイします。
   ```sh
   kubectl apply -f ingress-customdomain-http.yml
   ```
   {: pre}
3. `http://<customdomain>/` でアプリケーションにアクセスします。

### HTTPS の使用

現時点で HTTPS を使用して `https://<customdomain>/` でアプリケーションにアクセスしようとすると、接続がプライベートではないことを知らせるセキュリティー警告が Web ブラウザーに表示されるはずです。また、構成したばかりの Ingress は HTTPS トラフィックの送信方法を判別できないので、404 も返されるはずです。

1. ドメイン用に信頼できる SSL 証明書を取得します。証明書と鍵が必要になります。
  https://{DomainName}/docs/containers?topic=containers-ingress#public_inside_3
   [Let's Encrypt](https://letsencrypt.org/) を使用して、信頼できる証明書を生成できます。
2. 証明書と鍵を base64 の ASCII 形式のファイルに保存します。
3. 証明書と鍵を格納する TLS シークレットを作成します。
   ```
   kubectl create secret tls my-custom-domain-secret-name --cert=<custom-domain.cert> --key=<custom-domain.key>
   ```
   {: pre}
4. ドメインを指す Ingress ファイル `ingress-customdomain-https.yml` を作成します。
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
5. Ingress をデプロイします。
   ```
   kubectl apply -f ingress-customdomain-https.yml
   ```
   {: pre}
6. `https://<customdomain>/` でアプリケーションにアクセスします。

## アプリケーションの正常性をモニターする
{: #monitor_application}

1. アプリケーションの正常性を確認するには、[クラスター](https://{DomainName}/containers-kubernetes/clusters)にナビゲートしてクラスターのリストを表示し、前述の手順で作成したクラスターをクリックします。
2. **「Kubernetes ダッシュボード (Kubernetes Dashboard)」**をクリックして、新規タブでダッシュボードを起動します。
   ![](images/solution2/launch_kubernetes_dashboard.png)
3. 左側のペインで**「ノード」**を選択し、ノードの**「名前」**をクリックし、**「割り振りリソース (Allocation Resources)」**を参照してノードの正常性を確認します。
   ![](images/solution2/KubernetesDashboard.png)
4. コンテナーのアプリケーション・ログを確認するには、**「ポッド (Pods)」**、**ポッド名**、**「ログ」**を選択します。
5. コンテナーに対して **ssh** を使用するには、前の手順で確認したポッド名を指定して、次のように実行します。
   ```sh
   kubectl exec -it <pod-name> -- bash
   ```
   {: pre}

## Kubernetes ポッドをスケーリングする
{: #scale_cluster}

アプリケーションの負荷が増加したら、デプロイメントのポッド・レプリカの数を手動で増やすことができます。レプリカは [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) によって管理されます。アプリケーションを 2 つのレプリカにスケーリングするには、次のコマンドを実行します。

   ```sh
 kubectl scale deployment <nameofproject>-deployment --replicas=2
   ```
   {: pre}

しばらくすると、Kubernetes ダッシュボードにアプリケーションの 2 つのポッドが表示されます (`kubectl get pods` を使用することもできます)。クラスターの Ingress コントローラーが、2 つのレプリカ間のロード・バランシングを処理します。水平スケーリングは自動的に行うこともできます。

手動および自動のスケーリングについて詳しくは、Kubernetes の資料を参照してください。

   * [Scaling a deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)
   * [Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)

## リソースの削除

* クラスターを削除します。クラスターを再使用する場合は、アプリケーションのために作成した Kubernetes 成果物のみを削除します。

## 関連コンテンツ

* [IBM Cloud Kubernetes Service](https://{DomainName}/docs/containers?topic=containers-container_index#container_index)
<!-- * [IBM Cloud App Service](https://{DomainName}/docs/cloudnative/index.html#web-mobile) -->
* [Kubernetes への継続的デプロイメント](https://{DomainName}/docs/tutorials?topic=solution-tutorials-continuous-deployment-to-kubernetes#continuous-deployment-to-kubernetes)
