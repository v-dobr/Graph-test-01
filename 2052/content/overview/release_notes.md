# Microsoft Graph 发行说明和已知问题

本文介绍了 2015 年 11 月发行的 Microsoft Graph API 中提供的面向开发者的新功能，以及任何您可能需要注意的已知问题。 


## Microsoft Graph API 中的 GA 功能

以下 Microsoft Graph API 功能已全面推出：

* 用户
* 组
* 文件
* 邮件
* 日历
* 个人联系人 
* 创建、读取、更新和删除 (CRUD) 操作
* 跨域资源共享 (CORS) 支持。

    
## Microsoft Graph API 中的预览功能

以下 Microsoft Graph API 预览功能已全面推出：

* 注释 
* 任务
* Excel
* 人脉
* 组织联系人
* 应用程序
* 服务主体
* 目录架构扩展
* Webhook
* 数据解析和关系：常用文档和合作用户
* 创建后，组可即时访问内容
* 个人帐户的聚合身份验证模型以及工作和学校帐户。 虽然这是一项预览功能，但已经在 `/v1.0` 和 `/beta` 这两个版本上推出。


## Microsoft Graph 已知问题

下面介绍了 Microsoft Graph 的已知问题。

### 用户
#### 创建后无法即时访问
可通过在用户实体上使用 POST 来即时创建用户。必须先向用户分配 Office 365 许可证，然后用户才能访问 Office 365 服务。尽管如此，由于服务具有分散特性，因此用户可能需要先等待 15 分钟，然后才能通过 Microsoft Graph API 使用文件、邮件和事件实体。在此期间，应用会收到一个 404 HTTP 错误响应。 

#### 照片限制
只有当用户有邮箱时，才能读取和更新用户的个人资料照片。另外，之前*可能*使用 **thumbnailPhoto** 属性（使用 Office 365 统一 API 预览或 Azure AD Graph，或通过 AD Connect 同步）存储的所有照片无法再通过 Microsoft Graph 用户照片属性进行访问了。在这种情况下，无法读取或更新照片会生成以下错误：

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **注意**：在 GA 推出后，很快就会启用用户个人资料照片的存储和检索（即使用户没有邮箱，也会启用），而且此错误应该也会消失。

#### 默认联系人文件夹

在 `/v1.0` 版本中，`GET /me/contactFolders` 不包括用户的默认联系人文件夹。 

将会提供修复程序。同时，您还可以使用以下[列出联系人](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts)查询和 **parentFolderId** 属性作为一种解决方法，
来获取默认联系人文件夹的文件夹 ID：

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
在上面的查询中：
1. `/me/contacts?$top=1` 获取默认联系人文件夹中 [联系人](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact) 的属性。
2. 附加 `&$select=parentFolderId` 将仅返回联系人的 **parentFolderId** 属性，它是默认联系人文件夹的 ID。

#### 在用户邮箱中添加和访问基于 ICS 的日历
目前，还有部分支持基于 Internet 日历订阅 (ICS) 的日历：
* 你可以通过用户界面，而不是通过 Microsoft Graph API 为用户邮箱添加基于 ICS 的日历。 
* [列出用户的日历](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars) 允许你获取用户默认日历组中或指定日历组中的每个 
[日历](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar) 的**名称**、**颜色**和 **id** 属性，包括所有基于 ICS 的日历。 
你无法存储或访问日历资源中的 ICS URL。
* 你还可以 [列出基于 ICS 的日历事件](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events)。

### 组
#### 策略
使用 Microsoft Graph 创建并命名统一组会跳过通过 Outlook Web App 配置的所有统一组策略。 

#### 组权限范围
Microsoft Graph 公开了两个权限范围（*Group.Read.All* 和 *Group.ReadWrite.All*）用于访问组 API。  这些权限范围必须征得管理员同意（与预览不同的地方）。  我们计划在将来添加新的可被用户同意的组范围。

#### 添加和获取组文章的附件
向组帖子 [添加](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments) 附件、
[列出](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments) 和获取组帖子的附件目前返回错误消息“OData 请求不受支持”。 
已经为 `/v1.0` 和 `/beta` 版本推出修复程序，并预计到 2016 年 1 月底会广泛推出。

### 联系人
* 目前只支持个人联系人。 `/v1.0` 中目前暂不支持组织联系人，但可以在 `/beta` 中找到组织联系人。
* 个人联系人的移动电话不会返回，但很快就会添加。同时，可以通过 Outlook API 访问它。

