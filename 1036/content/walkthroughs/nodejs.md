# Appel de Microsoft Graph avec une application Node.js

Dans cet article, nous examinons les tâches minimales requises pour connecter votre application à Office 365 et appeler l’API Microsoft Graph. Nous utilisons le code de l’[exemple de connexion d’une application Node.js à Office 365 à l’aide de Microsoft Graph](https://github.com/microsoftgraph/nodejs-connect-rest-sample) pour expliquer les concepts principaux que vous devez mettre en œuvre dans votre application.

![Capture d’écran d’un exemple de connexion de Node.js à Office 365](./images/web-screenshot.png)

## Vue d’ensemble

Pour appeler l’API Microsoft Graph, votre application web doit effectuer les tâches suivantes.

1. Inscription de l’application dans Azure Active Directory 
2. Installation de la bibliothèque cliente Azure Active Directory pour Node
3. Redirection du navigateur vers la page de connexion
4. Réception d’un code d’autorisation dans la page de votre URL de réponse
5. Utilisation de `adal-node` pour demander un jeton d’accès
6. Demande à l’API Microsoft Graph

<!--<a name="register"/>-->
## Inscription de votre application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application pour accéder au compte professionnel ou scolaire d’un utilisateur à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Pour la gérer, vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com)

Vous pouvez également consulter la section relative à l’[inscription de votre application de serveur web auprès du Portail de gestion Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) pour obtenir des instructions sur la façon d’inscrire manuellement l’application. Gardez à l’esprit les informations suivantes :

* Spécifiez une page dans votre application Node.js comme **URL d’authentification** à l’étape 6. Dans le cas de l’exemple de connexion, l’URL est http://localhost:8080/login, qui correspond au routage [/login](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L33).
* [Configurez les **autorisations déléguées**](https://github.com/microsoftgraph/nodejs-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) requises par votre application. L’exemple de connexion requiert l’autorisation d’**envoi de messages en tant qu’utilisateur connecté**.

Prenez note des valeurs suivantes dans la page de **configuration** de votre application Azure.

* ID du client
* Une clé valide
* URL de réponse

Ces valeurs sont nécessaires comme paramètres dans le flux OAuth dans votre application.

<!--<a name="adal">-->
## Installation de la bibliothèque cliente Azure Active Directory pour Node

La bibliothèque ADAL pour Node.js facilite l’authentification pour les applications Node.js dans AAD pour accéder aux ressources web protégées AAD.
Pour ajouter adal-node à votre `package.json` existant, saisissez les informations suivantes dans votre terminal préféré.

`npm install adal-node --save`

Pour plus d’informations sur la bibliothèque cliente adal-node, consultez les informations de son package sur [npm](https://www.npmjs.com/package/adal-node). 
Pour des informations sur les problèmes, le code source et les correctifs et fonctionnalités à venir, consultez le projet d’adal-node sur [Github](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs).

<!--<a name="redirect"/>-->
## Redirection du navigateur vers la page de connexion

Votre application doit rediriger le navigateur vers la page de connexion pour obtenir un code d’autorisation et poursuivre le flux OAuth 2.0.

Dans l’exemple de connexion, l’URL d’authentification de [`authHelper.js#getAuthUrl`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js#L17) est redirigée par la fonction [`login.hbs#login`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/views/login.hbs#L2) via un événement `onclick` côté client .

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
## Réception d’un code d’autorisation dans la page de votre URL de réponse

Une fois que l’utilisateur se connecte, le flux renvoie le navigateur à l’URL de réponse dans votre application. Le code d’autorisation est fourni dans la variable de chaîne de requête `code`.

```javascript
router.get('/<application reply url>', function (req, res, next) {
  var authCode = req.query.code;
  // your router's implementation
});
```

Voir le [code correspondant](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L34) dans l’exemple de connexion

<!--<a name="accesstoken"/>-->
## Utiliser `adal-node` pour demander un jeton d’accès

Maintenant que nous avons été authentifié via Azure Active Directory, l’étape suivante consiste à acquérir un jeton d’accès via adal-node. Une fois cela fait, nous pourrons effectuer des requêtes REST à l’API Microsoft Graph.

Pour demander un jeton d’accès, adal-node fournit deux fonctions de rappel.

|                          Fonction                         |                                      Paramètres                                      | Description                                                                                             |
|:---------------------------------------------------------:|:--------------------------------------------------------------------------------:|---------------------------------------------------------------------------------------------------------|
| `AuthenticationContext.acquireTokenWithAuthorizationCode` | `authCode`, `redirect_uri`, `resource`, `client_id`, `client_secret`, `callback` | fournit un jeton d’accès pour une ressource spécifiée, en fonction du code d’autorisation renvoyé lors de la connexion |
| `AuthenticationContext.acquireTokenWithRefreshToken`      | `token`, `client_id`, `client_secret`, `resource`, `callback`                    | fournit un jeton d’accès pour une ressource spécifiée, en fonction d’un jeton d’actualisation                             |

Dans l’exemple de connexion, les demandes sont acheminées via [`authHelper.js`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js) afin que les `client_id` et `client_secret` puissent être ajoutés.

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
## Demande à l’API Microsoft Graph

Pour identifier nos demandes à l’API Graph, elles doivent être signées avec un en-tête `Authorization` contenant le jeton d’accès de toute ressource de service web que nous demandons. Un en-tête d’autorisation correctement formé inclura le jeton d’accès d’adal-node et aura la forme suivante.

`Authorization: Bearer <access token>`

À l’aide de `adal-node`, combiné avec notre logique d’authentification de la section précédente, nous pouvons maintenant utiliser notre jeton d’accès pour signer des demandes.

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

Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la [référence d’API](http://graph.microsoft.io/docs/api-reference/v1.0) pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph.

<!--## Additional resources

- [Office 365 Node.js Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Nodejs-Unified-API-Connect)-->

