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

# 建置已啟用語音的 Android 聊天機器人
{: #android-watson-chatbot}

瞭解如何使用 {{site.data.keyword.Bluemix_short}} 上的 {{site.data.keyword.conversationshort}}、{{site.data.keyword.texttospeechshort}} 和 {{site.data.keyword.speechtotextshort}} 服務，輕而易舉地快速建立已啟用語音的 Android 原生聊天機器人。

本指導教學會引導您定義目的和實體，以及為聊天機器人建置對話流程以回應客戶查詢的處理程序。您將瞭解如何啟用 {{site.data.keyword.speechtotextshort}} 和 {{site.data.keyword.texttospeechshort}} 服務以與 Android 應用程式進行輕鬆互動。
{:shortdesc}

## 目標
{: #objectives}

- 使用 {{site.data.keyword.conversationshort}} 自訂和部署聊天機器人。
- 容許一般使用者使用語音和音訊與聊天機器人互動。
- 配置和執行 Android 應用程式。

## 使用的服務
{: #services}

本指導教學使用下列產品：

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## 架構
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* 使用者使用自己的聲音與行動應用程式互動。
* 音訊使用 {{site.data.keyword.speechtotextfull}} 轉錄為文字。
* 文字傳遞到 {{site.data.keyword.conversationfull}}。
* 來自 {{site.data.keyword.conversationfull}} 的回覆由 {{site.data.keyword.texttospeechfull}} 轉換為音訊，然後結果會傳送回行動應用程式。

## 開始之前
{: #prereqs}

- 下載並安裝 [Android Studio](https://developer.android.com/studio/index.html)。

## 建立服務
{: #setup}

在本節中，從 {{site.data.keyword.conversationshort}} 開始，您將建立指導教學所需的服務，以建置可協助客戶的認知虛擬助理。

1. 移至 [**{{site.data.keyword.Bluemix_notm}} 型錄**](https://{DomainName}/catalog/)，然後選取 [{{site.data.keyword.conversationshort}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation) 服務 > **精簡**方案：
   1. 將**名稱**設定為 **android-chatbot-assistant**。
   1. **建立**。
2. 按一下左側窗格中的**服務認證**，然後按一下**新建認證**。
   1. 將**名稱**設定為 **for-android-app**。
   1. **新增**。
3. 按一下**檢視認證**以查看認證。記下 **API 金鑰**及 **URL**，行動應用程式將需要它。

{{site.data.keyword.speechtotextshort}} 服務用於將人聲轉換為書面文字，然後可以將其作為輸入傳送到 {{site.data.keyword.Bluemix_short}} 上的 {{site.data.keyword.conversationshort}} 服務。

1. 移至 [**{{site.data.keyword.Bluemix_notm}} 型錄**](https://{DomainName}/catalog/)，然後選取 [{{site.data.keyword.speechtotextshort}}](https://{DomainName}/catalog/services/speech-to-text) 服務 > **精簡**方案。
   1. 將**名稱**設定為 **android-chatbot-stt**。
   1. **建立**。
2. 按一下左窗格的**服務認證**，然後按一下**新認證**來新增認證。
   1. 將**名稱**設定為 **for-android-app**。
   1. **新增**。
3. 按一下**檢視認證**以查看認證。記下 **API 金鑰**及 **URL**，行動應用程式將需要它。

{{site.data.keyword.texttospeechshort}} 服務用於處理文字和自然語言，以產生具有適當節奏和語調的合成音訊輸出。此服務提供多種語音，可以在 Android 應用程式中進行配置。

1. 移至 [**{{site.data.keyword.Bluemix_notm}} 型錄**](https://{DomainName}/catalog/)，然後選取 [{{site.data.keyword.texttospeechshort}}](https://{DomainName}/catalog/services/text-to-speech) 服務 > **精簡**方案。
   1. 將**名稱**設定為 **android-chatbot-tts**。
   1. **建立**。
2. 按一下左窗格的**服務認證**，然後按一下**新認證**來新增認證。
   1. 將**名稱**設定為 **for-android-app**。
   1. **新增**。
3. 按一下**檢視認證**以查看認證。記下 **API 金鑰**及 **URL**，行動應用程式將需要它。

## 建立技能
{: #create_workspace}

技能是一種容器，用於包含定義交談流程的構件。

在本指導教學中，您將儲存並使用 [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) 檔案，其中包含預先定義的目的、實體和與機器的對話流程。

1. 在 {{site.data.keyword.conversationshort}} 服務詳細資料頁面中，導覽至左側窗格中的**管理**，按一下**啟動工具**以查看 {{site.data.keyword.conversationshort}} 儀表板。
1. 按一下**技能**標籤。
1. 按一下**建立新的項目**，然後按一下**匯入技能**，並選擇上面下載的 JSON 檔案。
1. 選取**全部**選項，然後按一下**匯入**。這將使用預先定義的目的、實體和對話流程建立新技能。
1. 回到「技能」清單。選取 `Ana` 技能上的動作功能表以**檢視 API 詳細資料**。

### 定義目的
{:#define_intent}

目的代表使用者輸入的用途，例如回答問題或處理帳單付款。您可以為您想要應用程式支援的每一種使用者要求類型定義一個目的。藉由識別使用者輸入中表達的目的，{{site.data.keyword.conversationshort}} 服務可以選擇正確的對話流程來進行回應。在工具中，目的名稱前面一律會加上 `#` 字元。

簡單來說，目的就是一般使用者的意圖。下面是目的名稱的範例。
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. 按一下新建立的技能 - **Ana**。

   Ana 是一個保險機器人，使用者可以使用該機器人來查詢自己的健康福利和提出理賠。
{:tip}
2. 按一下第一個標籤來查看所有**目的**。
3. 按一下**新增目的**來建立新目的。在 `#` 後輸入 `Cancel_Policy` 作為目的名稱，並提供選用的描述。
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. 按一下**建立目的**。
5. 新增要求取消保單的使用者範例
   - `I want to cancel my policy`
   - `Drop my policy now`
   - `I wish to stop making payments on my policy.`
6. 逐一新增上述使用者範例，然後按一下**新增範例**。對於所有其他使用者範例，重複此步驟。

   請記住至少新增 5 個使用者範例，以便更完善地訓練機器人。
   {:tip}

7. 按一下目的名稱旁邊的**關閉** ![](images/solution28-watson-chatbot-android/close_icon.png) 按鈕以儲存目的。
8. 按一下**內容型錄**，然後選取**一般**。按一下**新增到技能**。

   內容型錄藉由新增現有目的（銀行、客戶服務、保險、電話公司、電子商務等），可協助您更快開始使用。這些目的已根據使用者可能會問的常見問題進行訓練。
   {:tip}

### 定義實體
{:#define_entity}

實體代表與目的相關並為目的提供特定環境定義的術語或物件。您可為每個實體和使用者可能輸入的同義字列出可能的值。藉由識別使用者輸入中提及的實體，{{site.data.keyword.conversationshort}} 服務可以選擇執行特定動作來實現目的。在工具中，實體名稱前面一律會加上 `@` 字元。

下面是實體名稱的範例。
 - `@location`
 - `@menu_item`
 - `@product`

1. 按一下**實體**標籤來查看現有實體。
2. 按一下**新增實體**，然後在 `@` 後面輸入實體的名稱 `location`。按一下**建立實體**。
3. 輸入 `address` 作為值名稱，然後選取**同義字**。
4. 新增 `place` 作為同義字，然後按一下 ![](images/solution28-watson-chatbot-android/plus_icon.png) 圖示。對同義字 `office`、`centre`、`branch` 等重複此作業，然後按一下**新增值**。
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. 按一下**關閉** ![](images/solution28-watson-chatbot-android/close_icon.png) 以儲存變更。
6. 按一下**系統實體**標籤來查看 IBM 建立的可在任何使用案例中使用的公共實體。

   系統實體可用來辨識其代表之物件類型的更大範圍的值。例如，`@sys-number` 系統實體符合任何數值，包括整數、小數位數，甚至是以字組形式寫出的數字。
   {:tip}
7. 對於 @sys-person 和 @sys-location 系統實體，將**狀態**從 off 切換到 `on`。

### 建置對話流程
{:#build_dialog}

對話是一種分支交談流程，用於定義應用程式在識別到定義的目的和實體時如何進行回應。使用工具中的對話建置器可建立與使用者的交談，並根據在其輸入中識別到的目的和實體來提供回應。

1. 按一下**對話**標籤來查看具有目的和實體的現有對話流程。
2. 按一下**新增節點**向對話新增新節點。
3. 在**如果助理識別到**下，輸入 `#Cancel_Policy`。
4. 在**則回應為**下，輸入回應：`This facility is not available online. Please visit our nearest branch to cancel your policy.`
5. 按一下 ![](images/solution28-watson-chatbot-android/save_node.png) 以關閉並儲存該節點。
6. 捲動至看到 `#greeting` 節點。按一下該節點來查看詳細資料。
   ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. 按一下 ![](images/solution28-watson-chatbot-android/add_condition.png) 圖示來**新增條件**。從下拉清單中選取 `or`，然後輸入 `#General_Greetings` 作為目的。**則回應為**區段會顯示助理在使用者問候時的回應。
   ![](images/solution28-watson-chatbot-android/apply_condition.png)

   環境定義變數是在節點中定義的變數，選擇性為其指定預設值。其他節點或應用程式邏輯可以隨後設定或變更環境定義變數的值。應用程式可以將資訊傳遞給對話，而對話可以更新此資訊，並將它傳回給應用程式或後續節點。對話使用環境定義變數來執行此作業。
   {:tip}

8. 使用**試用**按鈕來測試對話流程。

## 將技能鏈結到助理

**助理**是一種認知機器人，您可以根據自己的商業需求進行自訂，並跨多個頻道進行部署，以隨時隨地根據客戶需要向客戶提供協助。可以藉由向助理新增滿足客戶目標所需的**技能**來自訂助理。

1. 在 {{site.data.keyword.conversationshort}} 工具中，切換到**助理**，然後使用**建立新的項目**。
   1. 將**名稱**設定為 **android-chatbot-assistant**。
   1. **建立**
1. 使用**新增對話技能**以選取前幾節中建立的技能。
   1. **新增現有技能**
   1. 選取 **Ana**
1. 選取「助理」> **設定** > **API 詳細資料**中的動作功能表，記下**助理 ID**，在行動應用程式中需要參照此 ID（在 Android 應用程式的 `config.xml` 檔案中）。

## 配置和執行 Android 應用程式
{:#configure_run_android_app}

儲存庫包含具有必要 Gradle 相依關係的 Android 應用程式碼。

1. 執行以下指令來複製 [GitHub 儲存庫](https://github.com/IBM-Cloud/chatbot-watson-android)：
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. 啟動 Android Studio > **開啟現有 Android Studio 專案**，然後指向已下載的程式碼。這將自動觸發 **Gradle** 建置，並下載所有相依關係。
3. 開啟 `app/src/main/res/values/config.xml` 以查看服務認證的位置保留元 (`ASSISTANT_ID_HERE`)。在個別位置保留元中輸入服務認證（稍早儲存的服務認證），然後儲存該檔案。
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
4. 建置專案並在實際裝置上或使用模擬器啟動應用程式。
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. 在下面提供的空白處**輸入查詢**，然後按一下箭頭圖示以將查詢傳送到 {{site.data.keyword.conversationshort}} 服務。
6. 回應將傳遞到 {{site.data.keyword.texttospeechshort}} 服務，並且您應該會聽到有聲音在讀出回應。
7. 按一下應用程式左下角的**麥克風**圖示，以輸入轉換為文字的語音，然後按一下箭頭圖示將其傳送到 {{site.data.keyword.conversationshort}} 服務。


## 移除資源
{:removeresources}

1. 導覽至[資源清單](https://{DomainName}/resources/)。
1. 刪除已建立的服務：
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## 相關內容
{:related}

- [建立實體、同義字和系統實體](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [環境定義變數](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [建置複雜對話](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [使用空位收集資訊](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [部署選項                                     ](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [改善您的技能](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
