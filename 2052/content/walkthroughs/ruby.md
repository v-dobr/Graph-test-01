# 在 Ruby 应用中调用 Microsoft Graph 

在本文中，我们将了解从 Azure Active Directory (AD) 中获取访问令牌，以及调用 Microsoft Graph API 至少需要完成的任务。我们使用 [Office 365 Ruby Connect 示例（使用 Microsoft Graph）](https://github.com/microsoftgraph/ruby-connect-rest-sample)中的代码，说明需要在你的应用中实现的主要概念。

![Office 365 Ruby Connect 示例的屏幕截图](./images/web-screenshot.png)

## 概述

若要调用 Microsoft Graph API，您的 Ruby 应用必须完成以下任务。

1. 在 Azure Active Directory 中注册应用程序
2. 将浏览器重定向到登录页
3. 在回复 URL 页面中接收授权代码
4. 从令牌终结点请求访问令牌
5. 在对 Microsoft Graph API 提出的请求中使用访问令牌 

<!--<a name="register"/>-->
## 在 Azure Active Directory 中注册应用程序

您需要先注册应用程序并设置 Microsoft Graph 服务的使用权限，然后才能开始使用 Office 365。只需单击几下，即可使用[应用程序注册工具](https://dev.office.com/app-registration)将应用程序注册为访问用户的工作或学校帐户。
若要进行管理，您需要转到 [Microsoft Azure 管理门户](https://manage.windowsazure.com)

您也可以参阅[使用 Azure 管理门户注册 Web 服务器应用](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp)部分，了解关于如何手动注册应用的说明，并注意以下细节：

* 在 Ruby 应用中指定一个路由作为步骤 6 中的**登录 URL**。在 Connect 示例中，这是 [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L41)。
* [配置应用需要的**委托的权限**](https://github.com/microsoftgraph/ruby-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)。Connect 示例需要获得**以登录用户身份发送邮件**的权限。

请记下 Azure 应用程序的“**配置**”页中的下列值。

* 客户端 ID
* 有效键
* 回复 URL

你需要使用这些值作为应用中 OAuth 流中的参数。

<!--<a name="redirect"/>-->
## 将浏览器重定向到登录页

您的应用程序需要将浏览器重定向到登录页，以获取授权代码，然后继续 OAuth 流。

在 Connect 示例中，重定向由 OmniAuth 库处理。我们的应用只是将执行委托给由 OmniAuth 管理的 [`/auth/azureactivedirectory`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L30) 路由。

<!--<a name="authcode"/>-->
## 在回复 URL 页面中接收授权代码

用户登录后，该流将浏览器返回到应用中的回复 URL。Azure 将授权代码附加到查询字符串。Connect 示例将 [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L38) 路由用于此目的。

授权代码在 `code` 查询字符串变量中提供。Connect 示例将代码保存至本地变量中供以后使用。

```ruby
code = params[:code]
```

<!--<a name="accesstoken"/>-->
## 从令牌终结点请求访问令牌

获取授权代码之后，你可以与从 Azure AD 获取的客户端 ID、密钥以及回复 URL 值一起使用，以请求访问令牌。 

> **注意：** <br />
请求还必须指定我们尝试要使用的资源。在 Microsoft Graph 中，资源值为 `https://graph.microsoft.com`。

同样，Connect 示例将此任务委托给 OmniAuth 库。[`acquire_access_token`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L65) 函数调用该库并将上一部分中保存的身份验证代码与回复 URL、客户端 ID、客户端密码以及资源 ID 一起传递。

```ruby
def acquire_access_token(auth_code, reply_url)
    AUTH_CTX.acquire_token_with_authorization_code(
                  auth_code,
                  reply_url,
                  CLIENT_CRED,
                  GRAPH_RESOURCE)
end
```

> **注意：** <br />
> 客户端 ID 和客户端密码在上一代码段中的 `CLIENT_CRED` 参数中提供。

<!--<a name="request"/>-->
## 在对 Microsoft Graph API 提出的请求中使用访问令牌

使用访问令牌，你的应用可以对 Microsoft Graph API 提出身份验证请求。你的应用必须在每个请求的**授权**标头中提供访问令牌。

Connect 示例使用 Microsoft Graph API 中的 **sendMail** 终结点发送电子邮件。代码位于 [`send_mail`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L82) 函数中。这是显示如何发送授权标头中的访问代码的代码。

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

> **注意：** <br />
> 该请求还必须发送包含 Microsoft Graph API 接受的值的 **Content-Type** 标头，例如 `application/json;odata.metadata=minimal;odata.streaming=true`。

Microsoft Graph API 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 API 参考，了解您还可以使用 Microsoft Graph API 完成什么任务。

<!--## Additional resources

-  [Office 365 Ruby Connect sample using Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference](http://graph.microsoft.io/en-us/docs)-->
