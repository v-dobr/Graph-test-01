# Appel de Microsoft Graph dans une application ASP.NET MVC

Dans cet article, nous examinons les tâches minimales requises pour connecter votre application à Office 365 et appeler l’API Microsoft Graph. Cette rubrique ne créera pas une application de A à Z. Nous utilisons le code de l’[exemple de connexion d’une application ASP.NET MVC à Office 365 à l’aide de Microsoft Graph](https://github.com/microsoftgraph/aspnet-connect-rest-sample) pour expliquer les concepts principaux que vous devez mettre en œuvre dans votre application.

Voici une capture d’écran de la page d’envoi de courrier.

![Capture d’écran de l’exemple de connexion d’une application ASP.NET MVC à Office 365](./images/O365AspNetMVCSendMailPageScreenshot.png)

## Vue d’ensemble

Pour appeler l’API Microsoft Graph, vous devez effectuer les tâches suivantes.

1. Inscription de l’application dans Azure Active Directory
2. Authentification d’un utilisateur et obtention d’un jeton d’accès en appelant des méthodes sur la bibliothèque Azure AD Authentication Library pour .NET. (ADAL)
3. Utilisation d’ADAL pour obtenir un jeton d’accès
4. Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph
5. Déconnexion de la session

<!--<a name="register"></a>-->
## Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application pour accéder au compte professionnel ou scolaire d’un utilisateur à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Pour la gérer, vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com)

Vérifiez que vous spécifiez http://localhost:55065/ comme [URL d’authentification](https://msdn.microsoft.com/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp).

* Vérifiez que vous spécifiez http://localhost:55065/ comme **URL d’authentification**.
* Une fois que vous avez inscrit l’application, [configurez les **autorisations déléguées**](https://github.com/microsoftgraph/aspnet-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) requises par votre application Angular. L’exemple de connexion requiert l’autorisation d’**envoi de messages en tant qu’utilisateur connecté**.

Prenez note des valeurs suivantes dans la page de **configuration** de votre application Azure car ces valeurs sont requises pour configurer dans votre application.

* ID client (unique pour votre application)
* Clé (également appelé secret client)
* URL de réponse (également appelée URL de redirection). Pour cet exemple, il s’agit de http://localhost:55065/.

  > Remarque :  la valeur de l’URL de réponse est renseignée automatiquement avec la valeur de l’URL d’authentification que vous spécifiez lorsque vous inscrivez l’application.

<!--<a name="#auth"></a>-->
## Authentification dans l’exemple de connexion

La bibliothèque d’authentification Azure AD Authentication Library (ADAL) pour .NET permet aux développeurs d’applications clientes d’authentifier les utilisateurs et ensuite d’obtenir les jetons d’accès pour effectuer des appels d’API.  Vous pouvez inclure cette bibliothèque dans votre projet ASP.NET MVC via **Gérer les packages NuGet** dans Visual Studio.

Voici une capture d’écran de la page d’accueil.

![Capture d’écran de l’exemple de connexion d’une application ASP.NET MVC à Office 365](./images/O365AspNetMVCHomePageScreenshot.png)

Le flux d’authentification peut être divisé en deux étapes de base :

1. Demande d’un code d’autorisation
2. Utilisation d’un code d’autorisation pour demander un jeton d’accès.

>  **Remarque** : Vous obtiendrez également un jeton d’actualisation avec le jeton d’accès.

L’exemple de connexion utilise les valeurs d’inscription de l’application Azure et un ID utilisateur pour l’authentification. Le flux d’authentification ADAL requiert l’ID client, la clé et l’URL de réponse (également appelée URL de redirection) que vous obtenez dans le processus d’inscription Azure.

Pour demander un code d’autorisation, redirigez d’abord l’application vers l’URL de demande d’autorisation Azure AD comme illustré ci-dessous (voir fichier HomeController.cs).


```c#
        public ActionResult Login()
        {
            if (string.IsNullOrEmpty(Settings.ClientId) || string.IsNullOrEmpty(Settings.ClientSecret))
            {
                ViewBag.Message = "Please set your client ID and client secret in the Web.config file";
                return View();
            }


            var authContext = new AuthenticationContext(Settings.AzureADAuthority);

            // Generate the parameterized URL for Azure login.
            Uri authUri = authContext.GetAuthorizationRequestURL(
                Settings.O365UnifiedAPIResource,
                Settings.ClientId,
                loginRedirectUri,
                UserIdentifier.AnyUser,
                null);

            // Redirect the browser to the login page, then come back to the Authorize method below.
            return Redirect(authUri.ToString());
        }

```
Lorsque cette méthode de **connexion** est appelée, l’application redirige l’utilisateur vers une page de connexion. L’application accède alors à la page de connexion. Une fois que les informations d’identification utilisateur sont authentifiées, Azure redirige l’application vers l’URL de redirection mentionnée dans le code indiqué par *loginRedirectUri*. Cette URL de redirection est une URL vers une autre action dans l’application ASP.NET MVC comme illustré.

```c#

 Uri loginRedirectUri => new Uri(Url.Action(nameof(Authorize), "Home", null, Request.Url.Scheme));

```
L’URL contient également le code d’autorisation mentionné aux étapes 1 et 2 ci-dessus.  Elle reçoit le code d’autorisation à partir des paramètres de demande. Grâce au code d’autorisation, l’application effectue un appel à Azure Active pour obtenir le jeton d’accès. Une fois le jeton d’accès obtenu, nous le stockons dans la session pour pouvoir l’utiliser pour des demandes multiples.

L’action d’autorisation mentionnée dans l’action de l’URL de redirection ressemble à ceci.

```c#
        public async Task<ActionResult> Authorize()
        {
            var authContext = new AuthenticationContext(Settings.AzureADAuthority);


            // Get the token.
            var authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
                Request.Params["code"],                                         // the auth 'code' parameter from the Azure redirect.
                loginRedirectUri,                                               // same redirectUri as used before in Login method.
                new ClientCredential(Settings.ClientId, Settings.ClientSecret), // use the client ID and secret to establish app identity.
                Settings.O365UnifiedAPIResource);

            // Save the token in the session.
            Session[SessionKeys.Login.AccessToken] = authResult.AccessToken;

            // Get info about the current logged in user.
            Session[SessionKeys.Login.UserInfo] = await UnifiedApiHelper.GetUserInfoAsync(authResult.AccessToken);

            return RedirectToAction(nameof(Index), "Message");

        }

```
>  **Remarque** :  pour plus d’informations sur le flux d’autorisation, voir [Flux d’octroi d’un code d’autorisation] (https://msdn.microsoft.com/fr-fr/library/azure/dn645542.aspx)

<!--<a name="request"></a>-->
## Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Une fois que l’utilisateur s’est connecté, l’exemple de connexion lui montre une activité pour envoyer un message électronique.  Avec un jeton d’accès, votre application peut effectuer des demandes authentifiés à l’API Microsoft Graph.

Par exemple, le fichier UnifiedApiHelper.cs contient le code qui :

1)  Obtient les informations sur l’utilisateur connecté actuel.  La méthode ``GetUserInfoAsync`` prend un seul argument (valeur de jeton d’accès) afin d’effectuer un appel à **https://graph.microsoft.com/v1.0/me** pour obtenir des informations sur l’utilisateur actuellement connecté.

 ```c#

        public static async Task<UserInfo> GetUserInfoAsync(string accessToken)
        {
            UserInfo myInfo = new UserInfo();

            using (var client = new HttpClient())
            {
                using (var request = new HttpRequestMessage(HttpMethod.Get, Settings.GetMeUrl))
                {
                    request.Headers.Accept.Add(Json);
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

                    using (var response = await client.SendAsync(request))
                    {
                        if (response.StatusCode == HttpStatusCode.OK)
                        {
                            var json = JObject.Parse(await response.Content.ReadAsStringAsync());
                            myInfo.Name = json?["displayName"]?.ToString();
                            myInfo.Address = json?["mail"]?.ToString().Trim().Replace(" ", string.Empty);

                        }
                    }
                }
            }

            return myInfo;
        }

```



2)  Construit et envoie le message que l’utilisateur connecté souhaite envoyer par courrier électronique. La méthode ``SendMessageAsync`` construit et envoie une demande POST à l’URL de ressource **https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail** à l’aide de la valeur de jeton d’accès comme l’un des arguments.


```c#

        public static async Task<SendMessageResponse> SendMessageAsync(string accessToken, SendMessageRequest sendMessageRequest)
        {
            var sendMessageResponse = new SendMessageResponse { Status = SendMessageStatusEnum.NotSent };

            using (var client = new HttpClient())
            {
                using (var request = new HttpRequestMessage(HttpMethod.Post, Settings.SendMessageUrl))
                {
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
                    request.Content = new StringContent(JsonConvert.SerializeObject(sendMessageRequest), Encoding.UTF8, "application/json");
                    using (HttpResponseMessage response = await client.SendAsync(request))
                    {
                        if (response.IsSuccessStatusCode)
                        {
                            sendMessageResponse.Status = SendMessageStatusEnum.Sent;
                            sendMessageResponse.StatusMessage = null;
                        }
                        else
                        {
                            sendMessageResponse.Status = SendMessageStatusEnum.Fail;
                            sendMessageResponse.StatusMessage = response.ReasonPhrase;
                        }
                    }
                }
            }

            return sendMessageResponse;
        }

```


Le fichier ``MessageController.cs `` contient du code qui gère les messages électroniques. Par exemple, le bouton **Envoyer un message**. La méthode ``SendMessageSubmit `` envoie le message lorsque les utilisateurs cliquent sur le bouton **Envoyer un message**.


```c#

        public async Task<ActionResult> SendMessageSubmit(UserInfo userInfo)
        {
            // After Index method renders the View, user clicks Send Mail, which comes in here.
            EnsureUser(ref userInfo);

            // Send email using O365 unified API.
            var sendMessageResult = await UnifiedApiHelper.SendMessageAsync(
                (string)Session[SessionKeys.Login.AccessToken],
                GenerateEmail(userInfo));

            // Reuse the Index view for messages (sent, not sent, fail) .
            // Redirect to tell the browser to call the app back via the Index method.
            return RedirectToAction(nameof(Index), new RouteValueDictionary(new Dictionary<string,object>{
                { "Status", sendMessageResult.Status },
                { "StatusMessage", sendMessageResult.StatusMessage },
                { "Address", userInfo.Address },
            }));
        }

```


La méthode ``CreateEmailObject`` crée l’objet de messagerie dans le contrat de données/format de demande requis par le corps POST :


  ```c#

        private SendMessageRequest CreateEmailObject(UserInfo to, string subject, string body)
        {
            return new SendMessageRequest
            {
                Message = new Message
                {
                    Subject = subject,
                    Body = new MessageBody
                    {
                        ContentType = "Html",
                        Content = body
                    },
                    ToRecipients = new List<Recipient>
                    {
                        new Recipient
                        {
                            EmailAddress = new UserInfo
                            {
                                 Name =  to.Name,
                                 Address = to.Address
                            }
                        }
                    }
                },
                SaveToSentItems = true
            };

```

Une autre tâche consiste à construire une chaîne de message JSON valide et à l’envoyer au point de terminaison ``https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail`` à l’aide d’une demande HTTP POST. Étant donné que le corps du message électronique doit être envoyé sous la forme d’un document HTML, la demande définit la valeur ``ContentType`` du message électronique sur HTML et code le contenu en tant que JSON pour la requête HTTP POST. Le fichier UnifiedApiMessageModels.cs contient les contrats de schéma ou de données entre cette application et le serveur de l’API unifiée d’Office 365.



```c#


    public class SendMessageResponse
    {
        public SendMessageStatusEnum Status { get; set; }
        public string StatusMessage { get; set; }
    }

    public class SendMessageRequest
    {
        public Message Message { get; set; }

        public bool SaveToSentItems { get; set; }
    }

    public class Message
    {
        public string Subject { get; set; }
        public MessageBody Body { get; set; }
        public List<Recipient> ToRecipients { get; set; }
    }
    public class Recipient
    {
        public UserInfo EmailAddress { get; set; }
    }

    public class MessageBody
    {
        public string ContentType { get; set; }
        public string Content { get; set; }
    }

    public class UserInfo
    {
        public string Name { get; set; }
        public string Address { get; set; }
    }

}

```
<!--<a name="logout"></a>-->
## Déconnexion de la session

Lorsque l’utilisateur clique sur **Déconnecter** dans la page d’envoi du courrier, l’utilisateur est déconnecté de la session. Le code effectue cette opération en :
* désactivant la session locale ;
* redirigeant le navigateur vers le point de terminaison de déconnexion (pour qu’Azure puisse effacer ses propres cookies).

La méthode de **déconnexion** (voir fichier HomeController.cs) décrit cette procédure.


```c#
        public ActionResult Logout()
        {
            Session.Clear();
            return Redirect(Settings.LogoutAuthority + logoutRedirectUri.ToString());
        }

```

##Étapes suivantes
L’API Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la référence d’API pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph. 
Découvrez nos autres exemples ASP.NET sur [GitHub](http://aka.ms/aspnetgraphsamples).


