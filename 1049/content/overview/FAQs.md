
# Microsoft Graph: вопросы и ответы

## Какие платформы поддерживает API Microsoft Graph?
<!--
Apps can use the Microsoft Graph API to perform create, read, update, and delete (CRUD) operations on data sources and entities, giving them seamless access to work data. 

**Ease of use--one endpoint, all Office 365 data under one roof**

You can use the API in four steps:
1.  Select your programming language and development environment.
2.  Build your app.
3.  Optionally, host your app in Microsoft Azure or any cloud platform you choose.
4.  Authenticate your users by using single sign-on with Azure AD.

As a developer you can use the API to create custom apps that access and interact with all the richness of enterprise and productivity data--users, groups, organizational contacts, files, folders, mail, calendar, insights and relationships--and build apps across all mobile, web, and desktop platforms. No matter your development platform or tools. Using a single service endpoint to access those entities and data. And a single authentication flow.  -->

Можно выполнить следующие действия.

<!--Just like in Office 365 APIs, Office 365 unified endpoint API  allows you to build apps using any development environment of your choice:  -->

- использовать любую знакомую среду разработки, например .NET, PHP, Java, Python или Ruby;
- использовать любой язык программирования, платформу разработки и среду размещения;
- создать приложение с доступом к API, используя любой веб-язык, в том числе JavaScript, HTML5, Python, Ruby, PHP и ASP.NET;  
- использовать любой интерфейс: Visual Studio, Eclipse, Android Studio, Xcode или другой;
- размещать приложения в службе Microsoft Azure или на любой облачной платформе;
- разрабатывать приложения на универсальной платформе для Windows, iOS, Android или другой платформе;
- вызывать API из надстроек Office (ранее — приложения для Office) или надстроек SharePoint (ранее — приложения для SharePoint).
 


## Преимущества API Microsoft Graph

Предположим, вы хотите программным путем получить файлы и изображение профиля пользователя, а также найти руководителя пользователя, который изменил определенный файл в вашей организации. Так как сведения хранятся в нескольких службах (Azure Active Directory, SharePoint и Exchange), при использовании API Office 365 для этого необходимо выполнить несколько действий: 

1. найти различные конечные точки служб, используя службу обнаружения; 
2. определить URL-адреса служб, к которым нужно подключить приложения Office 365;
3. затем получить токен доступа для каждой службы, добавить его и отправить запрос.

Эту же сложную операцию можно выполнить через одну конечную точку API REST с помощью API Microsoft Graph. Не нужно находить конечную точку для каждой службы и переходить в нее, получать токен доступа для каждой службы и добавлять его, а также иметь дело с разрозненными службами и разными моделями данных.

##Примеры запросов

В следующем примере показана текущая модель взаимодействия с API Office 365 с использованием разрозненных конечных точек служб. Обратите внимание, насколько проще работать с API Microsoft Graph.

**Конечные точки разрозненных служб**

|   **Operation**                  |  **API**                          |  **Конечная точка службы** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Обнаружение конечных точек служб для API Office 365               |     `Discovery Service`           | _https://_**api.office.com**_/discovery/v1.0/me/services_ |
| Получение списка пользователей           |     `Azure AD Graph API` | _https://_**graph.windows.net**_/contoso.com/users?api-version=2013-04-05_|
| Получение сообщений из папки "Входящие"       |     `Office 365 API`           | _https://_**outlook.office365.com**_/api/v1.0/me/messages_  |
| Получение файлов Сергея   |     `Office 365 API`  | _https://_**contoso-my.sharepoint.com**_/personal/joe_contoso_com/_api/v1.0/files_ |


Если вы используете API Microsoft Graph, вам не нужно сначала находить конечные точки служб, а затем перебирать различные конечные точки, чтобы получить файлы, почту и другие данные пользователя. Работать нужно только с одним пространством имен URL-адреса REST, а именно _**graph.microsoft.com**_.

**API Microsoft Graph**

|   **Operation**                  |  **API**                          |  **Конечная точка службы** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Обнаружение конечных точек служб для API Office 365                |     `Microsoft Graph`           | Не требуется |
| Получение списка пользователей           |     `Microsoft Graph` | _https://_**graph.microsoft.com**_/v1.0/contoso.onmicrosoft.com/users_ |
| Получение сообщений из папки "Входящие"       |     `Microsoft Graph`           | _https://_**graph.microsoft.com**_/v1.0/me/messages_  |
| Получение файлов Сергея   |     `Microsoft Graph `  | _https://_**graph.microsoft.com**_/v1.0/me/drive/root/children_ |


## Преимущества API Microsoft Graph

Ниже приведены некоторые преимущества API Microsoft Graph.

**Единый и упрощенный интерфейс доступа к облачным службам Майкрософт для разработчиков**

-   Одно пространство имен для всех конечных точек служб. Не требуется обнаружение конечных точек служб.
-   Один токен для доступа ко всем ресурсам.
-   Интегрированная и прямая навигация между разрозненными службами (например, получение сведений об отделе и руководителях пользователя, который создал определенный документ).
-   Требуется только один набор API, т. е. к нескольким службам можно подключиться с помощью API Microsoft Graph.
-   Единый и развернутый API REST и объекты со всей платформы Office. 
-   Одинаковые названия свойств и единообразные схемы во всех объектах, в том числе свойств навигации.

**Более продуктивная работа с Office**

-   Контекстно-зависимое содержимое. Например, поиск документов по группе, проекту, команде или популярности.
-   Контекстно-зависимые связи пользователей. Например, вы можете искать пользователей по членству в группах, интересам, навыкам и опыту.  Вы также можете просмотреть организационную диаграмму.

**Возможность разработки приложений на любом языке программирования и на любой платформе**

-   Средства и ресурсы разработки для всех разработчиков. Возможность разработки с использованием любой платформы и языка. 
-   Разработка мобильных приложений для всех платформ, использующих открытые технологии.  
-   Для доступа к объектам API Microsoft Graph не требуются специальные знания по Exchange, SharePoint или Azure AD.

<!---<a name="msg_v2auth"> </a>-->

## Поддерживает ли API Microsoft Graph конечную точку проверки подлинности приложений версии 2.0?

Да. Дополнительные сведения см. в статье [Проверка подлинности конечных точек Microsoft Graph с помощью конечной точки версии 2.0](http://graph.microsoft.io/docs/authorization/converged_auth).



  > Ваш отзыв важен для нас. Для связи с нами используйте сайт [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Помечайте свои вопросы тегами [MicrosoftGraph] и [office365].








