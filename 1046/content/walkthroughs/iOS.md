# Chamar o Microsoft Graph em um aplicativo do iOS

Neste artigo, analisamos as tarefas mínimas necessárias para conectar seu aplicativo ao Office 365 e chamar a API do Microsoft Graph. Usamos código de [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) para explicar os principais conceitos que você deve implementar em seu aplicativo. Esse exemplo cobre as noções básicas para autenticar com o AAD (Microsoft Azure Active Directory) e para fazer uma simples chamada de serviço em relação ao serviço de email do Office 365 usando a API do Microsoft Graph (enviando um email). É recomendável que você clone ou baixe o projeto deste repositório para acompanhar o artigo. 


Este artigo faz referência à [ADAL (Biblioteca de autenticação do Microsoft Azure Active Directory) para iOS e OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc), e o exemplo [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) autentica usando essa biblioteca. Confira esse repositório para saber mais sobre o uso e a implementação do projeto de iOS.


## Visão geral

Para chamar a API do Microsoft Graph, seu aplicativo iOS deve concluir as seguintes tarefas:

1. Registrar o aplicativo do AD (Active Directory) do Microsoft Azure.
2. Solicitar e obter um token de acesso do AD do Azure.
3. Usar o token de acesso em uma solicitação REST para a API do Microsoft Graph. 



## Registrar o aplicativo no Azure Active Directory

