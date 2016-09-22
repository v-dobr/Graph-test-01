# 在 PHP 应用中调用 Microsoft Graph 

在本文中，我们将了解从 Azure Active Directory (AD) 中获取访问令牌，以及调用 Microsoft Graph API 至少需要完成的任务。我们使用 [Office 365 Connect 示例（使用 Microsoft Graph）](https://github.com/microsoftgraph/php-connect-rest-sample)中的代码，说明需要在你的应用中实现的主要概念。

![Office 365 PHP 连接示例的屏幕截图](./images/web-screenshot.png)

## 概述

若要调用 Microsoft Graph API，您的 PHP 应用必须完成以下任务。

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

* 在 PHP 应用中指定一个页面作为步骤 6 中的**登录 URL**。在 Connect 示例中，该页面为 [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php)。
* [配置应用需要的**委托的权限**](https://github.com/microsoftgraph/php-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)。Connect 示例需要获得**以登录用户身份发送邮件**的权限。

请记下 Azure 应用程序的“**配置**”页中的下列值。

* 客户端 ID
* 有效键
* 回复 URL

你需要使用这些值作为应用中 OAuth 流中的参数。

<!--<a name="redirect"/>-->
## 将浏览器重定向到登录页

您的应用程序需要将浏览器重定向到登录页，以获取授权代码，然后继续 OAuth 流。

在 Connect 示例中，重定向浏览器的代码位于 [`AuthenticationManager.connect`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L41) 函数。

```php
// Redirect the browser to the authorization endpoint. Auth endpoint is
// https://login.microsoftonline.com/common/oauth2/authorize
$redirect = Constants::AUTHORITY_URL . Constants::AUTHORIZE_ENDPOINT . 
            '?response_type=code' . 
            '&client_id=' . urlencode(Constants::CLIENT_ID) . 
            '&redirect_uri=' . urlencode(Constants::REDIRECT_URI);
header("Location: {$redirect}");
exit();
```

> **注意：** <br />
> 在将任何输出写入页面之前，你必须发送**位置**标头。

<!--<a name="authcode"/>-->
## 在回复 URL 页面中接收授权代码

用户登录后，该流将浏览器返回到应用中的回复 URL。Azure 将授权代码附加到查询字符串。Connect 示例出于此目的使用 [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php) 页面。

授权代码在 `code` 查询字符串变量中提供。Connect 示例将代码保存至会话变量中供以后使用。

```php
if (isset($_GET['code'])) {
    $_SESSION['code'] =  $_GET['code'];
}
```

<!--<a name="accesstoken"/>-->
## 从令牌终结点请求访问令牌

获取授权代码之后，你可以与从 Azure AD 获取的客户端 ID、密钥以及回复 URL 值一起使用，以请求访问令牌。 

> **注意：** <br />
> 请求还必须指定我们尝试要使用的资源。在 Microsoft Graph API 中，资源值为 `https://graph.microsoft.com`。

Connect 示例使用 [`AuthenticationManager.acquireToken`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L62) 函数中的代码请求令牌。下面是最为相关的代码。

```php
$tokenEndpoint = Constants::AUTHORITY_URL . Constants::TOKEN_ENDPOINT;

// Send a POST request to the token endpoint to retrieve tokens.
// Token endpoint is:
// https://login.microsoftonline.com/common/oauth2/token
$response = RequestManager::sendPostRequest(
    $tokenEndpoint, 
    array(),
    array(
        'client_id' => Constants::CLIENT_ID,
        'client_secret' => Constants::CLIENT_SECRET,
        'code' => $_SESSION['code'],
        'grant_type' => 'authorization_code',
        'redirect_uri' => Constants::REDIRECT_URI,
        'resource' => Constants::RESOURCE_ID
    )

// Store the raw response in JSON format.
$jsonResponse = json_decode($response, true);

// The access token response has the following parameters:
// access_token - The requested access token.
// expires_in - How long the access token is valid.
// expires_on - The time when the access token expires.
// id_token - An unsigned JSON Web Token (JWT).
// refresh_token - An OAuth 2.0 refresh token.
// resource - The App ID URI of the web API (secured resource).
// scope - Impersonation permissions granted to the client application.
// token_type - Indicates the token type value.
foreach ($jsonResponse as $key=>$value) {
    $_SESSION[$key] = $value;
}
```

> **注意：** <br />
> 响应提供了更多信息，不仅仅是访问令牌，例如，你的应用可以获取刷新令牌来请求新的访问令牌，无需用户显式登录。

你的 PHP 应用现在使用会话变量 `access_token` 将已通过身份验证的请求发送到 Microsoft Graph API。

<!--<a name="request"/>-->
## 在对 Microsoft Graph API 提出的请求中使用访问令牌

使用访问令牌，你的应用可以对 Microsoft Graph API 提出身份验证请求。你的应用必须在每个请求的**授权**标头中提供访问令牌。

Connect 示例使用 Microsoft Graph API 中的 **sendMail** 终结点发送电子邮件。代码位于 [`MailManager.sendWelcomeMail`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/MailManager.php#L40) 函数中。这是显示如何发送授权标头中的访问代码的代码。

```php
// Send the email request to the sendmail endpoint, 
// which is in the following URI:
// https://graph.microsoft.com/v1.0/me/microsoft.graph.sendMail
// Note that the access token is attached in the Authorization header
RequestManager::sendPostRequest(
    Constants::RESOURCE_ID . Constants::SENDMAIL_ENDPOINT,
    array(
        'Authorization: Bearer ' . $_SESSION['access_token'],
        'Content-Type: application/json;' . 
                      'odata.metadata=minimal;' .
                      'odata.streaming=true'
    ),
    $email
);
```

> **注意：** <br />
> 该请求还必须发送包含 Microsoft Graph API 接受的值的 **Content-Type** 标头，例如 `application/json;odata.metadata=minimal;odata.streaming=true`。

Microsoft Graph API 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 API 参考，了解您还可以使用 Microsoft Graph API 完成什么任务。

<!--## Additional resources

-  [Office 365 PHP Connect sample using Microsoft Graph API](https://github.com/OfficeDev/O365-PHP-Unified-API-Connect)-->
