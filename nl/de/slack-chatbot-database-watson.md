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

# Datenbankgesteuerten Slackbot erstellen
{: #slack-chatbot-database-watson}

In diesem Lernprogramm erstellen Sie einen Slackbot, um Db2-Datenbankeinträge zu erstellen und nach Events und Konferenzen zu durchsuchen. Der Slackbot wird durch den {{site.data.keyword.conversationfull}}-Service unterstützt. Sie integrieren Slack und {{site.data.keyword.conversationfull}} mithilfe einer Assistentenintegration.

Die Slack-Integration dient als Kanal für Nachrichten zwischen Slack und {{site.data.keyword.conversationshort}}. Darin werden von einigen serverseitigen Dialogaktionen SQL-Abfragen für eine Db2-Datenbank ausgeführt. Der gesamte (nicht umfangreiche) Code wird in Node.js geschrieben.

## Ziele
{: #objectives}

* {{site.data.keyword.conversationfull}} durch eine Integration mit  Slack verbinden
* Node.js-Aktionen in {{site.data.keyword.openwhisk_short}} erstellen, bereitstellen und binden
* Auf eine Db2-Datenbank über {{site.data.keyword.openwhisk_short}} mithilfe von Node.js zugreifen

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
   * [{{site.data.keyword.conversationfull}}](https://{DomainName}/catalog/services/conversation)
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}} ](https://{DomainName}/catalog/services/db2-warehouse) oder [{{site.data.keyword.databases-for-postgresql}}](https://{DomainName}/catalog/services/databases-for-postgresql)


Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur
{: #architecture}

<p style="text-align: center;">

  ![Architektur](images/solution19/SlackbotArchitecture.png)
</p>

## Vorbereitende Schritte
{: #prereqs}

Zum Ausführen dieses Lernprogramms müssen die neueste Version der [{{site.data.keyword.Bluemix_notm}}-CLI](/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) und das {{site.data.keyword.openwhisk_short}}-[Plug-in installiert sein](/docs/cli?topic=cloud-cli-plug-ins).


## Service und Umgebung einrichten
In diesem Abschnitt richten Sie die erforderlichen Services ein und bereiten die Umgebung vor. Die meisten dieser Schritte können mithilfe von Scripts über die Befehlszeilenschnittstelle (Command Line Interface, CLI) ausgeführt werden. Die Scripts sind auf GitHub verfügbar.

1. Klonen Sie das [GitHub-Repository](https://github.com/IBM-Cloud/slack-chatbot-database-watson) und navigieren Sie zum geklonten Verzeichnis:
   ```bash
   git clone https://github.com/IBM-Cloud/slack-chatbot-database-watson
   cd slack-chatbot-database-watson
   ```
2. Wenn Sie nicht angemeldet sind, melden Sie sich mit `ibmcloud login` interaktiv an.
3. Geben Sie die Organisation und den Bereich als Ziel an, in dem der Datenbankservice erstellt werden soll:
   ```
   ibmcloud target --cf
   ```
4. Erstellen Sie eine {{site.data.keyword.dashdbshort}}-Instanz mit dem Namen **eventDB**:
   ```
   ibmcloud service create dashDB Entry eventDB
   ```
   {:codeblock}
   Sie können auch einen anderen als den **Entry**-Plan verwenden.
5. Um später von {{site.data.keyword.openwhisk_short}} aus auf den Datenbankservice zuzugreifen, benötigen Sie die entsprechende Autorisierung. Aus diesem Grund erstellen Sie Serviceberechtigungsnachweise mit dem Namen **slackbotkey**:   
   ```
   ibmcloud service key-create eventDB slackbotkey
   ```
   {:codeblock}
6. Erstellen Sie eine Instanz des {{site.data.keyword.conversationshort}}-Service. Verwenden Sie den Namen **eventConversation** und den kostenlosen Lite-Plan.
   ```
   ibmcloud service create conversation free eventConversation
   ```
   {:codeblock}
7. Als nächstes registrieren Sie Aktionen für {{site.data.keyword.openwhisk_short}} und binden Serviceberechtigungsnachweise an diese Aktion. Einige der Aktionen werden als Webaktionen aktiviert und durch einen geheimen Schlüssel vor unberechtigtem Aufrufen geschützt. Wählen Sie einen geheimen Schlüsse aus und übergeben Sie ihn als Parameter (ersetzen Sie dabei **YOURSECRET** durch den entsprechenden Schlüssel).

   Eine der Aktionen wird aufgerufen, um eine Tabelle in {{site.data.keyword.dashdbshort}} zu erstellen. Wenn Sie eine Aktion aus {{site.data.keyword.openwhisk_short}} verwenden, ist weder ein lokaler Db2-Treiber noch die browserbasierte Schnittstelle erforderlich, um die Tabelle manuell zu erstellen. Führen Sie die Registrierung und Konfiguration mit dem nachfolgenden Befehl aus, um die Datei **setup.sh** auszuführen, die alle erforderlichen Aktionen enthält. Wenn Ihr System keine Shell-Befehle unterstützt, kopieren Sie jede Zeile aus der Datei **setup.sh** und führen Sie sie einzeln aus.

   ```bash
   sh setup.sh YOURSECRET
   ```
   {:codeblock}   

   **Hinweis:** Das Script fügt standardmäßig einige Zeilen mit Beispieldaten ein. Sie können dies verhindern, indem Sie die folgende Zeile im Script auf Kommentar setzen: `#ibmcloud fn action invoke slackdemo/db2Setup -p mode "[\"sampledata\"]" -r`.
8. Extrahieren Sie die Namensbereichsinformationen für die bereitgestellten Aktionen:

   ```bash
   ibmcloud fn action list | grep eventInsert
   ```
   {:codeblock}   
   
   Notieren Sie den Teil vor **/slackdemo/eventInsert**. Dieser Teil enthält die codierte Organisation und den Bereich. Sie benötigten diese Angaben im nächsten Abschnitt.

## Laden Sie den Skill bzw. Arbeitsbereich
In diesem Teil des Lernprogramms laden Sie einen vordefinierten Arbeitsbereich oder Skill in den {{site.data.keyword.conversationshort}}-Service.
1. Öffnen Sie in der [{{site.data.keyword.Bluemix_short}}-Ressourcenliste](https://{DomainName}/resources) die Übersicht über Ihre Services. Suchen Sie die Instanz des {{site.data.keyword.conversationshort}}-Service, die im vorherigen Abschnitt erstellt wurde. Klicken Sie auf den zugehörigen Eintrag und danach auf den Servicealias, um die Servicedetails zu öffnen.
2. Klicken Sie auf **Tool starten**, um das Tool {{site.data.keyword.conversationshort}} aufzurufen.
3. Wechseln Sie zu **Skills** und klicken Sie anschließend auf **Skill erstellen** und danach auf **Skill importieren**.
4. Klicken Sie im Dialogmodul auf **JSON-Datei auswählen** und wählen Sie die Datei **assistant-skill.json** im lokalen Verzeichnis aus. Behalten Sie für die Importoption die Einstellung **Alles (Absichten, Entitäten und Dialogmodul)** bei und klicken Sie auf **Importieren**. Daraufhin wird ein neuer Skill mit dem Namen **TutorialSlackbot** erstellt.
5. Klicken Sie auf **Dialogmodul**, um die Dialogmodulknoten anzuzeigen. Sie können die Knoten erweitern, damit eine Struktur wie die unten dargestellte angezeigt wird.

   Das Dialogmodul enthält Knoten für Anfragen nach Hilfe oder einfach für 'Danke'. Der Knoten **newEvent** und das zugehörige untergeordnete Element sammeln die erforderlichen Eingaben und rufen eine Aktion zum Einfügen eines neuen Ereignisses in Db2 auf.

   Der Knoten **query events** gibt an, ob Ereignisse nach ID oder nach Datum durchsucht werden. Der eigentliche Suchvorgang und das Erfassen der erforderlichen Daten werden von den untergeordneten Knoten **query events by shortname** und **query event by dates** ausgeführt.

   Der Knoten **credential_node** konfiguriert den geheimen Schlüssel für die Dialogmodulaktionen und die Informationen zur Cloud Foundry-Organisation. Die Organisationsinformationen werden zum Aufrufen der Aktionen benötigt.

  Die Details werden später erläutert, nachdem die Konfiguration abgeschlossen ist.
  ![](images/solution19/SlackBot_Dialog.png)   
6. Klicken Sie auf den Dialogmodulknoten **credential_node** und öffnen Sie den JSON-Editor durch Klicken auf das Menüsymbol rechts neben **Dann antworten mit**.

   ![](images/solution19/SlackBot_DialogCredentials.png)   

   Ersetzen Sie **org_space** durch die codierten Organisations- und Bereichsinformationen, die Sie zuvor abgerufen haben. Ersetzen Sie jedes Zeichen '@' durch '%40'. Ersetzen Sie anschließend **YOURSECRET** durch Ihren geheimen Schlüssel. Schließen Sie den JSON-Editor durch erneutes Klicken auf das Symbol.

## Assistent erstellen und in Slack integrieren

Sie erstellen nun einen Assistenten, der dem oben angegebenen Skill zugeordnet ist, und integrieren den Assistenten in Slack. 
1. Klicken Sie oben links auf **Skills** und wählen Sie danach **Assistenten** aus. Klicken Sie anschließend auf **Assistent erstellen**.
2. Geben Sie im Dialogfenster den Namen **LernprogrammAssistent** an und klicken Sie auf **Assistent erstellen**. Wählen Sie in der nächsten Anzeige **Dialogskill hinzufügen** aus. Wählen Sie danach **Vorhandenen Skill hinzufügen** aus, wählen Sie **TutorialSlackbot** in der Liste aus und fügen Sie diesen Skill hinzu.
3. Klicken Sie nach dem Hinzufügen des Skills auf **Integration hinzufügen** und wählen Sie in der Liste **Verwaltete Integrationen** den Eintrag **Slack** aus.
4. Folgen Sie den Anweisungen, um Ihren Chat-Bot in Slack zu integrieren.

## Slackbot testen und erkunden
Öffnen Sie Ihren Slack-Arbeitsbereich, um einen Testlauf mit dem Chat-Bot durchzuführen. Beginnen Sie einen direkten Chat mit dem Bot.

1. Geben Sie **Hilfe** in das Nachrichtenformular ein. Der Bot müsste daraufhin entsprechende Anweisungen anzeigen.
2. Geben Sie nun **neues Ereignis** ein, um mit dem Erfassen von Daten für einen neuen Ereignisdatensatz zu beginnen. Die erforderlichen Eingaben werden in {{site.data.keyword.conversationshort}}-Slots gesammelt.
3. Das erste Element ist die Ereignis-ID oder der Ereignisname. Für dieses Element sind Anführungszeichen erforderlich. Sie ermöglichen die Eingabe komplexer Namen. Geben Sie **"Meeting: IBM Cloud"** als Ereignisnamen ein. Der Ereignisname ist als eine musterbasierte Entität **eventName** definiert. Am Anfang und am Ende dieser Entität werden verschiedene Arten von doppelten Anführungszeichen unterstützt.
4. Das nächste Element ist die Ereignisposition. Die Eingabe basiert auf der [Systementität **sys-location**](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entity-details). Es können nur Städte verwendet werden, die von {{site.data.keyword.conversationshort}} erkannt werden. Geben Sie die Stadt **Friedrichshafen** ein.
5. Im nächsten Schritt müssen Kontaktinformationen (z. B. eine E-Mail-Adresse oder der URI einer Website) angegeben werden. Beginnen Sie mit **https://www.ibm.com/events**. Für dieses Feld verwenden Sie eine musterbasierte Entität.
6. Als nächstes müssen Anfangs- und Endzeitpunkt (Datum und Uhrzeit) erfasst werden. Hierfür werden die Elemente **sys-date** und **sys-time** verwendet, die verschiedene Eingabeformate zulassen. Verwenden Sie **Nächsten Donnerstag** als Startdatum und **18 Uhr** als Uhrzeit. Verwenden Sie das exakte Datum für den nächsten Donnerstag (z. B. **2019-05-09**) und **22:00** als Enddatum und -zeit.
7. Nachdem alle Daten erfasst sind, wird eine Zusammenfassung ausgegeben und eine Serveraktion (implementiert als {{site.data.keyword.openwhisk_short}}-Aktion) aufgerufen, um einen neuen Datensatz in Db2 einzufügen. Anschließend ruft das Dialogmodul einen untergeordneten Knoten auf, um die Verarbeitungsumgebung zu bereinigen und die Kontextvariablen zu entfernen. Der gesamte Eingabeprozess kann durch die Eingabe von **cancel**, **exit** oder eines ähnlichen Befehls jederzeit abgebrochen werden. In diesem Fall wird die Benutzerauswahl bestätigt und die Umgebung bereinigt. ![](images/solution19/SlackSampleChat.png)   

Sobald Beispieldaten vorhanden sind, kann ein Suchvorgang ausgeführt werden.
1. Geben Sie **Ereignisinformationen anzeigen** ein. Daraufhin wird angefragt, ob nach ID oder Datum gesucht werden soll. Geben Sie einen **Namen** und für die nächste Frage **"Think 2019"** ein. Daraufhin müsste der Chat-Bot Informationen zu dem betreffenden Event anzeigen. Das Dialogmodul kann aus vielen verfügbaren Antworten auswählen.
2. Mit {{site.data.keyword.conversationshort}} als Back-End können komplexere Ausdrücke eingegeben und dadurch Teile des Dialogmoduls übersprungen werden. Verwenden Sie die Eingabe **show event by the name "Think 2019"**. Der Chat-Bot gibt sofort den Ereignisdatensatz zurück.
3. Als nächstes führen Sie eine Suche nach Datum durch. Dabei wird durch zwei Datumsangaben ein Zeitraum definiert, in dem das Startdatum für das Event liegen muss. Geben Sie **search conference by date in February 2019** als Eingabe an. Als Ergebnis müsste wieder das Event **Think 2019** zurückgegeben werden. Die Entität **February** wird in die beiden Datumsangaben 'February 1st' und 'February 28th' umgesetzt, die den Anfang und das Ende des Datumsbereichs angeben. [Wenn die Jahreszahl 2019 fehlt, wird ein zukünftiger Monat Februar verwendet](https://{DomainName}/docs/services/assistant?topic=assistant-system-entities#system-entities-sys-date-time). 

Nach weiteren Suchvorgängen und neuen Einträgen für Events können Sie erneut das Chatverlaufsprotokoll aufrufen, um das Dialogmodul zu optimieren. Folgen Sie den Anweisungen in der [{{site.data.keyword.conversationshort}}-Dokumentation zum Thema **Verständnis verbessern**](https://{DomainName}/docs/services/assistant?topic=assistant-logs-intro#logs_intro).


## Ressourcen entfernen
{:removeresources}

Wenn Sie das Bereinigungsscript im Hauptverzeichnis ausführen, werden die Ereignistabelle aus {{site.data.keyword.dashdbshort}} und die Aktionen aus {{site.data.keyword.openwhisk_short}} entfernt. Dies kann hilfreich sein, wenn Sie den Code modifizieren und erweitern möchten. Das Bereinigungsscript nimmt keine Änderungen am {{site.data.keyword.conversationshort}}-Arbeitsbereich oder -Skill vor.   
```bash
sh cleanup.sh
```
{:codeblock}

Öffnen Sie in der [{{site.data.keyword.Bluemix_short}}-Ressourcenliste](https://{DomainName}/resources) die Übersicht über Ihre Services. Suchen Sie die Instanz des {{site.data.keyword.conversationshort}}-Service und löschen Sie sie.

## Lernprogramm erweitern
Sie möchten dieses Lernprogramm ergänzen oder ändern? Hier einige Vorschläge:
1. Fügen Sie Suchfunktionen hinzu, z. B. die Platzhaltersuche oder die Suche nach der Dauer von Events ("give me all events longer than 8 hours").
2. Verwenden Sie {{site.data.keyword.databases-for-postgresql}} anstelle von {{site.data.keyword.dashdbshort}}. Das [GitHub-Repository für dieses Slackbot-Lernprogramm](https://github.com/IBM-Cloud/slack-chatbot-database-watson) enthält bereits Code, der {{site.data.keyword.databases-for-postgresql}} unterstützt.
3. Fügen Sie einen Wetterdienst hinzu und rufen Sie Vorhersagedaten für das Datum und den Standort des Events ab.
4. Exportieren Sie Eventdaten als iCalendar-Datei (**.ics**).
5. Verbinden Sie den Chat-Bot mit Facebook Messenger, indem Sie eine weitere Integration hinzufügen.
6. Fügen Sie interaktive Elemente (z. B. Schaltflächen) zur Ausgabe hinzu.      


## Zugehörige Inhalte
{:related}

Hier finden Sie Links mit weiteren Informationen zu den in diesem Lernprogramm behandelten Themen.

Auf Chat-Bots bezogene Blogbeiträge:
* [Chatbots: Some tricks with slots in IBM Watson Conversation](https://www.ibm.com/blogs/bluemix/2018/02/chatbots-some-tricks-with-slots-in-ibm-watson-conversation/)
* [Lively chatbots: Best Practices](https://www.ibm.com/blogs/bluemix/2017/07/lively-chatbots-best-practices/)
* [Building chatbots: more tips and tricks](https://www.ibm.com/blogs/bluemix/2017/06/building-chatbots-tips-tricks/)

Dokumentation und SDKs:
* GitHub-Repository mit [Tipps und Tricks zur Handhabung von Variablen in IBM Watson Conversation](https://github.com/IBM-Cloud/watson-conversation-variables)
* [{{site.data.keyword.openwhisk_short}}-Dokumentation](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Dokumentation: [IBM Knowledge Center für {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [Kostenlose Db2 Developer Community Edition](https://www.ibm.com/us-en/marketplace/ibm-db2-direct-and-developer-editions) für Entwickler
* Dokumentation: [API-Beschreibung für den Node.js-Treiber für ibm_db](https://github.com/ibmdb/node-ibm_db)
* [{{site.data.keyword.cloudantfull}}-Dokumentation](https://{DomainName}/docs/services/Cloudant?topic=cloudant-overview#overview)
