# Llamar a Microsoft Graph en una aplicación de servicio o demonio

En este artículo veremos las tareas mínimas necesarias para conectar una aplicación de servicio o demonio de inquilino único a Office 365 y llamar a la API de Microsoft Graph.

## Información general

Para llamar a la API de Microsoft Graph en una aplicación de servicio o demonio, deben completarse las tareas siguientes.

1. Registrar la aplicación en Azure Active Directory
2. Solicitar un token de acceso al punto de conexión que emite los tokens
3. Usar el token de acceso en una solicitud a la API de Microsoft Graph

## Registrar la aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, registre la aplicación y establezca los permisos para usar los servicios de Microsoft Graph. Con tan solo unos clics, puede registrar su aplicación mediante la [herramienta de registro de aplicaciones](https://dev.office.com/app-registration). Para administrarla, vaya al [Portal de administración de Microsoft Azure](https://manage.windowsazure.com).

Si prefiere registrar manualmente la aplicación, consulte la sección [Registrar una aplicación de servidor web con el Portal de administración de Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) y tenga en cuenta lo siguiente:

* Después de registrar la aplicación, configure los **permisos de la aplicación** requeridos por la aplicación de servicio o demonio.

Anote los siguientes valores de la página Configurar de la aplicación de Azure porque los necesitará para configurar el flujo de OAuth en su aplicación de servicio o demonio.

* Id. de cliente (exclusivo de la aplicación)
* Una clave de aplicación (exclusiva de la aplicación)
* El punto de conexión del token de OAuth 2.0 de su aplicación
  * Para ver este valor, haga clic en *Ver extremos* en la parte inferior del Portal de administración de Azure, en la página de su aplicación. El punto de conexión será algo similar a `https://login.microsoftonline.com/<tenantId>/oauth2/token`.

## Solicitar un token de acceso desde el token que emite un punto de conexión

A diferencia de las aplicaciones cliente, la aplicación de servicio o demonio no puede tener un inicio de sesión de usuario ni puede autorizar su aplicación. En lugar de suplantar a un usuario, la aplicación tiene que implementar el flujo de concesión de credenciales de cliente de OAuth 2.0; este flujo le permitirá usar sus propias credenciales, su identificador de cliente y una clave de aplicación con los que se autenticará al llamar a Microsoft Graph. Vea [Autorización del acceso a aplicaciones web mediante OAuth 2.0 y Azure Active Directory](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx) para obtener más información sobre el flujo de autenticación.

Realice una solicitud POST de HTTP en el punto de conexión que emite los tokens con los siguientes parámetros, reemplazando `<clientId>` y `<clientSecret>` con el identificador de cliente de la aplicación y la clave de la aplicación, respectivamente.

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

La respuesta incluirá un acceso token e información de expiración.

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

## Usar el token de acceso en una solicitud a la API de Microsoft Graph

El token de acceso permite que su aplicación cree solicitudes autenticadas en la API de Microsoft Graph. Su aplicación debe anexar el token de acceso al encabezado de **autorización** de cada solicitud.

Por ejemplo, una aplicación de servicio o demonio puede recuperar todos los usuarios de un inquilino si tiene el permiso *Leer los perfiles completos de todos los usuarios* seleccionado en el Portal de administración de Azure. 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```

Microsoft Graph es una API unificadora muy potente que permite trabajar con todos los tipos de datos de Microsoft. Consulte la [referencia de la API](http://graph.microsoft.io/docs/api-reference/v1.0) para ver todas las posibilidades que ofrece la API de Microsoft Graph.
