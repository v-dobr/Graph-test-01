# Microsoft Graph のオプション クエリ パラメーター
## オプションの OData クエリ パラメーター
Microsoft Graph にはいくつかのオプションのクエリ パラメーターがあり、指定して応答で返されるデータの量を制御するために使用できます。Microsoft Graph では、次のクエリ オプションがサポートされています。 

|名前|値|説明|
|:---------------|:--------|:-------|
|$select|string|応答に含めるプロパティを示すコンマ区切りのリスト。|
|$expand|string|展開して応答に含める関係を示すコンマ区切りのリスト。  |
|$orderby|string|応答コレクション内の項目の順序を並べ替えるために使用されるプロパティを示すコンマ区切りのリスト。|
|$filter|string|一連の基準に基づいて応答をフィルター処理します。|
|$top|int|結果セットに返す項目の数。|
|$skip|int|結果セットでスキップする項目の数。|
|$skipToken|string|次の結果セットを取得するために使用されるページング トークン。|
|$count|なし|コレクションおよびコレクション内の項目の数。|


**クエリ パラメーターのエンコード**

- [Microsoft Graph Explorer](https://graphexplorer2.azurewebsites.net/) でクエリ パラメーターを試している場合、クエリ文字列に URL エンコードを適用せずに以下の例をコピーして貼り付けることができます。
次の使用例は _Graph Explorer 内で_スペース、引用符文字をエンコードせずに正常に動作します。
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 
- 一般に、クエリ パラメーターを _in app_ に指定するとき、[URI で特別な意味のために予約](https://tools.ietf.org/html/rfc3986#section-2.2)されている文字が適切にエンコードされていることを確認してください。
たとえば、最後の例では以下のように、スペースと引用符文字をエンコードします。
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address%20eq%20%27jon@contoso.com%27
```

### $select
返されるプロパティのサブセットを指定するには、**$select** クエリ オプションを使用します。たとえば、メッセージを取得する場合、メッセージの **from** と **subject** プロパティだけが返されるように選択することができます。

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

Microsoft Graph API 要求では、参照されている項目のオブジェクトまたはコレクションへのナビゲーションは自動的には展開されません。
これは仕様です。サービスからの応答を生成するためのネットワーク トラフィックと時間を減らすためです。しかし、応答にそれらの結果を含めることが必要な場合があります。

**$expand** クエリ文字列パラメーターを使用して、子オブジェクトまたはコレクションを展開しそれらの結果を含めるよう API に指示できます。

たとえば、ルート ドライブ情報とドライブの最上位レベルの子項目を取得するには、**$expand** パラメーターを使用します。
またこの例では、子項目の **id** および _name_ プロパティだけを返すために、_$select_ ステートメントも使用しています。

```http
GET https://graph.microsoft.com/v1.0/me/drive/root?$expand=children($select=id,name)
```

>  **注**:要求に対して拡張されるオブジェクトの最大数は 20 です。 

> また、[ユーザー](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/user)のリソースにクエリを実行する場合、
プロパティを取得するために **$expand** を使用すると子オブジェクト またはコレクションを一度に 1 つずつしか取得できません。 

次の例では**ユーザー** オブジェクトを取得します。それぞれは展開された **directReports** コレクション内で最大 20 個の **directReport** オブジェクトを取得します。
```http
GET https://graph.microsoft.com/v1.0/users?$expand=directReports
```
他のいくつかのリソースにも制限がある場合があるため、潜在的なエラーを常にチェックします。


<!---The following shows a sample result that is returned in the response body.-->


### $orderby

API から返される項目の並べ替え順序を指定するには、**$orderby** クエリ オプションを使用します。 

たとえば、組織のユーザーを表示名順で戻す場合、構文は次のようになります。

```http
GET https://graph.microsoft.com/v1.0/users?$orderBy=displayName
``` 

複合型のエンティティでも並べ替えが可能です。次の例では、メッセージを取得し、それらを **emailAddress** という複合型である **from** プロパティの **address** フィールドで並べ替えます。

```http
GET https://graph.microsoft.com/v1.0/me/messages?$orderBy=from/emailAddress/address
``` 

昇順または降順で結果を並べ替えるには、`asc` または `desc` のいずれかをスペースで区切ってフィールド名の後に追加します。たとえば次のようにします。 
`?orderby=name%20desc`.

 >  **注**: **$orderby** は $filter 式と組み合わせることはできません。

### $filter
一連の基準に基づいて応答データをフィルターするには、**$filter** クエリ オプションを使用します。
たとえば、組織のユーザーを "Garth" で始まる表示名でフィルターして返す場合、構文は次のようになります。

```http
GET https://graph.microsoft.com/v1.0/users?$filter=startswith(displayName,'Garth')
```

複合型のエンティティでフィルターすることもできます。次の例では、**from** プロパティの **address** フィールドが "jon@contoso.com" と等しいメッセージを返します。
**from** プロパティは、複合型 **emailAddress** です。

```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 

### $top
結果セットに返す項目の最大数を指定するには、**$top** クエリ オプションを使用します。 **$top** クエリ オプションは、コレクションのサブセットを指定します。
このサブセットは、最初の N 項目のみが選択されて形成されます。ここで、N はこのクエリ オプションで指定される正の整数です。たとえば、ユーザーのメールボックスの最初の 5 つのメッセージを返す構文は、次のとおりです。

```http
GET https://graph.microsoft.com/v1.0/me/messages?$top=5
```

### $skip
コレクション内の項目を取得する前にスキップする項目数を設定するには、**$skip** クエリ オプションを使用します。
たとえば、作成日付順に並べ替えたイベントを 21 番目から戻す構文は、次のとおりです。

```http
GET  https://graph.microsoft.com/v1.0/me/events?$orderby=createdDateTime&$skip=20
```

### $skipToken
次に返される結果セットをページ単位で指定するには、**$skipToken** クエリ オプションを使用します。**$skipToken** クエリ オプションの値は、コレクション内の開始点を指定するトークンです。たとえば、**$skipToken** クエリ オプションの値によって、コレクションの最初の項目を指定したり、20 個の項目を含むコレクション内の 8 番目の項目やコレクション内の他の任意の位置の項目を指定できます。

**$skipToken** クエリ オプションはコレクションにインデックスを指定するので、クエリでこのオプションを使用して項目のサブセットを指定できます。セット内の (**$skipToken** クエリ オプションの値によって指定される) インデックスの最初の項目から始まり、セットの最後の項目で終わるサブセットです。

応答によっては、`@odata.nextLink` という値が表示される場合があります。それらには、**$skipToken** 値が含まれる場合があります。**$skipToken** 値は、次の結果セットを再開する位置をサービスに示すマーカーのようなものです。次の例は、応答の `@odata.nextLink` 値の例です。

```
"@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$orderby=displayName&$top=3&$skiptoken=X%2783630372100000000000000000000%27"
```

たとえば、結果で一度に返す数を 3 に制限して、組織内のユーザーの次のセットを返す構文は、次のとおりです。

```http
GET  https://graph.microsoft.com/v1.0/users?$orderby=displayName&$top=3&$skiptoken=X%2783630372100000000000000000000%27
```

### $count
次に示す例のように、**$count** をクエリ パラメーターとして使用します。
```http
GET  https://graph.microsoft.com/v1.0/me/contacts?$count=true
```
これは**連絡先**コレクションおよび`@odata.id`プロパティにある**連絡先**コレクション内のアイテムの数の両方を返します。

注:これは [directoryObject](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/directoryobject) コレクションではサポートされていません。
