# Chamar o Microsoft Graph em um aplicativo PHP 

Neste artigo, analisamos as tarefas mínimas necessárias para obter um token de acesso do Azure AD (Active Directory) e chamar a API do Microsoft Graph. Usamos código do [exemplo do Office 365 Connect para PHP usando o Microsoft Graph](https://github.com/microsoftgraph/php-connect-rest-sample) para explicar os principais conceitos que você precisa implementar em seu aplicativo.

![Captura de tela do exemplo de conexão com o Office 365 para PHP](./images/web-screenshot.png)

## Visão geral

Para chamar a API do Microsoft Graph, seu aplicativo PHP deve concluir as seguintes tarefas:

1. Registrar o aplicativo no Azure Active Directory
2. Redirecionar o navegador para a página de entrada
3. Receber um código de autorização em sua página de URL de resposta
4. Solicitar um token de acesso do ponto de extremidade do token
5. Usar o token de acesso em uma solicitação para a API do Microsoft Graph

<!--<a name="register"/>-->
## Registrar o aplicativo no Azure Active Directory

Antes de começar a trabalhar com o Office 365, você precisa registrar seu aplicativo e definir permissões para usar os serviços do Microsoft Graph. 
Com apenas alguns cliques, você pode registrar seu aplicativo para acessar uma conta corporativa ou de estudante de um usuário usando a [Ferramenta de Registro de Aplicativo](https://dev.office.com/app-registration). Para gerenciá-la, você precisará acessar o [portal de Gerenciamento do Microsoft Azure](https://manage.windowsazure.com)

Como alternativa, confira o artigo [Registrar seu aplicativo de servidor Web no Portal de Gerenciamento do Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) para obter instruções sobre como registrar manualmente o aplicativo. Lembre-se dos seguintes detalhes:

* Especifique uma página em seu aplicativo PHP como a **URL de Entrada** na etapa 6. No caso do exemplo do Connect, a página é [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php).
* [Configure as **Permissões delegadas**](https://github.com/microsoftgraph/php-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) que o aplicativo exige. O exemplo do Connect exige uma permissão **Enviar emails como usuário conectado**.

Anote os valores a seguir na página **Configurar** do seu aplicativo Azure.

* ID do cliente
* Uma chave válida
* Uma URL de resposta

Você precisa desses valores como parâmetros no fluxo OAuth no aplicativo.

<!--<a name="redirect"/>-->
## Redirecionar o navegador para a página de entrada

O aplicativo precisa redirecionar o navegador para a página de entrada a fim de obter um código de autorização e continuar o fluxo do OAuth.

No exemplo do Connect, o código que redireciona o navegador está na função [`AuthenticationManager.connect`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L41).

```php
// Redirect the browser to the authorization endpoint. Auth endpoint is
// https://login.microsoftonline.com/common/oauth2/authorize
$redirect = Constants::AUTHORITY_URL . Constants::AUTHORIZE_ENDPOINT . 
            '?response_type=code' . 
            '&client_id=' . urlencode(Constants::CLIENT_ID) . 
            '&redirect_uri=' . urlencode(Constants::REDIRECT_URI);
header("Location: {$redirect}");
exit();
```

> **Observação:** <br />
> Você deve enviar o cabeçalho **Location** antes de escrever qualquer saída para a página.

<!--<a name="authcode"/>-->
## Receber um código de autorização em sua página de URL de resposta

Depois que o usuário entrar, o fluxo fará o navegador retornar para a URL de resposta em seu aplicativo. O Azure anexa um código de autorização à cadeia de caracteres de consulta. O exemplo do Connect usa a página [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php) para essa finalidade.

O código de autorização é fornecido na variável `code` da cadeia de consulta. O exemplo do Connect salva o código em uma variável de sessão para usá-lo mais tarde.

```php
if (isset($_GET['code'])) {
    $_SESSION['code'] =  $_GET['code'];
}
```

<!--<a name="accesstoken"/>-->
## Solicitar um token de acesso do ponto de extremidade do token

Quando tiver o código de autorização, você poderá usá-lo com os valores ID do cliente, chave e URL de resposta obtidos do Azure AD para solicitar um token de acesso. 

> **Observação:** <br />
> A solicitação também tem que especificar um recurso que estamos tentando consumir. No caso da API do Microsoft Graph, o valor do recurso é `https://graph.microsoft.com`.

O exemplo do Connect requer um token usando o código na função [`AuthenticationManager.acquireToken`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L62). Aqui está o código mais relevante.

```php
$tokenEndpoint = Constants::AUTHORITY_URL . Constants::TOKEN_ENDPOINT;

// Send a POST request to the token endpoint to retrieve tokens.
// Token endpoint is:
// https://login.microsoftonline.com/common/oauth2/token
$response = RequestManager::sendPostRequest(
    $tokenEndpoint, 
    array(),
    array(
        'client_id' => Constants::CLIENT_ID,
        'client_secret' => Constants::CLIENT_SECRET,
        'code' => $_SESSION['code'],
        'grant_type' => 'authorization_code',
        'redirect_uri' => Constants::REDIRECT_URI,
        'resource' => Constants::RESOURCE_ID
    )

// Store the raw response in JSON format.
$jsonResponse = json_decode($response, true);

// The access token response has the following parameters:
// access_token - The requested access token.
// expires_in - How long the access token is valid.
// expires_on - The time when the access token expires.
// id_token - An unsigned JSON Web Token (JWT).
// refresh_token - An OAuth 2.0 refresh token.
// resource - The App ID URI of the web API (secured resource).
// scope - Impersonation permissions granted to the client application.
// token_type - Indicates the token type value.
foreach ($jsonResponse as $key=>$value) {
    $_SESSION[$key] = $value;
}
```

> **Observação:** <br />
> A resposta fornece mais informações do que apenas o token de acesso; por exemplo, seu aplicativo poderá obter um token de atualização para solicitar novos tokens de acesso sem fazer com que o usuário tenha que entrar.

Seu aplicativo PHP agora pode usar a variável de sessão `access_token` para emitir solicitações autenticadas à API do Microsoft Graph.

<!--<a name="request"/>-->
## Usar o token de acesso em uma solicitação para a API do Microsoft Graph

Com um token de acesso, o aplicativo pode fazer solicitações autenticadas à API do Microsoft Graph. O aplicativo deve fornecer o token de acesso no cabeçalho **Authorization** de cada solicitação.

O exemplo do Connect envia um email usando o ponto de extremidade **sendMail** na API do Microsoft Graph. O código está na função [`MailManager.sendWelcomeMail`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/MailManager.php#L40). Esse é o código que mostra como enviar o código de acesso no cabeçalho Authorization.

```php
// Send the email request to the sendmail endpoint, 
// which is in the following URI:
// https://graph.microsoft.com/v1.0/me/microsoft.graph.sendMail
// Note that the access token is attached in the Authorization header
RequestManager::sendPostRequest(
    Constants::RESOURCE_ID . Constants::SENDMAIL_ENDPOINT,
    array(
        'Authorization: Bearer ' . $_SESSION['access_token'],
        'Content-Type: application/json;' . 
                      'odata.metadata=minimal;' .
                      'odata.streaming=true'
    ),
    $email
);
```

> **Observação:** <br />
> A solicitação também deve enviar um cabeçalho **Content-Type** com um valor aceito pela API do Microsoft Graph, por exemplo, `application/json;odata.metadata=minimal;odata.streaming=true`.

A API do Microsoft Graph é uma API unificadora muito poderosa que pode ser usada para interagir com todos os tipos de dados da Microsoft. Confira a referência de API para explorar o que mais você pode fazer com a API do Microsoft Graph.

<!--## Additional resources

-  [Office 365 PHP Connect sample using Microsoft Graph API](https://github.com/OfficeDev/O365-PHP-Unified-API-Connect)-->
