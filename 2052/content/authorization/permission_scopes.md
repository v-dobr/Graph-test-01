# Microsoft Graph 权限范围

Microsoft Graph 公开 OAuth 2.0 权限范围，用于控制应用的数据访问权限。作为开发者，您可以为应用配置与所需访问权限相对应的权限范围。通常您可以通过 Azure 门户执行此操作。登录期间，用户或管理员将有机会同意允许您的应用访问属于您配置的权限范围的数据。出于这个原因，您应选择提供应用所需的最小权限级别的权限范围。有关如何为您的应用以及在同意流程中配置权限的详细信息，请参阅[将应用程序与 Azure Active Directory 集成](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/)。


##权限范围的概念

###仅限应用与委托的范围
权限范围可以仅限应用，也可以是委托的范围。仅限应用的范围（也称为应用角色）授予应用该范围提供的整套权限。仅限应用的范围通常由无需登录用户在场的情况下作为服务运行的应用所使用。委派的权限范围是针对用户登录的应用。这些范围将登录用户的权限委托给应用，使应用作为登录用户运行。授予应用程序的实际权限将是范围授予的权限和登录用户所拥有的权限的最小权限组合（交集）。例如，如果权限范围授予写入所有目录对象的委托权限，但登录用户只有更新自身用户个人资料的权限，那么应用将只能写入登录用户的个人资料，而无法写入其他对象。

###用户和组的完整和基本个人资料
用户或组的完整个人资料（或个人资料）包括所有实体的声明属性。因为该个人资料可能包含敏感目录信息或个人身份信息 (PII)，多个范围将限制应用访问有限的属性集，称为基本个人资料。对于用户，基本个人资料仅包括以下属性：显示名称、名字和姓氏、照片和电子邮件地址。对于组，基本个人资料仅包含显示名称。 

<!---   <a name="msg_perm_details"> </a>  -->

##权限范围的详细信息
您必须将您的应用配置为具有必要的权限访问 Microsoft Graph API 资源。各个资源的权限进行了范围划分，读取权限、写入权限或者两者皆有。 

