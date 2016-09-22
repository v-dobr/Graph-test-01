# Вызов Microsoft Graph в приложении Angular 

В этой статье мы рассмотрим, как подключить приложение к Office 365 и вызвать API Microsoft Graph. Мы объясним основные понятия на примере [приложения Office 365 Connect на Angular, использующего Microsoft Graph](https://github.com/microsoftgraph/angular-connect-rest-sample).

![Снимок экрана приложения Angular Connect для Office 365](./images/web-screenshot.png)

## Необходимые компоненты  

В этой статье предполагается, что:

* вы уверенно читаете код JavaScript и [AngularJS](https://angularjs.org/).

## Обзор

Чтобы вызвать API Microsoft Graph, необходимо выполнить следующие задачи:

1. Зарегистрировать приложение в Azure Active Directory.
2. Настроить библиотеку Azure Active Directory для JavaScript (ADAL JS).
3. Получить токен доступа с помощью ADAL JS.
4. Использование токена доступа в запросе к API Microsoft Graph

<!--<a name="register"></a>-->
## Регистрация приложения в Azure Active Directory

Прежде чем начать работу с Office 365, необходимо зарегистрировать свое приложение и настроить разрешения на использование служб Microsoft Graph. С помощью [средства регистрации приложений](https://dev.office.com/app-registration) вы можете с легкостью зарегистрировать свое приложение для доступа к рабочей или учебной учетной записи пользователя. 
Для управления своей учетной записью пользователю необходимо перейти на [портал управления Microsoft Azure](https://manage.windowsazure.com).

Инструкции по регистрации приложения вручную см. в разделе [Регистрация браузерного веб-приложения на портале управления Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp). Обратите внимание на следующие рекомендации:

* Обязательно укажите http://127.0.0.1:8080/ в качестве **URL-адреса входа**.
* Зарегистрировав приложение, [настройте **делегированные разрешения**](https://github.com/microsoftgraph/angular-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), необходимые для вашего приложения Angular. Для приложения Connect необходимо разрешение **Отправлять почту как вошедший пользователь**.

Запишите следующие значения на странице **Настройка** приложения Azure, так как они понадобятся для настройки [ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) в приложении Angular:

* идентификатор клиента (уникальный для приложения);
* URL-адрес ответа (http://127.0.0.1:8080/)

<!--<a name="adal"></a>-->
## Настройка библиотеки Azure Active Directory для JavaScript (ADAL JS)

[ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) — это библиотека JavaScript, которая обеспечивает полную поддержку входа пользователей Azure AD в одностраничных приложениях (SPA), например Connect, и управление токенами, а также другие возможности. Чтобы использовать эту библиотеку, ее необходимо добавить в приложение Angular и настроить.

Просто добавьте библиотеку и ее модуль для приложений Angular, используя сеть Microsoft CDN.

```html
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal.min.js"></script>
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal-angular.min.js"></script>
```

После этого необходимо настроить службу ADAL JS в файле с зависимостями приложения Angular. В приложении Connect для этого используется файл [*public/app.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/scripts/app.js). 

Чтобы настроить библиотеку ADAL JS, сначала добавьте ссылку на модуль ADAL, вставив ```AdalAngular``` в массив требований модуля, а затем передайте ```adalAuthenticationServiceProvider``` в функцию ```config```. Настройте библиотеку с помощью функции ```init```, передав ей идентификатор клиента своего приложения и объект ```endpoints``` , который объявляет, в какие API приложение Angular должно отправлять запросы CORS.

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
## Получение токена доступа с помощью ADAL JS

Ваше приложение должно перенаправлять браузер на страницу входа, чтобы пользователь мог войти и предоставить приложению доступ к данным. Для этого приложение Connect использует библиотеку ADAL JS. 

В одном из контроллеров приложения сначала добавьте ссылку на службу ADAL, вставив ```adalAuthenticationService``` в контроллер, а затем определите функцию, использующую функцию службы ```login```, которую может вызывать пользовательский интерфейс. В приложении Connect для этого используется файл [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js). 

```javascript
/**
  * Expose the login method from ADAL to the view.
  */
function connect() {
  adalAuthenticationService.login();
};
```

После вызова этой функции приложение перенаправит пользователя на страницу входа. После входа и авторизации приложения пользователь будет перенаправлен в приложение с токеном доступа в строке запроса, которую библиотека ADAL JS извлечет и сохранит. 

<!--<a name="request"></a>-->
## Использование токена доступа в запросе к API Microsoft Graph

Используя токен доступа, ваше приложение может отправлять прошедшие проверку запросы в API Microsoft Graph. Библиотека ADAL JS автоматически перехватывает все HTTP-запросы и добавляет к ним токен доступа, поэтому вам не нужно вручную задавать заголовок при использовании библиотеки. 

Приложение Connect отправляет электронную почту, используя конечную точку ```me/sendMail``` в API Microsoft Graph в файле [*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js). 

Microsoft Graph — это единый API, с помощью которого можно взаимодействовать со всеми данными Майкрософт. Сведения о других возможностях API Microsoft Graph см. в [справочнике по API](http://graph.microsoft.io/docs/api-reference/v1.0).

