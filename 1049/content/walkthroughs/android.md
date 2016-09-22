# Вызов Microsoft Graph в приложении Android

В этой статье мы рассмотрим минимальный набор задач, необходимый для получения токена доступа из Azure Active Directory (AD) и вызова Microsoft Graph. Мы рассмотрим основные понятия, которые необходимо реализовать в приложении, на примере кода [приложения Android Connect для Office 365, использующего Microsoft Graph](https://github.com/microsoftgraph/android-java-connect-rest-sample).

На следующем изображении представлено действие примера приложения по отправке почты, которое появляется после подключения пользователя к Office 365.

![Снимок экрана приложения Android Connect для Office 365, использующего единый API](./images/AndroidConnect.png)

## Обзор

Для вызова API Microsoft Graph [приложение Office 365 Connect для Android](https://github.com/microsoftgraph/android-java-connect-rest-sample) выполняет следующие задачи:

1. Проверка подлинности пользователя и получение токена доступа путем вызова методов в библиотеке Azure Active Directory.
2. Создание запроса на почтовое сообщение в качестве операции REST в конечной точке API Microsoft Graph.

<!--<a name="register"/>-->
## Регистрация приложения в Azure Active Directory

Прежде чем начать работу с Office 365, необходимо зарегистрировать свое приложение и настроить разрешения на использование служб Microsoft Graph. С помощью [средства регистрации приложений](https://dev.office.com/app-registration) вы можете с легкостью зарегистрировать свое приложение для доступа к рабочей или учебной учетной записи пользователя. 
Для управления своей учетной записью пользователю необходимо перейти на [портал управления Microsoft Azure](https://manage.windowsazure.com).

Инструкции по регистрации приложения вручную см. в разделе **Регистрация собственного приложения на портале управления Azure** статьи [Регистрация приложения вручную в Azure AD для доступа к API Office 365](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually). Обратите внимание на следующие рекомендации:

* Настройте **делегированные разрешения**, необходимые приложению. Для приложения Connect требуется разрешение **Отправлять почту как вошедший пользователь**.

Запишите следующие значения со страницы **Настройка** приложения Azure:

* Идентификатор клиента
* URL-адрес перенаправления.

Эти значения необходимы для настройки кода проверки подлинности в приложении.

## Зависимости Gradle в приложении Connect
Приложение Connect принимает зависимости от библиотек, показанные в следующем фрагменте кода build.gradle:

```gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.1.1'

    // Azure Active Directory Library
    compile 'com.microsoft.aad:adal:1.1.7'

    // Retrofit + custom HTTP
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.0.0'
    compile 'com.squareup.okhttp:okhttp:2.0.0'
    compile 'com.squareup.retrofit:retrofit:1.9.0'
}

```
<!--<a name="authenticate"/>-->
## Проверка подлинности в приложении Connect
В приложении Connect для проверки подлинности используются значения регистрации приложения Azure и идентификатор пользователя. Приложение Connect поддерживает два метода проверки подлинности.

* Проверка подлинности по запросу. Используется, если идентификатор пользователя не сохранен в кэше на устройстве Android.
* Автоматическая проверка подлинности. Используется, если идентификатор пользователя сохранен в кэше и необязательно отправлять запрос.

В классе [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) доступен вспомогательный метод `isConnected()`, который позволяет найти все кэшированные идентификаторы пользователей и определить используемый метод проверки подлинности.


```java
    private boolean isConnected(){
        SharedPreferences settings = this
                .mContextActivity
                .getSharedPreferences(PREFERENCES_FILENAME, Context.MODE_PRIVATE);

        return settings.contains(USER_ID_VAR_NAME);
    }

```

При использовании любого метода для проверки подлинности ADAL требуются идентификатор клиента и URL-адрес перенаправления, полученные в процессе регистрации Azure. Эти строки хранятся в исходном коде приложения и извлекаются, прежде чем объект диспетчера проверит подлинность пользователя.

В интерфейсе [Constants.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/Constants.java) доступны две статические строки для идентификатора клиента и URL-адреса перенаправления.

```java
interface Constants {
    String AUTHORITY_URL = "https://login.microsoftonline.com/common";
    // Update these two constants with the values for your application:
    String CLIENT_ID = "<Your client id here>";
    String REDIRECT_URI = "<Your redirect uri here>";
    String UNIFIED_API_ENDPOINT = "https://graph.microsoft.com/v1.0/";
    String UNIFIED_ENDPOINT_RESOURCE_ID = "https://graph.microsoft.com/";
}
```
### Создание класса AuthenticationManager
Конструктор [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) не принимает аргументы, а задает поле строки класса из файла Constants.java с помощью URL-адреса конечной точки Graph. Эта строка ресурса используется в обоих методах проверки подлинности.

```java
    private AuthenticationManager() {
        mResourceId = Constants.UNIFIED_ENDPOINT_RESOURCE_ID;
    }
```

### Проверка подлинности по запросу

В классе [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) доступен метод `authenticatePrompt()`, который позволяет получить токен доступа для вызовов REST в единой конечной точке.

Метод `acquireToken()` библиотеки ADAL является асинхронным. Аргументы метода: ссылка на контекст текущих действий, ресурс, идентификатор клиента и URL-адрес перенаправления. Ссылка на текущие действия позволяет библиотеке ADAL показывать страницу запроса учетных данных в списке действий. 
После успешной проверки подлинности библиотека ADAL инициирует обратный вызов метода `onSuccess()`. Он выполняет две функции:

* Сохраняет токен доступа в `mAccessToken`. При вызове REST для отправки почтового сообщения приложение Connect помещает этот токен доступа в заголовок Authorization.
* Сохраняет идентификатор пользователя в сохраненных параметрах.


```java
    /**
     * Calls acquireToken to prompt the user for credentials.
     *
     * @param authenticationCallback The callback to notify when the processing is finished.
     */
    private void authenticatePrompt(final AuthenticationCallback<AuthenticationResult> authenticationCallback) {
        getAuthenticationContext().acquireToken(
                this.mContextActivity,
                this.mResourceId,
                Constants.CLIENT_ID,
                Constants.REDIRECT_URI,
                PromptBehavior.Always,
                new AuthenticationCallback<AuthenticationResult>() {
                    @Override
                    public void onSuccess(final AuthenticationResult authenticationResult) {
                        if (authenticationResult != null) {
                            if (authenticationResult.getStatus() == AuthenticationStatus.Succeeded) {
                                setUserId(authenticationResult.getUserInfo().getUserId());
                                mAccessToken = authenticationResult.getAccessToken();
                                authenticationCallback.onSuccess(authenticationResult);
                            } else {
                                // We need to make sure that there is no data stored with the failed auth
                                AuthenticationManager.getInstance().disconnect();
                                // This condition can happen if user signs in with an MSA account
                                // instead of an Office 365 account
                                authenticationCallback.onError(
                                        new AuthenticationException(
                                                ADALError.AUTH_FAILED,
                                                authenticationResult.getErrorDescription()));
                            }
                        } else {
                            // I could not authenticate the user silently,
                            // falling back to prompt the user for credentials.
                            authenticatePrompt(authenticationCallback);
                        }
                    }

                    @Override
                    public void onError(Exception e) {
                        // We need to make sure that there is no data stored with the failed auth
                        AuthenticationManager.getInstance().disconnect();
                        authenticationCallback.onError(e);
                    }
                }
        );
    }

```

###Автоматическая проверка подлинности
В классе [AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) имеется метод `authenticateSilent()`, который позволяет получить токен доступа для вызовов REST в единой конечной точке.

Метод `acquireTokenSilent()` библиотеки ADAL является асинхронным. Кроме идентификаторов клиента и ресурса, полученных при регистрации в Azure, он принимает идентификатор пользователя, который хранится в общих параметрах. 
Вспомогательный метод `getUserId()` получает идентификатор пользователя из хранилища.

В случае успешной проверки подлинности вызывается метод `onSuccess()`. Метод `onSuccess` сохраняет токен доступа в `mAccessToken`. При вызове REST для отправки почтового сообщения приложение Connect помещает этот токен доступа в заголовок Authorization.
```java
    /**
     * Calls acquireTokenSilent with the user id stored in shared preferences.
     * In case of an error, it falls back to {@link AuthenticationManager#authenticatePrompt(AuthenticationCallback)}.
     *
     * @param authenticationCallback The callback to notify when the processing is finished.
     */
    private void authenticateSilent(final AuthenticationCallback<AuthenticationResult> authenticationCallback) {
        getAuthenticationContext().acquireTokenSilent(
                this.mResourceId,
                Constants.CLIENT_ID,
                getUserId(),
                new AuthenticationCallback<AuthenticationResult>() {
                    @Override
                    public void onSuccess(final AuthenticationResult authenticationResult) {
                        if (authenticationResult != null) {
                            if (authenticationResult.getStatus() == AuthenticationStatus.Succeeded) {
                                mAccessToken = authenticationResult.getAccessToken();
                                authenticationCallback.onSuccess(authenticationResult);
                            } else {
                                authenticationCallback.onError(
                                        new Exception(authenticationResult.getErrorDescription()));

                            }
                        } else {
                            // I could not authenticate the user silently,
                            // falling back to prompt the user for credentials.
                            authenticatePrompt(authenticationCallback);
                        }
                    }

                    @Override
                    public void onError(Exception e) {
                        // I could not authenticate the user silently,
                        // falling back to prompt the user for credentials.
                        authenticatePrompt(authenticationCallback);
                    }
                }
        );
    }

```
<!--<a name="sendmail"/>-->
## Отправка сообщения электронной почты с помощью Office 365

Когда пользователь входит в Azure, приложение Connect предлагает ему отправить сообщение. С помощью класса [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) приложение Connect отправляет сообщение после нажатия кнопки "Отправить почту".

### Вспомогательный класс адаптера REST
Класс [RESTHelper.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/RESTHelper.java) позволяет добавлять заголовок Authorization в каждый вызов REST, совершаемый приложением. Он использует токен доступа, предоставленный диспетчером проверки подлинности.

```java
       //This method catches outgoing REST calls and injects the Authorization and host headers before
        //sending to REST endpoint
        RequestInterceptor requestInterceptor = new RequestInterceptor() {
            @Override
            public void intercept(RequestFacade request) {
                final String token = mAccessToken;
                if (null != token) {
                    request.addHeader("Authorization", "Bearer " + token);
                }
            }
        };
```
### Класс UnifiedAPIController
Класс [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) создает запрос REST в методе `sendMail()`.


```java
    /**
     * Sends an email message using the Unified API on Office 365. The mail is sent
     * from the address of the signed in user.
     *
     * @param emailAddress The recipient email address.
     * @param subject      The subject to use in the mail message.
     * @param body         The body of the message.
     * @param callback     UI callback to be invoked by Retrofit call when
     *                     operation completed
     */
    public void sendMail(
            final String emailAddress,
            final String subject,
            final String body,
            Callback<Void> callback) {
        ensureService();
        // Use the Unified API service on Office 365 to create the message.
        mUnifiedAPIService.sendMail(
                "application/json",
                createMailPayload(
                        subject,
                        body,
                        emailAddress),
                callback);
    }

```
### Интерфейс UnifiedAPIService
Интерфейс [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) предоставляет сигнатуры методов для вызовов REST, которые осуществляет приложение с помощью аннотаций Retrofit.

```java
    @POST("/me/sendMail")
    void sendMail(
            @Header("Content-type") String contentTypeHeader,
            @Body TypedString mail,
            Callback<Void> callback);


```

## Дальнейшие действия
API Microsoft Graph — это функциональный единый API, с помощью которого можно взаимодействовать со все типами данных Майкрософт. Сведения о других возможностях API Microsoft Graph см. в [документации по Microsoft Graph](http://graph.microsoft.io/docs).

Мы опубликовали много примеров приложений Android для Office 365. В каждом из этих примеров используются понятия, представленные в приложении Connect. Чтобы открыть новые возможности приложений Android, изучите [другие примеры](http://aka.ms/androidgraphsamples) на сайте Office GitHub.
 
