# Вызов Microsoft Graph в приложении ASP.NET MVC

В этой статье мы рассмотрим минимальный набор действий, необходимых для подключения вашего приложения к Office 365 и вызова API Microsoft Graph. В этой статье мы не будем создавать приложение с нуля. Мы рассмотрим основные понятия, которые необходимо реализовать в приложении, на примере кода [приложения ASP.NET MVC Connect для Office 365, использующего Microsoft Graph](https://github.com/microsoftgraph/aspnet-connect-rest-sample).

Ниже приведен снимок экрана со страницей отправки почты.

![Снимок экрана с примером ASP.NET MVC в Office 365](./images/O365AspNetMVCSendMailPageScreenshot.png)

## Обзор

Чтобы вызвать API Microsoft Graph, необходимо выполнить следующие задачи:

1. Зарегистрировать приложение в Azure Active Directory.
2. Проверить подлинность пользователя и получить маркер доступа, вызвав методы в библиотеке Azure AD Authentication Library (ADAL) для .NET.
3. Получить маркер доступа с помощью библиотеки ADAL.
4. Использовать маркер доступа в запросе к API Microsoft Graph
5. Завершить сеанс.

<!--<a name="register"></a>-->
## Регистрация приложения в Azure Active Directory

Прежде чем начать работу с Office 365, необходимо зарегистрировать свое приложение и настроить разрешения на использование служб Microsoft Graph. 
С помощью [средства регистрации приложений](https://dev.office.com/app-registration) вы можете с легкостью зарегистрировать свое приложение для доступа к рабочей или учебной учетной записи пользователя. Для управления своей учетной записью пользователю необходимо перейти на [портал управления Microsoft Azure](https://manage.windowsazure.com).

Альтернативные инструкции см. в разделе [Регистрация браузерного веб-приложения на портале управления Azure](https://msdn.microsoft.com/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp). Обратите внимание на следующие рекомендации:

* Обязательно укажите http://localhost:55065/ в поле **URL-адрес входа**.
* Зарегистрировав приложение, [настройте **делегированные разрешения**](https://github.com/microsoftgraph/aspnet-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), необходимые для вашего приложения Angular. Для приложения Connect необходимо разрешение **Отправлять почту как вошедший пользователь**.

Запишите следующие значения, указанные на странице **Настройка** приложения Azure, так как они понадобятся для настройки в приложении:

* идентификатор клиента (уникальный для приложения);
* ключ (который также называется секретом клиента);
* URL-адрес ответа (который также называется URL-адресом перенаправления). В нашем примере это http://localhost:55065/.

  > Примечание.  В качестве URL-адреса ответа автоматически задается URL-адрес входа, указанный при регистрации приложения.

<!--<a name="#auth"></a>-->
## Проверка подлинности в приложении Connect

С помощью библиотеки проверки подлинности Azure AD (ADAL) для .NET разработчики клиентских приложений могут проверять подлинность пользователей и получать маркеры доступа для вызовов API.  Вы можете добавить эту библиотеку в проект ASP.NET MVC с помощью команды **Управление пакетами NuGet** в Visual Studio.

Ниже приведен снимок экрана с домашней страницей.

![Снимок экрана с примером ASP.NET MVC в Office 365](./images/O365AspNetMVCHomePageScreenshot.png)

Проверка подлинности состоит из двух основных этапов:

1. Запрос кода авторизации.
2. Использование кода авторизации для запроса маркера доступа.

>  **Примечание**. Вместе с маркером доступа вы также получите маркер обновления. С помощью маркера обновления можно получить новый маркер доступа после истечения срока действия текущего маркера доступа.

В приложении Connect для проверки подлинности используются значения регистрации приложения Azure и идентификатор пользователя. Для проверки подлинности ADAL требуется идентификатор клиента, ключ и URL-адрес ответа (который также называется URL-адресом перенаправления), полученные в процессе регистрации Azure.

Чтобы запросить код авторизации, сначала перенаправьте приложение на URL-адрес запроса авторизации Azure AD, как показано ниже (см. файл HomeController.cs).


```c#
        public ActionResult Login()
        {
            if (string.IsNullOrEmpty(Settings.ClientId) || string.IsNullOrEmpty(Settings.ClientSecret))
            {
                ViewBag.Message = "Please set your client ID and client secret in the Web.config file";
                return View();
            }


            var authContext = new AuthenticationContext(Settings.AzureADAuthority);

            // Generate the parameterized URL for Azure login.
            Uri authUri = authContext.GetAuthorizationRequestURL(
                Settings.O365UnifiedAPIResource,
                Settings.ClientId,
                loginRedirectUri,
                UserIdentifier.AnyUser,
                null);

            // Redirect the browser to the login page, then come back to the Authorize method below.
            return Redirect(authUri.ToString());
        }

```
После вызова этого метода **Login** приложение перенаправит пользователя на страницу входа. В приложении откроется страница входа. После успешной проверки подлинности учетных данных пользователя Azure перенаправит приложение на URL-адрес перенаправления, обозначенный в коде как *loginRedirectUri*. Этот URL-адрес перенаправления указывает на другое действие в приложении ASP.NET MVC, как показано ниже.

```c#

 Uri loginRedirectUri => new Uri(Url.Action(nameof(Authorize), "Home", null, Request.Url.Scheme));

```
URL-адрес также будет содержать код авторизации, указанный на шагах 1 и 2 выше.  Код авторизации будет получен из параметров запроса. Используя код авторизации, приложение выполнит вызов в Azure AD, чтобы получить маркер доступа. После получения маркера доступа его необходимо сохранить в сеансе, чтобы использовать для нескольких запросов.

Действие Authorize, указанное в действии URL-адреса перенаправления, выглядит следующим образом:

```c#
        public async Task<ActionResult> Authorize()
        {
            var authContext = new AuthenticationContext(Settings.AzureADAuthority);


            // Get the token.
            var authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
                Request.Params["code"],                                         // the auth 'code' parameter from the Azure redirect.
                loginRedirectUri,                                               // same redirectUri as used before in Login method.
                new ClientCredential(Settings.ClientId, Settings.ClientSecret), // use the client ID and secret to establish app identity.
                Settings.O365UnifiedAPIResource);

            // Save the token in the session.
            Session[SessionKeys.Login.AccessToken] = authResult.AccessToken;

            // Get info about the current logged in user.
            Session[SessionKeys.Login.UserInfo] = await UnifiedApiHelper.GetUserInfoAsync(authResult.AccessToken);

            return RedirectToAction(nameof(Index), "Message");

        }

```
>  **Примечание**.  Дополнительные сведения о потоке авторизации см. в статье [Поток предоставления кода авторизации] (https://msdn.microsoft.com/ru-RU/library/azure/dn645542.aspx)

<!--<a name="request"></a>-->
## Использование токена доступа в запросе к API Microsoft Graph

После входа пользователя в приложении Connect отображается действие для отправки почтового сообщения.  Используя маркер доступа, ваше приложение может отправлять прошедшие проверку запросы в API Microsoft Graph.

Например, файл UnifiedApiHelper.cs содержит код, который:

1)  получает сведения о текущем пользователе.  Метод ``GetUserInfoAsync`` принимает один аргумент (значение маркера доступа) для вызова **https://graph.microsoft.com/v1.0/me**, чтобы получить сведения о текущем пользователе;

 ```c#

        public static async Task<UserInfo> GetUserInfoAsync(string accessToken)
        {
            UserInfo myInfo = new UserInfo();

            using (var client = new HttpClient())
            {
                using (var request = new HttpRequestMessage(HttpMethod.Get, Settings.GetMeUrl))
                {
                    request.Headers.Accept.Add(Json);
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

                    using (var response = await client.SendAsync(request))
                    {
                        if (response.StatusCode == HttpStatusCode.OK)
                        {
                            var json = JObject.Parse(await response.Content.ReadAsStringAsync());
                            myInfo.Name = json?["displayName"]?.ToString();
                            myInfo.Address = json?["mail"]?.ToString().Trim().Replace(" ", string.Empty);

                        }
                    }
                }
            }

            return myInfo;
        }

```



2)  составляет и отправляет сообщение, которое вошедший пользователь хочет отправить по электронной почте. Метод ``SendMessageAsync`` составляет и отправляет запрос POST на URL-адрес ресурса **https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail**, используя значение маркера доступа в качестве одного из аргументов.


```c#

        public static async Task<SendMessageResponse> SendMessageAsync(string accessToken, SendMessageRequest sendMessageRequest)
        {
            var sendMessageResponse = new SendMessageResponse { Status = SendMessageStatusEnum.NotSent };

            using (var client = new HttpClient())
            {
                using (var request = new HttpRequestMessage(HttpMethod.Post, Settings.SendMessageUrl))
                {
                    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
                    request.Content = new StringContent(JsonConvert.SerializeObject(sendMessageRequest), Encoding.UTF8, "application/json");
                    using (HttpResponseMessage response = await client.SendAsync(request))
                    {
                        if (response.IsSuccessStatusCode)
                        {
                            sendMessageResponse.Status = SendMessageStatusEnum.Sent;
                            sendMessageResponse.StatusMessage = null;
                        }
                        else
                        {
                            sendMessageResponse.Status = SendMessageStatusEnum.Fail;
                            sendMessageResponse.StatusMessage = response.ReasonPhrase;
                        }
                    }
                }
            }

            return sendMessageResponse;
        }

```


Файл ``MessageController.cs `` содержит код, управляющий сообщениями электронной почты. К примеру, рассмотрим кнопку **Отправить почту**. Метод ``SendMessageSubmit `` отправляет сообщение, когда пользователь нажимает кнопку **Отправить почту**.


```c#

        public async Task<ActionResult> SendMessageSubmit(UserInfo userInfo)
        {
            // After Index method renders the View, user clicks Send Mail, which comes in here.
            EnsureUser(ref userInfo);

            // Send email using O365 unified API.
            var sendMessageResult = await UnifiedApiHelper.SendMessageAsync(
                (string)Session[SessionKeys.Login.AccessToken],
                GenerateEmail(userInfo));

            // Reuse the Index view for messages (sent, not sent, fail) .
            // Redirect to tell the browser to call the app back via the Index method.
            return RedirectToAction(nameof(Index), new RouteValueDictionary(new Dictionary<string,object>{
                { "Status", sendMessageResult.Status },
                { "StatusMessage", sendMessageResult.StatusMessage },
                { "Address", userInfo.Address },
            }));
        }

```


Метод ``CreateEmailObject`` создает объект электронной почты в формате или контракте данных, указанном в основном тексте запроса POST:


  ```c#

        private SendMessageRequest CreateEmailObject(UserInfo to, string subject, string body)
        {
            return new SendMessageRequest
            {
                Message = new Message
                {
                    Subject = subject,
                    Body = new MessageBody
                    {
                        ContentType = "Html",
                        Content = body
                    },
                    ToRecipients = new List<Recipient>
                    {
                        new Recipient
                        {
                            EmailAddress = new UserInfo
                            {
                                 Name =  to.Name,
                                 Address = to.Address
                            }
                        }
                    }
                },
                SaveToSentItems = true
            };

```

Кроме того, необходимо создать допустимую строку сообщения JSON и отправить ее в конечную точку ``https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail`` с помощью запроса HTTP POST. Так как текст сообщения должен отправляться в виде документа HTML, запрос задает для параметра ``ContentType`` сообщения значение HTML и преобразует содержимое в формат JSON для запроса HTTP POST. Файл UnifiedApiMessageModels.cs содержит контракты данных или схем между этим приложением и сервером единого API Office 365.



```c#


    public class SendMessageResponse
    {
        public SendMessageStatusEnum Status { get; set; }
        public string StatusMessage { get; set; }
    }

    public class SendMessageRequest
    {
        public Message Message { get; set; }

        public bool SaveToSentItems { get; set; }
    }

    public class Message
    {
        public string Subject { get; set; }
        public MessageBody Body { get; set; }
        public List<Recipient> ToRecipients { get; set; }
    }
    public class Recipient
    {
        public UserInfo EmailAddress { get; set; }
    }

    public class MessageBody
    {
        public string ContentType { get; set; }
        public string Content { get; set; }
    }

    public class UserInfo
    {
        public string Name { get; set; }
        public string Address { get; set; }
    }

}

```
<!--<a name="logout"></a>-->
## Завершение сеанса

Когда пользователь нажмет **Отключить** на странице отправки почты, сеанс будет завершен. Для этого код:
* очищает локальный сеанс;
* перенаправляет браузер в конечную точку выхода (чтобы служба Azure могла очистить свои файлы cookie).

В методе **Logout** (см. файл HomeController.cs) показано, как это сделать.


```c#
        public ActionResult Logout()
        {
            Session.Clear();
            return Redirect(Settings.LogoutAuthority + logoutRedirectUri.ToString());
        }

```

##Дальнейшие действия
API Microsoft Graph — это единый API, с помощью которого можно взаимодействовать со всеми данными Майкрософт. Сведения о других возможностях API Microsoft Graph см. в справочнике по API. 
Другие примеры приложений ASP.NET см. на сайте [GitHub](http://aka.ms/aspnetgraphsamples).


