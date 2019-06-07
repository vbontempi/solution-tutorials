---
copyright:
  years: 2018, 2019
lastupdated: "2019-04-22"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# データベース駆動型の Slackbot の作成
{: #slack-chatbot-database-watson}

このチュートリアルでは、イベントと会議に関する Db2 データベース・エントリーを作成、検索する Slackbot を作成します。Slackbot は {{site.data.keyword.conversationfull}} サービスに基づいています。アシスタント統合を使用して Slack と {{site.data.keyword.conversationfull}} を統合します。

Slack 統合により、Slack と {{site.data.keyword.conversationshort}} の間でメッセージが交換されます。この際、一部のサーバー・サイド・ダイアログ・アクションによって、Db2 データベースに対する SQL 照会が実行されます。すべての (大量ではない) コードが Node.js に記述されています。

## 達成目標
{: #objectives}

* 統合を使用して {{site.data.keyword.conversationfull}} を Slack に接続する
* {{site.data.keyword.openwhisk_short}} で Node.js アクションを作成、デプロイ、バインドする
* Node.js を使用して {{site.data.keyword.openwhisk_short}} から Db2 データベースにアクセスする

## 使用するサービス
{: #services}

このチュートリアルでは、以下のランタイムとサービスを使用します。
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}} ](https://{DomainName}/catalog/services/db2-warehouse)または[{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


このチュートリアルでは、費用が発生する場合があります。[料金カリキュレーター](https://{DomainName}/pricing/)を使用して、使用見積もりに基づいてコスト見積もりを生成してください。

## アーキテクチャー
{: #architecture}

<p style="text-align: center;">

  ![アーキテクチャー](images/solution19/SlackbotArchitecture.png)
</p>

## 始める前に
{: #prereqs}

このチュートリアルを実行するには、最新バージョンの [{{site.data.keyword.Bluemix_notm}} CLI](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) と {{site.data.keyword.openwhisk_short}} [プラグインがインストールされている](/docs/cli?topic=cloud-cli-plug-ins)必要があります。


## サービスと環境のセットアップ
このセクションでは、必要なサービスをセットアップし、環境を準備します。ほとんどの作業はコマンド・ライン・インターフェース (CLI) からスクリプトを使用して実行できます。これらは GitHub から入手可能です。

1. [GitHub リポジトリー](https://github.com/IBM-Cloud/slack-chatbot-database-watson)を複製し、複製されたディレクトリーに移動します。
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. まだログインしていない場合は、`ibmcloud login` を使用して対話式にログインします。
3. 以下のコマンドを使用して、データベース・サービスを作成する組織とスペースをターゲットとして設定します。
   ```
   ibmcloud target --cf
   ```
4. {{site.data.keyword.dashdbshort}} インスタンスを作成し、このインスタンス名として **eventDB** を設定します。
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
**エントリー (Entry)** プラン以外のプランも使用できます。
5. 後で {{site.data.keyword.openwhisk_short}} からデータベース・サービスにアクセスするための許可が必要です。このため、サービス資格情報を作成し、**slackbotkey** というラベルを付けます。   
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. {{site.data.keyword.conversationshort}} サービスのインスタンスを作成します。名前として **eventConversation** を使用し、無料のライト (Lite) プランを使用します。
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. 次に {{site.data.keyword.openwhisk_short}} のアクションを登録し、サービス資格情報をそれらのアクションにバインドします。アクションの一部は Web アクションとして有効になっており、不正な呼び出しを防ぐためにシークレットが設定されています。シークレットを選択し、パラメーターとして渡します。**YOURSECRET** は適切な値に置き換えてください。

   いずれかのアクションが呼び出され、{{site.data.keyword.dashdbshort}} に表が作成されます。{{site.data.keyword.openwhisk_short}} のアクションを使用することで、ローカル Db2 ドライバーが不要になり、またブラウザー・ベースのインターフェースを使用して表を手動で作成する必要もなくなります。登録とセットアップを行うため、以下のコマンドを実行します。これにより、すべてのアクションが含まれている **setup.sh** ファイルが実行されます。システムでシェル・コマンドがサポートされていない場合は、**setup.sh** ファイルから各行をコピーして、個別に実行します。

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **注:** デフォルトでは、このスクリプトによっていくつかのサンプル・データ行も挿入されます。これを無効にするには、前述のスクリプトで次の行をコメント化します: `#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r`
8. デプロイされているアクションの名前空間情報を抽出します。

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   **/slackdemo/eventInsert** より前の部分をメモします。これはエンコードされた組織とスペースです。次のセクションで必要です。

## スキル / ワークスペースのロード
チュートリアルのこの部分では、事前に定義されているワークスペースまたはスキルを {{site.data.keyword.conversationshort}} サービスにロードします。
1. [{{site.data.keyword.Bluemix_short}} リソース・リスト](https://{DomainName}/resources)で、サービスの概要を開きます。以前のセクションで作成した {{site.data.keyword.conversationshort}} サービスのインスタンスを見つけます。そのエントリーをクリックし、次にサービス別名をクリックします。サービスの詳細が開きます。
2. **「ツールの起動 (Launch Tool)」**をクリックして、{{site.data.keyword.conversationshort}} ツールを表示します。
3. **「スキル (Skills)」**に切り替え、**「スキルの作成 (Create skill)」**をクリックしてから、**「スキルのインポート」**をクリックします。
4. ダイアログで**「JSON ファイルを選択 (Choose JSON file)」**をクリックしてから、ローカル・ディレクトリーにある **assistant-skill.json** ファイルを選択します。**「すべて (インテント、エンティティー、およびダイアログ) (Everything (Intents, Entities, and Dialog))」**のインポート・オプションを変更しないでおき、**「インポート」**をクリックします。これにより、**TutorialSlackbot** という名前の新しいスキルが作成されます。
5. **「ダイアログ (Dialog)」**をクリックします。ダイアログ・ノードが表示されます。ダイアログ・ノードを展開すると、次のような構造が表示されます。

   ダイアログには、支援が必要な質問と単純な感謝を処理するためのノードがあります。**newEvent** ノードとその下位ノードは、必要な入力を収集してから、新しいイベント・レコードを Db2 に挿入するアクションを呼び出します。

   **query events** ノードは、イベントがその ID または日付で検索されたかどうかを明確にします。その後、実際の検索と必要なデータの収集が下位ノード **query events by shortname** と **query event by dates** により実行されます。

   **credential_node** は、ダイアログ・アクションのシークレットと、Cloud Foundry 組織に関する情報をセットアップします。後者は、アクションを呼び出す際に必要です。

  詳細については、すべてのセットアップが完了した後で説明します。
  ![](images/solution19/SlackBot_Dialog.png)   
6. ダイアログ・ノード **credential_node** をクリックし、**「応答 (Then respond with)」**の右側にあるメニュー・アイコンをクリックして JSON エディターを開きます。

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   **org_space** を、以前に取得したエンコードされた組織およびスペースの情報に置き換えます。"@" をすべて "%40" に置き換え、**YOURSECRET** を前述の実際のシークレットに変更します。アイコンをもう一度クリックして JSON エディターを閉じます。

## アシスタントの作成と Slack との統合

次に、前述のスキルに関連するアシスタントを作成し、Slack に統合します。 
1. 左上にある**「スキル (Skills)」**をクリックし、**「アシスタント (Assistants)」**を選択します。次に**「アシスタントの作成 (Create assistant)」**をクリックします。
2. ダイアログで、名前として **TutorialAssistant** を入力してから、**「アシスタントの作成 (Create assistant)」**をクリックします。次に表示される画面で、**「ダイアログ・スキルの追加 (Add dialog skill)」**を選択します。その後**「既存のスキルの追加 (Add existing skill)」**を選択し、リストから**「TutorialSlackbot」**を選択して追加します。
3. スキルを追加したら、**「統合の追加 (Add integration)」**をクリックして、**「管理対象の統合 (Managed integrations)」**リストから**「Slack」**を選択します。
4. 指示に従ってチャットボットを Slack と統合します。

## Slackbot のテストと学習
チャットボットのテスト・ドライブの Slack ワークスペースを開きます。ボットと直接チャットを開始します。

1. メッセージング・フォームに **help** と入力します。ボットが何らかのガイダンスで応答します。
2. 次に **new event** と入力して、新しいイベント・レコードのデータの収集を開始します。必要なすべての入力を収集するため、{{site.data.keyword.conversationshort}} スロットを使用します。
3. 最初にイベント ID またはイベント名を入力します。引用符が必要です。これにより、複雑な名前を入力できます。イベント名として **"Meetup: IBM Cloud"** と入力します。このイベント名は、パターン・ベースのエンティティー **eventName** として定義されています。開始位置と終了位置にはさまざまな種類の二重引用符を使用できます。
4. 次に、イベントの場所を入力します。入力は[システム・エンティティー **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details) に基づいています。使用できる市町村名は {{site.data.keyword.conversationshort}} が認識している市町村名のみという制約があります。市町村名として **Friedrichshafen** を使用します。
5. 次のステップでは、E メール・アドレスや Web サイトの URI などの連絡先情報が求められます。最初に **https://www.ibm.com/events** を使用します。このフィールドにはパターン・ベースのエンティティーを使用します。
6. 次の質問は、開始と終了の日時を収集します。さまざまな入力形式に対応している **sys-date** と **sys-time** が使用されます。開始日に **next Thursday**、時刻に **6 pm** を使用し、終了日付/時刻として次の木曜日の正確な日付 (例: **2019-05-09**) と **22:00** を使用します。
7. 最後に、すべてのデータが収集されるとサマリーが出力され、{{site.data.keyword.openwhisk_short}} アクションとして実装されているサーバー・アクションが呼び出され、新しいレコードが Db2 に挿入されます。その後、ダイアログが下位ノードに切り替わり、コンテキスト変数が削除され、処理環境がクリーンアップされます。入力プロセス全体はいつでもキャンセルできます。キャンセルするには、**cancel**、**exit**、または類似の文字列を入力します。この場合、ユーザーの選択内容が認識され、環境のクリーンアップが行われます。
![](images/solution19/SlackSampleChat.png)   

サンプル・データが入力されたので、ここで検索を実行します。
1. **show event information** と入力します。次に、ID または日付のいずれで検索を行うかが尋ねられます。**name** と入力し、次の質問に対しては **"Think 2019"** と入力します。チャットボットにより、そのイベントに関する情報が表示されるはずです。ダイアログには複数の回答が表示され、この中から選択できます。
2. {{site.data.keyword.conversationshort}} をバックエンドとして使用している場合、より複雑な句を入力して、ダイアログの一部をスキップできます。入力として **show event by the name "Think 2019"** を使用します。チャットボットからイベント・レコードが直接返されます。
3. 次に、日付での検索を行います。検索は日付ペアにより定義されます。イベント開始日はこれらの日付の間でなければなりません。**search conference by date in February 2019** を入力として使用すると、結果は再び **Think 2019** イベントになります。エンティティー **February** は 2 つの日付 (2 月 1 日と 2 月 28 日) として解釈されるため、日付範囲の開始日と終了日を入力したことになります。[年として 2019 が指定されていない場合、それ以降の 2 月として識別される可能性があります](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time)。 

さらに検索を行い新しいイベント・エントリーが作成されたら、チャット履歴を再び表示して、今後のダイアログを改善できます。[{{site.data.keyword.conversationshort}} 資料にある**理解の改善**](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro)に関する説明に従います。


## リソースの削除
{:removeresources}

ルート・ディレクトリーでクリーンアップ・スクリプトを実行すると、{{site.data.keyword.dashdbshort}} からイベント表が削除され、{{site.data.keyword.openwhisk_short}} からアクションが削除されます。これは、コードの変更と拡張を開始する際に役立ちます。クリーンアップ・スクリプトは、{{site.data.keyword.conversationshort}} ワークスペースまたはスキルを変更しません。   
```bash
sh cleanup.sh
```
{:codeblock}

[{{site.data.keyword.Bluemix_short}} リソース・リスト](https://{DomainName}/resources)で、サービスの概要を開きます。{{site.data.keyword.conversationshort}} サービスのインスタンスを見つけ、削除します。

## チュートリアルを発展させる
このチュートリアルに追加またはチュートリアルを変更する必要がありますか?以下にいくつかのアイデアを示します。
1. ワイルドカード検索やイベント期間の検索 ("give me all events longer than 8 hours") などの検索機能を追加します。
2. {{site.data.keyword.dashdbshort}} の代わりに {{site.data.keyword.databases-for-postgresql}} を使用します。[この Slackbot チュートリアル用の GitHub リポジトリー](https://github.com/IBM-Cloud/slack-chatbot-database-watson)には既に、{{site.data.keyword.databases-for-postgresql}} に対応するためのコードが含まれています。
3. 天気予報サービスを追加し、イベントの日付と場所の予報データを取得します。
4. イベント・データを iCalendar **.ics** ファイルとしてエクスポートします。
5. 別の統合を追加して、チャットボットを Facebook Messenger に接続します。
6. インタラクティブ要素 (ボタンなど) を出力に追加します。      


## 関連コンテンツ
{:related}

このチュートリアルで扱ったトピックに関する追加情報へのリンクを以下に示します。

チャットボット関連のブログ投稿:
* [Chatbots: Some tricks with slots in IBM Watson Conversation](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [Lively chatbots: Best Practices](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [Building chatbots: more tips and tricks](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

資料と SDK:
* [IBM Watson Conversation での変数の処理に関するヒントとコツ](https://github.com/IBM-Cloud/watson-conversation-variables)を含む GitHub リポジトリー
* [{{site.data.keyword.openwhisk_short}} の資料](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* 資料: [IBM Knowledge Center for {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [無料の Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) (開発者向け)
* 資料: [ibm_db Node.js ドライバーに関する API の説明](https://github.com/ibmdb/node-ibm_db)
* [{{site.data.keyword.cloudantfull}} の資料](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
