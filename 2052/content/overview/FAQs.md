
# Microsoft Graph 常见问题 (FAQ)

## Microsoft Graph API 支持哪些平台？
<!--
Apps can use the Microsoft Graph API to perform create, read, update, and delete (CRUD) operations on data sources and entities, giving them seamless access to work data. 

**Ease of use--one endpoint, all Office 365 data under one roof**

You can use the API in four steps:
1.  Select your programming language and development environment.
2.  Build your app.
3.  Optionally, host your app in Microsoft Azure or any cloud platform you choose.
4.  Authenticate your users by using single sign-on with Azure AD.

As a developer you can use the API to create custom apps that access and interact with all the richness of enterprise and productivity data--users, groups, organizational contacts, files, folders, mail, calendar, insights and relationships--and build apps across all mobile, web, and desktop platforms. No matter your development platform or tools. Using a single service endpoint to access those entities and data. And a single authentication flow.  -->

您可以：

<!--Just like in Office 365 APIs, Office 365 unified endpoint API  allows you to build apps using any development environment of your choice:  -->

- 使用你熟悉的任何开发环境，例如 .NET、 PHP、Java、Python 或 Ruby
- 使用任何编程语言、开发平台和托管环境
- 使用任何 Web 语言构建访问 API 的应用，包括 JavaScript、HTML5、Python、Ruby、PHP 和 ASP.NET  
- 使用你选择的 IDE，不管是 Visual Studio、Eclipse、Android Studio、Xcode 或者你选择的其他 IDE
- 在 Microsoft Azure 或任何云平台中托管您的应用
- 为 Windows 通用、iOS、Android 开发应用，或者在另一设备平台上开发应用
- 从 Office 加载项（ Office 的前应用）或 SharePoint 外接程序（ SharePoint 的前应用）调用 API
 


## 为何使用 Microsoft Graph API？

假设你要以编程方式检索用户的文件、个人资料图片，并在你的组织中寻找上次编辑该文件的用户的经理。因为信息存储在多个服务（Azure Active Directory、SharePoint 和 Exchange），所以任务涉及使用 Office 365 API 执行多个步骤： 

1. 使用 Discovery Service 查找各服务终结点 
2. 确定您的 Office 365 应用要连接到的服务的 URL
3. 然后获取并管理每个服务的访问令牌，并直接对服务提出请求

现在，您可以使用 Microsoft Graph API 通过单个 REST API 终结点执行相同的复杂操作。您不需要为每个服务发现和定位不同的终结点、为每个服务获取并管理单独的访问令牌、处理孤立的服务和各种数据模型。

##示例查询

以下示例显示了用于通过不同的服务终结点与 Office 365 API 进行交互的当前模型，以及如果通过 Microsoft Graph API 进行又会变得如何简单。

**不同的服务终结点**

|   **操作**                  |  **API**                          |  **服务终结点** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| 发现 Office 365 API 的服务终结点               |     `Discovery Service`           | _https://_**api.office.com**_/discovery/v1.0/me/services_ |
| 获取用户           |     `Azure AD Graph API` | _https://_**graph.windows.net**_/contoso.com/users?api-version=2013-04-05_|
| 获取收件箱中的邮件集合       |     `Office 365 API`           | _https://_**outlook.office365.com**_/api/v1.0/me/messages_  |
| 获取 Joe 的文件   |     `Office 365 API`  | _https://_**contoso-my.sharepoint.com**_/personal/joe_contoso_com/_api/v1.0/files_ |


使用 Microsoft Graph API，你无需先发现服务终结点，然后遍历不同的终结点来获取用户的文件、邮件等。你只需要与一个 REST URL 命名空间交互，即 _**graph.microsoft.com**_。

**Microsoft Graph API**

|   **操作**                  |  **API**                          |  **服务终结点** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| 发现 Office 365 API 的服务终结点                |     `Microsoft Graph`           | 不需要 |
| 获取用户           |     `Microsoft Graph` | _https://_**graph.microsoft.com**_/v1.0/contoso.onmicrosoft.com/users_ |
| 获取收件箱中的邮件集合       |     `Microsoft Graph`           | _https://_**graph.microsoft.com**_/v1.0/me/messages_  |
| 获取 Joe 的文件   |     `Microsoft Graph `  | _https://_**graph.microsoft.com**_/v1.0/me/drive/root/children_ |


## 使用 Microsoft Graph API 的好处有哪些？

使用 Microsoft Graph API 的一些好处如下所示：

**可以为开发人员使用 Microsoft 云服务带来一致简化的体验**

-   所有服务终结点只要一个命名空间。不需要发现服务终结点。
-   一个令牌可以访问所有资源
-   当前孤立服务之间可以进行集成且直接的定位（例如，获取编写特定文档的用户的部门和管理链）
-   只需要使用一个 API 集，即，只需要使用 Microsoft Graph API 连接到多个服务
-   跨 Office 平台统一和扩展 REST API 和实体 
-   在实体（包括实体之间的导航属性）中实现一致的属性命名和方案

**可以使用 Office 实现更高效率的工作生活体验**

-   上下文相关的内容。例如，按组、项目、小组或趋势查找文档
-   上下文相关的用户关系。例如，您可以按组成员身份、兴趣、技能和专业知识来查找用户。你还可以获取组织图表关系

**支持在任意平台上以任意编程语言开发应用**

-   面向所有开发人员的开发工具和资源。您可以使用任意平台和语言来开发 
-   面向使用开放技术的所有平台的移动开发  
-   不需要任何专业的 Exchange、SharePoint 或 Azure AD 知识来访问 Microsoft Graph API 实体

<!---<a name="msg_v2auth"> </a>-->

## Microsoft Graph API 是否支持 2.0 版应用身份验证终结点？

可以。有关详细信息，请参阅[使用 2.0 版身份验证终结点对 Microsoft Graph 终结点进行身份验证](http://graph.microsoft.io/docs/authorization/converged_auth)。



  > 我们非常重视您的反馈意见。请在[Stack Overflow](http://stackoverflow.com/questions/tagged/office365)上与我们联系。使用 [MicrosoftGraph] 和 [office365] 标记出您的问题。








