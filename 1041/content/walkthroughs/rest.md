# Microsoft Graph および REST 入門

この記事では、Microsoft Graph を使用して Office 365 と Outlook.com の電子メール メッセージを取得する方法について説明します。 この記事では、OAuth および REST の要求と応答に焦点を当てます。 メッセージの認証と取得のためにアプリが使用する要求と応答を、順を追って説明します。

## OAuth 2.0 を使用して認証する

Microsoft Graph を呼び出すには、アプリに Azure Active Directory (Azure AD) からのアクセス トークンが必要です。 以下の例で、アプリは Azure AD からアクセス トークンを取得するために標準の [OAuth 2.0 プロトコル](http://tools.ietf.org/html/rfc6749)に従って認証コードの付与フローを実装します。

### アプリを登録する

現在、アプリを登録するには 2 つのオプションがあります。

  1. Office 365 の商用ユーザー、職場または学校のアカウントのみをサポートするモデルを使用して、アプリを登録します。
 
  このモデルは、Office 365 の商用プランでのみ動作します。アプリの登録後は、[Azure 管理ポータル](https://manage.windowsazure.com)から管理できます。

  2. コンシューマーと商用の両方の Office 365 サービスで機能する最新機能 (認証エンドポイント v2.0 と呼ぶ) を使用して登録します。
 
  職場や学校のアカウントと個人のアカウントの両方について、単一の認証サービスが使用可能になりました。このモデルでは、職場と学校 (Azure AD) と個人 (Microsoft) の両方の ID に単一の認証サービスを提供します。現在、アプリに単一の認証フローを実装するだけで、ユーザーは Office 365 や OneDrive for Business などの職場または学校のアカウント、あるいは Outlook.com や OneDrive などの個人アカウントを使用できるようになります。
   
[アプリケーション登録ポータル](https://apps.dev.microsoft.com/)を使用して、アプリを登録し、職場と学校のアカウントおよび個人アカウントの両方をサポートします。

v2.0 エンドポイントは、以前の認証エンドポイントからのすべてのシナリオをカバーするために段階的に拡張していることに注意してください。適切な選択をするには、[この記事](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/)をお読みください

アプリを登録すると、クライアント ID とシークレットを取得できます。これらの値は認証コードの付与フローで使用されます。

このドキュメントの残りの部分では、v2.0 モデルでの登録を想定しています。V2.0 エンドポイントでサポートされているフローに関する詳しいガイドは[この記事](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/)を、また認証コードの付与フローに関する詳しいガイドは[この記事](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/)をご参照ください

### 認証コードを取得する

認証コードの付与フローでは、まず認証コードを取得します。このコードは、ユーザーがログインし、アプリが要求したアクセス レベルに同意すると、承認サーバーによってアプリに返されます。

最初に、アプリはユーザーのログオン URL を構築します。この URLは、ユーザーがログインし、同意できるようにブラウザーで開く必要があります。

ログオンのベース URL は、`https://login.microsoftonline.com/common/oauth2/v2.0/authorize` のようになります。

アプリはこのベース URL にクエリ パラメーターを追加して、ログオンを要求しているアプリと、要求されているアクセス許可を承認サーバーに通知します。

- `client_id` - アプリを登録することによって生成されるクライアント ID です。これにより、どのアプリがログオンを要求しているかが Azure AD に通知されます。
- `redirect_uri` - ユーザーがアプリに同意した後、Azure によってリダイレクトされる場所です。この値は、アプリを登録したときに使用された**リダイレクト URI** の値に対応している必要があります。
- `response_type` - アプリが想定している応答タイプです。この値は、認証コードの付与フローの `code` です。
- `scope` - アプリが要求しているスコープをスペースで区切った一覧。詳細については、[この記事](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/)をご覧ください
- `state` - トークン応答でも返される要求に含まれている値。

たとえば、電子メールへの読み取りアクセスを要求するアプリケーションの要求 URL は次のようになります。

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<CLIENT ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
```

次に、ユーザーをログオン URL にリダイレクトします。ユーザーには、アプリ名が表示されているサインイン画面が表示されます。サインインすると、アプリが必要とするアクセス許可の一覧が表示され、許可するか拒否するかを尋ねられます。必要なアクセスが許可されると仮定した場合、ブラウザーは最初の要求で指定したリダイレクト URL にリダイレクトされ、クエリ文字列に認証コードが表示されます。

```http
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

OpenId Connect for Single Sign On も使用している場合は、追加のパラメーターが必要です。詳細については、[この記事](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/)をご参照ください。 

次の手順では、返された認証コードをアクセス トークンと交換します。

### アクセス トークンを取得する

アクセス トークンを取得するため、アプリはフォームでエンコードされたパラメーターに以下のパラメーターを付けてトークン要求 URL (`https://login.microsoftonline.com/common/oauth2/v2.0/token`) にポストします。

- `client_id`:アプリを登録することによって生成されるクライアント ID です。
- `client_secret`:アプリを登録することによって生成されるクライアント シークレットです。
- `code`:前の手順で取得した認証コードです。
- `redirect_uri`:この値は、認証コード要求で使用した値と同じである必要があります。
- `grant_type`:アプリが使用している付与の種類です。この値は、認証コードの付与フローの `code` です。
- `scope` - アプリが要求しているスコープをスペースで区切った一覧。詳細については、[この記事](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/)をご覧ください

前の手順のコードを使用する、アプリケーションの要求 URL は次のようになります。

```http
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
  &client\_id=<CLIENT ID>
  &client\_secret=<CLIENT SECRET>
}
```

サーバーはアクセス トークンを含む JSON ペイロードで応答します。

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

アクセス トークンは JSON ペイロードの `access_token` フィールドにあります。アプリが API に REST 呼び出しを行うときに、アプリはこの値を認証ヘッダーの設定で使用します。

## Microsoft Graph を呼び出し中

アクセス トークンを取得すると、Microsoft Graph を呼び出す準備ができます。 このサンプル アプリはメッセージを取得するため、`https://graph.microsoft.com/v1.0/me/messages` エンドポイントに対する HTTP GET 要求を使用します。

### 要求を絞り込む

アプリでは、OData クエリ パラメーターを使用して GET 要求の動作を制御できます。 アプリがこれらのパラメーターを使用して返される結果の数と、各項目に返されるフィールドの数を制限することをお勧めします。 

サンプル アプリでは、表にメッセージの件名、送信者、メッセージを受信した日時が表示されます。 表には最大 25 行が表示され、最新の受信メッセージが上位にくるように並べ替えられます。 アプリでは、次のクエリ パラメーターを使用してこれらの結果を取得します。

- `$select` - `subject`、`sender`、および `dateTimeReceived` の各フィールドのみを指定します。
- `$top` -最大で 25 項目を指定します。
- `$orderby` - `dateTimeReceived` フィールドに基づいて結果を並べ替えます。

これにより、次の要求が行われます。

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

これで、Microsoft Graph を呼び出す方法が理解できたため、API リファレンスを使用して、アプリで必要なその他の呼び出しを作成することができます。 ただしアプリでは、呼び出しを行うための適切なアクセス許可をアプリ登録で設定する必要がある点に注意してください。


