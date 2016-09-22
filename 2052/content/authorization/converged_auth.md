# 使用 v2 身份验证终结点对 Microsoft Graph 终结点进行身份验证


<!--
### Preview documentation
There are features and functionality of the converged authentication model that are not yet supported in the public preview period. You should be aware of them if you are building applications during the public preview. For more information, see [Limitations and restrictions of the converged authentication model preview](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).
-->

## 使用一个身份验证模型登录 Microsoft 帐户和 Azure AD 用户帐户

您可以使用 v2 身份验证终结点创建接受工作、学校 (Azure AD) 以及个人（Microsoft 帐户）身份的应用。

过去，如果你想开发支持 Microsoft 帐户和 Azure Active Directory 两者的应用，则必须集成两个完全独立的系统。现在，使用 v2 身份验证终结点，你可以通过一次集成即可支持两种类型的帐户。一个简单的进程即可跨越成千上万的使用个人和工作/学校帐户的用户立即访问一个受众。   

在你将应用于 v2 身份验证终结点集成后，它们可以立即访问可用于个人和工作/学校帐户的 Microsoft Graph 终结点，例如： 

| 数据              | 终结点                                       |
|:------------------|:-----------------------------------------------|
| 用户配置文件      | `https://graph.microsoft.com/v1.0/me`          |
| Outlook 邮件      | `https://graph.microsoft.com/v1.0/me/messages` |
| Outlook 联系人  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Outlook 日历 | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**注意：**某些 Microsoft Graph 终结点（如组和任务）不适用于个人帐户。  

## Microsoft Graph API 身份验证范围

v2 身份验证终结点支持列出的所有权限范围，其均可使用 [Microsoft Graph 权限范围](permission_scopes.md) 主题中的 Azure AD 身份验证。不过，v2 身份验证终结点当前不支持仅限应用范围。

>**注意：**目前，必须将资源 URL“https://graph.microsoft.com”作为范围字符串的前缀传递。例如，要使用 `Files.Read` 范围，请将范围指定为 `https://graph.microsoft.com/Files.Read`。

若要详细了解如何结合使用 v2 身份验证终结点和身份验证范围，以及这与在 Azure AD 中使用资源有何不同，请参阅[范围，而非资源](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources)。

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


## 后续步骤

[将应用注册为使用 v2 身份验证终结点](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/)

## 了解详细信息

[v2 身份验证终结点的最新动态](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare)

[v2 身份验证模型的限制和约束](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

[Microsoft Azure v2 身份验证终结点文档](https://azure.microsoft.com/en-us/documentation/articles/?service=active-directory&term=app+model+v2.0)
