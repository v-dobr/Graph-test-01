# Notas de la versión de Microsoft Graph y problemas conocidos

Este artículo proporciona información acerca de las nuevas características para desarrolladores que están disponibles en la versión de noviembre de 2015 de la API de Microsoft Graph y cualquier problema conocido que quizás querría conocer. 


## Características de GA en la API de Microsoft Graph

Las siguientes características de la API de Microsoft Graph están generalmente disponibles:

* Usuarios
* Groups
* Files
* Correo
* Calendario
* Contactos personales 
* Crear, leer, actualizar y eliminar (CRUD) operaciones 
* Soporte técnico de compartir recursos de origen cruzado (CORS).

    
## Características de la vista previa en la API de Microsoft Graph

Están disponibles las siguientes características de la vista previa de la API de Microsoft Graph:

* Notas 
* Tareas
* Excel
* Contactos
* Contactos de la organización
* Aplicaciones
* Entidades de servicio
* Extensiones de esquema de directorio
* Webhooks
* Información y relaciones: Tendencias alrededor y trabajar con
* Grupos de acceso instantáneo al contenido después de la creación
* Modelo de autenticación convergente para cuentas personales así como para cuentas profesionales y educativas. Aunque esta es una versión preliminar de la característica, está disponible tanto en la versión `/v1.0` como en la `/beta`.


## Problemas conocidos de Microsoft Graph

Los siguientes son problemas conocidos de Microsoft Graph.

### Usuarios
#### No hay acceso instantáneo después de la creación
Los usuarios pueden crearse inmediatamente a través de un POST en la entidad del usuario. Una licencia de Office 365 primero se debe asignar a un usuario, con el fin de obtener acceso a los servicios de Office 365. Incluso entonces, debido a la naturaleza distribuida del servicio, puede llevar 15 minutos que los archivos, mensajes y entidades de eventos estén disponibles para su uso para este usuario, a través de la API de Microsoft Graph. Durante este tiempo, las aplicaciones recibirán una respuesta de error HTTP 404. 

#### Restricciones de la foto
Leer y actualizar la foto de perfil de un usuario solo es posible si el usuario tiene un buzón.  Además, cualquier foto que *pueda* haberse almacenado previamente usando la propiedad **thumbnailPhoto** (usando la versión preliminar de la API unificada de Office 365 o Azure AD Graph, o mediante la sincronización de AD Connect) ya no será accesible mediante la propiedad de foto del usuario de Microsoft Graph.  En este caso, un error al leer o actualizar una foto provocaría el error siguiente:

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **NOTA**:  Poco después de GA, se habilitará el almacenamiento y la recuperación de las fotos de perfil del usuario, incluso cuando el usuario no tenga un buzón, y este error debería desaparecer.

#### Carpeta de contactos predeterminada

En la versión `/v1.0`, `GET /me/contactFolders` no incluye la carpeta de contactos predeterminada del usuario. 

Estará disponible una corrección. Mientras tanto, puede usar la siguiente consulta [enumerar contactos](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts) y la propiedad **parentFolderId** como una solución alternativa para obtener el identificador de la carpeta de contactos predeterminada:

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
En la consulta anterior:
1. `/me/contacts?$top=1` obtiene las propiedades de un elemento [contact](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact) de la carpeta de contactos predeterminada.
2. Al anexar `&$select=parentFolderId` se devuelve solo la propiedad **parentFolderId** del contacto, que es el identificador de la carpeta de contactos predeterminada.

#### Agregar y acceder a calendarios basados en archivos ICS en el buzón del usuario
Actualmente, existe una compatibilidad parcial con un calendario basado en una suscripción a calendarios de Internet (ICS):
* Puede agregar un calendario basado en ICS a un buzón de usuario mediante la interfaz de usuario, pero no mediante la API de Microsoft Graph. 
* [Enumerar los calendarios del usuario](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars) le permite obtener las propiedades **name**, **color** e **id** de cada [calendario](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar) en el grupo de calendarios predeterminado del usuario, o en un grupo de calendarios especificado, incluidos los calendarios basados en ICS. No puede almacenar ni acceder a la dirección URL de una ICS en el recurso del calendario.
* También puede [enumerar los eventos](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events) de un calendario basado en ICS.

### Grupos
#### Directiva
Usar Microsoft Graph para crear y nombrar un grupo unificado omite cualquier directiva de grupo unificada que esté configurada a través de Outlook Web App. 

#### Ámbitos de permiso de grupo
Microsoft Graph expone dos ámbitos de permisos (*Group.Read.All* y *Group.ReadWrite.All*) para obtener acceso a las API de grupos.  Estos ámbitos de permisos deben ser aceptados por un administrador (lo que supone un cambio respecto a la versión preliminar).  En el futuro, planeamos agregar nuevos ámbitos para grupos que pueden ser aceptados por los usuarios.

#### Agregar y obtener los datos adjuntos de las publicaciones de grupo
[Agregar](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments) datos adjuntos a las publicaciones de grupo, [enumerar](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments) y obtener los datos adjuntos de las publicaciones de grupo actualmente devuelven el mensaje de error "La solicitud de OData no es compatible". Se ha desarrollado una corrección tanto para la versión `/v1.0` como para la `/beta` y se espera que esté disponible a finales de enero de 2016.

