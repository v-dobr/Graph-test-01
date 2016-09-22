# Llamar a Microsoft Graph en una aplicación PHP 

En este artículo, veremos las tareas mínimas necesarias para obtener un token de acceso de Azure Active Directory (AD) y llamar a la API de Microsoft Graph. Para explicar los conceptos principales que se deben implementar en la aplicación, usaremos el código del [ejemplo Connect de PHP para Office 365 con Microsoft Graph](https://github.com/microsoftgraph/php-connect-rest-sample).

![Captura de pantalla del ejemplo Connect de PHP para Office 365](./images/web-screenshot.png)

## Información general

Para llamar a la API de Microsoft Graph, su aplicación PHP debe completar las siguientes tareas.

1. Registrar la aplicación en Azure Active Directory
2. Redirigir el explorador a la página de inicio de sesión
3. Recibir un código de autorización en la página de la dirección URL de respuesta
4. Solicitar un token de acceso desde el punto de conexión del token
5. Usar el token de acceso en una solicitud a la API de Microsoft Graph

<!--<a name="register"/>-->
## Registrar la aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, registre la aplicación y establezca los permisos para usar los servicios de Microsoft Graph. Con la [herramienta de registro de aplicaciones](https://dev.office.com/app-registration) puede registrar en pocos pasos su aplicación para que tenga acceso a la cuenta profesional o educativa de un usuario. Para administrarla, vaya al [Portal de administración de Microsoft Azure](https://manage.windowsazure.com).

Si prefiere registrar manualmente la aplicación, consulte la sección [Registrar una aplicación de servidor web con el Portal de administración de Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) y tenga en cuenta lo siguiente:

* Especifique una página en su aplicación PHP como **dirección URL de inicio de sesión** en el paso 6. En el caso del ejemplo Connect, esta página es [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php).
* [Configure los **permisos delegados**](https://github.com/microsoftgraph/php-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) que requiera su aplicación. En el ejemplo Connect, se requiere el permiso **Enviar correo como usuario con sesión iniciada**.

Anote los siguientes valores de la página **Configurar** de la aplicación de Azure.

* Identificador de cliente
* Una clave válida
* Una dirección URL de respuesta

Necesita estos valores como parámetros en el flujo de OAuth en su aplicación.

<!--<a name="redirect"/>-->
## Redirigir al explorador a la página de inicio de sesión

Su aplicación necesita redirigir el explorador a la página de inicio de sesión para obtener un código de autorización y continuar el flujo de OAuth.

En el ejemplo Connect, el código que redirige al explorador está en la función [`AuthenticationManager.connect`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L41).

```php
// Redirect the browser to the authorization endpoint. Auth endpoint is
// https://login.microsoftonline.com/common/oauth2/authorize
$redirect = Constants::AUTHORITY_URL . Constants::AUTHORIZE_ENDPOINT . 
            '?response_type=code' . 
            '&client_id=' . urlencode(Constants::CLIENT_ID) . 
            '&redirect_uri=' . urlencode(Constants::REDIRECT_URI);
header("Location: {$redirect}");
exit();
```

> **Nota:** <br />
> Debe enviar el encabezado de **ubicación** antes de escribir ningún resultado en la página.

<!--<a name="authcode"/>-->
## Recibir un código de autorización en la página de la dirección URL de respuesta

Cuando el usuario inicia sesión, el flujo devuelve el explorador a la dirección URL de respuesta en su aplicación. Azure anexa un código de autorización a la cadena de consulta. En el ejemplo Connect, se usa la página [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php) para este propósito.

El código de autorización se proporciona en la variable de la cadena de consulta `code`. En el ejemplo Connect, se guarda el código en una variable de la sesión para usarlo más adelante.

```php
if (isset($_GET['code'])) {
    $_SESSION['code'] =  $_GET['code'];
}
```

<!--<a name="accesstoken"/>-->
## Solicitar un token de acceso desde el punto de conexión del token

Cuando tenga el código de autorización, podrá usarlo junto con los valores de los parámetros ID de cliente, Clave y Dirección URL de respuesta que obtuvo en Azure AD para solicitar un token de acceso. 

> **Nota:** <br />La solicitud debe especificar también un recurso que estemos intentando usar. En el caso de la API de Microsoft Graph, el valor de recurso es `https://graph.microsoft.com`.

En el ejemplo Connect, se solicita un token que use el código de la función [`AuthenticationManager.acquireToken`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L62). Este es el código más relevante.

```php
$tokenEndpoint = Constants::AUTHORITY_URL . Constants::TOKEN_ENDPOINT;

// Send a POST request to the token endpoint to retrieve tokens.
// Token endpoint is:
// https://login.microsoftonline.com/common/oauth2/token
$response = RequestManager::sendPostRequest(
    $tokenEndpoint, 
    array(),
    array(
        'client_id' => Constants::CLIENT_ID,
        'client_secret' => Constants::CLIENT_SECRET,
        'code' => $_SESSION['code'],
        'grant_type' => 'authorization_code',
        'redirect_uri' => Constants::REDIRECT_URI,
        'resource' => Constants::RESOURCE_ID
    )

// Store the raw response in JSON format.
$jsonResponse = json_decode($response, true);

// The access token response has the following parameters:
// access_token - The requested access token.
// expires_in - How long the access token is valid.
// expires_on - The time when the access token expires.
// id_token - An unsigned JSON Web Token (JWT).
// refresh_token - An OAuth 2.0 refresh token.
// resource - The App ID URI of the web API (secured resource).
// scope - Impersonation permissions granted to the client application.
// token_type - Indicates the token type value.
foreach ($jsonResponse as $key=>$value) {
    $_SESSION[$key] = $value;
}
```

> **Nota:** <br />
> La respuesta proporciona más información aparte del token de acceso. Por ejemplo, su aplicación puede obtener un token de actualización para solicitar tokens de acceso nuevos sin que el usuario tenga que iniciar sesión expresamente.

Su aplicación PHP ahora puede usar la variable de sesión `access_token` para emitir solicitudes autenticadas a la API de Microsoft Graph.

<!--<a name="request"/>-->
## Usar el token de acceso en una solicitud a la API de Microsoft Graph

El token de acceso permite que su aplicación cree solicitudes autenticadas en la API de Microsoft Graph. Su aplicación debe proporcionar el token de acceso en el encabezado de **autorización** de cada solicitud.

En el ejemplo Connect, se envía un correo electrónico usando el punto de conexión **sendEmail** en la API de Microsoft Graph. El código está en la función [`MailManager.sendWelcomeMail`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/MailManager.php#L40). Este es el código que muestra cómo enviar el código de acceso en el encabezado de autorización.

```php
// Send the email request to the sendmail endpoint, 
// which is in the following URI:
// https://graph.microsoft.com/v1.0/me/microsoft.graph.sendMail
// Note that the access token is attached in the Authorization header
RequestManager::sendPostRequest(
    Constants::RESOURCE_ID . Constants::SENDMAIL_ENDPOINT,
    array(
        'Authorization: Bearer ' . $_SESSION['access_token'],
        'Content-Type: application/json;' . 
                      'odata.metadata=minimal;' .
                      'odata.streaming=true'
    ),
    $email
);
```

> **Nota:** <br />
> La solicitud también debe enviar un encabezado **Content-Type** con un valor que acepte la API de Microsoft Graph. Por ejemplo, `application/json;odata.metadata=minimal;odata.streaming=true`.

La API de Microsoft Graph es una interfaz unificadora muy potente que permite trabajar con todos los tipos de datos de Microsoft. Consulte la referencia de la API para ver todas las posibilidades que ofrece la API de Microsoft Graph.

<!--## Additional resources

-  [Office 365 PHP Connect sample using Microsoft Graph API](https://github.com/OfficeDev/O365-PHP-Unified-API-Connect)-->
