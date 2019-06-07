---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-07"
---

{:java: #java .ph data-hd-programlang='java'}
{:swift: #swift .ph data-hd-programlang='swift'}
{:ios: data-hd-operatingsystem="ios"}
{:android: data-hd-operatingsystem="android"}
{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# サーバーレス・バックエンドを備えたモバイル・アプリケーション
{: #serverless-mobile-backend}

このチュートリアルでは、{{site.data.keyword.openwhisk}} とコグニティブ・サービスおよびデータ・サービスを使用して、モバイル・アプリケーションのサーバーレス・バックエンドを構築する方法を学習します。
{:shortdesc}

すべてのモバイル開発者がサーバー・サイド・ロジックの管理や、そもそものサーバーの管理を経験しているわけではありません。開発者は構築するアプリに取り組むことを優先します。それでは、開発者が既に持っている開発スキルを再利用して、モバイル・バックエンドを作成するとしたらどうでしょうか。

{{site.data.keyword.openwhisk_short}} は、サーバーレス・イベント・ドリブン・プラットフォームです。[この例で強調されているように](https://{DomainName}/docs/tutorials?topic=solution-tutorials-serverless-api-webapp#serverless-api-webapp)、デプロイするアクションは、Web アプリケーション・バックエンド API を構築する *Web アクション*として HTTP エンドポイントに容易に変換できます。Web アプリケーションが REST API のクライアントであるため、この例からさらに踏み込んで、同じ手法を使用してモバイル・アプリのバックエンドを容易に構築できます。また、モバイル開発者は  {{site.data.keyword.openwhisk_short}} を使用して、モバイル・アプリと同じ言語、Java (Android の場合)、および Swift (iOS の場合) でアクションを作成できます。

このチュートリアルは、各自のターゲット・プラットフォームに基づいて構成可能です。現在ご覧になっているのは、このチュートリアルの **iOS / Swift** バージョンの資料です。このチュートリアルの **Android / Java** バージョンを選択するには、この文書の上部にあるタブを使用します。
{: swift}

このチュートリアルは、各自のターゲット・プラットフォームに基づいて構成可能です。現在ご覧になっているのは、このチュートリアルの **Android / Java** バージョンの資料です。このチュートリアルの **iOS / Swift** バージョンを選択するには、この文書の上部にあるタブを使用します。
{: java}

## 達成目標
{: #objectives}

* {{site.data.keyword.openwhisk_short}} を使用してサーバーレス・モバイル・バックエンドをデプロイする。
* {{site.data.keyword.appid_short}} を使用してモバイル・アプリにユーザー認証を追加する。
* {{site.data.keyword.toneanalyzershort}} を使用してユーザー・フィードバックを分析する。
* {{site.data.keyword.mobilepushshort}} を使用して通知を送信する。

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk)
   * [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID)
   * [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB)
   * [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer)
   * [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush)

このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

このチュートリアルに示されているアプリケーションは、フィードバック・テキストのトーンをスマートに分析し、{{site.data.keyword.mobilepushshort}} で顧客を適切に認識するフィードバック・アプリです。

<p style="text-align: center;">
![](images/solution11/Architecture.png)
</p>

1. ユーザーが [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) に対して認証を行います。{{site.data.keyword.appid_short}} からアクセス・トークンと ID トークンが提供されます。
2. それ以降のバックエンド API 呼び出しにはこのアクセス・トークンが含まれます。
3. [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk) を使用してバックエンドが実装されます。 Web アクションとして公開されるサーバーレス・アクションは、要求ヘッダーに入れられてトークンが送信されることを必要とし、実際の API へのアクセスを許可する前にトークンの妥当性 (署名と有効期限日) を検証します。
4. ユーザーがフィードバックを送信すると、そのフィードバックは [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) に保管されます。
5. [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) を使用して、フィードバックのテキストが処理されます。
6. 分析結果に基づき、[{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) によって通知がユーザーに返されます。
7. ユーザーはこの通知をデバイスで受信します。

## 始める前に
{: #prereqs}

このチュートリアルでは、リソースのプロビジョンとコードのデプロイに {{site.data.keyword.Bluemix_notm}} コマンド・ライン・ツールを使用します。`ibmcloud` コマンド・ライン・ツールを必ずインストールしてください。

* [{{site.data.keyword.Bluemix_notm}} 開発者用ツール](https://github.com/IBM-Cloud/ibm-cloud-developer-tools) - ibmcloud CLI と必須プラグインをインストールするスクリプト (Cloud Foundry および{{site.data.keyword.openwhisk_short}})

また、次のソフトウェアとアカウントも必要です。

   1. Java 8
   2. Android Studio 2.3.3
   3. Google Developer アカウント (Firebase Cloud Messaging を構成するため)
   4. Bash シェル、cURL
   {: java}


   1. Xcode
   2. Apple Developer アカウント (Apple Push Notification Service を構成するため)
   3. Bash シェル、cURL
   {: swift}

このチュートリアルでは、アプリケーションのプッシュ通知を構成します。このチュートリアルは、ユーザーが [Android](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#android-mobile-push-analytics) または [iOS](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#ios-mobile-push-analytics) 向けの基本的な {{site.data.keyword.mobilepushshort}} チュートリアルを完了しており Firebase Cloud Messaging または Apple Push Notification Service の構成に慣れていることを前提としています。
{:tip}

Windows 10 ユーザーがコマンド・ラインを使用する場合には、[この記事](https://msdn.microsoft.com/en-us/commandline/wsl/install-win10)の説明に従って Windows Subsystem for Linux と Ubuntu をインストールすることをお勧めします。
{: tip}
{: java}

## アプリケーション・コードの取得

リポジトリーには、モバイル・アプリケーション・コードと {{site.data.keyword.openwhisk_short}} アクション・コードの両方が含まれています。

1. GitHub リポジトリーからコードをチェックアウトします。

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-android
   ```
   {: pre}
   {: java}

   ```sh
   git clone https://github.com/IBM-Cloud/serverless-followupapp-ios
   ```
   {: pre}
   {: swift}

2. コードの構造を確認します。

| ファイル                                     | 説明                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/actions) | サーバーレス・モバイル・バックエンドの {{site.data.keyword.openwhisk_short}} アクションのコード |
| [**android**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/android) | モバイル・アプリケーションのコード          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-android/tree/master/deploy.sh) | {{site.data.keyword.openwhisk_short}} トリガー、アクション、およびルールをインストール、アンインストール、および更新するためのヘルパー・スクリプト |
{: java}

| ファイル                                     | 説明                              |
| ---------------------------------------- | ---------------------------------------- |
| [**actions**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/actions) | サーバーレス・モバイル・バックエンドの {{site.data.keyword.openwhisk_short}} アクションのコード |
| [**followupapp**](https://github.com/IBM-Cloud/serverless-followupapp-ios/tree/master/followupapp) | モバイル・アプリケーションのコード          |
| [**deploy.sh**](https://github.com/IBM-Cloud/serverless-followupapp-ios/blob/master/deploy.sh) | {{site.data.keyword.openwhisk_short}} トリガー、アクション、およびルールをインストール、アンインストール、および更新するためのヘルパー・スクリプト |
{: swift}

## ユーザー認証、フィードバック維持、および分析を処理するサービスのプロビジョン
{: #provision_services}

このセクションでは、アプリケーションが使用するサービスをプロビジョンします。{{site.data.keyword.Bluemix_notm}} カタログからサービスをプロビジョンするか、または `ibmcloud` コマンド・ラインを使用してサービスをプロビジョンできます。

サービスをプロビジョンし、サーバーレス・バックエンドをデプロイするために、新規スペースを作成することが推奨されます。これにより、すべてのリソースをまとめて維持できます。

### {{site.data.keyword.Bluemix_notm}} カタログからのサービスのプロビジョン

1. [{{site.data.keyword.Bluemix_notm}} カタログ](https://{DomainName}/catalog/)に移動します。
2. **ライト (Lite)** プランで [{{site.data.keyword.cloudant_short_notm}}](https://{DomainName}/catalog/services/cloudantNoSQLDB) サービスを作成します。名前を **serverlessfollowup-db** に設定します。
3. **標準 (Standard)** プランで [{{site.data.keyword.toneanalyzershort}}](https://{DomainName}/catalog/services/tone_analyzer) サービスを作成します。名前を **serverlessfollowup-tone** に設定します。
4. **段階課金制 (Graduated tier)** プランで [{{site.data.keyword.appid_short}}](https://{DomainName}/catalog/services/AppID) サービスを作成します。名前を **serverlessfollowup-appid** に設定します。
5. **ライト (Lite)** プランで [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/catalog/services/imfpush) サービスを作成します。名前を **serverlessfollowup-mobilepush** に設定します。

### コマンド・ラインからのサービスのプロビジョン

サービスをプロビジョンし、サービスの資格情報を取得するため、コマンド・ラインで以下のコマンドを実行します。

   ```sh
   ibmcloud cf create-service cloudantNoSQLDB Lite serverlessfollowup-db
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-db for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service tone_analyzer standard serverlessfollowup-tone
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-tone for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service AppID "Graduated tier" serverlessfollowup-appid
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-appid for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service imfpush lite serverlessfollowup-mobilepush
   ```
   {: pre}

   ```sh
   ibmcloud cf create-service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

   ```sh
   ibmcloud cf service-key serverlessfollowup-mobilepush for-cli
   ```
   {: pre}

## {{site.data.keyword.mobilepushshort}} の構成
{: #push_notifications}

ユーザーが新しいフィードバックを送信すると、アプリケーションはこのフィードバックを分析し、通知をユーザーに送信します。ユーザーは既に他のタスクに移っているか、またはモバイル・アプリを開始していない可能性があるので、ユーザーとのコミュニケーションにプッシュ通知を使用することが推奨されます。{{site.data.keyword.mobilepushshort}} サービスにより、1 つの統合 API を使用して iOS ユーザーまたは Android ユーザーに通知を送信できます。このセクションでは、ターゲット・プラットフォームに合わせて {{site.data.keyword.mobilepushshort}} サービスを構成します。

### Firebase Cloud Messaging (FCM) の構成
{: java}

   1. [Firebase コンソール](https://console.firebase.google.com)で、新規プロジェクトを作成します。 名前を **serverlessfollowup** に設定します。
   2. プロジェクトの**設定**に移動します。
   3. **「General」**タブで 2 つのアプリケーションを追加します。
      1. パッケージ名が **com.ibm.mobilefirstplatform.clientsdk.android.push** に設定されたアプリケーション
      2. パッケージ名が **serverlessfollowup.app** に設定されているアプリケーション
   4. 2 つの定義済みアプリケーションが含まれている `google-services.json` を Firebase コンソールからダウンロードし、このファイルを checkout ディレクトリーの `android/app` フォルダーに保存します。
   5. **「Cloud Messaging」**タブの下で、送信側 ID およびサーバー・キー (API キーとも後述されます) を見つけます。
   6. 「Push Notifications」サービス・ダッシュボードで、Sender ID と API Key の値を設定します。
   {: java}

### Apple Push Notifications Service (APNs) の構成
{: swift}

1. [Apple Developer](https://developer.apple.com/) ポータルに移動し、アプリ ID を登録します。
2. 開発および配布用 APNs SSL 証明書を作成します。
3. 開発プロビジョニング・プロファイルを作成します。
4. {{site.data.keyword.Bluemix_notm}} で {{site.data.keyword.mobilepushshort}} サービス・インスタンスを構成します。詳しい手順については、[APNs 資格情報の取得および {{site.data.keyword.mobilepushshort}} サービスの構成](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials-and-configure-push-notifications-service-instance-)を参照してください。
{: swift}

## サーバーレス・バックエンドのデプロイ
{: #serverless_backend}

すべてのサービスの構成が完了したら、サーバーレス・バックエンドをデプロイできます。このセクションでは以下の {{site.data.keyword.openwhisk_short}} 成果物が作成されます。

| 成果物                      | タイプ                           | 説明                              |
| ----------------------------- | ------------------------------ | ---------------------------------------- |
| `serverlessfollowup`          | パッケージ                        | アクションをグループにまとめ、すべてのサービス資格情報を保持するためのパッケージ |
| `serverlessfollowup-cloudant` | パッケージ・バインディング        | 組み込み {{site.data.keyword.cloudant_short_notm}} パッケージにバインドされている   |
| `serverlessfollowup-push`     | パッケージ・バインディング        | {{site.data.keyword.mobilepushshort}} パッケージにバインドされている  |
| `auth-validate`               | アクション                         | アクセス・トークンと ID トークンを検証する |
| `users-add`                   | アクション                         | ユーザー情報 (ID、名前、E メール、写真、デバイス ID) を保持する |
| `users-prepare-notify`        | アクション                         | {{site.data.keyword.mobilepushshort}} で使用するためにメッセージをフォーマットする |
| `feedback-put`                | アクション                         | ユーザー・フィードバックをデータベースに保管する   |
| `feedback-analyze`            | アクション                         | {{site.data.keyword.toneanalyzershort}} を使用してフィードバックを分析する   |
| `users-add-sequence`          | Web アクションとして公開されるシーケンス | `auth-validate` および `users-add`          |
| `feedback-put-sequence`       | Web アクションとして公開されるシーケンス | `auth-validate` および `feedback-put`       |
| `feedback-analyze-sequence`   | シーケンス                       | {{site.data.keyword.cloudant_short_notm}} からの `read-document`、{{site.data.keyword.mobilepushshort}} を使用した `feedback-analyze`、`users-prepare-notify` および `sendMessage` |
| `feedback-analyze-trigger`    | トリガー                        | フィードバックがデータベースに保管されると {{site.data.keyword.openwhisk_short}} により呼び出される |
| `feedback-analyze-rule`       | ルール                           | トリガー `feedback-analyze-trigger` をシーケンス `feedback-analyze-sequence` にリンクする|

### コードのコンパイル
{: java}
1. checkout ディレクトリーのルートから、アクション・コードをコンパイルします。
{: java}

   ```sh
   ./android/gradlew -p actions clean jar
   ```
   {: pre}
   {: java}

### アクションの構成とデプロイ
{: java}

2. template.local.env を local.env にコピーします。

   ```sh
   cp template.local.env local.env
   ```
3. {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.toneanalyzershort}}、{{site.data.keyword.mobilepushshort}}、
および {{site.data.keyword.appid_short}} サービスの資格情報を {{site.data.keyword.Bluemix_notm}} ダッシュボード (または以前に実行した ibmcloud コマンドの出力) から取得し、`local.env` のプレースホルダーを対応する値に置き換えます。これらのプロパティーはパッケージに注入されるため、すべてのアクションがデータベースにアクセスできるようになります。
4. アクションを {{site.data.keyword.openwhisk_short}} にデプロイします。{{site.data.keyword.cloudant_short_notm}} データベース (users、feedback、および moods) を作成し、アプリケーションの {{site.data.keyword.openwhisk_short}} 成果物をデプロイするための資格情報が、`deploy.sh` により `local.env` からロードされます。
   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   チュートリアルが完了したら、`./deploy.sh --uninstall` を使用して {{site.data.keyword.openwhisk_short}} 成果物を削除できます。
   {: tip}

### アクションの構成とデプロイ
{: swift}

1. template.local.env を local.env にコピーします。
   ```sh
   cp template.local.env local.env
   ```
2. {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.toneanalyzershort}}、{{site.data.keyword.mobilepushshort}}、
および {{site.data.keyword.appid_short}} サービスの資格情報を {{site.data.keyword.Bluemix_notm}} ダッシュボード (または以前に実行した ibmcloud コマンドの出力) から取得し、`local.env` のプレースホルダーを対応する値に置き換えます。これらのプロパティーはパッケージに注入されるため、すべてのアクションがデータベースにアクセスできるようになります。
3. アクションを {{site.data.keyword.openwhisk_short}} にデプロイします。{{site.data.keyword.cloudant_short_notm}} データベース (users、feedback、および moods) を作成し、アプリケーションの {{site.data.keyword.openwhisk_short}} 成果物をデプロイするための資格情報が、`deploy.sh` により `local.env` からロードされます。

   ```sh
   ./deploy.sh --install
   ```
   {: pre}

   チュートリアルが完了したら、`./deploy.sh --uninstall` を使用して {{site.data.keyword.openwhisk_short}} 成果物を削除できます。
   {: tip}

## ユーザー・フィードバックを収集するネイティブ・モバイル・アプリケーションの構成と実行
{: #mobile_app}

これで、{{site.data.keyword.openwhisk_short}} アクションをモバイル・アプリに使用できます。モバイル・アプリを実行する前に、作成したサービスをターゲットとするようにモバイル・アプリの設定を構成する必要があります。

1. Android Studio で、checkout ディレクトリーの `android` フォルダーにあるプロジェクトを開きます。
2. `android/app/src/main/res/values/credentials.xml` を編集し、空白部分に資格情報の値を入力します。{{site.data.keyword.appid_short}} `tenantId`、{{site.data.keyword.mobilepushshort}} `appGuid`、`clientSecret` と、{{site.data.keyword.openwhisk_short}} がデプロイされている組織とスペースの名前が必要です。API ホストの場合、コマンド・プロンプトまたはターミナルを起動して次のコマンドを実行します。
   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
3. `android/app/src/main/java/serverlessfollowup/app/LoginActivity.java` と `LoginAndRegistrationListener.java` を開き、サービス・インスタンスが作成されている場所に応じてプッシュ通知サービス (BMSClient) の地域と AppID の地域を更新します。
4. プロジェクトをビルドします。
5. 実際のデバイスまたはエミュレーターでアプリケーションを開始します。
エミュレーターでプッシュ通知を受信するには、Google API で画像を選択し、エミュレーター内で Google アカウントを使用してログインしてください。
   {: tip}
6. バックグラウンドで {{site.data.keyword.openwhisk_short}} を監視します。
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
7. アプリケーションで**「ログイン」**を選択し、Facebook または Google アカウントを使用して認証します。ログインが完了したら、フィードバック・メッセージを入力し、**「フィードバックの送信」**ボタンをクリックします。フィードバック送信から数秒後に、デバイスにプッシュ通知が届きます。通知テキストをカスタマイズするには、{{site.data.keyword.cloudant_short_notm}} サービス・インスタンスで `moods` データベースのテンプレート文書を変更します。**「トークンの表示 (View token)」**ボタンを使用して、ログイン時に {{site.data.keyword.appid_short}} により生成されたアクセス・トークンと ID トークンを調べます。
{: java}


1. プッシュ・クライアント SDK とその他の SDK は、CocoaPods および Carthage で入手できます。このソリューションでは、CocoaPods を使用します。
2. ターミナルを開き、`cd` で `followupapp` フォルダーに移動します。以下のコマンドを実行して、必要な依存関係をインストールします。
   ```sh
   pod install
   ```
   {: pre}
3. checkout ディレクトリーの `followupapp` フォルダーにある拡張子が `.xcworkspace` のファイルを開き、コードを Xcode で開始します。
4. `BMSCredentials.plist` ファイルを編集し、空白部分に資格情報の値を入力します。{{site.data.keyword.appid_short}} `tenantId`、{{site.data.keyword.mobilepushshort}} `appGuid`、および `clientSecret` と、{{site.data.keyword.openwhisk_short}} がデプロイされている組織とスペースの名前が必要です。API ホストの場合、ターミナルを起動して次のコマンドを実行します。

   ```sh
   ibmcloud fn property get --apihost
   ```
   {: pre}
5. `AppDelegate.swift` を開き、サービス・インスタンスが作成されている場所に応じてプッシュ通知サービス (BMSClient) の地域と AppID の地域を更新します。
6. プロジェクトをビルドします。
7. 実際のデバイスまたはシミュレーターでアプリケーションを開始します。
8. ターミナルで次のコマンドを実行し、バックグラウンドで {{site.data.keyword.openwhisk_short}} を監視します。
   ```sh
   ibmcloud fn activation poll
   ```
   {: pre}
9. アプリケーションで**「ログイン」**を選択し、Facebook または Google アカウントを使用して認証します。ログインが完了したら、フィードバック・メッセージを入力し、**「フィードバックの送信」**ボタンをクリックします。フィードバック送信から数秒後に、デバイスにプッシュ通知が届きます。通知テキストをカスタマイズするには、{{site.data.keyword.cloudant_short_notm}} サービス・インスタンスで `moods` データベースのテンプレート文書を変更します。**「トークンの表示 (View token)」**ボタンを使用して、ログイン時に {{site.data.keyword.appid_short}} により生成されたアクセス・トークンと ID トークンを調べます。
{: swift}

## リソースの削除

1. `deploy.sh` を使用して {{site.data.keyword.openwhisk_short}} 成果物を削除します。

   ```sh
   ./deploy.sh --uninstall
   ```
   {: pre}

2. {{site.data.keyword.cloudant_short_notm}}、{{site.data.keyword.appid_short}}、{{site.data.keyword.mobilepushshort}}、および {{site.data.keyword.toneanalyzershort}} サービスを {{site.data.keyword.Bluemix_notm}} コンソールから削除します。

## 関連コンテンツ

* {{site.data.keyword.appid_short}} には、ID プロバイダーの初期セットアップに役立つデフォルト構成が用意されています。アプリケーションの公開前に、[構成を自分の資格情報になるように更新してください](https://{DomainName}/docs/services/appid?topic=appid-social#social)。また、[ログイン・ウィジェットをカスタマイズ](https://{DomainName}/docs/services/appid?topic=appid-login-widget#login-widget)できます。


* Swift ソース・ファイル (`actions` フォルダー内の .swift ファイル) を使用して OpenWhisk Swift アクションを作成した場合、アクションを実行する前に、バイナリーにコンパイルしておく必要があります。この作業が完了し、アクションを保持しているコンテナーがパージされるまでが、そのアクションに対する後続の呼び出しが大幅に高速になります。この遅延は、コールド・スタートの遅延と呼ばれます。
  コールド・スタートの遅延を避けるために、Swift ファイルをバイナリーにコンパイルしてから、zip ファイルとして OpenWhisk にアップロードすることができます。 OpenWhisk スキャフォールドが必要になるため、バイナリーを作成する最も簡単な方法は、それを実行するのと同じ環境内でビルドすることです。詳しい手順については、[Swift 実行可能ファイルとしてのアクションのパッケージ化](https://{DomainName}/docs/openwhisk?topic=cloud-functions-creating-swift-actions#creating-swift-actions)を参照してください。
{: swift}
