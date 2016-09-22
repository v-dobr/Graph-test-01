# v2 認証エンドポイントを使用して Microsoft Graph のエンドポイントを認証する


<!--
### Preview documentation
There are features and functionality of the converged authentication model that are not yet supported in the public preview period. You should be aware of them if you are building applications during the public preview. For more information, see [Limitations and restrictions of the converged authentication model preview](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).
-->

## Microsoft アカウントと Azure AD ユーザーの単一の認証モデルを使用したサインイン

v2 認証エンドポイントを使用すると、職場または学校 (Azure AD) ID と、個人 (Microsoft アカウント) ID の両方を受け入れるアプリを作成できます。

以前は、Microsoft アカウントと Azure Active Directory の両方をサポートするアプリを開発する場合、完全に独立した 2 つのシステムと統合する必要がありました。v2 認証エンドポイントを使用して、1 回の統合で両方の種類のアカウントをサポートできるようなりました。1 つの単純なプロセスで、個人のアカウントと、仕事/学校のアカウントを持つ数百万に及ぶ両方の対象ユーザーにすぐに到達できます。   

v2 認証エンドポイントとアプリケーションを統合すると、次のように、個人と仕事/学校の両方のアカウントで利用できる Microsoft Graph エンドポイントに即座にアクセスできます。 

| データ              | エンドポイント                                       |
|:------------------|:-----------------------------------------------|
| ユーザー プロファイル      | `https://graph.microsoft.com/v1.0/me`          |
| Outlook のメール      | `https://graph.microsoft.com/v1.0/me/messages` |
| Outlook の連絡先  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Outlook の予定表 | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**注:**グループやタスクなどのいくつかの Microsoft Graph エンドポイントは、個人用のアカウントに該当しません。  

## Microsoft Graph API 認証のスコープ

v2 認証エンドポイントは、「[Microsoft Graph のアクセス許可スコープ](permission_scopes.md)」トピックで Azure AD の認証で使用できるものとして一覧表示されているすべてのアクセス許可スコープをサポートしています。ただし、v2 認証エンドポイントでは、現在のところアプリ専用のスコープはサポートされていません。

>**注**:現在のところ、スコープ文字列のプレフィックスとしてリソース URL 'https://graph.microsoft.com' を渡す必要があります。たとえば、`Files.Read` のスコープを使用するためには、`https://graph.microsoft.com/Files.Read` のようにスコープを指定します。

v2 認証エンドポイントでのスコープの使用に関する詳細、および Azure AD でのリソースの使用との違いについては、「[リソースではなく、スコープ](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources)」をご覧ください。

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


## 次の手順

[アプリを v2 認証エンドポイントを使用するよう登録する](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/)

## 詳細を見る

[v2 認証モデルの新機能](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare)

[v2 認証モデルの制限事項と制約事項](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

[Microsoft Azure v2 認証エンドポイントのドキュメント](https://azure.microsoft.com/en-us/documentation/articles/?service=active-directory&term=app+model+v2.0)
