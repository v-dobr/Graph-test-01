# Вызов Microsoft Graph в приложении Python 

В этой статье мы рассмотрим, как подключить приложение к Office 365 и вызвать API Microsoft Graph. Мы объясним основные понятия на примере [приложения Office 365 Connect на Python, использующего Microsoft Graph](https://github.com/microsoftgraph/python3-connect-rest-sample).

![Снимок экрана приложения Python Connect для Office 365](./images/web-screenshot.png)

##  Необходимые компоненты

В этой статье предполагается следующее:

* вы уверенно читаете код Python;
* вы знакомы с понятиями OAuth.

## Обзор

Чтобы вызвать API Microsoft Graph в приложении Python, необходимо выполнить следующие задачи:

1. Зарегистрировать приложение в Azure Active Directory.
2. Перенаправить браузер на страницу входа.
3. Получить код авторизации на странице URL-адреса ответа.
4. Запросить токен доступа из конечной точки выдачи токенов.
5. Использование токена доступа в запросе к API Microsoft Graph 

<!--<a name="register"></a>-->
## Регистрация приложения в Azure Active Directory

Прежде чем начать работу с Office 365, необходимо зарегистрировать свое приложение и настроить разрешения на использование служб Microsoft Graph. 
С помощью [средства регистрации приложений](https://dev.office.com/app-registration) вы можете с легкостью зарегистрировать свое приложение для доступа к рабочей или учебной учетной записи пользователя. Для управления своей учетной записью пользователю необходимо перейти на [портал управления Microsoft Azure](https://manage.windowsazure.com).

Инструкции по регистрации приложения вручную см. в разделе [Регистрация приложения веб-сервера на портале управления Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp). Обратите внимание на следующие рекомендации:

* Обязательно укажите http://127.0.0.1:8000/connect/get_token/ в качестве **URL-адреса для входа**.
* Когда зарегистрируете приложение, [настройте **делегированные разрешения**](https://github.com/microsoftgraph/python3-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), необходимые для приложения Python. Для приложения Connect необходимо разрешение **Отправлять почту как вошедший пользователь**.

Запишите следующие значения на странице **Настройка** приложения Azure, так как они понадобятся вам для настройки потока OAuth в приложении Python:

* идентификатор клиента (уникальный для приложения);
* URL-адрес ответа (http://127.0.0.1:8000/connect/get_token/);
* ключ приложения (уникальный);

<!--<a name="redirect"></a>-->
## Перенаправление браузера на страницу входа

Приложение должно перенаправить браузер на страницу входа, чтобы начать поток OAuth и получить код авторизации. 

В примере Connect приведенный ниже код (расположенный в файле [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) создает URL-адрес, на который приложение должно перенаправлять пользователя, и передается в окно, в котором его можно использовать для перенаправления. 

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
## Получение кода авторизации на странице URL-адреса ответа

После входа пользователя браузер перенаправляется на URL-адрес ответа, функцию ```get_token``` в файле [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), и к строке запроса добавляется код авторизации в виде переменной ```code```. 

Приложение Connect получает код из строки запроса, чтобы затем обменять его на токен доступа.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## Запрос токена доступа из конечной точки выдачи токенов

Используя код авторизации и значения идентификатора клиента, ключа и URL-адреса ответа, полученные из Azure Active Directory, вы можете запросить токен доступа. 

> **Примечание**. В запросе также необходимо указать ресурс. Значение ресурса для Microsoft Graph — `https://graph.microsoft.com`.

Приложение Connect запрашивает токен в функции ```get_token_from_code``` в файле [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py).

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

> **Примечание**. Ответ содержит не только токен доступа, но и другие сведения. Например, приложение может получить токен обновления для запроса новых токенов доступа без повторного входа пользователя.

<!--<a name="request"></a>-->
## Использование токена доступа в запросе к API Microsoft Graph

Используя токен доступа, приложение может отправлять проверенные запросы в API Microsoft Graph. Приложение должно добавлять токен доступа в заголовок **Authorization** каждого запроса.

Приложение Connect отправляет электронную почту, используя конечную точку ```me/microsoft.graph.sendMail``` в API Microsoft Graph. Код содержится в функции ```call_sendMail_endpoint``` в файле [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py). Этот код показывает, как добавить код доступа в заголовок "Authorization".

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Примечание**. В запросе необходимо также отправлять заголовок **Content-Type** со значением, принимаемым API Graph, например `application/json`.

API Microsoft Graph — это функциональный единый API, с помощью которого можно взаимодействовать со все типами данных Майкрософт. Сведения о других возможностях API Microsoft Graph см. в справочнике по API.

<!--
## Additional resources

-  [Office 365 Python Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Python-Microsoft-Graph-Connect)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference]()-->
