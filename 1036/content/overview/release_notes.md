# Problèmes connus et notes de publication de Microsoft Graph

Cet article fournit des informations sur les nouvelles fonctionnalités pour les développeurs qui sont disponibles dans la version de novembre 2015 de l’API Microsoft Graph, ainsi que les problèmes connus qui peuvent vous être utiles. 


## Fonctionnalités GA dans l’API Microsoft Graph

Les fonctionnalités de l’API Microsoft Graph suivantes sont généralement disponibles :

* Utilisateurs
* Groups
* Files
* Application de messagerie
* Calendrier
* Contacts personnels 
* Créer, lire, mettre à jour et supprimer des opérations
* Prise en charge du partage des ressources d’origine croisée (CORS).

    
## Fonctionnalités d’évaluation dans l’API Microsoft Graph

Les fonctionnalités d’évaluation de l’API Microsoft Graph suivantes sont disponibles :

* Notes 
* Tâches
* Excel
* Contacts 
* Contacts organisationnels
* Applications
* Principaux de service
* Extensions du schéma Directory
* Webhooks
* Perspectives et relations : documents suggérés et collaborateurs
* Accès instantané des groupes au contenu après sa création
* Modèle d’authentification convergée pour les comptes personnels, ainsi que comptes professionnels et scolaires. Bien qu’il s’agisse d’une fonctionnalité d’évaluation, elle est disponible à la fois dans les versions `/v1.0` et `/beta`.


## Problèmes connus de Microsoft Graph

Voici les problèmes connus avec Microsoft Graph.

### Utilisateurs
#### Pas d’accès instantané après création
Les utilisateurs peuvent être créées immédiatement via un POST sur l’entité de l’utilisateur. Une licence Office 365 doit d’abord être affectée à un utilisateur, afin d’obtenir un accès aux services Office 365. Même ensuite, en raison de la nature distribuée du service, un délai de 15 minutes peut s’écouler avant que les entités de fichiers, messages et événements puissent être utilisées pour cet utilisateur, via l’API Microsoft Graph. Pendant ce temps, les applications recevront une réponse d’erreur HTTP 404. 

#### Restrictions de photo
La lecture et la mise à jour de la photo du profil d’un utilisateur ne sont possibles que si l’utilisateur dispose d’une boîte aux lettres.  En outre, les photos qui ont *peut-être* été précédemment stockées à l’aide de la propriété **thumbnailPhoto** (avec la version préliminaire de l’API unifiée Office 365, Azure AD Graph, ou via la synchronisation AD Connect) ne seront plus accessibles via la propriété de photo utilisateur de Microsoft Graph.  L’échec de lecture ou de mise à jour d’une photo, dans ce cas, entraînerait l’erreur suivante :

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **REMARQUE** :  Peu de temps après la version GA, le stockage et l’extraction des photos du profil utilisateur seront activés, même si l’utilisateur ne dispose pas d’une boîte aux lettres, et cette erreur devrait disparaître.

#### Dossier des contacts par défaut

Dans la version `/v1.0`, `GET /me/contactFolders` n’inclut pas de dossier des contacts par défaut de l’utilisateur. 

Un correctif sera disponible. En attendant, vous pouvez utiliser la requête de 
[liste de contacts](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts) suivante et la propriété **parentFolderId** pour contourner le problème et obtenir l’ID du dossier des contacts par défaut :

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
Dans la requête ci-dessus :
1. `/me/contacts?$top=1` obtient les propriétés d’un [contact](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact) dans le dossier de contacts par défaut.
2. L’ajout de `&$select=parentFolderId` renvoie uniquement la propriété **parentFolderId** du contact, qui est l’ID du dossier de contacts par défaut.