### Contactos
* Solo los contactos personales son compatibles actualmente. Actualmente no se admiten contactos de la organización en `/v1.0`, pero pueden encontrarse en `/beta`.
* No se está devolviendo el teléfono móvil del contacto personal a un contacto. Se agregará en breve. Mientras tanto, puede tener acceso a este a través de las API de Outlook.

### Unidades, archivos y streaming de contenido
* La primera vez que accede a una unidad personal del usuario a través de Microsoft Graph antes de que el usuario acceda a su sitio personal a través del explorador, se produce una respuesta 401.
* La carga y descarga de archivos (archivos de grupos de Office, unidades o datos adjuntos de archivos del correo) está limitada a 4 MB.

### La funcionalidad solo está disponible en las API REST de Office 365 existentes
#### Sincronización
Las funcionalidades de sincronización de Outlook, OneDrive y Azure AD (en Azure AD también se conoce como consulta diferencial) no están disponibles en `/v1.0` ni en `/beta`.  Si su aplicación requiere funcionalidades de sincronización, continúe usando las API de REST de Azure AD y Office 365 existentes o explore la nueva versión preliminar de la característica Webhooks que se ofrece a través de Microsoft Graph para eventos, mensajes y contactos.

> **NOTA**:  El objetivo es contrarrestar la diferencia entre las API existentes y Microsoft Graph lo más rápidamente posible, incluida la sincronización.

#### Procesamiento por lotes
El procesamiento por lotes no se admite en Microsoft Graph. Sin embargo, puede usar el punto de conexión en versión beta de Outlook y [las llamadas REST por lotes de Outlook](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests). 

#### Disponibilidad en China
El servicio de Microsoft Graph está operado por 21Vianet (y ahora disponible en China). Consulte [Implementaciones de nube soberana de Microsoft Graph](http://graph.microsoft.io/docs/overview/deployments) para obtener más información, incluidas las restricciones.

#### Funciones y acciones del servicio
`isMemberOf` y `getObjectsById` no están disponibles en Microsoft Graph

### Permisos de Microsoft Graph
Consulte el [tema sobre ámbitos de permiso](http://graph.microsoft.io/docs/authorization/permission_scopes) para obtener la información más reciente sobre los permisos delegados y de aplicación admitidos por Microsoft Graph.  Además, existen las siguientes limitaciones para `v1.0`:

|Permisos |   Tipo de permiso | Limitación |  Alternativa |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| Delegado    | No se puede actualizar el número de teléfono móvil|    Seleccionar también `Directory.AccessAsUser.All`| 
|_User.ReadWrite.All_|  Delegado|  No se puede realizar ninguna operación CRUD en `User` que no sea la actualización de la foto HD del usuario y las propiedades de perfil extendidas| Seleccione también `Directory.ReadWrite.All` o `Directory.AccessAsUser.All` si se requiere la eliminación del usuario.|
|_User.Read.All_|   Aplicación |No se puede realizar ninguna operación de lectura en otros usuarios| Seleccionar también `Directory.Read.All`|
| _User.ReadWrite.All_ |    Aplicación |   No se puede realizar ninguna operación CRUD en `User` que no sea la actualización de la foto HD del usuario y las propiedades de perfil extendidas |    Seleccionar también`Directory.ReadWrite.All`. **NOTA**: La eliminación del usuario no será posible.|
|_Group.Read.All_   | Aplicación | No se pueden enumerar los grupos ni las pertenencias a estos.  Todavía se puede leer el contenido del grupo para los grupos de Office   | Seleccionar también `Directory.Read.All` |
|_Group.ReadWrite.All_  | Aplicación   | No se pueden enumerar grupos ni pertenencias a grupos, crear grupos, actualizar pertenencias a grupos ni eliminar grupos.  Todavía se puede leer y actualizar el contenido del grupo para los grupos de Office.   | Seleccionar también `Directory.ReadWrite.All`. **NOTA**:  La eliminación del grupo no será posible.|

Además, existen las siguientes `/beta` limitaciones:

|Permiso |   Tipo de permiso | Limitación |  Alternativa |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | Delegado | No se puede leer ni actualizar las tareas del planificador de los grupos de Office  | Seleccionar también `Tasks.ReadWrite`|
|_Tasks.ReadWrite_  | Delegado | No se puede leer ni actualizar las tareas del usuario que inició sesión| Seleccionar también `Group.ReadWrite.All`|

### Limitaciones relacionadas con OData
* Limitaciones **$expand**: 
 * No se admite para `nextLink`
 * No se admite para más de un nivel de expansión
 * No se admite con parámetros adicionales (**$filter**, **$select**)
* No se admiten varios espacios de nombres
* Las solicitudes GET en `$ref` y la conversión no se admiten en usuarios, grupos, dispositivos, entidades de servicio y aplicaciones.
* `@odata.bind` no se admite.  Esto significa que los desarrolladores no podrán establecer correctamente `Accepted` o `RejectedSenders` en un grupo.
* `@odata.id` no está presente en las navegaciones de no contención (como los mensajes) cuando se usan metadatos mínimos
* No está disponible el filtrado o la búsqueda de carga de trabajo cruzada. 
* La búsqueda de texto completo (con **$search**) solo está disponible para algunas entidades, como los mensajes.

  >  Su opinión es importante para nosotros. Conecte con nosotros en [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Etiquete sus preguntas con [MicrosoftGraph] y [office365].

  
             .

