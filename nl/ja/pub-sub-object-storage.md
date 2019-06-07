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

# オブジェクト・ストレージとパブリッシュ/サブスクライブのメッセージングによる非同期データ処理
{: #pub-sub-object-storage}
このチュートリアルでは、Apache Kafka ベースのメッセージング・サービスを使用して、Kubernetes クラスターで実行するアプリケーションの実行時間の長いワークロードをオーケストレーションする方法を説明します。このパターンを使用してアプリケーションを分離すると、スケーリングおよびパフォーマンスの操作性がはるかに高まります。{{site.data.keyword.messagehub}} を使用すると、プロデューサー・アプリケーションに影響を与えることなく、実行する処理をキューに入れることができるので、実行時間の長いタスクにとって理想的なシステムになります。 

{:shortdesc}

ここでは、ファイル処理の例を使用してこのパターンをシミュレートします。まずは、UI アプリケーションを作成します。これは、ファイルをオブジェクト・ストレージにアップロードし、実行対象の処理を示すメッセージを生成します。次に、別のワーカー・アプリケーションを作成します。これは、ユーザーからアップロードされたファイルを、メッセージを受け取ったときに非同期で処理します。

## 達成目標
{: #objectives}

* {{site.data.keyword.messagehub}} を使用してプロデューサー/コンシューマーのパターンを実装する
* サービスを Kubernetes クラスターにバインドする

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/catalog/services/cloud-object-storage)
* [{{site.data.keyword.messagehub}}](https://{DomainName}/catalog/services/event-streams)
* [{{site.data.keyword.containershort_notm}}](https://{DomainName}/catalog/infrastructure/containers-kubernetes)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

このチュートリアルでは、このパターンの柔軟性を強調するために、UI アプリケーションは Node.js で作成し、ワーカー・アプリケーションは Java で作成しています。このチュートリアルでは両方のアプリケーションを同じ Kubernetes クラスターで実行しますが、どちらも、Cloud Foundry アプリケーションまたはサーバーレス機能として実装することもできるアプリケーションです。

<p style="text-align: center;">

   ![](images/solution25/Architecture.png)
</p>

1. ユーザーが UI アプリケーションを使用してファイルをアップロードします。
2. ファイルが {{site.data.keyword.cos_full_notm}} に保存されます。
3. 新しいファイルが処理を待っていることを示すメッセージが、{{site.data.keyword.messagehub}} トピックに送信されます。
4. ワーカーは準備ができたら、メッセージを listen し、新しいファイルの処理を開始します。

## 始める前に
{: #prereqs}

* [IBM Cloud Developer Tools](/docs/cli?topic=cloud-cli-install-ibmcloud-cli) - {{site.data.keyword.cloud_notm}} CLI、Kubernetes、Helm、および Docker をインストールするためのツール。

## Kubernetes クラスターを作成する
{: #create_kube_cluster}

1. [「カタログ」](https://{DomainName}/containers-kubernetes/launch)から Kubernetes クラスターを作成します。このチュートリアルに沿って作業しやすいように、`mycluster` という名前を付けてください。このチュートリアルは、**無料**クラスターを使用して実行できます。
   ![IBM Cloud での Kubernetes クラスターの作成](images/solution25/KubernetesClusterCreation.png)
2. **クラスター**と**ワーカー・ノード**の状況を確認し、**「準備完了 (Ready)」**になるまで待ちます。

### kubectl を構成する

この手順では、作成した新規クラスターを指すように、kubectl を構成します。[kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) は、Kubernetes クラスターの操作に使用するコマンド・ライン・ツールです。

1. `ibmcloud login` を使用して、対話式でログインします。クラスターを作成した組織、ロケーション、スペースを指定してください。詳細を再確認するには、`ibmcloud target` コマンドを実行します。
2. クラスターの準備ができたら、クラスター構成を取得します。
   ```bash
   ibmcloud cs cluster-config <cluster-name>
   ```
   {: pre}
3. 指示に従って **export** コマンドをコピーして貼り付け、KUBECONFIG 環境変数を設定します。KUBECONFIG 環境変数が適切に設定されたかどうかを確認するために、コマンド `echo $KUBECONFIG` を実行します。
  
4. `kubectl` コマンドが正しく構成されたことを確認します。
   ```bash
   kubectl cluster-info
   ```
  {: pre}
   ![](images/solution2/kubectl_cluster-info.png)

## {{site.data.keyword.messagehub}} インスタンスを作成する
 {: #create_messagehub}

{{site.data.keyword.messagehub}} は、高速でスケーラブルなフルマネージドのメッセージング・サービスです。Apache Kafka をベースとするオープン・ソースの高スループットのメッセージング・システムであり、リアルタイム・データ・フィードを処理するための低遅延のプラットフォームを提供します。

 1. ダッシュボードで[**「リソースの作成」**](https://{DomainName}/catalog/)をクリックし、「アプリケーション・サービス」セクションで[**「{{site.data.keyword.messagehub}}」**](https://{DomainName}/catalog/services/event-streams)を選択します。
 2. サービスに `mymessagehub` という名前を付け、**「作成」**をクリックします。
 3. サービス・インスタンスを `default` Kubernetes 名前空間にバインドして、サービス資格情報をクラスターに提供します。
 ```
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service mymessagehub
 ```

cluster-service-bind コマンドは、サービス・インスタンスの資格情報を JSON 形式で保持するクラスター・シークレットを作成します。`binding-mymessagehub` という名前の生成されたシークレットを参照するには、`kubectl get secrets `を使用します。詳細については、[サービスの統合](https://{DomainName}/docs/containers?topic=containers-integrations#integrations)を参照してください。

{:tip}

## オブジェクト・ストレージ・インスタンスを作成する

{: #create_cos}

{{site.data.keyword.cos_full_notm}} は、暗号化され、複数の地理的ロケーションに分散されます。アクセスは、REST API を使用して HTTP で行います。{{site.data.keyword.cos_full_notm}} は、非構造化データのための柔軟でコスト効率が高いスケーラブルなクラウド・ストレージです。これを使用して、UI でアップロードされたファイルを保管します。

1. ダッシュボードで[**「リソースの作成」**](https://{DomainName}/catalog/)をクリックし、「ストレージ」セクションで[**「{{site.data.keyword.cos_short}}」**](https://{DomainName}/catalog/services/cloud-object-storage)を選択します。
2. サービスに `myobjectstorage` という名前を付け、**「作成」**をクリックします。
3. **「バケットの作成 (Create bucket)」**をクリックします。 
4. バケット名に固有の名前 (`username-mybucket` など) を設定します。
5. 耐障害性に対して**「クロス・リージョン (Cross Region)」**を選択し、ロケーションに対して**「米国地域 (us-geo)」 **を選択して、 **「作成」**をクリックします。
6. サービス・インスタンスを `default` Kubernetes 名前空間にバインドして、サービス資格情報をクラスターに提供します。
 ```sh
 ibmcloud resource service-alias-create myobjectstorage --instance-name myobjectstorage
 ibmcloud cs cluster-service-bind --cluster mycluster --namespace default --service myobjectstorage
 ```
![](images/solution25/cos_bucket.png)

## UI アプリケーションをクラスターにデプロイする

この UI アプリケーションは、ユーザーがファイルをアップロードできる Node.js Express の単純な Web アプリケーションです。これが、上記で作成したオブジェクト・ストレージ・インスタンスにファイルを保管します。そして、新しいファイルが処理できる状態であることを示すメッセージを、{{site.data.keyword.messagehub}} トピック「work-topic」に送信します。

1. サンプル・アプリケーション・リポジトリーをローカルに複製し、ディレクトリーを `pubsub-ui` フォルダーに移動します。
```sh
  git clone https://github.com/IBM-Cloud/pub-sub-storage-processing
  cd pub-sub-storage-processing/pubsub-ui
```
2. `config.js` を開き、バケット名を指定して COSBucketName を更新します。
3. アプリケーションをビルドしてデプロイします。deploy コマンドは、Docker イメージを生成し、{{site.data.keyword.registryshort_notm}} にプッシュしてから、Kubernetes デプロイメントを作成します。対話式の指示に従って、アプリをデプロイしてください。
```sh
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
4. アプリケーションにアクセスし、`sample-files` フォルダーにあるファイルをアップロードします。アップロードされたファイルがオブジェクト・ストレージに保管され、状況が「待機中 (awaiting)」になります。これは、ファイルがワーカー・アプリケーションで処理されるまで変わりません。このブラウザーのウィンドウは開いたままにしてください。

   ![](images/solution25/files_uploaded.png)

## ワーカー・アプリケーションをクラスターにデプロイする

このワーカー・アプリケーションは、{{site.data.keyword.messagehub}} Kafka の「work-topic」トピックのメッセージを listen する Java アプリケーションです。新規メッセージが届くと、ワーカーはそのメッセージからファイルの名前を取得し、オブジェクト・ストレージからファイルの内容を取得します。それから、ファイルの処理をシミュレートし、完了したら、別のメッセージを「result-work」トピックに送信します。UI アプリケーションは、このトピックを listen し、状況を更新します。

1. `pubsub-worker` ディレクトリーに移動します。
```sh
  cd ../pubsub-worker
```
2. `resources/cos.properties` を開き、バケット名を指定して `bucket.name` プロパティーを更新します。
2. ワーカー・アプリケーションをビルドしてデプロイします。
```
  ibmcloud dev build
  ibmcloud dev deploy -t container
```
3. デプロイメントが完了したら、また Web アプリケーションでブラウザー・ウィンドウを確認します。各ファイルの隣の状況が、「処理済み (processed)」に変わったことに注目してください。
![](images/solution25/files_processed.png)

このチュートリアルでは、Kafka ベースの {{site.data.keyword.messagehub}} を使用して、プロデューサー/コンシューマーのパターンを実装する方法を学習しました。このパターンにより、Web アプリケーションを高速化し、重い処理を他のアプリケーションにオフロードできるようになります。処理を実行する必要が生じると、プロデューサー (Web アプリケーション) がメッセージを作成します。そして、メッセージをサブスクライブする 1 つ以上のワーカーの間で処理がロード・バランシングされます。Kubernetes で実行される Java アプリケーションを処理の実行に使用しましたが、このようなアプリケーションは [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_common_use_cases#data-processing) にすることもできます。実行時間が長くて負荷の高いワークロードには、Kubernetes で実行されるアプリケーションが理想的です。一方、実行時間が短い処理には {{site.data.keyword.openwhisk_short}} が適しています。

## リソースの削除
{:removeresources}

[「リソース・リスト」](https://{DomainName}/resources/)にナビゲートします。
1. Kubernetes クラスター `mycluster` を削除します。
2. {{site.data.keyword.cos_full_notm}} `myobjectstorage` を削除します。
3. {{site.data.keyword.messagehub}} `mymessagehub` を削除します。
4. 左側のメニューから**「Kubernetes」**を選択し、**「レジストリー (Registry)」**を選択してから、`pubsub-xxx` リポジトリーを削除します。

## 関連コンテンツ
{:related}

* [{{site.data.keyword.cos_full_notm}}](https://{DomainName}/docs/services/cloud-object-storage?topic=cloud-object-storage-about-ibm-cloud-object-storage#about-ibm-cloud-object-storage)
* [{{site.data.keyword.messagehub_full}}](https://{DomainName}/docs/services/EventStreams?topic=eventstreams-getting_started#messagehub)
* [オブジェクト・ストレージへのアクセスの管理](https://{DomainName}/docs/infrastructure/cloud-object-storage-infrastructure?topic=cloud-object-storage-infrastructure-managing-access#managing-access)
* [{{site.data.keyword.openwhisk_short}} による {{site.data.keyword.messagehub}} データの処理 ](https://github.com/IBM/openwhisk-data-processing-message-hub)
