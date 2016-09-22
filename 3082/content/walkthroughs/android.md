# Llamar a Microsoft Graph desde una aplicación de Android

En este artículo veremos las tareas mínimas necesarias para obtener un token de acceso de Azure Active Directory (AD) y llamar a Microsoft Graph. Para explicar los conceptos principales que se deben implementar en la aplicación, usaremos el código del [ejemplo Office 365 Android Connect con Microsoft Graph](https://github.com/microsoftgraph/android-java-connect-rest-sample).

La pantalla siguiente aparece después de que un usuario se conecta a Office 365 y muestra la actividad de envío de correo de la aplicación de ejemplo.

![Captura de pantalla del ejemplo Connect de la API unificada para Android de Office 365](./images/AndroidConnect.png)

## Información general

Para llamar a la API de Microsoft Graph, en el [ejemplo Office 365 Android Connect](https://github.com/microsoftgraph/android-java-connect-rest-sample), se realizan las siguientes tareas.

1. Llamar a métodos en la biblioteca de Azure Active Directory para autenticar a un usuario y obtener un token de acceso.
2. Crear una solicitud de mensaje de correo como una operación REST en el punto de conexión de la API de Microsoft Graph.

<!--<a name="register"/>-->
## Registrar la aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, registre la aplicación y establezca los permisos para usar los servicios de Microsoft Graph. Con la [herramienta de registro de aplicaciones](https://dev.office.com/app-registration) puede registrar en pocos pasos su aplicación para que tenga acceso a la cuenta profesional o educativa de un usuario. Para administrarla, vaya al [Portal de administración de Microsoft Azure](https://manage.windowsazure.com).

Si prefiere registrar manualmente la aplicación, vea la sección **Register your native app with the Azure Management Portal** (Registrar una aplicación nativa con el Portal de administración de Azure) del artículo [Manually register your app with Azure AD so it can access Office 365 APIs](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually) (Registrar manualmente la aplicación con Azure AD para que tenga acceso a las API de Office 365) para obtener instrucciones sobre cómo hacerlo. Tenga en cuenta lo siguiente:

* Configure los **permisos delegados** que requiere su aplicación. El ejemplo Connect requiere el permiso **Enviar correo como usuario con sesión iniciada**.

Anote los siguientes valores de la página **Configurar** de la aplicación de Azure.

* Identificador de cliente
* URL de redireccionamiento

Estos valores son necesarios para configurar el código de autenticación en la aplicación.

## Dependencias de Gradle en el ejemplo Connect
El ejemplo toma las dependencias de las bibliotecas que se muestran en el siguiente fragmento de código build.gradle.

```gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.1.1'

    // Azure Active Directory Library
    compile 'com.microsoft.aad:adal:1.1.7'

    // Retrofit + custom HTTP
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.0.0'
    compile 'com.squareup.okhttp:okhttp:2.0.0'
    compile 'com.squareup.retrofit:retrofit:1.9.0'
}

```
<!--<a name="authenticate"/>-->
## Autenticación en el ejemplo Connect
Para la autenticación, el ejemplo Connect usa los valores del registro de aplicación de Azure y un identificador de usuario. El ejemplo Connect admite dos comportamientos de autenticación.

* Autenticación solicitada. Se usa cuando no hay ningún identificador de usuario en caché en las preferencias del dispositivo Android.
* Autenticación silenciosa. Se usa cuando hay un identificador de usuario en caché y no es necesario pedirlo.

La clase [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) proporciona un método auxiliar de `isConnected()` para buscar cualquier id. de usuario en caché y determinar el comportamiento de autenticación que se debe usar.


```java
    private boolean isConnected(){
        SharedPreferences settings = this
                .mContextActivity
                .getSharedPreferences(PREFERENCES_FILENAME, Context.MODE_PRIVATE);

        return settings.contains(USER_ID_VAR_NAME);
    }

```

Independientemente del comportamiento, el flujo de autenticación de ADAL necesita el identificador de cliente y una URL de redireccionamiento que se obtienen en el proceso de registro de Azure. En el ejemplo, estas cadenas se mantienen en el código fuente y se recuperan antes de que el objeto administrador de autenticación autentique al usuario.

La interfaz [Constants.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/Constants.java) expone dos cadenas estáticas para el id. de cliente y la URL de redireccionamiento.

```java
interface Constants {
    String AUTHORITY_URL = "https://login.microsoftonline.com/common";
    // Update these two constants with the values for your application:
    String CLIENT_ID = "<Your client id here>";
    String REDIRECT_URI = "<Your redirect uri here>";
    String UNIFIED_API_ENDPOINT = "https://graph.microsoft.com/v1.0/";
    String UNIFIED_ENDPOINT_RESOURCE_ID = "https://graph.microsoft.com/";
}
```
### Construir la clase AuthenticationManager
El constructor [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) no toma ningún argumento, pero establece un campo de cadena de clase del archivo Constants.java con la dirección URL del punto de conexión de Graph. Esta cadena de recursos se usa en ambos comportamientos de autenticación.

```java
    private AuthenticationManager() {
        mResourceId = Constants.UNIFIED_ENDPOINT_RESOURCE_ID;
    }
```

### Autenticación solicitada

La clase [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) proporciona un método de `authenticatePrompt()` para adquirir el token de acceso usado en las llamadas REST del punto de conexión unificado.

El método `acquireToken()` de la biblioteca ADAL es asincrónico. Los argumentos del método son el recurso, el identificador de cliente y la URL, así como una referencia al contexto de la actividad actual. Esta referencia permite a la biblioteca de ADAL mostrar una página de desafío de credencial en la actividad. Si la autenticación se realiza correctamente, la biblioteca ADAL invoca la devolución de llamada `onSuccess()`. Esta devolución de llamada hace dos cosas:

* Almacena el token de acceso en `mAccessToken`. Al realizar una llamada REST para enviar un mensaje de correo, el ejemplo coloca este token de acceso en un encabezado de autorización.
* Almacenar el identificador del usuario en las preferencias.


```java
    /**
     * Calls acquireToken to prompt the user for credentials.
     *
     * @param authenticationCallback The callback to notify when the processing is finished.
     */
    private void authenticatePrompt(final AuthenticationCallback<AuthenticationResult> authenticationCallback) {
        getAuthenticationContext().acquireToken(
                this.mContextActivity,
                this.mResourceId,
                Constants.CLIENT_ID,
                Constants.REDIRECT_URI,
                PromptBehavior.Always,
                new AuthenticationCallback<AuthenticationResult>() {
                    @Override
                    public void onSuccess(final AuthenticationResult authenticationResult) {
                        if (authenticationResult != null) {
                            if (authenticationResult.getStatus() == AuthenticationStatus.Succeeded) {
                                setUserId(authenticationResult.getUserInfo().getUserId());
                                mAccessToken = authenticationResult.getAccessToken();
                                authenticationCallback.onSuccess(authenticationResult);
                            } else {
                                // We need to make sure that there is no data stored with the failed auth
                                AuthenticationManager.getInstance().disconnect();
                                // This condition can happen if user signs in with an MSA account
                                // instead of an Office 365 account
                                authenticationCallback.onError(
                                        new AuthenticationException(
                                                ADALError.AUTH_FAILED,
                                                authenticationResult.getErrorDescription()));
                            }
                        } else {
                            // I could not authenticate the user silently,
                            // falling back to prompt the user for credentials.
                            authenticatePrompt(authenticationCallback);
                        }
                    }

                    @Override
                    public void onError(Exception e) {
                        // We need to make sure that there is no data stored with the failed auth
                        AuthenticationManager.getInstance().disconnect();
                        authenticationCallback.onError(e);
                    }
                }
        );
    }

```

###Autenticación silenciosa
La clase [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) proporciona un método `authenticateSilent()` para adquirir el token de acceso usado en las llamadas REST del punto de conexión unificado.

El método `acquireTokenSilent()` de la biblioteca ADAL es asincrónico. Además del identificador de cliente del registro de Azure y del identificador de recurso, también toma el identificador de usuario que está almacenado en las preferencias compartidas. El método auxiliar `getUserId()` obtiene el id. de usuario en el almacenamiento.

Si la autenticación se realiza correctamente, se invoca el método `onSuccess()`. `onSuccess` almacena el token de acceso en `mAccessToken`. Al realizar una llamada REST para enviar un mensaje de correo, el ejemplo coloca este token de acceso en un encabezado de autorización.
```java
    /**
     * Calls acquireTokenSilent with the user id stored in shared preferences.
     * In case of an error, it falls back to {@link AuthenticationManager#authenticatePrompt(AuthenticationCallback)}.
     *
     * @param authenticationCallback The callback to notify when the processing is finished.
     */
    private void authenticateSilent(final AuthenticationCallback<AuthenticationResult> authenticationCallback) {
        getAuthenticationContext().acquireTokenSilent(
                this.mResourceId,
                Constants.CLIENT_ID,
                getUserId(),
                new AuthenticationCallback<AuthenticationResult>() {
                    @Override
                    public void onSuccess(final AuthenticationResult authenticationResult) {
                        if (authenticationResult != null) {
                            if (authenticationResult.getStatus() == AuthenticationStatus.Succeeded) {
                                mAccessToken = authenticationResult.getAccessToken();
                                authenticationCallback.onSuccess(authenticationResult);
                            } else {
                                authenticationCallback.onError(
                                        new Exception(authenticationResult.getErrorDescription()));

                            }
                        } else {
                            // I could not authenticate the user silently,
                            // falling back to prompt the user for credentials.
                            authenticatePrompt(authenticationCallback);
                        }
                    }

                    @Override
                    public void onError(Exception e) {
                        // I could not authenticate the user silently,
                        // falling back to prompt the user for credentials.
                        authenticatePrompt(authenticationCallback);
                    }
                }
        );
    }

```
<!--<a name="sendmail"/>-->
## Enviar un mensaje de correo electrónico con Office 365

Después de que el usuario inicie sesión en Azure, en el ejemplo Connect, se muestra al usuario una actividad para el envío de un mensaje de correo. En el ejemplo Connect, se usa la clase [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) para enviar el mensaje cuando los usuarios hacen clic en el botón Enviar correo.

### Clase de aplicación auxiliar de adaptador de REST
La clase [RESTHelper.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/RESTHelper.java) proporciona un método para insertar un encabezado de autorización en cada llamada REST que realice el ejemplo. Usa el token de acceso proporcionado por el administrador de autenticación.

```java
       //This method catches outgoing REST calls and injects the Authorization and host headers before
        //sending to REST endpoint
        RequestInterceptor requestInterceptor = new RequestInterceptor() {
            @Override
            public void intercept(RequestFacade request) {
                final String token = mAccessToken;
                if (null != token) {
                    request.addHeader("Authorization", "Bearer " + token);
                }
            }
        };
```
### Clase UnifiedAPIController
La clase [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) genera la solicitud REST en el método `sendMail()`.


```java
    /**
     * Sends an email message using the Unified API on Office 365. The mail is sent
     * from the address of the signed in user.
     *
     * @param emailAddress The recipient email address.
     * @param subject      The subject to use in the mail message.
     * @param body         The body of the message.
     * @param callback     UI callback to be invoked by Retrofit call when
     *                     operation completed
     */
    public void sendMail(
            final String emailAddress,
            final String subject,
            final String body,
            Callback<Void> callback) {
        ensureService();
        // Use the Unified API service on Office 365 to create the message.
        mUnifiedAPIService.sendMail(
                "application/json",
                createMailPayload(
                        subject,
                        body,
                        emailAddress),
                callback);
    }

```
### Interfaz de UnifiedAPIService
La interfaz de [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) proporciona firmas de método para las llamadas REST que realiza el ejemplo mediante anotaciones Retrofit.

```java
    @POST("/me/sendMail")
    void sendMail(
            @Header("Content-type") String contentTypeHeader,
            @Body TypedString mail,
            Callback<Void> callback);


```

## Pasos siguientes
La API de Microsoft Graph es una interfaz unificadora muy potente que permite trabajar con todos los tipos de datos de Microsoft. Vea la [documentación de Microsoft Graph](http://graph.microsoft.io/docs) para ver todas las posibilidades que ofrece la API de Microsoft Graph.

Hemos publicado muchos ejemplos Android para Office 365. Cada uno de estos ejemplos se basa en los conceptos que presentamos en el ejemplo Connect. Si desea realizar más acciones con sus aplicaciones para Android, consulte [más ejemplos Android para Office 365](http://aka.ms/androidgraphsamples) en la organización de GitHub de Office.
 
