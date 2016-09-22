# Chamar o Microsoft Graph em um aplicativo universal do Windows 10

Neste artigo, analisamos as tarefas mínimas necessárias para se obter um token de acesso do Azure AD (Active Directory) e chamar o Microsoft Graph. Usamos código do [exemplo do Office 365 Connect para UWP usando o Microsoft Graph](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample) para explicar os principais conceitos que você precisa implementar em seu aplicativo.

## Exemplo de interface do usuário

O exemplo contém uma interface do usuário muito simples, que consiste em uma barra de comando superior, um botão **Conectar**, um botão **Enviar email** e uma caixa de texto que é preenchida automaticamente com o endereço de email do usuário conectado, mas que pode ser editada. A barra de comando também contém um botão que habilita os desenvolvedores a localizar o URI de redirecionamento do aplicativo.

O botão **Enviar email** é desabilitado quando o usuário não está conectado:

![Tela mostrando o botão Conectar habilitado e o botão Enviar email desabilitado](images/SignedOut.png)

A barra de comando superior contém um botão Desconectar quando o usuário está conectado:

![Tela mostrando o endereço de email do usuário conectado e o botão Enviar email habilitado](images/SignedIn.png)

Todas as cadeias de caracteres de interface do usuário do exemplo são armazenadas no arquivo Resources.resw dentro da pasta Ativos.

## Registrar o aplicativo
 
O Windows 10 fornece a cada aplicativo um URI exclusivo e garante que as mensagens enviadas para esse URI sejam enviadas somente para esse aplicativo. Você precisa criar seu aplicativo e encontrar esse URI gerado pelo sistema antes de registrá-lo. No exemplo, você encontrará esse método no arquivo AuthenticationHelper.cs:

```c#
        public static string GetAppRedirectURI()
        {
            // Windows 10 universal apps require redirect URI in the format below. Add a breakpoint to this line and run the app before you register it, so that
            // you can supply the correct redirect URI value.
            return string.Format("ms-appx-web://microsoft.aad.brokerplugin/{0}", WebAuthenticationBroker.GetCurrentApplicationCallbackUri().Host).ToUpper();
        }
```

O método é disparado no exemplo pelo botão **Copiar URI de redirecionamento**, mas você também pode seguir o padrão no exemplo [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM), em que a cadeia de caracteres é definida na declaração da classe MainPage, e pode obtê-lo usando o depurador do Visual Studio. 

