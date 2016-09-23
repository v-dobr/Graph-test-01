# Types de ressources et réponses d’erreur Microsoft Graph

<!--In this article:
  
-   [Status code](#msg_status_code)
-   [Error resource type](#msg_error_resource_type)
-   [Code property](#msg_code_property)

<a name="msg_error_response"> </a> -->

Les erreurs dans Microsoft Graph sont renvoyées à l’aide de codes d’état HTTP standard, 
ainsi que d’un objet de réponse d’erreur JSON.

## Codes d’état HTTP

Le tableau suivant répertorie et décrit les codes d’état HTTP pouvant être renvoyés.

| Code d'état | Message d’état                  | Description                                                                                                                            |
|:------------|:--------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|
| 400         | Demande incorrecte                     | Impossible de traiter la demande car elle est de format non valide ou incorrecte.                                                                       |
| 401         | Non autorisé                    | Informations d’authentification nécessaires manquantes ou non valides pour la ressource.                                                   |
| 403         | Interdit                       | L’accès à la ressource demandée est refusé. L’utilisateur ne dispose peut-être pas des autorisations suffisantes.                                                 |
| 404         | Introuvable                       | La ressource demandée n’existe pas.                                                                                                  |
| 405         | Méthode non autorisée              | La méthode HTTP dans la demande n’est pas autorisée sur la ressource.                                                                         |
| 406         | Non acceptable                  | Ce service ne prend pas en charge le format demandé dans l’en-tête Accept.                                                                |
| 409         | Conflit                        | L’état actuel n’est pas compatible avec les attentes de la demande. Par exemple, le dossier parent spécifié n’existe peut-être pas.                   |
| 410         | Non disponible                            | La ressource demandée n’est plus disponible sur le serveur.                                               |
| 411         | Longueur requise                 | Un en-tête Content-Length est requise sur la demande.                                                                                    |
| 412         | Échec de la condition préalable             | Une condition préalable fournie dans la demande (un en-tête if-match, par exemple) ne correspond pas à l’état actuel de la ressource.                       |
| 413         | Entité de demande trop grande        | La taille de la demande dépasse la limite maximale.                                                                                            |
| 415         | Type de support non pris en charge          | Le type de contenu de la demande est un format qui n’est pas pris en charge par le service.                                                      |
| 416         | La gamme demandée ne peut pas être satisfaite | La gamme d’octets spécifiée n’est pas valide ou n’est pas disponible.                                                                                    |
| 422         | Impossible de traiter l’entité            | Impossible de traiter la demande car elle est sémantique incorrecte.                                                                       |
| 429         | Trop de demandes               | L’application cliente a été limitée et ne doit pas tenter de répéter la demande tant qu’un certain délai ne s’est pas écoulé.                |
| 500         | Erreur interne du serveur           | Une erreur du serveur interne s’est produite lors du traitement de la demande.                                                                       |
| 501         | Non implémenté                 | La fonctionnalité demandée n’est pas implémentée.                                                                                               |
| 503         | Service non disponible             | Le service est temporairement indisponible. Vous pouvez répéter la demande après un délai. Il peut y avoir un en-tête Retry-After.                   |
| 507         | Stockage insuffisant            | Le quota de stockage maximum a été atteint.                                                                                            |
| 509         | Limite de bande passante dépassée        | Votre application a été limitée car elle a dépassé la capacité maximale de la bande passante. Votre application peut renouveler la demande une fois qu’un délai plus long s’est écoulé. |

La réponse d’erreur est un seul objet JSON qui contient une propriété unique nommée **error**. Cet objet inclut tous les détails de l’erreur. 
Vous pouvez utiliser les informations renvoyées ici à la place, ou en plus du code d’état HTTP. Voici un exemple de corps d’erreur JSON complet.

<!-- { "blockType": "example", "@odata.type": "sample.error", "expectError": true, "name": "example-error-response"} -->
```json
{
  "error": {
    "code": "invalidRange",
    "message": "Uploaded fragment overlaps with existing data.",
    "innerError": {
      "requestId": "request-id",
      "date": "date-time"
    }
  }
}
```

<!--<a name="msg_error_resource_type"> </a> -->

## Type de ressource d’erreur

La ressource d’erreur est renvoyée chaque fois qu’une erreur se produit dans le traitement d’une demande.

Les réponses d’erreur suivent la définition de la spécification 
[OData v4](http://docs.oasis-open.org/odata/odata-json-format/v4.0/os/odata-json-format-v4.0-os.html#_Toc372793091) 
pour les réponses d’erreur.

### Représentation JSON

La ressource d’erreur est composée de ces ressources :

<!-- { "blockType": "resource", "@odata.type": "sample.error" } -->
```json
{
  "error": { "@odata.type": "odata.error" }  
}
```

#### type de ressource odata.error

À l’intérieur de la réponse d’erreur se trouve une ressource d’erreur qui inclut 
les propriétés suivantes :

<!-- { "blockType": "resource", "@odata.type": "odata.error", "optionalProperties": [ "target", "details", "innererror"] } -->
```json
{
  "code": "string",
  "message": "string",
  "innererror": { "@odata.type": "odata.error" }
}
```

| Nom de la propriété  | Valeur                  | Description\                                                                                               |
|:---------------|:-----------------------|:-----------------------------------------------------------------------------------------------------------|
| **code**       | chaîne                 | Chaîne de code pour l’erreur qui s’est produite                                                            |
| **message**    | chaîne                 | Message ready de développeur sur l’erreur qui s’est produite. Il ne doit pas être affiché pour l’utilisateur directement. |
| **innererror** | objet d’erreur           | Facultatif. Objets d’erreur supplémentaires qui peuvent être plus spécifiques que l’erreur de niveau supérieur.                     |
<!-- {
  "type": "#page.annotation",
  "description": "Understand the error format for the API and error codes.",
  "keywords": "error response, error, error codes, innererror, message, code",
  "section": "documentation",
  "tocPath": "Misc/Error Responses"
} -->

<!--<a name="msg_code_property"> </a> -->

#### Propriété du code

La propriété `code` contient l’une des valeurs possibles suivantes. 
Les applications doivent être prêtes à gérer l’une de ces erreurs.

| Code                      | Description
|:--------------------------|:--------------
| **accessDenied**          | L’appelant n’est pas autorisé à effectuer l’action. 
| **activityLimitReached**  | L’application ou l’utilisateur a été limité(e).
| **generalException**      | Une erreur inconnue s’est produite.
| **invalidRange**          | La gamme d’octets spécifiée n’est pas valide ou n’est pas disponible.
| **invalidRequest**        | La demande est de format non valide ou incorrecte.
| **itemNotFound**          | La ressource est introuvable.
| **malwareDetected**       | Un programme malveillant a été détecté dans la ressource demandée.
| **nameAlreadyExists**     | Le nom de l’élément spécifié existe déjà.
| **notAllowed**            | L’action n’est pas autorisée par le système.
| **notSupported**          | La demande n’est pas prise en charge par le système.
| **resourceModified**      | La ressource mise à jour a changé depuis que l’appelant l’a lue, généralement une discordance eTag.
| **resyncRequired**        | Le jeton delta n’est plus valide et l’application doit rétablir l’état de synchronisation.
| **serviceNotAvailable**   | Le service n’est pas disponible. Réessayez la demande après un délai. Il peut y avoir un en-tête Retry-After. 
| **quotaLimitReached**     | L’utilisateur a atteint sa limite de quota.
| **unauthenticated**       | L’appelant n’est pas authentifié.

L’objet `innererror` peut contenir de manière récursive plus d’objets `innererror` 
avec des codes d’erreur supplémentaires, plus spécifiques. Lors de la gestion d’une erreur, 
les applications doivent mettre tous les codes d’erreur disponibles en boucle et utiliser 
le code le plus détaillé qu’elles comprennent. Certains codes plus détaillés 
sont répertoriés en bas de cette page.

Pour vérifier qu’un objet d’erreur est une erreur que vous attendez, vous devez effectuer 
une boucle sur les objets `innererror` pour rechercher les codes d’erreur attendus. Par exemple :

```csharp
public bool IsError(string expectedErrorCode)
{
    OneDriveInnerError errorCode = this.Error;
    while (null != errorCode)
    {
        if (errorCode.Code == expectedErrorCode)
            return true;
        errorCode = errorCode.InnerError;
    }
    return false;
}
```

Pour un exemple de gestion correcte des erreurs, voir 
[Gestion des codes d’erreur](https://gist.github.com/rgregg/a1866be15e685983b441).

La propriété `message` à la racine contient un message d’erreur destiné au développeur. 
Les messages d’erreur ne sont pas localisés et ne doivent pas être affichés directement 
pour l’utilisateur. Lors de la gestion des erreurs, votre code ne doit pas s’articuler 
autour des valeurs `message`, car elles peuvent changer à tout moment et elles contiennent 
souvent des informations dynamiques spécifiques de la demande ayant échoué. 
Vous devez coder uniquement par rapport aux codes d’erreur renvoyés dans les propriétés `code`.

#### Codes d’erreur détaillés
Vous trouverez ci-dessous quelques erreurs supplémentaires que votre application peut rencontrer à l’intérieur 
des objets `innererror` imbriqués. Les applications ne sont pas requises pour les gérer mais elles le peuvent, 
le cas échéant. Le service peut ajouter de nouveaux codes d’erreur ou cesser de renvoyer d’anciens codes à tout moment. 
Il est donc important que toutes les applications soient en mesure de gérer les [codes d’erreur de base](#code-property).

| Code                               | Description
|:-----------------------------------|:----------------------------------------------------------
| **accessRestricted**               | L’accès est limité au propriétaire de l’élément.
| **cannotSnapshotTree**             | Impossible d’obtenir un instantané delta cohérent. Essayez à nouveau ultérieurement.
| **childItemCountExceeded**         | La limite maximale du nombre d’éléments enfants a été atteinte.
| **entityTagDoesNotMatch**          | ETag ne correspond pas à la valeur de l’élément actuelle.
| **fragmentLengthMismatch**         | La taille totale déclarée pour ce fragment est différente de celle de la session de téléchargement.
| **fragmentOutOfOrder**             | Le fragment téléchargé ne fonctionne pas.
| **fragmentOverlap**                | Le fragment téléchargé remplace les données existantes.
| **invalidAcceptType**              | Type d’acceptation non valide.
| **invalidParameterFormat**         | Format de paramètre non valide.
| **invalidPath**                    | Le nom contient des caractères non valides.
| **invalidQueryOption**             | Option de requête non valide.
| **invalidStartIndex**              | Index de début non valide.
| **lockMismatch**                   | Le jeton de verrouillage ne correspond pas au verrouillage existant.
| **lockNotFoundOrAlreadyExpired**   | Il n’existe actuellement aucun verrou non expiré sur l’élément.
| **lockOwnerMismatch**              | L’ID du propriétaire du verrouillage ne correspond pas à l’ID fourni.
| **malformedEntityTag**             | Le format de l’en-tête ETag est incorrect. Les en-têtes ETag doivent être des chaînes entre guillemets.
| **maxDocumentCountExceeded**       | La limite maximale sur le nombre de documents est atteinte.
| **maxFileSizeExceeded**            | La taille maximale du fichier est dépassée.
| **maxFolderCountExceeded**         | La limite maximale sur le nombre de dossiers est atteinte.
| **maxFragmentLengthExceeded**      | La taille maximale du fichier est dépassée.
| **maxItemCountExceeded**           | La limite maximale sur le nombre d’éléments est atteinte.
| **maxQueryLengthExceeded**         | La longueur de requête maximale est dépassée.
| **maxStreamSizeExceeded**          | La taille maximale de flux est dépassée.
| **parameterIsTooLong**             | La longueur maximale du paramètre est dépassée.
| **parameterIsTooSmall**            | Le paramètre est inférieur à la valeur minimale.
| **pathIsTooLong**                  | Le chemin d’accès dépasse la longueur maximale.
| **pathTooDeep**                    | La limite de profondeur de hiérarchie de dossier est atteinte.
| **propertyNotUpdateable**          | La propriété ne peut pas être mise à jour.
| **resyncApplyDifferences**         | Resynchronisation requise. Remplacez les éléments locaux par la version du serveur (y compris les suppressions) si vous êtes sûr que le service était à jour avec vos modifications locales lors de la dernière synchronisation. Téléchargez les modifications locales que le serveur ignore.
| **resyncRequired**                 | La resynchronisation est requise.
| **resyncUploadDifferences**        | Resynchronisation requise. Téléchargez les éléments locaux que le service n’a pas renvoyés, et téléchargez les fichiers qui diffèrent de la version du serveur (tout en conservant les deux copies si vous ne savez pas quelle est la plus récente).
| **serviceNotAvailable**            | Le serveur ne peut pas traiter la demande actuelle.
| **serviceReadOnly**                | La ressource est temporairement en lecture seule.
| **throttledRequest**               | Trop de demandes.
| **tooManyResultsRequested**        | Trop de résultats demandés.
| **tooManyTermsInQuery**            | Trop de termes dans la requête.
| **totalAffectedItemCountExceeded** | L’opération n’est pas autorisée, car le nombre d’éléments concernés dépasse le seuil.
| **truncationNotAllowed**           | La troncation des données n’est pas autorisée.
| **uploadSessionFailed**            | La session de téléchargement a échoué.
| **uploadSessionIncomplete**        | La session de téléchargement est incomplète.
| **uploadSessionNotFound**          | Session de téléchargement introuvable.
| **virusSuspicious**                | Ce document est suspect et contient peut-être un virus.
| **zeroOrFewerResultsRequested**    | Zéro résultat ou moins demandés.

<!-- ##Additional Resources##

- [Microsoft Graph API release notes and known issues](microsoft-graph-api-release-notes-known-issues.md )
- [Hands on lab: Deep dive into the Microsoft Graph API](http://dev.office.com/hands-on-labs/4585) -->
