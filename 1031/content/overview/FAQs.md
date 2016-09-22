
# Häufig gestellte Fragen (FAQs) zu Microsoft Graph

## Welche Plattformen werden von der Microsoft Graph-API unterstützt?
<!--
Apps can use the Microsoft Graph API to perform create, read, update, and delete (CRUD) operations on data sources and entities, giving them seamless access to work data. 

**Ease of use--one endpoint, all Office 365 data under one roof**

You can use the API in four steps:
1.  Select your programming language and development environment.
2.  Build your app.
3.  Optionally, host your app in Microsoft Azure or any cloud platform you choose.
4.  Authenticate your users by using single sign-on with Azure AD.

As a developer you can use the API to create custom apps that access and interact with all the richness of enterprise and productivity data--users, groups, organizational contacts, files, folders, mail, calendar, insights and relationships--and build apps across all mobile, web, and desktop platforms. No matter your development platform or tools. Using a single service endpoint to access those entities and data. And a single authentication flow.  -->

Sie haben folgenden Möglichkeiten:

<!--Just like in Office 365 APIs, Office 365 unified endpoint API  allows you to build apps using any development environment of your choice:  -->

- Verwenden Sie eine beliebige Entwicklungsumgebung, mit der Sie vertraut sind, wie z. B. .NET, PHP, Java, Python oder Ruby.
- Verwenden Sie eine beliebige Programmiersprache, Entwicklungsplattform und Hostumgebung.
- Erstellen Sie eine App, die mithilfe einer beliebigen Internetsprache auf die API zugreifen kann, einschließlich JavaScript, HTML5, Python, Ruby, PHP und ASP.NET.  
- Verwenden Sie Ihre bevorzuge IDE, z. B. Visual Studio, Eclipse, Android Studio, Xcode oder eine andere vertraute Entwicklungsumgebung.
- Hosten Sie Ihre Apps in Microsoft Azure oder einer anderen beliebigen Cloudplattform.
- Entwickeln Sie Apps für Windows Universal, iOS, Android oder andere Geräteplattformen.
- Rufen Sie die API über Office-Add-Ins (zuvor Apps für Office) oder SharePoint-Add-Ins (zuvor Apps für SharePoint) auf.
 


## Was spricht für die Verwendung der Microsoft Graph-API?

Nehmen wir an, dass Sie Dateien eines Benutzers, ein Profilbild und den Manager der Person, die die Datei zuletzt in der Organisation bearbeitet hat, programmgesteuert abrufen möchten. Da die Informationen in verschiedenen Diensten wie Azure Active Directory, SharePoint und Exchange gespeichert sind, umfasst diese Aufgabe mehrere Schritte, die mit den Office 365-APIs durchgeführt werden: 

1. Verwenden des Ermittlungsdiensts zum Suchen der verschiedenen Dienstendpunkte 
2. Ermitteln der URL der Dienste, zu denen Ihre Office 365-Apps eine Verbindung herstellen möchten
3. Anschließend Erwerben und Verwalten des Zugriffstokens für die einzelnen Dienste und Stellen der Anfrage direkt an den Dienst

Jetzt können Sie die Microsoft Graph-API verwenden, um den gleichen komplexen Vorgang über einen einzelnen REST-API-Endpunkt auszuführen. Sie müssen nicht für jeden der Dienste einen anderen Endpunkt ermitteln und zu diesem wechseln, keine separaten Zugriffstoken für die einzelnen Dienste abrufen und verwalten und auch keine Silodienste und unterschiedliche Datenmodelle behandeln.

##Beispiele für Abfragen

Das folgende Beispiel zeigt das aktuelle Modell für die Interaktion mit der Office 365-API bei Verwendung unterschiedlicher Dienstendpunkte und wie dies durch die Microsoft Graph-API wesentlich vereinfacht wird.

**Verschiedene Dienstendpunkte**

