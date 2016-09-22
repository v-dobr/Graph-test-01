# Вызов Microsoft Graph в универсальном приложении для Windows 10

В этой статье мы рассмотрим, как получить маркер доступа из Azure Active Directory (AD) и вызвать Microsoft Graph. Мы объясним основные понятия на примере [приложения Office 365 Connect для универсальной платформы Windows, использующего Microsoft Graph](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample).

## Пользовательский интерфейс приложения

Приложение имеет очень простой пользовательский интерфейс, состоящий из верхней панели команд, **кнопки подключения**, кнопки **отправки почты** и текстового поля, в которое автоматически подставляется адрес электронной почты вошедшего пользователя, но его можно изменить. На панели команд также есть кнопка, с помощью которой разработчики могут определить URI перенаправления приложения.

Если пользователь не подключен, кнопка **отправки почты** неактивна:

![Экран, на котором кнопка подключения активна, а кнопка отправки почты неактивна](images/SignedOut.png)

Если пользователь подключен, на верхней панели команд отображается кнопка отключения:

![Экран с адресом электронной почты подключенного пользователя и активной кнопкой отправки почты](images/SignedIn.png)

Все строки пользовательского интерфейса хранятся в файле Resources.resw в папке Assets.

## Регистрация приложения
 
Windows 10 предоставляет каждому приложению уникальное значение URI, благодаря чему сообщения, отправленные на этот URI, направляются только в это приложение. Прежде чем зарегистрировать приложение, его необходимо создать и определить созданный системой URI. В файле AuthenticationHelper.cs примера вы найдете следующий метод:

```c#
        public static string GetAppRedirectURI()
        {
            // Windows 10 universal apps require redirect URI in the format below. Add a breakpoint to this line and run the app before you register it, so that
            // you can supply the correct redirect URI value.
            return string.Format("ms-appx-web://microsoft.aad.brokerplugin/{0}", WebAuthenticationBroker.GetCurrentApplicationCallbackUri().Host).ToUpper();
        }
```

В примере этот метод запускается с помощью кнопки **копирования URI перенаправления**, но можно также следовать модели в примере [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM), в котором строка определяется в объявлении класса MainPage. Эту строку можно получить при помощи отладчика Visual Studio. 

Чтобы зарегистрировать приложение после получения URI перенаправления, следуйте указаниям в разделе [Регистрация и настройка приложения](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample#register) файла Readme примера.

При настройке приложения для проверки подлинности вам потребуется идентификатор клиента со страницы **Настройка** приложения Azure.

## Подключение к Microsoft Graph

Для проверки подлинности пользователей в примере используется основной API WebAccountManager Windows 10. Он соответствует шаблону, который описан в записи блога [Разработка универсальных приложений для Windows с помощью Azure AD и API удостоверений Windows 10](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx), и показан в примере [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM).

Файл App.xaml содержит пары "ключ-значение", которые требуются приложению для проверки подлинности пользователя и отправки сообщений электронной почты:

```xml
    <Application.Resources>
        <!-- Add your client id here. -->
        <x:String x:Key="ida:ClientID"><your client id></x:String>
        <x:String x:Key="ida:AADInstance">https://login.microsoftonline.com/</x:String>
        <!-- Add your developer tenant domain here. -->
        <x:String x:Key="ida:Domain">yourtenant.onmicrosoft.com</x:String>
    </Application.Resources>
```

Добавьте идентификатор клиента, полученный при регистрации приложения, в качестве значения ключа **ida:ClientID**. Измените значение ключа **ida:Domain**, чтобы оно соответствовало вашему клиенту Office 365.

Файл AuthenticationHelper.cs содержит код проверки подлинности, а также дополнительную логику, которая хранит сведения о пользователях и выполняет обязательную проверку подлинности только при выходе пользователя из приложения.

Определенный в этом файле метод ``GetTokenHelperAsync`` выполняется при проверке подлинности пользователя, а затем каждый раз при вызове Microsoft Graph приложением. Первая задача — определить поставщика учетной записи Azure AD:

```c#
           aadAccountProvider = await WebAuthenticationCoreManager.FindAccountProviderAsync("https://login.microsoft.com", authority);
```

Значение ``authority`` — это объединенная строка из двух значений, которые хранятся в файле App.xaml: значений ключей **ida:AADInstance** и **ida:Domain**. На основе этого значения создаются полномочия отдельных клиентов. Кроме того, если вы хотите, чтобы приложение выполнялось в любом клиенте Azure AD, можно использовать "организации" строк.

Когда пользователь проходит проверку подлинности, приложение сохраняет его идентификатор в ``ApplicationData.Current.RoamingSettings``. При наличии этого значения метод ``GetTokenHelperAsync`` пытается проверить подлинность без уведомления:

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

Приложение использует конечную точку Microsoft Graph — **https://graph.microsoft.com/** — как значение ресурса. При создании ``WebTokenRequest`` оно использует идентификатор клиента, добавленный в файл App.xaml. Так как приложение знает идентификатор пользователя и пользователь не отключился, API WebAccountManager может найти его учетную запись и передать ее в запрос маркера. Метод ``WebAuthenticationCoreManager.RequestTokenAsync`` возвращает маркер доступа с необходимыми разрешениями.

Если приложение не находит значения ``userID`` в параметрах перемещения, оно создает объект ``WebTokenRequest``, после чего пользователь должен войти через пользовательский интерфейс:

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

Если маркер получен, метод ``GetTokenHelperAsync`` сохраняет важные сведения о пользователе в параметрах перемещения и возвращает значение маркера. В противном случае метод сбрасывает параметры перемещения и возвращает пустое значение.

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

## Отправка электронной почты с помощью Microsoft Graph

Файл MailHelper.cs содержит код, который создает и отправляет сообщение электронной почты. Он состоит из одного метода — ``ComposeAndSendMailAsync``, который создает и отправляет запрос POST в конечную точку **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail**. 

Метод ``ComposeAndSendMailAsync`` принимает три строковых значения — ``subject``, ``bodyContent`` и ``recipients``, которые передаются в него из файла MainPage.xaml.cs. Строки ``subject`` и ``bodyContent`` хранятся в файле Resources.resw вместе со всеми остальными строками пользовательского интерфейса. Строка ``recipients`` отражает значение из поля "Адрес" в интерфейсе приложения. 

Так как пользователь может указать несколько адресов, строку ``recipients`` необходимо разбить на набор объектов EmailAddress, которые затем можно передать в POST-тексте запроса:

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

Затем необходимо создать допустимый объект JSON Message и отправить его в конечную точку **me/microsoft.graph.SendMail** с помощью HTTP-запроса POST. Так как строка ``bodyContent`` — это документ HTML, запрос задает для параметра **ContentType** значение HTML. Обратите внимание на вызов ``AuthenticationHelper.GetTokenHelperAsync`` и убедитесь, что в запросе передается новый маркер доступа.

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

Отправка запроса REST, регистрация приложения и проверка подлинности пользователя — это три действия, необходимых для взаимодействия с Microsoft Graph.


<!--## Additional resources

* [Develop Windows Universal Apps with Azure AD and the Windows 10 Identity API](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx)
* [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM)
* [Office Dev Center](http://dev.office.com)-->