#### Ajout de calendriers ICS et accès à ces calendriers dans la boîte aux lettres de l’utilisateur
Actuellement, il existe une prise en charge partielle pour un calendrier basé sur un abonnement de calendrier Internet (ICS) :
* Vous pouvez ajouter un calendrier ICS à la boîte aux lettres d’un utilisateur via l’interface utilisateur, mais pas via l’API Microsoft Graph. 
* [La liste des calendriers de l'utilisateur](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars) vous permet d’obtenir les propriétés **name**, **color** et **id** de chaque 
[calendrier](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar) dans le groupe des calendriers par défaut de l’utilisateur, ou un groupe de calendriers spécifié, y compris les calendriers ICS. 
Vous ne pouvez pas stocker ou accéder à l'URL ICS dans la ressource de calendrier.
* Vous pouvez également [répertorier les événements](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events) d'un calendrier ICS.

### Groupes
#### Stratégie
Microsoft Graph vous permet de créer et nommer un groupe unifié qui ignore les stratégies de groupe unifié qui sont configurées via Outlook Web App. 

#### Étendues d’autorisation de groupe
Microsoft Graph expose deux étendues d’autorisation (*Group.Read.All* et *Group.ReadWrite.All*) pour accéder à des API de groupe.  Un administrateur doit avoir consenti à ces étendues d’autorisation (ce qui est un changement par rapport à la version préliminaire).  À l’avenir, nous prévoyons d’ajouter de nouvelles étendues pour les groupes auxquels peuvent consentir des utilisateurs.

#### Ajout et obtention de pièces jointes de publications de groupe
L’[ajout](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments) de pièces jointes à des publications de groupe, 
et l’[énumération](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments) et l’obtention de pièces jointes de publications de groupe renvoient actuellement le message d’erreur indiquant que la requête OData n’est pas prise en charge. 
Un correctif a été transféré à la fois pour les versions `/v1.0` et `/beta`, et devrait être largement disponible d’ici la fin du mois de janvier 2016.

### Contacts
* Seuls les contacts personnels sont actuellement pris en charge. Les contacts organisationnels ne sont actuellement pas pris en charge dans `/v1.0`, mais vous pouvez les trouver dans `/beta`.
* Le téléphone mobile d’un contact personnel n’est pas renvoyé pour un contact. Il sera ajouté prochainement. En attendant, vous pouvez y accéder via les API Outlook.

### Lecteurs, fichiers et diffusion en continu
* Le premier accès au lecteur personnel d’un utilisateur via Microsoft Graph, avant que l’utilisateur accède à son site personnel via un navigateur, entraîne une réponse 401.
* Le chargement et téléchargement de fichiers (dans des groupes Office, des lecteurs ou des pièces jointes de fichier) est limité à 4 Mo.

### Fonctionnalités disponibles uniquement dans les API REST Office 365 existantes
#### Synchronisation
Les fonctionnalités de synchronisation d’Outlook, OneDrive et Azure AD (dans Azure AD, la désignation requête différentielle est également utilisée) ne sont pas disponibles dans `/v1.0` ou `/beta`.  Si votre application requiert des fonctionnalités de synchronisation, continuez à utiliser l’API Office 365 et les API REST Azure AD, ou explorez la nouvelle fonctionnalité d’évaluation webhooks offerte par Microsoft Graph pour les événements, les messages et les contacts.

> **REMARQUE** :  L’objectif est de rapprocher les API existantes et Microsoft Graph le plus rapidement possible, y compris la synchronisation.

#### Traitement par lots
Le traitement par lots n’est pas pris en charge par Microsoft Graph. Toutefois, vous pouvez utiliser le point de terminaison bêta Outlook et 
[traiter par lot les appels REST Outlook](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests). 

#### Disponibilité en Chine
Le service Microsoft Graph est géré par 21Vianet (et est désormais disponible en Chine). Pour plus d’informations, y compris sur les restrictions, veuillez consulter la page relative aux [déploiements cloud souverains Microsoft Graph](http://graph.microsoft.io/docs/overview/deployments).

#### Fonctions et actions de service
`isMemberOf` et `getObjectsById` ne sont pas disponibles dans Microsoft Graph

### Autorisations de Microsoft Graph
Veuillez consulter la [rubrique sur les étendues d’autorisation](http://graph.microsoft.io/docs/authorization/permission_scopes) pour les dernières informations sur les autorisations déléguées et les autorisations d’application prises en charge par Microsoft Graph.  En outre, il existe les limitations suivantes pour `v1.0` :

|Autorisation |   Type d’autorisation | Restriction |  Alternative |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| Déléguée    | Ne peut pas mettre à jour le numéro de téléphone mobile|    Sélectionner également `Directory.AccessAsUser.All`| 
|_User.ReadWrite.All_|  Déléguée|  Impossible d’effectuer des opérations CRUD sur `User`, à part la mise à jour de photo HD utilisateur et des propriétés de profil étendues| Sélectionnez également `Directory.ReadWrite.All` ou `Directory.AccessAsUser.All` si la suppression de l’utilisateur est requise.|
|_User.Read.All_|   Application |Impossible d’effectuer des opérations de lecture sur d’autres utilisateurs| Sélectionner également `Directory.Read.All`|
| _User.ReadWrite.All_ |    Application |   Impossible d’effectuer des opérations CRUD sur `User`, à part la mise à jour de photo HD utilisateur et des propriétés de profil étendues |    Sélectionner également `Directory.ReadWrite.All`. **REMARQUE** : la suppression de l’utilisateur ne sera pas possible.|
|_Group.Read.All_   | Application | Impossible d’énumérer les groupes ou les appartenances à un groupe.  Lecture toujours possible du contenu de groupe pour les groupes Office   | Sélectionner également `Directory.Read.All` |
|_Group.ReadWrite.All_  | Application   | Impossible d’énumérer les groupes ou les appartenances à un groupe, créer des groupes, mettre à jour les appartenances à un groupe ou supprimer des groupes.  Lecture et mise à jour toujours possibles du contenu de groupe pour les groupes Office.   | Sélectionner également `Directory.ReadWrite.All`. **REMARQUE** :  la suppression d’un groupe ne sera pas possible.|

En outre, il existe les limitations `/beta` suivantes :

|Autorisation |   Type d’autorisation | Restriction |  Alternative |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | Déléguée | Impossible de lire ou de mettre à jour les tâches de planificateur dans les groupes Office  | Sélectionner également `Tasks.ReadWrite`|
|_Tasks.ReadWrite_  | Déléguée | Impossible de lire ou de mettre à jour les tâches de l’utilisateur connecté| Sélectionner également `Group.ReadWrite.All`|

### Limites liées à OData
* Limites **$expand** : 
 * Pas de prise en charge pour `nextLink`
 * Pas de prise en charge pour plus d’un niveau de développement
 * Pas de prise en charge avec des paramètres supplémentaires (**$filtre**, **$select**)
* Les espaces de noms multiples ne sont pas pris en charge
* GET sur `$ref` et le cast n’est pas pris en charge sur les utilisateurs, groupes, appareils, principaux de service et applications.
* `@odata.bind` n’est pas pris en charge.  Cela signifie que les développeurs ne seront en pas en mesure de configurer correctement les `Accepted` ou `RejectedSenders` sur un groupe.
* `@odata.id` n’est pas présent sur les navigations autres que les navigations d’imbrication (comme les messages) lors de l’utilisation de métadonnées minimales
* La recherche/Le filtrage entre charges de travail n’est pas disponible. 
* La recherche de texte intégral (à l’aide de **$search**) est disponible uniquement pour certaines entités, comme les messages.

  >  Votre avis compte beaucoup pour nous. Communiquez avec nous sur [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Posez vos questions avec les tags [MicrosoftGraph] et [Office 365].

  
             .

