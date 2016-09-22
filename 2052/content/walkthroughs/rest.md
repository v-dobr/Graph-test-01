# 开始使用 Microsoft Graph 和 REST

本文介绍如何调用 Microsoft Graph 来检索 Office 365 和 Outlook.com 中的电子邮件。 本文重点介绍 OAuth 和 REST 请求及响应。 它包括应用在进行身份验证和检索邮件时使用的请求和响应的顺序。

## 使用 OAuth 2.0 进行身份验证

为了调用 Microsoft Graph，应用需要从 Azure Active Directory (Azure AD) 获取访问令牌。 在以下示例中，应用会实现授权代码授予流，以便从遵循标准 [OAuth 2.0 协议](http://tools.ietf.org/html/rfc6749) 的 Azure AD 获取访问令牌。

### 注册应用

应用注册选项目前有两个：

  1. 使用仅支持 Office 365 商业用户和工作或学校帐号的模型进行注册。
 
  这种模型只适用于 Office 365 商业产品。注册完应用后，你便可以通过 [Azure 管理门户](https://manage.windowsazure.com)对其进行管理。

  2. 使用适用于消费者和商业 Office 365 服务的最新功能进行注册（我们将其称为 Auth endpoint v2.0）。
 
  现在，适用于工作或学校帐户和个人帐户的单一身份验证服务已经可用。此模型提供适用于工作和学校 (Azure AD) 身份信息以及个人 (Microsoft) 身份信息的单一身份验证服务。现在，你只需在应用中实现一个身份验证流，即可让用户使用工作或学校帐户（如 Office 365 或 OneDrive for Business）或个人帐户（如 Outlook.com 或 OneDrive）。
   
使用[应用程序注册门户](https://apps.dev.microsoft.com/)注册你的应用并提供对工作和学校帐号以及个人帐号的支持。

请注意 v2.0 终结点正逐渐发展成为涵盖从早期验证终结点至今的所有方案。若要选择适合你的，请阅读[本文](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

注册后，你便会获得客户端 ID 和密码。授权代码授予流中会使用这些值。

本文档的其余部分假定在 v2.0 模型上进行注册。有关 v2.0 终结点中支持的流的完整指南，请参阅[本文](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/)，有关授权代码授予流的完整指南，请参阅[本文](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/)

### 获取授权代码

授权代码授予流的第一步是获取授权代码。当用户登录并同意应用所需的访问级别时，授权服务器向应用返回授权代码。

首先，应用会构造用户的登录 URL。必须在浏览器中打开此 URL，这样用户才能登录并给予同意。

登录基 URL 如下所示：`https://login.microsoftonline.com/common/oauth2/v2.0/authorize`。

应用会将查询参数附加到此基 URL，以便授权服务器知道哪个应用正在请求登录及其正在请求获取的权限。

- `client_id` - 通过注册应用生成的客户端 ID，以便 Azure AD 知道哪个应用正在请求登录。
- `redirect_uri` - 在用户已向应用授予同意后，Azure 会重定向到的位置。此值必须对应于注册应用时所使用的**重定向 URI** 值。
- `response_type` - 应用预期的响应类型。此值是适用于授权代码授予流的 `code`。
- `scope` - 应用请求的作用域的列表（用空格分隔）。详细信息在[本文](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/)中
- `state` - 请求中包含的值也将在令牌响应中返回。

例如，需要获取邮件读取访问权限的应用程序的请求 URL 如下所示。

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<CLIENT ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
```

接下来，将用户重定向到登录 URL。用户会看到登录屏幕（其中显示应用名称）。登录后，用户会看到应用需要获取的权限列表，并被要求确认是允许还是拒绝。假设用户允许所需的访问权限，浏览器会重定向到初始请求中指定的重定向 URI（查询字符串中包含授权代码）。

```http
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

如果你也使用 OpenId Connect 进行单一登录，则需要其他参数，有关详细信息，请参阅[本文](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/)。 

下一步是交换返回的授权代码，以获得访问令牌。

### 获取访问令牌

为了获取访问令牌，应用会使用以下参数将表单编码参数发布到令牌请求 URL (`https://login.microsoftonline.com/common/oauth2/v2.0/token`) 中。

- `client_id`：通过注册应用生成的客户端 ID。
- `client_secret`：通过注册应用生成的客户端密码。
- `code`：在上一步中获得的授权代码。
- `redirect_uri`：此值必须与授权代码请求中使用的值相同。
- `grant_type`：应用正在使用的授予类型。此值是适用于授权代码授予流的 `code`。
- `scope` - 应用请求的作用域的列表（用空格分隔）。详细信息在[本文](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/)中

我们应用程序的请求 URL（使用上一步中的代码）如下所示。

```http
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
  &client\_id=<CLIENT ID>
  &client\_secret=<CLIENT SECRET>
}
```

服务器响应的是包括访问令牌的 JSON 有效负载。

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

访问令牌位于 JSON 有效负载的 `access_token` 字段中。在对 API 执行 REST 调用时，应用使用此值来设置授权标头。

## 调用 Microsoft Graph

在获得访问令牌后，应用便可以调用 Microsoft Graph。 由于此示例应用是要检索邮件，因此它会对 `https://graph.microsoft.com/v1.0/me/messages` 终结点使用 HTTP GET 请求。

### 优化请求

应用可以使用 OData 查询参数来控制 GET 请求的行为。 建议应用使用这些参数来限制返回的结果数，以及针对每一项返回的字段。 

该示例应用会在表格中显示邮件，此表会显示主题、发件人以及邮件接收日期和时间。 此表最多显示 25 行，并进行排序，将最近收到的邮件置顶。 该应用使用以下查询参数来获得这些结果。

- `$select` - 仅指定 `subject`、`sender` 和 `dateTimeReceived` 字段。
- `$top` - 最多指定 25 个项。
- `$orderby` - 按 `dateTimeReceived` 字段对结果进行排序。

这会生成以下请求。

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

既然你已经了解如何调用 Microsoft Graph，现在就可以使用 API 参考来构造应用需要执行的其他任何种类调用。 不过，请注意，应用需要获得应用注册时配置的适当权限，才能执行调用。


