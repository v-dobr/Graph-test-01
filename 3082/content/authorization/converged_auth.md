# Autenticar los puntos de conexión de Microsoft Graph con el punto de conexión de autenticación v2


<!--
### Preview documentation
There are features and functionality of the converged authentication model that are not yet supported in the public preview period. You should be aware of them if you are building applications during the public preview. For more information, see [Limitations and restrictions of the converged authentication model preview](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).
-->

## Iniciar sesión en la cuenta de Microsoft y usuarios de AD de Azure con un único modelo de autenticación

Al usar el punto de conexión de autenticación v2, puede crear aplicaciones que acepten tanto identidades profesionales o educativas (Azure AD) como personales (cuenta Microsoft).

En el pasado, si deseaba desarrollar una aplicación que fuera compatible tanto con las cuentas de Microsoft como con las de Azure Active Directory, necesitaba integrarlas en dos sistemas completamente independientes. Ahora, puede crear aplicaciones compatibles con ambos tipos de cuenta usando una única integración mediante el punto de conexión de autenticación v2. Un único proceso que le permite llegar inmediatamente a un público que abarca millones de usuarios tanto con cuentas personales como profesionales o educativas.   

Una vez que hayan integrado sus aplicaciones con el extremo de autenticación v2, podrán obtener acceso de forma instantánea a los puntos de conexión disponibles de Microsoft Graph para las cuentas personales, profesionales o educativas. Por ejemplo: 

| Datos              | Punto de conexión                                       |
|:------------------|:-----------------------------------------------|
| Perfil de usuario      | `https://graph.microsoft.com/v1.0/me`          |
| Correo de Outlook      | `https://graph.microsoft.com/v1.0/me/messages` |
| Contactos de Outlook  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Calendarios de Outlook | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**Nota:** Algunos puntos de conexión de Microsoft Graph, como los grupos y las tareas, no se aplican a las cuentas personales.  

## Ámbitos de autenticación de la API de Microsoft Graph

El punto de conexión de autenticación v2 admite todos los ámbitos de permiso enumerados para usar con la autenticación de Azure AD en el tema [Microsoft Graph permission scopes](permission_scopes.md) (Ámbitos de permiso de Microsoft Graph). Sin embargo, el punto de conexión de autenticación v2 en estos momentos no admite ámbitos solo de la aplicación.

>**Nota**: Actualmente, se requiere pasar la dirección URL del recurso "https://graph.microsoft.com" como prefijo de la cadena del ámbito. Por ejemplo, para usar el ámbito `Files.Read`, debería especificarlo como `https://graph.microsoft.com/Files.Read`.

Para más información sobre el uso de ámbitos con el punto de conexión de autenticación v2 y cómo se diferencia del uso de recursos en Azure AD, vea [Ámbitos, no recursos](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources).

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


## Pasos siguientes

[Cómo registrar una aplicación con el punto de conexión v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/)

## Obtener más información

[Novedades sobre el punto de conexión de autenticación v2](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare)

[Limitaciones y restricciones en el modelo de autenticación v2](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

[Documentación de punto de conexión de autenticación de Microsoft Azure v2](https://azure.microsoft.com/en-us/documentation/articles/?service=active-directory&term=app+model+v2.0)