Antes de começar a trabalhar com o Office 365, você precisa registrar seu aplicativo e definir permissões para usar os serviços do Microsoft Graph. Com apenas alguns cliques, você pode registrar seu aplicativo para acessar uma conta corporativa ou de estudante de um usuário usando a [Ferramenta de Registro de Aplicativo](https://dev.office.com/app-registration). 
Para gerenciá-la, você precisará acessar o [portal de Gerenciamento do Microsoft Azure](https://manage.windowsazure.com)

Como alternativa, confira a seção **Registrar seu aplicativo nativo no Portal de gerenciamento do Azure** no artigo [Registrar manualmente seu aplicativo no Azure AD para que ele possa acessar as APIs do Office 365](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually) a fim de obter instruções sobre como registrar manualmente o aplicativo. Tenha em mente os seguintes detalhes:

* Você precisará fornecer um URI de redirecionamento para o registro. Ele é um valor obrigatório que especifica para onde o usuário será redirecionado após uma tentativa de autenticação bem-sucedida. Se você não especificar o URI de redirecionamento correto, a solicitação de autenticação falhará.
* No registro, o aplicativo deve receber a **permissão Enviar email como um usuário conectado** para o **Microsoft Graph**.  


Anote os valores a seguir na página **Configurar** do seu aplicativo Azure.

* ID do cliente
* URI de redirecionamento

Você precisa desses valores para configurar o fluxo OAuth em seu aplicativo. 

## Solicitar e obter um token de acesso do Azure AD

Para solicitar e obter um token de acesso para chamar a API do Microsoft Graph, você pode usar **acquireAuthTokenWithResource:clientId:redirectUri:completionBlock:** fornecido pela [ADAL (Biblioteca de autenticação do Azure Active Directory) para iOS e OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc). Esse SDK dá ao seu aplicativo toda a funcionalidade do Microsoft Azure AD, incluindo suporte a protocolo padrão do setor para OAuth2, integração de API Web com consentimento no nível do usuário e suporte à autenticação de dois fatores.

Esse método usa os seguintes parâmetros:

1. **resourceID** – É o recurso necessário que você deseja acessar. Por exemplo, queremos acessar a API do Microsoft Graph; portanto, esse valor seria "https://graph.microsoft.com".
2. **clientID** – O valor atribuído para identificar seu aplicativo quando você concluiu o registro no AAD.
3. **redirectURI** – Novamente, esse é um valor necessário que especifica para onde o usuário será redirecionado após uma tentativa de autenticação bem-sucedida.


Primeiro, você precisa especificar um contexto de autenticação. Isso simplesmente define a autoridade de onde você deseja obter o token de acesso. No nosso caso, é de um locatário do AAD e você precisará declará-lo:

    @property (strong,    nonatomic) ADAuthenticationContext *context;

Em seguida, inicialize-o com a localização da autoridade ("https://login.microsoftonline.com/common"):

    self.context = [ADAuthenticationContext authenticationContextWithAuthority:self.authority]; 


No exemplo [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample), criamos uma classe de autenticação única (**AuthenticationManager**), para fins de demonstração, que é inicializada com a autoridade e os parâmetros necessários. Novamente, essa classe é simplesmente um exemplo de como abordar o fluxo de trabalho de autenticação. Um segmento de código de seu interesse: 



    - (void)acquireAuthTokenWithResource:(NSString *)resourceID
                            clientID:(NSString*)clientID
                         redirectURI:(NSURL*)redirectURI
                          completion:(void (^)(ADAuthenticationError *error))completion {
    
    NSLog(@"acquireAuthTokenWithResource");
    [self.context acquireTokenWithResource:resourceID
                                  clientId:clientID
                               redirectUri:redirectURI
                           completionBlock:^(ADAuthenticationResult *result) {
                               NSLog(@"Completion");
                               
                               if (result.status !=AD_SUCCEEDED){
                                   NSLog(@"error");
                                   completion(result.error);
                               }
                               
                               else{
                                   NSLog(@"complete!");
                                   self.accessToken = result.accessToken;
                                   self.userID = result.tokenCacheStoreItem.userInformation.userId;
                                   completion(nil);
                               }
                           }];


Na primeira vez em que este aplicativo for executado, o Gerenciador de Autenticação enviará uma solicitação à autoridade, 
que o redirecionará para uma página de entrada. Você fornecerá suas credenciais e a resposta conterá o resultado de autenticação. 
Se for bem-sucedida, ela também conterá tokens de atualização e de acesso. Na segunda vez em que o aplicativo for executado e supondo 
que você não limpe o cache e os cookies do token, o Gerenciador de Autenticação e usará o token de acesso ou de atualização em cache para autenticar as solicitações de clientes. 
Isso resultará em uma chamada ao serviço se você precisar obter um token de acesso. 


## Usar o token de acesso em uma solicitação para a API do Microsoft Graph

Com um token de acesso, o aplicativo pode fazer solicitações autenticadas à API do Microsoft Graph. Seu aplicativo deve anexar o token de acesso ao cabeçalho de solicitação HTTP em **Authorization**.

O exemplo [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) envia um email usando o ponto de extremidade sendMail na API do Microsoft Graph. Mais uma vez, em nosso exemplo, criamos uma classe de autenticação simples (AuthenticationManager) que é inicializada com o token de acesso. Precisaremos do token de acesso para construir nossa solicitação.



    - (void)sendMailREST {
    
    AuthenticationManager *authManager = [AuthenticationManager sharedInstance];

    //Helper method used to construct the email message
    NSData *postData = [self mailContent];
    
    //Microsoft Graph API endpoint for sending mail
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:@"https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail"]];

    [request setHTTPMethod:@"POST"];
    [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    [request setValue:@"application/json, text/plain, */*" forHTTPHeaderField:@"Accept"];
    
    // Access token required for request header
    NSString *authorization = [NSString stringWithFormat:@"Bearer %@", authManager.accessToken];
    [request setValue:authorization forHTTPHeaderField:@"Authorization"];
    [request setHTTPBody:postData];

    NSURLConnection *conn = [[NSURLConnection alloc] initWithRequest:request delegate:self];
    
    if(conn) {
        NSLog(@"Connection Successful");
    } else {
        NSLog(@"Connection could not be made");
    }
    
    [conn start];

Como você pode ver, a resposta é tratada com os representantes NSURLConnection, ou seja, NSURLConnectionDelegate e NSURLConnectionDataDelegate.

## Próximas etapas

Se o token de acesso expirar ou estiver prestes a expirar, você poderá usar **acquireTokenSilentWithResource:clientId:redirectUri:completionBlock:** do ADAuthenticationContext para adquirir um novo token de acesso. Seu uso é abordado no exemplo [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample). Além disso, você pode encontrar o código para limpar o cache de token e os cookies armazenados.  

A API do Microsoft Graph é uma API unificadora muito poderosa que pode ser usada para interagir com todos os tipos de dados da Microsoft. Confira a [referência de API](http://graph.microsoft.io/docs/api-reference/v1.0) para explorar o que mais você pode fazer com a API do Microsoft Graph.

