# Unabhängige Cloudbereitstellungen


Dieser Artikel enthält Informationen über die verschiedenen unabhängigen Cloudinstanzen von Microsoft Graph und die Features, die Entwicklern zur Verfügung stehen. 


## Microsoft Graph betrieben von 21Vianet in China

Dieser Abschnitt enthält Informationen über Microsoft Graph betrieben von 21Vianet, und die Features, die Entwicklern zur Verfügung stehen.

### Stammendpunkte des Diensts
| Microsoft Graph betrieben von 21Vianet | Microsoft Graph|
|---------------------------|----------------|
| https://microsoftgraph.chinacloudapi.cn | https://graph.microsoft.com|

### Microsoft Graph-Explorer
| Microsoft Graph betrieben von 21Vianet | Microsoft Graph|
|---------------------------|----------------|
|https://graphexplorerchina.azurewebsites.net| https://graphexplorer2.azurewebsites.net|

### Azure OpenID Connect und OAuth2.0
Die zum Erwerben von Tokens zum Anmelden bei oder Aufrufen von Microsoft Graph, betrieben von 21Vianet, verwendeten Endpunkte weichen von denen bei anderen Angeboten ab. 

| Microsoft Graph betrieben von 21Vianet | Microsoft Graph|
|---------------------------|----------------|
| https://login.chinacloudapi.cn | https://login.microsoftonline.com|
 
Verwenden Sie https://login.chinacloudapi.cn/common/oauth2/authorize zum Authentifizieren des Benutzers und https://login.chinacloudapi.cn/common/oauth2/token zum Abrufen eines Tokens für Ihre App, um Microsoft Graph betrieben von 21Vianet aufzurufen.

> **HINWEIS**: Die aktuellen [v2.0-Autorisierungs- und Tokenendpunkte](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview/) stehen NICHT zur Verwendung mit Microsoft Graph betrieben von 21Vianet zur Verfügung.  Apps können nur auf Organisationsdaten und nicht auf Konsumentendaten zugreifen. 

### Dienstfunktionen für Microsoft Graph betrieben von 21Vianet
Die folgenden Features von Microsoft Graph sind allgemein verfügbar (auf dem `/v1.0`-Endpunkt):

* Benutzer
* Gruppen
* Dateien
* E-Mails
* Kalender
* Persönliche Kontakte 
* CRUD-Vorgänge (Erstellen, Lesen, Aktualisieren und Löschen)
* CORS-Unterstützung (Cross-Origin Resource Sharing)

Die folgenden Features von Microsoft Graph stehen auch als Preview zur Verfügung (auf dem `/beta`-Endpunkt):

* Organisationskontakte
* Anwendungen
* Dienstprinzipale
* Verzeichnisschemaerweiterungen
