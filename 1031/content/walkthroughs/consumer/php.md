# Erste Schritte mit Microsoft Graph in einer PHP-App

Dieser Artikel beschreibt die erforderlichen Aufgaben zum Abrufen eines Zugriffstokens vom v2.0-Authentifizierungsendpunkt und zum Aufrufen von Microsoft Graph. Sie werden durch die Erstellung der [Connect-Beispiels für PHP](https://github.com/microsoftgraph/php-connect-rest-sample) geführt und erhalten Informationen zu den Hauptkonzepten, die Sie zur Verwendung von Microsoft Graph implementieren. In diesem Artikel wird auch beschrieben, wie Sie mithilfe von REST-Aufrufen auf Microsoft Graph zugreifen.

Um Microsoft Graph in Ihrer PHP-App zu verwenden, müssen Sie für Benutzer die Microsoft-Anmeldeseite anzeigen. Auf dem folgenden Screenshot ist eine Anmeldeseite für Microsoft-Konten dargestellt.

![Anmeldeseite für Microsoft-Konten](images/MicrosoftSignIn.png)

**Sie möchten keine App erstellen?** Laden Sie sich für einen Schnelleinstieg das [Connect-Beispiel für PHP](https://github.com/microsoftgraph/php-connect-rest-sample) herunter, auf dem dieser Artikel basiert.


## Voraussetzungen

Für die ersten Schritte benötigen Sie: 

- Ein [Microsoft-Konto](https://www.outlook.com/) oder ein [Office 365 for Business-Konto](http://dev.office.com/devprogram).
- PHP, Version 5.5.9 oder höher
- [Composer](https://getcomposer.org/)


## Registrieren der App
Registrieren Sie eine App im Microsoft App-Registrierungsportal. Dadurch wird die APP-ID und das Kennwort generiert, mit der bzw. dem Sie die App konfigurieren.

1. Melden Sie sich beim [Microsoft-App-Registrierungsportal](https://apps.dev.microsoft.com/) entweder mit Ihrem persönlichen oder geschäftlichen Konto oder mit Ihrem Schulkonto an.

2. Klicken Sie auf **App hinzufügen**.

3. Geben Sie einen Namen für die App ein, und wählen Sie **Anwendung erstellen** aus. 
    
   Die Registrierungsseite wird angezeigt, und die Eigenschaften der App werden aufgeführt.

4. Wählen Sie **Neues Kennwort generieren** aus.

5. Kopieren Sie die Anwendungs-ID und das Kennwort:

6. Wählen Sie **Plattform hinzufügen** und **Web** aus.

7. Geben Sie im Feld **Umleitungs-URI** `http://localhost:8000/oauth` ein.

8. Wählen Sie **Speichern** aus.


## Konfigurieren des Projekts

Starten Sie ein neues Projekt mithilfe von Composer. Um ein neues PHP-Projekt mithilfe des Laravel-Frameworks zu erstellen, geben Sie den folgenden Befehl ein:

```bash
composer create-project --prefer-dist laravel/laravel getstarted
```
 
Auf diese Weise wird ein **getstarted**-Ordner erstellt, den wir für dieses Projekt verwenden können.

## Authentifizierung des Benutzers und Abrufen eines Zugriffstokens
Wir verwenden eine OAuth-Bibliothek, um den Authentifizierungsprozess zu vereinfachen. [Die PHP League](http://thephpleague.com/) bietet eine [OAuth-Clientbibliothek](https://github.com/thephpleague/oauth2-client) an, die wir in diesem Projekt verwenden können.

### Hinzufügen der Abhängigkeit zu Composer

Öffnen Sie die `composer.json`-Datei, und fügen Sie die folgende Abhängigkeit in den Abschnitt **require** ein:

```json
"league/oauth2-client": "^1.4"
```

Aktualisieren Sie die Abhängigkeiten, indem Sie den folgenden Befehl ausführen:

```bash
composer update
```

### Starten des Authentifizierungsflusses

1. Öffnen Sie die Datei **resources** > **views** > **welcome.blade.php**. Ersetzen Sie das **title**-div-Element durch den folgenden Code.
    ```html
    <div class="title" onClick="window.location='/oauth'">Sign in to Microsoft</div>
    ```
    
2. Fügen Sie die `Illuminate\Http\Request`-Klasse mithilfe von Type Hinting in die Datei **app** > **Http** > **routes.php** ein. Fügen Sie die folgende Zeile vor einer Routendeklaration ein.
    ```php
    use Illuminate\Http\Request;
    ```
    
3. Fügen Sie eine */oauth*-Route zur Datei **app** > **Http** > **routes.php** hinzu. Um die Route hinzuzufügen, kopieren Sie den folgenden Code hinter die Standardroutendeklaration. Fügen Sie die **Anwendungs-ID** und das **Kennwort** Ihrer App in den Platzhalter ein, der mit **\<YOUR_APPLICATION_ID\>** bzw. **\<YOUR_PASSWORD\>** markiert ist.
    ```php
    Route::get('/oauth', function () {
        $provider = new \League\OAuth2\Client\Provider\GenericProvider([
            'clientId'                => '<YOUR_APPLICATION_ID>',
            'clientSecret'            => '<YOUR_PASSWORD>',
            'redirectUri'             => 'http://localhost:8000/oauth',
            'urlAuthorize'            => 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
            'urlAccessToken'          => 'https://login.microsoftonline.com/common/oauth2/v2.0/token',
            'urlResourceOwnerDetails' => '',
            'scopes'                  => 'openid mail.send'
        ]);

        if (!$request->has('code')) {
            return redirect($provider->getAuthorizationUrl());
        }
    });
    ```
    
Nun sollten Sie eine PHP-App haben, in der *Bei Microsoft anmelden* angezeigt wird. Wen Sie auf den Text klicken, wird in der App die Microsoft-Anmeldeseite angezeigt. Der nächste Schritt besteht darin, den Code zu behandeln, den der Autorisierungsserver an den Umleitung-URI sendet, und diesen durch ein Zugriffstoken zu ersetzen.

### Ersetzen des Autorisierungscodes durch ein Zugriffstoken

Wir müssen die Antwort des Autorisierungsserver verarbeiten, die einen Code enthält, der durch ein Zugriffstoken ersetzt werden kann.

Aktualisieren Sie die */oauth*-Route so, dass ein Zugriffstoken mit dem Autorisierungscode abgerufen werden kann. Öffnen Sie hierfür die Datei **app** > **Http** > **routes.php**, und fügen Sie die folgende *else*Bedingungsklausel zu der vorhandenen *if*-Anweisung hinzu.

```php
if (!$request->has('code')) {
    ...
    // add the following lines
} else {
    $accessToken = $provider->getAccessToken('authorization_code', [
        'code'     => $request->input('code')
    ]);
    exit($accessToken->getToken());
}
```
    
Beachten Sie, dass in dieser Zeile `exit($accessToken->getToken());` ein Zugriffstoken vorhanden ist. Sie können nun Code hinzufügen, um Microsoft Graph aufzurufen. 

## Aufrufen von Microsoft Graph mithilfe von REST
Microsoft Graph kann mithilfe von REST aufgerufen werden. Die [Microsoft Graph-REST-API](http://graph.microsoft.io/docs) macht mehrere APIs aus Microsoft-Clouddiensten über einen einzelnen REST-API-Endpunkt verfügbar. Um die REST-API zu verwenden, ersetzen Sie die Zeile `exit($accessToken->getToken());` durch den folgenden Code. Fügen Sie Ihre E-Mail-Adresse in den Platzhalter ein, der mit **\<YOUR_EMAIL_ADDRESS\>** markiert ist.

```php
$client = new \GuzzleHttp\Client();

$email = "{
    Message: {
    Subject: 'Sent using the Microsoft Graph REST API',
    Body: {
        ContentType: 'text',
        Content: 'This is the email body'
    },
    ToRecipients: [
        {
            EmailAddress: {
            Address: '<YOUR_EMAIL_ADDRESS>'
            }
        }
    ]
    }}";

$response = $client->request('POST', 'https://graph.microsoft.com/v1.0/me/sendmail', [
    'headers' => [
        'Authorization' => 'Bearer ' . $accessToken->getToken(),
        'Content-Type' => 'application/json;odata.metadata=minimal;odata.streaming=true'
    ],
    'body' => $email
]);
if($response.getStatusCode() === 201) {
    exit('Email sent, check your inbox');
} else {
    exit('There was an error sending the email. Status code: ' . $response.getStatusCode());
}
```

## Ausführen der App
Sie können Ihre PHP-App nun testen.

1. Geben Sie in der Shell den folgenden Befehl ein:
    ```bash
    php artisan serve
    ```
    
2. Navigieren Sie zu `http://localhost:8000` im Webbrowser.
3. Wählen Sie **Bei Microsoft anmelden** aus.
4. Melden Sie sich mit Ihrem persönlichen Konto oder mit Ihrem Geschäfts- oder Schulkonto an, und gewähren Sie die erforderlichen Berechtigungen.

Überprüfen Sie den Posteingang der E-Mail-Adresse, die Sie im Abschnitt [Aufrufen von Microsoft Graph mithilfe von REST](#call-the-microsoft-graph-using-rest) konfiguriert haben. Dort sollten Sie eine E-Mail von dem Konto vorfinden, das Sie zum Anmelden bei der App verwendet haben.

## Nächste Schritte
- Testen Sie die REST-API mithilfe des [Graph-Explorers](https://graph.microsoft.io/graph-explorer).


## Siehe auch
* [Übersicht über Microsoft Graph](http://graph.microsoft.io/docs)
* [Protokolle für Azure AD, Version 2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Token für Azure AD, Version 2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
