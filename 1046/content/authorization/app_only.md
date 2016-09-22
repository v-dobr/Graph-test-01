# Chamar o Microsoft Graph em um aplicativo de serviço ou daemon

Neste artigo, analisamos as tarefas mínimas necessárias para conectar seu aplicativo de serviço ou daemon de locatário único ao Office 365 e chamar a API do Microsoft Graph.

## Visão geral

Para chamar a API do Microsoft Graph em um aplicativo de serviço ou daemon, você deve concluir as tarefas a seguir.

1. Registrar o aplicativo no Active Directory do Azure.
2. Solicite um token de acesso do ponto de extremidade de emissão do token.
3. Usar o token de acesso em uma solicitação para a API do Microsoft Graph.

## Registrar o aplicativo no Azure Active Directory

Antes de começar a trabalhar com o Office 365, você precisa registrar o aplicativo e definir permissões para usar os serviços do Microsoft Graph. 
Com apenas alguns cliques, você pode registrar o aplicativo usando a [Ferramenta de Registro de Aplicativo](https://dev.office.com/app-registration). Você precisará ir para o [portal de Gerenciamento do Microsoft Azure](https://manage.windowsazure.com) para gerenciá-lo.

Como alternativa, confira o artigo [Registrar seu aplicativo de servidor Web no Portal de Gerenciamento do Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) para obter instruções sobre como registrar manualmente o aplicativo. Lembre-se dos seguintes detalhes:

* Depois de registrar o aplicativo, configure as **Permissões de aplicativo** exigidas pelo seu aplicativo de serviço ou daemon.

Anote os seguintes valores na página Configurar do seu aplicativo do Azure porque você precisará deles para configurar o fluxo do OAuth em seu aplicativo de serviço ou daemon.

* ID do cliente (exclusiva para seu aplicativo)
* Uma chave do aplicativo (exclusiva para seu aplicativo)
* O ponto de extremidade de token OAuth 2.0 de seu aplicativo
  * Encontre esse valor clicando em *Exibir Pontos de Extremidade* na parte inferior do Portal de Gerenciamento do Azure, na página do seu aplicativo. O ponto de extremidade terá a seguinte aparência: `https://login.microsoftonline.com/<tenantId>/oauth2/token`.

## Solicitar um token de acesso do ponto de extremidade de emissão do token

Diferentemente dos aplicativos clientes, o aplicativo de serviço ou daemon é incapaz de fazer um usuário entrar e autorizar seu aplicativo. Em vez disso, seu aplicativo tem que implementar o fluxo de concessão de credenciais de cliente OAuth 2.0, que permite que ele use suas próprias credenciais, sua ID do cliente e uma chave do aplicativo para se autenticar ao chamar o Microsoft Graph, em vez de representar um usuário. Confira [Chamadas serviço a serviço usando as credenciais do cliente](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx) para obter detalhes sobre o fluxo de autenticação.

Faça uma solicitação HTTP POST ao ponto de extremidade de emissão do token com os seguintes parâmetros, substituindo `<clientId>` e `<clientSecret>` pela ID do cliente e pela chave do aplicativo, respectivamente.

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

A resposta incluirá um token de acesso e informações sobre a expiração.

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## Usar o token de acesso em uma solicitação para a API do Microsoft Graph

Com um token de acesso, o aplicativo pode fazer solicitações autenticadas à API do Microsoft Graph. Seu aplicativo deve anexar o token de acesso ao cabeçalho **Authorization** de cada solicitação.

Por exemplo, um aplicativo de serviço ou daemon poderá recuperar todos os usuários de um locatário se tiver a permissão *Ler perfis completos de todos os usuários* selecionada no Portal de Gerenciamento do Azure. 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```

O Microsoft Graph é uma API unificadora muito poderosa que pode ser usada para interagir com todos os tipos de dados da Microsoft. Confira a [referência de API](http://graph.microsoft.io/docs/api-reference/v1.0) para explorar o que mais você pode fazer com a API do Microsoft Graph.
