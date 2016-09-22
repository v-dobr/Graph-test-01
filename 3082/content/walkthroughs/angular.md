# Llamar a Microsoft Graph desde una aplicación de Angular 

En este artículo, veremos las tareas mínimas necesarias para conectar la aplicación a Office 365 y llamar a la API de Microsoft Graph. Para explicar los conceptos principales que se deben implementar en la aplicación, usaremos el código del ejemplo [Connect de Angular para Office 365 con Microsoft Graph](https://github.com/microsoftgraph/angular-connect-rest-sample).

![Captura de pantalla de ejemplo Connect de Angular de Office 365](./images/web-screenshot.png)

## Requisitos previos  

En este tema, se da por supuesto que usted:

* está familiarizado con la lectura de código JavaScript y [AngularJS](https://angularjs.org/).

## Información general

Para llamar a la API de Microsoft Graph, deben completarse las tareas siguientes.

1. Registrar la aplicación en Azure Active Directory
2. Configurar la biblioteca de Azure Active Directory para JavaScript (ADAL JS)
3. Obtener un token de acceso mediante ADAL JS
4. Usar el token de acceso en una solicitud a la API de Microsoft Graph

<!--<a name="register"></a>-->
## Registrar la aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, registre la aplicación y establezca los permisos para usar los servicios de Microsoft Graph. Con la [herramienta de registro de aplicaciones](https://dev.office.com/app-registration) puede registrar en pocos pasos su aplicación para que tenga acceso a la cuenta profesional o educativa de un usuario. Para administrarla, vaya al [Portal de administración de Microsoft Azure](https://manage.windowsazure.com).

Si prefiere registrar manualmente la aplicación, consulte [Registrar una aplicación basada en explorador web con el Portal de administración de Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp) para obtener instrucciones y tenga en cuenta lo siguiente:

* Asegúrese de especificar http://127.0.0.1:8080/ como **dirección URL de inicio de sesión**.
* Después de registrar la aplicación, [configure los **permisos delegados**](https://github.com/microsoftgraph/angular-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) requeridos por la aplicación de Angular. El ejemplo Connect requiere el permiso **Enviar correo como usuario con sesión iniciada**.

Anote los siguientes valores de la página **Configurar** de la aplicación de Azure porque los necesitará para configurar [ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) en su aplicación de Angular.

* Id. de cliente (exclusivo de la aplicación)
* Dirección URL de respuesta (http://127.0.0.1:8080/)

<!--<a name="adal"></a>-->
## Configurar la biblioteca de Azure Active Directory para JavaScript (ADAL JS)

[ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) es una biblioteca JavaScript que proporciona soporte completo para el inicio de sesión de los usuarios de Azure AD en las aplicaciones de página única (SPA), tales como el ejemplo Connect y la administración de tokens, entre otras características. Para poder aprovechar esta biblioteca, debe estar incluida y configurada en la aplicación de Angular.

Simplemente incluya la biblioteca y su módulo específico para Angular mediante Microsoft CDN.

```html
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal.min.js"></script>
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal-angular.min.js"></script>
```

A continuación, configure el servicio ADAL JS donde quiera configurar las dependencias de la aplicación de Angular. En el ejemplo Connect, su configuración se realiza en [*public/app.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/scripts/app.js). 

Para configurar JS ADAL, primero debe incluir una referencia al módulo ADAL. Para ello, agregue ```AdalAngular``` a la matriz de su módulo y pase ```adalAuthenticationServiceProvider``` a la función ```config```. Configure la biblioteca con la función ```init```; pásele el ID de cliente de la aplicación y un objeto ```endpoints``` con el que se declaren las API a las que la aplicación de Angular debe efectuar las solicitudes CORS.

```javascript
// Initialize the ADAL provider with your clientID (found in the Azure Management Portal) and 
// the API URL (to enable CORS requests).
adalAuthenticationServiceProvider.init(
  {
    clientId: clientId,
    // The endpoints here are resources for cross origin requests.
    endpoints: {
      'https://graph.microsoft.com': 'https://graph.microsoft.com'
    }
  },
  $httpProvider
);
```

<!--<a name="accessToken"></a>-->
## Obtener un token de acceso mediante ADAL JS

La aplicación debe redirigir el explorador a una página de inicio de sesión para que el usuario pueda iniciar sesión y conceder a la aplicación acceso a sus datos. En el ejemplo Connect, se usa ADAL JS para controlar esta tarea. 

En uno de los controladores de la aplicación, primero agregue una referencia al servicio ADAL: inserte ```adalAuthenticationService``` en el controlador y, a continuación, defina una función que use la función ```login``` del servicio a la que pueda llamar la interfaz de usuario. En el ejemplo Connect, esta acción se realiza en el archivo [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js). 

```javascript
/**
  * Expose the login method from ADAL to the view.
  */
function connect() {
  adalAuthenticationService.login();
};
```

Cuando se llame a esta función, la aplicación redirigirá al usuario a una página de inicio de sesión. Después de iniciar sesión y autorizar la aplicación, el usuario volverá a la aplicación con el token de acceso en la cadena de consulta que ADAL JS recuperará y almacenará. 

<!--<a name="request"></a>-->
## Usar el token de acceso en una solicitud a la API de Microsoft Graph

El token de acceso permite a su aplicación efectuar solicitudes autenticadas en la API de Microsoft Graph. De manera automática, ADAL JS interceptará todas las solicitudes HTTP y les agregará el token de acceso, por lo que no le hará falta establecer manualmente ese encabezado al usar la biblioteca. 

En el ejemplo Connect, se envía un correo electrónico usando el punto de conexión ```me/sendMail``` de la API de Microsoft Graph que se incluye en el archivo [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js). 

Microsoft Graph es una API unificadora muy potente que permite trabajar con todos los tipos de datos de Microsoft. Consulte la [referencia de la API](http://graph.microsoft.io/docs/api-reference/v1.0) para ver todas las posibilidades que ofrece la API de Microsoft Graph.

