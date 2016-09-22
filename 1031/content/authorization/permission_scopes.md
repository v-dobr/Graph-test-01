# Microsoft Graph-Berechtigungsbereiche

Microsoft Graph macht OAuth 2.0-Berechtigungsbereiche verfügbar, die den Zugriff von Apps auf Daten steuern. Als Entwickler konfigurieren Sie Ihre App mit den Berechtigungsbereichen, die für den benötigten Zugriff erforderlich sind. In der Regel verwenden Sie hierzu das Azure-Portal. Bei der Anmeldung haben Benutzer oder Administratoren die Möglichkeit, zuzustimmen, dass Ihre App auf Daten mit den konfigurierten Berechtigungsbereichen zugreifen kann. Aus diesem Grund sollten Sie Berechtigungsbereiche auswählen, die die niedrigste von Ihrer App benötigte Berechtigungsstufe bereitstellt. Weitere Details zum Konfigurieren von Berechtigungen für Ihre App und zum Zustimmungsprozess finden Sie unter [Integrieren Applikationen in Azure Active Directory](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/).


##Konzepte im Hinblick auf Berechtigungsbereiche

###Nur App verglichen mit delegierten Bereichen
Berechtigungsbereiche können entweder nur für die App gelten oder delegiert sein. Nur-App-Bereiche (auch als App-Rollen bezeichnet) gewähren der App den vollen Satz von in dem Bereich enthaltenen Berechtigungen. Nur-App-Bereiche werden in der Regel von Apps verwendet, die als Dienst ausgeführt werden, ohne dass ein angemeldeter Benutzer vorhanden ist. Delegierte Berechtigungsbereiche sind für Apps gedacht, bei denen sich ein Benutzer anmeldet. Diese Bereiche delegieren die Berechtigungen des angemeldeten Benutzers an die App, sodass die App als angemeldeter Benutzer fungieren kann. Die der App tatsächlich gewährten Berechtigungen ist die Kombination mit den wenigsten Berechtigungen (die Schnittmenge) der vom Berechtigungsbereich gewährten Rechte und der des angemeldeten Benutzers. Wenn z. B. der Berechtigungsbereich delegierte Berechtigungen zum Schreiben aller Verzeichnisobjekte erteilt, der angemeldete Benutzer aber nur über Berechtigungen zum Aktualisieren seines eigenen Benutzerprofils verfügt, kann die App nur das Profil des angemeldeten Benutzers, aber keine anderen Objekte schreiben.

###Vollständige und einfache Profile für Benutzer und Gruppen
Das vollständige Profil (oder das Profil) eines Benutzers oder einer Gruppe umfasst alle deklarierten Eigenschaften der Entität. Da das Profil möglicherweise vertrauliche Informationen oder personenbezogene Informationen (PII) enthält, schränken mehrere Bereiche den App-Zugriff auf eine begrenzten Gruppe von Eigenschaften, die als einfaches Profil bezeichnet werden. Für Benutzer umfasst das einfache Profil nur die folgenden Eigenschaften: Anzeigename, Vor-und Nachname, Foto und E-Mail-Adresse. Für Gruppen enthält das einfache Profil nur den Anzeigenamen. 

<!---   <a name="msg_perm_details"> </a>  -->

##Details zu Berechtigungsbereichen
Sie müssen Ihre App so konfigurieren, dass sie über die erforderlichen Berechtigungen verfügt, um auf die Microsoft Graph-API-Ressourcen zuzugreifen. Die Berechtigungen sind auf einzelne Ressourcen für die Rechte zum Lesen, Schreiben oder beides beschränkt. 

