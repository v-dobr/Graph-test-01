# 在 Python 应用中调用 Microsoft Graph 

在本文中，我们将了解连接你的应用程序与 Office 365，以及调用 Microsoft Graph API 至少需要完成的任务。我们使用 [Office 365 Python Connect 示例（使用 Microsoft Graph）](https://github.com/microsoftgraph/python3-connect-rest-sample)中的代码，说明需要在你的应用中实现的主要概念。

![Office 365 Python 连接示例的屏幕截图](./images/web-screenshot.png)

##  先决条件

若要理解本主题，您需要具备以下条件：

* 您习惯阅读 Python 代码。
* 您熟悉 OAuth 的概念。

## 概述

若要调用 Microsoft Graph API，您的 Python 应用必须完成以下任务。

1. 在 Azure Active Directory 中注册应用程序
2. 将浏览器重定向到登录页
3. 在回复 URL 页面中接收授权代码
4. 从令牌颁发终结点请求访问令牌
5. 在对 Microsoft Graph API 提出的请求中使用访问令牌 

<!--<a name="register"></a>-->
## 在 Azure Active Directory 中注册应用程序

您需要先注册应用程序并设置 Microsoft Graph 服务的使用权限，然后才能开始使用 Office 365。只需单击几下，即可使用[应用程序注册工具](https://dev.office.com/app-registration)将应用程序注册为访问用户的工作或学校帐户。
若要进行管理，您需要转到 [Microsoft Azure 管理门户](https://manage.windowsazure.com)

您也可以参阅[使用 Azure 管理门户注册 Web 服务器应用](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp)部分，了解关于如何手动注册应用的说明，并注意以下细节：

* 务必指定 http://127.0.0.1:8000/connect/get_token/ 作为**登录 URL**。
* 注册应用程序后，[配置**委托的权限**](https://github.com/microsoftgraph/python3-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)（其为 Python 应用所需）。Connect 示例需要获得**以登录用户身份发送邮件**的权限。

请记下 Azure 应用程序的**配置**页中的下列值，因为你需要使用这些值在 Python 应用中配置 OAuth 流。

* 客户端 ID（您的应用程序专用）
* 回复 URL (http://127.0.0.1:8000/connect/get_token/)
* 应用程序键（您的应用程序专用）

<!--<a name="redirect"></a>-->
## 将浏览器重定向到登录页

你的应用程序需要将浏览器重定向到登录页开始 OAuth 流程并获得授权代码。 

在 Connect 示例中，以下代码（位于 [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)）将生成应用需要将用户重定向到的 URL 并发送到它可用于重定向的视图。 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## 在回复 URL 页面中接收授权代码

用户登录后，浏览器将重定向到你的回复 URL，[*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py) 中的 ```get_token``` 函数和授权代码附加到查询字符串作为 ```code``` 变量。 

Connect 示例从查询字符串获取代码，以便可以用其交换访问令牌。

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## 从令牌颁发终结点请求获取访问令牌

获取授权代码之后，你可以与从 Azure Active Directory 获取的客户端 ID、密钥以及回复 URL 值一起使用，以请求访问令牌。 

> **注意** 请求还必须指定你尝试要使用的资源。在 Microsoft Graph 情况中，资源值为 `https://graph.microsoft.com`。

Connect 示例请求 [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py) 文件中的 ```get_token_from_code``` 函数中的令牌。

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **注意** 响应提供了更多信息，不仅仅是访问令牌。例如，你的应用将获取一个刷新令牌来请求新的访问令牌，无需用户再次显式登录。

<!--<a name="request"></a>-->
## 在对 Microsoft Graph API 提出的请求中使用访问令牌

使用访问令牌，您的应用可以对 Microsoft Graph API 提出身份验证请求。您的应用必须将访问令牌附加到各个请求的**授权**头中。

Connect 示例使用 Microsoft Graph API 中的 ```me/microsoft.graph.sendMail``` 终结点发送电子邮件。代码位于 [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py) 文件的 ```call_sendMail_endpoint``` 函数中。这是显示如何将访问代码附加到授权标头中的代码。

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **注意** 该请求还必须发送包含 Microsoft Graph API 接受的值的 **Content-Type** 标头，例如 `application/json`。

Microsoft Graph API 的功能非常强大，统一了可用于与各种 Microsoft 数据进行交互的 API。请查看 API 参考，了解您还可以使用 Microsoft Graph API 完成什么任务。

<!--
## Additional resources

-  [Office 365 Python Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Python-Microsoft-Graph-Connect)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference]()-->
