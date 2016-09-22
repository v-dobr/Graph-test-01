# Introdução ao Microsoft Graph e REST

Este artigo descreve como chamar o Microsoft Graph para recuperar mensagens de email no Office 365 e no Outlook.com. Este artigo tem foco nas respostas e solicitações do OAuth e REST. Ele aborda a sequência de solicitações e respostas que um aplicativo usa para autenticar e recuperar as mensagens.

## Usando OAuth 2.0 para autenticar

Para chamar o Microsoft Graph, seu aplicativo precisa de um token de acesso do AD do Azure (Active Directory do Azure). No exemplo a seguir, o aplicativo implementa o fluxo de Concessão de Código de Autorização para obter os tokens de acesso do Azure AD, seguindo os [protocolos OAuth 2.0](http://tools.ietf.org/html/rfc6749) padrão.

### Registrando um aplicativo

Há duas opções para registrar o aplicativo:

  1. Registrar um aplicativo usando o modelo que dá suporte a usuários comerciais do Office 365; apenas para contas corporativas ou de estudante.
 
  Esse modelo só funciona com ofertas comerciais do Office 365. Após registrar o aplicativo, você poderá gerenciá-lo no [Portal de Gerenciamento do Azure](https://manage.windowsazure.com).

  2. Registre usando a funcionalidade mais recente que funciona para serviços de consumidor e comerciais do Office 365 (chamamos isso de ponto de extremidade Auth v 2.0).
 
  Um serviço de autenticação único para contas corporativas e de estudante está disponível agora. Este modelo fornece um serviço de autenticação único para identidades corporativas e de estudante (AD do Azure), além de identidades pessoais (Microsoft). Agora, você só precisa implementar um fluxo de autenticação em seu aplicativo para habilitar os usuários a usar contas corporativas ou de estudante, como as do Office 365 ou do OneDrive for Business, ou contas pessoais, como as do Outlook.com ou do OneDrive.
   
Use o [Portal de Registro de Aplicativo](https://apps.dev.microsoft.com/) para registrar o aplicativo e dar suporte a contas corporativas e de estudante.

Observe que o ponto de extremidade v2.0 está crescendo gradualmente para abranger todos os cenários do ponto de extremidade de autenticação anterior. Para decidir se é a opção certa para você, leia [este artigo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

Após o registro, você obterá uma ID de cliente e um segredo. Esses valores são usados no fluxo de concessão de código de autorização.

O restante deste documento pressupõe um registro no modelo  v2.0l. Para obter um guia completo dos fluxos com suporte no ponto de extremidade v2.0, confira [este artigo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/). Para obter um guia completo do fluxo de Concessão de Código de Autorização, confira [este artigo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/)

### Obtendo um código de autorização

A primeira etapa no fluxo de concessão de código de autorização é obter um código de autorização. Esse código é retornado ao aplicativo pelo servidor autorização quando o usuário faz logon e permite o nível de acesso que o aplicativo solicitou.

Primeiro, o aplicativo cria uma URL de entrada para o usuário. Essa URL deve ser aberta em um navegador para que o usuário possa entrar e dar permissão.

A URL base para logon é semelhante a `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`.

O aplicativo acrescenta parâmetros da consulta a essa URL base para informar ao servidor de autenticação qual aplicativo está solicitando o logon e quais permissões está solicitando.

- `client_id` - a ID do cliente gerada pelo registro do aplicativo. Isso permite que o AD do Azure saiba qual aplicativo está solicitando o logon.
- `redirect_uri` - o local para o qual o Azure será redirecionado quando o usuário conceder permissão para o aplicativo. Esse valor deve corresponder ao valor **URI de redirecionamento** usado ao registrar o aplicativo.
- `response_type` - o tipo de resposta que o aplicativo está esperando. Esse valor é `code` para o fluxo de Concessão de Código de Autorização.
- `scope` - uma lista de escopos separados por espaços que o aplicativo está solicitando. Há mais informações [neste artigo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/)
- `state` - um valor incluído na solicitação que também será retornado na resposta de token.

Por exemplo, a URL de solicitação para nosso aplicativo que requer acesso de leitura para os emails teria a seguinte aparência:

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<CLIENT ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
```

Em seguida, redirecione o usuário para a URL de logon. O usuário verá uma tela de entrada exibindo o nome do aplicativo. Depois de entrar, o usuário verá uma lista das permissões que o aplicativo exige e será solicitado a permiti-las ou negá-las. Pressupondo que eles permitam o acesso necessário, o navegador será redirecionado para o URI de redirecionamento especificado na solicitação inicial com o código de autorização na cadeia de caracteres de consulta.

```http
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

Se você também estiver usando OpenId Connect para Logon Único, serão necessários parâmetros adicionais. Confira [este artigo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/) para saber mais. 

A próxima etapa é trocar o código de autorização retornado por um token de acesso

### Obtendo um token de acesso

Para obter um token de acesso, o aplicativo posta parâmetros codificados em formulário na URL de solicitação de token (`https://login.microsoftonline.com/common/oauth2/v2.0/token`) com os parâmetros a seguir.

- `client_id`: A ID do cliente gerada durante o registro do aplicativo.
- `client_secret`: O segredo de cliente gerado no registro do aplicativo.
- `code`: O código de autorização obtido na etapa anterior.
- `redirect_uri`: Esse valor deve ser igual ao valor usado na solicitação de código de autorização.
- `grant_type`: O tipo de concessão que o aplicativo está usando. Esse valor é `code` para o fluxo de Concessão de Código de Autorização.
- `scope` - uma lista de escopos separados por espaços que o aplicativo está solicitando. Há mais informações [neste artigo](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/)

A URL de solicitação de nosso aplicativo, usando o código da etapa anterior, teria a seguinte aparência:

```http
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
  &client\_id=<CLIENT ID>
  &client\_secret=<CLIENT SECRET>
}
```

O servidor responde com uma carga JSON que inclui o token de acesso.

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

O token de acesso é encontrado no campo `access_token` da carga JSON. O aplicativo usa esse valor para definir o cabeçalho Authorization ao fazer chamadas REST à API.

## Chamando o Microsoft Graph

Depois que o aplicativo tem um token de acesso, ele está pronto para chamar o Microsoft Graph. Como esse aplicativo de exemplo está recuperando mensagens, ele usará uma solicitação HTTP GET para o ponto de extremidade `https://graph.microsoft.com/v1.0/me/messages`.

### Refinamento da solicitação

Os aplicativos podem controlar o comportamento de solicitações GET, usando parâmetros de consulta OData. Recomendamos que os aplicativos usem esses parâmetros para limitar o número de resultados retornados e limitar os campos retornados para cada item. 

Este aplicativo de exemplo exibirá mensagens em uma tabela que mostra o assunto, o remetente e a data e hora em que a mensagem foi recebida. A tabela exibe um máximo de 25 linhas e está classificada de modo que a mensagem recebida mais recentemente esteja na parte superior. O aplicativo usa os parâmetros de consulta a seguir para obter esses resultados.

- `$select` - Especifica apenas os campos `subject`, `sender` e `dateTimeReceived`.
- `$top` - Especifica um máximo de 25 itens.
- `$orderby` - Classifica os resultados pelo campo `dateTimeReceived`.

Isso resulta na solicitação a seguir.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

Agora que você viu como fazer chamadas ao Microsoft Graph, pode usar a referência de API para construir outros tipos de chamada que seu aplicativo tenha que fazer. No entanto, tenha em mente que seu aplicativo precisa ter as permissões apropriadas configuradas no registro de aplicativo para as chamadas feitas.


