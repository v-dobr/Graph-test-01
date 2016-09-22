# Вызов Microsoft Graph с помощью приложения Node.js

В этой статье мы рассмотрим, как подключить приложение к Office 365 и вызвать API Microsoft Graph. Мы объясним основные понятия на примере [приложения Office 365 Connect для Node.js, использующего Microsoft Graph](https://github.com/microsoftgraph/nodejs-connect-rest-sample).

![Снимок экрана с примером приложения Node.js, подключающегося к Office 365](./images/web-screenshot.png)

## Обзор

Чтобы вызвать API Microsoft Graph в веб-приложении, необходимо выполнить следующие задачи:

1. Зарегистрировать приложение в Azure Active Directory. 
2. Установить клиентскую библиотеку Azure Active Directory для платформы Node.
3. Перенаправить браузер на страницу входа.
4. Получить код авторизации на странице URL-адреса ответа.
5. Запросить маркер доступа с помощью библиотеки `adal-node`.
6. Отправить запрос в API Microsoft Graph

<!--<a name="register"/>-->
## Зарегистрировать приложение в Azure Active Directory

Прежде чем начать работу с Office 365, необходимо зарегистрировать свое приложение и настроить разрешения на использование служб Microsoft Graph. 
С помощью [средства регистрации приложений](https://dev.office.com/app-registration) вы можете с легкостью зарегистрировать свое приложение для доступа к рабочей или учебной учетной записи пользователя. Для управления своей учетной записью пользователю необходимо перейти на [портал управления Microsoft Azure](https://manage.windowsazure.com).

Инструкции по регистрации приложения вручную см. в разделе [Регистрация приложения веб-сервера на портале управления Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp). Обратите внимание на следующие рекомендации:

* Укажите страницу в приложении Node.js в качестве **URL-адреса для входа** на шаге 6. В приложении Connect используется URL-адрес http://localhost:8080/login, которой сопоставляется с маршрутом [/login](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L33).
* [Настройте **делегированные разрешения**](https://github.com/microsoftgraph/nodejs-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), необходимые приложению. Для приложения Connect требуется разрешение **Отправлять почту как вошедший пользователь**.

Запишите следующие значения со страницы **Настройка** приложения Azure:

* Идентификатор клиента
* действительный ключ;
* URL-адрес ответа.

Эти значения необходимы для настройки потока OAuth в приложении.

<!--<a name="adal">-->
## Установка клиентской библиотеки Azure Active Directory для платформы Node

Библиотека ADAL для Node.js облегчает проверку подлинности приложений Node.js в службе AAD для доступа к защищенным веб-ресурсам AAD.
Чтобы добавить библиотеку adal-node в существующий файл `package.json`, введите указанные ниже сведения в выбранном вами терминале.

`npm install adal-node --save`

Дополнительные сведения о клиентской библиотеке adal-node см. на сайте [npm](https://www.npmjs.com/package/adal-node). Проблемы, исходный код, а также сведения о новых функциях и исправлениях см. 
в разделе, посвященном проекту adal-node, на сайте [Github](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs).

<!--<a name="redirect"/>-->
## Перенаправление браузера на страницу входа

Приложение должно перенаправить браузер на страницу входа, чтобы получить код авторизации и продолжить поток OAuth 2.0.

В приложении Connect URL-адрес проверки подлинности из [`authHelper.js#getAuthUrl`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js#L17) перенаправляется функцией [`login.hbs#login`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/views/login.hbs#L2) через событие `onclick` на стороне клиента.

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
## Получение кода авторизации на странице URL-адреса ответа

После входа пользователь перенаправляется на URL-адрес ответа в приложении. Код авторизации указан в переменной строки запроса `code`.

```javascript
router.get('/<application reply url>', function (req, res, next) {
  var authCode = req.query.code;
  // your router's implementation
});
```

[Соответствующий код](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L34) см. в примере приложения Connect

<!--<a name="accesstoken"/>-->
## Запросить маркер доступа с помощью библиотеки `adal-node`.

После проверки подлинности в службе Azure Active Directory нужно получить токен доступа через библиотеку adal-node. После этого мы сможем отправлять запросы REST в API Microsoft Graph.

Для запроса токена доступа в библиотеке adal-node используются две функции обратного вызова.

|                          Функция                         |                                      Параметры                                      | Описание                                                                                             |
|:---------------------------------------------------------:|:--------------------------------------------------------------------------------:|---------------------------------------------------------------------------------------------------------|
| `AuthenticationContext.acquireTokenWithAuthorizationCode` | `authCode`, `redirect_uri`, `resource`, `client_id`, `client_secret`, `callback` | Предоставляет маркер доступа для указанного ресурса на основе кода авторизации, полученного при входе. |
| `AuthenticationContext.acquireTokenWithRefreshToken`      | `token`, `client_id`, `client_secret`, `resource`, `callback`                    | Предоставляет маркер доступа для указанного ресурса на основе маркера обновления.                             |

В приложении Connect запросы маршрутизируются через [`authHelper.js`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js), что позволяет добавить `client_id` и `client_secret`.

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
## Отправка запроса в API Microsoft Graph

Запросы к API Graph должны быть подписаны заголовком `Authorization`, содержащим маркер доступа к запрошенному ресурсу веб-службы. Правильный заголовок авторизации содержит маркер доступа из библиотеки adal-node и выглядит так, как показано ниже.

`Authorization: Bearer <access token>`

Теперь мы можем подписывать запросы с помощью маркера доступа, используя библиотеку `adal-node` в сочетании с логикой проверки подлинности из предыдущего раздела.

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

Microsoft Graph — это единый API, с помощью которого можно взаимодействовать со всеми данными Майкрософт. Сведения о других возможностях API Microsoft Graph см. в [справочнике по API](http://graph.microsoft.io/docs/api-reference/v1.0).

<!--## Additional resources

- [Office 365 Node.js Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Nodejs-Unified-API-Connect)-->