|   **Vorgang**                  |  **API**                          |  **Dienstendpunkt** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Dienstendpunkte für die Office 365-API ermitteln               |     `Discovery Service`           | _https://_**api.office.com**_/discovery/v1.0/me/services_ |
| Benutzer abrufen           |     `Azure AD Graph API` | _https://_**graph.windows.net**_/contoso.com/users?api-version=2013-04-05_|
| Nachrichtensammlung aus Posteingang abrufen       |     `Office 365 API`           | _https://_**outlook.office365.com**_/api/v1.0/me/messages_  |
| Joes Dateien abrufen   |     `Office 365 API`  | _https://_**contoso-my.sharepoint.com**_/personal/joe_contoso_com/_api/v1.0/files_ |


Bei Verwendung der Microsoft Graph-API müssen Sie nicht zuerst die Dienstendpunkte ermitteln und dann die verschiedenen Endpunkte durchlaufen, um die Dateien eines Benutzers, seine E-Mail-Nachrichten usw. abzurufen. Sie müssen nur mit einem einzelnen REST-URL-Namespace interagieren, der _**graph.microsoft.com**_ lautet.

**Microsoft Graph-API**

|   **Vorgang**                  |  **API**                          |  **Dienstendpunkt** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Dienstendpunkte für die Office 365-API ermitteln                |     `Microsoft Graph`           | Nicht benötigt |
| Benutzer abrufen           |     `Microsoft Graph` | _https://_**graph.microsoft.com**_/v1.0/contoso.onmicrosoft.com/users_ |
| Nachrichtensammlung aus Posteingang abrufen       |     `Microsoft Graph`           | _https://_**graph.microsoft.com**_/v1.0/me/messages_  |
| Joes Dateien abrufen   |     `Microsoft Graph `  | _https://_**graph.microsoft.com**_/v1.0/me/drive/root/children_ |


## Was sind die Vorteile bei der Verwendung der Microsoft Graph-API?

Nachfolgend sind einige der Vorteile bei der Verwendung der Microsoft Graph-API aufgeführt:

**Einheitliche und optimierte Entwicklerumgebung zur Nutzung von Microsoft-Clouddiensten**

-   Einzelner Namespace für alle Dienstendpunkte. Die Ermittlung von Dienstendpunkten ist nicht mehr notwendig.
-   Ein Token für den Zugriff auf alle Ressourcen
-   Integrierte und direkte Navigation zwischen aktuellen Silodiensten (z. B. Abrufen der Abteilungs- und Managementkette des Benutzers, der ein bestimmtes Dokument verfasst hat)
-   Für die Verbindung mit mehreren Diensten ist nur ein einziger API-Satz erforderlich, d. h. nur die Microsoft Graph-API
-   Einheitliche und erweiterte REST-API und Entitäten übergreifend über die Office-Plattform 
-   Konsistente Eigenschaftenbenennung und Schemas über Entitäten hinweg, einschließlich Navigationseigenschaften zwischen Entitäten

**Produktivere Arbeitsumgebung mit Office**

-   Kontextbezogener Inhalt. Dokumente können zum Beispiel nach Gruppe, Projekt, Team oder Trends gesucht werden.
-   Kontextbezogene Benutzerbeziehungen. Benutzer können zum Beispiel nach Gruppenmitgliedschaft, Interessen, Fertigkeiten und Fachwissen gesucht werden.  Es können auch die Organigrammbeziehungen ermittelt werden.

**Erstellen von Apps mit einer beliebigen Programmiersprache auf einer beliebigen Plattform**

-   Entwicklungstools und -ressourcen für alle Entwickler. Sie können die Entwicklung mit jeder Plattform und Sprache durchführen. 
-   Entwicklung für mobile Geräte für alle Plattformen mit offenen Technologien.  
-   Es ist kein Fachwissen über Exchange, SharePoint oder Azure AD erforderlich, um auf Microsoft Graph-API-Entitäten zuzugreifen.

<!---<a name="msg_v2auth"> </a>-->

## Unterstützt die Microsoft Graph-API den App-Authentifizierungsendpunkt v2.0?

Ja. Weitere Informationen finden Sie unter [Authentifizieren von Microsoft Graph-Endpunkten mit dem v2-Authentifizierungsendpunkt](http://graph.microsoft.io/docs/authorization/converged_auth).



  > Ihr Feedback ist uns wichtig. Nehmen Sie unter [Stack Overflow](http://stackoverflow.com/questions/tagged/office365) Kontakt mit uns auf. Taggen Sie Ihre Fragen mit [MicrosoftGraph] und [office365].








