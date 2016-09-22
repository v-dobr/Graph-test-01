


# Обзор Microsoft Graph

Microsoft Graph (прежнее название — единый API Office 365) предоставляет несколько API из облачных служб Майкрософт через одну конечную точку API REST (**https://graph.microsoft.com**). Microsoft Graph значительно упрощает сложные или составные запросы. 
 
Microsoft Graph обеспечивает:

- единую конечную точку API для доступа к сводным данным из нескольких облачных служб Майкрософт в одном ответном пакете; 
- легкую навигацию между объектами и их отношениями; 
- доступ к аналитике и статистике, поступающим из облака Майкрософт.

Для всех этих задач нужен только один токен аутентификации.

API можно использовать для доступа к фиксированным объектам, например пользователям, группам, почте, сообщениям, календарям, задачам и заметкам из различных служб, например Outlook, OneDrive, Azure Active Directory, Planner, OneNote и других. Вы также можете получать вычисляемые отношения на базе Office Graph (только для пользователей платных служб), например список пользователей, с которыми вы работаете, или документы, популярные в вашей компании.

Microsoft Graph предоставляет две конечные точки. Общедоступная конечная точка /v1.0 и предварительная версия конечной точки /beta.  В рабочих приложениях можно использовать /v1.0, но не /beta.  Предварительная версия конечной точки /beta предназначена для экспериментов с последними функциями и сбора отзывов, API в бета-версии могут измениться в любой момент и не готовы для использования в рабочей среде.

<!--<a name="msg_queries"> </a>-->

##Общие запросы

Ниже приведено несколько примеров общих запросов с использованием API Microsoft Graph.

| **Операция** | **Конечная точка службы** |
|:--------------------------|:----------------------------------------|
|   Получение своего профиля |    `https://graph.microsoft.com/v1.0/me` |
|   Получение своих файлов|   `https://graph.microsoft.com/v1.0/me/drive/root/children` |
|   Получение своей фотографии     | `https://graph.microsoft.com/v1.0/me/photo/$value` |
|   Получение своей почты |   `https://graph.microsoft.com/v1.0/me/messages` |
|   Получение электронной почты высокой важности | `https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'` |
|   Получение своего календаря |   `https://graph.microsoft.com/v1.0/me/calendar` |
|   Получение сведений о руководителе  | `https://graph.microsoft.com/v1.0/me/manager` |
|   Получение сведений о последнем пользователе, изменившем файл foo.txt |  `https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser` |
|   Получение списка единых групп, в которые вы входите|   `https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')` |
|   Получение списка пользователей в организации     | `https://graph.microsoft.com/v1.0/users` |
|   Получение групповых бесед |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations` |
|   Получение списка связанных пользователей    | `https://graph.microsoft.com/beta/me/people` |
|   Получение файлов, популярных в вашей компании |  `https://graph.microsoft.com/beta/me/trendingAround` |
|   Получение списка коллег     | `https://graph.microsoft.com/beta/me/workingWith` |
|   Получение своих задач    | `https://graph.microsoft.com/beta/me/tasks` |
|   Получение своих заметок |  `https://graph.microsoft.com/beta/me/notes/notebooks` |

<!-- <a name="msg_roof"> </a> -->

## Все данные Office 365 собраны вместе

На следующей схеме показан стек разработчика Microsoft Graph и принцип его работы.

![Стек разработчика API Microsoft Graph.](./images/MicrosoftGraph_DevStack.png)

 >  Ваше мнение важно для нас. Для связи с нами используйте сайт [Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph). Помечайте свои вопросы тегами [MicrosoftGraph] и [office365].



