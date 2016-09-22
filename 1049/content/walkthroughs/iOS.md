# Вызов Microsoft Graph в приложении iOS

В этой статье мы рассмотрим, как подключить приложение к Office 365 и вызвать API Microsoft Graph. Мы объясним основные понятия на примере приложения [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample). На этом примере мы покажем основные принципы проверки подлинности с помощью Microsoft Azure Active Directory (AAD) и вызова почтовой службы Office 365 с использованием API Microsoft Graph (отправки сообщения). Прежде чем приступить к изучению данной статьи, рекомендуем клонировать или скачать проект из этого репозитория. 


В этой статье упоминается [библиотека Microsoft Azure Active Directory Authentication Library (ADAL) для iOS и OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc), с ее помощью приложение [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) выполняет проверку подлинности. Дополнительные сведения об использовании и реализации в проекте iOS см. в этом репозитории.


## Обзор

Чтобы вызвать API Microsoft Graph в приложении iOS, необходимо выполнить следующие действия:

1. Зарегистрировать приложение в Microsoft Azure Active Directory (AD).
2. Запросить и получить токен доступа в Azure AD.
3. Использовать токен доступа в запросе REST к API Microsoft Graph. 



## Регистрация приложения в Azure Active Directory

Прежде чем начать работу с Office 365, необходимо зарегистрировать свое приложение и настроить разрешения на использование служб Microsoft Graph. С помощью [средства регистрации приложений](https://dev.office.com/app-registration) вы можете с легкостью зарегистрировать свое приложение для доступа к рабочей или учебной учетной записи пользователя. 
Для управления своей учетной записью пользователю необходимо перейти на [портал управления Microsoft Azure](https://manage.windowsazure.com).

Инструкции по регистрации приложения вручную см. в разделе **Регистрация собственного приложения на портале управления Azure** статьи [Регистрация приложения вручную в Azure AD для доступа к API Office 365](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually). Обратите внимание на следующие рекомендации:

* Во время регистрации необходимо указать URI перенаправления. Это обязательное значение, которое определяет, куда будет перенаправлен пользователь после успешной проверки подлинности. Если вы не укажете правильный URI перенаправления, запрос на проверку подлинности будет отклонен.
* Во время регистрации приложению необходимо предоставить разрешение **Отправлять почту как вошедший пользователь** для **Microsoft Graph**.  


Запишите следующие значения со страницы **Настройка** приложения Azure:

* Идентификатор клиента
* URI перенаправления.

Эти значения понадобятся для настройки потока OAuth в приложении. 

## Запрос и получение токена доступа из Azure AD

Чтобы запросить и получить токен доступа для вызова API Microsoft Graph, вы можете использовать пакет SDK **acquireAuthTokenWithResource:clientId:redirectUri:completionBlock:**, доступный в [библиотеке Microsoft Azure Active Directory Authentication Library (ADAL) для iOS и OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc). Этот пакет SDK обеспечивает приложению доступ ко всем функциям Microsoft Azure AD, в том числе поддержку стандартного протокола OAuth2, интеграцию веб-API с согласием пользователя, а также поддержку двухфакторной проверки подлинности.

Метод принимает следующие параметры:

1. **resourceID**. Ресурс, к которому нужно получить доступ. Например, чтобы вызвать API Microsoft Graph, задайте для этого параметра значение "https://graph.microsoft.com".
2. **clientID**. Значение, идентифицирующее приложение после регистрации в службе AAD.
3. **redirectURI**. Это обязательное значение, которое определяет, куда будет перенаправлен пользователь после успешной проверки подлинности.


Сначала необходимо указать контекст проверки подлинности. Это просто центр, из которого необходимо получить токен доступа. В нашем случае это клиент AAD, который нужно объявить:

    @property (strong,    nonatomic) ADAuthenticationContext *context;

Затем инициализируйте его, указав расположение центра ("https://login.microsoftonline.com/common").

    self.context = [ADAuthenticationContext authenticationContextWithAuthority:self.authority]; 


В примере [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) мы создали одноэлементный класс проверки подлинности (**AuthenticationManager**), который инициализируется с помощью центра и необходимых параметров. Этот класс — просто пример того, как должна работать проверка подлинности. Нас интересует следующий сегмент кода: 



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


При первом запуске этого приложения диспетчер проверки подлинности отправит запрос в центр, 
который перенаправит вас на страницу входа. Вы укажите учетные данные, а ответ будет содержать результат проверки подлинности. 
Если она пройдена успешно, он также будет включать маркеры обновления и доступа. 
Если вы не очищали кэш маркеров и файлы cookie, то при втором запуске этого приложения диспетчер проверки подлинности использует маркер доступа или обновления в кэше, чтобы проверить подлинность клиентских запросов. 
Это приведет к вызову службы, если вам нужно получить маркер доступа. 


## Использование маркера доступа в запросе к API Microsoft Graph

Используя токен доступа, приложение может отправлять проверенные запросы в API Microsoft Graph. Приложение должно добавлять токен доступа в заголовок HTTP-запроса **Authorization**.

Приложение [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) отправляет электронную почту, используя конечную точку sendMail в API Microsoft Graph. В нашем примере мы создали одноэлементный класс проверки подлинности (AuthenticationManager), который инициализируется с помощью токена доступа. Токен доступа понадобится нам для создания запроса.



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

Как видите, для обработки запроса используются делегаты NSURLConnection, а именно NSURLConnectionDelegate и NSURLConnectionDataDelegate.

## Дальнейшие действия

Если срок действия токена доступа истек или скоро истечет, вы можете получить новый токен доступа с помощью метода **acquireTokenSilentWithResource:clientId:redirectUri:completionBlock:** класса ADAuthenticationContext. Сведения о его использовании см. в примере [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample). Кроме того, вы можете найти код для очистки кэша токенов и сохраненных файлов cookie.  

API Microsoft Graph — это функциональный единый API, с помощью которого можно взаимодействовать со все типами данных Майкрософт. Сведения о других возможностях API Microsoft Graph см. в [справочнике по API](http://graph.microsoft.io/docs/api-reference/v1.0).

