# Вызов Microsoft Graph в приложении PHP 

В этой статье мы рассмотрим, как получить токен доступа из Azure Active Directory (AD) и вызвать API Microsoft Graph. Мы объясним основные понятия на примере [приложения Office 365 Connect на PHP, использующего Microsoft Graph](https://github.com/microsoftgraph/php-connect-rest-sample).

![Снимок экрана с примером приложения на PHP, подключающегося к Office 365](./images/web-screenshot.png)

## Обзор

Чтобы вызвать API Microsoft Graph в приложении PHP, необходимо выполнить следующие задачи:

1. Зарегистрировать приложение в Azure Active Directory.
2. Перенаправить браузер на страницу входа.
3. Получить код авторизации на странице URL-адреса ответа.
4. Запросить токен доступа из конечной точки токена.
5. Использование токена доступа в запросе к API Microsoft Graph

<!--<a name="register"/>-->
## Регистрация приложения в Azure Active Directory

Прежде чем начать работу с Office 365, необходимо зарегистрировать свое приложение и настроить разрешения на использование служб Microsoft Graph. С помощью [средства регистрации приложений](https://dev.office.com/app-registration) вы можете с легкостью зарегистрировать свое приложение для доступа к рабочей или учебной учетной записи пользователя. 
Для управления своей учетной записью пользователю необходимо перейти на [портал управления Microsoft Azure](https://manage.windowsazure.com).

Инструкции по регистрации приложения вручную см. в разделе [Регистрация приложения веб-сервера на портале управления Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp). Обратите внимание на следующие рекомендации:

* Укажите страницу в приложении PHP в качестве **URL-адреса для входа** на шаге 6. В примере Connect это страница [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php).
* [Настройте **делегированные разрешения**](https://github.com/microsoftgraph/php-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), необходимые приложению. Для приложения Connect требуется разрешение **Отправлять почту как вошедший пользователь**.

Запишите следующие значения со страницы **Настройка** приложения Azure:

* Идентификатор клиента
* действительный ключ;
* URL-адрес ответа.

Эти значения необходимы для настройки потока OAuth в приложении.

<!--<a name="redirect"/>-->
## Перенаправление браузера на страницу входа

Приложение должно перенаправить браузер на страницу входа, чтобы получить код авторизации и продолжить поток OAuth.

Приложение Connect перенаправляет браузер с помощью кода в функции [`AuthenticationManager.connect`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L41).

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

> **Примечание.** <br />
> Необходимо отправить заголовок **Location**, прежде чем записывать выходные данные на странице.

<!--<a name="authcode"/>-->
## Получение кода авторизации на странице URL-адреса ответа

После входа пользователя браузер перенаправляется на страницу URL-адреса ответа в приложении. Azure добавляет код авторизации в строку запроса. В приложении Connect для этого используется страница [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php).

Код авторизации указан в переменной строки запроса `code`. Приложение Connect сохраняет код в переменной сеанса для дальнейшего использования.

```php
if (isset($_GET['code'])) {
    $_SESSION['code'] =  $_GET['code'];
}
```

<!--<a name="accesstoken"/>-->
## Запрос токена доступа из конечной точки токена

Используя код авторизации и значения идентификатора клиента, ключа и URL-адреса ответа, полученные из Azure AD, вы можете запросить токен доступа. 

> **Примечание.** <br />
> В запросе необходимо также указать ресурс. Для API Microsoft Graph значение ресурса — `https://graph.microsoft.com`.

Приложение Connect запрашивает токен с помощью кода в функции [`AuthenticationManager.acquireToken`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L62). Ниже приведен наиболее подходящий код.

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

> **Примечание.** <br />
> Ответ содержит не только токен доступа, но и другие сведения. Например, приложение может получить токен обновления для запроса новых токенов доступа без входа пользователя.

Приложение PHP теперь может отправлять проверенные запросы в API Microsoft Graph, используя переменную сеанса `access_token`.

<!--<a name="request"/>-->
## Использование токена доступа в запросе к API Microsoft Graph

Используя токен доступа, приложение может отправлять проверенные запросы в API Microsoft Graph. Приложение должно указывать токен доступа в заголовке **Authorization** каждого запроса.

Приложение Connect отправляет электронную почту, используя конечную точку **sendMail** в API Microsoft Graph. Код находится в функции [`MailManager.sendWelcomeMail`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/MailManager.php#L40). Этот код показывает, как отправить код доступа в заголовке Authorization.

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

> **Примечание.** <br />
> В запросе необходимо также отправить заголовок **Content-Type** со значением, принимаемым API Microsoft Graph, например `application/json;odata.metadata=minimal;odata.streaming=true`.

API Microsoft Graph — это функциональный единый API, с помощью которого можно взаимодействовать со все типами данных Майкрософт. Сведения о других возможностях API Microsoft Graph см. в справочнике по API.

<!--## Additional resources

-  [Office 365 PHP Connect sample using Microsoft Graph API](https://github.com/OfficeDev/O365-PHP-Unified-API-Connect)-->
