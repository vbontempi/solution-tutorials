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

# プッシュ通知を使用する Android ネイティブ・モバイル・アプリケーション
{: #android-mobile-push-analytics}

{{site.data.keyword.Bluemix_notm}} での {{site.data.keyword.mobilepushshort}} などの、高価値のモバイル・サービスを使用するネイティブ Android アプリケーションを迅速かつ簡単に作成できることを説明します。

このチュートリアルでは、モバイル・スターター・アプリケーションを作成し、モバイル・サービスを追加し、クライアント SDK を設定し、Android Studio にコードをインポートしてアプリケーションをさらに拡張する方法について説明します。

## 達成目標
{: #objectives}

* {{site.data.keyword.mobilepushshort}} サービスを使用するモバイル・アプリを作成します。
* FCM 資格情報を取得します。
* コードをダウンロードし、必要なセットアップを実行します。
* {{site.data.keyword.mobilepushshort}} を構成、送信およびモニターします。

![](images/solution9/Architecture.png)

## 製品
{: #products}

このチュートリアルでは、以下の製品を使用します。
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 始める前に
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html) (コードをインポートし、拡張する)。
- Google アカウント (Firebase コンソールにログインして送信者 ID とサーバー API キーを取得する)。

## スターター・キットからのモバイル・アプリの作成
{: #get_code}
{{site.data.keyword.Bluemix_notm}} モバイル・ダッシュボードでスターター・キットからアプリを作成することによって、モバイル・アプリの開発を迅速化できます。
1. [モバイル・ダッシュボード](https://{DomainName}/developer/mobile/dashboard)にナビゲートします。
2. **「スターター・キット」**をクリックし、**「アプリの作成」**をクリックします。
    ![](images/solution9/mobile_dashboard.png)
3. アプリ名を入力します。これは Android プロジェクト名にもできます。
4. プラットフォームとして**「Android」**を選択し、**「作成」**をクリックします。

    ![](images/solution9/create_mobile_project.png)
5. **「リソースの追加」**>「モバイル」>**「プッシュ通知」**をクリックし、サービスをプロビジョンする場所、リソース・グループおよび**「ライト」**料金プランを選択します。
6. **「作成」**をクリックして、{{site.data.keyword.mobilepushshort}}サービスをプロビジョンします。**「アプリ」**タブに新しいアプリが作成されます。

    **注:** {{site.data.keyword.mobilepushshort}}サービスは、空のスターターと一緒に追加されている必要があります。
    次のステップで、Firebase Cloud Messaging (FCM) 資格情報を取得します。

次のステップで、スキャフォールド・コードをダウンロードし、プッシュ Android SDK をセットアップします。

## コードのダウンロードと必要なセットアップの実行
{: #download_code}

コードをまだダウンロードしていない場合は、{{site.data.keyword.Bluemix_notm}} モバイル・ダッシュボードを使用して、「アプリ」>**「モバイル・アプリ (Your Mobile App)」**の下の**「コードのダウンロード」**ボタンをクリックし、コードを取得します。
ダウンロードしたコードには、**{{site.data.keyword.mobilepushshort}}**クライアント SDK が含まれています。このクライアント SDK は Gradle と Maven で使用できます。このチュートリアルでは、**Gradle** を使用します。

1. Android Studio を起動し、**既存の Android Studio プロジェクトを開き**、ダウンロードしたコードをポイントします。
2. **Gradle** ビルドが自動的にトリガーされ、すべての依存関係がダウンロードされます。
3. モジュール・レベルの `build.gradle (Module: app)` ファイルの末尾、`dependencies{.....}` の後に、**Google Play サービス**の依存関係を追加します。
   ```
   apply plugin: 'com.google.gms.google-services'
   ```
4. 作成してダウンロードした `google-services.json` ファイルを Android アプリケーション・モジュールのルート・ディレクトリーにコピーします。`google-service.json` ファイルには、追加されたパッケージ名が含まれていることに注意してください。
5. 必要なすべての許可は、`AndroidManifest.xml` ファイルおよび依存関係内にあります。プッシュおよび分析は、**build.gradle (Module: app)** に含まれています。
6. `RECEIVE` と `REGISTRATION` の各イベント通知に対する **Firebase Cloud Messaging (FCM)** インテント・サービスおよびインテント・フィルターは、`AndroidManifest.xml` に含まれています。

## FCM 資格情報の取得
{: #obtain_fcm_credentials}

Firebase Cloud Messaging (FCM) は、{{site.data.keyword.mobilepushshort}}を Android デバイス, Google Chrome ブラウザー、および Chrome Apps & Extensions に配信するために使用されるゲートウェイとなります。コンソールで{{site.data.keyword.mobilepushshort}}サービスをセットアップするには、FCM 資格情報 (送信者 ID と API キー) を取得する必要があります。

API キーはセキュアに格納され、{{site.data.keyword.mobilepushshort}} サービスによって FCM サーバーと接続するために使用されます。送信者 ID (プロジェクト番号) はクライアント・サイドで Google Chrome と Mozilla Firefox のために Android SDK および JS SDK によって使用されます。FCM をセットアップし、資格情報を取得するには、以下のステップを実行します。

1. [Firebase コンソール](https://console.firebase.google.com/?pli=1)にアクセスします。Google ユーザー・アカウントが必要です。
2. **「Add project」**を選択します。
3. **「Create a project」**ウィンドウで、プロジェクト名を指定し、国または地域を選択して**「Create project」**をクリックします。
4. 左のナビゲーション・ペインで、**「Settings」**(**「Overview」**の隣の「Settings」アイコンをクリック) >**「Project settings」**の順に選択します。
5. 「Cloud Messaging」タブを選択し、プロジェクト資格情報、つまりサーバー API キーと送信側 ID を取得します。
    **注:** FCM にリストされるサーバー・キーは、サーバー API キーと同じです。
    ![](images/solution9/fcm_console.png)

また、`google-services.json` ファイルを生成することも必要です。以下のステップを実行します。

1. Firebase コンソールで、作成したプロジェクトの**「Project Settings」**アイコン>**「General」**タブの順にクリックし、**「Add Firebase to your Android App」**を選択します。

    ![](images/solution9/firebase_project_settings.png)
2. **「Add Firebase to your Android」**アプリ・モーダル・ウィンドウで、{{site.data.keyword.mobilepushshort}} Android SDK に登録する「Package Name」として **com.ibm.mobilefirstplatform.clientsdk.android.push** を追加します。アプリ・ニックネームと SHA-1 フィールドはオプションです。**「REGISTER APP」**>**「Continue」**>**「Finish」**の順にクリックします。

    ![](images/solution9/add_firebase_to_your_app.png)

3. **「ADD APP」**>**「Add Firebase to your app」**の順にクリックします。パッケージ名 **com.ibm.mysampleapp** を入力することによって、アプリケーションのパッケージ名を含めた後、Firebase の Android アプリ・ウィンドウへの追加に進みます。アプリ・ニックネームと SHA-1 フィールドはオプションです。**「REGISTER APP」**>「Continue」>「Finish」の順にクリックします。
     **注:** アプリケーションのパッケージ名は、コードをダウンロードした後、`AndroidManifest.xml` ファイルで参照できます。
4. **「Your apps」**から、最新の構成ファイル `google-services.json` をダウンロードします。

    ![](images/solution9/google_services.png)

    **注**: FCM は Google Cloud Messaging (GCM) の新しいバージョンです。新しいアプリに対して、FCM 資格情報を使用していることを確認してください。既存のアプリは引き続き GCM の構成で機能します。

*手順と Firebase コンソールの UI は変更されることがあります。必要に応じて、Google の資料の Firebase の部分を参照してください。*

## {{site.data.keyword.mobilepushshort}}の構成、送信およびモニター

{: #configure_push}

1. {{site.data.keyword.mobilepushshort}} SDK はアプリに既にインポートされ、プッシュ初期化コードは `MainActivity.java` ファイルにあります。

    **注:** サービス資格情報は `/res/values/credentials.xml` ファイルの一部となります。
2. 通知への登録は `MainActivity.java` で発生します。(オプション) 固有の USER_ID を指定します。
3. 物理デバイスかエミュレーターでアプリを実行し、通知を受信します。
4. {{site.data.keyword.Bluemix_notm}} モバイル・ダッシュボードの**「モバイル・サービス (Mobile Services)」**>**「既存のサービス (Existing services)」**で{{site.data.keyword.mobilepushshort}} サービスを開き、基本的な{{site.data.keyword.mobilepushshort}}を送信するために、以下のステップを実行します。
   - **「管理」**>**「構成」**の順にクリックします。
   - **「モバイル」**を選択した後、「GCM/FCM プッシュ資格情報 (GCM/FCM Push Credentials)」タブの、Firebase コンソールで最初に作成した送信者 ID/プロジェクト番号および API キー (サーバー・キー) を更新します。

     ![](images/solution9/configure_push_notifications.png)
   - **「保存」**をクリックします。これで、{{site.data.keyword.mobilepushshort}} サービスが構成されました。
   - **「通知の送信」**を選択し、送信オプションを選択してメッセージを構成します。サポートされるオプションは、「デバイス (タグ別)」、「デバイス ID」、「ユーザー ID」、「android デバイス」、「IOS デバイス」、「Web 通知」、および「すべてのデバイス」です。
     **注:** **「すべてのデバイス」**オプションを選択すると、{{site.data.keyword.mobilepushshort}}をサブスクライブするすべてのデバイスが通知を受信します。
   - **「メッセージ」**フィールドで、メッセージを作成します。 必要に応じてオプションの設定を構成してください。
   - **「送信」**をクリックし、物理デバイスが通知を受信していることを確認します。

     ![](images/solution9/android_send_notifications.png)
5. Android デバイスに通知が表示されます。

      ![](images/solution9/android_notifications1.png)   ![](images/solution9/android_notifications2.png)
6. 送信された通知をモニターするには、{{site.data.keyword.mobilepushshort}}サービスの**「モニタリング」**にナビゲートします。
     これで、IBM {{site.data.keyword.mobilepushshort}}サービスの機能が拡張され、ユーザー・データからグラフを生成することによって、プッシュのパフォーマンスをモニターできるようになりました。ユーティリティーを使用して、送信されたすべての{{site.data.keyword.mobilepushshort}}をリストしたり、登録されたすべてのデバイスをリストしたり、日次、週次、または月次ベースで情報を分析したりすることができます。
      ![](images/solution6/monitoring_messages.png)

## 関連コンテンツ
{: #related_content}
- [{{site.data.keyword.mobilepushshort}}の設定のカスタマイズ](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_4#push_step_4_Android)
- [タグ・ベースの通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)でのセキュリティー

