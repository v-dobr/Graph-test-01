# Prise en main de Microsoft Graph dans une application PHP

Cet article décrit les tâches requises pour obtenir un jeton d’accès du point de terminaison d’authentification v2.0 et appeler Microsoft Graph. Il vous guidera dans la construction de l’[exemple de connexion pour PHP](https://github.com/microsoftgraph/php-connect-rest-sample) et décrit les principaux concepts à mettre en œuvre pour utiliser Microsoft Graph. Cet article explique également comment accéder à Microsoft Graph à l’aide d’appels REST.

Pour utiliser Microsoft Graph dans votre application PHP, vous devez afficher la page de connexion Microsoft à vos utilisateurs. La capture d’écran suivante montre la page de connexion pour les comptes Microsoft.

![Page de connexion pour les comptes Microsoft](images/MicrosoftSignIn.png)

**Pas envie de créer une application ?** Soyez rapidement opérationnel en téléchargeant l’[exemple de connexion pour PHP](https://github.com/microsoftgraph/php-connect-rest-sample) sur lequel repose cet article.


## Conditions préalables

Voici ce dont vous aurez besoin pour démarrer : 

- Un [compte Microsoft](https://www.outlook.com/) ou un [compte Office 365 pour les entreprises](http://dev.office.com/devprogram)
- PHP version 5.5.9 ou supérieure
- [Compositeur](https://getcomposer.org/)


## Inscription de l’application
Inscrivez une application sur le portail d’inscription des applications Microsoft. Cette action génère l’ID d’application et le mot de passe que vous utiliserez pour configurer l’application.

1. Connectez-vous au [portail d’inscription des applications Microsoft](https://apps.dev.microsoft.com/) en utilisant votre compte personnel, professionnel ou scolaire.

2. Choisissez **Ajouter une application**.

3. Entrez un nom pour l’application, puis choisissez **Créer une application**. 
    
   La page d’inscription s’affiche, répertoriant les propriétés de votre application.

4. Choisissez **Générer un nouveau mot de passe**.

5. Copiez l’ID de l’application et le mot de passe.

6. Choisissez **Ajouter une plateforme** et **Web**.

7. Dans le champ **URI de redirection**, saisissez `http://localhost:8000/oauth`.

8. Cliquez sur **Enregistrer**.


## Configurer le projet

Démarrez un nouveau projet à l’aide du compositeur. Pour créer un projet PHP à l’aide de l’infrastructure Laravel, saisissez la commande suivante :

```bash
composer create-project --prefer-dist laravel/laravel getstarted
```
 
Cette action crée un dossier **getstarted** que nous pouvons utiliser pour ce projet.

## Authentifier l’utilisateur et obtenir un jeton d’accès
Nous allons utiliser une bibliothèque OAuth pour simplifier le processus d’authentification. [La Ligue des Packages Extraordinaires](http://thephpleague.com/) fournit une [bibliothèque cliente OAuth](https://github.com/thephpleague/oauth2-client) que nous pouvons utiliser dans ce projet.

### Ajouter la dépendance au compositeur

Ouvrez le fichier `composer.json` et incluez la dépendance suivante dans la section **require** :

```json
"league/oauth2-client": "^1.4"
```

Mettez à jour les dépendances en exécutant la commande suivante :

```bash
composer update
```

### Démarrer le flux d’authentification

1. Ouvrez le fichier **resources** > **views** > **welcome.blade.php**. Remplacez l’élément div **title** par le code suivant.
    ```html
    <div class="title" onClick="window.location='/oauth'">Sign in to Microsoft</div>
    ```
    
2. Saisissez des informations sur la classe `Illuminate\Http\Request` sur le fichier **app** > **Http** > **routes.php**. Ajoutez la ligne suivante avant toute déclaration d’acheminement.
    ```php
    use Illuminate\Http\Request;
    ```
    
3. Ajoutez un acheminement */oauth* au fichier **app** > **Http** > **routes.php**. Pour ajouter l’acheminement, copiez le code suivant après la déclaration d’acheminement par défaut. Insérez l’**ID d’application** et le **mot de passe** de votre application dans l’espace réservé marqué avec **\<YOUR_APPLICATION_ID\>** et **\<YOUR_PASSWORD\>** respectivement.
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
    
À ce stade, vous devez disposer d’une application PHP qui affiche *Se connecter à Microsoft*. Si vous cliquez sur le texte, l’application affiche la page de connexion à Microsoft. L’étape suivante consiste à gérer le code que le serveur d’autorisation envoie à l’URI de redirection et à l’échanger contre un jeton d’accès.

### Échanger le code d’autorisation contre un jeton d’accès

Votre application doit être prête à traiter la réponse du serveur d’autorisation, qui contient un code que vous pouvez échanger contre un jeton d’accès.

Mettez à jour l’acheminement */oauth* afin qu’il puisse obtenir un jeton d’accès avec le code d’autorisation. Pour ce faire, ouvrez le fichier **app** > **Http** > **routes.php** et ajoutez la clause conditionnelle *else* suivante à l’instruction *if* existante.

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
    
Notez que nous avons un jeton d’accès sur cette ligne `exit($accessToken->getToken());`. Vous êtes maintenant prêt à ajouter un code pour appeler Microsoft Graph. 

## Appeler Microsoft Graph à l’aide de REST
Nous pouvons appeler Microsoft Graph à l’aide de REST. L’[API REST Microsoft Graph](http://graph.microsoft.io/docs) expose plusieurs API des services de cloud computing Microsoft via un seul point de terminaison d’API REST. Pour utiliser l’API REST, remplacez la ligne `exit($accessToken->getToken());` par le code suivant. Insérez votre adresse électronique dans l’espace réservé marqué avec **\<YOUR_EMAIL_ADDRESS\>**.

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

## Exécuter l’application
Vous êtes prêt à essayer votre application PHP.

1. Dans votre interpréteur de commandes, entrez la commande suivante :
    ```bash
    php artisan serve
    ```
    
2. Accédez à `http://localhost:8000` dans votre navigateur web.
3. Choisissez **Se connecter à Microsoft**.
4. Connectez-vous à votre compte personnel, professionnel ou scolaire et accordez les autorisations demandées.

Vérifiez la boîte de réception de l’adresse électronique que vous avez configurée dans la section [Appeler Microsoft Graph à l’aide de REST](#call-the-microsoft-graph-using-rest). Vous devriez avoir reçu un courrier électronique du compte utilisé pour vous connecter à l’application.

## Étapes suivantes
- Testez l’API REST à l’aide de l’[Afficheur Graph](https://graph.microsoft.io/graph-explorer).


## Voir aussi
* [Présentation de Microsoft Graph](http://graph.microsoft.io/docs)
* [Protocoles Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Jetons Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
