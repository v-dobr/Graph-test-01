# Appel de l’API Microsoft Graph

Pour accéder à une ressource Microsoft Graph et la manipuler, vous appelez et spécifiez les URL de ressource à l’aide de l’une des opérations suivantes :   

- GET
- POST
- PATCH
- PUT
- DELETE 

Toutes les demandes de l’API Microsoft Graph utilisent le modèle d’URL de base suivant :

```
    https://graph.microsoft.com/{version}/{resource}?[odata_query_parameters]
```

Pour cette URL :

- `https://graph.microsoft.com` est le point de terminaison de l’API Microsoft Graph.
- `{version}` est la version du service cible, par exemple `v1.0` ou `beta`.
- `{resource}` est le segment ou le chemin d’accès de la ressource, tel que :
  - `users`, `groups`, `devices`, `organization`
  - L’alias `me`, qui est résolu en utilisateur connecté
   - Les ressources appartenant à un utilisateur, telles que `me/events`, `me/drive` ou `me/messages`
  - L’alias `myOrganization`, qui est résolu en client de l’utilisateur connecté de l’organisation
- `[odata_query_parameters]` représente des paramètres de requête OData supplémentaires, tels que `$filter` et `$select`.

Si vous le souhaitez, vous pouvez également spécifier le client dans le cadre de votre demande. Lorsque vous utilisez `me`, ne spécifiez pas le client. Pour obtenir une liste des demandes courantes, voir [Présentation de Microsoft Graph](overview.md).

##Métadonnées de l’API Microsoft Graph
Le document de service ($metadata) est publié au niveau de la racine de service. Par exemple, vous pouvez afficher le document de service pour les versions v1.0 et bêta via les URL suivantes.

Métadonnées de la version `v1.0` de l’API Microsoft Graph.
```
    https://graph.microsoft.com/v1.0/$metadata
```
Métadonnées `beta` de l’API Microsoft Graph.
```
    https://graph.microsoft.com/beta/$metadata
```

Les métadonnées vous permettent de visualiser les entités, les types et les ensembles d’entités, les types complexes et enums de l’API Microsoft Graph. Grâce aux métadonnées et aux outils tiers disponibles, vous pouvez créer des objets sérialisés et générer des bibliothèques clientes pour une utilisation simplifiée de l’API REST.  

Une URL de ressource est déterminée par le modèle de données d’entité Microsoft Graph. La prescription est décrite dans le schéma de métadonnées de l’entité ($metadata). 

Les noms de la ressource de chemin d’accès, les paramètres de la requête et les valeurs et paramètres d’action ne respectent pas la casse. 
Toutefois, les valeurs que vous attribuez, les ID d’entité et autres valeurs codées en Base64 respectent la casse.

Les sections suivantes présentent quelques appels de mode de programmation de base à l’API Microsoft Graph.

##Navigation d’un ensemble vers un membre

Pour afficher les informations relatives à un utilisateur, vous obtenez l’entité `User` de la collection `users` associée à l’utilisateur spécifique identifié par son identificateur, à l’aide d’une demande HTTPS GET. Pour une entité `User`, la propriété `id` ou `userPrincipalName` peut être utilisée comme identificateur. La demande de l’exemple suivant utilise la valeur `userPrincipalName` comme ID utilisateur. 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com HTTP/1.1
Authorization : Bearer <access_token>
```

Si l’opération réussit, vous devez obtenir une réponse 200 OK contenant la représentation de la ressource utilisateur dans la charge, comme illustré ci-dessous :

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 982

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
    "city": "Redmond",
    "country": "USA",
    "department": "Help Center",
    "displayName": "John Doe",
    "givenName": "John",
    "userPrincipalName": "Johndoe@contoso.onmicrosoft.com",

    ... 
}
```


##Projection d’une entité vers des propriétés
Pour récupérer uniquement les données biographiques de l’utilisateur, telles que la description de ses _informations personnelles_ et ses compétences, vous pouvez ajouter l’option de requête $select à la demande précédente. Par exemple :

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com?$select=displayName,aboutMe,skills HTTP/1.1
Authorization : Bearer <access_token>
```

La réponse réussie renvoie l’état 200 OK et une charge au format suivant :

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 169

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "aboutMe": "A cool and nice guy.",
    "displayName": "John Doe",
    "skills": [
        "n-Lingual",
        "public speaking",
        "doodling"
    ]
}
```

Ici, au lieu des ensembles de propriétés entiers sur l’entité `user`, seules les propriétés `aboutMe`, `displayName` et `skills` sont renvoyées.

##Traversée vers une autre ressource via une relation
Un responsable a une relation `directReports` avec les autres utilisateurs qui lui font des signalements. Pour interroger la liste des collaborateurs d’un utilisateur, vous pouvez utiliser la requête HTTPS GET pour naviguer vers la cible souhaitée via la traversée de relation. 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com/directReports HTTP/1.1
Authorization : Bearer <access_token>
```

La réponse réussie renvoie l’état 200 OK et une charge au format suivant :

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 152
    
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#directoryObjects/$entity",
    "@odata.type": "#microsoft.graph.user",
    "id": "c37b074d-fe9d-4e68-83ad-b4401d3be174",
    "department": "Sales & Marketing",
    "displayName": "Bonnie Kearney",

    ...
}
```

De même, vous pouvez suivre une relation pour naviguer vers des ressources connexes. Par exemple, la relation `user => messages` permet la traversée d’un nœud Azure AD vers un nœud Exchange Online. L’exemple ci-dessous décrit comment procéder dans un appel d’API REST :


