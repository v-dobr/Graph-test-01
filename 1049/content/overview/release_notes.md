# Заметки о выпуске Microsoft Graph и известные проблемы

В этой статье приведены сведения о новых возможностях для разработчиков, которые доступны в выпуске API Microsoft Graph за ноябрь 2015 г., а также рассмотрены известные проблемы. 


## Общедоступные версии функций API Microsoft Graph

Следующие функции API Microsoft Graph общедоступны:

* Пользователи
* Группы
* Files
* Почтовое
* Календарь
* Личные контакты 
* Операции CRUD (создание, чтение, обновление, удаление)
* Поддержка общего доступа к ресурсам независимо от источника (CORS).

    
## Предварительные версии функций API Microsoft Graph

Следующие функции API Microsoft Graph доступны в предварительной версии:

* Заметки 
* Задачи
* Excel
* Люди
* Контакты организации
* Приложения
* Субъекты-службы
* Расширения схемы каталога
* Веб-перехватчики
* Статистика и отношения: популярные документы и коллеги
* Мгновенный доступ групп к контенту после создания
* Модель единой проверки подлинности личных, рабочих и учебных учетных записей. Несмотря на то что это предварительная версия функции, она также доступна в версиях `/v1.0` и `/beta`.


## Известные проблемы Microsoft Graph

Ниже приведены известные проблемы, связанные с Microsoft Graph.

### Пользователи
#### Нет мгновенного доступа после создания
Пользователей можно создавать мгновенно с помощью метода POST для объекта user. Для доступа к службам Office 365 пользователю нужна соответствующая лицензия. Из-за распределенного характера службы файлы, сообщения и события этого пользователя станут доступны посредством API Microsoft Graph через 15 минут после назначения лицензии. В течение этого времени приложения будут получать ошибку HTTP 404. 

#### Ограничения для фотографий
Просмотр и обновление фотографии профиля пользователя возможны, только если у пользователя есть почтовый ящик.  Кроме того, все фотографии, которые *могли* ранее храниться со свойством **thumbnailPhoto** (с помощью предварительной версии единого API Office 365 или Azure AD Graph или службы синхронизации AD Connect), больше не будут доступны через свойство фотографии пользователя Microsoft Graph.  В этом случае сбой при просмотре или обновлении фотографии вызовет следующую ошибку:

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **ПРИМЕЧАНИЕ**.  Вскоре после выхода общедоступной версии будет добавлена возможность хранения и извлечения фотографий профилей пользователей, даже если у них нет почтового ящика, и эта ошибка должна исчезнуть.

#### Папка контактов по умолчанию

В версии `/v1.0` запрос `GET /me/contactFolders` не включает папку контактов пользователя по умолчанию. 

Эта ошибка будет исправлена. Чтобы получить идентификатор папки контактов по умолчанию, вы можете использовать следующий запрос на 
[отображение контактов](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts) и свойство **parentFolderId**:

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
В указанном выше запросе:
1. `/me/contacts?$top=1` возвращает свойства [контакта](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact) в папке контактов по умолчанию.
2. При добавлении `&$select=parentFolderId` возвращается только свойство контакта **parentFolderId** — идентификатор папки контактов по умолчанию.

#### Добавление календарей ICS в почтовый ящик пользователя и доступ к ним
В настоящее время календари ICS поддерживаются частично:
* Вы можете добавить календарь ICS в почтовый ящик пользователя через интерфейс пользователя, но не через API Microsoft Graph. 
* [Список календарей пользователя](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars) 
позволяет получить свойства **name**, **color** и **id** всех [календарей](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar) 
в группе календарей пользователя по умолчанию или указанной группе календарей, в том числе календарей ICS. URL-адрес ICS невозможно хранить и открывать в календаре ресурсов.
* Вы также можете [вывести список событий](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events) календаря ICS.

### Группы
#### Политика
С помощью Microsoft Graph можно создать единую группу и присвоить ей название в обход групповой политики, настроенной через Outlook Web App. 

#### Разрешения для групп
Microsoft Graph предоставляет два разрешения (*Group.Read.All* и *Group.ReadWrite.All*) для доступа к API-интерфейсам групп.  В полной версии, в отличие от предварительной, эти разрешения должен подтвердить администратор.  В будущем мы планируем добавить новые разрешения для групп, которые смогут предоставлять пользователи.

#### Добавление и получение вложений в записях групп
В настоящее время при [добавлении](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments) вложений в записи группы, 
[отображении](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments) и получении вложений возвращается сообщение об ошибке "Запрос OData не поддерживается". 
Для версий `/v1.0` и `/beta` подготовлено исправление. Оно станет доступно всем пользователям в конце января 2016 года.

### Контакты
* В настоящее время поддерживаются только личные контакты. В настоящее время контакты организации не поддерживаются в версии `/v1.0`, но их можно найти в версии `/beta`.
* Не возвращается номер мобильного телефона контакта. Он будет добавлен чуть позже. Доступ к нему можно получить с помощью API Outlook.

