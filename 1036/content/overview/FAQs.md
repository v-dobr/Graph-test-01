
# Forum aux questions (FAQ) sur Microsoft Graph 

## Quelles sont les plateformes prises en charge par l’API Microsoft Graph ?
<!--
Apps can use the Microsoft Graph API to perform create, read, update, and delete (CRUD) operations on data sources and entities, giving them seamless access to work data. 

**Ease of use--one endpoint, all Office 365 data under one roof**

You can use the API in four steps:
1.  Select your programming language and development environment.
2.  Build your app.
3.  Optionally, host your app in Microsoft Azure or any cloud platform you choose.
4.  Authenticate your users by using single sign-on with Azure AD.

As a developer you can use the API to create custom apps that access and interact with all the richness of enterprise and productivity data--users, groups, organizational contacts, files, folders, mail, calendar, insights and relationships--and build apps across all mobile, web, and desktop platforms. No matter your development platform or tools. Using a single service endpoint to access those entities and data. And a single authentication flow.  -->

Vous pouvez effectuer les opérations suivantes :

<!--Just like in Office 365 APIs, Office 365 unified endpoint API  allows you to build apps using any development environment of your choice:  -->

- Utiliser n’importe quel environnement de développement que vous connaissez bien, tel que .NET, PHP, Java, Python ou Ruby
- Utiliser n’importe quel langage de programmation, une plateforme de développement et un environnement d’hébergement
- Créer une application qui accède à l’API à l’aide de n’importe quel langage web, notamment JavaScript, HTML5, Python, Ruby, PHP et ASP.NET  
- Utiliser l’EDI de votre choix, que ce soit Visual Studio, Eclipse, Android Studio, Xcode ou un autre élément de votre choix
- Héberger vos applications dans Microsoft Azure ou n’importe quelle plateforme de cloud
- Développer des applications pour Windows Universal, iOS, Android ou la plateforme d’un autre appareil
- Appeler l’API à partir de compléments Office (anciennement Applications pour Office) ou de compléments SharePoint (anciennement Applications pour SharePoint)
 


## Pourquoi utiliser l’API Microsoft Graph?

Supposons que vous souhaitiez récupérer par programmation les fichiers d’un utilisateur, l’image de profil, et rechercher le responsable de la dernière personne qui a modifié ce fichier dans votre organisation. Étant donné que les informations sont stockées dans plusieurs services (Azure Active Directory, SharePoint et Exchange) la tâche implique plusieurs étapes utilisant les API Office 365 : 

1. Utiliser le service de découverte pour rechercher les différents points de terminaison du service 
2. Déterminer l’URL des services auxquels vos applications Office 365 souhaitent se connecter
3. Acquérir et gérer le jeton d’accès pour chaque service et faire la demande au service directement

À présent, vous pouvez utiliser l’API Microsoft Graph API pour effectuer la même opération complexe via un point de terminaison API REST unique. Vous n’êtes pas obligé de découvrir et de parcourir un point de terminaison différent pour chaque service, acquérir et gérer un jeton d’accès distinct pour chaque service, gérer des services cloisonnés et différents modèle de données.

##Exemples de requêtes

L’exemple suivant décrit le modèle actuel d’interaction avec l’API Office 365 à l’aide de points de terminaison de service disparates et montre comme il devient beaucoup plus simple avec l’API Microsoft Graph.

**Points de terminaison de service disparates**

|   **Opération**                  |  **API**                          |  **Point de terminaison de service** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Découvrir les points de terminaison de service pour l’API Office 365               |     `Discovery Service`           | _https://_**api.office.com**_/discovery/v1.0/me/services_ |
| Obtenir les utilisateurs           |     `Azure AD Graph API` | _https://_**graph.windows.net**_/contoso.com/users?api-version=2013-04-05_|
| Obtenir la collection de messages de la boîte de réception       |     `Office 365 API`           | _https://_**outlook.office365.com**_/api/v1.0/me/messages_  |
| Obtenir les fichiers de Jean   |     `Office 365 API`  | _https://_**contoso-my.sharepoint.com**_/personal/joe_contoso_com/_api/v1.0/files_ |


Lorsque vous utilisez l’API Microsoft Graph, vous ne devez pas découvrir tout d’abord les points de terminaison de service puis traverser différents points de terminaison pour obtenir les fichiers d’un utilisateur, les messages et ainsi de suite. Vous devez interagir uniquement avec un espace de noms URL REST unique : _**graph.microsoft.com**_.

**API Microsoft Graph**

|   **Opération**                  |  **API**                          |  **Point de terminaison de service** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Découvrir les points de terminaison de service pour l’API Office 365                |     `Microsoft Graph`           | Inutile |
| Obtenir les utilisateurs           |     `Microsoft Graph` | _https://_**graph.microsoft.com**_/v1.0/contoso.onmicrosoft.com/users_ |
| Obtenir la collection de messages de la boîte de réception       |     `Microsoft Graph`           | _https://_**graph.microsoft.com**_/v1.0/me/messages_  |
| Obtenir les fichiers de Jean   |     `Microsoft Graph `  | _https://_**graph.microsoft.com**_/v1.0/me/drive/root/children_ |


## Avantages que présente l’utilisation de l’API Microsoft Graph

Voici quelques-uns des avantages que présente l’utilisation de l’API Microsoft Graph :

**Expérience développeur cohérente et simplifiée pour l’utilisation des services de cloud Microsoft**

-   Espace de noms unique pour tous les points de terminaison de service. Découverte de point de terminaison de service inutile.
-   Un seul jeton pour accéder à toutes les ressources
-   Navigation directe et intégrée directe entre les services actuellement cloisonnés (par exemple, obtenir la chaîne de service et de gestion de l’utilisateur qui a créé un document particulier)
-   Utilisation d’un seul ensemble d’API, c’est-à-dire utilisation de l’API Microsoft Graph uniquement pour se connecter à plusieurs services
-   Entités et API REST étendues et unifiées au sein de la plateforme Office 
-   Modèles et appellation de propriété cohérents à travers les entités, y compris les propriétés de navigation entre entités

**Activation d’expériences de vie professionnelle plus productives à l’aide d’Office**

-   Contenu contextuel. Par exemple, rechercher des documents par groupe, projet, équipe ou tendances
-   Relations utilisateur contextuelles. Par exemple, vous pouvez trouver des utilisateurs par appartenance à un groupe, intérêt, compétences et savoir-faire.  Vous pouvez également obtenir des relations d’organigramme

**Activation d’applications de développement dans un langage de programmation sur n’importe quelle plateforme**

-   Ressources et outils de développement pour tous les développeurs. Vous pouvez développer à l’aide de n’importe quelle plateforme et n’importe quel langage 
-   Développement mobile pour toutes les plateformes utilisant des technologies ouvertes  
-   Connaissances spécialisées d’Exchange, SharePoint ou Azure AD inutiles pour accéder aux entités de l’API Microsoft Graph

<!---<a name="msg_v2auth"> </a>-->

## L’API Microsoft Graph prend-elle en charge le point de terminaison d’authentification d’application v2.0 ?

Oui. Pour plus d’informations, voir la page relative à l’[authentification des points de terminaison Microsoft Graph à l’aide du point de terminaison d’authentification v2](http://graph.microsoft.io/docs/authorization/converged_auth).



  > Votre avis compte beaucoup pour nous. Communiquez avec nous sur [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Posez vos questions avec les tags [MicrosoftGraph] et [Office 365].








