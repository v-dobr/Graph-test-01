# Llamar a Microsoft Graph en una aplicación iOS

En este artículo, veremos las tareas mínimas necesarias para conectar la aplicación a Office 365 y llamar a la API de Microsoft Graph. Usamos el código de [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) para explicar los conceptos principales que debe implementar en la aplicación. Los ejemplos tratan los elementos básicos para autenticar con Microsoft Azure Active Directory (AAD) y realizar una simple llamada de servicio en el servicio de correo de Office 365 mediante la API de Microsoft Graph (enviar un correo). Se recomienda clonar o descargar el proyecto desde este repositorio como complemento de este artículo. 


Este artículo hace referencia a la [Biblioteca de autenticación de Microsoft Azure Active Directory (ADAL) para iOS y OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc) y el ejemplo [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) se autentica usando esta biblioteca. Consulte este repositorio para obtener más información acerca del uso y la implementación de su proyecto de iOS.


## Información general

Para llamar a la API de Microsoft Graph, su aplicación iOS debe completar lo siguiente:

1. Registrar la aplicación de Azure Active Directory (AD) de Microsoft.
2. Solicitar y obtener un token de acceso de Azure AD.
3. Usar el token de acceso en una solicitud REST a la API de Microsoft Graph. 



## Registrar la aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, registre la aplicación y establezca los permisos para usar los servicios de Microsoft Graph. Con la [herramienta de registro de aplicaciones](https://dev.office.com/app-registration) puede registrar en pocos pasos su aplicación para que tenga acceso a la cuenta profesional o educativa de un usuario. Para administrarla, vaya al [Portal de administración de Microsoft Azure](https://manage.windowsazure.com).

Si prefiere registrar manualmente la aplicación, vea la sección **Register your native app with the Azure Management Portal** (Registrar una aplicación nativa con el Portal de administración de Azure) del artículo [Manually register your app with Azure AD so it can access Office 365 APIs](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually) (Registrar manualmente la aplicación con Azure AD para que tenga acceso a las API de Office 365) para obtener instrucciones sobre cómo hacerlo. Tenga en cuenta lo siguiente:

* Para el registro necesitará proporcionar un URI de redireccionamiento. Esto es un valor requerido que especifica dónde se redirigirá a un usuario después de un intento de autenticación correcto. Si no especifica el URI de redireccionamiento correcto, la solicitud de autenticación no se podrá realizar.
* En el registro, se debe conceder a la aplicación el permiso **Enviar correo como usuario con sesión iniciada** para **Microsoft Graph**.  


Anote los siguientes valores de la página **Configurar** de la aplicación de Azure.

* Identificador de cliente
* Redirigir URI

Necesita estos valores para configurar el flujo de OAuth en su aplicación. 

## Solicitar y adquirir un token de acceso de Azure AD

Para solicitar y adquirir un token de acceso para llamar a la API de Microsoft Graph, puede usar el elemento **acquireAuthTokenWithResource:clientId:redirectUri:completionBlock:** que le ha proporcionado la [Biblioteca de autenticación de Microsoft Azure Active Directory (ADAL) para iOS y OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc). Este SDK proporciona a su aplicación la funcionalidad completa de Microsoft Azure AD, que incluye el soporte técnico del protocolo estándar del sector para OAuth2, la integración de la API web con el consentimiento del usuario y el soporte técnico para la autenticación en dos fases.

Este método toma los parámetros siguientes:

1. **resourceID**: este es el recurso requerido al que desea tener acceso. Por ejemplo, si queremos tener acceso a la API de Microsoft Graph, este valor será "https://graph.microsoft.com".
2. **clientID**: valor que proporcionó para identificar su aplicación cuando completó el registro de AAD.
3. **redirectURI**: de nuevo, este es un valor necesario que especifica dónde se redirigirá al usuario después de un intento de autenticación correcto.


En primer lugar, necesitará especificar un contexto de autenticación. Esto simplemente define la autoridad desde donde desea obtener su token de acceso. En nuestro caso es desde un inquilino de AAD y necesitará manifestarlo:

    @property (strong,    nonatomic) ADAuthenticationContext *context;

A continuación, inicialícelo con la ubicación de la autoridad ("https://login.microsoftonline.com/common"):

    self.context = [ADAuthenticationContext authenticationContextWithAuthority:self.authority]; 