### Диски, файлы и потоковая передача контента
* При первом доступе к личному диску пользователя с помощью Microsoft Graph до открытия им своего личного сайта в браузере возвращается ответ 401.
* Максимальный размер отправляемых и скачиваемых файлов (файлов в группах, почтовых вложениях и на дисках Office) — 4 МБ.

### Функции, доступные только в существующих API REST для Office 365
#### Синхронизация
В версиях `/v1.0` и `/beta` недоступны возможности синхронизации Outlook, OneDrive и Azure AD (в Azure AD их также называют разностными запросами).  Если приложению необходимы возможности синхронизации, продолжайте использовать существующие REST API для Office 365 и Azure AD или изучите новую предварительную версию веб-перехватчиков, доступную в Microsoft Graph для событий, сообщений и контактов.

> **ПРИМЕЧАНИЕ**.  Мы планируем устранить различия между существующими API и Microsoft Graph как можно скорее, включая синхронизацию.

#### Пакетная обработка
В Microsoft Graph пакетная обработка не поддерживается. Тем не менее можно использовать конечную точку бета-версии Outlook и 
[пакетные вызовы Outlook REST](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests). 

#### Доступность в Китае
Служба Microsoft Graph предоставляется компанией 21Vianet (и теперь доступна в Китае). Дополнительные сведения, в том числе ограничения, см. в документе о [независимых облачных развертываниях Microsoft Graph](http://graph.microsoft.io/docs/overview/deployments).

#### Действия и функции службы
В Microsoft Graph недоступны методы `isMemberOf` и `getObjectsById`.

### Разрешения Microsoft Graph
Последние сведения о поддерживаемых разрешениях для приложений и делегированных разрешениях Microsoft Graph см. в [статье, посвященной разрешениям Microsoft Graph](http://graph.microsoft.io/docs/authorization/permission_scopes).  Кроме того, для версии `v1.0` действуют следующие ограничения:

|Разрешение |   Тип разрешения | Ограничение |  Альтернатива |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| Делегированное    | Невозможно обновить номер мобильного телефона.|    Выберите также `Directory.AccessAsUser.All`.| 
|_User.ReadWrite.All_|  Делегированное|  С объектом `User` невозможно выполнять никаких операций CRUD, кроме обновления фотографии пользователя в формате HD и расширенных свойств профиля.| Выберите также `Directory.ReadWrite.All` или `Directory.AccessAsUser.All`, если необходимо удалить пользователя.|
|_User.Read.All_|   Приложение |Невозможно выполнять операции чтения с другими пользователями.| Выберите также `Directory.Read.All`.|
| _User.ReadWrite.All_ |    Приложение |   С объектом `User` невозможно выполнять никаких операций CRUD, кроме обновления фотографии пользователя в формате HD и расширенных свойств профиля. |    Выберите также `Directory.ReadWrite.All`. **ПРИМЕЧАНИЕ**. Удалять пользователей невозможно.|
|_Group.Read.All_   | Приложение | Невозможно перечислять группы или их участников.  При этом можно просматривать содержимое групп Office.   | Выберите также `Directory.Read.All`. |
|_Group.ReadWrite.All_  | Приложение   | Невозможно перечислять группы или их участников, создавать группы, обновлять членство в группах или удалять их.  При этом можно просматривать и обновлять содержимое групп Office.   | Выберите также `Directory.ReadWrite.All`. **ПРИМЕЧАНИЕ**.  Удаление групп невозможно.|

Кроме того, для версии `/beta` действуют следующие ограничения:

|Разрешение |   Тип разрешения | Ограничение |  Альтернатива |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | Делегированное | Невозможно просматривать или обновлять задачи планировщика в группах Office.  | Выберите также `Tasks.ReadWrite`.|
|_Tasks.ReadWrite_  | Делегированное | Невозможно просматривать или обновлять задачи вошедшего пользователя.| Выберите также `Group.ReadWrite.All`.|

### Ограничения, связанные с OData
* Ограничения **$expand**: 
 * Отсутствует поддержка `nextLink`.
 * Поддерживается не более одного уровня развертывания.
 * Не поддерживаются дополнительные параметры (**$filter**, **$select**).
* Не поддерживается несколько пространств имен.
* Не поддерживаются запросы GET для `$ref` и приведение для пользователей, групп, устройств, субъектов-служб и приложений.
* Не поддерживается `@odata.bind`.  Это значит, что разработчики не смогут правильно задавать параметры `Accepted` и `RejectedSenders` для группы.
* Отсутствует `@odata.id` для навигации по объектам без вложений (например, сообщениям) при использовании минимальных метаданных.
* Не поддерживаются фильтрация и поиск для нескольких рабочих нагрузок. 
* Полнотекстовый поиск (с помощью параметра **$search**) доступен только для некоторых объектов, например сообщений.

  >  Ваш отзыв важен для нас. Для связи с нами используйте сайт [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Помечайте свои вопросы тегами [MicrosoftGraph] и [office365].

  
             .

