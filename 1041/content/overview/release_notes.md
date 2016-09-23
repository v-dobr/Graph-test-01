# Microsoft Graph のリリース ノートと既知の問題

この記事では、2015 年 11 月の Microsoft Graph API のリリースで使用可能になった開発者向けの新機能、および知っておくべき既知の問題について説明します。 


## Microsoft Graph API の GA の機能

次の Microsoft Graph API の機能が一般公開されています。

* ユーザー
* グループ
* ファイル
* メール
* 予定表
* 個人用連絡先 
* 作成、読み取り、更新、削除 (CRUD) 操作
* CORS (クロスオリジン リソース共有) サポート。

    
## Microsoft Graph API のプレビューの機能

次の Microsoft Graph API のプレビューの機能が公開されています。

* メモ 
* タスク
* Excel
* 連絡先
* 組織の連絡先
* アプリケーション
* サービス プリンシパル
* ディレクトリ スキーマの拡張
* Webhooks
* 洞察と関係:トレンディングおよび操作
* グループの作成後コンテンツへ即座にアクセス
* 個人アカウント、および仕事や学校のアカウントの統合認証モデル。 これはプレビュー機能ですが、`/v1.0` と `/beta` の両方のバージョンで使用できます。


## Microsoft Graph の既知の問題

Microsoft Graph に関する既知の問題は次のとおりです。

### ユーザー
#### 作成後にすぐにアクセスできない
ユーザーは、ユーザー エンティティの POST ですぐに作成できます。Office 365 サービスにアクセスするには、まず Office 365 ライセンスをユーザーに割り当てる必要があります。それでも、サービスが分散しているという性質上、Microsoft Graph API を介してこのユーザーがファイル、メッセージ、イベント エンティティを使用できるようになるまで 15 分かかる場合があります。この間、アプリには HTTP 404 エラー応答が送られてきます。 

#### 写真の制限
ユーザーのプロファイルの写真の読み取りと更新は、ユーザーが受信トレイを持っている場合にのみ使用できます。さらに、以前 **thumbnailPhoto** プロパティを使用して (Office 365 統合 API のプレビュー、Azure AD Graph、AD 接続同期を使用して) 保存*された*いかなる写真も Microsoft Graph ユーザー写真プロパティを使用してアクセスできなくなります。写真の読み取りまたは更新が失敗すると、この例では次のエラーが発生します:

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **注**:GA 後すぐ、ユーザーのプロファイルの写真の保存と取得が有効になり、ユーザーが受信トレイを持たない場合でも、このエラーは表示されなくなります。

#### 既定の連絡先フォルダー

`/v1.0` バージョンでは、`GET /me/contactFolders` にユーザーの既定の連絡先フォルダーは含まれません。 

修正プログラムが用意される予定です。それまでは、回避策として次の[連絡先一覧表示](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts)クエリと **parentFolderId** プロパティを使用して、
既定の連絡先フォルダーのフォルダー ID を取得できます。

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
上記のクエリで、
1. `/me/contacts?$top=1` は既定の連絡先フォルダーの[連絡先](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact)のプロパティを取得します。
2. `&$select=parentFolderId` を追加すると、連絡先の **parentFolderId** プロパティ (既定の連絡先フォルダーの ID) のみが返されます。

#### ユーザーのメールボックスに ICS ベースの予定表を追加してアクセスする
現在、インターネット予定表購読 (ICS) に基づく予定表の部分的なサポートがあります。
* ユーザー インターフェイスを通して ICS ベースの予定表をユーザーのメールボックスに追加できますが、Microsoft Graph API を通してこれを行うことはできません。 
* [ユーザーの予定表の一覧表示](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars)を行うと、ICS ベースの予定表を含む、ユーザーの既定の予定表のグループ、
または指定した予定表のグループにある各[予定表](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar)の**名前**、**色**、**ID** のプロパティを取得できます。 
予定表リソースの ICS URL を保存したり、アクセスしたりすることはできません。
* また、ICS ベースの予定表の[イベントを一覧表示](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events)することもできます。

### グループ
#### ポリシー
Microsoft Graph を使用して統合グループを作成および名前付けすると、Outlook Web App で構成されたすべての統合グループ ポリシーがバイパスされます。 

#### グループ アクセス許可のスコープ
Microsoft Graph では、グループ API にアクセスするために 2 つのアクセス許可のスコープ (*Group.Read.All* と *Group.ReadWrite.All*) を公開します。  これらのアクセス許可のスコープは、管理者による同意が必要です (これがプレビューとの違いです)。  将来的には、グループのためのユーザーが同意する新しいスコープを追加する予定です。

#### グループの投稿の添付ファイルを追加して取得する
現在のところ、グループの投稿への添付ファイルの[追加](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments)、
グループの投稿の添付ファイルの[一覧表示](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments)と取得を行うと、
「OData 要求がサポートされていません」というエラー メッセージが返されます。 `/v1.0` と `/beta` の両方のバージョンに対する修正プログラムが準備されており、2016 年 1 月の終わりまでには一般に利用可能になると予測されます。

