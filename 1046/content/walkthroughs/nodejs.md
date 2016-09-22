# Chamar o Microsoft Graph com um aplicativo Node.js

Neste artigo, analisamos as tarefas mínimas necessárias para conectar seu aplicativo ao Office 365 e chamar a API do Microsoft Graph. Usamos código do [exemplo do Office 365 Node.js Connect usando o Microsoft Graph](https://github.com/microsoftgraph/nodejs-connect-rest-sample) para explicar os principais conceitos que você precisa implementar no seu aplicativo.

![Captura de tela do exemplo de conexão com o Office 365 para Node.js](./images/web-screenshot.png)

## Visão geral

Para chamar a API do Microsoft Graph, seu aplicativo Web deve concluir as seguintes tarefas:

1. Registrar o aplicativo no Azure Active Directory 
2. Instalar a biblioteca de cliente do Azure Active Directory para Node
3. Redirecionar o navegador para a página de entrada
4. Receber um código de autorização em sua página de URL de resposta
5. Usar `adal-node` para solicitar um token de acesso
6. Fazer uma solicitação à API do Microsoft Graph

<!--<a name="register"/>-->
## Registrar o aplicativo no Azure Active Directory

Antes de começar a trabalhar com o Office 365, você precisa registrar seu aplicativo e definir permissões para usar os serviços do Microsoft Graph. Com apenas alguns cliques, você pode registrar seu aplicativo para acessar uma conta corporativa ou de estudante de um usuário usando a [Ferramenta de Registro de Aplicativo](https://dev.office.com/app-registration). 
Para gerenciá-la, você precisará acessar o [portal de Gerenciamento do Microsoft Azure](https://manage.windowsazure.com)

Como alternativa, confira o artigo [Registrar seu aplicativo de servidor Web no Portal de Gerenciamento do Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) para obter instruções sobre como registrar manualmente o aplicativo. Lembre-se dos seguintes detalhes:

* Especifique uma página em seu aplicativo Node.js como a **URL de Entrada** na etapa 6. No caso do exemplo do Connect, a URL é http://localhost:8080/login, que é mapeada para a rota [/login](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L33).
* [Configure as **Permissões delegadas**](https://github.com/microsoftgraph/nodejs-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) que seu aplicativo exige. O exemplo do Connect exige uma permissão **Enviar emails como usuário conectado**.

Anote os valores a seguir na página **Configurar** do seu aplicativo Azure.

* ID do cliente
* Uma chave válida
* Uma URL de resposta

Você precisa desses valores como parâmetros no fluxo OAuth no aplicativo.

<!--<a name="adal">-->
## Instalar a biblioteca de cliente do Azure Active Directory para Node

A ADAL para Node.js facilita a autenticação dos aplicativos Node.js no AAD a fim de acessar recursos Web protegidos do AAD.
Para adicionar adal-node ao seu `package.json` existente, insira o que vem a seguir no seu terminal preferencial.

`npm install adal-node --save`

Para saber mais sobre a biblioteca de cliente adal-node, confira as informações do pacote em [npm](https://www.npmjs.com/package/adal-node). 
Para problemas, código-fonte e as notícias mais recentes sobre recursos e correções, confira o projeto adal-node no [Github](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs).

<!--<a name="redirect"/>-->
## Redirecionar o navegador para a página de entrada

O aplicativo precisa redirecionar o navegador para a página de entrada a fim de obter um código de autorização e continuar o fluxo do OAuth 2.0.

No exemplo do Connect, a URL de autenticação de [`authHelper.js#getAuthUrl`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js#L17) redirecionada pela função [`login.hbs#login`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/views/login.hbs#L2) por um evento do lado do cliente `onclick`.

**authHelper.js#getAuthUrl**
```javascript
/**
 * Generate a fully formed uri to use for authentication based on the supplied resource argument
 * @return {string} a fully formed uri with which authentcation can be completed
 */
function getAuthUrl() {
    return credentials.authority + "/oauth2/authorize" +
        "?client_id=" + credentials.client_id +
        "&response_type=code" +
        "&redirect_uri=" + credentials.redirect_uri;
};
```

**login.hbs#login**
```javascript
function login() {
    window.location = '{{auth_url}}'.replace(/&amp;/g, '&'); // transform HTML special char from .hbs template rendering
}
```

<!--<a name="authcode"/>-->
## Receber um código de autorização em sua página de URL de resposta

Depois que o usuário entrar, o fluxo fará o navegador retornar para a URL de resposta em seu aplicativo. O código de autorização é fornecido na variável `code` da cadeia de consulta.

```javascript
router.get('/<application reply url>', function (req, res, next) {
  var authCode = req.query.code;
  // your router's implementation
});
```

Confira o [código relevante](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L34) no exemplo do Connect

<!--<a name="accesstoken"/>-->
## Usar `adal-node` para solicitar um token de acesso

Agora que autenticamos com o Azure Active Directory, nossa próxima etapa é adquirir um token de acesso via adal-node. Depois disso, estaremos prontos para fazer solicitações REST à API do Microsoft Graph.

Para solicitar um token de acesso, o adal-node fornece duas funções de retorno de chamada.

|                          Função                         |                                      Parâmetros                                      | Descrição                                                                                             |
|:---------------------------------------------------------:|:--------------------------------------------------------------------------------:|---------------------------------------------------------------------------------------------------------|
| `AuthenticationContext.acquireTokenWithAuthorizationCode` | `authCode`, `redirect_uri`, `resource`, `client_id`, `client_secret`, `callback` | Fornece um token de acesso para um recurso especificado baseado no código de autorização retornado durante o logon |
| `AuthenticationContext.acquireTokenWithRefreshToken`      | `token`, `client_id`, `client_secret`, `resource`, `callback`                    | Fornece um token de acesso para um recurso especificado baseado em um token de atualização                             |

Na amostra do Connect, as solicitações são roteadas por [`authHelper.js`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js) para que as `client_id` e `client_secret` possam ser adicionadas.

```javascript
// The application registration (must match Azure AD config)
var credentials = {
    authority: "https://login.microsoftonline.com/common",
    client_id: "<your client id here>",
    client_secret: "<your client secret>",
    redirect_uri: "http://localhost:8080/login"
};

/**
 * Gets a token for a given resource.
 * @param {string} code An authorization code returned from a client.
 * @param {string} res A URI that identifies the resource for which the token is valid.
 * @param {AcquireTokenCallback} callback The callback function.
 */
function getTokenFromCode(res, code, callback) {
    var authContext = new AuthenticationContext(credentials.authority);
    authContext.acquireTokenWithAuthorizationCode(code, credentials.redirect_uri, res, credentials.client_id, credentials.client_secret, function (err, response) {
        if (err) {
            callback(null);
        }
        else {
            callback(response);
        }
    });
};
```

<!--<a name="request"/>-->
## Fazer uma solicitação à API do Microsoft Graph

Para identificar nossas solicitações à Graph API, elas devem ser assinadas com um cabeçalho `Authorization` contendo o token de acesso para qualquer recurso de serviço Web solicitado. Um cabeçalho de autorização corretamente formado incluirá o token de acesso do adal-node e terá o formato a seguir.

`Authorization: Bearer <access token>`

O uso do `adal-node` combinado com nossa lógica de autenticação da seção anterior permite o uso do nosso token de acesso para assinar solicitações.

```javascript
/* GET home page. */
router.get('/<application reply url>', function (req, res, next) {
    var authCode = req.query.code;
    authHelper.getTokenFromCode('https://graph.microsoft.com/', req.query.code, function (token) {
        if (token !== null) {
            // Use this token to sign requests
            var headers = {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer ' + token
                };
            // request implementation...
        } else {
            // error handling
        }
    });
});
```

O Microsoft Graph é uma API unificadora muito poderosa que pode ser usada para interagir com todos os tipos de dados da Microsoft. Confira a [referência de API](http://graph.microsoft.io/docs/api-reference/v1.0) para explorar o que mais você pode fazer com a API do Microsoft Graph.

<!--## Additional resources

- [Office 365 Node.js Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Nodejs-Unified-API-Connect)-->

