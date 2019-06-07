---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-29"
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

# 音声対応の Android チャットボットの作成
{: #android-watson-chatbot}

{{site.data.keyword.Bluemix_short}} の {{site.data.keyword.conversationshort}}、{{site.data.keyword.texttospeechshort}}、および {{site.data.keyword.speechtotextshort}} サービスで音声対応の Android ネイティブのチャットボットを素早く簡単に作成する方法を学習します。

このチュートリアルでは、インテントとエンティティーを定義し、顧客の照会に応答するチャットボットのダイアログ・フローを作成するプロセスについて説明します。Android アプリと簡単に対話するための {{site.data.keyword.speechtotextshort}} サービスおよび {{site.data.keyword.texttospeechshort}} サービスを有効にする方法を学習します。
{:shortdesc}

## 達成目標
{: #objectives}

- {{site.data.keyword.conversationshort}} を使用して、チャットボットのカスタマイズとデプロイを行います。
- エンド・ユーザーが音声を使用してチャットボットと対話できるようにします。
- Android アプリを構成して実行します。

## 使用するサービス
{: #services}

このチュートリアルでは、以下の製品を使用します。

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* ユーザーが音声を使用してモバイル・アプリケーションと対話します。
* {{site.data.keyword.speechtotextfull}} を使用して音声がテキストに転記されます。
* テキストが {{site.data.keyword.conversationfull}} に渡されます。
* {{site.data.keyword.conversationfull}} からの応答が {{site.data.keyword.texttospeechfull}} によって音声に変換され、結果がモバイル・アプリケーションに送信されます。

## 始める前に
{: #prereqs}

- [Android Studio](https://developer.android.com/studio/index.html) をダウンロードしてインストールします。

## サービスの作成
{: #setup}

このセクションでは、{{site.data.keyword.conversationshort}} で開始されるチュートリアルに必要なサービスを作成して、顧客を支援するコグニティブな仮想アシスタントを作成します。

1. [**{{site.data.keyword.Bluemix_notm}} カタログ**](https://{DomainName}/catalog/)に移動して、[「{{site.data.keyword.conversationshort}}」サービス](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)>**「ライト」**プランを選択します。
   1. **「名前」**を **android-chatbot-assistant** に設定します。
   1. **「作成」**。
2. 左側のペインで**「サービス資格情報」**をクリックし、**「新規資格情報」**をクリックします。
   1. **「名前」**を **for-android-app** に設定します。
   1. **「追加」**。
3. **「資格情報の表示」**をクリックして、資格情報を表示します。モバイル・アプリケーションに必要となる **API キー**と **URL** をメモします。

{{site.data.keyword.speechtotextshort}} サービスは、人間の声を、{{site.data.keyword.Bluemix_short}} の {{site.data.keyword.conversationshort}} サービスに入力として送信できる書き言葉に変換します。

1. [**{{site.data.keyword.Bluemix_notm}} カタログ**](https://{DomainName}/catalog/)に移動して、[「{{site.data.keyword.speechtotextshort}}」サービス](https://{DomainName}/catalog/services/speech-to-text) >**「ライト」**プランを選択します。
   1. **「名前」**を **android-chatbot-stt** に設定します。
   1. **「作成」**。
2. 左側のペインで**「サービス資格情報」**をクリックし、**「新規資格情報」**をクリックして、新規資格情報を追加します。
   1. **「名前」**を **for-android-app** に設定します。
   1. **「追加」**。
3. **「資格情報の表示」**をクリックして、資格情報を表示します。モバイル・アプリケーションに必要となる **API キー**と **URL** をメモします。

{{site.data.keyword.texttospeechshort}} サービスは、テキストおよび自然言語を処理し、適切なリズムとイントネーションを備えた合成音声出力を生成します。このサービスは複数の音声を提供し、Android アプリで構成できます。

1. [**{{site.data.keyword.Bluemix_notm}} カタログ**](https://{DomainName}/catalog/)に移動して、[「{{site.data.keyword.texttospeechshort}}」サービス](https://{DomainName}/catalog/services/text-to-speech) >**「ライト」**プランを選択します。
   1. **「名前」**を **android-chatbot-tts** に設定します。
   1. **「作成」**。
2. 左側のペインで**「サービス資格情報」**をクリックし、**「新規資格情報」**をクリックして、新規資格情報を追加します。
   1. **「名前」**を **for-android-app** に設定します。
   1. **「追加」**。
3. **「資格情報の表示」**をクリックして、資格情報を表示します。モバイル・アプリケーションに必要となる **API キー**と **URL** をメモします。

## スキルの作成
{: #create_workspace}

スキルとは、会話フローを定義する成果物のコンテナーです。

このチュートリアルでは、インテント、エンティティー、およびマシンへのダイアログ・フローが事前定義されている [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) ファイルを保存して使用します。

1. {{site.data.keyword.conversationshort}} サービスの詳細ページの左側のペインで**「管理」**にナビゲートし、**「ツールの起動 (Launch tool)」**をクリックして、{{site.data.keyword.conversationshort}} ダッシュボードを表示します。
1. **「スキル (Skills)」**タブをクリックします。
1. **「新規作成 (Create new)」**、**「スキルのインポート (Import skill)」**の順にクリックし、前述のダウンロードされた JSON ファイルを選択します。
1. **「すべて (Everything)」**オプションを選択し、**「インポート (Import)」**をクリックします。事前定義されたインテント、エンティティー、およびダイアログ・フローを使用して新しいスキルが作成されます。
1. スキルのリストに戻ります。`Ana` スキルのアクション・メニューとして**「API 詳細の表示 (View API Details)」**を選択します。

### インテントの定義
{:#define_intent}

インテントは、ユーザー入力の目的 (問い合わせへの回答や請求書の支払い処理など) を表します。インテントの定義は、アプリケーションがサポートするユーザー要求の各タイプに対して行います。 ユーザーの入力で表現されたインテントを認識することによって、{{site.data.keyword.conversationshort}} サービスは、そのインテントに応答するための正しいダイアログ・フローを選択することができます。ツールでは、インテントの名前には常に `#` 文字が接頭部として付けられます。

インテントとは、簡単に言って、エンド・ユーザーの意図です。 以下にインテント名の例を示します。
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. 新しく作成したスキル**「Ana」**をクリックします。

   Ana は、ユーザーが医療サービスについて照会し、請求を行うための保険ボットです。
   {:tip}
2. 最初のタブをクリックして、すべての**インテント**を表示します。
3. **「インテントの追加 (Add intent)」**をクリックして、新しいインテントを作成します。インテント名として `#` の後に `Cancel_Policy` と入力し、オプションの説明を指定します。
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. **「インテントの作成 (Create intent)」**をクリックします。
5. 要求されたら、ユーザー例を追加して、ポリシーを取り消します。
   - `I want to cancel my policy`
   - `Drop my policy now`
   - `I wish to stop making payments on my policy.`
6. ユーザー例を順に追加して、**「例の追加 (add example)」**をクリックします。その他すべてのユーザー例について、これを繰り返します。

   ボットをより適切にトレーニングするために、少なくとも 5 つのユーザー例を追加してください。
   {:tip}

7. インテント名の横の**「閉じる (close)」**![](images/solution28-watson-chatbot-android/close_icon.png) ボタンをクリックして、インテントを保存します。
8. **「コンテンツ・カタログ (Content Catalog)」**をクリックし、**「一般 (General)」**を選択します。**「スキルに追加 (Add to skill)」**をクリックします。

   コンテンツ・カタログは、既存のインテント (銀行、カスタマー・ケア、保険、通信会社、e-コマースなど) を追加することによって、より迅速に開始するために役立ちます。これらのインテントは、ユーザーが問い合わせる可能性がある一般的な質問でトレーニングされます。
   {:tip}

### エンティティーの定義
{:#define_entity}

エンティティーは、インテントに関連し、かつ、インテントに特定のコンテキストを提供する用語またはオブジェクトを表します。各エンティティーに対して可能性のある値、およびユーザーが入力する可能性のある同義語をリストします。ユーザーの入力で示されたエンティティーを認識することによって、{{site.data.keyword.conversationshort}} サービスは、インテントを充足するために実行する特定のアクションを選択できます。ツールでは、エンティティーの名前には常に `@` 文字が接頭部として付けられます。

以下にエンティティー名の例を示します
 - `@location`
 - `@menu_item`
 - `@product`

1. **「エンティティー (Entities)」**タブをクリックして、既存のエンティティーを表示します。
2. **「エンティティーの追加 (Add entity)」**をクリックし、エンティティー名として `@` の後に `location` と入力します。**「エンティティーの作成 (Create entity)」**をクリックします。
3. 値名として `address` を入力し、**「同義語 (Synonyms)」**を選択します。
4. 同義語として `place` を追加し、![](images/solution28-watson-chatbot-android/plus_icon.png) アイコンをクリックします。同義語 `office`、`centre`、`branch` などで繰り返し、**「値の追加 (Add Value)」**をクリックします。
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. **「閉じる (close)」**![](images/solution28-watson-chatbot-android/close_icon.png) をクリックして、変更を保存します。
6. **「システム・エンティティー (System entities)」**タブをクリックして、任意のユース・ケースで使用できる、IBM によって作成された共通のエンティティーを確認します。

   システム・エンティティーを使用すると、それらのエンティティーが表すオブジェクト・タイプの、広範囲の値を認識することができます。 例えば、`@sys-number` システム・エンティティーは、単語として表記された数字も含め、整数、小数など、あらゆる数値と一致します。
   {:tip}
7. @sys-person および @sys-location システム・エンティティーの**「状況」**を off から `on` に切り替えます。

### ダイアログ・フローのビルド
{:#build_dialog}

ダイアログは、定義済みのインテントとエンティティーを認識したときに、アプリケーションがどのように応答するかを定義する会話フロー・ブランチです。 ツールのダイアログ・ビルダーを使用して、ユーザー入力内で認識されるインテントとエンティティーに基づいて応答を提供するユーザーとの会話を作成します。

1. **「ダイアログ (Dialog)」**タブをクリックして、インテントおよびエンティティーとともに既存のダイアログ・フローを表示します。
2. **「ノードの追加 (Add node)」**をクリックして、ダイアログに新しいノードを追加します。
3. **「アシスタントが認識した場合 (if assistant recognizes)」**に `#Cancel_Policy` と入力します。
4. **「応答 (Then respond with)」**に `This facility is not available online. Please visit our nearest branch to cancel your policy.` と入力します。
5. ![](images/solution28-watson-chatbot-android/save_node.png) をクリックして、ノードを閉じて保存します。
6. スクロールして、`#greeting` ノードを表示します。 このノードをクリックして、詳細を表示します。
   ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. ![](images/solution28-watson-chatbot-android/add_condition.png) アイコンをクリックして、**新しい条件を追加します**。 ドロップダウンから `or` を選択し、インテントとして `#General_Greetings` を入力します。 **「セクションで応答 (Then respond with section)」**に、ユーザーが応じた場合のアシスタントの応答が表示されます。
   ![](images/solution28-watson-chatbot-android/apply_condition.png)

   コンテキスト変数はノード内で定義する変数で、必要な場合にはデフォルト値を指定できます。コンテキスト変数の値は、後から他のノードまたはアプリケーション・ロジックで設定したり変更したりできます。アプリケーションからダイアログに情報を渡すことができます。また、ダイアログでその情報を更新してアプリケーションに戻したり、後続のノードに渡したりすることもできます。 そのために、ダイアログはコンテキスト変数を使用します。
   {:tip}

8. **「試す (Try it)」**ボタンを使用して、ダイアログ・フローをテストします。

## アシスタントへのスキルのリンク

**アシスタント**は、ビジネス・ニーズに合わせてカスタマイズし、必要な場面で顧客にヘルプを提供するために複数のチャネルにデプロイできるコグニティブ・ボットです。 顧客の目標を達成するために必要な**スキル**を追加することで、アシスタントをカスタマイズします。

1. {{site.data.keyword.conversationshort}} ツールで、**「アシスタント (Assistants)」**に切り替えて**「新規作成 (Create new)」**を使用します。
   1. **「名前」**を **android-chatbot-assistant** に設定します
   1. **「作成」**
1. **「ダイアログ・スキルの追加 (Add Dialog skill)」**を使用して、前のセクションで作成したスキルを選択します。
   1. **「既存のスキルの追加 (Add existing skill)」**
   1. **Ana** を選択します
1. 「アシスタント (Assistant)」>**「設定」**>**「API 詳細 (API Details)」**でアクション・メニューを選択し、モバイル・アプリケーションから参照する必要がある**「アシスタント ID (Assistant ID)」**をメモします (Android アプリの `config.xml` ファイル)。

## Android アプリの構成と実行
{:#configure_run_android_app}

リポジトリーには、必要な gradle 依存関係とともに Android アプリケーション・コードが含まれています。

1. 以下のコマンドを実行して、[GitHub リポジトリー](https://github.com/IBM-Cloud/chatbot-watson-android)を複製します。
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. Android Studio を起動し、**既存の Android Studio プロジェクトを開き**、ダウンロードされたコードをポイントします。**Gradle** ビルドが自動的に起動し、すべての依存関係がダウンロードされます。
3. `app/src/main/res/values/config.xml` を開き、サービス資格情報のプレースホルダー (`ASSISTANT_ID_HERE`) を表示します。 それぞれのプレースホルダーにサービス資格情報 (前に保存したもの) を入力し、ファイルを保存します。
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <!--Watson Assistant service credentials-->
       <!-- REPLACE `ASSISTANT_ID_HERE` with ID of the Assistant to use -->
       <string name="assistant_id">ASSISTANT_ID_HERE</string>

       <!-- REPLACE `ASSISTANT_API_KEY_HERE` with Watson Assistant service API Key-->
       <string name="assistant_apikey">ASSISTANT_API_KEY_HERE</string>

       <!-- REPLACE `ASSISTANT_URL_HERE` with Watson Assistant service URL-->
       <string name="assistant_url">ASSISTANT_URL_HERE</string>

       <!--Watson Speech To Text(STT) service credentials-->
       <!-- REPLACE `STT_API_KEY_HERE` with Watson Speech to Text service API Key-->
       <string name="STT_apikey">STT_API_KEY_HERE</string>

       <!-- REPLACE `STT_URL_HERE` with Watson Speech to Text service URL-->
       <string name="STT_url">STT_URL_HERE</string>

       <!--Watson Text To Speech(TTS) service credentials-->
       <!-- REPLACE `TTS_API_KEY_HERE` with Watson Text to Speech service API Key-->
       <string name="TTS_apikey">TTS_API_KEY_HERE</string>

       <!-- REPLACE `TTS_URL_HERE` with Watson Text to Speech service URL-->
       <string name="TTS_url">TTS_URL_HERE</string>
   </resources>
   ```
4. プロジェクトを作成し、実際のデバイスまたはシミュレーターでアプリケーションを開始します。
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. 下部に提供されているスペースに**照会を入力**し、矢印アイコンをクリックして、照会を {{site.data.keyword.conversationshort}} サービスに送信します。
6. 応答が {{site.data.keyword.texttospeechshort}} サービスに渡され、この応答を読み上げる音声が聞こえます。
7. アプリの左下隅にある**「マイク (mic)」**アイコンをクリックして、音声を入力します。これはテキストに変換され、矢印アイコンをクリックすると、{{site.data.keyword.conversationshort}} サービスに送信できます。


## リソースの削除
{:removeresources}

1. [「リソース・リスト」](https://{DomainName}/resources/)にナビゲートします。
1. 作成したサービスを削除します。
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## 関連コンテンツ
{:related}

- [エンティティー、同義語、システム・エンティティーの作成](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [コンテキスト変数](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [複雑なダイアログの作成](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [スロットを使用した情報の収集](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [デプロイメント・オプション](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [スキルの改善](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