### 連絡先
* 現在、個人用連絡先しかサポートされていません。 組織の連絡先は現在 `/v1.0` でサポートされていませんが、`/beta` で参照できます。
* 個人用連絡先の携帯電話番号が、連絡先で返されません。まもなく追加されます。その間、Outlook の API を通じてアクセスできます。

### ドライブ、ファイルおよびコンテンツのストリーミング
* ユーザーが自分の個人用サイトにブラウザーでアクセスする前に、Microsoft Graph でユーザーの個人ドライブに初めてアクセスした場合、401 応答になります。
* ファイル (Office グループやドライブ内のファイル、またはメール添付ファイル) のアップロードとダウンロードは、4 MB 以下に制限されています。

### 既存の Office 365 REST API でのみ使用可能な機能
#### 同期
Outlook、OneDrive、Azure AD 同期機能 (Azure AD では差分のクエリとも呼ばれます) は `/v1.0` や `/beta` では使用できません。  アプリケーションが同期機能を必要とする場合は、既存の Office 365 および Azure AD REST API を使用し続けてください。または、Microsoft Graph で提供されるイベント、メッセージ、連絡先のための新しい webhooks プレビュー機能を調べてみてください。

> **注**:同期を含め、可能な限り迅速に既存の API と Microsoft Graph の差を埋めることを目標としています。

#### バッチ処理
Microsoft Graph では、バッチ処理はサポートされていません。 ただし、Outlook のベータ版のエンドポイントを使用して、
[Outlook REST 呼び出しをバッチ処理](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests)することは可能です。 

#### 中国での可用性
Microsoft Graph サービスは 21Vianet によって運営されています (中国でも使用可能になりました)。制限を含む詳細については、「[Microsoft Graph の独立したクラウド展開](http://graph.microsoft.io/docs/overview/deployments)」をご覧ください。

#### サービスのアクションと関数
`isMemberOf` と `getObjectsById` は、Microsoft Graph では使用できません

### Microsoft Graph のアクセス許可
Microsoft Graph をサポートするアプリケーションと委任されたアクセス許可の詳細については、[アクセス許可のスコープに関するトピック](http://graph.microsoft.io/docs/authorization/permission_scopes)をご覧ください。  さらに、`v1.0` には次の制限があります:

|アクセス許可 |   アクセス許可の種類 | 制限 |  代替手段 |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| 委任    | 携帯電話番号を更新できません|    `Directory.AccessAsUser.All` も選択してください| 
|_User.ReadWrite.All_|  委任|  `User` では、ユーザーの HD 写真と拡張プロファイルのプロパティの更新以外の CRUD 操作を実行することはできません| ユーザーの削除が必要な場合は `Directory.ReadWrite.All` や `Directory.AccessAsUser.All` も選択してください。|
|_User.Read.All_|   アプリケーション |他のユーザーに対して読み取り操作を実行することはできません| `Directory.Read.All` も選択してください|
| _User.ReadWrite.All_ |    アプリケーション |   `User` では、ユーザーの HD 写真と拡張プロファイルのプロパティの更新以外の CRUD 操作を実行することはできません |    `Directory.ReadWrite.All` も選択してください。 **注**:ユーザーの削除を実行できなくなります。|
|_Group.Read.All_   | アプリケーション | グループまたはグループ メンバーシップを列挙することはできません。  Office グループのグループ コンテンツを読み取ることはできます   | `Directory.Read.All` も選択してください |
|_Group.ReadWrite.All_  | アプリケーション   | グループまたはグループ メンバーシップを列挙したり、グループを作成したり、グループ メンバーシップの更新やグループの削除はできません。Office グループのグループ コンテンツの読み取りや更新はできます。	   | `Directory.ReadWrite.All` も選択してください。 **注**:グループの削除を実行できなくなります。|

さらに、`/beta` には次の制限があります:

|アクセス許可 |   アクセス許可の種類 | 制限 |  代替手段 |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | 委任 | Office のグループでプランナーのタスクの読み取りおよび更新はできません	  | `Tasks.ReadWrite` も選択してください|
|_Tasks.ReadWrite_  | 委任 | サインイン中のユーザーのタスクを読み取りまたは更新することはできません| `Group.ReadWrite.All` も選択してください|

### OData 関連の制限事項
* **$expand** の制限: 
 * `nextLink` はサポートされていません
 * 1 レベルを超える展開はサポートされていません
 * 余分なパラメーターはサポートされていません (**$filter**、**$select**)
* 複数の名前空間はサポートされていません
* `$ref` の GET とキャストはユーザー、グループ、デバイス、サービス プリンシパル、アプリケーションではサポートされていません。
* `@odata.bind` はサポートされていません。  つまり、開発者は `Accepted` や `RejectedSenders` を適切にグループに設定することができません。
* `@odata.id` は最低限のメタデータを使用する場合、非包含構造のナビゲーション (メッセージなど) には存在しません
* ワークロード間でのフィルター/検索は利用できません。 
* フルテキスト検索 (**$search** を使用した検索) はメッセージなどのいくつかのエンティティに対してのみ使用できます。

  >  お客様からのフィードバックを重視しています。 [Stack Overflow](http://stackoverflow.com/questions/tagged/office365)でご連絡いただけます。 質問には [MicrosoftGraph] と [office365] でタグ付けしてください。

  
             .

