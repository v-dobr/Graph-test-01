# Области разрешений Microsoft Graph

Microsoft Graph предоставляет разрешения OAuth 2.0, которые используются для управления доступом приложения к данным. Как разработчик вы можете настроить необходимые разрешения для своего приложения. Обычно для этого используется портал Azure. При входе пользователи и администраторы могут разрешить вашему приложению доступ к своим данным в соответствии с настроенными разрешениями. Поэтому следует выбирать разрешения, которые обеспечивают минимальный уровень привилегий, необходимый приложению. Дополнительные сведения о настройке разрешений для приложения и процессе согласия см. в статье [Интеграция приложений с Azure Active Directory](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/).


##Основные понятия, связанные с разрешениями

###Разрешения только для приложений и делегированные разрешения
Есть два типа разрешений: только для приложений и делегированные. Имея разрешение только для приложений (т. н. роль приложений), приложение имеет полный набор прав, предусмотренных этим разрешением. Разрешения только для приложений обычно используются приложениями, которые работают как служба без входа пользователей. Делегированные разрешения предназначены для приложений, в которые входит пользователь. Приложение получает права вошедшего пользователя и может действовать от его имени. Фактические права, предоставляемые приложению, определяются пересечением прав, предусмотренных разрешением, и прав вошедшего пользователя. Например, если разрешение предусматривает делегированные права на запись всех объектов каталога, но вошедший пользователь может только обновлять собственный профиль, приложение сможет записывать только профиль этого пользователя.

###Полный и базовый профили пользователей и групп
Полный профиль (или просто профиль) пользователя или группы включает все объявленные свойства объекта. Так как профиль может содержать конфиденциальную информацию о каталоге или личные сведения, доступ приложений к определенному набору свойств, который называется базовым профилем, ограничен. Для пользователей базовый профиль включает только следующие свойства: отображаемое имя, имя и фамилия, фотография и адрес электронной почты. Для групп базовый профиль содержит только отображаемое имя. 

<!---   <a name="msg_perm_details"> </a>  -->

##Сведения о разрешениях
Чтобы получить необходимые разрешения на доступ к ресурсам API Microsoft Graph, нужно настроить приложение. Разрешения распространяются на отдельные ресурсы и представляют собой права на чтение, запись или обе эти операции. 

