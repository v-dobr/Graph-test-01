# Appel de Microsoft Graph dans une application PHP 

Dans cet article, nous examinons les tâches minimales requises pour obtenir un jeton d’accès d’Azure Active Directory (AD) et appeler l’API Microsoft Graph. Nous utilisons le code de l’[exemple de connexion de PHP à Office 365 à l’aide de Microsoft Graph](https://github.com/microsoftgraph/php-connect-rest-sample) pour expliquer les concepts principaux que vous devez mettre en œuvre dans votre application.

![Capture d’écran d’un exemple de connexion de PHP à Office 365](./images/web-screenshot.png)

## Vue d’ensemble

Pour appeler l’API Microsoft Graph, votre application PHP doit effectuer les tâches suivantes.

1. Inscription de l’application dans Azure Active Directory
2. Redirection du navigateur vers la page de connexion
3. Réception d’un code d’autorisation dans la page de votre URL de réponse
4. Demande de jeton d’accès à partir du point de terminaison du jeton
5. Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

<!--<a name="register"/>-->
## Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application pour accéder au compte professionnel ou scolaire d’un utilisateur à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Pour la gérer, vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com)

Vous pouvez également consulter la section relative à l’[inscription de votre application de serveur web auprès du Portail de gestion Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) pour obtenir des instructions sur la façon d’inscrire manuellement l’application. Gardez à l’esprit les informations suivantes :

* Spécifiez une page dans votre application PHP comme **URL d’authentification** à l’étape 6. Dans le cas de l’exemple de connexion, cette page est [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php).
* [Configurez les **autorisations déléguées**](https://github.com/microsoftgraph/php-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) requises par votre application. L’exemple de connexion requiert l’autorisation d’**envoi de messages en tant qu’utilisateur connecté**.

Prenez note des valeurs suivantes dans la page de **configuration** de votre application Azure.

* ID du client
* Une clé valide
* URL de réponse

Ces valeurs sont nécessaires comme paramètres dans le flux OAuth dans votre application.

<!--<a name="redirect"/>-->
## Redirection du navigateur vers la page de connexion

Votre application doit rediriger le navigateur vers la page de connexion pour obtenir un code d’autorisation et poursuivre le flux OAuth.

Dans l’exemple de connexion, le code qui redirige le navigateur se trouve dans la fonction [`AuthenticationManager.connect`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L41).

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

> **Remarque :** <br />
> Vous devez envoyer l’en-tête **Location** avant d’écrire un résultat sur la page.

<!--<a name="authcode"/>-->
## Réception d’un code d’autorisation dans la page de votre URL de réponse

Une fois que l’utilisateur se connecte, le flux renvoie le navigateur à l’URL de réponse dans votre application. Azure ajoute un code d’autorisation dans la chaîne de requête. L’exemple de connexion utilise la page [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php) à cet effet.

Le code d’autorisation est fourni dans la variable de la chaîne de requête `code`. L’exemple de connexion enregistre le code dans une variable de session pour une utilisation ultérieure.

```php
if (isset($_GET['code'])) {
    $_SESSION['code'] =  $_GET['code'];
}
```

<!--<a name="accesstoken"/>-->
## Demande de jeton d’accès à partir du point de terminaison du jeton

Une fois que vous avez le code d’autorisation, vous pouvez l’utiliser avec l’ID client, la clé et les valeurs de l’URL de réponse obtenues d’Azure AD pour demander un jeton d’accès. 

> **Remarque :** <br />
> La demande doit également spécifier une ressource que nous essayons de consommer. Dans le cas de l’API Microsoft Graph, la valeur de la ressource est `https://graph.microsoft.com`.

L’exemple de connexion demande un jeton à l’aide de la fonction [`AuthenticationManager.acquireToken`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L62). Voici le code le plus pertinent.

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

> **Remarque :** <br />
> La réponse fournit davantage d’informations que le jeton d’accès uniquement, par exemple, votre application peut obtenir un jeton d’actualisation pour demander de nouveaux jetons d’accès sans que l’utilisateur se connecte explicitement.

Votre application PHP peut à présent utiliser la variable de session `access_token` pour émettre des demandes authentifiées à l’API Microsoft Graph.

<!--<a name="request"/>-->
## Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Avec un jeton d’accès, votre application peut effectuer des demandes authentifiées à l’API Microsoft Graph. Votre application doit fournir le jeton d’accès dans l’en-tête **Authorization** de chaque demande.

L’exemple de connexion envoie un message électronique à l’aide du point de terminaison **sendMail** dans l’API Microsoft Graph. Le code se trouve dans la fonction [`MailManager.sendWelcomeMail`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/MailManager.php#L40). Il s’agit du code qui montre comment envoyer le code d’accès dans l’en-tête Authorization.

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

> **Remarque :** <br />
> La demande doit également envoyer un en-tête **Content-Type** avec une valeur acceptée par l’API Microsoft Graph, par exemple, `application/json;odata.metadata=minimal;odata.streaming=true`.

L’API Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la référence API pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph.

<!--## Additional resources

-  [Office 365 PHP Connect sample using Microsoft Graph API](https://github.com/OfficeDev/O365-PHP-Unified-API-Connect)-->
