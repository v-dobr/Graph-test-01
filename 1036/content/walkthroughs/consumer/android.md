# Prise en main de Microsoft Graph dans une application Android

Cet article décrit les tâches requises pour obtenir un jeton d’accès du point de terminaison d’authentification v2.0 et appeler Microsoft Graph. Il vous guidera dans la construction de l’[exemple de connexion pour Android](https://github.com/microsoftgraph/android-java-connect-sample) et décrit les principaux concepts à mettre en œuvre pour utiliser Microsoft Graph dans votre application pour Android. Cet article explique également comment accéder à Microsoft Graph à l’aide du [kit de développement logiciel (SDK) Microsoft Graph pour Android](https://github.com/microsoftgraph/msgraph-sdk-android) ou des appels REST bruts.

Pour utiliser Microsoft Graph dans votre application pour Android, vous devez afficher la page de connexion Microsoft pour vos utilisateurs, comme présenté dans l’écran suivant.

![Page de connexion pour les comptes Microsoft sur Android](images/AndroidConnect.png)

**Pas envie de créer une application ?** Soyez rapidement opérationnel en téléchargeant l’[exemple de connexion pour Android](https://github.com/microsoftgraph/android-java-connect-sample) sur lequel repose cet article.


## Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

- Un [compte Microsoft](https://www.outlook.com/) ou un [compte Office 365 pour les entreprises](http://dev.office.com/devprogram)
- Android Studio 2.0 ou version ultérieure


## Inscription de l’application
Inscrivez une application sur le portail d’inscription des applications Microsoft. Cette action génère l’ID d’application et le mot de passe que vous utiliserez pour configurer l’application.

1. Connectez-vous au [portail d’inscription des applications Microsoft](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.

2. Choisissez **Ajouter une application**.

3. Entrez un nom pour l’application, puis choisissez **Créer une application**. 
    
   La page d’inscription s’affiche, répertoriant les propriétés de votre application.

4. Copiez l’ID de l’application. Il s’agit de l’identificateur unique de votre application. 

5. Choisissez **Ajouter une plateforme** et **Application mobile**.

    > **Remarque :** Le portail d’inscription des applications fournit un URI de redirection avec la valeur *urn:ietf:wg:oauth:2.0:oob*. Toutefois, vous devez utiliser la valeur d’URI de redirection par défaut *https://login.microsoftonline.com/common/oauth2/nativeclient*.

6. Cliquez sur **Enregistrer**.


## Configurer le projet

Démarrez un nouveau projet dans Android Studio. Vous pouvez laisser les valeurs par défaut le plus souvent dans l’Assistant, veillez simplement à choisir les options suivantes :

* Appareils Android cibles - **Téléphone et tablette**
    * Kit de développement logiciel (SDK) minimal - **API 16 : Android 4.1 (Jelly Bean)**
* Ajouter une activité à Mobile - **Activité de base**
 
Cela génère un projet Android avec une activité et un bouton que vous pouvez utiliser pour authentifier l’utilisateur.

## Authentifier l’utilisateur et obtenir un jeton d’accès
Vous allez utiliser une bibliothèque OAuth pour simplifier le processus d’authentification. [OpenID](http://openid.net) fournit [AppAuth pour Android](https://github.com/openid/AppAuth-Android), une bibliothèque que vous pouvez utiliser dans ce projet.

### Ajouter la dépendance à app/build.gradle

Ouvrez le fichier `build.gradle` dans le module d’application et incluez la dépendance suivante :

```gradle
compile 'net.openid:appauth:0.3.0'
```

### Démarrer le flux d’authentification

1. Ouvrez le fichier **MainActivity** et déclarez un objet **AuthorizationService** dans la méthode **onCreate**.
    ```java
    final AuthorizationService authorizationService =
        new AuthorizationService(this);
    ```
    
2. Recherchez le gestionnaire d’événements pour l’événement de clic de *FloatingActionButton*. Remplacez la méthode **onClick** par le code suivant. Insérez l’**ID d’application** de votre application dans l’espace réservé marqué avec **\<YOUR_APPLICATION_ID\>**.
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
    
À ce stade, vous devez disposer d’une application Android avec un bouton. Si vous appuyez sur le bouton, l’application présente une page d’authentification sur le navigateur de l’appareil. L’étape suivante consiste à gérer le code que le serveur d’autorisation envoie à l’URI de redirection et à l’échanger contre un jeton d’accès.

### Échanger le code d’autorisation contre un jeton d’accès

Votre application doit être prête à traiter la réponse du serveur d’autorisation, qui contient un code que vous pouvez échanger contre un jeton d’accès.

1. Vous devez indiquer au système Android que l’application peut gérer les demandes envoyées à *https://login.microsoftonline.com/common/oauth2/nativeclient*. Pour ce faire, ouvrez le fichier **AndroidManifest** et ajoutez l’élément d’activité suivant.
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

2. L’activité est appelée lorsque le serveur d’autorisation envoie une réponse. Vous pouvez demander un jeton d’accès avec la réponse du serveur d’autorisation. Revenez à **MainActivity** et ajoutez le code suivant à la méthode **onCreate**.
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

Notez que vous avez un jeton d’accès sur cette ligne `String accessToken = tokenResponse.accessToken;`. Vous êtes maintenant prêt à ajouter un code pour appeler Microsoft Graph. 

## Appeler Microsoft Graph
Vous pouvez [utiliser le kit de développement logiciel (SDK) Microsoft Graph](#call-microsoft-graph-using-the-microsoft-graph-sdk) ou l’[API REST Microsoft Graph](#call-microsoft-graph-using-the-microsoft-graph-rest-api) pour appeler Microsoft Graph.

### Appeler Microsoft Graph à l’aide du kit de développement logiciel (SDK) Microsoft Graph
Le [kit de développement logiciel (SDK) Microsoft Graph pour Android](https://github.com/microsoftgraph/msgraph-sdk-android) fournit des classes qui génèrent des demandes et traitent les résultats à partir de Microsoft Graph. Procédez comme suit pour utiliser le kit de développement logiciel (SDK) Microsoft Graph.

1. Ajoutez des autorisations Internet à votre application. Ouvrez le fichier **AndroidManifest** et ajoutez l’enfant suivant à l’élément de manifeste.
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

2. Ajoutez des dépendances au kit de développement logiciel (SDK) Microsoft Graph et à GSON.
   ```gradle
    compile 'com.microsoft.graph:msgraph-sdk-android:1.0.0'
    compile 'com.google.code.gson:gson:2.4'
   ```
   
3. Remplacez la ligne `String accessToken = tokenResponse.accessToken;` par le code suivant. Insérez votre adresse électronique dans l’espace réservé marqué avec **\<YOUR_EMAIL_ADDRESS\>**.
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

### Appeler Microsoft Graph à l’aide de l’API REST Microsoft Graph
L’[API REST Microsoft Graph](http://graph.microsoft.io/docs) expose plusieurs API des services de cloud computing Microsoft via un seul point de terminaison d’API REST. Suivez ces étapes pour utiliser l’API REST.

1. Ajoutez des autorisations Internet à votre application. Ouvrez le fichier **AndroidManifest** et ajoutez l’enfant suivant à l’élément de manifeste.
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

2. Ajoutez une dépendance à la bibliothèque HTTP Volley.
   ```gradle
    compile 'com.android.volley:volley:1.0.0'
   ```
   
3. Remplacez la ligne `String accessToken = tokenResponse.accessToken;` par le code suivant. Insérez votre adresse électronique dans l’espace réservé marqué avec **\<YOUR_EMAIL_ADDRESS\>**.
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

## Exécuter l’application
Vous êtes prêt à essayer votre application Android.

1. Démarrez votre émulateur Android ou connectez votre appareil physique à votre ordinateur.
2. Dans Android Studio, appuyez sur Maj + F10 pour exécuter votre application.
3. Choisissez votre émulateur Android ou votre appareil dans la boîte de dialogue de déploiement.
4. Appuyez sur le bouton d’action flottant sur l’activité principale.
5. Connectez-vous à votre compte personnel, professionnel ou scolaire et accordez les autorisations demandées.
6. Dans la boîte de dialogue de sélection de l’application, appuyez sur votre application pour continuer.

Vérifiez la boîte de réception de l’adresse électronique que vous avez configurée dans la section [Appeler Microsoft Graph](#call-the-microsoft-graph). Vous devriez avoir reçu un courrier électronique du compte utilisé pour vous connecter à l’application.

## Étapes suivantes
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).
- Recherchez des exemples d’opérations courantes dans l’[exemple d’extraits de code pour Android](https://github.com/microsoftgraph/android-java-snippets-sample) ou explorez nos autres [exemples Android](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=android) sur GitHub.


## Voir aussi
* [Présentation de Microsoft Graph](http://graph.microsoft.io/docs)
* [Kit de développement logiciel (SDK) Microsoft Graph pour Android](https://github.com/microsoftgraph/msgraph-sdk-android) 
* [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
