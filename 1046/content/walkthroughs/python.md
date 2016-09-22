# Chamar o Microsoft Graph em um aplicativo Python 

Neste artigo, analisamos as tarefas mínimas necessárias para conectar seu aplicativo ao Office 365 e chamar a API do Microsoft Graph. Usamos código do [exemplo do Office 365 Python Connect usando o Microsoft Graph](https://github.com/microsoftgraph/python3-connect-rest-sample) para explicar os principais conceitos que você precisa implementar no seu aplicativo.

![Captura de tela do exemplo do Office 365 Connect para Python](./images/web-screenshot.png)

##  Pré-requisitos

Este tópico pressupõe o seguinte:

* Você lê código em Python com facilidade.
* Você está familiarizado com conceitos de OAuth.

## Visão geral

Para chamar a API do Microsoft Graph, seu aplicativo Python deve concluir as seguintes tarefas:

1. Registrar o aplicativo no Azure Active Directory
2. Redirecionar o navegador para a página de entrada
3. Receber um código de autorização em sua página de URL de resposta
4. Solicitar um token de acesso do ponto de extremidade de emissão do token
5. Usar o token de acesso em uma solicitação para a API do Microsoft Graph 

<!--<a name="register"></a>-->
## Registrar o aplicativo no Azure Active Directory

Antes de começar a trabalhar com o Office 365, você precisa registrar seu aplicativo e definir permissões para usar os serviços do Microsoft Graph. 
Com apenas alguns cliques, você pode registrar seu aplicativo para acessar uma conta corporativa ou de estudante de um usuário usando a [Ferramenta de Registro de Aplicativo](https://dev.office.com/app-registration). Para gerenciá-la, você precisará acessar o [portal de Gerenciamento do Microsoft Azure](https://manage.windowsazure.com)

Como alternativa, confira o artigo [Registrar seu aplicativo de servidor Web no Portal de Gerenciamento do Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) para obter instruções sobre como registrar manualmente o aplicativo. Lembre-se dos seguintes detalhes:

* Não deixe de especificar http://127.0.0.1:8000/connect/get_token/ como a **URL de Logon**.
* Depois de registrar o aplicativo, [configure as **Permissões delegadas**](https://github.com/microsoftgraph/python3-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) que seu aplicativo Python exige. O exemplo do Connect exige a permissão **Enviar emails como usuário conectado**.

Anote os seguintes valores na página **Configurar** do seu aplicativo do Azure porque você precisará deles para configurar o fluxo do OAuth em seu aplicativo Python.

* ID do cliente (exclusiva para seu aplicativo)
* Uma URL de resposta (http://127.0.0.1:8000/connect/get_token/)
* Uma chave do aplicativo (exclusiva para seu aplicativo)

<!--<a name="redirect"></a>-->
## Redirecionar o navegador para a página de entrada

O aplicativo precisa redirecionar o navegador para a página de entrada a fim de iniciar o fluxo OAuth e obter um código de autorização. 

No exemplo do Connect, o código a seguir (localizado em [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) cria a URL que o aplicativo precisa para redirecionar o usuário e é direcionado ao modo de exibição em ele pode ser usado para o redirecionamento. 

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
## Receber um código de autorização em sua página de URL de resposta

Depois que o usuário entra, o navegador é redirecionado para a URL de resposta, a função ```get_token``` em [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), com um código de autorização anexado à cadeia de caracteres de consulta como a variável ```code```. 

O exemplo do Connect obtém o código da cadeia de caracteres de consulta para poder trocá-lo por um token de acesso.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## Solicitar um token de acesso do ponto de extremidade de emissão do token

Quando tiver o código de autorização, você poderá usá-lo com os valores ID do cliente, chave e URL de resposta obtidos do Azure Active Directory para solicitar um token de acesso. 

> **Observação** A solicitação também tem que especificar um recurso que você esteja tentando consumir. No caso do Microsoft Graph, o valor do recurso é `https://graph.microsoft.com`.

O exemplo do Connect solicita um token na função ```get_token_from_code``` no arquivo [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py).

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

> **Observação** A resposta fornece mais informações além do token de acesso. Por exemplo, o aplicativo pode obter um token de atualização para solicitar novos tokens de acesso sem ter que conectar o usuário novamente.

<!--<a name="request"></a>-->
## Usar o token de acesso em uma solicitação para a API do Microsoft Graph

Com um token de acesso, o aplicativo pode fazer solicitações autenticadas à API do Microsoft Graph. Seu aplicativo deve anexar o token de acesso ao cabeçalho **Authorization** de cada solicitação.

O exemplo do Connect envia um email usando o ponto de extremidade ```me/microsoft.graph.sendMail``` na API do Microsoft Graph. O código está na função ```call_sendMail_endpoint``` no arquivo [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py). Esse é o código que mostra como anexar o código de acesso ao cabeçalho Authorization.

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Observação** A solicitação também deve enviar um cabeçalho **Content-Type** com um valor aceito pela API do Microsoft Graph, por exemplo, `application/json`.

A API do Microsoft Graph é uma API unificadora muito poderosa que pode ser usada para interagir com todos os tipos de dados da Microsoft. Confira a referência de API para explorar o que mais você pode fazer com a API do Microsoft Graph.

<!--
## Additional resources

-  [Office 365 Python Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Python-Microsoft-Graph-Connect)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference]()-->
