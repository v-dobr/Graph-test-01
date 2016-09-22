# Llamar a Microsoft Graph en una aplicación de Windows 10 universal

En este artículo veremos las tareas mínimas necesarias para obtener un token de acceso de Azure Active Directory (AD) y llamar a Microsoft Graph. Para explicar los conceptos principales que se deben implementar en la aplicación, usaremos el código del [ejemplo Connect de UWP para Office 365 con Microsoft Graph](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample).

## Interfaz de usuario de ejemplo

El ejemplo contiene una interfaz de usuario muy sencilla, que consta de una barra de comandos superior, el **botón Conectar**, el botón **Enviar correo** y un cuadro de texto que se rellena automáticamente con la dirección de correo electrónico del usuario que ha iniciado sesión pero puede modificarse. La barra de comandos también contiene un botón que permite a los desarrolladores encontrar el URI de redireccionamiento de la aplicación.

El botón **Enviar correo** está deshabilitado cuando el usuario no está conectado:

![Pantalla que muestra el botón Conectar habilitado y el botón Enviar correo deshabilitado](images/SignedOut.png)

En la barra de comandos superior, aparece un botón de desconexión cuando el usuario está conectado:

![Pantalla que muestra la dirección de correo electrónico del usuario conectado y el botón Enviar correo habilitado](images/SignedIn.png)

Todas las cadenas de la interfaz de usuario del ejemplo se almacenan en el archivo Resources.resw dentro de la carpeta Activos.

## Registrar la aplicación
 
Windows 10 proporciona a cada aplicación un URI único y asegura que los mensajes enviados a ese URI solo se envían a esa aplicación. Debe crear su aplicación y encontrar este URI generado por el sistema antes de que registre su aplicación. En el ejemplo, encontrará este método en el archivo AuthenticationHelper.cs:

```c#
        public static string GetAppRedirectURI()
        {
            // Windows 10 universal apps require redirect URI in the format below. Add a breakpoint to this line and run the app before you register it, so that
            // you can supply the correct redirect URI value.
            return string.Format("ms-appx-web://microsoft.aad.brokerplugin/{0}", WebAuthenticationBroker.GetCurrentApplicationCallbackUri().Host).ToUpper();
        }
```

Ese método lo desencadena el botón **Copiar URI de redireccionamiento** en el ejemplo, pero también puede seguir el patrón del ejemplo [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM), en el que la cadena está definida en la declaración de clase MainPage y puede capturarla con el depurador de Visual Studio. 

