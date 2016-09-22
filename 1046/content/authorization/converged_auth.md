# Autenticar pontos de extremidade do Microsoft Graph usando o ponto de extremidade de autenticação v2


<!--
### Preview documentation
There are features and functionality of the converged authentication model that are not yet supported in the public preview period. You should be aware of them if you are building applications during the public preview. For more information, see [Limitations and restrictions of the converged authentication model preview](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).
-->

## Conectando usuários na conta da Microsoft e no Azure AD com um modelo de autenticação único

Com o ponto de extremidade de autenticação v2 você pode criar aplicativos que aceitam identidades corporativas e de estudante (Azure AD), bem como pessoais (conta da Microsoft).

No passado, para desenvolver um aplicativo para dar suporte a contas da Microsoft e do Azure Active Directory, você precisava se integrar a dois sistemas completamente separados. Usando o ponto de extremidade de autenticação v2, agora você pode dar suporte aos dois tipos de contas com uma única integração. Um processo simples para alcançar imediatamente um público que abrange milhões de usuários com contas pessoais e corporativas/de estudante.   

Depois que você integra os aplicativos ao ponto de extremidade de autenticação v2, eles podem acessar instantaneamente os pontos de extremidade do Microsoft Graph disponíveis para contas pessoais e corporativas/de estudante, como: 

| Dados              | Ponto de extremidade                                       |
|:------------------|:-----------------------------------------------|
| Perfil de usuário      | `https://graph.microsoft.com/v1.0/me`          |
| Email do Outlook      | `https://graph.microsoft.com/v1.0/me/messages` |
| Contatos do Outlook  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Calendários do Outlook | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**Observação:** Alguns pontos de extremidade do Microsoft Graph, como grupos e tarefas, não são aplicáveis a contas pessoais.  

## Escopos de autenticação da API do Microsoft Graph

O ponto de extremidade de autenticação v2 oferece suporte a todos os escopos de permissão listados para uso com a autenticação do Azure AD no tópico [escopos de permissão do Microsoft Graph](permission_scopes.md). No entanto, o ponto de extremidade de autenticação v2 atualmente não oferece suporte a escopos somente do aplicativo.

>**Observação**: atualmente, é necessário passar a URL de recurso "https://graph.microsoft.com" como prefixo para a cadeia de caracteres do escopo. Por exemplo, para usar o escopo `Files.Read`, especifique o escopo como `https://graph.microsoft.com/Files.Read`.

Para saber mais sobre como usar escopos com o ponto de extremidade de autenticação v2 e como ele difere do uso de recursos no Azure AD, confira [Escopos, não recursos](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources).

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


## Próximas etapas

[Registrar um aplicativo para usar o ponto de extremidade de autenticação v2](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/)

## Learn more

[Novidades sobre o modelo de autenticação v2](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare)

[Limitações e restrições no modelo de autenticação v2](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

[Documentação do ponto de extremidade de autenticação do Microsoft Azure v2](https://azure.microsoft.com/en-us/documentation/articles/?service=active-directory&term=app+model+v2.0)