Siga as etapas em [Registrar e configurar o aplicativo](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample#register) do arquivo Leiame do exemplo para registrar o aplicativo depois de receber o valor do URI de redirecionamento.

Você precisará do valor da ID do cliente da página **Configurar** do aplicativo do Azure ao configurar a autenticação do aplicativo.

## Conectar-se ao Microsoft Graph

O exemplo usa a API nativa do Windows 10 WebAccountManager para autenticar os usuários. Ele segue um padrão semelhante ao descrito na postagem de blog [Desenvolver Aplicativos Universais do Windows com o Azure AD e a API de identidade do Windows 10](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx) e demonstrada no exemplo [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM).

O arquivo App.xaml contém os pares chave/valor de que o aplicativo precisa para autenticar o usuário e autorizar o aplicativo a enviar um email:

```xml
    <Application.Resources>
        <!-- Add your client id here. -->
        <x:String x:Key="ida:ClientID"><your client id></x:String>
        <x:String x:Key="ida:AADInstance">https://login.microsoftonline.com/</x:String>
        <!-- Add your developer tenant domain here. -->
        <x:String x:Key="ida:Domain">yourtenant.onmicrosoft.com</x:String>
    </Application.Resources>
```

Adicione o valor da ID do cliente que você obteve ao registrar o aplicativo como o valor da chave **ida:ClientID**. Altere o valor da chave **ida:Domain** para que ele corresponda ao seu locatário do Office 365.

O arquivo AuthenticationHelper.cs contém todo o código de autenticação, juntamente com a lógica adicional que armazena informações do usuário e força a autenticação somente quando o usuário se desconecta do aplicativo.

O método ``GetTokenHelperAsync`` definido nesse arquivo é executado quando o usuário autentica e toda vez que o aplicativo faz uma chamada ao Microsoft Graph depois disso. Sua primeira tarefa é encontrar um provedor de conta do Azure AD:

```c#
           aadAccountProvider = await WebAuthenticationCoreManager.FindAccountProviderAsync("https://login.microsoft.com", authority);
```

O valor ``authority`` é uma cadeia de caracteres concatenada criada usando dois valores armazenados no arquivo App.xaml: o valor da chave **ida:AADInstance** mais o valor da chave **ida:Domain**. Isso cria uma autoridade específica do locatário. Você também poderá usar a cadeia de caracteres "organizations" se quiser que o aplicativo seja executado em qualquer locatário do Azure AD.

Depois da autenticação do usuário, o aplicativo armazena o valor da ID do usuário em ``ApplicationData.Current.RoamingSettings``. O método ``GetTokenHelperAsync`` primeiro verifica se existe esse valor e, nesse caso, tentará autenticar silenciosamente:

```c#
            // Check if there's a record of the last account used with the app
            var userID = _settings.Values["userID"];

            if (userID != null)
            {

                WebTokenRequest webTokenRequest = new WebTokenRequest(aadAccountProvider, string.Empty, clientId);
                webTokenRequest.Properties.Add("resource", ResourceUrl);

                // Get an account object for the user
                userAccount = await WebAuthenticationCoreManager.FindAccountAsync(aadAccountProvider, (string)userID);


                // Ensure that the saved account works for getting the token we need
                WebTokenRequestResult webTokenRequestResult = await WebAuthenticationCoreManager.RequestTokenAsync(webTokenRequest, userAccount);
                if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.Success || webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.AccountSwitch)
                {
                    WebTokenResponse webTokenResponse = webTokenRequestResult.ResponseData[0];
                    userAccount = webTokenResponse.WebAccount;
                    token = webTokenResponse.Token;

                }
                else
                {
                    // The saved account could not be used for getting a token
                    // Make sure that the UX is ready for a new sign in
                    SignOut();
                }

            }
```

O aplicativo usa o ponto de extremidade do Microsoft Graph, **https://graph.microsoft.com/**, como o valor de recurso. Ao construir o ``WebTokenRequest``, ele usa o valor de ID do cliente que você adicionou ao arquivo App.xaml. Como o aplicativo sabe que a ID de usuário e o usuário ainda não se desconectaram, a API WebAccountManager localiza a conta de usuário e a passa para a solicitação de token. O método ``WebAuthenticationCoreManager.RequestTokenAsync`` retorna um token de acesso com as permissões apropriadas atribuídas a ele.

Se o aplicativo não localiza nenhum valor para ``userID`` nas configurações em roaming, ele constrói uma ``WebTokenRequest`` que força o usuário a autenticar usando a interface do usuário:

```c#
            else
            {
                // There is no recorded user. Start a sign in flow without imposing a specific account.

                WebTokenRequest webTokenRequest = new WebTokenRequest(aadAccountProvider, string.Empty, clientId, WebTokenRequestPromptType.ForceAuthentication);
                webTokenRequest.Properties.Add("resource", ResourceUrl);

                WebTokenRequestResult webTokenRequestResult = await WebAuthenticationCoreManager.RequestTokenAsync(webTokenRequest);
                if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.Success)
                {
                    WebTokenResponse webTokenResponse = webTokenRequestResult.ResponseData[0];
                    userAccount = webTokenResponse.WebAccount;
                    token = webTokenResponse.Token;

                }
            }
```

Se uma das tentativas de recuperar um token for bem-sucedida, o método ``GetTokenHelperAsync`` conclui armazenando informações importantes do usuário nas configurações de roaming e retornando o valor do token. Caso contrário, ele garante que as configurações em roaming são nulas e retorna um valor nulo.

```c#
            // We succeeded in getting a valid user.
            if (userAccount != null)
            {
                // save user ID in local storage
                _settings.Values["userID"] = userAccount.Id;
                _settings.Values["userEmail"] = userAccount.UserName;
                _settings.Values["userName"] = userAccount.Properties["DisplayName"];

                return token;
            }

            // We didn't succeed in getting a valid user. Clear the app settings so that another user can sign in.
            else
            {
                
                SignOut();
                return null;
            }
```

## Enviar um email com o Microsoft Graph

O arquivo MailHelper.cs contém o código que constrói e envia um email. Consiste em um único método, ``ComposeAndSendMailAsync``, que constrói e envia uma solicitação POST para o ponto de extremidade **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail**. 

O método ``ComposeAndSendMailAsync`` tem três valores de cadeia de caracteres, ou seja, ``subject``, ``bodyContent`` e ``recipients``, que são passados para ele pelo arquivo MainPage.xaml.cs. As cadeias de caracteres ``subject`` e ``bodyContent`` são armazenadas, juntamente com todos os outras cadeias de caracteres de interface do usuário, no arquivo Resources.resw. A cadeia de caracteres ``recipients`` vem da caixa de endereço na interface do aplicativo. 

Já que o usuário pode passar mais de um endereço, a primeira tarefa é dividir a cadeia de caracteres ``recipients`` em uma coleção de objetos EmailAddress que podem ser passados no corpo POST da solicitação:

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }
```

A segunda tarefa é construir um objeto de mensagem JSON válido e enviá-lo para o ponto de extremidade **me/microsoft.graph.SendMail** por meio de uma solicitação HTTP POST. Já que a cadeia de caracteres ``bodyContent`` é um documento HTML, a solicitação define o valor **ContentType** como HTML. Além disso, anote a chamada a ``AuthenticationHelper.GetTokenHelperAsync`` para garantir um token de acesso atualizado a fim de passar na solicitação.

```c#
                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenHelperAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }
```

Após fazer uma solicitação REST bem-sucedida, você realizou as três etapas necessárias para interagir com o Microsoft Graph: registro do aplicativo, autenticação do usuário e solicitação REST.


<!--## Additional resources

* [Develop Windows Universal Apps with Azure AD and the Windows 10 Identity API](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx)
* [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM)
* [Office Dev Center](http://dev.office.com)-->

