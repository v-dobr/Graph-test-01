# Authentifizieren von Microsoft Graph-Endpunkten mithilfe des v2-Authentifizierungsendpunkts


<!--
### Preview documentation
There are features and functionality of the converged authentication model that are not yet supported in the public preview period. You should be aware of them if you are building applications during the public preview. For more information, see [Limitations and restrictions of the converged authentication model preview](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).
-->

## Anmelden des Microsoft-Kontos und von Azure AD-Benutzern mit einem einzelnen Authentifizierungsmodell

Mit dem v2-Authentifizierungsendpunkt können Sie Apps erstellen, für die sowohl Geschäfts- und Schulkonten- (Azure AD) als auch persönliche Identitäten (Microsoft-Konto) verwendet werden können.

In der Vergangenheit mussten Sie, wenn Sie eine App mit Unterstützung für Microsoft-Konten und Azure Active Directory entwickeln wollten, zwei separate Systeme integrieren. Jetzt können Sie mithilfe des v2-Authentifizierungsendpunkts beide Arten von Konten in einer einzigen Integration unterstützen. Mit einem einfachen Prozess können Sie direkt eine Zielgruppe erreichen, die Millionen von Benutzern mit persönlichen und Geschäfts-/Schulkonten umfasst.   

Nachdem Sie Ihre Apps mit dem Authentifizierungsendpunkt v2 integriert haben, können Sie sofort auf die Microsoft Graph-Endpunkte für private und Geschäfts-/Schulkonten zugreifen, zum Beispiel: 

| Daten              | Endpunkt                                       |
|:------------------|:-----------------------------------------------|
| Benutzerprofil      | `https://graph.microsoft.com/v1.0/me`          |
| Outlook-Mail      | `https://graph.microsoft.com/v1.0/me/messages` |
| Outlook-Kontakte  | `https://graph.microsoft.com/v1.0/me/contacts` |
| Outlook-Kalender | `https://graph.microsoft.com/v1.0/me/events`   |
| OneDrive          | `https://graph.microsoft.com/v1.0/me/drive`    |

 >**Hinweis:** Einige Microsoft Graph-Endpunkte, wie z. B. Gruppen und Aufgaben, gelten nicht für private Konten.  

## Authentifizierungsbereiche der Microsoft Graph-API

Der v2-Authentifizierungsendpunkt unterstützt alle Berechtigungsbereiche, die für die Verwendung mit der Azure AD-Authentifizierung im Thema [Microsoft Graph-Berechtigungsbereiche](permission_scopes.md) aufgeführt sind. Der v2-Authentifizierungsendpunkt unterstützt derzeit jedoch keine Nur-App-Bereiche.

>**Hinweis**: Derzeit ist es erforderlich, die Ressourcen-URL „https://graph.microsoft.com“ als Präfix der Bereichszeichenfolge zu übergeben. Um beispielsweise den Bereich `Files.Read` zu verwenden, müssen Sie den Bereich als `https://graph.microsoft.com/Files.Read` angeben.

Weitere Informationen zur Verwendung von Bereichen mit dem v2-Authentifizierungsendpunkt und zu Unterschieden zur Verwendung von Ressourcen in Azure AD finden Sie unter [Bereiche, keine Ressourcen](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare/#scopes-not-resources).

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


## Nächste Schritte

[Registrieren einer App zum Verwenden des c2-Authentifizierungsendpunkts](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-app-registration/)

## Weitere Informationen

[Neuheiten beim v2-Authentifizierungsmodell](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-compare)

[Einschränkungen des v2-Authentifizierungsmodells](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)

[Dokumentation zum Microsoft Azure v2-Authentifizierungsendpunkt](https://azure.microsoft.com/en-us/documentation/articles/?service=active-directory&term=app+model+v2.0)
