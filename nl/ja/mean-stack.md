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


# MEAN スタックを使用した最新の Web アプリケーション
{: #mean-stack}

このチュートリアルでは、一般的な MEAN スタックを使用した Web アプリケーションの作成について説明します。これは、**M**ongo DB、**E**xpress Web フレームワーク、**A**ngular フロントエンド・フレームワークおよび Node.js ランタイムで構成されます。ここでは、MEAN スターターをローカルで実行する方法、管理対象 Database as a Service (DBasS) を作成して使用する方法、アプリを {{site.data.keyword.cloud_notm}} にデプロイしてアプリケーションをモニターする方法について説明します。  

## 達成目標

{: #objectives}

- スターター Node.js アプリをローカルで作成して実行します。
- 管理対象 Database as a Service (DBasS) を作成します。
- Node.js アプリをクラウドにデプロイします。
- MongoDB リソースをスケーリングします。
- アプリケーション・パフォーマンスをモニターする方法について学習します。

## 使用するサービス

{: #products}

このチュートリアルでは、以下の {{site.data.keyword.Bluemix_notm}} サービスを使用します。

- [{{site.data.keyword.composeForMongoDB}}](https://{DomainName}/catalog/services/compose-for-mongodb)
- [{{site.data.keyword.runtime_nodejs_notm}}](https://{DomainName}/catalog/starters/sdk-for-nodejs)

**注意:** このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー

{:#architecture}

<p style="text-align: center;">

![アーキテクチャー図](images/solution7/Architecture.png)</p>

1. ユーザーが Web ブラウザーを使用してアプリケーションにアクセスします。
2. Node.js アプリが {{site.data.keyword.composeForMongoDB}} データベースに接続してデータをフェッチします。

## 始める前に

{: #prereqs}

1. [Git のインストール](https://git-scm.com/)
2. [{{site.data.keyword.Bluemix_notm}} CLI のインストール](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)


ローカルでアプリケーションを開発して実行するために:
1. [Node.js と NPM のインストール](https://nodejs.org/)
2. [MongoDB Community Edition のインストールと実行](https://docs.mongodb.com/manual/administration/install-community/)

## MEAN アプリのローカルでの実行

{: #runapplocally}

このセクションでは、ローカルの MongoDB データベースを実行し、MEAN サンプル・コードを複製して、ローカルでアプリケーションを実行してローカルの MongoDB データベースを使用します。

{: shortdesc}

1. [こちら](https://docs.mongodb.com/manual/administration/install-community/)の指示に従って、ローカルに MongoDB データベースをインストールし、実行します。インストールが完了した後、以下のコマンドを使用して、**mongod** サーバーが実行されていることを確認します。using the  Confirm your database is running with the following command.
  ```sh
  mongo
  ```
  {: codeblock}

2. MEAN スターター・コードを複製します。

  ```sh
  git clone https://github.com/IBM-Cloud/nodejs-MEAN-stack
  cd nodejs-MEAN-stack
  ```
  {: codeblock}

3. 必要なパッケージをインストールします。

  ```sh
  npm install
  ```
  {: codeblock}

4. .env.example ファイルを .env にコピーします。必要な情報を編集します。少なくとも、独自の SESSION_SECRET を追加します。

5. node server.js を実行してアプリを開始します。
  ```sh
  node server.js
  ```
  {: codeblock}

6. アプリケーションにアクセスし、新しいユーザーを作成してログインします。

## クラウドでの MongoDB データベースのインスタンスの作成

{: #createdatabase}

このセクションでは、{{site.data.keyword.composeForMongoDB}} データベースをクラウドに作成します。{{site.data.keyword.composeForMongoDB}} は Database as a Service で、一般的に構成が容易で、バックアップとスケーリングが組み込まれています。[IBM Cloud カタログ](https://{DomainName}/catalog/?category=data)で多種多様なデータベースが提供されています。{{site.data.keyword.composeForMongoDB}}を作成するには、以下の手順を実行します。

{: shortdesc}

1. コマンド・ライン経由で {{site.data.keyword.cloud_notm}} アカウントにログインし、{{site.data.keyword.cloud_notm}} アカウントをターゲットします。 

  ```sh
  ibmcloud login
  ibmcloud target --cf
  ```
  {: codeblock}

  より多くの CLI コマンドは [こちら](/docs/cli?topic=cloud-cli-install-ibmcloud-cli)を参照してください。

2. {{site.data.keyword.composeForMongoDB}} のインスタンスを作成します。このことは、[コンソール UI](https://{DomainName}/catalog/services/compose-for-mongodb) を使用して実行することもできます。サービス名は、**mean-starter-mongodb** にする必要があります。これは、アプリケーションがこの名前でサービスを探すように構成されているためです。

  ```sh
  ibmcloud cf create-service compose-for-mongodb Standard mean-starter-mongodb
  ```
  {: codeblock}

## クラウドへのアプリのデプロイ

{: #deployapp}

このセクションでは、管理対象 MongoDB データベースを使用していた {{site.data.keyword.cloud_notm}} に node.js アプリをデプロイします。ソース・コードには、以前に作成された「mongodb」サービスを使用するように構成された、[**manifest.yml**](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/manifest.yml) ファイルが含まれています。このアプリケーションでは VCAP_SERVICES 環境変数を使用して Compose MongoDB データベース資格情報にアクセスします。このことは、[server.js ファイル](https://github.com/IBM-Cloud/nodejs-MEAN-stack/blob/master/server.js)で参照できます。 

{: shortdesc}

1. コードをクラウドにプッシュします。

   ```sh
   ibmcloud cf push
   ```

   {: codeblock}

2. コードがプッシュされた後、ブラウザーでアプリを表示できます。`https://mean-random-name.mybluemix.net` のような、ランダムなホスト名が生成されます。アプリケーション URL は、コンソール・ダッシュボードかコマンド・ラインから取得できます。![Live App](images/solution7/live-app.png)


## MongoDB データベース・リソースのスケーリング
{: #scaledatabase}

サービスが追加ストレージを必要とする場合や、サービスに割り振られたストレージの量を減らす場合は、リソースをスケーリングします。

{: shortdesc}

1. コンソール・**ダッシュボード**を使用して、**「接続」**セクションに移動し、**MongoDB インスタンス**のデータベースをクリックします。
2. **「デプロイメントの詳細 (deployment details)」**パネルで、**「リソースのスケーリング (scale resources)」**オプションをクリックします。
  ![](images/solution7/mongodb-scale-show.png)
3. **スライダー**を調整して、{{site.data.keyword.composeForMongoDB}} データベース・サービスに割り振られたストレージを増減させます。
4. **「デプロイメントのスケーリング」**をクリックして、スケール変更をトリガーし、ダッシュボード概要に戻ります。「スケーリングが開始された」というメッセージがページの上部に表示され、スケール変更が進行中であることが示されます。
  ![](images/solution7/scaling-in-progress.png)


## アプリケーション・パフォーマンスのモニター
{: #monitorapplication}

アプリケーションの正常性を確認するために、組み込みの Availability Monitoring サービスを使用できます。Availability Monitoring サービスは、クラウド内のアプリケーションに自動的に接続されます。Availability Monitoring サービスでは、訪問者への影響が発生する前に、パフォーマンスの問題を予防的に検出して修正するために、世界中の場所から 24 時間合成テストを実行します。モニター・ダッシュボードに移動するには、以下の手順を実行します。

{: shortdesc}

1. コンソール・ダッシュボードを使用して、アプリケーションの下で、**「モニタリング」**タブを選択します。
2. **「すべてのテストの表示」**をクリックしてテストを表示します。
   ![](images/solution7/alert_frequency.png)


## 関連コンテンツ

{: #related}

- ソース管理と[継続的デリバリー](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp#devops)のセットアップ。
- [複数の場所](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-webapp)を横断するセキュアな Web アプリケーション。
- [REST API](https://{DomainName}/docs/tutorials?topic=solution-tutorials-create-manage-secure-apis#create-manage-secure-apis) の作成、保護および管理。
