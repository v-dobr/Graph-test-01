# Aufrufen von Microsoft Graph mit einer Node.js-App

In diesem Artikel werden Aufgaben beschrieben, die zum Herstellen einer Verbindung zwischen Ihrer Anwendung und Office 365 und zum Aufrufen der Microsoft Graph-API mindestens erforderlich sind. Wir verwenden den Code aus dem [Office 365 Node.js Connect-Beispiel unter Verwendung von Microsoft Graph](https://github.com/microsoftgraph/nodejs-connect-rest-sample), um die wichtigsten Konzepte zu erläutern, die Sie in Ihrer App implementieren müssen.

![Screenshot des Office 365 Node.js Connect-Beispiels](./images/web-screenshot.png)

## Übersicht

Zum Aufrufen der Microsoft Graph-API muss Ihre Web-App die folgenden Aufgaben ausführen:

1. Registrieren der Anwendung in Azure Active Directory 
2. Installieren der Azure Active Directory-Clientbibliothek für Node.js
3. Weiterleiten des Browsers zur Anmeldeseite
4. Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite
5. Verwenden von `adal-node` zum Anfordern eines Zugriffstokens
6. Senden einer Anforderung an die Microsoft Graph-API

<!--<a name="register"/>-->
## Registrieren Ihrer Anwendung in Azure Active Directory

Bevor Sie  mit Office 365 arbeiten können, müssen Sie Ihre Anwendung registrieren und die Berechtigungen für die Verwendung der Microsoft Graph-Dienste festlegen. 
Mithilfe des [App-Registrierungstools](https://dev.office.com/app-registration) können Sie mit nur wenigen Klicks Ihre Anwendung für den Zugriff auf ein Geschäfts- oder Schulkonto des Benutzers registrieren. Zum Verwalten müssen Sie das [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) verwenden.

Alternativ können Sie die App manuell registrieren. Die Anweisungen dazu finden Sie im Abschnitt [Registrieren der Webserver-App mithilfe des Azure-Verwaltungsportals](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp). Beachten Sie dabei Folgendes:

* Geben Sie eine Seite in Ihrer Node.js-App als **Anmelde-URL** in Schritt 6 an. Im Falle des Connect-Beispiels ist die URL „http://localhost:8080/login“, die der [/login](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L33)-Weiterleitung zugeordnet ist.
* [Konfigurieren Sie die **delegierten Berechtigungen**](https://github.com/microsoftgraph/nodejs-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), die für Ihre App erforderlich sind. Für das Connect-Beispiel ist die Berechtigung zum **Senden von E-Mails als angemeldeter Benutzer** erforderlich.

Notieren Sie sich die folgenden Werte auf der Seite **Konfigurieren** Ihrer Azure-Anwendung.

* Client-ID
* Ein gültiger Schlüssel
* Eine Antwort-URL

Sie benötigen diese Werte als Parameter im OAuth-Fluss in Ihrer App.

<!--<a name="adal">-->
## Installieren der Azure Active Directory-Clientbibliothek für Node.js

Die ADAL für die Node.js-Bibliothek vereinfacht Node.js-Anwendungen das Authentifizieren bei AAD, um auf geschützte AAD-Webressourcen zuzugreifen.
Um adal-node zum vorhandenen `package.json` hinzuzufügen, geben Sie Folgendes in Ihrem bevorzugten Terminal ein.

`npm install adal-node --save`

Weitere Informationen zur adal-node-Clientbibliothek finden Sie in den Paketinformationen zu [npm](https://www.npmjs.com/package/adal-node). 
Informationen zu Problemen, Quellcode und zu neuen bevorstehenden Funktionen und Korrekturen finden Sie im adal-node-Projekt auf [Github](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs).

<!--<a name="redirect"/>-->
## Weiterleiten des Browsers zur Anmeldeseite

Die App muss den Browser an die Anmeldeseite weiterleiten, um einen Autorisierungscode zu erhalten und den OAuth 2.0-Fluss fortzusetzen.

Im Connect-Beispiel wird die Authentifizierungs-URL aus [`authHelper.js#getAuthUrl`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js#L17) von der [`login.hbs#login`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/views/login.hbs#L2)-Funktion über ein clientseitiges `onclick`-Ereignis umgeleitet.

**authHelper.js#getAuthUrl**
```javascript
/**
 * Generate a fully formed uri to use for authentication based on the supplied resource argument
 * @return {string} a fully formed uri with which authentcation can be completed
 */
function getAuthUrl() {
    return credentials.authority + "/oauth2/authorize" +
        "?client_id=" + credentials.client_id +
        "&response_type=code" +
        "&redirect_uri=" + credentials.redirect_uri;
};
```

**login.hbs#login**
```javascript
function login() {
    window.location = '{{auth_url}}'.replace(/&amp;/g, '&'); // transform HTML special char from .hbs template rendering
}
```

<!--<a name="authcode"/>-->
## Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite

Nachdem der Benutzer sich angemeldet hat, gibt der Fluss den Browser an die Antwort-URL in Ihrer App zurück. Der Autorisierungscode ist in der `code`-Variablen für die Abfragezeichenfolge verfügbar.

```javascript
router.get('/<application reply url>', function (req, res, next) {
  var authCode = req.query.code;
  // your router's implementation
});
```

Den [relevanten Code](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L34) finden Sie im Connect-Beispiel.

<!--<a name="accesstoken"/>-->
## Verwenden von `adal-node` zum Anfordern eines Zugriffstokens

Da wir nun mit Azure Active Directory authentifiziert haben, ist der nächste Schritt das Abrufen eines Zugriffstokens über adal-node. Danach können wir REST-Anforderungen an die Microsoft Graph-API senden.

Um ein Zugriffstoken anzufordern, bietet adal-node zwei Rückruffunktionen.

|                          Funktion                         |                                      Parameter                                      | Beschreibung                                                                                             |
|:---------------------------------------------------------:|:--------------------------------------------------------------------------------:|---------------------------------------------------------------------------------------------------------|
| `AuthenticationContext.acquireTokenWithAuthorizationCode` | `authCode`, `redirect_uri`, `resource`, `client_id`, `client_secret`, `callback` | stellt ein Zugriffstoken für eine angegebene Ressource basierend auf dem Autorisierungscode bereit, der bei der Anmeldung zurückgegeben wird. |
| `AuthenticationContext.acquireTokenWithRefreshToken`      | `token`, `client_id`, `client_secret`, `resource`, `callback`                    | stellt ein Zugriffstoken für eine angegebene Ressource basierend auf einem Aktualisierungstoken bereit.                             |

Im Connect-Beispiel werden Anforderungen durch [`authHelper.js`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js) umgeleitet, sodass `client_id` und `client_secret` hinzugefügt werden können.

```javascript
// The application registration (must match Azure AD config)
var credentials = {
    authority: "https://login.microsoftonline.com/common",
    client_id: "<your client id here>",
    client_secret: "<your client secret>",
    redirect_uri: "http://localhost:8080/login"
};

/**
 * Gets a token for a given resource.
 * @param {string} code An authorization code returned from a client.
 * @param {string} res A URI that identifies the resource for which the token is valid.
 * @param {AcquireTokenCallback} callback The callback function.
 */
function getTokenFromCode(res, code, callback) {
    var authContext = new AuthenticationContext(credentials.authority);
    authContext.acquireTokenWithAuthorizationCode(code, credentials.redirect_uri, res, credentials.client_id, credentials.client_secret, function (err, response) {
        if (err) {
            callback(null);
        }
        else {
            callback(response);
        }
    });
};
```

<!--<a name="request"/>-->
## Senden einer Anforderung an die Microsoft Graph-API

Um unsere Anforderungen an die Graph-API zu identifizieren, müssen unsere Anforderungen mit einem `Authorization`-Header signiert werden, der das Zugriffstoken für alle angeforderten Webdienstressourcen enthält. Ein richtig formatierter Autorisierungsheader enthält das Zugriffstoken aus adal-node und hat das folgende Format.

`Authorization: Bearer <access token>`

Mit `adal-node` und unserer Authentifizierungslogik aus dem vorherigen Abschnitt können wir jetzt unser Zugriffstoken zum Signieren von Anforderungen verwenden.

```javascript
/* GET home page. */
router.get('/<application reply url>', function (req, res, next) {
    var authCode = req.query.code;
    authHelper.getTokenFromCode('https://graph.microsoft.com/', req.query.code, function (token) {
        if (token !== null) {
            // Use this token to sign requests
            var headers = {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer ' + token
                };
            // request implementation...
        } else {
            // error handling
        }
    });
});
```

Microsoft Graph ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit der Microsoft Graph-API finden Sie in der [API-Referenz](http://graph.microsoft.io/docs/api-reference/v1.0).

<!--## Additional resources

- [Office 365 Node.js Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Nodejs-Unified-API-Connect)-->

