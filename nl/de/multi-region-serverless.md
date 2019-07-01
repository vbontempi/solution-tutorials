---
copyright:
  years: 2018, 2019
lastupdated: "2019-03-07"
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

# Serverunabhängige Apps in mehreren Regionen bereitstellen
{: #multi-region-serverless}

In diesem Lernprogramm erfahren Sie, wie Sie IBM Cloud Internet Services und {{site.data.keyword.openwhisk_short}} konfigurieren können, um serverunabhängige Apps in mehreren Regionen bereitzustellen.

Serverunabhängige IT-Plattformen bieten Entwicklern eine Möglichkeit zur schnellen Erstellung von APIs ohne Server. {{site.data.keyword.openwhisk}} unterstützt die automatische REST-API-Generierung für Aktionen, die Umwandlung von Aktionen in HTTP-Endpunkte und die Möglichkeit zum Aktivieren der sicheren API-Authentifizierung. Diese Funktionalität ist hilfreich zum Bereitstellen von APIs für externe Nutzer und zum Erstellen von Mikroservices-Anwendungen.

{{site.data.keyword.openwhisk_short}} ist an mehreren {{site.data.keyword.cloud_notm}}-Standorten verfügbar. Um die Ausfallsicherheit zu erhöhen und die Netzlatenz zu verringern, können Anwendungen das zugehörige Back-End an mehreren Standorten bereitstellen. Anschließend können Entwickler mit IBM Cloud Internet Services (CIS) einen einzigen Eingangspunkt bereitstellen, der für die Verteilung des Datenverkehrs auf das nächstgelegene betriebsbereite Back-End verantwortlich ist.

## Ziele
{: #objectives}

* {{site.data.keyword.openwhisk_short}}-Aktionen bereitstellen
* Aktionen über {{site.data.keyword.APIM}} mit einer angepassten Domäne verfügbar machen
* Datenverkehr mit Cloud Internet Services auf mehrere Standorte verteilen

## Verwendete Services
{: #services}

Im vorliegenden Lernprogramm werden die folgenden Laufzeiten und Services verwendet:
* [{{site.data.keyword.openwhisk_short}}](https://{DomainName}/openwhisk/)
* [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)
* IBM Cloud [Internet Services](https://{DomainName}/catalog/services/internet-svcs)

Für dieses Lernprogramm können Kosten anfallen. Mit dem [Preisrechner](https://{DomainName}/pricing/) können Sie einen Kostenvoranschlag für Ihre geplante Nutzung erstellen.

## Architektur
{: #architecture}

Das Lernprogramm bezieht sich auf ein öffentliche Webanwendung mit einem Back-End, die mit {{site.data.keyword.openwhisk_short}} implementiert wurde. Um die Netzlatenz zu verringern und Ausfallzeiten zu vermeiden, wird die Anwendung an mehreren Standorten bereitgestellt. Im Lernprogramm werden zwei Standorte konfiguriert.

<p style="text-align: center;">

  ![Architektur](images/solution44-multi-region-serverless/Architecture.png)
</p>

1. Benutzer greifen auf die Anwendung zu. Die Anforderung wird über Internet Services ausgeführt.
2. Internet Services leitet die Benutzer zum nächstgelegenen betriebsbereiten API-Back-End um.
3. {{site.data.keyword.cloudcerts_short}} stellt die API mit dem SSL-Zertifikat bereit. Der Datenverkehr wird durchgängig verschlüsselt.
4. Die API wird mit {{site.data.keyword.openwhisk_short}} bereitgestellt.

## Vorbereitende Schritte
{: #prereqs}

1. Cloud Internet Services setzt voraus, dass Sie über eine eigene, angepasste Domäne verfügen und den DNS für diese Domäne so konfigurieren können, dass er auf Cloud Internet Services-Namensserver verweist. Wenn Sie nicht über eine eigene Domäne verfügen, können Sie von einem Registrator (z. B. [godaddy.com](http://godaddy.com)) eine Domäne erwerben.
1. Installieren Sie alle erforderlichen Befehlszeilentools (CLI-Tools), indem Sie [die folgenden Schritte ausführen](https://{DomainName}/docs/cli?topic=cloud-cli-ibmcloud-cli#overview).

## Angepasste Domäne konfigurieren

Erstellen Sie zuerst eine Instanz von IBM Cloud Internet Services (CIS) und verweisen Sie von Ihrer angepassten Domäne auf CIS-Namensserver.

1. Navigieren Sie zu [Internet Services](https://{DomainName}/catalog/services/internet-services) im {{site.data.keyword.Bluemix_notm}}-Katalog.
1. Legen Sie den Servicenamen fest und klicken Sie auf **Erstellen**, um eine Instanz des Service zu erstellen. Für dieses Lernprogramm können Sie beliebige Preistarife verwenden.
1. Legen Sie nach dem Bereitstellen der Serviceinstanz Ihren Domänennamen fest, indem Sie auf **Beginnen** und anschließend auf **Domäne hinzufügen** klicken.
1. Klicken Sie auf **Nächster Schritt**. Konfigurieren Sie nach dem Zuordnen der Namensserver den gewünschten Registrator oder DNS-Provider für die Verwendung der aufgelisteten Namensserver.
1. Nach dem Konfigurieren des gewünschten Registrators oder DNS-Providers kann es bis zu 24 Stunden dauern, bis die Änderungen wirksam werden.

   Wenn der Status der Domäne auf der Seite 'Übersicht' von *Anstehend* in *Aktiv* geändert ist, können Sie mit dem Befehl `dig <your_domain_name> ns` überprüfen, ob die neuen Namensserver verwendet werden.
   {:tip}

### Zertifikat für angepasste Domäne anfordern

Für das Bereitstellen von {{site.data.keyword.openwhisk_short}}-Aktionen in einer angepassten Domäne ist eine sichere HTTPS-Verbindung erforderlich. Sie sollten ein SSL-Zertifikat für die Domäne und Unterdomäne abrufen, die Sie mit dem serverunabhängigen Back-End verwenden möchten. Wenn eine Domäne wie *mydomain.com* verwendet wird, können die Aktionen unter *api.mydomain.com* gehostet werden. In diesem Fall muss das Zertifikat für *api.mydomain.com* ausgegeben werden.

Kostenlose SSL-Zertifikate können Sie bei [Let's Encrypt](https://letsencrypt.org/) anfordern. Während dieses Vorgangs müssen Sie gegebenenfalls einen DNS-Datensatz im TXT-Format in der DNS-Schnittstelle von Cloud Internet Services konfigurieren, um sich als Eigner der Domäne auszuweisen.
{:tip}

Nachdem Sie das SSL-Zertifikat und den privaten Schlüssel für Ihre Domäne erhalten haben, müssen Sie diese in das [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail)-Format umwandeln.

1. So wandeln Sie ein Zertifikat in das PEM-Format um:
   ```
   openssl x509 -in domain-crt.txt -out domain-crt.pem -outform PEM
   ```
   {: pre}
1. So wandeln Sie einen privaten Schlüssel in das PEM-Format um:
   ```
   openssl rsa -in domain-key.txt -out domain-key.pem -outform PEM
   ```
   {: pre}

### Zertifikat in zentrales Repository importieren

1. Erstellen Sie eine [{{site.data.keyword.cloudcerts_short}}](https://{DomainName}/catalog/services/cloudcerts)-Instanz an einem unterstützten Standort.
1. Verwenden Sie im Service-Dashboard die Funktion **Zertifikat importieren**:
   * Geben Sie für **Name** die angepasste Unterdomäne und Domäne an (z. B. *api.mydomain.com*).
   * Suchen Sie nach der **Zertifikatsdatei** im PEM-Format.
   * Suchen Sie nach der **Datei mit dem privaten Schlüssel** im PEM-Format.
   * Klicken Sie auf **Importieren**.

## Aktionen an mehreren Standorten bereitstellen

In diesem Abschnitt erstellen Sie Aktionen, stellen diese als API bereit und weisen die API der angepassten Domäne mit einem SSL-Zertifikat zu, das in {{site.data.keyword.cloudcerts_short}} gespeichert ist.

<p style="text-align: center;">

  ![API-Architektur](images/solution44-multi-region-serverless/api-architecture.png)
</p>

Die Aktion **doWork** implementiert eine Ihrer API-Operationen. Die Aktion **healthz** wird später zum Überprüfen des einwandfreien Zustands Ihrer API verwendet. Die Aktion könnte ohne weitere Auswirkungen einfach nur *OK* zurückgeben oder in einem komplexeren Vorgang die Datenbanken oder andere kritische Services, die für Ihre API erforderlich sind, mit Ping überprüfen.

Die drei folgenden Abschnitte müssen für jeden Standort wiederholt werden, an dem das Back-End der Anwendung gehostet werden soll. Für das vorliegende Lernprogramm können Sie die Standorte *Dallas (us-south)* und *London (eu-gb)* als Ziele auswählen.

### Aktionen definieren

1. Rufen Sie [{{site.data.keyword.openwhisk_short}} / Aktionen](https://{DomainName}/openwhisk/actions) auf.
1. Wechseln Sie zum Zielstandort und wählen Sie eine Organisation und einen Bereich zum Bereitstellen der Aktionen aus.
1. Erstellen Sie eine Aktion.
   1. Geben Sie für **Name** den Namen **doWork** an.
   1. Geben Sie für **Umschließendes Paket** die Einstellung **default** an.
   1. Geben Sie für **Laufzeit** die neueste Version von **Node.js** an.
   1. Aktivieren Sie die Option **Erstellen**.
1. Ändern Sie den Aktionscode wie folgt:
   ```js
   function main(params) {
     msg = "Hello, " + params.name + " from " + params.place;
     return { greeting:  msg, host: params.__ow_headers.host };
   }
   ```
   {: codeblock}
1. Aktivieren Sie die Option **Speichern**.
1. Erstellen Sie eine weitere Aktion, die als Statusprüfung für Ihre API verwendet werden soll:
   1. Geben Sie für **Name** den Namen **healthz** an.
   1. Geben Sie für **Umschließendes Paket** die Einstellung **default** an.
   1. Geben Sie für **Laufzeit** die neueste Version von **Node.js** an.
   1. Aktivieren Sie die Option **Erstellen**.
1. Ändern Sie den Aktionscode wie folgt:
   ```js
   function main(params) {
     return { ok: true };
   }
   ```
   {: codeblock}
1. Aktivieren Sie die Option **Speichern**.

### Aktionen durch verwaltete API bereitstellen

Im nächsten Schritt wird eine verwaltete API erstellt, um Ihre Aktionen bereitzustellen.

1. Rufen Sie [{{site.data.keyword.openwhisk_short}} / API](https://{DomainName}/openwhisk/apimanagement) auf.
1. Erstellen Sie eine neue verwaltete {{site.data.keyword.openwhisk_short}}-API:
   1. Geben Sie für **API-Name** den Namen **App API** an.
   1. Geben Sie für **Basispfad** die Zeichenfolge **/api** an.
1. Erstellen Sie eine Operation:
   1. Geben Sie für **Pfad** die Zeichenfolge **/do** an.
   1. Geben Sie für **Verb** die Zeichenfolge **GET** an.
   1. Geben Sie für **Paket** die Einstellung **default** an.
   1. Geben Sie für **Aktion** die Zeichenfolge **doWork** an.
   1. Aktivieren Sie die Option **Erstellen**.
1. Erstellen Sie eine weitere Operation:
   1. Geben Sie für **Pfad** die Zeichenfolge **/healthz** an.
   1. Geben Sie für **Verb** die Zeichenfolge **GET** an.
   1. Geben Sie für **Paket** die Einstellung **default** an.
   1. Geben Sie für **Aktion** die Zeichenfolge **healthz** an.
   1. Aktivieren Sie die Option **Erstellen**.
1. Aktivieren Sie die Option **Speichern**, um die API zu speichern.

### Angepasste Domäne für verwaltete API konfigurieren

Beim Erstellen einer verwalteten API wird ein Standardendpunkt wie `https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/1234abcd/app` bereitgestellt. Im vorliegenden Abschnitt konfigurieren Sie diesen Endpunkt, damit Anforderungen verarbeitet werden können, die von Ihrer angepassten Unterdomäne stammen (d. h. von der Domäne, die später in IBM Cloud Internet Services konfiguriert wird).

1. Rufen Sie [APIs / Angepasste Domänen](https://{DomainName}/apis/domains) auf.
1. Wählen Sie im Selektorelement **Region** den Zielstandort aus.
1. Suchen Sie die angepasste Domäne, die mit der Organisation und dem Bereich verknüpft ist, in der bzw. dem Sie die Aktionen und die verwaltete API erstellt haben. Klicken Sie im Aktionsmenü auf **Einstellungen ändern**.
1. Notieren Sie den Wert für **Standarddomäne / Alias**.
1. Wählen Sie die Option **Angepasste Domäne anwenden** aus.
   1. Geben Sie für **Domänenname** die Domäne an, die Sie mit CIS Global Load Balancer verwenden möchten (z. B. *api.mydomain.com*).
   1. Wählen Sie die {{site.data.keyword.cloudcerts_short}}-Instanz aus, die das Zertifikat enthält.
   1. Wählen Sie das Zertifikat für die Domäne aus.
1. Rufen Sie das Dashboard für Ihre Instanz von **Cloud Internet Services** auf und erstellen Sie unter **Zuverlässigkeit / DNS** einen neuen **DNS-TXT-Datensatz**:
   1. Geben Sie für **Name** Ihre angepasste Unterdomäne an (z. B. **api**).
   1. Geben Sie für **Inhalt** den Wert für **Standarddomäne / Alias** an.
   1. Speichern Sie den Datensatz.
1. Speichern Sie die Einstellungen für die angepasste Domäne. Das Dialogmodul überprüft, ob der DNS-TXT-Datensatz vorhanden ist.

   Wenn der TXT-Datensatz nicht gefunden wird, müssen Sie gegebenenfalls warten, bis der Datensatz weitergegeben wurde. Versuchen Sie danach erneut, die Einstellungen zu speichern. Der DNS-TXT-Datensatz kann entfernt werden, nachdem die Einstellungen angewendet wurden.{: tip}

Wiederholen Sie die vorherigen Abschnitte, um weitere Standorte zu konfigurieren.

## Datenverkehr auf Standorte verteilen

**Sie haben nun Aktionen an mehreren Standorten eingerichtet**, aber es gibt keinen zentralen Endpunkt, über den die Aktionen erreichbar sind. In diesem Abschnitt konfigurieren Sie eine globale Lastausgleichsfunktion (Global Load Balancer, GLB), um Datenverkehr auf die Standorte zu verteilen.

<p style="text-align: center;">

  ![Architektur der globalen Lastausgleichsfunktion](images/solution44-multi-region-serverless/glb-architecture.png)
</p>

### Statusprüfung erstellen

Dieser Endpunkt wird von Internet Services wiederholt aufgerufen, um den Back-End-Status zu überprüfen.

1. Rufen Sie das Dashboard Ihrer IBM Cloud Internet Services-Instanz auf.
1. Erstellen Sie unter **Zuverlässigkeit/Globaler Lastausgleich** eine Statusprüfung:
   1. Geben Sie für **Monitortyp** die Einstellung **HTTPS** an.
   1. Geben Sie für **Pfad** die Zeichenfolge **/api/healthz** an.
   1. **Stellen Sie die Ressource bereit**.

### Ursprungspools erstellen

Wenn Sie einen Pool pro Standort erstellen, können Sie später in Ihrer globalen Lastausgleichsfunktion geografische Routen konfigurieren, um Benutzer zum nächstgelegenen Standort umzuleiten. Eine weitere Möglichkeit besteht darin, einen einzelnen Pool mit allen Standorten zu erstellen, damit der Lastausgleichszyklus die einzelnen Ursprungselemente im Pool nacheinander anfragen kann.

Führen Sie für jeden Standort Folgendes aus:
1. Erstellen Sie einen Ursprungspool.
1. Geben Sie für **Name** den Namen **app-&lt;standort&gt;** an (z. B. _app-Dallas_).
1. Wählen Sie die zuvor erstellte Statusprüfung aus.
1. Geben Sie als **Region für Statusprüfung** eine Region in der Nähe des Standorts an, an dem {{site.data.keyword.openwhisk_short}}-Ressourcen bereitgestellt sind.
1. Geben Sie für **Ursprungsname** den Namen **app-&lt;standort&gt;** an.
1. Geben Sie für **Ursprungsadresse** die Standarddomäne bzw. den Standardalias der verwalteten API an (z. B. _5d3ffd1eb6.us-south.apiconnect.appdomain.cloud_).
1. **Stellen Sie die Ressource bereit**.

### Globlae Lastausgleichsfunktion erstellen

1. Erstellen Sie eine Lastausgleichsfunktion.
1. Geben Sie für **Hostname für Lastausgleichsfunktion** den Namen **api.mydomain.com** an.
1. Fügen Sie die regionalen Ursprungspools hinzu.
1. **Stellen Sie die Ressource bereit**.

Rufen Sie nach einer kurzen Wartezeit `https://api.mydomain.com/api/do?name=John&place=Earth` auf. Als Antwort müsste die im ersten betriebsbereiten Pool enthaltene Funktion zurückgegeben werden.

### Failover testen

Zum Testen der Failover-Funktion muss ein Pool ausfallen, damit die globale Lastausgleichsfunktion (Global Load Balancer, GLB) eine Umleitung zum nächsten betriebsbereiten Pool einrichtet. Um einen Ausfall zu simulieren, können Sie die Funktion für Statusprüfung entsprechend präparieren.

1. Rufen Sie [{{site.data.keyword.openwhisk_short}} / Aktionen](https://{DomainName}/openwhisk/actions) auf.
1. Wählen Sie den ersten Standort aus, der in der GLB konfiguriert ist.
1. Bearbeiten Sie die Funktion `healthz` und ändern Sie die Implementierung der Funktion in `throw new Error()`.
1. Speichern Sie die Änderung.
1. Warten Sie, bis die Statusprüfung für diesen Ursprungspool ausgeführt wird.
1. Rufen Sie `https://api.mydomain.com/api/do?name=John&place=Earth` erneut auf. Diesmal müsste eine Umleitung zu dem anderen, betriebsbereiten, Ursprungspool erfolgen.
1. Machen Sie die Codeänderung rückgängig, sodass wieder ein einwandfreier Ursprungspool aufgerufen wird.

## Ressourcen entfernen
{: #removeresources}

### CIS-Ressourcen entfernen

1. Entfernen Sie die GLB.
1. Entfernen Sie die Ursprungspools.
1. Entfernen Sie die Statusprüfungen.

### Aktionen entfernen

1. Entfernen Sie [APIs](https://{DomainName}/openwhisk/apimanagement).
1. Entfernen Sie [Aktionen](https://{DomainName}/openwhisk/actions).

## Zugehörige Inhalte
{: #related}

* IBM Cloud [Internet Services](https://{DomainName}/docs/infrastructure/cis?topic=cis-getting-started-with-ibm-cloud-internet-services-cis-#getting-started-with-ibm-cloud-internet-services-cis-)
* [Ausfallsichere und geschützte Kubernetes-Cluster in mehreren Regionen durch Cloud Internet Services](https://{DomainName}/docs/tutorials?topic=solution-tutorials-multi-region-k8s-cis#multi-region-k8s-cis)
