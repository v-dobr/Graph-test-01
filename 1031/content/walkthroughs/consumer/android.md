# Erste Schritte mit Microsoft Graph in einer Android-App

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens vom v2.0-Authentifizierungsendpunkt und zum Aufrufen von Microsoft Graph. Sie werden durch die Erstellung der [Connect-Beispiels für Android](https://github.com/microsoftgraph/android-java-connect-sample) geführt und erhalten Informationen zu den Hauptkonzepten, die Sie zur Verwendung von Microsoft Graph in Ihrer App für Android implementieren. In diesem Artikel wird auch beschrieben, wie Sie mithilfe des [Microsoft Graph-SDKs für Android](https://github.com/microsoftgraph/msgraph-sdk-android) oder reinen REST-Aufrufen auf Microsoft Graph zugreifen.

Um Microsoft Graph in Ihrer App für Android zu verwenden, müssen Sie für Benutzer die Microsoft-Anmeldeseite anzeigen, wie im folgenden Screenshot dargestellt.

![Anmeldeseite für Microsoft-Konten auf Android](images/AndroidConnect.png)

**Sie möchten keine App erstellen?** Laden Sie sich für einen Schnelleinstieg das [Connect-Beispiel für Android](https://github.com/microsoftgraph/android-java-connect-sample) herunter, auf dem dieser Artikel basiert.


## Voraussetzungen

Für die ersten Schritte benötigen Sie: 

- Ein [Microsoft-Konto](https://www.outlook.com/) oder ein [Office 365 for Business-Konto](http://dev.office.com/devprogram).
- Android Studio 2.0 oder eine höhere Version


## Registrieren der App
Registrieren Sie eine App im Microsoft App-Registrierungsportal. Dadurch wird die APP-ID und das Kennwort generiert, mit der bzw. dem Sie die App konfigurieren.

1. Melden Sie sich beim [Microsoft-App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.

2. Klicken Sie auf **App hinzufügen**.

3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus. 
    
   Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.

4. Kopieren Sie die Anwendungs-ID: Dies ist der eindeutige Bezeichner für Ihre App. 

5. Wählen Sie **Platform hinzufügen** und **Mobile Anwendung** aus.

    > **Hinweis:** Das Anwendungsregistrierungsportal stellt einen Umleitungs-URI mit dem Wert *urn:ietf:wg:oauth:2.0:oob* bereit. Sie werden aber den Umleitungs-URI mit dem Standardwert *https://login.microsoftonline.com/common/oauth2/nativeclient* verwenden.

6. Wählen Sie **Speichern** aus.


## Konfigurieren des Projekts

Starten Sie ein neues Projekt in Android Studio. Für den Großteil des Assistenten können Sie die Standardwerte belassen, stellen Sie aber sicher, dass Sie die folgenden Optionen auswählen:

* Ziel-Android-Geräte **Telefon und Tablet**
    * Minimales SDK - **API 16: Android 4.1 (Jelly Bean)**
* Hinzufügen einer Aktivität zu Mobile - **Grundlegende Aktivität**
 
Auf diese Weise erhalten Sie ein Android-Projekt mit einer Aktivität und einer Schaltfläche, die Sie zum Authentifizieren des Benutzers verwenden können.

## Authentifizierung des Benutzers und Abrufen eines Zugriffstokens
Sie verwenden eine OAuth-Bibliothek, um den Authentifizierungsprozess zu vereinfachen. [OpenID](http://openid.net) enthält [AppAuth für Android](https://github.com/openid/AppAuth-Android), eine Bibliothek, die Sie in diesem Projekt verwenden können.

### Hinzufügen der Abhängigkeit zu „app/build.gradle“

Öffnen Sie die `build.gradle`-Datei im App-Modul, und fügen Sie die folgende Abhängigkeit ein:

```gradle
compile 'net.openid:appauth:0.3.0'
```

### Starten des Authentifizierungsflusses

1. Öffnen Sie die Datei **MainActivity**, und deklarieren Sie ein **AuthorizationService**-Objekt in der **OnCreate**-Methode.
    ```java
    final AuthorizationService authorizationService =
        new AuthorizationService(this);
    ```
    
2. Suchen Sie den Ereignishandler für das Click-Ereignis der *FloatingActionButton*. Ersetzen Sie die **onClick**-Methode durch den folgenden Code. Fügen Sie die **Anwendungs-ID** Ihrer App und den Platzhalter ein, der mit **\<YOUR_APPLICATION_ID\>** markiert ist.
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
    
Nun sollten Sie über eine Android-App mit einer Schaltfläche verfügen. Wenn Sie auf die Schaltfläche klicken, wird in der App eine Authentifizierungsseite über den Browser des Geräts angezeigt. Der nächste Schritt besteht darin, den Code zu behandeln, den der Autorisierungsserver an den Umleitung-URI sendet, und diesen durch ein Zugriffstoken zu ersetzen.

### Ersetzen des Autorisierungscodes durch ein Zugriffstoken

Sie müssen Ihre App so vorbereiten, dass die Antwort des Autorisierungsservers verarbeitet wird, die einen Code enthält, der durch ein Zugriffstoken ersetzt werden kann.

1. Sie müssen dem Android-System mitteilen, dass die App Anforderungen an *https://login.microsoftonline.com/common/oauth2/nativeclient* verarbeiten kann. Öffnen Sie hierzu die Datei **AndroidManifest**, und fügen Sie das folgende Aktivitätselement hinzu.
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

2. Die Aktivität wird aufgerufen, wenn der Autorisierungsserver eine Antwort sendet. Sie können Zugriffstoken mit der Antwort vom Autorisierungsserver anfordern. Gehen Sie wieder zurück zu Ihrer **MainActivity**, und fügen Sie den folgenden Code an die **OnCreate**-Methode an.
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

Beachten Sie, dass in dieser Zeile `String accessToken = tokenResponse.accessToken;` ein Zugriffstoken vorhanden ist. Sie können nun Code hinzufügen, um Microsoft Graph aufzurufen. 

## Aufrufen von Microsoft Graph
Zum Aufrufen von Microsoft Graph können Sie [das Microsoft Graph-SDK](#call-microsoft-graph-using-the-microsoft-graph-sdk) oder die [Microsoft Graph-REST-API](#call-microsoft-graph-using-the-microsoft-graph-rest-api) verwenden.

### Aufrufen von Microsoft Graph mit dem Microsoft Graph-SDK
Das [Microsoft Graph-SDK für Android](https://github.com/microsoftgraph/msgraph-sdk-android) stellt Klassen bereit, aus denen Anforderungen und Prozessergebnisse aus Microsoft Graph erstellt werden. Führen Sie die folgenden Schritte aus, um das Microsoft Graph-SDK zu verwenden.

1. Gewähren Sie Ihrer Anwendung entsprechende Internetberechtigungen. Öffnen Sie die Datei **AndroidManifest**, und fügen Sie das folgende untergeordnete Element zum Manifest hinzu.
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

2. Fügen Sie Abhängigkeiten zum Microsoft Graph-SDK und zu GSON hinzu.
   ```gradle
    compile 'com.microsoft.graph:msgraph-sdk-android:1.0.0'
    compile 'com.google.code.gson:gson:2.4'
   ```
   
3. Ersetzen Sie die Zeile `String accessToken = tokenResponse.accessToken;` durch den folgenden Code. Fügen Sie Ihre E-Mail-Adresse in den Platzhalter ein, der mit **\<YOUR_EMAIL_ADDRESS\>** markiert ist.
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

### Aufrufen von Microsoft Graph mit der Microsoft Graph-REST-API
Die [Microsoft Graph-REST-API](http://graph.microsoft.io/docs) macht mehrere APIs aus Microsoft-Clouddiensten über einen einzelnen REST-API-Endpunkt verfügbar. Gehen Sie folgendermaßen vor, um die REST-API zu verwenden.

1. Gewähren Sie Ihrer Anwendung entsprechende Internetberechtigungen. Öffnen Sie die Datei **AndroidManifest**, und fügen Sie das folgende untergeordnete Element zum Manifest hinzu.
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

2. Fügen Sie der Volley HTTP-Bibliothek eine Abhängigkeit hinzu.
   ```gradle
    compile 'com.android.volley:volley:1.0.0'
   ```
   
3. Ersetzen Sie die Zeile `String accessToken = tokenResponse.accessToken;` durch den folgenden Code. Fügen Sie Ihre E-Mail-Adresse in den Platzhalter ein, der mit **\<YOUR_EMAIL_ADDRESS\>** markiert ist.
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

## Ausführen der App
Sie können Ihre Android-App nun testen.

1. Starten Sie Ihren Android-Emulator, oder schließen Sie das Gerät an Ihren Computer an.
2. Drücken Sie in Android Studio UMSCHALT+F10, um die App auszuführen.
3. Wählen Sie im Dialogfeld für die Bereitstellung Ihren Android-Emulator oder das Android-Gerät aus.
4. Tippen Sie in der Hauptaktivität auf die Floating-Action-Button.
5. Melden Sie sich mit Ihrem persönlichen Konto oder mit Ihrem Geschäfts- oder Schulkonto an, und gewähren Sie die erforderlichen Berechtigungen.
6. Tippen Sie im App-Auswahldialogfeld auf die App, um fortzufahren.

Überprüfen Sie den Posteingang der E-Mail-Adresse, die Sie im Abschnitt [Aufrufen von Microsoft Graph](#call-the-microsoft-graph) konfiguriert haben. Dort sollten Sie eine E-Mail von dem Konto vorfinden, das Sie zum Anmelden bei der App verwendet haben.

## Nächste Schritte
- Testen Sie die REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).
- Beispiele für allgemeine Vorgänge finden Sie im [Codeausschnittbeispiel für Android](https://github.com/microsoftgraph/android-java-snippets-sample). Sie können auch unsere anderen [Android-Beispiele](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=android) auf GitHub erkunden.


## Siehe auch
* [Übersicht über Microsoft Graph](http://graph.microsoft.io/docs)
* [Microsoft Graph-SDK für Android](https://github.com/microsoftgraph/msgraph-sdk-android) 
* [Protokolle für Azure AD, Version 2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Token für Azure AD, Version 2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
