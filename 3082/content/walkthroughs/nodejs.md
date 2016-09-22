# Llamar a Microsoft Graph con una aplicación de Node.js

En este artículo, veremos las tareas mínimas necesarias para conectar la aplicación a Office 365 y llamar a la API de Microsoft Graph. Para explicar los conceptos principales que se deben implementar en la aplicación, usaremos el código del ejemplo [Connect de Node.js para Office 365 con Microsoft Graph](https://github.com/microsoftgraph/nodejs-connect-rest-sample).

![Captura de pantalla del ejemplo Connect de Node.js para Office 365](./images/web-screenshot.png)

## Información general

Para llamar a la API de Microsoft Graph, su aplicación web debe completar las siguientes tareas.

1. Registrar la aplicación en Azure Active Directory 
2. Instalar la biblioteca de cliente de Azure Active Directory para Node
3. Redirigir el explorador a la página de inicio de sesión
4. Recibir un código de autorización en la página de la dirección URL de respuesta
5. Usar `adal-node` para solicitar un token de acceso
6. Realizar una solicitud a la API de Microsoft Graph

<!--<a name="register"/>-->
## Registrar su aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, registre la aplicación y establezca los permisos para usar los servicios de Microsoft Graph. Con la [herramienta de registro de aplicaciones](https://dev.office.com/app-registration) puede registrar en pocos pasos su aplicación para que tenga acceso a la cuenta profesional o educativa de un usuario. Para administrarla, vaya al [Portal de administración de Microsoft Azure](https://manage.windowsazure.com).

Si prefiere registrar manualmente la aplicación, consulte la sección [Registrar una aplicación de servidor web con el Portal de administración de Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) y tenga en cuenta lo siguiente:

* Especifique una página en su aplicación Node.js com **dirección URL de inicio de sesión** en el paso 6. En el caso del ejemplo Connect, la dirección URL es http://localhost:8080/login, que se asigna a la ruta [/login](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L33).
* [Configure los **permisos delegados**](https://github.com/microsoftgraph/nodejs-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) que requiere su aplicación. El ejemplo Connect requiere el permiso **Enviar correo como usuario con sesión iniciada**.

Anote los siguientes valores de la página **Configurar** de la aplicación de Azure.

* Identificador de cliente
* Una clave válida
* Una dirección URL de respuesta

Necesita estos valores como parámetros en el flujo de OAuth en su aplicación.

<!--<a name="adal">-->
## Instalar la biblioteca de cliente de Azure Active Directory para Node

La biblioteca ADAL para Node.js facilita a las aplicaciones de Node.js autenticar a AAD para acceder a los recursos web protegidos de AAD.
Para agregar adal-node a su `package.json` existente, escriba lo siguiente en su terminal preferido.

`npm install adal-node --save`

Para obtener más información acerca de la biblioteca de cliente de adal-node, consulte su información del paquete en [npm](https://www.npmjs.com/package/adal-node). En caso de que tuviera algún problema, necesitara obtener el código fuente o las últimas novedades acerca de las próximas características y correcciones, consulte el proyecto de adal-node en [Github](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs).

<!--<a name="redirect"/>-->
## Redirigir al explorador a la página de inicio de sesión

Su aplicación necesita redirigir al explorador a la página de inicio de sesión para obtener un código de autorización y continuar el flujo de OAuth 2.0.

En el ejemplo Connect, la dirección URL de autenticación de [`authHelper.js#getAuthUrl`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js#L17) se redirige mediante la función [`login.hbs#login`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/views/login.hbs#L2) a través del evento `onclick` del lado del cliente.

**authHelper.js#getAuthUrl**
```javascript
/**
 * Generate a fully formed uri to use for authentication based on the supplied resource argument
 * @return {string} a fully formed uri with which authentcation can be completed
 */
function getAuthUrl() {
    return credentials.authority + "/oauth2/authorize" +
        "?client_id=" + credentials.client_id +
        "&response_type=code" +
        "&redirect_uri=" + credentials.redirect_uri;
};
```

**login.hbs#login**
```javascript
function login() {
    window.location = '{{auth_url}}'.replace(/&amp;/g, '&'); // transform HTML special char from .hbs template rendering
}
```

<!--<a name="authcode"/>-->
## Recibir un código de autorización en la página de la dirección URL de respuesta

Cuando el usuario inicia sesión, el flujo devuelve el explorador a la dirección URL de respuesta en su aplicación. El código de autorización se proporciona en la variable de la cadena de consulta `code`.

```javascript
router.get('/<application reply url>', function (req, res, next) {
  var authCode = req.query.code;
  // your router's implementation
});
```

Consultar el [código relevante](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L34) en el ejemplo Connect

<!--<a name="accesstoken"/>-->
## Usar `adal-node` para solicitar un token de acceso

Ahora que nos hemos autenticado con Azure Active Directory, el siguiente paso es adquirir un token de acceso mediante adal-node. Una vez hecho eso, estaremos preparados para realizar solicitudes REST a la API de Microsoft Graph.

Para solicitar un token de acceso, adal-node proporciona dos funciones de devolución de llamada.

|                          Función                         |                                      Params                                      | Descripción                                                                                             |
|:---------------------------------------------------------:|:--------------------------------------------------------------------------------:|---------------------------------------------------------------------------------------------------------|
| `AuthenticationContext.acquireTokenWithAuthorizationCode` | `authCode`, `redirect_uri`, `resource`, `client_id`, `client_secret`, `callback` | proporciona un token de acceso para un recurso específico basado en el código de autorización devuelto durante el inicio de sesión |
| `AuthenticationContext.acquireTokenWithRefreshToken`      | `token`, `client_id`, `client_secret`, `resource`, `callback`                    | proporciona un token de acceso para un recurso específico a partir de un token de actualización                             |

En el ejemplo Connect, las solicitudes se redirigen a través de [`authHelper.js`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js) de forma que `client_id` y `client_secret` pueden agregarse.

```javascript
// The application registration (must match Azure AD config)
var credentials = {
    authority: "https://login.microsoftonline.com/common",
    client_id: "<your client id here>",
    client_secret: "<your client secret>",
    redirect_uri: "http://localhost:8080/login"
};

/**
 * Gets a token for a given resource.
 * @param {string} code An authorization code returned from a client.
 * @param {string} res A URI that identifies the resource for which the token is valid.
 * @param {AcquireTokenCallback} callback The callback function.
 */
function getTokenFromCode(res, code, callback) {
    var authContext = new AuthenticationContext(credentials.authority);
    authContext.acquireTokenWithAuthorizationCode(code, credentials.redirect_uri, res, credentials.client_id, credentials.client_secret, function (err, response) {
        if (err) {
            callback(null);
        }
        else {
            callback(response);
        }
    });
};
```

<!--<a name="request"/>-->
## Realizar una solicitud a la API de Microsoft Graph

Para poder identificar nuestras solicitudes a la API de Graph, estas deben firmarse con un encabezado `Authorization` que contenga el token de acceso para cualquier recurso de servicio web que solicitemos. Un encabezado de autorización correctamente formado incluirá el token de acceso de adal-node y tomará la forma que se indica a continuación.

`Authorization: Bearer <access token>`

Con `adal-node`, combinado con nuestra lógica de autenticación de la sección anterior, podemos usar ahora nuestro token de acceso para firmar solicitudes.

```javascript
/* GET home page. */
router.get('/<application reply url>', function (req, res, next) {
    var authCode = req.query.code;
    authHelper.getTokenFromCode('https://graph.microsoft.com/', req.query.code, function (token) {
        if (token !== null) {
            // Use this token to sign requests
            var headers = {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer ' + token
                };
            // request implementation...
        } else {
            // error handling
        }
    });
});
```

Microsoft Graph es una API unificadora muy potente que permite trabajar con todos los tipos de datos de Microsoft. Consulte la [referencia de la API](http://graph.microsoft.io/docs/api-reference/v1.0) para ver todas las posibilidades que ofrece la API de Microsoft Graph.

<!--## Additional resources

- [Office 365 Node.js Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Nodejs-Unified-API-Connect)-->