### 驱动器、文件和内容流式传输
* 在通过浏览器访问个人站点之前，用户首次通过 Microsoft Graph 访问个人驱动器会生成 401 响应。
* 上载和下载的文件（Office 组、驱动器或邮件附件中的文件）大小上限为 4 MB。

### 仅在现有 Office 365 REST API 中可用的功能
#### 同步
在 `/v1.0` 或 `/beta` 中，Outlook、OneDrive 和 Azure AD 同步功能（在 Azure AD 中，这也称为“差异查询”）均不可用。  如果您的应用程序需要使用同步功能，请继续使用现有的 Office 365 和 Azure AD REST API，或浏览通过 Microsoft Graph 为事件、邮件和联系人提供的新 webhook 预览功能。

> **注意**：目标是尽快缩小现有 API 与 Microsoft Graph 之间的差距（包括同步）。

#### 批处理
Microsoft Graph 不支持批处理。 不过，你可以使用 Outlook beta 版本终结点和 
[批处理 Outlook REST 调用](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests)。 

#### 在中国可用的功能
Microsoft Graph 服务由世纪互联运营（现可在中国使用）。请查看 [Microsoft Graph sovereign 云部署](http://graph.microsoft.io/docs/overview/deployments)了解包括限制在内的更多详细信息。

#### 服务操作和函数
`isMemberOf` 和 `getObjectsById` 在 Microsoft Graph 中不可用。

### Microsoft Graph 权限
请查阅 [权限范围主题](http://graph.microsoft.io/docs/authorization/permission_scopes)，详细了解 Microsoft Graph 支持的应用程序和委托的权限的最新详细信息。  此外，`v1.0` 还存在以下限制：

|权限 |   权限类型 | 限制 |  替代方法 |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| 委托    | 无法更新移动电话号码|    同时选择 `Directory.AccessAsUser.All`| 
|_User.ReadWrite.All_|  委托|  无法对 `User` 执行任何 CRUD 操作（更新用户 HD 照片和扩展配置文件属性除外）| 如果必须删除用户，则同时选择 `Directory.ReadWrite.All` 或 `Directory.AccessAsUser.All`。|
|_User.Read.All_|   应用程序 |无法对其他用户执行任何读取操作| 同时选择 `Directory.Read.All`|
| _User.ReadWrite.All_ |    应用程序 |   无法对 `User` 执行任何 CRUD 操作（更新用户 HD 照片和扩展配置文件属性除外） |    同时选择 `Directory.ReadWrite.All` **注意**：无法删除用户。|
|_Group.Read.All_   | 应用程序 | 无法枚举组或组成员身份。  仍可以读取 Office 组的组内容   | 同时选择 `Directory.Read.All` |
|_Group.ReadWrite.All_  | 应用程序   | 无法枚举组或组成员身份、创建组、更新组成员身份或删除组。仍可读取和更新 Office 组的组内容。	   | 同时选择 `Directory.ReadWrite.All` **注意**：无法删除组。|

此外，`/beta` 也存在以下限制：

|权限 |   权限类型 | 限制 |  替代方法 |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | 委托 | 无法读取或更新 Office 组中的规划器任务	  | 同时选择 `Tasks.ReadWrite`|
|_Tasks.ReadWrite_  | 委托 | 无法读取或更新登录用户的任务| 同时选择 `Group.ReadWrite.All`|

### OData 相关限制
* **$expand** 限制： 
 * 不支持 `nextLink`
 * 不支持 1 级以上扩展
 * 不支持其他参数（**$filter**、**$select**）
* 不支持多个命名空间
* 在用户、组、设备、服务主体和应用程序上，不支持对 `$ref` 执行获取操作和投射。
* 不支持 `@odata.bind`。  也就是说，开发者无法正确地在组上设置 `Accepted` 或 `RejectedSenders`。
* 使用极少的元数据时，非包容导航（如邮件）上不存在 `@odata.id`
* 跨工作负载筛选/搜索不可用。 
* 全文搜索（使用 **$search**）仅对某些实体（如邮件）可用。

  >  我们非常重视您的反馈意见。 请在 [Stack Overflow](http://stackoverflow.com/questions/tagged/office365) 上与我们联系。 使用 [MicrosoftGraph] 和 [office365] 标记出您的问题。

  
             .