In den folgenden Tabellen Listen sind die Microsoft Graph-API-Berechtigungsbereiche aufgeführt, und es wird der von jeder Berechtigung gewährte Zugriff erläutert. 
- In der Spalte **Umfang** ist der Bereichsname enhalten. Bereichsnamen weisen die Form „resource.operation.constraint“ auf, zum Beispiel „Group.ReadWrite.All“. Bei der Einschränkung „Alles“ wird der App vom Bereich die Möglichkeit gewährt, die Operation (ReadWrite) für alle angegebenen Ressourcen (Gruppe) im Verzeichnis auszuführen. Andernfalls lässt der Bereich die Operation nur in dem Profil des angemeldeten Benutzers zu. Bereiche können eingeschränkte Berechtigungen für den angegebenen Vorgang gewähren; in der Spalte **Beschreibung** finden Sie weitere Details.
- In der Spalte **Berechtigung** wird gezeigt, wie der Bereich im Azure-Portal angezeigt wird. 
- Die Spalte **Beschreibung** beschreibt den vollständigen Satz von Berechtigungen, die von dem Bereich gewährt werden. Bei delegierten Bereichen ist der der App tatsächlich gewährte Zugriff die Kombination mit den wenigsten Berechtigungen (die Schnittmenge) der vom Bereich gewährten Rechte und der des angemeldeten Benutzers. 
- Die Bereiche werden danach gruppiert, ob die Berechtigungen die Zustimmung des Administrators erfordern.

  > **Hinweis**: Informationen zu Einschränkungen von Berechtigungsbereichen finden Sie unter [Versionshinweise](http://graph.microsoft.io/docs/overview/release_notes) für `v1.0` und `beta`.
  
###Berechtigungen, die die Zustimmung des Administrators erfordern

|   **Bereich**                  |  **Berechtigung für Azure-Verwaltungsportal**                          |  **Beschreibung** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read.All_                |     `Read all user's full profiles`           | Identisch mit User.ReadBasic.All, mit der Ausnahme, dass die App das vollständige Profil aller Benutzer in der Organisation lesen kann. Dabei können auch Navigationseigenschaften, z. B. Manager- und direkte Berichte, gelesen werden. Das vollständige Profil enthält alle deklarierten Eigenschaften der **Benutzer**-Entität. Um die Gruppen zu lesen, in denen ein Benutzer Mitglied ist, benötigt die App auch „Group.Read.All“ oder „Group.ReadWrite.All“. |
| _User.ReadWrite.All_           |     `Read and write all user's full profiles` | Die App kann den vollständigen Satz von Profileigenschaften, Berichte und Vorgesetzte von anderen Benutzern in Ihrer Organisation im Namen des angemeldeten Benutzers lesen und schreiben. |
| _Directory.Read.All_           |     `Read directory data`                     | Ermöglicht der App, Daten im Verzeichnis Ihrer Organisation zu lesen, z. B. Benutzer, Gruppen und Apps. |
| _Directory.ReadWrite.All_      |     `Read and write directory data`           | Die App kann Daten im Verzeichnis Ihrer Organisation lesen und schreiben, z. B. Benutzer und Gruppen.  Ermöglicht nicht das Löschen eines Benutzers oder einer Gruppe. Die App kann nicht Benutzer oder Gruppen löschen oder Benutzerkennwörter zurücksetzen. |
| _Directory.AccessAsUser.All_   |     `Access directory as the signed-in user`  | Die App kann den gleichen Zugriff auf Informationen im Verzeichnis wie der angemeldete Benutzer haben.|
| _Group.Read.All_ |    `Read all groups` | Die App kann Gruppen aufführen und deren Eigenschaften sowie alle Gruppenmitgliedschaften im Namen des angemeldeten Benutzers lesen.  Die App kann außerdem, den Kalender, Unterhaltungen, Dateien und andere Gruppeninhalte für alle Gruppen, auf die der Benutzer zugreifen kann, lesen. |
| _Group.ReadWrite.All_ |    `Read and write all groups`| Die App kann Gruppen erstellen und alle Gruppeneigenschaften und -mitgliedschaften im Namen des angemeldeten Benutzers lesen.  Darüber hinaus können Gruppenbesitzer ihre eigenen Gruppen verwalten, und Gruppenmitglieder können Gruppeninhalte aktualisieren. |


###Berechtigungen, die keine Zustimmung des Administrators erfordern

|   **Bereich**    |  **Berechtigung für Azure-Verwaltungsportal**   |  **Beschreibung** |
|:---------------|:------------------|:-----------------|
| _User.Read_       |    `Sign-in and read user profile` | Damit können Benutzer sich bei der App anmelden, und die App kann das Profil von angemeldeten Benutzern lesen. Das vollständige Profil enthält alle deklarierten Eigenschaften der Benutzer-Entität. Die App kann keine Navigationseigenschaften lesen, z. B. Manager- oder direkte Berichte. Ermöglicht außerdem der App das Lesen der folgenden grundlegenden Firmeninformationen des angemeldeten Benutzers (über das **TenantDetail**-Objekt): Mandanten-ID, Mandantenanzeigename und überprüfte Domänen.|
| _User.ReadWrite_ |    `Read and write access to user profile` | Die App kann Ihr Profil lesen. Darüber hinaus kann die App Ihre Profilinformationen in Ihrem Namen aktualisieren. |
| _User.ReadBasic.All_ |    `Read all user's basic profiles` | Ermöglicht der App das Lesen der grundlegenden Profile aller Benutzer in der Organisation im Namen des angemeldeten Benutzers. Das grundlegende Profil eines Benutzers umfasst nur die folgenden Eigenschaften: Anzeigename, Vor- und Nachname, Foto und E-Mail-Adresse. Um die Gruppen zu lesen, in denen ein Benutzer Mitglied ist, benötigt die App auch „Group.Read.All“ oder „Group.ReadWrite.All“.| 
| _Mail.Read_ |    `Read user mail` | Die App kann E-Mails in Benutzerpostfächern lesen.  |
| _Mail.ReadWrite_ |    `Read and write access to user mail` | Die App kann E-Mails in Benutzerpostfächern erstellen, lesen, aktualisieren und löschen. Umfasst nicht über die Berechtigung zum Senden von E-Mails.|
| _Mail.Send_ |    `Send mail as a user` | Die App kann E-Mails im Namen der Benutzer in der Organisation senden.  |
| _Calendars.Read_ |    `Read user calendars`  | Die App kann Ereignisse im Benutzerkalender lesen.|
| _Calendars.ReadWrite_ |    `Have full access to user calendars`  | Die App kann Ereignisse im Benutzerkalender erstellen, lesen, aktualisieren und löschen. |
| _Contacts.Read_ |    `Read user contacts`  | Die App kann Benutzerkontakte lesen. |
| _Contacts.ReadWrite_ |    `Have full access to user contacts`  | Die App kann Benutzerkontakte erstellen, lesen, aktualisieren und löschen. |
| _Files.Read_ |    `Read user files and files shared with user` | Die App kann die Dateien des angemeldeten Benutzers sowie für den Benutzer freigegebene Dateien lesen.| 
| _Files.ReadWrite_ |   `Have full access to user files and files shared with user` | Die App kann die Dateien des angemeldeten Benutzers sowie für den Benutzer freigegebene Dateien lesen, erstellen, aktualisieren und löschen. |
| _Files.ReadWrite.Selected_ |    `Read and write files that the user selects` | Die App kann Dateien, die der Benutzer auswählt, lesen und schreiben. Die App hat mehrere Stunden, nachdem der Benutzer eine Datei ausgewählt hat, Zugriff. |
| _Files.Read.Selected_ |    `Read files that the user selects`  | Die App kann Dateien, die der Benutzer auswählt, lesen. Die App hat mehrere Stunden, nachdem der Benutzer eine Datei ausgewählt hat, Zugriff. |
| _Sites.Read.All_ |    `Read items in all site collections` | Die App kann Dokumente lesen und Elemente in allen Websitesammlungen im Namen des angemeldeten Benutzers aufführen. |
| _OpenID_ |    `Sign users in` (Vorschau) | Damit können Benutzer sich mit Ihren Geschäfts- oder Schulkonten bei der App anmelden, und die App kann grundlegende Benutzerprofilinformationen lesen.|
| _offline_access_ |    `Access user's data anytime` (Vorschau) | Die App kann Benutzerdaten lesen und aktualisieren, auch wenn diese die App derzeit nicht verwenden.|

###Nur-App-Berechtigungen, die die Zustimmung des Administrators erfordern

|   **Bereich**    |  **Berechtigung für Azure-Verwaltungsportal**   |  **Beschreibung** |
|:---------------|:------------------|:-----------------|
| _Mail.Read_       |    `Read mail in all mailboxes` | Die App kann E-Mails in allen Postfächern ohne angemeldeten Benutzer lesen.|
| _Mail.ReadWrite_ |    `Read and write mail in all mailboxes` | Die App kann E-Mails in allen Postfächern ohne angemeldeten Benutzer erstellen, lesen, aktualisieren und löschen. Umfasst nicht über die Berechtigung zum Senden von E-Mails. |
| _Mail.Send_ |    `Send mail as any user` | Die App kann E-Mails als beliebiger Benutzer ohne angemeldeten Benutzer senden. | 
| _Calendars.Read_ |    `Read calendars in all mailboxes` | Die App kann Ereignisse aller Kalender ohne angemeldeten Benutzer lesen. |
| _Calendars.ReadWrite_ |    `Read and write calendars in all mailboxes` | Die App kann Ereignisse aller Kalender ohne angemeldeten Benutzer erstellen, lesen, aktualisieren und löschen.|
| _Contacts.Read_ |    `Read contacts in all mailboxes` | Die App kann alle Kontakte in allen Postfächern ohne angemeldeten Benutzer lesen. |
| _Contacts.ReadWrite_ |    `Read and write contacts in all mailboxes`  |Die App kann alle Kontakte in allen Postfächern ohne angemeldeten Benutzer erstellen, lesen, aktualisieren und löschen.|
| _User.ReadBasic.All_ |    `Read all users' basic profiles`  | Ermöglicht der App, einen einfachen Satz von Profileigenschaften anderer Benutzer in Ihrer Organisation ohne angemeldeten Benutzer zu lesen. Umfasst den Anzeigenamen, den Vor-und Nachnamen, das Foto und Abwesenheitsnotizen.|
| _User.Read.All_ |    `Read all users' full profiles` | Ermöglicht der App, den vollständigen Satz von Profileigenschaften, Gruppenmitgliedschaften, Berichte und Vorgesetzte von anderen Benutzern in Ihrer Organisation ohne angemeldeten Benutzers zu lesen.| 
| _User.ReadWrite.All_ |   `Read and write all users' full profiles` | Ermöglicht der App, den vollständigen Satz von Profileigenschaften, Gruppenmitgliedschaften, Berichten und Vorgesetzten von anderen Benutzern in Ihrer Organisation ohne angemeldeten Benutzer zu lesen und zu schreiben.|


##Vorschau
### Berechtigungen, die keine Zustimmung des Administrators erfordern (Vorschau)

|   **Bereich**    |  **Berechtigung für Azure-Verwaltungsportal**   |  **Beschreibung** |
|:---------------|:------------------|:-----------------|
| _Tasks.ReadWrite_ |    `Create, read, update and delete user tasks and plans` (Vorschau) | Die App kann Aufgaben und Pläne (und darin enthaltene Aufgaben), die dem angemeldeten Benutzer zugewiesen oder für diesen freigegeben sind, erstellen, lesen, aktualisieren und löschen.|
| _People.Read_ |    `Read users' relevant people lists` (Vorschau) | Die App kann eine bewertete Liste relevanter Personen des angemeldeten Benutzers lesen. Die Liste enthält lokale Kontakte, Kontakte aus sozialen Netzwerken, aus dem Verzeichnis Ihrer Organisation und Personen aus kürzlichen Unterhaltungen (z. B. E-Mail und Skype).|
| _People.ReadWrite_ |    `Read and write users' relevant people lists` (Vorschau) | Die App kann eine bewertete Liste relevanter Personen des angemeldeten Benutzers erstellen, lesen und schreiben. Die Liste enthält lokale Kontakte, Kontakte aus sozialen Netzwerken, aus dem Verzeichnis Ihrer Organisation und Personen aus kürzlichen Unterhaltungen (z. B. E-Mail und Skype).|
| _Notes.Create_ |    `Create pages in users' notebooks` (Vorschau) | Die App kann die Titel von Notizbüchern und Abschnitten lesen und neue Seiten, Notizbücher und Abschnitte im Namen des angemeldeten Benutzers erstellen.|
| _Notes.ReadWrite.CreatedByApp_ |    `Limited notebook access` (Vorschau) | Die App kann die Titel von Notizbüchern und Abschnitten lesen und neue Seiten im Namen des angemeldeten Benutzers erstellen. Darüber hinaus kann die App von der App erstellte Seiten lesen und aktualisieren. |
| _Notes.Read_ |    `Read user notebooks` (Vorschau) | Die App kann die Titel von OneNote-Notizbüchern und Abschnitten anzeigen und alle Seiten im Namen des angemeldeten Benutzers lesen. Sie kann keine kennwortgeschützten Abschnitte anzeigen. |
| _Notes.ReadWrite_ |    `Read and write user notebooks` (Vorschau) | Die App kann die Titel von Notizbüchern und Abschnitten lesen, alle Seiten lesen, alle Seiten schreiben und neue Seiten im Namen des angemeldeten Benutzers erstellen.  Sie kann nicht auf kennwortgeschützte Abschnitte zugreifen. |
| _Notes.Read.All_ |    `Read all notebooks that the user can access` (Vorschau) | Die App kann die Inhalte aller Notizbücher und Abschnitte lesen, auf die der Benutzer zugreifen kann.   Sie kann keine kennwortgeschützten Abschnitte lesen. |
| _Notes.ReadWrite.All_ |    `Read and write notebooks that the user can access` (Vorschau) | Die App kann die Inhalte aller Notizbücher und Abschnitte lesen und schreiben, auf die der Benutzer zugreifen kann.  Sie kann nicht auf kennwortgeschützte Abschnitte zugreifen.|


<!-- -->

##Szenarien zu Berechtigungsbereichen
Nachfolgend finden Sie einige App-Szenarien unter Verwendung der Ressourcen `User` und `Group` und den entsprechenden erforderlichen Bereichen. Die folgende Tabelle zeigt die für eine App erforderlichen Berechtigungsbereiche, damit eine App bestimmte Vorgänge ausführen kann. Beachten Sie, dass die Möglichkeit des Ausführens einiger Vorgänge für die App davon abhängig ist, ob der Berechtigungsbereich nur für die App gilt oder delegiert wird, und (im Falle delegierter Berechtigungsbereiche) von den Berechtigungen des angemeldeten Benutzers. 

###Zugriffsszenarien unter Verwendung der User-Ressource und der erforderlichen Bereiche

| **App-Aufgaben, die den Benutzer einbeziehen**   |  **Erforderliche Bereiche** | **Berechtigungen für Azure-Verwaltungsportal** |
|:-------------------------------|:---------------------|:---------------|
| Die App möchte die grundlegenden Informationen von anderen Benutzern lesen (nur Anzeigename und Bild), um diese bei einer Personenauswahl anzuzeigen.   | _User.ReadBasic.All_  |  `Read all user's basic profiles` |
| Die App möchte das vollständige Benutzerprofil für den angemeldeten Benutzer lesen (direkt unterstellte Personen, Vorgesetzte usw.).  | _User.Read_ | `Enable sign-in and read user profile`|
| Die App möchte das vollständige Benutzerprofil aller Benutzer lesen.  | _User.Read.All_ |  `Read all user's full profiles`   |
| Die App möchte Dateien, E-Mails und Kalenderinformationen für den angemeldeten Benutzer lesen.  | _User.Read_, _Files.Read_, _Mail.Read_, _Calendar.Read_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read user mail`,  `Read user calendars` |
| Die App möchte die Dateien des angemeldeten Benutzers (eigene) Dateien und Dateien, die andere Benutzer für den angemeldeten Benutzer (mich) freigegeben haben, lesen. | _User.Read_, _Files.Read_, _Sites.Read.All_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read items in all site collections` |
| Die App möchte das vollständige Benutzerprofil für den angemeldeten Benutzer lesen und schreiben.   | _User.ReadWrite_ | `Read and write access to user profile` |
| Die App möchte das vollständige Benutzerprofil aller Benutzer lesen und schreiben.    | _User.ReadWrite.All_ | `Read and write all user's full profiles` |
| Die App möchte Dateien, E-Mails und Kalenderinformationen für den angemeldeten Benutzer lesen und schreiben.    | _User.ReadWrite_, _Files.ReadWrite_, _Mail.ReadWrite_, _Calendar.ReadWrite_  |  `Read and write access to user profile`,  `Read and write access to user profile`,  `Read and write access to user mail`, `Have full access to user calendars` |
   

###Zugriffsszenarien unter Verwendung der Group-Ressource und der erforderlichen Bereiche
    
| **App-Aufgaben, die die Gruppe einbeziehen**  |  **Erforderliche Bereiche** |  **Berechtigungen für Azure-Verwaltungsportal** |
|:-------------------------------|:---------------------|:---------------|
| Die App möchte die grundlegenden Gruppeninformationen (nur Anzeigename und Bild) lesen, um diese bei einer Gruppenauswahl anzuzeigen.  | _Group.Read.All_  | `Read all groups`|
| Die App möchte alle Inhalte in allen einheitlichen Gruppen lesen, einschließlich Dateien und Unterhaltungen.  Außerdem muss die App Gruppenmitgliedschaften anzeigen und in der Lage sein, Gruppenmitgliedschaften zu aktualisieren (wenn Besitzer).  |  _Group.Read.All_ | `Read items in all site collections`, `Read all groups`|
| Die App möchte alle Inhalte in allen einheitlichen Gruppen lesen und schreiben, einschließlich Dateien und Unterhaltungen.  Außerdem muss die App Gruppenmitgliedschaften anzeigen und in der Lage sein, Gruppenmitgliedschaften zu aktualisieren (wenn Besitzer).  |  _Group.ReadWrite.All_, _Sites.ReadWrite.All_ |  `Read and write all groups`, `Edit or delete items in all site collections` |
| Die App möchte eine einheitliche Gruppe ermitteln (suchen). Der Benutzer kann eine bestimmte Gruppe suchen und aus der Aufzählungsliste eine Gruppe auswählen, damit der Benutzer der Gruppe beitreten kann.     | _Group.ReadWrite.All_ | `Read and write all groups`|
| Die App möchte über AAD Graph eine Gruppe erstellen. |   _Group.ReadWrite.All_ | `Read and write all groups`|
 

<!---   ##Additional Resources##


- [Authenticate Microsoft Graph endpoints using the v2.0 app model preview](authenticate-MSGraph-using-v2.md )
- [Hands on lab: Deep dive into the Office 365 unified API](http://dev.office.com/hands-on-labs/4585) -->

