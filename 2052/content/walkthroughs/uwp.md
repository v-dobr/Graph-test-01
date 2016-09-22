# 在通用 Windows 10 应用中调用 Microsoft Graph

在本文中，我们将了解从 Azure Active Directory (AD) 中获取访问令牌，以及调用 Microsoft Graph 至少需要完成的任务。我们使用[适用于 UWP 的 Office 365 Connect 示例（使用 Microsoft Graph）](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample)中的代码，说明需要在你的应用中实现的主要概念。

## 示例用户界面

该示例包含一个非常简单的用户界面，其中包括一个顶部命令栏、一个**连接按钮**、一个**发送邮件**按钮，以及一个会自动填充登录用户电子邮件地址的文本框但是该文本框可以进行编辑。命令栏还包含一个使开发人员能够找到应用重定向 URI 的按钮。

当用户未连接时，**发送邮件**按钮处于禁用状态：

![屏幕显示连接按钮已启用和发送邮件按钮已禁用](images/SignedOut.png)

当用户已连接时，顶部命令栏包含一个断开连接按钮。

![屏幕显示已连接用户的电子邮件地址和发送邮件按钮已启用](images/SignedIn.png)

所有示例的 UI 字符串存储在 Assets 文件夹中的 Resources.resw 文件中。

## 注册应用
 
Windows 10 为每个应用提供了唯一的 URI，确保发送到此 URI 的邮件只发送到该应用。注册应用之前，您需要创建应用并找到该系统生成的 URI。在本示例中，你将会在 AuthenticationHelper.cs 文件中找到此方法：

```c#
        public static string GetAppRedirectURI()
        {
            // Windows 10 universal apps require redirect URI in the format below. Add a breakpoint to this line and run the app before you register it, so that
            // you can supply the correct redirect URI value.
            return string.Format("ms-appx-web://microsoft.aad.brokerplugin/{0}", WebAuthenticationBroker.GetCurrentApplicationCallbackUri().Host).ToUpper();
        }
```

该方法通过**复制重定向 URI**按 钮在示例中触发，但是你也可以按照 [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM) 示例中的模式触发，其中字符串在 MainPage 类声明中定义，你可以通过使用 Visual Studio 调试器获取它。 

按照示例自述文件的[注册和配置应用](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample#register)中介绍的步骤进行操作，以便在获取重定向 URI 值后注册应用。

配置应用进行身份验证时，你将需要 Azure 应用程序**配置**页面中的客户端 ID 值。

## 连接到 Microsoft Graph

该示例使用本机 Windows 10 WebAccountManager API 对用户进行身份验证。它遵循与在[Develop Windows Universal Apps with Azure AD and the Windows 10 Identity API](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx)（借助 Azure AD 和 Windows 10 Identity API 开发 Windows 通用应用）博客文章中介绍并在 [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM) 示例中演示的模式类似的模式。

App.xaml 文件中包含键/值对，你的应用将需要该键/值对以便对用户进行身份验证并授权应用发送电子邮件：

```xml
    <Application.Resources>
        <!-- Add your client id here. -->
        <x:String x:Key="ida:ClientID"><your client id></x:String>
        <x:String x:Key="ida:AADInstance">https://login.microsoftonline.com/</x:String>
        <!-- Add your developer tenant domain here. -->
        <x:String x:Key="ida:Domain">yourtenant.onmicrosoft.com</x:String>
    </Application.Resources>
```

当将应用注册为 **ida:ClientID** 键的值时，请添加你获取的客户端 ID 值。更改 **ida:Domain** 键的值，以便与你的 Office 365 租户匹配。

AuthenticationHelper.cs 文件中包含所有身份验证代码，以及存储用户信息并仅在用户与应用断开连接时强制进行身份验证的其他逻辑。

当用户进行身份验证并且之后应用每次调用 Microsoft Graph 时，都会运行该文件中定义的 ``GetTokenHelperAsync`` 方法。其首要任务是找到 Azure AD 帐户提供程序：

```c#
           aadAccountProvider = await WebAuthenticationCoreManager.FindAccountProviderAsync("https://login.microsoft.com", authority);
```

``authority``值是通过 App.xaml 文件中存储的两个值构造的串联字符串：**ida:AADInstance** 键值加 **ida:Domain** 键值。这将创建一个特定于租户的颁发机构。如果你想要你的应用在任何 Azure AD 租户上运行，还可以使用字符串“organizations”。

应用对用户进行身份验证后，会将用户 ID 值存储在 ``ApplicationData.Current.RoamingSettings`` 中。``GetTokenHelperAsync`` 方法首先检查是否存在此值，如果存在，它会尝试自动进行身份验证：

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

该应用使用 Microsoft Graph 终结点 -- **https://graph.microsoft.com/** -- 作为资源值。当它构建 ``WebTokenRequest`` 时，将会使用你添加到 App.xaml 文件中的客户端 ID 值。由于应用知道用户 ID 且用户没有断开连接，因此 WebAccountManager API 可以找到用户帐户并将其传递到令牌请求。``WebAuthenticationCoreManager.RequestTokenAsync`` 方法将返回一个访问令牌，其分配有适当的权限。

如果应用在漫游设置中未为 ``userID`` 找到任何值，它会构建 ``WebTokenRequest``，强制用户通过 UI 进行身份验证：

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

如果尝试检索令牌成功，``GetTokenHelperAsync`` 方法最后会将重要的用户信息存储在漫游设置中，然后返回令牌值。否则，它确保漫游设置为 null 并返回 null 值。

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

## 使用 Microsoft Graph 发送电子邮件

MailHelper.cs 文件中包含构建并发送电子邮件的代码。它包含一个方法 -- ``ComposeAndSendMailAsync`` -- 该方法构造 POST 请求并将其发送到 **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail** 终结点。 

``ComposeAndSendMailAsync`` 方法采用三个字符串值 -- ``subject``、``bodyContent`` 和 ``recipients`` -- 通过 MainPage.xaml.cs 文件将这些值传递给它。``subject`` 和 ``bodyContent`` 字符串随所有其他 UI 字符串存储在 Resources.resw 文件中。``recipients`` 字符串来自应用界面中的地址框中。 

由于用户可以传递多个地址，因此第一项任务是将 ``recipients`` 字符串拆分为一组 EmailAddress 对象，然后能够以请求 POST 正文的形式传递这些对象：

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

另一项任务是构建有效的 JSON 邮件对象，然后通过 HTTP POST 请求将它发送至 **me/microsoft.graph.SendMail** 终结点。由于 ``bodyContent`` 字符串是一个 HTML 文档，因此请求将 **ContentType** 值设置为 HTML。另请注意对 ``AuthenticationHelper.GetTokenHelperAsync`` 的调用，以确保我们在请求中传递全新的访问令牌。

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

成功进行 REST 请求后，你即执行了与 Microsoft Graph 进行交互所需的三个步骤：应用注册、用户身份验证以及进行 REST 请求。


<!--## Additional resources

* [Develop Windows Universal Apps with Azure AD and the Windows 10 Identity API](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx)
* [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM)
* [Office Dev Center](http://dev.office.com)-->

