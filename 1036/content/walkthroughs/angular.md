# Appel de Microsoft Graph dans une application Angular 

Dans cet article, nous examinons les tâches minimales requises pour connecter votre application à Office 365 et appeler l’API Microsoft Graph. Nous utilisons le code de l’[exemple de connexion d’une application Angular à Office 365 à l’aide de Microsoft Graph](https://github.com/microsoftgraph/angular-connect-rest-sample) pour expliquer les concepts principaux que vous devez mettre en œuvre dans votre application.

![Capture d’écran d’un exemple de connexion d’une application Angular à Office 365](./images/web-screenshot.png)

## Conditions préalables  

Cette rubrique suppose que la condition suivante est satisfaite.

* Vous êtes à l’aise avec la lecture de code JavaScript et [AngularJS](https://angularjs.org/).

## Vue d’ensemble

Pour appeler l’API Microsoft Graph, vous devez effectuer les tâches suivantes.

1. Inscription de l’application dans Azure Active Directory
2. Configuration d’Azure Active Directory Library pour JavaScript (ADAL JS)
3. Utilisation d’ADAL JS pour obtenir un jeton d’accès
4. Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

<!--<a name="register"></a>-->
## Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application pour accéder au compte professionnel ou scolaire d’un utilisateur à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Pour la gérer, vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com)

Vous pouvez également consulter l’article relatif à l’[inscription de votre application web basée sur le navigateur auprès du Portail de gestion Azure Management](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp) pour obtenir des instructions sur la façon d’inscrire manuellement l’application. Gardez à l’esprit les informations suivantes :

* Vérifiez que vous spécifiez http://127.0.0.1:8080/ comme **URL d’authentification**.
* Une fois que vous avez inscrit l’application, [configurez les **autorisations déléguées**](https://github.com/microsoftgraph/angular-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) requises par votre application Angular. L’exemple de connexion requiert l’autorisation d’**envoi de messages en tant qu’utilisateur connecté**.

Prenez note des valeurs suivantes dans la page **Configure** de votre application Azure car ces valeurs sont requises pour configurer [ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) dans votre application Angular.

* ID client (unique pour votre application)
* URL de réponse (http://127.0.0.1:8080/)

<!--<a name="adal"></a>-->
## Configuration d’Azure Active Directory Library pour JavaScript (ADAL JS)

[ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) est une bibliothèque JavaScript qui fournit une prise en charge complète pour la connexion des utilisateurs d’Azure AD dans des applications web monopage (SPA), telles que l’exemple de connexion et la gestion des jetons, ainsi que d’autres fonctionnalités. Pour tirer parti de cette bibliothèque, votre application Angular doit l’inclure et la configurer.

Incluez simplement la bibliothèque et son module propre à Angular à l’aide de la fonctionnalité de réseau de distribution de contenu (CDN) Microsoft.

```html
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal.min.js"></script>
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal-angular.min.js"></script>
```

Ensuite, vous devez configurer le service ADAL JS là où vous configurez les dépendances de votre application Angular. L’exemple de connexion effectue sa configuration dans [*public/app.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/scripts/app.js). 

Pour configurer ADAL JS, incluez d’abord une référence au module ADAL en ajoutant ```AdalAngular``` au tableau requis de votre module et transférez ```adalAuthenticationServiceProvider``` à votre fonction ```config```. Configurez la bibliothèque avec la fonction ```init```, en lui transférant l’ID client de votre application et un objet ```endpoints``` qui déclare les API auxquelles votre application Angular doit effectuer des demandes CORS.

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
## Utilisation d’ADAL JS pour obtenir un jeton d’accès

Votre application doit rediriger le navigateur vers une page de connexion, pour que l’utilisateur puisse se connecter et octroyer à votre application l’accès à ses données. L’exemple de connexion utilise ADAL JS pour gérer cette tâche. 

Dans un des contrôleurs de votre application, ajoutez d’abord une référence au service ADAL en injectant ```adalAuthenticationService``` dans votre contrôleur, puis définissez une fonction qui utilise la fonction ```login``` du service que votre interface utilisateur peut appeler. L’exemple de connexion effectue cette opération dans le fichier [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js). 

```javascript
/**
  * Expose the login method from ADAL to the view.
  */
function connect() {
  adalAuthenticationService.login();
};
```

Lorsque cette fonction est appelée, votre application redirige l’utilisateur vers une page de connexion. Une fois qu’il se connecte et autorise votre application, il revient à votre application avec le jeton d’accès dans la chaîne de requête qu’ADAL récupérera et enregistrera. 

<!--<a name="request"></a>-->
## Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Avec un jeton d’accès, votre application peut effectuer des demandes authentifiés à l’API Microsoft Graph. ADAL JS intercepte automatiquement toutes les demandes HTTP et leur ajoute votre jeton d’accès pour que vous n’ayez pas à définir manuellement cet en-tête lors de l’utilisation de la bibliothèque. 

L’exemple de connexion envoie un courrier électronique à l’aide du point de terminaison ```me/sendMail``` de l’API Microsoft Graph dans le fichier [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js). 

Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la [référence d’API](http://graph.microsoft.io/docs/api-reference/v1.0) pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph.

