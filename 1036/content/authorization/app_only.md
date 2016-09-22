# Appel Microsoft Graph dans une application de service ou de démon

Dans cet article, nous examinons les tâches minimales requises pour connecter votre service à client unique ou votre application de démon à Office 365 et appeler l’API Microsoft Graph.

## Vue d’ensemble

Pour appeler l’API Microsoft Graph dans une application de service ou de démon, vous devez effectuer les tâches suivantes.

1. Inscription de l’application dans Azure Active Directory.
2. Demande de jeton d’accès à partir du point de terminaison émettant le jeton.
3. Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph.

## Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com) pour la gérer.

Vous pouvez également consulter la section relative à l’[inscription de votre application de serveur web auprès du Portail de gestion Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) pour obtenir des instructions sur la façon d’inscrire manuellement l’application. Gardez à l’esprit les informations suivantes :

* Une fois que vous avez inscrit l’application, configurez les **autorisations de l’application** requises par votre application de service ou de démon.

Prenez note des valeurs suivantes dans la page de configuration de votre application Azure, car ces valeurs sont requises pour configurer le flux OAuth dans votre application de service ou de démon.

* ID client (unique pour votre application)
* Une clé d’application (unique pour votre application)
* Le point de terminaison du jeton OAuth 2.0 de votre application
  * Recherchez cette valeur en cliquant sur *Points de terminaison* en bas du portail de gestion Azure dans la page de votre application. Le point de terminaison ressemblera à `https://login.microsoftonline.com/<tenantId>/oauth2/token`.

## Demande de jeton d’accès à partir du point de terminaison émettant le jeton

Contrairement aux applications client, votre application de service ou de démon ne peut pas avoir un utilisateur qui se connecte et qui autorise votre application. À la place, votre application doit implémenter le flux d’octroi d’informations d’identification du client OAuth 2.0 qui lui permet d’utiliser ses propres informations d’identification, son ID client et une clé d’application pour l’authentification lors de l’appel de Microsoft Graph au lieu d’usurper l’identité d’un utilisateur. Reportez-vous à [Appels de service à service à l’aide d’informations d’identification de client](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx) pour plus d’informations sur la procédure d’authentification.

Effectuez une demande HTTP POST au point de terminaison émettant le jeton avec les paramètres suivants, en remplaçant `<clientId>` et `<clientSecret>` par l’ID client et la clé de votre application, respectivement.

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

La réponse inclura un jeton d’accès et des informations sur l’expiration.

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Avec un jeton d’accès, votre application peut effectuer des demandes authentifiées à l’API Microsoft Graph. Votre application doit ajouter le jeton d’accès à l’en-tête **Authorization** de chaque demande.

Par exemple, une application de service ou de démon peut récupérer tous les utilisateurs d’un client si elle dispose de l’autorisation de *lecture des profils complets de tous les utilisateurs* sélectionnée dans le portail de gestion Azure. 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```

Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la [référence d’API](http://graph.microsoft.io/docs/api-reference/v1.0) pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph.
