# Проверка подлинности конечных точек Microsoft Graph с помощью конечной точки проверки подлинности версии 2


<!--
### Preview documentation
There are features and functionality of the converged authentication model that are not yet supported in the public preview period. You should be aware of them if you are building applications during the public preview. For more information, see [Limitations and restrictions of the converged authentication model preview](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).
-->

## Вход в учетную запись Майкрософт и проверка подлинности пользователей Azure AD с помощью единой модели

Используя конечную точку проверки подлинности (версия 2), вы можете создавать приложения, принимающие удостоверения как для рабочих и учебных (Azure AD), так и личных (учетная запись Майкрософт) учетных записей.

Ранее для поддержки учетных записей Майкрософт и Azure Active Directory приложение нужно было интегрировать с двумя отдельными системами. Теперь можно использовать конечную точку проверки подлинности версии 2. Одна простая процедура позволяет охватить миллионы пользователей с личными и рабочими или учебными учетными записями.   

После интеграции приложений с помощью конечной точки проверки подлинности версии 2 они получают доступ к конечным точкам Microsoft Graph, доступным для личных и рабочих или учебных учетных записей, например: 

| Данные              | Конечная точка                                       |
|:------------------|:-----------------------------------------------|
| Профиль пользователя      | `https://graph.microsoft.com/v1.0/me`          |
| Почта Outlook      | `https://graph.microsoft.com/v1.0/me/messages` |
| Контакты Outlook  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Календари Outlook | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**Примечание.** Некоторые конечные точки Microsoft Graph, такие как группы и задачи, не применимы к личным учетным записям.  

## Разрешения API Microsoft Graph

Конечная точка проверки подлинности (версия 2) поддерживает все разрешения для проверки подлинности Azure AD, перечисленные в статье [Разрешения Microsoft Graph](permission_scopes.md). Тем не менее конечная точка проверки подлинности (версия 2) в настоящий момент не поддерживает разрешения только для приложений.

>**Примечание**. В настоящее время перед строкой разрешений необходимо указывать URL-адрес ресурса https://graph.microsoft.com. Например, чтобы использовать разрешения `Files.Read`, укажите их в таком виде: `https://graph.microsoft.com/Files.Read`.

Дополнительные сведения об использовании разрешений с конечной точкой проверки подлинности (версия 2) и о том, как она отличается от ресурсов в Azure AD, см. в разделе [Разрешения, а не ресурсы](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources).

<!--
The table below lists the authentication scopes to use with the converged authentication model preview. For more information about using scopes with the converged authentication model, and how it differs from using resources in Azure AD, see [Scopes, not resources](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources).


| **Scope**             | **Permission**                        | **Description**                                                                                                                                         |
|:----------------------|:--------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `User.Read`           | Enable sign-in and read user profile  | Allows users to sign-in to the app, and allows the app to read the profile. It also allow the app to read basic company information of signed-in users. |
| `User.ReadWrite`      | Read and write access to user profile | Allows the app to read the profile of signed-in users, and to update profile information on behalf of signed-in users.                                  |
| `Mail.Read`           | Read user mail                        | Allows this app to read messages in user mailboxes.                                                                                                     |
| `Mail.ReadWrite`      | Read and write access to user mail    | Allows the app to read, update, create, and delete messages in user mailboxes.                                                                          |
| `Mail.Send`           | Send mail as a user                   | Allows the app to send messages as users in the organization.                                                                                           |
| `Contacts.Read`       | Read user contacts                    | Allows the app to read user contacts.                                                                                                                   |
| `Contacts.ReadWrite`  | Have full access to user contacts     | Allows the app to read, update, create and delete user contacts.                                                                                        |
| `Calendars.Read`      | Read user calendars                   | Allows the app to read events in user calendars.                                                                                                        |
| `Calendars.ReadWrite` | Have full access to user calendars    | Allows the app to read, update, create, and delete events in user calendars.                                                                            |
| `Files.Read`          | Read users' files                     | Allows the application to read the current user's files.                                                                                                |
| `Files.ReadWrite`     | Edit or delete users' files           | Allows the app to edit or delete the current user's files.                                                                                              |
| `openid`              | Sign users in                         | Allows users to sign in to the app and allows the app to see basic user profile information.                                                            |
| `offline_access`      | Read and write user's information     | Allows the app to see and update user's data, even when the user is not actively using the app.                                                         |

**Note**: currently it is required to pass the resource url of 'https://graph.microsoft.com' as prefix for the scope string. For example, to use the `Files.Read` scope you would specify the scope as `https://graph.microsoft.com/Files.Read`.
-->


## Дальнейшие действия

[Регистрация приложения для использования конечной точки проверки подлинности (версия 2)](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/)

## Подробнее

[Новые возможности модели проверки подлинности (версия 2)](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare)

[Ограничения модели проверки подлинности (версия 2)](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

[Документация по конечным точкам проверки подлинности Microsoft Azure (версия 2)](https://azure.microsoft.com/en-us/documentation/articles/?service=active-directory&term=app+model+v2.0)
