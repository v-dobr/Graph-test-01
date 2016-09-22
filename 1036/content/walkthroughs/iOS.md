# Appel de Microsoft Graph dans une application iOS

Dans cet article, nous examinons les tâches minimales requises pour connecter votre application à Office 365 et appeler l’API Microsoft Graph. Nous utilisons le code de l’exemple [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) pour expliquer les concepts principaux que vous devez mettre en œuvre dans votre application. Cet exemple présente les concepts de base pour s’authentifier auprès de Microsoft Azure Active Directory (AAD) et effectuer un appel de service simple par rapport au service de messagerie Office 365 à l’aide de l’API Microsoft Graph (envoi d’un courrier électronique). Il est recommandé de cloner ou de télécharger le projet de ce référentiel en accompagnement de cet article. 


Cet article fait référence à la [bibliothèque d’authentification Microsoft Azure Active Directory Authentication Library (ADAL) pour iOS et OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc), et à l’exemple [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) qui s’authentifie à l’aide de cette bibliothèque. Consultez ce référentiel pour plus d’informations sur l’utilisation et l’implémentation dans votre projet iOS.


## Vue d’ensemble

Pour appeler l’API Microsoft Graph, votre application iOS doit effectuer les opérations suivantes :

1. Inscription de l’application de Microsoft Azure Active Directory (AD).
2. Demande et acquisition d’un jeton d’accès d’Azure AD.
3. Utilisation du jeton d’accès dans une requête REST à l’API Microsoft Graph. 



## Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application pour accéder au compte professionnel ou scolaire d’un utilisateur à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Pour la gérer, vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com)

Vous pouvez également vous reporter à la section relative à l’**inscription de votre application native auprès du Portail de gestion Azure** dans l’article concernant l’[inscription manuelle de votre application avec Azure AD afin qu’elle puisse accéder aux API Office 365](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually) pour obtenir des instructions sur la façon d’inscrire manuellement l’application. Gardez à l’esprit les informations suivantes :

* Pour l’inscription, vous devrez fournir un URI de redirection. Il s’agit d’une valeur requise qui spécifie où un utilisateur sera redirigé après une tentative d’authentification réussie. Si vous ne spécifiez pas l’URI de redirection correct, la demande d’authentification échoue.
* Dans l’inscription, l’application doit disposer de l’**autorisation d’envoi de messages en tant qu’utilisateur connecté** pour **Microsoft Graph**.  


Prenez note des valeurs suivantes dans la page de **configuration** de votre application Azure.

* ID du client
* URI de redirection

Ces valeurs sont nécessaires pour configurer le flux OAuth dans votre application. 

## Demande et acquisition d’un jeton d’accès d’Azure AD

Pour demander et acquérir un jeton d’accès pour appeler l’API Microsoft Graph, vous pouvez utiliser **acquireAuthTokenWithResource:clientId:redirectUri:completionBlock:** fourni par la bibliothèque [Microsoft Azure Active Directory Authentication Library (ADAL) pour iOS et OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc). Ce Kit de développement logiciel (SDK) donne à votre application toutes les fonctionnalités de Microsoft Azure AD, y compris la prise en charge du protocole standard OAuth2, l’intégration de l’API web avec consentement au niveau de l’utilisateur, et la prise en charge de l’authentification à deux facteurs.

Cette méthode utilise les paramètres suivants :

1. **resourceID** - Il s’agit de la ressource requise à laquelle vous souhaitez accéder. Par exemple, nous souhaitons accéder à l’API Microsoft Graph donc cette valeur serait « https://graph.microsoft.com. »
2. **clientID** - La valeur donnée pour identifier votre application à la fin de l’inscription AAD.
3. **redirectURI** - Là encore, il s’agit d’une valeur requise qui spécifie où un utilisateur sera redirigé après une tentative d’authentification réussie.


Tout d’abord, vous devrez spécifier un contexte d’authentification. Ceci définit simplement l’autorité à partir de laquelle vous souhaitez obtenir votre jeton d’accès. Dans notre cas, c’est à partir d’un client AAD et vous devrez le déclarer :

    @property (strong,    nonatomic) ADAuthenticationContext *context;

Puis, initialisez-le avec l’emplacement de l’autorité (« https://login.microsoftonline.com/common ») :

    self.context = [ADAuthenticationContext authenticationContextWithAuthority:self.authority]; 


Dans l’exemple [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample), nous avons créé une classe d’authentification singleton (**AuthenticationManager**) à des fins de démonstration initialisée avec l’autorité et les paramètres requis. Là encore, cette classe est simplement un exemple de la façon d’aborder le flux de travail d’authentification. Un segment de code intéressant : 



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


À la première exécution de cette application, le gestionnaire d’authentification envoie une demande à l’autorité qui vous redirigera vers une page de connexion. 
Vous devrez fournir vos informations d’identification et la réponse contiendra le résultat de l’authentification. 
En cas de réussite, elle contiendra également vos jetons d’accès et d’actualisation. À la seconde exécution de cette application, et en supposant que vous n’avez pas effacé votre cache de jeton et vos cookies, 
le gestionnaire d’authentification utilisera le jeton d’accès ou d’actualisation dans le cache pour authentifier les demandes des clients. 
Ainsi, un appel au service sera effectué si vous devez obtenir un jeton d’accès. 


## Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Avec un jeton d’accès, votre application peut effectuer des demandes authentifiées à l’API Microsoft Graph. Votre application doit ajouter le jeton d’accès à l’en-tête de la requête HTTP sous **Autorization**.

L’exemple [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) envoie un message électronique à l’aide du point de terminaison sendMail dans l’API Microsoft Graph. Là encore, dans notre exemple, nous avons créé une classe d’authentification singleton (AuthenticationManager) qui est initialisée avec le jeton d’accès. Nous aurons besoin du jeton d’accès pour construire notre demande.



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

Comme vous pouvez le voir, la réponse est gérée avec les délégués NSURLConnection : NSURLConnectionDelegate et NSURLConnectionDataDelegate.

## Étapes suivantes

Si le jeton d’accès est arrivé à expiration ou arrive à expiration, vous pouvez utiliser **acquireTokenSilentWithResource:clientId:redirectUri:completionBlock:** d’ADAuthenticationContext pour acquérir un nouveau jeton d’accès. Son utilisation est traitée dans l’exemple [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample). En outre, vous pouvez rechercher le code pour effacer votre cache de jeton et les cookies stockés.  

L’API Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la [référence API](http://graph.microsoft.io/docs/api-reference/v1.0) pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph.

