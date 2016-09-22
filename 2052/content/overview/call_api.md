# 调用 Microsoft Graph API

若要访问和操纵 Microsoft Graph 资源，请使用以下一项操作来调用和指定资源 URL。   

- GET
- 发布
- 修补
- 放置
- 删除 

所有 Microsoft Graph API 请求均使用下列基 URL 模式：

```
    https://graph.microsoft.com/{version}/{resource}?[odata_query_parameters]
```

对于此 URL：

- `https://graph.microsoft.com` 是 Microsoft Graph API 终结点。
- `{version}` 是目标服务版本（例如 `v1.0` 或 `beta`）。
- `{resource}` 是资源段或路径，如：
  - `users`, `groups`, `devices`, `organization`
  - 解析为登录用户的别名 `me`
   - 属于用户的资源（如 `me/events`、`me/drive` 或 `me/messages`）
  - 解析为登录用户所属组织的租户的别名 `myOrganization`
- `[odata_query_parameters]` 代表其他 OData 查询参数（如 `$filter` 和 `$select`）。

或者，也可以将租户指定为请求的一部分。 使用 `me` 时，请勿指定租户。 有关常见请求的列表，请参阅 [Microsoft Graph 概述](overview.md)。

##Microsoft Graph API 元数据
服务文档 ($metadata) 在服务根处发布。例如，您可以通过以下 URL 查看 v1.0 和 beta 版本的服务文档。

Microsoft Graph API `v1.0` 元数据。
```
    https://graph.microsoft.com/v1.0/$metadata
```
Microsoft Graph API `beta` 元数据。
```
    https://graph.microsoft.com/beta/$metadata
```

借助元数据，可以查看实体、实体类型和实体集、复杂类型和 Microsoft Graph API 的枚举。 使用元数据和现成的第三方工具，可以创建序列化请求并生成 REST API 简化使用的客户端库。  

资源 URL 由 Microsoft Graph 实体数据模型确定。 实体元数据构架 ($metadata) 中概述了惯例。 

路径 URL 资源名称、查询参数以及操作参数和值不区分大小写。
不过，你分配的值、实体 ID 和其他 base64 编码值均区分大小写。

以下部分介绍了几个对 Microsoft Graph API 的基本编程模式调用。

##从集导航至成员

若要查看用户相关信息，可以使用一个 HTTPS GET 请求从 `users` 集合至标识符标识的指定用户获取 `User` 实体。 对于 `User` 实体，可以将 `id` 或 `userPrincipalName` 属性用作标识符。 以下示例请求使用 `userPrincipalName` 值作为用户 ID。 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com HTTP/1.1
Authorization : Bearer <access_token>
```

如果成功，应该会收到 200 OK 响应，其中包含有效负载中的用户资源声明，如下所示：

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 982

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
    "city": "Redmond",
    "country": "USA",
    "department": "Help Center",
    "displayName": "John Doe",
    "givenName": "John",
    "userPrincipalName": "Johndoe@contoso.onmicrosoft.com",

    ... 
}
```


##从一个实体投影至属性
若要仅检索用户的传记数据（如用户提供的_本人简介_描述和技能集），则可以在上一个请求中添加 $select 查询选项。 例如：

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com?$select=displayName,aboutMe,skills HTTP/1.1
Authorization : Bearer <access_token>
```

成功的响应返回 200 OK 状态，以及具有如下格式的有效负载：

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 169

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "aboutMe": "A cool and nice guy.",
    "displayName": "John Doe",
    "skills": [
        "n-Lingual",
        "public speaking",
        "doodling"
    ]
}
```

此时，仅返回 `user`、`aboutMe` 和 `displayName` 属性（而不是 `skills` 实体上的整个属性集）。

##通过关系遍历到其他资源
经理与向其报告的其他用户保持 `directReports` 关系。若要查询用户的直接下属列表，则你可以使用以下 HTTPS GET 请求，通过关系遍历，导航到预期目标。 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com/directReports HTTP/1.1
Authorization : Bearer <access_token>
```

成功的响应返回 200 OK 状态，以及具有如下格式的有效负载：

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 152
    
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#directoryObjects/$entity",
    "@odata.type": "#microsoft.graph.user",
    "id": "c37b074d-fe9d-4e68-83ad-b4401d3be174",
    "department": "Sales & Marketing",
    "displayName": "Bonnie Kearney",

    ...
}
```

同样地，可以根据关系导航至相关资源。 例如，借助 `user => messages` 关系，可以从 Azure AD 节点遍历到 Exchange Online 节点。 下面的示例显示如何在 REST API 调用中完成这一点：


```no-highlight 
GET https://graph.microsoft.com/v1.0/me/messages HTTP/1.1
Authorization : Bearer <access_token>
```

    
成功的响应返回 200 OK 状态，以及具有如下格式的有效负载：


