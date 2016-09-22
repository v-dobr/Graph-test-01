# Microsoft Graph のエラー応答とリソースの種類

<!--In this article:
  
-   [Status code](#msg_status_code)
-   [Error resource type](#msg_error_resource_type)
-   [Code property](#msg_code_property)

<a name="msg_error_response"> </a> -->

Microsoft Graph のエラーは、標準の HTTP 状態コード、
および JSON エラー応答オブジェクトを使用して返されます。

## HTTP 状態コード

次の表は、返される可能性のある HTTP ステータス コードの一覧とその説明を示します。

| 状態コード | ステータス メッセージ                  | 説明                                                                                                                            |
|:------------|:--------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|
| 400         | 要求が正しくありません                     | 形式が正しくない、または無効なため、要求を処理できません。                                                                       |
| 401         | 権限がありません                    | リソースの必要な認証情報が見つからないか、無効です。                                                   |
| 403         | Forbidden                       | 要求されたリソースへのアクセスが拒否されました。ユーザーに十分なアクセス許可がない可能性があります。                                                 |
| 404         | 見つかりません                       | 要求されたリソースは存在しません。                                                                                                  |
| 405         | メソッドが許可されていません              | 要求の HTTP メソッドはリソースで許可されていません。                                                                         |
| 406         | 許容されません                  | このサービスでは、Accept ヘッダーで要求された形式をサポートしていません。                                                                |
| 409         | Conflict                        | 現在の状態が要求に必要なものと競合しています。たとえば、指定された親フォルダーが存在しない可能性があります。                   |
| 410         | 使用されていないリソース                            | 要求されたリソースはサーバーで使用できなくなっています。                                               |
| 411         | 長さが必要                 | 要求に Content-Length ヘッダーが必要です。                                                                                    |
| 412         | 必須条件に失敗しました             | 要求で提供されている必須条件 (If-match ヘッダーなど) がリソースの現在の状態と一致しません。                       |
| 413         | 要求エンティティが大きすぎます        | 要求サイズが上限を超えています。                                                                                            |
| 415         | メディアの種類がサポートされていません          | 要求のコンテンツの種類がサービスによってサポートされていない形式です。                                                      |
| 416         | 要求された範囲が満たされません | 指定したバイト範囲が正しくない、または利用できません。                                                                                    |
| 422         | 処理できないエンティティです            | 意味的に正しくないため、要求を処理できません。                                                                       |
| 429         | 要求数が多すぎます               | クライアント アプリケーションは調整されており、一定の時間が経過するまで要求を繰り返しません。                |
| 500         | 内部サーバー エラー           | 要求の処理中に内部サーバー エラーが発生しました。                                                                       |
| 501         | 実装されていません                 | 要求された機能は実装されていません。                                                                                               |
| 503         | Service Unavailable             | サービスは一時的に利用できません。後で要求を繰り返すことができます。Retry-After ヘッダーがある可能性があります。                   |
| 507         | 記憶域の不足            | 記憶域の最大クォータに達しました。                                                                                            |
| 509         | 帯域幅の上限を超えました        | 帯域幅の上限を超えたため、アプリケーションが調整されました。 さらに時間が経過すると、アプリは要求を再試行できます。 |

エラー応答は、**エラー**という名前の 1 つのプロパティを含む 1 つの JSON オブジェクトです。 このオブジェクトには、すべてのエラーの詳細が含まれます。 
HTTP 状態コードの代わりに、またはこれに加えて、ここに返される情報を使用することができます。 完全な JSON エラー本体の例を以下に示します。

<!-- { "blockType": "example", "@odata.type": "sample.error", "expectError": true, "name": "example-error-response"} -->
```json
{
  "error": {
    "code": "invalidRange",
    "message": "Uploaded fragment overlaps with existing data.",
    "innerError": {
      "requestId": "request-id",
      "date": "date-time"
    }
  }
}
```

<!--<a name="msg_error_resource_type"> </a> -->

## エラー リソースの種類

要求の処理中にエラーが発生するたびに、エラー リソースが返されます。

エラー応答は、エラー応答に対する 
[OData v4](http://docs.oasis-open.org/odata/odata-json-format/v4.0/os/odata-json-format-v4.0-os.html#_Toc372793091) 
仕様の定義に従います。

### JSON 表記

エラー リソースは、これらのリソースで構成されます。

<!-- { "blockType": "resource", "@odata.type": "sample.error" } -->
```json
{
  "error": { "@odata.type": "odata.error" }  
}
```

#### odata.error のリソースの種類

エラー内で、応答は次のプロパティを含むエラー 
リソースです。

<!-- { "blockType": "resource", "@odata.type": "odata.error", "optionalProperties": [ "target", "details", "innererror"] } -->
```json
{
  "code": "string",
  "message": "string",
  "innererror": { "@odata.type": "odata.error" }
}
```

| プロパティ名  | 値                  | 説明\                                                                                               |
|:---------------|:-----------------------|:-----------------------------------------------------------------------------------------------------------|
| **code**       | string                 | 発生したエラーのエラー コード文字列                                                            |
| **message**    | string                 | 発生したエラーに関する開発者用メッセージ。 これはユーザーには直接表示されません。 |
| **内部エラー** | error object           | 省略可能。最上位レベルのエラーよりも詳細である可能性のある追加のエラー オブジェクト。                     |
<!-- {
  "type": "#page.annotation",
  "description": "Understand the error format for the API and error codes.",
  "keywords": "error response, error, error codes, innererror, message, code",
  "section": "documentation",
  "tocPath": "Misc/Error Responses"
} -->

<!--<a name="msg_code_property"> </a> -->

#### コードのプロパティ

`code` プロパティには次のいずれかの値が含まれます。 
アプリは、これらのエラーのいずれかを処理するように準備する必要があります。

| コード                      | 説明
|:--------------------------|:--------------
| **accessDenied**          | 呼び出し元にアクションを実行するアクセス許可がありません。 
| **activityLimitReached**  | アプリまたはユーザーが調整されました。
| **generalException**      | 不明なエラーが発生しました。
| **invalidRange**          | 指定したバイト範囲が無効か、使用できません。
| **invalidRequest**        | 要求は形式が正しくないか、無効です。
| **itemNotFound**          | リソースが見つかりませんでした。
| **malwareDetected**       | 要求されたリソースでマルウェアが検出されました。
| **nameAlreadyExists**     | 指定されたアイテム名は既に存在します。
| **notAllowed**            | このアクションはシステムで許可されていません。
| **notSupported**          | 要求はシステムでサポートされていません。
| **resourceModified**      | 呼び出し元による最後の読み取り以降に、更新対象のリソースが変更されました。通常、eTag の不一致です。
| **resyncRequired**        | デルタ トークンは無効になりました。アプリの同期状態をリセットする必要があります。
| **serviceNotAvailable**   | サービスが利用できません。 後で要求をもう一度お試しください。 Retry-After ヘッダーである可能性があります。 
| **quotaLimitReached**     | ユーザーがクォータ制限に達しました。
| **unauthenticated**       | 呼び出し元が認証されていません。

`innererror` オブジェクトには、さらに詳細な追加のエラー 
コードを持つ別の複数の `innererror` オブジェクトが再帰的に含まれる可能性があります。 
エラーを処理する際に、アプリは使用可能なすべてのエラー コード間をループして、
アプリが理解する最も詳細なコードを使用する必要があります。 
より詳細なコードの一部は、このページの下部に一覧表示されています。

エラー オブジェクトが予期したとおりのエラーであることを確認するには、`innererror` オブジェクト全体をループして、
予期したエラー コードを検索する必要があります。 次に例を示します。

```csharp
public bool IsError(string expectedErrorCode)
{
    OneDriveInnerError errorCode = this.Error;
    while (null != errorCode)
    {
        if (errorCode.Code == expectedErrorCode)
            return true;
        errorCode = errorCode.InnerError;
    }
    return false;
}
```

エラーを正しく処理する方法を示す例については、「
[エラー コードの処理](https://gist.github.com/rgregg/a1866be15e685983b441)」を参照してください。

ルートにある `message` プロパティには、開発者が読み取ることを目的としたエラー メッセージが含まれます。 
エラー メッセージはローカライズされておらず、直接ユーザーに表示すべきではありません。 
エラーを処理する際に、コードは `message` 値を入力するべきではありません。
これらの値はいつでも変更でき、多くの場合失敗した要求には固有の動的情報が含まれるためです。 
`code` プロパティで返されるエラー コードに対してのみ、
コードを記述する必要があります。

#### 詳細なエラー コード
入れ子になった `innererror` オブジェクト内でアプリが検出する可能性のある他のエラーの一部を、以下に示します。 
アプリでこれらの処理を行うことは必須ではありませんが、処理することもできます。 
サービスでは常に新しいエラー コードが追加されたり、以前のコードが返されなくなったりすることがあります。
そのため、[基本のエラー コード](#code-property)をすべてのアプリが処理できることが重要です。

| コード                               | 説明
|:-----------------------------------|:----------------------------------------------------------
| **accessRestricted**               | アイテムの所有者に限定されたアクセス。
| **cannotSnapshotTree**             | 一貫性のあるデルタ スナップショットを取得できませんでした。 後でもう一度お試しください。
| **childItemCountExceeded**         | 子アイテムの上限数に達しました。
| **entityTagDoesNotMatch**          | eTag が現在のアイテムの値と一致しません。
| **fragmentLengthMismatch**         | このフラグメントの宣言された合計サイズは、アップロード セッションのものとは異なります。
| **fragmentOutOfOrder**             | アップロードされたフラグメントは順序どおりではありません。
| **fragmentOverlap**                | アップロードされたフラグメントは既存のデータと重複しています。
| **invalidAcceptType**              | 承諾の種類が無効です。
| **invalidParameterFormat**         | パラメーターの書式が無効です。
| **invalidPath**                    | 名前に無効な文字が含まれています。
| **invalidQueryOption**             | クエリのオプションが無効です。
| **invalidStartIndex**              | 開始インデックスが無効です。
| **lockMismatch**                   | ロック トークンが既存のロックと一致しません。
| **lockNotFoundOrAlreadyExpired**   | 現在、期限切れでないロックがアイテムにありません。
| **lockOwnerMismatch**              | ロック所有者の ID が提供された ID と一致しません。
| **malformedEntityTag**             | eTag ヘッダーの形式が正しくありません。 eTag は、引用符で囲まれた文字列である必要があります。
| **maxDocumentCountExceeded**       | ドキュメントの上限数に達しました。
| **maxFileSizeExceeded**            | ファイル サイズの上限を超えました。
| **maxFolderCountExceeded**         | フォルダーの上限数に達しました。
| **maxFragmentLengthExceeded**      | ファイル サイズの上限を超えました。
| **maxItemCountExceeded**           | アイテムの上限数に達しました。
| **maxQueryLengthExceeded**         | クエリの長さの上限を超えました。
| **maxStreamSizeExceeded**          | ストリーム サイズの上限を超えました。
| **parameterIsTooLong**             | パラメーターの長さが上限を超えています。
| **parameterIsTooSmall**            | パラメーターの値が最小値未満です。
| **pathIsTooLong**                  | パスの長さが上限を超えています。
| **pathTooDeep**                    | フォルダー階層の深さの限度に達しました。
| **propertyNotUpdateable**          | プロパティを更新できません。
| **resyncApplyDifferences**         | 再同期が必要です。最後に同期したときに、サービスをローカル変更に対して最新の状態にしたことが確実な場合、すべてのローカル項目をサーバーのバージョンと置き換えます (削除を含む)。サーバーが把握していないすべてのローカル変更をアップロードします。
| **resyncRequired**                 | 再同期が必要です。
| **resyncUploadDifferences**        | 再同期が必要です。サービスが返さないすべてのローカル アイテムをアップロードして、サーバーのバージョンと異なるすべてのファイルをアップロードします (どちらがより最新の状態であるかわからない場合は、両方のコピーを保持する)。
| **serviceNotAvailable**            | サーバーが現在の要求を処理できません。
| **serviceReadOnly**                | リソースが一時的に読み取り専用になっています。
| **throttledRequest**               | 要求数が多すぎます。
| **tooManyResultsRequested**        | 要求された結果が多すぎます。
| **tooManyTermsInQuery**            | クエリ内のアイテムが多すぎます。
| **totalAffectedItemCountExceeded** | 影響を受けるアイテム数がしきい値を超えているため、操作は許可されません。
| **truncationNotAllowed**           | データの切り捨ては許可されていません。
| **uploadSessionFailed**            | アップロード セッションが失敗しました。
| **uploadSessionIncomplete**        | アップロード セッションが完了していません。
| **uploadSessionNotFound**          | アップロード セッションが見つかりません。
| **virusSuspicious**                | このドキュメントは疑わしいドキュメントであり、ウイルスを含んでいる可能性があります。
| **zeroOrFewerResultsRequested**    | 0 個以下の結果が要求されました。

<!-- ##Additional Resources##

- [Microsoft Graph API release notes and known issues](microsoft-graph-api-release-notes-known-issues.md )
- [Hands on lab: Deep dive into the Microsoft Graph API](http://dev.office.com/hands-on-labs/4585) -->
