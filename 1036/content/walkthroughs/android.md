# Appel de Microsoft Graph dans une application Android

Dans cet article, nous examinons les tâches minimales requises pour obtenir un jeton d’accès d’Azure Active Directory (AD) et appeler Microsoft Graph. Nous utilisons le code de l’[exemple de connexion d’un appareil Android à Office 365 à l’aide de Microsoft Graph](https://github.com/microsoftgraph/android-java-connect-rest-sample) pour expliquer les concepts principaux que vous devez mettre en œuvre dans votre application.

L’image suivante indique l’activité d’envoi de messages de l’exemple d’application qui s’affiche une fois qu’un utilisateur se connecte à Office 365.

![Capture d’écran de l’exemple de connexion d’API unifiée d’un appareil Android à Office 365](./images/AndroidConnect.png)

## Vue d’ensemble

Pour appeler l’API Microsoft Graph, l’[exemple de connexion d’un appareil Android à Office 365 à l’aide de Microsoft Graph](https://github.com/microsoftgraph/android-java-connect-rest-sample) effectue les tâches ci-dessous.

1. Authentification d’un utilisateur et obtention d’un jeton d’accès en appelant des méthodes sur la bibliothèque Azure Active Directory.
2. Création d’une demande de message électronique en tant qu’opération REST sur le point de terminaison de l’API Microsoft Graph.

<!--<a name="register"/>-->
## Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application pour accéder au compte professionnel ou scolaire d’un utilisateur à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Pour la gérer, vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com)

Vous pouvez également vous reporter à la section relative à l’**inscription de votre application native auprès du Portail de gestion Azure** dans l’article concernant l’[inscription manuelle de votre application avec Azure AD afin qu’elle puisse accéder aux API Office 365](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually) pour obtenir des instructions sur la façon d’inscrire manuellement l’application. Gardez à l’esprit les informations suivantes :

* Configurez les **autorisations déléguées** requises par votre application. L’exemple de connexion requiert l’autorisation d’**envoi de messages en tant qu’utilisateur connecté**.

Prenez note des valeurs suivantes dans la page de **configuration** de votre application Azure.

* ID du client
* URL de redirection

Ces valeurs sont nécessaires pour configurer le code d’authentification dans votre application.

## Dépendances Gradle dans l’exemple de connexion
L’exemple prend des dépendances sur les bibliothèques indiquées dans l’extrait de code build.gradle suivant :

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
## Authentification dans l’exemple de connexion
L’exemple de connexion utilise les valeurs d’inscription de l’application Azure et un ID utilisateur pour l’authentification. Il existe deux comportements d’authentification pris en charge par l’exemple de connexion.

* Authentification demandée. Utilisée lorsqu’un ID utilisateur n’est pas mis en cache dans des préférences stockées sur l’appareil Android
* Authentification en mode silencieux. Utilisée lorsqu’un ID utilisateur est mis en cache et que l’invitation n’est pas nécessaire.

La classe [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) fournit une méthode d’assistance `isConnected()` pour trouver un ID utilisateur mis en cache et déterminer le comportement d’authentification à utiliser.


```java
    private boolean isConnected(){
        SharedPreferences settings = this
                .mContextActivity
                .getSharedPreferences(PREFERENCES_FILENAME, Context.MODE_PRIVATE);

        return settings.contains(USER_ID_VAR_NAME);
    }

```

Avec l’un ou l’autre des comportements, le flux d’authentification ADAL requiert l’ID client et l’URL de redirection que vous obtenez dans le processus d’inscription Azure. L’exemple conserve ces chaînes dans le code source et les récupère avant que l’objet du gestionnaire d’authentification authentifie l’utilisateur.

L’interface [Constants.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/Constants.java) expose deux chaînes statiques pour l’ID client et l’URL de redirection.

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
### Construction de la classe AuthenticationManager
Le constructeur [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) ne prend aucun argument mais définit un champ de chaîne de classe à partir du fichier Constants.java avec l’URL du point de terminaison Graph. Cette chaîne de ressource est utilisée pour les deux comportements d’authentification.

```java
    private AuthenticationManager() {
        mResourceId = Constants.UNIFIED_ENDPOINT_RESOURCE_ID;
    }
```

### Authentification demandée

La classe [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) fournit une méthode `authenticatePrompt()` pour acquérir le jeton d’accès utilisé pour les appels REST sur le point de terminaison unifié.

La méthode `acquireToken()` de la bibliothèque ADAL est asynchrone. Les arguments de la méthode incluent une référence au contexte de l’activité actuelle, ainsi que la ressource, l’ID client et l’URL de redirection. 
La référence d’activité actuelle permet à la bibliothèque ADAL d’afficher une page de demande de confirmation des informations d’identification dans l’activité. Si l’authentification réussit, la bibliothèque ADAL invoque le rappel `onSuccess()`. Ce rappel effectue deux opérations

* Enregistrement du jeton d’accès dans `mAccessToken`. Lorsque vous effectuez un appel REST pour envoyer un message électronique, l’exemple place ce jeton d’accès dans un en-tête d’autorisation.
* Enregistrement de l’ID utilisateur dans les préférences enregistrées.


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

###Authentification en mode silencieux
La classe [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) fournit une méthode `authenticateSilent()` pour acquérir le jeton d’accès utilisé pour les appels REST sur le point de terminaison unifié.

La méthode `acquireTokenSilent()` de la bibliothèque ADAL est asynchrone. Outre l’ID client d’inscription Azure et l’ID client, elle prend l’ID utilisateur qui est enregistré dans les préférences partagées. 
La méthode d’assistance `getUserId()` récupère l’ID utilisateur de l’enregistrement.

Si l’authentification réussit, la méthode `onSuccess()` est invoquée. `onSuccess` enregistre le jeton d’accès dans `mAccessToken`. Lorsque vous effectuez un appel REST pour envoyer un message électronique, l’exemple place ce jeton d’accès dans un en-tête d’autorisation.
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
## Envoi d’un message électronique à l’aide d’Office 365

Une fois que l’utilisateur s’est connecté à Azure, l’exemple de connexion lui montre une activité pour envoyer un message électronique. L’exemple de connexion utilise la classe [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) pour envoyer le message lorsque l’utilisateur clique sur le bouton Envoyer un message.

### Classe d’assistance de carte REST
La classe [RESTHelper.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/RESTHelper.java) fournit une méthode permettant d’injecter un en-tête d’autorisation dans chaque appel REST effectué par l’exemple. Elle utilise le jeton d’accès fourni par le gestionnaire d’authentification.

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
### Classe UnifiedAPIController
La classe [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) génère la demande REST dans la méthode `sendMail()`.


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
### Interface UnifiedAPIService
L’interface [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) fournit des signatures de méthode pour les appels REST effectués par exemple à l’aide d’annotations Retrofit.

```java
    @POST("/me/sendMail")
    void sendMail(
            @Header("Content-type") String contentTypeHeader,
            @Body TypedString mail,
            Callback<Void> callback);


```

## Étapes suivantes
L’API Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la [documentation Microsoft Graph](http://graph.microsoft.io/docs) pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph.

Nous avons publié de nombreux exemples Android pour Office 365. Chacun de ces exemples repose sur les concepts que nous présentons dans l’exemple de connexion. Pour en savoir plus sur les tâches que vous pouvez exécuter avec vos applications Android, consultez la page relative à d’[autres exemples Android pour Office 365](http://aka.ms/androidgraphsamples) dans l’organisation GitHub d’Office.
 
