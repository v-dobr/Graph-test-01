# Appel Microsoft Graph dans une application Python 

Dans cet article, nous examinons les tâches minimales requises pour connecter votre application à Office 365 et appeler l’API Microsoft Graph. Nous utilisons le code de l’[exemple de connexion d’une application Python à Office 365 à l’aide de Microsoft Graph](https://github.com/microsoftgraph/python3-connect-rest-sample) pour expliquer les concepts principaux que vous devez mettre en œuvre dans votre application.

![Capture d’écran d’un exemple de connexion d’une application Python à Office 365](./images/web-screenshot.png)

##  Conditions préalables

Cette rubrique suppose que :

* Vous êtes à l’aise avec la lecture du code Python.
* Vous connaissez les concepts OAuth.

## Vue d’ensemble

Pour appeler l’API Microsoft Graph, votre application Python doit effectuer les tâches suivantes.

1. Inscription de l’application dans Azure Active Directory
2. Redirection du navigateur vers la page de connexion
3. Réception d’un code d’autorisation dans la page de votre URL de réponse
4. Demande de jeton d’accès à partir du point de terminaison émettant le jeton
5. Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph 

<!--<a name="register"></a>-->
## Inscription de l’application dans Azure Active Directory

Avant de commencer à utiliser Office 365, vous devez inscrire votre application et définir des autorisations pour utiliser les services Microsoft Graph. En quelques clics, vous pouvez inscrire votre application pour accéder au compte professionnel ou scolaire d’un utilisateur à l’aide de l’[outil d’inscription des applications](https://dev.office.com/app-registration). 
Pour la gérer, vous devrez accéder au [Portail de gestion Microsoft Azure](https://manage.windowsazure.com)

Vous pouvez également consulter la section relative à l’[inscription de votre application de serveur web auprès du Portail de gestion Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) pour obtenir des instructions sur la façon d’inscrire manuellement l’application. Gardez à l’esprit les informations suivantes :

* Vérifiez que vous spécifiez http://127.0.0.1:8000/connect/get_token/ comme **URL d’authentification**.
* Une fois que vous avez inscrit l’application, [configurez les **autorisations déléguées**](https://github.com/microsoftgraph/python3-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) requises par votre application Python. L’exemple de connexion requiert l’autorisation d’**envoi de messages en tant qu’utilisateur connecté**.

Prenez note des valeurs suivantes dans la page de **configuration** de votre application Azure car ces valeurs sont requises pour configurer le flux OAuth dans votre application Python.

* ID client (unique pour votre application)
* URL de réponse (http://127.0.0.1:8000/connect/get_token/)
* Une clé d’application (unique pour votre application)

<!--<a name="redirect"></a>-->
## Redirection du navigateur vers la page de connexion

Votre application doit rediriger le navigateur vers la page de connexion pour démarrer le flux OAuth et obtenir un code d’autorisation. 

Dans l’exemple de connexion, le code suivant (situé dans [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) génère l’URL vers laquelle l’application redirige l’utilisateur et est transmis à la vue où il peut être utilisé pour la redirection. 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## Réception d’un code d’autorisation dans la page de votre URL de réponse

Une fois que l’utilisateur se connecte, le navigateur est redirigé vers votre URL de réponse, la fonction ```get_token``` dans [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), avec un code d’autorisation ajouté à la chaîne de requête comme variable ```code```. 

L’exemple de connexion obtient le code de la chaîne de requête, afin de pouvoir ensuite l’échanger contre un jeton d’accès.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## Demande de jeton d’accès à partir du point de terminaison émettant le jeton

Une fois que vous avez le code d’autorisation, vous pouvez l’utiliser avec l’ID client, la clé et les valeurs de l’URL de réponse obtenues d’Azure Active Directory pour demander un jeton d’accès. 

> **Remarque** La demande doit également spécifier une ressource que vous essayez d’utiliser. Dans le cas de Microsoft Graph, la valeur de la ressource est `https://graph.microsoft.com`.

L’exemple de connexion demande un jeton dans la fonction ```get_token_from_code``` du fichier [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py).

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **Remarque** La réponse fournit davantage d’informations que le jeton d’accès. Par exemple, votre application peut obtenir un jeton d’actualisation pour demander de nouveaux jetons d’accès sans que l’utilisateur se connecte de nouveau explicitement.

<!--<a name="request"></a>-->
## Utilisation du jeton d’accès dans une demande à l’API Microsoft Graph

Avec un jeton d’accès, votre application peut effectuer des demandes authentifiées à l’API Microsoft Graph. Votre application doit ajouter le jeton d’accès à l’en-tête **Authorization** de chaque demande.

L’exemple de connexion envoie un message électronique à l’aide du point de terminaison ```me/microsoft.graph.sendMail``` dans l’API Microsoft Graph. Le code se trouve dans la fonction ```call_sendMail_endpoint``` du fichier [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py). Il s’agit du code qui montre comment ajouter le code d’accès dans l’en-tête Authorization.

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Remarque** La demande doit également envoyer un en-tête **Content-Type** avec une valeur acceptée par l’API Graph, par exemple `application/json`.

L’API Microsoft Graph est une API d’unification très puissante, qui peut être utilisée pour interagir avec tous les types de données Microsoft. Consultez la référence API pour découvrir les autres tâches que vous pouvez exécuter avec l’API Microsoft Graph.

<!--
## Additional resources

-  [Office 365 Python Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Python-Microsoft-Graph-Connect)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference]()-->
