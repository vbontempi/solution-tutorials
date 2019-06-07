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

# プッシュ通知を使用するハイブリッド・モバイル・アプリケーション
{: #hybrid-mobile-push-analytics}

{{site.data.keyword.Bluemix_notm}} での {{site.data.keyword.mobilepushshort}} のような付加価値の高いモバイルサービスを使用した、Hybrid Cordova アプリケーションを素早く簡単に作成する方法について説明します。

Apache Cordova は、オープン・ソースのモバイル開発フレームワークです。 これにより、標準の Web テクノロジー (クロスプラットフォーム開発のための HTML5、CSS3、および JavaScript) を使用できます。 アプリケーションは、各プラットフォームをターゲットとするラッパー内で実行され、標準に準拠した API バインディングに依存して、各デバイスのセンサー、データ、ネットワーク状況などの機能にアクセスします。

このチュートリアルでは、モバイル・サービスの追加、クライアント SDK のセットアップ、およびスキャフォールドされたコードのダウンロードを行ってから、アプリケーションをさらに拡張して、Cordova モバイル・スターター・アプリケーションを作成する方法について説明します。

## 達成目標

* {{site.data.keyword.mobilepushshort}} サービスを使用してモバイル・プロジェクトを作成します。
* APNs および FCM の各資格情報を取得する方法について説明します。
* コードをダウンロードし、必要なセットアップを実行します。
* {{site.data.keyword.mobilepushshort}} を構成、送信およびモニターします。

 ![](images/solution15/Architecture.png)

## 製品

このチュートリアルでは、以下の製品を使用します。
* [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 始める前に
{: #prereqs}

- Cordova [CLI](https://cordova.apache.org/docs/en/latest/guide/cli/) (Cordova コマンドを実行する)。
- Cordova-iOS [前提条件](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html)および Cordova-Android [前提条件](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html)
- Google アカウント (Firebase コンソールにログインして送信者 ID とサーバー API キーを取得する)。
- [Apple Developer](https://developer.apple.com/) アカウント ({{site.data.keyword.Bluemix_notm}} (プロバイダー) での {{site.data.keyword.mobilepushshort}} サービス・インスタンスからのリモート通知を iOS デバイスおよび iOS アプリケーションに送信する)。
- Xcode および Android Studio (コードをインポートし、さらに拡張する)。

## スターター・キットからの Cordova モバイル・プロジェクトの作成
{: #get_code}
{{site.data.keyword.Bluemix_notm}} モバイル・ダッシュボードを使用すると、スターター・キットからプロジェクトを作成することにより、モバイル・アプリ開発を迅速に行うことができます。
1. [モバイル・ダッシュボード](https://{DomainName}/developer/mobile/dashboard)にナビゲートします。
2. **「スターター・キット」**をクリックし、**「アプリの作成」**をクリックします。
    ![](images/solution15/mobile_dashboard.png)
3. プロジェクト名を入力します。この名前は、アプリ名としても使用されます。
4. プラットフォームとして**「Cordova」**を選択し、**「作成」**をクリックします。

    ![](images/solution15/create_cordova_project.png)
5. **「リソースの追加」**>「モバイル」>**「プッシュ通知」**をクリックし、サービスをプロビジョンする場所、リソース・グループおよび**「ライト」**料金プランを選択します。
6. **「作成」**をクリックして、{{site.data.keyword.mobilepushshort}}サービスをプロビジョンします。**「アプリ」**タブに新しいアプリが作成されます。

    **注:** {{site.data.keyword.mobilepushshort}} サービスは、空のスターターに追加する必要があります。

次のステップでは、スキャフォールドされたコードをダウンロードし、必要なセットアップを完了します。

## コードのダウンロードと必要なセットアップの実行
{: #download_code}

コードをまだダウンロードしていない場合は、{{site.data.keyword.Bluemix_notm}} モバイル・ダッシュボードを使用し、「プロジェクト」>**「モバイル・プロジェクト (Your Mobile Project)」**の下の**「コードのダウンロード (Download Code)」**ボタンをクリックして、コードを取得します。

1. 任意の IDE で、`/platforms/android/project.properties` にナビゲートし、最後の 2 行 (library.1 および library.2) を次の行で置き換えます。

  ```
  cordova.system.library.1=com.android.support:appcompat-v7:26.1.0
  cordova.system.library.2=com.google.firebase:firebase-messaging:10.2.6
  cordova.system.library.3=com.google.android.gms:play-services-location:10.2.6
  ```
  上記の変更は Android に固有です。
  {: tip}
2. Android エミュレーター上でアプリを起動するには、次のコマンドを端末またはコマンド・プロンプトで実行します。
```
 $ cordova build android
 $ cordova run android
```
 エラーが表示される場合は、エミュレーターを起動し、上記のコマンドを実行してみます。
 {: tip}
3. アプリを iOS シミュレーターでプレビューするには、次のコマンドを端末で実行します。

    ```
    $ cordova build ios
    ```
    ```
    $ open platforms/ios/{YOUR_PROJECT_NAME}.xcworkspace/
    ```
    プロジェクト名を `config.xml` ファイルで検索するには、`cordova info` コマンドを実行します。
    {: tip}

## FCM および APNs の資格情報の取得
{: #obtain_fcm_apns_credentials}

 ### Firebase Cloud Messaging (FCM) の構成

  1. [Firebase コンソール](https://console.firebase.google.com)で、新規プロジェクトを作成します。 名前を **hybridmobileapp** に設定します。
  2. プロジェクトの**設定**に移動します。
  3. **「General」**タブで 2 つのアプリケーションを追加します。
       1. パッケージ名が **com.ibm.mobilefirstplatform.clientsdk.android.push** に設定されたアプリケーション
       2. およびパッケージ名が **io.cordova.hellocordovastarter** に設定されたアプリケーション
  4. **「Cloud Messaging」**タブの下で、送信側 ID およびサーバー・キー (API キーとも後述されます) を見つけます。
  5. {{site.data.keyword.mobilepushshort}} サービス・ダッシュボードで、送信側 ID および API キーの値を設定します。


詳しいステップについては、[FCM 資格情報の取得](https://{DomainName}/docs/tutorials?topic=solution-tutorials-android-mobile-push-analytics#obtain-fcm-credentials)を参照してください。
{: tip}

### Apple {{site.data.keyword.mobilepushshort}} Service (APNs) の構成

  1. [Apple Developer](https://developer.apple.com/) ポータルに移動し、アプリ ID を登録します。
  2. 開発および配布用 APNs SSL 証明書を作成します。
  3. 開発プロビジョニング・プロファイルを作成します。
  4. {{site.data.keyword.Bluemix_notm}} で、{{site.data.keyword.mobilepushshort}} サービス・インスタンスを構成します。

詳しいステップについては、[APNs 資格情報の取得および {{site.data.keyword.mobilepushshort}} サービスの構成](https://{DomainName}/docs/tutorials?topic=solution-tutorials-ios-mobile-push-analytics#obtain-apns-credentials)を参照してください。
{: tip}

## {{site.data.keyword.mobilepushshort}}の構成、送信およびモニター
{: #configure_push}

1. index.js の `onDeviceReady` 関数で、`{pushAppGuid}` 値および

   `{pushClientSecret}` 値をプッシュ・サービスの**資格情報** (*appGuid* および *clientSecret*) で置き換えます。

2. モバイル・ダッシュボードから、「プロジェクト」>「Cordova プロジェクト (Cordova Project)」に移動し、{{site.data.keyword.mobilepushshort}} サービスをクリックして、以下のステップに従います。

### APNs - サービス・インスタンスの構成

{{site.data.keyword.mobilepushshort}} サービスを使用して通知を送信するには、前述のステップで作成した .p12 証明書をアップロードします。 この証明書には、アプリケーションのビルドと公開に必要な、秘密鍵と SSL 証明書が含まれています。

**注:** `.cer` ファイルがキー・チェーン・アクセスに配置された後に、それをコンピューターにエクスポートして `.p12` 証明書を作成します。

1. 「サービス」セクションの下で `{{site.data.keyword.mobilepushshort}}` をクリックするか、{{site.data.keyword.mobilepushshort}} サービスの横にある垂直に並んだ 3 つのドットをクリックして、`「Open dashboard」`を選択します。
2. {{site.data.keyword.mobilepushshort}}ダッシュボードが開き、`「Manage」>「Send Notifications」`の下に`「Configure」`オプションが表示されます。

`Push Notification services` コンソールで APNs をセットアップするには、以下のステップを実行します。

1. Push Notification サービス・ダッシュボードから、`「Configure」`を選択します。
2. `「Mobile option」`を選択して、「APN プッシュ資格情報 (APNs Push Credentials)」フォームの情報を更新します。
3. 必要に応じて`「Sandbox (development)」`または`「Production (distribution)」`を選択してから、作成した `p.12` 証明書をアップロードします。
4. 「パスワード」フィールドに、.p12 証明書ファイルに関連付けられるパスワードを入力してから、「保存」をクリックします。

### FCM - サービス・インスタンスの構成

1. **「モバイル」**を選択した後、「GCM/FCM プッシュ資格情報 (GCM/FCM Push Credentials)」タブの、Firebase コンソールで最初に作成した送信者 ID/プロジェクト番号および API キー (サーバー・キー) を更新します。
2. **「保存」**をクリックします。これで、{{site.data.keyword.mobilepushshort}} サービスが構成されました。

### {{site.data.keyword.mobilepushshort}} の送信

1. **「通知の送信」**を選択し、送信オプションを選択してメッセージを構成します。サポートされるオプションは、「デバイス (タグ別)」、「デバイス ID」、「ユーザー ID」、「Android デバイス」、「IOS デバイス」、「Web 通知」、および「すべてのデバイス」です。
   **注:** **「すべてのデバイス」**オプションを選択すると、{{site.data.keyword.mobilepushshort}}をサブスクライブするすべてのデバイスが通知を受信します。

2. **「メッセージ」**フィールドで、メッセージを作成します。 必要に応じてオプションの設定を構成してください。
3. **「送信」**をクリックし、物理デバイスが通知を受信していることを確認します。

### 送信済み通知のモニター

送信済み通知をモニターするには、**「モニタリング」**セクションにナビゲートします。
これで、IBM {{site.data.keyword.mobilepushshort}}サービスの機能が拡張され、ユーザー・データからグラフを生成することによって、プッシュのパフォーマンスをモニターできるようになりました。ユーティリティーを使用して、送信されたすべての{{site.data.keyword.mobilepushshort}}をリストしたり、登録されたすべてのデバイスをリストしたり、日次、週次、または月次ベースで情報を分析したりすることができます。
      ![](images/solution6/monitoring_messages.png)

## 関連コンテンツ
{: #related_content}

- [タグ・ベースの通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)でのセキュリティー

