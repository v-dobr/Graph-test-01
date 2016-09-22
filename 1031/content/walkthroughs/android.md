# Aufrufen von Microsoft Graph in einer Android-App

In diesem Artikel werden Aufgaben beschrieben, die zum Abrufen eines Zugriffstokens aus Azure Active Directory (AD) und Aufrufen von Microsoft Graph mindestens erforderlich sind. Wir verwenden den Code aus [Office 365 Android Connect-Beispiel unter Verwendung von Microsoft Graph](https://github.com/microsoftgraph/android-java-connect-rest-sample), um die wichtigsten Konzepte zu erläutern, die Sie in Ihrer App implementieren müssen.

In der folgenden Abbildung wird die Sendeaktivität von E-Mails der Beispiel-App dargestellt, die statt findet, nachdem ein Benutzer eine Verbindung zu Office 365 hergestellt hat.

![Screenshot eines Office 365 Android Unified API-Beispiels](./images/AndroidConnect.png)

## Übersicht

Zum Aufrufen der Microsoft Graph-API werden im [Office 365 Android Connect-Beispiel](https://github.com/microsoftgraph/android-java-connect-rest-sample) die folgenden Aufgaben durchgeführt.

1. Authentifizieren eines Benutzers und Abrufen eines Zugriffstokens durch Aufrufen von Methoden in der Azure Active Directory-Bibliothek.
2. Erstellen einer E-Mail-Anforderung als REST-Vorgang am Microsoft Graph-API-Endpunkt.

<!--<a name="register"/>-->
## Registrieren der Anwendung in Azure Active Directory

Bevor Sie  mit Office 365 arbeiten können, müssen Sie Ihre Anwendung registrieren und die Berechtigungen für die Verwendung der Microsoft Graph-Dienste festlegen. Mithilfe des [App-Registrierungstools](https://dev.office.com/app-registration) können Sie mit nur wenigen Klicks Ihre Anwendung für den Zugriff auf ein Geschäfts- oder Schulkonto des Benutzers registrieren. 
Zum Verwalten müssen Sie das [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) verwenden.

Alternativ können Sie die App manuell registrieren. Die Anweisungen dazu finden Sie im Abschnitt **Registrieren der systemeigenen App mit dem Azure-Verwaltungsportal** des Artikels [Manuelles Registrieren der App mit Azure AD, damit Sie auf Office 365-APIs zugreifen kann](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually). Beachten Sie dabei Folgendes:

* Konfigurieren Sie die **delegierten Berechtigungen**, die für Ihre App erforderlich sind. Für das Connect-Beispiel ist die Berechtigung zum **Senden von E-Mails als angemeldeter Benutzer** erforderlich.

Notieren Sie sich die folgenden Werte auf der Seite **Konfigurieren** Ihrer Azure-Anwendung.

* Client-ID
* Eine Umleitungs-URL

Sie benötigen diese Werte zum Konfigurieren des Authentifizierungscodes in Ihrer App.

## Gradle-Abhängigkeiten im Connect-Beispiel
Im Beispiel werden die Bibliotheksabhängigkeiten verwendet, die im folgenden build.gradle-Codeausschnitt dargestellt sind.

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
## Authentifizierung im Connect-Beispiel
Im Connect-Beispiel werden die Azure-App-Registrierungswerte und eine Benutzer-ID für die Authentifizierung verwendet. Im Connect-Beispiel werden zwei Arten von Authentifizierung unterstützt.

* Aufforderung zur Authentifizierung. Wird verwendet, wenn eine Benutzer-ID nicht im Cache mit den bevorzugten Einstellungen auf dem Android-Gerät gespeichert ist.
* Automatische Authentifizierung. Wird verwendet, wenn eine Benutzer-ID im Cache gespeichert ist und die Aufforderung nicht erforderlich ist.

Die [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java)-Klasse stellt eine `isConnected()`-Hilfsmethode für die Suche von im Cache gespeicherten Benutzer-IDs bereit und legt die zu verwendende Art der Authentifizierung fest.


```java
    private boolean isConnected(){
        SharedPreferences settings = this
                .mContextActivity
                .getSharedPreferences(PREFERENCES_FILENAME, Context.MODE_PRIVATE);

        return settings.contains(USER_ID_VAR_NAME);
    }

```

Bei beiden Arten der Authentifizierung sind für den ADAL-Authentifizierungsfluss die Client-ID und die Umleitungs-URL erforderlich, die Sie während des Registrierungsvorgangs in Azure erhalten haben. In dem Beispiel werden diese Zeichenfolgen im Quellcode beibehalten und abgerufen, bevor das Authentifizierungs-Manager-Objekt den Benutzer authentifiziert.

Die [Constants.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/Constants.java)-Schnittstelle stellt zwei statische Zeichenfolgen für die Client-ID und die Umleitungs-URL zur Verfügung.

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
### Erstellen der AuthenticationManager-Klasse
Der [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java)-Konstruktor verwendet keine Argumente, legt jedoch ein Klassenzeichenfolgenfeld aus der Constants.java-Datei mit der URL des Graph-Endpunkts fest. Diese Ressourcenzeichenfolge wird für beide Arten der Authentifizierung verwendet.

```java
    private AuthenticationManager() {
        mResourceId = Constants.UNIFIED_ENDPOINT_RESOURCE_ID;
    }
```

### Aufforderung zur Authentifizierung

Die [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java)-Klasse stellt eine `authenticatePrompt()`-Methode zum Abrufen des Zugriffstokens bereit, das für REST-Aufrufe im einheitlichen Endpunkt verwendet wird.

Die `acquireToken()`-Methode für die ADAL-Bibliothek ist asynchron. Die Methodenargumente enthalten einen Verweis auf den Kontext der aktuellen Aktivität zusammen mit der Ressource, der Client-ID und der Umleitungs-URL. Anhand des aktuellen Aktivitätsverweises kann die ADAL-Bibliothek eine Seite zum Anfordern von Anmeldeinformationen in der Aktivität anzeigen. 
Wenn die Authentifizierung erfolgreich ist, ruft die ADAL-Bibliothek den `onSuccess()`-Rückruf auf. Dieser Rückruf führt die folgenden zwei Aktionen durch:

* Er speichert das Zugriffstoken in `mAccessToken`. Bei einem REST-Aufruf zum Senden einer E-Mail wird im Beispiel dieses Zugriffstoken im Autorisierungsheader platziert.
* Er speichert die Benutzer-ID in den bevorzugten Einstellungen.


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

###Automatische Authentifizierung
Die [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java)-Klasse stellt eine `authenticateSilent()`-Methode zum Abrufen des Zugriffstokens bereit, das für REST-Aufrufe im einheitlichen Endpunkt verwendet wird.

Die `acquireTokenSilent()`-Methode für die ADAL-Bibliothek ist asynchron. Neben der Azure-Registrierungsclient-ID und der Ressourcen-ID verwendet diese die Benutzer-ID, die in den freigegebenen Voreinstellungen gespeichert ist. 
Die `getUserId()`-Hilfsmethode ruft die Benutzer-ID aus dem Speicher ab.

Wenn die Authentifizierung erfolgreich ist, wird die `onSuccess()`-Methode aufgerufen. `onSuccess` speichert das Zugriffstoken in `mAccessToken`. Bei einem REST-Aufruf zum Senden einer E-Mail wird im Beispiel dieses Zugriffstoken im Autorisierungsheader platziert.
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
## Senden einer E-Mail mit Office 365

Nachdem der Benutzer sich bei Azure angemeldet hat, wird im Connect-Beispiel eine Aktivität zum Senden einer E-Mail angezeigt. Im Connect-Beispiel wird die [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java)-Klasse zum Senden der Nachricht verwendet, wenn der Benutzer auf die Schaltfläche „E-Mail senden“ klickt.

### REST-Adapterhilfsklasse
Die [RESTHelper.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/RESTHelper.java)-Klasse stellt eine Methode zum Einfügen eines Autorisierungsheaders in jedem REST-Aufruf in dem Beispiel bereit. Sie verwendet das von dem Authentifizierungs-Manager bereitgestellt Zugriffstoken.

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
### UnifiedAPIController-Klasse
Die [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java)-Klasse generiert die REST-Anforderung in der `sendMail()`-Methode.


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
### Die UnifiedAPIService-Schnittstelle
Die [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java)-Schnittstelle stellt Methodensignaturen für REST-Aufrufe mit Retrofit-Anmerkungen im Beispiel bereit.

```java
    @POST("/me/sendMail")
    void sendMail(
            @Header("Content-type") String contentTypeHeader,
            @Body TypedString mail,
            Callback<Void> callback);


```

## Nächste Schritte
Die Microsoft Graph-API ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit der Microsoft Graph-API finden Sie in der [Microsoft Graph-Dokumentation](http://graph.microsoft.io/docs).

Wir haben zahlreiche Android-Beispiele für Office 365 veröffentlicht. Jedes dieser Beispiele baut auf Konzepten auf, die im Connect-Beispiel erläutert wurden. Informationen zu weiteren Möglichkeiten mit Android-Apps finden Sie unter [Weitere Android-Beispiele für Office 365](http://aka.ms/androidgraphsamples) in der Office GitHub-Organisation.
 
