# Introdução ao Microsoft Graph em um aplicativo PHP

Este artigo descreve as tarefas necessárias para obter um token de acesso do ponto de extremidade de autenticação v2.0 e chamar o Microsoft Graph. Ele orientará você em relação a como criar um [Exemplo do Connect para PHP](https://github.com/microsoftgraph/php-connect-rest-sample) e explica os principais conceitos que você implementa para utilizar o Microsoft Graph. O artigo também descreve como acessar o Microsoft Graph usando chamadas REST.

Para usar o Microsoft Graph em seu aplicativo PHP, é preciso mostrar a página de entrada da Microsoft aos seus usuários. A captura de tela a seguir mostra uma página de entrada para contas da Microsoft.

![Página de entrada das contas da Microsoft](images/MicrosoftSignIn.png)

**Não está com vontade de criar um aplicativo?** Comece rapidamente, baixando o [Exemplo do Connect para PHP](https://github.com/microsoftgraph/php-connect-rest-sample) no qual este artigo se baseia.


## Pré-requisitos

Para começar, será necessário: 

- Uma [conta da Microsoft](https://www.outlook.com/) ou uma [conta do Office 365 para empresas](http://dev.office.com/devprogram)
- PHP versão 5.5.9 ou superior
- [Criador](https://getcomposer.org/)


## Registrar o aplicativo
Registre um aplicativo no Portal de Registro de Aplicativos da Microsoft. Isso gera a ID do aplicativo e a senha que você usará para configurar o aplicativo.

1. Entre no [Portal de Registro de Aplicativos da Microsoft](https://apps.dev.microsoft.com/) usando sua conta pessoal ou uma conta corporativa ou de estudante.

2. Escolha **Adicionar um aplicativo**.

3. Insira um nome para o aplicativo e escolha **Criar aplicativo**. 
    
   A página de registro é exibida, listando as propriedades do seu aplicativo.

4. Escolha **Gerar Nova Senha**.

5. Copie a ID e a senha do aplicativo.

6. Escolha **Adicionar Plataforma** e **Web**.

7. No campo **URI de Redirecionamento**, digite `http://localhost:8000/oauth`.

8. Escolha **Salvar**.


## Configurar o projeto

Inicie um novo projeto usando o criador. Para criar um novo projeto PHP, usando a estrutura do Laravel, digite o seguinte comando:

```bash
composer create-project --prefer-dist laravel/laravel getstarted
```
 
Isso criará uma pasta **getstarted** que podemos usar para este projeto.

## Autenticar o usuário e obter um token de acesso
Vamos usar uma biblioteca do OAuth para simplificar o processo de autenticação. [O PHP League](http://thephpleague.com/) fornece uma [biblioteca cliente do OAuth](https://github.com/thephpleague/oauth2-client) que podemos usar neste projeto.

### Adicionar a dependência ao criador

Abra o arquivo `composer.json` e inclua a seguinte dependência na seção **requer**:

```json
"league/oauth2-client": "^1.4"
```

Atualize as dependências executando o seguinte comando:

```bash
composer update
```

### Iniciar o fluxo de autenticação

1. Abra o arquivo **resources** > **views** > **welcome.blade.php**. Substitua o elemento div **título** pelo seguinte código.
    ```html
    <div class="title" onClick="window.location='/oauth'">Sign in to Microsoft</div>
    ```
    
2. Digite a classe `Illuminate\Http\Request` no arquivo **app** > **Http** > **routes.php**. Adicione a seguinte linha antes de qualquer declaração de rota.
    ```php
    use Illuminate\Http\Request;
    ```
    
3. Adicione uma rota */oauth* ao arquivo **app** > **Http** > **routes.php**. Para adicionar a rota, copie o código a seguir após a declaração da rota padrão. Insira o **ID do aplicativo** e a **senha** do seu aplicativo nos espaços reservados marcados com **\<ID_DO_SEU_APLICATIVO\>** e **\<SUA_SENHA\>** respectivamente.
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
    
Agora, você deve ter um aplicativo PHP que exibe *Entrar na Microsoft*. Se você clicar no texto, o aplicativo apresentará a página de entrada da Microsoft. O próximo passo é manipular o código que o servidor de autorização envia para o URI de redirecionamento e trocá-lo por um token de acesso.

### Trocar o código de autorização para solicitar um token de acesso

Precisamos lidar com a resposta do servidor de autorização, que contém um código que podemos substituir por um token de acesso.

Atualize a rota */oauth* para que ela tenha um token de acesso com o código de autorização. Para fazer isso, abra o arquivo **app** > **Http** > **routes.php** e adicione a seguinte cláusula condicional *else* à instrução *if*.

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
    
Observe que temos um token de acesso nesta linha `exit($accessToken->getToken());`. Agora, você está pronto para adicionar código para chamar o Microsoft Graph. 

## Chamar o Microsoft Graph usando o REST
Podemos chamar o Microsoft Graph usando o REST. A [API REST do Microsoft Graph](http://graph.microsoft.io/docs) expõe várias APIs dos serviços de nuvem da Microsoft por meio de um único ponto de extremidade da API REST. Para usar a API REST, substitua a linha `exit($accessToken->getToken());` pelo seguinte código. Insira seu endereço de email no espaço reservado marcado com **\<SEU_ENDEREÇO_DE_EMAIL\>**.

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

## Executar o aplicativo
Você está pronto para experimentar seu aplicativo PHP.

1. No shell, digite o seguinte comando:
    ```bash
    php artisan serve
    ```
    
2. Acesse `http://localhost:8000` no navegador da Web.
3. Escolha **Entrar na Microsoft**.
4. Entre com sua conta pessoal, comercial ou de estudante, e conceda as permissões solicitadas.

Verifique a caixa de entrada do endereço de email que você configurou na seção [Chamar o Microsoft Graph usando o REST](#call-the-microsoft-graph-using-rest). É provável que você tenha um email da conta que utilizou para entrar no aplicativo.

## Próximas etapas
- Experimente a API REST, usando o [Explorador do Graph](https://graph.microsoft.io/graph-explorer).


## Ver também
* [Visão geral do Microsoft Graph](http://graph.microsoft.io/docs)
* [Protocolos do Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Tokens do Azure AD v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
