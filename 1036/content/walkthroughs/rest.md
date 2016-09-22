# Prise en main de Microsoft Graph et REST

Cet article décrit comment appeler Microsoft Graph pour récupérer les messages électroniques dans Office 365 et Outlook.com. Cet article se concentre sur les demandes et les réponses OAuth et REST. Il décrira la séquence des demandes et réponses qu’une application utilise pour authentifier et récupérer des messages.

## Utilisation d’OAuth 2.0 pour l’authentification

Pour appeler Microsoft Graph, votre application a besoin d’un jeton d’accès d’Azure Active Directory (Azure AD). Dans l’exemple suivant, l’application implémente le flux d’octroi de code d’autorisation pour obtenir les jetons d’accès d’Azure AD, en suivant les [protocoles OAuth 2.0](http://tools.ietf.org/html/rfc6749) standard.

### Inscription d’une application

Il existe actuellement deux options pour inscrire votre application :

  1. Inscrire une application à l’aide du modèle qui prend en charge les comptes professionnels ou scolaires et les comptes d’utilisateurs commerciaux d’Office 365.
 
  Ce modèle fonctionne uniquement avec des offres commerciales Office 365. Une fois que vous avez inscrit votre application, vous pouvez la gérer par le biais du [Portail de gestion Azure](https://manage.windowsazure.com).

  2. Inscrire l’application à l’aide de la toute dernière fonctionnalité, qui fonctionne pour les services grand public et commerciaux d’Office 365 (nous l’appelons point de terminaison d’authentification v2.0).
 
  Un service d’authentification unique pour les comptes scolaires ou professionnels et les comptes personnels est désormais disponible. Ce modèle fournit un service d’authentification unique à la fois pour les identités professionnelles et scolaires (Azure AD) et les identités personnelles (Microsoft). Désormais, il vous suffit d’implémenter un flux d’authentification dans votre application pour permettre aux utilisateurs d’utiliser des comptes scolaires ou professionnels, tels qu’Office 365 ou OneDrive for Business, ou des comptes personnels, tels qu’Outlook.com ou OneDrive.
   
Utilisez le [portail d’inscription des applications](https://apps.dev.microsoft.com/) pour inscrire votre application et configurer à la fois des comptes personnels et des comptes scolaires ou professionnels.

Veuillez noter que le point de terminaison version 2.0 est développé progressivement de manière à couvrir toutes les configurations prises en charge par le point de terminaison d’authentification précédent. Pour déterminer si cette version est adaptée à votre cas, consultez [cet article](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).

Après l’inscription, vous obtenez un ID client et un code secret. Ces valeurs sont utilisées dans le flux d’octroi de code d’autorisation.

Le reste de ce document suppose que votre inscription suit le modèle de version 2.0. Pour obtenir un guide complet des flux pris en charge par le point de terminaison de version 2.0, consultez [cet article](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/). Pour obtenir un guide complet sur la procédure d’octroi de code d’autorisation, consultez [cet article](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/)

### Obtention d’un code d’autorisation

La première étape dans le flux d’octroi de code d’autorisation consiste à obtenir un code d’autorisation. Ce code est renvoyé à l’application par le serveur d’autorisation lorsque l’utilisateur se connecte et accepte le niveau d’accès requis par l’application.

Tout d’abord, l’application crée une URL de connexion pour l’utilisateur. Cette URL doit être ouverte dans un navigateur pour que l’utilisateur puisse se connecter et fournir son consentement.

L’URL de base pour la connexion ressemble à `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`.

L’application ajoute des paramètres de requête à cette URL de base pour indiquer au serveur d'authentification quelle application demande la connexion et quelles autorisations elle requiert.

- `client_id` - ID client généré en inscrivant l’application. Il permet à Azure AD de savoir quelle application demande la connexion.
- `redirect_uri` - Emplacement vers lequel Azure redirige l’utilisateur une fois qu’il a accordé son consentement à l’application. Cette valeur doit correspondre à la valeur de l’**URI de redirection** utilisé lors de l’inscription de l’application.
- `response_type` - Type de réponse attendu par l’application. Cette valeur est `code` pour la procédure d’octroi de code d’autorisation.
- `scope` - Liste des étendues demandées par l’application, séparées par des espaces. Pour plus d’informations, consultez [cet article](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/).
- `state` - Valeur incluse dans la demande qui est également renvoyée dans la réponse du jeton.

Par exemple, l’URL de requête pour notre application nécessitant un accès en lecture au courrier électronique se présente comme suit.

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<CLIENT ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
```

Ensuite, redirigez l’utilisateur vers l’URL de connexion. L’utilisateur voit apparaître un écran de connexion qui affiche le nom de l’application. Une fois connecté, l’utilisateur voit apparaître une liste des autorisations requises par l’application. Il est invité à l’autoriser ou la refuser. En supposant que l’accès requis soit autorisé, le navigateur sera redirigé vers l’URI de redirection spécifié dans la demande initiale avec le code d’autorisation dans la chaîne de requête.

```http
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

Si vous utilisez également OpenId Connect pour l’authentification unique, d’autres paramètres sont nécessaires ; pour plus d’informations, consultez [cet article](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/). 

L’étape suivante consiste à échanger le code d’autorisation renvoyé pour un jeton d’accès.

### Obtention d’un jeton d’accès

Pour obtenir un jeton d’accès, l’application publie des paramètres encodés dans l’URL de demande de jeton (`https://login.microsoftonline.com/common/oauth2/v2.0/token`) avec les paramètres suivants.

- `client_id` : L’ID client généré en inscrivant l’application..
- `client_secret` : Le code secret client généré en inscrivant l’application.
- `code` : Le code d’autorisation obtenu à l’étape précédente.
- `redirect_uri` : Cette valeur doit être identique à la valeur utilisée dans la demande de code d’autorisation.
- `grant_type` : Le type d’octroi utilisé par l’application. Cette valeur est `code` pour la procédure d’octroi de code d’autorisation.
- `scope` - Liste des étendues demandées par l’application, séparées par des espaces. Pour plus d’informations, consultez [cet article](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/).

L’URL de requête pour notre application, en utilisant le code de l’étape précédente, se présente comme suit.

```http
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
  &client\_id=<CLIENT ID>
  &client\_secret=<CLIENT SECRET>
}
```

Le serveur répond avec une charge JSON qui inclut le jeton d’accès.

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

Le jeton d’accès se trouve dans le champ `access_token` de la charge JSON. L’application utilise cette valeur pour définir l’en-tête Authorization lors de l’exécution d’appels REST à l’API.

## Appel de Microsoft Graph

Une fois que l’application a un jeton d’accès, elle peut appeler Microsoft Graph. Étant donné que cet exemple d’application récupère des messages, elle utilisera une requête HTTP GET au point de terminaison `https://graph.microsoft.com/v1.0/me/messages`.

### Amélioration de la requête

Les applications peuvent contrôler le comportement des requêtes GET à l’aide de paramètres de requête OData. Il est recommandé que les applications utilisent ces paramètres afin de limiter le nombre de résultats renvoyés et les champs renvoyés pour chaque élément. 

Cet exemple d’application affichera les messages dans un tableau qui indiquera l’objet, l’expéditeur et la date et l’heure de réception du message. Le tableau affiche un maximum de 25 lignes et est trié de sorte que le message reçu en dernier apparaît en haut. L’application utilise les paramètres de requête suivant pour obtenir ces résultats.

- `$select` - Ne spécifie que les champs `subject`, `sender`, et `dateTimeReceived`.
- `$top` - Spécifie un maximum de 25 éléments.
- `$orderby` - Trie les résultats en fonction du champ `dateTimeReceived`.

Cela entraîne la demande suivante.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

Maintenant que vous avez vu comment effectuer des appels à Microsoft Graph, vous pouvez utiliser la référence d’API pour créer tous les autres types d’appels que votre application doit effectuer. Toutefois, gardez à l’esprit que votre application doit disposer des autorisations appropriées configurées lors de l’inscription de l’application pour les appels qu’elle effectue.


