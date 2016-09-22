# Appel de Microsoft Graph dans une application Ruby 

Dans cet article, nous examinons les tâches minimales requises pour obtenir un jeton d’accès d’Azure Active Directory (AD) et appeler l’API Microsoft Graph. Nous utilisons le code de l’[exemple de connexion de Ruby à Office 365 à l’aide de Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample) pour expliquer les concepts principaux que vous devez mettre en œuvre dans votre application.

![Capture d’écran d’un exemple de connexion de Ruby à Office 365](./images/web-screenshot.png)

## Vue d’ensemble

Pour appeler l’API Microsoft Graph, votre application Ruby doit effectuer les tâches suivantes.

1. Inscription de l’application dans Azure Active Directory
2. Redirection du navigateur vers la page de connexion
3. Réception d’un code d’autorisation dans la page de votre URL de réponse
4. Demande de jeton d’accès à partir du point de terminaison du jeton
5. Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph 

<!--<a name="register"/>-->
## Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application pour accéder au compte professionnel ou scolaire d’un utilisateur à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Pour la gérer, vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com)

Vous pouvez également consulter la section relative à l’[inscription de votre application de serveur web auprès du Portail de gestion Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) pour obtenir des instructions sur la façon d’inscrire manuellement l’application. Gardez à l’esprit les informations suivantes :

* Spécifiez un itinéraire dans votre application Ruby comme **URL d’authentification** à l’étape 6. Dans le cas de l’exemple de connexion, il s’agit de [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L41).
* [Configurez les **autorisations déléguées**](https://github.com/microsoftgraph/ruby-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) requises par votre application. L’exemple de connexion requiert l’autorisation d’**envoi de messages en tant qu’utilisateur connecté**.

Prenez note des valeurs suivantes dans la page de **configuration** de votre application Azure.

* ID du client
* Une clé valide
* URL de réponse

Ces valeurs sont nécessaires comme paramètres dans le flux OAuth dans votre application.

<!--<a name="redirect"/>-->
## Redirection du navigateur vers la page de connexion

Votre application doit rediriger le navigateur vers la page de connexion pour obtenir un code d’autorisation et poursuivre le flux OAuth.

Dans l’exemple de connexion, la redirection est gérée par la bibliothèque OmniAuth. Notre application délègue uniquement l’exécution à l’itinéraire [`/auth/azureactivedirectory`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L30) géré par OmniAuth.

<!--<a name="authcode"/>-->
## Réception d’un code d’autorisation dans la page de votre URL de réponse

Une fois que l’utilisateur se connecte, le flux renvoie le navigateur à l’URL de réponse dans votre application. Azure ajoute un code d’autorisation dans la chaîne de requête. L’exemple de connexion utilise l’itinéraire [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L38) à cet effet.

Le code d’autorisation est fourni dans la variable de la chaîne de requête `code`. L’exemple de connexion enregistre le code dans une variable locale pour une utilisation ultérieure.

```ruby
code = params[:code]
```

<!--<a name="accesstoken"/>-->
## Demande de jeton d’accès à partir du point de terminaison du jeton

Une fois que vous avez le code d’autorisation, vous pouvez l’utiliser avec l’ID client, la clé et les valeurs de l’URL de réponse obtenues d’Azure AD pour demander un jeton d’accès. 

> **Remarque :** <br />
> La demande doit également spécifier une ressource que nous essayons de consommer. Dans le cas de Microsoft Graph, la valeur de la ressource est `https://graph.microsoft.com`.

Là encore, l’exemple de connexion délègue cette tâche à la bibliothèque OmniAuth. La fonction [`acquire_access_token`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L65) appelle la bibliothèque et transfère le code d’authentification enregistré dans la section précédente avec l’URL de réponse, l’ID client, le code secret client et l’ID de la ressource.

```ruby
def acquire_access_token(auth_code, reply_url)
    AUTH_CTX.acquire_token_with_authorization_code(
                  auth_code,
                  reply_url,
                  CLIENT_CRED,
                  GRAPH_RESOURCE)
end
```

> **Remarque :** <br />
> L’ID client et le code secret client sont fournis dans le paramètre `CLIENT_CRED` dans l’extrait de code précédent.

<!--<a name="request"/>-->
## Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Avec un jeton d’accès, votre application peut effectuer des demandes authentifiées à l’API Microsoft Graph. Votre application doit fournir le jeton d’accès dans l’en-tête **Authorization** de chaque demande.

L’exemple de connexion envoie un message électronique à l’aide du point de terminaison **sendMail** dans l’API Microsoft Graph. Le code se trouve dans la fonction [`send_mail`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L82). Il s’agit du code qui montre comment envoyer le code d’accès dans l’en-tête Authorization.

```ruby
def send_mail
  # Used in the template
  @name = session[:name]
  @email = params[:specified_email]
  @recipient = params[:specified_email]
  @mailSent = false
  
  sendMailEndpoint = URI("#{GRAPH_RESOURCE}#{SENDMAIL_ENDPOINT}")
  http = Net::HTTP.new(sendMailEndpoint.host, sendMailEndpoint.port)
  http.use_ssl = true
  
  emailBody = File.read("app/assets/MailTemplate.html")
  emailBody.sub! "{given_name}", @name
  
  emailMessage = "{
          Message: {
          Subject: 'Welcome to Office 365 development with Ruby',
          Body: {
              ContentType: 'HTML',
              Content: '#{emailBody}'
          },
          ToRecipients: [
              {
                  EmailAddress: {
                      Address: '#{@recipient}'
                  }
              }
          ]
          },
          SaveToSentItems: true
          }"

  response = http.post(
    SENDMAIL_ENDPOINT, 
    emailMessage, 
    initheader = 
    {
      "Authorization" => "Bearer #{session[:access_token]}", 
      "Content-Type" => CONTENT_TYPE
    }
  )

  # The send mail endpoint returns a 202 - Accepted code on success
  if response.code == "202"
    @mailSent = true
  else
    @mailSent = false
    flash[:httpError] = "#{response.code} - #{response.message}"
  end
  
  render "callback"
end
```

> **Remarque :** <br />
> La demande doit également envoyer un en-tête **Content-Type** avec une valeur acceptée par l’API Microsoft Graph, par exemple, `application/json;odata.metadata=minimal;odata.streaming=true`.

L’API Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la référence API pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph.

<!--## Additional resources

-  [Office 365 Ruby Connect sample using Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference](http://graph.microsoft.io/en-us/docs)-->
