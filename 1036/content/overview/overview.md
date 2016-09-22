


# Présentation de Microsoft Graph

Microsoft Graph (précédemment appelé API unifiée Office 365) expose plusieurs API des services cloud Microsoft via un seul point de terminaison API REST (**https://graph.microsoft.com**). Microsoft Graph vous permet de transformer des requêtes auparavant difficiles ou complexes en simples navigations. 
 
Microsoft Graph offre :

- Un point de terminaison d’API unifiée pour accéder aux données agrégées de plusieurs services cloud Microsoft dans une seule réponse 
- Navigation transparente entre entités et relations 
- Accès aux analyses et aux renseignements du cloud Microsoft

Et tout cela à l’aide d’un jeton d’authentification unique.

Vous pouvez utiliser l’API pour accéder à des entités fixes telles que des utilisateurs, groupes, courrier, messages, calendriers, tâches et notes provenant de services tels qu’Outlook, OneDrive, Azure Active Directory, Organiseur, OneNote, etc. Vous pouvez également obtenir des relations calculées optimisées par Office Graph (uniquement pour les utilisateurs commerciaux) comme la liste des utilisateurs avec lesquels vous travaillez ou les documents qui vous sont suggérés.

Microsoft Graph expose deux points de terminaison. Un point de terminaison généralement disponible /v1.0 et un point de terminaison d’évaluation /beta.  Vous pouvez utiliser /v1.0 dans vos applications de production, mais pas /beta.  Le point de terminaison d’évaluation /beta nous permet de mettre les fonctionnalités les plus récentes à la disposition des développeurs afin que ceux-ci les testent et fournissent des commentaires. Les API en version bêta peuvent changer à tout moment et ne sont pas prêtes pour une utilisation de production.

<!--<a name="msg_queries"> </a>-->

##Requêtes courantes

Voici quelques exemples de requêtes courantes utilisant l’API Microsoft Graph :

| **Opération** | **Point de terminaison de service** |
|:--------------------------|:----------------------------------------|
|   GET mon profil |    `https://graph.microsoft.com/v1.0/me` |
|   GET mes fichiers|   `https://graph.microsoft.com/v1.0/me/drive/root/children` |
|   GET ma photo     | `https://graph.microsoft.com/v1.0/me/photo/$value` |
|   GET mon courrier |   `https://graph.microsoft.com/v1.0/me/messages` |
|   GET mon courrier Importance haute | `https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'` |
|   GET mon calendrier |   `https://graph.microsoft.com/v1.0/me/calendar` |
|   GET mon responsable  | `https://graph.microsoft.com/v1.0/me/manager` |
|   GET le dernier utilisateur pour modifier le fichier foo.txt |  `https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser` |
|   GET des groupes unifiés dont je suis membre|   `https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')` |
|   GET des utilisateurs dans mon organisation     | `https://graph.microsoft.com/v1.0/users` |
|   GET des conversations de groupe |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations` |
|   GET des personnes en relation avec moi    | `https://graph.microsoft.com/beta/me/people` |
|   GET les fichiers qui me sont suggérés |  `https://graph.microsoft.com/beta/me/trendingAround` |
|   GET les personnes avec qui je collabore     | `https://graph.microsoft.com/beta/me/workingWith` |
|   GET mes tâches    | `https://graph.microsoft.com/beta/me/tasks` |
|   GET mes notes |  `https://graph.microsoft.com/beta/me/notes/notebooks` |

<!-- <a name="msg_roof"> </a> -->

## Toutes les données d’Office 365 sous un même toit

Le diagramme suivant illustre la pile de développeurs Microsoft Graph et son fonctionnement.

![Pile de développeurs de l’API Microsoft Graph.](./images/MicrosoftGraph_DevStack.png)

 >  Votre avis compte beaucoup pour nous. Communiquez avec nous sur [Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph). Posez vos questions avec les tags [MicrosoftGraph] et [Office 365].