Siga los pasos de [Registrar y configurar la aplicación](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample#register) del archivo Léame de ejemplo para registrar su aplicación después de obtener el valor del URI de redireccionamiento.

Necesitará el valor del parámetro ID de cliente de la página **Configurar** de su aplicación de Azure cuando configure su aplicación para la autenticación.

## Conectarse a Microsoft Graph

El ejemplo usa la API nativa de WebAccountManager de Windows 10 para autenticar a los usuarios. Sigue un patrón similar al que se describe en la publicación del blog [Desarrollar aplicaciones universales de Windows con Azure AD y la API de identidad de Windows 10](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx) y al que se demuestra en el ejemplo [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM).

El archivo App.xaml contiene los pares clave-valor que su aplicación necesitará para autenticar el usuario y autorizar a la aplicación para enviar un correo electrónico:

```xml
    <Application.Resources>
        <!-- Add your client id here. -->
        <x:String x:Key="ida:ClientID"><your client id></x:String>
        <x:String x:Key="ida:AADInstance">https://login.microsoftonline.com/</x:String>
        <!-- Add your developer tenant domain here. -->
        <x:String x:Key="ida:Domain">yourtenant.onmicrosoft.com</x:String>
    </Application.Resources>
```

Agregue el valor del parámetro ID de cliente que obtuvo al registrar su aplicación como valor para la clave **ida:ClientID**. Cambie el valor de la clave **ida:Domain** de forma que coincida con su inquilino de Office 365.

El archivo AuthenticationHelper.cs contiene el código de autenticación completo junto con la lógica adicional que almacena la información del usuario y fuerza la autenticación solo cuando el usuario se ha desconectado de la aplicación.

El método ``GetTokenHelperAsync`` definido en este archivo se ejecuta cuando el usuario se autentica y, posteriormente, cada vez que la aplicación llama a Microsoft Graph. Su primera tarea consiste en encontrar un proveedor de cuenta de Azure AD:

```c#
           aadAccountProvider = await WebAuthenticationCoreManager.FindAccountProviderAsync("https://login.microsoft.com", authority);
```

El valor del parámetro ``authority`` es una cadena concatenada generada a partir de dos valores almacenados en el archivo App.xaml: el valor de la clave **ida:AADInstance** más el valor de la clave **ida:Domain**. Esto crea una autoridad específica de inquilinos. También puede usar la cadena "organizations" si desea que su aplicación se ejecute en cualquier inquilino de Azure AD.

Cuando el usuario se autentica, la aplicación almacena el valor del parámetro ID de usuario en ``ApplicationData.Current.RoamingSettings``. El método ``GetTokenHelperAsync`` comprueba primero si este valor existe y, si es así, intenta autenticar de forma silenciosa:

```c#
            // Check if there's a record of the last account used with the app
            var userID = _settings.Values["userID"];

            if (userID != null)
            {

                WebTokenRequest webTokenRequest = new WebTokenRequest(aadAccountProvider, string.Empty, clientId);
                webTokenRequest.Properties.Add("resource", ResourceUrl);

                // Get an account object for the user
                userAccount = await WebAuthenticationCoreManager.FindAccountAsync(aadAccountProvider, (string)userID);


                // Ensure that the saved account works for getting the token we need
                WebTokenRequestResult webTokenRequestResult = await WebAuthenticationCoreManager.RequestTokenAsync(webTokenRequest, userAccount);
                if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.Success || webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.AccountSwitch)
                {
                    WebTokenResponse webTokenResponse = webTokenRequestResult.ResponseData[0];
                    userAccount = webTokenResponse.WebAccount;
                    token = webTokenResponse.Token;

                }
                else
                {
                    // The saved account could not be used for getting a token
                    // Make sure that the UX is ready for a new sign in
                    SignOut();
                }

            }
```

La aplicación usa el punto de conexión de Microsoft Graph (**https://graph.microsoft.com/**) como valor del recurso. Al crear el elemento ``WebTokenRequest``, usa el valor del parámetro ID de cliente que agregó al archivo App.xaml. Ya que la aplicación conoce el ID de usuario y el usuario no se ha desconectado, la API de WebAccountManager puede encontrar la cuenta del usuario y pasarla a la solicitud de token. El método ``WebAuthenticationCoreManager.RequestTokenAsync`` devuelve un token de acceso con los permisos apropiados asignados a este.

Si la aplicación no encuentra ningún valor para ``userID`` en la configuración de itinerancia, construye un ``WebTokenRequest`` que fuerza al usuario a autenticarse a través de la interfaz de usuario:

```c#
            else
            {
                // There is no recorded user. Start a sign in flow without imposing a specific account.

                WebTokenRequest webTokenRequest = new WebTokenRequest(aadAccountProvider, string.Empty, clientId, WebTokenRequestPromptType.ForceAuthentication);
                webTokenRequest.Properties.Add("resource", ResourceUrl);

                WebTokenRequestResult webTokenRequestResult = await WebAuthenticationCoreManager.RequestTokenAsync(webTokenRequest);
                if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.Success)
                {
                    WebTokenResponse webTokenResponse = webTokenRequestResult.ResponseData[0];
                    userAccount = webTokenResponse.WebAccount;
                    token = webTokenResponse.Token;

                }
            }
```

Si alguno de los intentos de recuperar un token se realiza correctamente, el método ``GetTokenHelperAsync`` finaliza almacenando la información de usuario importante en la configuración de itinerancia y, a continuación, devuelve el valor del token. De lo contrario, se asegura de que la configuración de itinerancia es nula y devuelve un valor nulo.

```c#
            // We succeeded in getting a valid user.
            if (userAccount != null)
            {
                // save user ID in local storage
                _settings.Values["userID"] = userAccount.Id;
                _settings.Values["userEmail"] = userAccount.UserName;
                _settings.Values["userName"] = userAccount.Properties["DisplayName"];

                return token;
            }

            // We didn't succeed in getting a valid user. Clear the app settings so that another user can sign in.
            else
            {
                
                SignOut();
                return null;
            }
```

## Enviar un correo electrónico con Microsoft Graph

El archivo MailHelper.cs contiene el código que crea y envía correos electrónicos. Consta de un único método (``ComposeAndSendMailAsync``) que crea y envía una solicitud POST al punto de conexión **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail**. 

El método ``ComposeAndSendMailAsync`` adquiere tres valores de cadena (``subject``, ``bodyContent`` y ``recipients``) que le pasa el archivo MainPage.xaml.cs. Las cadenas ``subject`` y ``bodyContent`` se almacenan, junto con todas las demás cadenas de la interfaz de usuario, en el archivo Resources.resw. La cadena ``recipients`` proviene del cuadro de dirección en la interfaz de la aplicación. 

Ya que el usuario puede pasar potencialmente más de una dirección, la primera tarea es dividir la cadena ``recipients`` en un conjunto de objetos EmailAddress que pueden pasarse en el cuerpo POST de la solicitud:

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }
```

La segunda tarea es construir un objeto de mensaje JSON válido y enviarlo al punto de conexión **me/microsoft.graph.SendMail** a través de una solicitud HTTP POST. Ya que la cadena ``bodyContent`` es un documento HTML, la solicitud establece el valor del parámetro **ContentType** en HTML. Tenga en cuenta también la llamada a ``AuthenticationHelper.GetTokenHelperAsync`` para garantizar que tenemos un token de acceso reciente para pasar en la solicitud.

```c#
                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenHelperAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }
```

Una vez que haya realizado una solicitud REST correcta, ya habrá llevado a cabo los tres pasos requeridos para interactuar con Microsoft Graph: registro de la aplicación, autenticación del usuario y realizar una solicitud REST.


<!--## Additional resources

* [Develop Windows Universal Apps with Azure AD and the Windows 10 Identity API](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx)
* [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM)
* [Office Dev Center](http://dev.office.com)-->

