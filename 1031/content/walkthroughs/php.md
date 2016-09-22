# Aufrufen von Microsoft Graph in einer PHP-App 

In diesem Artikel werden Aufgaben beschrieben, die zum Abrufen eines Zugriffstokens aus Azure Active Directory (AD) und Aufrufen der Microsoft Graph-API mindestens erforderlich sind. Wir verwenden den Code aus dem [Office 365 PHP Connect-Beispiel unter Verwendung von Microsoft Graph](https://github.com/microsoftgraph/php-connect-rest-sample), um die wichtigsten Konzepte zu erläutern, die Sie in Ihrer App implementieren müssen.

![Screenshot des Office 365 PHP Connect-Beispiels](./images/web-screenshot.png)

## Übersicht

Zum Aufrufen der Microsoft Graph-API muss Ihre PHP-App die folgenden Aufgaben ausführen:

1. Registrieren der Anwendung in Azure Active Directory
2. Weiterleiten des Browsers zur Anmeldeseite
3. Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite
4. Anfordern eines Zugriffstokens vom Tokenendpunkt
5. Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

<!--<a name="register"/>-->
## Registrieren der Anwendung in Azure Active Directory

Bevor Sie  mit Office 365 arbeiten können, müssen Sie Ihre Anwendung registrieren und die Berechtigungen für die Verwendung der Microsoft Graph-Dienste festlegen. 
Mithilfe des [App-Registrierungstools](https://dev.office.com/app-registration) können Sie mit nur wenigen Klicks Ihre Anwendung für den Zugriff auf ein Geschäfts- oder Schulkonto des Benutzers registrieren. Zum Verwalten müssen Sie das [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) verwenden.

Alternativ können Sie die App manuell registrieren. Die Anweisungen dazu finden Sie im Abschnitt [Registrieren der Webserver-App mithilfe des Azure-Verwaltungsportals](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp). Beachten Sie dabei Folgendes:

* Geben Sie eine Seite in Ihrer PHP-App als die **Anmelde-URL** in Schritt 6 an. Im Falle des Connect-Beispiels lautet diese Seite [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php).
* [Konfigurieren Sie die **delegierten Berechtigungen**](https://github.com/microsoftgraph/php-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), die für Ihre App erforderlich sind. Für das Connect-Beispiel ist die Berechtigung zum **Senden von E-Mails als angemeldeter Benutzer** erforderlich.

Notieren Sie sich die folgenden Werte auf der Seite **Konfigurieren** Ihrer Azure-Anwendung.

* Client-ID
* Ein gültiger Schlüssel
* Eine Antwort-URL

Sie benötigen diese Werte als Parameter im OAuth-Fluss in Ihrer App.

<!--<a name="redirect"/>-->
## Weiterleiten des Browsers zur Anmeldeseite

Die App muss den Browser an die Anmeldeseite weiterleiten, um einen Autorisierungscode zu erhalten und den OAuth-Fluss fortzusetzen.

Im Connect-Beispiel ist der Code zum Umleiten des Browsers in der Funktion [`AuthenticationManager.connect`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L41) enthalten.

```php
// Redirect the browser to the authorization endpoint. Auth endpoint is
// https://login.microsoftonline.com/common/oauth2/authorize
$redirect = Constants::AUTHORITY_URL . Constants::AUTHORIZE_ENDPOINT . 
            '?response_type=code' . 
            '&client_id=' . urlencode(Constants::CLIENT_ID) . 
            '&redirect_uri=' . urlencode(Constants::REDIRECT_URI);
header("Location: {$redirect}");
exit();
```

> **Hinweis:** <br />
> Sie müssen den Header **Adresse** senden, bevor Sie eine Ausgabe auf die Seite schreiben.

<!--<a name="authcode"/>-->
## Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite

Nachdem der Benutzer sich angemeldet hat, gibt der Fluss den Browser an die Antwort-URL in Ihrer App zurück. Azure fügt der Abfragezeichenfolge einen Autorisierungscode hinzu. Im Connect-Beispiel wird die [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php)-Seite für diesen Zweck verwendet.

Der Autorisierungscode ist in der `code`-Variablen für die Abfragezeichenfolge verfügbar. Das Connect-Beispiel speichert den Code in einer Sitzungsvariable für die spätere Verwendung.

```php
if (isset($_GET['code'])) {
    $_SESSION['code'] =  $_GET['code'];
}
```

<!--<a name="accesstoken"/>-->
## Anfordern eines Zugriffstokens vom Tokenendpunkt

Nachdem Sie den Autorisierungscode erhalten haben, können Sie diesen zusammen mit den Werten für die Client-ID, den Schlüssel und die Antwort-URL verwenden, die Sie von Azure AD zum Anfordern eines Zugriffstokens erhalten haben. 

> **Hinweis:** <br />
> Außerdem muss die Anforderung eine Ressource angeben, die wir nutzen möchten. Im Falle der Microsoft Graph-API ist der Ressourcenwert `https://graph.microsoft.com`.

Das Connect-Beispiel fordert ein Token mithilfe des Codes in der Funktion [`AuthenticationManager.acquireToken`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L62) an. Dies ist der wichtigste Code.

```php
$tokenEndpoint = Constants::AUTHORITY_URL . Constants::TOKEN_ENDPOINT;

// Send a POST request to the token endpoint to retrieve tokens.
// Token endpoint is:
// https://login.microsoftonline.com/common/oauth2/token
$response = RequestManager::sendPostRequest(
    $tokenEndpoint, 
    array(),
    array(
        'client_id' => Constants::CLIENT_ID,
        'client_secret' => Constants::CLIENT_SECRET,
        'code' => $_SESSION['code'],
        'grant_type' => 'authorization_code',
        'redirect_uri' => Constants::REDIRECT_URI,
        'resource' => Constants::RESOURCE_ID
    )

// Store the raw response in JSON format.
$jsonResponse = json_decode($response, true);

// The access token response has the following parameters:
// access_token - The requested access token.
// expires_in - How long the access token is valid.
// expires_on - The time when the access token expires.
// id_token - An unsigned JSON Web Token (JWT).
// refresh_token - An OAuth 2.0 refresh token.
// resource - The App ID URI of the web API (secured resource).
// scope - Impersonation permissions granted to the client application.
// token_type - Indicates the token type value.
foreach ($jsonResponse as $key=>$value) {
    $_SESSION[$key] = $value;
}
```

> **Hinweis:** <br />
> Die Antwort bietet mehr Informationen als nur das Zugriffstoken. Beispielsweise kann Ihre App ein Aktualisierungstoken abrufen, um neue Zugriffstoken anzufordern, ohne dass sich der Benutzer explizit anmelden muss.

Ihre PHP-App kann mittlerweile die Sitzungsvariable `access_token` nutzen, um authentifizierte Anforderungen für die Microsoft Graph-API auszugeben.

<!--<a name="request"/>-->
## Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

Mit einem Zugriffstoken kann Ihre App authentifizierte Anforderungen an die Microsoft Graph-API senden. Ihre App muss das Zugriffstoken an den **Autorisierungs**-Header jeder Anforderung anfügen.

Im Connect-Beispiel wird eine E-Mail mithilfe des **sendMail**-Endpunkts in der Microsoft Graph-API gesendet. Der Code ist in der Funktion [`MailManager.sendWelcomeMail`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/MailManager.php#L40) enthalten. Dies ist der Code, der zeigt, wie der Zugriffscode im Header „Autorisierung“ gesendet wird.

```php
// Send the email request to the sendmail endpoint, 
// which is in the following URI:
// https://graph.microsoft.com/v1.0/me/microsoft.graph.sendMail
// Note that the access token is attached in the Authorization header
RequestManager::sendPostRequest(
    Constants::RESOURCE_ID . Constants::SENDMAIL_ENDPOINT,
    array(
        'Authorization: Bearer ' . $_SESSION['access_token'],
        'Content-Type: application/json;' . 
                      'odata.metadata=minimal;' .
                      'odata.streaming=true'
    ),
    $email
);
```

> **Hinweis:** <br />
> Die Anforderung muss auch einen Header des Typs **Content-Type** mit einem Wert senden, der von der Microsoft Graph-API, z. B. `application/json;odata.metadata=minimal;odata.streaming=true`, akzeptiert wird.

Die Microsoft Graph-API ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit der Microsoft Graph-API finden Sie in der API-Referenz.

<!--## Additional resources

-  [Office 365 PHP Connect sample using Microsoft Graph API](https://github.com/OfficeDev/O365-PHP-Unified-API-Connect)-->
