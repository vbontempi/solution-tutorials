---
copyright:
  years: 2017, 2019
lastupdated: "2019-03-08"
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

# プッシュ通知を使用する iOS モバイル・アプリケーション
{: #ios-mobile-push-analytics}

付加価値の高いモバイル・サービスである {{site.data.keyword.Bluemix_short}} での {{site.data.keyword.mobilepushshort}} を使用した、iOS Swift アプリケーションを素早く簡単に作成する方法について説明します。

このチュートリアルでは、モバイル・サービスの追加、クライアント SDK のセットアップ、および Xcode へのコードのインポートを行ってから、アプリケーションをさらに拡張して、モバイル・スターター・アプリケーションを作成する方法について説明します。
{:shortdesc: .shortdesc}

## 達成目標
{:#objectives}

- 基本 Swift スターター・キットから、{{site.data.keyword.mobilepushshort}} および {{site.data.keyword.mobileanalytics_short}} の各サービスを使用したモバイル・アプリを作成します。
- APNs 資格情報を取得して、{{site.data.keyword.mobilepushshort}} サービス・インスタンスを構成します。
- コードをダウンロードしてクライアント SDK をセットアップします。
- {{site.data.keyword.mobilepushshort}} を送信およびモニターします。

  ![](images/solution6/Architecture.png)

## 製品
{:#products}

このチュートリアルでは、以下の製品を使用します。
   * [{{site.data.keyword.pushfull}}](https://{DomainName}/catalog/services/push-notifications)

## 始める前に
{: #prereqs}

1. [Apple Developer](https://developer.apple.com/) アカウント ({{site.data.keyword.Bluemix_short}} (プロバイダー) での {{site.data.keyword.mobilepushshort}} サービス・インスタンスからのリモート通知を iOS デバイスおよび iOS アプリケーションに送信する)。
2. Xcode (コードをインポートし、拡張する)。

## 基本 Swift スターター・キットからのモバイル・アプリの作成
{: #get_code}

1. [モバイル・ダッシュボード](https://{DomainName}/developer/mobile/dashboard)にナビゲートして、事前定義された `Starter Kits` から `App` を作成します。
2. **「スターター・キット」**をクリックし、スクロールダウンして**「基本」**スターター・キットを選択します。
    ![](images/solution6/mobile_dashboard.png)
3. アプリ名を入力します。この名前は、Xcode プロジェクトとアプリの名前としても使用されます。
4. プラットフォームとして`「iOS Swift」`を選択し、**「作成」**をクリックします。
    ![](images/solution6/create_mobile_project.png)
5. **「リソースの追加」**>「モバイル」>**「プッシュ通知」**をクリックし、サービスをプロビジョンする場所、リソース・グループおよび**「ライト」**料金プランを選択します。
6. **「作成」**をクリックして、{{site.data.keyword.mobilepushshort}}サービスをプロビジョンします。**「アプリ」**タブに新しいアプリが作成されます。

​      **注:** {{site.data.keyword.mobilepushshort}}サービスは、既に空のスターターと一緒に追加されている必要があります。

## コードのダウンロードおよびクライアント SDK のセットアップ
{: #download_code}

コードをまだダウンロードしていない場合は、「アプリ」>`「Your Mobile App」`の下の`「Download Code」`をクリックします。
コードは、クライアント SDK が組み込まれた **{{site.data.keyword.mobilepushshort}}** とともにダウンロードされます。クライアント SDK は、CocoaPods および Carthage で入手できます。このソリューションには、CocoaPods を使用します。

1. CocoaPods をマシンにインストールするには、`「Terminal」`を開いて次のコマンドを実行します。
   ```
   sudo gem install cocoapods
   ```
   {: pre:}
2. ダウンロードされたコードを unzip し、端末を使用して、unzip したフォルダーにナビゲートします。

   ```
   cd <プロジェクト名>
   ```
   {: pre:}
3. このフォルダーには、必要な依存関係を持つ `podfile` が既に含まれています。 次のコマンドを実行して依存関係 (クライアント SDK) をインストールすると、必要な依存関係がインストールされます。

  ```
  pod install
  ```
  {: pre:}

## APNs 資格情報を取得して、{{site.data.keyword.mobilepushshort}} サービス・インスタンスを構成します。
{: #obtain_apns_credentials}

   iOS デバイスおよび iOS アプリケーションの場合、Apple Push Notification Service (APNs) を使用すると、アプリケーション開発者は、{{site.data.keyword.Bluemix_short}} (プロバイダー) 上の {{site.data.keyword.mobilepushshort}} サービス・インスタンスから iOS デバイスおよび iOS アプリケーションにリモート通知を送信できます。 メッセージは、デバイス上のターゲット・アプリケーションに送信されます。

   APNs 資格情報を取得して構成する必要があります。 APNs の証明書は、{{site.data.keyword.mobilepushshort}} サービスによって安全に管理され、プロバイダーとしての APNs サーバーへの接続に使用されます。

### アプリ ID の登録

   アプリ ID (バンドル ID) は、特定のアプリケーションを識別する固有の ID です。 各アプリケーションにアプリ ID が必要です。 {{site.data.keyword.mobilepushshort}} サービスのようなサービスが、アプリ ID に対して構成されます。
   [Apple Developer](https://developer.apple.com/) アカウントを持っていることを確認してください。 これは、必須の前提条件です。

   1. [Apple Developer](https://developer.apple.com/) ポータルに移動し、`「Member Center」`をクリックして、`「Certificates, IDs & Profiles」`を選択します。
   2. `「Identifiers」`>「App IDs」セクションに移動します。
   3. `「Registering App IDs」`ページで、「App ID Description」の「Name」フィールドにアプリ名を入力します。 例えば、ACME {{site.data.keyword.mobilepushshort}} です。 「App ID Prefix」にストリングを入力します。
   4. 「App ID Suffix」には`「Explicit App ID」`を選択し、「Bundle ID」に値を入力します。 ドメイン名を逆にしたスタイルのストリングを指定することをお勧めします。 例えば、com.ACME.push です。
   5. `「{{site.data.keyword.mobilepushshort}}」`チェック・ボックスを選択して、`「Continue」`をクリックします。
   6. 設定を確認してから、`「Register」`>`「Done」`をクリックします。
     これで、アプリ ID が登録されました。

     ![](images/solution6/push_ios_register_appid.png)

### 開発および配布用 APNs SSL 証明書の作成
   APNs 証明書を取得する前に、最初に証明書署名要求 (CSR) を生成し、それを Apple の認証局 (CA) にサブミットしておく必要があります。 CSR には、ユーザーの会社を識別する情報および Apple {{site.data.keyword.mobilepushshort}} に署名するために使用する公開鍵と秘密鍵が含まれます。 次に、iOS Developer Portal で SSL 証明書を生成します。 証明書は、公開鍵および秘密鍵と共に「Keychain Access」に保管されます。
   APNs は、以下の 2 つのモードで使用できます。

- サンドボックス・モード。開発およびテスト用です。
- 実動モード。App Store (またはその他のエンタープライズ配布メカニズム) を介してアプリケーションを配布するときのモードです。

   開発環境と配布環境では別々の証明書を取得する必要があります。 証明書は、リモート通知の受信者であるアプリのアプリ ID に関連付けられます。 実動の場合、2 つまで証明書を作成できます。 {{site.data.keyword.Bluemix_short}} では、その証明書を使用して、APNs との SSL 接続を確立します。

   1. Apple Developer Web サイトに移動し、**「Member Center」**をクリックして、**「Certificates, IDs & Profiles」**を選択します。
   2. **「Identifiers」**領域で、**「App IDs」**をクリックします。
   3. アプリ ID のリストから自分のアプリ ID を選択してから、`「Edit」`を選択します。
   4. **「{{site.data.keyword.mobilepushshort}}」**チェック・ボックスを選択してから、**「Development SSL certificate」**ペインで、**「Create Certificate」**をクリックします。

     ![プッシュ通知の SSL 証明書](images/solution6/certificate_createssl.png)

   5. **「About Creating a Certificate Signing Request (CSR)」画面**が表示されたら、示された指示に従って、証明書署名要求 (CSR) ファイルを作成します。 開発用 (サンドボックス) と配布用 (実動) のいずれの証明書であるかを識別するのに役立つ、わかりやすい名前にしてください。例えば、sandbox-apns-certificate や production-apns-certificate です。
   6. **「Continue」**をクリックして、「Upload CSR file」画面で、**「Choose File」**をクリックし、作成した **CertificateSigningRequest.certSigningRequest** ファイルを選択します。 **「続行 (Continue)」**をクリックします。
   7. 「Download, Install and Backup」ペインで、「Download」をクリックします。 **aps_development.cer** ファイルがダウンロードされます。
        ![証明書のダウンロード](images/solution6/push_certificate_download.png)

        ![証明書の生成](images/solution6/generate_certificate.png)
   8. Mac で、**「Keychain Access」**、**「File」**、**「Import」**を開き、ダウンロードした .cer ファイルを選択してインストールします。
   9. 新しい証明書および秘密鍵を右クリックしてから、**「Export」**を選択して、**「File Format」**を個人情報交換フォーマット (`.p12` フォーマット) に変更します。
     ![証明書および鍵のエクスポート](images/solution6/keychain_export_key.png)
   10. **「Save As」**フィールドで、証明書に意味のある名前を指定します。 例えば、`sandbox_apns.p12` や **production_apns.p12** と指定してから、「Save」をクリックします。
     ![証明書および鍵のエクスポート](images/solution6/certificate_p12v2.png)
   11. **「Enter a password」**フィールドで、エクスポートする項目を保護するためのパスワードを入力し、「OK」をクリックします。 このパスワードは、{{site.data.keyword.mobilepushshort}} サービス・コンソールで APNs 設定を構成する際に使用できます。
       ![証明書および鍵のエクスポート](images/solution6/export_p12.png)
   12. **「Key Access.app」**に、**「Keychain」**画面から鍵をエクスポートするように求めるプロンプトが表示されます。 ご使用の Mac の管理パスワードを入力して、システムによるそれらの項目のエクスポートを許可し、`「Always Allow」`オプションを選択します。 `.p12` 証明書がデスクトップ上に生成されます。

      実動 SSL の場合は、**「Production SSL certificate」**ペインで**「Create Certificate」**をクリックし、上記のステップ 5 から 12 を繰り返します。
      {:tip}

### 開発プロビジョニング・プロファイルの作成
   プロビジョニング・プロファイルは、アプリ ID を使用して機能し、アプリをインストールして実行できるデバイス、およびアプリからアクセスできるサービスを判別します。 アプリ ID ごとに、2 つのプロビジョニング・プロファイル、つまり、開発用に 1 つ、配布用にもう 1 つを作成します。Xcode は、開発プロビジョニング・プロファイルを使用して、アプリケーションのビルドを許可されている開発者と、アプリケーションのテストが許可されているデバイスを判別します。

   アプリ ID を登録済みであること、それを {{site.data.keyword.mobilepushshort}} サービス用に使用可能化していること、および開発および実動の APNs SSL 証明書を使用するように構成済みであることを確認します。

   以下のように、開発プロビジョニング・プロファイルを作成します。

   1. [Apple Developer](https://developer.apple.com/) ポータルに移動し、`「Member Center」`をクリックして、`「Certificates, IDs & Profiles」`を選択します。
   2. [Mac Developer Library](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html#//apple_ref/doc/uid/TP40012582-CH30-SW62site) に移動し、`「Creating Development Provisioning Profiles」`セクションまでスクロールして、指示に従って開発プロファイルを作成します。
     **注:** 開発プロビジョン・プロファイルを構成する際に、以下のオプションを選択します。

     - **iOS App Development**
     - **For iOS and watchOS apps**

### ストア配布プロビジョニング・プロファイルの作成
   ストア・プロビジョニング・プロファイルを使用して、ご使用のアプリを配布用にアプリ・ストアにサブミットします。

   1. [Apple Developer](https://developer.apple.com/) に移動し、`「Member Center」`をクリックして、`「Certificates, IDs & Profiles」`を選択します。
   2. ダウンロードしたプロビジョニング・プロファイルをダブルクリックして、Xcode にインストールします。
     資格情報を取得したら、次のステップでは、[サービス・インスタンスを構成](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-push_step_2#push_step_2)します。

### サービス・インスタンスの構成

   {{site.data.keyword.mobilepushshort}} サービスを使用して通知を送信するには、前述のステップで作成した .p12 証明書をアップロードします。 この証明書には、アプリケーションのビルドと公開に必要な、秘密鍵と SSL 証明書が含まれています。

   **注:** `.cer` ファイルがキー・チェーン・アクセスに配置された後に、それをコンピューターにエクスポートして、`.p12` 証明書を作成します。

1. 「サービス」セクションの下で「{{site.data.keyword.mobilepushshort}}」をクリックするか、{{site.data.keyword.mobilepushshort}} サービスの横にある垂直に並んだ 3 つのドットをクリックして、`「Open dashboard」`を選択します。
2. {{site.data.keyword.mobilepushshort}}ダッシュボードが開き、`「Manage」`の下に`「Configure」`オプションが表示されます。

`Push Notification services` コンソールで APNs をセットアップするには、以下のステップを実行します。

1. `「Mobile option」`を選択して、「APN プッシュ資格情報 (APNs Push Credentials)」フォームの情報を更新します。
2. 必要に応じて`「Sandbox/Development APNs Server」`または`「Production APNs Server」`を選択してから、作成した `.p12` 証明書をアップロードします。
3. 「パスワード」フィールドに、.p12 証明書ファイルに関連付けられるパスワードを入力してから、「保存」をクリックします。

![](images/solution6/Mobile_push_configure.png)

## {{site.data.keyword.mobilepushshort}} の構成、送信、およびモニター
{: #configure_push}

1. プッシュ初期化コード (`func application` の下) および通知登録コードは、`AppDelegate.swift` にあります。 固有の USER_ID を指定します (オプション)。
2. 通知は iPhone シミュレーターに送信できないため、物理デバイス上でアプリを実行します。
3. {{site.data.keyword.Bluemix_short}} モバイル・ダッシュボードの`「Mobile Services」`>**「既存のサービス (Existing services)」**で {{site.data.keyword.mobilepushshort}} サービスを開き、基本的な {{site.data.keyword.mobilepushshort}} を送信するために、以下のステップを実行します。
  * `「メッセージ」`を選択して、送信先オプションを選択することでメッセージを構成します。 サポートされるオプションは、「デバイス (タグ別)」、「デバイス ID」、「ユーザー ID」、「Android デバイス」、「iOS デバイス」、「Web 通知」、「すべてのデバイス」、および「その他のブラウザー (other browsers)」です。

       **注:** 「すべてのデバイス」オプションを選択すると、{{site.data.keyword.mobilepushshort}}をサブスクライブするすべてのデバイスが通知を受信します。
  * `「メッセージ」`フィールドで、メッセージを作成します。 必要に応じてオプションの設定を構成してください。
  * `「Send」`をクリックし、物理デバイスが通知を受信していることを確認します。
    ![](images/solution6/send_notifications.png)

4. iPhone に通知が表示されます。

   ![](images/solution6/iphone_notification.png)

5. 送信された通知をモニターするには、{{site.data.keyword.mobilepushshort}}サービスの`「Monitoring」`にナビゲートします。

これで、IBM {{site.data.keyword.mobilepushshort}}サービスの機能が拡張され、ユーザー・データからグラフを生成することによって、プッシュのパフォーマンスをモニターできるようになりました。ユーティリティーを使用して、送信されたすべての{{site.data.keyword.mobilepushshort}}をリストしたり、登録されたすべてのデバイスをリストしたり、日次、週次、または月次ベースで情報を分析したりすることができます。
      ![](images/solution6/monitoring_messages.png)

## 関連コンテンツ
{: #related_content}

- [タグ・ベースの通知](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-tag_based_notifications#tag_based_notifications)
- [{{site.data.keyword.mobilepushshort}} API](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-apis-for-push-notification#apis-for-push-notification)
- [{{site.data.keyword.mobilepushshort}}](https://{DomainName}/docs/services/mobilepush?topic=mobile-pushnotification-security-in-push-notifications#overview-push)でのセキュリティー
