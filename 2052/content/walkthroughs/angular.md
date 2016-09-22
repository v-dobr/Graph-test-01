# 在 Angular 应用中调用 Microsoft Graph 

在本文中，我们将了解连接您的应用程序与 Office 365，以及调用 Microsoft Graph API 至少需要完成的任务。我们使用 [Office 365 Angular 连接示例（使用 Microsoft Graph）](https://github.com/microsoftgraph/angular-connect-rest-sample)中的代码，说明需要在您的应用中实现的主要概念。

![Office 365 Angular 连接示例的屏幕截图](./images/web-screenshot.png)

## 先决条件  

若要理解本主题，您需要具备以下条件。

* 您习惯阅读 JavaScript 和 [AngularJS](https://angularjs.org/) 代码。

## 概述

若要调用 Microsoft Graph API，您必须完成以下任务。

1. 在 Azure Active Directory 中注册应用程序
2. 配置 JavaScript (ADAL JS) 的 Azure Active Directory Library
3. 使用 ADAL JS 获取访问令牌
4. 在对 Microsoft Graph API 提出的请求中使用访问令牌

<!--<a name="register"></a>-->
## 在 Azure Active Directory 中注册应用程序

您需要先注册应用程序并设置 Microsoft Graph 服务的使用权限，然后才能开始使用 Office 365。只需单击几下，即可使用[应用程序注册工具](https://dev.office.com/app-registration)将应用程序注册为访问用户的工作或学校帐户。
若要进行管理，您需要转到 [Microsoft Azure 管理门户](https://manage.windowsazure.com)

您也可以参阅[在 Azure 管理门户中注册基于浏览器的 Web 应用](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp)一文，了解关于如何手动注册应用的说明，并注意以下细节：

* 务必指定 http://127.0.0.1:8080/ 作为**登录 URL**。
* 注册应用程序后，[配置**委托的权限**](https://github.com/microsoftgraph/angular-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)（其为 Angular 应用所需）。连接示例需要获得**以登录用户身份发送邮件**的权限。

请记下 Azure 应用程序的“**配置**”页中的下列值，因为你需要使用这些值在 Angular 应用中配置 [ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js)。

* 客户端 ID（您的应用程序专用）
* 回复 URL (http://127.0.0.1:8080/)

<!--<a name="adal"></a>-->
## 配置 JavaScript (ADAL JS) 的 Azure Active Directory Library

[ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) 是一个 JavaScript 库，可对在单页应用程序 (SPA)（如连接示例和令牌管理）中登录 Azure AD 用户，以及其他功能提供完整支持。您的 Angular 应用必须添加和配置此库，才能利用它。

只需使用 Microsoft CDN，即可添加此库及其 Angular 专用模块。

```html
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal.min.js"></script>
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal-angular.min.js"></script>
```

接下来，您需要配置 ADAL JS 服务，无论您在何处配置 Angular 应用的依赖项。连接示例在 [*public/app.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/scripts/app.js) 中完成配置。 

若要配置 ADAL JS，请先通过向模块需要的数组添加 ```AdalAngular``` 来添加对 ADAL 模块的引用，然后将 ```adalAuthenticationServiceProvider``` 传递给 ```config``` 函数。使用 ```init``` 函数配置此库，同时将您的应用程序客户端 ID 和 ```endpoints``` 对象（声明 Angular 应用需要向哪些 API 提出 CORS 请求）传递给它。

```javascript
// Initialize the ADAL provider with your clientID (found in the Azure Management Portal) and 
// the API URL (to enable CORS requests).
adalAuthenticationServiceProvider.init(
  {
    clientId: clientId,
    // The endpoints here are resources for cross origin requests.
    endpoints: {
      'https://graph.microsoft.com': 'https://graph.microsoft.com'
    }
  },
  $httpProvider
);
```

<!--<a name="accessToken"></a>-->
## 使用 ADAL JS 获取访问令牌

您的应用需要将浏览器重定向到登录页，以便用户可以登录并授权您的应用程序访问其数据。连接示例利用 ADAL JS 来处理此任务。 

在应用程序的一个控制器中，首先通过将 ```adalAuthenticationService``` 插入控制器来添加对 ADAL 服务的引用，然后定义一个函数来使用服务的 ```login``` 函数（UI 可以调用此函数）。连接示例在 [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js) 文件中完成此任务。 

```javascript
/**
  * Expose the login method from ADAL to the view.
  */
function connect() {
  adalAuthenticationService.login();
};
```

调用此函数后，应用程序会将用户重定向到登录页。在用户登录和授权应用后，系统会使用 ADAL JS 将检索和存储的查询字符串中的访问令牌将用户返回到应用 。 

<!--<a name="request"></a>-->
## 在对 Microsoft Graph API 提出的请求中使用访问令牌

使用访问令牌，您的应用可以对 Microsoft Graph API 提出身份验证请求。ADAL JS 会自动截获所有 HTTP 请求，并向它们添加访问令牌。因此，在使用此库时，您无需手动设置标头。 

连接示例使用 [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js) 文件的 Microsoft Graph API 中的 ```me/sendMail``` 终结点发送电子邮件。 

Microsoft Graph 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 [API 参考](http://graph.microsoft.io/docs/api-reference/v1.0)，了解您还可以使用 Microsoft Graph API 完成什么任务。

