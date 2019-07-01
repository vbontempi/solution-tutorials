---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Serverunabhängige Komponenten und Cloud Foundry für Datenabruf und -analyse kombinieren
{: #serverless-github-traffic-analytics}
In diesem Lernprogramm erstellen Sie eine Anwendung zum automatischen Erfassen von GitHub-Datenverkehrsstatistiken für Repositorys und legen die Grundlage für die Datenverkehrsanalyse. GitHub macht nur Datenverkehrsdaten für die letzten 14 Tage zugänglich. Wenn Sie Statistiken über einen längeren Zeitraum analysieren möchten, müssen Sie die entsprechenden Daten selbst herunterladen und speichern. In diesem Lernprogramm stellen Sie eine serverunabhängige Aktion zum Abrufen der Datenverkehrsdaten und zum Speichern dieser Daten in einer SQL-Datenbank bereit. Außerdem wird eine Cloud Foundry-App verwendet, um Repositorys zu verwalten und den Zugriff auf die Statistiken für die Datenanalyse zu ermöglichen. Die im vorliegenden Lernprogramm dargestellte App und serverunabhängige Aktion stellen eine Multi-Tenant-fähige Lösung bereit (die ursprünglichen Funktionen unterstützen den Single-Tenant-Modus).

![](images/solution24-github-traffic-analytics/Architecture.png)

## Ziele

* Eine Python-Datenbank-App mit Multi-Tenant-Unterstützung und sicherem Zugriff bereitstellen
* Die App-ID als OpenID Connect-Authentifizierungsprovider integrieren
* Automatisierte und serverunabhängige Erfassung von GitHub-Datenverkehrsstatistiken einrichten
* {{site.data.keyword.dynamdashbemb_short}} für die grafische Datenverkehrsanalyse integrieren

## Produkte
Im vorliegenden Lernprogramm werden die folgenden Produkte verwendet:
   * [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
   * [{{site.data.keyword.dashdblong}}](https://{DomainName}/catalog/services/db2-warehouse)
   * [{{site.data.keyword.appid_long}}](https://{DomainName}/catalog/services/app-id)
   * [{{site.data.keyword.dynamdashbemb_notm}}](https://{DomainName}/catalog/services/ibm-cognos-dashboard-embedded)

## Vorbereitende Schritte
{: #prereqs}

Zum Ausführen dieses Lernprogramms müssen die neueste Version der [IBM Cloud-CLI](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview) und das {{site.data.keyword.openwhisk_short}} [-Plug-in installiert sein](/docs/cli?topic=cloud-cli-install-ibmcloud-cli).

## Service- und Umgebungskonfiguration (Shell)
In diesem Abschnitt richten Sie die erforderlichen Services ein und bereiten die Umgebung vor. Alle erforderlichen Schritte können in der Shellumgebung ausgeführt werden.

1. Klonen Sie das [GitHub-Repository](https://github.com/IBM-Cloud/github-traffic-stats) und navigieren Sie im geklonten Verzeichnis zum Unterverzeichnis **backend**:
   ```bash
   git clone https://github.com/IBM-Cloud/github-traffic-stats
   cd github-traffic-stats/backend
   ```
   {:codeblock}

2. Melden Sie sich mit `ibmcloud login` interaktiv bei {{site.data.keyword.Bluemix_short}} an. Sie können die Details überprüfen, indem Sie den Befehl `ibmcloud target` ausführen. Sie müssen eine Organisation und einen Bereich eingerichtet haben.

3. Erstellen Sie eine {{site.data.keyword.dashdbshort}}-Instanz mit dem **Entry**-Plan und ordnen Sie der Instanz den Namen **ghstatsDB** zu:
   ```
   ibmcloud service create dashDB Entry ghstatsDB
   ```
   {:codeblock}

4. Um später von {{site.data.keyword.openwhisk_short}} aus auf den Datenbankservice zuzugreifen, benötigen Sie die entsprechende Autorisierung. Aus diesem Grund erstellen Sie Serviceberechtigungsnachweise mit dem Namen **ghstatskey**:   
   ```
   ibmcloud service key-create ghstatsDB ghstatskey
   ```
   {:codeblock}

5. Erstellen Sie eine Instanz des {{site.data.keyword.appid_short}}-Service. Verwenden Sie den Namen **ghstatsAppID** und den Plan mit gestaffelter Preisstufe (**graduated tier**).
   ```
   ibmcloud resource service-instance-create ghstatsAppID appid graduated-tier us-south
   ```
   {:codeblock}
   Erstellen Sie anschließend einen Alias für die neue Serviceinstanz im Cloud Foundry-Bereich. Ersetzen Sie **YOURSPACE** durch den Bereich, in dem die Bereitstellung erfolgen soll.
   ```
   ibmcloud resource service-alias-create ghstatsAppID --instance-name ghstatsAppID -s YOURSPACE
   ```
  {:codeblock}

6. Erstellen Sie eine Instanz des {{site.data.keyword.dynamdashbemb_short}}-Service unter Verwendung des **Lite**-Plans:
   ```
   ibmcloud resource service-instance-create ghstatsDDE dynamic-dashboard-embedded lite us-south
   ```
   {:codeblock}
   Erstellen Sie auch für diese neue Serviceinstanz einen Alias und ersetzen Sie **YOURSPACE**:
   ```
   ibmcloud resource service-alias-create ghstatsDDE --instance-name ghstatsDDE -s YOURSPACE
   ```
  {:codeblock}
7. Übertragen Sie die Anwendung vom Verzeichnis **backend** aus in IBM Cloud. Der Befehl verwendet eine zufällige Route für Ihre Anwendung.
   ```bash
   ibmcloud cf push
   ```
   {:codeblock}
   Warten Sie, bis die Bereitstellung abgeschlossen ist. Die Anwendungsdateien werden hochgeladen, die Laufzeitumgebung wird erstellt und die Services werden an die Anwendung gebunden. Die Serviceinformationen werden aus der Datei `manifest.yml` übernommen. Sie müssen diese Datei aktualisieren, wenn Sie andere Servicenamen verwendet haben. Sobald der Prozess erfolgreich abgeschlossen ist, wird der URI der Anwendung angezeigt.

   Im obigen Befehl wird eine zufällige und eindeutige Route für die Anwendung verwendet. Wenn Sie selbst eine Route auswählen möchten, fügen Sie die Route als zusätzlichen Parameter zum Befehl hinzu (z. B. `ibmcloud cf push your-app-name`). Alternativ können Sie in der Datei `manifest.yml` einen anderen Wert für **name** angeben und den Wert für **random-route** von **true** in **false** ändern.
   {:tip}

## App-ID und GitHub-Konfiguration (Browser)
Die folgenden Schritte werden in Ihrem Internet-Browser ausgeführt. Zuerst konfigurieren Sie {{site.data.keyword.appid_short}} für die Verwendung von Cloud Directory und der Python-App. Anschließend erstellen Sie ein GitHub-Zugriffstoken. Das Token wird für die bereitgestellte Funktion zum Abrufen der Datenverkehrsdaten benötigt.

1. Öffnen Sie in der [{{site.data.keyword.Bluemix_short}}-Ressourcenliste](https://{DomainName}/resources) die Übersicht über Ihre Services. Lokalisieren Sie die Instanz des {{site.data.keyword.appid_short}}-Service im Abschnitt **Services**. Klicken Sie auf den zugehörigen Eintrag, um die Details zu öffnen.
2. Klicken Sie im Service-Dashboard auf **Verwalten** unter **Identitätsprovider** im Menü auf der linken Seite. Eine Liste der verfügbaren Identitätsprovider wie Facebook, Google, SAML 2.0 Federation und Cloud Directory wird angezeigt. Setzen Sie Cloud Directory auf **Ein** und alle übrigen Provider auf **Aus**.
   
   Möglicherweise möchten Sie die Mehrfaktorauthentifizierung ([Multi-Factor Authentication, MFA)](https://{DomainName}/docs/services/appid?topic=appid-cd-mfa#cd-mfa) und erweiterte Kennwortregeln konfigurieren. Diese Komponenten werden im vorliegenden Lernprogramm nicht behandelt.{:tip}

3. Am Ende der Seite wird die Liste der Umleitungs-URLs angezeigt. Geben Sie die **URL** Ihrer Anwendung und den Weiterleitungs-URI ein. Beispiel: `https://github-traffic-stats-random-word.mybluemix.net/redirect_uri`.

   Die Umleitungs-URL zum lokalen Testen der App lautet `http://0.0.0.0:5000/redirect_uri`. Sie können mehrere Umleitungs-URLs konfigurieren.
   {:tip}

   ![](images/solution24-github-traffic-analytics/ManageIdentityProviders.png)
4. Klicken Sie im Menü auf der linken Seite auf **Benutzer**. Die Liste der Benutzer in Cloud Directory wird angezeigt. Klicken Sie auf die Schaltfläche **Benutzer hinzufügen**, um Ihren Benutzernamen als ersten Benutzer hinzuzufügen. Damit ist die Konfiguration des {{site.data.keyword.appid_short}}-Service abgeschlossen.
5. Rufen Sie im Browser [Github.com](https://github.com/settings/tokens) auf und navigieren Sie zu **Einstellungen -> Entwicklereinstellungen -> Persönliche Zugriffstoken**. Klicken Sie auf die Schaltfläche **Neues Token generieren**. Geben Sie **Lernprogramm für GHStats** als **Tokenbeschreibung** ein. Aktivieren Sie anschließend **public_repo** in der Kategorie **repo** und **read:org** unter **admin:org**. Klicken Sie nun unten auf dieser Seite auf **Token generieren**. Das neue Zugriffstoken wird auf der nächsten Seite angezeigt. Sie benötigen das Token beim nachfolgenden Einrichten der Anwendungskonfiguration.![](images/solution24-github-traffic-analytics/GithubAccessToken.png)


## Python-App konfigurieren und testen
Im Anschluss an die Vorbereitung konfigurieren und testen Sie die App. Die App wurde in Python mit dem weit verbreiteten Microframework [Flask](http://flask.pocoo.org/) geschrieben. Sie können Repositorys in der Statistikerfassung hinzufügen und entfernen. Die Datenverkehrsdaten können in einer tabellarischen Ansicht aufgerufen werden.

1. Öffnen Sie den URI der bereitgestellten App in einem Browser. Eine Begrüßungsseite müsste angezeigt werden.
   ![](images/solution24-github-traffic-analytics/WelcomeScreen.png)

2. Fügen Sie im Browser `/admin/initialize-app` zum URI hinzu und rufen Sie die Seite auf. Sie wird zum Initialisieren der Anwendung und der zugehörigen Daten verwendet. Klicken Sie auf die Schaltfläche **Initialisierung starten**. Daraufhin wird eine kennwortgeschützte Konfigurationsseite geöffnet. Die E-Mail-Adresse, mit der Sie sich anmelden, wird als Identifikation für den Systemadministrator verwendet. Verwenden Sie die E-Mail-Adresse und das Kennwort, die Sie zuvor konfiguriert haben.

3. Geben Sie auf der Konfigurationsseite einen Namen ein (er wird als Begrüßung verwendet), sowie den GitHub-Benutzernamen und das Zugriffstoken, das Sie zuvor generiert haben. Klicken Sie auf **Initialisieren**. Daraufhin werden die Datenbanktabellen erstellt und einige Konfigurationswerte eingefügt. Abschließend werden Datenbankdatensätze für den Systemadministrator und einen Tenant erstellt.
   ![](images/solution24-github-traffic-analytics/InitializeApp.png)

4. Nachdem der Vorgang abgeschlossen ist, wird die Liste der verwalteten Repositorys angezeigt. Sie können nun Repositorys hinzufügen, indem Sie den Namen des GitHub-Kontos oder der Organisation und den Namen des Repositorys angeben. Klicken Sie nach dem Eingeben der Daten auf **Repository hinzufügen**. Das Repository mit einer neu zugeordneten Kennung müsste in der Tabelle angezeigt werden. Sie können Repositorys aus dem System entfernen, indem Sie die zugehörigen IDs eingeben und auf **Repository löschen** klicken.
![](images/solution24-github-traffic-analytics/RepositoryList.png)

## Cloudfunktion und Auslöser bereitstellen
Wenn die Management-App eingerichtet ist, stellen Sie eine Aktion, einen Auslöser und eine Regel zum Verbinden beider Objekte in {{site.data.keyword.openwhisk_short}} bereit. Diese Objekte dienen zum automatischen Erfassen der GitHub-Datenverkehrsdaten gemäß dem angegebenen Zeitplan. Die Aktion stellt die Verbindung zur Datenbank her, geht nacheinander alle Tenants und die zugehörigen Repositorys durch und ruft die Anzeige- und Klondaten für jedes Repository ab. Diese Statistikdaten werden in der Datenbank zusammengeführt.

1. Wechseln Sie in das Verzeichnis **functions**:
   ```bash
   cd ../functions
   ```
   {:codeblock}   
2. Erstellen Sie eine neue  Aktion **collectStats**. Sie verwendet eine [Python 3-Umgebung](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_reference#openwhisk_ref_python_environments), die bereits den erforderlichen Datenbanktreiber enthält. Der Quellcode für die Aktion wird in der Datei `ghstats.zip` bereitgestellt.
   ```bash
   ibmcloud fn action create collectStats --kind python-jessie:3 ghstats.zip
   ```
   {:codeblock}   

   Wenn Sie den Quellcode für die Aktion modifizieren (`__main__.py`), können Sie das ZIP-Archiv mit dem Befehl `zip -r ghstats.zip  __main__.py github.py` erneut packen. Details hierzu finden Sie in der Datei `setup.sh`.
   {:tip}
3. Binden Sie die Aktion an den Datenbankservice. Verwenden Sie die Instanz und den Serviceschlüssel, die Sie beim Einrichten der Umgebung erstellt haben.
   ```bash
   ibmcloud fn service bind dashDB collectStats --instance ghstatsDB --keyname ghstatskey
   ```
   {:codeblock}   
4. Erstellen Sie einen Auslöser auf der Basis des [Alarmpakets](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_catalog_alarm#openwhisk_catalog_alarm). Das Paket unterstützt verschiedene Methoden zum Angeben des Alarms. Verwenden Sie den an [cron](https://en.wikipedia.org/wiki/Cron) angelehnten Stil. Vom 21. April bis zum 21. Dezember wird der Auslöser täglich um 6:00 Uhr UTC ausgelöst. Stellen Sie sicher, dass ein Startdatum angegeben wird, das in der Zukunft liegt.
   ```bash
   ibmcloud fn trigger create myDaily --feed /whisk.system/alarms/alarm \
              --param cron "0 6 * * *" --param startDate "2018-04-21T00:00:00.000Z"\
              --param stopDate "2018-12-31T00:00:00.000Z"
   ```
  {:codeblock}   

  Sie können die tägliche Ausführung des Auslösers in eine wöchentliche Ausführung ändern, indem Sie `"0 6 * * 0"` anwenden. Dies bedeutet, dass der Auslöser jeden Sonntag um 6:00 Uhr aktiviert wird.
  {:tip}
5. Abschließend erstellen Sie eine Regel **myStatsRule**, die den Auslöser **myDaily** mit der Aktion **collectStats** verknüpft. Der Auslöser sorgt nun dafür, dass die Aktion gemäß dem Zeitplan ausgelöst wird, der im vorherigen Schritt angegeben wurde.
   ```bash
   ibmcloud fn rule create myStatsRule myDaily collectStats
   ```
   {:codeblock}   
6. Rufen Sie die Aktion für einen ersten Testlauf auf. Der Rückgabewert für **repoCount** müsste die Anzahl der Repositorys angeben, die Sie zuvor konfiguriert haben.
   ```bash
   ibmcloud fn action invoke collectStats  -r
   ```
   {:codeblock}   
   Die Ausgabe sieht so aus:
   ```
   {
       "repoCount": 18
   }
   ```
7. Sie können nun in Ihrem Browserfenster mit der App-Seite den Repository-Datenverkehr anzeigen. Standardmäßig werden 10 Einträge angezeigt. Sie können stattdessen andere Werte angeben. Darüber hinaus können Sie die Tabellenspalten sortieren oder mithilfe des Suchfelds nach bestimmten Repositorys filtern. Sie können ein Datum und einen Organisationsnamen eingeben und anschließend nach dem Anzeigezähler sortieren, um die höchsten Zählerwerte für einen bestimmten Tag aufzulisten.![](images/solution24-github-traffic-analytics/RepositoryTraffic.png)

## Schlussbemerkungen
Im vorliegenden Lernprogramm haben Sie eine serverunabhängige Aktion mit einem zugehörigen Auslöser und einer zugehörigen Regel bereitgestellt. Sie ermöglichen es, Datenverkehrsdaten für GitHub-Repositorys automatisch abzurufen. Informationen zu diesen Repositorys (einschließlich des tenantspezifischen Zugriffstokens) werden in einer SQL-Datenbank ({{site.data.keyword.dashdbshort}}) gespeichert. Diese Datenbank wird von der Cloud Foundry-App verwendet, um Benutzer und Repositorys zu verwalten und Datenverkehrsstatistiken im App-Portal anzuzeigen. Für Benutzer werden die Datenverkehrsstatistiken in durchsuchbaren Tabellen angezeigt oder in einem integrierten Dashboard ({{site.data.keyword.dynamdashbemb_short}}-Service, siehe nachfolgende Abbildung) grafisch dargestellt. Die Liste der Repositorys und die Datenverkehrsdaten können außerdem als CSV-Dateien heruntergeladen werden.

Die Cloud Foundry-App verwaltet den Zugriff über einen OpenID Connect-Client, der eine Verbindung zu {{site.data.keyword.appid_short}} herstellt.
![](images/solution24-github-traffic-analytics/EmbeddedDashboard.png)

## Bereinigung
Zum Bereinigen der Ressourcen, die für dieses Lernprogramm verwendet wurden, können Sie die zugehörigen Services sowie die App, den Auslöser und die Regel in der umgekehrten Reihenfolge ihrer Erstellung löschen.

1. Löschen Sie die {{site.data.keyword.openwhisk_short}}-Regel, den Auslöser und die Aktion:
   ```bash
   ibmcloud fn rule delete myStatsRule
   ibmcloud fn trigger delete myDaily
   ibmcloud fn action delete collectStats
   ```
   {:codeblock}   
2. Löschen Sie die Python-App und die zugehörigen Services:
   ```bash
   ibmcloud resource service-instance-delete ghstatsAppID
   ibmcloud resource service-instance-delete ghstatsDDE
   ibmcloud service delete ghstatsDB
   ibmcloud cf delete github-traffic-stats
   ```
   {:codeblock}   


## Lernprogramm erweitern
Sie möchten dieses Lernprogramm ergänzen oder ändern? Hier einige Vorschläge:
* Erweitern Sie die App durch Multi-Tenant-Unterstützung.
* Integrieren Sie ein Diagramm für die Daten.
* Verwenden Sie Identitätsprovider für soziale Netze.
* Fügen Sie auf der Statiskseite ein Datumsauswahlfeld zum Filtern der angezeigten Daten hinzu.
* Verwenden Sie eine angepasste Anmeldeseite für {{site.data.keyword.appid_short}}.
* Erkunden Sie die Beziehungen für soziale Codes zwischen Entwicklern mithilfe von [{{site.data.keyword.DRA_short}}](https://{DomainName}/catalog/services/devops-insights).

## Zugehörige Inhalte
Hier finden Sie Links mit weiteren Informationen zu den in diesem Lernprogramm behandelten Themen.

Dokumentation und SDKs:
* [{{site.data.keyword.openwhisk_short}}-Dokumentation](https://{DomainName}/docs/openwhisk?topic=cloud-functions-openwhisk_about#about-cloud-functions)
* Dokumentation: [IBM Knowledge Center für {{site.data.keyword.dashdbshort}}](https://www.ibm.com/support/knowledgecenter/en/SS6NHC/com.ibm.swg.im.dashdb.kc.doc/welcome.html)
* [{{site.data.keyword.appid_short}}-Dokumentation](https://{DomainName}/docs/services/appid?topic=appid-gettingstarted#gettingstarted)
* [Python-Laufzeit in IBM Cloud](https://{DomainName}/docs/runtimes/python?topic=Python-python_runtime#python_runtime)