下表列出了 Microsoft Graph API 的权限范围，并说明了每个范围授予的访问权限。 
- **范围**列中列出了范围名称。范围名称采用 resource.operation.constraint 的形式，例如，Group.ReadWrite.All。如果约束为“全部”，则范围将授予应用对目录中的所有指定资源（组）执行操作（读写）的能力；否则，范围仅允许对登录用户的个人资料执行操作。范围可能会为指定的操作授予有限的权限，请参阅**说明**获取详细信息。
- **权限**列显示范围在 Azure 的门户上的显示方式。 
- **说明**列描述由范围授予的完整权限集。对于委托范围而言，授予应用程序的实际访问权限将是范围授予的访问权限和登录用户的权限的最小权限组合（交集）。 
- 根据权限是否需要管理员的许可，对域进行分组。

  > **注意**：请参阅[发行说明](http://graph.microsoft.io/docs/overview/release_notes)了解 `v1.0` 和 `beta` 的权限范围限制。
  
###需要管理员同意的权限

|   **范围**                  |  **Azure 管理门户上的权限**                          |  **说明** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read.All_                |     `Read all user's full profiles`           | 与 User.ReadBasic.All 相同，区别是它允许应用读取组织中所有用户的完整个人资料，并且读取经理和直接下属等导航属性时，完整个人资料包括 **User** 实体所有声明的属性。若要读取用户所在的组，应用还将需要 Group.Read.All 或 Group.ReadWrite.All。 |
| _User.ReadWrite.All_           |     `Read and write all user's full profiles` | 允许应用代表登录用户读取和写入组织中其他用户的整套个人资料属性、下属和经理。 |
| _Directory.Read.All_           |     `Read directory data`                     | 允许应用读取组织目录中的数据，如用户、组和应用。 |
| _Directory.ReadWrite.All_      |     `Read and write directory data`           | 允许应用读取和写入组织目录中的数据，如用户和组。不允许删除用户或组。它不允许应用删除用户或组，或重置用户密码。 |
| _Directory.AccessAsUser.All_   |     `Access directory as the signed-in user`  | 允许应用与登录用户对目录中的信息具有相同的访问权限。|
| _Group.Read.All_ |    `Read all groups` | 允许应用代表登录用户列出组，并读取其属性以及所有组成员身份。此外，还允许应用读取登录用户可以访问的所有组的日历、 对话、 文件和其他组内容。 |
| _Group.ReadWrite.All_ |    `Read and write all groups`| 允许应用代表登录用户创建组并读取所有组属性和成员身份。此外，还允许组所有者管理他们的组并允许组成员更新组内容。 |


###不需要管理员同意的权限

|   **范围**    |  **Azure 管理门户上的权限**   |  **说明** |
|:---------------|:------------------|:-----------------|
| _User.Read_       |    `Sign-in and read user profile` | 允许用户登录应用，并允许应用读取登录用户的个人资料。整套个人资料包括“User”实体所有声明的属性。应用无法读取经理或直接下属导航属性。此外，还允许应用读取登录用户的以下基本公司信息（通过 **TenantDetail** 对象）：租户 ID、租户显示名称和已验证的域。|
| _User.ReadWrite_ |    `Read and write access to user profile` | 允许应用读取您的个人资料。此外，它还允许应用代表您更新您的个人资料。 |
| _User.ReadBasic.All_ |    `Read all user's basic profiles` | 允许应用代表登录用户读取组织中所有用户的基本个人资料。以下属性包括用户的基本个人资料：显示名称、名字和姓氏、照片和电子邮件地址。若要读取用户所在的组，应用还将需要 Group.Read.All 或 Group.ReadWrite.All。| 
| _Mail.Read_ |    `Read user mail` | 允许应用读取用户邮箱中的电子邮件。  |
| _Mail.ReadWrite_ |    `Read and write access to user mail` | 允许应用创建、读取、更新和删除用户邮箱中的电子邮件。不包括发送电子邮件的权限。|
| _Mail.Send_ |    `Send mail as a user` | 允许应用以组织用户身份发送邮件。 |
| _Calendars.Read_ |    `Read user calendars`  | 允许应用读取用户日历中的事件。|
| _Calendars.ReadWrite_ |    `Have full access to user calendars`  | 允许应用创建、读取、更新和删除用户日历中的事件。 |
| _Contacts.Read_ |    `Read user contacts`  | 允许应用读取用户联系人。 |
| _Contacts.Read_ |    `Have full access to user contacts`  | 允许应用创建、读取、更新和删除用户联系人。 |
| _Files.Read_ |    `Read user files and files shared with user` | 允许应用读取登录用户的文件以及与该用户共享的文件。| 
| _Files.ReadWrite_ |   `Have full access to user files and files shared with user` | 允许应用读取、创建、更新和删除登录用户的文件以及与该用户共享的文件。 |
| _Files.ReadWrite.Selected_ |    `Read and write files that the user selects` | 允许应用读取和写入用户选择的文件。用户选择文件后，应用具有几个小时的访问权限。 |
| _Files.Read.Selected_ |    `Read files that the user selects`  | 允许应用读取用户选择的文件。用户选择文件后，应用具有几个小时的访问权限。 |
| _Sites.Read.All_ |    `Read items in all site collections` | 允许应用程序代表登录用户读取文档，并列出所有网站集中的项目。 |
| _openid_ |    `Sign users in`（预览） | 允许用户以其工作或学校帐户登录应用，并允许应用查看用户的基本个人资料信息。|
| _offline_access_ |    `Access user's data anytime`（预览） | 允许应用读取和更新用户数据，即使他们目前没有使用该应用。|

###需要管理员同意的仅限应用权限

|   **范围**    |  **Azure 管理门户上的权限**   |  **说明** |
|:---------------|:------------------|:-----------------|
| _Mail.Read_       |    `Read mail in all mailboxes` | 允许应用在没有登录用户的情况下读取所有邮箱中的邮件。|
| _Mail.ReadWrite_ |    `Read and write mail in all mailboxes` | 允许应用在没有登录用户的情况下创建、读取、更新和删除所有邮箱中的邮件。不包括发送电子邮件的权限。 |
| _Mail.Send_ |    `Send mail as any user` | 允许应用在没有登录用户的情况下以任何用户发送邮件。 | 
| _Calendars.Read_ |    `Read calendars in all mailboxes` | 允许应用在没有登录用户的情况下读取所有日历的事件。 |
| _Calendars.ReadWrite_ |    `Read and write calendars in all mailboxes` | 允许应用在没有登录用户的情况下创建、读取、更新和删除所有日历的事件。|
| _Contacts.Read_ |    `Read contacts in all mailboxes` | 允许应用在没有登录用户的情况下读取所有邮箱中的所有联系人。 |
| _Contacts.Read_ |    `Read and write contacts in all mailboxes`  |允许应用在没有登录用户的情况下创建、读取、更新和删除所有邮箱中的所有联系人。|
| _User.ReadBasic.All_ |    `Read all users' basic profiles`  | 允许应用在没有登录用户的情况下读取组织中其他用户的一套基本个人资料属性。其中包括显示名称、名字和姓氏、照片和公司外的邮件。|
| _User.Read.All_ |    `Read all users' full profiles` | 允许应用在没有登录用户的情况下读取组织中其他用户的整套个人资料属性、组成员身份、下属和经理。| 
| _User.ReadWrite.All_ |   `Read and write all users' full profiles` | 允许应用在没有登录用户的情况下读取和写入组织中其他用户的整套个人资料属性、组成员身份、下属和经理。|


##预览
###不需要管理员同意的权限（预览）

|   **范围**    |  **Azure 管理门户上的权限**   |  **说明** |
|:---------------|:------------------|:-----------------|
| _Tasks.ReadWrite_ |    `Create, read, update and delete user tasks and plans`（预览） | 允许应用创建、读取、更新和删除分配给登录用户或与登录用户共享的任务和计划（以及项目中的任务）。|
| _People.Read_ |    `Read users' relevant people lists`（预览） | 允许应用读取登录用户相关人员的排名列表。该列表包括当地联系人、来自社交网络的联系人、您所在组织的目录以及来自最近通信（例如电子邮件和 Skype）的人员。|
| _People.ReadWrite_ |    `Read and write users' relevant people lists`（预览） | 允许应用创建、读取和写入登录用户相关人员的排名列表。该列表包括当地联系人、来自社交网络的联系人、您所在组织的目录以及来自最近通信（例如电子邮件和 Skype）的人员。|
| _Notes.Create_ |    `Create pages in users' notebooks`（预览） | 允许应用代表登录用户读取笔记本和分区标题并创建新的页面、笔记本和分区。|
| _Notes.ReadWrite.CreatedByApp_ |    `Limited notebook access`（预览） | 允许应用代表登录用户读取笔记本和分区标题、创建新的页面。此外，还允许应用读取和更新应用所创建的页面。 |
| _Notes.Read_ |    `Read user notebooks`（预览） | 允许应用代表登录用户查看 OneNote 笔记本和分区标题并读取所有页面。它不能查看受密码保护的分区。 |
| _Notes.ReadWrite_ |    `Read and write user notebooks`（预览） | 允许应用代表登录用户读取笔记本和分区标题、读取所有页面、写入所有页面并创建新的页面。 它不能访问受密码保护的分区。 |
| _Notes.Read.All_ |    `Read all notebooks that the user can access`（预览） | 允许应用读取登录用户可以访问的所有笔记本和分区的内容。 它不能读取受密码保护的分区。 |
| _Notes.ReadWrite.All_ |    `Read and write notebooks that the user can access`（预览） | 允许应用读取和写入登录用户可以访问的所有笔记本和分区的内容。它不能访问受密码保护的分区。|


<!-- -->

##权限范围的场景
以下是使用 `User` 和 `Group` 资源及其对应的所需范围的一些应用场景。下表显示了应用能够执行特定操作所需的权限范围。请注意，在某些情况下，应用能否执行某些操作将取决于权限范围是仅限应用还是委托的范围，并且如果是委托的权限范围，则取决于登录用户的权限。 

###使用用户资源和所需范围的访问场景

| **涉及用户的应用任务**   |  **必需的作用域** | **Azure 管理门户上的权限** |
|:-------------------------------|:---------------------|:---------------|
| 应用想要读取其他用户的基本信息（仅限显示名称和图片），例如展示人员挑选经验   | _User.ReadBasic.All_  |  `Read all user's basic profiles` |
| 应用想要读取登录用户的完整用户个人资料（请参见直接下属和经理等)  | _User.Read_ | `Enable sign-in and read user profile`|
| 应用想要读取所有用户的完整用户个人资料  | _User.Read.All_ |  `Read all user's full profiles`   |
| 应用想要读取登录用户的文件、邮件和日历信息  | _User.Read_, _Files.Read_, _Mail.Read_, _Calendar.Read_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read user mail`,  `Read user calendars` |
| 应用想要读取登录用户（我）的文件，以及其他用户与登录用户（我）共享的文件。 | _User.Read_, _Files.Read_, _Sites.Read.All_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read items in all site collections` |
| 应用想要读取和写入登录用户的完整用户个人资料   | _User.ReadWrite_ | `Read and write access to user profile` |
| 应用想要读取和写入所有用户的完整用户个人资料    | _User.ReadWrite.All_ | `Read and write all user's full profiles` |
| 应用想要读取和写入登录用户的文件、邮件和日历信息    | _User.ReadWrite_, _Files.ReadWrite_, _Mail.ReadWrite_, _Calendar.ReadWrite_  |  `Read and write access to user profile`,  `Read and write access to user profile`,  `Read and write access to user mail`, `Have full access to user calendars` |
   

