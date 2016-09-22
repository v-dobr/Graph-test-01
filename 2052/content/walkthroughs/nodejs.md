# 使用 Node.js 应用调用 Microsoft Graph

在本文中，我们将了解连接你的应用程序与 Office 365，以及调用 Microsoft Graph API 至少需要完成的任务。我们使用 [Office 365 Node.js Connect 示例（使用 Microsoft Graph）](https://github.com/microsoftgraph/nodejs-connect-rest-sample)中的代码，说明需要在你的应用中实现的主要概念。

![Office 365 Node.js Connect 示例的屏幕截图](./images/web-screenshot.png)

## 概述

若要调用 Microsoft Graph API，您的 Web 应用必须完成以下任务。

1. 在 Azure Active Directory 中注册应用程序 
2. 为节点安装 Azure Active Directory Client Library
3. 将浏览器重定向到登录页
4. 在回复 URL 页面中接收授权代码
5. 使用 `adal-node` 请求访问令牌
6. 对 Microsoft Graph API 提出请求

<!--<a name="register"/>-->
## 在 Azure Active Directory 中注册应用程序

您需要先注册应用程序并设置 Microsoft Graph 服务的使用权限，然后才能开始使用 Office 365。只需单击几下，即可使用[应用程序注册工具](https://dev.office.com/app-registration)将应用程序注册为访问用户的工作或学校帐户。
若要进行管理，您需要转到 [Microsoft Azure 管理门户](https://manage.windowsazure.com)

您也可以参阅[使用 Azure 管理门户注册 Web 服务器应用](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp)部分，了解关于如何手动注册应用的说明，并注意以下细节：

* 在 Node.js 应用中指定一个页面作为步骤 6 中的**登录 URL**。在 Connect 示例中，URL 是 http://localhost:8080/login，它映射到 [/login](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L33) 路由。
* [配置应用需要的**委托的权限**](https://github.com/microsoftgraph/nodejs-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)。 连接示例需要获得**以登录用户身份发送邮件**的权限。

请记下 Azure 应用程序的“**配置**”页中的下列值。

* 客户端 ID
* 有效键
* 回复 URL

你需要使用这些值作为应用中 OAuth 流中的参数。

<!--<a name="adal">-->
## 为节点安装 Azure Active Directory Client Library

Node.js 库的 ADAL 使得 Node.js 应用程序能够轻松进行 AAD 身份验证以便访问受 AAD 保护的 Web 资源。
若要将 adal 节点添加到现有 `package.json`，请在首选终端中输入以下内容。

`npm install adal-node --save`

有关 adal 节点客户端库的详细信息，请参阅其在 [npm](https://www.npmjs.com/package/adal-node) 上的包信息。
有关问题、源代码以及最新和即将发布的功能和修复程序，请参阅 [Github](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs) 上 adal 节点的项目。

<!--<a name="redirect"/>-->
## 将浏览器重定向到登录页

你的应用需要将浏览器重定向到登录页来获取授权代码并继续 OAuth 2.0 流。

在连接示例中，[`authHelper.js#getAuthUrl`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js#L17) 中的 URL 通过客户端 `onclick` 事件由 [`login.hbs#login`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/views/login.hbs#L2) 函数重定向。

**authHelper.js#getAuthUrl**
```javascript
/**
 * Generate a fully formed uri to use for authentication based on the supplied resource argument
 * @return {string} a fully formed uri with which authentcation can be completed
 */
function getAuthUrl() {
    return credentials.authority + "/oauth2/authorize" +
        "?client_id=" + credentials.client_id +
        "&response_type=code" +
        "&redirect_uri=" + credentials.redirect_uri;
};
```

**login.hbs#login**
```javascript
function login() {
    window.location = '{{auth_url}}'.replace(/&amp;/g, '&'); // transform HTML special char from .hbs template rendering
}
```

<!--<a name="authcode"/>-->
## 在回复 URL 页面中接收授权代码

用户登录后，流会将浏览器返回到您应用中的回复 URL。 授权代码在 `code` 查询字符串变量中提供。

```javascript
router.get('/<application reply url>', function (req, res, next) {
  var authCode = req.query.code;
  // your router's implementation
});
```

请参阅 Connect 示例中的 [相关代码](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L34)

<!--<a name="accesstoken"/>-->
## 使用 `adal-node` 请求访问令牌

现在，我们已经进行 Azure Active Directory 身份验证，我们的下一步是通过 adal 节点获取访问令牌。完成之后，我们将准备对 Microsoft Graph API 提出 REST 请求。

为了请求访问令牌，adal 节点提供了两个回调函数。

|                          函数                         |                                      参数                                      | 说明                                                                                             |
|:---------------------------------------------------------:|:--------------------------------------------------------------------------------:|---------------------------------------------------------------------------------------------------------|
| `AuthenticationContext.acquireTokenWithAuthorizationCode` | `authCode`, `redirect_uri`, `resource`, `client_id`, `client_secret`, `callback` | 根据在登录过程中返回的授权代码为指定资源提供访问令牌 |
| `AuthenticationContext.acquireTokenWithRefreshToken`      | `token`, `client_id`, `client_secret`, `resource`, `callback`                    | 基于刷新令牌为指定资源提供访问令牌                             |

在连接的示例中，可以通过 [`authHelper.js`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js) 路由请求，以便可以添加 `client_id` 和 `client_secret`。

```javascript
// The application registration (must match Azure AD config)
var credentials = {
    authority: "https://login.microsoftonline.com/common",
    client_id: "<your client id here>",
    client_secret: "<your client secret>",
    redirect_uri: "http://localhost:8080/login"
};

/**
 * Gets a token for a given resource.
 * @param {string} code An authorization code returned from a client.
 * @param {string} res A URI that identifies the resource for which the token is valid.
 * @param {AcquireTokenCallback} callback The callback function.
 */
function getTokenFromCode(res, code, callback) {
    var authContext = new AuthenticationContext(credentials.authority);
    authContext.acquireTokenWithAuthorizationCode(code, credentials.redirect_uri, res, credentials.client_id, credentials.client_secret, function (err, response) {
        if (err) {
            callback(null);
        }
        else {
            callback(response);
        }
    });
};
```

<!--<a name="request"/>-->
## 对 Microsoft Graph API 提出请求

要确定我们对 Graph API 的请求，我们的请求必须使用 `Authorization` 标头签名，该标头包含我们请求的任何 Web 服务资源的访问令牌。 正确格式的授权标头将包括 adal 节点中的访问令牌，并且将采用以下形式。

`Authorization: Bearer <access token>`

结合使用 `adal-node` 和我们上一节中的身份验证逻辑，我们现在可以使用我们的访问令牌来签署请求。

```javascript
/* GET home page. */
router.get('/<application reply url>', function (req, res, next) {
    var authCode = req.query.code;
    authHelper.getTokenFromCode('https://graph.microsoft.com/', req.query.code, function (token) {
        if (token !== null) {
            // Use this token to sign requests
            var headers = {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer ' + token
                };
            // request implementation...
        } else {
            // error handling
        }
    });
});
```

Microsoft Graph 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 [API 参考](http://graph.microsoft.io/docs/api-reference/v1.0)，了解您还可以使用 Microsoft Graph API 完成什么任务。

<!--## Additional resources

- [Office 365 Node.js Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Nodejs-Unified-API-Connect)-->