En el ejemplo [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample), creamos una clase de autenticación singleton (**AuthenticationManager**) para propósitos de demostración que se inicializa con la autoridad y los parámetros requeridos. De nuevo, esta clase es simplemente un ejemplo acerca de cómo enfocar el flujo de trabajo de autenticación. Segmento de código de interés: 



    - (void)acquireAuthTokenWithResource:(NSString *)resourceID
                            clientID:(NSString*)clientID
                         redirectURI:(NSURL*)redirectURI
                          completion:(void (^)(ADAuthenticationError *error))completion {
    
    NSLog(@"acquireAuthTokenWithResource");
    [self.context acquireTokenWithResource:resourceID
                                  clientId:clientID
                               redirectUri:redirectURI
                           completionBlock:^(ADAuthenticationResult *result) {
                               NSLog(@"Completion");
                               
                               if (result.status !=AD_SUCCEEDED){
                                   NSLog(@"error");
                                   completion(result.error);
                               }
                               
                               else{
                                   NSLog(@"complete!");
                                   self.accessToken = result.accessToken;
                                   self.userID = result.tokenCacheStoreItem.userInformation.userId;
                                   completion(nil);
                               }
                           }];


La primera vez que se ejecute esta aplicación, el administrador de autenticación enviará una solicitud a la autoridad, que le redirigirá a una página de inicio de sesión. Proporcionará sus credenciales y la respuesta contendrá el resultado de autenticación. Si se realiza correctamente, también contendrá los tokens de actualización y acceso. La segunda vez que se ejecute esta aplicación, y suponiendo que no borró las cookies ni la caché de su token, el Administrador de autenticación usará el token de acceso o de actualización en la caché para autenticar las solicitudes del cliente. Esto dará como resultado una llamada al servicio si necesita obtener un token de acceso. 


## Usar el token de acceso en una solicitud a la API de Microsoft Graph

El token de acceso permite que su aplicación cree solicitudes autenticadas en la API de Microsoft Graph. Su aplicación debe anexar el token de acceso al encabezado de solicitud HTTP en **Autorización**.

El ejemplo [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) envía un correo electrónico usando el punto de conexión sendMail en la API de Microsoft Graph. De nuevo, en nuestro ejemplo creamos una clase de autenticación singleton (AuthenticationManager) que se inicializa con el token de acceso. Necesitaremos el token de acceso para crear nuestra solicitud.



    - (void)sendMailREST {
    
    AuthenticationManager *authManager = [AuthenticationManager sharedInstance];

    //Helper method used to construct the email message
    NSData *postData = [self mailContent];
    
    //Microsoft Graph API endpoint for sending mail
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:@"https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail"]];

    [request setHTTPMethod:@"POST"];
    [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    [request setValue:@"application/json, text/plain, */*" forHTTPHeaderField:@"Accept"];
    
    // Access token required for request header
    NSString *authorization = [NSString stringWithFormat:@"Bearer %@", authManager.accessToken];
    [request setValue:authorization forHTTPHeaderField:@"Authorization"];
    [request setHTTPBody:postData];

    NSURLConnection *conn = [[NSURLConnection alloc] initWithRequest:request delegate:self];
    
    if(conn) {
        NSLog(@"Connection Successful");
    } else {
        NSLog(@"Connection could not be made");
    }
    
    [conn start];

Como puede ver, la respuesta se controla con los delegados NSURLConnection. Más concretamente, con NSURLConnectionDelegate y NSURLConnectionDataDelegate.

## Pasos siguientes

Si el token de acceso ha expirado o está a punto de expirar, puede usar el **acquireTokenSilentWithResource:clientId:redirectUri:completionBlock:** de ADAuthenticationContext para adquirir un nuevo token de acceso. Puede consultarse su uso en el ejemplo [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample). Allí, también encontrará el código para borrar la caché del token y las cookies almacenadas.  

La API de Microsoft Graph es una interfaz unificadora muy potente que permite trabajar con todos los tipos de datos de Microsoft. Consulte la [referencia de la API](http://graph.microsoft.io/docs/api-reference/v1.0) para ver todas las posibilidades que ofrece la API de Microsoft Graph.