В таблицах ниже приведены разрешения API Microsoft Graph, а также описаны предусмотренные ими права доступа. 
- В столбце **Разрешение** указано название разрешения. Названия разрешений представляются в формате "ресурс.операция.ограничение" (например, Group.ReadWrite.All). Если задано ограничение All, приложение может выполнять операцию (ReadWrite) со всеми указанными ресурсами (Group) в каталоге. В противном случае приложение может выполнять эту операцию только в профиле вошедшего пользователя. Разрешения могут предоставлять ограниченные привилегии для определенной операции. Подробные сведения см. в столбце **Описание**.
- В столбце **Разрешение на портале управления Azure** показано, как разрешение отображается на портале Azure. 
- В столбце **Описание** представлено описание всех привилегий, предоставляемых разрешением. В случае делегированных разрешений фактические права доступа, предоставляемые приложению, определяются пересечением прав, предусмотренных разрешением, и привилегий вошедшего пользователя. 
- Разрешения группируются в соответствии с тем, требуется ли для них согласие администратора.

  > **Примечание**. Ограничения для разрешений указаны в [примечаниях к выпуску](http://graph.microsoft.io/docs/overview/release_notes) для `v1.0` и `beta`.
  
###Разрешения, требующие согласия администратора

|   **Область применения**                  |  **Разрешение на портале управления Azure**                          |  **Описание** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read.All_                |     `Read all user's full profiles`           | Не отличается от разрешения User.ReadBasic.All за исключением того, что приложение сможет просматривать полные профили всех пользователей в организации, а также свойства иерархической организации, например руководителя и подчиненных. Полный профиль включает все объявленные свойства объекта **User**. Чтобы просматривать группы, в которых состоит пользователь, приложению также потребуется разрешение Group.Read.All или Group.ReadWrite.All. |
| _User.ReadWrite.All_           |     `Read and write all user's full profiles` | Приложение сможет просматривать и записывать полный набор свойств профиля, подчиненных и руководителей других пользователей в организации от имени вошедшего пользователя. |
| _Directory.Read.All_           |     `Read directory data`                     | Приложение сможет просматривать данные в каталоге организации, например данные пользователей, групп и приложений. |
| _Directory.ReadWrite.All_      |     `Read and write directory data`           | Приложение сможет просматривать и записывать в каталоге организации данные, например сведения о пользователях и группах.  Не разрешается удалять пользователей или группы. Приложение не сможет удалять пользователей или группы, а также сбрасывать пароли пользователей. |
| _Directory.AccessAsUser.All_   |     `Access directory as the signed-in user`  | Приложение получит те же права доступа к данным в каталоге, что и вошедший пользователь.|
| _Group.Read.All_ |    `Read all groups` | Приложение сможет выводить список групп, а также просматривать их свойства и все данные о членстве в группах от имени вошедшего пользователя.  Оно также сможет просматривать календарь, беседы, файлы и другое содержимое всех групп, к которым у вошедшего пользователя есть доступ. |
| _Group.ReadWrite.All_ |    `Read and write all groups`| Приложение сможет создавать группы, а также просматривать все их свойства и данные о членстве от имени вошедшего пользователя.  Кроме того, владельцы групп смогут управлять своими группами, а члены групп — обновлять содержимое групп. |


###Разрешения, не требующие согласия администратора

|   **Область применения**    |  **Разрешение на портале управления Azure**   |  **Описание** |
|:---------------|:------------------|:-----------------|
| _User.Read_       |    `Sign-in and read user profile` | Пользователи смогут входить в приложение, а оно сможет просматривать профили вошедших пользователей. Полный профиль включает все объявленные свойства объекта User. Приложение не сможет просматривать свойства иерархической организации, например руководителя или подчиненных. Приложение также сможет просматривать следующие базовые сведения о компании вошедшего пользователя (через объект **TenantDetail**): идентификатор клиента, отображаемое имя клиента и проверенные домены.|
| _User.ReadWrite_ |    `Read and write access to user profile` | Приложение сможет просматривать ваш профиль. Кроме того, оно сможет обновлять данные вашего профиля от вашего имени. |
| _User.ReadBasic.All_ |    `Read all user's basic profiles` | Приложение сможет просматривать базовые профили всех пользователей в организации от имени вошедшего пользователя. Базовый профиль пользователя включает следующие свойства: отображаемое имя, имя и фамилия, фотография и адрес электронной почты. Чтобы просматривать группы, в которых состоит пользователь, приложению также потребуется разрешение Group.Read.All или Group.ReadWrite.All.| 
| _Mail.Read_ |    `Read user mail` | Приложение сможет просматривать сообщения в почтовых ящиках пользователей.  |
| _Mail.ReadWrite_ |    `Read and write access to user mail` | Приложение сможет создавать, просматривать, обновлять и удалять сообщения в почтовых ящиках пользователей. Не включает разрешение на отправку почты.|
| _Mail.Send_ |    `Send mail as a user` | Приложение сможет отправлять почту от имени пользователей в организации.  |
| _Calendars.Read_ |    `Read user calendars`  | Приложение сможет просматривать события в календарях пользователей.|
| _Calendars.ReadWrite_ |    `Have full access to user calendars`  | Приложение сможет создавать, просматривать, обновлять и удалять события в календарях пользователей. |
| _Contacts.Read_ |    `Read user contacts`  | Приложение сможет просматривать контакты пользователей. |
| _Contacts.ReadWrite_ |    `Have full access to user contacts`  | Приложение сможет создавать, просматривать, обновлять и удалять контакты пользователей. |
| _Files.Read_ |    `Read user files and files shared with user` | Приложение сможет просматривать файлы вошедшего пользователя, и файлы, к которым ему предоставлен доступ.| 
| _Files.ReadWrite_ |   `Have full access to user files and files shared with user` | Приложение сможет просматривать, создавать, обновлять и удалять файлы вошедшего пользователя и файлы, к которым ему предоставлен доступ. |
| _Files.ReadWrite.Selected_ |    `Read and write files that the user selects` | Приложение сможет просматривать и записывать файлы, выбранные пользователем. У приложения будет доступ в течение нескольких часов после выбора файла пользователем. |
| _Files.Read.Selected_ |    `Read files that the user selects`  | Приложение сможет просматривать файлы, выбранные пользователем. У приложения будет доступ в течение нескольких часов после выбора файла пользователем. |
| _Sites.Read.All_ |    `Read items in all site collections` | Приложение сможет просматривать документы и элементы списков во всех семействах веб-сайтов от имени вошедшего пользователя. |
| _openid_ |    `Sign users in` (предварительная версия) | Пользователи смогут входить в приложение с помощью своей рабочей или учебной учетной записи, а приложение сможет просматривать основные данные профилей пользователей.|
| _offline_access_ |    `Access user's data anytime` (предварительная версия) | Приложение сможет просматривать и обновлять данные пользователей, даже если они в настоящее время не используют приложение.|

###Разрешения только для приложений, требующие согласия администратора

|   **Область применения**    |  **Разрешение на портале управления Azure**   |  **Описание** |
|:---------------|:------------------|:-----------------|
| _Mail.Read_       |    `Read mail in all mailboxes` | Приложение сможет просматривать сообщения во всех почтовых ящиках без входа пользователя.|
| _Mail.ReadWrite_ |    `Read and write mail in all mailboxes` | Приложение сможет создавать, просматривать, обновлять и удалять сообщения во всех почтовых ящиках без входа пользователя. Не включает разрешение на отправку почты. |
| _Mail.Send_ |    `Send mail as any user` | Приложение сможет отправлять почту от имени любого пользователя без его входа. | 
| _Calendars.Read_ |    `Read calendars in all mailboxes` | Приложение сможет просматривать события во всех календарях без входа пользователя. |
| _Calendars.ReadWrite_ |    `Read and write calendars in all mailboxes` | Приложение сможет создавать, просматривать, обновлять и удалять события во всех календарях без входа пользователя.|
| _Contacts.Read_ |    `Read contacts in all mailboxes` | Приложение сможет просматривать все контакты во всех почтовых ящиках без входа пользователя. |
| _Contacts.ReadWrite_ |    `Read and write contacts in all mailboxes`  |Приложение сможет создавать, просматривать, обновлять и удалять все контакты во всех почтовых ящиках без входа пользователя.|
| _User.ReadBasic.All_ |    `Read all users' basic profiles`  | Приложение сможет просматривать базовый набор свойств профилей других пользователей в организации без входа пользователя. Включает имя и фамилию, отображаемое имя, фотографию и сообщение об отсутствии на рабочем месте.|
| _User.Read.All_ |    `Read all users' full profiles` | Приложение сможет просматривать полный набор свойств профиля, данные о членстве в группах, сведения о подчиненных и руководителях других пользователей в организации без входа пользователя.| 
| _User.ReadWrite.All_ |   `Read and write all users' full profiles` | Приложение сможет просматривать и записывать полный набор свойств профиля, данные о членстве в группах, сведения о подчиненных и руководителях других пользователей в организации без входа пользователя.|


##Предварительная версия
###Разрешения, не требующие согласия администратора (предварительная версия)

|   **Область применения**    |  **Разрешение на портале управления Azure**   |  **Описание** |
|:---------------|:------------------|:-----------------|
| _Tasks.ReadWrite_ |    `Create, read, update and delete user tasks and plans` (предварительная версия) | Приложение сможет создавать, просматривать, обновлять и удалять задачи и планы (а также задачи в них), назначенные или предоставленные вошедшему пользователю.|
| _People.Read_ |    `Read users' relevant people lists` (предварительная версия) | Приложение сможет просматривать ранжированный список контактов вошедшего пользователя. Список включает локальные контакты, контакты из социальных сетей и каталога вашей организации, а также пользователей, с которыми он недавно общался (например, с помощью электронной почты и Skype).|
| _People.ReadWrite_ |    `Read and write users' relevant people lists` (предварительная версия) | Приложение сможет создавать, просматривать и записывать ранжированный список контактов вошедшего пользователя. Список включает локальные контакты, контакты из социальных сетей и каталога вашей организации, а также пользователей, с которыми он недавно общался (например, с помощью электронной почты и Skype).|
| _Notes.Create_ |    `Create pages in users' notebooks` (предварительная версия) | Приложение сможет просматривать названия записных книжек и разделов, а также создавать страницы, записные книжки и разделы от имени вошедшего пользователя.|
| _Notes.ReadWrite.CreatedByApp_ |    `Limited notebook access` (предварительная версия) | Приложение сможет просматривать названия записных книжек и разделов, а также создавать страницы от имени вошедшего пользователя. Кроме того, приложение сможет просматривать и обновлять созданные им страницы. |
| _Notes.Read_ |    `Read user notebooks` (предварительная версия) | Приложение сможет просматривать названия записных книжек и разделов OneNote, а также просматривать все страницы от имени вошедшего пользователя. Оно не сможет просматривать разделы, защищенные паролем. |
| _Notes.ReadWrite_ |    `Read and write user notebooks` (предварительная версия) | Приложение сможет просматривать названия записных книжек и разделов, все страницы, а также записывать и создавать страницы от имени вошедшего пользователя.  Оно не получит доступ к разделам, защищенным паролем. |
| _Notes.Read.All_ |    `Read all notebooks that the user can access` (предварительная версия) | Приложение сможет просматривать содержимое всех записных книжек и разделов, к которым есть доступ у вошедшего пользователя.   Оно не сможет просматривать разделы, защищенные паролем. |
| _Notes.ReadWrite.All_ |    `Read and write notebooks that the user can access` (предварительная версия) | Приложение сможет просматривать и записывать содержимое всех записных книжек и разделов, доступных вошедшему пользователю.  Оно не получит доступ к разделам, защищенным паролем.|


<!-- -->

##Сценарии использования разрешений
Ниже приведено несколько сценариев с использованием ресурсов `User` и `Group`, а также соответствующие необходимые разрешения. В таблице ниже приведены разрешения, необходимые приложению для выполнения определенных операций. Обратите внимание, что в некоторых случаях способность приложения выполнять некоторые операции зависит от типа разрешения (только для приложений или делегированное), а в случае делегированных разрешений — от прав вошедшего пользователя. 

###Сценарии доступа с использованием ресурса User и необходимые разрешения

| **Задачи приложения, связанные с пользователем**   |  **Требуемые области** | **Разрешения на портале управления Azure** |
|:-------------------------------|:---------------------|:---------------|
| Приложение запрашивает разрешение просматривать основные сведения других пользователей (только отображаемое имя и изображение), например для отображения при выборе людей.   | _User.ReadBasic.All_  |  `Read all user's basic profiles` |
| Приложение запрашивает разрешение просматривать полный профиль вошедшего пользователя (подчиненные, руководитель и т. д).  | _User.Read_ | `Enable sign-in and read user profile`|
| Приложение запрашивает разрешение просматривать полные профили всех пользователей.  | _User.Read.All_ |  `Read all user's full profiles`   |
| Приложение запрашивает разрешение просматривать файлы, почту и данные календаря вошедшего пользователя.  | _User.Read_, _Files.Read_, _Mail.Read_, _Calendar.Read_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read user mail`,  `Read user calendars` |
| Приложение запрашивает разрешение просматривать файлы вошедшего пользователя и файлы, доступ к которым ему предоставили другие пользователи. | _User.Read_, _Files.Read_, _Sites.Read.All_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read items in all site collections` |
| Приложение запрашивает разрешение просматривать и записывать полный профиль вошедшего пользователя.   | _User.ReadWrite_ | `Read and write access to user profile` |
| Приложение запрашивает разрешение просматривать и записывать полные профили всех пользователей.    | _User.ReadWrite.All_ | `Read and write all user's full profiles` |
| Приложение запрашивает разрешение просматривать и записывать файлы, почту и данные календаря вошедшего пользователя.    | _User.ReadWrite_, _Files.ReadWrite_, _Mail.ReadWrite_, _Calendar.ReadWrite_  |  `Read and write access to user profile`,  `Read and write access to user profile`,  `Read and write access to user mail`, `Have full access to user calendars` |
   

###Сценарии доступа с использованием ресурса Group и необходимые разрешения
    
| **Задачи приложения, связанные с группой**  |  **Требуемые области** |  **Разрешения на портале управления Azure** |
|:-------------------------------|:---------------------|:---------------|
| Приложение запрашивает разрешение просматривать основные сведения о группе (только отображаемое имя и изображение), например для отображения при выборе групп.  | _Group.Read.All_  | `Read all groups`|
| Приложение запрашивает разрешение просматривать все содержимое во всех единых группах, в том числе файлы и беседы.  Ему также требуется показывать членов группы и обновлять эти данные (если пользователь — владелец).  |  _Group.Read.All_ | `Read items in all site collections`, `Read all groups`|
| Приложение запрашивает разрешение просматривать и записывать все содержимое во всех единых группах, в том числе файлы и беседы.  Ему также требуется показывать членов группы и обновлять эти данные (если пользователь — владелец).  |  _Group.ReadWrite.All_, _Sites.ReadWrite.All_ |  `Read and write all groups`, `Edit or delete items in all site collections` |
| Приложение запрашивает разрешение на поиск единой группы. Пользователь сможет найти определенную группу, выбрать ее из нумерованного списка, чтобы затем присоединиться к ней.     | _Group.ReadWrite.All_ | `Read and write all groups`|
| Приложение запрашивает разрешение на создание группы с помощью AAD Graph. |   _Group.ReadWrite.All_ | `Read and write all groups`|
 

<!---   ##Additional Resources##


- [Authenticate Microsoft Graph endpoints using the v2.0 app model preview](authenticate-MSGraph-using-v2.md )
- [Hands on lab: Deep dive into the Office 365 unified API](http://dev.office.com/hands-on-labs/4585) -->

