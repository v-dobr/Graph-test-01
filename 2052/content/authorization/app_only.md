# 在服务或守护程序应用中调用 Microsoft Graph

在本文中，我们将介绍连接您的单租户服务或守护程序应用与 Office 365，以及调用 Microsoft Graph API 之前至少需要完成的任务。

## 概述

若要在服务或守护程序应用中调用 Microsoft Graph API，您必须完成以下任务。

1. 在 Azure Active Directory 中注册应用程序。
2. 从令牌颁发终结点请求获取访问令牌。
3. 在对 Microsoft Graph API 提出的请求中使用访问令牌。

## 在 Azure Active Directory 中注册应用程序

您需要先注册您的应用程序，并设置 Microsoft Graph 服务的使用权限，然后才能开始使用 Office 365。
只需单击几下，即可使用[应用程序注册工具](https://dev.office.com/app-registration)注册您的应用程序。若要进行管理，您需要转到 [Microsoft Azure 管理门户](https://manage.windowsazure.com)。

您也可以参阅[使用 Azure 管理门户注册 Web 服务器应用](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp)部分，了解关于如何手动注册应用的说明，并注意以下细节：

* 注册应用程序后，配置服务或守护程序应用需要的**应用程序权限**。

记下 Azure 应用程序的“配置”页中的下列值，因为您需要使用这些值在服务或守护程序应用中配置 OAuth 流。

* 客户端 ID（您的应用程序专用）
* 应用程序键（您的应用程序专用）
* 你的应用 OAuth 2.0 令牌终结点
  * 在你的应用页面中，单击 Azure 管理门户底部的“*查看终结点*”，找到此值。 终结点看起来像 `https://login.microsoftonline.com/<tenantId>/oauth2/token`。

## 从令牌颁发终结点请求获取访问令牌

与客户端应用不同，服务或守护程序应用不具有用户登录功能，也无法授权您的应用程序。相反，您的应用程序必须实现允许其使用它自己的凭据、客户端 ID 和应用程序键的 OAuth 2.0 客户端凭据授予流，以便在调用 Microsoft Graph 时进行身份验证（而不是模拟用户）。有关身份验证流的详细信息，请参阅[使用客户端凭据的服务到服务调用](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx)。

对令牌颁发终结点提出包含以下参数的 HTTP POST 请求（用您应用的客户端 ID 和应用程序键分别替换 `<clientId>` 和 `<clientSecret>`）。

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

响应将包含访问令牌和过期信息。

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## 在对 Microsoft Graph API 提出的请求中使用访问令牌

使用访问令牌，您的应用可以对 Microsoft Graph API 提出身份验证请求。您的应用必须将访问令牌附加到各个请求的**授权**头中。

例如，如果你在 Azure 管理门户中为服务或守护程序应用选取了“*读取所有用户的完整配置文件*”权限，则服务或守护程序应用可以检索租户中的所有用户。 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```

Microsoft Graph 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 [API 参考](http://graph.microsoft.io/docs/api-reference/v1.0)，了解您还可以使用 Microsoft Graph API 完成什么任务。
