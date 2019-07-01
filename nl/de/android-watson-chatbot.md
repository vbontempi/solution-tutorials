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

# Sprachgesteuerten Android-Chatbot erstellen
{: #android-watson-chatbot}

Erfahren Sie, wie einfach es ist, schnell einen sprachaktivierten Android-basierten Chatbot mit {{site.data.keyword.conversationshort}}-, {{site.data.keyword.texttospeechshort}}- und {{site.data.keyword.speechtotextshort}}-Services unter {{site.data.keyword.Bluemix_short}} zu erstellen.

In diesem Lernprogramm wird gezeigt, wie Sie Absichten und Entitäten definieren und einen Dialogablauf für Ihren Chatbot zum Antworten aus Kundenanfragen erstellen können. Außerdem erfahren Sie, wie Sie {{site.data.keyword.speechtotextshort}}- und {{site.data.keyword.texttospeechshort}}-Services für eine einfache Interaktion mit der Android-App aktivieren können.
{:shortdesc}

## Lernziele
{: #objectives}

- {{site.data.keyword.conversationshort}} verwenden, um einen Chatbot anzupassen und zu implementieren
- Endbenutzern die Interaktion über einen Chatbot mit Sprachunterstützung und Audio ermöglichen
- Android-App konfigurieren und ausführen

## Verwendete Services
{: #services}

Dieses Lernprogramm verwendet die folgenden Produkte:

- [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)
- [{{site.data.keyword.speechtotextfull}}](https://{DomainName}/catalog/services/speech-to-text)
- [{{site.data.keyword.texttospeechfull}}](https://{DomainName}/catalog/services/text-to-speech)

## Architektur
{: #architecture}

<p style="text-align: center;">

![](images/solution28-watson-chatbot-android/architecture.png)
</p>

* Benutzer interagieren mit einer mobilen Anwendung über ihre Stimme.
* Der Audiotext wird mit {{site.data.keyword.speechtotextfull}} in Text umgeschrieben.
* Der Text wird an {{site.data.keyword.conversationfull}} übergeben.
* Die Antwort von {{site.data.keyword.conversationfull}} wird von {{site.data.keyword.texttospeechfull}} in Audio konvertiert und das Ergebnis wird zurück an die mobile Anwendung gesendet.

## Vorbereitende Schritte
{: #prereqs}

- Laden Sie [Android Studio](https://developer.android.com/studio/index.html) herunter und installieren Sie es.

## Services erstellen
{: #setup}

In diesem Abschnitt erstellen Sie beginnend mit {{site.data.keyword.conversationshort}} die Services, die für das Lernprogramm erforderlich sind, um kognitive virtuelle Assistenten zur Unterstützung Ihrer Kunden zu erstellen.

1. Rufen Sie den [**{{site.data.keyword.Bluemix_notm}}-Katalog**](https://{DomainName}/catalog/) auf und wählen Sie [{{site.data.keyword.conversationshort}}](https://{DomainName}/catalog/services/watson-assistant-formerly-conversation)-Service > **Lite-Plan** aus.
   1. Setzen Sie den **Namen** auf **android-chatbot-assistant**.
   1. Klicken Sie auf **Erstellen**.
2. Klicken Sie im linken Teilfenster auf **Serviceberechtigungsnachweise** und klicken Sie auf **Neuer Berechtigungsnachweis**.
   1. Setzen Sie den **Namen** auf **for-android-app**.
   1. Klicken Sie auf **Hinzufügen**.
3. Klicken Sie auf **Berechtigungsnachweise anzeigen**, um die Berechtigungsnachweise anzuzeigen. Notieren Sie sich den **API-Schlüssel** und die **URL**. Sie benötigen sie für die mobile Anwendung.

Der {{site.data.keyword.speechtotextshort}}-Service konvertiert die menschliche Stimme in das geschriebene Wort, das als Eingabe an den {{site.data.keyword.conversationshort}}-Service in {{site.data.keyword.Bluemix_short}} gesendet werden kann.

1. Rufen Sie den [**{{site.data.keyword.Bluemix_notm}}-Katalog**](https://{DomainName}/catalog/) auf und wählen Sie [{{site.data.keyword.speechtotextshort}}](https://{DomainName}/catalog/services/speech-to-text)-Service > **Lite-Plan** aus.
   1. Setzen Sie den **Namen** auf **android-chatbot-stt**.
   1. Klicken Sie auf **Erstellen**.
2. Klicken Sie im linken Teilfenster auf **Serviceberechtigungsnachweise** und klicken Sie auf **Neuer Berechtigungsnachweis**, um einen neuen Berechtigungsnachweis hinzuzufügen.
   1. Setzen Sie den **Namen** auf **for-android-app**.
   1. Klicken Sie auf **Hinzufügen**.
3. Klicken Sie auf **Berechtigungsnachweise anzeigen**, um die Berechtigungsnachweise anzuzeigen. Notieren Sie sich den **API-Schlüssel** und die **URL**. Sie benötigen sie für die mobile Anwendung.

Der {{site.data.keyword.texttospeechshort}}-Service verarbeitet Text und natürliche Sprache, um eine synthetisierte Audioausgabe mit entsprechender Kadenz und Intonation zu erzeugen. Der Service stellt mehrere Stimmen zur Verfügung und kann in der Android-App konfiguriert werden.

1. Rufen Sie den [**{{site.data.keyword.Bluemix_notm}}-Katalog**](https://{DomainName}/catalog/) auf und wählen Sie [{{site.data.keyword.texttospeechshort}}](https://{DomainName}/catalog/services/text-to-speech)-Service > **Lite-Plan** aus.
   1. Setzen Sie den **Namen** auf **android-chatbot-tts**.
   1. Klicken Sie auf **Erstellen**.
2. Klicken Sie im linken Teilfenster auf **Serviceberechtigungsnachweise** und klicken Sie auf **Neuer Berechtigungsnachweis**, um einen neuen Berechtigungsnachweis hinzuzufügen.
   1. Setzen Sie den **Namen** auf **for-android-app**.
   1. Klicken Sie auf **Hinzufügen**.
3. Klicken Sie auf **Berechtigungsnachweise anzeigen**, um die Berechtigungsnachweise anzuzeigen. Notieren Sie sich den **API-Schlüssel** und die **URL**. Sie benötigen sie für die mobile Anwendung.

## Skill erstellen
{: #create_workspace}

Ein Skill ist ein Container für die Artefakte, die den Dialogablauf definieren.

Für dieses Lernprogramm speichern und verwenden Sie die Datei [Ana_skill.json](https://github.com/IBM-Cloud/chatbot-watson-android/raw/master/training/Ana_skill.json) mit vordefinierten Absichten, Entitäten und Dialogabläufen auf Ihrer Maschine.

1. Navigieren Sie auf der Seite mit den Servicedetails von {{site.data.keyword.conversationshort}} im linken Teilfenster zu **Verwalten** und klicken Sie auf **Tool starten**, um das {{site.data.keyword.conversationshort}}-Dashboard anzuzeigen.
1. Klicken Sie auf die Registerkarte **Skills**.
1. Klicken Sie auf **Neu erstellen** und anschließend auf **Skill importieren** und wählen Sie die heruntergeladene JSON-Datei aus.
1. Wählen Sie die Option **Alles** aus und klicken Sie auf **Importieren**. Ein neuer Skill wird mit vordefinierten Absichten, Entitäten und Dialogabläufen erstellt.
1. Wechseln Sie zurück zur Liste der Skills. Wählen Sie das Aktionsmenü für den Skill `Ana` aus, um **API-Details anzuzeigen**.

### Absicht definieren
{:#define_intent}

Eine Absicht stellt den Zweck einer Benutzereingabe dar, z. B. die Beantwortung einer Frage oder die Verarbeitung einer Rechnungszahlung. Sie definieren eine Absicht für jeden Typ von Benutzeranforderung, den Ihre Anwendung unterstützen soll. Indem er die in einer Benutzereingabe ausgedrückte Absicht erkennt, kann der {{site.data.keyword.conversationshort}}-Service als Reaktion darauf den richtigen Dialogablauf auswählen. Im Tool wird dem Namen einer Absicht immer das Präfix `#` vorangestellt.

Einfach ausgedrückt sind Absichten die Intentionen des Endbenutzers. Nachfolgend sind Beispiele von Absichtsnamen aufgeführt.
 - `#weather_conditions`
 - `#pay_bill`
 - `#escalate_to_agent`

1. Klicke Sie auf den neu erstellten Skill **Ana**.

   Ana ist ein Versicherungs-Bot für Benutzer, um ihre Gesundheitsleistungen abzufragen und Ansprüche anzumelden.
   {:tip}
2. Klicken Sie auf die erste Registerkarte, um alle **Absichten** anzuzeigen.
3. Klicken Sie auf **Absicht hinzufügen**, um eine neue Absicht zu erstellen. Geben Sie `Cancel_Policy` als Name der Absicht hinter `#` ein und geben Sie eine optionale Beschreibung an.
   ![](images/solution28-watson-chatbot-android/add_intent.png)
4. Klicken Sie auf **Absicht erstellen**.
5. Fügen Sie Benutzerbeispiele hinzu, wenn Sie aufgefordert werden, eine Police zu kündigen.
   - `Ich möchte meine Police kündigen`
   - `Meine Police soll jetzt gelöscht werden`
   - `Ich möchte keine weiteren Zahlungen für meine Police leisten`
6. Fügen Sie Benutzerbeispiele nacheinander hinzu und klicken Sie auf **Beispiel hinzufügen**. Wiederholen Sie dies für alle anderen Benutzerbeispiele.

   Sie sollten mindestens fünf Benutzerbeispiele für ein optimales Training für Ihren Bot hinzufügen.
   {:tip}

7. Klicken Sie auf die Schaltfläche **Schließen** ![](images/solution28-watson-chatbot-android/close_icon.png) neben dem Namen der Absicht, um die Absicht zu speichern.
8. Klicken Sie auf **Inhaltskatalog** und wählen Sie **Allgemein** aus. Klicken Sie auf **Zu Skill hinzufügen**.

   Der Inhaltskatalog hilft Ihnen dabei, schneller zu arbeiten, indem Sie vorhandene Absichten (Bankwesen, Kundenbetreuung, Versicherung, Telekommunikationsunternehmen, E-Commerce und viele mehr) hinzufügen. Diese Absichten werden für allgemeine Fragen trainiert, die Benutzer stellen können.
   {:tip}

### Entität definieren
{:#define_entity}

Eine Entität stellt eine Bedingung oder ein Objekt dar, die/das für Ihre Absichten von Bedeutung ist und einen bestimmten Kontext für eine Absicht liefert. Sie listen die möglichen Werte für jede Entität und Synonyme auf, die Benutzer eingeben können. Durch die Erkennung der Entitäten, die in der Eingabe des Benutzers erwähnt werden, kann der {{site.data.keyword.conversationshort}}-Service die spezifischen Aktionen auswählen, die ausgeführt werden müssen, um eine Absicht zu erfüllen. Im Tool wird dem Namen einer Entität immer das Präfix `@` vorangestellt.

Nachfolgend sind Beispiele von Entitätsnamen aufgeführt.
 - `@location`
 - `@menu_item`
 - `@product`

1. Klicken Sie auf die Registerkarte **Entitäten**, um die vorhandenen Entitäten anzuzeigen.
2. Klicken Sie auf **Entität hinzufügen** und geben Sie den Namen der Entität als `location` hinter `@` ein. Klicken Sie auf **Entität erstellen**.
3. Geben Sie `address` als Wertnamen ein und wählen Sie **Synonyme** aus.
4. Fügen Sie `place` als Synonym hinzu und klicken Sie auf das Symbol ![](images/solution28-watson-chatbot-android/plus_icon.png). Wiederholen Sie diese Schritte mit den Synonymen `office`, `center`, `branch` usw. und klicken Sie auf **Wert hinzufügen**.
   ![](images/solution28-watson-chatbot-android/add_entity.png)
5. Klicken Sie auf **Schließen** ![](images/solution28-watson-chatbot-android/close_icon.png), um die Änderungen zu speichern.
6. Klicken Sie auf die Registerkarte **Systementitäten**, um die von IBM erstellten allgemeinen Entitäten zu überprüfen, die in allen Anwendungsfällen verwendet werden können.

   Systementitäten können verwendet werden, um einen breiten Wertebereich für die Objekttypen zu erkennen, die sie darstellen. Beispiel: Die Systementität `@sys-number` stimmt mit einem beliebigen numerischen Wert überein, einschließlich ganzer Zahlen, Dezimalzahlen oder sogar Zahlen, die als Wörter geschrieben werden.
   {:tip}
7. Ändern Sie für die Systementitäten '@sys-person' und '@sys-location' den **Status** von 'off' in `on`.

### Dialogablauf erstellen
{:#build_dialog}

Ein Dialog ist ein Verzweigungsdialogfluss, der definiert, wie Ihre Anwendung reagiert, wenn sie die definierten Absichten und Entitäten erkennt. Sie verwenden den Dialogbuilder im Tool, um Dialoge mit Benutzern zu erstellen und Antworten basierend auf den Absichten und Entitäten bereitzustellen, die Sie in ihrer Eingabe erkennen.

1. Klicken Sie auf die Registerkarte **Dialog**, um den vorhandenen Dialogablauf mit den Absichten und Entitäten anzuzeigen.
2. Klicken Sie auf **Knoten hinzufügen**, um dem Dialog einen neuen Knoten hinzuzufügen.
3. Geben Sie unter **Assistent erkennt Folgendes** den Wert `#Cancel_Policy` ein.
4. Geben Sie unter **Antwort** die Antwort `Diese Einrichtung ist online nicht verfügbar. Bitte besuchen Sie unsere nächstgelegene Filiale, um Ihre Police zu kündigen.` ein.
5. Klicken Sie auf ![](images/solution28-watson-chatbot-android/save_node.png), um den Knoten zu schließen und zu speichern.
6. Blättern Sie zum Knoten `#greeting`. Klicken Sie auf den Knoten, um die Details anzuzeigen.
   ![](images/solution28-watson-chatbot-android/build_dialog.png)
7. Klicken Sie auf das Symbol ![](images/solution28-watson-chatbot-android/add_condition.png), um **eine neue Bedingung hinzuzufügen**. Wählen Sie `oder` aus der Dropdown-Liste aus und geben Sie `#General_Greetings` als Absicht ein. Der Abschnitt **Antwort** zeigt die Antwort des Assistenten an, wenn er vom Benutzer begrüßt wird.
   ![](images/solution28-watson-chatbot-android/apply_condition.png)

   Eine Kontextvariable ist eine Variable, die Sie in einem Knoten definieren und für die Sie optional einen Standardwert angeben können. Mit anderen Knoten oder einer Anwendungslogik kann der Wert der Kontextvariable nachfolgend festgelegt oder geändert werden. Die Anwendung kann Informationen an den Dialog übergeben und der Dialog kann diese Informationen aktualisieren und an die Anwendung oder an einen nachfolgenden Knoten übergeben. Der Dialog verwendet dafür Kontextvariablen.
   {:tip}

8. Testen Sie den Dialogablauf über die Schaltfläche **Testen**.

## Skill mit dem Assistenten verknüpfen

Ein **Assistent** ist ein kognitiver Bot, den Sie für Ihre Geschäftsanforderungen anpassen über mehrere Kanäle hinweg implementieren können, um Ihre Kunden bei Bedarf entsprechend zu unterstützen. Sie passen den Assistenten an, indem Sie ihm die **Skills** hinzufügen, die er zum Erfüllen der Ziele Ihrer Kunden benötigt.

1. Wechseln Sie im {{site.data.keyword.conversationshort}}-Tool zu **Assistenten** und verwenden Sie die Option **Neu erstellen**.
   1. Setzen Sie den **Namen** auf **android-chatbot-assistant**.
   1. **Erstellen** die den Assistenten.
1. Verwenden Sie die Option **Dialog-Skill hinzufügen**, um den in den vorherigen Abschnitten erstellten Skill auszuwählen.
   1. Wählen Sie die Option **Vorhandenen Skill hinzufügen** aus.
   1. Wählen Sie **Ana** aus.
1. Wählen Sie das Aktionsmenü unter 'Assistent > **Einstellungen** > **API-Details** aus und notieren Sie sich die **Assistant-ID**. Auf diese müssen Sie von der mobile App aus (in der Datei `config.xml` der Android-App) verweisen.

## Android-App konfigurieren und ausführen
{:#configure_run_android_app}

Das Repository enthält Android-Anwendungscode mit den erforderlichen Gradle-Abhängigkeiten.

1. Führen Sie den folgenden Befehl aus, um das [GitHub-Repository](https://github.com/IBM-Cloud/chatbot-watson-android) zu klonen:
   ```bash
   git clone https://github.com/IBM-Cloud/chatbot-watson-android
   ```
   {: codeblock}

2. Starten Sie Android Studio. > Wählen Sie **Vorhandenes Android Studio-Projekt öffnen** aus und verweisen Sie auf den heruntergeladenen Code. Der **Gradle**-Builde wird automatisch ausgelöst und alle Abhängigkeiten werden automatisch herutergeladen.
3. Öffnen Sie die Datei `app/src/main/res/values/config.xml`, um die Platzhalter (`ASSISTANT-ID_HIER`) für Serviceberechtigungsnachweise anzuzeigen. Geben Sie die Serviceberechtigungsnachweise (die Sie zuvor gespeichert haben) in den entsprechenden Platzhaltern ein und speichern Sie die Datei.
   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <resources>
       <!--Serviceberechtigungsnachweis von Watson Assistant-->
       <!-- ERSETZEN SIE `ASSISTANT-ID_HIER` durch die die ID der zu verwendenden Assistant-Instanz -->
       <string name="assistant_id">ASSISTANT-ID_HIER</string>

       <!-- ERSETZEN SIE `ASSISTANT-API-SCHLÜSSEL_HIER` durch den API-Schlüssel des Watson Assistant-Service-->
       <string name="assistant_apikey">ASSISTANT-API-SCHLÜSSEL_HIER</string>

       <!-- ERSETZEN SIE `ASSISTANT-URL_HIER` durch die URL des Watson Assistant-Service-->
       <string name="assistant_url">ASSISTANT-URL_HIER</string>

       <!--Watson Speech To Text(STT) service credentials-->
       <!-- ERSETZEN SIE `STT-API-SCHLÜSSEL_HIER` durch den API-Schlüssel des Watson Speech to Text-Service-->
       <string name="STT_apikey">STT-API-SCHLÜSSEL_HIER</string>

       <!-- ERSETZEN SIE `STT-URL_HIER` durch die URL des Watson Speech to Text-Service-->
       <string name="STT_url">STT-URL_HIER</string>

       <!--Serviceberechtigungen des Watson Text to Speech-Service (TTS)-->
       <!-- ERSETZEN SIE `TTS-API-SCHLÜSSEL_HIER` durch den API-Schlüssel des Watson Text to Speech-Service-->
       <string name="TTS_apikey">TTS-API-SCHLÜSSEL_HIER</string>

       <!-- ERSETZEN SIE `TTS-URL_HIER` durch die URL des Watson Text to Speech-Service-->
       <string name="TTS_url">TTS-URL_HIER</string>
   </resources>
   ```
4. Erstellen Sie das Projekt und starten Sie die Anwendung auf einem realen Gerät oder mit einem Simulator.
   <p style="text-align: center; width:200">
   ![](images/solution28-watson-chatbot-android/android_watson_chatbot.png)![](images/solution28-watson-chatbot-android/android_chatbot.png)

    </p>
5. **Geben Sie Ihre Abfrage** in den unten angegebenen Bereich ein und klicken Sie auf das Pfeilsymbol, um die Abfrage an den {{site.data.keyword.conversationshort}}-Service zu senden.
6. Die Antwort wird an den {{site.data.keyword.texttospeechshort}}-Service übergeben und Sie sollten eine Stimme hören, die die Antwort vorliest.
7. Klicken Sie unten in der linken unteren Ecke der App auf das **Mikrofon**-Symbol, um etwas zu sagen, was in Text konvertiert wird und anschließend durch das Klicken auf das Pfeilsymbol an den {{site.data.keyword.conversationshort}}-Service gesendet werden kann.


## Ressourcen entfernen
{:removeresources}

1. Navigieren Sie zur [Ressourcenliste](https://{DomainName}/resources/).
1. Löschen Sie die von Ihnen erstellten Services:
   - {{site.data.keyword.conversationfull}}
   - {{site.data.keyword.speechtotextfull}}
   - {{site.data.keyword.texttospeechfull}}

## Zugehöriger Inhalt
{:related}

- [Entitäten, Synonyme und Systementitäten erstellen](https://{DomainName}/docs/services/assistant?topic=assistant-entities#creating-entities)
- [Kontextvariablen](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-runtime#dialog-runtime-context-variables)
- [Komplexen Dialog erstellen](https://{DomainName}/docs/services/assistant?topic=assistant-tutorial#tutorial)
- [Informationen mit Slots sammeln](https://{DomainName}/docs/services/assistant?topic=assistant-dialog-slots#dialog-slots)
- [Implementierungsoptionen](https://{DomainName}/docs/services/assistant?topic=assistant-deploy-integration-add#deploy-integration-add)
- [Skill verbessern](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs-intro)