```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
odata-version: 4.0
content-length: 147
    
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/Messages",
  "@odata.nextLink": "https://graph.microsoft.com/v1.0/me/messages?$top=1&$skip=1",
  "value": [
    {
      "@odata.etag": "W/\"FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej\"",
      "id": "<id-value>",
      "createdDateTime": "2015-11-14T00:24:42Z",
      "lastModifiedDateTime": "2015-11-14T00:24:42Z",
      "changeKey": "FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej",
      "categories": [],
      "receivedDateTime": "2015-11-14T00:24:42Z",
      "sentDateTime": "2015-11-14T00:24:28Z",
      "hasAttachments": false,
      "subject": "Did you see last night's game?",
      "body": {
        "ContentType": "HTML",
        "Content": "<content>"
      },
      "BodyPreview": "it was great!",
      "Importance": "Normal",
            
       ...
    }
  ]
}
```

##从多个实体投影至属性
除了能从一个实体投影至其属性之外，您还可以将类似的 `$select` 查询选项应用于实体集合，以便将它们投影至其一些属性的集合。例如，若要查询登录用户的驱动器项目名称，则您可以提交以下 HTTPS GET 请求：

```no-highlight 
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name HTTP/1.1
Authorization : Bearer <access_token>
```

成功的响应返回 200 OK 状态代码，以及包含共享文件的名称和类型的有效负载，如以下示例所示：

```no-highlight 
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/drive/root/children(name,type)",
  "value": [
    {
      "@odata.etag": "\"{896A8E4D-27BF-424B-A0DA-F073AE6570E2},2\"",
      "name": "Shared with Everyone"
    },
    {
      "@odata.etag": "\"{B39D5D2E-E968-491A-B0EB-D5D0431CB423},1\"",
      "name": "Documents"
    },
    {
      "@odata.etag": "\"{9B51EA38-3EE6-4DC1-96A6-230E325EF054},2\"",
      "name": "newFile.txt"
    }
  ]
}
```

##使用筛选查询选项查询用户子集
若要查找组织中担任特定职务的员工，则您可以从用户集合中导航，然后指定 $filter 查询选项。示例如下所示：

    
```no-highlight 
GET https://graph.microsoft.com/v1.0/users/?$filter=jobTitle+eq+%27Helper%27 HTTP/1.1
Authorization : Bearer <access_token>
```

成功的响应返回 200 OK 状态代码，以及担任特定职务 (`'Helper'`) 的用户列表，如以下示例所示：

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal;odata.streaming=true;IEEE754Compatible=false;charset=utf-8
odata-version: 4.0
content-length: 986

{
    "@odata.context": "https://graph.microsoft.com/v1.0/contoso.onmicrosoft.com/$metadata#users",
    "value": [
        {
            "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
            "city": "Redmond",
            "country": "USA",
            "department": "Help Center",
            "displayName": "Jane Doe",
            "givenName": "Jane",
            "jobTitle": "Helper",
            ......
            "mailNickname": "Jane",
            "mobile": null,
            "otherMails": [
                "jane.doe@contoso.onmicrosoft.com"
            ],
            ......
            "surname": "Doe",
            "usageLocation": "US",
            "userPrincipalName": "help@contoso.onmicrosoft.com",
            "userType": "Member"
        },
        
        ...
    ]
}
```

##调用 OData 操作或函数
Microsoft Graph 还支持可操纵资源的 OData 操作和函数。 
例如，以下 HTTPS POST 请求允许登录用户 (`me`) 发送电子邮件：
```no-highlight 
POST https://graph.microsoft.com/v1.0/me/microsoft.graph.sendMail HTTP/1.1
authorization: bearer <access_token>
content-type: application/json
content-length: 96

{
  "message": {
    "subject": "Meet for lunch?",
    "body": {
      "contentType": "Text",
      "content": "The new cafeteria is open."
    },
    "toRecipients": [
      {
        "emailAddress": {
          "address": "garthf@a830edad9050849NDA1.onmicrosoft.com"
        }
      }
    ],
    "attachments": [
      {
        "@odata.type": "#Microsoft.OutlookServices.FileAttachment",
        "name": "menu.txt",
        "contentBytes": "bWFjIGFuZCBjaGVlc2UgdG9kYXk="
      }
    ]
  },
  "saveToSentItems": "false"
}
```

请求有效负载中包含对 `microsoft.graph.sendMail` 操作的输入（$metadata 中也对此进行了定义）。

Microsoft Graph 带有单一的统一终结点，能简化 Microsoft 云中的服务应用程序编程接口。 因此，其他孤立服务的边界会消失。 作为应用开发者，你不再需要持续跟踪数据源以及实现多种数据源之间的自定义接口。 


