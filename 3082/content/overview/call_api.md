# Llamar a la API de Microsoft Graph

Para tener acceso y manipular un recurso de Microsoft Graph, se debe llamar a las direcciones URL del recurso y especificarlas mediante una de las siguientes operaciones:   

- GET
- POST
- PATCH
- PUT
- DELETE 

Todas las solicitudes de la API de Microsoft Graph usan el siguiente patrón de URL básico:

```
    https://graph.microsoft.com/{version}/{resource}?[odata_query_parameters]
```

Para esta URL:

- `https://graph.microsoft.com` es el punto de conexión de la API de Microsoft Graph.
- `{version}` es la versión del servicio de destino; por ejemplo, `v1.0` o `beta`.
- `{resource}` es el segmento o la ruta de los recursos, como:
  - `users`, `groups`, `devices`, `organization`
  - El alias `me`, que se resuelve en el usuario que inició sesión.
   - Los recursos que pertenecen a un usuario, tales como `me/events`, `me/drive` o `me/messages`.
  - El alias `myOrganization`, que se resuelve en el inquilino de la organización del usuario que ha iniciado sesión
- `[odata_query_parameters]` representa parámetros de consulta de OData adicionales como `$filter` y `$select`.

Opcionalmente, puede especificar también el inquilino como parte de su solicitud. Si usa `me`, no especifique el inquilino. Para obtener una lista de las solicitudes comunes, vea [Información general de Microsoft Graph](overview.md).

##Metadatos de la API de Microsoft Graph
El documento de servicio ($metadata) se publica en la raíz del servicio. Por ejemplo, puede ver el documento de servicio para las versiones v1.0 y beta a través de las siguientes direcciones URL.

Metadatos `v1.0` de la API de Microsoft Graph.
```
    https://graph.microsoft.com/v1.0/$metadata
```
Metadatos `beta` de la API de Microsoft Graph.
```
    https://graph.microsoft.com/beta/$metadata
```

Los metadatos permiten ver entidades, tipos de entidades y conjuntos, tipos complejos y enumeraciones de la API de Microsoft Graph. Con los metadatos y las herramientas de terceros ya disponibles, se pueden crear objetos serializados y generar bibliotecas de cliente para simplificar el uso de la API de REST.  

Una URL de recurso se determina mediante el Entity Data Model de Microsoft Graph. La prescripción se describe en el esquema de metadatos de la entidad ($metadata). 

Los nombres de recursos de la URL de la ruta de acceso, los parámetros de consulta, así como los parámetros de acción y los valores no distinguen mayúsculas de minúsculas. Sin embargo, los valores que usted asigne, los identificadores de entidad y otros valores codificados en base64 distinguen mayúsculas de minúsculas.

Las siguientes secciones muestran algunas llamadas de modelo de programación básico a la API de Microsoft Graph.

##Navegación desde un conjunto a un miembro

Para ver la información de un usuario, se usa una solicitud HTTPS GET con la que se obtiene la entidad `User` de la colección `users` para el usuario concreto identificado por su identificador. Para una entidad `User`, se puede usar como identificador la propiedad `id` o la propiedad `userPrincipalName`. En la solicitud de ejemplo siguiente se usa el valor `userPrincipalName` como identificador del usuario. 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com HTTP/1.1
Authorization : Bearer <access_token>
```

Si es correcta, se obtiene una respuesta 200 OK que contiene la representación de recursos del usuario en la carga, como se muestra a continuación:

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


##Proyección desde una entidad a las propiedades
Para recuperar tan solo los datos biográficos del usuario, como la descripción _Acerca de mí_ del usuario y su conjunto de habilidades, puede agregar la opción de consulta $select a la solicitud anterior. Por ejemplo:

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com?$select=displayName,aboutMe,skills HTTP/1.1
Authorization : Bearer <access_token>
```

La respuesta correcta devuelve el estado 200 OK y una carga en el formato siguiente:

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

Aquí, en lugar de todo el conjunto de propiedades de la entidad `user`, solo se devuelven las propiedades `aboutMe`, `displayName` y `skills`.

##Recorrido a otro recurso a través de la relación
Un administrador mantiene una relación `directReports` con el resto de los usuarios a su cargo. Para consultar la lista de relaciones directas de un usuario, se puede usar la siguiente solicitud HTTPS GET para navegar hasta el destino deseado mediante un recorrido de relaciones. 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com/directReports HTTP/1.1
Authorization : Bearer <access_token>
```

La respuesta correcta devuelve el estado 200 OK y una carga en el formato siguiente:

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

De forma similar, se puede seguir una relación para ir a los recursos relacionados. Por ejemplo, la relación `user => messages` permite un recorrido desde un nodo de Azure AD a un nodo de Exchange Online. En el ejemplo siguiente se muestra cómo hacer esto en una llamada de la API de REST:


```no-highlight 
GET https://graph.microsoft.com/v1.0/me/messages HTTP/1.1
Authorization : Bearer <access_token>
```

    
La respuesta correcta devuelve el estado 200 OK y una carga en el formato siguiente:


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

##Proyección desde entidades a las propiedades
Además de la proyección desde una única entidad a sus propiedades, también es posible aplicar la misma opción de consulta `$select` a una colección de entidades para proyectarlas a algunas de sus propiedades. Por ejemplo, para consultar el nombre de elementos de la unidad del usuario que ha iniciado sesión, se puede enviar la siguiente solicitud HTTPS GET:

```no-highlight 
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name HTTP/1.1
Authorization : Bearer <access_token>
```

La respuesta correcta devuelve un código de estado 200 OK y una carga que contiene los nombres y los tipos de los archivos compartidos, tal como se muestra en el ejemplo siguiente:

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

##Consultar un subconjunto de usuarios con la opción de consulta de filtrado
Para encontrar los empleados de un puesto específico dentro de una organización, se puede navegar desde la colección users y, después, especificar una opción de consulta $filter. A continuación se muestra un ejemplo:

    
```no-highlight 
GET https://graph.microsoft.com/v1.0/users/?$filter=jobTitle+eq+%27Helper%27 HTTP/1.1
Authorization : Bearer <access_token>
```

La respuesta correcta devuelve el código de estado 200 OK y una lista de usuarios con el puesto de trabajo especificado (`'Helper'`), como se muestra en el ejemplo siguiente:

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

##Llamar a acciones o funciones OData
Microsoft Graph también admite acciones y funciones OData para manipular los recursos. Por ejemplo, la siguiente solicitud HTTPS POST permite al usuario que ha iniciado sesión (`me`) enviar un mensaje de correo electrónico:
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

La carga de la solicitud contiene la entrada a la acción `microsoft.graph.sendMail`, que también está definida en $metadata.

Con un solo punto de conexión unificado, Microsoft Graph simplifica la interfaz de programación de aplicaciones para los servicios en la nube de Microsoft. Como resultado, desaparecen los límites de los servicios que, de lo contrario, estarían aislados. Los desarrolladores de aplicaciones ya no tienen que realizar un seguimiento de los orígenes de datos ni implementar interfaces personalizadas entre varios orígenes de datos. 


