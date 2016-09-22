# Introducción a Microsoft Graph y REST

En este artículo se describe cómo llamar a Microsoft Graph para recuperar mensajes de correo electrónico en Office 365 y Outlook.com. Este artículo se centra en las solicitudes y respuestas de OAuth y REST. Cubre la secuencia de solicitudes y respuestas que una aplicación usa para autenticar y recuperar mensajes.

## Usar OAuth 2.0 para autenticar

Para llamar a Microsoft Graph, su aplicación necesita un token de acceso de Azure Active Directory (Azure AD). En el ejemplo siguiente, la aplicación implementa el flujo de concesión del código de autorización para obtener los tokens de acceso desde Azure AD, siguiendo los [protocolos de OAuth 2.0](http://tools.ietf.org/html/rfc6749) estándar.

### Registrar una aplicación

Actualmente, hay dos opciones para registrar su aplicación:

  1. Registre una aplicación con el modelo que admita usuarios comerciales de Office 365, solo cuentas profesionales o educativas.
 
  Este modelo solo funciona con ofertas comerciales de Office 365. Una vez que haya registrado su aplicación, puede administrarla a través del [Portal de administración de Azure](https://manage.windowsazure.com).

  2. Regístrese mediante la última funcionalidad que funciona para los servicios de cliente y comerciales de Office 365 (denominada punto de conexión de autenticación v2.0).
 
  Ahora se encuentra disponible un único servicio de autenticación tanto para las cuentas personales, profesionales o educativas. Este modelo proporciona un solo servicio de autenticación tanto para identidades profesionales o educativas (Azure AD) como personales (Microsoft). Ahora solo necesita implementar un flujo de autenticación en su aplicación para permitir que los usuarios usen tanto las cuentas profesionales como educativas, como Office 365 o OneDrive para la Empresa, o cuentas personales como Outlook.com o OneDrive.
   
Use el [Portal de registro de aplicaciones](https://apps.dev.microsoft.com/) para registrar su aplicación y admitir tanto las cuentas personales como las profesionales y educativas.

Tenga en cuenta de el punto de conexión v2.0 está creciendo gradualmente para cubrir todos los escenarios del punto de conexión de autenticación anterior. Para decidir si es adecuado, lea [este artículo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).

Después del registro, obtendrá un secreto y un identificador de cliente. Estos valores se usan en el flujo de concesión del código de autorización.

El resto de este documento supone un registro en el modelo v2.0. Para obtener una guía completa de los flujos compatibles con el punto de conexión v2.0 vea [este artículo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/). Para obtener una guía completa del flujo de concesión del código de autorización, vea [este artículo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/).

### Obtener un código de autorización

El primer paso en el flujo de concesión del código de autorización es obtener un código de autorización. Ese código se devuelve a la aplicación mediante el servidor de autorización cuando el usuario inicia sesión y consiente el nivel de acceso que ha solicitado la aplicación.

En primer lugar, la aplicación crea una dirección URL de inicio de sesión para el usuario. Esta dirección URL debe estar abierta en un explorador para que el usuario pueda iniciar sesión y proporcionar su consentimiento.

La dirección URL base para el inicio de sesión es similar a `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`.

La aplicación anexa parámetros de consulta a esta dirección URL base para permitir que el servidor de autorización conozca qué aplicación está solicitando el inicio de sesión y qué permisos está solicitando.

- `client_id` - El Id. de cliente generado al registrar la aplicación. Esto permite que Azure AD conozca qué aplicación está solicitando el inicio de sesión.
- `redirect_uri` - La ubicación a la que Azure redirigirá cuando el usuario haya concedido el consentimiento a la aplicación. Este valor debe corresponder al valor de **URI de redireccionamiento** usado al registrar la aplicación.
- `response_type` - El tipo de respuesta que espera la aplicación. Este valor es `code` para el flujo de concesión del código de autorización.
- `scope`: una lista separada por espacios de los ámbitos que está solicitando la aplicación. Puede obtener más información en [este artículo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/).
- `state`: un valor que se incluye en la solicitud que también se devolverá en la respuesta de token.

Por ejemplo, la dirección URL de solicitud para nuestra aplicación que requiere acceso de lectura al correo sería similar a la siguiente.

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<CLIENT ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
```

A continuación, redirija al usuario a la dirección URL de inicio de sesión. El usuario aparecerá con un signo en la pantalla que muestra el nombre de la aplicación. Después de que inicien sesión, al usuario se le presentarán un listado de permisos que la aplicación requieren y se le pedirá que los permita o los deniegue. Suponiendo que permiten el acceso requerido, el explorador se redirigirá al URI de redireccionamiento especificado en la solicitud inicial con el código de autorización en la cadena de consulta.

```http
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

Si también está usando OpenId Connect para el inicio de sesión único, se requieren parámetros adicionales. Vea [este artículo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/) para obtener más información. 

El paso siguiente consiste en intercambiar el código de autorización devuelto por un token de acceso.

### Obtener un token de acceso

Para obtener un token de acceso, la aplicación publica los parámetros del formulario codificado a la dirección URL de solicitud del token (`https://login.microsoftonline.com/common/oauth2/v2.0/token`) con los siguientes parámetros.

- `client_id`: El Id. de cliente generado al registrar la aplicación.
- `client_secret`: El secreto de cliente generado al registrar la aplicación.
- `code`: El código de autorización que se obtuvo en el paso anterior.
- `redirect_uri`: Este valor debe ser el mismo que el valor usado en la solicitud del código de autorización.
- `grant_type`: El tipo de concesión que está usando la aplicación. Este valor es `code` para el flujo de concesión del código de autorización.
- `scope`: una lista separada por espacios de los ámbitos que está solicitando la aplicación. Puede obtener más información en [este artículo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/).

La dirección URL de solicitud para nuestra aplicación, usando el código del paso anterior, tendría el siguiente aspecto.

```http
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
  &client\_id=<CLIENT ID>
  &client\_secret=<CLIENT SECRET>
}
```

El servidor responde con una carga JSON que incluye el token de acceso.

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

El token de acceso se encuentra en el campo `access_token` de la carga JSON. La aplicación usa este valor para establecer el encabezado de autorización al realizar las llamadas REST a la API.

## Llamar a Microsoft Graph

Cuando la aplicación tenga un token de acceso, estará lista para llamar a Microsoft Graph. Dado que esta aplicación de ejemplo está recuperando mensajes, usará una solicitud HTTP GET al punto de conexión `https://graph.microsoft.com/v1.0/me/messages`.

### Refinar la solicitud

Las aplicaciones pueden controlar el comportamiento de las solicitudes GET usando parámetros de consulta OData. Se recomienda que las aplicaciones usen estos parámetros para limitar el número de resultados que se devuelven y los campos que se devuelven para cada elemento. 

Esta aplicación de ejemplo mostrará los mensajes en una tabla que muestra el asunto, el remitente y la fecha y hora en que se recibió el mensaje. La tabla muestra un máximo de 25 filas y está ordenada de forma que los mensajes recibidos recientemente se encuentren en la parte superior. La aplicación usa los siguientes parámetros de consulta para obtener estos resultados.

- `$select` - Especifica solo los campos `subject`, `sender`, y `dateTimeReceived`.
- `$top` - Especifica un máximo de 25 elementos.
- `$orderby` - Ordena los resultados por el campo `dateTimeReceived`.

Esto da como resultado la siguiente solicitud.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

Ahora que ha visto cómo realizar llamadas a Microsoft Graph, puede usar la referencia de la API para construir cualquier otro tipo de llamada que su aplicación necesite realizar. Sin embargo, tenga en cuenta que su aplicación debe tener los permisos adecuados configurados en el registro de aplicación para las llamadas que realice.


