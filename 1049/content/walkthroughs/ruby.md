# Вызов Microsoft Graph в приложении Ruby 

В этой статье мы рассмотрим, как получить токен Microsoft Graph. Мы объясним основные понятия на примере [приложения Office 365 Connect на Ruby, использующего Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample).

![Снимок экрана с примером приложения на Ruby, подключающегося к Office 365](./images/web-screenshot.png)

## Обзор

Чтобы вызвать API Microsoft Graph в приложении Ruby, необходимо выполнить следующие задачи:

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

* На шаге 6 укажите маршрут в приложении Ruby в качестве **URL-адрес для входа**. В приложении Connect это [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L41).
* [Настройте **делегированные разрешения**](https://github.com/microsoftgraph/ruby-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), необходимые приложению. Для приложения Connect требуется разрешение **Отправлять почту как вошедший пользователь**.

Запишите следующие значения со страницы **Настройка** приложения Azure:

* Идентификатор клиента
* действительный ключ;
* URL-адрес ответа.

Эти значения необходимы для настройки потока OAuth в приложении.

<!--<a name="redirect"/>-->
## Перенаправление браузера на страницу входа

Приложение должно перенаправить браузер на страницу входа, чтобы получить код авторизации и продолжить поток OAuth.

В приложении Connect перенаправление выполняет библиотека OmniAuth. Наше приложение просто делегирует выполнение маршруту [`/auth/azureactivedirectory`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L30), управляемому OmniAuth.

<!--<a name="authcode"/>-->
## Получение кода авторизации на странице URL-адреса ответа

После входа пользователя браузер перенаправляется на страницу URL-адреса ответа в приложении. Azure добавляет код авторизации в строку запроса. В приложении Connect для этого используется маршрут [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L38).

Код авторизации указан в переменной строки запроса `code`. Приложение Connect сохраняет код в локальной переменной для дальнейшего использования.

```ruby
code = params[:code]
```

<!--<a name="accesstoken"/>-->
## Запрос токена доступа из конечной точки токена

Используя код авторизации и значения идентификатора клиента, ключа и URL-адреса ответа, полученные из Azure AD, вы можете запросить токен доступа. 

> **Примечание.** <br />
> В запросе необходимо также указать ресурс. Значение ресурса для Microsoft Graph — `https://graph.microsoft.com`.

Приложение Connect делегирует эту задачу библиотеке OmniAuth. Функция [`acquire_access_token`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L65) вызывает библиотеку и передает код проверки подлинности, сохраненный в предыдущем разделе вместе с URL-адресом ответа, идентификатором клиента, секретом клиента и ИД ресурса.

```ruby
def acquire_access_token(auth_code, reply_url)
    AUTH_CTX.acquire_token_with_authorization_code(
                  auth_code,
                  reply_url,
                  CLIENT_CRED,
                  GRAPH_RESOURCE)
end
```

> **Примечание.** <br />
> Идентификатор и секрет клиента указаны в параметре `CLIENT_CRED` в предыдущем фрагменте кода.

<!--<a name="request"/>-->
## Использование токена доступа в запросе к API Microsoft Graph

Используя токен доступа, приложение может отправлять проверенные запросы в API Microsoft Graph. Приложение должно указывать токен доступа в заголовке **Authorization** каждого запроса.

Приложение Connect отправляет электронную почту, используя конечную точку **sendMail** в API Microsoft Graph. Код находится в функции [`send_mail`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L82). Этот код показывает, как отправить код доступа в заголовке Authorization.

```ruby
def send_mail
  # Used in the template
  @name = session[:name]
  @email = params[:specified_email]
  @recipient = params[:specified_email]
  @mailSent = false
  
  sendMailEndpoint = URI("#{GRAPH_RESOURCE}#{SENDMAIL_ENDPOINT}")
  http = Net::HTTP.new(sendMailEndpoint.host, sendMailEndpoint.port)
  http.use_ssl = true
  
  emailBody = File.read("app/assets/MailTemplate.html")
  emailBody.sub! "{given_name}", @name
  
  emailMessage = "{
          Message: {
          Subject: 'Welcome to Office 365 development with Ruby',
          Body: {
              ContentType: 'HTML',
              Content: '#{emailBody}'
          },
          ToRecipients: [
              {
                  EmailAddress: {
                      Address: '#{@recipient}'
                  }
              }
          ]
          },
          SaveToSentItems: true
          }"

  response = http.post(
    SENDMAIL_ENDPOINT, 
    emailMessage, 
    initheader = 
    {
      "Authorization" => "Bearer #{session[:access_token]}", 
      "Content-Type" => CONTENT_TYPE
    }
  )

  # The send mail endpoint returns a 202 - Accepted code on success
  if response.code == "202"
    @mailSent = true
  else
    @mailSent = false
    flash[:httpError] = "#{response.code} - #{response.message}"
  end
  
  render "callback"
end
```

> **Примечание.** <br />
> В запросе необходимо также отправить заголовок **Content-Type** со значением, принимаемым API Microsoft Graph, например `application/json;odata.metadata=minimal;odata.streaming=true`.

API Microsoft Graph — это функциональный единый API, с помощью которого можно взаимодействовать со все типами данных Майкрософт. Сведения о других возможностях API Microsoft Graph см. в справочнике по API.

<!--## Additional resources

-  [Office 365 Ruby Connect sample using Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference](http://graph.microsoft.io/en-us/docs)-->
