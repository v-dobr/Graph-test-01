# Introducción a Microsoft Graph en una aplicación de Android

Este artículo describe las tareas necesarias para obtener un token de acceso desde el punto de conexión de autenticación v2.0 y llamar a Microsoft Graph. Le muestra los pasos para la creación del [Ejemplo de conexión para Android](https://github.com/microsoftgraph/android-java-connect-sample) y explica los conceptos principales que implementará para usar Microsoft Graph en la aplicación de Android. El artículo también describe cómo tener acceso a Microsoft Graph usando el [SDK de Microsoft Graph para Android](https://github.com/microsoftgraph/msgraph-sdk-android) o llamadas de REST sin formato.

Para utilizar Microsoft Graph en su aplicación para Android, debe mostrar la página de inicio de sesión de Microsoft a los usuarios, como se ve en la siguiente captura de pantalla.

![Página de inicio de sesión para cuentas de Microsoft en Android](images/AndroidConnect.png)

**¿No quiere crear una aplicación?** Puede ponerse a trabajar rápidamente descargando el [Ejemplo de conexión para Android](https://github.com/microsoftgraph/android-java-connect-sample) en el que se basa este artículo.


## Requisitos previos

Para comenzar, necesitará: 

- Una [cuenta de Microsoft](https://www.outlook.com/) o una [cuenta de Office 365 para empresas](http://dev.office.com/devprogram)
- Versión 2.0 de Android Studio o superior


## Registrar la aplicación
Registre una aplicación en el Portal de registro de aplicaciones de Microsoft. Este proceso generará el identificador de la aplicación y la contraseña que utilizará para configurar la aplicación.

1. Inicie sesión en el [Portal de registro de aplicaciones de Microsoft](https://apps.dev.microsoft.com/) mediante su cuenta personal, profesional o educativa.

2. Seleccione **Agregar una aplicación**.

3. Escriba un nombre para la aplicación y seleccione **Crear aplicación**. 
    
   Se muestra la página de registro, indicando las propiedades de la aplicación.

4. Copie el identificador de la aplicación. Se trata del identificador único para su aplicación. 

5. Elija **Agregar plataforma** y **Aplicación móvil**.

    > **Nota:** El Portal de registro de aplicaciones proporciona un URI de redireccionamiento con un valor de *urn: ietf:wg:oauth:2.0:oob*. Sin embargo, utilizará el valor de URI de redireccionamiento predeterminado de *https://login.microsoftonline.com/common/oauth2/nativeclient*.

6. Seleccione **Guardar**.


## Configurar el proyecto

Inicie un nuevo proyecto en Android Studio. Puede dejar los valores predeterminados para la mayoría del proceso del asistente, pero asegúrese de elegir las siguientes opciones:

* Target Android Devices - **Phone and Tablet**Dispositivos Android de destino: teléfonos y tablets)
    * Minimum SDK - **API 16: Android 4.1 (Jelly Bean)** (SDK mínimo: API 16: Android 4.1 (Jelly Bean))
* Add an Activity to Mobile - **Basic Activity** (Agregar una actividad al dispositivo móvil: actividad básica)
 
Esto le proporcionará un proyecto de Android con una actividad y un botón que puede utilizar para autenticar al usuario.

## Autenticar al usuario y obtener un token de acceso
Utilizará una biblioteca OAuth para simplificar el proceso de autenticación. [OpenID](http://openid.net) proporciona [AppAuth para Android](https://github.com/openid/AppAuth-Android), una biblioteca que se puede utilizar en este proyecto.

### Agregar la dependencia a app/build.gradle

Abra el archivo `build.gradle` en el módulo de la aplicación e incluya la siguiente dependencia:

```gradle
compile 'net.openid:appauth:0.3.0'
```

### Inicie el flujo de autenticación

1. Abra el archivo **MainActivity** y declare un objeto **AuthorizationService** en el método **onCreate**.
    ```java
    final AuthorizationService authorizationService =
        new AuthorizationService(this);
    ```
    
2. Busque el controlador de eventos para el evento de clic de *FloatingActionButton*. Reemplace el método **onClick** en el archivo con el código siguiente. Inserte el **id. de la aplicación** en el marcador de posición que se marca con **\<YOUR_APPLICATION_ID\>**.
    ```java
    @Override
    public void onClick(View view) {
        Uri authorizationEndpoint =
            Uri.parse("https://login.microsoftonline.com/common/oauth2/v2.0/authorize");
        Uri tokenEndpoint =
            Uri.parse("https://login.microsoftonline.com/common/oauth2/v2.0/token");
        AuthorizationServiceConfiguration config =
            new AuthorizationServiceConfiguration(
                    authorizationEndpoint,
                    tokenEndpoint,null);

        List<String> scopes = new ArrayList<>(
            Arrays.asList("openid mail.send".split(" ")));

        AuthorizationRequest authorizationRequest = new AuthorizationRequest.Builder(
            config,
            "<YOUR_APPLICATION_ID>",
            ResponseTypeValues.CODE,
            Uri.parse("https://login.microsoftonline.com/common/oauth2/nativeclient"))
            .setScopes(scopes)
            .build();

        Intent intent = new Intent(view.getContext(), MainActivity.class);

        PendingIntent redirectIntent =
            PendingIntent.getActivity(
                    view.getContext(),
                    authorizationRequest.hashCode(),
                    intent, 0);

        authorizationService.performAuthorizationRequest(
            authorizationRequest,
            redirectIntent);
    }
    ```
    
En este punto, debe tener una aplicación Android con un botón. Si presiona el botón, la aplicación presenta una página de autenticación en el explorador del dispositivo. El siguiente paso es controlar el código que envía el servidor de autorización a la URI de redirección y cambiarlo por un token de acceso.

### Cambiar el código de autorización por un token de acceso

Es necesario que prepare la aplicación para controlar la respuesta del servidor de autorización, que contiene un código que se puede intercambiar por un token de acceso.

1. Es necesario indicar al sistema Android que la aplicación puede controlar las solicitudes a *https://login.microsoftonline.com/common/oauth2/nativeclient*. Para ello, abra el archivo **AndroidManifest** y agregue el siguiente elemento de actividad.
    ```xml
    <activity android:name="net.openid.appauth.RedirectUriReceiverActivity">
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:scheme="https"/>
            <data android:host="login.microsoftonline.com"/>
            <data android:path="/common/oauth2/nativeclient"/>
        </intent-filter>
    </activity>
    ```

2. La actividad se invoca cuando el servidor de autorización envía una respuesta. Puede solicitar un token de acceso con la respuesta del servidor de autorización. Regrese a **MainActivity** y anexe el siguiente código al método **onCreate**.
    ```java
    Bundle extras = getIntent().getExtras();
    if (extras != null) {
        AuthorizationResponse authorizationResponse = AuthorizationResponse.fromIntent(getIntent());
        AuthorizationException authorizationException = AuthorizationException.fromIntent(getIntent());
        final AuthState authState = new AuthState(authorizationResponse, authorizationException);

        if (authorizationResponse != null) {
            HashMap<String, String> additionalParams = new HashMap<>();
            TokenRequest tokenRequest = authorizationResponse.createTokenExchangeRequest(additionalParams);

            authorizationService.performTokenRequest(
                tokenRequest,
                new AuthorizationService.TokenResponseCallback() {
                    @Override
                    public void onTokenRequestCompleted(
                            @Nullable TokenResponse tokenResponse,
                            @Nullable AuthorizationException ex) {
                        authState.update(tokenResponse, ex);
                        if (tokenResponse != null) {
                            String accessToken = tokenResponse.accessToken;
                        }
                    }
                });
        } else {
            Log.i("MainActivity", "Authorization failed: " + authorizationException);
        }
    }
    ```

Tenga en cuenta que hay un token de acceso en esta línea `String accessToken = tokenResponse.accessToken;`. Ahora está preparado para agregar código para llamar a Microsoft Graph. 

## Llamar a Microsoft Graph
Puede [utilizar el SDK de Microsoft Graph](#call-microsoft-graph-using-the-microsoft-graph-sdk) o la [API de REST de Microsoft Graph](#call-microsoft-graph-using-the-microsoft-graph-rest-api) para llamar a Microsoft Graph.

### Llame a Microsoft Graph mediante el SDK de Microsoft Graph
El [SDK de Microsoft Graph para Android](https://github.com/microsoftgraph/msgraph-sdk-android) proporciona clases que generan solicitudes y procesan los resultados de Microsoft Graph. Siga estos pasos para utilizar el SDK de Microsoft Graph.

1. Conceda permisos de Internet a la aplicación. Abra el archivo **AndroidManifest** y agregue el siguiente elemento secundario al elemento de manifiesto.
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

2. Agregue dependencias al SDK de Microsoft Graph y GSON.
   ```gradle
    compile 'com.microsoft.graph:msgraph-sdk-android:1.0.0'
    compile 'com.google.code.gson:gson:2.4'
   ```
   
3. Reemplace la línea `String accessToken = tokenResponse.accessToken;` con el siguiente código. Introduzca su dirección de correo electrónico en el marcador de posición que se marca con **\<YOUR_EMAIL_ADDRESS\>**.
    ```java
    final String accessToken = tokenResponse.accessToken;
    final IClientConfig clientConfig = 
            DefaultClientConfig.createWithAuthenticationProvider(new IAuthenticationProvider() {
        @Override
        public void authenticateRequest(IHttpRequest request) {
            request.addHeader("Authorization", "Bearer " + accessToken);
        }
    });

    final IGraphServiceClient graphServiceClient = new GraphServiceClient
        .Builder()
        .fromConfig(clientConfig)
        .buildClient();

    final Message message = new Message();
    EmailAddress emailAddress = new EmailAddress();
    emailAddress.address = "<YOUR_EMAIL_ADDRESS>";
    Recipient recipient = new Recipient();
    recipient.emailAddress = emailAddress;
    message.toRecipients = Collections.singletonList(recipient);
    ItemBody itemBody = new ItemBody();
    itemBody.content = "This is the email body";
    itemBody.contentType = BodyType.text;
    message.body = itemBody;
    message.subject = "Sent using the Microsoft Graph SDK";

    AsyncTask.execute(new Runnable() {
        @Override
        public void run() {
            graphServiceClient
                .getMe()
                .getSendMail(message, false)
                .buildRequest()
                .post();
        }
    });
    ```

### Llamar a Microsoft Graph mediante la API de REST de Microsoft Graph
La [API de REST de Microsoft Graph](http://graph.microsoft.io/docs) expone varias API de servicios de nube de Microsoft a través de un único punto de conexión de la API de REST. Siga estos pasos para utilizar la API de REST.

1. Conceda permisos de Internet a la aplicación. Abra el archivo **AndroidManifest** y agregue el siguiente elemento secundario al elemento de manifiesto.
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

2. Agregue una dependencia a la biblioteca HTTP Volley.
   ```gradle
    compile 'com.android.volley:volley:1.0.0'
   ```
   
3. Reemplace la línea `String accessToken = tokenResponse.accessToken;` con el siguiente código. Introduzca su dirección de correo electrónico en el marcador de posición que se marca con **\<YOUR_EMAIL_ADDRESS\>**.
    ```java
    final String accessToken = tokenResponse.accessToken;

    final RequestQueue queue = Volley.newRequestQueue(getApplicationContext());
    String url ="https://graph.microsoft.com/v1.0/me/sendMail";
    final String body = "{" +
        "  Message: {" +
        "    subject: 'Sent using the Microsoft Graph REST API'," +
        "    body: {" +
        "      contentType: 'text'," +
        "      content: 'This is the email body'" +
        "    }," +
        "    toRecipients: [" +
        "      {" +
        "        emailAddress: {" +
        "          address: '<YOUR_EMAIL_ADDRESS>'" +
        "        }" +
        "      }" +
        "    ]}" +
        "}";

    final StringRequest stringRequest = new StringRequest(Request.Method.POST, url,
        new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                Log.d("Response", response);
            }
        },
        new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.d("ERROR","error => " + error.getMessage());
            }
        }
    ) {
        @Override
        public Map<String, String> getHeaders() throws AuthFailureError {
            Map<String,String> params = new HashMap<>();
            params.put("Authorization", "Bearer " + accessToken);
            params.put("Content-Length", String.valueOf(body.getBytes().length));
            return params;
        }
        @Override
        public String getBodyContentType() {
            return "application/json";
        }
        @Override
        public byte[] getBody() throws AuthFailureError {
            return body.getBytes();
        }
    };

    AsyncTask.execute(new Runnable() {
        @Override
        public void run() {
            queue.add(stringRequest);
        }
    });
    ```

## Ejecutar la aplicación
Está preparado para probar la aplicación de Android.

1. Inicie el emulador de Android o conecte el dispositivo físico a su equipo.
2. En Android Studio, presione Mayús + F10 para ejecutar la aplicación.
3. Elija el emulador de  Android o el dispositivo en el cuadro de diálogo de implementación.
4. Pulse el botón de acción flotante en la actividad principal.
5. Inicie sesión con su cuenta personal, profesional o educativa y conceda los permisos solicitados.
6. En el cuadro de diálogo de selección de la aplicación, pulse en la aplicación para continuar.

Compruebe la Bandeja de entrada de la dirección de correo electrónico que configuró en [Llamar a Microsoft Graph](#call-the-microsoft-graph). Debe tener un correo electrónico de la cuenta que utilizó para iniciar sesión en la aplicación.

## Pasos siguientes
- Pruebe la API de REST mediante el [Probador de Graph](https://graph.microsoft.io/graph-explorer).
- Encuentre ejemplos de operaciones comunes en [Fragmentos de código de ejemplo para Android](https://github.com/microsoftgraph/android-java-snippets-sample), o explore nuestros otros [Ejemplos de Android](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=android) en GitHub.


## Vea también
* [Información general de Microsoft Graph](http://graph.microsoft.io/docs)
* [SDK de Microsoft Graph para Android](https://github.com/microsoftgraph/msgraph-sdk-android) 
* [Protocolos de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Tokens de Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
