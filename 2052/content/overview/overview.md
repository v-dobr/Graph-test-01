


# Microsoft Graph 概述

Microsoft Graph（旧称“Office 365 统一 API”）通过一个 REST API 终结点从 Microsoft 云服务公开了多个 API (**https://graph.microsoft.com**)。使用 Microsoft Graph，您可以将原来困难或复杂的查询转换成简单导航。 
 
Microsoft Graph 具有以下优势：

- 统一 API 终结点，用于在一个响应中从多个 Microsoft 云服务访问聚合数据 
- 在实体和实体关系之间无缝导航 
- 访问来自 Microsoft 云的智慧和见解

只需使用一个身份验证令牌，即可获得所有这些优势。

您可以使用 API 访问固定实体，如用户、组、邮件、消息、日历、任务和注释（来自 Outlook、OneDrive、Azure Active Directory、Planner、OneNote 及其他服务）。您还可以获得由 Office Graph 强力驱动的计算关系（仅针对商业用户），如您要合作的用户列表或您常用的文档列表。

Microsoft Graph 公开了两个终结点。公开发布的终结点 /v1.0 和预览终结点 /beta。你可以在你的生产应用程序中使用 /v1.0 但不能使用 /beta。我们通过预览终结点 /beta 提供最新功能以便开发人员进行试验并提供反馈，beta 版中的 API 可能会随时更改而且尚未准备好用于生产用途。

<!--<a name="msg_queries"> </a>-->

##常见查询

下面是一些示例，展示了使用 Microsoft Graph API 的常见查询：

| **操作** | **服务终结点** |
|:--------------------------|:----------------------------------------|
|   获取我的个人资料 |    `https://graph.microsoft.com/v1.0/me` |
|   获取我的文件|   `https://graph.microsoft.com/v1.0/me/drive/root/children` |
|   获取我的照片	     | `https://graph.microsoft.com/v1.0/me/photo/$value` |
|   获取我的邮件 |   `https://graph.microsoft.com/v1.0/me/messages` |
|   获取我的高重要性的邮件 | `https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'` |
|   获取我的日历 |   `https://graph.microsoft.com/v1.0/me/calendar` |
|   获取我的经理	  | `https://graph.microsoft.com/v1.0/me/manager` |
|   获取上一个修改文件 foo.txt 的用户 |  `https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser` |
|   获取我隶属于的统一组|   `https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')` |
|   获取我组织中的用户	     | `https://graph.microsoft.com/v1.0/users` |
|   获取群组聊天 |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations` |
|   获取与我相关的人员    | `https://graph.microsoft.com/beta/me/people` |
|   获取我常用的文件 |  `https://graph.microsoft.com/beta/me/trendingAround` |
|   获取与我合作的人员     | `https://graph.microsoft.com/beta/me/workingWith` |
|   获取我的任务    | `https://graph.microsoft.com/beta/me/tasks` |
|   获取我的注释 |  `https://graph.microsoft.com/beta/me/notes/notebooks` |

<!-- <a name="msg_roof"> </a> -->

## 全部 Office 365 数据都在同一个屋檐下

下图展示了 Microsoft Graph 开发者堆栈及其工作方式。

![Microsoft Graph API 开发者堆栈。](./images/MicrosoftGraph_DevStack.png)

 >  我们非常重视您的反馈意见。请在[堆栈溢出](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph)上与我们联系。使用 [MicrosoftGraph] 和 [office365] 标记出您的问题。