###使用组资源和所需范围的访问场景
    
| **涉及组的应用任务**  |  **必需的作用域** |  **Azure 管理门户上的权限** |
|:-------------------------------|:---------------------|:---------------|
| 应用想要读取基本组信息（仅限显示名称和图片），例如展示组挑选经验  | _Group.Read.All_  | `Read all groups`|
| 应用想要读取所有统一组的所有内容，包括文件、对话。它还需要显示组成员身份，能够更新组成员身份（如果是所有者）。  |  _Group.Read.All_ | `Read items in all site collections`, `Read all groups`|
| 应用想要读取和写入所有统一组中的所有内容，包括文件、对话。它还需要显示组成员身份，能够更新组成员身份（如果是所有者）。  |  _Group.ReadWrite.All_, _Sites.ReadWrite.All_ |  `Read and write all groups`, `Edit or delete items in all site collections` |
| 应用想要发现（找到）统一组。它允许用户搜索特定组，然后从枚举列表中选择一个用户以允许该用户加入组。     | _Group.ReadWrite.All_ | `Read and write all groups`|
| 应用想要通过 AAD Graph 创建一个组 |   _Group.ReadWrite.All_ | `Read and write all groups`|
 

<!---   ##Additional Resources##


- [Authenticate Microsoft Graph endpoints using the v2.0 app model preview](authenticate-MSGraph-using-v2.md )
- [Hands on lab: Deep dive into the Office 365 unified API](http://dev.office.com/hands-on-labs/4585) -->

