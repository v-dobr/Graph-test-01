# Microsoft Graph 可选的查询参数
## OData 可选查询参数
Microsoft Graph 提供多个可选查询参数，您可以用来指定和控制响应中返回的数据量。Microsoft Graph 支持以下查询选项。 

|名称|值|说明|
|:---------------|:--------|:-------|
|$select|string|要在响应中添加的属性（以逗号分隔的列表）。|
|$expand|string|要在响应中扩展和添加的关系（以逗号分隔的列表）。  |
|$orderby|string|用于在响应集合中对项目进行排序的属性（以逗号分隔的列表）。|
|$filter|string|根据一组条件筛选响应。|
|$top|int|要在结果集中返回的项目数。|
|$skip|int|在结果集中跳过的项目数。|
|$skipToken|string|给用于获取下一个结果集的令牌进行分页。|
|$count|无|一个集合以及集合中的项目数。|


**编码的查询参数**

- 如果您尝试在 [Microsoft Graph 资源管理器](https://graphexplorer2.azurewebsites.net/)中查询参数，
您可以只复制和粘贴以下示例，无需将任何 URL 编码应用到查询字符串。以下示例_在 Graph 资源管理器中_运行正常，无需对空格和引号字符进行编码：
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 
- 一般情况下，当你在“_你的应用_”中指定查询参数，请确保相应的编码字符[保留 URI 中的特殊含义](https://tools.ietf.org/html/rfc3986#section-2.2)。
例如，在最后一个示例中对空格和引号字符进行编码，如下所示：
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address%20eq%20%27jon@contoso.com%27
```

### $select
若要指定返回的属性子集，请使用 **$select** 查询选项。例如，在检索邮件时，您可能希望选择仅返回邮件的**发件人**和**主题**属性。

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=from,subject
```

<!--For example, when retrieving the children of an item on a drive, you want to select that only the **name** and **size** properties of items are returned.

```http
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name,size
```

By submitting the request with the `$select=name,size` query string, the objects
in the response will only have those property values included. 


```json
{
  "value": [
    {
      "id": "13140a9sd9aba",
      "name": "Documents",
      "size": 1024
    },
    {
      "id": "123901909124a",
      "name": "Pictures",
      "size": 1012010210
    }
  ]
}
```--> 

### $expand

在 Microsoft Graph API 请求中，对参考项目的对象或集合的导航不会自动扩展。这是有意为之，
因为这样可以减少网络流量以及从服务生成响应所需的时间。不过，在某些情况下，您可能希望在响应中添加这些结果。

您可以使用 **$expand** 查询字符串参数，指示 API 扩展子对象或集合，并添加这些结果。

例如，若要检索根驱动器信息和驱动器中的顶级子项目，请使用 **$expand** 参数。
此示例还使用 **$select** 声明，以便仅返回子项的 _ID_ 和_名称_属性。

```http
GET https://graph.microsoft.com/v1.0/me/drive/root?$expand=children($select=id,name)
```

>  **注意**：请求最多可扩展 20 个对象。 

> 此外，如果您要查询[用户](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/user)资源，
您可以使用 **$expand**，一次仅可获取一个子对象 或集合的属性。 

以下示例获取**用户**对象时，每个在 **directReports** 扩展集合中都有多达 20 个 **directReport** 对象：
```http
GET https://graph.microsoft.com/v1.0/users?$expand=directReports
```
某些其他资源可能也会存在限制，因此务必检查可能存在的错误。


<!---The following shows a sample result that is returned in the response body.-->


### $orderby

若要指定从 API 返回的项目的排序顺序，请使用 **$orderby** 查询选项。 

例如，若要返回组织中的用户，并按显示名称进行排序，请使用以下语法：

```http
GET https://graph.microsoft.com/v1.0/users?$orderBy=displayName
``` 

你还可以按复杂的类型实体进行排序。以下示例获取邮件并按“**发件人**”属性（属于复杂类型 **emailAddress**）中的“**地址**”字段对这些邮件进行排序：

```http
GET https://graph.microsoft.com/v1.0/me/messages?$orderBy=from/emailAddress/address
``` 

若要以升序或降序排列结果，请在字段名称中附上 `asc` 或 `desc`（用空格隔开），
例如， `?orderby=name%20desc`.

 >  **注意**：**$orderby** 不能与 $filter 表达式结合使用。

### $filter
若要根据一组条件筛选响应数据，请使用 **$filter** 查询选项。例如，若要返回组织中的用户，
并筛选出以“Garth”开头的显示名称，请使用以下语法。

```http
GET https://graph.microsoft.com/v1.0/users?$filter=startswith(displayName,'Garth')
```

您还可以按复杂的类型实体进行筛选。以下示例将返回**发件人**属性的**地址**字段为“jon@contoso.com”的邮件。
**发件人**属性属于复杂的类型 **emailAddress**。

```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 

### $top
若要指定要在结果集中返回的最大项目数，请使用 **$top** 查询选项。**$top** 查询选项用于确定集合中的子集。此子集通过以下方式形成：仅选择集合的前 N 个项目，其中 N 是此查询选项指定的正整数。
例如，若要返回用户邮箱中的前 5 封邮件，请使用以下语法：

```http
GET https://graph.microsoft.com/v1.0/me/messages?$top=5
```

### $skip
若要在检索集合中项目之前，设置要跳过的项目数，请使用 **$skip** 查询选项。例如，若要返回按创建日期排序的事件，
并从第 21 个事件开始返回，请使用以下语法。

```http
GET  https://graph.microsoft.com/v1.0/me/events?$orderby=createdDateTime&$skip=20
```

### $skipToken
若要分页和指定返回的下一组结果，请使用 **$skipToken** 查询选项。**$skipToken** 查询选项的值是用于标识集合中起始点的令牌。例如，**$skipToken** 查询选项的值可以标识集合中的第一项，也可以标识包含 20 个项目的集合中的第 8 项，亦可标识集合中的其他任何位置。

由于 **$skipToken** 查询选项的值可标识集合索引，因此能够在查询中使用它来标识项目子集。标识的子集始于索引中的第一项（由 **$skipToken** 查询选项的值标识），一直到集合中的最后一项。

在一些响应中，您会看到 `@odata.nextLink` 值。其中一些也包括 **$skipToken** 值。**$skipToken** 值就像是一个标记，可指示服务在何处继续返回下一组结果。下面的示例展示了响应中的 `@odata.nextLink` 值。

```
"@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$orderby=displayName&$top=3&$skiptoken=X%2783630372100000000000000000000%27"
```

例如，若要返回组织中的下一组用户，并将人数限制为结果中每次返回 3 个，请使用以下语法。

```http
GET  https://graph.microsoft.com/v1.0/users?$orderby=displayName&$top=3&$skiptoken=X%2783630372100000000000000000000%27
```

### $count
使用 **$count** 作为查询参数，如以下示例所示：
```http
GET  https://graph.microsoft.com/v1.0/me/contacts?$count=true
```
这将同时返回**联系人**集合和 `@odata.id` 属性中**联系人**集合中的项目数。

注意：[directoryObject](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/directoryobject) 集合对此不支持。
