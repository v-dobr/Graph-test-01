# Chamar o Microsoft Graph em um aplicativo Ruby 

Neste artigo, analisamos as tarefas mínimas necessárias para se obter um token de acesso do Azure AD (Active Directory) e chamar o Microsoft Graph. Usamos código do [exemplo do Office 365 Ruby Connect usando o Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample) para explicar os principais conceitos que você precisa implementar em seu aplicativo.

![Captura de tela do exemplo de conexão com o Office 365 para Ruby](./images/web-screenshot.png)

## Visão geral

Para chamar a API do Microsoft Graph, seu aplicativo Ruby deve concluir as seguintes tarefas:

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

* Especifique uma rota em seu aplicativo Ruby como a **URL de Logon** na etapa 6. No caso do exemplo do Connect, isso é [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L41).
* [Configure as **Permissões delegadas**](https://github.com/microsoftgraph/ruby-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) que o aplicativo exige. O exemplo do Connect exige uma permissão **Enviar emails como usuário conectado**.

Anote os valores a seguir na página **Configurar** do seu aplicativo Azure.

* ID do cliente
* Uma chave válida
* Uma URL de resposta

Você precisa desses valores como parâmetros no fluxo OAuth no aplicativo.

<!--<a name="redirect"/>-->
## Redirecionar o navegador para a página de entrada

O aplicativo precisa redirecionar o navegador para a página de entrada a fim de obter um código de autorização e continuar o fluxo do OAuth.

No exemplo do Connect, o redirecionamento é tratado pela biblioteca OmniAuth. Nosso aplicativo apenas delega a execução para a rota [`/auth/azureactivedirectory`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L30) gerenciada por OmniAuth.

<!--<a name="authcode"/>-->
## Receber um código de autorização em sua página de URL de resposta

Depois que o usuário entrar, o fluxo fará o navegador retornar para a URL de resposta em seu aplicativo. O Azure anexa um código de autorização à cadeia de caracteres de consulta. O exemplo do Connect usa a rota [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L38) para essa finalidade.

O código de autorização é fornecido na variável `code` da cadeia de consulta. O exemplo do Connect salva o código em uma variável local para usá-lo mais tarde.

```ruby
code = params[:code]
```

<!--<a name="accesstoken"/>-->
## Solicitar um token de acesso do ponto de extremidade do token

Quando tiver o código de autorização, você poderá usá-lo com os valores ID do cliente, chave e URL de resposta obtidos do Azure AD para solicitar um token de acesso. 

> **Observação:** <br />
> A solicitação também tem que especificar um recurso que estamos tentando consumir. No caso do Microsoft Graph, o valor do recurso é `https://graph.microsoft.com`.

Novamente, o exemplo do Connect delega essa tarefa para a biblioteca OmniAuth. A função [`acquire_access_token`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L65) chama a biblioteca e transmite o código de autenticação salvo na seção anterior, juntamente com a URL de resposta, a ID do cliente, o segredo do cliente e a ID do recurso.

```ruby
def acquire_access_token(auth_code, reply_url)
    AUTH_CTX.acquire_token_with_authorization_code(
                  auth_code,
                  reply_url,
                  CLIENT_CRED,
                  GRAPH_RESOURCE)
end
```

> **Observação:** <br />
> A ID do cliente e o segredo de cliente são fornecidos no parâmetro `CLIENT_CRED` no trecho de código anterior.

<!--<a name="request"/>-->
## Usar o token de acesso em uma solicitação para a API do Microsoft Graph

Com um token de acesso, o aplicativo pode fazer solicitações autenticadas à API do Microsoft Graph. O aplicativo deve fornecer o token de acesso no cabeçalho **Authorization** de cada solicitação.

O exemplo do Connect envia um email usando o ponto de extremidade **sendMail** na API do Microsoft Graph. O código está na função [`send_mail`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L82). Esse é o código que mostra como enviar o código de acesso no cabeçalho Authorization.

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

> **Observação:** <br />
> A solicitação também deve enviar um cabeçalho **Content-Type** com um valor aceito pela API do Microsoft Graph, por exemplo, `application/json;odata.metadata=minimal;odata.streaming=true`.

A API do Microsoft Graph é uma API unificadora muito poderosa que pode ser usada para interagir com todos os tipos de dados da Microsoft. Confira a referência de API para explorar o que mais você pode fazer com a API do Microsoft Graph.

<!--## Additional resources

-  [Office 365 Ruby Connect sample using Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference](http://graph.microsoft.io/en-us/docs)-->
