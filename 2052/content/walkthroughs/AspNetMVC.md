# 在 ASP.NET MVC 应用中调用 Microsoft Graph

在本文中，我们将了解连接您的应用程序与 Office 365，以及调用 Microsoft Graph API 至少需要完成的任务。本主题不会从头开始创建应用。我们使用 [Office 365 ASP.NET MVC 连接示例（使用 Microsoft Graph）](https://github.com/microsoftgraph/aspnet-connect-rest-sample)中的代码，说明需要在您的应用中实现的主要概念。

下面是“发送邮件”页的屏幕截图。

![Office 365 ASP.NET MVC 示例的屏幕截图](./images/O365AspNetMVCSendMailPageScreenshot.png)

## 概述

若要调用 Microsoft Graph API，您必须完成以下任务。

1. 在 Azure Active Directory 中注册应用程序
2. 对用户进行身份验证，并通过在 .NET 适用的 Azure AD Authentication Library (ADAL) 上调用方法来获取访问令牌
3. 使用 ADAL 获取访问令牌
4. 在对 Microsoft Graph API 提出的请求中使用访问令牌
5. 断开会话连接

<!--<a name="register"></a>-->
## 在 Azure Active Directory 中注册应用程序

您需要先注册应用程序并设置 Microsoft Graph 服务的使用权限，然后才能开始使用 Office 365。只需单击几下，即可使用[应用程序注册工具](https://dev.office.com/app-registration)将应用程序注册为访问用户的工作或学校帐户。
若要进行管理，您需要转到 [Microsoft Azure 管理门户](https://manage.windowsazure.com)

请参阅[在 Azure 管理门户中注册基于浏览器的 Web 应用](https://msdn.microsoft.com/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp)，了解相关的替代说明，并注意以下细节。

* 务必指定 http://localhost:55065/ 作为**登录 URL**。
* 注册应用程序后，[配置**委托的权限**](https://github.com/microsoftgraph/aspnet-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)（其为 Angular 应用所需）。连接示例需要获得**以登录用户身份发送邮件**的权限。

请记下 Azure 应用程序的“**配置**”页中的下列值，因为你需要使用这些值在应用中进行配置。

* 客户端 ID（您的应用程序专用）
* 密钥（亦称为“客户端密码”）
* 回复 URL（亦称为“重定向 URL”）。对于此示例，回复 URL 为 http://localhost:55065/。

  > 注意：回复 URL 值中会自动填充您在注册应用程序时指定的登录 URL 值。

<!--<a name="#auth"></a>-->
## 连接示例中的身份验证

借助 .NET 适用的 Azure AD Authentication Library (ADAL)，客户端应用程序开发者可以对用户进行身份验证，然后获取访问令牌进行 API 调用。您可以通过 Visual Studio 中的**管理 NuGet 包**，将此库添加到 ASP.NET MVC 项目中。

下面是主页的屏幕截图。

![Office 365 ASP.NET MVC 示例的屏幕截图](./images/O365AspNetMVCHomePageScreenshot.png)

身份验证流可以分成两个基本步骤：

1. 请求获取授权代码。
2. 使用授权代码请求获取访问令牌。

>  **注意**：除了访问令牌之外，您还会获得刷新令牌。如果当前访问令牌过期，则您可以使用刷新令牌获取新的访问令牌。

连接示例使用 Azure 应用注册值和用户的 ID 进行身份验证。ADAL 身份验证流需要使用您在 Azure 注册过程中获得的客户端 ID、密钥和回复 URL（亦称为“重定向 URL”）。

若要请求获取授权代码，请先将应用重定向到 Azure AD 授权请求 URL（如下所示，见 HomeController.cs 文件）。


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
在调用**登录**方法后，应用会将用户重定向到登录页。这会将应用转至登录页。成功验证用户凭据后，Azure 便会将应用重定向到代码中提及的重定向 URL（以 *loginRedirectUri* 表示）。此重定向 URL 是转至 ASP.NET MVC 应用中另一操作的 URL（如下所示）。

```c#

 Uri loginRedirectUri => new Uri(Url.Action(nameof(Authorize), "Home", null, Request.Url.Scheme));

```
此 URL 还包含上述第 1 步和第 2 步中提及的授权代码。这会通过请求参数获得授权代码。应用会使用授权代码来调用 Azure AD，以便获取访问令牌。在获得访问令牌后，我们将它存储在会话中，以便用于多个请求。

重定向 URL 操作中提及的授权操作如下所示。

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
>  **注意**：有关授权流的详细信息，请参阅 [授权代码授予流] (https://msdn.microsoft.com/zh-CN/library/azure/dn645542.aspx)

<!--<a name="request"></a>-->
## 在对 Microsoft Graph API 提出的请求中使用访问令牌

在用户登录后，连接示例会向用户展示发送邮件活动。使用访问令牌，您的应用可以对 Microsoft Graph API 提出身份验证请求。

例如，UnifiedApiHelper.cs 文件中包含的代码可以：

1)  获取当前登录用户的相关信息。``GetUserInfoAsync`` 方法提取一个自变量（访问令牌值）来调用 **https://graph.microsoft.com/v1.0/me**，以便获取当前登录用户的相关信息。

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



2)  构造并发送登录用户想通过电子邮件方式发送的邮件。``SendMessageAsync`` 方法将访问令牌值用作一个自变量，构造并发送对 **https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail** 资源 URL 提出的 POST 请求。


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


``MessageController.cs `` 文件中包含用于管理电子邮件的代码。 例如，“**发送邮件**”按钮。在用户单击“**发送邮件**”按钮时，``SendMessageSubmit `` 方法会发送邮件。


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


``CreateEmailObject`` 方法创建采用 POST 主体所需的请求格式/数据协定的电子邮件对象：


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

另一项任务是构造有效的 JSON 邮件字符串，然后使用 HTTP POST 请求将它发送至 ``https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail`` 终结点。由于电子邮件正文将作为 HTML 文档发送，因此请求将电子邮件的 ``ContentType`` 值设置为 HTML，并针对 HTTP POST 请求将内容编码为 JSON。UnifiedApiMessageModels.cs 文件中包含此应用与 Office 365 统一 API 服务器之间的数据或架构协定。



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
## 断开会话连接

单击“发送邮件”页中的“**断开连接**”时，用户会退出会话。 为此，代码通过
* 清除本地会话
* ，将浏览器重定向到退出终结点（以便 Azure 可以清除它自己的 Cookie）

**退出**方法（见 HomeController.cs 文件）展示了这是如何实现的。


```c#
        public ActionResult Logout()
        {
            Session.Clear();
            return Redirect(Settings.LogoutAuthority + logoutRedirectUri.ToString());
        }

```

##后续步骤
Microsoft Graph API 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 API 参考，了解你还可以使用 Microsoft Graph API 完成什么任务。
在 [GitHub](http://aka.ms/aspnetgraphsamples) 上了解我们其他的 ASP.NET 示例。


