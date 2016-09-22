# Aufrufen von Microsoft Graph in einer Angular-App 

In diesem Artikel werden Aufgaben beschrieben, die zum Herstellen einer Verbindung zwischen Ihrer Anwendung und Office 365 und zum Aufrufen der Microsoft Graph-API mindestens erforderlich sind. Wir verwenden den Code aus [Office 365 Angular Connect-Beispiel unter Verwendung von Microsoft Graph](https://github.com/microsoftgraph/angular-connect-rest-sample), um die wichtigsten Konzepte zu erläutern, die Sie in Ihrer App implementieren müssen.

![Screenshot des Office 365 Angular Connect-Beispiels](./images/web-screenshot.png)

## Anforderungen  

Folgendes wird in diesem Thema vorausgesetzt:

* Sie sind mit JavaScript und dem [AngularJS](https://angularjs.org/)-Code vertraut.

## Übersicht

Zum Aufrufen der Microsoft Graph-API müssen Sie die folgenden Aufgaben ausführen.

1. Registrieren der Anwendung in Azure Active Directory
2. Konfigurieren der Azure Active Directory-Bibliothek für JavaScript (ADAL JS)
3. Verwenden von ADAL JS zum Abrufen eines Zugriffstokens
4. Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

<!--<a name="register"></a>-->
## Registrieren der Anwendung in Azure Active Directory

Bevor Sie  mit Office 365 arbeiten können, müssen Sie Ihre Anwendung registrieren und die Berechtigungen für die Verwendung der Microsoft Graph-Dienste festlegen. Mithilfe des [App-Registrierungstools](https://dev.office.com/app-registration) können Sie mit nur wenigen Klicks Ihre Anwendung für den Zugriff auf ein Geschäfts- oder Schulkonto des Benutzers registrieren. 
Zum Verwalten müssen Sie das [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) verwenden.

Alternativ können Sie die App manuell registrieren. Die Anweisungen dazu finden Sie im Artikel [Registrieren der browserbasierten Web-App mithilfe des Azure-Verwaltungsportals](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp). Beachten Sie dabei Folgendes:

* Geben Sie http://127.0.0.1:8080/ als die **Anmelde-URL** an.
* Konfigurieren Sie nach der Registrierung der Anwendung [die **delegierten Berechtigungen**](https://github.com/microsoftgraph/angular-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), die für Ihre Angular-App erforderlich sind. Für das Connect-Beispiel ist die Berechtigung zum **Senden von E-Mails als angemeldeter Benutzer** erforderlich.

Notieren Sie sich die folgenden Werte auf der Seite **Konfigurieren** in Ihrer Azure-Anwendung, da Sie diese Werte zum Konfigurieren von [ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) in Ihrer Angular-App benötigen.

* Client-ID (eindeutig für Ihre Anwendung)
* Eine Antwort-URL (http://127.0.0.1:8080/)

<!--<a name="adal"></a>-->
## Konfigurieren der Azure Active Directory-Bibliothek für JavaScript (ADAL JS)

[ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) ist eine JavaScript-Bibliothek, die umfassende Unterstützung für die Anmeldung von Azure AD-Benutzern in Einzelseiten-Anwendungen (Single-page applications, SPAs) bietet, z. B. das Connect-Bespiel und Tokenverwaltung sowie weitere Features. Zur Nutzung dieser Bibliothek muss Ihre Angular-App diese einbeziehen und konfigurieren.

Beziehen Sie die Bibliothek und deren Angular-spezifisches Modul mithilfe des Microsoft CDN mit ein.

```html
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal.min.js"></script>
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal-angular.min.js"></script>
```

Als Nächstes müssen Sie den ADAL JS-Dienst konfigurieren, unabhängig davon, wo Sie die Abhängigkeiten der Angular-App konfigurieren. In dem Connect-Beispiel erfolgt die Konfiguration in [*public/app.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/scripts/app.js). 

Um ADAL JS zu konfigurieren, müssen Sie zunächst einen Verweis auf das ADAL-Modul durch Hinzufügen von ```AdalAngular``` zum erforderlichen Array des Moduls hinzufügen und ```adalAuthenticationServiceProvider``` an die ```config```-Funktion übergeben. Konfigurieren Sie die Bibliothek mit der ```init```-Funktion, übergeben Sie die Client-ID Ihrer Anwendung und ein ```endpoints```-Objekt, das deklariert, welche APIs Ihre App für CORS-Anforderungen benötigt.

```javascript
// Initialize the ADAL provider with your clientID (found in the Azure Management Portal) and 
// the API URL (to enable CORS requests).
adalAuthenticationServiceProvider.init(
  {
    clientId: clientId,
    // The endpoints here are resources for cross origin requests.
    endpoints: {
      'https://graph.microsoft.com': 'https://graph.microsoft.com'
    }
  },
  $httpProvider
);
```

<!--<a name="accessToken"></a>-->
## Verwenden von ADAL JS zum Abrufen eines Zugriffstokens

Die App muss den Browser zu einer Anmeldeseite umleiten, damit der Benutzer sich anmelden und der Anwendung den Zugriff auf seine Daten gewähren kann. Im Connect-Beispiel wird ADAL JS für diese Aufgabe verwendet. 

Fügen Sie in einem der Anwendungscontroller zunächst einen Verweis auf den ADAL-Dienst durch Einfügen von ```adalAuthenticationService``` in Ihren Controller hinzu, und definieren Sie dann eine Funktion, die die ```login```-Funktion des Dienst verwendet, mit der die Benutzeroberfläche aufgerufen werden kann. Im Connect-Beispiel erfolgt dies in der Datei [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js) 

```javascript
/**
  * Expose the login method from ADAL to the view.
  */
function connect() {
  adalAuthenticationService.login();
};
```

Wenn diese Funktion aufgerufen wird, wird Ihre Anwendung den Benutzer zu einer Anmeldeseite umleiten. Nachdem sie sich angemeldet und Ihre App autorisiert haben, werden Benutzer mit dem Zugriffstoken in der Abfragezeichenfolge, die von ADAL JS abgerufen und gespeichert wird, zu Ihrer App umgeleitet. 

<!--<a name="request"></a>-->
## Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

Mit einem Zugriffstoken kann Ihre App authentifizierte Anforderungen an die Microsoft Graph-API senden. ADAL JS fängt automatisch alle HTTP-Anforderungen ab und fügt das Zugriffstoken zu diesen hinzu, sodass Sie diesen Header beim Verwenden der Bibliothek nicht manuell festlegen müssen. 

Im Connect-Beispiel wird eine E-Mail unter Verwendung des ```me/sendMail```-Endpunkts in der Microsoft Graph-API in der Datei [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js) gesendet. 

Microsoft Graph ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit der Microsoft Graph-API finden Sie in der [API-Referenz](http://graph.microsoft.io/docs/api-reference/v1.0).

