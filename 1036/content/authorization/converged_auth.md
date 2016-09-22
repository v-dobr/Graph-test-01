# Authentification des points de terminaison Microsoft Graph à l’aide du point de terminaison d’authentification v2


<!--
### Preview documentation
There are features and functionality of the converged authentication model that are not yet supported in the public preview period. You should be aware of them if you are building applications during the public preview. For more information, see [Limitations and restrictions of the converged authentication model preview](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).
-->

## Connexion d’utilisateurs de comptes Microsoft et Azure AD avec un modèle d’authentification unique

Le point de terminaison d’authentification v2 vous permet de créer des applications qui acceptent aussi bien les identités professionnelles et scolaires (Azure AD) que les identités personnelles (compte Microsoft).

Dans le passé, si vous souhaitiez développer une application pour prendre en charge les comptes Microsoft et Azure Active Directory, vous deviez vous intégrer à deux systèmes totalement distincts. Le point de terminaison d’authentification v2 vous permet désormais de prendre en charge les deux types de comptes avec une seule intégration. Vous disposez d’un processus simple pour toucher immédiatement un public couvrant des millions d’utilisateurs avec des comptes à la fois personnels et professionnels/scolaires.   

Une fois que vous intégrez vos applications avec le point de terminaison d’authentification v2, celles-ci peuvent immédiatement accéder aux points de terminaison Microsoft Graph disponibles pour les comptes à la fois personnels et professionnels/scolaires, tels que : 

| Données              | Point de terminaison                                       |
|:------------------|:-----------------------------------------------|
| Profil utilisateur      | `https://graph.microsoft.com/v1.0/me`          |
| Messagerie Outlook      | `https://graph.microsoft.com/v1.0/me/messages` |
| Contacts Outlook  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Calendriers Outlook | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**Remarque :** certains points de terminaison Microsoft Graph tels que les groupes et les tâches ne sont pas applicables aux comptes personnels.  

## Étendues d’authentification de l’API Microsoft Graph

Le point de terminaison d’authentification v2 prend en charge toutes les étendues d’autorisations répertoriées pour une utilisation avec l’authentification Azure AD dans la rubrique relative aux [étendues d’autorisation de Microsoft Graph](permission_scopes.md). Toutefois, le point de terminaison d’authentification v2 ne prend pas en charge actuellement les étendues d’application uniquement.

>**Remarque :** actuellement, il est nécessaire de transmettre l’URL de ressource « https://graph.microsoft.com » comme préfixe pour la chaîne de l’étendue. Par exemple, pour utiliser l’étendue `Files.Read`, spécifiez l’étendue en tant que `https://graph.microsoft.com/Files.Read`.

Pour plus d’informations sur l’utilisation d’étendues avec le point de terminaison d’authentification v2 et la façon dont elle diffère de l’utilisation de ressources dans Azure AD, voir [Des étendues, pas des ressources](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources).

<!--
The table below lists the authentication scopes to use with the converged authentication model preview. For more information about using scopes with the converged authentication model, and how it differs from using resources in Azure AD, see [Scopes, not resources](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources).


| **Scope**             | **Permission**                        | **Description**                                                                                                                                         |
|:----------------------|:--------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `User.Read`           | Enable sign-in and read user profile  | Allows users to sign-in to the app, and allows the app to read the profile. It also allow the app to read basic company information of signed-in users. |
| `User.ReadWrite`      | Read and write access to user profile | Allows the app to read the profile of signed-in users, and to update profile information on behalf of signed-in users.                                  |
| `Mail.Read`           | Read user mail                        | Allows this app to read messages in user mailboxes.                                                                                                     |
| `Mail.ReadWrite`      | Read and write access to user mail    | Allows the app to read, update, create, and delete messages in user mailboxes.                                                                          |
| `Mail.Send`           | Send mail as a user                   | Allows the app to send messages as users in the organization.                                                                                           |
| `Contacts.Read`       | Read user contacts                    | Allows the app to read user contacts.                                                                                                                   |
| `Contacts.ReadWrite`  | Have full access to user contacts     | Allows the app to read, update, create and delete user contacts.                                                                                        |
| `Calendars.Read`      | Read user calendars                   | Allows the app to read events in user calendars.                                                                                                        |
| `Calendars.ReadWrite` | Have full access to user calendars    | Allows the app to read, update, create, and delete events in user calendars.                                                                            |
| `Files.Read`          | Read users' files                     | Allows the application to read the current user's files.                                                                                                |
| `Files.ReadWrite`     | Edit or delete users' files           | Allows the app to edit or delete the current user's files.                                                                                              |
| `openid`              | Sign users in                         | Allows users to sign in to the app and allows the app to see basic user profile information.                                                            |
| `offline_access`      | Read and write user's information     | Allows the app to see and update user's data, even when the user is not actively using the app.                                                         |

**Note**: currently it is required to pass the resource url of 'https://graph.microsoft.com' as prefix for the scope string. For example, to use the `Files.Read` scope you would specify the scope as `https://graph.microsoft.com/Files.Read`.
-->


## Étapes suivantes

[Inscription d’une application avec le point de terminaison v2](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/)

## En savoir plus

[Quelles sont les particularités du point de terminaison v2](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare)

[Dois-je utiliser le point de terminaison v2 ?](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

[Documentation relative au point de terminaison d’authentification Microsoft Azure v2](https://azure.microsoft.com/en-us/documentation/articles/?service=active-directory&term=app+model+v2.0)