```no-highlight 
GET https://graph.microsoft.com/v1.0/me/messages HTTP/1.1
Authorization : Bearer <access_token>
```

    
La réponse réussie renvoie l’état 200 OK et une charge au format suivant :


```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
odata-version: 4.0
content-length: 147
    
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/Messages",
  "@odata.nextLink": "https://graph.microsoft.com/v1.0/me/messages?$top=1&$skip=1",
  "value": [
    {
      "@odata.etag": "W/\"FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej\"",
      "id": "<id-value>",
      "createdDateTime": "2015-11-14T00:24:42Z",
      "lastModifiedDateTime": "2015-11-14T00:24:42Z",
      "changeKey": "FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej",
      "categories": [],
      "receivedDateTime": "2015-11-14T00:24:42Z",
      "sentDateTime": "2015-11-14T00:24:28Z",
      "hasAttachments": false,
      "subject": "Did you see last night's game?",
      "body": {
        "ContentType": "HTML",
        "Content": "<content>"
      },
      "BodyPreview": "it was great!",
      "Importance": "Normal",
            
       ...
    }
  ]
}
```

##Projection d’entités vers des propriétés
Outre la projection d’une entité unique vers ses propriétés, vous pouvez également appliquer la même option de requête `$select` à une collection d’entités pour les projeter vers une collection de certaines de leurs propriétés. Par exemple, pour interroger le nom des éléments de lecteur de l’utilisateur connecté, vous pouvez envoyer la requête HTTPS GET suivante :

```no-highlight 
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name HTTP/1.1
Authorization : Bearer <access_token>
```

La réponse réussie renvoie un code d’état 200 OK et une charge contenant les noms et les types de fichiers partagés, comme indiqué dans l’exemple suivant :

```no-highlight 
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/drive/root/children(name,type)",
  "value": [
    {
      "@odata.etag": "\"{896A8E4D-27BF-424B-A0DA-F073AE6570E2},2\"",
      "name": "Shared with Everyone"
    },
    {
      "@odata.etag": "\"{B39D5D2E-E968-491A-B0EB-D5D0431CB423},1\"",
      "name": "Documents"
    },
    {
      "@odata.etag": "\"{9B51EA38-3EE6-4DC1-96A6-230E325EF054},2\"",
      "name": "newFile.txt"
    }
  ]
}
```

##Interrogation d’un sous-ensemble d’utilisateurs avec l’option de requête de filtrage
Pour rechercher les employés d’une fonction spécifique au sein d’une organisation, vous pouvez naviguer à partir de la collection d’utilisateurs, puis spécifier une option de requête $filter. Un exemple est indiqué comme suit :

    
```no-highlight 
GET https://graph.microsoft.com/v1.0/users/?$filter=jobTitle+eq+%27Helper%27 HTTP/1.1
Authorization : Bearer <access_token>
```

La réponse réussie renvoie le code d’état 200 OK et une liste d’utilisateurs avec la fonction spécifiée (`'Helper'`), comme indiqué dans l’exemple suivant :

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal;odata.streaming=true;IEEE754Compatible=false;charset=utf-8
odata-version: 4.0
content-length: 986

{
    "@odata.context": "https://graph.microsoft.com/v1.0/contoso.onmicrosoft.com/$metadata#users",
    "value": [
        {
            "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
            "city": "Redmond",
            "country": "USA",
            "department": "Help Center",
            "displayName": "Jane Doe",
            "givenName": "Jane",
            "jobTitle": "Helper",
            ......
            "mailNickname": "Jane",
            "mobile": null,
            "otherMails": [
                "jane.doe@contoso.onmicrosoft.com"
            ],
            ......
            "surname": "Doe",
            "usageLocation": "US",
            "userPrincipalName": "help@contoso.onmicrosoft.com",
            "userType": "Member"
        },
        
        ...
    ]
}
```

##Appel de fonctions ou d’actions OData
Microsoft Graph prend également en charge les fonctions et actions OData pour manipuler les ressources. 
Par exemple, la requête POST HTTPS suivante permet à l’utilisateur connecté (`me`) d’envoyer un message électronique :
```no-highlight 
POST https://graph.microsoft.com/v1.0/me/microsoft.graph.sendMail HTTP/1.1
authorization: bearer <access_token>
content-type: application/json
content-length: 96

{
  "message": {
    "subject": "Meet for lunch?",
    "body": {
      "contentType": "Text",
      "content": "The new cafeteria is open."
    },
    "toRecipients": [
      {
        "emailAddress": {
          "address": "garthf@a830edad9050849NDA1.onmicrosoft.com"
        }
      }
    ],
    "attachments": [
      {
        "@odata.type": "#Microsoft.OutlookServices.FileAttachment",
        "name": "menu.txt",
        "contentBytes": "bWFjIGFuZCBjaGVlc2UgdG9kYXk="
      }
    ]
  },
  "saveToSentItems": "false"
}
```

La charge de la demande contient l’entrée vers l’action `microsoft.graph.sendMail`, qui est également définie dans $metadata.

Avec un point de terminaison unifié unique, Microsoft Graph simplifie l’interface de programmation d’applications pour les services dans le cloud Microsoft. Par conséquent, les limites des services, qui autrement seraient cloisonnés, disparaissent. En tant que développeur d’applications, vous ne devez plus effectuer le suivi des sources de données et implémenter des interfaces personnalisées entre diverses sources de données. 


